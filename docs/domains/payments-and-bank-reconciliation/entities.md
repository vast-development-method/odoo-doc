# Entities

This file specifies every entity of the payments and bank reconciliation domain: its purpose, its lifecycle, its complete field table, its relations, its uniqueness rules, its defaults, its computed fields with the exact rule that computes them, its ordering, its display rule, its archival behavior and its multi-company behavior.

Field tables use three columns: `Field (storage name)`, `Type`, `Meaning and rules`. Unless a field says otherwise it is stored, writable, copied when the record is duplicated, and not tracked in the record's message history.

Two entities defined in the general ledger domain are used on almost every page here. They are summarised at the end of this file under *Borrowed entities*, with only the fields this domain reads or writes.

---

## 1. Payment

**Payment** (`account.payment`, table `account_payment`).

### 1.1 Purpose

A Payment records one movement of money between the company and one counterparty: money received from a customer, money sent to a vendor, money received back from a vendor for a refund, money sent back to a customer for a credit note, or money moved between two of the company's own liquidity accounts. It owns, at most, one Journal Entry, which carries the actual accounting. It is not itself an accounting record: it is the business-level description of the movement, and the entry is derived from it and kept synchronised with it while the entry is still a draft.

A Payment answers four questions:

1. *Which direction?* — `payment_type` (payment direction): `inbound` means the company receives money, `outbound` means the company sends money.
2. *Which kind of counterparty?* — `partner_type`: `customer` or `supplier`. It selects which of the counterparty's two default accounts is used as the destination.
3. *Through which liquidity journal and by which method?* — `journal_id` and `payment_method_line_id`.
4. *How much, in which currency, on which date?* — `amount`, `currency_id`, `date`.

### 1.2 Lifecycle

A Payment passes through five states: `draft`, `in_process`, `paid`, `canceled`, `rejected`. The state machine, its guards and its side effects are specified in `state-machines.md`. In summary:

- A Payment is created in `draft` and normally has no Journal Entry yet. The single exception is a Payment created with an explicit write-off line, an explicit forced balance or explicit journal item values: those are created together with their Journal Entry and are immediately `in_process`.
- Confirming a Payment creates the Journal Entry if it does not yet exist, posts it and moves the Payment to `in_process`, unless the outstanding account used is a cash-type account, in which case the Payment goes straight to `paid`.
- A Payment becomes `paid` when the money is confirmed: either the liquidity journal item has no residual left (it has been matched with a bank transaction), or the liquidity account is not reconcilable at all, or all the documents it settles are themselves fully paid.
- A Payment can be cancelled (its draft entries are deleted, its posted entries are cancelled) or rejected (used when an external settlement fails).

### 1.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | text | The payment number. Computed and stored, editable only through the computation. Recomputed whenever the Journal Entry's name or the Payment's state changes. Rule: if the Payment has a database identifier, and (it has no name yet, or it has a Journal Entry whose name differs from the Payment's name), and its state is `in_process` or `paid`, then the name becomes the Journal Entry's name if the entry has one, otherwise the next value of the number sequence with code `account.payment` taken in the Payment's company for the Payment's date. Not copied when duplicating (the sequence is re-drawn on the copy when it is confirmed). |
| `date` | date | The accounting date of the movement. Required. Default: today in the user's time zone. Tracked in the message history. Written onto the Journal Entry. |
| `move_id` | link to Journal Entry | The generated Journal Entry. Indexed. Not copied. Company-checked. Empty for a draft Payment that has not been confirmed and was not created with explicit lines. |
| `journal_id` | link to Journal | The liquidity journal. Required. Computed, stored, user-editable, precomputed at creation. Company-checked. Not indexed on its own; covered by the composite index on journal and company. Computation rule: (a) if the Payment has no original record (that is, it is being created) and either a counterparty or a direction is known, take the counterparty's default payment method line for that direction (property field `property_inbound_payment_method_line_id` or `property_outbound_payment_method_line_id`, read in the Payment's company) and, if that line has a journal, use it; (b) otherwise, if no journal is set or the current journal belongs to another company, take the first journal of the company whose type is `bank`, `cash` or `credit`, in the journal ordering. |
| `company_id` | link to Company | Required. Computed, stored, user-editable, precomputed. Rule: if the journal's company is not among the ancestors of the current company value, set it to the first accessible branch of the journal's company, or of the active company when the journal has none. Covered by the composite index on journal and company. |
| `state` | selection | Required, default `draft`, tracked, not copied. Values: `draft` ("Draft"), `in_process` ("In Process"), `paid` ("Paid"), `canceled` ("Canceled"), `rejected` ("Rejected"). Computed, stored and user-editable; the computation is specified in `state-machines.md`. |
| `is_reconciled` | boolean | Computed and stored, read-only. True when the counterpart and write-off journal items of the Payment that sit on a reconcilable account have no residual left. See `calculations.md` for the exact rule. |
| `is_matched` | boolean | Computed and stored, read-only. Labelled "Is Matched With a Bank Statement". True when the liquidity journal item of the Payment has no residual left, that is, when the outstanding amount has been confirmed by a bank transaction; also true by definition when the journal's default account is itself used as the liquidity account. |
| `is_sent` | boolean | Read-only, not copied. Set by the operations *mark as sent* and *unmark as sent*. Used by check printing and by transfer-file generation to remember that the payment instruction has left the company. |
| `available_partner_bank_ids` | many-to-many to Bank Account | Computed, not stored. For an inbound Payment: the bank account attached to the journal. For an outbound Payment: the counterparty's bank accounts whose company is empty or the Payment's company. |
| `partner_bank_id` | link to Bank Account | "Recipient Bank Account". Computed, stored, user-editable, tracked. Restricted to `available_partner_bank_ids`. Company-checked. Deletion of the bank account is restricted while a Payment points at it. Computation: if the current value is not among the available accounts, take the first available account. |
| `qr_code` | rich text | Computed, not stored. Holds a rendered image of a payable quick response code plus the caption "Scan me with your banking app." when all of the following hold: the state is `draft` or `in_process`; a recipient bank account is set; that account is trusted for outgoing payments; the payment method code is `manual`; the direction is `outbound`; a currency is set; and a code could actually be generated for the amount, the memo and the currency. Otherwise empty. |
| `paired_internal_transfer_payment_id` | link to Payment | Cross-reference between the two Payments of an internal transfer. Indexed when not empty. Not copied. |
| `payment_method_line_id` | link to Payment Method Line | "Payment Method". Computed, stored, user-editable, not copied. Restricted to `available_payment_method_line_ids`. Computation, in order: if the direction is `inbound` and the counterparty's default inbound payment method line is available, use it; else if the direction is `outbound` and the counterparty's default outbound payment method line is available, use it; else if the current value is still available, keep it; else take the first available line; else empty. |
| `available_payment_method_line_ids` | many-to-many to Payment Method Line | Computed, not stored. The journal's payment method lines for the Payment's direction, minus the codes excluded by the exclusion hook (empty by default). |
| `payment_method_id` | link to Payment Method | Related to `payment_method_line_id.payment_method_id`, stored, tracked. Labelled "Method". |
| `available_journal_ids` | many-to-many to Journal | Computed, not stored. All journals of type `bank`, `cash` or `credit` that are an ancestor or a descendant of the active company and that have at least one payment method line in the Payment's direction. |
| `amount` | monetary in `currency_id` | The absolute amount of the movement. Always positive or zero; the direction is carried by `payment_type`, not by the sign. Database check constraint: the amount must be greater than or equal to zero. |
| `payment_type` | selection | Required, default `inbound`, tracked. Values: `outbound` ("Send"), `inbound` ("Receive"). |
| `partner_type` | selection | Required, default `customer`, tracked. Values: `customer` ("Customer"), `supplier` ("Vendor"). |
| `memo` | text | Free label. Tracked. Has an inverse: writing it writes the same text into the Journal Entry's reference field. |
| `payment_reference` | text | "Payment Reference": the reference of the instrument used, for example a check number or a transfer file name. Not copied. Tracked. |
| `currency_id` | link to Currency | The payment currency. Computed, stored, user-editable. Rule: the journal's currency, or, when the journal has none, the journal's company currency. |
| `company_currency_id` | link to Currency | Related to `company_id.currency_id`, read-only. |
| `partner_id` | link to Partner | "Customer/Vendor". Deletion of the partner is restricted while a Payment points at it. Restricted to partners that are either a top-level record or a company. Tracked. Company-checked. |
| `outstanding_account_id` | link to Account | Computed and stored, read-only, indexed when not empty. Equal to the payment account of the selected payment method line. This is the account the liquidity journal item is written on. Company-checked. |
| `destination_account_id` | link to Account | Computed, stored, user-editable, indexed when not empty. Restricted to accounts of type `asset_receivable` or `liability_payable`. Company-checked. Computation: for `customer`, the counterparty's receivable account read in the Payment's company, or, when there is no counterparty, the first receivable account of the company; for `supplier`, the counterparty's payable account, or the first payable account of the company. |
| `invoice_ids` | many-to-many to Journal Entry | The documents this Payment was created for, stored in the relation table `account_move__account_payment` (column `payment_id` towards the Payment, column `invoice_id` towards the entry). Populated even when the document has no entry yet and even when nothing has been reconciled. Not copied. |
| `reconciled_invoice_ids` | many-to-many to Journal Entry | Computed, not stored, searchable. The sale documents actually matched with this Payment through partial reconciliations, plus the sale documents already listed in `invoice_ids`. |
| `reconciled_invoices_count` | integer | Computed, not stored. The number of entries in the previous field. |
| `reconciled_invoices_type` | selection | Computed, not stored. `credit_note` when every reconciled sale document is a credit note, `invoice` otherwise. Used only to choose the label of the stat button. |
| `reconciled_bill_ids` | many-to-many to Journal Entry | Computed, not stored, searchable. The purchase documents matched with this Payment, same rule as for sale documents. |
| `reconciled_bills_count` | integer | Computed, not stored. |
| `reconciled_statement_line_ids` | many-to-many to Bank Transaction | Computed, not stored. The bank transactions whose journal items are matched, on the Payment's outstanding account, with a journal item of the Payment. |
| `reconciled_statement_lines_count` | integer | Computed, not stored. |
| `payment_method_code` | text | Related to `payment_method_line_id.code`, read-only. Used by every rule that branches on the method. |
| `payment_receipt_title` | text | Computed, not stored. Default value: "Payment Receipt". A hook for localisations that must print another title. |
| `need_cancel_request` | boolean | Related to `move_id.need_cancel_request`. True when the entry cannot be cancelled directly and a cancellation request must be raised instead. |
| `show_partner_bank_account` | boolean | Computed, not stored. False when the journal is a cash journal; otherwise true when the payment method code is one of the codes that use a bank account (by default only `manual`). |
| `require_partner_bank_account` | boolean | Computed, not stored. True when the state is `draft` and the payment method code is one of the codes that need a bank account (by default the list is empty; capability packages for transfer files add their codes to it). |
| `country_code` | text | Related to `company_id.account_fiscal_country_id.code`. Used by localisation-specific view rules. |
| `amount_signed` | monetary in `currency_id` | Computed, not stored, tracked. Equal to minus `amount` for an outbound Payment, to `amount` for an inbound one. |
| `amount_company_currency_signed` | monetary in `company_currency_id` | Computed and stored. When a Journal Entry exists: the sum of the balances of the liquidity journal items. Otherwise: `amount_signed` converted from the payment currency to the company currency at the Payment's date. |
| `duplicate_payment_ids` | many-to-many to Payment | Computed, not stored. Other Payments of the same company with the same counterparty, the same date, the same direction and the same amount, whose state is `draft` or `in_process`. See `business-rules.md` for the full duplicate rule. |
| `attachment_ids` | one-to-many to Attachment | The files attached to the Payment. |

Fields added when the online-payment capability package is installed:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `payment_transaction_id` | link to Payment Transaction | Read-only. The online transaction that produced or will produce this Payment. Access to the Payment implies access to the transaction. |
| `payment_token_id` | link to Payment Token | "Saved Payment Token". Restricted to `suitable_payment_token_ids`. Only tokens of providers that capture immediately are offered. |
| `amount_available_for_refund` | monetary | Computed. Equal to `amount` minus the sum of the amounts of the Payments whose source payment is this one, but only when the Payment came from a transaction, the provider and the method both support refunds, and the transaction is not itself a refund. Zero otherwise. |
| `suitable_payment_token_ids` | many-to-many to Payment Token | Computed. Tokens of the Payment's company and counterparty, belonging to the provider of the selected payment method line, whose provider does not capture manually. Empty when the method is not an electronic one. |
| `use_electronic_payment_method` | boolean | Computed. True when the payment method code equals one of the provider codes. |
| `source_payment_id` | link to Payment | Related to the payment of the source transaction of this Payment's transaction. Stored and indexed so refunds can be grouped. |
| `refunds_count` | integer | Computed. The number of Payments whose source payment is this one and whose transaction operation is a refund. |

### 1.4 Relations

- One Payment owns at most one Journal Entry (`move_id`); the Journal Entry points back through `origin_payment_id`, and a one-to-many `payment_ids` on the Journal Entry exists purely to make that reverse lookup cheap.
- A Payment may be listed on many Journal Entries through the many-to-many `matched_payment_ids` on the entry side, which the register-payment flow fills to remember which Payment settled which document.
- A Payment may appear on many Bank Transactions through the many-to-many relation `account_payment_account_bank_statement_line_rel`, filled when a reconciliation creates a Payment on the fly from a bank transaction.
- A Payment references exactly one Payment Method Line, one liquidity Journal, one outstanding Account, one destination Account, at most one Partner and at most one Bank Account.

### 1.5 Ordering, display and copying

- Default ordering: by `date` descending, then by `name` descending.
- Display name: the Payment's `name`, or the text "Draft Payment" when it has none.
- Copying: the copy keeps the journal and the payment method line of the original and drops the Journal Entry, the state, the number, the sent flag, the payment reference, the linked documents and the paired transfer reference.
- Deleting a Payment first resets any non-draft Journal Entry to draft, then deletes the entry, then deletes the Payment, and finally forces the payment state of the documents that were linked to it to be recomputed.

### 1.6 Multi-company behavior

- The company is derived from the journal and can only be a company the user may act for.
- Company consistency is enforced automatically across `journal_id`, `move_id`, `partner_id`, `partner_bank_id`, `outstanding_account_id` and `destination_account_id`.
- A record rule limits visibility to Payments whose company is among the user's active companies.
- Two composite indexes exist: one on (journal, company), and one on (journal, company) restricted to the rows where `is_matched` is not true, which is the index the liquidity dashboard uses.

---

## 2. Payment Method

**Payment Method** (`account.payment.method`, table `account_payment_method`).

### 2.1 Purpose

The catalogue of ways money can move. A Payment Method is global: it is not attached to a company or a journal. It is identified by a `code` used by program logic, has a display `name` that is translatable, and has a direction. Capability packages add entries to this catalogue (a check method, a direct debit method, one method per online provider) and describe, in the *method information registry*, how each code may be attached to journals.

### 2.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | text | Required, translatable. The label the user sees, for example "Manual Payment". |
| `code` | text | Required. The internal identifier the logic branches on, for example `manual`. |
| `payment_type` | selection | Required. `inbound` ("Inbound") or `outbound` ("Outbound"). |

Uniqueness: the pair (`code`, `payment_type`) is unique across the whole database. Violating it raises "The combination code/payment type already exists!".

### 2.3 The method information registry

Each method code has an entry in a registry that the platform builds at run time. An entry has up to four keys:

| Key | Meaning |
|---|---|
| `mode` | `unique` — the method may be attached to at most one journal per company. `electronic` — the method may be attached to at most one journal per company and per online provider. `multi` — the method may be attached to any number of journals and repeated on the same journal. |
| `type` | The journal types the method may be attached to, as a tuple drawn from `bank`, `cash`, `credit`. Default when absent: all three. |
| `currency_ids` | The currencies for which the method is eligible. When present, a journal qualifies if its own currency is one of them, or if the journal has no currency and its company's currency is one of them. |
| `country_id` | The country the company's fiscal country must equal for the method to be eligible. |

The core package ships exactly one entry: code `manual`, mode `multi`, journal types `bank`, `cash` and `credit`. The online-payment capability package adds one entry per provider code (excluding the codes `none` and `custom`), each with mode `electronic` and journal type `bank`.

### 2.4 Eligibility domain

For a given code, the set of journals the method may be attached to is built as follows.

1. If the code is empty, every journal qualifies.
2. Start from: journal type is in the registry entry's `type` (default: `bank`, `cash`, `credit`).
3. If currency filtering is requested and the entry names currencies, add: (the journal has no currency **and** the journal's company currency is one of them) **or** (the journal's currency is one of them).
4. If country filtering is requested and the entry names a country, add: the company's fiscal country equals that country.

The same domain is evaluated twice with different strictness: with both filters when checking whether a method may be offered on a journal, and without them when deciding which methods are technically available for a journal.

### 2.5 Lifecycle

- On creation of a Payment Method whose registry mode is `multi`, one Payment Method Line is created automatically on every journal matching the eligibility domain, named after the method.
- Deleting a Payment Method first deletes every Payment Method Line that points at it, then the method itself.

---

## 3. Payment Method Line

**Payment Method Line** (`account.payment.method.line`, table `account_payment_method_line`).

### 3.1 Purpose

The activation of one Payment Method on one Journal. It carries the two pieces of per-journal configuration that a Payment needs: the label to display and, above all, the **payment account** — the outstanding account the liquidity side of a Payment is written on when it is confirmed.

### 3.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | text | Computed, stored, user-editable. Defaults to the payment method's name when empty. When the online-payment capability package is installed and the line has a provider, it defaults to the provider's name instead. |
| `sequence` | integer | Default 10. Orders the lines inside a journal and therefore decides which method is proposed first. |
| `payment_method_id` | link to Payment Method | Required. Restricted to methods whose direction matches the line's direction (when a direction is known) and that appear in the journal's `available_payment_method_ids`. |
| `payment_account_id` | link to Account | The outstanding account. Not copied. Deletion of the account is restricted while a line points at it. Company-checked. Restricted to accounts of type `asset_current` or `liability_current`, plus the journal's own default account. |
| `journal_id` | link to Journal | The journal the method is activated on. Indexed when not empty. Company-checked. May be emptied instead of deleting the line (see below). |
| `default_account_id` | link to Account | Related to `journal_id.default_account_id`, read-only. Present so the restriction above can name it. |
| `code` | text | Related to `payment_method_id.code`, read-only. |
| `payment_type` | selection | Related to `payment_method_id.payment_type`, read-only. |
| `company_id` | link to Company | Related to `journal_id.company_id`, read-only. |
| `available_payment_method_ids` | many-to-many to Payment Method | Related to `journal_id.available_payment_method_ids`, read-only. |

Fields added by the online-payment capability package:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `payment_provider_id` | link to Payment Provider | Computed, stored, user-editable. Restricted to providers whose code equals the line's code. The computation picks, among the providers of the company that have this code, the first one not already used by another electronic line of the same journal. |
| `payment_provider_state` | selection | Related to `payment_provider_id.state`, read-only. |

### 3.3 Ordering, display and company scope

- Default ordering: by `sequence`, then by identifier.
- Display name: the line's name followed by the journal's name in parentheses, for example "Manual Payment (Bank)". When the context flag `hide_payment_journal_id` is set, the plain name is used instead.
- Company domain: a line is visible from a company if the line's company is an ancestor of it.

### 3.4 Deletion

Deleting Payment Method Lines is only allowed for lines that have never been used. The rule is applied line by line:

1. Count the Payments that point at the line (counted with elevated privileges so that a restricted user cannot bypass the check).
2. If the count is greater than zero, do not delete the line: instead clear its journal, which detaches it from the journal while preserving the historical link from the Payments.
3. Delete only the lines whose count is zero.

### 3.5 Uniqueness

Writing the name of a line re-runs the journal's payment method multiplicity check, specified in `business-rules.md`. In short: on one journal and for one direction, two lines may not share the same (method, name) pair unless the method's mode is `multi`; and a method whose mode is `unique` or `electronic` may not be attached twice within the same company (or the same company-and-provider pair).

---

## 4. Bank Statement

**Bank Statement** (`account.bank.statement`, table `account_bank_statement`).

### 4.1 Purpose

A Bank Statement is a checkpoint. It names a contiguous run of Bank Transactions of one journal and records two balances that the bank itself reported: the balance before the first transaction of the run and the balance after the last one. The system then checks two things: that the sum of the transactions in the run agrees with the difference between those two balances (*completeness*), and that the run starts exactly where the previous statement of the same journal ended (*validity*). A statement is therefore a reconciliation of the company's records against the bank's own report, independent of whether the individual transactions have been matched with invoices.

A statement is optional. Transactions may exist without ever being grouped into one.

### 4.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | text | "Reference". Computed, stored, user-editable, not copied. Recomputed when the creation timestamp changes. Rule: the journal's sequence prefix followed by a space (omitted when there is no journal) followed by the text "Statement " and the statement's date, or the date part of the creation timestamp when the statement has no date yet. |
| `reference` | text | "External Reference". Not copied. Holds the reference of whatever produced the statement outside the system: the name of an imported file, the identifier of an online synchronisation. |
| `date` | date | Computed, stored, user-editable, indexed. Recomputed when the transactions' ordering index or state changes. Rule: among the statement's transactions that have an ordering index, sorted by that index ascending, keep those whose state is `posted`; the statement's date is the date of the last one. Empty when there is none. |
| `first_line_index` | text | Computed and stored, read-only. The ordering index of the first transaction of the statement: among the statement's transactions that have an index, sorted ascending, the index of the first one. This is the field statements are ordered and chained by, because two statements may share a date. |
| `balance_start` | monetary in `currency_id` | "Starting Balance". Computed, stored, user-editable. See `calculations.md` for the anchoring algorithm. |
| `balance_end` | monetary | "Computed Balance". Computed and stored, read-only. Equal to `balance_start` plus the sum of the amounts of the statement's transactions whose state is `posted`. |
| `balance_end_real` | monetary | "Ending Balance". Computed, stored, user-editable. Its computation simply copies `balance_end`, so that a freshly created statement is complete by construction and the user only has to correct it when the bank says something else. |
| `company_id` | link to Company | Related to `journal_id.company_id`, stored. |
| `currency_id` | link to Currency | Computed and stored. The journal's currency, or the company's currency when the journal has none. |
| `journal_id` | link to Journal | Computed and stored, read-only in the computation sense. Equal to the journal of the statement's transactions. Company-checked. |
| `line_ids` | one-to-many to Bank Transaction | The transactions of the statement, linked through their `statement_id`. |
| `is_complete` | boolean | Computed and stored. True when the statement has at least one posted transaction **and** `balance_end` compares equal to `balance_end_real` in the statement currency. |
| `is_valid` | boolean | Computed, not stored, searchable. True when the statement is the first one of its journal or when its `balance_start` compares equal to the `balance_end_real` of the previous statement of the same journal. See `calculations.md`. |
| `journal_has_invalid_statements` | boolean | Related to `journal_id.has_invalid_statements`. |
| `problem_description` | text | Computed, not stored. When the statement is not valid: "The starting balance doesn't match the ending balance of the previous statement, or an earlier statement is missing." Otherwise, when the statement is not complete: "The running balance (the computed balance) doesn't match the specified ending balance." — the computed balance is substituted as a formatted amount in the statement's currency. Empty when the statement is both valid and complete. |
| `attachment_ids` | many-to-many to Attachment | "Attachments". Search access is bypassed on this relation. On creation and on single-record writes, the attachments named in the values are re-pointed at the statement (their model becomes the statement model and their identifier the statement's). Writing attachments on more than one statement at a time is silently ignored. |

### 4.3 Ordering, indexes and display

- Default ordering: by `first_line_index` descending, so the most recent statement comes first even when several statements share a date.
- Two indexes: (journal, date descending, identifier descending) and (journal, first line index).

### 4.4 Lifecycle and defaults

When a statement form is opened, its transactions may be pre-filled from the context in three ways:

1. **Split** — the context names a transaction to split at. The system finds the last transaction before it that belongs to a *different*, non-empty statement, and selects every transaction of the same journal whose ordering index is greater than that boundary and less than or equal to the named transaction's, in descending index order. This carves out the run that is not yet checkpointed and ends at the chosen transaction.
2. **Single transaction** — the context names one transaction and at most one record is active; that transaction alone is selected.
3. **Multiple transactions** — the context names one transaction and several records are active; all active transactions are selected, sorted. Two checks then run: every selected transaction must belong to the same journal, otherwise "A statement should only contain lines from the same journal."; and the selection must be contiguous once cancelled transactions are set aside, otherwise "Unable to create a statement due to missing transactions. You may want to reorder the transactions before proceeding." Cancelled transactions found inside the range are added to the selection so the run stays unbroken.

### 4.5 Archival and deletion

A Bank Statement has no archive flag. Deleting a statement is allowed; deleting a *transaction* that belongs to a statement that is both valid and complete is refused (see `business-rules.md`).

---

## 5. Bank Transaction

**Bank Transaction** (`account.bank.statement.line`, table `account_bank_statement_line`). Called a *statement line* in the storage names, a *transaction* in the user interface.

### 5.1 Purpose

One movement reported by the bank on one of the company's liquidity accounts. It is the anchor of bank reconciliation. A Bank Transaction always owns a Journal Entry, which is created and posted as soon as the transaction exists. That entry always has a *liquidity* journal item on the journal's default account, and, until the transaction has been fully reconciled, a *suspense* journal item on the journal's suspense account that holds the not-yet-explained side of the movement.

The entity delegates to its Journal Entry: every field of the Journal Entry (date, reference, state, narration, checked flag, company, journal, currency, journal items) is readable and writable directly on the Bank Transaction, and the entry is deleted with it.

### 5.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `move_id` | link to Journal Entry | Required, read-only, indexed, company-checked. Deleted together with the transaction. This is the delegation link: the Bank Transaction is stored as a satellite table of the Journal Entry. |
| `journal_id` | link to Journal | Inherited from the Journal Entry, stored, writable, precomputed, required. Not indexed alone; covered by the main composite index. |
| `company_id` | link to Company | Inherited from the Journal Entry, stored, writable, precomputed, required. Covered by the main composite index. |
| `statement_id` | link to Bank Statement | "Statement". Indexed. Empty while the transaction has not been checkpointed. |
| `payment_ids` | many-to-many to Payment | "Auto-generated Payments", stored in relation table `account_payment_account_bank_statement_line_rel`. The Payments created while reconciling this transaction. They are deleted when the reconciliation is undone. |
| `sequence` | integer | Default 1. Orders transactions that share a date. Note that it works in reverse: a **higher** sequence sorts **earlier**, because the ordering index stores its complement. |
| `partner_id` | link to Partner | The counterparty, when known. Deletion of the partner is restricted. Restricted to partners that are top-level or companies. Company-checked. Written onto both journal items of the entry. |
| `account_number` | text | "Bank Account Number". The counterparty's account number as the bank reported it, kept until a Bank Account record is created or found for it. |
| `partner_name` | text | The third-party name as the bank reported it, used when the partner could not be identified. Indexed when not empty. |
| `transaction_type` | text | The transaction type code as the bank reported it. |
| `payment_ref` | text | "Label". The description the bank reported. Indexed for substring search. Written as the label of both journal items. |
| `currency_id` | link to Currency | "Journal Currency". Computed and stored. The journal's currency, or the company's currency when the journal has none. |
| `amount` | monetary in `currency_id` | The amount as reported by the bank, in the journal's currency, signed: positive for money coming in, negative for money going out. Defaults to zero when not given at creation. |
| `running_balance` | monetary | Computed, not stored. The balance of the journal after this transaction, in occurrence order, anchored on statements. See `calculations.md`. |
| `foreign_currency_id` | link to Currency | "Foreign Currency". Optional. Set when the bank reports the movement in a second currency, for instance a payment in a foreign currency debited from an account held in the company currency. |
| `amount_currency` | monetary in `foreign_currency_id` | "Amount in Currency". Computed, stored, user-editable. Rule: when there is no foreign currency, it is cleared; otherwise, when a date is known and the field is still empty, it is set to `amount` converted from the journal currency to the foreign currency at the transaction's date. An explicit value is never overwritten. |
| `amount_residual` | floating-point number | "Residual Amount". Computed and stored. The part of the transaction that is not yet explained, signed like the journal items of the entry, expressed in the transaction currency. See `calculations.md`. |
| `country_code` | text | Related to `company_id.account_fiscal_country_id.code`. |
| `internal_index` | text | "Internal Reference". Computed and stored. The sort key of the transaction. See `calculations.md` for its exact construction. |
| `is_reconciled` | boolean | Computed and stored. True when the transaction is fully explained. |
| `statement_complete` | boolean | Related to `statement_id.is_complete`. |
| `statement_valid` | boolean | Related to `statement_id.is_valid`. |
| `statement_balance_end_real` | monetary | Related to `statement_id.balance_end_real`. |
| `statement_name` | text | Related to `statement_id.name`. |
| `transaction_details` | structured data | Read-only. The raw payload the bank or the import produced, kept for audit and for label matching. |

### 5.3 Indexes and ordering

- Default ordering: by `internal_index` descending — most recent transaction first.
- Three partial indexes, all on (journal, company, internal index): one unrestricted, one restricted to rows where `is_reconciled` is not true, one restricted to rows with no statement.

### 5.4 The two-or-three journal items

The Journal Entry of a Bank Transaction is split into three groups by account:

| Group | Selection rule |
|---|---|
| liquidity | journal items whose account equals the journal's default account. When no item matches, the group falls back to the items whose account type is `asset_cash` or `liability_credit_card`, and those items are then removed from the *other* group. |
| suspense | journal items whose account equals the journal's suspense account. |
| other | every remaining journal item. |

Exactly one liquidity item must exist at all times. At most one suspense item may exist. Those two invariants are enforced on synchronisation and are stated with their messages in `business-rules.md`.

### 5.5 Creation

Creating a Bank Transaction does all of the following, in order:

1. If a statement is given but no journal, the journal is copied from the statement.
2. If a journal and a foreign currency are both given and the foreign currency equals the journal's own currency (or the company currency when the journal has none), the foreign currency is cleared and the foreign amount set to zero. A transaction never carries a foreign currency equal to its journal currency.
3. The entry type is forced to a plain entry, overriding any default type inherited from the context.
4. An explicit counterpart account may be passed to replace the suspense account for this transaction only; it is removed from the values before the record is created.
5. A missing amount defaults to zero.
6. The records are created with an empty label so that the Journal Entry's own naming does not interfere.
7. For each transaction created without explicit journal items, the two default journal items are prepared (see `accounting-effects.md`) and created in one batch.
8. The Journal Entry is pointed back at the transaction, its narration is copied and its label cleared.
9. The Journal Entry's numbering is scheduled and its narration recomputation cancelled.
10. Every Journal Entry created this way is posted immediately. A Bank Transaction is never left in draft by the creation flow.

### 5.6 Deletion

- Transactions belonging to a statement that is both valid and complete cannot be deleted: "You can not delete a transaction from a valid statement.\nIf you want to delete it, please remove the statement first."
- Otherwise: transactions of a company whose audit trail is restrictive have their Journal Entry cancelled instead of deleted; the remaining entries are deleted outright after the transactions themselves are removed.

### 5.7 Undoing a reconciliation

The operation *undo reconciliation* on a set of transactions:

1. Refuses, for each transaction that is checked and reconciled and whose entry the current user may not review, with "Validated entries can only be changed by your accountant."
2. Removes every partial reconciliation touching any journal item of the transactions.
3. Deletes every Payment auto-generated from those transactions.
4. For each transaction, rewrites the entry's journal items from scratch: it clears them all and recreates the default liquidity and suspense pair, and it sets the checked flag to whether the current user is able to review the entry.

---

## 6. Reconciliation Model

**Reconciliation Model** (`account.reconcile.model`, table `account_reconcile_model`).

### 6.1 Purpose

A named preset used while reconciling bank transactions. It answers two questions: *does this transaction look like the case I described?* (the matching conditions) and *if so, what counterpart journal items should be written?* (the model lines). A model can be applied on demand by the user, or marked as automated so that a matching transaction is reconciled without intervention.

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `active` | boolean | Default true. Archiving a model removes it from every proposal without deleting it. |
| `name` | text | Required, translatable. |
| `sequence` | integer | Required, default 10. Models are evaluated in this order. |
| `company_id` | link to Company | Required, read-only after creation. Default: the active company. |
| `trigger` | selection | Required, default `manual`, tracked. Values: `manual` ("Manual") and `auto_reconcile` ("Automated"). An automated model validates the transaction by itself when it matches. |
| `next_activity_type_id` | link to Activity Type | "Next Activity". When set, applying the model schedules an activity of that type. |
| `can_be_proposed` | boolean | Computed and stored, not copied. True when the model is **not** a partner mapping and at least one of the following holds: it has a label condition, it has an amount condition, it has a counterparty condition, or it is automated. A model with no condition at all and no automation is only usable by picking it explicitly. |
| `mapped_partner_id` | link to Partner | Computed and stored, not copied. Set when the model is a *partner mapping*: it has a label condition, it has exactly one line, that line names a counterparty and names no account. Such a model does not propose counterpart lines; it only tells the reconciliation which counterparty a transaction belongs to. |
| `match_journal_ids` | many-to-many to Journal | "Journals". Restricted to journals of type `bank`, `cash` or `credit`. Company-checked. Empty means "every liquidity journal". |
| `match_amount` | selection | "Amount". Optional. Values: `lower` ("Is lower than or equal to"), `greater` ("Is greater than or equal to"), `between` ("Is between"). Tracked. |
| `match_amount_min` | floating-point number | "Amount Min Parameter". Tracked. Required in the form when an amount condition other than `lower` is chosen. |
| `match_amount_max` | floating-point number | "Amount Max Parameter". Tracked. Required in the form when the amount condition is `between`. |
| `match_label` | selection | "Label". Optional. Values: `contains` ("Contains"), `not_contains` ("Not Contains"), `match_regex` ("Match Regex"). Tracked. The condition is tested against the transaction's label, its raw transaction details and its note. |
| `match_label_param` | text | "Label Parameter". Tracked. The string to look for, or the regular expression to apply. |
| `match_partner_ids` | many-to-many to Partner | "Partners". Empty means "every counterparty". |
| `line_ids` | one-to-many to Reconciliation Model Line | The counterpart lines. Copied when the model is duplicated. |

### 6.3 Ordering, display and duplication

- Default ordering: by `sequence`, then by identifier.
- Duplicating a model appends " (copy)" to the name, repeatedly if needed, until the name is unique. When an explicit name is supplied in the copy values, it is kept as given.
- The model keeps a message thread, so changes to the tracked fields are logged.

### 6.4 Company scope

A record rule makes a model visible from a company when the model's company is an ancestor of it. The same rule applies to the model's lines.

---

## 7. Reconciliation Model Line

**Reconciliation Model Line** (`account.reconcile.model.line`, table `account_reconcile_model_line`).

### 7.1 Purpose

One counterpart journal item that the model writes. A line describes the account, the label, the taxes, the analytic distribution and, above all, **how much**.

### 7.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `model_id` | link to Reconciliation Model | Read-only, indexed when not empty. Deleted with its model. |
| `company_id` | link to Company | Related to `model_id.company_id`, stored. |
| `sequence` | integer | Required, default 10. Orders the lines; the order matters because percentage-of-balance lines consume the balance left by the previous lines. |
| `account_id` | link to Account | "Account". Deleted with the account (the line is removed when the account is). Excludes accounts of type `off_balance`. Company-checked. Required in the form unless the line names a counterparty (the partner-mapping case). |
| `partner_id` | link to Partner | "Partner". Used both to force the counterparty on the created journal item and, when it is the only content of a one-line model with a label condition, to turn the model into a partner mapping. |
| `label` | text | "Label", translatable. The label of the created journal item. |
| `amount_type` | selection | Required, default `percentage`. Values: `fixed` ("Fixed"), `percentage` ("Percentage of balance"), `percentage_st_line` ("Percentage of statement line"), `regex` ("From label"). |
| `amount` | floating-point number | "Float Amount". Computed and stored. The numeric reading of `amount_string`: the number it denotes, or zero when it is not a number. |
| `amount_string` | text | "Amount". Required, default `100`. Holds either a number or a regular expression, depending on `amount_type`. |
| `tax_ids` | many-to-many to Tax | "Taxes", stored in relation table `account_reconcile_model_line_account_tax_rel`. Deletion of a tax is restricted while a line uses it. Company-checked. |
| `analytic_distribution` | percentage map | Inherited from the analytic mixin. The analytic distribution to copy onto the created journal item. |

### 7.3 Amount semantics

| `amount_type` | `amount_string` holds | Meaning |
|---|---|---|
| `fixed` | a number | The counterpart line carries exactly this amount. A negative number produces a debit, a positive number a credit (the help text states this explicitly). |
| `percentage` | a number between 0 and 100 | The counterpart line carries this percentage of the **open balance**, that is, of what is still unexplained at the moment the line is evaluated. |
| `percentage_st_line` | a number | The counterpart line carries this percentage of the **transaction amount** as reported by the bank, regardless of what previous lines consumed. |
| `regex` | a regular expression | The amount is extracted from the transaction label by applying the expression and concatenating its capture groups. No delimiters are needed. Two documented shapes: a single group such as `BRT: ([\d,]+)` applied to a label like `R:9672938 10/07 AX 9415126318 T:5L:NA BRT: 3358,07 C:` yields 3358,07; and a two-group form such as `\s+0*(\d+?)(\d{2})(?=\s)` applied to `01870912 0009065 00115` splits the integer part from the last two decimal digits and yields 90.65. |

Changing the amount type resets the amount text: to `100` for the two percentage types, to `([\d,]+)` for the regular-expression type, and to the empty string for the fixed type.

### 7.4 Validation of the amount

Checked whenever the amount text changes:

| Condition | Message |
|---|---|
| Type is `fixed` and the numeric reading is zero | "The amount is not a number" |
| Type is `percentage_st_line` and the numeric reading is zero | "Statement line percentage can't be 0" |
| Type is `percentage` and the numeric reading is zero | "Balance percentage can't be 0" |
| Type is `regex` and the text does not compile as a regular expression | "The regex is not valid" |

### 7.5 Ordering

Default ordering: by `sequence`, then by identifier.

---

## 8. Partial Reconciliation

**Partial Reconciliation** (`account.partial.reconcile`, table `account_partial_reconcile`).

### 8.1 Purpose

The atom of matching. One record links exactly one journal item on the debit side to exactly one journal item on the credit side and records how much of each was matched. Everything else — the residual of a journal item, the payment state of an invoice, the matched flag of a Payment, the reconciled flag of a Bank Transaction — is derived from the set of Partial Reconciliations.

Three amounts are stored because two journal items may be expressed in two different currencies while both are also expressed in the company currency.

### 8.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `debit_move_id` | link to Journal Item | Required, indexed. The item whose balance is positive. |
| `credit_move_id` | link to Journal Item | Required, indexed. The item whose balance is negative. |
| `full_reconcile_id` | link to Full Reconciliation | "Full Reconcile". Not copied. Indexed when not empty. Set when the connected set this record belongs to becomes fully matched. |
| `exchange_move_id` | link to Journal Entry | Indexed when not empty. The exchange-difference entry created together with this matching, when one was needed. |
| `draft_caba_move_vals` | structured data | The values that produced a draft cash-basis tax entry, kept so the system can tell, when the invoice is posted, whether the matching may be kept or must be redone. |
| `company_currency_id` | link to Currency | Related to `company_id.currency_id`. A helper so the company-currency amount can be expressed as a monetary value. |
| `debit_currency_id` | link to Currency | Related to `debit_move_id.currency_id`, stored, precomputed. |
| `credit_currency_id` | link to Currency | Related to `credit_move_id.currency_id`, stored, precomputed. |
| `amount` | monetary in `company_currency_id` | Always positive. The matched amount expressed in the company currency. |
| `debit_amount_currency` | monetary in `debit_currency_id` | Always positive. The matched amount expressed in the currency of the debit item. |
| `credit_amount_currency` | monetary in `credit_currency_id` | Always positive. The matched amount expressed in the currency of the credit item. |
| `company_id` | link to Company | Computed, stored, user-editable, precomputed. Rule: when the debit item's entry is an invoice-like document, the debit item's company; otherwise the credit item's company. This makes the exchange-difference entry and the cash-basis entry land on the invoice's side when there is one. |
| `max_date` | date | "Max Date of Matched Lines". Computed, stored, precomputed. The later of the two items' dates. It is the date at which the matching becomes visible in the aged receivable and payable reports. |

### 8.3 Constraints

Both currency fields must be set. Otherwise: "Missing foreign currencies on partials having ids: <the identifiers>".

### 8.4 Creation side effects

Creating Partial Reconciliations, in order:

1. Creates the records.
2. Moves to `paid` the Payments that have no outstanding account, are currently `in_process`, and whose signed amount is matched exactly by this reconciliation (or, for a group payment, by the accumulation of reconciliations covering each of its documents). The exact rule is in `state-machines.md`.
3. Recomputes the matching number of every journal item involved and of every item transitively matched with them.

### 8.5 Deletion side effects

Deleting Partial Reconciliations, in order:

1. Collects the Payments without outstanding account that are currently `paid` and whose amount was covered by these reconciliations.
2. Collects the cash-basis tax entries whose origin is one of these reconciliations, and the exchange-difference entries linked to them.
3. Collects the Full Reconciliations they belong to.
4. Deletes the Partial Reconciliations themselves first, to avoid re-entering the deletion recursively.
5. Deletes the collected Full Reconciliations.
6. For the collected entries: those still posted are reversed with the flag that cancels both entries, dated on the original entry's date, or, when that date violates a lock date, on the day after the latest violated lock date, with the reference "Reversal of: <the entry number>"; those still in draft are deleted.
7. Recomputes the matching number of every still-existing involved journal item.
8. Moves the Payments collected in step 1 back to `in_process`.

---

## 9. Full Reconciliation

**Full Reconciliation** (`account.full.reconcile`, table `account_full_reconcile`).

### 9.1 Purpose

A marker. It is created when a connected set of matched journal items has nothing left to match, and it is what gives that set its permanent matching number: the numeric identifier of the Full Reconciliation, written as text on each item's `matching_number`.

### 9.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `partial_reconcile_ids` | one-to-many to Partial Reconciliation | "Reconciliation Parts". All the matchings inside the set. |
| `reconciled_line_ids` | one-to-many to Journal Item | "Matched Journal Items". Every item of the set. |

### 9.3 Creation

Creation takes the two lists as link commands, creates the marker with message tracking disabled, then writes the marker's identifier onto the listed journal items and onto the listed partial reconciliations in bulk, and finally recomputes the matching number of every item of the set.

### 9.4 Deletion

Deleting a Full Reconciliation clears the pointer from the journal items automatically, but the matching number is a plain text field that nothing recomputes on its own. The deletion therefore explicitly recomputes the matching number of every item that had pointed at the marker: each ends up either with no matching number at all, or with a partial matching number of the form `P` followed by a number when partial reconciliations survive.

---

## 10. Bank Account

**Bank Account** (`res.partner.bank`, table `res_partner_bank`).

### 10.1 Purpose

An account number held by a partner at a bank. It is used to identify the recipient of an outgoing payment, to identify the company's own account behind a liquidity journal, and to recognise the counterparty of an incoming bank transaction from the account number the bank reported.

### 10.2 Field table (platform layer)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `active` | boolean | Default true. Archiving is used instead of deletion. Tracked. |
| `acc_type` | selection | "Type". Computed, not stored. The type inferred from the number. The base list contains one value, `bank` ("Normal"); the international bank account number capability package appends `iban` ("IBAN"). The inference is specified in `calculations.md`. |
| `acc_number` | text | "Account Number". Required. Tracked. Searching on it sanitizes the searched value first and searches the sanitized column instead. |
| `clearing_number` | text | "Clearing Number". Tracked. A national routing number used where the account number alone does not identify the branch. |
| `sanitized_acc_number` | text | Computed and stored, read-only. The account number with every non-alphanumeric character removed and the rest upper-cased. Writing directly on this field is redirected: the value written is treated as the account number and re-sanitized. |
| `acc_holder_name` | text | "Account Holder Name". Computed, stored, user-editable, tracked. Defaults to the partner's name. Used when the holder's name differs from the partner's. |
| `partner_id` | link to Partner | "Account Holder". Required, indexed, tracked. Deleted with the partner. Restricted to partners that are companies or top-level records. |
| `allow_out_payment` | boolean | "Send Money". Default **false**, not copied, tracked. The trust flag: the account may only be used for outgoing payments when it is true. Help text: sending fake invoices with a fraudulent account number is a common phishing practice; always verify new account numbers, preferably by calling the vendor, because phishing usually happens when their electronic mail is compromised. |
| `bank_id` | link to Bank | "Bank". Tracked. |
| `bank_name` | text | Related to `bank_id.name`, writable. |
| `bank_bic` | text | Related to `bank_id.bic`, writable. The bank identifier code. |
| `sequence` | integer | Default 10. Orders a partner's accounts; the first one is the default recipient. |
| `currency_id` | link to Currency | Tracked. The currency the account is held in. |
| `company_id` | link to Company | Related to `partner_id.company_id`, stored, read-only. Empty for an account owned by a partner that is not company-restricted. |
| `country_code` | text | Related to `partner_id.country_code`. |
| `note` | long text | "Notes". |
| `color` | integer | Computed. 10 when the account is trusted for outgoing payments, 1 otherwise. Drives the colour of the card in the user interface. |

### 10.3 Field table (accounting layer)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `journal_id` | one-to-many to Journal | "Account Journal". Read-only, restricted to journals of type `bank`. Company-checked. Despite the singular name it is a collection, because the database allows several journals to point at one account; a constraint then forbids more than one. |
| `has_iban_warning` | boolean | Computed and stored. True when the account is untrusted, has a sanitized number, is of type `iban`, the partner has a country, and the first two characters of the sanitized number differ from that country's code. |
| `partner_country_name` | text | Related to `partner_id.country_id.name`. |
| `has_money_transfer_warning` | boolean | Computed and stored. True when the account is untrusted, has a sanitized number of type `iban`, and characters five to seven of the sanitized number (the institution code) are one of the known money-transfer service codes. |
| `money_transfer_service` | text | Computed. The name of the money-transfer service matching the institution code, when there is one. The shipped table maps `967` to "Wise", `977` to "Paynovate" and `974` to "PPS EU SA". |
| `partner_supplier_rank` | integer | Related to `partner_id.supplier_rank`. |
| `partner_customer_rank` | integer | Related to `partner_id.customer_rank`. |
| `related_moves` | one-to-many to Journal Entry | The entries that name this account as their counterparty account. |
| `user_has_group_validate_bank_account` | boolean | Computed per user. Whether the current user may set or clear the trust flag. |
| `lock_trust_fields` | boolean | Computed. True when the record already exists in the database and is trusted. While true, the account number, the sanitized number, the partner and the type may not be changed. |
| `duplicate_bank_partner_ids` | many-to-many to Partner | Computed. The partners of the other active accounts that carry the same account number and that are either in the same company or in no company at all. |

Fields added by the merchant-presented quick response code capability package:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `display_qr_setting` | boolean | Computed. False in the base package; country-specific packages turn it on where the merchant-presented code is usable. |
| `include_reference` | boolean | "Include Reference". When true, the payment reference is embedded in the generated code. |
| `proxy_type` | selection | "Proxy Type". Base value list: `none` ("None"). Country-specific packages add the proxy kinds their scheme accepts, such as a mobile number or a national identifier. |
| `country_proxy_keys` | text | Computed. The proxy kinds accepted for the account's country, empty in the base package. |
| `proxy_value` | text | "Proxy Value". The value of the chosen proxy. |

### 10.4 Uniqueness, creation, modification and deletion

- Database uniqueness: the pair (sanitized account number, partner) is unique. Message: "The combination Account Number/Partner must be unique."
- Only one journal may point at a Bank Account: "A bank account can belong to only one journal."
- Creation forces the trust flag to false first and only sets it afterwards, and only if the current user may trust accounts. Creation also refuses to create an account whose (partner, number) pair exists but is archived: "A bank account with Account Number <the number> already exists for Partner <the partner name>, but is archived. Please unarchive it instead." Every creation posts a note on the partner's message thread: "Bank Account <a link to the account> created".
- Modification: changing the trust flag at all requires the right to trust accounts, otherwise "You do not have the rights to trust or un-trust accounts."; and while an account is trusted, changing its number, its sanitized number, its partner or its type is refused with "You cannot modify the account number or partner of an account that has been trusted." unless the same write also sets the trust flag to false. Each write that changes a tracked field posts "Bank Account <a link> updated" on the partner's thread, and also on the previous partner's thread when the owner changed.
- Deletion is never a deletion: it archives the record and posts "Bank Account <a link> with number <the number> archived" on the partner's thread. Archiving through the dedicated operation also reloads the screen so the change is visible.
- Ordering: by `sequence`, then by identifier.
- Display name: the account number, followed by " - " and the bank's name when a bank is set. Under the display flag `display_account_trust`, the word "trusted" or "untrusted" is appended in parentheses.
- Company domain: an account is visible from a company when its company is an ancestor of it (an account with no company is visible everywhere).

---

## 11. Bank

**Bank** (`res.bank`, table `res_bank`).

### 11.1 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | text | Required. |
| `street`, `street2`, `zip`, `city` | text | Postal address parts. |
| `state` | link to Country State | "Fed. State". Restricted to states of the selected country when one is set. |
| `country` | link to Country | |
| `country_code` | text | Related to `country.code`, read-only. |
| `email` | text | |
| `phone` | text | |
| `active` | boolean | Default true. |
| `bic` | text | "Bank Identifier Code". Indexed. Stored upper-cased: both creation and modification upper-case the value before storing it. |

### 11.2 Ordering, display and search

- Default ordering: by `name`, then by identifier.
- Display name: the name, followed by " - " and the bank identifier code when there is one.
- Name search covers both the name and the bank identifier code. A "contains" search is rewritten as: the bank identifier code starts with the searched text (case-insensitive), or the name contains it. The negated form is the negation of that pair.
- Choosing a country that differs from the selected state's country clears the state; choosing a state fills the country from it.

---

## 12. Payment Register

**Payment Register** (`account.payment.register`). A transient record: it exists only for the duration of one user interaction and is discarded afterwards.

### 12.1 Purpose

It turns a selection of open receivable or payable journal items into one or several Payments, and it is where the user decides the date, the amount, the journal, the method, the recipient account, whether payments are grouped, and what to do with any difference between the amount proposed and the amount actually paid.

### 12.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `payment_date` | date | "Payment Date". Required. Default: today in the user's time zone. |
| `amount` | monetary in `currency_id` | Computed, stored, user-editable. The amount the user is about to pay. Its computation is specified in `calculations.md`. |
| `hide_writeoff_section` | boolean | Computed. True in early-payment-discount mode. |
| `communication` | text | "Memo". Computed, stored, user-editable. See `calculations.md` for the derivation. |
| `group_payment` | boolean | "Group Payments". Computed, stored, user-editable. Help: only one payment will be created per counterparty (and bank account) instead of one per document. Computation: when the wizard is editable, true exactly when every selected journal item belongs to the same document; otherwise false. |
| `early_payment_discount_mode` | boolean | Computed. True when a discount applies to at least one document **and** the amount equals either the default amount or the full amount. |
| `currency_id` | link to Currency | Computed, stored, user-editable, precomputed. The journal's currency, or the source currency of the batch, or the company currency. |
| `journal_id` | link to Journal | Computed, stored, user-editable, precomputed. Restricted to `available_journal_ids`. Company-checked. |
| `available_journal_ids` | many-to-many to Journal | Computed. The union, over all batches, of the liquidity journals of the batch's company that have at least one payment method line in the batch's direction. |
| `available_partner_bank_ids` | many-to-many to Bank Account | Computed. For an editable wizard: the recipient accounts available for the single batch. Empty otherwise. |
| `partner_bank_id` | link to Bank Account | "Recipient Bank Account". Computed, stored, user-editable. Restricted to the available accounts. |
| `company_currency_id` | link to Currency | Related to `company_id.currency_id`. |
| `qr_code` | rich text | Computed. Same conditions as on a Payment, plus a non-zero amount: it shows the code image and the caption "Scan me with your banking app." |
| `batches` | opaque value | Computed, not stored, not translated. The grouping of the selected journal items. See `calculations.md`. |
| `installments_mode` | selection | Computed, stored, user-editable, not translated. Values: `next` ("Next Installment"), `overdue` ("Overdue Amount"), `before_date` ("Before Next Payment Date"), `full` ("Full Amount"). |
| `installments_switch_html` | rich text | Computed. The one- or two-line explanation shown under the amount with a clickable phrase that switches to the other amount. |
| `installments_switch_amount` | monetary in `currency_id` | Computed. The amount that the clickable phrase switches to. |
| `custom_user_amount` | monetary in `currency_id` | Set when the user types an amount that matches none of the four proposed amounts. It makes the amount stop following the automatic computation. |
| `custom_user_currency_id` | link to Currency | The currency the custom amount was typed in, so that changing the currency can convert it. |
| `line_ids` | many-to-many to Journal Item | "Journal items", read-only, not copied, stored in relation table `account_payment_register_move_line_rel` (column `wizard_id`, column `line_id`). The selection the wizard works on. |
| `payment_type` | selection | Computed and stored, not copied. `outbound` ("Send Money") or `inbound` ("Receive Money"). |
| `partner_type` | selection | Computed and stored, not copied. `customer` or `supplier`. |
| `source_amount` | monetary in `company_currency_id` | "Amount to Pay (company currency)". Computed and stored. The absolute sum of the residuals of the batch's journal items in the company currency. |
| `source_amount_currency` | monetary in `source_currency_id` | "Amount to Pay (foreign currency)". Computed and stored. Equal to `source_amount` when the batch currency is the company currency, otherwise the absolute sum of the residuals in the batch currency. |
| `source_currency_id` | link to Currency | "Source Currency". Computed and stored. The currency shared by the journal items of the batch. |
| `can_edit_wizard` | boolean | Computed and stored. True when the selection forms exactly one batch. When false, the wizard shows no amount, no counterparty and no recipient account: it can only create the payments as the batches dictate. |
| `can_group_payments` | boolean | Computed and stored. Whether the grouping switch is offered. |
| `company_id` | link to Company | Computed and stored. |
| `partner_id` | link to Partner | "Customer/Vendor". Computed and stored. Deletion of the partner is restricted. Empty when the wizard is not editable. |
| `payment_method_line_id` | link to Payment Method Line | "Payment Method". Computed, stored, user-editable. Restricted to `available_payment_method_line_ids`. |
| `available_payment_method_line_ids` | many-to-many to Payment Method Line | Computed. The journal's lines for the wizard's direction; empty when no journal is chosen. |
| `payment_method_code` | text | Related to `payment_method_line_id.code`. |
| `payment_difference` | monetary in `currency_id` | Computed. The difference between what the selected documents ask for and the amount entered. |
| `payment_difference_handling` | selection | "Payment Difference Handling". Computed, stored, user-editable. Values: `open` ("Keep open"), `reconcile` ("Mark as fully paid"). |
| `writeoff_account_id` | link to Account | "Difference Account". Not copied. Company-checked. |
| `writeoff_label` | text | "Journal Item Label". Default `Write-Off`. The label of the write-off journal item. |
| `writeoff_is_exchange_account` | boolean | Computed. True when the wizard is editable, the difference is to be written off, the payment currency differs from the source currency, and the chosen difference account is one of the company's two exchange-difference accounts. |
| `show_payment_difference` | boolean | Computed. True when the difference is non-zero, the wizard is not in discount mode, the wizard is editable, grouping is either unavailable or switched on, and the chosen payment method line has an outstanding account. |
| `show_partner_bank_account` | boolean | Computed. Same rule as on a Payment. |
| `require_partner_bank_account` | boolean | Computed. Same rule as on a Payment, without the draft-state condition. |
| `country_code` | text | Related to `company_id.account_fiscal_country_id.code`, read-only. |
| `duplicate_payment_ids` | many-to-many to Payment | Computed. Only filled when the wizard is editable: the Payments that would be duplicates of the one about to be created, searched with the states `draft` and `posted`. |
| `is_register_payment_on_draft` | boolean | Computed. True when at least one selected journal item belongs to a draft entry. |
| `actionable_errors` | structured data | Computed. Currently one possible entry, keyed `unpaid_matched_payments`, produced when any document already has a Payment in state `in_process`: message "There are payments in progress. Make sure you don't pay twice.", action label "Check them", severity `danger`, and an action that opens those Payments. |
| `untrusted_bank_ids` | many-to-many to Bank Account | Computed. The recipient accounts that are needed but not trusted. |
| `total_payments_amount` | integer | Computed. The number of Payments that will be created. |
| `untrusted_payments_count` | integer | Computed. How many of them would target an untrusted account. |
| `missing_account_partners` | many-to-many to Partner | Computed. The counterparties for which a recipient account is required but none exists. |

Fields added by the online-payment capability package: `payment_token_id`, `suitable_payment_token_ids`, `use_electronic_payment_method`, with the same meanings as on a Payment; the token is copied into the created Payment.

---

## 13. Bank Setup Wizard

**Bank Setup Wizard** (`account.setup.bank.manual.config`). A transient record that delegates to a Bank Account: every Bank Account field is readable and writable on it.

### 13.1 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `res_partner_bank_id` | link to Bank Account | Required. Deleted with the wizard. The Bank Account being configured. |
| `new_journal_name` | text | Required. Default: the name of the linked journal. Writing it runs the journal-linking step. Changing the account number copies it into this field. |
| `linked_journal_id` | link to Journal | "Journal". Computed, and writing it runs the journal-linking step. Company-checked. Computation: the first journal already pointing at this Bank Account, or the default unlinked journal for the wizard's journal type. |
| `bank_bic` | text | "Bic". Related to `bank_id.bic`, writable. |
| `num_journals_without_account_bank` | integer | Default: the number of journals of type `bank` that have no bank account and are not the default unlinked bank journal. |
| `num_journals_without_account_credit` | integer | Same count for journals of type `credit`. |
| `company_id` | link to Company | Required, computed. Set to the active company when empty. |

### 13.2 Behavior

- On creation the partner is forced to the active company's partner: the wizard only ever configures the company's own accounts. The journal name is forced to the account number. When no bank is chosen but a bank identifier code is given, a Bank is searched by that code and created with that code as both name and code when not found.
- The default unlinked journal for a type is the first journal of that type that has no bank account **and** has never carried a journal entry.
- Saving the wizard links the journal: when no journal is selected, a new journal is created with the entered name, the next free sequence prefix for that journal type in the active company, that type, the active company, this Bank Account and the bank feed source `undefined`; when a journal is selected, that journal's bank account is set to this Bank Account and its name replaced by the entered name.
- Validating the wizard reloads the screen. Capability packages that import statements extend this step.

---

## 14. Borrowed entities

### 14.1 Journal — liquidity aspects

**Journal** (`account.journal`, table `account_journal`) is specified in `../general-ledger/entities.md`. The fields this domain depends on:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `type` | selection | The three liquidity types are `bank` ("Bank"), `cash` ("Cash") and `credit` ("Credit Card"). Every payment, every bank statement and every reconciliation model in this domain is restricted to those three. |
| `default_account_id` | link to Account | The liquidity account. For a `bank` journal the allowed account types are `asset_cash` and `liability_credit_card`; for `credit`, only `liability_credit_card`; for `cash`, only `asset_cash`. |
| `suspense_account_id` | link to Account | "Suspense Account". Computed, stored, user-editable, company-checked, restricted to accounts of type `asset_current`. Rule: for a non-liquidity journal it is empty; an explicit value is kept; otherwise it falls back to the company's journal suspense account; otherwise it is empty. Every Bank Transaction posts its unexplained side here. |
| `profit_account_id` | link to Account | Restricted to account types `income` and `income_other`. Records a surplus when a cash register's closing balance is above the computed one. |
| `loss_account_id` | link to Account | Restricted to account type `expense`. Records a shortfall in the same situation. |
| `inbound_payment_method_line_ids` | one-to-many to Payment Method Line | Computed, stored, user-editable, not copied, company-checked, filtered to direction `inbound`. The default computation creates one line per default inbound method, preserving the payment account already configured for that method if there was one. |
| `outbound_payment_method_line_ids` | one-to-many to Payment Method Line | The same for direction `outbound`. |
| `available_payment_method_ids` | many-to-many to Payment Method | Computed. The methods that may still be added to this journal, given each method's multiplicity mode and the methods already used within the company. |
| `selected_payment_method_codes` | text | Computed. The codes of all the journal's method lines, comma-separated and wrapped in commas, so a view can test for one code with a substring condition. |
| `bank_account_id` | link to Bank Account | "Bank Account". Deletion restricted, not copied, indexed when not empty, company-checked, restricted to accounts whose holder is the company's partner. |
| `bank_statements_source` | selection | "Bank Feeds". Default `undefined` ("Undefined Yet"). Capability packages that import or synchronise statements add their own values. |
| `bank_acc_number` | text | Related to `bank_account_id.acc_number`, writable. |
| `bank_id` | link to Bank | Related to `bank_account_id.bank_id`, writable. |
| `has_invalid_statements` | boolean | Computed, not stored. True when at least one statement of the journal is invalid. |
| `currency_id` | link to Currency | When set, the journal keeps its books in that currency and every payment and transaction defaults to it. |
| `company_id` | link to Company | Required. |
| `code` | text | The sequence prefix, at most five characters. It also prefixes the statement name. |

The default inbound method is the manual inbound method; the default outbound method is the manual outbound method.

### 14.2 Journal Item — reconciliation aspects

**Journal Item** (`account.move.line`, table `account_move_line`) is specified in `../general-ledger/entities.md`. The fields this domain depends on:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `account_id` | link to Account | The account. Only items on a reconcilable account, or on an account of type `asset_cash` or `liability_credit_card`, take part in reconciliation. |
| `balance` | monetary in company currency | Debit minus credit. |
| `debit`, `credit` | monetary in company currency | The two one-sided views of the balance; exactly one of them is non-zero for a non-zero item. |
| `amount_currency` | monetary in `currency_id` | The signed amount in the item's own currency; positive on the debit side. |
| `currency_id` | link to Currency | The item's currency. Equal to the company currency when the item is not in a foreign currency. |
| `amount_residual` | monetary in company currency | Computed and stored. What is left to match, in the company currency. |
| `amount_residual_currency` | monetary in `currency_id` | Computed and stored. What is left to match, in the item's currency. |
| `reconciled` | boolean | Computed and stored. True when both residuals are zero. |
| `matched_debit_ids` | one-to-many to Partial Reconciliation | The matchings where this item is on the credit side. |
| `matched_credit_ids` | one-to-many to Partial Reconciliation | The matchings where this item is on the debit side. |
| `full_reconcile_id` | link to Full Reconciliation | Set when the item belongs to a fully matched set. |
| `matching_number` | text | The label of the matching set: the Full Reconciliation's identifier as text when fully matched, `P` followed by a number when only partially matched, a value starting with `I` for an import placeholder, empty when unmatched. |
| `reconciled_lines_ids` | many-to-many to Journal Item | Computed. The items directly matched with this one, filtered to those the user may read. |
| `reconciled_lines_excluding_exchange_diff_ids` | many-to-many to Journal Item | Computed. The same, minus the journal items of the exchange-difference entries. |
| `exchange_move_ids` | many-to-many to Journal Entry | Computed. The exchange-difference entries produced by the matchings of this item. |
| `statement_line_id` | link to Bank Transaction | Set on the items of a Bank Transaction's entry. |
| `payment_id` | link to Payment | Set on the items of a Payment's entry. |
| `reconcile_model_id` | link to Reconciliation Model | Set on a journal item that a reconciliation model created, so the model can count what it produced. |
| `date_maturity` | date | The due date. It is the primary sort key of the reconciliation ordering. |
| `discount_date`, `discount_amount_currency`, `discount_balance` | date, monetary, monetary | The early-payment-discount triple, defined in `../accounts-receivable/`. Read here by the installment computation. |
| `payment_date` | date | Computed. The discount date when there is one and today is not past it, otherwise the due date. |
