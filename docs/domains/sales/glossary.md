# Sales — Glossary

Every term used in this domain folder, defined in full. Terms are listed alphabetically. Where a
term corresponds to a reproduced storage name, that name is given in code font.

---

## A

**Access token** — An opaque string stored on an order that lets someone who holds it open the
order's customer page without signing in. It is created on demand: the first time an order is sent
without a message template, the first time a payment form is prepared, and whenever a portal list
page is rendered. Possession of the token is equivalent to read access on that one document.

**Accountable line** — An order line that carries economic content: it has a product and a unit,
or it is an advance-invoice line. The opposite of a *display line*. A database check enforces the
distinction.

**Accrual, revenue** — A period-end journal entry that recognises revenue for what has been
delivered but not yet invoiced, and defers revenue for what has been invoiced but not yet
delivered. It is always paired with a reversal dated after the accrual date. Specified in
[accounting-effects.md](accounting-effects.md), section 5.

**Accrual date** — The date at which a revenue accrual is evaluated. Supplied by the operator;
defaults to the last day of the previous month. When it is in the past it also changes how the
delivered and invoiced quantities of a line are computed, because those are then restricted to
documents dated on or before it.

**Advance invoice** — A customer invoice issued before the goods or services are delivered,
charging a percentage or a fixed amount of the order. Also called a *down payment invoice*. It
creates *advance-invoice lines* on the order and is deducted on the closing invoice. See
[calculations.md](calculations.md), section 7.

**Advance-invoice account** — The account on which advance-invoice lines are booked. Selected by
the four-step rule of [accounting-effects.md](accounting-effects.md), section 3.2. Configuring it
on the company as a current-liability account makes advances a liability rather than revenue.

**Advance-invoice line** (`is_downpayment` set) — A priced order line that represents one advance
already invoiced, for one tax group. It carries an ordered quantity of zero and its amount in the
unit price. It is not copied when the order is duplicated, and it is excluded from the invoice
status computation of the order.

**Advance-invoice section line** — A display line flagged as an advance, used to group the advance
lines at the bottom of the order and, on an invoice, to introduce the deduction block. Its
description is "Down Payments".

**Amount before discount** (`amount_undiscounted`) — The sum, over all lines except those with a
special kind, of the untaxed total each line would have with a zero discount. A comparison figure,
not an accounting amount, and deliberately unrounded.

**Amount paid** (`amount_paid`) — The sum of the amounts of the payment transactions linked to the
order whose state is *authorized* or *done*. Not an accounting figure; it is what the confirmation
check compares against the required prepayment amount.

**Amount to invoice** — See *un-invoiced balance*.

**Analysis, sales** (`sale.report`) — The read-only reporting view that produces one row per order
line, converted into the reading company's currency and into each product's reference unit.

**Analytic distribution** — A map from analytic accounts to percentages, carried by an order line
and copied onto the invoice line it produces. Validated against the analytic domain's rules at
confirmation and before sending a quotation.

**Authorized transaction** — A payment transaction that has reserved funds without capturing them.
An order with at least one such transaction offers the "Capture Transaction" and "Void
Transaction" operations.

---

## B

**Backorder** — The remainder transfer created when a delivery is validated for less than the
demanded quantity. Each validation raises the delivered quantity of the order lines it touches, so
an order delivered in two steps becomes invoiceable twice when its products are invoiced on
delivered quantities.

**Base line** — The structure handed to the tax engine to describe one taxable amount: taxes,
quantity, unit price, discount, currency, rate, partner, product, analytic distribution, extra tax
data and an optional *special kind*. Order lines, early-payment adjustments, global discounts and
advances are all expressed as base lines.

---

## C

**Campaign tracking parameters** — The campaign, medium and source recorded on an order and copied
onto the invoices it produces, so that revenue can be attributed to a marketing effort. Deleting
one of the referenced records empties the field rather than blocking the deletion.

**Catalogue** — The grid screen that lets a salesperson add products to an order by typing
quantities. Its contract is specified in [interfaces.md](interfaces.md), section 6.10.

**Collapse composition** — A flag on a section line. When set, the section's lines are hidden in
the printed document and in the customer page, and only the section and its total are shown.

**Collapse prices** — A flag on a section line. When set, the prices of the section's lines are
hidden in the printed document and in the customer page, while the lines themselves are shown; the
section total is shown.

**Combo** — A product whose price covers a set of *choices*, from each of which the customer picks
one *combo item*. The combo line itself always has a price of zero and no tax; the price is spread
over the item lines in proportion to the choices' declared base prices.

**Combo item** — One selectable product inside a combo choice, possibly carrying an extra price.
The order line representing a chosen item is linked to the combo line and carries the combo item.

**Commercial customer** — The top-level company record above a contact in the customer hierarchy.
Portal visibility, credit exposure and the display suffix of an order line all use it.

**Confirmation** — The transition of an order from *Quotation* or *Quotation Sent* to *Sales
Order*, together with everything it triggers. Specified as a single algorithm in
[workflows.md](workflows.md), section 4.

**Confirmation amount** — The amount that must have been paid before a quotation is confirmed
automatically: the required prepayment amount. "Reached" means the required amount is not greater
than the amount paid, compared with the order currency's rounding.

**Confirmation template** — The message template used to notify the customer that the order is
confirmed. Selected by the rule of [workflows.md](workflows.md), section 3.2.

**Consolidated billing** — The switch of the invoicing dialogue that merges orders sharing company,
customer, delivery address, currency and fiscal position onto one invoice. When it is off, one
invoice per order is produced.

**Credit exposure** — The customer's outstanding amount used by the credit-limit warning. This
domain contributes the un-invoiced balances of confirmed orders, and excludes from the warning the
part of a draft invoice that already represents those balances.

**Currency rate** (`currency_rate`) — The conversion rate from the company currency to the order
currency, looked up once on the order date and stored. Every conversion inside the domain divides
by this stored value, so an order's reported figures never move when the rate table changes.

**Customer reference** (`client_order_ref`) — The reference the customer uses for the order. It is
copied onto the invoice as the invoice's reference and participates in duplicate detection.

---

## D

**Declining** — The customer's refusal of a quotation from the customer page, accompanied by a
reason. It cancels the order and posts the reason as a comment. Only possible while the order still
has to be signed.

**Delivered quantity** (`qty_delivered`) — What has actually been handed over for one line. Its
source depends on the *delivered-quantity method*: typed by a user, summed from analytic lines,
summed from stock moves, or summed from recorded time.

**Delivered-quantity method** (`qty_delivered_method`) — The selector that decides where a line's
delivered quantity comes from: `manual`, `analytic`, `stock_move` or the time-tracking value. Its
precedence is given in [calculations.md](calculations.md), section 5.1.

**Delivery status** (`delivery_status`) — The order-level summary of its transfers: empty,
*Not Delivered*, *Started*, *Partially Delivered*, *Fully Delivered*.

**Discount** (`discount`) — A percentage reduction applied to a line's unit price before tax. It is
derived from the price-list rule when the discount feature is enabled, and may be typed manually.
It is never posted to a separate account; it simply reduces the untaxed amount.

**Discount, global** — A reduction expressed at the level of the order rather than per line. It is
materialised as one negative order line per tax group, all carrying the company's discount product,
so that each tax is reduced in the right proportion.

**Discount product** — The service product, invoiced on ordered quantities, that global-discount
lines carry. Configured on the company or created on demand by the discount dialogue.

**Display line** — An order line whose display type is *section*, *subsection* or *note*. It has no
product, no price, no quantity, no unit and no lead time.

**Down payment** — See *advance invoice*.

**Duplicate order** — Another order of the same company and customer, not cancelled, whose
reference equals this order's source document or whose customer reference equals this order's
customer reference. Detected only for draft orders that have a customer reference.

---

## E

**Early payment discount** — A reduction offered by the payment term when the customer pays within
a short window. In the *mixed* computation mode it reduces the tax base without reducing the
untaxed amount, which the totals computation implements by appending two cancelling base lines per
line.

**Effective date** (`effective_date`) — The completion instant of the earliest completed transfer
of the order whose destination is a customer location. Copied onto the invoice as the delivery
date.

**Expected date** (`expected_date`) — The date on which the whole order can be promised, computed
from the lines' lead times and the shipping policy. Never stored, because it depends on the current
instant while the order is still a quotation.

**Expiration** — The lapse of a quotation once its expiration date has passed. Blocks the customer
page's signature and payment, but not back-office confirmation.

**Extra tax data** (`extra_tax_data`) — A technical payload on a line carrying manual tax amounts
and a computation key. A key beginning `global_discount,` marks a global-discount line; a key
beginning `down_payment,` marks an advance line. Reversing it negates the manual tax amounts, which
is how an advance deduction matches the advance exactly.

---

## F

**Final run** — An invoicing run that closes the order: advances already invoiced are deducted,
lines with a negative quantity to invoice are picked up, and a document whose total is negative is
switched to a credit note. Controlled by the "deduct down payments" switch of the invoicing
dialogue.

**Fiscal position** — The mapping that substitutes taxes and accounts according to the customer's
legal situation. Computed on the order from the customer, the delivery address and the company, and
copied onto the invoice. Every tax on every line has already been mapped through it.

---

## G

**Grouping key, invoice** — The tuple company, customer, delivery address, currency, fiscal
position. Orders sharing it are merged onto one invoice when consolidated billing is on.

---

## I

**Invoice address** (`partner_invoice_id`) — The contact to which the invoice is addressed. It, and
not the order's customer, becomes the accounting partner of the invoice.

**Invoice status, line** (`invoice_status`) — One of *Nothing to Invoice*, *To Invoice*,
*Upselling Opportunity*, *Fully Invoiced*, derived from the quantities.

**Invoice status, order** (`invoice_status`) — The same four values, derived from the lines,
excluding advance lines and display lines, with the "may not be invoiced alone" correction.

**Invoiceable line** — A line selected by the invoicing algorithm for the document being created.
Sections and subsections become invoiceable only when at least one of their children is.

**Invoiced quantity** (`qty_invoiced`) — The signed sum of the quantities of the linked invoice
lines, converted into the line's unit, with credit notes subtracting. Draft invoices already count.

**Invoicing journal** (`journal_id`) — The journal the order's invoices will use. When empty, the
receivables domain chooses the sale journal of the company with the lowest sequence.

**Invoicing policy** — The product-level setting that decides whether a line is invoiced on the
*ordered* quantity or on the *delivered* quantity.

---

## L

**Lead time, customer** (`customer_lead`) — The number of days between confirmation and shipment
for one line. Defaults to the product's sale delay with the inventory coupling, and to zero
otherwise.

**Line, priced** — Any order line that is not a display line: an ordinary product line, an advance
line or a global-discount line.

**Linked line** — The relation that attaches an optional product line, or a combo item line, to the
line it belongs to. Before either line is saved the relation is carried by provisional identifiers
instead.

**Lock** (`locked`) — An orthogonal flag that freezes the commercially significant fields of an
order's lines and forbids its cancellation. Set manually, or automatically at confirmation when the
lock feature is enabled.

---

## M

**Margin** — The difference between a line's untaxed amount and its cost, the cost being the
product's standard cost converted into the line's unit and into the order's currency. Summed at the
order level, with the percentage measured against the untaxed amount.

**Mark as sent** — The instruction that makes a message post move a draft order into *Quotation
Sent* without producing a status tracking note.

**May not be invoiced alone** — A property of a line whose product is the company's discount
product (and, with the couplings, of delivery-charge and reward lines). An order whose only
invoiceable lines have this property reports "Nothing to Invoice".

---

## N

**Note line** — A display line carrying free text. It is always considered invoiceable, but an
invoice consisting only of notes and sections is never created.

---

## O

**Optional line** (`is_optional`) — A flag on a section line, available with the quotation-template
capability. The section's lines are offered to the customer in the customer page as additions they
may accept.

**Optional product** — A product suggested alongside another, added to the order as a line linked
to the main line and described with the suffix "Option for: *main product*".

**Order date** (`date_order`) — The creation instant while the document is a quotation; the
confirmation instant from confirmation onwards. Required whenever the status is *Sales Order*.

**Ordered quantity** (`product_uom_qty`) — What the customer asked for, expressed in the line's
unit. Forced to zero on display lines.

---

## P

**Parent section line** (`parent_id`) — The section or subsection a line belongs to, derived by
scanning the order's lines in sequence order. Not stored.

**Payment reference** (`reference`) — The communication the customer should quote when paying the
order. Filled automatically for a manual provider from the provider's communication kind.

**Payment transaction** — The record of one attempt to take money through a provider. Linked to
orders through a many-to-many relation. Its state changes drive the confirmation and the automatic
invoicing.

**Pending message** — A status message queued on the order instead of being sent inline, when
asynchronous sending is enabled. The scheduled job sends it and clears the queue.

**Portal** — The customer-facing area where an order can be viewed, signed, paid, declined and
downloaded.

**Prepayment percentage** (`prepayment_percent`) — The fraction of the total that must be paid
before a quotation confirms itself. Strictly greater than zero and not greater than one whenever
online payment is required.

**Price-list rule** (`pricelist_item_id`) — The rule the price list selected for a line's product,
quantity, unit and date. Cached on the line as a non-stored computed value and used for both the
price and the discount.

**Priced line** — See *line, priced*.

**Pro-forma document** — A rendering of the order that looks like an invoice but is not one. Its
notifications carry no access button for customers. Available only with the pro-forma feature
group.

**Procurement** — The request that the replenishment engine turns into a transfer, a purchase
request or a manufacturing order. One request per goods line is raised at confirmation, for the
quantity that is not already procured.

---

## Q

**Quantity to invoice** (`qty_to_invoice`) — The quantity that the next invoice should carry:
ordered minus invoiced for an *ordered quantities* product, delivered minus invoiced for a
*delivered quantities* product, and zero whenever the order is not confirmed or the line is a
display line. It can be negative.

**Quotation** — The role of the document while its status is *Quotation* or *Quotation Sent*: a
non-binding priced offer.

**Quotation template** — A reusable skeleton of lines and conditions applied to a new quotation.

---

## R

**Reduce to target amount** — The tax-engine routine that takes a set of base lines and produces a
smaller set whose tax-inclusive total equals a requested amount exactly, with each tax reduced in
the same proportion and the rounding residues smoothed across the produced lines. It underlies both
advance invoices and global discounts.

**Refundable move** — A return move flagged so that it reduces the delivered quantity of the order
line. Without the flag, a return changes nothing on the order.

**Required prepayment amount** — The order total multiplied by the prepayment percentage, rounded
to the order currency; zero when online payment is not required.

---

## S

**Sale warning** — Free text configured on a customer or on a product that is surfaced as a banner
when that customer or product is used on an order. Visible only to readers in the sale-warning
group.

**Sales order** — The role of the document once its status is *Sales Order*: the commitment that
authorises delivery, execution and invoicing.

**Sales team** (`crm.team`) — A group of salespeople with a monthly invoicing target, a dashboard
and a shared visibility dimension.

**Sales team member** (`crm.team.member`) — The membership of one user in one team, archivable, and
unique per active pair in single-membership mode.

**Section** (`line_section`) — A display line that introduces a block of lines. A section resets
both the section and the subsection scope.

**Sequence, line** (`sequence`) — The integer that orders the lines of a document. It also orders
the lines produced on the invoice, and is renumbered when several orders are merged.

**Shipping policy** (`picking_policy`) — Whether the order is delivered as soon as possible (the
expected date is the minimum of the line expected dates) or when all products are ready (the
maximum).

**Signature** — The image, signer name and instant recorded when a customer accepts a quotation in
the customer page. Cleared when the order is reset to a quotation.

**Special kind** — The marker on a base line that identifies it as a global discount, an advance or
an early-payment adjustment. Lines with a special kind are excluded from the amount before
discount, and are sorted last when rounding residues are distributed.

**Subsection** (`line_subsection`) — A display line that introduces a block of lines inside a
section. A subsection attaches to the last section; a line attaches to the last subsection if one
is open, else to the last section. A subsection with no section above it is promoted to a section.

---

## T

**Tax country** (`tax_country_id`) — The country whose taxes may be put on a line: the fiscal
position's country when that position declares a foreign tax registration, otherwise the company's
fiscal country.

**Technical unit price** (`technical_price_unit`) — The last automatically computed unit price. A
difference between it and the unit price, larger than one currency step, is what marks the price as
manually set and stops recomputation.

**Terms and conditions** (`note`) — The text printed at the bottom of the order document. Taken
from the company or overridden by the quotation template.

**Totals summary** (`tax_totals`) — The structured value produced by the tax engine, carrying the
untaxed amount, the per-tax-group breakdown, the tax amount, the total and the early-payment
presentation. Used by the screen, by the customer page and by the printed document, so that all
three agree exactly.

**To-do activity, upsell** — The reminder scheduled on an order when it enters the upselling state,
assigned to the salesperson, worded "Upsell *order* for customer *customer*".

**Transfer** — The inventory document that moves goods to the customer. Created at confirmation by
the replenishment engine.

---

## U

**Un-invoiced balance** (`amount_to_invoice`) — The tax-inclusive amount of an order or a line that
is still open: the quantity to consider minus the quantity on posted invoices, multiplied by the
line's tax-inclusive unit total.

**Untaxed amount invoiced** (`untaxed_amount_invoiced`) — The signed sum of the untaxed amounts of
the *posted* invoice lines of a line, converted into the order currency at each invoice's date.

**Untaxed amount to invoice** (`untaxed_amount_to_invoice`) — What remains to be invoiced, excluding
tax, computed from the raw price multiplication rather than from the stored subtotal, and floored at
zero when the invoice lines used a different discount. Draft invoices are ignored.

**Upselling** — The state of a line whose product is invoiced on ordered quantities and which has
been delivered beyond what was ordered. It signals a commercial opportunity, not an error.

---

## V

**Validity date** — See *expiration*.

**Variant suffix** — The part of a line description that lists the chosen values of attributes that
do not create variants, and the custom texts typed by the customer.

---

## W

**Warehouse** (`warehouse_id`) — The stock location from which the order is served. Defaulted from
the salesperson and recomputed only while the order is a quotation. Mandatory for a confirmed order
with goods lines when the company has any warehouse.

---

## Reproduced identifiers

Every reproduced identifier used in this folder, with its full meaning.

### Transport names (entities)

| Identifier | Full name |
|---|---|
| `sale.order` | Sales Order |
| `sale.order.line` | Sales Order Line |
| `sale.order.template` | Quotation Template |
| `sale.order.template.line` | Quotation Template Line |
| `sale.report` | Sales Analysis |
| `sale.advance.payment.inv` | Advance Payment Invoice dialogue |
| `sale.order.discount` | Discount dialogue |
| `sale.mass.cancel.orders` | Mass Cancel Orders dialogue |
| `account.accrued.orders.wizard` | Accrued Orders dialogue |
| `payment.link.wizard` | Payment Link dialogue |
| `crm.team` | Sales Team |
| `crm.team.member` | Sales Team Member |
| `crm.tag` | Sales Tag |
| `account.move` | Journal Entry, and in its invoice role the Customer Invoice or Customer Credit Note |
| `account.move.line` | Journal Item, and in its invoice role the Invoice Line |
| `payment.transaction` | Payment Transaction |
| `payment.provider` | Payment Provider |
| `res.partner` | Customer, or more generally a Contact |
| `res.company` | Company |
| `res.users` | User |
| `product.template` | Product Template |
| `product.product` | Product Variant |
| `product.pricelist.item` | Price list Item |
| `account.analytic.line` | Analytic Line |
| `utm.campaign` | Campaign |

### Table names

| Identifier | Full name |
|---|---|
| `sale_order` | the Sales Order table |
| `sale_order_line` | the Sales Order Line table |
| `sale_order_template` | the Quotation Template table |
| `sale_order_template_line` | the Quotation Template Line table |
| `sale_report` | the Sales Analysis view |
| `crm_team` | the Sales Team table |
| `crm_team_member` | the Sales Team Member table |
| `crm_tag` | the Sales Tag table |
| `sale_order_line_invoice_rel` | the relation between order lines and invoice lines |
| `sale_order_transaction_rel` | the relation between orders and payment transactions |
| `sale_order_tag_rel` | the relation between orders and tags |
| `stock_reference_sale_rel` | the relation between orders and procurement references |
| `team_favorite_user_rel` | the relation between teams and the users who favour them |
| `product_optional_rel` | the relation between a product and its optional products |

### Field names on the Sales Order

| Identifier | Full name |
|---|---|
| `name` | Order Reference |
| `company_id` | Company |
| `partner_id` | Customer |
| `partner_invoice_id` | Invoice Address |
| `partner_shipping_id` | Delivery Address |
| `state` | Status |
| `locked` | Locked |
| `client_order_ref` | Customer Reference |
| `origin` | Source Document |
| `reference` | Payment Reference |
| `date_order` | Order Date |
| `commitment_date` | Delivery Date |
| `validity_date` | Expiration |
| `is_expired` | Is Expired |
| `type_name` | Type Name |
| `pending_email_template_id` | Pending Email Template |
| `fiscal_position_id` | Fiscal Position |
| `payment_term_id` | Payment Terms |
| `preferred_payment_method_line_id` | Payment Method |
| `pricelist_id` | Price list |
| `currency_id` | Currency |
| `currency_rate` | Currency Rate |
| `user_id` | Salesperson |
| `team_id` | Sales Team |
| `note` | Terms and conditions |
| `journal_id` | Invoicing Journal |
| `tag_ids` | Tags |
| `order_line` | Order Lines |
| `amount_untaxed` | Untaxed Amount |
| `amount_tax` | Taxes |
| `amount_total` | Total |
| `amount_to_invoice` | Un-invoiced Balance |
| `amount_invoiced` | Already invoiced |
| `amount_undiscounted` | Amount Before Discount |
| `tax_totals` | Tax Totals |
| `tax_country_id` | Tax Country |
| `invoice_ids` | Invoices |
| `invoice_count` | Invoice Count |
| `invoice_status` | Invoice Status |
| `require_signature` | Online signature |
| `require_payment` | Online payment |
| `prepayment_percent` | Prepayment percentage |
| `signature` | Signature |
| `signed_by` | Signed By |
| `signed_on` | Signed On |
| `transaction_ids` | Transactions |
| `authorized_transaction_ids` | Authorized Transactions |
| `has_authorized_transaction_ids` | Has Authorized Transactions |
| `amount_paid` | Payment Transactions Amount |
| `expected_date` | Expected Date |
| `has_archived_products` | Has archived products |
| `sale_warning_text` | Sale Warning |
| `partner_credit_warning` | Partner credit warning |
| `duplicated_order_ids` | Duplicated orders |
| `show_update_fpos` | Has Fiscal Position Changed |
| `show_update_pricelist` | Has Pricelist Changed |
| `has_active_pricelist` | Has active price list |
| `country_code` | Country code |
| `terms_type` | Terms type |
| `access_url` | Access address |
| `incoterm` | Incoterm |
| `incoterm_location` | Incoterm Location |
| `picking_policy` | Shipping Policy |
| `warehouse_id` | Warehouse |
| `picking_ids` | Transfers |
| `delivery_count` | Delivery Orders |
| `delivery_status` | Delivery Status |
| `late_availability` | Late Availability |
| `stock_reference_ids` | References |
| `effective_date` | Effective Date |
| `json_popover` | Delay popover data |
| `show_json_popover` | Has late transfer |
| `sale_order_template_id` | Quotation Template |
| `opportunity_id` | Opportunity |
| `campaign_id`, `medium_id`, `source_id` | Campaign, Medium, Source |
| `margin`, `margin_percent` | Margin, Margin percentage |

### Field names on the Sales Order Line

| Identifier | Full name |
|---|---|
| `order_id` | Order Reference |
| `sequence` | Sequence |
| `company_id` | Company |
| `currency_id` | Currency |
| `order_partner_id` | Customer |
| `salesman_id` | Salesperson |
| `state` | Order Status |
| `tax_country_id` | Tax Country |
| `display_type` | Display Type |
| `parent_id` | Parent Section Line |
| `collapse_prices` | Collapse Prices |
| `collapse_composition` | Collapse Composition |
| `is_optional` | Optional Line |
| `linked_line_id` | Linked Order Line |
| `linked_line_ids` | Linked Order Lines |
| `virtual_id` | Provisional identifier |
| `linked_virtual_id` | Provisional link |
| `selected_combo_items` | Selected combo items |
| `combo_item_id` | Combo item |
| `product_id` | Product |
| `product_template_id` | Product Template |
| `product_template_attribute_value_ids` | Variant values |
| `product_custom_attribute_value_ids` | Custom Values |
| `product_no_variant_attribute_value_ids` | Extra Values |
| `is_configurable_product` | Is the product configurable |
| `is_product_archived` | Is product archived |
| `product_type` | Product type |
| `service_tracking` | Service tracking |
| `categ_id` | Category |
| `name` | Description |
| `translated_product_name` | Translated product name |
| `sale_line_warn_msg` | Sale warning |
| `product_uom_qty` | Quantity |
| `product_uom_id` | Unit |
| `allowed_uom_ids` | Allowed units |
| `product_uom_readonly` | Unit readonly |
| `customer_lead` | Lead Time |
| `tax_ids` | Taxes |
| `pricelist_item_id` | Price list rule |
| `price_unit` | Unit Price |
| `technical_price_unit` | Technical unit price |
| `discount` | Discount percentage |
| `price_subtotal` | Subtotal |
| `price_tax` | Total Tax |
| `price_total` | Total |
| `price_reduce_taxexcl` | Price Reduce Tax excluded |
| `price_reduce_taxinc` | Price Reduce Tax included |
| `extra_tax_data` | Extra tax data |
| `is_downpayment` | Is a down payment |
| `is_expense` | Is expense |
| `product_updatable` | Can Edit Product |
| `qty_delivered_method` | Method to update delivered quantity |
| `qty_delivered` | Delivery Quantity |
| `qty_delivered_at_date` | Delivered at date |
| `invoice_lines` | Invoice Lines |
| `qty_invoiced` | Invoiced Quantity |
| `qty_invoiced_posted` | Invoiced Quantity, posted only |
| `qty_invoiced_at_date` | Invoiced at date |
| `qty_to_invoice` | Quantity To Invoice |
| `invoice_status` | Invoice Status |
| `untaxed_amount_invoiced` | Untaxed Invoiced Amount |
| `amount_invoiced` | Invoiced Amount |
| `untaxed_amount_to_invoice` | Untaxed Amount To Invoice |
| `amount_to_invoice` | Un-invoiced Balance |
| `amount_to_invoice_at_date` | Amount at date |
| `analytic_line_ids` | Analytic lines |
| `analytic_distribution` | Analytic Distribution |
| `route_ids` | Routes |
| `move_ids` | Stock Moves |
| `warehouse_id` | Warehouse |
| `is_storable` | Is storable |
| `virtual_available_at_date` | Forecasted at date |
| `scheduled_date` | Scheduled date |
| `forecast_expected_date` | Forecast expected date |
| `free_qty_today` | Free today |
| `qty_available_today` | On hand today |
| `qty_to_deliver` | Quantity to deliver |
| `is_mto` | Replenish on order |
| `display_qty_widget` | Show availability widget |
| `purchase_line_ids` | Generated Purchase Lines |
| `purchase_line_count` | Number of generated purchase items |
| `project_id`, `task_id` | Generated Project, Generated Task |
| `margin`, `margin_percent`, `purchase_price` | Margin, Margin percentage, Cost |

### Selection values

| Identifier | Label |
|---|---|
| `draft` | Quotation |
| `sent` | Quotation Sent |
| `sale` | Sales Order |
| `cancel` | Cancelled |
| `no` | Nothing to Invoice |
| `to invoice` | To Invoice |
| `upselling` | Upselling Opportunity |
| `invoiced` | Fully Invoiced |
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |
| `manual` | Manual (delivered-quantity method) |
| `analytic` | Analytic From Expenses |
| `stock_move` | Stock Moves |
| `order` | Ordered quantities (invoicing policy) |
| `delivery` | Delivered quantities (invoicing policy) |
| `delivered` | Regular invoice (invoicing method) |
| `percentage` | Down payment, percentage |
| `fixed` | Down payment, fixed amount |
| `sol_discount` | On All Order Lines |
| `so_discount` | Global Discount |
| `amount` | Fixed Amount |
| `direct` | As soon as possible (shipping policy) |
| `one` | When all products are ready (shipping policy) |
| `pending` | Not Delivered |
| `started` | Started |
| `partial` | Partially Delivered |
| `full` | Fully Delivered |
| `no` | No (re-invoicing policy) |
| `cost` | At cost |
| `sales_price` | Sales price |
| `so_name` | Based on Document Reference |
| `partner` | Based on Customer ID |
| `hidden` | Hidden (document visibility) |
| `quotation` | On quote |
| `sale_order` | On confirmed order |
| `out_invoice` | Customer invoice |
| `out_refund` | Customer credit note |

### Sequence and parameter keys

| Identifier | Full name |
|---|---|
| `sale.order` (sequence code) | the Sales Order numbering sequence |
| `sale.default_confirmation_template` | the default confirmation message template parameter |
| `sale.default_invoice_email_template` | the default invoice message template parameter |
| `sale.async_emails` | the asynchronous message sending parameter |
| `sale.automatic_invoice` | the automatic invoicing parameter |
| `sales_team.membership_multi` | the multiple team membership parameter |

### Route paths

| Identifier | Purpose |
|---|---|
| `/my/quotes` | the customer's quotation list |
| `/my/orders` | the customer's confirmed order list |
| `/my/orders/<identifier>` | one order's customer page |
| `/my/orders/<identifier>/accept` | recording the signature |
| `/my/orders/<identifier>/decline` | declining with a reason |
| `/my/orders/<identifier>/document/<document identifier>` | downloading a product document |
| `/my/orders/<identifier>/download_edi` | downloading the structured order document |
| `/my/orders/<identifier>/transaction` | creating a payment transaction |

### Decimal precision names

| Identifier | Purpose |
|---|---|
| `Product Unit` | quantities |
| `Product Price` | unit prices |
| `Discount` | discount percentages |
