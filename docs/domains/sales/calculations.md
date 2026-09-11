# Sales — Calculations

Every formula and every algorithm of the sales domain, with its rounding rule, its precision, its
currency, its evaluation order and at least one worked numeric example.

## Conventions

- `round_to_currency(x)` rounds the amount `x` using the rounding step of the currency in question
  (for example 0.01 for a currency with two decimal places), applying the *round half away from
  zero* rule described in [../multi-currency/calculations.md](../multi-currency/calculations.md).
- `round_to(x, precision_name)` rounds `x` to the number of decimal places configured under that
  decimal-precision name. The precision names used in this domain are `Product Unit` (quantities,
  shipped default two decimal places), `Product Price` (unit prices, shipped default two decimal
  places) and `Discount` (discount percentages, shipped default two decimal places).
- `compare(a, b, precision)` returns −1, 0 or +1 after rounding both operands to the given
  precision; `is_zero(a, precision)` is `compare(a, 0, precision) = 0`.
- "the order currency" is the currency stored on the order; "the company currency" is the currency
  of the order's company.
- Quantities are always expressed in the *line unit* unless stated otherwise; conversions to the
  product's reference unit are written explicitly.

---

## 1. Header-level computations

### 1.1 Validity date (expiration)

```formula
if quotation_template_duration_days > 0 :
    expiration_date = today + quotation_template_duration_days days
else if company_default_validity_days > 0 :
    expiration_date = today + company_default_validity_days days
else :
    expiration_date = empty
```

`today` is the current date in the reader's time zone. The company default is shipped as 30 days; a
negative value is refused by a database check.

**Worked example.** Company default 30 days, no template, today is 11 September 2026 → expiration
date 11 October 2026. With a template whose duration is 7 days → 18 September 2026.

### 1.2 Currency of the order

```formula
order_currency = pricelist_currency  if a price list is set and it has a currency
               = company_currency    otherwise
```

### 1.3 Currency rate

```formula
currency_rate = conversion_rate( from = company_currency , to = order_currency ,
                                 company = order_company , date = date_part(order_date) )
```

The lookup is the standard rate lookup of the multi-currency domain: the most recent rate of the
target currency, for that company, whose validity date is not later than the given date. When the
order date is empty the current instant is used. The stored value is kept unrounded (no decimal
limit).

Every conversion from the order currency into the company currency inside this domain is performed
by **dividing** by this stored rate, not by re-reading the rate table. This guarantees that the
reporting figures of an order stay consistent with the amounts the customer sees even if the rate
table changes afterwards.

```formula
amount_in_company_currency = amount_in_order_currency ÷ currency_rate
```

**Worked example.** Company currency is the euro; the price list is in dollars; the rate on the
order date is 1.0870 dollars per euro. An order total of 1 087.00 dollars is reported internally as
1 087.00 ÷ 1.0870 = 1 000.00 euros.

### 1.4 Expected date

```formula
for each line L that is a goods line, not a display line and not a delivery-charge line :
    line_expected_date(L) = base_date(L) + customer_lead(L) days

base_date(L) = order_date      if the order status is 'sale' and the order date is set
             = current instant otherwise

expected_date = min over L of line_expected_date(L)        when the shipping policy is "as soon as possible"
              = max over L of line_expected_date(L)        when the shipping policy is "when all products are ready"
              = empty                                      when no such line exists, or the order is cancelled
```

Without inventory integration there is no shipping policy and the minimum is always taken.

**Worked example.** A confirmed order dated 1 March 2026 at 09:00 has three goods lines with lead
times of 3, 10 and 5 days. With the "as soon as possible" policy the expected date is 4 March 2026
at 09:00; with "when all products are ready" it is 11 March 2026 at 09:00.

### 1.5 Prepayment required amount

```formula
prepayment_required_amount = 0                                                if online payment is not required
                           = round_to_currency( amount_total × prepayment_percent )   otherwise
```

`prepayment_percent` is a fraction between zero (exclusive) and one (inclusive). Rounding uses the
order currency.

**Worked example.** Order total 3 000.00 euros, prepayment percentage 0.30 → 900.00 euros.

### 1.6 Confirmation amount reached

```formula
confirmation_amount_reached = ( compare_amounts( prepayment_required_amount , amount_paid ) ≤ 0 )
```

`compare_amounts` compares the two figures after rounding both to the order currency. The amount
paid is the sum of the amounts of the linked transactions whose state is *authorized* or *done*.

### 1.7 Order is paid

```formula
order_is_paid = ( compare_amounts( amount_paid , amount_total ) ≥ 0 )
```

### 1.8 Suggested amount on a payment link

```formula
remaining_balance = amount_total − amount_paid

suggested_amount = prepayment_required_amount   if the status is draft or sent and online payment is required
                 = remaining_balance            otherwise

maximum_amount   = remaining_balance
```

---

## 2. Line pricing

### 2.1 Line description

The description is rebuilt whenever the product or the linked line changes, in the language of the
order (the customer's language when the customer is not a public visitor, otherwise the reader's
language).

1. If the line has no product and is not an advance-invoice line, the description is left as it is.
2. If the line has a product:
   a. start with the product's customer-facing multi-line description;
   b. append the *variant suffix* (section 2.2);
   c. if the line is linked to another line and is **not** a combo item, append a new physical line
      reading "Option for: *display name of the linked line's product*", where the display name is
      rendered without the internal reference;
   d. if the order uses a quotation template and that template contains a line with the same
      product that carries its own description, replace steps (a) and (b) by that template
      description in the customer's language, followed by the variant suffix.
3. If the line is an advance-invoice line, the description is the advance-line description of
   [state-machines.md](state-machines.md), section 7.2.

### 2.2 Variant suffix

Let *chosen values* be the values of attributes that do not create variants which the customer
actually chose — that is, those whose display kind is *multiple choice*, plus those belonging to an
attribute line that offers more than one value. Let *custom values* be the free-text values the
customer typed.

1. If there are no custom values and no chosen values, the suffix is empty.
2. For every chosen value that is neither a multiple-choice value nor the target of a custom value,
   append a new physical line containing the value's display name.
3. Group the multiple-choice values by attribute and, for each attribute, append a new physical
   line reading "*attribute name*: *value*, *value*, …".
4. For each attribute value that carries a custom text, in the entity's own sort order, append a
   new physical line containing the custom value's display name (which itself reads
   "*attribute value*: *typed text*").

**Worked example.** A configurable desk with attributes *Legs* (two values, "Steel" chosen),
*Colour* (single value, not offered as a choice) and a multiple-choice *Options* attribute with
"Cable management" and "Power socket" chosen, plus a custom engraving text "For Anna". The
description is:

```
Desk
Steel
Options: Cable management, Power socket
Engraving: For Anna
```

### 2.3 Unit price

The unit price is recomputed whenever the product, the unit or the quantity changes, subject to
three suppression rules.

**Suppression rules — the price is left untouched when any of these holds.**

1. The line has no order, or is an advance-invoice line, or is a global-discount line.
2. The price was set manually and recomputation was not forced. "Set manually" means
   `compare_amounts(technical_price_unit, price_unit) ≠ 0` in the line currency (falling back to
   the company currency, then to the reader's company currency for unsaved records).
3. The invoiced quantity is strictly greater than zero.
4. The product's re-invoicing policy is *at cost* and the line is an expense line.

**Computation when not suppressed.**

```formula
if the line has no unit or no product :
    price_unit = 0 ;  technical_price_unit = 0
else :
    display_price   = see section 2.4
    price_unit      = tax_included_adjust( display_price )
    technical_price_unit = price_unit
```

`tax_included_adjust` is the product-level helper that converts a price expressed with the
product's own tax configuration into a price expressed with the taxes that actually apply after
fiscal-position mapping: when the product's sale taxes include tax in the price but the mapped
taxes do not (or the reverse), the price is corrected accordingly. The full rule is specified in
[../taxes/calculations.md](../taxes/calculations.md).

### 2.4 Display price

```formula
if product_type = combo :
    display_price = 0
else if the line carries a combo item :
    display_price = combo_item_price   (section 2.6)
else :
    display_price = display_price_ignoring_combos
```

```formula
pricelist_price = price computed by the selected price-list rule
                  for ( product with the extra-price context ,
                        quantity = ordered quantity, or 1 when the ordered quantity is zero ,
                        unit = line unit , date = order date , currency = order currency )

if the selected rule does not show a discount :
    display_price_ignoring_combos = pricelist_price
else :
    base_price = price computed by the same rule *before* its own discount
    display_price_ignoring_combos = max( base_price , pricelist_price )
```

"Does not show a discount" means either no rule was selected, or the discount feature group is
disabled, or the rule's computation method is not *percentage*. The maximum guarantees that a
*negative* discount (a surcharge) is folded into the displayed price instead of being shown as a
negative discount percentage.

The "extra-price context" adds, to the product's list price, the extra prices of the chosen
attribute values that do not create variants.

**Worked example.** List price 100.00; a price-list rule gives a 20 percent discount and the
discount feature is enabled. The rule price is 80.00, the base price is 100.00, so the display
price is max(100.00, 80.00) = 100.00 and the discount percentage becomes 20 (section 2.5). With the
discount feature disabled the display price is simply 80.00 and the discount stays at zero.

### 2.5 Discount derivation

Recomputed on the same triggers as the unit price.

1. If the line has no product or is a display line, the discount is set to zero.
2. If the order has no price list, or the discount feature group is disabled, or the line has no
   unit, stop (leaving the current value).
3. If the line is a combo item, the discount is copied from its combo line and the algorithm stops.
4. Set the discount to zero.
5. If the selected rule does not show a discount, stop.
6. Otherwise:

```formula
discount_candidate = ( base_price − pricelist_price ) ÷ base_price × 100

discount = discount_candidate    if ( discount_candidate > 0 and base_price > 0 )
                                 or ( discount_candidate < 0 and base_price < 0 )
         = 0                     otherwise
```

Division by zero is avoided by skipping the computation when the base price is zero. The stored
value is rounded to the `Discount` precision. The asymmetric test means a surcharge on a positive
price is never displayed as a negative discount.

**Worked example.** Base price 100.00, rule price 80.00 → (100 − 80) ÷ 100 × 100 = 20.00 percent.
Base price 100.00, rule price 110.00 → candidate −10.00 with a positive base → discount stays 0 and
the display price becomes 110.00.

### 2.6 Combo item price

A combo product line always has a price of zero; the price of the combo is spread over its item
lines in proportion to the *base price* declared on each combo choice.

1. Let `combo_product_price` be the display price of the combo line, computed while ignoring the
   combo logic (section 2.4).
2. For each combo choice of the combo product, convert its declared base price from the choice's
   currency into the order currency, at the order date, in the order's company. Call the results
   `base(c)` and their sum `total_base`.
3. If `total_base ≠ 0`:

```formula
combo_price(c) = round_to_currency( base(c) × combo_product_price ÷ total_base )
```

   otherwise every choice receives the same share:

```formula
combo_price(c) = round_to_currency( combo_product_price ÷ number_of_choices )
```

4. Compute the rounding residue and give it entirely to the **last** choice:

```formula
delta = combo_product_price − Σ over c of combo_price(c)
combo_price(last choice) = combo_price(last choice) + delta
```

5. The item line's display price is:

```formula
item_display_price = combo_price( choice of this item )
                   + convert( combo_item_extra_price
                              + extra price of the chosen no-variant attribute values ,
                              from = combo item currency , to = order currency ,
                              company = order company , date = order date )
```

**Worked example.** A meal combo priced at 25.00 euros with three choices whose base prices are
10.00, 5.00 and 5.00 euros (total 20.00). The shares are
10 ÷ 20 × 25 = 12.50, 5 ÷ 20 × 25 = 6.25 and 6.25, summing to exactly 25.00, so the residue is zero.
If the customer picks an item in the first choice that carries an extra price of 2.00, that item
line's price is 14.50 while the combo line itself stays at 0.00.

**Worked example with a residue.** Combo price 10.00 with three choices of equal base price 1.00.
Each share is round(1 ÷ 3 × 10) = round(3.3333) = 3.33; the sum is 9.99; the residue 0.01 is added
to the third choice, which becomes 3.34.

### 2.7 Discounted unit price

```formula
discounted_unit_price = price_unit × ( 1 − discount ÷ 100 )
```

Used by the catalogue screen and by the order-line update operation.

### 2.8 Gross unit price (used by revenue accrual)

```formula
p = price_unit
if discount ≠ 0 :  p = p × ( 1 − discount ÷ 100 )
if the line has taxes :
    q = ordered quantity, or 1 when it is zero
    p = ( tax_engine_total_void( p , quantity = q , currency = order currency ,
                                 rounding = global ) ) ÷ q
if line unit ≠ product reference unit :
    p = p × ( product reference unit factor ÷ line unit factor )
```

`tax_engine_total_void` is the "void" total of the tax engine — the amount excluding every tax that
is not itself part of the base. The result is therefore a price per *reference* unit, net of
discount and net of tax.

---

## 3. Line amounts

### 3.1 Base line handed to the tax engine

Each priced line is converted into a *base line* for the tax engine with these values:

| Base-line value | Source |
|---|---|
| taxes | the line's tax set |
| quantity | the ordered quantity |
| unit price | the line's unit price |
| discount | the line's discount percentage |
| partner | the order's customer |
| currency | the order currency, falling back to the company currency |
| rate | the order's stored currency rate |
| label | the line description |
| product | the line's product |
| analytic distribution | the line's analytic distribution |
| extra tax data | the line's extra tax data |
| special kind | `global_discount` when the line is a global-discount line; `down_payment` when the line is an advance-invoice line; otherwise none |

### 3.2 Line subtotal, tax and total

```formula
raw_line_base = quantity × unit_price × ( 1 − discount ÷ 100 )

price_subtotal = total_excluded_in_currency( tax engine applied to the base line )
price_total    = total_included_in_currency( tax engine applied to the base line )
price_tax      = price_total − price_subtotal
```

The tax engine is invoked in two steps: first the tax details are added to the base line, then the
details of that single base line are rounded. Consequently, when the company rounds taxes **per
line**, `price_subtotal` and `price_total` are each rounded to the order currency; when the company
rounds taxes **globally**, the per-line figures still carry the per-line rounding but the order
totals are recomputed from unrounded details (section 4.1), which is why the sum of the stored line
subtotals can differ from the stored order untaxed amount by a rounding step.

**Worked example, tax excluded.** Quantity 3, unit price 100.00, discount 10 percent, one tax of 21
percent excluded, currency with two decimals:

```
raw_line_base  = 3 × 100.00 × (1 − 10 ÷ 100) = 270.00
price_subtotal = 270.00
tax            = round(270.00 × 0.21) = 56.70
price_total    = 326.70
price_tax      = 56.70
```

**Worked example, tax included.** Quantity 2, unit price 121.00 tax included, one tax of 21 percent
included:

```
raw_line_base  = 242.00 (tax included)
price_subtotal = round(242.00 ÷ 1.21) = 200.00
price_total    = 242.00
price_tax      = 42.00
```

### 3.3 Unit prices after discount

```formula
price_reduce_taxexcl = price_subtotal ÷ quantity   ( 0 when the quantity is 0 )
price_reduce_taxinc  = price_total    ÷ quantity   ( 0 when the quantity is 0 )
```

---

## 4. Order totals

### 4.1 The three stored totals

1. Collect the *priced lines* — every line whose display type is empty.
2. Convert each into a base line (section 3.1).
3. Append the early-payment-discount base lines (section 4.2) when they apply.
4. Add the tax details to all base lines at once, then round the tax details of all base lines at
   once. Rounding "at once" is what implements the company's global rounding option: deltas are
   distributed across lines instead of being lost line by line.
5. Ask the tax engine for the totals summary in the order currency (falling back to the company
   currency) for the order's company.

```formula
amount_untaxed = base_amount_in_currency  of the totals summary
amount_tax     = tax_amount_in_currency   of the totals summary
amount_total   = total_amount_in_currency of the totals summary
```

The same five steps also produce the structured `tax_totals` value used for display, which
additionally carries the per-tax-group breakdown and the early-payment presentation.

### 4.2 Early payment discount in the mixed computation mode

When the order's payment term declares an early payment discount **and** its tax computation mode
is *mixed* **and** its discount percentage is non-zero, two extra base lines are appended **per
priced line**:

```formula
line_discount_amount = ( price_subtotal of the line ÷ 100 ) × discount_percentage
```

- First extra base line: quantity 1, unit price `− line_discount_amount`, special kind
  *early payment*, special mode *total excluded*, currency the order currency, carrying the line's
  taxes **flattened** and **restricted to taxes whose amount type is not fixed**.
- Second extra base line: quantity 1, unit price `+ line_discount_amount`, special kind
  *early payment*, special mode *total excluded*, **no taxes at all**.

The net effect on the untaxed amount is zero (the two lines cancel), while the tax base is reduced
by the discount amount — which is exactly what the *mixed* mode prescribes: tax computed on the
discounted base, total amount left undiscounted.

**Worked example.** One line with a subtotal of 1 000.00 and a 21 percent tax; payment term with a
2 percent early discount in mixed mode.

```
line_discount_amount = 1 000.00 ÷ 100 × 2 = 20.00
extra line A : price −20.00 with the 21 percent tax   → base −20.00, tax −4.20
extra line B : price +20.00 with no tax               → base +20.00, tax 0
amount_untaxed = 1 000.00 − 20.00 + 20.00 = 1 000.00
amount_tax     = 210.00 − 4.20 = 205.80
amount_total   = 1 205.80
```

### 4.3 Amount before discount

```formula
amount_undiscounted = Σ over all lines L , excluding lines whose special kind is set ,
                        of raw_total_excluded_in_currency( base line of L with discount forced to 0 )
```

"Excluding lines whose special kind is set" removes global-discount lines and advance-invoice
lines. The raw total excluded is the *unrounded* figure produced by the tax engine, so the result
is a comparison figure, not an accounting amount.

**Worked example.** Three lines: 3 × 100.00 with 10 percent discount, 1 × 250.00 with no discount,
2 × 40.00 with 25 percent discount. The undiscounted amount is 300.00 + 250.00 + 80.00 = 630.00,
while the untaxed amount is 270.00 + 250.00 + 60.00 = 580.00.

### 4.4 Un-invoiced balance and invoiced amount of the order

```formula
amount_to_invoice = Σ over lines of line_amount_to_invoice        (section 6.5)
amount_invoiced   = Σ over lines of line_amount_invoiced          (section 6.4)
```

### 4.5 Margin

Present when the margin capability is installed.

```formula
line_cost_unit = convert_to_order_currency(
                     price_of_product_cost_expressed_in_line_unit ,
                     from = product cost currency )

price_of_product_cost_expressed_in_line_unit
                = convert_price( product standard cost ,
                                 from = product reference unit , to = line unit )
```

```formula
if qty_delivered ≠ 0 and ordered quantity = 0 :
    calculated_subtotal = price_unit × qty_delivered
    line_margin         = calculated_subtotal − ( line_cost_unit × qty_delivered )
    line_margin_percent = line_margin ÷ calculated_subtotal      ( 0 when the subtotal is 0 )
else :
    line_margin         = price_subtotal − ( line_cost_unit × ordered quantity )
    line_margin_percent = line_margin ÷ price_subtotal           ( 0 when the subtotal is 0 )
```

```formula
order_margin         = Σ over lines of line_margin
order_margin_percent = order_margin ÷ amount_untaxed             ( 0 when the untaxed amount is 0 )
```

The first branch exists for lines created by a delivery or an expense, where nothing was ordered
but something was delivered.

**Worked example.** Line of 10 units at 150.00 with a 10 percent discount; product cost 90.00 per
reference unit, line unit equal to the reference unit; order currency equals cost currency.

```
price_subtotal      = 10 × 150.00 × 0.90 = 1 350.00
line_cost_unit      = 90.00
line_margin         = 1 350.00 − 900.00 = 450.00
line_margin_percent = 450.00 ÷ 1 350.00 = 0.3333 (33.33 percent)
```

---

## 5. Delivered quantity

### 5.1 Choosing the method

```formula
qty_delivered_method = 'analytic'    if the line is an expense line
                     = 'stock_move'  if the product is a goods product and the line is not an expense line
                     = 'timesheet'   if the product is a service tracked by recorded time
                     = 'manual'      otherwise
```

The stock-move value exists only with inventory integration; the timesheet value only with the
time-tracking coupling. The ordering above is the effective precedence.

### 5.2 Manual method

The stored value is whatever a user typed. The recomputation explicitly preserves it: a line is
overwritten only when its current delivered quantity is zero or when the recomputation produced a
value for it.

### 5.3 Analytic method (expenses and vendor bills)

1. Select the analytic lines pointing at the order lines whose method is *analytic*, restricted to
   those whose signed amount is not positive (an expense is a negative amount).
2. Group them by unit and by order line; for each group take the sum of the recorded quantities,
   the number of distinct source journal items, and the number of analytic lines.
3. Per group:

```formula
group_quantity = unit_amount_sum ÷ number_of_analytic_lines    if there is exactly one distinct
                                                                journal item and more than one analytic line
               = unit_amount_sum                                otherwise
```

   The division prevents counting the same journal item several times when a single item was split
   across several analytic accounts.
4. Convert the group quantity from the group's unit into the order line's unit, rounding half up.
5. Sum the converted quantities per order line.

**Worked example.** An expense of 8 hours is split 50/50 over two analytic accounts, producing two
analytic lines of 8 hours each that both point at the same journal item. The group has
`unit_amount_sum` = 16, one distinct journal item and two analytic lines, so the delivered quantity
is 16 ÷ 2 = 8 hours, not 16.

### 5.4 Stock-move method

```formula
qty_delivered = Σ over completed outgoing moves of convert( move quantity ,
                                                            from = move unit , to = line unit ,
                                                            rounding = half up )
              − Σ over completed incoming moves of convert( move quantity ,
                                                            from = move unit , to = line unit ,
                                                            rounding = half up )
```

An *outgoing* move of the line is a non-cancelled move of the same product, in the same company,
whose destination is not an inventory-adjustment location, which is not a returned drop-shipment,
whose destination (strict mode) is an outgoing location, and which is either not a return or is a
return flagged as refundable.

An *incoming* move counts only when it is flagged as refundable **and** it is a genuine return —
that is, it has an originating returned move, or it comes from an outgoing location, or the ordered
quantity is not positive, or its transfer is itself a return. This exclusion prevents a
drop-shipment receipt from reducing the delivered quantity.

**Worked example.** Ordered 10 units. A first transfer delivers 6, a second delivers 4, then the
customer returns 3 with the refund flag set. The delivered quantity is 6 + 4 − 3 = 7.

### 5.5 Delivered quantity as of a past date

When an accrual date is supplied and it is strictly earlier than today, the delivered quantity is
recomputed with the move set restricted to moves whose date, in the reader's time zone, is not
later than that accrual date. Otherwise the stored delivered quantity is reused unchanged.

---

## 6. Invoiced quantities and amounts

### 6.1 Invoiced quantity

```formula
qty_invoiced = Σ over invoice lines IL linked to the order line , where
                  IL's invoice is not cancelled, or its payment state is the externally-invoiced state (`invoicing_legacy`) :

                  + convert( IL.quantity , from = IL unit , to = line unit , round = no )
                        when IL's invoice is a customer invoice
                  − convert( IL.quantity , from = IL unit , to = line unit , round = no )
                        when IL's invoice is a customer credit note
```

Draft invoices count. Cancelled invoices do not. The conversion is unrounded. Deliberately, a
credit note that was **not** created from the order still reduces the invoiced quantity — the
protection against automatic re-invoicing lies in the *final* switch of the invoicing algorithm,
not here.

### 6.2 Invoiced quantity restricted to posted invoices

```formula
qty_invoiced_posted = Σ over invoice lines IL whose invoice is posted, or carries the externally-invoiced state :
                        convert( IL.quantity , from = IL unit , to = line unit , round = yes )
                        × ( − direction_sign( IL's invoice ) )
```

The direction sign is −1 for a customer invoice and +1 for a customer credit note, so the product
of the two signs makes invoices add and credit notes subtract. Unlike the previous formula, the
unit conversion is rounded here.

### 6.3 Untaxed invoiced amount

```formula
untaxed_amount_invoiced = Σ over invoice lines IL whose invoice is posted, or carries the externally-invoiced state :
                              + convert_currency( IL.price_subtotal , to = order currency ,
                                                  on = invoice date or today )
                                    when IL's invoice is a customer invoice
                              − convert_currency( IL.price_subtotal , to = order currency ,
                                                  on = invoice date or today )
                                    when IL's invoice is a customer credit note
```

### 6.4 Invoiced amount, tax included

```formula
amount_invoiced = Σ over invoice lines IL whose invoice is posted, or carries the externally-invoiced state :
                      convert_currency( IL.price_total , to = order currency ,
                                        on = invoice date or today )
                      × ( − direction_sign( IL's invoice ) )
```

### 6.5 Quantity to invoice

```formula
if the order status is not 'sale' , or the line is a display line :
    qty_to_invoice = 0
else if the product's invoicing policy is 'ordered quantities' :
    qty_to_invoice = ordered_quantity − qty_invoiced
else :
    qty_to_invoice = qty_delivered − qty_invoiced
```

**Combo lines.** A line whose product is a combo is handled after all its item lines:

```formula
if any item line of the combo has a non-zero quantity to invoice :
    combo_line.qty_to_invoice = combo_line.ordered_quantity − combo_line.qty_invoiced
else :
    combo_line.qty_to_invoice = 0
```

**Worked example, ordered quantities.** Ordered 10, invoiced 4 → 6 to invoice.
**Worked example, delivered quantities.** Ordered 10, delivered 6, invoiced 4 → 2 to invoice.
**Worked example, negative.** Ordered 10, delivered 6, invoiced 8 → −2 to invoice; the line is
"to invoice" and a *final* invoicing run will turn it into a credit note.

### 6.6 Untaxed amount to invoice

```formula
quantity_to_consider = qty_delivered      if the product's invoicing policy is 'delivered quantities'
                     = ordered_quantity   otherwise

price_reduce  = price_unit × ( 1 − discount ÷ 100 )
price_subtotal_reference = price_reduce × quantity_to_consider

if any tax of the line includes tax in the price :
    price_subtotal_reference = total_excluded( tax engine applied to price_reduce ,
                                               quantity = quantity_to_consider ,
                                               currency = order currency ,
                                               product = line product ,
                                               partner = order delivery address )
```

Then, comparing with the invoice lines already produced:

```formula
if any linked invoice line carries a discount different from the line's discount :
    already = Σ over linked invoice lines IL of
                    ( total_excluded( tax engine applied to convert(IL.price_unit) ) × IL.quantity )
                        when IL has a tax that includes tax in the price
                    ( convert(IL.price_unit) × IL.quantity )
                        otherwise
    untaxed_amount_to_invoice = max( price_subtotal_reference − already , 0 )
else :
    untaxed_amount_to_invoice = price_subtotal_reference − untaxed_amount_invoiced
```

`convert(IL.price_unit)` converts the invoice line's unit price into the order currency at the
invoice line's date (today when it has none), unrounded. The value is zero whenever the order
status is not `sale`. Draft invoices are deliberately ignored, so the figure always describes what
the order still owes, not what a draft happens to contain.

The deliberate use of the raw price multiplication instead of the stored line subtotal matters for
expense lines: such a line can have an ordered quantity of zero and a delivered quantity of four,
which would make the stored subtotal zero even though there is something to invoice.

**Worked example.** Line of 10 units at 150.00 with a 10 percent discount, invoicing policy
*ordered*, one posted invoice for 4 units at the same discount.

```
price_reduce               = 150.00 × 0.90 = 135.00
price_subtotal_reference   = 135.00 × 10 = 1 350.00
untaxed_amount_invoiced    = 4 × 135.00 = 540.00
untaxed_amount_to_invoice  = 1 350.00 − 540.00 = 810.00
```

### 6.7 Un-invoiced balance of a line, tax included

```formula
if ordered_quantity ≠ 0 :
    quantity_to_consider = qty_delivered      if the invoicing policy is 'delivered quantities'
                         = ordered_quantity   otherwise
    quantity_open        = quantity_to_consider − qty_invoiced_posted
    unit_price_total     = price_total ÷ ordered_quantity
    amount_to_invoice    = unit_price_total × quantity_open
else :
    amount_to_invoice    = 0
```

**Worked example.** Ordered 10, delivered 6, posted invoices cover 4, total including tax 1 633.50
for the whole line. The unit total is 163.35; the open quantity is 2; the un-invoiced balance is
326.70.

### 6.8 Amount to invoice at a past date (revenue accrual)

```formula
quantity_open_at_date = convert( qty_delivered_at_date − qty_invoiced_at_date ,
                                 from = line unit , to = product reference unit )

amount_to_invoice_at_date = quantity_open_at_date × gross_unit_price      (section 2.8)
```

---

## 7. Advance invoices (down payments)

### 7.1 What the wizard computes

The advance-invoice amount is not a simple percentage of the total: it must be split per tax so
that the advance invoice carries exactly the right proportion of each tax, and the resulting tax
amounts must add up exactly to the requested figure. The algorithm is the *reduce to target
amount* routine of the tax engine, invoked with the order's priced lines.

### 7.2 Preparation

1. Take every line of the order whose display type is empty.
2. Convert each into a base line; add the tax details; round the tax details of the whole set.
3. Split out of each base line every tax that may not be discounted, folding the corresponding
   amount into the base of a new sub-line. What remains is a set of base lines whose taxes can all
   be proportionally reduced.

### 7.3 Turning the request into a percentage

```formula
sign            = −1 if the requested amount is negative, else +1
signed_amount   = sign × requested amount

if the method is 'fixed' :
    percentage                        = signed_amount ÷ total_amount_in_currency     ( 0 if that total is 0 )
    expected_total_in_currency        = round_to_currency( requested amount )
    expected_total_in_company_currency= round_to_company_currency( expected_total_in_currency ÷ rate )
else  (the method is 'percentage', the amount is expressed in percent) :
    percentage                        = signed_amount ÷ 100
    expected_total_in_currency        = round_to_currency( total_amount_in_currency × sign × percentage )
    expected_total_in_company_currency= round_to_company_currency( total_amount × sign × percentage )
```

`total_amount_in_currency` is the sum, over the prepared base lines, of the total excluded plus the
tax amount — that is, the tax-inclusive total of the order.

### 7.4 Expected amounts per tax

For every tax found on the prepared base lines:

```formula
expected_tax_amount(t)  = round_to_currency( current_tax_amount(t)  × sign × percentage )
expected_base_amount(t) = round_to_currency( current_base_amount(t) × sign × percentage )
```

and the residual base:

```formula
expected_base_total = expected_total_in_currency − Σ over t of expected_tax_amount(t)
```

The same two figures are computed in company currency using the company's rounding.

### 7.5 Producing the advance base lines

1. Reduce the prepared base lines to as few lines as possible by grouping them — by default by tax
   set; the invoicing wizard keeps that default, the global-discount wizard groups everything onto
   the company's discount product.
2. For each reduced base line, create a new base line whose unit price is
   `original unit price × sign × percentage` and whose computation key is
   `down_payment,<wizard identifier>`.
3. Add and round the tax details of the new base lines.
4. **Smooth the residues.** Sort the new base lines with the non-special ones first and, within
   each group, by decreasing total excluded. For each tax, compute the difference between the
   expected tax amount and the tax amount actually obtained, and the same for the base amount; push
   those differences, one currency unit at a time, onto the sorted lines until they are absorbed.
   The identical operation is repeated in company currency.
5. Correct the tax details of any line that carries a manual tax amount.

The result is a small set of base lines whose tax-inclusive total equals the requested amount to
the last minor unit, and whose per-tax amounts are exactly the requested proportion of the order's
per-tax amounts.

### 7.6 From base lines to order lines

Each produced base line becomes one new order line on the order with:

| Order-line field | Value |
|---|---|
| advance flag | true |
| ordered quantity | 0.0 |
| unit price | the base line's unit price (the advance amount for that tax group) |
| taxes | the base line's taxes |
| analytic distribution | the base line's analytic distribution |
| extra tax data | the exported extra tax data of the base line, which carries the manual tax amounts and the computation key |
| sequence | the highest sequence found on the order (or 10 when there is none) plus one, plus the index of the line |

A section line flagged as an advance line is created first, if there is not already one, with the
same sequence rule. The lines are created with the "do not log new lines" instruction so that no
"Extra line with …" note is posted.

### 7.7 From order lines to the advance invoice

The invoice is prepared exactly as an ordinary invoice header (section 8.1) and its lines are
produced from the advance order lines with:

| Invoice-line field | Value |
|---|---|
| label | "Down payment of *percentage*%" for the percentage method, where the percentage is formatted with the reader's number format; "Down Payment" for the fixed method |
| quantity | 1.0 |
| unit price | the advance order line's unit price |
| account | the first non-empty of: the company's advance-invoice account mapped through the order's fiscal position; the account carried by the base line; the product's advance-invoice account; the product's income account |
| everything else | as in the ordinary invoice-line mapping (section 8.2) |

**Worked example — thirty percent advance.** An order with two lines: 1 000.00 with a 21 percent
tax and 500.00 with a 6 percent tax. Totals: base 1 500.00, tax 210.00 + 30.00 = 240.00, total
1 740.00.

```
percentage                 = 30 ÷ 100 = 0.30
expected_total_in_currency = round( 1 740.00 × 0.30 ) = 522.00
expected_tax_amount(21 %)  = round( 210.00 × 0.30 ) = 63.00
expected_tax_amount(6 %)   = round(  30.00 × 0.30 ) =  9.00
expected_base_total        = 522.00 − 63.00 − 9.00 = 450.00
```

Two advance order lines are created: one of 300.00 with the 21 percent tax, one of 150.00 with the
6 percent tax. The advance invoice carries 300.00 + 63.00 and 150.00 + 9.00, totalling 522.00.

### 7.8 Deducting advances on the final invoice

When an invoice is created with the *final* switch on, every advance order line whose quantity to
invoice is non-zero — including a negative one — is added to the invoice with:

```formula
invoice_line_quantity = −1.0
invoice_line_extra_tax_data = reverse_quantity( extra tax data of the advance order line )
```

The unit price is unchanged, so the line lands on the invoice as a negative amount of exactly the
advance already invoiced, tax by tax. Reversing the extra tax data negates the manual tax amounts
stored on the advance line, so the deducted tax matches the tax of the advance invoice to the
minor unit.

A section line labelled "Down Payments" is inserted before the first advance line of the invoice.

**Worked example — final invoice after a thirty percent advance.** Continuing the previous example,
the customer is now invoiced in full.

```
ordinary line 1 : 1 000.00 base, 210.00 tax
ordinary line 2 :   500.00 base,  30.00 tax
advance line 1  :  −300.00 base, −63.00 tax
advance line 2  :  −150.00 base,  −9.00 tax
invoice base    = 1 500.00 − 450.00 = 1 050.00
invoice tax     =   240.00 −  72.00 =   168.00
invoice total   = 1 218.00
```

which is exactly 1 740.00 − 522.00.

### 7.9 Advance amount already invoiced

```formula
downpayment_price_unit_already_posted =
    Σ over invoice lines IL of the advance order line , where IL's invoice is posted
      and IL's invoice is not among the invoices being created :
        + IL.price_unit   when IL's invoice is a customer invoice
        − IL.price_unit   when IL's invoice is a customer credit note
```

This lets an advance that was itself partly credited be deducted at its net value.

---

## 8. Invoice preparation

### 8.1 Invoice header mapping

Every value of the created customer invoice, field by field. The left column is the invoice field;
the right column is where the value comes from on the order.

| Invoice field (storage name) | Value taken from the order |
|---|---|
| Document type (`move_type`) | the constant `out_invoice` (customer invoice) |
| Reference (`ref`) | the customer reference, falling back to the order reference |
| Narration (`narration`) | the order's terms and conditions text |
| Currency (`currency_id`) | the order currency |
| Campaign (`campaign_id`) | the order's campaign |
| Medium (`medium_id`) | the order's medium |
| Source (`source_id`) | the order's source |
| Sales Team (`team_id`) | the order's team |
| Customer (`partner_id`) | the order's **invoice address** |
| Delivery Address (`partner_shipping_id`) | the order's delivery address |
| Fiscal Position (`fiscal_position_id`) | the order's fiscal position; when the order has none, the fiscal position that the mapping engine would pick for the invoice address |
| Source Document (`invoice_origin`) | the order reference |
| Payment Terms (`invoice_payment_term_id`) | the order's payment term |
| Payment Method (`preferred_payment_method_line_id`) | the order's preferred payment method line |
| Salesperson on the invoice (`invoice_user_id`) | the order's salesperson |
| Responsible (`user_id`) | the order's salesperson |
| Payment Reference (`payment_reference`) | the order's payment reference |
| Transactions (`transaction_ids`) | the transactions of the order that are *pending*, *authorized*, or *done with an unreconciled payment* |
| Company (`company_id`) | the order's company |
| Invoice lines (`invoice_line_ids`) | built by section 8.2 |
| Journal (`journal_id`) | the order's invoicing journal — **only when the order carries one**; otherwise the field is omitted and the accounting domain picks the sale journal with the lowest sequence |
| Incoterm (`invoice_incoterm_id`) | the order's incoterm (with inventory integration) |
| Delivery Date (`delivery_date`) | the order's effective date, converted to the reader's time zone (with inventory integration) |

### 8.2 Invoice line mapping

For an ordinary (non-combo) order line:

| Invoice-line field (storage name) | Value taken from the order line |
|---|---|
| Display type (`display_type`) | the order line's display type, or the constant `product` when it is empty |
| Sequence (`sequence`) | the order line's sequence, overridden by the running sequence counter of the invoicing run |
| Label (`name`) | the "full journal item name" built from the order line description and the product display name |
| Product (`product_id`) | the order line's product |
| Unit (`product_uom_id`) | the order line's unit |
| Quantity (`quantity`) | the order line's **quantity to invoice**, overridden by −1 for an advance line on a final invoice and by 1 on an advance invoice |
| Discount (`discount`) | the order line's discount percentage |
| Unit price (`price_unit`) | the order line's unit price |
| Taxes (`tax_ids`) | the order line's tax set |
| Linked order lines (`sale_line_ids`) | a link to this order line |
| Advance flag (`is_downpayment`) | the order line's advance flag |
| Extra tax data (`extra_tax_data`) | the order line's extra tax data, reversed in quantity for an advance line on a final invoice |
| Collapse prices (`collapse_prices`) | the order line's flag |
| Collapse composition (`collapse_composition`) | the order line's flag |
| Account (`account_id`) | normally left unset so the accounting domain derives it from the product and the fiscal position; forced to the account of the first existing advance invoice line when the line is an advance line that already has one; forced to empty for a display line |
| Analytic distribution (`analytic_distribution`) | the order line's analytic distribution, only when the line is not a display line |

For a line whose product is a combo, the invoice line is a **section** instead:

| Invoice-line field | Value |
|---|---|
| Display type | `line_section` |
| Sequence | the order line's sequence |
| Label | "*product name* x *quantity to invoice*", the quantity written without decimals when it is a whole number |
| Unit | the order line's unit |
| Quantity | the order line's quantity to invoice |
| Linked order lines | a link to this order line |
| Collapse prices / collapse composition | the order line's flags |

### 8.3 Re-sequencing when orders are grouped

If the number of created invoices is smaller than the number of orders — which means at least one
grouping occurred — every invoice's lines are re-numbered from 1 upwards in their current order, so
that sections of different orders do not interleave. The new number is the running counter; a
coupling may override the rule by returning the old number instead.

**Worked example.** Order one has a section at sequence 10 and a product at 11; order two has a
section at 10 and a product at 11. Grouped naïvely the invoice would read section one, section two,
product one, product two. After re-sequencing it reads section one (1), product one (2), section
two (3), product two (4).

---

## 9. Global discount wizard

### 9.1 On all order lines

```formula
for every line of the order :  discount = discount_percentage × 100
```

The wizard stores the percentage as a fraction, so 0.1 becomes a 10 percent discount on every line,
including display lines (writing a discount on a display line is harmless because the database
check only constrains product, price, quantity, unit and lead time).

### 9.2 Global discount and fixed amount

Both use the *reduce to target amount* routine of section 7.3 through 7.5, with:

- amount kind *percentage* and amount = `discount_percentage × 100` for a global discount;
- amount kind *fixed* and amount = the entered amount for a fixed discount;
- the amount negated, because a discount reduces the order;
- a grouping function that forces every produced line onto the company's discount product;
- the computation key `global_discount,<wizard identifier>`.

Each produced base line becomes one new order line at sequence 999 with:

| Field | Value |
|---|---|
| description | see below |
| product | the company's discount product |
| unit price | the base line's unit price (a negative amount) |
| technical unit price | 0 |
| quantity | the base line's quantity |
| taxes | the base line's taxes |
| extra tax data | the exported extra tax data of the base line |

The description depends on whether the order has more than one distinct non-empty tax set:

| Several tax sets? | Global discount | Fixed amount |
|---|---|---|
| no | "Discount *percentage*%" | "Discount" |
| yes | "Discount *percentage*%- On products with the following taxes *tax names*" | "Discount- On products with the following taxes *tax names*" |

The percentage is written with the `Discount` precision.

**Worked example.** An order with 1 000.00 at 21 percent and 500.00 at 6 percent; a global discount
of 10 percent is applied. Two discount lines are created: −100.00 with the 21 percent tax and
−50.00 with the 6 percent tax, so the order base falls to 1 350.00 and the tax to 189.00 + 27.00 =
216.00.

### 9.3 Creating the discount product on demand

If the company has no discount product, one is created with: name "Discount", type *service*,
invoicing policy *ordered quantities*, list price 0.00, the order's company, no taxes and the
shipped "Services" product category when it exists. Creation is attempted only when the reader may
create products, may write on the company and may write that particular company field; otherwise
the operation fails with the message quoted in [business-rules.md](business-rules.md).

---

## 10. Sales analysis measures

All monetary measures of the analysis entity are converted from the order currency into the
presentation currency of the reading company:

```formula
converted_amount = amount ÷ order_currency_rate × presentation_rate
```

where a rate that is absent or zero is replaced by one. All quantity measures are converted from
the line unit into the product's reference unit:

```formula
converted_quantity = quantity × line_unit_factor ÷ product_reference_unit_factor
```

The discount measure is:

```formula
discount_amount = Σ over grouped lines of ( price_unit × ordered_quantity × discount ÷ 100 )
                  converted as above
```

The unit price measure is an **average**, not a sum; the discount percentage measure is likewise an
average.

---

## 11. Product and team aggregates

### 11.1 Quantity sold on a product

```formula
sales_count( product ) = round_to_product_unit(
        Σ of the analysis measure "quantity ordered"
          over rows whose status is 'sale' ,
             whose product is this product ,
             and whose order date is not earlier than today minus 365 days )
```

The rounding uses the product's reference unit. Readers who are not salespeople always see zero.

**Worked example.** Today is 11 September 2026. Orders confirmed on 1 October 2025 (5 units) and 1
March 2026 (7 units) count; an order confirmed on 1 August 2025 does not, because it is more than
365 days old. The quantity sold is 12.

### 11.2 Invoiced this month on a team

```formula
invoiced( team ) = Σ of the signed untaxed amount, in company currency,
                     of every posted customer invoice, customer credit note or customer receipt
                     whose team is this team,
                     whose payment state is 'in payment', 'paid' or 'reversed',
                     and whose accounting date lies between the first day of the current month
                     and today, both inclusive
```

Because the amount is *signed*, credit notes reduce the figure.

---

## 12. Revenue accrual amount

The period-end accrual for confirmed but uninvoiced sales is computed per order line, with the
accrual date supplied by the operator (default: the last day of the previous month).

1. Keep the lines that are not display lines, are not advance lines, belong to the selected orders,
   and whose amount to invoice at the accrual date is non-zero at the line unit's rounding.
2. For each such line:

```formula
open_quantity = qty_delivered_at_date − qty_invoiced_at_date

if open_quantity > 0 :        (delivered, not yet invoiced)
    amount_in_order_currency = amount_to_invoice_at_date            (section 6.8)
else if open_quantity < 0 :   (invoiced, not yet delivered)
    walk the posted invoice lines of the order line in descending order, accumulating
        amount_in_order_currency = amount_in_order_currency − invoice_line.price_subtotal
        processed_quantity       = processed_quantity      + invoice_line.quantity
    until processed_quantity ≥ |open_quantity|
```

3. Convert the amount into the company currency at the wizard's date.
4. The journal item's label reads: "*order reference* - *first twenty characters of the line
   description, truncated with an ellipsis*; *invoiced quantity at date* Invoiced, *delivered
   quantity at date* Delivered at *unit price* each".

The counterpart amount is the negation of the sum of all the line amounts.

**Worked example.** A line of 10 units at 100.00, of which 6 were delivered and 4 invoiced before
30 June. The open quantity is 2, the gross unit price is 100.00, so the accrued revenue is 200.00,
credited to the income account with a debit of 200.00 on the accrual account.

---

## 13. Duplicate order detection

Two orders of the same company are duplicates of each other when all of these hold:

1. they are different records;
2. neither of the two candidates is cancelled (the *other* order must not be cancelled);
3. they share the same customer;
4. **either** the first order's source document equals the second order's reference, **or** the two
   orders share the same customer reference.

The detection only runs for orders in the `draft` status that have an identifier and a non-empty
customer reference.

---

## 14. Rounding summary

| Quantity | Precision used | Where |
|---|---|---|
| Ordered, delivered, invoiced and to-invoice quantities | decimal precision `Product Unit` | comparisons and zero tests in the invoice-status and invoicing algorithms |
| Unit prices | decimal precision `Product Price` as a *minimum display* precision; stored unrounded | line unit price, margin cost |
| Discount percentages | decimal precision `Discount` | stored discount, analysis measure, discount-line descriptions |
| Line subtotals and totals | the order currency's rounding | stored line amounts |
| Order totals | the order currency's rounding, applied by the tax engine over the whole set of base lines | stored order amounts |
| Advance amounts | the order currency for the requested figure, the company currency for its counterpart | advance-invoice preparation |
| Currency conversions inside the domain | unrounded when feeding another formula; rounded to the target currency when stored | delivered-quantity amounts, analysis measures |
| Unit conversions for delivered quantities | rounded half up | stock-move and analytic methods |
| Unit conversions for invoiced quantities | unrounded for the running total, rounded for the posted-only total | invoiced-quantity formulas |
