# Configuration of the Accounts Receivable domain

Settings, system parameters, sequences and numbering formats, default records, security groups, the
access rights matrix, record rules and scheduled jobs.

---

## 1. Company settings that affect customer documents

All of these live on the company record and are surfaced in the accounting settings page. A setting
described as *company-dependent* stores one value per company on a shared record rather than on the
company itself.

### 1.1 Invoicing

| Setting (storage name) | Type | Default | Effect |
| --- | --- | --- | --- |
| Display the payment quick response code on invoices (`qr_code`) | boolean | false | Enables the payment quick response code on the printed document for customer invoices, sales receipts, vendor bills and purchase receipts. |
| Display the link quick response code (`link_qr_code`) | boolean | false | Enables the second code, which encodes the portal payment address instead of bank payment data. |
| Total amount of invoice in letters (`display_invoice_amount_total_words`) | boolean | false | Prints the gross total spelled out in words. |
| Taxes in company currency (`display_invoice_tax_company_currency`) | boolean | false | Adds a second totals block expressed in the company currency on a foreign-currency document. |
| Default Terms and Conditions (`invoice_terms`) | rich text, translatable | empty | The text copied into a new sale document's terms when the terms feature is on and the terms kind is the plain one. |
| Default Terms and Conditions as a Web page (`invoice_terms_html`) | rich text, translatable | a shipped skeleton | The page served at the terms address when the terms kind is the web-page one. |
| Terms kind (`terms_type`) | selection | `plain` | `plain` (Add a Note) copies the text onto the document; `html` (Add a Link to a Web Page) writes "Terms & Conditions: " followed by the company's base address and `/terms`. |
| Sales Credit Limit (`account_use_credit_limit`) | boolean | false | Turns on the credit limit warning on draft customer invoices. |
| Default Credit Limit (`account_default_credit_limit`) | monetary | zero | The company-wide fallback credit limit for partners that carry no specific limit. Stored as the company-dependent fallback of the partner's credit limit field. |
| Default incoterm (`incoterm_id`) | many-to-one | empty | Proposed on outgoing documents. |
| Customer Invoices Discounts Account (`account_discount_expense_allocation_id`) | many-to-one to Account | empty | When set, every discounted product line of a sale document gets a pair of discount allocation lines moving the discounted part to this account. Restricted to income and expense accounts. |
| Vendor Bills Discounts Account (`account_discount_income_allocation_id`) | many-to-one to Account | empty | The same for purchase documents. |
| Early Discount Loss (`account_journal_early_pay_discount_loss_account_id`) | many-to-one to Account | empty | The write-off account used when an early payment discount is granted on an inbound document. Restricted to income and expense accounts. |
| Early Discount Gain (`account_journal_early_pay_discount_gain_account_id`) | many-to-one to Account | empty | The same for an outbound document. |
| Quick encoding (`quick_edit_mode`) | selection | empty | `out_invoices` (Customer Invoices), `in_invoices` (Vendor Bills), `out_and_in_invoices` (Customer Invoices and Vendor Bills). Enables the "type the total, let the platform make the line" mode and relaxes the deletion rule on numbered documents. |
| Storno accounting (`account_storno`) | boolean | false | Makes credit notes book as negative amounts on the original side instead of on the opposite side. |
| Restrictive Audit Trail (`restrictive_audit_trail`) | boolean | false | Forbids deleting a document that has been posted at least once; it must be cancelled instead. |
| Tax Calculation Rounding Method (`tax_calculation_rounding_method`) | selection | `round_globally` | `round_globally` (Round per Tax) or `round_per_line` (Round per Line). Governs how the tax engine rounds; see [`../taxes/calculations.md`](../taxes/calculations.md). |
| Exchange Gain or Loss Journal (`currency_exchange_journal_id`) | many-to-one to Journal | set by the chart template | Where exchange difference entries are booked. |
| Gain Exchange Rate Account (`income_currency_exchange_account_id`) | many-to-one to Account | set by the chart template | Restricted to income accounts. |
| Loss Exchange Rate Account (`expense_currency_exchange_account_id`) | many-to-one to Account | set by the chart template | |
| Journal Suspense Account (`account_journal_suspense_account_id`) | many-to-one to Account | set by the chart template | Fallback for the automatic balancing line. |

### 1.2 Lock dates

Each of these blocks changes to documents dated on or before it; they belong to the ledger domain
and are listed here only because posting a customer invoice consults them:

the fiscal year lock date, the sale lock date, the purchase lock date, the tax lock date and the
hard lock date. Their per-user variants take lock date exceptions into account. See
[`../general-ledger/configuration.md`](../general-ledger/configuration.md).

### 1.3 The fiscal year for numbering

The company's fiscal year end day and fiscal year end month decide the year part of a starting
document number: a calendar fiscal year produces a four-digit year, a staggered one produces a
two-digit range. See [`calculations.md`](calculations.md) section 8.3.

---

## 2. Journal settings that affect customer documents

On a journal of the sale kind:

| Setting (storage name) | Type | Default | Effect |
| --- | --- | --- | --- |
| Communication Type (`invoice_reference_type`) | selection, required | `invoice` | `partner` (Based on Customer) or `invoice` (Based on Invoice). Together with the next field it chooses the payment reference generator. |
| Communication Standard (`invoice_reference_model`) | selection, required | see below | `system` (Full Reference, for example `INV/2024/00001`), `euro` (European, for example `RF83INV202400001`), `number` (Numbers only, for example `202400001`). The default is the first value whose name begins with the company country's code in lower case, and `system` when none matches. |
| Dedicated Credit Note Sequence (`refund_sequence`) | boolean | true for sale and purchase journals | Numbers credit notes in their own series, prefixed with `R` at the start. |
| Dedicated Payment Sequence (`payment_sequence`) | boolean | computed per journal kind | Numbers payments in their own series, prefixed with `P`. |
| Dedicated Debit Note Sequence (`debit_sequence`) | boolean | true for sale and purchase journals | Numbers debit notes in their own series, prefixed with `D`. It is not a plain stored flag: it is **computed and stored with a manual override**, and its rule is re-evaluated whenever the journal's kind changes, setting it to true for a sale or purchase journal and to false for any other kind. A manual choice therefore survives every change except a change of kind, which silently overwrites it — changing a journal from the sale kind to the bank kind clears a manually ticked box. The field is hidden on the journal form unless the kind is sale or purchase. |
| Invoice report (`invoice_template_pdf_report_id`) | many-to-one to Report | empty | The printable layout proposed for documents of this journal, when the partner does not impose one. |
| Number pattern (`sequence_override_regex`) | text | empty | A pattern that overrides all five built-in numbering grammars for this journal. It may name the parts: a first separator, the year, a second separator, the month, a third separator, the incrementing number and a suffix. Only a member of the accountant group may type a number that does not match it; doing so clears the pattern. |
| Secure Posted Entries with Hash (`restrict_mode_hash_table`) | boolean | false | Hashes entries on posting, which makes them unresettable. |
| Default account (`default_account_id`) | many-to-one to Account | set by the chart template | Last-resort account for a line, and first choice for the automatic balancing line. |
| Currency (`currency_id`) | many-to-one to Currency | empty | When set, documents of this journal default to this currency, and the journal is preferred when a matching currency is chosen. |

---

## 3. System parameters

| Key | Default | Meaning |
| --- | --- | --- |
| `account.use_invoice_terms` | not set (off) | When on, a new sale document copies the company's default terms into its Terms and Conditions. |
| `account.pdf_generation_batch` | `80` | How many documents are rendered in one batch by the sending flow. |
| `account.product_name_similarity_threshold` | `0.9` | Threshold used when matching a product by name during document import. |
| `account_payment.enable_portal_payment` | shipped enabled | Master switch for paying an invoice from the customer portal. |

---

## 4. Sequences and numbering formats

### 4.1 Document numbers

Customer document numbers are **not** produced by a counter record. They are derived from the highest
existing number in the journal's scope, as specified in [`calculations.md`](calculations.md)
section 8. The shipped shapes are:

| Journal kind | Fiscal year | Starting number | Example first number |
| --- | --- | --- | --- |
| sale, bank, cash, credit | calendar | `<journal code>/<four-digit year>/00000` | `INV/2026/00001` |
| sale, bank, cash, credit | staggered | `<journal code>/<two-digit start>-<two-digit end>/0000` | `INV/25-26/0001` |
| any other | calendar | `<journal code>/<four-digit year>/<two-digit month>/0000` | `MISC/2026/03/0001` |
| any other, self-billing | calendar | `<journal code><partner identifier padded to five digits>/<year>/<month>/0000` | `BILL00042/2026/03/0001` |

with the prefixes `R` for a credit note (dedicated credit note sequence), `P` for a payment
(dedicated payment sequence) and `D` for a debit note (dedicated debit note sequence), applied in
that order of description — a credit note prefix and a payment prefix cannot both apply to the same
document.

The five reset rules the numbering grammar can detect are: never, yearly, monthly, by fiscal year
range, and by fiscal year range and month.

### 4.2 The payment sequence record

One shipped counter record exists in this area:

| Name | Code | Prefix | Padding | Company |
| --- | --- | --- | --- | --- |
| Payment | `account.payment` | `PAY` | 5 | shared by every company |

It numbers payment records themselves, not their journal entries.

### 4.3 The payment reference

Not a sequence: it is derived from the document number, the partner reference or the partner
identifier, according to the journal's reference model and type. See
[`calculations.md`](calculations.md) section 9.

---

## 5. Default records shipped

### 5.1 Reports, templates, page geometry and actions

| Record | Purpose |
| --- | --- |
| Printable layout **Invoice with payments** (report `account.report_invoice_with_payments`) | The default customer document layout, including the payment history block. Marked as an invoice layout. Visible to the invoicing and read-only accounting groups. Bound to the journal entry entity as a printing action. |
| Printable layout **Invoice without payments** (report `account.report_invoice`) | The same document without the payment history block. Bound the same way, limited to documents that are not plain entries. |
| Printable layout **Original Bills** (report `account.report_original_vendor_bill`) | Purchase side: returns the originally received file. |
| Mail template **Invoice: Sending** | Default template for a customer invoice. Subject: *the company name* ` Invoice (Ref ` *the document number or* `n/a` `)`. |
| Mail template **Credit Note: Sending** | Used when every document being sent is a customer credit note. Subject: *the company name* ` Credit Note (Ref …)`. |
| Mail template **Self-billing invoice: Sending** | Used when every document is a self-billed vendor bill. |
| Mail template **Self-billing credit note: Sending** | Used when every document is a self-billed vendor credit note. |
| Mail template **Journal Notification** | Sent to a journal's invoice subscribers. Subject: *the company name* ` - New invoice in ` *the journal name or* `Invoices` ` journal`. |
| Mail template **Payment: Payment Receipt** | Sent to acknowledge a payment. |
| Paper format **A4 - statement** | The default page geometry. |
| Server action **Share** | Bound to the journal entry entity, offered on the **form** view only, and available on customer documents as well as on plain entries. Choosing it calls the document's share operation, which produces a shareable address of the document for an external reader. Its name is exactly "Share". |

### 5.2 The ten shipped payment terms

These ten payment terms are shipped unconditionally by the accounting capability itself, not by a
country chart template; a chart template may ship further country-specific terms **on top** of them.
Every one of them carries a note, which is the text printed on the document when the term asks to be
displayed. All the lines below use the percent kind; the balance rule of
[`entities.md`](entities.md) section 4 applies to the last line of each term.

| Name | Note | Lines (kind, amount, delay) | Early payment discount |
| --- | --- | --- | --- |
| Immediate Payment | Payment terms: Immediate Payment | one line: percent, 100, `days_after` 0 days | none |
| 15 Days | Payment terms: 15 Days | one line: percent, 100, `days_after` 15 days | none |
| 21 Days | Payment terms: 21 Days | one line: percent, 100, `days_after` 21 days | none |
| 30 Days | Payment terms: 30 Days | one line: percent, 100, `days_after` 30 days | none |
| 45 Days | Payment terms: 45 Days | one line: percent, 100, `days_after` 45 days | none |
| End of Following Month | Payment terms: End of Following Month | one line: percent, 100, `days_after_end_of_next_month` 0 days | none |
| 10 Days after End of Next Month | Payment terms: 10 Days after End of Next Month | one line: percent, 100, `days_after_end_of_next_month` 10 days | none |
| 30% Now, Balance 60 Days | Payment terms: 30% Now, Balance 60 Days | two lines: percent, 30, `days_after` 0 days; then percent, 70, `days_after` 60 days | none |
| 2/7 Net 30 | Payment terms: 30 Days, 2% Early Payment Discount under 7 days | one line: percent, 100, `days_after` 30 days | yes: displayed on the invoice, discount of 2 per cent, granted within 7 days |
| 90 days, on the 10th | Payment terms: 90 days, on the 10th | one line: percent, 100, `days_end_of_month_on_the` 90 days, day of the following month 10 | none |

The delay kinds are the stored values of the payment term line's delay field, defined with their
arithmetic in [`entities.md`](entities.md) section 4.3 and
[`calculations.md`](calculations.md) section 2. Only the term named 2/7 Net 30 sets the
early-payment-discount fields; the other nine leave them off.

### 5.3 The eleven shipped international commercial terms

Eleven records are shipped, each with a three-letter code and a name in capital letters. The document
fields that point at them are in [`entities.md`](entities.md) section 1.11.

| Code | Name |
| --- | --- |
| `EXW` | EX WORKS |
| `FCA` | FREE CARRIER |
| `FAS` | FREE ALONGSIDE SHIP |
| `FOB` | FREE ON BOARD |
| `CFR` | COST AND FREIGHT |
| `CIF` | COST, INSURANCE AND FREIGHT |
| `CPT` | CARRIAGE PAID TO |
| `CIP` | CARRIAGE AND INSURANCE PAID TO |
| `DPU` | DELIVERED AT PLACE UNLOADED |
| `DAP` | DELIVERED AT PLACE |
| `DDP` | DELIVERED DUTY PAID |

### 5.4 The shipped decimal precision record

One decimal precision record is shipped by the accounting capability in this area:

| Name | Digits | What is evaluated at this precision |
| --- | --- | --- |
| Payment Terms | 6 | The percentage carried by a payment term line, and the check that the percentages of a term's percent lines add up to one hundred. |

Six digits means that a percentage such as 33.333333 is kept exactly and that the sum check accepts
33.333333 + 33.333333 + 33.333334 = 100.000000 while refusing a sum that differs in the sixth
decimal. The check itself and its refusal message are in
[`business-rules.md`](business-rules.md) section 10.1.

### 5.5 What a country chart template adds

Chart templates additionally ship, per country: the receivable and payable accounts, the default
sale taxes, the cash discount accounts, the exchange accounts and often extra payment terms and cash
rounding methods. See [`../fiscal-localizations/README.md`](../fiscal-localizations/README.md).

---

## 6. Security groups

| Group | Label | Implies | What it grants in this domain |
| --- | --- | --- | --- |
| `account.group_account_invoice` | Invoicing | the internal user group | Create, read, update and delete customer documents and their lines; post them; read the payment widgets and the credit figures; use the reversal, debit note, send and validate wizards; manage cash rounding methods. |
| `account.group_account_readonly` | Show Accounting Features - Readonly | the internal user group | Read customer documents, their lines, the payment widgets, the credit figures, the invoice analysis projection and the cash rounding methods. No write. |
| `account.group_account_basic` | Basic | the invoicing group | Additional accounting features (the dashboard, basic bank reconciliation). |
| `account.group_account_user` | Show Full Accounting Features | the basic group and the read-only group | The accountant: everything except the most advanced configuration. |
| `account.group_account_manager` | Administrator | the invoicing group | Full access including configuration; may type a number that breaks the journal's pattern; may delete a numbered document that is not last in its chain; manages payment terms. Granted by default to the system user and the administrator user. |
| `account.group_account_secured` | Show Inalterability Features | — | Shows the hash-related fields and actions. |
| `account.group_cash_rounding` | Allow the cash rounding management | — | Shows the cash rounding menu and the cash rounding field on documents. Granted by the "Cash Rounding" setting. |
| `account.group_delivery_invoice_address` | Delivery Address | — | Shows the delivery address on the form and on the printed document. Granted by the "Customer Addresses" setting. |
| `account.group_partial_purchase_deductibility` | Partial Purchase Deductibility | — | Purchase side; granted automatically to a user who posts a bill with a partial deductibility. |
| `account.group_validate_bank_account` | Validate bank account | implied by the system administration group | Allows marking a bank account as trusted for outgoing payments. |
| `base.group_portal` | Portal | — | Read-only access to one's own documents through the portal. |

The group graph, when only invoicing is installed, is: the invoicing group implies the internal user
group and is implied by the administrator group; the read-only group is a separate branch implied by
the full-accounting group.

---

## 7. Access rights matrix

Rows are entities, columns are the four operations. A blank means the group has no access at all.

| Entity | Group | Create | Read | Update | Delete |
| --- | --- | --- | --- | --- | --- |
| Journal Entry (`account.move`) | Invoicing | yes | yes | yes | yes |
| Journal Entry | Show Accounting Features - Readonly | no | yes | no | no |
| Journal Entry | Administrator | no | yes | no | no |
| Journal Entry | Portal | no | yes | no | no |
| Journal Item (`account.move.line`) | Invoicing | yes | yes | yes | yes |
| Journal Item | Show Accounting Features - Readonly | no | yes | no | no |
| Journal Item | Administrator | no | yes | no | no |
| Journal Item | Portal | no | yes | no | no |
| Payment Term (`account.payment.term`) | internal user | no | yes | no | no |
| Payment Term | Administrator | yes | yes | yes | yes |
| Payment Term | Portal | no | yes | no | no |
| Payment Term Line (`account.payment.term.line`) | internal user | no | yes | no | no |
| Payment Term Line | Administrator | yes | yes | yes | yes |
| Cash Rounding Method (`account.cash.rounding`) | Invoicing | yes | yes | yes | yes |
| Cash Rounding Method | Show Accounting Features - Readonly | no | yes | no | no |
| Invoice Analysis (`account.invoice.report`) | Invoicing | no | yes | no | no |
| Invoice Analysis | Show Accounting Features - Readonly | no | yes | no | no |
| Invoice Analysis | Administrator | no | yes | no | no |
| Invoice Reversal Wizard (`account.move.reversal`) | Invoicing | yes | yes | yes | no |
| Debit Note Wizard (`account.debit.note`) | Invoicing | yes | yes | yes | no |
| Validate Entries Wizard (`validate.account.move`) | Invoicing | yes | yes | yes | no |
| Invoice Send Wizard (`account.move.send.wizard`) | Invoicing | yes | yes | yes | yes |
| Invoice Batch Send Wizard (`account.move.send.batch.wizard`) | Invoicing | yes | yes | yes | yes |
| Payment Transaction (`payment.transaction`) | Invoicing | yes | yes | yes | no |
| Payment Link Wizard (`payment.link.wizard`) | Invoicing | yes | yes | yes | no |
| Payment Refund Wizard (`payment.refund.wizard`) | Invoicing | yes | yes | yes | no |

The Debit Note Wizard row is the *only* access entry that entity has: no other group, not even the
administrator group, is granted anything on it directly, so a user outside the invoicing group cannot
open the create-debit-note dialogue at all.

The last three rows are granted by the online-payment capability. They give a billing user the right
to create, read and update a payment transaction, a payment link and a payment refund, and never the
right to delete one: a transaction is part of the audit record of a settlement and is cancelled or
voided rather than removed.

Note that the Administrator group's *direct* entries on the journal entry and journal item entities
are read-only; its write access comes from the invoicing group it implies.

Beyond the matrix, the platform enforces the extra checks of
[`business-rules.md`](business-rules.md) section 16: posting requires the invoicing group; reviewing
requires the reviewer right; several fields are readable only by the invoicing or read-only
accounting groups.

---

## 8. Record rules

| Rule | Entity | Applies to | Domain |
| --- | --- | --- | --- |
| Account Entry | Journal Entry | everyone | the company is among the user's allowed companies |
| Entry lines | Journal Item | everyone | the company is among the user's allowed companies |
| All Journal Entries | Journal Entry | the invoicing group | unrestricted (this rule *adds* access rather than restricting it, so an invoicing user sees every document of an allowed company) |
| All Journal Items | Journal Item | the invoicing group | unrestricted |
| Portal Personal Account Invoices | Journal Entry | the portal group | the status is neither cancelled nor draft, the document type is one of customer invoice, customer credit note, vendor bill or vendor credit note, and the partner is the user's commercial entity or one of its children |
| Portal Invoice Lines | Journal Item | the portal group | the document's status is neither cancelled nor draft, the document type is one of the same four, and the document's partner is the user's commercial entity or one of its children |
| Account payment term company rule | Payment Term | everyone | the term has no company, or its company is an ancestor of one of the user's allowed companies |
| Invoice Analysis multi-company | Invoice Analysis | everyone | the company is among the user's allowed companies |
| Readonly Invoice Send and Print (single) | Invoice Send Wizard | the invoicing group | unrestricted; like the two rules above it, it *adds* access rather than restricting it, so an invoicing user may work on any single-document send dialogue |
| Readonly Invoice Send and Print (batch) | Invoice Batch Send Wizard | the invoicing group | unrestricted, with the same effect for the batch dialogue |
| Access every token | Payment Token | the invoicing group | unrestricted. This rule exists to *reset* the restricting rule that otherwise limits a stored payment token to the customer who created it, so a billing user sees every stored token of every customer. The token entity itself belongs to [`../payment-providers/`](../payment-providers/README.md). |

Note the portal rules exclude sales receipts and purchase receipts: a receipt is not visible in the
portal even though the portal controller's list domain includes the receipt types. The record rule
wins, so receipts never appear.

---

## 9. Scheduled jobs

| Job | Frequency | What it does |
| --- | --- | --- |
| **Account: Post draft entries with auto_post enabled and accounting date up to today** | daily, first run at 02:00 the next day | Selects up to one hundred draft documents whose auto-post value is not "No" and whose accounting date is on or before today, and posts them. It first attempts the whole batch; on any failure it rolls back and retries one at a time, locking each document and re-checking that it still matches, writing on the failing document's thread: *"The move could not be posted for the following reason: "* followed by the error. It reports the remaining count so the scheduler can pace itself. |
| **Send invoices automatically** | daily, run as the system user | Selects up to ten posted documents that carry sending data, ordered by accounting date, then document date, then sequence position, then identifier, locks them, and runs the generate-and-send flow in the "from job" mode (errors are written on the thread instead of raised, and a retryable error keeps the sending data for the next run). Archiving this job disables batch sending; the send wizard warns about it. |

Other jobs that touch receivable documents but belong elsewhere: the payment-provider
post-processing job, the follow-up job, and the electronic document exchange jobs.

---

## 10. Configuration checklist for a working receivable setup

1. A chart of accounts is installed for the company, providing at least one receivable account, one
   income account and the tax accounts.
2. At least one journal of the sale kind exists, with a default account and a code.
3. The journal's communication type and standard are chosen if a structured payment reference is
   wanted.
4. The customer has a receivable account (directly, or through the company's own partner record, or
   through any active receivable account of the company).
5. The customer has a customer payment term if instalments are wanted; otherwise the whole total
   falls due on the document's due date.
6. When early payment discounts are used: the payment term carries a single one-hundred-percent line,
   a strictly positive percentage, a strictly positive number of days, and the company has the two
   cash discount write-off accounts.
7. When cash rounding is used: the cash rounding feature group is granted, a method exists, and for
   the "add a rounding line" strategy the profit and loss accounts are set **for each company**.
8. When discount allocation is used: the company's customer-invoice discount account is set.
9. For foreign-currency invoicing: the currency is active, rates are loaded, and the exchange journal
   and the two exchange accounts are set.
10. For sending: the sending background job is active for batch sending; the mail templates exist;
    the company has a document layout configured.
11. For portal payment: the portal payment parameter is on, at least one payment provider is enabled
    for the company and the currency, and the customers have portal access.
