# Configuration of the Inventory Valuation and Costing domain

Settings, system parameters, sequences and numbering formats, default records, security
groups, access rights, record rules and scheduled jobs.

---

## 1. Company-level settings

These live on the company record and act as the fallback for every product whose
category leaves the corresponding field empty.

| Setting (storage name) | Type | Default | Meaning |
|---|---|---|---|
| Cost Method (`cost_method`) | Selection, required | `standard` | The costing method used when the product's category carries none for this company. Values: `standard` — "Standard Price"; `fifo` — "First In First Out (FIFO)"; `average` — "Average Cost (AVCO)". |
| Valuation (`inventory_valuation`) | Selection | `periodic` | The valuation mode used when the product's category carries none for this company. Values: `periodic` — "Periodic (at closing)"; `real_time` — "Perpetual (at invoicing)". |
| Inventory Period (`inventory_period`) | Selection, required | `manual` | Whether and how often the scheduled job posts a closing entry. Values: `manual` — "Manual"; `daily` — "Daily"; `monthly` — "Monthly". |
| Inventory Journal (`account_stock_journal_id`) | Journal reference | set by the chart of accounts | The journal used by the valuation entries of goods movements and by the closing entry. Must belong to the company. |
| Inventory Valuation Account (`account_stock_valuation_id`) | Account reference | set by the chart of accounts | The asset account used when the product's category carries none. Must belong to the company. |
| Production Work In Progress Account (`account_production_wip_account_id`) | Account reference | set by the chart of accounts where the template provides one | The debit side of the work-in-progress entry. |
| Production Work In Progress Overhead Account (`account_production_wip_overhead_account_id`) | Account reference | set by the chart of accounts where the template provides one | The credit side of the overhead part of the work-in-progress entry. |
| Use anglo-saxon accounting (`anglo_saxon_accounting`) | Boolean | set by the chart of accounts; the generic chart sets it to true | Decides whether the cost of goods sold is recognised at the customer invoice (true) or the purchase is expensed at the bill and the variation posted at the closing (false). |
| Default Landed Cost Journal (`lc_journal_id`) | Journal reference | empty | The default journal of a new landed cost document. Only present when landed costs are installed. |

### 1.1 Settings exposed in the inventory settings page

| Setting (storage name) | Type | Effect |
|---|---|---|
| Landed Costs (`module_stock_landed_costs`) | Boolean | Installs or uninstalls the landed cost capability. Help text: affect landed costs on reception operations and split them among products to update their cost price. Documentation anchor: the landed costs article. |
| Default Journal (`lc_journal_id`) | Journal reference | Shown only while the landed cost capability is enabled; writes the company's landed cost journal. |
| Display Lots and Serial Numbers on Invoices (`group_lot_on_invoice`) | Boolean | Adds every user to the "Display Serial and Lot Number on Invoices" group. Shown only while lot and serial number tracking is enabled. Help text: lots and serial numbers will appear on the invoice. |

---

## 2. Category-level settings

All of these are **company dependent**: the same category record carries a different
value per company, and an empty value means "use the company fallback".

| Setting (storage name) | Type | Default | Meaning |
|---|---|---|---|
| Costing Method (`property_cost_method`) | Selection | the current company's fallback costing method | Overrides the company's costing method for the products of this category. Tracked. Copied when the category is duplicated. |
| Inventory Valuation (`property_valuation`) | Selection | empty (a company-level default of `periodic` is written at installation) | Overrides the company's valuation mode. Tracked. Copied when the category is duplicated. |
| Inventory Journal (`property_stock_journal`) | Journal reference | empty | Overrides the company's inventory journal for the manufacturing labour entry, the landed cost default journal and the work-in-progress wizard default. |
| Inventory Valuation Account (`property_stock_valuation_account_id`) | Account reference | empty | Overrides the company's inventory valuation account. |
| Price Difference Account (`property_price_difference_account_id`) | Account reference | empty | Used only under the combination standard price + perpetual valuation + anglo-saxon accounting. |
| Production Account (`property_stock_account_production_cost_id`) | Account reference | empty | Present only when manufacturing accounting is installed. The valuation counterpart for components and finished goods of a manufacturing order. |

### 2.1 Company-dependent defaults written at installation

Two defaults are written globally at installation, so that every category created
afterwards starts with them:

| Field | Default written |
|---|---|
| `property_cost_method` | `standard` |
| `property_valuation` | `periodic` |

### 2.2 Company-dependent defaults written when a company's category defaults are set up

Four defaults are written **for that company**:

| Field | Value |
|---|---|
| `property_valuation` | the company's valuation mode |
| `property_cost_method` | the company's fallback costing method |
| `property_stock_journal` | the company's inventory journal |
| `property_stock_valuation_account_id` | the company's inventory valuation account |

---

## 3. Account-level settings

| Setting (storage name) | Type | Shown when | Meaning |
|---|---|---|---|
| Variation Account (`account_stock_variation_id`) | Account reference | the account's type is current asset | The counterpart used by parts two and three of the closing entry. Help text: at closing, register the inventory variation of the period into a specific account. |
| Expense Account (`account_stock_expense_id`) | Account reference | the account's type is current asset **and** the reader is in the technical-features group | The counterpart of part three of the closing (the continental perpetual period variation). Placeholder text: "For Perpetual Continental Only". Help text: counterpart used at closing for accounting adjustments to inventory valuation. |

Both are set from the chart of accounts data at installation, for the accounts the chart
template names.

---

## 4. Location-level settings

| Setting (storage name) | Type | Shown when | Meaning |
|---|---|---|---|
| Inventory Valuation Account (`valuation_account_id`) | Account reference, restricted to accounts whose type is not receivable, payable, cash or credit card | the location's usage is `inventory` (labelled "Loss Account") or `production` (labelled "Cost of Production") | The counterpart account used by the valuation entry when goods cross this location under perpetual valuation. Help text: expense account used to re-qualify products removed from stock and sent to this location. |

A location with no valuation account causes no entry to be produced when goods cross it.

---

## 5. Product-level settings

| Setting (storage name) | Where | Meaning |
|---|---|---|
| Valuation by Lot/Serial (`lot_valuated`) | product template, next to the serial-number prefix format; hidden while tracking is `none`; rendered with a confirmation widget | Turns on per-lot valuation. |
| Cost (`standard_price`) | product template and variant; shown read-only in the product list of this domain | The unit cost. |
| Is a Landed Cost (`landed_cost_ok`) | product template, in the billing group; shown only for service products | Marks the product as a cost to spread. |
| Default Split Method (`split_method_landed_cost`) | product template, in the billing group; shown only when the landed-cost flag is on and the type is service | The default split method when the product is used as a landed cost line. |

---

## 6. Work centre settings

| Setting (storage name) | Meaning |
|---|---|
| Expense Account (`expense_account_id`) | The account credited by the manufacturing labour entry for this work centre. Help text: the expense is accounted for when the manufacturing order is marked as done; if not set, the expense account of the final product is used instead. Must belong to the same company. |
| Analytic Distribution | The distribution used to create analytic lines for the work centre's cost. |

---

## 7. System parameters

| Parameter key | Type | Default | Meaning |
|---|---|---|---|
| `stock_account.skip_lock_date_check` | text, read as truthy or falsy | unset | When truthy, the constraint that refuses a completed transfer's date inside a locked fiscal period is disabled entirely. |
| `<company identifier>.stock_valuation_closing_ids` | comma-separated list of journal entry identifiers | unset | The company's **closing register**. A closing appends its entry identifier; the list is capped at ten (appending an eleventh drops the oldest). The last closing anchor is found by walking the list backwards for the first entry that still exists and is posted. |

Both are ordinary configuration parameters read with elevated privileges.

---

## 8. Sequences and numbering

| Sequence | Code | Prefix | Padding | Company | Produces |
|---|---|---|---|---|---|
| Stock Landed Costs | `stock.landed.cost` | `LC/<the four-digit year>/` | 4 | none (shared across companies) | `LC/2026/0001`, `LC/2026/0002`, … |

The landed cost document takes its number at creation, replacing the placeholder text
**"New"**. A document created with an explicit name keeps it.

No other record of this domain is numbered. Valuation entries take their number from the
journal they are posted in; the closing entry takes its number from the inventory
journal.

---

## 9. Default records created at installation

### 9.1 Journal

| Record | Values |
|---|---|
| Inventory Valuation journal | name **"Inventory Valuation"**, code `STJ`, type general, sequence 10, hidden from the accounting dashboard |

The installation hook first looks for an existing general journal with the code `STJ` in
the company; if one exists it is adopted and given the external identifier
`account.<the company identifier>_inventory_valuation`. Otherwise the journal above is
created from the chart of accounts template.

The chart of accounts template also declares, in its own data, that the key
`stock_journal` maps to the `inventory_valuation` journal, so that a freshly installed
chart wires the company's inventory journal automatically.

### 9.2 Initial cost history

For every company and every consumable-type product visible to that company (a product
with no owning company, or owned by that company), one valuation history record is
created with:

| Field | Value |
|---|---|
| Product | the product |
| Value | the product's unit cost in that company |
| Date | today |
| Company | that company |
| Description | **"Initial cost"** |

### 9.3 Company and account values from the chart of accounts

For every company that has a chart of accounts, in company-tree order:

- the company's inventory journal, inventory valuation account, production
  work-in-progress account and production work-in-progress overhead account are loaded
  from the chart's company data;
- the variation account and the closing expense account are loaded onto the accounts the
  chart names; for an account the chart names but that does not yet exist, the chart's
  full account definition is merged in so that the account is created.

When manufacturing accounting is installed, a second hook additionally loads the
category production account from the chart's template data.

### 9.4 Message subtype

| Record | Values |
|---|---|
| Landed cost validated subtype | name **"Done"**, applies to the landed cost document, description **"Landed cost validated"** |

A landed cost document that moves to the `done` state posts its tracking message under
this subtype; every other change uses the generic subtype.

### 9.5 Saved filter

| Record | Values |
|---|---|
| Inventory Valuation filter on the invoice analysis | name **"Inventory Valuation"**, on the invoice analysis report, condition "the product is storable", grouped by product, pivot column grouped by invoice month, measured by the inventory value |

---

## 10. Security groups

| Group | Created by | Purpose |
|---|---|---|
| Display Serial and Lot Number on Invoices (`stock_account.group_lot_on_invoice`) | this domain | Members see the lot and serial number table on the printed invoice. Granted by the corresponding setting. |
| Inventory manager (`stock.group_stock_manager`) | the inventory operations domain | The main holder of valuation rights here: reads accounts and journals, sees stock quantity values, manages landed costs, sees the unit cost history. |
| Accounting read-only, accounting invoicing, accounting user, accounting manager | the accounting domain | Consume valuation data; the manager may post the work-in-progress entry. |
| Technical features (`base.group_no_one`) | the base domain | Sees the closing expense account field and the "update quantities on the order" switch on return lines. |
| Multi-company (`base.group_multi_company`) | the base domain | Sees the company column on the landed cost list and form. |

---

## 11. Access rights matrix

| Record | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| Account (`account.account`) | Inventory manager | no | **yes** | no | no |
| Journal (`account.journal`) | Inventory manager | no | **yes** | no | no |
| Product Value (`product.value`) | Inventory manager | **yes** | **yes** | **yes** | **yes** |
| Transfer (`stock.picking`) | Accounting read-only | no | **yes** | no | no |
| Transfer (`stock.picking`) | Accounting invoicing | **yes** | **yes** | **yes** | no |
| Goods Movement (`stock.move`) | Accounting read-only | no | **yes** | no | no |
| Goods Movement (`stock.move`) | Accounting invoicing | **yes** | **yes** | **yes** | no |
| Average Cost History (`stock.avco.report`) | Accounting read-only | no | **yes** | no | no |
| Average Cost History (`stock.avco.report`) | Inventory manager | no | **yes** | no | no |
| Landed Cost (`stock.landed.cost`) | Inventory manager | **yes** | **yes** | **yes** | **yes** |
| Landed Cost Line (`stock.landed.cost.lines`) | Inventory manager | **yes** | **yes** | **yes** | **yes** |
| Valuation Adjustment Line (`stock.valuation.adjustment.lines`) | Inventory manager | **yes** | **yes** | **yes** | **yes** |
| Bill of Materials (`mrp.bom`) | Accounting read-only | no | **yes** | no | no |
| Bill of Materials (`mrp.bom`) | Accounting invoicing | no | **yes** | no | no |
| Bill of Materials Line (`mrp.bom.line`) | Accounting read-only | no | **yes** | no | no |
| Bill of Materials Line (`mrp.bom.line`) | Accounting invoicing | no | **yes** | no | no |
| Work In Progress Wizard (`mrp.account.wip.accounting`) | Accounting manager | **yes** | **yes** | **yes** | no |
| Work In Progress Wizard Line (`mrp.account.wip.accounting.line`) | Accounting manager | **yes** | **yes** | **yes** | **yes** |

Nothing in this domain grants write, create or delete on the average cost history,
because it is a database view.

---

## 12. Record rules

All three are multi-company rules of the same shape: a record is visible when its company
is among the companies in the current scope. They are declared as non-updatable data, so
a customisation of them survives an upgrade.

| Rule name | Applies to | Condition |
|---|---|---|
| Stock Average Cost Report multi-company | Average Cost History (`stock.avco.report`) | the row's company is in the current scope |
| Product Value multi-company | Product Value (`product.value`) | the record's company is in the current scope |
| stock_landed_cost multi-company | Landed Cost (`stock.landed.cost`) | the document's company is in the current scope |

---

## 13. Scheduled jobs

### 13.1 Inventory Valuation Closing

| Attribute | Value |
|---|---|
| Name | **"Stock Account: Inventory Valuation Closing"** |
| Acts on | the company record |
| Kind | code |
| Body | runs the closing job routine |
| Active | yes |
| Runs as | the root user |
| Interval | every 1 day |
| Force-created | yes (the record is created even if it was deleted) |

**What it does.**

1. Build the list of periods to process: always `daily`; plus `monthly` when today is the
   last day of the month.
2. Select every company whose inventory period is in that list.
3. For each, run the closing with automatic posting on, marking the run as coming from
   the scheduled job.
4. If the closing raises a user-facing error for a company — nothing to close, no
   journal, no valuation account, or a date conflict — **skip that company silently** and
   continue with the next.

Companies whose inventory period is `manual` are never processed.

> The last-day-of-month test is written as "today equals the thirty-first day of the
> current month", which the date arithmetic resolves to the actual last day of the month
> — the twenty-eighth or twenty-ninth in February, the thirtieth in April, and so on.

---

## 14. Decimal precisions consumed

| Precision name | Default | Where this domain uses it |
|---|---|---|
| Product Price | 2 | The display of unit costs; the tolerance of the "is the cost-of-goods-sold unit price non-zero" test; the tolerance of the "does the bill line's unit price match its computed unit price" test; the rounding of the purchase order line's unit price for stock. |
| Product Unit | 2 | The tolerance of quantity comparisons in the running replay of the unit cost history. |
| Stock Weight | 2 | The display of the weight on a valuation adjustment line. |
| Volume | 2 | The display of the volume on a valuation adjustment line. |
| Percentage Analytic | (set by the analytic domain) | The rounding of each analytic share. |

The currency's own decimal places govern every monetary amount.

---

## 15. Module composition

| Capability | Depends on | Installed automatically? |
|---|---|---|
| Inventory valuation core | inventory operations and accounting | **yes**, as soon as both are present |
| Landed costs | the valuation core and purchasing integration | no — installed by the corresponding setting |
| Landed costs on manufacturing orders | landed costs and manufacturing | with manufacturing landed costs |
| Landed costs on subcontracted manufacturing | landed costs on manufacturing and subcontracting | with subcontracting landed costs |
| Manufacturing accounting | manufacturing and the valuation core | **yes**, as soon as both are present |
| Purchasing valuation integration | purchasing and inventory operations | with purchasing |
| Sales cost-of-goods-sold integration | sales and inventory operations | with sales |
| Drop shipping | purchasing, sales and inventory operations | no |

---

## 16. Installation hooks in order

1. **Configure journals** — adopt or create the `STJ` journal per company, in company
   tree order, and apply the chart's inventory journal and valuation account values.
2. **Create initial costs** — one valuation history record per company per
   consumable-type product, described "Initial cost".
3. **Configure company data** — load the chart's company values (inventory journal,
   inventory valuation account, both work-in-progress accounts) and the chart's account
   values (variation account, closing expense account).

When manufacturing accounting is installed, a further hook loads the chart's category
production account for every company that has a chart of accounts, in company tree order.

---

## 17. Configuration checklist for a working perpetual, anglo-saxon setup

1. Company: valuation mode `real_time`, costing method as desired, anglo-saxon flag on,
   inventory journal set, inventory valuation account set.
2. On the inventory valuation account: variation account set (needed by the closing when
   the physical value and the ledger ever diverge).
3. Category: costing method and valuation mode set or deliberately left to the company;
   inventory valuation account set if it should differ from the company's; price
   difference account set if the costing method is `standard`.
4. Inventory-loss locations: valuation account set, otherwise inventory adjustments post
   nothing and the difference only surfaces at the closing.
5. Scrap locations: valuation account set, for the same reason.
6. Production locations: valuation account set, otherwise neither the component
   consumption nor the finished-goods entry nor the labour entry is produced.
7. Products: expense account reachable (own, category tree, or company default),
   otherwise no cost-of-goods-sold line is injected at the invoice.
8. Inventory period: `manual`, `daily` or `monthly` as desired.

## 18. Configuration checklist for a working periodic setup

1. Company: valuation mode `periodic`, costing method as desired, inventory journal set,
   inventory valuation account set.
2. On the inventory valuation account: variation account set — without it, and without a
   company default expense account, the account is silently skipped by the closing and
   the books never move.
3. Inventory-loss, scrap and production locations: valuation account set if the closing
   should reclassify the goods that passed through them.
4. Inventory period: `daily` or `monthly` if the closing should be posted automatically.
5. No price difference account is needed; no cost-of-goods-sold lines are injected.
