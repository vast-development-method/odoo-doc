# Analytic Accounting

## Scope

This domain specifies the system's **second, parallel bookkeeping**.

Financial accounting answers the question "what kind of money moved?" — it classifies every
amount by nature: a sale, a purchase of goods, a salary, a receivable, a bank balance. Analytic
accounting answers a different question about exactly the same amounts: "**what was that money
for?**" — which project, which department, which vehicle, which cost centre, which customer
contract. The two books are kept side by side. They share their source events, but they are
structurally independent: an analytic item is not required to balance, it has no debit and
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
4. An **Analytic Item** (`account.analytic.line`, table `account_analytic_line`) is one posted
   slice: a dated, signed amount attached to one account on each axis, created when the
   financial document it comes from is posted, and destroyed and recreated when the distribution
   changes.

Around those four sit the supporting machinery: **applicability rules** that make a plan
optional, mandatory or unavailable depending on what kind of document is being entered and for
which company; **distribution models** that pre-fill a distribution automatically from the
partner, the partner category, the product, the product category, the financial account prefix
and the company, resolved by a specificity score; and the **validation** that refuses to post a
document whose mandatory plans are not fully distributed.

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
  that eliminates a rule outright, the extra criteria the accounting package adds (financial
  account prefix and product category), and the resulting optional / mandatory / unavailable
  answer.
- **Analytic accounts** — every field, the display-name rule, the customer link, the company
  scoping, the archival flag, the debit, credit and balance computations with their date
  filtering and their cross-currency conversion, the grouped-total override, and the guard that
  forbids moving an account to another company once it has items.
- **Distributions** — the storage shape, the percentage normalisation, the search behaviour, the
  grouping behaviour, the merge algorithm that combines a partially-updating distribution with
  the one already present, the cross-plan product rule, and the validation with its exact
  message.
- **Distribution models** — every condition field, the matching algorithm, the prefix condition
  evaluated in memory rather than in the query, the ordering, the first-match-per-root-plan
  rule, and the company consistency constraint.
- **Analytic items** — every field, the signed amount formula derived from a journal item, the
  rounding of the slices and the redistribution of the rounding error, the automatic
  back-computation of the distribution when an item is edited by hand, the creation on posting,
  the deletion on reset to draft, the recreation on redistribution, the profitability
  classification, and the constraint tying the item to the financial account of its journal
  item.
- **Cross-domain hooks** — what the accounting package adds, what the project package adds, and
  the contract every other domain must implement to participate.
- **Configuration and security** — the permission group, the access matrix, the record rules,
  the decimal precision record that fixes percentage precision, the system parameter naming the
  base plan, and the shipped default plan.

## What this domain is *not*

- It does not specify journal entries, posting or reconciliation; those belong to
  [../general-ledger/README.md](../general-ledger/README.md). This domain specifies only what
  analytic items posting produces.
- It does not specify project profitability reporting in full; the profitability sections that
  read analytic items are described here only as a data contract, and the project entity itself
  belongs to [../projects-and-tasks/README.md](../projects-and-tasks/README.md).
- It does not specify timesheets, expenses or manufacturing cost capture. Those domains create
  analytic items through the contract described here.

## Entities

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Analytic Plan | `account.analytic.plan` | `account_analytic_plan` | One axis of analysis, arranged in a tree; each root owns a stored column on analysed records. |
| Analytic Plan Applicability | `account.analytic.applicability` | `account_analytic_applicability` | A rule making a plan optional, mandatory or unavailable for a business domain, a company, an account prefix or a product category. |
| Analytic Account | `account.analytic.account` | `account_analytic_account` | One value on one axis, with its balance, its customer and its company. |
| Analytic Item | `account.analytic.line` | `account_analytic_line` | One dated, signed amount posted against one account per axis. |
| Analytic Distribution Model | `account.analytic.distribution.model` | `account_analytic_distribution_model` | A rule that pre-fills a distribution from the partner, category, product, product category, account prefix and company. |
| Analytic Mixin | `analytic.mixin` | (no table of its own) | The contract a record implements to carry a distribution; supplies the distribution field, its search, its grouping and its validation. |
| Analytic Plan Fields Mixin | `analytic.plan.fields.mixin` | (no table of its own) | The contract a record implements to carry one account column per root plan. |

## Reading order

1. **[entities.md](entities.md)** — every field of every entity, with defaults, computations,
   constraints, ordering and display rules, plus the dynamic column contract in full.
2. **[calculations.md](calculations.md)** — the arithmetic: the signed amount formula, the
   percentage rounding and error redistribution, the applicability score, the distribution-model
   specificity, the merge algorithm, the balance computation, and worked numeric examples
   including the sixty–forty split, the cross-plan split and the thirty-three / thirty-three /
   thirty-four rounding case.
3. **[state-machines.md](state-machines.md)** — the states that govern analytic items: the
   posting state of the parent entry, the archival state of accounts and plans, and the
   distribution validity flag.
4. **[workflows.md](workflows.md)** — creating a plan, creating accounts, setting a
   distribution, posting, editing after posting, resetting to draft, reversing, moving an
   account between plans, and re-parenting a plan.
5. **[business-rules.md](business-rules.md)** — every validation with its exact message.
6. **[accounting-effects.md](accounting-effects.md)** — what this domain does and does not post
   to the ledger, and the analytic items every ledger event produces.
7. **[configuration.md](configuration.md)** — the permission group, the access matrix, the
   record rules, the decimal precision, the system parameter and the shipped data.
8. **[interfaces.md](interfaces.md)** — navigation, views, the distribution editor data
   contract, named remote operations, and import and export.
9. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given/When/Then scenarios.
10. **[glossary.md](glossary.md)** — every term defined in full.

## Dependencies on other domains

| Depends on | For what |
|---|---|
| [../identity-and-access/README.md](../identity-and-access/README.md) | Companies, root companies, users, permission groups and record rules. |
| [../contacts-and-organizations/README.md](../contacts-and-organizations/README.md) | Partners and partner categories used as distribution-model conditions and as the customer of an account. |
| [../multi-currency/README.md](../multi-currency/README.md) | The company currency an analytic item is valued in, and the conversion used when balances span companies with different currencies. |
| [../units-of-measure-and-packaging/README.md](../units-of-measure-and-packaging/README.md) | The unit recorded alongside the quantity on an analytic item. |
| [../general-ledger/README.md](../general-ledger/README.md) | Journal items, their balance, their account and their posting state — the source of almost every analytic item. |
| [../products-and-catalog/README.md](../products-and-catalog/README.md) | Products and product categories used as distribution-model and applicability conditions. |

## Domains that depend on this one

| Domain | For what |
|---|---|
| [../general-ledger/README.md](../general-ledger/README.md) | The distribution field on journal items, validation at posting, item creation and deletion. |
| [../accounts-receivable/README.md](../accounts-receivable/README.md) | Distribution defaulting on invoice lines and the `invoice` business domain. |
| [../accounts-payable/README.md](../accounts-payable/README.md) | Distribution defaulting on bill lines and the `bill` business domain. |
| [../purchasing/README.md](../purchasing/README.md) | Distribution on order lines carried to the bill. |
| [../sales/README.md](../sales/README.md) | Distribution on order lines carried to the invoice. |
| [../projects-and-tasks/README.md](../projects-and-tasks/README.md) | A project owns an analytic account; profitability reads analytic items. |
| [../timesheets/README.md](../timesheets/README.md) | Timesheet lines are analytic items with a cost amount. |
| [../expenses/README.md](../expenses/README.md) | Expense lines carry a distribution posted with the expense entry. |
| [../manufacturing/README.md](../manufacturing/README.md) | Work-centre and component costs posted as analytic items. |
| [../financial-reporting/README.md](../financial-reporting/README.md) | Analytic filtering and grouping of financial reports. |

## The five rules that matter

A rebuild that gets nothing else right must get these right.

1. **A distribution is a map from a key to a percentage, and the key may be compound.** The key
   is a list of analytic account identifiers. A key naming two accounts means that slice is
   tagged on both axes at once and produces **one** analytic item carrying both accounts.
2. **Percentages are validated per root plan, not globally.** A distribution is complete when,
   for each *mandatory* root plan, the percentages of the keys mentioning an account of that
   root plan add up to exactly one hundred, compared at the percentage precision. Two mandatory
   plans therefore require two hundred percent in total across the map.
3. **The analytic amount is the negated balance of the journal item, scaled by the
   percentage.** A debit of one thousand produces an analytic amount of minus one thousand at
   one hundred percent. Costs are negative and revenues are positive in the analytic book.
4. **The last slice of a plan absorbs the remainder.** When the running total of a plan's
   percentages reaches exactly one hundred, the slice that reaches it is computed from the
   remaining percentage rather than from its own, so that the sum of the slices equals the
   journal item amount exactly.
5. **After the per-slice rounding there is a second, explicit error-cancelling pass.** Rounded
   slices are compared with their unrounded originals; the accumulated error is pushed back
   onto the slices one rounding unit at a time until it is zero.

Each of these is stated as arithmetic, with worked numbers, in
[calculations.md](calculations.md).
