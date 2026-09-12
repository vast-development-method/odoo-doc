# Business rules

The complete rule catalogue of the repair and maintenance domain: validations, constraints, invariants, exact messages, permission checks and locking rules. Every rule carries a stable identifier of the form `RM-nnn`, unique within this file. Rules numbered `RM-001` to `RM-122` govern Repair Orders and their bridges; rules numbered `RM-200` to `RM-260` govern maintenance.

Messages are reproduced exactly as the system emits them, in quotation marks. A placeholder inside a message is described in words rather than written as a symbol, so that a rebuild substitutes its own values and produces the same text.

## Rule index

| Identifier | Subject |
|---|---|
| RM-001 | The reference comes from the operation type's numbering sequence |
| RM-002 | Changing the operation type renumbers an open repair |
| RM-003 | Every repair owns a shared stock reference |
| RM-004 | The operation type must be a repair type of the repair's company |
| RM-005 | Company consistency across every reference of a repair |
| RM-006 | Required fields of a Repair Order and of a part line |
| RM-007 | Which products may be repaired |
| RM-008 | Which transfers may source a repair |
| RM-009 | Which units of measure may be used |
| RM-010 | Negative part quantities are refused |
| RM-011 | The product to repair must be present, or the shortfall accepted |
| RM-012 | Procurement methods are adjusted only against repair rules |
| RM-013 | Confirmation touches only repairs that are still new |
| RM-014 | Parts added to a running repair are confirmed at once |
| RM-020 | A repair can only be ended while it is under repair |
| RM-021 | A tracked product must carry a lot or serial number at completion |
| RM-022 | A short part quantity requires an explicit confirmation |
| RM-023 | Parts with no recorded quantity are cancelled, not completed |
| RM-024 | The picked marking is all or nothing |
| RM-025 | Repair movements are never split and produce no backorder |
| RM-026 | Owner of the repaired product |
| RM-027 | A repair service sold on a Sales Order is reported as delivered at completion |
| RM-028 | The repaired-product movement carries no part kind and no transfer |
| RM-029 | The consumed-lines link |
| RM-030 | A completed repair cannot be cancelled |
| RM-031 | Cancelling a repair zeroes the Sales Order Line that raised it |
| RM-032 | Cancelling a repair cancels its parts and empties the quotation |
| RM-033 | Setting a cancelled repair back to new restores the quotation |
| RM-034 | Deleting a repair cancels it first and deletes its parts |
| RM-040 | A repair may not be bound to two Sales Orders |
| RM-041 | A quotation requires a customer |
| RM-042 | Only added parts are billed |
| RM-043 | Quantities flow from the repair to the Sales Order, never back |
| RM-044 | A repair under warranty is billed at zero |
| RM-045 | One repair per qualifying Sales Order Line, whatever the quantity |
| RM-046 | Quantity changes on a repair-raising line drive the repair's state |
| RM-047 | Repair-backed Sales Order Lines create no delivery |
| RM-048 | The delivered quantity of a part line is the repair movement's recorded quantity |
| RM-049 | A repair-backed line settled in the point of sale carries its whole ordered quantity |
| RM-050 | A repair may only create lots when its operation type allows it |
| RM-051 | Serial number generation needs a sequence or a precedent |
| RM-052 | Repair counters of a lot |
| RM-053 | Removed and recycled parts count as returned serial numbers, and release them |
| RM-060 | The part kind determines the locations of a part |
| RM-061 | Changing a repair location relocates the parts |
| RM-062 | Two locations mirror the operation type and cannot be edited |
| RM-063 | Warehouse mismatch warning |
| RM-064 | The scheduled date propagates to the movements |
| RM-070 | Tag names are unique |
| RM-071 | A new tag gets a random colour |
| RM-080 | A warehouse needs a production location |
| RM-081 | A warehouse needs an inventory-loss location |
| RM-082 | Every warehouse gets a repair replenish-on-order rule |
| RM-083 | Repair operation type defaults |
| RM-084 | Repair sequence naming |
| RM-085 | Renaming or archiving a warehouse follows through |
| RM-090 | A product used by a repair cannot change its reference unit freely |
| RM-091 | The parts catalog offers goods only |
| RM-100 | A repair consumption is never valued twice |
| RM-101 | A repair-backed Sales Order Line reports no valued movements |
| RM-110 | Model access for repair entities |
| RM-111 | Multi-company visibility of repairs |
| RM-112 | Field-level visibility on the repair side |
| RM-113 | Menu visibility on the repair side |
| RM-120 | Quantity comparisons use the rounding of the relevant unit |
| RM-121 | Time zone of the date category search |
| RM-122 | Lateness of a repair on the overview |
| RM-200 | A category in use cannot be deleted |
| RM-201 | Serial numbers of equipment are unique |
| RM-202 | Display name of an equipment |
| RM-203 | Choosing a category overwrites the technician |
| RM-204 | Assignment exclusivity |
| RM-205 | The assignment date is stamped on every assignment change |
| RM-206 | Owner derivation on an equipment |
| RM-207 | Subscription rules for equipment |
| RM-208 | Assignment changes are announced |
| RM-209 | Serial matching requires the lot permissions |
| RM-210 | Equipment used by a maintenance request cannot be deleted |
| RM-220 | The scheduled window must be ordered |
| RM-221 | The repetition interval is at least one |
| RM-222 | Only preventive requests may recur |
| RM-223 | A bounded series needs an end date |
| RM-224 | Reaching a closing stage creates the successor of a recurrent preventive request |
| RM-225 | The close date follows the stage |
| RM-226 | Stage changes reset the within-stage signal |
| RM-227 | Exactly one maintenance activity per open request |
| RM-228 | Cancelling a request ends its series |
| RM-229 | Reopening returns a request to the first stage |
| RM-230 | A stage in use cannot be deleted |
| RM-235 | Default team of a new request |
| RM-236 | Team derivation and company guard |
| RM-237 | Technician derivation and company guard |
| RM-238 | Only closed corrective requests feed the effectiveness measurements |
| RM-239 | Alias defaults of a team |
| RM-240 | Open request counts differ between an item and a category |
| RM-241 | An empty category is folded |
| RM-242 | Team dashboard counters |
| RM-250 | Model access for maintenance entities |
| RM-251 | Record rule on maintenance requests for ordinary users |
| RM-252 | Record rule on equipment for ordinary users |
| RM-253 | Record rules for equipment managers |
| RM-254 | Multi-company visibility of maintenance records |
| RM-255 | Human resources officers are equipment managers |
| RM-256 | Menu visibility on the maintenance side |
| RM-257 | Field-level visibility in maintenance |
| RM-260 | Departure may free the equipment |

---

# Part one: repair rules

## Identity, numbering and references

### RM-001: The reference comes from the operation type's numbering sequence

**Scope.** Creating a Repair Order.

**Rule.** When the supplied reference is empty or equal to the literal text `New`, the reference is drawn from the numbering sequence attached to the operation type resolved for the record. When the caller supplied any other reference, that reference is kept.

**Effect.** The reference is a permanent, human-readable identifier. No database uniqueness constraint enforces it; uniqueness follows from the sequence.

### RM-002: Changing the operation type renumbers an open repair

**Scope.** Writing the operation type on an existing Repair Order.

**Condition.** The repair's state is neither `cancel` nor `done`, and the new operation type differs from the current one.

**Effect.** A new reference is drawn from the new operation type's numbering sequence and replaces the current one. The part movements of the repair inherit the new reference as their own document reference and origin, and are unreserved and then reserved again from the new locations. Writing the same operation type a second time changes nothing, because the condition requires a difference.

**Effect on cancelled or completed repairs.** None. Such repairs keep their reference even when the operation type is rewritten.

### RM-003: Every repair owns a shared stock reference

**Scope.** Creating a Repair Order.

**Rule.** When the caller supplied no references collection, one Stock Reference record is created, named after the repair's reference, and linked to the repair.

**Effect.** Every replenishment document created for a part of the repair shares that reference, which is how the manufacturing counter, the purchase counter and the corresponding actions find the Manufacturing Orders and Purchase Orders that feed the repair.

### RM-004: The operation type must be a repair type of the repair's company

**Scope.** Choosing the operation type on a Repair Order.

**Condition.** The operation type's code is `repair_operation` and its company is the repair's company.

**Effect.** Values failing the condition are not offered in the selector and are refused by the company-consistency check of `RM-005` when written by an integration.

### RM-005: Company consistency

**Scope.** Creating or writing a Repair Order, and confirming it.

**Rule.** Every company-bearing record referenced by the repair must belong to the repair's company or to no company. The referenced fields are the customer, the responsible user, the product to repair, the lot, the operation type, the component source location, the product source location, the product destination location, the added-parts destination location, the removed-parts destination location, the recycled-parts destination location, the part movements, the repaired-product movement, the Sales Order, the Sales Order Line and the return transfer. The same check runs over every part movement of the repair when the repair is confirmed and when a part is created on a running repair.

**Effect.** A violation raises the platform's company-consistency error, which names the offending record and the company. A reference to a record belonging to no company at all is accepted.

### RM-006: Required fields

**Scope.** Saving a Repair Order.

**Rule.** The reference, the company, the state, the scheduled date, the operation type, the component source location, the product source location, the product destination location, the added-parts destination location, the removed-parts destination location and the recycled-parts destination location are required — eleven fields. The product to repair, the customer, the lot and the responsible user are optional. Every part line requires a part kind and a product.

**Effect.** A save with any required field empty is refused and the empty field is reported as required.

### RM-007: Which products may be repaired

**Scope.** Choosing the product to repair.

**Condition.** The product's kind is the stored value `consu`, meaning goods rather than a service; **and** the product's company is the repair's company or no company; **and**, when the repair is bound to a return transfer, the product is among the products that transfer moved, or is that transfer's own headline product.

**Effect.** Services cannot be named as the product to repair. When the repair is bound to a transfer, only the products moved by that transfer may be chosen, and when the transfer moved exactly one product, that product is preselected.

### RM-008: Which transfers may source a repair

**Scope.** Choosing the return transfer.

**Condition.** The transfer's return-origin reference is set — that is, the transfer is itself the return of another transfer — **and**, when a product is already chosen on the repair, the transfer carries that product.

**Effect.** Only a transfer that is itself a return may be named. An ordinary receipt or delivery cannot.

### RM-009: Which units of measure may be used

**Scope.** Choosing the unit of the repair quantity.

**Condition.** The unit belongs to the union of the product's reference unit, the product's other permitted units, and the units used by the product's vendor price entries.

**Effect.** Units outside that set are not offered.

## Confirmation

### RM-010: Negative part quantities are refused

**Scope.** Confirming a Repair Order.

**Condition.** At least one part movement of the repair has a demanded quantity strictly below zero.

**Effect.** The operation is refused with the message "You can not enter negative quantities." Nothing is written; the repair stays in `draft`.

### RM-011: The product to repair must be physically present, or the user must accept the shortfall

**Scope.** Confirming a Repair Order.

**Condition for the check to apply.** The repair names a product to repair and that product is a storable good. When it names none, or the product is not storable, the check is skipped entirely and confirmation proceeds.

**Test.** Two quantities are read from the stock quantity records of the repair's product at the repair's product source location, for the repair's lot: the *owned quantity*, restricted to records whose owner is the repair's customer, and the *unowned quantity*, restricted to records with no owner. The *required quantity* is the repair's product quantity converted from the repair's unit into the product's reference unit. The check passes when the owned quantity is greater than or equal to the required quantity, or when the unowned quantity is greater than or equal to the required quantity, each comparison made at the *Product Unit* decimal precision.

**Effect when the test passes.** Confirmation proceeds.

**Effect when the test fails.** The Insufficient Repair Quantity Warning dialogue is opened. Accepting it confirms the repair unchanged; discarding it leaves the repair in `draft` and writes nothing.

**Note.** The comparison is made against the two owner buckets separately. A product of which the customer owns part and the company owns part, neither part alone sufficing, fails the test even when the combined quantity would suffice. Calculation 14 works that case through.

### RM-012: Procurement methods are adjusted only against repair rules

**Scope.** Confirming a Repair Order; creating a part on a confirmed or running repair.

**Rule.** When the procurement method of the part movements is adjusted, only rules whose operation type carries the repair code are considered. A part therefore becomes a make-to-order part only through the warehouse's repair replenish-on-order rule, never through a delivery rule or a manufacturing rule that happens to match the same pair of locations.

### RM-013: Confirmation touches only repairs that are still new

**Scope.** The confirmation procedure.

**Rule.** Only repairs whose state is `draft` are checked, adjusted, confirmed and written to `confirmed`. Repairs already in any other state are silently left alone.

**Effect.** A batch operation over a mixed selection never regresses a running or completed repair, and no movement of such a repair is touched.

### RM-014: Parts added to a running repair are confirmed at once

**Scope.** Creating a Stock Move that names a repair.

**Condition.** The move is being created in the `draft` state and its repair is `confirmed` or `under_repair`.

**Effect.** The movement's company is checked, its procurement method is adjusted under `RM-012`, it is confirmed, and the replenishment scheduler is triggered for it, all inside the creation. Parts created on a repair that is still `draft` stay in `draft`.

## Execution and completion

### RM-020: A repair can only be ended while it is under repair

**Scope.** Ending a repair.

**Condition.** At least one repair in the operated set has a state other than `under_repair`.

**Effect.** The operation is refused with the message "Repair must be under repair in order to end reparation." Nothing is written for any repair of the set.

### RM-021: A tracked product must carry a lot or serial number at completion

**Scope.** Completing a repair.

**Condition.** The repair names a product, that product's tracking mode is not `none`, and no lot or serial number is chosen on the repair.

**Effect.** The operation is refused with the message "Serial number is required for product to repair : " followed by the product's display name. The repair stays in `under_repair`.

### RM-022: A short part quantity requires an explicit confirmation

**Scope.** Ending a repair from the form.

**Condition.** The incomplete-parts flag is true: at least one part movement has a recorded quantity strictly lower than its demanded quantity, compared at the rounding of that movement's unit of measure.

**Effect.** The screen asks, before doing anything: "For some of the parts, there is a difference between the initial demand and the actual quantity that was used. Are you sure you want to confirm ?" Discarding writes nothing. A recorded quantity **higher** than the demand does not trigger the prompt.

### RM-023: Parts with no recorded quantity are cancelled, not completed

**Scope.** Completing a repair.

**Condition.** A part movement's recorded quantity is zero at the rounding of its unit of measure.

**Effect.** The movement is cancelled. Cancelling it sets the ordered quantity of its Sales Order Line, when it has one, to zero. The repair's part list keeps the cancelled line, which stays visible as a record of what was planned but not used.

### RM-024: The picked marking is all or nothing

**Scope.** Completing a repair.

**Rule.** For each repair, when **none** of its part movements is marked picked, all of them are marked picked. When at least one is already marked picked, the rest are left untouched, and only the already-picked ones are treated as consumed.

**Consequence.** Because completion runs with backorder creation suppressed (`RM-025`), a part movement that is left unmarked is not carried forward: the completion cancels it. Its goods are not consumed and no residual document records them. Marking one part picked by hand and then ending the repair therefore discards every other part, which is why the marking is described as all or nothing.

### RM-025: Repair movements are never split and produce no backorder

**Scope.** Completing a repair.

**Rule.** A movement that belongs to a repair is excluded from the splitting procedure. Completion is run with backorder creation suppressed. A part whose recorded quantity differs from its demand, in either direction, therefore remains exactly one movement carrying the recorded quantity, and no residual document is created.

### RM-026: Owner of the repaired product

**Scope.** Creating the repaired-product movement at completion.

**Test.** The available quantity of the repair's product at the repair's **component source** location, for the repair's lot, owned by the repair's customer, is read with a strict test — that is, only records naming exactly that location, that lot and that owner are counted. When that quantity is greater than or equal to the repair's product quantity at the *Product Unit* precision, the customer becomes the owner of the movement's detail line; otherwise no owner is set.

**Effect.** The detail line of the repaired product records the customer as the owner of the goods when the customer already owned enough of them; otherwise it records no owner, which means the goods belong to the company. Goods recorded as owned by a contact other than the company are excluded from valuation.

**Note.** The availability is read at the repair's **component source** location, while the movement itself runs from the **product source** location. Both default to the warehouse stock location, so the two coincide in the shipped configuration. This is recorded as an observed behaviour and marked a **compatibility finding**: a corrected behaviour would read the availability at the product source location, which is the location the movement actually draws from. A rebuild that changes it will report a different owner in the single case where the two locations have been configured differently and the customer's goods sit only at the product source location.

### RM-027: A repair service sold on a Sales Order is reported as delivered at completion

**Scope.** Completing a repair that was raised by a Sales Order Line.

**Condition.** The originating line's product is a service **and** either the deployment carries no delivery-policy field at all, or that product's delivery policy is the stored value `ordered_prepaid`, meaning that delivery is recognised on ordered quantities.

**Effect.** The originating line's delivered quantity is set equal to its ordered quantity, whatever the number of repairs or parts. A repair service whose delivery policy is based on delivered quantities is not touched by this rule.

### RM-028: The repaired-product movement carries no part kind and no transfer

**Scope.** Completing a repair.

**Rule.** The movement created for the repaired product points at the repair but carries no part kind and no transfer. Because the repair's parts collection is filtered on a non-empty part kind, the movement is invisible there; because it carries no transfer, it never appears on the return transfer that supplied the product.

### RM-029: The consumed-lines link

**Scope.** Completing a repair.

**Rule.** The single detail line of the repaired-product movement records every detail line of every part movement of the repair as a consumed line.

**Effect.** This is what makes the repair appear in the traceability report of each part and of the repaired serial number, with the repaired-product movement as the parent node of the parts consumed into it.

## Cancellation and reopening

### RM-030: A completed repair cannot be cancelled

**Scope.** Cancelling a repair, setting a repair back to new, deleting a repair.

**Condition.** At least one repair in the operated set has the state `done`.

**Effect.** The operation is refused with the message "You cannot cancel a Repair Order that's already been completed". Because deletion cancels first, a completed repair also cannot be deleted, and because the reset operation cancels first, it cannot be reopened either.

### RM-031: Cancelling a repair zeroes the Sales Order Line that raised it

**Scope.** Cancelling a repair that has an originating Sales Order.

**Effect.** The originating Sales Order Line's ordered quantity is written to zero. The customer is therefore not charged for a repair service that will not be performed.

### RM-032: Cancelling a repair cancels its parts and empties the quotation

**Scope.** Cancelling a repair.

**Effect.** Every part movement is cancelled, which releases its reservations and sets the ordered quantity of its Sales Order Line, when it has one, to zero.

### RM-033: Setting a cancelled repair back to new restores the quotation

**Scope.** Setting a repair back to new.

**Effect.** For every Sales Order Line attached to the repair's parts, belonging to a Sales Order that is not itself cancelled, and currently carrying a zero ordered quantity at the rounding of the line's unit, the line's quantity is recomputed from its movements: the sum of the demanded quantities of the `add` movements attached to it, or zero when the movement's part kind is `remove` or `recycle`. Every part movement is then written back to the `draft` state, and the repair follows.

### RM-034: Deleting a repair cancels it first and deletes its parts

**Scope.** Deleting a Repair Order.

**Effect.** Every repair of the set that is not already cancelled is cancelled first, with all the effects of `RM-030` to `RM-032`, and then the records are deleted. The part movements point at the repair with a cascading link and are deleted with it. Their outgoing quantity claims are released.

## Billing

### RM-040: A repair may not be bound to two Sales Orders

**Scope.** Creating a quotation from one or more repairs.

**Condition.** At least one repair in the operated set already has a Sales Order.

**Effect.** The operation is refused. The message is "You cannot create a quotation for a repair order that is already linked to an existing sale order." followed by a line break, then "Concerned repair order(s):", then a line break, then the references of the offending repairs, one per line.

### RM-041: A quotation requires a customer

**Scope.** Creating a quotation from one or more repairs.

**Condition.** At least one repair in the operated set has no customer.

**Effect.** The operation is refused. The message is "You need to define a customer for a repair order in order to create an associated quotation." followed by a line break, then "Concerned repair order(s):", then a line break, then the references of the offending repairs, one per line.

### RM-042: Only added parts are billed

**Scope.** Creating and maintaining the Sales Order Lines of a repair.

**Rule.** A Sales Order Line is created only for a part movement whose part kind is `add`, that has no Sales Order Line yet, and whose repair has a Sales Order. Parts of kind `remove` and `recycle` never produce a Sales Order Line. A part whose kind is changed away from `add` has the ordered quantity of its existing Sales Order Line set to zero; the line itself is kept, because deleting a line of a confirmed order is not permitted. Changing the kind back to `add` restores the quantity, or creates the line when none exists.

### RM-043: Quantities flow from the repair to the Sales Order, never back

**Scope.** Quantity changes on either side.

**Rule.** Changing a part's demanded quantity rewrites the ordered quantity of its Sales Order Line to the sum of the demanded quantities of all movements attached to that line. Changing the quantity on the Sales Order Line has no effect on the repair part, and is overwritten by the next change made on the repair side.

### RM-044: A repair under warranty is billed at zero

**Scope.** Ticking or unticking the warranty flag; creating a Sales Order Line for a part.

**Rule.**

- While the flag is true, every Sales Order Line created for an `add` part receives a unit price of zero.
- Ticking the flag on an existing repair writes zero into both the unit price and the stored manual-price marker of every such line.
- Unticking it recomputes every such line's unit price from the price list, the customer and the quantity.

**Not affected.** Discounts entered by hand on those lines, and the price of any line that was not created from an `add` part of this repair.

### RM-045: One repair per qualifying Sales Order Line, whatever the quantity

**Scope.** Confirming a Sales Order.

**Rule.** Each line whose product is a service with service tracking `repair`, whose quantity is strictly positive, and whose movements do not already belong to a repair, produces exactly one Repair Order. A quantity of three produces one repair, not three.

### RM-046: Quantity changes on a repair-raising line drive the repair's state

**Scope.** Writing the ordered quantity on a Sales Order Line of an order in the `sale` or `done` state.

| Transition of the quantity, compared at the rounding of the line's unit | Effect on the bound repair |
|---|---|
| from zero or less to strictly positive | Every bound repair in `cancel` is set back to `draft` and then confirmed. |
| from strictly positive to zero or less | Every bound repair that is not `done` is cancelled. |
| any other change | None. |

### RM-047: Repair-backed Sales Order Lines create no delivery

**Scope.** Confirming a Sales Order; building the delivery of a point-of-sale order.

**Rule.** A line whose movements belong to a Repair Order is excluded from the ordinary delivery rule. The goods move through the repair's own movements instead. With the point-of-sale bridge installed, the same lines are excluded when a point-of-sale order builds its delivery.

### RM-048: The delivered quantity of a part line is the repair movement's recorded quantity

**Scope.** Computing the delivered quantity of a Sales Order Line.

**Condition.** The line has exactly one movement that belongs to a repair and is in the `done` state.

**Effect.** The delivered quantity is that movement's recorded quantity, replacing the ordinary computation. A line with none, or with more than one such movement, uses the ordinary computation.

### RM-049: A repair-backed line settled in the point of sale carries its whole ordered quantity

**Scope.** Settling a Sales Order into a point-of-sale order, with the point-of-sale bridge installed.

**Condition.** The Sales Order Line reports that it is backed by a repair.

**Effect.** The point-of-sale line takes the Sales Order Line's ordered quantity as it stands. The ordinary rule, which offers only the part of the ordered quantity that no delivery has covered so far, is not applied to such a line. A repair-backed line's goods move through the repair's own movements and never through a delivery, so the ordered quantity is the amount the cashier charges for. Lines on the same order that are not repair-backed keep the ordinary rule.

## Lots and serial numbers

### RM-050: A repair may only create lots when its operation type allows it

**Scope.** Creating a Lot or Serial Number while working inside a Repair Order.

**Condition.** The surrounding context names an active repair and that repair's operation type does not allow creating new lots.

**Effect.** The creation is refused with the message "You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers"." The inner quotation marks around the option name are part of the message.

### RM-051: Serial number generation needs a sequence or a precedent

**Scope.** Generating a lot or serial number for the product to repair.

**Condition.** Neither the product's own lot numbering sequence nor the derivation from the product's last serial number produces a name.

**Effect.** The operation is refused with the message "Please set the first Serial Number or a default sequence".

### RM-052: Repair counters of a lot

**Scope.** Reading a Lot or Serial Number.

| Counter | Definition |
|---|---|
| In repair count | Repair Orders whose lot is this lot and whose state is neither `done` nor `cancel`. |
| Repaired count | Repair Orders whose lot is this lot and whose state is `done`. |
| Repair part count, and the list of repairs behind it | Repair Orders holding a **completed** part movement, of any part kind, one of whose detail lines carries this lot. |

### RM-053: Removed and recycled parts count as returned serial numbers, and release them

**Scope.** Counting how many units of a serial-numbered product came back; and the guard that stops a unique serial number being consumed twice.

**Rule, first effect.** A detail line whose movement has a part kind of `remove` or `recycle` and whose destination location is internal counts as a return, in addition to the ordinary cases.

**Rule, second effect, on manufacturing.** The [manufacturing](../manufacturing/) domain refuses to consume a unique serial number that a Manufacturing Order has already consumed, with the message "The serial number " followed by the serial number, " used for component " followed by the component name, " has already been consumed". That guard weighs consumptions against releases: a completed detail line of exactly one unit of the serial **into** a production location, under a Manufacturing Order, is a consumption; a completed detail line of exactly one unit of the serial **out of** a production location, under no Manufacturing Order, releases it. Every `remove` and `recycle` part of a repair runs out of the repair's added-parts destination location, which is a production location in the shipped configuration, and belongs to no Manufacturing Order. Taking a component out of a repaired product therefore releases its serial number, and the very same unit may be consumed by a later Manufacturing Order.

**The two effects are independent.** A part removed into the inventory-loss location does not add to the returned-serial-number count, because the destination is not internal, yet it still releases the serial, because what releases the serial is the direction out of the production location. This domain owns neither the manufacturing guard nor its message; it owns the movement whose direction releases the serial.

## Locations

### RM-060: The part kind determines the locations of a part

**Scope.** Every part movement of a repair.

| Part kind | Source location of the movement | Destination location of the movement |
|---|---|---|
| `add` | the repair's component source location | the repair's added-parts destination location |
| `remove` | the repair's added-parts destination location | the repair's removed-parts destination location |
| `recycle` | the repair's added-parts destination location | the repair's recycled-parts destination location |

A movement with a repair but no part kind — that is, the repaired-product movement — keeps the locations it was created with, namely the repair's product source and product destination locations.

### RM-061: Changing a repair location relocates the parts

**Scope.** Writing the component source location, the added-parts destination location, the removed-parts destination location or the recycled-parts destination location on a Repair Order.

**Effect.** Every part movement is relocated according to `RM-060`. Writing the product source location or the product destination location does **not** relocate any part, because those two locations concern only the repaired product.

### RM-062: Two locations mirror the operation type and cannot be edited

**Scope.** The added-parts destination location and the removed-parts destination location on a Repair Order.

**Rule.** They always equal, respectively, the default destination location and the default remove destination location of the repair's operation type. They are read-only on every screen. The other four locations are derived from the operation type at creation but may then be edited on the order.

### RM-063: Warehouse mismatch warning

**Scope.** Editing the component source location or the return transfer on the Repair Order form.

**Condition.** The component source location belongs to a warehouse, the transfer's destination location belongs to a warehouse, and the two warehouses differ.

**Effect.** A non-blocking warning is shown with the title "Warning" and the message "Note that the warehouses of the return and repair locations don't match!" The user may proceed; the record saves, the repair confirms, and nothing about the movements changes. When either side belongs to no warehouse at all, no warning is raised.

### RM-064: The scheduled date propagates to the movements

**Scope.** Writing the scheduled date on a Repair Order.

**Effect.** The date of the repaired-product movement and of every part movement whose state is neither `done` nor `cancel` is written to the new scheduled date.

## Tags

### RM-070: Tag names are unique

**Scope.** Creating or renaming a Repair Tag.

**Constraint.** A database uniqueness constraint over the tag name, spanning the whole installation.

**Effect.** A duplicate is refused with the message "Tag name already exists!".

### RM-071: A new tag gets a random colour

**Scope.** Creating a Repair Tag without an explicit colour.

**Rule.** The colour index is a pseudo-random whole number drawn uniformly from one to eleven inclusive. Zero is never drawn. Nothing in the domain reads the value: two tags sharing a colour behave identically. A colour supplied explicitly is kept and no draw takes place.

## Warehouse and route configuration

### RM-080: A warehouse needs a production location

**Scope.** Creating a Warehouse while the repair capability is installed.

**Rule.** The company must own a location of production usage. When none exists, one is created before the operation types are built. When one still cannot be found at the moment the repair operation type is prepared, the creation is refused with the message "Can't find any production location."

### RM-081: A warehouse needs an inventory-loss location

**Scope.** Creating a Warehouse while the repair capability is installed.

**Condition.** No location of inventory-loss usage exists for the warehouse's company or in the shared set.

**Effect.** The creation is refused with the message "No location of type Inventory Loss found".

### RM-082: Every warehouse gets a repair replenish-on-order rule

**Scope.** Creating a Warehouse.

**Rule.** A pull rule is created on the installation-wide replenish-on-order route with: procurement method make to order, action pull, automation manual, source the warehouse stock location, destination the repair operation type's default destination location (the production location), operation type the warehouse's repair operation type, company the warehouse's company. The rule is active. When the route does not exist it is created, named "Replenish on Order (MTO)".

### RM-083: Repair operation type defaults

**Scope.** Creating a Warehouse; creating a repair operation type by hand.

| Default | Value |
|---|---|
| Name | "Repairs" |
| Code | `repair_operation` |
| Default source location (components) | the warehouse stock location |
| Default destination location (components) | the lowest-numbered production-usage location of the company |
| Default remove destination location | the lowest-numbered inventory-loss location of the company, or of the shared set when the company owns none |
| Default recycle destination location | the warehouse stock location |
| Default product source location | the warehouse stock location |
| Default product destination location | the warehouse stock location |
| Sequence code | `RO` |
| New lots may be created | yes |
| Existing lots may be used | yes |

### RM-084: Repair sequence naming

**Scope.** Creating a Warehouse.

**Rule.** The numbering sequence of the repair operation type is named as the warehouse name followed by " Sequence repair". Its prefix is the warehouse short code, a forward slash, the operation type's sequence code (falling back to `RO` when none is set), and a further forward slash. It is padded to five digits and owned by the warehouse's company.

**Example.** Warehouse code `WH`, sequence code `RO`: the first repair of that type is `WH/RO/00001`.

### RM-085: Renaming or archiving a warehouse follows through

**Scope.** Writing on a Warehouse.

**Effect.** The repair operation type's active flag follows the warehouse's active flag, and its barcode becomes the warehouse short code with spaces removed, upper-cased, followed by the letters `RO`.

## Products

### RM-090: A product used by a repair cannot change its reference unit freely

**Scope.** Changing the reference unit of a product.

**Condition.** At least one Repair Order names this product with a unit of measure different from the product's current reference unit.

**Effect.** The change is refused with the message "As other units of measure (ex : " followed by the offending unit, ") than " followed by the current reference unit, " have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one." The absence of a space after the full stop before "If you want" is reproduced as the system emits it.

**When the condition does not hold.** Every Repair Order naming the product is rewritten to the new unit and the change proceeds.

### RM-091: The parts catalog offers goods only

**Scope.** Opening the product catalog from a Repair Order.

**Condition.** The product's kind is the stored value `consu`.

**Effect.** Service products are absent from the catalog. A dedicated filter narrows the catalog to the products already on this repair; with the manufacturing bridge, a second filter narrows it to the components of the bill of materials of the product being repaired.

## Valuation

### RM-100: A repair consumption is never valued twice

**Scope.** Posting a customer invoice.

**Condition.** At least one inventory movement behind the invoice line is an `add` part of a Repair Order and that movement already carries its own journal entry.

**Effect.** The invoice line is not eligible for the automatic cost-of-goods-sold entry. The details and the worked examples are in [accounting-effects.md](accounting-effects.md).

### RM-101: A repair-backed Sales Order Line reports no valued movements

**Scope.** Checking whether a Sales Order Line has valued movements.

**Effect.** A line whose movements belong to a repair answers no, even when those movements are neither cancelled nor draft. This keeps the ordinary costing entry from being produced a second time from the sales side. A line whose delivery movement belongs to no repair answers yes and is costed in the ordinary way.

## Access and visibility

### RM-110: Model access

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Repair Order | Inventory User | yes | yes | yes | yes |
| Repair Tag | Inventory User | yes | yes | yes | yes |
| Insufficient Repair Quantity Warning | Inventory User | yes | yes | yes | **no** |

No other group is granted access to these three entities by this domain. The Inventory Administrator group holds exactly the same rights on them, because it inherits them through the ordinary group implication and this domain grants it nothing extra.

### RM-111: Multi-company visibility of repairs

**Scope.** Reading, writing and deleting Repair Orders.

**Record rule.** The repair's company must be among the acting user's allowed companies.

Unlike the maintenance rules, a Repair Order with no company is **not** visible, because the company is required on a Repair Order.

### RM-112: Field-level visibility

| Field | Visible to |
|---|---|
| The manufacturing counter on a Repair Order | holders of the manufacturing user group |
| The purchase counter on a Repair Order | holders of the purchase user group |
| The repair collection and the repair counter on a Sales Order | holders of the inventory user group |
| The repair counter on a Manufacturing Order and on a Purchase Order | holders of the inventory user group |
| The equipment collection on an Employee | holders of the human resources user group |
| The cost on an Equipment | holders of the equipment manager group |
| The category on a Maintenance Request | holders of the equipment manager group |
| The lot or serial number on the Repair Order form and on the printed document | holders of the lot and serial number tracking group |
| The unit columns on the Repair Order form and on the printed document | holders of the unit of measure group |
| The company fields | holders of the multiple companies group |

### RM-113: Menu visibility

The Repairs menu and its Orders entry require the inventory user group. The Reporting and Configuration sections under it require the inventory administrator group. The Product Variants entry additionally requires the product variants group. The Repair Orders Tags entry is a developer-only entry. A reader who holds neither the inventory user group nor the inventory administrator group does not see the Repairs menu at all.

## Rounding, comparison and dates

### RM-120: Quantity comparisons use the rounding of the relevant unit

**Scope.** Every quantity comparison in this domain.

| Comparison | Rounding used |
|---|---|
| part recorded quantity against part demanded quantity, for the incomplete-parts flag | the rounding of the part movement's own unit of measure |
| part recorded quantity against zero, for the cancel-empty-parts rule | the rounding of the part movement's own unit of measure |
| part forecast availability against part quantity, for readiness | the rounding of the product's reference unit |
| available quantity of the product to repair against the required quantity, at confirmation and when choosing the owner | the *Product Unit* decimal precision setting |
| Sales Order Line ordered quantity against zero | the rounding of the Sales Order Line's unit of measure |

### RM-121: Time zone of the date category search

**Scope.** Searching on the scheduled-date category.

**Rule.** The day boundaries are computed in the acting user's time zone and then expressed in coordinated universal time. The full rule and its worked example are in [calculations.md](calculations.md), calculation 12. The only operator the search supports is membership in a list of categories; any other operator is rejected.

### RM-122: Lateness of a repair on the overview

**Scope.** Counting the late repairs of an operation type.

**Condition.** The repair's state is `confirmed` **and** either its scheduled date is before today or its lateness boolean is true.

Repairs already under repair are not counted as late, and neither are completed or cancelled repairs.

---

# Part two: maintenance rules

## Equipment and categories

### RM-200: A category in use cannot be deleted

**Scope.** Deleting an Equipment Category.

**Condition.** The category holds at least one Equipment record or at least one Maintenance Request.

**Effect.** The deletion is refused with the message "You can’t delete an equipment category if some equipment or maintenance requests are linked to it." The apostrophe in "can’t" is the typographic right single quotation mark, reproduced exactly.

### RM-201: Serial numbers of equipment are unique

**Scope.** Creating or writing an Equipment.

**Constraint.** A database uniqueness constraint over the serial number. It spans every company and includes archived records. The serial number is not carried into a duplicate, precisely so that duplication does not break it.

**Effect.** A duplicate is refused with the message "Another asset already exists with this serial number!".

### RM-202: Display name of an equipment

**Scope.** Rendering an Equipment anywhere.

**Rule.** When a serial number is set, the display name is the equipment name, a forward slash, and the serial number. When no serial number is set, the display name is the equipment name alone.

### RM-203: Choosing a category overwrites the technician

**Scope.** Editing the category on the Equipment form.

**Effect.** The technician is replaced by the responsible user of the chosen category, even when a technician had been chosen by hand.

### RM-204: Assignment exclusivity

**Scope.** Editing the assignment mode on an Equipment, with the people bridge installed.

| Assignment mode | Assigned employee | Assigned department |
|---|---|---|
| `employee` | kept | cleared |
| `department` | cleared | kept |
| `other` | kept | kept |

### RM-205: The assignment date is stamped on every assignment change

**Scope.** Recomputing the assignment of an Equipment.

**Effect.** The assigned date is set to today in the acting user's time zone whenever the assignment mode changes, whichever value is chosen. The field remains editable afterwards.

### RM-206: Owner derivation

**Scope.** Recomputing the owner on an Equipment, with the people bridge installed.

**Rule.** The owner starts as the acting user. When the assignment mode is `employee`, it becomes the user account of the assigned employee. When the mode is `department`, it becomes the user account of the assigned department's manager. When the mode is `other`, it stays the acting user. When the designated employee has no user account, or the department has no manager or the manager has no user account, the owner ends empty.

### RM-207: Subscription rules for equipment

**Scope.** Creating or writing an Equipment.

| Trigger | Contact subscribed |
|---|---|
| creation with an owner | the owner's contact |
| creation with an assigned employee, with the people bridge | that employee's user's contact |
| creation with an assigned department, with the people bridge | the department manager's user's contact |
| writing a non-empty owner | the new owner's contact |
| writing a non-empty assigned employee, with the people bridge | that employee's user's contact |
| writing a non-empty assigned department, with the people bridge | the department manager's user's contact |

Subscribing never unsubscribes the previous holder.

### RM-208: Assignment changes are announced

**Scope.** Tracking changes on an Equipment.

**Rule.** A change that sets the owner to a non-empty value posts the tracking message under the "Equipment Assigned" subtype. With the people bridge, a change that sets the assigned employee or the assigned department to a non-empty value posts under the same subtype. That subtype has a category-level counterpart, so the followers of the equipment's category are notified as well.

### RM-209: Serial matching requires the lot permissions

**Scope.** Computing the serial-match flag on an Equipment.

**Condition.** The acting user can read Lot or Serial Number records **and** holds the lot and serial number tracking group.

**Effect when the condition is false.** The flag is false for every equipment, without a query, and the Serial Number button never appears.

**Effect when the condition is true.** The flag is true when at least one Lot or Serial Number record bears exactly the equipment's serial number text.

### RM-210: Equipment used by a maintenance request cannot be deleted

**Scope.** Deleting an Equipment.

**Rule.** A Maintenance Request points at Equipment with a restricting link. Deleting an equipment that any request names, archived requests included, is refused by the platform's referential guard.

## Maintenance requests

### RM-220: The scheduled window must be ordered

**Scope.** Writing the scheduled end on a Maintenance Request.

**Condition.** Both the scheduled date and the scheduled end are set and the scheduled date is strictly later than the scheduled end.

**Effect.** The write is refused with the message "End date cannot be earlier than start date." A window whose start equals its end is accepted and yields a duration of zero.

### RM-221: The repetition interval is at least one

**Scope.** Writing the repeat interval on a Maintenance Request.

**Condition.** The repeat interval is strictly less than one.

**Effect.** The write is refused with the message "The repeat interval cannot be less than 1." The check applies to every request, recurrent or not.

### RM-222: Only preventive requests may recur

**Scope.** Writing the maintenance kind.

**Condition.** The maintenance kind is not `preventive`.

**Effect.** The recurrence flag is forced to false. Because the recomputation is triggered by the kind, switching a recurrent preventive request to corrective silently ends its series.

### RM-223: A bounded series needs an end date

**Scope.** The form.

**Rule.** When the repeat kind is `until`, the end date is required. When the recurrence flag is true, the repeat interval, the repeat unit and the repeat kind are all required. When the recurrence flag is false, none of the four is required.

### RM-224: Reaching a closing stage creates the successor of a recurrent preventive request

**Scope.** Writing the stage on a Maintenance Request, where the target stage carries the closing flag.

**Condition per request.** The maintenance kind is `preventive` and the recurrence flag is true.

**Effect.** Exactly one successor is created, **before** the stage write is applied and therefore from the record's current values. Its schedule is computed by [calculations.md](calculations.md), calculation 30; it is placed in the stage with the lowest sequence; the rest of its fields are copied from the original according to the duplication rules of [entities.md](entities.md). When the repeat kind is `until` and the computed start falls after the end date, no successor is created and the series ends.

### RM-225: The close date follows the stage

**Scope.** Creating or writing a Maintenance Request.

| Situation | Effect on the close date |
|---|---|
| creation, a close date supplied, target stage not a closing stage | cleared |
| creation, no close date supplied, target stage a closing stage | set to today |
| write that changes the stage, target stage a closing stage | set to today, overwriting any value in the same write |
| write that changes the stage, target stage not a closing stage | cleared, overwriting any value in the same write |
| write that does not change the stage | the supplied value is kept as written |

The last row is what allows a close date to be corrected by hand without a stage change.

### RM-226: Stage changes reset the within-stage signal

**Scope.** Writing the stage on a Maintenance Request.

**Condition.** The same write does not also set the kanban state.

**Effect.** The kanban state is forced to `normal` before the write is applied.

### RM-227: Exactly one maintenance activity per open request

**Scope.** Creating a Maintenance Request; writing its technician, its scheduled date, its equipment or its stage.

**Rules.**

1. A request with no scheduled date carries no maintenance activity; an existing one is deleted.
2. A request with a scheduled date carries exactly one, deadlined on the scheduled date expressed in the acting user's time zone and assigned to the technician, falling back to the created-by user, falling back to the acting user.
3. A stage change marks the pending activity as done with feedback; when the target stage is not a closing stage, a fresh activity is then created or the existing one rescheduled. A request in a closing stage therefore carries no open activity.
4. A change of equipment deletes and recreates the activity, because its note names the equipment.

### RM-228: Cancelling a request ends its series

**Scope.** The Cancel button on a Maintenance Request.

**Effect.** The archive flag is set to true and the recurrence flag to false in the same write. A cancelled recurrent request produces no further occurrence, even when it is later dragged into a closing stage.

### RM-229: Reopening returns a request to the first stage

**Scope.** The Reopen Request button.

**Effect.** The archive flag is set to false and the stage to the one with the lowest sequence, which by `RM-225` and `RM-226` also clears the close date and resets the within-stage signal, and by `RM-227` recreates the activity.

### RM-230: A stage in use cannot be deleted

**Scope.** Deleting a Maintenance Stage.

**Rule.** A Maintenance Request points at a Maintenance Stage with a restricting link. A stage that any request occupies cannot be deleted. Moving the occupying requests elsewhere first makes the deletion succeed, and the sequences of the remaining stages are unchanged by it.

## Derivations that clear themselves on a company mismatch

### RM-235: Default team of a new request

**Scope.** Creating a Maintenance Request without an explicit team.

**Rule.** The team is the first Maintenance Team whose company is the acting company, taken by identifier; when that company owns none, the first Maintenance Team of any company, taken by identifier.

The team is required, so a deployment with no team at all cannot create a request.

### RM-236: Team derivation and company guard

**Scope.** Recomputing the team on a Maintenance Request.

**Rule.** When the request names equipment and that equipment has a team, the request's team becomes that team. Then, when the resulting team has a company and that company differs from the request's company, the team is cleared. The same company guard, without the equipment step, applies to the team on any maintainable item.

### RM-237: Technician derivation and company guard

**Scope.** Recomputing the technician on a Maintenance Request.

**Rule.** When the request names equipment, the technician becomes that equipment's technician, or — when the equipment has none — the responsible user of the equipment's category. Then, when the resulting user is set and the request's company is not among that user's allowed companies, the technician is cleared.

### RM-238: Only closed corrective requests feed the effectiveness measurements

**Scope.** Computing the four effectiveness measurements of a maintainable item.

**Condition.** The request's maintenance kind is `corrective` **and** its stage carries the closing flag.

**Effect.** Preventive requests, and corrective requests not yet in a closing stage, contribute nothing. Archived requests **are** included, because the filter does not test the archive flag.

### RM-239: Alias defaults of a team

**Scope.** Creating or rewriting the incoming-mail alias of a Maintenance Team.

**Rule.** The alias's target entity is Maintenance Request. Its default-values map receives the entry forcing the team to that team, merged into whatever default values were already stored on the alias.

## Counting rules

### RM-240: Open request counts differ between an item and a category

| Counter | Owner | Definition |
|---|---|---|
| Maintenance count | any maintainable item, therefore Equipment | every request of the item, archived ones included |
| Current maintenance | any maintainable item | requests of the item whose stage is not a closing stage **and** which are not archived |
| Maintenance count | Equipment Category | every request whose equipment belongs to the category, archived ones included |
| Current maintenance | Equipment Category | requests of the category which are not archived, **whatever their stage** |

The difference is deliberate: the category counter answers "how many live requests exist here", the item counter answers "how much work is still open on this asset".

### RM-241: An empty category is folded

**Scope.** Computing the folding flag on an Equipment Category.

**Rule.** The folding flag is true exactly when the equipment count of the category is zero. Every category in the computed set is first set to not folded, to break the circular dependency between the flag and the grouped count that reads it.

### RM-242: Team dashboard counters

**Scope.** Computing the five dashboard counters of a Maintenance Team.

**Population.** Requests whose team is this team, whose stage does not carry the closing flag, and which are not archived.

| Counter | Definition over that population |
|---|---|
| Number of requests | the size of the population |
| Number of requests scheduled | those whose scheduled date is set |
| Number of requests in high priority | those whose priority is `3` |
| Number of requests blocked | those whose kanban state is `blocked` |
| Number of requests unscheduled | the total minus the scheduled count |

## Access and visibility

### RM-250: Model access

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Equipment | Internal User | yes | no | no | no |
| Equipment | Equipment Manager | yes | yes | yes | yes |
| Equipment Category | Internal User | yes | no | no | no |
| Equipment Category | Equipment Manager | yes | yes | yes | yes |
| Maintenance Stage | Internal User | yes | no | no | no |
| Maintenance Stage | Equipment Manager | yes | yes | yes | yes |
| Maintenance Team | Internal User | yes | no | no | no |
| Maintenance Team | Equipment Manager | yes | yes | yes | yes |
| Maintenance Request | Internal User | yes | yes | yes | yes |
| Activity Type | Equipment Manager | yes | yes | yes | yes |

Every internal user may create, read, update and delete Maintenance Requests at the model level; what they may actually see is narrowed by the record rules below.

### RM-251: Record rule on maintenance requests for ordinary users

**Scope.** Internal Users who are not Equipment Managers.

**Rule.** A request is visible when its created-by user is the acting user, **or** the acting user's contact is among its followers, **or** its technician is the acting user.

**Effect.** A user sees the requests they raised, the requests assigned to them as technician, and the requests they follow. They do not see the rest.

### RM-252: Record rule on equipment for ordinary users

**Scope.** Internal Users who are not Equipment Managers.

**Rule.** An equipment is visible when the acting user's contact is among its followers.

**Effect.** A user sees only the equipment they follow. This is the supported way to let a person raise requests for a machine without granting them the Equipment Manager role: add them as a follower of that machine.

### RM-253: Record rules for equipment managers

**Scope.** Holders of the equipment manager group.

**Rule.** Unrestricted on Maintenance Request and on Equipment, subject only to the multi-company rules.

### RM-254: Multi-company visibility of maintenance records

**Scope.** Maintenance Request, Equipment, Maintenance Team, Equipment Category.

**Rule.** The record's company must be among the acting user's allowed companies, **or** be empty. A record with no company is visible from every company. Maintenance Stage carries no company and no record rule; stages are global.

### RM-255: Human resources officers are equipment managers

**Scope.** Group implications, with the people bridge installed.

**Rule.** The human resources user group implies the equipment manager group. A human resources officer therefore manages equipment without a separate grant, and sees the fields that group reveals.

### RM-256: Menu visibility

The Maintenance menu, its Dashboard entry, its Maintenance section (Maintenance Requests and Maintenance Calendar), its Equipment entry and its two Reporting sections require either the equipment manager group or the internal user group. The Configuration section requires the equipment manager group. The Settings entry under Configuration additionally requires the settings administration group. The Maintenance Stages entry and the Activity Types entry are developer-only entries. The Equipment Categories entry under Configuration carries no group of its own and is therefore visible to anyone who can see the Configuration section. An account holding neither the equipment manager group nor the internal user group sees nothing of the maintenance menu.

### RM-257: Field-level visibility in maintenance

| Field | Visible to |
|---|---|
| The cost on an Equipment | holders of the equipment manager group |
| The category on a Maintenance Request, on the form, in the list and in the search panel | holders of the equipment manager group |
| The assigned date and the scrap date on the Equipment form | developer-only fields |
| The carbon-copy address field on a Maintenance Request | a developer-only field |
| The request date column of the Maintenance Request list | a developer-only column |
| The company fields on Equipment, Equipment Category, Maintenance Request and Maintenance Team | holders of the multiple companies group |

## Departure

### RM-260: Departure may free the equipment

**Scope.** Registering an employee departure, with the people bridge installed.

**Condition.** The Free Equiments option is ticked, which is the default.

**Effect.** The equipment collection of every departing employee is cleared, which empties the assigned employee on each of those Equipment records and, by `RM-206`, empties the owner as well while the assignment mode is still `employee`. The Equipment records themselves are neither archived nor deleted. When the option is unticked, nothing is changed.

---

## Mapping of the former rule identifiers

Two independently written descriptions of this domain used two different rule-numbering schemes. Both are mapped onto the single scheme of this file below, so that a reader holding either earlier text can find the rule again. The first scheme used the prefixes `R-` for repair and `M-` for maintenance; the second used `REP-RULE-` and `MNT-RULE-`.

| Former identifier, first scheme | Former identifier, second scheme | Identifier in this file |
|---|---|---|
| — | REP-RULE-001 | RM-001 |
| — | REP-RULE-002 | RM-002 |
| — | REP-RULE-003 | RM-003 |
| — | REP-RULE-004 | RM-004 |
| — | REP-RULE-005 | RM-005 |
| — | REP-RULE-006 | RM-006 |
| — | REP-RULE-007 | RM-007 |
| — | REP-RULE-008 | RM-008 |
| — | REP-RULE-009 | RM-009 |
| — | REP-RULE-010 | RM-010 |
| — | REP-RULE-011 | RM-011 |
| — | REP-RULE-012 | RM-012 |
| — | REP-RULE-013 | RM-013 |
| — | REP-RULE-014 | RM-014 |
| — | REP-RULE-020 | RM-020 |
| R-08 | REP-RULE-021 | RM-021 |
| — | REP-RULE-022 | RM-022 |
| — | REP-RULE-023 | RM-023 |
| — | REP-RULE-024 | RM-024 |
| — | REP-RULE-025 | RM-025 |
| — | REP-RULE-026 | RM-026 |
| — | REP-RULE-027 | RM-027 |
| — | REP-RULE-028 | RM-028 |
| — | REP-RULE-029 | RM-029 |
| — | REP-RULE-030 | RM-030 |
| — | REP-RULE-031 | RM-031 |
| — | REP-RULE-032 | RM-032 |
| — | REP-RULE-033 | RM-033 |
| R-14 | REP-RULE-034 | RM-034 |
| — | REP-RULE-040 | RM-040 |
| — | REP-RULE-041 | RM-041 |
| — | REP-RULE-042 | RM-042 |
| — | REP-RULE-043 | RM-043 |
| — | REP-RULE-044 | RM-044 |
| — | REP-RULE-045 | RM-045 |
| — | REP-RULE-046 | RM-046 |
| — | REP-RULE-047 | RM-047 |
| — | REP-RULE-048 | RM-048 |
| — | REP-RULE-049 | RM-049 |
| — | REP-RULE-050 | RM-050 |
| — | REP-RULE-051 | RM-051 |
| — | REP-RULE-052 | RM-052 |
| — | REP-RULE-053 | RM-053 |
| — | REP-RULE-060 | RM-060 |
| — | REP-RULE-061 | RM-061 |
| — | REP-RULE-062 | RM-062 |
| — | REP-RULE-063 | RM-063 |
| — | REP-RULE-064 | RM-064 |
| — | REP-RULE-070 | RM-070 |
| — | REP-RULE-071 | RM-071 |
| — | REP-RULE-080 | RM-080 |
| — | REP-RULE-081 | RM-081 |
| — | REP-RULE-082 | RM-082 |
| — | REP-RULE-083 | RM-083 |
| — | REP-RULE-084 | RM-084 |
| — | REP-RULE-085 | RM-085 |
| — | REP-RULE-090 | RM-090 |
| — | REP-RULE-091 | RM-091 |
| — | REP-RULE-100 | RM-100 |
| — | REP-RULE-101 | RM-101 |
| — | REP-RULE-110 | RM-110 |
| — | REP-RULE-111 | RM-111 |
| — | REP-RULE-112 | RM-112 |
| — | REP-RULE-113 | RM-113 |
| — | REP-RULE-120 | RM-120 |
| — | REP-RULE-121 | RM-121 |
| — | REP-RULE-122 | RM-122 |
| — | MNT-RULE-010 | RM-200 |
| — | MNT-RULE-011 | RM-201 |
| — | MNT-RULE-012 | RM-202 |
| — | MNT-RULE-013 | RM-203 |
| — | MNT-RULE-014 | RM-204 |
| — | MNT-RULE-015 | RM-205 |
| — | MNT-RULE-016 | RM-206 |
| — | MNT-RULE-017 | RM-207 |
| — | MNT-RULE-018 | RM-208 |
| — | MNT-RULE-019 | RM-209 |
| — | MNT-RULE-019A | RM-210 |
| — | MNT-RULE-020 | RM-220 |
| M-02 | MNT-RULE-021 | RM-221 |
| — | MNT-RULE-022 | RM-222 |
| — | MNT-RULE-023 | RM-223 |
| — | MNT-RULE-024 | RM-224 |
| — | MNT-RULE-025 | RM-225 |
| — | MNT-RULE-026 | RM-226 |
| — | MNT-RULE-027 | RM-227 |
| — | MNT-RULE-028 | RM-228 |
| — | MNT-RULE-029 | RM-229 |
| — | MNT-RULE-029A | RM-230 |
| — | MNT-RULE-035 | RM-235 |
| — | MNT-RULE-036 | RM-236 |
| — | MNT-RULE-037 | RM-237 |
| — | MNT-RULE-038 | RM-238 |
| — | MNT-RULE-039 | RM-239 |
| — | MNT-RULE-040 | RM-240 |
| — | MNT-RULE-041 | RM-241 |
| — | MNT-RULE-042 | RM-242 |
| — | MNT-RULE-050 | RM-250 |
| — | MNT-RULE-051 | RM-251 |
| — | MNT-RULE-052 | RM-252 |
| — | MNT-RULE-053 | RM-253 |
| — | MNT-RULE-054 | RM-254 |
| — | MNT-RULE-055 | RM-255 |
| — | MNT-RULE-056 | RM-256 |
| — | MNT-RULE-057 | RM-257 |
| — | MNT-RULE-060 | RM-260 |

The first scheme numbered only the three rules that its own text referenced; the two blanks in its column are not omissions but rules that scheme never named.

## Reconciliation notes

1. **The deletion message of an Equipment Category.** One earlier text spelled the apostrophe of "can't" as a plain vertical apostrophe, the other as a typographic right single quotation mark. The system emits the typographic right single quotation mark; `RM-200` reproduces that form.
2. **The location at which the owner of the repaired product is tested.** Both earlier texts described the test, but only one noticed that the availability is read at the component source location while the movement runs from the product source location. The observed behaviour is recorded in `RM-026` and marked a **compatibility finding**, with the corrected behaviour stated.
3. **The unit-change refusal message.** One earlier text inserted a space after the full stop in "can not be done.If you want to change it". The system emits no space there; `RM-090` reproduces the message exactly and says so.
4. **The scope of the negative-quantity guard.** One earlier text placed the guard on the whole operated set, the other on the single repair being confirmed. The guard is evaluated on the single repair whose Confirm Repair operation was invoked, because that operation accepts one subject only; `RM-010` states it that way, while `RM-020` and `RM-030`, whose operations do accept a set, state the set form.
