# Business rules of the Inventory Valuation and Costing domain

Every validation, constraint, invariant, permission check, locking rule and edge-case
behaviour of the domain, with the exact user-facing message where one exists.

Messages are reproduced exactly as the system produces them; placeholders are written in
words between underscores.

---

## 1. Complete list of error conditions

| # | Condition | Message | Raised when |
|---|---|---|---|
| E1 | A filter on the valuation mode uses an operator other than equality | **"You can only use the '=' operator to search on valuation field."** | A filter or a remote search is built on the computed valuation mode field of a product with any operator other than `=`. |
| E2 | A filter on the valuation mode uses an unknown value | **"Only the value 'periodic' and 'real_time' are accepted to search on valuation field."** | The compared value is neither `periodic` nor `real_time`. |
| E3 | Valuation by lot is switched on while untracked stock exists | **"You cannot enable lot valuation because the following products have on-hand quantities without a lot/serial number:"** followed by a newline and the display names of the offending products | A write sets the valuation-by-lot switch to true on a product template and at least one stock quantity record of its variants sits inside the valued perimeter, carries no lot or serial number and holds a non-zero quantity. |
| E4 | A dated unit cost is asked for a product that does not use standard price | **"You can only get the standard price at a given date for products with 'Standard Price' as cost method."** | The dated-unit-cost lookup is called with a date other than today for a product whose costing method is not `standard`. |
| E5 | A closing is requested for a date earlier than the last closing | **"It exists closing entries after the selected date. Cancel them before generate an entry prior to them"** | A report date is supplied and a posted closing entry exists whose date is later. |
| E6 | There is nothing to close | **"Everything is correctly closed"** | The three parts of the closing produce no journal item at all **and** the request did not come from the scheduled job. |
| E7 | The company has no inventory journal | **"Please set the Journal for Inventory Valuation in the settings."** | A closing would produce items but the company's inventory journal is empty. |
| E8 | The company has no inventory valuation account | **"Please set the Valuation Account for Inventory Valuation in the settings."** | A closing would produce items but the company's inventory valuation account is empty. |
| E9 | A filter on the remaining quantity uses anything other than "is set" | **"Only is set (= True) is supported in search for remaining_qty."** | A filter on the computed remaining quantity of a goods movement uses an operator other than equality, a non-boolean value, or the value false. |
| E10 | The value-adjustment action is invoked on more than one movement | **"You can only adjust valuation for one move at a time."** | The contextual "Adjust Valuation" action is run on a selection whose size is not exactly one. |
| E11 | An incoming movement of a lot-valuated product has a line with no lot | **"A lot/serial number is required for product '_the product display name_' as it has lot valuation enabled."** | A movement being valued as incoming belongs to a product with valuation by lot and at least one of its lines carries no lot or serial number. |
| E12 | A completed transfer's completion date is moved into a locked period | **"You cannot modify the scheduled date of operation _the transfer display name_ because it falls within a locked fiscal period."** | The completion date of a transfer is written and that date violates the fiscal-year lock or the hard lock of the transfer's company, **and** the lock-check bypass parameter is not set. |
| E13 | A landed-cost service product is turned into something else while it is in use | **"You cannot change the product type or disable landed cost option because the product is used in an account move line."** | A write changes the type of a service product flagged as a landed cost to a non-service type, or clears the flag, while at least one journal item of that product carries the landed-cost-line flag. |
| E14 | A posted landed cost is cancelled or deleted | **"Validated landed costs cannot be cancelled, but you could create negative landed costs to reverse them"** | The cancel operation is run on a selection containing a document in the `done` state; deletion runs the cancel operation first, so it fails the same way. |
| E15 | The split does not reconcile with the costs | **"Cost and adjustments lines do not match. You should maybe recompute the landed costs."** | At validation, the sum of the allocated amounts differs from the document total, or the sum allocated from one cost line differs from that cost line's amount. |
| E16 | The chosen targets contain nothing that can carry a landed cost | **"You cannot apply landed costs on the chosen _the target label_(s). Landed costs can only be applied for products with FIFO or average costing method."** | The split computation finds no usable goods movement among the targets. The target label is "Transfers" or "Manufacturing Orders". |
| E17 | A non-draft document is validated | **"Only draft landed costs can be validated"** | Any document in the selection is not in the `draft` state. |
| E18 | A document has no targets | **"Please define _the target label_ on which those additional costs should apply."** | A document in the selection has no targeted goods movement. |
| E19 | A cost line has no counterpart account | **"Please configure Stock Expense Account for product: _the cost product name_."** | The journal entry is being built and neither the cost line's account nor the cost product's expense account can be found. |
| E20 | The work-in-progress entry is unbalanced | **"Please make sure the total credit amount equals the total debit amount."** | The wizard is confirmed and the sums of the debits and the credits of its lines differ in the company currency. |
| E21 | The work-in-progress reversal is not after the posting | **"Reversal date must be after the posting date."** | The wizard is confirmed with a posting date set and a reversal date that is not strictly later. |
| E22 | A work-in-progress line carries both a debit and a credit | **"A single line cannot be both credit and debit."** | The database constraint named "check debit credit" on the wizard line is violated. |
| E23 | A search on the location perimeter flag uses an unsupported operator | **"Invalid search operator or value"** | A filter on the "inside the valued perimeter" flag uses an operator other than equality or inequality. |

---

## 2. Structural constraints

### 2.1 Database constraints

| Record | Constraint | Rule | Message |
|---|---|---|---|
| Work In Progress Accounting Line | `check_debit_credit` | `debit = 0 OR credit = 0` | **"A single line cannot be both credit and debit."** |

### 2.2 Required fields

| Record | Required fields |
|---|---|
| Product Value | value, company, date, user |
| Landed Cost | date, target, journal, company |
| Landed Cost Line | landed cost, product, amount, split method |
| Valuation Adjustment Line | landed cost, product, quantity |
| Company | inventory period, costing method |
| Work In Progress Accounting Wizard | reversal date, journal |
| Average Cost History row | date, user, company, product, reference, description, resource model name, quantity, value |

### 2.3 Delete-restriction rules

| Reference | On delete |
|---|---|
| Product Category → Inventory Valuation Account | restrict: the account cannot be deleted while any category references it. |
| Product Category → Price Difference Account | restrict. |
| Product Template → Price Difference Account | restrict. |
| Product Category → Production Account | restrict. |
| Landed Cost Line → Landed Cost | cascade: deleting the document deletes its cost lines. |
| Valuation Adjustment Line → Landed Cost | cascade. |
| Goods Movement → Purchase Order Line | the reference is cleared. |
| Operation type code `dropship` removed | affected records become archived outgoing types. |
| Landed cost target `manufacturing` removed | affected records fall back to the default target, `picking`. |
| Analytic applicability business domain `manufacturing_order` removed | cascade. |

### 2.4 Company-check rules

Every account and journal reference listed below must belong to the same company as the
record holding it, or to no company:

- Product Category → inventory valuation account, price difference account, production
  account.
- Product Template → price difference account.
- Company → inventory journal, inventory valuation account, production work-in-progress
  account, production work-in-progress overhead account.
- Work Center → expense account.

### 2.5 Domain restrictions on account selection

| Field | Allowed accounts |
|---|---|
| Location → inventory valuation account | any account whose type is **not** receivable, payable, cash or credit card. |
| Category → income and expense accounts | any account whose type is not receivable, payable, cash, credit card or off-balance. |

---

## 3. Invariants

These must hold after every operation. A rebuild that breaks any of them is not
behaviourally equivalent.

| # | Invariant |
|---|---|
| I1 | A movement whose state is not `done` has all three classification flags false. |
| I2 | A movement that is not incoming has a remaining quantity of zero and a remaining value of zero. |
| I3 | A movement whose lines all carry an owner other than the company's partner is neither incoming nor outgoing, whatever its locations. |
| I4 | A movement both of whose locations are inside the valued perimeter, or both outside it, is neither incoming nor outgoing — unless it is a drop shipment or a returned drop shipment, which are both classified separately. |
| I5 | The sum of the remaining quantities of the incoming movements of a product, inside one company and one lot scope, equals the quantity on hand of that product in the valued perimeter, whenever that quantity is positive. |
| I6 | Under first in first out, the total value of a product equals the first in first out valuation of its quantity on hand, which equals the sum of the remaining values of its incoming movements up to currency rounding. |
| I7 | Under average cost with no manual correction in force, the total value of a product equals its quantity on hand multiplied by its unit cost, up to currency rounding. |
| I8 | Under standard price, the total value of a product equals its quantity on hand multiplied by its unit cost, exactly. |
| I9 | A landed cost document in the `done` state has, for each of its cost lines, allocated amounts summing to that cost line's amount within currency rounding, and allocated amounts summing overall to the document total within currency rounding. |
| I10 | A landed cost document in the `done` state either has a journal entry or produced no qualifying adjustment line. |
| I11 | After a closing entry is posted, the posted ledger balance of every inventory valuation account of the company equals the physical value of the goods attributed to it as of the closing date — unless the account has neither a variation account nor a company default expense account, in which case it was skipped. |
| I12 | The injected cost-of-goods-sold items and price-difference items always come in balanced pairs on the same document, so posting them never unbalances the document. |
| I13 | For one sales order line, the sum of the costs recognised across every customer invoice and credit note equals the value that actually left stock for that line — because each invoice recognises the cumulative cost minus what was already recognised. |
| I14 | A valuation history record is never created by an engine-initiated write; only a human's write of a unit cost, a human's value adjustment, the product creation hook and the installation hook create them. |
| I15 | A movement's value, once written, is only changed by: a re-valuation triggered by one of the events listed in [calculations.md](calculations.md#24-which-correction-the-caller-uses), or an outgoing correction. Nothing else writes it. |
| I16 | A journal entry produced by a goods movement is always balanced, because it is built as pairs of equal debits and credits. |
| I17 | A closing entry is always balanced, because every part produces balanced pairs. |
| I18 | The company's closing register never holds more than ten identifiers. |

---

## 4. Rules governing whether a movement is valued

### 4.1 The valued perimeter

A location is **inside** the valued perimeter when **both**:

1. it belongs to a company; and
2. its usage is `internal` or `transit`.

Consequences:

| Location | Inside? | Note |
|---|---|---|
| A warehouse stock location | yes | |
| An input, quality-control, packing or output zone | yes | usage `internal` |
| A transit location of the company | yes | usage `transit` with a company |
| An inter-company transit location | **no** | usage `transit` but no company |
| A vendor location | no | |
| A customer location | no | |
| A production location | no | |
| An inventory-loss location | no | |
| A scrap location | no | usage `inventory` |
| A virtual location with no company | no | |
| An archived internal location of the company | **yes** | the perimeter test does not look at the active flag, and the perimeter search runs with archived records included |

### 4.2 Exclusion by ownership

A movement line whose owner is set and is **not the company's own partner** is excluded
from valuation entirely: it counts neither as incoming nor as outgoing, and its quantity
never enters a valued quantity. The same rule applies to stock quantity records: such a
record's monetary value is zero.

The one place where such lines are still counted is the **cost-of-goods-sold unit price**,
which adds a signed *valued consigned quantity* to its denominator so that a consignment
flow does not distort the unit cost (see
[calculations.md](calculations.md#91-cost-of-goods-sold-unit-price-of-a-set-of-movements)).

### 4.3 Exclusion by picking

A movement line that is not marked as picked never counts as incoming or outgoing.

### 4.4 Drop shipments

A movement from a vendor location (or a company-less transit location) to a customer
location (or a company-less transit location) is a **drop shipment**. It is valued — the
value priority chain runs on it just as it does for an incoming movement — but:

- it is **not** classified as incoming (the incoming test excludes returned drop
  shipments; the drop-shipment classification is carried on its own flag);
- it never produces a valuation journal entry, because the valuation-entry test requires
  the incoming or outgoing flag;
- any invoice or bill line whose reachable movements include a drop shipment is **not
  eligible for stock accounting**, so no cost-of-goods-sold line and no price-difference
  line is injected for it, and the bill line is not redirected to the inventory asset.

The reverse direction — customer to vendor — is a **returned drop shipment** and is
excluded from the incoming classification by name.

---

## 5. Rules governing whether a journal entry is produced

A goods movement produces a valuation entry only when **all five** hold:

1. the product is storable;
2. the movement is valued (incoming or outgoing);
3. at least one of the two locations carries a location valuation account;
4. the movement quantity is not zero at the precision of its unit of measure;
5. the product's valuation mode is `real_time`.

A landed cost adjustment line produces journal items only when **all three** hold:

1. it names a goods movement;
2. the movement's product uses `real_time` valuation;
3. the movement's remaining quantity is not zero.

A customer invoice line produces cost-of-goods-sold items only when **all five** hold:

1. the document is a sales document;
2. the line is eligible for stock accounting (storable product, no drop shipment among
   its movements);
3. the product's valuation mode is `real_time`;
4. both an inventory valuation account and a counterpart (the product's expense account
   or the journal's default account) can be selected;
5. the computed amount is not zero for the document currency and the computed unit price
   is not zero at the Product Price precision.

A vendor bill line produces price-difference items only when **all six** hold:

1. the document type is vendor bill, vendor credit note or vendor receipt;
2. the company uses anglo-saxon accounting;
3. the line is eligible for stock accounting;
4. the product's costing method is `standard`;
5. a price difference account can be selected;
6. the subtotal difference is not zero for the document currency **and** the line's
   stored unit price agrees with its computed unit price at the Product Price precision.

---

## 6. Permission checks

### 6.1 Who may read values

| Field | Visibility rule |
|---|---|
| Stock quantity → monetary value and currency | Restricted to the inventory manager group. |
| Product → total value, average cost, valuation currency | Computed with elevated privileges, so the computation itself never fails on access; the fields themselves follow the ordinary product access rules. |
| Lot → total value, average cost, valuation currency | Computed with elevated privileges. |
| Lot → cost | Readable by any internal user. |
| Product and lot unit cost | Readable by any internal user. |
| Category → valuation mode | Shown on the form only to the accounting read-only group or the inventory manager group. |
| Account → closing expense account | Shown on the form only to the technical-features group. |
| Landed cost → manufacturing orders | Restricted to the inventory manager group. |

### 6.2 Elevated-privilege operations

The following run with elevated privileges so that a user who lacks accounting access can
still complete an inventory operation:

- Creating valuation history records from a unit-cost change or from a lot cost change.
- Reading the valuation history when evaluating the value priority chain.
- Writing the unit cost during a recomputation.
- Creating and posting the valuation journal entry of a goods movement.
- Creating analytic lines that mirror a movement.
- Creating and posting the work-in-progress entry and its reversal.
- Reading lots when rendering the lot table on an invoice.

### 6.3 Access rights granted by this domain

| Record | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Account | Inventory manager | yes | no | no | no |
| Journal | Inventory manager | yes | no | no | no |
| Product Value | Inventory manager | yes | yes | yes | yes |
| Transfer | Accounting read-only | yes | no | no | no |
| Transfer | Accounting invoicing | yes | yes | yes | no |
| Goods Movement | Accounting read-only | yes | no | no | no |
| Goods Movement | Accounting invoicing | yes | yes | yes | no |
| Average Cost History | Accounting read-only | yes | no | no | no |
| Average Cost History | Inventory manager | yes | no | no | no |
| Landed Cost | Inventory manager | yes | yes | yes | yes |
| Landed Cost Line | Inventory manager | yes | yes | yes | yes |
| Valuation Adjustment Line | Inventory manager | yes | yes | yes | yes |
| Work In Progress Wizard | Accounting manager | yes | yes | yes | no |
| Work In Progress Wizard Line | Accounting manager | yes | yes | yes | yes |
| Bill of Materials | Accounting read-only | yes | no | no | no |
| Bill of Materials | Accounting invoicing | yes | no | no | no |
| Bill of Materials Line | Accounting read-only | yes | no | no | no |
| Bill of Materials Line | Accounting invoicing | yes | no | no | no |

Nobody is granted write, create or delete on the average cost history: it is a database
view.

### 6.4 Record rules

| Record | Rule | Condition |
|---|---|---|
| Product Value | multi-company | the record's company is among the companies in the current scope |
| Average Cost History | multi-company | the row's company is among the companies in the current scope |
| Landed Cost | multi-company | the document's company is among the companies in the current scope |

---

## 7. Locking rules

### 7.1 Fiscal lock on transfer dates

**Rule.** A transfer's completion date may not be written such that it falls inside a
locked fiscal period.

**Which locks apply.** Only the **fiscal-year lock** and the **hard lock** of the
transfer's company. The sales lock, the purchase lock and the tax lock are explicitly
excluded, because a goods movement is none of those.

**How it is checked.** On every write of the completion date, and on the computation of
whether the date is editable.

**Bypass.** When the system parameter `stock_account.skip_lock_date_check` (the parameter
that suppresses the lock check on transfers) holds a truthy value, the constraint returns
immediately and no check is made.

**Editability.** A completed or cancelled transfer's date is marked non-editable when its
completion date falls inside a locked period, so the interface prevents the attempt
before the constraint fires.

**Message on violation.** **"You cannot modify the scheduled date of operation _the
transfer display name_ because it falls within a locked fiscal period."**

**Note.** The planned date of a transfer that is not yet completed is **not** subject to
this rule; only the completion date is.

### 7.2 The closing anchor as a soft lock

A posted closing entry acts as a lower bound: the next closing only considers movements
dated after the anchor instant, and refuses outright to be run for a date earlier than
the last closing date. It is not a lock in the accounting sense — it does not prevent a
goods movement from being recorded in the closed period — but it prevents the same period
being closed twice.

A closing entry left in **draft** is not an anchor. This is a real operational hazard:
the next closing will recompute the same period from the previous anchor, and posting
both entries double-counts the variation. The rule for an operator is to post or delete a
draft closing before running another.

---

## 8. Edge-case behaviours

### 8.1 A movement created already completed

A movement created directly in the completed state — for instance when a line is added to
an already-completed transfer — is valued by the **creation** hook rather than by the
completion hook: the outgoing ones are valued, then the valuation entries are created for
all of them. The incoming ones are **not** valued at creation; they are valued when their
lines are created, because creating a line triggers a full revaluation of an incoming
parent.

### 8.2 A movement that is both incoming and outgoing

A movement whose lines cross the perimeter in both directions carries both flags. Every
algorithm treats the incoming part first:

- the value is set by the incoming branch, which overwrites nothing else, and the
  outgoing branch is then skipped for that movement (the routine's incoming branch ends
  with a continuation to the next movement);
- the average replay applies the incoming branch, then the outgoing branch, using the
  unit cost updated by the incoming branch;
- the valued quantity of the movement is the **incoming** quantity, because the
  incoming test is checked first.

### 8.3 A zero-quantity movement

A movement whose quantity is zero at the precision of its unit of measure never produces
a journal entry. Its value may still be set (to zero) and its flags may still be true if
it has picked lines with zero quantities.

### 8.4 A movement whose product has no cost and no source of value

Every source of the priority chain declines, so the product-cost source applies the
product's unit cost, which is zero. The movement is valued at zero, and the goods enter
stock at no value. The ledger and the physical value stay consistent, but the goods carry
no cost. This is the situation an operator must avoid by setting a unit cost before the
first movement of a product.

### 8.5 A manual correction on a movement that also has landed costs

The manual correction claims the whole quantity **and suppresses the extra source**, so
the landed costs are **not** added on top. The value is exactly what the human typed.
The landed cost document remains posted and its journal entry remains, so the ledger and
the physical value diverge by the landed cost amount until the next closing absorbs it.

### 8.6 Several manual corrections on the same movement

Only the latest by date, ties broken by the highest identifier, is used. Earlier ones
remain as history. There is no accumulation.

### 8.7 A manual correction dated in the past

The correction applies from its date. A valuation as of an instant before that date does
not see it; a valuation as of an instant at or after it does. Because the lookup uses
"date not after the as-of instant", a correction dated exactly at the as-of instant is
seen.

### 8.8 A product with no valuation history at all

The average replay cannot anchor, so it replays from the beginning of time. This is slow
but correct. The batch optimisation that narrows the movement filter to "not before the
oldest anchor" is only applied when **every** product of the batch has an anchor, so one
anchorless product in a batch disables the optimisation for the whole batch rather than
producing a wrong result for it.

### 8.9 Deleting valuation history records

Nothing prevents it. The consequences are: past valuations under standard price fall back
to the product's current cost; the average replay loses its anchor and replays from the
beginning; and the unit cost history report loses the corresponding "Adjustment" rows.

### 8.10 A product moved to a different category

The costing method may change as a result. The template write handler detects this before
the write, by comparing each variant's current costing method against the costing method
of the new category (or the company's fallback when no category is given), and schedules
a recomputation for the variants whose method changes.

### 8.11 Kits

A kit product is excluded from the valuation product domain and its computed total value
and average cost are forced to zero, so that the value of its components is not counted
twice. The inventory valuation report additionally suppresses the kit expansion of the
quantity computation, both for performance and because an accounting user may not read
bills of materials.

### 8.12 Branch companies

The scope-company rule on the computed costing method and valuation mode is: when the
product belongs to a company and the company in the current scope is **not** that company
or one of its descendants, the product's own company is used; otherwise the company in
the current scope is used. This lets a branch company override the valuation account of a
category inherited from its parent, and the bill line of a branch company then picks up
the branch's account.

### 8.13 A negative landed cost amount

Accepted everywhere. The split computation handles it identically (the rounding residue
is computed the same way), the journal entry swaps its two sides, and the value priority
chain adds a negative amount.

### 8.14 A landed cost whose movement went negative

The remaining quantity of the movement is negative, so the posted amount is negative and
the journal entry swaps its sides. The comment on the routine notes this case explicitly:
the remaining quantity is negative when the movement is outgoing and delivered products
that were not in stock.

### 8.15 Rounding residue on a landed cost with several cost lines

The residue of each cost line is attached to the adjustment line with the **highest
identifier among all adjustment lines accumulated so far in the whole computation**, not
to the last line of that cost line. With several cost lines, the residues therefore all
pile onto the same adjustment line. The sum check still passes overall, but the per-cost-
line check can fail in that situation, which is why the recomputation is offered
explicitly through the "Compute" button.

### 8.16 A transfer spanning several companies

A closing runs for one company at a time. The daily job iterates over companies and
skips, silently, any company whose closing raises a user-facing error, so one
misconfigured company does not block the rest.

### 8.17 A goods movement in a company other than the one in scope

The valuation routine groups movements by company and re-scopes the records to each
company before valuing them, so a batch spanning companies is valued correctly. The first
in first out stack search, however, spans **every company in the current scope**, while
the average replay is restricted to the single company in scope.

### 8.18 The value of a movement is not cleared when the movement is un-completed

Reverting a movement out of the completed state recomputes the three flags to false but
leaves the stored value in place. Nothing reads it while the flags are false, and
re-completing the movement re-values it, but a reader querying the raw field on a
non-completed movement may find a stale number.

### 8.19 A cost-of-goods-sold line on an invoice whose sales order line was partly credited

The already-recognised quantity and the already-recognised value are both computed over
**every** customer invoice and credit note of the sales orders concerned, with credit
notes contributing negative quantities and the value sign flipped by the negation. A
partial credit therefore reduces both, and the next invoice recognises the difference.

### 8.20 An invoice line with no label

The injected items take the first 64 characters of the label, or the empty string when
there is none. A missing label does not prevent posting.

### 8.21 A cost-of-goods-sold reversal for a standard-price or average-cost product

The shortcut returns the unit price of the **first** matching item found on the original
document. If the original invoice carried several lines of the same product and unit of
measure, the first one's unit price is used for all of them.

### 8.22 Changing the reference of a manufacturing order

Writing a new name on an order rewrites the reference of the analytic lines of its
component movements, and the reference and the label of the analytic lines of its work
orders, so that analytic reporting keeps pointing at the right document.

### 8.23 A work order cancelled or deleted

Both of its analytic line collections are deleted.

### 8.24 The landed-cost flag on a bill line whose product is not a service

The onchange forces the flag back to false. A line can therefore never be marked as a
landed cost line unless its product is a service.

### 8.25 A bill from which landed costs were already created

The "Create Landed Costs" button disappears, because the visibility computation returns
false as soon as the bill has at least one landed cost document. A second document must
be created by hand and linked through its vendor bill field.

---

## 9. Ordering rules that affect results

| Ordering | Where | Why it matters |
|---|---|---|
| Valuation history: date descending, identifier descending | the latest record per product, per lot and per movement | Two records at the same instant are resolved by the higher identifier, so the most recently created wins. |
| Incoming movements: date descending, identifier descending | building the first in first out stack | The stack is built newest-first and then reversed, so two receipts at the same instant are consumed in identifier order, oldest identifier first. |
| Movements in the average replay: product, date, identifier | the replay | Two movements at the same instant are replayed in identifier order. |
| Movements of a purchase order line: date, then identifier | deciding which earlier movements already absorbed the bill | A movement with the same date and a **higher** identifier than the one being valued is considered *later* and does not absorb. |
| Landed cost documents: date descending, identifier descending | the document list | Most recent first. |
| Average cost history rows: date descending, identifier descending | the report | Most recent first, but the running replay sorts ascending internally. |
| Adjustment lines: by identifier | attaching the rounding residue | The residue goes to the highest identifier. |
| Companies during installation: by their position in the company tree | the installation hooks | Parents are configured before their branches. |

---

## 10. Concurrency and batching rules

| Rule | Detail |
|---|---|
| Outgoing before incoming | Within one completion, every outgoing movement is valued **before** the state change and every incoming movement **after** it. This is the only ordering that yields the correct first in first out consumption and the correct average. |
| Several outgoing movements of the same product in one batch | A per-product "quantity already processed" counter is threaded through the first in first out stack computation, so the second movement of the batch does not re-consume the same units. |
| Incremental fast path gating | The fast path for the average unit cost is enabled only when **no** outgoing movement was valued in the same completion. Otherwise the full replay runs, because the fast path assumes the quantity on hand moved only by the incoming amount. |
| Batched valuation memory bounds | The average replay fetches movements in pages of fifty thousand identifiers, fetching only the product and the date on the first pass and the full field set on the second, invalidating the cache between pages. A conforming implementation need not reproduce the page size but must produce the same results for arbitrarily large histories. |
| First in first out stack paging | Incoming movements are fetched a hundred at a time, newest first, and more pages are fetched while the stack size is still positive. |
| Valuation history cache | The set of movements carrying a manual correction is cached per (as-of date, product) for the duration of one database transaction. |

---

## 11. Rules about what must **not** happen

| Rule | Consequence of breaking it |
|---|---|
| An engine-initiated unit-cost write must never create a valuation history record. | The average replay would anchor on engine writes and stop replaying the movements, freezing the cost. |
| A manual correction must suppress the landed cost source. | Landed costs would be counted twice on a corrected movement. |
| An outgoing movement's value must never be recomputed from the costing method after the fact. | The first in first out stack would be consumed twice, or the average of a later day would be applied to an earlier delivery. |
| The valuation entry of a goods movement must never be amended after posting. | The ledger would silently disagree with the posted history; the correction path is the closing entry. |
| A lot-valuated product must never accept an incoming movement with an unlotted line. | The lot costs would not add up to the product's value. |
| The injected invoice items must never be re-derived from the currency or from the product. | Their carefully computed amounts would be overwritten by the generic machinery. |
| The injected invoice items must be dropped when the document is copied. | A duplicated invoice would post a second, spurious cost recognition. |
| A consigned line must never enter a valued quantity. | Goods the company does not own would be capitalised. |
