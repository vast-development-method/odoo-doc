# Calculations

Every formula and algorithm of the Loyalty and Promotions domain, with its inputs, its outputs, its precision, its order of operations and at least one worked example with real numbers. The algorithms are written as numbered steps over named quantities; they are the authoritative description of the behavior that a replacement implementation must reproduce.

## 1. Notation, precision and rounding

| Symbol | Meaning |
|---|---|
| `round(x, n)` | Round `x` half away from zero to `n` decimal places. |
| `round_down(x, n)` | Truncate `x` towards zero to `n` decimal places. |
| `round_currency(x, c)` | Round `x` to the rounding step of currency `c` (normally two decimal places, but a currency may declare any step, including a step larger than one). |
| `floor(x)` | The largest integer less than or equal to `x`. |
| `convert(x, from, to)` | Convert amount `x` from currency `from` into currency `to` using the rate of the program's company (or the acting company when the program has none) at today's date. |
| `document currency` | The currency of the document being evaluated (the sales order, the ticket, the cart). |
| `program currency` | The `currency_id` of the Loyalty Program. |

Precision rules:

1. Point values are stored and compared with **two decimal places**. Every point computation that is not exact truncates downward to two decimal places, never upward, so that a customer is never credited a fraction of a point they did not earn.
2. A point value read from a card is rounded to the **program currency's rounding step** only when at least one rule of the program grants points per unit of currency spent. For a program whose rules grant points per order or per unit paid, the value is used unrounded. This distinction matters for a currency whose rounding step is larger than one: a single point of an order-based program must not be collapsed to zero.
3. Monetary reward amounts are rounded by the document currency when they are written onto a line; intermediate factors are kept at full precision.
4. Comparisons between two monetary amounts use the document currency's rounding step: two amounts that differ by less than half a step are equal.

## 2. The reference date of a document

Program validity, card expiry and usage limits are judged against a single reference date computed per document.

1. Determine the **evaluation time zone**: the time zone of the company's own contact record; when it is empty, the time zone stored in the system parameter `loyalty.timezone`; when that parameter is absent, coordinated universal time. For an online cart, the time zone of the website's salesperson takes precedence over both, when that user has one.
2. Collect the creation timestamps of every payment transaction of the document whose state is "done" or "authorized".
3. When at least one such transaction exists, the reference date is the **earliest** of those creation timestamps, converted into the evaluation time zone and reduced to a date.
4. When none exists, the reference date is today's date in the evaluation time zone.

Rationale: a shopper who starts paying at 23:58 on the last day of a promotion and whose payment is confirmed at 00:02 keeps the promotion, because validity is judged at the moment the payment was accepted, not at the moment the order is finally written.

Worked example: the company contact declares the `Europe/London` time zone. A cart is paid with a transaction created at 2026-03-31 23:10 coordinated universal time; the transaction is confirmed and a second transaction is created at 2026-04-01 00:20. The earliest confirmed transaction is the first one; converted into `Europe/London` (one hour ahead in summer time) it is 2026-04-01 00:10, so the reference date is 2026-04-01. A program whose `date_to` is 2026-03-31 is **not** applicable.

## 3. Applicability filters

### 3.1 The program filter

A Loyalty Program is applicable to a document when all of the following hold. `today` is the reference date of section 2.

- `active` is true;
- the channel flag of the document's channel is true;
- `company_id` is empty, or it is the document's company, or it is a parent of the document's company;
- `pricelist_ids` is empty, or it contains the document's price list;
- `date_from` is empty, or it is not later than `today`;
- `date_to` is empty, or it is not earlier than `today`.

The **sales channel flag** is `sale_ok` for a sales order. When the order belongs to a website, the leaf is replaced by `ecommerce_ok = true` and an extra condition is added:

- `website_id` is empty, or it equals the document's storefront.

### 3.2 The rule filter

A Loyalty Rule is reachable from a document when the same conditions hold on its program:

- the rule's `active` is true;
- the program's channel flag for that document's channel is true;
- the program's `company_id` is empty, or it is the document's company, or a parent of it;
- the program's `pricelist_ids` is empty, or it contains the document's price list;
- the program's `date_from` is empty, or it is not later than `today`;
- the program's `date_to` is empty, or it is not earlier than `today`.

with the same website substitution when the document belongs to a website.

### 3.3 The automatic candidate filter

When a document is re-evaluated, the programs considered for automatic application are those that satisfy the program filter and additionally:

- the program is not already granting points on this document;
- its `trigger` is `auto`;
- at least one of its rules has `mode` equal to `auto`;
- `limit_usage` is false, or `total_order_count` is strictly below `max_usage`.

### 3.4 The valid product set of a rule

For a rule `R` and a set of products `P`:

1. Build the product condition of `R`: the disjunction of `identifier IN R.products` (when `R.product_ids` is not empty), `category IS category_of(R.product_category) OR ANY DESCENDANT` (when `R.product_category_id` is set) and `tags CONTAINS R.product_tag` (when `R.product_tag_id` is set). When none of the three is set, the disjunction is the always-true condition. When `R.product_domain` is not `[]`, combine it with a conjunction.
2. When the condition is a real restriction, the valid set is the subset of `P` that satisfies it.
3. When the condition is the always-true condition and the program type is **not** `gift_card`, the valid set is the whole of `P`.
4. When the condition is the always-true condition and the program type **is** `gift_card`, the rule is **omitted entirely** from the result. A gift card program that names no trigger product therefore grants no points and issues no card.

## 4. Points granted by a document to a program

This is the central computation. Its inputs are the document and a set of programs; its output, per program, is either an error message or a list of point values. The first element of the list is the aggregated point count; every further element is a separate card to create, used by the split option.

### 4.1 Preparation

1. **Counting lines**: the document lines that carry a product and no reward, excluding lines that are items of a combination product. When the shipping capability package is present, shipping lines are excluded as well.
2. **Quantities per product**: for every counting line, convert the line quantity from the line's unit of measure into the product's reference unit of measure and add it to that product's total.
3. **Valid products per rule**, computed with section 3.4 over the products of the counting lines.
4. **Valid products per rule over the whole document**, computed with section 3.4 over the products of every line of the document, including reward lines. This second map is used for the amount computations.
5. **Threshold lines per rule**: start from every line of the document except the lines that have no effect on the threshold (shipping lines and free shipping reward lines when the shipping capability package is present; nothing otherwise). Skip a line when it is a discount reward line of an automatic program, and skip a line that is an item of a combination product. For each remaining line and each program being evaluated, skip the line when it is a discount reward line of that same program; then, for each rule of the program, add the line to that rule's threshold lines when the line's product belongs to the rule's whole-document valid set. A combination product line contributes its item lines instead of itself.

The two skips implement a deliberate asymmetry: a discount that the system applied by itself does not reduce the amount used to judge whether a threshold is met, whereas a discount the customer had to unlock with a code does.

### 4.2 Per-program evaluation

Let `code_matched`, `minimum_amount_matched` and `product_qty_matched` all start at `true` when the program has **no rule at all** and its `applies_on` is `current`, and at `false` otherwise. Let `points` be zero and `split_points` be an empty list.

For each rule `R` of the program, in the program's rule order:

1. **Bottomless wallet guard**: when the program type is `ewallet` and the program names no trigger product, stop evaluating this program's rules entirely.
2. **Code gate**: when `R.mode` is `with_code` and `R` is not among the document's activated code rules, skip this rule.
3. Set `code_matched` to true.
4. **Amount gate**: let `threshold = convert(R.minimum_amount, program currency, document currency)`; let `untaxed` be the sum of the tax-excluded subtotals of `R`'s threshold lines and `taxed` the sum of their tax amounts. When `R.minimum_amount_tax_mode` is `incl`, compare `threshold` with `untaxed + taxed`, otherwise with `untaxed`. When `threshold` is strictly greater than the compared amount, skip this rule.
5. Set `minimum_amount_matched` to true.
6. **Product gate**: when `R` has no entry in the valid-products map (which happens for an unrestricted gift card rule), skip this rule. Let `matched_quantity` be the sum of the per-product quantities of `R`'s valid products. When `matched_quantity` is strictly less than `R.minimum_qty`, or when the valid set is empty, skip this rule.
7. Set `product_qty_matched` to true.
8. When `R.reward_point_amount` is zero, skip this rule.
9. **Grant**:
   - **Split branch**, taken when the program's `applies_on` is `future`, `R.reward_point_split` is true and `R.reward_point_mode` is not `order`:
     - mode `unit`: append `R.reward_point_amount` to `split_points` once for each whole unit of `matched_quantity` (the quantity is truncated to an integer).
     - mode `money`: for every document line that is not a reward line, is not a combination item, whose product belongs to `R`'s valid set and whose quantity is strictly positive, let `line_total` be the tax-included total of the line (the sum over its priced item lines for a combination product) and compute
       ```formula
       points_per_unit = round_down(R.reward_point_amount × line_total ÷ line_quantity, 2)
       ```
       When `points_per_unit` is zero, skip the line; otherwise append `points_per_unit` to `split_points` once for each whole unit of the line quantity.
   - **Aggregate branch**, taken otherwise:
     - mode `order`: `points = points + R.reward_point_amount`.
     - mode `money`: let `amount_paid` be zero; for every document line except the threshold-neutral ones, skip the line when it is a combination item or when it is a reward line of a program of type `gift_card`, of type `ewallet`, or of the same type as the program being evaluated; otherwise add the line's tax-included total to `amount_paid` when the line's product belongs to `R`'s whole-document valid set. Then
       ```formula
       points = points + round_down(R.reward_point_amount × amount_paid, 2)
       ```
     - mode `unit`: `points = points + R.reward_point_amount × matched_quantity`.

The `money` mode deliberately counts what was actually paid, so a discount already applied reduces the points earned: one point per unit of currency on a purchase of 100 that carries a 30 percent discount yields 70 points.

### 4.3 Result or error

1. When the program is **not** nominative:
   - `code_matched` false: the result is the error `This program requires a code to be applied.`
   - otherwise `minimum_amount_matched` false: the result is the error `To take advantage of this offer, your order must include at least <amount> <currency name> of the eligible products.`, where `<amount>` is the smallest `minimum_amount` among the program's rules and `<currency name>` is the name of the program currency.
   - otherwise `product_qty_matched` false: the result is the error `You don't have the required product quantities on your sales order.`
2. When the program **is** nominative and the document's customer is the public customer and the document does not allow nominative programs, the result is the error `This program is not available for public users.`
3. Otherwise the result is the list `[points] + split_points`.

A nominative program with no matching rule therefore still produces the list `[0]`, which is what keeps a customer's loyalty card and electronic wallet attached to the document even when the document earns nothing.

### 4.4 Worked example: points per unit of currency spent

A loyalty program grants `0.1` point per unit of currency spent on every product, with `applies_on` `both`. The order carries one line: quantity 3 of a product priced 100.00 with no tax, and a 10 percent global discount reward already applied by an automatic program.

1. Counting lines: the product line only (the discount line carries a reward).
2. Threshold lines for the rule: the product line only; the discount reward line is skipped because its program is automatic.
3. Amount gate: `minimum_amount` is zero, so the gate passes.
4. Product gate: the rule has no product filter, `matched_quantity` is 3, `minimum_qty` is 1, so the gate passes.
5. Aggregate branch, mode `money`: the loop over document lines includes the product line (tax-included total 300.00) and the discount reward line (tax-included total −30.00, kept because its program type is `promotion`, not a payment program and not the loyalty program being evaluated). `amount_paid = 300.00 − 30.00 = 270.00`.
6. `points = round_down(0.1 × 270.00, 2) = 27.00`.

## 5. Points available on a card for a document

Input: a card `C` and a document `D`. Output: the number of points that may be spent on `D` right now.

1. `points = C.points` (the persisted balance).
2. When `D` is not yet confirmed:
   - when `C`'s program has `applies_on` other than `future`, add the pending point promise of `D` towards `C`;
   - subtract the sum of the point costs of every line of `D` whose card is `C`.
3. When any rule of `C`'s program has `reward_point_mode` equal to `money`, set `points = round_currency(points, C.currency)`.
4. Return `points`.

Step 2 is what allows a customer to earn and spend on the same order: an order that will grant 120 points to a loyalty card holding 30 makes 150 points available immediately, minus whatever the already-claimed rewards on that order cost.

Worked example: a card holds 30.00 points. The order will grant 120.00 points (pending promise) and already carries a reward line costing 100.00 points. The program grants points per unit of currency spent and the program currency rounds to two decimals. Available points `= round_currency(30.00 + 120.00 − 100.00, two decimals) = 50.00`.

### 5.1 Point changes of a document

The net effect of a document on every card, used at confirmation and at cancellation:

1. Start with a change of zero for every card.
2. For each pending point entry of the document, add its `points` to the change of its card.
3. For each line of the document that carries both a reward and a card, subtract that line's
   `points_cost` from the change of that card.

Confirming the document adds `change[card]` to each card's balance; cancelling a confirmed document subtracts it again.

## 6. Claimable rewards

Input: a document, and optionally a forced set of cards. Output: a mapping from card to the set of rewards that card can pay for right now.

1. Let `cards` be the forced set when one is given; otherwise the union of the cards of the document's pending promises, the cards referenced by its lines, and its manually applied cards. When `cards` is empty, return an empty mapping.
2. Let `has_payment_reward` be true when at least one line of the document carries a reward of a payment program.
3. Let `applied_global` be the reward of the first line of the document whose reward is a global discount, or nothing.
4. Let `discountable` be the discountable amount of the document ignoring `applied_global` (section 8.1). Evaluate it lazily: it is only needed when a global discount or a discount reward is examined.
5. Let `total_is_zero` be true when `discountable` is zero within the document currency's rounding step.
6. For each card `C` in `cards`:
   1. Skip `C` when its program's `applies_on` is `future` and `C` was generated by this very document. Points earned for future orders may not be spent on the order that earned them.
   2. Skip `C` when it has an expiration date strictly earlier than the document's reference date.
   3. Let `points` be the points available (section 5).
   4. For each reward `W` of `C`'s program:
      1. Skip `W` when it is a global discount, a global discount is already applied, and the applied one is at least as good (section 7).
      2. Skip `W` when it is a discount, `total_is_zero` is true, and either no payment reward is applied or `W` itself belongs to a payment program. A discount on a zero total is meaningless, except that a discount may still be claimed on top of a gift card that already brought the total to zero, because removing the gift card would restore the total.
      3. Skip `W` when it is a discount that does not belong to a payment program and is already applied on the document. A payment reward, by contrast, may be applied several times, once per card.
      4. Skip `W` when it is a free product reward whose product is archived: a reward of type `product` must have either an active reward product, when it names no tag, or at least one active product behind its tag.
      5. Keep `W` when `points` is greater than or equal to `W.required_points`.

## 7. Comparing two global discounts

Only one global discount may be applied at a time. When a second one appears, the following decision selects the one to keep. Inputs: the currently applied reward `A`, the candidate reward `B`, and optionally a precomputed discountable amount.

1. When `A` and `B` are the same reward, `A` wins.
2. When no discountable amount is given, compute it as the discountable amount of the document ignoring `A` (section 8.1). A value of zero that was explicitly provided is used as is, and is not recomputed.
3. Compute the discount amount of each reward against that discountable amount:
   - mode `per_order`: `convert(reward.discount, program currency, document currency)`;
   - mode `percent`: `discountable × reward.discount ÷ 100`.
4. When **both** amounts are greater than or equal to the discountable amount, the **smaller** one wins: the customer saves the same amount either way and keeps the more valuable voucher for another purchase. `A` wins when its amount is less than or equal to `B`'s.
5. Otherwise `A` wins when its amount is greater than or equal to `B`'s.

When `A` loses, its lines are reset and reused for `B`.

Worked example: the order's discountable amount is 500.00. The applied reward is a fixed discount of 80.00 and the candidate is a 10 percent discount. `A` gives 80.00, `B` gives 50.00. Neither exceeds 500.00, so `A` wins because 80.00 is greater than 50.00; the candidate is refused with `A better global discount is already applied.`

Second worked example: the order's discountable amount is 40.00. The applied reward is a fixed discount of 80.00 and the candidate is a fixed discount of 50.00. Both exceed 40.00, so the smaller one wins: `A` gives 80.00 which is greater than 50.00, so `A` loses and the candidate replaces it, leaving the eighty-unit voucher unspent.

## 8. Discountable amounts

A discountable amount is what a discount is allowed to reduce. Every discount computation produces two results: a single total, and a breakdown per tax combination. The breakdown is what allows the discount to be split into one line per tax so that the tax amounts of the document stay exact.

Fixed-amount taxes (a flat amount per unit, such as an environmental levy) are **never** discounted for a non-payment program: they are excluded from both the total and the breakdown. A payment program (gift card, electronic wallet) discounts everything, including fixed-amount taxes, because it is a means of payment and must be able to bring the total to zero.

### 8.1 The comparison amount

Used only to compare two global discounts, never to build a line.

1. Start at zero.
2. For every line of the document except the threshold-neutral ones:
   - skip the line when its reward is one of the rewards being ignored;
   - skip the line when its quantity is zero or its unit price is zero;
   - compute the tax breakdown of `unit price × quantity` with the line's taxes;
   - add the tax-excluded base, then add the amounts of the line's taxes that are not fixed-amount taxes.
3. Return the total.

This amount is computed from the unit price and the quantity only; a per-line discount percentage written on the line is not applied. It is used solely for the relative comparison of two rewards.

Worked example. An order in a currency with two decimal places and a rounding step of 0.01 carries five lines, and reward `W` is the reward being ignored:

| Line | Quantity | Unit price | Taxes | Other |
|---|---|---|---|---|
| 1 | 2 | 50.00 | one tax-excluded tax of 15 percent | a per-line discount of 10 percent is written on the line |
| 2 | 1 | 30.00 | one fixed amount of 5.00 per unit | none |
| 3 | 1 | 12.00 | none | a shipping line, therefore threshold-neutral |
| 4 | 3 | 0.00 | none | none |
| 5 | 1 | −25.00 | none | a reward line of `W` |

Applying step 2 line by line:

- Line 1 is kept. `unit price × quantity = 50.00 × 2 = 100.00`; the per-line discount of 10 percent is deliberately ignored. The tax breakdown of 100.00 with a 15 percent tax-excluded tax is a base of 100.00 and a tax of 15.00, which is not a fixed-amount tax, so the line contributes `100.00 + 15.00 = 115.00`. Running total 115.00.
- Line 2 is kept. Base `30.00 × 1 = 30.00`; its only tax is a fixed amount of 5.00, which is excluded, so the line contributes 30.00. Running total 145.00.
- Line 3 is skipped: it is threshold-neutral.
- Line 4 is skipped: its unit price is zero.
- Line 5 is skipped: its reward is among the rewards being ignored.

The comparison amount is **145.00**. A candidate global discount of 10 percent is therefore worth `145.00 × 0.10 = 14.50` for the comparison of section 7, and a candidate fixed discount of 20.00 is worth 20.00; the fixed one is the larger of the two and wins.

### 8.2 Order applicability

Input: a reward whose applicability is `order`.

1. Let `lines` be every line of the document that is not a section, a subsection or a note.
2. When the reward's program is **not** a payment program, remove the threshold-neutral lines (shipping lines and free shipping reward lines).
3. When the reward's program **is** a payment program, remove instead the lines whose product is one of the program's trigger products. This prevents topping up an electronic wallet by paying with that same wallet.
4. For each remaining line, prepare its tax base; let `discount_taxes` be the line's taxes and `discountable_taxes` be the same taxes with every tax group flattened into its children. When the program is not a payment program, remove the fixed-amount taxes from both sets.
5. Compute and round the tax details of all the prepared lines together, using the company's tax rounding method.
6. Aggregate the tax details by the pair (`discount_taxes` of the line, a skip flag). A detail is skipped when its tax is not in the line's `discountable_taxes` or when the line is not among `lines`.
7. For every non-skipped group:
   ```formula
   discountable = discountable + raw_base + raw_tax
   discountable_per_tax[taxes] = discountable_per_tax[taxes]
                                 + raw_base
                                 + sum of raw_tax of the details whose tax is tax-included
   ```
8. Return the total and the breakdown.

The two sums differ on purpose. The **total** is what the customer perceives: base plus every discountable tax. The **breakdown** is what is written on the reward lines: for a tax-excluded tax the line carries the base only and the system recomputes the tax on the negative line; for a tax-included tax the line carries base plus tax, because that is what a tax-included unit price means.

### 8.3 Cheapest applicability

1. Find the **cheapest line**: among the document lines that are not threshold-neutral, ignore lines that carry a reward, lines that are items of a combination product, lines with zero quantity, lines whose unit price (the sum of the unit prices of its priced item lines for a combination product) is zero, and lines whose product does not satisfy the reward's discounted-product filter. Keep the line with the smallest unit price; ties are broken by document order, the first one encountered winning. Expand a combination product line into its priced item lines.
2. When there is no such line, return no discountable amount at all; the reward cannot be applied.
3. Otherwise, for each line of the expansion:
   ```formula
   discountable = discountable + line.tax_included_total ÷ line.quantity
   discountable_per_tax[non-fixed taxes of the line] += line.unit_price × (1 − line.discount ÷ 100)
   ```

The division by the quantity is what makes the reward apply to **one unit** of the cheapest line rather than to the whole line.

Worked example. An order with no tax carries line 1 of 3 units at 30.00 (tax-included total 90.00) and line 2 of 2 units at 10.00 (tax-included total 20.00). The reward is a 50 percent discount with `discount_applicability` `cheapest` and a discounted-product filter that matches both products.

1. Neither line is threshold-neutral, carries a reward, is a combination item, has a zero quantity or has a zero unit price, and both products satisfy the filter. The unit prices are 30.00 and 10.00, so the cheapest line is line 2.
2. A cheapest line exists, so the computation continues.
3. The expansion is line 2 alone: `discountable = 20.00 ÷ 2 = 10.00`, and the breakdown for the empty tax set is `10.00 × (1 − 0 ÷ 100) = 10.00`.

The reward of section 9.2 then produces `max_discount = 10.00 × 0.50 = 5.00`, so the reward line carries **−5.00**, which is half of one unit of the cheaper product and not half of the whole line.

Tie-breaking and per-line discounts. If a third line of 4 units at 10.00 is added after line 2, both lines share the smallest unit price of 10.00 and line 2 wins because it comes first in document order. If line 2 instead carries a per-line discount of 20 percent, its tax-included total is `10.00 × 2 × 0.80 = 16.00`, so `discountable = 16.00 ÷ 2 = 8.00` and the breakdown is `10.00 × 0.80 = 8.00`, giving a reward line of −4.00.

### 8.4 Specific applicability

This is the most intricate computation, because a specific discount must never push a line below zero when another discount has already eaten into it.

1. Let `lines_to_discount` be the document lines (excluding threshold-neutral ones) that carry no reward, are not combination items, whose product satisfies the reward's discounted-product filter, and whose quantity and tax-included total are both non-zero. Combination product lines are expanded into their priced item lines.
2. Let `remaining[line]` be the tax-included total of every document line (excluding threshold-neutral ones) whose quantity and total are non-zero.
3. Group the existing discount reward lines by their reward grouping code into `discount_groups`.
4. Let `plain_lines` be the document lines (excluding threshold-neutral ones) that carry no reward at all.
5. For each group `G` of `discount_groups`, with reward `W`:
   - `affected` is `plain_lines` when `W.discount_applicability` is `order`, the cheapest-line expansion of `W` when it is `cheapest` (computed once and reused), and the specific discountable lines of `W` when it is `specific`. When `affected` is empty, skip the group.
   - When `W.discount_mode` is `percent`: for every line of `affected`, multiply `remaining[line]` by `(1 − W.discount ÷ 100)` for applicability `order` and `specific`, and by `(1 − W.discount ÷ 100 ÷ line.quantity)` for applicability `cheapest`.
   - When `W.discount_mode` is not `percent` (a fixed amount): build `budget[tax set]` from the absolute tax-included totals of the lines of `G`, keyed by the non-fixed taxes of each of those lines. Then walk the lines of `affected` that are **not** in `lines_to_discount` first, and the ones that are in `lines_to_discount` afterwards; for each such line take `key` as the non-fixed taxes of the group's lines when `W` belongs to a payment program and the non-fixed taxes of the walked line otherwise; when `budget[key]` is zero, continue; otherwise consume `min(remaining[line], budget[key])` from both `budget[key]` and `remaining[line]`.
6. For each line of `lines_to_discount`:
   ```formula
   discountable = discountable + remaining[line]
   line_base = line.unit_price × line.quantity × (1 − line.discount ÷ 100)
   discountable_per_tax[non-fixed taxes of the line] += line_base × (remaining[line] ÷ line.tax_included_total)
   ```

The ordering in step 5 (lines outside the new reward's scope first) is what makes an existing fixed discount consume the budget it can only have consumed on lines the new reward does not touch, before it eats into the lines the new reward wants.

Worked example, cascading percentages: an order carries quantity 3 of a product priced 100.00 with no tax, total 300.00. A promotion offers a 10 percent discount on specific products matching everything, and may be claimed once per point earned (one point per unit, so three points).

- First application: no discount group exists yet; `remaining[line] = 300.00`; `discountable = 300.00`; the breakdown for the empty tax set is `300.00 × (300.00 ÷ 300.00) = 300.00`; `max_discount = 300.00 × 0.10 = 30.00`; the reward line carries `−30.00`. Order total 270.00.
- Second application: one discount group exists with reward `W` of mode `percent` and applicability `specific`; `affected` is the product line; `remaining[line] = 300.00 × 0.9 = 270.00`; `discountable = 270.00`; `max_discount = 27.00`; the new reward line carries `−27.00`. Order total 243.00.
- Third application: two discount groups exist; `remaining[line] = 300.00 × 0.9 × 0.9 = 243.00`; `max_discount = 24.30`. Order total 218.70.

## 9. Building the reward lines

### 9.1 Common values

Every reward line receives:

- the reward's `discount_line_product_id` (for a discount and for free shipping) or the chosen free product (for a free product reward);
- quantity 1 for a discount and for free shipping;
- the reward reference, the card reference and a freshly generated random reward grouping code, shared by every line of this application;
- a sequence equal to `max(sequence of the lines that are not reward lines, default 10) + 1` for discounts and free products, and `max(sequence of the lines that are not reward lines, default 0) + 1` for free shipping, so that reward lines sort after the ordinary lines;
- the name of the reward description, possibly suffixed as described below.

When a line already exists from a previous application of the same reward and the new values name the same product, the **existing line description is preserved**, so a description edited by a salesperson survives a re-evaluation.

### 9.2 Discount rewards

Input: a discount reward `W`, a card `C`.

1. Obtain `discountable` and `discountable_per_tax` from section 8.2, 8.3 or 8.4 according to `W.discount_applicability`.
2. When `discountable` is zero or absent:
   - when `W`'s program is not a payment program and at least one line of the document carries a payment reward, produce a single **placeholder line**: description `TEMPORARY DISCOUNT LINE`, unit price zero, quantity zero, point cost zero. The placeholder keeps the reward attached to the document so that it can come back if the payment reward is removed.
   - otherwise refuse with `There is nothing to discount`.
3. Compute the ceiling:
   ```formula
   max_discount = convert(W.discount_max_amount, program currency, document currency)
   if W.discount_max_amount = 0 then max_discount = +infinity
   max_discount = min(max_discount, document.total_including_tax)
   ```
4. Apply the mode:
   - `per_point`: let `points` be the points available on `C` (section 5); when `W`'s program is not a payment program, floor them to a whole multiple of `W.required_points` with `points = floor(points ÷ W.required_points) × W.required_points`, because a reward may not be granted partially; then `max_discount = min(max_discount, convert(W.discount × points, program currency, document currency))`.
   - `per_order`: `max_discount = min(max_discount, convert(W.discount, program currency, document currency))`.
   - `percent`: `max_discount = min(max_discount, discountable × W.discount ÷ 100)`.
5. Compute the point cost:
   ```formula
   if W.clear_wallet then point_cost = points available on C
   else point_cost = W.required_points
   if W.discount_mode = "per_point" and not W.clear_wallet then
       converted = convert(min(max_discount, discountable), document currency, program currency)
       point_cost = round_currency(converted ÷ W.discount, C.currency)
   ```
6. **Payment program branch**: produce exactly **one** line with `unit price = −min(max_discount, discountable)` and the point cost above.
   - For a gift card program only, the line then adopts the taxes of the discount product: map the discount product's company taxes through the document's fiscal position; when there are any, recompute the price treating it as tax-included and without rounding, then set the unit price to the tax-excluded base plus the amounts of the tax-included taxes, and set the line's taxes to the mapped set. This makes the gift card reduce the tax-included total by exactly its face value whatever the taxes on it.
   - For an electronic wallet program the line carries no tax.
7. **Ordinary branch**:
   ```formula
   discount_factor = min(1, max_discount ÷ discountable)   (1 when discountable is zero)
   ```
   For every entry `(taxes, amount)` of `discountable_per_tax` whose `amount` is not zero, produce one line:
   - map `taxes` through the document's fiscal position into `mapped`;
   - `unit price = −(amount × discount_factor)`;
   - the line's taxes are `mapped`;
   - the description is `Discount <reward description>` when `mapped` is not empty, and the plain reward description otherwise; when the breakdown has **more than one** entry and at least one mapped tax has a name, the suffix ` - On products with the following taxes: <comma separated tax names>` is appended;
   - point cost zero.
   Then assign the computed `point_cost` to the **first** produced line only, so that the cost is counted once however many tax lines were produced.

### 9.3 Free product rewards

Input: a free product reward `W`, a card `C`, optionally a chosen product.

1. Let `eligible` be `W.reward_product_ids`. The product is the chosen one, or the first eligible one when none was chosen.
2. When there is no product, or the chosen product is not in `eligible`, refuse with `Invalid product to claim.`
3. Map the product's company taxes through the document's fiscal position.
4. Let `points` be the points available on `C`.
5. ```
   claimable_count = 1                                     if W.clear_wallet
   claimable_count = floor(points ÷ W.required_points)      otherwise
   point_cost = points                                      if W.clear_wallet
   point_cost = claimable_count × W.required_points          otherwise
   ```formula
6. Produce one line: the chosen product, quantity `W.reward_product_qty × claimable_count`, a discount percentage of 100, the mapped taxes and `point_cost`. The 100 percent discount is what makes the line free while keeping the product's list price visible on the document.

### 9.4 Free shipping rewards

Available only when the shipping capability package is present.

1. Let `shipping_line` be the first line of the document flagged as a shipping line.
2. Map the shipping product's company taxes through the document's fiscal position.
3. ```
   max_discount = W.discount_max_amount, or +infinity when it is zero
   unit price = −min(max_discount, shipping_line.unit_price, or 0 when there is no shipping line)
   point_cost = W.required_points, or the points available on C when W.clear_wallet
   ```
4. Produce one line with quantity 1, the mapped taxes, and the description `Free Shipping - <reward description>`.

Only one shipping reward may be applied at a time: when the document already carries one, every further shipping reward is removed from the claimable set.

Worked example. A reward `W` described `Free Shipping` with `required_points` 1 and `clear_wallet` false, on a card `C` holding 37.00 points, against an order whose shipping line has a unit price of 12.50:

- With `W.discount_max_amount` of 0, `max_discount` is `+infinity`, so the unit price of the reward line is `−min(+infinity, 12.50) = −12.50` and the shipping becomes free. The line has quantity 1, the shipping product's company taxes mapped through the order's fiscal position, the description `Free Shipping - Free Shipping`, and a point cost of 1.
- With `W.discount_max_amount` of 8.00, the unit price is `−min(8.00, 12.50) = −8.00`, and the customer still pays `12.50 − 8.00 = 4.50` of shipping.
- On an order that carries no shipping line at all, the shipping unit price reads as 0, so the unit price is `−min(8.00, 0) = 0.00` and a reward line of 0.00 is produced.
- With `W.clear_wallet` true, the point cost is not 1 but the whole balance of `C`, that is 37.00, whatever the amount discounted.

## 10. Free quantity at a counter

The counter application must decide how many free units a ticket has earned **without** letting the free units feed back into the point count that earned them. Two computations are used.

### 10.1 The buy-some-take-some distribution

Given a number of items `n`, a rule "buy `b` take `t`":

```formula
factor      = truncate(n ÷ (b + t))
free        = factor × t
charged     = n − free
x           = (factor + 1) × b
y           = x + (factor + 1) × t
adjustment  = charged − x        when x ≤ charged < y
adjustment  = 0                  otherwise
result      = floor(free + adjustment)
```

Worked example, buy 2 take 1: for `n = 7`, `factor = truncate(7 ÷ 3) = 2`, `free = 2`, `charged = 5`, `x = 6`, `y = 9`; `x ≤ charged` is false so `adjustment = 0`; result 2. For `n = 9`, `factor = 3`, `free = 3`, `charged = 6`, `x = 8`, `y = 12`; `8 ≤ 6` is false, adjustment 0; result 3. The full table for buy 2 take 1 over `n` from 1 to 10 is free = 0, 0, 1, 1, 1, 2, 2, 2, 3, 3; for buy 2 take 3 it is 0, 0, 1, 2, 3, 3, 3, 4, 5, 6.

### 10.2 Unclaimed free quantity

Inputs: a reward `W`, a card, a product `P`, the remaining points. The four steps are performed strictly in this order; step 2 and step 3 are the two exclusive branches of one decision, and step 4 always runs.

1. Walk the ticket lines and accumulate:
   - `available`: the quantity of lines whose product is an eligible reward product. When no reward line exists yet, only lines whose product is exactly `P` count; once reward lines exist, every eligible product counts.
   - `claimed`: the quantity of reward lines of this same reward; their point cost is added back to the remaining points.
   - a flag `needs_correction` when a reward line of a **different** reward gives out one of the eligible products.
2. **The plain branch.** When the program's `trigger` is `with_code`, or when `P` is not part of any rule of the program, or when the program's `applies_on` is `future`:
   ```formula
   free_quantity = floor((remaining_points ÷ W.required_points) × W.reward_product_qty)
   ```
   The floor is taken after the multiplication, not before it, so a reward that gives 3 units for 2 points yields 4 units for 3 points and not 3.
3. **The point-factor branch.** Otherwise, compute the point factor of `P` over the rules that were actually counted for the program (or over every rule when that information is not available): for each such rule whose valid products contain `P` or which has no product filter, add `rule.reward_point_amount` to `order_points` when its mode is `order`, add `round(rule.reward_point_amount × P.list_price, product price precision)` to `factor` when its mode is `money`, and add `rule.reward_point_amount` to `factor` when its mode is `unit`.
   - 3a. When `factor` is zero: `free_quantity = floor((remaining_points ÷ W.required_points) × W.reward_product_qty)`, exactly as in step 2.
   - 3b. Otherwise, let `correction` be the point correction of section 10.3 when `needs_correction` is set and zero otherwise, and
     ```formula
     free_quantity = buy_some_take_some((remaining_points − correction − order_points) ÷ factor,
                                        W.required_points ÷ factor,
                                        W.reward_product_qty)
                     + floor((order_points ÷ W.required_points) × W.reward_product_qty)
     ```
     Dividing by `factor` is what converts a point budget into a **number of items**: `factor` is the number of points one unit of `P` earns, so `remaining_points ÷ factor` is how many units of `P` those points represent, and `W.required_points ÷ factor` is how many units of `P` must be paid for to earn one application. Both arguments may be fractional; `buy_some_take_some` is defined for fractional arguments and its floor at the end absorbs the fraction.
4. Return `min(available, free_quantity) − claimed`. The result may be negative, which means the ticket now carries more free units than it is entitled to and the caller must reduce the reward line.

#### Worked example 1: points per unit, whole arguments

A promotion with `trigger` `auto` and `applies_on` `current`. Its single rule grants `reward_point_amount` 1 in mode `unit` on product Alpha. Its single reward `W` gives `reward_product_qty` 1 unit of Alpha for `required_points` 2. The ticket carries 7 units of Alpha and no reward line yet, so the card holds 7 points.

1. No reward line exists, so only the lines whose product is exactly Alpha count: `available = 7`. `claimed = 0`, `needs_correction` is not set, `remaining_points = 7`.
2. The trigger is `auto`, Alpha is part of the rule and `applies_on` is `current`, so step 2 does not apply.
3. The only rule is in mode `unit`, so `factor = 1` and `order_points = 0`. `factor` is not zero, and `needs_correction` is not set, so `correction = 0`:
   ```formula
   free_quantity = buy_some_take_some((7 − 0 − 0) ÷ 1, 2 ÷ 1, 1) + floor((0 ÷ 2) × 1)
                 = buy_some_take_some(7, 2, 1) + 0
   ```
   With `n = 7`, `b = 2`, `t = 1`: `factor = truncate(7 ÷ 3) = 2`, `free = 2`, `charged = 5`, `x = 3 × 2 = 6`, `y = 6 + 3 × 1 = 9`; `6 ≤ 5` is false so `adjustment = 0`; the result is 2. So `free_quantity = 2`.
4. Return `min(7, 2) − 0 = 2`. Two units of Alpha are given free, and the ticket therefore charges 5 of the 7 units.

#### Worked example 2: points per unit of currency, fractional arguments

The same program shape, but the rule grants `reward_point_amount` 1 in mode `money` on any product, and `W` gives 1 unit of product Beta for `required_points` 10. Beta's list price is 3.00 and the product price precision is two decimal places. The ticket carries 8 units of Beta at 3.00, that is 24.00, so the card holds 24 points.

1. `available = 8`, `claimed = 0`, `needs_correction` not set, `remaining_points = 24`.
2. Does not apply, for the same reasons as before.
3. The rule is in mode `money`, so `factor = round(1 × 3.00, 2) = 3.00` and `order_points = 0`:
   ```formula
   free_quantity = buy_some_take_some((24 − 0 − 0) ÷ 3.00, 10 ÷ 3.00, 1) + floor((0 ÷ 10) × 1)
                 = buy_some_take_some(8, 3.3333…, 1)
   ```
   With `n = 8`, `b = 3.3333…`, `t = 1`: `factor = truncate(8 ÷ 4.3333…) = truncate(1.846…) = 1`, `free = 1 × 1 = 1`, `charged = 8 − 1 = 7`, `x = 2 × 3.3333… = 6.6667`, `y = 6.6667 + 2 × 1 = 8.6667`; `6.6667 ≤ 7 < 8.6667` is true, so `adjustment = 7 − 6.6667 = 0.3333`; the result is `floor(1 + 0.3333) = 1`. So `free_quantity = 1`.
4. Return `min(8, 1) − 0 = 1`. One unit of Beta is given free: the 24.00 spent buys one application worth 10 points and is 6.67 points short of the second one, which the floor discards.

#### Worked example 3: a rule that grants points per order as well

The same program, with two rules on Alpha: rule A grants `reward_point_amount` 2 in mode `unit`, rule B grants `reward_point_amount` 10 in mode `order`. `W` gives 1 unit of Alpha for `required_points` 4. The ticket carries 6 units of Alpha, so the card holds `6 × 2 + 10 = 22` points.

1. `available = 6`, `claimed = 0`, `needs_correction` not set, `remaining_points = 22`.
2. Does not apply.
3. Rule A contributes to the factor and rule B to the order points: `factor = 2`, `order_points = 10`:
   ```formula
   free_quantity = buy_some_take_some((22 − 0 − 10) ÷ 2, 4 ÷ 2, 1) + floor((10 ÷ 4) × 1)
                 = buy_some_take_some(6, 2, 1) + floor(2.5)
   ```
   With `n = 6`, `b = 2`, `t = 1`: `factor = truncate(6 ÷ 3) = 2`, `free = 2`, `charged = 4`, `x = 3 × 2 = 6`, `y = 6 + 3 = 9`; `6 ≤ 4` is false so `adjustment = 0`; the result is 2. And `floor(2.5) = 2`. So `free_quantity = 2 + 2 = 4`.
4. Return `min(6, 4) − 0 = 4`. The per-order points are handled outside the buy-some-take-some distribution because they are earned once for the whole ticket and are not tied to any number of units.

#### Worked example 4: the plain branch, and the second pass

Take worked example 1 again but with `W.required_points` 2 and `W.reward_product_qty` 3, and a program whose `trigger` is `with_code`, so step 2 is taken. With `remaining_points = 7`: `free_quantity = floor((7 ÷ 2) × 3) = floor(10.5) = 10`, and the return value is `min(available, 10) − claimed`, which is capped by what the ticket actually contains.

Now the second pass of worked example 1, run after the two free units have been materialized as one reward line of quantity 2 at a point cost of 4. Reward lines now exist, so every eligible product counts: `available = 7` from the Alpha line. The reward line belongs to `W`, so `claimed = 2` and its point cost of 4 is added back to the remaining points, which were 3 after the claim, giving `remaining_points = 7` again. Step 3 repeats the arithmetic of the first pass and yields `free_quantity = 2`. Step 4 returns `min(7, 2) − 2 = 0`: nothing further is unclaimed, which is what makes the evaluation idempotent.

### 10.3 The point correction

Free product lines are priced at the negative of the product price, so a rule that grants points per unit of currency spent would count them as a payment. The correction removes their contribution.

For every rule of the program and every reward line of the ticket whose reward is a free product reward, whose product belongs to the rule (or the rule has no product filter), and whose reward belongs to the same program as the rule:

- when the rule's mode is `money`: subtract `round(rule.reward_point_amount × line.tax_included_total, product price precision)` from the correction;
- when the rule's mode is `unit`: add `rule.reward_point_amount × line.quantity` to the correction;
- when the rule's mode is `order`: contribute nothing.

The correction is zero when the ticket does not meet the program's own minimum amount and minimum quantity conditions. It is also zero when `needs_correction` was not set in step 1 of section 10.2, because that step only asks for it when a reward line of a **different** reward gives out one of the eligible products.

Worked example. A program carries two rules, neither with a product filter: rule A grants `reward_point_amount` 2 in mode `unit`, rule B grants `reward_point_amount` 1 in mode `money`. The product price precision is two decimal places. The ticket already carries one free product reward line of this program: quantity 2 of a product whose price is 20.00, written as a line on the hidden discount product at a unit price of −20.00, so its tax-included total is −40.00, and its reward product is that product.

- Rule A is in mode `unit`, the line's reward is a free product reward, the line's reward product belongs to rule A (which has no product filter), and the reward belongs to the same program as the rule, so the line contributes `+2 × 2 = +4.00` to the correction.
- Rule B is in mode `money`, and the same three conditions hold, so the line contributes `−round(1 × (−40.00), 2) = −(−40.00) = +40.00` to the correction.
- A third rule in mode `order` would contribute nothing.

The correction is `4.00 + 40.00 = 44.00`. Step 3b of section 10.2 then evaluates `remaining_points − 44.00 − order_points` before dividing by the factor. If instead the ticket carried the same line but the reward line belonged to the very reward being evaluated, `needs_correction` would not be set and the correction used would be 0.00, however large the figure above.

## 11. Order-level aggregates

### 11.1 Reward total

```formula
reward_total = Σ over lines carrying a reward:
                 + line.tax_excluded_subtotal              when the reward is not a free product
                 − line.product_list_price × line.quantity  when the reward is a free product
```

Discount reward lines carry negative subtotals, so a document with a 30.00 discount and a free product of list price 25.00 has a reward total of `−30.00 − 25.00 = −55.00`.

### 11.2 Loyalty summary of a confirmed order

For a confirmed order that has history movements, the summary holds:

- `point_name`: the `point_name` of the single card involved when exactly one card is involved, and the literal `Points` otherwise;
- `issued`: the sum of the `issued` of the order's history movements;
- `cost`: the sum of the `used` of the order's history movements.

### 11.3 Amount excluding shipping

When the shipping capability package is present, the amount used to decide whether shipping is free above a threshold excludes the payment reward lines:

```formula
amount_total_without_shipping = base amount without shipping
                                − Σ unit prices of the lines whose card belongs to
                                   a gift card or electronic wallet program
```

Since those unit prices are negative, the subtraction adds their absolute value back: paying part of an order with a gift card must not cost the customer the free shipping they had earned.

Worked example. A shipping method is free above 100.00. An order carries product lines totalling 120.00 including tax and a gift card payment reward line of unit price −50.00 whose card belongs to a gift card program.

- The base amount without shipping is `120.00 − 50.00 = 70.00`.
- The sum of the unit prices of the payment reward lines is −50.00.
- `amount_total_without_shipping = 70.00 − (−50.00) = 120.00`.
- `120.00 ≥ 100.00`, so the shipping stays free. Without the correction the comparison would have used 70.00, which is below the threshold, and the shopper would have been charged for shipping because they paid with their own gift card.

Two further cases on the same threshold of 100.00:

- An order of 110.00 paid in part by an electronic wallet line of −30.00: the base amount without shipping is 80.00, the sum of the payment unit prices is −30.00, and `80.00 − (−30.00) = 110.00`, which is above the threshold, so the shipping stays free.
- An order of 110.00 carrying an ordinary promotional discount reward line of −22.00: that line's card belongs to neither a gift card nor an electronic wallet program, so it is not added back. The amount is `110.00 − 22.00 = 88.00`, which is below the threshold, and the shipping is charged. A price reduction genuinely lowers what the customer spends, whereas a gift card only changes how they pay for it.

## 12. Worked example A: a ten percent order discount over two tax rates

Setup:

| Line | Product | Quantity | Unit price | Tax |
|---|---|---|---|---|
| 1 | Product Alpha | 4 | 100.00 | 15 percent, tax excluded |
| 2 | Product Beta | 3 | 100.00 | 10 percent, tax included |

Document totals before the reward:

| | Tax excluded | Tax | Tax included |
|---|---|---|---|
| Line 1 | 400.00 | 60.00 | 460.00 |
| Line 2 | 272.73 | 27.27 | 300.00 |
| **Total** | **672.73** | **87.27** | **760.00** |

A promotion offers 10 percent on the order, applicability `order`, mode `percent`, one point per order, one point required.

1. Points granted: mode `order`, so `points = 1`. The card created for the order holds the promise of 1 point.
2. Points available: `0 + 1 − 0 = 1`, which is greater than or equal to the required 1, so the reward is claimable.
3. Discountable amount, order applicability:
   - Line 1: raw base 400.00, raw tax 60.00, no tax-included tax. `discountable += 460.00`; `discountable_per_tax[15 percent excluded] += 400.00`.
   - Line 2: raw base 272.73, raw tax 27.27, the tax is tax-included. `discountable += 300.00`; `discountable_per_tax[10 percent included] += 272.73 + 27.27 = 300.00`.
   - `discountable = 760.00`.
4. Ceiling: no maximum, so `max_discount = min(+infinity, 760.00) = 760.00`; mode `percent` gives `max_discount = min(760.00, 760.00 × 0.10) = 76.00`.
5. `discount_factor = min(1, 76.00 ÷ 760.00) = 0.10`.
6. Point cost: mode is not `per_point`, `clear_wallet` is false, so `point_cost = 1`, assigned to the first produced line.
7. Two reward lines are produced, because the breakdown has two entries and at least one mapped tax has a name:

| Reward line | Description | Quantity | Unit price | Taxes | Tax excluded | Tax | Tax included | Point cost |
|---|---|---|---|---|---|---|---|---|
| A | `Discount 10% on your order - On products with the following taxes: 15 percent excluded` | 1 | −40.00 | 15 percent excluded | −40.00 | −6.00 | −46.00 | 1 |
| B | `Discount 10% on your order - On products with the following taxes: 10 percent included` | 1 | −30.00 | 10 percent included | −27.27 | −2.73 | −30.00 | 0 |

8. Document totals after the reward: tax excluded `672.73 − 40.00 − 27.27 = 605.46`; tax `87.27 − 6.00 − 2.73 = 78.54`; tax included `760.00 − 76.00 = 684.00`. The discount is exactly ten percent of the tax-included total and both tax amounts remain exactly ten percent lighter than before.

Variant with a ceiling: set `discount_max_amount` to 50.00. Then `max_discount = min(+infinity → 50.00, 760.00) = 50.00`, then `min(50.00, 76.00) = 50.00`, and `discount_factor = 50.00 ÷ 760.00 = 0.0657894736…`. Line A carries `−(400.00 × 0.0657894736…) = −26.32` and line B carries `−(300.00 × 0.0657894736…) = −19.74`. The tax-included reduction is `26.32 × 1.15 + 19.74 = 30.27 + 19.74 = 50.01`, which differs from the ceiling by one cent because each line is rounded independently; the ceiling is a ceiling on the amount the reward distributes, not on the resulting tax-included reduction.

## 13. Worked example B: buy three, get one free

Setup: a promotion of type `promotion`, `applies_on` `current`, `trigger` `auto`. One rule: products limited to Large Cabinet, `reward_point_mode` `unit`, `reward_point_amount` 1, `minimum_qty` 3. One reward: type `product`, reward product Large Cabinet, `reward_product_qty` 1, `required_points` 3. Large Cabinet is priced 320.00 with no tax.

| Cabinets ordered | Quantity gate | Points granted | `claimable_count` | Free units | Point cost | Order total |
|---|---|---|---|---|---|---|
| 2 | `2 < 3`, gate fails, rule skipped | 0 (no rule granted) | not applicable | 0, the reward line is removed | not applicable | 640.00 |
| 3 | passes | `1 × 3 = 3` | `floor(3 ÷ 3) = 1` | `1 × 1 = 1` | 3 | 960.00 |
| 6 | passes | `1 × 6 = 6` | `floor(6 ÷ 3) = 2` | 2 | 6 | 1 920.00 |
| 75 | passes | `1 × 75 = 75` | `floor(75 ÷ 3) = 25` | 25 | 75 | 24 000.00 |

The free units never inflate the point count: the counting lines of section 4.1 exclude reward lines, so a document with 75 paid cabinets and 25 free ones still grants exactly 75 points. Lowering the paid quantity from 75 back to 6 immediately recomputes 6 points and 2 free units, because the whole computation is re-run from the current line set rather than adjusted incrementally.

The free line itself is a normal product line: product Large Cabinet, quantity 25, unit price 320.00, discount percentage 100, tax-excluded subtotal 0.00.

## 14. Worked example C: one point per ten units of currency, five units of currency for one hundred points

Setup: a program of type `loyalty`, `applies_on` `both`, `trigger` `auto`, point name `Loyalty point(s)`, program currency with two decimal places. One rule: no product filter, `reward_point_mode` `money`, `reward_point_amount` 0.1 (one point for every ten units of currency spent), `minimum_qty` 1, `minimum_amount` 0. One reward: type `discount`, `discount_mode` `per_order`, `discount` 5 (five units of currency), `discount_applicability` `order`, `required_points` 100.

The customer already owns a card of this program holding 90.00 points. The customer orders 250.00 worth of goods, no tax.

1. The order is evaluated. The program is nominative (`applies_on` is `both`), so the customer's card is automatically attached to the order.
2. Points granted: mode `money`, `amount_paid = 250.00`, `points = round_down(0.1 × 250.00, 2) = 25.00`. A pending promise of 25.00 points is recorded between the order and the card.
3. Points available: `90.00 + 25.00 − 0 = 115.00`, rounded by the program currency to `115.00`.
4. Claimable rewards: `115.00 ≥ 100`, so the discount is claimable.
5. The reward is claimed. Discountable amount, order applicability, one tax group (the empty tax set): `discountable = 250.00`, `discountable_per_tax[no tax] = 250.00`.
6. Ceiling: `max_discount = min(+infinity, 250.00) = 250.00`; mode `per_order` gives `max_discount = min(250.00, 5.00) = 5.00`.
7. `discount_factor = min(1, 5.00 ÷ 250.00) = 0.02`.
8. Point cost: mode is not `per_point`, so `point_cost = 100.00`.
9. One reward line: the description derived by section 3.5 of [entities.md](entities.md) is the formatted amount followed by ` on your order`, which for a program currency whose symbol is placed after the amount reads `5 <symbol> on your order`; quantity 1, unit price `−(250.00 × 0.02) = −5.00`, no tax, point cost 100.00.
10. Order total: 245.00.
11. Points available now: `90.00 + 25.00 − 100.00 = 15.00`, so the reward is no longer claimable a second time.
12. On confirmation, the net change for the card is `+25.00 − 100.00 = −75.00` and the balance becomes 15.00. One history movement is written on the card: `issued` 25.00, `used` 100.00, description `Order S00042`, referencing the order.
13. Cancelling the order afterwards deletes that history movement and adds 75.00 back, restoring the balance to 90.00.

## 15. Worked example D: a gift card of fifty spent on an order of eighty

Setup: a program of type `gift_card`, `applies_on` `future`, `trigger` `auto`, portal visible, point name the currency symbol. One rule: products limited to the gift card product, `reward_point_mode` `money`, `reward_point_amount` 1, `reward_point_split` true, `minimum_qty` 0. One reward: type `discount`, `discount_mode` `per_point`, `discount` 1, `discount_applicability` `order`, `required_points` 1. The reward's hidden discount product carries no tax.

**Part one: buying the card.** A customer orders one gift card product priced 50.00.

1. Points granted: the split branch applies (`applies_on` is `future`, `reward_point_split` is true, mode is `money`). For the gift card line: `points_per_unit = round_down(1 × 50.00 ÷ 1, 2) = 50.00`; the quantity is 1, so `split_points = [50.00]`. The aggregate part is `0`, so the result is `[0, 50.00]`.
2. Two point entries are created: one carrying 0 points and one carrying 50.00 points, each with its own new card. The zero entry belongs to the aggregate slot and produces a card that will be removed at confirmation because it claims nothing.
3. On confirmation, the card receives 50.00 points, its code is generated, and the "at creation" communication sends the gift card email with the printed gift card attached.

**Part two: spending the card.** A different order carries 80.00 of goods with no tax.

1. The customer enters the gift card code. The code resolves to a card, the card's program is applicable, the card is not expired, and its balance 50.00 is at least the smallest required points of the program (1), so the card is attached to the order.
2. Points available on the card for this order: the program's `applies_on` is `future`, so the pending promise of this order is **not** added; the card holds 50.00 and no line consumes it yet, so 50.00 points are available. The program has a rule in `money` mode, so the value is rounded by the card currency: 50.00.
3. Claimable rewards: `50.00 ≥ 1`, the reward is claimable.
4. Discountable amount, order applicability, payment program branch: the lines are every non-display line minus the lines whose product is the gift card trigger product (there are none here); fixed-amount taxes are **not** excluded. `discountable = 80.00`.
5. Ceiling: `max_discount = min(+infinity, 80.00) = 80.00`. Mode `per_point` with a payment program: the points are **not** floored to a multiple of `required_points`; `max_discount = min(80.00, 1 × 50.00) = 50.00`.
6. Point cost: mode `per_point` and not `clear_wallet`, so `converted = convert(min(50.00, 80.00)) = 50.00` and `point_cost = round_currency(50.00 ÷ 1, card currency) = 50.00`.
7. One reward line: description `Gift Card`, quantity 1, unit price `−min(50.00, 80.00) = −50.00`, no tax (the discount product carries none), point cost 50.00.
8. Order total: 30.00. On confirmation the card's balance becomes `50.00 − 50.00 = 0.00` and a history movement records `used` 50.00.

**Variant: the card is worth more than the order.** With an order of 30.00 and a card of 50.00: `discountable = 30.00`; `max_discount = min(30.00, 50.00) = 30.00`; `point_cost = round_currency(30.00 ÷ 1) = 30.00`; the reward line carries `−30.00`; the order total is 0.00 and the card keeps 20.00 for a later purchase.

**Variant: the gift card product carries a tax-included tax.** When the reward's hidden discount product is given a 10 percent tax-included tax, the payment line adopts it. The granted amount `−50.00` is recomputed with every tax forced to behave as tax-included and without rounding: the tax-excluded base is `−45.45` and the tax is `−4.55`; the tax on the product is natively tax-included, so its amount is added back and the unit price stays `−50.00`. The line then carries a tax-excluded base of `−45.45`, a tax of `−4.55` and a tax-included total of `−50.00`. The customer's tax-included total drops by exactly 50.00.

**Variant: the gift card product carries a tax-excluded tax.** With a 15 percent tax-excluded tax on the discount product and a granted amount of `−100.00`: the recomputation forces the tax to behave as tax-included, giving a tax-excluded base of `−86.96` and a tax of `−13.04`; the tax is **not** natively tax-included, so nothing is added back and the unit price becomes `−86.96`. The line then carries a tax-excluded base of `−86.96`, a tax of `−13.04` and a tax-included total of `−100.00`. Again the customer's tax-included total drops by exactly the face value spent.

**Variant: a fixed-amount tax on the goods.** An order carries a product priced 90.00 with a flat tax of 10.00 per unit, total 100.00. A gift card of 100.00 is applied. Because the program is a payment program, the fixed tax is **not** excluded from the discountable amount: `discountable = 100.00`, `max_discount = min(100.00, 100.00) = 100.00`, the reward line carries `−100.00` and the order total is exactly 0.00. Had the same amount come from an ordinary percentage discount, the fixed tax would have been protected and the order could not have reached zero.

## 16. Ordering of reward applications

When a document is re-evaluated, the rewards already applied are re-applied in a fixed order so that the amounts are reproducible:

1. Every reward that does not belong to a payment program, in the order in which its first line appears on the document.
2. Every reward that belongs to a payment program (gift cards and electronic wallets), in the order in which its first line appears.

Payment rewards are always last because their amount depends on the total after every ordinary discount has been taken. A gift card of 50.00 on an order of 100.00 that also carries a 10 percent discount therefore pays against 90.00, not against 100.00, and the order total becomes 40.00.

## 17. Automatic invoicing of a fully rewarded order

When an order is confirmed, its total including tax is zero and its reward total is not zero, and the deployment enables automatic invoicing, an invoice is produced anyway: the lines are forced to the "ordered quantities" invoicing policy, an invoice is created and posted, and, when it is ready to be sent, it is marked as sent and dispatched with the configured invoice email template. Without this rule a fully discounted order would never produce an accounting document.
