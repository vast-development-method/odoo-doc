# General Ledger — Interfaces

Menus and window actions as user-visible navigation, views and what each shows, named remote operations with their inputs and outputs, routes, reports and printable documents, notifications, external integrations, and import and export formats.

Contents:

1. [Navigation](#1-navigation)
2. [Window actions](#2-window-actions)
3. [Views](#3-views)
4. [The accounting dashboard](#4-the-accounting-dashboard)
5. [Named remote operations](#5-named-remote-operations)
6. [Routes](#6-routes)
7. [Reports and printable documents](#7-reports-and-printable-documents)
8. [Notifications and messages](#8-notifications-and-messages)
9. [Import and export](#9-import-and-export)
10. [External integrations](#10-external-integrations)
11. [The dashboard data contract](#11-the-dashboard-data-contract)
12. [Client-side widgets fed by this domain](#12-client-side-widgets-fed-by-this-domain)

---

## 1. Navigation

The accounting application is reached from the main menu entry "Invoicing". Its submenus, in display order, are:

| Order | Menu | Visible to | Contents |
|---|---|---|---|
| 1 | Dashboard | Basic | The journal cards |
| 2 | Customers | Invoicing | Invoices, Credit Notes, Payments, Products, Customers |
| 3 | Vendors | Invoicing | Bills, Refunds, Payments, Products, Vendors |
| 4 | Accounting | Show Accounting Features - Readonly | Transactions (Journal Entries, Analytic Items) and Closing |
| 7 | Review | Show Accounting Features - Readonly | Control (Journal Items) and Logs |
| 20 | Reporting | Show Accounting Features - Readonly, or Invoicing | Statement Reports, Partner Reports, Taxes & Fiscal, Management (Invoice Analysis, Analytic Report) |
| 35 | Configuration | Administrator | Settings, Accounting, Invoicing, Online Payments, Analytic Accounting |

Inside Configuration → Accounting, in display order: Chart of Accounts, Taxes, Journals, Reporting, Currencies, Fiscal Positions, Multi-Ledger, Tax Groups (developer mode only), Cash Roundings (with the cash-rounding group).

The entries that belong to this domain are:

| Menu path | Action |
|---|---|
| Accounting → Transactions → Journal Entries | the entry list |
| Review → Control → Journal Items | the item list |
| Configuration → Accounting → Chart of Accounts | the account list |
| Configuration → Accounting → Journals | the journal list |
| Configuration → Accounting → Multi-Ledger | the journal-group list |
| Configuration → Settings | the accounting settings |
| Dashboard | the journal cards |

---

## 2. Window actions

| Name | Entity | Views offered | Default filters and grouping |
|---|---|---|---|
| Dashboard | Journal | cards, form | only the journals marked for the dashboard |
| Journal Entries | Journal Entry | list, cards, form, activity | default document type plain entry; posted only; the due-date column hidden |
| Journal Items | Journal Item | list, pivot, graph, cards | accountable items only; not cancelled; posted only |
| Journal Items (grouped by entry) | Journal Item | list, pivot, graph, cards | accountable items only; posted only; grouped by entry; creation disabled |
| Journal Items (grouped by matching) | Journal Item | list, pivot, graph, cards | accountable items only; posted only; groups expanded |
| Sales | Journal Item | list, pivot, graph | posted only; sale journals; grouped by entry |
| Partner Ledger | Journal Item | list, pivot, graph | posted only; trade receivable and trade payable; unreconciled; grouped by counterpart |
| Journal Items of one counterpart | Journal Item | list | posted only; filtered and defaulted on the counterpart of the record it is opened from |
| Chart of Accounts | Account | list, cards, form | — |
| Journals | Journal | list, cards, form | — |
| Multi-ledger | Journal Group | list, form | — |
| Settings | Settings | form | the accounting section |
| Unmerge account | Account | list | used only as the target of the confirmation dialogue |
| Merge accounts | Account merge wizard | form, in a dialogue | Bound to the Account as a contextual action of its list and card presentations, restricted to the Administrator group. It carries the name "Merge accounts". A second window action on the same form exists as an operation of the wizard itself, so that the dialogue can be shown again on the record it is already filling; that one carries the name "Merge Accounts" |

---

## 3. Views

### The account list (Chart of Accounts)

An editable list showing, per row: the code (the per-company code of the active company, or the code of the first visible company followed by that company name in parentheses), the name, the account type, the reconcilable flag, the default taxes, the tags, the current balance and the companies. A side panel filters by account root, that is by the first two characters of the code.

Search: typing matches the name by substring, the code by prefix, or the description by substring.

| Filter | Condition |
|---|---|
| Receivable | the account type is Receivable |
| Payable | the account type is Payable |
| Equity | the internal group is equity |
| Assets | the internal group is asset |
| Liability | the internal group is liability |
| Income | the internal group is income |
| Expenses | the internal group is expense |
| Account with Entries | the account carries at least one journal item |
| Inactive Accounts | the account is archived |

Grouping: by account type. A search field on the account type is also offered.

Three further filters exist but are hidden, used only when the list is opened from another screen: Fixed Assets, Frequent Expenses and Cost of revenue.

### The account form

Shows the name, the code (and, when the user belongs to several companies, a tab with one editable code per company), the type, the reconcilable flag, the account currency, the default taxes, the tags, the non-trade flag, the internal notes, the companies, the opening amounts and the description. A button leads to the taxes that use the account, showing the count. The audit trail and the activity panel are attached.

### The journal entry list

Columns: the number, the counterpart, the accounting date, the reference, the total, the status. Multi-row editing is available on a dedicated variant used by the actions that show a filtered set.

### The journal entry form

- Header: the state buttons (Post, Reset to Draft, Cancel, Request Cancellation, Reverse, Secure) and the status bar with the three states.
- Body: the number (with the placeholder showing the number that would be taken), the counterpart, the accounting date, the reference, the journal, the company, the currency, the automatic posting mode and its end date.
- Above the number, a warning line appears whenever the "number too low" flag of `entities.md` is set. Its text is "The current highest number is *the highest number of the chain*. You might want to put a higher number here." — the number is shown inline inside the sentence, between the first full stop and the word that follows it. The line is hidden as soon as the flag clears.
- A notebook with the journal items (account, label, counterpart, taxes, tax grids, analytic distribution, debit, credit, foreign amount, currency, matching number, due date), the other information and, when relevant, the hash.
- The audit trail, the activity panel and the attachments.

### The journal item list

Columns: the accounting date, the entry number, the journal, the account, the counterpart, the label, the taxes, the tax grids, the matching number, the debit, the credit, the balance, the foreign amount, the currency, the running total. The running total column is only filled when the list requests it.

Search: typing matches the label, the entry number, the entry reference or the counterpart. Dedicated search fields exist for the label, the reference, the document date, the accounting date, the due date, the discount date, the amount (matching either the debit or the credit), the account, the account type, the counterpart, the journal, the ledger group, the product, the product category, the entry, the taxes, the originating tax and the reconciliation model.

| Filter | Condition |
|---|---|
| Unposted | the entry is draft |
| Posted | the entry is posted |
| Not Secured | the entry is posted and carries no hash (shown with the inalterability group) |
| To Review | the entry is not draft and is not marked reviewed |
| Unreconciled | the item is not reconciled |
| With residual | the residual is not zero and the account allows matching |
| Sales / Purchases / Bank / Cash / Credit / Miscellaneous | the journal type |
| Journals | the journals given in the context (hidden unless a context supplies them) |
| Payable | the account type is Payable and the account is a trade account |
| Receivable | the account type is Receivable and the account is a trade account |
| Non Trade Payable, Non Trade Receivable | the same with the non-trade flag (hidden) |
| Profit-and-loss accounts | the internal group is income or expense |
| No Bank Transaction | the item comes neither from a bank transaction nor from a payment (hidden) |
| Date, Invoice Date | period filters on the two dates |
| Report Dates | the accounting date inside the range given in the context (hidden) |
| Analytic Accounts | the distribution mentions the analytic accounts given in the context (hidden) |

Grouping: by entry number, account, counterpart, journal, accounting date, document date, taxes, tax grid, matching number. A side panel groups by account root.

### The journal entry search view

Typing matches the number, the reference or the counterpart. Dedicated search fields: the number, the reference, the document date, the accounting date, the total, the counterpart, the journal.

| Filter | Condition |
|---|---|
| Unposted | the state is draft |
| Posted | the state is posted |
| Not Secured | the state is posted and no hash is present (shown with the inalterability group) |
| Reversed | the payment status is reversed |
| Manual | the automatic posting mode is "No" |
| Sales / Purchases / Bank / Cash / Credit / Miscellaneous | the journal type |
| Date, Invoice Date | period filters |

Grouping: by counterpart, journal, status, payment method, accounting date, document date, company.

### The journal form

Shows the name, the code, the type, the company, the currency, and a notebook whose tabs depend on the type: the journal entries settings (default account, dedicated numbering switches, hash switch, numbering override), the incoming electronic mail, the money-in and money-out method lines with their outstanding accounts, and for a bank journal the bank account and the statement source.

### The journal group form

Shows the name, the company and the excluded journals.

### The lock exception list and form

Shows the beneficiary, the lock date being relaxed, the value it is relaxed to, the original company value, the validity, the reason and the state. A button opens the journal items created or modified while the exception was in force.

---

## 4. The accounting dashboard

One card per journal marked for the dashboard, ordered by the journal ordering, coloured by the journal colour.

### Figures shown on every card

| Figure | Definition |
|---|---|
| Entries count | the number of entries of the journal in the active companies |
| Has entries / has posted entries | whether at least one entry, respectively at least one posted entry, exists |
| Numbering holes | whether at least one entry of the journal flagged as opening a gap exists, after the fiscal lock date that applies to the user with exceptions ignored; the check is grouped by numbering prefix |
| Unhashed entries | whether the journal secures posted entries and at least one posted, unhashed entry exists after the applicable fiscal lock date |
| Activities | the scheduled activities of the journal, grouped by state |

### Figures shown on a miscellaneous journal card

| Figure | Definition |
|---|---|
| Number of draft entries | the entries of the journal in the active companies that are draft and whose automatic posting mode is "No" |
| Drop zone | accepts files and creates entries with them; restricted to the full-accounting group; the label is "Drop to create journal entries with attachments." |
| Onboarding checklist | the accounting checklist of the company |

### Figures shown on a liquidity journal card

Specified in `../payments-and-bank-reconciliation/`; the balance figure itself is defined in `calculations.md` of this folder because it is an aggregation of journal items: the sum of the foreign amounts of the items on the default account of the journal when the journal has a foreign currency, and the sum of their balances otherwise, excluding cancelled entries and non-accountable items, **regardless of which journal the items belong to**.

### Figures shown on a sale or purchase journal card

Specified in `../accounts-receivable/` and `../accounts-payable/`.

### Buttons on the cards

| Button | Effect |
|---|---|
| New entry | opens a new entry of that journal |
| Post all entries | opens the validate-entries wizard on the draft entries of that journal |
| Show sequence holes | opens the entry list restricted to the entries of the journal whose numbering prefix contains a hole |
| Show unhashed entries | opens an entry list named "Journal Entries to Hash", restricted to the entries of the chains that would be hashed. When that list holds exactly one entry, the form of that entry is opened instead of a list. The selection is **wider** than the one behind the indicator that makes the button appear: see the note below |
| Configure | for a bank journal with no statement source, opens the bank setup dialogue |

**The unhashed-entries button and the unhashed-entries indicator do not select the same entries.** Both start from the same search: the entries of that journal whose journal secures posted entries, which carry no hash, and whose accounting date is **strictly after** the effective fiscal lock date of the acting user for that journal. Both then run the chain selection of `state-machines.md` with hashing forced, so the journal setting is ignored at that step. They differ in one flag:

| | Indicator `has_unhashed_entries` | Button "Show unhashed entries" |
|---|---|---|
| Entries whose counter is **below** the last already hashed entry of the chain | excluded | **included** |
| Stops at the first hit | yes, it only answers "is there any" | no, it collects them all |

So the list the user sees can contain entries that the indicator never counted: those are the entries that were left unhashed behind a part of the chain that was hashed later. The reverse cannot happen. The shared cut-off at the fiscal lock date means that neither the indicator nor the button ever offers an entry of a closed period.

---

## 5. Named remote operations

These are the operations callable by name on a record or on the model. Each is listed with its inputs and its output.

### On the Journal Entry

| Operation | Inputs | Output | Purpose |
|---|---|---|---|
| Post | a set of entries | nothing, or the abnormal-figure confirmation dialogue, or the automatic-posting proposal dialogue | posts the entries in hard mode |
| Validate with confirmation | a set of entries | nothing, or the confirmation dialogue | posts directly the entries that need no confirmation and opens the wizard for the others |
| Reset to draft | a set of entries | true | see the workflow |
| Cancel | a set of entries | nothing | see the workflow |
| Request cancellation | one entry | nothing | refuses unless the document needs an approved cancellation |
| Secure | a set of entries | nothing | hashes the chains, ignoring the journal setting |
| Reverse | a set of entries | the reversal wizard | opens the wizard, renamed "Credit Note" when the selection is an invoice |
| Switch document type | a set of entries | nothing | turns an invoice into a credit note and back; refuses when a number was consumed |
| Duplicate | one entry | the form of the copy | |
| Mark as reviewed | a set of entries | nothing | sets the reviewed flag on the posted entries |
| Toggle payment block | one entry | nothing | switches the payment status to or from blocked |
| Activate the currency | one entry | nothing | unarchives the currency of the entry |
| Delete duplicates | a set of entries | nothing | deletes the entries detected as duplicates of the selection |
| Attach an outstanding item | one entry, one item identifier | the result of the reconciliation | matches that item with the unmatched items of the entry on the same account |
| Remove a matched amount | one entry, one match identifier | the result of the deletion | deletes that match |
| Open the matched payments | one entry | a window action on the payments | |
| Open the matched items | one entry | a window action on the items of the matched group | |
| Open the source document | one entry | a window action on the record that produced the entry | |
| Open the created cash-basis entries | one entry | a window action | |
| Open the adjusting entries | one entry | a window action on the adjusting entries it produced, always named "Adjusting Entries" | |
| Open the origin entries of an adjusting entry | one entry | a window action on the entries it was produced from, whose name is the origin label of the entry when the entry has exactly **one** adjusting entry, and the literal "Invoices" in every other case | see the note below |
| Read the currency rate | a company, a currency, a date | the rate | used by the client to display the expected rate |
| Refresh the currency rate | a set of entries | nothing | resets the stored rate to the expected one |
| Check the numbering chain | a set of entries | true when they are the last ones of their chains | |

The name of the action that opens the origin entries of an adjusting entry mixes two conditions that are not the same one. The origin label of `entities.md` is the label of the document type of the single **origin** entry, and it is empty when there are several origins; the choice between that label and "Invoices" is made on the number of **adjusting** entries. So an entry with one adjusting entry and several origins is opened under an empty name, and an entry with one origin and two adjusting entries is opened under "Invoices" although its single origin has a perfectly good label. **Compatibility finding.** A corrected behaviour would test the number of origin entries, which is what the label describes, and fall back to "Invoices" only when the label is empty. The observed behaviour is the one specified above.

### On the Journal Item

| Operation | Inputs | Output |
|---|---|---|
| Reconcile | a set of items | nothing; the matches are created |
| Remove the reconciliation | a set of items | nothing |
| Unreconcile the matched entries | the selection in the context | nothing; the whole matched group is undone |
| Open the matched items | one item | a window action |
| Open the source document | one item | a window action |
| Open the automatic transfer wizard | a set of items, optionally a default action | the wizard |
| Register a payment | a set of items | the payment register wizard |

### On the Account

| Operation | Inputs | Output |
|---|---|---|
| Open the related taxes | one account | a window action on the taxes that use it |
| Merge | a set of accounts | the merge dialogue, that is the form of the account merge wizard filled from the selection |
| Unmerge | a set of accounts | a client reload; raises the confirmation dialogue first |
| Read the import templates | none | the list of downloadable templates |

### On the Journal

| Operation | Inputs | Output |
|---|---|---|
| Create documents from files | a set of attachment identifiers | a window action on the created documents |
| Configure the bank journal | one journal | the bank setup dialogue |
| Open the dashboard action | one journal, a chosen target | the corresponding window action, prepared as described below |
| Post all entries | one journal | the validate-entries wizard |
| Show the numbering holes | one journal | a window action named "Journal Entries" |
| Show the unhashed entries | one journal | a window action named "Journal Entries to Hash", or the form of the single entry when there is only one |

**How a dashboard action is prepared.** Opening an action from a journal card takes the shipped window action of the requested target and adjusts it before returning it:

1. The context gets the journal as the default journal of anything created from the action.
2. When the context asked for the journal to be a default search filter, that filter is added and the request is then removed, so that the filter is not applied a second time further down.
3. Any grouping carried in the context is dropped.
4. When, and only when, the context asks for the action to be restricted by a domain, two things happen: the restriction is applied — either the explicit one carried in the context, or, by default, "the journal is this journal, or the record has no journal at all" — and the **name of the action is rewritten** to the pattern "*the original name of the action* for journal *the name of the journal*". An action opened without that request keeps its shipped name unchanged.

So the same shipped action appears under its plain name when it is opened unrestricted and under the composed name when it is opened restricted to one journal.
| Fetch incoming electronic invoices | one journal | nothing in the core; overridden by exchange packages |
| Refresh the status of outgoing electronic invoices | one journal | nothing in the core |

### On the Company

| Operation | Inputs | Output |
|---|---|---|
| Check the hash integrity | one company | the printable integrity report |
| Open the bank setup | none | the bank setup dialogue |
| Open the credit-card setup | none | the credit-card setup dialogue |
| Get the chart of accounts or fail | one company | the first account, or a redirect to the configuration panel |
| Compute the fiscal year of a date | one company, a date | the start and the end of the fiscal year |
| Get the next group-payment communication | one company | the next value of the group-payment sequence |

### On the wizards

| Wizard | Operation | Output |
|---|---|---|
| Reversal | Reverse | a window action on the reversals |
| Reversal | Reverse and Modify | a window action on the new draft copies |
| Automatic transfer | Do the action | a window action on the created entries, named "Generated Entries"; when exactly one entry was created the form of that entry is opened instead of the list |
| Resequence | Resequence | nothing |
| Validate entries | Validate | a close action, or the automatic-posting proposal dialogue |
| Secure entries | Secure entries | nothing |
| Lock exception | Revoke | nothing |
| Account merge | Merge | a success notification carrying the message "Accounts successfully merged!", which closes the dialogue when it is dismissed |
| Fiscal year opening | Save the fiscal-year step | marks the "Set Periods" checklist step as done, refreshes the checklist when the step had not been done before, and reloads the client |
| Lock exception | Show the audit trail during the exception | a window action on the journal items |

---

## 6. Routes

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/invoices` and `/my/invoices/page/<page number>` | hypertext transfer | signed-in user | The list of the documents of the counterpart, paginated, with sorting and filtering. Specified in `../accounts-receivable/`. |
| `/my/invoices/<invoice identifier>` | hypertext transfer | public, with an access token | One document, optionally as a printable file. Specified in `../accounts-receivable/`. |
| `/my/journal/<journal identifier>/unsubscribe` | hypertext transfer, read and write | public, with a signed token | Removes one electronic-mail address from the copy list of a journal. |
| `/account/download_invoice_attachments/<attachment identifiers>` | hypertext transfer | signed-in user | Downloads the given attachments, as a single file or as an archive. |
| `/account/download_invoice_documents/<entry identifiers>/<file type>` | hypertext transfer | signed-in user | Downloads the printable file or the structured file of the given documents. |
| `/account/download_move_attachments/<entry identifiers>` | hypertext transfer | signed-in user | Downloads every attachment of the given entries, renaming duplicates. |
| `/terms` | hypertext transfer | public | Renders the terms and conditions page of the company. |

### The unsubscribe route

```
 1. The link carries a token signed with the internal signing key, over the scope
    "account_journal_notification_unsubscribe" and the pair (address, journal identifier).
 2. The route verifies the token; an invalid token renders an error page.
 3. On confirmation the address is removed from the copy list of the journal; when the
    address was not in the list, nothing happens and the page says so.
```

---

## 7. Reports and printable documents

### The hash integrity report

A printable document produced for one company. Its stored report name is "Hash integrity result" followed by the three-letter abbreviation of Portable Document Format.

Header: the printing date.

First table, one row per journal and numbering prefix:

| Column | Width | Content |
|---|---|---|
| Journal (Sequence Prefix) | 30 % | the journal name followed by the prefix in parentheses |
| Restricted | 20 % | the letter V when the journal secures posted entries, the letter X otherwise |
| Check | 50 % | the status message: "Entries are correctly hashed", "Corrupted data on journal entry with id *the identifier* (*the number*)." or "There is no journal entry flagged for accounting data inalterability yet." |

Second table, one row per verified chain:

| Column | Content |
|---|---|
| Journal | the journal name with the prefix |
| First Hash | the hash of the first verified entry |
| First Entry | the number of the first verified entry and its date |
| Last Hash | the hash of the last verified entry |
| Last Entry | the number of the last verified entry and its date |

### Extra items added to the print menu

The print menu of the entry list and of the entry form is not fixed: an operation is asked, for the current selection, which extra items it should carry. This domain adds exactly one.

| Key | Label | Condition | Effect |
|---|---|---|---|
| `download_all` | "Export ZIP" | At least one selected entry has exportable documents | Downloads the exportable documents of every selected entry that has some, as one archive |

An entry has exportable documents only when it is **posted**. For a posted purchase document (vendor bill, vendor credit note or purchase receipt) the exportable document is the main attachment of its message thread, and only that one; an entry without a main attachment contributes nothing. For every other posted entry the exportable documents are its legal documents — the printable file and the structured files produced for it — as specified in `../accounts-receivable/` and `../electronic-invoicing-and-document-exchange/`. A selection in which no entry has any exportable document adds no item at all, so the menu shows no "Export ZIP" entry.

### Server actions bound to a record

One server action is shipped by this domain and appears in the action menu of the record it is bound to.

| Name | Bound to | Shown on | Effect |
|---|---|---|---|
| "Share" | Journal Entry | the form only | Opens the sharing dialogue for the entry: it prepares a link carrying an access token so that the document can be opened by someone who has no account, and offers to send that link. The dialogue itself and the token belong to the portal mechanism described in `../accounts-receivable/`. |

### Reports produced by neighbouring domains from this data

| Report | Where specified |
|---|---|
| General Ledger, Trial Balance, Balance Sheet, Profit and Loss, Cash Flow Statement, Partner Ledger, Aged Receivable and Payable, Journal Audit, Tax Report | `../financial-reporting/` |
| Invoice Analysis | `../accounts-payable/` |
| Printable invoice, credit note and receipt | `../accounts-receivable/` |
| Payment receipt and bank statement | `../payments-and-bank-reconciliation/` |

The three built-in list views of journal items reachable from the menus stand in for the three basic ledgers when the reporting package is not installed:

| View | Default filters and grouping | Stands in for |
|---|---|---|
| Journal Items | posted only | the general ledger |
| Journal Items grouped by entry | posted only, grouped by entry | the journal audit |
| Partner Ledger | posted only, trade receivable and trade payable, unreconciled, grouped by counterpart | the partner ledger |

Each of them offers the pivot and the graph presentation, so a trial balance is obtained by grouping the journal-item pivot by account with the balance as the measure.

---

## 8. Notifications and messages

### Messages logged by this domain

| Event | Message |
|---|---|
| An entry is duplicated | "This entry has been duplicated from *link to the original*" |
| An entry is created as a reversal | "This entry has been reversed from *link to the original*" |
| An entry belongs to a recurrence | "This recurring entry originated from *link to the first entry*" (appended to the previous one) |
| An entry is reversed | on the original: "This entry has been *link labelled "reversed"*" |
| An entry is secured | "This journal entry has been secured." |
| An entry is scheduled instead of posted | "This move will be posted at the accounting date: *the date*" |
| The automatic posting job fails on an entry | "The move could not be posted for the following reason: *the error message*" |
| Automatic posting is switched off because of a duplicate | "Auto-post was disabled on this invoice because a potential duplicate was detected." |
| An item of a previously posted entry is created, updated or deleted | "Journal Item *link* created", "… updated", "… deleted", each with the tracked values |
| An account is split off | "This account was split off from *link to the original* (*the company name*)." |
| A lock exception is granted | on the company: "*link labelled "Exception"* for *the beneficiary or "everyone"* valid until *the end moment* for '*the reason*'." |
| An adjusting entry is created | see the three messages in `workflows.md` |
| A transfer entry is created | see the two messages in `workflows.md` |

### Electronic-mail templates

| Template | Trigger | Recipients |
|---|---|---|
| Invoice subscriber notification | a document is sent or received on a journal that has a copy list | every address of the copy list, one message each, with a personal unsubscribe link |
| Electronic invoice notification | the same, used when the newer template is not available | the copy list, one message |

The templates of the commercial documents themselves belong to `../accounts-receivable/`.

---

## 9. Import and export

### Import templates offered

| Entity | Template |
|---|---|
| Account | "Import Template for Chart of Accounts", a spreadsheet |
| Journal Item | "Import Template for Journal Items", a spreadsheet |
| Journal Entry | one spreadsheet chosen by the document type, or none; see immediately below |

The Journal Entry is the only entity of the domain whose offer is **conditional**: the list of templates it returns is decided by the default document type carried in the reading context, that is by the screen the user opened the import from, and not by any record.

| Default document type in the context | Template offered |
|---|---|
| `entry` (a plain entry) | "Import Template for Misc. Operations" |
| `out_invoice` (a customer invoice) | "Import Template for Invoices" |
| `out_refund` (a customer credit note) | "Import Template for Credit Notes" |
| `in_invoice` (a vendor bill) | "Import Template for Bills" |
| `in_refund` (a vendor credit note) | "Import Template for Refunds" |
| none, or any other value | **no template at all** |

Exactly one template is offered in each of the five cases; the list never holds two. The two customer templates are two names for one and the same spreadsheet, and so are the two vendor templates, so three distinct files exist in all. The plain-entry template is the one that belongs to this domain; the four commercial ones are specified in `../accounts-receivable/` and `../accounts-payable/`. The sales-receipt and purchase-receipt document types are deliberately absent from the table: opening the import from a receipt screen offers nothing.

### Import behavior specific to this domain

| Entity | Behavior |
|---|---|
| Account | A name that begins with a token containing a digit is split into a code and a name. The uniqueness check of the codes runs once at the end of the import instead of per row, so that a whole chart can be reordered in one operation. An opening-balance column feeds the opening entry through the buffered mechanism. |
| Journal | A row without a type is imported as a miscellaneous journal. A row without a code takes the first five characters of the name, or the next free type-based code when that collides. |
| Journal Item | A matching number supplied by the import is stored with the letter `I` in front so that it cannot be confused with a real one; it is turned into a real reconciliation when every entry sharing that label is posted, switching the account to reconcilable if needed. |

### Export behavior

Every list view can be exported. Fields explicitly marked as non-exportable are the binary payloads used by the client: the outstanding-payments widget data, the matched-payments widget data, the totals structure, the payment-term details, the needed-terms structure, the quick-encoding values and the second view of the items named `journal_line_ids` in `entities.md`.

---

## 10. External integrations

This domain has no external service of its own. Three hooks exist for companion packages:

| Hook | Contract |
|---|---|
| Creating documents from files | A journal receives a set of attachments and returns the created documents. The core implementation creates one document per attachment group and then applies the automatic-posting rule; decoding packages override the extension point that turns a file into values. |
| Fetching and refreshing electronic invoices | Two operations on the journal, doing nothing in the core, and two computed switches that decide whether the buttons are shown. |
| Requesting a cancellation | One computed flag on the entry, false in the core, and one operation that refuses in the core. A country package sets the flag for documents already declared to an authority and implements the request. |

The incoming electronic-mail alias of a sale or purchase journal is an integration point of the messaging domain: the alias points at the journal entry model, and its default values force the company, the document type (a customer invoice for a sale journal, a vendor bill for a purchase journal, a plain entry otherwise) and the journal. The local part is derived from the first of the explicit alias name, the journal name, the journal code and the journal type that can be encoded and sanitised, and is suffixed with the company name (or identifier) when the company is not the main one, and further suffixed with the journal code when that local part already exists in the alias domain.

---

## 11. The dashboard data contract

The accounting dashboard reads two things per journal: a set of **computed fields** on the journal itself, and one **structured payload** produced for every displayed journal in a single batch.

### The computed fields read by the card

| Field | Type | Meaning |
|---|---|---|
| `show_on_dashboard` | boolean | Whether the journal is displayed at all |
| `color` | integer | The colour index of the card |
| `kanban_dashboard` | text | The serialised payload described below |
| `kanban_dashboard_graph` | text | The serialised series of the small graph |
| `json_activity_data` | text | The serialised list of activities |
| `entries_count` | integer | How many entries the journal holds in the active companies |
| `has_entries` | boolean | At least one entry exists |
| `has_posted_entries` | boolean | At least one posted entry exists |
| `has_sequence_holes` | boolean | At least one entry of the journal carries the gap flag after the applicable fiscal lock date |
| `has_unhashed_entries` | boolean | The journal secures posted entries **and** the search behind it finds at least one entry to hash after that date. The search itself does not test the state: it selects the entries of the journal that carry no hash, whose journal secures posted entries, and whose accounting date is strictly after the effective fiscal lock date of the acting user for that journal; the posted-state condition is applied only later, inside the chain selection, which also forces hashing regardless of the journal setting. A draft entry of such a chain therefore takes part in the search that produces this indicator, which is one of the two reasons the indicator and the list opened by the button can disagree; the other is stated in section 4 |
| `has_invalid_statements` | boolean | At least one statement of the journal is not valid or not complete |
| `current_statement_balance`, `has_statement_lines`, `last_statement_id` | various | Liquidity figures; specified in `../payments-and-bank-reconciliation/` |

### The keys of the payload, for every journal

| Key | Meaning |
|---|---|
| `currency_id` | The currency in which the figures are expressed: the journal currency, or the company currency |
| `show_company` | True when more than one company is active, or when the journal belongs to a company other than the current one |
| `company_name` | The name of the company of the journal |
| `onboarding` | The checklist to show on the card, with the state of each step and the action that opens it; a sale journal gets the invoicing checklist and a miscellaneous journal gets the accounting checklist |

### The keys added for a miscellaneous journal

| Key | Meaning |
|---|---|
| `number_draft` | The number of draft entries of the journal, in the active companies, whose automatic posting mode is "No" |
| `drag_drop_settings` | The drop zone: an image, the text "Drop to create journal entries with attachments." and the group allowed to use it, which is the full-accounting group |

The keys added for a liquidity journal and for a sale or purchase journal are specified in `../payments-and-bank-reconciliation/`, `../accounts-receivable/` and `../accounts-payable/`.

### The activity payload

One list per journal, built from the activities attached to the entries of the journal **and** from the activities attached to the journal itself, restricted to the active companies and to non-archived activities. Each element carries:

| Field | Meaning |
|---|---|
| the activity identifier | |
| the record identifier and the record model | either an entry or the journal |
| the summary | the free text of the activity |
| the status | `late` when the deadline is strictly before today, `future` otherwise |
| the activity type identifier, its name and its category | |
| the deadline | |

### The graph

| Journal type | Graph |
|---|---|
| bank, cash, credit card | the projected balance day by day, built from the current balance and the future-dated transactions |
| sale, purchase | the residual amount per period, with a caption and the legend "Residual amount" |
| miscellaneous | no graph: the caption and the legend are empty |

---

## 12. Client-side widgets fed by this domain

| Widget | Data field | Content |
|---|---|---|
| Outstanding payments to attach | `invoice_outstanding_credits_debits_widget` | The items available to be matched with the document, each with its identifier, its number, its amount in the document currency, its date and its currency; visible only to the Invoicing and Readonly groups |
| Matched payments | `invoice_payments_widget` | The matches already made, each with the amount, the date, the counterpart entry and the identifier of the match so that it can be removed; same visibility |
| Totals | `tax_totals` | The tax-and-total summary of an invoice-like document; specified in `../taxes/` |
| Payment term details | `payment_term_details` | The instalments with their dates and amounts; specified in `../accounts-receivable/` |
| Alerts | `alerts` | The warnings to show at the top of the form, each with a message, a severity and an optional action |
| Quick-encoding values | `quick_encoding_vals` | The account, the unit price and the taxes to propose on the next line in the total-driven capture mode |

All six are marked non-exportable: they exist only to render the form and are never part of an export.
