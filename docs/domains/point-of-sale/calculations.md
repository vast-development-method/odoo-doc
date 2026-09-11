# Point of Sale — Calculations

Every formula and algorithm of the domain, with its rounding rule, its precision, its
currency, its unit conversion, its date handling and at least one worked numeric example.

A central requirement runs through this whole file: **the browser and the server must
produce identical numbers**. The selling application computes prices, taxes, rounding and
totals while the cashier works; the server recomputes them when the order is transmitted
and again when the session is closed. Any divergence surfaces as an unbalanced closing
entry. Section 12 states the contract that makes the two agree.

---

## 1. Rounding primitives

### 1.1 Rounding to a precision with a method

All monetary rounding in this domain is rounding a number to a multiple of a **rounding
step**, using one of five **rounding methods**.

```formula
quotient = value ÷ step
```

```formula
rounded_quotient = apply_method( quotient )
```

```formula
result = rounded_quotient × step
```

| Method | Label | `apply_method` |
| --- | --- | --- |
| `HALF-UP` | Round half away from zero | The nearest whole number; a quotient exactly halfway is moved away from zero. |
| `HALF-DOWN` | Round half toward zero | The nearest whole number; a quotient exactly halfway is moved toward zero. |
| `HALF-EVEN` | Round half to even | The nearest whole number; a quotient exactly halfway is moved to the nearest even whole number. |
| `UP` | Round away from zero | The whole number of largest magnitude not smaller in magnitude than the quotient. |
| `DOWN` | Round toward zero | The whole number of smallest magnitude not larger in magnitude than the quotient. |

The implementation compensates for the finite precision of binary floating-point
arithmetic before applying the method, so that a value that is mathematically exactly
halfway is treated as halfway rather than as marginally above or below.

**Worked examples** with a step of 0.05:

| Value | HALF-UP | HALF-DOWN | UP | DOWN |
| --- | --- | --- | --- | --- |
| 12.13 | 12.15 | 12.15 | 12.15 | 12.10 |
| 12.12 | 12.10 | 12.10 | 12.15 | 12.10 |
| 12.125 | 12.15 | 12.10 | 12.15 | 12.10 |
| −12.125 | −12.15 | −12.10 | −12.15 | −12.10 |

### 1.2 Symmetric and asymmetric rounding

Two rounding behaviors coexist.

**Symmetric rounding** applies the configured method unchanged to negative numbers, so
that rounding a value and rounding its negation give results that are negations of each
other.

**Asymmetric rounding** inverts the method for negative numbers:

| Configured method | Method used for a negative value |
| --- | --- |
| `UP` | `DOWN` |
| `DOWN` | `UP` |
| `HALF-UP` | `HALF-DOWN` |
| `HALF-DOWN` | `HALF-UP` |
| `HALF-EVEN` | `HALF-EVEN` |

Asymmetric rounding is used when rounding the remaining amount due and the change of an
order, so that a refund is rounded in the direction that favours the same party as the
equivalent sale.

**Worked example.** Step 0.05, method `UP`. Symmetric rounding of −12.12 gives −12.15
(away from zero). Asymmetric rounding of −12.12 uses `DOWN` and gives −12.10 (toward
zero) — the mirror of rounding +12.12 *up* to 12.15 would have been −12.15, but the
asymmetric rule keeps the customer from being charged more on a return.

### 1.3 Currency rounding

Every currency carries a rounding step (its smallest representable increment, for example
0.01) and a number of decimal places. Rounding "to the currency" means rounding to that
step with the `HALF-UP` method.

```formula
round_to_currency( value ) = round( value , step = currency_rounding_step , method = HALF-UP )
```

### 1.4 Comparison and zero test

Comparison is done on rounded values, not on raw ones.

```formula
compare( a , b ) :
    a' = round( a , step , method )
    b' = round( b , step , method )
    d  = a' − b'
    if d = 0 or round( d , step , method ) = 0  then  equal
    else if d < 0                               then  a is smaller
    else                                              a is greater
```

```formula
is_zero( a )  ⟺  compare( a , 0 ) = equal
```

The double test (first the raw difference, then the rounded difference) absorbs the case
where two values round to different multiples but their difference still rounds to zero.

---

## 2. Currency conversion

### 2.1 The conversion used by the session

```formula
converted = round_to_company_currency( amount × conversion_rate( from = selling currency ,
                                                                 to   = company currency ,
                                                                 company = the session's company ,
                                                                 at date = the contribution date ) )
```

Rounding may be suppressed (see section 2.3). The contribution date differs per
aggregation bucket and is stated in
[`accounting-effects.md`](accounting-effects.md) section 3.

### 2.2 The order's own rate

Every order stores the rate applicable at its date:

```formula
order_currency_rate = conversion_rate( from = company currency , to = selling currency ,
                                       company = the order's company , at date = the order date )
```

This rate is what the tax engine uses to produce the balance of every base line and tax
line of the order, so the closing entry's converted amounts for sales and taxes come from
the *order's* rate, while the converted amounts for tenders come from a conversion done at
the *payment's* date. The two can legitimately differ when a session spans a rate change.

### 2.3 Unrounded conversion

Cost conversion is done **without** rounding, so that the cost of a line of many units is
not distorted by rounding a per-unit figure:

```formula
total_cost = quantity × convert_unrounded( unit_cost , from = cost currency , to = order currency ,
                                           company = the line's company , at date = the order date )
```

### 2.4 Conversion in the selling application

The selling application does not call the server for rates. It receives, for every
currency it needs, the currency's rate relative to the company currency at loading time,
and converts by ratio:

```formula
converted = amount × ( target_currency_rate ÷ source_currency_rate )
```

This is used only for pricelist prices expressed in another currency, and for the product
prices converted at load time.

**Worked example.** The point of sale currency has rate 1.0 (it is the company currency);
a pricelist is expressed in a currency with rate 0.85. A rule fixing the price at 20.00 in
the pricelist currency yields `20.00 × (1.0 ÷ 0.85) = 23.529…`, later rounded to the
product-price precision.

---

## 3. Price of a product

### 3.1 The base price

```formula
base_price = list_price_of_the_variant + attribute_price_extra
```

The list price of the variant already includes the price supplements of the attribute
values that create a variant. The extra added here is the sum of the supplements of the
attribute values that do **not** create a variant, plus any custom-value supplement.

When no pricelist applies, the base price is the price.

### 3.2 Selecting the pricelist rule

The rules of the pricelist are examined in four groups, in this order, and the first group
that yields a rule wins:

1. Rules targeting the exact **variant**.
2. Rules targeting the **product template**.
3. Rules targeting any **product category** that is an ancestor of, or equal to, the
   product's category.
4. **Global** rules (targeting neither a product nor a category).

Within a group, the best rule is chosen as follows:

```
best = nothing
for each rule in the group:
    if the rule has a start date and that date is in the future, skip it
    if the rule has an end date and that date is in the past, skip it
    if the rule has a minimum quantity and that quantity is greater than the quantity asked, skip it
    if best is nothing, or the rule's minimum quantity is greater than best's minimum quantity:
        best = rule
```

So among the applicable rules of a group, the one with the **largest** minimum quantity
wins — the most specific quantity break. Rules are examined in the order in which the
pricelist holds them, which is the platform's own ordering of pricelist rules, so ties on
minimum quantity are broken by that order (the last one examined wins).

**Lot-tracked products.** When the price is computed for a line of a lot-tracked product,
the quantity used for the quantity break is not the line's own quantity but the **sum of
the quantities of every line of the order carrying the same product**. This makes a
quantity break apply across lines that had to be split because they carry different lots.

### 3.3 Applying the rule

Let `price` start at the base price.

**Step 1 — the base of the rule.**

| Rule base | Effect |
| --- | --- |
| The product's sales price (the default) | `price` stays the base price. |
| Another pricelist | `price` becomes the result of running this whole algorithm again against that pricelist, with the same quantity and the same attribute extra. The recursion alerts the operator when the base pricelist was not loaded into the selling application. |
| The product's cost | `price` becomes the standard price of the variant, or of the template when no variant is given. |

**Step 2 — currency alignment.** When the pricelist currency differs from the selling
currency:

```formula
price = price × ( pricelist_currency_rate ÷ selling_currency_rate )
```

**Step 3 — the computation.**

| Computation | Formula |
| --- | --- |
| Fixed price | `price = fixed_price` |
| Percentage | `price = price − price × ( percentage ÷ 100 )` |
| Formula | See below. |

The formula computation, in this exact order, with `limit` set to the value of `price`
before any of these steps:

```formula
price = price − price × ( price_discount ÷ 100 )
```

```formula
price = round( price , step = price_round )                       when a rounding step is given
```

```formula
price = price + price_surcharge                                   when a surcharge is given
```

```formula
price = maximum( price , limit + price_min_margin )               when a minimum margin is given
```

```formula
price = minimum( price , limit + price_max_margin )               when a maximum margin is given
```

**Step 4 — currency restoration.** When step 2 applied:

```formula
price = price × ( selling_currency_rate ÷ pricelist_currency_rate )
```

The result is **not** rounded here. It must be rounded to the product-price precision by
the caller. Rounding inside the algorithm would corrupt the recursive case where one
pricelist is based on another.

**Worked example.** Product with list price 100.00, attribute extra 5.00, so the base
price is 105.00. A pricelist rule of the formula kind with a discount of 10 percent, a
rounding step of 0.10, a surcharge of −0.01 and a minimum margin of 0.00, both currencies
equal:

```formula
limit = 105.00
price = 105.00 − 105.00 × 0.10 = 94.50
price = round( 94.50 , 0.10 ) = 94.50
price = 94.50 + ( − 0.01 ) = 94.49
price = maximum( 94.49 , 105.00 + 0.00 ) = 105.00
```

The minimum margin overrides the discount entirely; the resulting price is 105.00.

### 3.4 Price of a combo

A combo product has a single list price which must be distributed over the chosen
component lines so that the components sum back to it.

Let the parent list price be `P` (obtained by running section 3 for the combo product with
quantity 1), let each chosen component `i` have a combo base price `b_i`, a quantity
`q_i`, a parent coefficient `c_i` (one unless the parent line itself has a quantity
greater than one), an extra price `e_i` and a sum of non-variant attribute supplements
`a_i`.

```formula
original_total = Σ over chosen components of ( b_i × q_i )
```

Before the loop, when the last chosen component has a quantity greater than one and a
parent coefficient of one, it is split into two entries: one with its quantity reduced by
one and one with a quantity of one. This guarantees that the residual can always be put on
a component of quantity one.

Then, walking the components in order and carrying a running remainder initialised to `P`:

```formula
unit_price_i = round_to_product_price( ( b_i × P × c_i ) ÷ original_total )
```

```formula
remainder = remainder − ( unit_price_i × q_i ) ÷ c_i
```

and for the **last** component only:

```formula
unit_price_last = unit_price_last + remainder ;   remainder = 0
```

Finally:

```formula
line_price_i = unit_price_i + a_i + e_i
```

Components chosen from a combo whose free quantity is zero are **extra** components; they
are priced at their combo base price, plus a share of any remainder that survived the
first loop, plus their attribute supplements and extra price:

```formula
extra_unit_price_j = round_to_product_price( b_j )
```

```formula
share_j = round_to_product_price( ( b_j × P ) ÷ extra_original_total )      when a remainder survives
```

```formula
extra_unit_price_j = extra_unit_price_j + share_j ;  remainder = remainder − share_j × q_j
```

and the last extra component additionally receives `remainder ÷ q_last`.

The resulting component list is sorted by the position of each combo item inside the
parent product's combos, so that the receipt always shows the components in the configured
order.

**Worked example.** A menu priced at 12.00 with two combos: "main" with base price 8.00
and "drink" with base price 4.00. The customer chooses one main and one drink, both with
free quantity one, no extras, no attribute supplements.

```formula
original_total = 8.00 × 1 + 4.00 × 1 = 12.00
```

```formula
unit_price_main  = round( ( 8.00 × 12.00 × 1 ) ÷ 12.00 ) = 8.00 ;  remainder = 12.00 − 8.00 = 4.00
unit_price_drink = round( ( 4.00 × 12.00 × 1 ) ÷ 12.00 ) = 4.00 ;  remainder = 4.00 − 4.00 = 0.00
unit_price_drink = 4.00 + 0.00 = 4.00
```

The two component lines are 8.00 and 4.00, summing to the menu price. If the menu were
priced at 11.99 instead, the main would take
`round( 8.00 × 11.99 ÷ 12.00 ) = round( 7.9933 ) = 7.99` leaving 4.00, the drink would take
`round( 4.00 × 11.99 ÷ 12.00 ) = round( 3.9967 ) = 4.00` leaving 0.00, and the last
component absorbs nothing — total 11.99. The residual always lands on the last component,
so the components always sum exactly to the menu price.

---

## 4. Line amounts

### 4.1 The discounted unit price

```formula
price_after_discount = price_unit × ( 1 − discount_percentage ÷ 100 )
```

No rounding is applied here; the raw value is handed to the tax engine.

### 4.2 The signed quantity

```formula
order_sign = −1  when the order is a refund order ( its refund flag is set, or its total is negative )
             +1  otherwise
```

```formula
signed_quantity = line_quantity × order_sign
```

A line is itself a **refund line** when `price_unit × quantity < 0`. Note that this is a
property of the line, while the order sign is a property of the order: an ordinary order
may contain a refund line (a line typed with a negative quantity) and a refund order may
contain a line with a positive quantity.

### 4.3 The two line amounts

The taxes to apply — the line's taxes after the order's fiscal position has mapped them —
are evaluated over `price_after_discount` for `signed_quantity` units, in the order's
currency, for the product and the order's customer.

```formula
price_subtotal      = tax_exclusive_total   returned by the engine
```

```formula
price_subtotal_incl = tax_inclusive_total   returned by the engine
```

When the line has no taxes at all, both amounts are simply
`price_after_discount × quantity`.

### 4.4 Worked example

Unit price 12.10 including a 21 percent tax, quantity 3, discount 10 percent, ordinary
sale.

```formula
price_after_discount = 12.10 × ( 1 − 10 ÷ 100 ) = 10.89
```

```formula
line_inclusive = round_to_currency( 10.89 × 3 ) = 32.67
```

```formula
line_exclusive = round_to_currency( 32.67 ÷ 1.21 ) = 27.00
```

```formula
line_tax = 32.67 − 27.00 = 5.67
```

---

## 5. Taxes

### 5.1 Price-excluded tax

```formula
tax_amount = round_to_currency( base × rate ÷ 100 )
```

```formula
total_included = base + tax_amount
```

### 5.2 Price-included tax

The unit price already contains the tax. The base is obtained by division and the tax by
subtraction, never the other way round.

```formula
base = round_to_currency( inclusive_amount ÷ ( 1 + rate ÷ 100 ) )
```

```formula
tax_amount = inclusive_amount − base
```

**Worked example — the mandatory case.** A price-included tax of twenty-one percent on
twelve point one zero:

```formula
base = round_to_currency( 12.10 ÷ 1.21 ) = round_to_currency( 10.000000 ) = 10.00
```

```formula
tax_amount = 12.10 − 10.00 = 2.10
```

**Why subtraction and not multiplication.** On an inclusive price of 12.13:

```formula
base = round_to_currency( 12.13 ÷ 1.21 ) = round_to_currency( 10.024793 ) = 10.02
```

```formula
tax_amount_by_subtraction   = 12.13 − 10.02 = 2.11
```

```formula
tax_amount_by_multiplication = round_to_currency( 10.02 × 0.21 ) = 2.10
```

Only the subtraction keeps `base + tax = 12.13`. The engine always uses subtraction, and a
reimplementation must do the same or the closing entry will drift by one hundredth per
affected line.

### 5.3 Rounding scope

The company chooses between two tax rounding scopes:

| Scope | Meaning |
| --- | --- |
| Round per line | Each base line's taxes are rounded to the currency independently, then summed. |
| Round globally | The taxes are computed unrounded per line, summed per tax, and the sum is rounded once. The per-line figures are then adjusted so that they still add up to the rounded total. |

The selling application receives the company's choice and applies the same scope, so that
the receipt total matches the invoice total.

**Worked example.** Three lines of 3.33 each with a 21 percent price-excluded tax.

- Per line: `round(3.33 × 0.21) = 0.70` three times, total tax 2.10, total 12.09.
- Globally: `3.33 × 0.21 × 3 = 2.0979`, rounded once to 2.10, total 12.09. Here the two
  agree; with lines of 1.11 they do not: per line
  `round(1.11 × 0.21) = 0.23` three times, total 0.69, while globally
  `1.11 × 0.21 × 3 = 0.6993` rounds to 0.70.

### 5.4 Fiscal position mapping

The taxes of the line are mapped through the order's fiscal position before any
computation. The mapping is:

```
if the fiscal position has no tax mapping at all:
    if any of the taxes is itself attached to a fiscal position, the result is the empty set
    otherwise the taxes pass through unchanged
otherwise, for each tax:
    if the mapping contains an entry for that tax, append every replacement tax of that entry
    otherwise append the tax itself
```

The same rule governs the mapping of accounts: the income account and the expense account
are replaced when the fiscal position holds a mapping for them.

### 5.5 Tax groups on the receipt

The taxes of a line are grouped for display by their tax group's receipt label. The label
shown on a line is the set of distinct non-empty receipt labels of the line's taxes after
fiscal position mapping, joined by a single space.

---

## 6. Order totals

### 6.1 The server computation

1. `amount_paid` = the sum of the amounts of the order's payments (change, being negative,
   reduces it).
2. `amount_return` = minus the sum of the negative payment amounts.
3. Build the base lines, add the tax details, round the tax details per base line.
4. Ask for the tax totals summary, in the order currency, for the order's company, handing
   it the configuration's cash rounding definition **only when** the configuration has
   cash rounding, does not restrict rounding to cash, and has a rounding method.
5. `refund_factor` = −1 when the order is a refund order, +1 otherwise.
6. `amount_tax` = `refund_factor × summary tax amount in currency`.
7. `amount_total` = `refund_factor × summary total amount in currency`.
8. `amount_difference` = `amount_paid − amount_total`.

### 6.2 The client computation

The selling application computes the same summary but keeps two totals side by side:

```formula
total_with_rounding    = summary total amount in currency
```

```formula
total_without_rounding = summary total amount in currency − summary cash rounding base amount in currency
```

and exposes:

| Quantity | Definition |
| --- | --- |
| Price including tax | `total_without_rounding` |
| Rounded price including tax | `total_with_rounding` |
| Price excluding tax | The summary's base amount |
| Tax amount | The summary's tax amount in currency |
| Total due | `round_to_currency( total_without_rounding )` when the configuration has cash rounding, otherwise `round_to_currency( total_with_rounding )` |
| Amount paid | `round_to_currency( Σ over payments that are settled and are not change of the payment amount )` |

Note that the client's amount-paid figure **excludes** change payments, because change
lines are created after the order is transmitted; the server's figure **includes** them,
because by then they exist and their negative amount is exactly what makes the paid amount
equal the net cash kept.

### 6.3 The displayed price

```formula
displayed_amount = round_to_currency( price including tax )    when the tax display setting is tax-included
                   round_to_currency( price excluding tax )    when it is tax-excluded
```

The same switch governs the unit price shown on each line and on each product button.

---

## 7. Cash rounding, remaining due and change

### 7.1 Is the order rounded at all?

```formula
order_is_rounded ⟺  ( cash_rounding is enabled and rounding is not restricted to cash )
                    or ( cash_rounding is enabled and at least one tender uses a cash method )
```

### 7.2 The remaining amount due

```formula
raw_remaining = round_to_currency( total_due − amount_paid )
```

```formula
is_negative_order = total_due < 0
```

If the order is negative and `raw_remaining ≥ 0`, or the order is positive and
`raw_remaining ≤ 0`, the remaining due is exactly zero — the tenders already cover the
total.

Otherwise:

```formula
signed_remaining = − raw_remaining   when the order is negative,  raw_remaining otherwise
```

```formula
magnitude = 0                     when order_is_rounded and asymmetric_round( signed_remaining ) = 0
            | raw_remaining |     otherwise
```

```formula
remaining_due = round_to_currency( − magnitude )  when the order is negative
                round_to_currency(   magnitude )  otherwise
```

In words: a residue smaller than half a rounding step (or smaller than a whole step, for
the non-half methods) is treated as fully settled.

### 7.3 The applied rounding

```formula
total = price including tax ( unrounded )
```

```formula
raw_remaining = round_to_currency( total − amount_paid )
```

```formula
signed_remaining = − raw_remaining when total < 0, raw_remaining otherwise
```

```formula
is_done ⟺ order_is_rounded and ( signed_remaining ≤ 0 or asymmetric_round( signed_remaining ) = 0 )
```

When the order is not done, the applied rounding is zero. Otherwise:

```formula
rounded_remaining = asymmetric_round( signed_remaining )
```

```formula
applied_rounding = round_to_currency( signed_remaining − rounded_remaining )   when total < 0
                   round_to_currency( rounded_remaining − signed_remaining )   otherwise
```

### 7.4 The change

```formula
raw_remaining = total_due − amount_paid
```

If the order is negative and `raw_remaining ≤ 0`, or the order is positive and
`raw_remaining ≥ 0`, the change is zero.

Otherwise:

```formula
gross = | price including tax | − | amount paid | + ( − applied_rounding when negative, + applied_rounding otherwise )
```

```formula
change = − round_to_currency( gross )   when the order is negative
           round_to_currency( gross )   otherwise
```

and finally, when the configuration has cash rounding at all:

```formula
change = asymmetric_round( change )
```

### 7.5 The default amount proposed for a tender

```formula
proposal = round( remaining_due , step , method )   when the method is a cash method and cash rounding is enabled
           remaining_due                            otherwise
```

and, when that proposal is zero, the change is proposed instead (so that the payment
screen offers to hand the change back).

### 7.6 The payable amount on the server

```formula
payable = round( amount , step , method )
```
when the configuration has cash rounding and either the caller forces rounding or rounding
is not restricted to cash;

```formula
non_cash_amount = Σ over tenders whose method is not a cash method of the tender amount
```

```formula
payable = non_cash_amount + round( amount − non_cash_amount , step , method )
```
when rounding is restricted to cash and at least one cash tender exists. Only the residue
settled in cash is rounded: the non-cash tenders pay their exact share.

Finally the result is rounded to the currency.

**Worked example — the mandatory five-hundredths case.** Rounding step 0.05, method
`HALF-UP`, restriction off. An order totalling 12.13:

```formula
payable = round( 12.13 , 0.05 , HALF-UP ) = 12.15
```

The customer hands over a twenty:

```formula
amount_paid = 20.00 ;  total_due = 12.13 ;  applied_rounding = 12.15 − 12.13 = 0.02
```

```formula
gross = | 12.13 | − | 20.00 | + 0.02 = − 7.85
```

```formula
change = round_to_currency( − 7.85 ) = − 7.85
```

The magnitude handed back is 7.85, and `20.00 − 7.85 = 12.15`, exactly the payable amount.
The change is then asymmetrically rounded, which leaves it unchanged since it is already a
multiple of 0.05.

With the restriction on and 10.00 already taken on a card:

```formula
payable = 10.00 + round( 12.13 − 10.00 , 0.05 , HALF-UP ) = 10.00 + 2.15 = 12.15
```

and the cash tender is 2.15.

---

## 8. Cash balances

### 8.1 Captured cash payments

```formula
captured_cash = Σ  over Point of Sale Payments of the session
                   whose payment method is the first cash-kind method of the configuration
                   and whose order state is paid, invoiced or posted
                of the payment amount
```

Change, being a negative payment on the same cash method, is included and correctly
reduces the figure.

### 8.2 Theoretical closing balance

```formula
statement_total = cash_real_transaction   when the session is closed
                  Σ of the amounts of the session's cash statement lines   otherwise
```

```formula
theoretical_closing_balance = starting_balance + statement_total + captured_cash
```

### 8.3 The difference

```formula
cash_difference = counted_ending_balance − theoretical_closing_balance
```

A negative difference is a **loss** (less cash in the drawer than expected); a positive
one is a **profit**.

### 8.4 The opening difference

```formula
opening_difference = counted_opening_amount − pre_filled_starting_balance
```

where the pre-filled starting balance is the counted ending balance of the previous
session of the same configuration, or zero. The difference is only reported in the message
thread; the starting balance is then overwritten with the counted amount, so an opening
discrepancy never becomes an accounting entry.

### 8.5 Worked example — the mandatory scenario

Opening count 0.00, previous session closed at 0.00, no manual cash movement, one cash
tender of 25.00, counted closing amount 24.50.

```formula
opening_difference = 0.00 − 0.00 = 0.00
```

```formula
captured_cash = 25.00
```

```formula
theoretical_closing_balance = 0.00 + 0.00 + 25.00 = 25.00
```

```formula
cash_difference = 24.50 − 25.00 = − 0.50
```

A loss of 0.50, posted as a statement line debiting the cash loss account and crediting
the cash account.

### 8.6 The authorised difference check

When the configuration sets a maximum difference and the acting user is not an
administrator:

```formula
closing_allowed ⟺ | cash_difference | ≤ amount_authorized_diff
```

The check is performed in the selling application, which receives both the limit and the
user's role.

---

## 9. Cost and margin

### 9.1 The unit cost of a line

```
1. If stock moves are available and the product is storable with a first-in-first-out or
   average cost method:
       unit_cost = the valuation unit price of the moves of that product
       if unit_cost is zero at the cost currency's precision and the order has a shipping date:
           if the line refunds another line:
               unit_cost = refunded_line_total_cost ÷ refunded_line_quantity
           otherwise:
               unit_cost = the product's standard price
2. Otherwise:
       unit_cost = the product's standard price
```

### 9.2 The total cost of a line

```formula
total_cost = line_quantity × convert_unrounded( unit_cost ,
                                                from = the product's cost currency ,
                                                to   = the order's currency ,
                                                company = the line's company ,
                                                at date = the order date or today )
```

The cost is computed once and then frozen by the computed flag, so a later change of the
product's standard price never rewrites history.

### 9.3 Margins

```formula
line_margin = ( line_tax_excluded_amount × order_sign ) − line_total_cost
```

```formula
line_margin_percentage = line_margin ÷ ( line_tax_excluded_amount × order_sign )
```

The percentage is zero when the tax-excluded amount is zero at the currency's precision. A
line whose product is a combo header always reports zero for both.

```formula
order_margin = Σ over lines of line_margin              when every line has its cost computed
               0                                         otherwise
```

```formula
order_untaxed = round_to_currency( Σ over lines of line_tax_excluded_amount ) × order_sign
```

```formula
order_margin_percentage = order_margin ÷ order_untaxed    ( zero when the denominator is zero )
```

### 9.4 Worked example

A line of 3 units sold at 32.67 tax-excluded 27.00, with a unit cost of 6.00 in the
company currency and an order currency equal to the company currency:

```formula
total_cost = 3 × 6.00 = 18.00
```

```formula
line_margin = 27.00 − 18.00 = 9.00
```

```formula
line_margin_percentage = 9.00 ÷ 27.00 = 0.333333    ( rendered as 33.3333 percent )
```

---

## 10. Refund quantities

### 10.1 The quantity of a refund line

```formula
refund_quantity = − ( original_line_quantity − already_refunded_quantity )
```

where the already-refunded quantity is a positive number (see below). A refund created
from a line that has not been refunded at all therefore carries the negation of the whole
original quantity.

### 10.2 The already-refunded quantity of a line

```formula
refunded_qty = − Σ over the refunding lines of this line whose order is not cancelled
                 of the refunding line's quantity
```

Because the refunding quantities are negative, the sum is negative and its negation is
positive.

### 10.3 The refund guard

```formula
already_refunded = Σ over the other refunding lines of the same original line
                     whose order is not cancelled
                   of the refunding line's quantity
```

```formula
total_refunded = | already_refunded | + | proposed_quantity |
```

The change is refused when `| original_line_quantity | − total_refunded < 0`, with *"You
cannot refund more than the outstanding quantity for this product."*

### 10.4 Has refundable lines

```formula
has_refundable_lines ⟺ ∃ a line with  compare( line_quantity , refunded_qty ,
                                               precision = product-unit precision ) = greater
```

### 10.5 Worked example — the mandatory single-line refund

An order with one line of 3 units at 12.10 including 21 percent tax. One unit is returned.

```formula
refunded_qty before the return = 0
```

The cashier reduces the proposed refund quantity from −3 to −1.

```formula
already_refunded = 0 ;  total_refunded = 0 + 1 = 1 ;  3 − 1 = 2 ≥ 0 , so the change is allowed
```

The refund line carries quantity −1, unit price 12.10, the same 21 percent tax:

```formula
line_inclusive = − 12.10 ;  line_exclusive = round_to_currency( − 12.10 ÷ 1.21 ) = − 10.00 ; tax = − 2.10
```

After the refund order is paid, the original line's already-refunded quantity becomes
`− ( − 1 ) = 1`, and the original order still has refundable lines because `3 > 1`.

---

## 11. Numbering formulas

### 11.1 The receipt number

```formula
receipt_number = last_two_digits_of_current_year ‖ device_identifier ‖ "-" ‖ configuration_identifier ‖ "-" ‖ next_backend_sequence_value
```

### 11.2 The tracking number

```formula
tracking_number = next_backend_sequence_value  modulo  1000
```

rendered as a decimal number without leading zeroes.

### 11.3 The session-unique sequence number

```formula
sequence_number = next_order_sequence_value  with the sequence prefix removed from the front
                                             and the sequence suffix removed from the end
```

### 11.4 The session name

```formula
session_name = ( configuration_name when the session sequence prefix is exactly "/" , else nothing )
               ‖ next_session_sequence_value
               ‖ ( the previous value of the name field, when that value was not "/" )
```

### 11.5 The order name

```formula
order_name = refunded_order_name ‖ " REFUND"                                   for a refund order
```

```formula
order_name = prefix ‖ " - " ‖ last_hyphen_separated_part_of_receipt_number ‖ suffix
```

with `prefix` the order sequence's prefix, falling back to the configuration name when the
sequence has none, and `suffix` equal to `" - "` followed by the sequence suffix when the
sequence has one, and empty otherwise.

**Worked example.** Configuration 3 named "Shop", backend sequence value `000127`, device
identifier `0`, year 2026, order sequence with no prefix and no suffix:

```formula
receipt_number  = "26" ‖ "0" ‖ "-" ‖ "3" ‖ "-" ‖ "000127" = "260-3-000127"
tracking_number = 127 modulo 1000 = "127"
order_name      = "Shop" ‖ " - " ‖ "000127" = "Shop - 000127"
```

---

## 12. The client-server agreement contract

The selling application and the server must produce identical figures. The contract that
makes this possible has six clauses.

1. **Same engine, same inputs.** The selling application runs a faithful port of the tax
   engine. It receives, at session opening, every tax with its kind, amount, sequence,
   price-included flag, base-affecting flag, negative-factor flag, children and tax group;
   every currency with its rounding step, decimal places and rate; the company's tax
   rounding scope; every fiscal position with its tax map; and every pricelist with its
   rules. It computes with exactly those inputs.
2. **Rate is one on the client.** The base lines built by the selling application are given
   a conversion rate of one, because the selling application works only in the selling
   currency. The server supplies the real rate when it rebuilds the base lines, so the
   amounts in currency agree and the balances are the server's business.
3. **The client writes the amounts it computed.** Before transmission the selling
   application stamps the order with its own figures: the paid amount, the tax amount, the
   total (rounded to the currency), the change, and each line's tax-excluded and
   tax-inclusive amounts.
4. **The server does not trust the paid amount.** On receipt the server immediately
   recomputes `amount_paid` as the sum of the payment amounts and writes it back, before
   anything else happens. Every other stamped figure is accepted as transmitted, and the
   server's own recomputation runs only when the payments or the lines are changed
   afterwards.
5. **Rounding happens in the same places.** Both sides round the tax details per base line
   before summing, both apply the company's rounding scope, both apply cash rounding to the
   whole document only when the configuration does not restrict it to cash, and both use
   round-half-away-from-zero for currency rounding.
6. **Divergence is caught at closing.** The closing entry is checked for balance before it
   is posted. When it does not balance, the whole transaction is rolled back and the
   operator is offered the forced-close wizard rather than a broken ledger. This is the
   backstop that makes clauses one to five auditable: a systematic divergence shows up as a
   systematic imbalance.

---

## 13. Aggregation formulas of the closing entry

Restated here as pure arithmetic; the accounting consequences are in
[`accounting-effects.md`](accounting-effects.md).

### 13.1 Sales bucket

```formula
sales_amount( account , sign , taxes , base_tags , product )
    = Σ over the base lines of the non-invoiced closed orders matching that key
      of the base line's amount in currency
```

```formula
sales_balance( key ) = Σ of the base line's balance
```

```formula
sales_quantity( key ) = Σ of the base line's quantity          ( only when the per-product option is on )
```

### 13.2 Tax bucket

```formula
tax_amount( account , repartition_line , tags ) = Σ of the tax line's amount in currency
```

```formula
tax_balance( key ) = Σ of the tax line's balance
```

```formula
tax_base( key ) = Σ of the tax line's tax base amount
```

### 13.3 Receivable buckets

```formula
aggregated_cash( method )    = Σ over non-pay-later cash tenders of non-identifying methods of the tender amount
aggregated_bank( method )    = Σ over non-pay-later bank tenders of non-identifying methods of the tender amount
per_payment_cash( payment )  = the tender amount                       ( identifying cash methods )
per_payment_bank( payment )  = the tender amount                       ( identifying bank methods )
aggregated_later( method )   = Σ over pay-later tenders of non-invoiced orders of non-identifying methods
per_payment_later( payment ) = the tender amount                       ( identifying pay-later methods )
invoiced_aggregated( method )= Σ over non-pay-later tenders of invoiced orders of non-identifying methods
invoiced_per_payment(payment)= the tender amount                       ( identifying methods, invoiced orders )
```

Tenders whose amount is zero at the selling currency's precision are skipped before any of
these sums.

### 13.4 Rounding bucket

```formula
rounding_difference = Σ over the non-invoiced closed orders of
                      ( order_amount_paid + Σ of the order's base line and tax line amounts in currency )
```

### 13.5 Cost buckets

```formula
move_amount = ( convert_unrounded( move_quantity , move_unit , product_reference_unit )
                × ( −1 when the move is incoming, +1 otherwise ) )
              × move_unit_price
```

```formula
cost_of_goods_sold( expense_account ) = Σ over all considered moves of move_amount
stock_delivered( valuation_account )  = Σ over outgoing moves of move_amount
stock_returned( valuation_account )   = Σ over incoming moves of move_amount
```

### 13.6 The balance invariant

The closing entry balances when

```formula
Σ of every line's balance = 0
```

Expanding, with all figures in company currency and with sales and taxes negative for
sales:

```formula
  Σ sales_balance
+ Σ tax_balance
+ Σ aggregated_cash + Σ per_payment_cash
+ Σ aggregated_bank + Σ per_payment_bank
+ Σ aggregated_later + Σ per_payment_later
− Σ invoiced_aggregated − Σ invoiced_per_payment
+ rounding_balance
+ Σ cost_of_goods_sold − Σ stock_delivered − Σ stock_returned
= 0
```

The cost and valuation terms cancel each other exactly, because they are the same amounts
on opposite sides. The invoiced terms cancel the corresponding tender terms exactly. What
remains is that the tenders of the uninvoiced orders must equal their sales plus their
taxes plus the rounding, which is precisely what the fully-paid test guarantees per order.

---

## 14. The sales details report

The sales details document aggregates a set of sessions, or a date range and a set of
configurations, into one printable summary. It is the document a manager prints at the end
of a day.

### 14.1 Selecting the period

```
if a start instant is supplied:
    start = that instant
else:
    start = today at midnight in the acting time zone, expressed in universal time
if a stop instant is supplied:
    stop = that instant
    if stop is earlier than start:
        stop = start + 1 day − 1 second
else:
    stop = start + 1 day − 1 second
```

### 14.2 Selecting the orders

```
condition = the order state is paid or posted
if session identifiers are supplied:
    condition = condition and the order's session is one of them
else:
    condition = condition and the order date lies between start and stop inclusive
    if configuration identifiers are supplied:
        condition = condition and the order's configuration is one of them
orders = every order matching the condition
```

### 14.3 Selecting the sessions and the reporting currency

```
if configuration identifiers are supplied:
    configurations = those configurations
    candidate currencies = their currencies
    sessions = the supplied sessions when there are any,
               otherwise every session of those configurations whose opening instant is at
               or after start and whose closing instant is at or before stop
else:
    sessions = the supplied sessions
    configurations = their configurations
    candidate currencies = the currencies of those configurations
```

```formula
reporting_currency = the single candidate currency when they are all the same,
                     otherwise the acting company's currency
```

### 14.4 The grand total

```formula
total = Σ over the selected orders of
        ( order_total                                             when the order's pricelist currency is the reporting currency
          convert( order_total , from = pricelist currency , to = reporting currency ,
                   company = the order's company ,
                   at = the order date or today )                  otherwise )
```

### 14.5 Products sold and products refunded

The lines of the selected orders are split in two: lines of orders that are not refund
orders feed the **sold** section, lines of refund orders feed the **refunded** section.
Both are built the same way.

The grouping is two-level. The outer key is the name of the **first** counter category of
the line's product template, or the label "Not Categorized" when the product has none. The
inner key is the triple (product variant, unit price, discount percentage).

For each inner group:

```formula
group_quantity = round( Σ of | line quantity | , precision = the product-unit precision )
```

```formula
group_total_paid = Σ over the lines of round_to_currency( price_unit × quantity × ( 100 − discount ) ÷ 100 )
```

```formula
group_base_amount = Σ of the line tax-excluded amounts
```

When a line is a combo header, the group also carries a label made of the names of its
component products, joined by a comma and a space, wrapped in parentheses.

Each product row reports the product identifier, its display name, its barcode, the
quantity, the unit price, the discount, the unit name, the total paid, the base amount and
the combo label. Rows are sorted by product name inside a category; categories are sorted
by name.

Per category:

```formula
category_quantity = round( Σ of the row quantities , precision = the product-unit precision )
```

```formula
category_total = round( Σ of the row base amounts , precision = the product-price precision )
```

The section grand totals are computed over the **distinct** rows (rows that are identical
in every field are counted once):

```formula
section_quantity = Σ over distinct rows of the row quantity
```

```formula
section_total = Σ over distinct rows of the row base amount
```

### 14.6 Taxes

Two tax tables are built, one for the sold section and one for the refunded section. For
each line:

- when the line has taxes after fiscal position mapping, the taxes are evaluated over
  `price_unit × ( 1 − discount ÷ 100 )` for the line quantity, in the line currency, for
  the product and the order's customer; each resulting tax contributes its amount to that
  tax's running tax amount and its base, rounded to the currency, to that tax's running
  base amount;
- when the line has no tax, its tax-inclusive amount is added to the base amount of a
  pseudo-tax labelled "No Taxes".

In parallel:

```formula
section_base_amount = Σ over the lines of ( line tax-excluded amount × ( −1 for a refund order, +1 otherwise ) )
```

and the summary line of each table is:

```formula
table_tax_amount = Σ over the taxes of the table of the tax amount
```

```formula
table_base_amount = section_base_amount
```

### 14.7 Payments

Payments of the selected orders are grouped by payment method **and** session, each group
reporting the method identifier, the session, the method name, whether the method is a
cash method, the summed amount and the method's journal.

For each session, each of its payment groups is completed:

**Cash method group.**

```formula
final_count = group_total + session_starting_balance + session_frozen_cash_transactions
```

```formula
money_counted = session_counted_ending_balance
```

```formula
money_difference = money_counted − final_count
```

and a movement list is built: an entry "Cash Opening" with the starting balance when that
balance is not zero, then one entry per cash statement line of the session, labelled with
the statement label or, when it has none, "Cash in *n*" or "Cash out *n*" numbered
separately per direction.

**Non-cash method group with a recorded closing difference.** The closing-difference entry
is found by searching for an entry whose reference is
`Closing difference in <method name> (<session identifier>)` in the method's journal. When
it exists:

```formula
final_count = group_total
```

```formula
money_difference = − entry_total  when the entry touches the journal's loss account,
                   + entry_total  otherwise
```

```formula
money_counted = final_count + money_difference
```

and, when the difference is not zero, a single movement entry is added labelled
"Difference observed during the counting (Profit)" or "… (Loss)".

**Non-cash method group with accounting payments.** When no closing-difference entry
exists but accounting payments of that method exist for that session:

```formula
final_count = group_total
```

```formula
money_counted = Σ of the signed amounts of those accounting payments
```

```formula
money_difference = money_counted − final_count
```

with the same single movement entry when the difference is not zero.

**Sessions with no cash method at all.** A synthetic cash row named
`Cash <session identifier>` is inserted first, with a total of zero and:

```formula
final_count = previous_closed_session_counted_ending_balance + session_frozen_cash_transactions
```

```formula
money_counted = session_counted_ending_balance
```

```formula
cash_difference = money_counted − final_count
```

Its movement list starts with a "Cash Opening" entry when the previous session's counted
ending balance is greater than zero, and then lists the session's cash statement lines
ordered by date — dropping the **last** one when the cash difference is not zero at the
session currency's precision, because that last line is the difference itself and must not
be double-counted.

Finally the payments are also totalled per method across sessions:

```formula
method_total = Σ over the groups of that method of the group total
```

and each group's displayed name becomes the method name followed by a space and the
session identifier. The per-method totals are shown only when the document was requested
for a date range rather than for specific sessions.

### 14.8 Cash rounding total

```formula
cash_rounding_total = round_to_reporting_currency(
      Σ over the selected orders of
        ( ( order_paid_amount − order_total )                              when the session currency is the reporting currency
          convert( order_paid_amount − order_total ,
                   from = the session currency , to = the reporting currency ,
                   company = the order's company , at = the order date or today )   otherwise ) )
```

### 14.9 Discounts

```formula
discount_number = the number of lines of the selected orders whose discount percentage is greater than zero
```

```formula
discount_amount = Σ over those lines of the line's discount amount
```

### 14.10 Invoices

For each session: the session identifier and, for each invoiced order of that session, the
invoice identifier, the signed invoice total in company currency, the invoice number and
the order's receipt number.

```formula
invoice_total = Σ over the sessions of Σ over their invoiced orders of the order's paid amount
```

```formula
total_paid = Σ over the sessions of the session's captured payments total
```

### 14.11 Header values

The document reports: the opening and closing notes when exactly one session is covered;
the state, which is the session's state when exactly one session was explicitly requested
and the literal value "multiple" otherwise; the reporting currency's symbol, its position
relative to the amount, the grand total and its decimal places; the number of orders; the
period; the session name when exactly one session was explicitly requested; and the names
of the configurations covered.

## 15. Preset time slots

```formula
slot_count_per_interval = the preset's capacity
```

```formula
interval_length = the preset's interval length, in minutes
```

The bookable instants are generated from the preset's working schedule: for each
attendance line of the schedule, starting at the line's start hour and advancing by the
interval length until the line's end hour is reached. A slot is bookable when

```formula
number of orders already booked at that instant  <  slot_count_per_interval
```

where the booked orders are the unfinished or paid orders of this preset, in an opened
session, created within the last day, whose scheduled time equals that instant.

**Worked example.** A preset with a capacity of 5 and an interval of 20 minutes, a
schedule running from 11.5 (half past eleven) to 14.0 (two o'clock). The generated
instants are 11:30, 11:50, 12:10, 12:30, 12:50, 13:10, 13:30, 13:50. If four orders are
already booked at 12:30 and one more is placed, the slot becomes unavailable to a sixth.

---

## 16. Digest indicator

The domain contributes one periodic indicator: the total of counter sales over the period.

```formula
counter_sales_total = Σ over the orders of the period, in the acting company,
                        whose state is paid, invoiced or posted
                      of the order total
```

rendered in the company currency.
