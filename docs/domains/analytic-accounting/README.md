# Analytic Accounting

## Scope

This domain specifies the system's **second, parallel bookkeeping**.

Financial accounting answers the question "what kind of money moved?" — it classifies every
amount by nature: a sale, a purchase of goods, a salary, a receivable, a bank balance. Analytic
accounting answers a different question about exactly the same amounts: "**what was that money
for?**" — which project, which department, which vehicle, which cost centre, which customer
contract. The two books are kept side by side. They share their source events, but they are
structurally independent: an analytic line is not required to balance, it has no debit and
credit columns, it is not part of the double-entry invariant, and an analytic account is not a
financial account.

The mechanism has four moving parts.

1. An **Analytic Plan** (`account.analytic.plan`, table `account_analytic_plan`) is one axis of
   analysis — "Projects", "Departments", "Vehicles". Plans form a tree: a plan may have a parent
   plan, and the topmost ancestor is called the root plan. Each *root* plan materialises, at run
   time, as a real stored column on every record that can carry analysis, so that an amount can
   be tagged simultaneously on several independent axes.
2. An **Analytic Account** (`account.analytic.account`, table `account_analytic_account`) is one
   value on one axis — "Website redesign", "Marketing department", "Van 3". Every analytic
   account belongs to exactly one plan, and therefore to exactly one root plan.
3. An **Analytic Distribution** is a map from a set of analytic accounts to a percentage. It is
   stored as a structured document on the record being analysed — an invoice line, a purchase
   order line, an expense — and it says how the amount of that record is to be spread. A key of
   the map may name more than one account, which is how one slice of the amount can be tagged on
   two axes at once.
4. An **Analytic Line** (`account.analytic.line`, table `account_analytic_line`, shown to users
   as *Analytic Item*) is one posted slice: a dated, signed amount attached to one account on
   each axis, created when the financial document it comes from is posted, and destroyed and
   recreated when the distribution changes.

Around those four sit the supporting machinery: **applicability rules** that make a plan
optional, mandatory or unavailable depending on what kind of document is being entered and for
which company; **distribution models** that pre-fill a distribution automatically from the
partner, the partner category, the product, the product category, the financial account prefix
and the company; and the **validation** that refuses to post a document whose mandatory plans are
not fully distributed.

Concretely the domain covers:

- **Plans** — the tree, the materialised path, the complete name, the colour, the ordering
  sequence, the default applicability, the derived root, the counts of accounts and sub-plans,
  and the two operations that keep the dynamic columns in step when a plan is created, renamed,
  re-parented or deleted.
- **The dynamic column contract** — the exact naming rule for the stored column each root plan
  owns, the naming rule for the derived grouping column each sub-plan level owns, the index
  created alongside, the deletion rule that keeps a grouping column alive while any plan still
  sits at that depth, and the designated base plan whose column has a fixed name.
- **Applicability** — the per-business-domain rules, the scoring algorithm that picks the
  winning rule, the baseline score that a company-only match may never beat, the veto score
  that eliminates a rule outright, the extra criteria the general ledger capability adds
  (financial account prefix and product category), and the resulting optional, mandatory or
  unavailable answer.
- **Analytic accounts** — every field, the display-name rule, the customer link, the company
  scoping, the archival flag, the debit, credit and balance computations with their date
  filtering and their cross-currency conversion, the grouped-total override, and the guard that
  forbids moving an account to another company once it has analytic lines.
- **Distributions** — the storage shape, the percentage normalisation, the search behaviour, the
  grouping behaviour, the merge algorithm that combines a partially-updating distribution with
  the one already present, the cross-plan product rule, and the validation with its exact
  message.
- **Distribution models** — every condition field, the matching algorithm, the prefix condition
  evaluated in memory rather than in the stored query, the ordering, the first-match-per-root-plan
  rule, and the company consistency constraint.
- **Analytic lines** — every field, the signed amount formula derived from a journal item, the
  rounding of the slices and the redistribution of the rounding error, the automatic
  back-computation of the distribution when a line is edited by hand, the creation on posting,
  the deletion on reset to draft, the recreation on redistribution, the splitting of a line by
  writing a distribution on it, the profitability classification, and the constraint tying the
  line to the financial account of its journal item.
- **Cross-domain hooks** — what the general ledger capability adds, what the project accounting
  capability adds (the project profitability sections computed from analytic lines and vendor
  bills), and the contract every other domain must implement to participate.
- **Configuration and security** — the permission group, the access matrix, the record rules,
  the decimal precision record that fixes percentage precision, the system parameter naming the
  base plan, and the shipped default plan.

## What this domain is *not*

- It does not specify journal entries, posting or reconciliation; those belong to
  [../general-ledger/README.md](../general-ledger/README.md). This domain specifies only what
  analytic lines posting produces.
- It does not specify project management, the project entity or the full profitability panel;
  the project entity belongs to [../projects-and-tasks/README.md](../projects-and-tasks/README.md).
  The three profitability sections this domain's project accounting capability contributes are
  specified here, in [calculations.md](calculations.md), because they are computed from analytic
  lines.
- It does not specify budgeting. The edition described here contains no budget capability. The
  accounting settings screen shows a toggle "Use budgets to compare actual with expected revenues
  and costs" whose only effect inside this edition is to switch the analytic accounting setting
  on; no budget record, budget line or budget report exists.
- It does not specify timesheets, expenses, inventory valuation or manufacturing cost capture.
  Those domains create analytic lines through the contract described here; see
  [../timesheets/README.md](../timesheets/README.md),
  [../expenses/README.md](../expenses/README.md),
  [../inventory-valuation-and-costing/README.md](../inventory-valuation-and-costing/README.md)
  and [../manufacturing/README.md](../manufacturing/README.md).

## Entities owned by this domain

| Entity | Transport name | Table | Kind | One-line purpose |
|---|---|---|---|---|
| Analytic Plan | `account.analytic.plan` | `account_analytic_plan` | Persistent, hierarchical | One axis of analysis, arranged in a tree; each root owns a stored column on analysed records. |
| Analytic Plan Applicability | `account.analytic.applicability` | `account_analytic_applicability` | Persistent | A rule making a plan optional, mandatory or unavailable for a business domain, a company, an account prefix or a product category. |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | Persistent, archivable, discussion thread | One value on one axis, with its balance, its customer and its company. |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | Persistent | One dated, signed amount posted against one account per axis. Shown to users as *Analytic Item*. |
| Analytic Distribution Model | `account.analytic.distribution.model` | `account_analytic_distribution_model` | Persistent | A rule that pre-fills a distribution from the partner, partner category, product, product category, account prefix and company. |
| Analytic Mixin | `analytic.mixin` | none of its own | Abstract behaviour | The contract a record implements to carry a distribution; supplies the distribution field, its search, its grouping and its validation. |
| Analytic Plan Fields Mixin | `analytic.plan.fields.mixin` | none of its own | Abstract behaviour | The contract a record implements to carry one account column per root plan. |

The candidate scope of this folder also lists `account.analytic.line.calendar.employee` (the
entity catalogued as *Personal Filters on Employees for the Calendar view*). It is a timesheet
calendar preference, not an analytic record, and it is owned by
[../timesheets/entities.md](../timesheets/entities.md).

## Entities of other domains extended by this domain

| Entity (owning domain) | What this domain adds |
|---|---|
| Journal Item ([../general-ledger/](../general-ledger/README.md)) | The Analytic Mixin, therefore `analytic_distribution` (analytic distribution) and its companions; `analytic_line_ids` (analytic lines); the derived proposal of a distribution from related documents and distribution models; `has_invalid_analytics` (invalid analytics flag); the validation of mandatory plans at posting; the generation, rounding and regeneration of analytic lines; the back-computation from analytic lines; the propagation of distributions to tax lines, early payment discount lines and discount allocation lines. |
| Journal Entry ([../general-ledger/](../general-ledger/README.md)) | Posting refuses archived analytic accounts and creates analytic lines; resetting to draft deletes them. |
| Configuration Settings ([../platform-foundation/](../platform-foundation/README.md)) | The analytic accounting toggle bound to the analytic accounting permission group. |
| System Parameter ([../platform-foundation/](../platform-foundation/README.md)) | The guard on the base-plan parameter `analytic.project_plan`: its value must name an existing root plan, and writing it re-synchronises the dynamic plan columns. |
| Project ([../projects-and-tasks/](../projects-and-tasks/README.md)) | The three profitability sections computed from analytic lines and from vendor bills that mention the project's analytic account, and the embedded navigation to the analytic items of a project. |

Other domains add their own fields to the entities owned here; those additions are listed in
[entities.md](entities.md) with a pointer to the owning domain.

## Reading order

1. **[entities.md](entities.md)** — every field of every entity, with defaults, computations,
   constraints, ordering and display rules, plus the dynamic column contract in full and the
   fields other domains add.
2. **[state-machines.md](state-machines.md)** — the state-bearing fields: the archival flag of an
   analytic account, the existence state of the analytic lines of a journal entry, the hierarchy
   state of a plan, the applicability state of a plan, and the invalid-analytics flag.
3. **[calculations.md](calculations.md)** — the arithmetic: column naming, the applicability
   score, relevant plans, distribution-model matching and combination, the partial-update merge,
   normalisation, the signed amount formula, the closing-line rule, the rounding error
   cancellation, the back-computation, line splitting, balances, the discount allocation
   weighting, the redistribution used by valuation documents, profitability and project
   profitability.
4. **[workflows.md](workflows.md)** — creating a plan, creating accounts, setting a distribution,
   posting, editing after posting, resetting to draft, reversing, moving an account between
   plans, re-parenting a plan, mass editing and reporting.
5. **[business-rules.md](business-rules.md)** — every validation, guard and permission check with
   its exact message, numbered `AA-001` and upwards.
6. **[accounting-effects.md](accounting-effects.md)** — what this domain does and does not post
   to the ledger, and the analytic lines every ledger event produces.
7. **[configuration.md](configuration.md)** — the permission group, the access matrix, the record
   rules, the decimal precision, the system parameter and the shipped data.
8. **[interfaces.md](interfaces.md)** — navigation, views, the distribution editor, named
   operations, notifications, import and export.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given, When and Then scenarios.
10. **[glossary.md](glossary.md)** — every term defined in full.

## Files in this folder

| File | Content |
|---|---|
| [README.md](README.md) | This page: scope, entities, reading order, dependencies. |
| [entities.md](entities.md) | Every entity in full, the distribution structure, the dynamic column contract, the extensions contributed to and by other domains. |
| [state-machines.md](state-machines.md) | Every state-bearing field with its values, transitions, guards, refusal messages and diagram. |
| [workflows.md](workflows.md) | End-to-end procedures with the records each step writes. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with exact messages and the mapping of former rule identifiers. |
| [calculations.md](calculations.md) | Every formula and algorithm with rounding, precision and worked numeric examples. |
| [accounting-effects.md](accounting-effects.md) | The ledger boundary, the analytic records produced per ledger event, sign conventions, reversal behaviour. |
| [configuration.md](configuration.md) | Settings, parameters, precisions, shipped data, groups, access rights, record rules, prerequisites. |
| [interfaces.md](interfaces.md) | Screens, named operations, the distribution editor, notifications, exports. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When, Then scenarios with concrete numbers. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |

## Dependencies on other domains

| Depends on | For what |
|---|---|
| [../platform-foundation/README.md](../platform-foundation/README.md) | Dynamic field definitions and stored view definitions (the plan columns are created at run time), system parameters, decimal precisions, per-company stored values, translations, transaction caches. |
| [../identity-and-access/README.md](../identity-and-access/README.md) | Companies, parent companies, users, permission groups and record rules. |
| [../contacts-and-organizations/README.md](../contacts-and-organizations/README.md) | Partners and partner categories used as distribution-model conditions and as the customer of an account, and the commercial partner used in the display name. |
| [../multi-currency/README.md](../multi-currency/README.md) | The company currency an analytic line is valued in, and the conversion used when balances span companies with different currencies. |
| [../units-of-measure-and-packaging/README.md](../units-of-measure-and-packaging/README.md) | The unit recorded alongside the quantity on an analytic line. |
| [../general-ledger/README.md](../general-ledger/README.md) | Journal items, their balance, their account, their journal and their posting state — the source of almost every analytic line; account codes for prefix matching and account types for profitability; fiscal year dates. |
| [../products-and-catalog/README.md](../products-and-catalog/README.md) | Products and product categories used as distribution-model and applicability conditions, and the product cost used by the manual valuation helper. |
| [../messaging-and-activities/README.md](../messaging-and-activities/README.md) | The discussion thread carried by an analytic account. |

## Domains that depend on this one

| Domain | For what |
|---|---|
| [../general-ledger/README.md](../general-ledger/README.md) | The distribution field on journal items, validation at posting, analytic line creation and deletion. |
| [../accounts-receivable/README.md](../accounts-receivable/README.md) | Distribution defaulting on invoice lines and the `invoice` business domain. |
| [../accounts-payable/README.md](../accounts-payable/README.md) | Distribution defaulting on bill lines and the `bill` business domain. |
| [../purchasing/README.md](../purchasing/README.md) | Distribution on order lines carried to the bill, and the `purchase_order` business domain. |
| [../sales/README.md](../sales/README.md) | Distribution on order lines carried to the invoice, and the `sale_order` business domain. |
| [../projects-and-tasks/README.md](../projects-and-tasks/README.md) | A project owns an analytic account; profitability reads analytic lines. |
| [../timesheets/README.md](../timesheets/README.md) | Timesheet lines are analytic lines with a cost amount, and the `timesheet` business domain. |
| [../expenses/README.md](../expenses/README.md) | Expense lines carry a distribution posted with the expense entry, and the `expense` business domain. |
| [../manufacturing/README.md](../manufacturing/README.md) | Work-centre and component costs posted as analytic lines, and the `manufacturing_order` business domain. |
| [../inventory-operations/README.md](../inventory-operations/README.md) | Transfer validation checks mandatory plans through the `stock_picking` business domain. |
| [../inventory-valuation-and-costing/README.md](../inventory-valuation-and-costing/README.md) | The in-place redistribution of the analytic lines of a valuation document. |
| [../payments-and-bank-reconciliation/README.md](../payments-and-bank-reconciliation/README.md) | Reconciliation model lines carry a distribution; exchange difference entries post without the mandatory plan check. |
| [../financial-reporting/README.md](../financial-reporting/README.md) | Analytic filtering and grouping of financial reports. |
| [../customer-portal/README.md](../customer-portal/README.md) | Portal visibility of timesheet analytic lines. |

## The seven rules that matter

A rebuild that gets nothing else right must get these right.

1. **A distribution is a map from a key to a percentage, and the key may be compound.** The key
   is a list of analytic account identifiers. A key naming two accounts means that slice is
   tagged on both axes at once and produces **one** analytic line carrying both accounts.
2. **Percentages are validated per root plan, not globally.** A distribution is complete when,
   for each *mandatory* root plan, the percentages of the keys mentioning an account of that
   root plan add up to exactly one hundred, compared at the percentage precision. Two mandatory
   plans therefore require two hundred percent in total across the map.
3. **The analytic amount is the negated balance of the journal item, scaled by the
   percentage.** A debit of one thousand produces an analytic amount of minus one thousand at
   one hundred percent. Costs are negative and revenues are positive in the analytic book.
4. **The slice that completes a plan absorbs the remainder.** When the running total of a plan's
   percentages reaches exactly one hundred, the slice that reaches it is computed from the
   remaining percentage rather than from its own, so that the sum of the slices equals the
   journal item amount exactly.
5. **After the per-slice rounding there is a second, explicit error-cancelling pass.** Rounded
   slices are compared with their unrounded originals; the accumulated error is pushed back
   onto the slices, starting with the first, one rounding unit at a time until it is zero.
6. **A distribution need not total one hundred percent.** Only a mandatory plan forces that.
   A total below one hundred analyses part of the amount; a total above one hundred analyses the
   amount more than once on that axis, which is a legitimate configuration.
7. **The two books are kept in step in both directions.** Posting derives analytic lines from the
   distribution; editing an analytic line by hand derives the distribution back from the lines.
   Every system-driven write of analytic lines runs with the synchronisation guard raised so the
   two directions never loop.

Each of these is stated as arithmetic, with worked numbers, in
[calculations.md](calculations.md).
