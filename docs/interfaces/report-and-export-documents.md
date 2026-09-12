# Reports, printed documents, exports and imports

Every printed document the system produces, every file it exports, every file it imports, and the data files the business
domains generate for exchange with outside parties. The first part specifies the rendering model that all printed
documents share. The second part catalogues the ninety-four printed documents by domain, with the sections, fields,
totals, grouping, language and paper format of each. The third part specifies the generic export mechanisms, the fourth
the generic import mechanism, and the fifth the domain-specific data files.

## Part 1: the document rendering model

This part states the parts of the rendering model a reader of the document catalogue needs in order to read Part 2: what
a report definition holds, how a print request becomes a file, which language the file is rendered in, which layout
surrounds it and which paper geometry it is laid out on. The rendering model itself — the template grammar, the field
converters, the splitting of one rendered document into one file per record, the barcode generator, the label layouts
and the failure codes of the conversion — is specified in
[`../runtime/report-rendering.md`](../runtime/report-rendering.md), and that document is the authority wherever the two
overlap. The pointers below name the section of it that carries each topic in full.

### The report definition

A printed document is defined by a **report definition** record. Its transport name is `ir.actions.report` and its full
name is Report Action; the complete entity, its resolution rules, the way a print entry is produced and the letterhead
detour taken on the first print of a fresh installation are specified in
[`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 3. The fields are reproduced here because
every entry of the catalogue in Part 2 is read against them.

| Identifier | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `name` | Name | Text, translatable | yes | none | The label shown in the print menu, and the file name when no name expression exists. |
| `model` | Model Name | Text | yes | none | The transport name of the entity whose records the document is printed for. |
| `model_id` | Model | Many-to-one to Entity, computed with a search rule | no | derived | The same entity as a link, so that the configuration screen can offer a selector. |
| `report_type` | Report Type | Selection: `qweb-html`, `qweb-pdf`, `qweb-text` | yes | `qweb-pdf` | The form of the produced file. The three stored values are reproduced exactly as they are stored; their labels are `HTML`, `PDF` and `Text`. A formatted document is always rendered first: `qweb-pdf` then produces the paginated form through the document converter and downloads it, `qweb-html` returns the rendered form as a page shown in place, and `qweb-text` returns unformatted text without the surrounding layout. |
| `report_name` | Template Name | Text | yes | none | The external identifier of the document template that renders one record. |
| `report_file` | Report File | Text, stored, writable | no | empty | The path of the file the template is declared in. |
| `print_report_name` | Printed Report Name | Text, translatable | no | empty | An expression evaluated per record that produces the file name shown to the user. When empty, the report name and the record display name are used. |
| `attachment` | Save as Attachment Prefix | Text | no | empty | An expression evaluated per record that produces the name under which the rendered document is stored as an attachment of the record. Empty means the result is not stored. |
| `attachment_use` | Reload from Attachment | Boolean | no | false | When true and an attachment with that name already exists on the record, the stored file is returned instead of rendering again. This is what makes a posted invoice always print byte-identical. |
| `binding_model_id` | Action Binding | Many-to-one to Entity | no | empty | The entity on whose records the document appears in the print menu. When empty, the document is only reachable from a button or from an action. |
| `binding_type` | Action Binding Type | Selection: `action`, `report` | no | `report` | Which menu of the interface the entry appears in: the action menu or the print menu. A report definition defaults to `report`. |
| `binding_view_types` | Binding View Types | Text listing view kinds | no | `list,form` | The view kinds whose print menu offers the entry. |
| `multi` | On Multiple Doc. | Boolean | no | false | When true, the entry is not offered on a single-record screen. |
| `paperformat_id` | Paper Format | Many-to-one to Paper Format, indexed when not empty | no | empty | The paper format to apply. When empty the paper format of the company applies, and when that is empty the installation default applies. |
| `group_ids` | Groups | Many-to-many to Access Group, through the association table `res_groups_report_rel` | no | empty | When set, only members of one of these groups see the document in the print menu and may render it. |
| `domain` | Filter domain | Text holding a condition | no | empty | Restricts which records of the entity the document may be printed for. |

Every identifier and every stored selection value in the table is reproduced exactly as the system stores it, because a
replacement that has to import an existing database writes these column names and these values.

### Rendering pipeline

The steps below are the print request as a person experiences it. The internal stages — the rendering context, the
splitting of one rendered document into one file per record, the geometry arguments handed to the converter, the merge
of several files and the failure codes of the conversion — are specified in
[`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 4.

1. The caller supplies the report and the record identifiers, either through the print menu of a list or form, through a
   button that returns a print action, or through the rendering endpoints described in
   [`endpoint-catalog.md`](endpoint-catalog.md).
2. The records are read with the rights of the caller. A record the caller may not read makes the whole request fail
   with the access refusal; no partial document is produced.
3. The rendering language is chosen (see the next section) and the records are re-read in that language.
4. When the report stores an attachment and reuse is enabled, each record whose attachment already exists contributes
   its stored file and is skipped by the renderer.
5. The template is rendered once per record into a formatted document. Templates that declare a container render all
   records into one document; templates that do not are concatenated.
6. The formatted document is converted to the requested output form. For the portable form, the paper format supplies
   the page size, the orientation, the four margins, the header spacing, the header rule, the resolution and the
   header and footer templates. Page numbers are rendered as "Page &lt;current&gt; / &lt;total&gt;" in the footer of the
   portable form only.
7. When the report defines an attachment name, the produced file is stored as an attachment of each record, replacing an
   attachment of the same name.
8. The file is returned with its media type and a disposition that carries the file name. The file name is sanitized:
   path separators are removed.

**Failure paths.** A missing document converter makes the availability check report false and the print action falls
back to the formatted form. A template that raises during rendering fails the whole request and the error page names the
template. A record set that the filter condition excludes produces no document and the user is told that nothing can be
printed.

### Language of a printed document

The per-recipient language rule of the renderer is specified in
[`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 4.14; the decision as it applies to the
documents catalogued in Part 2 is this. The rendering language is decided per record, in this order:

1. The language explicitly passed in the print context, when one is passed.
2. The language of the partner the document is addressed to, when the document has one (customer of an invoice, vendor
   of a purchase order, attendee of a ticket, employee of a payslip).
3. The language of the user who prints.

Step two is the observed behavior of every customer-facing document, whose template re-reads the record in the language
of the addressed partner before rendering. Steps one and three are an industry-standard completion for documents that
name no partner, in order that a rule exists for every document rather than only for the addressed ones.

Every translatable value is rendered in that language: field labels, selection values, product names and descriptions,
payment term notes, fiscal position notes, terms and conditions, and the layout texts. Amounts, dates and numbers are
formatted with the conventions of that language, while the currency and the decimal precision come from the document.

### Shared layouts

The layout templates, the company branding fields they read, the address block and the page-numbering rule are
specified in [`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 6. Four page layouts are
shipped and one is selected per company. They differ only in decoration; each renders the same
blocks.

| Block | Content |
|---|---|
| Header, left | The company logo, when one is set. |
| Header, right | The company tagline; the company address block, or, when the company defines a details block, that block instead; the tax identification label of the company country followed by the tax identification number of the company. When the document forces another tax identification (a foreign registration through a fiscal position), that number is shown instead. |
| Address zone | The recipient address, and, when the document has a different delivery address, both addresses side by side under their own titles. |
| Document title | The title the document computes, followed by its reference. |
| Body | The report template. |
| Footer | The company footer text, the document name when the layout asks for it, and the page number over the page count. |

The layout used by a document is chosen by the report: external layout for documents sent to third parties, internal
layout for documents used inside the company, basic layout for labels and badges, and no layout for machine-readable
label output.

### Paper formats

The Paper Format entity, its field table, its refusal message and the named page sizes it offers are specified in
[`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 5, whose own table lists five of the
formats. The table below lists every format of the whole installation, those contributed by the business capability
packages included. The installation ships eighteen named paper formats besides the one a company may define for itself. Page shrinking is
disabled on the label and ticket formats, in order that a label prints at its exact size.

| Paper format | Page | Orientation | Margins top / bottom / left / right (millimetres) | Header spacing | Header rule | Resolution (dots per inch) |
|---|---|---|---|---|---|---|
| A4 (default) | A4 | portrait | 52 / 32 / 0 / 0 | 52 | no | 90 |
| US Letter | Letter | portrait | 52 / 32 / 0 / 0 | 52 | no | 90 |
| A4 for bank and cash statements | A4 | portrait | 52 / 32 / 0 / 0 | 52 | no | 90 |
| US batch deposit | Letter | portrait | 15 / 30 / 10 / 10 | 15 | no | 90 |
| Event badge | A4 | portrait | 0 / 0 / 0 / 0 | none | none | 96 |
| Event full page ticket | A4 | portrait | 0 / 8 / 0 / 0 | none | none | 96 |
| Employee badge | A4 | portrait | 5 / 0 / 0 / 0 | none | none | 96 |
| Time off summary | custom, 210 by 297 millimetres | landscape | 30 / 23 / 5 / 5 | 20 | no | 90 |
| Employee resume | A4 | portrait | 12 / 12 / 5 / 5 | 20 | no | 90 |
| European A4 without borders | A4 | portrait | 0 / 0 / 0 / 0 | 0 | no | default |
| European A4, German business letter type A | A4 | portrait | 27 / 40 / 20 / 10 | 27 | no | 70 |
| European A4, German business letter type B | A4 | portrait | 45 / 40 / 20 / 10 | 45 | no | 70 |
| A4 for the Indian electronic waybill | A4 | portrait | 5 / 5 / 7 / 7 | 15 | no | 90 |
| Saudi Arabian A4 | default page | portrait | 65 / 32 / 0 / 0 | 65 | none | default |
| Two-dimensional code page for self ordering | default page | default | defaults | none | no | default |
| A4 label sheet | A4 | portrait | 0 / 0 / 0 / 0 | none | none | 96 |
| Small label roll | custom, 32 by 57 millimetres | landscape | 0 / 0 / 0 / 0 | none | none | 96 |
| Certification | A4 | landscape | 0 / 0 / 0 / 0 | 0 | no | 96 |

One format is amended rather than added when a further capability package is installed: the bottom margin of the event
full page ticket format becomes 29 millimetres instead of 8 when the exhibitor capability is installed, in order to
leave room for the sponsor images printed at the foot of the ticket.

A company may define its own paper format; it then applies to every document that does not name one. When neither the
report nor the company names one, the default A4 format is used.

### Reading the catalog

Each entry of the catalog below states: the document name, the entity it prints, the output form, the paper format when
it differs from the company default, the print menu it appears in, the file name rule, whether the rendered file is
stored on the record, and the content: the blocks in order, the fields printed in each, the line table columns, the
totals and the grouping.

## Part 2: the printed document catalog

The installation ships ninety-four printed document definitions. The table lists all of them; the sections that follow
describe the content of each, grouped by domain.

| Domain | Document | Entity | Output | Paper format | In a print menu |
|---|---|---|---|---|---|
| Country packages | Commercial Invoice | Journal Entry | portable document | company default | yes |
| Country packages | Commercial Invoice | Journal Entry | portable document | company default | yes |
| Country packages | Compliance Letter | Company | portable document | company default | yes |
| Country packages | Delivery Guide (Argentina) | Transfer | portable document | company default | yes |
| Country packages | Disbursement Voucher (Philippines) | Payment | portable document | company default | yes |
| Country packages | Duplicate (2 Copies) | Journal Entry | portable document | company default | yes |
| Country packages | Electronic Waybill | Electronic Waybill | portable document | A4 for the waybill | yes |
| Country packages | Hash integrity result | Company | portable document | company default | no |
| Country packages | Payment slip with two-dimensional code | Journal Entry | portable document | European A4 without borders | no |
| Country packages | Transport document (Italy) | Transfer | portable document | company default | no |
| Country packages | Triplicate (3 Copies) | Journal Entry | portable document | company default | yes |
| Country packages | Voucher | Journal Entry | portable document | company default | yes |
| Electronic invoicing | Invoice rendered from structured data | Journal Entry | portable document | company default | no |
| Events | Attendee List | Event | portable document | company default | yes |
| Events | Attendee List | Event Registration | portable document | company default | yes |
| Events | Badge | Event Registration | portable document | Event badge | yes |
| Events | Badge Example | Event | portable document | Event badge | yes |
| Events | Full Page Ticket | Event Registration | portable document | Event full page ticket | yes |
| Events | Full Page Ticket Example | Event | portable document | Event full page ticket | yes |
| Events | Responsive Full Page Ticket | Event Registration | portable document | company default | no |
| Expenses | Expense receipt images | Expense | portable document | company default | no |
| Expenses | Expenses Report | Expense | portable document | company default | yes |
| General ledger and invoicing | Accounting Tests | Accounting Consistency Test | portable document | company default | yes |
| General ledger and invoicing | Hash integrity result | Company | portable document | company default | no |
| General ledger and invoicing | Invoice | Journal Entry | portable document | company default | yes |
| General ledger and invoicing | Invoice without Payment | Journal Entry | portable document | company default | yes |
| General ledger and invoicing | Original Bills | Journal Entry | portable document | company default | yes |
| General ledger and invoicing | Payment Receipt | Payment | portable document | company default | yes |
| General ledger and invoicing | Statement | Bank Statement | portable document | A4 for statements | no |
| Human resources | Employee Resume | Employee | portable document | Employee resume | no |
| Human resources | Print Badge | Employee | portable document | Employee badge | yes |
| Inventory | Batch Transfer | Batch Transfer | portable document | company default | yes |
| Inventory | Count Sheet | Stock Quantity Record | portable document | company default | yes |
| Inventory | Delivery Slip | Transfer | portable document | company default | yes |
| Inventory | Location Barcode | Location | portable document | A4 label sheet | yes |
| Inventory | Lot and Serial Number Label | Lot or Serial Number | portable document | A4 label sheet | yes |
| Inventory | Lot and Serial Number Label (label printer command language) | Lot or Serial Number | label printer command text | company default | yes |
| Inventory | Operation Type Label | Operation Type | portable document | A4 label sheet | yes |
| Inventory | Operation Type Label (label printer command language) | Operation Type | label printer command text | company default | yes |
| Inventory | Package Barcode (label printer command language) | Package | label printer command text | company default | yes |
| Inventory | Package Barcode (label printer command language) | Package History | label printer command text | company default | yes |
| Inventory | Package Barcode (small) | Package | portable document | company default | yes |
| Inventory | Package Barcode (small) | Package History | portable document | company default | yes |
| Inventory | Package Barcode with Contents | Package | portable document | company default | yes |
| Inventory | Package Barcode with Contents | Package History | portable document | company default | yes |
| Inventory | Packages | Transfer | portable document | company default | yes |
| Inventory | Packaging Barcodes (label printer command language) | Product Packaging Unit | label printer command text | company default | yes |
| Inventory | Picking Operations | Transfer | portable document | company default | yes |
| Inventory | Product Label (label printer command language) | Product Variant | label printer command text | company default | no |
| Inventory | Product Routes Report | Product Template | rich text | company default | no |
| Inventory | Reception Report | Transfer | portable document | company default | no |
| Inventory | Reception Report Label | Stock Move | portable document | Small label roll | no |
| Inventory | Return slip | Transfer | portable document | company default | yes |
| Loyalty | Coupon Code | Loyalty Card | portable document | company default | yes |
| Loyalty | Gift Card | Loyalty Card | portable document | company default | yes |
| Manufacturing | Bill of Materials Overview | Bill of Materials | portable document | company default | yes |
| Manufacturing | Finished Product Label | Manufacturing Order | portable document | A4 label sheet | yes |
| Manufacturing | Finished Product Label (label printer command language) | Manufacturing Order | label printer command text | company default | yes |
| Manufacturing | Manufacturing Order Overview | Manufacturing Order | portable document | company default | no |
| Manufacturing | Production Order | Manufacturing Order | portable document | company default | yes |
| Manufacturing | Work Order | Work Order | portable document | company default | yes |
| Manufacturing | Work in progress report | Analytic Line | portable document | company default | yes |
| Messaging | Live Chat Conversation | Discussion Channel | portable document | company default | no |
| Platform | Entity overview | Entity Definition | portable document | company default | yes |
| Platform | Preview External Report | Company | portable document | company default | yes |
| Platform | Preview Internal Report | Company | portable document | company default | yes |
| Platform | Report Layout Preview | Company | portable document | company default | yes |
| Platform | Technical guide | Capability Package | portable document | company default | yes |
| Point of sale | Sales Details | Point of Sale Session | portable document | company default | yes |
| Point of sale | Two-dimensional codes page | Point of Sale Configuration | portable document | company default | no |
| Point of sale | User Labels | User | portable document | company default | yes |
| Products | Packaging Barcodes | Product Packaging Unit | portable document | company default | yes |
| Products | Pricelist | Product Variant | portable document | company default | no |
| Products | Product Label (small label roll) | Product Template | portable document | Small label roll | no |
| Products | Product Label, four by seven | Product Template | portable document | A4 label sheet | no |
| Products | Product Label, four by twelve | Product Template | portable document | A4 label sheet | no |
| Products | Product Label, four by twelve without price | Product Template | portable document | A4 label sheet | no |
| Products | Product Label, two by seven | Product Template | portable document | A4 label sheet | no |
| Purchasing | Purchase Agreements | Purchase Agreement | portable document | company default | yes |
| Purchasing | Purchase Order | Purchase Order | portable document | company default | yes |
| Purchasing | Request for Quotation | Purchase Order | portable document | company default | yes |
| Repair | Repair Order | Repair Order | portable document | company default | yes |
| Sales | Proforma Invoice | Sales Order | portable document | company default | yes |
| Sales | Quotation / Order | Sales Order | portable document | company default | yes |
| Sales | Quotation / Order | Sales Order | portable document | company default | yes |
| Sales | Quotation with merged offer documents | Sales Order | portable document | company default | no |
| Surveys | Certifications | Survey Participation | portable document | Certification | yes |
| Time off | Time Off Summary | Time Off Allocation | portable document | Time off summary | no |
| Timesheets | Timesheets | Analytic Line | portable document | company default | yes |
| Timesheets | Timesheets | Task | portable document | company default | yes |
| Timesheets | Timesheets | Project | portable document | company default | yes |
| Timesheets | Timesheets | Analytic Line | portable document | company default | no |
| Timesheets | Timesheets | Sales Order | portable document | company default | yes |
| Timesheets | Timesheets | Journal Entry | portable document | company default | yes |


### General ledger and invoicing documents

#### Invoice

Entity: Journal Entry. Output: portable document. Layout: external. Stored on the record under the invoice name.
File name: the base file name of the document, which is its reference with path separators removed. Print menu: journal
entries. Visible to the invoicing and read-only accounting groups. Two variants exist: **Invoice** prints the payment
lines, **Invoice without Payment** omits them; a third, **Original Bills**, returns the vendor document that was
received rather than a rendered one, reusing the stored attachment.

Sections in order:

1. **Address zone.** When the delivery address differs from the invoicing address, the delivery address is printed on
   the left under the title "Shipping Address" (visible only to users of the delivery address group) and the invoicing
   address on the right; otherwise only the invoicing address is printed, right aligned. Under the invoicing address:
   the tax identification label of the company fiscal country (or the words "Tax Identification" when the country defines none)
   followed by the tax identification number of the customer, and, for Moroccan customers, the company registry number
   under the label "ICE".
2. **Title.** One of: Invoice, Draft Invoice, Cancelled Invoice, Credit Note, Draft Credit Note, Cancelled Credit Note,
   Vendor Bill, Self Billing, Vendor Credit Note, Self Billing Credit Note; each prefixed with "Proforma" when the
   document is printed as a proforma. The reference follows, omitted while it is still the placeholder.
3. **Information row.** Printed only when at least one of its values exists: Invoice Date (labelled Credit Note Date or
   Receipt Date according to the document kind, and Date otherwise), Due Date (only for a posted customer invoice),
   Taxable Supply date (when the fiscal rules of the company require it), Delivery Date, Source document, Customer Code,
   Reference, and the delivery term code with its named place.
4. **Line table.** Columns: Description; Quantity (with the unit of measure, and, when the line unit differs from the
   reference unit of the product, the converted quantity in the reference unit underneath); Unit Price; Discount
   percentage (column shown only when at least one line carries a discount); Taxes (the labels of the taxes of the
   line, column shown only when at least one line carries a tax); Amount (the amount excluding tax when the company
   prices exclude tax, the amount including tax otherwise). Section headings and sub-section headings are printed in
   bold across the width with their own subtotal in the amount column; notes are printed in italics across the full
   width. Lines whose composition is collapsed print one aggregate row; lines whose prices are hidden print the
   quantity but no amounts. Sections indent their lines by three units and sub-sections by four.
5. **Totals.** The untaxed amount per subtotal group, then one row per tax group with its base and its amount, then the
   total. When payments are printed, one row per matched payment with "Paid on &lt;date&gt;" or "Reversed on &lt;date&gt;"
   and its amount, followed by the bold row "Amount Due" with the residual amount. When the company asks for it, the
   total in words follows. When the invoice currency differs from the company currency and the company asks for it, the
   tax summary is repeated in the company currency.
6. **Payment block.** The fiscal position note, the legal tax notes, the payment term note, the early payment discount
   line ("&lt;amount&gt; due if paid before &lt;date&gt;"), and, for a term with several instalments, one line per
   instalment reading "&lt;number&gt; - Installment of &lt;amount&gt; due on &lt;date&gt;".
7. **Payment communication.** For a customer invoice or a vendor credit note that carries a payment reference: the
   reference in bold, and the bank account to pay into when one is set.
8. **Payment code.** When the document is configured to show it and an amount remains due: the two-dimensional payment
   code image with the caption "Scan this Quick Response Code with your banking application". When the online payment code is
   enabled: a second code linking to the portal payment page with the caption "PAY IN A FLASH! Scan the Quick Response
   Code or click to pay online".
9. **Terms and conditions.** The document note, justified.

#### Payment Receipt

Entity: Payment. Output: portable document. Layout: external. Print menu: payments. Prints the payment reference, its
date, its journal, the payer or payee, the payment method, the amount in the payment currency, and the list of invoices
the payment is matched against with their reference, their date, their total and the amount allocated.

#### Bank and cash statement

Entity: Bank Statement. Output: portable document. Paper format: A4 for statements. Layout: internal.

Blocks: a title reading "Bank Statement" or "Cash Statement" according to the journal type; the journal display name;
the bank account display name when the journal has one; the journal code; the statement name; the statement date.
Then the balance block: "Starting Balance" with the date of the earliest line and the opening balance, and "Ending
Balance" with the date of the latest line and the closing balance. Then one row per statement line: date, partner name,
the owner of the counterpart bank account in parentheses when it differs from the partner, the account number, the
payment reference, and the amount.

#### Hash integrity result

Entity: Company. Output: portable document. Print menu: none; produced by the integrity check action. Prints, per
journal, the first and last secured entry, their dates, the result of the chain verification, and the reason when the
chain is broken.

#### Accounting consistency tests

Entity: Accounting Consistency Test. Output: portable document. Prints the executed consistency tests with, for each,
its name, its description, its expected result and the records that failed it.

### Sales documents

#### Quotation and order

Entity: Sales Order. Output: portable document. Layout: external. Print menu: sales orders. File name: "Quotation -
&lt;reference&gt;" while the order is a draft or was sent, "Order - &lt;reference&gt;" afterwards. A second definition,
**Quotation without the assembled offer**, renders the same template without the merged offer documents, and a third,
**Proforma Invoice**, renders the same body under the proforma title.

Blocks: the customer address, with the tax identification line; the invoicing address and the delivery address, printed
together under "Invoicing and Shipping Address" when they are the same contact and separately otherwise; the title
("Quotation #", "Order #" or "Pro-Forma Invoice #") with the reference; the information row with Your Reference, the
quotation or order date, the expiration date (only while the document is a draft or was sent), the delivery date, and
the salesperson.

Line table columns: Description, Quantity with its unit of measure (and the converted quantity in the reference unit
when they differ), Unit Price (struck through and replaced by the discounted price when the discount is negative),
Discount percentage (column shown only when a line carries one), Taxes, Amount. Section and sub-section headings print
their own totals; notes print across the full width; a combination line prints the combination name and quantity and,
underneath, its chosen items; a section summary row prints the aggregated quantity, discount and taxes of the section.

Totals: the same tax total block as the invoice. Then the signature block, showing the signature image and the signer
name when the order was signed online and the print context asks for it; the order note; the payment term note; and the
fiscal position remark under the title "Fiscal Position Remark:".

#### Purchase order and request for quotation

Entity: Purchase Order. Output: portable document. Layout: external. Print menu: purchase orders. File name: "Request
for Quotation - &lt;reference&gt;" while the document is a draft, sent or waiting for approval, "Purchase Order -
&lt;reference&gt;" afterwards.

Blocks: the vendor address; the shipping address when a destination address is set, under "Shipping address"; the title
("Request for Quotation #", "Purchase Order #" or "Cancelled Purchase Order #") with the reference; the information row
with Buyer, Your Order Reference, "Order Date:" (the approval date for a confirmed order, the order deadline
otherwise), and "Expected Arrival:".

Line table columns: Description, Quantity with its unit of measure (and the converted quantity in the reference unit),
Unit Price, Discount percentage, Taxes, Amount. Sections print a bold heading and, at the end of the section, a
"Subtotal" row with the section total. Notes print across the width.

Totals: the shared tax total block. Then the document note, the payment terms under "Payment Terms:", and, when the
vendor has a portal user and the document can be exchanged electronically, the invitation block "Connect your software
&lt;company&gt; to create quotes automatically."

The second definition, **Request for Quotation**, prints a shorter document intended to ask for prices: vendor address,
shipping address, requested shipping date, the title with the reference, and a line table with Description, Expected
Date, Quantity and unit of measure only, with no prices, followed by the document note and the same invitation block.

#### Purchase agreement

Entity: Purchase Agreement. Output: portable document. File name: "Purchase Agreement - &lt;reference&gt;". Prints the
agreement reference, its type, its validity dates, its responsible buyer, the products with the quantities requested,
and the list of vendors invited with the state of their answer.

### Inventory documents

#### Transfer operations sheet

Entity: Transfer. Output: portable document. Layout: report layout. File name: "Picking Operations - &lt;partner&gt; -
&lt;reference&gt;". Print menu: transfers.

Blocks: the operation kind and the transfer reference, with the reference also printed as a barcode; the delivery
address when the operation prints one; the customer address for an outgoing operation whose contact differs; the source
document; the warehouse; the contact for an internal operation; the status; the operator; the completion date for a done
transfer and the scheduled date otherwise; the operation instructions when the transfer carries a warning.

Line table columns: Product (with the transfer description underneath when it differs from the product name), Quantity
(with its unit of measure, and the package quantity and package unit in parentheses when the line is packed), From
(source location, and the source package when the whole package is not moved), To (destination location and destination
package), Barcode. The From, To and Barcode columns appear only when at least one line needs them. Lines that move an
entire package are listed separately, one row per package.

#### Delivery slip

Entity: Transfer. Output: portable document. Layout: external. File name: "Delivery Slip - &lt;partner&gt; -
&lt;reference&gt;". Print menu: transfers.

Blocks: the delivery address, or the warehouse address for a non-internal operation without one, or the vendor address
for an incoming operation, or the customer under the title "Customer"; the operation kind name with the transfer
reference; the source document; the contact for an internal operation; the shipping date (the completion date for a
done transfer, the scheduled date otherwise); the operator.

Line table columns: Product (with the transfer description underneath), Ordered (the demanded quantity with its unit,
and the package quantity when the line is packed), Delivered (the done quantity with its unit). The Ordered column is
omitted when the transfer is done and the two quantities are equal. Serial and lot numbers are listed under their line
when the products are tracked, with the quantity per number. Packages are printed as their own blocks with their
contents.

#### Return slip

Entity: Transfer. Output: portable document. Print menu: transfers. Prints the return instructions, the address the
goods must be returned to, the original transfer reference, the products with the quantities to return, and the
carrier's return code when one was generated.

#### Package contents

Entity: Transfer. Output: portable document. File name: "Packages - &lt;reference&gt;". Print menu: transfers. Prints
one block per package of the transfer: the package name as a barcode, the package type, the destination and the
contents with quantities, lots and serial numbers.

#### Inventory count sheet

Entity: Stock Quantity Record. Output: portable document. Layout: internal. File name: "Count Sheet". Print menu: stock
quantity records. Prints the title "Inventory Count Sheet", then, grouped by location in location order, a table with
the columns Location, Product, Lot/Serial Number, Package and Quantity. The Quantity column is left blank when the
installation hides expected quantities during a count, in order that the counter writes the figure by hand.

#### Reception report

Entity: Transfer. Output: portable document. Prints, for an incoming transfer, the products received and, per product,
the outgoing documents waiting for it, with their reference, their customer, their scheduled date and the quantity that
can be assigned.

#### Reception report labels

Entity: Stock Move. Output: portable document. Paper format: small label roll. Prints one label per assignment of a
received quantity: product name, quantity, destination document and destination location.

#### Location label, lot and serial number label, operation type label

Entities: Location, Lot or Serial Number, Operation Type. Output: portable document. Paper format: A4 label sheet.
File names: "Location - &lt;name&gt;", "Lot-Serial - &lt;name&gt;", "Operation-type - &lt;name&gt;". Each label prints
the name as text and as a barcode; the lot label additionally prints the product name and, when set, the expiry date.

#### Package barcode

Entities: Package, Package History. Output: portable document, in two sizes. The large form prints the package name as
text and barcode with the package contents underneath (product, quantity, lot or serial number); the small form prints
only the name and the barcode.

#### Product routes report

Entity: Product Template. Output: rich text. Prints, per product and warehouse, the supply chain that applies: the
rules in order, their action, their source and destination locations, their operation type and their lead time.

#### Batch transfer

Entity: Batch Transfer. Output: portable document. Print menu: batch transfers. Prints the batch reference and, per
transfer of the batch, its reference, its destination and its lines, sorted by the picking order of the batch.

#### Machine-readable labels

Five definitions produce label printer command text rather than a page: **Product Label**, **Lot and Serial Number
Label**, **Package Barcode**, **Package History Barcode**, **Packaging Barcode** and **Operation Type Label**. Their
output form is plain text; the file is sent to a label printer that interprets the command language. Each prints the
same values as the portable form of the same label: the name, the barcode and, for products, the price and the
reference.

### Manufacturing documents

#### Production order

Entity: Manufacturing Order. Output: portable document. Layout: internal. File name: "Production Order -
&lt;reference&gt;". Print menu: manufacturing orders.

Blocks: the reference as text and as a barcode; the source document; the responsible user; the deadline (printed only
while the order is neither done nor cancelled); the product; the variant description; the quantity to produce with its
unit; the quantity being produced when one is set.

Then the operations table, titled "Operations Done" for a done order, "Operations Planned" for a planned order and
"Operations" otherwise, with the columns Operation, Work Centre, Duration in minutes (the expected duration, replaced by
the recorded duration once the order is done), Actual Duration in minutes (only while the order is in progress or ready
to close), and Barcode.

Then the components table: product, description, quantity to consume with its unit, the source location, and the lot or
serial numbers when the component is tracked.

#### Work order

Entity: Work Order. Output: portable document. Layout: internal. File name: "Work Order - &lt;reference&gt;". Prints
the operation name and the production reference as a barcode, the responsible user, the manufacturing order, the
finished product, the quantity to produce with its unit, the components of the operation, the work instructions and the
quality checks attached to it.

#### Bill of materials overview

Entity: Bill of Materials. Output: portable document. File name: "Bill of Materials Overview - &lt;display name&gt;". Prints the
finished product and quantity, then the exploded structure as an indented tree: per level the component, the quantity
per unit and in total, the unit of measure, the availability, the lead time, and the unit cost, with the accumulated
cost per level and the total cost at the bottom.

#### Manufacturing order overview

Entity: Manufacturing Order. Output: portable document. File name: "Manufacturing Order Overview - &lt;display name&gt;". Prints the
order, its components and its operations with, per line, the planned quantity, the consumed or produced quantity, the
planned cost, the real cost and the difference, and the totals of each column.

#### Finished product label

Entity: Manufacturing Order. Output: portable document on the A4 label sheet, and label printer command text in a second
definition. File name: "Finished products - &lt;reference&gt;". Prints one label per produced unit or per lot: the
product name, the lot or serial number as text and barcode, and the production date.

#### Work in progress report

Entity: Analytic Line. Output: portable document. Prints the value still held in production at the reporting date,
grouped by manufacturing order: the consumed component value, the recorded operation cost, the value of the finished
products already received, and the remaining balance.

### Human resources documents

#### Employee badge

Entity: Employee. Output: portable document. Paper format: employee badge. File name: "Badge - &lt;name&gt;". Print
menu: employees. Prints the company logo, the employee photograph, the employee name, the job title, the company name
and the badge barcode.

#### Employee resume

Entity: Employee. Output: portable document. Paper format: resume. File name: "CV - &lt;name&gt;". Prints the employee
photograph, name, job title, contact details, the biography, then the resume lines grouped by type (experience,
education, internal certification, and the other configured types) in reverse date order, then the skills grouped by
skill type with their level and progress, rendered in the two colours passed to the print action.

#### Expense report

Entity: Expense. Output: portable document. Layout: external. File name: "Expense - &lt;employee&gt; -
&lt;reference&gt;". Print menu: expenses. Prints the title "Expenses Report", the report name, the employee, the date,
the manager, the payment mode, then a table with the columns Name, Unit Price, Quantity, Taxes, Subtotal in currency
(only when at least one line is in a foreign currency) and Subtotal, followed by the totals Untaxed Amount, Taxes and
Total. A second definition, **Expense receipt images**, prints the attached receipt images, one per page, with no other content.

#### Time off summary

Entity: Time Off Allocation. Output: portable document. Paper format: time off summary, landscape. Prints the title
"Time Off Summary", the analysed period ("Analyze from &lt;start&gt; to &lt;end&gt;") and the kind of time off
selected, then a calendar grid: one column per day of the period grouped under its month, one row per department and one
row per employee of that department, each cell coloured by the time off type taken that day, and a final "Sum" column
with the number of days per employee. A legend at the bottom maps each colour to its time off type.

#### Timesheets

Entity: Analytic Line, and four further definitions that print the same table for a Task, a Project, a Sales Order and
a Journal Entry. Output: portable document. Layout: internal. Prints the title "Timesheets", followed by "for the
&lt;project&gt;" when all lines belong to one project, then a table with the columns Date, Employee, Project, Task,
Description and Hours Spent, sorted by date, with the total hours at the bottom. The sales variant adds the invoicing
state of each line and the invoiced amount.

### Point of sale documents

#### Sales details

Entity: Point of Sale Session. Output: portable document. Layout: basic. Print menu: sessions; also produced by the
sales detail endpoint for a date range. Prints the company, the period, the sessions included, then: total sales
excluding tax and including tax; one row per product sold with quantity and amount, grouped by product category; one row
per payment method with the number of payments and the amount; the taxes collected per tax with base and amount; the
opening and closing cash balances; the cash movements in and out with their reason; and the discounts granted with their
count and total.

#### User labels

Entity: User. Output: portable document. Prints one label per user with the user name and the badge barcode used to
identify the cashier at the station.

#### Self ordering codes page

Entity: Point of Sale Configuration. Output: portable document. Paper format: two-dimensional code page. File name:
"Quick Response codes". Prints one page per table with the table name and the two-dimensional code that opens the self ordering
surface for that table, plus a generic code for the counter.

### Product and pricing documents

#### Product labels

Entity: Product Template. Output: portable document. Paper format: A4 label sheet, in four densities: two columns by
seven rows, four by seven, four by twelve, and four by twelve without price. A fifth definition prints one label per
page on the small label roll. File name: "Products Labels - &lt;name&gt;". Each label prints the product name, the sales
price (except in the no-price density), and the barcode of the product or of the chosen packaging, rendered from the
product barcode or, when none exists, from the internal reference.

#### Packaging barcodes

Entity: Product Packaging Unit. Output: portable document on the label sheet, and label printer command text in a second
definition. File name: "Products packaging - &lt;unit name&gt;". Prints the product name, the packaging name, the
contained quantity and the packaging barcode.

#### Price list

Entity: Product Variant. Output: portable document. Prints the price list name, then a table whose first two columns are
Product and Unit of Measure and whose further columns are one per requested quantity, headed "Quantity (&lt;quantity&gt;
Units of Measure)", each cell holding the unit price of that product at that quantity under that price list. Variants
are printed under their template when the report is asked for templates.

### Event documents

#### Full page ticket

Entity: Event Registration. Output: portable document. Paper format: event full page ticket. File name: "Full Page
Ticket - &lt;event&gt; - &lt;attendee&gt;". Print menu: registrations. Prints the event name, its dates and place, the
attendee name and electronic mail address, the ticket type, the registration reference as a two-dimensional code for
scanning at the door, and the event description block authored by the organizer. A second definition, catalogued as **Full Page Ticket Example**, prints the same
ticket for an event rather than a registration, filled with sample values, to preview the layout.

#### Badge

Entity: Event Registration. Output: portable document. Paper format: event badge. File name: "Badge - &lt;event&gt; -
&lt;attendee&gt;". Prints the attendee name, the company, the ticket type and the registration barcode, in the badge
format chosen on the event: one badge per page, four badges per sheet, or the folded format. A second definition, catalogued as **Badge Example**, prints
the same badge for an event with sample values.

#### Responsive ticket

Entity: Event Registration. Output: portable document rendered from the message layout, used as the ticket embedded in
the confirmation message such that it displays correctly on a telephone.

#### Attendee list

Entities: Event, and Event Registration for a chosen selection. Output: portable document. File name: "Attendee List -
&lt;event&gt;" or "Attendee List". Prints the event name and date, then the attendees sorted by name with their
electronic mail address, telephone, company, ticket type and registration state, and the count per ticket type at the
bottom.

### Learning and survey documents

#### Certification

Entity: Survey Participation. Output: portable document. Paper format: certification, landscape. File name:
"Certification - &lt;certification name&gt;". Stored on the participation. Prints the certification background, the
name of the person, the certification title, the completion date and the score, with the signature block configured on
the certification.

### Marketing and loyalty documents

#### Coupon code

Entity: Loyalty Card. Output: portable document. Print menu: loyalty cards. Prints one block per card: the programme
name, the code as text and as a barcode, the validity dates and the rules of the programme in the language of the
customer.

#### Gift card

Entity: Loyalty Card. Output: portable document. Print menu: loyalty cards. Prints the gift card design with the card
code as text and barcode, the remaining amount, the expiry date and the terms, in the language of the customer.

### Project and service documents

#### Repair order

Entity: Repair Order. Output: portable document. File name: "Repair Order - &lt;reference&gt;". Print menu: repair
orders. Prints the customer address, the reference, the product being repaired with its lot or serial number, the
responsible user, the scheduled date, then the parts table (product, quantity, unit price, taxes, amount) split between
parts to add and parts to remove, the totals, and the repair notes.

### Messaging documents

#### Live conversation transcript

Entity: Discussion Channel. Output: portable document. Prints the conversation header (visitor identity, operator, start
time, duration, rating) and then every message in order with its author, its time and its body, including the notes the
operator added.

### Platform documents

| Document | Entity | Content |
|---|---|---|
| Entity overview | Entity Definition | The entity, its description, its fields with type, label and required flag, its operations, its access rules and the capability packages that extend it. |
| Technical guide | Capability Package | The package, its declared dependencies, the entities it defines, the views it ships and the data records it loads. |
| Preview of the internal layout | Company | A sample document rendered with the internal layout, in order to check the layout settings of the company. |
| Preview of the external layout | Company | A sample document rendered with the external layout, with a sample address block, a sample line table and sample totals. |
| Layout preview | Company | The same sample rendered with the layout currently being chosen in the settings, refreshed at each change. |

### Country-specific documents

| Document | Entity | Country | Content |
|---|---|---|---|
| Delivery guide | Transfer | Argentina | The legally numbered delivery note: numbering, carrier, vehicle, origin and destination, and the goods with quantities. File name and stored attachment name both "Remito - &lt;number&gt;", with the placeholder "s/n" when the number is missing. |
| Payment slip with two-dimensional code | Journal Entry | Switzerland | The invoice with the national payment slip at the bottom, carrying the structured reference and the two-dimensional payment code. Paper format: European A4 without borders. File name "Quick Response bill-&lt;reference&gt;". |
| Accounting voucher | Journal Entry | China | The voucher form: journal, date, reference, and one row per journal item with account, label, debit and credit, plus the totals and the signature boxes. File name "Voucher_&lt;reference&gt;". |
| Point of sale hash integrity | Company | France | The chain verification result of the point of sale orders, per station and period. |
| Invoice in two copies, invoice in three copies | Journal Entry | India | The invoice document repeated twice or three times on separate pages, each copy labelled (original for the recipient, duplicate for the transporter, triplicate for the supplier). |
| Electronic waybill | Electronic Waybill | India | The transport document: the waybill number, its validity, the consignor and consignee, the transport details and the goods. Paper format: A4 for the waybill. File name "Ewaybill - &lt;number&gt;". |
| Transport document | Transfer | Italy | The legally numbered transport document with its number and date, the parties, the goods, the packages, the weight, the reason for transport and the carrier. File name "DDT - &lt;partner&gt; - &lt;number&gt;". |
| Commercial invoice | Journal Entry | Sri Lanka, Thailand | The invoice in the commercial form the country requires for customs and tax purposes. |
| Compliance letter | Company | Malta | The fiscal compliance statement of the point of sale software, printed for the tax authority. |
| Disbursement voucher | Payment | Philippines | The disbursement form with the payee, the amount in figures and in words, the accounts charged and the approval signatures. |
| Invoice generated by the system | Journal Entry | International | The printable rendering attached to an electronic invoice when the sender supplies only structured data, produced from that structured data. |

## Part 3: export mechanisms

### Exporting records from a list

Any list of records offers an export. The user selects records (or exports the whole filtered set when none is
selected), chooses the fields and the file format, and receives a file.

**Field selection.** The field chooser presents a tree of the exportable fields of the entity:

1. The available fields are the fields of the entity that are marked exportable. In import-compatible mode, read-only
   fields are removed as well, because they cannot be written back.
2. The technical identity field is offered under the label "External Identifier"; in normal mode a second entry "Database Identifier" is
   offered, which exports the internal record number.
3. Fields are sorted by their translated label, case-insensitively.
4. A relational field can be expanded: its own fields appear as children under the path
   `parent_field/child_field`. Expansion is limited to three path segments. In import-compatible mode a relation
   expands only to its identity field and its display name, because those are the two values an import can resolve; in
   normal mode every field of the related entity is offered.
5. Dynamic user-defined fields declared on a parent record are offered under their own labels, resolved from the records
   currently selected.
6. Fields marked as compatible with the default export are preselected when the user opens the chooser in
   import-compatible mode.

**Saved templates.** A field selection can be saved as an **export template** record holding a name, the entity it
applies to and its ordered list of field paths. Selecting a template loads its field list with the labels resolved for
the current entity; a field of the template that no longer exists is reported as unknown and dropped.

**Import-compatible mode.** When the user asks for a file that can be imported back, the column headers are the field
paths rather than the labels, the identity field is exported as the external identifier, and read-only fields are not
offered. When the mode is off, the column headers are the translated labels.

**Grouped export.** When the list is grouped and the format supports it, the export keeps the grouping: each group
becomes a header row carrying the group label, the record count and the aggregated values of the numeric columns
(sum, or average for fields declared as averages), and the records of the group follow indented under it. Nested groups
produce nested headers. The comma-separated format refuses a grouped export with the message, reproduced verbatim,
`"Exporting grouped data to csv is not supported."`

**Row production.** Records are read in the order of the list. Values are converted with the export conversion of each
field type: a date as its stored date, a date and time as the stored moment converted to the time zone of the user, a
selection as its translated label, a link to one record as its display name (or its external identifier in
import-compatible mode), a link to many records as the display names joined by a comma, a binary value as its encoded
text. Records are read in batches, therefore a large export does not hold everything in memory.

**Comma-separated values file.** Encoding: text with the comma as separator and every value quoted. The first row holds
the column headers. An empty value is written as an empty string. A value that begins with an equals sign, a plus sign
or a minus sign is prefixed with an apostrophe, because spreadsheet applications would otherwise read it as a formula.
The media type is the comma-separated text type and the file name is "&lt;entity label&gt; (&lt;entity name&gt;)" with the comma-separated values extension.

**Spreadsheet workbook file.** One sheet, the first row holding the headers in bold. Cell formats: dates use the date
format of the language, date and time values use the date and time format, decimal values use a thousands-separated
numeric format with two decimals, monetary values use a numeric format with the decimal places of the currency, and
everything else is written as text. A text value longer than the maximum a cell accepts is replaced by the message "The
content of this cell is too long for an XLSX file (more than &lt;maximum&gt; characters). Please use the comma-separated values format for
this export." A binary value that is not text-decodable raises "Binary fields can not be exported to Excel unless their
content is base64-encoded. That does not seem to be the case for &lt;column&gt;." Carriage returns inside a value are
replaced by spaces. Group header rows are bold and carry the aggregated values in the matching numeric format. Media
type is the spreadsheet workbook type; file name "&lt;entity label&gt; (&lt;entity name&gt;)" with the spreadsheet workbook extension.

**Auditing.** Every export writes a log line naming the user, the number of records, the entity, the network address of
the caller, the exported field paths, and either the first ten record identifiers or the filter condition used.

**Failure.** Any failure during the export is answered with the server-error status and a structured error body, in
order that the client can show the reason rather than downloading a broken file.

### Exporting an analysis table

A pivot table is exported as a spreadsheet workbook: the row groups become the leading columns, the column groups become
the header rows, the measures fill the cells, and the totals of each axis are written as their own row and column. The
formatting rules are those of the list export.

### Exporting a spreadsheet or dashboard

A spreadsheet document and a shared dashboard are downloaded as a spreadsheet workbook containing their sheets, their
values and their formatting. A dashboard shared by link is downloaded through its sharing address, which carries the
sharing token instead of requiring a session.

### Exporting a price list

The price list report can be downloaded as a comma-separated values file or as a spreadsheet workbook instead of a
printable document. The columns are Product, Unit of Measure, and one column per requested quantity headed
"Quantity (&lt;quantity&gt; Units of Measure)". Rows are the selected products, with one row per variant when variants
are requested. In the workbook form each column is widened to its longest value. The file is named
"Pricelist - &lt;price list name&gt;" with the extension of the chosen form.

### Documents downloaded from the portal

Portal pages offer their own downloads, each rendered with the report of the document and delivered with the file name
rule of that report: the invoice, the quotation or order, the delivery slip, the return label, the subcontracting
production sheet, the certification, the ticket, the badge, the transcript of a live conversation, and the structured
interchange form of an order or a bill. Each is authorized either by the access rights of the signed-in portal user or
by the access token of the document.

## Part 4: importing records

### Accepted files

| Form | Detection | Notes |
|---|---|---|
| Comma-separated values | the comma-separated text media type | Encoding, separator and quoting are detected and can be overridden. |
| Spreadsheet workbook, current form | the media type of the current workbook form | One sheet is imported at a time; the sheet is chosen by the user, defaulting to the first. |
| Spreadsheet workbook, earlier form | the media type of the earlier workbook form | Same sheet handling. |
| Open document spreadsheet | the media type of the open document spreadsheet | Same sheet handling. |

Any other file is refused with "Unsupported file format \"&lt;type&gt;\", import only supports comma-separated values, open document spreadsheet and spreadsheet workbook files".
A file whose reader library is missing is refused with "Unable to load \"&lt;extension&gt;\" file: requires the reader
component \"&lt;component&gt;\"" (a replacement states its own equivalent).

### Options

| Option | Default | Effect |
|---|---|---|
| `encoding` | detected | The character encoding of a text file. A decoding failure reports "There was an issue decoding the file using encoding “&lt;encoding&gt;”." followed by whether the encoding was detected automatically or chosen by the user. A byte-order mark is stripped when present. |
| `separator` | inferred | The column separator of a text file. Inference tries the candidates and keeps the one that yields a consistent column count on every row. |
| `quoting` | double quote | The text delimiter; it must be exactly one character, otherwise "Error while importing records: Text Delimiter should be a single character." |
| `has_headers` | true | Whether the first row holds column names. |
| `skip` | 0 | How many data rows to skip, used to resume an interrupted import. |
| `limit` | none | How many rows to process in this batch; the answer carries the next row to continue from. |
| `sheet` | first sheet | Which sheet of a workbook to read. |
| `date_format`, `datetime_format` | detected | The patterns used to read dates. Detection tries the pattern of the language first, then a fixed list of common patterns, and keeps the first that parses every sampled value. |
| `float_thousand_separator`, `float_decimal_separator` | detected | The number separators. Detection looks at the sampled values: a value with several occurrences of a separator makes that separator the thousands separator, and the pair is then fixed for the whole file. |
| `advanced` | false | Whether relational sub-fields may be mapped. |
| `keep_matches` | false | Whether to keep the mapping the user already adjusted rather than proposing it again. |
| `fallback_values` | none | Per column, the value to write when a cell is empty or unparseable, used for selection and boolean columns. |
| `name_create_enabled_fields` | none | Per relational column, whether a missing related record may be created from its name. |
| `import_set_empty_fields` | none | The columns whose empty cells overwrite the stored value instead of being ignored. |
| `import_skip_records` | none | The columns whose empty cells cause the whole row to be skipped. |

### Column to field matching

1. The header is compared with the field labels and the field names of the entity, case-insensitively, including nested
   paths written with a separator.
2. When a previous import of the same entity mapped the same header, that mapping is proposed again. Each successful
   run stores or updates the mapping of every header, which is how a recurring import from the same outside system
   becomes one click.
3. When nothing matches exactly, the closest field by string distance is proposed, restricted to fields whose type can
   hold the sampled values: a column whose values parse as dates proposes only date fields, one whose values parse as
   numbers proposes only numeric fields, one whose values are `true` or `false` proposes only boolean fields, and any
   other column proposes only text-like fields.
4. Duplicate suggestions are resolved in favour of the shorter path, and the loser falls back to its next best match.
5. A column mapped to nothing is ignored. At least one column must be mapped, otherwise the import is refused with "You
   must configure at least one field to import".

### Resolving relations

A column that targets a link to another record accepts three shapes, distinguished by the suffix of the field path:

| Path | Value | Behaviour |
|---|---|---|
| `field` | the display name | The related record is searched by name. No match, or several matches, is an error, unless creation from the name is enabled for that column, in which case the record is created. |
| `field/id` | the external identifier | The record carrying that external identifier is used. An unknown identifier is an error. |
| `field/.id` | the internal record number | The record carrying that database identifier is used. |

For a link to many records, the values are separated by commas. For a list of child records, the child columns are
written on the row of the parent and on the following rows that leave the parent columns empty.

### Creating and updating

The identity column decides the mode. A row with an external identifier that already exists updates that record; a row
with a new external identifier creates the record and registers the identifier; a row with no identity column always
creates. The import runs in batches: a batch is attempted as one operation and, when it fails, each row of the batch is
retried alone in order to attribute the error to a row. Every failure rolls back to the savepoint taken before the row,
therefore a failed row never leaves a half-written record.

### Test run

A test run performs every validation and every write and then rolls everything back, and it clears the caches the
rolled-back writes touched. It returns the same message list as a real run, which lets a user see every error before
committing anything.

### Result and messages

The answer carries: the identifiers of the records written, the next row to continue from when the batch limit was
reached, the list of names of the imported records, and a list of messages. Each message carries a kind (`error` or
`warning`), a text, the row number, the field path when the error belongs to one field, and the offending record data.
Representative texts: "Import file has no content or is corrupt"; "Column &lt;column&gt; contains incorrect values
(value: &lt;value&gt;)"; "Column &lt;column&gt; contains incorrect values. Error in line &lt;line&gt;:
&lt;error&gt;"; "Error Parsing Date [&lt;field&gt;:L&lt;line&gt;]: &lt;error&gt;"; "Field '&lt;field&gt;' does not
accept date/time values."; "Invalid cell value at row &lt;row&gt;, column &lt;column&gt;: &lt;value&gt;"; "Invalid cell
format at row &lt;row&gt;, column &lt;column&gt;: &lt;value&gt;, with format: &lt;format&gt;, as (&lt;kind&gt;) formats
are not supported."; "Found invalid image data, images should be imported as either URLs or base64-encoded data.";
"You can not import file via Web Address, check with your administrator or support for the reason."; "File size exceeds
configured maximum (&lt;bytes&gt; bytes)"; "Could not retrieve Web Address: &lt;address&gt; [&lt;field&gt;:L&lt;line&gt;]:
&lt;error&gt;"; "Unknown database error: '&lt;error&gt;'". A uniqueness or check violation of the data store is
translated into the message of that constraint and attributed to the field when the failing column can be identified.

### Files and images inside an import

A binary column accepts either encoded content or an address to fetch. Fetching an address is allowed only when the
installation enables it; the fetched file is limited by the configured maximum size. The file name of a fetched or
supplied document is kept and reported back in the result, which lets the client show which attachments were created.

### Shipped import templates

Each of these entities offers a downloadable starter file whose columns are the fields the import expects, with one
sample row.

| Entity | Template label |
|---|---|
| Lead | Import Template for Leads & Opportunities |
| Employee | Import Template for Employees |
| Bill of Materials | Import Template for Bills of Materials |
| Journal Entry (miscellaneous) | Import Template for Misc. Operations |
| Journal Entry (customer invoices and credit notes) | Import Template for Invoices |
| Account | Import Template for Chart of Accounts |
| Journal Item | Import Template for Journal Items |
| Task | Import Template for Tasks |
| Purchase Order | Import Template for Requests for Quotation |
| Vendor Price List | Import Template for Vendor Pricelists |
| Price List | Import Template for Pricelists |
| Product | Import Template for Products (a purchasing variant and a sales variant exist, each with the fields of that application) |
| Time Entry | Import Template for Timesheets |
| Stock Quantity Record | Import Template for Inventory Adjustments |
| Sales Order | Import Template for Quotations |
| Address Redirection | Import Template for Redirects |

## Part 5: data files produced by the domains

### Electronic invoice and interchange documents

Structured invoice and order documents are produced by the electronic invoicing domain and are attached to the document
they describe; they are also transmitted over the document exchange networks. Their formats, their profiles per country
and their validation rules are specified in
[`../domains/electronic-invoicing-and-document-exchange/README.md`](../domains/electronic-invoicing-and-document-exchange/README.md);
the transport contract is in [`external-integrations.md`](external-integrations.md). The download endpoints that serve
them are catalogued in [`endpoint-catalog.md`](endpoint-catalog.md): one invoice returns one file, several invoices are
returned as a compressed archive, and a request that cannot produce the structured form for a single invoice fails with
"Error while creating Markup Document:" followed by the list of reasons.

### Accounting audit file

An accounting audit export produces one flat file of every journal item of a period, for the tax authority.

| Property | Value |
|---|---|
| Trigger | The audit export assistant, with a start date, an end date, a selection of journals to exclude and a flag that marks the file as a test. |
| Form | Text file, one row per journal item, values separated by the vertical bar, rows terminated by carriage return and line feed, with a header row. |
| Columns | Journal code; journal label; entry number; entry date; account number; account label; auxiliary account number (the contact reference); auxiliary account label; document reference; document date; entry label; debit; credit; reconciliation code; reconciliation date; validation date; amount in foreign currency; currency code. |
| Opening balances | Before the movement rows, the export writes the opening balance rows: one row per account carrying the balance brought forward at the start date, and one row carrying the result of the previous years on the retained earnings account. |
| File name | The legal identification of the company, then the letters of the audit file, then the end date without separators, then the suffix for a non-official export, then the text extension. |
| Side effect | A non-test export sets the fiscal year lock date of the company to the end date of the export, which freezes the exported period. |
| Ordering | Journal, then entry number, then line sequence. |

Other countries ship their own audit or declaration files with their own column sets; each is produced by the same
pattern: an assistant that takes a period, a streamed file, a name that encodes the company and the period, and a lock
of the exported period.

### Payment instruments and bank files

| File | Produced by | Content |
|---|---|---|
| Printed cheque | The payment, when the payment method is cheque printing | The cheque form of the chosen layout, with the payee, the amount in figures and in words, the date, the memo and the cheque number, plus one or two detachable stubs listing the invoices settled with their reference, date, total, discount and amount paid. When more invoices exist than fit on the stubs, the stub is cropped and marked as such; a company setting decides whether several stub pages are printed instead. A batch prints the cheques in cheque-number order, one per page. |
| Batch deposit slip | A deposit of received cheques | The deposit form on its own paper format, listing each cheque with its payer, its number and its amount, with the total and the deposit date. |
| Structured payment reference | The invoice and the payment | Not a file: the structured creditor reference printed on the invoice and carried in the payment message, checked by its check digits. |
| Payment code image | The invoice and the portal | The two-dimensional payment code, generated from the bank account, the amount, the currency and the structured reference, embedded in the printed invoice and in the portal page. |

Files for automated bank transfers (credit transfer and direct debit instruction files) are produced by the localization
packages that implement the national or regional scheme; each writes the file shaped by that scheme, attaches it to the
payment batch and marks the batch as sent. The generic behavior stated here is an industry-standard completion, because
the scheme implementations are delivered separately: one file per batch, per journal and per execution date; the file
carries the creditor identification, the account numbers, the amounts, the currencies, the execution date and the
end-to-end references, which are the payment references of the platform; a batch already marked as sent is not written
again, and regenerating it requires resetting the batch, which is what prevents a double instruction.

### Label files

| File | Form | Content |
|---|---|---|
| Product labels, packaging labels, lot and serial labels, location labels, operation type labels, package labels, reception labels, finished product labels | Portable document on a label sheet, or label printer command text | The name, the barcode and, where the label carries it, the price, the reference and the expiry date. The sheet forms tile the labels across the page in the chosen density and start at the label the user selects, which lets a partly used sheet be reused. |
| Two-dimensional code pages | Portable document | One code per table or counter for the self ordering surface. |
| Badges and tickets | Portable document | The event and employee badges and tickets described in the catalog. |

### Files served to outside systems

| File | Address | Content |
|---|---|---|
| Product feed for shopping advertisement | The feed address of the site, authorized by the feed token | One entry per published product: the identifier (the internal reference, or the record number when none), the title, the description, the link to the product page in the language and price list of the feed, the image links, the price and the sale price, the availability, the product identity codes, the brand, the category and the shipping data. The feed is rendered in the language of the feed and with its price list. |
| Page index for crawlers | The index address of the site | One entry per published page with its address and its last change date, regenerated when the stored index is older than the configured age. |
| Crawler instruction file | The instruction address | A deny-all rule followed by the explicit allow rules the installed capability packages contribute. |
| Calendar files | The calendar addresses of an event and of a talk | The event or talk as a calendar entry with its title, its start and end, its place, its description and its organizer. |
| Electronic business card | The card address of a contact | The contact as a business card file with name, company, job title, addresses, telephone numbers, electronic mail address and website. |
| Application descriptors | The descriptor addresses | The name, colours, icons, start address and scope of the installable application. |

### Archives

Two endpoints produce compressed archives: the attachments of a set of journal entries, and the legal documents of a set
of invoices. In both, a single selected document is returned on its own; several are packed into one archive, and names
that would collide are suffixed with a counter in order that every member keeps a distinct name.

## Part 6: rules

| Rule | Statement |
|---|---|
| RPT-RULE-001 | A printed document is rendered with the rights of the caller. A record the caller may not read makes the whole request fail. |
| RPT-RULE-002 | A report that names an attachment stores its output on each record, replacing an attachment of the same name. |
| RPT-RULE-003 | A report that reuses stored attachments returns the stored file unchanged; it never renders again for a record that already has one. |
| RPT-RULE-004 | The rendering language is the language of the print context, else the language of the addressed partner, else the language of the printing user. |
| RPT-RULE-005 | A report restricted to access groups is neither listed nor renderable for a user outside those groups. |
| RPT-RULE-006 | Page numbering is rendered only in the portable form, as the current page over the page count. |
| EXP-RULE-001 | An export in import-compatible mode uses field paths as headers and exports identities as external identifiers; an export not in that mode uses translated labels. |
| EXP-RULE-002 | A value that begins with an equals sign, a plus sign or a minus sign is prefixed with an apostrophe in the comma-separated form. |
| EXP-RULE-003 | A grouped export is refused in the comma-separated form and produces group header rows with aggregates in the workbook form. |
| EXP-RULE-004 | Every export writes an audit log line naming the user, the record count, the entity, the caller address, the fields and the selection. |
| IMP-RULE-001 | An import must map at least one column to a field. |
| IMP-RULE-002 | A row whose external identifier exists updates that record; a row whose identifier is new creates one; a row without an identity column always creates. |
| IMP-RULE-003 | A failing row is rolled back to the savepoint taken before it and reported with its row number, its field and its data; the remaining rows continue. |
| IMP-RULE-004 | A test run performs every write and rolls everything back, and returns the same messages as a real run. |
| IMP-RULE-005 | A successful import stores the mapping of each header for the entity, and that mapping is proposed on the next import. |
| IMP-RULE-006 | An empty cell leaves the stored value untouched unless the column is declared as one whose empty cells overwrite, and a row is skipped when a column declared as skip-on-empty is empty. |

### Acceptance criteria

**AC-REPORT-001 — Invoice document content and storage.** Given a posted customer invoice with two lines, one of which
carries a discount, when the invoice document is printed, then the line table shows the Discount column, the totals show
the untaxed amount, one row per tax group and the total, the rendered file is stored on the invoice under the invoice
reference, and printing it again returns the stored file byte for byte.

**AC-REPORT-002 — Language of a printed document.** Given an invoice addressed to a customer whose language is French,
when the invoice document is printed by a user whose language is English, then the document is rendered in French,
including the field labels, the payment term note and the terms and conditions.

**AC-REPORT-003 — Grouped export to a workbook.** Given a list of two hundred contacts grouped by country, when the user
exports the list to a spreadsheet workbook with the fields Name and City, then the file contains one header row per
country carrying the country label and the record count, and the contacts of each country follow under their header.

**AC-REPORT-004 — Grouped export to the flat format.** Given the same list, when the user exports it to a
comma-separated values file, then the export is refused with the message `"Exporting grouped data to csv is not
supported."`.

**AC-REPORT-005 — Remembered column mapping.** Given an import file whose first column is headed "Customer" and whose
values are customer names, and a previous import of the same entity that mapped the header "Customer" to the partner
field, when the user opens the mapping step, then the column is proposed as the partner field without any user action.

**AC-REPORT-006 — Test run of an import.** Given an import file of ten rows where row seven carries a date that does not
parse, when the user runs a test import, then no record is written, the result carries one error message naming row
seven, the field and the unparseable value, and the other nine rows are reported as importable.

**AC-REPORT-007 — Access is enforced by the renderer.** Given a user who may not read one of three selected invoices,
when the invoice document is printed for the three of them, then the whole request fails with the access refusal and no
file is produced for the two readable ones.

**AC-REPORT-008 — A report restricted to a group.** Given a report definition restricted to an access group and a user
outside it, when the user opens the print menu of the entity, then the report is not listed, and invoking it directly
fails with the access refusal.

**AC-REPORT-009 — Paper format selection.** Given a report definition that names its own paper format and a company that
declares a different default, when the report is rendered, then the report's own paper format decides the page size, the
orientation and the margins, and the company default is ignored.

**AC-REPORT-010 — Layout selection.** Given a company whose document layout is the second of the shipped layouts, when
any report that uses the external layout is rendered, then the header, the footer, the colours and the font of that
layout are applied, and the preview action renders the same sample with the same layout.

**AC-REPORT-011 — Page numbering.** Given a document that spans three pages, when it is rendered in the portable form,
then each page carries its number over the page count; when the same document is rendered as markup, then no page
numbering appears.

**AC-REPORT-012 — Re-importable export round trip.** Given twenty contacts exported in the re-importable mode with the
external identifier column, when the produced file is imported into the same installation without any change, then the
same twenty records are updated, none is created, and the answer reports twenty identifiers.

**AC-REPORT-013 — Formula protection in the flat form.** Given a contact whose name begins with an equals sign, when the
list is exported to the comma-separated form, then the cell is prefixed with an apostrophe.

**AC-REPORT-014 — Empty cells on import.** Given an import file with a column declared as one whose empty cells
overwrite and a row whose cell in that column is empty, when the file is imported, then the stored value of that field
is cleared; given the same file with the column not so declared, then the stored value is left untouched.

**AC-REPORT-015 — Structured invoice download.** Given a posted customer invoice, when the structured form is requested
for it alone, then one file is returned; when it is requested for three invoices, then a compressed archive of three
files is returned; when it is requested for an invoice whose data cannot produce the structured form, then the request
fails and the answer names the reasons.

## Reconciliation notes

The drafts merged into this document were checked against the source of the system and against the generated catalogue
[`../references/reports.md`](../references/reports.md), which lists the ninety-four report definitions with their
entity, output kind, template, file-name rule and attachment rule. The points that had to be settled:

1. **The refusal message of a grouped flat export.** One draft paraphrased it as "Exporting grouped data to
   comma-separated values is not supported." The message the system shows, reproduced verbatim, is `"Exporting grouped
   data to csv is not supported."`; the abbreviation is part of the string and is therefore kept. The other interface
   documents already quoted it correctly.
2. **The paper format table.** One draft listed nineteen rows, of which one, an "exhibitor page" format, is not a
   format of its own: it is an amendment that a further capability package makes to the event full page ticket format,
   raising its bottom margin from 8 to 29 millimetres to leave room for the sponsor images. The table now lists the
   eighteen formats the installation ships and states the amendment underneath.
3. **Two document variants were described but not findable by their catalogued name.** The sample ticket and the sample
   badge printed from an event rather than from a registration are described inside the ticket and badge entries; their
   catalogued names are now given there, so that every one of the ninety-four rows of the catalogue can be traced to the
   paragraph that describes it.
4. **Acceptance criteria.** One draft ended with six unnumbered scenarios in plain blocks. They are now numbered
   scenarios with stable identifiers, and nine further scenarios were added to cover access enforcement, group-restricted
   reports, paper format selection, layout selection, page numbering, the re-importable round trip, formula protection,
   the two kinds of empty cell on import, and the structured invoice download.
5. **The field table of the report definition.** One draft named the fields of the report definition with invented,
   readable spellings — an entity field, an output-kind field, a template field, a file-name rule, an attachment-name
   rule, a reuse flag, a print-menu binding, a paper-format field, a groups field and a filter condition — and presented
   the output kind as a selection over three descriptive values. None of those spellings is stored, and a replacement
   importing an existing database from that table would have written the wrong column names and the wrong stored
   values. The table now reproduces the stored identifiers `name`, `model`, `model_id`, `report_type`, `report_name`,
   `report_file`, `print_report_name`, `attachment`, `attachment_use`, `binding_model_id`, `binding_type`,
   `binding_view_types`, `multi`, `paperformat_id`, `group_ids` and `domain`, carries a column for the full name of
   each as the documentation rules require of a field table, and gives the stored selection values of the output form
   as `qweb-html`, `qweb-pdf` and `qweb-text` with `qweb-pdf` as the default. The same identifiers and the same values
   are used by [`../runtime/report-rendering.md`](../runtime/report-rendering.md), section 3, and both were verified
   against the source of the system.
6. **Part 1 and the runtime document.** The rendering model is the subject of
   [`../runtime/report-rendering.md`](../runtime/report-rendering.md). Part 1 of this document previously restated parts
   of it without linking to it once, which is how the invented field names entered. Part 1 now opens by naming that
   document as the authority for the rendering model, and each of its five subsections points at the section of it that
   carries the topic in full: section 3 for the report definition, section 4 for the pipeline, section 4.14 for the
   language per recipient, section 5 for paper formats and section 6 for the layouts, headers and footers. What Part 1
   keeps is what a reader of the document catalogue in Part 2 needs in order to read an entry: the configurable fields
   of a definition, the order of the steps, the language decision, the blocks of the shared layout and the geometry of
   the eighteen paper formats of the whole installation, of which the runtime document's own table lists five. The two
   documents agree on every value they both state.
