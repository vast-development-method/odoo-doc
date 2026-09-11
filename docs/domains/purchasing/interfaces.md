# Purchasing — Interfaces

Everything the purchasing domain exposes: the navigation a user sees, the screens and what
each one shows, the operations a client may call by name, the network routes, the printable
documents, the message templates and notifications, the integrations with outside services,
and the import and export formats.

Reproduced identifiers (route paths, operation names, field storage names, selection values)
are given in code font because external contracts depend on them.

> **Reproduced text.** Status labels, button labels, subtype names and message bodies are
> reproduced exactly as the system produces them, because a rebuilt implementation must
> produce the same text. Some shipped strings contain the short form of *request for
> quotation*; that short form appears only inside such reproduced strings and never in this
> specification's own prose. See the conventions in [`README.md`](README.md).

---

## 1. Navigation

The application appears as a top-level entry named **Purchase**, visible to holders of the
purchase user or purchase administrator privilege, placed at sequence 135.

| Menu path | Sequence | Opens | Visible to |
|---|---|---|---|
| Purchase | 135 | — | Purchase user or administrator |
| Purchase ▸ Orders | 1 | — | Same |
| Purchase ▸ Orders ▸ Requests for Quotation | 0 | The requests-for-quotation screen | Same |
| Purchase ▸ Orders ▸ Purchase Orders | 6 | The confirmed-orders screen | Same |
| Purchase ▸ Orders ▸ Vendors | 15 | The supplier list of the accounting domain | Same |
| Purchase ▸ Orders ▸ Purchase Agreements | — | The agreements screen | Purchase user, when the agreements capability is installed |
| Purchase ▸ Products | 5 | — | Same |
| Purchase ▸ Products ▸ Products | 20 | Product templates filtered to purchasable ones | Same |
| Purchase ▸ Products ▸ Product Variants | 21 | Product variants filtered to purchasable ones | Same, and only when product variants are enabled |
| Purchase ▸ Reporting ▸ Purchase | — | The purchase analysis screen | Purchase user |
| Purchase ▸ Reporting ▸ Vendor Delay | — | The vendor delay screen | Purchase user, when inventory is installed |
| Purchase ▸ Configuration | 100 | — | Purchase administrator only |
| Purchase ▸ Configuration ▸ Vendor Pricelists | 1 | Vendor pricelist entries | Purchase administrator |
| Purchase ▸ Configuration ▸ Products | 30 | — | Purchase administrator |
| Purchase ▸ Configuration ▸ Products ▸ Attributes | 1 | Product attributes | Purchase administrator, when variants are enabled |
| Purchase ▸ Configuration ▸ Products ▸ Product Categories | 3 | Product categories | Purchase administrator |
| Purchase ▸ Configuration ▸ Products ▸ Units & Packagings | 10 | Units of measure | Purchase administrator, when units are enabled |
| Purchase ▸ Configuration ▸ Settings | — | The settings screen | Purchase administrator |

---

## 2. Window actions

| Action | Opens | Default screens | Default restriction | Default context |
|---|---|---|---|---|
| Requests for Quotation | Purchase Order | list, kanban, form, pivot, graph, calendar, activity | none | marks the screen as quotation-only, which hides the expected-arrival column |
| Purchase Orders | Purchase Order | list, kanban, form, pivot, graph, calendar, activity | status is `purchase` | none |
| Requests for Quotation (single record) | Purchase Order | form only | none | opens in the main area |
| Purchase History | Purchase Order Line | list, pivot, graph | supplied by the caller | groups by product and restricts to the last 365 days |
| Purchase Analysis | Purchase Analysis Entry | graph, pivot, list | none | none |
| Vendor Delay | Vendor Delay Entry | graph, pivot | none | none |
| Purchase Agreements | Purchase Agreement | list, kanban, form | none | none |
| Bill Matching | Purchase and Bill Line Match Entry | list | the vendor and its commercial parent, the active company, and either this order or no order | none |
| Purchase Matching | Purchase and Bill Line Match Entry | list | the vendor and its commercial parent, the reader's companies narrowed to this bill's company branch, and either this bill or no bill | none |
| Add to Purchase Order | Bill To Purchase Order Assistant | form, as a dialog | — | the commercial partner, the single order when there is one, and a flag saying whether any selected line has a product |
| Create alternative | Alternative Order Creation Assistant | form, as a dialog | — | the originating order |
| What about the alternative Requests for Quotations? | Alternative Order Warning Assistant | form, as a dialog | — | the open siblings and the orders being confirmed |
| Compare Order Lines | Purchase Order Line | list, using the comparison layout | the order and its alternatives, product lines only | groups by product; remembers which order the user came from |
| Accrued Expense Entry | Accrued Orders Assistant | form, as a dialog | — | the selected orders |
| Products | Product Template | kanban, list, form, activity | none | filters to purchasable products and marks the screen as the purchasing product screen |
| Product Variants | Product Variant | list, kanban, form, activity | none | filters to purchasable products |

---

## 3. Server actions bound to menus

| Action | Bound to | Where it appears | Visible to | Effect |
|---|---|---|---|---|
| Share | Purchase Order | The form's action menu | Everyone who can read the record | Produces a portal share link for the order. |
| Send Reminder | Purchase Order | The form's action menu | Holders of the receipt-reminder privilege | Opens the email composer pre-loaded with the reminder template for the single selected order. |
| Merge RFQs | Purchase Order | The list's action menu | Holders of the accounting-invoicing privilege | Runs the merge algorithm on the selection. |
| Confirm request for quotation | Purchase Order | The list and kanban action menus | Everyone who can write the record | Confirms every selected order; when the confirmation returns the alternative question, that dialog is opened. |
| Add/Remove Followers | Purchase Order | The list and kanban action menus | Everyone who can write the record | Opens the follower-editing assistant for the selection. |
| Accrued Expense Entry | Purchase Order | The form and list action menus | Holders of the accounting-user privilege | Opens the accrual assistant. |
| Purchase Order (print) | Purchase Order | The print menu | Every internal user | Produces the purchase order document. |
| Request for Quotation (print) | Purchase Order | The print menu | Every internal user | Produces the quotation document. |

---

## 4. The purchase order form

### 4.1 Header buttons, in the order they appear

| Button | Shown when | Privilege | Effect |
|---|---|---|---|
| Send request for quotation (highlighted) | status is `draft` | — | Opens the email composer with the request-for-quotation template. |
| Confirm Order (highlighted) | status is `sent` | — | Confirms; requests analytic validation. |
| Approve Order (highlighted) | status is `to approve` | Purchase administrator | Approves. |
| Send request for quotation | status is `sent` | — | Same as above, not highlighted. |
| Confirm Order | status is `draft` | — | Confirms. |
| Upload bill | status is `purchase` | — | A file-upload control that creates a vendor bill from the dropped documents. |
| Send purchase order | status is `purchase` | — | Opens the email composer with the purchase-order template. |
| Acknowledge | status is `purchase` and the order is not acknowledged | — | Sets the acknowledged flag. |
| Set to Draft | status is `cancel` | — | Returns the order to draft. |
| Print | status is not `purchase` | Any internal user | Produces the quotation document and moves a draft order to sent. |
| Print | status is `purchase` | Any internal user | Produces the purchase order document. |
| Cancel | status is draft, sent, to approve or purchase, and the order is not locked | — | Cancels. |
| Lock | the order is not locked, status is `purchase`, and the company policy is *lock* | — | Sets the locked flag. |
| Unlock | the order is locked | Purchase administrator | Clears the locked flag. |

A status bar shows the progression draft → sent → purchase; the to-approve and cancelled
statuses appear only when the order is in them.

### 4.2 Banners

| Banner | Shown when | Content |
|---|---|---|
| Purchase warning | The purchase warning text is not empty | The accumulated vendor and product warnings, in bold. |
| Duplicate warning | The status is draft and duplicates were detected | *Warning: this order might be a duplicate of* followed by buttons opening each duplicate. |
| Locked badge | The order is locked | A pill reading *Locked* with a padlock. |

### 4.3 Statistic buttons

| Button | Shown when | Opens |
|---|---|---|
| Bill Matching | The vendor is set, the status is `purchase`, the vendor has bills, the billing status is not *Fully Billed*, and the reader holds the accounting-invoicing privilege | The matching screen scoped to this order. |
| Vendor Bills, with the bill count | At least one bill and the status is not draft, sent or to approve | The bills of the order. |
| Price Comparison | At least one product of the order also appears on another confirmed order | The purchase history for those products. |
| Receipt, with the transfer count | Inventory is installed and at least one transfer exists | The transfers of the order. |
| Sources Sale Orders | The sales bridge is installed and at least one source sales order exists; visible to salespeople | The originating sales orders. |
| Manufacturing Source | The manufacturing bridge is installed; visible to manufacturing users | The manufacturing orders that needed the goods. |
| Repair Source | The repair bridge is installed; visible to inventory users | The repair orders that needed the parts. |

### 4.4 Header fields

Two columns. Left: vendor (searchable by name, tax identification number, email or internal
reference, restricted to suppliers, read-only once the status is cancelled or purchase), vendor
reference, currency (shown only in a multi-currency installation, read-only once the status is
cancelled or purchase). Right: order deadline (hidden once confirmed), confirmation date (shown
only once confirmed), expected arrival, and the reminder block.

The reminder block — visible only to holders of the receipt-reminder privilege — shows a
checkbox, the words *Ask confirmation*, and, when the checkbox is ticked, a small number field
followed by *day(s) before* and a preview button whose tooltip is *Preview the reminder email by
sending it to yourself.*

When inventory is installed the right column also shows the operation type (*Deliver To*) and,
when the operation type's destination is a customer location, the dropship address.

### 4.5 The Products page

An editable list of order lines with a drag handle, using the section-and-note layout with
subsections enabled, limited to 200 rows, read-only when the order is cancelled or locked, and
decorated in warning colour when a line carries a product warning.

Three creation controls plus one button: *Add a product*, *Add a section*, *Add a note*, and
*Catalog*.

| Column | Notes |
|---|---|
| Product | Restricted to purchasable products of the order's company or of no company; read-only once the status is purchase, to approve or cancelled, or on a down-payment line; required on a line that is neither a display line nor a down payment. |
| Description | Free text, section-aware. |
| Expected Arrival | Hidden by default; required on a line that is neither a display line nor a down payment. |
| Analytic Distribution | Shown only to holders of the analytic privilege; declares the business domain *purchase order*. |
| Quantity | Read-only on a down-payment line. |
| Received | Shown only once the status is purchase; editable only when the method is manual. |
| Billed | Shown only once the status is purchase; always read-only. |
| Unit | Shown only when units are enabled; read-only once the status is purchase or cancelled. |
| Unit Price | Read-only once the line has a non-zero billed quantity. |
| Taxes | Restricted to purchase taxes of the order's company chain, of the order's tax country, and active. |
| Disc.% | Hidden by default; read-only once the line has a non-zero billed quantity. |
| Amount | The line subtotal. |

Below the list: the tax totals block, and the terms and conditions.

### 4.6 Other pages

| Page | Content |
|---|---|
| Other Information | Buyer, company, payment terms, fiscal position, incoterm and incoterm location, and — when the agreements capability is installed — the agreement and the alternatives. |
| Attachments and chatter | The message thread, the followers and the scheduled activities. |

---

## 5. Other purchase order screens

| Screen | What it shows |
|---|---|
| Key-figure list | The default list for requests for quotation: reference, vendor, buyer, company, order deadline, vendor reference, total, total in company currency (hidden by default), billing status as a coloured badge, expected arrival (hidden on the quotation-only screen), and the status. |
| Plain list | The default list for confirmed orders. |
| Kanban with dashboard | Cards grouped by status, with a dashboard band at the top; see section 6. |
| Kanban without dashboard | The same cards with no band, used when purchase orders are embedded in another screen. |
| Calendar | Places each order at its calendar start date — the confirmation date once confirmed, the order deadline before — coloured by vendor, showing the vendor reference and the total, with at most five events per day and no in-place creation. |
| Pivot | Rows by vendor, measuring the total. |
| Graph | Grouped by vendor, measuring the total. |
| Activity | One row per order showing the reference, the total, the vendor and the status as a coloured badge. |

### 5.1 Search filters on the requests-for-quotation screen

Free text matches the reference, the vendor reference, or the vendor and its children.

| Filter | Restriction |
|---|---|
| My Purchases | The buyer is the reader. |
| Starred | The priority is urgent. |
| To Approve | The status is to approve. Hidden by default; used by the dashboard. |
| New | The status is draft. |
| Sent | The status is sent. |
| Purchase Orders | The status is purchase. |
| Late | The status is draft, sent or to approve, and the order deadline is in the past. |
| Not Acknowledged | The status is purchase (or the historic done status) and the order is not acknowledged. |
| Late Receipts | The status is purchase (or done) and the late flag is true. |
| Order Date | A date window on the order deadline. |
| Warnings | Orders carrying an exception activity. |
| Activity filters | My, late, today and upcoming activities. Hidden by default. |

Groupings offered: vendor, buyer, order date.

### 5.2 Search filters on the confirmed-orders screen

The same free-text rule. Filters: My Orders; Starred; **Waiting Bills** (billing status is
*to invoice*, described as orders that include lines not invoiced); **Bills Received** (billing
status is *invoiced*); Order Date; Warnings; the activity filters. Same groupings.

### 5.3 Search on purchase order lines

Free text on the order, the product and the vendor. Filters: *Hide cancelled lines*; *Status*
(the order is confirmed); a date window on the order date; and a hidden filter *Order Date:
Last 365 Days*. Groupings: vendor, product, order reference, order date.

---

## 6. The buyer dashboard

The kanban screen for requests for quotation shows a band of counters, fetched by the named
operation `retrieve_dashboard` (see section 9). Every counter is produced twice, once globally
and once for the reader's own orders, and each carries a total and an urgent-only subtotal.

| Counter | Counts |
|---|---|
| draft | Orders in the draft status. |
| sent | Orders in the sent status. |
| late | Orders in draft, sent or to approve whose order deadline is in the past. |
| not_acknowledged | Orders in the purchase (or historic done) status that are not acknowledged. |
| late_receipt | Orders in the purchase (or done) status whose late flag is true. |
| days_to_order | The average number of days between creation and confirmation over the last three months, to two decimals. |

When inventory is installed two more values are added: `otd`, the on-time delivery percentage
over the orders whose expected arrival falls in the last three months, formatted as a whole
percentage; and `days_to_purchase`, the company's configured days-to-purchase value.

Reading the dashboard requires an internal user and a successful read check on purchase orders.

---

## 7. The matching screen

A list over the Purchase and Bill Line Match Entry, showing per row: the product, the reference
(the order's display name or the bill's), the quantity in the product's reference unit, the
unit price in the product's reference unit, the ordered untaxed amount, the billed untaxed
amount, the billed quantity, the quantity to invoice, and the status. The quantity and the unit
price are editable in place and write back to the underlying order line or bill line.

Two operations are offered on the selection: **match or create bill** and **add to purchase
order**; both are specified in [`workflows.md`](workflows.md). Clicking a row opens the
underlying bill or order.

---

## 8. Vendor bill additions

On a vendor bill, purchasing adds:

| Control | Purpose |
|---|---|
| Auto-complete | A picker over the Purchases and Bills Union: choosing a posted bill copies it, choosing a confirmed order pulls its lines. |
| Purchase Order | A picker that pulls one order's lines directly. |
| Purchase Matching | A statistic button opening the matching screen scoped to this bill. |
| Source Purchase Orders | A statistic button opening the orders behind this bill; the count and the single order's name are exposed as fields. |
| Purchase warning | A banner with the accumulated vendor and product warnings, shown only on vendor bills. |
| Matched indicator | A flag that is false as soon as one product line has no purchase order line. |

On each bill line, purchasing adds the purchase order line link, the purchase order (derived),
the down-payment flag and the product's purchase warning.

---

## 9. Named operations

These are the operations a client may invoke by name on a record set. Inputs and outputs are
described in plain terms; every one of them is also reachable through the generic remote call
contract of the platform.

### 9.1 On a Purchase Order

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_rfq_send` | The context flag `send_rfq` decides which template is used | An instruction to open the email composer | Prepares the composer; marks the order as sent when the message is posted. |
| `button_confirm` | none | a success indicator, or an instruction to open the alternative question | Confirms, as specified in [`workflows.md`](workflows.md). |
| `button_approve` | an optional force flag, currently ignored by the guard | an empty instruction | Approves the orders whose approval test succeeds. |
| `button_draft` | none | an empty instruction | Sets the status to draft. |
| `button_cancel` | none | none | Cancels, subject to the lock and bill guards. |
| `button_lock` / `button_unlock` | none | none | Sets or clears the locked flag. |
| `action_acknowledge` | none | none | Sets the acknowledged flag. |
| `print_quotation` | none | a document-production instruction | Moves draft orders to sent, then produces the quotation. |
| `action_create_invoice` | an optional list of attachment identifiers | an instruction opening the created bill or bills | Creates the vendor bills. |
| `action_view_invoice` | an optional set of bills | an instruction opening them | Opens the bills of the order. |
| `action_merge` | none | an instruction opening the survivor or survivors | Merges the selected requests for quotation. |
| `action_bill_matching` | none | an instruction opening the matching screen | — |
| `action_purchase_comparison` | none | an instruction opening the purchase history | — |
| `retrieve_dashboard` | none | the dashboard structure of section 6 | Read-only. |
| `send_reminder_preview` | none | a toast message, or nothing | Sends the reminder to the acting user only. |
| Send the vendor reminder | an optional single-order flag | an instruction opening the composer, or nothing | The reminder logic; also the body of the scheduled job. |
| `get_acknowledge_url` | none | the portal address with the acknowledgement flag | — |
| `get_confirm_url` | an optional kind | a portal address | Retained for compatibility with older links; the kinds *reminder*, *reception* and *decline* all resolve to the acknowledgement address. |
| `get_update_url` | none | the portal address with the update flag | — |
| `get_portal_url` | standard portal arguments | a portal address | Provided by the portal mixin. |
| `action_view_picking` | none | an instruction opening the transfers | Present when inventory is installed. |
| `action_purchase_order_suggest` | the suggestion parameters in the context | the net change in the number of lines | Fills the order from the replenishment suggestion. Present when inventory is installed. |
| `action_add_from_catalog` | none | an instruction opening the product catalog | — |
| Update a catalog line | a product, a quantity, an optional section | the resulting discounted unit price | The catalog's write path. |
| `action_create_alternative` | none | an instruction opening the creation assistant | Present when the agreements capability is installed. |
| `action_compare_alternative_lines` | none | an instruction opening the comparison list | Same. |
| `get_tender_best_lines` | none | three lists of line identifiers: best total, best arrival date, best unit price | Same. |
| `create_document_from_attachment` | a list of attachment identifiers | an instruction opening the generated orders | Creates orders from vendor documents. |
| `get_import_templates` | none | one entry: the label *Import Template for Requests for Quotation* and the address of the shipped spreadsheet | — |
| `action_open_business_doc` | none | an instruction opening the order's form | — |

### 9.2 On a Purchase Order Line

| Operation | Inputs | Output | Effect |
|---|---|---|---|
| `action_open_order` | none | an instruction opening the order | — |
| `action_add_from_catalog` | the order identifier in the context | an instruction opening the catalog | — |
| Read a catalog line | none | a structure with the quantity, the price, a read-only flag, the unit name, and optionally a minimum quantity, a unit factor and a warning | The catalog's read path. Raises when the selection spans more than one product. |
| `action_product_forecast_report` | none | an instruction opening the forecast report positioned on this line | Present when inventory is installed. |
| `action_clear_quantities` | none | nothing, or a notification | Sets the quantity of every selected line to zero unless its order is cancelled or confirmed. Present when the agreements capability is installed. |
| `action_choose` | none | nothing, or a notification | Clears the competing lines of the same products across the alternative group. Same. |

### 9.3 On a Purchase Agreement

| Operation | Effect |
|---|---|
| `action_confirm` | Confirms, publishing the vendor pricelist entries for a blanket order. |
| `action_done` | Closes, deleting the published entries. |
| `action_cancel` | Cancels, deleting the published entries and cancelling draft requests. |
| `action_draft` | Returns to draft. |

### 9.4 On a Journal Entry

| Operation | Effect |
|---|---|
| `action_purchase_matching` | Opens the matching screen scoped to this bill. |
| `action_view_source_purchase_orders` | Opens the source orders. |
| Find and link source orders | Given candidate references, a vendor, a total, a scan flag and a time budget, matches and links the bill to open orders. This is the entry point electronic and scanned document importers call. |
| Append order lines to the bill | Appends the given purchase order lines to the bill as new lines. |
| Replace or append the lines of given orders | Replaces or appends the lines of the given orders, inserting a section per order. |

### 9.5 On a Purchase and Bill Line Match Entry

| Operation | Effect |
|---|---|
| `action_match_lines` | Matches the selection, or creates a bill from the selected order lines. |
| `action_add_to_po` | Opens the bill-to-order assistant. |
| `action_open_line` | Opens the underlying bill or order. |

### 9.6 On the assistants

| Assistant | Operations |
|---|---|
| Bill To Purchase Order | `action_add_to_po`, `action_add_downpayment` |
| Alternative Order Creation | `action_create_alternative` |
| Alternative Order Warning | `action_keep_alternatives`, `action_cancel_alternatives` |
| Accrued Orders | `create_entries` |

---

## 10. Routes

All portal routes render website-aware pages. *Authentication: user* means a signed-in user is
required; *public* means the page may also be reached with a valid access token.

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/rfq` and `/my/rfq/page/<page number>` | Web request | user | The list of the reader's requests for quotation, that is, orders in the sent status. Accepts a creation-date window, a sort key and a page number. |
| `/my/purchase` and `/my/purchase/page/<page number>` | Web request | user | The list of the reader's purchase orders, that is, orders in the purchase or cancelled statuses. Accepts the same arguments plus a filter key. |
| `/my/purchase/<order identifier>` | Web request | public | One order. Accepts an access token, a report type (`html`, `pdf` or `text`) which serves the quotation document for a request and the purchase order document otherwise, a download flag, an `acknowledge` flag which sets the acknowledged flag, and an `update` flag which renders the date-editing variant of the page. Redirects to the portal home when access fails. |
| `/my/purchase/<order identifier>/update` | Remote call returning nothing | public | Accepts a map of line identifiers to dates formatted as four-digit year, two-digit month, two-digit day separated by hyphens. Applies them as described in [`workflows.md`](workflows.md) and answers with an empty success response. Redirects when access fails or when a supplied identifier is not a number or not a line of this order; silently skips a date that does not parse. |
| `/my/purchase/<order identifier>/download_edi` | Web request | public | Serves the structured machine-readable representation of the order as an attachment, with the content type `text/xml`, the correct content length and a content-disposition naming the file. Redirects to the portal home when no builder is configured or when access fails. |

### 10.1 Portal sort and filter keys

| Sort key | Label | Ordering |
|---|---|---|
| `date` | Newest | Creation date descending, then identifier descending. This is the default. |
| `name` | Name | Reference ascending, then identifier ascending. |
| `amount_total` | Total | Total descending, then identifier descending. |

| Filter key | Label | Restriction | Screen |
|---|---|---|---|
| `all` | All | status is purchase or cancelled | Purchase orders. This is the default. |
| `purchase` | Purchase Order | status is purchase | Purchase orders. |
| `cancel` | Cancelled | status is cancelled | Purchase orders. |

The requests-for-quotation screen offers no filter keys.

### 10.2 Portal home counters

| Counter key | Counts |
|---|---|
| `rfq_count` | Orders in the sent status the reader may read; zero when the reader may not read orders. |
| `purchase_count` | Orders in the purchase or cancelled statuses the reader may read; zero likewise. |

### 10.3 What the portal order page shows

The order, its lines and its totals, plus a helper that resizes an image to 48 by 48 points for
the vendor's logo. The page remembers which list the reader came from so that previous and next
navigation works: the requests list for an order in the sent status, the orders list otherwise.

---

## 11. Printable documents

### 11.1 The purchase order document

| Section | Content |
|---|---|
| Heading | *Purchase Order* followed by the reference when the status is purchase or cancelled; *Request for Quotation* followed by the reference otherwise. |
| Addresses | The company's own address and the vendor's address; the shipping address when a dropship address is set. |
| Information block | The vendor reference, the buyer, the order deadline (labelled *Order Deadline* for a request and *Confirmation Date* for an order), and the expected arrival. |
| Line table | One row per line: the description, the expected arrival, the quantity with its unit, the unit price, the discount when any line has one, the taxes, and the amount. Sections and subsections are rendered as headings; notes as free text. |
| Totals | The untaxed amount, the tax amounts per group, and the total, all in the order currency. |
| Variant grids | When the grid capability is installed and the order asks for them, one grid per configurable product that has more than one line. |
| Footer | The terms and conditions, the incoterm and the company's document footer. |

Produced name: *Purchase Order - the order reference* once confirmed, *Request for Quotation -
the order reference* before. Download file name: *Purchase Order-the order reference*.

### 11.2 The quotation document

The same structure with prices presented as a request rather than a commitment, always named
*Request for Quotation - the order reference*.

### 11.3 The agreement document

| Section | Content |
|---|---|
| Heading | The agreement type label followed by the agreement name. |
| Information | The vendor, the purchase representative, the reference, the source document, and — for a blanket order only — the agreement validity, showing the end date. |
| Line table | One row per agreement line: the product, the description, the quantity with its unit, and the agreed unit price. |
| Footer | The agreement description. |

### 11.4 Embedded structured documents

When the structured electronic order capability is installed and exactly one record is printed,
the produced file gains one embedded attachment per configured builder, each of content type
`text/xml` and named by the builder. The base platform configures no builder, so nothing is
embedded until the capability is installed.

---

## 12. Message templates and notifications

### 12.1 Templates

| Template | Subject | Body summary | Attachment | Extra |
|---|---|---|---|---|
| Purchase: Request For Quotation | *the company name Order (Ref the order reference or n/a)* | Addresses the vendor by name, and its parent company in parentheses when it has one; states that a request for quotation is attached, with the vendor reference when set, from the company; invites questions; signs with the buyer's signature when set. | The quotation document | Described as *Sent manually to vendor to request a quotation*. |
| Purchase: Purchase Order | The same subject | Same opening; states that a purchase order is attached, with the vendor reference when set, amounting to the formatted total, from the company. When the order has an expected arrival, adds that the receipt is expected for that date, asks for acknowledgement, and renders an **Acknowledge** button pointing at the acknowledgement address. | The purchase order document | Described as *Sent to vendor with the purchase order in attachment*. |
| Purchase: Vendor Reminder | The same subject | Same opening; reminds that the delivery of the order — with the vendor reference in parentheses when set — is expected for the formatted expected arrival, or the word *undefined* when there is none; asks for confirmation; renders the **Acknowledge** button. | The purchase order document | Sender address is the buyer's formatted address, falling back to the acting user's. Described as *Sent to vendors before expected arrival, based on the purchase order setting*. |

All three resolve their recipient through the template's default-recipient rule rather than a
fixed partner, and all three delete the outgoing message record once sent.

### 12.2 The notification wrapper

When a message is sent from an order, the notification layout's *view* button is adjusted:

- for a portal recipient, the label becomes *View Quotation* while the order is draft or sent,
  and *View Order* afterwards, and the target becomes the order's portal address;
- when the message is a reminder, the label is simply *View*.

The email subtitles are the order reference, followed by *Order due* and the formatted order
deadline for a draft or sent order, or the formatted total for a confirmed one.

### 12.3 Thread subtypes

| Subtype | Posted when | Subscribed by default |
|---|---|---|
| request for quotation Sent | The status becomes sent | No |
| request for quotation Confirmed | The status becomes to approve, or becomes purchase from any status other than to approve | No |
| request for quotation Approved | The status becomes purchase from to approve | No |

### 12.4 Automatic notes

| Note | Posted on | When |
|---|---|---|
| Extra line with *the product name* | The order | A product line is created on an order already in the purchase status. |
| Ordered-quantity change note | The order | A line's ordered quantity changes on an order in the purchase status. |
| Received-quantity change note | The order | A line's received quantity changes on an order in the purchase status. |
| *RFQ merged with the survivor name and the absorbed names* | The survivor | A merge completes. |
| *RFQ merged with the survivor link* | Each absorbed order | The same. |
| *Cancelled by the agreement associated to this quotation.* | Each cancelled request | An agreement is cancelled. |
| *This vendor bill has been created from: the order links* | The bill | A bill is created carrying purchase order links. |
| *This vendor bill has been modified from: the order links* | The bill | A write adds new source orders. |
| Origin link to the agreement | The order | An order is created against an agreement, or its agreement is written. |
| Origin link to the order | The receipt | A receipt is created. |
| *The purchase order the order link this receipt is linked to was cancelled.* | A done transfer | Its order is cancelled. |
| *Accrual entry created on the date: the entry link. And its reverse entry: the reverse entry link.* | The order | An accrual entry is produced. |

### 12.5 Automatic activities

| Activity | Raised on | Summary or note |
|---|---|---|
| Date Updated | The order, for its buyer | Created or extended when a vendor updates expected arrivals through the portal. Its note lists one bullet per line, *the product name from the old date to the new date*, and ends with a sentence about the receipt when inventory is installed. |
| Warning | The first bill of a line | *The quantities on your purchase order indicate less than billed. You should ask for a refund.* |
| Warning | The impacted transfers | A rendered exception listing the orders, the quantity changes and the next impacted transfers, when a quantity is decreased on a confirmed order. |
| Warning | The originating sales orders | A rendered exception when a purchase order carrying re-purchased service lines is cancelled. |
| Warning | The purchase orders | A rendered exception when a sold quantity of a re-purchased service is decreased. |
| To-do | The order | *Some information could not be imported:* followed by the details, when a received structured order document could not be fully understood. |

---

## 13. Integrations with outside parties

| Integration | Direction | Contract |
|---|---|---|
| Vendor email | Outbound | The three message templates of section 12. |
| Vendor portal | Inbound | The routes of section 10: viewing, acknowledging, updating expected arrivals, downloading the structured order. |
| Structured order document, export | Outbound | One machine-readable document per configured builder, embedded in the printed file and downloadable from the portal. The shipped builder produces an order document conforming to the pan-European public procurement online network's ordering profile identified by the customisation identifier `urn:fdc:peppol.eu:poacc:trns:order:3`. |
| Structured order document, import | Inbound | A received file is recognised as such an order document when its customisation identifier is exactly that value; a decoder of priority 20 then builds a purchase order from it. Information that cannot be mapped raises the to-do activity of section 12.5. |
| Vendor bill documents | Inbound | Attachments dropped on an order create a bill and are attached to it. Attachments processed by the platform's document import produce bills that are then matched to orders by the algorithm of [`workflows.md`](workflows.md). |
| Vendor order documents | Inbound | Attachments dropped on the requests-for-quotation screen create one order per attachment, with the acting user's partner pre-set. |

The structured document's own field-by-field mapping belongs to
[`../electronic-invoicing-and-document-exchange/README.md`](../electronic-invoicing-and-document-exchange/README.md);
the purchasing-specific parts are that the document is built from the order's non-display
lines, that line unit prices are forced to be non-negative before export, and that the totals
are rounded globally to six digits before the document's own rounding is applied.

---

## 14. Import and export

### 14.1 Import templates

| Template | Offered on | File |
|---|---|---|
| Import Template for Requests for Quotation | Purchase Order | A shipped spreadsheet at `/purchase/static/xls/requests_for_quotation_import_template.xlsx` |
| Import Template for Products | Product Template, when the screen is the purchasing product screen | A shipped spreadsheet at `/purchase/static/xls/product_purchase.xls` |

### 14.2 What an import must supply

A minimal request-for-quotation import supplies, per row: the order reference (repeated on
every line of the same order), the vendor, and per line the product, the quantity, the unit and
the unit price. Because the line creation routine fills in missing values from the product — the
description, the price, the quantity, the unit, the taxes and the expected arrival are all
deduced when absent — an import may omit any of them.

### 14.3 Export

Every stored field of every entity is exportable through the platform's generic export, with
two exceptions on the Purchase Order: the tax totals structure is explicitly marked
non-exportable, and computed non-stored fields (the duplicates, the comparison flag, the
warning text, the late flag, the shipped flag, the transfer count) cannot be exported because
they are not stored.

The purchase analysis entity and the vendor delay entity are read-only views and export like
any other list.

---

## 15. Client-side components

| Component | Where | Purpose |
|---|---|---|
| Purchase file uploader | The order form header, when the status is purchase | Accepts dropped vendor documents and turns them into a bill through `action_create_invoice`. |
| Toaster button | The reminder block on the order form | Calls `send_reminder_preview` and shows the returned toast. |
| Product catalog kanban | The catalog screens | A purchasing-specific product card showing the vendor price, the minimum quantity and, when inventory is installed, the stock situation and the suggested quantity. |
| Comparison list | The alternative comparison screen | A list that highlights the best total, the best unit price and the best arrival date per product, using the three identifier lists returned by `get_tender_best_lines`. |
| Dashboard band | The requests-for-quotation kanban | Renders the structure returned by `retrieve_dashboard`. |
| Variant grid | The order form, when the grid capability is installed | Renders and saves a matrix of variant quantities. |
| Portal date editor | The portal order page in update mode | Collects the new expected arrivals and posts them to the update route. |
