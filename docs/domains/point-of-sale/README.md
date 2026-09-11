# Point of Sale

## 1. Purpose of this domain

The point of sale domain covers the counter-selling capability of the platform: a
touch-oriented selling application that runs in a browser, keeps working when the network
is unavailable, records sales against a cash drawer, and at the end of the working period
turns the whole period's trading into a single balanced accounting document.

Three characteristics distinguish this domain from the ordinary sales domain:

1. **The selling application is a client-side replica of the server's pricing and tax
   engine.** Prices, discounts, taxes, combos, rounding and totals are all computed in the
   browser while the cashier works, and the same numbers must be reproduced exactly by the
   server when the order is saved. Any divergence produces an unbalanced closing entry.
2. **Trading is grouped into sessions.** A session is the unit of cash accountability and
   the unit of accounting posting. Orders are not individually posted to the general
   ledger; instead they are accumulated and posted once, at session closing, unless the
   customer asked for an invoice.
3. **Operation is offline-tolerant.** All master data needed to sell (products, prices,
   taxes, partners, payment methods, categories, fiscal positions, rounding rules) is
   downloaded when the session opens, orders are held locally until they can be
   transmitted, and the transmission protocol is idempotent on a client-generated
   universally unique identifier.

## 2. Capabilities covered

| Area | What is covered |
| --- | --- |
| Configuration | Every setting of a point of sale configuration: journals, payment methods, pricelists, fiscal positions, rounding, receipt headers and footers, category restriction, tips, ship-later, presets, printers, customer display, trusted configurations, sequences |
| Session lifecycle | Opening control, opening cash count, trading, closing control, closing cash count, validation, posting, rescue sessions, forced closing with a balancing line |
| Session closing entry | The complete line-by-line specification of the accounting document produced when a session closes: sales aggregation keys, tax lines, receivable lines per payment method, cash statement lines, cash difference, rounding lines, cost of goods sold and stock valuation lines, the treatment of invoiced orders, and every reconciliation performed |
| Orders | Creation, transmission, payment, refunds, invoicing, tips, ship later, cancellation, editing tracking, tracking numbers, receipt numbering |
| Order lines | Pricing, discounts, taxes, combos, attributes, custom values, lots and serial numbers, notes, refund linkage, cost and margin |
| Payments | Payment methods of the three kinds (cash, bank, pay later), terminal integrations, quick response code payments, change, split payments, online payments |
| Pricing and taxes in the client | The rounding contract that must hold between the browser computation and the server computation |
| Cash rounding | Rounding of the payable amount to a cash denomination, with both application scopes and both rounding methods |
| Inventory | The delivery documents created immediately or at session closing, lot capture, ship later through procurement rules |
| Receipts and reports | Receipt content, the sales details report, the order analysis report, the invoice document |
| Restaurant extension | Floors, tables, order transfer, bill splitting, course/preparation printing, order-change tracking |
| Self-ordering extension | Kiosk and mobile self-ordering, self-order presets, online payment of self orders, table identification |
| Employee extension | Employee login and employee-scoped permissions inside the selling application |
| Sales-order extension | Settling an existing sales order at the counter, down payments |
| Loyalty extension | Loyalty programs, coupons and rewards applied at the counter |
| Discount extension | A global discount button |
| Synchronization | The data loaded at session opening, the change feed, the idempotent order transmission, offline storage, conflict rules |

## 3. Entities of this domain

Entities are listed with their transport name (the name used on the wire by the remote
operation layer) and their storage table.

### 3.1 Core entities

| Entity | Transport name | Table | One-line purpose |
| --- | --- | --- | --- |
| Point of Sale Configuration | `pos.config` | `pos_config` | One physical or logical till: its journals, payment methods, devices, pricing and behavioral settings |
| Point of Sale Session | `pos.session` | `pos_session` | One trading period of one configuration; the unit of cash accountability and of accounting posting |
| Point of Sale Order | `pos.order` | `pos_order` | One sale, refund or mixed transaction recorded at the counter |
| Point of Sale Order Line | `pos.order.line` | `pos_order_line` | One product line of an order, with its price, discount, taxes and cost |
| Point of Sale Payment | `pos.payment` | `pos_payment` | One tender applied to an order, including change given back |
| Point of Sale Payment Method | `pos.payment.method` | `pos_payment_method` | A way of being paid: cash, bank (card/terminal/quick response code) or customer account |
| Point of Sale Category | `pos.category` | `pos_category` | A merchandising category tree used to lay out the product buttons |
| Point of Sale Pack Operation Lot | `pos.pack.operation.lot` | `pos_pack_operation_lot` | A lot or serial number captured on an order line |
| Point of Sale Bill | `pos.bill` | `pos_bill` | A coin or banknote denomination offered as a shortcut on the payment screen |
| Point of Sale Note | `pos.note` | `pos_note` | A predefined note that can be attached to an order or a line |
| Point of Sale Preset | `pos.preset` | `pos_preset` | A named bundle of service settings (pricelist, fiscal position, identification requirement, return mode, time slots) |
| Point of Sale Printer | `pos.printer` | `pos_printer` | A preparation printer and the categories it prints |
| Stock Reference | `stock.reference` | `stock_reference` | A grouping token that links deferred delivery documents back to their originating counter orders |

### 3.2 Reporting and wizard entities

| Entity | Transport name | Purpose |
| --- | --- | --- |
| Point of Sale Order Analysis | `report.pos.order` | A read-only analytical view over orders and lines used for pivots and graphs |
| Point of Sale Details Wizard | `pos.details.wizard` | Collects a date range and a set of configurations, then renders the sales details report |
| Point of Sale Daily Sales Reports Wizard | `pos.daily.sales.reports.wizard` | Renders the sales details report for one session |
| Point of Sale Payment Wizard | `pos.make.payment` | Registers a payment on an order from the administrative interface |
| Point of Sale Invoice Wizard | `pos.make.invoice` | Creates one invoice, or one invoice per order, for a selection of orders |
| Point of Sale Close Session Wizard | `pos.close.session.wizard` | Offers a forced close with a balancing account when the closing entry does not balance |
| Point of Sale Confirmation Wizard | `pos.confirmation.wizard` | A generic confirm-or-cancel dialog used by administrative actions |

### 3.3 Entities contributed by extension packages

| Entity | Transport name | Contributing capability |
| --- | --- | --- |
| Restaurant Floor | `restaurant.floor` | Restaurant: a named seating area with a background and a set of tables |
| Restaurant Table | `restaurant.table` | Restaurant: a table with a number, a shape, a position and a seat count |
| Point of Sale Self Order Custom Link | `pos_self_order.custom_link` | Self-ordering: an extra navigation button shown on the self-ordering landing page |

Entities defined elsewhere but extended by this domain (their base definition belongs to
the referenced domain, only the point-of-sale-specific additions are described here):
Journal Entry and Journal Item, Bank Statement Line, Accounting Payment, Cash Rounding,
Fiscal Position, Journal, Account, Tax, Tax Group, Currency, Product Template and Product
Variant, Product Category, Product Combo and Combo Item, Pricelist and Pricelist Item,
Partner, User, Company, Transfer (delivery document), Warehouse, Unit of Measure, Barcode
Rule, Language, Country and Country State, Decimal Precision, Digest.

## 4. Reading order

1. `README.md` (this file) — scope and vocabulary.
2. [`glossary.md`](glossary.md) — every term used, defined in full. Read this before the
   detailed files if the vocabulary of counter selling is unfamiliar.
3. [`entities.md`](entities.md) — the complete data model: every field, every default,
   every computed value, every relation, every uniqueness rule.
4. [`state-machines.md`](state-machines.md) — the session state machine, the order state
   machine, the payment status values, the transfer and invoice statuses, with guards and
   side effects.
5. [`calculations.md`](calculations.md) — every formula: price, discount, tax, rounding,
   change, cash balance, cost, margin, currency conversion.
6. [`workflows.md`](workflows.md) — the end-to-end procedures: opening a session, selling,
   refunding, invoicing, closing, rescuing, splitting a bill, self-ordering.
7. [`accounting-effects.md`](accounting-effects.md) — **the critical file.** The session
   closing entry specified line by line, plus every other accounting document the domain
   produces.
8. [`business-rules.md`](business-rules.md) — every validation, constraint, permission
   check and error message.
9. [`configuration.md`](configuration.md) — settings, parameters, sequences, shipped
   default records, security groups, access rights, record rules, scheduled jobs.
10. [`interfaces.md`](interfaces.md) — menus, views, named remote operations, routes,
    reports, templates and integration contracts.
11. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered Given/When/Then
    scenarios with concrete numbers, including the eight mandatory scenarios.

## 5. Dependencies on other domains

| Domain | What this domain relies on |
| --- | --- |
| [General Ledger](../general-ledger/README.md) | Journals, accounts, journal entries and journal items, posting, reconciliation, lock dates |
| [Accounts Receivable](../accounts-receivable/README.md) | Customer invoices and credit notes created from counter orders, cash rounding on invoices, payment terms |
| [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) | Accounting payments, outstanding accounts, bank statement lines, reconciliation plans |
| [Taxes](../taxes/README.md) | The tax computation engine, price-included taxes, fiscal position mapping, tax repartition and tax tags |
| [Multi-Currency](../multi-currency/README.md) | Currency rounding, conversion of session amounts to company currency, conversion rates by date |
| [Products and Catalog](../products-and-catalog/README.md) | Products, variants, attributes, combos, barcodes, product categories, income and expense accounts |
| [Pricing and Pricelists](../pricing-and-pricelists/README.md) | Pricelists and rules, the price computation algorithm replicated in the selling application |
| [Inventory Operations](../inventory-operations/README.md) | Transfers, moves, move lines, lots and serial numbers, operation types, warehouses |
| [Inventory Valuation and Costing](../inventory-valuation-and-costing/README.md) | The cost of a sold unit, valuation layers, the interim accounts used for cost of goods sold |
| [Units of Measure and Packaging](../units-of-measure-and-packaging/README.md) | Unit conversion of sold quantities |
| [Sales](../sales/README.md) | Settling a sales order at the counter, down payments |
| [Loyalty and Promotions](../loyalty-and-promotions/README.md) | Loyalty programs, coupons and rewards evaluated inside the selling application |
| [Payment Providers](../payment-providers/README.md) | Online payment of self-ordered and pay-later transactions |
| [Messaging and Activities](../messaging-and-activities/README.md) | Order and session chatter, receipt emails, the stale-session activity |

## 6. What this domain does *not* cover

- The generic invoice model, its numbering and its printing — see the accounts receivable
  domain. Only the counter-specific invoice preparation is described here.
- The generic tax engine — see the taxes domain. Only the counter-specific calls into it
  and the rounding contract are described here.
- The generic reconciliation mechanics — see the general ledger domain. Only which lines
  this domain reconciles against which is described here.
- Hardware drivers, printer firmware and card-reader protocols below the contract level.

## 7. Conventions used in the detailed files

- **Storage names** appear in code font and are always accompanied, on first use in each
  file, by their full name in words: for example `amount_total` (the order total including
  tax).
- **Amounts in the selling currency** means amounts expressed in the currency of the point
  of sale configuration. **Amounts in company currency** means amounts converted with the
  rate applicable at the relevant date. Every accounting line carries both when the two
  currencies differ.
- **Rounding to currency** means rounding to the rounding step of the currency (its
  smallest representable increment), using the round-half-away-from-zero method, unless
  explicitly stated otherwise.
- **The closing entry** always means the single accounting document created by the session
  validation procedure and referenced by the session.
- **Signed quantities**: a refund line carries a negative quantity. The sign conventions
  are stated explicitly wherever they matter.
