# Sales — Interfaces

Everything the domain exposes: navigation, screens, buttons, filters, named operations, network
addresses, printable documents, messages and integration contracts.

---

## 1. Navigation

The menu tree is given in [configuration.md](configuration.md), section 15. This section states
what each entry opens.

| Menu path | Opens | Default filter |
|---|---|---|
| Sales ▸ Orders ▸ Quotations | The order list restricted to nothing, with the quotation search view | "My Quotations" |
| Sales ▸ Orders ▸ Orders | The order list with the order search view | "Sales Orders" (status is `sale`) |
| Sales ▸ Orders ▸ Sales Teams | The sales-team dashboard | — |
| Sales ▸ Orders ▸ Customers | The customer list of the receivables domain | — |
| Sales ▸ To Invoice ▸ Orders to Invoice | Orders whose invoice status is "To Invoice"; creation disabled | — |
| Sales ▸ To Invoice ▸ Orders to Upsell | Orders whose invoice status is "Upselling Opportunity"; creation disabled | — |
| Sales ▸ Products ▸ Products | The product-template list filtered to sellable products | — |
| Sales ▸ Products ▸ Product Variants | The product-variant list filtered to sellable products | — |
| Sales ▸ Products ▸ Pricelists | The price-list configuration | — |
| Sales ▸ Reporting ▸ Sales | The sales analysis, all dimensions | — |
| Sales ▸ Reporting ▸ Salespersons | The sales analysis grouped by salesperson | — |
| Sales ▸ Reporting ▸ Products | The sales analysis grouped by product | — |
| Sales ▸ Reporting ▸ Customers | The sales analysis grouped by customer | — |
| Sales ▸ Configuration ▸ Settings | The settings screen of the sales application | — |
| Sales ▸ Configuration ▸ Sales Teams | The sales-team configuration list | — |
| Sales ▸ Configuration ▸ Sales Orders ▸ Tags | The sales-tag list | — |
| Sales ▸ Configuration ▸ Products ▸ … | Attributes, combo choices, categories, tags, units | — |
| Sales ▸ Configuration ▸ Online Payments ▸ … | Providers, payment methods, tokens, transactions | — |
| Sales ▸ Configuration ▸ Activities ▸ … | Activity types and activity plans addressed to orders | — |

---

## 2. Window actions

| Action | Entity | Views offered | Domain | Context |
|---|---|---|---|---|
| Sales Orders | Sales Order | list, kanban, form, calendar, pivot, graph, activity | none | the "Sales Orders" filter pre-selected |
| Quotations (with onboarding) | Sales Order | list, kanban, form, calendar, pivot, graph, activity | none | the "My Quotations" filter pre-selected; the list and kanban views carry the onboarding renderers |
| Quotations | Sales Order | list, kanban, form, calendar, pivot, graph, activity | none | the "My Quotations" filter pre-selected |
| Orders to Invoice | Sales Order | list, form, calendar, graph, pivot, kanban, activity | invoice status equals "to invoice" | creation disabled |
| Orders to Upsell | Sales Order | list, form, calendar, graph, pivot, kanban, activity | invoice status equals "upselling" | creation disabled |
| Sales Analysis (all) | Sales Analysis | graph, pivot, list | none | — |
| Sales Analysis by salesperson / product / customer | Sales Analysis | graph, pivot | none | the grouping pre-selected |
| Sales Analysis for a team | Sales Analysis | graph, pivot | the team | opened from the team dashboard button |
| Add or remove followers | Followers dialogue | form | — | pre-filled with the order entity and the selected records |
| Create Invoice | Advance Payment Invoice dialogue | form, as a dialogue | — | optionally pre-selects the percentage method |
| Discount | Discount dialogue | form, as a dialogue | — | the active order |
| Cancel orders | Mass Cancel dialogue | form, as a dialogue | — | the selected orders |
| Generate payment link | Payment Link dialogue | form, as a dialogue | — | the active order |
| Accrued Revenue Entry | Accrued Orders dialogue | form, as a dialogue | — | the selected orders or order lines |

---

## 3. The order form

### 3.1 Header buttons

Listed in display order, with the exact condition under which each is shown.

| Button | Visible when | What it does |
|---|---|---|
| Capture Transaction | the order has authorized transactions | Captures every linked transaction. |
| Void Transaction | the order has authorized transactions | Voids every authorized transaction. Asks for confirmation: "Are you sure you want to void the authorized transaction? This action can't be undone." |
| Create Invoice (primary) | the invoice status is "to invoice" | Opens the advance-payment dialogue. |
| Create Invoice (secondary) | the invoice status is "no" and the status is `sale` | Opens the same dialogue with the percentage method pre-selected. This is the path for taking an advance on an order that has nothing invoiceable yet. |
| Send (primary) | the status is `draft` | Opens the message composer with the quotation template, asking for analytic validation and for the document-layout check. |
| Send PRO-FORMA Invoice (primary) | the status is `draft` and the order has no invoice; requires the pro-forma group | Opens the composer in pro-forma mode. |
| Confirm (primary) | the status is `sent` | Runs the confirmation algorithm. |
| Print | the status is not `sale` | Renders the quotation document. |
| Confirm (secondary) | the status is `draft` | Runs the confirmation algorithm. |
| Send PRO-FORMA Invoice (secondary) | the status is not `draft` and the order has no invoice; requires the pro-forma group | As above. |
| Send (secondary) | the status is `sent` or `sale` | As above. |
| Unlock | the order is locked; requires the Administrator group | Clears the lock. Shown even when the lock feature is disabled, so that orders locked by an external synchronisation can be released. |
| Preview | always | Opens the customer portal page of the order. |
| Cancel | the status is `draft`, `sent` or `sale`, the record exists and is not locked | Cancels the order. Asks for confirmation: "Are you sure you want to cancel this order? This may affect related documents or processes." |
| Set to Quotation | the status is `cancel` | Returns the order to `draft` and clears the signature. |
| Lock | the status is `sale` and the order is not locked; requires both the lock feature group and the Administrator group | Sets the lock. Help text: "If the sale is locked, you can not modify it anymore. However, you will still be able to invoice or deliver." |

The status bar shows the three forward statuses `draft`, `sent`, `sale`; the cancelled status
appears only when the order is in it.

### 3.2 Smart buttons

| Button | Shows | Opens |
|---|---|---|
| Invoices | the invoice count | The customer invoices and credit notes of the order. |
| Delivery | the transfer count | The transfers of the order (inventory coupling). |
| Purchase | the number of generated purchase requests | The purchase requests (purchasing coupling). |
| Projects / Tasks | the project and task counts | The generated projects and tasks (project coupling). |
| Expenses | the expense count | The re-invoiced expenses (expense coupling). |

### 3.3 The line editor

Columns, in order: sequence handle, product, description, analytic distribution, ordered quantity,
delivered quantity, invoiced quantity, unit, unit price, taxes, discount percentage, subtotal, and
the availability widget for goods lines. The discount column is hidden unless the discount feature
group is enabled. Section, subsection and note rows span the whole width.

Buttons inside the editor:

| Button | Effect |
|---|---|
| Add a product | Appends a priced line. |
| Add a section | Appends a section header. |
| Add a subsection | Appends a subsection header. |
| Add a note | Appends a note. |
| Catalog | Opens the product catalogue for this order. |
| Update Prices | Recomputes prices and discounts on all non-display lines. Shown while the "price list changed" flag is raised. |
| Update Taxes | Re-maps taxes on all non-display lines. Shown while the "fiscal position changed" flag is raised. |
| Discount (cog menu) | Opens the discount dialogue. |
| Upload document (cog menu) | Creates orders from attached structured documents. |

### 3.4 Warnings surfaced on the form

| Banner | Source |
|---|---|
| Sale warning | the assembled customer and product warnings |
| Credit limit warning | the receivables domain's credit message |
| Duplicate orders | the list of duplicate orders of the same customer |
| Archived products | raised when a line references an inactive product |
| Expired | raised when the quotation's validity date has passed |
| Delay alert | raised when a transfer of the order carries a delay alert (inventory coupling) |

---

## 4. Search view

### 4.1 Filters

| Filter | Meaning |
|---|---|
| My Orders / My Quotations | the salesperson is the reader |
| Quotations | the status is `draft` or `sent` |
| Sales Orders | the status is `sale` |
| To Invoice | the invoice status is "to invoice" |
| To Upsell | the invoice status is "upselling" |
| Order Date | a date range on the order date |
| Create Date | a date range on the creation date |
| My Activities, Late Activities, Today Activities, Future Activities | the generic activity filters, hidden by default |

### 4.2 Groupings

Salesperson, Customer, Order Date, Payment Method. Couplings add Delivery Status, Sales Team and
Campaign.

### 4.3 Text search

The search box matches the order reference; when the reading context asks for it, it also matches
the customer name.

---

## 5. Other screens

### 5.1 Quotation template form

Name, active flag, company, quotation duration in days, online signature, online payment,
prepayment percentage, confirmation message template, invoicing journal, the line editor (with
sections, subsections, notes and the optional flag) and the terms text.

### 5.2 Sales team dashboard

A kanban card per team showing: the team name and leader, the invoiced amount of the current month
against the invoicing target with a progress bar, the number of non-cancelled orders, and a
primary button labelled "Sales Analysis" that opens the analysis filtered to that team. The target
is editable inline; writing it rounds the typed value to the nearest integer.

### 5.3 Advance payment dialogue

Radio buttons for the three methods; the percentage field for the percentage method; the amount
field and its currency for the fixed method; the "deduct down payments" switch, shown only for the
regular method when the orders already carry advances; the "consolidated billing" switch, shown
only when several orders are selected; the "already invoiced" figure; and a warning when any of the
selected orders already has a draft invoice, with a link that opens those drafts.

### 5.4 Discount dialogue

Radio buttons for the three kinds; a percentage field for the two percentage kinds; an amount field
with its currency for the fixed kind.

### 5.5 Mass cancel dialogue

The number of selected orders and, when at least one is confirmed, a warning that confirmed orders
are included.

---

## 6. Named operations

These are the operations a remote caller may invoke on the entities of this domain. Each is given
with its inputs and its result. Names are reproduced exactly because external callers depend on
them.

### 6.1 On the Sales Order (`sale.order`)

| Operation | Inputs | Result | Notes |
|---|---|---|---|
| `action_confirm` | none | true | Runs the confirmation algorithm. Honours the context values "send email" and "include signature". |
| `action_cancel` | none | true | Cancels; refuses locked orders. |
| `action_draft` | none | the write result | Resets cancelled or sent orders to `draft`. |
| `action_quotation_sent` | none | the write result | Marks draft orders as sent. |
| `action_quotation_send` | none | a window action | Opens the message composer. |
| `action_lock` / `action_unlock` | none | none | Sets or clears the lock. |
| `action_preview_sale_order` | none | an address action | Opens the customer portal page. Read-only operation. |
| `action_update_prices` | none | none | Recomputes prices and discounts, then posts a note. |
| `action_update_taxes` | none | none | Re-maps taxes, then posts a note. |
| `action_view_invoice` | optionally a set of invoices | a window action | Opens the invoices. Read-only operation. |
| `action_open_discount_wizard` | none | a window action | Opens the discount dialogue. Read-only operation. |
| `action_open_business_doc` | none | a window action | Opens this order's form. Read-only operation. |
| `action_view_delivery` | none | a window action | Opens the transfers (inventory coupling). |
| `action_view_purchase_orders` | none | a window action | Opens the generated purchase requests (purchasing coupling). |
| `payment_action_capture` | none | the capture result | Captures the linked transactions; checks the caller's rights on the record set first. |
| `payment_action_void` | none | none | Voids the authorized transactions; same rights check. |
| `get_portal_last_transaction` | none | the last transaction | Evaluated with elevated rights. |
| `create_document_from_attachment` | a list of attachment identifiers | a window action listing the created orders, titled "Generated Orders" | Fails with "No attachment was provided" on an empty list. |
| `get_import_templates` | none | one entry: the label "Import Template for Quotations" and the spreadsheet path | |
| `_cron_send_pending_emails` | none | none | The scheduled job. Invoked on the entity, not on a record. |

### 6.2 On the Sales Order Line (`sale.order.line`)

| Operation | Inputs | Result |
|---|---|---|
| `action_add_from_catalog` | the order identifier in the context | a window action opening the catalogue for that order |

### 6.3 On the advance payment dialogue (`sale.advance.payment.inv`)

| Operation | Inputs | Result |
|---|---|---|
| `create_invoices` | none | a window action opening the created invoices |
| `view_draft_invoices` | none | a window action listing the draft invoices of the selected orders |

### 6.4 On the discount dialogue (`sale.order.discount`)

| Operation | Inputs | Result |
|---|---|---|
| `action_apply_discount` | none | none; the order's lines are modified |

### 6.5 On the accrued orders dialogue (`account.accrued.orders.wizard`)

| Operation | Inputs | Result |
|---|---|---|
| `create_entries` | none | a window action listing the accrual entry and its reversal |

### 6.6 On the Sales Team (`crm.team`)

| Operation | Inputs | Result |
|---|---|---|
| `update_invoiced_target` | the new target | the write result; the value is rounded to the nearest integer |
| `action_primary_channel_button` | none | the sales analysis action when invoked inside the sales application |

### 6.7 On the Payment Transaction (`payment.transaction`)

| Operation | Inputs | Result |
|---|---|---|
| `action_view_sales_orders` | none | a window action opening the linked orders. Read-only operation. |
| `_cron_send_invoice` | none | none. The scheduled job. |

### 6.8 On the Customer Invoice (`account.move`)

| Operation | Inputs | Result |
|---|---|---|
| `action_view_source_sale_orders` | none | a window action opening the orders behind the invoice's lines |

### 6.9 On the Product Template and Product Variant

| Operation | Inputs | Result |
|---|---|---|
| `action_view_sales` | none | the sales analysis filtered to this product, with the ordered-quantity measure pre-selected and the order-date filter and grouping applied. Read-only operation. |

### 6.10 Catalogue contract

| Operation | Inputs | Result |
|---|---|---|
| `_update_order_line_info` on the order | the product identifier, the quantity, optionally the section identifier | the discounted unit price of the resulting line, or the price-list price when no line results |
| `_get_product_catalog_order_data` on the order | the products | per product: the price-list price for a quantity of one in the order currency at the order date, plus the product's sale warning for readers in the warning group |
| `_get_product_catalog_lines_data` on the lines | none | the quantity, the price, a read-only flag, the unit display name and — with the inventory coupling — the delivered quantity |

The read-only flag is true when the order is read-only (cancelled or locked), when the line is a
combo item, or when several lines share the product.

---

## 7. Network addresses

All addresses below are served by the customer portal. "Token" means the document access token
passed either as a query value or in the request body.

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/quotes` and `/my/quotes/page/<page number>` | plain request | signed-in user | Lists the reader's quotations — orders of the reader's commercial customer tree whose status is `sent`. Not read-only, because opening the list ensures an access token on each listed order. Stores the first hundred identifiers in the session for the previous/next navigation. |
| `/my/orders` and `/my/orders/page/<page number>` | plain request | signed-in user | Lists the reader's confirmed orders (status `sale`), with the same token behaviour and session history. |
| `/my/orders/<order identifier>` | plain request | public | Shows one order. Accepts the query values `access_token`, `report_type` (`html`, `pdf` or `text`), `message`, `download`, `payment_amount` and `amount_selection` (`down_payment` or `full_amount`). Logs the "quotation viewed" note under the conditions of [workflows.md](workflows.md), section 5.1. |
| `/my/orders/<order identifier>/accept` | remote-procedure call over the web | public | Records the signature and, when payment is not also required, confirms the order. Inputs: the token, the signer name, the signature image. Output: either an error text, or an instruction to reload at a given address. |
| `/my/orders/<order identifier>/decline` | plain request, method POST | public | Cancels the order and posts the decline text. Inputs: the token, the decline message. |
| `/my/orders/<order identifier>/document/<document identifier>` | plain request | public, read-only | Streams a product document attached to the order as a download. |
| `/my/orders/<order identifier>/download_edi` | plain request | public | Returns the structured order document with the extensible-markup-language content type, its length and a download file name. |
| `/my/orders/<order identifier>/transaction` | remote-procedure call over the web | public | Creates a draft payment transaction for the order and returns the values the payment form needs. |

The portal home page shows two counters, "quotation count" and "order count", computed with the
same two domains and suppressed to zero when the reader may not read orders.

Sorting on the two list pages offers a single criterion, "Order Date", descending.

---

## 8. Printable documents

| Document | Entity | Output | File name | Restricted to |
|---|---|---|---|---|
| Quotation / Order | Sales Order | portable document format, rendered from a markup template | "Quotation - *order reference*" while the status is `draft` or `sent`, otherwise "Order - *order reference*" | everyone with access to the order |
| PRO-FORMA Invoice | Sales Order | portable document format | "PRO-FORMA - *order reference*" | the pro-forma group |

Both are offered as contextual print actions on the order.

### 8.1 Content of the order document, section by section

1. **Header**: the company's external layout — logo, company name, address, tax registration
   identifier, and the report title, which is the type name ("Quotation" or "Sales Order") followed
   by the order reference.
2. **Addresses**: the invoice address, and the delivery address when it differs.
3. **Document data**: the order reference, the order date (labelled "Quotation Date" before
   confirmation and "Order Date" after), the expiration date for a quotation, the customer
   reference when present, the salesperson, and the payment term.
4. **Lines**: for each line to report (section 8.2), the description, the quantity with its unit,
   the unit price, the discount percentage when the discount feature is on and the value is
   non-zero, the taxes, and the subtotal. Sections print as a heading with, when their prices are
   not collapsed, a section total; subsections print as a sub-heading; notes print as free text.
5. **Totals**: the untaxed amount, one line per tax group, and the total — assembled from the
   structured totals value, so that the early-payment-discount presentation and the tax-group
   breakdown match the screen exactly.
6. **Signature**: when the rendering context asks for it and the order carries a signature, the
   signature image with the signer name and the signature instant.
7. **Terms**: the terms and conditions text of the order.
8. **Fiscal footnotes**: any legal mentions the localisation adds.

### 8.2 Which lines the document prints

1. Advance-invoice lines are shown only when their state is empty (that is, at least one advance
   invoice has been posted); the advance section line is shown only when at least one such line
   exists.
2. Any other line is shown when it is a section, **or** when neither its parent section nor its
   grandparent section has the "collapse composition" flag.
3. A section with collapsed prices prints its lines without prices and shows the section total.
4. A section with collapsed composition prints only itself and its total.

### 8.3 Structured document attached to the printed file

When the order is rendered alone and at least one structured-document builder is configured, the
produced file is reopened and the structured document produced by each builder is embedded as an
attachment inside it, named by the builder and declared as extensible-markup-language content. The
rule applies to the three order report names.

---

## 9. Messages and notifications

### 9.1 Templates

The four shipped templates and their subject lines are listed in
[configuration.md](configuration.md), section 10. Their bodies address the customer, quote the
order reference and the total, and carry the access button.

### 9.2 The access button on a notification

The label of the button offered to a customer or portal recipient depends on the acceptance state:

| Condition | Label |
|---|---|
| Must sign and must pay, last transaction pending | "View Quotation" |
| Must sign and must pay, otherwise | "Sign & Pay Quotation" |
| Must sign only | "Accept & Sign Quotation" |
| Must pay only, last transaction not pending | "Accept & Pay Quotation" |
| Status `draft` or `sent`, nothing required | "View Quotation" |
| Rendering as a pro-forma document | no button at all for customer, portal, follower and customer-portal recipients |

### 9.3 Subtitles of a notification

The first subtitle is "*order reference* - *customer name*", or just the order reference when the
customer has no name. When the order total is non-zero, a second subtitle carries the total
formatted in the order currency and in the recipient's language. A zero total is omitted, because
storefront orders are created empty.

### 9.4 Notes posted on the thread

| Note | When |
|---|---|
| status tracking notes | every status change, with the subtypes of [configuration.md](configuration.md), section 9 |
| "Quotation viewed by customer *name*" | the first portal view of the day of a quotation by a shared user |
| "Order signed by *name*", with the signed document attached | portal signature |
| the decline text | portal decline |
| "Product prices have been recomputed according to pricelist *link*." | the price-update operation |
| "Product taxes have been recomputed according to fiscal position *link*." | the tax-update operation |
| "Extra line with *product*" | a line added to a confirmed order |
| the quantity-change list | an ordered quantity changed on a confirmed order |
| "*link labelled "Down payment invoice"* has been created" | an advance invoice is created |
| "Invoice *number* paid" | an invoice behind the order becomes fully paid |
| "Accrual entry created on *date*: *link*. And its reverse entry: *link*." | the accrual dialogue |
| an origin link note on each created invoice | the invoicing algorithm |

### 9.5 Activities

| Activity | Raised on | Assigned to | Note |
|---|---|---|---|
| To-do | the order | the order's salesperson, else the customer's salesperson | "Upsell *order link* for customer *customer link*" |
| Warning | a transfer impacted by a quantity decrease | the transfer's responsible | the rendered quantity-decrease explanation |
| Warning | a transfer whose delivery address no longer matches | the acting user | "The delivery address has been changed on the Sales Order From "*old address*" to "*new address*", You should probably update the partner on this document." |
| Warning | a purchase request whose originating sale line lost quantity | the purchase responsible, else the acting user | the rendered quantity-decrease explanation |
| Warning | a purchase request whose originating order was cancelled | the purchase responsible, else the acting user | the rendered cancellation explanation |

Activity plans addressed to the order entity can be configured by the sales administrator.

### 9.6 Message access policy

Posting a message on an order requires only read access, not write access. This is what lets a
portal customer comment on a quotation.

---

## 10. External service integrations

| Integration | Direction | Contract |
|---|---|---|
| Payment providers | outbound and inbound | The order is linked to transactions; the provider's outcome drives the post-processing of [workflows.md](workflows.md), section 6. Each provider declares how the payment communication for an order is built (by document reference or by customer identifier). |
| Structured order documents | outbound | A list of builders; each can export the order as a structured document and name the resulting file. Used by the portal download address and by the attachment embedded in the printed file. |
| Structured document import | inbound | Attachments are decoded into draft orders through the document-import mixin. |
| Print-on-demand production | outbound | On confirmation the order is transmitted to the external production service. |
| Marketplace synchronisation | inbound | Orders are created by the marketplace connectors with their own reference and customer mapping. |
| Text messages | outbound | The order has no telephone field of its own; text messaging falls back to the customer's telephone numbers. |

---

## 11. Import and export

### 11.1 Import

- The order entity offers the quotation import template of [configuration.md](configuration.md),
  section 16.
- During an import, the combo-item field of a line is written after creation rather than as part
  of the creation values, so that the line's linked line already exists when the integrity rule
  runs.
- The line's display type, when supplied, forces the ordered quantity to zero.

### 11.2 Export

- The structured totals value is explicitly excluded from exports, because it is a computed
  presentation structure rather than data.
- The analysis entity is the recommended export surface for reporting: it is flat, read-only and
  already currency-converted.

---

## 12. Screen-only helpers

These exist purely to drive the interface and are never stored.

| Helper | Purpose |
|---|---|
| "has fiscal position changed" | shows the "Update Taxes" button |
| "has price list changed" | shows the "Update Prices" button |
| "has active price list" | hides the price-list field when no price list exists |
| "show availability widget" on a line | shows the forecast popover |
| "delay popover data" and "has late transfer" | draw the delay alert |
| "can edit product" on a line | makes the product cell read-only |
| "unit readonly" on a line | makes the unit cell read-only |
| "is the product configurable" | opens the configurator instead of a plain selection |
| "has archived products" | shows the archived-product banner |
| "is expired" | shows the expiry banner and the expired ribbon |

---

## 13. Client-side contracts

Two dialogues exchange structured data with the server and must be reproduced faithfully.

### 13.1 Product configurator

Given a product template, the order's price list, currency, date and customer, the server returns
per candidate variant: the identifier, the display name, the description, the price (through the
configurator price hook), the applied price-list rule, the attribute lines with their values and
extra prices, whether each value is available, and any optional products to propose. The client
returns the chosen variant, the chosen values of attributes that do not create variants, the
custom texts, the quantity, and the list of optional products to add as linked lines.

### 13.2 Combo configurator

Given a combo product, the server returns the combo choices, their items, each item's product, its
extra price and its configurability. The client returns, for each choice, the selected item: its
product identifier, its combo item identifier, the chosen values of attributes that do not create
variants, and the custom texts. The server then rebuilds the item lines as described in
[entities.md](entities.md), section 2.4, deleting the previous ones, inserting the new ones
immediately after the combo line and shifting the sequence of every later line by the number of
items.
