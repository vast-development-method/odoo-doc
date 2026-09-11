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

Of 45 domains, 12 have all eleven documents, 14 are partially written and 19 are not yet started. The narrative currently holds 156,971 lines.

A domain is counted complete when all eleven documents exist. Depth is reported separately, because an eleven-document folder of thin documents is not a specification.

| Domain | Documents | Lines | Entities | Fields | Operations | Messages | Narrative |
|---|---|---|---|---|---|---|---|
| [inventory-valuation-and-costing](../domains/inventory-valuation-and-costing/) | 11 of 11 | 10,135 | 6 | 62 | 35 | 6 | complete |
| [contacts-and-organizations](../domains/contacts-and-organizations/) | 9 of 11 | 10,037 | 132 | 2,435 | 2,890 | 516 | in progress |
| [point-of-sale](../domains/point-of-sale/) | 10 of 11 | 9,700 | 30 | 584 | 740 | 189 | in progress |
| [sales](../domains/sales/) | 11 of 11 | 9,015 | 15 | 447 | 729 | 83 | complete |
| [taxes](../domains/taxes/) | 11 of 11 | 8,683 | 59 | 1,533 | 2,988 | 461 | complete |
| [products-and-catalog](../domains/products-and-catalog/) | 11 of 11 | 8,632 | 30 | 501 | 784 | 110 | complete |
| [human-resources-core](../domains/human-resources-core/) | 10 of 11 | 8,609 | 31 | 564 | 476 | 43 | in progress |
| [manufacturing](../domains/manufacturing/) | 9 of 11 | 8,297 | 28 | 357 | 492 | 78 | in progress |
| [accounts-receivable](../domains/accounts-receivable/) | 11 of 11 | 8,251 | 57 | 1,522 | 2,951 | 461 | complete |
| [financial-reporting](../domains/financial-reporting/) | 11 of 11 | 8,124 | 57 | 1,509 | 2,942 | 457 | complete |
| [payments-and-bank-reconciliation](../domains/payments-and-bank-reconciliation/) | 11 of 11 | 8,061 | 59 | 1,543 | 2,971 | 465 | complete |
| [inventory-operations](../domains/inventory-operations/) | 11 of 11 | 8,019 | 55 | 968 | 1,260 | 145 | complete |
| [general-ledger](../domains/general-ledger/) | 11 of 11 | 8,005 | 55 | 1,504 | 2,940 | 457 | complete |
| [purchasing](../domains/purchasing/) | 11 of 11 | 7,876 | 13 | 232 | 298 | 29 | complete |
| [identity-and-access](../domains/identity-and-access/) | 9 of 11 | 7,687 | 145 | 2,553 | 2,987 | 517 | in progress |
| [projects-and-tasks](../domains/projects-and-tasks/) | 9 of 11 | 7,151 | 24 | 399 | 529 | 24 | in progress |
| [units-of-measure-and-packaging](../domains/units-of-measure-and-packaging/) | 11 of 11 | 6,791 | 1 | 22 | 25 | 5 | complete |
| [accounts-payable](../domains/accounts-payable/) | 11 of 11 | 6,014 | 57 | 1,514 | 2,947 | 461 | complete |
| [messaging-and-activities](../domains/messaging-and-activities/) | 3 of 11 | 4,056 | 89 | 918 | 1,277 | 139 | in progress |
| [customer-relationship-management](../domains/customer-relationship-management/) | 2 of 11 | 967 | 40 | 388 | 392 | 40 | in progress |
| [time-off](../domains/time-off/) | 2 of 11 | 951 | 16 | 278 | 285 | 56 | in progress |
| [pricing-and-pricelists](../domains/pricing-and-pricelists/) | 2 of 11 | 741 | 27 | 481 | 762 | 99 | in progress |
| [multi-currency](../domains/multi-currency/) | 2 of 11 | 669 | 179 | 3,918 | 5,791 | 960 | in progress |
| [attendances-and-working-time](../domains/attendances-and-working-time/) | 1 of 11 | 183 | 10 | 151 | 194 | 15 | in progress |
| [analytic-accounting](../domains/analytic-accounting/) | 1 of 11 | 178 | 7 | 103 | 168 | 28 | in progress |
| [expenses](../domains/expenses/) | 1 of 11 | 139 | 6 | 80 | 113 | 31 | in progress |
| [automation-and-integration](../domains/automation-and-integration/) | 0 of 11 | 0 | 28 | 173 | 224 | 26 | not started |
| [calendar-and-scheduling](../domains/calendar-and-scheduling/) | 0 of 11 | 0 | 17 | 154 | 328 | 35 | not started |
| [delivery-and-shipping](../domains/delivery-and-shipping/) | 0 of 11 | 0 | 4 | 74 | 77 | 14 | not started |
| [electronic-invoicing-and-document-exchange](../domains/electronic-invoicing-and-document-exchange/) | 0 of 11 | 0 | 29 | 106 | 974 | 80 | not started |
| [events](../domains/events/) | 0 of 11 | 0 | 39 | 514 | 401 | 29 | not started |
| [fiscal-localizations](../domains/fiscal-localizations/) | 0 of 11 | 0 | 98 | 521 | 784 | 104 | not started |
| [fleet](../domains/fleet/) | 0 of 11 | 0 | 14 | 192 | 95 | 4 | not started |
| [learning-surveys-and-gamification](../domains/learning-surveys-and-gamification/) | 0 of 11 | 0 | 29 | 548 | 429 | 54 | not started |
| [loyalty-and-promotions](../domains/loyalty-and-promotions/) | 0 of 11 | 0 | 12 | 153 | 123 | 23 | not started |
| [lunch-ordering](../domains/lunch-ordering/) | 0 of 11 | 0 | 9 | 138 | 67 | 8 | not started |
| [marketing-and-mass-mailing](../domains/marketing-and-mass-mailing/) | 0 of 11 | 0 | 28 | 308 | 282 | 26 | not started |
| [payment-providers](../domains/payment-providers/) | 0 of 11 | 0 | 6 | 224 | 272 | 59 | not started |
| [recruitment](../domains/recruitment/) | 0 of 11 | 0 | 13 | 130 | 98 | 10 | not started |
| [repair-and-maintenance](../domains/repair-and-maintenance/) | 0 of 11 | 0 | 9 | 142 | 104 | 10 | not started |
| [replenishment-and-procurement](../domains/replenishment-and-procurement/) | 0 of 11 | 0 | 52 | 908 | 1,192 | 130 | not started |
| [spreadsheets-and-dashboards](../domains/spreadsheets-and-dashboards/) | 0 of 11 | 0 | 5 | 24 | 25 | 3 | not started |
| [timesheets](../domains/timesheets/) | 0 of 11 | 0 | 5 | 52 | 27 | 0 | not started |
| [website-and-storefront](../domains/website-and-storefront/) | 0 of 11 | 0 | 91 | 995 | 851 | 99 | not started |
| [work-entries](../domains/work-entries/) | 0 of 11 | 0 | 4 | 48 | 42 | 10 | not started |

### Document presence in detail

| Domain | README | entities | state machines | workflows | business rules | calculations | accounting effects | configuration | interfaces | acceptance criteria | glossary |
|---|---|---|---|---|---|---|---|---|---|---|---|
| accounts-payable | 142 | 810 | 434 | 418 | 451 | 1187 | 596 | 299 | 413 | 987 | 277 |
| accounts-receivable | 130 | 823 | 411 | 537 | 598 | 1938 | 746 | 266 | 660 | 1665 | 477 |
| analytic-accounting | 178 | — | — | — | — | — | — | — | — | — | — |
| attendances-and-working-time | 183 | — | — | — | — | — | — | — | — | — | — |
| automation-and-integration | — | — | — | — | — | — | — | — | — | — | — |
| calendar-and-scheduling | — | — | — | — | — | — | — | — | — | — | — |
| contacts-and-organizations | 250 | 1384 | 346 | 807 | 775 | 2847 | 209 | 3187 | — | — | 232 |
| customer-relationship-management | 164 | 803 | — | — | — | — | — | — | — | — | — |
| delivery-and-shipping | — | — | — | — | — | — | — | — | — | — | — |
| electronic-invoicing-and-document-exchange | — | — | — | — | — | — | — | — | — | — | — |
| events | — | — | — | — | — | — | — | — | — | — | — |
| expenses | 139 | — | — | — | — | — | — | — | — | — | — |
| financial-reporting | 157 | 867 | 373 | 504 | 654 | 2282 | 461 | 330 | 372 | 1700 | 424 |
| fiscal-localizations | — | — | — | — | — | — | — | — | — | — | — |
| fleet | — | — | — | — | — | — | — | — | — | — | — |
| general-ledger | 100 | 1226 | 449 | 837 | 705 | 1868 | 383 | 388 | 514 | 1332 | 203 |
| human-resources-core | 215 | 1606 | 474 | 811 | 802 | 1499 | 102 | 552 | 554 | 1994 | — |
| identity-and-access | 177 | 2123 | 517 | 776 | 1681 | 1207 | 99 | 633 | 474 | — | — |
| inventory-operations | 104 | 1150 | 454 | 717 | 663 | 1774 | 114 | 371 | 538 | 1794 | 340 |
| inventory-valuation-and-costing | 219 | 1305 | 276 | 840 | 556 | 1947 | 797 | 370 | 551 | 2882 | 392 |
| learning-surveys-and-gamification | — | — | — | — | — | — | — | — | — | — | — |
| loyalty-and-promotions | — | — | — | — | — | — | — | — | — | — | — |
| lunch-ordering | — | — | — | — | — | — | — | — | — | — | — |
| manufacturing | 206 | 2084 | 447 | 1118 | 695 | 1752 | 721 | 577 | 697 | — | — |
| marketing-and-mass-mailing | — | — | — | — | — | — | — | — | — | — | — |
| messaging-and-activities | 240 | 2872 | 944 | — | — | — | — | — | — | — | — |
| multi-currency | 177 | 492 | — | — | — | — | — | — | — | — | — |
| payment-providers | — | — | — | — | — | — | — | — | — | — | — |
| payments-and-bank-reconciliation | 109 | 770 | 296 | 417 | 582 | 2408 | 532 | 281 | 460 | 1676 | 530 |
| point-of-sale | 169 | 1460 | 372 | 1090 | 668 | 1509 | 1663 | 495 | 548 | 1726 | — |
| pricing-and-pricelists | 172 | 569 | — | — | — | — | — | — | — | — | — |
| products-and-catalog | 241 | 1229 | 385 | 663 | 1005 | 2064 | 210 | 352 | 438 | 1680 | 365 |
| projects-and-tasks | 163 | 1725 | 584 | 1033 | 784 | 1544 | 212 | 515 | 591 | — | — |
| purchasing | 145 | 850 | 372 | 1432 | 582 | 1003 | 518 | 349 | 578 | 1524 | 523 |
| recruitment | — | — | — | — | — | — | — | — | — | — | — |
| repair-and-maintenance | — | — | — | — | — | — | — | — | — | — | — |
| replenishment-and-procurement | — | — | — | — | — | — | — | — | — | — | — |
| sales | 199 | 1038 | 547 | 1177 | 679 | 1202 | 571 | 480 | 496 | 1860 | 766 |
| spreadsheets-and-dashboards | — | — | — | — | — | — | — | — | — | — | — |
| taxes | 78 | 727 | 292 | 475 | 480 | 3737 | 445 | 294 | 331 | 1451 | 373 |
| time-off | 147 | 804 | — | — | — | — | — | — | — | — | — |
| timesheets | — | — | — | — | — | — | — | — | — | — | — |
| units-of-measure-and-packaging | 228 | 629 | 292 | 655 | 783 | 1915 | 245 | 420 | 405 | 924 | 295 |
| website-and-storefront | — | — | — | — | — | — | — | — | — | — | — |
| work-entries | — | — | — | — | — | — | — | — | — | — | — |

A dash means the document does not yet exist. A number is its line count.

## Repository contents

| Grouping | Key | Files | Lines |
|---|---|---|---|
| purpose | README.md documentation | 1 | 92 |
| purpose | data documentation | 1 | 11 |
| purpose | domain specification | 203 | 157,079 |
| purpose | interfaces documentation | 1 | 12 |
| purpose | machine-readable catalogue | 2,519 | 4,060,385 |
| purpose | overview documentation | 6 | 6,259 |
| purpose | references documentation | 1,005 | 187,489 |
| purpose | reimplementation documentation | 7 | 867 |
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

## Rule compliance

The [documentation rules](../references/documentation-rules.md) are checked mechanically over every authored document. The check strips fenced blocks, code font, quoted messages and emphasis spans before looking, because the rules make those spans contractual: a stored value, a reproduced error message or a reproduced label must appear exactly as the system produces it.

What the check still reports is almost entirely reproduced text that the stripping rules do not recognise: a message quoted inside a table cell, a country code in a shipped data table, an example company name carrying a legal-form suffix, or an operator inside a pseudo-procedure. Each was inspected. Where a finding was genuine it was corrected: fixture names written in capitals were changed to ordinary title case, bare logical operators in numbered procedures were written as words, and acronyms in the specification's own prose were expanded.

The rule that no finding may be dismissed without inspection is itself part of the review procedure. A residual count is not a pass; a residual count whose every entry has been read and classified is.

## How to use this report

Before claiming a conformance level from [conformance profiles](conformance-profiles.md), check three things. First, that every domain in the claim shows a complete narrative here. Second, that the artefacts in scope for those domains have tests in the four reporting columns of the equivalence test plan, and specifically that the verified count, not the exercised count, covers them. Third, that every acknowledged difference is recorded with a reason and a decision.

An honest partial claim is more useful than an overstated complete one, because it tells the next team where to look.

