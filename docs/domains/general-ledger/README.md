# General Ledger

## Scope

This domain specifies the double-entry accounting core of the system: the chart of accounts, the journals, the journal entries and their journal items, the way an entry becomes permanent (posting, numbering, sequence integrity, locking, hashing), the way it can be undone (reversal, reset to draft, cancellation), and the way journal items are matched against each other (reconciliation, partial and full, with the residual amounts in both the company currency and the foreign currency of the item).

It is the foundation on which every other accounting domain rests. Customer invoices, vendor bills, payments, bank statements, point-of-sale sessions, inventory valuation, payroll and expense reports all end in a Journal Entry built by the rules of this domain, numbered by the rules of this domain, checked by the constraints of this domain and posted by the algorithm of this domain.

Everything in this folder is derived from the behavior of the core Accounting capability package: the account model, the account group and account root models, the account tag model, the journal and journal-group models, the journal entry and journal item models, the full and partial reconciliation models, the lock exception model, the automatic sequence mixin, the company accounting settings, the partner accounting fields, the journal dashboard aggregation, the chart-template loading mechanism, and the wizards for reversal, automatic transfer entries, resequencing, entry validation and entry securing, together with the shipped security groups, access rights, record rules, sequences, scheduled jobs and system parameters of that package.

## What is in scope and what is not

In scope:

| Subject | Where specified |
|---|---|
| Chart of accounts: account types, internal groups, roots, groups, tags, reconcilable flag, archival, company-dependent code, currency restriction, non-trade flag, opening balances, unmerge | `entities.md`, `business-rules.md`, `calculations.md`, `configuration.md` |
| Journals: all six types, default account, suspense account, profit and loss accounts, outstanding accounts through payment method lines, sequence prefix, sequence override expression, dedicated credit-note and payment sequences, restricted hash mode, ledger groups, incoming document alias, dashboard figures | `entities.md`, `business-rules.md`, `calculations.md`, `configuration.md`, `interfaces.md` |
| Journal entries: the three-state machine, the balance invariant, the posting algorithm step by step, the accounting date derivation, every kind of lock date and its check, lock date exceptions, the numbering grammar with its five reset periodicities, gap detection, resequencing, the inalterability hash chain with its exact input string and ordering, the audit trail, the reversal methods, automatic and recurring posting, cancellation requests, deletion rules | `state-machines.md`, `workflows.md`, `business-rules.md`, `calculations.md`, `configuration.md` |
| Journal items: account, partner, debit, credit, balance, foreign-currency amount, currency, dates, display types, matching number, residual amounts, tax and analytic hooks, deletion and modification rules | `entities.md`, `business-rules.md`, `calculations.md` |
| Reconciliation core: the matching algorithm, partial reconciliations, residuals in both currencies, full-reconcile detection, the matching number, unreconciliation, the exchange-difference hook | `calculations.md`, `workflows.md`, `accounting-effects.md` |
| Fiscal years, opening entries and the current-year-earnings account | `calculations.md`, `workflows.md`, `accounting-effects.md` |
| Company accounting settings that belong to the ledger, the accounting onboarding steps, the security groups, the access rights matrix, the record rules and the scheduled jobs | `configuration.md` |
| Automatic transfer entries, resequencing, entry validation, entry securing and the hash integrity report | `workflows.md`, `interfaces.md`, `calculations.md` |
| The chart-template loading mechanism (how a template is selected, what it ships and how it is applied) | `workflows.md`, `configuration.md` |

Out of scope, specified in a neighbouring folder:

| Subject | Folder |
|---|---|
| Customer invoices, credit notes, receipts, payment terms, cash rounding, invoice sending, portal payment | `../accounts-receivable/` |
| Vendor bills, vendor refunds, purchase receipts, bill upload and decoding, check printing | `../accounts-payable/` |
| Payments, payment methods, bank statements, reconciliation models, the payment register | `../payments-and-bank-reconciliation/` |
| Taxes, tax groups, tax distribution, fiscal positions, tax grids, cash-basis tax entries | `../taxes/` |
| Currencies, rates, rounding arithmetic, exchange-difference amounts | `../multi-currency/` |
| Analytic plans, analytic accounts, analytic distribution models, analytic lines | `../analytic-accounting/` |
| The report engine, the shipped financial statements and the tax closing entry | `../financial-reporting/` |
| Per-country chart templates, their accounts, taxes and legal reports | `../fiscal-localizations/` |

This folder describes the hooks those domains attach to (the analytic distribution field on a journal item, the tax grid field, the exchange-difference creation point, the report drill-down) but not their content.

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Account | `account.account` | `account_account` | One line of the chart of accounts: a code per company, a name, a type, a reconcilable flag |
| Account Group | `account.group` | `account_group` | A code-prefix range grouping accounts into a hierarchy for reporting |
| Account Root | `account.root` | none (computed view) | The first one or two characters of an account code, used as a search facet |
| Account Tag | `account.account.tag` | `account_account_tag` | A free label attachable to accounts, taxes or products for custom reporting |
| Account Code Mapping | `account.code.mapping` | `account_code_mapping` | The code of one account in one company, shown as an editable row |
| Journal | `account.journal` | `account_journal` | A book of entries with its own numbering prefix, default accounts and settings |
| Journal Group | `account.journal.group` | `account_journal_group` | A named selection of journals (by exclusion) used as a report filter |
| Journal Entry | `account.move` | `account_move` | One balanced accounting document: header, state, number, date, lines |
| Journal Item | `account.move.line` | `account_move_line` | One debit or credit posting of a Journal Entry on one account |
| Partial Reconciliation | `account.partial.reconcile` | `account_partial_reconcile` | One matched amount between exactly one debit item and one credit item |
| Full Reconciliation | `account.full.reconcile` | `account_full_reconcile` | The marker created when a set of matched items nets to zero, carrying the matching number |
| Lock Exception | `account.lock_exception` | `account_lock_exception` | A time-limited or user-limited relaxation of one lock date |
| Automatic Sequence (abstract) | `sequence.mixin` | none | The shared numbering behavior: prefix, period, counter, gap detection |
| Company accounting settings | `res.company` | `res_company` | The lock dates, the fiscal year definition, the default accounts, the opening entry |
| Partner accounting fields | `res.partner` | `res_partner` | The receivable and payable accounts and the ledger totals of a counterpart |
| Reversal wizard | `account.move.reversal` | transient | Collects the reversal parameters and creates the reverse entries |
| Automatic transfer wizard | `account.automatic.entry.wizard` | transient | Moves amounts between accounts or between periods with two mirrored entries |
| Resequence wizard | `account.resequence.wizard` | transient | Renumbers a selected set of entries in a chosen order |
| Validate entries wizard | `validate.account.move` | transient | Posts a selected set of draft entries in bulk |
| Secure entries wizard | `account.secure.entries.wizard` | transient | Hashes all eligible entries up to a chosen date |
| Change lock date wizard | `account.change.lock.date` | transient | Edits the company lock dates and records the change in the audit trail |
| Chart template mechanism | `account.chart.template` | none (abstract) | Loads a country chart of accounts and all the records that come with it |

## Reading order

1. `glossary.md` — the vocabulary: balance, residual, reconciliation, sequence chain, lock date, hash chain, storno.
2. `entities.md` — the data model, field by field.
3. `state-machines.md` — the entry state machine, the payment status, the sequence and hash lifecycles.
4. `calculations.md` — the balance and residual arithmetic, the numbering grammar, the reconciliation algorithm, the hash input string.
5. `business-rules.md` — every constraint and its exact message, every permission check, every lock check.
6. `workflows.md` — the operational sequences end to end.
7. `accounting-effects.md` — the entries this domain itself creates.
8. `configuration.md` — settings, sequences, groups, access rights, record rules, scheduled jobs.
9. `interfaces.md` — menus, views, remote operations, routes, reports.
10. `acceptance-criteria.md` — the scenarios an implementation must pass.

## Dependencies on other domains

| Domain | Dependency |
|---|---|
| `../multi-currency/` | Currency rounding, decimal places, rate lookup; every monetary field is rounded with the rules of that domain |
| `../taxes/` | Tax lines are journal items; tax grids are stored on journal items; the tax lock date is checked here |
| `../analytic-accounting/` | The analytic distribution field on a journal item and the analytic lines derived from it |
| `../accounts-receivable/`, `../accounts-payable/` | Invoice and bill documents are Journal Entries with a commercial type; their dynamic lines are built on the journal-item model specified here |
| `../payments-and-bank-reconciliation/` | Payments and statement lines produce Journal Entries and consume the reconciliation algorithm specified here |
| `../financial-reporting/` | Reads posted journal items; the audit drill-down and the hash integrity check are specified there but rely on the chain defined here |
| `../messaging-and-activities/` | The audit trail is a message thread; tracked field changes and the cancellation-request activity use that domain |
| `../contacts-and-organizations/` | Partners, their commercial entity, their bank accounts and their company hierarchy |

## Conventions used in this folder

- Every amount expressed "in company currency" is rounded to the decimal precision of the company currency; every amount "in foreign currency" is rounded to the decimal precision of that currency. Rounding is round-half-away-from-zero on the currency rounding step unless stated otherwise.
- "Signed" means multiplied by the direction sign of the document (defined in `calculations.md`).
- A *debit* item has a positive balance; a *credit* item has a negative balance. Exactly one of the two stored columns `debit` and `credit` is non-zero for a normal item, with the exception described under storno accounting.
- Dates are calendar dates without a time component unless stated otherwise.
- Wherever the source of a behavior leaves a detail unspecified and this document fills it with the common practice of accounting software, the sentence is marked with the phrase **industry-standard default**.
