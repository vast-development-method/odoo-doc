# Configuration

Settings, system parameters, sequences and numbering formats, default records, security groups, the access rights matrix, record rules and scheduled jobs of the payments and bank reconciliation domain.

---

## 1. Company settings

All of these are fields of the Company record, surfaced through the accounting settings screen.

| Setting (storage name) | Type | Meaning and default |
|---|---|---|
| `account_journal_suspense_account_id` | link to Account | "Bank Suspense". The account every Bank Transaction posts its unexplained side on, and the fallback for a liquidity journal that has no suspense account of its own. Restricted to accounts of type `asset_current` or `liability_current`. Help: bank transactions are posted immediately after import or synchronisation; their counterpart is the bank suspense account; reconciliation replaces it by the definitive accounts. |
| `transfer_account_id` | link to Account | "Internal Transfer". The intermediary account used when moving money from one liquidity account to another. Restricted to accounts that are reconcilable **and** of type `asset_current`. Also used as the last-resort outstanding account when a Payment must produce accounting and no outstanding account can be found. |
| `currency_exchange_journal_id` | link to Journal | The journal every exchange-difference entry is written in. |
| `income_currency_exchange_account_id` | link to Account | The gain account of an exchange difference: used when the drift is negative. |
| `expense_currency_exchange_account_id` | link to Account | The loss account of an exchange difference: used when the drift is positive. |
| `account_journal_early_pay_discount_gain_account_id` | link to Account | "Cash Discount Write-Off Gain Account". Used for the discount counterpart of an outbound document (a vendor bill). |
| `account_journal_early_pay_discount_loss_account_id` | link to Account | "Cash Discount Write-Off Loss Account". Used for the discount counterpart of an inbound document (a customer invoice). |
| `tax_cash_basis_journal_id` | link to Journal | The journal every cash-basis tax entry produced by a reconciliation is written in. |
| `tax_exigibility` | boolean | "Cash Basis". When on, a reconciliation that touches a receivable or payable account also produces cash-basis tax entries. Delegated from the root company. |
| `bank_account_code_prefix` | text | The prefix used when the chart template creates the bank, suspense and outstanding accounts. |
| `cash_account_code_prefix` | text | The prefix used when the chart template creates the cash accounts. |
| `transfer_account_code_prefix` | text | The prefix used when the chart template creates the inter-bank transfer account. |
| `batch_payment_sequence_id` | link to Number Sequence | Read-only, not copied. The sequence that numbers a grouped payment's communication. Created automatically (section 3.2). |
| `restrictive_audit_trail` | boolean | When on, deleting a Bank Transaction cancels its entry instead of deleting it. |

Capability switches that belong to this domain:

| Switch (storage name) | Effect when turned on |
|---|---|
| `module_account_payment` | "Invoice Online Payment": installs the capability that links Payments to online payment transactions and tokens. |
| `module_account_check_printing` | "Allow check printing and deposits": adds a check payment method, per-journal check numbering and the printing flow. |
| `module_account_batch_payment` | "Use batch payments": allows grouping payments into a single batch and eases reconciliation. |
| `module_account_iso20022` | Adds the credit-transfer payment methods that produce a transfer file. |
| `module_account_sepa_direct_debit` | Adds the direct-debit payment method and its mandates. |
| `module_account_bank_statement_import_qif` | Adds an import format for bank statements and a matching bank-feed source. |
| `module_account_bank_statement_extract` | "Bank Statement Digitization": adds automatic reading of scanned statements. Only settable once document digitisation is installed. |

---

## 2. System parameters

| Parameter | Values | Effect |
|---|---|---|
| `account.skip_create_bank_account_on_reconcile` | any value that reads as true or false | When true, reconciling a bank transaction never creates a Bank Account from the account number the bank reported; it only searches for an existing one matching the number, the counterparty, and a company that is empty or the transaction's company. Default: absent, which means the account is created. |

---

## 3. Sequences and numbering formats

### 3.1 The payment sequence

| Property | Value |
|---|---|
| Code | `account.payment` |
| Name | Payment |
| Prefix | `PAY` |
| Padding | 5 |
| Company | none — the sequence is shared, and is drawn in the Payment's company context |

Format: `PAY` followed by the counter on five digits, for example `PAY00001`.

This sequence is only used for a Payment that has **no** Journal Entry, that is, a Payment whose payment method line has no payment account. Every other Payment takes its number from its Journal Entry, which is numbered by the journal's own sequence under the rules of `../general-ledger/`.

### 3.2 The grouped-payment communication sequence

One per company, created automatically: when the accounting capability is installed, for every company that has none; and whenever a company is created.

| Property | Value |
|---|---|
| Name | Group Payments Number Sequence |
| Implementation | gapless |
| Padding | 5 |
| Date ranges | used |
| Company | the company itself |
| Prefix | `GROUP/<the range year>/` |

Format: `GROUP/`, the year of the date range, `/`, the counter on five digits, for example `GROUP/2026/00001`.

Drawn when a payment is registered for several documents that are **not** outbound documents and that do not all belong to one document: the resulting memo is the next value of this sequence.

### 3.3 Bank statement naming

A Bank Statement's reference is not drawn from a sequence; it is computed:

```formula
name = ( the journal's sequence prefix ‖ " " )   when the statement has a journal, otherwise ""
     ‖ "Statement "
     ‖ the statement's date, or the date part of its creation timestamp when it has no date
```

Example: `BNK1 Statement 2026-03-31`.

### 3.4 Bank transaction ordering

A Bank Transaction has no number of its own; its Journal Entry is numbered by the journal's sequence. Its ordering key is the computed index of `calculations.md`, section 7.1.

---

## 4. Default records shipped with the domain

### 4.1 Payment methods

Two records, created at installation:

| Reference name | Name | Code | Direction |
|---|---|---|---|
| the manual inbound method | Manual Payment | `manual` | `inbound` |
| the manual outbound method | Manual Payment | `manual` | `outbound` |

Their registry entry is: mode `multi`, journal types `bank`, `cash` and `credit`, no currency restriction, no country restriction. Because the mode is `multi`, one Payment Method Line is created automatically on every liquidity journal for each of them, and a journal may carry several lines of the same method under different names.

### 4.2 Payment method lines on a journal

Every liquidity journal is created with:

- one inbound line per default inbound method — that is, one line for the manual inbound method;
- one outbound line per default outbound method — that is, one line for the manual outbound method.

Each line takes the method's name. When the journal already had a line for that method, the payment account of the old line is preserved on the new one.

### 4.3 Accounts created by the chart template

Created once per root company when a chart of accounts is loaded. A branch company inherits the values of its first ancestor instead of creating its own.

| Reference name | Label | Prefix | Type | Reconcilable | Set on |
|---|---|---|---|---|---|
| bank suspense | Bank Suspense Account | the company's bank-account code prefix | `asset_current` | no | the company's suspense-account setting, and every liquidity journal that has none |
| outstanding receipts | Outstanding Receipts | the company's bank-account code prefix | `asset_current` | **yes** | not on the company; referenced by name when an outstanding account must be forced onto an inbound Payment |
| outstanding payments | Outstanding Payments | the company's bank-account code prefix | `asset_current` | **yes** | same, for an outbound Payment |
| liquidity transfer | Liquidity Transfer | the company's transfer-account code prefix | `asset_current` | **yes** | the company's inter-bank transfer account |
| cash difference gain | Cash Difference Gain | prefix `999` | `income_other` | no | the company's cash-difference income account, and every liquidity journal's profit account |
| cash difference loss | Cash Difference Loss | prefix `999` | `expense` | no | the company's cash-difference expense account, and every liquidity journal's loss account |
| cash discount loss | Cash Discount Loss | code `999998` | `expense` | no | the company's early-discount loss account |
| cash discount gain | Cash Discount Gain | code `999999` (the counterpart of the loss account) | `income_other` | no | the company's early-discount gain account |

The two outstanding accounts are created without being stored on any company field: they are looked up by reference name when needed.

After the accounts are created, the loading also sets, on every journal of type `cash`, `bank` or `credit` of the company that has none: the suspense account from the company's setting, the profit account from the cash-difference income account, and the loss account from the cash-difference expense account.

### 4.4 Reconciliation models shipped with the chart template

Two models are created per company when a chart of accounts is loaded.

| Reference name | Name | Conditions | Lines |
|---|---|---|---|
| internal transfer | Internal Transfers | none | one line: amount mode `percentage`, amount `100`, label "Internal Transfers"; its account is set, after loading, to the company's **inter-bank transfer account** |
| bank fees | Bank Fees | label condition `contains`, parameter `Bank Fees` | one line: amount mode `percentage`, amount `100`, label "Bank Fees"; its account is set, after loading, to the bank-fees account chosen by the localisation |

Because the internal-transfer model has no condition at all and is not automated, its *can be proposed* flag is false: it is only usable by choosing it explicitly.

### 4.5 Reconciliation models shipped with the demonstration data

| Name | Conditions | Lines |
|---|---|---|
| Line with Bank Fees | label condition `contains`, parameter `BRT` | line 1: label "Due amount", an income account, amount mode `regex`, amount text `BRT: ([\d,.]+)`; line 2: label "Bank Fees", a finance-expense account, amount mode `percentage`, amount text `100` |
| Owner's Current Account | none | one line: label "Owner's Current Account", an account of type `asset_receivable`, amount mode `percentage`, amount text `100` |

### 4.6 Printable documents and templates

| Reference name | Kind | Bound to | Content |
|---|---|---|---|
| Payment Receipt | printable document, portable-document format, offered from the Payment's print menu | Payment | Section 6 of `interfaces.md`. |
| Statement | printable document, portable-document format, with a dedicated page format | Bank Statement | Section 6 of `interfaces.md`. |
| A4 - statement | page format | — | A4 portrait, top margin 52, bottom margin 32, no side margins, no header line, header spacing 52, 90 dots per inch, margins expressed in the style sheet. Marked as the default page format. |
| the payment-receipt message template | electronic-mail template | Payment | Used by the *send receipt by electronic mail* operation. |

---

## 5. Security groups

| Group | Name | Implies | What it grants in this domain |
|---|---|---|---|
| billing | Invoicing | the internal-user group | Create and edit Payments, use the register-payment screen, read bank statements and transactions, read and update reconciliation models, create and delete payment methods, create and edit payment method lines, create and delete reconciliations. |
| basic accounting | Basic | billing | Create, edit and delete bank statements and bank transactions; create and delete reconciliation models and their lines. |
| accounting reader | Show Accounting Features - Readonly | the internal-user group | Read everything of this domain, change nothing. |
| accountant | Show Full Accounting Features | basic accounting and accounting reader | Everything above plus the reconciliation of journal items. |
| accounting administrator | Administrator | billing | Everything, plus the configuration screens: journals, settings, the bank setup screen. |
| bank-account validator | Validate bank account | implied by the system-administrator group | Set and clear the *trusted for outgoing payments* flag on a Bank Account. Shown under the *Bank* privilege of the accounting category. |

Note the shape of the hierarchy: the billing group is **not** implied by the accountant group directly; the accountant group implies the basic accounting group which implies billing.

---

## 6. Access rights matrix

Read, create, update, delete, per group. A blank cell means not granted by that line.

| Entity | Group | Read | Create | Update | Delete |
|---|---|---|---|---|---|
| Payment | accounting reader | yes | | | |
| Payment | billing | yes | yes | yes | yes |
| Payment Method | internal user | yes | | | |
| Payment Method | billing | yes | yes | | yes |
| Payment Method Line | internal user | yes | | | |
| Payment Method Line | billing | yes | yes | yes | yes |
| Bank Statement | accounting reader | yes | | | |
| Bank Statement | billing | yes | | | |
| Bank Statement | basic accounting | yes | yes | yes | yes |
| Bank Transaction | accounting reader | yes | | | |
| Bank Transaction | billing | yes | | | |
| Bank Transaction | basic accounting | yes | yes | yes | yes |
| Reconciliation Model | accounting reader | yes | | | |
| Reconciliation Model | billing | yes | | yes | |
| Reconciliation Model | basic accounting | yes | yes | yes | yes |
| Reconciliation Model Line | accounting reader | yes | | | |
| Reconciliation Model Line | billing | yes | | yes | |
| Reconciliation Model Line | basic accounting | yes | yes | yes | yes |
| Partial Reconciliation | accounting reader | yes | | | |
| Partial Reconciliation | billing | yes | yes | yes | yes |
| Partial Reconciliation | accountant | yes | yes | yes | yes |
| Full Reconciliation | accounting reader | yes | | | |
| Full Reconciliation | billing | yes | yes | yes | yes |
| Full Reconciliation | accountant | yes | yes | yes | yes |
| Payment Register (transient) | billing | yes | yes | yes | |
| Bank Setup Wizard (transient) | accounting administrator | yes | yes | yes | |
| Bank Account | internal user | yes | | | |
| Bank Account | contact manager | yes | yes | yes | yes |
| Bank | internal user | yes | | | |
| Bank | contact manager | yes | yes | yes | yes |
| Bank | system administrator | yes | yes | yes | yes |

Notice that **updating a Payment Method is granted to no group**: the catalogue entries are created and deleted, never edited.

---

## 7. Record rules

| Entity | Rule name | Applies to | Condition |
|---|---|---|---|
| Payment | Account payment company rule | everyone | the Payment's company is one of the user's active companies |
| Bank Statement | Account bank statement company rule | everyone | the statement's company is one of the user's active companies, **or is empty** |
| Bank Transaction | Account bank statement line company rule | everyone | the transaction's company is one of the user's active companies |
| Reconciliation Model | Account reconcile model template company rule | everyone | the model's company is an ancestor of one of the user's active companies |
| Reconciliation Model Line | Account reconcile model_line template company rule | everyone | the line's company is an ancestor of one of the user's active companies |
| Bank Account | Billing: Allow accessing employee bank accounts | the billing group | unrestricted — the rule grants full visibility, so that a billing officer can see counterparty bank accounts even where another capability restricts the model |

The company domain of a Bank Account, a Payment Method Line and a Journal is *ancestor-of*: a record whose company is an ancestor of the active company is visible from it, and a record with no company is visible everywhere.

---

## 8. Scheduled jobs

**This domain ships no scheduled job of its own.** Two jobs of the surrounding accounting capability touch it indirectly:

| Job | Effect on this domain |
|---|---|
| the automatic posting of draft entries | Posts entries whose automatic-posting date has come. A Payment entry created in draft and marked for automatic posting is posted by it, which moves the Payment forward. |
| the sending of pending documents | Sends queued printable documents and electronic mail, including payment receipts queued for sending. |

Capability packages that synchronise bank feeds add their own scheduled job, which creates Bank Transactions and, usually, Bank Statements.

---

## 9. Onboarding

Two onboarding steps of the accounting dashboard belong to this domain.

| Step | What it opens | What it does |
|---|---|---|
| Bank account | the bank setup screen, opened as a medium dialogue titled "Setup Bank Account" | Creates the company's own Bank Account and the liquidity journal that goes with it (`workflows.md`, section 15). |
| Credit card account | the same screen with the journal type set to `credit`, titled "Setup Credit Card Account" | The same, for a credit-card journal. |

A liquidity journal's dashboard card shows a *Bank Setup* button whenever the journal is of type `bank`, has no bank account, and its bank-feed source is `undefined` or an online synchronisation that has not been configured.

---

## 10. Configuration checklist for a working installation

1. A chart of accounts is loaded, so that the suspense account, the two outstanding accounts, the inter-bank transfer account, the two cash-difference accounts and the two cash-discount accounts exist and are stored on the company.
2. At least one journal of type `bank`, `cash` or `credit` exists, with:
   - a default account of the right type;
   - a suspense account (inherited from the company when not set);
   - at least one inbound and one outbound payment method line, each with a payment account.
3. The company names an exchange-difference journal and both exchange-difference accounts, otherwise any multi-currency reconciliation fails with the messages of `business-rules.md`, section 8.4.
4. When cash-basis taxes are enabled, the company names a cash-basis journal, otherwise any reconciliation on a trade account fails.
5. The company's own partner has a Bank Account for each bank journal, and each such account backs exactly one journal.
6. Counterparty bank accounts used for outgoing payments are trusted by a bank-account validator.
7. For discount handling, the company names both cash-discount accounts.
8. For internal transfers, the company names the inter-bank transfer account and it is reconcilable.
