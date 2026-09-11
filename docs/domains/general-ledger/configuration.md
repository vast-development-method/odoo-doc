# General Ledger — Configuration

Settings, system parameters, sequences and numbering formats, default records, security groups, the complete access rights matrix, the record rules and the scheduled jobs of the domain.

Contents:

1. [Company accounting settings](#1-company-accounting-settings)
2. [Journal settings](#2-journal-settings)
3. [Account settings](#3-account-settings)
4. [System parameters](#4-system-parameters)
5. [Numbering and sequences](#5-numbering-and-sequences)
6. [Default records shipped with the domain](#6-default-records-shipped-with-the-domain)
7. [Security groups](#7-security-groups)
8. [Access rights matrix](#8-access-rights-matrix)
9. [Record rules](#9-record-rules)
10. [Scheduled jobs](#10-scheduled-jobs)
11. [Onboarding steps](#11-onboarding-steps)

---

## 1. Company accounting settings

These are stored on the company and edited from the accounting settings screen. Only the fields belonging to the general ledger are listed; the tax fields are in `../taxes/`, the invoice-presentation fields in `../accounts-receivable/`.

### Fiscal periods

| Setting | Storage name | Type | Default | Effect |
|---|---|---|---|---|
| Fiscal year last day | `fiscalyear_last_day` | Integer, required | 31 | With the month, defines the fiscal year. Delegated to the root company. |
| Fiscal year last month | `fiscalyear_last_month` | Selection of the twelve months, required | December | Same. |
| Global Lock Date | `fiscalyear_lock_date` | Date | empty | No entry can be added or modified on or before it. |
| Tax Return Lock Date | `tax_lock_date` | Date | empty | No entry affecting the tax report can be added or modified on or before it. Set automatically when a tax closing entry is posted. |
| Sales Lock Date | `sale_lock_date` | Date | empty | Same, restricted to sale journals. |
| Purchase Lock Date | `purchase_lock_date` | Date | empty | Same, restricted to purchase journals. |
| Hard Lock Date | `hard_lock_date` | Date | empty | Same as the Global Lock Date, but irreversible and impossible to relax. |

All five are tracked in the company audit trail.

### Default accounts and journals

| Setting | Storage name | Restricted to |
|---|---|---|
| Internal Transfer account | `transfer_account_id` | reconcilable Current Assets accounts |
| Journal Suspense account | `account_journal_suspense_account_id` | any account of the company |
| Cash Difference Income | `default_cash_difference_income_account_id` | any account of the company |
| Cash Difference Expense | `default_cash_difference_expense_account_id` | any account of the company |
| Exchange Gain or Loss Journal | `currency_exchange_journal_id` | miscellaneous journals |
| Gain Exchange Rate Account | `income_currency_exchange_account_id` | accounts of the income group |
| Loss Exchange Rate Account | `expense_currency_exchange_account_id` | Expenses and Other Expenses accounts |
| Cash Discount Write-Off Gain | `account_journal_early_pay_discount_gain_account_id` | any account of the company |
| Cash Discount Write-Off Loss | `account_journal_early_pay_discount_loss_account_id` | any account of the company |
| Expense accrual account | `expense_accrual_account_id` | liability accounts that are neither payable nor receivable |
| Revenue accrual account | `revenue_accrual_account_id` | asset accounts that are neither payable nor receivable |
| Automatic entry journal | `automatic_entry_default_journal_id` | miscellaneous journals |
| Income Account | `income_account_id` | any account of the company |
| Expense Account | `expense_account_id` | any account of the company |
| Price Difference Account | `price_difference_account_id` | any account of the company |
| Separate account for income discount | `account_discount_income_allocation_id` | any account of the company |
| Separate account for expense discount | `account_discount_expense_allocation_id` | any account of the company |
| Default point-of-sale receivable account | `account_default_pos_receivable_account_id` | any account of the company |

### Chart and code prefixes

| Setting | Storage name | Effect |
|---|---|---|
| Expects a Chart of Accounts | `expects_chart_of_accounts` | Default true. When false the company works without a chart. |
| Chart template | `chart_template` | The loaded template code; the choice list depends on the company country. |
| Prefix of the bank accounts | `bank_account_code_prefix` | Starting point for generated bank and credit-card account codes. Changing it renumbers the existing ones. |
| Prefix of the cash accounts | `cash_account_code_prefix` | Same for cash accounts; falls back to the bank prefix. |
| Prefix of the transfer accounts | `transfer_account_code_prefix` | Starting point for the liquidity transfer account. |

### Ledger behavior switches

| Setting | Storage name | Default | Effect |
|---|---|---|---|
| Storno accounting | `account_storno` | true in the countries that require it | A reversal is booked as a negative amount on the same side instead of an amount on the opposite side. Delegated to the root company. |
| Show the storno switch | `display_account_storno` | computed | The switch is shown only in the countries where storno accounting is mandatory or optional. |
| Restricted Audit Trail | `restrictive_audit_trail` | false | Log messages of journal entries cannot be deleted and a posted entry can never be deleted. Tracked. |
| Forced Audit Trail | `force_restrictive_audit_trail` | computed, always false in the core | When a country package sets it, the restricted audit trail cannot be switched off. |
| Anglo-saxon accounting | `anglo_saxon_accounting` | false | Cost of goods sold recognised at invoicing. Specified in `../inventory-valuation-and-costing/`. |
| Auto-validate bills | `autopost_bills` | true | Bills of trusted counterparts are posted automatically. |
| Quick encoding | `quick_edit_mode` | empty | Enables the total-driven encoding mode for customer invoices, vendor bills or both. Also relaxes the chain-end guard on deletion. |
| Sales Credit Limit | `account_use_credit_limit` | false | Enables the counterpart credit limit. |

### Opening entry

| Setting | Storage name | Effect |
|---|---|---|
| Opening Journal Entry | `account_opening_move_id` | The entry holding the initial balances. |
| Opening Journal | `account_opening_journal_id` | Related to the journal of that entry; writable. |
| Opening Entry date | `account_opening_date` | The date the books open; the entry is dated the day before. |

### Country lists that drive the storno default

| Behavior | Countries |
|---|---|
| Storno mandatory (the switch defaults to on) | Bosnia and Herzegovina, China, Czechia, Croatia, Poland, Romania, Serbia, Russia, Slovenia, Slovakia, Ukraine |
| Storno offered (the switch is visible) | the mandatory list plus Austria, Switzerland, Germany, Italy |

---

## 2. Journal settings

| Setting | Storage name | Default | Effect |
|---|---|---|---|
| Sequence Prefix | `code` | the type prefix and the smallest free number | Used to build the number of the entries. |
| Dedicated Credit Note Sequence | `refund_sequence` | true for sale and purchase journals | Credit notes get their own numbering chain, prefixed with the letter R. |
| Dedicated Payment Sequence | `payment_sequence` | true for bank, cash and credit-card journals | Payment entries get their own numbering chain, prefixed with the letter P. |
| Secure Posted Entries with Hash | `restrict_mode_hash_table` | false | Posting hashes the chain. Cannot be switched off once entries are hashed. |
| Sequence override pattern | `sequence_override_regex` | empty | Replaces the built-in numbering grammar for this journal. |
| Self Billing | `is_self_billing` | false | The numbering chain is split per counterpart. |
| Communication Type | `invoice_reference_type` | based on the invoice | Which subject the structured payment reference identifies. |
| Communication Standard | `invoice_reference_model` | the first value whose name begins with the country code, otherwise the full reference | Which reference format is produced. |
| Bank Feeds | `bank_statements_source` | undefined | How statements reach the journal. |
| Show journal on dashboard | `show_on_dashboard` | true | Whether the journal appears as a card. |
| Color Index | `color` | 0 | The colour of the card. |
| Order | `sequence` | 10 | The card and list order. |
| Send Copy To | `incoming_einvoice_notification_email` | empty | Semicolon-separated addresses receiving a copy of every sent and received invoice. |
| Ledger Group | `journal_group_ids` | empty | The report filters this journal participates in. |

---

## 3. Account settings

| Setting | Storage name | Effect |
|---|---|---|
| Allow Reconciliation | `reconcile` | Journal items on the account may be matched. Mandatory for Receivable and Payable, forbidden for Off-Balance Sheet. |
| Account Currency | `currency_id` | Forces every item on the account to that foreign currency. |
| Non trade | `non_trade` | Reports the account under non-trade receivables or payables. |
| Default Taxes | `tax_ids` | Taxes proposed when the account is chosen and the product carries none. |
| Tags | `tag_ids` | Free labels for custom reporting. |
| Active | `active` | Archiving. |

---

## 4. System parameters

| Key | Default | Effect |
|---|---|---|
| `sequence.mixin.constraint_start_date` | `1970-01-01` | The date-alignment check of the numbering is applied only to documents dated after this value. Raising it lets already misaligned historical documents be edited. |
| `account.product_name_similarity_threshold` | `0.9` | The similarity above which two product names are considered the same when decoding a received document. |
| `account.show_sale_receipts` | not set | Shows the sales-receipt document type. |
| `account.use_invoice_terms` | not set | Fills the terms and conditions of new sale documents from the company text. |
| `account.display_name_in_footer` | not set | Adds the company display name to the printed footer. |
| `account.pdf_generation_batch` | not set | The batch size used when generating printable files in the background. |

---

## 5. Numbering and sequences

### The numbering of journal entries

Journal entries are **not** numbered by a stored counter record. They are numbered by the automatic sequence behavior described in `calculations.md`, which derives the next number from the previous number of the same chain and reserves it with a database lock.

A *chain* is identified by:

| Component | Always | Additional splits |
|---|---|---|
| the journal | yes | — |
| the numbering prefix | yes | — |
| credit note versus other documents | — | only when the journal has a dedicated credit-note numbering |
| payment entry versus other entries | — | only when the journal has a dedicated payment numbering |
| the commercial counterpart | — | only when the journal is a self-billing journal |

### The default formats

| Journal type | Fiscal year ends 31 December | Fiscal year staggered |
|---|---|---|
| Sales, Bank, Cash, Credit Card | `CODE/YYYY/NNNNN`, restarting each calendar year | `CODE/YY-YY/NNNN`, restarting each fiscal year |
| Purchase, Miscellaneous | `CODE/YYYY/MM/NNNN`, restarting each month | `CODE/YY-YY/MM/NNNN`, restarting each month of the fiscal year |
| Self-billing (purchase) | `CODEPPPPP/YYYY/MM/NNNN` where `PPPPP` is the counterpart identifier padded to five characters | `CODEPPPPP/YY-YY/MM/NNNN` |

with the prefix letter `R` in front for a credit note in a journal with a dedicated credit-note numbering, and `P` in front for a payment entry in a journal with a dedicated payment numbering. Both prefixes can apply.

### The shipped numbering sequence records

Only one stored numbering sequence belongs to this package, and it does not number entries:

| Name | Code | Prefix | Padding | Company |
|---|---|---|---|---|
| Payment | `account.payment` | `PAY` | 5 | shared by all companies |

One further sequence is created per company on demand:

| Name | Implementation | Prefix | Padding | Uses a date range |
|---|---|---|---|---|
| Group Payments Number Sequence | without gaps | `GROUP/<year of the range>/` | 5 | yes |

Both belong to `../payments-and-bank-reconciliation/`; they are listed here because the sequence records live in the same package.

---

## 6. Default records shipped with the domain

### Account tags

Three tags of applicability "accounts" are shipped and cannot be deleted while the chart definitions use them:

| Purpose |
|---|
| Operating cash flow |
| Financing cash flow |
| Investing cash flow |

### Journals created by a chart template

See the table in `workflows.md`: Sales, Purchases, Miscellaneous Operations, Exchange Difference, Cash Basis Taxes and Bank.

### Reconciliation models created by a chart template

Internal Transfers and Bank Fees; see `workflows.md`.

### Utility accounts created by a chart template

Bank Suspense Account, Cash Discount Loss, Cash Discount Gain, Cash Difference Gain, Cash Difference Loss, Liquidity Transfer, Outstanding Receipts, Outstanding Payments, and the Profit or Loss Appropriation account of type Current Year Earnings; see `workflows.md`.

---

## 7. Security groups

Five groups form the accounting ladder, plus three feature groups.

| Group | Label | Implies | Meaning |
|---|---|---|---|
| `group_account_invoice` | Invoicing | the internal-user group | Create and edit invoices, credit notes, payments and the documents behind them; cannot see the accounting screens (journal entries, reports, reconciliation). Ordering 20 in the accounting privilege. |
| `group_account_readonly` | Show Accounting Features - Readonly | the internal-user group | See everything, including journal entries, advanced configuration and reports; change nothing. |
| `group_account_basic` | Basic | Invoicing | Additional accounting features such as basic bank reconciliation, but not journal entries. |
| `group_account_user` | Show Full Accounting Features | Basic and Readonly | The accountant: everything except the advanced configuration. |
| `group_account_manager` | Administrator | Invoicing | Full access including the configuration. Ordering 50 in the accounting privilege. Granted to the two built-in administrator accounts. |
| `group_account_secured` | Show Inalterability Features | — | Shows the hash column and the securing buttons. Activated automatically for the Readonly and the Invoicing groups the first time an entry is hashed in a journal that does not secure by default. |
| `group_cash_rounding` | Allow the cash rounding management | — | Shows the cash-rounding fields. |
| `group_partial_purchase_deductibility` | Partial Purchase Deductibility | — | Shows the deductibility field on vendor lines. Granted automatically to a user who posts a vendor bill carrying a deductibility other than one hundred. |
| `group_delivery_invoice_address` | Delivery Address | — | Shows the separate delivery address on documents. |
| `group_validate_bank_account` | Validate bank account | implied by the settings-administration group | Allows marking a bank account as trusted for outgoing payments. |

The ladder differs depending on which capability packages are installed. With only the invoicing package, the two groups intended for use are Invoicing and Administrator. With the accounting package added, Basic sits between Invoicing and Administrator. With the full accounting package, Show Full Accounting Features sits between Readonly and Administrator.

Two privileges group these in the settings screen: "Accounting" (ordering 7 under the accounting category) carrying the Invoicing and Administrator groups, and "Bank" (ordering 50) carrying the bank-account validation group.

---

## 8. Access rights matrix

Rights are create, read, update, delete. A blank cell means the group has no rule for that entity and therefore no access through that route.

### Chart of accounts

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Account | Administrator | yes | yes | yes | yes |
| Account | Show Accounting Features - Readonly | — | yes | — | — |
| Account | Invoicing | — | yes | — | — |
| Account | internal user | — | yes | — | — |
| Account | contact manager | — | yes | — | — |
| Account Group | Administrator | yes | yes | yes | yes |
| Account Group | Show Accounting Features - Readonly | — | yes | — | — |
| Account Group | Basic | — | yes | — | — |
| Account Root | Administrator | — | yes | — | — |
| Account Root | Show Accounting Features - Readonly | — | yes | — | — |
| Account Tag | Show Full Accounting Features | yes | yes | yes | yes |
| Account Tag | Show Accounting Features - Readonly | — | yes | — | — |
| Account Tag | Invoicing | — | yes | — | — |
| Account Tag | internal user | — | yes | — | — |
| Account Code Mapping | Administrator | yes | yes | — | — |
| Account Code Mapping | Show Accounting Features - Readonly | — | yes | — | — |

### Journals

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Journal | Administrator | yes | yes | yes | yes |
| Journal | Show Accounting Features - Readonly | — | yes | — | — |
| Journal | Invoicing | — | yes | — | — |
| Journal Group | Administrator | yes | yes | yes | yes |
| Journal Group | Show Accounting Features - Readonly | — | yes | — | — |
| Journal Group | Invoicing | — | yes | — | — |

### Entries and items

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Journal Entry | Invoicing | yes | yes | yes | yes |
| Journal Entry | Administrator | — | yes | — | — |
| Journal Entry | Show Accounting Features - Readonly | — | yes | — | — |
| Journal Entry | portal user | — | yes | — | — |
| Journal Item | Invoicing | yes | yes | yes | yes |
| Journal Item | Administrator | — | yes | — | — |
| Journal Item | Show Accounting Features - Readonly | — | yes | — | — |
| Journal Item | portal user | — | yes | — | — |

The Administrator group deliberately has only read access on entries and items through this route: an administrator obtains write access because the Administrator group implies the Invoicing group.

### Reconciliation

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Partial Reconciliation | Show Full Accounting Features | yes | yes | yes | yes |
| Partial Reconciliation | Invoicing | yes | yes | yes | yes |
| Partial Reconciliation | Show Accounting Features - Readonly | — | yes | — | — |
| Full Reconciliation | Show Full Accounting Features | yes | yes | yes | yes |
| Full Reconciliation | Invoicing | yes | yes | yes | yes |
| Full Reconciliation | Show Accounting Features - Readonly | — | yes | — | — |

### Lock exceptions

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Lock Exception | internal user | yes | — | — | — |
| Lock Exception | Administrator | yes | — | yes | — |

An ordinary internal user can therefore **request** an exception but can neither read the list nor revoke one; the Administrator can also modify them. Revocation is additionally guarded in the operation itself.

### Wizards

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Reversal wizard | Invoicing | yes | yes | yes | — |
| Validate entries wizard | Invoicing | yes | yes | yes | — |
| Automatic transfer wizard | Show Full Accounting Features | yes | yes | yes | — |
| Resequence wizard | Administrator | yes | yes | yes | — |
| Secure entries wizard | Administrator | yes | yes | yes | — |

---

## 9. Record rules

A record rule restricts which records a group may reach. A rule with no group applies to every user ("global"); rules attached to groups are combined with "or" among themselves and with "and" against the global rules.

### Multi-company rules (global)

| Entity | Condition |
|---|---|
| Journal Entry | the company of the entry is one of the companies currently active for the user |
| Journal Item | the company of the item is one of the companies currently active for the user |
| Journal | the company of the journal is an ancestor of, or equal to, one of the active companies |
| Journal Group | the group has no company, or its company is an ancestor of, or equal to, one of the active companies |
| Account | at least one company of the account is an ancestor of, or equal to, one of the active companies |
| Account Group | the company is an ancestor of, or equal to, one of the active companies |

Note the asymmetry: entries and items are restricted to the **exact** active companies, while journals, accounts and account groups are visible from a child company because their company is a parent.

### Functional rules

| Rule | Entity | Group | Condition | Rights |
|---|---|---|---|---|
| All Journal Entries | Journal Entry | Invoicing | always true | all |
| All Journal Items | Journal Item | Invoicing | always true | all |
| Readonly Move | Journal Entry | Show Accounting Features - Readonly | always true | read only |
| Readonly Move Line | Journal Item | Show Accounting Features - Readonly | always true | read only |
| Portal Personal Account Invoices | Journal Entry | portal user | the entry is neither draft nor cancelled, its type is one of the four invoice types, and its counterpart is the commercial entity of the user or a descendant of it | read |
| Portal Invoice Lines | Journal Item | portal user | the same, expressed on the parent entry | read |

The two "always true" rules for the Invoicing and Readonly groups exist so that a companion package that narrows the visibility of documents for salespeople does not accidentally hide the accounting view from a billing clerk or an auditor.

### Rules contributed to neighbouring domains by the same package

| Entity | Group | Condition | Rights |
|---|---|---|---|
| Analytic Line | Invoicing | always true | all |
| Analytic Line | Show Accounting Features - Readonly | always true | read only |
| Bank account of a counterpart | Invoicing | always true | all — so that a billing clerk can reach the bank accounts that the human-resources package otherwise restricts |

---

## 10. Scheduled jobs

| Job | Runs | Acts on | What it does |
|---|---|---|---|
| Post draft entries with automatic posting enabled and an accounting date up to today | every day, first run at two o'clock in the morning of the day after installation | Journal Entry | See the algorithm in `workflows.md`: batch of one hundred, whole-batch attempt then one at a time, failures reported on the entry and the automatic posting switched off |
| Send invoices automatically | every day, run as the internal system account | Journal Entry | Processes the posted entries that carry pending sending data, ten at a time, ordered by accounting date, then document date, then counter, then identifier; specified in `../accounts-receivable/` |

Both jobs report their progress to the job framework so that a long backlog is spread over several runs.

---

## 11. Onboarding steps

| Attached to the accounting dashboard | Order | Title | Description | Button | Once done |
|---|---|---|---|---|---|
| yes | 1 | Set Company Data | Set your company's data for documents header/footer. | Let's start! | Looks great! |
| yes | 1 | Set Periods | Define your fiscal years & tax returns periodicity. | Configure | Step completed! |
| yes | 4 | Review Chart of Accounts | Set up your chart of accounts and record initial balances. | Review Accounts | Chart of accounts set! |
| no | 3 | Documents Layout | Customize the look of your documents. | Customize | Looks great! |
| no | 100 | Taxes | Choose a default sales tax for your products. | Set taxes | Step Completed! |

The checklist is created for a company the first time the accounting dashboard is opened for it, and again whenever a company is created. The company-data step is marked done as soon as the street of the company is filled; the tax step is marked done when the user validates it.

The dashboard also shows, on the sales journal card, the checklist of `../accounts-receivable/`, and on the miscellaneous journal card the accounting checklist described above.
