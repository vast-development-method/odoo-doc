# Calculations

Replenishment arithmetic answers three questions: when to order, how much to order, and from whom. The formulas and algorithms below answer them: the walk that builds a rule chain, the cumulative lead time and its narrative breakdown, the forecast horizon, the quantity to order and its rounding to a multiple, the deadline date, the purchase order and purchase order line dates, vendor selection, the grouping windows, the move scheduled date and deadline propagation, the forecast reconciliation that feeds the forecast report, the replenishment demand graph, the replenishment report shortage detection, the on-time delivery rate, the effective days to arrival, and the purchase suggestion quantities. Every formula states its inputs, its output, its precision, its order of operations and at least one worked example with real numbers.

---

## 0. Conventions

### 0.1 Precision and rounding

| Concept | Rule |
|---|---|
| Stored quantity precision | Every quantity of this domain is stored with the precision of the shared decimal precision setting named "Product Unit", whose default is two decimal places. |
| Comparison | `compare(a, b, unit)` rounds both values to the rounding step of *unit* and then compares. It returns −1 when *a* is lower, 0 when they are equal and 1 when *a* is greater. Exact equality on stored decimals is never used. |
| `is_zero(a, unit)` | True when *a* rounded to the rounding step of *unit* equals zero. |
| Half-up rounding | `round_half_up(x, step)` rounds *x* to the nearest multiple of *step*, and away from zero on an exact tie. Used for every conversion between a request unit and a purchase line unit. |
| Round up | `round_up(x, step)` returns the smallest multiple of *step* that is not lower than *x*. Used for the replenishment multiple. |
| Unit conversion | `convert(quantity, from_unit, to_unit)` is the shared conversion of `../units-of-measure-and-packaging/calculations.md`. Unless a formula says otherwise, the conversion result is rounded to the rounding step of the target unit; `convert_exact` performs the same conversion without rounding. |

**Worked example.** The unit `Units` has a rounding step of 0.01. Then `compare(5.004, 5.001, Units)` rounds both values to 5.00 and returns 0, so the two quantities are treated as equal; `compare(5.006, 5.001, Units)` rounds them to 5.01 and 5.00 and returns 1. `is_zero(0.004, Units)` is true and `is_zero(0.006, Units)` is false. `round_half_up(2.345, 0.01)` is 2.35 and `round_half_up(−2.345, 0.01)` is −2.35, because an exact tie rounds away from zero. `round_up(7.0001, 1)` is 8 and `round_up(7.0000, 1)` is 7. Converting 46 `Units` into `Box of 6` with a ratio of 6 gives `convert_exact(46, Units, Box of 6)` = 7.666667 and `convert(46, Units, Box of 6)` = 7.67 when the box unit rounds to 0.01.

### 0.2 Date arithmetic

| Concept | Rule |
|---|---|
| Day | Every lead time in this domain is a number of **calendar** days. Weekends, public holidays, working calendars and work centre capacity are never taken into account. |
| `date − n days` | Subtracts *n* calendar days, keeping the time of day. A fractional number of days is truncated to a whole number of days wherever a date is produced, unless the formula says otherwise. |
| `start_of_day(d)` | The moment 00:00:00 of the calendar day of *d*. |
| `end_of_day(d)` | The moment 23:59:59.999999 of the calendar day of *d*. |
| `weekday(d)` | The number of the day of the week of *d*, with Monday = 1 and Sunday = 7. |
| `today` | The current calendar date in the user's time zone. |
| `now` | The current moment in coordinated universal time. |

---

## 1. The rule chain from a location

**Purpose.** Given a product and a location, produce the ordered set of Stock Rules that a need at that location would travel through. The chain is the input of every lead time computation.

**Inputs.** A product, a location, an optional set of preferred routes.

**Output.** A set of Stock Rules.

**Steps.**

1. Let *seen* be the empty set.
2. Let *warehouse* be the warehouse of the current location.
3. Select the rule for the product at the current location with the preferred routes and *warehouse* (algorithm in `workflows.md`, section "Rule selection for a need"), considering active rules only.
4. When the selected rule is already in *seen*, the configuration is circular: raise `Invalid rule's configuration, the following rule causes an endless loop: <rule display name>` and stop.
5. When no rule is found, return *seen*.
6. When the rule's supply method is `make_to_stock`, or the rule's action is neither `pull` nor `pull_push`, return *seen* plus the rule. A `buy` rule and a `manufacture` rule therefore always terminate the chain.
7. Otherwise set the current location to the rule's `location_source`, add the rule to *seen*, and repeat from step 2.

**Worked example.** A warehouse receives in two steps and the product carries the Buy route. A need at the warehouse stock location selects the rule "Vendors to Stock" of the reception route, whose supply method is `make_to_order` when the Purchase Inventory capability package is installed. The chain therefore continues at that rule's source location, the vendor location, where the Buy rule is selected. The Buy rule's action is `buy`, so the chain stops. The chain is {"Vendors to Stock", "Buy"}.

---

## 2. Lead days

**Purpose.** Turn a rule chain into a cumulative number of days and a narrative breakdown.

**Inputs.** A rule chain, a product, and an optional value map that may carry `forced_vendor_price` (a chosen Vendor Price), `bill_of_materials` (a chosen bill of materials) and `days_to_order`.

**Outputs.** A map of named delay components, and an ordered list of description entries. Each description entry is a pair (label, value); the value is either a text (a "+ n day(s)" caption) or a number of days (a date step, see section 3 of `interfaces.md`).

**Delay components.**

| Component | Meaning |
|---|---|
| `total_delay` | The sum that positions the order date relative to the need date. |
| `purchase_delay` | The part of `total_delay` contributed by vendor lead times. Used alone to compute an order date from a planned date. |
| `manufacture_delay` | The part contributed by manufacturing lead times. |
| `no_vendor_found_delay` | The penalty applied when a `buy` rule exists but no vendor price can be found. |
| `no_bill_of_materials_found_delay` | The penalty applied when a `manufacture` rule exists but no bill of materials can be found. |
| `horizon_time` | The company's replenishment horizon. Not part of `total_delay`. |

**Steps.**

1. Start with every component at zero and an empty description.
2. **Rule lead times.** Let *delaying rules* be the rules of the chain whose action is `pull` or `pull_push` and whose `lead_time_days` is not zero. Add the sum of their `lead_time_days` to `total_delay`. Append, for each such rule in chain order, the description entry (`Delay on <rule name>`, `+ <lead time> day(s)`).
3. **Purchasing.** When the chain contains a rule with action `buy`:
   1. Let *vendor price* be `values.forced_vendor_price` when supplied, otherwise the vendor price selected for the product in the company of the `buy` rule with no quantity restriction.
   2. When there is no vendor price at all: add 365 to `total_delay` and to `no_vendor_found_delay`, append the description entry (`No Vendor Found`, `+ 365 day(s)`) and stop the purchasing step.
   3. Otherwise, unless the caller asked to ignore the vendor lead time: add the vendor price's lead time to `total_delay` and to `purchase_delay`, and append the two description entries (`Receipt Date`, *vendor lead time as a date step*) and (`Vendor Lead Time`, `+ <vendor lead time> day(s)`).
   4. Add the company's `days_to_purchase` to `total_delay`, and append the two description entries (`Order Deadline`, *days to purchase as a date step*) and (`Days to Purchase`, `+ <days to purchase> day(s)`). The days to purchase is deliberately **not** added to `purchase_delay`, which is why it widens the forecast window without moving the order deadline of a purchase order.
**Note on purchase-side buffers.** There are exactly two of them: the vendor lead time, which moves both the expected arrival and the order deadline of a purchase order, and the days to purchase, which moves neither and only widens the forecast window. There is no separate purchase security lead time; a company that needs one raises its days to purchase.

4. **Manufacturing.** When the chain contains a rule with action `manufacture`:
   1. Let *bill* be `values.bill_of_materials` when supplied, otherwise the best matching bill of materials for the product, the rule's operation type and the rule's company.
   2. When there is no bill: add 365 to `total_delay` and to `no_bill_of_materials_found_delay`, and append the description entry (`No Bill of Materials Found`, `+ 365 day(s)`). Note that, unlike the vendor case, the computation continues.
   3. Add the bill's manufacturing lead time to `total_delay` and to `manufacture_delay`, and append the two description entries (`Production End Date`, *manufacturing lead time as a date step*) and (`Manufacturing Lead Time`, `+ <manufacturing lead time> day(s)`).
   4. When the bill is of type `normal` and the warehouse of the `manufacture` rule's destination location does not manufacture in one step, build the rule chain for the product at the production location of the product with the warehouse's pre-production route, remove from it the rules already in the current chain, evaluate this same lead-days computation on the remainder with the replenishment horizon forced to zero, and add every resulting component and description entry to the current result.
   5. Add `values.days_to_order` when supplied, otherwise the bill's "days to prepare manufacturing order", to `total_delay`, and append the two description entries (`Production Start Date`, *that number as a date step*) and (`Days to Supply Components`, `+ <that number> day(s)`).
5. **Replenishment horizon.** Unless the caller asked to bypass it, add the applicable replenishment horizon to `horizon_time` and append the description entry (`Time Horizon`, `+ <horizon> day(s)`). The applicable horizon is the value forced in the opening context when there is one, otherwise the `replenishment_horizon_days` of the company of the records being computed, otherwise the `replenishment_horizon_days` of the current user's company.
6. Return the components and the description.

**Worked example 1 (buy chain).** A product has one vendor with a lead time of 7 days. The company's days to purchase is 0 and its replenishment horizon is 365. The reception route rule has no lead time. Then `total_delay` = 0 + 7 + 0 = 7, `purchase_delay` = 7, `horizon_time` = 365. The description is: (`Receipt Date`, 7), (`Vendor Lead Time`, `+ 7 day(s)`), (`Order Deadline`, 0), (`Days to Purchase`, `+ 0 day(s)`), (`Time Horizon`, `+ 365 day(s)`).

**Worked example 2 (buy chain with days to purchase).** Vendor lead time 1 day, days to purchase 2 days, horizon 0. Then `total_delay` = 1 + 2 = 3 and `purchase_delay` = 1. A need created today is forecast three days ahead; the purchase order that satisfies it shows an order deadline of today plus 2 and an expected arrival of today plus 3.

**Worked example 3 (no vendor).** A product carries the Buy route but has no vendor price. Then `total_delay` = 365 and `no_vendor_found_delay` = 365. The reordering rule of that product reports a lead time of 365 days, which is the deliberate signal that the configuration is incomplete.

---

## 3. Lead horizon date and lead days of a reordering rule

```
lead_days            = total_delay of the rule chain of (product, source_location, route)
lead_horizon_date    = today + lead_days + horizon_time
```

`horizon_time` is the company's replenishment horizon (default 365 days). The computation is run with the narrative description switched off, because only the numbers are needed.

When the rule has no product or no source location, `lead_days` is zero and `lead_horizon_date` is empty.

**Worked example.** Today is 1 March. The product has a vendor with a 7-day lead time, the company's days to purchase is 2 and its replenishment horizon is 365. Then `lead_days` = 9 and `lead_horizon_date` = 1 March + 9 + 365 = 10 March of the following year.

---

## 4. Days to order

`days_to_order` is the number of days in advance that the demand is created. It is zero by default and is then overridden, in this order:

1. When the rule chain contains a rule with action `buy`, `days_to_order` becomes the company's `days_to_purchase`.
2. When the rule chain contains a rule with action `manufacture` and the product has at least one bill of materials, `days_to_order` becomes the "days to prepare manufacturing order" of the rule's chosen bill of materials, or of the first variant bill, or of the first product bill, whichever is found first.

The value is passed back into the lead-days computation as `values.days_to_order`, where it replaces the bill's own "days to prepare manufacturing order" (see section 2, step 4.5). For a pure purchasing chain it has no further effect, because the days to purchase is added there anyway.

**Worked example 1 (a purchasing chain).** The company's `days_to_purchase` is 2. The rule chain of product `P` at the warehouse stock location is the single `buy` rule "Vendors to Stock". Step 1 applies, so `days_to_order` = 2. The lead-days computation of section 2 then adds the same 2 days at its step 3.4 whether or not `values.days_to_order` is supplied, so `total_delay` is unchanged: with a vendor lead time of 7, `total_delay` = 7 + 2 = 9.

**Worked example 2 (a manufacturing chain).** The company's `days_to_purchase` is 2. Product `K` carries the Manufacture route only, and its chosen bill of materials has a manufacturing lead time of 4 days and a "days to prepare manufacturing order" of 3 days. Step 1 does not apply, because the chain holds no `buy` rule. Step 2 applies and `days_to_order` = 3. The lead-days computation adds 4 at its step 4.3 and then, at step 4.5, adds `values.days_to_order` = 3 rather than reading the bill again, so `total_delay` = 4 + 3 = 7 and `manufacture_delay` = 4.

**Worked example 3 (a chain that both buys and manufactures).** Product `K` carries both the Buy route and the Manufacture route, and rule selection returns the `manufacture` rule first but the chain walked from it also reaches a `buy` rule for a purchased sub-assembly. Step 1 runs first and sets `days_to_order` = 2 (the days to purchase). Step 2 then runs and overwrites it with the bill's 3 days, because the overrides are applied in the stated order and the last one that applies wins. So `days_to_order` = 3.

**Worked example 4 (manufacture without a bill).** Product `Q` carries the Manufacture route and has no bill of materials at all. Step 1 does not apply and step 2 does not apply either, because it requires at least one bill. `days_to_order` stays at its default of 0, and the lead-days computation adds its 365-day "no bill of materials found" penalty at step 4.2.

---

## 5. Quantity in progress

**Purpose.** Quantities that are already committed but that the stock forecast cannot see, because they are not stock moves yet.

**Inputs.** A set of reordering rules, or a set of (product, location) pairs.

**Output.** For each reordering rule, a quantity expressed in the reordering rule's unit.

**Steps (purchasing contribution).**

1. Select every purchase order line whose `state` is `draft`, `sent` or `to approve` and whose product is one of the products asked for, restricted by the location condition below.
2. Group the selected lines by (order, product, line unit, reordering rule, final location) and sum the ordered quantity.
3. Attribute each group to a location: the `source_location` of the group's reordering rule when there is one; otherwise the group's final location when there is one; otherwise the default destination location of the order's operation type.
4. Convert the summed quantity from the line unit into the product's reference unit **without rounding** and add it to the (product, location) total and to the (product, warehouse of that location) total.

**Location condition.** A line is selected for a set of locations when either:

- it has no reordering rule, and either it has no final location and the default destination location of its order's operation type is one of the locations, or it has no stock moves and its final location is one of the locations or a descendant of one; or
- it has no downstream moves and the `source_location` of its reordering rule is one of the locations.

**Warehouse condition.** A line is selected for a set of warehouses when either it has no reordering rule and the warehouse of its order's operation type is one of the warehouses, or it has no downstream moves and the warehouse of its reordering rule is one of the warehouses.

**Steps (manufacturing contribution).** For a product that is not a kit, the quantities of draft manufacturing orders linked to the reordering rule are added, converted into the reordering rule's unit. For a product that is a kit, the quantity in progress is
```
quantity_in_progress_of_kit =
      min over components of ( available_per_kit(component) + in_progress_per_kit(component) )
    − min over components of ( available_per_kit(component) )
```
where, for each storable component of the exploded kit,
```
quantity_per_kit(component)      = convert(component quantity for one kit, bill line unit, component reference unit)
available_per_kit(component)     = on-hand quantity of the component ÷ quantity_per_kit(component)
in_progress_per_kit(component)   = quantity in progress of the component at the rule location ÷ quantity_per_kit(component)
```
and the result is converted into the reordering rule's unit without rounding. Components that are not storable, and components whose quantity per kit rounds to zero, are skipped.

**Worked example.** A reordering rule watches product P at the warehouse stock location. A request for quotation in state `sent` carries 10 units of P with that reordering rule attached. The stock forecast does not see the request for quotation because no stock move exists yet. The quantity in progress is 10, and the forecast used by the reordering rule is the stock forecast plus 10.

---

## 6. The forecast used by a reordering rule

```
quantity_on_hand  = on-hand quantity of the product in source_location and its descendants
quantity_forecast = forecast quantity of the product in source_location
                    at end_of_day(lead_horizon_date)
                  + quantity_in_progress
```

The forecast quantity is the shared forecast of `../inventory-operations/calculations.md`: on-hand plus incoming minus outgoing, counting only the stock moves that are not yet completed and whose date is not later than the given moment.

**Worked example.** Today is 1 June. The rule's `lead_horizon_date` is 10 June. The product has 20 units on hand at the stock location, an outgoing move of 15 units scheduled on 5 June, an incoming move of 10 units scheduled on 20 June, and a request for quotation in state `draft` for 5 units attached to this rule. Then `quantity_on_hand` = 20 and `quantity_forecast` = 20 − 15 + 0 + 5 = 10; the incoming move of 20 June falls outside the horizon date and is not counted.

---

## 7. Quantity to order

**Purpose.** The quantity a reordering rule proposes.

**Inputs.** `product_minimum_quantity`, `product_maximum_quantity`, `quantity_forecast`, the replenishment multiple, the product unit.

**Output.** `quantity_to_order_computed`, expressed in the reordering rule's unit.

**Steps.**

1. When `compare(quantity_forecast, product_minimum_quantity, product unit)` is not −1, the quantity to order is zero and the computation stops. A forecast equal to the minimum orders nothing.
2. Otherwise read the forecast again at `end_of_day(lead_horizon_date)` and add the quantity in progress; call the result *visible forecast*. (This second read uses the same inputs as `quantity_forecast`; it exists so that a batch computation can reuse one cached read.)
3. Compute the raw quantity:

```
raw_quantity = max(product_minimum_quantity, product_maximum_quantity) − visible_forecast
```

4. Round it to the replenishment multiple (section 8).
5. The result is `quantity_to_order_computed`.

Finally:

```
quantity_to_order = quantity_to_order_manual   when quantity_to_order_manual is not zero
                  = quantity_to_order_computed otherwise
```

**Worked example 1 (the required example).** `product_minimum_quantity` = 10, `product_maximum_quantity` = 50, replenishment multiple = a packaging unit worth 12 product units, `quantity_forecast` = 4.

```
compare(4, 10) = −1                          → a replenishment is needed
raw_quantity   = max(10, 50) − 4 = 46
46 product units ÷ 12 = 3.8333… multiples
round_up(3.8333…, 1) = 4 multiples
4 multiples × 12 = 48 product units
quantity_to_order_computed = 48
```

The forecast after ordering is 4 + 48 = 52, which is above the maximum of 50; the rule therefore also reports `unwanted_replenish` = true (section 10).

**Worked example 2 (no multiple).** Minimum 15, maximum 30, on hand 14.5, nothing else in play. `compare(14.5, 15)` = −1, `raw_quantity` = 30 − 14.5 = 15.5, no multiple applies, so the quantity to order is 15.5.

**Worked example 3 (multiple of ten).** Same rule with a replenishment multiple worth 10 product units. `raw_quantity` = 15.5; 15.5 ÷ 10 = 1.55 multiples; rounded up, 2 multiples; 2 × 10 = 20 product units. The forecast after ordering is 34.5, above the maximum of 30.

**Worked example 4 (multiple equal to the product unit).** Same rule with the product unit itself as the replenishment multiple. `raw_quantity` = 15.5; 15.5 ÷ 1 = 15.5 units; rounded up, 16 units. The quantity to order is 16.

**Worked example 5 (fractional multiple).** Minimum 4, maximum 5.1, forecast 0, multiple a unit worth 0.1 product unit. `raw_quantity` = 5.1; 5.1 ÷ 0.1 = 51 multiples; rounded up, 51; 51 × 0.1 = 5.1. The quantity to order equals the maximum exactly.

---

## 8. Rounding to the replenishment multiple

**Inputs.** A raw quantity expressed in the product's reference unit, the reordering rule's `replenishment_unit_of_measure`, and, when that is empty, the fallback multiple of section 9.

**Output.** A quantity in the product's reference unit.

```
multiple = replenishment_unit_of_measure, or the fallback multiple when that is empty
when no multiple:
    rounded_quantity = raw_quantity
else:
    quantity_in_multiples = convert(raw_quantity, product reference unit, multiple)
    whole_multiples       = round_up(quantity_in_multiples, 1)
    rounded_quantity      = convert(whole_multiples, multiple, product reference unit)
```

The rounding is always **upward**, never to the nearest. Rounding up can push the forecast above the maximum quantity; that is intended, because a partial pack cannot be ordered.

**Worked example 1 (a multiple is set).** The product's reference unit is `Units`. The reordering rule's `replenishment_unit_of_measure` is `Box of 6`, whose ratio is 6 reference units per box. The raw quantity of section 7 is 46 units.

```
quantity_in_multiples = convert(46, Units, Box of 6) = 46 ÷ 6        = 7.666667
whole_multiples       = round_up(7.666667, 1)                       = 8
rounded_quantity      = convert(8, Box of 6, Units) = 8 × 6          = 48
```

The rule orders 48 units, that is 8 boxes. The minimum was 10 and the maximum 50, so the forecast after the order is 4 + 48 = 52, which is 2 units above the maximum. That excess is accepted, because 7 boxes (42 units) would leave the forecast at 46, below the maximum, and a seventh-and-a-half box cannot be bought.

**Worked example 2 (rounding up by a fraction of a unit).** The same rule, with a raw quantity of 42.01 units. Then `quantity_in_multiples` = 42.01 ÷ 6 = 7.001667, `whole_multiples` = 8, and `rounded_quantity` = 48. One hundredth of a unit above a whole box therefore costs a whole extra box, which is the intended behavior of an upward rounding.

**Worked example 3 (an exact multiple).** The same rule, with a raw quantity of 42 units. Then `quantity_in_multiples` = 7.000000, `whole_multiples` = round_up(7, 1) = 7, and `rounded_quantity` = 42. An exact multiple is never pushed to the next one.

**Worked example 4 (no multiple).** The same rule with `replenishment_unit_of_measure` empty and no fallback multiple derivable by section 9. Then `rounded_quantity` = `raw_quantity` = 46 units, unrounded apart from the "Product Unit" precision of section 0.1.

---

## 9. The fallback replenishment multiple

When `replenishment_unit_of_measure` is empty, a fallback multiple may still be derived. It is also what the form shows as a grey placeholder.

1. Let *routes* be the reordering rule's effective route, or, when there is none, the product's own routes.
2. **Manufacturing fallback.** When any rule of *routes* has action `manufacture`: the fallback is the unit of the reordering rule's bill of materials, or, when none is set, of the best matching bill of type `normal` for the product and company.
3. **Purchasing fallback.** Otherwise, when any rule of *routes* has action `buy`:
   1. Compute the procurement date exactly as the scheduler does (`RP-RULE-229`).
   2. Compute the dates information (section 12) for that date, the rule's location and the rule's route.
   3. Let *vendor price* be the rule's `vendor_price` when set, otherwise the vendor price selected for the product in the rule's company for the quantity being rounded, at the later of the date part of the dates-information order date and today, expressed in the rule's unit.
   4. The fallback is the unit of that vendor price.
4. Otherwise there is no fallback and no rounding to a multiple happens.

**Worked example.** A product is bought in boxes of 6. The vendor price is expressed in "Box of 6". The reordering rule leaves the multiple empty. The form shows "Box of 6" as a grey placeholder, and a raw quantity of 46 units is rounded up to 8 boxes, that is 48 units.

---

## 10. Unwanted replenish

```
unwanted_replenish =
      false                                        when the rule has no product
      false                                        when is_zero(quantity_to_order, product unit)
      false                                        when compare(product_maximum_quantity, 0, product unit) = −1
      compare(forecast_after, product_maximum_quantity, product unit) > 0   otherwise

forecast_after = forecast quantity of the product in source_location (no horizon date)
               + quantity_to_order
```

The flag warns the user that ordering the proposed quantity will overshoot the maximum, which happens whenever a replenishment multiple forces a larger order.

**Worked example.** Minimum 10, maximum 50, forecast 4, quantity to order 48 (worked example 1 of section 7). `forecast_after` = 4 + 48 = 52 > 50, so the flag is true.

---

## 11. Deadline date

**Purpose.** The last date on which an order must be placed to avoid falling below the minimum quantity.

**Inputs.** `quantity_on_hand`, `product_minimum_quantity`, `lead_days`, the company's replenishment horizon, the confirmed incoming and outgoing moves of the product at the rule's location.

**Output.** `deadline_date`, a date, or empty.

**Steps.**

1. When `quantity_on_hand` is lower than `product_minimum_quantity`, the deadline date is **today** and the computation stops for that rule. The shortage already exists.
2. Otherwise, for each company separately, let `horizon_date` = `today + replenishment_horizon_days` of that company.
3. Read every stock move of the product whose state is `waiting`, `confirmed`, `assigned` or `partially_available` and whose date is not later than `horizon_date`, split into incoming moves (grouped by product, destination location and calendar day) and outgoing moves (grouped by product, source location and calendar day). Archived moves are included.
4. Build, per (product, location), a map from calendar day to net quantity: incoming quantities add, outgoing quantities subtract.
5. Set *running quantity* to `quantity_on_hand` and *tentative deadline* to `horizon_date`.
6. Walk the days of the map for the rule's (product, `source_location`) in ascending date order. For each day, add the net quantity of that day to *running quantity*. The first time *running quantity* falls below `product_minimum_quantity`, set

```
tentative_deadline = that day − lead_days days
```

   and stop walking.
7. The deadline date is *tentative deadline* when it is strictly earlier than `horizon_date`, and empty otherwise. A rule whose stock never dips inside the horizon therefore shows no deadline.

**Worked example (three products, horizon 365, lead days 0).** Today is 2 September. Each product has 20 units on hand, a minimum of 10 and a maximum of 50.

| Product | Confirmed moves | Running quantity by day | Deadline date |
|---|---|---|---|
| P0 | 15 out on 17 September; 10 in on 27 September | 17 Sept: 20 − 15 = 5, below 10 | 17 September |
| P1 | 10 out on 27 September; 5 out on 7 October | 27 Sept: 10, not below 10; 7 Oct: 5, below 10 | 7 October |
| P2 | 15 out and 15 in on 17 September; 15 out on 27 September | 17 Sept: 20 − 15 + 15 = 20; 27 Sept: 5, below 10 | 27 September |

Before the moves are confirmed, none of them is counted and all three deadline dates are empty. Lowering the company's replenishment horizon to 30 days moves `horizon_date` to 2 October: P0 keeps 17 September, P2 keeps 27 September, and P1 loses its deadline because its dip on 7 October now lies outside the horizon.

**Worked example (with a lead time).** Same product P0, but the chain has `lead_days` = 5. The dip is still on 17 September, so the deadline date is 17 September − 5 = 12 September: the order must be placed five days before the stock would run short.

---

## 12. Dates information: planned date and order date

**Purpose.** Split one "goods needed on date D" into the date the goods must arrive and the date the supplying document must be placed.

**Inputs.** A date *D*, a location, an optional set of routes, a product.

**Output.** A pair.

```
rules        = rule chain of (product, location, routes)              (section 1)
delays       = lead days of that chain, description switched off      (section 2)
date_planned = D
date_order   = D − delays.purchase_delay days
```

Only `purchase_delay` is subtracted. Rule lead times, the days to purchase, the manufacturing lead time and the replenishment horizon do not move the order date.

**Worked example.** A reordering rule needs goods on 10 June. The chain is a receipt rule with no lead time plus a `buy` rule whose vendor has a 3-day lead time; the company's days to purchase is 2. Then `date_planned` = 10 June and `date_order` = 10 June − 3 = 7 June. The days to purchase widened the horizon that produced 10 June in the first place, but does not move the order date.

---

## 13. Purchase order dates

### 13.1 The order date of a new purchase order

```
date_order = minimum over the positive requests of
                 ( values.date_order                                   when it is set
                   values.date_planned − vendor lead time days         otherwise )
```

### 13.2 The planned date of a purchase order line

```
line.date_planned = values.date_planned
```

adjusted by the weekly shift of section 13.4 when the vendor groups weekly on a named weekday.

### 13.3 Advancing the order date for a new line on an existing order

```
planned              = order.date_planned when set, otherwise
                       the minimum date_planned among the lines about to be created
candidate_order_date = planned − vendor lead time days
when date_part(candidate_order_date) < date_part(order.date_order):
    order.date_order = candidate_order_date
```

The order date is only ever moved earlier by this rule, never later.

### 13.4 The weekly shift

When the vendor's grouping mode is `week` and `grouping_weekday` names a target weekday *t*:

```
delta_days        = (7 + t − weekday(values.date_planned)) modulo 7
line.date_planned = values.date_planned + delta_days days
when order.date_planned is empty or order.date_planned ≥ line.date_planned:
    order.date_order = order.date_order + delta_days days
```

Shifting the order date by the same number of days preserves the interval between the order deadline and the expected arrival, which is the vendor lead time.

### 13.5 Worked example: make to order with a three-day vendor and two days to purchase

Configuration: the product carries the Buy route and the "Replenish on Order" route; its single vendor has a lead time of 3 days; the company's `days_to_purchase` is 2; the sales safety days is 0; the warehouse receives in one step; the vendor's grouping mode is `default`.

A sales order line for 10 units is confirmed on day 0 with a promised delivery date on day 20.

| Step | Computation | Result |
|---|---|---|
| The sales order line builds a need at the customer location | `date_deadline` = day 20; `date_planned` = day 20 − sales safety days = day 20 | need on day 20 |
| Rule selection returns the delivery rule, supply method `make_to_order`; the pull action creates the delivery move | `date` = day 20 − 0 = day 20; `deadline` = day 20 | delivery move on day 20 |
| Confirming the delivery move builds a need at the warehouse stock location | dates information: `date_planned` = day 20, `date_order` = day 20 − `purchase_delay` 3 = day 17 | need on day 20, to order on day 17 |
| Rule selection returns the Buy rule; the buy action selects the vendor and creates the purchase order | `date_order` = day 17 (taken from `values.date_order`); the single line's `date_planned` = day 20 | Order Deadline day 17, Expected Arrival day 20 |
| Confirming the purchase order creates the receipt move | `date` = day 20; `deadline` = day 20; linked as the origin move of the delivery move | receipt on day 20 |

The days to purchase of 2 did not move either date on the purchase order. It only widens the forecast window: a reordering rule for the same product would have read its forecast at `today + 3 + 2 + horizon`.

With a sales safety days of 2 the first step gives `date_planned` = day 18 and `date_deadline` = day 20; the purchase order then shows Order Deadline day 15 and Expected Arrival day 18, and the receipt is scheduled on day 18 with a deadline of day 20.

### 13.6 Worked example: the scheduler and a seven-day vendor

The company's replenishment horizon is 365, its days to purchase is 0, and the product's single vendor has a lead time of 7 days. A reordering rule with minimum 10 and maximum 50 has a positive quantity to order. The scheduler runs on 1 March.

```
lead_days         = 7 (vendor) + 0 (days to purchase) = 7
lead_horizon_date = 1 March + 7 + 365 = 8 March of the following year
procurement date  = 8 March of the following year at 12:00 in the company partner's time zone,
                    expressed in coordinated universal time, minus the horizon of 365 days
                  = 8 March of the current year at 12:00
dates information : date_planned = 8 March at 12:00
                    date_order   = 8 March at 12:00 − purchase_delay 7 = 1 March at 12:00
```

The purchase order is therefore created with an order deadline of today at noon and an expected arrival of today plus seven days at noon. This is the just-in-time behavior: the request for quotation is raised exactly the vendor lead time before the goods are needed, and not 365 days early, even though the forecast that decided to order was read 372 days ahead.

---

## 14. Vendor selection

**Purpose.** Choose the Vendor Price that a `buy` request will use.

**Inputs.** The product, the requested quantity, the request unit, the company, and the request values.

**Output.** One Vendor Price, or nothing.

**Steps.**

1. Compute the date: when `values.date_planned` is present, the later of its date part and today; otherwise no date.
2. When `values.forced_vendor_price` is set, that Vendor Price is the answer.
3. Otherwise, when `values.orderpoint` is set and that reordering rule has a `vendor_price`, that Vendor Price is the answer.
4. Otherwise call the shared vendor-selection operation of `../pricing-and-pricelists/` with: the contact restriction (`values.vendor_contact`, or `values.partner` when `values.force_unit_of_measure` is true, otherwise none; always none for a rule of the global Dropship route), the requested quantity, the date from step 1, the request unit, and the parameter `force_unit_of_measure` taken from `values.force_unit_of_measure`. That operation keeps only the vendor prices of the product or of its template that are valid at the date, whose minimum quantity is reached by the requested quantity converted into the vendor's unit, and that belong to this company or to no company; among those it takes the cheapest, and among equally cheap ones the one with the highest minimum quantity.
5. When the previous steps yield nothing, take the first Vendor Price of the product that belongs to this company or to no company, ignoring price, minimum quantity and validity dates.
6. When even that yields nothing, the answer is nothing, and `RP-RULE-094` or `RP-RULE-095` applies.

**Worked example.** A product has two vendor prices for the same vendor: 10 units at 30.00 and 20 units at 25.00. A request for 15 units selects the first bracket, at 30.00. A second request for 10 units arrives and merges into the same line: the line total becomes 25 units, vendor selection runs again for 25 units and now selects the second bracket, so the line price becomes 25.00 for 25 units.

---

## 15. Purchase order line quantities and prices

### 15.1 Updating an existing line

```
added_quantity = round_half_up(convert(request quantity, request unit, line unit), line unit rounding)
new_quantity   = line.product_quantity + added_quantity
vendor_price   = vendor selection for (vendor contact, new_quantity, date_part(order.date_order), line unit, force_unit_of_measure)
price          = fix_tax_included(vendor_price.price, product vendor taxes, line taxes, order company)   when a vendor price was found
               = line.price_unit                                                                          otherwise
price_unit     = convert_currency(price, vendor_price.currency, order.currency, order.company, today)      when the currencies differ
               = price                                                                                    otherwise
```

The line is then written with `product_quantity` = `new_quantity`, `price_unit` = `price_unit`, and its downstream moves extended with the request's downstream moves. When the selected vendor price uses a different unit than the line and `force_unit_of_measure` is not true:

```
line.product_quantity    = round_half_up(convert(new_quantity, line unit, vendor unit), vendor unit rounding)
line.unit_of_measure = vendor unit
```

When the request carries a reordering rule, that rule is written on the line.

**Worked example.** An existing draft purchase order line holds 10 `Units` of product `P` at 30.00 in the order currency, and the order date is 4 March. A new request for 1 `Dozen` of `P` merges into it. The line unit is `Units` and one dozen converts to 12 units, so `added_quantity` = round_half_up(12, 0.01) = 12.00 and `new_quantity` = 10 + 12 = 22.00. Vendor selection for 22 units at 4 March finds the vendor's second bracket, 20 units at 25.00, so `price` = 25.00. The vendor price is in the order currency, so `price_unit` = 25.00. The line is written with `product_quantity` = 22.00 and `price_unit` = 25.00, and the downstream moves of the request are appended to the line's own. The price of the ten units already on the line therefore falls from 30.00 to 25.00, because a purchase order line carries one price for its whole quantity.

**Worked example (a vendor unit differs from the line unit).** The same line, but vendor selection returns a price expressed in `Box of 6` and `force_unit_of_measure` is not set. Then `line.product_quantity` = round_half_up(convert(22, Units, Box of 6), the box rounding of 0.01) = round_half_up(3.666667, 0.01) = 3.67, and `line.unit_of_measure` becomes `Box of 6`. Had `force_unit_of_measure` been true, the line would have stayed at 22.00 `Units`.

### 15.2 Creating a new line

```
when not force_unit_of_measure and vendor unit ≠ request unit:
    line_quantity = convert(request quantity, request unit, vendor unit)
    line_unit     = vendor unit
else:
    line_quantity = request quantity
    line_unit     = request unit
```

The rest of the line values (name, taxes, price, analytic distribution) come from the shared purchase-line preparation of `../purchasing/`.

**Worked example.** A request asks for 12 units. The vendor sells in dozens. `force_unit_of_measure` is false. The created line carries 1 dozen. A second request for 6 units merges into that line: 6 units convert to 0.5 dozen, half-up rounded at the dozen's rounding of 0.01 gives 0.5, and the line becomes 1.5 dozens, that is 18 units.

---

## 16. The grouping windows

Given a request planned date *P* and a vendor:

| Grouping mode | Window on the order's `date_planned` | Reference component |
|---|---|---|
| `default` ("On Order") | none | the order must share one of the request's references; when the request has none, the order must have none |
| `all` ("Always") | none | none |
| `day` ("Daily") | `start_of_day(P)` to `end_of_day(P)` | only when the operation type has code `dropship` |
| `week` ("Weekly"), `grouping_weekday` = `default` | `start_of_day(P − weekday(P) days)` to `end_of_day(P + (6 − weekday(P)) days)` | only when the operation type has code `dropship` |
| `week`, `grouping_weekday` = a weekday *t* | `start_of_day(P + delta)` to `end_of_day(P + delta)` where `delta = (7 + t − weekday(P)) modulo 7` | only when the operation type has code `dropship` |

**Worked example (weekly window, expected date).** *P* is Wednesday 10 September, so `weekday(P)` = 3. The window runs from the start of Sunday 7 September to the end of Saturday 13 September. Two needs planned on 11 and 12 September fall in the same window and share one order.

**Worked example (weekly window, named weekday).** Today is Wednesday 10 September at 10:00. The vendor groups weekly on Tuesday, so *t* = 2. A first need is planned on Friday 12 September: `weekday` = 5, `delta` = (7 + 2 − 5) modulo 7 = 4, so the line is planned on Tuesday 16 September. A second need is planned on Saturday 13 September: `weekday` = 6, `delta` = (7 + 2 − 6) modulo 7 = 3, so that line is also planned on Tuesday 16 September, and the two needs share one order. The second product's vendor lead time is 2 days, so the order's deadline is 16 September − 2 = Sunday 14 September.

---

## 17. Move scheduled date, deadline and lateness

### 17.1 First assignment

```
pull rule:   move.date     = values.date_planned  − rule.lead_time_days days
             move.deadline = values.date_deadline − rule.lead_time_days days   (empty when there is no deadline)
push rule:   move.date     = source move.date     + rule.lead_time_days days
             move.deadline = source move.deadline                              (copied unchanged)
purchase:    move.date     = move.deadline = line.date_planned, or order.date_planned when the line has none
```

**Worked example (pull).** A need is planned for 20 March at 12:00 with a deadline of 20 March at 12:00, and the selected pull rule has `lead_time_days` = 2. The created move gets `date` = 18 March at 12:00 and `deadline` = 18 March at 12:00. Had the need carried no deadline, the move's deadline would be empty while its date would still be 18 March at 12:00.

**Worked example (push).** The same move is completed and a push rule with `lead_time_days` = 1 applies to it. The pushed move gets `date` = 19 March at 12:00, that is the source move's date plus one day, and `deadline` = 18 March at 12:00, copied unchanged from the source move. The pushed move is therefore already reported late relative to its own deadline, which is what makes the delay alert of 17.3 fire.

**Worked example (purchase).** A confirmed purchase order line is planned for 6 March at 09:00. The receipt move created for it gets `date` = 6 March at 09:00 and `deadline` = 6 March at 09:00. When the line carries no planned date of its own, both take the order's planned date instead.

### 17.2 Deadline propagation

```
delta = current deadline − new deadline          (zero when the move had no deadline)
for every origin and destination move M that is neither done nor cancelled and not yet visited:
    when M has a deadline and delta ≠ 0:
        M.deadline = M.deadline − delta          (which propagates recursively)
```

The visited set prevents a cycle from looping forever. The effect is that the whole chain shifts by the same amount while keeping the relative offsets that the rule lead times introduced.

**Worked example.** A delivery move and its origin internal move both have a deadline of day 30. The delivery deadline is moved to day 24. `delta` = 30 − 24 = 6 days, so the origin move's deadline becomes 30 − 6 = day 24 as well. Completing the origin move afterwards sets its scheduled date to the moment of completion but leaves both deadlines at day 24.

### 17.3 Delay alert

```
delay_alert_date = max( scheduled date of the origin moves that are not done )
                   when that maximum is later than this move's own scheduled date
                 = empty otherwise
                 = empty always, for a move that is done or cancelled
```

**Worked example.** A delivery is scheduled on day 10. Its origin receipt is scheduled on day 12 and is not completed. The delivery's delay alert date is day 12 and the delivery is shown as late; the pop-over names the receipt as the responsible document.

---

## 18. The "take from stock, otherwise trigger another rule" split

For a move whose rule's supply method is `make_to_stock_else_make_to_order`, at confirmation:

```
when move real quantity ≤ 0 or the move's source location bypasses reservation:
    quantity_to_procure = move.demand_quantity
else:
    free       = free quantity of the product at the move's source location      (read once per pair)
    available  = max(free − already_consumed[source location, product], 0)
    missing    = max(move.demand_quantity_in_reference_unit − available, 0)
    quantity_to_procure = round_half_up(convert(missing, product reference unit, move unit), move unit rounding)
    already_consumed[source location, product] += min(move.demand_quantity_in_reference_unit, available)
```

The move itself stays with supply method `make_to_stock` and keeps its full demand; only the missing part becomes a new need, and that new need is **not** linked to the move (`RP-RULE-143`).

**Worked example.** Two moves of 10 units each are confirmed in the same batch from a location holding 12 free units. The first move sees `available` = 12, `missing` = 0, and consumes 10. The second move sees `available` = 12 − 10 = 2, `missing` = 8, and consumes 2. One need for 8 units is created.

---

## 19. Forecast reconciliation

**Purpose.** Explain, line by line, where every unit of forecast stock in one warehouse comes from and where it goes.

**Inputs.** A set of products, the warehouse locations (the warehouse's view location and all its descendants), and the warehouse's stock location.

**Output.** An ordered list of report lines. Each line carries a quantity, optionally an outgoing move, optionally an incoming move, a "replenishment filled" flag, an "in transit" flag, an optional reservation document, and the lateness flags.

**Step 1: collect the moves.**

- *Outgoing moves*: moves with a non-zero demand quantity whose source location is one of the warehouse locations and whose destination location is **not** one of them, or whose final location is set and is not one of them; and whose state is `waiting`, `confirmed`, `partially_available` or `assigned`.
- *Incoming moves*: moves with a non-zero demand quantity whose source location is **not** one of the warehouse locations and whose destination location is; and whose state is one of the same four.

**Step 2: order the moves.**

- Outgoing moves whose reservation date is not later than today, ordered by priority descending, then scheduled date ascending, then surrogate key ascending.
- Then outgoing moves whose reservation date is later than today or empty, ordered by reservation date ascending, then priority descending, then scheduled date ascending, then surrogate key ascending.
- Incoming moves are ordered by priority descending, then scheduled date ascending, then surrogate key ascending.

**Step 3: build the linked-move set per outgoing move.** Roll up the origin moves of each outgoing move recursively, stopping at the incoming moves, and remove the incoming moves themselves from the result. The remainder is the set of internal moves that feed that outgoing move.

**Step 4: read the current stock.** Read the positive stock quantity records in the warehouse locations, grouped by product and location. A quantity in a descendant of the warehouse stock location is added both to its own location total and to the warehouse stock location total.

**Step 5: count the reserved quantity per outgoing move.** For each product, for each outgoing move in order, walk its linked moves:

```
for each linked move whose state is partially_available or assigned:
    reserved = convert(linked move.quantity, linked move unit, product reference unit)
    reserved = min(reserved − already_used[linked move], outgoing move.demand_quantity_in_reference_unit)
    reserved_out += reserved
    already_used[linked move] += reserved
    current[product, linked move source location] −= reserved
    when that source location is a descendant of the warehouse stock location:
        current[product, warehouse stock location] −= reserved
    stop when reserved_out reaches the outgoing move's quantity
```

The first linked move that contributed a non-zero reserved quantity is remembered as the reservation document of the line.

**Step 6: count what is taken from unreserved stock.** For the same outgoing move:

```
demand_out = outgoing move.demand_quantity_in_reference_unit − reserved_out
for each linked move whose state is not draft, cancel, assigned or done:
    reserved  = convert(linked move.quantity, linked move unit, product reference unit)
    demand    = max(linked move.demand_quantity_in_reference_unit − reserved, 0)
    demand    = min(demand, demand_out)
    skip when demand rounds to zero
    when the linked move has origin moves:
        move_available = (sum of the quantities of its completed origin moves)
                       − (sum of the quantities of the completed moves fed by those origin moves, other than itself)
                       − reserved
    else:
        move_available = current[product, linked move source location]
    taken = min(demand, move_available, current[product, linked move source location])
    when taken > 0:
        current[product, linked move source location] −= taken
        when that source location is a descendant of the warehouse stock location:
            current[product, warehouse stock location] −= taken
        taken_from_stock_out += taken
    demand_out −= taken
```

**Step 7: emit the lines.** For each product:

```
free_stock    = current[product, warehouse stock location]
transit_stock = (sum of current[product, L] over the warehouse locations L that are not descendants
                 of the warehouse stock location) − free_stock
```

then for each outgoing move of that product, in order:

1. When `reserved_out` is greater than zero, emit a line of that quantity with the outgoing move, the reservation document, and the "in transit" flag set when the reservation document itself has origin moves; subtract it from the remaining demand.
2. When the remaining demand rounds to zero, continue with the next outgoing move.
3. When `taken_from_stock_out` is greater than zero, emit a line of that quantity with the outgoing move; subtract it.
4. When the remaining demand rounds to zero, continue.
5. Take `unreservable = min(remaining demand, transit_stock)`; when it is greater than zero, emit a line of that quantity with the outgoing move and the "in transit" flag, subtract it from both the remaining demand and `transit_stock`.
6. When the remaining demand rounds to zero, continue.
7. Reconcile with the incoming moves that are linked to this outgoing move: for each such incoming move with a remaining quantity, take `min(remaining demand, remaining quantity of that incoming move)`, emit a line carrying both moves, and reduce both counters. Stop when the remaining demand rounds to zero.
8. When demand remains, remember the outgoing move as unreconciled.

Then, in a second pass, reconcile every unreconciled outgoing move with **any** remaining incoming move of the product, emitting one line per pairing. What still remains after that pass is emitted as one line with the outgoing move and the "replenishment filled" flag set to false.

Finally: when `transit_stock` does not round to zero, emit a line for it with the "in transit" flag and no move; when `free_stock` does not round to zero, or when no line at all was emitted for this product, emit the free-stock lines; and for each incoming move with a remaining quantity, emit a line with that incoming move alone.

**Line flags.**

```
is_late       = outgoing move.date < incoming move.date       (only when the line carries both)
delivery_late = outgoing move is not done and its date < now
receipt_late  = incoming move is not done and its date < now
```

**Worked example.** A warehouse holds 5 units of a product in its stock location. Two deliveries are confirmed: D1 for 3 units on day 5 (reserved) and D2 for 6 units on day 8 (not reserved). One receipt R1 for 4 units is expected on day 7, linked to nothing.

1. D1: reserved 3. Line 1: quantity 3, outgoing D1, reservation D1. Current stock drops to 2.
2. D2: reserved 0, taken from stock `min(6, 2, 2)` = 2. Line 2: quantity 2, outgoing D2. Current stock drops to 0. Remaining demand 4.
3. D2 has no linked incoming move, so nothing is reconciled in the first pass; D2 is unreconciled with 4.
4. Second pass: R1 has 4 units left. Line 3: quantity 4, outgoing D2, incoming R1. The line's `is_late` flag is false because R1 (day 7) is not later than D2 (day 8).
5. No free stock and no transit stock remain, and no incoming move has a remaining quantity, so nothing more is emitted.

---

## 20. The replenishment demand graph

**Purpose.** Show, for one reordering rule, how long the interval between two replenishments would be at the historic rate of demand.

**Inputs.** The reordering rule, the wizard's `based_on` period, the wizard's `percent_factor`, the minimum and maximum quantities.

**Step 1: the period.**

| `based_on` | Start date | Limit date |
|---|---|---|
| `one_week` | now minus 1 week | now |
| `one_month` | now minus 1 month | now |
| `three_months` | now minus 3 months | now |
| `one_year` | now minus 1 year | now |
| `last_year` | the first day of the current month of last year | start plus 1 month |
| `last_year_next_month` | the first day of the current month of last year plus 1 month | start plus 1 month |
| `last_year_month_after_next` | the first day of the current month of last year plus 2 months | start plus 1 month |
| `last_year_quarter` | the first day of the current month of last year | start plus 3 months |

**Step 2: the quantities.** Over the stock moves of the product in the rule's company whose state is `assigned`, `confirmed`, `partially_available` or `done` and whose date lies between the start date and the end of the limit date:

```
quantity_out      = sum of the real quantity of the moves whose destination location usage is customer or production
quantity_returned = sum of the real quantity of the moves whose source location usage is customer
```

**Step 3: the daily demand.**

```
daily_demand = ( (quantity_out − quantity_returned) ÷ (limit_date − start_date in days) )
               × (percent_factor ÷ 100)
```

**Step 4: the derived figures.**

```
when product_maximum_quantity < product_minimum_quantity:
    product_maximum_quantity = product_minimum_quantity
average_stock   = product_minimum_quantity + (product_maximum_quantity − product_minimum_quantity) ÷ 2
quantity_range  = product_maximum_quantity − product_minimum_quantity, or 1 when that is zero
ordering_period = max(1, integer part of (quantity_range ÷ daily_demand))       when daily_demand ≠ 0
                = 0                                                             when daily_demand = 0
```

`daily_demand` and `average_stock` are rounded to the rounding of the product's unit; `ordering_period` is rounded to a whole number of days.

**Step 5: the curve.** When `daily_demand` is zero, the horizontal axis carries the two empty labels `''` and `' '`, and the saw-tooth curve is empty. Otherwise the horizontal axis carries the empty label followed by the three labels `In <1 × ordering_period> day(s)`, `In <2 × ordering_period> day(s)` and `In <3 × ordering_period> day(s)`; the saw-tooth curve starts at the maximum at the empty label, and then, at each of the three labels, drops to the minimum and rises back to the maximum, except that the very last rise is removed. The maximum line and the minimum line are drawn as horizontal lines through every label.

**Worked example 1.** Today is 14 August. The rule has minimum 10 and maximum 50. One delivery of 15 units was completed and no return exists. `based_on` is `one_month` and `percent_factor` is 100.

```
period            = 14 July to 14 August = 31 days
quantity_out      = 15, quantity_returned = 0
daily_demand      = (15 − 0) ÷ 31 × 1 = 0.4838… → 0.48
average_stock     = 10 + (50 − 10) ÷ 2 = 30.0
quantity_range    = 40
ordering_period   = max(1, integer part of (40 ÷ 0.4838…)) = max(1, 82) = 82
horizontal axis   = ['', 'In 82 day(s)', 'In 164 day(s)', 'In 246 day(s)']
curve             = [50, 10, 50, 10, 50, 10]
```

**Worked example 2.** The same rule, now with `based_on` = `one_week`, `percent_factor` = 200, minimum 20 and maximum 40.

```
period            = 7 August to 14 August = 7 days
daily_demand      = (15 ÷ 7) × 2 = 4.2857… → 4.29
average_stock     = 20 + (40 − 20) ÷ 2 = 30.0
quantity_range    = 20
ordering_period   = max(1, integer part of (20 ÷ 4.2857…)) = max(1, 4) = 4
horizontal axis   = ['', 'In 4 day(s)', 'In 8 day(s)', 'In 12 day(s)']
curve             = [40, 20, 40, 20, 40, 20]
```

**Worked example 3.** A further outgoing move of 15 units dated five days in the past is confirmed. It falls inside the seven-day window and its state is `confirmed`, so it counts.

```
daily_demand      = ((15 + 15) ÷ 7) × 2 = 8.5714… → 8.57
ordering_period   = max(1, integer part of (20 ÷ 8.5714…)) = max(1, 2) = 2
horizontal axis   = ['', 'In 2 day(s)', 'In 4 day(s)', 'In 6 day(s)']
```

---

## 21. The replenishment report shortage detection

**Purpose.** Find every (replenishment location, product) pair whose forecast is negative, cheaply.

**Step 1: the scope.** Every goods product that has at least one stock move; every location whose `replenish_location` flag is true.

**Step 2: the fast filter.** Read, in three grouped queries over all products at once:

- the stock quantity records inside the replenishment locations, summed per product and location;
- the incoming moves in state `waiting`, `confirmed`, `assigned` or `partially_available` whose destination or final location is inside the locations, or which are internal to them, summed per product, destination location and final location;
- the outgoing moves in the same states whose source location is inside the locations, or which are internal to them, summed per product and source location.

For each (location, product) pair:

```
candidate = compare(on_hand + incoming − outgoing, 0, product unit) < 0
```
counting only the records whose location is the replenishment location itself or a descendant of it.

**Step 3: group by lead time.** For each candidate, build the rule chain at that location and compute its lead days, and file the product under the key (`total_delay + horizon_time`, location).

**Step 4: the exact pass.** For each key, read the forecast of its products at that location at `end_of_day(today) + <the key's day count> days`. Keep the pairs whose forecast is still negative; the shortage is the forecast value, a negative number.

**Step 5: deduct what is in progress.** For the surviving pairs, read the quantity in progress per (product, location) (section 5) and add the `quantity_to_order` of the reordering rules that already exist for the same pair:

```
in_progress = quantity_in_progress[product, location] + sum of quantity_to_order of existing rules
when in_progress = 0:
    shortage stays unchanged
else:
    shortage = shortage + in_progress
```

Pairs whose shortage is no longer strictly negative at the "Product Unit" precision are dropped.

**Step 6: create or top up.** For each remaining pair: when a reordering rule already exists (including an archived one), increase its `quantity_forecast` by the (negative) shortage; otherwise create a temporary rule (`RP-RULE-192`).

**Worked example.** A warehouse receives in one step and has no stock of product P. A delivery of 3 units of P is confirmed from the warehouse stock location to a replenishment sub-location. The fast filter finds, at the warehouse stock location, `0 + 0 − 3` = −3, a candidate. The rule chain at that location is the reception rule, whose lead days are 0, plus a horizon of 365, so the exact pass reads the forecast at today plus 365 days: still −3. No purchase order line and no existing rule contribute, so the shortage stays −3. One temporary rule is created at the warehouse stock location with a quantity to order of 3.

---

## 22. On-time delivery rate

### 22.1 Per vendor contact

**Inputs.** The stored parameter `purchasing.on_time_delivery_days`, default 365.

```
window_start = today − on_time_delivery_days days
lines        = purchase order lines such that
                   the line's contact is this contact
               AND the line's order date is later than window_start
               AND the line's received quantity is not zero
               AND the order's state is "purchase"
               AND the product is not a service
on_time_quantity(line) = sum of the quantity of the completed stock moves of the line
                         whose date part is not later than the date part of the line's planned date
on_time_rate = ( sum over lines of on_time_quantity(line) )
             ÷ ( sum over lines of the line's ordered quantity ) × 100
             = −1 when the denominator is zero
```

The sentinel value −1 means "no data" and is displayed as such rather than as a rate of zero percent.

**Worked example.** A vendor has two qualifying lines. Line A ordered 10 units, planned on 3 May, and its receipt of 10 units was completed on 2 May: 10 on time. Line B ordered 5 units, planned on 3 May, and its receipt of 5 units was completed on 5 May: 0 on time. The rate is `(10 + 0) ÷ (10 + 5) × 100` = 66.67 percent.

### 22.2 The Vendor Delay Report

One row per purchase order line that has at least one stock move:

```
date            = minimum date among the stock moves of the line
quantity_total  = the ordered quantity of the line, in the line's unit
quantity_on_time = sum over the stock move lines of the line's moves of
                       ( move line quantity × factor(move line unit) ÷ factor(product reference unit) )
                   counting only the move lines whose move state is "done"
                   and whose move date part is not later than the line's planned date part;
                   every other move line contributes zero
```

When a report query asks for the sum of the on-time rate, the value returned is a weighted average, not a sum:

```
on_time_rate_percentage = sum(quantity_on_time) ÷ sum(quantity_total) × 100   when sum(quantity_total) ≠ 0
                        = 100                                                 otherwise
```

and groups whose `sum(quantity_total)` is not greater than zero are removed from the result.

**Worked example.** A purchase order line ordered 12 units and was received in two move lines: 8 units on a move completed on time and 4 units on a move completed late. `quantity_total` = 12, `quantity_on_time` = 8, and the reported rate is `8 ÷ 12 × 100` = 66.67 percent. A second line on the same vendor ordered 10 units, all on time. Grouped by vendor, the rate is `(8 + 10) ÷ (12 + 10) × 100` = 81.82 percent, not the arithmetic mean of 66.67 and 100.

---

## 23. Purchase suggestion quantities

**Purpose.** Propose, inside the product catalogue of a purchase order, how much of each product to buy.

### 23.1 Monthly demand

**Inputs.** The contact's `suggest_based_on`, the warehouse named in the opening context.

Period:

| `suggest_based_on` | Start date | Limit date | Divisor |
|---|---|---|---|
| empty, `actual_demand` or `30_days` | now minus 30 days | end of today | 1 |
| `one_week` | now minus 1 week | end of today | 7 ÷ (365.25 ÷ 12) |
| `three_months` | now minus 3 months | end of today | 3 |
| `one_year` | now minus 1 year | end of today | 12 |
| `last_year` | the first day of the current month of last year | start plus 1 month | 1 |
| `last_year_next_month` | that day plus 1 month | start plus 1 month | 1 |
| `last_year_month_after_next` | that day plus 2 months | start plus 1 month | 1 |
| `last_year_quarter` | the first day of the current month of last year | start plus 3 months | 3 |

Moves counted: state `waiting`, `assigned`, `confirmed`, `partially_available` or `done`, date in the period, and:

- when no warehouse is named: destination location usage is customer, production or transit; or the final location usage is customer or production and the move has no downstream move;
- when a warehouse is named: the source location belongs to that warehouse, and (the destination location belongs to a different warehouse, or the final location belongs to a different warehouse and the move has no downstream move), and the destination location usage is not inventory-loss (which excludes scrapping).

```
monthly_demand = ( sum of the real quantity of the counted moves ) ÷ divisor
```

The divisor turns the quantity observed over the period into a quantity per month: a period of three months is divided by 3, a period of one year by 12, a period of one week by the fraction of a month that one week represents, and a period that is already one month long by 1. The average month used throughout is 365.25 ÷ 12 = 30.4375 days.

**Worked example 1 (three months).** `suggest_based_on` is `three_months`. The counted moves out of the warehouse over the last three months total 138 units. Then `monthly_demand` = 138 ÷ 3 = 46.00 units per month.

**Worked example 2 (one week).** `suggest_based_on` is `one_week`. The divisor is 7 ÷ 30.4375 = 0.229979. The counted moves over the last seven days total 14 units. Then `monthly_demand` = 14 ÷ 0.229979 = 60.88 units per month, rounded to two decimal places for display. The same demand expressed over a week (14) and over a month (60.88) is consistent: 60.88 × 0.229979 = 14.00.

**Worked example 3 (same month last year).** `suggest_based_on` is `last_year` and today is 12 September. The period runs from 1 September of last year to 1 October of last year, the divisor is 1, and the counted moves total 52 units. Then `monthly_demand` = 52 ÷ 1 = 52.00 units per month.

**Worked example 4 (last year quarter).** `suggest_based_on` is `last_year_quarter` and today is 12 September. The period runs from 1 September of last year to 1 December of last year, the divisor is 3, and the counted moves total 150 units. Then `monthly_demand` = 150 ÷ 3 = 50.00 units per month.

### 23.2 Suggested quantity

```
when suggest_based_on = "actual_demand":
    when forecast quantity ≥ 0: suggested_quantity = 0
    else: suggested_quantity = max( round_up( −forecast quantity × suggest_percent ÷ 100, 1 ), 0 )
else:
    when monthly_demand ≤ 0: suggested_quantity = 0
    else:
        monthly_ratio      = suggest_days ÷ (365.25 ÷ 12)
        raw                = monthly_demand × monthly_ratio × suggest_percent ÷ 100
        raw                = raw − ( max(on-hand quantity, 0) + max(incoming quantity, 0) )
        suggested_quantity = max( round_up(raw, 1), 0 )
```

The suggested quantity is always a whole number.

When `suggest_based_on` is set and `suggest_days` is present, the forecast quantities of the product are themselves read at `now + suggest_days days` rather than over all time, so that the kanban card shows the forecast at the end of the covered period.

**Worked example 1 (the historic branch).** `suggest_based_on` is `three_months`, so `monthly_demand` is 46.00 as computed in worked example 1 of section 23.1. The contact's `suggest_days` is 14 and its `suggest_percent` is 110. The product has 2 units on hand and none incoming.

```
monthly_ratio      = 14 ÷ 30.4375                    = 0.459951
raw                = 46.00 × 0.459951 × 110 ÷ 100    = 23.273505
raw                = 23.273505 − (max(2, 0) + max(0, 0)) = 21.273505
suggested_quantity = max( round_up(21.273505, 1), 0 )   = 22
```

**Worked example 2 (the actual-demand branch, a shortage).** `suggest_based_on` is `actual_demand` and `suggest_percent` is 120. The forecast quantity of the product is −12.5.

```
forecast quantity  = −12.5, which is below zero
suggested_quantity = max( round_up( 12.5 × 120 ÷ 100, 1 ), 0 )
                   = max( round_up( 15.0, 1 ), 0 )       = 15
```

**Worked example 3 (the actual-demand branch, no shortage).** Same contact, but the forecast quantity of the product is +4.0. Because the forecast is not below zero, `suggested_quantity` is 0 and the product is not proposed.

**Worked example 4 (the historic branch cancelled by stock).** `monthly_demand` is 46.00, `suggest_days` is 7, `suggest_percent` is 100, the product has 20 units on hand and 5 incoming.

```
monthly_ratio      = 7 ÷ 30.4375                     = 0.229979
raw                = 46.00 × 0.229979 × 100 ÷ 100    = 10.579034
raw                = 10.579034 − (20 + 5)            = −14.420966
suggested_quantity = max( round_up(−14.420966, 1), 0 ) = 0
```

### 23.3 Estimated price

```
seller = vendor selection for suggested_quantity, or, failing that, the vendor price with the lowest minimum quantity
price  = seller.price_discounted   when a seller was found
       = product cost price        otherwise
suggest_estimated_price = price × suggested_quantity
```

**Worked example.** A product has a monthly demand of 60 units. The contact's `suggest_days` is 7 and `suggest_percent` is 100. On hand 5, incoming 3.

```
monthly_ratio      = 7 ÷ (365.25 ÷ 12) = 7 ÷ 30.4375 = 0.22998…
raw                = 60 × 0.22998… × 1 = 13.799…
raw                = 13.799… − (5 + 3) = 5.799…
suggested_quantity = round_up(5.799…, 1) = 6
```

At a vendor price of 4.50 for a bracket that 6 units reaches, the estimated price is 6 × 4.50 = 27.00.

---

## 24. The rule description message

The sentence shown on a Stock Rule form is built from four values:

```
source            = location_source display name,      or "Source Location" when empty
destination       = destination_location display name, or "Destination Location" when empty
direct_destination= the operation type's default destination location display name,
                    but only when it differs from destination_location; false otherwise
operation         = operation_type name,               or "Operation Type" when empty
```

A suffix is then built:

```
suffix = ""
when action is pull or pull_push, direct_destination exists and location_destination_from_rule is false:
    suffix += "<br>The products will be moved towards <b>{direct_destination}</b>, <br/> as specified from <b>{operation}</b> destination."
when procure_method = make_to_order and location_source is set:
    suffix += "<br>A need is created in <b>{source}</b> and a rule will be triggered to fulfill it."
when procure_method = make_to_stock_else_make_to_order and location_source is set:
    suffix += "<br>If the products are not available in <b>{source}</b>, a rule will be triggered to bring the missing quantity in this location."
```

and the message itself:

| Action | Message |
|---|---|
| `pull` | `When products are needed in <b>{destination}</b>, <br> <b>{operation}</b> are created from <b>{source}</b> to fulfill the need. {suffix}` |
| `push` | `When products arrive in <b>{source}</b>, <br> <b>{operation}</b> are created to send them to <b>{destination}</b>.` |
| `pull_push` | the pull message, then two line breaks, then the push message |
| `buy` | `When products are needed in <b>{destination}</b>, <br/> a request for quotation is created to fulfill the need.<br/>Note: This rule will be used in combination with the rules<br/>of the reception route(s)` |
| `manufacture` | The manufacturing message owned by `../manufacturing/`. |

**Worked example.** A pull rule from "WH/Stock" to "Partners/Customers" with the operation type "Delivery Orders", supply method `make_to_order`, produces: `When products are needed in <b>Partners/Customers</b>, <br> <b>Delivery Orders</b> are created from <b>WH/Stock</b> to fulfill the need. <br>A need is created in <b>WH/Stock</b> and a rule will be triggered to fulfill it.`

---

## 25. Defaults derived for a reordering rule

### 25.1 Default route

1. **Purchasing.** When the product has at least one Vendor Price and the intersection of the rule chain's routes with the routes of the `buy` rules is not empty, the first route of that intersection is the default route.
2. **Manufacturing.** Otherwise, when the product has at least one bill of materials and the intersection of the rule chain's routes with the routes of the `manufacture` rules is not empty, the first route of that intersection is the default route.
3. **Base.** Otherwise, read the active rules whose action is `pull` or `pull_push`, whose route is selectable on products or on product categories, and whose destination location is the rule's `source_location`, grouped by destination location and route; the first route of that grouping that is one of the product's own routes or one of the product's category's routes, and whose destination location is exactly the rule's `source_location`, is the default route.
4. When nothing matches, there is no default route.

`effective_route` is the rule's own `route` when set, otherwise the default route. `route_identifier_placeholder` is the display name of the default route, shown as a grey placeholder.

### 25.2 Default vendor price

When the rule shows the vendor column (that is, when `effective_route` is one of the routes holding a `buy` rule), the default vendor price is the result of vendor selection (section 14) for the product, the rule's `quantity_to_order`, the rule's unit, the rule's company, and an empty value map. Otherwise there is no default vendor price.

`effective_vendor` is the contact of the rule's `vendor_price` when set, otherwise the contact of the default vendor price.

### 25.3 Default bill of materials

When the rule shows the bill column (that is, when `effective_route` is one of the routes holding a `manufacture` rule), the default bill of materials is the bill that the rule chain's `manufacture` rule would match for the product and company. Otherwise there is no default bill.

### 25.4 Allowed replenishment multiples

```
allowed_replenishment_unit_of_measures = the units of the product
                                       ∪ the units of the product's Vendor Prices, when any rule of the chain has action buy
                                       ∪ the unit of every matching bill of materials, when any rule of the chain has action manufacture
```

### 25.5 Worked examples

**Worked example 1 (a purchased product).** Product `P` carries the routes "Buy" and "Vendors to Stock", has two Vendor Prices (vendor `V1` at 30.00 for 10 units expressed in `Box of 6`, vendor `V2` at 28.00 for 50 units expressed in `Units`) and has no bill of materials. A reordering rule for `P` at `WH/Stock` has `quantity_to_order` = 25 and leaves `route`, `vendor_price` and `replenishment_unit_of_measure` empty.

- 25.1: the product has at least one Vendor Price, and the rule chain at `WH/Stock` contains the `buy` rule of the "Buy" route, so the intersection of the chain's routes with the routes of the `buy` rules is {Buy} and the default route is "Buy". `effective_route` = "Buy", and `route_identifier_placeholder` shows `Buy` in grey.
- 25.2: `effective_route` holds a `buy` rule, so the vendor column is shown. Vendor selection (section 14) for 25 units selects `V1` at 30.00, because its bracket of 10 units is satisfied by 25 and it is the first eligible row. The default vendor price is that row; `effective_vendor` = `V1`.
- 25.3: `effective_route` holds no `manufacture` rule, so there is no default bill of materials and the bill column is hidden.
- 25.4: the chain has action `buy`, so the allowed replenishment multiples are the units of `P` (`Units`, and any packaging unit of `P`) together with the units of its Vendor Prices (`Box of 6` and `Units`). The list therefore offers `Units` and `Box of 6`.

**Worked example 2 (a manufactured product).** Product `M` carries the routes "Manufacture" and "Replenish on Order", has no Vendor Price, and has two bills of materials of type `normal`, the first expressed in `Units` and the second in `Pallet of 100`.

- 25.1: rule 1 does not apply, because `M` has no Vendor Price. Rule 2 applies: `M` has a bill of materials and the chain at `WH/Stock` contains the `manufacture` rule of the "Manufacture" route, so the default route is "Manufacture".
- 25.2: `effective_route` holds no `buy` rule, so the vendor column is hidden and there is no default vendor price.
- 25.3: the bill column is shown, and the default bill of materials is the first bill, the one the `manufacture` rule would match for `M` in the rule's company.
- 25.4: the allowed multiples are the units of `M` together with the units of both matching bills, that is `Units` and `Pallet of 100`.

**Worked example 3 (neither buy nor manufacture).** Product `T` is resupplied from another warehouse. It carries neither a Vendor Price nor a bill of materials, and its own routes include "WH-A: Supply Product from WH-B".

- 25.1: rules 1 and 2 do not apply. Rule 3 reads the active `pull` and `pull_push` rules that are selectable on products or product categories and whose destination location is `WH-A/Stock`; the resupply rule "WH-B: Transit to WH-A Stock" qualifies, its route is one of `T`'s own routes, and its destination location is exactly `WH-A/Stock`, so the default route is "WH-A: Supply Product from WH-B".
- 25.2 and 25.3: neither the vendor column nor the bill column is shown, so there is no default vendor price and no default bill.
- 25.4: the chain has neither a `buy` nor a `manufacture` rule, so the allowed multiples are the units of `T` alone.

**Worked example 4 (nothing matches).** Product `X` has no Vendor Price, no bill of materials, and no route at all, and its product category has none either. All four steps of 25.1 fail, so there is no default route, `route_identifier_placeholder` is empty, `effective_route` is empty, both columns are hidden, and the allowed multiples are the units of `X` alone. Section 26 then raises the supply warning, because the rule chain is empty.

---

## 26. The supply warning

```
show_supply_warning = true   when the rule chain is empty
                    = true   when the rule chain contains a rule with action buy
                             and the product has no Vendor Price
                    = false  otherwise
```

**Worked example.** A product with minimum 10 and maximum 50 in the default warehouse has a rule chain containing the warehouse's one-step reception rule, so the warning is false. Archiving that route empties the chain and the warning becomes true. Adding a route "Vendors to Stock" to the product rebuilds a chain and the warning becomes false again.

---

## 27. The replenishment option lead time

For one inter-warehouse resupply option:

```
rule      = rule selection for (product, the supplying warehouse's stock location)
            with routes = the option's route and warehouse = the supplying warehouse
lead_time = "<total_delay of the lead days of that rule> days"       when a rule is found
          = "0 days"                                                 otherwise
free_to_use_quantity = the unreserved quantity of the product in the supplying warehouse's stock location
```

Options are listed sorted by `free_to_use_quantity` descending.

**Worked example.** Warehouse A is resupplied by warehouse B and by warehouse C, and the reordering rule of product P in warehouse A has a quantity to order of 25 units.

- For the option that uses the route "A: Supply Product from B": rule selection for P at B's stock location, with that route and warehouse B, finds the rule "B: Stock to Transit", whose `lead_time_days` is 2. The chain stops at that rule because it takes from stock, so the lead days of the chain give `total_delay` = 2 and the option shows `lead_time` = "2 days". B's stock location holds 18 unreserved units, so `free_to_use_quantity` is 18. Because 18 is lower than 25, `warning_message` reads `B can only provide 18.0 Units, while the quantity to order is 25.0 Units.`
- For the option that uses the route "A: Supply Product from C": the matching rule "C: Stock to Transit" has `lead_time_days` = 5, so `lead_time` is "5 days", and C's stock location holds 40 unreserved units, so `free_to_use_quantity` is 40 and `warning_message` is empty.
- The two options are listed C first (40 free units) and B second (18 free units). Pressing "Order available quantity" on the B option would set the rule's `quantity_to_order` to 18; pressing "Order all" on the C option would leave it at 25.
- Had no rule been found at all for P at B's stock location, the B option would show `lead_time` = "0 days" instead of being hidden.

---

## 28. The supply method of a generated resupply rule

For each pull rule generated inside an inter-warehouse resupply route:

```
procure_method = make_to_stock   when the rule's source location is the supplying warehouse's stock location
               = make_to_order   otherwise
```

**Worked example.** The supplying warehouse delivers in two steps. Its resupply route holds three rules: "stock to Output" (source is the stock location, so `make_to_stock`), "Output to Transit" (source is the Output location, so `make_to_order`) and "Transit to supplied stock" (source is the transit location, so `make_to_order`). The chain therefore pulls all the way back to the supplying warehouse's stock.

---

## 29. Warehouse-generated rule values

For a list of routings (source location, destination location, operation type, action) and a set of shared values:

```
for each routing, following the given order:
    name           = "<warehouse code>: <source name> → <destination name>" plus " (<suffix>)" when a suffix is given
    location_source, destination_location, action, operation_type from the routing
    auto           = manual
    procure_method = make_to_stock for the first routing of the list, make_to_order for every later one
    warehouse      = the warehouse
    company        = the warehouse company
    then overwritten by the shared values
when the shared values requested propagate_cancel and the list is not empty:
    the last rule of the list has propagate_cancel forced to false
```

Route names are formatted as `<warehouse name>: <route label>`.

**Worked example.** A warehouse whose code is `WH` and whose name is `Main` is configured to receive in three steps. The routing list is: (Vendors → WH/Stock, Receipt, pull), (WH/Input → WH/Quality Control, Quality Control, push), (WH/Quality Control → WH/Stock, Storage, push). The generated rules are named `WH: Vendors → Stock`, `WH: Input → Quality Control` and `WH: Quality Control → Stock`; their supply methods are `make_to_stock`, `make_to_order` and `make_to_order`; their cancel propagation flags are true, true and false. The route is named `Main: Receive, Quality Control, then Store (3 steps)`.

---

## 30. Effective days to arrival

**Purpose.** Measure, for the purchase analysis view, how long a purchase actually took from the day it was ordered to the day the goods arrived.

**Inputs.** One purchase order line. Its order's `date_order`. The earliest completion date among the order's completed transfers whose destination location is not a vendor location, call it *arrival*. The line's planned date.

**Output.** A number of days with two decimal places, aggregated as a plain average over the rows of a group.

**Steps.**

1. Let *reference* be *arrival* when the order has at least one such completed transfer, and the line's planned date otherwise. An order that has not been received yet therefore reports the number of days it is still expected to take, not an empty value.
2. Compute the elapsed time between `date_order` and *reference* as a whole number of seconds.
3. Divide by 86400, the number of seconds in a day, and round to two decimal places.

```
reference       = earliest completion date among the order's completed transfers
                  whose destination location usage is not "supplier"
                = the line's planned date, when there is no such transfer
days_to_arrival = round( (reference − date_order) in seconds ÷ 86400, 2 )
```

**Worked example 1 (received).** A purchase order is placed on 1 March at 08:00. Its receipt is validated on 6 March at 20:00. The elapsed time is 5 days and 12 hours, that is 475200 seconds. Then `days_to_arrival` = 475200 ÷ 86400 = 5.50 days.

**Worked example 2 (two receipts).** The same order is received in two transfers, the first validated on 6 March at 20:00 and the second on 9 March at 09:00. Only the earliest counts, so both lines of the order still report 5.50 days.

**Worked example 3 (returned receipt).** The buyer then returns part of the goods to the vendor on 12 March. That return transfer has a vendor location as destination, so it is ignored, and `days_to_arrival` stays 5.50 days.

**Worked example 4 (not received yet).** A purchase order is placed on 1 March at 08:00 and its line is planned for 11 March at 08:00. Nothing has been received. Then `days_to_arrival` = 864000 ÷ 86400 = 10.00 days.

**Worked example 5 (averaging).** A group contains three lines whose values are 5.50, 10.00 and 3.00. The reported average is (5.50 + 10.00 + 3.00) ÷ 3 = 6.17 days, rounded to two decimal places. Quantities and amounts play no part in the average.

---

## 31. Finding the purchase order line behind a move

**Purpose.** When goods travel back towards a vendor, or when a push rule sends them to a vendor location, the move that finally leaves the company must be attached to the purchase order line that originally brought the goods in, and must carry the vendor as counterparty. The move itself has no such link, because it was created several steps later in the chain.

**Inputs.** One stock move.

**Outputs.** A purchase order line, or nothing; and a contact, or nothing.

**Steps.** This is a breadth-first walk backwards along the chain of origin moves.

1. Put the starting move into a queue. Let *seen* be the empty set.
2. While the queue is not empty:
   1. Take the first move out of the queue, call it *current*.
   2. When *current* carries a purchase order line, return that line together with the counterparty of the transfer that carries *current*, and stop.
   3. Add *current* to *seen*.
   4. Append to the end of the queue every origin move of *current* that is neither already in the queue nor in *seen*.
3. When the queue empties without a match, return nothing for both outputs.

The walk is breadth-first, not depth-first, so that the nearest purchase order line in the chain wins when a move is fed by several branches.

**Worked example.** A warehouse receives in three steps, so a purchase of 10 units produces three moves: Vendors to Input (this move carries the purchase order line), Input to Quality Control, Quality Control to Stock. The buyer returns 2 units, which creates a return move from Stock to an internal "Vendor returns processing" location; a push rule then creates a second move from that location to the vendor location. Walking backwards from that last move: the push move has no purchase order line, its origin move is the return move, which has none either; the return move's origin move is "Quality Control to Stock", which has none; then "Input to Quality Control", which has none; then "Vendors to Input", which carries the line. The walk returns that line and the vendor of its receipt, so the move that leaves for the vendor is stamped with both. The received quantity of the line falls from 10 to 8.
