# Cross-domain transactions

A business event almost never stops inside one domain. Confirming a sales order writes a
status, raises a procurement request, creates a transfer and schedules an invoice; validating
that transfer moves goods and changes what they are worth; posting the invoice recognises
revenue, tax, a receivable and the cost of the goods that left; settling the receivable moves
money through an outstanding account, then through a bank transaction, and closes the
receivable by reconciliation. Each domain folder specifies its own part of that chain
completely. This file follows the chain itself.

Every domain folder remains the authority for the rule it owns. A trace never restates a rule;
it applies it, names the file that specifies it, and shows the number that comes out. Where a
trace and a domain file disagree, the domain file is the one to correct.

---

## What a trace is

A trace is one business event followed from its first record to its last journal item, across
every domain it touches, with the money carried to the last unit of currency. Every trace in
this file has the same six parts.

**(a) Starting records.** Concrete and complete: the company and its currency, the products
with their costs, prices, units of measure and accounting configuration, the taxes with their
rates and accounts, the accounts themselves, the journals, the warehouse and its routes, the
payment terms, the partner and the dates. A trace that starts from an abstract setting cannot
prove an amount, so nothing here is left to the reader to supply.

**(b) Steps.** A numbered list. Each step names the operation, the domain that owns it with a
relative link to the file in that domain's folder that specifies it, the records the step
creates or changes with their key field values, and the state transitions it causes.

**(c) The ledger.** One table listing every journal item the whole trace produces, in the
order the entries are posted, with the journal, the account, the debit, the credit, the amount
in the document currency and the rate where the document is in a foreign currency, the
counterparty, the accounting date, and what the item is reconciled against. Every ledger table
ends with proven totals, and every account that should return to zero is shown returning to
zero.

**(d) Stock consequences.** The quantity held at each location after the trace, and the value
carried by every completed goods movement: the value that entered or left the company because
of it, the part of its quantity still considered on hand, and the value of that part. The
platform stores inventory value **on the goods movements themselves** rather than in separate
layer records, so the movement table *is* the valuation layer table; the fields are specified
in [inventory-valuation-and-costing/entities.md](inventory-valuation-and-costing/entities.md).

**(e) Variations.** The configuration choices that change the trace, each stated as a delta
against the ledger of part (c): the invoicing policy, the costing method, the anglo-saxon and
continental treatments, a foreign currency, cash rounding, an early payment discount, and
several companies.

**(f) Failure points.** Every place the trace can be refused, with the exact text the system
shows, and the domain whose rule refuses.

---

## How to read the ledger tables

- **Side.** Every ledger table has a debit column and a credit column, as a reader of the
  ledger sees them. Internally each journal item carries one signed **balance** in company
  currency: a positive balance is a debit, a negative balance is a credit. Under storno
  accounting the two columns are derived from the balance the other way round for items marked
  as storno; that presentation is specified in
  [accounts-receivable/accounting-effects.md](accounts-receivable/accounting-effects.md) and
  changes no amount.
- **Amount in currency.** Every accountable item also carries an amount in the item's own
  currency, with the same sign as the balance. When the item's currency is the company
  currency the two figures are equal, and the traces below omit the column unless a foreign
  currency is in play.
- **Date.** Every item of an entry carries the entry's accounting date. A payment term item
  additionally carries a **maturity date**, which is the date the instalment falls due and is
  not the accounting date.
- **Counterparty.** On every accountable item of an invoice or a bill the counterparty is the
  **commercial entity** of the document's partner, never a child contact. Posting rewrites any
  item that disagrees.
- **Reconciled against.** Only items on a reconcilable account can be reconciled; in practice
  that means receivable items, payable items, outstanding-receipt and outstanding-payment
  items, the inter-bank transfer account and the point-of-sale receivable account. A
  reconciliation writes **no** accounting of its own: it records partial reconciliation links
  and, where rates differ, causes a separate exchange-difference entry.
- **Display kind.** Items marked as cost-of-goods-sold items are *injected* into an invoice or
  a bill at posting time rather than typed by a user. They are dropped when the document is
  copied and deleted when it is reset to draft or cancelled. The traces mark them.

---

## Conventions for amounts and rounding

1. **The currency step.** Every currency carries a rounding step and a number of decimal
   places. The company currency of every trace in this file is the euro, whose step is one
   hundredth and whose display carries two decimals.
2. **The rounding method.** "Round to the currency" means round to the currency's step with
   the **half away from zero** method, which is the default of the tax engine everywhere. The
   exact primitive, including its tie-correction, is in
   [taxes/calculations.md](taxes/calculations.md). Only the cash rounding feature can select
   another method.
3. **Where rounding happens.** Each step of a trace states what it rounds. The general order
   is: a line's untaxed total is computed unrounded from quantity, unit price and discount,
   then rounded to the currency; each tax amount is computed from the unrounded base and
   rounded to the currency; the document totals are the sums of the rounded figures, with the
   engine's delta redistribution applied so that the visible subtotals and the visible totals
   agree; the payment term instalments are rounded individually except the last, which takes
   the whole remaining residual so that the instalments add up exactly.
4. **Unit costs.** A product's unit cost is stored with the decimal precision named
   `Product Price`, which these traces set to two decimals. A quantity is compared and rounded
   with the decimal precision named `Product Unit`, also two decimals here. A value that a
   trace carries at more decimals than the stored precision is shown at full length and then
   at its stored length, so that the loss is visible.
5. **Currency conversion.** A rate is written as the number of units of the foreign currency
   for one unit of the company currency. Converting a foreign amount into the company currency
   therefore **divides** by the rate, and the result is rounded to the company currency. The
   payment term distribution uses the *effective* rate, derived by dividing the document's
   foreign total by its company-currency total, rather than the stored rate, so that the
   instalments add up in both currencies.
6. **Signs.** A customer invoice's direction sign is minus one and a customer credit note's is
   plus one; a vendor bill's is plus one and a vendor credit note's is minus one. The sign
   multiplies the tax engine's positive totals to produce the stored balances, which is why a
   customer invoice's revenue item is a credit and a vendor bill's expense item is a debit. The
   full derivation is in
   [accounts-receivable/accounting-effects.md](accounts-receivable/accounting-effects.md).
7. **Quantities in the reference unit.** Valuation always works in the product's reference unit
   of measure. When a document line uses another unit, the trace shows the conversion before
   the value is computed.

---

## The twelve traces

| # | Trace | Principal domains |
|---|---|---|
| 1 | [Quote to cash with delivered goods](#1-quote-to-cash-with-delivered-goods) | sales, inventory operations, inventory valuation and costing, taxes, accounts receivable, payments and bank reconciliation |
| 2 | [Procure to pay with received goods](#2-procure-to-pay-with-received-goods) | replenishment and procurement, purchasing, inventory operations, inventory valuation and costing, accounts payable, payments and bank reconciliation |
| 3 | [Make to stock and make to order with components and work orders](#3-make-to-stock-and-make-to-order-with-components-and-work-orders) | manufacturing, replenishment and procurement, inventory operations, inventory valuation and costing, sales, accounts receivable |
| 4 | [Point of sale session from opening to closing](#4-point-of-sale-session-from-opening-to-closing) | point of sale, taxes, inventory valuation and costing, accounts receivable, payments and bank reconciliation |
| 5 | [Storefront checkout with online payment, shipping and promotion](#5-storefront-checkout-with-online-payment-shipping-and-promotion) | website and storefront, loyalty and promotions, delivery and shipping, payment providers, sales, accounts receivable |
| 6 | [Expense to reimbursement with customer re-invoicing](#6-expense-to-reimbursement-with-customer-re-invoicing) | expenses, analytic accounting, accounts payable, sales, accounts receivable |
| 7 | [Timesheet to invoice on a service project](#7-timesheet-to-invoice-on-a-service-project) | timesheets, projects and tasks, analytic accounting, sales, accounts receivable |
| 8 | [Return, credit note and refund](#8-return-credit-note-and-refund) | inventory operations, inventory valuation and costing, sales, accounts receivable, payments and bank reconciliation |
| 9 | [Drop shipping and inter-company trade](#9-drop-shipping-and-inter-company-trade) | replenishment and procurement, purchasing, sales, inventory valuation and costing, accounts payable, accounts receivable |
| 10 | [Bank statement to reconciliation with write-off and exchange difference](#10-bank-statement-to-reconciliation-with-write-off-and-exchange-difference) | payments and bank reconciliation, multi currency, taxes, general ledger |
| 11 | [Landed costs and price differences on standard cost](#11-landed-costs-and-price-differences-on-standard-cost) | inventory valuation and costing, purchasing, accounts payable |
| 12 | [Hire to work: applicant, employee, working time, absence and work entries](#12-hire-to-work-applicant-employee-working-time-absence-and-work-entries) | recruitment, human resources core, attendances and working time, time off, work entries |

---

## The shared fixture

Traces 1 to 4 share one company, one chart of accounts and one set of journals, so that the
account balances of one trace can be read against another. Each trace then adds the products,
partners and configuration it needs.

**Company.** Northwind Trading. Company currency: the euro, step one hundredth, two decimal
places. Fiscal year: the calendar year. Storno accounting: off. Anglo-saxon accounting: on
unless a variation says otherwise. Decimal precision `Product Price`: two. Decimal precision
`Product Unit`: two.

**Accounts.**

| Code | Name | Kind | Reconcilable |
|---|---|---|---|
| 1010 | Bank | Current asset, of the bank kind | No |
| 1020 | Cash | Current asset, of the cash kind | No |
| 1099 | Suspense Account | Current asset | Yes |
| 1150 | Point of Sale Receivable | Current asset | Yes |
| 1200 | Trade Receivables | Receivable | Yes |
| 1310 | Tax Receivable | Current asset | No |
| 1400 | Inventory | Current asset | No |
| 1410 | Inventory Variation | Expense | No |
| 1450 | Outstanding Receipts | Current asset | Yes |
| 1455 | Outstanding Payments | Current asset | Yes |
| 1499 | Inventory Loss | Expense | No |
| 2100 | Trade Payables | Payable | Yes |
| 2510 | Tax Payable | Current liability | No |
| 4000 | Product Sales | Income | No |
| 5000 | Cost of Goods Sold | Expense | No |
| 5100 | Price Difference | Expense | No |
| 5200 | Production Cost | Expense | No |
| 5300 | Manufacturing Overhead | Expense | No |
| 6560 | Foreign Exchange Loss | Expense | No |
| 6580 | Cash Rounding Loss | Expense | No |
| 6581 | Cash Difference Loss | Expense | No |
| 6900 | Cash Discount Granted | Expense | No |
| 7560 | Foreign Exchange Gain | Income | No |
| 7580 | Cash Rounding Gain | Income | No |
| 7581 | Cash Difference Gain | Income | No |
| 7900 | Cash Discount Taken | Income | No |

**Journals.**

| Name | Kind | Default account | Suspense account | Profit account | Loss account |
|---|---|---|---|---|---|
| Customer Invoices | Sale | 4000 Product Sales | — | — | — |
| Vendor Bills | Purchase | 5000 Cost of Goods Sold | — | — | — |
| Bank | Bank | 1010 Bank | 1099 Suspense Account | 7581 Cash Difference Gain | 6581 Cash Difference Loss |
| Cash | Cash | 1020 Cash | 1099 Suspense Account | 7581 Cash Difference Gain | 6581 Cash Difference Loss |
| Inventory Valuation | General | — | — | — | — |
| Point of Sale | General | 4000 Product Sales | — | — | — |
| Exchange Difference | General | — | — | — | — |
| Miscellaneous | General | — | — | — | — |

**Company-level settings used by the traces.** Inventory journal: Inventory Valuation.
Inventory valuation account: 1400 Inventory. Exchange gain account: 7560 Foreign Exchange Gain.
Exchange loss account: 6560 Foreign Exchange Loss. Cash discount write-off loss account: 6900
Cash Discount Granted. Cash discount write-off gain account: 7900 Cash Discount Taken. Default
point-of-sale receivable account: 1150 Point of Sale Receivable. Inventory period: manual.
Production work-in-progress account and production work-in-progress overhead account: unset,
because traces 1 to 4 never run the work-in-progress assistant.

**Taxes.**

| Name | Rate | Scope | Computation | Account of the invoice repartition | Account of the refund repartition |
|---|---|---|---|---|---|
| Sales 21 % | 21 % | Sale | Percentage, price-excluded | 2510 Tax Payable | 2510 Tax Payable |
| Sales 21 % included | 21 % | Sale | Percentage, price-included | 2510 Tax Payable | 2510 Tax Payable |
| Purchase 21 % | 21 % | Purchase | Percentage, price-excluded | 1310 Tax Receivable | 1310 Tax Receivable |

All three taxes have one hundred per cent of the base on a single base repartition line and one
hundred per cent of the tax on a single tax repartition line, and none is a cash-basis tax.

**Warehouse.** One warehouse, *Main Warehouse*, whose locations are `WH/Stock` (internal),
`WH/Input` (internal), `WH/Production` (production) and `WH/Output` (internal). The shared
partner locations are `Partners/Vendors` (vendor kind) and `Partners/Customers` (customer
kind), and the company's inventory adjustment location is `Virtual Locations/Inventory
adjustment` (inventory-loss kind), which carries the location valuation account 1499 Inventory
Loss. `WH/Production` carries the location valuation account 5200 Production Cost. No other
location carries a valuation account. The valued perimeter is therefore `WH/Stock`, `WH/Input`
and `WH/Output`; every other location named above is outside it.

**Why that matters everywhere below.** Under perpetual valuation a goods movement posts a
journal entry **only** when at least one of its two locations carries a location valuation
account ([inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)).
A receipt from a vendor location into the warehouse and a delivery from the warehouse to a
customer location therefore post **nothing by themselves**: the inventory asset is debited by
the vendor bill line, which is redirected onto the inventory valuation account, and it is
credited by the cost-of-goods-sold pair injected into the customer invoice at posting. Only
movements through the production location, the inventory-loss location and the scrap location
post an entry of their own. Traces 1 to 4 show both halves of that arrangement.

---

# 1. Quote to cash with delivered goods

A salesperson quotes goods and a service, the customer accepts, the warehouse ships in two
rounds, each round is invoiced, and the customer settles by bank transfer. The trace crosses
[sales](sales/), [replenishment and procurement](replenishment-and-procurement/),
[inventory operations](inventory-operations/),
[inventory valuation and costing](inventory-valuation-and-costing/), [taxes](taxes/),
[accounts receivable](accounts-receivable/) and
[payments and bank reconciliation](payments-and-bank-reconciliation/).

## 1.a Starting records

Everything of the [shared fixture](#the-shared-fixture) applies. In addition:

**Product — Desk lamp.** A goods product, storable. Reference unit of measure `Units`.
Costing method **first in first out**; valuation mode **perpetual**. Invoicing policy
**delivered quantities**. Sales price 149.95. Customer tax: Sales 21 %. Product category
*Lighting*, which carries the income account 4000 Product Sales, the expense account 5000 Cost
of Goods Sold, the inventory valuation account 1400 Inventory and the inventory journal
Inventory Valuation. The product itself declares no income or expense account, so both are
taken from the category by the walk specified in
[inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md).
Routes: the warehouse's one-step delivery route. Removal strategy at `WH/Stock`: first in first
out.

**Product — Installation service.** A service product. Invoicing policy **ordered quantities**
(the prepaid, fixed-price policy). Sales price 300.00. Customer tax: Sales 21 %. Income account
4000 Product Sales through the same category. Not storable, so it never reaches inventory or
valuation.

**Opening stock of the Desk lamp.** Two completed incoming goods movements, both already billed
and both posted to the ledger through their vendor bills:

| Movement | Date | Source | Destination | Quantity | Value | Unit value |
|---|---|---|---|---|---|---|
| R1 | 2026-01-08 | `Partners/Vendors` | `WH/Stock` | 40 `Units` | 3 200.00 | 80.00 |
| R2 | 2026-02-03 | `Partners/Vendors` | `WH/Stock` | 10 `Units` | 880.00 | 88.00 |

Quantity on hand 50 `Units`; total value 4 080.00; stored unit cost 4 080.00 ÷ 50 = **81.60**.
Account 1400 Inventory carries 4 080.00 for this product, debited by the two vendor bills.

**Customer.** Baxter Interiors, its own commercial entity. Receivable account 1200 Trade
Receivables. No fiscal position. The order's delivery address is the child contact *Baxter
Interiors, Warehouse*, which declares no customer location of its own, so the shared
`Partners/Customers` location is used. The invoice address is Baxter Interiors itself, and that
is the partner written on every journal item.

**Payment term — "30 % now, balance in 30 days".** Two lines, in this stored order: (1) a
percentage line of 30 %, delay type *days after invoice date*, 0 days; (2) a balance line,
delay type *days after invoice date*, 30 days. No early payment discount. No cash rounding on
the company.

**Dates.** Quotation 2026-03-02. Sent 2026-03-03. Confirmed 2026-03-05. First delivery
2026-03-09. First invoice 2026-03-10. First payment 2026-03-10, on the bank statement
2026-03-12. Second delivery 2026-03-20. Second invoice 2026-03-23. Final payment 2026-04-09, on
the bank statement 2026-04-10.

**The quotation.** Reference `S00042`, warehouse *Main Warehouse*, currency euro.

| Line | Product | Quantity | Unit price | Discount | Tax |
|---|---|---|---|---|---|
| 1 | Desk lamp | 12 `Units` | 149.95 | 5 % | Sales 21 % |
| 2 | Installation service | 1 `Units` | 300.00 | — | Sales 21 % |

Order arithmetic, rounded as specified in [sales/calculations.md](sales/calculations.md) and
[taxes/calculations.md](taxes/calculations.md):

```formula
line_1_discounted_unit_price = 149.95 × ( 1 − 5 ÷ 100 ) = 142.4525
line_1_untaxed  = round_to_currency( 12 × 142.4525 ) = round_to_currency( 1 709.43 ) = 1 709.43
line_1_tax      = round_to_currency( 1 709.43 × 21 ÷ 100 ) = round_to_currency( 358.9803 ) = 358.98
line_2_untaxed  = round_to_currency( 1 × 300.00 ) = 300.00
line_2_tax      = round_to_currency( 300.00 × 21 ÷ 100 ) = 63.00
untaxed_total   = 1 709.43 + 300.00 = 2 009.43
tax_total       = 358.98 + 63.00 = 421.98
grand_total     = 2 009.43 + 421.98 = 2 431.41
```

The discounted unit price is **not** rounded before it is multiplied by the quantity; only the
line total and each tax amount are rounded. Rounding the unit price first would give
12 × 142.45 = 1 709.40 and a tax of 358.97, which is two cents and one cent adrift.

## 1.b Steps

1. **Create the quotation.** Domain: sales,
   [sales/workflows.md](sales/workflows.md) section 1. A Sales Order is created with status
   `draft`, order date 2026-03-02, the customer, the invoice and delivery addresses, the
   payment term and the warehouse, plus the two order lines above. The lines price themselves
   from the price list; no ledger effect whatever.

2. **Send the quotation.** Domain: sales, [sales/workflows.md](sales/workflows.md) section 3.
   Status `draft` → `sent`. A message is posted. No ledger effect.

3. **Confirm the order.** Domain: sales, [sales/workflows.md](sales/workflows.md) section 4;
   state transitions in [sales/state-machines.md](sales/state-machines.md) section 1.
   - Phase A checks that the status is `draft` or `sent` and that every non-display line
     carries a product. Both hold.
   - Phase C writes status `sale` and rewrites the order date to the confirmation instant,
     2026-03-05.
   - Phase E, step E3, launches procurement for line 1 only, because line 2 is a service. One
     procurement request is collected: product Desk lamp, 12 `Units`, final location
     `Partners/Customers`, warehouse *Main Warehouse*, deadline 2026-03-09, company Northwind
     Trading, originating order line = line 1.
   - Domain: replenishment and procurement,
     [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
     sections 1 and 4. The rule search walks `Partners/Customers` and finds the delivery route's
     pull rule: source `WH/Stock`, destination `Partners/Customers`, operation type *Delivery
     Orders*, supply method take-from-stock. The pull action creates one Stock Move of 12
     `Units`, `WH/Stock` → `Partners/Customers`, and confirms it. Confirmation classifies the
     move as `confirmed` (the supply method is take-from-stock, so no supplying need is raised)
     and groups it into a Transfer `WH/OUT/00021`.
   - Derived statuses after confirmation, by
     [sales/state-machines.md](sales/state-machines.md) sections 4 and 5: line 1 is invoiced on
     delivered quantities and nothing is delivered, so its quantity to invoice is 0 and its
     invoice status is `no`; line 2 is invoiced on ordered quantities, so its quantity to
     invoice is 1 and its invoice status is `to invoice`. The order mixes `to invoice` and
     `no`, and the one invoiceable line is not a line that may not be invoiced alone, so the
     order's invoice status is `to invoice` from the moment it is confirmed. Delivery status:
     `pending`.
   - No journal item is produced. Confirmation is a commitment, not an accounting event; the
     reasoning is in [sales/accounting-effects.md](sales/accounting-effects.md) section 1.

4. **Reserve the delivery.** Domain: inventory operations,
   [inventory-operations/workflows.md](inventory-operations/workflows.md) section 7, step 3.
   The operation type reserves at confirmation, so the reservation runs at once: 12 `Units` are
   gathered at `WH/Stock` under the first-in-first-out **removal** strategy, detail lines are
   created and the reserved counters rise. The Transfer's status becomes `assigned`. The
   removal strategy decides which physical quantities are taken; it is independent of the first
   in first out **valuation** stack, which is rebuilt from the completed incoming movements at
   the moment a movement is valued.

5. **Validate the delivery for 8 of 12 units, with a backorder.** Domain: inventory operations,
   [inventory-operations/workflows.md](inventory-operations/workflows.md) sections 13 and 14.1.
   The operator types 8 and validates on 2026-03-09.
   - The sanity check passes (the Transfer is not empty, the processed quantity is not zero, the
     product is not tracked by lot).
   - The backorder policy is *ask* and 8 is below 12, so the backorder screen opens; the
     operator accepts. The short move is split: the original move keeps a demand of 8 and the
     new move of 4 is carried into a backorder Transfer `WH/OUT/00021-001`, which is confirmed
     and reserved.
   - **Valuation, before the completion is written.** Domain: inventory valuation and costing,
     [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
     sections 2 and 5. The outgoing movement is valued against the stock situation as it stands
     *before* the goods land. The first-in-first-out stack is built by walking completed
     incoming movements newest first until the quantity on hand, 50, is covered: R2 offers 10
     and leaves 40 to cover; R1 offers 40 and leaves 0. Reversed, the stack is R1 then R2, with
     a bottom quantity of 40 on R1. Consuming 8:

     ```formula
     offered_quantity = 40                       ( the bottom quantity of R1 )
     offered_value    = 3 200.00 × 40 ÷ 40 = 3 200.00
     offered_value    = 3 200.00 × 8 ÷ 40 = 640.00      ( scaled, because 40 exceeds the 8 wanted )
     movement_D1_value = 640.00                  ( 80.00 per unit )
     ```

   - The movement completes. Its source is `WH/Stock` (inside the valued perimeter) and its
     destination is `Partners/Customers` (outside it), so it is an **outgoing** movement; its
     outgoing flag is stored, its value is 640.00, and its remaining quantity is zero because
     only incoming movements hold a remaining quantity.
   - **No journal entry is produced**: neither `WH/Stock` nor `Partners/Customers` carries a
     location valuation account. The credit to the inventory asset will come from the invoice.
   - The product's unit cost is recomputed by the first-in-first-out full path:

     ```formula
     quantity_on_hand = 50 − 8 = 42
     total_value      = 4 080.00 − 640.00 = 3 440.00
     unit_cost        = 3 440.00 ÷ 42 = 81.904761904…
     stored_unit_cost = 81.90                     ( two decimals, the Product Price precision )
     ```

   - Transfer `WH/OUT/00021` becomes `done`. Line 1's delivered quantity becomes 8, so its
     quantity to invoice becomes 8 − 0 = 8 and its invoice status becomes `to invoice`. The
     order's delivery status becomes `partial`.

6. **Create the first invoice from the order.** Domain: sales,
   [sales/workflows.md](sales/workflows.md) section 8.3, with the field mapping of
   [sales/calculations.md](sales/calculations.md) sections 8.1 and 8.2. The advance-payment
   dialogue is opened with the *Regular invoice* method; the "deduct down payments" switch is
   off, so the run is not final; the "consolidated billing" switch is off, so one invoice is
   produced per order. The selection of section 8.4 keeps line 1 (8 to invoice) and line 2
   (1 to invoice).
   A draft customer invoice `INV/2026/00311` is created in the Customer Invoices journal, with
   the partner Baxter Interiors, the delivery address, invoice date 2026-03-10, the order's
   payment term, no fiscal position, and two invoice lines each linked to its order line. No
   account is forced on either line; the receivables domain derives them.
   The invoiced quantities of the order lines rise immediately, because a **draft** invoice
   already counts ([sales/calculations.md](sales/calculations.md) section 6.1).

7. **Post the first invoice.** Domain: accounts receivable,
   [accounts-receivable/accounting-effects.md](accounts-receivable/accounting-effects.md)
   section 1, with the tax arithmetic of [taxes/calculations.md](taxes/calculations.md) and the
   payment term distribution of
   [accounts-receivable/calculations.md](accounts-receivable/calculations.md) section 3.
   - Line amounts:

     ```formula
     line_A_untaxed = round_to_currency( 8 × 142.4525 ) = round_to_currency( 1 139.62 ) = 1 139.62
     line_A_tax     = round_to_currency( 1 139.62 × 21 ÷ 100 ) = round_to_currency( 239.3202 ) = 239.32
     line_B_untaxed = 300.00
     line_B_tax     = round_to_currency( 300.00 × 21 ÷ 100 ) = 63.00
     untaxed_total  = 1 139.62 + 300.00 = 1 439.62
     tax_total      = 239.32 + 63.00 = 302.32
     grand_total    = 1 741.94
     ```

     Both lines carry the same tax and therefore the same tax repartition line, and that
     repartition line's account is the same; the accounting grouping key is identical, so the
     two tax amounts aggregate into **one** tax journal item of 302.32, not two.
   - Account selection: neither line carries a product-level income account, so the income
     account comes from the *Lighting* category, 4000 Product Sales, mapped through the
     document's fiscal position — here the identity, because there is none. The receivable
     account is Baxter Interiors' own, 1200 Trade Receivables.
   - Payment term distribution on the accounting-signed totals, whose rate is 1 because the
     document currency is the company currency:

     ```formula
     instalment_1 = round_to_currency( 1 741.94 × 30 ÷ 100 ) = round_to_currency( 522.582 ) = 522.58
                    maturity = 2026-03-10 + 0 days  = 2026-03-10
     instalment_2 = 1 741.94 − 522.58 = 1 219.36        ( the balance rule: the last line takes the residual )
                    maturity = 2026-03-10 + 30 days = 2026-04-09
     ```

   - **The cost-of-goods-sold pair is injected at posting.** Domain: inventory valuation and
     costing,
     [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
     section 2, with the arithmetic of
     [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
     section 9. Line A qualifies: the product is storable, no reachable movement is a drop
     shipment, the valuation mode is perpetual, and both accounts resolve. Line B does not: a
     service product is not storable.

     ```formula
     cogs_quantity   = 0 ( nothing posted yet ) + 8 = 8
     unit_price      = 640.00 ÷ 8 = 80.00        ( from the one completed movement, D1 )
     already_posted  = 0.00
     cogs_unit_price = | 80.00 × 8 − 0.00 | ÷ 8 = 80.00
     amount_currency = +1 × 8 × 80.00 = 640.00
     ```

     The inventory-side item carries minus that amount and is therefore a **credit** of 640.00
     on 1400 Inventory; the expense-side item carries plus it and is a **debit** of 640.00 on
     5000 Cost of Goods Sold. Both carry display kind *cost of goods sold*, the product, the
     line's unit and quantity, and a link back to the invoice line.
   - The document is numbered from the Customer Invoices journal sequence and its status becomes
     `posted`. Its payment status is `not_paid`.

8. **Register the first payment.** Domain: payments and bank reconciliation,
   [payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
   section 1. The customer transfers the first instalment on 2026-03-10. A Payment is created
   from the invoice with amount 522.58, direction inbound, counterparty kind customer, journal
   Bank, payment method *Manual Payment* whose payment account is 1450 Outstanding Receipts.
   Its entry debits 1450 and credits 1200. Confirming it posts the entry; its counterpart item
   is reconciled with the invoice's first payment-term item, which brings both residuals to
   zero for that instalment. The invoice's residual is 1 219.36, so its payment status becomes
   `partial`.

9. **Import and reconcile the bank transaction.** Domain: payments and bank reconciliation,
   [payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
   section 3. A Bank Transaction of +522.58 dated 2026-03-12 is created in the Bank journal. Its
   entry is created and posted at once: a liquidity debit of 522.58 on 1010 Bank and a
   counterpart credit of 522.58 on the journal's suspense account 1099. Matching it against the
   outstanding Payment rewrites the counterpart item onto the Payment's outstanding account,
   1450 Outstanding Receipts, and reconciles it with the Payment's liquidity item. The
   outstanding account returns to zero for this payment, the Payment becomes matched and moves
   from `in_process` to `paid`. The invoice stays `partial`, because its second instalment is
   still open.

10. **Validate the backorder.** Domain: inventory operations, then inventory valuation and
    costing. On 2026-03-20 the remaining 4 `Units` are delivered from `WH/OUT/00021-001`.
    - Stack rebuild at that moment: the quantity on hand is 42. Walking newest first, R2 offers
      10 and leaves 32; R1 offers 40, of which only 32 is needed, so R1's bottom quantity is 32.
      Reversed, the stack is R1 (bottom 32) then R2. Consuming 4:

      ```formula
      offered_quantity = 32
      offered_value    = 3 200.00 × 32 ÷ 40 = 2 560.00
      offered_value    = 2 560.00 × 4 ÷ 32 = 320.00
      movement_D2_value = 320.00                  ( 80.00 per unit )
      ```

    - Again no journal entry: neither location carries a valuation account.
    - Unit cost recomputed:

      ```formula
      quantity_on_hand = 42 − 4 = 38
      total_value      = 3 440.00 − 320.00 = 3 120.00
      unit_cost        = 3 120.00 ÷ 38 = 82.105263157…
      stored_unit_cost = 82.11
      ```

    - Line 1's delivered quantity becomes 12 and its invoiced quantity is 8, so its quantity to
      invoice becomes 4. The order's delivery status becomes `full`; its invoice status returns
      to `to invoice`.

11. **Create and post the second invoice.** Same two domains as steps 6 and 7. Invoice
    `INV/2026/00352`, invoice date 2026-03-23, one line: Desk lamp, 4 `Units` at 149.95 less
    5 %.

    ```formula
    line_untaxed = round_to_currency( 4 × 142.4525 ) = round_to_currency( 569.81 ) = 569.81
    line_tax     = round_to_currency( 569.81 × 21 ÷ 100 ) = round_to_currency( 119.6601 ) = 119.66
    grand_total  = 689.47
    instalment_1 = round_to_currency( 689.47 × 30 ÷ 100 ) = round_to_currency( 206.841 ) = 206.84
                   maturity = 2026-03-23
    instalment_2 = 689.47 − 206.84 = 482.63 , maturity = 2026-04-22
    ```

    The cost pair now has to avoid recognising the cost twice:

    ```formula
    posted_quantity = 8          ( from the cost item of INV/2026/00311 on account 1400 )
    cogs_quantity   = 8 + 4 = 12
    unit_price      = ( 640.00 + 320.00 ) ÷ ( 8 + 4 ) = 960.00 ÷ 12 = 80.00
    already_posted  = 640.00
    cogs_unit_price = | 80.00 × 12 − 640.00 | ÷ 4 = 320.00 ÷ 4 = 80.00
    amount_currency = 4 × 80.00 = 320.00
    ```

    The second invoice therefore recognises exactly the 320.00 that the first one did not.
    Line 1's invoiced quantity reaches 12, so its invoice status becomes `invoiced`; line 2 is
    already `invoiced`; the order's invoice status becomes `invoiced`.

12. **Settle the rest.** On 2026-04-09 the customer pays 1 219.36 + 689.47 = 1 908.83 in one
    transfer, clearing the second instalment of the first invoice and the whole second invoice
    ahead of its second maturity. One Payment is created for 1 908.83 against both documents;
    its entry carries a single counterpart credit of 1 908.83 on 1200 Trade Receivables, and the
    allocation across the three open payment-term items happens entirely in the reconciliation —
    three partial reconciliations, no extra journal item
    ([payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
    section 1.6). Both invoices reach a zero residual; because the counterpart is a payment that
    is not yet matched with a bank transaction, both take the in-payment value.

13. **Import and reconcile the second bank transaction.** On 2026-04-10 a Bank Transaction of
    +1 908.83 is created, its counterpart is moved onto 1450 Outstanding Receipts and matched
    with the Payment's liquidity item. The Payment becomes `paid` and both invoices become
    `paid`.

## 1.c The ledger

Every journal item the trace produces, in posting order. The counterparty on every item is
Baxter Interiors except where the column says otherwise; every amount is in euro, which is both
the document currency and the company currency, so the amount in currency equals the balance
and is not repeated.

| # | Date | Journal | Entry | Account | Debit | Credit | Maturity | Reconciled against |
|---|---|---|---|---|---|---|---|---|
| 1 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 4000 Product Sales | | 1 139.62 | | — |
| 2 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 4000 Product Sales | | 300.00 | | — |
| 3 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 2510 Tax Payable | | 302.32 | | — |
| 4 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 1200 Trade Receivables | 522.58 | | 2026-03-10 | item 9 |
| 5 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 1200 Trade Receivables | 1 219.36 | | 2026-04-09 | item 18 |
| 6 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 5000 Cost of Goods Sold | 640.00 | | | — (injected cost item) |
| 7 | 2026-03-10 | Customer Invoices | `INV/2026/00311` | 1400 Inventory | | 640.00 | | — (injected cost item) |
| 8 | 2026-03-10 | Bank | Payment `BNK1/2026/0044` | 1450 Outstanding Receipts | 522.58 | | | item 11 |
| 9 | 2026-03-10 | Bank | Payment `BNK1/2026/0044` | 1200 Trade Receivables | | 522.58 | | item 4 |
| 10 | 2026-03-12 | Bank | Transaction `BNK1/2026/00017` | 1010 Bank | 522.58 | | | — |
| 11 | 2026-03-12 | Bank | Transaction `BNK1/2026/00017` | 1450 Outstanding Receipts | | 522.58 | | item 8 |
| 12 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 4000 Product Sales | | 569.81 | | — |
| 13 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 2510 Tax Payable | | 119.66 | | — |
| 14 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 1200 Trade Receivables | 206.84 | | 2026-03-23 | item 18 |
| 15 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 1200 Trade Receivables | 482.63 | | 2026-04-22 | item 18 |
| 16 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 5000 Cost of Goods Sold | 320.00 | | | — (injected cost item) |
| 17 | 2026-03-23 | Customer Invoices | `INV/2026/00352` | 1400 Inventory | | 320.00 | | — (injected cost item) |
| 18 | 2026-04-09 | Bank | Payment `BNK1/2026/0061` | 1200 Trade Receivables | | 1 908.83 | | items 5, 14, 15 |
| 19 | 2026-04-09 | Bank | Payment `BNK1/2026/0061` | 1450 Outstanding Receipts | 1 908.83 | | | item 21 |
| 20 | 2026-04-10 | Bank | Transaction `BNK1/2026/00024` | 1010 Bank | 1 908.83 | | | — |
| 21 | 2026-04-10 | Bank | Transaction `BNK1/2026/00024` | 1450 Outstanding Receipts | | 1 908.83 | | item 19 |

The Payment entry emits its liquidity item before its counterpart item; the table shows item 8
before item 9 and item 19 before item 18 accordingly, and rows 18 and 19 belong to one entry.

**Totals.**

```formula
debits  = 522.58 + 1 219.36 + 640.00 + 522.58 + 522.58
        + 206.84 + 482.63 + 320.00 + 1 908.83 + 1 908.83 = 8 254.23
credits = 1 139.62 + 300.00 + 302.32 + 640.00 + 522.58 + 522.58
        + 569.81 + 119.66 + 320.00 + 1 908.83 + 1 908.83 = 8 254.23
```

**Per-account proof.**

| Account | Debits | Credits | Balance after the trace | Meaning |
|---|---|---|---|---|
| 1010 Bank | 2 431.41 | — | 2 431.41 debit | The cash actually received |
| 1200 Trade Receivables | 2 431.41 | 2 431.41 | 0.00 | Nothing owed |
| 1400 Inventory | — | 960.00 | 3 120.00 debit (4 080.00 opening − 960.00) | Equals the physical value, part (d) |
| 1450 Outstanding Receipts | 2 431.41 | 2 431.41 | 0.00 | Every payment confirmed by the bank |
| 2510 Tax Payable | — | 421.98 | 421.98 credit | The tax on 2 009.43 of net sales |
| 4000 Product Sales | — | 2 009.43 | 2 009.43 credit | The order's untaxed total, exactly |
| 5000 Cost of Goods Sold | 960.00 | — | 960.00 debit | The value of the 12 lamps that left |

Gross margin on the goods: 1 709.43 − 960.00 = 749.43, plus 300.00 of service revenue with no
cost.

## 1.d Stock consequences

**Quantities per location.**

| Location | Desk lamp before | After the first delivery | After the second |
|---|---|---|---|
| `WH/Stock` | 50 | 42 | 38 |
| `Partners/Customers` | 0 | −8 | −12 |

A customer location is never counted, so its quantity runs negative; that is the normal state
of an outgoing location and is specified in
[inventory-operations/workflows.md](inventory-operations/workflows.md) section 7.

**Value carried by each completed goods movement.** The platform stores the value on the
movement; the remaining quantity and the remaining value are derived from the first-in-first-out
stack, as specified in
[inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
section 5.3.

| Movement | Direction | Quantity | Value | Remaining quantity | Remaining value |
|---|---|---|---|---|---|
| R1 (2026-01-08) | incoming | 40 | 3 200.00 | 28 | 3 200.00 × 28 ÷ 40 = 2 240.00 |
| R2 (2026-02-03) | incoming | 10 | 880.00 | 10 | 880.00 |
| D1 (2026-03-09) | outgoing | 8 | 640.00 | 0 | 0.00 |
| D2 (2026-03-20) | outgoing | 4 | 320.00 | 0 | 0.00 |

```formula
physical_value = 2 240.00 + 880.00 = 3 120.00
ledger_value   = 4 080.00 − 640.00 − 320.00 = 3 120.00
difference     = 0.00
```

The two agree, which is the invariant the inventory valuation closing exists to check
([inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
section 6). Stored unit cost at the end: 3 120.00 ÷ 38 = 82.105263157… stored as 82.11.

## 1.e Variations

### 1.e.1 Invoicing policy *ordered quantities* on the Desk lamp

Line 1's quantity to invoice is 12 from the instant of confirmation, so the whole order can be
invoiced on 2026-03-05, before anything ships. One invoice replaces the two. Its untaxed total
is 1 709.43 + 300.00 = 2 009.43, its tax 421.98 and its total 2 431.41, with instalments
round(2 431.41 × 0.30) = 729.42 and 2 431.41 − 729.42 = 1 701.99.

The cost pair is still injected at posting, but no completed movement is reachable from the
line, so the cost falls back to the first-in-first-out value of the cost-of-goods-sold quantity:

```formula
cogs_quantity   = 12
fifo_value( 12 ) = 12 × 80.00 = 960.00        ( entirely from R1's 40 units at 80.00 )
unit_price      = 960.00 ÷ 12 = 80.00
cogs_unit_price = | 80.00 × 12 − 0.00 | ÷ 12 = 80.00
amount          = 960.00
```

**Delta against part (c).** Items 1 to 7 and 12 to 17 are replaced by a single entry dated
2026-03-05: credit 4000 Product Sales 1 709.43 and 300.00; credit 2510 Tax Payable 421.98; debit
1200 Trade Receivables 729.42 and 1 701.99; debit 5000 Cost of Goods Sold 960.00; credit 1400
Inventory 960.00. The total recognised cost is unchanged at 960.00, but it is recognised four
days before the goods leave, and the inventory account is credited while the goods are still on
the shelf. If the warehouse later delivered 13 rather than 12, line 1 would enter the
`upselling` status rather than becoming invoiceable again.

### 1.e.2 Costing method *average cost*

The opening unit cost is 4 080.00 ÷ 50 = 81.60. An outgoing movement under average cost is
valued at the stored unit cost, which an outgoing movement never changes:

```formula
movement_D1_value = 8 × 81.60 = 652.80
movement_D2_value = 4 × 81.60 = 326.40
total_cost_recognised = 979.20
```

**Delta.** Item 6 becomes a debit of 652.80 and item 7 a credit of 652.80; item 16 becomes a
debit of 326.40 and item 17 a credit of 326.40. Account 1400 Inventory ends at
4 080.00 − 979.20 = 3 100.80, and the physical value is 38 × 81.60 = 3 100.80 — still equal.
A credit note raised later against either invoice would reuse the unit price recorded on the
original document's inventory-side item rather than recompute the cost, because that shortcut
applies to the standard-price and average-cost methods.

### 1.e.3 Costing method *standard price* at 82.00

```formula
movement_D1_value = 8 × 82.00 = 656.00
movement_D2_value = 4 × 82.00 = 328.00
total_cost_recognised = 984.00
```

**Delta.** Items 6 and 7 become 656.00 and items 16 and 17 become 328.00. Account 1400 Inventory
ends at 4 080.00 − 984.00 = 3 096.00, while the physical value is 38 × 82.00 = 3 116.00. The
gap of 20.00 existed before the trace as well — the opening 50 units were bought for 4 080.00
but are valued at 50 × 82.00 = 4 100.00 — and it is the inventory valuation closing, not any
step of this trace, that posts it: a debit of 20.00 on 1400 Inventory against a credit of 20.00
on 1410 Inventory Variation.

### 1.e.4 Anglo-saxon against continental accounting

The company-level anglo-saxon switch governs **only** the price-difference items injected into a
vendor bill for a product under standard costing; that is the subject of trace 2's variations
and of [trace 11](#11-landed-costs-and-price-differences-on-standard-cost). It does **not** gate
the cost-of-goods-sold items of a customer invoice, nor the redirection of a vendor bill line to
the inventory valuation account. Both of those are governed by the product's **valuation mode**.

The continental treatment is therefore obtained by setting the category's valuation mode to
**periodic** rather than by turning the switch off. Under periodic valuation:

- The two opening receipts would have been expensed by their vendor bills, so 1400 Inventory
  would hold nothing at the start of the trace.
- The deliveries still post nothing.
- Items 6, 7, 16 and 17 disappear entirely: no cost pair is injected, because the condition
  "the product's valuation mode is perpetual" fails.
- The inventory valuation closing run on 2026-03-31 posts part two of its entry, whose balance
  is the physical value less the ledger value less what part one already proposed:

  ```formula
  balance = 3 120.00 − 0.00 − 0.00 = 3 120.00
  ```

  giving a debit of 3 120.00 on 1400 Inventory and a credit of 3 120.00 on 1410 Inventory
  Variation. The expense of the period is then the components of cost that ran through 1410
  rather than a per-invoice cost item.

### 1.e.5 A foreign-currency order

The same order written in United States dollars, with a rate of 1.0850 dollars for one euro on
2026-03-10. The commercial figures are unchanged in dollars; the balances are the dollar amounts
divided by the rate and rounded to the euro.

| Item | Account | Amount in currency (dollar) | Debit (euro) | Credit (euro) |
|---|---|---|---|---|
| 1 | 4000 Product Sales | −1 139.62 | | 1 050.34 |
| 2 | 4000 Product Sales | −300.00 | | 276.50 |
| 3 | 2510 Tax Payable | −302.32 | | 278.64 |
| 4 | 1200 Trade Receivables | +522.58 | 481.64 | |
| 5 | 1200 Trade Receivables | +1 219.36 | 1 123.84 | |

```formula
round_to_currency( 1 139.62 ÷ 1.0850 ) = round_to_currency( 1 050.341014 ) = 1 050.34
round_to_currency(   300.00 ÷ 1.0850 ) = round_to_currency(   276.497696 ) = 276.50
round_to_currency(   302.32 ÷ 1.0850 ) = round_to_currency(   278.635945 ) = 278.64
company_total  = 1 050.34 + 276.50 + 278.64 = 1 605.48
effective_rate = 1 741.94 ÷ 1 605.48 = 1.084996…
instalment_1_company  = round_to_currency( 1 605.48 × 30 ÷ 100 ) = round_to_currency( 481.644 ) = 481.64
instalment_1_currency = round_to_currency( 1 741.94 × 30 ÷ 100 ) = 522.58
instalment_2_company  = 1 605.48 − 481.64 = 1 123.84
instalment_2_currency = 1 741.94 − 522.58 = 1 219.36
```

Both columns balance: euro debits 481.64 + 1 123.84 = 1 605.48 against euro credits
1 050.34 + 276.50 + 278.64 = 1 605.48; dollar amounts −1 139.62 − 300.00 − 302.32 + 522.58 +
1 219.36 = 0.00. The distribution uses the **effective** rate, not the stored 1.0850, which is
what makes both columns close.

**Settlement at a different rate.** The second instalment of 1 219.36 dollars is paid on
2026-04-09 at a rate of 1.1000. The payment's counterpart carries 1 219.36 dollars and a balance
of round(1 219.36 ÷ 1.1000) = round(1 108.509091) = 1 108.51 euro. Reconciling it with item 5
brings the dollar residual to zero but leaves a euro residual of
1 123.84 − 1 108.51 = 15.33 as a debit. An exchange-difference entry is produced in the Exchange
Difference journal, dated on the later of the two matched items' dates:

| Account | Amount in currency (dollar) | Debit (euro) | Credit (euro) |
|---|---|---|---|
| 1200 Trade Receivables | 0.00 | | 15.33 |
| 6560 Foreign Exchange Loss | 0.00 | 15.33 | |

The receivable item of the exchange entry is reconciled with item 5, which brings the euro
residual to zero as well. Note the **zero** amount in currency on both items: an exchange
difference corrects only the company-currency side.

**Compatibility finding — the cost pair in a foreign-currency document.** The injected cost
items are written with an amount *in currency* equal to quantity × the cost-of-goods-sold unit
price, and that unit price is a **company-currency** cost. The company-currency balance is then
derived from the amount in currency at the document's rate, so on this dollar invoice the
inventory account is credited round(640.00 ÷ 1.0850) = 589.86 euro rather than the 640.00 euro
of value that actually left stock, and the inventory account no longer matches the physical
value. This is recorded as observed. A corrected behaviour would write the cost items' balance
directly as the company-currency cost and set the amount in currency to that cost multiplied by
the document rate, leaving the ledger value equal to the physical value in every currency.

### 1.e.6 Cash rounding to the nearest five hundredths

The company configures a cash rounding method *Nearest 0.05*, half away from zero, with the
add-a-rounding-line strategy, a profit account 7580 Cash Rounding Gain and a loss account 6580
Cash Rounding Loss, and it is applied to `INV/2026/00311`.

```formula
rounded_total = round_to_step( 1 741.94 , 0.05 , half away from zero ) = 1 741.95
difference    = 1 741.95 − 1 741.94 = 0.01
```

The rounding item raises the untaxed amount, which on a customer invoice is a credit, so its
balance is negative rather than strictly positive and the **profit** account is used. It carries
no tax and no tax grid, so the tax return is untouched. The payment term then distributes the
new total, and the cash-rounding correction is applied to every instalment except the last:

```formula
instalment_1 = round_to_currency( 1 741.95 × 30 ÷ 100 ) = round_to_currency( 522.585 ) = 522.59
correction   = round_to_step( 522.59 , 0.05 , half away from zero ) − 522.59 = 522.60 − 522.59 = 0.01
instalment_1 = 522.60
instalment_2 = 1 741.95 − 522.60 = 1 219.35
```

**Delta against part (c).** One new item: credit 0.01 on 7580 Cash Rounding Gain. Item 4 becomes
522.60 and item 5 becomes 1 219.35. Totals: credits 1 139.62 + 300.00 + 302.32 + 0.01 =
1 741.95 against debits 522.60 + 1 219.35 = 1 741.95. The cost items are untouched.

### 1.e.7 An early payment discount of two per cent within ten days

An early payment discount requires a **single-line** payment term, so the term becomes one line
of 100 % at 30 days carrying a discount of 2 % within 10 days, in the computation mode *On early
payment*. `INV/2026/00311` then carries one receivable item of 1 741.94 maturing 2026-04-09,
stamped with a discount deadline of 2026-03-20 and a discounted amount of

```formula
discount_amount = 1 741.94 × 2 ÷ 100 = 34.8388
amount_due      = round_to_currency( 1 741.94 − 34.8388 ) = round_to_currency( 1 707.1012 ) = 1 707.10
```

The invoice is issued at its full value; nothing extra appears on it. The customer pays 1 707.10
on 2026-03-15, inside the deadline, and the register-payment assistant adds the write-off items
to the **payment's own** entry so that the receivable clears exactly:

| Account | Label | Debit | Credit |
|---|---|---|---|
| 1450 Outstanding Receipts | `INV/2026/00311` | 1 707.10 | |
| 6900 Cash Discount Granted | `Early Payment Discount` | 28.79 | |
| 2510 Tax Payable | `Early Payment Discount (Sales 21 %)` | 6.05 | |
| 1200 Trade Receivables | `INV/2026/00311` | | 1 741.94 |

```formula
net_discount = round_to_currency( 1 439.62 × 2 ÷ 100 ) = round_to_currency( 28.7924 ) = 28.79
tax_on_it    = round_to_currency( 28.79 × 21 ÷ 100 ) = round_to_currency( 6.0459 ) = 6.05
28.79 + 6.05 = 34.84 = 1 741.94 − 1 707.10
```

The write-off base item carries the base grids of the 21 % tax with the refund sign and the
write-off tax item carries that tax's grids with the refund sign, so the tax return reports a
base reduction of 28.79 and a tax reduction of 6.05. In the *Never* mode only the net 28.79
would be written off and no tax item would be produced; in the *Always (upon invoice)* mode the
invoice itself would carry a cancelling pair of items on 4000 Product Sales, one with the tax
and one without, and the tax would be computed on 1 439.62 − 28.79 = 1 410.83 from the start.

### 1.e.8 Several companies

Selling from one company of the group to another turns this trace into the inter-company case,
where the customer invoice of the seller is mirrored by a vendor bill of the buyer. That is
traced in
[trace 9](#9-drop-shipping-and-inter-company-trade).

## 1.f Failure points

Each entry gives the step, the condition, the domain that refuses and the exact text.

**Step 3, confirming an order whose status is not `draft` or `sent`** (sales). The whole
operation fails and no selected order is touched.

`Some orders are not in a state requiring confirmation.`

**Step 3, confirming an order with a line that has no product** (sales). Display lines and
advance-invoice lines are exempt.

`Some order lines are missing a product, you need to correct them before going further.`

**Step 3, confirming when a mandatory analytic plan is unsatisfied** (sales, delegating to
analytic accounting). The confirmation aborts with the analytic domain's message; the rule is
specified in [sales/business-rules.md](sales/business-rules.md).

**Step 3, procurement finds no rule** (replenishment and procurement). Raised when the product
carries no route that reaches `Partners/Customers` from the warehouse. The message is two lines.

`No rule has been found to replenish "<the product display name>" in "<the location display name>".`

`Verify the routes configuration on the product.`

**Step 5, validating a transfer with no moves and no detail lines** (inventory operations).

`You can’t validate an empty transfer. Please add some products to move before proceeding.`

**Step 5, validating a transfer where every move to process has a zero processed quantity**
(inventory operations). The line break in the middle is part of the message.

`Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!`

**Step 5, validating a transfer of a tracked product without a lot or serial number** (inventory
operations), when the operation type allows creating or using lots.

`You need to supply a Lot/Serial number for products <the comma-separated product display names>.`

**Step 6, invoicing when nothing is invoiceable** (sales). Raised when the run produced no
header values at all — typically because the product is invoiced on delivered quantities and
nothing has shipped. The message is multi-line and is reproduced whole:

```
Cannot create an invoice. No items are available to invoice.

To resolve this issue, please ensure that:
   • The products have been delivered before attempting to invoice them.
   • The invoicing policy of the product is configured correctly.

If you want to invoice based on ordered quantities instead:
   • For consumable or storable products, open the product, go to the 'General Information' tab and change the 'Invoicing Policy' from 'Delivered Quantities' to 'Ordered Quantities'.
   • For services (and other products), change the 'Invoicing Policy' to 'Prepaid/Fixed Price'.
```

**Step 7, posting without a customer** (accounts receivable). Two lines.

`The 'Customer' field is required to validate the invoice.\nYou probably don't want to explain to your auditor that you invoiced an invisible man :)`

**Step 7, posting a document whose total is negative** (accounts receivable).

`You cannot validate an invoice with a negative total amount. You should create a credit note instead. Use the action menu to transform it into a credit note or refund.`

**Step 7, posting a document with no accountable line** (accounts receivable). Sections,
subsections and notes do not count.

`Even magicians can't post nothing!`

**Step 7, posting a document that is not draft** (accounts receivable).

`The entry <the document number> (id <the internal identifier>) must be in draft.`

**Step 7, posting with an archived account, journal or currency** (accounts receivable), one
message per case.

`You cannot use an archived account.`

`You cannot post an entry in an archived journal (<the journal name>)`

`You cannot validate a document with an inactive currency: <the currency name>`

**Step 7, posting when the document does not balance** (accounts receivable). This cannot arise
from the trace above, because the payment term's balance rule guarantees it, but it is the guard
that protects the injected cost items and any manual edit.

`The entry is not balanced.`

**Step 8, registering a payment when the payment method line has no outstanding account**
(payments and bank reconciliation).

`You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal.`

**Step 9, creating a bank transaction in a journal with no suspense account** (payments and bank
reconciliation).

`You can't create a new statement line without a suspense account set on the <the journal display name> journal.`

**Step 11, a missing account on the cost pair.** No message is raised. When the inventory
valuation account or the expense counterpart cannot be resolved for a line, that line is simply
**skipped** and the invoice posts with revenue, tax and a receivable but no cost recognition.
The consequence is silent: the inventory account keeps a value the goods no longer have. A
rebuild should reproduce the skip, because it is what the platform does, and should surface the
condition in the inventory valuation closing report, where the difference appears.
