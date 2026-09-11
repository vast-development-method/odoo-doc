# Accounts Payable — Interfaces

---

## 1. Navigation

| Path | Target | Notes |
|---|---|---|
| Accounting → **Vendors** | a menu group, sequence 3 | groups everything payable |
| Accounting → Vendors → **Bills** | the *Bills* window action | the main payable list |
| Accounting → Vendors → **Refunds** | the *Refunds* window action | vendor credit notes |
| Accounting → Vendors → **Vendors** | the supplier partner list | |
| Accounting → Reporting → **Invoice Analysis** | the *Invoices Analysis* window action | the same report entity, filtered to customers |
| The accounting dashboard, purchase journal card | drag-and-drop upload zone, *Create invoice/bill*, *Bills to pay*, *to review*, *Bills Analysis* | |
| The accounting dashboard, bank journal card | *Checks to print* counter, when at least one cheque is waiting | |

### 1.1 Window actions

| Action | Title | Entity | Path segment | Views | Domain | Default context |
|---|---|---|---|---|---|---|
| Bills (by type) | Bills | Journal Entry | `bills` | list, kanban, form, activity | type is one of `in_invoice`, `in_refund`, `in_receipt` | default type `in_invoice`; show the bank-account trust indicator |
| Bills (menu) | Bills | Journal Entry | `vendor-bills` | list, kanban, form, activity | the same | the same, plus the *Bills* and *Receipt* search filters pre-applied |
| Refunds | Refunds | Journal Entry | `vendor-refunds` | list, kanban, form, activity | type is `in_invoice` or `in_refund` | default type `in_refund`; the *Refunds* search filter pre-applied |
| Bills Analysis | Bills Analysis | Invoice Analysis Report | `vendor-bills-analysis` | graph, pivot | — | the *Invoiced* and *Vendors* filters pre-applied, grouped by bill month |
| Invoices Analysis | Invoices Analysis | Invoice Analysis Report | `customer-invoices-analysis` | graph, pivot | — | the *Invoiced* and *Customers* filters pre-applied, grouped by bill month |
| Amounts to Settle | Amounts to Settle | Journal Item | — | list | — | the payable and receivable lines awaiting settlement |
| Checks to Print | Checks to Print | Payment | — | list, form, graph | — | the *Checks to Print* filter, the journal, the outbound direction and the cheque method pre-set |
| Reverse Moves | Reverse Moves | Journal Entry | — | form or list | the produced reversals | the produced type as default |
| Debit Notes | Debit Notes | Journal Entry | — | form or list | the produced debit notes | the produced type as default |
| Generated Documents | Generated Documents | Journal Entry | — | form when one, else list, kanban, form | the produced documents | inherited |

Empty-state texts:

- Bills: *Create a vendor bill* / *Capture invoices, register payments and keep track of the discussions with your vendors.*
- Refunds: *Create a vendor credit note* / *Note that the easiest way to create a vendor credit note is to do it directly from the vendor bill.*
- Bills Analysis: *From this report, you can have an overview of the amount invoiced from your vendors. The search tool can also be used to personalise your Invoices reports and so, match this analysis to your needs.*

---

## 2. The bill form

### 2.1 Header

| Element | Visibility |
|---|---|
| **Confirm** | the document is draft |
| **Reset to Draft** | the reset-to-draft indicator is true |
| **Request Cancellation** | the document needs a cancellation request |
| **Cancel** | the document is draft |
| **Register Payment** | the document is posted and unpaid |
| **Set as Reviewed** | the document is posted and not yet reviewed; restricted to the accountant group |
| **Add Credit Note** / **Reverse** | the document is posted |
| **Add Debit Note** | *(Debit Notes package.)* the document is posted |
| Status bar | draft → posted; the secured variant of the widget is shown to the inalterability group |

### 2.2 Banners, in display order

| Key | Level | Text |
|---|---|---|
| `account_tax_lock_date` | warning | *The date is being set prior to: «lock dates». The Journal Entry will be accounted on «date» upon posting.* Draft documents only; visible to the read-only accounting or invoicing groups |
| `account_auto_post_at_date` | information | *This move is configured to be posted automatically at the accounting date: «date».* Draft documents only |
| `account_auto_post_on_period` | information | *«mode» auto-posting enabled. Next accounting date: «date».*, extended with *The recurrence will end on «date» (included).* Draft documents only |
| `account_remove_empty_lines` | information | *We've noticed some empty lines on your invoice.* with the action *Remove empty lines*, which deletes them. Shown only on a **purchase** document in draft with **two or more** lines whose total is zero |
| `account_is_being_sent` | information | *This invoice is being sent in the background.* |
| `account_partner_credit_warning` | warning | the partner credit warning text; visible to the read-only accounting or invoicing groups |
| `account_abnormal_amount_warning` | warning | the abnormal-amount text of `calculations.md` §12.3 |
| `account_abnormal_date_warning` | warning | the abnormal-date text of `calculations.md` §12.2 |
| duplicate, exact | danger (red) | *This document might be a duplicate of* + a button naming the first duplicate (showing its total) + *Delete duplicate* / *Delete all duplicates* when a duplicate is still draft |
| duplicate, probable | warning (amber) | the same, shown when duplicates exist, the exact flag is not set and the document is draft |
| outstanding debits | information | *You have outstanding debits listed below for this vendor.* Shown on a posted `in_invoice` or `in_receipt` that is not fully paid and has outstanding counterparts; visible to the invoicing or read-only accounting groups |

### 2.3 Body — the payable-specific fields

| Field | Placement | Notes |
|---|---|---|
| **Vendor** | first | required to post |
| **Vendor Reference** (`ref`) | beside the vendor, in the number block | shown only for `in_invoice`, `in_receipt`, `in_refund`; it carries the form's default focus |
| **Bill Date** (`invoice_date`) | date block | read-only once posted; decorated as a warning when the abnormal-date warning is set |
| **Accounting Date** (`date`) | date block | |
| **Due Date** (`invoice_date_due`) | date block | shown for every invoice-like type |
| **Payment Terms** | date block | |
| **Recipient Bank** (`partner_bank_id`) | payment block | |
| **Payment Reference** | payment block | |
| Incoterm and Incoterm Location | other information | hidden on receipts |
| Deductibility (`deductible_amount`) column | line list | shown to the partial-purchase-deductibility group |
| Journal, currency, currency rate, fiscal position, cash rounding | other information | |
| **Total (Tax inc.)** | totals block | quick encoding only |
| Amount in words | totals block | |

### 2.4 Line list columns

Product, label, deductibility, account, analytic distribution, quantity, unit, unit price, discount, taxes, subtotal. Section, subsection and note rows carry only a label.

---

## 3. Lists, search and grouping

### 3.1 The bill list

Columns: number (decorated in red when the document made a numbering gap while posted; rendered by the "reviewed badge" widget so that an unreviewed posted document is marked), vendor (labelled **Vendor** for purchase types), bill date (labelled **Bill Date**), due date, vendor reference, total, amount due, status, payment status. The duplicate columns are loaded but hidden so that custom lists can shade cells.

### 3.2 The bill search view

It is the invoice search view re-labelled and re-filtered:

| Element | Change for bills |
|---|---|
| the number field | labelled **Bill** |
| *My Invoices* filter | removed |
| *Invoice Date* filter | labelled **Bill Date** |
| *Salesperson* grouping | removed |
| *Invoices* filter | replaced by **Bills** (type is `in_invoice`) |
| *Receipts* filter | replaced by **Receipt** (type is `in_receipt`) |
| *Credit Notes* filter | replaced by **Refunds** (type is `in_refund`) |
| *Not Sent* filter | hidden for purchase types |
| *Accounting Date* filter | visible for purchase types (it is hidden for sale types) |

Filters inherited unchanged: *Draft*, *Posted*, *Cancelled*, *Not Secured* (to the inalterability group), **To Review** (`checked` is false and the status is not draft), **To pay** (status is not cancelled, payment status is `not_paid` or `partial`, type is not a miscellaneous entry), **In payment**, **Overdue** (*Overdue invoices, maturity date passed*: due date before today, posted, payment status `not_paid` or `partial`, not a miscellaneous entry), the three date filters, and the activity filters.

Groupings: Partner, Status, Payment Method (to the invoicing or read-only accounting groups), Journal, Company (multi-company only), Bill Date, Due Date, Accounting Date, Sequence Prefix (hidden).

On the journal-item search view the **To Review** filter reads *the document's reviewed flag is false and the parent status is not draft*.

A separate primary search view adds **Irregular Sequences**: documents that made a gap, **or** that are draft or cancelled with a non-zero sequence number and a number other than `/`.

On the journal-item list a **Bills** filter exists, defined as *amount due is below zero*, visible only when the journal type in the context is `purchase`.

### 3.3 The bill upload list

A list renderer used for the upload flow. It shades the **vendor reference** cell: red when the exact-duplicate flag is set, amber when duplicates exist and the document is draft. It also embeds the bill-capture guide component.

---

## 4. The analysis report views

### 4.1 List

Columns, in order, with their default visibility:

| Column | Label | Shown by default | Aggregation |
|---|---|---|---|
| `move_id` | Invoice Number | yes | — |
| `journal_id` | Journal | hidden | — |
| `partner_id` | Partner | yes | — |
| `country_id` | Country | hidden | — |
| `invoice_date` | Invoice Date | yes | — |
| `invoice_date_due` | Due Date | yes | — |
| `invoice_user_id` | Salesperson (avatar widget) | hidden | — |
| `product_categ_id` | Product Category | hidden | — |
| `product_id` | Product | yes | — |
| `company_id` | Company (multi-company only) | yes | — |
| `price_average` | Average Price | hidden | sum labelled *Total* — but the engine substitutes the weighted average of `calculations.md` §15.1 |
| `quantity` | Product Quantity | hidden | sum |
| `price_subtotal_currency` | Untaxed Amount in Currency | yes | sum |
| `price_subtotal` | Untaxed Amount | yes | sum |
| `price_total` | Total | yes | sum |
| `price_total_currency` | Total in Currency | yes | sum |
| `price_margin` | Margin | hidden | — |
| `inventory_value` | Inventory Value | hidden | sum |
| `state` | Invoice Status | hidden | — |
| `payment_state` | Payment Status | hidden | — |
| `move_type` | Type | hidden | — |

### 4.2 Pivot and graph

The pivot puts the product category in columns and the bill date in rows, measuring the untaxed amount. The graph is a line chart of the untaxed amount by product category. Both use sample data when empty.

### 4.3 Search view

| Element | Definition |
|---|---|
| *My Invoices* | salesperson is the current user |
| *To Invoice* | status is `draft` (help: *Draft Invoices*) |
| *Invoiced* | status is neither `draft` nor `cancel` |
| *Customers* | type is `out_invoice` or `out_refund` |
| **Vendors** | type is `in_invoice` or `in_refund` |
| *Invoices* | type is `out_invoice` or `in_invoice` |
| *Credit Notes* | type is `out_refund` or `in_refund` |
| date filters | on the bill date and on the due date |
| searchable fields | partner (matching children too), salesperson, product, product category (matching child categories) |
| groupings | Salesperson, Partner, Product Category, Status, Company, Date (by default, or by week when the context asks), Due Date by month |

### 4.4 Shipped saved filters

| Name | Grouping | Domain |
|---|---|---|
| By Salespersons | bill month, then salesperson | — |
| By Product | bill month, then product | — |
| By Product Category | bill month, then product category | — |
| By Credit Note | bill month, then salesperson | type is `out_refund` |
| By Country | bill month, then country | — |

---

## 5. Journal and payment interfaces

### 5.1 Purchase journal form

The alias fields are shown when at least one alias domain exists. The reception address carries the help text *Send one separate email for each invoice.* / *Any file extension will be accepted.* / *Only PDF and XML files will be interpreted*.

### 5.2 Bank journal form — the cheque group

Placed inside the outgoing payment settings, in a group titled **Check Printing**, visible only when the journal's selected outgoing method codes contain `check_printing` and the journal type is `bank`. It shows: the cheque sequence (hidden, technical), **Manual Numbering**, **Next Check Number** (only when manual numbering is on), and **Check Layout** with the placeholder *Default*.

### 5.3 Payment form — the cheque additions

| Element | Visibility |
|---|---|
| **Print Check** button (highlighted, shortcut `g`) | the method code is `check_printing`, the payment is not sent, and a cheque layout is available |
| **Unmark Sent** button (shortcut `l`) | the method code is `check_printing` and the payment is sent |
| **Void Check** button (shortcut `o`) | the method code is `check_printing`, the status is `in_process` and the payment is sent |
| **Amount in Words** field | the method code is `check_printing`; restricted to the technical-features group |
| **Check Number** field | the "show cheque number" flag is true |
| *Sent* ribbon | the payment is sent and its status is neither rejected nor cancelled |

### 5.4 Payment search view

A **Checks to Print** filter: the method code is `check_printing`, the status is `in_process`, and the payment has not been sent.

### 5.5 Dashboard

| Card | Element |
|---|---|
| purchase journal | title *Bills to pay*; counts and sums of drafts, waiting, late and to-review documents, each rendered in the journal's currency; the number-to-review counter and its summed amount; the irregular-sequence indicator with the hint *Irregularities due to draft, cancelled or deleted bills with a sequence number since last lock date.*; the unhashed-entries indicator; the upload zone with the image of a bill and the text *Drop and let the AI process your bills automatically.*; the *Bills Analysis* link (to the read-only accounting group) |
| bank journal | *«count» Check to print* / *«count» Checks to print*, linking to the Checks to Print action; shown only when the count is not zero |

The purchase card's sums of waiting and late amounts are **negated** relative to the sale card, so that a payable shows as a positive amount owed. Sample-data detection marks a journal as showing sample data when it has no document at all.

---

## 6. Named operations

These are the operations a client or an integration may call. Inputs are the record set plus the named arguments listed; outputs are described in words.

### 6.1 On the Journal Entry

| Operation | Input | Output and effect |
|---|---|---|
| Post | the documents; optionally a soft-scheduling flag | Posts them; answers nothing, or the autopost learning dialogue, or the confirm dialogue |
| Confirm with dialogue | the documents | Posts the ones not needing confirmation and opens the dialogue on the rest |
| Reset to draft | the documents | Moves them to draft |
| Cancel | the documents | Moves them to cancelled |
| Request cancellation | one document | Refuses unless a localisation overrides |
| Reverse (open the dialogue) | the documents | The reversal dialogue, titled *Credit Note* for invoice-like documents |
| Switch type | the documents | Flips invoice ↔ credit note and, when the total is negative, negates every product line's quantity |
| Duplicate | one document | Copies it and opens the copy |
| Register payment | the documents | The payment register dialogue |
| Force register payment | the documents | The same, skipping the posted check |
| Set reviewed | the documents | Marks the posted ones reviewed |
| Check selected | the documents named in the context | The same |
| Delete duplicates | the documents | Deletes every detected duplicate of each |
| Toggle payment block | one document | Blocks or unblocks |
| Activate currency | the documents | Reactivates their archived currencies |
| Open payments | one document | The reconciled payments |
| Open reconciliation view | one document | The reconciliation of its lines |
| Assign an outstanding line | one document, one journal item identifier | Reconciles that line with the document's lines on the same account |
| Remove a reconciled partial | one document, one partial identifier | Unreconciles |
| Download the printable document | the documents; a target, `download` by default | A navigation to the download route |
| Download every attachment | the documents | A navigation to the attachment archive route |
| Print | one document | The printable document, possibly preceded by the layout configurator |
| Send and print | the documents | The sending dialogue, single or batch |
| Preview | one document | A navigation to the portal page |
| Currency rate lookup | a company, a currency, a date | The conversion rate from the company currency to that currency at that date |
| Refresh currency rate | the documents | Resets the document rate to the expected one |
| Create documents from attachments | a list of attachments; optionally a grouping method | The created documents |
| Extend with attachments | one document, a list of file-data records, a newly-created flag | True when at least one document was decoded |

### 6.2 On the Journal

| Operation | Input | Output and effect |
|---|---|---|
| Create a document from attachments | attachment identifiers | A navigation to the created document(s) under the title *Generated Documents* |
| Create a new document | — | A navigation to a new document form with the journal and the type pre-set (`in_invoice`, or `in_refund` when the context asks for a refund) |
| Create the sample vendor bill | — | A navigation to the sample bill |
| Checks to print | — | A navigation to the Checks to Print action for this journal |
| Is the sample action available | — | True when the demonstration partner exists |

### 6.3 On the Payment

| Operation | Input | Output and effect |
|---|---|---|
| Print cheques | the payments | Either the pre-numbered dialogue or the printable document |
| Do print cheques | the payments | Resolves the layout, marks them sent, answers the printable document |
| Void cheque | one payment | Resets to draft then cancels |
| Mark as sent / Unmark as sent | the payments | Sets or clears the sent flag |
| Open paid bills | one payment | The bills it settled |

### 6.4 On the wizards

| Wizard | Operation | Effect |
|---|---|---|
| Confirm Entries | Confirm | as in `workflows.md` §9 |
| Autopost Bills | Activate auto-validation / Ask me later / Never for this vendor | sets the vendor policy |
| Add Debit Note | Create Debit Note | as in `workflows.md` §13 |
| Reversal | Reverse / Reverse and Modify | as in `workflows.md` §12 |
| Print Pre-numbered Cheques | Print | as in `workflows.md` §14.3 |

### 6.5 Server actions

| Name | Bound to | Views | Groups | Effect |
|---|---|---|---|---|
| Print Checks | Payment | list, kanban | Accountant | Calls the cheque printing operation on the selection when it is not empty |
| Confirm Entries | Journal Entry | list, kanban | Invoicing | Calls the confirm-with-dialogue operation on the selection |

---

## 7. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/invoices` and `/my/invoices/page/<page number>` | Hypertext Transfer Protocol, rendered page | logged-in user | The portal list of documents whose partner is the visitor's commercial entity. The base domain is *status is neither cancelled nor draft* and *type is one of the six invoice-like types*. Sorting: **Date** (bill date descending, the default), **Due Date**, **Reference** (number descending), **Status** (payment status). Filters: **All**, **Overdue invoices**, **Invoices** (the three outbound sale types), **Bills** (`in_invoice`, `in_refund`, `in_receipt`). A date range on the creation date is accepted. Pagination uses the portal page size; the last hundred visited documents are remembered in the session |
| `/my/invoices/<document identifier>` | Hypertext Transfer Protocol, rendered page | public, with an access token | One document's portal page. With the report type `pdf` and the download flag on a posted document, the printable document is returned instead. Access failures redirect to the portal home |
| `/my/journal/<journal identifier>/unsubscribe` | Hypertext Transfer Protocol, `GET` and `POST` | public | Unsubscribes from a journal's notifications |
| `/account/download_invoice_attachments/<attachments>` | Hypertext Transfer Protocol | logged-in user | Returns one attachment directly, or an archive of several. Every attachment must belong to a journal entry. When all of them belong to one document, the archive is named after that document; otherwise it is named `invoices.zip` |
| `/account/download_invoice_documents/<documents>/<file type>` | Hypertext Transfer Protocol | logged-in user | Returns the legal documents of the given documents in the requested type, or every type when the type is `all`. One document is returned directly; several are archived as `invoices.zip`. A failure to build a structured document on a single-document request raises *Error while creating XML:* followed by one dashed line per error |
| `/account/download_move_attachments/<documents>` | Hypertext Transfer Protocol | logged-in user | Returns an archive of everything attached to the given documents, renaming duplicates by appending ` (n)` before the extension |
| `/product/catalog/get_sections`, `/product/catalog/create_section`, `/product/catalog/resequence_sections` | remote procedure call over Hypertext Transfer Protocol, using JavaScript Object Notation | logged-in user | Support the catalogue picker used when adding lines |
| `/terms` | Hypertext Transfer Protocol, rendered page | public | The terms and conditions page referenced by documents |

The portal counters expose `invoice_count` (the three outbound sale types) and `bill_count` (the three purchase types), each computed with the same "not cancelled, not draft" base domain.

---

## 8. Printed documents

| Document | Produced by | Content |
|---|---|---|
| Invoice | the invoice report definition | The document section by section. On a purchase document it is used mainly for self-billing. Its data contains the documents, the quick-response-code payment links when the company enables them, and the report type. A separate definition adds the payment lines |
| Cheque | the layout named by the company or journal setting | The base package ships **no** layout; each country package adds one. The data handed to it is the page structure of `calculations.md` §14.6: the cheque number, whether numbering is manual, the formatted date, the payee partner and name, the company name, the currency, the payment status, the formatted amount (or `VOID` after page one), the filled amount in words (or `VOID`), the memo, the cropped flag, and the stub lines. Margins and the date caption come from the company settings |
| Debit note section | *(Debit Notes package.)* an extension of the invoice report | Adds the link back to the debited document |

---

## 9. Electronic mail templates and notifications

| Template | Used for |
|---|---|
| *Invoice: Sending* | the default sending template |
| *Credit Note: Sending* | when every selected document is a customer credit note |
| *Self-billing invoice: Sending* | when every selected document is a vendor bill in a **self-billing** journal |
| *Self-billing credit note: Sending* | when every selected document is a vendor credit note in a self-billing journal |
| *Payment: Payment Receipt* | payment acknowledgements |
| *New eInvoices Notification* | notifies the journal's notification addresses of received electronic documents |
| *Journal Notification* | the journal subscriber notification |
| the mail-gateway failure body | rendered into the bounce sent when a message reaches a journal address **without** any attachment. Its text is: *Hi,* / *Your email has been discarded. the e-mail address you have used only accepts new invoices:* / a list with *For new invoices, please ensure a PDF or electronic invoice file is attached* and *To add information to a previously sent invoice, reply to your "sent" email* / *For any other question, write to «the company electronic mail address».* / a dash separator / the company name |

### 9.1 Chatter notifications produced by this domain

| Event | Message |
|---|---|
| a document is created from attachments | *This document was created from the following attachment(s).* with those attachments |
| the created document could not be extended | *There was an error while importing the bill, you can find attached the incoming XML* |
| a decoder refuses | *Attachment «file name» not imported: «reason»* |
| a decoder raises | *Error importing attachment «descriptor»:* / *This specific error occurred during the import:* / the error text |
| automatic posting suppressed by a duplicate | *Auto-post was disabled on this invoice because a potential duplicate was detected.* |
| a document is scheduled instead of posted | *This move will be posted at the accounting date: «date»* |
| a document is reversed | on the original: *This entry has been reversed* with a link; on the copy: *This entry has been reversed from «link»* |
| a document is duplicated | *This entry has been duplicated from «link»* |
| a debit note is created | *This debit note was created from: «link»* |
| a bill is created | the creation message *Vendor Bill Created*; for a vendor credit note, *Refund Created*; for a purchase receipt, *Purchase Receipt Created* |

The creation subtype for a **sale** document is the dedicated "invoice created" subtype; purchase documents use the generic one. The payment-status subtype fires when the status becomes `paid`; the validation subtype fires only for **sale** documents.

Field tracking on a purchase document covers: the number, the vendor reference, the accounting date, the status, the type, the partner, the recipient bank account, the currency, the payment reference, the payment status, the reviewed flag, the untaxed amount, the total, the salesperson and the origin. Every tracked change is recorded as a notification message, and those messages are what the restrictive audit trail protects.

---

## 10. Display names and labels

| Where | Rule |
|---|---|
| A document with a number | the number |
| A draft never posted | *Draft Bill*, *Draft Vendor Credit Note*, *Draft Purchase Receipt* |
| The vendor column when the partner is empty | *@From: «source electronic mail address»* when the document came by mail; otherwise *#Created by: «the creator's name»* |
| A payable term line with a payment reference and a different vendor reference | `«vendor reference» - «payment reference»` |
| A payable term line with only a vendor reference | the vendor reference |
| A payable term line of a multi-instalment term | the above followed by ` installment #«index»` |
| A cheque payment's journal items | *Checks - «number»*, plus *: «memo»* when a memo exists |
| The private-share total line, after posting | *«document number» - private part* |
| The private-share tax line, after posting | *«document number» - private part (taxes)* |

---

## 11. Import and export

| Direction | Mechanism |
|---|---|
| Import of supplier documents | The business document import contract of `entities.md` §9: attachments in, decoded documents out. Concrete formats are supplied by the electronic-invoicing packages |
| Import of records | The generic record import applies. A journal imported without a type is treated as a miscellaneous journal, and its code is derived from the first five characters of its name, made unique when necessary — failing that, *Cannot generate an unused journal code. Please change the name for journal «name».* |
| Export of records | The generic export applies. The synchronization technical fields (the term key, the early-discount key and needed values, the discount allocation key and needed values) are excluded from the exportable field list, as are the payment widget structures and the needed-terms map |
| Export of documents | The three download routes of §7: one attachment, the legal documents of a set of documents in a given type, or every attachment of a set of documents as an archive |

---

## 12. External service integrations

| Integration | Contract |
|---|---|
| Structured document decoding | The decoder contract of `entities.md` §9.4. A decoder may be implemented by a local format package or by a remote scanning service; the contract is identical |
| Purchase order matching | The matching hook of `workflows.md` §6: it receives a list of references, the vendor identifier, the document total, a "came from a scan" flag and a timeout |
| Intercompany clearing | Reads the payment transactions attached to the document and the paying company's accounts; see `accounting-effects.md` §8 |

---

## 13. Client-side components specific to payables

| Component | Purpose |
|---|---|
| The bill upload list renderer | Shades the vendor reference cell for duplicates and embeds the capture guide |
| The document file uploader | Sends dropped files to the journal's document-creation operation |
| The "number with placeholder and review badge" widget | Renders the number cell, showing the placeholder for drafts and a badge for posted-but-unreviewed documents |
| The actionable-errors widget | Renders the banner dictionary, including the clickable *Remove empty lines* action |
| The many-to-many button widget used for duplicates | Shows the first duplicate as a button labelled with its total |
