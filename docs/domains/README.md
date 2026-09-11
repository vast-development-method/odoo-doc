# Domains

One folder per business domain. Every folder contains the same eleven documents, so a reader always knows where to find a fact.

| File | Content |
|---|---|
| `README.md` | Scope, capabilities covered, the entities in the domain, reading order, dependencies on other domains |
| `entities.md` | Every entity in full: purpose, lifecycle, complete field table, relations, uniqueness, defaults, computed rules, ordering, display name, archival, company behaviour |
| `state-machines.md` | Every state field: states, transitions, guards, side effects, diagrams |
| `workflows.md` | End-to-end operational procedures, step by step, with the records each step creates or changes |
| `business-rules.md` | Validations, constraints, invariants, exact error messages, permission checks, locking rules |
| `calculations.md` | Every formula and algorithm with rounding, precision, currency and unit handling, and worked numeric examples |
| `accounting-effects.md` | Every ledger entry the domain produces, item by item |
| `configuration.md` | Settings, system parameters, sequences, default records, groups, access rights, record rules, scheduled jobs |
| `interfaces.md` | Menus, views, named operations, routes, printable documents, message templates, external integrations, import and export |
| `acceptance-criteria.md` | Numbered Given, When and Then scenarios with concrete numbers |
| `glossary.md` | Every term of the domain, defined |

Beyond the domain folders, [cross-domain transactions](cross-domain-transactions.md) traces what one business event does across every part of the system at once, with the money followed to the last unit.

## The domains

### Platform

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [automation and integration](automation-and-integration/) | 28 | 173 | 224 | 0 of 11 |
| [contacts and organizations](contacts-and-organizations/) | 132 | 2,435 | 2,890 | 0 of 11 |
| [identity and access](identity-and-access/) | 145 | 2,553 | 2,987 | 0 of 11 |
| [messaging and activities](messaging-and-activities/) | 89 | 918 | 1,277 | 0 of 11 |
| [spreadsheets and dashboards](spreadsheets-and-dashboards/) | 5 | 24 | 25 | 0 of 11 |

### Accounting

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [accounts payable](accounts-payable/) | 57 | 1,514 | 2,947 | 1 of 11 |
| [accounts receivable](accounts-receivable/) | 57 | 1,522 | 2,951 | 2 of 11 |
| [analytic accounting](analytic-accounting/) | 7 | 103 | 168 | 0 of 11 |
| [electronic invoicing and document exchange](electronic-invoicing-and-document-exchange/) | 29 | 106 | 974 | 0 of 11 |
| [financial reporting](financial-reporting/) | 57 | 1,509 | 2,942 | 0 of 11 |
| [fiscal localizations](fiscal-localizations/) | 98 | 521 | 784 | 0 of 11 |
| [general ledger](general-ledger/) | 55 | 1,504 | 2,940 | 2 of 11 |
| [multi currency](multi-currency/) | 179 | 3,918 | 5,791 | 0 of 11 |
| [payments and bank reconciliation](payments-and-bank-reconciliation/) | 59 | 1,543 | 2,971 | 1 of 11 |
| [taxes](taxes/) | 59 | 1,533 | 2,988 | 1 of 11 |

### Supply chain

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [delivery and shipping](delivery-and-shipping/) | 4 | 74 | 77 | 0 of 11 |
| [inventory operations](inventory-operations/) | 55 | 968 | 1,260 | 2 of 11 |
| [inventory valuation and costing](inventory-valuation-and-costing/) | 6 | 62 | 35 | 1 of 11 |
| [manufacturing](manufacturing/) | 28 | 357 | 492 | 0 of 11 |
| [products and catalog](products-and-catalog/) | 30 | 501 | 784 | 2 of 11 |
| [purchasing](purchasing/) | 13 | 232 | 298 | 1 of 11 |
| [repair and maintenance](repair-and-maintenance/) | 9 | 142 | 104 | 0 of 11 |
| [replenishment and procurement](replenishment-and-procurement/) | 52 | 908 | 1,192 | 0 of 11 |
| [units of measure and packaging](units-of-measure-and-packaging/) | 1 | 22 | 25 | 1 of 11 |

### Sales

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [customer relationship management](customer-relationship-management/) | 40 | 388 | 392 | 0 of 11 |
| [loyalty and promotions](loyalty-and-promotions/) | 12 | 153 | 123 | 0 of 11 |
| [payment providers](payment-providers/) | 6 | 224 | 272 | 0 of 11 |
| [point of sale](point-of-sale/) | 30 | 584 | 740 | 0 of 11 |
| [pricing and pricelists](pricing-and-pricelists/) | 27 | 481 | 762 | 0 of 11 |
| [sales](sales/) | 15 | 447 | 729 | 1 of 11 |
| [website and storefront](website-and-storefront/) | 91 | 995 | 851 | 0 of 11 |

### People

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [attendances and working time](attendances-and-working-time/) | 10 | 151 | 194 | 0 of 11 |
| [expenses](expenses/) | 6 | 80 | 113 | 0 of 11 |
| [fleet](fleet/) | 14 | 192 | 95 | 0 of 11 |
| [human resources core](human-resources-core/) | 31 | 564 | 476 | 0 of 11 |
| [lunch ordering](lunch-ordering/) | 9 | 138 | 67 | 0 of 11 |
| [recruitment](recruitment/) | 13 | 130 | 98 | 0 of 11 |
| [time off](time-off/) | 16 | 278 | 285 | 0 of 11 |
| [work entries](work-entries/) | 4 | 48 | 42 | 0 of 11 |

### Services

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [projects and tasks](projects-and-tasks/) | 24 | 399 | 529 | 0 of 11 |
| [timesheets](timesheets/) | 5 | 52 | 27 | 0 of 11 |

### Marketing

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [events](events/) | 39 | 514 | 401 | 0 of 11 |
| [learning surveys and gamification](learning-surveys-and-gamification/) | 29 | 548 | 429 | 0 of 11 |
| [marketing and mass mailing](marketing-and-mass-mailing/) | 28 | 308 | 282 | 0 of 11 |

### Communication

| Domain | Entities | Fields | Operations | Documents |
|---|---|---|---|---|
| [calendar and scheduling](calendar-and-scheduling/) | 17 | 154 | 328 | 0 of 11 |

Counts come from the generated catalogues and are regenerated with them. Current depth per document is reported in [coverage and evidence](../reimplementation/coverage-and-evidence.md).
