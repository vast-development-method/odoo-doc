# Sales

This domain specifies the complete sell-side order management capability of the system: the
quotation that is prepared for a customer, the sales order it becomes when the customer accepts it,
everything the confirmation triggers in other domains, the invoicing of the order (including advance
invoices and their later deduction), the customer-facing acceptance flow (view, sign, pay, decline),
the sales organisation (teams and salespeople), and the analytical reporting built on top of the
orders.

The domain is the hub of the order-to-cash chain. It does not, by itself, move stock, post journal
entries, or take money. Instead it creates the documents that other domains act upon: procurement
requests, transfers, customer invoices, projects and tasks, purchase requests, payment transactions.
Every one of those hand-offs is specified here field by field, so that a re-implementation produces
the same downstream records.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Quotation authoring | Create a priced offer for a customer with lines, sections, subsections, notes, optional lines, discounts, taxes, delivery address, payment terms, validity date. |
| Quotation templates | Pre-built line sets, terms and conditions, validity duration, signature and payment requirements, confirmation mail, invoicing journal; optional sections that the customer may add from the customer portal. |
| Product configuration on a line | Variant attributes, attributes that do not create variants, custom attribute values, product combos with proportional price allocation, optional (linked) lines, grid entry of a variant matrix. |
| Pricing on a line | Price from the price list rule, discount derived from the rule, manual override protection, tax mapping through the fiscal position, price recomputation actions. |
| Order state machine | Quotation → Quotation Sent → Sales Order → Cancelled, plus an orthogonal lock flag, plus an expiration derived from the validity date. |
| Confirmation | A single algorithm that validates the order, stamps the confirmation date, triggers every downstream document creation, optionally locks the order, and optionally sends the confirmation message. |
| Customer portal acceptance | Viewing the quotation with an access token, electronic signature, online payment of the whole amount or of a prepayment percentage, declining with a reason, downloading attached product documents and the structured order document. |
| Invoicing | Regular invoices from ordered or delivered quantities, advance invoices as a percentage or as a fixed amount, deduction of advance invoices on the final invoice, grouping of several orders onto one invoice, refunds when the computed invoice is negative, invoice status per line and per order including the upselling state. |
| Delivery coupling | Delivered quantity feedback from transfers, the shipping policy, warehouse selection, availability and lead-time warnings, cancellation propagation, return-driven quantity decrease. |
| Service coupling | Service lines that create projects, tasks or milestones; service lines re-invoiced from expenses or from timesheets; service lines that raise a purchase request. |
| Sales organisation | Sales teams with members, capacities, invoicing targets and dashboards; salesperson assignment; per-salesperson and per-team record visibility. |
| Analysis | A read-only reporting entity aggregating order lines by many dimensions, with quantity, revenue, margin and delay measures. |
| Ancillary flows | Global and per-line discount wizard, mass cancellation, payment links, accrued revenue entries at period close, duplicate-order detection, partner credit warning, partner and product sale warnings. |

---

## 2. Entities of the domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Sales Order | `sale.order` | `sale_order` | The offer and, after confirmation, the contract with the customer. |
| Sales Order Line | `sale.order.line` | `sale_order_line` | One priced position, section header, subsection header or note on an order. |
| Quotation Template | `sale.order.template` | `sale_order_template` | A reusable skeleton of lines and conditions applied when creating a quotation. |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | One line of a quotation template, possibly a section, subsection, note or optional line. |
| Sales Team | `crm.team` | `crm_team` | A group of salespeople with a target, a dashboard and shared document visibility. |
| Sales Team Member | `crm.team.member` | `crm_team_member` | The membership of one user in one team, with the assignment capacity. |
| Sales Tag | `crm.tag` | `crm_tag` | A free classification label attachable to orders and to opportunities. |
| Sales Analysis | `sale.report` | database view `sale_report` | Read-only aggregation of order lines for reporting. |
| Advance Payment Invoice Wizard | `sale.advance.payment.inv` | transient | The invoicing dialogue: regular invoice, percentage advance, fixed advance. |
| Discount Wizard | `sale.order.discount` | transient | Applies a per-line discount, a global percentage discount or a fixed discount amount. |
| Mass Cancel Wizard | `sale.mass.cancel.orders` | transient | Cancels several quotations or orders at once. |
| Payment Link Wizard | `payment.link.wizard` | transient | Produces a customer-facing payment address for an order. |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | transient | Produces the period-end revenue accrual entry for confirmed but uninvoiced orders. |

Entities that this domain extends rather than owns — and whose extensions are specified here — are
the Customer (`res.partner`), the Product Template and Product Variant (`product.template`,
`product.product`), the Company (`res.company`), the Customer Invoice (`account.move`) and its lines
(`account.move.line`), the Payment Transaction (`payment.transaction`), the Payment Provider
(`payment.provider`), the Analytic Line (`account.analytic.line`), and the Campaign
(`utm.campaign`).

---

## 3. Reading order

1. `README.md` — this file: scope, entity list, vocabulary of the domain.
2. `glossary.md` — every term used below, defined in full. Read it before the detailed files.
3. `entities.md` — every field of every entity with its type, default, computation and rules.
4. `state-machines.md` — the order state machine, the lock flag, the expiration flag, the invoice
   status machine per line and per order, the delivery status machine, and the template and team
   life cycles.
5. `calculations.md` — every formula: line amounts, order totals, delivered and invoiced
   quantities, quantity to invoice, amounts to invoice, prepayment amount, advance-invoice amount
   distribution, combo price proration, expected date, margin, accrual amount.
6. `workflows.md` — the end-to-end operational procedures, above all the confirmation algorithm and
   the invoicing algorithm, each as numbered steps with preconditions and postconditions.
7. `business-rules.md` — every validation, constraint, permission check, locking rule and edge case,
   with the exact message text.
8. `accounting-effects.md` — what the domain does and does not post, and the exact mapping from
   order to invoice and from invoice back to the order.
9. `configuration.md` — settings, parameters, sequences, groups, access rights, record rules,
   scheduled jobs, shipped data.
10. `interfaces.md` — menus, views, buttons, filters, remote operations, routes, reports, templates.
11. `acceptance-criteria.md` — numbered Given/When/Then scenarios with concrete numbers, including
    all the mandatory worked examples.

---

## 4. Dependencies on other domains

| Domain | What the sales domain needs from it | What it hands back |
|---|---|---|
| [Products and catalog](../products-and-catalog/README.md) | Product templates and variants, the sellable flag, attributes and their extra prices, combos and combo items, product documents, the invoicing policy, the service tracking setting, the description used for the customer. | Sale-specific counters on the product (quantity sold, order count) and the sale warning message. |
| [Units of measure and packaging](../units-of-measure-and-packaging/README.md) | The unit of a line, the allowed units for a product, and the conversion used whenever a delivered or invoiced quantity expressed in another unit is folded back into the line's unit. | Nothing. |
| [Pricing and price lists](../pricing-and-pricelists/README.md) | The rule-selection and price-computation algorithm that produces the unit price and, when the rule permits, the discount percentage of a line. | The order's price list and currency; the flag that enables the discount column. |
| [Taxes](../taxes/README.md) | The tax computation engine: base-line preparation, per-line rounding, the totals summary, the advance-invoice and global-discount reduction algorithms, the early payment discount handling. | Base lines built from order lines, including the special kinds *global discount*, *down payment* and *early payment*. |
| [Accounts receivable](../accounts-receivable/README.md) | The customer invoice and credit note entities, the payment term, the fiscal position mapping, the credit-limit warning text. | Fully prepared invoice values and invoice line values, the link back from each invoice line to its order line, and the reversal relation for refunds. |
| [General ledger](../general-ledger/README.md) | Journals (the invoicing journal on an order), accounts (the advance-invoice account). | Nothing directly; only through the invoices it prepares. |
| [Analytic accounting](../analytic-accounting/README.md) | Analytic distributions, distribution models and their validation. | The analytic distribution copied onto invoice lines; analytic lines are read back to compute delivered quantities for expense-driven lines. |
| [Inventory operations](../inventory-operations/README.md) | Transfers, moves, warehouses, routes, the procurement run. | Procurement values per line at confirmation; quantity changes propagated to moves; cancellation propagated to transfers. |
| [Replenishment and procurement](../replenishment-and-procurement/README.md) | The rule matching that turns a line into a transfer, a purchase request or a manufacturing order; the lead-time date computations. | The procurement group and the per-line procurement values. |
| [Multi-currency](../multi-currency/README.md) | Currency rounding and the conversion rate on the order date. | The order currency and the stored order rate. |
| [Payment providers](../payment-providers/README.md) | Providers, methods, tokens and transactions; the post-processing hook. | The order linked to the transaction, the amount required for confirmation, and the automatic invoicing that follows a completed transaction. |
| [Purchasing](../purchasing/README.md) | Purchase requests raised by service lines whose product is configured to buy on sale. | The originating order line on each purchase line; the delivered quantity fed back from the received quantity. |
| [Projects and tasks](../projects-and-tasks/README.md) | Projects, tasks, milestones and their templates. | Creation values for the project or task generated by a service line; sales-order profitability data. |
| [Timesheets](../timesheets/README.md) | Recorded time on tasks linked to a line. | The delivered quantity of a time-based service line. |
| [Expenses](../expenses/README.md) | Posted expenses re-invoiced to a customer. | New or updated order lines carrying the re-invoiced cost or sales price. |
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread, followers, message subtypes, activity scheduling, mail templates and their rendering. | The three order subtypes, the upsell activity, and the notification button labels used on the customer's message. |
| [Website and storefront](../website-and-storefront/README.md) | The customer portal layout, pagination and access tokens. | The order pages, the signature and decline endpoints, and the payment endpoint. |
| [Delivery and shipping](../delivery-and-shipping/README.md) | Carriers and their rating engines. | The shipping line added to the order and re-rated when lines change. |
| [Customer relationship management](../customer-relationship-management/README.md) | Opportunities that a quotation may originate from; the team assignment defaults. | Orders counted back on the opportunity; the won transition when the order is confirmed. |
| [Loyalty and promotions](../loyalty-and-promotions/README.md) | Programs, coupons and rewards. | Reward lines placed on the order and excluded from several computations. |

---

## 5. Conventions used throughout this folder

- Monetary amounts are always rounded with the rounding rule of the currency in question; the
  currency of an order is the currency of its price list, falling back to the currency of its
  company.
- Quantities are rounded with the decimal precision named `Product Unit`; discounts with the
  precision named `Discount`; unit prices with the precision named `Product Price`.
- "The order currency" means the currency stored on the order; "the company currency" means the
  currency of the company that owns the order. Conversions between them use the rate stored on the
  order (see [calculations.md](calculations.md), section "Currency rate").
- A *display line* is an order line whose display type is section, subsection or note; it carries no
  product, no quantity, no price and no tax. A *priced line* is any line that is not a display line.
- A *down payment line* (advance-invoice line) is a priced line flagged as an advance; it carries a
  quantity of zero and a unit price equal to the advance amount for its tax group.
- Wherever the text says "industry-standard default", the behaviour is not stated by the code and
  the value given is the conventional one that a re-implementation should adopt.
