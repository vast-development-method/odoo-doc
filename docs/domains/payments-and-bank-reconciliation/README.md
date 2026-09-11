# Payments and Bank Reconciliation

## Scope

This domain specifies everything that happens between a business document (a customer invoice, a vendor bill, an expense report, a point-of-sale session) and the money that settles it: the Payment record and the journal entry it produces, the register-payment flow that creates payments in bulk from selected journal items, the bank statements and bank transactions that carry what the bank actually reports, the reconciliation of transactions against outstanding journal items, the reconciliation models that pre-fill counterpart lines and automate matching, the outstanding accounts that make the difference between "money promised" and "money on the bank account", the bank accounts and banks that identify the counterparties, and the check arithmetic that validates an international bank account number or a structured payment reference before it is used.

It sits between three neighbours. It consumes receivable and payable journal items produced by the receivable and payable domains. It produces liquidity journal entries that live under the rules of the general ledger domain. It produces and consumes reconciliations, whose core mechanics (partial reconciliation records, residual computation, full-reconcile detection) are the general ledger's, while the operational use of reconciliation (settling an invoice with a payment, matching a bank transaction) is specified here.

Everything in this folder is derived from the behavior of the core accounting capability package (the payment model, the payment method and payment method line models, the bank statement and bank transaction models, the reconciliation model and its lines, the partial and full reconciliation models, the reconciliation operations on journal items, the journal model in its liquidity aspects, the journal dashboard aggregation for liquidity journals, the register-payment wizard and the bank setup wizard), from the platform's bank and bank-account models, from the international bank account number capability package, from the two quick response code capability packages, and from the capability package that links payments to online payment transactions.

## What is in scope and what is not

In scope:

| Subject | Where specified |
|---|---|
| Payment: every field, the five-state machine, the journal entry it generates, the account selection rules, the outstanding account, the destination account, the reconciliation flags, the link to invoices and to bank transactions, duplicate detection, cancellation and reset | `entities.md`, `state-machines.md`, `workflows.md`, `accounting-effects.md`, `business-rules.md` |
| Payment methods and payment method lines: the registry, the three multiplicity modes, the eligibility domain per journal, the payment account, the uniqueness rules, deletion behavior | `entities.md`, `business-rules.md`, `configuration.md` |
| The register-payment flow in full: the eligible-line filter, the batch grouping key, the batch merge rule, the wizard fields, the installment modes, the early payment discount mode, the payment difference and its write-off, group payments versus one payment per document, the creation, posting and immediate reconciliation of the produced payments | `workflows.md`, `calculations.md`, `business-rules.md`, `accounting-effects.md` |
| Internal transfers between liquidity journals | `workflows.md`, `accounting-effects.md` |
| Bank statements: the running balance chain, the starting and ending balances, completeness, validity, the anchoring rule, statement splitting, deletion protection | `entities.md`, `calculations.md`, `business-rules.md` |
| Bank transactions (statement lines): the internal ordering index, the automatic journal entry with its liquidity line and its suspense line, the foreign-currency triple, the residual amount, the reconciled flag, the synchronization both ways with the journal entry, undoing a reconciliation, partner and bank-account detection | `entities.md`, `calculations.md`, `workflows.md`, `accounting-effects.md` |
| Reconciliation models: every matching condition, the ordering of models, the counterpart line kinds with their four amount modes, the automatic trigger, the partner mapping shortcut, the proposal eligibility, the shipped models | `entities.md`, `calculations.md`, `workflows.md`, `configuration.md` |
| The reconciliation operation itself: the plan, the ordering, the currency split, the per-pair partial computation, exchange differences, full reconcile creation, the matching number graph, unreconciliation | `calculations.md`, `workflows.md`, `accounting-effects.md` |
| Outstanding accounts and the distinction between "in payment" and "paid" | `workflows.md`, `accounting-effects.md`, `glossary.md` |
| Structured payment references: the seven national and international check algorithms written as arithmetic | `calculations.md` |
| International bank account number validation written as arithmetic, the per-country length and mask table, the derivation of the basic bank account number and of the bank, branch and account parts | `calculations.md`, `entities.md` |
| Bank accounts and banks: fields, sanitization, uniqueness, the trust flag and who may set it, the warnings, archival instead of deletion, the find-or-create algorithm | `entities.md`, `business-rules.md` |
| Quick response code payment data: the two shipped generators, their eligibility conditions, their field-by-field payload and their checksum arithmetic | `calculations.md`, `interfaces.md` |
| The bank setup wizard and the journal it creates | `workflows.md`, `interfaces.md` |
| Dashboard figures for liquidity journals and how each one is computed | `calculations.md`, `interfaces.md` |
| Settings, sequences, default records, security groups, the access rights matrix, record rules | `configuration.md` |

Out of scope, specified in a neighbouring folder:

| Subject | Folder |
|---|---|
| Customer invoices, credit notes, payment terms, early payment discount definition and its three computation modes, the payment state of an invoice | `../accounts-receivable/` |
| Vendor bills, vendor refunds, check printing and the amount-in-words algorithm | `../accounts-payable/` |
| Chart of accounts, journals in general, journal entries, journal items, posting, numbering, lock dates, the hash chain, the generic reconciliation entities | `../general-ledger/` |
| Currencies, rates, the rounding arithmetic, the conversion algorithm, the exchange-difference accounts | `../multi-currency/` |
| Taxes on reconciliation-model counterpart lines, cash-basis tax entries triggered by reconciliation | `../taxes/` |
| Analytic distribution on counterpart lines | `../analytic-accounting/` |
| Online payment providers, payment tokens, payment transactions, refunds through a provider | `../payment-providers/` |
| Aged receivable and payable reports, bank reconciliation reports | `../financial-reporting/` |

## Capabilities covered

1. Record an inbound or outbound payment, by itself or from one or several open receivable or payable journal items, in any currency, through any configured payment method, and let it either sit in an outstanding account until the bank confirms it or land directly on the bank account.
2. Register a payment for many documents at once, either one payment per document or one grouped payment per counterparty, with an automatically proposed amount that follows the installment schedule and the early payment discount of the documents, and an explicit handling of any difference between the proposed amount and the amount actually paid.
3. Move money between two liquidity journals of the same company through an intermediate transfer account.
4. Record what the bank reports, either transaction by transaction or as a statement with a starting and an ending balance, keep a running balance per journal, and detect both an incomplete statement and a broken chain of statements.
5. Match a bank transaction with the journal items it settles, with an existing payment, or with a freshly created counterpart line, and undo that match.
6. Define reusable reconciliation models that recognise a transaction by journal, amount, label or counterparty and propose or apply a set of counterpart lines whose amounts are fixed, a percentage of the open balance, a percentage of the transaction, or extracted from the transaction label.
7. Validate and store bank identification data: bank accounts with their sanitized number, their type, their owner and their trust flag; banks with their bank identifier code; and payment references that carry their own check digits.
8. Produce a payable quick response code for an outbound payment when the recipient bank account and the currency allow it.

## Entities

| Entity | Transport name | Purpose |
|---|---|---|
| Payment | `account.payment` | One movement of money to or from a counterparty, with its own journal entry, its state, its method and its links to the documents it settles. |
| Payment Method | `account.payment.method` | The catalogue entry for a way of paying or being paid, identified by a code and a direction. |
| Payment Method Line | `account.payment.method.line` | The activation of a payment method on one journal, carrying the outstanding account to use and the label to display. |
| Bank Statement | `account.bank.statement` | A named checkpoint over a contiguous run of bank transactions of one journal, with a starting balance and an ending balance reported by the bank. |
| Bank Transaction | `account.bank.statement.line` | One line of what the bank reports, owning a journal entry with a liquidity line and a suspense line until it is reconciled. |
| Reconciliation Model | `account.reconcile.model` | A named preset of matching conditions and counterpart lines used while reconciling bank transactions. |
| Reconciliation Model Line | `account.reconcile.model.line` | One counterpart line of a reconciliation model, with its account, its label, its taxes and its amount rule. |
| Partial Reconciliation | `account.partial.reconcile` | The link between exactly one debit journal item and one credit journal item, carrying the matched amount in three currencies. |
| Full Reconciliation | `account.full.reconcile` | The marker created when a connected set of matched journal items has no residual left; it gives the set its permanent matching number. |
| Bank Account | `res.partner.bank` | An account number owned by a partner, sanitized and typed, optionally trusted for outgoing payments and optionally linked to a journal. |
| Bank | `res.bank` | A financial institution with its address and its bank identifier code. |
| Payment Register | `account.payment.register` | The transient record that collects the user's choices while turning selected journal items into payments. |
| Bank Setup Wizard | `account.setup.bank.manual.config` | The transient record that creates a company bank account and the journal that goes with it. |

Two further entities of the general ledger are used constantly here and are only summarised in `entities.md`: Journal (`account.journal`) in its liquidity aspects, and Journal Item (`account.move.line`) in its reconciliation aspects.

## Reading order

1. `glossary.md` — the vocabulary. Outstanding account, suspense account, liquidity line, residual, matching number, transaction versus statement. Read it first if any of those terms is unfamiliar.
2. `entities.md` — the data model: every field of every entity, with its type, its default, its computation and its constraints.
3. `state-machines.md` — the state of a Payment, the state of the journal entry behind a bank transaction, the reconciliation status fields, and the derived payment state of a settled document.
4. `calculations.md` — the arithmetic: residuals, the reconciliation pair algorithm, exchange differences, the running balance, the installment and early-discount amounts, the check-digit algorithms, the quick response code payloads, the dashboard figures.
5. `accounting-effects.md` — every journal entry this domain writes, line by line, with account selection, side and amount.
6. `workflows.md` — the operations end to end, in order, with the records created or updated at each step.
7. `business-rules.md` — every validation, every invariant, every error message, every permission check.
8. `configuration.md` — settings, sequences, default records, groups, access rights, record rules.
9. `interfaces.md` — menus, views, buttons, named operations, reports, exports.
10. `acceptance-criteria.md` — numbered scenarios with concrete numbers that a rebuilt system must reproduce.

## Dependencies on other domains

| Dependency | What this domain needs from it |
|---|---|
| `../general-ledger/` | Journal Entry and Journal Item semantics, the posting algorithm, the balance invariant, accounting dates and lock dates, the account model with its `reconcile` (reconcilable) flag and its account types, the partial and full reconciliation records, the sequence mixin for numbering the liquidity entries. |
| `../multi-currency/` | The rounding function of a currency, the zero test, the comparison, the rate lookup for a date and company, and the conversion between two currencies. Every formula in `calculations.md` that says "round to the currency" means the rounding rule defined there. |
| `../accounts-receivable/` | The receivable journal items that a payment settles, the payment state of an invoice, the early payment discount fields on a payment term line and the counterpart lines a discount produces. |
| `../accounts-payable/` | The payable journal items that a payment settles, and the vendor bank account selection. |
| `../taxes/` | Tax computation on a reconciliation-model counterpart line, and the cash-basis tax entries triggered when a reconciliation touches a receivable or payable account of a company that uses cash-basis taxes. |
| `../analytic-accounting/` | The analytic distribution stored on a reconciliation-model counterpart line and copied to the journal item it creates. |
| `../payment-providers/` | The online transaction that may drive a payment's state, the saved token that may be charged when a payment is posted, and the refund flow. Only the attachment points are described here. |

## Conventions used in this folder

- Amounts are written in the currency they belong to. When a formula mixes currencies, each quantity names its currency explicitly.
- "Round to the currency" always means: round the value to the number of decimal places of that currency, half away from zero, on the currency's rounding multiple, exactly as specified in `../multi-currency/calculations.md`.
- Debit and credit are always given from the point of view of the company keeping the books. A positive balance is a debit; a negative balance is a credit.
- A *signed amount in foreign currency* on a journal item is positive on the debit side and negative on the credit side, in every currency.
- Where the behavior of a screen is only observable through the user interface and the stored model leaves it implicit, the text says so and gives the industry-standard default.
