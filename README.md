# Enterprise Resource Planning: Language-Neutral Rebuild Specification

This repository is a complete, self-contained, language-neutral specification for rebuilding a mature, integrated enterprise resource planning system. It maps the observable application architecture, business capabilities, persistent data model, service contracts, operational workflows, cross-domain accounting effects and acceptance criteria of the reference system, without describing any implementation language, framework or deployment technology.

The intended outcome is **behavioral equivalence**: a replacement written in Go, Rust, PHP, C++, Java or any other suitable language must make the same business decisions, preserve the same records and financial consequences, enforce the same state transitions, exposures and validations, and offer the same operational capabilities.

The repository is licensed under the Apache License, Version 2.0 (see `LICENSE`).

## How to use this repository

1. Read this file end to end. It fixes the naming conventions, gives the architecture at a glance, maps the business capabilities, and lists the reading paths.
2. Read `docs/overview/` to understand the platform: the persistence layer, the model and field system, inheritance and extension, security, views and actions, the module system, the request and transport model, reporting, messaging and scheduled execution.
3. Read `docs/data/` for the domain model, identity and value rules, the physical data catalog and reference data.
4. Read the domain folders under `docs/domains/` in the order given by the reimplementation sequence below. Every domain folder contains the same set of files (entities, workflows, business rules, calculations, accounting effects, configuration, interfaces, acceptance criteria, glossary).
5. Use `schemas/` as the machine-readable counterpart: every entity, field, state, constraint, operation, endpoint, report, scheduled job, access rule, sequence, reference data record and acceptance scenario is cataloged there in structured text notation for tooling and code generation.
6. Use `docs/reimplementation/` for the build sequence, the milestone acceptance gates, the equivalence test plan and the traceability rules.

## Canonical naming conventions

The specification uses full words everywhere. No acronyms or abbreviations appear in headings, names, identifiers or prose. The following conventions are binding across `docs/` and `schemas/`:

| Convention | Rule |
|---|---|
| Entity names | Full business name in title case in prose (for example Journal Entry, Journal Item, Transfer, Stock Move, Stock Quantity Record, Reordering Rule, Sales Order Line, Employee Version). The canonical identifier is the same name in lower snake case (`journal_entry`, `stock_quantity_record`). The full table is in `docs/references/canonical-entity-names.md` and `schemas/traceability/entity-name-dictionary.json`. |
| Field names | Lower snake case, full words, no relational suffixes: `partner`, `unit_of_measure`, `demand_quantity`, `invoice_lines`, `tax_identification_number`, `can_be_sold`. Relations to many records are plural. |
| State values | Lower snake case as stored (`draft`, `posted`, `cancelled`, `assigned`, `done`), each with its full label. |
| Operations | Descriptive verb phrases in prose ("post the journal entry") and full-word snake case identifiers in tables (`post_journal_entry`, `validate_transfer`, `register_payment`). |
| Capability packages | The reference system is delivered as installable feature packages. They are named by full name (for example "Inventory", "Belgium Localization", "Point of Sale Restaurant"); the table is in `docs/references/capability-packages.md`. |
| Data types | `boolean`, `integer`, `decimal` (with precision rule), `monetary`, `text`, `long_text`, `rich_text`, `date`, `datetime` (coordinated universal time), `binary`, `image`, `selection`, `reference`, `many_to_one`, `one_to_many`, `many_to_many`, `structured_data`, `properties`. |
| Shared fields | Every persistent entity carries `identifier`, `created_on`, `created_by_user`, `last_updated_on`, `last_updated_by_user`; archivable entities carry `active`. |
| Expansions | unit of measure, bill of materials, point of sale, customer relationship management, human resources, text message, manufacturing order, purchase order, sales order, request for quotation, identifier, web address, rich text, portable document, comma-separated values, first in first out, average cost, cost of goods sold, value-added tax, international bank account number, electronic data interchange, universal business language, application programming interface, remote procedure call, quick response code, make to order, key performance indicator. |

Product names of third-party services (payment providers, shipping carriers, document exchange networks) are proper nouns and are described by their function on first use.

## Architecture at a glance

The system is a single modular application built on one relational database per tenant. Its layers, from the bottom up:

1. **Persistence and model layer.** Every business object is an *entity* (a model) with typed fields persisted in one table per entity, plus association tables for many-to-many relations. Entities support classical inheritance (extending an existing entity in place, adding fields and overriding operations), prototype inheritance (copying an entity's definition into a new one) and delegation inheritance (embedding a parent record so the child transparently exposes the parent's fields). Fields can be stored, computed (with declared dependencies and automatic recomputation), related (following a path through relations), company-dependent, translatable, or tracked for audit. The layer provides record-set operations (create, read, update, delete, search, read-group, name-search, copy), an in-memory cache with invalidation, and a unit-of-work that flushes pending writes inside a database transaction with serializable isolation and automatic retry on conflicts.
2. **Business rules.** Constraints (database-level uniqueness and checks; entity-level validation rules with messages), on-change behaviors in forms, default values, sequences for document numbering, state machines on documents, scheduled actions, automation rules and server actions.
3. **Security.** Users, access groups arranged in privilege categories with implied-group closure, per-entity access rights (create, read, update, delete) per group, record rules (row-level filters per group and per operation), field-level group restrictions, multi-company scoping, and portal (external) versus internal users.
4. **Presentation contracts.** Views (form, list, kanban, search, calendar, pivot, graph, activity, hierarchy, gantt) are declarative documents describing which fields, buttons, filters and groupings a client renders; window actions open views on an entity with a domain filter and context; menus organize actions; client actions and web address actions cover non-entity screens. The client talks to the server through a remote procedure call transport over structured text documents, with a fixed set of generic entity operations plus named business operations.
5. **Business domains.** Accounting (general ledger, receivables, payables, taxes, payments and bank reconciliation, multi-currency, analytic, electronic invoicing, fiscal localizations), supply chain (products, units of measure and packaging, inventory, valuation, replenishment, purchasing, manufacturing, repair and maintenance), sales (pricing, sales, loyalty, point of sale, customer relationship management, payment providers), human resources (core, time off, attendances, recruitment, expenses, fleet, engagement), services (projects, timesheets), communication (messaging and activities, calendar), marketing (events, marketing, surveys and courses), and web (website, storefront, portal), plus spreadsheets and dashboards and automation and integration services.
6. **Operational runtime.** Request lifecycle, sessions and authentication, transactions and concurrency, caching, scheduled job execution, the notification bus, attachments and the file store, report rendering (printable documents from templates), email sending and receiving, and background workers.

## Business capability map

| Group | Domain folder | Capabilities delivered |
|---|---|---|
| Framework | `docs/domains/platform-foundation` | Model registry, field definitions, access rules, views, actions, menus, sequences, attachments, scheduled actions, system parameters, web assets, request routing |
| Framework | `docs/domains/identity-and-access` | Users, groups, privileges, record rules, password policies, two-factor authentication, passkeys, open authorization sign-in, directory servers, sign-up, session timeout, application keys |
| Shared | `docs/domains/contacts-and-organizations` | Contacts, companies, addresses, bank accounts, banks, currencies, countries, languages, industries, tags, contact enrichment, postal mail |
| Accounting | `docs/domains/general-ledger` | Chart of accounts, journals, journal entries and items, posting, numbering, reversal, lock dates, inalterability, reconciliation core |
| Accounting | `docs/domains/accounts-receivable` | Customer invoices, credit notes, receipts, payment terms, early payment discounts, cash rounding, sending, portal payment |
| Accounting | `docs/domains/accounts-payable` | Vendor bills, refunds, debit notes, bill import, automatic posting, check printing |
| Accounting | `docs/domains/taxes` | Tax engine, distribution lines, tax groups, grids and reports, cash basis, fiscal positions |
| Accounting | `docs/domains/payments-and-bank-reconciliation` | Payments, payment methods, bank statements, reconciliation models, partial and full reconciliation, structured references |
| Accounting | `docs/domains/multi-currency` | Currencies, rates, conversion, foreign currency journal items, exchange differences |
| Accounting | `docs/domains/analytic-accounting` | Analytic plans, accounts, distributions, lines, applicability |
| Accounting | `docs/domains/electronic-invoicing-and-interchange` | Electronic document formats, import and export, document exchange network, certificates |
| Accounting | `docs/domains/fiscal-localizations` | Country packages: chart templates, taxes, fiscal positions, reports, country rules |
| Supply chain | `docs/domains/products-and-catalog` | Products, variants, attributes, categories, barcodes, expiry, matrices, combos |
| Supply chain | `docs/domains/units-of-measure-and-packaging` | Units, conversion, packagings, package types, quantity matrices |
| Sales | `docs/domains/pricing-and-pricelists` | Pricelists, rules, discount policies, vendor prices |
| Supply chain | `docs/domains/inventory-operations` | Warehouses, locations, operation types, transfers, moves, quantities, lots, packages, batches, adjustments, shipping |
| Supply chain | `docs/domains/inventory-valuation-and-costing` | Valuation layers, costing methods, automated entries, landed costs, cost of goods sold |
| Supply chain | `docs/domains/replenishment-and-procurement` | Routes, rules, reordering rules, scheduler, lead times, drop shipping |
| Supply chain | `docs/domains/purchasing` | Requests for quotation, purchase orders, receipts, bill control, agreements |
| Sales | `docs/domains/sales` | Quotations, orders, invoicing policies, down payments, templates, margins, teams |
| Sales | `docs/domains/loyalty-and-promotions` | Programs, rules, rewards, coupons, gift cards, wallets |
| Supply chain | `docs/domains/manufacturing` | Bills of materials, manufacturing orders, work orders, work centers, subcontracting, costing |
| Supply chain | `docs/domains/repair-and-maintenance` | Repair orders, equipment, maintenance requests |
| Sales | `docs/domains/point-of-sale` | Configurations, sessions, orders, payments, restaurant, self-ordering, terminals |
| Sales | `docs/domains/customer-relationship-management` | Leads, opportunities, stages, scoring, assignment, partnerships, campaign tracking |
| Sales | `docs/domains/payment-providers` | Providers, methods, tokens, transactions |
| Human resources | `docs/domains/human-resources-core` | Employees, versions and contracts, departments, jobs, skills, work entries |
| Human resources | `docs/domains/time-off` | Types, requests, allocations, accrual plans, public holidays |
| Human resources | `docs/domains/attendances-and-working-time` | Working schedules, attendances, overtime |
| Human resources | `docs/domains/recruitment` | Jobs, candidates, applicants, stages, job board |
| Human resources | `docs/domains/expenses` | Expenses, reports, approval, posting, re-invoicing |
| Human resources | `docs/domains/fleet` | Vehicles, contracts, services, costs |
| Human resources | `docs/domains/employee-services-and-engagement` | Lunch, gamification, digests |
| Services | `docs/domains/projects-and-tasks` | Projects, tasks, stages, milestones, updates, ratings |
| Services | `docs/domains/timesheets` | Timesheet lines, timers, billing, profitability |
| Communication | `docs/domains/messaging-and-collaboration` | Threads, messages, followers, activities, templates, aliases, channels, live chat, text messages |
| Communication | `docs/domains/calendar-and-scheduling` | Events, attendees, recurrences, reminders, external synchronization |
| Marketing | `docs/domains/events` | Events, tickets, registrations, booths, tracks, sponsors |
| Marketing | `docs/domains/marketing` | Mass mailings, lists, traces, link tracking, campaign tracking |
| Marketing | `docs/domains/surveys-and-elearning` | Surveys, courses, quizzes, certifications, forums |
| Web | `docs/domains/website-and-content-management` | Websites, pages, menus, themes, editor, forms, visitors, blogs |
| Web | `docs/domains/commerce-storefront` | Shop, cart, checkout, delivery options, wishlists, comparison |
| Web | `docs/domains/customer-portal` | External document access, sharing, ratings, signatures |
| Shared | `docs/domains/spreadsheets-and-dashboards` | Spreadsheets, dashboards, boards |
| Framework | `docs/domains/automation-and-integration-services` | Automation rules, data import, purchased services, external connectors, devices |

## Documentation index

- `docs/README.md`: navigation of the documentation tree.
- `docs/overview/`: platform architecture (persistence and fields, inheritance and extension, security model, views and actions, module system and data loading, transport and service contracts, reporting, messaging, scheduled execution, multi-company, translations, import and export).
- `docs/data/`: domain model overview, persistence identity and values, physical data catalog, reference data and migration boundaries.
- `docs/domains/`: one folder per business domain (see the capability map).
- `docs/interfaces/`: desktop workflows, endpoint catalogs, external integrations, remote transport contracts, report and export documentation, service layer.
- `docs/runtime/`: request lifecycle, sessions, transactions and concurrency, caching, scheduled jobs, notification bus, attachments, background workers.
- `docs/references/`: canonical entity names, capability packages, field type reference, selection value catalog, error message catalog, glossary.
- `docs/reimplementation/`: build sequence, milestones, acceptance gates, equivalence test plan.

## Machine-readable catalogs

- `schemas/data/`: entity catalogs (one structured document per entity: fields, relations, constraints, states, operations, derivations), the physical table catalog, association tables and reference data.
- `schemas/interfaces/`: endpoint catalog, service operation catalog, view contracts, actions and menus, reports and exports, client registry.
- `schemas/operational/`: scheduled actions, sequences, access groups, access rules, record rules, server actions, automation rules, email templates, activity types.
- `schemas/source-artifacts/`: capability package catalog with dependencies, categories and shipped data.
- `schemas/source-file-distribution/`: distribution of the reference implementation's artifacts by package and by kind, as evidence of coverage.
- `schemas/traceability/`: entity name dictionary, field name dictionary, domain ownership, coverage matrix and acceptance scenario catalog.

## Recommended reading paths

- **Architect**: this file, `docs/overview/`, `docs/data/`, `docs/runtime/`, then the domain README files.
- **Accounting implementer**: `docs/domains/general-ledger`, `taxes`, `accounts-receivable`, `accounts-payable`, `payments-and-bank-reconciliation`, `multi-currency`, `analytic-accounting`, then `inventory-valuation-and-costing`, `fiscal-localizations`.
- **Supply chain implementer**: `products-and-catalog`, `units-of-measure-and-packaging`, `inventory-operations`, `replenishment-and-procurement`, `purchasing`, `manufacturing`, `inventory-valuation-and-costing`.
- **Sales implementer**: `pricing-and-pricelists`, `sales`, `customer-relationship-management`, `loyalty-and-promotions`, `point-of-sale`, `payment-providers`, `commerce-storefront`.
- **Human resources implementer**: `human-resources-core`, `attendances-and-working-time`, `time-off`, `recruitment`, `expenses`, `timesheets`, `fleet`.
- **Integrator**: `docs/interfaces/`, `schemas/interfaces/`, `docs/domains/electronic-invoicing-and-interchange`, `payment-providers`, `automation-and-integration-services`.

## Reimplementation sequence

1. Platform foundation: persistence layer, fields, inheritance, transactions, security, sequences, attachments, scheduled actions, transport, views and actions.
2. Shared entities: contacts and organizations, identity and access, messaging and activities, units of measure, currencies.
3. Products and catalog, pricing.
4. General ledger, taxes, multi-currency, analytic accounting.
5. Accounts receivable, accounts payable, payments and bank reconciliation.
6. Inventory operations, valuation and costing, replenishment.
7. Purchasing, sales, manufacturing, repair and maintenance.
8. Point of sale, loyalty, customer relationship management, payment providers.
9. Human resources family, projects and timesheets.
10. Calendar, events, marketing, surveys and courses, website, storefront, portal, spreadsheets and dashboards, automation and integration services, electronic invoicing, fiscal localizations.

Each step has acceptance gates in `docs/reimplementation/`.

## Coverage and evidence

The specification was produced by systematic extraction of the reference implementation's model definitions, fields, constraints, operations, state fields, configuration records, access rules, endpoints, reports, scheduled actions and automated behavior scenarios, followed by domain-by-domain reading of the operational logic and its translation into language-neutral rules. `schemas/traceability/` records, for every entity and field, the domain that documents it and the evidence location, and `schemas/source-file-distribution/` records the size and kind of the artifacts covered.

## Documentation rules

1. Full words only; no acronyms or abbreviations.
2. No product, vendor or brand name of the reference system; no references to its source files.
3. No programming-language code; behavior is described in prose, tables, formulas, state tables and language-neutral notation.
4. Current behavior only; no legacy, migration or deprecation content.
5. No deployment or infrastructure instructions; runtime behavior is in scope, provisioning is not.
6. Every list is complete; nothing is abbreviated with "and so on".
7. Every numeric rule has a worked example; every state field has a transition table; every workflow has actors, preconditions, steps and postconditions.
8. Where the reference behavior leaves a gap, the specification completes it with established industry practice and says so explicitly.
