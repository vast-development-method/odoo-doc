# Manufacturing — Workflows

End-to-end operational sequences, step by step: who performs each step, what must be true
before it, what records are created or changed by it, and what it leaves behind.

Roles referred to throughout:

| Role | Meaning |
|---|---|
| Manufacturing user | A member of the manufacturing user group. May create, read, update and delete Manufacturing Orders, Work Orders, Unbuild Orders and Productivity Logs; may only read recipes, operations, work centres and capacities. |
| Manufacturing administrator | A member of the manufacturing administrator group. Additionally creates and edits recipes, operations, work centres, capacities, tags and loss reasons, and is the only role that may close an order whose consumption policy is `strict` with unmatched consumption. |
| Inventory user | A member of the inventory user group. May read Manufacturing Orders and recipes. |
| Subcontractor | An external partner with portal access, restricted by record rules to the subcontracted orders whose subcontractor is that partner's commercial partner. |
| The system | An automatic actor: the scheduler, a rule, or a computed field. |

---

## 1. Designing a recipe

**Actor.** Manufacturing administrator.

**Preconditions.** The finished product exists and is of type goods. Every component
product exists.

**Steps.**

1. Create a Bill of Materials, choosing the product template and, optionally, one specific
   variant. The recipe unit defaults to the template's own unit; the recipe quantity
   defaults to 1.
2. Choose the recipe kind:
   - *Manufacture this product* (`normal`) — the recipe drives Manufacturing Orders;
   - *Kit* (`phantom`) — the recipe is exploded away at the document that references it;
   - *Subcontracting* (`subcontract`, available when the subcontracting capability is
     installed) — the recipe drives an order created automatically by a receipt from a
     named subcontractor.
3. Add component lines: the component, the quantity per recipe output, the line unit, the
   sequence, and, when the recipe has operations, the consuming operation. A line may be
   restricted to particular attribute values of the finished product.
4. Optionally add by-product lines: the by-product, the quantity per recipe output, the
   producing operation and the percentage cost share.
5. Optionally add operations: the name, the work centre, the duration mode (fixed or
   computed), the duration, and, when operation dependencies are enabled on the recipe, the
   predecessors.
6. Set the flexible-consumption policy, the manufacturing readiness mode, the manufacturing
   lead time, the days to prepare, and the batch size if the product must always be
   produced in fixed batches.
7. Save.

**What the system does at each save.**

- It refuses a configuration that would create a cycle between products, naming the
  products in the message.
- It refuses a variant restriction on a recipe that is already bound to a single variant.
- It refuses a by-product equal to the finished product, a negative cost share, and a total
  cost share above 100 for any variant.
- It refuses a kit recipe for a product that has a reordering rule.
- It flags every draft order that uses the recipe, and every confirmed order that uses it
  and produces one of its products, as carrying an outdated recipe.

**Postconditions.** The recipe is selectable by the recipe-selection algorithm and appears
in the recipe structure report.

**Related operations.**

- *Copy existing operations* opens a list of all operations of all active recipes and
  copies the chosen ones into this recipe.
- *Compute days to prepare* recomputes the days-to-prepare field from the structure report.
- *Compute price from recipe* (on the product) writes the rolled-up recipe cost into the
  product's standard price.
- Archiving the recipe archives its operations; archiving the product archives its recipes.

---

## 2. Creating a Manufacturing Order by hand

**Actor.** Manufacturing user.

**Preconditions.** The company has at least one warehouse and therefore at least one
operation type of code `mrp_operation`.

**Steps.**

1. The user opens a new order. The system assigns:
   - the operation type, from the caller's default, or from the recipe, or from the first
     manufacturing operation type of the company's warehouses; when the company has no
     warehouse at all the warehouse-creation warning is raised;
   - the components location and the finished-products location, from the operation type's
     defaults, falling back to the stock location of the company's first warehouse;
   - the start date, either the supplied deadline minus one hour or the current instant;
   - the finish date, either the supplied deadline or the current instant plus one hour;
   - the responsible, the acting user;
   - the locked flag, true unless the user belongs to the unlocked-by-default group.
2. The user chooses the product. The system selects the `normal` recipe for it, sets the
   quantity to the recipe quantity (or 1 with no recipe) and the unit to the recipe unit
   (or the product's own unit).
3. The system generates the component moves from the recipe explosion, the finished move
   for the product, one finished move per applicable by-product, and one Work Order per
   applicable operation of every exploded recipe.
4. The user may adjust the quantity, the unit, the dates, the locations, the responsible,
   the never-variant attribute values, and may add or remove component moves, by-product
   moves and Work Orders by hand.
5. Saving creates the order. The system:
   - allocates the reference from the operation type's sequence and refuses a duplicate
     reference within the company;
   - creates a Production Group named after the reference and stamps it on every move;
   - creates a Stock Reference named after the order, linked to the order and its moves;
   - writes the start date onto the component moves as both date and deadline, and the
     finish date onto the finished moves.

**Postconditions.** The order is in state `draft`. Nothing is reserved, nothing is
procured, and the component and finished moves are in state `draft`.

**Variant — generating a recipe from the order.** When the order has no recipe but has
components or Work Orders, the *generate recipe* control opens a new recipe pre-filled with
the order's components, by-products (with their cost shares) and operations (each operation
taking `duration_expected × ratio` as its manual duration, where the ratio is the reciprocal
of the production quantity, or the order-to-recipe ratio when the Work Order already had an
operation). Saving that recipe links it to the order.

---

## 3. Creating a Manufacturing Order automatically

**Actor.** The system, through a manufacture rule answering a procurement.

**Trigger.** A demand exists at a location the manufacture rule serves: a component move of
another order with make-to-order procurement, a sales-driven delivery of a made-to-order
product, a reordering rule, or a manual replenishment.

**Steps.**

1. **Kit interception.** Before any rule runs, every procurement whose product has a kit
   recipe is replaced by one procurement per exploded leaf component:

   ```formula
   qty_to_produce = convert( procurement_quantity , procurement_unit , kit_recipe_unit , unrounded )
                    ÷ kit_recipe_quantity
   ```

   Each leaf's quantity is adjusted to a sensible procurement unit, and the leaf's recipe
   line is carried in the procurement values.
2. **Rule matching.** The ordinary rule matching of the replenishment domain selects the
   manufacture rule.
3. **Recipe choice.** For each procurement:
   - the recipe supplied in the procurement values wins;
   - otherwise the recipe of the procurement's reordering rule, if it has one;
   - otherwise the `normal` recipe for the product with the rule's operation type and
     company;
   - otherwise the `normal` recipe for the product with **no** operation type restriction.
4. **Negative guard.** A procurement whose quantity is not strictly positive creates
   nothing.
5. **Merging into an existing order.** Unless the procurement's origin is the master
   production schedule, the system looks for an existing order to absorb the demand:
   same recipe, same product, state `draft` or `confirmed`, not planned, same operation
   type, same company, no responsible, exactly the same Stock References, and — when the
   procurement carries a production group — whose production group has that group as a
   parent. When the procurement comes from a reordering rule, the candidate must also
   satisfy: state `draft` with a deadline at or before *(the planned date minus the
   recipe's manufacturing lead time, at the end of that day)*, or state `confirmed` with a
   start date at or before that same instant.

   If a candidate is found and the recipe does not enable batch sizing, the candidate's
   quantity is increased through the change-quantity assistant (§6) by the procured
   quantity converted into the order unit, and the procurement's downstream moves are
   attached to the candidate's finished move.
6. **Creating orders.** Otherwise one or more orders are created. When the recipe enables
   batch sizing, the procured quantity is emitted as repeated orders of the batch size
   until it is covered:

   ```formula
   batch_size = convert( recipe_batch_size , recipe_unit , procurement_unit )
   ```

   and each emitted order carries
   `convert(batch_size, procurement_unit, recipe_unit)` as its quantity. Otherwise a single
   order carries the whole procured quantity.

   Each order is created with:

   | Field | Value |
   |---|---|
   | `origin` | The procurement's origin text. |
   | `product_id`, `product_description_variants`, `never_product_template_attribute_value_ids` | From the procurement. |
   | `product_qty` | The procured quantity converted into the recipe unit, or the raw quantity with no recipe. |
   | `product_uom_id` | The recipe unit, or the procurement unit with no recipe. |
   | `location_src_id` | The operation type's default source. |
   | `location_dest_id` | The operation type's default destination, falling back to the procurement's destination; overridden by the rule's destination when the rule forces it. |
   | `location_final_id` | The procurement's destination. |
   | `bom_id` | The chosen recipe. |
   | `date_start` | The planned date minus the recipe's manufacturing lead time, minus one hour when that subtraction changed nothing. |
   | `date_deadline` | The procurement's deadline, or the planned start plus the manufacturing lead time. |
   | `reference_ids` | The procurement's references. |
   | `propagate_cancel` | The rule's propagate flag. |
   | `orderpoint_id` | The procurement's reordering rule. |
   | `picking_type_id` | The recipe's operation type, or the rule's, or the warehouse's manufacturing type. |
   | `move_dest_ids` | The procurement's downstream moves. |
   | `user_id` | Empty — an automatically created order has no responsible. |

   Orders are created with elevated rights, because the user who triggered the demand (for
   example a salesperson) need not have manufacturing rights.
7. **Automatic confirmation.** An order is confirmed immediately when:
   - it has no component moves and (it has no Work Orders and either it came from a
     reordering rule or its downstream move takes from stock); or
   - it has component moves and did **not** come from a reordering rule.
8. **Chain links and notes.** For each created order:
   - when the procurement carries a production group, that group becomes a **parent** of the
     new order's group;
   - when the order came from a reordering rule created by the system with a manual trigger,
     a note "This production order has been created from Replenishment Report." is posted;
   - when it came from any other reordering rule, an origin-link note pointing at that rule
     is posted;
   - when it came from a component move of another order, an origin-link note pointing at
     that order is posted.

**Postconditions.** One or more orders exist, most of them confirmed, each linked to the
demand that created them so that completing them reserves the demand.

---

## 4. Confirming, reserving and planning

### 4.1 Confirming

**Actor.** Manufacturing user, or the system.

**Preconditions.** The order is `draft`.

**Steps.**

1. Company consistency is checked between the order and every record it references.
2. The consumption policy is copied from the recipe.
3. When the product is serial-tracked and the order unit is not the product's own unit, the
   quantity and unit are converted to the product's own unit, and so is the finished move.
4. The procurement method of every component move is adjusted (a move whose product is
   made to order becomes make-to-order, the rest take from stock).
5. Every component and finished move is confirmed **without merging** — merging would
   destroy the one-move-per-recipe-line correspondence.
6. Every Work Order is confirmed, which re-links the dependency chain and reassigns moves
   to Work Orders (§4.3).
7. Each Work Order's cost mode is copied once from its operation.
8. The scheduler is triggered for the component moves forecast to be short, carrying the
   list of orders already being confirmed so that a cycle cannot occur.
9. Every transfer of the production group that is neither done nor cancelled is confirmed.
10. Only the orders that were `draft` are written to `confirmed`.

**Postconditions.** The order is `confirmed`; its readiness is computed from its component
moves; supplying procurements have been launched for made-to-order components.

### 4.2 Reserving

**Actor.** Manufacturing user (explicit *check availability*), or the system (at
confirmation when the operation type reserves at confirmation, or through the scheduler).

**Steps.** The component moves are reserved by the ordinary reservation algorithm of the
inventory domain, honouring the removal strategy of their source location, the order's
priority (higher priorities are served first) and any lot or package restriction.

**Unreserving.** The *unreserve* control is offered only when no component move is marked
picked and at least one component move line exists. It unreserves every component and
finished move that is neither done nor cancelled and that is not a by-product move.

### 4.3 Linking Work Orders and moves

Run at every confirmation and whenever the Work Order collection changes.

1. Build a mapping from each operation to its Work Order.
2. Copy the recipe's operation-dependency flag onto the order.
3. **With dependencies:** for each Work Order in sequence order, set its predecessors to the
   Work Orders of its operation's predecessors that exist on this order; a Work Order with
   no successor is recorded as the last Work Order of its recipe.
4. **Without dependencies:** for each Work Order in sequence order, set its single
   predecessor to the previous Work Order; the last one is recorded as the last Work Order
   of its recipe.
5. For every component and finished move that names an operation, set its Work Order to the
   Work Order of that operation (or clear it when there is none).

### 4.4 Planning

**Actor.** Manufacturing user (*plan*), or the system (*plan with component availability*).

**Preconditions.** The order is not already planned.

**Steps.**

1. Draft orders in the selection are confirmed first.
2. For each order, the Work Orders and moves are re-linked (§4.3).
3. Every Work Order with no successor is planned, which recursively plans its predecessors
   (the algorithm is in [state-machines.md](state-machines.md) §3.6).
4. The order's start becomes the earliest calendar-slot start and its finish the latest
   slot end, over the Work Orders that are neither done nor cancelled.

**Variant — plan with component availability.** For each order in state `draft` or
`confirmed`: confirm it if it is a draft; take the maximum forecast expected date over the
component moves; when that date exists and the components are not reported unavailable, move
the order's start to it. Then plan every order that is `confirmed`.

**Unplanning.** Refused when any Work Order is done or in progress. Otherwise every calendar
slot is deleted, every Work Order's planned window is cleared, and the order's planned flag
becomes false. Changing the start date of a planned order unplans it automatically unless
the caller forces the date.

---

## 5. Producing and closing

### 5.1 Registering production

**Actor.** Manufacturing user.

**Preconditions.** The order is `confirmed`, `progress` or `to_close`.

**Ways to register.**

| Way | Effect |
|---|---|
| Type a quantity into *Quantity Producing* | Runs the distribution of [calculations.md](calculations.md) §7.2 with manual-consumption moves left alone. |
| Press *Produce All* | Offered when the quantity producing is zero or already the full quantity; sets the producing quantity to the full remaining quantity and closes. |
| Press *Produce* | Offered when the quantity producing is a partial amount; closes at that amount. |
| Edit a component move's consumed quantity | Marks that move as manual-consumption and picked; the value is then never overwritten by the distribution. |
| Start and finish a Work Order | Finishing a Work Order fills the consumed quantities of its own component moves and of the by-product moves of its operation, and marks them picked. |
| Assign serial numbers | Sets the producing quantity to the number of numbers assigned and redistributes. |

The order becomes `progress` as soon as a component move is picked, a Work Order starts or
finishes, or the producing quantity becomes non-zero. It becomes `to_close` when every Work
Order is done or cancelled, or (without Work Orders) when the producing quantity reaches the
quantity to produce.

### 5.2 Assigning lots and serial numbers

**For a lot-tracked finished product.** The *generate* control creates one lot and assigns
it. A second attempt is refused: **"You cannot set more than 1 lot per product"**. Saving
more than one lot is refused: **"You cannot set more than 1 lot"**.

**For a serial-tracked finished product with a quantity of one.** The *generate* control
creates one serial number, assigns it, sets the producing quantity to 1 and redistributes.

**For a serial-tracked finished product with a quantity above one.** The *generate* control
opens the Serial Number Assignment assistant:

1. The first number defaults to the first already-assigned number, or to the next value of
   the product's lot sequence, or to the product's serial prefix followed by its next
   serial.
2. The count defaults to the order's quantity to produce.
3. *Generate* fills the text area with that many names generated from the first one.
4. Duplicate lines are removed automatically whenever the text changes.
5. *Apply* creates the missing lots (reusing existing lots of the same product and company,
   and advancing the product's lot sequence when a created name equals its next value),
   assigns them all to the order, sets the producing quantity to their number and
   redistributes.
6. *Split and assign* instead splits the order into one order of one unit per serial
   number, assigning one number to each.
7. Either way, if the operation type asks for automatic label printing, the label actions
   are returned.

**Number generation.** The next name after a given one is produced by the shared
lot-name-generation rule of the inventory domain: the trailing digit group is incremented,
keeping its width.

### 5.3 Marking the order as done

**Actor.** Manufacturing user (or manufacturing administrator when the consumption policy
is `strict` and consumption does not match).

**The pre-close sequence**, in order:

1. **Sanity checks.** Company consistency, then serial-number uniqueness
   ([business-rules.md](business-rules.md) §6).
2. **Automatic filling decision.** An order is filled automatically when **any** of:
   - nothing among its components and finished products is tracked; or
   - its total quantity is exactly 1; or
   - its product is not serial-tracked **and** its readiness is `assigned`, `confirmed` or
     `waiting`.

   For an order that does **not** qualify and has no producing lot, the system stops and
   opens the serial-number flow; with several such orders it refuses with **"You need to
   generate Lot/Serial Number(s) to mark as done some productions"**.
3. **Automatic filling.** For each qualifying order: generate a lot or serial number when
   the product is tracked and none is set; when the producing quantity is zero, set it to
   `product_qty − qty_produced` and redistribute; mark the by-product moves as produced.
4. For qualifying orders, mark picked every component move that is not a manual-consumption
   move, is not already picked, and whose consumed quantity already equals its demand.
5. For **all** orders, mark picked every manual-consumption component move that is not yet
   picked.
6. For the orders that did **not** qualify, mark the by-product moves as produced.
7. **Consumption check** ([calculations.md](calculations.md) §9.1). Any difference opens the
   Consumption Warning assistant and stops here.
8. **Backorder check.** For each order,
   `quantity_to_backorder = max(product_qty − qty_producing, 0)`; a non-zero value is an
   issue. Then:
   - orders whose operation type forces backordering (`always`) are collected;
   - orders whose operation type asks (`ask`) are collected separately;
   - orders whose operation type never backorders (`never`) are simply closed at the
     produced quantity.

   If any order asks, the Backorder Confirmation assistant opens, carrying the forced list
   in its context. Otherwise, if any order forces, the completion is re-run immediately with
   the backorder question skipped and the forced list supplied.

**The completion itself**, once the questions are answered:

1. Split the orders that are to be backordered ([calculations.md](calculations.md) §8),
   producing the backorders.
2. Finish every Work Order of every order in the selection.
3. Post the inventory of the orders that are not backordered, then of those that are, in
   both cases cancelling the backorder of the underlying moves:
   1. Partition the component moves: those already `done`; those not picked (to be
      cancelled); the rest (to be posted).
   2. Post the to-be-posted component moves; cancel the not-picked ones.
   3. For each order, for each finished move for the order's own product that is neither
      done nor cancelled:
      - when the move is tracked and has no lot yet, assign the order's producing lots, and
        for a lot-tracked product assign the first producing lot to every move line without
        one;
      - set the move's quantity to
        `round_half_up_at(order_unit, (qty_producing − qty_produced) × unit_factor)`, so that
        several finished moves (after a split or a merge) share the produced quantity in
        proportion;
      - apply any extra values the configuration adds to the finished move lines.
   4. For each Work Order: recompute the expected duration when it is not done or cancelled;
      set the real duration of a cancelled Work Order to zero; give a Work Order with a zero
      real duration its expected duration, and set its duration per unit to
      `round_to_2_decimals(duration ÷ max(qty_produced, 1))`.
   5. Compute the production cost for the order and write the unit prices of the finished
      and by-product moves ([calculations.md](calculations.md) §9.3).
   6. Mark every remaining finished move picked and post them.
   7. Link the finished move lines to the consumed component move lines for traceability.
   8. Post the labour journal entry when the configuration requires it
      ([accounting-effects.md](accounting-effects.md) §2.4).
4. Trigger reservation of anything the newly available finished goods can now cover.
5. For the orders that were **not** backordered, write every component and finished move
   that is neither done nor cancelled to state `done` with a demand of zero — deliberately
   not cancelled, so that a later edit of the consumed quantity does not fight a cancelled
   move.
6. For every order in the selection, write the finish date to the current instant, the
   priority to `0`, the locked flag to true and the state to `done`.
7. Reserve the backorders whose operation type reserves at confirmation.
8. Collect the automatic printing actions (§12.3) and decide where the user lands:
   - with no backorder and coming from a Work Order, the order's form;
   - with no backorder and the allocation group, the allocation report when any finished
     move is storable, not cancelled, picked, has no downstream move, and the order's
     operation type shows the report automatically;
   - with one backorder, that backorder's form;
   - with several backorders, their list.

### 5.4 Worked example — producing ten with a partial completion of six and a backorder

The complete numeric walk-through is in [calculations.md](calculations.md) §7.3. In
workflow terms:

1. A manufacturing user confirms an order for 10 Tables (40 Legs, 10 Table tops) and
   reserves the components.
2. The user types 6 into *Quantity Producing*. The system fills 24 Legs and 6 Table tops and
   marks both moves picked. The order becomes `progress`.
3. The user presses *Produce*. The sanity checks pass; the automatic-filling decision
   qualifies the order (nothing is tracked) but changes nothing; the consumption check finds
   no difference; the backorder check finds 4 units unproduced.
4. The operation type asks, so the Backorder Confirmation assistant opens with one line.
   The user ticks *To Backorder* and confirms.
5. The system renames the order `…-001`, sets its quantity to 6, creates the backorder
   `…-002` for 4 in state `confirmed` in the same production group, splits the moves 24/16
   and 6/4 and 6/4, and redistributes the reservations.
6. The system posts the original order: 24 Legs and 6 Table tops consumed, 6 Tables
   produced, the production cost computed, the order `done` and locked.
7. The user lands on the backorder's form, ready to produce the remaining 4.

### 5.5 Cancelling

**Actor.** Manufacturing user.

**Preconditions.** The order is not `done` — otherwise **"You cannot cancel a manufacturing
order that is already done."**

**Steps.**

1. For every component move that is neither done nor cancelled and that has origin moves,
   prepare an exception activity on the supplying documents.
2. Unless the caller suppresses activities, log a downside-quantity activity on the parent
   document of every finished move that is neither done nor cancelled, marked as a
   cancellation.
3. Cancel every Work Order that is neither done nor cancelled (deleting its calendar slot
   and closing its timers).
4. Cancel every component and finished move that is neither done nor cancelled, with the
   order-level cancellation check suppressed so that the cancellation does not recurse.
5. Cancel every transfer of the order that is neither done nor cancelled, that has no
   downstream moves, and none of whose orders is done.
6. Log the prepared exception activities, skipping those whose parent is this order itself
   or a cancelled transfer.
7. Write `done` — not `cancel` — onto any order whose recipe's consumption policy is
   `flexible` and that is still neither done nor cancelled. The reasoning: with a flexible
   recipe the move states alone cannot say whether the order is finished or abandoned, and
   an explicit cancellation means the user wants the order finished one way or the other.

**Postconditions.** The order is `cancel` (or `done` in the flexible case). It can no longer
be confirmed. It may be deleted.

**Related.** Cancelling **every** component move of an order directly, outside the order's
own cancellation, also cancels the order.

### 5.6 Unlocking and correcting a done order

**Actor.** Manufacturing user.

A done order is locked. Pressing *unlock* clears the flag and allows the produced quantity
to be edited; writing the producing quantity then rewrites the quantity of the done finished
move for the order's product. Pressing *lock* restores the flag. The lock control is also
shown for running orders when the user is not in the unlocked-by-default group.

Changing the unlocked-by-default setting immediately rewrites the locked flag of every order
that is neither done nor cancelled.

---

## 6. Changing the quantity to produce

**Actor.** Manufacturing user, or the system (when a procurement is absorbed into an
existing order).

**Preconditions.** The order exists. The assistant does not itself check the state; the
callers do.

**Steps** (the arithmetic is in [calculations.md](calculations.md) §17.2):

1. `factor = new_quantity ÷ old_quantity`.
2. Rescale every component move that is neither done nor cancelled, rounding upward at the
   move's unit, and record the change for the exception log.
3. For every component move whose change matters to the documents supplying it, prepare an exception activity on
   the supplying documents, and log them.
4. Rescale the finished moves: for each, `delta = (new − old) × unit_factor`; a move with
   downstream moves and a non-zero delta is **copied** with just the delta and the copy is
   confirmed, so that the downstream chain is extended; otherwise the move's demand is
   increased by the delta. Then reserve the finished moves.
5. Write the new quantity onto the order.
6. When the producing quantity is non-zero and the order has no Work Orders, set the
   producing quantity to the new quantity and redistribute.
7. For every Work Order: recompute the expected duration with the extra ratio
   `new ÷ old`; set its producing quantity to `qty_production − qty_produced`, or to 1 (or
   0) for a serial-tracked product; move a `done` Work Order back to `progress` when it has
   now produced less than the production quantity, and a `progress` Work Order to `done`
   when it has produced exactly the production quantity; reattach the component and finished
   moves of its operation, with the last Work Order additionally receiving every move that
   names no operation.
8. Trigger the scheduler for the component moves of orders in state `confirmed` or
   `progress`.

---

## 7. Applying an updated recipe to a running order

**Actor.** Manufacturing user.

**Trigger.** The order shows the outdated-recipe flag.

**Steps.**

1. Remember whether the order was planned.
2. Link the recipe (§7.1).
3. If the order was planned, replan it (the Work Orders were recreated unplanned).
4. Clear the outdated-recipe flag.

### 7.1 Linking a recipe to an order

**Draft, done or cancelled orders.** Simply assign the recipe. For a draft order, the
component moves and Work Orders are deleted first (after the new ones have been generated by
the computed fields) and the quantity and unit are restored to what they were, because
assigning a recipe would otherwise reset them.

**Running orders (confirmed, in progress, to close).**

1. `ratio = recipe_quantity_in_product_unit ÷ order_total_quantity`.
2. Explode the recipe for the recipe's own quantity and index the resulting leaf lines by
   *(line, component)*, accumulating `exploded_quantity ÷ original_quantity` per key. Lines
   whose variant restriction does not match the order's product are dropped.
3. Index the recipe's by-products and operations, again dropping those whose variant
   restriction does not match.
4. **Operations.** A Work Order that is in progress, done or cancelled keeps its operation,
   which is removed from the index so it is not recreated. Every other Work Order is
   deleted. Then one Work Order is created per remaining indexed operation, in state
   `blocked`, with the operation's name and work centre and the order's unit.
5. **Components.** For each existing component move, find its recipe line: first by exact
   *(line, product)* match, otherwise the first indexed line with the same product. When a
   line is found and anything differs (the move had no line, or the line belongs to another
   recipe, or the operation differs, or the recipe line quantity differs from the move's
   rescaled quantity), rewrite the move's recipe line, product, demand
   (`indexed_quantity ÷ ratio`), unit, operation, Work Order and manual-consumption flag.
   A move with no matching line is queued for removal.
6. Create one component move per remaining indexed line, with the demand
   `indexed_quantity ÷ ratio`.
7. **By-products.** The same matching, by *(by-product line)* then by product. A matched move
   takes the line, the cost share, the demand `line_quantity ÷ ratio` and the line unit. An
   unmatched move is queued for removal. Each remaining indexed by-product produces a new
   finished move.
8. In a two- or three-step warehouse configuration, set the demand of the queued moves to
   zero before cancelling them (so the supplying pick is reduced rather than orphaned), then
   cancel and delete them.
9. Assign the recipe.

---

## 8. Splitting and merging orders

### 8.1 Splitting

**Actor.** Manufacturing user.

**Preconditions.** Every selected order is `draft` or `confirmed` and has a recipe;
otherwise **"Only manufacturing orders in either a draft or confirmed state can be
split."** or **"Only manufacturing orders with a Bill of Materials can be split."**

**Steps.**

1. Selecting one order opens the split assistant directly; selecting several opens the
   multi-split assistant holding one per order.
2. The assistant proposes a maximum batch size (the recipe's batch size when enabled,
   otherwise the whole quantity) and derives the number of splits and the detail lines
   ([calculations.md](calculations.md) §8.6). The user may edit the maximum batch size or the
   detail lines directly; the assistant reports whether the detail quantities still sum to
   the order quantity.
3. Confirming runs the split with the detail quantities as the amounts. Each resulting
   order then receives the responsible and start date of its detail line.
4. In the multi-split assistant, the split order is removed from the list and the assistant
   reopens on the remainder.

### 8.2 Merging

**Actor.** Manufacturing user.

**Preconditions.**

- at least two orders: **"You need at least two production orders to merge them."**;
- every order `draft` or `confirmed` with a recipe (same messages as §8.1 with "merged");
- all orders share the same product **and** the same recipe: **"You can only merge
  manufacturing orders of identical products with same BoM."**;
- no order has an extra component move (one with no recipe line) or an extra by-product move
  (one with no by-product line): **"You can only merge manufacturing orders with no
  additional components or by-products."**;
- all orders share one state: **"You can only merge manufacturing with the same state."**;
- all orders share one operation type: **"You can only merge manufacturing with the same
  operation type"**;
- with the subcontracting capability, no order is a subcontracted order: **"Subcontracted
  manufacturing orders cannot be merged."**

**Steps.**

1. Collect the origin links per recipe line and the downstream links per by-product line.
2. Create one new order with the shared product and recipe, the shared operation type, the
   **sum of the total quantities** as its quantity, the product's own unit, the shared final
   location when every order has one and they agree, the shared responsible when all orders
   agree (otherwise the acting user), the union of the Stock References, and an origin equal
   to the sorted, comma-separated list of the merged orders' names.
3. Re-stamp every Stock Move of the merged orders' production groups onto the new order's
   production group, so the transfers follow.
4. Copy the merged groups' parents and children onto the new group.
5. Restore the origin links on the new order's component moves and the downstream links on
   its finished moves.
6. Point every downstream move of the merged orders at the new order.
7. When any merged order was `confirmed`, adjust the procurement method of the new order's
   component moves, write `confirmed` onto its moves, and confirm it.
8. Cancel the merged orders with activities suppressed, detach them from their groups, and
   delete any group left with no orders.
9. Push the new order's start date onto the deadline of the origin moves feeding its
   components.
10. Post on every merged order "This production has been merge in *the new order*".

---

## 9. Unbuilding

**Actor.** Manufacturing user.

**Two entry points.**

- From a done order: the *unbuild* control opens a simplified form pre-filled with the
  order's product, its first producing lot, the order itself, the company, the order's
  finished-products location as the source and its components location as the destination.
- From the Unbuild Orders list: the user chooses the product (and optionally an order); the
  recipe, quantity, unit and locations are derived.

**Steps.**

1. The user sets the quantity to unbuild and, for a tracked product, the lot or serial
   number.
2. Pressing *unbuild* first validates the stock: the available quantity of the product at
   the source location for that lot, compared strictly, must be at least the quantity to
   unbuild converted into the product's own unit. When it is not, the insufficient-quantity
   warning opens, titled "*the product*: Insufficient Quantity To Unbuild"; confirming it
   proceeds anyway.
3. The unbuild algorithm runs ([calculations.md](calculations.md) §10.2). In outline:
   - consume moves are created for the finished product and every by-product, from the
     source location back into the production location;
   - produce moves are created for every component, from the production location to the
     destination location;
   - lots are matched back to the original consumption, excluding those already returned by
     an earlier unbuild of the same order;
   - everything is marked picked and posted;
   - traceability links are written;
   - a note is posted on the source order.
4. The Unbuild Order's state becomes `done`. It can no longer be deleted.

**Refusals.**

| Condition | Message |
|---|---|
| The product is tracked and no lot is set | You should provide a lot number for the final product. |
| The named order is not done | You cannot unbuild a undone manufacturing order. |
| A tracked component or by-product must be restored but no order is named | Please specify a manufacturing order. It will allow us to retrieve the lots/serial numbers of the correct components and/or byproducts. |

**Worked example — unbuilding two units.** See [calculations.md](calculations.md) §10.4:
2 Tables consumed, 8 Legs and 2 Table tops returned, from an order that produced 10.

---

## 10. Scrapping

**Actor.** Manufacturing user.

**From an order.** The *scrap* control opens the scrap form restricted to the products of
the order's unfinished component moves and its done finished moves, with the order and the
company pre-filled. The source location is the order's components location while the order
is not done, and its finished-products location afterwards.

**From a Work Order.** The same, with the Work Order recorded as well and the source
location taken from the Work Order's order's components location.

**Effects.**

- The scrap move carries the order as a **finished** order when the scrapped product is one
  of the order's finished products, and as a **component** order otherwise. Either way the
  move's origin defaults to the order's name.
- Scrapping a kit product explodes the scrap move into component moves; the scrapped
  quantity is then derived back from the component moves by the kit-quantity computation.
- Scrapping a serial-tracked component of an order unmarks as picked the component move
  lines that carry that serial number, so the operator must pick a replacement.
- Replenishing after a scrap carries the order's production group, so the replacement lands
  in the same chain.
- Scrapping a serial number warns when the number is not where it is expected, and may
  propose the recommended location.

---

## 11. Subcontracting

Subcontracting is manufacturing performed by an external partner. The finished product is
**received** rather than produced in house, and the corresponding Manufacturing Order is
created and closed automatically by that receipt.

### 11.1 Configuration

**Actor.** Manufacturing administrator.

1. Enable the subcontracting capability.
2. The system creates, per company, an internal location named **Subcontracting**, and sets
   it as the company-level default subcontractor location for partners.
3. Each warehouse gains:
   - a *Resupply Subcontractors* flag, default true;
   - an operation type **Subcontracting** of code `mrp_operation`, sequence code `SBC`,
     with component-lot creation allowed, inactive by default, whose default source is the
     subcontracting location and whose default destination is the production location;
   - an operation type **Resupply Subcontractor** of code `internal`, sequence code `RES`,
     with existing lots allowed and new lots forbidden, label printing on, whose default
     source is the stock location and whose default destination is the subcontracting
     location;
   - a route *Resupply Subcontractor* containing a pull rule from the stock location to the
     subcontracting location through that resupply operation type;
   - a make-to-order rule from stock to the subcontracting location on the ordinary
     replenish-on-order route;
   - a rule from the subcontracting location to the production location on the global route
     *Resupply Subcontractor on Order*.
4. A partner may be given its **own** subcontractor location (a company-dependent property);
   when it has none, the company's subcontracting location is used.
5. A recipe of kind **Subcontracting** is created for the product, listing the components
   the subcontractor consumes and naming the subcontractors. Such a recipe may have **no**
   operations and **no** by-products: **"You can not set a Bill of Material with operations
   or by-product line as subcontracting."**

### 11.2 Finding the subcontracting recipe for a receipt move

The ordinary recipe-selection domain is used with kind `subcontract`, plus the condition
that the receipt's partner is the subcontractor or a descendant of one of the recipe's
subcontractors. The first match by sequence, variant and identifier wins. With no partner,
nothing matches.

### 11.3 The receipt-driven flow

**Trigger.** A receipt move is confirmed whose source location has supplier usage, whose
destination does not, that has no production behind it yet, and for which a subcontracting
recipe is found for the receipt's partner.

**Steps.**

1. The move is marked as a subcontract receipt, its production group is cleared, and its
   **source location is rewritten** to the partner's subcontractor location (or the
   company's). It is then re-reserved, because rewriting the location broke the link.
2. The move is confirmed by the ordinary machinery.
3. For the transfer, one Manufacturing Order is prepared per subcontract move:

   | Field | Value |
   |---|---|
   | `company_id` | The move's company. |
   | `subcontractor_id` | The receipt partner's commercial partner. |
   | `picking_ids` | The receipt transfer. |
   | `product_id`, `product_uom_id` | From the move. |
   | `bom_id` | The subcontracting recipe. |
   | `location_src_id`, `location_dest_id` | Both the subcontractor location. |
   | `product_qty` | The move's demand, or its done quantity when the demand is zero. |
   | `picking_type_id` | The warehouse's subcontracting operation type. |
   | `date_start` | The move's date minus the recipe's manufacturing lead time. |
   | `origin` | The transfer's name. |
   | `reference_ids` | The transfer's references, creating one named after the transfer when it has none. |

   The warehouse is the move's warehouse, falling back to the operation type's warehouse,
   falling back to the warehouse of the downstream move's operation type.
4. The orders are created and confirmed (with procurement suppressed when the receipt is a
   backorder and backordering is not being cancelled), their finish date is set to the
   move's date, their finished move for the product is pointed at the receipt move, and they
   are reserved.
5. **When the receipt move already has a production behind it** the flow differs:
   - if the move has more than one downstream move (the backorder case), the existing order
     is split so that the backorder receipt gets its own order with its reservations
     preserved, and the new order's finished move is pointed at the backorder move;
   - otherwise nothing is created — the existing order's quantity was merely updated.
6. A subcontract receipt move **always bypasses reservation** and is excluded from the
   ordinary available-move-line computation: the goods are at the subcontractor, not in the
   warehouse.

### 11.4 Keeping the receipt and the orders in step

Whenever the receipt move's quantities change:

- **Untracked product.** There is exactly one order. Its quantity is changed, through the
  change-quantity assistant, to the move's done quantity (or its demand when the done
  quantity is zero), and it is re-reserved.
- **Tracked product.** There is one order per lot on the move.
  1. Group the move's lines by lot and sum the quantities in the product's own unit.
  2. For each lot that already has an order whose quantity differs, change that order's
     quantity; for each lot with no order, remember it.
  3. Create the missing orders by **splitting** an existing subcontracting order, so that the
     component reservations are preserved, with the remainder cancelled, and assign one lot
     to each new order.
  4. Delete the orders whose lot no longer appears on the move — but always keep at least
     one order (clearing its lot instead), so that a further split always has something to
     split from.
  5. Re-reserve the affected orders.

Writing a new date on the receipt move writes it as both the start and the finish of the
orders behind it (with the start additionally shifted back by each order's recipe lead
time).

### 11.5 Recording the production

**Actor.** Subcontractor (through the portal) or manufacturing user.

The receipt's *record components* control opens the subcontracting order form. The
subcontractor sees a restricted form and may write only: the component move lines, the
producing lots, the producing quantity and the quantity to produce. Any other field is
refused with **"You cannot write on fields *the field list* in mrp.production."**

A portal user may not create a Stock Move in state `done`, nor set one to `done`:
**"Portal users cannot create a stock move with a state 'Done' or change the current state
to 'Done'."**

Component move lines edited on the portal form are pushed back onto the component moves by
product; a line for a product with no component move creates an **additional** component
move for the summed quantity.

*Split* on the subcontracting order requires a lot to be set first — **"Please set a
lot/serial for the currently opened subcontracting MO first."** — and refuses when the goods
are already received: **"The subcontracted goods have already been received."** It adds an
empty-lot line of one unit to the receipt move when every line already has a lot, and then
reopens the order list filtered on the empty lot.

### 11.6 Validating the receipt

Validating the receipt transfer:

1. Posts the receipt moves by the ordinary machinery.
2. Marks every subcontracting order behind the transfer as done — with the consumption check
   **skipped**, because the subcontractor's declared consumption is authoritative.
3. Backdates every component and finished move of those orders, and their move lines, to one
   second before the earliest receipt move line date, so that the traceability report and
   the product-moves list show the production before the receipt.

### 11.7 Resupplying the subcontractor

Components can reach the subcontractor in two ways.

- **On order.** The component is given the route *Resupply Subcontractor on Order*.
  Confirming the subcontracting order runs the make-to-order chain: a rule pulls the
  component from stock to the subcontracting location through the *Resupply Subcontractor*
  operation type, producing an internal transfer whose destination is the partner's
  subcontractor location and whose partner is the subcontractor. The component move of the
  subcontracting order carries the subcontractor as its partner.
- **Manually.** The user creates a resupply transfer directly.

A component that is **not** resupplied is assumed to be owned and provided by the
subcontractor; it still appears on the subcontracting order and is still consumed there, but
no transfer exists for it.

### 11.8 Worked example — a subcontracted product with resupplied components

Configuration:

- Product *Assembled panel*, bought from subcontractor *Ply Works* at 27.00 per unit,
  standard costing.
- A subcontracting recipe for *Assembled panel* producing 1 unit from 4 *Screws* and
  1 *Raw panel*, with *Ply Works* as the subcontractor, a manufacturing lead time of 1 day.
- Both components carry the route *Resupply Subcontractor on Order*.
- The company's subcontracting location is used (*Ply Works* has no specific location).

Flow for a receipt of **10 Assembled panels**:

1. A purchase order for 10 *Assembled panels* at 27.00 is confirmed, creating a receipt
   transfer with a move of 10 units from the vendor location to stock, with partner
   *Ply Works*.
2. Confirming the move finds the subcontracting recipe for that partner. The move is marked
   as a subcontract receipt and its **source location becomes the subcontracting location**.
3. One Manufacturing Order is created: product *Assembled panel*, quantity 10, recipe the
   subcontracting recipe, both locations the subcontracting location, operation type
   *Subcontracting*, subcontractor *Ply Works*, start date the receipt date minus one day,
   origin the receipt's name. It is confirmed and reserved.
4. Because both components are resupplied on order, confirming the order creates an internal
   transfer *Resupply Subcontractor* moving 40 Screws and 10 Raw panels from stock to the
   subcontracting location, addressed to *Ply Works*.
5. The warehouse ships that transfer and validates it. The 40 Screws and 10 Raw panels are
   now held at the subcontracting location — still owned by the company, still valued in its
   inventory.
6. *Ply Works* logs into the portal, opens the subcontracting order and records the
   components consumed (or the company records them). The order's component moves are
   reserved from the subcontracting location.
7. The warehouse receives the 10 panels and validates the receipt. The system marks the
   subcontracting order done with the consumption check skipped, backdates its moves, and
   posts:
   - 40 Screws and 10 Raw panels out of the subcontracting location into the production
     location;
   - 10 Assembled panels out of the production location into the subcontracting location;
   - 10 Assembled panels out of the subcontracting location into stock (the receipt move
     itself).
8. **Cost.** Before the ordinary production cost is computed, the subcontracting cost
   override runs: the last done receipt move behind the finished move is a subcontract
   receipt, so

   ```formula
   extra_unit_cost = ( value_taken_from_the_vendor_bill + value_taken_from_the_purchase_order )
                     ÷ received_quantity
   ```

   and, when neither the bill nor the order yields a value, `extra_unit_cost` is simply the
   receipt move's unit price. Here, with no bill yet, the purchase order supplies
   `10 × 27.00 = 270.00`, so `extra_unit_cost = 27.00`.

   The ordinary formula then gives

   ```formula
   total_cost = component_value + work_centre_cost + extra_unit_cost × quantity
              = ( 40 screws + 10 raw panels at their own value ) + 0 + 27.00 × 10
   ```

   With screws at 0.50 and raw panels at 12.00, the component value is
   `40 × 0.50 + 10 × 12.00 = 20.00 + 120.00 = 140.00`, so
   `total_cost = 140.00 + 0 + 270.00 = 410.00` and the finished unit price is
   `410.00 ÷ 10 = 41.00`.
9. **Standard-cost variant.** When *Assembled panel* uses standard costing, the finished
   moves take the product's standard price instead, and the difference between the bill and
   the standard price is posted as a price difference; the components' value is added to the
   price-difference base so that the subcontracting service and the component consumption
   are both accounted for.
10. **Recipe cost roll-up.** Computing the product's price from its subcontracting recipe
    adds the subcontractor's price on top of the component and operation costs: the seller
    is selected for the recipe quantity, the recipe unit and the recipe's subcontractors,
    its price is converted into the company currency at today's rate, converted from the
    seller's unit into the product's own unit, and added.

### 11.9 Lead time for a subcontracted purchase

When a buy rule and a subcontracting recipe both apply:

```formula
subcontracting_delay = max( vendor_lead_time ,
                            manufacturing_lead_time + days_to_prepare )
                       + days_to_purchase
```

Concretely: when the vendor's lead time is greater than or equal to
`produce_delay + days_to_prepare_mo`, the vendor's lead time is added and explained as
("Receipt Date", the vendor lead time) and ("Vendor Lead Time", "+ *n* day(s)"). Otherwise
the recipe's manufacturing lead time and days to prepare are added instead and explained as
("Receipt Date", the manufacturing lead time), ("Manufacturing Lead Time", "+ *n* day(s)"),
("Production Start Date", the days to prepare) and ("Days to Supply Components",
"+ *n* day(s)"). Both branches add the purchase-side delays computed with the vendor lead
time ignored.

### 11.10 Other subcontracting variants

- **Drop shipping.** The subcontractor ships directly to the customer: the company gains a
  subcontracting-and-drop-shipping location set, the drop-ship operation type is used for
  the outgoing side, and the receipt-and-delivery pair is replaced by a single chain from
  the subcontractor to the customer.
- **Repair.** A subcontracted product may be repaired; the repair consumes and produces at
  the subcontracting location.
- **Landed costs.** A landed cost that targets a subcontract receipt move is redirected onto
  the moves behind it — that is, onto the subcontracting order's finished move — so the extra
  cost lands on the production rather than on the receipt.

---

## 12. Reporting, printing and notification

### 12.1 The recipe structure report

**Actor.** Any manufacturing user.

Opened from a recipe. The user chooses a quantity, a variant and a warehouse; the report
shows, recursively, every component with its quantity, cost, availability and resupply
route, every operation with its duration and cost, and every by-product with its allocated
cost. It offers three modes of unfolding and a printable version. The algorithms are in
[calculations.md](calculations.md) §15.

### 12.2 The order overview report

Opened from an order. It shows, for every component, operation and by-product, three cost
columns — the order's own cost, the recipe's cost and the real cost — plus the replenishment
documents behind every component, the availability figures, and, for a done order with
by-products, a unit-cost breakdown per output product. The algorithms are in
[calculations.md](calculations.md) §16.

### 12.3 Automatic printing on completion

When an order is marked done, the following are printed automatically according to its
operation type:

| Operation type flag | What is printed |
|---|---|
| Print the production order automatically | The production order document, for every such order. |
| Print the finished product labels automatically | The product-label document (portable-document format) or the label-printer document, according to the chosen format, for every such order. |
| Print the allocation report automatically (with the allocation group) | The allocation report, for every order of code `mrp_operation` whose finished moves have downstream moves. |
| Print the allocation labels automatically (with the allocation group) | The transfer-label document for the downstream moves, with one label per whole unit (the quantity is rounded up). |
| Print the produced lot labels automatically (with the lot group) | The lot-label document (portable-document format or label-printer format) for the lots of the finished move lines. |

When a lot or serial number is generated and the operation type asks for it, the lot label
is printed immediately in the configured format.

### 12.4 Exception activities

- **Cancelling** an order or **reducing** the quantity of a component move logs an
  exception activity on the supplying documents that were supplying that component, naming
  the order and the quantities.
- **Cancelling** a finished move logs a downside-quantity activity on the downstream
  documents that were waiting for it.
- **Cancelling** a downstream move of a running order logs an exception on that order.
- All of these are rendered from the shared manufacturing exception template.

### 12.5 Tracked changes

The order's discussion thread logs every change to the quantity to produce, the state and
the readiness. Each state change posts under its own subtype, so followers can subscribe to
just the transitions they care about:

| State reached | Subtype |
|---|---|
| `confirmed` | Confirmed |
| `progress` | In Progress |
| `to_close` | To Close |
| `done` | Done |
| `cancel` | Cancelled |

Editing the lot, source location or quantity of a **done** move line of an order posts a
tracked message on the order naming the change.

---

## 13. Multi-step warehouse configurations

### 13.1 One step

The order's components location is the warehouse's stock location and its finished-products
location is the stock location too. Components are consumed directly from stock; finished
goods appear directly in stock. No additional transfer exists.

### 13.2 Two steps — pick components then manufacture

1. The warehouse's pre-production location is activated.
2. The manufacturing operation type's default source becomes the pre-production location;
   its default destination stays the stock location.
3. A route *Pick components and then manufacture* is created, containing a pull rule from
   the stock location to the pre-production location through the *Pick Components*
   operation type, and a pull rule from the pre-production location to the production
   location through the manufacturing operation type.
4. A make-to-order rule from stock to the pre-production location is activated on the
   replenish-on-order route.

**Effect on the flow.** Confirming an order launches the pick rule, producing a *Pick
Components* transfer that moves the components from stock to the pre-production location.
The order's component moves are then reserved from the pre-production location and are in
state `waiting` until that transfer is validated.

### 13.3 Three steps — pick components, manufacture, then store products

Everything of the two-step configuration, plus:

1. The warehouse's post-production location is activated.
2. The manufacturing operation type's default destination becomes the post-production
   location.
3. The route gains a **push** rule from the post-production location to the stock location
   through the *Store Finished Product* operation type.
4. The manufacture rule is created with cancellation propagation enabled.

**Effect on the flow.** Closing an order puts the finished goods in the post-production
location and pushes them onward: a *Store Finished Product* transfer moves them into stock.
A finished move created in this configuration records the push rule's destination as its
final location.

### 13.4 Switching configuration

Writing the manufacturing step configuration on a warehouse activates or deactivates the
pre-production and post-production locations, rewrites the operation types' default
locations, and activates or deactivates the corresponding routes, rules and operation types.
Turning *Manufacture to Resupply* off deactivates the manufacture rule, the make-to-order
rules, the manufacturing operation type and the auxiliary operation types, and removes the
warehouse from the global manufacture route.
