# Manufacturing — Calculations

Every formula and algorithm of the manufacturing domain, with the unit each quantity is
expressed in, the rounding rule and precision, the evaluation order, and at least one
worked numeric example.

## Conventions used throughout this file

- **Rounding a quantity at a unit.** `round_at(u, x)` rounds *x* to the nearest multiple of
  the rounding increment of unit *u*, half away from zero. `round_up_at(u, x)` and
  `round_down_at(u, x)` use the upward and downward methods. `round_half_up_at(u, x)` uses
  the half-up method where the code names it explicitly.
- **Comparing at a unit.** `compare_at(u, a, b)` returns −1, 0 or +1 after rounding the
  difference *a − b* at unit *u*. `is_zero_at(u, x)` is true when *x* rounds to zero at
  unit *u*.
- **Converting between units.** `convert(x, from_unit, to_unit)` converts a quantity of the
  same measurement category. Unless "unrounded" is stated, the conversion rounds the result
  at the destination unit. The arithmetic itself is specified in
  [units of measure and packaging](../units-of-measure-and-packaging/calculations.md).
- **Converting a price between units.** `convert_price(p, from_unit, to_unit)` converts a
  price per unit; it is the reciprocal transformation of a quantity conversion.
- **Rounding money.** `round_to_currency(x)` rounds *x* at the rounding increment of the
  company currency.
- **Decimal precision "Product Unit".** The shared decimal precision used for every
  quantity field of the domain. Where an explicit number of decimal places is needed (for
  example to render a quantity into a text), that precision is used.
- The **order unit** is the unit of the Manufacturing Order, the **recipe unit** is the unit
  of the Bill of Materials, the **line unit** is the unit of a recipe line, and the
  **reference unit** is the product's own unit of measure.

---

## 1. Recipe selection

Given a set of products, an optional operation type, an optional company and an optional
recipe kind, at most one recipe is chosen per product.

### 1.1 Algorithm

**Preconditions.** The product set is non-empty after products of type `service` are
removed.

1. Remove from the input every product of type `service`. If nothing remains, return an
   empty mapping.
2. Build the candidate filter:
   - the recipe's specific variant is one of the products, **or** the recipe has no specific
     variant and its product template is one of the products' templates;
   - **and** the recipe is active;
   - **and**, when a company is supplied by argument or by the caller's context, the
     recipe's company is empty or that company;
   - **and**, when an operation type is supplied, the recipe's operation type is empty or
     that operation type;
   - **and**, when a recipe kind is supplied, the recipe's kind equals it.
3. Order candidates by sequence ascending, then specific variant ascending, then identifier
   ascending.
4. If exactly one product was requested, return the single first candidate (or nothing).
5. Otherwise walk the ordered candidates once:
   - a candidate with a specific variant is recorded for that variant when no template-wide
     recipe has been recorded for that variant's template and no recipe has been recorded
     for that variant;
   - a candidate without a specific variant is recorded for its template when no recipe has
     been recorded for that template.
6. For every requested product without a variant-specific recipe whose template has a
   template-wide recipe, assign that recipe.

**Postcondition.** Each product maps to zero or one recipe.

### 1.2 Worked example

Four recipes exist for a product template *Chair* with variants *Chair (Red)* and
*Chair (Blue)*:

| Recipe | Specific variant | Sequence | Kind | Operation type |
|---|---|---|---|---|
| A | — | 10 | normal | — |
| B | Chair (Red) | 10 | normal | — |
| C | — | 5 | phantom | — |
| D | — | 20 | normal | Manufacturing (second plant) |

- Asking for a `normal` recipe for *Chair (Red)* with no operation type: candidates are A,
  B and D. Ordered by sequence then variant then identifier: A (10, no variant), B (10,
  variant), D (20). Because the ordering places the empty variant first at equal sequence,
  A is recorded for the template; B is then skipped for *Chair (Red)* because a
  template-wide recipe already exists for its template. The result for a single-product
  request is therefore the first candidate **A**. When several products are requested the
  same reasoning applies through the walk.
- Asking for a `phantom` recipe for either variant returns **C**.
- Asking for a `normal` recipe with operation type *Manufacturing (second plant)* returns
  **A** as well, because A's empty operation type matches any and sorts first.

---

## 2. Recipe explosion

The explosion turns a recipe and a multiplier into two lists: the recipes visited
(*recipes done*) and the leaf component lines with their quantities (*lines done*). It is
the single algorithm behind component move generation, kit expansion, Work Order
generation, cost roll-up and unbuild.

### 2.1 Inputs

| Input | Meaning |
|---|---|
| the recipe | The recipe to explode. |
| the product | The finished product variant, used for the variant skip rule. |
| the quantity | **The number of times the recipe is needed**, not the number of finished units. The caller converts: for an order it is `convert(order_quantity, order_unit, recipe_unit, unrounded) ÷ recipe_quantity`. |
| the operation type | Optional, used when looking up nested kit recipes. |
| the never-variant values | Optional, used by the skip rule. |

### 2.2 Algorithm

1. Initialise *recipes done* with a single entry for the root recipe, carrying the
   quantity, the product, the original quantity and an empty parent line.
2. Initialise the work list with one entry per component line of the root recipe, each
   carrying *(the line, the product, the quantity, no parent line)*.
3. Pre-resolve the kit recipes of the root's components in one lookup, using the supplied
   operation type or the recipe's own operation type, the recipe's company, and the recipe
   kind `phantom`.
4. While the work list is not empty, take its first entry *(line, product, quantity, parent
   line)* and remove it:
   1. If the line is skipped for *product* under the never-variant values (the rule of
      [entities.md](entities.md) §2.2), continue with the next entry.
   2. Compute `line_quantity = quantity × line.product_qty`. This is expressed in the line's
      unit.
   3. If the line's component has no pre-resolved kit recipe, resolve the pending batch of
      kit recipes now.
   4. If the component **has** a kit recipe *K*:
      - compute the nested multiplier

        ```formula
        nested_quantity = convert( line_quantity ÷ K.product_qty , line_unit , K.product_uom_id , unrounded )
        ```

      - push the component lines of *K* to the **front** of the work list, each carrying
        *(the nested line, the component, nested_quantity, this line)*, so that a kit is
        expanded depth-first immediately;
      - record *K* in *recipes done* with the nested quantity, the current product, the
        root quantity and this line as the parent line.
   5. If the component has **no** kit recipe, it is a leaf:

      ```formula
      leaf_quantity = round_up_at( line_unit , line_quantity )
      ```

      The rounding is deliberately upward: if a little more must be consumed, a whole unit
      of the line's unit is consumed. Record the leaf in *lines done* with that quantity,
      the current product, the root quantity and the parent line.
5. Return *recipes done* and *lines done*.

**Postconditions.** *lines done* contains only lines whose component has no kit recipe.
Every quantity in *lines done* is expressed in its own line's unit and is a whole multiple
of that unit's rounding increment. A component that appears in several branches appears
several times in *lines done*; the callers that need a single figure per product sum them.

**Failure condition.** A recipe whose component graph contains a cycle cannot be saved (the
cycle constraint prevents it), so the loop always terminates.

### 2.3 Worked example — one table from four legs and one top

Recipe *Table*, kind `normal`, quantity 1, recipe unit Units:

| Line | Component | Quantity | Line unit |
|---|---|---|---|
| 1 | Leg | 4 | Units |
| 2 | Table top | 1 | Units |

An order for 1 Table. The multiplier is
`convert(1, Units, Units, unrounded) ÷ 1 = 1`.

Explosion:

- *recipes done* = [ (Table recipe, quantity 1, product Table, original quantity 1, no
  parent) ].
- Line 1: not skipped; `line_quantity = 1 × 4 = 4`; Leg has no kit recipe;
  `leaf_quantity = round_up_at(Units, 4) = 4`. Recorded.
- Line 2: `line_quantity = 1 × 1 = 1`; `leaf_quantity = 1`. Recorded.

Component moves created: 4 Legs and 1 Table top, both from the components location to the
production location, both with `procure_method` of take-from-stock.

Unit factors (see §7.1): the legs move has `4 ÷ max(1 − 0, 1) = 4`; the top move has
`1 ÷ 1 = 1`.

### 2.4 Worked example — a two-level recipe with a made-to-order sub-assembly

Recipes:

| Recipe | Product | Kind | Quantity | Recipe unit | Lines |
|---|---|---|---|---|---|
| *Cabinet* | Cabinet | normal | 1 | Units | 1 × Door assembly, 4 × Screw |
| *Door assembly* | Door assembly | normal | 1 | Units | 1 × Door panel, 2 × Hinge |

The product *Door assembly* carries the replenish-on-order route and the manufacture route;
it is therefore made to order.

Exploding *Cabinet* for a quantity of 1:

- *Door assembly* has **no kit** recipe (its recipe is `normal`), so the explosion does not
  descend into it. It is a leaf of the explosion: `leaf_quantity = 1`.
- *Screw*: `leaf_quantity = 4`.

The Manufacturing Order for *Cabinet* therefore has two component moves: 1 Door assembly
and 4 Screws. Because the door assembly's component move is made to order, confirming the
Cabinet order runs the manufacture rule for *Door assembly*, which creates a **second**
Manufacturing Order for 1 Door assembly, itself exploding into 1 Door panel and 2 Hinges.

Chain effects:

- The child order's finished move has the Cabinet order's component move as its downstream
  move (`move_dest_ids`), so completing the child reserves the parent's component.
- The child order's production group is linked as a **child group** of the Cabinet order's
  group, which is what makes the parent report one generated order and the child report one
  source order.
- The Cabinet order's component move for the door assembly is in state `waiting` until the
  child order is done; the Cabinet order's readiness is therefore `waiting` (Waiting Another
  Operation) rather than `confirmed`.
- The lead time of the Cabinet order includes the manufacturing lead time of *Door assembly*
  (see §11.1).

Exploding the same *Cabinet* recipe when *Door assembly* has a **kit** recipe instead gives
a very different result: the explosion descends, and the Cabinet order has three component
moves — 1 Door panel, 2 Hinges and 4 Screws — and no child order at all.

### 2.5 Worked example — unit conversion inside the explosion

Recipe *Paint batch*, quantity 100, recipe unit Litres, with a component line "Pigment"
of 2.5 Kilograms.

An order for 250 Litres. The multiplier is
`convert(250, Litres, Litres, unrounded) ÷ 100 = 2.5`.

- Pigment: `line_quantity = 2.5 × 2.5 = 6.25` Kilograms;
  `leaf_quantity = round_up_at(Kilograms, 6.25)`. With a Kilogram rounding increment of
  0.01, the result is 6.25 Kilograms.
- If the Kilogram rounding increment were 1, the result would be `round_up_at(1, 6.25) = 7`
  Kilograms — the upward rounding consumes a whole kilogram.

---

## 3. Kit explosion on a Stock Move

A kit product never moves. Wherever a Stock Move names a kit product, that move is replaced
by moves for the kit's leaf components. This happens at confirmation, and again at
completion for any move that escaped confirmation.

### 3.1 Algorithm

For each move in the batch:

1. If the move has no operation type **and** the caller is not scrapping and has not asked
   to skip transfer assignment, keep the move unchanged — a move with no operation type is
   not attached to a transfer and exploding it would scatter it. Also keep unchanged a
   finished move whose product is its order's own product.
2. Look up a `phantom` recipe for the move's product in the move's company. If there is
   none, keep the move unchanged.
3. Compute the multiplier:

   ```formula
   factor = convert( move_demand , move_unit , recipe_unit ) ÷ recipe_quantity
   ```

   except that when the move's demand is zero at the move's unit, the move's **done**
   quantity is used instead of the demand.
4. Explode the recipe with that factor, the move's product and the move's never-variant
   values.
5. For each exploded leaf line produce one or more new move values (§3.2), and mark the
   original move for deletion.
6. Create the new moves, adjust their procurement method, and explode them again (a
   component may itself be a kit).
7. Set the marked moves' done quantity to zero, cancel them and delete them.
8. Return the union of the kept moves and the newly created moves.

### 3.2 Values of a phantom move

Each exploded leaf produces a copy of the original move overridden with:

| Field | Value |
|---|---|
| `picking_id` | The original move's transfer, if any. |
| `product_id` | The leaf component. |
| `product_uom` | The leaf line's unit. |
| `product_uom_qty` | The exploded quantity when the original demand was non-zero, otherwise 0. |
| `quantity` | 0 when the original demand was non-zero, otherwise the exploded quantity (this is the scrap and done-completion path). |
| `picked` | The original move's picked flag. |
| `bom_line_id` | The leaf recipe line. |
| `description_picking` | The original product's display name. |
| `cost_share` | The cost share carried by the exploded line data, or 0. |
| `state` | `assigned` when the original move was `assigned`. |

A leaf whose component is not of type `consu` produces **no** move at all — services in a
kit are documentation only.

### 3.3 Worked example — a kit sold and delivered

A kit product *Desk set* has a `phantom` recipe producing 1 Desk set from:

| Component | Quantity | Unit |
|---|---|---|
| Desk lamp | 1 | Units |
| Pen holder | 2 | Units |
| Notebook | 3 | Units |

A customer orders **5 Desk sets** at 90.00 each.

1. The sales order line names *Desk set*, quantity 5, unit price 90.00, so the untaxed
   amount is 450.00. The sales document is entirely unaffected by the kit: one line, one
   product, one price.
2. Confirming the sale creates one delivery move for 5 Desk sets. At confirmation the move
   is exploded:
   - `factor = convert(5, Units, Units) ÷ 1 = 5`;
   - Desk lamp: `round_up_at(Units, 5 × 1) = 5`;
   - Pen holder: `round_up_at(Units, 5 × 2) = 10`;
   - Notebook: `round_up_at(Units, 5 × 3) = 15`.
   The delivery transfer therefore shows three lines: 5 Desk lamps, 10 Pen holders and
   15 Notebooks. The *Desk set* move no longer exists.
3. Each component move carries the recipe line, so the delivery document groups the three
   lines under the heading *Desk set* and labels them "Desk set - 1/3", "Desk set - 2/3",
   "Desk set - 3/3".
4. Reservation happens on the components. The kit product itself has no quant and no
   reservation.
5. Suppose the warehouse ships 5 lamps, 10 pen holders but only 12 notebooks. The delivered
   quantity of the sales order line is recomputed by the kit-quantity algorithm (§4):
   - lamps: `5 ÷ 1 = 5`;
   - pen holders: `10 ÷ 2 = 5`;
   - notebooks: `12 ÷ 3 = 4`;
   - the minimum is 4, floored to 4 whole kits. **The delivered quantity is 4**, not 5.
6. Invoicing at the delivered-quantities policy therefore bills 4 Desk sets at 90.00, that
   is 360.00 — the price stays on the kit, never on the components.
7. The cost of goods sold, where the anglo-saxon recognition applies, uses the sum of the
   component values: see [accounting-effects.md](accounting-effects.md) §6.
8. Attempting to count the kit product directly in an inventory adjustment is refused:
   **"You should update the components quantity instead of directly updating the quantity of
   the kit product."**
9. Attempting to create a reordering rule for *Desk set* is refused: **"A product with a
   kit-type bill of materials can not have a reordering rule."**

---

## 4. How many kits a set of component moves represents

Used to compute the delivered quantity of a sold kit, the received quantity of a purchased
kit and the scrapped quantity of a scrapped kit.

### 4.1 Algorithm

Inputs: the kit product, the ordered kit quantity, the kit recipe, and two filters naming
which moves count as incoming and which as outgoing.

1. `kit_qty = ordered_kit_quantity ÷ recipe_quantity`.
2. Explode the kit recipe for the kit product with multiplier `kit_qty`.
3. Initialise an empty list of ratios.
4. For each exploded leaf line:
   1. Skip a line whose component is a service.
   2. Skip a line whose exploded quantity is zero at the line's unit (an optional
      component), to avoid dividing by zero.
   3. Select the moves in the batch bound to that recipe line. **If there are none at all,
      return 0 immediately** — the kit cannot be considered delivered when a component has
      no move.
   4. Compute the quantity of the component needed for one kit:

      ```formula
      uom_qty_per_kit = exploded_line_quantity ÷ line_original_quantity
      qty_per_kit = convert( uom_qty_per_kit ÷ recipe_quantity , line_unit , component_reference_unit , unrounded )
      ```

      Skip the line when `qty_per_kit` is zero.
   5. Among the selected moves, keep the incoming ones, remove those that are an origin of
      another selected incoming move (so that in a multi-step flow only the last move of
      each chain counts), and sum their quantities. The quantity of one move is its done
      quantity converted into the component's reference unit with half-up rounding when the
      move is marked picked, and its demand in the reference unit otherwise.
   6. Do the same for the outgoing moves.
   7. `qty_processed = incoming_qty − outgoing_qty`.
   8. Append `round_at( component_reference_unit , qty_processed ÷ qty_per_kit )` to the
      ratios.
5. If no ratio was produced, return 0. Otherwise return the integer part (floor) of the
   minimum ratio — a partial kit is not a kit.

### 4.2 Worked example

Continuing §3.3 with the shipment of 5 lamps, 10 pen holders and 12 notebooks:

- lamps: `qty_per_kit = 1`; `qty_processed = 5`; ratio `5`.
- pen holders: `qty_per_kit = 2`; `qty_processed = 10`; ratio `5`.
- notebooks: `qty_per_kit = 3`; `qty_processed = 12`; ratio `4`.
- minimum = 4; floor = **4 kits delivered**.

If the customer later returns 3 notebooks, the notebook outgoing quantity becomes
`12 − 3 = 9`, the ratio becomes 3, and the delivered quantity drops to 3.

---

## 5. Work Order expected duration

### 5.1 The general formula

Let *capacity*, *setup* and *cleanup* be resolved from the Work Order's work centre for the
Work Order's product and unit, with the recipe quantity (or 1) as the default capacity
(§6). Let *efficiency* be the work centre's time efficiency percentage.

**Case A — the Work Order has an operation.**

```formula
quantity           = qty_producing  when it is non-zero, otherwise qty_production
cycle_number       = round_up_to_integer( quantity ÷ capacity )
duration_expected  = setup + cleanup + cycle_number × operation_time_cycle × 100 ÷ efficiency
```

**Case B — the Work Order has no operation** (a manually added Work Order). The stored
expected duration is rescaled rather than recomputed from a cycle time:

```formula
working_part = max( ( duration_expected − setup − cleanup ) × efficiency ÷ 100 , 0 )
qty_ratio    = qty_producing ÷ ( previous_qty_producing or qty_production )
               when qty_producing is none of 0, qty_production, previous_qty_producing
qty_ratio    = 1 otherwise
duration_expected' = setup + cleanup + working_part × qty_ratio × ratio × 100 ÷ efficiency
```

where *ratio* is an extra multiplier supplied by the caller (1 by default; the change-quantity
assistant passes the new-to-old quantity ratio).

**Case C — an alternative work centre is being evaluated.** First strip the current work
centre's setup, cleanup, efficiency and cycle count out of the stored expected duration,
then re-apply the alternative's:

```formula
working_per_cycle = max( ( duration_expected − setup − cleanup ) × efficiency
                         ÷ ( 100 × cycle_number ) , 0 )
capacity' , setup' , cleanup' = capacity of the alternative work centre
cycle_number'      = round_up_to_integer( quantity ÷ capacity' )
duration_expected' = setup' + cleanup' + cycle_number' × working_per_cycle × 100 ÷ efficiency'
```

When the Work Order has no work centre at all, the expected duration is left unchanged.

### 5.2 When the expected duration is recomputed

Only when the Work Order is neither `done` nor `cancel`, and either the quantity producing
differs from the original production quantity, or (during an interactive edit) the stored
record had a non-zero quantity producing and the edited value differs from it.

It is also recomputed explicitly: when the work centre changes, when the order is split (for
the backorders), when the quantity to produce is changed through the assistant, and when the
inventory is posted (for Work Orders that are not yet done or cancelled).

### 5.3 Worked example — a work order expected at sixty minutes taking seventy-five

An operation *Assemble* has a fixed cycle time of 60 minutes. Its work centre has no setup
time, no cleanup time, an efficiency of 100 percent and a capacity of 1 unit. The order
produces 1 unit.

- `cycle_number = round_up_to_integer(1 ÷ 1) = 1`
- `duration_expected = 0 + 0 + 1 × 60 × 100 ÷ 100 = 60` minutes.

The operator starts the Work Order at 09:00 and finishes at 10:15. A single Productivity
Log is open from 09:00 to 10:15.

1. **Real duration.** The log is not yet closed, so its end is taken as the current instant
   while the timer runs. When the operator finishes, the log is closed at 10:15 and the real
   duration becomes the merged interval length in minutes:
   `duration = 75.00` minutes.
2. **Timer splitting.** Closing the timer sees that 75 > 60. It computes
   `productive_end = 10:15 − (75 − 60) minutes = 10:00`. Because 10:00 is after the log's
   start of 09:00, the log is copied with a start of 10:00 (the over-performance part,
   10:00 → 10:15) and the original is shortened to 09:00 → 10:00. The copy is rewritten
   with the first Productivity Loss Reason of category `performance`, which in the shipped
   data is **Reduced Speed**.
3. **Per-category durations.** The productive category now holds 60 minutes and the
   performance category 15 minutes. Because durations are merged per category and then
   added, the total is again `60 + 15 = 75` minutes.
4. **Deviation.**
   `duration_percent = 100 × (60 − 75) ÷ 60 = −25`, stored as the integer **−25**.
5. **Progress.** While running, `progress = 75 × 100 ÷ 60 = 125` percent; once the state is
   `done`, the progress is forced to **100**.
6. **Duration per unit.** With 1 unit produced,
   `duration_unit = round_to_2_decimals(75 ÷ 1) = 75.00` minutes.
7. **Work centre effectiveness.** For the month, this Work Order contributes 60 productive
   minutes and 15 blocked minutes (the performance category counts as blocked for the
   effectiveness formula). It contributes 60 expected minutes and 75 real minutes to the
   performance formula.
8. **Cost.** With an hourly cost of 48.00 and cost mode `actual`:
   `cost = (75 ÷ 60) × 48.00 = 60.00`. With cost mode `estimated`:
   `cost = (60 ÷ 60) × 48.00 = 48.00`.

### 5.4 Worked example — capacity and setup

The same operation, but the work centre has 12 minutes of setup, 8 minutes of cleanup, an
efficiency of 90 percent and a capacity of 4 units, and the order produces 10 units.

- `cycle_number = round_up_to_integer(10 ÷ 4) = round_up_to_integer(2.5) = 3`
- `duration_expected = 12 + 8 + 3 × 60 × 100 ÷ 90 = 20 + 200 = 220` minutes.

If the same Work Order is evaluated on an alternative work centre with 5 minutes of setup,
5 of cleanup, efficiency 120 percent and capacity 5:

- `working_per_cycle = max((220 − 12 − 8) × 90 ÷ (100 × 3), 0) = max(200 × 90 ÷ 300, 0) = 60`
- `cycle_number' = round_up_to_integer(10 ÷ 5) = 2`
- `duration_expected' = 5 + 5 + 2 × 60 × 100 ÷ 120 = 10 + 100 = 110` minutes.

The alternative finishes sooner, so the planner will choose it if a slot is available.

---

## 6. Work centre capacity resolution

```formula
sort_key(line) = ( not ( line.product = product and line.unit = product_reference_unit ) ,
                   not ( line.product is empty and line.unit = requested_unit ) ,
                   not ( line.product is empty and line.unit = product_reference_unit ) )
```

Lines are sorted ascending on that triple (false before true), and the first is taken.

1. If there is a first line, and its product is either the requested product or empty, and
   its unit is either the product's reference unit or the requested unit:
   - when its capacity is zero at zero decimal places:
     return *(default capacity, line setup, line cleanup)*;
   - otherwise return
     *(convert(line capacity, line unit, requested unit), line setup, line cleanup)*.
2. Otherwise return *(default capacity, work centre setup, work centre cleanup)*.

**Worked example.** A work centre has three capacity lines:

| Line | Product | Unit | Capacity | Setup | Cleanup |
|---|---|---|---|---|---|
| 1 | — | Units | 6 | 10 | 5 |
| 2 | Chair | Units | 4 | 15 | 5 |
| 3 | — | Dozens | 0 | 20 | 0 |

Resolving for product *Chair*, requested unit Units, default capacity 1:

- key of line 2 = (false, true, true) — it matches product and reference unit;
- key of line 1 = (true, false, false);
- key of line 3 = (true, true, true).

Sorted ascending: line 2, line 1, line 3. Line 2 wins, its capacity is non-zero, so the
result is *(4, 15, 5)*.

Resolving for product *Table* (no specific line), requested unit Units, default capacity 1:

- key of line 1 = (true, false, false) wins. Result *(6, 10, 5)*.

Resolving for product *Table*, requested unit Dozens: line 3's key is (true, false, true)
and line 1's is (true, true, false); line 3 sorts first. Its capacity is zero, so the result
is *(1, 20, 0)* — the default capacity with line 3's setup and cleanup.

---

## 7. Distributing the quantity being produced

### 7.1 Unit factor

```formula
unit_factor = move_demand_in_move_unit ÷ max( order_quantity − order_produced_quantity , 1 )
```

- The order is the component order for a component move and the finished order for a
  finished move; a move belonging to neither has a unit factor of 1.
- The denominator is taken as 1 whenever the difference is zero — never a division by zero.
- Because the denominator shrinks as production proceeds, the unit factor of the surviving
  demand rises, which keeps the distribution correct for the remaining units.

### 7.2 The distribution algorithm

Triggered whenever the quantity producing or the producing lots change, and whenever the
quantities are set automatically before closing.

1. If the product is serial-tracked:
   ```formula
   producing_in_reference_unit = round_half_up( convert( qty_producing , order_unit , reference_unit ) )
   to_produce_in_reference_unit = round_half_up( convert( product_qty , order_unit , reference_unit ) )
   ```
   If the two differ, and it is not the case that the producing quantity is zero while the
   stored record had a different producing quantity (which is how the mass-produce flow
   clears the field), then set

   ```formula
   qty_producing = round_half_up( convert( number_of_producing_lots , reference_unit , order_unit ) )
   ```

2. For every component move, and for every finished move whose product is not the order's
   product **or** whose product is serial-tracked:
   1. Skip a by-product move or a manual-consumption move that is already marked picked —
      a produced by-product is never rewritten.
   2. Skip a move that should be bypassed: one in state `done` or `cancel`, or one whose
      demand is zero at its unit (extra products are not rewritten).
   3. Compute

      ```formula
      new_quantity = round_at( move_unit , ( qty_producing − qty_produced ) × unit_factor )
      ```

   4. If the move's product is tracked, cap the new quantity by what upstream has actually
      delivered:
      - let *relevant origins* be the origin moves that are neither `draft` nor `cancel`;
      - let *available* be the sum, converted into the move's unit, of the done quantities of
        the origin moves in state `done`;
      - let *taken* be the sum, converted into the move's unit, of the done quantities of the
        sibling moves (the other destination moves of those origins) in state `done`;
      - when there is at least one relevant origin and `available ≥ taken` at the move's
        unit, set `new_quantity = min(new_quantity, available − taken)`.
   5. Write the done quantity of the move to *new_quantity* (which redistributes the move
      lines).
   6. Mark the move picked when it is not a by-product move, its done quantity is non-zero,
      it is either a component move or a move of a non-serial product, and either the move is
      not a manual-consumption move or the caller asked to pick manual-consumption moves.

**Note on the caller flag.** The interactive path (changing the quantity producing on the
form) calls the algorithm with "do not pick manual-consumption moves"; the explicit
"set quantities" action and the automatic pre-close path call it with the same flag, so
manual-consumption moves are picked separately, just before the consumption check.

### 7.3 Worked example — producing ten with a partial completion of six and a backorder

Recipe *Table*: 1 Table from 4 Legs and 1 Table top. A Manufacturing Order is created for
**10 Tables**, confirmed, and all components reserved.

**State after confirmation**

| Move | Product | Demand | Unit factor |
|---|---|---|---|
| Component | Leg | 40 | `40 ÷ max(10 − 0, 1) = 4` |
| Component | Table top | 10 | `10 ÷ 10 = 1` |
| Finished | Table | 10 | `10 ÷ 10 = 1` |

**Step 1 — the operator declares 6 produced.** Setting the quantity producing to 6 runs the
distribution:

- Legs: `round_at(Units, (6 − 0) × 4) = 24`, marked picked.
- Table top: `round_at(Units, (6 − 0) × 1) = 6`, marked picked.
- The finished move is **not** touched here (it is the order's own product and not
  serial-tracked); its quantity is set during the completion.

The order state becomes `progress` (a component move is picked and the quantity producing is
non-zero). It does **not** become `to_close`, because 6 is less than 10.

**Step 2 — mark as done.**

1. The sanity checks run (company consistency, serial-number uniqueness).
2. The automatic-production check decides whether to auto-fill. With no tracking anywhere,
   the check passes, so the quantities are set automatically — but the quantity producing is
   already 6, so nothing changes.
3. The consumption check compares consumed against expected. The expected quantity per
   product is

   ```formula
   expected = ( exploded_quantity_in_reference_unit ) × qty_producing ÷ product_qty
   ```

   - Legs: exploded demand 40, `expected = 40 × 6 ÷ 10 = 24`; consumed 24 → no issue.
   - Table top: `expected = 10 × 6 ÷ 10 = 6`; consumed 6 → no issue.

   No consumption warning is raised, whatever the policy.
4. The backorder check computes `quantity_to_backorder = max(10 − 6, 0) = 4`, which is not
   zero, so a quantity issue exists. The operation type's backorder policy decides:
   - `always` → the backorder is created without asking;
   - `ask` → the Backorder Confirmation assistant opens; the answer decides;
   - `never` → no backorder; the order closes at 6 and the remaining 4 are dropped.
5. Assume the backorder is created. The split algorithm (§8) runs with the amounts
   `[6, 4]`.

**Step 3 — the split.**

- The original order is renamed. Its backorder sequence was 0, so it becomes 1 and the name
  gains the suffix `-001`: an order named `WH/MO/00021` becomes `WH/MO/00021-001`.
- The original order's quantity to produce becomes 6.
- One backorder is created with quantity 4, backorder sequence 2, name `WH/MO/00021-002`,
  state `confirmed`, the same production group, the same references, the same deadline, the
  same origin, no producing lots.
- Every non-additional move is split proportionally. For the legs move the
  per-unit factor is `initial_demand ÷ initial_order_quantity = 40 ÷ 10 = 4`; the original
  move's demand becomes `6 × 4 = 24` and the backorder's `4 × 4 = 16`. For the top move the
  factor is 1, giving 6 and 4. For the finished move the factor is 1, giving 6 and 4.
- The reserved move lines are redistributed: the original move keeps as much as its new
  demand allows, the remainder moves to the backorder's move. Moves fully covered are set to
  `assigned`, partially covered ones to `partially_available`. Move lines left with zero
  quantity are detached and deleted.
- The backorder's Work Orders (if any) get their expected duration recomputed for the new
  quantity, and their carried quantity set (§8.4).

**Step 4 — posting.**

- The original order posts its inventory: the two component moves (24 Legs, 6 Table tops)
  are posted `done`; the finished move is given
  `round_half_up_at(order_unit, (6 − 0) × 1) = 6` and posted `done`.
- The production cost is computed and the finished move's unit price set (§9).
- The original order's finish date becomes the current instant, its priority is reset, it is
  locked, and its state becomes `done`.
- Its remaining moves with no quantity — there are none in this example — would be written
  to `done` with a demand of zero.
- The backorder is left `confirmed`, with 4 Tables to produce, 16 Legs and 4 Table tops
  demanded, and it is re-reserved if its operation type reserves at confirmation.

**Invariant check.** `6 + 4 = 10`: the sum of the quantities to produce over the production
group equals the original quantity.

---

## 8. Splitting an order

### 8.1 Determining the amounts

The caller may supply, per order, a list of amounts; the first is the quantity kept by the
original order and the rest become backorders. When no list is supplied the default is
`[qty_producing, max(product_qty − qty_producing, 0)]`.

For a supplied list, let *total* be its sum and let
`diff = compare_at(order_unit, product_qty, total)`:

- `diff > 0` and the caller did not ask to cancel the remainder → append
  `product_qty − total` as one more amount, and remember that this last backorder is "to be
  ignored" for the purpose of pre-filling consumed quantities;
- `diff < 0`, or the order is `done` or `cancel`, and the caller did not allow more →
  fail with **"Unable to split with more than the quantity to produce."**

### 8.2 Naming the backorders

```formula
suffix(sequence) = "-" + zero_pad( sequence , 3 )
```

More precisely, the suffix is a hyphen followed by
`SIZE − 1 − floor(log10(sequence))` zeros and then the sequence, where `SIZE` is 3. So
sequence 1 gives `-001`, sequence 12 gives `-012`, sequence 123 gives `-123`, and sequence
1234 gives `-1234` (the zero count becomes negative and no padding is added).

A name that already ends in a hyphen followed by digits has that suffix **replaced** rather
than appended, but only when the maximum backorder sequence already present in the
production group is greater than 1, or the sequence being applied is greater than 1.
Otherwise the suffix is appended.

A sequence of 0 leaves the name unchanged.

**Worked example.** `WH/MO/00021` with backorder sequence 1 → `WH/MO/00021-001` (appended,
because the name does not end in a hyphen and digits). Then the next backorder, sequence 2,
is named from the **already renamed** original: `WH/MO/00021-001` ends in `-001`, and the
sequence 2 is greater than 1, so the suffix is replaced, giving `WH/MO/00021-002`.

### 8.3 Splitting the moves

For every non-additional component and finished move of an order being split:

```formula
per_unit = move_demand ÷ initial_order_quantity
new_demand_of_original = original_new_quantity × per_unit
new_demand_of_backorder_k = backorder_k_quantity × per_unit
```

The original move is rewritten (with unreservation and procurement suppressed) and one new
move is created per backorder, carrying:

| Field | Value |
|---|---|
| `state` | `draft` when the source move was draft, otherwise `confirmed`. |
| `reservation_date`, `date_deadline`, `procure_method` | Copied from the source move. |
| `manual_consumption` | Recomputed from the recipe line. |
| `move_orig_ids`, `move_dest_ids` | Linked to the same upstream and downstream moves as the source. |
| `raw_material_production_id` or `production_id` | The backorder. |

Before the split, every move line that is not itself picked but belongs to a picked,
non-additional move is deleted, so that unregistered reservations do not follow the split.

### 8.4 Redistributing the reservations

Rather than unreserving and re-reserving (which could let another order take the stock, or
change the first-in-first-out selection), the reserved quantities are moved by hand:

1. For each original move, build the list of its move lines with their available quantity
   converted into the product's reference unit with half-up rounding, skipping non-positive
   ones. (Only when the move is **not** picked; a picked move's lines are left alone.)
2. Walk the target moves in order — the original move first, then its backorder moves.
   Maintain `move_qty_to_reserve` initialised to the current target's demand in the reference
   unit.
3. First pass: for each existing move line in order, take
   `taken = min(line_available, move_qty_to_reserve)`, convert it into the line's unit with
   half-up rounding, and if it is not zero at that unit, rewrite the line with that quantity
   and move it to the current target. Decrease `move_qty_to_reserve`; when it reaches zero
   or below, mark the target as `assigned` and advance to the next target.
4. Second pass: for the remainder of each line, while a target remains, take
   `taken = min(move_qty_to_reserve, remaining)`; if the target is the original move, add
   the converted amount to the existing line; otherwise queue a new move line for the target
   with that quantity. Advance targets the same way.
5. When a target is left with `move_qty_to_reserve` different from its full demand, mark it
   `partially_available`.
6. Delete the original move lines left at zero quantity (detaching them from their move
   first, to avoid a needless state recomputation).
7. Reserve the newly created backorder moves that are `confirmed` or `partially_available`
   and that either bypass reservation, or belong to an operation type reserving at
   confirmation, or have a reservation date on or before today.

When the caller asked for consumed quantities to be set (the serial-number split path), a
move line is additionally created for every component and by-product move of every target
except the trailing "to be ignored" backorder, with the quantity equal to the target's
demand, and all those moves are marked picked.

### 8.5 Redistributing the Work Order progress

1. For each backorder Work Order, recompute the expected duration for the backorder's
   quantity.
2. For the original order's Work Orders, in identifier order, record
   `remaining_k = max(initial_order_quantity − carried_k − produced_k, 0)` and then clamp
   the produced quantity to the new production quantity (unless this order is one that the
   caller flagged as being backordered).
3. For each backorder Work Order at index *i*, with *L* the number of Work Orders per order:
   ```formula
   carried = max( backorder_production_quantity − remaining_{i mod L} , 0 )
   ```
   and then, when `remaining_{i mod L}` was non-zero, decrease it by the backorder Work
   Order's produced quantity (bounded below by zero); when it was zero, the backorder Work
   Order is cancelled instead.
4. Cancel the collected Work Orders, then confirm the backorders' Work Orders (which
   re-links their dependency chain).

### 8.6 Splitting by batch size

The split assistant offers a maximum batch size:

```formula
max_batch_size = recipe_batch_size   when the recipe enables batch sizing
max_batch_size = order_quantity      otherwise
num_splits     = round_up_to_integer( order_quantity ÷ max_batch_size )   when max_batch_size > 0
num_splits     = 0                                                        otherwise
```

The detail lines are then `num_splits` lines, each of
`min(max_batch_size, remaining)` with the remainder decreased and rounded at the order unit
after each line.

**Worked example.** An order for 17 units with a recipe batch size of 5:
`num_splits = round_up_to_integer(17 ÷ 5) = 4`; the lines are 5, 5, 5 and 2. The validity
check confirms `5 + 5 + 5 + 2 = 17`.

---

## 9. Production cost

### 9.1 The consumption check

Run when the order is marked done, unless the caller skips it. For each order whose policy
is not `flexible` and that has a recipe with at least one component line:

1. Recompute the expected component values from the recipe (the same generation as at draft
   time). For each of them:

   ```formula
   expected_qty_in_reference_unit[product] +=
       convert( expected_demand , expected_unit , product_reference_unit )
       × qty_producing ÷ product_qty
   ```

2. For each component move:

   ```formula
   consumed = convert( picked_quantity_of_the_move , move_unit , product_reference_unit )
   ```

   - If the move's product is not in the expected map **and** the move is picked **and** the
     consumed quantity is not zero at the reference unit, record an issue
     *(order, product, consumed, 0)* — an extra component was consumed.
   - Otherwise add `consumed` (or 0 when the move is not picked) to the consumed map.
3. For each product in the expected map, if
   `compare_at(product_reference_unit, expected, consumed) ≠ 0`, record an issue
   *(order, product, consumed, expected)*.

The issues open the Consumption Warning assistant. A policy of `warning` lets any
manufacturing user confirm; a policy of `strict` requires a manufacturing administrator.

### 9.2 Worked example — a strict consumption policy rejecting forty-one legs for ten tables

Recipe *Table*, consumption policy **Blocked** (`strict`), 1 Table from 4 Legs and 1 Table
top. An order for **10 Tables** is confirmed; the order copies the policy `strict`.

The operator sets the quantity producing to 10. The distribution fills 40 Legs and 10 Table
tops. The operator then edits the legs component move to **41** — which, because the edited
quantity differs from the demand, automatically marks that move as manual-consumption and
picked.

Marking the order as done:

1. Sanity checks pass.
2. The automatic-production check passes (nothing is tracked), so quantities are set —
   but the manual-consumption move is not rewritten, so the 41 stands.
3. The consumption check runs because the policy is not `flexible`:
   - expected for Legs: `40 × 10 ÷ 10 = 40`;
   - consumed for Legs: 41;
   - `compare_at(Units, 40, 41) = −1 ≠ 0` → issue *(order, Leg, 41, 40)*;
   - expected for Table top: `10 × 10 ÷ 10 = 10`; consumed 10 → no issue.
4. One issue exists, so the Consumption Warning assistant opens with a single line:

   | Manufacturing Order | Product | Unit | Consumed | To Consume | Policy |
   |---|---|---|---|---|---|
   | the order | Leg | Units | 41.00 | 40.00 | Blocked |

   The assistant's own policy is the strictest of its lines: `strict`.
5. Because the policy is `strict`, **the confirm control is not available to a plain
   manufacturing user**; only a manufacturing administrator may confirm and close the order.
   A plain user's only options are:
   - **set quantities** — which rewrites the legs move to the expected 40, marks it picked,
     zeroes the assistant's expected figure so a duplicate product is not distributed twice,
     and then confirms; the order closes having consumed 40 Legs;
   - abandon the closing.
6. If a manufacturing administrator confirms, the completion re-runs with the consumption
   check skipped, and the order closes having consumed **41 Legs**. The extra leg is part of
   the production cost and therefore raises the unit cost of the ten tables.

**Contrast with the other policies.** With `warning` the same assistant opens but any
manufacturing user may confirm. With `flexible` no check runs at all and the order closes
silently at 41 Legs.

### 9.3 The production cost formula

Run for each order when its inventory is posted, after the component moves are done and
before the finished moves are posted.

Let *finished moves* be the finished moves for the order's own product that are neither
`done` nor `cancel` and whose done quantity is strictly positive. If there are none, no cost
is computed.

```formula
work_centre_cost = sum over the order's work orders of work_order_cost
quantity         = sum over finished moves of convert( done_quantity , move_unit , reference_unit )
extra_cost       = extra_unit_cost × quantity
total_cost       = sum over consumed component moves of move_value
                   + work_centre_cost + extra_cost
```

where

```formula
work_order_cost = ( duration_expected ÷ 60 ) × ( work_order_cost_per_hour or work_centre_cost_per_hour )
                      when the work order estimates its cost
work_order_cost = ( merged_tracked_minutes ÷ 60 ) × ( work_order_cost_per_hour or work_centre_cost_per_hour )
                      otherwise
```

A Work Order estimates its cost exactly when its state is `progress` or `done`, it has a
non-zero expected duration, and its cost mode is `estimated`. The tracked minutes are the
merged intervals of its closed time logs (optionally only those ending before a given date).

**By-product allocation.** Let *by-product moves* be the by-product moves that are neither
`done` nor `cancel` and whose done quantity is strictly positive. Walk them, accumulating
`byproduct_cost_share` as the plain sum of their cost-share percentages, and for each:

- if the by-product's costing method is first-in-first-out or average:
  - skip it when its cost share is zero at two decimals (its unit price is left alone);
  - otherwise

    ```formula
    byproduct_quantity = convert( done_quantity , move_unit , byproduct_reference_unit )
    byproduct_price_unit = total_cost × cost_share ÷ 100 ÷ byproduct_quantity
                           ( or 0 when the quantity is zero )
    ```

- otherwise (a standard-price by-product) its unit price is its own standard price, and its
  cost share still counts toward `byproduct_cost_share`.

**Finished allocation.**

- If the finished product's costing method is neither first-in-first-out nor average, the
  finished moves take the product's standard price.
- Otherwise

  ```formula
  finished_price_unit = total_cost × round_to_4_decimals( 1 − byproduct_cost_share ÷ 100 ) ÷ quantity
  ```

  The rounding of the complement to four decimal places is load-bearing: it is what makes a
  cost share of, say, 33.33 percent leave exactly 0.6667 of the cost on the finished
  product.

### 9.4 Worked example — a by-product with a ten percent cost share

Recipe *Plank cutting*: 1 Plank set from 1 Log, with a by-product **Sawdust** of 2
Kilograms at a cost share of **10 percent**. The operation *Cut* takes 30 minutes at a work
centre costing 60.00 per hour, efficiency 100 percent, no setup, no cleanup, capacity 1.

An order for 1 Plank set. The log consumed is valued at 240.00. The extra unit cost is 0.

1. `work_centre_cost = (30 ÷ 60) × 60.00 = 30.00`.
2. `quantity = 1` Plank set.
3. `extra_cost = 0 × 1 = 0`.
4. `total_cost = 240.00 + 30.00 + 0 = 270.00`.
5. By-product Sawdust: cost share 10; `byproduct_cost_share = 10`.
   - `byproduct_quantity = 2` Kilograms;
   - `byproduct_price_unit = 270.00 × 10 ÷ 100 ÷ 2 = 27.00 ÷ 2 = 13.50` per Kilogram;
   - the Sawdust move's value is therefore `2 × 13.50 = 27.00`.
6. Finished Plank set:
   - `round_to_4_decimals(1 − 10 ÷ 100) = round_to_4_decimals(0.9) = 0.9`;
   - `finished_price_unit = 270.00 × 0.9 ÷ 1 = 243.00`.
7. Check: `243.00 + 27.00 = 270.00` — the whole production cost is allocated.

**Two by-products.** Add a second by-product *Offcut*, 1 Unit, cost share 5 percent, on the
same order. Then `byproduct_cost_share = 15`, the Offcut price is
`270.00 × 5 ÷ 100 ÷ 1 = 13.50`, the Sawdust price is unchanged at 13.50 per Kilogram, and
the finished price is `270.00 × round_to_4_decimals(0.85) ÷ 1 = 270.00 × 0.85 = 229.50`.
Check: `229.50 + 27.00 + 13.50 = 270.00`.

**A zero cost share.** A by-product with cost share 0 whose costing method is first-in-
first-out or average has **no unit price written at all** (the loop skips it), so it is
valued by the ordinary incoming-valuation rule of the inventory valuation domain; the whole
270.00 stays on the finished product.

### 9.5 Recipe cost roll-up (setting a product's cost from its recipe)

Used by the "compute price from recipe" action.

1. Find the recipe of the product. If there is none, look for a recipe that lists the
   product as a by-product with a non-zero cost share, ordered by sequence, specific variant
   and identifier, and take the first; if there is none either, do nothing.
2. Start from `total = 0`.
3. For each operation of the recipe that is not skipped for the product, add its cost
   (`(time_total ÷ 60) × work_centre_cost_per_hour`, with the contextual quantity being the
   recipe quantity).
4. For each component line that is not skipped for the product:
   - if the line has a sub-recipe **and** that sub-recipe is in the set being recomputed,
     recurse to obtain the sub-recipe's unit cost *c*, and add
     `convert_price(c, component_reference_unit, line_unit) × line_quantity`;
   - otherwise add
     `convert_price(component_standard_price, component_reference_unit, line_unit) × line_quantity`.
5. **When the product is the by-product of the recipe:**

   ```formula
   byproduct_quantity = sum over the matching by-product lines of
                        convert( line_quantity , line_unit , product_reference_unit , unrounded )
   byproduct_share    = sum over the matching by-product lines of cost_share
   unit_cost          = total × byproduct_share ÷ 100 ÷ byproduct_quantity
   ```

   (and 0 when either the share or the quantity is zero).
6. **When the product is the finished product:**

   ```formula
   byproduct_share = sum over all by-product lines of cost_share
   total'          = total × round_to_4_decimals( 1 − byproduct_share ÷ 100 )   when the share is non-zero
   unit_cost       = convert_price( total' ÷ recipe_quantity , recipe_unit , product_reference_unit )
   ```

7. Write the result to the product's standard price.

**Worked example.** Recipe *Plank cutting* above, with a Log standard price of 240.00, the
30-minute operation at 60.00 per hour, and the 10 percent Sawdust share:

- operations: `(30 ÷ 60) × 60.00 = 30.00`;
- components: `240.00 × 1 = 240.00`;
- `total = 270.00`;
- finished: `270.00 × 0.9 = 243.00`, divided by the recipe quantity 1 and converted from
  the recipe unit to the reference unit → standard price **243.00**;
- Sawdust, computed separately: `270.00 × 10 ÷ 100 ÷ 2 = 13.50` per Kilogram.

---

## 10. Unbuild

### 10.1 Building the move lists

**Consume moves** — what the unbuild takes away:

- *With a source order:* for each finished move of that order in state `done` (the finished
  product and every by-product), create a move of
  `done_quantity × factor` where

  ```formula
  factor = unbuild_quantity ÷ convert( order_produced_quantity , order_unit , unbuild_unit )
  ```

  from the unbuild's source location to the finished move's **source** location (that is,
  back into the production location), tagged with the unbuild as the consumed-unbuild link
  through the copy, and with the finished move recorded as the originating returned move.
- *Without a source order:* create one move for the unbuild's own product and quantity, from
  the unbuild's source location to the product's production location, plus, for every
  by-product line of the recipe that is not skipped, a move of

  ```formula
  byproduct_quantity = byproduct_line_quantity × factor
  factor             = convert( unbuild_quantity , unbuild_unit , recipe_unit ) ÷ recipe_quantity
  ```

  from the unbuild's source location to the by-product's production location.

**Produce moves** — what the unbuild gives back:

- *With a source order:* for each component move of that order in state `done`, create a
  move of `done_quantity × factor` (the same factor) from the component move's **destination**
  location (the production location) to the unbuild's destination location.
- *Without a source order:* explode the recipe with the same factor and create one move per
  leaf line, from the product's production location to the unbuild's destination location.

Every created move uses `make_to_stock`, carries the unbuild's creation timestamp as its
date, and the warehouse of its destination location.

### 10.2 The unbuild algorithm

**Preconditions.** Company consistency; a lot is provided when the product is tracked; the
source order, if named, is `done`.

1. Create and confirm the consume moves. Create and confirm the produce moves, and set their
   done quantity to zero.
2. Collect the component lots already restored by **other** unbuilds of the same source
   order, for serial-tracked components.
3. Separate from the consume moves the ones for the unbuild's own product (*finished moves*)
   — the rest are the by-product consume moves.
4. If any produce move is tracked and no source order is named, fail with **"Please specify a
   manufacturing order. It will allow us to retrieve the lots/serial numbers of the correct
   components and/or byproducts."** The same check applies to the by-product consume moves.
5. For each finished move whose demand exceeds its done quantity, create a move line for the
   difference, carrying the unbuild's lot.
6. For every produce move and by-product consume move whose demand is greater than or equal
   to its done quantity:
   1. Find the *original* move of the source order for the same product: a component move
      when the move is a produce move, a finished move when it is a consume move.
   2. If there is none, simply set the move's done quantity to its rounded demand and
      continue.
   3. Otherwise walk the original move's lines. For a produce move with a lot on the unbuild,
      restrict them to the lines whose produced-line lots include the unbuild's lot and whose
      own lot is not among the lots already restored by a previous unbuild.
   4. For each such line take
      `taken = round_at(move_unit, min(still_needed, line_quantity − already_used_on_that_line))`;
      when it is non-zero, create a move line on the new move with that quantity, that line's
      lot, that line's owner if any, and apply the put-away strategy; decrease
      *still_needed* and increase *already_used* for that original line.
   5. For a produce move with a positive *still_needed* left over, add it to the move's done
      quantity without a lot.
7. Mark every finished, consume and produce move picked, then post them in that order:
   finished, then consume, then produce.
8. Link traceability: the move lines of the consume moves record the produced lines (the
   move lines of the produce moves with a positive quantity) as their produced lines.
9. When a source order is named, post a note on it reading "*the quantity* *the unit name*
   unbuilt in *a link to the unbuild order*".
10. Write the state `done`.

### 10.3 The validate action

```formula
available = available_quantity_of( product , source_location , lot , strict )
required  = convert( unbuild_quantity , unbuild_unit , product_reference_unit )
```

If `compare(available, required)` at the "Product Unit" decimal precision is greater than or
equal to zero, perform the unbuild. Otherwise open the insufficient-quantity warning titled
**"*the product display name*: Insufficient Quantity To Unbuild"**, pre-filled with the
product, the source location, the unbuild and the required quantity.

### 10.4 Worked example — unbuilding two units

A Manufacturing Order produced **10 Tables** from 40 Legs and 10 Table tops and is `done`.
Its produced quantity is 10, its unit is Units. An Unbuild Order is created from that order
for **2 Tables**.

1. `factor = 2 ÷ convert(10, Units, Units) = 2 ÷ 10 = 0.2`.
2. Consume moves: the order's finished move for 10 Tables is `done`, so one consume move of
   `10 × 0.2 = 2` Tables is created, from the unbuild's source location (stock) to the
   finished move's source location (the production location). There are no by-products, so
   that is the only consume move.
3. Produce moves: the order's two component moves are `done` with 40 Legs and 10 Table tops,
   so two produce moves are created — `40 × 0.2 = 8` Legs and `10 × 0.2 = 2` Table tops —
   each from the production location to the unbuild's destination location (stock).
4. Because nothing is tracked, no lot matching is needed; each produce move takes its full
   demand: 8 Legs and 2 Table tops.
5. All three moves are marked picked and posted. Stock changes by −2 Tables, +8 Legs,
   +2 Table tops.
6. A note is posted on the source order: "2.0 Units unbuilt in *the unbuild order*".
7. The Unbuild Order's state becomes `done` and it can no longer be deleted.

**Valuation.** The consume move takes the Tables out at their costing method's outgoing
value; the produce moves bring the components back in at their incoming valuation.
See [accounting-effects.md](accounting-effects.md) §4.

**A second unbuild of 3 more Tables** on the same order uses `factor = 3 ÷ 10 = 0.3` and
therefore consumes 3 Tables and returns 12 Legs and 3 Table tops. The factor is always taken
against the order's **produced** quantity, never against what is left, so two unbuilds of 2
and 3 together return exactly half the components — the same as one unbuild of 5.

**With serial-tracked components** the lot matching of §10.2 step 6 restricts the returned
serial numbers to those whose produced lines carry the unbuild's finished lot, and excludes
the serial numbers already returned by a previous unbuild of the same order.

---

## 11. Lead times and dates

### 11.1 The manufacturing contribution to the lead time

When a manufacture rule is part of the rule chain answering a demand, it contributes:

1. Look up the recipe for the product with the rule's operation type and company. If there
   is none:

   ```formula
   total_delay        += 365
   no_bom_found_delay += 365
   ```

   and the explanation gains the pair ("No BoM Found", "+ 365 day(s)").
2. Add the recipe's manufacturing lead time:

   ```formula
   total_delay       += produce_delay
   manufacture_delay += produce_delay
   ```

   and the explanation gains ("Production End Date", *produce_delay*) and
   ("Manufacturing Lead Time", "+ *produce_delay* day(s)").
3. When the recipe's kind is `normal`, add the pre-production rules of the destination
   warehouse: for each warehouse of the rule's destination location whose manufacturing step
   configuration is not one step, find the rules that bring the product to its production
   location along the pre-production route, exclude the manufacture rule itself, and add
   their delays (computed with the global horizon suppressed).
4. Add the days to order:

   ```formula
   days_to_order  = the caller's value, defaulting to the recipe's days_to_prepare_mo
   total_delay   += days_to_order
   ```

   and the explanation gains ("Production Start Date", *days_to_order*) and
   ("Days to Supply Components", "+ *days_to_order* day(s)").

### 11.2 The planned dates of a generated order

```formula
date_planned  = requested_date − produce_delay days
date_planned  = date_planned − 1 hour            when the subtraction changed nothing
date_deadline = the caller's deadline, or date_planned + produce_delay days
date_start    = date_planned
```

The one-hour adjustment guarantees that an order with a zero manufacturing lead time still
starts strictly before the moment the goods are needed.

**Worked example.** A sale needs 20 units on the 20th at 17:00. The recipe's manufacturing
lead time is 3 days and its days to prepare is 2.

- `date_planned = the 20th at 17:00 − 3 days = the 17th at 17:00`; the subtraction changed
  the value, so no hour is removed.
- `date_deadline = the 17th at 17:00 + 3 days = the 20th at 17:00`.
- The order starts on the 17th at 17:00.
- The scheduler's horizon for creating the order is `3 + 2 = 5` days plus the component
  rules' own delays, so the order is created and confirmed on the 15th.

### 11.3 The order's finish date

Specified in [entities.md](entities.md) §11.6. In short:

```formula
date_finished = date_start + produce_delay days
```

and, when that leaves the finish equal to the start:

```formula
date_finished = the work-centre-availability finish, when every work order's work centre has a schedule
                and every slot search succeeds
date_finished = date_start + max( sum of the work orders' expected durations , 60 ) minutes   otherwise
```

**Worked example.** An order starting on the 1st at 08:00 with a recipe lead time of 0 and
two Work Orders expected at 90 and 45 minutes, whose work centres have no working schedule:
`date_finished = the 1st at 08:00 + max(135, 60) minutes = the 1st at 10:15`.

The same order with no Work Orders at all:
`date_finished = the 1st at 08:00 + max(0, 60) minutes = the 1st at 09:00`.

---

## 12. Finding the first available slot at a work centre

This is the scheduling primitive. Given a start instant, a duration in minutes, a direction
and optional intervals to ignore or to add, it returns the first window in the work centre's
working schedule that is long enough and free of other Work Orders, or a failure.

### 12.1 Algorithm (forward direction)

1. Read the maximum number of planning iterations from the system parameter
   `mrp.workcenter_max_planning_iterations`, defaulting to 50, floored at 1.
2. Let `remaining = duration = max(duration, 1 ÷ 60)` — a zero-length Work Order is treated
   as one second.
3. Let the step be 14 days. For each iteration *n* from 0 to the maximum minus one:
   1. The examined window is `[start + n × 14 days, start + (n + 1) × 14 days]`.
   2. Obtain the **available intervals**: the working intervals of the work centre's schedule
      for its resource in that window.
   3. Obtain the **occupied intervals**: the leaves of kind `other` for that resource in that
      window, excluding the leaves the caller asked to ignore.
   4. For each available interval *(start_i, stop_i)* in chronological order:
      1. Set the candidate interval's start to the running start if one is already held,
         otherwise to *start_i*.
      2. Let `interval_minutes` be the length of *(start_i, stop_i)* in minutes.
      3. While the candidate window
         `[running_start or start_i, start_i + min(remaining, interval_minutes) minutes]`
         conflicts with an occupied interval or with one of the caller's extra intervals:
         - move *start_i* to the latest end of the conflicting intervals, bounded above by
           *stop_i*;
         - recompute `interval_minutes`;
         - reset the running start to *start_i* when the interval is not exhausted, or clear
           it when it is, and reset `remaining` to the full duration.
      4. If `interval_minutes ≥ remaining` compared at three decimal places, return
         *(running start, start_i + remaining minutes)*.
      5. Otherwise subtract `interval_minutes` from `remaining` and continue with the next
         available interval.
4. If no window is found, return the failure *(nothing, "No available slot 700 days after
   the planned start")*.

The 50 iterations of 14 days give the documented horizon of 700 days.

### 12.2 Backward direction

The same, with the window running backwards from the start instant, the available intervals
reversed, the candidate window anchored at its end, conflicts pushing the end backwards to
the earliest conflicting start, and the search stopping as soon as the examined window
begins before the current instant.

### 12.3 Worked example

A work centre works Monday to Friday, 08:00–12:00 and 13:00–17:00. A Work Order of 300
minutes is to be planned from Monday 10:00. One other Work Order already occupies Monday
14:00–15:00.

- Available intervals in the first fortnight: Monday 08:00–12:00, Monday 13:00–17:00,
  Tuesday 08:00–12:00, …
- The search starts at Monday 10:00, so the first usable interval is Monday 10:00–12:00,
  120 minutes. The candidate window 10:00 → 10:00 + min(300, 120) = 12:00 does not conflict.
  120 < 300, so `remaining = 300 − 120 = 180` and the running start stays at 10:00.
- Next interval Monday 13:00–17:00, 240 minutes. The candidate window is 10:00 → 13:00 +
  min(180, 240) = 16:00. That conflicts with the occupied 14:00–15:00, so *start_i* moves to
  15:00, `interval_minutes` becomes 120, the running start is reset to 15:00 and `remaining`
  is reset to 300. The new candidate is 15:00 → 15:00 + min(300, 120) = 17:00, which does not
  conflict. 120 < 300, so `remaining = 180`, running start 15:00.
- Next interval Tuesday 08:00–12:00, 240 minutes. Candidate 15:00 (Monday) → 08:00 +
  min(180, 240) = 11:00 (Tuesday); no conflict; 240 ≥ 180, so the slot is
  **Monday 15:00 → Tuesday 11:00**.

Note that the returned window spans the closed hours: the slot is the calendar reservation,
and the duration is measured only over working time.

---

## 13. Quantities of a kit product

A kit product has no stock of its own. Its on-hand, forecast, incoming, outgoing and free
quantities are derived from its components.

### 13.1 Algorithm

1. Partition the products into those with a `phantom` recipe in the active company (the
   kits) and the rest. Compute the rest by the ordinary rule.
2. For each kit:
   1. Explode its kit recipe for a multiplier of 1.
   2. Group the leaf lines by component.
   3. For each component group, compute the quantity of that component needed per single
      recipe output:

      ```formula
      qty_per_kit = sum over the group's lines of
          convert( line_exploded_quantity ÷ line_original_quantity ,
                   line_unit , component_reference_unit , unrounded )
      ```

      Lines whose component is not storable, and lines whose exploded quantity is zero at the
      line's unit, are skipped. A component whose `qty_per_kit` ends up zero is skipped.
   4. For each of the five quantities *q* in {forecast, on hand, incoming, outgoing, free},
      append

      ```formula
      ratio_q = round_down_at( component_reference_unit , component_q ÷ qty_per_kit )
      ```

   5. If at least one ratio was produced:

      ```formula
      kit_q = floor( round_at( component_reference_unit , min( ratios_q ) × recipe_quantity ) )
      ```

      Otherwise all five quantities are 0.

### 13.2 Worked example

Kit *Desk set* = 1 Desk lamp + 2 Pen holders + 3 Notebooks, recipe quantity 1. On hand: 17
lamps, 9 pen holders, 40 notebooks.

- lamps: `qty_per_kit = 1`; ratio = `round_down(17 ÷ 1) = 17`;
- pen holders: `qty_per_kit = 2`; ratio = `round_down(9 ÷ 2) = round_down(4.5) = 4`;
- notebooks: `qty_per_kit = 3`; ratio = `round_down(40 ÷ 3) = round_down(13.33) = 13`;
- minimum = 4; `kit_on_hand = floor(round(4 × 1)) = 4`.

The on-hand quantity of *Desk set* is **4**.

If the recipe produced 2 Desk sets per kit output (recipe quantity 2, components 2 lamps,
4 pen holders, 6 notebooks), the per-kit-output quantities would be the same relative to
the recipe output, the ratios would be 8, 2 and 6, the minimum 2, and
`kit_on_hand = floor(round(2 × 2)) = 4` — the same answer, as it must be.

---

## 14. Producible quantity and production capacity

### 14.1 On the order

```formula
production_capacity = min( product_qty ,
                           round_at( product_reference_unit ,
                                     min over storable component moves of
                                       convert( component_on_hand , component_reference_unit , move_unit )
                                       ÷ unit_factor ) )
```

Only component moves with a non-zero unit factor whose product type is not `consu` — that
is, the storable ones — are considered; when there are none, the capacity is the order
quantity.

### 14.2 On the recipe structure report

```formula
producible_qty = min over storable components with a non-zero base line quantity of
                   round_down_to_integer( component_free_to_manufacture_qty ÷ component_base_line_qty )
                 × recipe_quantity
```

where the base line quantity is the recipe line's own quantity (not the exploded quantity),
summed across duplicate lines for the same component, and the free-to-manufacture quantity
is the component's free quantity expressed in the recipe unit, floored at zero. When no
component qualifies, the producible quantity is 0.

At the top level the report labels the result "*the producible quantity* Ready To Produce",
or "No Ready To Produce" when it is zero. At any lower level, when the required quantity
exceeds the available quantity, the label is "*the missing quantity* To *the route name*",
with the route name defaulting to "Order".

### 14.3 The order overview's readiness label

For a `draft` or `confirmed` order with component data:

1. For each storable component, accumulate the required, reserved and free quantities in the
   component's reference unit.
2. `producible = order_quantity`. For each component with a non-zero requirement:

   ```formula
   component_producible = round_down_at( order_unit ,
       order_quantity × ( reserved + free ) ÷ required )
   ```

   If it is not strictly positive, the label is **"Not Ready"**. Otherwise
   `producible = min(producible, component_producible)`.
3. If `producible` is not strictly positive the label is **"Not Ready"**; if it is strictly
   less than the order quantity the label is "*the producible quantity* Ready"; otherwise the
   label is **"Ready"**.

---

## 15. Recipe structure report

### 15.1 Recursive data

For a recipe at a given level, with a current quantity expressed in the recipe unit:

```formula
current_quantity = line_qty                                                    at the root
current_quantity = convert( line_qty , parent_line_unit , recipe_unit )        below the root
```

For each component line not skipped for the product:

```formula
line_quantity = ( current_quantity ÷ max( recipe_quantity , 1 ) ) × line_quantity_on_the_recipe
```

- A line whose component has a sub-recipe recurses at level + 1 with that line quantity.
- A line whose component has no sub-recipe is a leaf, valued at

  ```formula
  component_cost = round_to_currency(
      convert_price( component_standard_price , component_reference_unit , line_unit ) × line_quantity )
  ```

Two components with the same product and the same unit are merged: quantities and costs are
added, and the availability of the worse of the two is kept.

```formula
recipe_cost_before_operations = sum over components of component_cost
operations_cost               = sum over operations of round_to_currency( operation_cost )
operations_time               = sum over operations of operation_total_duration
operations_delay              = max over operations of the operation availability delay
recipe_cost_with_operations   = recipe_cost_before_operations + operations_cost
```

The operations are evaluated with the contextual product and the current quantity rounded
**upward** at the "Product Unit" decimal precision.

By-products:

```formula
byproduct_quantity   = ( current_quantity ÷ max( recipe_quantity , 1 ) ) × byproduct_line_quantity
byproduct_share      = byproduct_cost_share ÷ 100   when the line quantity is positive, else 0
byproduct_portion    = sum over by-products of byproduct_share
byproduct_cost       = round_to_currency( recipe_cost_with_operations × byproduct_share )
cost_share_of_output = round_to_4_decimals( 1 − byproduct_portion )
final_recipe_cost    = recipe_cost_with_operations × cost_share_of_output
```

### 15.2 Availability

A component's availability has two parts.

**Stock availability** — one of `available` (delay 0), `expected` (delay in days) or
`unavailable` (no delay):

1. A non-storable product is always `available` with delay 0.
2. Accumulate the consumption of the product across the whole report; when the accumulated
   consumption is less than or equal to the free quantity, the state is `available` with
   delay 0.
3. Otherwise search the forecast series for the earliest date at or after today whose
   forecast quantity is at least the accumulated consumption. When one exists the state is
   `expected` with `delay = (that date − today) in days`.
4. Otherwise `unavailable`.

**Resupply availability** — computed from the route found for the product:

- when the route is a manufacture route:

  ```formula
  max_component_delay = max over components of the component availability delay
  produce_delay       = manufacture_delay + max_component_delay
  ```

  and the state is `estimated` with that delay. When any component has no delay at all (it
  is unavailable and cannot be resupplied), the whole result is `unavailable`.
- otherwise `unavailable`.

The reported availability is the stock availability when the level is not the root and the
stock state is not `unavailable`; otherwise it is the resupply availability.

The route information for a manufacture route reports:

```formula
manufacture_delay = recipe_produce_delay + sum of the delays of the rules found
lead_time         = manufacture_delay + recipe_days_to_prepare_mo
```

where "the rules found" are the rules from the warehouse stock location to the product plus
the rules from the product's production location along the warehouse routes, excluding
duplicates.

The display text is "Available", "Not Available", "Expected *today + delay*" or
"Estimated *today + delay*".

### 15.3 Planning simulation

When the availability state at a level is `unavailable` or `estimated` and the recipe has
operations, the report simulates the planning to give each operation an estimated finish:

```formula
qty_requested  = convert( current_quantity , recipe_unit , product_reference_unit )
qty_to_produce = convert( max( 0 , qty_requested − ( product_forecast when below the root else 0 ) ) ,
                          product_reference_unit , recipe_unit )
simulation_start = today + max_component_delay days, at the start of that day
```

The simulation then mirrors the real planner: with operation dependencies allowed, each
operation with no successor is planned recursively; without them, the operations are planned
one after another, each starting where the previous finished. Each operation is tried on its
work centre and every alternative, each with its own total duration, and the earliest finish
wins; the chosen window is added to a per-work-centre list of simulated occupations so that
later operations do not overlap it. Failure to find any slot raises **"Impossible to plan.
Please check the workcenter availabilities."**

```formula
operation_availability_delay = ( simulated_finish_date − simulation_start_date ) in days
level_availability_delay     = max_component_delay + max( recipe_produce_delay , operations_delay )
```

---

## 16. Order overview report

### 16.1 Cost columns

Three cost columns are produced for every component, every operation and the order itself.

| Column | Meaning |
|---|---|
| Manufacturing Order cost | What the order, as it currently stands, is expected to cost. |
| Recipe cost | What the recipe says it should cost. |
| Real cost | What it has actually cost so far. |

For an operation:

```formula
expected_operation_cost = ( duration_expected ÷ 60 ) × ( work_order_cost_per_hour or work_centre_cost_per_hour )
current_operation_cost  = ( tracked_minutes ÷ 60 ) × ( work_order_cost_per_hour or work_centre_cost_per_hour )
recipe_operation_cost   = the operation's cost for the order's product, quantity and unit
```

with the real cost equal to the expected cost when the Work Order estimates its cost.

For the order:

```formula
mo_cost   = ( sum of the components' order costs + the operations' order cost ) × remaining_cost_share
bom_cost  = ( sum of the components' recipe costs + the operations' recipe cost ) × remaining_cost_share
real_cost = ( sum of the components' real costs + the operations' real cost ) × remaining_cost_share
unit_cost = cost ÷ ( quantity or 1 )
```

where `remaining_cost_share` is 1 minus the total by-product share, and the quantity is the
order's quantity to produce, or its produced quantity once the order is done.

A recipe cost is additionally increased, before the by-product share is applied, by the
components and operations that the recipe has but the order does not: for a missing
component line,

```formula
missing_line_cost = round_to_currency(
    convert_price( component_standard_price , component_reference_unit , line_unit )
    × line_quantity × order_total_quantity ÷ recipe_quantity )
```

and for a missing operation the operation's cost scaled the same way.

### 16.2 The cost breakdown

Produced only for a done order that has by-product moves. For each by-product move that is
neither cancelled nor of zero cost share at two decimals:

```formula
byproduct_quantity[p] += round_half_up( convert( done_quantity , move_unit , product_reference_unit ) )
total_cost[p]         += total_real_cost × cost_share ÷ 100
component_cost[p]     += total_real_cost_of_components × cost_share ÷ 100
operation_cost[p]     += total_real_cost_of_operations × cost_share ÷ 100
```

and the finished product's line is

```formula
finished_component_unit_cost = total_real_cost_of_components × remaining_cost_share ÷ order_total_quantity
finished_operation_unit_cost = total_real_cost_of_operations × remaining_cost_share ÷ order_total_quantity
finished_total_unit_cost     = total_real_cost × remaining_cost_share ÷ order_total_quantity
```

Each by-product line divides its accumulated costs by its accumulated quantity to give a
unit cost.

### 16.3 Comparison indicators

```formula
indicator( expected , current ) = none      when compare_to_currency( current , expected ) = 0
                                            or either value is absent
indicator( expected , current ) = "danger"  when current > expected
indicator( expected , current ) = "success" when current < expected
```

At the order level the indicator is only shown when at least one component or the operations
block carries the same indicator.

---

## 17. Miscellaneous formulas

### 17.1 Ratio between an order and a recipe

```formula
bom_quantity_in_product_unit = convert( recipe_quantity , recipe_unit , recipe_product_reference_unit )
ratio                        = bom_quantity_in_product_unit ÷ order_total_quantity
```

Used when a recipe is linked to a running order, to rescale the order's component demands
into recipe-line quantities and back.

**Worked example.** A recipe produces 5 units; an order produces 20 units.
`ratio = 5 ÷ 20 = 0.25`. A recipe line of 3 components per recipe output therefore becomes
`3 ÷ 0.25 = 12` components on the order.

### 17.2 Changing the quantity to produce

```formula
factor = new_quantity ÷ old_quantity
```

- Every component move that is neither `done` nor `cancel`:
  `new_demand = round_up_at(move_unit, old_demand × factor)`; the move is rewritten only when
  the result is strictly positive.
- Every finished move that is neither `done` nor `cancel`:
  `delta = (new_quantity − old_quantity) × unit_factor`. When the move has downstream moves
  and the delta is not zero at the move's unit, a **copy** of the move carrying just the
  delta is created and confirmed (so the downstream chain is extended rather than rewritten);
  otherwise the move's demand is increased by the delta. The finished moves are then
  reserved.
- Each Work Order's expected duration is recomputed with the extra ratio
  `new_quantity ÷ old_quantity`, and its producing quantity is set to
  `qty_production − qty_produced` (or to 1, or 0, for a serial-tracked product). A Work
  Order that had produced less than the new production quantity but was `done` returns to
  `progress`; one that has now produced exactly the production quantity and was `progress`
  becomes `done`.
- The moves are re-attached to their Work Orders, the last Work Order receiving every move
  with no operation.
- The scheduler is triggered for the component moves of orders in state `confirmed` or
  `progress`.

**Worked example.** An order for 10 Tables (40 Legs, 10 Table tops) is raised to 14.
`factor = 14 ÷ 10 = 1.4`.
- Legs: `round_up_at(Units, 40 × 1.4) = 56`.
- Table tops: `round_up_at(Units, 10 × 1.4) = 14`.
- Finished Table move: unit factor `10 ÷ max(10 − 0, 1) = 1`, delta `(14 − 10) × 1 = 4`; with
  no downstream move the demand becomes 14.

### 17.3 Production capacity shown on the split assistant

The split assistant displays the order's production capacity (§14.1) so the user can see how
many units the components on hand actually allow.

### 17.4 Work centre load

```formula
workcenter_load = sum of duration_expected over work orders in state blocked, ready or progress
```

expressed in minutes.

### 17.5 The weekly load graph bars

```formula
load_limit_hours = sum of the durations in hours of the attendance lines of the work centre's schedule
load_bar_w       = min( load_hours_in_week_w , load_limit_hours )
excess_bar_w     = max( round_half_up_to_1_decimal( load_hours_in_week_w − load_limit_hours ) , 0 )
```

with `load_hours_in_week_w` the sum, rounded to one decimal, of the expected durations in
minutes divided by 60 of the Work Orders in state `pending`, `waiting`, `ready` or
`progress` whose production date falls in week *w*.

### 17.6 Manufactured quantity of a product

```formula
mrp_product_qty = round_at( product_reference_unit ,
    sum over finished moves of done orders started in the last 365 days,
        not cancelled and marked picked, of
        convert( done_quantity , move_unit , product_reference_unit ) )
```

### 17.7 Kit unit price for valuation

When a kit product must be valued (for example for the cost of goods sold of a sold kit):

```formula
component_qty_per_kit[c] = sum over exploded lines of c of
    convert( line_exploded_quantity , line_unit , component_reference_unit , unrounded )
price_of_kit_batch       = sum over components c of
    component_unit_price(c) × ( component_qty_per_kit[c] ÷ recipe_quantity )
kit_unit_price           = price_of_kit_batch ÷ valuated_quantity
```

and 0 when the valuated quantity is zero at the product's reference unit. The component unit
price is the drop-shipped price when any of that component's valued moves is drop-shipped,
and the ordinary price otherwise.
