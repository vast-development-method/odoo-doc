# Interfaces of the Accounts Receivable domain

Window actions and menus as user-visible navigation, views and what each shows, named remote
operations, routes, reports, mail templates and notifications, external integrations, and import and
export formats.

---

## 1. Navigation

The invoicing application's top-level menu is **Invoicing**, visible to the invoicing group and to
the read-only accounting group. The receivable branch is:

```
Invoicing
├── Dashboard                       (accounting basic group)
├── Customers
│   ├── Invoices                    → the customer documents list
│   ├── Credit Notes                → the customer credit notes list
│   ├── Payments                    → the inbound payments list
│   ├── Products                    → the sellable products list
│   └── Customers                   → the customers list
├── Vendors                         (see the payable domain)
├── Accounting                      (read-only accounting group)
├── Review                          (read-only accounting group)
├── Reporting
│   └── Management
│       └── Invoice Analysis        → the invoice analysis projection
└── Configuration                   (accounting administration group)
    ├── Settings
    ├── Accounting
    │   ├── ...
    │   └── Cash Roundings          (cash rounding group)
    └── Invoicing
        ├── Payment Terms
        ├── Incoterms               (technical users only)
        └── Product Categories
```

### 1.1 Window actions

| Action | Title | Entity | Views | Domain | Preset context |
| --- | --- | --- | --- | --- | --- |
| Customer documents | Invoices | Journal Entry | list, kanban, form, activity | the document type is customer invoice, customer credit note or sales receipt | the customer-invoice and sales-receipt filters are preselected; new records default to customer invoice |
| Customer credit notes | Credit Notes | Journal Entry | list, kanban, form, activity | the document type is customer invoice or customer credit note | the credit note filter is preselected; new records default to customer credit note; the bank-account trust indicator is shown |
| Journal entries | Journal Entries | Journal Entry | list, kanban, form, activity | none | new records default to a plain entry; the posted filter is preselected; maturity dates are hidden |
| Selected entries | Journal Entries | Journal Entry | list, kanban, form, activity | the identifiers passed by the caller | used when returning from a batch operation |
| Reverse | Reverse | Invoice Reversal Wizard | form, opened as a dialogue | — | bound to the journal entry entity as a contextual action on list and kanban views; restricted to the invoicing group |
| Create Debit Note | Create Debit Note | Debit Note Wizard | form, opened as a dialogue | — | bound the same way |
| Send | Send | Invoice Send Wizard or Invoice Batch Send Wizard | form, opened as a dialogue | — | the single wizard for one document, the batch wizard for several |
| Confirm Entries | Confirm Entries | Validate Entries Wizard | form, opened as a dialogue | — | opened when documents need an explicit confirmation before posting |
| Payment terms | Payment Terms | Payment Term | list, form | — | — |
| Cash roundings | Cash Roundings | Cash Rounding Method | list, form | — | — |
| Invoice analysis | Invoice Analysis | Invoice Analysis Line | pivot, graph, list | — | — |

Empty-state help texts shown when a list has no record:

- Customer documents: heading "Create a customer invoice", body "Create invoices, register payments
  and keep track of the discussions with your customers."
- Customer credit notes: heading "Create a credit note", body "Note that the easiest way to create a
  credit note is to do it directly from the customer invoice."
- Journal entries: heading "Create a journal entry", body "A journal entry consists of several
  journal items, each of which is either a debit or a credit transaction."

---

## 2. The customer document form

### 2.1 Header buttons

| Button | Shown when | Effect |
| --- | --- | --- |
| **Confirm** | draft | Posts the document (non-soft). On a plain entry the label is **Post**. |
| **Send** | posted sale document | Opens the send wizard, with the option to allow recipients without an e-mail address, preceded by the document-layout configurator when the layout is not yet set up. |
| **Print** | posted | Renders the printable document with the resolved layout, preceded by the layout configurator. |
| **Register Payment** | posted, unpaid or partially paid | Opens the register-payment wizard. A secondary variant exists for the cases where the primary slot is taken. |
| **Preview** | any | Opens the customer's portal view of the document in the browser. |
| **Credit Note** | an invoice | Opens the reversal wizard with the title "Credit Note". |
| **Reverse Entry** | a plain entry | Opens the reversal wizard. |
| **Cancel** | draft or posted, invoicing group | Cancels. On a plain entry the label is **Cancel Entry**. |
| **Reset to Draft** | posted or cancelled, invoicing group, and the reset-to-draft indicator is on | Resets. |
| **Lock** | invoicing group, when hashing is available | Hashes the document chain immediately. |
| **Mark as Reviewed** | posted and not yet reviewed | Sets the reviewed flag. |

The status bar shows Draft and Posted; a secured variant of the widget adds the hash state.

### 2.2 Banners

- The alerts block, rendered from the alerts structure (see
  [`business-rules.md`](business-rules.md) section 14.3), each entry with its level, its message and
  its optional action link.
- The duplicate block, listing the detected duplicates as buttons, with a **Delete duplicates**
  action.

### 2.3 Statistic buttons

| Button | Shows | Opens |
| --- | --- | --- |
| Source document | the linked business document | that document |
| Payments | the number of reconciled payments | the payments list |
| Reconciliation | — | the reconciled items view |
| Cash basis entries | — | the cash-basis entries created from this document |
| Adjusting entries | their count | the adjusting entries created from this document |
| Adjusting entry origins | their count and the origin label | the source documents of this adjusting entry |
| Debit notes | their count | the debit notes created from this document |

### 2.4 Header fields

| Field | Notes |
| --- | --- |
| Number | Shown when a number or a placeholder exists, or in quick-encoding mode. Editable only while draft; the placeholder shows the number that would be taken. |
| Customer | A rich partner selector. Followed by a button that re-applies the fiscal position to the lines when the fiscal position was changed. |
| Delivery Address | Shown to the delivery-address group when it differs. |
| Total (Tax inc.) | Quick-encoding mode only. |
| Customer Reference | The document's reference, on the "Other Info" tab for sale documents. |
| Invoice/Bill Date | With a warning when set in the future. |
| Accounting Date | |
| Payment Reference | With the placeholder "Standard communication"; readonly once the document is hashed. |
| Recipient Bank | Restricted, for a customer invoice, to bank accounts of partners that refer to the company; for a customer credit note, to the customer's own accounts. Readonly once the document has been sent and is not draft. |
| Due Date | Editable when there is no payment term. |
| Payment Terms | |
| Taxable Supply Date | Shown only when a localization enables it. |
| Delivery Date | Shown only on a sale document that has one. |
| Journal | Shown when more than one is suitable. |
| Currency, Company Currency, Currency Rate | Shown in a multi-currency setup; the rate is displayed with six decimals and is editable while draft. |

### 2.5 The Invoice Lines tab

An editable list with a drag handle for ordering and the ability to insert sections, subsections and
notes. Columns:

| Column | Notes |
| --- | --- |
| Product | |
| Label | Rendered with the section-and-note text widget, so a section spans the row. |
| Account | Restricted to accounts of the document's company or an ancestor. |
| Analytic | Shown to the analytic group; the business domain passed to the distribution models is "invoice" for customer documents, "bill" for vendor documents and "general" otherwise. |
| Quantity | |
| Unit | Shown to the units-of-measure group; restricted to the product's allowed units. |
| Price | |
| Disc.% | Optional column, hidden by default. |
| Taxes | Restricted to taxes usable on the document's side and belonging to the company tree. |
| Deductibility | Purchase side only, and only with the partial-deductibility group. |
| Subtotal | Read-only. |
| Total | Read-only, optional. |

A **Catalog** button opens the product catalogue to add lines in bulk.

Below the list: the totals block (editable per tax group so a user may correct a rounding cent), the
instalment table, the early payment discount notice, and the terms and conditions editor.

### 2.6 The Journal Items tab

Visible to the read-only accounting group. It shows every line, including the derived ones, with:
account, partner, label, analytic distribution, due date, amount in currency, currency, taxes, debit
(summed), credit (summed), discount date, discount amount and tax grids, plus a cut-off button on
income and expense lines of a posted document. It is read-only unless the document is draft.

When the payment status is the imported-balance value (`invoicing_legacy`) and the document is not a plain entry, a notice replaces
the editing affordances:

> This entry has been generated through the Invoicing app, before installing Accounting. Its balance
> has been imported separately.

### 2.7 The Other Info tab

Two groups.

**Invoice** (customer invoice and customer credit note only): Customer Reference, Salesperson,
Source Document, Recipient Bank, Payment Reference, the payment quick response code generator (only when codes are
enabled), Delivery Date.

**Accounting**: Company (multi-company only), Incoterm and its location (not on receipts), Fiscal
Position, Secured indicator (inalterability group), Payment Method (restricted to inbound methods on
a sale document and outbound methods on a purchase document), Cash Rounding Method (cash rounding
group, editable while draft), Source Email (purchase side), Auto-post and its end date, Reviewed.

### 2.8 Attachments and discussion

A document preview pane is shown for draft invoices and credit notes. The discussion thread, the
follower list and the activity scheduler are attached to the document and reload when an attachment
changes.

---

## 3. Lists, kanban and search

### 3.1 The customer document list

Columns typically shown: number, customer, document date, due date, next activity, salesperson,
tax-excluded total, tax, total, amount due, payment status (as a badge), sent indicator and status.
Rows are decorated by status and by lateness.

### 3.2 The search panel for customer documents

**Search fields**: Invoice (matching the number, the reference, the payment reference or the partner),
Number, Reference, Payment Reference, Total, Journal, Ledger, Customer (matching descendants too),
Salesperson, Period, Next Payment Date, Invoice Line, Activities of, Activity type.

**Filters**:

| Filter | Condition |
| --- | --- |
| Draft | status is draft |
| Posted | status is posted |
| Cancelled | status is cancelled |
| Not Secured | posted and not hashed (inalterability group) |
| Not Sent | the sent flag is false |
| Invoices | document type is customer invoice |
| Receipts | document type is sales receipt |
| Credit Notes | document type is customer credit note |
| To Review | not reviewed and not draft |
| To pay | not cancelled, payment status is not paid or partially paid, and the type is not a plain entry |
| In payment | posted and payment status is in payment |
| Overdue | the due date has passed and the document is still owed |
| Invoice Date, Accounting Date, Due Date | date range filters |

**Groupings**: Salesperson, Partner, Status, Payment Method, Journal, Company, Invoice Date, Due
Date, Accounting Date, Sequence Prefix (hidden, used by the numbering report).

### 3.3 The activity view

Each card shows the number, the total, the commercial entity and the status as a badge.

---

## 4. Named remote operations

These are the operations a client or an integration calls on a document. Inputs and outputs are
described in words.

| Operation | Input | Output | Effect |
| --- | --- | --- | --- |
| Post | a set of documents | nothing, or a confirmation dialogue descriptor | Posts them; may return a wizard when confirmation is needed. |
| Validate with confirmation | a set of documents | nothing, or a confirmation dialogue descriptor | Posts those that need no confirmation and opens the wizard for the others. Fails with "There are no journal items in the draft state to post." when nothing is selectable. |
| Reset to draft | a set of documents | true | Resets them. |
| Cancel | a set of documents | nothing | Cancels them. |
| Request cancellation | one document | nothing | Base platform: always fails with "You can only request a cancellation for invoice sent to the government." |
| Lock | a set of documents | nothing | Forces the hashing of the chain. |
| Mark as reviewed | a set of documents | nothing | Sets the reviewed flag on the posted ones. |
| Toggle payment block | one document | nothing | Blocks or unblocks the payment status. |
| Send and print | a set of documents | a dialogue descriptor for the send wizard | Checks the constraints first. |
| Download document files | a set of documents | a browser redirection to `/account/download_invoice_documents/<identifiers>/pdf` | |
| Download all attachments | a set of documents | a browser redirection to `/account/download_move_attachments/<identifiers>` | |
| Print | one document | a printing action descriptor | |
| Preview | one document | a browser redirection to the document's portal address | |
| Reverse | a set of documents | a dialogue descriptor for the reversal wizard, titled "Credit Note" for an invoice | |
| Create debit note | a set of documents | a dialogue descriptor for the debit note wizard | |
| Duplicate | one document | a form action on the copy | |
| Register payment | a set of documents | a dialogue descriptor for the register-payment wizard | |
| Force register payment | a set of documents | the same, bypassing the usual eligibility filter | |
| Attach an outstanding line | one document, one line identifier | the reconciliation result | Reconciles that line with the document's unreconciled lines on the same account. |
| Detach a partial reconciliation | one document, one partial reconciliation identifier | nothing | Deletes it. |
| Switch document type | one document | nothing | Swaps invoice and credit note. |
| Refresh the currency rate | one document | nothing | Recomputes the rate from the table. |
| Get a currency rate | a company, a target currency, a date | the rate | Used by the form to preview a rate change. |
| Open payments | one document | a list or form action on the reconciled payments | |
| Open reconciliation view | one document | an action showing the reconciled items | |
| Open cash basis entries | one document | a list action | |
| Open adjusting entries / their origins | one document | list or form actions | |
| View debit notes | one document | a list action filtered on this document as origin | |
| View payment transactions | one document | a list or form action on the transactions | |
| Capture payment transactions | one document | the capture result | Captures every authorised transaction linked to the document. |
| Void payment transactions | one document | nothing | Voids every authorised transaction. |
| Get the last portal transaction | one document | the transaction | Used by the portal page. |
| Add from catalog | one document | a catalogue action | |
| Update fiscal position values | a set of documents | nothing | Re-applies the fiscal position to the lines. |
| Activate the currency | one document | nothing | Un-archives the document's currency. |
| Delete duplicates | a set of documents | nothing | Deletes every detected duplicate. |

---

## 5. Routes

| Path | Method | Authentication | Purpose |
| --- | --- | --- | --- |
| `/my/invoices` and `/my/invoices/page/<number>` | web page | signed-in user | The customer's list of documents, with sorting, filtering, a date range and paging. |
| `/my/invoices/<identifier>` | web page | public (an access token authorises an anonymous visitor) | The document page. Accepts a report kind (web page, printable file or plain text), a download flag, a payment flag, a custom amount and a signed amount token. Redirects to the portal home when access is refused or the document is missing. |
| `/my/invoices/overdue` | web page, read only | public | The batch payment page for the signed-in customer's overdue invoices. Redirects to the document list when there is nothing to pay. |
| `/my/journal/<identifier>/unsubscribe` | web page, read and write | public | Removes an address from a journal's incoming-document notification list, authorised by a signed token. Responses: the confirmation page; "Invalid token" with a forbidden status; "Already unsubscribed" with a not-found status; "Deprecated link" with a gone status for the earlier link form. |
| `/account/download_invoice_attachments/<attachments>` | file download | signed-in user | Returns the named attachments, as one file or a compressed archive. |
| `/account/download_invoice_documents/<documents>/<kind>` | file download | signed-in user | Returns the documents' files of the given kind. |
| `/account/download_move_attachments/<documents>` | file download | signed-in user | Returns every attachment of the documents. |
| `/terms` | web page | public | The company's terms and conditions page, when the terms kind is the web-page one. Listed in the site map. |
| `/invoice/transaction/<identifier>` | remote call | public | Creates a draft payment transaction for one document. Requires a valid access token, otherwise: "The access token is invalid." Accepts an extra key naming the instalment being paid. |
| `/invoice/transaction/overdue` | remote call | public | Creates a draft payment transaction covering every overdue invoice of the signed-in customer. Fails with "Please log in to pay your overdue invoices" when anonymous, and with "Impossible to pay all the overdue invoices if they don't share the same currency." when the currencies differ. |
| `/payment/pay` (extended) | web page | public | The generic payment page, extended so that an invoice identifier fixes the reference, the currency, the partner and the company, and so that the landing page is the document's portal page. Invalid parameters or a bad signed token: "The provided parameters are invalid." |

The document's own portal address is built from the portal mixin: it is the document page path with
the document identifier, plus an access token when the document is shared with an anonymous visitor.

---

## 6. The printable customer document, section by section

The default layout is the one with payments (report `account.report_invoice_with_payments`); a
variant without the payment history exists (report `account.report_invoice`). Both wrap the document body in the company's external layout (letterhead, footer with the
company's tax registration identifier — replaced by the fiscal position's foreign tax registration
identifier when one is set).

### 6.1 Addresses

- When a delivery address exists and differs from the customer: an information block on the left
  titled **Shipping Address** with that address (shown only to the delivery-address group), and the
  customer's address on the right.
- When a delivery address exists and equals the customer: the customer's address on the right only.
- When there is no delivery address: the customer's address on the right only.

Under the customer's address, when the customer has a tax registration identifier: the country's own
label for it (or the words "Tax ID" when the country defines none), a colon, and the identifier. For
a Moroccan customer with a company register number, the label "ICE" and that number.

### 6.2 Title

One line combining the document kind and the number. The kind depends on the type, the status and
whether the *pro forma* mode is on:

| Type and status | Title |
| --- | --- |
| Customer invoice, posted | Invoice |
| Customer invoice, draft | Draft Invoice |
| Customer invoice, cancelled | Cancelled Invoice |
| Customer credit note, posted | Credit Note |
| Customer credit note, draft | Draft Credit Note |
| Customer credit note, cancelled | Cancelled Credit Note |
| Vendor credit note | Vendor Credit Note, or Self Billing Credit Note on a self-billing journal |
| Vendor bill | Vendor Bill, or Self Billing on a self-billing journal |

In *pro forma* mode each of these is prefixed with "Proforma" (for example "Proforma Invoice",
"Draft Proforma Credit Note"). The number is appended when it exists and is not `/`.

### 6.3 The information strip

Shown when at least one of its cells has content. Cells, in order:

| Cell | Label | Shown when |
| --- | --- | --- |
| Document date | "Invoice Date" for a customer invoice, "Credit Note Date" for a customer credit note, "Receipt Date" for a sales receipt, "Date" otherwise | the document date is set |
| Due date | Due Date | the due date is set, the type is customer invoice and the status is posted |
| Taxable supply date | Taxable Supply | a localization enables it |
| Delivery date | Delivery Date | it is set |
| Origin | Source | the origin is set |
| Customer code | Customer Code | the customer has an internal reference |
| Reference | Reference | the document reference is set |
| Incoterm | Incoterm | an incoterm is set; the location is printed underneath when set |

### 6.4 The line table

Columns: **Description**, **Quantity**, **Unit Price**, **Disc.%** (only when at least one printed
line carries a discount), **Taxes** (only when at least one invoice line carries a tax), **Amount**.

Rows are the printable lines in document order. Sections and subsections render as spanning
headings; notes render as spanning text. A section marked "hide composition" suppresses its child
lines; a section marked "hide prices" suppresses their price columns. Only product lines carry
figures.

### 6.5 The totals block

- The untaxed amount.
- One row per tax group, labelled with the group and its base.
- The gross total.
- When cash rounding applies, the rounding row is part of the same structure (as a net row for the
  "add a rounding line" strategy, as part of the tax row for the "biggest tax" strategy).
- When payments exist and the payment status is not the imported-balance one: one row per non-exchange payment,
  labelled "Paid on " or "Reversed on " followed by its date, and then a bold **Amount Due** row.
- When the company asks for it: "Total amount in words:" followed by the spelled-out total.
- When the company asks for it and the currencies differ: a second totals block in the company
  currency.

### 6.6 The closing block

In order:

1. The fiscal position's note, when it has one.
2. The taxes' legal notes, when any.
3. The payment term's description on the invoice, when it has one.
4. When the payment term asks to show instalment dates and the document qualifies:
   - the early payment discount line, when the document is still eligible at its own date: the
     discounted amount, then " due if paid before ", then the deadline;
   - when there is more than one instalment, one line per instalment: the position, " - Installment
     of ", the amount, " due on ", the date.
5. For a customer invoice or a vendor credit note with a payment reference: "Payment Communication: "
   followed by the reference in bold, and, when a recipient bank account is set, a second line
   "on this account: " followed by the account.
6. The payment quick response code, when codes are enabled and the residual is non-zero, with the
   caption "Scan this QR Code with\nyour banking application" — reproduced exactly as the layout
   emits it. The code is rendered silently: if it
   cannot be produced, no code and no caption appear.
7. The portal-link quick response code, when the link-code setting is on and the residual is
   non-zero, wrapped in a link to the portal payment address, with the captions "PAY IN A FLASH!" and
   "Scan the QR code\nor click to pay online" — again the exact emitted text.
8. The terms and conditions.

---

## 7. Mail templates and notifications

### 7.1 Templates

| Template | Chosen when | Subject |
| --- | --- | --- |
| Invoice: Sending | the default | *the company name* ` Invoice (Ref ` *the document number, or* `n/a` `)` |
| Credit Note: Sending | every document being sent is a customer credit note | *the company name* ` Credit Note (Ref …)` |
| Self-billing invoice: Sending | every document is a vendor bill on a self-billing journal | *the company name* ` Self-billing invoice (Ref …)` |
| Self-billing credit note: Sending | every document is a vendor credit note on a self-billing journal | *the company name* ` Self-billing credit note (Ref …)` |
| Journal Notification | sent to a journal's invoice subscribers after a document is sent | *the company name* ` - New invoice in ` *the journal name, or* `Invoices` ` journal` |
| Payment: Payment Receipt | sent to acknowledge a payment | *the company name* ` Payment Receipt (Ref …)` |
| New eInvoices Notification | sent when electronic invoices are received | New Electronic Invoices Received |

Every send posts a comment message on the document's thread, using the notification layout that
carries the responsible person's signature, with the document type name as the model description and
the produced files as attachments. The message's attachments are re-parented onto the message so they
do not duplicate on the document.

### 7.2 Live notifications pushed to the sender

| Situation | Title | Body | Button |
| --- | --- | --- | --- |
| batch send queued | Sending invoices | Invoices are being sent in the background. | — |
| batch send succeeded | Invoices sent | Invoices sent successfully. | **Open**, labelled "Sent invoices", opening the documents |
| batch send failed | Invoices in error | One or more invoices couldn't be processed. | **Open**, labelled "Invoices in error", opening the documents |

### 7.3 Messages written on the document thread

| Event | Body |
| --- | --- |
| soft posting of a future-dated document | This move will be posted at the accounting date: *the date* |
| reversal | This entry has been **reversed** *(a link to the reverse)* |
| debit note creation | This debit note was created from: *(a link to the source)* |
| a line was updated on a document that has been posted before | Journal Item *(a link labelled with the line identifier)* updated, with the tracked values |
| a line was deleted on such a document | Journal Item *(a link)* deleted, with the tracked values |
| the auto-post job failed on this document | The move could not be posted for the following reason: *the error* |
| a send failed | the formatted error: its title, then a bulleted list of the individual errors |

Tracked fields on the document itself (each change writes a tracking row): number, reference,
accounting date, status, document type, partner, recipient bank account, salesperson, currency,
untaxed amount, total, payment status, reviewed, origin, source e-mail address. Tracked fields on a
line: account, label, due date.

---

## 8. External integrations

| Integration | Where it plugs in |
| --- | --- |
| Payment providers | The portal payment form, the transaction routes, and the automatic creation and reconciliation of a payment once a transaction succeeds. |
| Electronic document exchange | Extra sending channels and extra electronic deliveries offered by the send wizard; the electronic format chosen per partner; the pre-render and post-render service calls in the generation flow. |
| Bank payment codes | The quick response code payload generators; two are shipped (see section 9). |
| Journal subscribers | Addresses that receive a copy of every document sent from a journal, with an unsubscribe route. |
| Document layout configurator | Opened before the first send or print so the company letterhead is set up. |

---

## 9. The payment quick response code

### 9.1 How a code is produced

1. The document asks its recipient bank account for a code, passing: the amount (the residual), the
   free communication (the document number), the structured communication (the payment reference),
   the currency, and the debtor (the commercial entity).
2. The bank account lists the available generators, ordered by their declared priority, and takes
   either the generator the document names or, when it names none, the first one that can produce a
   code.
3. For each candidate, two checks run: an *eligibility* check (is this generator applicable at all,
   given the country, the currency and the account kind) and a *data* check (is the data complete).
   A candidate that fails either is skipped. When the caller asks for errors rather than silence, the
   first failure is raised as:
   > The following error prevented '*the generator name*' QR-code to be generated though it was
   > detected as eligible: *the error message*
4. The chosen generator returns the payload and the rendering parameters. The code is rendered as a
   square image of one hundred and twenty-eight by one hundred and twenty-eight, with no quiet zone
   and with the human-readable text enabled, and is embedded in the document as an inline image.

Without a bank account, no code is produced. A currency must always be supplied, otherwise:

> Currency must always be provided in order to generate a QR-code

### 9.2 The Single Euro Payments Area credit transfer generator (stored value `sct_qr`, priority 20)

**Eligibility.** Refused, with one message per failing condition joined by a line break, when:

- the currency is not the euro:
  > Can't generate a SEPA QR Code with the *currency name* currency.
- the account is not an international bank account number:
  > Can't generate a SEPA QR code if the account type isn't IBAN.
- the sanitised account number's first two characters are not the country code of a Single Euro
  Payments Area country that uses international bank account numbers (the area's country list minus
  the territories that share another country's account prefix):
  > Can't generate a SEPA QR code with a non SEPA iban.

**Data check.** The account must have either an account holder name or a partner name:

> The account receiving the payment must have an account holder name or partner name set.

**Payload.** Twelve lines, in this exact order, separated by line breaks:

| # | Field | Value |
| --- | --- | --- |
| 1 | Service tag | the literal `BCD` |
| 2 | Version | the literal `002` |
| 3 | Character set | the literal `1` |
| 4 | Identification code | the literal `SCT` |
| 5 | Beneficiary bank identifier code | the bank's identifier code, or empty |
| 6 | Beneficiary name | the account holder name, else the partner name, truncated to seventy-one characters |
| 7 | Beneficiary account number | the sanitised account number |
| 8 | Currency and amount | the currency code immediately followed by the amount rounded to the currency and rendered with the currency's number of decimals |
| 9 | Purpose | empty |
| 10 | Structured remittance information | the structured communication when it is a valid structured reference, after sanitising it; empty otherwise |
| 11 | Unstructured remittance information | empty when a structured one was used; otherwise the free communication truncated to one hundred and forty-one characters |
| 12 | Beneficiary-to-originator information | empty |

Sanitising a structured reference removes every whitespace character, and additionally removes the
plus signs, asterisks and slashes of the Belgian grouped form (three digits, slash, four digits,
slash, five digits, optionally wrapped in three plus signs or three asterisks).

### 9.3 The merchant-presented generator (stored value `emv_qr`, priority 30)

This is the merchant-presented standard used across Asia and elsewhere. The base platform supplies
the framing; each country variant supplies the merchant account information, the additional data
field and the merchant category code.

**Eligibility.** Without a bank account:
> A bank account is required for EMV QR Code generation.

With a bank account but no country variant that matches its country:
> No EMV QR Code is available for the country of the account *the account number*.

**Data check.** In order, the first failure is reported:

> Missing Merchant Account Information.
> Missing Merchant City.
> Missing Proxy Type.
> Missing Proxy Value.

**Encoding.** Every element is written as a two-digit identifier, a two-digit length and the value;
an element whose value is empty or absent is omitted entirely.

| Identifier | Element | Value |
| --- | --- | --- |
| 00 | Payload format indicator | the literal `01` |
| 01 | Point of initiation | the literal `12`, meaning a dynamic code |
| variable | Merchant account information | supplied by the country variant, under the identifier the variant chooses |
| 52 | Merchant category code | supplied by the country variant; the base value is `0000` |
| 53 | Transaction currency | the three-digit numeric currency code, looked up in the shipped currency table |
| 54 | Transaction amount | omitted when the amount rounds to zero in the currency; otherwise the amount, written without decimals when it is a whole number |
| 58 | Country code | the account's country code |
| 59 | Merchant name | the partner name with accents removed and truncated to twenty-five characters, or the literal `NA` |
| 60 | Merchant city | the partner city with accents removed and truncated to fifteen characters, or empty |
| 62 | Additional data field | supplied by the country variant from the communication, and only when the account asks to include the reference |

Accent removal also maps the Latin letter d with stroke, in both cases, to a plain `d` or `D`.

The communication used for the additional data field is the structured communication when there is
one, else the free communication, with accents removed and every character outside the set
[space, letters, digits, underscore, at sign, full stop, backslash, slash, number sign, ampersand,
plus sign, hyphen] deleted.

**Checksum.** After all the elements, the literal `6304` is appended, then a four-character
upper-case hexadecimal checksum computed over the whole resulting text encoded as eight-bit
characters. The checksum is a sixteen-bit cyclic redundancy check with the generator polynomial
0x1021 and the initial value 0xFFFF:

```formula
crc = 0xFFFF
for each byte b of the text:
    crc = crc XOR ( b shifted left by 8 )
    repeat 8 times:
        if crc AND 0x8000 is non-zero:
            crc = ( crc shifted left by 1 ) XOR 0x1021
        else:
            crc = crc shifted left by 1
    crc = crc AND 0xFFFF
checksum = crc written as four upper-case hexadecimal digits
```

**The shipped numeric currency codes** (currency code → three-digit numeric code): Argentine peso
032, Australian dollar 036, Bahraini dinar 048, Cambodian riel 116, Canadian dollar 124, Sri Lankan
rupee 144, Chinese yuan 156, Croatian kuna 191, Czech koruna 203, Danish krone 208, Hong Kong dollar
344, Hungarian forint 348, Icelandic króna 352, Indian rupee 356, Israeli new shekel 376, Japanese
yen 392, South Korean won 410, Kuwaiti dinar 414, Malaysian ringgit 458, Mauritian rupee 480, Mexican
peso 484, Nepalese rupee 524, New Zealand dollar 554, Norwegian krone 578, Qatari riyal 634, Russian
rouble 643, Saudi riyal 682, Singapore dollar 702, Vietnamese dong 704, South African rand 710,
Swedish krona 752, Swiss franc 756, Thai baht 764, United Arab Emirates dirham 784, Tunisian dinar
788, pound sterling 826, United States dollar 840, new Taiwan dollar 901, Serbian dinar 941, Romanian
leu 946, Turkish lira 949, West African franc 952, French Pacific franc 953, Bulgarian lev 975, euro
978, Ukrainian hryvnia 980, Polish złoty 985, Brazilian real 986.

A currency not in this table cannot be encoded by this generator.

### 9.4 Bank account fields that feed the codes

| Field | Effect |
| --- | --- |
| Include Reference | Whether the additional data field carrying the communication is written. |
| Proxy Type and Proxy Value | The payment alias (for example a mobile number or a national identifier) that a country variant turns into the merchant account information. |
| Account holder name, partner name, partner city, country | Used by both generators as shown above. |

### 9.5 The portal-link code

Independent of the payment code. It encodes the document's portal payment address, produced by the
payment-link wizard for that document, rendered as a one-hundred-and-twenty-eight-pixel square image
with the quiet zone enabled, and wrapped in a hyperlink to the same address.

---

## 10. Import and export

- **Export.** Every non-technical field of the document and of its lines is exportable. Fields marked
  as not exportable are excluded: the needed terms, the payment widgets, the totals structure, the
  payment term details, the quick-encoding values and the synchronisation keys.
- **Import.** Documents and lines can be imported through the generic import mechanism. An imported
  line is flagged as such, which suppresses the automatic defaulting of its unit price, its taxes and
  the archived-account check.
- **Structured electronic documents.** Producing and consuming them belongs to
  [`../electronic-invoicing-and-document-exchange/interfaces.md`](../electronic-invoicing-and-document-exchange/interfaces.md);
  this domain contributes the source data and the extra channels shown in the send wizard.
- **Attachments.** Files dropped on a draft document are attached and, on the purchase side, may be
  decoded into lines; on the receivable side they are stored as plain attachments.

---

## 11. The customer portal pages

| Template | Content |
| --- | --- |
| Portal home tile | A card for "Invoices / Bills" with the count, and a separate card for overdue invoices with its own count. |
| My Invoices and Payments | The list page: search bar with the sortings and the filters of section 4 of [`workflows.md`](workflows.md), one row per document with its number, date, due date, amount, amount due and payment status badge, plus a paging control and an "overdue" callout with a pay-all button. |
| Invoice/Bill page | The sidebar with the download and print buttons; the rendered document; the instalment table; the early payment discount notice with its message; the payment history; and, when payable, the payment form. |
| Overdue payments page | The aggregate amount, the batch communication and the payment form. |
| Invoice payment / Invoice paid | The payment dialogue and the already-paid message. |
| Journal e-mail notification settings | The unsubscribe confirmation page, with its error variants. |
| My details | Extended with the customer's preferred invoice sending method and electronic invoice format. |

---

## 12. File names of the generated documents

| File | Name |
| --- | --- |
| The printable document | The layout's print-name expression evaluated on the document; when the layout defines none, the document number. Every forward slash is replaced by an underscore, then the extension is appended. For the shipped layouts the print-name expression is the document's display name, so a posted customer invoice numbered `INV/2026/00001` gives `INV_2026_00001.pdf`. |
| A dynamic report attached by the mail template | The same rule applied to that report's own print-name expression; when the report defines none, the report name in lower case, an underscore, the document number and the extension. |
| The *pro forma* document | The document's display name with every space and every forward slash replaced by an underscore, then `_proforma.pdf`. |
| A multi-document download | A compressed archive named after the first document with the archive extension. |
| A detached file after a reset to draft | The original name, then a space, then a parenthetical reading "detached by *the user name* on *today's date*", then the original extension. For example `INV_2026_00001 (detached by Jane Doe on 03/20/2026).pdf`. |

---

## 13. Data the interface reads that is not a stored field

Several structures exist only to feed the interface. An implementation must produce them with the
same shape because the client and the printed document depend on them.

| Structure | Shape | Consumer |
| --- | --- | --- |
| Totals structure (`tax_totals`) | the net subtotal, one entry per tax group with its base and its tax amount, the gross total, and optional rows for cash rounding and the early payment discount; it also carries a flag saying whether a second block in the company currency must be shown | the form's totals block and the printed document |
| Outstanding credits or debits (`invoice_outstanding_credits_debits_widget`) | a title, a flag saying these are outstanding items, the document identifier, and a list of entries each with the counterpart's label, the amount expressed in the document currency, the currency, the line identifier, the document identifier, the date, the payment identifier and the reference | the one-click attach block |
| Payments applied (`invoice_payments_widget`) | a title, a flag saying these are applied items, and a list of entries each with the label, the journal name, the company name when it differs, the reconciled amount, the currency, the date, the partial reconciliation identifier, the payment identifier and its method name, the document identifier, whether it is a refund, the reference, whether it is an exchange difference, and the formatted amounts in both currencies | the applied-payments block and the printed payment history |
| Payment term details (`payment_term_details`) | a list, sorted by maturity date, of entries each with the formatted date and the customer-facing amount | the printed instalment table |
| Alerts (`alerts`) | a map from a stable key to an entry with a level, a message and, optionally, an action label plus either an action descriptor or a direct operation to call | the banner area |
| Next payment values | the payment status, the instalment state, the next amount to pay, the next reference, the amount paid, the amount due, the next due date, the document due date, the list of unreconciled instalments, whether only one remains, and, in the early-discount case, the discount amount in both currencies, the deadline, the days left, the discount line and the message | the portal page and the payment link wizard |
