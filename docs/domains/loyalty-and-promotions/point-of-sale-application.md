# The point-of-sale application

At a counter the whole evaluation runs on the cashier's device, not on the server. The device holds a copy of the programs, the rules and the rewards, evaluates them on every change of the ticket, and only talks to the server twice: once to validate the point movements just before payment, and once to create the cards and write the history after the ticket has been pushed. This file is the complete contract that a replacement device application and a replacement server must both honor.

## 1. Data loaded onto the device

Opening a counter session loads four loyalty entities in addition to the ordinary counter data.

### 1.1 The programs available at a counter

A program is available at a counter when all of the following hold, evaluated with today's date in the counter's time zone:

```
pos_ok = true
AND (pos_config_ids CONTAINS this counter OR pos_config_ids IS EMPTY)
AND (date_from IS EMPTY OR date_from <= today)
AND (date_to IS EMPTY OR date_to >= today)
AND (pricelists IS EMPTY OR pricelists INTERSECTS the counter's available pricelists)
AND currency = the counter's currency
AND (limit_usage = false OR total_order_count < max_usage)
```

An empty counter restriction means every counter, which is the opposite of the usual convention and is deliberate: a program that names no counter is published everywhere.

### 1.2 Fields transferred

| Entity | Selection | Fields sent to the device |
|---|---|---|
| Loyalty Program | the programs of section 1.1 | name, trigger, applies on, program type, pricelists, start date, end date, usage limit flag, maximum usage, total document count, nominative flag, portal visibility, portal point name, trigger products, rules, rewards. Read with elevated rights so that a cashier who may not read a program's company still receives it. |
| Loyalty Rule | the rules of those programs | program, valid products, "any product" flag, currency, reward point amount, split flag, reward point mode, minimum quantity, minimum amount, minimum amount tax mode, mode, code. |
| Loyalty Reward | the rewards of those programs, excluding free product rewards whose product and whose tag products are all archived | description, program, reward type, required points, clear wallet flag, currency, discount value, discount mode, discount applicability, expanded discounted products, global discount flag, maximum discount, hidden discount product, reward product, multi-product flag, reward products, reward product quantity, reward product unit of measure, serialized discounted-product condition. |
| Loyalty Card | none at load time | owner, code, balance, formatted balance, program, expiration date, last write timestamp. Cards are fetched on demand. |

Two transformations are applied to the reward data before it is sent:

1. The serialized discounted-product condition is rewritten so that the device can evaluate it: every condition on a relational product field that uses a text-matching operator is replaced by an explicit list of matching identifiers resolved on the server. `matches text` becomes `is in the list`, `does not match text` becomes `is not in the list`.
2. The product fields named by any reward condition are added to the product fields transferred to the device, so that the condition can actually be evaluated there.

The valid products of a rule are additionally restricted, at the counter, to products flagged as available at a counter, and the set is computed once per distinct filter so that two identical rules share one computation.

The product catalogue sent to the device is extended with every hidden discount product, every reward product and every trigger product the cashier may read, and the shipped gift card product, the shipped wallet top-up product and the trigger products of electronic wallet programs are marked as specially displayed so that they appear in the counter's product list.

### 1.3 Local identifiers

Cards that do not exist on the server yet are given **negative** identifiers, allocated by a counter that starts at −1 and decreases. Every structure that refers to a card refers to it by identifier, so a local card behaves exactly like a stored one until the ticket is pushed, at which point the server returns the mapping from negative identifiers to real ones.

## 2. The loyalty state of a ticket

| Structure | Content |
|---|---|
| Point changes | A map from card identifier to a record holding the points the ticket grants to that card, the program, the card identifier, an optional barcode, the list of rules that were actually counted, and, for a gift card, the product sold, the expiration date, the typed code, the owner and a "manual" flag. |
| Activated code rules | The list of rule identifiers whose code the cashier has entered on this ticket. |
| Disabled rewards | The set of reward identifiers the cashier removed by hand; they are never re-claimed automatically on this ticket. |
| Code-activated cards | The list of cards the cashier attached by scanning or typing a code. |
| Invalid-cards flag | Set when the ticket is loaded, so that the next evaluation drops point changes whose card is no longer in the local store. |

The disabled rewards are serialized with the ticket so that they survive a reload; the point changes are serialized as well, and any point change whose program is no longer loaded is dropped when the ticket is restored.

### 2.1 Events that reset part of the state

| Event | Effect |
|---|---|
| The customer of the ticket changes | Every point change whose program is nominative is deleted, so the counting of loyalty and wallet points restarts for the new customer. |
| The pricelist of the ticket changes | Every point change whose program is restricted to pricelists that do not include the new one is deleted. |
| The cashier presses the reset action | The disabled rewards, the activated code rules, the point changes and the code-activated cards are all cleared and every reward line is deleted. |
| A ticket is selected in the ticket list | The rewards are re-evaluated. |
| The table of a restaurant order changes | The rewards are re-evaluated. |

## 3. Applicability on the device

A program is applicable to the ticket when:

1. its `trigger` is `auto` and it has at least one rule whose `mode` is `auto` or whose identifier is among the activated code rules; or its `trigger` is `with_code` and at least one of its rules is among the activated code rules;
2. it is not nominative, or the ticket has a customer;
3. its start date, taken at the beginning of that day, is not in the future;
4. its end date, taken at the end of that day, is not in the past;
5. its usage ceiling has not been reached;
6. its pricelist restriction is empty or contains the ticket's pricelist.

## 4. Point computation on the device

The device computation follows the same shape as the server computation of section 4 of [calculations.md](calculations.md), with three deliberate differences, which a replacement must reproduce because the two results are compared when the ticket is pushed.

| Aspect | Server | Device |
|---|---|---|
| Rounding in the `money` mode | Truncate downward to two decimals | Round half away from zero to the product price precision |
| Lines excluded | Lines with no effect on a threshold | Lines flagged as ignored for loyalty points: the trigger product line of a gift card or wallet program other than the program being evaluated, lines settling an invoice, lines settling a sales order |
| Free product lines | Excluded from the counting lines | Counted with a **negative** quantity, because a free product is materialized as a negative line on the hidden discount product |

### 4.1 The algorithm

1. Build the ticket lines, excluding the item lines of a combination product.
2. Build, per rule, the lines whose product matches the rule or whose rule has no product filter, skipping discount reward lines of automatic programs, skipping lines the ticket declares invalid for loyalty points, and skipping, for each program, the discount reward lines of that same program.
3. For each program and each of its rules:
   1. Skip the rule when its `mode` is `with_code` and it is not among the activated code rules.
   2. Compute the tax-included and the tax-excluded totals of that rule's lines (a combination line contributes the totals of its items). Compare `minimum_amount` with the tax-included total when the tax mode is `incl` and with the tax-excluded total otherwise; skip the rule when the minimum is greater.
   3. Walk the ticket lines again and accumulate, for the lines that match the rule and are not ignored for loyalty points:
      - the quantity per product, counting a free product reward line **negatively**;
      - the paid amount, as the tax-included line total (or the combination total);
      - the total matched quantity and a flag saying that at least one non-reward line matched.
      Reward lines of the same program, and reward lines of gift card and wallet programs, are skipped inside this accumulation.
   4. Skip the rule when it has a product filter and no non-reward line matched.
   5. Skip the rule when the total matched quantity is strictly less than `minimum_qty`. Negative quantities count, which is what makes a refund of a wallet top-up reduce the balance instead of increasing it.
   6. Record the rule as counted for that program.
   7. Grant the points, with the split branch and the aggregate branch of the server algorithm, using round-half-away-from-zero at the product price precision instead of truncation.
4. The result per program is `[{points}]` when the aggregate value is non-zero or the program type is `coupons`, an empty list otherwise, followed by one entry per split value. A split entry for a physical gift card additionally carries the typed barcode and the local card identifier.

### 4.2 Reconciling the result with the existing point changes

1. Pair the computed values with the existing point changes of the program, in order, stopping at the first change flagged "manual"; a manual change is never overwritten.
2. When there are fewer computed values than existing changes, or the program stopped being applicable, delete every point change of that program.
3. When there are more computed values than existing changes, count the values by the pair (points, barcode), subtract the ones already covered by an existing change, and create one point change per remaining value. Each new change needs a card:
   - for a nominative program, fetch the customer's card of that program from the local store, then from the server, and create a local card with a negative identifier and balance zero when the server has none;
   - for any other program, create a local card with a negative identifier, no code, balance zero and the ticket's customer.
   - For a gift card program the new change additionally records the product sold, an expiration date one year from now, the code typed on the line and the ticket's customer.
4. For a nominative program with no computed value and a customer on the ticket, a change of zero points is added anyway, so the customer's card is loaded and its rewards become visible.
5. Cards attached by code whose program has `applies_on` equal to `current` and which produced no point value at all are detached from the ticket.

## 5. Points available and claimable rewards on the device

Points available on a card for the ticket:

```
points = balance of the card known to the device
       + the point change of this ticket for that card, unless the program's applies_on is "future"
       − Σ point costs of the reward lines of this ticket that use that card
```

Claimable rewards, for an optional card filter, an optional program filter and an "automatic" flag:

1. Build the candidate pairs (program, card) from the point changes and from the code-activated cards, excluding cards whose point change is both manual and bound to an existing code.
2. Skip a program whose pricelist restriction does not contain the ticket's pricelist.
3. For a program whose `trigger` is `with_code`, require that every rule's minimum amount and minimum quantity are met by the ticket and that at least one line matches each rule's product filter; for such programs the rules act as conditions rather than as point grants.
4. For each reward of the program, skip it when:
   - the points available are less than `required_points`;
   - the program type is `coupons` and the reward is already on the ticket;
   - the automatic flag is set, the reward is a discount that does not belong to a payment program, and it is already on the ticket;
   - the automatic flag is set and the reward is among the disabled rewards;
   - the reward is a global discount whose discount value is less than or equal to the discount value of the global discount already applied;
   - the reward is a discount and the ticket total including tax is zero;
   - the reward is a free product reward and no product can be determined (for a multi-product reward in automatic mode the product must be the one on the selected line), or the unclaimed free quantity is zero or negative.
5. Return the surviving pairs with the reward, the potential quantity and the product.

## 6. Automatic claiming

After every recomputation the device claims, without asking, every claimable reward whose program has exactly one reward, whose program is not nominative, and which is either not a free product reward or has a determined product. It then refreshes the existing reward lines, and, when anything changed, recomputes the programs once more, because a reward changes the amounts that the point computation reads.

Refreshing the existing reward lines means: remember every applied reward with its card, its product, its price, its quantity, its cost and its grouping code; delete the lines; merge the remembered entries that describe the same reward, the same price and the same product by summing their quantities and costs; then re-apply them in the order free products, other rewards, payment rewards. An entry whose card is no longer known is dropped. An entry of a `coupons` program whose reward is already back on the ticket is dropped. When a reward offers exactly one product and it is the only reward of its program on the ticket, the remembered quantity is forgotten so that the maximum possible quantity is claimed again.

## 7. Building the reward lines on the device

### 7.1 Discount rewards

1. Choose the discountable computation from the applicability:
   - **order**: for every line with a quantity, add the tax-included total to the discountable and add the line's base price to the bucket keyed by the line's taxes. The key keeps the fixed-amount taxes for a payment program and drops them otherwise.
   - **cheapest**: find the line with the lowest price per unit among the lines that carry no reward, are not items of a combination, have a quantity and whose product is one of the reward's expanded discounted products. The discountable and the single bucket are that line's combination base price.
   - **specific**: the cascading computation of section 8.4 of [calculations.md](calculations.md), adapted to the device, including the clamping of a percentage discount by the reward's maximum discount while the remaining amounts are being reduced.
2. Clamp the discountable to the ticket total including tax. When it is zero, produce no line at all.
3. Apply the mode ceiling, exactly as on the server, with the same rule that a non-payment `per_point` reward first truncates the points to a whole multiple of `required_points`.
4. Compute the point cost, exactly as on the server.
5. **Payment programs** produce one line on the hidden discount product priced at the negative of the granted amount, with the discount product's taxes, computed in "total included" mode so that the tax breakdown of the negative line is exact, and carrying the extra tax details that the server needs to reproduce the same amounts.
6. **Other programs** produce one line per non-empty bucket, priced at the negative of the bucket amount clamped by the ticket total and multiplied by the distribution factor, carrying the bucket's taxes, with the point cost on the first line only and a shared grouping code.
7. An unknown applicability produces the message `Unknown discount type`.

### 7.2 Free product rewards

Unlike the sales application, a free product at a counter is **not** a hundred-percent-discounted product line. The product must already be in the basket, and the reward adds a **negative line on the hidden discount product** that cancels its price.

1. Determine the product: the one passed by the caller when it is among the reward's products, otherwise the first of them.
2. Compute the unclaimed free quantity (section 10.2 of [calculations.md](calculations.md)). When it is zero or negative, answer `There are not enough products in the basket to claim this reward.`
3. ```
   claimable_count = 1                                                   when clear_wallet
   claimable_count = min(ceiling(unclaimed ÷ reward_product_qty),
                         floor(points ÷ required_points))                otherwise
   cost            = points                                              when clear_wallet
   cost            = min(claimable_count × required_points, requested cost) otherwise
   free_quantity   = min(unclaimed,
                         reward_product_qty × claimable_count,
                         requested quantity)
   ```
4. Produce one line: the hidden discount product, quantity `free_quantity`, unit price the negative of the product's price for that quantity under the ticket's pricelist rounded by the ticket currency, the product's taxes, the chosen product remembered on the line, the point cost and a fresh grouping code.

### 7.3 Applying a reward

1. Refuse with `There are not enough points on the coupon to claim this reward.` when the points available are below `required_points`.
2. For a global discount, when another global discount is applied whose discount value is greater than or equal to the candidate's, refuse with `A better global discount is already applied.`; when it is smaller, delete its lines.
3. Build the lines. An empty result is refused with `The reward could not be applied.`; a textual result is the refusal message itself.
4. Create the lines on the ticket with a manual price type.

## 8. Codes at the counter

### 8.1 Entering or scanning a code

The cashier types a code in the code popup (placeholder `Gift card or Discount code`) or scans a barcode that the shipped coupon barcode rule recognizes. The code is trimmed and processed as follows.

1. Look for a loaded rule whose `mode` is `with_code` and whose code or whose barcode equals the text.
2. Ask the server whether the code is the code of a loyalty card that belongs to a customer. When it is, select that customer on the ticket, fetching the contact when it is not loaded, re-evaluate the rewards and stop. This is how a loyalty card doubles as a customer card.
3. **When a rule matched**:
   - refuse with `That promo code program is not yet valid.` when the ticket date is before the program's start date taken at the beginning of the day;
   - refuse with `That promo code program is expired.` when the ticket date is after the program's end date taken at the end of the day;
   - refuse with `That promo code program requires a specific pricelist.` when the program's pricelist restriction does not contain the ticket's pricelist;
   - refuse with `That promo code program has already been activated.` when the rule is already among the activated code rules;
   - otherwise add the rule to the activated code rules, re-evaluate the programs and compute the rewards claimable from that program.
4. **When no rule matched**:
   - refuse with `That coupon code has already been scanned and activated.` when a code-activated card already carries that code;
   - otherwise ask the server to redeem the code (section 8.2). On refusal, show the server's message.
   - On success, when the program is a gift card program and the card has no source document, ask the cashier `This gift card is not linked to any order. Do you really want to apply its reward?` under the title `Unpaid gift card`. A refusal answers `Unpaid gift card rejected.` and stops.
   - Create a local copy of the card with the identifier, code, program, owner, balance and formatted balance returned by the server, attach it to the ticket, re-evaluate the programs and compute the rewards claimable from that card.
5. When exactly one reward is claimable and it is not a multi-product free product reward, claim it immediately and refresh.
6. When no rule matched, the ticket is empty and a card was found, answer `<program name>: <code>` followed by a new line and `Balance: <formatted balance>`, so that scanning a gift card on an empty ticket simply reports its balance.

### 8.2 The server redemption service

Input: the counter, the code, the ticket creation timestamp, the customer and the pricelist.

1. Search among the cards of the counter's programs for one whose code matches and whose owner is empty or is the ticket's customer, or whose program type is `gift_card`, ordered by owner and then by balance descending, and take the first one.
2. Refuse with `This coupon is invalid (<code>).` when no card is found or its program is archived.
3. Refuse with `This coupon is expired (<code>).` when the card's expiration date is earlier than the date part of the ticket timestamp, or the program's end date is earlier than today, or the program's usage ceiling has been reached.
4. Refuse with `This coupon is not yet valid (<code>).` when the program's start date is later than today.
5. Refuse with `No reward can be claimed with this coupon.` when the program has no reward, or no reward whose `required_points` is at most the card's balance.
6. Refuse with `This coupon is not available with the current pricelist.` when the program's pricelist restriction does not contain the ticket's pricelist.
7. Refuse with `This programs requires a code to be applied.` when the program type is `promo_code`; such a program must be reached through its rule code, not through a card code.
8. Otherwise answer with the program identifier, the card identifier, the card owner, the balance, the formatted balance and whether the card has a source document.

## 9. Selling a physical gift card

A physical gift card is a pre-printed card whose code is not generated by the system.

1. The cashier adds the gift card product to the ticket and opens the gift card management dialog from the line.
2. The dialog asks for the printed code and the amount, and proposes an expiration date one year from now.
3. Each time the code field stops changing for half a second, the device asks the server for the status of that code:
   - the code is accepted when it does not exist at all, or when it exists, is not expired, has a positive balance, belongs to a gift card program, has no owner and has never been used on a document;
   - a refused code shows `Invalid Gift Card Code` with the message `This code seems to be invalid, please check the Gift Card code and try again.` and clears the field;
   - an accepted code that already exists fills the amount with the card's balance rounded by the counter currency, fills the expiration date from the card, and locks both fields, so that an existing card can only be topped up at its own value;
   - a communication failure shows `An error occurred while checking the gift card.`
4. Confirming with an empty code or an empty amount marks the field in error and does nothing.
5. Confirming with a code that the ticket already uses shows `Validation Error` with the message `A coupon/loyalty card must have a unique code.`
6. Otherwise the ticket line is rewritten: the quantity is reduced by one, or the line is removed when it was the last unit; a new line is added with the typed amount as its price and the typed code stored on it; and a point change is recorded for a new local card carrying the program, the amount, the code, the owner, the product and the expiration date, flagged "manual". An existing non-manual point change with the same amount, program and product is consumed first, so that the manual card replaces the automatic one instead of doubling it.
7. The quantity and the price of a line that carries a typed gift card code may not be changed afterwards: `You cannot change the quantity or price of a physical gift card.`

## 10. Electronic wallets at the counter

1. The wallet button lists the wallet rewards claimable on the ticket whose card is not expired.
2. When no wallet reward is claimable and the ticket total is not negative, the button answers `No valid eWallet found` with the message `Please select a customer and a valid eWallet.`
3. When the ticket total is **negative** (a refund) and at least one wallet program exists, the cashier is offered the wallet programs under the title `Refund with eWallet`; choosing one adds a top-up line of the wallet's trigger product priced at the absolute value of the negative total, which credits the refund to the customer's wallet.
4. When wallet rewards are claimable, the cashier picks one (title `Use eWallet to pay`, each entry labelled `<reward description> (<program name>)`) and it is applied; a refusal is shown in an error dialog titled `Error`.
5. Paying a ticket that carries a wallet top-up line without a customer asks `Customer needed` with the message `eWallet requires a customer to be selected` and opens the customer selection.
6. A negative quantity or a negative price may not be set on a gift card or wallet line: `You cannot set negative quantity or price to gift card or ewallet.`
7. Refunding a top-up line or a reward line of a gift card or wallet program is refused with the notification `Refunding a top up or reward product for an eWallet or gift card program is not allowed.`

## 11. Editing and removing rewards on the ticket

1. Deleting a reward line deletes every line sharing the same reward, card and grouping code.
2. Pressing the removal key on a reward line that was not claimed manually first asks `Deactivating reward` with the message `Are you sure you want to remove <reward description> from this order?` followed by a new line and ` You will still be able to claim it through the reward button.`, with the buttons `Yes` and `No`.
3. Removing a reward line adds its reward to the disabled rewards of the ticket and, when its card was attached by code, deletes the local card.
4. A reward line may not have its quantity or price edited; only removal is accepted.
5. Reward lines are always displayed after the ordinary lines and in italics; a gift card or wallet reward line shows the remaining balance of its card.
6. The last selected line of a ticket, used by the numeric pad, never points at a reward line.

## 12. The two server exchanges

### 12.1 Validation before payment

Just before the payment screen accepts the ticket, the device sends:

- a map from **positive** card identifier to the net points the ticket will move on that card (the granted points minus the costs of the reward lines using it);
- the list of new codes the ticket will create, that is the barcodes of point changes that carry a barcode and no existing card.

The server answers success, or failure with one of:

| Condition | Message | Extra payload |
|---|---|---|
| A card identifier no longer exists, or its program is archived | `Some coupons are invalid. The applied coupons have been updated. Please check the order.` | the list of invalid card identifiers, which the device deletes locally |
| A card's balance is smaller than the points the ticket wants to spend, compared at two decimal places | `There are not enough points for the coupon: <code>.` | the current balances of the cards, which the device writes into its local copies |
| One of the new codes already exists | `The following codes already exist in the database, perhaps they were already sold?` followed by a new line and the comma-separated colliding codes | none |

A failure shows the message under the title `Error validating rewards` and stops the validation. A communication failure is ignored: the check is a convenience, not a gate, and the authoritative check happens at confirmation.

### 12.2 Confirmation after the ticket is pushed

Once the ticket exists on the server, the device sends the card data: per card identifier, the net points, the points earned, the points spent, the program, the owner for a nominative program or a next-order coupon program, the expiration date (the program end date for anything but a loyalty program), the grouping codes of the reward lines that used it, and, for a new card, its code and barcode. Cards of programs whose `applies_on` is `current` that claimed no reward are dropped from the payload, because their points would be lost anyway.

The server then:

1. **Re-points nominative cards**: for every entry that names an owner, look for an existing card of the same program owned by that contact whose program type is `loyalty` or `ewallet`; when one exists, move the entry onto that card's identifier. This is what merges a locally created card with the customer's real one.
2. **Drops duplicates**: an entry whose program already has a history entry for this very ticket is dropped, so that a retried confirmation does not double the points.
3. **Applies gift card entries separately**: for every entry whose program is a gift card program, find the card by code or by identifier; when it exists:
   - when it has no owner and the ticket has a customer, set the owner and write a history entry `Assigning partner <customer name>` with `issued` equal to the current balance;
   - when it has never been used on a document, set its source ticket and write a history entry `Assigning order <ticket display name>` with `issued` equal to the current balance;
   - when the entry's points differ from the balance, add the entry's points to the balance and write a history entry `Onsite <ticket display name>` carrying the movement as `used` when it is negative and as `issued` when it is positive.
   The entry is then removed from the payload.
4. **Creates the remaining new cards** (negative identifiers with points or with reward grouping codes): program, owner (validated against the contact table, defaulting to the ticket's customer), code taken from the payload code, then the payload barcode, then a freshly generated code, balance zero, expiration date from the payload, and the source ticket. Creation runs with elevated rights and with the "at creation" communication suppressed.
5. **Applies the points**: every card in the payload receives its points, and every reward line whose grouping code appears in an entry is bound to that card.
6. **Sends the creation communications** for the newly created cards, this time with sending enabled.
7. **Writes the history**: one entry per card carrying `issued` equal to the points earned and `used` equal to the points spent, with the description `Onsite <ticket display name>` and a reference to the ticket. Entries with neither value, and entries for cards that no longer exist, are skipped.
8. **Answers** with:
   - the card updates for nominative programs: the old (possibly negative) identifier, the new identifier, the balance, the code, the program and the owner;
   - the new usage count of every program involved, so that the device's ceiling check stays exact;
   - for every new card of a program whose `applies_on` is `future` and whose type is neither `gift_card` nor `ewallet`, the program name, the expiration date and the code, to be printed on the receipt;
   - the printed documents to render, grouped by document definition, listing the cards to print.

The device then rewrites its local cards with the returned identifiers, rebinds the ticket lines, updates the usage counts, renders the printed documents and stores the new card information on the ticket so that the receipt shows it. When the payload is empty no exchange happens at all.

### 12.3 Attaching a printed gift card to the receipt email

When the receipt is emailed and the counter's gift card programs declare a printed document, every gift card created by that ticket is rendered into a portable document, attached to the message under the receipt's name, and stored against the ticket.

## 13. Offline behavior and code reservation

1. Every evaluation, every reward computation and every reward line is produced on the device. A counter that has lost its connection keeps selling, keeps applying automatic promotions, keeps claiming rewards from cards already loaded, and keeps issuing new cards with locally generated identifiers.
2. Codes typed for physical gift cards are held locally until confirmation. The uniqueness of a typed code is checked twice: locally against the ticket's own point changes, and on the server at validation time. A code that was sold on another counter in the meantime is caught by the validation exchange with the "codes already exist" message.
3. Cards fetched from the server for a nominative program are cached on the device and reused; a card for a customer who has none is created locally with a negative identifier and only becomes real at confirmation.
4. A card that no longer exists on the server when a queued ticket is finally pushed is simply skipped when the history is written.
5. Cards are kept in the device database as long as at least one unsynchronized ticket line references them.

## 14. Restaurant specifics

1. Every reward line produced on a restaurant ticket is assigned to the **last course** of that ticket, so that a discount or a free product travels with the course it belongs to and is not printed to the kitchen on its own.
2. Changing the table of an order re-evaluates the rewards, so that the rewards stay on the order when the cashier leaves a table and comes back.

## 15. Settling a sales order at the counter

1. While a sales order is being settled onto a ticket, both the program update and the reward update are suppressed. Without this, the lines copied from the order would be counted as new purchases.
2. Once the settlement finishes, the programs are updated once and the rewards are re-evaluated.
3. A ticket line that comes from a sales order never earns loyalty points: it is excluded from the counting lines and declared invalid for loyalty points.
4. A ticket line copied from a sales order line that carries a reward takes the ordered quantity as it is, instead of going through the ordinary quantity derivation, so that a discount agreed on the quotation is reproduced exactly.
5. The reward reference of a sales order line is part of the data transferred when a sales order is loaded onto a counter.

## 16. Failure of a reward condition on the device

When a reward's discounted-product condition cannot be evaluated on the device, the reward is dropped from the loaded data and the cashier sees the title `A reward could not be loaded` with the message `The reward "<description>" contain an error in its domain, your domain must be compatible with the point of sale client`, and a button to reload. This is why the condition is pre-translated on the server (section 1.2).

## 17. The ticket line payload

Five loyalty fields are added to the ticket line data exchanged between the device and the server, in both directions:

| Field | Meaning |
|---|---|
| reward line flag | Whether the line is part of a reward. |
| reward | The reward the line materializes. |
| card | The card whose points paid for it. |
| reward grouping code | Links the several lines produced by one reward application. |
| point cost | How many points the line consumes on the card. |

The device additionally keeps four purely local fields on a line, which are never stored on the server:

| Local field | Meaning |
|---|---|
| electronic wallet or gift card program | The payment program a top-up line belongs to, used to forbid merging two top-up lines of different programs, to forbid a negative quantity or price, and to exclude the line from the points of every other payment program. |
| gift card code | The code typed for a physical gift card, which locks the line's quantity and price. |
| gift card barcode | The barcode captured for a physical gift card, sent as the new code at confirmation. |
| rewarded product | For a free product reward line, the real product being given, since the line itself carries the hidden discount product. |

A line that carries a gift card or wallet program shows its price as a positive amount even on a refund ticket, so that a top-up line is never displayed as a negative sale.

## 18. Points display on the ticket and the customer receipt

For every point change of a program of type `loyalty`, the device exposes five numbers, each rounded to two decimal places:

```
balance = the balance of the card known to the device
won     = the points the ticket grants − the point correction of section 10.3
spent   = Σ point costs of the reward lines of this ticket that use that card
total   = balance + won − spent
name    = the program's point name when the program is portal visible, otherwise the literal "Points"
```

`total` is shown while the ticket is being built and `balance` once it is paid. The point correction removes the contribution of the free product lines, which would otherwise be counted as a payment by a rule that grants points per unit of currency spent.

In the customer list, a card of an electronic wallet program is rendered as `<program name>: <formatted monetary balance>`; a card of any other program is rendered as `<balance with two decimals> <point name>` when the program is portal visible, and as `<balance with two decimals> Points` otherwise.
