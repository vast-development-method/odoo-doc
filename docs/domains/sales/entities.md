# Sales — Entities

This file specifies every entity owned by the sales domain and every field that the domain adds to
entities owned elsewhere. For each entity it gives the purpose, the life cycle, the complete field
table, the relations, the uniqueness rules, the ordering, the display-name rule, the archival
behaviour and the multi-company behaviour.

Field tables use three columns:

- **Field (storage name)** — the human name followed by the reproduced storage name in code font.
- **Type** — the storage type. `Text` is a single-line string unless stated; `Long text` is
  multi-line; `Rich text` is markup; `Selection` lists its values; `Link to X` is a many-to-one
  foreign key; `Collection of X` is a one-to-many back-reference; `Set of X` is a many-to-many
  relation with its relation table named where it matters; `Decimal` carries a precision name;
  `Money` is a decimal rounded with a currency.
- **Meaning and rules** — required, default, computed and from what, stored or not, readonly,
  copy behaviour on duplication, change tracking, company scoping, indexing, delete behaviour,
  selection labels.

Unless a field row says otherwise: the field is optional, is stored, is writable, is copied when
the record is duplicated, and is not tracked in the discussion thread.

---

## 1. Sales Order

**Transport name** `sale.order` — **table** `sale_order`.

### 1.1 Purpose

A Sales Order is one document that passes through two economic roles. While it is a *quotation* it
is a non-binding priced offer addressed to a customer; once confirmed it is a *sales order*, the
commitment that authorises delivery, service execution and invoicing. There is no separate
quotation entity: the same record, the same identifier and the same number carry both roles, and
the role is read from the status field.

The record is a discussion thread (it accepts messages, followers, activities and field tracking),
it is a portal document (it can be opened by the customer through a tokenised address), it
participates in the product catalogue mixin (products can be added from a catalogue view), it
carries campaign tracking parameters, and it can be produced by decoding an incoming structured
document.

### 1.2 Life cycle

1. **Created** in status *Quotation* (`draft`). A number is drawn from the order sequence at
   creation unless a number is supplied.
2. **Sent** — status *Quotation Sent* (`sent`), set either explicitly, or implicitly the first time
   a message is posted with the "mark as sent" instruction, or when a pending payment transaction
   is post-processed.
3. **Confirmed** — status *Sales Order* (`sale`). The confirmation date is stamped, downstream
   documents are created, the order may be locked.
4. **Cancelled** — status *Cancelled* (`cancel`). Draft invoices linked to the order are cancelled
   too. A cancelled order may be reset to *Quotation*.
5. **Locked** — an orthogonal boolean, not a status. A locked order is read-only for the fields
   that matter commercially; it must be unlocked before cancellation.
6. **Deleted** — only allowed while in *Quotation* or *Cancelled*.

There is no "done" status: completion is expressed by the invoice status, the delivery status and
the lock flag.

### 1.3 Ordering, display name, archival, company

- **Default ordering**: order date descending, then identifier descending. A database index exists
  on that exact pair, `(date_order desc, id desc)`.
- **Display name**: the order reference (`name`). When the reading context asks for the customer to
  be shown, the display name becomes `reference - customer name` (the separator is a space, hyphen,
  space). The name search matches on the reference only, or on the reference and the customer name
  when the same context flag is set.
- **Archival**: the entity has no archive flag. Withdrawal from circulation is expressed by
  cancellation.
- **Multi-company**: the company is required and indexed. A record rule restricts visibility to the
  companies enabled in the reader's session. Automatic company consistency checking is enabled, so
  every link that declares company checking (customer, invoice address, delivery address, fiscal
  position, payment term, payment method line, price list, team, journal, quotation template,
  warehouse) must belong to the order's company or be company-neutral.

### 1.4 Field table — identification and status

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Reference (`name`) | Text | Required. Not copied on duplication. Indexed for partial-word search. Default is the literal word `New`; at creation, if the value is missing or still equals `New`, it is replaced by the next value of the sequence coded `sale.order`, drawn in the company of the record and dated with the order date converted to the reader's time zone. Writable. |
| Company (`company_id`) | Link to Company | Required, indexed. Default is the active company of the session. Changing it on a saved order is allowed but raises the warning described in [business-rules.md](business-rules.md). |
| Customer (`partner_id`) | Link to Customer | Required, indexed, company-checked, tracked (tracking order 1). Marked as a field that may drive record defaults. Drives, by computation, the invoice address, the delivery address, the fiscal position, the payment term, the preferred payment method line, the price list, the salesperson and the terms text. |
| Status (`state`) | Selection | Required. Readonly (changed only by actions). Not copied. Indexed. Tracked (tracking order 3). Grouping in list views expands to all values even when empty. Default `draft`. Values: `draft` "Quotation", `sent` "Quotation Sent", `sale` "Sales Order", `cancel` "Cancelled". |
| Locked (`locked`) | Boolean | Default false. Not copied. Tracked. Help text: "Locked orders cannot be modified." |
| Customer Reference (`client_order_ref`) | Text | The customer's own reference for the order. Not copied. Used for duplicate detection and copied to the invoice as its reference. |
| Source Document (`origin`) | Text | Reference of the document that produced this order request. Used for duplicate detection. |
| Payment Reference (`reference`) | Text | The communication to quote when paying this order. Not copied. Filled automatically when a transaction of a manual ("custom") provider is pending. |
| Creation Date (`create_date`) | Date and time | Readonly, indexed. Set by the framework. |
| Order Date (`date_order`) | Date and time | Required. Not copied. Default is the current instant. While the order is a quotation it means "creation date of the draft"; from confirmation it means "confirmation date" and is overwritten at confirmation with the current instant. A database check constraint requires it to be present whenever the status is `sale`; its violation message is "A confirmed sales order requires a confirmation date." |
| Delivery Date (`commitment_date`) | Date and time | Not copied. The delivery date promised to the customer. When set, downstream transfers are scheduled from it instead of from product lead times. |
| Expiration (`validity_date`) | Date | Computed from the company (and from the quotation template when one is set), stored, writable, precomputed, not copied. See [calculations.md](calculations.md), "Validity date". |
| Is Expired (`is_expired`) | Boolean | Computed, not stored. True when the status is `draft` or `sent`, a validity date exists, and that date is strictly earlier than today. |
| Type Name (`type_name`) | Text | Computed, not stored, language-dependent. "Quotation" while the status is `draft`, `sent` or `cancel`; "Sales Order" otherwise. Used as the model description in notifications and as the printable document title. |
| Pending Email Template (`pending_email_template_id`) | Link to Mail Template | Readonly. Deleting the template sets this to empty. Holds the template of a status message queued for asynchronous sending. |

### 1.5 Field table — addresses, terms and commercial context

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Invoice Address (`partner_invoice_id`) | Link to Customer | Required, company-checked, indexed (index skips empty values). Computed from the customer as its invoicing address, stored, writable, precomputed. |
| Delivery Address (`partner_shipping_id`) | Link to Customer | Required, company-checked, indexed (index skips empty values). Computed from the customer as its delivery address, stored, writable, precomputed. |
| Fiscal Position (`fiscal_position_id`) | Link to Fiscal Position | Computed from the customer, the delivery address and the company, stored, writable, precomputed, company-checked. Determines the tax and account mapping. When the computed value differs from the previous one and lines already exist, the flag "has fiscal position changed" is raised so the interface can offer to re-map taxes. |
| Payment Terms (`payment_term_id`) | Link to Payment Term | Computed from the customer's sale payment term, stored, writable, precomputed. Domain limits to terms of the order company or company-neutral terms. Participates in the totals computation because of early payment discounts. |
| Payment Method (`preferred_payment_method_line_id`) | Link to Payment Method Line | Computed from the customer's preferred inbound payment method line, stored, writable, precomputed, company-checked. Domain limits to inbound lines of the order company. Copied to the invoice. |
| Price list (`pricelist_id`) | Link to Price list | Computed from the customer's price list, stored, writable, precomputed, company-checked, tracked (tracking order 1). Only recomputed while the status is `draft`. Cannot be written once the status is `sale`. Help text: "If you change the pricelist, only newly added lines will be affected." |
| Currency (`currency_id`) | Link to Currency | Computed from the price list currency, falling back to the company currency; stored; precomputed; deletion of the currency is blocked while referenced. |
| Currency Rate (`currency_rate`) | Decimal, unrounded | Computed and stored, precomputed. The conversion rate from the company currency to the order currency, looked up for the date part of the order date in the order's company. See [calculations.md](calculations.md), "Currency rate". |
| Salesperson (`user_id`) | Link to User | Computed from the customer, stored, writable, precomputed, indexed, tracked (tracking order 2). Domain: internal users who are members of the salesperson group and belong to the order's company. |
| Sales Team (`team_id`) | Link to Sales Team | Computed from the salesperson, stored, writable, precomputed, indexed, tracked, company-checked, marked as a field that may drive record defaults. Deleting the team empties this field. |
| Terms and conditions (`note`) | Rich text | Computed from the company's invoice terms (or from the quotation template when one is set), stored, writable, precomputed. |
| Invoicing Journal (`journal_id`) | Link to Journal | Computed (empty by default, or from the quotation template), stored, writable, precomputed, company-checked. Domain restricts to journals of the sale type. Help: if set, the order invoices into this journal; otherwise the sale journal with the lowest sequence is used. |
| Tags (`tag_ids`) | Set of Sales Tag, relation `sale_order_tag_rel` | Visible to the salesperson group only. |
| Campaign (`campaign_id`), Medium (`medium_id`), Source (`source_id`) | Links to campaign-tracking entities | Inherited from the tracking mixin; the sales domain forces the delete behaviour to "empty the field" when the referenced record disappears. Copied onto the invoice. |

### 1.6 Field table — lines and amounts

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Lines (`order_line`) | Collection of Sales Order Line, reverse key `order_id` | Copied on duplication, but only the lines that are not advance-invoice lines (see "Duplication" below). Searching across this collection bypasses the reader's access filter on lines. |
| Untaxed Amount (`amount_untaxed`) | Money | Computed and stored, tracked (tracking order 5). The base amount of the totals summary. |
| Taxes (`amount_tax`) | Money | Computed and stored. The tax amount of the totals summary. |
| Total (`amount_total`) | Money | Computed and stored, tracked (tracking order 4). The total amount of the totals summary. |
| Un-invoiced Balance (`amount_to_invoice`) | Money | Computed, not stored. Sum of the per-line un-invoiced balances. |
| Already invoiced (`amount_invoiced`) | Money | Computed, not stored. Sum of the per-line invoiced amounts. |
| Amount Before Discount (`amount_undiscounted`) | Decimal, unrounded | Computed, not stored. Sum over all lines of the untaxed total recomputed with a zero discount, excluding lines marked as a global discount or an advance invoice. |
| Tax Totals (`tax_totals`) | Structured value | Computed, not stored, language-dependent, excluded from exports. The complete totals summary used by the interface and by the printable document: per-tax-group subtotals, base and tax amounts, currency, and the early-payment-discount presentation. |
| Tax calculation rounding method (`tax_calculation_rounding_method`) | Selection, mirrored from the company | Read-only mirror; tells whether taxes are rounded per line or globally. |
| Company price include (`company_price_include`) | Selection, mirrored from the company | Read-only mirror of the company-level default for tax-inclusive prices. |
| Tax Country (`tax_country_id`) | Link to Country | Computed, not stored, evaluated with elevated rights. The country of the fiscal position when that position declares a foreign tax registration, otherwise the company's fiscal country. Used to filter selectable taxes on lines. |

### 1.7 Field table — invoicing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Invoices (`invoice_ids`) | Set of Customer Invoice | Computed, not stored, searchable through a dedicated search implementation, not copied. The set of customer invoices and customer credit notes that contain at least one invoice line linked to a line of this order. |
| Invoice Count (`invoice_count`) | Integer | Computed together with the previous field. |
| Invoice Status (`invoice_status`) | Selection | Computed and stored. Values: `upselling` "Upselling Opportunity", `invoiced` "Fully Invoiced", `to invoice` "To Invoice", `no` "Nothing to Invoice". Algorithm in [state-machines.md](state-machines.md). |

### 1.8 Field table — customer acceptance and payment

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Online signature (`require_signature`) | Boolean | Computed from the company setting (or from the quotation template), stored, writable, precomputed. When true, the customer must sign in the portal before the order can be confirmed from the portal. |
| Online payment (`require_payment`) | Boolean | Computed from the company setting (or from the quotation template), stored, writable, precomputed. When true, the customer must pay at least the prepayment amount before the order is confirmed from the portal. |
| Prepayment percentage (`prepayment_percent`) | Decimal, unrounded | Computed from the company setting (or from the quotation template), stored, writable, precomputed. A fraction in the interval greater than zero and not greater than one. When online payment is required, a validation rule enforces that interval. Setting it to zero through the interface switches online payment off. |
| Signature (`signature`) | Image | Not copied, stored as an attachment, resized to at most 1024 by 1024 picture elements. |
| Signed By (`signed_by`) | Text | Not copied. The name typed by the signer. |
| Signed On (`signed_on`) | Date and time | Not copied. The instant of signature. |
| Transactions (`transaction_ids`) | Set of Payment Transaction, relation `sale_order_transaction_rel` | Readonly, not copied, visible only to the invoicing group. |
| Authorized Transactions (`authorized_transaction_ids`) | Set of Payment Transaction | Computed with elevated rights, not stored, not copied, visible only to the invoicing group. The subset whose status is *authorized*. |
| Has Authorized Transactions (`has_authorized_transaction_ids`) | Boolean | Computed with elevated rights, not stored. |
| Payment Transactions Amount (`amount_paid`) | Decimal, unrounded | Computed with elevated rights, not stored. Sum of the amounts of the linked transactions whose status is *authorized* or *done*. |

### 1.9 Field table — user-experience and derived helpers

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Expected Date (`expected_date`) | Date and time | Computed, never stored (it depends on the current instant). The earliest date on which the whole order can be promised. See [calculations.md](calculations.md). |
| Has archived products (`has_archived_products`) | Boolean | Computed, not stored. True when at least one line references a product that is no longer active. |
| Sale Warning (`sale_warning_text`) | Long text | Computed, not stored. Concatenation, one per line, of the customer's sale warning, the parent customer's sale warning and every distinct product-level sale warning found on the lines. Empty unless the reader belongs to the sale-warning group. |
| Partner credit warning (`partner_credit_warning`) | Long text | Computed, not stored. Non-empty only while the status is `draft` or `sent` and the company enables credit limits; contains the credit-limit message built by the receivables domain, evaluated with the order total converted to company currency by dividing by the order rate. |
| Duplicated orders (`duplicated_order_ids`) | Set of Sales Order | Computed, not stored. Only filled for draft orders; see "Duplicate detection" in [business-rules.md](business-rules.md). |
| Has Fiscal Position Changed (`show_update_fpos`) | Boolean | Not stored at all; a screen-only flag raised when the fiscal position changes while lines exist. |
| Has Pricelist Changed (`show_update_pricelist`) | Boolean | Not stored at all; a screen-only flag raised when the price list changes while lines exist. |
| Has active pricelist (`has_active_pricelist`) | Boolean | Computed, not stored. True when at least one active price list exists for the order's company or company-neutral. |
| Country code (`country_code`) | Text, mirrored | The two-letter code of the company's fiscal country. |
| Terms type (`terms_type`) | Selection, mirrored from the company | Whether terms are plain text or a web address. |
| Access address (`access_url`) | Text | From the portal mixin; forced to the path `/my/orders/<identifier>`. |

### 1.10 Fields added by the delivery coupling

Present when inventory integration is installed.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Incoterm (`incoterm`) | Link to Incoterm | The international commercial term. Copied to the invoice. |
| Incoterm Location (`incoterm_location`) | Text | Free text naming the place that qualifies the term. |
| Shipping Policy (`picking_policy`) | Selection | Required, default `direct`. Values: `direct` "As soon as possible", `one` "When all products are ready". With `direct` the expected date is the minimum of the line expected dates; with `one` it is the maximum. |
| Warehouse (`warehouse_id`) | Link to Warehouse | Computed from the salesperson and the company, stored, writable, precomputed, company-checked. Recomputed only while the status is `draft` or `sent` or the record is unsaved. Falls back to the model-level default for the company, else the salesperson's default warehouse. |
| Transfers (`picking_ids`) | Collection of Transfer, reverse key `sale_id` | Every transfer generated for this order. |
| Delivery Orders (`delivery_count`) | Integer | Computed, not stored. |
| Delivery Status (`delivery_status`) | Selection | Computed and stored. Values: `pending` "Not Delivered", `started` "Started", `partial` "Partially Delivered", `full` "Fully Delivered"; empty when there is no transfer or all transfers are cancelled. |
| Late Availability (`late_availability`) | Boolean | Computed, not stored, searchable. True when any related transfer reports a late availability state. |
| References (`stock_reference_ids`) | Set of Stock Reference, relation `stock_reference_sale_rel` | Not copied. The procurement grouping references created for this order. |
| Effective Date (`effective_date`) | Date and time | Computed and stored. The completion instant of the earliest completed transfer whose destination is a customer location. Copied to the invoice as the delivery date. |
| Delay popover data (`json_popover`), Has late transfer (`show_json_popover`) | Text / Boolean | Computed, not stored. Screen data listing the transfers that carry a delay alert. |

### 1.11 Fields added by the quotation-template capability

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Quotation Template (`sale_order_template_id`) | Link to Quotation Template | Computed, stored, writable, precomputed, company-checked. Set to the company's default template when the company declares one and the current value differs, except for orders created through the storefront. Domain limits to templates of the order company or company-neutral templates. |

### 1.12 Relations summary

- One Sales Order has many Sales Order Lines (deleting the order deletes its lines).
- One Sales Order references exactly one Customer, one Invoice Address and one Delivery Address;
  all three are Customer records; the two addresses default from the customer's address book.
- Many Sales Orders belong to one Sales Team, one Salesperson, one Price list, one Payment Term,
  one Fiscal Position, one Quotation Template, one Warehouse and one Journal.
- Sales Orders and Payment Transactions form a many-to-many relation.
- Sales Orders and Customer Invoices are related indirectly: each invoice line may point to one or
  more order lines through the relation table `sale_order_line_invoice_rel`.

### 1.13 Duplication

Duplicating an order copies the header fields that are marked copyable and rebuilds the line
collection from the *copiable* lines only — that is, every line that is not an advance-invoice line.
The following header fields are deliberately not copied: order reference, customer reference,
payment reference, order date, delivery date, expiration date, signature, signed-by, signed-on,
status, lock flag, invoices, transactions, and the inventory references. The duplicate therefore
starts as a fresh quotation with a new number and today's date.

---

## 2. Sales Order Line

**Transport name** `sale.order.line` — **table** `sale_order_line`.

### 2.1 Purpose

A Sales Order Line is either a *priced line* (a product, a quantity, a unit, a unit price, a
discount and a tax set) or a *display line* (a section header, a subsection header or a free note).
Advance-invoice lines and global-discount lines are priced lines with special flags. The line is
also the anchor of every downstream quantity: what was delivered, what was invoiced, what remains
to invoice.

The entity mixes in analytic distribution support.

### 2.2 Life cycle

A line has no status of its own; it mirrors the status of its order through a stored mirrored field.
Its practical life cycle is:

1. Created on a draft order; freely editable.
2. Frozen partially at confirmation: the product can no longer be changed once anything has been
   delivered or invoiced; the unit cannot be changed at all once the order is confirmed.
3. Frozen further when the order is locked: the protected fields listed in
   [business-rules.md](business-rules.md) cannot be written.
4. Not deletable once the order is confirmed, except display lines and advance-invoice lines that
   have not yet been invoiced. The prescribed alternative is to set the quantity to zero.

### 2.3 Ordering, display name, uniqueness, company

- **Default ordering**: by order, then by the sequence number, then by identifier.
- **Display name**: `<order reference> - <description>` followed by the customer in parentheses.
  The description part is the product's display name when the line description still equals the
  default generated description; otherwise it is the second physical line of the description text
  when one exists, else the product display name; for lines without a product it is the first
  physical line of the description. The parenthesised suffix is the commercial customer's internal
  reference when present, else the commercial customer's name.
- **Name search** matches the description and the order reference.
- **Uniqueness**: none. Two lines may carry the same product.
- **Multi-company**: the company mirrors the order's company, is stored and indexed; a record rule
  restricts visibility to the reader's enabled companies; automatic company consistency checking is
  on.
- **Database check constraints**:
  - *Accountable lines need their required fields*: either the display type is set, or the line is
    an advance-invoice line, or both the product and the unit are present. Violation message:
    "Missing required fields on accountable sale order line."
  - *Non-accountable lines must be empty*: either the display type is absent, or all of product,
    unit price, ordered quantity, unit and lead time are empty or zero. Violation message:
    "Forbidden values on non-accountable sale order line".

### 2.4 Field table — structure and link to the order

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Reference (`order_id`) | Link to Sales Order | Required, indexed, not copied. Deleting the order deletes the line. |
| Sequence (`sequence`) | Integer | Default 10. Governs display order and the order in which lines are copied onto an invoice. |
| Company (`company_id`) | Link to Company, mirrored from the order | Stored, indexed, precomputed. |
| Currency (`currency_id`) | Link to Currency, mirrored from the order | Stored, precomputed. |
| Customer (`order_partner_id`) | Link to Customer, mirrored from the order's customer | Stored, indexed, precomputed. |
| Salesperson (`salesman_id`) | Link to User, mirrored from the order's salesperson | Stored, precomputed. Used by the per-salesperson record rule on lines. |
| Order Status (`state`) | Selection, mirrored from the order status | Stored, precomputed, not copied. |
| Tax Country (`tax_country_id`) | Link to Country, mirrored from the order | Filters the selectable taxes. |
| Display Type (`display_type`) | Selection | Default empty. Values: `line_section` "Section", `line_subsection` "Subsection", `line_note` "Note". A non-empty value makes the line a display line. It can never be changed after creation except from subsection to section. |
| Parent Section Line (`parent_id`) | Link to Sales Order Line | Computed, not stored. The section or subsection the line belongs to, derived by scanning the order's lines in sequence order: a section resets both levels; a subsection attaches to the last section; any other line attaches to the last subsection if one is open, else to the last section. |
| Collapse Prices (`collapse_prices`) | Boolean | Default false, copied. On a section: hide the prices of the section's lines in the printable document and in the portal. |
| Collapse Composition (`collapse_composition`) | Boolean | Default false, copied. On a section: hide the section's lines entirely in the printable document and in the portal, showing only the section total. |
| Optional Line (`is_optional`) | Boolean | Present with the quotation-template capability. Default false, copied. On a section: the section's lines are offered to the customer as optional additions in the portal. |
| Linked Order Line (`linked_line_id`) | Link to Sales Order Line | Not copied, indexed, deleting the parent deletes this line. Used for optional products attached to a main line and for the items of a combo. Domain: lines of the same order. |
| Linked Order Lines (`linked_line_ids`) | Collection of Sales Order Line | The reverse of the previous field. |
| Provisional identifier (`virtual_id`) | Text | Identifies a line before it has a database identifier, so that unsaved lines can be linked together. |
| Provisional link (`linked_virtual_id`) | Text | Points at the provisional identifier of the line this one is linked to. |

### 2.5 Field table — product and description

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | Link to Product Variant | Indexed (index skips empty values), company-checked, deletion of the product is blocked while referenced, may drive record defaults. Domain: products flagged as sellable. |
| Product Template (`product_template_id`) | Link to Product Template | Computed from the product, writable, not stored, searchable. Deliberately not a mirror so that the configurator can change it without touching the variant. Inherits the product's domain. |
| Variant values (`product_template_attribute_value_ids`) | Set of Template Attribute Value, mirrored from the product | Read-only mirror. |
| Custom Values (`product_custom_attribute_value_ids`) | Collection of Custom Attribute Value, reverse key `sale_order_line_id` | Computed, stored, writable, precomputed, copied. Recomputation drops any custom value whose attribute value does not belong to the line's product template. |
| Extra Values (`product_no_variant_attribute_value_ids`) | Set of Template Attribute Value | Computed, stored, writable, precomputed; deletion of a referenced value is blocked. Holds the chosen values of attributes that do not create variants, so their extra price and their description can be applied. Recomputation drops values that do not belong to the line's product template. |
| Is the product configurable? (`is_configurable_product`) | Boolean, mirrored from the template | True when the template has configurable attributes. |
| Is product archived (`is_product_archived`) | Boolean | Computed, not stored. |
| Product type (`product_type`) | Selection, mirrored from the product | Goods, service or combo. |
| Service tracking (`service_tracking`) | Selection, mirrored from the product | What the service line creates on confirmation. |
| Category (`categ_id`) | Link to Product Category, mirrored from the product | |
| Combo item (`combo_item_id`) | Link to Combo Item | Set only on the lines that represent the chosen items of a combo product. Never set manually. |
| Selected combo items (`selected_combo_items`) | Text, never stored | A transient payload written by the combo configurator; consumed and cleared by the line-collection change handler. |
| Description (`name`) | Long text | Required. Computed from the product (and from the quotation template line when one matches), stored, writable, precomputed. Composition rules in [calculations.md](calculations.md), "Line description". |
| Translated product name (`translated_product_name`) | Long text | Computed, not stored. The product display name rendered in the customer's language. |
| Sale warning (`sale_line_warn_msg`) | Long text | Computed, not stored. The product's sale warning, or empty when the reader is not in the sale-warning group. |

### 2.6 Field table — quantity and unit

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Quantity (`product_uom_qty`) | Decimal, precision `Product Unit` | Required, default 1. Computed, stored, writable, precomputed: forced to zero whenever the line is a display line. Writing it on a confirmed order posts a tracking note and, with inventory integration, re-runs procurement. |
| Unit (`product_uom_id`) | Decimal link to Unit of Measure | Computed from the product, stored, writable, precomputed; deletion of the unit is blocked while referenced. Reset to the product's reference unit whenever the product changes or the current unit does not match. Domain limited to the allowed units. |
| Allowed units (`allowed_uom_ids`) | Set of Unit of Measure | Computed, not stored. The product's reference unit plus the additional units declared on the product. |
| Unit readonly (`product_uom_readonly`) | Boolean | Computed, not stored. True once the line exists in the database and the order status is `sale` or `cancel`. |
| Lead Time (`customer_lead`) | Decimal, unrounded | Required, computed, stored, writable, precomputed. Zero by default; with inventory integration it defaults to the product's sale delay, and writing it on a confirmed order without a promised delivery date pushes a new deadline onto the related moves. Number of days between confirmation and shipment. |

### 2.7 Field table — pricing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Taxes (`tax_ids`) | Set of Tax | Computed from the product and the company, stored, writable, precomputed, company-checked. Reading context disables the active filter and hides original taxes. Domain: taxes of the sale usage whose country equals the order's tax country. Computation: take the product's sale taxes filtered to the company, then map them through the order's fiscal position; a combo product line always ends with no tax. |
| Price list rule (`pricelist_item_id`) | Link to Price list Item | Computed, not stored. The rule the price list selected for this product, quantity, unit and date. Empty for display lines and when the order has no price list. |
| Unit Price (`price_unit`) | Decimal, minimum display precision `Product Price` | Required, computed, stored, writable, precomputed. See [calculations.md](calculations.md), "Unit price". |
| Technical unit price (`technical_price_unit`) | Decimal, unrounded | The last automatically computed unit price. Used to detect a manual override: if it differs from the unit price by more than a currency rounding step, the price is treated as manual and is no longer recomputed. |
| Discount (%) (`discount`) | Decimal, precision `Discount` | Computed, stored, writable, precomputed. Percentage; see [calculations.md](calculations.md), "Discount derivation". |
| Subtotal (`price_subtotal`) | Money | Computed and stored, precomputed. Amount excluding tax after discount. |
| Total Tax (`price_tax`) | Decimal, unrounded | Computed and stored, precomputed. Total minus subtotal. |
| Total (`price_total`) | Money | Computed and stored, precomputed. Amount including tax. |
| Price Reduce Tax excl (`price_reduce_taxexcl`) | Money | Computed and stored. Subtotal divided by the ordered quantity, or zero when the quantity is zero. |
| Price Reduce Tax incl (`price_reduce_taxinc`) | Money | Computed and stored. Total divided by the ordered quantity, or zero when the quantity is zero. |
| Extra tax data (`extra_tax_data`) | Structured value | Technical payload carrying manual tax amounts and the computation key used by the tax engine. Non-empty with a key beginning `global_discount,` marks a global-discount line; non-empty with a key beginning `down_payment,` marks an advance-invoice line produced by the invoicing wizard. |

### 2.8 Field table — flags

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Is a down payment (`is_downpayment`) | Boolean | Marks an advance-invoice line (or the section that groups them). Such lines are not copied when the order is duplicated. |
| Is expense (`is_expense`) | Boolean | True when the line originates from a re-invoiced expense or vendor bill. Drives the delivered-quantity method to the analytic mode. |
| Can Edit Product (`product_updatable`) | Boolean | Computed, not stored. False when the line is an advance-invoice line, or the order is cancelled, or the order is confirmed and (the order is locked, or something has been invoiced, or something has been delivered). With inventory integration it is also false as soon as a non-cancelled move exists. |

### 2.9 Field table — delivered quantity

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Method to update delivered qty (`qty_delivered_method`) | Selection | Computed and stored, precomputed. Base values: `manual` "Manual", `analytic` "Analytic From Expenses". Inventory integration adds `stock_move` "Stock Moves"; time tracking adds a timesheet mode. Selection rule: an expense line uses `analytic`; a goods line that is not an expense line uses `stock_move`; everything else uses `manual`. |
| Delivery Quantity (`qty_delivered`) | Decimal, precision `Product Unit` | Computed, stored, writable, not copied, default zero. In the manual mode the stored value is kept; in every other mode it is recomputed from the underlying documents. |
| Delivered (`qty_delivered_at_date`) | Decimal, precision `Product Unit` | Computed, not stored, depends on the accrual-date context value. Equal to the delivered quantity unless an accrual date in the past is supplied, in which case it is recomputed as of that date. |

### 2.10 Field table — invoiced quantity and amounts

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Invoice Lines (`invoice_lines`) | Set of Invoice Line, relation `sale_order_line_invoice_rel` (columns `order_line_id`, `invoice_line_id`) | Not copied. |
| Invoiced Quantity (`qty_invoiced`) | Decimal, precision `Product Unit` | Computed and stored. Signed sum over the linked invoice lines, converting each invoice line's quantity into the order line's unit without rounding; invoice lines add, credit-note lines subtract; lines of a cancelled invoice are skipped unless the invoice carries the legacy-invoicing payment state. |
| Invoiced Quantity (posted) (`qty_invoiced_posted`) | Decimal, precision `Product Unit` | Computed, not stored. The same sum restricted to posted invoices, with rounding on the unit conversion, and signed by the document direction. |
| Quantity To Invoice (`qty_to_invoice`) | Decimal, precision `Product Unit` | Computed and stored. See [calculations.md](calculations.md), "Quantity to invoice". |
| Invoice Status (`invoice_status`) | Selection | Computed and stored. Values: `upselling` "Upselling Opportunity", `invoiced` "Fully Invoiced", `to invoice` "To Invoice", `no` "Nothing to Invoice". |
| Untaxed Invoiced Amount (`untaxed_amount_invoiced`) | Money | Computed and stored. Signed sum of the subtotals of the posted linked invoice lines, each converted into the order currency at the invoice date. |
| Invoiced Amount (`amount_invoiced`) | Money | Computed with elevated rights, not stored. The same sum taken on tax-inclusive totals and signed by the document direction. |
| Untaxed Amount To Invoice (`untaxed_amount_to_invoice`) | Money | Computed and stored. See [calculations.md](calculations.md), "Untaxed amount to invoice". |
| Un-invoiced Balance (`amount_to_invoice`) | Money | Computed with elevated rights, not stored. |
| Amount at date (`amount_to_invoice_at_date`) | Decimal, unrounded | Computed, not stored, depends on the accrual-date context value. Used by the revenue-accrual wizard. |
| Invoiced (`qty_invoiced_at_date`) | Decimal, precision `Product Unit` | Computed, not stored, depends on the accrual-date context value. |
| Analytic lines (`analytic_line_ids`) | Collection of Analytic Line, reverse key `so_line` | Every analytic line that points back at this order line. |
| Analytic Distribution (`analytic_distribution`) | Structured value from the analytic mixin | Computed from the customer and the product through the distribution models, stored, writable. Only computed for non-display lines; the model's answer wins, otherwise the current value is kept. |

### 2.11 Fields added by the delivery coupling

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Routes (`route_ids`) | Set of Route | Deletion of a referenced route is blocked. Domain: routes flagged as selectable on a sale. Overrides the product's own routes for this line. |
| Stock Moves (`move_ids`) | Collection of Move, reverse key `sale_line_id` | Every move generated for this line. |
| Warehouse (`warehouse_id`) | Link to Warehouse | Computed and stored. The order's warehouse unless the line's routes name a rule whose source location belongs to another warehouse. |
| Is storable (`is_storable`) | Boolean, mirrored from the product | |
| Forecasted at date (`virtual_available_at_date`) | Decimal, precision `Product Unit` | Computed, not stored. |
| Scheduled date (`scheduled_date`) | Date and time | Computed, not stored. The promised delivery date, else the line's expected date. |
| Forecast expected date (`forecast_expected_date`) | Date and time | Computed, not stored. |
| Free today (`free_qty_today`) | Decimal, precision `Product Unit` | Computed, not stored. |
| On hand today (`qty_available_today`) | Decimal, unrounded | Computed, not stored. |
| Quantity to deliver (`qty_to_deliver`) | Decimal, precision `Product Unit` | Computed, not stored. Ordered quantity minus delivered quantity. |
| Replenish on order (`is_mto`) | Boolean | Computed, not stored. True when the applicable routes include the make-to-order route of the line's warehouse. |
| Show availability widget (`display_qty_widget`) | Boolean | Computed, not stored. |

### 2.12 Relations summary

- Each line belongs to exactly one order; deleting the order deletes the line.
- Lines form a self-referencing tree through the linked-line relation (a main line with its
  optional products, or a combo line with its item lines).
- Lines and invoice lines form a many-to-many relation; one order line can feed many invoice lines
  (an advance invoice, a final invoice, a credit note) and one invoice line can, in principle,
  aggregate several order lines.
- Each line may own analytic lines, moves, purchase lines, tasks or timesheet entries depending on
  the installed couplings.

---

## 3. Quotation Template

**Transport name** `sale.order.template` — **table** `sale_order_template`.

### 3.1 Purpose

A reusable skeleton applied to a new quotation: a list of lines (including sections, subsections,
notes and optional sections), the terms text, the validity duration, the signature and payment
requirements, the prepayment percentage, the confirmation message template and the invoicing
journal.

### 3.2 Ordering, display name, archival, company

- **Ordering**: sequence, then identifier.
- **Display name**: the template name.
- **Archival**: an active flag exists. Archiving a template also clears it from every company that
  used it as a default.
- **Multi-company**: an optional company. A template without a company is shared; such a template
  may not contain products that are restricted to a company.

### 3.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Quotation Template (`name`) | Text | Required, translatable. |
| Active (`active`) | Boolean | Default true. |
| Sequence (`sequence`) | Integer | Default 10. |
| Company (`company_id`) | Link to Company | Default is the active company. |
| Terms and conditions (`note`) | Rich text, translatable | Replaces the order's terms text when non-empty. |
| Confirmation Mail (`mail_template_id`) | Link to Mail Template | Domain: templates addressed to the order entity. Sent on confirmation; when empty, nothing extra is sent. |
| Quotation Duration (`number_of_days`) | Integer | Number of days used to compute the expiration date of orders created from this template. Only applied when strictly positive. |
| Online Signature (`require_signature`) | Boolean | Computed from the company setting, stored, writable. |
| Online Payment (`require_payment`) | Boolean | Computed from the company setting, stored, writable. |
| Prepayment percentage (`prepayment_percent`) | Decimal | Computed from the company setting, stored, writable. Validation: when online payment is required it must be greater than zero and not greater than one; message "Prepayment percentage must be a valid percentage." Setting it to zero switches online payment off. |
| Lines (`sale_order_template_line_ids`) | Collection of Quotation Template Line, reverse key `sale_order_template_id` | Copied on duplication. |
| Invoicing Journal (`journal_id`) | Link to Journal, company-dependent | Company-checked; domain restricted to sale journals. The value is stored per company. |

### 3.4 Behaviour on write and create

After every creation and every write, the descriptions of the template lines are refreshed in every
active language: for each line whose description still equals the product's default customer-facing
description, the translation of that description in the target language is written back. This keeps
translated templates in step with translated product descriptions.

Deactivating a template clears the default-template field of every company that pointed at it.

---

## 4. Quotation Template Line

**Transport name** `sale.order.template.line` — **table** `sale_order_template_line`.

### 4.1 Purpose

One line of a quotation template. It may be a product line, a section, a subsection or a note, and
it may be flagged optional (only meaningful on a section, whose child lines then become the
customer-selectable optional block).

### 4.2 Ordering, constraints, company

- **Ordering**: template, then sequence, then identifier.
- **Company**: mirrored from the template, stored and indexed.
- **Database check constraints**:
  - *Accountable lines*: either the display type is set, or both product and unit are present.
    Message: "Missing required product and UoM on accountable sale quote line."
  - *Non-accountable lines*: either the display type is absent, or product, quantity and unit are
    all empty or zero. Message: "Forbidden product, quantity and UoM on non-accountable sale quote
    line".

### 4.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Quotation Template Reference (`sale_order_template_id`) | Link to Quotation Template | Required, indexed; deleting the template deletes the line. |
| Sequence (`sequence`) | Integer | Default 10. |
| Company (`company_id`) | Link to Company, mirrored | Stored, indexed. |
| Product (`product_id`) | Link to Product Variant | Company-checked. Domain: sellable products that are not of the combo type. |
| Description (`name`) | Long text, translatable | When set, overrides the description generated from the product on the created order line. |
| Unit (`product_uom_id`) | Link to Unit of Measure | Computed from the product, stored, writable, precomputed. Domain: the allowed units. |
| Allowed units (`allowed_uom_ids`) | Set of Unit of Measure | Computed, not stored. |
| Quantity (`product_uom_qty`) | Decimal, precision `Product Unit` | Required, default 1. |
| Display Type (`display_type`) | Selection | Values `line_section` "Section", `line_subsection` "Subsection", `line_note` "Note". Cannot be changed after creation. Setting it at creation forces product, quantity and unit to empty. |
| Parent Section Line (`parent_id`) | Link to Quotation Template Line | Computed, not stored; same scanning rule as on order lines. |
| Optional Line (`is_optional`) | Boolean | Default false, copied. |

### 4.4 Mapping to an order line

When a template is applied, each template line produces one order line with these values: display
type, product, quantity, unit, optional flag and sequence, plus the description when the template
line carries one. The first produced line is given the sequence number minus ninety-nine so that a
re-sequencing of the first screen page cannot interleave it with later pages.

---

## 5. Sales Team

**Transport name** `crm.team` — **table** `crm_team`.

### 5.1 Purpose

A sales team groups salespeople, carries an invoicing target and a dashboard, and acts as a
visibility and reporting dimension on orders and invoices.

### 5.2 Ordering, display name, archival, company

- **Ordering**: sequence ascending, then creation date descending, then identifier descending.
- **Display name**: the team name.
- **Archival**: an active flag exists; archiving is the recommended alternative to deletion.
- **Multi-company**: the company is optional and indexed. A team without a company is shared.
  Automatic company consistency checking is on.

### 5.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Team (`name`) | Text | Required, translatable. |
| Sequence (`sequence`) | Integer | Default 10. |
| Active (`active`) | Boolean | Default true. |
| Company (`company_id`) | Link to Company | Indexed, optional. |
| Currency (`currency_id`) | Link to Currency, mirrored from the company | Readonly. |
| Team Leader (`user_id`) | Link to User | Company-checked. Domain excludes portal and public users. |
| Multiple Memberships Allowed (`is_membership_multi`) | Boolean | Computed, not stored. Reads the system parameter `sales_team.membership_multi`. |
| Salespersons (`member_ids`) | Set of User | Computed from the membership records, writable through an inverse, searchable. Domain: internal users belonging to the team's company (or to any company when the team has none). |
| Member companies (`member_company_ids`) | Set of Company | Computed, not stored. The team's company if set, otherwise all companies. |
| Membership Issue Warning (`member_warning`) | Long text | Computed, not stored. In single-membership mode, names the users who already belong to other teams and the teams concerned. |
| Sales Team Members (`crm_team_member_ids`) | Collection of Sales Team Member, reverse key `crm_team_id` | Only active memberships. |
| Sales Team Members including inactive (`crm_team_member_all_ids`) | Collection of Sales Team Member | The same collection with the active filter switched off. |
| Color Index (`color`) | Integer | Default: a pseudo-random integer from 1 to 11 inclusive. |
| Favorite Members (`favorite_user_ids`) | Set of User, relation `team_favorite_user_rel` | Default: the creating user. |
| Show on dashboard (`is_favorite`) | Boolean | Computed from the previous field for the reading user, writable through an inverse that adds or removes the reader. |
| Dashboard Button (`dashboard_button_name`) | Text | Computed, not stored. Inside the sales application it reads "Sales Analysis". |
| Invoiced This Month (`invoiced`) | Decimal, unrounded | Computed, not stored. Sum of the signed untaxed amounts of posted customer invoices, credit notes and receipts of the team whose payment state is *in payment*, *paid* or *reversed*, dated between the first day of the current month and today inclusive. |
| Invoicing Target (`invoiced_target`) | Decimal, unrounded | The monthly revenue target, used to draw the dashboard progress bar. Written through a dedicated operation that rounds the given value to the nearest integer. |
| Sale Orders count (`sale_order_count`) | Integer | Computed, not stored. Number of non-cancelled orders of the team. |

### 5.4 Behaviour

- On creation, the creator is not auto-subscribed to the team's thread; every member is added to
  the favourite list. Writing the member set also adds the new members to the favourite list.
- Writing the company forces a re-check of the membership company rule.
- Deleting a team is refused when it is one of the two shipped default teams, and refused when it
  has five or more non-cancelled orders; the message in the latter case is "Team *team name* has
  *count* active sale orders. Consider cancelling them or archiving the team instead."
- A validation rule refuses a company on the team when any member user does not belong to that
  company; the message is "The following team members are not allowed in company '*company*' of the
  Sales Team '*team*': *user names*".

---

## 6. Sales Team Member

**Transport name** `crm.team.member` — **table** `crm_team_member`.

### 6.1 Purpose

The membership of one user in one team. It exists as a record of its own so that per-member
settings (and, in the opportunity domain, assignment capacities) can be stored, and so that a
membership can be archived rather than deleted.

### 6.2 Ordering, display name, archival

- **Ordering**: creation date ascending, then identifier. This ordering defines "the main team" of
  a user: the team of the earliest membership.
- **Display name**: the user's display name.
- **Archival**: an active flag exists and is the normal way to end a membership.
- It is a discussion thread (tracked fields are logged), but creators are not auto-subscribed.

### 6.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Team (`crm_team_id`) | Link to Sales Team | Required, indexed; deleting the team deletes the membership. Default empty. Grouping expands to all teams. Company checking is deliberately disabled on this link. |
| Salesperson (`user_id`) | Link to User | Required, indexed, company-checked; deleting the user deletes the membership. Domain: internal users not already in the chosen team (in single-membership mode) and belonging to the allowed companies. |
| Active (`active`) | Boolean | Default true. |
| Users already in teams (`user_in_teams_ids`) | Set of User | Computed, not stored. Screen helper listing users that must not be offered. |
| Allowed companies (`user_company_ids`) | Set of Company | Computed, not stored. |
| Multiple Memberships Allowed (`is_membership_multi`) | Boolean | Computed, not stored, from the system parameter. |
| Membership warning (`member_warning`) | Long text | Computed, not stored. |
| Image (`image_1920`), Image 128 (`image_128`) | Images, mirrored from the user | |
| Name (`name`) | Text, mirrored from the user display name | Writable through the mirror. |
| Email (`email`), Phone (`phone`) | Text, mirrored from the user | |
| Company (`company_id`) | Link to Company, mirrored from the user | |

### 6.4 Uniqueness and synchronisation

Uniqueness is enforced by a validation rule, not by a database constraint, because archived
duplicates are allowed. In single-membership mode a user may hold at most one *active* membership.
Two mechanisms enforce it:

1. **Validation** on the team, the user and the active flag: search all active memberships with the
   same teams and users; any duplicate pair found (other than the record itself) is reported with
   the message "You are trying to create duplicate membership(s). We found that *user (team)*, …
   already exist(s)."
2. **Synchronisation** at creation, and at any write that sets the active flag, in
   single-membership mode: every other active membership of the same user, in a different team, is
   archived.

A second validation rule refuses a membership whose team has a company that the user does not
belong to; message: "User '*user*' is not allowed in the company '*company*' of the Sales Team
'*team*'."

Archiving a user archives all their memberships.

### 6.5 Fields added to the User entity

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Teams (`crm_team_ids`) | Set of Sales Team through the membership table | Computed from the active memberships, readonly, not copied, searchable, company-checked. |
| Sales Team Members (`crm_team_member_ids`) | Collection of Sales Team Member, reverse key `user_id` | |
| User Sales Team (`sale_team_id`) | Link to Sales Team | Computed and stored, readonly. The team of the earliest membership; empty when there is none. Used to default the team on invoices and similar documents. |

---

## 7. Sales Tag

**Transport name** `crm.tag` — **table** `crm_tag`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Tag Name (`name`) | Text | Required, translatable. A unique database constraint on the name; violation message "Tag name already exists!". |
| Color (`color`) | Integer | Default: a pseudo-random integer from 1 to 11 inclusive. |

The entity has no company scoping and no archive flag.

---

## 8. Sales Analysis

**Transport name** `sale.report` — a read-only database view named `sale_report`, one row per
order line (plus the rows contributed by other couplings).

### 8.1 Purpose

A denormalised, read-only aggregation used for pivot, graph and list analysis of sales. It is never
written; it is rebuilt by the database from the order and order-line tables joined to the product,
the customer, the team and the currency.

### 8.2 Ordering and grouping

Default ordering is by order date descending. Grouping and filtering are performed by the analysis
views described in [interfaces.md](interfaces.md).

### 8.3 Column table

Every column is read-only.

| Column (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Reference (`name`) | Text | The order reference of the source order. |
| Order Date (`date`) | Date and time | The order date of the source order. |
| Customer (`partner_id`) | Link to Customer | |
| Company (`company_id`) | Link to Company | |
| Price list (`pricelist_id`) | Link to Price list | |
| Sales Team (`team_id`) | Link to Sales Team | |
| Salesperson (`user_id`) | Link to User | |
| Status (`state`) | Selection | The four order statuses with the same labels. |
| Order Invoice Status (`invoice_status`) | Selection | The four order-level invoice statuses. |
| Campaign (`campaign_id`), Medium (`medium_id`), Source (`source_id`) | Links | Campaign tracking dimensions of the order. |
| Customer Entity (`commercial_partner_id`) | Link to Customer | The commercial parent of the customer. |
| Customer Country (`country_id`) | Link to Country | From the customer record. |
| Customer Industry (`industry_id`) | Link to Industry | From the customer record. |
| Customer postal code (`partner_zip`) | Text | From the customer record. |
| Customer State (`state_id`) | Link to Country State | From the customer record. |
| Order (`order_reference`) | Polymorphic reference | Points at the source order; aggregated by counting distinct values, which is how "number of orders" is measured. |
| Product Category (`categ_id`) | Link to Product Category | |
| Product Variant (`product_id`) | Link to Product Variant | |
| Product (`product_tmpl_id`) | Link to Product Template | |
| Unit (`product_uom_id`) | Link to Unit of Measure | The product's reference unit, not the line's unit. |
| Qty Ordered (`product_uom_qty`) | Decimal | Sum over the grouped lines of the ordered quantity converted from the line unit to the product reference unit. Zero when the line has no product. |
| Qty Delivered (`qty_delivered`) | Decimal | Same conversion applied to the delivered quantity. |
| Qty To Deliver (`qty_to_deliver`) | Decimal | Same conversion applied to ordered minus delivered. |
| Qty Invoiced (`qty_invoiced`) | Decimal | Same conversion applied to the invoiced quantity. |
| Qty To Invoice (`qty_to_invoice`) | Decimal | Same conversion applied to the quantity to invoice. |
| Unit Price (`price_unit`) | Decimal, averaged | Average of the line unit prices converted into the presentation currency. |
| Untaxed Total (`price_subtotal`) | Money | Sum of the line subtotals converted into the presentation currency. |
| Total (`price_total`) | Money | Sum of the line totals converted into the presentation currency. |
| Untaxed Amount To Invoice (`untaxed_amount_to_invoice`) | Money | Sum, converted; computed for product lines and for advance-invoice lines. |
| Untaxed Amount Invoiced (`untaxed_amount_invoiced`) | Money | Sum, converted; same scope. |
| Invoice Status (`line_invoice_status`) | Selection | The per-line invoice status. |
| Gross Weight (`weight`) | Decimal | Sum of the product weight multiplied by the converted ordered quantity. |
| Volume (`volume`) | Decimal | Sum of the product volume multiplied by the converted ordered quantity. |
| Discount % (`discount`) | Decimal, averaged | The line discount percentage. |
| Discount Amount (`discount_amount`) | Money | Sum of unit price × ordered quantity × discount ÷ 100, converted into the presentation currency. |
| # of Lines (`nbr`) | Integer | Count of source rows in the group. |
| Currency (`currency_id`) | Link to Currency | Always the currency of the reading company. |

### 8.4 How a row is built

1. The source is the order-line table joined to the order, the customer, the product variant, the
   product template, the line unit, the product reference unit and a currency conversion table
   keyed by company.
2. Only lines whose display type is empty are kept; sections, subsections and notes never appear.
3. Rows are grouped by product variant, order, unit price, line invoice status, product reference
   unit, product category, order reference, order date, customer, salesperson, order status, order
   invoice status, company, the three campaign dimensions, price list, team, product template, the
   five customer attributes, the advance-invoice flag, the discount percentage, the order
   identifier and the currency conversion rate.
4. Quantities are converted between units by multiplying by the line unit's factor and dividing by
   the product reference unit's factor.
5. Monetary values are converted by dividing by the order's stored currency rate and multiplying by
   the conversion rate of the presentation currency; a rate that is absent or zero is treated as
   one.

The row identifier is the smallest line identifier in the group, which lets the drill-down open the
source document.

## 9. Advance Payment Invoice Wizard

**Transport name** `sale.advance.payment.inv` — transient.

### 9.1 Purpose

The dialogue that turns one or several confirmed orders into customer invoices. It offers three
mutually exclusive methods and two switches.

### 9.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Create Invoice (`advance_payment_method`) | Selection | Required, default `delivered`. Values: `delivered` "Regular invoice", `percentage` "Down payment (percentage)", `fixed` "Down payment (fixed amount)". |
| Sales Orders (`sale_order_ids`) | Set of Sales Order | Default: the records the dialogue was opened on. |
| Order Count (`count`) | Integer | Computed, not stored. |
| Has down payments (`has_down_payments`) | Boolean | Computed. True when any selected order already carries an advance-invoice line. |
| Deduct down payments (`deduct_down_payments`) | Boolean | Default true. Only meaningful for the regular-invoice method; it is passed as the "final" switch of the invoice creation algorithm. |
| Down Payment (`amount`) | Decimal, unrounded | The percentage to invoice in advance, expressed in percent (for example 30 for thirty percent). |
| Down Payment Amount (Fixed) (`fixed_amount`) | Money | The fixed amount to invoice in advance, in the order currency. |
| Currency (`currency_id`) | Link to Currency | Computed and stored. Only filled when exactly one order is selected. |
| Company (`company_id`) | Link to Company | Computed and stored. Only filled when exactly one order is selected. |
| Already invoiced (`amount_invoiced`) | Money | Computed. Sum of the invoiced amounts of the selected orders; only confirmed (posted) advances count. |
| Draft invoice warning (`display_draft_invoice_warning`) | Boolean | Computed. True when any selected order already has a draft invoice. |
| Consolidated Billing (`consolidated_billing`) | Boolean | Default true. When true, orders that share company, customer, delivery address, currency and fiscal position are merged into one invoice; when false, one invoice per order. |

### 9.3 Validation

Before creating anything: with the percentage method the percentage must be strictly positive; with
the fixed method the fixed amount must be strictly positive. Both failures raise "The value of the
down payment amount must be positive."

A record rule restricts each wizard record to the user who created it.

---

## 10. Discount Wizard

**Transport name** `sale.order.discount` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Order (`sale_order_id`) | Link to Sales Order | Required; defaults to the record the dialogue was opened on. |
| Company (`company_id`) | Link to Company, mirrored | |
| Currency (`currency_id`) | Link to Currency, mirrored | |
| Amount (`discount_amount`) | Money | Used by the fixed-amount method. |
| Percentage (`discount_percentage`) | Decimal, unrounded | A fraction, not a percentage: 0.1 means ten percent. |
| Discount type (`discount_type`) | Selection | Default `sol_discount`. Values: `sol_discount` "On All Order Lines", `so_discount` "Global Discount", `amount` "Fixed Amount". |

Validation: for the two percentage methods the fraction may not exceed one; the message is "Invalid
discount amount". A record rule restricts the wizard to its creator.

---

## 11. Mass Cancel Wizard

**Transport name** `sale.mass.cancel.orders` — transient.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sale orders (`sale_order_ids`) | Set of Sales Order | Required; defaults to the selected records. |
| Number of orders (`sale_orders_count`) | Integer | Computed, not stored. |
| Has confirmed order (`has_confirmed_order`) | Boolean | Computed, not stored. True when at least one selected order has the status `sale`, so the dialogue can warn that confirmed orders are about to be cancelled. |

The confirming operation cancels every selected order through the ordinary cancellation path. A
record rule restricts the wizard to its creator.

---

## 12. Extensions to entities owned by other domains

### 12.1 Company (`res.company`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Online Signature (`portal_confirmation_sign`) | Boolean | Default true. The default value of the per-order signature requirement. |
| Online Payment (`portal_confirmation_pay`) | Boolean | Default false. The default value of the per-order payment requirement. |
| Prepayment percentage (`prepayment_percent`) | Decimal | Default 1.0 (one hundred percent). Validation: when online payment is enabled it must be greater than zero and not greater than one; message "Prepayment percentage must be a valid percentage." |
| Default Quotation Validity (`quotation_validity_days`) | Integer | Default 30. A database check refuses negative values with the message "You cannot set a negative number for the default quotation validity. Leave empty (or 0) to disable the automatic expiration of quotations." Zero disables automatic expiration. |
| Discount Product (`sale_discount_product_id`) | Link to Product Variant | Company-checked. Domain: service products invoiced on ordered quantities. The product carried by global-discount lines. Created on demand by the discount dialogue. |
| Downpayment Account (`downpayment_account_id`) | Link to Account | Tracked. Domain: accounts of the income, other-income or current-liability types. Overrides the account used on advance-invoice lines. |
| Default Sale Template (`sale_order_template_id`) | Link to Quotation Template | Company-checked. Applied to new quotations of the company. |
| Sale onboarding payment method (`sale_onboarding_payment_method`) | Selection | Records the choice made in the guided setup: `digital_signature` "Sign online", `paypal`, `stripe`, `other` "Pay with another payment provider", `manual` "Manual Payment". |

### 12.2 Customer (`res.partner`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sale Order Count (`sale_order_count`) | Integer | Computed, not stored, visible to the salesperson group. Counts the orders of the customer and of every descendant address, rolled up to each ancestor. |
| Sales Order (`sale_order_ids`) | Collection of Sales Order, reverse key `partner_id` | |
| Message for Sales Order (`sale_warn_msg`) | Long text | Free text shown as a warning when the customer is used on an order. |

Two editing restrictions are added: the customer's country cannot be changed, and the customer's
tax registration identifier cannot be changed, once a non-draft order exists for that customer (for
the country: an order where the customer is either the customer or the invoice address; for the tax
registration identifier: an order for any address under the same commercial customer). "Non-draft"
means a status of `sent` or `sale`.

The credit-to-invoice computation is extended: for each confirmed order of the current company
whose lines still have an untaxed amount to invoice, the order's un-invoiced balance is converted
into company currency at today's rate and added to the commercial customer's credit-to-invoice
figure.

### 12.3 Product Template (`product.template`) and Product Variant (`product.product`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Invoicing Policy (`invoice_policy`) | Selection | Computed from the product type, stored, writable, precomputed, tracked. Values: `order` "Ordered quantities", `delivery` "Delivered quantities". Forced to `order` for goods and whenever empty. |
| Track Service (`service_type`) | Selection | Computed from the type, stored, writable, precomputed. Base value: `manual` "Manually set quantities on order". Forced to `manual` for goods and whenever empty. Other couplings add timesheet-driven and milestone-driven values. |
| Re-Invoice Costs (`expense_policy`) | Selection | Computed, stored, writable. Default `no`. Values: `no` "No", `cost` "At cost", `sales_price` "Sales price". Forced to `no` when the product is not sellable. |
| Re-Invoice Policy visible (`visible_expense_policy`) | Boolean | Computed, not stored. Visible only to readers of the analytic-accounting group and only for purchasable products. |
| Sold (`sales_count`) | Decimal, precision `Product Unit` | Computed, not stored. Sum over the variants, rounded with the product's unit. On a variant it is the quantity sold in confirmed orders, converted into the product's reference unit. |
| Sales Order Line Warning (`sale_line_warn_msg`) | Long text | Free text shown as a warning when the product is put on a line. |
| Optional Products (`optional_product_ids`) | Set of Product Template, relation `product_optional_rel` | Company-checked. Products suggested as cross-sell when this product is added. |

A validation rule refuses restricting a product to a company when that product already appears on
order lines of another company; the message is "The following products cannot be restricted to the
company *company* because they have already been used in quotations or sales orders in another
company: *products* You can archive these products and recreate them with your company restriction
instead, or leave them as shared product."

Changing the product type raises a warning when the product has already been sold: "You cannot
change the product's type because it is already used in sales orders."

### 12.4 Customer Invoice (`account.move`) and Invoice Line (`account.move.line`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Orders on the invoice (`sale_order_ids`) | Set of Sales Order | Computed, not stored. The distinct orders behind the invoice lines. |
| Sale order count (`sale_order_count`) | Integer | Computed, not stored. |
| Sales Order Lines on an invoice line (`sale_line_ids`) | Set of Sales Order Line, relation `sale_order_line_invoice_rel` | The link that makes invoiced quantities flow back to the order. |
| Is a down payment (`is_downpayment`) on an invoice line | Boolean | Copied from the order line. |
| Collapse Prices / Collapse Composition on an invoice line | Booleans | Copied from the order line's section flags so the invoice prints the same way. |

### 12.5 Payment Provider (`payment.provider`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Communication (`so_reference_type`) | Selection | Default `so_name`. Values: `so_name` "Based on Document Reference", `partner` "Based on Customer ID". Decides the payment communication proposed for an order. |

### 12.6 Payment Transaction (`payment.transaction`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Orders (`sale_order_ids`) | Set of Sales Order, relation `sale_order_transaction_rel` | Readonly, not copied. |
| Number of Sales Orders (`sale_order_ids_nbr`) | Integer | Computed, not stored. |

### 12.7 Campaign (`utm.campaign`)

The campaign entity gains the count of orders and the invoiced revenue attributed to it, so that a
campaign can be evaluated against the sales it produced.

---

## 13. Entities contributed by the service couplings

These entities are owned by other domains; only the sales-specific fields are listed.

### 13.1 Purchase request raised by a sale (service-to-purchase)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sale Order Line (`sale_line_id`) on a purchase line | Link to Sales Order Line | Set when the purchase line was raised by a service line configured to buy on sale. Feeds the delivered quantity of that order line from the received quantity. |
| Purchase Lines (`purchase_line_ids`) on an order line | Collection of Purchase Line | The reverse. |
| Purchase Line Count (`purchase_line_count`) | Integer | Computed, not stored. |

### 13.2 Project and task created by a service line

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Generated Project (`project_id`) on an order line | Link to Project | The project created or reused by the line. |
| Generated Task (`task_id`) on an order line | Link to Task | The task created by the line. |
| Sale Order Item (`sale_line_id`) on a task | Link to Sales Order Line | The line whose delivery the task realises. |
| Sales Order (`sale_order_id`) on a project | Link to Sales Order | |

### 13.3 Expense re-invoicing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Customer to Reinvoice (`sale_order_id`) on an expense | Link to Sales Order | Chosen by the person recording the expense. |
| Sales Order Item (`sale_line_id`) on an expense | Link to Sales Order Line | The line created or reused to carry the re-invoiced cost. |
| Expense Count (`expense_count`) on an order | Integer | Computed, not stored. |
