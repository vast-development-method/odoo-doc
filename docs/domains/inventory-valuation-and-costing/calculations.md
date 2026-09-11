# Calculations of the Inventory Valuation and Costing domain

Every formula, every algorithm, every rounding rule and every worked example of the
domain. Formulas are written as plain mathematics; quantities are named in words.

Read [glossary.md](glossary.md) first if any term is unfamiliar.

> **Reproduced text.** Quoted, bolded strings in this file are user-facing text the
> system emits character for character — error messages, selection labels, button
> labels, action titles. Where such a string contains an abbreviation it belongs to
> the emitted string, not to this specification's prose: "FIFO" stands for *first in
> first out*, "AVCO" for *average cost*, "WIP" for *work in progress*, "MOs" for
> *manufacturing orders*, "BoM" for *bill of materials*, `STJ` for the inventory
> valuation journal code and `LC/` for the landed cost sequence prefix.

---

## 0. Precision, rounding and units

### 0.1 The three precisions that matter

| Name | Where it lives | Default | What it governs |
|---|---|---|---|
| Currency decimal places | On the currency record of the company | 2 for most currencies | Every monetary field: the value of a goods movement, the remaining value, the total value, the amounts of journal items, the amounts of landed cost lines and adjustment lines. |
| Product Price decimal precision | A named decimal precision record | 2 | The **display** of the unit cost, and the comparison tolerance used in two places: the test that decides whether a cost-of-goods-sold line is worth creating, and the test that decides whether a price difference is worth posting. |
| Product Unit decimal precision / unit-of-measure rounding | A named decimal precision record and the rounding of each unit of measure | 2 / 0.01 | Every comparison of a quantity to zero or to another quantity. |

**The unit cost of a product and of a lot is stored at full floating precision.** It is
*displayed* with at least the Product Price precision, but it is not truncated on
storage. This matters: after receiving one unit at 1.00, one at 1.00 and one at 1.00 and
then correcting the first movement to 2.00, the unit cost becomes
4 ÷ 3 = 1.333333… and the product's total value is exactly 4.00, while the sum of the
three movements' *remaining values* is 3.99 — because each remaining value is rounded to
the currency separately (1.33 + 1.33 + 1.33).

### 0.2 Rounding functions used

```formula
round_to_currency( amount ) = the amount rounded to the decimal places of the currency,
                              half away from zero
```

```formula
round_half_up( amount, step ) = the amount rounded to the nearest multiple of the step,
                                ties away from zero
```

```formula
is_zero_for_currency( amount ) = true when round_to_currency( amount ) equals 0
```

```formula
compare_quantities( a, b ) = −1, 0 or +1, comparing a and b after rounding both
                             to the rounding step of the unit of measure
```

A monetary field is rounded to the currency **at the moment it is written**. Intermediate
arithmetic is not rounded unless the text says so.

### 0.3 Units of measure

Every valuation quantity is expressed in the **product reference unit of measure**. A
goods movement records its quantity in the movement unit of measure; the conversion is:

```formula
reference_quantity = movement_quantity × movement_unit_factor ÷ reference_unit_factor
```

**Worked example.** A product whose reference unit is the kilogram is received with a
movement expressed in grams, quantity 11. The gram factor relative to the kilogram is
0.001, so the reference quantity is 11 × 0.001 = 0.011, which rounds to 0.01 kilogram at
the product-unit precision. The movement's own quantity stays 11 grams; the valuation
quantity is 0.01 kilogram.

**Worked example.** A product whose reference unit is the unit is received with a
movement expressed in dozens, quantity 1, and the product's unit cost is 10. The
reference quantity is 12, and the value of the movement is 12 × 10 = 120.

---

## 1. The value of a goods movement: the priority chain

### 1.1 Statement

The value of one completed goods movement is not a single formula; it is the result of
consulting **six sources in a fixed order**, each of which may claim a part of the
movement's valued quantity and contribute a value for that part. When a source claims
part of the quantity, the remainder is passed to the next source. A seventh source, the
*extra* source, is added on top of the whole quantity rather than claiming any of it.

Let *valued quantity* be the quantity of the movement that counts for valuation, as
defined in [entities.md](entities.md#valued-quantity-of-a-movement).

### 1.2 Algorithm

**Preconditions.** Exactly one movement. Optional inputs: a forced unit cost, an as-of
instant, a flag to ignore manual corrections, and a flag to suppress the extra source.

**Postconditions.** A value, the valued quantity and a human-readable justification made
of one line per contributing source in the order consulted.

1. Set *remaining quantity* and *valued quantity* to the movement's valued quantity. Set
   *value* to zero. Start an empty list of justification lines. Set *add extra* to true
   unless the caller suppressed it.

2. **Source 0 — manual correction** (skipped when the caller asked to ignore manual
   corrections). Find the latest valuation history record attached to this movement whose
   date is not after the as-of instant (when there is one), ties broken by the highest
   identifier. If one exists:
   - its value becomes a contribution to *value*;
   - it claims the **whole** remaining quantity (its quantity contribution equals the
     remaining quantity at that moment);
   - *add extra* is turned **off**, because a manual correction is understood to be the
     final total value of the movement, landed costs included;
   - the justification gains the line **"Adjusted on _the date_ by _the user name_"**,
     and, when the record carries a description, a newline and that description.

3. **Source 1 — accounting documents (vendor bills and vendor credit notes).** See
   [section 1.3](#13-value-from-vendor-bills). Contributes a value and a claimed
   quantity; subtract the claimed quantity from the remaining quantity.

4. **Source 2 — production.** Only when the movement belongs to a manufacturing order.
   The contribution is:

   ```formula
   production_value = remaining_quantity × move_price_unit
   ```

   where the movement's unit price was set by the manufacturing cost computation (see
   [section 8](#8-production-value)). It claims the whole remaining quantity. The
   justification gains **"_the formatted value_ for _the quantity_ _the unit_ from _the
   manufacturing order name_"**.

5. **Source 3 — quotations (purchase order lines).** Only when the movement is linked to
   a purchase order line. See [section 1.4](#14-value-from-a-purchase-order-line).

6. **Source 4 — returns.** Only when the movement has an originating returned movement
   **and that originating movement is outgoing**. The contribution is:

   ```formula
   return_value = 0                                              when the originating valued quantity is zero
   return_value = origin_value × remaining_quantity
                  ÷ origin_valued_quantity                       otherwise
   ```

   It claims the whole remaining quantity. The justification gains **"Value based on
   original move _the originating reference_"**. This is what makes a customer return
   reverse the delivery at the price the goods left at, rather than revalue them at
   today's cost.

7. **Source 5 — the product cost.** Always reached when a remaining quantity is left.
   Determine the unit cost as follows, in order:
   - if an as-of instant is given **and** the costing method is `standard`: the forced
     cost if one was given, else the product's current unit cost, else the product's unit
     cost as of that instant from the valuation history;
   - else if the product is valuated by lot **and** the movement carries exactly one lot:
     that lot's cost;
   - else if no forced cost was given, an as-of instant is given and the costing method
     is `fifo`: the movement's own current value divided by its valued quantity (zero
     when the valued quantity is zero);
   - otherwise: whatever forced cost was given.

   The contribution is:

   ```formula
   cost_value = ( chosen_unit_cost or product_unit_cost ) × remaining_quantity
   ```

   — that is, when the steps above left the chosen unit cost empty or zero, the product's
   current unit cost is used. The justification gains **"_the quantity_ _the reference
   unit name_ at product's cost"**.

8. **Source 6 — extra (landed costs).** Only when *add extra* is still on. It does
   **not** claim quantity; it is added to the value computed so far. See
   [section 7.6](#76-how-a-landed-cost-reaches-the-value-of-a-movement).

9. Return the value, the valued quantity, and the justification lines joined by
   newlines.

### 1.3 Value from vendor bills

**Precondition.** The movement is linked to a purchase order line. Otherwise this source
contributes nothing.

1. If the as-of instant is an instant with a time component, reduce it to a plain date,
   because bill lines carry a date without a time.
2. Walk every bill line of that purchase order line. Skip a line whose date is after the
   as-of date. Skip a line whose document is not posted. For each remaining line,
   remember its identifier and accumulate:
   - **vendor bill**: add the line's quantity, converted from the line unit of measure to
     the product reference unit of measure, to *billed quantity*; add the line's value to
     *billed value*;
   - **vendor credit note**: subtract both.
3. If *billed quantity* is not positive, contribute nothing.
4. Compute the quantity already absorbed by **other** movements of the same purchase
   order line: walk the movements of the line, skipping this movement itself, skipping
   movements of a different product, and skipping movements that are **later** than this
   one (a later date, or the same date with a higher identifier). Add the valued quantity
   of each earlier movement that is incoming or a drop shipment; subtract the valued
   quantity of each earlier movement that is outgoing.

   > Note the arithmetic of the outgoing branch: the subtraction is applied to the
   > negation of the valued quantity, so an earlier **outgoing** movement of the same
   > purchase order line **increases** the quantity considered already absorbed.

5. If *billed quantity* is not greater than the absorbed quantity, contribute nothing:
   the bill has already been consumed by earlier movements.
6. Otherwise, cut the absorbed part away, keeping the value proportional:

   ```formula
   available_value    = billed_value × ( billed_quantity − absorbed_quantity ) ÷ billed_quantity
   available_quantity = billed_quantity − absorbed_quantity
   ```

7. Contribute:

   ```formula
   claimed_quantity = min( remaining_quantity, available_quantity )
   contributed_value = available_value                                     when remaining_quantity ≥ available_quantity
   contributed_value = remaining_quantity × available_value
                       ÷ available_quantity                                otherwise
   ```

8. The justification gains **"_the formatted value_ for _the available quantity_ _the
   reference unit name_ from _the list of bill display names_"**.

**Value of one bill line.** The amount taken from a bill line is its subtotal converted
into the company currency at the **document's own rate**, then rounded to the company
currency:

```formula
bill_line_value = round_to_currency( line_subtotal ÷ line_currency_rate )
```

The currency rate is the rate stored on the line, so the conversion uses the rate that
was in force for that bill, not today's rate.

**Quantity of one bill line.**

```formula
bill_line_quantity = convert( line_quantity, line_unit_of_measure → product_reference_unit )
```

### 1.4 Value from a purchase order line

**Precondition.** The movement is linked to a purchase order line.

1. Compute the line's unit price for stock purposes, in the company currency, with the
   conversion date forced to the movement's date (see
   [section 1.5](#15-the-unit-price-of-a-purchase-order-line-for-stock)).
2. Clamp the quantity:

   ```formula
   claimed_quantity = min( remaining_quantity,
                           convert( move_quantity, move_unit → product_reference_unit ) )
   ```

3. The cost ratio is, by default, the claimed quantity itself; a manufacturing kit
   installation may replace it (see [section 8.4](#84-kit-price-unit)).
4. The contribution is:

   ```formula
   quotation_value = purchase_unit_price × cost_ratio
   ```

5. The justification gains **"_the formatted value_ for _the quantity_ _the reference
   unit name_ from _the purchase order display name_ (not billed)"**.

### 1.5 The unit price of a purchase order line for stock

```formula
step 1: price = discounted_unit_price
step 2: if the line carries taxes:
            quantity_for_tax = line_quantity or 1
            price = total_void( taxes applied to price for quantity_for_tax,
                                rounded globally ) ÷ quantity_for_tax
step 3: if the line unit of measure differs from the product reference unit:
            price = price ÷ line_unit_factor × reference_unit_factor
step 4: if the order currency differs from the company currency:
            price = convert( price, order_currency → company_currency,
                             at the conversion date, without rounding )
step 5: price = round_half_up( price, 10^(−Product Price precision) )
```

*total_void* is the part of the tax computation that represents non-deductible taxes
that must be capitalised into the cost. The conversion date is the date forced by the
caller (the movement's date when the caller is the valuation engine), else the order
date, else today.

**Worked example.** A purchase order line orders 5 packs of 6 units at 60.00 per pack,
with no taxes, in the company currency. The reference unit is the unit; the pack-of-six
factor relative to the unit is 6. Step 3 gives 60 ÷ 6 × 1 = 10.00 per unit. A receipt of
2 packs has a reference quantity of 12, so its value from the quotation source is
10.00 × 12 = 120.00.

---

## 2. Setting the value of movements on completion

### 2.1 Statement

The valuation of a set of movements is performed by one routine which is called twice
during completion — once for the outgoing movements before the state change, once for
the incoming and drop-shipment movements after it — and again whenever a correction
arrives. It also maintains the product unit cost.

### 2.2 Algorithm

**Input.** A set of movements, and optionally a *correction quantity* (a signed amount by
which an outgoing movement's quantity has just changed).

**Preconditions.** None; movements that do not qualify are silently skipped.

**Postconditions.** Each qualifying movement has its value written; the products and lots
touched have their unit cost brought up to date.

1. Start three empty collections: products to recompute, lots to recompute, and a map
   from product to the first in first out quantity already processed in this call.
2. Group the movements by company. For each company, with that company in scope:
   1. Start two empty maps: extra value per product, extra quantity per product.
   2. For each movement in the group, re-scoped to that company:
      - **If the movement is incoming:**
        1. Add its product to the products to recompute.
        2. If the product is valuated by lot and **any** line of the movement carries no
           lot or serial number, refuse with the error **"A lot/serial number is required
           for product '_the product display name_' as it has lot valuation enabled."**
        3. Otherwise add every lot of the movement's lines to the lots to recompute.
        4. Set the movement's value to the result of the priority chain
           ([section 1](#1-the-value-of-a-goods-movement-the-priority-chain)), evaluated
           with elevated privileges.
        5. If the "incremental unit cost recomputation" flag is on **and** the product is
           storable, accumulate the movement's value into the extra value of that product
           and its valued quantity into the extra quantity of that product.
        6. Continue to the next movement.
      - **If the movement does not count as outgoing**, skip it.
      - **If a correction quantity was supplied** (see
        [section 2.3](#23-outgoing-correction)): let *previous quantity* be the
        movement's quantity minus the correction. If the previous quantity is non-zero,
        scale the value and continue to the next movement.
      - **If the product is valuated by lot**: the value is the sum, over the movement's
        lines, of the line quantity in the reference unit multiplied by the **lot's**
        cost, or, for a line with no lot, by the **product's** unit cost. Continue.
      - **If the costing method is `fifo`**: the value is the first in first out
        valuation of the movement's valued quantity, computed with the quantity already
        processed for this product in this call subtracted from the stack size, so that
        several outgoing movements of the same product validated together consume the
        stack in sequence. Then add the valued quantity to the processed map.
      - **Otherwise** (standard price or average cost): the value is the product's
        current unit cost multiplied by the movement's valued quantity.
   3. Recompute the unit cost of every product in the products-to-recompute collection,
      passing the extra value and extra quantity maps
      ([section 3](#3-maintaining-the-product-unit-cost)).
   4. Recompute the unit cost of every lot in the lots-to-recompute collection.

**Failure conditions.** The missing-lot error above is the only failure raised by the
routine itself.

### 2.3 Outgoing correction

When the quantity of an already-completed outgoing movement changes, its value is not
recomputed from the costing method — that would consume the first in first out stack a
second time, or apply today's average instead of the average of the day. Instead the
existing value is scaled:

```formula
previous_quantity = new_move_quantity − correction_quantity
ratio             = correction_quantity ÷ previous_quantity
new_value         = old_value + ratio × old_value
```

which is algebraically

```formula
new_value = old_value × ( 1 + correction_quantity ÷ previous_quantity )
          = old_value × new_move_quantity ÷ previous_quantity
```

When the previous quantity is zero, no scaling is possible and the routine falls through
to the ordinary costing computation.

**Worked example.** A delivery of 10 units was valued at 100.00 under first in first out.
A line is edited so that the delivered quantity becomes 12; the correction quantity is
+2 and the previous quantity is 10. The ratio is 0.2 and the new value is
100.00 + 0.2 × 100.00 = 120.00.

**Worked example.** The same delivery is corrected down to 8. The correction quantity is
−2, the previous quantity is 10, the ratio is −0.2 and the new value is
100.00 − 20.00 = 80.00.

### 2.4 Which correction the caller uses

| Situation | Call |
|---|---|
| A line is added to, or edited on, an **incoming** movement | Full revaluation, no correction quantity. |
| A line is added to, or edited on, an **outgoing** movement | Correction quantity equal to the sum of the per-line quantity deltas over the lines not excluded for valuation. |
| A vendor bill is posted | Full revaluation of every incoming or drop-shipment movement reachable from the bill's lines. |
| A purchase order line's price, quantity or unit of measure changes | Full revaluation of every valued movement of that line. |
| A landed cost is validated | Full revaluation of every movement named by its adjustment lines. |
| A valuation history record is created for a movement | Full revaluation of that movement. |

---

## 3. Maintaining the product unit cost

### 3.1 Statement

The unit cost of a product is an input under standard price, a derived figure under
average cost, and a reporting figure under first in first out. This routine brings it up
to date.

### 3.2 Algorithm

**Input.** A set of products; optionally an *extra value* map and an *extra quantity* map
(product → amount), which together enable the incremental fast path.

1. For each product in the set:
   - If the product is valuated by lot **and** its costing method is not `standard`: set
     its unit cost to its computed average cost, writing with elevated privileges and the
     "disable automatic revaluation" flag, and skip the rest for this product. (Its cost
     is an aggregate of its lots' costs.)
   - Otherwise put it in the group of its costing method.
2. For each costing method group:
   - **Standard price**: do nothing. The cost is an input.
   - **Both other methods, when the extra maps were supplied**: take the products that
     appear in the extra value map, scope them to the current company and to the valued
     perimeter, and for each of them:

     ```formula
     previous_quantity = current_quantity_on_hand − added_quantity

     new_unit_cost = ( previous_quantity × old_unit_cost + added_value )
                     ÷ current_quantity_on_hand
                          when previous_quantity > 0 and current_quantity_on_hand > 0

     new_unit_cost = added_value ÷ added_quantity
                          when the first case does not apply and added_quantity ≠ 0

     (no change)          when neither case applies
     ```

     Write the new unit cost with the "disable automatic revaluation" flag. Remove these
     products from the group; the remaining ones go through the full path below.
   - **First in first out, full path**: for each product, let *quantity on hand* be the
     quantity in the valued perimeter. If it is positive, set the unit cost to
     *total value ÷ quantity on hand*. If it is not positive, find the most recent
     incoming movement of the product; if it exists and yields a non-zero unit price, set
     the unit cost to that unit price; otherwise leave the cost unchanged.
   - **Average cost, full path**: replay the average over the whole set with the
     recomputation forced ([section 4.3](#43-the-average-replay)) and write the resulting
     unit cost onto each product that the replay produced a figure for.

All writes use the "disable automatic revaluation" flag, so that the engine's own
recomputations never create valuation history records; only a human's write does.

### 3.3 Worked example of the incremental fast path

A product using average cost has 10 units on hand at a unit cost of 10.00. A receipt of
10 units valued at 120.00 completes, with no outgoing movement validated at the same
time, so the fast path is enabled. The added value is 120.00 and the added quantity is
10. After the receipt the quantity on hand is 20.

```formula
previous_quantity = 20 − 10 = 10
new_unit_cost = ( 10 × 10.00 + 120.00 ) ÷ 20 = 220.00 ÷ 20 = 11.00
```

### 3.4 Worked example of the fast path from zero

A product using average cost has nothing on hand and a unit cost of 10.00. A receipt of
10 units valued at 120.00 completes. Previous quantity is 0, so the first case does not
apply; the added quantity is 10, so:

```formula
new_unit_cost = 120.00 ÷ 10 = 12.00
```

### 3.5 The last incoming movement

```formula
last_incoming_move = the movement with the greatest ( date, identifier )
                     among the movements of the product whose incoming flag is set
                     and whose date is not after the as-of instant, when one is given
```

Its unit price is:

```formula
unit_price = total_value_of_the_selection ÷ total_valued_quantity_of_the_selection
unit_price = 0   when the total valued quantity is zero
```

(The selection is normally a single movement, so this reduces to *value ÷ valued
quantity*.)

---

## 4. Batch valuation routines

Three routines take a set of products and produce, for each, a unit-cost figure and a
total value. They may be run as of a past instant and may be restricted to one lot.

### 4.1 Standard price batch

```formula
unit_cost(product)  = product_unit_cost                                   when no as-of instant
unit_cost(product)  = value_of_the_latest_valuation_history_record        when an as-of instant is given
                      for that product not after the as-of instant,
                      falling back to product_unit_cost when there is none

total_value(product) = quantity_on_hand(product) × unit_cost(product)
```

The quantity on hand is the one already scoped by the caller (valued perimeter, company,
and as-of instant when given).

**Worked example.** A product using standard price had its cost set to 10.00 on day one
and to 12.00 on day five. It has 30 units on hand. Valued today, its unit cost is 12.00
and its total value is 360.00. Valued as of day three, its unit cost is 10.00 and its
total value is *the quantity on hand as of day three* multiplied by 10.00.

### 4.2 Selecting the latest valuation history record per product

**Input.** A set of products, an optional as-of instant, an optional lot.

1. Restrict to records of those products whose movement reference is **empty** (only cost
   changes, never movement corrections) and whose company is the company in the current
   scope.
2. If a lot was given, restrict to records whose lot is that lot **or** whose lot is
   empty. If no lot was given, restrict to records whose lot is empty.
3. If an as-of instant was given, restrict to records whose date is not after it.
4. Order by product, then date descending, then identifier descending, and keep the
   **first record per product**.

### 4.3 The average replay

**Statement.** The weighted average is not stored as a running figure; it is *replayed*
from the movements. The replay may start from the latest manual cost change rather than
from the beginning of time, which is what makes it affordable.

**Input.** A set of products; optionally an as-of instant, a lot, and a "force
recomputation" flag.

1. **Fast exit.** If there is no as-of instant and the force flag is off, return the
   product's current unit cost and *quantity on hand × unit cost* as the total value, for
   every product. No replay happens.
2. Build the movement filter: movements of the products in the set, of the company in
   the current scope, that are incoming **or** outgoing. When a lot is given, add "has a
   line carrying that lot". When an as-of instant is given, add "date not after it".
3. Find the latest manual cost change per product ([section 4.2](#42-selecting-the-latest-valuation-history-record-per-product)).
   Let *oldest anchor* be the earliest date among them. If every product in the set has
   an anchor, add "date not before the oldest anchor" to the movement filter — the replay
   can then start there instead of at the beginning of time.

   > This condition is deliberately strict: if even one product has no anchor, the filter
   > is not narrowed, because that product must be replayed from the beginning. Deleting
   > the valuation history of a product therefore makes the replay start from the
   > beginning of time for it, rather than producing a wrong figure.

4. Seed the running state from the anchors. For each product that has an anchor:
   - the running **unit cost** is the anchor's value;
   - the running **quantity** is the quantity of the lot (when a lot is given) or of the
     product (otherwise) as it stood **at the anchor's date**;
   - the running **value** is *anchor value × that quantity*;
   - the anchor's date is remembered as that product's replay start.
5. Fetch the movements ordered by product, then date, then identifier. Drop every
   movement whose date is **not after** its product's replay start (a movement exactly at
   the anchor instant is considered already reflected in the anchor).
6. Replay the surviving movements in order. For each movement, with *quantity*, *unit
   cost* and *value* being the running state of that movement's product (unit cost
   defaulting to *movement value ÷ movement valued quantity*, or zero when that quantity
   is zero, for a product with no state yet):

   **If the movement is incoming:**

   ```formula
   entering_quantity = movement_valued_quantity
   entering_value    = movement_value

   when a lot filter is active:
       lot_quantity   = movement_valued_quantity_for_that_lot
       entering_value = entering_value × lot_quantity ÷ entering_quantity
                        ( 0 when entering_quantity is 0 )
       entering_quantity = lot_quantity

   previous_quantity = quantity
   quantity          = quantity + entering_quantity

   when previous_quantity > 0:                       ← the ordinary case
       value     = value + entering_value
       unit_cost = value ÷ quantity

   when previous_quantity ≤ 0:                       ← recovering from negative stock
       unit_cost = entering_value ÷ entering_quantity   ( unchanged when entering_quantity is 0 )
       value     = unit_cost × quantity
   ```

   **If the movement is outgoing:**

   ```formula
   leaving_quantity = movement_valued_quantity
   leaving_value    = leaving_quantity × unit_cost

   when a lot filter is active:
       lot_quantity  = movement_valued_quantity_for_that_lot
       leaving_value = leaving_value × lot_quantity ÷ leaving_quantity
                       ( 0 when leaving_quantity is 0 )
       leaving_quantity = lot_quantity

   value    = value − leaving_value
   quantity = quantity − leaving_quantity
   ```

   Note that the outgoing branch uses the unit cost **as it stands before the movement**,
   and does not change it. A movement can be both incoming and outgoing (different lines
   crossing the perimeter in opposite directions); in that case the incoming branch runs
   first and the outgoing branch then uses the updated unit cost.

7. Return the running unit cost and running value per product.

### 4.4 Worked example of the average replay with negative stock

| Step | Event | Previous quantity | Quantity after | Unit cost after | Value after |
|---|---|---|---|---|---|
| 1 | Deliver 5 units, product unit cost 10.00 | 0 | −5 | 10.00 | −50.00 |
| 2 | Receive 10 units valued at 110.00 | −5 | 5 | 110.00 ÷ 10 = 11.00 | 11.00 × 5 = 55.00 |

At step 2 the previous quantity was negative, so the unit cost is taken **from the
entering movement alone** (11.00) and the value is rebuilt as *unit cost × quantity*
rather than accumulated. The net effect is that the 5 units delivered while stock was
negative are retrospectively considered to have cost 11.00 each: the value that was
−50.00 becomes, after the receipt of 110.00, 55.00 rather than 60.00, which is exactly
5 × 11.00. The 5.00 difference is the correction of the earlier delivery.

### 4.5 First in first out batch

```formula
for each product:
    quantity     = quantity_on_hand( product )        , or the lot quantity when a lot is given
    total_value  = fifo_value( product, quantity, lot, as_of_instant, location )
    unit_cost    = total_value ÷ quantity             , or 0 when quantity is 0
```

---

## 5. The first in first out algorithm

### 5.1 Building the stack

**Purpose.** Produce the ordered list of completed **incoming** movements whose
quantities, taken from the most recent backwards, add up to at least the quantity on
hand — then reverse it, so that the oldest is first. Together with it, produce the
quantity of the **bottom** movement that is still considered on hand.

**Input.** One product; optionally a lot, an as-of instant and a location.

1. If a location was given, scope the product's quantity computation to that location.
   Remember whether that location is **outside** the valued perimeter.
2. Determine the **stack size**:
   - the lot's quantity, when a lot was given;
   - otherwise the product's quantity on hand, scoped to the valued perimeter and to the
     as-of instant.
3. If the context carries a "first in first out quantity already processed" amount,
   subtract it from the stack size. This is what lets several outgoing movements of the
   same product, validated in one batch, consume different parts of the stack even though
   the quantity on hand has not been updated yet.
4. If the stack size is not greater than zero (compared at the unit-of-measure
   precision), return an empty stack and a bottom quantity of zero.
5. Build the movement filter: movements of this product, of any company in the current
   scope. Add "has a line carrying the lot" when a lot was given. Add "date not after the
   as-of instant" when one was given. Add "destination location is that location" when a
   location was given. Then: if the location is outside the valued perimeter, require the
   **outgoing** flag; otherwise require the **incoming** flag.
6. Fetch movements matching the filter ordered by **date descending, identifier
   descending**, in pages of one hundred.
7. Walk them newest first. For each movement: take its valued quantity (restricted to the
   lot when a lot was given), append the movement to the stack, set the **bottom
   quantity** to the smaller of that valued quantity and the stack size as it stands, then
   subtract the valued quantity from the stack size. Stop when the stack size is no longer
   greater than zero, or when the movements run out. Fetch the next page when the current
   page is exhausted and the stack size is still positive.
8. Reverse the stack, so that the oldest movement is first, and return it together with
   the bottom quantity.

**What the bottom quantity means.** The stack has been built by going backwards until the
quantity on hand was covered. The last movement appended (which becomes the *first* after
the reversal) is usually only **partly** still on hand: only the bottom quantity of it
remains. Everything after it in the reversed stack is entirely still on hand.

### 5.2 Valuing a quantity under first in first out

**Input.** One product, a quantity to value; optionally a lot, an as-of instant and a
location.

1. **Non-positive quantity.** If the quantity is not greater than zero:
   - let *reference cost* be the lot's cost when a lot was given, else the product's unit
     cost;
   - when an as-of instant was given, look up the last incoming movement not after that
     instant; if one exists, use **its** unit price instead of the reference cost;
   - return *quantity × the chosen cost*. (For a zero or negative quantity, this yields
     zero or a negative value.)
2. Build the stack ([section 5.1](#51-building-the-stack)).
3. Set the accumulated cost to zero. While the quantity still to value is greater than
   zero and the stack is not empty:
   1. Pop the oldest movement from the stack; remember it as the last movement used.
   2. Determine the quantity and value this movement offers:
      - **for the bottom movement only** (the first iteration, when a bottom quantity was
        returned): the offered quantity is the bottom quantity and the offered value is
        *movement value × bottom quantity ÷ movement valued quantity*; then the bottom
        quantity is cleared so that this branch is not taken again;
      - **otherwise**: the offered quantity is the movement's valued quantity, restricted
        to the lot when a lot was given, and the offered value is the movement's value;
        when a lot was given, the offered value is further scaled by *lot quantity ÷
        movement valued quantity* (zero when the valued quantity is zero).
   3. If the offered quantity exceeds the quantity still to value, take only part of it:
      the offered value is scaled by *quantity still to value ÷ offered quantity*, and the
      offered quantity becomes the quantity still to value.
   4. Add the offered value to the accumulated cost and subtract the offered quantity
      from the quantity still to value.
4. **Extrapolation.** If a quantity still remains after the stack is exhausted — that is,
   more was asked for than the company ever received — add:

   ```formula
   extrapolated = remaining_quantity × ( last_move_value ÷ last_move_quantity )
                       when a last movement was used and it has a non-zero quantity
   extrapolated = remaining_quantity × product_unit_cost
                       otherwise
   ```

5. Return the accumulated cost.

### 5.3 Remaining quantity per movement

**Purpose.** Distribute the quantity on hand back onto the incoming movements that are
considered to still hold it, so that a reader can see which receipt is still in stock.

**Input.** A set of products, scoped to a company.

For each product:

1. Determine the lots to walk: when the product is valuated by lot, the lots of its
   positive stock quantities inside the valued perimeter belonging to this company;
   otherwise a single pass with no lot.
2. For each lot (or the single no-lot pass): build the stack. If it is empty, skip.
   Assign the **bottom quantity** to the first movement of the stack, and the **full
   valued quantity** to every subsequent movement of the stack, accumulating per movement
   across lots.
3. The resulting map, product → (movement → quantity), is the remaining quantity map.

A movement not present in the map has a remaining quantity of zero — in particular every
outgoing movement, every internal movement and every incoming movement entirely consumed.

### 5.4 Worked example of first in first out

| Step | Event | Stack after (oldest first) | Value of the movement |
|---|---|---|---|
| 1 | Receive 68 at 15.00 | 68 × 15.00 | 1 020.00 |
| 2 | Receive 140 at 15.50 | 68 × 15.00, 140 × 15.50 | 2 170.00 |
| 3 | Deliver 94 | 114 × 15.50 | 68 × 15.00 + 26 × 15.50 = 1 020.00 + 403.00 = **1 423.00** |
| 4 | Receive 40 at 16.00 | 114 × 15.50, 40 × 16.00 | 640.00 |
| 5 | Receive 78 at 16.50 | 114 × 15.50, 40 × 16.00, 78 × 16.50 | 1 287.00 |
| 6 | Deliver 116 | 38 × 16.00, 78 × 16.50 | 114 × 15.50 + 2 × 16.00 = 1 767.00 + 32.00 = **1 799.00** |
| 7 | Deliver 62 | 54 × 16.50 | 38 × 16.00 + 24 × 16.50 = 608.00 + 396.00 = **1 004.00** |
| 8 | Move 10 to a transit location of the company | unchanged | **0.00** — both ends are inside the valued perimeter |
| 9 | Deliver 10 | 44 × 16.50 | 10 × 16.50 = **165.00** |

### 5.5 Worked example of first in first out with a quantity increase after the fact

| Step | Event | Effect |
|---|---|---|
| 1 | Receive 10 at 10.00 | movement A, value 100.00, remaining 10 |
| 2 | Receive 10 at 8.00 | movement B, value 80.00, remaining 10 |
| 3 | Deliver 3 | value 3 × 10.00 = 30.00; A's remaining becomes 7 |
| 4 | Edit movement A's quantity from 10 to 12 and set its value to 12 × 10.00 = 120.00 | A's value is 120.00. The quantity on hand becomes 19. Rebuilding the stack from the newest backwards: B offers 10, leaving 9 to cover; A offers 12, of which only 9 is needed. A's remaining quantity is therefore **9**, not 7 — the two added units enter at the *top* of the queue because the stack is rebuilt from the current quantity on hand, not from a consumption ledger. |
| 5 | Deliver 9 | the stack is A (bottom quantity 9) then B; the delivery takes 9 units from A at 120.00 ÷ 12 = 10.00 each → **90.00** |
| 6 | Deliver 20 | the quantity on hand is 10 (all in B); the stack offers 10 units at 80.00; the remaining 10 units are extrapolated at the last movement used, B, whose unit price is 80.00 ÷ 10 = 8.00 → 80.00 + 80.00 = **160.00**. The quantity on hand becomes −10. |

---

## 6. Negative stock and its correction

### 6.1 What happens when goods leave before they arrive

Nothing special is stored. An outgoing movement made against a negative or zero quantity
on hand is valued exactly as described above:

- under **standard price**, at the product's unit cost;
- under **average cost**, at the product's current unit cost (which is whatever the last
  replay left behind);
- under **first in first out**, by extrapolation from the last incoming movement used, or
  from the product's unit cost when there is none.

The quantity on hand becomes negative and so does the product's total value. The ledger,
under periodic valuation, has not moved, so the closing computation reports the whole
negative amount as a variation.

### 6.2 How the correction happens

There is no separate correction pass, no queue and no scheduled "vacuum". The correction
is a **consequence of the replay**:

- Under **average cost**, the branch of the replay that fires when the previous quantity
  was zero or negative rebuilds the value as *unit cost of the arriving goods × resulting
  quantity* instead of accumulating. That single assignment absorbs the difference
  between what the earlier delivery was valued at and what the goods actually cost.
- Under **first in first out**, the value of the earlier outgoing movement is not
  rewritten, but the product's total value — which is always recomputed from the current
  stack — reflects the real cost of the goods now on hand. The difference appears in the
  closing entry as a variation.

### 6.3 Worked example under first in first out

| Step | Event | Movement value | Quantity on hand | Total value | Ledger balance after closing |
|---|---|---|---|---|---|
| 0 | The product's unit cost is set to 8.00 | — | 0 | 0.00 | 0.00 |
| 1 | Deliver 50 units | 50 × 8.00 = **400.00** | −50 | −400.00 | −400.00 (valuation credited 400.00, variation debited 400.00) |
| 2 | Receive 40 units at 15.00 | **600.00** | −10 | −150.00 | The closing now moves 250.00 the other way: valuation debited 250.00, variation credited 250.00, bringing the valuation balance to −150.00 |
| 3 | Receive 20 units at 25.00 | **500.00** | 10 | 250.00 | The closing moves a further 400.00: valuation debited 400.00, variation credited 400.00, bringing the valuation balance to **250.00** |

At the end the product holds 10 units worth 250.00 — the 10 units left from the second
receipt at 25.00 each — and the ledger agrees. The expense account was never touched.

### 6.4 Worked example under average cost

| Step | Event | Quantity | Unit cost | Total value |
|---|---|---|---|---|
| 1 | Receive 10 at 10.00 | 10 | 10.00 | 100.00 |
| 2 | Receive 10 at 15.00 | 20 | (100 + 150) ÷ 20 = 12.50 | 250.00 |
| 3 | Deliver 15 | 5 | 12.50 | 62.50 |
| 4 | Deliver 10 | −5 | 12.50 | −62.50 |
| 5 | Edit the movement of step 2 from 10 to 20 units, value 300.00 | 5 | 13.333333… | 66.67 |

At step 5 the replay runs again over all four movements: 10 in at 100.00 → cost 10.00,
quantity 10, value 100.00; 20 in at 300.00 → previous quantity 10 > 0, so value 400.00,
quantity 30, cost 13.333333…; 15 out at 13.333333… → value 400.00 − 200.00 = 200.00,
quantity 15; 10 out at 13.333333… → value 200.00 − 133.333333… = 66.666666…, quantity 5.
The total value reported is 66.67 after currency rounding, and the unit cost is
13.3333333…

---

## 7. Landed costs

### 7.1 Collecting the valuation lines

**Input.** One landed cost document.

1. Determine the **targeted movements**:
   - the movements of the selected transfers;
   - plus, when manufacturing landed costs are installed, the finished-goods movements of
     the selected manufacturing orders, **minus** the by-product movements whose cost
     share is zero;
   - and then, when subcontracting landed costs are installed, every targeted movement
     that is a subcontracting movement is replaced by its originating movements.
2. For each targeted movement, skip it when **any** of the following holds: its product's
   costing method is neither `fifo` nor `average`; its state is cancelled; its quantity is
   zero.
3. For each surviving movement, build a valuation line with:

   ```formula
   quantity    = convert( move_quantity, move_unit → product_reference_unit )
   former_cost = the value produced by the priority chain for that movement
   weight      = product_weight × quantity
   volume      = product_volume × quantity
   ```

4. If no line survives, refuse with the error **"You cannot apply landed costs on the
   chosen _the target label_(s). Landed costs can only be applied for products with first in first out
   or average costing method."**, where the target label is "Transfers" or "Manufacturing
   Orders".

### 7.2 The split computation

**Input.** A set of landed cost documents.

1. Delete every existing valuation adjustment line of those documents.
2. For each document that has at least one targeted movement, with the document's company
   in scope:
   1. Let *rounding* be the rounding step of the company currency.
   2. Collect the valuation lines ([section 7.1](#71-collecting-the-valuation-lines)) and,
      for **each** of them, create one adjustment line **per cost line** of the document.
      While doing so accumulate:

      ```formula
      total_quantity = Σ line_quantity                     over the valuation lines
      total_weight   = Σ line_weight                       over the valuation lines
      total_volume   = Σ line_volume                       over the valuation lines
      total_cost     = Σ round_to_currency( line_former_cost )
      total_lines    = the number of valuation lines
      ```

      Note that the totals are summed **once per valuation line**, not once per created
      adjustment line, so adding a second cost line does not double the totals.
   3. For each cost line of the document:
      1. Set the running *split total* to zero.
      2. For each adjustment line of the document belonging to **this** cost line,
         compute the raw share:

         | Split method | Condition | Raw share |
         |---|---|---|
         | `by_quantity` | total quantity non-zero | `cost_amount ÷ total_quantity × line_quantity` |
         | `by_weight` | total weight non-zero | `cost_amount ÷ total_weight × line_weight` |
         | `by_volume` | total volume non-zero | `cost_amount ÷ total_volume × line_volume` |
         | `equal` | always | `cost_amount ÷ total_lines` |
         | `by_current_cost_price` | total cost non-zero | `cost_amount ÷ total_cost × line_former_cost` |
         | any of the above | its condition fails | `cost_amount ÷ total_lines` |

      3. Round the raw share to the currency, **half away from zero**, and add the rounded
         share to the split total.
      4. Accumulate the rounded share onto that adjustment line (a line may receive
         contributions from several cost lines when it was created once per cost line —
         in practice each adjustment line belongs to exactly one cost line, so it receives
         exactly one).
      5. After the cost line is fully split, compute the rounding residue:

         ```formula
         residue = round_to_currency( cost_amount − split_total )
         ```

         If the residue is not zero for the currency, add it to the adjustment line with
         the **highest identifier** among all adjustment lines accumulated so far across
         the whole computation.
3. Write the accumulated amounts onto the adjustment lines.

**Rounding note.** The residue is attached to the line with the greatest identifier, not
to the largest line and not to the last line of the same cost line. With a single cost
line this is the last adjustment line created, which is the natural place; with several
cost lines the residues of the earlier cost lines all land on the same (currently
highest) line.

### 7.3 Worked example: one hundred split by quantity across two products

A landed cost of 100.00 is applied to a receipt containing two movements: 30 units of
product A and 20 units of product B. Both products use average cost. The split method is
by quantity.

```formula
total_quantity = 30 + 20 = 50
per_unit       = 100.00 ÷ 50 = 2.00
share(A)       = round_to_currency( 30 × 2.00 ) = 60.00
share(B)       = round_to_currency( 20 × 2.00 ) = 40.00
split_total    = 100.00
residue        = round_to_currency( 100.00 − 100.00 ) = 0.00   → nothing to correct
```

Product A's movement gains 60.00 and product B's gains 40.00.

### 7.4 Worked example with a rounding residue

A landed cost of 100.00 is split equally over three movements.

```formula
raw share  = 100.00 ÷ 3 = 33.333333…
rounded    = 33.33 for each of the three
split_total = 99.99
residue     = round_to_currency( 100.00 − 99.99 ) = 0.01
```

The 0.01 is added to the adjustment line with the highest identifier, which becomes
33.34. The three shares are 33.33, 33.33 and 33.34 and they sum to exactly 100.00.

### 7.5 The sum check

Before a document may be validated, two conditions must hold for each document:

```formula
| Σ additional_landed_cost over all adjustment lines − amount_total | is zero for the currency
```

and, for **every** cost line that has at least one adjustment line:

```formula
| cost_line_amount − Σ additional_landed_cost over that cost line's adjustment lines |
    is zero for the currency
```

If either fails, the validation is refused with the error **"Cost and adjustments lines
do not match. You should maybe recompute the landed costs."**

### 7.6 How a landed cost reaches the value of a movement

The *extra* source of the value priority chain
([section 1.2](#12-algorithm), step 8) works as follows:

1. Collect the adjustment lines attached to this movement whose document is in the
   **posted** state; when an as-of instant is given, restrict to documents whose date is
   not after it.
2. If there are none, contribute nothing.
3. Otherwise add the sum of their additional landed costs to the value, and build the
   justification:
   - per adjustment line, **"+ _the formatted amount_ from _the vendor bill display
     name_ (Landed Cost: _the document display name_)"** when the document has a vendor
     bill, else **"+ _the formatted amount_ (Landed Cost: _the document display name_)"**;
   - the lines are joined by newlines under the heading **"Additional landed costs:"**.

Because this source is suppressed whenever a manual correction claimed the quantity, a
human who adjusts a movement's value is stating the **final** value, landed costs
included.

### 7.7 The landed cost entry amount

For each adjustment line whose movement's product has the `real_time` valuation mode:

```formula
posted_amount = additional_landed_cost × ( remaining_quantity ÷ line_quantity )
```

where the remaining quantity is the movement's remaining quantity at validation time.
When the remaining quantity is zero, **no journal items are produced for that line** —
the goods have already left, so there is nothing left in stock to revalue. When the
remaining quantity is negative (goods delivered that were never in stock), the amount is
negative and the two journal items swap sides.

**Worked example.** A landed cost of 60.00 was allocated to a receipt of 30 units. By the
time it is validated, only 18 of those units are still on hand.

```formula
posted_amount = 60.00 × ( 18 ÷ 30 ) = 36.00
```

36.00 is debited to the inventory valuation account and credited to the cost line's
counterpart account. The full 60.00 nevertheless enters the movement's value through the
extra source, so the *remaining value* of the movement rises by the proportional share.

---

## 8. Production value

### 8.1 The cost of a finished good

When a manufacturing order computes the price of its finished goods:

1. Let *finished movements* be the finished-goods movements of the order for the order's
   own product whose state is neither completed nor cancelled and whose quantity is
   positive.
2. If there are none, stop.
3. Sum the work-centre cost over the order's work orders.
4. Let *quantity* be the sum, over the finished movements, of the movement quantity
   converted into the product reference unit.
5. Compute:

   ```formula
   extra_cost  = order_extra_unit_cost × quantity
   total_cost  = Σ value of the consumed component movements
                 + work_centre_cost
                 + extra_cost
   ```

6. For each by-product movement of the order that is neither completed nor cancelled and
   has a positive quantity, accumulate its cost share and set its unit price:

   ```formula
   by_product_quantity = convert( by_product_move_quantity → product_reference_unit )

   by_product_unit_price = total_cost × cost_share ÷ 100 ÷ by_product_quantity
                               when the by-product's costing method is fifo or average
                               and the cost share is non-zero at two decimals
                               ( 0 when the quantity is zero )

   by_product_unit_price = by_product_unit_cost
                               when the costing method is standard price
   ```

   A by-product whose cost share rounds to zero at two decimals and whose method is fifo
   or average keeps its existing unit price and is skipped, but its (zero) cost share is
   still accumulated.

7. Set the unit price of the finished movements:

   ```formula
   finished_unit_price = finished_product_unit_cost
                             when the finished product's costing method is standard price

   finished_unit_price = total_cost
                         × round_half_up( 1 − total_by_product_cost_share ÷ 100, 0.0001 )
                         ÷ quantity
                             otherwise
   ```

### 8.2 Worked example

A manufacturing order produces 10 units of a finished good using average cost. It
consumes three component movements valued at 78.00, 180.00 and 20.00. Its work orders
cost 200.00. The extra unit cost is zero. There are no by-products.

```formula
total_cost          = 78.00 + 180.00 + 20.00 + 200.00 + 0.00 = 478.00
by_product_share    = 0
finished_unit_price = 478.00 × round_half_up( 1 − 0 ÷ 100, 0.0001 ) ÷ 10
                    = 478.00 × 1.0000 ÷ 10 = 47.80
```

The finished-goods movement then values at 10 × 47.80 = 478.00 through the production
source of the priority chain.

### 8.3 Worked example with a by-product

The same order also produces a by-product carrying a cost share of 25 percent, in a
quantity of 5 units, using average cost.

```formula
by_product_unit_price = 478.00 × 25 ÷ 100 ÷ 5 = 119.50 ÷ 5 = 23.90
finished_unit_price   = 478.00 × round_half_up( 1 − 25 ÷ 100, 0.0001 ) ÷ 10
                      = 478.00 × 0.7500 ÷ 10 = 35.85
```

The finished goods take 358.50 and the by-product takes 119.50; together 478.00.

### 8.4 Kit price unit

For a product built from a kit bill of materials, the unit price of a set of movements is
not a single movement's price but a weighted recombination:

1. Explode the kit for the valued quantity, obtaining, per component, the quantity
   required.
2. Accumulate, per component, the quantity per kit:

   ```formula
   component_quantity_per_kit = Σ convert( exploded_line_quantity → component_reference_unit )
   ```

3. For each component, take the unit price of the movements of that component — the
   drop-shipment unit price when any of them is a drop shipment, otherwise the ordinary
   unit price:

   ```formula
   total_price_unit = Σ over components of
                      component_unit_price × ( component_quantity_per_kit ÷ kit_quantity )
   ```

4. Return:

   ```formula
   kit_price_unit = total_price_unit ÷ valued_quantity        ( 0 when the valued quantity is zero )
   ```

### 8.5 Cost of a bill of materials

The cost of a product computed from its bill of materials, used by the "compute price
from bill of materials" action:

1. Start at zero. Add the cost of every operation of the bill of materials that is not
   skipped for this product.
2. For every component line that is not skipped for this product:
   - if the line has a child bill of materials that is itself scheduled for
     recomputation: recursively compute that child's cost, convert it from the child
     product's reference unit into the line unit, and multiply by the line quantity;
   - otherwise: convert the component's unit cost from the component reference unit into
     the line unit and multiply by the line quantity.
3. **For the main product:**

   ```formula
   total = total × round_half_up( 1 − Σ by_product_cost_share ÷ 100, 0.0001 )
                   when at least one by-product carries a cost share
   result = convert_price( total ÷ bill_quantity,
                           bill_unit_of_measure → product_reference_unit )
   ```

4. **For a by-product:** let *by-product quantity* be the sum of the by-product line
   quantities converted into the product reference unit, and *by-product share* the sum
   of their cost shares:

   ```formula
   result = total × by_product_share ÷ 100 ÷ by_product_quantity
            when both the share and the quantity are non-zero, else 0
   ```

---

## 9. Cost of goods sold

### 9.1 Cost of goods sold unit price of a set of movements

**Input.** A set of goods movements, all of the same product; a quantity in the product
reference unit.

```formula
signed_quantity = Σ over the movements of
                  valued_quantity × ( −1 when the movement is incoming, +1 otherwise )

consigned_quantity = Σ over the consigned valued lines of the movements of
                     line_quantity_in_reference_unit
                     × ( −1 when the destination is inside the valued perimeter, +1 otherwise )

total_valued_quantity = signed_quantity + consigned_quantity
```

Then:

```formula
signed_value = Σ over the movements of
               move_value × ( −1 when the movement is incoming, +1 otherwise )

unit_price = signed_value ÷ total_valued_quantity
                 when total_valued_quantity ≠ 0
                 and ( the costing method is fifo or average, or consigned_quantity ≠ 0 )

unit_price = product_unit_cost
                 otherwise
```

If the set contains movements of more than one product, the result is zero.

The sign convention makes deliveries positive and returns negative, so a delivery of 10
followed by a return of 3 yields a net quantity of 7 and a net value equal to the cost of
7 units.

### 9.2 Cost of goods sold value of an invoice line

**Input.** One invoice or credit-note line.

1. **Shortcut for standard price and average cost.** If the product's costing method is
   `standard` or `average`, look for the corresponding cost-of-goods-sold line on the
   *original* document:
   - the reversed document's lines whose display type is `cogs`, whose product and unit
     of measure match, and whose unit price is not negative;
   - plus, when the line belongs to a credit note that is **not** a reversal, the
     cost-of-goods-sold lines of the customer invoices of the same sales order lines,
     matched the same way.

   If any is found, return the **unit price of the first of them**. This makes a credit
   note reverse the cost that was recognised, rather than recompute it at today's cost.
2. If there is no product, or the line quantity is zero, return the line's unit price.
3. Compute the **cost-of-goods-sold quantity** (see
   [section 9.3](#93-cost-of-goods-sold-quantity)).
4. Find the completed goods movements reachable from the line. If there are any, take
   their cost-of-goods-sold unit price ([section 9.1](#91-cost-of-goods-sold-unit-price-of-a-set-of-movements)).
   If there are none:
   - under standard price or average cost, take the product's unit cost;
   - under first in first out, take *first in first out value of the cost-of-goods-sold
     quantity ÷ that quantity*, or zero when the quantity is zero.
5. Let *line quantity in the reference unit* be the line quantity converted from the line
   unit of measure into the product reference unit. Return:

   ```formula
   cogs_unit_price = | unit_price × cogs_quantity − already_posted_cogs_value |
                     ÷ line_quantity_in_reference_unit
   ```

   The absolute value is taken because the convention of the injected lines already
   carries the sign.

### 9.3 Cost-of-goods-sold quantity

```formula
own_quantity = convert( line_quantity → product_reference_unit )
               × ( −1 when the document is a customer credit note, +1 otherwise )
```

When the sales integration is installed, the quantity for which cost has **already** been
recognised on the same sales order lines is added:

```formula
posted_quantity = Σ over the already-posted cost-of-goods-sold lines
                  that hit the product's inventory valuation account
                  and whose origin line belongs to the same sales order lines, of
                  convert( posted_line_quantity → product_reference_unit )
                  × ( −1 when that document is a customer credit note, +1 otherwise )

cogs_quantity = posted_quantity + own_quantity
```

### 9.4 Already-posted cost-of-goods-sold value

```formula
already_posted_cogs_value =
    − Σ over the already-posted cost-of-goods-sold lines
      that hit the product's inventory valuation account
      and whose origin line belongs to the same sales order lines, of
      line_balance
```

The negation converts the credit balances of the inventory-side lines into a positive
amount of cost already recognised.

### 9.5 Worked example: invoicing a partly delivered order

A sales order is for 10 units of a product using first in first out. The company uses
perpetual valuation with the anglo-saxon convention. Six units are delivered; the
delivery is valued at 6 × 10.00 = 60.00.

**First invoice, for the 6 delivered units.**

```formula
cogs_quantity   = 0 (nothing posted yet) + 6 = 6
unit_price      = 60.00 ÷ 6 = 10.00        ( from the completed delivery )
already_posted  = 0.00
cogs_unit_price = | 10.00 × 6 − 0.00 | ÷ 6 = 10.00
amount          = 6 × 10.00 = 60.00
```

The invoice carries, besides revenue and receivable, a debit of 60.00 to the expense
account and a credit of 60.00 to the inventory valuation account.

**The remaining 4 units are delivered** at 12.00 each; the second delivery is valued at
48.00. The two deliveries together are worth 108.00 for 10 units.

**Second invoice, for the 4 remaining units.**

```formula
posted_quantity = 6
cogs_quantity   = 6 + 4 = 10
unit_price      = ( 60.00 + 48.00 ) ÷ ( 6 + 4 ) = 10.80      ( over both completed deliveries )
already_posted  = 60.00
cogs_unit_price = | 10.80 × 10 − 60.00 | ÷ 4 = 48.00 ÷ 4 = 12.00
amount          = 4 × 12.00 = 48.00
```

The second invoice recognises exactly the 48.00 that was not yet recognised, even though
the average unit price over both deliveries is 10.80. Total cost recognised: 108.00,
exactly the value that left stock.

### 9.6 Worked example: invoicing before delivery

The same order is invoiced in full for 10 units before anything is delivered.

```formula
completed movements: none
unit_price      = fifo_value( 10 ) ÷ 10
already_posted  = 0.00
cogs_quantity   = 10
```

If the company holds 10 units at 10.00 each, the first in first out value of 10 units is
100.00 and the cost-of-goods-sold unit price is 10.00. The cost is recognised at the
invoice even though nothing has shipped; the inventory valuation account is credited and
the expense account debited by 100.00.

---

## 10. Price difference at the vendor bill under standard price

### 10.1 The comparison price

```formula
valuation_unit_price = convert_price( product_unit_cost,
                                      product_reference_unit → line_unit_of_measure )

valuation_unit_price = − valuation_unit_price       when the document is a vendor credit note

valuation_unit_price = convert( valuation_unit_price,
                                company_currency → document_currency,
                                at the line date, without rounding )
```

The comparison is therefore made **in the document currency**, so that a foreign-currency
bill is compared against the company-currency cost translated at the line's date.

### 10.2 The difference

```formula
price_unit_difference = gross_unit_price − valuation_unit_price
relevant_quantity     = line_quantity
price_subtotal_difference = relevant_quantity × price_unit_difference
```

where the gross unit price is the line's unit price net of discount, as defined in
[entities.md](entities.md#13-journal-item-accountmoveline-table-accountmoveline).

### 10.3 When a difference is posted

Both of the following must hold:

1. The subtotal difference is **not zero for the document currency**.
2. The line's raw stored unit price and its computed unit price agree at the Product
   Price precision. (When a discount has been applied, the unit price can no longer be
   rounded for comparison, and the difference is not posted.)

Additionally, the whole mechanism only runs when: the document is a vendor bill, vendor
credit note or vendor receipt; the company uses anglo-saxon accounting; the line is
eligible for stock accounting; and the product's costing method is `standard`. A price
difference account must be selectable (the category's price difference account, mapped
through the document's fiscal position); when it is empty the line is skipped.

### 10.4 The amounts

Two journal items are produced, each carrying the line's analytic distribution, no taxes,
display type `cogs`, and the relevant quantity:

```formula
balance( price difference account ) = convert( relevant_quantity × price_unit_difference,
                                               document_currency → company_currency,
                                               at today's date )

balance( the line's own account )   = convert( relevant_quantity × ( − price_unit_difference ),
                                               document_currency → company_currency,
                                               at today's date )
```

The second line reverses the first on the account the bill line itself used — which,
under perpetual valuation, is the inventory valuation account. The net effect is to move
the difference out of the inventory asset and into the price difference account.

### 10.5 Worked example: a bill priced differently from its receipt

A product uses standard price with a unit cost of 9.00, perpetual valuation and
anglo-saxon accounting. The category's price difference account is set. Ten units are
received and then billed at 10.00 each, in the company currency, with no discount and no
taxes.

The bill's own lines, before the mechanism runs:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 100.00 | |
| Accounts payable | | 100.00 |

The mechanism computes:

```formula
valuation_unit_price      = 9.00
gross_unit_price          = 10.00
price_unit_difference     = 10.00 − 9.00 = 1.00
relevant_quantity         = 10
price_subtotal_difference = 10.00   → not zero, and the unit prices agree at 2 decimals
```

and adds:

| Account | Debit | Credit |
|---|---|---|
| Price difference | 10.00 | |
| Inventory valuation | | 10.00 |

so that the inventory asset is debited by 90.00 net — ten units at the standard cost of
9.00 — and the 10.00 overpayment is expensed to the price difference account.

---

## 11. The inventory valuation closing

### 11.1 The two sides being compared

```formula
physical_value( account ) = Σ over the products whose inventory valuation account is that account of
                            total_value( product, as of the report date )

ledger_value( account )   = Σ over the posted journal items on that account,
                            of the company, dated not after the report date, of
                            line_balance
```

The physical value uses the valuation engine; the ledger value uses the general ledger.
Their difference is what the closing entry corrects.

### 11.2 Part one — location reclassification

**Purpose.** Goods that left the valued perimeter through a location that carries its own
valuation account (an inventory-loss location, a production location, a scrap location)
must be moved off the inventory asset and onto that location's account.

**Input.** A report date (optional) and an optional extra restriction on the locations.

1. Select the locations of the company that carry a valuation account, restricted by the
   caller's location condition when one is given.
2. Build the base movement filter: the product is storable **and** the product's valuation
   mode is `periodic`. When a previous closing exists, add "date **after** the last
   closing instant". When a report date is given, add "date not after it".
3. **Goods that went into those locations** — a movement whose **outgoing** flag is set
   and whose destination is one of those locations — grouped by destination location and
   product category, summing the movement value.
4. **Goods that came back out of those locations** — a movement whose **incoming** flag
   is set and whose source is one of those locations — grouped by source location and
   product category, summing the movement value.
5. Build a balance per (location account, inventory valuation account) pair, where the
   inventory valuation account is the category's account for that company, falling back
   to the company's account:

   ```formula
   balance( location_account, valuation_account ) =
       Σ value of the movements that went out into that location
       − Σ value of the movements that came back from that location
   ```

6. For each pair whose balance is not exactly zero, produce a balanced pair of journal
   items debiting the **location account** and crediting the **inventory valuation
   account** by that balance, with the label **"Closing: Location Reclassification -
   [_the location account display name_]"**. A negative balance swaps the two sides (see
   [section 11.5](#115-the-balanced-pair-rule)).

### 11.3 Part two — the global stock variation

**Purpose.** Bring each inventory valuation account's ledger balance to the physical
value of the goods it represents.

**Input.** The product-to-account map, an optional report date, and the journal items
already proposed by part one.

1. Compute the **extra balance** per account from the already-proposed items:

   ```formula
   extra_balance( account ) = Σ ( debit − credit ) over the proposed items on that account
   ```

2. Take the physical value per account (supplied by the caller when the report already
   computed it, otherwise computed now) and the ledger value per account.
3. For each account appearing in either:
   1. Let *variation account* be that account's variation account; when it is empty, the
      company's default expense account; when that is also empty, **skip this account
      entirely**.
   2. Compute:

      ```formula
      balance = physical_value( account ) − ledger_value( account ) − extra_balance( account )
      ```

   3. If the balance is zero for the company currency, skip.
   4. Produce a balanced pair of journal items debiting the **inventory valuation
      account** and crediting the **variation account** by that balance, with the label
      **"Closing: Stock Variation Global for company [_the company display name_]"**.

### 11.4 Part three — the continental perpetual period variation

**Purpose.** Under perpetual valuation without the anglo-saxon convention, the inventory
variation of the period is never posted by the movements themselves; this part posts it.
It runs only for accounts that carry **both** a variation account and a closing expense
account.

1. Compute the extra balance per account from the items already proposed by parts one and
   two.
2. Let *fiscal year start* be the first day of the fiscal year containing today.
3. Compute the ledger value per account **today** and **as of the fiscal year start**.
4. For each account appearing in either:
   1. Let *variation account* and *expense account* be that account's variation and
      closing expense accounts. If either is empty, skip this account.
   2. Compute:

      ```formula
      balance_today        = ledger_value_today( account ) − extra_balance( account )
      balance_year_start   = ledger_value_at_fiscal_year_start( account )
      balance_over_period  = balance_today − balance_year_start

      existing_variation   = Σ balance of the posted journal items on the variation account,
                             of the company, dated not after the report date

      balance_over_period  = balance_over_period + existing_variation
      ```

   3. If the result is zero for the company currency, skip.
   4. Produce a balanced pair of journal items debiting the **closing expense account**
      and crediting the **variation account** by that amount, with the label
      **"Closing: Stock Variation Over Period"**.

### 11.5 The balanced pair rule

Every closing item is produced by the same helper. Given a nominal debit account, a
nominal credit account and a balance:

```formula
when balance < 0:  swap the two accounts and replace the balance by its absolute value

item 1: account = the credit account,  debit = 0,        credit = |balance|
item 2: account = the debit account,   debit = |balance|, credit = 0
```

Both items carry the same label, and a product reference when the caller supplied one
(the closing never supplies one).

### 11.6 The last closing instant

```formula
1. Read the stored list of closing entry identifiers for the company.
2. Walk it from the end backwards until an entry is found that still exists and is posted.
3. If none is found, there is no last closing instant.
4. Otherwise, find the latest tracked change of that entry's state field.
   If such a change exists and the date part of its creation instant equals the entry's
   date, the last closing instant is that creation instant (date and time).
   Otherwise it is the entry's date widened to an instant.
```

Using the *creation instant* rather than the date lets two closings be posted on the same
day without the second one re-including the movements the first one already covered.

### 11.7 The closing operation

**Preconditions and failures**, in the order they are checked:

1. Exactly one company.
2. If a report date is given and a last closing date exists and the report date is before
   it, refuse with **"It exists closing entries after the selected date. Cancel them
   before generate an entry prior to them"**.
3. Compute the three parts. If they produce **no** journal items at all:
   - when the operation was started by the scheduled job, return silently (other
     companies may still have work);
   - otherwise refuse with **"Everything is correctly closed"**.
4. If the company has no inventory journal, refuse with **"Please set the Journal for
   Inventory Valuation in the settings."**
5. If the company has no inventory valuation account, refuse with **"Please set the
   Valuation Account for Inventory Valuation in the settings."**

**Effects.**

1. Create a journal entry in the company's inventory journal, dated at the report date or
   today, referenced **"Stock Closing"**, in the company, carrying every proposed item.
2. Append its identifier to the company's stored closing list, dropping the oldest when
   the list would exceed ten.
3. Post it when the caller asked for automatic posting.
4. Return a window action opening that entry, titled **"Journal Items"**.

### 11.8 Worked example of a closing

A company uses periodic valuation. One inventory valuation account is in play, with a
variation account attached. Nothing has been closed before.

| Fact | Amount |
|---|---|
| Physical value of the goods on hand | 250.00 |
| Posted ledger balance of the inventory valuation account | 0.00 |
| Location reclassification items proposed | none |

```formula
extra_balance = 0.00
balance = 250.00 − 0.00 − 0.00 = 250.00
```

The entry is:

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 250.00 | |
| Inventory variation | | 250.00 |

A month later the physical value is 310.00 and the ledger balance (after the first
closing was posted) is 250.00.

```formula
balance = 310.00 − 250.00 − 0.00 = 60.00
```

| Account | Debit | Credit |
|---|---|---|
| Inventory valuation | 60.00 | |
| Inventory variation | | 60.00 |

And if the physical value had fallen to 200.00 instead:

```formula
balance = 200.00 − 250.00 − 0.00 = −50.00   → the sides swap
```

| Account | Debit | Credit |
|---|---|---|
| Inventory variation | 50.00 | |
| Inventory valuation | | 50.00 |

---

## 12. Analytic distribution of a movement

### 12.1 The amount analytic lines carry

```formula
when the movement is cancelled or draft:            no analytic lines
when the movement is not completed and not picked:  no analytic lines

when the movement is not completed but picked:
    unit_amount = convert( move_quantity → product_reference_unit )
    amount      = unit_amount × product_unit_cost          ← an estimate

when the movement is completed:
    amount      = move_value
    unit_amount = move_valued_quantity

when the movement counts as outgoing:
    amount = − amount
```

If the movement already has analytic lines and both the amount and the unit amount are
zero, the existing lines are deleted and nothing is created.

The estimate used for a picked but not-yet-completed movement deliberately uses the
product's unit cost even under first in first out, because re-estimating every analytic
line at each outgoing movement would be prohibitively expensive and the figure is
provisional.

### 12.2 Distributing an amount across analytic accounts

Given a distribution (a map from one or more analytic accounts to a percentage), an
amount, a unit amount, the existing lines and the object being mirrored:

1. If the distribution is empty, delete every existing line and return nothing.
2. Resolve each key of the distribution into the set of analytic accounts it names.
3. Compute, per analytic plan, the total percentage assigned to it across the whole
   distribution.
4. For every existing line: determine the set of accounts it currently carries. If that
   set is in the distribution, update the line:

   ```formula
   new_amount = the plan's share of the amount (see below)
   new_unit_amount = unit_amount                          , or, in additive mode,
                     unit_amount + existing_unit_amount
   ```

   and, in additive mode, add the existing amount to the new amount. If the new amount is
   zero for the account's currency (or the object's company currency when the account has
   none), delete the line instead. Remove that set from the distribution so it is not
   applied twice. If the set is **not** in the distribution, delete the line.
5. For each set remaining in the distribution, create a line with the computed amount,
   unless that amount is zero for the currency.

**The plan's share** is computed so that the amounts distributed inside one plan always
add up to exactly the whole amount:

```formula
allocated_percentage = already_distributed_percentage + this_percentage

when allocated_percentage equals the plan's total percentage
     ( compared at the analytic-percentage precision ):
    calculated_amount = amount × total_percentage ÷ 100 − already_distributed_amount

otherwise:
    calculated_amount = amount × this_percentage ÷ 100

already_distributed_amount += round( calculated_amount, analytic-percentage precision )
already_distributed_percentage = allocated_percentage
```

The first branch is the closing correction: when a line completes a plan's total, it
receives the exact outstanding amount rather than its own proportional share, so the
compounding rounding error is absorbed.

**Worked example.** An amount of 100.00 is distributed over three analytic accounts of
the same plan at 33.33, 33.33 and 33.34 percent, and the plan's total percentage is
100.00.

```formula
line 1: allocated = 33.33 ≠ 100 → amount = 100 × 33.33 ÷ 100 = 33.33 ; distributed = 33.33
line 2: allocated = 66.66 ≠ 100 → amount = 100 × 33.33 ÷ 100 = 33.33 ; distributed = 66.66
line 3: allocated = 100.00 = 100 → amount = 100 × 100 ÷ 100 − 66.66 = 33.34
```

---

## 13. The value of a stock quantity record

```formula
reference_quantity = lot_quantity_in_company        when the product is valuated by lot
reference_quantity = product_quantity_on_hand_in_company_in_the_valued_perimeter   otherwise

reference_value    = lot_total_value_in_company     when the product is valuated by lot
reference_value    = product_total_value_in_company otherwise

quant_value = 0
                  when the location is outside the valued perimeter,
                  or the owner is somebody other than the company's partner,
                  or the quantity is zero,
                  or the reference quantity is zero

quant_value = quant_quantity × reference_value ÷ reference_quantity
                  otherwise
```

**Worked example.** A product using average cost has 100 units on hand across two
locations worth 1 250.00 in total. A quantity record at one of those locations holds 40
units.

```formula
quant_value = 40 × 1 250.00 ÷ 100 = 500.00
```

**Worked example with consignment.** The same product has an additional 10 units held for
a third-party owner. Those 10 units do not count towards the reference quantity (the
quantity on hand computation excludes them), and their own quantity record is valued at
zero because its owner is not the company's partner.

---

## 14. Costing method change

When the costing method of a category changes, or a product moves to a category with a
different method, the unit cost of every affected product is recomputed with the **new**
method, following [section 3](#3-maintaining-the-product-unit-cost):

| New method | Effect on the unit cost |
|---|---|
| `standard` | Nothing. The cost keeps whatever value it had at the moment of the change and stops moving. |
| `average` | The average is replayed with the recomputation forced, from the latest manual cost change if every affected product has one, otherwise from the beginning of time. The cost becomes the replay result. |
| `fifo` | The cost becomes *total value ÷ quantity on hand* when the quantity on hand is positive; otherwise it becomes the unit price of the last incoming movement when one exists and yields a non-zero price; otherwise it is left unchanged. |

Because the recomputation goes through the "disable automatic revaluation" path, **no
valuation history record is created** by a costing method change. Under periodic
valuation, no journal entry is created either; the change surfaces in the next closing as
a variation. Under perpetual valuation, the change likewise produces no entry of its own:
only goods movements and invoices post.

Every affected product that is valuated by lot also has all of its lots recomputed with
the new method.

---

## 15. Valuation at a past date

Any of the batch routines may be run as of an instant. Three things change:

1. **The quantity on hand** is the quantity as of that instant, which the inventory
   operations domain computes by unwinding the movements made since.
2. **The movement filter** gains "date not after the instant" everywhere: in the average
   replay, in the first in first out stack, in the last-incoming-movement lookup and in
   the landed cost lookup.
3. **The unit cost** is taken from the valuation history rather than from the product,
   under standard price; under first in first out, the product-cost source of the priority
   chain uses the movement's own value divided by its valued quantity instead of today's
   unit cost, so that re-evaluating a past movement does not drift with the product's
   current cost.

**Date widening.** When the caller supplies a plain date rather than an instant, it is
widened to the **last instant of that day**, so that "as of the fifth" includes everything
that happened on the fifth.

**Worked example.** A product using standard price is created with a cost of 10.00 on day
one. On day four the cost is changed to 20.00. Receipts of 10 units happen on day one and
on day three, and another on day six.

- Valued as of day two: the cost from the history is 10.00; the quantity on hand is 10;
  the total value is 100.00.
- Valued as of day five: the cost from the history is 20.00; the quantity on hand is 20;
  the total value is 400.00.
- Valued today: the cost is 20.00; the quantity on hand is 30; the total value is 600.00.

---

## 16. Cost used for sales margins

### 16.1 Statement

A sales order line carries a **cost** figure and a **margin** figure. The cost figure is
normally derived from the product's unit cost, but when the line has goods movements and
the product's category uses a costing method other than standard price, it is derived
from the value that actually left stock.

### 16.2 The line's cost

**Input.** One sales order line.

1. If the line has **no** goods movement in a state other than cancelled or draft, fall
   back to the generic rule (step 5 below).
2. Otherwise, if the line has a product whose category carries a costing method other
   than `standard`:

   ```formula
   delivered_quantity = qty_delivered

   delivery_unit_price = the delivery unit price of the line's completed movements
                              when delivered_quantity > 0
   delivery_unit_price = 0    otherwise

   when delivered_quantity ≤ 0:
       cost = product_unit_cost

   otherwise:
       remaining_quantity = max( ordered_quantity − delivered_quantity , 0 )
       cost = ( delivered_quantity × delivery_unit_price
                + remaining_quantity × product_unit_cost )
              ÷ ( delivered_quantity + remaining_quantity )
   ```

   Then convert the cost from the product reference unit into the line's unit of measure,
   and convert it from the product's cost currency into the line's currency.
3. Otherwise, if the line has no ordered quantity but does have a delivered quantity —
   the case of a line created from a delivery under standard price — fall back to the
   generic rule.
4. Otherwise leave the existing cost untouched. This is what lets a human override the
   cost on a line of a standard-price product without the engine overwriting it.
5. **Generic rule.** Convert the product's unit cost from the product reference unit into
   the line's unit of measure, then convert it from the product's cost currency into the
   line's currency.

### 16.3 The delivery unit price of a set of movements

```formula
dropship_moves     = the movements of the set that are drop shipments or returned drop shipments
dropship_quantity  = Σ valued quantity over those movements
dropship_price     = Σ value over those movements ÷ dropship_quantity
                          ( 0 when dropship_quantity is 0 )

regular_moves      = the remaining movements whose outgoing flag is set
regular_quantity   = Σ valued quantity over those movements
regular_price      = Σ value over those movements ÷ Σ valued quantity over them
                          ( 0 when that sum is 0 )

total_quantity     = dropship_quantity + regular_quantity

delivery_unit_price = the plain unit price of the whole set      when total_quantity is 0
delivery_unit_price = ( dropship_quantity × dropship_price
                        + regular_quantity × regular_price )
                      ÷ total_quantity                            otherwise
```

The **plain unit price** of a set of movements is:

```formula
plain_unit_price = Σ move value ÷ Σ move valued quantity
plain_unit_price = 0    when the total valued quantity is 0
plain_unit_price = 0    when the set covers more than one product
```

The **drop-shipment unit price** is the same shape, but the value of each movement is
re-evaluated through the priority chain rather than read from the stored field:

```formula
dropship_unit_price = Σ evaluated value ÷ Σ valued quantity
                      ( 0 when the total valued quantity is 0 )
```

### 16.4 The margin

```formula
when delivered_quantity ≠ 0 and ordered_quantity = 0:
    calculated_subtotal = unit_price × delivered_quantity
    margin              = calculated_subtotal − cost × delivered_quantity
    margin_percentage   = margin ÷ calculated_subtotal
                              ( 0 when the calculated subtotal is 0 )

otherwise:
    margin            = line_subtotal − cost × ordered_quantity
    margin_percentage = margin ÷ line_subtotal
                              ( 0 when the subtotal is 0 )
```

### 16.5 Worked example

A sales order line orders 10 units at 20.00 each, subtotal 200.00. The product uses first
in first out with a unit cost of 11.00. Six units have been delivered, and those
deliveries are worth 60.00 in total.

```formula
delivered_quantity  = 6
delivery_unit_price = 60.00 ÷ 6 = 10.00
remaining_quantity  = max( 10 − 6 , 0 ) = 4
cost                = ( 6 × 10.00 + 4 × 11.00 ) ÷ 10 = ( 60.00 + 44.00 ) ÷ 10 = 10.40
margin              = 200.00 − 10.40 × 10 = 200.00 − 104.00 = 96.00
margin_percentage   = 96.00 ÷ 200.00 = 0.48
```

---

## 17. Value shown on the forecast report

For a product and a set of warehouse locations, when the reader is an inventory manager:

```formula
report_value = Σ value over the stock quantity records of the product
               in the company of the first of those locations
               that sit in one of those locations
```

The result is formatted with the company currency's decimal places and its symbol, placed
before or after the number according to the currency's position setting.
