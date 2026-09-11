# Purchasing — Calculations

Every formula and algorithm of the purchasing domain, with its rounding rule, its precision,
its currency handling, its unit handling, its evaluation order and at least one worked numeric
example.

## 0. Conventions

### 0.1 Named precisions

The platform keeps a small set of named decimal precisions, each a number of decimal digits
that an administrator may change. Purchasing uses three of them:

| Precision name | Shipped value | Used for |
|---|---|---|
| Product Unit | 2 digits | Every quantity: ordered, received, billed, to bill. |
| Product Price | 2 digits | The unit price handed to a stock move, and the comparison that decides whether a price difference exists. |
| Discount | 2 digits | The discount percentage on a line. |

A **currency rounding** is separate: each currency carries its own rounding step (for most
currencies 0.01) and its own number of decimal places. "Round to the currency" below means
round half away from zero to the nearest multiple of that step.

### 0.2 Rounding vocabulary

- **round to currency(x)** — round *x* half away from zero to the nearest multiple of the
  currency's rounding step.
- **round to precision(x, d)** — round *x* half away from zero to *d* decimal digits.
- **round up(x, d)** — round *x* away from zero to *d* decimal digits, never downwards.
- **is zero at precision(x, d)** — true when |*x*| is smaller than half of 10 to the power
  −*d*, that is, when *x* rounds to 0.
- **compare at precision(a, b, d)** — round both to *d* digits and return −1, 0 or 1.
- **convert quantity(q, from unit, to unit)** and **convert price(p, from unit, to unit)** —
  the unit-of-measure conversions specified in
  [`../units-of-measure-and-packaging/calculations.md`](../units-of-measure-and-packaging/calculations.md).
  A quantity conversion rounds to the target unit's own rounding unless a different rounding
  method is named; "rounding half up" below names the half-away-from-zero method explicitly.

### 0.3 Currency conversion

```formula
converted_amount = round_to_currency( amount × ( rate(to_currency, company, date) ÷ rate(from_currency, company, date) ) )
```

Several purchasing computations ask for conversion **without** rounding; those are marked
explicitly. The rate lookup itself is specified in
[`../multi-currency/calculations.md`](../multi-currency/calculations.md).

---

## 1. Line amounts

### 1.1 The base line handed to the tax engine

Each non-display order line is turned into a base line for the tax engine carrying:

| Quantity handed over | Value |
|---|---|
| quantity | the line's ordered quantity |
| unit price | the line's unit price |
| discount percentage | the line's discount |
| taxes | the line's taxes |
| partner | the order's vendor |
| currency | the order's currency, or the company currency when the order has none |
| rate | the order's currency rate |
| label | the line's description |

### 1.2 Subtotal, total and tax

The tax engine returns, per line, a total excluding tax and a total including tax in the
order currency. The line stores:

```formula
price_subtotal = total_excluded_in_order_currency
price_total    = total_included_in_order_currency
price_tax      = price_total − price_subtotal
```

Before rounding, the engine computes the untaxed base of a line as:

```formula
line_base = quantity × unit_price × ( 1 − discount_percentage ÷ 100 )
```

and then applies the taxes. Whether the rounding happens per line or once for the whole
document is decided by the company's tax rounding method; both variants and the exact
ordering are specified in [`../taxes/calculations.md`](../taxes/calculations.md). The purchase
domain's only contribution is the list of base lines and the currency.

### 1.3 Header amounts

```formula
amount_untaxed  = base_amount_in_order_currency      (from the tax engine, over all non-display lines)
amount_tax      = tax_amount_in_order_currency       (from the tax engine)
amount_total    = total_amount_in_order_currency     (from the tax engine)
amount_total_cc = total_amount_in_company_currency   (from the tax engine)
```

The engine is given the whole set of base lines at once, so that a company using global tax
rounding rounds the document's tax once rather than line by line.

### 1.4 Worked example — two lines, per-line rounding, 21 % tax

| Line | Quantity | Unit price | Discount | Tax |
|---|---|---|---|---|
| 1 | 7 | 13.33 | 0 % | 21 % not included |
| 2 | 3 | 49.99 | 10 % | 21 % not included |

Line 1: base = 7 × 13.33 × (1 − 0) = 93.31 → subtotal 93.31; tax = round to currency(93.31 ×
0.21) = round to currency(19.5951) = 19.60; total 112.91.

Line 2: base = 3 × 49.99 × (1 − 0.10) = 3 × 44.991 = 134.973 → subtotal, rounded to the
currency, 134.97; tax = round to currency(134.97 × 0.21) = round to currency(28.3437) = 28.34;
total 163.31.

Header: untaxed 93.31 + 134.97 = 228.28; tax 19.60 + 28.34 = 47.94; total 276.22.

Under **global** rounding the tax would instead be round to currency((93.31 + 134.973) × 0.21)
= round to currency(228.283 × 0.21) = round to currency(47.93943) = 47.94 — the same here, but
the two methods do diverge on other figures, which is why the method is a company setting.

### 1.5 Discounted unit price and unit-price restatements

```formula
price_unit_discounted = price_unit × ( 1 − discount ÷ 100 )
```

```formula
price_unit_product_uom = convert_price( price_unit , line_unit , product_reference_unit )
```

The second is zero for display lines and for down-payment lines.

### 1.6 The company-currency subtotal used to compare alternatives

```formula
price_total_cc = price_subtotal ÷ currency_rate
```

where the currency rate is the order's rate from the company currency to the order currency.
Dividing therefore brings the amount back to the company currency. Worked example: an offer of
11.00 per unit for 500 units in a foreign currency whose order rate is 1.1458333… gives a
subtotal of 5 500.00 in that currency and a company subtotal of 5 500.00 ÷ 1.1458333… =
4 800.00.

---

## 2. The order currency rate

```formula
currency_rate = conversion_rate( from = company_currency , to = order_currency , company = order_company , date = date_part( date_order or now ) )
```

It is stored with full precision (no rounding) and recomputed whenever the currency, the order
deadline or the company changes.

---

## 3. The approval threshold

An order may be approved by the acting user when:

```formula
approval_allowed =
        ( company.po_double_validation = "one_step" )
     OR ( company.po_double_validation = "two_step"
          AND amount_total < convert( company.po_double_validation_amount ,
                                      from = active_company.currency ,
                                      to   = order.currency ,
                                      company = order.company ,
                                      date = date_part( date_order ) or today ) )
     OR user_has_privilege( purchase_administrator )
```

The comparison is a strict "less than": an order whose total is **exactly** the threshold
requires the second approval.

### 3.1 Worked example — an approval required above five thousand, in a foreign currency

The company currency is the euro; the double-validation amount is 5 000.00; the order is priced
in United States dollars; the rate on the order deadline is 1 euro = 1.0800 dollars.

```
threshold_in_order_currency = 5 000.00 × 1.0800 = 5 400.00
```

- An order totalling 5 399.99 dollars: 5 399.99 < 5 400.00 → straight to Purchase Order.
- An order totalling 5 400.00 dollars: not strictly below → stops at To Approve.
- An order totalling 5 400.01 dollars: → stops at To Approve.

A purchase administrator approves any of them regardless.

---

## 4. Expected arrival of a line

```formula
date_planned = ( date_order or now ) + ( selected_vendor_price.delay or 0 ) days
```

The addition is a calendar-day addition that preserves the time of day; no working-calendar is
consulted. The header's expected arrival is then:

```formula
order.date_planned = min over non-display lines that have one ( line.date_planned )
```

and is empty when no line qualifies.

**Worked example.** Order deadline the 3rd at 09:15, two lines whose selected vendor prices
have lead times of 4 and 10 days. Line arrivals are the 7th at 09:15 and the 13th at 09:15; the
header arrival is the 7th at 09:15.

### 4.1 The portal's noon conversion

When a vendor types a new arrival **date** on the portal, it is turned into a timestamp as:

```formula
new_timestamp = to_utc( localise( combine( typed_date , 12:00:00 ) , order_timezone ) )
```

where the order time zone is the buyer's, failing that the company partner's, failing that
coordinated universal time. Midday is used so that the stored timestamp lands on the intended
calendar day in every reasonable time zone.

**Worked example.** The buyer's time zone is 5 hours behind coordinated universal time; the
vendor types the 14th. The stored value is the 14th at 17:00 coordinated universal time.

---

## 5. Received quantity

### 5.1 From stock moves

Let *L* be a line whose method is stock moves, *u* its unit, and *M* the set of its done moves
whose product equals the line product (and, when an accrual date *a* is supplied, whose date is
on or before *a*).

```formula
qty_received = Σ over m in M of contribution(m)
```

```formula
contribution(m) =
   − convert_quantity( m.quantity , m.unit , u , rounding = half_up )
        when m is a purchase return AND ( m has no original move OR m.to_refund )
     0  when m is a purchase return otherwise
     0  when m returns a dropshipped move and m is not itself a dropship return
     0  when m returns a purchase return and not m.to_refund
     0  when m must not be counted for received quantities
   + convert_quantity( m.quantity , m.unit , u , rounding = half_up )   otherwise
```

The tests are applied in that order; the first that matches wins.

**Worked example.** A line orders 10 boxes of a product whose reference unit is "Units" and
where 1 box = 12 units. Three done moves exist, all expressed in units:

| Move | Quantity | Nature |
|---|---|---|
| A | 72 units | ordinary incoming |
| B | 24 units | ordinary incoming (backorder) |
| C | 12 units | return to the vendor, flagged to be refunded |

Contributions: A → +72 ÷ 12 = +6 boxes; B → +24 ÷ 12 = +2 boxes; C → −12 ÷ 12 = −1 box.
Received quantity = 6 + 2 − 1 = 7 boxes. Had move C not been flagged to be refunded **and**
had it carried an original move, it would have contributed 0 and the received quantity would
have stayed at 8 boxes.

### 5.2 Manual

```formula
qty_received = qty_received_manual (or 0)
```

Writing the received quantity writes the manual field when the method is manual, and forces the
manual field to zero otherwise.

### 5.3 Kits

For a line whose product resolves to a kit structure in the line's company, the received
quantity is the number of **complete kits** obtainable from the component moves:

1. Let *ordered in kit units* = convert quantity(line total quantity, line unit, kit unit).
2. Explode the kit for that quantity: for each component *c*, the required quantity per kit is
   *r(c)*.
3. From the line's done moves that are not inventory adjustments, compute for each component
   *c* the net received quantity *n(c)* = incoming quantities (counting a return of an incoming
   move only when it is flagged to be refunded) minus outgoing quantities that are flagged to be
   refunded, all expressed in the component's reference unit.
4. ```formula
   qty_received_in_kits = min over components c of ( n(c) ÷ r(c) )
   ```
5. Convert that back into the line unit.

**Worked example.** A kit "Desk Set" is one desk plus two drawers. A line orders 5 kits. The
receipt records 5 desks and 8 drawers. Then *n(desk) ÷ r(desk)* = 5 ÷ 1 = 5 and
*n(drawer) ÷ r(drawer)* = 8 ÷ 2 = 4, so the received quantity is min(5, 4) = **4 kits**. The
fifth kit becomes receivable only when two more drawers arrive.

---

## 6. Billed quantity and quantity to bill

### 6.1 Billed quantity

```formula
qty_invoiced = Σ over bill lines b of sign(b) × convert_quantity( b.quantity , b.unit , line_unit )
```

```formula
sign(b) = +1  when b's document type is a vendor bill
          −1  when b's document type is a vendor refund
```

A bill line whose document is cancelled is skipped, unless the document's payment state marks
it as historically invoiced.

### 6.2 Quantity to bill

```formula
qty_to_invoice =
    0                              when order.state ≠ "purchase"
    product_qty − qty_invoiced     when product.purchase_method = "purchase"
    qty_received − qty_invoiced    when product.purchase_method = "receive"
```

### 6.3 Worked example — the three situations side by side

A line orders 10 units at 60.00.

| Event | Received | Billed | To bill, policy *ordered* | To bill, policy *received* |
|---|---|---|---|---|
| after confirmation | 0 | 0 | 10 | 0 |
| 6 received | 6 | 0 | 10 | 6 |
| 6 billed | 6 | 6 | 4 | 0 |
| 4 more received | 10 | 6 | 4 | 4 |
| 4 more billed | 10 | 10 | 0 | 0 |
| 2 returned and refundable | 8 | 10 | 0 | −2 |

Only the last row differs in sign, and only under the received-quantity policy; that negative
value is what produces a vendor refund.

---

## 7. Amount still to bill at a date

Used by the accrued-expense computation.

```formula
qty_to_invoice_at_date =
    product_qty          − qty_invoiced_at_date   when purchase_method = "purchase"
    qty_received_at_date − qty_invoiced_at_date   when purchase_method = "receive"
```

```formula
amount_to_invoice_at_date = convert_quantity( qty_to_invoice_at_date , line_unit , product_reference_unit ) × gross_unit_price
```

where the **gross unit price** is built as follows, in this order:

1. start from the line's unit price;
2. if a discount exists, multiply by (1 − discount ÷ 100);
3. if taxes exist, let *q* be the ordered quantity or 1 when it is zero, compute the taxes on
   that price for quantity *q* in the order currency with **global** rounding, take the
   resulting amount that excludes every tax including those that are not deductible, and divide
   it by *q*;
4. if the line unit differs from the product reference unit, multiply by (product reference unit
   factor ÷ line unit factor).

```formula
gross_price_unit = ( total_void( price_unit × (1 − discount ÷ 100) , taxes , q ) ÷ q ) × ( product_unit_factor ÷ line_unit_factor )
```

**Worked example.** A line orders 10 boxes (1 box = 12 units) at 120.00 per box with a 5 %
discount and a fully recoverable 20 % tax that is not included in the price. Received 10,
billed 4, policy "on received quantities".

- quantity to bill at date = 10 − 4 = 6 boxes;
- in the product unit: 6 × 12 = 72 units;
- discounted price = 120.00 × 0.95 = 114.00 per box;
- the tax is not included, so the amount excluding every tax is 114.00 per box;
- per product unit: 114.00 × (1 ÷ 12) = 9.50 — note the factors are those of the two units
  relative to the same reference, so the multiplication by *product unit factor ÷ line unit
  factor* performs the box-to-unit restatement;
- amount still to bill = 72 × 9.50 = **684.00**.

### 7.1 The "at date" variants

When the caller supplies an accrual date **in the past**, the received and billed quantities
are recomputed from the restricted sets described in sections 5 and 6; when the supplied date
is today or in the future, or when no date is supplied, the stored values are used unchanged.

---

## 8. The unit price handed to a stock move

This is the value inventory uses to value an incoming move before any bill exists.

1. Start from the line's **discounted** unit price.
2. If the line has taxes: let *q* = the ordered quantity, or 1 when it is zero. Compute the
   taxes on that price for quantity *q*, in the order currency, for this product and this
   vendor, with **global** rounding. Take the amount that excludes every tax, including
   non-deductible ones, and divide by *q*. (For fully recoverable taxes this leaves the price
   unchanged; for a non-recoverable tax it adds the tax into the cost.)
3. If the line unit differs from the product reference unit, divide by the line unit factor and
   multiply by the product unit factor.
4. If the order currency differs from the company currency, convert into the company currency
   for the order's company at the conversion date — which is the date the caller supplies, or
   the order deadline, or today — without rounding.
5. Round the result to the **Product Price** precision.

```formula
move_price_unit = round_to_precision(
      convert_to_company_currency(
          ( total_void( price_unit × (1 − discount ÷ 100) , taxes , q ) ÷ q )
          × ( product_unit_factor ÷ line_unit_factor ) ) ,
      product_price_digits )
```

**Worked example.** A line buys 10 boxes at 120.00 per box with a 5 % discount; 1 box = 12
units; the tax is 20 % fully recoverable; the order is in a currency whose rate to the company
currency is 1.25 (one company-currency unit buys 1.25 order-currency units).

- discounted price 114.00 per box;
- excluding tax, still 114.00 per box;
- per unit: 114.00 ÷ 12 = 9.50;
- into the company currency: 9.50 ÷ 1.25 = 7.60;
- rounded to 2 digits: **7.60** per unit.

Ten boxes therefore bring 120 units into stock at 7.60 each, a valuation of 912.00 in the
company currency.

---

## 9. Bill line preparation

When a purchase order line becomes a bill line:

```formula
bill_line.quantity  = qty_to_invoice          (negated when the target document is a refund)
bill_line.price_unit = convert( line.price_unit , from = order_currency , to = bill_currency ,
                                company = line_company , date = bill_date or today , round = false )
bill_line.discount   = line.discount
```

and, when inventory is installed and no balance was supplied:

```formula
bill_line.balance = convert_to_company_currency(
                        total_excluded( price_unit × (1 − discount ÷ 100) , taxes , qty_to_invoice ) ,
                        round = false )
```

The balance is computed with tax rounding disabled at both the tax and the base level, so that
the bill's own rounding is applied once, later.

---

## 10. Vendor punctuality

### 10.1 The vendor's on-time delivery rate

Let *D* be the lookback window in days, taken from the system parameter
`purchase_stock.on_time_delivery_days` (full name: the purchase-stock on-time-delivery-days
parameter) and defaulting to 365.

1. Select the purchase order lines whose partner is the vendor, whose order date is later than
   today minus *D* days, whose received quantity is not zero, whose order is in the purchase
   status, and whose product is not a service.
2. Select the done stock moves of those lines.
3. Keep the moves whose **date part** is on or before the **date part** of their line's expected
   arrival; sum their quantities per line into *on time(line)*.
4. Aggregate per vendor:

```formula
ordered(vendor) = Σ over selected lines of line.product_uom_qty
on_time(vendor) = Σ over selected lines of on_time(line)
on_time_rate    = on_time(vendor) ÷ ordered(vendor) × 100      when ordered(vendor) ≠ 0
                = −1                                            otherwise
```

A rate of −1 is the sentinel for "no data" and is what the interface tests before showing a
percentage.

**Worked example.** Over the window, one vendor has three qualifying lines: 100, 50 and 25
units ordered (in the product reference unit), with 100, 50 and 0 units delivered on time. Then
ordered = 175, on time = 150, and the rate is 150 ÷ 175 × 100 = **85.71 %**.

Note that the numerator sums **move quantities** while the denominator sums **line total
quantities** in the product reference unit; over-deliveries can therefore push the rate above
100 %.

### 10.2 The vendor delay entry

Per purchase order line:

```formula
qty_total   = line.product_uom_qty
qty_on_time = Σ over move details d of
                 ( d.quantity × d.unit_factor ÷ product_unit_factor )
                 when d's move is done AND date_part(line.date_planned) ≥ date_part(move.date)
                 else 0
```

and, when rows are aggregated:

```formula
on_time_rate = Σ qty_on_time ÷ Σ qty_total × 100     when Σ qty_total ≠ 0
             = 100                                   otherwise
```

Rows whose summed total quantity is zero are excluded from any grouping that aggregates the
rate.

### 10.3 The dashboard on-time figure

Over the purchase orders whose status is purchase and whose expected arrival is within the last
three months:

```formula
on_time_count = count of orders having an arrival date whose date part is ≤ the date part of the expected arrival
otd           = round_to_precision( on_time_count ÷ total_count × 100 , 0 ) , or 100 when total_count = 0
```

The same figure is computed a second time restricted to the current user's own orders.

### 10.4 The dashboard days-to-order figure

Over the purchase orders whose status is purchase, created within the last three months and
carrying a confirmation date:

```formula
average_seconds = Σ ( date_approve − create_date , in seconds ) ÷ count
days_to_order   = round_to_precision( average_seconds ÷ 60 ÷ 60 ÷ 24 , 2 )
```

with 0 when there are no such orders. The same figure is computed again restricted to the
current user's own orders, using that user's own count as the divisor.

**Worked example.** Three orders were confirmed 2 days, 5 days and 0.5 days after creation.
The average is (172 800 + 432 000 + 43 200) ÷ 3 = 216 000 seconds = **2.50** days.

---

## 11. Replenishment suggestions

Present when inventory is installed. Three parameters come from the panel: a **basis**, a
**horizon in days** *h*, and a **percentage** *p*.

### 11.1 Monthly demand

Select the stock moves of the product whose status is waiting, ready, confirmed, partially
available or done, whose date falls in the window implied by the basis, and whose direction
counts as demand (leaving the selected warehouse, or reaching a customer, production or
transit location). Sum their demanded quantities into *Q*. Then:

```formula
monthly_demand = Q ÷ factor
```

| Basis | Factor |
|---|---|
| one year | 12 |
| three months, or the same quarter last year | 3 |
| one week | 7 ÷ (365.25 ÷ 12) = 0.2300… |
| thirty days (the shipped default) | 1 |

### 11.2 Suggested quantity

For the basis **actual demand**:

```formula
suggested_qty = max( round_up( − forecast_quantity × p ÷ 100 , 0 ) , 0 )   when forecast_quantity < 0
              = 0                                                          otherwise
```

For every other basis:

```formula
monthly_ratio = h ÷ ( 365.25 ÷ 12 )
raw           = monthly_demand × monthly_ratio × p ÷ 100
                − max( on_hand_quantity , 0 ) − max( incoming_quantity , 0 )
suggested_qty = max( round_up( raw , 0 ) , 0 )
```

with 0 whenever the monthly demand is not strictly positive. The forecast used by the *actual
demand* branch is evaluated at now plus *h* days.

**Worked example.** Basis "thirty days", horizon 7 days, percentage 100 %. Over the last 30
days, 260 units left the warehouse, so the monthly demand is 260 ÷ 1 = 260. The monthly ratio
is 7 ÷ 30.4375 = 0.229938…, so the raw need is 260 × 0.229938… × 1 = 59.784. On hand is 20 and
incoming is 15, so raw = 59.784 − 20 − 15 = 24.784, and the suggestion is round up(24.784, 0) =
**25 units**. With a percentage of 50 % instead, the raw need would be 29.892, giving
29.892 − 35 = −5.108 and a suggestion of **0**.

### 11.3 Suggested estimated price

```formula
suggest_estimated_price = suggested_qty × price
```

where *price* is the **discounted** price of the vendor pricelist entry selected for the
suggested quantity; failing that, the discounted price of the entry with the smallest minimum
quantity; failing that, the product's cost. The value is 0 when the suggested quantity is not
positive.

---

## 12. Grouping procurement needs onto one request for quotation

Let *d* be the requested arrival timestamp and *w(d)* its day-of-week index with Monday = 1 and
Sunday = 7.

| Grouping policy | Accepted window for an existing order's expected arrival |
|---|---|
| On Order | No date window. Instead the order must carry the same procurement references, or no reference at all when the procurement has none. |
| Daily | From *d* at 00:00:00 to *d* at 23:59:59.999999. |
| Weekly, no target week day | From (*d* − *w(d)* days) at 00:00:00 to (*d* + (6 − *w(d)*) days) at 23:59:59.999999. |
| Weekly, target week day *t* | The single day *d* + δ where δ = (7 + *t* − *w(d)*) mod 7, from 00:00:00 to 23:59:59.999999. |
| Always | No date window at all. |

```formula
δ = ( 7 + target_weekday − weekday(requested_date) ) mod 7
```

**Worked example.** The requested arrival is Wednesday the 14th, so *w(d)* = 3.

- *Weekly with no target*: the window runs from Sunday the 11th at 00:00 to Saturday the 17th
  at 23:59:59.999999 — note that this "week" starts on the Sunday, because subtracting the
  day-of-week index from a Wednesday lands two days before the Monday.
- *Weekly with target Friday* (*t* = 5): δ = (7 + 5 − 3) mod 7 = 9 mod 7 = 2, so the single
  accepted day is Friday the 16th. The new line's expected arrival is pushed forward to the
  16th, and, if the order's own expected arrival is not earlier than the new one, the order
  deadline is pushed forward by the same 2 days.
- *Weekly with target Monday* (*t* = 1): δ = (7 + 1 − 3) mod 7 = 5 mod 7 = 5, so the accepted
  day is Monday the 19th — the **next** Monday, never the one already past.

### 12.1 The order deadline chosen for a new order

```formula
date_order = min over the group's procurements of
                 ( explicit_order_date , or requested_arrival − vendor_lead_time days )
```

### 12.2 Pulling the order deadline back for a new line

After a new line's expected arrival is known:

```formula
candidate = ( order.date_planned or min over new lines of date_planned ) − vendor_lead_time days
if date_part(candidate) < date_part(order.date_order) then order.date_order = candidate
```

---

## 13. Lead-time contribution of a buy rule

When the scheduler asks a route how long a replenishment takes, a buy rule contributes:

```formula
total_delay += vendor_lead_time          (unless the caller asks to ignore the vendor lead time)
total_delay += company.days_to_purchase
```

and, when the product has **no** vendor at all:

```formula
total_delay += 365
```

with the explanation line *No Vendor Found: + 365 day(s)*. The other explanation lines are
*Receipt Date* with the vendor lead time, *Vendor Lead Time: + n day(s)*, *Order Deadline* with
the days-to-purchase value, and *Days to Purchase: + n day(s)*.

**Worked example.** A product needs to be available on the 30th; the vendor's lead time is 7
days and the company needs 2 days to purchase. The scheduler therefore plans a request for
quotation dated the 30th − 7 − 2 = the **21st**, with an expected arrival of the 28th.

---

## 14. Merging requests for quotation

Two lines merge when they agree on product, unit, analytic distribution and discount, and when
their expected arrivals are close enough:

```formula
| line_a.date_planned − line_b.date_planned | ≤ 86 400 seconds
```

On merge:

```formula
surviving_line.product_qty = surviving_line.product_qty + incoming_line.product_qty
surviving_line.price_unit  = min( surviving_line.price_unit , incoming_line.price_unit )
```

**Worked example.** Two requests for the same vendor, currency and destination each carry a
line for the same product: 30 units at 4.10 expected on the 12th at 08:00, and 20 units at 3.95
expected on the 12th at 23:00. The gap is 15 hours, under the 24-hour tolerance, so the lines
merge to 50 units at min(4.10, 3.95) = **3.95**, a subtotal of 197.50.

Had the second been expected on the 13th at 09:00 (a 25-hour gap) the lines would not have
merged and the survivor would carry both.

---

## 15. Matching an incoming bill to orders

### 15.1 Tolerance

```formula
TOLERANCE = 0.02
```

expressed in the bill's currency and applied to totals, never to unit prices.

### 15.2 Remaining amount of an order line

```formula
amount_to_invoice(line) = ( 1 − qty_invoiced ÷ product_qty ) × price_total
```

Only lines with a non-zero ordered quantity are considered, so the division is safe.

### 15.3 Total match test

```formula
total_match  ⇔  amount_total − TOLERANCE  <  Σ amount_to_invoice(line)  <  amount_total + TOLERANCE
```

### 15.4 Vendor-and-amount fallback

```formula
candidates = orders of this vendor (or of any of its children) in the purchase status,
             with billing status in { "to invoice" , "no" } ,
             and amount_total between amount_total_of_bill − TOLERANCE
                                  and amount_total_of_bill + TOLERANCE
```

The fallback fires only when exactly one candidate is found.

### 15.5 The subset search

Recursive, on the lines sorted by remaining amount descending. At each position *i* with the
remaining target *g*:

```formula
if amount_to_invoice(line_i) < g − TOLERANCE:
        recurse on the lines after i with target g − amount_to_invoice(line_i)
        and prepend line_i to every solution found
else if g − TOLERANCE ≤ amount_to_invoice(line_i) ≤ g + TOLERANCE:
        record the single-line solution [ line_i ]
if more than one solution has been recorded at this level: abandon and return nothing
```

**Worked example.** An order has four lines with remaining amounts 500.00, 300.00, 200.00 and
120.00. A scanned bill totals 500.01.

- Sorted: 500.00, 300.00, 200.00, 120.00.
- Position 1: 500.00 is not below 500.01 − 0.02 = 499.99, and it lies within
  [499.99, 500.03], so the single-line solution [500.00] is recorded.
- Position 2: 300.00 < 499.99, so recurse with target 200.01 over [200.00, 120.00]. There,
  200.00 lies within [199.99, 200.03], so [300.00, 200.00] is also a solution.
- Two solutions now exist, so the search abandons and returns nothing: the bill falls through
  to the *order match* branch and every line of the order is proposed with its full quantity.

Had the bill totalled 620.01 instead, only [500.00, 120.00] would match and that unique subset
would be returned.

### 15.6 The line pairing similarity

Candidates require an **exact** unit-price equality and a bill quantity not exceeding the order
line's remaining quantity:

```formula
candidate  ⇔  invoice_line.price_unit = purchase_line.price_unit
              AND invoice_line.quantity ≤ purchase_line.product_qty − purchase_line.qty_invoiced
```

Ties are broken by the textual similarity ratio between the bill line's label and the order
line's description, a number between 0 and 1 where 1 means identical; the highest wins.

---

## 16. The price difference at billing time

Present when inventory valuation is installed and the company uses the cost-of-goods-sold
recognition style in which stock is debited at bill time. It applies **only** to products whose
cost method is the standard-price method.

```formula
valuation_price_unit = convert( convert_price( product.standard_price ,
                                               product_reference_unit , bill_line_unit ) ,
                                from = company_currency , to = bill_currency ,
                                company = bill_company , date = bill_line_date , round = false )
```

negated when the document is a vendor refund. Then:

```formula
price_unit_val_dif = gross_unit_price − valuation_price_unit
price_subtotal     = relevant_quantity × price_unit_val_dif
```

where the relevant quantity is the bill line's quantity. Two balancing journal items are
produced when the subtotal is not zero in the bill's currency **and** the line's stored unit
price still equals its computed unit price at the Product Price precision (a discount that has
been applied makes the two differ, and the comparison then suppresses the adjustment). Their
amounts and accounts are in [`accounting-effects.md`](accounting-effects.md).

**Worked example.** A product's standard cost is 9.00. A bill line invoices 10 units at 10.00
in the company currency, no discount, no tax. Then the valuation price is 9.00, the difference
per unit is 1.00, and the subtotal is 10 × 1.00 = 10.00: ten units' worth of extra cost is
moved out of the stock account and into the price-difference account.

---

## 17. Valuing an incoming move from its bills

When inventory asks a purchase-linked move what it is worth, the answer prefers the bills over
the order:

1. Walk the purchase line's bill lines. Skip those dated after the requested date, and those
   whose document is not posted. For a vendor bill add, for a vendor refund subtract:

```formula
billed_quantity += convert_quantity( bill_line.quantity , bill_line.unit , product_reference_unit )
billed_value    += round_to_currency( bill_line.price_subtotal ÷ bill_line.currency_rate )
```

2. If the billed quantity is not positive, fall back to the generic valuation.
3. Compute how much of that billed quantity earlier moves of the same line have already
   consumed:

```formula
other_candidates_qty = Σ over moves of the same line and product,
                         dated before this move (ties broken by identifier),
                         of ( + valued quantity for incoming and dropship moves ,
                              − valued quantity for outgoing moves )
```

4. If the billed quantity does not exceed that consumed quantity, fall back to the generic
   valuation. Otherwise reduce both:

```formula
billed_value    = billed_value × ( ( billed_quantity − other_candidates_qty ) ÷ billed_quantity )
billed_quantity = billed_quantity − other_candidates_qty
```

5. Finally:

```formula
if requested_quantity ≥ billed_quantity :  value = billed_value ,               quantity = billed_quantity
else                                     :  value = requested_quantity × billed_value ÷ billed_quantity ,
                                            quantity = requested_quantity
```

When no bill exists at all, the move is valued from the order instead, at the move unit price of
section 8 multiplied by the quantity, capped at the move's own done quantity converted into the
product reference unit.

**Worked example.** A purchase line for 10 units is received in two moves of 6 and 4. One
posted bill covers 10 units with a subtotal of 600.00 at a currency rate of 1. The first move
asks for the value of 6 units: billed quantity 10, billed value 600.00, no earlier move, so the
requested 6 is below 10 and the value is 6 × 600.00 ÷ 10 = **360.00**. The second move asks for
4: the earlier move has already consumed 6, so the remaining billed value is
600.00 × ((10 − 6) ÷ 10) = 240.00 for a remaining quantity of 4; the requested 4 is not below 4,
so the value is **240.00**.

---

## 18. Purchase analysis figures

All amounts are first divided by the order's own currency rate — bringing them to the company
currency — and then multiplied by the presentation rate of the currency table built for the
reader's allowed companies.

```formula
delay      = ( date_approve − date_order ) in days
delay_pass = ( line.date_planned − date_order ) in days
```

```formula
price_total   = round_to_precision( Σ ( line.price_total   ÷ order.currency_rate ) , 2 ) × presentation_rate
untaxed_total = round_to_precision( Σ ( line.price_subtotal ÷ order.currency_rate ) , 2 ) × presentation_rate
```

```formula
price_average = round_to_precision(
                    Σ ( line.product_qty × line.price_unit ÷ order.currency_rate )
                  ÷ Σ ( line.product_qty × line_unit_factor ÷ product_unit_factor ) , 2 )
                × presentation_rate
```

The denominator restates the quantities into the product reference unit, so that the average is
a price per product unit rather than per line unit. When the denominator is zero the result is
undefined and is returned as empty.

```formula
qty_ordered  = Σ ( line.product_qty  × line_unit_factor ÷ product_unit_factor )
qty_received = Σ ( line.qty_received × line_unit_factor ÷ product_unit_factor )
qty_billed   = Σ ( line.qty_invoiced × line_unit_factor ÷ product_unit_factor )
```

```formula
qty_to_be_billed = qty_ordered  − qty_billed    when product.purchase_method = "purchase"
                 = qty_received − qty_billed    when product.purchase_method = "receive"
```

```formula
weight = Σ ( product.weight × line.product_qty × line_unit_factor ÷ product_unit_factor )
volume = Σ ( product.volume × line.product_qty × line_unit_factor ÷ product_unit_factor )
```

```formula
days_to_arrival = ( earliest_done_receipt_date  or  line.date_planned ) − date_order , in days
```

### 18.1 Re-aggregating the average price

When rows are grouped, a plain average of the row averages would weight a one-unit line as
heavily as a thousand-unit line. The aggregation is therefore replaced by:

```formula
grouped_price_average = Σ ( row.price_average × row.qty_ordered ) ÷ Σ row.qty_ordered
```

returning nothing when the denominator is zero.

**Worked example.** Two rows: 100 units averaging 10.00, and 5 units averaging 30.00. The plain
average would be 20.00; the weighted average is (100 × 10.00 + 5 × 30.00) ÷ 105 = 1 150 ÷ 105 =
**10.95**.

---

## 19. The "late" search

The stored search for late orders is built from two parts combined with a logical *and*:

```formula
order_part = ( state = "purchase" ) AND ( date_planned ≤ now )
             AND ( no transfer exists OR some transfer's state ∉ { done , cancel } )   [with inventory]
line_part  = ( qty_received < product_qty )
```

and the whole reads: *the order has at least one line whose order satisfies the order part and
whose received quantity is strictly below its ordered quantity*. Searching for **not late**
inverts the line part to *received quantity is greater than or equal to ordered quantity* while
keeping the same order part. Only equality and inequality operators are accepted; anything else
raises *"Unsupported operator"*.

---

## 20. Purchased quantity over the last year

```formula
purchased_product_qty(variant) = round_to_unit(
        Σ over lines of confirmed orders whose confirmation date ≥ today − 1 year
          of line.product_uom_qty ,
        product_reference_unit )
```

```formula
purchased_product_qty(template) = round_to_unit( Σ over its variants of purchased_product_qty(variant) ,
                                                 template_reference_unit )
```

Archived variants are included in the template's sum.

---

## 21. Project profitability contribution

For a project whose analytic account is *A*, over the purchase order lines whose analytic
distribution mentions *A* and whose order is confirmed:

```formula
analytic_contribution(line) = ( Σ over distribution keys containing A of percentage ) ÷ 100
line_amount_to_invoice      = convert( line.price_subtotal , to = project_currency ) × analytic_contribution(line)
```

and, over the bill lines of that purchase line that are not cancelled and mention *A*:

```formula
cost(bill_line) = convert( bill_line.price_subtotal , to = project_currency )
                  × analytic_contribution(bill_line)
                  × ( −1 when the bill line is a refund , +1 otherwise )
```

```formula
amount_invoiced   −= cost(bill_line)    for every posted bill line
amount_to_invoice −= cost(bill_line)    for every draft bill line
amount_to_invoice −= line_amount_to_invoice − Σ cost(bill_line) over non-refund bill lines
```

When a purchase line has no bill line at all, only the last term applies. The signs are
negative because purchases are costs.

**Worked example.** A purchase line of 1 000.00 is split 60 % to project *A*. Its contribution
to the amount still to bill is −600.00. A posted bill line of 400.00 also split 60 % to *A*
contributes −240.00 to the amount billed and reduces the amount still to bill by the same
240.00, leaving −360.00 still to bill.
