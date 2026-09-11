# Workflows of the Inventory Valuation and Costing domain

Each workflow is given as: who performs it, what must be true before it starts, the
numbered steps, what records are created or changed at each step, and what is true when
it ends. Cross-references point at the formulas and the journal entries.

Roles referred to throughout:

| Role | Meaning |
|---|---|
| Inventory user | May create and validate transfers and see quantities. May not see values. |
| Inventory manager | Everything the inventory user may do, plus: read accounts and journals, see the monetary value on stock quantity records, see the unit cost history report, create and validate landed cost documents, adjust the value of a movement. |
| Accounting user | May create and post invoices and bills. Gains read and write access to transfers and goods movements. |
| Accounting read-only user | May read transfers, goods movements and the unit cost history report. |
| Accounting manager | Everything the accounting user may do, plus the work-in-progress accounting wizard. |
| System | The scheduled job runner, acting as the root user. |

---

## 1. Configuring valuation for a product family

**Performed by.** Inventory manager together with an accounting manager (the valuation
mode field is readable only by an accounting read-only user or an inventory manager).

**Preconditions.** A chart of accounts is installed for the company. The company has an
inventory journal and an inventory valuation account (both are set by the chart of
accounts installation; see [configuration.md](configuration.md#installation)).

**Steps.**

1. Open the product category. Set the **costing method**. The selection defaults to the
   company's fallback costing method for a new category.
2. Set the **valuation mode**. Leaving it empty means "use the company's".
3. Set the **inventory valuation account**. Leaving it empty means "use the
   company-level fallback, then the company's account".
4. On that account, set the **variation account** — the counterpart the closing entry
   will use. Without it, the closing falls back to the company's default expense account,
   and if that is empty too the account is silently skipped by the closing.
5. Under the continental perpetual arrangement only, also set the **closing expense
   account** on the inventory valuation account, so that part three of the closing can
   post the period variation.
6. Optionally set the **inventory journal** on the category; otherwise the company's is
   used.
7. If the costing method is `standard` and the valuation mode is `real_time`, set the
   **price difference account**; the form only shows the field in that combination.
8. Save.

**What changes.** If the costing method changed, every product of the category has its
unit cost recomputed with the new method, and every lot of every lot-valuated product of
the category is recomputed too (see
[calculations.md](calculations.md#14-costing-method-change)). The change to the costing
method and to the valuation mode is recorded in the category's message history.

**Postconditions.** Every product filed under the category now reports the new costing
method and valuation mode, per company.

**Failure conditions.** None specific to this workflow; account references are restricted
to accounts of the same company and cannot be deleted while referenced.

---

## 2. Turning on valuation by lot or serial number

**Performed by.** Inventory manager.

**Preconditions.** The product's tracking mode is not `none`.

**Steps.**

1. Open the product template and tick **valuation by lot or serial number**. The field is
   hidden while the tracking mode is `none`; the interface asks for confirmation before
   applying it.
2. On save, the system searches for stock quantity records of the product's variants that
   sit inside the valued perimeter, carry no lot or serial number and hold a non-zero
   quantity.
3. If any exist, the save is refused with **"You cannot enable lot valuation because the
   following products have on-hand quantities without a lot/serial number:"** followed by
   the display names of the offending products. Clear those quantities (by assigning them
   a lot through an inventory adjustment, or by shipping them) and try again.
4. On success, every variant is scheduled for a unit-cost recomputation and every lot of
   every variant for a lot-cost recomputation. Lots with no cost yet receive the
   product's unit cost; the rest are recomputed with the product's costing method.

**Postconditions.** From now on, an incoming movement of that product whose lines do not
all carry a lot or serial number is **refused** at completion with **"A lot/serial number
is required for product '_the product display name_' as it has lot valuation enabled."**
Outgoing movements are valued line by line at the lot's cost. The product's unit cost
becomes an aggregate: it is set to the product's computed average cost whenever a
recomputation runs.

---

## 3. Receiving goods

**Performed by.** Inventory user (validation) and, indirectly, the accounting user (the
bill).

**Preconditions.** A transfer whose destination is inside the valued perimeter and whose
source is outside it.

**Steps.**

1. The user validates the transfer. The generic completion routine runs.
2. **Before** the state change, the movements that *would* count as outgoing are valued.
   A receipt normally has none, so this step does nothing.
3. The state change happens; quantities are applied to the stock quantity records; the
   three classification flags are recomputed and the movement becomes **incoming**.
4. The movements that are now incoming or drop shipments are valued
   ([calculations.md](calculations.md#1-the-value-of-a-goods-movement-the-priority-chain)).
   Because no outgoing movement was valued in step 2, the **incremental fast path** is
   enabled for the unit-cost update.
   - If the product is valuated by lot and any line lacks a lot, the whole validation
     fails with the missing-lot error.
   - Otherwise the value is written on the movement.
5. The product's unit cost is updated
   ([calculations.md](calculations.md#3-maintaining-the-product-unit-cost)); the lots
   touched have their cost updated.
6. The valuation journal entry is built for the batch. A plain receipt from a vendor
   location into a warehouse produces **none**, because neither location carries a
   location valuation account.
7. Analytic lines are created or refreshed for the incoming and outgoing movements.

**Records created or changed.**

| Record | Change |
|---|---|
| Goods movement | value written; incoming flag set; remaining quantity becomes meaningful |
| Product | unit cost updated (average and first in first out only) |
| Lot | cost updated when the product is valuated by lot |
| Journal entry | only when a location valuation account is involved |
| Analytic lines | created or updated when an analytic distribution applies |

**Postconditions.** The product's total value has risen by the movement's value. Under
periodic valuation the ledger has not moved; the difference will appear at the next
closing. Under perpetual valuation the ledger has not moved either — it moves when the
**bill** is posted.

**Where the value came from.** For a receipt against a purchase order that has not yet
been billed, the value comes from the purchase order line
([calculations.md](calculations.md#14-value-from-a-purchase-order-line)). For a receipt
with no purchase order, it comes from the product's unit cost. For a receipt against a
purchase order that has already been billed, it comes from the bill
([calculations.md](calculations.md#13-value-from-vendor-bills)).

---

## 4. Posting the vendor bill after the receipt

**Performed by.** Accounting user.

**Preconditions.** A receipt has been validated and valued from the purchase order line.
The bill lines are linked to the same purchase order lines.

**Steps.**

1. The user posts the bill.
2. **Before** the generic posting: the cost-of-goods-sold lines for customer invoices are
   prepared (none, for a bill), and, when the purchasing integration is installed, the
   **price difference lines** are prepared for lines whose product uses the standard
   costing method under anglo-saxon accounting
   ([accounting-effects.md](accounting-effects.md#4-price-difference-at-the-vendor-bill)).
3. The generic posting runs. Under perpetual valuation, each eligible line's account was
   already redirected to the **inventory valuation account** when the line was built, so
   the bill debits the asset rather than an expense.
4. **After** the generic posting: every goods movement reachable from the bill's lines
   that is incoming or a drop shipment is **re-valued**. The bill now sits above the
   purchase order line in the priority chain, so the movement's value changes from the
   ordered price to the billed price.
5. Re-valuing those movements updates the product's unit cost again.

**Postconditions.**

- Under **first in first out** and **average cost**: the value of the receipt movement
  now equals the billed amount (net of what earlier movements of the same purchase order
  line already absorbed), converted into the company currency at the bill's own rate. The
  product's total value and unit cost reflect the real purchase price. No price
  difference is posted; the difference is absorbed into the goods.
- Under **standard price**: the value of the receipt movement is **not** changed by the
  bill in the way one might expect — the priority chain still puts the bill first, so the
  movement's value does follow the bill, but the *asset* is corrected by the price
  difference mechanism so that the ledger carries the standard cost and the difference
  sits in the price difference account.

**Failure conditions.** None specific; the generic posting's own validations apply
(balanced entry, open period, and so on).

---

## 5. Delivering goods

**Performed by.** Inventory user.

**Preconditions.** A transfer whose source is inside the valued perimeter and whose
destination is outside it.

**Steps.**

1. The user validates the transfer.
2. **Before** the state change, the movements that would count as outgoing are valued.
   This ordering is essential: the first in first out stack and the average unit cost must
   be read as they stand **before** the goods leave.
   - If the product is valuated by lot: the value is the sum over the lines of *line
     quantity × the lot's cost*, using the product's unit cost for a line with no lot.
   - Else if the costing method is `fifo`: the first in first out valuation of the
     movement's valued quantity, with the quantity already processed for that product in
     this same batch subtracted from the stack size.
   - Otherwise: *the product's unit cost × the valued quantity*.
3. The generic completion runs; the outgoing flag is set.
4. The incoming movements of the same batch (if any) are valued; the fast path is
   **disabled**, because an outgoing movement was valued in step 2.
5. The valuation journal entry is built for the batch. A plain delivery to a customer
   location produces **none**, because neither location carries a location valuation
   account.
6. For every product of the outgoing set that uses first in first out, or that uses
   average cost and is valuated by lot, the unit cost is recomputed.
7. Analytic lines are created or refreshed.

**Postconditions.** The product's total value has fallen. Under first in first out the
product's unit cost becomes *total value ÷ quantity on hand*, so a delivery **does** move
the reported cost of a first in first out product. Under average cost the unit cost does
not move.

---

## 6. Invoicing the customer

**Performed by.** Accounting user.

**Preconditions.** Perpetual valuation for the product. The sales integration links the
invoice lines to the sales order lines and thence to the deliveries.

**Steps.**

1. The user posts the customer invoice.
2. **Before** the generic posting: the cost-of-goods-sold lines are prepared for every
   eligible line
   ([accounting-effects.md](accounting-effects.md#2-cost-of-goods-sold-at-the-customer-invoice)).
   The unit price is computed by
   [calculations.md](calculations.md#92-cost-of-goods-sold-value-of-an-invoice-line),
   which nets out the cost already recognised on earlier invoices of the same sales order
   lines.
3. The generic posting runs. The invoice now carries four items instead of two: revenue,
   receivable, the expense debit and the inventory credit.
4. **After** the generic posting: every incoming or drop-shipment movement reachable from
   the lines is re-valued. For a customer invoice there usually are none.

**Postconditions.** The inventory valuation account has been credited by the cost of the
goods invoiced; the expense account has been debited by the same amount.

**Resetting to draft.** The two injected items are deleted. Posting again recomputes
them, which may produce a different amount if goods moved in the meantime.

---

## 7. Returning goods to the vendor

**Performed by.** Inventory user.

**Preconditions.** A validated receipt.

**Steps.**

1. From the receipt, the user opens the return wizard, sets the quantity to return and
   the **"update quantities on the order"** switch, and creates the return.
2. The switch is copied onto the created return movement.
3. The user validates the return transfer. The movement goes from inside the valued
   perimeter to the vendor location, so it counts as **outgoing** and is valued like any
   other outgoing movement — at the current cost under standard price and average cost,
   or by consuming the first in first out stack.

   > Note that the *return* source of the priority chain is only consulted for movements
   > whose **originating** movement is **outgoing**. A vendor return's originating
   > movement is a receipt, which is incoming, so the return source does not apply and
   > the goods are valued at today's cost, not at the price they were received at.

**Postconditions.** The product's total value falls by the outgoing value. If the goods
were received at 12.00 and the average cost has since fallen to 11.00, the return removes
11.00 per unit, and the 1.00 per unit difference stays in the value of the remaining
stock.

---

## 8. Accepting a customer return

**Performed by.** Inventory user.

**Preconditions.** A validated delivery.

**Steps.**

1. From the delivery, the user opens the return wizard and creates the return. The
   created movement carries the delivery movement as its **originating returned
   movement**.
2. The user validates the return transfer. The movement goes from the customer location
   into the valued perimeter, so it counts as **incoming**.
3. The value priority chain reaches the **return source**, because the originating
   movement is outgoing:

   ```formula
   value = origin_value × returned_quantity ÷ origin_valued_quantity
   ```

   The justification reads **"Value based on original move _the originating reference_"**.
4. The product's unit cost is updated; under average cost the returned goods re-enter at
   exactly the cost at which they left, so the average does not drift.

**Postconditions.** The delivery and the return cancel each other exactly in value, even
if the cost has moved since.

**Variation.** A return created independently of any delivery — a customer sending goods
back with no originating movement recorded — falls through to the product cost source and
enters at today's unit cost.

---

## 9. Scrapping goods

**Performed by.** Inventory user.

**Preconditions.** Goods on hand; a scrap location (usage `inventory`) which, for the
entry to be produced, carries a location valuation account.

**Steps.**

1. The user creates and validates a scrap. It produces a goods movement from the
   warehouse to the scrap location.
2. The scrap location is outside the valued perimeter, so the movement counts as
   **outgoing** and is valued by the costing method.
3. Under perpetual valuation, because the destination carries a location valuation
   account, a journal entry is produced debiting that account and crediting the product's
   inventory valuation account by the movement's value.

**Postconditions.** The goods have left the valuation; the loss sits on the scrap
location's account (perpetual) or is reported as a location reclassification at the next
closing (periodic).

---

## 10. Adjusting the counted quantity

**Performed by.** Inventory user, or inventory manager when an accounting date is needed.

**Preconditions.** Inventory counting mode.

**Steps.**

1. The user enters the counted quantity on the stock quantity records.
2. Optionally the user sets an **accounting date** on the record, or supplies one in the
   naming wizard. The wizard only shows the field when at least one selected product uses
   perpetual valuation.
3. The user applies the adjustment. The records are grouped by accounting date; each
   group with a date runs with that date forced as the accounting period date, and the
   field is cleared afterwards.
4. For each record, a goods movement is produced between the warehouse location and the
   inventory-loss location, in the direction dictated by the sign of the difference. When
   an accounting period date is forced and no explicit name was supplied, the movement's
   inventory name is built as described in
   [entities.md](entities.md#inventory-adjustment-behaviour).
5. The movement completes and is valued:
   - an **increase** is incoming, valued at the product's unit cost (there is no purchase
     order, no bill and no originating movement, so the chain falls through to the
     product cost source);
   - a **decrease** is outgoing, valued by the costing method.
6. Under perpetual valuation, and because the inventory-loss location carries a location
   valuation account, a journal entry is produced; its date is the forced accounting
   period date when one is in effect.

**Postconditions.** Quantity and value both move. The adjustment always uses the
product's **current averaged cost** as the unit price, even under first in first out —
because an increase goes through the product cost source of the chain, and a decrease
under first in first out consumes the stack, which for a decrease is exactly the first in
first out cost.

---

## 11. Changing the unit cost of a product

**Performed by.** Any user with write access to the product (in practice an inventory
manager or a purchase manager).

**Preconditions.** The product's costing method is not `fifo` — for a first in first out
product the write is accepted but produces no valuation history record, because the cost
is a derived figure.

**Steps.**

1. The user writes the new unit cost on the product.
2. The old unit cost of each product in the write is remembered.
3. The write happens.
4. The unit-cost change handler runs
   ([entities.md](entities.md#unit-cost-change-handler)): for each product whose costing
   method is not `fifo` and whose cost actually changed, a **valuation history record** is
   created carrying the new unit cost, the effective instant, the acting user and the
   description **"Price update from _the old price_ to _the new price_ by _the user
   name_"**.
5. For a product valuated by lot, the new unit cost is written onto every lot, with the
   automatic-revaluation flag disabled so that the lots do not each create their own
   record.

**Postconditions.**

- The product's **total value** changes immediately: under standard price it becomes
  *quantity on hand × new cost*; under average cost the next replay anchors on the new
  record and produces *quantity at the anchor instant × new cost* as its starting value.
- The **remaining value** of every incoming movement of the product changes too, because
  for a non-first-in-first-out product it is *remaining quantity × the product's unit
  cost*.
- **No journal entry is produced.** The difference surfaces at the next closing.
- The change appears as an "Adjustment" row in the unit cost history report.

**Worked sequence.** A product using average cost holds 20 units worth 60.00 at a unit
cost of 3.00. The user writes 4.00. A valuation history record is created with value
4.00. The total value becomes 80.00. The next closing debits the inventory valuation
account and credits the variation account by 20.00.

---

## 12. Adjusting the value of one completed movement

**Performed by.** Inventory manager.

**Preconditions.** Exactly one completed movement is selected. Selecting more than one
fails with **"You can only adjust valuation for one move at a time."**

**Steps.**

1. From the valuation list (or from the "Adjust Valuation" contextual action on a
   movement), the user opens the adjustment dialog. It shows: the movement's current
   value; the detail line **"For _the quantity_ _the unit_ (_the unit price_ per _the
   unit_)"**; the movement's current justification; and, when it differs, the
   justification the movement would have if manual corrections were ignored, prefixed
   **"Computed value: _the formatted value_"**.
2. The user types the **new value** (a total, not a unit price) and a description.
3. On save, a **valuation history record** is created carrying the movement, the new
   value, the movement's company, the current instant and the description.
4. Creating the record triggers a re-valuation of the movement. The manual correction now
   sits at the very top of the priority chain, claims the whole valued quantity, and
   **suppresses the landed cost source** — the value the user typed is understood to be
   the final value, landed costs included.
5. The product's unit cost is recomputed, and so are the lots when the product is
   valuated by lot.

**Postconditions.** The movement's value is exactly what the user typed. Its
justification reads **"Adjusted on _the date_ by _the user name_"** followed by the
description. The product's total value and unit cost have moved. **No journal entry is
produced and the entry already posted for that movement is not amended**; the difference
surfaces at the next closing.

**Reverting.** There is no "undo". The user creates a further correction with the
previous value; the latest record by date and identifier wins.

---

## 13. Creating and validating a landed cost from a vendor bill

**Performed by.** Inventory manager (to validate) and accounting user (to create from the
bill).

**Preconditions.** A posted or draft vendor bill containing at least one line flagged as
a landed cost line. A line becomes flagged automatically when its product carries the
"is a landed cost" flag; the flag is forced off for a line whose product is not a
service.

**Steps.**

1. On the bill, the **"Create Landed Costs"** button is offered while the bill has no
   landed cost document yet and at least one line is flagged.
2. Pressing it creates a landed cost document in the bill's company, with the bill as its
   vendor bill, and one cost line per flagged bill line carrying:
   - the product;
   - the product's name as the description;
   - the product's expense account;
   - the amount:

     ```formula
     amount = sign × round_to_currency( line_subtotal ÷ line_currency_rate )
     sign   = −1 for a vendor credit note, +1 otherwise
     ```

   - the split method: the product's default split method, else `equal`.
3. The interface opens the new document in form view.
4. The user selects the **transfers** (or, when manufacturing landed costs are installed
   and the target is set to manufacturing orders, the **manufacturing orders**) whose
   goods should receive the cost. The transfer selection is restricted to transfers of the
   same company having at least one incoming or outgoing movement.
5. The user presses **"Compute"**, which builds the valuation adjustment lines
   ([calculations.md](calculations.md#72-the-split-computation)). The lines show, per
   goods movement and per cost line, the quantity, weight, volume, original value,
   allocated amount and new value. The allocated amounts may be edited by hand.
6. The user presses **"Validate"**. The guards run
   ([state-machines.md](state-machines.md#13-guard-failures-in-detail)); if the document
   has no adjustment lines yet, the split is computed first; the sum check must pass.
7. The journal entry is built from the qualifying adjustment lines and posted
   ([accounting-effects.md](accounting-effects.md#5-landed-cost-entry)).
8. Every goods movement named by the adjustment lines is re-valued, which pulls the
   allocation into the movement's value through the extra source.
9. The document moves to `done` and a message is posted under the "Landed cost validated"
   subtype.

**Postconditions.** The value of the receipt movements has risen by the allocated
amounts; the products' unit costs have been recomputed; the inventory asset has been
debited by the part of the allocation that is still on hand. A "Landed Costs" button
appears on the bill.

---

## 14. Creating a landed cost by hand

**Performed by.** Inventory manager.

**Steps.**

1. Open the landed cost list and create a new document. It receives a sequence number of
   the shape `LC/<year>/<four digits>`.
2. Set the date, the journal (defaulted from the company's landed cost journal or the
   category inventory journal fallback), and, optionally, a vendor bill.
3. Add cost lines. Choosing a product fills the description, the split method, the amount
   and the account.
4. Select the transfers or manufacturing orders.
5. Compute, review, validate — as in [section 13](#13-creating-and-validating-a-landed-cost-from-a-vendor-bill)
   from step 5.

---

## 15. Reversing a posted landed cost

**Performed by.** Inventory manager.

**Preconditions.** A posted landed cost document. It cannot be cancelled or deleted.

**Steps.**

1. Create a new landed cost document targeting the same transfers.
2. Add cost lines whose amounts are the **negatives** of the original ones, with the same
   split methods and the same accounts.
3. Compute and validate.

**Postconditions.** The journal entry produced swaps its sides, so the inventory asset is
credited and the counterpart account debited. The extra source of the priority chain now
sums the positive and the negative allocations to zero, so the movements' values return
to what they were.

---

## 16. Running the inventory valuation closing manually

**Performed by.** Accounting manager (or any user who can read the report and post
entries).

**Preconditions.** The company has an inventory journal and an inventory valuation
account.

**Steps.**

1. Open the inventory valuation report. It shows, for the chosen date (defaulting to
   today):
   - **Initial Balance** — the posted ledger balance of each inventory valuation account;
   - **Ending Stock** — the physical value attributed to each of those accounts;
   - **Inventory Loss**, when at least one location of usage `inventory` carries a
     valuation account — the reclassification lines;
   - **Stock Variation** — the proposed balancing lines.
2. Optionally change the date through the date filter. Changing it reloads the report.
3. Press **"Generate Entry"**. The date is passed only when it differs from today.
4. The closing operation runs
   ([calculations.md](calculations.md#117-the-closing-operation)). Guards:
   - a date before the last closing date is refused;
   - nothing to post is refused with **"Everything is correctly closed"**;
   - a missing journal or valuation account is refused with its own message.
5. The entry is created **in draft** (the manual path does not auto-post), registered in
   the company's closing list, and opened in form view under the title **"Journal
   Items"**.
6. The user reviews and posts it.

**Postconditions.** Once posted, the entry becomes the anchor for the next closing: only
movements dated after the moment it was posted (or after its date) are considered next
time.

**Caution.** A closing left in draft is **not** an anchor. Running the closing again will
recompute the same period, and posting both would double-count. Post or delete a draft
closing before running another.

---

## 17. The automatic closing

**Performed by.** System, once a day.

**Steps.**

1. The scheduled job runs. It builds the list of periods to process: always `daily`, plus
   `monthly` when today is the last day of the month.
2. It selects every company whose inventory period is in that list.
3. For each, it runs the closing with automatic posting on, in a context that marks the
   run as coming from the scheduled job.
4. A company for which the closing raises a user-facing error — nothing to close, no
   journal, no account, or a date conflict — is **skipped silently** and the job moves on
   to the next company.

**Postconditions.** Every company with a daily period has a posted closing entry for the
day; every company with a monthly period has one on the last day of the month. Companies
with the `manual` period are never touched.

---

## 18. Valuing the stock as of a past date

**Performed by.** Any user who can see values.

**Steps.**

1. From the stock report, choose **"Inventory at Date"** and pick a date. The date is
   pushed into the context, which reaches both the quantity computation and the valuation
   computation.
2. A plain date is widened to the **last instant of that day**.
3. Every figure is recomputed as of that instant
   ([calculations.md](calculations.md#15-valuation-at-a-past-date)).

Alternatively, the inventory valuation report's own date filter recomputes the whole
report — physical value, ledger balance, reclassification and variation — as of that
date.

**Caution.** Deleting valuation history records changes past valuations: the average
replay anchors on the latest record, and with no record for a product it replays from the
beginning of time. Deleting records for *some* products of a batch disables the
optimisation for the whole batch rather than producing wrong numbers.

---

## 19. Posting a work-in-progress entry for manufacturing

**Performed by.** Accounting manager.

**Preconditions.** One or more manufacturing orders in the `confirmed`, `progress` or
`to_close` state.

**Steps.**

1. Select the orders and open the work-in-progress accounting wizard. Orders in any other
   state are filtered out.
2. The wizard proposes: the posting date (today), the reversal date (the day after), the
   journal (the company-level fallback of the category inventory journal), a reference
   built from the order names, and three lines
   ([accounting-effects.md](accounting-effects.md#8-work-in-progress-entry-and-its-reversal)).
3. Changing the date recomputes the lines — unless the wizard has lines and no orders, in
   which case a manually built entry is left alone.
4. The user may edit the lines. Writing a debit forces the credit to zero and vice versa;
   a line carrying both is refused by a database constraint with **"A single line cannot
   be both credit and debit."**
5. Press **"Confirm"**. Guards: the total credit must equal the total debit
   (**"Please make sure the total credit amount equals the total debit amount."**), and
   the reversal date must be after the posting date (**"Reversal date must be after the
   posting date."**).
6. The entry is created, linked to the manufacturing orders, and posted. Its reversal is
   created, linked to the same orders, dated at the reversal date, referenced **"Reversal
   of: _the original reference_"**, and posted.

**Postconditions.** Both entries appear on the manufacturing orders' work-in-progress
entry list and on each entry's manufacturing order list.

---

## 20. Completing a manufacturing order

**Performed by.** Manufacturing user; the accounting consequences are automatic.

**Steps.**

1. The order's cost computation runs
   ([calculations.md](calculations.md#81-the-cost-of-a-finished-good)), setting the unit
   price of the finished-goods movements and of the by-product movements.
2. The component movements complete. They go from the warehouse to the production
   location, so they count as **outgoing** and are valued by their own costing methods.
   Under perpetual valuation, because the production location carries a cost-of-production
   account, each produces a journal entry debiting that account.
3. The finished-goods movement completes. It goes from the production location into the
   warehouse, so it counts as **incoming**. Its value comes from the **production source**
   of the priority chain: *quantity × the unit price set in step 1*. Under perpetual
   valuation it produces a journal entry debiting the finished good's inventory valuation
   account and crediting the cost-of-production account.
4. Once the order reaches the completed state, the **labour entry** is posted
   ([accounting-effects.md](accounting-effects.md#7-manufacturing-labour-entry)), unless
   the labour was already posted or the work-centre cost is zero.

**Postconditions.** The cost-of-production account nets to zero once every component, the
labour and the finished good have passed through it.

---

## 21. Applying a landed cost to a manufacturing order

**Performed by.** Inventory manager, with manufacturing landed costs installed.

**Steps.**

1. Create a landed cost document and set its target to **manufacturing orders**.
2. Select the orders. The selection is restricted to orders of the same company having at
   least one incoming finished-goods movement.
3. The targeted movements are the finished-goods movements of those orders, **minus** the
   by-product movements whose cost share is zero — a by-product that takes no share of the
   cost also takes no share of the landed cost.
4. Compute and validate as usual.

**With subcontracting landed costs installed**, every targeted movement that is a
subcontracting movement is replaced by its **originating** movements, so the cost lands
on the movements that actually brought the goods in rather than on the subcontracting
movement itself. The transfer selection is correspondingly widened to include transfers
having a completed subcontracting movement.

---

## 22. Correcting the quantity of an already-completed movement

**Performed by.** Inventory manager.

**Preconditions.** The transfer is completed. The fiscal lock constraint must allow the
change (see [business-rules.md](business-rules.md#fiscal-lock-on-transfer-dates)).

**Steps.**

1. The user edits the quantity on a movement line of the completed movement, or adds a
   line.
2. Before the write, the quantities of the lines of movements currently flagged incoming
   or outgoing are remembered.
3. The write happens; the stock quantity records are updated by the inventory operations
   domain.
4. The re-valuation rule runs
   ([entities.md](entities.md#7-stock-quantity-stockquant-table-stock_quant)):
   - an **incoming** movement is fully re-valued from the priority chain — which, for a
     receipt against a purchase order, re-reads the order price for the new quantity;
   - an **outgoing** movement has its value **scaled** by the ratio of the new quantity to
     the previous one, rather than re-costed.
5. The products' unit costs are recomputed.
6. Analytic lines are refreshed.

**Postconditions.** The value follows the quantity. Under first in first out, increasing
the quantity of an old receipt adds the extra units at the **top** of the stack, because
the stack is rebuilt from the current quantity on hand rather than from a consumption
ledger (see
[calculations.md](calculations.md#55-worked-example-of-first-in-first-out-with-a-quantity-increase-after-the-fact)).

---

## 23. Recovering from negative stock

**Performed by.** Nobody — it is automatic. This section describes what an operator
should expect.

**Sequence.**

1. Goods are delivered before the corresponding receipt is recorded. The outgoing movement
   is valued at the last known cost (the product's unit cost, or an extrapolation from the
   last incoming movement under first in first out). The quantity on hand and the total
   value both go negative.
2. Under periodic valuation the next closing posts the negative variation, crediting the
   inventory valuation account.
3. The receipt is eventually recorded at the real price.
   - Under **average cost**: the replay's negative-recovery branch rebuilds the value as
     *the arriving unit cost × the resulting quantity*, which retrospectively re-prices
     the goods that went out.
   - Under **first in first out**: the value of the earlier outgoing movement is left
     alone, but the product's total value is rebuilt from the current stack, so the
     reported value is correct going forward.
4. The next closing posts the difference, bringing the inventory valuation account back to
   the physical value.

**What an operator should do.** Before the very first outgoing movement of a product that
has never been received, set a sensible unit cost on the product; otherwise the outgoing
movement is valued at zero and the correction, when it comes, is the whole amount.

---

## 24. Reading why a movement is worth what it is worth

**Performed by.** Inventory manager.

**Steps.**

1. Open the valuation list for the product (from the product list's total value cell, or
   from the moves analysis).
2. The **value description** column shows one line per contributing source, in the order
   the priority chain consulted them, for example:

   ```
   Adjusted on 2026-03-02 08:14:11 by Aurélie Dubois
   Correction after the customs re-assessment
   ```

   or

   ```
   1 200.00 for 100.00 Units from BILL/2026/03/0007
   Additional landed costs:
   + 60.00 from BILL/2026/03/0011 (Landed Cost: LC/2026/0004)
   ```

3. When a manual correction is in force, the **computed value description** column shows
   what the movement would be worth without it, prefixed **"Computed value: _the
   formatted value_"**.

For a product using average cost, the **unit cost history** report gives the same story
at product level: every movement and every manual cost change in date order, with the
running quantity, running value and running unit cost after each.

---

## 25. Installing the valuation feature on an existing database

**Performed by.** System, once, at installation.

**Steps.**

1. **Journals.** For every company that already has a chart of accounts, ordered by their
   position in the company tree: read the chart of accounts data, keep only the inventory
   journal and inventory valuation account entries, and either adopt an existing general
   journal with the code `STJ` of that company or create the journal named "Inventory
   Valuation" with code `STJ`, type general, sequence 10, hidden from the accounting
   dashboard. Load the data and apply it to the company.
2. **Initial costs.** For every company and every consumable-type product visible to it
   (a product with no owning company, or owned by that company), create a **valuation
   history record** carrying the product's unit cost in that company, today's date and the
   description **"Initial cost"**. This gives every product an anchor for the average
   replay and a dated cost for past valuations.
3. **Company accounts.** For every company with a chart of accounts, in tree order: load
   the chart's values for the inventory journal, the inventory valuation account and the
   two production work-in-progress accounts onto the company, and the variation account
   and closing expense account onto the accounts the chart names.
4. **Category defaults.** Two defaults are written so that categories created afterwards
   start sensibly: the costing method defaults to `standard` and the valuation mode to
   `periodic`.

**Postconditions.** Every company can post a closing; every product has a dated initial
cost; new categories default to standard price and periodic valuation.
