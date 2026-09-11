# Coverage and evidence

This report states what the specification covers, how strong the evidence is for each part, and what is still uncertain. It is regenerated from the catalogues, so its numbers are measured rather than claimed.

Read it before making an equivalence claim. Cataloguing an entity is not the same as having specified every decision it makes, and specifying a decision is not the same as having checked its arithmetic against a running system.

## The four kinds of evidence

The specification distinguishes four kinds of evidence, from strongest structural guarantee to weakest behavioural one. They are not interchangeable.

| Kind | What it means | How it was produced | Confidence |
|---|---|---|---|
| Structural inventory | Every entity, field, relation, constraint, operation, route, view, action, menu, job, group, access right and record rule that the system declares is catalogued. | Every capability package definition was parsed mechanically. | Complete and mechanically checkable. A missing entry is a tooling defect, not a judgement. |
| Live schema | Every table, column, index, foreign key and constraint that a full installation actually creates is catalogued, together with the entity registry as it exists after every extension is applied. | Every capability package was installed into a database and the result introspected. | Complete for the installed set. This is observed fact, not inference from source. |
| Narrative specification | The decisions, states, rules, formulas and consequences are written out in prose, tables, numbered procedures and formulas, with worked examples. | Authored by reading the behaviour and its tests. | Varies by domain; measured below by document presence and depth. |
| Numerical verification | The arithmetic of a calculation has been independently recomputed and agrees. | Worked examples carried through by hand and checked against the behaviour and its tests. | Partial. Not the same as execution against a running reference application. |

A fifth kind, **execution against a running reference application**, would be the strongest evidence: driving the same operation on both systems and comparing the records field by field. It has not been performed. The [equivalence test plan](equivalence-test-plan.md) describes the harness that would do it, and any rebuild that can obtain a running instance should add it.

## Structural baseline

These counts are measured. They are the denominator for every coverage statement below.

| Measure | Count |
|---|---|
| Capability packages catalogued | 620 |
| Entities catalogued | 983 |
| Fields declared across entities | 14,028 |
| Fields resolved at run time after every extension | 23,235 |
| Operations catalogued on entities | 18,344 |
| Validation and error messages captured verbatim | 2,539 |
| Declared constraints | 390 |
| Computed fields with declared dependencies | 3,979 |
| Relational fields mapped | 3,985 |
| Selection fields with enumerated values | 853 |
| Persistent tables in a full installation | 1,240 |
| Columns across those tables | 13,518 |
| Indexes | 2,933 |
| Foreign keys | 4,575 |
| Unique and check constraints in the live schema | 296 |
| Association tables for many-to-many relations | 422 |
| Routes exposed over the transport | 1,023 |
| View declarations | 3,671 |
| Window actions | 978 |
| Server actions | 149 |
| Report definitions | 94 |
| Menus | 893 |
| Message templates | 69 |
| Rendering templates | 2,163 |
| Scheduled jobs | 93 |
| Security groups | 140 |
| Access rights | 1,933 |
| Record rules | 576 |
| Shipped reference data sets | 155 |
| Country chart template data sets | 1,078 |

Every entity in the catalogue is assigned to exactly one owning domain, so there is no part of the system that no document is responsible for. The assignment is in `schemas/traceability/entity-to-document.json`.

## Narrative coverage by domain

Of 45 domains, 0 have all eleven documents, 11 are partially written and 34 are not yet started. The narrative currently holds 3,563 lines.

A domain is counted complete when all eleven documents exist. Depth is reported separately, because an eleven-document folder of thin documents is not a specification.

| Domain | Documents | Lines | Entities | Fields | Operations | Messages | Narrative |
|---|---|---|---|---|---|---|---|
| [accounts-receivable](../domains/accounts-receivable/) | 2 of 11 | 936 | 57 | 1,522 | 2,951 | 461 | in progress |
| [general-ledger](../domains/general-ledger/) | 2 of 11 | 689 | 55 | 1,504 | 2,940 | 457 | in progress |
| [products-and-catalog](../domains/products-and-catalog/) | 2 of 11 | 553 | 30 | 501 | 784 | 110 | in progress |
| [inventory-operations](../domains/inventory-operations/) | 2 of 11 | 444 | 55 | 968 | 1,260 | 145 | in progress |
| [units-of-measure-and-packaging](../domains/units-of-measure-and-packaging/) | 1 of 11 | 228 | 1 | 22 | 25 | 5 | in progress |
| [inventory-valuation-and-costing](../domains/inventory-valuation-and-costing/) | 1 of 11 | 173 | 6 | 62 | 35 | 6 | in progress |
| [sales](../domains/sales/) | 1 of 11 | 132 | 15 | 447 | 729 | 83 | in progress |
| [purchasing](../domains/purchasing/) | 1 of 11 | 128 | 13 | 232 | 298 | 29 | in progress |
| [payments-and-bank-reconciliation](../domains/payments-and-bank-reconciliation/) | 1 of 11 | 109 | 59 | 1,543 | 2,971 | 465 | in progress |
| [accounts-payable](../domains/accounts-payable/) | 1 of 11 | 93 | 57 | 1,514 | 2,947 | 461 | in progress |
| [taxes](../domains/taxes/) | 1 of 11 | 78 | 59 | 1,533 | 2,988 | 461 | in progress |
| [analytic-accounting](../domains/analytic-accounting/) | 0 of 11 | 0 | 7 | 103 | 168 | 28 | not started |
| [attendances-and-working-time](../domains/attendances-and-working-time/) | 0 of 11 | 0 | 10 | 151 | 194 | 15 | not started |
| [automation-and-integration](../domains/automation-and-integration/) | 0 of 11 | 0 | 28 | 173 | 224 | 26 | not started |
| [calendar-and-scheduling](../domains/calendar-and-scheduling/) | 0 of 11 | 0 | 17 | 154 | 328 | 35 | not started |
| [contacts-and-organizations](../domains/contacts-and-organizations/) | 0 of 11 | 0 | 132 | 2,435 | 2,890 | 516 | not started |
| [customer-relationship-management](../domains/customer-relationship-management/) | 0 of 11 | 0 | 40 | 388 | 392 | 40 | not started |
| [delivery-and-shipping](../domains/delivery-and-shipping/) | 0 of 11 | 0 | 4 | 74 | 77 | 14 | not started |
| [electronic-invoicing-and-document-exchange](../domains/electronic-invoicing-and-document-exchange/) | 0 of 11 | 0 | 29 | 106 | 974 | 80 | not started |
| [events](../domains/events/) | 0 of 11 | 0 | 39 | 514 | 401 | 29 | not started |
| [expenses](../domains/expenses/) | 0 of 11 | 0 | 6 | 80 | 113 | 31 | not started |
| [financial-reporting](../domains/financial-reporting/) | 0 of 11 | 0 | 57 | 1,509 | 2,942 | 457 | not started |
| [fiscal-localizations](../domains/fiscal-localizations/) | 0 of 11 | 0 | 98 | 521 | 784 | 104 | not started |
| [fleet](../domains/fleet/) | 0 of 11 | 0 | 14 | 192 | 95 | 4 | not started |
| [human-resources-core](../domains/human-resources-core/) | 0 of 11 | 0 | 31 | 564 | 476 | 43 | not started |
| [identity-and-access](../domains/identity-and-access/) | 0 of 11 | 0 | 145 | 2,553 | 2,987 | 517 | not started |
| [learning-surveys-and-gamification](../domains/learning-surveys-and-gamification/) | 0 of 11 | 0 | 29 | 548 | 429 | 54 | not started |
| [loyalty-and-promotions](../domains/loyalty-and-promotions/) | 0 of 11 | 0 | 12 | 153 | 123 | 23 | not started |
| [lunch-ordering](../domains/lunch-ordering/) | 0 of 11 | 0 | 9 | 138 | 67 | 8 | not started |
| [manufacturing](../domains/manufacturing/) | 0 of 11 | 0 | 28 | 357 | 492 | 78 | not started |
| [marketing-and-mass-mailing](../domains/marketing-and-mass-mailing/) | 0 of 11 | 0 | 28 | 308 | 282 | 26 | not started |
| [messaging-and-activities](../domains/messaging-and-activities/) | 0 of 11 | 0 | 89 | 918 | 1,277 | 139 | not started |
| [multi-currency](../domains/multi-currency/) | 0 of 11 | 0 | 179 | 3,918 | 5,791 | 960 | not started |
| [payment-providers](../domains/payment-providers/) | 0 of 11 | 0 | 6 | 224 | 272 | 59 | not started |
| [point-of-sale](../domains/point-of-sale/) | 0 of 11 | 0 | 30 | 584 | 740 | 189 | not started |
| [pricing-and-pricelists](../domains/pricing-and-pricelists/) | 0 of 11 | 0 | 27 | 481 | 762 | 99 | not started |
| [projects-and-tasks](../domains/projects-and-tasks/) | 0 of 11 | 0 | 24 | 399 | 529 | 24 | not started |
| [recruitment](../domains/recruitment/) | 0 of 11 | 0 | 13 | 130 | 98 | 10 | not started |
| [repair-and-maintenance](../domains/repair-and-maintenance/) | 0 of 11 | 0 | 9 | 142 | 104 | 10 | not started |
| [replenishment-and-procurement](../domains/replenishment-and-procurement/) | 0 of 11 | 0 | 52 | 908 | 1,192 | 130 | not started |
| [spreadsheets-and-dashboards](../domains/spreadsheets-and-dashboards/) | 0 of 11 | 0 | 5 | 24 | 25 | 3 | not started |
| [time-off](../domains/time-off/) | 0 of 11 | 0 | 16 | 278 | 285 | 56 | not started |
| [timesheets](../domains/timesheets/) | 0 of 11 | 0 | 5 | 52 | 27 | 0 | not started |
| [website-and-storefront](../domains/website-and-storefront/) | 0 of 11 | 0 | 91 | 995 | 851 | 99 | not started |
| [work-entries](../domains/work-entries/) | 0 of 11 | 0 | 4 | 48 | 42 | 10 | not started |

### Document presence in detail

| Domain | README | entities | state machines | workflows | business rules | calculations | accounting effects | configuration | interfaces | acceptance criteria | glossary |
|---|---|---|---|---|---|---|---|---|---|---|---|
| accounts-payable | 93 | — | — | — | — | — | — | — | — | — | — |
| accounts-receivable | 130 | 806 | — | — | — | — | — | — | — | — | — |
| analytic-accounting | — | — | — | — | — | — | — | — | — | — | — |
| attendances-and-working-time | — | — | — | — | — | — | — | — | — | — | — |
| automation-and-integration | — | — | — | — | — | — | — | — | — | — | — |
| calendar-and-scheduling | — | — | — | — | — | — | — | — | — | — | — |
| contacts-and-organizations | — | — | — | — | — | — | — | — | — | — | — |
| customer-relationship-management | — | — | — | — | — | — | — | — | — | — | — |
| delivery-and-shipping | — | — | — | — | — | — | — | — | — | — | — |
| electronic-invoicing-and-document-exchange | — | — | — | — | — | — | — | — | — | — | — |
| events | — | — | — | — | — | — | — | — | — | — | — |
| expenses | — | — | — | — | — | — | — | — | — | — | — |
| financial-reporting | — | — | — | — | — | — | — | — | — | — | — |
| fiscal-localizations | — | — | — | — | — | — | — | — | — | — | — |
| fleet | — | — | — | — | — | — | — | — | — | — | — |
| general-ledger | 101 | 588 | — | — | — | — | — | — | — | — | — |
| human-resources-core | — | — | — | — | — | — | — | — | — | — | — |
| identity-and-access | — | — | — | — | — | — | — | — | — | — | — |
| inventory-operations | 104 | — | — | — | — | — | — | — | — | — | 340 |
| inventory-valuation-and-costing | 173 | — | — | — | — | — | — | — | — | — | — |
| learning-surveys-and-gamification | — | — | — | — | — | — | — | — | — | — | — |
| loyalty-and-promotions | — | — | — | — | — | — | — | — | — | — | — |
| lunch-ordering | — | — | — | — | — | — | — | — | — | — | — |
| manufacturing | — | — | — | — | — | — | — | — | — | — | — |
| marketing-and-mass-mailing | — | — | — | — | — | — | — | — | — | — | — |
| messaging-and-activities | — | — | — | — | — | — | — | — | — | — | — |
| multi-currency | — | — | — | — | — | — | — | — | — | — | — |
| payment-providers | — | — | — | — | — | — | — | — | — | — | — |
| payments-and-bank-reconciliation | 109 | — | — | — | — | — | — | — | — | — | — |
| point-of-sale | — | — | — | — | — | — | — | — | — | — | — |
| pricing-and-pricelists | — | — | — | — | — | — | — | — | — | — | — |
| products-and-catalog | 188 | — | — | — | — | — | — | — | — | — | 365 |
| projects-and-tasks | — | — | — | — | — | — | — | — | — | — | — |
| purchasing | 128 | — | — | — | — | — | — | — | — | — | — |
| recruitment | — | — | — | — | — | — | — | — | — | — | — |
| repair-and-maintenance | — | — | — | — | — | — | — | — | — | — | — |
| replenishment-and-procurement | — | — | — | — | — | — | — | — | — | — | — |
| sales | 132 | — | — | — | — | — | — | — | — | — | — |
| spreadsheets-and-dashboards | — | — | — | — | — | — | — | — | — | — | — |
| taxes | 78 | — | — | — | — | — | — | — | — | — | — |
| time-off | — | — | — | — | — | — | — | — | — | — | — |
| timesheets | — | — | — | — | — | — | — | — | — | — | — |
| units-of-measure-and-packaging | 228 | — | — | — | — | — | — | — | — | — | — |
| website-and-storefront | — | — | — | — | — | — | — | — | — | — | — |
| work-entries | — | — | — | — | — | — | — | — | — | — | — |

A dash means the document does not yet exist. A number is its line count.

## Repository contents

| Grouping | Key | Files | Lines |
|---|---|---|---|
| purpose | README.md documentation | 1 | 15 |
| purpose | data documentation | 1 | 11 |
| purpose | domain specification | 16 | 3,582 |
| purpose | interfaces documentation | 1 | 12 |
| purpose | machine-readable catalogue | 2,515 | 3,939,144 |
| purpose | overview documentation | 1 | 14 |
| purpose | references documentation | 997 | 108,928 |
| purpose | reimplementation documentation | 6 | 661 |
| purpose | repository root | 2 | 201 |
| purpose | runtime documentation | 1 | 16 |

## What is uncertain

The following are stated plainly so that a rebuild does not mistake silence for completeness.

1. **No execution against a running reference application.** Every behavioural statement was derived by reading the system's definition and its own test expectations. Where the two disagree with the prose, the prose is wrong. A differential harness is the correct remedy and is specified in the equivalence test plan.

2. **Editions beyond the openly available set.** The specification covers the capability packages present in the source baseline. Capabilities delivered only in a commercial edition, including some payroll engines and some advanced financial reporting, are outside the baseline and are not specified. Where a documented mechanism has an extension point that such a package would use, the extension point is specified and the package behind it is not.

3. **Client-side behaviour.** Screen behaviour is specified as the contract a client must honour: which fields, buttons, filters and groupings a view declares and what each means. Pixel layout, animation and styling are deliberately out of scope. Where business logic runs on the client because it must work without a network, notably at the sales counter, that logic is specified as rules, and the requirement that it match the server exactly is stated.

4. **Third-party service internals.** Integrations with payment providers, carriers, exchange networks, mail, calendar and metered services are specified as contracts: what is sent, what is received, how authenticity is established, and how failures and retries are handled. The remote services themselves are not specified and their behaviour can change independently.

5. **Depth varies by domain.** The table above reports line counts precisely so that a reader can see where depth is thin. A thin domain is a known gap, not a claim of simplicity.

6. **Numerical verification is partial.** Worked examples were carried through by hand for the calculations that carry money and quantity. They have not all been machine-checked. The mathematics catalogue exists so that they can be, and doing so should be the first task of a rebuild rather than the last.

## How to use this report

Before claiming a conformance level from [conformance profiles](conformance-profiles.md), check three things. First, that every domain in the claim shows a complete narrative here. Second, that the artefacts in scope for those domains have tests in the four reporting columns of the equivalence test plan, and specifically that the verified count, not the exercised count, covers them. Third, that every acknowledged difference is recorded with a reason and a decision.

An honest partial claim is more useful than an overstated complete one, because it tells the next team where to look.

