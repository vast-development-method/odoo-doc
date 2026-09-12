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

---

# 2. Procure to pay with received goods

A reordering rule discovers a shortage, the buy rule raises a request for quotation, the buyer
confirms it, the warehouse receives part of the order and then the rest, each part is billed at
a price higher than the one ordered, and the vendor is paid in two transfers. The trace crosses
[replenishment and procurement](replenishment-and-procurement/), [purchasing](purchasing/),
[inventory operations](inventory-operations/),
[inventory valuation and costing](inventory-valuation-and-costing/), [taxes](taxes/),
[accounts payable](accounts-payable/) and
[payments and bank reconciliation](payments-and-bank-reconciliation/).

Its point is the opposite of trace 1's. There, the inventory asset was **credited** by the
customer invoice. Here it is **debited** by the vendor bill, and the goods movement that
physically brought the goods in posts nothing at all. The trace also shows what happens when the
vendor charges more than the order said: under first in first out the difference is absorbed
into the value of the goods by a silent re-valuation, with no journal item of its own.

## 2.a Starting records

Everything of the [shared fixture](#the-shared-fixture) applies. In addition:

**Product — Filing cabinet.** A goods product, storable. Reference unit of measure `Units`.
Costing method **first in first out**; valuation mode **perpetual**. Purchase tax: Purchase
21 %. Bill control policy **on received quantities**, whose stored value is `receive`. Product
category *Storage*, which carries the income account 4000 Product Sales, the expense account
5000 Cost of Goods Sold, the inventory valuation account 1400 Inventory, the price difference
account 5100 Price Difference and the inventory journal Inventory Valuation. Quantity on hand
at `WH/Stock`: **zero**; stored unit cost 0.00; no completed goods movement of any kind. The
product carries the warehouse's reception route and no other.

**Product — Freight in.** A service product, therefore never storable and never valued. Bill
control policy **on ordered quantities**, whose stored value is `purchase`; the computation
forces that value for every service, so the configured default is irrelevant. Purchase tax:
Purchase 21 %. Expense account declared on the product itself: 5000 Cost of Goods Sold.

**Vendor.** Northgate Supplies, its own commercial entity. Payable account 2100 Trade Payables.
Supplier location: the shared `Partners/Vendors` location. Currency of its vendor pricelist
entry for the Filing cabinet: euro, minimum quantity 1, price 145.00, lead time 4 days. No
fiscal position.

**Payment term — "Half on receipt, half at end of next month".** Two lines, in this stored
order: (1) a percentage line of 50 %, delay type *days after invoice date*, 0 days; (2) a
balance line, delay type *days after the end of the next month*, 0 days. No early payment
discount.

**Reordering rule.** Product Filing cabinet, location `WH/Stock`, minimum quantity 4, maximum
quantity 30, replenishment multiple 1, trigger automatic, route: the warehouse's reception
route. Company replenishment horizon: zero days.

**Reception.** The Main Warehouse receives in **one step**: the reception route's pull rule runs
from `Partners/Vendors` to `WH/Stock` with the operation type *Receipts*, whose sequence
produces references of the form `WH/IN/…`. The two-step arrangement, which the fixture's
`WH/Input` location supports, is [variation 2.e.9](#2e9-receiving-in-two-steps).

**Dates.** Scheduler run and order creation 2026-06-01. Confirmation 2026-06-01. First receipt
2026-06-05. First bill 2026-06-11. First payment 2026-06-11, on the bank statement 2026-06-15.
Second receipt 2026-06-19. Second bill 2026-06-25. Final payment 2026-07-31, on the bank
statement 2026-08-03.

**Vendor reference.** The vendor's own document number on both bills is `NGS-4471` for the
first and `NGS-4520` for the second. These are reproduced strings: they are typed by the
accountant and printed back on the payment.

## 2.b Steps

1. **The reordering rule fires.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   sections 11, 12 and 12.1. The scheduled pass evaluates every automatic rule. For this one:
   the rule chain from `WH/Stock` finds the reception route's buy rule, so the lead days are the
   vendor's 4 days plus the rule's own delay; the forecast at the lead horizon date is 0, which
   is below the minimum of 4, so a quantity is ordered.

   ```formula
   quantity_forecast = 0
   compare( 0 , 4 , Units ) = −1                     ( a replenishment is needed )
   raw_quantity      = max( 4 , 30 ) − 0 = 30
   multiple          = 1 , so no rounding up applies
   quantity_to_order = 30
   ```

   One procurement request is built: product Filing cabinet, 30 `Units`, location `WH/Stock`,
   route the reception route, reordering rule this rule, company Northwind Trading.

2. **The rule search resolves to a buy rule.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   sections 1 and 3. The location chain from `WH/Stock` upwards is walked; the candidate route
   set contains the route named on the request, and the warehouse route filter keeps the
   reception route because the product has a vendor pricelist entry. The rule found has the
   action *buy*, so the request is filed under the buy action.

3. **The buy action creates the request for quotation.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   sections 5.1 to 5.4, and purchasing,
   [purchasing/workflows.md](purchasing/workflows.md) section 13. The vendor is selected from
   the product's vendor pricelist entries; no open request for quotation exists for that vendor,
   company, currency and order deadline, so one is created: reference `P00058`, vendor Northgate
   Supplies, currency euro, order deadline 2026-06-01, operation type *Receipts*, payment term
   "Half on receipt, half at end of next month", status `draft`. One line is created: Filing
   cabinet, 30 `Units`, unit price 145.00 from the vendor pricelist entry, tax Purchase 21 %,
   expected arrival 2026-06-05.

4. **The buyer adds the freight line and confirms.** Domain: purchasing,
   [purchasing/workflows.md](purchasing/workflows.md) sections 5, 6 and 7; states in
   [purchasing/state-machines.md](purchasing/state-machines.md) section 1.
   - A second line is typed by hand: Freight in, 1 `Units`, unit price 180.00, tax Purchase
     21 %, expected arrival 2026-06-01.
   - Order arithmetic, rounded as specified in
     [purchasing/calculations.md](purchasing/calculations.md) sections 1.2 and 1.3 and
     [taxes/calculations.md](taxes/calculations.md):

     ```formula
     line_1_untaxed = round_to_currency( 30 × 145.00 ) = 4 350.00
     line_1_tax     = round_to_currency( 4 350.00 × 21 ÷ 100 ) = round_to_currency( 913.50 ) = 913.50
     line_2_untaxed = round_to_currency( 1 × 180.00 ) = 180.00
     line_2_tax     = round_to_currency( 180.00 × 21 ÷ 100 ) = round_to_currency( 37.80 ) = 37.80
     untaxed_total  = 4 350.00 + 180.00 = 4 530.00
     tax_total      = 913.50 + 37.80 = 951.30
     grand_total    = 5 481.30
     ```

   - The confirmation check passes: both lines carry a product.
   - Vendor price learning runs before the status decision. Northgate Supplies is already a
     seller of the Filing cabinet, so nothing is created for line 1. It is **not** a seller of
     Freight in, so a new vendor pricelist entry is written on that product's template: partner
     Northgate Supplies, sequence 1, minimum quantity 1.0, price 180.00, currency euro, discount
     0, lead time 0.
   - The company's approval policy is one step, so the approval test succeeds. Status `draft` →
     `purchase`, confirmation date 2026-06-01.
   - No journal item is produced. A purchase order is a commitment; the reasoning is in
     [purchasing/accounting-effects.md](purchasing/accounting-effects.md) section 0.

5. **The receipt is created.** Domain: purchasing,
   [purchasing/workflows.md](purchasing/workflows.md) sections 8.1 to 8.3, with the unit price
   of [purchasing/calculations.md](purchasing/calculations.md) section 8.
   - Only line 1 has a consumable product, so exactly one goods movement is emitted.
   - The **movement unit price**, which is what the valuation engine will use if no bill ever
     arrives, is computed from the discounted unit price with non-deductible taxes capitalised.
     Purchase 21 % is fully deductible — its repartition line names account 1310 Tax Receivable
     — so the tax-free total equals the base:

     ```formula
     discounted_unit_price   = 145.00 × ( 1 − 0 ÷ 100 ) = 145.00
     tax_free_total          = 30 × 145.00 = 4 350.00
     price_per_ordered_unit  = 4 350.00 ÷ 30 = 145.00
     unit_factor_correction  = 1 ÷ 1 = 1              ( the line unit is the reference unit )
     move_price_unit         = round_half_up( 145.00 , 0.01 ) = 145.00
     ```

   - A Transfer `WH/IN/00071` is created with source `Partners/Vendors`, destination `WH/Stock`,
     partner Northgate Supplies, source document `P00058`, status draft; the movement of 30
     `Units` is created, confirmed and reserved. Because the source location bypasses
     reservation, the movement is immediately assigned and the Transfer becomes ready.
   - Derived statuses, by [purchasing/state-machines.md](purchasing/state-machines.md) sections
     4 and 5: line 1 is controlled on received quantities and nothing has arrived, so its
     quantity to bill is 0; line 2 is controlled on ordered quantities, so its quantity to bill
     is 1. At least one line is billable, so the order's billing status is *Waiting Bills*. The
     receipt status is *Not Received*.

6. **Validate the receipt for 20 of 30 units, with a backorder.** Domain: inventory operations,
   [inventory-operations/workflows.md](inventory-operations/workflows.md) sections 4, 13 and
   14.1. The operator types 20 and validates on 2026-06-05.
   - The sanity check passes; the backorder policy is *ask* and 20 is below 30, so the backorder
     screen opens and the operator accepts. The short movement is split: the original keeps a
     demand of 20, the new movement of 10 goes into `WH/IN/00071-001`, which is confirmed and
     reserved.
   - **Valuation of the completed movement.** Domain: inventory valuation and costing,
     [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
     sections 1.2 and 2.2. The movement is incoming: its source `Partners/Vendors` is outside
     the valued perimeter and its destination `WH/Stock` is inside it. Its value is the priority
     chain:

     | Source | Contribution | Quantity claimed |
     |---|---|---|
     | 0, a manual correction | none | 0 |
     | 1, vendor bills | none — no bill of this purchase order line is posted yet | 0 |
     | 2, production | not applicable — the movement belongs to no manufacturing order | 0 |
     | 3, the purchase order line | 145.00 × 20 = 2 900.00 | 20 |
     | 4, returns | not reached | — |
     | 5, the product cost | not reached | — |
     | 6, landed costs | none | — |

     ```formula
     movement_R_A_value = 145.00 × 20 = 2 900.00       ( 145.00 per unit )
     ```

   - **No journal entry is produced.** Neither `Partners/Vendors` nor `WH/Stock` carries a
     location valuation account, so the condition of
     [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
     section 1 fails and the batch contributes nothing. The inventory asset will be debited by
     the bill.
   - **The unit cost is recomputed** by the incremental fast path of
     [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
     section 3.2, because no outgoing movement was validated in the same call:

     ```formula
     added_value       = 2 900.00
     added_quantity    = 20
     quantity_on_hand  = 20
     previous_quantity = 20 − 20 = 0                   ( not greater than zero )
     unit_cost         = 2 900.00 ÷ 20 = 145.00
     ```

   - Line 1's received quantity becomes 20, so its quantity to bill becomes 20 − 0 = 20. The
     order's receipt status becomes *Partially Received*; the billing status stays *Waiting
     Bills*.

7. **Create the first bill.** Domain: purchasing,
   [purchasing/workflows.md](purchasing/workflows.md) sections 12.1 and 12.2, with the line
   mapping of [purchasing/calculations.md](purchasing/calculations.md) section 9. The buyer
   presses the bill-creation control. The prepared header carries the vendor, the euro, the
   order's payment term, the source document `P00058` and the company. The line walk emits line
   1 with quantity 20 and line 2 with quantity 1, both at the order's unit prices. One draft
   vendor bill is created in the Vendor Bills journal.

   The accountant then types the vendor's own document: the vendor charged **149.00** per
   cabinet, not the 145.00 that was ordered, and the accountant rewrites the unit price of the
   cabinet line to 149.00. The bill date and the accounting date are both 2026-06-11 and the
   vendor reference is `NGS-4471`.

8. **Post the first bill.** Domain: accounts payable,
   [accounts-payable/accounting-effects.md](accounts-payable/accounting-effects.md) section 2,
   with the account selection of
   [accounts-payable/calculations.md](accounts-payable/calculations.md) sections 6.1 and 6.2 and
   the payment term distribution of section 5.
   - Line amounts:

     ```formula
     line_A_untaxed = round_to_currency( 20 × 149.00 ) = 2 980.00
     line_A_tax     = round_to_currency( 2 980.00 × 21 ÷ 100 ) = round_to_currency( 625.80 ) = 625.80
     line_B_untaxed = round_to_currency( 1 × 180.00 ) = 180.00
     line_B_tax     = round_to_currency( 180.00 × 21 ÷ 100 ) = 37.80
     untaxed_total  = 2 980.00 + 180.00 = 3 160.00
     tax_total      = 625.80 + 37.80 = 663.60
     grand_total    = 3 823.60
     ```

     Both lines carry the same tax, whose single repartition line names account 1310 Tax
     Receivable, so the accounting grouping key is identical and the two tax amounts aggregate
     into **one** tax journal item of 663.60.
   - **Account selection.** Line B is a service, so it keeps the expense account declared on the
     product, 5000 Cost of Goods Sold. Line A is a storable product whose valuation mode is
     perpetual, and an inventory valuation account resolves for it, so the line's account is
     **replaced** by that account, 1400 Inventory
     ([inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
     section 3). This is the redirection that makes the balance sheet, and not the profit and
     loss account, carry the goods.
   - **No price difference is injected.** The mechanism of
     [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
     section 4 requires the product's costing method to be `standard`, the stored value of the
     standard-price method. The Filing cabinet uses `fifo`, the stored value of the
     first-in-first-out method, so the 4.00 per unit that the vendor charged above the order
     price is absorbed into the value of the goods instead. Step 9 shows how.
   - **Payment term distribution**, on the accounting-signed totals, with a rate of 1 because
     the document currency is the company currency
     ([accounts-payable/calculations.md](accounts-payable/calculations.md) section 5.2):

     ```formula
     instalment_1 = round_to_currency( 3 823.60 × 50 ÷ 100 ) = round_to_currency( 1 911.80 ) = 1 911.80
                    maturity = 2026-06-11 + 0 days = 2026-06-11
     instalment_2 = 3 823.60 − 1 911.80 = 1 911.80        ( the last line is always the balance )
                    maturity = last day of the month of ( 2026-06-11 + 1 month ) = 2026-07-31
     ```

     The document has a vendor reference and no payment reference, so each term line is labelled
     with that reference; the term has more than one line, so the instalment suffix is appended,
     giving `NGS-4471 installment #1` and `NGS-4471 installment #2`. Both strings are reproduced
     as the system builds them, including the shipped spelling of the word.
   - The document is numbered `BILL/2026/00087` from the Vendor Bills journal sequence and its
     status becomes `posted`. Its payment status is `not_paid`.
   - Line 1's billed quantity becomes 20 and line 2's becomes 1, so both quantities to bill
     become 0 and the order's billing status becomes *Fully Billed*, while its receipt status is
     still *Partially Received*.

9. **The bill re-values the goods already received.** Domain: inventory valuation and costing,
   [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
   sections 1.3, 2.4 and 3.2. Posting a vendor bill triggers a **full revaluation of every
   incoming movement reachable from the bill's lines**. For movement R‑A the priority chain now
   stops at source 1:

   ```formula
   billed_quantity   = 20                       ( converted into the reference unit )
   billed_value      = round_to_currency( 2 980.00 ÷ 1 ) = 2 980.00
   absorbed_quantity = 0                        ( no earlier movement of this order line )
   available_value    = 2 980.00 × ( 20 − 0 ) ÷ 20 = 2 980.00
   available_quantity = 20 − 0 = 20
   remaining_quantity = 20 , which is not below the available quantity
   contributed_value  = 2 980.00
   movement_R_A_value = 2 980.00                ( 149.00 per unit, was 145.00 )
   ```

   The justification recorded on the movement reads *"the formatted value for the available
   quantity the reference unit name from the list of bill display names"*.

   **No journal entry results from the re-valuation.** The movement still crosses no location
   that carries a valuation account. The ledger has already been told the right number by the
   bill itself: 1400 Inventory was debited 2 980.00, and the movement now carries 2 980.00. The
   two agree exactly, which is the whole design.

   The unit cost is recomputed by the first-in-first-out full path, because this call supplies no
   extra maps:

   ```formula
   quantity_on_hand = 20
   total_value      = 2 980.00
   unit_cost        = 2 980.00 ÷ 20 = 149.00
   ```

10. **Pay the first instalment.** Domain: payments and bank reconciliation,
    [payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
    sections 1.3 and 1.5. A Payment is created from the bill on 2026-06-11 with amount 1 911.80,
    direction outbound, counterparty kind vendor, journal Bank, payment method *Manual Payment*
    whose payment account is 1455 Outstanding Payments. Its entry emits the liquidity item first:
    a **credit** of 1 911.80 on 1455, then a **debit** of 1 911.80 on 2100 Trade Payables.
    Confirming it posts the entry; the counterpart item is reconciled with the bill's first
    payment-term item. The bill's residual becomes 1 911.80, so its payment status becomes
    `partial`.

11. **Import and reconcile the first bank transaction.** Domain: payments and bank
    reconciliation,
    [payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
    sections 3.1 and 3.4. A Bank Transaction of −1 911.80 dated 2026-06-15 is created in the Bank
    journal. Its entry is created and posted at once: a liquidity **credit** of 1 911.80 on 1010
    Bank and a counterpart **debit** of 1 911.80 on the journal's suspense account 1099. Matching
    it against the outstanding Payment rewrites the counterpart item onto 1455 Outstanding
    Payments and reconciles it with the Payment's liquidity item. The outstanding account returns
    to zero for this payment; the Payment moves from `in_process` to `paid`.

12. **Validate the backorder.** Domain: inventory operations, then inventory valuation and
    costing. On 2026-06-19 the remaining 10 `Units` arrive on `WH/IN/00071-001`.
    - Valuation, priority chain again. Source 1 is consulted first and now finds a posted bill,
      but that bill has already been consumed:

      ```formula
      billed_quantity   = 20
      billed_value      = 2 980.00
      absorbed_quantity = 20                     ( movement R-A is earlier and incoming )
      billed_quantity is not greater than absorbed_quantity  →  contribute nothing
      ```

      Source 3 therefore answers, from the **order** line, which still says 145.00 because only
      the bill line was rewritten:

      ```formula
      movement_R_B_value = 145.00 × 10 = 1 450.00        ( 145.00 per unit )
      ```

    - No journal entry, for the same reason as before.
    - Unit cost, incremental fast path:

      ```formula
      added_value       = 1 450.00
      added_quantity    = 10
      quantity_on_hand  = 30
      previous_quantity = 30 − 10 = 20                   ( greater than zero )
      unit_cost         = ( 20 × 149.00 + 1 450.00 ) ÷ 30 = ( 2 980.00 + 1 450.00 ) ÷ 30
                        = 4 430.00 ÷ 30 = 147.666666…
      stored_unit_cost  = 147.67                          ( two decimals, the Product Price precision )
      ```

    - Line 1's received quantity becomes 30, so its quantity to bill becomes 30 − 20 = 10. The
      receipt status becomes *Fully Received* and the billing status returns to *Waiting Bills*.

13. **Create and post the second bill.** Same two domains as steps 7 and 8. The line walk emits
    line 1 with quantity 10 and line 2 with quantity **0**, because its ordered quantity has
    already been billed in full; the walk emits every non-display line regardless of quantity.
    The accountant deletes the zero-quantity freight line and rewrites the cabinet price to
    149.00 again. Bill `BILL/2026/00104`, bill date and accounting date 2026-06-25, vendor
    reference `NGS-4520`.

    ```formula
    line_untaxed  = round_to_currency( 10 × 149.00 ) = 1 490.00
    line_tax      = round_to_currency( 1 490.00 × 21 ÷ 100 ) = round_to_currency( 312.90 ) = 312.90
    grand_total   = 1 802.90
    instalment_1  = round_to_currency( 1 802.90 × 50 ÷ 100 ) = round_to_currency( 901.45 ) = 901.45
                    maturity = 2026-06-25
    instalment_2  = 1 802.90 − 901.45 = 901.45
                    maturity = last day of the month of ( 2026-06-25 + 1 month ) = 2026-07-31
    ```

    Posting re-values **both** movements of the purchase order line, because both are reachable
    from the bill's line:

    ```formula
    billed_quantity = 20 + 10 = 30
    billed_value    = 2 980.00 + 1 490.00 = 4 470.00

    for movement R-B :
        absorbed_quantity  = 20                    ( R-A is earlier and incoming )
        available_value    = 4 470.00 × ( 30 − 20 ) ÷ 30 = 4 470.00 × 10 ÷ 30 = 1 490.00
        available_quantity = 10
        remaining_quantity = 10 , not below the available quantity
        movement_R_B_value = 1 490.00              ( 149.00 per unit, was 145.00 )

    for movement R-A :
        absorbed_quantity  = 0                     ( nothing is earlier than R-A )
        available_value    = 4 470.00 × 30 ÷ 30 = 4 470.00
        available_quantity = 30
        remaining_quantity = 20 , below the available quantity
        contributed_value  = 20 × 4 470.00 ÷ 30 = 2 980.00
        movement_R_A_value = 2 980.00              ( unchanged )
    ```

    The algorithm is stable: re-running it on an already-correct movement returns the same
    figure. The unit cost becomes 4 470.00 ÷ 30 = **149.00**.

    Line 1's billed quantity reaches 30, so its quantity to bill becomes 0 and the order's
    billing status becomes *Fully Billed* again.

14. **Settle the rest.** On 2026-07-31 one outbound Payment of
    1 911.80 + 901.45 + 901.45 = 3 714.70 is created against both bills. Its entry carries a
    single counterpart debit of 3 714.70 on 2100 Trade Payables, and the allocation across the
    three open payment-term items happens entirely in the reconciliation — three partial
    reconciliations, no extra journal item
    ([payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
    section 1.6). Both bills reach a zero residual and take the in-payment value, because the
    Payment is not yet matched with a bank transaction.

15. **Import and reconcile the second bank transaction.** On 2026-08-03 a Bank Transaction of
    −3 714.70 is created, its counterpart is moved onto 1455 Outstanding Payments and matched
    with the Payment's liquidity item. The Payment becomes `paid` and both bills become `paid`.

## 2.c The ledger

Every journal item the trace produces, in posting order. The counterparty on every accountable
item of a bill or a payment is Northgate Supplies; the two bank-transaction entries carry the
same counterparty once they are matched. Every amount is in euro, which is both the document
currency and the company currency, so the amount in currency equals the balance and is not
repeated. Each Payment entry emits its **liquidity** item before its counterpart item, and the
table follows that order.

| # | Date | Journal | Entry | Account | Debit | Credit | Maturity | Reconciled against |
|---|---|---|---|---|---|---|---|---|
| 1 | 2026-06-11 | Vendor Bills | `BILL/2026/00087` | 1400 Inventory | 2 980.00 | | | — |
| 2 | 2026-06-11 | Vendor Bills | `BILL/2026/00087` | 5000 Cost of Goods Sold | 180.00 | | | — |
| 3 | 2026-06-11 | Vendor Bills | `BILL/2026/00087` | 1310 Tax Receivable | 663.60 | | | — |
| 4 | 2026-06-11 | Vendor Bills | `BILL/2026/00087` | 2100 Trade Payables | | 1 911.80 | 2026-06-11 | item 7 |
| 5 | 2026-06-11 | Vendor Bills | `BILL/2026/00087` | 2100 Trade Payables | | 1 911.80 | 2026-07-31 | item 15 |
| 6 | 2026-06-11 | Bank | Payment `BNK1/2026/0102` | 1455 Outstanding Payments | | 1 911.80 | | item 9 |
| 7 | 2026-06-11 | Bank | Payment `BNK1/2026/0102` | 2100 Trade Payables | 1 911.80 | | | item 4 |
| 8 | 2026-06-15 | Bank | Transaction `BNK1/2026/00051` | 1010 Bank | | 1 911.80 | | — |
| 9 | 2026-06-15 | Bank | Transaction `BNK1/2026/00051` | 1455 Outstanding Payments | 1 911.80 | | | item 6 |
| 10 | 2026-06-25 | Vendor Bills | `BILL/2026/00104` | 1400 Inventory | 1 490.00 | | | — |
| 11 | 2026-06-25 | Vendor Bills | `BILL/2026/00104` | 1310 Tax Receivable | 312.90 | | | — |
| 12 | 2026-06-25 | Vendor Bills | `BILL/2026/00104` | 2100 Trade Payables | | 901.45 | 2026-06-25 | item 15 |
| 13 | 2026-06-25 | Vendor Bills | `BILL/2026/00104` | 2100 Trade Payables | | 901.45 | 2026-07-31 | item 15 |
| 14 | 2026-07-31 | Bank | Payment `BNK1/2026/0139` | 1455 Outstanding Payments | | 3 714.70 | | item 17 |
| 15 | 2026-07-31 | Bank | Payment `BNK1/2026/0139` | 2100 Trade Payables | 3 714.70 | | | items 5, 12, 13 |
| 16 | 2026-08-03 | Bank | Transaction `BNK1/2026/00068` | 1010 Bank | | 3 714.70 | | — |
| 17 | 2026-08-03 | Bank | Transaction `BNK1/2026/00068` | 1455 Outstanding Payments | 3 714.70 | | | item 14 |

**Totals.**

```formula
debits  = 2 980.00 + 180.00 + 663.60 + 1 911.80 + 1 911.80
        + 1 490.00 + 312.90 + 3 714.70 + 3 714.70 = 16 879.50
credits = 1 911.80 + 1 911.80 + 1 911.80 + 1 911.80
        + 901.45 + 901.45 + 3 714.70 + 3 714.70 = 16 879.50
```

**Per-account proof.**

| Account | Debits | Credits | Balance after the trace | Meaning |
|---|---|---|---|---|
| 1010 Bank | — | 5 626.50 | 5 626.50 credit | The cash actually paid, equal to 3 823.60 + 1 802.90 |
| 1310 Tax Receivable | 976.50 | — | 976.50 debit | 21 % of the 4 650.00 of net purchases |
| 1400 Inventory | 4 470.00 | — | 4 470.00 debit | Equals the physical value, part (d) |
| 1455 Outstanding Payments | 5 626.50 | 5 626.50 | 0.00 | Every payment confirmed by the bank |
| 2100 Trade Payables | 5 626.50 | 5 626.50 | 0.00 | Nothing owed |
| 5000 Cost of Goods Sold | 180.00 | — | 180.00 debit | The freight, which is a service and therefore never capitalised |

The untaxed purchase of 4 650.00 splits exactly into 4 470.00 of inventory asset and 180.00 of
expense.

**The window between the receipt and the bill.** Between 2026-06-05 and 2026-06-11 the warehouse
held 20 cabinets whose movement carried 2 900.00, while account 1400 Inventory carried nothing.
The gap is inherent to the arrangement of
[inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
section 0.1: under perpetual valuation a movement posts only when one of its two locations
carries a location valuation account, and neither a vendor location nor a warehouse does. A
balance sheet drawn on 2026-06-08 therefore understates the inventory asset by 2 900.00. The
inventory valuation closing is what repairs it: part two of the closing entry compares the
physical value with the ledger value of the inventory valuation account and posts the difference
as a debit on 1400 Inventory against a credit on 1410 Inventory Variation
([inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
section 6.2). Running the closing again after the bill has been posted reverses that adjustment
to zero, because the two sides then agree.

## 2.d Stock consequences

**Quantities per location.**

| Location | Filing cabinet before | After the first receipt | After the second |
|---|---|---|---|
| `WH/Stock` | 0 | 20 | 30 |
| `Partners/Vendors` | 0 | −20 | −30 |

A vendor location is never counted, so its quantity runs negative; it is outside the valued
perimeter, which is exactly why the movements that cross it are classified as **incoming**.

**Value carried by each completed goods movement**, after every re-valuation the trace performs.

| Movement | Direction | Quantity | Value at completion | Value after billing | Remaining quantity | Remaining value |
|---|---|---|---|---|---|---|
| R‑A (2026-06-05) | incoming | 20 | 2 900.00 | 2 980.00 | 20 | 2 980.00 |
| R‑B (2026-06-19) | incoming | 10 | 1 450.00 | 1 490.00 | 10 | 1 490.00 |

```formula
physical_value = 2 980.00 + 1 490.00 = 4 470.00
ledger_value   = 2 980.00 + 1 490.00 = 4 470.00
difference     = 0.00
```

Stored unit cost at the end: 4 470.00 ÷ 30 = **149.00**. The first-in-first-out stack now holds
R‑A at the bottom with 20 units at 149.00 and R‑B above it with 10 units at 149.00; a later
delivery would consume R‑A first.

## 2.e Variations

### 2.e.1 Bill control policy *on ordered quantities* on the Filing cabinet

Line 1's quantity to bill is 30 from the instant of confirmation, so the whole order can be
billed on 2026-06-02, before anything arrives. One bill replaces the two. Assuming the vendor
confirms the same 149.00 per cabinet, its figures are:

```formula
line_1_untaxed = round_to_currency( 30 × 149.00 ) = 4 470.00
line_2_untaxed = 180.00
untaxed_total  = 4 650.00
tax_total      = round_to_currency( 4 470.00 × 21 ÷ 100 ) + round_to_currency( 180.00 × 21 ÷ 100 )
               = 938.70 + 37.80 = 976.50
grand_total    = 5 626.50
instalment_1   = round_to_currency( 5 626.50 × 50 ÷ 100 ) = 2 813.25 , maturity 2026-06-02
instalment_2   = 5 626.50 − 2 813.25 = 2 813.25 , maturity 2026-07-31
```

The receipts are then valued from the bill rather than from the order, because source 1 of the
priority chain now answers:

```formula
first receipt of 20 :   available_value = 4 470.00 × 30 ÷ 30 = 4 470.00 , available_quantity = 30
                        remaining_quantity = 20 , below the available quantity
                        value = 20 × 4 470.00 ÷ 30 = 2 980.00
second receipt of 10 :  absorbed_quantity = 20
                        available_value = 4 470.00 × 10 ÷ 30 = 1 490.00 , available_quantity = 10
                        value = 1 490.00
```

**Delta against part (c).** Items 1 to 5 and 10 to 13 are replaced by a single entry dated
2026-06-02: debit 1400 Inventory 4 470.00; debit 5000 Cost of Goods Sold 180.00; debit 1310 Tax
Receivable 976.50; credit 2100 Trade Payables 2 813.25 and 2 813.25. Every per-account total of
part (c) is unchanged; only the dates and the number of documents move. The inventory asset is
now debited **before** the goods exist, so between 2026-06-02 and 2026-06-05 the ledger
overstates the inventory by 4 470.00 — the mirror of the window described in part (c).

### 2.e.2 Costing method *average cost*

An **incoming** movement is valued by the priority chain whatever the costing method, so R‑A and
R‑B carry exactly the same 2 980.00 and 1 490.00 as in part (c), and the bill items are
identical. **There is no delta in the ledger at all.**

What changes is the unit cost trajectory and what would happen to goods that had already left.
Under average cost the fast path gives 2 900.00 ÷ 20 = 145.00 after the first receipt; posting
the first bill re-values R‑A to 2 980.00 and the average replay of
[inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
section 4.3 sets the unit cost to 2 980.00 ÷ 20 = 149.00. The second receipt gives
( 20 × 149.00 + 1 450.00 ) ÷ 30 = 147.67, and posting the second bill replays the average to
4 470.00 ÷ 30 = 149.00. The end state is the same as under first in first out.

The difference bites when goods leave between the receipt and the bill. Suppose 5 cabinets were
delivered on 2026-06-08, when the unit cost was 145.00. Under average cost that delivery is
valued at 5 × 145.00 = 725.00 and is **never** revisited: the later re-valuation of R‑A raises
the value of the goods still on hand but does not chase the 5 that have gone, so the ledger keeps
725.00 of cost against goods that actually cost 5 × 149.00 = 745.00. Under first in first out the
same delivery is valued from the stack, and re-valuing R‑A afterwards leaves the delivery
untouched as well, with the same 20.00 discrepancy. In both methods the residue lands in the
inventory valuation account and is picked up by part two of the closing.

### 2.e.3 Costing method *standard price* at 145.00

Now the price-difference mechanism of
[inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
section 4 fires, because every one of its six conditions holds: the document is a vendor bill;
the company uses anglo-saxon accounting; the line is eligible for stock accounting; the costing
method is `standard`; the category's price difference account 5100 Price Difference resolves;
and the subtotal difference is not zero while the stored and computed unit prices agree at two
decimals.

```formula
valuation_unit_price      = 145.00                  ( the product's unit cost, in the document currency )
gross_unit_price          = 149.00
price_unit_difference     = 149.00 − 145.00 = 4.00
relevant_quantity         = 20                      ( on the first bill )
price_subtotal_difference = 20 × 4.00 = 80.00
```

**Delta on the first bill.** Two extra items, both marked as cost-recognition items and both
carrying the line's product, unit, quantity and analytic distribution:

| Account | Debit | Credit |
|---|---|---|
| 5100 Price Difference | 80.00 | |
| 1400 Inventory | | 80.00 |

so that the net debit on 1400 Inventory is 2 980.00 − 80.00 = 2 900.00 = 20 × 145.00, the
standard cost. **Delta on the second bill.** The same pair for 10 × 4.00 = 40.00, leaving a net
debit of 1 490.00 − 40.00 = 1 450.00 = 10 × 145.00.

The movements still carry 2 980.00 and 1 490.00, because the priority chain does not consult the
costing method; but a standard-cost product's reported value is not the sum of its movement
values, it is quantity on hand × unit cost
([inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
section 4.1):

```formula
physical_value = 30 × 145.00 = 4 350.00
ledger_value   = ( 2 980.00 − 80.00 ) + ( 1 490.00 − 40.00 ) = 4 350.00
difference     = 0.00
```

Had the accountant applied a discount on the bill line instead of retyping the unit price, the
stored unit price and the computed unit price would no longer agree at two decimals and **no
price difference would be posted at all**; the whole 149.00 would stay in the inventory asset
while the product's reported value stayed at 145.00 per unit, and the gap would surface only at
the closing.

### 2.e.4 Anglo-saxon against continental accounting

As in trace 1, the company-level anglo-saxon switch governs **only** the price-difference items
of variation 2.e.3. It does not gate the redirection of the bill line onto the inventory
valuation account; that is governed by the product's **valuation mode**. Turning the switch off
while keeping perpetual valuation therefore removes the two price-difference items of variation
2.e.3 and nothing else — under first in first out, as in part (c), it changes nothing whatever.

The continental treatment is obtained by setting the category's valuation mode to **periodic**:

- The condition of
  [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
  section 3 fails, so line 1 keeps the **expense** account 5000 Cost of Goods Sold.
- **Delta.** Item 1 becomes a debit of 2 980.00 on 5000 Cost of Goods Sold and item 10 a debit
  of 1 490.00 on the same account. Account 1400 Inventory is untouched by the whole trace and
  account 5000 ends at 2 980.00 + 1 490.00 + 180.00 = 4 650.00 debit, the entire untaxed
  purchase.
- The goods movements still post nothing, and their values are still 2 980.00 and 1 490.00.
- The inventory valuation closing run on 2026-06-30 posts part two of its entry:

  ```formula
  balance = 4 470.00 − 0.00 − 0.00 = 4 470.00
  ```

  a debit of 4 470.00 on 1400 Inventory against a credit of 4 470.00 on 1410 Inventory
  Variation, which restates the period's expense from 4 650.00 down to 180.00.

### 2.e.5 A foreign-currency order

The same order placed in United States dollars, with a rate of 1.0850 dollars for one euro on
2026-06-11. The commercial figures are unchanged in dollars; the balances are the dollar amounts
divided by the rate and rounded to the euro
([accounts-payable/accounting-effects.md](accounts-payable/accounting-effects.md) section 2.5,
variant).

| Item | Account | Amount in currency (dollar) | Debit (euro) | Credit (euro) |
|---|---|---|---|---|
| 1 | 1400 Inventory | +2 980.00 | 2 746.54 | |
| 2 | 5000 Cost of Goods Sold | +180.00 | 165.90 | |
| 3 | 1310 Tax Receivable | +663.60 | 611.61 | |
| 4 | 2100 Trade Payables | −1 911.80 | | 1 762.03 |
| 5 | 2100 Trade Payables | −1 911.80 | | 1 762.02 |

```formula
round_to_currency( 2 980.00 ÷ 1.0850 ) = round_to_currency( 2 746.543779 ) = 2 746.54
round_to_currency(   180.00 ÷ 1.0850 ) = round_to_currency(   165.898618 ) = 165.90
round_to_currency(   663.60 ÷ 1.0850 ) = round_to_currency(   611.612903 ) = 611.61
company_total  = 2 746.54 + 165.90 + 611.61 = 3 524.05
effective_rate = 3 823.60 ÷ 3 524.05 = 1.084999…
instalment_1_company  = round_to_currency( 3 524.05 × 50 ÷ 100 ) = round_to_currency( 1 762.025 ) = 1 762.03
instalment_1_currency = round_to_currency( 3 823.60 × 50 ÷ 100 ) = 1 911.80
instalment_2_company  = 3 524.05 − 1 762.03 = 1 762.02
instalment_2_currency = 3 823.60 − 1 911.80 = 1 911.80
```

Both columns close: euro debits 2 746.54 + 165.90 + 611.61 = 3 524.05 against euro credits
1 762.03 + 1 762.02 = 3 524.05; dollar amounts
2 980.00 + 180.00 + 663.60 − 1 911.80 − 1 911.80 = 0.00. Note that the two instalments differ by
one cent in euro although they are equal in dollars: the balance rule gives the last line the
whole residual, which is what absorbs the half-cent.

**The value handed to stock is always in the company currency.** The movement unit price of step
5 is converted into euro at the order deadline's rate and rounded to the Product Price precision
before it ever reaches the valuation engine, and the value taken from a bill line is the line
subtotal divided by **that bill's own stored rate**, not today's rate
([inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
section 1.3). With the rate above, R‑A is re-valued to
round_to_currency( 2 980.00 ÷ 1.0850 ) = 2 746.54 euro, exactly the figure the bill debited to
1400 Inventory. Inventory and ledger therefore stay equal in a foreign-currency purchase, which
is the opposite of the customer-invoice case recorded as a compatibility finding in
[variation 1.e.5](#1e5-a-foreign-currency-order).

**Settlement at a different rate.** The second instalment of 1 911.80 dollars is paid on
2026-07-31 at a rate of 1.0600. The payment's counterpart carries 1 911.80 dollars and a balance
of round_to_currency( 1 911.80 ÷ 1.0600 ) = round_to_currency( 1 803.584906 ) = 1 803.58 euro.
Reconciling it with item 5 brings the dollar residual to zero but leaves a euro residual of
1 803.58 − 1 762.02 = 41.56 on the credit side. An exchange-difference entry is produced in the
Exchange Difference journal, dated on the later of the two matched items' dates:

| Account | Amount in currency (dollar) | Debit (euro) | Credit (euro) |
|---|---|---|---|
| 2100 Trade Payables | 0.00 | 41.56 | |
| 7560 Foreign Exchange Gain | 0.00 | | 41.56 |

The gain account is the one used, because a payable whose foreign currency has weakened costs the
company fewer units of company currency than the amount at which it was recognised.

The payable item of the exchange entry is reconciled with item 5, which brings the euro residual
to zero as well. Both items carry a **zero** amount in currency: an exchange difference corrects
only the company-currency side.

### 2.e.6 Cash rounding to the nearest five hundredths

The company configures a cash rounding method *Nearest 0.05*, half away from zero, with the
add-a-rounding-line strategy, a profit account 7580 Cash Rounding Gain and a loss account 6580
Cash Rounding Loss, and it is applied to `BILL/2026/00087`.

```formula
rounded_total = round_to_step( 3 823.60 , 0.05 , half away from zero ) = 3 823.60
difference    = 3 823.60 − 3 823.60 = 0.00
```

The total is already a multiple of five hundredths, so **no rounding item is produced**. Change
the freight price to 179.99 and the arithmetic becomes:

```formula
line_B_untaxed = 179.99
line_B_tax     = round_to_currency( 179.99 × 21 ÷ 100 ) = round_to_currency( 37.7979 ) = 37.80
grand_total    = 2 980.00 + 179.99 + 625.80 + 37.80 = 3 823.59
rounded_total  = round_to_step( 3 823.59 , 0.05 , half away from zero ) = 3 823.60
difference     = 3 823.60 − 3 823.59 = 0.01
```

The rounding item raises what is owed, and on a vendor bill the untaxed side is a debit, so the
**loss** account is used ([accounts-payable/accounting-effects.md](accounts-payable/accounting-effects.md)
section 2.2). The payment term then distributes the rounded total, applying the cash-rounding
correction to every instalment except the last:

```formula
instalment_1 = round_to_currency( 3 823.60 × 50 ÷ 100 ) = 1 911.80
correction   = round_to_step( 1 911.80 , 0.05 , half away from zero ) − 1 911.80 = 0.00
instalment_2 = 3 823.60 − 1 911.80 = 1 911.80
```

**Delta against part (c).** Item 2 becomes 179.99; one new item appears, a debit of 0.01 on 6580
Cash Rounding Loss; items 4 and 5 are unchanged at 1 911.80 each. Totals: debits
2 980.00 + 179.99 + 663.60 + 0.01 = 3 823.60 against credits 1 911.80 + 1 911.80 = 3 823.60. The
rounding item carries no tax and no tax grid, so the tax return is untouched.

### 2.e.7 An early payment discount of two per cent within ten days

An early payment discount requires a **single-line** payment term, so the term becomes one line
of 100 % at 30 days carrying a discount of 2 % within 10 days, in the computation mode *On early
payment*. `BILL/2026/00087` then carries one payable item of 3 823.60 maturing 2026-07-11,
stamped with a discount deadline of 2026-06-21 and a discounted amount of

```formula
discount_amount = 3 823.60 × 2 ÷ 100 = 76.472
amount_due      = round_to_currency( 3 823.60 − 76.472 ) = round_to_currency( 3 747.128 ) = 3 747.13
```

The bill is posted at its full value; nothing extra appears on it. The company pays 3 747.13 on
2026-06-18, inside the deadline, and the register-payment assistant adds the write-off items to
the **payment's own** entry so that the payable clears exactly:

| Account | Label | Debit | Credit |
|---|---|---|---|
| 2100 Trade Payables | `NGS-4471` | 3 823.60 | |
| 1455 Outstanding Payments | `NGS-4471` | | 3 747.13 |
| 7900 Cash Discount Taken | `Early Payment Discount` | | 63.20 |
| 1310 Tax Receivable | `Early Payment Discount (Purchase 21 %)` | | 13.27 |

```formula
net_discount = round_to_currency( 3 160.00 × 2 ÷ 100 ) = round_to_currency( 63.20 ) = 63.20
tax_on_it    = round_to_currency( 63.20 × 21 ÷ 100 ) = round_to_currency( 13.272 ) = 13.27
63.20 + 13.27 = 76.47 = 3 823.60 − 3 747.13
```

The write-off base item carries the base grids of the Purchase 21 % tax with the refund sign and
the write-off tax item carries that tax's grids with the refund sign, so the tax return reports
a base reduction of 63.20 and a deductible-tax reduction of 13.27. In the *Never* mode only the
net 63.20 would be written off and no tax item would be produced; in the *Always (upon invoice)*
mode the bill itself would carry a cancelling pair of items on the line's own account, one with
the tax and one without, and the tax would be computed on 3 160.00 − 63.20 = 3 096.80 from the
start.

### 2.e.8 Several companies

Buying from another company of the group turns this trace into the inter-company case, where the
purchase order of the buyer is mirrored by a sales order of the seller and the vendor bill of the
buyer by a customer invoice of the seller. That is traced in
[trace 9](#9-drop-shipping-and-inter-company-trade). Within this trace, three rules bound the
single-company behaviour: a bill is always created in the **order's** company, so its accounts,
journals and taxes are that company's; grouping several orders onto one bill groups by company as
well as by vendor and currency, so a bill never spans two companies; and a product belonging to a
company outside the order company's accessible branch tree is refused at the order-level
consistency check, whose message is in part (f).

### 2.e.9 Receiving in two steps

The Main Warehouse is reconfigured to receive in two steps
([inventory-operations/workflows.md](inventory-operations/workflows.md) section 5). The reception
route then holds a pull rule `Partners/Vendors` → `WH/Stock` whose operation type *Receipts* has
`WH/Input` as its default destination, and a push rule `WH/Input` → `WH/Stock` with the operation
type *Internal Transfers*.

- The receipt movement now runs `Partners/Vendors` → `WH/Input`. `WH/Input` is an internal
  location belonging to the company, so it is **inside** the valued perimeter; the movement is
  still incoming and is valued exactly as in step 6.
- Completing it applies the push rule, which creates a second movement `WH/Input` → `WH/Stock`
  in a Transfer numbered from the storage operation type's sequence, with the supply method
  make to order and the receipt movement as its originating movement.
- That second movement has **both** ends inside the valued perimeter, so it is neither incoming
  nor outgoing, its valued quantity is zero, and it is not valued at all.

**Delta against part (c): none.** Not one journal item changes. What changes is the quantity
table of part (d), which gains an intermediate row: after the first receipt, `WH/Input` holds 20
and `WH/Stock` holds 0 until the storage Transfer is validated.

## 2.f Failure points

Each entry gives the step, the condition, the domain that refuses and the exact text.

**Step 1, the reordering rule finds no way to replenish** (replenishment and procurement). Raised
when the product carries no route that reaches `WH/Stock`, or when the warehouse route filter
drops the reception route because the product has no vendor pricelist entry. The message is two
lines; the scheduler records it as a warning activity on the product template rather than
aborting the whole pass.

`No rule has been found to replenish "<the product display name>" in "<the location display name>".`

`Verify the routes configuration on the product.`

**Step 3, the buy action finds no usable vendor price** (purchasing), when the run is driven by a
reordering rule.

`There is no matching vendor price to generate the purchase order for product the product name (no vendor defined, minimum quantity not reached, dates not valid, ...). Go on the product form and complete the list of vendors.`

**Step 4, confirming an order with a line that has no product** (purchasing). Display lines and
down-payment lines are exempt. The check is per order and aborts the whole operation for that
order; nothing is written.

`Some order lines are missing a product, you need to correct them before going further.`

**Step 4, confirming an order carrying a product of another company** (purchasing).

`Your quotation contains products from company the offending company names, comma separated whereas your quotation belongs to company the order company name. Please change the company of your quotation or remove the products from other companies (the offending product names, comma separated).`

**Step 4, confirming when a mandatory analytic plan is unsatisfied** (purchasing, delegating to
analytic accounting). Every analytic plan marked mandatory for the business domain *purchase
order* must be covered by each non-display line's analytic distribution; the message belongs to
[analytic-accounting/business-rules.md](analytic-accounting/business-rules.md).

**Step 5, the vendor has no supplier location** (purchasing). The receipt cannot be built.

`You must set a Vendor Location for this partner the vendor name`

**Step 5, the operation type and the reordering rule disagree about the warehouse** (purchasing).

`The warehouse of operation type (the operation type name) is inconsistent with location (the location name) of reordering rule (the reordering rule name) for product the product name. Change the operation type or cancel the request for quotation.`

**Step 6, validating a transfer with no movements and no detail lines** (inventory operations).

`You can’t validate an empty transfer. Please add some products to move before proceeding.`

**Step 6, validating a transfer where every movement to process has a zero processed quantity**
(inventory operations). The line break in the middle is part of the message.

`Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!`

**Step 6, validating a transfer of a tracked product without a lot or serial number** (inventory
operations), when the operation type allows creating or using lots.

`You need to supply a Lot/Serial number for products <the comma-separated product display names>.`

**Step 6, valuing an incoming movement of a product whose value is kept per lot, when a detail
line carries no lot** (inventory valuation and costing).

`A lot/serial number is required for product '<the product display name>' as it has lot valuation enabled.`

**Step 8, posting a purchase document with no vendor** (accounts payable).

`The field 'Vendor' is required, please complete it to validate the Vendor Bill.`

**Step 8, posting a purchase document with no bill date** (accounts payable). A sale document
would silently take today's date; a purchase document is refused.

`The Bill/Refund date is required to validate this document.`

**Step 8, posting a document whose total is negative** (accounts payable). This is what a
return-driven bill runs into before it is switched to a vendor refund.

`You cannot validate an invoice with a negative total amount. You should create a credit note instead. Use the action menu to transform it into a credit note or refund.`

**Step 8, posting a document with no accountable line** (accounts payable). Sections, subsections
and notes do not count.

`Even magicians can't post nothing!`

**Step 8, posting a document that is not draft** (accounts payable).

`The entry «the number» (id «the identifier») must be in draft.`

**Step 8, posting with an archived account, journal or currency** (accounts payable), one message
per case.

`The account «the account name» («the account code») is archived.`

`A line of this move is using a archived account, you cannot post it.`

`You cannot post an entry in an archived journal («the journal's display name»)`

`You cannot validate a document with an inactive currency: «the currency code»`

**Step 8, posting when the quick-encoding total disagrees with the computed total** (accounts
payable).

`The current total is «the computed total» but the expected total is «the typed total». In order to post the invoice/bill, you can adjust its lines or the expected Total (tax inc.).`

**Step 8 under variation 2.e.3, a kit whose components carry no value** (purchasing). Raised when
the cost-recognition entries cannot be produced because the number of complete kits derived from
the component movements is zero. The shipped text opens with the application's own name, rendered
here as *The system*.

`The system is not able to generate the anglo saxon entries. The total valuation of the product name is zero.`

**Step 10, registering a payment when the payment method line has no outstanding account**
(payments and bank reconciliation).

`You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal.`

**Step 11, creating a bank transaction in a journal with no suspense account** (payments and bank
reconciliation).

`You can't create a new statement line without a suspense account set on the <the journal display name> journal.`

**Cancelling the order after a bill has been posted** (purchasing).

`Unable to cancel purchase order(s): the order display names. You must first cancel their related vendor bills.`

**Deleting the order before cancelling it** (purchasing).

`In order to delete a purchase order, you must cancel it first.`

**Deleting a product line of a confirmed order** (purchasing). Sections, subsections and notes may
still be deleted. The label is the human label of the status, which for a confirmed order is
*Purchase Order*.

`Cannot delete a purchase order line which is in state “the status label”.`

**Lowering the ordered quantity below the billed quantity** (purchasing). No refusal: a warning
activity is scheduled on the first related bill, carrying the note

`The quantities on your purchase order indicate less than billed. You should ask for a refund.`

---

# 3. Make to stock and make to order with components and work orders

A reordering rule pulls a shelf unit into stock through a manufacturing order with one work
order; a customer then orders a display cabinet that is built to order from that shelf unit and
a glass front, through a second manufacturing order with its own work order; the cabinets are
delivered, invoiced and paid. The trace crosses
[replenishment and procurement](replenishment-and-procurement/), [manufacturing](manufacturing/),
[inventory operations](inventory-operations/),
[inventory valuation and costing](inventory-valuation-and-costing/), [sales](sales/),
[taxes](taxes/), [accounts receivable](accounts-receivable/) and
[payments and bank reconciliation](payments-and-bank-reconciliation/).

Its point is the production location. Unlike a vendor location or a customer location, the
production location **carries a location valuation account**, so every movement into and out of
it posts a journal entry of its own. Manufacturing is therefore the one part of the goods flow
that writes to the ledger at the moment the goods move rather than at the moment a document is
posted, and the production account is the hinge on which the arithmetic closes.

## 3.a Starting records

Everything of the [shared fixture](#the-shared-fixture) applies. In addition:

**Product categories.** Two are added, both carrying the income account 4000 Product Sales, the
expense account 5000 Cost of Goods Sold, the inventory valuation account 1400 Inventory, the
price difference account 5100 Price Difference and the inventory journal Inventory Valuation:
*Furniture* for the finished goods and *Components* for the parts. Neither declares a production
account of its own, so the company-level fallback applies and the production account of every
product below is **5200 Production Cost**, which is also the valuation account carried by the
`WH/Production` location.

**Product — Steel bracket.** Category *Components*. Goods product, storable, reference unit
`Units`. Costing method **first in first out**, valuation mode **perpetual**. Two completed
incoming goods movements, both already billed:

| Movement | Date | Source | Destination | Quantity | Value | Unit value |
|---|---|---|---|---|---|---|
| B1 | 2026-03-16 | `Partners/Vendors` | `WH/Stock` | 120 `Units` | 420.00 | 3.50 |
| B2 | 2026-04-09 | `Partners/Vendors` | `WH/Stock` | 100 `Units` | 380.00 | 3.80 |

Quantity on hand 220; total value 800.00; stored unit cost 800.00 ÷ 220 = 3.636363… stored as
**3.64**. Removal strategy at `WH/Stock`: first in first out.

**Product — Oak panel.** Category *Components*. Goods product, storable, reference unit `Units`.
Costing method **average cost**, valuation mode **perpetual**. One completed incoming goods
movement P1 of 60 `Units` on 2026-03-30 valued 1 470.00; quantity on hand 60, stored unit cost
**24.50**, total value 1 470.00.

**Product — Glass front.** Category *Components*. Goods product, storable, reference unit
`Units`. Costing method **first in first out**, valuation mode **perpetual**. One completed
incoming goods movement G1 of 30 `Units` on 2026-04-14 valued 1 560.00; stored unit cost
**52.00**.

**Product — Shelf unit.** Category *Furniture*. Goods product, storable, reference unit `Units`.
Costing method **average cost**, valuation mode **perpetual**. Quantity on hand zero; stored unit
cost 0.00. Routes: **Manufacture** only. Not sold on its own in this trace.

**Product — Display cabinet.** Category *Furniture*. Goods product, storable, reference unit
`Units`. Costing method **first in first out**, valuation mode **perpetual**. Quantity on hand
zero; stored unit cost 0.00. Sales price 690.00; customer tax Sales 21 %; invoicing policy
**delivered quantities**. Routes: **Manufacture** and **Replenish on Order**, the latter holding
a pull rule from `WH/Stock` to `Partners/Customers` whose supply method is make to order.

**Bill of materials — Shelf unit.** Type `normal`, quantity 1 `Units`, consumption policy
*Allowed with warning* (stored value `warning`). Components: 4 `Units` of Steel bracket and 2
`Units` of Oak panel. One operation, *Assemble*, at the work centre **Assembly bench**: cycle
time 30 minutes, capacity 1 unit, time efficiency 100 %, no setup time, no cleanup time, hourly
cost 48.00, expense account 5300 Manufacturing Overhead. Cost mode of the work order: *estimated*.
Manufacturing lead time: 2 days. Extra unit cost: 0.00. No by-product.

**Bill of materials — Display cabinet.** Type `normal`, quantity 1 `Units`, consumption policy
*Allowed with warning*. Components: 1 `Units` of Shelf unit and 1 `Units` of Glass front. One
operation, *Glazing*, at the work centre **Glazing station**: cycle time 45 minutes, capacity 1
unit, time efficiency 100 %, no setup time, no cleanup time, hourly cost 60.00, expense account
5300 Manufacturing Overhead. Cost mode: *estimated*. Manufacturing lead time: 3 days. Extra unit
cost: 0.00. No by-product.

**Reordering rule.** Product Shelf unit, location `WH/Stock`, minimum quantity 4, maximum quantity
10, replenishment multiple 1, trigger automatic, route Manufacture.

**Manufacturing operation type.** *Manufacturing*, source location `WH/Stock`, destination
location `WH/Stock`, components location `WH/Stock`, finished-products location `WH/Stock`,
backorder policy *ask*, sequence producing references of the form `WH/MO/…`. The recipe consumes
into and produces out of `WH/Production`, which is the company's production location and carries
the location valuation account 5200 Production Cost.

**Customer.** Ashgrove Fitters, its own commercial entity. Receivable account 1200 Trade
Receivables. No fiscal position. Payment term **"Immediate"**: one line, 100 %, delay type *days
after invoice date*, 0 days, no early payment discount.

**Dates.** Scheduler run 2026-04-27. First manufacturing order closed 2026-04-30. Sales order
2026-05-11. Second manufacturing order closed 2026-05-18. Delivery 2026-05-19. Invoice
2026-05-20. Payment 2026-05-22, on the bank statement 2026-05-25.

## 3.b Steps

### Part A — making the shelf units to stock

1. **The reordering rule fires.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   sections 11 and 12.1.

   ```formula
   quantity_forecast = 0
   compare( 0 , 4 , Units ) = −1                     ( a replenishment is needed )
   raw_quantity      = max( 4 , 10 ) − 0 = 10
   multiple          = 1 , so no rounding up applies
   quantity_to_order = 10
   ```

   One procurement request is built: product Shelf unit, 10 `Units`, location `WH/Stock`, route
   Manufacture, reordering rule this rule, company Northwind Trading, planned date the lead
   horizon date pulled back by the company's replenishment horizon.

2. **The rule search resolves to a manufacture rule.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   sections 1 and 3. The warehouse route filter keeps the Manufacture route because the product
   has a bill of materials of type `normal`. The rule found at `WH/Stock` has the action
   *manufacture*, so the request is filed under the manufacture action.

3. **The manufacture action creates the order.** Domain: replenishment and procurement,
   [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
   section 6; the order itself belongs to manufacturing,
   [manufacturing/workflows.md](manufacturing/workflows.md) section 3.
   - The bill of materials is the *Shelf unit* recipe. No open order matches the extension
     search, so a new one is created: reference `WH/MO/00021`, product Shelf unit, quantity 10
     `Units`, source location `WH/Stock`, destination location `WH/Stock`, final location
     `WH/Stock`, start date the planned date minus the recipe's 2 days of manufacturing lead
     time, operation type *Manufacturing*, reordering rule this rule, responsible user empty.
   - A production group named after the reference is created and stamped on every movement, and
     a Stock Reference named after the order is created.
   - The recipe explosion generates two component movements — 40 `Units` of Steel bracket and 20
     `Units` of Oak panel, both `WH/Stock` → `WH/Production` — one finished movement of 10
     `Units` of Shelf unit, `WH/Production` → `WH/Stock`, and one Work Order for the operation
     *Assemble*.
   - The order **came from a reordering rule and has component movements**, so it is *not*
     confirmed at once; it is left in state `draft` and confirmed in the post-processing step,
     after every reordering rule of the batch has run. Confirming earlier would let its component
     needs interfere with the rules still to be processed.

4. **Confirm the order.** Domain: manufacturing,
   [manufacturing/workflows.md](manufacturing/workflows.md) section 4.1; states in
   [manufacturing/state-machines.md](manufacturing/state-machines.md) section 1.
   - Company consistency is checked between the order and every record it references.
   - The consumption policy `warning` is copied from the recipe onto the order.
   - The supply method of each component movement is adjusted: neither Steel bracket nor Oak
     panel is made to order, so both take from stock.
   - Every component and finished movement is confirmed **without merging**, so the
     one-movement-per-recipe-line correspondence survives.
   - The Work Order is confirmed and its cost mode `estimated` is copied once from the operation.
   - Status `draft` → `confirmed`. Readiness is computed from the component movements.
   - **No journal item.** Confirming a manufacturing order posts nothing; the base entities of
     the domain never produce journal entries
     ([manufacturing/accounting-effects.md](manufacturing/accounting-effects.md), preamble).

5. **Reserve the components.** Domain: manufacturing,
   [manufacturing/workflows.md](manufacturing/workflows.md) section 4.2, delegating to
   [inventory-operations/workflows.md](inventory-operations/workflows.md) section 11. Both
   component movements are reserved at `WH/Stock` under the first-in-first-out removal strategy.
   Readiness becomes *Ready*.

6. **Plan and run the work order.** Domain: manufacturing,
   [manufacturing/workflows.md](manufacturing/workflows.md) section 4.4, with the duration of
   [manufacturing/calculations.md](manufacturing/calculations.md) section 5.1, case A.

   ```formula
   quantity          = 10                    ( the quantity to produce )
   cycle_number      = round_up_to_integer( 10 ÷ 1 ) = 10
   duration_expected = 0 + 0 + 10 × 30 × 100 ÷ 100 = 300 minutes
   ```

   The operator starts and finishes the Work Order on 2026-04-30. Its status becomes `done`; the
   order's state becomes `to_close`, because every Work Order is done.

7. **Mark the order as done.** Domain: manufacturing,
   [manufacturing/workflows.md](manufacturing/workflows.md) section 5.3.
   - Sanity checks pass: company consistency, and serial-number uniqueness vacuously, because
     nothing is tracked.
   - The automatic-filling decision qualifies the order, because nothing among its components
     and finished products is tracked. The quantity producing is set to 10 − 0 = 10 and the
     distribution of [manufacturing/calculations.md](manufacturing/calculations.md) section 7.2
     fills 40 Steel brackets and 20 Oak panels, marking both movements picked.
   - The consumption check runs, because the policy is not `flexible`. Expected for Steel
     bracket: 4 × 10 ÷ 1 = 40, consumed 40; expected for Oak panel: 2 × 10 ÷ 1 = 20, consumed 20.
     No difference, so the Consumption Warning assistant does not open.
   - The backorder check finds `max( 10 − 10 , 0 ) = 0` to backorder, so no question is asked.

8. **Post the component consumption.** Domain: inventory valuation and costing,
   [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
   sections 2.2 and 5.2, and
   [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
   section 1. Both component movements are **outgoing**: their source `WH/Stock` is inside the
   valued perimeter and their destination `WH/Production` — a production location — is outside
   it.
   - **Steel bracket, first in first out.** The stack is built by walking completed incoming
     movements newest first until the quantity on hand, 220, is covered: B2 offers 100 and
     leaves 120; B1 offers 120 and leaves 0. Reversed, the stack is B1 then B2, with a bottom
     quantity of 120 on B1. Consuming 40:

     ```formula
     offered_quantity = 120                     ( the bottom quantity of B1 )
     offered_value    = 420.00 × 120 ÷ 120 = 420.00
     offered_value    = 420.00 × 40 ÷ 120 = 140.00      ( scaled, because 120 exceeds the 40 wanted )
     component_value  = 140.00                  ( 3.50 per unit )
     ```

   - **Oak panel, average cost.** An outgoing movement takes the stored unit cost and never
     changes it:

     ```formula
     component_value = 20 × 24.50 = 490.00
     ```

   - Both movements qualify for a journal entry, because their **destination** carries a
     location valuation account. The two are folded into a **single** entry in the company's
     inventory journal, Inventory Valuation, dated today in the acting time zone, referenced
     with the movements' references joined by a comma and a space, with no counterparty because
     a manufacturing movement belongs to no transfer. Per movement, the destination carries the
     valuation account, so the second case of the account rule applies: **debit the destination
     location's valuation account, credit the product's inventory valuation account**.
   - Unit costs are recomputed. Steel bracket, first-in-first-out full path: quantity on hand
     220 − 40 = 180, total value 800.00 − 140.00 = 660.00, unit cost 660.00 ÷ 180 = 3.666666…
     stored as **3.67**. Oak panel, average cost: an outgoing movement changes neither the value
     per unit nor the replayed average, so the unit cost stays **24.50** and the total value
     becomes 1 470.00 − 490.00 = 980.00.

9. **Compute the production cost and post the finished goods.** Domain: manufacturing,
   [manufacturing/calculations.md](manufacturing/calculations.md) section 9.3, and
   [manufacturing/accounting-effects.md](manufacturing/accounting-effects.md) section 2.2.

   ```formula
   work_order_cost  = ( 300 ÷ 60 ) × 48.00 = 5 × 48.00 = 240.00
   work_centre_cost = 240.00
   quantity         = 10                       ( the done quantity of the finished movement )
   extra_cost       = 0.00 × 10 = 0.00
   total_cost       = ( 140.00 + 490.00 ) + 240.00 + 0.00 = 870.00
   byproduct_cost_share = 0
   finished_price_unit = 870.00 × round_to_4_decimals( 1 − 0 ÷ 100 ) ÷ 10
                       = 870.00 × 1 ÷ 10 = 87.00
   ```

   The Work Order's cost is **estimated**, because its state is `done`, its expected duration is
   non-zero and its cost mode is `estimated`; the expected 300 minutes are used whatever the
   operator's stopwatch said.

   The finished movement is **incoming**: its source `WH/Production` is outside the valued
   perimeter and its destination `WH/Stock` is inside it. Its value comes from source 2 of the
   priority chain, production:

   ```formula
   finished_move_value = 10 × 87.00 = 870.00
   ```

   Its **source** carries the location valuation account, so the first case of the account rule
   applies: **debit the product's inventory valuation account, credit the source location's
   valuation account**.

   The Shelf unit's unit cost is recomputed by the incremental fast path:

   ```formula
   added_value       = 870.00
   added_quantity    = 10
   quantity_on_hand  = 10
   previous_quantity = 10 − 10 = 0            ( not greater than zero )
   unit_cost         = 870.00 ÷ 10 = 87.00
   ```

10. **Post the labour entry.** Domain: manufacturing,
    [manufacturing/accounting-effects.md](manufacturing/accounting-effects.md) section 2.4. All
    four preconditions hold: the finished product's valuation mode is perpetual; the production
    location carries a valuation account; no time log of the order's Work Orders already carries
    a journal item; and the total work-centre cost, 240.00, is not zero at the company currency.

    ```formula
    labour_amount[ 5300 Manufacturing Overhead ] = round_to_currency( 240.00 ) = 240.00
    work_centre_cost                             = 240.00
    labour_amount[ 5200 Production Cost ]        = − 240.00
    line_balance( 5300 ) = − 240.00      →  a credit of 240.00
    line_balance( 5200 ) = + 240.00      →  a debit  of 240.00
    ```

    The entry is written in the finished product's inventory journal, Inventory Valuation, dated
    today, with the reference and every line name equal to the order reference followed by
    ` - Labour`. Each item except the last is written back onto the time logs of the Work Order
    whose cost it carries, which is what stops the labour being posted twice.

    The order's state becomes `done`, its finish date the current instant, its priority `0` and
    its locked flag true. Every component and finished movement that is neither done nor
    cancelled is written to `done` with a demand of zero.

    **The production account closes.** 140.00 + 490.00 + 240.00 − 870.00 = 0.00. It holds nothing
    once the components, the labour and the finished goods have all gone through it, which is the
    invariant that proves the allocation was complete.

### Part B — building the display cabinets to order

11. **Create and confirm the sales order.** Domain: sales,
    [sales/workflows.md](sales/workflows.md) sections 1 and 4. Order `S00061`, customer Ashgrove
    Fitters, order date 2026-05-11, warehouse *Main Warehouse*, payment term "Immediate", one
    line: Display cabinet, 2 `Units`, unit price 690.00, no discount, tax Sales 21 %.

    ```formula
    line_untaxed  = round_to_currency( 2 × 690.00 ) = 1 380.00
    line_tax      = round_to_currency( 1 380.00 × 21 ÷ 100 ) = round_to_currency( 289.80 ) = 289.80
    grand_total   = 1 669.80
    ```

    Phase C writes status `sale`; phase E launches procurement for the line: product Display
    cabinet, 2 `Units`, final location `Partners/Customers`, warehouse *Main Warehouse*, deadline
    2026-05-19, originating order line = the line.

12. **Procurement picks the make-to-order rule, not the delivery rule.** Domain: replenishment and
    procurement,
    [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
    sections 1 and 4. At the candidate location `Partners/Customers` the route precedence tries
    the routes named on the request, then the package type's routes, then **the product's own
    routes**, then the warehouse's. The product's routes are Manufacture and Replenish on Order;
    the Manufacture route has no rule whose destination is `Partners/Customers`, the Replenish on
    Order route has one, so that rule wins before the warehouse's delivery rule is ever
    considered.

    The pull action creates one Stock Move of 2 `Units`, `WH/Stock` → `Partners/Customers`, with
    supply method **make to order**, and confirms it. It is grouped into a Transfer
    `WH/OUT/00033`.

13. **Confirming that movement raises the supplying need.** Domain: replenishment and procurement,
    [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
    sections 8 and 9. Because the movement's supply method is make to order, it becomes
    *waiting* — it reserves nothing from general stock — and a procurement request is created for
    it: product Display cabinet, 2 `Units`, location `WH/Stock` (the movement's **source**
    location), downstream movements = this delivery movement, deadline 2026-05-19.

    The rule search at `WH/Stock` now finds the Manufacture route's rule, so the manufacture
    action runs.

14. **The second manufacturing order.** Domain: replenishment and procurement,
    [replenishment-and-procurement/workflows.md](replenishment-and-procurement/workflows.md)
    section 6, and manufacturing. Order `WH/MO/00022`, product Display cabinet, quantity 2
    `Units`, final location `Partners/Customers`, downstream movements = the delivery movement,
    start date 2026-05-19 minus the recipe's 3 days of manufacturing lead time. It has component
    movements and did **not** come from a reordering rule, so it is created **and confirmed
    immediately** — the contrast with step 3.

    The explosion generates two component movements — 2 `Units` of Shelf unit and 2 `Units` of
    Glass front, both `WH/Stock` → `WH/Production` — one finished movement of 2 `Units` of
    Display cabinet, `WH/Production` → `WH/Stock`, linked to the delivery movement, and one Work
    Order for the operation *Glazing*.

15. **Plan, run and close the second order.** Same domains and the same sequence as steps 5 to 10.

    ```formula
    cycle_number      = round_up_to_integer( 2 ÷ 1 ) = 2
    duration_expected = 0 + 0 + 2 × 45 × 100 ÷ 100 = 90 minutes
    work_order_cost   = ( 90 ÷ 60 ) × 60.00 = 1.5 × 60.00 = 90.00
    ```

    Component values on 2026-05-18:

    ```formula
    shelf_unit_value  = 2 × 87.00 = 174.00          ( average cost, at the stored unit cost )
    glass_front_value = 1 560.00 × 2 ÷ 30 = 104.00  ( first in first out, from the single layer G1 )
    ```

    The Glass front stack at that moment holds only G1, whose bottom quantity is 30; consuming 2
    scales its value by 2 ÷ 30.

    ```formula
    quantity            = 2
    total_cost          = ( 174.00 + 104.00 ) + 90.00 + 0.00 = 368.00
    finished_price_unit = 368.00 × 1 ÷ 2 = 184.00
    finished_move_value = 2 × 184.00 = 368.00
    ```

    Unit costs afterwards: Shelf unit stays at **87.00** with 8 on hand and 696.00 of value;
    Glass front, first-in-first-out full path, 28 on hand and 1 560.00 − 104.00 = 1 456.00 of
    value, unit cost 1 456.00 ÷ 28 = **52.00**; Display cabinet, first-in-first-out full path, 2
    on hand and 368.00 of value, unit cost **184.00**.

    The production account closes again: 174.00 + 104.00 + 90.00 − 368.00 = 0.00.

16. **Deliver.** Domain: inventory operations,
    [inventory-operations/workflows.md](inventory-operations/workflows.md) sections 7 and 13. The
    finished goods reaching `WH/Stock` trigger the reservation of the waiting delivery movement,
    which becomes `assigned`. On 2026-05-19 the operator validates `WH/OUT/00033` in full.
    - Valuation: the movement is outgoing; the Display cabinet's stack holds one layer, the
      production movement of 2 at 184.00.

      ```formula
      offered_quantity = 2
      offered_value    = 368.00 × 2 ÷ 2 = 368.00
      delivery_value   = 368.00                    ( 184.00 per unit )
      ```

    - **No journal entry**: neither `WH/Stock` nor `Partners/Customers` carries a location
      valuation account.
    - Unit cost, first-in-first-out full path: quantity on hand 0, so the cost falls back to the
      unit price of the most recent incoming movement, 368.00 ÷ 2 = **184.00**, and is left
      there.
    - The order line's delivered quantity becomes 2, so its quantity to invoice becomes 2 and
      its invoice status becomes `to invoice`; the order's delivery status becomes `full`.

17. **Create and post the invoice.** Domain: sales,
    [sales/workflows.md](sales/workflows.md) section 8.3, then accounts receivable,
    [accounts-receivable/accounting-effects.md](accounts-receivable/accounting-effects.md)
    section 1. Invoice `INV/2026/00512` in the Customer Invoices journal, partner Ashgrove
    Fitters, invoice date and accounting date 2026-05-20, one line: Display cabinet, 2 `Units` at
    690.00, tax Sales 21 %.
    - Amounts: untaxed 1 380.00, tax 289.80, total 1 669.80.
    - Payment term "Immediate", one line of 100 % at 0 days: a single receivable item of
      1 669.80 maturing 2026-05-20.
    - **The cost pair is injected at posting.** Domain: inventory valuation and costing,
      [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
      section 2, with the arithmetic of
      [inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
      sections 9.1 to 9.3.

      ```formula
      cogs_quantity   = 0 ( nothing posted yet ) + 2 = 2
      unit_price      = 368.00 ÷ 2 = 184.00      ( from the one completed delivery movement )
      already_posted  = 0.00
      cogs_unit_price = | 184.00 × 2 − 0.00 | ÷ 2 = 184.00
      amount_currency = +1 × 2 × 184.00 = 368.00
      ```

      The inventory-side item carries minus that amount and is a **credit** of 368.00 on 1400
      Inventory; the expense-side item is a **debit** of 368.00 on 5000 Cost of Goods Sold. Both
      carry display kind *cost of goods sold*.
    - The order line's invoiced quantity reaches 2, so its invoice status becomes `invoiced` and
      the order's invoice status becomes `invoiced`.

18. **Collect.** Domain: payments and bank reconciliation,
    [payments-and-bank-reconciliation/accounting-effects.md](payments-and-bank-reconciliation/accounting-effects.md)
    sections 1.3 and 1.4. On 2026-05-22 an inbound Payment of 1 669.80 is created from the
    invoice in the Bank journal with the payment method *Manual Payment*, whose payment account is
    1450 Outstanding Receipts. Its entry emits the liquidity item first, a debit of 1 669.80 on
    1450, then a credit of 1 669.80 on 1200 Trade Receivables; the counterpart item is reconciled
    with the invoice's receivable item, which brings the residual to zero.

19. **Match the bank transaction.** On 2026-05-25 a Bank Transaction of +1 669.80 is created in
    the Bank journal, its counterpart is moved from the suspense account onto 1450 Outstanding
    Receipts and matched with the Payment's liquidity item. The outstanding account returns to
    zero, the Payment becomes `paid` and the invoice becomes `paid`.

## 3.c The ledger

Every journal item the trace produces, in posting order. The three manufacturing entries of each
order are separate documents posted in the sequence components, finished goods, labour. They
carry **no counterparty**, because a manufacturing movement belongs to no transfer; the invoice
and payment items carry Ashgrove Fitters. Every amount is in euro and every manufacturing entry
is in the company currency by construction, so the amount in currency column is omitted.

| # | Date | Journal | Entry | Account | Debit | Credit | Reconciled against |
|---|---|---|---|---|---|---|---|
| 1 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` components | 5200 Production Cost | 140.00 | | — |
| 2 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` components | 1400 Inventory | | 140.00 | — |
| 3 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` components | 5200 Production Cost | 490.00 | | — |
| 4 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` components | 1400 Inventory | | 490.00 | — |
| 5 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` finished | 1400 Inventory | 870.00 | | — |
| 6 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` finished | 5200 Production Cost | | 870.00 | — |
| 7 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` labour | 5200 Production Cost | 240.00 | | — |
| 8 | 2026-04-30 | Inventory Valuation | `WH/MO/00021` labour | 5300 Manufacturing Overhead | | 240.00 | — |
| 9 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` components | 5200 Production Cost | 174.00 | | — |
| 10 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` components | 1400 Inventory | | 174.00 | — |
| 11 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` components | 5200 Production Cost | 104.00 | | — |
| 12 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` components | 1400 Inventory | | 104.00 | — |
| 13 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` finished | 1400 Inventory | 368.00 | | — |
| 14 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` finished | 5200 Production Cost | | 368.00 | — |
| 15 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` labour | 5200 Production Cost | 90.00 | | — |
| 16 | 2026-05-18 | Inventory Valuation | `WH/MO/00022` labour | 5300 Manufacturing Overhead | | 90.00 | — |
| 17 | 2026-05-20 | Customer Invoices | `INV/2026/00512` | 4000 Product Sales | | 1 380.00 | — |
| 18 | 2026-05-20 | Customer Invoices | `INV/2026/00512` | 2510 Tax Payable | | 289.80 | — |
| 19 | 2026-05-20 | Customer Invoices | `INV/2026/00512` | 1200 Trade Receivables | 1 669.80 | | item 23 |
| 20 | 2026-05-20 | Customer Invoices | `INV/2026/00512` | 5000 Cost of Goods Sold | 368.00 | | — (injected cost item) |
| 21 | 2026-05-20 | Customer Invoices | `INV/2026/00512` | 1400 Inventory | | 368.00 | — (injected cost item) |
| 22 | 2026-05-22 | Bank | Payment `BNK1/2026/0088` | 1450 Outstanding Receipts | 1 669.80 | | item 25 |
| 23 | 2026-05-22 | Bank | Payment `BNK1/2026/0088` | 1200 Trade Receivables | | 1 669.80 | item 19 |
| 24 | 2026-05-25 | Bank | Transaction `BNK1/2026/00042` | 1010 Bank | 1 669.80 | | — |
| 25 | 2026-05-25 | Bank | Transaction `BNK1/2026/00042` | 1450 Outstanding Receipts | | 1 669.80 | item 22 |

Item 19 carries the maturity date 2026-05-20; no other item carries one.

**Totals.**

```formula
debits  = 140.00 + 490.00 + 870.00 + 240.00 + 174.00 + 104.00 + 368.00 + 90.00
        + 1 669.80 + 368.00 + 1 669.80 + 1 669.80 = 7 853.40
credits = 140.00 + 490.00 + 870.00 + 240.00 + 174.00 + 104.00 + 368.00 + 90.00
        + 1 380.00 + 289.80 + 368.00 + 1 669.80 + 1 669.80 = 7 853.40
```

**Per-account proof.**

| Account | Debits | Credits | Balance after the trace | Meaning |
|---|---|---|---|---|
| 1010 Bank | 1 669.80 | — | 1 669.80 debit | The cash actually received |
| 1200 Trade Receivables | 1 669.80 | 1 669.80 | 0.00 | Nothing owed |
| 1400 Inventory | 1 238.00 | 1 276.00 | 38.00 credit for the trace | The net fall in the goods on hand, part (d) |
| 1450 Outstanding Receipts | 1 669.80 | 1 669.80 | 0.00 | The payment confirmed by the bank |
| 2510 Tax Payable | — | 289.80 | 289.80 credit | 21 % of 1 380.00 |
| 4000 Product Sales | — | 1 380.00 | 1 380.00 credit | The order's untaxed total |
| 5000 Cost of Goods Sold | 368.00 | — | 368.00 debit | The value of the two cabinets that left |
| 5200 Production Cost | 1 238.00 | 1 238.00 | 0.00 | Both productions allocated in full |
| 5300 Manufacturing Overhead | — | 330.00 | 330.00 credit | Labour already expensed elsewhere, now capitalised into goods |

Two invariants are worth reading off this table. First, **5200 Production Cost returns to zero**:
whatever entered the production location as components or labour left it as finished goods.
Second, **5300 Manufacturing Overhead ends with a credit**, not a debit: the wages were expensed
when they were paid, in an entry outside this trace, and the labour entry relieves that expense by
the amount now sitting inside the value of the goods. Gross margin on the sale is
1 380.00 − 368.00 = 1 012.00.

## 3.d Stock consequences

**Quantities per location.**

| Product | `WH/Stock` at the start | After `WH/MO/00021` | After `WH/MO/00022` | After the delivery |
|---|---|---|---|---|
| Steel bracket | 220 | 180 | 180 | 180 |
| Oak panel | 60 | 40 | 40 | 40 |
| Glass front | 30 | 30 | 28 | 28 |
| Shelf unit | 0 | 10 | 8 | 8 |
| Display cabinet | 0 | 0 | 2 | 0 |

`WH/Production` ends holding +40 Steel brackets, +20 Oak panels, +2 Glass fronts, −8 Shelf units
(2 consumed in, 10 produced out) and −2 Display cabinets. A production location is outside the
valued perimeter and is never counted as stock, so those figures are bookkeeping residue, not
goods. `Partners/Customers` ends at −2 Display cabinets.

**Value carried by each completed goods movement, for the products valued first in first out.**
The remaining quantity and the remaining value are derived from the stack, as specified in
[inventory-valuation-and-costing/calculations.md](inventory-valuation-and-costing/calculations.md)
section 5.3.

| Movement | Product | Direction | Quantity | Value | Remaining quantity | Remaining value |
|---|---|---|---|---|---|---|
| B1 (2026-03-16) | Steel bracket | incoming | 120 | 420.00 | 80 | 420.00 × 80 ÷ 120 = 280.00 |
| B2 (2026-04-09) | Steel bracket | incoming | 100 | 380.00 | 100 | 380.00 |
| C1 (2026-04-30) | Steel bracket | outgoing | 40 | 140.00 | 0 | 0.00 |
| G1 (2026-04-14) | Glass front | incoming | 30 | 1 560.00 | 28 | 1 560.00 × 28 ÷ 30 = 1 456.00 |
| C2 (2026-05-18) | Glass front | outgoing | 2 | 104.00 | 0 | 0.00 |
| F2 (2026-05-18) | Display cabinet | incoming | 2 | 368.00 | 0 | 0.00 |
| D1 (2026-05-19) | Display cabinet | outgoing | 2 | 368.00 | 0 | 0.00 |

**Value of the products valued at average cost.** A remaining quantity per movement is a
first-in-first-out notion; under average cost the value on hand is the quantity multiplied by the
stored unit cost.

| Product | Quantity on hand | Stored unit cost | Value |
|---|---|---|---|
| Oak panel | 40 | 24.50 | 980.00 |
| Shelf unit | 8 | 87.00 | 696.00 |

```formula
physical_value = 280.00 + 380.00 + 1 456.00 + 0.00 + 980.00 + 696.00 = 3 792.00
opening_value  = 800.00 + 1 470.00 + 1 560.00 = 3 830.00
ledger_change  = 1 238.00 debit − 1 276.00 credit = −38.00
ledger_value   = 3 830.00 − 38.00 = 3 792.00
difference     = 0.00
```

The 38.00 fall is not a loss: it is 368.00 of cabinets sold less the 330.00 of labour that the two
productions capitalised into the goods still on the shelf.

## 3.e Variations

### 3.e.1 Invoicing policy *ordered quantities* on the Display cabinet

The order line's quantity to invoice is 2 from the instant of confirmation, so the whole order can
be invoiced on 2026-05-11, eight days before anything is built. The revenue and tax items are
unchanged in amount and move to 2026-05-11.

The cost pair, however, **is not produced at all**. At that moment no completed movement is
reachable from the invoice line, so the cost falls back to the first-in-first-out value of the
cost-of-goods-sold quantity; the Display cabinet's stack is empty, so the algorithm extrapolates
with the product's stored unit cost, which is still 0.00:

```formula
cogs_quantity   = 2
fifo_value( 2 ) = 2 × 0.00 = 0.00       ( the stack is empty; the stored unit cost is used )
cogs_unit_price = 0.00
```

The injection is skipped whenever the amount in currency is zero for the document currency **or**
the unit price is zero at the Product Price precision, so no cost item is written.

**Delta against part (c).** Items 17, 18 and 19 move to 2026-05-11; items 20 and 21 disappear.
Account 5000 Cost of Goods Sold ends at 0.00 and account 1400 Inventory ends at 38.00 + 368.00 =
330.00 **debit** for the trace rather than 38.00 credit, holding the value of two cabinets that
are no longer there. The gap is found by the inventory valuation closing, whose part two compares
the physical value with the ledger value and posts the difference as a credit of 368.00 on 1400
Inventory against a debit of 368.00 on 1410 Inventory Variation. The margin of the sale is
therefore reported as 1 380.00 until the closing runs.

### 3.e.2 Costing method *first in first out* on the Shelf unit

With a single production there is only one layer, so the two methods give the same 174.00 for the
two shelf units consumed and there is **no delta**. The methods separate as soon as two
productions at different costs exist.

Suppose a second manufacturing order produced 5 more Shelf units on 2026-05-12 at a total cost of
455.00, that is 91.00 each, before `WH/MO/00022` consumed anything.

```formula
average cost after the second production :
    previous_quantity = 15 − 5 = 10
    unit_cost = ( 10 × 87.00 + 455.00 ) ÷ 15 = 1 325.00 ÷ 15 = 88.333333…
    stored_unit_cost = 88.33

consumption of 2 under average cost      = 2 × 88.33 = 176.66
consumption of 2 under first in first out = 2 × 87.00 = 174.00   ( from the oldest layer )
```

**Delta under average cost.** Items 9 and 10 become 176.66 instead of 174.00. The cabinet's total
cost becomes 176.66 + 104.00 + 90.00 = 370.66, so the finished unit price becomes 185.33, items 13
and 14 become 370.66, the delivery is valued at 370.66 and items 20 and 21 become 370.66. The
production account still closes: 176.66 + 104.00 + 90.00 − 370.66 = 0.00. Gross margin falls by
2.66 to 1 009.34.

### 3.e.3 Costing method *standard price* on the Display cabinet at 180.00

The production cost allocation of
[manufacturing/calculations.md](manufacturing/calculations.md) section 9.3 stops applying to the
finished movement: *if the finished product's costing method is neither first in first out nor
average, the finished movements take the product's standard price*.

```formula
total_cost          = 174.00 + 104.00 + 90.00 = 368.00     ( unchanged )
finished_price_unit = 180.00                                ( the product's own unit cost )
finished_move_value = 2 × 180.00 = 360.00
```

**Delta against part (c).** Items 13 and 14 become 360.00. The delivery is valued at 2 × 180.00 =
360.00, so items 20 and 21 become 360.00. Account 5200 Production Cost ends with
174.00 + 104.00 + 90.00 − 360.00 = **8.00 debit**, not zero: the 8.00 by which the real cost
exceeded the standard cost stays on the production account until it is analysed and cleared by
hand. Had the standard price been above the real cost, the residue would have been a credit
instead. Account 1400 Inventory ends at 1 230.00 debit against 1 268.00 credit, a net 38.00 credit
again, and the physical value of the Display cabinet is 0 × 180.00 = 0.00, so the two still agree.

### 3.e.4 Anglo-saxon against continental accounting

Everything in this trace is governed by the products' **valuation mode**, not by the company-level
anglo-saxon switch. The switch changes only the price-difference items injected into a vendor
bill, which this trace does not raise; turning it off changes not one item of part (c).

Setting the categories' valuation mode to **periodic** changes almost everything:

- The condition of
  [inventory-valuation-and-costing/accounting-effects.md](inventory-valuation-and-costing/accounting-effects.md)
  section 1 fails at its fifth clause for every movement, so items 1 to 6 and 9 to 14 disappear:
  no component entry, no finished-goods entry.
- The labour entry's first precondition — *the finished product's valuation mode is automated* —
  also fails, so items 7, 8, 15 and 16 disappear too. The labour therefore stays where it was
  originally expensed and is never capitalised.
- No cost pair is injected into the invoice, so items 20 and 21 disappear.
- What remains is items 17, 18, 19 and 22 to 25: revenue, tax, receivable, payment and bank
  transaction.
- The inventory valuation closing run on 2026-05-31 posts part two of its entry. Under periodic
  valuation the opening components were expensed by their vendor bills, so account 1400 Inventory
  holds nothing:

  ```formula
  balance = 3 792.00 − 0.00 − 0.00 = 3 792.00
  ```

  a debit of 3 792.00 on 1400 Inventory against a credit of 3 792.00 on 1410 Inventory Variation.
  Note that the periodic physical value of 3 792.00 still contains the 330.00 of capitalised
  labour, because the movement values are computed whether or not they are posted; the closing is
  what carries them into the balance sheet.

### 3.e.5 A foreign-currency order

The manufacturing entries never carry a foreign currency: the value of a movement is already in
the company currency, and both the goods-movement entry and the labour entry are written in it.
Items 1 to 16 are therefore identical whatever the customer's currency.

The sales side changes. The same order written in United States dollars, with a rate of 1.0850
dollars for one euro on 2026-05-20:

| Item | Account | Amount in currency (dollar) | Debit (euro) | Credit (euro) |
|---|---|---|---|---|
| 17 | 4000 Product Sales | −1 380.00 | | 1 271.89 |
| 18 | 2510 Tax Payable | −289.80 | | 267.10 |
| 19 | 1200 Trade Receivables | +1 669.80 | 1 538.99 | |

```formula
round_to_currency( 1 380.00 ÷ 1.0850 ) = round_to_currency( 1 271.889401 ) = 1 271.89
round_to_currency(   289.80 ÷ 1.0850 ) = round_to_currency(   267.096774 ) = 267.10
company_total  = 1 271.89 + 267.10 = 1 538.99
effective_rate = 1 669.80 ÷ 1 538.99 = 1.085001…
instalment_company  = 1 538.99      ( the single line is the balance line )
instalment_currency = 1 669.80
```

Both columns close: euro debits 1 538.99 against euro credits 1 271.89 + 267.10 = 1 538.99; dollar
amounts −1 380.00 − 289.80 + 1 669.80 = 0.00.

**The cost pair follows the compatibility finding of
[variation 1.e.5](#1e5-a-foreign-currency-order).** Its amount in currency is written as
quantity × the cost-of-goods-sold unit price, and that unit price is a company-currency figure, so
the balance becomes round_to_currency( 368.00 ÷ 1.0850 ) = 339.17 euro. Account 1400 Inventory is
credited 339.17 instead of the 368.00 of value that actually left, and the difference of 28.83
surfaces at the inventory valuation closing. The corrected behaviour is the one stated in trace 1:
write the cost items' balance directly as the company-currency cost and derive the amount in
currency from it.

### 3.e.6 Cash rounding to the nearest five hundredths

```formula
rounded_total = round_to_step( 1 669.80 , 0.05 , half away from zero ) = 1 669.80
difference    = 0.00
```

The total is already a multiple of five hundredths, so **no rounding item is produced**. Give the
line a discount of 3 % and the arithmetic bites:

```formula
discounted_unit_price = 690.00 × ( 1 − 3 ÷ 100 ) = 669.30
line_untaxed = round_to_currency( 2 × 669.30 ) = 1 338.60
line_tax     = round_to_currency( 1 338.60 × 21 ÷ 100 ) = round_to_currency( 281.106 ) = 281.11
grand_total  = 1 619.71
rounded_total = round_to_step( 1 619.71 , 0.05 , half away from zero ) = 1 619.70
difference    = 1 619.70 − 1 619.71 = −0.01
```

The rounding item **reduces** what is owed, so the **loss** account is used. On a customer invoice
the untaxed side is a credit, so the reduction is a debit.

**Delta against part (c).** Item 17 becomes a credit of 1 338.60; item 18 becomes a credit of
281.11; one new item appears, a debit of 0.01 on 6580 Cash Rounding Loss; item 19 becomes a debit
of 1 619.70. Totals: credits 1 338.60 + 281.11 = 1 619.71 against debits 1 619.70 + 0.01 =
1 619.71. The cost items are untouched: cash rounding never changes what the goods cost. Note that
a discount also suppresses the price-difference mechanism on the purchase side, which is why the
two features are usually described together.

### 3.e.7 An early payment discount of two per cent within ten days

The term becomes one line of 100 % at 30 days carrying a discount of 2 % within 10 days, in the
computation mode *On early payment*. `INV/2026/00512` then carries one receivable item of 1 669.80
maturing 2026-06-19, stamped with a discount deadline of 2026-05-30 and a discounted amount of

```formula
discount_amount = 1 669.80 × 2 ÷ 100 = 33.396
amount_due      = round_to_currency( 1 669.80 − 33.396 ) = round_to_currency( 1 636.404 ) = 1 636.40
```

The customer pays 1 636.40 on 2026-05-26, inside the deadline, and the register-payment assistant
adds the write-off items to the **payment's own** entry so that the receivable clears exactly:

| Account | Label | Debit | Credit |
|---|---|---|---|
| 1450 Outstanding Receipts | `INV/2026/00512` | 1 636.40 | |
| 6900 Cash Discount Granted | `Early Payment Discount` | 27.60 | |
| 2510 Tax Payable | `Early Payment Discount (Sales 21 %)` | 5.80 | |
| 1200 Trade Receivables | `INV/2026/00512` | | 1 669.80 |

```formula
net_discount = round_to_currency( 1 380.00 × 2 ÷ 100 ) = round_to_currency( 27.60 ) = 27.60
tax_on_it    = round_to_currency( 27.60 × 21 ÷ 100 ) = round_to_currency( 5.796 ) = 5.80
27.60 + 5.80 = 33.40 = 1 669.80 − 1 636.40
```

**Delta against part (c).** Item 19's maturity becomes 2026-06-19 and it gains a discount deadline
and a discounted amount; items 22 and 23 are replaced by the four items above; items 24 and 25
become 1 636.40. Nothing in the manufacturing half of the trace changes.

### 3.e.8 Several companies

A manufacturing order, its components, its finished goods, its operation type and its locations
must all belong to one company: the confirmation begins with a company consistency check across
every record the order references, and a component whose product belongs to another company is
refused there. The production account, the inventory journal and the work centre's expense account
are all resolved **in the company of the record being posted**, so a group with two manufacturing
companies keeps two production accounts and two sets of entries, one per company, with no
inter-company item.

Building in one company for another is not a variation of this trace: it is either subcontracting,
where the subcontractor's location replaces the production location, or inter-company trade, where
one company sells the finished good to the other. The latter is traced in
[trace 9](#9-drop-shipping-and-inter-company-trade).

### 3.e.9 An extra unit cost on the recipe

Set the Shelf unit recipe's extra unit cost to 3.00 and nothing else changes in the inputs.

```formula
extra_cost          = 3.00 × 10 = 30.00
total_cost          = 630.00 + 240.00 + 30.00 = 900.00
finished_price_unit = 900.00 ÷ 10 = 90.00
finished_move_value = 900.00
```

**Delta against part (c).** Items 5 and 6 become 900.00. Account 5200 Production Cost ends the
first order at 140.00 + 490.00 + 240.00 − 900.00 = **30.00 credit**, exactly the extra cost, and
it stays there: the extra unit cost is capitalised into the finished product **without any
counterpart entry**, which is the documented behaviour of the account. The Shelf unit's unit cost
becomes 90.00, so the second order's component value becomes 2 × 90.00 = 180.00, its total cost
180.00 + 104.00 + 90.00 = 374.00, its finished unit price 187.00, and items 9, 10, 13, 14, 20 and
21 move accordingly. Account 1400 Inventory ends 30.00 higher than in part (c), against a
production account 30.00 in credit; a rebuild should surface that residue in the production
account's own report.

## 3.f Failure points

Each entry gives the step, the condition, the domain that refuses and the exact text.

**Step 2, no rule reaches the location** (replenishment and procurement). The warehouse route
filter drops the Manufacture route when the product has no bill of materials of type `normal`, so
a product configured to be manufactured but given no recipe fails here rather than later. The
message is two lines.

`No rule has been found to replenish "<the product display name>" in "<the location display name>".`

`Verify the routes configuration on the product.`

**Step 2, a rule chain that never terminates** (replenishment and procurement). Raised when the
same rule is met twice while walking the chain from a reordering rule's location.

`Invalid rule's configuration, the following rule causes an endless loop: <the rule display name>`

**Step 3, a recipe whose finished product is also one of its by-products** (manufacturing). The
two spaces before the word *as* are part of the shipped text.

`You cannot have <the product name>  as the finished product and in the Byproducts`

**Step 4, company inconsistency** (manufacturing). The confirmation begins with a company
consistency check between the order and its product, operation type, locations and movements; the
message belongs to the platform's shared consistency check and is specified in
[manufacturing/business-rules.md](manufacturing/business-rules.md) section 11.3.

**Step 7, several orders need a lot or serial number and have none** (manufacturing). A single
such order does not raise: the system opens the lot-generation flow for it instead.

`You need to generate Lot/Serial Number(s) to mark as done some productions`

**Step 7, generating a second lot for a lot-tracked finished product** (manufacturing).

`You cannot set more than 1 lot per product`

**Step 7, saving more than one producing lot** (manufacturing).

`You cannot set more than 1 lot`

**Step 7, generating a number for a product with no sequence** (manufacturing).

`Please set the first Serial Number or a default sequence`

**Step 7, the consumption check finds a difference** (manufacturing). No refusal text: the
Consumption Warning assistant opens, listing one row per offending product with the consumed and
expected quantities and the policy. Under the policy `warning` any manufacturing user may confirm;
under `strict` only a manufacturing administrator may, and a plain user's only alternatives are to
reset the quantities to the expected values or to abandon the closing.

**Step 7, an expired lot among the components** (manufacturing), when expiry tracking is
installed. For a single expired lot the assistant reads

`You are going to use the component <the product display name>, <the lot name> which is expired.\nDo you confirm you want to proceed?`

and for several

`You are going to use some expired components.\nDo you confirm you want to proceed?`

**Step 8, an incoming or outgoing movement of a product whose value is kept per lot, when a detail
line carries no lot** (inventory valuation and costing).

`A lot/serial number is required for product '<the product display name>' as it has lot valuation enabled.`

**Step 9, the inventory journal is not configured** (inventory valuation and costing). Raised by
the closing operation rather than by the movement entry, which simply has no journal to post in.

`Please set the Journal for Inventory Valuation in the settings.`

**Unplanning an order whose work has begun** (manufacturing), one message per case.

`Some work orders are already done, so you cannot unplan this manufacturing order. It'd be a shame to waste all that progress, right?`

`Some work orders have already started, so you cannot unplan this manufacturing order. It'd be a shame to waste all that progress, right?`

**Cancelling or deleting a closed order** (manufacturing), one message per case.

`You cannot cancel a manufacturing order that is already done.`

`You cannot delete a manufacturing order that is already done.`

`Cannot delete a manufacturing order in done state.`

`<the comma-separated display names> cannot be deleted. Try to cancel them before.`

**Moving the dates of a closed or cancelled order** (manufacturing).

`You cannot move a manufacturing order once it is cancelled or done.`

**Splitting or merging outside draft and confirmed** (manufacturing), one message per case; the
word in italics is *split* or *merged* according to the operation.

`Only manufacturing orders in either a draft or confirmed state can be split / merged.`

`Only manufacturing orders with a Bill of Materials can be split / merged.`

`Unable to split with more than the quantity to produce.`

**Setting a scrap or adjustment location as the destination of a manufacturing operation type**
(manufacturing).

`You cannot set a scrap location as the destination location for a manufacturing type operation.`

**Step 11, confirming a sales order whose status is not `draft` or `sent`** (sales).

`Some orders are not in a state requiring confirmation.`

**Step 11, confirming a sales order with a line that has no product** (sales).

`Some order lines are missing a product, you need to correct them before going further.`

**Step 16, validating an empty transfer, a zero-quantity transfer or a tracked product with no
lot** (inventory operations), one message per case; the line break in the second is part of the
message.

`You can’t validate an empty transfer. Please add some products to move before proceeding.`

`Transfer trouble alert! Validating a zero quantity transfer? You're not moving invisible goods around are you?\nSet some quantities and let's get moving!`

`You need to supply a Lot/Serial number for products <the comma-separated product display names>.`

**Step 17, invoicing when nothing is invoiceable** (sales). Raised when the run produced no header
values at all, which under the delivered-quantities policy means nothing has shipped. The message
is multi-line and is reproduced whole:

```
Cannot create an invoice. No items are available to invoice.

To resolve this issue, please ensure that:
   • The products have been delivered before attempting to invoice them.
   • The invoicing policy of the product is configured correctly.

If you want to invoice based on ordered quantities instead:
   • For consumable or storable products, open the product, go to the 'General Information' tab and change the 'Invoicing Policy' from 'Delivered Quantities' to 'Ordered Quantities'.
   • For services (and other products), change the 'Invoicing Policy' to 'Prepaid/Fixed Price'.
```

**Step 17, posting failures** (accounts receivable). The full list is the one given in
[trace 1's failure points](#1f-failure-points): no customer, a negative total, no accountable
line, a document that is not draft, an archived account, journal or currency, and an unbalanced
entry.

**Step 17, a missing account on the cost pair.** No message is raised. When the inventory
valuation account or the expense counterpart cannot be resolved for a line, that line is **skipped**
and the invoice posts with revenue, tax and a receivable but no cost recognition.

**Step 18, registering a payment when the payment method line has no outstanding account**
(payments and bank reconciliation).

`You can't create a new payment without an outstanding payments/receipts account set either on the company or the <the payment method name> payment method in the <the journal display name> journal.`

**Step 19, creating a bank transaction in a journal with no suspense account** (payments and bank
reconciliation).

`You can't create a new statement line without a suspense account set on the <the journal display name> journal.`
