# Calculations

Every formula and every algorithm of the Delivery and Shipping domain, with its inputs named in
words, its evaluation order, its rounding rule and at least one worked numeric example carried to
the last decimal the rule produces.

Unless a section says otherwise:

- Monetary amounts are rounded by the currency named in the section, using the currency's own
  decimal places and its half-up rounding rule. The worked examples use a currency whose smallest
  unit is one hundredth.
- Weights are expressed in the unit named by the weight unit-of-measure system parameter, which
  resolves to the kilogram unless the parameter says pounds, and volumes in the unit named by the
  volume system parameter, which resolves to the cubic metre unless the parameter says cubic feet.
  The worked examples use kilograms and cubic metres.
- A *reference unit* quantity is a quantity converted into the product's own reference unit of
  measure by the conversion routine of
  [`../units-of-measure-and-packaging/`](../units-of-measure-and-packaging/).

| # | Subject |
|---|---|
| 1 | The availability filter |
| 2 | The destination address match and its regular expression |
| 3 | The total weight and total volume of the availability filter |
| 4 | The fixed charge engine |
| 5 | The rule-based charge engine: variable collection |
| 6 | The rule-based charge engine: rule evaluation and the rule's readable name |
| 7 | The complete charge pipeline |
| 8 | The estimated order weight and the shipping weight |
| 9 | Parcels built from a Sales Order |
| 10 | Parcels built from a Transfer |
| 11 | The value of a moved quantity |
| 12 | Package weights and the put-in-pack default |
| 13 | Carrier propagation and the transfer grouping key |
| 14 | Batch weight caps |
| 15 | The collection point record and the store distance |
| 16 | The per-store stock check |
| 17 | The quantity a product page advertises |
| 18 | The tracking link |
| 19 | The weight segment of the parcel barcode |
| 20 | The currency conversions used by this domain |

---

## 1. The availability filter

A Delivery Method is *available* for a source document — a Sales Order or a Transfer — when all
five tests below hold. The tests are evaluated in this order and the first failure stops the
evaluation.

1. **Destination address** — section 2.
2. **Required tags** — at least one of the method's required tags is carried by a product of the
   document, or the method requires none.
3. **Excluded tags** — none of the method's excluded tags is carried by a product of the document.
4. **Maximum weight** — section 3.
5. **Maximum volume** — section 3.

The set of products examined by tests 2 and 3 is:

- for a Sales Order: the products of every order line, including service lines, display lines and
  the shipping charge line itself;
- for a Transfer: the products of every move.

The tags examined are the product's complete tag set, which includes the tags of its template and
the tags of its variant.

A source document of any other kind is refused with "Invalid source document type".

For a Sales Order there is a sixth test, applied only by the storefront and only to the rule-based
kind: the rating routine must succeed. A rule-based method whose rules match nothing for the
current basket is therefore not offered at all, rather than being offered and then failing.

```formula
available =
      destination address matches
  AND (no required tags OR at least one required tag present)
  AND (no excluded tag present)
  AND (maximum weight is zero OR total weight ≤ maximum weight)
  AND (maximum volume is zero OR total volume ≤ maximum volume)
```

**Worked example.** A method restricted to one country, requiring the tag *Fragile*, excluding the
tag *Dangerous*, with a maximum weight of 10 kilograms and no volume limit. The order is delivered
to an address in that country and contains two lines: three units of a product weighing
1.200 kilograms tagged *Fragile*, and one unit of a service product with no weight and no tag.

```formula
destination address matches = true
required tag present        = true          (Fragile is on the first product)
excluded tag present        = false
total weight                = 1.200 kg × 3 + 0.000 kg × 1 = 3.600 kg
3.600 kg ≤ 10.000 kg        = true
maximum volume              = 0             (no limit)
available                   = true
```

---

## 2. The destination address match and its regular expression

Three independent restrictions are applied to the destination contact of the document — the
delivery address of a Sales Order, the contact of a Transfer.

1. **Countries.** When the method lists at least one country, the contact's country must be one of
   them. A contact with no country fails.
2. **Regions.** When the method lists at least one region, the contact's region must be one of
   them. A contact with no region fails.
3. **Postal-code prefixes.** When the method lists at least one prefix, the contact's postal code
   must match. A contact with no postal code fails.

The postal-code test is built as follows:

1. Take the `name` of every Delivery Postal Code Prefix attached to the method. Each is already
   stored upper-cased.
2. Prefix each of them with the start-of-string marker.
3. Join them with the alternation marker into one pattern.
4. Upper-case the contact's postal code.
5. Match the pattern against the start of the upper-cased postal code.

Because each alternative is anchored at the start and nothing is anchored at the end, a prefix
matches every postal code that begins with it. Appending the end-of-string marker to a prefix makes
it match that postal code exactly and no longer one.

```formula
pattern = "^" + prefix₁ + "|" + "^" + prefix₂ + "|" … + "^" + prefixₙ
matches = the upper-cased postal code starts with one of the alternatives
```

**Worked example.** The method carries the prefixes `10`, `750` and `2000$`. The pattern is
`^10|^750|^2000$`.

| Postal code | Upper-cased | Matches | Why |
|---|---|---|---|
| `1000` | `1000` | yes | begins with `10` |
| `10` | `10` | yes | begins with `10` |
| `75008` | `75008` | yes | begins with `750` |
| `2000` | `2000` | yes | equals `2000`, so the anchored alternative matches |
| `20001` | `20001` | no | `2000$` requires the code to end there, and neither `10` nor `750` starts it |
| `b100` | `B100` | no | the pattern is anchored at the start |
| *(empty)* | — | no | a contact with no postal code fails the test outright |

---

## 3. The total weight and total volume of the availability filter

The two limits are evaluated over the document as a whole.

For a **Sales Order**:

```formula
total weight of the order =
    Σ over every order line of ( unit weight of the product in the line
                                 × reference-unit quantity of the line )

total volume of the order =
    Σ over every order line of ( unit volume of the product in the line
                                 × reference-unit quantity of the line )
```

For a **Transfer**:

```formula
total weight of the transfer =
    Σ over every move of ( unit weight of the product in the move
                           × demanded quantity of the move in the product's reference unit )

total volume of the transfer =
    Σ over every move of ( unit volume of the product in the move
                           × demanded quantity of the move in the product's reference unit )
```

No line is excluded. A shipping charge line contributes the weight of its delivery product, which
is normally zero because a service product carries no weight. A cancelled line still contributes,
because the availability filter reads the lines as they stand.

Neither sum is rounded. The comparison is a plain arithmetic comparison against the method's limit,
with a limit of zero meaning *no limit*.

**Worked example — unit conversion matters.** A product weighs 1.000 kilogram per unit and its
reference unit is the piece. An order line sells two dozen of it. The reference-unit quantity is
2 × 12 = 24 pieces, so the contribution to the total weight is 1.000 × 24 = 24.000 kilograms, not
1.000 × 2 = 2.000 kilograms. A method whose maximum weight is 10.000 kilograms is therefore not
available for that order.

**Worked example — volume.** A product occupies 1.000 cubic metre per piece. An order line sells
one dozen. The reference-unit quantity is 12 pieces and the total volume is 12.000 cubic metres. A
method whose maximum volume is 10.000 cubic metres is not available.

---

## 4. The fixed charge engine

The fixed engine answers a rate request in two steps.

1. **Destination test.** Only the destination-address test of section 2 is repeated — not the tag
   tests and not the weight and volume tests. When it fails, the engine returns failure with the
   message "Error: this delivery method is not available for this address." and a charge of zero.
2. **Price lookup.** The charge is the price of the method's delivery product for a quantity of one
   under the order's price list, obtained from
   [`../pricing-and-pricelists/`](../pricing-and-pricelists/).

```formula
fixed charge = price of the delivery product for one unit under the order's price list
```

Two consequences follow and both are deliberate:

- The `fixed_price` field is not read by the engine. It is a writable mirror of the delivery
  product's sales price, so the value the user types on the method is the value the price list
  returns when no price-list rule overrides it.
- A price-list rule on the delivery product therefore overrides the method's own charge, including
  across currencies: the price list returns its own currency and the pipeline of section 7 performs
  no conversion for the fixed engine.

**Worked example — plain.** The delivery product's sales price is 9.95. The order's price list has
no rule for it. The engine returns 9.95.

**Worked example — overridden by a price list.** The delivery product's sales price is 10.00. The
order's price list carries a fixed rule of 5.00 on that exact product variant. The engine returns
5.00, and the charge line on the order is 5.00.

**Worked example — overridden by a price list in another currency.** The company currency is the
one whose sales price is 10.00; the order's price list is expressed in a second currency and
carries a fixed rule of 5.00 on the delivery product. The engine returns 5.00. The pipeline treats
that amount as already being in the order's currency and performs no conversion, so the charge line
is 5.00 in the second currency.

---

## 5. The rule-based charge engine: variable collection

Before any rule is evaluated, five quantities are collected from the Sales Order. The collection
runs with elevated rights, so that a salesperson who cannot read costs can still obtain a rate.

Lines are **skipped** when any of the following holds:

- the line's state is `cancel`;
- the line has no product;
- the line is the shipping charge line;
- the line's product is of kind `service` or of kind `combo`.

For every line that is not skipped, let *q* be the line's ordered quantity converted into the
product's reference unit.

```formula
weight   = Σ over kept lines of ( unit weight of the product × q )
volume   = Σ over kept lines of ( unit volume of the product × q )
wv       = Σ over kept lines of ( unit weight × unit volume × q )
quantity = Σ over kept lines of ( q )
```

The fifth quantity, the price, is the order total without carriage, converted from the order
currency into the company currency:

```formula
order total without carriage = order total including taxes
                             − Σ over shipping charge lines of ( line total including taxes )

price = order total without carriage converted from the order currency into the company currency
        at the order date
```

The weight actually used is then chosen by this precedence:

1. the weight passed by the caller — the selection wizard passes the weight the user typed;
2. failing that, the order's stored shipping weight;
3. failing that, the weight summed above.

A weight of zero at step 1 or 2 falls through to the next step, because the precedence is a
first-truthy precedence, not a first-present precedence.

Finally, the *weight times volume* variable is given a fallback: when the accumulated `wv` is zero
it becomes the product of the totals.

```formula
wv used = wv accumulated,  when wv accumulated ≠ 0
wv used = volume × weight, when wv accumulated = 0
```

The five variables are then, by their stored names: `price`, `volume`, `weight`, `wv`, `quantity`.

**Worked example.** An order carries three lines:

| Line | Product | Ordered | Reference unit | Unit weight | Unit volume | Total including taxes |
|---|---|---|---|---|---|---|
| 1 | Desk | 3 pieces | piece | 1.500 kg | 2.500 m³ | 345.00 |
| 2 | Assembly service | 2 hours | hour | — | — | 150.00 |
| 3 | Carriage | 1 piece | piece | 0.000 kg | 0.000 m³ | 12.00 |

Line 2 is skipped because its product is a service; line 3 is skipped because it is the shipping
charge line. The order total including taxes is 507.00 and the company currency is the order
currency.

```formula
weight   = 1.500 kg × 3 = 4.500 kg
volume   = 2.500 m³ × 3 = 7.500 m³
wv       = 1.500 × 2.500 × 3 = 11.250
quantity = 3
price    = 507.00 − 12.00 = 495.00
```

---

## 6. The rule-based charge engine: rule evaluation and the rule's readable name

### 6.1 Evaluation

The Delivery Price Rules of the method are walked in their stored order — by `sequence` ascending,
then by `list_price` ascending, then by identifier ascending. For each rule the condition is
evaluated as:

```formula
condition = ( value of the rule's condition variable )
            ⟨ rule's operator ⟩
            ( rule's maximum value )
```

where the operator is one of `==`, `<=`, `<`, `>=`, `>` and the comparison is a plain arithmetic
comparison with no tolerance. The first rule whose condition holds produces the charge and the walk
stops:

```formula
charge before margins = rule's base amount
                      + rule's factor amount × ( value of the rule's factor variable )
```

The condition variable and the factor variable are chosen independently: a rule may test the weight
and charge per unit of quantity.

When no rule's condition holds, the engine fails with the message "Not available for current
order". The failure is turned into a rate result carrying that text as its error message; it is not
raised to the user by the rating step itself.

The charge produced here is expressed in the **company currency**, because the price variable was
converted into the company currency and the rules' amounts are stored in the method currency, which
is the delivery product's currency. The rating routine then converts the result from the company
currency into the order currency at the order date.

### 6.2 Worked example — a weight band

A method carries three rules:

| Sequence | Condition | Base amount | Factor amount | Factor variable | Readable name |
|---|---|---|---|---|---|
| 10 | `weight` `<=` 5.00 | 20.00 | 0.00 | `weight` | `if weight <= 5.00 then fixed price 20.00` |
| 10 | `weight` `>=` 5.00 | 50.00 | 0.00 | `weight` | `if weight >= 5.00 then fixed price 50.00` |
| 20 | `price` `>=` 300.00 | 0.00 | 0.00 | `weight` | `if price >= 300.00 then fixed price 0.00 plus 0.00 times weight` |

An order weighing 7.500 kilograms with a total without carriage of 250.00 in the company currency
is evaluated as follows:

1. Rule 1 — the two rules of sequence 10 are ordered by their factor amount, which is equal, then
   by identifier, so the `<=` rule comes first. 7.500 ≤ 5.00 is false. Continue.
2. Rule 2 — 7.500 ≥ 5.00 is true. Stop.

```formula
charge before margins = 50.00 + 0.00 × 7.500 = 50.00
```

### 6.3 Worked example — a charge per unit of weight times volume

A method carries one rule: condition `price` `>=` 0.00, base amount 0.00, factor amount 2.00,
factor variable `wv`. The order of section 5 is rated:

```formula
value of the factor variable = wv used = 11.250
charge before margins        = 0.00 + 2.00 × 11.250 = 22.50
```

### 6.4 Worked example — no rule matches

The method of section 6.2 is asked to rate an order weighing 7.500 kilograms while the first rule
has been changed to `weight` `<=` 5.00 and the second to `weight` `>=` 10.00, and the third has
been deleted. Neither condition holds, so the engine fails with the error message "Not available
for current order" and a charge of zero, and the storefront does not offer the method at all.

### 6.5 The readable name of a rule

The name shown in the rule list is built from the rule's own fields. Let *v* be the stored value of
the condition variable, *o* the stored value of the operator, *m* the maximum value formatted with
exactly two decimals, *f* the stored value of the factor variable, *b* the base amount and *p* the
factor amount. When the rule's currency is known, *b* and *p* are formatted as amounts in that
currency; otherwise they are formatted with exactly two decimals and no currency.

```formula
prefix = "if " + v + " " + o + " " + m + " then"

name = prefix + " fixed price " + b                                  when b ≠ 0 and p = 0
name = prefix + " " + p + " times " + f                              when p ≠ 0 and b = 0
name = prefix + " fixed price " + b + " plus " + p + " times " + f   otherwise
```

The third branch is also what a rule with both amounts equal to zero produces.

**Worked example.** A rule tests `weight` with the operator `<=` against 30, charges a base of 5.00
and a factor of 0.00 per unit of `weight`, in a currency written with a leading symbol. Its name is
`if weight <= 30.00 then fixed price 5.00` with the currency symbol in front of the amount. The
same rule with a base of 0.00 and a factor of 1.25 is named `if weight <= 30.00 then 1.25 times
weight`; with a base of 5.00 and a factor of 1.25 it is named `if weight <= 30.00 then fixed price
5.00 plus 1.25 times weight`.

---

## 7. The complete charge pipeline

This is the single most important algorithm of the domain. Every rate request, wherever it comes
from — the selection wizard, the storefront, the express-checkout list, a carrier integration —
passes through it.

### 7.1 The steps in order

1. **Dispatch.** Look up the rating routine named after the method's provider kind. When there is
   none, return failure at once with the error message "Error: this delivery method is not
   available." and a charge of zero. Nothing further runs.
2. **Engine.** Call the routine. It returns four values: whether it succeeded, a charge, an error
   message and a warning message. The two core engines are specified in sections 4 and 6; the
   in-store engine always succeeds and returns the delivery product's sales price; a carrier
   integration returns the carrier's own quotation.
3. **Company resolution.** The company used by the following steps is the method's company, failing
   that the order's company, failing that the current company.
4. **Fiscal position adaptation.** The charge is passed through the tax-inclusive unit-price helper
   of [`../taxes/`](../taxes/) with the delivery product, the resolved company, the company
   currency both as the target currency and as the charge's own currency, the order date and the
   order's fiscal position. The helper:
   - takes the delivery product's sale taxes restricted to the resolved company;
   - when there are such taxes **and** the order carries a fiscal position, maps them through the
     fiscal position and adapts the charge from the original tax set to the mapped tax set, so that
     a tax-inclusive charge stays correct when a tax-inclusive tax is replaced by a tax-exclusive
     one;
   - performs no unit conversion, because the delivery product's own reference unit is used;
   - performs no currency conversion, because the target currency and the charge's currency are the
     same.
5. **Margins.** For the fixed kind the charge is returned unchanged — margins are deliberately
   ignored. For every other kind:

   ```formula
   fixed margin in order currency =
       method's fixed margin converted from the company currency into the order currency
       at the order date

   charge after margins = charge after step 4 × ( 1 + method's margin )
                        + fixed margin in order currency
   ```

6. **Rounding.** The charge after margins is rounded by the **order currency**, to that currency's
   decimal places, half up.
7. **Real charge kept.** The rounded charge is copied into a second value, the carrier charge, so
   that the interface can still show what the carriage really costs after the waiver has set the
   charge to zero.
8. **Free-above-a-threshold waiver.** When all four of the following hold, the charge becomes zero
   and the warning message becomes "The shipping is free since the order amount exceeds <threshold
   with two decimals>.", where the placeholder is the method's threshold amount:
   - the engine succeeded;
   - the method's waiver flag is set;
   - the method's provider kind is **not** `base_on_rule`;
   - the order total without carriage, converted from the order currency into the company currency
     at the order date, is greater than or equal to the method's threshold.

   The exclusion of the rule-based kind at step 8 means that a rule-based method never waives its
   charge through this mechanism. The same seller can obtain the same effect with a Delivery Price
   Rule whose condition is `price` `>=` the threshold and whose amounts are zero. The interface
   hides the waiver controls for the rule-based kind accordingly.

9. **Result.** The four values are returned, plus the carrier charge of step 7.

### 7.2 Worked example — a fixed charge with margins configured

Method: fixed kind, delivery product priced 9.95, margin 0.2, fixed margin 3.00, no waiver. Order
currency equals company currency. No fiscal position.

```formula
step 2  engine              = 9.95
step 4  fiscal adaptation   = 9.95        (no fiscal position)
step 5  margins             = 9.95        (the fixed kind ignores both margins)
step 6  rounding            = 9.95
step 7  carrier charge      = 9.95
step 8  waiver              = not applicable
charge written on the order = 9.95
```

### 7.3 Worked example — a rule-based charge with margins and a currency conversion

Method: rule-based, no company, so the company used for the conversions is the main company, whose
currency is the first currency. One rule: condition `price` `>=` 0.00, base amount 15.00, factor
amount 0.00, factor variable `weight`. Fixed margin 10.00, margin 0.

The order belongs to a company whose currency is a second currency, and the rate on the order date
is 0.5 second-currency units per first-currency unit.

```formula
step 2  engine, in the first currency      = 15.00 + 0.00 × weight = 15.00
        rule engine converts to the order currency: 15.00 × 0.5    = 7.50
step 4  fiscal adaptation                                          = 7.50
step 5  fixed margin converted: 10.00 × 0.5                        = 5.00
        charge after margins: 7.50 × ( 1 + 0 ) + 5.00              = 12.50
step 6  rounded by the order currency                              = 12.50
step 7  carrier charge                                             = 12.50
step 8  no waiver configured
charge written on the order                                        = 12.50
```

### 7.4 Worked example — the waiver

Method: fixed kind, delivery product priced 9.95, waiver flag set, threshold 100.00. Order currency
equals company currency. The order carries one line of 120.00 including taxes and no shipping
charge line yet.

```formula
step 2  engine                        = 9.95
step 4  fiscal adaptation             = 9.95
step 5  margins                       = 9.95
step 6  rounding                      = 9.95
step 7  carrier charge                = 9.95
step 8  order total without carriage  = 120.00
        120.00 ≥ 100.00               = true
        charge                        = 0.00
        warning message               = "The shipping is free since the order amount exceeds 100.00."
charge written on the order           = 0.00
cost shown in the wizard              = 9.95
```

The order line's description becomes the method's name, a line break and the reproduced text
"Free Shipping".

### 7.5 Worked example — the waiver just below the threshold

The same method and an order carrying one line of 99.99 including taxes.

```formula
step 8  order total without carriage = 99.99
        99.99 ≥ 100.00               = false
charge written on the order          = 9.95
```

The comparison is *greater than or equal to*: an order of exactly 100.00 is waived.

### 7.6 Worked example — a fiscal position that swaps a tax-inclusive tax for a tax-exclusive one

The delivery product carries a tax of ten per cent declared as included in the price, and the
order's fiscal position maps that tax to a tax of fifteen per cent declared as excluded. The
engine returns 10.00.

```formula
step 2  engine                                     = 10.00
step 4  amount without the included ten per cent   = 10.00 ÷ 1.10 = 9.090909…
        rounded to the currency's two decimals     = 9.09
        charge carried forward                     = 9.09
step 5  margins ignored (fixed kind)               = 9.09
step 6  rounding                                   = 9.09
charge written on the order                        = 9.09
line total including the fifteen per cent tax      = 9.09 × 1.15 = 10.4535 → 10.45
```

The same result is obtained when the delivery product is added to the order as an ordinary line,
which is the property the adaptation exists to preserve.

### 7.7 The shipment-time variant of the waiver

When a shipment is created the waiver is applied again, in a shorter form, to the charge the
carrier asked for. Before margins are applied to that charge:

```formula
when the method's waiver flag is set and the transfer has a Sales Order
and ( order total without carriage converted into the company currency ) ≥ threshold
then the carrier's charge becomes 0.00
```

Note two differences from step 8 of section 7.1: there is no exclusion of the rule-based kind here,
and no warning message is produced. The charge then passes through the margin step of section 7.1
step 5 and is stored on the transfer.

---

## 8. The estimated order weight and the shipping weight

### 8.1 The estimated weight of a Sales Order

```formula
estimated weight of the order =
    Σ over order lines that satisfy all four conditions below
      of ( reference-unit quantity of the line × unit weight of the product )
```

The four conditions are:

1. the product's kind is `consu`, that is, goods rather than a service or a combination;
2. the line is not the shipping charge line;
3. the line is not a display line — not a section and not a note;
4. the ordered quantity is strictly greater than zero.

The fourth condition means a corrective line with a negative quantity does not reduce the estimated
weight.

The sum is not rounded.

**Worked example.** An order carries two lines for the same product, which weighs 1.000 kilogram
per piece: one line of 1 piece and one line of −1 piece. The estimated weight is 1.000 kilogram,
not 0.000.

**Worked example with a unit conversion.** A line sells 10 pieces of a product weighing
1.000 kilogram: the estimated weight is 10.000 kilograms. Changing the line to 100 pieces makes it
100.000 kilograms, and the selection wizard opened afterwards proposes 100.

### 8.2 The stored shipping weight of a Sales Order

The order's shipping weight is a stored computed field whose value is the estimated weight of
section 8.1. It is recomputed whenever a line's ordered quantity or unit changes, and it is
writable: the selection wizard's weight control writes through to it, so a salesperson can rate a
shipment at a weight that differs from the sum of the line weights. The manual value survives until
the next change of a line quantity or unit.

### 8.3 The weight of a Stock Move

```formula
weight of the move = demanded quantity in the product's reference unit × unit weight of the product
```

The value is zero whenever the product's unit weight is not strictly positive. The field is stored
and recomputed when the product, the demanded quantity or the line unit changes. It is *not*
recomputed when the product's own weight changes, so a move keeps the weight the product had when
the move was written. The value is displayed with the stock-weight precision.

**Worked example.** A move demands 1 piece of a product weighing 1.000 kilogram: the move weighs
1.000. Raising the product's weight to 2.000 kilograms leaves the move at 1.000. Replacing the
move's product by one weighing 2.000 kilograms makes the move weigh 2.000.

### 8.4 The weight of a Transfer

```formula
weight of the transfer = Σ over moves whose state is not "cancel" of ( weight of the move )
```

Stored, recomputed when a move's weight changes, computed with elevated rights so that a user who
cannot read a move can still see the transfer's weight.

### 8.5 The estimated weight of a Transfer

A separate estimate, used when parcels are built from a return transfer:

```formula
estimated weight of the transfer =
    Σ over every move of ( demanded quantity in the product's reference unit × unit weight )
```

Unlike section 8.4 this reads the product's current weight and includes cancelled moves.

### 8.6 The shipping weight of a Transfer

The shipping weight is the weight actually handed to a carrier. It is owned by
[`../inventory-operations/`](../inventory-operations/) and is repeated here because every rate and
every parcel depends on it:

```formula
shipping weight of the transfer =
      bulk weight
    + Σ over the outermost packages of the transfer of
        ( package's shipping weight when it is set, otherwise the package's computed weight )

bulk weight =
    Σ over move lines that are not in any package of
      ( moved quantity converted into the product's reference unit × unit weight of the product )
```

The field is stored and writable, so a user may override it.

**Worked example.** A transfer carries two move lines, 5 units of a product weighing 2.400
kilograms and 5 units of a product weighing 0.300 kilograms, none of them packed.

```formula
bulk weight     = 2.400 × 5 + 0.300 × 5 = 12.000 + 1.500 = 13.500 kg
shipping weight = 13.500 kg
```

The first product is then packed into a container whose type has no base weight, and the package's
shipping weight is typed as 5:

```formula
bulk weight     = 0.300 × 5 = 1.500 kg
shipping weight = 1.500 + 5.000 = 6.500 kg
```

Setting the package's shipping weight back to 12.000 restores 13.500.

---

## 9. Parcels built from a Sales Order

A carrier integration that must quote or ship from a quotation, before any transfer exists, builds
its parcels from the order. The algorithm takes the order and a default container type.

### 9.1 Steps

1. **Declared value.** For every order line that is neither the shipping charge line nor a display
   line:

   ```formula
   total declared value = Σ ( reference-unit quantity of the line × unit cost of the product,
                              expressed in the currency described below )
   ```

   The conversion applied is from the **company currency into the product's currency** at today's
   date. In the ordinary configuration the two are the same currency and the conversion is the
   identity. Where they differ the conversion runs in the direction opposite to the one the name
   suggests; this is recorded as **compatibility finding** DSH-051 in
   [business-rules.md](business-rules.md).

2. **Total weight.**

   ```formula
   total weight = estimated weight of the order (section 8.1)
                + base weight of the default container type
   ```

   When the caller supplies a weight — as the selection wizard does — that weight replaces the sum
   entirely, including the container's base weight.

3. **Refusal on a weightless order.** When the total weight is exactly zero, the operation is
   refused with "The package cannot be created because the total weight of the products in the
   picking is 0.0 <weight unit label>", where the placeholder is the display name of the weight
   unit named by the weight system parameter.

4. **Splitting.** Let *M* be the default container type's maximum weight, or, when that is zero,
   the total weight plus one — a value deliberately chosen so that the division below yields a
   single parcel.

   ```formula
   number of full parcels = integer part of ( total weight ÷ M )
   remainder              = total weight − M × number of full parcels
   parcel weights         = M repeated ( number of full parcels ) times,
                            followed by the remainder when the remainder is not zero
   ```

5. **Spreading the value.** Let *n* be the number of parcel weights.

   ```formula
   declared value per parcel = total declared value ÷ n
   ```

   The division is not rounded.

6. **Commodities.** The commodity list is built once, from the order, by the rule of section 9.2,
   and is then divided by *n*:

   ```formula
   declared unit value of each commodity = original declared unit value ÷ n
   whole-unit quantity of each commodity = maximum of 1 and
                                           the integer part of ( original whole-unit quantity ÷ n )
   ```

   The same commodity list object is attached to every parcel, so all parcels of one order declare
   the same goods.

7. **Assembly.** One Delivery Parcel is produced per parcel weight, each carrying the commodity
   list, that weight, the default container type, the declared value per parcel, the company
   currency of the order and a reference back to the order.

### 9.2 Commodities of an order

For every order line that is not the shipping charge line, is not a display line and whose product
is of kind `consu`:

```formula
exact quantity      = ordered quantity converted into the product's reference unit
whole-unit quantity = maximum of 1 and ( exact quantity rounded to zero decimals, half away from zero )
declared unit value = the line's tax-inclusive unit price after discount
                    = line total including taxes ÷ ordered quantity
country of origin   = the product's country-of-origin code,
                      or the country code of the order's warehouse address when the product has none
```

### 9.3 Worked example

An order carries 7 pieces of a product weighing 1.000 kilogram whose unit cost is 20.00, sold at
30.00 each with a ten per cent tax, and the default container type has a base weight of 0.500
kilogram and a maximum weight of 3.000 kilograms. Company currency and product currency are the
same.

```formula
total declared value   = 7 × 20.00 = 140.00
total weight           = 7 × 1.000 + 0.500 = 7.500 kg
M                      = 3.000 kg
number of full parcels = integer part of ( 7.500 ÷ 3.000 ) = 2
remainder              = 7.500 − 3.000 × 2 = 1.500 kg
parcel weights         = 3.000 kg, 3.000 kg, 1.500 kg
n                      = 3
declared value per parcel = 140.00 ÷ 3 = 46.666666…
```

The single commodity, before division, is 7 whole units at a declared unit value of
30.00 × 1.10 = 33.00. After division by 3:

```formula
declared unit value = 33.00 ÷ 3 = 11.00
whole-unit quantity = maximum of 1 and integer part of ( 7 ÷ 3 ) = maximum of 1 and 2 = 2
```

Three parcels are produced, each declaring 2 units at 11.00 and each carrying a declared value of
46.666666…. Note that three parcels of two units declare six of the seven units; the division is a
proportional split and not an exact allocation. This is recorded as **compatibility finding**
DSH-052.

### 9.4 Worked example — no maximum weight

The same order with a default container type whose maximum weight is zero.

```formula
M                      = 7.500 + 1 = 8.500 kg
number of full parcels = integer part of ( 7.500 ÷ 8.500 ) = 0
remainder              = 7.500 − 8.500 × 0 = 7.500 kg
parcel weights         = 7.500 kg
n                      = 1
```

One parcel weighing 7.500 kilograms carrying the whole declared value of 140.00 and 7 units at
33.00.

---

## 10. Parcels built from a Transfer

### 10.1 A return transfer

A transfer that is a return produces exactly one parcel:

```formula
weight = estimated weight of the transfer (section 8.5) + base weight of the default container type
```

It carries the commodities of every move line of the transfer, the default container type, a
declared value of zero, the company currency and a reference back to the transfer.

### 10.2 An ordinary transfer

1. **One parcel per package.** For every distinct package named as the destination package of a
   move line of the transfer:

   ```formula
   weight = the package's shipping weight when it is set, otherwise the package's computed weight

   declared value = Σ over the quantities stored in the package of
                      ( stored quantity × unit cost of its product, converted as in section 9 step 1 )
   ```

   The parcel carries the commodities built from that package's move lines, the package's own
   container type, the package's name, the declared value, the company currency and a reference
   back to the transfer.

2. **One parcel for the loose goods.** When the transfer's bulk weight is not zero, one further
   parcel is produced carrying:

   ```formula
   weight         = the transfer's bulk weight
   declared value = Σ over every move line of the transfer of
                      ( moved quantity × unit cost of its product, converted as in section 9 step 1 )
   ```

   with the commodities of every move line of the transfer, the default container type and the
   reproduced name `Bulk Content`.

   Note that this parcel's declared value is computed over **every** move line, including the ones
   already inside a package, so a transfer that mixes packed and loose goods declares the packed
   goods twice. This is recorded as **compatibility finding** DSH-053.

3. **Refusal.** When the bulk weight is zero **and** no package was found, the operation is refused
   with "The package cannot be created because the total weight of the products in the picking is
   0.0 <weight unit label>".

### 10.3 Commodities of a set of move lines

Only move lines whose product is of kind `consu` are considered. They are grouped by product, and
for each group:

```formula
exact quantity      = Σ over the group's lines of
                        ( moved quantity converted into the product's reference unit )
whole-unit quantity = maximum of 1 and ( exact quantity rounded to zero decimals, half away from zero )
declared unit value = ( Σ over the group's lines of the line's sale value ) ÷ whole-unit quantity
country of origin   = the product's country-of-origin code, or the country code of the address of
                      the warehouse of the operation type of the first line's transfer
```

The line's sale value is the quantity defined in section 11.

### 10.4 Worked example

A transfer ships 3 units of a product sold at 100.00 each with a twenty per cent tax, packed into
one package whose type has a base weight of 1.000 kilogram; the product weighs 2.000 kilograms per
unit; the package's shipping weight has not been typed; the product's unit cost is 60.00 and the
company and product currencies are the same.

```formula
package computed weight = 1.000 + 2.000 × 3 = 7.000 kg
parcel weight           = 7.000 kg
declared value          = 3 × 60.00 = 180.00
commodity exact quantity      = 3
commodity whole-unit quantity = 3
commodity sale value          = 3 × 100.00 × 1.20 = 360.00
commodity declared unit value = 360.00 ÷ 3 = 120.00
```

One parcel is produced. The transfer's bulk weight is zero, so no second parcel is produced and the
refusal of step 3 does not apply because a package was found.

---

## 11. The value of a moved quantity

The sale value of a move line is what the delivery slip prints and what a commercial invoice
declares.

### 11.1 When the move comes from a sales order line for the same product

1. Build a taxation base from the sales order line, which carries the line's unit price, discount,
   taxes and currency.
2. Replace its quantity by the move line's moved quantity converted into the **sales order line's**
   unit of measure.
3. Ask [`../taxes/`](../taxes/) for the tax details of that base.
4. Take the raw tax-inclusive total in the line currency.
5. Round it by the sales order line's currency.

```formula
sale value = round( tax-inclusive total of the sales order line recomputed for the moved quantity,
                    by the sales order line's currency )
```

### 11.2 When there is no such sales order line

This happens for the components of a kit and for any move created outside a sale.

```formula
sale value = the product's sales price × ( moved quantity converted into the product's reference unit )
```

No tax is applied and no rounding is specified beyond the ordinary decimal arithmetic.

### 11.3 Worked examples

A sales order line sells 2 units at 750.00 with a tax of fifteen per cent excluded from the price.
The transfer moves both units.

```formula
sale value = 2 × 750.00 × 1.15 = 1725.00
```

The same line is delivered by two serial numbers, so two move lines of 1 unit each:

```formula
sale value of each line = 1 × 750.00 × 1.15 = 862.50
```

A sales order line sells 180 units at 1.49 with a tax of fifteen per cent excluded from the price,
in a currency whose smallest unit is one hundredth. The full quantity is moved:

```formula
sale value = 180 × 1.49 × 1.15 = 308.43
```

Only 150 units are moved:

```formula
sale value = 150 × 1.49 = 223.50
             223.50 × 1.15 = 257.025
             rounded by the line currency to two decimals, half up = 257.03
```

---

## 12. Package weights and the put-in-pack default

### 12.1 The computed weight of a package

Two computations exist and they differ.

**Without a transfer in context** — the weight of the package as it stands in the warehouse:

```formula
weight = base weight of the package's container type
       + Σ over every package contained at any depth of ( base weight of its container type )
       + Σ over every quantity contained at any depth of ( stored quantity × unit weight of its product )
```

**With a transfer in context** — the weight of the package as this transfer will hand it over:

```formula
weight = base weight of the package's container type
       + Σ over the move lines of this transfer destined to this package of
           ( moved quantity converted into the product's reference unit × unit weight )
       + Σ over every child package this transfer sends to the same place of
           ( base weight of its container type
             + Σ over the move lines of this transfer destined to that child of
                 ( moved quantity in the reference unit × unit weight ) )
```

The second form exists because a package being filled by an unvalidated transfer holds no stored
quantities yet: its content is still described by move lines.

**Worked example — nested packages holding stored quantities.** Products weigh 2.000 and 5.000
kilograms. Container types have base weights of 1.000 (small box), 4.000 (big box) and 10.000
(pallet). Box A holds 5 units of the first product, box B holds 3 units of the second, both boxes
are inside the big box, and the pallet holds the big box plus 1 unit of the second product.

```formula
box A   = 1.000 + 5 × 2.000 = 11.000 kg
box B   = 1.000 + 3 × 5.000 = 16.000 kg
big box = 4.000 + 11.000 + 16.000 = 31.000 kg
pallet  = 10.000 + 31.000 + 1 × 5.000 = 46.000 kg
```

**Worked example — the same nesting during an unvalidated transfer.** The transfer moves 2 units of
the first product and 2 units of the second. Box A receives the first product, box B the second,
both go into the big box and the big box onto the pallet.

```formula
box A   = 1.000 + 2 × 2.000 = 5.000 kg
box B   = 1.000 + 2 × 5.000 = 11.000 kg
big box = 4.000 + 5.000 + 11.000 = 20.000 kg
pallet  = 10.000 + 20.000 = 30.000 kg
```

### 12.2 The default shipping weight proposed by the put-in-pack dialogue

```formula
proposed shipping weight =
      base weight of the chosen container type,
        or, when no container type was chosen, the base weight of the chosen existing package's type,
        or zero
    + the chosen existing package's own shipping weight, when an existing package was chosen
    + Σ over the move lines being packed of
        ( quantity converted into the product's reference unit × unit weight of the product )
    + Σ over the packages being packed of ( their shipping weight )
```

**Worked example.** Two move lines are packed: 5 units of a product weighing 2.400 kilograms and 5
units of a product weighing 0.300 kilograms. No container type is chosen.

```formula
proposed shipping weight = 0.000 + 2.400 × 5 + 0.300 × 5 = 13.500 kg
```

Only the first line is marked as picked, so only it is offered for packing:

```formula
proposed shipping weight = 0.000 + 2.400 × 5 = 12.000 kg
```

**Worked example — packing two packages into one.** Two packages already carry shipping weights of
15.000 and 3.000 kilograms and the container type chosen for the outer package has no base weight:

```formula
proposed shipping weight = 0.000 + 15.000 + 3.000 = 18.000 kg
```

**Worked example — packing across two transfers of a batch.** Two transfers each move 1 unit of a
product weighing 1.000 kilogram; the chosen container type has a base weight of 1.000 kilogram:

```formula
proposed shipping weight = 1.000 + 1.000 × 1 + 1.000 × 1 = 3.000 kg
```

### 12.3 The maximum-weight warning

Whenever the container type, the existing package or the shipping weight changes in the dialogue:

```formula
limit = maximum weight of the chosen container type,
        or, when no container type was chosen, the maximum weight of the chosen package's type

warn = ( the carrier kind is set ) AND ( limit ≠ 0 ) AND ( shipping weight > limit )
```

The warning is titled "Package too heavy!" and its body is "The weight of your package is higher
than the maximum weight authorized for this package type. Please choose another package type." when
a container type was chosen, and "The weight of your package is higher than the maximum weight
authorized for its package type. Please choose another package." when an existing package was
chosen. It is a warning, not a refusal: the user may proceed.

### 12.4 The carrier kind of a package

```formula
carrier kind = "none"                       when the Delivery Method's provider kind is
                                            "fixed" or "base_on_rule"
carrier kind = the provider kind's own value otherwise
```

The mapping exists because the two core kinds mean *no carrier integration*, and the Package Type's
carrier field spells that `none`.

Before the mapping, two conditions are checked over the set of lines being packed: they must name
exactly one Delivery Method, and every line must name one. When either fails the packing is refused
with "You cannot pack products into the same package when they have different carriers (i.e. check
that all of their transfers have a carrier assigned and are using the same carrier)."

---

## 13. Carrier propagation and the transfer grouping key

### 13.1 At the creation of a transfer

When moves are grouped into a new transfer, and at least one of their rules carries the
carrier-propagation flag:

```formula
candidate method = the Delivery Method of the sales orders referenced by the moves,
                   when exactly one such method exists, otherwise nothing

candidate reference = the tracking reference of the first origin transfer of the moves
                      that carries one, otherwise nothing

when the origin transfers of the moves name exactly one Delivery Method,
     the candidate method is replaced by that one
```

The candidate method is written onto the new transfer when it is not empty, and the candidate
reference likewise. The replacement in the third clause exists so that a method changed on a
transfer, rather than on the order, is what propagates.

When no rule carries the flag, nothing is written.

**Worked example — two orders with different methods feeding one receipt.** A receipt's move
references two sales orders carrying two different methods. Exactly one method does not exist, so
the candidate method is empty. The origin transfers of a receipt's move are none, so the third
clause does not apply either. The internal transfer created by the push rule receives no method,
and the validation does not fail.

### 13.2 At the validation of a transfer

After a transfer is successfully validated, and only when it carries a Delivery Method:

```formula
next transfers = the transfers of the destination moves of this transfer's moves,
                 excluding the transfers that return this one

targets = next transfers that carry no Delivery Method
          and at least one of whose moves' rules carries the carrier-propagation flag

each target receives this transfer's Delivery Method and this transfer's tracking reference
```

This second mechanism exists because the method may be set on a transfer after that transfer was
created, in which case the mechanism of section 13.1 has already run.

**Worked example — a three-step delivery.** A warehouse delivers in three steps and every rule of
the route carries the propagation flag. A quotation is given a method and confirmed.

1. The picking transfer is created with the method, because the order names exactly one.
2. Validating the picking transfer copies the method and the empty tracking reference onto the
   packing transfer.
3. Validating the packing transfer copies them onto the shipping transfer.

**Worked example — the flag cleared on one rule.** The same route with the propagation flag cleared
on the third rule. A method and a tracking reference are set by hand on the first transfer. After
the first validation the second transfer carries both. After the second validation the third
transfer carries neither, because none of its moves' rules carries the flag.

### 13.3 The grouping key

The key that decides which moves share a transfer is extended with the Delivery Method of the
move's sales order. Two moves of two orders carried by two different methods therefore never end up
in the same transfer, even when every other component of the key agrees.

---

## 14. Batch weight caps

Three guards are added to automatic batching. All three are skipped when the operation type's
maximum batch weight is zero.

**Guard one — a transfer may join another transfer in a new batch:**

```formula
weight of this transfer + weight of the candidate transfer ≤ maximum batch weight
```

**Guard two — a transfer may join an existing batch:**

```formula
Σ over the transfers already in the batch of ( their weight ) + weight of the candidate
    ≤ maximum batch weight
```

**Guard three — a line may join a wave:**

```formula
Σ over the moves already in the batch of ( their weight ) + weight of the candidate line
    ≤ maximum batch weight
```

In addition, when the operation type groups batches by carrier:

- the set of transfers that may be batched with this one is narrowed to those carrying the same
  Delivery Method, where *no method* is itself a value that must match;
- the set of batches this transfer may join is narrowed to batches at least one of whose transfers
  carries the same Delivery Method, again with *no method* matching *no method*;
- the automatic batch's description is extended with the Delivery Method's name, preceded by a
  comma and a space when the description was not empty.

**Worked example.** The outgoing operation type caps a batch at 30 kilograms and groups by carrier.
Three transfers weigh 12, 12 and 9 kilograms and all carry the same method.

1. The first two are batched: 12 + 12 = 24 ≤ 30.
2. The third is offered to that batch: 24 + 9 = 33 > 30, so it is refused and starts a batch of its
   own.

**Worked example — the carrier set after confirmation.** A transfer is confirmed with no method and
a second with a method; the method is then set on the first one before validating it. Because the
method is compared at the moment the batch is formed, the two transfers produced at the next step
of the route carry the same method and are batched together.

---

## 15. The collection point record and the store distance

### 15.1 Preparing a store's collection point record

1. **Coordinates.** When the store address's latitude *and* longitude are both exactly zero, the
   address is geolocated by [`../contacts-and-organizations/`](../contacts-and-organizations/).
   When they are still both zero afterwards, the pair 1000 and 1000 is written instead. Those are
   deliberately impossible coordinates: because the geolocation is only attempted when both values
   are zero, writing them stops the system from geolocating a badly formed address over and over.
2. **Values.** The record is assembled with these members:

   | Member | Value |
   |---|---|
   | `id` | The warehouse's identifier |
   | `name` | The store address's name |
   | `street` | The store address's street, or the empty string |
   | `city` | The store address's city, or the empty string |
   | `state` | The code of the store address's region, or the empty string |
   | `zip_code` | The store address's postal code, or the empty string |
   | `country_code` | The store address's country code |
   | `latitude` | The store address's latitude |
   | `longitude` | The store address's longitude |
   | `opening_hours` | The mapping of section 15.2 |

3. **Badly formed address.** When any of those members cannot be read from the address, the record
   is empty and the store is skipped by the selector.

### 15.2 Opening hours

```formula
for each day index from 0 (Monday) to 6 (Sunday): an empty list

for each attendance line of the store's working schedule
    whose period is "morning", "afternoon" or "full_day":
        append to the list of that line's day index the text
        ( start hour formatted as hours and minutes )
        + " - "
        + ( end hour formatted as hours and minutes )
```

An attendance line whose period is `lunch` is skipped, so a store that is closed at midday shows
two ranges for that day. When the store has no working schedule, the mapping is empty.

**Worked example.** A store's schedule carries three Monday lines: a morning line from 8 to 12, a
lunch line from 12 to 13 and an afternoon line from 13 to 17. The mapping is:

| Day index | Ranges |
|---|---|
| 0 | `08:00 - 12:00`, `13:00 - 17:00` |
| 1 to 6 | *(empty)* |

### 15.3 The distance between two addresses

The distance is the great-circle distance on a sphere of radius 6371 kilometres, computed by the
half-versed-sine formula. Let *φ₁*, *λ₁* be the latitude and longitude of the first address in
degrees and *φ₂*, *λ₂* those of the second.

```formula
Δφ = ( φ₂ − φ₁ ) in radians
Δλ = ( λ₂ − λ₁ ) in radians

a = sin( Δφ ÷ 2 ) × sin( Δφ ÷ 2 )
  + cos( φ₁ in radians ) × cos( φ₂ in radians ) × sin( Δλ ÷ 2 ) × sin( Δλ ÷ 2 )

distance in kilometres = 2 × 6371 × arctangent2( square root of a, square root of ( 1 − a ) )
```

No rounding is applied. The result is used only as a sort key: the collection points are returned
in ascending order of distance.

**Worked example — the same point.** *φ₁* = *φ₂* = 1.0 and *λ₁* = *λ₂* = 2.0.

```formula
Δφ = 0, Δλ = 0
a  = 0 + cos(0.017453…) × cos(0.017453…) × 0 = 0
distance = 2 × 6371 × arctangent2( 0, 1 ) = 0.0 km
```

**Worked example — one degree of latitude apart.** *φ₁* = 0.0, *φ₂* = 1.0, *λ₁* = *λ₂* = 0.0.

```formula
Δφ = 0.0174532925 radians
a  = sin( 0.00872664626 )² = 0.0087265355² = 0.0000761524…
distance = 2 × 6371 × arctangent2( 0.00872665…, 0.99996192… )
         = 2 × 6371 × 0.00872664626
         = 111.194926… km
```

### 15.4 The stock figure attached to a collection point

When the selector is opened from a product page, each collection point carries the stock of that
product in that store; when it is opened from the checkout page, it carries whether the whole cart
can be supplied by that store.

```formula
from a product page:
    free quantity  = the product's free quantity in that store
    in stock       = ( free quantity > 0 ) OR the product allows selling out of stock
    show quantity  = the product shows its availability
                     AND free quantity > 0
                     AND the product's display threshold ≥ free quantity
    quantity       = free quantity

from the checkout page:
    in stock = the cart has no line that the store cannot supply (section 16)
```

---

## 16. The per-store stock check

Given a cart and a store, the check produces the mapping of order lines that the store cannot
supply to the greatest quantity it can supply for each of them.

1. Group the order lines by product.
2. Skip a product that is not storable, and skip a product that allows being sold out of stock.
3. Read the product's free quantity in that store; call it *f*.
4. For each line of that product, in the order the lines appear:

   ```formula
   available in the line's unit =
       maximum of 0 and
       the integer part of ( f converted from the product's reference unit into the line's unit,
                             rounded downwards )

   when the line's ordered quantity > available in the line's unit:
       record the line with that available quantity
       set the line's warning to "<available>/<ordered> available at this location"
         where the first placeholder is the available quantity in the line's unit
         and the second is the ordered quantity truncated to a whole number

   f = f − ( the line's ordered quantity converted into the product's reference unit )
   ```

The subtraction at the end runs whether or not the line was short, so later lines of the same
product see what the earlier ones consumed. The downward rounding exists because only whole units
can be sold.

The cart is *in stock* for that store when the mapping is empty.

**Worked example — enough stock.** The store holds 10 pieces; the cart carries one line of 5
pieces. 5 ≤ 10, nothing is recorded, the cart is in stock.

**Worked example — not enough stock.** The store holds 10 pieces; the cart carries one line of 15
pieces. The line is recorded with an available quantity of 10 and receives the warning "10/15
available at this location".

**Worked example — two lines of the same product in two units.** The store holds 10 pieces. The
cart carries one line of 1 pack of six and one line of 5 pieces, and the pack-of-six line comes
first.

```formula
line 1: available in packs of six = integer part of ( 10 ÷ 6 ) rounded down = 1
        ordered 1 ≤ 1, nothing recorded
        f = 10 − 6 = 4
line 2: available in pieces = 4
        ordered 5 > 4, recorded with 4, warning "4/5 available at this location"
```

**Worked example — the same two lines when the store holds enough.** The cart carries 4 pieces and
1 pack of six, the pieces line first.

```formula
line 1: available in pieces = 10, ordered 4 ≤ 10, nothing recorded, f = 10 − 4 = 6
line 2: available in packs of six = integer part of ( 6 ÷ 6 ) = 1, ordered 1 ≤ 1, nothing recorded
```

The cart is in stock.

---

## 17. The quantity a product page advertises

When collection in store is enabled for a website — the website names a warehouse and a published
in-store Delivery Method exists — the free quantity a product page advertises is widened.

```formula
base quantity = the free quantity in the website's own warehouse

when there is no cart, or the cart carries no Delivery Method:
    advertised quantity = maximum of ( base quantity,
                                       the greatest free quantity among the in-store method's stores )

when the cart's Delivery Method is the in-store kind and a collection point is chosen:
    advertised quantity = the free quantity in the order's warehouse

otherwise:
    advertised quantity = base quantity
```

The same widening is applied to the quantity used when a line is added to the cart: with no method
chosen, the greatest quantity among the in-store stores is what limits the line.

**Worked example.** The website's warehouse holds nothing; the in-store method has two stores
holding 10 and 15 pieces.

| Situation | Advertised |
|---|---|
| No cart | maximum of 0 and 15 = 15 |
| Cart with no method | maximum of 0 and 15 = 15 |
| Cart with the in-store method and the first store chosen | 10 |
| Cart with an ordinary method | 0 |
| In-store method unpublished | 0 |

---

## 18. The tracking link

### 18.1 The two core kinds

```formula
tracking link = the method's tracking-link pattern
                with every occurrence of the placeholder <shipmenttrackingnumber>
                replaced by the transfer's tracking reference
```

The link is empty when the method has no pattern or the transfer has no reference.

**Worked example.** The pattern is `https://example.com/track/<shipmenttrackingnumber>` and the
reference is `1Z999AA10123456784`. The link is
`https://example.com/track/1Z999AA10123456784`.

### 18.2 The parcel-point network

When the method is a parcel-point network method, both core kinds are overridden and the link
becomes the network's own tracking address, whose three parameters are the method's brand code, the
transfer's tracking reference and a language. The address is reproduced because it is part of the
integration contract:

```formula
https://www.mondialrelay.com/public/permanent/tracking.aspx?ens=<brand>&exp=<reference>&language=<language>
```

```formula
language = the part of the destination contact's language code before the underscore,
           or "fr" when the contact has no language
```

**Worked example.** Brand code `BDTEST  `, reference `12345678`, contact language `nl_BE`. The
language becomes `nl` and the address is
`https://www.mondialrelay.com/public/permanent/tracking.aspx?ens=BDTEST  &exp=12345678&language=nl`.

### 18.3 Several tracking links on one transfer

A carrier integration may store, instead of a single link, a structured list of pairs of a label
and a link. The transfer reads the stored tracking link as such a list; when it parses, the links
are posted in the transfer's history under the heading "Tracking links for shipment:" and an
informational dialogue is opened saying "You have multiple tracker links, they are available in the
chatter." When it does not parse, the stored value is opened directly in a new window titled
"Shipment Tracking Page".

---

## 19. The weight segment of the parcel barcode

The parcel label and the parcel barcode report append a weight segment to the barcode of the
package, following the application-identifier convention for net weight.

1. Take the package's shipping weight when it is set, otherwise its computed weight.
2. Scale it by the rounding step of the weight unit and truncate to a whole number:

   ```formula
   scaled weight = integer part of ( weight ÷ rounding step of the weight unit )
   ```

3. When the decimal representation of the scaled weight is longer than six characters, no segment
   is appended at all.
4. Otherwise append, in this order:

   ```formula
   marker    = "310" when the weight unit is the kilogram, otherwise "320"
   decimals  = the number of characters after the decimal point in the decimal
               representation of the rounding step
   padding   = enough zeroes to bring the scaled weight to six characters
   segment   = marker + decimals + padding + scaled weight
   ```

**Worked example — kilograms with a rounding step of 0.01.** The package's shipping weight is
13.5 kilograms.

```formula
scaled weight = integer part of ( 13.5 ÷ 0.01 ) = 1350
length of "1350" = 4, which is at most 6
marker   = "310"
decimals = the rounding step written as "0.01" has 2 characters after the point → "2"
padding  = "00"
segment  = "310" + "2" + "00" + "1350" = "3102001350"
```

**Worked example — pounds with a rounding step of 0.001.** The package's computed weight is 4.2
pounds, and no shipping weight has been typed.

```formula
scaled weight = integer part of ( 4.2 ÷ 0.001 ) = 4200
marker   = "320"
decimals = the rounding step written as "0.001" has 3 characters after the point → "3"
padding  = "00"
segment  = "320" + "3" + "00" + "4200" = "3203004200"
```

**Worked example — a weight too large for the segment.** A package weighing 12000 kilograms with a
rounding step of 0.001 gives a scaled weight of 12000000, whose decimal representation is eight
characters long. No segment is appended and the barcode carries only its other application
identifiers.

Beside the barcode the label prints "Shipping Weight: <weight> <weight unit label>" when a shipping
weight was typed, and "Weight: <weight> <weight unit label>" otherwise.

---

## 20. The currency conversions used by this domain

Four conversions occur, all of them performed by
[`../multi-currency/`](../multi-currency/) with the order's company and the order's date, falling
back to today's date when the order carries none.

| # | Where | From | To | Rounded |
|---|---|---|---|---|
| 1 | The price variable of the rule-based engine | The order currency | The company currency | Not rounded at this step |
| 2 | The result of the rule-based engine | The company currency | The order currency | Not rounded at this step |
| 3 | The fixed margin, before it is added | The company currency | The order currency | Not rounded at this step |
| 4 | The order total compared with the waiver threshold | The order currency | The company currency | Not rounded at this step |

The *company currency* in conversions 1 to 4 is the currency of the Delivery Method's company, and,
when the method carries no company, the currency of the main company. The *order currency* is the
currency of the order's price list.

When the two currencies are the same record, the conversion is skipped entirely rather than run
with a rate of one, so no rounding error can creep in.

The only rounding in the pipeline is step 6 of section 7.1: the charge is rounded once, by the
order currency, after the margins and before the waiver.

**Worked example.** The method has no company; the main company's currency is the first currency.
The order belongs to a company whose currency is the second currency and whose price list is in the
second currency. The rate on the order date is 0.5 second-currency units per first-currency unit.
The order total without carriage is 750.00 in the second currency; the rule-based engine has one
rule whose condition is `price` `>=` 0 and whose base amount is 15.00; the fixed margin is 10.00.

```formula
conversion 1: 750.00 ÷ 0.5 = 1500.00 in the first currency
rule engine:  15.00 + 0.00 × weight = 15.00 in the first currency
conversion 2: 15.00 × 0.5 = 7.50 in the second currency
conversion 3: 10.00 × 0.5 = 5.00 in the second currency
margins:      7.50 × ( 1 + 0 ) + 5.00 = 12.50
rounding:     12.50
```
