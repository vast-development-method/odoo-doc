# Manufacturing — Acceptance Criteria

Numbered Given / When / Then scenarios with concrete numbers. A reimplementation that
passes all of them behaves equivalently to the specified system for this domain.

Unless a scenario says otherwise, the following fixture applies.

**Fixture F.**

| Product | Type | Tracking | Costing | Reference unit | Standard price |
|---|---|---|---|---|---|
| Dining table | goods, storable | none | average | Units | — |
| Leg | goods, storable | none | standard | Units | 25.00 |
| Table top | goods, storable | none | standard | Units | 468.75 |
| Glass | goods, storable | none | standard | Units | 100.00 |
| Screw | goods, storable | none | standard | Units | 0.50 |

- Recipe **T1**: product *Dining table*, kind `normal`, quantity 1, unit Units, components
  4 × Leg and 1 × Table top, consumption policy *Allowed with warning*, no operations, no
  by-products, manufacturing lead time 0, days to prepare 0.
- One warehouse coded `WH`, one step, stock location *WH/Stock*, manufacturing operation
  type *WH: Manufacturing* with sequence code `MO`, backorder policy *ask*.
- The company currency rounds to two decimals; the unit *Units* has a rounding increment of
  1; the shared "Product Unit" decimal precision is 2.
- Inventory valuation is automated for every product; the production account, the stock
  valuation account and the stock journal are configured.

---

## 1. Recipes

### Scenario 1.1 — One table from four legs and one top

**Given** fixture F,
**and** stock holds 20 Legs and 20 Table tops at *WH/Stock*.

**When** a manufacturing user creates a Manufacturing Order and selects the product
*Dining table*,

**Then** the recipe **T1** is selected automatically,
**and** the quantity to produce is **1** with unit *Units*,
**and** the operation type is *WH: Manufacturing*,
**and** the components location and the finished-products location are both *WH/Stock*,
**and** exactly two component moves exist:

| Component | Demand | Unit | Source | Destination | Procurement method | Manual consumption |
|---|---|---|---|---|---|---|
| Leg | 4 | Units | WH/Stock | Production | take from stock | no |
| Table top | 1 | Units | WH/Stock | Production | take from stock | no |

**and** exactly one finished move exists: 1 *Dining table*, from Production to *WH/Stock*,
**and** no Work Order exists,
**and** the state is `draft`,
**and** the readiness is empty.

**When** the order is saved,

**Then** the reference is `WH/MO/00001`,
**and** a Production Group named `WH/MO/00001` exists and is stamped on all three moves,
**and** a Stock Reference named `WH/MO/00001` exists, linked to the order and to all three
moves.

**When** the order is confirmed,

**Then** the state is `confirmed`,
**and** the consumption policy of the order is *Allowed with warning*,
**and** the unit factor of the Leg move is `4 ÷ max(1 − 0, 1) = 4`,
**and** the unit factor of the Table top move is `1`,
**and** the unit factor of the finished move is `1`.

**When** the components are reserved,

**Then** the readiness is `assigned` (Ready),
**and** the should-consume quantity of the Leg move is 0 (the producing quantity is still
zero).

**When** the quantity producing is set to 1,

**Then** the Leg move's consumed quantity is `round(1 × 4) = 4` and it is marked picked,
**and** the Table top move's consumed quantity is `round(1 × 1) = 1` and it is marked
picked,
**and** the state is `to_close` (there are no Work Orders and the producing quantity has
reached the quantity to produce).

**When** the order is marked done,

**Then** no consumption warning is raised (4 consumed against 4 expected; 1 against 1),
**and** no backorder question is raised (`max(1 − 1, 0) = 0`),
**and** the state is `done`,
**and** the order is locked,
**and** the priority is `0`,
**and** the finish date is the instant of closing,
**and** the finished move's quantity is `round_half_up(1 × 1) = 1` and it is done,
**and** stock holds 16 Legs, 19 Table tops and 1 Dining table,
**and** the production cost is `4 × 25.00 + 1 × 468.75 = 568.75`,
**and** the finished move's unit price is `568.75 × 1.0 ÷ 1 = 568.75`.

### Scenario 1.2 — A recipe cycle is refused

**Given** a recipe producing *Frame* from 1 *Tube*,

**When** an administrator saves a recipe producing *Tube* from 1 *Frame*,

**Then** the save is refused with **"The current configuration is incorrect because it
would create a cycle between these products: Tube, Frame."** (the names are those of the
finished products reached in the cycle).

### Scenario 1.3 — A variant-restricted component line

**Given** a product template *Chair* with an attribute *Material* of creation mode
*always*, values *Wood* and *Metal*, producing variants *Chair (Wood)* and *Chair (Metal)*,
**and** a recipe for the template with two component lines: 4 × *Wooden leg* restricted to
{Wood}, and 4 × *Metal leg* restricted to {Metal},

**When** an order is created for *Chair (Wood)* with a quantity of 3,

**Then** exactly one component move exists, for 12 *Wooden legs*,
**and** no move exists for *Metal leg*.

**When** an order is created for *Chair (Metal)* with a quantity of 3,

**Then** exactly one component move exists, for 12 *Metal legs*.

### Scenario 1.4 — A never-variant restriction

**Given** the same template with an additional attribute *Engraving* of creation mode
*no variant*, values *Yes* and *No*,
**and** a component line 1 × *Engraving kit* restricted to {Yes},

**When** an order for *Chair (Wood)* is created **without** choosing any never-variant
value,

**Then** no move exists for *Engraving kit* (a never-variant restriction with no chosen
value is always skipped).

**When** an order for *Chair (Wood)* is created **with** the never-variant value *Yes*,

**Then** a move for 1 *Engraving kit* exists.

### Scenario 1.5 — A by-product may not be the finished product

**Given** a recipe for *Dining table*,

**When** an administrator adds a by-product line for *Dining table*,

**Then** the save is refused with **"By-product *the recipe display name* should not be the
same as bill of materials product."**

### Scenario 1.6 — The total by-product cost share is capped

**Given** a recipe with a by-product of cost share 70,

**When** a second by-product of cost share 40 is added, applicable to the same variant,

**Then** the save is refused with **"The total cost share for a bill of materials's by-products cannot
exceed 100."**

**But when** the two by-products are restricted to mutually exclusive variants,

**Then** the save succeeds, because the sum is checked per variant.

### Scenario 1.7 — A kit may not have a reordering rule

**Given** a reordering rule for the product *Desk set*,

**When** an administrator creates a kit recipe for *Desk set*,

**Then** the save is refused with **"You can not create a kit-type bill of materials for
products that have at least one reordering rule."**

**And conversely**, given a kit recipe for *Desk set*, creating a reordering rule for it is
refused with **"A product with a kit-type bill of materials can not have a reordering
rule."**

### Scenario 1.8 — A recipe with running orders cannot be deleted

**Given** recipe T1 and a confirmed order using it,

**When** an administrator deletes recipe T1,

**Then** the deletion is refused with **"You can not delete a Bill of Material with running
manufacturing orders. Please close or cancel it first."**

### Scenario 1.9 — Recipe selection order

**Given** four recipes for the template *Chair*:

| Recipe | Variant | Sequence | Kind | Operation type |
|---|---|---|---|---|
| A | — | 10 | normal | — |
| B | Chair (Red) | 10 | normal | — |
| C | — | 5 | phantom | — |
| D | — | 20 | normal | WH2: Manufacturing |

**When** a `normal` recipe is requested for *Chair (Red)* with no operation type,

**Then** recipe **A** is returned.

**When** a `phantom` recipe is requested for either variant,

**Then** recipe **C** is returned.

**When** a `normal` recipe is requested with operation type *WH2: Manufacturing*,

**Then** recipe **A** is returned, because its empty operation type matches any and sorts
first at sequence 10.

---

## 2. Producing, backorders and splits

### Scenario 2.1 — Producing ten with a partial completion of six and a backorder

**Given** fixture F,
**and** stock holds 100 Legs and 40 Table tops,
**and** an order `WH/MO/00021` for **10 Dining tables** using recipe T1, confirmed and
fully reserved,
**and** the operation type's backorder policy is *ask*.

**Then** the component demands are 40 Legs and 10 Table tops,
**and** the unit factors are `40 ÷ 10 = 4` and `10 ÷ 10 = 1`.

**When** the quantity producing is set to 6,

**Then** the Leg move's consumed quantity is `round(6 × 4) = 24` and it is picked,
**and** the Table top move's consumed quantity is `round(6 × 1) = 6` and it is picked,
**and** the state is `progress` (6 < 10).

**When** the order is marked done,

**Then** the consumption check computes, for Legs,
`expected = 40 × 6 ÷ 10 = 24` against consumed 24, and for Table tops
`expected = 10 × 6 ÷ 10 = 6` against consumed 6, so **no warning** is raised,
**and** the backorder quantity is `max(10 − 6, 0) = 4`,
**and** the Backorder Confirmation assistant opens with exactly one line naming the order.

**When** the user ticks *To Backorder* and confirms,

**Then** the original order is renamed `WH/MO/00021-001`,
**and** its backorder sequence becomes 1,
**and** its quantity to produce becomes 6,
**and** one backorder `WH/MO/00021-002` is created with backorder sequence 2, quantity 4,
state `confirmed`, the same production group, the same references, the same deadline and no
producing lot,
**and** the Leg moves are 24 (original) and 16 (backorder),
**and** the Table top moves are 6 and 4,
**and** the finished moves are 6 and 4,
**and** the reserved move lines are redistributed without unreserving and re-reserving,
**and** the original order posts 24 Legs and 6 Table tops as consumed and 6 Dining tables
as produced,
**and** the production cost is `24 × 25.00 + 6 × 468.75 = 600.00 + 2812.50 = 3412.50`,
**and** the finished unit price is `3412.50 ÷ 6 = 568.75`,
**and** the original order is `done` and locked,
**and** the backorder is `confirmed` with 4 to produce,
**and** `6 + 4 = 10`.

### Scenario 2.2 — The backorder policy "never"

**Given** the same situation but the operation type's backorder policy is *never*,

**When** the order is marked done at a producing quantity of 6,

**Then** no assistant opens,
**and** the order closes with 6 produced,
**and** the remaining component and finished moves that have no quantity are written to
state `done` with a demand of **zero** — not cancelled,
**and** no backorder exists.

### Scenario 2.3 — The backorder policy "always"

**Given** the same situation but the policy is *always*,

**When** the order is marked done at a producing quantity of 6,

**Then** no assistant opens,
**and** the backorder `WH/MO/00021-002` for 4 is created automatically,
**and** the completion is re-run with the backorder question skipped.

### Scenario 2.4 — Backorder naming

**Given** an order named `WH/MO/00021` with backorder sequence 0,

**When** it is split into 6 and 4,

**Then** the original becomes `WH/MO/00021-001` (the suffix is **appended**, because the
name does not end in a hyphen and digits),
**and** the backorder becomes `WH/MO/00021-002` (the suffix is **replaced**, because the
already-renamed original ends in `-001` and the new sequence 2 is above 1).

**When** the backorder is itself split,

**Then** the next name is `WH/MO/00021-003`.

### Scenario 2.5 — Splitting by batch size

**Given** an order for 17 units whose recipe enables batch sizing with a batch size of 5,

**When** the split assistant is opened,

**Then** the maximum batch size proposes 5,
**and** the number of splits is `round_up(17 ÷ 5) = 4`,
**and** the detail lines are 5, 5, 5 and 2,
**and** the validity indicator is true because `5 + 5 + 5 + 2 = 17`.

**When** the split is confirmed,

**Then** four orders exist with quantities 5, 5, 5 and 2, all in the same production group.

### Scenario 2.6 — Splitting with more than the quantity to produce

**Given** an order for 10,

**When** a split is requested with the amounts 7 and 5,

**Then** the split is refused with **"Unable to split with more than the quantity to
produce."**

### Scenario 2.7 — Merging

**Given** two confirmed orders for 4 and 6 *Dining tables*, both using recipe T1, both of
operation type *WH: Manufacturing*, neither with an extra component,

**When** they are merged,

**Then** one new order exists for **10** *Dining tables*,
**and** its origin is the sorted, comma-separated list of the two references,
**and** the two merged orders are cancelled,
**and** every move of the merged orders' production groups is re-stamped onto the new
order's group,
**and** each merged order shows the note "This production has been merge in *the new
order*".

**When** instead the two orders use different recipes,

**Then** the merge is refused with **"You can only merge manufacturing orders of identical
products with same bill of materials."**

### Scenario 2.8 — Changing the quantity to produce

**Given** a confirmed order for 10 *Dining tables* (40 Legs, 10 Table tops),

**When** the quantity is changed to 14,

**Then** `factor = 14 ÷ 10 = 1.4`,
**and** the Leg move's demand becomes `round_up(40 × 1.4) = 56`,
**and** the Table top move's demand becomes `round_up(10 × 1.4) = 14`,
**and** the finished move's demand becomes `10 + (14 − 10) × 1 = 14`,
**and** the scheduler is triggered for the component moves.

**When** instead the finished move has a downstream move,

**Then** a **copy** of the finished move carrying only the delta of 4 is created and
confirmed, and the original keeps 10, so that the downstream chain is extended rather than
rewritten.

---

## 3. Consumption policies

### Scenario 3.1 — A strict consumption policy rejecting forty-one legs for ten tables

**Given** fixture F with recipe T1's consumption policy set to **Blocked** (`strict`),
**and** stock holds 100 Legs and 40 Table tops,
**and** an order for **10 Dining tables**, confirmed and fully reserved,
**and** the acting user is a plain manufacturing user.

**When** the quantity producing is set to 10,

**Then** the Leg move's consumed quantity is 40 and the Table top move's is 10, both
picked,
**and** the state is `to_close`.

**When** the user edits the Leg move's consumed quantity to **41**,

**Then** that move's manual-consumption flag becomes true automatically,
**and** the move is marked picked.

**When** the user marks the order done,

**Then** the sanity checks pass,
**and** the automatic-filling pass does not overwrite the manual-consumption move,
**and** the consumption check computes, for Legs, `expected = 40 × 10 ÷ 10 = 40` against
consumed 41, and the comparison at the reference unit is non-zero, so an issue is recorded
as *(the order, Leg, 41, 40)*,
**and** for Table tops `expected = 10` against consumed 10, so no issue,
**and** the Consumption Warning assistant opens with exactly one line:

| Manufacturing Order | Product | Unit | Consumed | To Consume | Policy |
|---|---|---|---|---|---|
| the order | Leg | Units | 41.00 | 40.00 | Blocked |

**and** the assistant's policy is `strict`,
**and** the plain manufacturing user **cannot** confirm.

**When** the user chooses *set quantities* instead,

**Then** the Leg move's consumed quantity is rewritten to 40 and it stays picked,
**and** the assistant's expected figure for that line is zeroed so a second move of the
same product is not distributed twice,
**and** the completion re-runs with the consumption check skipped,
**and** the order closes having consumed 40 Legs and 10 Table tops.

**When** instead a manufacturing administrator confirms,

**Then** the completion re-runs with the consumption check skipped,
**and** the order closes having consumed **41 Legs**,
**and** the production cost is `41 × 25.00 + 10 × 468.75 = 1025.00 + 4687.50 = 5712.50`,
**and** the finished unit price is `5712.50 ÷ 10 = 571.25` instead of 568.75.

### Scenario 3.2 — The same overrun under "Allowed with warning"

**Given** the same situation with the policy *Allowed with warning*,

**When** a plain manufacturing user marks the order done at 41 Legs,

**Then** the same assistant opens,
**and** the plain user **may** confirm,
**and** the order closes at 41 Legs.

### Scenario 3.3 — The same overrun under "Allowed"

**Given** the same situation with the policy *Allowed*,

**When** the order is marked done at 41 Legs,

**Then** no check runs and no assistant opens,
**and** the order closes silently at 41 Legs.

### Scenario 3.4 — An extra component

**Given** an order under policy *Allowed with warning*,
**and** a manufacturing user adds an extra component move for 2 *Screws* and marks it
picked,

**When** the order is marked done,

**Then** the consumption check records an issue *(the order, Screw, 2, 0)*, because the
product is not in the expected map,
**and** the assistant shows the line with **To Consume** 0.00 and **Consumed** 2.00.

### Scenario 3.5 — A missing component

**Given** an order under policy *Allowed with warning* whose Table top move was deleted
before confirmation,

**When** the order is marked done at a producing quantity of 10,

**Then** the consumption check records an issue *(the order, Table top, 0, 10)*,
**and** choosing *set quantities* creates an **additional** picked component move for 10
Table tops, because no move exists to rewrite.

---

## 4. Kits

### Scenario 4.1 — A kit sold and delivered

**Given** a kit recipe for *Desk set*, kind `phantom`, quantity 1, components 1 × *Desk
lamp*, 2 × *Pen holder*, 3 × *Notebook*,
**and** stock holds 50 lamps, 50 pen holders and 50 notebooks,
**and** a customer orders **5 Desk sets** at 90.00 each.

**Then** the sales order shows **one** line: 5 *Desk sets* at 90.00, untaxed amount 450.00.

**When** the sale is confirmed,

**Then** the delivery move for 5 *Desk sets* is exploded at confirmation,
**and** the delivery transfer shows three moves:

| Component | Demand |
|---|---|
| Desk lamp | `round_up(5 × 1) = 5` |
| Pen holder | `round_up(5 × 2) = 10` |
| Notebook | `round_up(5 × 3) = 15` |

**and** no move for *Desk set* exists,
**and** each component move carries its recipe line, so the document labels them "Desk set
- 1/3", "Desk set - 2/3" and "Desk set - 3/3",
**and** the *Desk set* product itself holds no stock and no reservation.

**When** the warehouse ships 5 lamps, 10 pen holders and only 12 notebooks and validates,

**Then** the delivered quantity of the sales line is computed as
`min(5 ÷ 1, 10 ÷ 2, 12 ÷ 3) = min(5, 5, 4) = 4`, floored to **4**,
**and** invoicing at delivered quantities bills 4 *Desk sets* at 90.00 = 360.00.

**When** the customer returns 3 notebooks,

**Then** the notebook processed quantity becomes `12 − 3 = 9`, the ratio becomes 3, and the
delivered quantity drops to **3**.

### Scenario 4.2 — The on-hand quantity of a kit

**Given** the same kit,
**and** stock holds 17 lamps, 9 pen holders and 40 notebooks,

**When** the on-hand quantity of *Desk set* is read,

**Then** the ratios are `floor(17 ÷ 1) = 17`, `floor(9 ÷ 2) = 4` and `floor(40 ÷ 3) = 13`,
**and** the on-hand quantity is `floor(min(17, 4, 13) × 1) = 4`,
**and** the forecast control is hidden for the kit product.

### Scenario 4.3 — A kit cannot be counted

**Given** the same kit,

**When** a user tries to set an inventory quantity for *Desk set*,

**Then** the operation is refused with **"You should update the components quantity instead
of directly updating the quantity of the kit product."**

### Scenario 4.4 — A kit inside a manufactured recipe

**Given** a recipe for *Cabinet* with a component line 1 × *Door assembly*,
**and** *Door assembly* has a **kit** recipe of 1 × *Door panel* and 2 × *Hinge*,

**When** an order for 1 *Cabinet* is created,

**Then** the explosion descends into the kit and the order has **three** component moves:
1 *Door panel*, 2 *Hinges* and whatever else the *Cabinet* recipe lists,
**and** no move for *Door assembly* exists,
**and** no child order is created.

### Scenario 4.5 — A procurement for a kit

**Given** the same kit *Desk set*,

**When** a procurement for 5 *Desk sets* reaches the rules,

**Then** it is replaced, before any rule runs, by three procurements: 5 lamps, 10 pen
holders and 15 notebooks, each carrying its recipe line,
**and** no manufacture rule is run for *Desk set*.

---

## 5. Multi-level manufacturing

### Scenario 5.1 — A two-level bill with a made-to-order sub-assembly

**Given**:

| Recipe | Product | Kind | Quantity | Components |
|---|---|---|---|---|
| C1 | Cabinet | normal | 1 | 1 × Door assembly, 4 × Screw |
| D1 | Door assembly | normal | 1 | 1 × Door panel, 2 × Hinge |

**and** *Door assembly* carries the replenish-on-order route and the manufacture route,
**and** stock holds 0 *Door assembly*, 10 *Door panel*, 20 *Hinge* and 100 *Screw*,
**and** recipe D1 has a manufacturing lead time of 2 days; recipe C1 has 1 day.

**When** an order for **1 Cabinet** is created and confirmed,

**Then** the Cabinet order has exactly two component moves: 1 *Door assembly* and 4
*Screws*, because D1 is a `normal` recipe and the explosion does not descend into it,
**and** the *Door assembly* move's procurement method is **make to order**,
**and** confirming runs the manufacture rule for *Door assembly*,
**and** a second Manufacturing Order is created for 1 *Door assembly* with components 1
*Door panel* and 2 *Hinges*,
**and** the child order's finished move has the Cabinet's *Door assembly* component move as
its downstream move,
**and** the child order's production group is a **child group** of the Cabinet order's
group,
**and** the Cabinet order reports 1 generated order and the child reports 1 source order,
**and** the Cabinet order's *Door assembly* component move is in state `waiting`,
**and** the Cabinet order's readiness is `waiting` (Waiting Another Operation),
**and** the *Screw* move is reserved and the Cabinet order therefore is not ready.

**When** the child order is produced and closed,

**Then** 1 *Door assembly* enters stock,
**and** the Cabinet order's *Door assembly* component move is triggered for reservation and
becomes `assigned`,
**and** the Cabinet order's readiness becomes `assigned`.

**When** the Cabinet order is produced and closed,

**Then** stock holds 0 *Door assembly*, 9 *Door panel*, 18 *Hinge*, 96 *Screw* and
1 *Cabinet*.

**And** the lead-time contribution of the chain, computed for the Cabinet, includes the
Cabinet's own manufacturing lead time of 1 day and the days to prepare, plus the child
chain's own delays, so that the Cabinet order is created early enough for the child to
finish first.

### Scenario 5.2 — Cancelling a child order

**Given** the same chain with the child order confirmed,

**When** the child order is cancelled,

**Then** an activity is logged on the Cabinet order, because the child's finished moves
that are neither done nor cancelled trigger the downside-quantity log with the cancellation
flag.

### Scenario 5.3 — A missing recipe in the chain

**Given** a manufacture rule and a product with **no** recipe,

**When** the lead time of that chain is computed,

**Then** 365 days are added to the total delay and to the no-recipe delay,
**and** the explanation gains the pair ("No BoM Found", "+ 365 day(s)").

---

## 6. Work Orders and work centres

### Scenario 6.1 — A work order expected at sixty minutes taking seventy-five

**Given** an operation *Assemble* with a fixed cycle time of 60 minutes on a work centre
with 0 minutes setup, 0 cleanup, efficiency 100 percent, capacity 1, hourly cost 48.00, cost
mode *actual*,
**and** an order for 1 unit with that operation,
**and** the shipped productivity loss reasons.

**Then** the Work Order's expected duration is
`0 + 0 + round_up(1 ÷ 1) × 60 × 100 ÷ 100 = 60` minutes.

**When** the operator starts the Work Order at 09:00,

**Then** the Work Order's state is `progress`,
**and** one Productivity Log is open from 09:00 with the reason *Fully Productive Time*
(category productive), because the current duration does not exceed the expected duration,
**and** the order's state is `progress` and its start date is 09:00.

**When** the operator finishes the Work Order at 10:15,

**Then** the open log is closed at 10:15,
**and** because `75 > 60`, the log is split at
`10:15 − (75 − 60) minutes = 10:00`: the original becomes 09:00 → 10:00 and a copy becomes
10:00 → 10:15,
**and** the copy is rewritten with the first reason of category `performance`, which is
*Reduced Speed*,
**and** the productive category holds 60 minutes and the performance category 15 minutes,
**and** the real duration is `60 + 15 = 75.00` minutes,
**and** the duration deviation is `100 × (60 − 75) ÷ 60 = −25`, stored as the integer −25,
**and** the progress is forced to 100 because the state is `done`,
**and** with 1 unit produced the duration per unit is `round_to_2_decimals(75 ÷ 1) = 75.00`,
**and** the Work Order's hourly cost is snapshotted at 48.00,
**and** the operation cost is `(75 ÷ 60) × 48.00 = 60.00`.

**When** instead the operation's cost mode is *estimated*,

**Then** the operation cost is `(60 ÷ 60) × 48.00 = 48.00`, whatever the real duration.

### Scenario 6.2 — Expected duration with capacity, setup and efficiency

**Given** an operation with a fixed cycle time of 60 minutes on a work centre with 12
minutes setup, 8 minutes cleanup, efficiency 90 percent and capacity 4,
**and** an order for 10 units,

**Then** the Work Order's expected duration is
`12 + 8 + round_up(10 ÷ 4) × 60 × 100 ÷ 90 = 20 + 3 × 66.666… = 20 + 200 = 220` minutes.

**And given** an alternative work centre with 5 minutes setup, 5 cleanup, efficiency 120
percent and capacity 5,

**Then** the alternative's expected duration is
`working_per_cycle = max((220 − 12 − 8) × 90 ÷ (100 × 3), 0) = 60`,
`cycle_number' = round_up(10 ÷ 5) = 2`,
`5 + 5 + 2 × 60 × 100 ÷ 120 = 10 + 100 = 110` minutes,
**and** the planner prefers the alternative if a slot is available sooner.

### Scenario 6.3 — A work centre effectiveness computation

**Given** a cutting work centre with, over the last month, the closed Productivity Logs:

| Log | Category | Duration (minutes) |
|---|---|---|
| 1 | productive | 420 |
| 2 | productive | 300 |
| 3 | availability (Equipment Failure) | 60 |
| 4 | quality (Process Defect) | 30 |
| 5 | performance (Reduced Speed) | 45 |

**and** two finished Work Orders in the same month: one expected 60 minutes taking 75, one
expected 120 minutes taking 105.

**Then** the productive time is `(420 + 300) ÷ 60 = 12.00` hours,
**and** the blocked time is `(60 + 30 + 45) ÷ 60 = 2.25` hours (the performance category
counts as blocked here),
**and** the overall equipment effectiveness is
`round_to_2_decimals(720 × 100 ÷ (720 + 135)) = round_to_2_decimals(84.2105…) = 84.21`
percent,
**and** the performance is `100 × (60 + 120) ÷ (75 + 105) = 100` percent, stored as the
integer 100,
**and** the effectiveness target shown alongside is the configured value, by default 90.

**And when** the work centre has an open log of category `availability`,

**Then** its working state is `blocked`, and starting or validating any of its Work Orders
is refused with **"Please unblock the work center to start the work order."** and
**"Please unblock the work center to validate the work order"** respectively.

**And when** the work centre has an open log of category `productive` or `performance`,

**Then** its working state is `done` (In Progress) — note that the same category counts as
*blocked* for the effectiveness formula and as *in use* for the working state.

### Scenario 6.4 — Slot finding

**Given** a work centre working Monday to Friday 08:00–12:00 and 13:00–17:00,
**and** one Work Order already occupying Monday 14:00–15:00,

**When** a Work Order of 300 minutes is planned from Monday 10:00,

**Then** the search consumes Monday 10:00–12:00 (120 minutes, remaining 180),
**and** on Monday 13:00–17:00 it detects the conflict at 14:00–15:00, restarts at 15:00,
resets the remaining to 300, consumes 15:00–17:00 (120 minutes, remaining 180),
**and** on Tuesday 08:00–12:00 it finds 240 ≥ 180 and returns the window
**Monday 15:00 → Tuesday 11:00**,
**and** the calendar slot spans the closed hours, while the duration is counted only over
working time.

**When** no slot is found within 50 windows of 14 days,

**Then** the search fails with the text "No available slot 700 days after the planned
start", and planning refuses with **"Impossible to plan the workorder. Please check the
workcenter availabilities."**

### Scenario 6.5 — Work Order readiness through dependencies

**Given** an order for 10 units with two dependent Work Orders, *Cut* then *Assemble*,
**and** *Cut* has produced 6 and carried 0,
**and** *Assemble* has produced 0 and carried 0.

**Then** *Assemble*'s remaining quantity is `max(round(10 − 0 − 0), 0) = 10`,
**and** its ready quantity is `min(10 + 0, 6 + 0) − 0 − 0 = 6`,
**and** its state is `ready`.

**When** *Assemble* produces 4,

**Then** its ready quantity becomes `min(6 + 4, 6) − 4 − 0 = 2`.

**When** *Cut* is cancelled,

**Then** *Assemble*'s ready quantity becomes its remaining quantity, because every
predecessor is cancelled.

### Scenario 6.6 — Unplanning is refused after work has started

**Given** a planned order whose first Work Order is `progress`,

**When** the user unplans,

**Then** the operation is refused with **"Some work orders have already started, so you
cannot unplan this manufacturing order. It'd be a shame to waste all that progress,
right?"**

### Scenario 6.7 — A cyclic dependency

**Given** two Work Orders A and B of the same order where A is blocked by B,

**When** B is made blocked by A,

**Then** the save is refused with **"You cannot create cyclic dependency."**

---

## 7. By-products

### Scenario 7.1 — A by-product with a ten percent cost share

**Given** a recipe *Plank cutting*: 1 *Plank set* from 1 *Log*, with a by-product
2 Kilograms of *Sawdust* at a cost share of **10** percent,
**and** an operation *Cut* of 30 minutes at a work centre costing 60.00 per hour, efficiency
100 percent, capacity 1, no setup, no cleanup,
**and** the Log is valued at 240.00,
**and** both outputs use average costing,
**and** the extra unit cost is 0,
**and** an order for **1 Plank set**, produced and closed with the operation taking exactly
30 minutes.

**Then** `work_centre_cost = (30 ÷ 60) × 60.00 = 30.00`,
**and** `total_cost = 240.00 + 30.00 + 0 = 270.00`,
**and** the Sawdust unit price is `270.00 × 10 ÷ 100 ÷ 2 = 13.50` per Kilogram,
**and** the Sawdust move's value is `2 × 13.50 = 27.00`,
**and** the finished unit price is
`270.00 × round_to_4_decimals(1 − 0.10) ÷ 1 = 270.00 × 0.9 = 243.00`,
**and** `243.00 + 27.00 = 270.00`.

**And** the journal entries are:

| Entry | Account | Debit | Credit |
|---|---|---|---|
| Component consumed | Production | 240.00 | |
| | Stock Valuation (Log) | | 240.00 |
| Outputs produced | Stock Valuation (Plank set) | 243.00 | |
| | Production | | 243.00 |
| | Stock Valuation (Sawdust) | 27.00 | |
| | Production | | 27.00 |
| Labour | Production | 30.00 | |
| | Manufacturing overhead expense | | 30.00 |

**and** the Production account nets to `240.00 + 30.00 − 243.00 − 27.00 = 0.00`.

### Scenario 7.2 — Two by-products

**Given** the same recipe with an additional by-product 1 *Offcut* at a cost share of 5
percent,

**Then** the total share is 15,
**and** the Offcut unit price is `270.00 × 5 ÷ 100 ÷ 1 = 13.50`,
**and** the finished unit price is `270.00 × 0.85 = 229.50`,
**and** `229.50 + 27.00 + 13.50 = 270.00`.

### Scenario 7.3 — A by-product with a zero cost share

**Given** the same recipe with the Sawdust cost share set to 0,

**When** the order is closed,

**Then** **no** unit price is written on the Sawdust move (the allocation loop skips a zero
share under average costing), and it is valued by the ordinary incoming rule,
**and** the finished unit price is `270.00 × 1.0 ÷ 1 = 270.00`.

### Scenario 7.4 — A standard-cost by-product

**Given** the same recipe with Sawdust valued at standard cost 10.00 per Kilogram and a
cost share of 10,

**When** the order is closed,

**Then** the Sawdust move's unit price is its standard price **10.00**, not the allocated
13.50,
**and** the cost share of 10 still counts toward the total share,
**and** the finished unit price is still `270.00 × 0.9 = 243.00`,
**and** the total allocated is `243.00 + 2 × 10.00 = 263.00`, which differs from 270.00 —
the difference stays on the Production account.

---

## 8. Unbuilding

### Scenario 8.1 — Unbuilding two units

**Given** an order that produced **10 Dining tables** from 40 Legs and 10 Table tops,
`done`,
**and** the Dining table is valued at 568.75, the Leg at 25.00 and the Table top at 468.75,

**When** an Unbuild Order is created from that order for **2 Dining tables** and validated,

**Then** `factor = 2 ÷ 10 = 0.2`,
**and** one consume move of `10 × 0.2 = 2` *Dining tables* is created from stock to the
production location,
**and** two produce moves of `40 × 0.2 = 8` *Legs* and `10 × 0.2 = 2` *Table tops* are
created from the production location to stock,
**and** all three are marked picked and posted,
**and** stock changes by −2 Dining tables, +8 Legs, +2 Table tops,
**and** the source order shows the note "2.0 Units unbuilt in *the unbuild order*",
**and** the Unbuild Order is `done` and can no longer be deleted.

**And** the journal entries are:

| Entry | Account | Debit | Credit |
|---|---|---|---|
| Product consumed | Production | 1137.50 | |
| | Stock Valuation (Dining table) | | 1137.50 |
| Components returned | Stock Valuation (Leg) | 200.00 | |
| | Production | | 200.00 |
| | Stock Valuation (Table top) | 937.50 | |
| | Production | | 937.50 |

**and** the Production account nets to `1137.50 − 200.00 − 937.50 = 0.00`.

### Scenario 8.2 — A second unbuild of the same order

**Given** the same order after the unbuild of 2,

**When** a second Unbuild Order for 3 is validated,

**Then** `factor = 3 ÷ 10 = 0.3` — computed against the **produced** quantity, not against
what is left,
**and** 3 Dining tables are consumed and 12 Legs and 3 Table tops are returned,
**and** the two unbuilds together have returned exactly half of the original components.

### Scenario 8.3 — Insufficient stock

**Given** the same order but only 1 Dining table on hand,

**When** an Unbuild Order for 2 is validated,

**Then** the insufficient-quantity warning opens, titled **"Dining table: Insufficient
Quantity To Unbuild"**,
**and** confirming it performs the unbuild anyway.

### Scenario 8.4 — A tracked product without a lot

**Given** a serial-tracked finished product,

**When** an Unbuild Order is validated with no lot,

**Then** it is refused with **"You should provide a lot number for the final product."**

### Scenario 8.5 — Tracked components without a source order

**Given** a recipe whose component is serial-tracked,

**When** an Unbuild Order is validated with **no** source order,

**Then** it is refused with **"Please specify a manufacturing order. It will allow us to
retrieve the lots/serial numbers of the correct components and/or byproducts."**

### Scenario 8.6 — Serial numbers are not returned twice

**Given** an order that produced 2 units, each consuming one serial-tracked component
(serial numbers `S1` and `S2`), and produced serial numbers `F1` and `F2`,

**When** an Unbuild Order for 1 unit naming lot `F1` is validated,

**Then** only the component serial number that was consumed for `F1` is returned.

**When** a second Unbuild Order for 1 unit naming lot `F2` is validated,

**Then** the serial number already returned by the first unbuild is excluded from the
matching, so the other serial number is returned.

### Scenario 8.7 — Unbuilding an undone order

**Given** a confirmed order,

**When** an Unbuild Order naming it is validated,

**Then** it is refused with **"You cannot unbuild a undone manufacturing order."**

---

## 9. Subcontracting

### Scenario 9.1 — A subcontracted product with resupplied components

**Given** the subcontracting capability installed,
**and** a company subcontracting location *Subcontracting*,
**and** a subcontractor *Ply Works* with no specific location,
**and** a subcontracting recipe for *Assembled panel* producing 1 unit from 4 *Screws* and
1 *Raw panel*, naming *Ply Works*, with a manufacturing lead time of 1 day,
**and** both components carrying the *Resupply Subcontractor on Order* route,
**and** a vendor price of 27.00 per *Assembled panel* from *Ply Works*,
**and** Screws valued at 0.50 and Raw panels at 12.00,
**and** *Assembled panel* using average costing.

**When** a purchase order for **10 Assembled panels** at 27.00 is confirmed,

**Then** a receipt transfer with one move of 10 units from the vendor location to stock is
created, with partner *Ply Works*.

**When** that move is confirmed,

**Then** the subcontracting recipe is found for that partner,
**and** the move is marked as a subcontract receipt,
**and** its **source location becomes the subcontracting location**,
**and** it is re-reserved,
**and** it always bypasses reservation thereafter,
**and** one Manufacturing Order is created with: product *Assembled panel*, quantity 10,
that recipe, both locations the subcontracting location, operation type *Subcontracting*,
subcontractor *Ply Works*, start date the receipt date minus 1 day, origin the receipt's
name,
**and** that order is confirmed and reserved,
**and** its finished move points at the receipt move.

**When** the subcontracting order is confirmed,

**Then** the resupply chain creates an internal transfer *Resupply Subcontractor* for 40
*Screws* and 10 *Raw panels* from stock to the subcontracting location, addressed to *Ply
Works*.

**When** that resupply transfer is validated,

**Then** 40 Screws and 10 Raw panels sit at the subcontracting location, still owned and
valued by the company.

**When** the receipt is validated,

**Then** the subcontracting order is marked done **with the consumption check skipped**,
**and** its component and finished moves are backdated to one second before the earliest
receipt move line,
**and** the posted moves are: 40 Screws and 10 Raw panels out of the subcontracting
location into the production location; 10 Assembled panels out of the production location
into the subcontracting location; 10 Assembled panels out of the subcontracting location
into stock.

**And** the cost is computed as follows:

- with no vendor bill yet, the purchase order supplies `10 × 27.00 = 270.00`, so
  `extra_unit_cost = 270.00 ÷ 10 = 27.00`;
- the component value is `40 × 0.50 + 10 × 12.00 = 20.00 + 120.00 = 140.00`;
- `total_cost = 140.00 + 0 + 27.00 × 10 = 410.00`;
- the finished unit price is `410.00 ÷ 10 = 41.00`.

**And** the journal item written for the finished move is reduced by the subcontracting
service already carried on the receipt:
`journal_item_value = 410.00 − 27.00 × 10 = 140.00`, so the service is not counted twice.

### Scenario 9.2 — The quantity received changes

**Given** the same receipt, before validation,

**When** the received quantity is changed from 10 to 8 on an **untracked** product,

**Then** the single subcontracting order's quantity is changed to 8 through the
change-quantity assistant and it is re-reserved,
**and** no second order is created.

**When** instead the product is **serial-tracked** and the receipt lines carry serial
numbers `A`, `B` and `C`,

**Then** three subcontracting orders exist, one per serial number, each of quantity 1,
created by **splitting** an existing order so that the component reservations survive,
**and** removing a serial number from the receipt deletes its order, except that at least
one order is always kept (with its lot cleared) so that a further split has something to
split from.

### Scenario 9.3 — Portal restrictions

**Given** a portal user belonging to *Ply Works*,

**When** that user opens a subcontracting order through the portal and writes the component
move lines, the producing lots, the producing quantity or the quantity to produce,

**Then** the write succeeds.

**When** that user writes any other field,

**Then** the write is refused with **"You cannot write on fields *the field list* in
mrp.production."**

**When** that user tries to create a Stock Move in state `done`,

**Then** it is refused with **"Portal users cannot create a stock move with a state 'Done'
or change the current state to 'Done'."**

### Scenario 9.4 — A subcontracting recipe with an operation

**Given** a recipe of kind *Subcontracting*,

**When** an administrator adds an operation or a by-product line,

**Then** the save is refused with **"You can not set a Bill of Material with operations or
by-product line as subcontracting."**

### Scenario 9.5 — Merging subcontracted orders

**Given** two subcontracting orders,

**When** a user merges them,

**Then** the merge is refused with **"Subcontracted manufacturing orders cannot be
merged."**

### Scenario 9.6 — The subcontracting lead time

**Given** a subcontracting recipe with a manufacturing lead time of 3 days and 2 days to
prepare,
**and** a vendor lead time of 4 days,
**and** 1 day to purchase.

**Then** because `4 ≥ 3 + 2` is false, the manufacturing branch applies:
the total delay gains 3 days ("Receipt Date", "Manufacturing Lead Time + 3 day(s)") and
2 days ("Production Start Date", "Days to Supply Components + 2 day(s)"), plus the
purchase-side delays computed with the vendor lead time ignored.

**And when** the vendor lead time is 6 days,

**Then** `6 ≥ 5` holds and the vendor branch applies: the total delay gains 6 days
("Receipt Date", "Vendor Lead Time + 6 day(s)").

---

## 10. Multi-step warehouses

### Scenario 10.1 — Two steps

**Given** the warehouse switched to *Pick components then manufacture*,

**Then** the *Pre-Production* location is active,
**and** the manufacturing operation type's default source is *Pre-Production*,
**and** the route *Pick components and then manufacture* contains a pull rule
Stock → Pre-Production through *Pick Components* and a pull rule Pre-Production → Production
through *Manufacturing*.

**When** an order for 10 *Dining tables* is confirmed,

**Then** a *Pick Components* transfer is created for 40 Legs and 10 Table tops from Stock to
Pre-Production,
**and** the order's component moves are sourced from Pre-Production and are in state
`waiting`,
**and** the order's readiness is `waiting`.

**When** the pick transfer is validated,

**Then** the order's component moves become `assigned` and the readiness becomes
`assigned`.

### Scenario 10.2 — Three steps

**Given** the warehouse switched to *Pick components, manufacture, then store products*,

**Then** the *Post-Production* location is active,
**and** the manufacturing operation type's default destination is *Post-Production*,
**and** the route additionally contains a **push** rule Post-Production → Stock through
*Store Finished Product*,
**and** the manufacture rule propagates cancellation.

**When** the order is closed,

**Then** the finished goods land in *Post-Production*,
**and** a *Store Finished Product* transfer is created to move them into Stock,
**and** the finished move records the push rule's destination as its final location.

---

## 11. Locking, permissions and multi-company

### Scenario 11.1 — The unlocked-by-default setting

**Given** two orders in state `confirmed`, both locked,

**When** an administrator turns *Unlock Manufacturing Orders* on,

**Then** both orders become unlocked immediately.

**When** the setting is turned off again,

**Then** both orders become locked immediately.

### Scenario 11.2 — Correcting a done order

**Given** a done, locked order that produced 6,

**When** a user tries to change the produced quantity,

**Then** the field is not editable.

**When** the user unlocks the order and writes 5 as the producing quantity,

**Then** the done finished move's quantity becomes 5.

### Scenario 11.3 — A recipe shared by all companies

**Given** a recipe with **no** company,
**and** a user allowed in company A only,

**Then** the user sees that recipe,
**and** an order in company A may use it.

**Given** an order whose company is B,
**and** the same user,

**Then** the user does not see that order.

### Scenario 11.4 — Company consistency

**Given** an order in company A,

**When** a component move naming a location of company B is added and the order is
confirmed,

**Then** the confirmation fails on the company-consistency check.

---

## 12. Automatic order creation

### Scenario 12.1 — A reordering rule creates an order

**Given** a product with a manufacture route, a `normal` recipe of lead time 3 days and 2
days to prepare, and a reordering rule with a minimum of 10 and a maximum of 50,
**and** an on-hand quantity of 4,

**When** the scheduler runs,

**Then** one order is created for the quantity the rule computes, converted into the recipe
unit,
**and** its start date is the planned date minus 3 days,
**and** its deadline is that start plus 3 days,
**and** its responsible is empty,
**and** it records the reordering rule,
**and** it posts an origin-link note pointing at the rule (or, when the rule was created by
The system with a manual trigger, the note "This production order has been created from
Replenishment Report."),
**and** it is confirmed **after** every reordering rule has run, not during.

### Scenario 12.2 — A procurement is absorbed by an existing order

**Given** a confirmed order for 10 units created by a rule, not planned, with no
responsible, with the same recipe, product, operation type, company and references as a new
procurement for 5 units,

**When** the new procurement runs,

**Then** no new order is created,
**and** the existing order's quantity becomes 15 through the change-quantity assistant,
**and** the new procurement's downstream moves are attached to the existing order's finished
move.

### Scenario 12.3 — Batch sizing

**Given** a recipe with batch sizing enabled and a batch size of 4,

**When** a procurement for 10 units runs,

**Then** three orders are created, each of quantity 4 (the third over-producing by 2),
because the loop emits a batch while the remaining quantity is positive:
`10 → 6 → 2 → −2`.

### Scenario 12.4 — A negative procurement

**Given** a manufacture rule,

**When** a procurement of −3 units reaches it,

**Then** no order is created.

---

## 13. Rounding and unit edge cases

### Scenario 13.1 — Upward rounding of a component quantity

**Given** a recipe producing 100 Litres of *Paint batch* from 2.5 Kilograms of *Pigment*,
**and** the Kilogram has a rounding increment of 1,

**When** an order for 250 Litres is created,

**Then** the explosion multiplier is `250 ÷ 100 = 2.5`,
**and** the Pigment line quantity is `2.5 × 2.5 = 6.25` Kilograms,
**and** the component move's demand is `round_up_at(1, 6.25) = 7` Kilograms.

**When** the Kilogram's rounding increment is 0.01 instead,

**Then** the demand is 6.25 Kilograms.

### Scenario 13.2 — A zero-quantity component line

**Given** a recipe with a component line of quantity 0,

**Then** the line is accepted (the check allows zero),
**and** it produces a component move of demand 0,
**and** it is skipped by the kit-quantity computation and by the kit on-hand derivation, so
no division by zero occurs.

### Scenario 13.3 — A service component

**Given** a recipe with a component line for a service product,

**Then** no component move is created for it,
**and** it is skipped by the kit-quantity computation,
**and** it still contributes to the recipe cost roll-up.

### Scenario 13.4 — A serial-tracked product in a different unit

**Given** a serial-tracked product whose reference unit is *Units*,
**and** an order for 2 *Dozens* of it,

**When** the order is confirmed,

**Then** the quantity becomes `convert(2, Dozens, Units) = 24` and the unit becomes *Units*,
**and** the finished move is converted the same way.

### Scenario 13.5 — The unit factor never divides by zero

**Given** an order whose produced quantity already equals its quantity to produce,

**Then** the denominator of the unit factor is `max(qty − produced, 1) = 1`, so the factor
is the move's demand itself and no division by zero occurs.

---

## 14. Work-in-progress accounting

### Scenario 14.1 — A work-in-progress entry and its reversal

**Given** two open orders at the last day of a month:
order one has consumed three components worth 40.00 each and logged 1 hour at a work centre
costing 60.00 per hour; order two has consumed one component worth 250.00 and logged 30
minutes at the same work centre.

**When** the work-in-progress assistant is opened with both orders selected and the date set
to the last day of the month,

**Then** the component value is `3 × 40.00 + 1 × 250.00 = 370.00`,
**and** the overhead value is `(60 ÷ 60) × 60.00 + (30 ÷ 60) × 60.00 = 90.00`,
**and** the lines are:

| Label | Account | Debit | Credit |
|---|---|---|---|
| `WIP - Component Value` | Stock Valuation | | 370.00 |
| `WIP - Overhead` | Production Work In Progress Overhead | | 90.00 |
| `Manufacturing WIP - ` followed by *the order references* | Production Work In Progress | 460.00 | |

**and** the reversal date defaults to the day after.

**When** the assistant is confirmed,

**Then** the entry is posted,
**and** a reversal referenced "Reversal of: *the reference*" is posted on the reversal date,
**and** both entries record the two orders.

**When** the total credit differs from the total debit,

**Then** confirming is refused with **"Please make sure the total credit amount equals the
total debit amount."**

**When** the reversal date is not after the posting date,

**Then** confirming is refused with **"Reversal date must be after the posting date."**

---

## 15. Reporting

### Scenario 15.1 — The producible quantity on the recipe structure report

**Given** recipe T1 (4 Legs, 1 Table top per table),
**and** free quantities of 17 Legs and 3 Table tops,

**Then** the producible quantity is
`min(floor(17 ÷ 4), floor(3 ÷ 1)) × 1 = min(4, 3) × 1 = 3`,
**and** the top-level status reads "3.0 Ready To Produce".

**When** the free quantities are 2 Legs and 0 Table tops,

**Then** the producible quantity is 0 and the status reads "No Ready To Produce".

### Scenario 15.2 — The recipe cost roll-up

**Given** recipe *Plank cutting* (1 Log at 240.00, one operation of 30 minutes at 60.00 per
hour, a Sawdust by-product at 10 percent),

**When** the price of *Plank set* is computed from the recipe,

**Then** the operations contribute `(30 ÷ 60) × 60.00 = 30.00`,
**and** the components contribute `240.00`,
**and** the total is 270.00,
**and** the standard price becomes `270.00 × 0.9 ÷ 1 = 243.00`.

**When** the price of *Sawdust* is computed from the same recipe (it is a by-product),

**Then** the standard price becomes `270.00 × 10 ÷ 100 ÷ 2 = 13.50` per Kilogram.

### Scenario 15.3 — The order overview readiness label

**Given** a confirmed order for 10 *Dining tables*,
**and** 12 Legs reserved and 0 free, 10 Table tops reserved,

**Then** the Legs producible quantity is
`floor_at_order_unit(10 × (12 + 0) ÷ 40) = floor(3) = 3`,
**and** the Table tops producible quantity is `floor(10 × 10 ÷ 10) = 10`,
**and** the label reads "3.0 Ready".

**When** the Legs reserved and free quantities are both 0,

**Then** the label reads "Not Ready".

### Scenario 15.4 — The cost breakdown of a done order with by-products

**Given** the *Plank cutting* order of scenario 7.1, done,

**Then** the cost breakdown has two lines:

| Product | Unit | Component unit cost | Operation unit cost | Total unit cost |
|---|---|---|---|---|
| Plank set | Units | `240.00 × 0.9 ÷ 1 = 216.00` | `30.00 × 0.9 ÷ 1 = 27.00` | `270.00 × 0.9 ÷ 1 = 243.00` |
| Sawdust | Kilograms | `240.00 × 0.1 ÷ 2 = 12.00` | `30.00 × 0.1 ÷ 2 = 1.50` | `270.00 × 0.1 ÷ 2 = 13.50` |

---

## 16. Regression guards

These scenarios exist to lock behaviours that are easy to get wrong.

### Scenario 16.1 — Moves with no quantity are done, not cancelled

**Given** an order closed without a backorder while a component move has no consumed
quantity,

**Then** that move ends in state `done` with a demand of **0**, not `cancel`, so that a
later edit of the consumed quantity does not fight a cancelled move.

### Scenario 16.2 — Cancelling a flexible order marks it done

**Given** an order whose recipe's consumption policy is *Allowed*,
**and** some components already consumed,

**When** the order is cancelled,

**Then** after the cancellation pass the order is written to **`done`**, not `cancel`.

### Scenario 16.3 — Confirming does not merge moves

**Given** an order with two component moves for the same product coming from two different
recipe lines,

**When** the order is confirmed,

**Then** the two moves remain separate, because confirmation is run with merging disabled.

### Scenario 16.4 — Changing the operation type renumbers the order

**Given** a confirmed order `WH/MO/00007`,

**When** its operation type is changed to another manufacturing type whose sequence code is
`MO2`,

**Then** a new reference is drawn from that type's sequence, for instance `WH/MO2/00001`,
**and** the Stock Reference named `WH/MO/00007` is renamed to the new reference,
**and** every component move is unreserved and re-reserved.

### Scenario 16.5 — A tracked component capped by the supply behind it

**Given** a component whose supplying chain has delivered only 6 of the 10 units demanded,
**and** no sibling move has consumed any,

**When** the producing quantity is set so that the distribution would write 10,

**Then** the written quantity is `min(10, 6 − 0) = 6`.

### Scenario 16.6 — Two overlapping productivity logs

**Given** two logs of the **same** category overlapping between 10:00 and 10:30,

**Then** the overlap is counted once.

**Given** two logs of **different** categories overlapping between 10:00 and 10:30,

**Then** both are counted, because the merge is done per category.

### Scenario 16.7 — A blocking log spanning closed hours

**Given** a work centre working 08:00–17:00,
**and** an availability log from 16:00 on one day to 09:00 on the next,

**Then** its duration counts only the working hours — 1 hour on the first day and 1 hour on
the second, that is 120 minutes — not the 17 wall-clock hours.

**Given** a productive log over the same interval,

**Then** its duration is the full 1020 wall-clock minutes.

### Scenario 16.8 — The serial number of a consumed and unbuilt component

**Given** a serial number consumed by a production and later returned by an unbuild of that
production,

**When** the same serial number is consumed by a new production,

**Then** the uniqueness check passes, because the cancelled quantity (the move out of the
production location with no production order, or with a production order for the same
product) offsets the consumed quantity.

### Scenario 16.9 — Writing down a Work Order's real duration

**Given** a Work Order with logs of 30, 20 and 10 minutes, total 60,

**When** the real duration is written as 25,

**Then** the amount to remove is 35: the first log (30) is removed whole, leaving 5 to
remove; the second log (20) is larger, so its start is moved forward by 5 minutes, leaving
it at 15 minutes,
**and** the remaining logs total `15 + 10 = 25`.

### Scenario 16.10 — The kit description on a delivery document

**Given** the *Desk set* kit delivered as three component moves,

**Then** the document groups the three lines under the kit's display name,
**and** labels them "Desk set - 1/3", "Desk set - 2/3" and "Desk set - 3/3",
**and** the top-level list, asked without a kit filter, omits the lines that belong to a
kit.
