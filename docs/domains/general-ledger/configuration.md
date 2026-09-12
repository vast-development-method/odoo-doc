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

### Decimal precisions

Exactly one decimal precision is shipped by this package. A decimal precision is a named setting that tells the system how many decimal places a family of numbers carries; it is created with the force-create flag, meaning that it is restored if it was deleted.

| Name | Digits | What it governs |
|---|---|---|
| "Payment Terms" | 6 | The number of decimal places kept on the percentages and the fixed amounts of a payment term line, so that a term split into thirds does not lose a cent over a large amount. The payment term itself is specified in `../accounts-receivable/`. |

No other decimal precision belongs to this domain; currency decimal places are a property of the currency and are specified in `../multi-currency/`.

### Message subtypes shipped for the Journal Entry

A message subtype is a category of notification on the message thread of a record: followers subscribe to subtypes, and a subtype flagged as default is subscribed to automatically. Three subtypes are shipped on the Journal Entry, and **none of them is a default**, so a follower receives none of them until someone subscribes deliberately.

| Name | Description | Default | Hidden from the subscription list |
|---|---|---|---|
| "Validated" | "Invoice validated" | no | no |
| "Paid" | "Invoice paid" | no | no |
| "Invoice Created" | "Invoice Created" | no | **yes** |

The third is hidden, which means it never appears in the subscription dialogue: it exists so that the creation of a document can be logged as a typed message that automated rules can filter on, not so that a person can follow it. The events that raise the first two belong to `../accounts-receivable/`; the thread itself belongs to `../messaging-and-activities/`.

### The generic chart of accounts

One complete chart of accounts is shipped by this package itself, under the name "Generic Chart of Accounts". It is the chart offered to a company whose country has no chart of its own, and it is the chart every worked example of this folder uses. It declares **no country**, so it is offered everywhere; the country-specific charts are built by the same mechanism, described in `workflows.md`, from their own sets of the same four kinds of record.

**What it writes on the company.**

| Setting | Value |
|---|---|
| Anglo-Saxon accounting | on |
| Fiscal country | United States |
| Bank account code prefix | `1014` |
| Cash account code prefix | `1015` |
| Transfer account code prefix | `1017` |
| Receivable account of a counterpart | Account Receivable, code `1210` |
| Payable account of a counterpart | Account Payable, code `2110` |
| Point-of-sale receivable account | code `1013` |
| Exchange gain account | Foreign Exchange Gain, code `4410` |
| Exchange loss account | Foreign Exchange Loss, code `6410` |
| Cash difference gain account | Cash Difference Gain, code `4420` |
| Cash difference loss account | Cash Difference Loss, code `6420` |
| Early-payment discount loss account | Cash Discount Loss, code `4430` |
| Early-payment discount gain account | Cash Discount Gain, code `6430` |
| Default expense account | Expenses, code `6000` |
| Default income account | Product Sales, code `4000` |
| Inventory valuation journal | the miscellaneous journal named Inventory Valuation |
| Stock valuation account | Stock Valuation, code `1101` |
| Work-in-progress account of production | Work in Progress, code `1105` |
| Work-in-progress overhead account of production | Cost of Production, code `1104` |

It also writes, on the Stock Valuation account itself, the stock variation account: Stock Variation, code `6100`. The last five settings belong to `../inventory-valuation-and-costing/` and `../manufacturing/`; they are listed here because the chart is what sets them.

**The forty-six accounts.** The column "Matches" repeats the reconcilable flag, and "Non-trade" the non-trade flag; the cash-flow tags are the three tags listed above.

| Code | Name | Account type | Matches | Non-trade | Cash-flow tag |
|---|---|---|---|---|---|
| `1010` | Current Assets | Current Assets | no | no | — |
| `1013` | Account Receivable (`PoS`) | Receivable | yes | no | — |
| `1101` | Stock Valuation | Current Assets | no | no | — |
| `1104` | Cost of Production | Current Assets | yes | no | — |
| `1105` | Work in Progress | Current Assets | yes | no | — |
| `1210` | Account Receivable | Receivable | yes | no | — |
| `1211` | Products to receive | Current Assets | yes | no | — |
| `1220` | Owner's Current Account | Receivable | yes | **yes** | — |
| `1280` | Prepaid Expenses | Current Assets | no | no | — |
| `1310` | Tax Paid | Current Assets | no | no | — |
| `1320` | Tax Receivable | Receivable | yes | **yes** | — |
| `1410` | Prepayments | Prepayments | no | no | — |
| `1510` | Fixed Asset | Fixed Assets | no | no | — |
| `1910` | Non-current assets | Non-current Assets | no | no | — |
| `2010` | Current Liabilities | Current Liabilities | no | no | — |
| `2110` | Account Payable | Payable | yes | no | — |
| `2111` | Bills to receive | Current Liabilities | yes | no | — |
| `2120` | Deferred Revenue | Current Liabilities | no | no | — |
| `2300` | Salary Payable | Current Liabilities | yes | no | — |
| `2301` | Employee Payroll Taxes | Current Liabilities | yes | no | — |
| `2302` | Employer Payroll Taxes | Current Liabilities | yes | no | — |
| `2510` | Tax Received | Current Liabilities | no | no | — |
| `2520` | Tax Payable | Payable | yes | **yes** | — |
| `2910` | Non-current Liabilities | Non-current Liabilities | no | no | — |
| `3010` | Capital | Equity | no | no | — |
| `3020` | Dividends | Equity | no | no | — |
| `4000` | Product Sales | Income | no | no | Operating |
| `4410` | Foreign Exchange Gain | Income | no | no | Financing |
| `4420` | Cash Difference Gain | Income | no | no | Investing |
| `4430` | Cash Discount Loss | Expenses | no | no | — |
| `4500` | Other Income | Other Income | no | no | — |
| `5000` | Cost of Goods Sold | Cost of Revenue | no | no | Operating |
| `6000` | Expenses | Expenses | no | no | Operating |
| `6100` | Stock Variation | Expenses | no | no | — |
| `6110` | Purchase of Equipments | Expenses | no | no | Investing |
| `6120` | Rent | Expenses | no | no | Investing |
| `6200` | Bank Fees | Expenses | no | no | Financing |
| `6300` | Salary Expenses | Expenses | no | no | Operating |
| `6410` | Foreign Exchange Loss | Expenses | no | no | Financing |
| `6420` | Cash Difference Loss | Expenses | no | no | Investing |
| `6430` | Cash Discount Gain | Income | no | no | — |
| `9610` | RD Expenses | Expenses | no | no | Investing |
| `9620` | Sales Expenses | Expenses | no | no | Investing |
| `201100` | Credit Card | Credit Card | no | no | — |
| `999998` | Accumulated Retained Earnings | Current Year Earnings | no | no | — |
| `999999` | Profit or Loss Appropriation | Current Year Earnings | no | no | — |

The three code prefixes written on the company (`1014`, `1015` and `1017`) carry no shipped account: they are the starting points from which the loading mechanism invents the codes of the bank, cash and transfer accounts it creates, by the walk described in `entities.md`.

Two oddities of the shipped data are reproduced as they are, because a rebuild that "corrects" them would produce different figures from the same template. **Compatibility finding.** The account named Cash Discount Loss, code `4430`, is of type Expenses although its code sits in the income range, and the account named Cash Discount Gain, code `6430`, is of type Income although its code sits in the expense range: the two names are swapped with respect to their codes and their types. A corrected chart would name `4430` the gain and `6430` the loss, or renumber them. The observed data is the one specified above, and the company settings quoted earlier point the early-payment discount **loss** setting at `4430` and the **gain** setting at `6430`, consistently with the names and inconsistently with the types.

**The two tax groups.**

| Name | Country | Account credited when the group is payable | Account debited when the group is receivable |
|---|---|---|---|
| "Tax 15%" | United States | Tax Payable, code `2520` | Tax Receivable, code `1320` |
| "Tax 0%" | United States | Tax Payable, code `2520` | Tax Receivable, code `1320` |

**The four taxes.** Each has an amount expressed as a percentage of the base and a distribution of four lines: on an invoice, the whole base to the grids and the whole tax to one account; on a credit note, the same two lines again. No tax grid is attached, because the generic chart carries no tax report.

| Name | Applies to | Rate | Group | Account of the tax line | Attached fiscal position | Replaces |
|---|---|---|---|---|---|---|
| "15%" | sales | 15 percent | Tax 15% | Tax Received, code `2510` | Domestic | — |
| "15%" | purchases | 15 percent | Tax 15% | Tax Paid, code `1310` | Domestic | — |
| "0% Exports" | sales | 0 percent | Tax 0% | none | Foreign Trade | the sales tax "15%" |
| "0% Imports" | purchases | 0 percent | Tax 0% | none | Foreign Trade | the purchase tax "15%" |

In each of the four, the two base lines (one for the invoice, one for the credit note) carry the whole hundred percent of the base and no account, and the two tax lines carry the whole hundred percent of the tax and the account named above. The two zero-rate taxes have no account at all on their tax lines, which is why a zero-rate document produces no tax item. The tax mechanism itself is specified in `../taxes/`.

**The two fiscal positions.**

| Order | Name | Country | Applied automatically |
|---|---|---|---|
| 10 | "Domestic" | United States | yes |
| 20 | "Foreign Trade" | none, so every country | yes |

Because both are applied automatically and the domestic one is tried first, a counterpart in the United States gets the fifteen-percent taxes and every other counterpart gets the zero-rate ones. Fiscal positions are specified in `../taxes/`.

**What the chart does not ship.** The generic chart ships no account group, no account tag of its own beyond the three cash-flow tags listed above, no reconciliation model of its own and no journal of its own: the journals and the reconciliation models listed earlier in this section are created by the loading mechanism for every chart, generic or not, and the utility accounts listed earlier are created by the same mechanism when the chart does not supply them.

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
| Lock Exception | internal user | — | yes | — | — |
| Lock Exception | Administrator | yes | yes | — | — |

An ordinary internal user can therefore **read** the exceptions but can neither create nor modify nor delete one; the Administrator can read them and create them, but cannot modify an existing one and cannot delete one. Revocation is not a deletion: it is a state change performed by a dedicated operation that runs with elevated rights, and it is additionally guarded by the permission check of `business-rules.md`.

### Wizards

| Entity | Group | C | R | U | D |
|---|---|---|---|---|---|
| Reversal wizard | Invoicing | yes | yes | yes | — |
| Validate entries wizard | Invoicing | yes | yes | yes | — |
| Automatic transfer wizard | Show Full Accounting Features | yes | yes | yes | — |
| Resequence wizard | Administrator | yes | yes | yes | — |
| Secure entries wizard | Administrator | yes | yes | yes | — |
| Account merge wizard | Administrator | yes | yes | yes | **yes** |
| Account merge wizard line | Administrator | yes | yes | yes | **yes** |
| Fiscal year opening wizard | Administrator | yes | yes | yes | — |

The account merge wizard and its line are the only two wizards of the domain that also carry delete rights: the dialogue rebuilds its whole list of lines whenever the grouping switch or the selection of accounts changes, which means removing the previous lines, and that removal goes through the ordinary delete right. Every other wizard of the table builds its rows once and never removes them, so create, read and update are enough. The window action that opens the account merge wizard is in addition restricted to the Administrator group, so the dialogue cannot be reached at all by a lower group.

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
