# Purchasing — Entities

This document defines every entity the purchasing domain owns, and every field other domains'
entities gain because purchasing is installed. For each entity it gives the purpose, the
lifecycle, the complete field table, the relations, the uniqueness rules, the defaults, the
computed values with their exact rules, the default ordering, the display-name rule, the
archival behaviour and the multi-company behaviour.

Conventions used in every field table:

- **Field (storage name)** — the human name followed by the exact column or relation name in
  code font. Reproduced identifiers are exact because external contracts depend on them.
- **Type** — the logical type. `Many-to-one` is a stored foreign key; `One-to-many` is the
  reverse of a foreign key held by the other entity and stores nothing itself;
  `Many-to-many` is a join table; `Monetary` is a decimal rounded to the precision of a
  companion currency field; `Selection` is a short string restricted to a fixed list.
- **Meaning and rules** — required, default, computed and from what, stored or not, readonly,
  copy behaviour on duplication, whether the field is tracked in the message thread, company
  scoping, indexing, delete behaviour, and for selections the full list of values with labels.

Unless a field row says otherwise: the field is optional, writable, stored, copied when the
record is duplicated, and not tracked.

---

## 1. Entity index

### Entities owned by the purchasing domain

| Entity | Transport name | Storage | Kind |
|---|---|---|---|
| Purchase Order | `purchase.order` | table `purchase_order` | Persistent, with message thread |
| Purchase Order Line | `purchase.order.line` | table `purchase_order_line` | Persistent |
| Purchase Agreement | `purchase.requisition` | table `purchase_requisition` | Persistent, with message thread |
| Purchase Agreement Line | `purchase.requisition.line` | table `purchase_requisition_line` | Persistent |
| Alternative Order Group | `purchase.order.group` | table `purchase_order_group` | Persistent, technical |
| Purchase Analysis Entry | `purchase.report` | read-only database query | Reporting |
| Vendor Delay Entry | `vendor.delay.report` | read-only database view | Reporting |
| Purchases and Bills Union Entry | `purchase.bill.union` | read-only database view | Reporting |
| Purchase and Bill Line Match Entry | `purchase.bill.line.match` | read-only database query | Reporting |
| Bill To Purchase Order Assistant | `bill.to.po.wizard` | transient | Assistant |
| Alternative Order Creation Assistant | `purchase.requisition.create.alternative` | transient | Assistant |
| Alternative Order Warning Assistant | `purchase.requisition.alternative.warning` | transient | Assistant |

### Entities owned elsewhere and extended here

| Entity | Transport name | What purchasing adds |
|---|---|---|
| Company | `res.company` | Order modification policy, approval levels, double-validation amount, days to purchase. |
| Partner | `res.partner` | Supplier currency, purchase warning message, reminder settings, buyer, purchase order count, on-time delivery rate, request-for-quotation grouping policy, suggestion parameters. |
| Product Template | `product.template` | Purchasable flag usage, billing control policy, purchase warning message, purchased quantity. |
| Product Variant | `product.product` | Purchased quantity, membership of the current order, monthly demand, suggested quantity, suggested estimated price. |
| Vendor Pricelist Entry | `product.supplierinfo` | Link to the agreement line that produced it; currency defaulting from the partner; company narrowing by order. |
| Journal Entry | `account.move` | Auto-complete controls, purchase order counters, matched flag, purchase warning, matching actions, price-difference lines. |
| Journal Item | `account.move.line` | Link to the purchase order line, down-payment flag, purchase warning, analytic inheritance. |
| Tax | `account.tax` | Reverse link to purchase order lines; the used flag. |
| Analytic Account | `account.analytic.account` | Purchase order counter and the action that lists them. |
| Analytic Plan Applicability | `account.analytic.applicability` | The additional business domain value for purchase orders. |
| Stock Move | `stock.move` | Link to the purchase order line, created purchase order lines, purchase-return test, valuation from bills. |
| Stock Transfer | `stock.picking` | Link to the purchase order and the purchase order counter. |
| Stock Rule | `stock.rule` | The buy action and everything it implies. |
| Stock Reference | `stock.reference` | Reverse link to purchase orders. |
| Replenishment Assistant Mixin | `stock.replenish.mixin` | Vendor selection when a buy route is chosen. |
| Sales Order Line | `sale.order.line` | Link to the purchase line generated for a re-purchased service. |
| Repair Order | `repair.order` | Purchase order counter for parts bought for the repair. |
| Project | `project.project` | Purchase order counter reaching the project's analytic account. |

---

## 2. Purchase Order

**Transport name** `purchase.order`, **table** `purchase_order`.

### 2.1 Purpose

A Purchase Order is a single document that plays two roles in succession:

1. While its status is draft or sent, it is a **request for quotation**: a list of products
   the buyer wishes to obtain from one vendor, with indicative prices and dates, that may be
   printed or emailed to that vendor.
2. Once confirmed, it is a **purchase order**: a commitment to the vendor. Confirmation is
   the event that creates the incoming transfer, that registers the vendor as a seller of
   each product, and that opens the order for billing.

The same record therefore carries both meanings, and the printable document, the message
templates and the button labels change with the status.

### 2.2 Lifecycle summary

```
draft ──send──▶ sent ──confirm──▶ (to approve) ──approve──▶ purchase ──▶ (billed, received)
  │               │                     │                        │
  └───────────────┴─────────────────────┴──── cancel ────────────┴──▶ cancel ──delete──▶ (gone)
```

The full state table, guards and side effects are in
[`state-machines.md`](state-machines.md).

### 2.3 Field table — identification and parties

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Reference (`name`) | Text | Required. Default is the literal word `New` until the record is created, at which point the creation routine replaces it with the next value of the numbering series whose code is `purchase.order` (see [`configuration.md`](configuration.md)); if the series yields nothing the value becomes `/`. Not copied when the record is duplicated (a duplicate gets a fresh number). Indexed for substring search. Also searchable by vendor reference. |
| Priority (`priority`) | Selection | Required in practice through its default. Values: `0` = Normal, `1` = Urgent. Default `0`. Indexed. Drives the first sort key of the default ordering and the starred marker in list views. |
| Source (`origin`) | Text | Free reference of the document that caused this order to exist, for example a sales order reference, a reordering rule run, or an agreement name. Not copied. When several sources contribute, the values are concatenated with `, ` separators, never duplicated. |
| Vendor Reference (`partner_ref`) | Text | The vendor's own reference for this order (their sales order number or bid number). Not copied. Used for duplicate detection and shown next to the order reference in the display name. |
| Vendor (`partner_id`) | Many-to-one to Partner | Required. Tracked in the message thread. Company-checked: the partner must be visible to the order's company. Indexed. Changing it is the trigger for defaulting the fiscal position, the payment terms, the buyer and the currency. |
| Dropship Address (`dest_address_id`) | Many-to-one to Partner | The address goods must be shipped to when the vendor ships directly to a customer. Company-checked. When inventory is installed it is computed from the operation type and cleared whenever the destination location of the operation type is not a customer location; the computed value can be overridden. |
| Buyer (`user_id`) | Many-to-one to User | Default: the user creating the record. Tracked. Company-checked. Indexed. Defaults from the vendor's assigned buyer when the vendor is set and that vendor has one. Used for the "my orders" dashboard split and as the responsible person of activities raised on the order. |
| Company (`company_id`) | Many-to-one to Company | Required. Default: the active company. Indexed. Determines the numbering series context, the currency fallback, the approval policy, the lock policy and the record-rule visibility. |
| Agreement (`requisition_id`) | Many-to-one to Purchase Agreement | Present when purchase agreements are installed. Not copied. Indexed when not empty. Links the order to the blanket order or purchase template it was generated from; it also restricts which vendor pricelist entries may price the lines. |
| Agreement Type (`requisition_type`) | Selection, related | Read-only mirror of the agreement's type. Used to make the vendor field read-only for blanket orders. |

### 2.4 Field table — dates

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Deadline (`date_order`) | Date and time | Required. Default: the current date and time. Not copied. Indexed. For a request for quotation this is the date by which the quotation should be confirmed; for a confirmed order it is the ordering date. It is also the date used to pick the currency conversion rate and the date used when selecting a vendor price. |
| Confirmation Date (`date_approve`) | Date and time | Read-only. Not copied. Indexed. Set to the current date and time at the instant the order reaches the purchase status through approval. Never set by any other path. |
| Expected Arrival (`date_planned`) | Date and time | Computed, stored, overridable, not copied, indexed. Computed as the **earliest** expected arrival across all order lines that are not display lines and that have an expected arrival. If no line qualifies the value is empty. Writing it from the form pushes the same value down to every non-display line (see the note on the propagation guard below). |
| Calendar Start (`date_calendar_start`) | Date and time | Computed and stored, read-only. Equals the confirmation date when the status is purchase, otherwise the order deadline. Exists so a calendar view can place requests for quotation by their deadline and orders by their confirmation. |
| Arrival (`effective_date`) | Date and time | Present when inventory is installed. Computed and stored, read-only, not copied. The earliest completion timestamp among the order's transfers that are done, whose destination is not a vendor location, and that have a completion timestamp. Empty while nothing has been received. |

**Propagation guard.** Writing the header expected arrival sets the same value on all
non-display lines. The reverse must not happen: when a line's expected arrival changes, the
header recomputes to the new minimum, but the recomputed header value must not be pushed back
down onto the other lines. The system enforces this by discarding any line-level expected
arrival change that the form engine produces as a side effect of a change to the line
collection itself.

### 2.5 Field table — status and locking

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Status (`state`) | Selection | Required through its default. Read-only to the user; changed only by the documented operations. Not copied (a duplicate starts at draft). Indexed. Tracked. Values: `draft` = "RFQ" (request for quotation), `sent` = "RFQ Sent", `to approve` = "To Approve", `purchase` = "Purchase Order", `cancel` = "Cancelled". Default `draft`. |
| Locked (`locked`) | Boolean | Default false. Not copied. Tracked. When true the order may not be cancelled and the interface makes the document read-only. Set automatically at approval when the company policy says confirmed orders are not editable. |
| Order Modification Policy (`lock_confirmed_po`) | Selection, related | Read-only mirror of the company's order modification policy. Values `edit` and `lock`. Used to decide whether approval also locks. |
| Acknowledged (`acknowledged`) | Boolean | Default false. Not copied. Tracked. Set to true when the vendor confirms reception of the order, either by the buyer pressing the acknowledge action or by the vendor following the acknowledgement link in the order email. Suppresses the automatic reminder. |
| Billing Status (`invoice_status`) | Selection | Computed and stored, read-only, not copied. Default `no`. Values: `no` = "Nothing to Bill", `to invoice` = "Waiting Bills", `invoiced` = "Fully Billed". Rule: if the status is not purchase the value is `no`; otherwise if any non-display line has a non-zero quantity to bill (compared at the product-unit decimal precision) the value is `to invoice`; otherwise if every non-display line has a zero quantity to bill **and** at least one bill is linked, the value is `invoiced`; otherwise `no`. |
| Receipt Status (`receipt_status`) | Selection | Present when inventory is installed. Computed and stored, read-only. Values: `pending` = "Not Received", `partial` = "Partially Received", `full` = "Fully Received". Rule: empty when there is no transfer or every transfer is cancelled; `full` when every transfer is done or cancelled; `partial` when at least one transfer is done; otherwise `pending`. |
| Shipped (`is_shipped`) | Boolean | Present when inventory is installed. Computed, not stored. True when the order has at least one transfer and every transfer is done or cancelled. |
| Late (`is_late`) | Boolean | Not stored, searchable only. The search implementation is given in [`calculations.md`](calculations.md). Conceptually: the order is in the purchase status, its expected arrival is in the past, at least one line has received quantity strictly below ordered quantity, and (when inventory is installed) it has either no transfer or at least one transfer that is neither done nor cancelled. |

### 2.6 Field table — currency and amounts

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Currency (`currency_id`) | Many-to-one to Currency | Required. Computed, stored, overridable, precomputed on creation. Rule: with no vendor the company currency; with a vendor, the vendor's supplier currency if set, else the company currency. |
| Company Currency (`company_currency_id`) | Many-to-one to Currency, related | Read-only mirror of the company's currency. |
| Currency Rate (`currency_rate`) | Decimal, unrounded | Computed, stored, precomputed. The conversion rate from the **company currency** to the **order currency** at the date part of the order deadline (or the current date if the deadline is empty), for the order's company. A value of 1 means the two currencies coincide. |
| Untaxed Amount (`amount_untaxed`) | Monetary in the order currency | Computed and stored, read-only, tracked. Sum of the line subtotals as produced by the tax engine. |
| Taxes (`amount_tax`) | Monetary in the order currency | Computed and stored, read-only. Total tax amount produced by the tax engine. |
| Total (`amount_total`) | Monetary in the order currency | Computed and stored, read-only. Untaxed amount plus taxes as produced by the tax engine. |
| Total in Company Currency (`amount_total_cc`) | Monetary in the company currency | Computed and stored, read-only. The total expressed in the company currency, taken from the tax engine's company-currency total. |
| Tax Totals (`tax_totals`) | Structured value | Computed, not stored, not exportable. The full breakdown the interface renders under the line list: base per tax group, tax per tax group, subtotals, grand total, and — when the order currency differs from the company currency — a formatted parenthesised company-currency total. |

All four amount fields are recomputed whenever any line subtotal, the company or the currency
changes. The exact evaluation order, the per-line rounding and the global rounding are in
[`calculations.md`](calculations.md).

### 2.7 Field table — taxes, terms and trade

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Fiscal Position (`fiscal_position_id`) | Many-to-one to Fiscal Position | Restricted to positions with no company or with the order's company. Defaulted from the vendor when the vendor changes. Changing it recomputes the taxes of every line. |
| Tax Country (`tax_country_id`) | Many-to-one to Country | Computed, not stored, evaluated with elevated rights so that reading an order of another company does not fail. Equals the fiscal position's country when that position declares a foreign tax registration, otherwise the company's fiscal country. Used only to filter the taxes offered on lines. |
| Tax Rounding Method (`tax_calculation_rounding_method`) | Selection, related, read-only | Mirror of the company setting that decides whether tax is rounded per line or globally. |
| Price Include Policy (`company_price_include`) | Selection, related | Mirror of the company's default tax-included policy. |
| Payment Terms (`payment_term_id`) | Many-to-one to Payment Term | Restricted to terms with no company or with the order's company. Defaulted from the vendor's supplier payment term when the vendor changes. Copied to the vendor bill. |
| Incoterm (`incoterm_id`) | Many-to-one to Incoterm | The predefined international commercial term. Copied to the vendor bill. |
| Incoterm Location (`incoterm_location`) | Text | Present when inventory is installed. The named place that completes the incoterm. Propagated to the vendor bill's incoterm location when the bill has none. |
| Terms and Conditions (`note`) | Rich text | Free text printed at the bottom of the order and copied into the vendor bill narration. |

### 2.8 Field table — receipts (present when inventory is installed)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Deliver To (`picking_type_id`) | Many-to-one to Operation Type | Required. Default: the first incoming operation type of a warehouse belonging to the context company; failing that the first incoming operation type with no warehouse; failing that the same search including archived records. Restricted to operation types with no warehouse or whose warehouse belongs to the order's company. Determines the destination location of the receipt and the warehouse used for forecast checks. |
| Destination Location Type (`default_location_dest_id_usage`) | Selection, related, read-only | Mirror of the usage of the operation type's default destination location. Used only to reveal the dropship address control. |
| Receptions (`picking_ids`) | Many-to-many to Transfer | Computed and stored, not copied. All transfers reached through the moves of the order's lines. |
| Incoming Shipment Count (`incoming_picking_count`) | Integer | Computed, not stored. Number of transfers. |
| References (`reference_ids`) | Many-to-many to Stock Reference | Not copied. The procurement references that caused this order; used to group later procurement needs onto the same order and copied onto the receipt and its moves. |
| On-Time Delivery Rate (`on_time_rate`) | Decimal, related | Mirror of the vendor's on-time delivery rate, evaluated with the current user's rights. A negative value means "no data". |

### 2.9 Field table — billing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Bills (`invoice_ids`) | Many-to-many to Journal Entry | Computed and stored, read-only, not copied. Every journal entry reached through the bill lines of the order's lines. |
| Bill Count (`invoice_count`) | Integer | Computed and stored, read-only, not copied, default 0. The number of distinct bills. |
| Vendor Bill Count (`partner_bill_count`) | Integer, related, read-only | Mirror of the vendor's supplier invoice count; used to show a shortcut to the vendor's other bills. |

### 2.10 Field table — reminders, warnings and helpers

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Receipt Reminder Email (`receipt_reminder_email`) | Boolean | Computed, stored, overridable. Defaults to the vendor's reminder flag read in the order's company. When true and the reminder privilege is granted, the daily job may email the vendor before the expected arrival. |
| Days Before Receipt (`reminder_date_before_receipt`) | Integer | Computed, stored, overridable. Defaults to the vendor's days-before-receipt value read in the order's company. The platform ships a fallback of 1 day for partners that have no explicit value. |
| Purchase Warning (`purchase_warning_text`) | Long text | Computed, not stored. Empty unless the purchase-warnings privilege is granted. Otherwise it is the newline-joined, duplicate-free list of: the vendor's own purchase warning prefixed by the vendor name; the vendor's parent company's purchase warning prefixed by that company's name; and, for every line, the product's purchase line warning prefixed by the product's display name. |
| Duplicates (`duplicated_order_ids`) | Many-to-many to Purchase Order | Computed, not stored. Only draft orders get a value; every other order gets an empty set. See the duplicate detection rule in [`business-rules.md`](business-rules.md). |
| Show Comparison (`show_comparison`) | Boolean | Computed, not stored. True when at least one product on this order also appears on a confirmed purchase order that is not this one. Reveals the purchase-history comparison action. |
| Product (`product_id`) | Many-to-one to Product Variant, related | Mirror of the first line's product. Exists only to let the interface default a product when an order is created from a product context. |
| Country Code (`country_code`) | Text, related, read-only | Mirror of the company's fiscal country code. Used by country-specific layouts. |
| Alternatives (`alternative_po_ids`) | One-to-many through the group, writable | Present when purchase agreements are installed. The other orders in the same alternative group. The interface restricts the selectable records to orders other than this one whose status is draft, sent or to approve. Company-checked. |
| Alternative Group (`purchase_group_id`) | Many-to-one to Alternative Order Group | Present when purchase agreements are installed. Indexed when not empty. Technical grouping record; automatically created, merged and deleted (see section 6). |

### 2.11 Relations

| Relation | Target | Cardinality | Delete behaviour |
|---|---|---|---|
| `order_line` | Purchase Order Line | One-to-many | Lines are deleted with the order (cascade from the line's own order reference). Lines are copied when the order is duplicated. |
| `partner_id` | Partner | Many-to-one | Restricted by database integrity: a vendor referenced by an order cannot be deleted. |
| `invoice_ids` | Journal Entry | Derived many-to-many | No direct constraint; the link exists through the bill lines. |
| `picking_ids` | Transfer | Derived many-to-many | No direct constraint; the link exists through the moves. |
| `requisition_id` | Purchase Agreement | Many-to-one | Plain reference; deleting an agreement is only allowed while it is draft or cancelled. |
| `purchase_group_id` | Alternative Order Group | Many-to-one | The group deletes itself when it would hold one order or fewer. |
| `reference_ids` | Stock Reference | Many-to-many | Join table `stock_reference_purchase_rel` with columns `purchase_id` and `reference_id`. |

### 2.12 Ordering, display name and search

- **Default ordering.** Priority descending, then identifier descending. Urgent orders
  therefore sort above normal ones, and within each group the most recently created first.
- **Display name.** The order reference; if a vendor reference exists, followed by a space and
  the vendor reference in parentheses; and, only when the caller asks for the total amount,
  followed by `: ` and the total amount formatted in the order currency according to the
  reader's language. Examples: `P00007`, `P00007 (SO-3391)`, `P00007 (SO-3391): $ 1,250.00`.
- **Name search.** Matching a typed string against the order reference or the vendor
  reference.

### 2.13 Uniqueness, archival and multi-company

- **Uniqueness.** There is no database uniqueness constraint on the order reference. Uniqueness
  is achieved in practice because the reference comes from a numbering series. Duplicate
  detection is advisory only and is described in [`business-rules.md`](business-rules.md).
- **Archival.** Purchase Orders have no archive flag. The only way to retire an order is to
  cancel it; a cancelled order may then be deleted.
- **Multi-company.** Every order belongs to exactly one company. A record rule limits
  visibility to orders whose company is among the reader's allowed companies. The order's
  company governs: the numbering series, the approval policy, the lock policy, the currency
  fallback, the fiscal position lookup, the operation type default and the accounts used by
  the resulting bill. A validation refuses an order that contains products belonging to a
  company outside the order company's accessible branch tree.

---

## 3. Purchase Order Line

**Transport name** `purchase.order.line`, **table** `purchase_order_line`.

### 3.1 Purpose

A Purchase Order Line is one of four things, decided by its display type:

- a **product line** (no display type) — a product, a quantity in a unit of measure, a unit
  price, a discount percentage, taxes, and an expected arrival date;
- a **section** (`line_section`) — a heading that groups the lines beneath it;
- a **subsection** (`line_subsection`) — a second-level heading nested inside a section;
- a **note** (`line_note`) — free text printed with the order.

A product line may additionally be flagged as a **down payment**, in which case it represents
an advance paid to the vendor rather than goods to receive.

### 3.2 Field table — identity and content

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Description (`name`) | Long text | Required. Computed, stored, overridable. For a product line the computed value is the product's display name in the vendor's language, followed on new lines by the product's purchase description when it has one, and then by one line per attribute value that does not create a variant, formatted `attribute name: value`. The computation deliberately preserves a description the user has customised: it only overwrites when the current text is empty or equals one of the descriptions that the product would produce for any of its vendors; when the user has customised the text but it merely starts with a vendor-specific product display name, only that prefix is replaced. For a section, subsection or note the field holds the free text. |
| Translated Product Name (`translated_product_name`) | Long text | Computed, not stored. The product's display name rendered in the vendor's language. Used by printable documents. |
| Sequence (`sequence`) | Integer | Default 10. Controls the order of the lines within the document and therefore which section a line belongs to. |
| Display Type (`display_type`) | Selection | Default empty (a product line). Values: `line_section` = "Section", `line_subsection` = "Subsection", `line_note` = "Note". It cannot be changed after creation. |
| Down Payment (`is_downpayment`) | Boolean | Default false. Marks a line as an advance payment rather than goods. A down-payment section line is also flagged. |
| Order (`order_id`) | Many-to-one to Purchase Order | Required. Indexed. Deleting the order deletes the line (database cascade). |
| Product (`product_id`) | Many-to-one to Product Variant | Restricted to products flagged as purchasable. Indexed when not empty. Deleting a product referenced by a line is refused. |
| Product Type (`product_type`) | Selection, related, read-only | Mirror of the product's type. |
| Attribute values that create variants (`product_template_attribute_value_ids`) | Many-to-many, related, read-only | Mirror of the product's attribute values. |
| Attribute values that do not create variants (`product_no_variant_attribute_value_ids`) | Many-to-many to Template Attribute Value | Chosen values of attributes configured never to create a variant. Deleting such a value while it is referenced is refused. They are appended to the description and to the receipt's move description. |
| Parent Section Line (`parent_id`) | Many-to-one to Purchase Order Line | Computed, not stored. For a subsection it is the section above it; for a product line it is the nearest subsection above it, or, failing that, the nearest section above it; for a section it is empty. Computed by walking the order's lines in sequence order. |
| Custom Description (`product_description_variants`) | Text | Present when inventory is installed. Extra wording appended to the description by a procurement, used to distinguish otherwise identical lines. |

### 3.3 Field table — quantities and unit

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Quantity (`product_qty`) | Decimal at the product-unit precision | Required. Expressed in the line's unit of measure. No default: a line created through the interface receives a suggested quantity (see below), a line created by a procurement receives the procured quantity. |
| Total Quantity (`product_uom_qty`) | Decimal | Computed and stored. The quantity converted into the **product's own reference unit**. When the line unit equals the product unit it is the quantity itself. |
| Unit (`product_uom_id`) | Many-to-one to Unit of Measure | Restricted to the allowed units listed below. Deleting a unit referenced by a line is refused. |
| Allowed Units (`allowed_uom_ids`) | Many-to-many to Unit of Measure | Computed, not stored. The union of: the product's reference unit; the units explicitly enabled on the product; and the units of every vendor pricelist entry of the product that is either generic or specific to this exact variant. |
| Received Quantity (`qty_received`) | Decimal at the product-unit precision | Computed and stored, with an inverse, evaluated with elevated rights. Expressed in the line's unit. How it is computed depends on the received-quantity method below. |
| Manual Received Quantity (`qty_received_manual`) | Decimal at the product-unit precision | Not copied. Holds the value a user typed when the method is manual. Forced to zero whenever the method is not manual. |
| Received Quantity Method (`qty_received_method`) | Selection | Computed and stored. Values: `manual` = "Manual"; and, when inventory is installed, `stock_moves` = "Stock Moves". Rule without inventory: `manual` for goods and services, empty when there is no product. Rule with inventory: `stock_moves` for goods (products that can be stocked or that are consumable), `manual` for services, empty when there is no product. When the inventory extension is uninstalled, every line that used the stock-moves method is converted in place to the manual method with its current received quantity frozen as the manual value. |
| Billed Quantity (`qty_invoiced`) | Decimal at the product-unit precision | Computed and stored. Sum over the line's bill lines, converted into the line's unit, counting vendor bills positively and vendor refunds negatively, and skipping bill lines whose document is cancelled unless that document is marked as historically invoiced. |
| Quantity To Bill (`qty_to_invoice`) | Decimal at the product-unit precision | Computed and stored, read-only. Zero unless the order status is purchase. Under the control policy "on ordered quantities" it is ordered quantity minus billed quantity; under "on received quantities" it is received quantity minus billed quantity. |
| Received at Date (`qty_received_at_date`) | Decimal, not stored | Equals the received quantity, except when the caller supplies an accrual date in the past, in which case only moves dated on or before that date are counted. |
| Billed at Date (`qty_invoiced_at_date`) | Decimal, not stored | Equals the billed quantity, except when the caller supplies an accrual date in the past, in which case only bill lines whose document is dated on or before that date are counted. |
| Amount at Date (`amount_to_invoice_at_date`) | Decimal, not stored | The value still to be billed at the accrual date. See the formula in [`calculations.md`](calculations.md). |

### 3.4 Field table — prices, discount and taxes

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Unit Price (`price_unit`) | Decimal | Required. Computed, stored, overridable. Expressed in the order currency and per one unit of the line's unit of measure. Aggregated as an average in grouped list views. |
| Technical Unit Price (`technical_price_unit`) | Decimal | A shadow copy of the last automatically computed unit price. The automatic price computation refuses to run when the stored unit price differs from this shadow copy, which is exactly the condition "the user has typed a price by hand". |
| Discount (`discount`) | Decimal at the discount precision | Computed, stored, overridable. A percentage between 0 and 100 taken from the selected vendor pricelist entry, or 0 when there is none. |
| Unit Price Discounted (`price_unit_discounted`) | Decimal | Computed, not stored. Unit price reduced by the discount percentage. |
| Unit Price in Product Unit (`price_unit_product_uom`) | Decimal | Computed, not stored. The unit price converted so that it applies to one product reference unit instead of one line unit. Zero for display lines and down payments. |
| Taxes (`tax_ids`) | Many-to-many to Tax | Stored in the join table `account_tax_purchase_order_line_rel` with columns `purchase_order_line_id` and `account_tax_id`. Archived taxes remain visible on existing lines. Defaulted from the product's vendor taxes filtered to the order's company and then mapped through the order's fiscal position. |
| Subtotal (`price_subtotal`) | Monetary in the order currency | Computed and stored. The line total excluding tax as produced by the tax engine. |
| Total (`price_total`) | Monetary in the order currency | Computed and stored. The line total including tax as produced by the tax engine. |
| Tax Amount (`price_tax`) | Decimal | Computed and stored. Total minus subtotal. |
| Company Subtotal (`price_total_cc`) | Monetary in the company currency | Present when purchase agreements are installed. Computed and stored. The subtotal divided by the order's currency rate; used to compare competing alternatives that are quoted in different currencies. |
| Selected Vendor Price (`selected_seller_id`) | Many-to-one to Vendor Pricelist Entry | Computed, not stored. The vendor pricelist entry that the selection algorithm returns for this product, this vendor, this quantity (absolute value), this date (the date part of the order deadline) and this unit. Empty when no entry qualifies. |

### 3.5 Field table — dates, company and links

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Expected Arrival (`date_planned`) | Date and time | Computed, stored, overridable, indexed. Computed as the order deadline plus the lead time in days of the selected vendor pricelist entry; when there is no order deadline, the current date plus that lead time; when there is no vendor entry the lead time is zero. |
| Order Date (`date_order`) | Date and time, related, read-only | Mirror of the order's deadline. |
| Confirmation Date (`date_approve`) | Date and time, related, read-only | Mirror of the order's confirmation date. |
| Company (`company_id`) | Many-to-one to Company, related, stored, read-only | Mirror of the order's company. Stored so that record rules and reporting can filter on it. |
| Company Currency (`company_currency_id`) | Many-to-one to Currency, related | Mirror of the company currency. |
| Currency (`currency_id`) | Many-to-one to Currency, related | Mirror of the order's currency. |
| Status (`state`) | Selection, related | Mirror of the order's status. |
| Partner (`partner_id`) | Many-to-one to Partner, related, stored, read-only | Mirror of the order's vendor. Indexed when not empty. Stored so that vendor statistics can be computed from lines. |
| Bill Lines (`invoice_lines`) | One-to-many to Journal Item | Read-only, not copied. Every vendor bill or refund line that points at this order line. |
| Tax Rounding Method (`tax_calculation_rounding_method`) | Selection, related, read-only | Mirror of the company setting. |
| Analytic Distribution (`analytic_distribution`) | Percentage map | Provided by the analytic mixin. Computed with an overridable default taken from the analytic distribution models that match the product, the product category, the vendor, the vendor's categories and the company. A user value is never overwritten by the default. |
| Purchase Line Warning (`purchase_line_warn_msg`) | Long text | Computed, not stored. The product's purchase line warning, or empty when the purchase-warnings privilege is not granted. |

### 3.6 Field table — receipt linkage (present when inventory is installed)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Reservation (`move_ids`) | One-to-many to Stock Move | Read-only, not copied. The incoming (and return) moves generated from this line. |
| Reordering Rule (`orderpoint_id`) | Many-to-one to Reordering Rule | Not copied. Indexed when not empty. Deleting the rule clears the reference. Records which reordering rule triggered the line, so that quantity-in-progress arithmetic can attribute the line to the right location. |
| Downstream Moves (`move_dest_ids`) | Many-to-many to Stock Move | Join table `stock_move_created_purchase_line_rel`, columns `created_purchase_line_id` and `move_id`. The moves that were waiting for this purchase, used for make-to-order chaining. |
| Propagate Cancellation (`propagate_cancel`) | Boolean | Default true. When true, cancelling the line cancels the downstream moves; when false, the downstream moves are switched to make-to-stock and their status recomputed. |
| Forecast Issue (`forecasted_issue`) | Boolean | Computed, not stored. True when the product's forecast quantity in the order's warehouse at the line's expected arrival is negative, counting this line's own quantity as still missing while the order is a draft. Drives the warning icon that offers the forecast report. |
| Storable (`is_storable`) | Boolean, related | Mirror of the product's storable flag. |
| Procurement Destination (`location_final_id`) | Many-to-one to Location | The final destination the procurement asked for; used to choose the receipt's destination location when it is a child of the operation type's destination. |

### 3.7 Constraints, ordering and display

- **Default ordering.** Order, then sequence, then identifier. This is the printing order and
  the order in which sections claim the lines that follow them.
- **Accountable line constraint** (database check named `_accountable_required_fields`): a line
  must satisfy *display type is set* **or** *it is a down payment* **or** *(product, unit and
  expected arrival are all set)*. Violation message: *"Missing required fields on accountable
  purchase order line."*
- **Non-accountable line constraint** (database check named `_non_accountable_null_fields`): a
  line must satisfy *display type is empty* **or** *(product is empty and unit price is zero
  and total quantity is zero and unit is empty and expected arrival is empty)*. Violation
  message: *"Forbidden values on non-accountable purchase order line"*.
- **Archival.** Lines have no archive flag.
- **Multi-company.** A record rule limits visibility to lines whose stored company is among the
  reader's allowed companies.

---

## 4. Purchase Agreement

**Transport name** `purchase.requisition`, **table** `purchase_requisition`.

### 4.1 Purpose

A Purchase Agreement is a pre-negotiated arrangement with a vendor, or a reusable shopping
list. Its type decides which of the two it is:

- **Blanket order** — a commitment with one named vendor covering a list of products at fixed
  unit prices for a validity period. Confirming it publishes those prices as vendor pricelist
  entries, so that every request for quotation raised against the agreement automatically
  picks up the agreed price. Quantities on the generated request start at zero: the buyer
  calls off whatever is needed.
- **Purchase template** — a reusable list of products and quantities with no validity period
  and no published prices. Generating a request for quotation copies both the products **and**
  their quantities.

### 4.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Agreement (`name`) | Text | Required, read-only to the user, not copied. Default is the literal word `New`; the creation routine replaces it with the next value of the numbering series for the chosen type: the blanket-order series for a blanket order, the purchase-template series otherwise. Changing the type or the company of a draft agreement renumbers it from the series of the new type and company. |
| Active (`active`) | Boolean | Default true. Setting it to false archives the agreement and hides it from every default list. |
| Reference (`reference`) | Text | Free reference supplied by the buyer, for example the vendor's contract number. |
| Agreement Type (`requisition_type`) | Selection | Required. Default `blanket_order`. Values: `blanket_order` = "Blanket Order", `purchase_template` = "Purchase Template". Changing it is refused unless the agreement is a draft. Changing it to purchase template clears both validity dates. |
| Vendor (`vendor_id`) | Many-to-one to Partner | Company-checked. Required by the interface for blanket orders and read-only once such an agreement leaves the draft status. Selecting a vendor that already has an open confirmed blanket order raises a non-blocking warning (see [`business-rules.md`](business-rules.md)). |
| Start Date (`date_start`) | Date | Tracked. The first day the agreement is valid. Also the earliest expected arrival that a generated request may carry. |
| End Date (`date_end`) | Date | Tracked. The last day the agreement is valid. Must not precede the start date. |
| Purchase Representative (`user_id`) | Many-to-one to User | Default: the user creating the record. Company-checked. |
| Description (`description`) | Rich text | Copied into the terms and conditions of every generated request for quotation. |
| Company (`company_id`) | Many-to-one to Company | Required. Default: the active company. Changing it is refused unless the agreement is a draft, and triggers renumbering. |
| Currency (`currency_id`) | Many-to-one to Currency | Required. Computed, stored, overridable, precomputed. The vendor's supplier currency when both the vendor and that currency exist, otherwise the company currency. |
| Status (`state`) | Selection | Required. Not copied. Tracked. Default `draft`. Values: `draft` = "Draft", `confirmed` = "Confirmed", `done` = "Closed", `cancel` = "Cancelled". |
| Products to Purchase (`line_ids`) | One-to-many to Purchase Agreement Line | Copied when the agreement is duplicated. |
| Product (`product_id`) | Many-to-one to Product Variant, related | Mirror of the first line's product; used only for defaulting in some contexts. |
| Purchase Orders (`purchase_ids`) | One-to-many to Purchase Order | Every request for quotation or order raised against this agreement. |
| Number of Orders (`order_count`) | Integer | Computed, not stored. The count of the above. |

### 4.3 Ordering, uniqueness, archival, multi-company

- **Default ordering.** Identifier descending — newest first.
- **Uniqueness.** None enforced in the database; names come from numbering series.
- **Archival.** Supported through the active flag. Archiving does not change the status and
  does not affect already generated orders.
- **Multi-company.** One company per agreement, enforced by a record rule on the reader's
  allowed companies. The numbering series is resolved in the agreement's company.
- **Deletion.** Allowed only while the status is draft or cancelled. Deleting an agreement
  first deletes its lines.

---

## 5. Purchase Agreement Line

**Transport name** `purchase.requisition.line`, **table** `purchase_requisition_line`.

### 5.1 Purpose

One product of an agreement with the agreed quantity and the agreed unit price. For a blanket
order the line also owns the vendor pricelist entry it publishes; that entry is created when
the agreement is confirmed and deleted when the agreement is closed or cancelled.

### 5.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | Many-to-one to Product Variant | Required. Restricted to purchasable products. |
| Unit (`product_uom_id`) | Many-to-one to Unit of Measure | Computed, stored, overridable, precomputed: the product's reference unit. |
| Quantity (`product_qty`) | Decimal at the product-unit precision | The agreed quantity. For a blanket order it must be strictly positive at confirmation. For a purchase template it is the quantity copied onto generated requests. |
| Description (`product_description_variants`) | Text | Extra wording appended to the generated purchase order line's description. |
| Unit Price (`price_unit`) | Decimal | Default 0. Computed, stored, overridable. The computation only runs for a **draft purchase template** that has a vendor and a product: it takes the price of the vendor pricelist entry selected for that vendor, that quantity, the agreement start date and the line unit, or the product's cost when no entry qualifies. For blanket orders the buyer types the negotiated price. |
| Ordered (`qty_ordered`) | Decimal | Computed, not stored. The total quantity of this product already ordered on confirmed purchase orders raised against the agreement, converted into the line's unit. When the same product appears on several agreement lines, only the first such line receives the total and the others receive zero, so that the total is not double counted. |
| Purchase Agreement (`requisition_id`) | Many-to-one to Purchase Agreement | Required. Indexed. Deleting the agreement deletes the line (database cascade). |
| Company (`company_id`) | Many-to-one to Company, related, stored, read-only | Mirror of the agreement's company. |
| Vendor Pricelist Entries (`supplier_info_ids`) | One-to-many to Vendor Pricelist Entry | The entries this line published. For a blanket order there is exactly one; for a purchase template there is none. |
| Analytic Distribution (`analytic_distribution`) | Percentage map | Provided by the analytic mixin. Copied onto the generated purchase order line. |

### 5.3 Ordering and display

- **Default ordering.** The framework default: identifier ascending.
- **Display name.** The product's display name.
- **Multi-company.** A record rule limits visibility by the stored company.

---

## 6. Alternative Order Group

**Transport name** `purchase.order.group`, **table** `purchase_order_group`.

### 6.1 Purpose

A purely technical record whose only job is to hold together the set of purchase orders that
are alternatives of one another in a call for tenders. It has one field.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Orders (`order_ids`) | One-to-many to Purchase Order | The members of the group. |

### 6.2 Self-destruction rule

Whenever a group is written to, it counts its members; if it holds one order or fewer it
deletes itself immediately. This guarantees that a group never survives as a degenerate
"group of one". The merging rules that create and combine groups are described in
[`workflows.md`](workflows.md).

---

## 7. Purchase Analysis Entry

**Transport name** `purchase.report`, a read-only query, not a table.

### 7.1 Purpose

One analytical row per combination of purchase order, line product, line unit, line unit
price and line expected arrival, aggregating the matching purchase order lines. It is the
data source of every purchase analysis chart, pivot and list. Display lines (sections,
subsections and notes) are excluded.

Amounts are converted to a single presentation currency using the currency-rate table built
for the reader's allowed companies, and each line amount is first divided by the order's own
currency rate to bring it back to the company currency.

### 7.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Order Date (`date_order`) | Date and time | The order deadline. |
| Status (`state`) | Selection | The order status, with the labels "Draft RFQ", "RFQ Sent", "To Approve", "Purchase Order", "Cancelled". |
| Product (`product_id`) | Many-to-one to Product Variant | The line product. |
| Vendor (`partner_id`) | Many-to-one to Partner | The order vendor. |
| Confirmation Date (`date_approve`) | Date and time | The order confirmation date. |
| Reference Unit of Measure (`product_uom_id`) | Many-to-one to Unit of Measure | The **product's** reference unit, not the line unit; every quantity below is expressed in it. |
| Company (`company_id`) | Many-to-one to Company | The order company. |
| Currency (`currency_id`) | Many-to-one to Currency | The company currency. |
| Buyer (`user_id`) | Many-to-one to User | The order buyer. |
| Days to Confirm (`delay`) | Decimal, two places, averaged | Confirmation date minus order deadline, in days. |
| Days to Receive (`delay_pass`) | Decimal, two places, averaged | Line expected arrival minus order deadline, in days. |
| Total (`price_total`) | Monetary | Sum of line totals including tax, brought to the company currency and then to the presentation currency. |
| Average Cost (`price_average`) | Monetary, averaged | Value-weighted average unit price. When rows are grouped, the average is recomputed as the sum of (average price × ordered quantity) divided by the sum of ordered quantity, so that grouping does not distort it. |
| Number of Lines (`nbr_lines`) | Integer | Count of aggregated lines. |
| Product Category (`category_id`) | Many-to-one to Product Category | The product's category. |
| Product Template (`product_tmpl_id`) | Many-to-one to Product Template | The product's template. |
| Partner Country (`country_id`) | Many-to-one to Country | The vendor's country. |
| Fiscal Position (`fiscal_position_id`) | Many-to-one to Fiscal Position | The order's fiscal position. |
| Commercial Entity (`commercial_partner_id`) | Many-to-one to Partner | The vendor's commercial parent. |
| Gross Weight (`weight`) | Decimal | Sum over lines of product weight × quantity converted to the product unit. |
| Volume (`volume`) | Decimal | Sum over lines of product volume × quantity converted to the product unit. |
| Order (`order_id`) | Many-to-one to Purchase Order | The order. |
| Untaxed Total (`untaxed_total`) | Monetary | Sum of line subtotals, converted like the total. |
| Quantity Ordered (`qty_ordered`) | Decimal | Sum of ordered quantities converted to the product unit. |
| Quantity Received (`qty_received`) | Decimal | Sum of received quantities converted to the product unit. |
| Quantity Billed (`qty_billed`) | Decimal | Sum of billed quantities converted to the product unit. |
| Quantity To Be Billed (`qty_to_be_billed`) | Decimal | Under the control policy "on ordered quantities": ordered minus billed. Under "on received quantities": received minus billed. |
| Warehouse (`picking_type_id`) | Many-to-one to Warehouse | Present when inventory is installed. The warehouse of the order's operation type. Note that the storage name says operation type but the value is the warehouse. |
| Effective Date (`effective_date`) | Date and time | Present when inventory is installed. The order's arrival date. |
| Effective Days To Arrival (`days_to_arrival`) | Decimal, two places, averaged | Present when inventory is installed. The earliest completion date among the order's non-vendor-destined done transfers, or, failing that, the line's expected arrival, minus the order deadline, in days. |

- **Default ordering.** Order date descending, then total descending.
- **Multi-company.** A record rule limits rows to the reader's allowed companies.
- **Access.** Readable by purchase users and purchase administrators only.

---

## 8. Vendor Delay Entry

**Transport name** `vendor.delay.report`, a read-only database view.

One row per purchase order line that produced at least one stock move. Used to measure vendor
punctuality by product and by category.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Vendor (`partner_id`) | Many-to-one to Partner | The line's vendor. |
| Product (`product_id`) | Many-to-one to Product Variant | The line's product. |
| Product Category (`category_id`) | Many-to-one to Product Category | The product's category. |
| Effective Date (`date`) | Date and time | The earliest move date of the line. |
| Total Quantity (`qty_total`) | Decimal | The line's quantity expressed in the product's reference unit. |
| On-Time Quantity (`qty_on_time`) | Decimal | Sum over the line's move details of the detail quantity converted to the product's reference unit, counted only when the move is done **and** the line's expected arrival date is on or after the move date. |
| On-Time Delivery Rate (`on_time_rate`) | Decimal | Aggregated as a weighted percentage: on-time quantity summed, divided by total quantity summed, times one hundred; when the summed total quantity is zero the result is one hundred. Rows with a zero summed total quantity are filtered out of any grouping that aggregates this field. |

---

## 9. Purchases and Bills Union Entry

**Transport name** `purchase.bill.union`, a read-only database view.

### 9.1 Purpose

A single list that mixes two kinds of source documents so that a user drafting a vendor bill
can pick either of them from one control: posted vendor bills and refunds (to copy their
lines) and confirmed purchase orders that still have something to bill (to pull their lines).

### 9.2 Composition

The view is the union of two selections:

1. **Bills.** Every journal entry whose type is a vendor bill or a vendor refund and whose
   status is posted. The row identifier is the journal entry identifier (a positive number).
2. **Orders.** Every purchase order whose status is purchase and whose billing status is
   "Waiting Bills" or "Nothing to Bill". The row identifier is the **negative** of the purchase
   order identifier. The sign is what lets one list address two different underlying entities,
   and it is relied upon by the assistant described in section 11.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Reference (`name`) | Text | The bill's number, or the order's reference. |
| Source (`reference`) | Text | The bill's payment reference, or the order's vendor reference. |
| Vendor (`partner_id`) | Many-to-one to Partner | The document's partner. |
| Date (`date`) | Date | The bill's accounting date, or the date part of the order deadline. |
| Amount (`amount`) | Decimal | The document's untaxed amount. |
| Currency (`currency_id`) | Many-to-one to Currency | The document's currency. |
| Company (`company_id`) | Many-to-one to Company | The document's company. |
| Vendor Bill (`vendor_bill_id`) | Many-to-one to Journal Entry | Set on bill rows, empty on order rows. |
| Purchase Order (`purchase_order_id`) | Many-to-one to Purchase Order | Set on order rows, empty on bill rows. |

- **Default ordering.** Date descending, then reference descending.
- **Display name.** The reference; then, when a source exists, ` - ` and the source; then `: `
  and the amount formatted in the row's currency. For an order row whose billing status is
  "Nothing to Bill" the displayed amount is forced to zero, so that the user is not tempted to
  pull an order that has nothing left to bill.
- **Name search.** On the reference or the source.
- **Multi-company.** A record rule limits rows to the reader's allowed companies, and also
  admits rows with no company.

---

## 10. Purchase and Bill Line Match Entry

**Transport name** `purchase.bill.line.match`, a read-only query.

### 10.1 Purpose

The data behind the matching screen, where a user reconciles what was ordered with what a
vendor has billed. It unions two sets of rows.

1. **Purchase order line rows.** Rows drawn from purchase order lines belonging to orders in
   the purchase status, kept when *the ordered quantity exceeds the billed quantity* **or**
   *the quantity to bill is not zero*; plus every down-payment line that is not a display line
   and has a positive billed quantity. The row identifier is the line identifier (positive).
2. **Vendor bill line rows.** Rows drawn from journal items that are product lines of a vendor
   bill or vendor refund, whose document status is draft or posted, and that are **not yet
   linked** to a purchase order line. The row identifier is the negative of the journal item
   identifier.

### 10.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order Line (`pol_id`) | Many-to-one to Purchase Order Line, read-only | Set on order rows. |
| Bill Line (`aml_id`) | Many-to-one to Journal Item, read-only | Set on bill rows. |
| Company (`company_id`) | Many-to-one to Company, read-only | The row's company. |
| Vendor (`partner_id`) | Many-to-one to Partner, read-only | The line's partner, or the bill's partner. |
| Product (`product_id`) | Many-to-one to Product Variant, read-only | The line's product. |
| Line Quantity (`line_qty`) | Decimal, read-only | The ordered quantity, or the billed quantity. |
| Line Unit (`line_uom_id`) | Many-to-one to Unit of Measure, read-only | The line's unit. |
| Billed Quantity (`qty_invoiced`) | Decimal, read-only | Set on order rows only. |
| Quantity to Invoice (`qty_to_invoice`) | Decimal, read-only | Set on order rows only. |
| Purchase Order (`purchase_order_id`) | Many-to-one to Purchase Order, read-only | Set on order rows. |
| Bill (`account_move_id`) | Many-to-one to Journal Entry, read-only | Set on bill rows. |
| Line Untaxed Amount (`line_amount_untaxed`) | Monetary, read-only | The order line's subtotal, or the bill line's amount in its currency. |
| Currency (`currency_id`) | Many-to-one to Currency, read-only | The row's currency. |
| Status (`state`) | Text, read-only | The order status, or the bill's status. |
| Product Unit (`product_uom_id`) | Many-to-one to Unit of Measure, related | The product's reference unit. |
| Quantity in Product Unit (`product_uom_qty`) | Decimal, writable through an inverse | The line quantity converted into the product's reference unit; when the row has no product it is the raw quantity. Writing it writes the bill line's quantity on a bill row. On an order row it writes the ordered quantity while preserving the unit price, because changing the quantity would otherwise re-derive the price from the vendor pricelist. |
| Unit Price in Product Unit (`product_uom_price`) | Decimal, writable through an inverse | The bill line's unit price, or the order line's unit price. Writing it writes that price back. |
| Billed Untaxed (`billed_amount_untaxed`) | Monetary | The line untaxed amount on bill rows, otherwise empty. |
| Ordered Untaxed (`purchase_amount_untaxed`) | Monetary | The line untaxed amount on order rows, otherwise empty. |
| Reference (`reference`) | Text | The order's display name, or the bill's display name. |

- **Default ordering.** Product, then bill line, then order line.
- **Display name.** The product's display name; failing that the bill line's label; failing
  that the order line's description.

---

## 11. Bill To Purchase Order Assistant

**Transport name** `bill.to.po.wizard`, a transient record.

Opened from the matching screen with a set of selected rows. It converts selected **bill line
rows** (the rows with negative identifiers) into purchase order lines, either as ordinary
lines on a new or existing order, or as down-payment lines.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order (`purchase_order_id`) | Many-to-one to Purchase Order | The order to extend. Defaulted from the selection when exactly one order is involved; left empty to create a new one. |
| Vendor (`partner_id`) | Many-to-one to Partner | The commercial partner shared by all selected bill lines. |

Its two operations are specified step by step in [`workflows.md`](workflows.md).

---

## 12. Alternative Order Creation Assistant

**Transport name** `purchase.requisition.create.alternative`, a transient record.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Original Order (`origin_po_id`) | Many-to-one to Purchase Order | The request for quotation the alternatives compete with. Defaulted from the record the user started from. |
| Vendor (`partner_ids`) | Many-to-many to Partner | Required. One alternative request is created per selected vendor. |
| Copy Products (`copy_products`) | Boolean | Default true. When true, the product lines of the original request are copied with the same quantities and units. |
| Warning Messages (`purchase_warn_msg`) | Long text | Computed, not stored, visible only with the purchase-warnings privilege. Concatenates, per selected vendor, the vendor's own purchase warning (or its parent company's when the vendor has none) as *"Warning for the vendor name:"* followed by the message; and, when products are copied, one *"Warning for the product name:"* block per product of the original request that carries a purchase line warning. |

---

## 13. Alternative Order Warning Assistant

**Transport name** `purchase.requisition.alternative.warning`, a transient record.

Shown when a request for quotation that belongs to an alternative group is confirmed while at
least one sibling is still open.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Orders to Confirm (`po_ids`) | Many-to-many to Purchase Order | Join table `warning_purchase_order_rel`. The orders the user asked to confirm. |
| Alternatives (`alternative_po_ids`) | Many-to-many to Purchase Order | Join table `warning_purchase_order_alternative_rel`. The siblings still in draft, sent or to-approve status. |

Two operations: keep the alternatives, or cancel them. Both then confirm the chosen orders
with the alternative check suppressed.

---

## 14. Fields added to entities owned by other domains

### 14.1 Company (`res.company`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order Modification (`po_lock`) | Selection | Default `edit`. Values: `edit` = "Allow to edit purchase orders", `lock` = "Confirmed purchase orders are not editable". When `lock`, approval also sets the order's locked flag. |
| Levels of Approvals (`po_double_validation`) | Selection | Default `one_step`. Values: `one_step` = "Confirm purchase orders in one step", `two_step` = "Get 2 levels of approvals to confirm a purchase order". |
| Double Validation Amount (`po_double_validation_amount`) | Monetary in the company currency | Default 5000. The total above which a second approval is required when the policy is two steps. |
| Days to Purchase (`days_to_purchase`) | Decimal | Present when inventory is installed. The number of days the organisation needs to turn a need into a confirmed order; added to the lead time when the scheduler computes when a purchase must start. |

### 14.2 Partner (`res.partner`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Supplier Currency (`property_purchase_currency_id`) | Many-to-one to Currency, company-dependent | The currency used when buying from this partner. |
| Purchase Order Count (`purchase_order_count`) | Integer | Computed, not stored, readable only with the purchase-user privilege. Counts the partner's own orders **plus** the orders of every descendant contact, rolled up the parent chain. Zero for a reader without the privilege. |
| Message for Purchase Order (`purchase_warn_msg`) | Long text | Free warning text surfaced on orders and bills for this partner and for its children through the parent. |
| Receipt Reminder (`receipt_reminder_email`) | Boolean, company-dependent | Enables the automatic reminder for orders placed with this partner. |
| Days Before Receipt (`reminder_date_before_receipt`) | Integer, company-dependent | How many days before the expected arrival the reminder is sent. The platform ships a fallback value of 1. |
| Buyer (`buyer_id`) | Many-to-one to User | The default buyer for orders placed with this partner; also used as the grouping key when the buy rule looks for an existing draft order. |
| Purchase Lines (`purchase_line_ids`) | One-to-many to Purchase Order Line | Present when inventory is installed. Every purchase line whose stored partner is this partner. |
| On-Time Delivery Rate (`on_time_rate`) | Decimal | Present when inventory is installed. Computed, not stored. Percentage of quantity received on or before the promised date over the lookback window. A value of −1 means "no data". The window in days comes from a system parameter and defaults to 365. |
| Group Requests for Quotation (`group_rfq`) | Selection | Present when inventory is installed. Required, default `default`. Values: `default` = "On Order", `day` = "Daily", `week` = "Weekly", `all` = "Always". Decides how procurement needs for this vendor are merged onto one request for quotation. |
| Week Day (`group_on`) | Selection | Present when inventory is installed. Required, default `default`. Values: `default` = "Expected Date", `1` to `7` = Monday to Sunday. The target week day when weekly grouping is used. |
| Suggestion Basis (`suggest_based_on`) | Text | Present when inventory is installed. Default `30_days`. Remembers the basis the buyer last used in the suggestion panel. |
| Suggestion Days (`suggest_days`) | Integer | Present when inventory is installed. Default 7. The horizon in days used by the suggestion panel. |
| Suggestion Percentage (`suggest_percent`) | Integer | Present when inventory is installed. Default 100. The fraction of the computed need to suggest. |

### 14.3 Product Template (`product.template`) and Product Variant (`product.product`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Control Policy (`purchase_method`) | Selection on the template | Computed, stored, overridable, precomputed. Values: `purchase` = "On ordered quantities", `receive` = "On received quantities". Rule: a service always gets `purchase`; anything else gets the default value configured for the field, which the platform ships as `receive`. |
| Message for Purchase Order Line (`purchase_line_warn_msg`) | Long text on the template | Free warning text surfaced on order lines, bill lines and the alternative-creation assistant. |
| Purchased (`purchased_product_qty`) | Decimal at the product-unit precision | Computed, not stored, on both template and variant. For a variant: the total ordered quantity, converted to the product reference unit and rounded to that unit, across lines of confirmed orders whose confirmation date falls within the last year. For a template: the rounded sum over its variants, including archived ones. |
| In this order (`is_in_purchase_order`) | Boolean on the variant | Computed, not stored, depends on an order supplied by the caller. True when the product already appears on that order. Searchable. |
| Purchase Lines (`purchase_order_line_ids`) | One-to-many on the variant | Present when inventory is installed. Every purchase line for this variant. |
| Monthly Demand (`monthly_demand`) | Decimal on the variant | Present when inventory is installed. Computed, not stored. Historical outbound demand per month; the window and the divisor depend on the basis the caller supplies. |
| Suggested Quantity (`suggested_qty`) | Integer on the variant | Present when inventory is installed. Computed, not stored, searchable. See the formula in [`calculations.md`](calculations.md). |
| Suggested Estimated Price (`suggest_estimated_price`) | Decimal on the variant | Present when inventory is installed. Computed, not stored. The suggested quantity multiplied by the discounted vendor price for that quantity, or by the product cost when no vendor price qualifies. |

### 14.4 Vendor Pricelist Entry (`product.supplierinfo`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Agreement (`purchase_requisition_id`) | Many-to-one to Purchase Agreement, related, read-only | The agreement of the line that published this entry. |
| Agreement Line (`purchase_requisition_line_id`) | Many-to-one to Purchase Agreement Line | Indexed when not empty. Set on entries published by a blanket order. |

Two behaviours are added: selecting a partner on an entry defaults the entry's currency to that
partner's supplier currency, or to the active company's currency when the partner has none;
and, when a candidate entry is filtered for an order, the company used for the filter is the
order's company rather than the ambient one. In addition, when the candidate entries of a
product are prepared for an order that supports agreements, entries published by an agreement
are kept only if they belong to **that** order's agreement.

### 14.5 Journal Entry (`account.move`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Auto-complete (`purchase_vendor_bill_id`) | Many-to-one to Purchases and Bills Union Entry, not stored | A transient control. Choosing a bill row copies that bill; choosing an order row loads that order's lines. Always cleared at the end of the operation. |
| Purchase Order (`purchase_id`) | Many-to-one to Purchase Order, not stored | The second transient control, which loads one order's lines directly. Also always cleared. |
| Purchase Order Count (`purchase_order_count`) | Integer | Computed, not stored. The number of distinct orders reached through the bill's lines. |
| Purchase Order Name (`purchase_order_name`) | Text | Computed, not stored. The single source order's display name when there is exactly one, otherwise empty. |
| Matched (`is_purchase_matched`) | Boolean | Computed, not stored. False as soon as one product line of the bill has no purchase order line; true otherwise, including when the bill has no product lines at all. |
| Purchase Warning (`purchase_warning_text`) | Long text | Computed, not stored. Empty unless the purchase-warnings privilege is granted and the document is a vendor bill. Same composition rule as on the order, using the bill's partner and the products of its lines. |

### 14.6 Journal Item (`account.move.line`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order Line (`purchase_line_id`) | Many-to-one to Purchase Order Line | Indexed when not empty. Not copied when the bill is duplicated by the generic copy, but explicitly re-copied by the business copy used for reversals and corrections. Deleting the purchase line clears the reference rather than deleting the journal item. |
| Purchase Order (`purchase_order_id`) | Many-to-one to Purchase Order, related, read-only | The order of the linked line. |
| Down Payment (`is_downpayment`) | Boolean | Marks a bill line that settles an advance. |
| Purchase Line Warning (`purchase_line_warn_msg`) | Long text | Computed, not stored. The product's purchase line warning when the privilege is granted. |

The analytic distribution a bill line derives from its context is extended with the analytic
distribution of the linked purchase order line.

### 14.7 Tax (`account.tax`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order Lines (`purchase_order_line_ids`) | Many-to-many to Purchase Order Line, read-only, not copied | The reverse of the line-to-tax relation, stored in `account_tax_purchase_order_line_rel`. |

A tax is additionally reported as "used" when at least one purchase order line references it,
so that the interface will not offer to delete it silently.

### 14.8 Stock Move (`stock.move`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Purchase Order Line (`purchase_line_id`) | Many-to-one to Purchase Order Line, read-only | Indexed when not empty. Deleting the purchase line clears the reference. Set on every move the order generates. |
| Created Purchase Order Lines (`created_purchase_line_ids`) | Many-to-many to Purchase Order Line | Join table `stock_move_created_purchase_line_rel`, columns `move_id` and `created_purchase_line_id`, not copied. The purchase lines this move caused to be created, used for make-to-order chaining. Cleared when moves are merged. |

Behaviours added to moves: both new fields join the set of fields that must agree before two
moves may be merged; the packaging unit of a purchase-generated move is the purchase line's
unit; the move description is prefixed with the attribute values that do not create variants
and with the vendor's product code and product name in square brackets when they are not
already present; a move's source document is its purchase order; and a move is recognised as a
**purchase return** when its destination is a vendor location, or when it is the return of a
move whose destination is a vendor location, or when it is the return of a move to the
inter-company transit location.

### 14.9 Stock Rule (`stock.rule`)

The action selection gains the value `buy`, labelled "Buy". A rule with this action produces a
request for quotation rather than a stock move. Its mechanics — the vendor selection, the
grouping domain, the merging of procurements and the creation or extension of the order — are
specified in [`workflows.md`](workflows.md) and in
[`../replenishment-and-procurement/README.md`](../replenishment-and-procurement/README.md).

### 14.10 Other extensions

- **Stock Transfer.** Gains a link back to the purchase order and a counter, so that a receipt
  can offer a shortcut to its order.
- **Stock Reference.** Gains the reverse many-to-many to purchase orders, sharing the join
  table `stock_reference_purchase_rel`.
- **Replenishment Assistant Mixin.** Gains a vendor field and a flag that reveals it exactly
  when the chosen route contains a buy rule.
- **Analytic Account.** Gains a purchase order counter: the number of purchase orders whose
  bill lines carry an analytic line on this account's root plan column, and an action listing
  them.
- **Analytic Plan Applicability.** Gains the business domain value `purchase_order`, labelled
  "Purchase Order", removed along with the purchasing capability.
- **Sales Order Line.** Gains a link to the purchase line generated for a service configured to
  be re-purchased; see section 15.
- **Repair Order.** Gains a purchase order counter and an action listing the orders that supply
  the parts.
- **Project.** Gains a purchase order counter reaching the project's analytic account.

---

## 15. Cross-domain entity notes

### 15.1 Service purchase from a sales order

When a product is a service whose re-invoicing configuration says it must be purchased, a
confirmed sales order line creates or extends a request for quotation for that service. The
sales line keeps a reference to the purchase line it generated, and the purchase line's
delivered quantity feeds back into the sales line's delivered quantity. The detailed mapping
is in [`workflows.md`](workflows.md).

### 15.2 Kits

When a purchased product is a kit, the receipt contains moves for the kit's components rather
than for the kit itself. The received quantity of the purchase line is then derived from the
component moves: the line is considered received in proportion to the smallest complete set of
components received. The arithmetic is in [`calculations.md`](calculations.md).

### 15.3 Product grids

When the product grid capability is installed, a request for quotation for a product with
several variant-creating attributes can be filled through a grid: one cell per variant
combination, each cell holding a quantity. Saving the grid creates or updates one purchase
order line per non-empty cell.

### 15.4 Structured electronic order documents

When the electronic order document capability is installed, printing a purchase order embeds a
structured machine-readable representation of the order inside the produced document file, one
attachment per configured builder. The purchase order itself exposes the list of builders; the
base platform ships an empty list and the capability fills it.
