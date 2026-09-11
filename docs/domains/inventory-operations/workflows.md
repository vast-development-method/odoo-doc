# Inventory Operations — Workflows

Each workflow below is an end-to-end sequence: who performs it, what must be true before it starts, the numbered steps, and the records created or updated at each step. Where a step runs an algorithm, the algorithm is named and located in `calculations.md` rather than repeated.

> **Reproduced literals.** A few strings in this file are reproduced exactly as the system emits them — error messages, selection labels, generated record names — and therefore keep abbreviations that this specification would otherwise spell out. They are: `UoM` for unit of measure, `SN` for serial number, `ZPL` for the Zebra printer command language, `PDF` for Portable Document Format, `GS1` for Global Standards One, and the suffix `(MTO)` for make to order, that is the supply method this specification calls *advanced* or *trigger another rule*. Wherever such a string is quoted, the quotation is verbatim and must be reproduced character for character.

Roles used in this file:

| Role | Security group | Typical actions |
|---|---|---|
| Inventory user | `stock.group_stock_user` | Create, confirm, reserve, validate, scrap and count. |
| Inventory manager | `stock.group_stock_manager` | Everything a user does, plus configuration, deleting quantity records and forcing counts. |
| Any internal user | `base.group_user` | Read the documents their record rules allow. |
| Automation | — | The scheduled jobs and the rule engine act with elevated rights. |

---

# 1. Setting up a warehouse

**Performed by:** an inventory manager.
**Precondition:** the company exists and has a contact.

1. The manager opens the warehouse screen and enters a name and a short name of at most five characters.
2. On save, the system creates the view Location named with the short name, then the five sublocations (stock, input, quality control, output, packing) with the active flags implied by the default one-step receipt and one-step delivery configuration, and assigns the barcodes derived from the short name where they are free.
3. The system inserts the Warehouse row and then creates the eight numbering sequences and the eight Operation Types described in `entities.md`, section 5.6. The receipt type and the delivery type are set as each other's return Operation Type.
4. The system creates the receipt Route and the delivery Route and their Stock Rules from the step configuration (section 2 below), and the supply-on-order rule inside the global replenish-on-order Route.
5. For every Warehouse listed as a supplier, the system creates a resupply Route (section 3).
6. The contact given as the Warehouse address has its customer stock Location and vendor stock Location rewritten to the company's internal transit Location.
7. The Warehouse identifier is stamped on the view Location and its direct children.
8. The multi-warehouse group is granted to every internal user when any company now has more than one active Warehouse; granting it also grants the multi-location group.

**Records created:** one Warehouse, six Locations, eight numbering sequences, eight Operation Types, two Routes with their Stock Rules, one supply-on-order Stock Rule, and one Route plus three or four Stock Rules per supplying Warehouse.

---

# 2. Warehouse step configuration

The receipt step configuration and the delivery step configuration decide which Locations are active, which Operation Types are active, and exactly which Stock Rules the two generated Routes contain.

## 2.1 The rule sets

Let *S* be the shared vendor Location and *C* be the shared customer Location.

**Receipts.**

| Configuration | Route name | Rules, in order |
|---|---|---|
| One step | "*warehouse name*: Receive in 1 step (stock)" | pull, from *S* to the stock Location, through the receipt Operation Type |
| Two steps | "*warehouse name*: Receive in 2 steps (input + stock)" | 1. pull, from *S* to the stock Location, through the receipt Operation Type. 2. push, from the input Location to the stock Location, through the storage Operation Type |
| Three steps | "*warehouse name*: Receive in 3 steps (input + quality + stock)" | 1. pull, from *S* to the stock Location, through the receipt Operation Type. 2. push, from the input Location to the quality control Location, through the quality control Operation Type. 3. push, from the quality control Location to the stock Location, through the storage Operation Type |

**Deliveries.**

| Configuration | Route name | Rules, in order |
|---|---|---|
| One step | "*warehouse name*: Deliver in 1 step (ship)" | pull, from the stock Location to *C*, through the delivery Operation Type |
| Two steps | "*warehouse name*: Deliver in 2 steps (pick + ship)" | 1. pull, from the stock Location to *C*, through the pick Operation Type. 2. push, from the output Location to *C*, through the delivery Operation Type |
| Three steps | "*warehouse name*: Deliver in 3 steps (pick + pack + ship)" | 1. pull, from the stock Location to *C*, through the pick Operation Type. 2. push, from the packing Location to the output Location, through the pack Operation Type. 3. push, from the output Location to *C*, through the delivery Operation Type |

The generated rules always take these values: the name "*short name*: *source location name* → *destination location name*"; the action as listed; manual automatic-move mode; the listed Operation Type; the Warehouse and its company; the supply method take-from-stock for the **first** rule of the list and advanced (make to order) for every later one. Receipt rules additionally carry the propagate-cancel flag, except the **last** rule of the chain, which has it cleared so that cancelling an intermediate step does not cancel the step that leaves the warehouse. Delivery rules additionally carry the propagate-carrier flag.

Both generated Routes are selectable on product categories and on the Warehouse, and not selectable on products. The receipt Route has sequence 50 and the delivery Route has sequence 60.

## 2.2 Changing the configuration

**Performed by:** an inventory manager.

1. The manager changes the receipt steps, the delivery steps or both on the Warehouse.
2. Missing Locations are re-created if any were deleted.
3. Location activity is rewritten: the quality control Location is active exactly for three-step receipts; the input Location is active for any receipt configuration other than one step; the packing Location is active exactly for three-step deliveries; the output Location is active for any delivery configuration other than one step.
4. The resupply consequences are evaluated (section 3.3).
5. The Operation Types are rewritten: default Locations, activity flags and barcodes as listed in `entities.md`, section 5.6. In particular the receipt type's destination becomes the input Location (or the stock Location for a one-step receipt) and the delivery type's source becomes the output Location (or the stock Location for a one-step delivery).
6. The affected Route is rewritten: its name is regenerated, **all of its existing rules are archived**, and the rule list of the new configuration is applied by looking for an archived rule with the same Operation Type, source Location, destination Location, Route and action and re-activating it, or creating it when none exists. This is why switching back and forth between two configurations reuses the same rule records.
7. The supply-on-order rule is rewritten to point from the stock Location to the destination of the first delivery rule, through that rule's Operation Type, and is renamed "*short name*: *source location name* → *destination location name* (MTO)".

**Records updated:** the Warehouse, four Locations, up to eight Operation Types, two Routes, and the Stock Rules of those Routes plus the supply-on-order rule.

---

# 3. Resupply between warehouses

## 3.1 Creating a resupply link

**Performed by:** an inventory manager.

1. The manager adds a supplying Warehouse to the supplied Warehouse's resupply list.
2. The system looks for an archived Route between the same two Warehouses and un-archives it; only when none exists does it create a new one.
3. Creating a new one performs, for each supplying Warehouse:
   1. Choose the transit Location: the company's internal transit Location when both Warehouses belong to the same company, otherwise the shared inter-company Location. Skip the supplier entirely when neither exists. Activate the chosen Location.
   2. Determine the supplying Warehouse's output Location: its stock Location for a one-step delivery, otherwise its output Location.
   3. When the supplying Warehouse delivers in one step, create an extra supply-on-order rule from that output Location to the transit Location through the supplying Warehouse's delivery Operation Type, named with the suffix "(MTO)". (For multi-step deliveries such a rule already exists.)
   4. Create the Route, named "*supplied warehouse name*: Supply Product from *supplying warehouse name*", selectable on the Warehouse, on products and on product categories, carrying both Warehouse links and the company shared by the two Warehouses.
   5. Create the pull rule from the supplying Warehouse's output Location to the transit Location, through the supplying Warehouse's delivery Operation Type, with the destination taken from the rule.
   6. When the supplying Warehouse delivers in more than one step, also create the pull rule from its stock Location to its output Location through its pick Operation Type.
   7. Create the pull rule from the transit Location to the supplied Warehouse's stock Location through the supplied Warehouse's receipt Operation Type.
   8. Each of those pull rules is given the supply method take-from-stock when its source Location is the supplying Warehouse's own stock Location, and advanced otherwise.

## 3.2 Removing a resupply link

The Routes between the two Warehouses are archived, which archives their rules.

## 3.3 Consequences of changing the delivery steps of a supplying warehouse

Only a change that crosses the one-step boundary matters.

**From one step to several.**

1. Every rule of every Route supplied by this Warehouse whose destination Location has transit usage is re-pointed at the new output Location and given the advanced supply method.
2. The archived "stock to output" rules of those Routes are re-activated; for the Routes that have none, one is created through the pick Operation Type.
3. Every supply-on-order rule of the global replenish-on-order Route that goes from this Warehouse's stock Location to a transit Location is archived, so that it cannot be chosen any more.

**From several to one step.**

1. Every such rule is re-pointed at the stock Location and given the take-from-stock supply method.
2. The "stock to output" rules through the pick Operation Type are archived.
3. One supply-on-order rule is created per distinct transit destination, from the stock Location through the delivery Operation Type, with the suffix "(MTO)".

---

# 4. Receiving goods in one step

**Performed by:** an inventory user.
**Precondition:** an Operation Type of kind receipt exists.

1. The user creates a Transfer, choosing the receipt Operation Type. The reference is drawn from that type's sequence (for example `WH/IN/00007`). The source Location defaults to the shared vendor Location — replaced by the contact's own vendor Location when the contact defines one — and the destination Location to the type's default destination.
2. The user adds one Stock Move per product with a demand. Each move inherits the Transfer's Locations, Operation Type, contact and scheduled date.
3. The user confirms. Each draft move is confirmed; because the source Location bypasses reservation, each move is immediately reserved (branch A of the reservation algorithm) and therefore becomes assigned, and the Transfer becomes ready. Detail lines are created without touching any reserved counter, then pushed through put-away, so their destination Location may be a sublocation of the Transfer's destination.
4. The user records what actually arrived: for each move, a processed quantity, and for tracked products a lot or serial number per detail line.
5. The user presses Validate. The validation algorithm of `calculations.md`, section 20, runs: sanity check, picked marking, backorder decision, completion.
6. Completion moves the goods: each detail line decreases nothing at the vendor Location (which bypasses reservation but is still written, producing a negative quantity record there) and increases the destination Location's quantity record, stamping the current instant as the incoming date.
7. Push rules are evaluated; with a one-step receipt there is none, so nothing more is created.
8. The re-reservation search runs, so that outgoing moves waiting for these products become ready.

**Records created or updated:** one Transfer, one Stock Move and at least one Stock Move Line per product, one or more Stock Quantity records per (product, destination Location, lot, container, owner), and a backorder Transfer when part of the demand was not received.

---

# 5. Receiving goods in two steps

**Performed by:** an inventory user.
**Precondition:** the Warehouse receives in two steps, so the input Location and the storage Operation Type are active and the receipt Route contains the pull rule and the push rule of section 2.1.

1. The user creates and confirms a receipt Transfer exactly as in section 4. Because the receipt Operation Type's default destination is now the **input** Location, the moves target the input Location.
2. The user validates the receipt. The goods land in the input Location.
3. During completion, the push step runs (`calculations.md`, section 18). The push rule from the input Location to the stock Location matches. Its automatic-move mode is manual, so a **new** Stock Move is created:
   - source Location: the completed move's destination Location (the input Location);
   - destination Location: the rule's destination Location (the stock Location), overridden by the final Location when that is a descendant of it;
   - date: the completed move's date plus the rule's lead time;
   - Operation Type: the storage Operation Type;
   - supply method: advanced;
   - the completed move is recorded as its originating move, because the input Location does not bypass reservation.
4. The new move is confirmed. Confirmation groups it into a Transfer: the system looks for an existing, unprinted, open Transfer with the same document references, the same source and destination Locations and the same Operation Type; when none is found it creates one, drawing its reference from the storage Operation Type's sequence (for example `WH/STOR/00003`).
5. The new move's status is waiting-another-move until the receipt move is done — which it already is — so the reservation of the destination moves that runs at the end of the completion immediately reserves it against the goods just received. Branch C of the reservation algorithm applies: the distribution map is built from the completed lines, so the storage move reserves precisely the Locations, lots and containers the receipt put the goods into.
6. The user validates the storage Transfer. The goods move from the input Location to the stock Location. Put-away may direct each line to a different shelf.

**Records created:** two Transfers, two Stock Moves per product linked as originating and destination, detail lines on both, and quantity records first in the input Location then in the stock Location.

---

# 6. Receiving goods in three steps

Identical to section 5 with one more hop. The receipt Transfer targets the input Location. Its completion pushes to the quality control Location through the quality control Operation Type; the completion of that Transfer pushes to the stock Location through the storage Operation Type. The chain is therefore: receipt move → quality control move → storage move, each the originating move of the next, each in its own Transfer numbered from its own Operation Type sequence (`WH/IN/...`, `WH/QC/...`, `WH/STOR/...`).

The propagate-cancel flag is set on the first two rules and cleared on the last, so cancelling the receipt cancels the quality control step, and cancelling the quality control step cancels nothing further.

---

# 7. Delivering goods in one step

**Performed by:** an inventory user.

1. A delivery Transfer is created, by hand or by the rule engine on behalf of another domain. Its Operation Type is the delivery type; its source Location is the stock Location and its destination Location is the shared customer Location, replaced by the contact's own customer Location when the contact defines one.
2. Moves are added and the Transfer is confirmed. The source Location does **not** bypass reservation, so each move becomes confirmed rather than assigned, and the reservation is attempted according to the Operation Type's reservation method:
   - at confirmation: immediately;
   - manually: never, until a person presses the availability button;
   - before the scheduled date: immediately when the computed reservation date is on or before today, otherwise by the scheduled job.
3. The reservation runs branch B of the reservation algorithm: gather at the stock Location with loose matching under the applicable removal strategy, take what is available, create the detail lines, raise the reserved counters.
4. The Transfer's status follows: ready when the shipping policy is as-soon-as-possible and at least one line is reserved, or when the policy is all-at-once and everything is reserved; waiting otherwise.
5. The user records the picked quantities and presses Validate.
6. Completion decreases the stock Location's quantity records and increases the customer Location's (which goes negative, as customer Locations are never counted), releases the reservations and stamps the date.
7. The confirmation message is posted using the company's delivery template when the company asks for it; the confirmation text message is sent when the company asks for it and the contact has a telephone number.

---

# 8. Delivering goods in two steps

1. The need arrives at the customer Location. The rule engine matches the first rule of the delivery Route — pull, stock Location to customer Location, through the **pick** Operation Type — and creates a move whose source Location is the stock Location, whose intermediate destination is the pick Operation Type's default destination (the output Location) and whose final Location is the customer Location.
2. That move is grouped into a pick Transfer (`WH/PICK/...`).
3. When the pick Transfer is validated, the push step matches the second rule — push, output Location to customer Location, through the delivery Operation Type — and creates the delivery move from the output Location to the customer Location, linked as a destination move.
4. The delivery move is grouped into a delivery Transfer (`WH/OUT/...`) and is reserved against exactly what the pick delivered.
5. Validating the delivery Transfer sends the goods out.

---

# 9. Delivering goods in three steps

**Performed by:** an inventory user.

1. The need arrives at the customer Location. The first rule — pull, stock Location to customer Location, through the **pick** Operation Type — creates the pick move. Its intermediate destination is the pick Operation Type's default destination, which for a three-step delivery is the **packing** Location; its final Location is the customer Location.
2. The pick move is grouped into a pick Transfer (`WH/PICK/...`). It reserves from the stock Location under the removal strategy.
3. Validating the pick Transfer moves the goods into the packing Location and pushes: the second rule — push, packing Location to output Location, through the **pack** Operation Type — creates the pack move, grouped into a pack Transfer (`WH/PACK/...`), linked as a destination move of the pick move, with the advanced supply method so that it consumes exactly what the pick brought.
4. The person packing uses the put-in-pack action (section 16) to create containers; each container becomes the destination container of the lines it holds.
5. Validating the pack Transfer moves the goods (and their containers) into the output Location and pushes: the third rule — push, output Location to customer Location, through the **delivery** Operation Type — creates the delivery move, grouped into a delivery Transfer (`WH/OUT/...`).
6. The delivery move reserves against the packed goods. Because whole containers were created, the whole-container detection marks each line with its container as destination container and as an entire package, so the delivery screen shows containers rather than loose products.
7. Validating the delivery Transfer sends the goods out. The container history snapshots are written before the containers move, so the delivery document can still be printed afterwards.

**Chain:** pick move → pack move → delivery move, three Transfers, three sequences, one shared set of document references.

---

# 10. Confirming a transfer

**Performed by:** an inventory user, or automatically when a move flagged as additional is added to an open Transfer.

1. Run the company consistency check on the Transfer.
2. Confirm every move whose status is draft (`state-machines.md`, section 1.2):
   - a move with originating moves, or with the advanced supply method, becomes waiting-another-move; the advanced ones also raise a supply request;
   - a move created by a take-from-stock-else-trigger rule becomes waiting and raises a supply request for the part the forecast does not cover;
   - every other move becomes waiting.
3. Moves whose Operation Type reserves at confirmation get today's date as their reservation date.
4. Moves with an Operation Type and no Transfer are grouped into Transfers.
5. The company consistency check runs again over the moves.
6. Unless merging was disabled, the moves are merged (`calculations.md`, section 13).
7. Negative moves are turned into returns: their source and destination Locations are swapped, their final Location becomes their old source, their chain links are re-wired by matching Locations, their demand is negated, their Operation Type becomes the return Operation Type when one is defined, and their supply method becomes take-from-stock. They are then grouped into Transfers. Before that, any negative move that still has a distinct final Location is pushed first.
8. The moves that bypass reservation, and those that are eligible for automatic reservation, are reserved.
9. The replenishment scheduler is triggered for the moves that are forecast short.

---

# 11. Checking availability (reserving on demand)

**Performed by:** an inventory user.

1. A Transfer in draft is confirmed first.
2. The moves that are neither draft, cancelled nor done are collected and sorted by: priority descending, then having a deadline before not having one, then deadline ascending, then date ascending, then identifier ascending.
3. When there are none, the action fails with "Nothing to check the availability for."
4. The reservation algorithm runs over that ordered set. The order matters: urgent and earliest-deadline moves take the scarce goods first.

---

# 12. Unreserving

**Performed by:** an inventory user.

1. The user presses the unreserve action on a Transfer, or the system unreserves implicitly (when the demand of a reserved move is raised beyond what is reserved, when a chained move upstream is edited, when a return is created, when a move is cancelled).
2. The unreserve algorithm of `calculations.md`, section 6, runs: picked lines survive, every other line is deleted and its reserved counter is given back, and the statuses are recomputed.

---

# 13. Validating a transfer

**Performed by:** an inventory user.
**Precondition:** the Transfer is not already done.

The complete sequence is the validation algorithm of `calculations.md`, section 20. In narrative form:

1. **Immediate transfer.** A Transfer still in draft is confirmed, and every move with a demand but no processed quantity has its demand copied into its processed quantity. This is what allows a person to create a Transfer and validate it in one gesture.
2. **Sanity check.** Empty Transfers, zero-quantity Transfers and tracked products without a lot are refused, with the exact messages listed in `business-rules.md`, section 4.
3. **Picked marking.** When quantities exist but nothing is marked as picked, every move is marked picked, so that a person who simply typed quantities does not also have to tick boxes.
4. **Backorder question.** When the Operation Type's policy is "ask" and at least one move is short, the backorder screen opens listing the Transfers concerned and, when more than one Transfer is being validated, one switch per Transfer. Answering resumes the validation.
5. **Text-message warning.** The first time a company validates a delivery with text-message confirmation on, a one-time warning screen offers to send or not to send; the answer is remembered on the company.
6. **Completion.** The moves are completed (`calculations.md`, section 16). Goods move, reservations are released, statuses become done, the completion instant is stamped, the priority is reset to normal.
7. **Backorder creation.** The unprocessed moves are carried into a new Transfer whose back-order link points at the original, with a note on the original: "The backorder *link* has been created."
8. **Downstream.** Push rules create the next step; destination moves are re-reserved; the re-reservation search runs for receipts and internal transfers.
9. **Communication.** The delivery confirmation message and text message are sent for deliveries when the company asks for them. Inter-company containers are unpacked when configured.
10. **Printing and reporting.** The automatic print actions are collected and returned; the Reception Report is opened when the Operation Type asks for it and something remains to allocate.

---

# 14. Partial processing and backorders

## 14.1 Partial delivery with a backorder

**Performed by:** an inventory user.

1. The Transfer demands 10 units; only 6 are available and reserved.
2. The user validates. The backorder decision finds the picked quantity (6) below the demand (10) and the policy is "ask", so the backorder screen opens.
3. The user chooses to create the backorder.
4. Completion runs with backorders allowed:
   - the backorder-move step splits each short move, reducing the original move's demand to 6 and producing a new move of 4 with the same supply method, the same chain links, the same unit price and the same deadline;
   - the original move completes and becomes done;
   - the Transfer backorder is created, receives the 4-unit move, clears its picked flags and its responsible, and is reserved when its Operation Type reserves at confirmation.
5. The original Transfer becomes done with a demand of 6; the backorder is open with a demand of 4.

## 14.2 Partial delivery without a backorder

1. Same starting point.
2. At the backorder screen the user chooses "No backorder", which re-runs the validation declaring the Transfer as not to be backordered. The same happens automatically when the Operation Type's policy is "never".
3. Completion runs with backorders forbidden:
   - the pruning step cancels every move whose processed quantity is zero or which is not picked;
   - no backorder move is split off, so the short move keeps its demand of 10 and its processed quantity of 6;
   - no backorder Transfer is created.
4. The original Transfer becomes done. The 4 missing units are simply never delivered.

## 14.3 Splitting a transfer without validating it

**Performed by:** an inventory user.

1. The user fills the processed quantities they want to keep on this Transfer and presses the split action.
2. The action refuses in three cases: every move has a zero processed quantity — "*the transfer reference*: Nothing to split. Fill the quantities you want in a new transfer in the done quantities"; every move is fully processed — "*the transfer reference*: Nothing to split, all demand is done. For split you need at least one line not fully fulfilled"; at least one move is over-processed — "*the transfer reference*: Can't split: quantities done can't be above demand".
3. Otherwise the moves that are neither done nor cancelled and have a non-zero processed quantity are split, the resulting backorder moves are collected together with the moves whose processed quantity is zero, and a backorder Transfer is created for exactly that set.
4. Nothing is completed: both Transfers remain open.

---

# 15. Returning goods

**Performed by:** an inventory user.
**Precondition:** the Transfer is done, else "You may only return Done pickings."

1. The user opens the return screen from the completed Transfer. Only one Transfer at a time may be returned: "You may only return one picking at a time."
2. The screen lists one line per move of the Transfer that is not cancelled and whose destination Location usage is not inventory loss, each with a quantity of zero by default. When no such move exists: "No products to return (only lines in Done state and not fully returned yet can be returned)."
3. The user types the quantities to return. Pressing the "return all" action instead fills each line with the move's processed quantity reduced by the quantities of the returns already made against it.
4. On confirmation, the destination moves of the moves being returned that are neither done nor cancelled are unreserved.
5. A new Transfer is created by copying the original, with: no moves, no detail lines, the original as return link, the origin "Return of *the original reference*", the source Location equal to the original's destination Location, the destination Location equal to the return Operation Type's default destination when that type is a receipt and to the original's source Location otherwise, and the return Operation Type when one is defined. Its responsible is cleared. A note linking it to the original is posted.
6. For each line with a non-zero quantity, the original move is copied with: the quantity as demand, the new Transfer, status draft, the current instant as date, the new Transfer's Locations, no final Location, the new Operation Type and its Warehouse, the original move as the original return move, the take-from-stock supply method, and the original Transfer's document references. When a return line has no original move, a plain new move is created instead.
7. The new move is wired into the chain so that a return of an intermediate step behaves correctly:
   - **originating moves** = the returns already made of the original move's destination moves, plus the original move itself, plus the non-cancelled originating moves of the original move's non-cancelled destination moves;
   - **destination moves** = the returns already made of the original move's originating moves, plus the non-cancelled destination moves of the non-cancelled originating moves of those already-made returns.
8. When no line had a non-zero quantity: "Please specify at least one non-zero quantity."
9. The return Transfer is confirmed and its availability is checked.

## 15.1 Exchange

1. The return is created exactly as above.
2. **For a receipt being returned**, a second Transfer is created by copying the return with the same preparation, its lines are re-created from the same quantities, the original-return links and originating links of its moves are cleared so that the exchange is independent, and it is confirmed and reserved. It is recorded as the return of the return.
3. **For any other kind**, no second Transfer is copied; instead one supply request per return line is raised at the original move's intermediate destination Location (or the Transfer's destination Location), carrying the Transfer's document references, the move's date, the Warehouse, the contact, the final Location and the company. The rule engine then produces the replacement documents through the normal routes.

## 15.2 Backorder policy on a return

A Transfer that is itself a return ignores its Operation Type's backorder policy: the "should ignore backorders" test is true whenever the return link is set.

---

# 16. Put in pack

**Performed by:** an inventory user.

1. The user selects lines (or presses the action on the whole Transfer or batch) and triggers put-in-pack. A done or cancelled Transfer refuses silently.
2. The selection is split into lines to pack and containers to pack (`calculations.md`, section 10.6). Picked lines win over unpicked ones when both exist.
3. When the lines point at more than one destination Location, the destination chooser opens; the chosen Location is written on all of them.
4. When the Operation Type asks to set a container type and nothing was supplied, the put-in-pack screen opens offering an existing container, a container type or a free name.
5. The container is created (drawing its name from the container type's sequence when no name was given, otherwise from the general container sequence) or taken.
6. When exactly one line is being packed, put-away is re-run for that line with the container, so the container is directed at a Location that can accept it.
7. The container is written as the destination container of the lines.
8. When the Operation Type asks to print the container label, the label document is returned.
9. When containers rather than lines were selected, the new container becomes their destination container, the stale links of the previous chain are cleared, and put-away is re-run on the new container's lines.

---

# 17. Moving a whole package

**Performed by:** an inventory user.

There are three distinct ways a whole container travels.

## 17.1 Adding an existing container to a transfer

1. From a Transfer that is neither done nor cancelled, the user opens the container list restricted to the containers located inside the Transfer's source Location and picks one.
2. The system collects that container and all of its descendants, deletes any existing detail line of the Transfer that already drew from one of them, and creates one detail line per contained quantity record with: the record's product, quantity, unit, Location, lot, container and owner, the Transfer's destination Location, the container as **both** source and destination container, and the entire-package flag.
3. Put-away is applied to those lines.
4. The container promotion of `calculations.md`, section 10.4, runs so that, if the whole parent container was added, the parent is taken over too.

## 17.2 Automatic detection during reservation

Every reservation pass re-runs the whole-container detection (`calculations.md`, section 10.3): when the lines of a single Transfer reproduce exactly the contents of a source container, and that container's type is not reusable, each of those lines is given the container as destination container and flagged as an entire package. This is what makes a delivery that happens to reserve a full pallet show one pallet instead of forty boxes.

## 17.3 Validation

At completion, the container history snapshots are taken first, then the quantities move with the container as their destination container, then the destination containers are applied to the containers themselves (`calculations.md`, section 10.5), which is what physically re-parents the container tree. Two failures can stop this: containers of the same group landing in different Locations, and a destination container that already holds goods elsewhere.

---

# 18. Relocating quantities without a transfer

**Performed by:** an inventory user.

1. The user selects quantity records and triggers the relocation action. It refuses when the selection spans more than one company, when any record has no company, or when any record has a non-positive quantity: "You can only move positive quantities stored in locations used by a single company per relocation."
2. The relocation screen asks for a destination Location, a destination container and a label (default "Quantity Relocated").
3. On confirmation, one adjustment-style move is created per record: the record's quantity, from its Location to the chosen Location (or the same Location when only the container changes), from its container to the chosen container, with the label as the move's reference, already picked, status confirmed, and one detail line carrying the same values.
4. When no destination container was chosen and the records are not being unpacked, each record keeps its own container, and the container's parent chain is carried along as far as the whole parent is being moved: a parent container is added to the move only when every one of its contained records is part of the selection, and the walk stops at the containers named as the upper limit.
5. The moves are completed immediately.

The same mechanism is used when a person writes a new Location directly on a container (the container's contents are relocated with the label "Package manually relocated"), when a person writes a new Location on a Lot (label "Lot/Serial Number Relocated"), and when a container is unpacked (label "Quantities unpacked").

---

# 19. Scrapping

**Performed by:** an inventory user.

## 19.1 Standalone scrap

1. The user creates a Scrap, choosing a product, a quantity, a source Location of internal usage and a scrap Location of inventory-loss usage. For a tracked product a lot is chosen; a serial number already located elsewhere raises the warning of `business-rules.md`, section 8.
2. The user validates. A zero quantity is refused: "You can only enter positive quantities."
3. The availability check runs: the on-hand quantity of the product at the exact source characteristics (Location, lot, container, owner, strict matching) must be at least the scrap quantity converted into the product unit. When it is not, the shortage screen opens naming the product: "*the product display name*: Insufficient Quantity To Scrap"; the person may confirm anyway.
4. On confirmation, the reference is drawn from the scrap sequence (prefix `SP/`, padding 5), one Stock Move is created already picked from the source Location to the scrap Location with one detail line carrying the lot, the container and the owner, and that move is completed with the scrap marker set so that **no backorder is created**.
5. The Scrap becomes done and the completion instant is stamped.
6. When the replenish switch is on, a supply request for the scrapped quantity at the source Location is run, which lets the normal routes re-order the goods.

## 19.2 Scrap from a transfer

1. From a Transfer, the user opens the scrap screen; it is pre-filled with the Transfer, and the product list is restricted to the products of the Transfer's moves that are neither draft nor cancelled.
2. The source Location defaults to the Transfer's destination Location when the Transfer is done, and to its source Location otherwise.
3. The created move carries the Transfer, so the scrap appears among the Transfer's moves and makes its "has scrap moves" indicator true.
4. Because the move's destination Location has inventory-loss usage, the Transfer's status computation treats it apart: a Transfer all of whose done moves are scraps and which also has a move cancelled for another reason becomes cancelled rather than done.

---

# 20. Counting stock

## 20.1 Entering a count

**Performed by:** an inventory user.

1. The user opens the physical inventory screen. Opening it runs the record housekeeping (merge, clean reservations, delete empties) unless the skip parameter is set, and puts the screen in counting mode. A user who is not a manager sees only the records assigned to them.
2. For each record the user types a counted quantity. Writing it turns the counted flag on and makes the difference field equal to counted minus on hand.
3. The user may instead press "set current quantity", which copies the on-hand quantity into the counted quantity and assigns the record to themselves. When some records already have a counted quantity, a warning screen appears first.
4. Records whose on-hand quantity moved after the count was typed show as outdated.

## 20.2 Applying a count

1. The user presses Apply on a selection, or "apply all" which first asks for a free reference label (default "Physical Inventory") and a counting date (default the current instant); "apply all" then restricts itself to the records whose counted flag is set.
2. When any selected record is outdated, the conflict screen opens listing the whole set and, separately, the outdated subset. The person chooses between two readings of their own count:
   - **Keep counted quantity** — the number they wrote is the truth: each record's difference is rewritten as `counted − on hand` and the counts are applied.
   - **Keep difference** — the *correction* they intended is the truth: each record's counted quantity is rewritten as `on hand + recorded difference` and the counts are applied on top of whatever moved meanwhile.
3. Applying runs the algorithm of `calculations.md`, section 23.2: one adjustment move per record, from or to the inventory-loss Location, completed immediately with destination containers ignored, followed by the re-reservation search, the stamping of the last-count date on the Locations, the recomputation of the next-count dates, and the clearing of the counted fields. When a counting date was given, it is written as the date of every created move.

## 20.3 Requesting a count

**Performed by:** an inventory manager.

1. The manager selects records and opens the count-request screen.
2. The screen asks for a scheduled date (required, defaulting to the current instant) and optionally a person to assign it to. It also exposes a switch that reads and writes the system parameter deciding whether the counting screen shows the theoretical quantity beside the counted one.
3. On confirmation the set of records is **extended**: when the lot group is active and at least one selected record carries a tracked product, every sibling record sharing the same (product, Location) pair is added, so that counting one lot forces the whole product at that Location to be counted.
4. The scheduled date, and the assignee when one was chosen, are written on every record of the extended set, in counting mode. The counted quantity is not touched.

## 20.4 Reverting a count

**Performed by:** an inventory user.

From the history of adjustment detail lines, the user selects lines and presses revert. One mirror move per line is created and completed, with the reference "*the original reference* [reverted]". When no selected line qualifies, the notification "There are no inventory adjustments to revert." is shown.

## 20.5 Creating stock from scratch

A record created in counting mode is special-cased: only the allowed fields may be given, else "Quant's creation is restricted, you can't do this operation." The system first gathers an existing record with the same strict characteristics and, when one exists, writes on it instead of creating a duplicate; when a lot was given, only records with a lot are considered. The counted quantity is then set (which triggers the adjustment) or, when the auto-apply field was used, written and applied at once.

---

# 21. Automatic reservation job

**Performed by:** automation, once a day (see `configuration.md`, section 6).

1. Select the moves whose status is confirmed, waiting-another-move or partially available, whose supply method is take-from-stock, and whose reservation date is on or before today.
2. Order them by priority descending, then date ascending, then identifier ascending.
3. Reserve them in that order.

This is what makes the "reserve a fixed number of days before the scheduled date" policy work.

---

# 22. Batch transfers

## 22.1 Creating a batch by hand

**Performed by:** an inventory user.

1. The user selects Transfers in a list and triggers "add to batch".
2. The screen offers an existing batch or a new one, with a responsible and a description.
3. The Transfers are attached. The batch takes its Operation Type from the first attached Transfer when it had none. The composition check runs and refuses Transfers whose state or Operation Type does not fit.
4. Attaching a Transfer to a batch that has a responsible re-assigns that responsible on the Transfer and posts a note on it.

## 22.2 Creating a wave by hand

1. The user selects detail lines and triggers "add to wave".
2. The screen offers an existing wave or a new one.
3. For each Transfer contributing lines: when every line and every move of the Transfer is being taken and no taken move has a zero quantity, the whole Transfer is simply attached to the wave. Otherwise the Transfer is **split**: a copy of it is created with the wave as batch, and for each move either the whole move is re-pointed at the copy (when all of its lines are taken) or the move is split by the taken quantity and the new move, carrying the taken lines, is created on the copy.
4. When the Operation Type auto-confirms, the wave is confirmed.

## 22.3 Automatic batching

**Performed by:** automation, at the moment a Transfer is confirmed and again for each backorder created at validation.

1. Skip when the Operation Type does not ask for automatic batching, when no batch grouping option is chosen, when the Transfer already belongs to a batch, when it has no moves, or when it is not ready.
2. Skip when the Transfer alone would already exceed the maximum-lines limit, or when the maximum-transfers limit is one or less.
3. Search the existing batches of the same Operation Type and company that are not waves, whose state is draft (or draft and in-progress when the Operation Type auto-confirms), and which match every chosen grouping criterion — contact, contact's country, source Location, destination Location — and which are not among the batches currently being validated. Attach the Transfer to the first one that can still absorb it without exceeding the limits.
4. When no batch fits, search another unbatched, ready Transfer of the same Operation Type, company and grouping values, and create a new batch holding both. Confirm it when the Operation Type auto-confirms.
5. When no such Transfer exists either, create a batch holding this Transfer alone, with the Transfer's responsible, and confirm it when the Operation Type auto-confirms.
6. The batch's description is built from the chosen criteria: the contact's name, the contact's country name, the source Location display name and the destination Location display name, joined by commas.

## 22.4 Automatic waving

**Performed by:** automation, for the lines of the backorders created at validation.

1. Keep the lines that are waveable: they belong to a Transfer, that Transfer is ready and the line has a non-zero quantity (unless the caller waives this), the line is not already in a wave, the Operation Type asks for wave grouping, and — when grouping by category is on — the product's category is among the configured categories.
2. When grouping by Location is on, find for each line the deepest configured wave Location that is an ancestor of the line's source Location (the configured Locations are scanned from the deepest to the shallowest); a line with no such ancestor is not waveable.
3. **Into existing waves.** For each Operation Type, search the waves of the same company matching the batch-level criteria and, when auto-confirm is on, any non-final state (otherwise only draft). When grouping by Location is on, keep only the waves all of whose line Locations sit under one configured Location, and remember that Location per wave. For each line, walk the waves and take the first one whose contact, country, source Location, destination Location, product, category and wave Location all match, and which can still absorb the line's move, Transfer and weight within the limits. Add the line to it.
4. **Into new waves.** The lines that found no wave are grouped among themselves by the same criteria: for each line, collect the other candidate lines that match, sort them by Transfer then move to minimise splits, and fill successive new waves up to the limits. Lines that match nothing get a wave of their own.
5. The description of a generated wave is the batch description followed by the product display name, the category full name and the wave Location full name, according to the criteria in use.

## 22.5 Validating a batch

**Performed by:** an inventory user.

1. Take the Transfers of the batch that are neither cancelled nor done.
2. Detach the *empty waiting* ones: those in waiting or waiting-another-operation state where every open move is unpicked or has a zero quantity, and those in ready state where every open move has a zero quantity.
3. Among the rest, find the *empty* ones (every open move unpicked or zero). When not all of the remaining Transfers are empty, detach those too, so that the partially processed ones can be validated without cancelling the untouched ones.
4. Run the sanity check once over the whole remaining set rather than per Transfer, so that a lot typed on one Transfer of the batch satisfies the check for the batch.
5. Post a note on each validated Transfer: "**Transferred by:** Batch Transfer *link to the batch*". Post a note on the batch naming the detached Transfers.
6. Validate the remaining Transfers together with the sanity check suppressed and with the detached list and the current batch recorded in the context.
7. After validation, the detached Transfers lose their batch link and their zero-quantity moves lose the picked flag; the backorders created are offered to automatic batching and automatic waving.
8. A Transfer that ends up done while others of its batch are not is detached from the batch, so that the batch status stays coherent.

## 22.6 Merging batches

1. At least two must be selected: "Please select at least two batch/wave transfers to merge."
2. They must share one Operation Type: "Batch/Wave transfers with different operation types cannot be merged."
3. They must be all batches or all waves: "Batch transfers cannot be merged with wave transfers and vice versa."
4. They must share one state: "Batch/Wave transfers with different states cannot be merged."
5. That state may be neither done nor cancelled: "You cannot merge done or cancelled batch/wave transfers."
6. The first selected batch becomes the target; the detail lines and Transfers of the others are moved into it; the responsible, description and scheduled date (and, with dispatch management, the vehicle and dock) of the **earliest-scheduled** batch are written onto the target; the others are deleted.

## 22.7 Dispatch

1. The Operation Type is marked as using dispatch management and a list of dock Locations is configured on it.
2. A batch is given a vehicle; its category, its capacities and its driver follow automatically.
3. A dock is chosen, or set automatically when every Transfer of the batch shares one source Location that is an allowed dock.
4. Setting the dock rewrites the moves: for receipts and internal transfers the destination Location becomes the dock; for deliveries the source Location becomes the dock. Clearing it restores each move to its Transfer's own Location.
5. The Transfers of the batch are ordered by the contact's postal code, and that order is stamped into their batch sequence, so that the printed batch document lists them along the delivery round.
6. The weight and volume load percentages are displayed against the vehicle category's capacities.

---

# 23. Reception report

**Performed by:** an inventory user, from a receipt or internal Transfer, or from a batch.

1. The report is built as described in `calculations.md`, section 25.1. It lists, grouped by the source document of each demand, the incoming quantities that could cover it.
2. The user presses Assign on a line. The assignment algorithm runs: the demand is split when only part of it is being covered, the incoming moves become its originating moves, its supply method becomes advanced, the document references are shared both ways, and the reservation is re-run so that an already-received quantity is immediately locked to that demand.
3. The user presses Unassign to undo it: the links are removed, the references are un-shared, the demand is split again if part of it is still linked, its supply method returns to take-from-stock and it is unreserved.
4. The user may print labels for the allocated moves; the number of labels per move is its demand rounded up to the next whole number.

---

# 24. Traceability

**Performed by:** any internal user with read access.

1. From a Lot, a product, a container or a detail line, the user opens the traceability screen.
2. The tree is built by walking completed detail lines upstream and downstream (`calculations.md`, section 26), including the production genealogy links when they exist.
3. Each node shows the reference, the date, the product, the lot, the quantity, the source and destination Locations and the document.
4. The tree can be printed.

Separately, a Lot exposes the outgoing Transfers that finally carried it, found by the delivery-discovery walk of `calculations.md`, section 26.1; from there the contacts who received it are listed.

---

# 25. Cancelling

**Performed by:** an inventory user.

1. The user presses Cancel on a Transfer.
2. Every move of the Transfer is cancelled (`state-machines.md`, section 1.4), which unreserves them, propagates the cancellation along the chain according to each move's propagate-cancel flag, and schedules the warning activity on the upstream documents.
3. The Transfer is locked.
4. A Transfer with no moves at all is written directly to cancelled.

Cancelling a done move is impossible: "You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place."

---

# 26. Editing a completed transfer

**Performed by:** an inventory user, after unlocking.

1. The user presses the lock toggle to unlock the Transfer.
2. The user changes a processed quantity, a lot, a container, an owner or a Location on a detail line.
3. Each change replays the movement (`calculations.md`, section 7.2): the original movement is undone on the quantity records, the new one is applied, other reservations are freed if the source goes negative, and the downstream moves are unreserved and re-reserved.
4. A note describing the change is posted on the Transfer.
5. Deleting a completed line is never allowed; the quantity must be set to zero instead.

---

# 27. Printing and communicating

1. **On demand.** The user prints the transfer document, the delivery document, the container content document, a container label, product labels or lot labels from the print menu (see `interfaces.md`, section 5). Printing the transfer document sets the printed flag.
2. **Automatically at validation.** The Operation Type's automatic-print switches are read and the corresponding documents are queued; when several apply, they are returned as one multi-print action, with the Reception Report as the follow-up action when it also applies.
3. **Delivery confirmation message.** For a delivery, when the company asks for email confirmation, the company's delivery template is rendered and posted in the Transfer's thread with the light notification layout and the comment subtype, forcing immediate sending.
4. **Delivery confirmation text message.** For a delivery whose contact has a telephone number, when the company's text-message validation is on, the company's text-message template is rendered and sent to the contact directly, not queued.
5. **Signature.** Writing a signature on a Transfer renders the delivery document, attaches it to a message in the thread and posts "Order signed by *the contact name*".

---

# 28. Barcode-driven flows (data contract)

The interactive scanning screens belong to a companion capability; this domain supplies the data they rely on.

1. **Location barcodes.** Unique per company; scanning one resolves the Location.
2. **Operation Type barcodes.** Generated from the Warehouse short name plus a suffix; scanning one opens that Operation Type's list.
3. **Container barcodes.** A container's name may itself be a serial shipping container code; the validity indicator says whether it is. Scanning a container that is reusable adds its **contents** to the Transfer; scanning a disposable container adds the container itself.
4. **Aggregate barcodes.** A set of quantity records can be rendered as one or more aggregate barcodes:
   1. Read the maximum length parameter (default 400) and the separator parameter. Without a separator, produce nothing.
   2. Collect the structured barcode application identifiers of every barcode rule that names a unit of measure other than the plain unit, keyed by that unit; the identifier is the rule pattern's characters two to four followed by the number of decimals implied by the unit's rounding.
   3. For each record in order: skip products with no barcode; when the product changes and its barcode is not a valid structured barcode, start the encoding with the raw product barcode; then append the record's own structured barcode, or, failing that, the serial number alone for a serial-tracked record.
   4. A record's structured barcode is: the identifier `01` followed by the product barcode left-padded with zeroes to fourteen digits, when the product's barcode is a valid one; then, unless the record is serial-tracked with a quantity of one, either the unit's quantity identifier followed by the quantity divided by the unit's rounding step, left-padded to six digits, or the identifier `30` followed by the rounded quantity left-padded to eight digits; then, when a lot exists and its name is at most twenty characters, the identifier `21` for a serial number or `10` for a lot, followed by the name. A lot name longer than twenty characters makes the whole record produce nothing.
   5. Concatenate with the separator, start a new aggregate barcode whenever the maximum length would be exceeded, and terminate every aggregate barcode with a tabulation character.

---

# 29. Internal transfers

**Performed by:** an inventory user.
**Precondition:** the multi-location group is active, so the internal-transfer Operation Type is available.

1. The user creates a Transfer with the internal Operation Type. Both default Locations are the Warehouse's stock Location, so the user changes at least one of them to a sublocation.
2. Moves are added and the Transfer is confirmed.
3. Because the source Location is internal, the moves are reserved under the removal strategy; the detail lines carry the exact sublocations the goods were found in.
4. Put-away applies to the destination side: each line's destination Location may be redirected to a sublocation of the Transfer's destination Location.
5. Validation moves the quantities between the two internal Locations. No Location goes negative unless the goods were not really there.
6. The completion triggers the re-reservation search, exactly as a receipt does, because the Operation Type kind is internal.

---

# 30. Cross docking

**Performed by:** automation and an inventory user.
**Precondition:** the Warehouse both receives in more than one step and delivers in more than one step, so the cross-dock Operation Type is active.

The cross-dock Operation Type moves goods straight from the input Location to the output Location, skipping the stock Location. It is created and kept active by the Warehouse generation, but no rule of the two generated Routes uses it: it is offered so that a person, or a companion capability, can build a rule that bypasses storage.

---

# 31. Consignment

**Performed by:** an inventory user.
**Precondition:** the owner group is active.

1. The owner field appears on Stock Quantity records, on Stock Move Lines and, as an owner restriction, on Stock Moves.
2. A Transfer may carry an owner; at validation that owner is written onto every move as its owner restriction and onto every detail line as its owner.
3. Gathering distinguishes owners: a strict gathering for owner *X* never returns a record owned by *Y* or by nobody, and a loose gathering that names an owner requires an exact match on it.
4. Quantity records of different owners in the same Location for the same product are therefore separate records and are reserved separately.

---

# 32. Working with several companies

**Performed by:** an inventory manager.

1. Each company has its own Warehouses, Operation Types and Locations. The shared vendor, customer and inter-company Locations have no company and are visible to all.
2. A resupply between two Warehouses of **different** companies uses the shared inter-company Location rather than the company's own transit Location; the shared Location is activated the first time such a Route is created.
3. When goods travel between companies, the sending company's delivery lands in the transit Location and the receiving company's receipt takes them out of it. The two halves are separate documents in separate ledgers.
4. The Transfer's contact is used to detect the counterpart company: the company whose contact is that contact or one of its ancestors. When that company differs from the Transfer's own, and the parameter is set, the destination containers of the transit-bound moves are unpacked so that the receiving company does not inherit the sender's containers.
5. Every entity of the domain carries a record rule restricting it to the reader's enabled companies; the ones that also accept an empty company are listed in `configuration.md`, section 6.

---

# 33. Changing a product's tracking mode

**Performed by:** an inventory manager, through `../products-and-catalog/`.

This domain reacts to the change:

1. Turning tracking **on** does not retroactively create lots: existing quantity records keep an empty lot and are still gathered, because gathering accepts an empty lot in both matching modes.
2. Turning tracking **off** is refused at the settings level while any product is tracked; at product level the change is allowed and the existing lot-bearing records simply stop being distinguished.
3. Switching a product from non-storable to storable creates a reserved quantity record for every already-reserved move of it, because the detail lines now have to be backed by counters.

---

# 34. Handling negative stock

**Performed by:** an inventory user, usually without meaning to.

A negative on-hand quantity arises whenever goods leave a Location that did not hold them: a delivery validated before its receipt, an over-processed delivery, a scrap confirmed through the shortage screen.

1. The completion writes the negative figure onto the quantity record rather than refusing.
2. When the available quantity at that key goes below zero, the reservation-freeing routine of `calculations.md`, section 7.4, takes the promised goods back from other open documents, current Transfer first.
3. The negative record coexists with any positive records of the same product in the same Location that differ by lot, container or owner.
4. From then on, the negative pocket mechanism of the reservation-quantity computation makes the positive records absorb the negative one before anything can be reserved out of them.
5. When the missing receipt is finally validated, the arrival is written onto the same key and the negative figure returns to zero; the housekeeping pass then deletes the now-empty record.
6. A lot-bearing negative is additionally compensated at completion time against untracked stock at the same Location, so that the lot-level figures stay coherent.

The whole mechanism is deliberate: the system never blocks a physical movement because its paperwork arrived in the wrong order, and it repairs itself once the paperwork catches up.

---

# 35. Emptying a location

**Performed by:** an inventory user.

1. Read the Location's emptiness indicator: it is true when the sum of on-hand quantities of the records directly in it is at most zero.
2. When it is not empty, either relocate the records (section 18) or count them to zero (section 20).
3. Only then can the Location be archived; archiving a parent archives the whole subtree and is refused when any internal descendant still holds stock.

---

# 36. Transferring responsibility for a transfer

1. Writing the responsible on a Transfer is tracked in its discussion thread.
2. Writing the responsible on a Batch Transfer re-assigns every Transfer of the batch and posts a note on each, naming the batch.
3. A backorder is always created with an empty responsible, so that it is picked up afresh.
4. A return is likewise created with an empty responsible.

---

# 37. Reading a transfer's history after the fact

**Performed by:** any internal user with read access.

| Question | Where the answer is |
|---|---|
| What was actually moved? | The Transfer's detail lines, each with its quantity, lot, containers and exact Locations, and each with the instant it was completed. |
| What was demanded? | The Transfer's moves, whose demand was reduced by whatever went into the backorder. |
| What was **originally** demanded? | The aggregation of `calculations.md`, section 24.9, which walks the backorder chain and adds the demands back. |
| Which containers travelled, and inside what? | The Package History snapshots, which freeze the container tree as it was. |
| Where did the goods come from? | The traceability tree, walking upstream. |
| Where did they go? | The traceability tree, walking downstream; for a lot, the delivery discovery. |
| Who changed what after the fact? | The notes posted in the discussion thread by each edit of a done line. |
| What did it weigh? | The stored shipping weight, frozen at validation unless a person rewrites it. |

---

# 38. Decision tables

These tables collapse the branching of the main workflows into a form that can be checked line by line.

## 38.1 What happens to a move at validation

| Picked | Processed quantity | Adjustment move | Backorders allowed | Demand | Outcome |
|---|---|---|---|---|---|
| no | 0 | no | yes | > 0 | Not completed; carried whole into the backorder |
| no | 0 | no | no | > 0 | Cancelled |
| no | 0 | no | either | 0 | Cancelled |
| no | > 0 | no | yes | > 0 | Not completed; carried whole into the backorder |
| no | > 0 | no | no | > 0 | Cancelled |
| yes | 0 | no | yes | > 0 | Not completed; carried whole into the backorder |
| yes | 0 | no | no | > 0 | Cancelled |
| yes | > 0 | no | yes | > processed | Completed for the processed quantity; a backorder move of the remainder is split off |
| yes | > 0 | no | yes | = processed | Completed; no backorder move |
| yes | > 0 | no | yes | < processed | Completed for the processed quantity; no backorder move (over-processing) |
| yes | > 0 | no | no | > processed | Completed for the processed quantity; the demand is left as it was; no backorder |
| either | any | yes | either | any | Completed; adjustment moves are exempt from the pruning and from the backorder split |

## 38.2 Whether the backorder question is asked

| Operation Type policy | Any move short or unpicked | Transfer is a return | Question asked | Backorder created |
|---|---|---|---|---|
| ask | no | either | no | no |
| ask | yes | no | **yes** | as answered |
| ask | yes | yes | no | yes |
| always | either | either | no | yes |
| never | either | either | no | no |

A move counts as *short or unpicked* when it has a non-zero demand and is not picked, or when its picked quantity is strictly below its demand at the `Product Unit` precision.

## 38.3 Which branch of the reservation runs

| Source Location bypasses reservation | Product storable | Has originating moves | Branch |
|---|---|---|---|
| yes | either | either | A — create lines without touching counters |
| no | no | either | A — the product itself bypasses |
| no | yes | no | B — gather at the source Location |
| no | yes | yes | C — distribute what the predecessors brought |

## 38.4 Which name a container shows

| Context asked for | Name shown |
|---|---|
| the record is done | the plain name |
| the destination path | the greater-than-joined path of destination containers |
| the source path | the greater-than-joined path of parent containers |
| nothing in particular | the plain name |
| the formatted form, and the type has all three dimensions | the chosen name, a tabulation, then the three dimensions between double hyphens |

## 38.5 Which Location a Transfer resolves to

| Operation Type kind | Default source | Default destination | Overridden by the contact when |
|---|---|---|---|
| receipt | the shared vendor Location | the input Location, or the stock Location for a one-step receipt | the source default has vendor usage and the contact defines its own vendor Location |
| delivery | the output Location, or the stock Location for a one-step delivery | the shared customer Location | the destination default has customer usage and the contact defines its own customer Location |
| internal | the stock Location | the stock Location | never |

The override only applies when the contact's own Location differs from the system-wide default for that field, so a contact that merely inherits the shared Location changes nothing.

## 38.6 Which document a validation prints

| Operation Type switch | Document | Extra condition |
|---|---|---|
| Auto Print Delivery Slip | the delivery document | — |
| Auto Print Return Slip | the return label | — |
| Auto Print Reception Report | the reception report | the kind is not delivery and the moves have destination moves; the reader is in the reception-report group |
| Auto Print Reception Report Labels | one label per destination move | the kind is not delivery; the reader is in the reception-report group |
| Auto Print Product Labels | product labels in the configured format | — |
| Auto Print Lot/SN Labels | lot labels in the configured format | the reader is in the lot group and the Transfer has lots |
| Auto Print Packages | the container document | the reader is in the container group and the Transfer has destination containers |
| Show Reception Report at Validation | the reception report screen | the reader is in the reception-report group and something remains to allocate |

## 38.7 Which lot rule applies at completion

| Product tracking | Operation Type creates lots | Operation Type uses existing lots | Line has a Lot | Line has a typed name | Outcome |
|---|---|---|---|---|---|
| none | — | — | — | — | Nothing required |
| lot or serial | — | — | yes | — | Accepted as is |
| lot or serial | no | no | no | — | Accepted **without** a lot |
| lot or serial | yes | either | no | yes | The Lot is found or created and linked |
| lot or serial | yes | either | no | no | Refused |
| lot or serial | no | yes | no | — | Refused |

A line whose move has **no** Operation Type at all, is not an adjustment move, has no Lot and does not belong to a Scrap is refused immediately, before any of the above is evaluated.

## 38.8 Which record the gathering returns first

| Strategy | First record |
|---|---|
| first in first out | the oldest incoming date; ties broken by the lowest identifier |
| last in first out | the newest incoming date; ties broken by the highest identifier |
| closest location | the alphabetically first full location name; ties broken by the highest identifier |
| least packages | inside the chosen container set, the oldest incoming date |

and in every case, among records that would otherwise tie, one carrying a lot comes before one carrying none.

---

# 39. Failure recovery

What to do, and what the system does by itself, when each thing goes wrong.

| Symptom | Cause | Recovery |
|---|---|---|
| A move stays `confirmed` although stock exists | Its Operation Type reserves manually, or its reservation date is in the future | Press the availability action, or change the reservation method |
| A move stays `waiting` although its predecessor is done | The predecessor's completion did not re-reserve it — typically because the no-auto-reserve parameter is set | Press the availability action |
| The reserved counters do not match the lines | A crash between two writes | The clean-reservations pass repairs it; it runs daily and whenever the quantity screens are opened |
| Two identical quantity records exist | Concurrent reservations | The merge pass collapses them |
| A quantity record sits at zero forever | Its assignee was never cleared | Clear the count; the empty-record pass then deletes it |
| A Location cannot be archived | It or a descendant still holds stock | Relocate or count the stock to zero first |
| A Warehouse cannot be archived | Open moves of its Operation Types, or foreign Operation Types using its Locations | Finish or cancel the moves; re-point the foreign Operation Types |
| A negative quantity record persists | A delivery was validated before its receipt | Validate the receipt; the record returns to zero and is deleted |
| A serial number appears in two places | A delivery was validated before its receipt | Same; the warning explains it resolves itself |
| A container is stuck as a destination container of nothing | A chain was broken by a removal | The removal already clears the chain where it is empty; otherwise remove the container from the Transfer again |
| A batch keeps a Transfer that is already done | Only possible if the automatic detachment was bypassed | Remove the Transfer from the batch by hand |
| A backorder was created that should not have been | The policy is "ask" and the wrong answer was given | Cancel the backorder |
| A backorder was **not** created that should have been | The policy is "never", or the Transfer is a return | Create a new Transfer by hand |
| Goods went to the wrong shelf | A put-away rule or a capacity limit sent them there | Relocate the quantity records; then fix the rule |
| A count was applied against stale figures | The conflict screen was answered with "Keep counted quantity" while a movement was in flight | Count again |
