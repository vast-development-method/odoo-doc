# Interfaces

Window actions and menus as user-visible navigation, views and what each shows, named remote operations with their inputs and outputs, routes, printable documents, message templates, external service integration points, and import and export formats.

---

## 1. Navigation

### 1.1 Menus

| Path | Opens | Visible to |
|---|---|---|
| Invoicing → Dashboard | the journal dashboard, showing one card per journal | basic accounting user |
| Invoicing → Customers → Payments | the *Customer Payments* list | billing user |
| Invoicing → Vendors → Payments | the *Vendor Payments* list | billing user |
| Invoicing → Configuration → Accounting → Journals | the journal list, from which a liquidity journal's payment method lines, suspense account and bank account are configured | accounting administrator |
| Invoicing → Configuration → Settings | the settings screen carrying every setting of `configuration.md`, section 1 | system administrator |

Bank statements, bank transactions and reconciliation models are not reached from the main menu in the base capability: they are reached from a liquidity journal's dashboard card (section 1.3) and from the internal navigation path of the reconciliation-model action.

### 1.2 Window actions

| Action | Name | Entity | Views | Domain | Preset context |
|---|---|---|---|---|---|
| all payments | Payments | Payment | list, kanban, form, graph, activity | none | none |
| customer payments | Customer Payments | Payment | list, kanban, form, graph, activity | none | default direction `inbound`, default counterparty kind `customer`, the inbound filter on, default journal types `bank` and `cash`, trust shown in account names. Navigation path `customer-payments`. |
| vendor payments | Vendor Payments | Payment | list, kanban, form, graph, activity | none | default direction `outbound`, default counterparty kind `supplier`, the outbound filter on, default journal types `bank` and `cash`, trust shown. Navigation path `vendor-payments`. |
| internal transfers | Internal Transfers | Payment | list, kanban, form, graph | empty | default direction `outbound`, the transfers filter on, trust shown |
| bank statements | Bank Statements | Bank Statement | list, pivot, graph, form | the journal is of type `bank` | journal type `bank` |
| credit statements | Credit Statements | Bank Statement | list, pivot, graph, form | the journal is of type `credit` | journal type `credit` |
| cash registers | Cash Registers | Bank Statement | list, pivot, graph, form | the journal is of type `cash` | journal type `cash` |
| reconciliation models | Reconciliation Models | Reconciliation Model | list, form | none | Navigation path `reconciliation-models`. |
| supplier bank accounts | Bank Accounts | Bank Account | list, form | none | none |
| bank setup | Setup Bank Account | Bank Setup Wizard | one dedicated form, opened as a medium dialogue | — | — |
| credit-card setup | Setup Credit Card Account | Bank Setup Wizard | one dedicated form, opened as a medium dialogue | — | journal type `credit` |

Empty-state guidance shown by the payment actions: "Register a payment" / "Payments are used to register liquidity movements. You can process those payments by your own means or by using installed facilities."

Empty-state guidance shown by the bank-statement action: "Register a bank statement" / a bank statement is a summary of all financial transactions occurring over a given period of time on a bank account; the user should receive it periodically from the bank; a statement line can be reconciled directly with the related sale or purchase invoices.

Empty-state guidance shown by the reconciliation-model action: "Create a new reconciliation model" / those can be used to quickly create journal items when reconciling a bank statement or an account.

### 1.3 The liquidity journal dashboard card

One card per journal of type `bank`, `cash` or `credit`. Its figures are defined in `calculations.md`, section 17.

**Header:** the journal name (a link that opens the journal's default action) and, in a multi-company setting, the company name.

**Primary buttons:**

| Button | Shown when | Effect |
|---|---|---|
| Bank Setup | the journal is of type `bank`, has no bank account, and its bank-feed source is `undefined` or an unconfigured online synchronisation | opens the bank setup dialogue with this journal preselected |
| Transactions | always | opens the journal's default action: the bank-statement list for a `bank` journal, the credit-statement list for a `credit` journal, the cash-register list for a `cash` journal |

**Figures on the right:**

| Figure | Shown when | Links to |
|---|---|---|
| Balance | the journal has at least one statement line or one direct bank payment, and the company is accessible | — |
| Last Statement | the journal has at least one statement, the balance differs from the last reported balance, and the last statement is visible given the fiscal-year lock date | — |
| Payments | there is at least one unmatched posted payment | the all-payments list, filtered on unmatched, this journal, and posted |
| Misc. Operations | there is at least one journal item on the default account that belongs to no transaction and no payment | the journal items list, filtered on that account, excluding items belonging to a transaction, between the last statement date (or the fiscal-year lock date) and today |
| Unhashed entries | the journal has unhashed entries | the entry list filtered accordingly |
| Irregular Sequences | the journal has gaps in its numbering | the entry list grouped by sequence prefix with the irregular filter on |
| Invalid Statement(s) | the journal has at least one invalid statement | the bank-statement list |

Two further figures are computed and available to the card but are only rendered by the extended accounting capability: the number of transactions to reconcile and the number and amount of transactions to check.

**The card's menu:**

| Section | Entry | Effect | Visible to |
|---|---|---|---|
| View | Statements (bank), Statements (credit), Cash Registers (cash) | the corresponding list, filtered on this journal | everyone who sees the card |
| View | Payments | the all-payments list for this journal | everyone |
| New | Customer Payment | a new inbound Payment form on this journal | basic accounting user |
| New | Vendor Payment | a new outbound Payment form on this journal | basic accounting user |
| Reconciliation | Models | the reconciliation-model list | basic accounting user |
| Settings | the colour picker, the *show on dashboard* switch, and a link to the journal's configuration | — | accounting administrator for the last two |

**The graph:** a filled line of the balance over the last thirty days, keyed "Bank: Balance", "Cash: Balance" or "Credit Card: Balance". When the journal has no transaction, sample data is shown and the card is marked as sample.

**Drop zone:** dropping a file on the card imports transactions, when an import capability is installed. Its caption is "Drop to import transactions" and it shows a bank symbol for `bank` and `credit` journals.

---

## 2. Views

### 2.1 Payment — list

Not editable in place. A draft row is shown in the information colour; a cancelled row is muted. A header button *Confirm* runs the confirmation on the selected rows.

Columns: date (read-only once the state is cancelled or in process), number, journal, company (hidden by default, multi-company only), payment method line (shown without the journal suffix), counterparty (labelled "Customer" in the customer list, "Vendor" in the vendor list, "Partner" in the generic list), signed amount in the payment currency (hidden by default when multi-currency is off, shown when it is on), payment currency (hidden by default), activities (hidden by default), signed amount in the company currency with a column total, and the state as a badge — information colour for draft, warning for in process, success for paid.

### 2.2 Payment — search

| Filter | Condition |
|---|---|
| Customer Payments | the counterparty kind is `customer` |
| Vendor Payments | the counterparty kind is `supplier` |
| Draft | the state is `draft` |
| In Process | the state is `in_process` |
| Sent | the sent flag is set |
| Not Sent | the sent flag is not set |
| No Bank Matching | the matched flag is false |
| Reconciled | the reconciled flag is true |
| Payment Date | a date filter on the payment date |

Groupings: counterparty, journal, payment method line, state, payment date, currency (multi-currency only), company (multi-company only). Four hidden activity filters are also present.

### 2.3 Payment — form

**Header buttons**, each with its visibility rule:

| Button | Shown when |
|---|---|
| Confirm | the state is `draft` |
| Validate | the state is `in_process` **and** there is no Journal Entry |
| Reject | the state is `in_process` **and** the payment has been marked as sent |
| Reset to Draft | the state is not `draft`; billing users only |
| Request Cancel | the state is `in_process`, there is an entry, and the entry needs a cancellation request; billing users only |
| Mark as Sent | the state is `in_process`, the payment is not yet sent, and the method code is `manual` |
| Unmark as Sent | the state is `in_process`, the payment is sent, and the method code is `manual` |
| Cancel | the record exists and either the state is `draft`, or the state is `in_process` and the payment has been marked as sent |

The state is shown as a status bar with the visible steps draft, in process and paid.

**Warning banner:** when there are duplicate payments and the state is draft — "This payment has the same partner, amount and date as " followed by buttons linking to each duplicate.

**Statistic buttons:**

| Button | Shown when | Opens |
|---|---|---|
| *n* Invoice / *n* Credit Note | at least one reconciled sale document | the paid invoices, named "Paid Invoices" |
| *n* Bill | at least one reconciled purchase document | the paid bills, named "Paid Bills" |
| *n* Transaction | at least one reconciled bank transaction | those transactions, named "Matched Transactions" |
| Journal Entry | the Payment has an entry; accountants and accounting readers only | that entry |

**Title:** the word "Draft" while the state is draft, otherwise the payment number, read-only.

**Left column:** the direction as a horizontal radio, the counterparty (labelled Customer or Vendor according to the counterparty kind, with company creation preset and no quick creation), the amount with its currency (the currency only in a multi-currency setting), the date and the memo. All of them are read-only once the state leaves draft, except the memo.

**Right column:** the journal (restricted to the available journals), the payment method line (mandatory, no creation, no opening, shown without the journal suffix), and the recipient bank account under three labels chosen by direction and counterparty kind — "Customer Bank Account", "Vendor Bank Account" or "Company Bank Account" — each shown only when the method uses a bank account and the direction matches, and required when the method needs one. The account list shows the trust marker.

**Below:** the quick response code image when one could be generated, centred.

**Footer:** an empty notebook reserved for localisation pages, a document preview pane and the message thread.

### 2.4 Payment — kanban and graph

A mobile kanban card without creation, and a graph view.

### 2.5 Payment Method Line — list and kanban

A non-editable list with two columns, the payment method name and the journal; and a mobile kanban showing the display name.

### 2.6 Bank Statement — list

Not creatable from the list. A row is shown in the danger colour when the statement has a journal and is not complete, or is not valid; a row with no journal is muted.

Columns: reference, date, journal, company (multi-company only), starting balance, reported ending balance. Three hidden columns carry the computed balance, the currency and the two flags so the row decoration can use them.

### 2.7 Bank Statement — search

Fields searched: the reference (labelled "Statement") and the date. Filters: *Empty* (no transactions), *Invalid* (not valid **or** not complete), a date filter, and a journal field restricted to liquidity journals. Groupings: journal and date.

### 2.8 Bank Statement — pivot and graph

Both show the date as the row axis and the starting balance and the computed ending balance as measures.

### 2.9 Reconciliation Model — list

Columns: the drag handle for the sequence, the name, the trigger, and the journals as tags (hidden by default).

### 2.10 Reconciliation Model — form

**Header:** *Set Manual* (hidden when the trigger is already manual), *Automate* (hidden when the trigger is already automated), and the trigger as a status bar.

**Statistic button:** *Journal Entries*, which lists every entry containing a journal item this model created, with the empty-state text "This reconciliation model has created no entry so far".

**Title:** the name, with the placeholder "Model Name" and the example "e.g. Bank Fees".

**Left group:** the journals (placeholder "All bank & cash journals", tags, no creation), the counterparties (placeholder "All partners", tags, no quick creation), the amount condition with its minimum and maximum — the minimum hidden when the condition is empty or *lower*, the maximum hidden when the condition is empty or *greater*, the word "and" between them only for *between*, and each required when shown — and the label condition with its parameter (hidden and not required when no condition is chosen, placeholder `BRT *([\d,\.]+)`).

**Right group:** the next activity type, placeholder "Nothing to do".

**Notebook, page "Counterpart Items":** an editable list of the model's lines with the drag handle, the counterparty, the account (required when no counterparty is given), the amount mode, the amount text, the taxes (hidden by default), the analytic distribution (only for users with analytic accounting) and the label.

**Footer:** the message thread.

### 2.11 Reconciliation Model — search

Searched field: the name. Filters: *Automated*, *With tax* (the model has at least one line with taxes), *Archived*. Groupings: *Journals Availability* and *Automation*.

### 2.12 Bank Account — form additions

The base form gains:

- a banner listing the partners that share the same account number, as buttons, labelled "Partners with same bank";
- the two phishing warnings and the money-transfer service name;
- the *Send Money* trust switch, locked as described in `business-rules.md`, section 9.2;
- the message thread, so every change to a tracked field is logged.

### 2.13 Bank Account — search additions

| Filter | Condition |
|---|---|
| Trusted | the trust flag is set |
| Untrusted | the trust flag is not set |
| To validate | the trust flag is not set **and** at least one entry references the account |
| Customers | the holder's customer rank is above zero |
| Vendors | the holder's vendor rank is above zero |
| Phishing risk: High | the money-transfer warning is set |
| Phishing risk: Medium | the country-mismatch warning is set |
| Created On | a date filter on the creation timestamp |

Groupings: creation date and creator.

### 2.14 Register-payment screen

**Banners**, in this order:

| Banner | Shown when | Text |
|---|---|---|
| discount, informative | the write-off section is hidden, that is, the screen is in discount mode | "Early Payment Discount of <the difference> has been applied." |
| untrusted accounts, warning | at least one payment would be skipped | "<n> out of <m> payments will be skipped due to **untrusted bank accounts**." where the bold phrase opens those accounts |
| missing accounts, warning | at least one counterparty has no account | "Payments related to partners with no bank account specified will be skipped. **View Partner(s)**" |
| duplicates, warning | duplicates were found | "This payment has the same partner, amount and date as " followed by buttons linking to each |
| actionable errors | any actionable error is present | rendered from the structured error list, currently the payments-in-progress danger message |

**Left group:** the journal (mandatory, no creation, no opening), the payment method line (mandatory, no creation, no opening, shown without the journal suffix), the recipient bank account (hidden unless the method uses one, the screen is editable, and grouping is on or unavailable; read-only for an inbound payment; required when the method needs one; placeholder "Account Number"; trust shown), the grouping switch (hidden when grouping is unavailable), and the difference block.

**The difference block** is shown only when the difference must be shown. It contains the difference amount, the handling as a radio (hidden when the screen was opened on a draft document), and, when the handling is *mark as fully paid* and the screen is not in discount mode, the difference account labelled "Post Difference In" (mandatory, no creation) and the write-off label (mandatory; both the label field and its caption hidden when the chosen account is an exchange-difference account).

**Right group:** the amount with its currency (both read-only when the screen is not editable or grouping is available but off; the currency only in a multi-currency setting), the switch sentence rendered as rich text when there is one, the payment date, and the memo (hidden when the screen is not editable or grouping is available but off).

**Below:** the quick response code image when one could be generated.

**Footer:** *Create Payment* when exactly one payment will be created, *Create Payments* otherwise, and *Discard*.

### 2.15 Bank setup screen

A single group with the account number (example `BE15001559627230`), the bank (example "Bank of America"), the bank identifier code labelled "Bank Identifier Code" (example `GEBABEBB`, absent from the credit-card variant) and the journal (no creation, placeholder "Leave empty to create new", hidden when there is no unlinked journal of the wanted type, restricted to journals of that type with no bank account). Footer: *Create* and *Cancel*.

---

## 3. Named operations

Every operation below is invoked on a set of records and returns either nothing or a navigation instruction.

### 3.1 On a Payment

| Operation | Input | Output | Effect |
|---|---|---|---|
| confirm | a set of Payments | nothing | `workflows.md`, section 6 |
| validate | a set of Payments | nothing | sets the state to `paid` |
| reject | a set of Payments | nothing | sets the state to `rejected` |
| cancel | a set of Payments | nothing | sets the state to `canceled`, deletes draft entries, cancels posted ones |
| reset to draft | a set of Payments | nothing | sets the state to `draft` and resets the entries |
| request cancellation | one Payment | whatever the entry's own operation returns | delegates to the entry |
| mark as sent / unmark as sent | a set of Payments | nothing | sets or clears the sent flag |
| open the settled invoices | one Payment | a navigation instruction named "Paid Invoices" on the reconciled sale documents, with creation disabled | — |
| open the settled bills | one Payment | a navigation instruction named "Paid Bills": a form when there is exactly one, a list otherwise | — |
| open the matched transactions | one Payment | a navigation instruction named "Matched Transactions": a form when there is exactly one, a list otherwise | — |
| open the journal entry | one Payment | a form on the Payment's entry, named "Journal Entry" | — |
| open the business document | one Payment | a form on the Payment itself, named "Payment" | used by generic drill-downs |
| open the refund screen | one Payment | a dialogue on the refund screen named "Refund" | online-payment capability |
| view the refunds | one Payment | a form on the single refund, or a list of the refunds | online-payment capability |

A server operation named **Post Payments** is bound to the Payment list and kanban views for billing users; it runs the confirmation on the selected records.

Two window actions are bound to the Payment for sending the receipt: *Send receipt by email* for one record and *Send receipts by email* for several, both opening a message composer.

### 3.2 On a Bank Transaction

| Operation | Input | Output | Effect |
|---|---|---|---|
| undo reconciliation | a set of transactions | nothing | `workflows.md`, section 12 |

### 3.3 On a Journal Item

| Operation | Input | Output | Effect |
|---|---|---|---|
| register payment | a set of journal items, optionally a context | a dialogue on the register-payment screen named "Register Payment" | `workflows.md`, sections 2 and 3 |
| register payment on selected items | a set of journal items | the same, with grouping switched on by default | — |
| reconcile | a set of journal items | nothing | matches them together as one plan |
| remove the reconciliation | a set of journal items | nothing | deletes every matching they take part in |
| unreconcile matched entries | the identifiers in the context | nothing | extends the selection to the connected matching group and removes every matching |
| open the reconciliation view | one journal item | a navigation instruction on the matched items | — |

### 3.4 On a Reconciliation Model

| Operation | Input | Output | Effect |
|---|---|---|---|
| set manual | a set of models | nothing | sets the trigger to `manual` |
| set automated | a set of models | nothing | sets the trigger to `auto_reconcile` |
| journal-entry statistics | one model | a navigation instruction on the entries containing at least one journal item created by the model, with the empty-state text "This reconciliation model has created no entry so far" | — |

### 3.5 On a Bank Account

| Operation | Input | Output | Effect |
|---|---|---|---|
| archive | one account | a reload instruction | archives the account and forces the screen to refresh |
| open the business document | a set of accounts | a navigation instruction on them | — |
| build the code rendering address | amount, free communication, structured communication, currency, payer, optionally a forced generator, optionally a silent-errors flag | a rendering address, or nothing | `calculations.md`, section 16.1 |
| build the embedded code | the same inputs | an inline image address, or nothing | the same, rendered and encoded |
| check an international number | a text | true or false | the validation of `calculations.md`, section 14.3, without raising |
| basic bank account number | one account | the number without its first four characters | fails when the account is not of the international type |

### 3.6 On a Journal

| Operation | Input | Output | Effect |
|---|---|---|---|
| open the default action | one journal | the statement or cash-register list for the journal | — |
| open an action by name | one journal, an action name in the context | that action with this journal preselected | — |
| open the payments | one journal, optionally a direction and a display mode | the matching payment list or a new payment form | — |
| create a customer payment | one journal | a new inbound payment form | — |
| create a vendor payment | one journal | a new outbound payment form | — |
| create a bank statement | one journal | a statement form with this journal preselected | — |
| open the bank difference | one journal | the journal items on the default account, excluding those of transactions, between the last statement date (or the fiscal-year lock date) and today | — |
| open the invalid statements | one journal | the bank statement list | — |
| configure the bank journal | one journal | the bank setup dialogue with this journal preselected | — |
| list the transactions to check | one journal | the posted, unchecked transactions of the journal | — |

### 3.7 On the register-payment screen

| Operation | Input | Output |
|---|---|---|
| create the payments | the screen | a form on the created Payment when there is one, a list otherwise; or nothing at all when the creation was made for sibling companies the user may not act for |
| open the untrusted accounts | the screen | a form on the single untrusted account, or a list of them |
| open the counterparties missing an account | the screen | a navigation instruction on those counterparties, using a dedicated list view when there is more than one |

### 3.8 On the bank setup screen

| Operation | Input | Output |
|---|---|---|
| validate | the screen | a reload instruction |

The journal-linking step runs as an inverse of both the journal field and the journal-name field, so it is triggered by saving.

### 3.9 On the company

| Operation | Input | Output |
|---|---|---|
| open the bank-account setup | the company | the bank setup dialogue, titled "Setup Bank Account", medium size |
| open the credit-card setup | the company | the same screen with the journal type set to `credit`, titled "Setup Credit Card Account" |
| next grouped-payment communication | the company | the next value of the company's group-payment sequence |

---

## 4. Routes

This domain adds no route of its own. It uses one route of the platform:

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/report/barcode/` | read | as the calling page | Renders a barcode or a quick response code from query arguments. The payable code address built by a Bank Account is this path followed by the generation parameters: the barcode type (`QR`), the quiet-zone width (0), the width (128), the height (128), the human-readable flag (1) and the payload text. |

---

## 5. Reports and printable documents

### 5.1 Payment Receipt

| Property | Value |
|---|---|
| Name | Payment Receipt |
| Entity | Payment |
| Kind | portable-document format, rendered from a page template |
| Binding | offered from the Payment's print menu |
| Language | the counterparty's language, or the company partner's language |

Content, in order:

1. A heading: the receipt title (by default "Payment Receipt") followed by ": " and the payment number.
2. "Payment Date: " and the date, when there is one.
3. "Customer: " or "Vendor: " according to the counterparty kind, and the counterparty; and, when the value dictionary allows it and a method line is set, "Payment Method: " and the method line's name.
4. "Payment Amount: " and the amount formatted in the payment currency; and "Memo: " and the memo, when there is one.
5. A table of the settled documents, shown when the value dictionary allows it. Its header is: Invoice Date, Invoice Number, Reference, optionally Amount In Currency, Amount. The optional column appears only when at least one settled document has a currency different from the currency of one of its matched payments.
   For each settled document that is not a plain entry:
   - one row with the document's invoice date, its number, its reference and its total;
   - one row per matching, with the payment entry's date, the payment entry's number, the payment's memo, the matched amount in the payment currency in the optional column (only when the two currencies differ), and the matched amount in the document's currency;
   - one row in bold reading "Due Amount for <the document number>" and the document's residual.

The value dictionary is produced by a hook returning two flags, both true by default: whether to display the invoice table and whether to display the payment method.

### 5.2 Statement

| Property | Value |
|---|---|
| Name | Statement |
| Entity | Bank Statement |
| Kind | portable-document format, using the dedicated page format "A4 - statement" |
| Binding | offered from the statement's print menu |

Content, in order: a centred heading reading "Bank Statement" for a `bank` journal and "Cash Statement" otherwise; a line with the journal's display name, the journal's bank account display name when there is one and the journal's sequence prefix; on the right, the statement's reference and its date; then a framed table opening with the starting balance and the date of the statement's earliest transaction; then one row per transaction; then the ending balance. The page format is A4 portrait with a top margin of 52, a bottom margin of 32, no side margins, no header line, a header spacing of 52 and 90 dots per inch.

---

## 6. Message templates and notifications

| Template | Entity | Used by |
|---|---|---|
| the payment-receipt template | Payment | the *send receipt by electronic mail* operation, for one or several Payments |

Notifications produced by this domain, all posted on the **counterparty's** message thread:

| Event | Text |
|---|---|
| A Bank Account is created | Bank Account <a link to the account> created |
| A tracked field of a Bank Account changes | Bank Account <a link to the account> updated, with the tracked values; also posted on the previous holder's thread when the holder changed |
| A Bank Account is archived | Bank Account <a link to the account> with number <the number> archived |

A Payment carries its own message thread, so every change to a tracked field (date, state, recipient bank account, payment method, direction, counterparty kind, memo, payment reference, counterparty, signed amount) is logged on it. A Reconciliation Model carries a message thread too, logging its trigger, amount condition, bounds, label condition and label parameter.

Attachments sent with a Payment's electronic mail that are not yet attached to anything are re-pointed at the Payment when it has no main attachment.

---

## 7. External service integration points

The base capability defines the contracts; the services themselves are supplied by capability packages.

| Integration point | Contract |
|---|---|
| Bank feed | A journal's *bank feed* selection is extensible. Each value names a way transactions arrive. The base value is `undefined`. A package adds a value and a way of creating Bank Transactions and, usually, Bank Statements. |
| Statement import | A file dropped on the dashboard card, or offered at the end of the bank setup screen, is turned into transactions. The fields a format may fill are: date, label, amount, foreign currency and foreign amount, the counterparty's account number, the counterparty's name, the transaction type code and the raw payload. |
| Quick response code generators | A generator registers a code, a display name and a sequence, and supplies: an eligibility check given the account, the payer and the currency; a data check given the amount, the currency, the payer and the two communications; a payload builder; and a rendering-parameters builder. Two are shipped (`calculations.md`, section 16). |
| Merchant-presented code schemes | A country-specific package supplies the merchant account information with its own tag, the merchant category code, the additional-data field builder, the accepted proxy kinds and whether the setting is displayed. |
| Online payment transactions | A Payment may carry a transaction and a saved token. Confirming a Payment that has a token creates the transaction, charges it, post-processes it, and then either posts the Payment (transaction done), leaves it (pending or authorised) or cancels it (anything else). |
| Payment method registry | A package adds entries to the method information registry, each with a multiplicity mode, the journal types it accepts, optional currencies and an optional country. |
| Withholding lines | A hook lets a package add journal items to a Payment's entry before the counterpart is computed; when it does, any supplied write-off lines are dropped. |
| Payment method codes needing or using a bank account | Two hooks return the lists of codes that *use* and that *need* a recipient bank account; the base lists are `manual` and empty respectively. |
| Excluded payment method codes | A hook lets a package hide methods for a specific Payment. |
| Partial-matching restriction | A hook lets a package forbid partial matching of a transaction against specific journal items; the online-payment capability uses it for transfer-file and provider payments. |
| Receipt title and receipt values | Two hooks let a package change the printed receipt's title and hide the invoice table or the payment method. |

---

## 8. Import and export formats

### 8.1 Import

| Entity | Note |
|---|---|
| Bank Transaction | The natural import target for bank data. The delegation to the Journal Entry means that importing a transaction also creates and posts its entry, so an import must supply at least the journal, the date, the label and the amount. |
| Bank Statement | Imported with its reference (the file name or the provider reference), its starting balance and its reported ending balance; its transactions are linked through their statement field. |
| Bank Account | Imported with the account number, the holder and optionally the bank; the number is sanitized and, when valid, prettified; the trust flag is always forced off on import. |
| Journal Item with a matching placeholder | A journal item may be imported with a matching number of the form `I` followed by any text. Those items are not matched at import. Once every entry of a placeholder group is posted, the deferred-matching operation reconciles the group per account, making the account reconcilable if needed, with exchange differences and cash-basis entries disabled. |

### 8.2 Export

All entities of this domain export through the platform's generic export. Two computed fields are worth naming because they are the ones a reconciliation audit needs: the matching number of a journal item, and the three amounts of a Partial Reconciliation.

### 8.3 Payload formats produced

| Format | Produced by | Shape |
|---|---|---|
| Single Euro Payments Area credit transfer code | a Bank Account | twelve newline-separated fields (`calculations.md`, section 16.2) |
| Merchant-presented code | a Bank Account | tag-length-value fields concatenated, terminated by a four-hexadecimal-digit checksum (`calculations.md`, section 16.3) |
