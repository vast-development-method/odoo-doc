# Financial Reporting

## 1. Purpose of this domain

The Financial Reporting domain is the declarative reporting engine of the accounting
application. It does two things:

1. It stores **report definitions** as ordinary data records — a report, its lines, the
   expressions that give every line its numbers, and the columns that display them. A report
   definition contains no program code of any kind: it contains formulas written in one of six
   small formula languages, each interpreted by a dedicated **computation engine**.
2. It **evaluates** those definitions against the accounting ledger for a chosen period, a chosen
   set of companies, a chosen set of journals and an optional analytic restriction, and returns a
   table of lines and figures that can be unfolded, drilled into, compared with other periods,
   annotated, exported and — for tax reports — turned into a posted closing Journal Entry.

Everything a rebuild needs to reproduce the shipped statements (balance sheet, profit and loss,
cash flow statement, general ledger, trial balance, partner ledger, aged receivable, aged payable,
journal audit, tax report and the one hundred and sixty-five shipped tax return definitions) is
specified here: the exact grammar of every formula language, the exact evaluation semantics of
every engine, the date windows, the sign conventions, the rounding, the carry-over mechanism, the
drill-down contract, the export contract and the tax closing entry.

The domain deliberately keeps the *definition* of a report separate from its *evaluation*. A
rebuild must keep that separation: a report is data, an engine is behavior, and adding a report
for a new country must never require new behavior.

## 2. Capabilities covered

| Capability | Summary |
|---|---|
| Report definition | Named, ordered, optionally country-scoped or chart-of-accounts-scoped report headers, with a per-report switch for every user-facing filter. |
| Variants | A report can declare a root report; the root is the generic, country-neutral model and the variants are the national adaptations selectable from one menu entry. |
| Composite reports | A report can be assembled from sections, each section being a whole report of its own, displayed and printed together. |
| Report lines | A recursive tree of named lines with a level, an ordering, an optional unique code, folding behavior, page-break behavior, zero-hiding and an optional drill-down action. |
| Expressions | Zero or more labelled expressions per line; each expression names a computation engine, a formula, an optional subformula, a date scope and a display type. |
| Computation engines | Six engines: record-filter (`domain`), tax tag (`tax_tags`), aggregation (`aggregation`), account code prefix (`account_codes`), external value (`external`) and custom function (`custom`). |
| Columns | An ordered list of columns; each column binds to an expression label and carries its own display type and its own drill-down action. |
| Date scopes | Six per-expression date windows, from "strictly the requested period" to "everything since the beginning of time" and "at the opening instant of the fiscal year". |
| Filters | Date, comparison, journals, analytic accounts and plans, companies and tax units, draft entries, unreconciled entries, account groups (hierarchy), partners, account type, budgets, saved journal-item filters, zero-hiding, horizontal splitting and rounding unit. |
| Comparisons | Period-over-period comparison with a configurable number of previous periods, and growth comparison expressed as a percentage with a good/bad direction per expression. |
| Grouping | Per-line grouping keys taken from Journal Item fields, both designer-fixed and user-chosen. |
| Unfolding | Lazy expansion of a line into sub-lines, including automatic prefix grouping when a line would expand into too many children. |
| Carry-over | Per-expression transfer of an amount from one period to the next without any Journal Entry, stored as external values. |
| External values | Manually entered figures and machine-generated carry-over figures, dated, company-scoped and attached to one expression. |
| Audit drill-down | From any auditable figure, the exact set of Journal Items that produced it. |
| Tax closing | Generation, from a tax report and a period, of the Journal Entry that clears the tax accounts against the tax payable, tax receivable and tax advance accounts. |
| Data integrity reporting | The hash integrity check over the secured Journal Entry chains, and the audit trail of tracked changes to accounting records. |
| Export | Printable document, spreadsheet workbook and country-specific electronic filing files. |

## 3. Entities of this domain

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Report Definition | `account.report` | `account_report` | The header of a report: its name, availability, filter switches and column set. |
| Report Line | `account.report.line` | `account_report_line` | One row of a report, possibly with children, possibly grouped, possibly foldable. |
| Report Expression | `account.report.expression` | `account_report_expression` | One labelled figure of a line: an engine plus a formula plus a date scope. |
| Report Column | `account.report.column` | `account_report_column` | One displayed column, bound to an expression label. |
| Report External Value | `account.report.external.value` | `account_report_external_value` | A manually entered or carried-over figure attached to one expression at one date for one company. |
| Account Tag | `account.account.tag` | `account_account_tag` | The signed tag that connects a tax repartition line to a tax report line; owned jointly with the Taxes domain but created and destroyed by this domain when tax tag expressions are written. |
| Tax Group | `account.tax.group` | `account_tax_group` | Holds the three accounts the tax closing entry posts to; owned by the Taxes domain and consumed here. |

Two further records are read but never written by this domain: the Journal Item
(`account.move.line`, table `account_move_line`), which is the only source of ledger figures, and
the Account (`account.account`, table `account_account`), which supplies codes, types and tags.

## 4. Reading order

1. [`entities.md`](entities.md) — every field of every entity, its type, its default, its computed
   rule and its constraints. Read this first; the rest of the domain refers to these fields by
   name throughout.
2. [`state-machines.md`](state-machines.md) — the report rendering state machine, the unfolding
   state of a line, the lifecycle of an external value, the tax return lifecycle and the audit
   check lifecycle.
3. [`calculations.md`](calculations.md) — the heart of the domain: the exact grammar and exact
   evaluation semantics of all six engines, the date scope windows, the comparison and growth
   arithmetic, the carry-over arithmetic, the aging buckets and the rounding rules, each with
   worked numeric examples.
4. [`workflows.md`](workflows.md) — how a report is opened, evaluated, unfolded, audited,
   annotated, compared, exported and closed, step by step, with the records created at each step.
5. [`business-rules.md`](business-rules.md) — every validation, every constraint, every error
   message, every access check and every locking rule.
6. [`accounting-effects.md`](accounting-effects.md) — the Journal Entries this domain produces:
   the tax closing entry, the carry-over (which produces none, and why), and the interaction with
   the lock dates.
7. [`configuration.md`](configuration.md) — settings, parameters, sequences, default records,
   security groups, the access rights matrix, record rules and scheduled jobs.
8. [`interfaces.md`](interfaces.md) — menus, views, named remote operations, routes, printable
   documents, export formats and notifications.
9. The shipped catalogue, inside [`calculations.md`](calculations.md) §14 and
   [`interfaces.md`](interfaces.md) §9: the complete enumeration of the one hundred and sixty-five
   shipped report definitions with their line and expression counts, and the full line-by-line
   structure of the statement reports and of a representative set of national tax returns.
10. [`acceptance-criteria.md`](acceptance-criteria.md) — numbered Given/When/Then scenarios with
    concrete amounts, including the five mandatory scenarios: a balance sheet that balances on a
    small chart of accounts, a comparison against the prior period, an aged receivable split
    across buckets at a stated reference date, a tax period closing with its exact entry, and a
    carried-over balance.
11. [`glossary.md`](glossary.md) — every term used in this domain, defined in full.

## 5. Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [General Ledger](../general-ledger/README.md) | The Journal Entry and Journal Item records, their posting state, their accounting date, the chart of accounts with account codes, account types, account groups and account tags, the journals, the fiscal year boundaries, every lock date and the secured-entry hash chain. |
| [Taxes](../taxes/README.md) | The tax records, their repartition lines, the tags those repartition lines stamp onto Journal Items, the tax group and its payable, receivable and advance accounts, the tax exigibility (accrual against payment) flag and the tax unit grouping of companies. |
| [Accounts Receivable](../accounts-receivable/README.md) | The receivable Journal Items, their due dates and their residual amounts, which are the raw material of the aged receivable report and the partner ledger. |
| [Accounts Payable](../accounts-payable/README.md) | The payable Journal Items, their due dates and their residual amounts, for the aged payable report. |
| [Multi-currency](../multi-currency/README.md) | Currency rounding, the rate lookup for a date, and the translation of foreign-currency balances into the presentation currency of the report. |
| [Analytic Accounting](../analytic-accounting/README.md) | The analytic distribution stored on Journal Items, which the analytic filter restricts on, and the analytic accounts and plans offered in that filter. |
| [Payments and Bank Reconciliation](../payments-and-bank-reconciliation/README.md) | The reconciliation state of Journal Items, used by the unreconciled filter and by the aging reports. |

Domains that depend on this one:

| Domain | What it takes from here |
|---|---|
| [Fiscal Localizations](../fiscal-localizations/README.md) | Every national tax return is a report definition of exactly the shape specified here; the localization packages ship the data, this domain ships the behavior. |
| [Spreadsheets and Dashboards](../spreadsheets-and-dashboards/README.md) | The spreadsheet formula that reads an account balance uses the same date-window and sign rules as the account code prefix engine specified here. |

## 6. Principles a rebuild must preserve

1. **Definitions are data.** Adding a report, a line, an expression or a column must never require
   new behavior. Every shipped national tax return is nothing but rows in the four definition
   tables.
2. **One figure, one expression.** A cell of the rendered table is identified by the triple
   (line, expression label, column period). Columns bind to expression labels, never to lines, so
   a line without an expression carrying the column's label simply shows nothing in that column.
3. **Ledger figures are signed balances.** Every engine that reads the ledger reads the signed
   balance of Journal Items, defined as debit minus credit, and applies the sign conventions of
   §"Sign conventions" of [`calculations.md`](calculations.md). No engine reads debit and credit
   separately except where the account code prefix engine's balance-character filter demands it.
4. **Evaluation is batched per engine.** All expressions of a report that use the same engine and
   the same date scope are evaluated in one pass over the ledger. A rebuild is free to choose its
   own query strategy, but it must produce the same figures for a line whether the line is
   evaluated alone or with the whole report.
5. **Aggregation is a dependency graph.** Expressions of the aggregation engine reference other
   expressions by line code and label. The graph must be resolved before evaluation, and a
   reference that cannot be resolved is an authoring error, not a runtime zero.
6. **Nothing is recomputed silently.** Carried-over amounts and manual figures are stored as dated
   records. Reopening a closed period must show the same figures it showed when it was closed,
   except for ledger movements actually recorded since.
7. **Every figure is auditable.** For every engine except the custom function engine, the user
   must be able to obtain the exact set of Journal Items behind the number, and the sum of those
   items must equal the number displayed, up to the report's rounding.

## 7. What this domain does not cover

- The computation of taxes themselves, the repartition of a tax into base and tax lines, and the
  stamping of tags onto Journal Items: see [Taxes](../taxes/README.md).
- The posting, numbering, hashing and locking of Journal Entries: see
  [General Ledger](../general-ledger/README.md). This domain only *reads* posted and draft
  entries, *reports* on the hash chain, and *creates* one entry, the tax closing entry.
- The invoice and bill documents themselves: see
  [Accounts Receivable](../accounts-receivable/README.md) and
  [Accounts Payable](../accounts-payable/README.md).
- Budget records and budget lines, which are supplied by a separate package and merely surfaced by
  the budget filter of this domain.
