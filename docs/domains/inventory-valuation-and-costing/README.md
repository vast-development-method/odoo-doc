# Inventory Valuation and Costing

## Purpose of this domain

This domain answers two questions for every unit of physical goods the company holds or
moves:

1. **What is it worth?** — the monetary value carried by the goods on hand, and the
   monetary value that entered or left the company when goods moved.
2. **Where does that value appear in the books?** — which accounts are debited and
   credited, when, and by how much, so that the balance of the inventory asset accounts
   in the general ledger can be reconciled against the physical stock.

The domain owns the *costing methods* (standard price, first in first out, average cost),
the *valuation modes* (periodic closing versus perpetual recognition), the *value carried
on each completed goods movement*, the *manual valuation corrections*, the *landed cost*
mechanism that adds freight, duties and handling to goods already received, the
*recognition of the cost of goods sold* under both the anglo-saxon and the continental
conventions, and the *inventory valuation closing report* that proposes the journal
entries reconciling the ledger with the physical inventory.

It does **not** own the physical movement of goods itself. Quantities, reservations,
locations, lots, transfers and their state machine belong to
[Inventory Operations](../inventory-operations/README.md). This domain reads those
records and attaches value to them.

## Capabilities covered

| Capability | Summary |
|---|---|
| Costing method selection | Per product category, per company, with a company-level fallback. Three methods: standard price, first in first out, average cost. |
| Valuation mode selection | Per product category, per company, with a company-level fallback. Two modes: periodic (at closing) and perpetual (at invoicing). |
| Valuation by lot or serial number | Optional per product; each lot or serial number then carries its own unit cost and total value. |
| Incoming valuation | The value attached to a completed goods receipt, built from a priority chain: manual correction, vendor bill, production order, purchase or sales quotation, originating return, product cost — plus landed costs. |
| Weighted average update | The recomputation of the product cost each time goods enter, including the incremental fast path and the full replay path. |
| Outgoing valuation | The value removed from stock by a completed delivery, consumption, scrap or vendor return, computed per costing method. |
| First in first out consumption | The stack-building algorithm that walks completed incoming movements backwards until the quantity on hand is covered, then consumes forwards, with extrapolation when the stack runs out. |
| Negative stock handling | Outgoing movements valued at the last known cost when no goods were on hand, and the automatic correction when goods later arrive at a different price. |
| Standard price changes and revaluation | Manual cost changes recorded as dated valuation-history records; their effect on the value of stock on hand at any date. |
| Manual valuation adjustment of a single movement | The "adjust valuation" action, its record, and its precedence over every computed value. |
| Costing method changes | What happens to the product cost and to the lot costs when a category switches method. |
| Cost of goods sold recognition | The anglo-saxon convention (interim stock account at delivery, expense at invoice) and the continental convention (expense at bill, variation posted at closing). |
| Journal entries for goods movements | Entries produced when a movement crosses a location that carries a valuation account, under perpetual valuation. |
| Landed costs | Creation from a vendor bill or by hand, the five split methods, the adjustment computation with its rounding correction, validation, and the journal entry produced. |
| Inventory valuation closing | The periodic computation that compares the physical inventory value against the ledger balance and proposes the balancing entry, plus the scheduled job that posts it. |
| Valuation at a date | Re-running any costing method as of a past instant, including the dated valuation-history records. |
| Product value and quantity reporting fields | Total value, average cost, valuation currency, remaining quantity and remaining value on movements, and quant value. |
| Work in progress accounting for production | The wizard that posts a work-in-progress entry and its automatic reversal. |
| Reporting | The inventory valuation closing report, the unit cost history report, the valuation movement list, and the inventory value shown on the forecast report. |

## Entity list

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Product Category | `product.category` | `product_category` | Carries the costing method, the valuation mode, the inventory journal and the valuation, variation and price-difference accounts, all per company. |
| Product Template | `product.template` | `product_template` | Exposes the effective costing method and valuation mode for the current company, and the "valuation by lot or serial number" switch. |
| Product Variant | `product.product` | `product_product` | Carries the unit cost, the computed total value, the computed average cost and the valuation currency; owns the costing algorithms. |
| Product Value | `product.value` | `product_value` | Dated history record of every manual value change: a new unit cost for a product, a new unit cost for a lot, or a new total value for one completed goods movement. |
| Stock Move | `stock.move` | `stock_move` | A completed goods movement carries the value that entered or left the company, the incoming and outgoing valuation flags, the remaining quantity and remaining value, and the link to the journal entry it produced. |
| Stock Move Line | `stock.move.line` | `stock_move_line` | The detailed line whose picked flag, source and destination location and owner decide whether the parent movement counts as incoming, outgoing or neither. |
| Stock Quantity | `stock.quant` | `stock_quant` | Carries the computed monetary value of the goods held at one location, the costing method shown to the user, and the accounting date override used for inventory adjustments. |
| Lot or Serial Number | `stock.lot` | `stock_lot` | When valuation by lot is enabled, carries its own unit cost, computed total value and computed average cost. |
| Stock Location | `stock.location` | `stock_location` | Carries the counterpart valuation account used when goods leave the valued perimeter through it, and the two computed flags that say whether it is inside or outside the valued perimeter. |
| Company | `res.company` | `res_company` | Carries the fallback costing method, the fallback valuation mode, the inventory journal, the inventory valuation account, the work-in-progress accounts, the closing period and the anglo-saxon flag. |
| Account | `account.account` | `account_account` | Carries the variation account and the closing expense account attached to an inventory valuation account. |
| Journal Entry | `account.move` | `account_move` | Receives the valuation entries, the cost-of-goods-sold lines, the price-difference lines, the landed cost entries, the closing entry and the work-in-progress entry. |
| Journal Item | `account.move.line` | `account_move_line` | Carries the cost-of-goods-sold origin link and the landed-cost-line flag. |
| Landed Cost | `stock.landed.cost` | `stock_landed_cost` | A document that spreads additional costs over the goods received (or produced) by one or more transfers. |
| Landed Cost Line | `stock.landed.cost.lines` | `stock_landed_cost_lines` | One additional cost to spread, with its amount, split method and counterpart account. |
| Valuation Adjustment Line | `stock.valuation.adjustment.lines` | `stock_valuation_adjustment_lines` | The computed share of one landed cost line allocated to one goods movement. |
| Average Cost History | `stock.avco.report` | database view `stock_avco_report` | Read-only chronological justification of the average cost of a product, movement by movement and adjustment by adjustment. |
| Inventory Valuation Report | `stock_account.stock.valuation.report` | none (computed) | Read-only report that compares physical inventory value with ledger balance and proposes the closing entry. |
| Inventory Adjustment Naming Wizard | `stock.inventory.adjustment.name` | transient | Collects the reference and the accounting date applied to the goods movements produced by an inventory adjustment. |
| Return Line Wizard | `stock.return.picking.line` | transient | Carries the "update quantities on the order" switch propagated onto the return movement. |
| Work In Progress Accounting Wizard | `mrp.account.wip.accounting` | transient | Builds and posts the work-in-progress entry for manufacturing orders still in progress, with an automatic reversal. |
| Work In Progress Accounting Line | `mrp.account.wip.accounting.line` | transient | One proposed debit or credit of the work-in-progress entry. |

Entities that belong to other domains but gain fields here are listed with only the added
fields in [entities.md](entities.md): Manufacturing Order (`mrp.production`), Work Center
(`mrp.workcenter`), Work Center Productivity (`mrp.workcenter.productivity`), Work Order
(`mrp.workorder`), Purchase Order Line (`purchase.order.line`), Analytic Account
(`account.analytic.account`), Analytic Line (`account.analytic.line`) and Analytic Plan
(`account.analytic.plan`).

## Reading order

1. **[README.md](README.md)** — this file. Scope, vocabulary, dependencies.
2. **[glossary.md](glossary.md)** — read this early; the rest of the domain uses the
   terms defined there with exact meanings (valued perimeter, incoming movement,
   remaining quantity, valuation history record, interim recognition).
3. **[entities.md](entities.md)** — every record type with its complete field table.
4. **[state-machines.md](state-machines.md)** — the landed cost lifecycle, the valuation
   status of a goods movement, the closing entry lifecycle.
5. **[calculations.md](calculations.md)** — every formula: the value-priority chain, the
   three costing methods, the average update, the first in first out stack, the landed
   cost split, the closing balance, the cost of goods sold unit price.
6. **[accounting-effects.md](accounting-effects.md)** — every journal entry, line by
   line, with account selection rule, side and amount formula.
7. **[workflows.md](workflows.md)** — the end-to-end operational sequences.
8. **[business-rules.md](business-rules.md)** — validations, constraints, invariants,
   exact error messages, permission checks, locking.
9. **[configuration.md](configuration.md)** — settings, parameters, sequences, default
   records, groups, access rights, record rules, scheduled jobs.
10. **[interfaces.md](interfaces.md)** — navigation, views, named operations, reports.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered scenarios with
    concrete numbers. Use these as the conformance suite.

## Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [Inventory Operations](../inventory-operations/README.md) | Goods movements and their completion algorithm, movement lines with their picked flag and owner, locations with their usage, quantities on hand, lots and serial numbers, transfers, returns, scrap, inventory adjustments. The valuation engine hooks into the completion of a movement and into the creation and modification of movement lines. |
| [General Ledger](../general-ledger/README.md) | Journal entries and journal items, journals, accounts, posting, reversal, fiscal lock dates, currency rounding, company currency. |
| [Products and Catalog](../products-and-catalog/README.md) | Products, product categories, the unit cost field, the storable flag, the tracking mode, the weight and volume used by two landed cost split methods, the income and expense account selection. |
| [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md) | Conversion of quantities between the movement unit of measure and the product reference unit of measure; every valuation quantity is expressed in the product reference unit of measure. |
| [Multi-currency](../multi-currency/README.md) | Conversion of a vendor bill amount expressed in a foreign currency into the company currency at the bill rate; rounding to the company currency. |
| [Analytic Accounting](../analytic-accounting/README.md) | Analytic distribution applied to goods movements, and the creation and update of analytic lines mirroring the value of a movement. |
| [Purchasing](../purchasing/README.md) | Purchase order lines supply the expected unit price of a receipt; vendor bills supply the actual price; the price-difference mechanism under standard costing. |
| [Sales](../sales/README.md) | Sales order lines link deliveries to customer invoices, which is how the cost of goods sold is matched to the goods actually shipped. |
| [Manufacturing](../manufacturing/README.md) | Manufacturing orders supply the production cost of finished goods and by-products; work centers supply the labour cost; the production location supplies the counterpart account. |
| [Accounts Payable](../accounts-payable/README.md) | Vendor bills, their posting, their lines and the landed cost creation from a bill. |
| [Accounts Receivable](../accounts-receivable/README.md) | Customer invoices and credit notes, their posting, and the additional cost-of-goods-sold lines injected at posting time. |

## Domains that depend on this one

- [Financial Reporting](../financial-reporting/README.md) reads the inventory valuation
  accounts and the closing entries.
- [Sales](../sales/README.md) reads the unit cost to compute margins.
- [Manufacturing](../manufacturing/README.md) reads component values to price finished
  goods.
- [Purchasing](../purchasing/README.md) reads the effect of a bill on the value of the
  goods already received.

## Two orthogonal choices that drive everything

The behaviour of this domain is governed by two independent selections made on the
product category (falling back to the company):

**Costing method** — how the monetary value of one unit is determined:

| Value | Label | Meaning |
|---|---|---|
| `standard` | Standard Price | The unit cost is a number a human maintains. Movements are valued at that number. It never changes by itself. |
| `fifo` | First In First Out | Goods that entered first are considered to leave first. An outgoing movement is valued by consuming the oldest still-unconsumed incoming movements. |
| `average` | Average Cost | The unit cost is the weighted average of everything that entered and is still on hand. It is recomputed on every entry. |

**Valuation mode** — when and whether the ledger is touched:

| Value | Label | Meaning |
|---|---|---|
| `periodic` | Periodic (at closing) | Goods movements never post anything by themselves. The inventory asset accounts are corrected by a closing entry, produced manually or by the daily or monthly scheduled job. |
| `real_time` | Perpetual (at invoicing) | A goods movement that crosses a location holding a counterpart valuation account posts a journal entry immediately; invoices and bills post the interim and expense entries. |

A third company-level switch, the anglo-saxon flag, decides **where the cost lands**
under perpetual valuation: with the flag on, the cost of goods sold is recognised at the
customer invoice and the vendor bill posts against the inventory asset; with the flag
off, the purchase is expensed at the bill and the closing entry moves the period
variation.

## Conventions used throughout this folder

- All monetary amounts are expressed in the **company currency** unless the text says
  otherwise. Conversion from a document currency to the company currency happens exactly
  where stated and nowhere else.
- All quantities used in valuation are expressed in the **product reference unit of
  measure**, never in the movement unit of measure, unless the text says otherwise.
- "Completed" means the goods movement has reached its terminal successful state; the
  storage value of that state is `done`.
- A *reproduced identifier* is written in code font and is accompanied by its full name
  in words on first use in each file; these identifiers are part of the external contract
  and must be reproduced exactly.
- Where the code leaves a behaviour implicit, the text states the behaviour explicitly
  and marks it with the phrase **industry-standard default**.
