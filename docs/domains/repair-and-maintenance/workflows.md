# Workflows

Every operational procedure of the repair and maintenance domain, end to end, with actors, preconditions, numbered steps, branches, the records written at each step and the postconditions.

Throughout this file, "the repair" means a Repair Order record and "the request" means a Maintenance Request record. Rule references of the form `RM-nnn` point at [business-rules.md](business-rules.md). Calculation references point at [calculations.md](calculations.md). Transition references of the form `T-nn` point at [state-machines.md](state-machines.md), which holds the complete transition tables, guards and diagrams for every state field named here.

---

# Part one: repair workflows

## Workflow 1: Create a Repair Order by hand

**Actor.** Repair Technician (Inventory User).

**Preconditions.** At least one operation type whose code is `repair_operation` exists for the acting user's company. The user's default warehouse, or some warehouse of the company, owns such a type.

**Steps.**

1. The user opens a new Repair Order form. The following defaults are applied, in this order:
   1. The company (`company_id`) is the acting user's active company.
   2. The operation type (`picking_type_id`) is resolved: the repair operation type of the responsible user's default warehouse in that company; when that warehouse has none, the first repair operation type belonging to a warehouse of that company. When the screen context supplied an operation type, that value wins.
   3. The component source location (`location_id`), the product source location (`product_location_src_id`), the product destination location (`product_location_dest_id`), the added-parts destination location (`location_dest_id`), the removed-parts destination location (`parts_location_id`) and the recycled-parts destination location (`recycle_location_id`) are each taken from the corresponding default of the operation type.
   4. The scheduled date (`schedule_date`) is the current moment.
   5. The responsible user (`user_id`) is the acting user.
   6. The state (`state`) is `draft`, the priority (`priority`) is `0`, the product quantity (`product_qty`) is `1.0`, the warranty flag (`under_warranty`) is false.
   7. The reference (`name`) is the literal text `New`.
   8. When the screen context carries a repair transfer identifier or a repair lot identifier, those become the return transfer (`picking_id`) and the lot (`lot_id`) respectively (see workflows 2 and 3).
2. The user chooses the customer (`partner_id`), the product to repair (`product_id`), and, when the product is tracked, the lot or serial number. Choosing the product recomputes the quantity, the unit of measure and the lot as described in [entities.md](entities.md), section 1.6.
3. The user optionally ticks the warranty flag, sets the scheduled date, the responsible user, the tags, the priority, the repair properties and the internal notes.
4. The user adds parts (workflow 5).
5. The user saves.

**Records written on save.**

| Record | Fields written |
|---|---|
| Stock Reference | Its name is the reference the repair is about to receive. One new record. |
| Repair Order | The reference is the next number of the operation type's numbering sequence, unless the user supplied a reference other than `New`; the references collection holds the new reference record; every value entered by the user; every derived location. |
| Stock Move, one per part line | The repair link (`repair_id`), the part kind (`repair_line_type`), the product, the demanded quantity, the unit, the origin text and the document reference both equal to the repair's reference, the repair's operation type, the repair's Stock References, the source and destination locations derived from the part kind, and the `draft` move state. |

**Postconditions.** The repair exists in state `draft` with a permanent reference (transition T-01). Nothing has been reserved and no stock has moved.

**Branches.**

- When the company owns only one repair operation type, the operation type field is hidden on the form.
- When the warehouse of the component source location differs from the warehouse of the linked transfer's destination location, and both warehouses are known, the form shows the non-blocking warning titled "Warning" carrying the message "Note that the warehouses of the return and repair locations don't match!" (`RM-063`). The user may proceed.

---

## Workflow 2: Create a Repair Order from a return transfer

**Actor.** Repair Technician.

**Preconditions.** A delivery to the customer was validated, a return of that delivery was created and validated, and the returned goods are back in an internal location.

**Steps.**

1. The user opens the validated return transfer and triggers Create Repair. The action is also available as a contextual action on the transfer form.
2. The system builds a new Repair Order form whose context carries:
   - the transfer as the repair's return transfer,
   - the repair operation type of the warehouse of the transfer's own operation type, falling back to the repair operation type of the acting user's default warehouse,
   - the transfer's contact as the repair's customer.
3. The new-record defaults of workflow 1 are applied, then the context values are applied on top. In particular the return transfer is set, which derives:
   - the customer from the transfer's contact,
   - the transfer products from the products of the transfer's moves, which restricts the choice of product to repair,
   - the allowed lots from the lots of the transfer's moves,
   - the lot automatically when the transfer moved exactly one lot in total,
   - the product quantity from the transfer's recorded quantities once the product, and the lot when the product is tracked, are known (calculation 13).
4. The user chooses the product to repair among the products of the transfer, adds the parts, and saves.

**Postconditions.** The repair exists in `draft`, bound to the transfer. The transfer's repair counter increases by one and its Repair Orders button appears. The parts added to the repair are **not** added to the return transfer; the two documents keep separate movement sets.

**Branch.** When the repair's component source location belongs to a different warehouse from the transfer's destination location, the warehouse-mismatch warning of `RM-063` is shown. It is informational only. When the transfer's destination location belongs to no warehouse at all — the customer location, for instance — no warning is raised, because the condition requires a warehouse on both sides.

---

## Workflow 3: Create a Repair Order from a lot or serial number

**Actor.** Repair Technician holding the lot and serial number tracking group.

**Preconditions.** A Lot or Serial Number record exists.

**Steps.**

1. The user opens the lot record and triggers the action that lists its repairs.
2. The system opens the Repair Orders screen filtered on Repair Orders whose lot is this lot, with a context that pre-fills a new order with this lot's product, this lot, and this lot's company, falling back to the current company when the lot has none.
3. The user creates a new order from that screen. The company defaults from the context, which is what allows a repair to be raised for a serial number registered in another company.
4. The user completes and saves the order as in workflow 1.

**Postconditions.** The repair exists, naming the lot. The lot's in-repair counter increases by one; when the repair completes, the repaired counter increases instead and the in-repair counter falls back.

---

## Workflow 4: Create Repair Orders by confirming a Sales Order

**Actor.** The salesperson confirms; the system creates the repairs.

**Preconditions.** A product exists that is a service whose service tracking is `repair`. A Sales Order quotes that product with a positive quantity. The warehouse of the Sales Order owns a repair operation type.

**Steps.**

1. The salesperson confirms the Sales Order.
2. The ordinary confirmation of the Sales Order runs first.
3. For each line of the order, in order:
   1. When a Repair Order already exists that is bound to this exact line and the line's quantity is strictly positive, that repair is reopened instead of a new one being created: every bound repair in state `cancel` is set back to `draft` (T-10) and then confirmed (T-03). The loop continues with the next line.
   2. Otherwise the line is skipped when any of the following holds: its product's service tracking is not `repair`; its moves already belong to a Repair Order; its quantity is zero or negative.
   3. Otherwise a Repair Order is prepared with state `confirmed`, the order's customer, the order as Sales Order, the line as originating Sales Order Line, and the operation type of the order's warehouse.
4. All prepared Repair Orders are created in one operation, with elevated rights, which applies the creation rules of workflow 1 — numbering, reference record, location derivation — but leaves the state at `confirmed` (T-02).

**Records written.** One Repair Order per qualifying line, plus one Stock Reference per repair.

**Postconditions.** The Sales Order shows a Repairs counter. Each repair shows the originating Sales Order, the originating line and the line's description as the repair request text.

**Important quantity rule.** The quantity on the Sales Order Line does **not** multiply the number of repairs. A line for three units of the repair service produces exactly one Repair Order (`RM-045`).

**Branches.**

- Setting the line quantity to zero or less cancels the bound repair (workflow 15), which also writes zero back onto the line.
- Raising the quantity from zero or less back to a positive value returns the bound repair to `draft` and then confirms it.
- Cancelling the whole Sales Order cancels every bound repair that is not already completed.
- A line that is a section or a note carries no product and is untouched by all of this.

---

## Workflow 5: Add, change and remove parts

**Actor.** Repair Technician.

**Preconditions.** The repair exists and is neither cancelled nor completed.

**Steps for adding a part.**

1. The user adds a line in the Parts list and chooses the part kind: `add`, `remove` or `recycle`. The kind is mandatory.
2. The user chooses the product, the demanded quantity, the unit of measure, and, for serial-tracked products, the lots.
3. On save, the move is created with:
   - the repair link and the chosen part kind,
   - the origin text set to the repair's reference,
   - the operation type taken from the repair,
   - the document reference set to the repair's reference,
   - the repair's Stock References linked onto it,
   - the source and destination locations derived from the part kind:

     | Part kind | Source location | Destination location |
     |---|---|---|
     | `add` | the repair's component source location | the repair's added-parts destination location |
     | `remove` | the repair's added-parts destination location | the repair's removed-parts destination location |
     | `recycle` | the repair's added-parts destination location | the repair's recycled-parts destination location |

4. When the repair is already `confirmed` or `under_repair` and the new move is still in `draft`, the move is immediately brought into line with the repair (T-22): its company is checked, its procurement method is adjusted against rules whose operation type carries the repair code, it is confirmed, and the replenishment scheduler is triggered for it (`RM-014`). This is what lets a technician add a part to a repair that is already running and have it reserved or replenished at once.
5. When the repair already has a Sales Order and the new part kind is `add`, a Sales Order Line is created for the part (workflow 21, step 5).
6. When the manufacturing bridge is installed and the part's product has a kit bill of materials, the move is replaced by one move per kit component (workflow 7).

**Steps for changing a part.**

1. Changing the demanded quantity of an `add` part whose Sales Order Line exists updates that line's ordered quantity to the sum of the demanded quantities of every move attached to it (calculation 7).
2. Changing the part kind from `add` to `remove` or to `recycle` sets the ordered quantity of the attached Sales Order Line to zero; the line is not deleted, because a line of a confirmed order may not be deleted.
3. Changing the part kind back to `add` sets the Sales Order Line quantity back to the move's demanded quantity, or creates a Sales Order Line when none exists yet.
4. Changing any of the repair's own part-driving location fields — the component source, the added-parts destination, the removed-parts destination or the recycled-parts destination — relocates every part move of the repair according to its kind (`RM-061`). Changing the product source or the product destination location relocates nothing.

**Steps for removing a part.**

1. Deleting a part line first cancels the move, which sets the ordered quantity of its Sales Order Line to zero, and then deletes it.
2. A move that is already done cannot be deleted.

**Postconditions.** The repair's part list, its readiness computation and, when a Sales Order exists, the Sales Order Lines are consistent.

**Rule that never reverses.** Quantities flow from the Repair Order to the Sales Order only. Editing the quantity on the Sales Order Line does not change the repair part; the next change made on the repair part overwrites the Sales Order Line again (`RM-043`).

---

## Workflow 6: Pick parts from the product catalog

**Actor.** Repair Technician.

**Preconditions.** The repair exists and is neither cancelled nor completed.

**Steps.**

1. From the Parts list the user opens the catalog.
2. The catalog lists products restricted to goods; services are excluded (`RM-091`). A dedicated filter shows only the products already present on this repair; when the manufacturing bridge is installed, a second filter shows only the components of the bill of materials of the product being repaired.
3. Each catalog card shows the product's list price and, when the product is already on the repair, its current quantity. Stock figures are shown for each product.
4. Setting a quantity on a card:
   - when a part move for that product already exists on the repair and the quantity is not zero, the move's demanded quantity is set to that value;
   - when a part move already exists and the quantity is zero, the move is deleted;
   - when no move exists and the quantity is strictly positive, a move is created with part kind `add`, the repair's component source location as source, the repair's added-parts destination location as destination, and the given quantity.
5. The catalog returns the product's list price for display.

**Postconditions.** Identical to workflow 5. Catalog-created parts are always of kind `add`.

---

## Workflow 7: Explode a kit part

**Actor.** The system, when the manufacturing bridge is installed.

**Preconditions.** A part move of the repair names a product that has a bill of materials of the kit kind in the repair's company.

**Steps.**

1. Immediately after a Repair Order is created, and immediately after any write on a Repair Order, every part move of the repair is examined.
2. For each part move whose product has a kit bill of materials:
   1. The explosion ratio is computed: the move's demanded quantity, converted into the unit of the bill of materials, divided by the quantity the bill of materials produces (calculation 9).
   2. The bill of materials is exploded at that ratio, recursively through nested kits, by the [manufacturing](../manufacturing/) domain.
   3. For every resulting component line whose product is not a service, a new move is prepared with the same repair, the same part kind, the same unit price, the same source and destination locations, the component quantity, and the `draft` move state.
   4. The original kit move is marked for deletion.
3. The marked moves are deleted and the prepared component moves are created. Creating them runs the ordinary part-creation rules of workflow 5, including immediate confirmation when the repair is already running.

**Postconditions.** The repair's Parts list shows the kit's components instead of the kit. Service components of the kit are silently dropped.

**Worked example.** A confirmed repair receives one unit of a kit product whose bill of materials produces one unit from two components. After the write, the repair holds two part moves, one per component, and no move for the kit.

---

## Workflow 8: Confirm a Repair Order

**Actor.** Repair Technician, through the Confirm Repair button.

**Preconditions.** The repair is in state `draft`. The button is shown only in that state.

**Steps.**

1. **Negative quantity guard.** When any part move of the repair has a negative demanded quantity, the operation is refused with the message "You can not enter negative quantities." Nothing is written (`RM-010`).
2. **Product availability check.**
   1. When the repair has no product to repair, or its product is not a storable good, the check is skipped and confirmation proceeds at step 3.
   2. Otherwise the system reads the stock quantity records of the product at the repair's product source location, for the repair's lot, twice: once restricted to the repair's customer as owner, once restricted to records with no owner. Each read is summed.
   3. The repair quantity is converted from the repair's unit of measure into the product's reference unit.
   4. When either sum is greater than or equal to the converted quantity, at the *Product Unit* decimal precision, confirmation proceeds at step 3.
   5. Otherwise the Insufficient Repair Quantity Warning dialogue is opened (workflow 9) and nothing is written yet (`RM-011`, calculation 14).
3. **Confirmation proper.** For every repair of the operated set that is in state `draft` — repairs in any other state are silently left alone (`RM-013`):
   1. The company consistency of the repair and of its part moves is checked (`RM-005`).
   2. The procurement method of every part move is adjusted, searching only rules whose operation type carries the repair code (`RM-012`). A move whose product can be pulled from a compatible make-to-order rule becomes a make-to-order move; otherwise it becomes a make-to-stock move.
   3. Every part move is confirmed. Make-to-stock moves attempt reservation according to the reservation policy of the operation type; make-to-order moves raise the procurement that will supply them.
   4. The replenishment scheduler is triggered for the part moves, which creates the Manufacturing Orders, Purchase Orders or internal transfers that the rules demand.
   5. The repair's state is written to `confirmed` (T-03).

**Records written.** The state of the repair; the state, procurement method and reservations of the part moves; whatever documents the procurement rules create.

**Postconditions.** The repair is `confirmed`. Its readiness fields become meaningful. The operation type's confirmed, ready and late counters update.

---

## Workflow 9: Confirm despite insufficient quantity

**Actor.** Repair Technician.

**Preconditions.** Workflow 8 step 2 found the product to repair insufficient.

**Steps.**

1. The system opens a dialogue titled with the product's display name, a colon, a space and the reproduced text "Insufficient Quantity To Repair", pre-filled with the product, the repair's product source location, the converted repair quantity, the product's reference unit name and the repair.
2. The dialogue lists every stock quantity record of that product in an internal location of the repair's company, with location, lot and quantity, so that the user can see where the product actually is.
3. The dialogue text reads "The product is not available in sufficient quantity in " followed by the location, then "You can still confirm the repair of " followed by the quantity, the unit and " from location " and the location, then " ? , but get ready to have fun taking inventory to fix the negative stock quantity!"
4. **Branch A, the user confirms.** The surrounding context is cleared of defaults that would otherwise pollute the records about to be created, and the confirmation proper of workflow 8 step 3 runs. The repair becomes `confirmed`.
5. **Branch B, the user discards.** Nothing is written. The repair stays in `draft`.

**Note.** This dialogue concerns only the product being repaired. It never concerns the parts; part shortages are reported through the readiness fields and handled through replenishment, not through this dialogue.

---

## Workflow 10: Check availability

**Actor.** Repair Technician, through the Check availability button.

**Preconditions.** The button is shown when the repair is `confirmed` or `under_repair` and at least one part move is unpicked, has a positive demand and sits in the `confirmed` or `partially_available` move state (calculation 11).

**Steps.**

1. Every part move of the repair is submitted to the reservation procedure of the [inventory operations](../inventory-operations/) domain.
2. Each move reserves what it can from its source location, creating or updating its detail lines and the reservations on the stock quantity records (T-24).

**Postconditions.** Moves become fully available, partially available, or stay waiting. The readiness fields recompute.

---

## Workflow 11: Unreserve

**Actor.** Repair Technician, through the Unreserve button.

**Preconditions.** The button is shown when the repair is not in `draft`, `done` or `cancel` and at least one detail line of its parts carries a non-zero reserved quantity.

**Steps.**

1. Every part move in the `assigned` or `partially_available` state has its reservations released (T-25).

**Postconditions.** The reserved quantities return to the stock quantity records. The moves fall back to `confirmed`. The readiness fields recompute.

---

## Workflow 12: Replenish a missing part

**Actor.** The system, driven by the replenishment rules.

**Preconditions.** The repair was confirmed (workflow 8) and at least one part move was adjusted to the make-to-order procurement method, or a reordering rule covers the part's source location.

**Steps.**

1. During confirmation, the procurement method adjustment searches for a rule whose destination is the part move's destination location, whose source is the move's source location or one of its ancestors, and whose operation type carries the repair code. The warehouse's replenish-on-order pull rule for repairs matches this search.
2. When such a rule is found and it is a make-to-order rule, the part move becomes a make-to-order move and, on confirmation, raises a procurement for its product and quantity at its source location, carrying the repair's Stock References.
3. The procurement is resolved by the [replenishment and procurement](../replenishment-and-procurement/) domain into one of:
   - a Manufacturing Order, when the product's route leads to manufacturing. The production's destination move points at the repair's part move. The repair then counts that production in its manufacturing counter and can open it; the production counts the repair in its repair counter and can open it.
   - a Purchase Order line, when the product's route leads to buying. The repair counts the resulting Purchase Orders in its purchase counter and can open them; the Purchase Order counts the repair in its repair counter.
   - an internal transfer, when the route leads to another location of the same warehouse.
4. When instead a reordering rule covers the part's source location, the scheduler trigger fired at confirmation creates an ordinary replenishment move inside a transfer. That move belongs to the transfer, not to the repair: it carries no repair link.

**Postconditions.** The part will arrive. The repair's readiness fields report the expected date until it does.

---

## Workflow 13: Start the repair

**Actor.** Repair Technician, through the Start Repair button.

**Preconditions.** The button is shown when the repair is `confirmed`.

**Steps.**

1. Every repair of the operated set that is not in `confirmed` is submitted to the confirmation proper of workflow 8 step 3, which affects only those still in `draft` (T-04).
2. Every repair of the set is written to state `under_repair` (T-05).

**Postconditions.** The repair is `under_repair`. The technician may now record actual quantities on the parts. The operation type's under-repair counter updates.

**Note.** No stock moves at this step. The state is informational: it separates planned-and-reserved work from work in progress.

---

## Workflow 14: End the repair

**Actor.** Repair Technician, through the End Repair button.

**Preconditions.** The repair is `under_repair`.

**Steps.**

1. **State guard.** When any repair of the operated set is not in `under_repair`, the operation is refused with the message "Repair must be under repair in order to end reparation." (`RM-020`).
2. **Confirmation prompt.** When the incomplete-parts flag is true — that is, when at least one part move has a recorded quantity strictly lower than its demand at the rounding of its unit — the screen first asks: "For some of the parts, there is a difference between the initial demand and the actual quantity that was used. Are you sure you want to confirm ?" Discarding stops here and writes nothing. Accepting continues (`RM-022`).
3. **Cancel empty parts.** Every part move whose recorded quantity is zero at the rounding of its unit is cancelled. Cancelling a move sets the ordered quantity of its Sales Order Line to zero (`RM-023`).
4. **Mark parts picked.** For each repair, when none of its part moves is marked picked, all of them are marked picked. When at least one is already picked, the others are left as they are (`RM-024`).
5. **Report the repair service as delivered.** When the repair was raised by a Sales Order Line and that line's product is a service whose delivery policy is ordered quantities — or whose deployment has no delivery-policy field at all — the line's delivered quantity is set equal to its ordered quantity (`RM-027`).
6. **Guard on the repaired product's lot.** When the repair names a product whose tracking is not `none` and no lot or serial number is chosen, the operation is refused with the message "Serial number is required for product to repair : " followed by the product's display name (`RM-021`).
7. **Choose the owner of the repaired product.** The available quantity of the product at the repair's **component source** location, for the repair's lot, owned by the repair's customer, is read with a strict test. When it is greater than or equal to the repair quantity at the *Product Unit* precision, the customer is used as the owner of the detail line; otherwise no owner is set (`RM-026`, calculation 15).
8. **Create the repaired-product movement.** One Stock Move is created per repair that names a product, with:

   | Field | Value |
   |---|---|
   | Product | the repair's product to repair |
   | Unit | the repair's unit of measure, falling back to the product's reference unit |
   | Demanded quantity | the repair's product quantity |
   | Partner | the repair's customer |
   | Source location | the repair's product source location |
   | Destination location | the repair's product destination location |
   | Picked | true |
   | Transfer | none; the move belongs to no transfer |
   | Repair | the repair |
   | Part kind | none, which is why it does not appear in the repair's Parts list |
   | Origin | the repair's reference |
   | Company | the repair's company |
   | Detail line | one line with the product, the lot, the unit, the repair quantity, no source package, no destination package, the owner chosen at step 7, the same two locations, the company, and a consumed-lines link to every detail line of every part move of the repair |

9. The created move is written onto the repair as its inventory move.
10. **Complete every movement.** All part moves and all repaired-product moves are completed together, with backorder creation suppressed. Because repair moves are never split, a move whose recorded quantity differs from its demand is completed as it stands; no residual move and no backorder are produced (`RM-025`). A part move that step 4 left unmarked, because at least one other part had already been marked by hand, is cancelled here rather than carried forward, since backorder creation is suppressed.
11. The repair's state is written to `done` (T-06).

**Records written.** One Stock Move and one Stock Move Line per repair for the repaired product; the state, recorded quantities and picked flags of every move; the stock quantity records affected by every completed move; the valuation journal entries described in [accounting-effects.md](accounting-effects.md); the delivered quantities of the Sales Order Lines attached to `add` parts.

**Postconditions.** The repair is `done`. The Product Moves button appears. The Sales Order Lines generated from `add` parts now carry a delivered quantity equal to the recorded quantity of their move. The lot's repaired counter increases and its in-repair counter decreases.

**Branch: no product to repair.** A repair with no product still completes; steps 6 to 9 are skipped and only the part moves are completed.

---

## Workflow 15: Cancel a repair

**Actor.** Repair Technician, through the Cancel Repair button; or the system, when a Sales Order is cancelled or a Sales Order Line quantity falls to zero.

**Preconditions.** The repair is not `done`. The button is hidden when the repair is `done` or `cancel`.

**Steps.**

1. **Guard.** When any repair of the operated set is `done`, the operation is refused with the message "You cannot cancel a Repair Order that's already been completed". Nothing is written (`RM-030`).
2. For every repair that has an originating Sales Order, the originating Sales Order Line's ordered quantity is written to zero (`RM-031`).
3. Every part move of the repair is cancelled. Cancelling a move sets the ordered quantity of its own Sales Order Line to zero, which removes the parts from the customer's quotation (`RM-032`).
4. The repair's state is written to `cancel` (T-07, T-08 or T-09 according to the state it came from).

**Postconditions.** The repair is `cancel`. Reservations are released by the move cancellation. The originating Sales Order Line and every part line on the quotation show zero.

---

## Workflow 16: Set a cancelled repair back to New

**Actor.** Repair Technician, through the Set to Draft button, shown only when the repair is `cancel`.

**Steps.**

1. When any repair of the operated set is not `cancel`, workflow 15 is run on the whole set first (T-11). A completed repair therefore makes the whole operation fail with the cancellation guard message.
2. The Sales Order Lines that are attached to this repair's part moves, that belong to a Sales Order which is not itself cancelled, and whose ordered quantity is currently zero at the rounding of the line's unit are collected. Their moves are submitted to the Sales-Order-Line refresh rule, which restores the ordered quantity of every `add` part line to the sum of the demanded quantities of its moves and re-zeroes the quantity of every `remove` and `recycle` part line (`RM-033`).
3. Every part move of the repair is written back to the `draft` move state.
4. The repair's state is written to `draft` (T-10).

**Postconditions.** The repair is `draft` again with its parts intact, and the quotation shows the parts again.

---

## Workflow 17: Delete a repair

**Actor.** Repair Technician.

**Preconditions.** The user holds delete rights on Repair Orders, which the Inventory User group carries.

**Steps.**

1. Before deletion, every repair of the operated set that is not already `cancel` is cancelled through workflow 15. A `done` repair therefore raises "You cannot cancel a Repair Order that's already been completed" and the deletion fails (`RM-034`).
2. The repair records are deleted. Because the part moves point at the repair with a cascading link, they are deleted with it (T-12, T-13).

**Postconditions.** The repair and its part moves no longer exist. Any outgoing quantities the draft moves had claimed are released.

---

## Workflow 18: Change the operation type of an open repair

**Actor.** Repair Technician.

**Preconditions.** The repair is neither `cancel` nor `done`.

**Steps.**

1. The user selects a different operation type and saves.
2. Before the write is applied, for every repair of the set that is neither `cancel` nor `done` and whose current operation type differs from the new one (`RM-002`):
   1. A new reference is drawn from the new operation type's numbering sequence and written onto the repair. The old reference is not kept and is never reused.
   2. All of the repair's part moves are collected for re-reservation.
3. The write is applied. The six location fields recompute from the new type's defaults, and the repair properties are re-based on the new type's property definition.
4. Because location fields changed, every part move is relocated according to its part kind and the repair's new locations.
5. Because the repair's reference changed, every part move's document reference and origin follow.
6. The collected moves are unreserved.
7. Among them, those that are in the `confirmed` or `partially_available` state and that either bypass reservation, or belong to an operation type that reserves at confirmation, or carry a reservation date that is not later than today, are reserved again from their new source location.

**Postconditions.** The repair carries a new reference drawn from the new sequence, its parts sit in the new locations and are reserved from them.

**Worked example.** A repair created under an operation type whose sequence prefix is `PT1/` is numbered `PT1/00001`; its single part move references `PT1/00001` and is sourced from that type's source location with zero reserved, because the part is not there. Switching the repair to an operation type whose prefix is `PT2/`, and whose source location does hold one unit, renumbers the repair to `PT2/00001`, rewrites the move reference to `PT2/00001`, moves the source location and reserves one unit. Switching back to the first type draws the next number of that sequence, `PT1/00002`. Saving the first type a second time, with no change, changes nothing: the reference stays `PT1/00002`.

---

## Workflow 19: Change the scheduled date

**Actor.** Repair Technician.

**Steps.**

1. The user changes the scheduled date and saves.
2. After the write, the date of every part move and of the repaired-product move that is neither `done` nor `cancel` is rewritten to the new scheduled date (`RM-064`).

**Postconditions.** The repair and its moves share one planning date. The lateness computation and the late counter on the operation type reflect the new date.

---

## Workflow 20: Change the warranty flag

**Actor.** Repair Technician.

**Steps.**

1. The user ticks or unticks the warranty flag and saves.
2. After the write, the `add` part moves of the repair that carry a Sales Order Line are collected.
3. **Branch A, the flag is now true.** Each of those Sales Order Lines has its unit price and its stored manual-price marker written to zero. The customer is therefore not charged for any part.
4. **Branch B, the flag is now false.** Each of those Sales Order Lines has its unit price recomputed from the price list, the customer and the quantity, by the [sales](../sales/) domain.

**Postconditions.** The quotation reflects the warranty decision (`RM-044`, calculation 6). Discounts already entered by hand on those lines are untouched by this operation, and so is the price of any line that was not created from an `add` part of this repair.

---

## Workflow 21: Create a quotation from the repair

**Actor.** Repair Technician, through the Create Quotation button, shown when the repair has a customer, is not cancelled and has no Sales Order yet.

**Preconditions.** The repair has a customer.

**Steps.**

1. **Guard, already bound.** When any repair of the operated set already has a Sales Order, the operation is refused with the message "You cannot create a quotation for a repair order that is already linked to an existing sale order." followed by a new line, "Concerned repair order(s):", a new line, and the references of the offending repairs, one per line (`RM-040`).
2. **Guard, no customer.** When any repair of the set has no customer, the operation is refused with the message "You need to define a customer for a repair order in order to create an associated quotation." followed by a new line, "Concerned repair order(s):", a new line, and the references, one per line (`RM-041`).
3. One Sales Order is created per repair, with:

   | Field | Value |
   |---|---|
   | Company | the repair's company |
   | Customer | the repair's customer |
   | Warehouse | the warehouse of the repair's operation type |
   | Repair Orders | the repair |
   | Origin | the repair's reference |

4. The link is made from the Sales Order side, which sets the Sales Order on the repair.
5. Every part move of the repair is submitted to the Sales-Order-Line creation rule. A line is created for each move that has no Sales Order Line yet, whose part kind is `add`, and whose repair now has a Sales Order (`RM-042`). Each line receives:

   | Field | Value |
   |---|---|
   | Order | the repair's Sales Order |
   | Product | the move's product |
   | Ordered quantity | the move's demanded quantity when the repair is not yet `done`, otherwise the move's recorded quantity |
   | Unit | the move's unit |
   | Moves | the move |
   | Delivered quantity | the move's recorded quantity when the move is already `done`, otherwise zero |
   | Unit price | zero when the repair is under warranty; otherwise the move's own unit price when it carries one; otherwise the price the price list computes |

6. The screen opens the created Sales Order.

**Postconditions.** A quotation exists carrying one line per added part. Removed and recycled parts are absent from it. The repair shows a Sale Order button.

**Note.** When the repair was itself raised by a Sales Order (workflow 4), this button is not offered: the repair already has a Sales Order, and every added part is appended to that same order as it is created.

---

## Workflow 22: Generate a lot or serial number for the product to repair

**Actor.** Repair Technician, through the plus button beside the lot field, shown when no lot is selected.

**Preconditions.** The repair names a product. The repair's operation type allows creating new lots (`RM-050`).

**Steps.**

1. A candidate name is drawn from the product's own lot numbering sequence.
2. When the candidate is empty, or when a lot with that name already exists for that product in the repair's company or in the shared set, a fresh candidate is derived by the [inventory operations](../inventory-operations/) domain from the last serial number of the product.
3. When no candidate can be produced, the operation is refused with the message "Please set the first Serial Number or a default sequence" (`RM-051`).
4. A Lot or Serial Number record is created for the product with that name, and it is written onto the repair.

**Postconditions.** The repair names a lot; the completion guard of workflow 14 step 6 will pass.

---

## Workflow 23: Invoice the repair

**Actor.** Salesperson and Accountant.

**Preconditions.** The repair is completed and a Sales Order exists.

**Steps.**

1. The salesperson confirms the quotation created in workflow 21. Confirming it does **not** create a delivery for the part lines: those lines are excluded from the ordinary delivery rule because their moves already belong to the repair (`RM-047`).
2. Each part line's delivered quantity is taken from its single completed repair move's recorded quantity (`RM-048`).
3. The salesperson creates the invoice from the Sales Order.
4. The invoice shows, for each part line of a tracked product, the lot or serial numbers of the repair move's detail lines.
5. The accountant posts the invoice. The journal entries produced, and the rule that suppresses the cost-of-goods-sold item when the repair already valued the consumption, are specified in [accounting-effects.md](accounting-effects.md).

**Postconditions.** The customer is invoiced for the added parts and, when a service line was sold, for the repair service.

---

## Workflow 24: Print the repair order

**Actor.** Repair Technician.

**Steps.**

1. The user triggers the Repair Order printed document, either from the form or as a contextual action on a list selection.
2. The document is rendered once per selected repair, in the language of that repair's customer, with the file name "Repair Order - " followed by the reference.

**Content.** Specified in [interfaces.md](interfaces.md).

---

## Workflow 25: Return the repaired product to the customer

**Actor.** Repair Technician and Warehouse Operator.

**Preconditions.** The repair is `done`. The product physically sits in the repair's product destination location.

**Steps.**

1. When the repair is not under warranty, the quotation of workflow 21 is confirmed and invoiced first.
2. The user opens the return transfer that brought the product in. That transfer now shows a Repair Orders button linking to the completed repair.
3. The user creates a return of that return, which is an outgoing transfer to the customer for the same product and quantity.
4. The user validates that transfer.

**Postconditions.** The repaired product has left stock towards the customer. This step belongs entirely to the [inventory operations](../inventory-operations/) domain; the Repair Order takes no part in it.

---

# Part two: maintenance workflows

## Workflow 26: Create an equipment category

**Actor.** Equipment Manager.

**Steps.**

1. The user opens a new Equipment Category form. The responsible user defaults to the acting user and the company to the active company.
2. The user enters the category name, optionally changes the responsible user, the company, the colour and the comments, and defines the equipment properties that the equipment of this category will carry.
3. The user saves.

**Postconditions.** The category exists with its folding flag true, because it holds no equipment yet. Its equipment and maintenance counters read zero.

**Deletion branch.** Deleting a category that already has equipment or Maintenance Requests is refused with "You can’t delete an equipment category if some equipment or maintenance requests are linked to it." (`RM-200`).

---

## Workflow 27: Create a piece of equipment

**Actor.** Equipment Manager.

**Preconditions.** A suitable Equipment Category exists.

**Steps.**

1. The user opens a new Equipment form. The effective date defaults to today in the acting user's time zone, the company to the active company, the archive flag to true, and, with the people bridge, the assignment mode to `employee`.
2. The user enters the equipment name and picks the category. Picking the category replaces the technician with the category's responsible user (`RM-203`).
3. The user picks the Maintenance Team and, when different from the category's default, the technician.
4. With the people bridge, the user picks the assignment: `employee` shows the Employee field and clears the department; `department` shows the Department field and clears the employee; `other` shows both. Whichever is chosen, the assigned date is stamped with today in the acting user's time zone and the owner is derived — the employee's user, the department manager's user, or the acting user (`RM-204`, `RM-205`, `RM-206`).
5. On the Product Information tab, the user enters the vendor, the vendor reference, the model designation, the serial number, the effective date, the cost and the warranty expiration date. With the inventory bridge, the user also picks the internal location in which the equipment is used.
6. The user saves.

**Records written.**

| Record | Effect |
|---|---|
| Equipment | The new record. |
| Thread subscription | The owner's contact is subscribed. With the people bridge, the assigned employee's user's contact and the assigned department manager's user's contact are subscribed as well (`RM-207`). |
| Thread message | Setting an owner, an employee or a department posts a message under the "Equipment Assigned" subtype, which also reaches the followers of the equipment's category through the category-level counterpart subtype (`RM-208`). |

**Postconditions.** The equipment exists. Its category's equipment counter increases and the category stops being folded. Its Maintenance tab shows a mean time between failures of zero, because there is no closed corrective request yet.

**Guard.** A second equipment with the same serial number is refused with "Another asset already exists with this serial number!" (`RM-201`).

---

## Workflow 28: Match an equipment to a registered lot or serial number

**Actor.** Equipment Manager holding the lot and serial number tracking group.

**Preconditions.** The inventory bridge is installed and the equipment carries a serial number.

**Steps.**

1. The system counts, for the equipment's serial number text, the Lot or Serial Number records bearing exactly that text. When there is at least one, the serial-match flag is true and a Serial Number button appears on the equipment form.
2. Triggering the button opens the single matching lot record, or the list of matching records when there is more than one.

**Postconditions.** None; the action only navigates. This is how a piece of equipment registered by serial number is tied back to the goods traceability of the same serial number, for example a machine that was received into stock as a serial-numbered product.

**Branch.** For a user who cannot read lots, or who does not hold the lot and serial number tracking group, the flag is always false and the button never appears (`RM-209`).

---

## Workflow 29: Create a maintenance request

**Actor.** Internal User (any employee) or Equipment Manager.

**Preconditions.** At least one Maintenance Stage exists. At least one Maintenance Team exists. To name a piece of equipment, the user must either hold the Equipment Manager role or be a follower of that equipment (`RM-252`).

**Entry points.**

| Entry point | Pre-filled values |
|---|---|
| Maintenance Requests screen | The technician is the acting user. The Active filter is applied. |
| Equipment form, Maintenance button | The equipment is that equipment; the screen is filtered on it, and the company and team are pre-filled from it. |
| Equipment Category form, Maintenance button | The category is that category; the screen is filtered on it. |
| Team dashboard card | The team is that team; the screen is filtered on it. |
| Maintenance calendar | The scheduled date and time are taken from the clicked slot. |
| Team incoming-mail alias | See workflow 30. |

**Steps.**

1. The form opens with these defaults: the request date is today in the acting user's time zone; the created-by user is the acting user; the kanban state is `normal`; the archive flag is false; the maintenance kind is `corrective`; the instruction medium is `text`; the repeat interval is `1`; the repeat unit is `week`; the repeat kind is `forever`; the stage is the one with the lowest sequence; the team is the first team of the current company, falling back to the first team of any company (`RM-235`); the company is the active company. With the people bridge, the employee is the acting user's own employee record.
2. The user enters the subject.
3. The user optionally picks the equipment. Doing that derives the category from the equipment, derives the technician from the equipment's technician falling back to the category's responsible user (`RM-237`), and derives the team from the equipment's team (`RM-236`). Each derivation is then cleared when the resulting record does not match the request's company.
4. The user picks the maintenance kind. Choosing `corrective` forces the recurrence flag to false and hides the recurrence block (`RM-222`).
5. The user sets the scheduled date. The scheduled end is derived as the start plus one hour, and the duration becomes 1.00. The user may override the end, which recomputes the duration. Setting an end earlier than the start is refused with "End date cannot be earlier than start date." (`RM-220`).
6. For a preventive request the user may tick Recurrent and set the interval, the unit, the end kind and, for the `until` end kind, the end date. An interval below one is refused with "The repeat interval cannot be less than 1." (`RM-221`).
7. The user sets the priority, writes the internal notes, and chooses one of the three instruction carriers: an uploaded document in the portable document format, a link to a publicly readable slide deck, or text typed in place. The chosen carrier is required.
8. The user saves.

**Records written on save.**

| Record | Effect |
|---|---|
| Maintenance Request | The new record, in the first stage unless a stage was supplied (T-30, T-31). |
| Close-date correction | When a close date was supplied and the stage is not a closing stage, the close date is cleared. When no close date was supplied and the stage is a closing stage, the close date is set to today (`RM-225`). |
| Thread subscription | The contacts of the created-by user and of the technician are subscribed. With the people bridge, the employee's user's contact is subscribed too. |
| Thread message | A message is posted under the "Request Created" subtype. It is hidden on the request itself but is relayed to the followers of the equipment's category. |
| Scheduled activity | When the request carries a scheduled date, one maintenance activity is scheduled (workflow 34). |

**Postconditions.** The request appears in the first column of the pipeline, in the team's dashboard counters and, when scheduled, on the calendar.

---

## Workflow 30: Create a maintenance request by electronic mail

**Actor.** Anyone who can send a message to the team's alias address.

**Preconditions.** The Maintenance Team has an alias local part and an alias domain.

**Steps.**

1. An inbound message arrives at the team's alias address.
2. The messaging layer creates a Maintenance Request from the message: the subject becomes the request subject, the body becomes the first message of the thread, attachments are attached to that message, and the alias's default values force the team to the team that owns the alias (`RM-239`).
3. With the people bridge, the sender's address is normalised and matched against user login names. When a user matches, the employee record of the **acting processing** user is written onto the request.
4. The ordinary creation rules of workflow 29 then run: first stage, close-date correction, subscriptions, the "Request Created" message and activity scheduling.
5. The carbon-copy addresses of the inbound message are retained on the request, so that later replies reach the same people.

**Postconditions.** A request exists in the first stage, assigned to the team that owns the alias.

---

## Workflow 31: Move a request across stages

**Actor.** Internal User on their own requests; Equipment Manager on any request.

**Preconditions.** The request exists and is not archived; an archived request hides the stage bar.

**Steps.**

1. The user drags the card to another column, or picks the stage on the status bar of the form.
2. **Before the write is applied**, when the target stage carries the closing flag, the recurrence rule runs for every request of the set (workflow 33).
3. When the same write does not explicitly set a kanban state, the kanban state is forced to `normal` (`RM-226`, T-43). A request that had been Blocked or Ready for next stage therefore returns to In Progress on every stage change.
4. The write is applied.
5. **After the write:**
   1. Requests now sitting in a closing stage have their close date set to today.
   2. Requests now sitting in a non-closing stage have their close date cleared.
   3. The pending maintenance activity of every request of the set is marked done with feedback, which removes it from the technician's to-do list and records the completion in the thread.
   4. Requests now sitting in a non-closing stage have their activity recreated or rescheduled (workflow 34).
6. A message is posted under the "Status Changed" subtype on each request.

**Postconditions.** The request sits in the new stage (T-32 or T-33). Its close date matches the stage's closing flag. Its equipment's counters and effectiveness measurements recompute, because they depend on the stage's closing flag, the close date and the request date.

**Branch: several requests written at once.** The rules above are written to work on a set. Writing the same stage and the same kanban state onto several requests in one operation applies both values to all of them.

---

## Workflow 32: Block and unblock a request

**Actor.** Internal User or Equipment Manager.

**Steps.**

1. The user sets the kanban state to `blocked` (Blocked) when the work cannot continue, to `done` (Ready for next stage) when it is finished within the stage, or back to `normal` (In Progress).
2. The value is tracked in the thread (T-40, T-41, T-42).

**Postconditions.** The team dashboard's blocked counter and the pipeline's progress bar reflect the change. The next stage change resets the value to `normal` unless the same write sets another value.

---

## Workflow 33: Repeat a preventive maintenance request

**Actor.** The system, when a recurrent preventive request reaches a closing stage.

**Preconditions.** The request's maintenance kind is `preventive` and its recurrence flag is true.

**Steps.** For each request of the set being moved into a closing stage:

1. Requests that are not preventive, or that are not recurrent, are skipped.
2. The base moment is the request's scheduled date; when the request has none, the current moment is used instead.
3. The base moment is advanced by the repeat interval, counted in the repeat unit as calendar days, calendar weeks, calendar months or calendar years, producing the successor's scheduled start.
4. The successor's scheduled end is the successor's start advanced by the request's duration in hours; when the duration is zero, by one hour.
5. **Branch A, the repeat kind is `forever`.** The successor is created.
6. **Branch B, the repeat kind is `until`.** The successor is created only when the date part of the successor's scheduled start is not later than the end date. Otherwise no successor is created and the series ends.
7. The successor is a full copy of the request, with the scheduled start, the scheduled end and the stage overridden. The stage is set to the one with the lowest sequence. Because the stage field is not carried into a copy and the close-date rules run on the copy's creation, the successor opens with no close date. Fields that their own definition excludes from a copy do not carry over.
8. The successor's creation runs workflow 29's creation rules: subscriptions, the "Request Created" message and exactly one scheduled activity.

**Postconditions.** Exactly one successor exists, in the first stage, scheduled one interval later (`RM-224`). The original sits in the closing stage with its close date stamped and its own activity marked done, leaving it with no open activity.

**Worked example.** See [calculations.md](calculations.md), calculation 30.

---

## Workflow 34: Keep the scheduled activity in step

**Actor.** The system.

**Trigger.** Creation of a request; a write that changes the technician or the scheduled date; a write that changes the equipment; a stage change into a non-closing stage.

**Steps.**

1. Requests with no scheduled date have their maintenance activity deleted.
2. For each request with a scheduled date:
   1. The deadline is the scheduled date and time converted into the acting user's time zone, reduced to its date part.
   2. The responsible person is the request's technician, falling back to the request's created-by user, falling back to the acting user.
   3. An existing maintenance activity is rescheduled to that deadline and reassigned to that person.
   4. When no maintenance activity exists, one is scheduled with that deadline and that person. Its note reads "Request planned for " followed by the equipment rendered as a link, or is left empty when the request names no equipment.
3. When the write changed the equipment, the existing activity is deleted first and then recreated, because its note names the equipment.

**Postconditions.** Each open request carries exactly one maintenance activity (`RM-227`). A request that reaches a closing stage carries none, because the stage change marks it done.

**Worked example: time zone.** A user whose time zone runs twelve hours ahead of coordinated universal time creates a request scheduled at 20:00 on 10 January in coordinated universal time, which is 08:00 on 11 January in that user's zone. The activity deadline is 11 January, not 10 January. Calculation 31 carries a second worked example with a thirteen-hour offset.

---

## Workflow 35: Cancel and reopen a maintenance request

**Actor.** Internal User or Equipment Manager.

**Cancel steps.**

1. The user triggers Cancel on the request form. The button is shown only when the archive flag is false.
2. The archive flag is set to true and the recurrence flag to false in one write (T-50, `RM-228`).
3. The form shows a "Cancelled" badge and hides the stage bar.

**Postconditions.** The request is excluded from the Active filter, from the team dashboard counters and from its equipment's open-request count. Because recurrence was switched off, moving it later into a closing stage produces no successor.

**Reopen steps.**

1. The user triggers Reopen Request. The button is shown only when the archive flag is true.
2. The archive flag is set to false and the stage to the one with the lowest sequence in one write (T-51, T-34, `RM-229`).
3. Because the stage changed, the kanban state resets to `normal`, the close date is cleared — the first stage is not a closing stage in the shipped configuration — and the activity is recreated.

**Postconditions.** The request is back in the first column of the pipeline.

---

## Workflow 36: Register an employee departure

**Actor.** Human Resources Officer.

**Preconditions.** The people bridge is installed.

**Steps.**

1. The officer opens the departure procedure for one or more employees.
2. The Free Equiments option is shown and is ticked by default.
3. The officer confirms the departure.
4. The ordinary departure procedure of the [human resources core](../human-resources-core/) domain runs first.
5. When the option is ticked, the equipment collection of every departing employee is cleared, which sets the assigned employee to empty on each of those Equipment records (`RM-260`).
6. Clearing the assigned employee recomputes the owner on each of them: because the assignment mode is still `employee` but there is no employee, the owner becomes empty.

**Postconditions.** The departing employee holds no equipment. The equipment records themselves survive, are not archived, and are ready to be reassigned.

---

# Part three: where the state tables live

The transition tables of the Repair Order lifecycle, of the readiness of its parts, of the Stock Move states a repair reaches, of the Maintenance Request pipeline, of the within-stage signal and of the hiding flag are in [state-machines.md](state-machines.md), together with their guards, their refusal messages and their diagrams. This file names those transitions by identifier wherever a workflow crosses one.
