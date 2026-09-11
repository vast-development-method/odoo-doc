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
