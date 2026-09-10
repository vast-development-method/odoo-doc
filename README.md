# Enterprise Resource Planning System: Language-Neutral Rebuild Specification

This repository is a complete, self-contained, language-neutral specification for rebuilding a mature, integrated enterprise resource planning system. It maps the observable application architecture, the business capabilities, the persistent data model, the service contracts, the operational workflows, the cross-domain accounting effects and the acceptance criteria of the system, without describing any implementation language, framework or deployment technology.

The intended outcome is **behavioral equivalence**: a replacement written in Go, Rust, PHP, C++, Java or any other suitable language must make the same business decisions, preserve the same records and financial consequences, enforce the same state transitions, exposures and validations, and offer the same operational capabilities.

The repository is licensed under the Apache License, Version 2.0 (see `LICENSE`).

## How to use this repository

1. Read this file end to end. It fixes the naming conventions, gives the architecture at a glance, maps the business capabilities, and lists the reading paths.
2. Read `docs/overview/` to understand the platform: the persistence layer, the entity and field system, inheritance and extension, security, views and actions, the package system, the request and transport model, reporting, messaging and scheduled execution.
3. Read `docs/data/` for the domain model, identity and value rules, the physical data catalog and the data loading contracts.
4. Read the domain folders under `docs/domains/` in the order given by the reimplementation sequence below. Every domain folder contains the same set of files: `README.md`, `entities.md`, `state-machines.md`, `workflows.md`, `business-rules.md`, `calculations.md`, `accounting-effects.md`, `configuration.md`, `interfaces.md`, `acceptance-criteria.md` and `glossary.md`.
5. Use `docs/references/` for the generated, exhaustive per-entity reference pages (every field, constraint, operation, validation message, view, action and access rule of every entity).
6. Use `schemas/` as the machine-readable counterpart: every entity, field, state, constraint, operation, route, report, scheduled job, access rule, record rule, sequence, reference data record and acceptance scenario is catalogued there in JavaScript Object Notation for tooling and code generation.
7. Use `docs/reimplementation/` for the build sequence, the milestone acceptance gates, the equivalence test plan and the traceability rules.

## Canonical naming conventions

The specification uses full words. No acronyms or abbreviations appear in headings, labels or prose. The following conventions are binding across `docs/` and `schemas/`:

| Convention | Rule |
|---|---|
| Entity names | Full business name in title case in prose (Journal Entry, Journal Item, Stock Move, Stock Quantity, Reordering Rule, Sales Order Line, Employee Version). Each entity also has a **transport name** (the dotted identifier used by the remote transport contract, for example `account.move`) and a **storage name** (the table name, for example `account_move`). Both are reproduced exactly, in code font, because external contracts and data exchange formats depend on them. The dictionary is in `docs/references/entity-name-dictionary.md` and `schemas/traceability/entity-name-dictionary.json`. |
| Field names | Storage names are reproduced exactly in code font (`partner_id`, `amount_untaxed`, `date_order`) and are always accompanied by their full label in words. The suffix `_id` on a storage name denotes a reference to one record; `_ids` denotes a reference to many records. |
| Reproduced identifiers | Storage names, transport names, column names, route paths, selection values, external identifiers and sequence codes are the only place where letters may be abbreviated, because they are contractual. Each such identifier is explained in words on first use in a document. |
| State values | Reproduced exactly as stored (`draft`, `posted`, `cancel`, `assigned`, `done`), each with its full label and meaning. |
| Operations | Descriptive verb phrases in prose ("post the journal entry") and reproduced operation names in code font in tables (`action_post`) so that the remote transport contract stays compatible. |
| Capability packages | The system is delivered as installable feature packages. They are named by full name (for example "Inventory", "Belgium Localization", "Point of Sale Restaurant"); the catalog is in `docs/references/capability-packages.md` and `schemas/source-artifacts/packages.json`. |
| Data types | `boolean`, `integer`, `float` (with decimal precision rule), `monetary` (float bound to a currency field), `char` (single line text), `text` (multi line), `html` (rich text), `date`, `datetime` (stored in coordinated universal time), `binary`, `image`, `selection`, `reference`, `many_to_one`, `one_to_many`, `many_to_many`, `json` (structured document), `properties`, `properties_definition`. |
| Shared fields | Every persistent entity carries `id`, `create_date`, `create_uid`, `write_date`, `write_uid`; archivable entities carry `active`. |
| Expansions | unit of measure, bill of materials, point of sale, customer relationship management, human resources, text message, manufacturing order, purchase order, sales order, request for quotation, identifier, universally unique identifier, uniform resource locator, rich text, Portable Document Format, comma-separated values, JavaScript Object Notation, Hypertext Transfer Protocol, first in first out, average cost, cost of goods sold, value-added tax, international bank account number, bank identifier code, Single Euro Payments Area, electronic data interchange, Universal Business Language, Cross Industry Invoice, application programming interface, remote procedure call, quick response code, make to order, key performance indicator, one-time password, two-factor authentication, optical character recognition, internet of things, Global Standards One, pan-European public procurement online network. |

Product names of third-party services (payment providers, shipping carriers, document exchange networks, mail services) are proper nouns and are described by their function on first use.

## Architecture at a glance

The system is a single modular application served from one relational database per tenant. Its layers, from the bottom up:

1. **Persistence and entity layer.** Every business object is an *entity* with typed fields persisted in one table per entity, plus association tables for many-to-many relations. Entities support classical inheritance (extending an existing entity in place, adding fields and overriding operations), prototype inheritance (copying an entity's definition into a new one) and delegation inheritance (embedding a parent record so the child transparently exposes the parent's fields). Fields can be stored, computed (with declared dependencies and automatic recomputation), related (following a path through relations), company-dependent, translatable, or tracked for audit. The layer provides record-set operations (create, read, update, delete, search, read-group, name-search, copy), an in-memory cache with invalidation, and a unit of work that flushes pending writes inside a database transaction with repeatable-read isolation and automatic retry on serialization conflicts.
2. **Business rules.** Constraints (database-level uniqueness and checks; entity-level validation rules with messages), on-change behaviors in forms, default values, sequences for document numbering, state machines on documents, scheduled actions, automation rules and server actions.
3. **Security.** Users, access groups arranged in privilege categories with implied-group closure, per-entity access rights (create, read, update, delete) per group, record rules (row-level filters per group and per operation), field-level group restrictions, multi-company scoping, and portal (external) versus internal users.
4. **Presentation contracts.** Views (form, list, kanban, search, calendar, pivot, graph, activity, hierarchy, gantt, map, cohort) are declarative documents describing which fields, buttons, filters and groupings a client renders; window actions open views on an entity with a domain filter and context; menus organize actions; client actions and web address actions cover non-entity screens. The client talks to the server through a remote procedure call transport over JavaScript Object Notation documents, with a fixed set of generic entity operations plus named business operations.
5. **Business domains.** Accounting (general ledger, receivables, payables, taxes, payments and bank reconciliation, multi-currency, analytic, financial reporting, electronic invoicing, fiscal localizations), supply chain (products, units of measure and packaging, inventory, valuation, replenishment, purchasing, manufacturing, repair and maintenance, delivery), sales (pricing, sales, loyalty, point of sale, customer relationship management, payment providers), human resources (core, time off, attendances, work entries, recruitment, expenses, fleet), services (projects, timesheets), communication (messaging and activities, calendar), marketing (events, mass mailing, surveys and learning), and web (website, storefront, portal), plus spreadsheets and dashboards and automation and integration services.
6. **Operational runtime.** Request lifecycle, sessions and authentication, transactions and concurrency, caching, scheduled job execution, the notification bus, attachments and the file store, report rendering (printable documents from templates), email sending and receiving, and background workers.

## Business capability map

| Group | Domain folder | Capabilities delivered |
|---|---|---|
| Platform | `docs/domains/identity-and-access` | Users, groups, privileges, record rules, password policies, two-factor authentication, passkeys, open authorization sign-in, directory servers, sign-up, session timeout, application keys, data protection |
| Platform | `docs/domains/contacts-and-organizations` | Contacts, companies, addresses, bank accounts, banks, currencies, countries, languages, industries, tags, contact enrichment, postal mail |
| Accounting | `docs/domains/general-ledger` | Chart of accounts, journals, journal entries and items, posting, numbering, reversal, lock dates, inalterability, reconciliation core |
| Accounting | `docs/domains/accounts-receivable` | Customer invoices, credit notes, receipts, payment terms, early payment discounts, cash rounding, sending, portal payment |
| Accounting | `docs/domains/accounts-payable` | Vendor bills, refunds, debit notes, bill import, automatic posting, check printing |
| Accounting | `docs/domains/taxes` | Tax engine, distribution lines, tax groups, grids and reports, cash basis, fiscal positions, withholding |
| Accounting | `docs/domains/payments-and-bank-reconciliation` | Payments, payment methods, bank statements, reconciliation models, partial and full reconciliation, structured references, quick response code payment |
| Accounting | `docs/domains/multi-currency` | Currencies, rates, conversion, foreign currency journal items, exchange differences |
| Accounting | `docs/domains/analytic-accounting` | Analytic plans, accounts, distributions, lines, applicability |
| Accounting | `docs/domains/financial-reporting` | Report engine: report definitions, lines, expressions, columns, tax report grids, audit trail |
| Accounting | `docs/domains/electronic-invoicing-and-document-exchange` | Electronic document formats, import and export, document exchange network, certificates |
| Accounting | `docs/domains/fiscal-localizations` | Country packages: chart templates, taxes, fiscal positions, reports, country rules |
| Supply chain | `docs/domains/products-and-catalog` | Products, variants, attributes, categories, barcodes, expiry, matrices, combos, documents |
| Supply chain | `docs/domains/units-of-measure-and-packaging` | Units, categories, conversion, packagings, package types, quantity matrices |
| Sales | `docs/domains/pricing-and-pricelists` | Pricelists, rules, discount policies, vendor prices |
| Supply chain | `docs/domains/inventory-operations` | Warehouses, locations, operation types, transfers, moves, quantities, lots, packages, batches, adjustments, barcodes |
| Supply chain | `docs/domains/inventory-valuation-and-costing` | Valuation layers, costing methods, automated entries, landed costs, cost of goods sold |
| Supply chain | `docs/domains/replenishment-and-procurement` | Routes, rules, reordering rules, scheduler, lead times, drop shipping |
| Supply chain | `docs/domains/purchasing` | Requests for quotation, purchase orders, receipts, bill control, agreements |
| Sales | `docs/domains/sales` | Quotations, orders, invoicing policies, down payments, templates, margins, teams, product configurator |
| Sales | `docs/domains/loyalty-and-promotions` | Programs, rules, rewards, coupons, gift cards, wallets |
| Supply chain | `docs/domains/manufacturing` | Bills of materials, manufacturing orders, work orders, work centers, subcontracting, costing |
| Supply chain | `docs/domains/repair-and-maintenance` | Repair orders, equipment, maintenance requests |
| Supply chain | `docs/domains/delivery-and-shipping` | Delivery methods, carriers, shipping cost rules, pickup points |
| Sales | `docs/domains/point-of-sale` | Configurations, sessions, orders, payments, restaurant, self-ordering, terminals |
| Sales | `docs/domains/customer-relationship-management` | Leads, opportunities, stages, scoring, assignment, partnerships, campaign tracking |
| Sales | `docs/domains/payment-providers` | Providers, methods, tokens, transactions |
| Human resources | `docs/domains/human-resources-core` | Employees, versions and contracts, departments, jobs, skills, organization chart, homeworking, presence |
| Human resources | `docs/domains/time-off` | Types, requests, allocations, accrual plans, public holidays |
| Human resources | `docs/domains/attendances-and-working-time` | Working schedules, resources, attendances, overtime |
| Human resources | `docs/domains/work-entries` | Work entry types, generation from schedules and time off, conflicts |
| Human resources | `docs/domains/recruitment` | Jobs, candidates, applicants, stages, job board |
| Human resources | `docs/domains/expenses` | Expenses, reports, approval, posting, re-invoicing |
| Human resources | `docs/domains/fleet` | Vehicles, models, contracts, services, odometer, costs |
| Human resources | `docs/domains/lunch-ordering` | Vendors, products, orders, alerts, cash moves |
| Services | `docs/domains/projects-and-tasks` | Projects, tasks, stages, milestones, to-do, updates |
| Services | `docs/domains/timesheets` | Timesheet lines, billing from timesheets, validation |
| Communication | `docs/domains/messaging-and-activities` | Threads, messages, followers, activities, templates, mail gateway, channels, live chat, digests |
| Communication | `docs/domains/calendar-and-scheduling` | Events, attendees, recurrence, alarms, external calendar synchronization surface |
| Marketing | `docs/domains/events` | Events, tickets, registrations, booths, tracks, sponsors |
| Marketing | `docs/domains/marketing-and-mass-mailing` | Mailing lists, mailings, text message campaigns, link tracking, campaign parameters, marketing cards |
| Marketing | `docs/domains/learning-surveys-and-gamification` | Surveys, courses, quizzes, certifications, badges, challenges, goals |
| Web | `docs/domains/website-and-storefront` | Websites, pages, storefront, cart and checkout, wishlist, comparison, forum, blog, portal |
| Platform | `docs/domains/spreadsheets-and-dashboards` | Spreadsheet documents, dashboards, boards, digests |
| Platform | `docs/domains/automation-and-integration` | Automation rules, server actions, webhooks, in-app purchase services, public application programming interface, remote transport |

## Documentation index

| Folder | Content |
|---|---|
| `docs/README.md` | Navigation of the documentation tree |
| `docs/overview/` | Architecture, package system, entity and field system, inheritance and extension, security model, views and actions, messaging, design principles |
| `docs/data/` | Domain model, persistence identity and value rules, physical data catalog, data loading and exchange |
| `docs/domains/` | One folder per business domain (see the capability map) |
| `docs/interfaces/` | Desktop workflows, endpoint catalog, external integrations, remote transport contracts, report and export documents, service layer |
| `docs/runtime/` | Request lifecycle, sessions, transactions and concurrency, caching, scheduled jobs, notification bus, mail gateway, attachments, report rendering, translation, background workers |
| `docs/references/` | Generated exhaustive references: entities, fields, operations, validation messages, routes, views, actions, menus, reports, scheduled jobs, sequences, groups, access rights, record rules, reference data, capability packages, entity name dictionary |
| `docs/reimplementation/` | Build sequence, milestones, acceptance gates, equivalence test plan, traceability rules |
| `schemas/` | Machine-readable catalogs (see below) |

## Machine-readable catalogs

| Folder | Content |
|---|---|
| `schemas/data/` | `entities/` one document per entity (fields, constraints, operations, validation messages, states, access, rules), `entity-index.json`, `physical-tables.json`, `relations.json`, `selection-values.json`, `reference-data/` |
| `schemas/interfaces/` | `routes.json`, `window-actions.json`, `server-actions.json`, `client-actions.json`, `menus.json`, `views/` per entity, `reports.json`, `mail-templates.json`, `remote-operations.json` |
| `schemas/operational/` | `scheduled-jobs.json`, `sequences.json`, `groups.json`, `access-rights.json`, `record-rules.json`, `system-parameters.json`, `decimal-precisions.json` |
| `schemas/source-artifacts/` | `packages.json` (capability packages, dependencies, categories), `package-dependency-graph.json`, `package-contents.json` |
| `schemas/source-file-distribution/` | Inventory of every file in this repository with purpose, domain, size and line count |
| `schemas/traceability/` | `entity-name-dictionary.json`, `capability-to-entity.json`, `entity-to-document.json`, `acceptance-to-workflow.json`, `coverage.json` |

## Recommended reading paths

- **Architect (two days):** this file; `docs/overview/*`; `docs/data/domain-model.md`; `docs/runtime/*`; `docs/reimplementation/build-sequence.md`.
- **Accounting engineer:** `docs/domains/general-ledger`, `taxes`, `accounts-receivable`, `accounts-payable`, `payments-and-bank-reconciliation`, `multi-currency`, `analytic-accounting`, `financial-reporting`, then `inventory-valuation-and-costing/accounting-effects.md`, `sales/accounting-effects.md`, `purchasing/accounting-effects.md`, `expenses/accounting-effects.md`, `point-of-sale/accounting-effects.md`.
- **Supply chain engineer:** `products-and-catalog`, `units-of-measure-and-packaging`, `inventory-operations`, `inventory-valuation-and-costing`, `replenishment-and-procurement`, `purchasing`, `manufacturing`, `delivery-and-shipping`, `repair-and-maintenance`.
- **Sales and commerce engineer:** `pricing-and-pricelists`, `sales`, `loyalty-and-promotions`, `point-of-sale`, `customer-relationship-management`, `payment-providers`, `website-and-storefront`.
- **People operations engineer:** `human-resources-core`, `attendances-and-working-time`, `time-off`, `work-entries`, `recruitment`, `expenses`, `fleet`, `lunch-ordering`, `timesheets`, `projects-and-tasks`.
- **Platform engineer:** `docs/overview`, `docs/runtime`, `docs/interfaces`, `identity-and-access`, `contacts-and-organizations`, `messaging-and-activities`, `automation-and-integration`, `spreadsheets-and-dashboards`, `schemas/`.
- **Tooling and code generation:** `schemas/README.md` then the catalogs.

## Reimplementation sequence

1. Platform foundation: entity registry, field types, inheritance, record sets, domains, unit of work, transactions, caching, sequences, external identifiers and data loading, translations.
2. Identity and access: users, groups, privileges, access rights, record rules, multi-company, sessions and authentication.
3. Presentation contracts and transport: views, actions, menus, remote operations, routes, attachments, report rendering.
4. Messaging and activities: threads, messages, followers, notifications, activities, mail gateway, scheduled jobs.
5. Contacts and organizations, currencies, countries, units of measure and packaging, products and catalog, pricing.
6. General ledger, taxes, multi-currency, analytic accounting, receivables, payables, payments and bank reconciliation, financial reporting, fiscal localizations, electronic invoicing.
7. Inventory operations, valuation and costing, replenishment, purchasing, delivery, manufacturing, repair and maintenance.
8. Sales, loyalty, point of sale, customer relationship management, payment providers, website and storefront.
9. Human resources core, attendances, time off, work entries, recruitment, expenses, fleet, lunch, projects and tasks, timesheets.
10. Calendar, events, marketing, learning and surveys, spreadsheets and dashboards, automation and integration.

Each step has an acceptance gate in `docs/reimplementation/milestones.md` and an equivalence test plan in `docs/reimplementation/equivalence-test-plan.md`.

## Coverage and evidence

`schemas/traceability/coverage.json` and `docs/reimplementation/coverage-and-evidence.md` record, for every entity, route, report, scheduled job and access rule, which document and which catalog entry describe it. The counts of specified artifacts are maintained there.

## Documentation rules

1. No product names, vendor names, source file paths, programming languages, framework names or code of any kind appear in the specification.
2. No acronyms or abbreviations appear in prose, headings or labels; reproduced identifiers (storage names, transport names, routes, selection values) are the only exception and are always accompanied by full names.
3. Only the current behavior of the system is described; there is no deprecated, historical or upgrade content, and no deployment or hosting guidance.
4. Every formula states its rounding rule, precision and evaluation order, and has a worked numeric example.
5. Every state machine lists all states, transitions, guards and side effects.
6. Every validation lists its condition and its exact user-facing message.
7. Every event that produces journal entries lists each journal item with account selection, side, amount formula, currency handling and reconciliation behavior.
8. Every domain has acceptance criteria in Given / When / Then form with concrete numbers.
9. Where the behavior is left implicit, the industry-standard behavior is stated explicitly and marked as "industry-standard default".
10. Cross-references are relative links inside this repository; nothing references material outside it.
