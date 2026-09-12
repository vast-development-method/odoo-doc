# Calculations

Every formula and algorithm of the repair and maintenance domain: its inputs, its output, its precision, its order of operations, its tie-breaking, and at least one worked example carried to the last decimal the rule produces.

Two conventions hold throughout.

- **Truncation toward zero.** A value assigned to a whole-number field is truncated toward zero, never rounded to nearest. A computed value of 53.99 stored in a whole-number field becomes 53. This matters for the effectiveness measurements, which are whole-number fields fed by divisions.
- **Rounding of quantity comparisons.** No quantity comparison is made by exact equality. Each comparison names the rounding it uses; section 33 collects them all in one table.

| Section | Calculation |
|---|---|
| 1 | Repair numbering and the reference format |
| 2 | Part location resolution from the part kind |
| 3 | Parts availability, the component status |
| 4 | The readiness and lateness booleans |
| 5 | The incomplete-parts flag |
| 6 | Unit price of a Sales Order Line created from a part |
| 7 | Ordered quantity of a Sales Order Line from the repair |
| 8 | Ordered and delivered quantities at completion |
| 9 | Kit explosion factor |
| 10 | Consumption links and the traceability tree |
| 11 | Reserve and unreserve button visibility |
| 12 | The scheduled-date category search |
| 13 | Product quantity derived from the return transfer |
| 14 | Availability of the product to repair at confirmation |
| 15 | Owner of the repaired product at completion |
| 16 | Overview counters of a repair operation type |
| 17 | Colour index of a new Repair Tag |
| 18 | The population of failures |
| 19 | Latest failure date |
| 20 | Mean time between failures |
| 21 | Mean time to repair |
| 22 | Estimated next failure |
| 23 | Expected mean time between failures |
| 24 | Request counters of a maintainable item |
| 25 | Request counters of an equipment category |
| 26 | Equipment counter and folding of a category |
| 27 | Team dashboard counters |
| 28 | Scheduled end of a maintenance request |
| 29 | Duration of a maintenance request |
| 30 | Recurrence of a preventive maintenance request |
| 31 | Activity deadline and assignee |
| 32 | Projected occurrences of a recurring request on the calendar |
| 33 | Rounding and comparison reference |

---

# Part one: repair calculations

## Calculation 1: Repair numbering and the reference format

**Inputs.** The numbering sequence attached to the repair's operation type: its prefix, built from the warehouse short code, a forward slash, the operation type's sequence code and a further forward slash; and its next number, padded to five digits.

```formula
repair reference = warehouse short code + "/" + sequence code + "/" + next number padded to five digits
```

The sequence code falls back to the two letters `RO` when the operation type carries none.

**Worked example.** Warehouse short code `WH`, sequence code `RO`. The first three repairs of that operation type are `WH/RO/00001`, `WH/RO/00002` and `WH/RO/00003`.

**Worked example of renumbering.** A repair created under an operation type whose sequence prefix is `PT1/` is numbered `PT1/00001`. Switching it to an operation type whose prefix is `PT2/` renumbers it `PT2/00001`. Switching it back to the first type draws the **next** number of that sequence, `PT1/00002`; the original `PT1/00001` is never reused. Saving the same operation type again with no change leaves the reference untouched, because the renumbering rule requires the new operation type to differ from the current one.

---

## Calculation 2: Part location resolution from the part kind

**Inputs.** The part's kind and the repair's six location fields.

| Part kind | Source location of the part movement | Destination location of the part movement |
|---|---|---|
| `add` | the repair's component source location | the repair's added-parts destination location |
| `remove` | the repair's added-parts destination location | the repair's removed-parts destination location |
| `recycle` | the repair's added-parts destination location | the repair's recycled-parts destination location |

The repair's own locations default from the operation type as follows.

| Repair location | Operation type default it is taken from | Overridable on the order |
|---|---|---|
| Component source location | default source location | yes |
| Added-parts destination location | default destination location | no, it is a permanent mirror |
| Removed-parts destination location | default remove destination location | no, it is a permanent mirror |
| Recycled-parts destination location | default recycle destination location | yes |
| Product source location | default product source location | yes |
| Product destination location | default product destination location | yes |

And the shipped defaults of a repair operation type belonging to a warehouse are:

| Operation type default | Shipped value |
|---|---|
| Default source location | the warehouse stock location |
| Default destination location | the company's production location |
| Default remove destination location | the company's inventory-loss location |
| Default recycle destination location | the warehouse stock location |
| Default product source location | the warehouse stock location |
| Default product destination location | the warehouse stock location |

### Worked example: a repair with two parts added and one removed

An espresso machine with the serial number `ESP-0042`, tracked by unique serial number, is returned by the customer Wood Corner and repaired under the repair operation type of warehouse `WH` with its shipped defaults. The repair's reference is `WH/RO/00007`. The technician records:

| Part line | Part kind | Product | Demand | Recorded |
|---|---|---|---|---|
| 1 | `add` | Group gasket | 2 | 2 |
| 2 | `add` | Water pump | 1 | 1 |
| 3 | `remove` | Water pump, the failed one | 1 | 1 |

The repair carries the component source location `WH/Stock`, the added-parts destination location `Virtual Locations/Production`, the removed-parts destination location `Virtual Locations/Inventory adjustment`, the recycled-parts destination location `WH/Stock`, the product source location `WH/Stock` and the product destination location `WH/Stock`.

The movements created while the repair is planned and confirmed are:

| Movement | Product | Quantity | From | To | Part kind |
|---|---|---|---|---|---|
| One | Group gasket | 2 | `WH/Stock` | `Virtual Locations/Production` | `add` |
| Two | Water pump | 1 | `WH/Stock` | `Virtual Locations/Production` | `add` |
| Three | Water pump | 1 | `Virtual Locations/Production` | `Virtual Locations/Inventory adjustment` | `remove` |

Every one of them carries the origin `WH/RO/00007`, the document reference `WH/RO/00007`, the repair's operation type, the repair's Stock Reference, and no transfer.

Ending the repair adds a fourth movement, which carries **no** part kind and therefore does not appear in the Parts list:

| Movement | Product | Quantity | From | To | Part kind |
|---|---|---|---|---|---|
| Four | Espresso machine | 1 | `WH/Stock` | `WH/Stock` | none |

Movement four carries exactly one detail line, for the serial number `ESP-0042`, whose consumed-lines list holds the detail lines of movements one, two and three.

Net physical effect:

| Location or item | Change |
|---|---|
| `WH/Stock` | −2 group gaskets, −1 water pump |
| `Virtual Locations/Production` | +2 group gaskets, +1 water pump, −1 water pump, a net +2 gaskets |
| `Virtual Locations/Inventory adjustment` | +1 water pump, the failed one, written off |
| Espresso machine `ESP-0042` | unchanged in quantity, moved from `WH/Stock` to `WH/Stock` |

**Variation, recycling instead of removing.** Had line three been of kind `recycle`, movement three would have run from `Virtual Locations/Production` to `WH/Stock`, putting the removed pump back into sellable stock instead of writing it off.

---

## Calculation 3: Parts availability, the component status

**Inputs.** For each part movement of the repair: its product, its forecast availability, its own quantity expressed in the product's reference unit, and its forecast expected date. Plus the repair's state and scheduled date.

**Preliminary rule on the forecast inputs.** For a part whose kind is `remove` or `recycle`, the forecast availability is set equal to the movement's own quantity and the forecast expected date is cleared, because such a part is taken out of the repaired product and not out of stock. Only `add` parts are actually forecast against stock.

**Algorithm.**

1. When the repair's state is neither `confirmed` nor `under_repair`, both the component status text and the component status code are empty. Stop.
2. Provisionally set the code to `available` and the text to "Available".
3. When any part movement's forecast availability is strictly lower than its own quantity, compared at the rounding of the product's reference unit, set the text to "Not Available" and the code to `late`. Stop.
4. Let the *forecast date* be the greatest forecast expected date among the part movements that carry one. When no part carries one, stop, leaving "Available" and `available`.
5. Set the text to "Exp " followed by the forecast date, formatted in the reader's language and date format.
6. When the repair carries a scheduled date, set the code to `late` when the forecast date is strictly later than the scheduled date, and to `expected` otherwise. When the repair carries no scheduled date, the code stays `available`. A repair always carries a scheduled date in practice, because the field is required.

**Worked example A, everything on hand.** A confirmed repair has two `add` parts, each demanding 2 units, each fully reserved. Their forecast availability equals their quantity and neither carries a forecast expected date. Step 3 does not fire, step 4 finds no date. Result: text "Available", code `available`, readiness boolean true, lateness boolean false.

**Worked example B, a shortage.** The same repair, but the second part demands 5 units of which the forecast can cover only 3. Step 3 compares 3.00 against 5.00 at two decimal places and finds it lower. Result: text "Not Available", code `late`, readiness boolean false, lateness boolean true.

**Worked example C, on the way but late.** The same repair, both parts covered by incoming receipts, the later of which is expected on 4 October 2025. The repair is scheduled for 1 October 2025. Step 3 does not fire; step 4 gives the forecast date 4 October 2025; step 6 compares it with 1 October 2025 and finds it later. Result: text "Exp 10/04/2025" in a month-first date format, code `late`, lateness boolean true.

**Worked example D, on the way and in time.** The same, but the repair is scheduled for 10 October 2025. Step 6 finds 4 October 2025 not later than 10 October 2025. Result: text "Exp 10/04/2025", code `expected`; both booleans false, so the repair is counted neither as ready nor as late on the operation-type overview.

---

## Calculation 4: The readiness and lateness booleans

**Inputs.** The component status code of each repair in the computed set.

**Algorithm.** Both booleans are set to false for every record first. Then, for each record whose code is non-empty: when the code is `available`, the readiness boolean becomes true; when the code is `late`, the lateness boolean becomes true.

```formula
readiness boolean = true exactly when component status code = available
lateness boolean  = true exactly when component status code = late
```

A record whose code is `expected` therefore has both booleans false, and so does a record whose code is empty. Both are stored so that the operation-type overview can count and group without recomputing availability.

---

## Calculation 5: The incomplete-parts flag

**Inputs.** For each part movement: its recorded quantity, its demanded quantity and its unit of measure.

**Rule.** The flag is true when at least one part movement has a unit and its recorded quantity compares as strictly lower than its demanded quantity at the rounding of that unit.

**Worked examples.**

| Part demands | Part recorded | Comparison at the unit's rounding | Contributes to the flag |
|---|---|---|---|
| 3.0 | 3.0 | equal | no |
| 3.0 | 1.0 | lower | yes |
| 4.0 | 2.0 | lower | yes |
| 4.0 | 5.0 | higher | no |
| 0.0 | 0.0 | equal | no |

A repair with parts demanding 3 and 4 units where the technician recorded 1 and 2 has the flag set; recording 3 and 4 clears it; recording 3 and 5 leaves it clear, because an excess does not count.

**Worked example at a coarse rounding.** A part movement whose unit is Units with a rounding of 1.0 — whole units only — demands 3.0 and records 2.6. The comparison at that rounding makes the two equal, so the flag is **not** set and ending the repair asks no partial-quantity question.

---

## Calculation 6: Unit price of a Sales Order Line created from a part

**Inputs.** The repair's warranty flag, the part movement's own unit price, and the price the price list computes for the product, the customer and the quantity.

**Rule at line creation.**

| Situation | Unit price written on the line |
|---|---|
| The repair is under warranty | 0 |
| The repair is not under warranty and the movement carries a non-zero unit price | the movement's unit price |
| The repair is not under warranty and the movement carries none | the price the price list computes |

**Rule when the flag is toggled afterwards.**

| Action | Effect on every Sales Order Line generated from an `add` part of this repair |
|---|---|
| Ticking the warranty flag | the unit price and the stored manual-price marker are both written to 0 |
| Unticking the warranty flag | the unit price is recomputed from the price list, the customer and the quantity |

**Worked example.** A conference chair with a list price of 30.00 is added to a repair that is not under warranty. The generated line reads 30.00. Ticking the warranty flag rewrites it to 0.00. Unticking it recomputes 30.00. A discount of 15 percent entered by hand on the line is left untouched by both operations, and so is the price of any line that was not created from an `add` part of this repair.

---

## Calculation 7: Ordered quantity of a Sales Order Line from the repair

**Inputs.** The part movements attached to one Sales Order Line, and their part kinds.

```formula
ordered quantity of the line = sum of the demanded quantities of every movement attached to that line
```

when the movements are `add` parts, and

```formula
ordered quantity of the line = 0
```

when the movement's part kind is `remove` or `recycle`, or when the movement was cancelled or deleted.

**Worked example.** A repair holds one `add` part for a conference chair demanding 1 unit; its Sales Order Line reads 1. The technician raises the demand to 3; the line reads 3. A salesperson edits the line to 5; the repair part still reads 3. The technician then changes the demand to 4; the line is rewritten to 4, discarding the salesperson's 5. Quantities flow one way only.

---

## Calculation 8: Ordered and delivered quantities at completion

**Inputs.** The part movement's demanded quantity, its recorded quantity and its state; the repair's state.

| Quantity | Rule |
|---|---|
| Ordered quantity at line creation | the movement's demanded quantity when the repair is not yet `done`; the movement's recorded quantity when the repair is already `done` |
| Delivered quantity at line creation | the movement's recorded quantity when the movement is already `done`; otherwise zero |
| Delivered quantity afterwards | when the line has exactly one movement that belongs to a repair and is `done`, the delivered quantity is that movement's recorded quantity, replacing the ordinary computation |

**Worked example.** A repair service is sold for 1 unit and a conference chair part is added with a demand of 1. The repair is completed with a recorded quantity of 1 on the chair. The service line reads ordered 1, delivered 1, by the completion rule that reports a repair service as delivered. The chair line reads ordered 1, delivered 1, by the rule above. The customer can be invoiced for both.

---

## Calculation 9: Kit explosion factor

**Inputs.** The part movement's demanded quantity and unit, and the kit bill of materials' quantity and unit.

```formula
explosion factor = demanded quantity of the part converted into the unit of the bill of materials
                 ÷ quantity the bill of materials produces
```

The component quantity of each resulting part line is the quantity the explosion returns for that component at that factor. Nested kits are exploded recursively. Components whose product is a service are skipped.

**Worked example.** A kit named "Brew group assembly" produces 1 unit from 2 gaskets and 1 seal. A part line demands 3 units of the kit in the same unit as the bill of materials.

```formula
explosion factor = 3 ÷ 1 = 3
gasket quantity  = 2 × 3 = 6
seal quantity    = 1 × 3 = 3
```

The kit movement is deleted and replaced by two movements: 6 gaskets and 3 seals, each carrying the same repair, the same part kind `add`, the same unit price and the same two locations as the kit movement had, each created in the `draft` state.

---

## Calculation 10: Consumption links and the traceability tree

**Inputs.** The detail lines of every part movement of the repair, and the single detail line of the repaired-product movement created at completion.

**Rule.** The detail line of the repaired-product movement records every detail line of every part movement of the repair as a consumed line. The traceability report then reads those links in both directions: when it finds no preceding line for a detail line of a repair, it uses that line's consumed links; when it finds no following usage, it uses that line's produced links. A detail line whose movement belongs to a repair reports the Repair Order as its source document, with the repair's reference as its label.

**Worked example.** In the repair of calculation 2, the detail line of movement four — the espresso machine with serial number `ESP-0042` — records the detail lines of movements one, two and three as consumed lines. Reading the traceability of `ESP-0042` therefore reaches the two gaskets and the two pumps through one node, and reading the traceability of a gasket reaches the espresso machine through the same node. Three consumed links exist for one produced node.

---

## Calculation 11: Reserve and unreserve button visibility

**Inputs.** The repair's state; the reserved quantity on the detail lines of its part movements; and, for each part movement, the picked flag, the demanded quantity and the move state.

```formula
unreserve is visible = repair state is not draft, not done and not cancel
                       AND at least one detail line of a part movement carries a reserved quantity other than zero

reserve is visible   = repair state is confirmed or under repair
                       AND at least one part movement is not picked
                           AND has a demanded quantity other than zero
                           AND is in the confirmed or partially available move state
```

**Reading of the two tests.** The unreserve button appears when there is something to give back, that is at least one detail line still holding a reservation. The reserve button appears when there is something still to take, that is at least one part that has not been picked, that demands something, and whose movement is not already fully reserved. The two are therefore shown together whenever the repair is partly reserved, which is the normal state of a repair waiting on one missing component.

**Worked example.** A Repair Order carries two `add` parts: part A demands 2.00 units of `Group gasket`, part B demands 5.00 units of `Water pump`. The warehouse holds 2.00 gaskets and 2.00 pumps. The table follows the repair through six moments.

| Moment | Repair state | Part A move state, reserved | Part B move state, reserved | Unreserve visible | Reserve visible |
|---|---|---|---|---|---|
| 1. The repair is still being written. | `draft` | `draft`, 0.00 | `draft`, 0.00 | false, the state is excluded | false, the state is not one of the two |
| 2. It has just been confirmed and nothing is reserved yet. | `confirmed` | `confirmed`, 0.00 | `confirmed`, 0.00 | false, no line holds a reservation | true, both parts qualify |
| 3. Reservation has run: A is complete, B is short by 3.00. | `confirmed` | `assigned`, 2.00 | `partially_available`, 2.00 | true, two lines hold 2.00 each | true, only part B qualifies |
| 4. The missing three pumps arrive and reservation runs again. | `confirmed` | `assigned`, 2.00 | `assigned`, 5.00 | true | false, no part is left in a reservable state |
| 5. The technician unreserves everything. | `confirmed` | `confirmed`, 0.00 | `confirmed`, 0.00 | false | true, both parts qualify again |
| 6. The repair is completed. | `done` | `done`, 0.00 | `done`, 0.00 | false, the state is excluded | false, the state is excluded |

At moment 3 both buttons are shown at once, which is the case the two formulas are written to allow. At moment 4 the unreserve button is still shown even though nothing more can be reserved: the repair is fully reserved and the only remaining choice is to give the goods back.

---

## Calculation 12: The scheduled-date category search

**Inputs.** The current moment and the acting user's time zone.

**Boundaries.**

```formula
start of today     = midnight of the current day in the acting user's time zone, expressed in coordinated universal time
start of yesterday = start of today − 1 day
start of day one   = start of today + 1 day
start of day two   = start of today + 2 days
start of day three = start of today + 3 days
```

**Conditions on the scheduled date.**

| Category | Condition |
|---|---|
| `before` | earlier than the start of yesterday |
| `yesterday` | at or after the start of yesterday and earlier than the start of today |
| `today` | at or after the start of today and earlier than the start of day one |
| `day_1` | at or after the start of day one and earlier than the start of day two |
| `day_2` | at or after the start of day two and earlier than the start of day three |
| `after` | at or after the start of day three |

Searching for several categories at once produces the union of their conditions. The only operator supported is membership in a list of categories; any other operator is rejected.

**Worked example.** A user in a zone two hours ahead of coordinated universal time searches at 14:00 local time on 11 September 2025.

```formula
midnight local     = 11 September 2025 00:00 local
start of today     = 10 September 2025 22:00 in coordinated universal time
start of yesterday = 9 September 2025 22:00
start of day one   = 11 September 2025 22:00
start of day two   = 12 September 2025 22:00
start of day three = 13 September 2025 22:00
```

A repair scheduled at 11 September 2025 06:00 in coordinated universal time falls in `today`, because it is at or after 10 September 2025 22:00 and before 11 September 2025 22:00. A repair scheduled at 11 September 2025 23:00 falls in `day_1`, because in the user's zone that is one o'clock in the morning of 12 September.

---

## Calculation 13: Product quantity derived from the return transfer

**Inputs.** The linked return transfer, the product to repair, the product's tracking mode and the chosen lot.

**Algorithm.**

1. When no return transfer is linked, the product quantity is 1.0.
2. When a return transfer is linked, the tracking mode is `serial` or `lot`, and a lot is chosen: the product quantity is the sum of the recorded quantities of the transfer's detail lines whose product is the repair's product and whose lot is the repair's lot.
3. Otherwise, when a return transfer is linked: the product quantity is the sum of the recorded quantities of the transfer's movements whose product is the repair's product.

The quantity used is the **recorded** quantity of the transfer's movements, not their demand.

**Worked example.** A return transfer brought back three units of a lot-tracked pump: two units of lot `L-001` and one unit of lot `L-002`.

| Choice on the repair | Branch taken | Product quantity |
|---|---|---|
| the product, no lot | 3 | 3.0 |
| the product and lot `L-001` | 2 | 2.0 |
| the product and lot `L-002` | 2 | 1.0 |
| no transfer linked at all | 1 | 1.0 |

---

## Calculation 14: Availability of the product to repair at confirmation

**Inputs.** The product to repair, the repair's product source location, the repair's lot, the repair's customer, the repair's product quantity and the repair's unit of measure.

```formula
owned quantity    = sum of the stock quantity records for the product,
                    at the product source location, for the lot, whose owner is the customer
unowned quantity  = the same sum, restricted to records with no owner
required quantity = product quantity converted from the repair's unit into the product's reference unit
```

The check passes when

```formula
owned quantity ≥ required quantity   OR   unowned quantity ≥ required quantity
```

each comparison made at the *Product Unit* decimal precision. The check is skipped entirely when the repair names no product, or when the product is not a storable good; the repair then confirms directly.

**Worked example.** The *Product Unit* precision is two decimal places. A repair asks for 1 unit of a serial-tracked laptop, serial number `SN-7`, for the customer Wood Corner, from the warehouse stock location. The stock quantity records there hold one unit of `SN-7` with no owner.

```formula
owned quantity    = 0.00
unowned quantity  = 1.00
required quantity = 1.00
0.00 ≥ 1.00 is false; 1.00 ≥ 1.00 is true
```

The second comparison passes and the repair is confirmed directly.

**Worked example of the split-owner trap.** The same laptop, but the stock quantity records hold 0.60 owned by Wood Corner and 0.40 with no owner, and the repair asks for 1.00.

```formula
owned quantity    = 0.60
unowned quantity  = 0.40
required quantity = 1.00
0.60 ≥ 1.00 is false; 0.40 ≥ 1.00 is false
```

Both comparisons fail and the insufficient-quantity dialogue is raised, even though the location physically holds one whole unit. The two buckets are never added together.

**Worked example at the rounding boundary.** The *Product Unit* precision is two decimal places, the repair asks for 1.004 and the available unowned quantity is 1.00. The comparison at two decimal places reads 1.00 against 1.00, which passes, and the repair confirms directly without raising the dialogue.

---

## Calculation 15: Owner of the repaired product at completion

**Inputs.** The product to repair, the repair's **component source** location, the repair's lot, the repair's customer and the repair's product quantity.

```formula
available quantity for the customer = available quantity of the product
                                      at the component source location,
                                      for the lot, owned by the customer,
                                      read with a strict test
```

```formula
owner of the detail line = the customer   when available quantity for the customer ≥ product quantity
owner of the detail line = none           otherwise
```

The comparison is made at the *Product Unit* decimal precision. A strict test means only records naming exactly that location, that lot and that owner are counted, with no aggregation over child locations.

**Worked example.** The *Product Unit* precision is two decimal places, the product is a storable, untracked espresso machine, the customer is Wood Corner, the repair quantity is 1.00, and the component source location and the product source location are both `WH/Stock`.

| What `WH/Stock` holds | Available for the customer | Comparison | Owner recorded |
|---|---|---|---|
| 1.00 recorded as owned by Wood Corner | 1.00 | 1.00 ≥ 1.00 | Wood Corner |
| 1.00 with no owner recorded | 0.00 | 0.00 ≥ 1.00 is false | none, so the goods belong to the company |
| 0.60 owned by Wood Corner and 0.40 with no owner | 0.60 | 0.60 ≥ 1.00 is false | none |

**Note on the location.** The availability is read at the repair's component source location while the movement itself runs from the product source location. Both default to the warehouse stock location, so the two coincide in the shipped configuration. This is recorded as a **compatibility finding** in [business-rules.md](business-rules.md), rule RM-026.

---

## Calculation 16: Overview counters of a repair operation type

**Inputs.** The Repair Orders of the operation type, with their state, their readiness boolean, their lateness boolean and their scheduled date.

```formula
confirmed counter    = count of repairs whose state = confirmed
under repair counter = count of repairs whose state = under repair
ready counter        = count of repairs whose state = confirmed AND readiness boolean = true
late counter         = count of repairs whose state = confirmed
                       AND ( scheduled date < today OR lateness boolean = true )
```

Only operation types whose code is `repair_operation` receive values; every other operation type receives false for all four. A repair that is already under repair is never counted as ready, however available its parts are, and is never counted as late.

**Worked example.** An operation type has three repairs confirmed with all parts available and scheduled next week, one confirmed with a missing part, two under repair, one completed and one cancelled.

```formula
confirmed counter    = 3 + 1 = 4
under repair counter = 2
ready counter        = 3
late counter         = 1
```

The single late repair is the confirmed one with the missing part, counted through its lateness boolean rather than through its scheduled date.

---

## Calculation 17: Colour index of a new Repair Tag

```formula
colour index = a pseudo-random whole number drawn uniformly from the eleven values 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 and 11
```

Zero is never drawn. The value is presentational only; nothing in the domain reads it, so two tags sharing a colour behave identically. A colour supplied explicitly at creation is kept and no draw takes place.

**Worked example.** Creating one hundred tags with no colour supplied produces one hundred indices, every one of them between 1 and 11 inclusive and none of them 0. Creating a tag with the colour index 4 supplied produces a tag carrying 4.

---

# Part two: maintenance calculations

The first six calculations of this part are computed together, in one pass, for any maintainable item. In this domain the only maintainable item is Equipment. They recompute whenever the item's effective date changes, or when any of its Maintenance Requests changes stage, close date or request date.

## Calculation 18: The population of failures

**Inputs.** The Maintenance Requests of the item, with their maintenance kind and the closing flag of their stage.

**Rule.** A request belongs to the population of failures when its maintenance kind is `corrective` **and** the closing flag of its stage is true.

**Notes.**

- Preventive requests never count, however many there are.
- A corrective request that has not yet reached a closing stage never counts.
- Archived requests **do** count, because the filter does not test the archive flag.
- The count of the population is written *number of failures* in the formulas below.

**Worked example.** The equipment `Espresso machine` carries six Maintenance Requests.

| Request | Maintenance kind | Stage | Stage closes | Archived | In the population |
|---|---|---|---|---|---|
| Leaking group head | `corrective` | Repaired | yes | no | yes |
| Pump replaced | `corrective` | Repaired | yes | yes | yes, the archive flag is not tested |
| Grinder jammed | `corrective` | In Progress | no | no | no, the stage does not close |
| Descaling of March | `preventive` | Repaired | yes | no | no, preventive work is never a failure |
| Descaling of June | `preventive` | Repaired | yes | no | no, for the same reason |
| Thermostat drifting | `corrective` | New Request | no | no | no, the stage does not close |

The population therefore holds two requests and the number of failures is 2. Adding a seventh request, corrective and moved to the closing stage "Scrap", would make it 3 the moment that stage change is saved, and every one of the measurements below would be recomputed from the larger population.

---

## Calculation 19: Latest failure date

**Inputs.** The population of failures, each with its request date.

```formula
latest failure date = the greatest request date among the failures
latest failure date = empty   when the population is empty
```

**Tie-breaking.** None is needed; the greatest of equal dates is that date.

**Edge case.** A failure whose request date was forced empty contributes nothing to the maximum but still counts in the number of failures.

**Worked example.** The two failures of calculation 18 carry the request dates 10 February 2025 and 11 April 2025. The four requests outside the population carry the request dates 2 May 2025, 20 July 2025, 1 March 2025 and 1 June 2025, three of them later than either failure.

| Candidate set | Dates considered | Latest failure date |
|---|---|---|
| The two failures | 10 February 2025, 11 April 2025 | 11 April 2025 |
| The same two, plus a third failure dated 11 April 2025 | 10 February 2025, 11 April 2025, 11 April 2025 | 11 April 2025, the tie resolving to the shared date |
| The same two, plus a third failure whose request date was forced empty | 10 February 2025, 11 April 2025, empty | 11 April 2025, while the number of failures rises from 2 to 3 |
| No failure at all | none | empty |

The later dates 2 May 2025, 1 June 2025 and 20 July 2025 never enter the maximum, because those requests are not in the population. The latest failure date therefore stays at 11 April 2025 even though newer requests exist, which is exactly what makes it a measure of failures rather than of activity.

---

## Calculation 20: Mean time between failures

**Inputs.** The item's effective date, the latest failure date, and the number of failures.

```formula
mean time between failures = truncate toward zero of
                             ( ( latest failure date − effective date ) in whole days ÷ number of failures )
mean time between failures = 0   when the latest failure date is empty,
                                 or when the division yields zero
```

**Precision.** The difference is taken in whole days. The quotient is truncated toward zero when stored in the whole-number field.

**Reading of the formula.** It is the average calendar span from the day the asset entered service to the day of its most recent failure, divided by the number of failures observed. It is therefore not the average gap between consecutive failures; it is the service life to date spread over the failures recorded.

### Worked example: three failures

An espresso machine entered service on 1 January 2025. Three corrective Maintenance Requests against it have reached a closing stage:

| Request | Request date | Close date |
|---|---|---|
| Pump leaking | 10 February 2025 | 12 February 2025 |
| Grinder jammed | 11 April 2025 | 14 April 2025 |
| Boiler thermostat | 10 June 2025 | 13 June 2025 |

```formula
number of failures         = 3
latest failure date        = the greatest of 10 February, 11 April and 10 June 2025 = 10 June 2025
span in whole days         = 10 June 2025 − 1 January 2025 = 160 days
raw quotient               = 160 ÷ 3 = 53.3333…
mean time between failures = truncate toward zero of 53.3333… = 53 days
```

The equipment form shows a mean time between failures of 53 days.

**Variation, a fourth failure.** A fourth closed corrective request dated 31 July 2025 gives four failures, a span of 211 days, a raw quotient of 52.75, and a stored value of 52 days. Adding a failure lowered the figure, which is the expected behaviour of this definition: more failures over a similar service life means a shorter average interval.

---

## Calculation 21: Mean time to repair

**Inputs.** The population of failures, each with its request date and its close date; the number of failures.

```formula
repair days of one failure = ( close date − request date ) in whole days,
                             when both dates are present
repair days of one failure = 0,
                             when either date is missing
```

```formula
mean time to repair = truncate toward zero of
                      ( sum of the repair days of every failure ÷ number of failures )
mean time to repair = 0   when the number of failures is zero
```

**Precision.** Each difference is taken in whole days; the sum is a whole number; the quotient is truncated toward zero.

**Edge case.** A request whose close date or request date was forced empty contributes zero to the numerator but still counts in the denominator, which pulls the average down. This is deliberate resilience: the equipment form must open even when a date was forced empty.

### Worked example, continuing the espresso machine

```formula
repair days of Pump leaking      = 12 February 2025 − 10 February 2025 = 2
repair days of Grinder jammed    = 14 April 2025 − 11 April 2025 = 3
repair days of Boiler thermostat = 13 June 2025 − 10 June 2025 = 3
sum                              = 2 + 3 + 3 = 8
raw quotient                     = 8 ÷ 3 = 2.6666…
mean time to repair              = truncate toward zero of 2.6666… = 2 days
```

The equipment form shows a mean time to repair of 2 days.

**Variation, a missing close date.** If the boiler thermostat request has its close date forced empty, the sum becomes 5, the raw quotient 1.6666…, and the stored value 1 day.

---

## Calculation 22: Estimated next failure

**Inputs.** The latest failure date and the mean time between failures.

```formula
estimated next failure = latest failure date + mean time between failures days
estimated next failure = empty   when the mean time between failures is zero
```

When the mean time between failures is zero — whether because there are no failures or because the truncated quotient came out at zero — no estimate is produced. An asset that failed on the day it entered service therefore shows no estimated next failure.

### Worked example, continuing the espresso machine

```formula
estimated next failure = 10 June 2025 + 53 days = 2 August 2025
```

Counting: 20 days to 30 June 2025, 31 more to 31 July 2025, 2 more to 2 August 2025, a total of 53.

---

## Calculation 23: Expected mean time between failures

This value is **not** computed. It is entered by hand on the equipment form, in days, and expresses the operator's expectation. Comparing it with calculation 20 is left to the reader of the form; the system draws no conclusion from the comparison and raises no alert.

**Worked example.** An operator types 60 into the field on the espresso machine of calculation 20, whose measured mean time between failures is 53. The field still reads 60 after every recomputation of the measured value, and nothing derives from the difference of 7 days.

---

## Calculation 24: Request counters of a maintainable item

**Inputs.** The item's Maintenance Requests, each with the closing flag of its stage and its archive flag.

```formula
maintenance count      = count of every request of the item
maintenance open count = count of the requests of the item
                         whose stage does not carry the closing flag
                         AND whose archive flag is false
```

Both are stored, so that they can be searched and grouped.

**Worked example.** An equipment has five requests: two in "New Request", one in "In Progress", one in "Repaired" (a closing stage), and one in "In Progress" that has been archived. Then the maintenance count is 5 and the open count is 3.

---

## Calculation 25: Request counters of an equipment category

**Inputs.** Every request whose equipment belongs to the category, grouped by the archive flag.

```formula
maintenance open count = count of the category's requests whose archive flag is false
maintenance count      = maintenance open count
                       + count of the category's requests whose archive flag is true
```

**Difference from calculation 24.** The category's open counter ignores the stage entirely. A request sitting in a closing stage and not archived counts as open for the category and as closed for the equipment.

**Worked example, the same five requests, all on equipment of one category.** The open count is 4 — every request except the archived one — and the total count is 5.

---

## Calculation 26: Equipment counter and folding of a category

```formula
equipment count = count of the equipment whose category is this category,
                  taken under the ordinary hiding of archived records
folding flag    = true exactly when equipment count = 0
```

Archived equipment is therefore not counted. The folding flag is stored, which is what allows the grouped screen to collapse empty categories without recounting; it is recomputed whenever the set of equipment attached to the category changes. Every category in the computed set is first set to not folded, to break the circular dependency between the flag and the grouped count that reads it.

**Worked example.** Three categories exist.

| Category | Equipment attached | Of which archived | Equipment count | Folding flag | Effect on the grouped screen |
|---|---|---|---|---|---|
| Computers | 3 | 0 | 3 | false | The column is shown open, with its three cards. |
| Printers | 2 | 2 | 0 | true | The column is shown collapsed, even though two records still point at the category. |
| Vehicles | 0 | 0 | 0 | true | The column is shown collapsed. |

**Worked example of a change.** The category "Vehicles" has an equipment count of 0 and a folding flag of true. A user creates the equipment `Van 1` in that category: the count becomes 1 and the stored flag becomes false, so the column opens. The user then moves `Van 1` to "Computers": "Vehicles" returns to 0 and true, and "Computers" goes from 3 to 4 and stays false.

---

## Calculation 27: Team dashboard counters

**Population.** The requests whose team is this team, whose stage does not carry the closing flag, and whose archive flag is false.

```formula
number of requests              = the size of the population
number of requests scheduled    = count of the population whose scheduled date is set
number of requests high         = count of the population whose priority = 3
number of requests blocked      = count of the population whose kanban state = blocked
number of requests unscheduled  = number of requests − number of requests scheduled
```

The counts are obtained from one grouped count over the population, grouped by the year of the scheduled date, the priority and the kanban state.

**Worked example.** A team has seven open, unarchived requests. Five carry a scheduled date, two of those five carry the High priority, and one of the two unscheduled ones is Blocked.

```formula
number of requests             = 7
number of requests scheduled   = 5
number of requests high        = 2
number of requests blocked     = 1
number of requests unscheduled = 7 − 5 = 2
```

---

## Calculation 28: Scheduled end of a maintenance request

**Inputs.** The scheduled date.

```formula
scheduled end = scheduled date + 1 hour   when the scheduled date is set
scheduled end = empty                     when the scheduled date is empty
```

The hour is added to the stored moment, which is held in coordinated universal time, so the addition is exactly sixty minutes and is never disturbed by a daylight-saving change in the reader's time zone.

The value is stored and overridable; a user-entered end survives until the start changes again.

**Worked example, four steps in order.**

| Step | Action | Scheduled date | Scheduled end | Duration |
|---|---|---|---|---|
| 1 | The user sets the planned start to 14 March 2025 09:30. | 14 March 2025 09:30 | 14 March 2025 10:30 | 1.00 |
| 2 | The user lengthens the work by hand, setting the planned end to 14 March 2025 14:00. | 14 March 2025 09:30 | 14 March 2025 14:00 | 4.50 |
| 3 | The user moves the work to 17 March 2025 08:00. The derivation runs again and overwrites the hand-entered end. | 17 March 2025 08:00 | 17 March 2025 09:00 | 1.00 |
| 4 | The user clears the planned start. | empty | empty | 0.00 |

Step 3 is the one that surprises users: moving the start silently discards a longer end that was typed in earlier, and the planned length falls back to one hour. A team that plans long jobs re-enters the end after every move of the start.

---

## Calculation 29: Duration of a maintenance request

**Inputs.** The scheduled date and the scheduled end.

```formula
duration in hours = round to two decimal places of
                    ( ( scheduled end − scheduled date ) in seconds ÷ 3600 )
duration in hours = 0   when either bound is empty
```

The rounding is to the nearest value at two decimal places.

**Worked examples.**

| Start | End | Seconds | Raw hours | Stored duration |
|---|---|---|---|---|
| 3 March 2025 09:00 | 3 March 2025 10:00 | 3600 | 1.0 | 1.00 |
| 3 March 2025 09:00 | 3 March 2025 11:30 | 9000 | 2.5 | 2.50 |
| 3 March 2025 09:00 | 3 March 2025 09:50 | 3000 | 0.8333… | 0.83 |
| 3 March 2025 09:00 | empty | none | none | 0.00 |

---

## Calculation 30: Recurrence of a preventive maintenance request

**Trigger.** A recurrent preventive request is written into a stage whose closing flag is set. The computation runs **before** that write is applied, and therefore reads the request's current values.

**Inputs.** The scheduled date, the duration, the repeat interval, the repeat unit, the repeat kind, the end date, and the current moment.

**Algorithm.**

1. The *base moment* is the request's scheduled date. When the scheduled date is empty, the base moment is the current moment.
2. The *successor start* is the base moment advanced by the repeat interval counted in the repeat unit, where the unit is calendar days, calendar weeks, calendar months or calendar years. Calendar arithmetic is used, so a monthly repetition keeps the day of the month and clamps it to the last day of a shorter month.
3. The *successor end* is the successor start advanced by the request's duration in hours, or by one hour when the duration is zero.
4. Decide whether to create the successor: when the repeat kind is `forever`, create it; when the repeat kind is `until`, create it only when the date part of the successor start is not later than the end date, and otherwise create nothing, ending the series.
5. The successor is a copy of the request with the scheduled date set to the successor start, the scheduled end set to the successor end, and the stage set to the one with the lowest sequence.

```formula
successor start = base moment + ( repeat interval × one repeat unit )
successor end   = successor start + duration hours,   when duration is not zero
successor end   = successor start + 1 hour,           when duration is zero
```

### Worked example: repeating every thirty days

A preventive request named "Descale the boiler" is recurrent, with a repeat interval of 30, a repeat unit of days and a repeat kind of `forever`. It is scheduled to start on 3 March 2025 at 09:00 and to end at 11:30, so its duration is 2.50 hours. A technician drags it into the closing stage "Repaired".

```formula
base moment     = 3 March 2025 09:00
successor start = 3 March 2025 09:00 + 30 days = 2 April 2025 09:00
duration        = 2.50 hours
successor end   = 2 April 2025 09:00 + 2.50 hours = 2 April 2025 11:30
repeat kind     = forever, therefore create
```

The successor is created in the stage "New Request", scheduled 2 April 2025 09:00 to 11:30, with the same subject, equipment, team, technician, priority, instructions and recurrence settings. Its close date is empty because the first stage is not a closing stage. Exactly one scheduled activity is created for it, deadlined on 2 April 2025 in the technician's time zone.

The original request stays in "Repaired" with its close date set to today, its within-stage signal reset to In Progress, and its own activity marked done, therefore carrying no open activity.

Dragging the successor into "Repaired" in its turn produces a third occurrence on 2 May 2025 at 09:00, and the series continues, each occurrence thirty calendar days after the previous occurrence's planned start, never after its actual completion.

### Worked example: a bounded series that ends

The same request, with a repeat kind of `until` and an end date of 1 April 2025.

```formula
successor start = 2 April 2025 09:00
date part       = 2 April 2025
2 April 2025 ≤ 1 April 2025 is false
```

No successor is created. The series is finished.

### Worked example: a monthly repetition crossing a short month

Repeat interval 1, repeat unit months, scheduled date 31 January 2025 08:00, duration 0.

```formula
successor start = 31 January 2025 08:00 + 1 month = 28 February 2025 08:00, clamped to the last day of February
successor end   = 28 February 2025 08:00 + 1 hour = 28 February 2025 09:00, the duration of zero falling back to one hour
```

### Worked example: a request with no schedule

Scheduled date empty, repeat interval 2, repeat unit weeks, the current moment 11 September 2025 14:22.

```formula
base moment     = 11 September 2025 14:22
successor start = 11 September 2025 14:22 + 2 weeks = 25 September 2025 14:22
successor end   = 25 September 2025 15:22, the duration of zero falling back to one hour
```

---

## Calculation 31: Activity deadline and assignee

**Inputs.** The scheduled date, held in coordinated universal time; the acting user's time zone; the request's technician and created-by user.

```formula
activity deadline = the date part of ( scheduled date converted from coordinated universal time
                                       into the acting user's time zone )
```

**Responsible person, in order of preference.**

| Order | Candidate | Used when |
|---|---|---|
| 1 | the request's technician | it is set |
| 2 | the request's created-by user | the technician is not set |
| 3 | the acting user | neither of the first two is set |

### Worked example

A user whose time zone runs thirteen hours ahead of coordinated universal time creates a request scheduled for 11 January 2024 at 08:00 local time. Stored, that is 10 January 2024 at 19:00 in coordinated universal time.

```formula
converted moment  = 10 January 2024 19:00 + 13 hours = 11 January 2024 08:00 local
activity deadline = 11 January 2024
```

The deadline is 11 January, not 10 January. Reading the stored value without converting it would produce the wrong day.

---

## Calculation 32: Projected occurrences of a recurring request on the calendar

**Purpose.** The calendar screen of Maintenance Requests paints, besides the real records, the future occurrences a recurrent request will eventually produce, so that a planner sees the shape of the coming weeks without any record having been created yet. A projected occurrence is a drawing, not a record: nothing is written, nothing is numbered, and the successor record itself is created only later, by calculation 30, when the request reaches a closing stage.

**Inputs.** Per request: the display name, the scheduled date, the scheduled end, the duration, the recurrence flag, the mirrored closing flag of the stage, the archive flag, the repeat interval, the repeat unit, the repeat kind and the end date. Per screen: the displayed range, given by its start and its end.

**Output.** One real event per fetched request, plus zero or more projected events, each with a start moment, an end moment and a label.

**Algorithm.**

1. Fetch the requests that satisfy the screen's filters **and** the single range condition that the scheduled date is not later than the end of the displayed range. There is deliberately no lower bound, so that a request scheduled long before the displayed range still loads and can project into it.
2. Emit one real event per fetched request. It runs from the scheduled date to the scheduled end; when the scheduled end is empty it runs for the duration in hours from the scheduled date. A real event may fall outside the displayed range, in which case the screen simply does not paint it.
3. For each fetched request, when the recurrence flag is false, or the mirrored closing flag is true, or the archive flag is true, produce no projected event and go to the next request. A request that has reached a closing stage, or that has been cancelled, therefore stops projecting immediately.
4. Set the projection limit: the end of the displayed range; and, when the repeat kind is `until`, the earlier of that end and the end of the day of the end date.
5. Set the occurrence length: the duration in hours, or exactly one hour when the duration is zero or empty.
6. Set the running moment to the request's scheduled date and the counter to 1.
7. Advance one step: the running moment becomes the running moment plus the repeat interval counted in the repeat unit, in calendar units, with the same clamping rule as calculation 30 — a day of the month that the target month does not have becomes that month's last day.
8. While the running moment is not later than the projection limit, do the following and then repeat from step 7 with the counter increased by one:
   - when the running moment is strictly after the start of the displayed range, emit a projected event starting at the running moment, ending at the running moment plus the occurrence length, labelled with the display name followed by a space, an opening parenthesis, a plus sign, the current counter and a closing parenthesis;
   - when the running moment is at or before the start of the displayed range, emit nothing, but still count the step.
9. Stop when the running moment is later than the projection limit.

**Two consequences of the step rule.** First, the counter counts steps taken from the request's own scheduled date, not events painted, so the first occurrence visible in a displayed range that starts later than the request may well be labelled with a large number. Second, because each step is taken from the previous computed moment and not from the original scheduled date, a clamped day of the month stays clamped for the rest of the projection; calculation 30, which always steps from the request's own stored date, does not have that property.

**Worked example one, a bounded weekly series inside its own month.** The request "Clean the room" is preventive and recurrent, scheduled 6 March 2025 10:00, duration 2.00, repeat interval 1, repeat unit weeks, repeat kind `until`, end date 27 March 2025. The calendar shows March 2025, so the displayed range runs from 1 March 2025 00:00 to 31 March 2025 23:59:59.

- Step 4: the projection limit is the earlier of 31 March 2025 23:59:59 and 27 March 2025 23:59:59, which is 27 March 2025 23:59:59.
- Step 5: the occurrence length is 2 hours.
- The steps: 13 March 2025 10:00 with counter 1, 20 March 2025 10:00 with counter 2, 27 March 2025 10:00 with counter 3, then 3 April 2025 10:00 which is past the limit, so the loop stops.

The calendar therefore paints four events for that request: the real "Clean the room" on 6 March from 10:00 to 12:00, and the projected "Clean the room (+1)", "Clean the room (+2)" and "Clean the room (+3)" on 13, 20 and 27 March, each from 10:00 to 12:00. Only the first can be dragged.

**Worked example two, a series whose record lies outside the displayed range.** The request "Replace the filters" is preventive and recurrent, scheduled 8 January 2025 08:00, duration 0.00, repeat interval 1, repeat unit months, repeat kind `forever`. The calendar shows April 2025, so the displayed range runs from 1 April 2025 00:00 to 30 April 2025 23:59:59.

- Step 1: the request is fetched, because 8 January 2025 08:00 is not later than 30 April 2025 23:59:59. Had the range condition also carried a lower bound, it would not have been.
- Step 2: the real event on 8 January is outside April and is not painted.
- Step 4: the projection limit is 30 April 2025 23:59:59. Step 5: the occurrence length is 1 hour, because the duration is zero.
- The steps: 8 February 2025 08:00 with counter 1, skipped because it is not after the start of the range; 8 March 2025 08:00 with counter 2, skipped for the same reason; 8 April 2025 08:00 with counter 3, painted; then 8 May 2025 08:00, past the limit, so the loop stops.

April therefore shows exactly one event for that request, the projected "Replace the filters (+3)" from 08:00 to 09:00 on 8 April 2025. The label jumps straight to three because the two earlier steps were counted even though they were not painted.

**Worked example three, sticky clamping at the end of a month.** The request "Calibrate the press" has a scheduled date of 31 January 2025 09:00, repeat interval 1, repeat unit months, repeat kind `forever`, duration 1.00, and the calendar shows the six months from January to June 2025.

| Counter | Step arithmetic | Projected start |
|---|---|---|
| 1 | 31 January 2025 plus one month; February has no thirty-first, so the day is clamped to the twenty-eighth | 28 February 2025 09:00 |
| 2 | 28 February 2025 plus one month, taken from the clamped moment | 28 March 2025 09:00 |
| 3 | 28 March 2025 plus one month | 28 April 2025 09:00 |
| 4 | 28 April 2025 plus one month | 28 May 2025 09:00 |
| 5 | 28 May 2025 plus one month | 28 June 2025 09:00 |

The series never returns to the thirty-first, or even to the thirtieth, because every step is taken from the previous clamped moment. Calculation 30, which creates the real successor record, steps from the request's own stored date instead, so the record it creates for the month after January is dated 28 February 2025 as well; the record created after **that** one is dated 28 March 2025, matching this projection because the successor inherits the clamped date. A planner reading the calendar therefore sees the same day of the month that the created records will carry.

---

# Part three: rounding and comparison reference

## Calculation 33: Where each rounding applies

Every quantity comparison in this domain is performed against a declared rounding, never with exact equality on binary decimals. The table collects them.

| Where | Values compared | Rounding used |
|---|---|---|
| Confirmation, product availability (calculation 14) | owned or unowned quantity against the required quantity | the *Product Unit* decimal precision setting |
| Completion, choosing the owner (calculation 15) | available quantity against the repair quantity | the *Product Unit* decimal precision setting |
| Completion, cancelling empty parts | the part's recorded quantity against zero | the rounding of the part's unit of measure |
| The incomplete-parts flag (calculation 5) | the part's recorded quantity against its demand | the rounding of the part's unit of measure |
| Parts readiness (calculation 3) | forecast availability against the part's quantity | the rounding of the product's reference unit |
| Sales Order Line quantity transitions | the line's ordered quantity against zero | the rounding of the line's unit of measure |
| Cancellation and reopening | the line's ordered quantity against zero | the rounding of the line's unit of measure |

**Worked examples of the boundary cases.**

| Case | Values | Rounding | Outcome |
|---|---|---|---|
| A part in Units rounded to whole units, demanding 3.0, recorded 2.6 | 2.6 against 3.0 | 1.0 | equal, so the incomplete-parts flag is not set and no partial-quantity prompt is shown |
| The same part with a recorded quantity of 0.4 | 0.4 against 0 | 1.0 | equal, so the part is cancelled at completion |
| A part whose product's reference unit rounds to two decimals, forecast availability 1.995, part quantity 2.00 | 2.00 against 2.00 | 0.01 | equal, so the part counts as available rather than short |
| A repair asking for 1.004 with 1.00 available, the *Product Unit* precision at two decimals | 1.00 against 1.00 | 0.01 | equal, so confirmation proceeds with no dialogue |
| A Sales Order Line in whole units carrying 0.4 | 0.4 against 0 | 1.0 | treated as zero, so the bound repair is cancelled |

Date differences in the effectiveness measurements are taken in whole days and never in fractions. Durations are taken in seconds and divided by 3600, then rounded to two decimal places. Calendar arithmetic in the recurrence rule and in the calendar projection uses calendar units, which clamp a day of the month that does not exist in the target month to that month's last day.

---

## Reconciliation notes

1. **Numbering of the calculations.** The two earlier texts numbered their formulas differently and neither covered the whole set: one numbered seventeen sections in a repair-first order, the other twenty-nine in a maintenance-first order. This file uses one sequence of thirty-three, repair first, and every reference in the other files of this folder points at it.
2. **The rounding of the maintenance duration.** One earlier text said the duration is rounded to two decimal places without saying by which method. The rounding is to the nearest value at two decimal places, which is stated in calculation 29 and shown by the 0.8333… example resolving to 0.83.
3. **The equipment count of a category.** One earlier text counted every equipment record pointing at the category; the other excluded archived ones. The grouped count runs under the ordinary hiding of archived records, so archived equipment is excluded. Calculation 26 states that and gives the "Printers" row as the case that distinguishes the two readings.
4. **The location used when choosing the owner of the repaired product.** Only one earlier text noticed that the availability is read at the component source location while the movement runs from the product source location. Calculation 15 records the observed behaviour and points at the compatibility finding in [business-rules.md](business-rules.md).
