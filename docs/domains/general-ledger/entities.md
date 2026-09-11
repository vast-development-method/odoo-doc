# General Ledger — Entities

This document describes every entity of the general ledger domain: its purpose, its lifecycle, every field with its type and rules, its relations, its uniqueness rules, its defaults, its computed fields and the rule behind each, its ordering, its display rule, its archival behavior and its multi-company behavior.

Field tables use three columns: the storage name of the field in code font, its type, and the rules that govern it. When a field is computed the table states what it is computed from and whether the computed value is stored in the database; when it is stored and editable, the table says so, because an editable computed field is recomputed only while its sources change and the user has not overridden it.

Contents:

1. [Account](#1-account)
2. [Account Group](#2-account-group)
3. [Account Root](#3-account-root)
4. [Account Tag](#4-account-tag)
5. [Account Code Mapping](#5-account-code-mapping)
6. [Journal](#6-journal)
7. [Journal Group](#7-journal-group)
8. [Journal Entry](#8-journal-entry)
9. [Journal Item](#9-journal-item)
10. [Partial Reconciliation](#10-partial-reconciliation)
11. [Full Reconciliation](#11-full-reconciliation)
12. [Lock Exception](#12-lock-exception)
13. [Automatic Sequence (abstract behavior)](#13-automatic-sequence-abstract-behavior)
14. [Company — ledger fields](#14-company--ledger-fields)
15. [Partner — ledger fields](#15-partner--ledger-fields)
16. [Wizard entities](#16-wizard-entities)

---

## 1. Account

Account (`account.account`, table `account_account`).

### Purpose

One line of the chart of accounts. An Account carries a code (which is **per company**, not global), a name, a classification called the account type, and a set of behavioral flags: whether journal items on it may be reconciled, whether it is restricted to one currency, whether it is a trade or a non-trade receivable or payable, and whether it is still in use.

An Account is shared by one or several companies. This is the key structural decision of the model: a single Account record can belong to a parent company and to all of its subsidiaries, and it then carries a **different code in each of them** while remaining the same record, so that a consolidated report can aggregate the journal items of all those companies on one account.

### Lifecycle

1. Created by loading a chart template, by the chart-of-accounts screen, by importing a spreadsheet, or automatically when a bank or cash journal needs a liquidity account.
2. Used: journal items reference it. From the first journal item on, the account can no longer be deleted and can no longer be detached from the company whose items use it.
3. Archived (`active` set to false) when it must disappear from selection lists but its history must stay readable. Archiving is the substitute for deletion.
4. Unmerged: an account shared by several companies can be split into one account per company; the journal items keep the code they had, so nothing changes from an accounting point of view.
5. Deleted only while it carries no journal item, is not referenced by a fiscal position account mapping and is not referenced by a tax distribution line.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. The account name. Indexed for word search. Tracked in the audit trail. |
| `description` | Long text, translatable | Free description shown under the account name in selection lists. |
| `code` | Text, at most 64 characters | The account code **in the currently active company**. Not a plain stored column: it reads and writes `code_store`, which is a per-company value. Required in the sense that a validation refuses an account that has no code in one of the companies it belongs to. Tracked. Only letters, digits and dots are accepted. Sorting of the chart of accounts is by this value. |
| `code_store` | Per-company text | The physical storage of the code: a map from company to code string. The value is looked up and written using the **root company** of the active company, so all companies that share a root share one code. |
| `placeholder_code` | Text, computed, not stored | The display code. Equal to `code` when the account has a code in the active company; otherwise the code in the first company (by identifier order) among the companies the user is allowed to see, followed by that company name in parentheses. Used as the second sort key of the chart of accounts. |
| `active` | Boolean | Default true. False means archived: the account disappears from selection lists and from the chart of accounts unless archived records are explicitly shown. Tracked. |
| `used` | Boolean, computed, not stored | True when at least one journal item references the account. Searchable. |
| `account_type` | Selection | Required, indexed, tracked. Computed with a default rule but editable, and precomputed at creation. See the table of account types below. Determines the internal group, whether the balance is carried forward, whether reconciliation is allowed by default, and the behavior at fiscal year closing. |
| `internal_group` | Selection, computed, not stored | The part of the account type before the first underscore: one of `equity`, `asset`, `liability`, `income`, `expense`, `off`. Searchable by prefix matching on the account type. |
| `include_initial_balance` | Boolean, computed, not stored | True when the internal group is neither income nor expense **and** the account type is not "current year earnings". Tells reports whether to accumulate journal items from the beginning of time instead of from the start of the fiscal year. |
| `reconcile` | Boolean | "Allow Reconciliation". Computed from the account type, stored, editable, precomputed. Tracked. The computation sets it to false for income, expense and equity accounts; true for receivable and payable accounts; false for bank-and-cash, credit-card and off-balance accounts; and leaves it untouched for every other asset or liability type. |
| `currency_id` | Link to Currency | Optional. When set, every journal item on this account must carry this currency as its foreign currency. Tracked. |
| `company_currency_id` | Link to Currency, computed, not stored | The currency of the active company. Used as the currency of the opening amounts and of the current balance. |
| `company_fiscal_country_code` | Text, computed, not stored | The country code of the fiscal country of the active company; used to show or hide country-specific fields. |
| `company_ids` | Multiple links to Company | Required: an account must belong to at least one company. Default: the active company. The read value depends on the acting user. |
| `code_mapping_ids` | Sub-records: Account Code Mapping | One virtual row per company the user may see, exposing the code of this account in that company as an editable cell. Written before `company_ids` so that adding a company and its code in one operation does not momentarily look like a missing code. |
| `tag_ids` | Multiple links to Account Tag | Optional labels for custom reporting. Computed from the code with a default rule (inherit the tags of the closest preceding account by code), stored, editable, precomputed. Tracked. A tag cannot be deleted while accounts still reference it. |
| `group_id` | Link to Account Group, computed, not stored | The most specific group whose prefix range contains the account code in the root company. "Most specific" means the group whose start prefix is the longest; ties are broken by the lower group identifier. |
| `root_id` | Link to Account Root, computed, not stored | The first two characters of the display code. |
| `opening_debit` | Money in company currency, computed with an inverse | The debit total of this account in the company opening entry. Writing it schedules an update of the opening entry. |
| `opening_credit` | Money in company currency, computed with an inverse | The credit total of this account in the company opening entry. |
| `opening_balance` | Money in company currency, computed with an inverse | The net of the two. Writing a positive value sets the debit side and clears the credit side; writing a negative value does the opposite. |
| `current_balance` | Number, computed, not stored | Sum of the balances of all **posted** journal items on this account whose company is the active company or one of its children. |
| `related_taxes_amount` | Integer, computed, not stored | How many taxes of the active company use this account in a distribution line. Used to warn before changing the account. |
| `non_trade` | Boolean | Default false. When true, the account is reported under non-trade receivables or non-trade payables instead of trade receivables or payables. |
| `tax_ids` | Multiple links to Tax | Default taxes proposed when this account is selected on a journal item that has no product-driven taxes. |
| `note` | Long text | Internal notes. Tracked. |
| `display_mapping_tab` | Boolean, not stored | User-interface switch: show the per-company code tab when the user belongs to more than one company. |

### Account types

The account type is the single most important classification. Its value is a two-part string whose first part is the internal group.

| Value | Label | Internal group | Balance carried forward | Reconciliation default |
|---|---|---|---|---|
| `asset_receivable` | Receivable | asset | yes | true (and mandatory) |
| `asset_cash` | Bank and Cash | asset | yes | false |
| `asset_current` | Current Assets | asset | yes | unchanged |
| `asset_non_current` | Non-current Assets | asset | yes | unchanged |
| `asset_prepayments` | Prepayments | asset | yes | unchanged |
| `asset_fixed` | Fixed Assets | asset | yes | unchanged |
| `liability_payable` | Payable | liability | yes | true (and mandatory) |
| `liability_credit_card` | Credit Card | liability | yes | false |
| `liability_current` | Current Liabilities | liability | yes | unchanged |
| `liability_non_current` | Non-current Liabilities | liability | yes | unchanged |
| `equity` | Equity | equity | yes | false |
| `equity_unaffected` | Current Year Earnings | equity | **no** | false |
| `income` | Income | income | no | false |
| `income_other` | Other Income | income | no | false |
| `expense` | Expenses | expense | no | false |
| `expense_other` | Other Expenses | expense | no | false |
| `expense_depreciation` | Depreciation | expense | no | false |
| `expense_direct_cost` | Cost of Revenue | expense | no | false |
| `off_balance` | Off-Balance Sheet | off | yes | false (and forbidden) |

"Reconciliation default" is what the computation of the reconcilable flag produces; "unchanged" means the previous value of the flag is kept, so the user choice survives a change of type between two neutral asset or liability types.

"Balance carried forward" is the value of the derived flag that tells reports whether to start the accumulation at the beginning of time. Income and expense accounts and the current-year-earnings account restart at each fiscal year.

### Default rule for the account type and the tags at creation

When an account is created with a code but without an explicit account type, and likewise when it is created without tags, the value is inherited from the **closest preceding account by code** in the active company:

1. Read every account of the active company with its code and the field being defaulted, sorted by code ascending.
2. Find the insertion position of the new code in that sorted list of codes.
3. Take the entry immediately before that position. Its value becomes the default.
4. If the new code sorts before every existing code, use the fallback: account type "Current Assets" (`asset_current`), or an empty tag list.

Editing the code schedules a recomputation of the account type, so retyping a code in the creation form reproposes a matching type.

### Splitting a name that contains a code

When a name is typed that begins with a token containing at least one digit, and the code is still empty, the leading token is moved into the code and the rest becomes the name. The split uses the rule "the first whitespace-free token that contains at least one digit, then the remainder trimmed". The same rule applies when importing a file in which code and name were typed in one column.

### Choosing a free code

When a code must be invented (duplicating an account, creating a liquidity account for a new journal, generating a chart from a prefix) the algorithm starts from a proposed code and walks forward:

1. If the proposed code is available, use it.
2. Otherwise split the proposed code into a leading part, a trailing digit block and a trailing non-digit part. If a digit block exists, increment it (keeping its width with leading zeros) and try each successive value until the digit block would overflow its width.
3. If that fails, try the proposed code followed by `.copy`, then `.copy2`, `.copy3` … up to `.copy99`.
4. If all fail, the operation is refused with the message "Cannot generate an unused account code."

A code is *available* in a company when no account carrying that code belongs to that company, to any of its parent companies or to any of its child companies.

Worked examples of the walk:

| Starting code | Codes tried in order |
|---|---|
| `102100` | `102101`, `102102`, `102103`, … |
| `1598` | `1599`, `1600`, `1601`, … |
| `10.01.08` | `10.01.09`, `10.01.10`, `10.01.11`, … |
| `10.01.97` | `10.01.98`, `10.01.99`, `10.01.97.copy2`, `10.01.97.copy3`, … |
| `1021A` | `1021A` (if free), `1022A`, `1023A`, … |
| `hello` | `hello.copy`, `hello.copy2`, `hello.copy3`, … |
| `9998` | `9999`, `9998.copy`, `9998.copy2`, … |

### Uniqueness

Codes are unique **within a company hierarchy**: two accounts may not carry the same code if one belongs to a company that is a parent or a child of a company of the other. Two unrelated companies may reuse the same code on different accounts. The check runs after creation, after any write that touches the companies, the code or the code mapping rows, and after a bulk import.

### Display name

- When the acting user has the read-only accounting group or better and the account has a code: the code, a space, then the name.
- Otherwise: the name alone.
- In the "rich" display mode used by drop-downs: the code (omitted for users without the accounting read group), the name, the word "Suggested" in back-ticks when the account is among the frequent accounts for the partner and document type of the current context, and the description on a second line between double dashes.

### Search behavior

Typing in an account field searches the code by prefix, the name by the chosen operator and the description by substring, joined by "or". When the context carries a document type:

- Without a search term and with a known partner, the drop-down proposes the accounts most frequently used with that partner, most frequent first.
- With a search term containing a digit, no type restriction is applied (the user is typing a code).
- With a search term containing no digit, the candidates are restricted to income accounts for a sale document, and to expense, fixed-asset and cost-of-revenue accounts for a purchase document.

The "most frequently used" ranking counts the journal items of the last 730 days for that partner in that company, restricted to the allowed types, grouped by account, ordered by descending count and then by ascending code.

### Multi-company behavior

- The code, the group, the root, the current balance, the opening amounts, the company currency and the related tax count all depend on the active company.
- An account of type "Bank and Cash" may belong to **one company only**.
- A company can be removed from the list of companies of an account only while no journal item of that company (or of a descendant) uses the account.
- Unmerging an account creates one copy per company, transfers every reference that belongs to that company (ordinary links, multi-value links, polymorphic references and per-company values) to the copy, keeps the first company on the original record, and writes a note in the audit trail of each copy saying that it was split off from the original account of the original company.

### Archival

Setting `active` to false hides the account. Archiving is refused for an account referenced by a fiscal position account mapping or by a tax distribution line, with the messages given in `business-rules.md`.

---

## 2. Account Group

Account Group (`account.group`, table `account_group`).

### Purpose

A named range of account-code prefixes, used to build the hierarchy shown in the chart of accounts and in the hierarchical mode of the financial reports. A group with prefixes from `40` to `41` contains every account whose code starts with `40` or `41`.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. |
| `code_prefix_start` | Text | The first prefix of the range. Computed from the end prefix, stored, editable, precomputed: when it is empty, or greater than the end prefix, it is set to the end prefix. |
| `code_prefix_end` | Text | The last prefix of the range. Computed from the start prefix, stored, editable, precomputed: when it is empty, or smaller than the start prefix, it is set to the start prefix. |
| `parent_id` | Link to Account Group | Read-only, maintained automatically (see below). Deleting a parent deletes its children by cascade. |
| `company_id` | Link to Company | Required, read-only, default the **root** of the active company. |

### Constraints

- A database check enforces that the start prefix and the end prefix have the same number of characters. Message: "The length of the starting and the ending code prefix must be the same".
- Two groups of the same company whose prefixes have the same length may not overlap. Message: "Account Groups with the same granularity can't overlap".
- The parent chain may not form a cycle. Message: "You cannot create recursive groups."

### Automatic parenting

After any creation and after any change of a prefix, the hierarchy is rebuilt for the affected companies:

1. For each group, consider every other group of the same company whose start prefix is **strictly shorter**, whose start prefix is less than or equal to the same-length beginning of the child start prefix, and whose end prefix is greater than or equal to the same-length beginning of the child end prefix.
2. Among those candidates, the parent is the one with the **longest** start prefix.
3. Groups with no candidate have no parent.

The rebuild is skipped while a chart template is being loaded (it is run once at the end instead).

### Display name

The start prefix; then, when the end prefix differs, a hyphen and the end prefix; then a space and the name. Example: `40-41 Suppliers`.

### Deletion

Deleting a group re-parents its children to the parent of the deleted group before removing it, so the hierarchy stays connected.

---

## 3. Account Root

Account Root (`account.root`) is not stored. It is a computed facet whose identifier **is** a string: the first characters of an account code.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, computed | Equal to the identifier, that is the prefix string itself. |
| `parent_id` | Link to Account Root, computed | The same prefix with its last character removed, or nothing when the prefix is a single character. |

An account is mapped to the root formed by the **first two characters** of its display code. Searching is restricted to two forms: "the root is one of these identifiers" and "the root is a prefix of one of these identifiers"; any other search is refused with the message "Filter on the Account or its Display Name instead". A search by root on accounts is translated into a prefix match on the display code: an exact root matches codes starting with that prefix, and a parent root matches codes starting with that prefix followed by anything.

---

## 4. Account Tag

Account Tag (`account.account.tag`, table `account_account_tag`).

### Purpose

A free label. Three applicabilities exist: tags on accounts (custom reporting), tags on taxes (the tax grids used by the tax report) and tags on products. This domain owns the first; the second is specified in `../taxes/`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. |
| `applicability` | Selection | Required, default `accounts`. Values: `accounts` (Accounts), `taxes` (Taxes), `products` (Products). |
| `color` | Integer | Colour index for the badge. |
| `active` | Boolean | Default true. Archiving hides the tag without removing it. |
| `country_id` | Link to Country | The country for which the tag exists when it is applied on taxes. |
| `report_expression_id` | Link to Report Expression, computed, not stored | The report expression whose formula equals this tag name, ignoring a leading minus sign. |
| `balance_negate` | Boolean, computed, not stored | True when that expression formula starts with a minus sign, meaning the report shows the negated balance. |

### Uniqueness

The triple (name, applicability, country) is unique. Message: "A tag with the same name and applicability already exists in this country."

### Display name

Normally the name. When the company has at least one foreign tax registration and the tag applies to taxes and its country differs from the fiscal country of the company, the name is followed by the country code in parentheses.

### Deletion

Three tags shipped as reference data — the operating, financing and investing cash-flow tags — cannot be deleted while they are referenced by the chart of account definition. Message: "You cannot delete this account tag (*the tag name*), it is used on the chart of account definition."

---

## 5. Account Code Mapping

Account Code Mapping (`account.code.mapping`) is not stored either. It exists only to render, in the account form, one editable row per company showing the code of that account in that company.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `account_id` | Link to Account, computed | Derived from the synthetic identifier. |
| `company_id` | Link to Company, computed | Derived from the synthetic identifier. |
| `code` | Text, computed with an inverse | The code of the account in that company. Writing it writes the code of the account with that company active. |

The synthetic identifier is built as the account identifier multiplied by ten thousand plus the company identifier; the two parts are recovered by integer division and remainder. Rows exist only for the companies the acting user may see, ordered by the company sequence then the company name. Reading the model without restricting it to specific accounts is refused with the message "Account Code Mapping cannot be accessed directly. It is designed to be used only through the Chart of Accounts."

---

## 6. Journal

Journal (`account.journal`, table `account_journal`).

### Purpose

A book of entries. Every Journal Entry belongs to exactly one Journal. The Journal determines the numbering prefix of its entries, the default account proposed on their lines, the accounts used for liquidity and for unidentified bank transactions, whether entries are secured with a hash chain, which payment methods are available, and which incoming electronic-mail address creates documents.

### Lifecycle

1. Created by loading a chart template, by the journal screen, or automatically when a bank account is configured.
2. Used by entries. From the first entry on, the company of the journal can no longer change.
3. Archived when obsolete. Archiving is refused while the journal still holds draft entries.
4. Deleted only when nothing references it; deleting a journal deletes its payment method lines and, when no other journal uses it, the bank account record it pointed to.

### Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. The journal name. |
| `name_placeholder` | Text, computed, not stored | The suggested name shown in the empty name box: the default name of the type followed, in parentheses, by the trailing number of the code (or `1`). The default names are Customer Invoices, Vendor Bills, Cash, Bank, Credit Card and Miscellaneous Operations. When no type is chosen yet, the placeholder is "Select a type". |
| `code` | Text, at most 5 characters | Required. Labelled "Sequence Prefix". Computed with a default rule, stored, editable, precomputed. Default: the type prefix (`INV`, `BILL`, `CSH`, `BNK`, `CCD`, `MISC`) followed by the smallest integer from 1 to 99 that yields a code not already used in the company. |
| `active` | Boolean | Default true. |
| `type` | Selection | Required. `sale` (Sales), `purchase` (Purchase), `cash` (Cash), `bank` (Bank), `credit` (Credit Card), `general` (Miscellaneous). |
| `sequence` | Integer | Default 10. Orders journals on the dashboard and in lists. |
| `company_id` | Link to Company | Required, read-only after creation, indexed, default the active company. |
| `currency_id` | Link to Currency | Optional. When set and different from the company currency, the journal works in that foreign currency: its liquidity account is forced to the same currency, its balance figures are expressed in it, and the journal display name carries the currency name in parentheses. |
| `country_code` | Text, related | The code of the fiscal country of the company. |
| `default_account_id` | Link to Account | The account proposed on lines of entries of this journal and, for liquidity journals, the account that holds the money. Restricted to the account types listed below per journal type. Deleting the account is refused while a journal points to it. Not copied when the journal is duplicated. |
| `default_account_type` | Text, computed, not stored | The account-type pattern used to filter the default account: `asset_cash` for bank and cash, `liability_credit_card` for credit card, `income%` for sale, `expense%` for purchase, `%` otherwise. |
| `suspense_account_id` | Link to Account | Computed, stored, editable. For bank, cash and credit-card journals: the account on which a bank transaction is parked until it is matched with a business document. Restricted to current-asset accounts. The computation sets it to the previous value when one exists, otherwise to the company journal-suspense account, otherwise to nothing; for other journal types it is cleared. |
| `profit_account_id` | Link to Account | Used to book a cash surplus when the counted cash exceeds the computed balance. Restricted to income and other-income accounts. |
| `loss_account_id` | Link to Account | Used to book a cash shortage. Restricted to expense accounts. |
| `non_deductible_account_id` | Link to Account | Account used to register the private (non-deductible) share of mixed expenses. |
| `restrict_mode_hash_table` | Boolean | "Secure Posted Entries with Hash". When true, posting an entry retroactively hashes every entry of the same numbering chain from the new entry back to the last already hashed one. |
| `refund_sequence` | Boolean | "Dedicated Credit Note Sequence". Computed from the type, stored, editable: true for sale and purchase journals. When true, credit notes get their own numbering chain, distinct from invoices. |
| `payment_sequence` | Boolean | "Dedicated Payment Sequence". Computed from the type, stored, editable, precomputed: true for bank, cash and credit-card journals. When true, payments and bank transactions posted in the journal get their own numbering chain. |
| `sequence_override_regex` | Long text | An explicit pattern that overrides the automatic understanding of the numbering format. It may define the named parts `prefix1`, `year`, `prefix2`, `month`, `prefix3`, `seq` and `suffix`. Used when a legally imposed number would otherwise be misread. |
| `invoice_reference_type` | Selection | Required, default `invoice`. `partner` (Based on Customer) or `invoice` (Based on Invoice). Which subject the structured payment reference identifies. |
| `invoice_reference_model` | Selection | Required. `odoo` (Full Reference), `euro` (European structured reference), `number` (Numbers only). Default: the first value whose name begins with the lowercase country code of the company, otherwise the full-reference model. |
| `bank_account_id` | Link to Bank Account | The bank account of the company behind a bank journal. Restricted to bank accounts whose holder is the company partner. Indexed when set. Not copied. Deleting the bank account is refused while a journal points to it. |
| `bank_acc_number` | Text, related and writable | The account number of that bank account; writing it on a bank journal without a bank account creates one. |
| `bank_id` | Link to Bank, related and writable | The bank of that bank account. |
| `bank_statements_source` | Selection | Default `undefined` ("Undefined Yet"). How bank statements reach the journal; other values are contributed by connector packages. |
| `company_partner_id` | Link to Partner, related, not stored | The partner record of the company; used to constrain the bank account choice. |
| `inbound_payment_method_line_ids` | Sub-records: Payment Method Line | Money-in methods of a liquidity journal, each with its outstanding-receipts account. Computed from the type and the currency, stored, editable. Not copied. The computation replaces the whole list with one line per default inbound method (the manual method) whenever the type or currency changes, preserving the previously chosen account when its currency still matches. |
| `outbound_payment_method_line_ids` | Sub-records: Payment Method Line | Money-out methods, each with its outstanding-payments account. Same rules. |
| `available_payment_method_ids` | Multiple links to Payment Method, computed, not stored | The methods that may still be added, given the multiplicity rule of each method (see below). |
| `selected_payment_method_codes` | Text, computed, not stored | The codes of the chosen methods, joined by commas and surrounded by commas, used to show or hide method-specific fields. |
| `journal_group_ids` | Multiple links to Journal Group | The ledger groups this journal participates in. |
| `alias_name` | Text | The local part of the electronic-mail address that creates documents in this journal. Only sale and purchase journals have one. |
| `incoming_einvoice_notification_email` | Text | Semicolon-separated addresses that receive a copy of every sent and received invoice of this journal. |
| `accounting_date` | Date, computed, not stored | The accounting date an entry of this journal would receive today (or at the date given in the context), after applying the lock-date shifting rule. |
| `has_invalid_statements` | Boolean, computed, not stored | True when the journal holds a bank statement that is not valid or not complete. |
| `invoice_template_pdf_report_id` | Link to Report | Which printable invoice template this journal uses. |
| `available_invoice_template_pdf_report_ids` | Sub-records: Report, computed | The templates that may be chosen. |
| `is_self_billing` | Boolean | The journal is for self-billing documents; invoices then use a separate numbering chain per partner. |
| `show_fetch_in_einvoices_button`, `show_refresh_out_einvoices_status_button` | Booleans, computed | Whether the electronic-invoice fetch buttons are shown; always false in the core and switched on by exchange packages. |

### Allowed account types for the default account

| Journal type | Allowed account types for the default account |
|---|---|
| `bank` | Bank and Cash, Credit Card |
| `credit` | Credit Card |
| `cash` | Bank and Cash |
| `sale` | Income, Other Income |
| `purchase` | Expenses, Depreciation, Cost of Revenue |
| `general` | every type except Off-Balance Sheet is offered; the list is the full set of business types |

### Uniqueness

The pair (company, code) is unique. Message: "Journal codes must be unique per company."

### Payment method multiplicity

Each payment method declares a mode:

| Mode | Rule |
|---|---|
| `unique` | The method may be attached to **one journal per company**. |
| `electronic` | The method may be attached to one journal per company **and per provider**. |
| `multi` | The method may be attached to any number of journals and repeated on a journal. |

Two lines of the same journal and the same direction may not carry the same method **and** the same name when the method is unique or electronic. Message: "You can't have two payment method lines of the same payment type (*inbound* or *outbound*) and with the same name (*the line name*) on a single journal." When the uniqueness across journals is broken the message is "Some payment methods supposed to be unique already exists somewhere else." followed by the offending method names in parentheses.

### Automatic completion at creation

When a journal is created, missing values are filled before the record is written:

1. If the type is missing and the creation comes from a file import, the type becomes `general`.
2. The company defaults to the active company.
3. For a bank or cash journal: the name defaults to the bank account number, then to the placeholder; a liquidity account is created when none is given; the profit and loss accounts default to the company cash-difference income and expense accounts.
4. For a purchase journal: the default account falls back to the company-wide default expense account of product categories.
5. For a sale journal: the default account falls back to the company-wide default income account of product categories.
6. For a credit-card journal: the default account is the first credit-card account of the company, or a newly created one.
7. On import without a code, the code is the first five characters of the name; if that collides, the next free type-based code is taken. If none can be found the creation fails with "Cannot generate an unused journal code. Please change the name for journal *the journal name*."
8. For sale and purchase journals, an incoming-mail local part is prepared and made unique.

### Creating the liquidity account of a new bank, cash or credit-card journal

1. Read any existing account of the company to learn how many characters codes have; use six when there is none.
2. Take the company bank-account code prefix for a bank or credit-card journal; for a cash journal take the cash prefix, falling back to the bank prefix; otherwise use an empty prefix.
3. Pad the prefix with trailing zeros up to the code length; that is the starting code.
4. Find the first free code from that starting point with the walk described for accounts.
5. Create an account with the journal name, that code, type "Bank and Cash" (or "Credit Card" for a credit-card journal), the journal currency, and the company.

### Duplication

Duplicating a journal invents a free code by stripping the digits from the original code and appending the smallest integer that is not already used in the company, truncated to five characters; the name becomes the original name followed by "(copy)". When no code can be found the copy fails with "Could not compute any code for the copy automatically. Please create it manually."

### Display name

The journal name; when the journal has a foreign currency different from the company currency, the currency name is appended in parentheses.

### Ordering

By the sequence number, then the type, then the code.

### Multi-company behavior

A journal belongs to exactly one company and cannot change company once entries exist. Accounts selected on a journal must be visible to the company of the journal or to one of its parents.

---

## 7. Journal Group

Journal Group (`account.journal.group`, table `account_journal_group`).

A named selection of journals, expressed **by exclusion**: the group contains every journal of the company except those listed. It is offered as a filter called "Ledger" in the reports and in the journal-item list.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text, translatable | Required. Labelled "Ledger group". |
| `company_id` | Link to Company | Default the active company. When empty the group is available to every company. |
| `excluded_journal_ids` | Multiple links to Journal | The journals **not** in the group. Archived journals are included in the choice list. |
| `sequence` | Integer | Default 10. Display order. |

Uniqueness: the pair (company, name). Message: "A Ledger group name must be unique per company."

Filtering an entry or a journal item by a ledger group is translated into "the journal is not one of the excluded journals of that group".

---

## 8. Journal Entry

Journal Entry (`account.move`, table `account_move`).

### Purpose

One accounting document. It holds a header (journal, date, number, state, partner, currency, reference) and a set of Journal Items that must sum to zero in the company currency. Every accounting document of the whole application is a Journal Entry: a plain miscellaneous entry, a customer invoice, a vendor bill, a credit note, a receipt, the entry of a payment, the entry of a bank transaction, an exchange-difference entry, a cash-basis tax entry, an opening entry.

The document type (`move_type`) decides which additional behavior applies. This document specifies the entity as a whole and the behavior of a plain entry; the invoice-specific behavior is specified in `../accounts-receivable/` and `../accounts-payable/`.

### Document types

| Value | Label | Family | Direction | Reverse type |
|---|---|---|---|---|
| `entry` | Journal Entry | miscellaneous | outbound (sign +1) | `entry` |
| `out_invoice` | Customer Invoice | sale | inbound (sign −1) | `out_refund` |
| `out_refund` | Customer Credit Note | sale | outbound (sign +1) | `out_invoice` |
| `in_invoice` | Vendor Bill | purchase | outbound (sign +1) | `in_refund` |
| `in_refund` | Vendor Credit Note | purchase | inbound (sign −1) | `in_invoice` |
| `out_receipt` | Sales Receipt | sale | inbound (sign −1) | `out_refund` |
| `in_receipt` | Purchase Receipt | purchase | outbound (sign +1) | `in_refund` |

The direction sign is 1 for a plain entry and for outbound documents, and −1 for inbound documents. Inbound documents are those that bring money in (customer invoices and sales receipts) or that reduce a payable (vendor credit notes). The sign converts a price into a balance and back.

### Accounting fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `name` | Text | The document number. Computed (see the numbering rules), stored, editable, precomputed. Not copied. Tracked. Indexed for word search. The placeholder value `/` means "no number yet". |
| `name_placeholder` | Text, computed, not stored | The number the document *would* get: shown in the empty number box when the document has no number, has a date, and is the first of its numbering chain. Computed by taking the next sequence format and incrementing the counter by one. |
| `ref` | Text | A free reference. Not copied. Tracked. Indexed for word search. |
| `date` | Date | Required, indexed. The **accounting date**: the date at which the entry enters the ledger and the date used by every report. Computed, stored, editable, precomputed. Not copied. Tracked. |
| `state` | Selection | Required, read-only, default `draft`. Values `draft` (Draft), `posted` (Posted), `cancel` (Cancelled). Not copied. Tracked. |
| `move_type` | Selection | Required, read-only after creation, indexed, default `entry`. Tracked. |
| `is_storno` | Boolean, computed, not stored | True when the entry is a refund and the company uses storno accounting, or when it was explicitly flagged. Under storno accounting a reversal is booked as a negative amount on the same side instead of an amount on the opposite side. |
| `journal_id` | Link to Journal | Required. Computed, stored, editable, precomputed, with an inverse. Restricted to the journals suitable for the document type. |
| `journal_group_id` | Link to Journal Group, not stored | A search-only field: filtering by a ledger group filters on "journal not in the excluded journals of the group". |
| `company_id` | Link to Company | Computed from the journal, stored, editable, precomputed, indexed, with an inverse. When the journal company is not among the ancestors of the current company, the entry takes the first accessible branch of the journal company (or of the active company). |
| `company_currency_id` | Link to Currency, related | The currency of the company; the currency in which the balance invariant is checked. |
| `currency_id` | Link to Currency | Required. The document currency. Computed, stored, editable, precomputed, with an inverse. Tracked. |
| `line_ids` | Sub-records: Journal Item | Every item of the entry, including tax lines, payment-term lines, rounding lines and display-only lines. Copied when the entry is duplicated. |
| `invoice_line_ids` | Sub-records: Journal Item | A view over `line_ids` restricted to product lines, sections, subsections and notes. Not copied. |
| `posted_before` | Boolean | Not copied. True once the entry has been posted at least once. Governs the numbering reset rule and the audit-trail deletion rule. |
| `checked` | Boolean | "Reviewed". Computed, stored, editable, tracked, not copied. The computation sets it to true when the entry is posted **and** either the journal is a miscellaneous journal or the acting user is allowed to review. |
| `always_tax_exigible` | Boolean | Computed, stored, editable. True when the entry is not an invoice-like document **and** it collects no cash-basis values, meaning its tax lines are immediately reportable. |
| `auto_post` | Selection | Required, default `no`, not copied. Values: `no` (No), `at_date` (At Date), `monthly` (Monthly), `quarterly` (Quarterly), `yearly` (Yearly). A value other than "No" means the entry is posted automatically by a scheduled job at its accounting date; the three periodic values additionally create the next occurrence. |
| `auto_post_until` | Date | Not copied. Computed, stored, editable: cleared whenever the automatic posting is "No" or "At Date". The last date up to and including which the recurrence produces new entries. |
| `auto_post_origin_id` | Link to Journal Entry | Read-only, not copied, indexed when set. The first entry of a recurrence; the original entry points to itself. |
| `hide_post_button` | Boolean, computed, not stored | True when the entry is not draft, or when it is scheduled for automatic posting and its date is in the future. |
| `made_sequence_gap` | Boolean, stored | True when this entry is the first one that breaks the natural numbering of its chain. Maintained by the gap-detection algorithm, not by an ordinary computation. |
| `highest_name` | Text, computed, not stored | The last number used in the numbering chain this entry belongs to. |
| `sequence_prefix` | Text, computed, stored | The part of the number before the trailing digit block. |
| `sequence_number` | Integer, computed, stored | The trailing digit block of the number, as an integer; zero when there is none. |
| `type_name` | Text, computed, not stored | The human label of the document type, with two overrides: a customer invoice is called "Invoice" and a customer credit note is called "Credit Note". |
| `restrict_mode_hash_table` | Boolean, related to the journal | Whether the journal secures posted entries with a hash. |
| `inalterable_hash` | Text, read-only, not copied, indexed when set | The hash of this entry in the chain (see the hash section of `calculations.md`). |
| `secure_sequence_number` | Integer, read-only, not copied, indexed | The position of the entry in a legacy gapless securing sequence; entries hashed through the current mechanism have no value here and are ordered by prefix and number instead. |
| `secured` | Boolean, computed, not stored | True when the entry carries a hash. Searchable only with the test "is true". |
| `show_reset_to_draft_button` | Boolean, computed, not stored | True when the entry is not restricted by hashing, carries no hash, and is either cancelled or posted without a pending cancellation request. |
| `need_cancel_request` | Boolean, computed, not stored | False in the core. A country package sets it to true for documents already declared to an authority, which then require an approved cancellation instead of a reset to draft. |
| `audit_trail_message_ids` | Sub-records: Message | Every notification message logged on this entry; the audit trail. |
| `attachment_ids` | Sub-records: Attachment | Files attached to the entry. |
| `no_followup` | Boolean, computed with an inverse | Excludes the entry from dunning reports. For an invoice it reads and writes the flag of the first receivable or payable line; for anything else it is true. |
| `partner_id` | Link to Partner | Optional on a plain entry. Tracked, indexed. Deleting the partner is refused while an entry points to it. Writing it recomputes the account of the payment-term lines. |
| `commercial_partner_id` | Link to Partner, computed, stored, read-only | The commercial entity of the partner: the top-most company in the partner hierarchy. Used for the payable and receivable accounts and for grouping. |
| `payment_reference` | Text | The communication the payer should quote. Computed, stored, editable, with an inverse, tracked, not copied. The computation fills it, for posted customer invoices only, with the structured reference derived from the journal settings. |
| `sanitize_payment_reference` | Text, computed, not stored | The payment reference stripped of every character that is not a letter or a digit. A functional index exists on the same expression for matching bank transactions. |
| `narration` | Rich text | Terms and conditions. Computed from the company settings, stored, editable. |
| `invoice_origin` | Text | Read-only, tracked, not copied. The document or documents that generated this one. |
| `reversed_entry_id` | Link to Journal Entry | Read-only, not copied, indexed when set. The entry this one reverses. |
| `reversal_move_ids` | Sub-records: Journal Entry | The reversals of this entry. |
| `origin_payment_id` | Link to Payment | Indexed when set, not copied. The payment whose journal entry this is. |
| `matched_payment_ids` | Multiple links to Payment | The payments attached to this invoice. |
| `statement_line_id` | Link to Statement Line | Indexed when set, not copied. The bank transaction whose journal entry this is. |
| `tax_cash_basis_rec_id` | Link to Partial Reconciliation | Indexed when set. The partial reconciliation that produced this cash-basis entry. |
| `tax_cash_basis_origin_move_id` | Link to Journal Entry | Read-only, indexed when set. The document whose taxes this cash-basis entry recognises. |
| `tax_cash_basis_created_move_ids` | Sub-records: Journal Entry | The cash-basis entries created from this document. |
| `exchange_diff_partial_ids` | Sub-records: Partial Reconciliation | The reconciliations that produced this exchange-difference entry. |
| `adjusting_entry_origin_move_ids` / `adjusting_entries_move_ids` | Multiple links to Journal Entry | The two directions of the link created by the automatic transfer wizard between an origin entry and the adjusting entry it produced. |

### Amount fields

All of these are computed and stored. They are recomputed whenever a line balance, a line foreign amount, a line residual, a reconciliation or the state changes.

| Field (storage name) | Currency | Meaning |
|---|---|---|
| `amount_untaxed` | document | Signed untaxed total in the document currency |
| `amount_tax` | document | Signed tax total in the document currency |
| `amount_total` | document | Signed grand total in the document currency |
| `amount_residual` | document | Signed amount still due in the document currency |
| `amount_untaxed_signed` | company | Untaxed total in the company currency, with the ledger sign |
| `amount_untaxed_in_currency_signed` | document | Untaxed total in the document currency, with the ledger sign |
| `amount_tax_signed` | company | Tax total in the company currency, with the ledger sign |
| `amount_total_signed` | company | Grand total in the company currency, with the ledger sign; for a plain entry the absolute value |
| `amount_total_in_currency_signed` | document | Grand total in the document currency, with the ledger sign; for a plain entry the absolute value |
| `amount_residual_signed` | company | Amount still due in the company currency, with the ledger sign |
| `amount_total_words` | — | The grand total spelled out in words in the language of the document |

The exact formulas and the sign conventions are in `calculations.md`.

### Payment status

| Value | Label | Meaning |
|---|---|---|
| `not_paid` | Not Paid | Nothing has been matched against the document |
| `in_payment` | In Payment | The document is fully matched but at least one counterpart payment is not itself matched with a bank transaction |
| `paid` | Paid | The document is fully matched and every counterpart payment is matched with a bank transaction |
| `partial` | Partially Paid | Some but not all of the document has been matched |
| `reversed` | Reversed | The document is fully matched and the only counterparts are its own reversals |
| `blocked` | Blocked | The user marked the document as not to be chased; set and cleared manually |
| `invoicing_legacy` | Invoicing App Legacy | A value kept for documents created by a stand-alone invoicing installation; never produced by this domain |

`status_in_payment` merges this with the document state for display: for a posted document it shows the payment status when it is partial, in payment, paid, reversed or blocked, otherwise "Sent" when the document was sent and otherwise the state; for a draft document it shows the payment status when it is partial, in payment, paid or blocked, otherwise the state.

### Date fields

| Field (storage name) | Meaning |
|---|---|
| `date` | The accounting date; see above |
| `invoice_date` | The document date printed on an invoice or read from a bill. Indexed, not copied |
| `invoice_date_due` | The due date; computed from the payment terms, stored, editable, indexed, not copied |
| `delivery_date` | The date the goods or services were delivered; computed, stored, editable, precomputed, not copied |
| `taxable_supply_date` | The date used for the tax point where a country requires one distinct from the document date |

The **accounting date source** is the document date when there is one, otherwise the accounting date itself. The computation of the accounting date is:

1. Take the accounting date source.
2. When there is none, or the document is not an invoice-like document: if no accounting date exists yet, set it to today, and stop.
3. When the document is **not** a sale document, shift the source through the accounting-date rule described in `calculations.md` (which pushes it out of any locked period and to the end of the numbering period when it lies in the past).
4. If the resulting date differs from the current accounting date, write it, and schedule the recomputation of the item dates and of the number.

### Uniqueness and indexes

- A unique index enforces that the pair (number, journal) is unique **among posted entries whose number is not the placeholder `/`**. Message: "Another entry with the same name already exists."
- Additional indexes exist on (journal, date), on (journal, company, date), on (journal, state, payment status, type, date), on the journal for unreviewed entries, on the reference for vendor documents, and on the sanitized payment reference.

### Ordering

Descending accounting date, then descending number, then descending document date, then descending identifier. In other words the newest entries first, and within a day the highest number first.

### Display name

- A draft entry shows "Draft" followed by the type name, then the reference in parentheses or the word "Unknown" when it has no number.
- A cancelled entry shows the number followed by "(Cancelled)".
- A posted entry shows the number.
- When the display is requested in "full" mode the partner name and the date are appended.

### Copying

Duplicating an entry:

- For a customer invoice or a vendor bill, only the *created* line commands are kept, so the copy is rebuilt from scratch rather than linking the original lines.
- For a plain entry, the partner is cleared unless the copy is a cancelling reversal.
- If the requested date, or the date of the original, falls on or before the fiscal lock date that applies to the user and the journal, the date of the copy is moved to the day after that lock date.
- The journal is not copied when the original journal is archived.
- A message is logged on the copy: "This entry has been duplicated from *link to the original*", or "This entry has been reversed from *link to the original*" when the copy is a reversal, followed by "This recurring entry originated from *link*" when the copy belongs to a recurrence.

### Archival

Journal Entries have no archived state. The equivalents are the cancelled state and the reversal.

---
