# Interfaces of the Inventory Valuation and Costing domain

Window actions and menus as the user navigates them, views and what each shows, named
remote operations with their inputs and outputs, client-side routes, reports and
printable documents, notifications, and import and export considerations.

---

## 1. Navigation

### 1.1 Menu entries added by this domain

| Menu path | Action | Who sees it |
|---|---|---|
| Inventory ▸ Operations ▸ Adjustments ▸ **Landed Costs** | the landed cost window action, sequence 115 | Inventory manager (through the access rights on the document) |

### 1.2 Actions reachable without a menu

| Action | Reached from | Purpose |
|---|---|---|
| **Inventory Valuation** (client action) | the accounting or inventory reporting navigation supplied by the surrounding application, and by the client-side path `stock-valuation-closing` | The valuation closing report. |
| **Unit Cost History** (window action) | clicking the unit cost cell of a product in the stock list | The chronological justification of a product's unit cost. |
| **Valuation** (window action) | clicking the total value cell of a product in the stock list | The list of valued movements of that product. |
| **Adjust Valuation** (contextual action on a goods movement) | the contextual actions of the goods movement list | Opens the value adjustment dialog for exactly one movement. |
| **Adjust Valuation** (window action, opened as a dialog) | the action above, and the button on a movement | The valuation history record form. |
| **Landed Costs** (from a vendor bill) | the "Landed Costs" statistics button on a bill that already has documents | Lists or opens the landed cost documents created from that bill. |
| **Create Landed Costs** (button on a vendor bill) | the bill's header | Creates a landed cost document from the bill's flagged lines and opens it. |
| **Post WIP Accounting Entry** (contextual action on manufacturing orders) | the contextual actions of the manufacturing order list, for the accounting user group | Opens the work-in-progress wizard as a dialog. |
| **WIP** (statistics button on a manufacturing order) | the order's button box, for the accounting user group, hidden when the count is zero | Opens the work-in-progress entries of that order. |
| **Manufacturing** (statistics button on a journal entry) | the entry's button box, for the manufacturing user group, hidden when the count is zero | Opens the manufacturing orders a work-in-progress entry was based on. |
| **Compute Price from BoM** (button and contextual action on products) | the unit-cost area of the product form, and the contextual actions of the product list and card views, for the manufacturing manager group | Recomputes the unit cost from the bill of materials. Hidden when the product has no bill of materials, and when the combination is perpetual valuation with first in first out costing. |

### 1.3 Client-side paths

| Path | Opens |
|---|---|
| `stock-valuation-closing` | the Inventory Valuation client action |
| `landed-costs` | the Landed Costs window action |

These are paths of the single-page web client, not server routes. **This domain adds no
server routes and no controllers.**

---

## 2. Views

### 2.1 Product category form

Two blocks are added.

**Inventory Valuation group**, placed after the logistics group:

| Field | Behaviour |
|---|---|
| Costing Method (`property_cost_method`) | required in the form |
| Inventory Valuation (`property_valuation`) | shown only to the accounting read-only group or the inventory manager group |

**Accounting group**, placed after the expense account:

| Field | Behaviour |
|---|---|
| Inventory Valuation Account (`property_stock_valuation_account_id`), labelled **"Stock Account"** | no inline creation |
| Inventory Variation Account (`account_stock_variation_id`), labelled **"Stock Variation"** | no inline creation; writes through to the valuation account's own variation account |
| Price Difference Account (`property_price_difference_account_id`) | no inline creation; **hidden** unless the costing method is `standard` **and** the valuation mode is `real_time` |

### 2.2 Product template form

| Addition | Placement | Behaviour |
|---|---|---|
| Valuation by Lot/Serial (`lot_valuated`) | before the serial-number prefix format label | rendered with a confirmation widget; hidden while the tracking mode is `none` |
| Is a Landed Cost (`landed_cost_ok`) | inside the billing group | hidden unless the product type is service |
| Default Split Method (`split_method_landed_cost`) | inside the billing group | hidden unless the landed-cost flag is on and the type is service |
| Compute Price from BoM button | inside the unit-cost area | manufacturing manager group; hidden when there is no bill of materials, or when the valuation mode is `real_time` and the costing method is `fifo` |

### 2.3 Product template list

The unit cost column is made **read-only** in the generic product list, because the
valuation engine maintains it.

### 2.4 Product stock list

Three columns are inserted before the quantity on hand:

| Column | Behaviour |
|---|---|
| Cost Method (`cost_method`) | optional, hidden by default |
| Unit Cost (`avg_cost`) | optional, shown by default; rendered with a widget that turns the cell into a link to the **Unit Cost History** action, passing the product as the active record and the costing method in the context; the link is disabled for products using first in first out |
| Total Value (`total_value`) | optional, shown by default; rendered with a widget that turns the cell into a link to the **Valuation** action, passing a filter on the product, a filter on incoming movements, a filter on movements with a remaining quantity, and the costing method and tracking mode in the context; the column is summed |

A hidden column carries the valuation currency so that the two monetary columns can be
formatted.

### 2.5 Product list at a date

Two optional columns, hidden by default, are inserted after the unit cost: the average
cost labelled **"Unit Cost"** and the total value labelled **"Total Value"**, the latter
summed.

### 2.6 Goods movement list (generic)

Three optional columns, hidden by default, are inserted after the state: the value, the
remaining quantity and the remaining value. The value is read-only.

### 2.7 Goods movement search

A filter named **"Remaining"** is added after the inventory filter, preceded by a
separator. Its condition is "the remaining quantity is set".

### 2.8 Valuation list (the dedicated list used by the Valuation action)

Ordered by date descending, then identifier descending. Columns:

| Column | Behaviour |
|---|---|
| Reference | fixed width |
| Date | |
| Quantity | |
| Unit of measure | shown only when the unit-of-measure feature is enabled |
| Lots | rendered as tags; hidden when the context says the product is untracked |
| Value | monetary, in the valuation currency; **hidden unless the context says the costing method is `fifo`** |
| Unit Cost (`standard_price`) | monetary; **hidden when the context says the costing method is `fifo`** |
| Remaining Quantity | summed, labelled "Total Remaining Qty" |
| Remaining Value | monetary, summed, labelled "Total Remaining Value" |
| Value Description | optional, hidden by default; **hidden unless the costing method is `fifo`** |

The action's own condition restricts the list to movements that are incoming or outgoing.

### 2.9 Value adjustment dialog

A form on the valuation history record, titled **"Adjust Valuation"**, opened as a
dialog.

| Element | Behaviour |
|---|---|
| Current Value | monetary, shown inline in a narrow field |
| Current Value Details | shown inline beside it: **"For _the quantity_ _the unit_ (_the unit price_ per _the unit_)"** |
| New Value (`value`) | monetary, the amount the user types |
| Description | free text |
| Current Value Description | rendered as an information banner, no label, spanning both columns |
| Computed Value Description | rendered as an information banner; **hidden when empty** |

The movement reference and the currency are carried as hidden fields.

### 2.10 Stock quantity lists

| List | Addition |
|---|---|
| Generic quantity list | a hidden currency column and an optional **Value** column, hidden by default |
| Editable quantity list | a hidden currency column, a hidden cost-method column and an optional **Value** column, hidden by default, summed under the label "Total Value" |
| Inventory counting list | an optional **Accounting Date** column, hidden by default |

### 2.11 Lot form

Inside the inventory group:

| Field | Behaviour |
|---|---|
| Total Value | monetary in the valuation currency; hidden unless the product is valuated by lot |
| Average Cost | monetary; hidden unless the product is valuated by lot |
| Cost (`standard_price`) | monetary; hidden unless the product is valuated by lot |

### 2.12 Location form

A group titled **"Accounting Information"**, placed after the additional information
group, visible only when the location's usage is `inventory` or `production`. It holds
the location's valuation account with the label **"Cost of Production"** for a production
location and **"Loss Account"** for an inventory-loss location.

### 2.13 Account form

Two fields after the tag list, both visible only for accounts of type current asset:

| Field | Behaviour |
|---|---|
| Variation Account (`account_stock_variation_id`) | |
| Expense Account (`account_stock_expense_id`) | placeholder **"For Perpetual Continental Only"**; technical-features group only |

### 2.14 Return wizard line list

An optional column, hidden by default, for the technical-features group: **"Update
quantities on the order"**.

### 2.15 Inventory adjustment naming form

The **Accounting Date** field is added after the adjustment name, hidden unless at least
one selected product uses perpetual valuation.

### 2.16 Inventory settings page

A block titled **"Valuation"**, placed after the lot and serial number block:

| Setting | Behaviour |
|---|---|
| Landed Costs | with a nested area that, once landed costs are installed, shows the **Default Journal** field |
| Display Lots and Serial Numbers on Invoices | hidden unless lot and serial number tracking is enabled |

### 2.17 Vendor bill form

| Addition | Placement | Behaviour |
|---|---|---|
| **Landed Costs** statistics button | the button box | inventory manager group; hidden when the bill has no landed cost document |
| **Create Landed Costs** button | before the state badge | inventory manager group **and** accounting invoicing group; hidden unless at least one line is flagged as a landed cost line and no document exists yet; keyboard shortcut `l` |
| Landed Costs column on the invoice line list | before the quantity | inventory manager group; optional, hidden by default; hidden entirely unless the document type is vendor bill or vendor receipt; read-only unless the product type is service |
| Landed-cost flag on the journal item list | after the label | inventory manager group; always hidden as a column, present so that the flag can be read |

### 2.18 Landed cost form

Header: a **Validate** button (highlighted, hidden unless the state is `draft`), a
**Cancel** button (hidden unless the state is `draft`), and the state as a status bar
showing `draft` and `done`.

Title: the document name.

Left group: the date (read-only once posted); the target selection rendered as radio
buttons (hidden until manufacturing landed costs are installed, then visible to the
inventory manager group); the transfers as tags (no inline creation or editing, hidden
unless the target is transfers, read-only once posted, restricted to transfers of the
same company having at least one incoming or outgoing movement).

With manufacturing landed costs installed, the manufacturing orders appear as tags,
hidden unless the target is manufacturing orders, restricted to orders of the same
company having at least one incoming finished-goods movement. With subcontracting landed
costs installed, the transfer restriction is widened to also accept transfers having a
completed subcontracting movement, and the manufacturing order selector uses the
subcontracting list and filter views.

Right group: the journal (read-only once posted), the company (multi-company group only),
the journal entry (hidden when empty), the vendor bill.

**Additional Costs** page: the cost lines, editable at the bottom while the document is
draft, showing the product (restricted to products flagged as landed costs, defaulting
new ones to that flag and to the service type), the description, the account (no inline
creation), the split method and the amount. Below, a subtotal footer showing the total
and a **Compute** button (hidden unless the state is `draft`).

**Valuation Adjustments** page: the adjustment lines, editable at the bottom, creation
disabled, showing the cost line (read-only), the product (read-only), the weight and the
volume (read-only, optional, hidden by default), the quantity (read-only), the original
value (read-only), the new value (read-only) and the **allocated amount**, which is the
only editable cell.

A message and activity panel closes the form.

### 2.19 Landed cost list, card view and search

**List**: name in bold, date (read-only once posted), company (multi-company group),
state as a badge (green when posted, blue when draft), and an activity-exception marker.
Draft rows are rendered in the informational style; cancelled rows are muted.

**Secondary list** (used when a bill opens several documents): name, date, total amount,
state, company.

**Card view**: the name and the state on one row; the date with a clock icon and the
journal on the next.

**Search**: fields for the name and the transfers; filters **Draft** and **Done**; a date
filter; the four hidden activity filters; groupings by **Status** and by **Date**.

**Empty-state help**: "Create a new landed cost" with the explanation "Landed costs allow
you to include additional costs (shipment, insurance, customs duties, etc) into the cost
of the product."

### 2.20 Unit cost history list

| Column | Behaviour |
|---|---|
| Reference | |
| Product | |
| Date | |
| Description | |
| Added Quantity (`quantity`) | hidden when the context says the costing method is `standard` |
| Added Value (`added_value`) | monetary; hidden when the costing method is `standard` |
| Unit Cost (`value`) | monetary; **shown only** when the costing method is `standard` |
| Total Value | monetary; hidden when the costing method is `standard` |
| Total Quantity | hidden when the costing method is `standard` |
| Unit Cost (`avco_value`) | monetary; hidden when the costing method is `standard` |
| Justification | optional, hidden by default; shown only when the costing method is `average` |

The action restricts rows to the active product.

### 2.21 Manufacturing order form

A hidden field carrying "show valuation" for the inventory manager group, and a **WIP**
statistics button in the button box for the accounting user group, hidden when the
work-in-progress entry count is zero.

### 2.22 Manufacturing order graph

The extra unit cost is added as an available measure.

### 2.23 Work centre form

The expense account and the analytic distribution are added by the manufacturing
accounting integration.

### 2.24 Work in progress wizard form

Titled **"Post Manufacturing WIP"**. Left group: journal, reference. Right group: date,
reversal date. The manufacturing orders are carried hidden. The lines are shown as an
editable list with the account, the label, the debit (summed, labelled "Total Debit") and
the credit (summed, labelled "Total Credit"), plus a hidden currency column. Footer: a
primary **Post WIP** button (keyboard shortcut `q`) and a **Discard** button (keyboard
shortcut `x`).

---

## 3. The Inventory Valuation report

### 3.1 Shape

A full-screen client action with a control panel. The control panel carries a buttons bar
and a date filter.

**Buttons bar**: a single **Generate Entry** button. (Two further buttons, one for a
portable-document-format export and one for a spreadsheet export, exist in the markup but
are disabled and commented out; the two corresponding operations on the report exist and
return nothing.)

**Date filter**: a dropdown holding a date picker, defaulting to today. Choosing a date
reloads the whole report as of that date.

### 3.2 Sections, in order

| Section | Content | Clicking it |
|---|---|---|
| **Initial Balance** | The posted ledger balance of each inventory valuation account as of the date, with one sub-line per account showing its display name and its balance. | Opens the journal items list, filtered to those accounts, grouped by account and by month, and — when the report date is not today — restricted to items dated not after it. |
| **Inventory Loss** (only when at least one location of usage `inventory` carries a valuation account) | The reclassification lines: one per account with its debit and its credit. The section total is the negation of the total debit. | Opens the goods movement list filtered to movements whose source or destination has the usage `inventory`, restricted to the report date when one is set. |
| **Cost of Production** (only when manufacturing accounting is installed **and** at least one location of usage `production` carries a valuation account) | The same computation restricted to production locations: one line per account with its debit and credit; the section total is the negation of the total debit. | — |
| **Stock Variation** | The proposed balancing lines: one per account with its debit and its credit. The section total is the total debit. | — |
| **Ending Stock** | The physical value attributed to each inventory valuation account, with one sub-line per account. | Opens the stock report; when the report date is not today, the date is pushed into its context. |

An **Accrual** section exists in the markup but is not rendered: the sales and purchasing
integrations extend it with "Goods Received Not Invoiced" and "Goods Delivered Not
Invoiced" blocks, but the server side supplies no data for them in the current
behaviour.

### 3.3 Behaviour of Generate Entry

The button calls the closing operation on the report's company. The date is passed **only
when it differs from today**. If the operation returns a window action, the client opens
it — which is the created journal entry in form view, titled "Journal Items". If the
operation raises, the error is shown to the user.

---

## 4. Named remote operations

These are the operations a client or an integration may call. Each is listed with its
receiver, its inputs and its outputs.

### 4.1 On the company

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_close_stock_valuation` | optionally a date (as text or a date), optionally a flag asking for automatic posting | a window action opening the created journal entry, or nothing when the run came from the scheduled job and had nothing to do | Runs the closing. Raises on the five guard failures listed in [business-rules.md](business-rules.md#1-complete-list-of-error-conditions). Receiver must be exactly one company. |
| `stock_value` | optionally a product-to-account map, optionally a date | a map from account to physical value | Sums the total value of the products, as of the date, per inventory valuation account. |
| `stock_accounting_value` | optionally a product-to-account map, optionally a date | a map from account to ledger balance | Sums the balance of the posted journal items on the inventory valuation accounts of the company, as of the date. |
| `_cron_post_stock_valuation` | none | none | The scheduled job body. |

### 4.2 On the inventory valuation report

| Operation | Inputs | Output |
|---|---|---|
| `get_report_values` | optionally a date | a structure holding the report data under the key `data` and an empty context under the key `context` |
| `action_print_as_pdf` | none | nothing |
| `action_print_as_xlsx` | none | nothing |

### 4.3 On a goods movement

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_adjust_valuation` | none; the receiver must hold exactly one movement | a window action opening the value adjustment dialog, targeted as a dialog, with the movement pre-set; when the selection resolves to a single product the action is renamed **"Adjust Valuation: _the product display name_"** | Raises **"You can only adjust valuation for one move at a time."** when the receiver holds more than one movement. |

### 4.4 On a landed cost document

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `button_validate` | none | true | Validates; raises on any guard failure. |
| `button_cancel` | none | the result of the write | Cancels; raises when any document is posted. |
| `compute_landed_cost` | none | true | Deletes and rebuilds the adjustment lines. |
| `get_valuation_lines` | none; exactly one document | the list of valuation line values | Raises when nothing is eligible. |

### 4.5 On a journal entry

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `button_create_landed_costs` | none; exactly one entry | a window action opening the created document in form view | Creates a landed cost document from the flagged lines. |
| `action_view_landed_costs` | none; exactly one entry | a window action listing or opening the documents, titled **"Landed Costs"**, with the bill pre-set as the default vendor bill | — |
| `action_view_wip_production` | none; exactly one entry | a window action opening the single manufacturing order in form view, or the list titled **"WIP MOs of _the entry name_"** | — |

### 4.6 On a manufacturing order

| Operation | Inputs | Output |
|---|---|---|
| `action_view_move_wip` | none; exactly one order | a window action opening the single work-in-progress entry in form view, or the list titled **"WIP Entries of _the order name_"** rendered with the generic journal entry list |

### 4.7 On a product

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `button_bom_cost` | none; exactly one product or template | nothing | Recomputes the unit cost from the bill of materials. |
| `action_bom_cost` | none | nothing | The same, over a set, recomputing child bills of materials that are themselves in the set. |

### 4.8 On the work-in-progress wizard

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `confirm` | none; exactly one wizard | nothing | Creates and posts the entry and its reversal; raises on the two guard failures. |

---

## 5. Reports and printable documents

### 5.1 Lot and serial number table on the invoice

The printed customer invoice gains, after the right-hand element block, a table restricted
to members of the **"Display Serial and Lot Number on Invoices"** group, with columns
**Product**, **Quantity** and **SN/LN**. When there is nothing to show, a placeholder
block is rendered instead so that the layout does not collapse.

**How the rows are built.** The computation runs only for a posted customer invoice or
credit note that has an invoice date.

1. Take the product lines of the document with a non-zero quantity whose product is a
   goods product.
2. Take every invoice line of the same sales order lines that qualifies for lot
   presentation, sorted by date, then document identifier, then line identifier. Find the
   position of this document's first line in that list; everything before it is the
   **previous** set.
3. On a customer invoice, drop from the previous set any line whose document was reversed
   **and** whose reversal also precedes this document.
4. Compute the quantity invoiced per product for this document and for the previous set.
   On a credit note, negate both.
5. Walk the completed movement lines carrying a lot of the sales order lines, sorted by
   date then identifier. Skip lines whose product is not invoiced here and lines that
   should not be shown. Convert each quantity into the product reference unit.
6. Decide whether the line is a **return** in the sense of the document: on an invoice, a
   line coming from a customer location and going to an internal or vendor location; on a
   credit note, a line going to a customer location and coming from an internal or vendor
   location. For a return, subtract from the running per-lot quantity up to what it holds,
   and turn the remainder negative.
7. Apply the previously-delivered bookkeeping: when the quantity is negative, or when the
   previously delivered quantity is still below the previously invoiced quantity, move
   part of the quantity into the previously-delivered tally so that it is not attributed
   to this document.
8. Accumulate the rest onto the lot.
9. Emit one row per lot with a positive quantity, capping it at the quantity still
   invoiced for that product and decrementing that remainder, reading the lot with
   elevated privileges so that a reader without inventory access can still print.

Each row carries the product display name, the quantity formatted at the product-unit
precision, the unit name and the lot name, plus the lot identifier so that a localisation
can add columns.

### 5.2 The inventory valuation report

Described in [section 3](#3-the-inventory-valuation-report). It has no printable form in
the current behaviour.

### 5.3 The unit cost history

Described in [section 2.20](#220-unit-cost-history-list). A read-only list over a
database view; it has no printable form.

### 5.4 Manufacturing order overview

The manufacturing accounting integration extends the order overview report with cost
figures. Its templates are supplied by that integration.

---

## 6. Notifications and messaging

| Event | Channel | Content |
|---|---|---|
| A landed cost document moves to `done` | the document's message thread | A tracking message posted under the subtype named "Done", described **"Landed cost validated"**. Followers of the document are notified. |
| Any other tracked change on a landed cost document (the name, the date, the total, the state moving to `cancel`) | the document's message thread | A tracking message under the generic subtype. |
| A change of the costing method or of the valuation mode on a category | the category's message thread | A tracking message recording the old and the new value. |
| A change of the sales price of a product template | the template's message thread | Tracked by the products domain; mentioned here because the unit cost is **not** tracked — its history lives in the valuation history records instead. |

There are no email templates and no scheduled digests in this domain.

---

## 7. External service integrations

None. This domain calls no external service, exposes no endpoint and defines no
controller.

---

## 8. Import and export

### 8.1 What can be imported

| Record | Notes |
|---|---|
| Product Category | The valuation fields are company dependent: an import writes them for the company in the current scope only. |
| Product Value | Importable. Creating one triggers the side effects described in [entities.md](entities.md#creation-behaviour-side-effects) — a movement correction re-values the movement, a lot cost change re-values the product. Importing a large history therefore performs a large amount of recomputation. |
| Landed Cost and its cost lines | Importable. The adjustment lines are **not** meant to be imported: they are produced by the split computation and are deleted and rebuilt by it. |
| Company and Account settings | Importable. |

### 8.2 What cannot be imported

| Record | Why |
|---|---|
| Average Cost History | It is a database view with no write access. |
| The computed value fields on a product, a lot or a stock quantity record | They are not stored. |
| The value of a goods movement | It is stored, and writing it directly is possible, but the supported path is to write the **manual value** field, which creates a valuation history record and preserves the audit trail. A direct write is overwritten by the next re-valuation. |

### 8.3 Export considerations

- The value of a stock quantity record cannot be summed by the database. A grouped export
  asking for its sum is served by collecting the records of each group and summing the
  computed values in memory. Very large groupings are therefore expensive.
- The total value and average cost of a product depend on three context keys — the as-of
  date, the company and the warehouse. An export that does not set them gets the figures
  for today, for the companies in the current scope, and across all warehouses.

---

## 9. Context keys that change what the interfaces show

| Key | Effect |
|---|---|
| `to_date` | Valuation and quantities are computed as of that instant. A plain date is widened to the last instant of the day. |
| `at_date` | The internal companion of the above, pushed alongside it. |
| `warehouse_id` | Restricts the quantity on hand, and therefore the value, to one warehouse. The valuation itself is still computed company-wide and then scaled by the ratio of the warehouse quantity to the company quantity. |
| `company` / `allowed_company_ids` | Selects which company's settings, costs and accounts are read. |
| `cost_method` | Passed by the product list into the valuation list and the unit cost history, so that those lists can hide the columns that do not apply. |
| `tracking` | Passed by the product list into the valuation list, so that it can hide the lot column for untracked products. |
| `force_period_date` | Forces the date of the valuation journal entry produced by a goods movement, and changes the inventory name given to an adjustment movement. |
| `valuation_date` | Forces the date of the valuation history record created by a unit-cost change. |
| `disable_auto_revaluation` | Suppresses the creation of a valuation history record when a unit cost is written. Used by every engine-initiated write. |
| `std_price_incremental_recompute` | Enables the incremental fast path of the average unit-cost update. |
| `fifo_qty_already_processed` | Subtracts an amount from the first in first out stack size, so that several outgoing movements of one batch consume the stack in sequence. |
| `closing_cron` | Marks a closing run as coming from the scheduled job, which turns the "nothing to close" error into a silent return. |
| `inventory_data` | Lets a caller feed a pre-computed physical value into the closing computation instead of having it recomputed. |
| `inventory_name` | Suppresses the automatic naming of an adjustment movement. |
| `move_reverse_cancel` | Suppresses the injection of cost-of-goods-sold lines and price-difference lines, and preserves them when copying. |
| `conversion_date` | Forces the currency conversion date of a purchase order line's unit price for stock. |
| `skip_kit_qty_available` | Suppresses the kit expansion of the quantity computation, used by the inventory valuation report. |
| `valuation_without_extra` | Suppresses the landed cost source when a subcontracting integration reads a movement's value. |
