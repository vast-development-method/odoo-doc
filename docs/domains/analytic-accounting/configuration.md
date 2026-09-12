# Analytic Accounting — Configuration

Every setting, system parameter, precision, shipped record, permission group, access right, record
rule and platform facility the domain needs, with its data type, its shipped value and its effect.
A replacement must ship the same records and honour the same defaults, because other domains and
the analytic behaviour itself depend on their existence.

Contents:

1. [Settings on the accounting settings screen](#1-settings-on-the-accounting-settings-screen)
2. [System parameters](#2-system-parameters)
3. [Decimal precisions](#3-decimal-precisions)
4. [Shipped reference data](#4-shipped-reference-data)
5. [Demonstration data](#5-demonstration-data)
6. [Permission groups](#6-permission-groups)
7. [The access rights matrix](#7-the-access-rights-matrix)
8. [Field-level visibility](#8-field-level-visibility)
9. [Record rules](#9-record-rules)
10. [Company consistency checks](#10-company-consistency-checks)
11. [Sequences, scheduled jobs, message templates and activity types](#11-sequences-scheduled-jobs-message-templates-and-activity-types)
12. [Platform facilities the domain requires](#12-platform-facilities-the-domain-requires)
13. [The closed list of business domains and who contributes each](#13-the-closed-list-of-business-domains-and-who-contributes-each)
14. [Entities that implement the two contracts](#14-entities-that-implement-the-two-contracts)
15. [The order of the configuration steps](#15-the-order-of-the-configuration-steps)
16. [What a rebuild must configure before the domain is usable](#16-what-a-rebuild-must-configure-before-the-domain-is-usable)
17. [Reconciliation notes](#17-reconciliation-notes)

---

## 1. Settings on the accounting settings screen

Both settings live in the block titled "Analytics" of the accounting settings screen and are shown
only to a reader who holds the full accounting features group.

### 1.1 The analytic accounting switch

| Aspect | Specification |
|---|---|
| Identifier | `group_analytic_accounting` (analytic accounting) |
| Label of the field | "Analytic Accounting" |
| Text shown beside the switch | "Track costs & revenues by project, department, etc" — reproduced, because it is the label a reader looks for |
| Hover text | "Allows you to use the analytic accounting." — reproduced |
| Type | Boolean bound to a permission group |
| Default | off |
| Scope | The whole database, not one company: the switch grants a group, and a group is global |
| Effect when switched on | The analytic accounting permission group of section 6 is granted to **every** user of the internal user group. In the same form, and before saving, the full accounting capability setting is switched on as well. |
| Effect when switched off | The group is revoked from the internal user group. No record of this domain is deleted: the plans, the accounts, the distributions and the analytic lines all survive and reappear when the switch is turned on again. |
| What becomes visible | The analytic plans, analytic accounts, analytic items and analytic distribution models screens; the analytic distribution cell on every record that carries one; the analytic account columns of the analytic item list; the analytic sections of a project's profitability panel. |

### 1.2 The budget switch

| Aspect | Specification |
|---|---|
| Identifier | `module_account_budget` (budget management) |
| Label of the field | "Budget Management" |
| Text shown beside the switch | "Use budgets to compare actual with expected revenues and costs" — reproduced |
| Hover text | "This allows accountants to manage analytic and crossovered budgets. Once the master budgets and the budgets are defined, the project managers can set the planned amount on each analytic account." — reproduced |
| Type | Boolean that requests the installation of a capability package |
| Default | off |
| Effect inside this edition | Switching it on switches `group_analytic_accounting` on as well, as a form-level consequence applied before saving. The budgeting capability itself is not part of this edition: no budget record, no budget line and no budget report exists, and nothing else happens. |

A rebuild that ships no budgeting capability keeps the switch, keeps its effect on the analytic
switch, and does nothing else with it.

---

## 2. System parameters

| Key | Type | Shipped value | Effect and constraints |
|---|---|---|---|
| `analytic.project_plan` (the base-plan parameter) | Text holding a decimal whole number | the text `1` | Designates the **base plan**: the one root plan whose analytic account column on every plan-bearing entity has the fixed name `account_id` (project account) instead of a name derived from the plan's identifier. |

**How the shipped value is correct.** The parameter is written with the literal text `1` when the
capability is installed, immediately before the shipped plan named *Project* is created. That plan
is the first record of its table in a fresh database and therefore receives the internal identifier
one, so the parameter designates it. A rebuild that creates its plans in another order must write
the identifier it actually obtained.

**Constraints, from [business-rules.md](business-rules.md).**

1. The parameter must always name an existing root plan. Every operation that needs the list of
   root plans fails otherwise, with "A 'Project' plan needs to exist and its id needs to be set as
   `analytic.project_plan` in the system variables" (`AA-016`).
2. Writing it is guarded: the new value must be a string of digits naming an existing plan that
   currently owns a stored column, which means an existing root plan. Anything else is refused with
   "The value for the key must be the ID to a valid analytic plan that is not a subplan", the
   placeholder being the parameter key (`AA-017`).
3. A successful write migrates **no data**: the former base plan receives a new, empty generated
   column and the generated column of the new base plan is deleted with its contents, while the
   fixed column keeps the values it held, which are from then on read as accounts of the new base
   plan (`AA-018`). The operation therefore belongs to configuration time, before any analytic line
   exists. The procedure is workflow 8 of [workflows.md](workflows.md).
4. The plan the parameter designates may never be given a parent (`AA-003`).

The domain defines no other system parameter and reads no other one.

---

## 3. Decimal precisions

| Name | Digits shipped | Effect |
|---|---|---|
| `Percentage Analytic` (the percentage precision) | 2 | The number of decimal places to which every distribution percentage is rounded on write (`AA-041`), and at which every comparison of a plan total against one hundred percent is made (`AA-081`, `AA-103`). It is also the editing precision of the percentage column of the distribution editor, which computes intermediate values with **two additional** decimal places and rounds back to this precision (`AA-107`). |

The record is shipped with the never-overwritten marker, so a later update of the capability does
not reset a value an administrator has changed. Changing it to more digits makes finer percentages
storable and makes the completeness comparison stricter; changing it to fewer makes it coarser.
Nothing else in the domain reads it.

Monetary rounding is not configured here: it is the rounding step of the **company currency** of the
journal item, owned by [../multi-currency/](../multi-currency/README.md).

---

## 4. Shipped reference data

| Record | Kind | Values | Purpose and update behaviour |
|---|---|---|---|
| Plan *Project* | Analytic Plan | name "Project"; default applicability `optional`; no parent; sequence 10; a random colour index | The base plan. Its identifier is what `analytic.project_plan` holds. It owns the fixed column `account_id` on every plan-bearing entity. It is created at installation only and is never recreated by a later update if it has been deleted. |
| `Percentage Analytic` | Decimal precision | 2 digits | Section 3. Never overwritten by a later update. |
| Analytic Accounting | Permission group | name "Analytic Accounting" | Section 6. |
| Five access-right records | Access right | one per entity of the domain, all four permissions granted to the analytic accounting group | Section 7. |
| Four record rules | Record rule | global, one per entity that carries a company | Section 9. |
| The default value of the default applicability | System-wide default | `optional` for the field `default_applicability` (default applicability) of Analytic Plan | Registered at installation; used by every company that has set no explicit value for a given plan. |

No sequence, no journal, no account, no message template and no activity type is shipped by this
domain.

---

## 5. Demonstration data

Installed only when the database is created with demonstration data. Nothing in the domain depends
on it, and a replacement may ship an equivalent set or none at all.

| Record | Kind | Values |
|---|---|---|
| Plan *Departments* | Analytic Plan | root plan, default applicability `optional` |
| Plan *Internal* | Analytic Plan | root plan, default applicability `unavailable` |
| *Time Off*, *Operating Costs* | Analytic Account | plan *Internal*, no company, no customer |
| *Our Super Product*, *Seagate P2*, *Millennium Industries*, *CampToCamp*, *Acme Corporation*, *Asustek*, *Delta PC*, *Spark Systems*, *Nebula*, *Luminous Technologies*, *Desertic - Hispafuentes*, *Lumber Inc*, *Camp to Camp* | Analytic Account | plan *Project*, no company, each with a customer taken from the demonstration contacts |
| *Active account* | Analytic Account | plan *Project*, no company, no customer, active |
| *Administrative*, *Commercial & Marketing*, *Research & Development*, *Human Resources*, *Legal*, *Finance*, *Production* | Analytic Account | plan *Departments*, no company, no customer |

The demonstration set is what makes the *Internal* plan a useful example: its default applicability
is `unavailable`, so it is never offered in a distribution editor until a rule or the default is
changed, while its two accounts still exist and still carry their history.

---

## 6. Permission groups

The domain defines exactly **one** group.

| Group | Name | Implied groups | Granted by |
|---|---|---|---|
| The analytic accounting permission group | "Analytic Accounting" | none | The accounting setting of section 1.1, which grants it to every user of the internal user group; it may also be granted directly on a user. |

No other group of this domain exists, and no other group grants any right on the entities of this
domain (`AA-109`). A reader outside the group sees no analytic screen, no analytic column and no
distribution editor (`AA-112`). Two neighbouring groups appear in the rules of this folder without
being defined by it: the read-only accounting group and the invoicing group, both owned by
[../general-ledger/configuration.md](../general-ledger/configuration.md), which govern the
visibility of the derived totals (section 8) and of the actions behind the project profitability
sections.

---

## 7. The access rights matrix

| Entity | Transport name | Group | Read | Create | Modify | Delete |
|---|---|---|---|---|---|---|
| Analytic Plan | `account.analytic.plan` | Analytic Accounting | yes | yes | yes | yes |
| Analytic Plan Applicability | `account.analytic.applicability` | Analytic Accounting | yes | yes | yes | yes |
| Analytic Account | `account.analytic.account` | Analytic Accounting | yes | yes | yes | yes |
| Analytic Line | `account.analytic.line` | Analytic Accounting | yes | yes | yes | yes |
| Analytic Distribution Model | `account.analytic.distribution.model` | Analytic Accounting | yes | yes | yes | yes |

Consequences a rebuild must reproduce:

1. A reader who holds only this group may create plans, accounts, applicability rules, distribution
   models and analytic lines, and the creating user is recorded as the author of the record
   (`AA-113`).
2. There is no read-only variant. A reader either has the whole matrix or has nothing.
3. The two abstract contracts — Analytic Mixin and Analytic Plan Fields Mixin — have no table and
   therefore no access right of their own; the rights of the host entity apply.
4. Four reads are performed with **elevated rights** and must stay elevated in a rebuild, otherwise
   the behaviour changes (`AA-115`): the resolution of the list of root plans; the company
   consistency count of `AA-031`, which must see analytic lines the reader may not; the name search
   on the customer of an analytic account; and the archived-account check of `AA-078`, which must
   see archived accounts.
5. The field descriptions and the view patching that insert one column per root plan are applied
   only when the reader may read Analytic Plan, and are skipped in the view-customisation mode
   (`AA-114`).

Neighbouring domains add their own restrictions on Analytic Line without changing this matrix: the
timesheets domain restricts a reader to the timesheets of their own employee record unless they
hold a timesheet approval group, and the customer portal restricts an external reader to the
analytic lines of the documents shared with them. Those rules belong to
[../timesheets/configuration.md](../timesheets/configuration.md) and
[../customer-portal/configuration.md](../customer-portal/configuration.md).

---

## 8. Field-level visibility

| Field | Entity | Shown to |
|---|---|---|
| `debit` (debit) and `credit` (credit) | Analytic Account | Hidden columns in the base account list. When the accounting capability is installed they become ordinary columns restricted to a reader holding the read-only accounting group or the invoicing group. |
| `balance` (balance) | Analytic Account | A visible column with a column total in the base account list; when the accounting capability is installed the same restriction as the debit and the credit applies to the **column**. The *Gross Margin* button of the account's form shows the balance to every member of the analytic accounting group, with no further restriction. |
| `company_id` (company) | every entity of the domain | Only in a multi-company database, with the placeholder "Visible to all" when empty. |
| `currency_id` (currency) | Analytic Account | Only to a reader for whom the multi-currency capability is enabled. |
| `default_applicability` (default applicability) and the applicability page | Analytic Plan | Hidden as soon as the plan has a parent, because applicability belongs to root plans only. |
| `account_prefix` (financial accounts prefixes) | Analytic Plan Applicability | Only when the derived flag `display_account_prefix` (show the prefix field) is true, which the general ledger capability computes for `general`, `invoice` and `bill` and the expenses capability forces true for `expense`. |
| `product_categ_id` (product category), `account_prefix` (accounts prefix), `product_id` (product) | Analytic Plan Applicability and Analytic Distribution Model | Contributed by the general ledger capability; absent when that capability is not installed. |

---

## 9. Record rules

Four **global** rules are shipped. Global means they apply to every reader, administrators
included, and cannot be switched off by granting a group.

| Entity | A record is visible when | Rule identifier in [business-rules.md](business-rules.md) |
|---|---|---|
| Analytic Account | it has no company, **or** its company is an ancestor of one of the active companies | `AA-096` |
| Analytic Plan Applicability | the same condition | `AA-097` |
| Analytic Distribution Model | the same condition | `AA-097` |
| Analytic Line | its company is **one of** the active companies | `AA-072` |

**The asymmetry is intentional and must be reproduced.** Master data — accounts, applicability
rules, distribution models — defined for a parent company is usable by its branches, and a record
with no company is usable everywhere. Facts — analytic lines — belong strictly to the company that
produced them: a line with no company is visible to nobody, and a line of a subsidiary is not
visible from the parent company unless that subsidiary is among the active companies.

Analytic Plan carries **no company at all** and no record rule restricts it: a plan is global, and
only its default applicability, which is stored per company, and its applicability rules, which may
name a company, behave differently from one company to the next (`AA-021`).

---

## 10. Company consistency checks

Beyond the record rules, the platform's company-consistency machinery is declared on the following
links; a violation raises the platform message quoted in `AA-036`.

| Entity | Link | Must satisfy |
|---|---|---|
| Analytic Account | `partner_id` (customer) | The customer has no company, or the account's company, or an ancestor of it. |
| Analytic Line | each plan column, `partner_id`, `product_id`, `general_account_id`, `journal_id` | Each has no company, or the line's company, or an ancestor of it. |
| Analytic Distribution Model | `partner_id`, `product_id` | The same, against the model's company. |
| Analytic Distribution Model | the accounts named in its distribution | A model whose distribution names an account belonging to one specific company must itself belong to that company (`AA-059`). |
| Analytic Account | the analytic lines that carry it | Its company may not be changed while a line outside the new company's sub-tree carries it (`AA-031`). |

---

## 11. Sequences, scheduled jobs, message templates and activity types

| Kind of configuration record | This domain ships |
|---|---|
| Sequences | **None.** No record of this domain is numbered: a plan, an account, a rule, a model and an analytic line are all identified by their internal identifier, and an analytic account's optional reference is typed by a human. |
| Scheduled jobs | **None.** Analytic lines appear and disappear only as a consequence of a posting, a reset to draft, a distribution change, a valuation or a direct edit. The automatic posting job of the general ledger creates analytic lines indirectly when it posts an entry, and it posts **without** the validation flag, so a mandatory plan never blocks it (`AA-079`). |
| Message templates | **None.** The domain sends no electronic mail. |
| Activity types | **None.** |
| Notifications | One, described in [interfaces.md](interfaces.md): the count of lines created by a split. |
| Discussion threads | One: Analytic Account carries a thread that tracks `name`, `code`, `active` and `partner_id`. The thread machinery belongs to [../messaging-and-activities/](../messaging-and-activities/README.md). |
| Reports and printable documents | **None.** |

---

## 12. Platform facilities the domain requires

A rebuild cannot implement this domain on a platform that lacks the following. Each item is a hard
prerequisite, not an optimisation, except where the row says otherwise.

1. **Run-time field definition.** The domain creates and deletes **stored columns** on existing
   entities while the application is running, in response to the creation, renaming, re-parenting
   and deletion of a plan. The platform must be able to add a stored link column with a partial
   index and a restricting deletion rule, to add a derived read-only relation field with a
   traversal path, and to remove both again. This is the single most demanding prerequisite of the
   domain; the contract is specified in [entities.md](entities.md) section 3.
2. **Stored view definitions that can refuse a deletion.** Deleting a plan whose column is still
   named by a stored view definition must be refused with "Cannot rename/delete fields that are
   still present in views:" followed by the field list and the view name, and nothing at all may be
   deleted (`AA-009`).
3. **View patching at read time.** A view containing the base plan's column must gain one column
   per other root plan, and a grouping filter on that column must gain one filter per root plan and
   per sub-plan depth, without the stored definition being modified ([entities.md](entities.md)
   section 9.4).
4. **A per-company stored value** for the default applicability of a plan: one value per company for
   the same record, with a system-wide default (`AA-019`).
5. **Translatable text** for the name of a plan and the name of an analytic account.
6. **A discussion thread** on Analytic Account, tracking four fields.
7. **A transaction-scoped cache** for the answer to *which plans are relevant*, keyed by the exact
   set of situation arguments and dropped by the operations of `AA-020`.
8. **A generalised inverted index** over the account identifiers extracted from the keys of a stored
   distribution, on every entity that stores one (`AA-052`). Without it, "which documents mention
   this analytic account" scans every row; the behaviour is unchanged, the cost is not.
9. **A client reload mechanism.** Every successful create, write or delete of a plan changes the set
   of fields of the plan-bearing entities, so every open client must reload its definition of those
   entities (`AA-021`).
10. **A structured document field** able to hold the distribution, to be searched with the four
    operators of `AA-053` and to be grouped with a count (`AA-058`).
11. **Elevated-rights reads** as listed in section 7, item 4.
12. **A currency conversion service** able to convert an amount from one currency into another at a
    given date for a given company, used by the derived totals of an analytic account and by the
    project profitability sections.

---

## 13. The closed list of business domains and who contributes each

The stored values of the field `business_domain` (business domain) of an applicability rule grow
with the installed capabilities. A rebuild that ships a subset of the capabilities ships a subset
of the values, and removing the capability that contributed a value deletes the rules carrying it.

| Value | Label | Contributed by | Supplied when |
|---|---|---|---|
| `general` | Miscellaneous | this domain | A journal item that is on neither a sale nor a purchase document is validated or edited. |
| `invoice` | Invoice | the general ledger capability | A journal item of a customer invoice, a credit note or a customer receipt is validated or edited. |
| `bill` | Vendor Bill | the general ledger capability | A journal item of a vendor bill, a refund or a purchase receipt is validated or edited. |
| `expense` | Expense | the expenses capability | An expense is approved. |
| `timesheet` | Timesheet | the timesheets capability | A timesheet line is created or modified. |
| `purchase_order` | Purchase Order | the purchasing capability | A purchase order is confirmed. |
| `sale_order` | Sale Order | the sales capability | A sales order is confirmed, invoiced or turned into a proforma document. |
| `manufacturing_order` | Manufacturing Order | the manufacturing accounting capability | A manufacturing order is confirmed. |
| `stock_picking` | Stock Picking | the project and inventory accounting capability | A warehouse transfer is validated. |

The account prefix criterion of a rule is offered only for `general`, `invoice`, `bill` and
`expense`; the product category criterion is offered for every value once the general ledger
capability is installed.

---

## 14. Entities that implement the two contracts

### 14.1 Entities that carry a distribution

Each of the following implements the Analytic Mixin and therefore gains
`analytic_distribution` (analytic distribution), `analytic_precision` (percentage precision) and
`distribution_analytic_account_ids` (analytic accounts of the distribution), with the index of
`AA-052` where it has a real table.

| Entity | Owning domain | What the distribution attributes |
|---|---|---|
| Journal Item | [../general-ledger/](../general-ledger/README.md) | the posted amount, which is what generates analytic lines |
| Analytic Distribution Model | this domain | the template itself |
| Sales Order Line | [../sales/](../sales/README.md) | the future invoice line, onto which it is copied |
| Purchase Order Line | [../purchasing/](../purchasing/README.md) | the future bill line, onto which it is copied |
| Expense | [../expenses/](../expenses/README.md) | the future expense journal item |
| Reconciliation Model Line | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/README.md) | the journal item the rule writes |
| Asset | [../general-ledger/](../general-ledger/README.md) | the depreciation entries |
| Work Centre | [../manufacturing/](../manufacturing/README.md) | the work order cost |
| Withholding tax line | [../taxes/](../taxes/README.md) | the withheld amount |

### 14.2 Entities that carry one account per axis

Each of the following implements the Analytic Plan Fields Mixin and therefore gains one stored
analytic account column per root plan, one derived grouping column per existing sub-plan depth, and
the magic column that stands for all of them.

| Entity | Owning domain | What the columns attribute |
|---|---|---|
| Analytic Line | this domain | the fact itself |
| The inventory valuation line holders | [../inventory-valuation-and-costing/](../inventory-valuation-and-costing/README.md) | the valuation of a stock move |
| The manufacturing cost line holders | [../manufacturing/](../manufacturing/README.md) | the cost of a work order or of a manufacturing order |

Every entity in this second table gains and loses its columns automatically, at run time, whenever a
plan is created, re-parented or deleted.

---

## 15. The order of the configuration steps

The steps below are the order in which an administrator brings the domain into service. Each step
states what breaks when it is skipped.

1. **Install the capability.** The permission group, the precision, the base-plan parameter and the
   shipped plan appear. Skipping this leaves no analytic entity at all.
2. **Switch the analytic accounting setting on.** Without it the group is granted to nobody and no
   analytic screen is reachable, although the records exist.
3. **Decide the base plan, once.** Either keep the shipped plan or designate another root plan
   before any analytic line exists, because the change migrates nothing (`AA-018`). Skipping this
   decision and changing the parameter later silently re-reads the values of the fixed column as
   accounts of the new base plan.
4. **Create the root plans.** Each one creates a stored column on every plan-bearing entity and
   makes every open client reload. Doing this on a live database is supported and is the intended
   way to add an axis.
5. **Create the sub-plans**, if the axis needs levels. Each new depth creates one grouping column
   per root plan on every plan-bearing entity.
6. **Create the analytic accounts.** A plan whose whole sub-tree has no account is never offered in
   a distribution editor (`AA-028`), so this is the step that makes an axis appear to users.
7. **Set the default applicability of each root plan**, per company. This is the value that applies
   when no rule wins.
8. **Add the applicability rules** that narrow the default by business domain, company, account
   prefix and product category. A rule that names only a company never wins (`AA-025`).
9. **Add the distribution models** that pre-fill distributions, and order them by sequence, because
   the sequence is the whole of the priority rule (`AA-062`).
10. **Check the mandatory plans against the documents that will be posted.** A plan made mandatory
    for a business domain blocks every interactive posting of that kind until every product line
    totals one hundred percent on it.

---

## 16. What a rebuild must configure before the domain is usable

1. A root plan designated as the base plan, with its identifier stored in `analytic.project_plan`.
   Nothing in the domain works without it: the list of root plans, the column naming, the
   distribution editor and the validation all fail with the message of `AA-016`.
2. The `Percentage Analytic` precision record, because both the normalisation on write and the
   completeness comparison read it.
3. The analytic accounting permission group and the five access-right records of section 7.
4. The four record rules of section 9, as **global** rules.
5. The system-wide default `optional` for the default applicability of a plan, so that a plan
   created without an explicit value behaves as optional rather than as unavailable.
6. The run-time field facility of section 12, item 1, which must already be able to add a column to
   the analytic line table at the moment the first root plan is created.

---

## 17. Reconciliation notes

1. **A single source, corrected in four places.** Only one of the two drafts of this folder carried
   a configuration document. It is kept in full, with the corrections below.
2. **The label of the analytic setting.** The former draft quoted the setting as "Track costs and
   revenues by project, department, and other axes". The text the system actually displays is
   "Track costs & revenues by project, department, etc". A label is reproduced, not authored, so
   section 1.1 quotes the emitted text; [README.md](README.md) and [workflows.md](workflows.md)
   have been aligned with it.
3. **The name of the base plan's column.** The former draft called it a "project plan account"
   column and named the generated columns "plan account" columns. The reproduced names are
   `account_id` for the base plan and `x_plan<plan identifier>_id` for every other root plan, as
   [entities.md](entities.md) section 3 states.
4. **The visibility of the three derived totals.** The former draft said the debit and the credit
   are restricted to the accounting groups while the balance is shown to everyone. In the account
   list all three columns carry that restriction once the accounting capability is installed, and
   the debit and the credit are hidden columns without it; the balance is unrestricted only on the
   *Gross Margin* button of the account's form. Section 8 states both halves, and
   [business-rules.md](business-rules.md) `AA-111` has been made precise in the same way.
5. **The base-plan parameter message.** The former draft rewrote it in full words. Messages are
   reproduced, so section 2 quotes "The value for the key must be the ID to a valid analytic plan
   that is not a subplan" as the system emits it, with the placeholder described in words.
6. **Screens moved.** The former draft listed the screens, their stable paths and their empty-state
   messages in this file. They belong to [interfaces.md](interfaces.md) under the charter's division
   of the eleven documents, and every row of that list is now there, with the menu paths added.
7. **Sections added.** The order of the configuration steps (section 15), the sequences, scheduled
   jobs, message templates and activity types statement (section 11) and the company consistency
   table (section 10) are new, written from the source, because a reader configuring the domain has
   to know both what exists and what deliberately does not.
