# Purchasing

This domain specifies everything the platform does between the moment a buyer decides that
goods or services must be obtained from an outside party and the moment the resulting vendor
bill is matched against what was ordered and what was received.

It is written so that an engineering team can rebuild the behaviour in any programming
language with behavioural equivalence: the same records, the same state transitions, the same
numbers, the same validation messages, the same documents and the same effects on inventory
and on the ledger.

## Conventions used throughout this folder

- **Reproduced identifiers.** Storage names, transport names, column names, route paths and
  selection values are reproduced exactly in code font — for example `partner_id` (the vendor
  reference field), `purchase.order` (the purchase order entity), `amount_untaxed` (the untaxed
  amount field) — because external contracts depend on them. Each one is accompanied by its
  full name in words on first use in a document.
- **Reproduced user-visible text.** Status labels, button labels, message-thread subtype names,
  notification bodies and error messages are reproduced exactly as the system produces them,
  because a rebuilt implementation must produce the same text. A few of those shipped strings
  contain the short form of *request for quotation*; that short form appears here only inside
  such a reproduced string, never in the specification's own prose. Runtime placeholders inside
  a reproduced message are written in words — *the order name*, *the product name*, *the
  company name*.
- **Numbers.** Every worked example states its rounding and its precision. Currency amounts are
  shown with the two decimals of the example currency.

## Scope

The purchasing domain owns:

- **Requests for quotation and purchase orders.** One entity, Purchase Order, covers both:
  a draft document is a request for quotation, a confirmed document is a purchase order. The
  full lifecycle is draft, sent, to approve, purchase, cancelled, with an optional double
  validation step governed by a per-company monetary threshold, and an independent lock flag.
- **Purchase order lines.** Quantity, unit of measure, unit price, discount percentage,
  taxes, expected arrival date, analytic distribution, sections, subsections and notes,
  down-payment lines, and the derived received and billed quantities.
- **Vendor price learning.** Automatic creation of vendor pricelist entries when an order is
  confirmed with a vendor who was not yet registered as a seller of the product.
- **Receipts.** Creation of incoming transfers and stock moves when an order is confirmed,
  their re-synchronisation when lines change, cancellation propagation, return handling, and
  the effect of receipts on the received quantity of each line.
- **Vendor bill creation and matching.** Both billing control policies (on ordered
  quantities and on received quantities), the quantity-to-bill formulas, grouping of several
  orders into one bill, down payments, refunds, the bill-to-order matching view, the
  automatic matching of an incoming electronic or scanned bill against open orders, and the
  three-way comparison between ordered, received and billed quantities.
- **Purchase agreements.** Blanket orders and purchase templates, their agreement lines with
  agreed prices and quantities, the vendor pricelist entries a confirmed blanket order
  creates, and the generation of requests for quotation from an agreement.
- **Calls for tenders.** Competing requests for quotation created as alternatives of one
  another, their grouping, the side-by-side line comparison with best price, best unit price
  and best arrival date highlighting, the clearing of losing quantities, and the
  confirm-time question about what to do with the losing alternatives.
- **Vendor reminders and acknowledgement.** The scheduled reminder that asks a vendor to
  confirm the promised arrival date a configurable number of days in advance, the
  acknowledgement flag, and the portal pages through which a vendor updates arrival dates.
- **Purchase reporting.** The purchase analysis entity, the vendor delay entity, the on-time
  delivery rate, and the buyer dashboard counters.
- **Cross-domain hooks.** Service products on a sales order that generate a request for
  quotation, kits received through a purchase order, repair-driven purchasing, analytic
  distribution on purchase lines, project links, product grids for variant-heavy products,
  and structured electronic order documents.

## What this domain does not own

- The tax engine itself, including how a tax amount is computed from a base amount and how
  a fiscal position maps taxes. See [`../taxes/calculations.md`](../taxes/calculations.md).
- The vendor bill entity itself, its posting, its payment state and its numbering. See
  [`../accounts-payable/README.md`](../accounts-payable/README.md).
- The generic stock transfer, stock move, reservation and valuation machinery. See
  [`../inventory-operations/README.md`](../inventory-operations/README.md) and
  [`../inventory-valuation-and-costing/README.md`](../inventory-valuation-and-costing/README.md).
- The procurement rules that decide *when* to buy (reordering rules, the scheduler, the buy
  rule, lead-time arithmetic). See
  [`../replenishment-and-procurement/README.md`](../replenishment-and-procurement/README.md).
  This domain describes only the request-for-quotation record that the buy rule produces.
- The vendor pricelist entity (the seller record) and its selection algorithm. See
  [`../pricing-and-pricelists/README.md`](../pricing-and-pricelists/README.md). This domain
  describes only how a purchase order line consumes the selected seller.
- Units of measure and their conversion arithmetic. See
  [`../units-of-measure-and-packaging/README.md`](../units-of-measure-and-packaging/README.md).

## Entities owned by this domain

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Purchase Order | `purchase.order` | `purchase_order` | A request for quotation that becomes a purchase order when confirmed; the header holding vendor, currency, dates, totals, status and links to receipts and bills. |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | One ordered product with quantity, unit, price, discount, taxes, expected arrival, and the derived received and billed quantities; also carries sections, subsections, notes and down payments. |
| Purchase Agreement | `purchase.requisition` | `purchase_requisition` | A blanket order (a negotiated fixed price with one vendor over a validity period) or a purchase template (a reusable list of products and quantities). |
| Purchase Agreement Line | `purchase.requisition.line` | `purchase_requisition_line` | One product of an agreement with an agreed quantity and an agreed unit price; for a blanket order it also produces a vendor pricelist entry. |
| Alternative Order Creation Assistant | `purchase.requisition.create.alternative` | transient | Short-lived record that creates one competing request for quotation per selected vendor, optionally copying the products of the originating request. |
| Alternative Order Warning Assistant | `purchase.requisition.alternative.warning` | transient | Short-lived record shown when a request for quotation that has open alternatives is confirmed, offering to keep or to cancel those alternatives. |
| Purchase Analysis Entry | `purchase.report` | database view `purchase_report` | Read-only analytical row, one per purchase order line group, exposing quantities, amounts, delays and dimensions for reporting. |
| Vendor Delay Entry | `vendor.delay.report` | database view `vendor_delay_report` | Read-only analytical row measuring, per vendor and product, the quantity received on time against the quantity received in total. |
| Purchases and Bills Union Entry | `purchase.bill.union` | database view `purchase_bill_union` | Read-only union of posted vendor bills and open purchase orders used as the source list of the auto-complete control on a vendor bill. |
| Purchase and Bill Line Match Entry | `purchase.bill.line.match` | database view `purchase_bill_line_match` | Read-only union of billable purchase order lines and unlinked vendor bill lines used by the matching screen. |
| Bill To Purchase Order Assistant | `bill.to.po.wizard` | transient | Short-lived record used to add selected vendor bill lines to a new or existing purchase order, or to convert them into down payments. |
| Alternative Order Line Comparison Entry | `purchase.order.group` | `purchase_order_group` | Link record grouping several purchase orders that are alternatives of one another for comparison. |

Entities extended by this domain but owned elsewhere are listed at the top of
[`entities.md`](entities.md).

## Reading order

1. [`glossary.md`](glossary.md) — every term used below, defined in full. Read first if any
   term is unfamiliar.
2. [`entities.md`](entities.md) — the data model: every field, every default, every computed
   value, every relation, every uniqueness rule.
3. [`state-machines.md`](state-machines.md) — the status fields and their transitions.
4. [`calculations.md`](calculations.md) — every formula with rounding and worked examples.
5. [`workflows.md`](workflows.md) — the end-to-end operational sequences.
6. [`business-rules.md`](business-rules.md) — validations, constraints, permissions, locking.
7. [`accounting-effects.md`](accounting-effects.md) — what this domain does and does not post
   to the ledger, and exactly which values it hands to the other domains that do.
8. [`configuration.md`](configuration.md) — settings, sequences, groups, access rights,
   record rules, scheduled jobs.
9. [`interfaces.md`](interfaces.md) — menus, screens, remote operations, routes, printable
   documents, message templates, import and export.
10. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered scenarios with concrete
    numbers that a rebuilt implementation must satisfy.

## Dependencies on other domains

| Depends on | What is used |
|---|---|
| Products and catalog | Product templates and variants, the purchasable flag, the purchase description, the control policy, the warning message, product categories. |
| Pricing and pricelists | The vendor pricelist record (the seller), its selection algorithm, its price, its discount, its minimum quantity, its lead time and its validity dates. |
| Units of measure and packaging | Unit conversion of quantities and of prices between the purchase unit, the product unit and the vendor unit. |
| Taxes | Tax computation on order lines, tax-included price correction, fiscal position mapping. |
| Multi-currency | Currency of the order, the order rate, and conversion of vendor prices and of amounts to the company currency. |
| Accounts payable | The vendor bill entity into which purchase order lines are pushed, its posting and its payment. |
| General ledger | Journals and accounts used by the resulting bill and by the stock interim postings. |
| Analytic accounting | The analytic distribution carried by a purchase order line and copied to the bill line. |
| Inventory operations | Incoming transfers, stock moves, operation types, vendor locations, returns. |
| Inventory valuation and costing | The unit price handed to each incoming stock move and the price difference handling at billing time. |
| Replenishment and procurement | The buy rule that creates requests for quotation, reordering rules, lead times, the procurement group. |
| Messaging and activities | The chatter thread on every order, the tracked fields, the reminder message, the activities raised when quantities or dates change. |
| Sales | Service products configured to be re-purchased, and the link back from a purchase line to the originating sales line. |
| Manufacturing | Kits received through a purchase order, and subcontracting receipts. |
| Repair and maintenance | Purchasing of parts consumed by a repair order. |
| Projects and tasks | The project counter of purchase orders reaching an analytic account. |
| Electronic invoicing and document exchange | The structured purchase order document attached to the printed order. |
