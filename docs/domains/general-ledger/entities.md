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

## 9. Journal Item

Journal Item (`account.move.line`, table `account_move_line`).

### Purpose

One posting line of a Journal Entry: an account, an amount on the debit or the credit side expressed in the company currency, and optionally the same amount expressed in a foreign currency. A Journal Item is the atom of the ledger: every report, every reconciliation and every balance is an aggregation of Journal Items.

Some items are not postings at all: items whose display type is a section, a subsection or a note carry no account and no amount and exist only to structure the printed document.

### Lifecycle

A Journal Item exists only inside a Journal Entry and is deleted with it. While the entry is draft the item can be freely created, modified and deleted. Once the entry is posted:

- Deleting an item with a non-zero amount is refused.
- Changing the taxes is refused.
- Changing any amount, account, currency, partner or date is checked against the lock dates and breaks any reconciliation the item takes part in.
- Any change is written into the audit trail of the entry.

### Parent and context fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `move_id` | Link to Journal Entry | Required, read-only, indexed. Deleted with the entry. |
| `journal_id` | Link to Journal, related to the entry, stored, indexed | Not copied. Denormalised for reporting. |
| `company_id` | Link to Company, related, stored, read-only, indexed | Denormalised. |
| `company_currency_id` | Link to Currency, related, stored | The currency in which the debit, credit and balance are expressed. |
| `move_name` | Text, related to the entry number, stored, indexed | Denormalised. |
| `parent_state` | Selection, related to the entry state, stored | Lets reports and searches filter posted items without joining. |
| `date` | Date, related to the entry date, stored, not copied | Aggregated with a minimum when grouped. |
| `invoice_date` | Date, related, stored, not copied | Aggregated with a minimum. |
| `ref` | Text, related to the entry reference, stored, not copied, word-indexed | |
| `move_type` | Selection, related | |
| `sequence` | Integer | The display order inside the entry. Computed from the display type, stored, editable, precomputed: 10 000 for a tax line, 11 000 for a rounding line, 12 000 for a payment-term line, 100 for everything else. |
| `journal_group_id` | Link to Journal Group, not stored | Search-only, same rule as on the entry. |

### Accounting fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `account_id` | Link to Account | Required for every item that is not a section, subsection or note (enforced by a database check). Computed, stored, editable, precomputed, with an inverse. Tracked. Deleting the account is refused while items exist. Off-balance accounts are excluded from the selection list. |
| `account_name`, `account_code` | Text, related | Denormalised for report configuration. |
| `name` | Text | The item label. Computed from the product and the entry reference, stored, editable, precomputed. Tracked. |
| `debit` | Money in company currency | Computed from the balance, stored, precomputed, with an inverse. Equal to the balance when it is positive, zero otherwise (reversed under storno accounting). |
| `credit` | Money in company currency | Computed from the balance, stored, precomputed, with an inverse. Equal to minus the balance when it is negative, zero otherwise (reversed under storno accounting). |
| `balance` | Money in company currency | Computed, stored, editable, precomputed. Tracked. The signed amount: debit minus credit. This is the field actually written; debit and credit are derived from it. |
| `amount_currency` | Money in the item currency | Computed from the balance and the rate, stored, editable, precomputed, with an inverse. The same economic amount expressed in the item currency. |
| `currency_id` | Link to Currency | Required. Computed, stored, editable, precomputed. The company currency for a cost-of-goods-sold line; the document currency for an invoice-like document; otherwise the previous value or the company currency. |
| `currency_rate` | Number, computed, not stored | The rate from the company currency to the item currency at the relevant date. |
| `is_same_currency` | Boolean, computed, not stored | True when the item currency equals the company currency. |
| `cumulated_balance` | Money, computed, not stored | The running total of the balance over the current list ordering and filter. Only computed when the list view requests it. |
| `partner_id` | Link to Partner | Computed from the entry partner (its commercial entity), stored, editable, precomputed, with an inverse. Deleting the partner is refused while items exist. |
| `date_maturity` | Date | The due date of a receivable or payable item. Indexed, tracked. |
| `is_storno` | Boolean | Computed, stored, editable, precomputed. Marks an item booked as a negative amount on its natural side. |
| `display_type` | Selection | Required. Computed, stored, editable, precomputed. See the table below. |
| `quantity` | Number | The optional quantity of the line. Computed, stored, editable, precomputed: one for a product line, nothing otherwise. |
| `product_id`, `product_uom_id`, `price_unit`, `discount`, `price_subtotal`, `price_total`, `deductible_amount` | various | Commercial fields of an invoice line; specified in `../accounts-receivable/` and `../accounts-payable/`. |
| `is_imported` | Boolean | The line was captured automatically (import, decoding of a received document) rather than typed. Relaxes the archived-account check. |

### Display types

| Value | Label | Accountable | Meaning |
|---|---|---|---|
| `product` | Product | yes | An ordinary line: goods, services or a free posting |
| `cogs` | Cost of Goods Sold | yes | A cost line added by inventory valuation |
| `tax` | Tax | yes | A line produced by the tax engine |
| `discount` | Discount | yes | A line carrying a separately booked discount |
| `rounding` | Rounding | yes | A cash-rounding line |
| `payment_term` | Payment Term | yes | A receivable or payable instalment line |
| `epd` | Early Payment Discount | yes | A line produced by an early-payment discount |
| `non_deductible_product_total`, `non_deductible_product`, `non_deductible_tax` | Non Deductible … | yes | Lines isolating the private share of a mixed expense |
| `line_section` | Section | no | A heading in the printed document |
| `line_subsection` | Subsection | no | A second-level heading |
| `line_note` | Note | no | A free text row |

The three non-accountable types are the only ones for which the account, the debit, the credit and the foreign amount must all be empty or zero; a database check enforces this in both directions.

### Tax fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `tax_ids` | Multiple links to Tax | The taxes that apply **to** this line (a base line). Computed, stored, editable, precomputed. Tracked. |
| `tax_line_id` | Link to Tax, related through the distribution line, stored | Set on a tax line; identifies the tax that produced it. |
| `tax_repartition_line_id` | Link to Tax Distribution Line | Read-only. The distribution line that produced this tax line. |
| `tax_group_id` | Link to Tax Group, related, stored | The group of the originating tax. |
| `group_tax_id` | Link to Tax, indexed when set | The group of taxes this line originates from, when the tax was a group. |
| `tax_base_amount` | Money in company currency, read-only | The base on which this tax line was computed. |
| `tax_tag_ids` | Multiple links to Account Tag | The tax grids this item feeds. Tracked. A tag cannot be deleted while items reference it. |
| `extra_tax_data` | Structured data | Internal data of the tax engine for this line. |

### Reconciliation fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `amount_residual` | Money in company currency, computed, stored | What is left to match, in the company currency. |
| `amount_residual_currency` | Money in the item currency, computed, stored | What is left to match, in the item currency. |
| `reconciled` | Boolean, computed, stored | True when both residuals are zero at their respective precisions. |
| `full_reconcile_id` | Link to Full Reconciliation | Read-only, not copied, indexed when set. Set when the matched group nets to zero. |
| `matched_debit_ids` | Sub-records: Partial Reconciliation | The partial matches in which this item is the **credit** side (the counterpart items are debits). |
| `matched_credit_ids` | Sub-records: Partial Reconciliation | The partial matches in which this item is the **debit** side. |
| `matching_number` | Text, indexed, not copied | The label of the matched group: the identifier of the full reconciliation when the group is closed, the letter `P` followed by a number while the group is only partially matched, or the letter `I` followed by anything for a number imported from another system and not yet turned into real matches. |
| `is_account_reconcile` | Boolean, related to the account | Whether the account allows matching. |
| `reconciled_lines_ids` | Multiple links to Journal Item, computed with an inverse | Every item matched with this one. Writing it triggers a reconciliation of the whole set. |
| `reconciled_lines_excluding_exchange_diff_ids` | Multiple links to Journal Item, computed | The same set without the exchange-difference items. |
| `exchange_move_ids` | Multiple links to Journal Entry, computed | The exchange-difference entries produced by the matches of this item. |

The residual is zero for an item on an account that neither allows matching nor is a bank-and-cash or credit-card account.

### Analytic fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `analytic_distribution` | Structured data, with an inverse | A map from analytic account (or a combination of analytic accounts) to a percentage. Writing it deletes and recreates the analytic lines of a posted item. |
| `analytic_line_ids` | Sub-records: Analytic Line | The analytic postings derived from this item. |
| `has_invalid_analytics` | Boolean, computed, not stored | True when the distribution breaks a mandatory analytic applicability rule. |

The content of the distribution and the rules that validate it belong to `../analytic-accounting/`.

### Early-payment fields

| Field (storage name) | Type | Meaning |
|---|---|---|
| `discount_date` | Date, read-only, stored | Last day on which the discounted amount may be paid |
| `discount_amount_currency` | Money in the item currency, stored | The discounted amount to pay |
| `discount_balance` | Money in company currency, stored | The discounted balance |
| `payment_date` | Date, computed, not stored | The nearer of the discount date and the due date; searchable |

### Database checks

| Name | Condition | Message |
|---|---|---|
| Credit or debit | For every accountable item, the product of debit and credit must be zero | "Wrong credit or debit value in accounting entry!" |
| Sign coherence | For every accountable item, the balance and the foreign amount must have the same sign (both may be zero) | "The amount expressed in the secondary currency must be positive when account is debited and negative when account is credited. If the currency is the same as the one from the company, this amount must strictly be equal to the balance." |
| Account required | Every accountable item must have an account | "Missing required account on accountable line." |
| Non-accountable emptiness | A section, subsection or note must have a zero foreign amount, a zero debit, a zero credit and no account | "Forbidden balance or account on non-accountable line" |

### Ordering

Descending date, then descending entry number, then ascending identifier.

### Display name

The item label, prefixed by the entry number and the entry reference when they exist, in the form "*number* *reference* *label*" with the parts that exist joined by spaces.

### Indexes

On (partner, reference); on (descending date, descending entry number, identifier); on (account, partner) restricted to unreconciled items; on the journal restricted to items with a negative residual; on (account, date). The account column is deliberately **not** indexed on its own because it is covered by the pair with the date.

---

## 10. Partial Reconciliation

Partial Reconciliation (`account.partial.reconcile`, table `account_partial_reconcile`).

### Purpose

One matched amount between exactly one debit item and exactly one credit item. Reconciling an invoice with a payment creates one Partial Reconciliation; reconciling an invoice with three payments creates three. The Partial Reconciliation carries **three** amounts, because the debit item and the credit item may be expressed in different currencies: the amount in the company currency, the amount in the currency of the debit item, and the amount in the currency of the credit item. All three are always positive.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `debit_move_id` | Link to Journal Item | Required, indexed. The item whose balance is positive. |
| `credit_move_id` | Link to Journal Item | Required, indexed. The item whose balance is negative. |
| `full_reconcile_id` | Link to Full Reconciliation | Not copied, indexed when set. Filled when the group closes. |
| `exchange_move_id` | Link to Journal Entry | Indexed when set. The exchange-difference entry this match produced, if any. |
| `draft_caba_move_vals` | Structured data | The values that produced the draft cash-basis entry, kept so that posting can detect that the source document changed in the meantime. |
| `company_currency_id` | Link to Currency, related to the company | |
| `debit_currency_id` | Link to Currency, related to the debit item, stored, precomputed | Required in effect: a validation refuses a match whose two currencies are not both known. |
| `credit_currency_id` | Link to Currency, related to the credit item, stored, precomputed | Same. |
| `amount` | Money in company currency | Always positive. The matched amount in the company currency. |
| `debit_amount_currency` | Money in the debit currency | Always positive. The matched amount seen from the debit item. |
| `credit_amount_currency` | Money in the credit currency | Always positive. The matched amount seen from the credit item. |
| `company_id` | Link to Company | Computed, stored, editable, precomputed: the company of the debit item when its entry is an invoice-like document, otherwise the company of the credit item. This decides where exchange-difference and cash-basis entries are created. |
| `max_date` | Date, computed, stored, precomputed | The later of the two item dates. Used to place the match on the aged balance reports. |

### Validation

A match whose debit currency or credit currency is unknown is refused with the message "Missing foreign currencies on partials having ids: *the identifiers*".

### Creation side effects

1. Any payment that was "in process" and whose amount now matches the amount of the match becomes "paid".
2. The matching numbers of both items and of every item transitively matched with them are recomputed.

### Deletion side effects

1. Any payment that was "paid" and whose amount matched becomes "in process" again.
2. The cash-basis entries created from this match and the exchange-difference entry of this match are collected.
3. The full reconciliation, if any, is deleted.
4. Collected entries that are not draft are reversed with a cancelling reversal dated at the date of the original, pushed to the day after the latest violated lock date when the original date is locked, and referenced "Reversal of: *the entry number*". Collected draft entries are simply deleted.
5. The matching numbers of the surviving items are recomputed.

---

## 11. Full Reconciliation

Full Reconciliation (`account.full.reconcile`, table `account_full_reconcile`).

### Purpose

The marker that a set of matched items nets exactly to zero. It carries no amount: its only role is to give the group a stable identifier that becomes the matching number printed on every item of the group.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `partial_reconcile_ids` | Sub-records: Partial Reconciliation | Every match that belongs to this closed group. |
| `reconciled_line_ids` | Sub-records: Journal Item | Every item of the closed group. |

### Creation

Creating a Full Reconciliation writes its identifier into the `full_reconcile_id` of every listed item and of every listed match in one operation, then recomputes the matching numbers of those items. The matching number of an item inside a closed group is the decimal identifier of the Full Reconciliation.

### Deletion

Deleting a Full Reconciliation clears the link on the items and recomputes their matching numbers: an item that still takes part in surviving matches falls back to the partial form (`P` followed by a number) and an item with no surviving match loses its matching number entirely.

---

## 12. Lock Exception

Lock Exception (`account.lock_exception`, table `account_lock_exception`).

### Purpose

A time-limited and optionally user-limited relaxation of **one** soft lock date, for **one** company. It lets a named user (or everybody) record entries in a period that is otherwise closed, while leaving an auditable trace of exactly who could do what and for how long.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `active` | Boolean | Default true. Set to false when the exception is revoked. |
| `state` | Selection, computed, not stored | `revoked` when inactive; `expired` when active with an end moment in the past; `active` otherwise. Searchable. |
| `company_id` | Link to Company | Required, read-only, default the active company. |
| `user_id` | Link to User | Default the acting user. When empty the exception applies to **everyone**. |
| `reason` | Text | Free explanation shown in the audit trail. |
| `end_datetime` | Date and time | When empty the exception never expires. |
| `lock_date_field` | Selection | Required. Which lock date is relaxed: `fiscalyear_lock_date` (Global Lock Date), `tax_lock_date` (Tax Return Lock Date), `sale_lock_date` (Sales Lock Date), `purchase_lock_date` (Purchase Lock Date). The hard lock date can never be relaxed. |
| `lock_date` | Date | The value the lock date takes for the beneficiary. An empty value means "no lock date at all". |
| `company_lock_date` | Date, not copied | The value the company lock date had when the exception was created; used to bound the audit query. |
| `fiscalyear_lock_date`, `tax_lock_date`, `sale_lock_date`, `purchase_lock_date` | Dates, computed, not stored | Convenience views: the field named by `lock_date_field` returns `lock_date`; the three others return the maximal representable date, meaning "unchanged". |

### Creation

A creation may be expressed either with the pair (`lock_date_field`, `lock_date`) or by assigning one of the four convenience fields; exactly one convenience field must be given, otherwise the creation fails with "A single exception must change exactly one lock date field."

At creation the exception records the current company lock date and posts a message on the company audit trail of the form:

> *link labelled "Exception"* for *the user display name, or the word "everyone"* valid until *the end moment* for '*the reason*'.

with a tracked value showing the lock date moving from the company value to the exception value. The parts "valid until …" and "for '…'" are omitted when there is no end moment and no reason.

### Duplication

Duplicating an exception is refused: "You cannot duplicate a Lock Date Exception."

### Revocation

Revoking sets the record inactive and stamps the end moment with the current time. Revocation requires the accounting-adviser group; otherwise it fails with "You cannot revoke Lock Date Exceptions. Ask someone with the 'Adviser' role."

### Re-creation when the company lock date moves

When a company lock date is written, every active exception for that field that relaxed the **previous** value is copied (so that the copy records the new company value) and the original is revoked. The beneficiary therefore keeps the same relaxation but the trace shows against which company value it was granted.

### Index

An index exists on (company, user, end moment) restricted to active exceptions, because the lookup of the applicable exception happens on every lock check.

---

## 13. Automatic Sequence (abstract behavior)

`sequence.mixin` is not a table: it is a shared behavior that any numbered document adopts. In this domain the Journal Entry adopts it. The behavior declares which field holds the number (here `name`), which field holds the date that governs the period (here `date`), and which field groups the numbering chains (here `journal_id`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `sequence_prefix` | Text, computed, stored | Everything in the number before the trailing digit block. |
| `sequence_number` | Integer, computed, stored | The trailing digit block as an integer, zero when absent. |

The full numbering grammar, the five reset periodicities, the derivation of the format from the previous number, the locking discipline that guarantees uniqueness under concurrency, and the chain-end test are specified in `calculations.md`.

Two database indexes are created for the adopting table: one on (grouping field, descending prefix, descending number, number field) and one on (grouping field, descending identifier, prefix). A unique index on the number field is expected; without one, concurrent numbering can produce duplicates.

---

## 14. Company — ledger fields

Only the fields that belong to the general ledger are listed. Tax settings are in `../taxes/`, currency settings in `../multi-currency/`, invoice-presentation settings in `../accounts-receivable/`.

### Fiscal year

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `fiscalyear_last_day` | Integer | Required, default 31. The day of the month on which the fiscal year ends. Delegated to the root company. |
| `fiscalyear_last_month` | Selection of the twelve months | Required, default December. Delegated to the root company. |

Validation: unless the chosen pair is the twenty-ninth of February (which is accepted because the year is unknown), the day must be between one and the number of days of that month in the year of the opening entry, or in the current year when there is no opening entry. Message: "Invalid fiscal year last day".

### Lock dates

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `fiscalyear_lock_date` | Date | "Global Lock Date". Tracked. Any entry dated on or before it is refused and, at posting, postponed. |
| `tax_lock_date` | Date | "Tax Return Lock Date". Tracked. Applies only to entries that affect the tax report. Set automatically when a tax closing entry is posted. |
| `sale_lock_date` | Date | "Sales Lock Date". Tracked. Applies only to entries in a sale journal. |
| `purchase_lock_date` | Date | "Purchase Lock date". Tracked. Applies only to entries in a purchase journal. |
| `hard_lock_date` | Date | "Hard Lock Date". Tracked. Irreversible: it can never be removed and never be moved backwards, and no exception can relax it. |
| `user_fiscalyear_lock_date`, `user_tax_lock_date`, `user_sale_lock_date`, `user_purchase_lock_date` | Dates, computed, not stored | The effective value of each soft lock date **for the acting user**, after applying the applicable exception and after taking the maximum over the company and all its ancestors. |
| `user_hard_lock_date` | Date, computed, not stored | The maximum hard lock date over the company and all its ancestors. |

### Default accounts and journals

| Field (storage name) | Meaning |
|---|---|
| `transfer_account_id` | Intermediary account used when money moves from one liquidity account to another; restricted to reconcilable current-asset accounts |
| `account_journal_suspense_account_id` | The default suspense account proposed on new liquidity journals |
| `default_cash_difference_income_account_id`, `default_cash_difference_expense_account_id` | The default profit and loss accounts proposed on new liquidity journals |
| `currency_exchange_journal_id` | The miscellaneous journal in which exchange-difference entries are booked |
| `income_currency_exchange_account_id` | The income account credited by an exchange gain; restricted to the income group |
| `expense_currency_exchange_account_id` | The expense account debited by an exchange loss; restricted to expense and other-expense types |
| `account_journal_early_pay_discount_gain_account_id`, `account_journal_early_pay_discount_loss_account_id` | The write-off accounts of an early-payment discount |
| `expense_accrual_account_id`, `revenue_accrual_account_id` | The accounts used by the automatic transfer wizard when it moves an amount to another period; restricted respectively to liability accounts that are not payable or receivable, and to asset accounts that are not payable or receivable |
| `automatic_entry_default_journal_id` | The miscellaneous journal used by default by the automatic transfer wizard |
| `income_account_id`, `expense_account_id` | The default income and expense accounts of the company, also written as the default income and expense accounts of product categories |
| `price_difference_account_id` | The account that absorbs the difference between a standard cost and a billed price |
| `account_discount_income_allocation_id`, `account_discount_expense_allocation_id` | The accounts used when a discount is booked separately |
| `account_default_pos_receivable_account_id` | The receivable account used by point-of-sale sessions |

### Chart and prefixes

| Field (storage name) | Meaning |
|---|---|
| `expects_chart_of_accounts` | Default true. Whether this company needs a chart of accounts at all |
| `chart_template` | The identifier of the loaded chart template; the list of choices depends on the company country |
| `bank_account_code_prefix`, `cash_account_code_prefix`, `transfer_account_code_prefix` | The code prefixes used when a liquidity or transfer account has to be created |

Changing a bank or cash prefix renumbers the existing bank-and-cash and credit-card accounts of the company that start with the old prefix: the new code is the new prefix followed by the old code with the old prefix removed, leading zeros stripped and the remainder right-aligned with zeros to the original width.

### Opening entry

| Field (storage name) | Meaning |
|---|---|
| `account_opening_move_id` | The Journal Entry holding the initial balances of every account |
| `account_opening_journal_id` | Related: the journal of that entry, writable |
| `account_opening_date` | The date of the opening; the entry is dated the day before it |

### Other ledger switches

| Field (storage name) | Meaning |
|---|---|
| `account_storno` | Storno accounting. Computed from the fiscal country, stored, editable: true by default in the countries that require it. Delegated to the root company |
| `restrictive_audit_trail` | When true, the log messages of journal entries may not be deleted and a posted entry may never be deleted |
| `force_restrictive_audit_trail` | Computed, always false in the core; a country package sets it to true and the restrictive audit trail then cannot be switched off |
| `anglo_saxon_accounting` | Whether the cost of goods sold is recognised at invoicing rather than at delivery |
| `autopost_bills` | Default true. Whether vendor bills from trusted partners are posted automatically |
| `batch_payment_sequence_id` | Read-only, not copied. The numbering sequence used for group payment communications |

The list of countries in which storno accounting is mandatory is: Bosnia and Herzegovina, China, Czechia, Croatia, Poland, Romania, Serbia, Russia, Slovenia, Slovakia, Ukraine. The countries in which it is offered but not mandatory are Austria, Switzerland, Germany and Italy.

### Write-time behavior

- Writing any lock date runs the lock validation described in `business-rules.md`.
- Writing the currency is refused when any journal item exists in the company or in a descendant: "You cannot change the currency of the company since some journal items already exist".
- Writing a soft lock date revokes and re-creates every active exception that relaxed the previous value.
- Creating a company loads the chart template of its first ancestor, if any, and creates its group-payment numbering sequence.

---

## 15. Partner — ledger fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `property_account_receivable_id` | Link to Account, per company | The receivable account used for this counterpart. |
| `property_account_payable_id` | Link to Account, per company | The payable account used for this counterpart. |
| `credit` | Money, computed, not stored, searchable | The total residual of the unreconciled posted items of this partner on **receivable** accounts, for the active company hierarchy. |
| `debit` | Money, computed, not stored, searchable | Minus the total residual of the unreconciled posted items of this partner on **payable** accounts, for the active company hierarchy. |
| `total_invoiced` | Money, computed, not stored | The sum of the untaxed subtotals of the customer invoices and credit notes of this partner and its children, excluding draft and cancelled documents. |
| `days_sales_outstanding` | Number, computed, not stored | See `calculations.md`. |
| `account_move_count` | Integer, computed | How many entries mention this partner; visible only to accounting users. |
| `trust` | Selection, per company | `good` (Good Debtor), `normal` (Normal Debtor), `bad` (Bad Debtor). |

The computation of the credit and debit totals is deliberately company-hierarchy-wide: it uses every posted, unreconciled item on a receivable or payable account whose company is the root of the active company or a descendant of it.

---

## 16. Wizard entities

These are transient records: they exist only for the duration of one user interaction and are purged by housekeeping.

### 16.1 Reversal wizard

`account.move.reversal`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `move_ids` | Multiple links to Journal Entry | The entries to reverse. Only posted entries are offered. Filled from the selection. |
| `new_move_ids` | Multiple links to Journal Entry | The entries produced. |
| `date` | Date | Default today. The date of the reversal. |
| `reason` | Text | Free text appended to the reference of the reversal. |
| `journal_id` | Link to Journal | Required. Computed from the selection (the first active journal of the selected entries), stored, editable. Must be of the same type as the journal of the entries. |
| `company_id` | Link to Company | Required, read-only. |
| `available_journal_ids` | Multiple links to Journal, computed | The journals of the company whose type matches one of the selected entries. |
| `residual`, `currency_id`, `move_type` | computed | Shown only when exactly one entry is selected, to size the credit note. |

Opening the wizard fails when the selection spans several companies ("All selected moves for reversal must belong to the same company.") or contains an entry that is not posted ("To reverse a journal entry, it has to be posted first."). Choosing a journal of a different type fails with "Journal should be the same type as the reversed entry."

### 16.2 Automatic transfer wizard

`account.automatic.entry.wizard`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `action` | Selection | Required. `change_period` (Change Period) or `change_account` (Change Account). |
| `move_line_ids` | Multiple links to Journal Item | The items to act on, taken from the selection. |
| `date` | Date | Required, default today. The date of the new entry. |
| `journal_id` | Link to Journal | Required, restricted to miscellaneous journals. Computed from the company default, with an inverse that stores the choice back on the company. |
| `company_id` | Link to Company | Required, read-only; the root company of the selected items. |
| `company_currency_id` | Link to Currency, related | |
| `percentage` | Number | The share of each item to move. Computed from the total amount, stored, editable. Must be greater than zero and at most one hundred when the action is "Change Period". |
| `total_amount` | Money in company currency | Computed from the percentage, stored, editable. The two fields are inverse views of each other. |
| `account_type` | Selection, computed, stored | `income` (Revenue) when the sum of the selected balances is negative, `expense` (Expense) otherwise. Decides which accrual account is used. |
| `expense_accrual_account`, `revenue_accrual_account` | Links to Account | Computed from the company, editable, with inverses that store the choice back on the company. Receivable, payable and off-balance accounts are excluded. |
| `destination_account_id` | Link to Account | The target account of a "Change Account" transfer. |
| `lock_date_message` | Text, computed | The lock-date warning of the first selected item, if any. |
| `display_currency_helper` | Boolean, computed | True when the destination account restricts the currency, so that a conversion warning is shown. |
| `move_data`, `preview_move_data` | Structured text, computed | The entries that will be created, and a trimmed version of them for the preview panel (at most four entries, four or five columns). |

Opening the wizard fails when the selection is not journal items ("This can only be used on journal items"), when any selected item belongs to an entry that is not posted ("Oops! You can only change the period or account for posted entries! Other ones aren't up for an adventure like that!"), when any selected item is reconciled ("Oops! You can only change the period or account for items that are not yet reconciled! Other ones aren't up for an adventure like that!"), when the items span several company hierarchies ("You cannot use this wizard on journal entries belonging to different companies.") or when no action is possible ("No possible action found with the selected lines."). Choosing a date inside a locked period fails with "The date selected is protected by: *the list of lock dates*."

### 16.3 Resequence wizard

`account.resequence.wizard`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `move_ids` | Multiple links to Journal Entry | The entries to renumber, taken from the selection. |
| `first_name` | Text | Required. Computed as the smallest existing number of the selection, editable. The first number of the new series; its shape also decides the reset periodicity. |
| `sequence_number_reset` | Text, computed | The periodicity deduced from the first number. |
| `ordering` | Selection | Required, default `keep`. `keep` (Keep current order) assigns the new numbers in the order of the current prefix and number; `date` (Reorder by accounting date) assigns them in the order of the accounting date, then the current number, then the identifier. |
| `first_date`, `end_date` | Dates | Informative bounds of the operation. |
| `new_values` | Structured text, computed | The proposed new number of every selected entry, under both orderings. |
| `preview_moves` | Structured text, computed | A condensed version for the preview: the first three rows, the last row, every row whose two orderings disagree, and every row that opens a new period; the skipped rows are collapsed into a row labelled "… (*count* other)". |

Opening the wizard fails when the selection spans several journals ("You can only resequence items from the same journal"), when the journal has a dedicated credit-note numbering and the selection mixes credit notes with other documents ("The sequences of this journal are different for Invoices and Refunds but you selected some of both types.") or when the journal has a dedicated payment numbering and the selection mixes payments with other documents ("The sequences of this journal are different for Payments and non-Payments but you selected some of both types.").

### 16.4 Validate entries wizard

`validate.account.move`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `move_ids` | Multiple links to Journal Entry | The draft entries to post. Filled from a selection of entries or from a journal. |
| `force_post` | Boolean | "Force". Post entries dated in the future immediately instead of scheduling them. |
| `display_force_post` | Boolean, computed | True when at least one selected entry is dated in the future. |
| `force_hash` | Boolean | "Force Hash". Post entries of hash-secured journals too. |
| `display_force_hash` | Boolean, computed | True when at least one selected entry belongs to a hash-secured journal. |
| `is_entries` | Boolean, computed | True when at least one selected document is a plain entry. |
| `abnormal_date_partner_ids`, `abnormal_amount_partner_ids` | Sub-records: Partner, computed | The partners of the selected documents that carry an abnormal-date or abnormal-amount warning. |
| `ignore_abnormal_date`, `ignore_abnormal_amount` | Booleans | When ticked, the corresponding warning is switched off permanently for those partners. |

Opening the wizard with no draft entry fails with "There are no journal items in the draft state to post."

### 16.5 Secure entries wizard

`account.secure.entries.wizard`.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `company_id` | Link to Company | Required, read-only, default the active company. |
| `hash_date` | Date | Required. "Hash All Entries". Computed from the maximum hashable date, stored, editable. Every eligible entry dated on or before it is hashed. |
| `max_hash_date` | Date, computed | The highest date such that every posted entry up to and including it is already secured. Computed as the day before the earliest date among the entries that still need hashing and the entries that cannot be hashed; empty when nothing remains. |
| `move_to_hash_ids` | Multiple links to Journal Entry, computed | Exactly the entries that will be hashed. |
| `chains_to_hash_with_gaps` | Structured data, computed | The first and last entry of every chain whose hashing would leave a numbering gap. |
| `not_hashable_unlocked_move_ids` | Multiple links to Journal Entry, computed | Entries before the date that cannot be hashed and are not protected by the hard lock date. |
| `unreconciled_bank_statement_line_ids` | Multiple links to Statement Line, computed | Unreconciled bank transactions before the date; their whole numbering chain is excluded from the operation. |
| `warnings` | Structured data, computed | The warnings to display, each with a message, a severity, a button label and an action. |

The warning messages are:

| Key | Message |
|---|---|
| unreconciled transactions | "There are still unreconciled bank statement lines before the selected date. The entries from journal prefixes containing them will not be secured: *the list of prefixes*" (severity: danger) |
| draft entries | "There are still draft entries before the selected date." |
| not hashable | "There are entries that cannot be hashed. They can be protected by the Hard Lock Date." |
| gap | "Securing these entries will create at least one gap in the sequence." |
| beyond the date | "Securing these entries will also secure entries after the selected date." |

Running the wizard without a date fails with "Set a date. The moves will be secured up to including this date."

---
