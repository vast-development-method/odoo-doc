# Domain model

The complete map of every entity the system stores, grouped by the domain that specifies it, with the role of each entity, the relations that link entities together, and the reference traffic between domains. This file is the index of the data model: it states what exists and how it is connected. The field-by-field specification of each entity lives in the domain folders under [`../domains/`](../domains/) and on the generated reference page of the entity under [`../references/entities/`](../references/entities/); the value and identity rules are in [`persistence-identity-and-values.md`](persistence-identity-and-values.md); the storage shapes are in [`physical-data-catalog.md`](physical-data-catalog.md); how records reach the tables is in [`data-loading-and-exchange.md`](data-loading-and-exchange.md); and the records a fresh installation must contain are in [`reference-data.md`](reference-data.md).

The model holds 983 entities: 600 persistent, 222 interactive assistants and 161 shared behaviours, linked by 3,985 relation fields.

## 1. How to read this map

### 1.1 Entity kinds

| Kind | Meaning | Storage |
|---|---|---|
| Persistent | A business record that is stored permanently and is visible to queries, access rules, reports and integrations. | One table, one row per record. |
| Interactive assistant | A short-lived record that holds the input of a multi-step operation: a dialogue that asks the user for values, runs an operation and disappears. Assistant records are removed by the periodic cleanup. | One table, rows deleted by the periodic cleanup. |
| Shared behaviour | A named bundle of fields, validations and operations merged into other entities — the discussion thread, scheduled activities, portal access, rating, image handling, address formatting and others. It never has records of its own. | No table. |

### 1.2 The name columns

Every entity is given three names, and all three are needed:

| Column | What it is |
|---|---|
| Entity | The full name in words. Where the label of an entity carries an abbreviation, a dotted technical name or a sentence fragment, this document writes the name in full words. |
| Transport name | The dotted name a client, an integration or an import file uses to address the entity. Reproduced exactly, because it is contractual. |
| Table | The name of the relation that stores the entity, which is the transport name with every dot replaced by an underscore. It reads `none` for a shared behaviour, which has no storage of its own, and `none, read from a stored query` for an entity whose rows are computed at read time. Where the relation is a stored query materialized as a view, the name given is the name of that view. |

The reference page of an entity is `../references/entities/<transport name>.md` and its machine-readable definition is `../../schemas/data/entities/<transport name>.json`.

### 1.3 The purpose line

The purpose column states the role of the entity. Where the entity carries a narrative description of its own, that description is used. Otherwise the line is composed from the structure of the entity: how many stored columns it has, the records it must belong to (its required links), the collections it owns, the states it carries, whether it is scoped to a company, and how many relation fields elsewhere point at it. A high reference count marks master data that many domains depend on.

### 1.4 Domain attribution

Every entity appears exactly once, under the domain folder that specifies it. Most entities are extended by capability packages belonging to other domains: those packages add fields, validations and operations to an entity they do not own, and those additions are specified in the folder of the package that adds them. When an entity carries fields from several domains this map still lists it once, and the cross-domain reference counts of section 4 show where the traffic comes from. The generic platform entities are attributed to the platform foundation and are specified in the overview and runtime documents rather than in a business folder.

### 1.5 Relation kinds and cardinality notation

| Relation kind | Meaning | Cardinality notation | Physical form |
|---|---|---|---|
| Link to one record | The source record names one target record. | `n : 1` when the link is required, `n : 0..1` when it is optional. | An integer column on the source table holding the target identifier. |
| List of records | The inverse view of a link to one record: the list of records that point back at this record. | `1 : 0..n` | No column of its own; it is the mirror of the target column named by the inverse field. |
| List on both sides | Both sides hold a list of the other. | `0..n : 0..n` | An association table with two identifier columns. |
| Polymorphic link | The source stores the transport name of the target entity in one column and the target identifier in another. | `n : 0..1, polymorphic` | Two columns and no foreign key, because the target entity varies per row. |
| Polymorphic reference | The source stores the transport name, a comma and the identifier in one text column. | `n : 0..1, polymorphic` | One text column and no foreign key. |

The deletion column states what happens when the target record is deleted: `restrict` refuses the deletion while a source record still points at the target, `cascade` deletes the source record with it, `set null` clears the link and keeps the source record. For a list on both sides, `cascade` removes the association row. For a list of records the behaviour is the one declared on the mirrored column of the target entity. "not declared" means the relation states no behaviour and the default of [`physical-data-catalog.md`](physical-data-catalog.md), section 9.2, applies.

The storage column separates relations that are materialized — a column or an association row that a replacement must create — from relations that are derived at read time, computed or followed through another relation, and therefore not stored.

## 2. Domain inventory

Folder gives the folder under [`../domains/`](../domains/) that specifies the domain; the platform foundation is specified in the platform documents instead. Every folder is listed in the domain index [`../domains/README.md`](../domains/README.md).

| Domain | Group | Folder | Persistent | Assistants | Shared behaviours | Relation fields | Scope |
|---|---|---|---|---|---|---|---|
| Automation and Integration | framework | [`../domains/automation-and-integration/`](../domains/automation-and-integration/) | 14 | 4 | 3 | 30 | Automation rules and their triggers, the data import assistant and its saved column mappings, in-application purchase services and accounts, connectors to external mail, calendar and storage accounts, telephone number validation, record recycling and privacy tools, and the remote-call test assistant. |
| Identity and Access | framework | [`../domains/identity-and-access/`](../domains/identity-and-access/) | 14 | 8 | 2 | 79 | Users, access groups and privileges, record rules, authentication methods including password change, two-factor devices, passkeys and application keys, sessions, devices and their logs, user settings and the deletion request queue. |
| Platform Foundation | framework | the platform documents | 50 | 19 | 58 | 237 | Core persistence, the entity and field registry, external identifiers, access control objects, actions, views and menus, sequences, attachments, scheduled jobs, configuration parameters, asset bundles, report layouts, paper formats, user-defined field definitions, the import and export machinery and the request-routing objects. |
| Calendar and Scheduling | shared | `../domains/calendar-and-scheduling/` | 6 | 4 | 7 | 29 | Calendar events, attendees, recurrence rules, reminders, providers and the synchronization with external calendars. |
| Contacts and Organizations | shared | [`../domains/contacts-and-organizations/`](../domains/contacts-and-organizations/) | 11 | 3 | 2 | 230 | Parties — individuals, organizations and bare addresses — companies, banks and bank accounts, countries, country subdivisions and country groups, cities, languages, partner tags, industries and the document layout. |
| Messaging and Activities | shared | [`../domains/messaging-and-activities/`](../domains/messaging-and-activities/) | 59 | 15 | 14 | 297 | Discussion threads, messages, followers and notifications, activities and activity plans, message templates, aliases and the incoming and outgoing mail gateways, channels and channel members, live chat, chatbots, text messages, postal mail, push notifications, link previews, reactions and ratings. |
| Multi-Currency | shared | [`../domains/multi-currency/`](../domains/multi-currency/) | 2 | 0 | 0 | 3 | Currencies and their dated rates, the rounding factor, currency conversion and the foreign-currency copy of every ledger amount. |
| Spreadsheets and Dashboards | shared | [`../domains/spreadsheets-and-dashboards/`](../domains/spreadsheets-and-dashboards/) | 3 | 0 | 2 | 8 | Spreadsheet documents, spreadsheet revisions, dashboards and dashboard groups. |
| Accounts Payable | accounting | [`../domains/accounts-payable/`](../domains/accounts-payable/) | 0 | 1 | 0 | 0 | Vendor bills, vendor credit notes, purchase receipts, debit notes, cheque printing, automatic bill posting and duplicate detection. |
| Accounts Receivable | accounting | [`../domains/accounts-receivable/`](../domains/accounts-receivable/) | 0 | 2 | 0 | 3 | Customer invoices, credit notes, customer receipts, debit notes raised on a customer document and the refund of a received payment. |
| Analytic Accounting | accounting | [`../domains/analytic-accounting/`](../domains/analytic-accounting/) | 5 | 0 | 2 | 47 | Analytic plans, analytic accounts, analytic distribution models, analytic lines and the applicability rules that decide when a distribution is required. |
| Electronic Invoicing and Document Exchange | accounting | [`../domains/electronic-invoicing-and-document-exchange/`](../domains/electronic-invoicing-and-document-exchange/) | 7 | 4 | 18 | 21 | Structured electronic document formats, document import and export, network delivery, digital certificates and the interchange proxy. |
| Financial Reporting | accounting | [`../domains/financial-reporting/`](../domains/financial-reporting/) | 1 | 0 | 0 | 0 | The definition and evaluation of financial statements built on the ledger. |
| Fiscal Localizations | accounting | `../domains/fiscal-localizations/` | 49 | 24 | 16 | 101 | Country packages: chart of account templates, tax definitions, fiscal positions, tax report structures, country-specific document rules, identification types and country-specific electronic invoicing. |
| General Ledger | accounting | [`../domains/general-ledger/`](../domains/general-ledger/) | 34 | 14 | 4 | 425 | The chart of accounts, account groups and tags, journals and journal groups, journal entries and journal items, posting, numbering, reversal, lock dates, the inalterability hash chain, the reconciliation core, financial report structures, payment terms and the fiscal year. |
| Payments and Bank Reconciliation | accounting | [`../domains/payments-and-bank-reconciliation/`](../domains/payments-and-bank-reconciliation/) | 1 | 2 | 0 | 11 | Payments, payment methods and method lines, outstanding accounts, bank statements and statement lines, reconciliation models and their lines, partial and full reconciliation. |
| Taxes | accounting | [`../domains/taxes/`](../domains/taxes/) | 1 | 2 | 1 | 9 | The tax engine: computation types, price inclusion, distribution lines, tax groups, tax grids and tags, cash basis, withholding, fiscal positions and their mappings. |
| Delivery and Shipping | supply chain | [`../domains/delivery-and-shipping/`](../domains/delivery-and-shipping/) | 3 | 1 | 0 | 18 | Delivery methods, carriers, price rules, shipping labels and tracking. |
| Inventory Operations | supply chain | [`../domains/inventory-operations/`](../domains/inventory-operations/) | 22 | 24 | 4 | 383 | Warehouses, locations, operation types, transfers, stock moves and move lines, reservations, quantities on hand, lots and serial numbers, packages and package types, batch transfers, putaway and removal strategies, scrap and inventory adjustments. |
| Inventory Valuation and Costing | supply chain | [`../domains/inventory-valuation-and-costing/`](../domains/inventory-valuation-and-costing/) | 4 | 0 | 2 | 28 | Valuation layers, costing methods, automated and manual valuation, stock accounts, cost of goods sold, landed costs, price difference handling and revaluation. |
| Manufacturing | supply chain | [`../domains/manufacturing/`](../domains/manufacturing/) | 14 | 11 | 0 | 148 | Bills of materials, manufacturing orders, work orders, work centres, operations, productivity records, by-products, unbuild orders and subcontracting. |
| Products and Catalog | supply chain | [`../domains/products-and-catalog/`](../domains/products-and-catalog/) | 19 | 3 | 2 | 156 | Product templates and variants, attributes and values, categories, tags, documents, barcodes and nomenclatures, expiry and combos. |
| Purchasing | supply chain | [`../domains/purchasing/`](../domains/purchasing/) | 8 | 3 | 0 | 88 | Requests for quotation, purchase orders and lines, vendor prices, receipts, bill control policies, three-way matching and purchase agreements. |
| Repair and Maintenance | supply chain | [`../domains/repair-and-maintenance/`](../domains/repair-and-maintenance/) | 7 | 1 | 1 | 52 | Repair orders and their lines, equipment, maintenance requests, teams and stages, and preventive maintenance scheduling. |
| Replenishment and Procurement | supply chain | [`../domains/replenishment-and-procurement/`](../domains/replenishment-and-procurement/) | 1 | 0 | 0 | 3 | Routes and rules, reordering rules, make-to-order, lead times, the scheduler, forecasted quantities and drop shipping. |
| Units of Measure and Packaging | supply chain | [`../domains/units-of-measure-and-packaging/`](../domains/units-of-measure-and-packaging/) | 1 | 0 | 0 | 6 | Unit of measure categories, conversion factors and rounding, product packagings and package types with dimensions and weight. |
| Customer Relationship Management | sales | [`../domains/customer-relationship-management/`](../domains/customer-relationship-management/) | 25 | 8 | 3 | 110 | Leads and opportunities, pipeline stages, sales teams and members, predictive lead scoring, lead assignment and enrichment, lost reasons, merging and partnerships. |
| Loyalty and Promotions | sales | [`../domains/loyalty-and-promotions/`](../domains/loyalty-and-promotions/) | 7 | 5 | 0 | 52 | Loyalty programs, rules, rewards, cards, coupons, gift cards and electronic wallets. |
| Payment Providers | sales | `../domains/payment-providers/` | 4 | 2 | 0 | 41 | Payment providers, payment methods, tokens and transactions with their state machine, capture and refund flows. |
| Point of Sale | sales | [`../domains/point-of-sale/`](../domains/point-of-sale/) | 17 | 6 | 2 | 162 | Point of sale configurations, sessions, orders and order lines, payments and payment methods, cash control, receipts, restaurant floors and tables, presets, self-ordering and payment terminals. |
| Pricing and Pricelists | sales | [`../domains/pricing-and-pricelists/`](../domains/pricing-and-pricelists/) | 0 | 1 | 0 | 0 | Pricelists, pricelist rules, computation bases, minimum quantities, date validity and discount policies. |
| Sales | sales | [`../domains/sales/`](../domains/sales/) | 10 | 4 | 0 | 146 | Quotations, sales orders and order lines, quotation templates, invoicing policies, down payments, margins and quotation documents. |
| Website and Storefront | sales | [`../domains/website-and-storefront/`](../domains/website-and-storefront/) | 37 | 7 | 12 | 131 | Websites, pages and menus, themes, content blocks, the editor, forms, visitors and tracking, blogs, search-engine metadata, rewrites, the online shop, the cart and checkout, wishlists and comparison. |
| Projects and Tasks | services | [`../domains/projects-and-tasks/`](../domains/projects-and-tasks/) | 12 | 8 | 3 | 98 | Projects, tasks, stages, milestones, recurrences, collaborators, project updates, profitability, tags and to-do items. |
| Timesheets | services | [`../domains/timesheets/`](../domains/timesheets/) | 3 | 1 | 0 | 24 | Timesheet lines on tasks and projects, employee hourly cost, timesheet billing to customers and the comparison with attendances. |
| Attendances and Working Time | human resources | [`../domains/attendances-and-working-time/`](../domains/attendances-and-working-time/) | 9 | 0 | 1 | 36 | Working schedules and their attendance lines, resource calendars and leaves, resources, check-in and check-out records, overtime rules and rulesets. |
| Expenses | human resources | [`../domains/expenses/`](../domains/expenses/) | 1 | 5 | 0 | 37 | Employee expenses, expense reports, approval, reimbursement or company payment and re-invoicing to customers. |
| Fleet | human resources | `../domains/fleet/` | 13 | 1 | 0 | 52 | Vehicles, models, brands and categories, contracts, services, odometer readings, assignment logs, states, tags and cost reporting. |
| Human Resources Core | human resources | [`../domains/human-resources-core/`](../domains/human-resources-core/) | 22 | 6 | 2 | 165 | Employees and employee versions, departments, job positions, work locations, skills and resumes, the organization chart, presence, remote work, departure reasons and hourly cost. |
| Lunch Ordering | human resources | `../domains/lunch-ordering/` | 9 | 0 | 0 | 35 | Meal suppliers, products and categories, locations, orders, cash movements and alerts. |
| Recruitment | human resources | [`../domains/recruitment/`](../domains/recruitment/) | 8 | 4 | 0 | 47 | Job openings, candidates, applicants, recruitment stages, sources, degrees, refuse reasons, interviews and the job board. |
| Time Off | human resources | [`../domains/time-off/`](../domains/time-off/) | 11 | 4 | 0 | 70 | Time off types, requests, allocations, accrual plans and levels, approval flows, public holidays and mandatory days. |
| Work Entries | human resources | [`../domains/work-entries/`](../domains/work-entries/) | 3 | 1 | 0 | 13 | Work entry types, generated work entries and their conflicts. |
| Events | marketing | [`../domains/events/`](../domains/events/) | 33 | 4 | 0 | 159 | Events and event types, tickets, registrations and answers, booths and booth categories, tracks and track stages, sponsors, tags, stages and event communications. |
| Learning, Surveys and Gamification | marketing | `../domains/learning-surveys-and-gamification/` | 25 | 4 | 0 | 143 | Surveys, questions and answers, participations and scoring, courses, slides and content, quizzes, certifications, forums and posts, badges, challenges, goals and karma. |
| Marketing and Mass Mailing | marketing | `../domains/marketing-and-mass-mailing/` | 15 | 6 | 0 | 52 | Mass mailings, mailing lists, contacts and subscriptions, traces and trace statistics, link tracking, campaign tracking, marketing cards and social links. |
| **Total** | | | **600** | **222** | **161** | **3,985** | |

## 3. Entity map by domain

### 3.1 Automation and Integration

Automation rules and their triggers, the data import assistant and its saved column mappings, in-application purchase services and accounts, connectors to external mail, calendar and storage accounts, telephone number validation, record recycling and privacy tools, and the remote-call test assistant.

Specified in [`../domains/automation-and-integration/`](../domains/automation-and-integration/).

#### Persistent entities (14)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Base Import Mapping | `base_import.mapping` | `base_import_mapping` | Persistent record with 3 stored columns. | Base import |
| Cloud Storage Migration Report | `cloud.storage.migration.report` | `none, read from a stored query` | Persistent record with 0 stored columns. | Cloud Storage Migration |
| Code Translation | `transifex.code.translation` | `transifex_code_translation` | Persistent record with 4 stored columns. | Transifex integration |
| in-app purchase Account | `iap.account` | `iap_account` | In Application Purchase Account. | In-App Purchases |
| in-app purchase Service | `iap.service` | `iap_service` | In Application Purchase Service. | In-App Purchases |
| Onboarding | `onboarding.onboarding` | `onboarding_onboarding` | Persistent record with 5 stored columns; owns Onboarding Progress Tracker; states of `current_onboarding_state`: x; referenced by 2 relation fields. | Onboarding Toolbox |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `onboarding_progress_step` | Persistent record with 3 stored columns; belongs to Onboarding Step; states of `step_state`: x; company scoped; referenced by 2 relation fields. | Onboarding Toolbox |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_progress` | Persistent record with 4 stored columns; belongs to Onboarding; states of `onboarding_state`: x; company scoped; referenced by 2 relation fields. | Onboarding Toolbox |
| Onboarding Step | `onboarding.onboarding.step` | `onboarding_onboarding_step` | Persistent record with 10 stored columns; owns Onboarding Progress Step Tracker; states of `current_step_state`: x; referenced by 2 relation fields. | Onboarding Toolbox |
| Privacy Log | `privacy.log` | `privacy_log` | Persistent record with 7 stored columns; belongs to User; referenced by 1 relation field. | Privacy |
| Recycling Model | `data_recycle.model` | `data_recycle_model` | Persistent record with 14 stored columns; belongs to Models; owns Recycling Record; referenced by 1 relation field. | Data Recycle |
| Recycling Record | `data_recycle.record` | `data_recycle_record` | Persistent record with 6 stored columns; company scoped. | Data Recycle |
| Tour's step | `web_tour.tour.step` | `web_tour_tour_step` | Persistent record with 6 stored columns; belongs to Tours. | Tours |
| Tours | `web_tour.tour` | `web_tour_tour` | Persistent record with 5 stored columns; owns Tour's step; referenced by 1 relation field. | Tours |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Base Import | `base_import.import` | `base_import_import` | Interactive assistant with 4 stored columns. | Base import |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `privacy_lookup_wizard` | Interactive assistant with 4 stored columns; owns Privacy Lookup Wizard Line; referenced by 1 relation field. | Privacy |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `privacy_lookup_wizard_line` | Interactive assistant with 9 stored columns. | Privacy |
| Sparse fields Test | `sparse_fields.test` | `sparse_fields_test` | Interactive assistant with 1 stored column. | Sparse Fields |

#### Shared behaviour entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| in-app purchase Lead Enrichment application programming interface | `iap.enrich.api` | `none` | In Application Purchase Lead Enrichment Application Programming Interface. | In-App Purchases |
| in-app purchase Partner Autocomplete application programming interface | `iap.autocomplete.api` | `none` | In Application Purchase Partner Autocomplete Application Programming Interface. | Partner Autocomplete |
| Transifex Translation | `transifex.translation` | `none` | Shared behaviour definition reused through composition. | Transifex integration |

#### Relationships (30)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Onboarding | `onboarding.onboarding` | `current_progress_id` | link to one record | Onboarding Progress Tracker | `onboarding.progress` | `n : 0..1` | `not declared` | derived |
| Onboarding | `onboarding.onboarding` | `progress_ids` | list of records | Onboarding Progress Tracker | `onboarding.progress` | `1 : 0..n` | `mirror of the target column` | derived |
| Onboarding | `onboarding.onboarding` | `step_ids` | list on both sides | Onboarding Step | `onboarding.onboarding.step` | `0..n : 0..n` | `not declared` | stored |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `cascade` | stored |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `progress_ids` | list on both sides | Onboarding Progress Tracker | `onboarding.progress` | `0..n : 0..n` | `not declared` | stored |
| Onboarding Progress Step Tracker | `onboarding.progress.step` | `step_id` | link to one record | Onboarding Step | `onboarding.onboarding.step` | `n : 1` | `cascade` | stored |
| Onboarding Progress Tracker | `onboarding.progress` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `cascade` | stored |
| Onboarding Progress Tracker | `onboarding.progress` | `onboarding_id` | link to one record | Onboarding | `onboarding.onboarding` | `n : 1` | `cascade` | stored |
| Onboarding Progress Tracker | `onboarding.progress` | `progress_step_ids` | list on both sides | Onboarding Progress Step Tracker | `onboarding.progress.step` | `0..n : 0..n` | `not declared` | stored |
| Onboarding Step | `onboarding.onboarding.step` | `current_progress_step_id` | link to one record | Onboarding Progress Step Tracker | `onboarding.progress.step` | `n : 0..1` | `not declared` | derived |
| Onboarding Step | `onboarding.onboarding.step` | `onboarding_ids` | list on both sides | Onboarding | `onboarding.onboarding` | `0..n : 0..n` | `not declared` | stored |
| Onboarding Step | `onboarding.onboarding.step` | `progress_ids` | list of records | Onboarding Progress Step Tracker | `onboarding.progress.step` | `1 : 0..n` | `mirror of the target column` | derived |
| Privacy Log | `privacy.log` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `line_ids` | list of records | Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Privacy Lookup Wizard | `privacy.lookup.wizard` | `log_id` | link to one record | Privacy Log | `privacy.log` | `n : 0..1` | `not declared` | stored |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Privacy Lookup Wizard Line | `privacy.lookup.wizard.line` | `wizard_id` | link to one record | Privacy Lookup Wizard | `privacy.lookup.wizard` | `n : 0..1` | `not declared` | stored |
| Recycling Model | `data_recycle.model` | `notify_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Recycling Model | `data_recycle.model` | `recycle_record_ids` | list of records | Recycling Record | `data_recycle.record` | `1 : 0..n` | `mirror of the target column` | derived |
| Recycling Model | `data_recycle.model` | `res_model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Recycling Model | `data_recycle.model` | `time_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Recycling Record | `data_recycle.record` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Recycling Record | `data_recycle.record` | `recycle_model_id` | link to one record | Recycling Model | `data_recycle.model` | `n : 0..1` | `cascade` | stored |
| Sparse fields Test | `sparse_fields.test` | `partner` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Tour's step | `web_tour.tour.step` | `tour_id` | link to one record | Tours | `web_tour.tour` | `n : 1` | `cascade` | stored |
| Tours | `web_tour.tour` | `step_ids` | list of records | Tour's step | `web_tour.tour.step` | `1 : 0..n` | `mirror of the target column` | derived |
| Tours | `web_tour.tour` | `user_consumed_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| in-app purchase Account | `iap.account` | `company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | stored |
| in-app purchase Account | `iap.account` | `service_id` | link to one record | in-app purchase Service | `iap.service` | `n : 1` | `not declared` | stored |
| in-app purchase Account | `iap.account` | `warning_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |

### 3.2 Identity and Access

Users, access groups and privileges, record rules, authentication methods including password change, two-factor devices, passkeys and application keys, sessions, devices and their logs, user settings and the deletion request queue.

Specified in [`../domains/identity-and-access/`](../domains/identity-and-access/).

#### Persistent entities (14)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Access Groups | `res.groups` | `res_groups` | Persistent record with 10 stored columns; owns Model Access; referenced by 28 relation fields. | Base |
| Authentication Device | `auth_totp.device` | `auth_totp_device` | Persistent record with 6 stored columns. | Two-Factor Authentication (time-based one-time password) |
| Company directory access protocol configuration | `res.company.ldap` | `none in the observed installation` | Persistent record with 0 stored columns; belongs to Companies. | Authentication via directory access protocol |
| Device Log | `res.device.log` | `res_device_log` | Persistent record with 11 stored columns. | Base |
| Devices | `res.device` | `res_device` | Persistent record with 11 stored columns. | Base |
| OAuth2 provider | `auth.oauth.provider` | `auth_oauth_provider` | Persistent record with 10 stored columns; referenced by 1 relation field. | OAuth2 Authentication |
| Passkey | `auth.passkey.key` | `auth_passkey_key` | Persistent record with 4 stored columns. | Passkeys |
| Privileges | `res.groups.privilege` | `res_groups_privilege` | Persistent record with 5 stored columns; owns Access Groups; referenced by 1 relation field. | Base |
| User | `res.users` | `res_users` | Persistent record with 31 stored columns; belongs to Companies, Contact; owns Authentication Device, Devices, Employee and 10 further collections; lifecycle states Invited, Confirmed; 3 state fields in all; company scoped; referenced by 188 relation fields. | Base |
| User Settings | `res.users.settings` | `res_users_settings` | Persistent record with 18 stored columns; belongs to User; owns User Settings Volumes, User Settings for Embedded Actions; referenced by 3 relation fields. | Base |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `res_users_settings_embedded_action` | Persistent record with 7 stored columns; belongs to Action Window, User Settings. | Web |
| Users application programming interface Keys | `res.users.apikeys` | `res_users_apikeys` | Persistent record with 6 stored columns; belongs to User. | Base |
| Users Deletion Request | `res.users.deletion` | `res_users_deletion` | Persistent record with 3 stored columns; lifecycle states To Do, Done, Failed. | Base |
| Users Log | `res.users.log` | `res_users_log` | Persistent record with 1 stored column. | Base |

#### Interactive assistant entities (8)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| 2-Factor Setup Wizard | `auth_totp.wizard` | `auth_totp_wizard` | Interactive assistant with 4 stored columns; belongs to User. | Two-Factor Authentication (time-based one-time password) |
| application programming interface Key Description | `res.users.apikeys.description` | `res_users_apikeys_description` | Interactive assistant with 3 stored columns. | Base |
| Change Password Wizard | `change.password.wizard` | `change_password_wizard` | Interactive assistant with 0 stored columns; owns User, Change Password Wizard; referenced by 1 relation field. | Base |
| Create a Passkey | `auth.passkey.key.create` | `auth_passkey_key_create` | Interactive assistant with 1 stored column. | Passkeys |
| Password Check Wizard | `res.users.identitycheck` | `res_users_identitycheck` | Interactive assistant with 2 stored columns. | Base |
| time-based one-time password rate limit logs | `auth.totp.rate.limit.log` | `auth_totp_rate_limit_log` | Interactive assistant with 3 stored columns; belongs to User. | Two-Factor Authentication (time-based one-time password) |
| User, change own password wizard | `change.password.own` | `change_password_own` | Interactive assistant with 2 stored columns. | Base |
| User, Change Password Wizard | `change.password.user` | `change_password_user` | Interactive assistant with 4 stored columns; belongs to Change Password Wizard, User. | Base |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| key performance indicator Provider | `kpi.provider` | `none` | Shared behaviour definition reused through composition. | Initial Setup Tools |
| Show application programming interface Key | `res.users.apikeys.show` | `none` | Shared behaviour definition reused through composition. | Base |

#### Relationships (79)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| 2-Factor Setup Wizard | `auth_totp.wizard` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Access Groups | `res.groups` | `all_implied_by_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | derived |
| Access Groups | `res.groups` | `all_implied_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | derived |
| Access Groups | `res.groups` | `all_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Access Groups | `res.groups` | `disjoint_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | derived |
| Access Groups | `res.groups` | `implied_by_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Access Groups | `res.groups` | `implied_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Access Groups | `res.groups` | `menu_access` | list on both sides | Menu | `ir.ui.menu` | `0..n : 0..n` | `not declared` | stored |
| Access Groups | `res.groups` | `model_access` | list of records | Model Access | `ir.model.access` | `1 : 0..n` | `mirror of the target column` | derived |
| Access Groups | `res.groups` | `privilege_id` | link to one record | Privileges | `res.groups.privilege` | `n : 0..1` | `not declared` | stored |
| Access Groups | `res.groups` | `rule_groups` | list on both sides | Record Rule | `ir.rule` | `0..n : 0..n` | `not declared` | stored |
| Access Groups | `res.groups` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Access Groups | `res.groups` | `view_access` | list on both sides | View | `ir.ui.view` | `0..n : 0..n` | `not declared` | stored |
| Change Password Wizard | `change.password.wizard` | `user_ids` | list of records | User, Change Password Wizard | `change.password.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Company directory access protocol configuration | `res.company.ldap` | `company` | link to one record | Companies | `res.company` | `n : 1` | `cascade` | derived |
| Company directory access protocol configuration | `res.company.ldap` | `user` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Device Log | `res.device.log` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Passkey | `auth.passkey.key` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Privileges | `res.groups.privilege` | `category_id` | link to one record | Application | `ir.module.category` | `n : 0..1` | `not declared` | stored |
| Privileges | `res.groups.privilege` | `group_ids` | list of records | Access Groups | `res.groups` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `action_id` | link to one record | Actions | `ir.actions.actions` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `all_group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | derived |
| User | `res.users` | `api_key_ids` | list of records | Users application programming interface Keys | `res.users.apikeys` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `auth_passkey_key_ids` | list of records | Passkey | `auth.passkey.key` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `badge_ids` | list of records | Gamification User Badge | `gamification.badge.user` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| User | `res.users` | `company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `create_employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `crm_team_ids` | list on both sides | Sales Team | `crm.team` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `crm_team_member_ids` | list of records | Sales Team Member | `crm.team.member` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `device_ids` | list of records | Devices | `res.device` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `employee_bank_account_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | derived |
| User | `res.users` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `employee_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `favorite_lunch_product_ids` | list on both sides | Lunch Product | `lunch.product` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `favorite_project_ids` | list on both sides | Project | `project.project` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `friday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `goal_ids` | list of records | Gamification Goal | `gamification.goal` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `karma_tracking_ids` | list of records | Track Karma Changes | `gamification.karma.tracking` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `last_lunch_location_id` | link to one record | Lunch Locations | `lunch.location` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `livechat_channel_ids` | list on both sides | Livechat Channel | `im_livechat.channel` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `livechat_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | derived |
| User | `res.users` | `livechat_lang_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | derived |
| User | `res.users` | `log_ids` | list of records | Users Log | `res.users.log` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `monday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `next_rank_id` | link to one record | Rank based on karma | `gamification.karma.rank` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `oauth_provider_id` | link to one record | OAuth2 provider | `auth.oauth.provider` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `outgoing_mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `restrict` | stored |
| User | `res.users` | `presence_ids` | list of records | User/Guest Presence | `mail.presence` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `property_warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `rank_id` | link to one record | Rank based on karma | `gamification.karma.rank` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `res_users_settings_id` | link to one record | User Settings | `res.users.settings` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `res_users_settings_ids` | list of records | User Settings | `res.users.settings` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `resource_ids` | list of records | Resources | `resource.resource` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `role_ids` | list on both sides | Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `0..n : 0..n` | `not declared` | stored |
| User | `res.users` | `sale_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `saturday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `sunday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `thursday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `totp_trusted_device_ids` | list of records | Authentication Device | `auth_totp.device` | `1 : 0..n` | `mirror of the target column` | derived |
| User | `res.users` | `tuesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User | `res.users` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| User | `res.users` | `wednesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| User Settings | `res.users.settings` | `embedded_actions_config_ids` | list of records | User Settings for Embedded Actions | `res.users.settings.embedded.action` | `1 : 0..n` | `mirror of the target column` | derived |
| User Settings | `res.users.settings` | `livechat_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | stored |
| User Settings | `res.users.settings` | `livechat_lang_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | stored |
| User Settings | `res.users.settings` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| User Settings | `res.users.settings` | `volume_settings_ids` | list of records | User Settings Volumes | `res.users.settings.volumes` | `1 : 0..n` | `mirror of the target column` | derived |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `action_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 1` | `cascade` | stored |
| User Settings for Embedded Actions | `res.users.settings.embedded.action` | `user_setting_id` | link to one record | User Settings | `res.users.settings` | `n : 1` | `cascade` | stored |
| User, Change Password Wizard | `change.password.user` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| User, Change Password Wizard | `change.password.user` | `wizard_id` | link to one record | Change Password Wizard | `change.password.wizard` | `n : 1` | `cascade` | stored |
| Users Deletion Request | `res.users.deletion` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `set null` | stored |
| Users Log | `res.users.log` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Users application programming interface Keys | `res.users.apikeys` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| time-based one-time password rate limit logs | `auth.totp.rate.limit.log` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |

### 3.3 Platform Foundation

Core persistence, the entity and field registry, external identifiers, access control objects, actions, views and menus, sequences, attachments, scheduled jobs, configuration parameters, asset bundles, report layouts, paper formats, user-defined field definitions, the import and export machinery and the request-routing objects.

Specified in the platform documents of [`../overview/`](../overview/), [`../runtime/`](../runtime/), [`../interfaces/`](../interfaces/) and this folder.

#### Persistent entities (50)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Action uniform resource locator | `ir.actions.act_url` | `ir_act_url` | Persistent record with 9 stored columns. | Base |
| Action Window | `ir.actions.act_window` | `ir_act_window` | Persistent record with 20 stored columns; owns Action Window View, Embedded Actions; referenced by 7 relation fields. | Base |
| Action Window Close | `ir.actions.act_window_close` | `ir_actions` | Persistent record with 7 stored columns. | Base |
| Action Window View | `ir.actions.act_window.view` | `ir_act_window_view` | Persistent record with 5 stored columns. | Base |
| Actions | `ir.actions.actions` | `ir_actions` | Persistent record with 7 stored columns; referenced by 5 relation fields. | Base |
| Application | `ir.module.category` | `ir_module_category` | Persistent record with 6 stored columns; owns Application, Module, Privileges; referenced by 3 relation fields. | Base |
| Asset | `ir.asset` | `ir_asset` | Persistent record with 10 stored columns. | Base |
| Attachment | `ir.attachment` | `ir_attachment` | Persistent record with 20 stored columns; owns Metadata for voice attachments; company scoped; referenced by 47 relation fields. | Base |
| Automation Rule | `base.automation` | `base_automation` | Persistent record with 18 stored columns; belongs to Models; owns Server Actions; referenced by 1 relation field. | Automation Rules |
| Client Action | `ir.actions.client` | `ir_act_client` | Persistent record with 12 stored columns. | Base |
| Configuration Wizards | `ir.actions.todo` | `ir_actions_todo` | Persistent record with 4 stored columns; belongs to Actions; lifecycle states To Do, Done. | Base |
| Custom View | `ir.ui.view.custom` | `ir_ui_view_custom` | Persistent record with 3 stored columns; belongs to User, View. | Base |
| Decimal Precision | `decimal.precision` | `decimal_precision` | Persistent record with 2 stored columns. | Base |
| Default Values | `ir.default` | `ir_default` | Persistent record with 5 stored columns; belongs to Fields; company scoped. | Base |
| Embedded Actions | `ir.embedded.actions` | `ir_embedded_actions` | Persistent record with 11 stored columns; belongs to Action Window; owns Filters; referenced by 1 relation field. | Base |
| Exports | `ir.exports` | `ir_exports` | Persistent record with 2 stored columns; owns Exports Line; referenced by 1 relation field. | Base |
| Exports Line | `ir.exports.line` | `ir_exports_line` | Persistent record with 2 stored columns. | Base |
| Fields | `ir.model.fields` | `ir_model_fields` | Persistent record with 41 stored columns; belongs to Models; owns Fields Selection; lifecycle states Custom Field, Base Field; referenced by 21 relation fields. | Base |
| Fields Selection | `ir.model.fields.selection` | `ir_model_fields_selection` | Persistent record with 4 stored columns; belongs to Fields; referenced by 2 relation fields. | Base |
| Filters | `ir.filters` | `ir_filters` | Persistent record with 10 stored columns; referenced by 1 relation field. | Base |
| Geo Provider | `base.geo_provider` | `base_geo_provider` | Persistent record with 2 stored columns; referenced by 1 relation field. | Partners Geolocation |
| Logging | `ir.logging` | `ir_logging` | Persistent record with 8 stored columns. | Base |
| Mail Server | `ir.mail_server` | `ir_mail_server` | Persistent record with 23 stored columns; owns Email Templates, Mass Mailing; referenced by 7 relation fields. | Base |
| Menu | `ir.ui.menu` | `ir_ui_menu` | Persistent record with 7 stored columns; owns Menu; referenced by 3 relation fields. | Base |
| Model Access | `ir.model.access` | `ir_model_access` | Persistent record with 8 stored columns; belongs to Models. | Base |
| Model Constraint | `ir.model.constraint` | `ir_model_constraint` | Persistent record with 6 stored columns; belongs to Models, Module. | Base |
| Model Data | `ir.model.data` | `ir_model_data` | Persistent record with 5 stored columns; referenced by 1 relation field. | Base |
| Model Inheritance Tree | `ir.model.inherit` | `ir_model_inherit` | Persistent record with 3 stored columns; belongs to Models. | Base |
| Models | `ir.model` | `ir_model` | Persistent record with 15 stored columns; owns Fields, Model Access, Record Rule and 1 further collections; lifecycle states Custom Object, Base Object; referenced by 38 relation fields. | Base |
| Module | `ir.module.module` | `ir_module_module` | Persistent record with 26 stored columns; owns Attachment, Module dependency, Module exclusion; lifecycle states x; referenced by 17 relation fields. | Base |
| Module dependency | `ir.module.module.dependency` | `ir_module_module_dependency` | Persistent record with 3 stored columns; lifecycle states x. | Base |
| Module exclusion | `ir.module.module.exclusion` | `ir_module_module_exclusion` | Persistent record with 2 stored columns; lifecycle states x. | Base |
| Paper Format Config | `report.paperformat` | `report_paperformat` | Persistent record with 15 stored columns; owns Report Action; referenced by 2 relation fields. | Base |
| Point of Sale Orders Report | `report.pos.order` | `report_pos_order` | Persistent record with 26 stored columns; lifecycle states New, Paid, Posted, Cancelled; company scoped. | Point of Sale |
| Profiling results | `ir.profile` | `ir_profile` | Persistent record with 12 stored columns. | Base |
| Progress of Scheduled Actions | `ir.cron.progress` | `ir_cron_progress` | Persistent record with 5 stored columns; belongs to Scheduled Actions. | Base |
| Properties Base Definition | `properties.base.definition` | `properties_base_definition` | Persistent record with 2 stored columns; belongs to Fields; referenced by 1 relation field. | Base |
| Record Rule | `ir.rule` | `ir_rule` | Persistent record with 9 stored columns; belongs to Models; referenced by 1 relation field. | Base |
| Relation Model | `ir.model.relation` | `ir_model_relation` | Persistent record with 3 stored columns; belongs to Models, Module. | Base |
| Report Action | `ir.actions.report` | `ir_act_report_xml` | Persistent record with 18 stored columns; referenced by 7 relation fields. | Base |
| Report Layout | `report.layout` | `report_layout` | Persistent record with 5 stored columns; belongs to View; referenced by 1 relation field. | Base |
| Sequence | `ir.sequence` | `ir_sequence` | Persistent record with 11 stored columns; owns Sequence Date Range; company scoped; referenced by 20 relation fields. | Base |
| Sequence Date Range | `ir.sequence.date_range` | `ir_sequence_date_range` | Persistent record with 4 stored columns; belongs to Sequence. | Base |
| Server Action History | `ir.actions.server.history` | `ir_actions_server_history` | Persistent record with 2 stored columns; belongs to Server Actions; referenced by 1 relation field. | Base |
| Server Actions | `ir.actions.server` | `ir_act_server` | Persistent record with 46 stored columns; belongs to Models; owns Scheduled Actions, Server Actions; lifecycle states Update Record, Create Record, Duplicate Record, Execute Code, Send Webhook Notification, Multi Actions, Create Activity, Send Email, Add Followers, Remove Followers, Send Text Message; referenced by 5 relation fields. | Base |
| Stock Quantity Report | `report.stock.quantity` | `report_stock_quantity` | Persistent record with 7 stored columns; lifecycle states Forecasted Stock, Forecasted Receipts, Forecasted Deliveries; company scoped. | Inventory |
| System Parameter | `ir.config_parameter` | `ir_config_parameter` | Persistent record with 2 stored columns. | Base |
| Tasks Analysis | `report.project.task.user` | `report_project_task_user` | Persistent record with 38 stored columns; owns Skill level for employee; lifecycle states In Progress, Done, Waiting, Approved, Cancelled, Changes Requested; company scoped. | Project |
| Triggered actions | `ir.cron.trigger` | `ir_cron_trigger` | Persistent record with 2 stored columns; belongs to Scheduled Actions; referenced by 1 relation field. | Base |
| View | `ir.ui.view` | `ir_ui_view` | Persistent record with 24 stored columns; owns Model Page, Page, View; referenced by 20 relation fields. | Base |

#### Interactive assistant entities (19)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Change Production Qty | `change.production.qty` | `change_production_qty` | Interactive assistant with 2 stored columns; belongs to Manufacturing Order. | Manufacturing |
| Config | `res.config` | `res_config` | Interactive assistant with 0 stored columns. | Base |
| Config Settings | `res.config.settings` | `res_config_settings` | Interactive assistant with 278 stored columns; belongs to Companies, Currency; owns Models; carries the state field `account_peppol_proxy_state`; 3 state fields in all; company scoped. | Base |
| Create Menu Wizard | `wizard.ir.model.menu.create` | `wizard_ir_model_menu_create` | Interactive assistant with 2 stored columns; belongs to Menu. | Base |
| Demo | `ir.demo` | `ir_demo` | Interactive assistant with 0 stored columns. | Base |
| Demo failure | `ir.demo_failure` | `ir_demo_failure` | Interactive assistant with 3 stored columns; belongs to Module. | Base |
| Demo Failure wizard | `ir.demo_failure.wizard` | `ir_demo_failure_wizard` | Interactive assistant with 0 stored columns; owns Demo failure; referenced by 1 relation field. | Base |
| Enable profiling for some time | `base.enable.profiling.wizard` | `base_enable_profiling_wizard` | Interactive assistant with 2 stored columns. | Base |
| Import Module | `base.import.module` | `base_import_module` | Interactive assistant with 6 stored columns; lifecycle states init, done. | Base import module |
| Install Language | `base.language.install` | `base_language_install` | Interactive assistant with 1 stored column. | Base |
| Language Export | `base.language.export` | `base_language_export` | Interactive assistant with 8 stored columns; lifecycle states choose, get. | Base |
| Language Import | `base.language.import` | `base_language_import` | Interactive assistant with 5 stored columns. | Base |
| Module Activation Request | `base.module.install.request` | `base_module_install_request` | Interactive assistant with 3 stored columns; belongs to Module, User. | Base - Module Install Request |
| Module Activation Review | `base.module.install.review` | `base_module_install_review` | Interactive assistant with 1 stored column; belongs to Module. | Base - Module Install Request |
| Module Uninstall | `base.module.uninstall` | `base_module_uninstall` | Interactive assistant with 1 stored column. | Base |
| Reset View Architecture Wizard | `reset.view.arch.wizard` | `reset_view_arch_wizard` | Interactive assistant with 3 stored columns. | Base |
| Server Action History Wizard | `server.action.history.wizard` | `server_action_history_wizard` | Interactive assistant with 2 stored columns; belongs to Server Action History. | Base |
| Update Module | `base.module.update` | `base_module_update` | Interactive assistant with 3 stored columns; lifecycle states init, done. | Base |
| Upgrade Module | `base.module.upgrade` | `base_module_upgrade` | Interactive assistant with 1 stored column. | Base |

#### Shared behaviour entities (58)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account report with payment lines | `report.account.report_invoice_with_payments` | `none` | Shared behaviour definition reused through composition. | Invoicing |
| Account report without payment lines | `report.account.report_invoice` | `none` | Shared behaviour merged into 1 entity. | Invoicing |
| Account Test Report | `report.account_test.report_accounttest` | `none` | Shared behaviour definition reused through composition. | Accounting Consistency Tests |
| Automatic Vacuum | `ir.autovacuum` | `none` | Shared behaviour definition reused through composition. | Base |
| Avatar Mixin | `avatar.mixin` | `none` | Shared behaviour merged into 5 entities. | Base |
| Base | `base` | `none` | Shared behaviour definition reused through composition. | Base |
| bill of materials Overview Report | `report.mrp.report_bom_structure` | `none` | Shared behaviour definition reused through composition. | Manufacturing |
| Employee Resume | `report.hr_skills.report_employee_cv` | `none` | Shared behaviour definition reused through composition. | Skills Management |
| Fields Converter | `ir.fields.converter` | `none` | Shared behaviour definition reused through composition. | Base |
| File streaming helper model for controllers | `ir.binary` | `none` | Shared behaviour definition reused through composition. | Base |
| Geo Coder | `base.geocoder` | `none` | Shared behaviour definition reused through composition. | Partners Geolocation |
| Get french point of sale hash integrity result as Portable Document Format. | `report.l10n_fr_pos_cert.report_pos_hash_integrity` | `none` | Shared behaviour definition reused through composition. | France - value-added tax Anti-Fraud Certification for Point of Sale (CGI 286 I-3 bis) |
| Get hash integrity result as Portable Document Format. | `report.account.report_hash_integrity` | `none` | Shared behaviour definition reused through composition. | Invoicing |
| Holidays Summary Report | `report.hr_holidays.report_holidayssummary` | `none` | Shared behaviour definition reused through composition. | Time Off |
| Hypertext Transfer Protocol Routing | `ir.http` | `none` | Shared behaviour definition reused through composition. | Base |
| Image Mixin | `image.mixin` | `none` | Shared behaviour merged into 13 entities. | Base |
| Lot Label Report | `report.stock.label_lot_template_view` | `none` | Shared behaviour definition reused through composition. | Inventory |
| manufacturing order Overview Report | `report.mrp.report_mo_overview` | `none` | Shared behaviour definition reused through composition. | Manufacturing |
| Module Reference Report (base) | `report.base.report_irmodulereference` | `none` | Shared behaviour definition reused through composition. | Base |
| Point of Sale Details | `report.point_of_sale.report_saledetails` | `none` | Shared behaviour merged into 1 entity. | Point of Sale |
| Point of Sale Invoice Report | `report.point_of_sale.report_invoice` | `none` | Shared behaviour definition reused through composition. | Point of Sale |
| Pricelist Report | `report.product.report_pricelist` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Product Label Report | `report.product.report_producttemplatelabel_dymo` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Product Label Report | `report.stock.label_product_product_view` | `none` | Shared behaviour definition reused through composition. | Inventory |
| Product Label Report 2x7 | `report.product.report_producttemplatelabel2x7` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Product Label Report 4x12 | `report.product.report_producttemplatelabel4x12` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Product Label Report 4x12 No Price | `report.product.report_producttemplatelabel4x12noprice` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Product Label Report 4x7 | `report.product.report_producttemplatelabel4x7` | `none` | Shared behaviour definition reused through composition. | Products & Pricelists |
| Properties Base Definition Mixin | `properties.base.definition.mixin` | `none` | Shared behaviour merged into 2 entities. | Base |
| Qweb | `ir.qweb` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field | `ir.qweb.field` | `none` | Shared behaviour merged into 17 entities. | Base |
| Qweb Field Barcode | `ir.qweb.field.barcode` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Contact | `ir.qweb.field.contact` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Date | `ir.qweb.field.date` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Datetime | `ir.qweb.field.datetime` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Duration | `ir.qweb.field.duration` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Float | `ir.qweb.field.float` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Float Time | `ir.qweb.field.float_time` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field hypertext markup language | `ir.qweb.field.html` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Image | `ir.qweb.field.image` | `none` | Shared behaviour merged into 1 entity. | Base |
| Qweb Field Image | `ir.qweb.field.image_url` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Integer | `ir.qweb.field.integer` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Many to One | `ir.qweb.field.many2one` | `none` | Shared behaviour merged into 2 entities. | Base |
| Qweb field many2many | `ir.qweb.field.many2many` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Monetary | `ir.qweb.field.monetary` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb field one2many | `ir.qweb.field.one2many` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field qweb | `ir.qweb.field.qweb` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Relative | `ir.qweb.field.relative` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Selection | `ir.qweb.field.selection` | `none` | Shared behaviour definition reused through composition. | Base |
| Qweb Field Text | `ir.qweb.field.text` | `none` | Shared behaviour definition reused through composition. | Base |
| Scheduled Actions | `ir.cron` | `none` | Shared behaviour definition reused through composition. | Base |
| Session sales details for a single employee | `report.pos_hr.single_employee_sales_report` | `none` | Shared behaviour definition reused through composition. | point of sale - human resources |
| Stock Reception Report | `report.stock.report_reception` | `none` | Shared behaviour definition reused through composition. | Inventory |
| Stock rule report | `report.stock.report_stock_rule` | `none` | Shared behaviour definition reused through composition. | Inventory |
| Swiss quick response-bill report | `report.l10n_ch.qr_report_main` | `none` | Shared behaviour definition reused through composition. | Switzerland - Accounting |
| template engine Field Time | `ir.qweb.field.time` | `none` | Shared behaviour definition reused through composition. | Base |
| Unknown | `_unknown` | `none` | Shared behaviour definition reused through composition. | Base |
| websocket message handling | `ir.websocket` | `none` | Shared behaviour definition reused through composition. | Instant Messaging Bus |

#### Relationships (237)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Action Window | `ir.actions.act_window` | `embedded_action_ids` | list of records | Embedded Actions | `ir.embedded.actions` | `1 : 0..n` | `mirror of the target column` | derived |
| Action Window | `ir.actions.act_window` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Action Window | `ir.actions.act_window` | `search_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Action Window | `ir.actions.act_window` | `view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `set null` | stored |
| Action Window | `ir.actions.act_window` | `view_ids` | list of records | Action Window View | `ir.actions.act_window.view` | `1 : 0..n` | `mirror of the target column` | derived |
| Action Window View | `ir.actions.act_window.view` | `act_window_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 0..1` | `cascade` | stored |
| Action Window View | `ir.actions.act_window.view` | `view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Actions | `ir.actions.actions` | `binding_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Application | `ir.module.category` | `child_ids` | list of records | Application | `ir.module.category` | `1 : 0..n` | `mirror of the target column` | derived |
| Application | `ir.module.category` | `module_ids` | list of records | Module | `ir.module.module` | `1 : 0..n` | `mirror of the target column` | derived |
| Application | `ir.module.category` | `parent_id` | link to one record | Application | `ir.module.category` | `n : 0..1` | `not declared` | stored |
| Application | `ir.module.category` | `privilege_ids` | list of records | Privileges | `res.groups.privilege` | `1 : 0..n` | `mirror of the target column` | derived |
| Asset | `ir.asset` | `theme_template_id` | link to one record | Theme Asset | `theme.ir.asset` | `n : 0..1` | `not declared` | stored |
| Asset | `ir.asset` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |
| Attachment | `ir.attachment` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Attachment | `ir.attachment` | `original_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Attachment | `ir.attachment` | `theme_template_id` | link to one record | Theme Attachments | `theme.ir.attachment` | `n : 0..1` | `not declared` | stored |
| Attachment | `ir.attachment` | `voice_ids` | list of records | Metadata for voice attachments | `discuss.voice.metadata` | `1 : 0..n` | `mirror of the target column` | derived |
| Attachment | `ir.attachment` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Automation Rule | `base.automation` | `action_server_ids` | list of records | Server Actions | `ir.actions.server` | `1 : 0..n` | `mirror of the target column` | derived |
| Automation Rule | `base.automation` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Automation Rule | `base.automation` | `on_change_field_ids` | list on both sides | Fields | `ir.model.fields` | `0..n : 0..n` | `not declared` | stored |
| Automation Rule | `base.automation` | `trg_date_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Automation Rule | `base.automation` | `trg_date_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Automation Rule | `base.automation` | `trg_selection_field_id` | link to one record | Fields Selection | `ir.model.fields.selection` | `n : 0..1` | `not declared` | stored |
| Automation Rule | `base.automation` | `trigger_field_ids` | list on both sides | Fields | `ir.model.fields` | `0..n : 0..n` | `not declared` | stored |
| Change Production Qty | `change.production.qty` | `mo_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 1` | `cascade` | stored |
| Config Settings | `res.config.settings` | `account_cash_basis_base_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_discount_expense_allocation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_discount_income_allocation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_interco_clearing_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_interco_payable_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_interco_receivable_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_journal_early_pay_discount_gain_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_journal_early_pay_discount_loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `account_journal_suspense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `active_provider_id` | link to one record | Payment Provider | `payment.provider` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `auth_signup_template_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `barcode_nomenclature_id` | link to one record | Barcode Nomenclature | `barcode.nomenclature` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `cloud_storage_migration_all_model_ids` | list of records | Models | `ir.model` | `1 : 0..n` | `mirror of the target column` | derived |
| Config Settings | `res.config.settings` | `cloud_storage_migration_message_model_ids` | list of records | Models | `ir.model` | `1 : 0..n` | `mirror of the target column` | derived |
| Config Settings | `res.config.settings` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `company_expense_allowed_payment_method_line_ids` | list on both sides | Payment Methods | `account.payment.method.line` | `0..n : 0..n` | `not declared` | derived |
| Config Settings | `res.config.settings` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `currency_exchange_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `digest_id` | link to one record | Digest | `digest.digest` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `expense_currency_exchange_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `expense_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `geoloc_provider_id` | link to one record | Geo Provider | `base.geo_provider` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `hr_expense_alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `income_currency_exchange_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `incoterm_id` | link to one record | Incoterms | `account.incoterms` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `invoice_mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `l10n_ar_tax_base_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `l10n_fr_reference_leave_type` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `l10n_mx_account_income_return_discount_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `l10n_pl_edi_certificate` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `l10n_vn_edi_default_symbol` | link to one record | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `lc_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `mass_mailing_mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_allowed_pricelist_ids` | list on both sides | Pricelist | `product.pricelist` | `0..n : 0..n` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_available_preset_ids` | list on both sides | Easily load a set of configuration options | `pos.preset` | `0..n : 0..n` | `not declared` | derived |
| Config Settings | `res.config.settings` | `pos_available_pricelist_ids` | list on both sides | Pricelist | `product.pricelist` | `0..n : 0..n` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_default_fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_default_preset_id` | link to one record | Easily load a set of configuration options | `pos.preset` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `pos_discount_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_fiscal_position_ids` | list on both sides | Fiscal Position | `account.fiscal.position` | `0..n : 0..n` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_iface_available_categ_ids` | list on both sides | Point of Sale Category | `pos.category` | `0..n : 0..n` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_selectable_categ_ids` | list on both sides | Point of Sale Category | `pos.category` | `0..n : 0..n` | `not declared` | stored |
| Config Settings | `res.config.settings` | `pos_sms_receipt_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `pos_tip_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Config Settings | `res.config.settings` | `predictive_lead_scoring_fields` | list on both sides | Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `0..n : 0..n` | `not declared` | derived |
| Config Settings | `res.config.settings` | `project_time_mode_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `purchase_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `sale_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `tax_cash_basis_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `transfer_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Config Settings | `res.config.settings` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |
| Config Settings | `res.config.settings` | `website_warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Configuration Wizards | `ir.actions.todo` | `action_id` | link to one record | Actions | `ir.actions.actions` | `n : 1` | `not declared` | stored |
| Create Menu Wizard | `wizard.ir.model.menu.create` | `menu_id` | link to one record | Menu | `ir.ui.menu` | `n : 1` | `cascade` | stored |
| Custom View | `ir.ui.view.custom` | `ref_id` | link to one record | View | `ir.ui.view` | `n : 1` | `cascade` | stored |
| Custom View | `ir.ui.view.custom` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Default Values | `ir.default` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `cascade` | stored |
| Default Values | `ir.default` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 1` | `cascade` | stored |
| Default Values | `ir.default` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `cascade` | stored |
| Demo Failure wizard | `ir.demo_failure.wizard` | `failure_ids` | list of records | Demo failure | `ir.demo_failure` | `1 : 0..n` | `mirror of the target column` | derived |
| Demo failure | `ir.demo_failure` | `module_id` | link to one record | Module | `ir.module.module` | `n : 1` | `not declared` | stored |
| Demo failure | `ir.demo_failure` | `wizard_id` | link to one record | Demo Failure wizard | `ir.demo_failure.wizard` | `n : 0..1` | `not declared` | stored |
| Embedded Actions | `ir.embedded.actions` | `action_id` | link to one record | Actions | `ir.actions.actions` | `n : 0..1` | `cascade` | stored |
| Embedded Actions | `ir.embedded.actions` | `filter_ids` | list of records | Filters | `ir.filters` | `1 : 0..n` | `mirror of the target column` | derived |
| Embedded Actions | `ir.embedded.actions` | `groups_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Embedded Actions | `ir.embedded.actions` | `parent_action_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 1` | `cascade` | stored |
| Embedded Actions | `ir.embedded.actions` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `cascade` | stored |
| Exports | `ir.exports` | `export_fields` | list of records | Exports Line | `ir.exports.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Exports Line | `ir.exports.line` | `export_id` | link to one record | Exports | `ir.exports` | `n : 0..1` | `cascade` | stored |
| Fields | `ir.model.fields` | `groups` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Fields | `ir.model.fields` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Fields | `ir.model.fields` | `related_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Fields | `ir.model.fields` | `relation_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Fields | `ir.model.fields` | `selection_ids` | list of records | Fields Selection | `ir.model.fields.selection` | `1 : 0..n` | `mirror of the target column` | derived |
| Fields | `ir.model.fields` | `serialization_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Fields Selection | `ir.model.fields.selection` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 1` | `cascade` | stored |
| Filters | `ir.filters` | `action_id` | link to one record | Actions | `ir.actions.actions` | `n : 0..1` | `cascade` | stored |
| Filters | `ir.filters` | `embedded_action_id` | link to one record | Embedded Actions | `ir.embedded.actions` | `n : 0..1` | `cascade` | stored |
| Filters | `ir.filters` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `cascade` | stored |
| Install Language | `base.language.install` | `first_lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | derived |
| Install Language | `base.language.install` | `lang_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | stored |
| Install Language | `base.language.install` | `website_ids` | list on both sides | Website | `website` | `0..n : 0..n` | `not declared` | stored |
| Language Export | `base.language.export` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | stored |
| Language Export | `base.language.export` | `modules` | list on both sides | Module | `ir.module.module` | `0..n : 0..n` | `not declared` | stored |
| Mail Server | `ir.mail_server` | `active_mailing_ids` | list of records | Mass Mailing | `mailing.mailing` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Server | `ir.mail_server` | `mail_template_ids` | list of records | Email Templates | `mail.template` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Server | `ir.mail_server` | `owner_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Menu | `ir.ui.menu` | `child_id` | list of records | Menu | `ir.ui.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Menu | `ir.ui.menu` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Menu | `ir.ui.menu` | `parent_id` | link to one record | Menu | `ir.ui.menu` | `n : 0..1` | `restrict` | stored |
| Model Access | `ir.model.access` | `group_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `restrict` | stored |
| Model Access | `ir.model.access` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Model Constraint | `ir.model.constraint` | `model` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Model Constraint | `ir.model.constraint` | `module` | link to one record | Module | `ir.module.module` | `n : 1` | `cascade` | stored |
| Model Inheritance Tree | `ir.model.inherit` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Model Inheritance Tree | `ir.model.inherit` | `parent_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Model Inheritance Tree | `ir.model.inherit` | `parent_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Models | `ir.model` | `access_ids` | list of records | Model Access | `ir.model.access` | `1 : 0..n` | `mirror of the target column` | derived |
| Models | `ir.model` | `field_id` | list of records | Fields | `ir.model.fields` | `1 : 0..n` | `mirror of the target column` | derived |
| Models | `ir.model` | `inherited_model_ids` | list on both sides | Models | `ir.model` | `0..n : 0..n` | `not declared` | derived |
| Models | `ir.model` | `rule_ids` | list of records | Record Rule | `ir.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Models | `ir.model` | `view_ids` | list of records | View | `ir.ui.view` | `1 : 0..n` | `mirror of the target column` | derived |
| Models | `ir.model` | `website_form_default_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Module | `ir.module.module` | `category_id` | link to one record | Application | `ir.module.category` | `n : 0..1` | `not declared` | stored |
| Module | `ir.module.module` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Module | `ir.module.module` | `dependencies_id` | list of records | Module dependency | `ir.module.module.dependency` | `1 : 0..n` | `mirror of the target column` | derived |
| Module | `ir.module.module` | `exclusion_ids` | list of records | Module exclusion | `ir.module.module.exclusion` | `1 : 0..n` | `mirror of the target column` | derived |
| Module | `ir.module.module` | `image_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Module Activation Request | `base.module.install.request` | `module_id` | link to one record | Module | `ir.module.module` | `n : 1` | `cascade` | stored |
| Module Activation Request | `base.module.install.request` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Module Activation Request | `base.module.install.request` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Module Activation Review | `base.module.install.review` | `module_id` | link to one record | Module | `ir.module.module` | `n : 1` | `cascade` | stored |
| Module Activation Review | `base.module.install.review` | `module_ids` | list on both sides | Module | `ir.module.module` | `0..n : 0..n` | `not declared` | derived |
| Module Uninstall | `base.module.uninstall` | `impacted_module_ids` | list on both sides | Module | `ir.module.module` | `0..n : 0..n` | `not declared` | stored |
| Module Uninstall | `base.module.uninstall` | `model_ids` | list on both sides | Models | `ir.model` | `0..n : 0..n` | `not declared` | derived |
| Module Uninstall | `base.module.uninstall` | `module_ids` | list on both sides | Module | `ir.module.module` | `0..n : 0..n` | `cascade` | stored |
| Module dependency | `ir.module.module.dependency` | `depend_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `not declared` | derived |
| Module dependency | `ir.module.module.dependency` | `module_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `cascade` | stored |
| Module exclusion | `ir.module.module.exclusion` | `exclusion_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `not declared` | derived |
| Module exclusion | `ir.module.module.exclusion` | `module_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `cascade` | stored |
| Paper Format Config | `report.paperformat` | `report_ids` | list of records | Report Action | `ir.actions.report` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders Report | `report.pos.order` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `pos_categ_id` | link to one record | Point of Sale Category | `pos.category` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `product_categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders Report | `report.pos.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Progress of Scheduled Actions | `ir.cron.progress` | `cron_id` | link to one record | Scheduled Actions | `ir.cron` | `n : 1` | `cascade` | stored |
| Properties Base Definition | `properties.base.definition` | `properties_field_id` | link to one record | Fields | `ir.model.fields` | `n : 1` | `cascade` | stored |
| Properties Base Definition Mixin | `properties.base.definition.mixin` | `properties_base_definition_id` | link to one record | Properties Base Definition | `properties.base.definition` | `n : 0..1` | `not declared` | derived |
| Record Rule | `ir.rule` | `groups` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `restrict` | stored |
| Record Rule | `ir.rule` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Relation Model | `ir.model.relation` | `model` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Relation Model | `ir.model.relation` | `module` | link to one record | Module | `ir.module.module` | `n : 1` | `cascade` | stored |
| Report Action | `ir.actions.report` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Report Action | `ir.actions.report` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | derived |
| Report Action | `ir.actions.report` | `paperformat_id` | link to one record | Paper Format Config | `report.paperformat` | `n : 0..1` | `not declared` | stored |
| Report Layout | `report.layout` | `view_id` | link to one record | View | `ir.ui.view` | `n : 1` | `not declared` | stored |
| Reset View Architecture Wizard | `reset.view.arch.wizard` | `compare_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Reset View Architecture Wizard | `reset.view.arch.wizard` | `view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Scheduled Actions | `ir.cron` | `ir_actions_server_id` | link to one record | Server Actions | `ir.actions.server` | `n : 1` | `restrict` | stored |
| Scheduled Actions | `ir.cron` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Sequence | `ir.sequence` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Sequence | `ir.sequence` | `date_range_ids` | list of records | Sequence Date Range | `ir.sequence.date_range` | `1 : 0..n` | `mirror of the target column` | derived |
| Sequence Date Range | `ir.sequence.date_range` | `sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 1` | `cascade` | stored |
| Server Action History | `ir.actions.server.history` | `action_id` | link to one record | Server Actions | `ir.actions.server` | `n : 1` | `cascade` | stored |
| Server Action History Wizard | `server.action.history.wizard` | `action_id` | link to one record | Server Actions | `ir.actions.server` | `n : 0..1` | `not declared` | stored |
| Server Action History Wizard | `server.action.history.wizard` | `revision` | link to one record | Server Action History | `ir.actions.server.history` | `n : 1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `restrict` | stored |
| Server Actions | `ir.actions.server` | `activity_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `available_model_ids` | list on both sides | Models | `ir.model` | `0..n : 0..n` | `not declared` | derived |
| Server Actions | `ir.actions.server` | `base_automation_id` | link to one record | Automation Rule | `base.automation` | `n : 0..1` | `cascade` | stored |
| Server Actions | `ir.actions.server` | `child_ids` | list of records | Server Actions | `ir.actions.server` | `1 : 0..n` | `mirror of the target column` | derived |
| Server Actions | `ir.actions.server` | `crud_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `ir_cron_ids` | list of records | Scheduled Actions | `ir.cron` | `1 : 0..n` | `mirror of the target column` | derived |
| Server Actions | `ir.actions.server` | `link_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Server Actions | `ir.actions.server` | `parent_id` | link to one record | Server Actions | `ir.actions.server` | `n : 0..1` | `cascade` | stored |
| Server Actions | `ir.actions.server` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `selection_value` | link to one record | Fields Selection | `ir.model.fields.selection` | `n : 0..1` | `cascade` | stored |
| Server Actions | `ir.actions.server` | `sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `set null` | stored |
| Server Actions | `ir.actions.server` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `set null` | stored |
| Server Actions | `ir.actions.server` | `update_field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `cascade` | stored |
| Server Actions | `ir.actions.server` | `update_related_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | stored |
| Server Actions | `ir.actions.server` | `webhook_field_ids` | list on both sides | Fields | `ir.model.fields` | `0..n : 0..n` | `not declared` | stored |
| Stock Quantity Report | `report.stock.quantity` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Stock Quantity Report | `report.stock.quantity` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Stock Quantity Report | `report.stock.quantity` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | stored |
| Stock Quantity Report | `report.stock.quantity` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `dependent_ids` | list on both sides | Task | `project.task` | `0..n : 0..n` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `parent_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `personal_stage_type_ids` | list on both sides | Task Stage | `project.task.type` | `0..n : 0..n` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `stage_id` | link to one record | Task Stage | `project.task.type` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `tag_ids` | list on both sides | Project Tags | `project.tags` | `0..n : 0..n` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Tasks Analysis | `report.project.task.user` | `user_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Triggered actions | `ir.cron.trigger` | `cron_id` | link to one record | Scheduled Actions | `ir.cron` | `n : 1` | `cascade` | stored |
| View | `ir.ui.view` | `controller_page_ids` | list of records | Model Page | `website.controller.page` | `1 : 0..n` | `mirror of the target column` | derived |
| View | `ir.ui.view` | `first_page_id` | link to one record | Page | `website.page` | `n : 0..1` | `not declared` | derived |
| View | `ir.ui.view` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| View | `ir.ui.view` | `inherit_children_ids` | list of records | View | `ir.ui.view` | `1 : 0..n` | `mirror of the target column` | derived |
| View | `ir.ui.view` | `inherit_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `restrict` | stored |
| View | `ir.ui.view` | `model_data_id` | link to one record | Model Data | `ir.model.data` | `n : 0..1` | `not declared` | derived |
| View | `ir.ui.view` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | derived |
| View | `ir.ui.view` | `page_ids` | list of records | Page | `website.page` | `1 : 0..n` | `mirror of the target column` | derived |
| View | `ir.ui.view` | `theme_template_id` | link to one record | Theme user interface View | `theme.ir.ui.view` | `n : 0..1` | `not declared` | stored |
| View | `ir.ui.view` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |

### 3.4 Calendar and Scheduling

Calendar events, attendees, recurrence rules, reminders, providers and the synchronization with external calendars.

Specified in the domain folder `../domains/calendar-and-scheduling/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (6)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Calendar Attendee Information | `calendar.attendee` | `calendar_attendee` | Persistent record with 6 stored columns; belongs to Calendar Event, Contact; lifecycle states x; referenced by 1 relation field. | Calendar |
| Calendar Event | `calendar.event` | `calendar_event` | Persistent record with 32 stored columns; owns Activity, Calendar Attendee Information; carries the state field `current_status`; referenced by 7 relation fields. | Calendar |
| Calendar Filters | `calendar.filters` | `calendar_filters` | Persistent record with 4 stored columns; belongs to Contact, User. | Calendar |
| Event Alarm | `calendar.alarm` | `calendar_alarm` | Persistent record with 9 stored columns; referenced by 1 relation field. | Calendar |
| Event Meeting Type | `calendar.event.type` | `calendar_event_type` | Persistent record with 2 stored columns; referenced by 1 relation field. | Calendar |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_recurrence` | Persistent record with 27 stored columns; owns Calendar Event; referenced by 2 relation fields. | Calendar |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `calendar_popover_delete_wizard` | Interactive assistant with 6 stored columns. | Calendar |
| Calendar Provider Configuration Wizard | `calendar.provider.config` | `calendar_provider_config` | Interactive assistant with 7 stored columns. | Calendar |
| Google Calendar Account Reset | `google.calendar.account.reset` | `google_calendar_account_reset` | Interactive assistant with 3 stored columns; belongs to User. | Google Calendar |
| Microsoft Calendar Account Reset | `microsoft.calendar.account.reset` | `microsoft_calendar_account_reset` | Interactive assistant with 3 stored columns; belongs to User. | Outlook Calendar |

#### Shared behaviour entities (7)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Event Alarm Manager | `calendar.alarm_manager` | `none` | Shared behaviour definition reused through composition. | Calendar |
| Google Gmail Mixin | `google.gmail.mixin` | `none` | Shared behaviour merged into 2 entities. | Google Gmail |
| Google Service | `google.service` | `none` | Shared behaviour definition reused through composition. | Google Users |
| Microsoft Outlook Mixin | `microsoft.outlook.mixin` | `none` | Shared behaviour merged into 2 entities. | Microsoft Outlook |
| Microsoft Service | `microsoft.service` | `none` | Shared behaviour definition reused through composition. | Microsoft Users |
| Synchronize a record with Google Calendar | `google.calendar.sync` | `none` | Shared behaviour merged into 2 entities. | Google Calendar |
| Synchronize a record with Microsoft Calendar | `microsoft.calendar.sync` | `none` | Shared behaviour merged into 2 entities. | Outlook Calendar |

#### Relationships (29)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Calendar Attendee Information | `calendar.attendee` | `event_id` | link to one record | Calendar Event | `calendar.event` | `n : 1` | `cascade` | stored |
| Calendar Attendee Information | `calendar.attendee` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Calendar Attendee Information | `calendar.attendee` | `recurrence_id` | link to one record | Event Recurrence Rule | `calendar.recurrence` | `n : 0..1` | `not declared` | derived |
| Calendar Event | `calendar.event` | `activity_ids` | list of records | Activity | `mail.activity` | `1 : 0..n` | `mirror of the target column` | derived |
| Calendar Event | `calendar.event` | `alarm_ids` | list on both sides | Event Alarm | `calendar.alarm` | `0..n : 0..n` | `restrict` | stored |
| Calendar Event | `calendar.event` | `applicant_id` | link to one record | Applicant | `hr.applicant` | `n : 0..1` | `set null` | stored |
| Calendar Event | `calendar.event` | `attendee_ids` | list of records | Calendar Attendee Information | `calendar.attendee` | `1 : 0..n` | `mirror of the target column` | derived |
| Calendar Event | `calendar.event` | `categ_ids` | list on both sides | Event Meeting Type | `calendar.event.type` | `0..n : 0..n` | `not declared` | stored |
| Calendar Event | `calendar.event` | `current_attendee` | link to one record | Calendar Attendee Information | `calendar.attendee` | `n : 0..1` | `not declared` | derived |
| Calendar Event | `calendar.event` | `invalid_email_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Calendar Event | `calendar.event` | `opportunity_id` | link to one record | Lead | `crm.lead` | `n : 0..1` | `set null` | stored |
| Calendar Event | `calendar.event` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Calendar Event | `calendar.event` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Calendar Event | `calendar.event` | `recurrence_id` | link to one record | Event Recurrence Rule | `calendar.recurrence` | `n : 0..1` | `not declared` | stored |
| Calendar Event | `calendar.event` | `res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Calendar Event | `calendar.event` | `unavailable_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Calendar Event | `calendar.event` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Calendar Event | `calendar.event` | `videocall_channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | stored |
| Calendar Filters | `calendar.filters` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Calendar Filters | `calendar.filters` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `calendar_event_id` | link to one record | Calendar Event | `calendar.event` | `n : 0..1` | `not declared` | stored |
| Calendar Popover Delete Wizard | `calendar.popover.delete.wizard` | `recipient_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Event Alarm | `calendar.alarm` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Event Alarm | `calendar.alarm` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Event Recurrence Rule | `calendar.recurrence` | `base_event_id` | link to one record | Calendar Event | `calendar.event` | `n : 0..1` | `set null` | stored |
| Event Recurrence Rule | `calendar.recurrence` | `calendar_event_ids` | list of records | Calendar Event | `calendar.event` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Recurrence Rule | `calendar.recurrence` | `trigger_id` | link to one record | Triggered actions | `ir.cron.trigger` | `n : 0..1` | `not declared` | stored |
| Google Calendar Account Reset | `google.calendar.account.reset` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Microsoft Calendar Account Reset | `microsoft.calendar.account.reset` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |

### 3.5 Contacts and Organizations

Parties — individuals, organizations and bare addresses — companies, banks and bank accounts, countries, country subdivisions and country groups, cities, languages, partner tags, industries and the document layout.

Specified in [`../domains/contacts-and-organizations/`](../domains/contacts-and-organizations/).

#### Persistent entities (11)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Bank | `res.bank` | `res_bank` | Persistent record with 15 stored columns; referenced by 5 relation fields. | Base |
| Bank Accounts | `res.partner.bank` | `res_partner_bank` | Persistent record with 24 stored columns; belongs to Contact; owns Journal, Journal Entry; company scoped; referenced by 15 relation fields. | Base |
| City | `res.city` | `res_city` | Persistent record with 5 stored columns; belongs to Country; owns Brazilian city zip range; referenced by 3 relation fields. | Extended Addresses |
| Companies | `res.company` | `res_company` | Persistent record with 334 stored columns; belongs to Contact, Currency; owns Account electronic data interchange proxy user, Certificate, Companies and 5 further collections; states of `account_peppol_proxy_state`: Not registered, Can send but not receive, Can send, pending registration to receive, Can send and receive, Rejected; 5 state fields in all; referenced by 211 relation fields. | Base |
| Contact | `res.partner` | `res_partner` | Persistent record with 175 stored columns; owns Analytic Account, Applicant, Argentinean Partner Taxes and 22 further collections; states of `peppol_verification_state`: Unchecked, Partner is not on Peppol, Partner cannot receive format, Partner is on Peppol; 5 state fields in all; company scoped; referenced by 232 relation fields. | Base |
| Country | `res.country` | `res_country` | Persistent record with 18 stored columns; owns Country state; referenced by 62 relation fields. | Base |
| Country Group | `res.country.group` | `res_country_group` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Base |
| Country state | `res.country.state` | `res_country_state` | Persistent record with 4 stored columns; belongs to Country; referenced by 22 relation fields. | Base |
| Industry | `res.partner.industry` | `res_partner_industry` | Persistent record with 3 stored columns; referenced by 3 relation fields. | Base |
| Languages | `res.lang` | `res_lang` | Persistent record with 12 stored columns; referenced by 16 relation fields. | Base |
| Partner Tags | `res.partner.category` | `res_partner_category` | Persistent record with 5 stored columns; owns Partner Tags; referenced by 5 relation fields. | Base |

#### Interactive assistant entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Company Document Layout | `base.document.layout` | `base_document_layout` | Interactive assistant with 3 stored columns; belongs to Companies; company scoped. | Web |
| Merge Partner Line | `base.partner.merge.line` | `base_partner_merge_line` | Interactive assistant with 3 stored columns; referenced by 1 relation field. | Base |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `base_partner_merge_automatic_wizard` | Interactive assistant with 12 stored columns; owns Merge Partner Line; lifecycle states Option, Selection, Finished; referenced by 1 relation field. | Base |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Address Format | `format.address.mixin` | `none` | Shared behaviour merged into 3 entities. | Base |
| Country Specific value-added tax Label | `format.vat.label.mixin` | `none` | Shared behaviour merged into 2 entities. | Base |

#### Relationships (230)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Bank | `res.bank` | `country` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Bank | `res.bank` | `intermediary_bank_id` | link to one record | Bank | `res.bank` | `n : 0..1` | `not declared` | stored |
| Bank | `res.bank` | `state` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Bank Accounts | `res.partner.bank` | `bank_id` | link to one record | Bank | `res.bank` | `n : 0..1` | `not declared` | stored |
| Bank Accounts | `res.partner.bank` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Bank Accounts | `res.partner.bank` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Bank Accounts | `res.partner.bank` | `duplicate_bank_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Bank Accounts | `res.partner.bank` | `employee_id` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | derived |
| Bank Accounts | `res.partner.bank` | `journal_id` | list of records | Journal | `account.journal` | `1 : 0..n` | `mirror of the target column` | derived |
| Bank Accounts | `res.partner.bank` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Bank Accounts | `res.partner.bank` | `related_moves` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| City | `res.city` | `country_id` | link to one record | Country | `res.country` | `n : 1` | `not declared` | stored |
| City | `res.city` | `l10n_br_zip_range_ids` | list of records | Brazilian city zip range | `l10n_br.zip.range` | `1 : 0..n` | `mirror of the target column` | derived |
| City | `res.city` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_cash_basis_base_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_default_pos_receivable_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_discount_expense_allocation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_discount_income_allocation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_edi_proxy_client_ids` | list of records | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `account_enabled_tax_country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | derived |
| Companies | `res.company` | `account_fiscal_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_interco_clearing_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_interco_payable_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_interco_receivable_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_journal_early_pay_discount_gain_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_journal_early_pay_discount_loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_journal_suspense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_opening_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `account_opening_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_peppol_edi_user` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `account_production_wip_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_production_wip_overhead_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_purchase_receipt_fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_purchase_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_sale_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_stock_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `account_stock_valuation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `all_child_ids` | list of records | Companies | `res.company` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `automatic_entry_default_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `bank_journal_ids` | list of records | Journal | `account.journal` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `batch_payment_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `child_ids` | list of records | Companies | `res.company` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `company_expense_allowed_payment_method_line_ids` | list on both sides | Payment Methods | `account.payment.method.line` | `0..n : 0..n` | `not declared` | stored |
| Companies | `res.company` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `currency_exchange_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Companies | `res.company` | `default_cash_difference_expense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `default_cash_difference_income_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `domestic_fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `downpayment_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `dropship_subcontractor_pick_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `expense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `expense_accrual_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `expense_currency_exchange_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `expense_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `external_report_layout_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `fiscal_position_ids` | list of records | Fiscal Position | `account.fiscal.position` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `income_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `income_currency_exchange_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `incoterm_id` | link to one record | Incoterms | `account.incoterms` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `internal_project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `internal_transit_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `restrict` | stored |
| Companies | `res.company` | `l10n_ar_tax_base_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_cz_tax_office_id` | link to one record | Tax office in Czech Republic | `l10n_cz.tax_office` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_ee_rounding_difference_loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_ee_rounding_difference_profit_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_es_edi_facturae_certificate_ids` | list of records | Certificate | `certificate.certificate` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `l10n_es_edi_verifactu_certificate_ids` | list of records | Certificate | `certificate.certificate` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `l10n_es_edi_verifactu_chain_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_es_sii_certificate_id` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_es_sii_certificate_ids` | list of records | Certificate | `certificate.certificate` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `l10n_es_tbai_certificate_id` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_es_tbai_certificate_ids` | list of records | Certificate | `certificate.certificate` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `l10n_es_tbai_chain_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_fr_closing_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_fr_pos_cert_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_fr_reference_leave_type` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_fr_rounding_difference_loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_fr_rounding_difference_profit_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_hr_mer_purchase_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_in_withholding_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_in_withholding_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_it_eco_index_office` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_it_edi_doi_fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_it_edi_doi_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_it_edi_proxy_user_id` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `l10n_it_edi_purchase_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_it_tax_representative_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_mx_income_re_invoicing_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_mx_income_return_discount_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_my_edi_default_import_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_my_edi_proxy_user_id` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `l10n_nl_rounding_difference_loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_nl_rounding_difference_profit_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_pl_edi_certificate` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_pl_reports_tax_office_id` | link to one record | Tax Office in Poland | `l10n_pl.l10n_pl_tax_office` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_ro_edi_anaf_imported_inv_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_sa_private_key_id` | link to one record | Cryptographic Keys | `certificate.key` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_tr_nilvera_purchase_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `l10n_vn_pos_default_symbol` | link to one record | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `lc_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `ldaps` | list of records | Company directory access protocol configuration | `res.company.ldap` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `leave_timesheet_task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `multi_vat_foreign_country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | derived |
| Companies | `res.company` | `nemhandel_edi_user` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `nemhandel_purchase_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `nomenclature_id` | link to one record | Barcode Nomenclature | `barcode.nomenclature` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `paperformat_id` | link to one record | Paper Format Config | `report.paperformat` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `parent_id` | link to one record | Companies | `res.company` | `n : 0..1` | `restrict` | stored |
| Companies | `res.company` | `parent_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | derived |
| Companies | `res.company` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Companies | `res.company` | `peppol_parent_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `peppol_purchase_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `peppol_self_billing_reception_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `price_difference_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `project_time_mode_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `restrict` | stored |
| Companies | `res.company` | `resource_calendar_ids` | list of records | Resource Working Time | `resource.calendar` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `revenue_accrual_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `root_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `sale_discount_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `sale_order_template_id` | link to one record | Quotation Template | `sale.order.template` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `sms_twilio_number_ids` | list of records | Twilio Number | `sms.twilio.number` | `1 : 0..n` | `mirror of the target column` | derived |
| Companies | `res.company` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | derived |
| Companies | `res.company` | `stock_mail_confirmation_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `stock_sms_confirmation_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `subcontracting_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `tax_cash_basis_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `timesheet_encode_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `transfer_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `uninstalled_l10n_module_ids` | list on both sides | Module | `ir.module.module` | `0..n : 0..n` | `not declared` | derived |
| Companies | `res.company` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Companies | `res.company` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Companies | `res.company` | `withholding_tax_base_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Company Document Layout | `base.document.layout` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Company Document Layout | `base.document.layout` | `report_layout_id` | link to one record | Report Layout | `report.layout` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `activation` | link to one record | Partner Activation | `res.partner.activation` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `applicant_ids` | list of records | Applicant | `hr.applicant` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `assigned_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `available_invoice_template_pdf_report_ids` | list of records | Report Action | `ir.actions.report` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `bank_ids` | list of records | Bank Accounts | `res.partner.bank` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `bom_ids` | list on both sides | Bill of Material | `mrp.bom` | `0..n : 0..n` | `not declared` | derived |
| Contact | `res.partner` | `buyer_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `category_id` | list on both sides | Partner Tags | `res.partner.category` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `channel_ids` | list on both sides | Discussion Channel | `discuss.channel` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `channel_member_ids` | list of records | Channel Member | `discuss.channel.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `chatbot_script_ids` | list of records | Chatbot Script | `chatbot.script` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `child_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `city_id` | link to one record | City | `res.city` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `contract_ids` | list of records | Analytic Account | `account.analytic.account` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `employee_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `grade_id` | link to one record | Partner Grade | `res.partner.grade` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `implemented_partner_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `industry_id` | link to one record | Industry | `res.partner.industry` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `invoice_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `invoice_template_pdf_report_id` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_ar_afip_responsibility_type_id` | link to one record | ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_ar_partner_tax_ids` | list of records | Argentinean Partner Taxes | `l10n_ar.partner.tax` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `l10n_es_edi_facturae_ac_role_type_ids` | list on both sides | Administrative Center Role Type | `l10n_es_edi_facturae.ac_role_type` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `l10n_in_pan_entity_id` | link to one record | Indian permanent account number Entity | `l10n_in.pan.entity` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `l10n_it_edi_doi_ids` | list of records | Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `l10n_latam_identification_type_id` | link to one record | Identification Types | `l10n_latam.identification.type` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_my_edi_industrial_classification` | link to one record | Malaysian Industry Classification | `l10n_my_edi.industry_classification` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_pe_district` | link to one record | District | `l10n_pe.res.city.district` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_pl_parent_lgu` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_tr_nilvera_customer_alias_id` | link to one record | Customer Alias on Nilvera | `l10n_tr.nilvera.alias` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_tr_nilvera_customer_alias_ids` | list of records | Customer Alias on Nilvera | `l10n_tr.nilvera.alias` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `l10n_tr_tax_office_id` | link to one record | Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `l10n_vn_edi_symbol` | link to one record | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `main_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `meeting_ids` | list on both sides | Calendar Event | `calendar.event` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `opportunity_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `parent_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `payment_token_ids` | list of records | Payment Token | `payment.token` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `pos_order_ids` | list of records | Point of Sale Orders | `pos.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | derived |
| Contact | `res.partner` | `project_ids` | list of records | Project | `project.project` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `property_account_payable_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `property_account_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_account_receivable_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `property_delivery_carrier_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_inbound_payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_outbound_payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_payment_term_id` | link to one record | Payment Terms | `account.payment.term` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `property_product_pricelist` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `property_purchase_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_stock_customer` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_stock_subcontractor` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_stock_supplier` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `property_supplier_payment_term_id` | link to one record | Payment Terms | `account.payment.term` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `purchase_line_ids` | list of records | Purchase Order Line | `purchase.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `ref_company_ids` | list of records | Companies | `res.company` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `rtc_session_ids` | list of records | Mail RTC session | `discuss.channel.rtc.session` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `sale_order_ids` | list of records | Sales Order | `sale.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `same_company_registry_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `same_vat_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `self` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Contact | `res.partner` | `slide_channel_completed_ids` | list of records | Course | `slide.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `slide_channel_ids` | list on both sides | Course | `slide.channel` | `0..n : 0..n` | `not declared` | derived |
| Contact | `res.partner` | `specific_property_product_pricelist` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `restrict` | stored |
| Contact | `res.partner` | `task_ids` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Contact | `res.partner` | `user_ids` | list of records | User | `res.users` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `visitor_ids` | list of records | Website Visitor | `website.visitor` | `1 : 0..n` | `mirror of the target column` | derived |
| Contact | `res.partner` | `website_tag_ids` | list on both sides | Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `0..n : 0..n` | `not declared` | stored |
| Contact | `res.partner` | `wishlist_ids` | list of records | Product Wishlist | `product.wishlist` | `1 : 0..n` | `mirror of the target column` | derived |
| Country | `res.country` | `address_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `not declared` | stored |
| Country | `res.country` | `country_group_ids` | list on both sides | Country Group | `res.country.group` | `0..n : 0..n` | `not declared` | stored |
| Country | `res.country` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Country | `res.country` | `state_ids` | list of records | Country state | `res.country.state` | `1 : 0..n` | `mirror of the target column` | derived |
| Country Group | `res.country.group` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Country Group | `res.country.group` | `exclude_state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | stored |
| Country Group | `res.country.group` | `pricelist_ids` | list on both sides | Pricelist | `product.pricelist` | `0..n : 0..n` | `not declared` | stored |
| Country state | `res.country.state` | `country_id` | link to one record | Country | `res.country` | `n : 1` | `not declared` | stored |
| Merge Partner Line | `base.partner.merge.line` | `wizard_id` | link to one record | Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `n : 0..1` | `not declared` | stored |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `current_line_id` | link to one record | Merge Partner Line | `base.partner.merge.line` | `n : 0..1` | `not declared` | stored |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `dst_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `line_ids` | list of records | Merge Partner Line | `base.partner.merge.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Merge Partner Wizard | `base.partner.merge.automatic.wizard` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Partner Tags | `res.partner.category` | `child_ids` | list of records | Partner Tags | `res.partner.category` | `1 : 0..n` | `mirror of the target column` | derived |
| Partner Tags | `res.partner.category` | `parent_id` | link to one record | Partner Tags | `res.partner.category` | `n : 0..1` | `cascade` | stored |
| Partner Tags | `res.partner.category` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |

### 3.6 Messaging and Activities

Discussion threads, messages, followers and notifications, activities and activity plans, message templates, aliases and the incoming and outgoing mail gateways, channels and channel members, live chat, chatbots, text messages, postal mail, push notifications, link previews, reactions and ratings.

Specified in [`../domains/messaging-and-activities/`](../domains/messaging-and-activities/).

#### Persistent entities (59)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Activity | `mail.activity` | `mail_activity` | Persistent record with 18 stored columns; lifecycle states Overdue, Today, Planned, Done. | Discuss |
| Activity Plan | `mail.activity.plan` | `mail_activity_plan` | Persistent record with 6 stored columns; belongs to Models; owns Activity plan template; company scoped; referenced by 3 relation fields. | Discuss |
| Activity plan template | `mail.activity.plan.template` | `mail_activity_plan_template` | Persistent record with 10 stored columns; belongs to Activity Plan, Activity Type. | Discuss |
| Activity Type | `mail.activity.type` | `mail_activity_type` | Persistent record with 15 stored columns; referenced by 15 relation fields. | Discuss |
| Canned Response | `mail.canned.response` | `mail_canned_response` | Persistent record with 4 stored columns. | Discuss |
| Channel Member | `discuss.channel.member` | `discuss_channel_member` | Persistent record with 13 stored columns; belongs to Discussion Channel; owns Keep the channel member history, Mail RTC session; referenced by 3 relation fields. | Discuss |
| Chatbot Message | `chatbot.message` | `chatbot_message` | Persistent record with 6 stored columns; belongs to Discussion Channel. | Live Chat |
| Chatbot Script | `chatbot.script` | `chatbot_script` | Persistent record with 4 stored columns; belongs to Contact; owns Chatbot Script Step; referenced by 5 relation fields. | Live Chat |
| Chatbot Script Answer | `chatbot.script.answer` | `chatbot_script_answer` | Persistent record with 4 stored columns; belongs to Chatbot Script Step; referenced by 2 relation fields. | Live Chat |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_step` | Persistent record with 5 stored columns; belongs to Chatbot Script; owns Chatbot Script Answer; referenced by 3 relation fields. | Live Chat |
| Communication Bus | `bus.bus` | `bus_bus` | Persistent record with 2 stored columns. | Instant Messaging Bus |
| Digest | `digest.digest` | `digest_digest` | Persistent record with 18 stored columns; lifecycle states Activated, Deactivated; company scoped; referenced by 1 relation field. | key performance indicator Digests |
| Digest Tips | `digest.tip` | `digest_tip` | Persistent record with 4 stored columns. | key performance indicator Digests |
| Discussion Channel | `discuss.channel` | `discuss_channel` | Persistent record with 32 stored columns; owns Calendar Event, Channel Member, Chatbot Message and 6 further collections; states of `livechat_status`: In progress, Waiting for customer, Looking for help; referenced by 14 relation fields. | Discuss |
| Document Followers | `mail.followers` | `mail_followers` | Persistent record with 3 stored columns; belongs to Contact. | Discuss |
| Email Aliases | `mail.alias` | `mail_alias` | Persistent record with 12 stored columns; belongs to Models; states of `alias_status`: Not Tested, Valid, Invalid; referenced by 2 relation fields. | Discuss |
| Email Domain | `mail.alias.domain` | `mail_alias_domain` | Persistent record with 5 stored columns; owns Companies; referenced by 8 relation fields. | Discuss |
| Email Templates | `mail.template` | `mail_template` | Persistent record with 21 stored columns; referenced by 30 relation fields. | Discuss |
| Guest | `mail.guest` | `mail_guest` | Persistent record with 6 stored columns; owns User/Guest Presence; referenced by 7 relation fields. | Discuss |
| ICE Server | `mail.ice.server` | `mail_ice_server` | Persistent record with 4 stored columns. | Discuss |
| Incoming Mail Server | `fetchmail.server` | `fetchmail_server` | Persistent record with 24 stored columns; owns Outgoing Mails; lifecycle states Not Confirmed, Confirmed; referenced by 1 relation field. | Discuss |
| Keep the call history | `discuss.call.history` | `discuss_call_history` | Persistent record with 4 stored columns; belongs to Discussion Channel; referenced by 1 relation field. | Discuss |
| Keep the channel member history | `im_livechat.channel.member.history` | `im_livechat_channel_member_history` | Persistent record with 13 stored columns; states of `help_status`: Help Requested, Help Provided; referenced by 5 relation fields. | Live Chat |
| Link between link previews and messages | `mail.message.link.preview` | `mail_message_link_preview` | Persistent record with 4 stored columns; belongs to Message, Store link preview data. | Discuss |
| Link text message to mailing/text message tracking models | `sms.tracker` | `sms_tracker` | Persistent record with 4 stored columns; referenced by 1 relation field. | text message gateway |
| Live Chat Conversation Tags | `im_livechat.conversation.tag` | `im_livechat_conversation_tag` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Live Chat |
| Live Chat Expertise | `im_livechat.expertise` | `im_livechat_expertise` | Persistent record with 1 stored column; referenced by 7 relation fields. | Live Chat |
| Livechat Channel | `im_livechat.channel` | `im_livechat_channel` | Persistent record with 11 stored columns; owns Discussion Channel, Livechat Channel Rules; referenced by 7 relation fields. | Live Chat |
| Livechat Channel Rules | `im_livechat.channel.rule` | `im_livechat_channel_rule` | Persistent record with 7 stored columns. | Live Chat |
| Livechat Support Channel Report | `im_livechat.report.channel` | `none, read from a stored query` | Persistent record with 0 stored columns. | Live Chat |
| Mail Blacklist | `mail.blacklist` | `mail_blacklist` | Persistent record with 3 stored columns. | Discuss |
| Mail Gateway Allowed | `mail.gateway.allowed` | `mail_gateway_allowed` | Persistent record with 2 stored columns. | Discuss |
| Mail Group | `mail.group` | `mail_group` | Persistent record with 12 stored columns; owns Mailing List Member, Mailing List Message, Mailing List black/white list; referenced by 3 relation fields. | Mail Group |
| Mail RTC session | `discuss.channel.rtc.session` | `discuss_channel_rtc_session` | Persistent record with 7 stored columns; belongs to Channel Member; referenced by 1 relation field. | Discuss |
| Mail Tracking Value | `mail.tracking.value` | `mail_tracking_value` | Persistent record with 14 stored columns; belongs to Message. | Discuss |
| Mailing List black/white list | `mail.group.moderation` | `mail_group_moderation` | Persistent record with 3 stored columns; belongs to Mail Group; states of `status`: Always Allow, Permanent Ban. | Mail Group |
| Mailing List Member | `mail.group.member` | `mail_group_member` | Persistent record with 4 stored columns; belongs to Mail Group. | Mail Group |
| Mailing List Message | `mail.group.message` | `mail_group_message` | Persistent record with 6 stored columns; belongs to Mail Group, Message; owns Mailing List Message; states of `moderation_status`: Pending Moderation, Accepted, Rejected; referenced by 2 relation fields. | Mail Group |
| Message | `mail.message` | `mail_message` | Persistent record with 25 stored columns; owns Keep the call history, Link between link previews and messages, Mail Tracking Value and 6 further collections; referenced by 19 relation fields. | Discuss |
| Message Notifications | `mail.notification` | `mail_notification` | Persistent record with 14 stored columns; belongs to Message; owns Link text message to mailing/text message tracking models; states of `notification_status`: Ready to Send, Processing, Sent, Delivered, Bounced, Exception, Cancelled; referenced by 1 relation field. | Discuss |
| Message Reaction | `mail.message.reaction` | `mail_message_reaction` | Persistent record with 4 stored columns; belongs to Message. | Discuss |
| Message subtypes | `mail.message.subtype` | `mail_message_subtype` | Persistent record with 10 stored columns; referenced by 7 relation fields. | Discuss |
| Message Translation | `mail.message.translation` | `mail_message_translation` | Persistent record with 4 stored columns; belongs to Message. | Discuss |
| Metadata for voice attachments | `discuss.voice.metadata` | `discuss_voice_metadata` | Persistent record with 1 stored column. | Discuss |
| Outgoing Mails | `mail.mail` | `mail_mail` | Persistent record with 14 stored columns; belongs to Message; owns Mailing Statistics; lifecycle states Outgoing, Sent, Received, Delivery Failed, Cancelled; referenced by 2 relation fields. | Discuss |
| Outgoing text message | `sms.sms` | `sms_sms` | Persistent record with 10 stored columns; owns Mailing Statistics; lifecycle states In Queue, Processing, Sent, Delivered, Error, Cancelled; referenced by 2 relation fields. | text message gateway |
| Partner in-app purchase | `res.partner.iap` | `res_partner_iap` | Partner In Application Purchase. | Mail Plugin |
| Push Notification Device | `mail.push.device` | `mail_push_device` | Persistent record with 4 stored columns; belongs to Contact; referenced by 1 relation field. | Discuss |
| Push Notifications | `mail.push` | `mail_push` | Persistent record with 2 stored columns; belongs to Push Notification Device. | Discuss |
| Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `res_role` | Persistent record with 1 stored column; referenced by 1 relation field. | Discuss |
| Save favorite Animated Image from Tenor application programming interface | `discuss.gif.favorite` | `discuss_gif_favorite` | Persistent record with 1 stored column. | Discuss |
| Scheduled Message | `mail.scheduled.message` | `mail_scheduled_message` | Persistent record with 10 stored columns; belongs to Contact. | Discuss |
| Scheduled Messages | `mail.message.schedule` | `mail_message_schedule` | Persistent record with 3 stored columns; belongs to Message. | Discuss |
| Snailmail Letter | `snailmail.letter` | `snailmail_letter` | Persistent record with 20 stored columns; belongs to Companies, Contact; owns Message Notifications; lifecycle states In Queue, Sent, Error, Cancelled; company scoped; referenced by 1 relation field. | Snail Mail |
| Store link preview data | `mail.link.preview` | `mail_link_preview` | Persistent record with 8 stored columns; owns Link between link previews and messages; referenced by 1 relation field. | Discuss |
| text message Templates | `sms.template` | `sms_template` | Persistent record with 7 stored columns; belongs to Models; referenced by 11 relation fields. | text message gateway |
| Twilio Number | `sms.twilio.number` | `sms_twilio_number` | Persistent record with 4 stored columns; belongs to Companies, Country; company scoped. | Twilio text message |
| User Settings Volumes | `res.users.settings.volumes` | `res_users_settings_volumes` | Persistent record with 4 stored columns; belongs to User Settings. | Discuss |
| User/Guest Presence | `mail.presence` | `mail_presence` | Persistent record with 5 stored columns; states of `status`: Online, Away, Offline. | Discuss |

#### Interactive assistant entities (15)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Activity schedule plan Wizard | `mail.activity.schedule` | `mail_activity_schedule` | Interactive assistant with 11 stored columns; owns Mail Activity Schedule Line; company scoped; referenced by 1 relation field. | Discuss |
| Email composition wizard | `mail.compose.message` | `mail_compose_message` | Interactive assistant with 35 stored columns. | Discuss |
| Email Template Preview | `mail.template.preview` | `mail_template_preview` | Interactive assistant with 3 stored columns; belongs to Email Templates. | Discuss |
| Followers edit wizard | `mail.followers.edit` | `mail_followers_edit` | Interactive assistant with 5 stored columns. | Discuss |
| Mail Activity Schedule Line | `mail.activity.schedule.line` | `mail_activity_schedule_line` | Interactive assistant with 4 stored columns; belongs to Activity schedule plan Wizard. | Discuss |
| Mail Template Reset | `mail.template.reset` | `mail_template_reset` | Interactive assistant with 0 stored columns. | Discuss |
| Reject Group Message | `mail.group.message.reject` | `mail_group_message_reject` | Interactive assistant with 4 stored columns; belongs to Mailing List Message. | Mail Group |
| Remove email from blacklist wizard | `mail.blacklist.remove` | `mail_blacklist_remove` | Interactive assistant with 2 stored columns. | Discuss |
| Send text message Wizard | `sms.composer` | `sms_composer` | Interactive assistant with 15 stored columns. | text message gateway |
| text message Account Registration Phone Number Wizard | `sms.account.phone` | `sms_account_phone` | Interactive assistant with 2 stored columns; belongs to in-app purchase Account. | text message gateway |
| text message Account Sender Name Wizard | `sms.account.sender` | `sms_account_sender` | Interactive assistant with 2 stored columns; belongs to in-app purchase Account. | text message gateway |
| text message Account Verification Code Wizard | `sms.account.code` | `sms_account_code` | Interactive assistant with 2 stored columns; belongs to in-app purchase Account. | text message gateway |
| text message Template Preview | `sms.template.preview` | `sms_template_preview` | Interactive assistant with 3 stored columns; belongs to text message Templates. | text message gateway |
| text message Template Reset | `sms.template.reset` | `sms_template_reset` | Interactive assistant with 0 stored columns. | text message gateway |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `sms_twilio_account_manage` | Interactive assistant with 2 stored columns; belongs to Companies; company scoped. | Twilio text message |

#### Shared behaviour entities (14)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Activity Mixin | `mail.activity.mixin` | `none` | Shared behaviour merged into 60 entities. | Discuss |
| Can send messages via bus.bus | `bus.listener.mixin` | `none` | Shared behaviour merged into 13 entities. | Instant Messaging Bus |
| Email Aliases Mixin | `mail.alias.mixin` | `none` | Shared behaviour merged into 5 entities. | Discuss |
| Email Aliases Mixin (light) | `mail.alias.mixin.optional` | `none` | Shared behaviour merged into 2 entities. | Discuss |
| Email Carbon Copy management | `mail.thread.cc` | `none` | Shared behaviour merged into 5 entities. | Discuss |
| Email Thread | `mail.thread` | `none` | Shared behaviour merged into 82 entities. | Discuss |
| Mail Blacklist mixin | `mail.thread.blacklist` | `none` | Shared behaviour merged into 4 entities. | Discuss |
| Mail Bot | `mail.bot` | `none` | Shared behaviour definition reused through composition. | The system bot |
| Mail Composer Mixin | `mail.composer.mixin` | `none` | Shared behaviour merged into 8 entities. | Discuss |
| Mail Main Attachment management | `mail.thread.main.attachment` | `none` | Shared behaviour merged into 8 entities. | Discuss |
| Mail Render Mixin | `mail.render.mixin` | `none` | Shared behaviour merged into 5 entities. | Discuss |
| Mixin to compute the time a record has spent in each value a many2one field can take | `mail.tracking.duration.mixin` | `none` | Shared behaviour merged into 4 entities. | Discuss |
| Publisher Warranty Contract | `publisher_warranty.contract` | `none` | Shared behaviour definition reused through composition. | Discuss |
| Template Reset Mixin | `template.reset.mixin` | `none` | Shared behaviour merged into 2 entities. | Discuss |

#### Relationships (297)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Activity | `mail.activity` | `activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `restrict` | stored |
| Activity | `mail.activity` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Activity | `mail.activity` | `calendar_event_id` | link to one record | Calendar Event | `calendar.event` | `n : 0..1` | `cascade` | stored |
| Activity | `mail.activity` | `previous_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `not declared` | stored |
| Activity | `mail.activity` | `recommended_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `not declared` | stored |
| Activity | `mail.activity` | `request_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Activity | `mail.activity` | `res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Activity | `mail.activity` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `cascade` | stored |
| Activity Mixin | `mail.activity.mixin` | `activity_calendar_event_id` | link to one record | Calendar Event | `calendar.event` | `n : 0..1` | `not declared` | derived |
| Activity Mixin | `mail.activity.mixin` | `activity_ids` | list of records | Activity | `mail.activity` | `1 : 0..n` | `mirror of the target column` | derived |
| Activity Mixin | `mail.activity.mixin` | `activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `not declared` | derived |
| Activity Mixin | `mail.activity.mixin` | `activity_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Activity Plan | `mail.activity.plan` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Activity Plan | `mail.activity.plan` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `cascade` | stored |
| Activity Plan | `mail.activity.plan` | `res_model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Activity Plan | `mail.activity.plan` | `template_ids` | list of records | Activity plan template | `mail.activity.plan.template` | `1 : 0..n` | `mirror of the target column` | derived |
| Activity Type | `mail.activity.type` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Activity Type | `mail.activity.type` | `default_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Activity Type | `mail.activity.type` | `mail_template_ids` | list on both sides | Email Templates | `mail.template` | `0..n : 0..n` | `not declared` | stored |
| Activity Type | `mail.activity.type` | `previous_type_ids` | list on both sides | Activity Type | `mail.activity.type` | `0..n : 0..n` | `not declared` | stored |
| Activity Type | `mail.activity.type` | `suggested_next_type_ids` | list on both sides | Activity Type | `mail.activity.type` | `0..n : 0..n` | `not declared` | stored |
| Activity Type | `mail.activity.type` | `triggered_next_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `restrict` | stored |
| Activity plan template | `mail.activity.plan.template` | `activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 1` | `restrict` | stored |
| Activity plan template | `mail.activity.plan.template` | `next_activity_ids` | list on both sides | Activity Type | `mail.activity.type` | `0..n : 0..n` | `not declared` | stored |
| Activity plan template | `mail.activity.plan.template` | `plan_id` | link to one record | Activity Plan | `mail.activity.plan` | `n : 1` | `cascade` | stored |
| Activity plan template | `mail.activity.plan.template` | `responsible_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `set null` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `activity_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Activity schedule plan Wizard | `mail.activity.schedule` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | derived |
| Activity schedule plan Wizard | `mail.activity.schedule` | `plan_available_ids` | list on both sides | Activity Plan | `mail.activity.plan` | `0..n : 0..n` | `not declared` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `plan_id` | link to one record | Activity Plan | `mail.activity.plan` | `n : 0..1` | `not declared` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `plan_on_demand_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Activity schedule plan Wizard | `mail.activity.schedule` | `plan_schedule_line_ids` | list of records | Mail Activity Schedule Line | `mail.activity.schedule.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Activity schedule plan Wizard | `mail.activity.schedule` | `res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Canned Response | `mail.canned.response` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Channel Member | `discuss.channel.member` | `agent_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | derived |
| Channel Member | `discuss.channel.member` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 1` | `cascade` | stored |
| Channel Member | `discuss.channel.member` | `chatbot_script_id` | link to one record | Chatbot Script | `chatbot.script` | `n : 0..1` | `not declared` | derived |
| Channel Member | `discuss.channel.member` | `fetched_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Channel Member | `discuss.channel.member` | `guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `cascade` | stored |
| Channel Member | `discuss.channel.member` | `livechat_member_history_ids` | list of records | Keep the channel member history | `im_livechat.channel.member.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Channel Member | `discuss.channel.member` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Channel Member | `discuss.channel.member` | `rtc_inviting_session_id` | link to one record | Mail RTC session | `discuss.channel.rtc.session` | `n : 0..1` | `not declared` | stored |
| Channel Member | `discuss.channel.member` | `rtc_session_ids` | list of records | Mail RTC session | `discuss.channel.rtc.session` | `1 : 0..n` | `mirror of the target column` | derived |
| Channel Member | `discuss.channel.member` | `seen_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Chatbot Message | `chatbot.message` | `discuss_channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 1` | `cascade` | stored |
| Chatbot Message | `chatbot.message` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Chatbot Message | `chatbot.message` | `script_step_id` | link to one record | Chatbot Script Step | `chatbot.script.step` | `n : 0..1` | `not declared` | stored |
| Chatbot Message | `chatbot.message` | `user_script_answer_id` | link to one record | Chatbot Script Answer | `chatbot.script.answer` | `n : 0..1` | `set null` | stored |
| Chatbot Script | `chatbot.script` | `operator_partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `restrict` | stored |
| Chatbot Script | `chatbot.script` | `script_step_ids` | list of records | Chatbot Script Step | `chatbot.script.step` | `1 : 0..n` | `mirror of the target column` | derived |
| Chatbot Script Answer | `chatbot.script.answer` | `script_step_id` | link to one record | Chatbot Script Step | `chatbot.script.step` | `n : 1` | `cascade` | stored |
| Chatbot Script Step | `chatbot.script.step` | `answer_ids` | list of records | Chatbot Script Answer | `chatbot.script.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Chatbot Script Step | `chatbot.script.step` | `chatbot_script_id` | link to one record | Chatbot Script | `chatbot.script` | `n : 1` | `cascade` | stored |
| Chatbot Script Step | `chatbot.script.step` | `crm_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Chatbot Script Step | `chatbot.script.step` | `operator_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | stored |
| Chatbot Script Step | `chatbot.script.step` | `triggering_answer_ids` | list on both sides | Chatbot Script Answer | `chatbot.script.answer` | `0..n : 0..n` | `not declared` | stored |
| Digest | `digest.digest` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Digest | `digest.digest` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Digest Tips | `digest.tip` | `group_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Digest Tips | `digest.tip` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `calendar_event_ids` | list of records | Calendar Event | `calendar.event` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `call_history_ids` | list of records | Keep the call history | `discuss.call.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `channel_member_ids` | list of records | Channel Member | `discuss.channel.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `channel_name_member_ids` | list of records | Channel Member | `discuss.channel.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `channel_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Discussion Channel | `discuss.channel` | `chatbot_current_step_id` | link to one record | Chatbot Script Step | `chatbot.script.step` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `chatbot_message_ids` | list of records | Chatbot Message | `chatbot.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `from_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `group_public_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `invited_member_ids` | list of records | Channel Member | `discuss.channel.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `livechat_agent_history_ids` | list of records | Keep the channel member history | `im_livechat.channel.member.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `livechat_agent_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_agent_providing_help_history` | link to one record | Keep the channel member history | `im_livechat.channel.member.history` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_agent_requesting_help_history` | link to one record | Keep the channel member history | `im_livechat.channel.member.history` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_bot_history_ids` | list of records | Keep the channel member history | `im_livechat.channel.member.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `livechat_bot_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_channel_member_history_ids` | list of records | Keep the channel member history | `im_livechat.channel.member.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `livechat_conversation_tag_ids` | list on both sides | Live Chat Conversation Tags | `im_livechat.conversation.tag` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_customer_guest_ids` | list on both sides | Guest | `mail.guest` | `0..n : 0..n` | `not declared` | derived |
| Discussion Channel | `discuss.channel` | `livechat_customer_history_ids` | list of records | Keep the channel member history | `im_livechat.channel.member.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `livechat_customer_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_operator_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `livechat_visitor_id` | link to one record | Website Visitor | `website.visitor` | `n : 0..1` | `not declared` | stored |
| Discussion Channel | `discuss.channel` | `parent_channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `cascade` | stored |
| Discussion Channel | `discuss.channel` | `pinned_message_ids` | list of records | Message | `mail.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `rtc_session_ids` | list of records | Mail RTC session | `discuss.channel.rtc.session` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `self_member_id` | link to one record | Channel Member | `discuss.channel.member` | `n : 0..1` | `not declared` | derived |
| Discussion Channel | `discuss.channel` | `sub_channel_ids` | list of records | Discussion Channel | `discuss.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Discussion Channel | `discuss.channel` | `subscription_department_ids` | list on both sides | Department | `hr.department` | `0..n : 0..n` | `not declared` | stored |
| Document Followers | `mail.followers` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Document Followers | `mail.followers` | `subtype_ids` | list on both sides | Message subtypes | `mail.message.subtype` | `0..n : 0..n` | `not declared` | stored |
| Email Aliases | `mail.alias` | `alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `restrict` | stored |
| Email Aliases | `mail.alias` | `alias_model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Email Aliases | `mail.alias` | `alias_parent_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | stored |
| Email Aliases Mixin (light) | `mail.alias.mixin.optional` | `alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | derived |
| Email Aliases Mixin (light) | `mail.alias.mixin.optional` | `alias_id` | link to one record | Email Aliases | `mail.alias` | `n : 0..1` | `restrict` | derived |
| Email Domain | `mail.alias.domain` | `company_ids` | list of records | Companies | `res.company` | `1 : 0..n` | `mirror of the target column` | derived |
| Email Template Preview | `mail.template.preview` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | derived |
| Email Template Preview | `mail.template.preview` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 1` | `not declared` | stored |
| Email Template Preview | `mail.template.preview` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | derived |
| Email Template Preview | `mail.template.preview` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Email Templates | `mail.template` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Email Templates | `mail.template` | `mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Email Templates | `mail.template` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Email Templates | `mail.template` | `ref_ir_act_window` | link to one record | Action Window | `ir.actions.act_window` | `n : 0..1` | `not declared` | stored |
| Email Templates | `mail.template` | `report_template_ids` | list on both sides | Report Action | `ir.actions.report` | `0..n : 0..n` | `not declared` | stored |
| Email Templates | `mail.template` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Email Thread | `mail.thread` | `message_follower_ids` | list of records | Document Followers | `mail.followers` | `1 : 0..n` | `mirror of the target column` | derived |
| Email Thread | `mail.thread` | `message_ids` | list of records | Message | `mail.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Email Thread | `mail.thread` | `message_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Email Thread | `mail.thread` | `rating_ids` | list of records | Rating | `rating.rating` | `1 : 0..n` | `mirror of the target column` | derived |
| Email Thread | `mail.thread` | `website_message_ids` | list of records | Message | `mail.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Email composition wizard | `mail.compose.message` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `set null` | stored |
| Email composition wizard | `mail.compose.message` | `mail_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `set null` | stored |
| Email composition wizard | `mail.compose.message` | `mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `mailing_list_ids` | list on both sides | Mailing List | `mailing.list` | `0..n : 0..n` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `cascade` | stored |
| Email composition wizard | `mail.compose.message` | `parent_id` | link to one record | Message | `mail.message` | `n : 0..1` | `set null` | stored |
| Email composition wizard | `mail.compose.message` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `record_alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `record_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `res_domain_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Email composition wizard | `mail.compose.message` | `subtype_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `set null` | stored |
| Email composition wizard | `mail.compose.message` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Followers edit wizard | `mail.followers.edit` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Guest | `mail.guest` | `channel_ids` | list on both sides | Discussion Channel | `discuss.channel` | `0..n : 0..n` | `not declared` | stored |
| Guest | `mail.guest` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Guest | `mail.guest` | `presence_ids` | list of records | User/Guest Presence | `mail.presence` | `1 : 0..n` | `mirror of the target column` | derived |
| Incoming Mail Server | `fetchmail.server` | `message_ids` | list of records | Outgoing Mails | `mail.mail` | `1 : 0..n` | `mirror of the target column` | derived |
| Incoming Mail Server | `fetchmail.server` | `object_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | stored |
| Keep the call history | `discuss.call.history` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 1` | `cascade` | stored |
| Keep the call history | `discuss.call.history` | `livechat_participant_history_ids` | list on both sides | Keep the channel member history | `im_livechat.channel.member.history` | `0..n : 0..n` | `not declared` | stored |
| Keep the call history | `discuss.call.history` | `start_call_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `agent_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `call_history_ids` | list on both sides | Keep the call history | `discuss.call.history` | `0..n : 0..n` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `cascade` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `chatbot_script_id` | link to one record | Chatbot Script | `chatbot.script` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `conversation_tag_ids` | list on both sides | Live Chat Conversation Tags | `im_livechat.conversation.tag` | `0..n : 0..n` | `not declared` | derived |
| Keep the channel member history | `im_livechat.channel.member.history` | `guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `member_id` | link to one record | Channel Member | `discuss.channel.member` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `rating_id` | link to one record | Rating | `rating.rating` | `n : 0..1` | `not declared` | stored |
| Keep the channel member history | `im_livechat.channel.member.history` | `session_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Keep the channel member history | `im_livechat.channel.member.history` | `session_livechat_channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | derived |
| Link between link previews and messages | `mail.message.link.preview` | `link_preview_id` | link to one record | Store link preview data | `mail.link.preview` | `n : 1` | `cascade` | stored |
| Link between link previews and messages | `mail.message.link.preview` | `message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Link text message to mailing/text message tracking models | `sms.tracker` | `mail_notification_id` | link to one record | Message Notifications | `mail.notification` | `n : 0..1` | `cascade` | stored |
| Link text message to mailing/text message tracking models | `sms.tracker` | `mailing_trace_id` | link to one record | Mailing Statistics | `mailing.trace` | `n : 0..1` | `cascade` | stored |
| Live Chat Conversation Tags | `im_livechat.conversation.tag` | `conversation_ids` | list on both sides | Discussion Channel | `discuss.channel` | `0..n : 0..n` | `not declared` | stored |
| Live Chat Expertise | `im_livechat.expertise` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Livechat Channel | `im_livechat.channel` | `available_operator_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Livechat Channel | `im_livechat.channel` | `channel_ids` | list of records | Discussion Channel | `discuss.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Livechat Channel | `im_livechat.channel` | `rule_ids` | list of records | Livechat Channel Rules | `im_livechat.channel.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Livechat Channel | `im_livechat.channel` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Livechat Channel Rules | `im_livechat.channel.rule` | `channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | stored |
| Livechat Channel Rules | `im_livechat.channel.rule` | `chatbot_script_id` | link to one record | Chatbot Script | `chatbot.script` | `n : 0..1` | `not declared` | stored |
| Livechat Channel Rules | `im_livechat.channel.rule` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Livechat Support Channel Report | `im_livechat.report.channel` | `agent_providing_help_history` | link to one record | Keep the channel member history | `im_livechat.channel.member.history` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `agent_requesting_help_history` | link to one record | Keep the channel member history | `im_livechat.channel.member.history` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `chatbot_script_id` | link to one record | Chatbot Script | `chatbot.script` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `conversation_tag_ids` | list on both sides | Live Chat Conversation Tags | `im_livechat.conversation.tag` | `0..n : 0..n` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `livechat_channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `session_expertise_ids` | list on both sides | Live Chat Expertise | `im_livechat.expertise` | `0..n : 0..n` | `not declared` | derived |
| Livechat Support Channel Report | `im_livechat.report.channel` | `visitor_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Mail Activity Schedule Line | `mail.activity.schedule.line` | `activity_schedule_id` | link to one record | Activity schedule plan Wizard | `mail.activity.schedule` | `n : 1` | `cascade` | stored |
| Mail Activity Schedule Line | `mail.activity.schedule.line` | `responsible_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Mail Blacklist | `mail.blacklist` | `opt_out_reason_id` | link to one record | Mailing Subscription Reason | `mailing.subscription.optout` | `n : 0..1` | `restrict` | stored |
| Mail Composer Mixin | `mail.composer.mixin` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | derived |
| Mail Group | `mail.group` | `access_group_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Mail Group | `mail.group` | `mail_group_message_ids` | list of records | Mailing List Message | `mail.group.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Group | `mail.group` | `member_ids` | list of records | Mailing List Member | `mail.group.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Group | `mail.group` | `member_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Mail Group | `mail.group` | `moderation_rule_ids` | list of records | Mailing List black/white list | `mail.group.moderation` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Group | `mail.group` | `moderator_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Mail Main Attachment management | `mail.thread.main.attachment` | `message_main_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Mail RTC session | `discuss.channel.rtc.session` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | stored |
| Mail RTC session | `discuss.channel.rtc.session` | `channel_member_id` | link to one record | Channel Member | `discuss.channel.member` | `n : 1` | `cascade` | stored |
| Mail RTC session | `discuss.channel.rtc.session` | `guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `not declared` | derived |
| Mail RTC session | `discuss.channel.rtc.session` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Mail Template Reset | `mail.template.reset` | `template_ids` | list on both sides | Email Templates | `mail.template` | `0..n : 0..n` | `not declared` | stored |
| Mail Tracking Value | `mail.tracking.value` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `set null` | stored |
| Mail Tracking Value | `mail.tracking.value` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `set null` | stored |
| Mail Tracking Value | `mail.tracking.value` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Mailing List Member | `mail.group.member` | `mail_group_id` | link to one record | Mail Group | `mail.group` | `n : 1` | `cascade` | stored |
| Mailing List Member | `mail.group.member` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Mailing List Message | `mail.group.message` | `group_message_child_ids` | list of records | Mailing List Message | `mail.group.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Mailing List Message | `mail.group.message` | `group_message_parent_id` | link to one record | Mailing List Message | `mail.group.message` | `n : 0..1` | `not declared` | stored |
| Mailing List Message | `mail.group.message` | `mail_group_id` | link to one record | Mail Group | `mail.group` | `n : 1` | `cascade` | stored |
| Mailing List Message | `mail.group.message` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Mailing List Message | `mail.group.message` | `moderator_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Mailing List black/white list | `mail.group.moderation` | `mail_group_id` | link to one record | Mail Group | `mail.group` | `n : 1` | `cascade` | stored |
| Message | `mail.message` | `account_audit_log_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `account_audit_log_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `account_audit_log_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `account_audit_log_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `account_audit_log_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Message | `mail.message` | `author_guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `not declared` | stored |
| Message | `mail.message` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `call_history_ids` | list of records | Keep the call history | `discuss.call.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `child_ids` | list of records | Message | `mail.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `letter_ids` | list of records | Snailmail Letter | `snailmail.letter` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `linked_message_ids` | list on both sides | Message | `mail.message` | `0..n : 0..n` | `not declared` | derived |
| Message | `mail.message` | `mail_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `mail_ids` | list of records | Outgoing Mails | `mail.mail` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Message | `mail.message` | `message_link_preview_ids` | list of records | Link between link previews and messages | `mail.message.link.preview` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `notification_ids` | list of records | Message Notifications | `mail.notification` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `notified_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Message | `mail.message` | `parent_id` | link to one record | Message | `mail.message` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Message | `mail.message` | `rating_id` | link to one record | Rating | `rating.rating` | `n : 0..1` | `not declared` | derived |
| Message | `mail.message` | `rating_ids` | list of records | Rating | `rating.rating` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `reaction_ids` | list of records | Message Reaction | `mail.message.reaction` | `1 : 0..n` | `mirror of the target column` | derived |
| Message | `mail.message` | `record_alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `record_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `starred_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Message | `mail.message` | `subtype_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `set null` | stored |
| Message | `mail.message` | `tracking_value_ids` | list of records | Mail Tracking Value | `mail.tracking.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Message Notifications | `mail.notification` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `set null` | stored |
| Message Notifications | `mail.notification` | `letter_id` | link to one record | Snailmail Letter | `snailmail.letter` | `n : 0..1` | `cascade` | stored |
| Message Notifications | `mail.notification` | `mail_mail_id` | link to one record | Outgoing Mails | `mail.mail` | `n : 0..1` | `not declared` | stored |
| Message Notifications | `mail.notification` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Message Notifications | `mail.notification` | `res_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Message Notifications | `mail.notification` | `sms_id` | link to one record | Outgoing text message | `sms.sms` | `n : 0..1` | `not declared` | derived |
| Message Notifications | `mail.notification` | `sms_tracker_ids` | list of records | Link text message to mailing/text message tracking models | `sms.tracker` | `1 : 0..n` | `mirror of the target column` | derived |
| Message Reaction | `mail.message.reaction` | `guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `cascade` | stored |
| Message Reaction | `mail.message.reaction` | `message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Message Reaction | `mail.message.reaction` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Message Translation | `mail.message.translation` | `message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Message subtypes | `mail.message.subtype` | `parent_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `set null` | stored |
| Metadata for voice attachments | `discuss.voice.metadata` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `cascade` | stored |
| Outgoing Mails | `mail.mail` | `fetchmail_server_id` | link to one record | Incoming Mail Server | `fetchmail.server` | `n : 0..1` | `not declared` | stored |
| Outgoing Mails | `mail.mail` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Outgoing Mails | `mail.mail` | `mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| Outgoing Mails | `mail.mail` | `mailing_trace_ids` | list of records | Mailing Statistics | `mailing.trace` | `1 : 0..n` | `mirror of the target column` | derived |
| Outgoing Mails | `mail.mail` | `recipient_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Outgoing Mails | `mail.mail` | `unrestricted_attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | derived |
| Outgoing text message | `sms.sms` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Outgoing text message | `sms.sms` | `mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| Outgoing text message | `sms.sms` | `mailing_trace_ids` | list of records | Mailing Statistics | `mailing.trace` | `1 : 0..n` | `mirror of the target column` | derived |
| Outgoing text message | `sms.sms` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Outgoing text message | `sms.sms` | `record_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `set null` | stored |
| Outgoing text message | `sms.sms` | `sms_tracker_id` | link to one record | Link text message to mailing/text message tracking models | `sms.tracker` | `n : 0..1` | `not declared` | derived |
| Partner in-app purchase | `res.partner.iap` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Push Notification Device | `mail.push.device` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Push Notifications | `mail.push` | `mail_push_device_id` | link to one record | Push Notification Device | `mail.push.device` | `n : 1` | `cascade` | stored |
| Reject Group Message | `mail.group.message.reject` | `mail_group_message_id` | link to one record | Mailing List Message | `mail.group.message` | `n : 1` | `not declared` | stored |
| Represents a role in the system used to categorize users. Each role has a unique name and can be associated with multiple users. Roles can be mentioned in messages to notify all associated users. | `res.role` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Scheduled Message | `mail.scheduled.message` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Scheduled Message | `mail.scheduled.message` | `author_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Scheduled Message | `mail.scheduled.message` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Scheduled Messages | `mail.message.schedule` | `mail_message_id` | link to one record | Message | `mail.message` | `n : 1` | `cascade` | stored |
| Send text message Wizard | `sms.composer` | `mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| Send text message Wizard | `sms.composer` | `template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Send text message Wizard | `sms.composer` | `utm_campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `set null` | stored |
| Snailmail Letter | `snailmail.letter` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `cascade` | stored |
| Snailmail Letter | `snailmail.letter` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `notification_ids` | list of records | Message Notifications | `mail.notification` | `1 : 0..n` | `mirror of the target column` | derived |
| Snailmail Letter | `snailmail.letter` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `report_template` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Snailmail Letter | `snailmail.letter` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Store link preview data | `mail.link.preview` | `message_link_preview_ids` | list of records | Link between link previews and messages | `mail.message.link.preview` | `1 : 0..n` | `mirror of the target column` | derived |
| Twilio Number | `sms.twilio.number` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `cascade` | stored |
| Twilio Number | `sms.twilio.number` | `country_id` | link to one record | Country | `res.country` | `n : 1` | `not declared` | stored |
| User Settings Volumes | `res.users.settings.volumes` | `guest_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| User Settings Volumes | `res.users.settings.volumes` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| User Settings Volumes | `res.users.settings.volumes` | `user_setting_id` | link to one record | User Settings | `res.users.settings` | `n : 1` | `cascade` | stored |
| User/Guest Presence | `mail.presence` | `guest_id` | link to one record | Guest | `mail.guest` | `n : 0..1` | `cascade` | stored |
| User/Guest Presence | `mail.presence` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `cascade` | stored |
| text message Account Registration Phone Number Wizard | `sms.account.phone` | `account_id` | link to one record | in-app purchase Account | `iap.account` | `n : 1` | `not declared` | stored |
| text message Account Sender Name Wizard | `sms.account.sender` | `account_id` | link to one record | in-app purchase Account | `iap.account` | `n : 1` | `not declared` | stored |
| text message Account Verification Code Wizard | `sms.account.code` | `account_id` | link to one record | in-app purchase Account | `iap.account` | `n : 1` | `not declared` | stored |
| text message Template Preview | `sms.template.preview` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `not declared` | derived |
| text message Template Preview | `sms.template.preview` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 1` | `cascade` | stored |
| text message Template Reset | `sms.template.reset` | `template_ids` | list on both sides | text message Templates | `sms.template` | `0..n : 0..n` | `not declared` | stored |
| text message Templates | `sms.template` | `model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| text message Templates | `sms.template` | `sidebar_action_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 0..1` | `not declared` | stored |
| text message Twilio Connection Wizard | `sms.twilio.account.manage` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |

### 3.7 Multi-Currency

Currencies and their dated rates, the rounding factor, currency conversion and the foreign-currency copy of every ledger amount.

Specified in [`../domains/multi-currency/`](../domains/multi-currency/).

#### Persistent entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Currency | `res.currency` | `res_currency` | Persistent record with 13 stored columns; owns Currency Rate; referenced by 97 relation fields. | Base |
| Currency Rate | `res.currency.rate` | `res_currency_rate` | Persistent record with 4 stored columns; belongs to Currency; company scoped. | Base |

#### Relationships (3)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Currency | `res.currency` | `rate_ids` | list of records | Currency Rate | `res.currency.rate` | `1 : 0..n` | `mirror of the target column` | derived |
| Currency Rate | `res.currency.rate` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Currency Rate | `res.currency.rate` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `cascade` | stored |

### 3.8 Spreadsheets and Dashboards

Spreadsheet documents, spreadsheet revisions, dashboards and dashboard groups.

Specified in [`../domains/spreadsheets-and-dashboards/`](../domains/spreadsheets-and-dashboards/).

#### Persistent entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Copy of a shared dashboard | `spreadsheet.dashboard.share` | `spreadsheet_dashboard_share` | Persistent record with 2 stored columns; belongs to Spreadsheet Dashboard. | Spreadsheet dashboard |
| Group of dashboards | `spreadsheet.dashboard.group` | `spreadsheet_dashboard_group` | Persistent record with 2 stored columns; owns Spreadsheet Dashboard; referenced by 1 relation field. | Spreadsheet dashboard |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `spreadsheet_dashboard` | Persistent record with 5 stored columns; belongs to Group of dashboards; referenced by 1 relation field. | Spreadsheet dashboard |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Board | `board.board` | `none` | Shared behaviour definition reused through composition. | Dashboards |
| Spreadsheet mixin | `spreadsheet.mixin` | `none` | Shared behaviour merged into 2 entities. | Spreadsheet |

#### Relationships (8)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Copy of a shared dashboard | `spreadsheet.dashboard.share` | `dashboard_id` | link to one record | Spreadsheet Dashboard | `spreadsheet.dashboard` | `n : 1` | `cascade` | stored |
| Group of dashboards | `spreadsheet.dashboard.group` | `dashboard_ids` | list of records | Spreadsheet Dashboard | `spreadsheet.dashboard` | `1 : 0..n` | `mirror of the target column` | derived |
| Group of dashboards | `spreadsheet.dashboard.group` | `published_dashboard_ids` | list of records | Spreadsheet Dashboard | `spreadsheet.dashboard` | `1 : 0..n` | `mirror of the target column` | derived |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | stored |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `dashboard_group_id` | link to one record | Group of dashboards | `spreadsheet.dashboard.group` | `n : 1` | `not declared` | stored |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Spreadsheet Dashboard | `spreadsheet.dashboard` | `main_data_model_ids` | list on both sides | Models | `ir.model` | `0..n : 0..n` | `not declared` | stored |

### 3.9 Accounts Payable

Vendor bills, vendor credit notes, purchase receipts, debit notes, cheque printing, automatic bill posting and duplicate detection.

Specified in [`../domains/accounts-payable/`](../domains/accounts-payable/).

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Print Pre-numbered Checks | `print.prenumbered.checks` | `print_prenumbered_checks` | Interactive assistant with 1 stored column. | Check Printing Base |

### 3.10 Accounts Receivable

Customer invoices, credit notes, customer receipts, debit notes raised on a customer document and the refund of a received payment.

Specified in [`../domains/accounts-receivable/`](../domains/accounts-receivable/).

#### Interactive assistant entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Add Debit Note wizard | `account.debit.note` | `account_debit_note` | Interactive assistant with 5 stored columns. | Debit Notes |
| Payment Refund Wizard | `payment.refund.wizard` | `payment_refund_wizard` | Interactive assistant with 2 stored columns. | Payment - Account |

#### Relationships (3)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Add Debit Note wizard | `account.debit.note` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Add Debit Note wizard | `account.debit.note` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Payment Refund Wizard | `payment.refund.wizard` | `payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |

### 3.11 Analytic Accounting

Analytic plans, analytic accounts, analytic distribution models, analytic lines and the applicability rules that decide when a distribution is required.

Specified in [`../domains/analytic-accounting/`](../domains/analytic-accounting/).

#### Persistent entities (5)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Analytic Account | `account.analytic.account` | `account_analytic_account` | Persistent record with 7 stored columns; belongs to Analytic Plans; owns Analytic Line, Project; company scoped; referenced by 6 relation fields. | Analytic Accounting |
| Analytic Distribution Model | `account.analytic.distribution.model` | `account_analytic_distribution_model` | Persistent record with 8 stored columns; company scoped. | Analytic Accounting |
| Analytic Line | `account.analytic.line` | `account_analytic_line` | Persistent record with 30 stored columns; belongs to Companies; carries the state field `sale_order_state`; company scoped; referenced by 3 relation fields. | Analytic Accounting |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `account_analytic_applicability` | Persistent record with 6 stored columns; company scoped. | Analytic Accounting |
| Analytic Plans | `account.analytic.plan` | `account_analytic_plan` | Persistent record with 8 stored columns; owns Analytic Account, Analytic Plan's Applicabilities, Analytic Plans; referenced by 5 relation fields. | Analytic Accounting |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Analytic Mixin | `analytic.mixin` | `none` | Shared behaviour merged into 10 entities. | Analytic Accounting |
| Analytic Plan Fields | `analytic.plan.fields.mixin` | `none` | Shared behaviour merged into 2 entities. | Analytic Accounting |

#### Relationships (47)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Analytic Account | `account.analytic.account` | `bom_ids` | list on both sides | Bill of Material | `mrp.bom` | `0..n : 0..n` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `line_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Analytic Account | `account.analytic.account` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `plan_id` | link to one record | Analytic Plans | `account.analytic.plan` | `n : 1` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `project_ids` | list of records | Project | `project.project` | `1 : 0..n` | `mirror of the target column` | derived |
| Analytic Account | `account.analytic.account` | `root_plan_id` | link to one record | Analytic Plans | `account.analytic.plan` | `n : 0..1` | `not declared` | stored |
| Analytic Account | `account.analytic.account` | `workcenter_ids` | list on both sides | Work Center | `mrp.workcenter` | `0..n : 0..n` | `not declared` | stored |
| Analytic Distribution Model | `account.analytic.distribution.model` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `cascade` | stored |
| Analytic Distribution Model | `account.analytic.distribution.model` | `partner_category_id` | link to one record | Partner Tags | `res.partner.category` | `n : 0..1` | `cascade` | stored |
| Analytic Distribution Model | `account.analytic.distribution.model` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `cascade` | stored |
| Analytic Distribution Model | `account.analytic.distribution.model` | `product_categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `cascade` | stored |
| Analytic Distribution Model | `account.analytic.distribution.model` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Analytic Line | `account.analytic.line` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Analytic Line | `account.analytic.line` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `encoding_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Analytic Line | `account.analytic.line` | `general_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Analytic Line | `account.analytic.line` | `global_leave_id` | link to one record | Resource Time Off Detail | `resource.calendar.leaves` | `n : 0..1` | `cascade` | stored |
| Analytic Line | `account.analytic.line` | `holiday_id` | link to one record | Time Off | `hr.leave` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `message_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Analytic Line | `account.analytic.line` | `milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | derived |
| Analytic Line | `account.analytic.line` | `move_line_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `cascade` | stored |
| Analytic Line | `account.analytic.line` | `parent_task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `so_line` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `timesheet_invoice_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Analytic Line | `account.analytic.line` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Analytic Mixin | `analytic.mixin` | `distribution_analytic_account_ids` | list on both sides | Analytic Account | `account.analytic.account` | `0..n : 0..n` | `not declared` | derived |
| Analytic Plan Fields | `analytic.plan.fields.mixin` | `account_id` | link to one record | Analytic Account | `account.analytic.account` | `n : 0..1` | `restrict` | derived |
| Analytic Plan Fields | `analytic.plan.fields.mixin` | `auto_account_id` | link to one record | Analytic Account | `account.analytic.account` | `n : 0..1` | `not declared` | derived |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `analytic_plan_id` | link to one record | Analytic Plans | `account.analytic.plan` | `n : 0..1` | `not declared` | stored |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Analytic Plan's Applicabilities | `account.analytic.applicability` | `product_categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Analytic Plans | `account.analytic.plan` | `account_ids` | list of records | Analytic Account | `account.analytic.account` | `1 : 0..n` | `mirror of the target column` | derived |
| Analytic Plans | `account.analytic.plan` | `applicability_ids` | list of records | Analytic Plan's Applicabilities | `account.analytic.applicability` | `1 : 0..n` | `mirror of the target column` | derived |
| Analytic Plans | `account.analytic.plan` | `children_ids` | list of records | Analytic Plans | `account.analytic.plan` | `1 : 0..n` | `mirror of the target column` | derived |
| Analytic Plans | `account.analytic.plan` | `parent_id` | link to one record | Analytic Plans | `account.analytic.plan` | `n : 0..1` | `cascade` | stored |
| Analytic Plans | `account.analytic.plan` | `root_id` | link to one record | Analytic Plans | `account.analytic.plan` | `n : 0..1` | `not declared` | derived |

### 3.12 Electronic Invoicing and Document Exchange

Structured electronic document formats, document import and export, network delivery, digital certificates and the interchange proxy.

Specified in [`../domains/electronic-invoicing-and-document-exchange/`](../domains/electronic-invoicing-and-document-exchange/).

#### Persistent entities (7)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `account_edi_proxy_client_user` | Persistent record with 11 stored columns; belongs to Companies, Cryptographic Keys; company scoped; referenced by 8 relation fields. | Proxy features for account_edi |
| Business Level Responses for Peppol | `account.peppol.response` | `account_peppol_response` | Persistent record with 11 stored columns; states of `peppol_state`: Pending Reception, Done, Error, Not Serviced; 2 state fields in all. | Peppol Business Response |
| Certificate | `certificate.certificate` | `certificate_certificate` | Persistent record with 13 stored columns; belongs to Companies; company scoped; referenced by 7 relation fields. | Certificate |
| Cryptographic Keys | `certificate.key` | `certificate_key` | Persistent record with 6 stored columns; belongs to Companies; company scoped; referenced by 4 relation fields. | Certificate |
| electronic data interchange format | `account.edi.format` | `account_edi_format` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Import/Export Invoices From extensible markup language/Portable Document Format |
| Electronic Document for an account.move | `account.edi.document` | `account_edi_document` | Persistent record with 6 stored columns; belongs to Journal Entry, electronic data interchange format; lifecycle states To Send, Sent, To Cancel, Cancelled. | Import/Export Invoices From extensible markup language/Portable Document Format |
| Peppol clarifications used for rejection | `account.peppol.clarification` | `account_peppol_clarification` | Persistent record with 4 stored columns; referenced by 2 relation fields. | Peppol Business Response |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Peppol Configuration Wizard | `peppol.config.wizard` | `peppol_config_wizard` | Interactive assistant with 3 stored columns; belongs to Companies; owns Peppol Service; carries the state field `account_peppol_proxy_state`; company scoped; referenced by 1 relation field. | Peppol |
| Peppol Registration | `peppol.registration` | `peppol_registration` | Interactive assistant with 2 stored columns; belongs to Companies; carries the state field `account_peppol_proxy_state`; company scoped. | Peppol |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `account_peppol_rejection_wizard` | Interactive assistant with 0 stored columns. | Peppol Business Response |
| Peppol Service | `account_peppol.service` | `account_peppol_service` | Interactive assistant with 4 stored columns. | Peppol |

#### Shared behaviour entities (18)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| A-NZ BIS Billing 3.0 | `account.edi.xml.ubl_a_nz` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Base helpers for Cross Industry Invoice | `account.edi.cii` | `none` | Shared behaviour merged into 1 entity. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Base helpers for Universal Business Language | `account.edi.ubl` | `none` | Shared behaviour merged into 3 entities. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| BIS3 DE (XRechnung) | `account.edi.xml.ubl_de` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Common functions for electronic data interchange documents: generate the data, the constraints, etc | `account.edi.common` | `none` | Shared behaviour merged into 3 entities. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| E-FFF (BE) | `account.edi.xml.ubl_efff` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Factur-x/ZUGFeRD Cross Industry Invoice 2.2.0 | `account.edi.xml.cii` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Point of Sale Order Universal Business Language 2.1 builder | `pos.edi.xml.ubl_21` | `none` | Shared behaviour merged into 1 entity. | Point of Sale Universal Business Language |
| Purchase Universal Business Language BIS Ordering 3.5 | `purchase.edi.xml.ubl_bis3` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic orders with Universal Business Language |
| Sale BIS Ordering 3.5 | `sale.edi.xml.ubl_bis3` | `none` | Shared behaviour definition reused through composition. | Import electronic orders with Universal Business Language |
| SG BIS Billing 3.0 | `account.edi.xml.ubl_sg` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| SI-Universal Business Language 2.0 (NLCIUS) | `account.edi.xml.ubl_nl` | `none` | Shared behaviour definition reused through composition. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language 2.0 | `account.edi.xml.ubl_20` | `none` | Shared behaviour merged into 3 entities. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language 2.1 | `account.edi.xml.ubl_21` | `none` | Shared behaviour merged into 8 entities. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language BIS Billing 3.0.12 | `account.edi.xml.ubl_bis3` | `none` | Shared behaviour merged into 13 entities. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language CEN-EN16931 | `account.edi.ubl_cen_en16931` | `none` | Shared behaviour merged into 1 entity. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language Peppol International Invoice | `account.edi.ubl_pint` | `none` | Shared behaviour merged into 1 entity. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |
| Universal Business Language Peppol International Invoice-European Union Layer | `account.edi.ubl_pint_eu` | `none` | Shared behaviour merged into 1 entity. | Import/Export electronic invoices with Universal Business Language/Cross Industry Invoice |

#### Relationships (21)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `private_key_id` | link to one record | Cryptographic Keys | `certificate.key` | `n : 1` | `not declared` | stored |
| Business Level Responses for Peppol | `account.peppol.response` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `cascade` | stored |
| Certificate | `certificate.certificate` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `cascade` | stored |
| Certificate | `certificate.certificate` | `issuer_cert_id` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | derived |
| Certificate | `certificate.certificate` | `private_key_id` | link to one record | Cryptographic Keys | `certificate.key` | `n : 0..1` | `not declared` | stored |
| Certificate | `certificate.certificate` | `public_key_id` | link to one record | Cryptographic Keys | `certificate.key` | `n : 0..1` | `not declared` | stored |
| Cryptographic Keys | `certificate.key` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `cascade` | stored |
| Electronic Document for an account.move | `account.edi.document` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Electronic Document for an account.move | `account.edi.document` | `edi_format_id` | link to one record | electronic data interchange format | `account.edi.format` | `n : 1` | `not declared` | stored |
| Electronic Document for an account.move | `account.edi.document` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `cascade` | stored |
| Peppol Configuration Wizard | `peppol.config.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Peppol Configuration Wizard | `peppol.config.wizard` | `service_ids` | list of records | Peppol Service | `account_peppol.service` | `1 : 0..n` | `mirror of the target column` | derived |
| Peppol Registration | `peppol.registration` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Peppol Registration | `peppol.registration` | `edi_user_id` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Peppol Registration | `peppol.registration` | `parent_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Peppol Registration | `peppol.registration` | `selected_company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `action_ids` | list on both sides | Peppol clarifications used for rejection | `account.peppol.clarification` | `0..n : 0..n` | `not declared` | stored |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Peppol Rejection wizard | `account.peppol.rejection.wizard` | `reason_ids` | list on both sides | Peppol clarifications used for rejection | `account.peppol.clarification` | `0..n : 0..n` | `not declared` | stored |
| Peppol Service | `account_peppol.service` | `wizard_id` | link to one record | Peppol Configuration Wizard | `peppol.config.wizard` | `n : 0..1` | `not declared` | stored |

### 3.13 Financial Reporting

The definition and evaluation of financial statements built on the ledger.

Specified in [`../domains/financial-reporting/`](../domains/financial-reporting/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Accounting Assert Test | `accounting.assert.test` | `accounting_assert_test` | Persistent record with 5 stored columns. | Accounting Consistency Tests |

### 3.14 Fiscal Localizations

Country packages: chart of account templates, tax definitions, fiscal positions, tax report structures, country-specific document rules, identification types and country-specific electronic invoicing.

Specified in the domain folder `../domains/fiscal-localizations/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (49)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Administrative Center Role Type | `l10n_es_edi_facturae.ac_role_type` | `l10n_es_edi_facturae_ac_role_type` | Persistent record with 2 stored columns; referenced by 1 relation field. | Spain - Facturae electronic data interchange |
| ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `l10n_ar_afip_responsibility_type` | Persistent record with 4 stored columns; referenced by 3 relation fields. | Argentina - Accounting |
| Argentinean Partner Taxes | `l10n_ar.partner.tax` | `l10n_ar_partner_tax` | Persistent record with 6 stored columns; belongs to Contact, Tax. | Argentina - Payment Withholdings |
| Brazilian city zip range | `l10n_br.zip.range` | `l10n_br_zip_range` | Persistent record with 3 stored columns; belongs to City. | Brazilian - Accounting |
| Business Level Responses for Nemhandel | `nemhandel.response` | `nemhandel_response` | Persistent record with 4 stored columns; states of `nemhandel_state`: Pending Reception, Done, Error, Not Serviced. | Nemhandel Business Response |
| CPV Code | `l10n_ro.cpv.code` | `l10n_ro_cpv_code` | Persistent record with 2 stored columns; referenced by 1 relation field. | Romania - CPV Code |
| Croatian KPD Category | `l10n_hr.kpd.category` | `l10n_hr_kpd_category` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Croatia - Electronic Invoicing |
| Croatian tax expence categories | `l10n.hr.tax.category` | `l10n_hr_tax_category` | Persistent record with 6 stored columns; referenced by 1 relation field. | Croatia - Electronic Invoicing |
| Customer Alias on Nilvera | `l10n_tr.nilvera.alias` | `l10n_tr_nilvera_alias` | Persistent record with 2 stored columns; referenced by 1 relation field. | Türkiye - Nilvera |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `l10n_it_edi_doi_declaration_of_intent` | Persistent record with 14 stored columns; belongs to Companies, Contact, Currency; owns Journal Entry, Sales Order; lifecycle states Draft, Active, Revoked, Terminated; company scoped; referenced by 2 relation fields. | Italy - Declaration of Intent |
| District | `l10n_pe.res.city.district` | `l10n_pe_res_city_district` | Persistent record with 3 stored columns; referenced by 1 relation field. | Peru - Accounting |
| Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `l10n_ro_edi_document` | Persistent record with 11 stored columns; lifecycle states Sent, Error, Validated. | Romania - Electronic Invoicing |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `l10n_id_efaktur_coretax_document` | Persistent record with 5 stored columns; belongs to Companies; owns Journal Entry; company scoped; referenced by 1 relation field. | Indonesia E-faktur (Coretax) |
| electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi.addendum` | `l10n_hr_edi_addendum` | Persistent record with 14 stored columns; belongs to Journal Entry; states of `business_document_status`: APPROVED, REJECTED, PAYMENT_FULFILLED, PAYMENT_PARTIALLY_FULLFILLED, RECEIVING_CONFIRMED, RECEIVED, None; 3 state fields in all. | Croatia - Electronic Invoicing |
| Electronic Waybill | `l10n.in.ewaybill` | `l10n_in_ewaybill` | Persistent record with 27 stored columns; lifecycle states Pending, Generated, Cancelled, Challan; company scoped; referenced by 1 relation field. | Indian - Electronic Waybill |
| Electronic Waybill Document Type | `l10n.in.ewaybill.type` | `l10n_in_ewaybill_type` | Persistent record with 6 stored columns; referenced by 1 relation field. | Indian - Electronic Waybill |
| Estimated Time of Arrival code for activity type | `l10n_eg_edi.activity.type` | `l10n_eg_edi_activity_type` | Persistent record with 2 stored columns; referenced by 1 relation field. | Egypt Electronic Invoicing |
| Estimated Time of Arrival code for the unit of measures | `l10n_eg_edi.uom.code` | `l10n_eg_edi_uom_code` | Persistent record with 2 stored columns; referenced by 1 relation field. | Egypt Electronic Invoicing |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `l10n_fr_pdp_reports_flow` | Persistent record with 15 stored columns; belongs to Companies; owns French Approved Dematerialization Platform Flow; lifecycle states x; 2 state fields in all; company scoped; referenced by 4 relation fields. | France - Electronic Invoicing (Approved Platform) |
| GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `l10n_tr_nilvera_trailer_plate` | Persistent record with 2 stored columns; referenced by 2 relation fields. | Türkiye - e-Irsaliye (e-Dispatch) |
| Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi.document` | `l10n_gr_edi_document` | Persistent record with 15 stored columns; lifecycle states Invoice sent, Invoice send failed, Expense classification ready to send, Expense classification sent, Expense classification send failed, Invoice submission pending; 2 state fields in all. | Greece - myDATA |
| Identification Types | `l10n_latam.identification.type` | `l10n_latam_identification_type` | Persistent record with 10 stored columns; referenced by 1 relation field. | LATAM Localization Base |
| Indian permanent account number Entity | `l10n_in.pan.entity` | `l10n_in_pan_entity` | Persistent record with 6 stored columns; owns Contact; referenced by 1 relation field. | Indian - Accounting |
| Indian port code | `l10n_in.port.code` | `l10n_in_port_code` | Persistent record with 3 stored columns; referenced by 1 relation field. | Indian - Accounting |
| indian section alert | `l10n_in.section.alert` | `l10n_in_section_alert` | Persistent record with 9 stored columns; owns Tax; referenced by 2 relation fields. | Indian - Accounting |
| Italian Document Type | `l10n_it.document.type` | `l10n_it_document_type` | Persistent record with 3 stored columns; referenced by 1 relation field. | Italy - Electronic Invoicing |
| KRA defined codes that justify a given tax rate / exemption | `l10n_ke.item.code` | `l10n_ke_item_code` | Persistent record with 3 stored columns; referenced by 1 relation field. | Kenya - Accounting |
| l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | `l10n_ar_earnings_scale` | Persistent record with 1 stored column; owns l10n_ar.earnings.scale.line; referenced by 2 relation fields. | Argentina - Payment Withholdings |
| l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | `l10n_ar_earnings_scale_line` | Persistent record with 5 stored columns; belongs to l10n_ar.earnings.scale. | Argentina - Payment Withholdings |
| Latam Document Type | `l10n_latam.document.type` | `l10n_latam_document_type` | Persistent record with 12 stored columns; belongs to Country; referenced by 6 relation fields. | LATAM Document |
| Malaysian Industry Classification | `l10n_my_edi.industry_classification` | `l10n_my_edi_industry_classification` | Persistent record with 2 stored columns; referenced by 1 relation field. | Malaysia - Electronic Invoicing |
| MyInvois Document | `myinvois.document` | `myinvois_document` | Persistent record with 18 stored columns; belongs to Companies, Currency; states of `myinvois_state`: Validation In Progress, Valid, Rejected, Invalid, Cancelled; company scoped; referenced by 3 relation fields. | Malaysia - Electronic Invoicing |
| PL Bank Account Verification | `l10n_pl.bank.account.verification` | `l10n_pl_bank_account_verification` | Persistent record with 8 stored columns; states of `verification_status`: Valid, Invalid, Incomplete partner, Partner not found, An error occurred during check with Government Application Programming Interface; referenced by 2 relation fields. | Poland - Accounting - Bank Account Verification |
| Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `l10n_gr_edi_preferred_classification` | Persistent record with 7 stored columns. | Greece - myDATA |
| Product categorization according to E-Faktur | `l10n_id_efaktur_coretax.product.code` | `l10n_id_efaktur_coretax_product_code` | Persistent record with 2 stored columns; referenced by 1 relation field. | Indonesia E-faktur (Coretax) |
| Record of QRIS transactions | `l10n_id.qris.transaction` | `l10n_id_qris_transaction` | Persistent record with 8 stored columns; referenced by 2 relation fields. | Indonesian - Accounting |
| Sale Closing | `account.sale.closing` | `account_sale_closing` | Persistent record with 11 stored columns; belongs to Companies; company scoped. | France - value-added tax Anti-Fraud Certification for Point of Sale (CGI 286 I-3 bis) |
| SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `l10n_vn_edi_viettel_sinvoice_symbol` | Persistent record with 2 stored columns; belongs to SInvoice template; referenced by 5 relation fields. | Vietnam - Electronic Invoicing |
| SInvoice template | `l10n_vn_edi_viettel.sinvoice.template` | `l10n_vn_edi_viettel_sinvoice_template` | Persistent record with 2 stored columns; owns SInvoice symbol; referenced by 1 relation field. | Vietnam - Electronic Invoicing |
| SRI Payment Method | `l10n_ec.sri.payment` | `l10n_ec_sri_payment` | Persistent record with 4 stored columns; referenced by 3 relation fields. | Ecuadorian Accounting |
| Tax office in Czech Republic | `l10n_cz.tax_office` | `l10n_cz_tax_office` | Persistent record with 4 stored columns; referenced by 1 relation field. | Czech - Accounting |
| Tax Office in Poland | `l10n_pl.l10n_pl_tax_office` | `l10n_pl_l10n_pl_tax_office` | Persistent record with 2 stored columns; referenced by 1 relation field. | Poland - Accounting |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `l10n_eg_edi_thumb_drive` | Persistent record with 4 stored columns; belongs to Companies, User; company scoped. | Egypt Electronic Invoicing |
| TicketBAI Document | `l10n_es_edi_tbai.document` | `l10n_es_edi_tbai_document` | Persistent record with 8 stored columns; belongs to Companies; lifecycle states To Send, Accepted, Rejected; company scoped; referenced by 3 relation fields. | Spain - TicketBAI |
| Transport Document | `l10n_it.ddt` | `l10n_it_ddt` | Persistent record with 2 stored columns; owns Journal Entry; referenced by 1 relation field. | Italy - Electronic Invoicing |
| Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `l10n_tr_nilvera_einvoice_extended_account_tax_code` | Persistent record with 4 stored columns; referenced by 2 relation fields. | Türkiye - Nilvera Electronic Invoice Extended |
| Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | `l10n_tr_nilvera_einvoice_extended_tax_office` | Persistent record with 3 stored columns; referenced by 1 relation field. | Türkiye - Nilvera Electronic Invoice Extended |
| unit of measure categorization according to E-Faktur | `l10n_id_efaktur_coretax.uom.code` | `l10n_id_efaktur_coretax_uom_code` | Persistent record with 2 stored columns; referenced by 1 relation field. | Indonesia E-faktur (Coretax) |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `l10n_es_edi_verifactu_document` | Persistent record with 9 stored columns; belongs to Companies; lifecycle states Rejected, Registered with Errors, Accepted; company scoped. | Spain - Veri*Factu |

#### Interactive assistant entities (24)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Approved Dematerialization Platform Registration | `pdp.registration` | `pdp_registration` | Interactive assistant with 3 stored columns; belongs to Companies; carries the state field `account_peppol_proxy_state`; 2 state fields in all; company scoped. | France - Electronic Invoicing (Approved Platform) |
| Approved Dematerialization Platform Response wizard | `pdp.response.wizard` | `pdp_response_wizard` | Interactive assistant with 7 stored columns; states of `status`: Paid, Cancelled, Suspended, Refused, Approved, In Hand, Completed. | France - Electronic Invoicing (Approved Platform) |
| Cancel Electronic Invoice | `l10n_in_edi.cancel` | `l10n_in_edi_cancel` | Interactive assistant with 3 stored columns; belongs to Journal Entry. | Indian - Electronic Invoicing |
| Cancel Electronic Waybill | `l10n.in.ewaybill.cancel` | `l10n_in_ewaybill_cancel` | Interactive assistant with 3 stored columns; belongs to Electronic Waybill. | Indian - Electronic Waybill |
| Compliance Letter for EXO Number | `compliance.letter.wizard` | `compliance_letter_wizard` | Interactive assistant with 1 stored column; belongs to Companies; company scoped. | Malta - Point of Sale |
| Consolidate Invoice Wizard | `myinvois.consolidate.invoice.wizard` | `myinvois_consolidate_invoice_wizard` | Interactive assistant with 3 stored columns. | Malaysia - Electronic Invoicing |
| Document Status Update Wizard | `myinvois.document.status.update.wizard` | `myinvois_document_status_update_wizard` | Interactive assistant with 3 stored columns; belongs to MyInvois Document. | Malaysia - Electronic Invoicing |
| Electronic Invoice cancellation wizard | `l10n_vn_edi_viettel.cancellation` | `l10n_vn_edi_viettel_cancellation` | Interactive assistant with 4 stored columns. | Vietnam - Electronic Invoicing |
| Exports 2307 data to a XLS file. | `l10n_ph_2307.wizard` | `l10n_ph_2307_wizard` | Interactive assistant with 0 stored columns. | Philippines - Accounting |
| Fichier Echange Informatise | `l10n_fr.fec.export.wizard` | `l10n_fr_fec_export_wizard` | Interactive assistant with 5 stored columns. | France - Accounting |
| Handles problems occurring while creating multiple quick response-invoices at once | `l10n_ch.qr_invoice.wizard` | `l10n_ch_qr_invoice_wizard` | Interactive assistant with 4 stored columns. | Switzerland - Accounting |
| Implements cancelling an ecpay invoice. | `l10n_tw_edi.invoice.cancel` | `l10n_tw_edi_invoice_cancel` | Interactive assistant with 2 stored columns; belongs to Journal Entry. | Taiwan - Electronic Invoicing |
| Implements printingan ecpay invoice. | `l10n_tw_edi.invoice.print` | `l10n_tw_edi_invoice_print` | Interactive assistant with 3 stored columns; belongs to Journal Entry. | Taiwan - Electronic Invoicing |
| MojEracun Reject Invoice Wizard | `l10n_hr_edi.mojeracun_reject_wizard` | `l10n_hr_edi_mojeracun_reject_wizard` | Interactive assistant with 3 stored columns; belongs to Journal Entry. | Croatia - Electronic Invoicing |
| Nemhandel Registration | `nemhandel.registration` | `nemhandel_registration` | Interactive assistant with 1 stored column; belongs to Companies; carries the state field `l10n_dk_nemhandel_proxy_state`; company scoped. | Denmark electronic data interchange - Nemhandel |
| Nemhandel Rejection wizard | `nemhandel.rejection.wizard` | `nemhandel_rejection_wizard` | Interactive assistant with 1 stored column. | Nemhandel Business Response |
| Payment register withholding lines | `l10n_ar.payment.register.withholding` | `l10n_ar_payment_register_withholding` | Interactive assistant with 5 stored columns; belongs to Pay, Tax. | Argentina - Payment Withholdings |
| Peppol Configuration Wizard | `pdp.config.wizard` | `pdp_config_wizard` | Interactive assistant with 1 stored column; belongs to Companies; carries the state field `account_peppol_proxy_state`; company scoped. | France - Electronic Invoicing (Approved Platform) |
| Receive Bills Wizard | `l10n_hu_edi_receive.bills.wizard` | `l10n_hu_edi_receive_bills_wizard` | Interactive assistant with 2 stored columns. | Hungary - Electronic Invoicing Receive Vendor Bills |
| Request Zakat Tax and Customs Authority one-time password | `l10n_sa_edi.otp.wizard` | `l10n_sa_edi_otp_wizard` | Interactive assistant with 3 stored columns; belongs to Journal. | Saudi Arabia - Electronic Invoicing |
| Send Approved Dematerialization Platform Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `l10n_fr_pdp_reports_send_wizard` | Interactive assistant with 2 stored columns; belongs to French Approved Dematerialization Platform Flow. | France - Electronic Invoicing (Approved Platform) |
| Tax audit export - Adóhatósági Ellenőrzési Adatszolgáltatás | `l10n_hu_edi.tax_audit_export` | `l10n_hu_edi_tax_audit_export` | Interactive assistant with 5 stored columns. | Hungary - Electronic Invoicing |
| Technical Annulment Wizard | `l10n_hu_edi.cancellation` | `l10n_hu_edi_cancellation` | Interactive assistant with 3 stored columns. | Hungary - Electronic Invoicing |
| Withhold Wizard | `l10n_in.withhold.wizard` | `l10n_in_withhold_wizard` | Interactive assistant with 7 stored columns; belongs to Journal, Tax; company scoped. | Indian - Accounting |

#### Shared behaviour entities (16)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Australia & New Zealand implementation of Peppol International (Peppol International Invoice) model for Billing | `account.edi.xml.pint_anz` | `none` | Shared behaviour definition reused through composition. | Australia & New Zealand - Universal Business Language Peppol International Invoice |
| CIUS human resources | `account.edi.xml.ubl_hr` | `none` | Shared behaviour definition reused through composition. | Croatia - Electronic Invoicing |
| CIUS RO | `account.edi.xml.ubl_ro` | `none` | Shared behaviour definition reused through composition. | Romania - Electronic Invoicing |
| Flow 10 extensible markup language Builder | `pdp.flow.10.xml.builder` | `none` | Shared behaviour definition reused through composition. | France - Electronic Invoicing (Approved Platform) |
| France Universal Business Language 2.1 Electronic Invoicing Format | `account.edi.xml.ubl_21_fr` | `none` | Shared behaviour definition reused through composition. | France - Electronic Invoicing (Approved Platform) |
| Japanese implementation of Peppol International (Peppol International Invoice) model for Billing | `account.edi.xml.pint_jp` | `none` | Shared behaviour definition reused through composition. | Japan - Universal Business Language Peppol International Invoice |
| Malaysian implementation of Peppol International (Peppol International Invoice) model for Billing | `account.edi.xml.pint_my` | `none` | Shared behaviour definition reused through composition. | Malaysia - Universal Business Language Peppol International Invoice |
| Malaysian implementation of universal business language for the MyInvois portal | `account.edi.xml.ubl_myinvois_my` | `none` | Shared behaviour definition reused through composition. | Malaysia - Electronic Invoicing |
| Public Electronic Invoicing Format 2.01 | `account.edi.xml.oioubl_201` | `none` | Shared behaviour definition reused through composition. | Denmark - Electronic Invoicing |
| Public Electronic Invoicing Format 2.1 | `account.edi.xml.oioubl_21` | `none` | Shared behaviour definition reused through composition. | Denmark electronic data interchange - Nemhandel |
| Singapore implementation of Peppol International (Peppol International Invoice) model for Billing | `account.edi.xml.pint_sg` | `none` | Shared behaviour definition reused through composition. | Singapore - Universal Business Language Peppol International Invoice |
| Universal Business Language 2.1 (JoFotara) | `account.edi.xml.ubl_21.jo` | `none` | Shared behaviour definition reused through composition. | Jordan Electronic Invoicing |
| Universal Business Language 2.1 (JoFotara) for Point of Sale Orders | `pos.edi.xml.ubl_21.jo` | `none` | Shared behaviour definition reused through composition. | Jordan Accounting electronic data interchange for point of sale |
| Universal Business Language 2.1 (RS eFaktura) | `account.edi.xml.ubl.rs` | `none` | Shared behaviour definition reused through composition. | Serbia - eFaktura Electronic Invoicing |
| Universal Business Language 2.1 (Zakat Tax and Customs Authority) | `account.edi.xml.ubl_21.zatca` | `none` | Shared behaviour definition reused through composition. | Saudi Arabia - Electronic Invoicing |
| Universal Business Language-TR 1.2 | `account.edi.xml.ubl.tr` | `none` | Shared behaviour definition reused through composition. | Türkiye - Nilvera Electronic Invoice |

#### Relationships (101)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Approved Dematerialization Platform Registration | `pdp.registration` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Approved Dematerialization Platform Registration | `pdp.registration` | `edi_user_id` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Approved Dematerialization Platform Response wizard | `pdp.response.wizard` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Approved Dematerialization Platform Response wizard | `pdp.response.wizard` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Argentinean Partner Taxes | `l10n_ar.partner.tax` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Argentinean Partner Taxes | `l10n_ar.partner.tax` | `tax_id` | link to one record | Tax | `account.tax` | `n : 1` | `not declared` | stored |
| Brazilian city zip range | `l10n_br.zip.range` | `city_id` | link to one record | City | `res.city` | `n : 1` | `not declared` | stored |
| Business Level Responses for Nemhandel | `nemhandel.response` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `cascade` | stored |
| Cancel Electronic Invoice | `l10n_in_edi.cancel` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `not declared` | stored |
| Cancel Electronic Waybill | `l10n.in.ewaybill.cancel` | `l10n_in_ewaybill_id` | link to one record | Electronic Waybill | `l10n.in.ewaybill` | `n : 1` | `not declared` | stored |
| Compliance Letter for EXO Number | `compliance.letter.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Customer Alias on Nilvera | `l10n_tr.nilvera.alias` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `invoice_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `sale_order_ids` | list of records | Sales Order | `sale.order` | `1 : 0..n` | `mirror of the target column` | derived |
| District | `l10n_pe.res.city.district` | `city_id` | link to one record | City | `res.city` | `n : 0..1` | `not declared` | stored |
| Document Status Update Wizard | `myinvois.document.status.update.wizard` | `document_id` | link to one record | MyInvois Document | `myinvois.document` | `n : 1` | `not declared` | stored |
| Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `batch_id` | link to one record | Batch Transfer | `stock.picking.batch` | `n : 0..1` | `not declared` | stored |
| Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `invoice_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| E-Faktur Document | `l10n_id_efaktur_coretax.document` | `invoice_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Electronic Invoice cancellation wizard | `l10n_vn_edi_viettel.cancellation` | `invoice_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Electronic Waybill | `l10n.in.ewaybill` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `partner_bill_from_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `partner_bill_to_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `partner_ship_from_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `partner_ship_to_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `transporter_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Electronic Waybill | `l10n.in.ewaybill` | `type_id` | link to one record | Electronic Waybill Document Type | `l10n.in.ewaybill.type` | `n : 0..1` | `not declared` | stored |
| Exports 2307 data to a XLS file. | `l10n_ph_2307.wizard` | `moves_to_export` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Fichier Echange Informatise | `l10n_fr.fec.export.wizard` | `excluded_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | stored |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `initial_flow_id` | link to one record | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `n : 0..1` | `not declared` | stored |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `payload_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `rectificative_flow_ids` | list of records | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `1 : 0..n` | `mirror of the target column` | derived |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `sent_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi.document` | `attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi.document` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `cascade` | stored |
| Identification Types | `l10n_latam.identification.type` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Implements cancelling an ecpay invoice. | `l10n_tw_edi.invoice.cancel` | `invoice_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `not declared` | stored |
| Implements printingan ecpay invoice. | `l10n_tw_edi.invoice.print` | `invoice_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `not declared` | stored |
| Indian permanent account number Entity | `l10n_in.pan.entity` | `partner_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Indian port code | `l10n_in.port.code` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Latam Document Type | `l10n_latam.document.type` | `country_id` | link to one record | Country | `res.country` | `n : 1` | `not declared` | stored |
| MojEracun Reject Invoice Wizard | `l10n_hr_edi.mojeracun_reject_wizard` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `not declared` | stored |
| MyInvois Document | `myinvois.document` | `Invoices` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| MyInvois Document | `myinvois.document` | `Orders` | list on both sides | Point of Sale Orders | `pos.order` | `0..n : 0..n` | `not declared` | stored |
| MyInvois Document | `myinvois.document` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| MyInvois Document | `myinvois.document` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| MyInvois Document | `myinvois.document` | `myinvois_file_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| MyInvois Document | `myinvois.document` | `pos_config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 0..1` | `not declared` | stored |
| Nemhandel Registration | `nemhandel.registration` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Nemhandel Registration | `nemhandel.registration` | `edi_user_id` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Nemhandel Rejection wizard | `nemhandel.rejection.wizard` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| PL Bank Account Verification | `l10n_pl.bank.account.verification` | `partner_bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `not declared` | stored |
| PL Bank Account Verification | `l10n_pl.bank.account.verification` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Payment register withholding lines | `l10n_ar.payment.register.withholding` | `payment_register_id` | link to one record | Pay | `account.payment.register` | `n : 1` | `cascade` | stored |
| Payment register withholding lines | `l10n_ar.payment.register.withholding` | `tax_id` | link to one record | Tax | `account.tax` | `n : 1` | `not declared` | stored |
| Peppol Configuration Wizard | `pdp.config.wizard` | `account_peppol_edi_user` | link to one record | Account electronic data interchange proxy user | `account_edi_proxy_client.user` | `n : 0..1` | `not declared` | derived |
| Peppol Configuration Wizard | `pdp.config.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `product_template_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | stored |
| Record of QRIS transactions | `l10n_id.qris.transaction` | `bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `not declared` | stored |
| Request Zakat Tax and Customs Authority one-time password | `l10n_sa_edi.otp.wizard` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `invoice_template_id` | link to one record | SInvoice template | `l10n_vn_edi_viettel.sinvoice.template` | `n : 1` | `not declared` | stored |
| SInvoice template | `l10n_vn_edi_viettel.sinvoice.template` | `invoice_symbols_ids` | list of records | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `1 : 0..n` | `mirror of the target column` | derived |
| Sale Closing | `account.sale.closing` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Sale Closing | `account.sale.closing` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Sale Closing | `account.sale.closing` | `last_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Send Approved Dematerialization Platform Flow Wizard | `l10n.fr.pdp.reports.send.wizard` | `flow_id` | link to one record | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `n : 1` | `cascade` | stored |
| Technical Annulment Wizard | `l10n_hu_edi.cancellation` | `invoice_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Thumb drive used to sign invoices in Egypt | `l10n_eg_edi.thumb.drive` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| TicketBAI Document | `l10n_es_edi_tbai.document` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| TicketBAI Document | `l10n_es_edi_tbai.document` | `xml_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Transport Document | `l10n_it.ddt` | `invoice_id` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Turkish Tax Office | `l10n_tr_nilvera_einvoice_extended.tax.office` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `json_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Veri*Factu Document | `l10n_es_edi_verifactu.document` | `pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Withhold Wizard | `l10n_in.withhold.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Withhold Wizard | `l10n_in.withhold.wizard` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Withhold Wizard | `l10n_in.withhold.wizard` | `related_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Withhold Wizard | `l10n_in.withhold.wizard` | `related_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Withhold Wizard | `l10n_in.withhold.wizard` | `tax_id` | link to one record | Tax | `account.tax` | `n : 1` | `not declared` | stored |
| electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi.addendum` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `cascade` | stored |
| indian section alert | `l10n_in.section.alert` | `l10n_in_section_tax_ids` | list of records | Tax | `account.tax` | `1 : 0..n` | `mirror of the target column` | derived |
| indian section alert | `l10n_in.section.alert` | `tax_report_line_id` | link to one record | Accounting Report Line | `account.report.line` | `n : 0..1` | `not declared` | stored |
| l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | `line_ids` | list of records | l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | `1 : 0..n` | `mirror of the target column` | derived |
| l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| l10n_ar.earnings.scale.line | `l10n_ar.earnings.scale.line` | `scale_id` | link to one record | l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | `n : 1` | `cascade` | stored |

### 3.15 General Ledger

The chart of accounts, account groups and tags, journals and journal groups, journal entries and journal items, posting, numbering, reversal, lock dates, the inalterability hash chain, the reconciliation core, financial report structures, payment terms and the fiscal year.

Specified in [`../domains/general-ledger/`](../domains/general-ledger/).

#### Persistent entities (34)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account | `account.account` | `account_account` | Persistent record with 15 stored columns; owns Mapping of account codes per company; referenced by 104 relation fields. | Invoicing |
| Account Cash Rounding | `account.cash.rounding` | `account_cash_rounding` | Persistent record with 6 stored columns; referenced by 2 relation fields. | Invoicing |
| Account codes first 2 digits | `account.root` | `none, read from a stored query` | Persistent record with 0 stored columns; referenced by 2 relation fields. | Invoicing |
| Account Group | `account.group` | `account_group` | Persistent record with 5 stored columns; belongs to Companies; company scoped; referenced by 2 relation fields. | Invoicing |
| Account Journal Group | `account.journal.group` | `account_journal_group` | Persistent record with 3 stored columns; company scoped; referenced by 3 relation fields. | Invoicing |
| Account Lock Exception | `account.lock_exception` | `account_lock_exception` | Persistent record with 8 stored columns; belongs to Companies; lifecycle states Active, Revoked, Expired; company scoped. | Invoicing |
| Account Tag | `account.account.tag` | `account_account_tag` | Persistent record with 5 stored columns; referenced by 4 relation fields. | Invoicing |
| Accounting Report | `account.report` | `account_report` | Persistent record with 31 stored columns; owns Accounting Report, Accounting Report Column, Accounting Report Line; referenced by 5 relation fields. | Invoicing |
| Accounting Report Column | `account.report.column` | `account_report_column` | Persistent record with 8 stored columns. | Invoicing |
| Accounting Report Expression | `account.report.expression` | `account_report_expression` | Persistent record with 11 stored columns; belongs to Accounting Report Line; referenced by 2 relation fields. | Invoicing |
| Accounting Report External Value | `account.report.external.value` | `account_report_external_value` | Persistent record with 8 stored columns; belongs to Accounting Report Expression, Companies; company scoped. | Invoicing |
| Accounting Report Line | `account.report.line` | `account_report_line` | Persistent record with 13 stored columns; belongs to Accounting Report; owns Accounting Report Expression, Accounting Report Line; referenced by 4 relation fields. | Invoicing |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `account_fiscal_position_account` | Persistent record with 4 stored columns; belongs to Account, Fiscal Position; company scoped. | Invoicing |
| Bank Statement | `account.bank.statement` | `account_bank_statement` | Persistent record with 11 stored columns; owns Bank Statement Line; company scoped; referenced by 2 relation fields. | Invoicing |
| Bank Statement Line | `account.bank.statement.line` | `account_bank_statement_line` | Persistent record with 20 stored columns; belongs to Companies, Journal, Journal Entry; company scoped; referenced by 4 relation fields. | Invoicing |
| Fiscal Position | `account.fiscal.position` | `account_fiscal_position` | Persistent record with 14 stored columns; belongs to Companies; owns Accounts Mapping of Fiscal Position, Preferred myDATA classification combinations for a particular product; company scoped; referenced by 20 relation fields. | Invoicing |
| Full Reconcile | `account.full.reconcile` | `account_full_reconcile` | Persistent record with 0 stored columns; owns Journal Item, Partial Reconcile; referenced by 2 relation fields. | Invoicing |
| Incoterms | `account.incoterms` | `account_incoterms` | Persistent record with 3 stored columns; referenced by 5 relation fields. | Invoicing |
| Invoices Statistics | `account.invoice.report` | `none, read from a stored query` | Persistent record with 0 stored columns; lifecycle states Draft, Open, Cancelled; 2 state fields in all; company scoped. | Invoicing |
| Journal | `account.journal` | `account_journal` | Persistent record with 64 stored columns; belongs to Companies; owns Payment Methods, Point of Sale Payment Methods, Report Action; carries the state field `account_peppol_proxy_state`; 3 state fields in all; company scoped; referenced by 62 relation fields. | Invoicing |
| Journal Entry | `account.move` | `account_move` | Persistent record with 249 stored columns; belongs to Currency, Journal; owns Analytic Line, Attachment, Bank Statement Line and 19 further collections; lifecycle states Draft, Posted, Cancelled; 27 state fields in all; company scoped; referenced by 73 relation fields. | Invoicing |
| Journal Item | `account.move.line` | `account_move_line` | Persistent record with 70 stored columns; belongs to Currency, Journal Entry; owns Account payment check, Analytic Line, Partial Reconcile and 1 further collections; carries the state field `parent_state`; referenced by 17 relation fields. | Invoicing |
| Mapping of account codes per company | `account.code.mapping` | `none, read from a stored query` | Persistent record with 0 stored columns; company scoped. | Invoicing |
| Partial Reconcile | `account.partial.reconcile` | `account_partial_reconcile` | Persistent record with 12 stored columns; belongs to Journal Item; company scoped; referenced by 1 relation field. | Invoicing |
| Payment Methods | `account.payment.method` | `account_payment_method` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Invoicing |
| Payment Methods | `account.payment.method.line` | `account_payment_method_line` | Persistent record with 7 stored columns; belongs to Payment Methods; carries the state field `payment_provider_state`; referenced by 12 relation fields. | Invoicing |
| Payment Terms | `account.payment.term` | `account_payment_term` | Persistent record with 10 stored columns; owns Payment Terms Line; company scoped; referenced by 6 relation fields. | Invoicing |
| Payment Terms Line | `account.payment.term.line` | `account_payment_term_line` | Persistent record with 6 stored columns; belongs to Payment Terms. | Invoicing |
| Payments | `account.payment` | `account_payment` | Persistent record with 35 stored columns; belongs to Companies, Journal; owns Account payment check, Attachment, Journal Entry and 1 further collections; lifecycle states Draft, In Process, Paid, Canceled, Rejected; 2 state fields in all; company scoped; referenced by 17 relation fields. | Invoicing |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `account_reconcile_model` | Persistent record with 13 stored columns; belongs to Companies; owns Rules for the reconciliation model; company scoped; referenced by 2 relation fields. | Invoicing |
| Rules for the reconciliation model | `account.reconcile.model.line` | `account_reconcile_model_line` | Persistent record with 10 stored columns; referenced by 1 relation field. | Invoicing |
| Tax | `account.tax` | `account_tax` | Persistent record with 76 stored columns; belongs to Companies, Country, Tax Group; owns Tax Repartition Line; company scoped; referenced by 30 relation fields. | Invoicing |
| Tax Group | `account.tax.group` | `account_tax_group` | Persistent record with 12 stored columns; belongs to Companies; company scoped; referenced by 1 relation field. | Invoicing |
| Tax Repartition Line | `account.tax.repartition.line` | `account_tax_repartition_line` | Persistent record with 8 stored columns; company scoped; referenced by 1 relation field. | Invoicing |

#### Interactive assistant entities (14)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account merge wizard | `account.merge.wizard` | `account_merge_wizard` | Interactive assistant with 1 stored column; owns Account merge wizard line; referenced by 1 relation field. | Invoicing |
| Account merge wizard line | `account.merge.wizard.line` | `account_merge_wizard_line` | Interactive assistant with 6 stored columns; belongs to Account merge wizard. | Invoicing |
| Account Move Reversal | `account.move.reversal` | `account_move_reversal` | Interactive assistant with 16 stored columns; belongs to Companies, Journal; company scoped. | Invoicing |
| Account Move Send Batch Wizard | `account.move.send.batch.wizard` | `account_move_send_batch_wizard` | Interactive assistant with 0 stored columns. | Invoicing |
| Account Move Send Wizard | `account.move.send.wizard` | `account_move_send_wizard` | Interactive assistant with 12 stored columns; belongs to Journal Entry; owns Report Action; company scoped. | Invoicing |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | `account_accrued_orders_wizard` | Interactive assistant with 7 stored columns; belongs to Account, Journal; company scoped. | Invoicing |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `account_autopost_bills_wizard` | Interactive assistant with 2 stored columns. | Invoicing |
| Bank setup manual config | `account.setup.bank.manual.config` | `account_setup_bank_manual_config` | Interactive assistant with 4 stored columns; belongs to Bank Accounts, Companies; company scoped. | Invoicing |
| Create Automatic Entries | `account.automatic.entry.wizard` | `account_automatic_entry_wizard` | Interactive assistant with 7 stored columns; belongs to Companies, Journal; company scoped. | Invoicing |
| Opening Balance of Financial Year | `account.financial.year.op` | `account_financial_year_op` | Interactive assistant with 1 stored column; belongs to Companies; company scoped. | Invoicing |
| Pay | `account.payment.register` | `account_payment_register` | Interactive assistant with 27 stored columns; owns Payment register check, Payment register withholding line, Payment register withholding lines; company scoped; referenced by 3 relation fields. | Invoicing |
| Remake the sequence of Journal Entries. | `account.resequence.wizard` | `account_resequence_wizard` | Interactive assistant with 4 stored columns. | Invoicing |
| Secure Journal Entries | `account.secure.entries.wizard` | `account_secure_entries_wizard` | Interactive assistant with 2 stored columns; belongs to Companies; company scoped. | Invoicing |
| Validate Account Move | `validate.account.move` | `validate_account_move` | Interactive assistant with 4 stored columns; owns Contact. | Invoicing |

#### Shared behaviour entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account Chart Template | `account.chart.template` | `none` | Shared behaviour definition reused through composition. | Invoicing |
| Account Move Send | `account.move.send` | `none` | Shared behaviour merged into 2 entities. | Invoicing |
| Automatic sequence | `sequence.mixin` | `none` | Shared behaviour merged into 2 entities. | Invoicing |
| Business document import mixin | `account.document.import.mixin` | `none` | Shared behaviour merged into 3 entities. | Invoicing |

#### Relationships (425)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Account | `account.account` | `account_stock_expense_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Account | `account.account` | `account_stock_variation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Account | `account.account` | `code_mapping_ids` | list of records | Mapping of account codes per company | `account.code.mapping` | `1 : 0..n` | `mirror of the target column` | derived |
| Account | `account.account` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Account | `account.account` | `company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | stored |
| Account | `account.account` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Account | `account.account` | `group_id` | link to one record | Account Group | `account.group` | `n : 0..1` | `not declared` | derived |
| Account | `account.account` | `l10n_in_tds_tcs_section_id` | link to one record | indian section alert | `l10n_in.section.alert` | `n : 0..1` | `not declared` | stored |
| Account | `account.account` | `root_id` | link to one record | Account codes first 2 digits | `account.root` | `n : 0..1` | `not declared` | derived |
| Account | `account.account` | `tag_ids` | list on both sides | Account Tag | `account.account.tag` | `0..n : 0..n` | `restrict` | stored |
| Account | `account.account` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Account Cash Rounding | `account.cash.rounding` | `loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Account Cash Rounding | `account.cash.rounding` | `profit_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Account Group | `account.group` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Account Group | `account.group` | `parent_id` | link to one record | Account Group | `account.group` | `n : 0..1` | `cascade` | stored |
| Account Journal Group | `account.journal.group` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Account Journal Group | `account.journal.group` | `excluded_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | stored |
| Account Lock Exception | `account.lock_exception` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Account Lock Exception | `account.lock_exception` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Account Move Reversal | `account.move.reversal` | `available_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | derived |
| Account Move Reversal | `account.move.reversal` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Account Move Reversal | `account.move.reversal` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Account Move Reversal | `account.move.reversal` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Account Move Reversal | `account.move.reversal` | `l10n_latam_available_document_type_ids` | list on both sides | Latam Document Type | `l10n_latam.document.type` | `0..n : 0..n` | `not declared` | derived |
| Account Move Reversal | `account.move.reversal` | `l10n_latam_document_type_id` | link to one record | Latam Document Type | `l10n_latam.document.type` | `n : 0..1` | `cascade` | stored |
| Account Move Reversal | `account.move.reversal` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Account Move Reversal | `account.move.reversal` | `new_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Account Move Send Batch Wizard | `account.move.send.batch.wizard` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Account Move Send Wizard | `account.move.send.wizard` | `available_pdf_report_ids` | list of records | Report Action | `ir.actions.report` | `1 : 0..n` | `mirror of the target column` | derived |
| Account Move Send Wizard | `account.move.send.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Account Move Send Wizard | `account.move.send.wizard` | `mail_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Account Move Send Wizard | `account.move.send.wizard` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `not declared` | stored |
| Account Move Send Wizard | `account.move.send.wizard` | `pdf_report_id` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | stored |
| Account Tag | `account.account.tag` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Account Tag | `account.account.tag` | `report_expression_id` | link to one record | Accounting Report Expression | `account.report.expression` | `n : 0..1` | `not declared` | derived |
| Account codes first 2 digits | `account.root` | `parent_id` | link to one record | Account codes first 2 digits | `account.root` | `n : 0..1` | `not declared` | derived |
| Account merge wizard | `account.merge.wizard` | `account_ids` | list on both sides | Account | `account.account` | `0..n : 0..n` | `not declared` | stored |
| Account merge wizard | `account.merge.wizard` | `wizard_line_ids` | list of records | Account merge wizard line | `account.merge.wizard.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Account merge wizard line | `account.merge.wizard.line` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `cascade` | stored |
| Account merge wizard line | `account.merge.wizard.line` | `wizard_id` | link to one record | Account merge wizard | `account.merge.wizard` | `n : 1` | `cascade` | stored |
| Accounting Report | `account.report` | `column_ids` | list of records | Accounting Report Column | `account.report.column` | `1 : 0..n` | `mirror of the target column` | derived |
| Accounting Report | `account.report` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Accounting Report | `account.report` | `line_ids` | list of records | Accounting Report Line | `account.report.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Accounting Report | `account.report` | `root_report_id` | link to one record | Accounting Report | `account.report` | `n : 0..1` | `not declared` | stored |
| Accounting Report | `account.report` | `section_main_report_ids` | list on both sides | Accounting Report | `account.report` | `0..n : 0..n` | `not declared` | stored |
| Accounting Report | `account.report` | `section_report_ids` | list on both sides | Accounting Report | `account.report` | `0..n : 0..n` | `not declared` | stored |
| Accounting Report | `account.report` | `variant_report_ids` | list of records | Accounting Report | `account.report` | `1 : 0..n` | `mirror of the target column` | derived |
| Accounting Report Column | `account.report.column` | `custom_audit_action_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 0..1` | `not declared` | stored |
| Accounting Report Column | `account.report.column` | `report_id` | link to one record | Accounting Report | `account.report` | `n : 0..1` | `not declared` | stored |
| Accounting Report Expression | `account.report.expression` | `report_line_id` | link to one record | Accounting Report Line | `account.report.line` | `n : 1` | `cascade` | stored |
| Accounting Report External Value | `account.report.external.value` | `carryover_origin_report_line_id` | link to one record | Accounting Report Line | `account.report.line` | `n : 0..1` | `not declared` | stored |
| Accounting Report External Value | `account.report.external.value` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Accounting Report External Value | `account.report.external.value` | `target_report_expression_id` | link to one record | Accounting Report Expression | `account.report.expression` | `n : 1` | `cascade` | stored |
| Accounting Report Line | `account.report.line` | `action_id` | link to one record | Actions | `ir.actions.actions` | `n : 0..1` | `not declared` | stored |
| Accounting Report Line | `account.report.line` | `children_ids` | list of records | Accounting Report Line | `account.report.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Accounting Report Line | `account.report.line` | `expression_ids` | list of records | Accounting Report Expression | `account.report.expression` | `1 : 0..n` | `mirror of the target column` | derived |
| Accounting Report Line | `account.report.line` | `parent_id` | link to one record | Accounting Report Line | `account.report.line` | `n : 0..1` | `set null` | stored |
| Accounting Report Line | `account.report.line` | `report_id` | link to one record | Accounting Report | `account.report` | `n : 1` | `cascade` | stored |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `account_dest_id` | link to one record | Account | `account.account` | `n : 1` | `not declared` | stored |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `account_src_id` | link to one record | Account | `account.account` | `n : 1` | `not declared` | stored |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 1` | `cascade` | stored |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | `account_id` | link to one record | Account | `account.account` | `n : 1` | `not declared` | stored |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Accrued Orders Wizard | `account.accrued.orders.wizard` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Autopost Bills Wizard | `account.autopost.bills.wizard` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Bank Statement | `account.bank.statement` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Bank Statement | `account.bank.statement` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Bank Statement | `account.bank.statement` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Bank Statement | `account.bank.statement` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Bank Statement | `account.bank.statement` | `line_ids` | list of records | Bank Statement Line | `account.bank.statement.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Bank Statement Line | `account.bank.statement.line` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `foreign_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `cascade` | stored |
| Bank Statement Line | `account.bank.statement.line` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Bank Statement Line | `account.bank.statement.line` | `payment_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `pos_session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Bank Statement Line | `account.bank.statement.line` | `statement_id` | link to one record | Bank Statement | `account.bank.statement` | `n : 0..1` | `not declared` | stored |
| Bank setup manual config | `account.setup.bank.manual.config` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | derived |
| Bank setup manual config | `account.setup.bank.manual.config` | `linked_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Bank setup manual config | `account.setup.bank.manual.config` | `res_partner_bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 1` | `cascade` | stored |
| Create Automatic Entries | `account.automatic.entry.wizard` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Create Automatic Entries | `account.automatic.entry.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Create Automatic Entries | `account.automatic.entry.wizard` | `destination_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Create Automatic Entries | `account.automatic.entry.wizard` | `expense_accrual_account` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Create Automatic Entries | `account.automatic.entry.wizard` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | derived |
| Create Automatic Entries | `account.automatic.entry.wizard` | `move_line_ids` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | stored |
| Create Automatic Entries | `account.automatic.entry.wizard` | `revenue_accrual_account` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Fiscal Position | `account.fiscal.position` | `account_ids` | list of records | Accounts Mapping of Fiscal Position | `account.fiscal.position.account` | `1 : 0..n` | `mirror of the target column` | derived |
| Fiscal Position | `account.fiscal.position` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Fiscal Position | `account.fiscal.position` | `country_group_id` | link to one record | Country Group | `res.country.group` | `n : 0..1` | `not declared` | stored |
| Fiscal Position | `account.fiscal.position` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Fiscal Position | `account.fiscal.position` | `l10n_ar_afip_responsibility_type_ids` | list on both sides | ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `0..n : 0..n` | `not declared` | stored |
| Fiscal Position | `account.fiscal.position` | `l10n_gr_edi_preferred_classification_ids` | list of records | Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `1 : 0..n` | `mirror of the target column` | derived |
| Fiscal Position | `account.fiscal.position` | `state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | stored |
| Fiscal Position | `account.fiscal.position` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Full Reconcile | `account.full.reconcile` | `partial_reconcile_ids` | list of records | Partial Reconcile | `account.partial.reconcile` | `1 : 0..n` | `mirror of the target column` | derived |
| Full Reconcile | `account.full.reconcile` | `reconciled_line_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Invoices Statistics | `account.invoice.report` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `invoice_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `l10n_ar_state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `l10n_latam_document_type_id` | link to one record | Latam Document Type | `l10n_latam.document.type` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `product_categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Invoices Statistics | `account.invoice.report` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | derived |
| Journal | `account.journal` | `available_invoice_template_pdf_report_ids` | list of records | Report Action | `ir.actions.report` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal | `account.journal` | `available_payment_method_ids` | list on both sides | Payment Methods | `account.payment.method` | `0..n : 0..n` | `not declared` | derived |
| Journal | `account.journal` | `bank_account_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `restrict` | stored |
| Journal | `account.journal` | `bank_id` | link to one record | Bank | `res.bank` | `n : 0..1` | `not declared` | derived |
| Journal | `account.journal` | `check_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Journal | `account.journal` | `company_partner` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Journal | `account.journal` | `company_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Journal | `account.journal` | `compatible_edi_ids` | list on both sides | electronic data interchange format | `account.edi.format` | `0..n : 0..n` | `not declared` | stored |
| Journal | `account.journal` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `default_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Journal | `account.journal` | `edi_format_ids` | list on both sides | electronic data interchange format | `account.edi.format` | `0..n : 0..n` | `not declared` | stored |
| Journal | `account.journal` | `inbound_payment_method_line_ids` | list of records | Payment Methods | `account.payment.method.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal | `account.journal` | `invoice_template_pdf_report_id` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `journal_group_ids` | list on both sides | Account Journal Group | `account.journal.group` | `0..n : 0..n` | `not declared` | stored |
| Journal | `account.journal` | `l10n_ar_afip_pos_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_ec_emission_address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_eg_activity_type_id` | link to one record | Estimated Time of Arrival code for activity type | `l10n_eg_edi.activity.type` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_eg_branch_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_sa_chain_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_sa_compliance_csid_certificate_id` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_sa_production_csid_certificate_id` | link to one record | Certificate | `certificate.certificate` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `l10n_tr_default_sales_return_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `last_statement_id` | link to one record | Bank Statement | `account.bank.statement` | `n : 0..1` | `not declared` | derived |
| Journal | `account.journal` | `loss_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `non_deductible_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `outbound_payment_method_line_ids` | list of records | Payment Methods | `account.payment.method.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal | `account.journal` | `pos_payment_method_ids` | list of records | Point of Sale Payment Methods | `pos.payment.method` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal | `account.journal` | `profit_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Journal | `account.journal` | `suspense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Journal Entry | `account.move` | `MyInvois Documents` | list on both sides | MyInvois Document | `myinvois.document` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `adjusting_entries_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `adjusting_entry_origin_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `audit_trail_message_ids` | list of records | Message | `mail.message` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `authorized_transaction_ids` | list on both sides | Payment Transaction | `payment.transaction` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `auto_post_origin_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `bank_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Journal Entry | `account.move` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Journal Entry | `account.move` | `debit_note_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `debit_origin_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `duplicated_ref_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `edi_document_ids` | list of records | Electronic Document for an account.move | `account.edi.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `exchange_diff_partial_ids` | list of records | Partial Reconcile | `account.partial.reconcile` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `expense_ids` | list of records | Expense | `hr.expense` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `restrict` | stored |
| Journal Entry | `account.move` | `invoice_cash_rounding_id` | link to one record | Account Cash Rounding | `account.cash.rounding` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `invoice_incoterm_id` | link to one record | Incoterms | `account.incoterms` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `invoice_line_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `invoice_payment_term_id` | link to one record | Payment Terms | `account.payment.term` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `invoice_pdf_report_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `invoice_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `invoice_vendor_bill_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `journal_group_id` | link to one record | Account Journal Group | `account.journal.group` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Journal Entry | `account.move` | `journal_line_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_ar_afip_responsibility_type_id` | link to one record | ARCA Responsibility Type | `l10n_ar.afip.responsibility.type` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_ar_withholding_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_ec_sri_payment_id` | link to one record | SRI Payment Method | `l10n_ec.sri.payment` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_es_edi_facturae_xml_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_es_edi_verifactu_document_ids` | list of records | Veri*Factu Document | `l10n_es_edi_verifactu.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_es_edi_verifactu_substituted_entry_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_es_edi_verifactu_substitution_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_es_tbai_cancel_document_id` | link to one record | TicketBAI Document | `l10n_es_edi_tbai.document` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_es_tbai_post_document_id` | link to one record | TicketBAI Document | `l10n_es_edi_tbai.document` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_es_tbai_reversed_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_fr_pdp_last_flow_id` | link to one record | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_fr_pdp_sent_in_flow_ids` | list on both sides | French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_gr_edi_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_gr_edi_correlation_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_gr_edi_document_ids` | list of records | Greece document object for tracking all sent extensible markup language to myDATA | `l10n_gr_edi.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_hr_edi_addendum_id` | list of records | electronic data interchange and fiscalization information for Croatian electronic invoicing | `l10n_hr_edi.addendum` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_hr_fiscal_user_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_id_coretax_document` | link to one record | E-Faktur Document | `l10n_id_efaktur_coretax.document` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_id_qris_transaction_ids` | list on both sides | Record of QRIS transactions | `l10n_id.qris.transaction` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_in_edi_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_in_ewaybill_ids` | list of records | Electronic Waybill | `l10n.in.ewaybill` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_in_reseller_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_in_shipping_port_code_id` | link to one record | Indian port code | `l10n_in.port.code` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_in_state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_in_withhold_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_in_withholding_line_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_in_withholding_ref_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_in_withholding_ref_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_it_ddt_id` | link to one record | Transport Document | `l10n_it.ddt` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_it_ddt_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_it_document_type` | link to one record | Italian Document Type | `l10n_it.document.type` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_it_edi_doi_id` | link to one record | Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_jo_edi_xml_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_latam_available_document_type_ids` | list on both sides | Latam Document Type | `l10n_latam.document.type` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_latam_document_type_id` | link to one record | Latam Document Type | `l10n_latam.document.type` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_pl_edi_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_pl_edi_upo_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_ro_edi_document_ids` | list of records | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `l10n_rs_edi_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_sa_edi_chain_head_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_tr_exemption_code_id` | link to one record | Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_tw_edi_file_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_vn_edi_invoice_symbol` | link to one record | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_vn_edi_replacement_origin_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `l10n_vn_edi_sinvoice_file_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_vn_edi_sinvoice_pdf_file_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `l10n_vn_edi_sinvoice_xml_file_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `landed_costs_ids` | list of records | Stock Landed Cost | `stock.landed.cost` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `line_ids` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `matched_payment_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `nemhandel_response_ids` | list of records | Business Level Responses for Nemhandel | `nemhandel.response` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `origin_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `partner_bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `restrict` | stored |
| Journal Entry | `account.move` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Journal Entry | `account.move` | `partner_shipping_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `payment_ids` | list of records | Payments | `account.payment` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `peppol_response_ids` | list of records | Business Level Responses for Peppol | `account.peppol.response` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `pos_order_ids` | list of records | Point of Sale Orders | `pos.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `pos_payment_ids` | list of records | Point of Sale Payments | `pos.payment` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `pos_refunded_invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `pos_session_ids` | list of records | Point of Sale Session | `pos.session` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `preferred_payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `purchase_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `purchase_vendor_bill_id` | link to one record | Purchases & Bills Union | `purchase.bill.union` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `reconciled_payment_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `reversal_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `reversed_entry_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `reversed_pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `statement_line_id` | link to one record | Bank Statement Line | `account.bank.statement.line` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `statement_line_ids` | list of records | Bank Statement Line | `account.bank.statement.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `stock_move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `suitable_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | derived |
| Journal Entry | `account.move` | `tax_cash_basis_created_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `tax_cash_basis_origin_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `tax_cash_basis_rec_id` | link to one record | Partial Reconcile | `account.partial.reconcile` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `tax_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Journal Entry | `account.move` | `timesheet_encode_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Entry | `account.move` | `transaction_ids` | list on both sides | Payment Transaction | `payment.transaction` | `0..n : 0..n` | `not declared` | stored |
| Journal Entry | `account.move` | `ubl_cii_xml_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | derived |
| Journal Entry | `account.move` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Journal Entry | `account.move` | `wip_production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Journal Item | `account.move.line` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Journal Item | `account.move.line` | `analytic_line_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Item | `account.move.line` | `cogs_origin_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Journal Item | `account.move.line` | `exchange_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Journal Item | `account.move.line` | `expense_id` | link to one record | Expense | `hr.expense` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `first_reconciled_lines_excluding_exchange_diff_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `first_reconciled_lines_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `full_reconcile_id` | link to one record | Full Reconcile | `account.full.reconcile` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `group_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `journal_group_id` | link to one record | Account Journal Group | `account.journal.group` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `l10n_hr_kpd_category_id` | link to one record | Croatian KPD Category | `l10n_hr.kpd.category` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `l10n_latam_check_ids` | list of records | Account payment check | `l10n_latam.check` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Item | `account.move.line` | `matched_credit_ids` | list of records | Partial Reconcile | `account.partial.reconcile` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Item | `account.move.line` | `matched_debit_ids` | list of records | Partial Reconcile | `account.partial.reconcile` | `1 : 0..n` | `mirror of the target column` | derived |
| Journal Item | `account.move.line` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 1` | `cascade` | stored |
| Journal Item | `account.move.line` | `parent_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `purchase_line_id` | link to one record | Purchase Order Line | `purchase.order.line` | `n : 0..1` | `set null` | stored |
| Journal Item | `account.move.line` | `purchase_order_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `reconcile_model_id` | link to one record | Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `reconciled_lines_excluding_exchange_diff_ids` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | derived |
| Journal Item | `account.move.line` | `reconciled_lines_ids` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | derived |
| Journal Item | `account.move.line` | `sale_line_ids` | list on both sides | Sales Order Line | `sale.order.line` | `0..n : 0..n` | `not declared` | stored |
| Journal Item | `account.move.line` | `search_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Journal Item | `account.move.line` | `statement_line_id` | link to one record | Bank Statement Line | `account.bank.statement.line` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Journal Item | `account.move.line` | `tax_line_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `tax_repartition_line_id` | link to one record | Tax Repartition Line | `account.tax.repartition.line` | `n : 0..1` | `restrict` | stored |
| Journal Item | `account.move.line` | `tax_tag_ids` | list on both sides | Account Tag | `account.account.tag` | `0..n : 0..n` | `restrict` | stored |
| Journal Item | `account.move.line` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 0..1` | `not declared` | stored |
| Journal Item | `account.move.line` | `vehicle_log_service_ids` | list of records | Services for vehicles | `fleet.vehicle.log.services` | `1 : 0..n` | `mirror of the target column` | derived |
| Mapping of account codes per company | `account.code.mapping` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Mapping of account codes per company | `account.code.mapping` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Opening Balance of Financial Year | `account.financial.year.op` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Partial Reconcile | `account.partial.reconcile` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `credit_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `credit_move_id` | link to one record | Journal Item | `account.move.line` | `n : 1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `debit_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `debit_move_id` | link to one record | Journal Item | `account.move.line` | `n : 1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `exchange_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Partial Reconcile | `account.partial.reconcile` | `full_reconcile_id` | link to one record | Full Reconcile | `account.full.reconcile` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `available_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `available_partner_bank_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `available_payment_method_line_ids` | list on both sides | Payment Methods | `account.payment.method.line` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Pay | `account.payment.register` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `custom_user_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `duplicate_payment_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `l10n_ar_withholding_ids` | list of records | Payment register withholding lines | `l10n_ar.payment.register.withholding` | `1 : 0..n` | `mirror of the target column` | derived |
| Pay | `account.payment.register` | `l10n_latam_move_check_ids` | list on both sides | Account payment check | `l10n_latam.check` | `0..n : 0..n` | `not declared` | stored |
| Pay | `account.payment.register` | `l10n_latam_new_check_ids` | list of records | Payment register check | `l10n_latam.payment.register.check` | `1 : 0..n` | `mirror of the target column` | derived |
| Pay | `account.payment.register` | `l10n_pl_bank_verification_ids` | list on both sides | PL Bank Account Verification | `l10n_pl.bank.account.verification` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `l10n_pl_bank_verification_invalid_bank_account_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `l10n_pl_incomplete_data_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `l10n_pl_not_found_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `line_ids` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | stored |
| Pay | `account.payment.register` | `missing_account_partners` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `partner_bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Pay | `account.payment.register` | `payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `payment_token_id` | link to one record | Payment Token | `payment.token` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `source_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `suitable_payment_token_ids` | list on both sides | Payment Token | `payment.token` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `untrusted_bank_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | derived |
| Pay | `account.payment.register` | `withholding_line_ids` | list of records | Payment register withholding line | `account.payment.register.withholding.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Pay | `account.payment.register` | `withholding_outstanding_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Pay | `account.payment.register` | `writeoff_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Payment Methods | `account.payment.method.line` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Payment Methods | `account.payment.method.line` | `payment_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Payment Methods | `account.payment.method.line` | `payment_method_id` | link to one record | Payment Methods | `account.payment.method` | `n : 1` | `not declared` | stored |
| Payment Methods | `account.payment.method.line` | `payment_provider_id` | link to one record | Payment Provider | `payment.provider` | `n : 0..1` | `not declared` | stored |
| Payment Terms | `account.payment.term` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Payment Terms | `account.payment.term` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Payment Terms | `account.payment.term` | `line_ids` | list of records | Payment Terms Line | `account.payment.term.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Payment Terms Line | `account.payment.term.line` | `payment_id` | link to one record | Payment Terms | `account.payment.term` | `n : 1` | `cascade` | stored |
| Payments | `account.payment` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Payments | `account.payment` | `available_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `available_partner_bank_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `available_payment_method_line_ids` | list on both sides | Payment Methods | `account.payment.method.line` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Payments | `account.payment` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `destination_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `duplicate_payment_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `force_outstanding_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Payments | `account.payment` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Payments | `account.payment` | `l10n_in_withhold_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Payments | `account.payment` | `l10n_latam_move_check_ids` | list on both sides | Account payment check | `l10n_latam.check` | `0..n : 0..n` | `not declared` | stored |
| Payments | `account.payment` | `l10n_latam_new_check_ids` | list of records | Account payment check | `l10n_latam.check` | `1 : 0..n` | `mirror of the target column` | derived |
| Payments | `account.payment` | `l10n_pl_verification_id` | link to one record | PL Bank Account Verification | `l10n_pl.bank.account.verification` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `outstanding_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `paired_internal_transfer_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `partner_bank_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `restrict` | stored |
| Payments | `account.payment` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `restrict` | stored |
| Payments | `account.payment` | `payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `payment_token_id` | link to one record | Payment Token | `payment.token` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `payment_transaction_id` | link to one record | Payment Transaction | `payment.transaction` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `pos_payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `pos_session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `reconciled_bill_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `reconciled_invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `reconciled_statement_line_ids` | list on both sides | Bank Statement Line | `account.bank.statement.line` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `source_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Payments | `account.payment` | `suitable_payment_token_ids` | list on both sides | Payment Token | `payment.token` | `0..n : 0..n` | `not declared` | derived |
| Payments | `account.payment` | `withholding_line_ids` | list of records | Payment withholding line | `account.payment.withholding.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `line_ids` | list of records | Rules for the reconciliation model | `account.reconcile.model.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `mapped_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `match_journal_ids` | list on both sides | Journal | `account.journal` | `0..n : 0..n` | `not declared` | stored |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `match_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `next_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `not declared` | stored |
| Remake the sequence of Journal Entries. | `account.resequence.wizard` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Rules for the reconciliation model | `account.reconcile.model.line` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `cascade` | stored |
| Rules for the reconciliation model | `account.reconcile.model.line` | `model_id` | link to one record | Preset to create journal entries during a invoices and payments matching | `account.reconcile.model` | `n : 0..1` | `cascade` | stored |
| Rules for the reconciliation model | `account.reconcile.model.line` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Rules for the reconciliation model | `account.reconcile.model.line` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `restrict` | stored |
| Secure Journal Entries | `account.secure.entries.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Secure Journal Entries | `account.secure.entries.wizard` | `move_to_hash_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Secure Journal Entries | `account.secure.entries.wizard` | `not_hashable_unlocked_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Secure Journal Entries | `account.secure.entries.wizard` | `unreconciled_bank_statement_line_ids` | list on both sides | Bank Statement Line | `account.bank.statement.line` | `0..n : 0..n` | `not declared` | derived |
| Tax | `account.tax` | `account_move_line_ids` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `account_reconcile_model_line_ids` | list on both sides | Rules for the reconciliation model | `account.reconcile.model.line` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `cash_basis_transition_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `children_tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Tax | `account.tax` | `country_id` | link to one record | Country | `res.country` | `n : 1` | `not declared` | stored |
| Tax | `account.tax` | `fiscal_position_ids` | list on both sides | Fiscal Position | `account.fiscal.position` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `hr_expense_ids` | list on both sides | Expense | `hr.expense` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `invoice_repartition_line_ids` | list of records | Tax Repartition Line | `account.tax.repartition.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Tax | `account.tax` | `l10n_ar_scale_id` | link to one record | l10n_ar.earnings.scale | `l10n_ar.earnings.scale` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `l10n_ar_state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `restrict` | stored |
| Tax | `account.tax` | `l10n_ar_withholding_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `l10n_hr_tax_category_id` | link to one record | Croatian tax expence categories | `l10n.hr.tax.category` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `l10n_in_section_id` | link to one record | indian section alert | `l10n_in.section.alert` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `l10n_ke_item_code_id` | link to one record | KRA defined codes that justify a given tax rate / exemption | `l10n_ke.item.code` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `l10n_tr_tax_withholding_code_id` | link to one record | Turkish Tax Codes (GIB Codes) | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | `n : 0..1` | `not declared` | stored |
| Tax | `account.tax` | `original_tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `cascade` | stored |
| Tax | `account.tax` | `pos_order_line_ids` | list on both sides | Point of Sale Order Lines | `pos.order.line` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `purchase_order_line_ids` | list on both sides | Purchase Order Line | `purchase.order.line` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `refund_repartition_line_ids` | list of records | Tax Repartition Line | `account.tax.repartition.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Tax | `account.tax` | `repartition_line_ids` | list of records | Tax Repartition Line | `account.tax.repartition.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Tax | `account.tax` | `replacing_tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Tax | `account.tax` | `tax_group_id` | link to one record | Tax Group | `account.tax.group` | `n : 1` | `not declared` | stored |
| Tax | `account.tax` | `withholding_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Tax Group | `account.tax.group` | `advance_tax_payment_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Tax Group | `account.tax.group` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Tax Group | `account.tax.group` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Tax Group | `account.tax.group` | `tax_payable_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Tax Group | `account.tax.group` | `tax_receivable_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Tax Repartition Line | `account.tax.repartition.line` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Tax Repartition Line | `account.tax.repartition.line` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Tax Repartition Line | `account.tax.repartition.line` | `tag_ids` | list on both sides | Account Tag | `account.account.tag` | `0..n : 0..n` | `restrict` | stored |
| Tax Repartition Line | `account.tax.repartition.line` | `tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `cascade` | stored |
| Validate Account Move | `validate.account.move` | `abnormal_amount_partner_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Validate Account Move | `validate.account.move` | `abnormal_date_partner_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Validate Account Move | `validate.account.move` | `move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |

### 3.16 Payments and Bank Reconciliation

Payments, payment methods and method lines, outstanding accounts, bank statements and statement lines, reconciliation models and their lines, partial and full reconciliation.

Specified in [`../domains/payments-and-bank-reconciliation/`](../domains/payments-and-bank-reconciliation/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account payment check | `l10n_latam.check` | `l10n_latam_check` | Persistent record with 11 stored columns; belongs to Payments; states of `issue_state`: Handed, Debited, Voided; referenced by 3 relation fields. | Third Party and Deferred/Electronic Checks Management |

#### Interactive assistant entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `l10n_latam_payment_mass_transfer` | Interactive assistant with 3 stored columns; company scoped. | Third Party and Deferred/Electronic Checks Management |
| Payment register check | `l10n_latam.payment.register.check` | `l10n_latam_payment_register_check` | Interactive assistant with 6 stored columns; belongs to Pay. | Third Party and Deferred/Electronic Checks Management |

#### Relationships (11)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Account payment check | `l10n_latam.check` | `bank_id` | link to one record | Bank | `res.bank` | `n : 0..1` | `not declared` | stored |
| Account payment check | `l10n_latam.check` | `current_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Account payment check | `l10n_latam.check` | `operation_ids` | list on both sides | Payments | `account.payment` | `0..n : 0..n` | `not declared` | stored |
| Account payment check | `l10n_latam.check` | `outstanding_line_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | stored |
| Account payment check | `l10n_latam.check` | `payment_id` | link to one record | Payments | `account.payment` | `n : 1` | `cascade` | stored |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `check_ids` | list on both sides | Account payment check | `l10n_latam.check` | `0..n : 0..n` | `not declared` | stored |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `destination_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Checks Mass Transfers | `l10n_latam.payment.mass.transfer` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Payment register check | `l10n_latam.payment.register.check` | `bank_id` | link to one record | Bank | `res.bank` | `n : 0..1` | `not declared` | stored |
| Payment register check | `l10n_latam.payment.register.check` | `payment_register_id` | link to one record | Pay | `account.payment.register` | `n : 1` | `cascade` | stored |

### 3.17 Taxes

The tax engine: computation types, price inclusion, distribution lines, tax groups, tax grids and tags, cash basis, withholding, fiscal positions and their mappings.

Specified in [`../domains/taxes/`](../domains/taxes/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Payment withholding line | `account.payment.withholding.line` | `account_payment_withholding_line` | Persistent record with 17 stored columns; belongs to Payments. | Withholding Tax on Payment |

#### Interactive assistant entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Payment register withholding line | `account.payment.register.withholding.line` | `account_payment_register_withholding_line` | Interactive assistant with 17 stored columns; belongs to Pay. | Withholding Tax on Payment |
| Update Tax Tags Wizard | `account.update.tax.tags.wizard` | `account_update_tax_tags_wizard` | Interactive assistant with 2 stored columns; belongs to Companies; company scoped. | Account - Allow updating tax grids |

#### Shared behaviour entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| withholding line | `account.withholding.line` | `none` | Shared behaviour merged into 2 entities. | Withholding Tax on Payment |

#### Relationships (9)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Payment register withholding line | `account.payment.register.withholding.line` | `payment_register_id` | link to one record | Pay | `account.payment.register` | `n : 1` | `cascade` | stored |
| Payment withholding line | `account.payment.withholding.line` | `payment_id` | link to one record | Payments | `account.payment` | `n : 1` | `cascade` | stored |
| Update Tax Tags Wizard | `account.update.tax.tags.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| withholding line | `account.withholding.line` | `account_id` | link to one record | Account | `account.account` | `n : 1` | `not declared` | derived |
| withholding line | `account.withholding.line` | `comodel_currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | derived |
| withholding line | `account.withholding.line` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | derived |
| withholding line | `account.withholding.line` | `source_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| withholding line | `account.withholding.line` | `source_tax_id` | link to one record | Tax | `account.tax` | `n : 0..1` | `not declared` | derived |
| withholding line | `account.withholding.line` | `tax_id` | link to one record | Tax | `account.tax` | `n : 1` | `not declared` | derived |

### 3.18 Delivery and Shipping

Delivery methods, carriers, price rules, shipping labels and tracking.

Specified in [`../domains/delivery-and-shipping/`](../domains/delivery-and-shipping/).

#### Persistent entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Delivery Price Rules | `delivery.price.rule` | `delivery_price_rule` | Persistent record with 8 stored columns; belongs to Shipping Methods. | Delivery Costs |
| Delivery Zip Prefix | `delivery.zip.prefix` | `delivery_zip_prefix` | Persistent record with 1 stored column; referenced by 1 relation field. | Delivery Costs |
| Shipping Methods | `delivery.carrier` | `delivery_carrier` | Persistent record with 29 stored columns; belongs to Product Variant; owns Delivery Price Rules; company scoped; referenced by 8 relation fields. | Delivery Costs |

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `choose_delivery_carrier` | Interactive assistant with 7 stored columns; belongs to Contact, Sales Order, Shipping Methods; company scoped. | Delivery Costs |

#### Relationships (18)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `available_carrier_ids` | list on both sides | Shipping Methods | `delivery.carrier` | `0..n : 0..n` | `not declared` | derived |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `carrier_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 1` | `not declared` | stored |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `cascade` | stored |
| Delivery Carrier Selection Wizard | `choose.delivery.carrier` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | derived |
| Delivery Price Rules | `delivery.price.rule` | `carrier_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 1` | `cascade` | stored |
| Shipping Methods | `delivery.carrier` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `excluded_tag_ids` | list on both sides | Product Tag | `product.tag` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `l10n_ro_edi_stock_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `must_have_tag_ids` | list on both sides | Product Tag | `product.tag` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `price_rule_ids` | list of records | Delivery Price Rules | `delivery.price.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Shipping Methods | `delivery.carrier` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `restrict` | stored |
| Shipping Methods | `delivery.carrier` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `warehouse_ids` | list on both sides | Warehouse | `stock.warehouse` | `0..n : 0..n` | `not declared` | stored |
| Shipping Methods | `delivery.carrier` | `zip_prefix_ids` | list on both sides | Delivery Zip Prefix | `delivery.zip.prefix` | `0..n : 0..n` | `not declared` | stored |

### 3.19 Inventory Operations

Warehouses, locations, operation types, transfers, stock moves and move lines, reservations, quantities on hand, lots and serial numbers, packages and package types, batch transfers, putaway and removal strategies, scrap and inventory adjustments.

Specified in [`../domains/inventory-operations/`](../domains/inventory-operations/).

#### Persistent entities (22)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | Persistent record with 27 stored columns; belongs to Companies; owns Document object for tracking CIUS-RO extensible markup language sent to E-Factura, Product Moves (Stock Move Line), Stock Move and 1 further collections; lifecycle states Draft, In progress, Done, Cancelled; 2 state fields in all; company scoped; referenced by 4 relation fields. | Warehouse Management: Batch Transfer |
| Inventory Locations | `stock.location` | `stock_location` | Persistent record with 16 stored columns; owns Contact, Inventory Locations, Product Moves (Stock Move Line) and 3 further collections; company scoped; referenced by 66 relation fields. | Inventory |
| Inventory Routes | `stock.route` | `stock_route` | Persistent record with 12 stored columns; owns Stock Rule, Warehouse; company scoped; referenced by 21 relation fields. | Inventory |
| Lot/Serial | `stock.lot` | `stock_lot` | Persistent record with 13 stored columns; belongs to Product Variant; owns Quants; company scoped; referenced by 13 relation fields. | Inventory |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `stock_warehouse_orderpoint` | Persistent record with 17 stored columns; belongs to Companies, Inventory Locations, Product Variant and 1 further required links; owns Inventory Locations; company scoped; referenced by 5 relation fields. | Inventory |
| Package | `stock.package` | `stock_package` | Persistent record with 10 stored columns; owns Package, Product Moves (Stock Move Line), Quants; company scoped; referenced by 15 relation fields. | Inventory |
| Picking Type | `stock.picking.type` | `stock_picking_type` | Persistent record with 75 stored columns; belongs to Companies, Inventory Locations; company scoped; referenced by 29 relation fields. | Inventory |
| Product Moves (Stock Move Line) | `stock.move.line` | `stock_move_line` | Persistent record with 23 stored columns; belongs to Companies, Inventory Locations, Product Unit of Measure; carries the state field `state`; company scoped; referenced by 6 relation fields. | Inventory |
| Putaway Rule | `stock.putaway.rule` | `stock_putaway_rule` | Persistent record with 9 stored columns; belongs to Companies, Inventory Locations; company scoped. | Inventory |
| Quants | `stock.quant` | `stock_quant` | Persistent record with 17 stored columns; belongs to Inventory Locations, Product Variant; referenced by 8 relation fields. | Inventory |
| Reference between stock documents | `stock.reference` | `stock_reference` | Persistent record with 1 stored column; referenced by 7 relation fields. | Inventory |
| Removal Strategy | `product.removal` | `product_removal` | Persistent record with 2 stored columns; referenced by 2 relation fields. | Inventory |
| Scrap | `stock.scrap` | `stock_scrap` | Persistent record with 18 stored columns; belongs to Companies, Inventory Locations, Product Unit of Measure and 1 further required links; owns Stock Move; lifecycle states Draft, Done; company scoped; referenced by 2 relation fields. | Inventory |
| Scrap Reason Tag | `stock.scrap.reason.tag` | `stock_scrap_reason_tag` | Persistent record with 3 stored columns; referenced by 1 relation field. | Inventory |
| Stock Move | `stock.move` | `stock_move` | Persistent record with 65 stored columns; belongs to Companies, Inventory Locations, Product Unit of Measure and 1 further required links; owns Package, Product Moves (Stock Move Line), Purchase Requisition Line and 2 further collections; lifecycle states New, Waiting Another Move, Waiting, Partially Available, Available, Done, Cancelled; company scoped; referenced by 12 relation fields. | Inventory |
| Stock Package History | `stock.package.history` | `stock_package_history` | Persistent record with 10 stored columns; belongs to Companies, Package; owns Product Moves (Stock Move Line); company scoped; referenced by 2 relation fields. | Inventory |
| Stock package type | `stock.package.type` | `stock_package_type` | Persistent record with 13 stored columns; owns Storage Category Capacity; company scoped; referenced by 6 relation fields. | Inventory |
| Stock Rule | `stock.rule` | `stock_rule` | Persistent record with 19 stored columns; belongs to Inventory Locations, Inventory Routes, Picking Type; company scoped; referenced by 12 relation fields. | Inventory |
| Storage Category | `stock.storage.category` | `stock_storage_category` | Persistent record with 4 stored columns; owns Inventory Locations, Storage Category Capacity; company scoped; referenced by 3 relation fields. | Inventory |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | Persistent record with 4 stored columns; belongs to Storage Category; company scoped. | Inventory |
| Transfer | `stock.picking` | `stock_picking` | Persistent record with 64 stored columns; belongs to Inventory Locations, Picking Type; owns Attachment, Document object for tracking CIUS-RO extensible markup language sent to E-Factura, Electronic Waybill and 5 further collections; lifecycle states Draft, Waiting Another Operation, Waiting, Ready, Done, Cancelled; 4 state fields in all; company scoped; referenced by 24 relation fields. | Inventory |
| Warehouse | `stock.warehouse` | `stock_warehouse` | Persistent record with 48 stored columns; belongs to Companies, Inventory Locations; owns Inventory Routes; company scoped; referenced by 26 relation fields. | Inventory |

#### Interactive assistant entities (24)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Backorder Confirmation | `stock.backorder.confirmation` | `stock_backorder_confirmation` | Interactive assistant with 1 stored column; owns Backorder Confirmation Line; referenced by 1 relation field. | Inventory |
| Backorder Confirmation Line | `stock.backorder.confirmation.line` | `stock_backorder_confirmation_line` | Interactive assistant with 3 stored columns. | Inventory |
| Batch Transfer Lines | `stock.picking.to.batch` | `stock_picking_to_batch` | Interactive assistant with 5 stored columns. | Warehouse Management: Batch Transfer |
| Choose the sheet layout to print lot labels | `lot.label.layout` | `lot_label_layout` | Interactive assistant with 2 stored columns. | Inventory |
| Choose whether to print product or lot/sn labels | `picking.label.type` | `picking_label_type` | Interactive assistant with 1 stored column. | Inventory |
| Confirm Stock text message | `confirm.stock.sms` | `confirm_stock_sms` | Interactive assistant with 0 stored columns. | Stock - text message |
| Conflict in Inventory | `stock.inventory.conflict` | `stock_inventory_conflict` | Interactive assistant with 0 stored columns. | Inventory |
| Inventory Adjustment Reference / Reason | `stock.inventory.adjustment.name` | `stock_inventory_adjustment_name` | Interactive assistant with 3 stored columns. | Inventory |
| Inventory Adjustment Warning | `stock.inventory.warning` | `stock_inventory_warning` | Interactive assistant with 0 stored columns. | Inventory |
| Product Replenish | `product.replenish` | `product_replenish` | Interactive assistant with 11 stored columns; belongs to Product, Product Unit of Measure, Product Variant and 1 further required links; company scoped. | Inventory |
| Put In Pack Wizard | `stock.put.in.pack` | `stock_put_in_pack` | Interactive assistant with 5 stored columns. | Inventory |
| Return Picking | `stock.return.picking` | `stock_return_picking` | Interactive assistant with 1 stored column; owns Return Picking Line; referenced by 1 relation field. | Inventory |
| Return Picking Line | `stock.return.picking.line` | `stock_return_picking_line` | Interactive assistant with 5 stored columns; belongs to Product Variant. | Inventory |
| Snooze Orderpoint | `stock.orderpoint.snooze` | `stock_orderpoint_snooze` | Interactive assistant with 2 stored columns. | Inventory |
| Stock Package Destination | `stock.package.destination` | `stock_package_destination` | Interactive assistant with 1 stored column; belongs to Inventory Locations; owns Inventory Locations. | Inventory |
| Stock Quantity History | `stock.quantity.history` | `stock_quantity_history` | Interactive assistant with 1 stored column. | Inventory |
| Stock Quantity Relocation | `stock.quant.relocate` | `stock_quant_relocate` | Interactive assistant with 3 stored columns. | Inventory |
| Stock Request an Inventory Count | `stock.request.count` | `stock_request_count` | Interactive assistant with 2 stored columns. | Inventory |
| Stock Rules report | `stock.rules.report` | `stock_rules_report` | Interactive assistant with 3 stored columns; belongs to Product, Product Variant. | Inventory |
| Stock supplier replenishment information | `stock.replenishment.info` | `stock_replenishment_info` | Interactive assistant with 3 stored columns; owns Stock warehouse replenishment option; referenced by 1 relation field. | Inventory |
| Stock warehouse replenishment option | `stock.replenishment.option` | `stock_replenishment_option` | Interactive assistant with 3 stored columns. | Inventory |
| Traceability Report | `stock.traceability.report` | `stock_traceability_report` | Interactive assistant with 0 stored columns. | Inventory |
| Warn Insufficient Scrap Quantity | `stock.warn.insufficient.qty.scrap` | `stock_warn_insufficient_qty_scrap` | Interactive assistant with 5 stored columns. | Inventory |
| Wave Transfer Lines | `stock.add.to.wave` | `stock_add_to_wave` | Interactive assistant with 3 stored columns. | Warehouse Management: Batch Transfer |

#### Shared behaviour entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Product Replenish Mixin | `stock.replenish.mixin` | `none` | Shared behaviour merged into 1 entity. | Inventory |
| Stock Replenishment Report | `stock.forecasted_product_product` | `none` | Shared behaviour merged into 1 entity. | Inventory |
| Stock Replenishment Report | `stock.forecasted_product_template` | `none` | Shared behaviour definition reused through composition. | Inventory |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | `none` | Shared behaviour merged into 3 entities. | Inventory |

#### Relationships (383)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Backorder Confirmation | `stock.backorder.confirmation` | `backorder_confirmation_line_ids` | list of records | Backorder Confirmation Line | `stock.backorder.confirmation.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Backorder Confirmation | `stock.backorder.confirmation` | `pick_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Backorder Confirmation Line | `stock.backorder.confirmation.line` | `backorder_confirmation_id` | link to one record | Backorder Confirmation | `stock.backorder.confirmation` | `n : 0..1` | `not declared` | stored |
| Backorder Confirmation Line | `stock.backorder.confirmation.line` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `allowed_picking_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Batch Transfer | `stock.picking.batch` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `dock_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `driver_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `l10n_ro_edi_stock_document_ids` | list of records | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Batch Transfer | `stock.picking.batch` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Batch Transfer | `stock.picking.batch` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Batch Transfer | `stock.picking.batch` | `picking_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Batch Transfer | `stock.picking.batch` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `vehicle_category_id` | link to one record | Category of the model | `fleet.vehicle.model.category` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 0..1` | `not declared` | stored |
| Batch Transfer | `stock.picking.batch` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Batch Transfer Lines | `stock.picking.to.batch` | `batch_id` | link to one record | Batch Transfer | `stock.picking.batch` | `n : 0..1` | `not declared` | stored |
| Batch Transfer Lines | `stock.picking.to.batch` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Choose the sheet layout to print lot labels | `lot.label.layout` | `move_line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Choose whether to print product or lot/sn labels | `picking.label.type` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Choose whether to print product or lot/sn labels | `picking.label.type` | `production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Confirm Stock text message | `confirm.stock.sms` | `pick_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Conflict in Inventory | `stock.inventory.conflict` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Conflict in Inventory | `stock.inventory.conflict` | `quant_to_fix_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Inventory Adjustment Reference / Reason | `stock.inventory.adjustment.name` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Inventory Adjustment Warning | `stock.inventory.warning` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Inventory Locations | `stock.location` | `child_ids` | list of records | Inventory Locations | `stock.location` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `child_internal_location_ids` | list on both sides | Inventory Locations | `stock.location` | `0..n : 0..n` | `not declared` | derived |
| Inventory Locations | `stock.location` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `incoming_move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `outgoing_move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `putaway_rule_ids` | list of records | Putaway Rule | `stock.putaway.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `quant_ids` | list of records | Quants | `stock.quant` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `removal_strategy_id` | link to one record | Removal Strategy | `product.removal` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `storage_category_id` | link to one record | Storage Category | `stock.storage.category` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `subcontractor_ids` | list of records | Contact | `res.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Locations | `stock.location` | `valuation_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Inventory Locations | `stock.location` | `warehouse_view_ids` | list of records | Warehouse | `stock.warehouse` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Routes | `stock.route` | `categ_ids` | list on both sides | Product Category | `product.category` | `0..n : 0..n` | `not declared` | stored |
| Inventory Routes | `stock.route` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Inventory Routes | `stock.route` | `product_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Inventory Routes | `stock.route` | `rule_ids` | list of records | Stock Rule | `stock.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Routes | `stock.route` | `supplied_wh_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Inventory Routes | `stock.route` | `supplier_wh_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Inventory Routes | `stock.route` | `warehouse_domain_ids` | list of records | Warehouse | `stock.warehouse` | `1 : 0..n` | `mirror of the target column` | derived |
| Inventory Routes | `stock.route` | `warehouse_ids` | list on both sides | Warehouse | `stock.warehouse` | `0..n : 0..n` | `not declared` | stored |
| Lot/Serial | `stock.lot` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lot/Serial | `stock.lot` | `delivery_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Lot/Serial | `stock.lot` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Lot/Serial | `stock.lot` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `purchase_order_ids` | list on both sides | Purchase Order | `purchase.order` | `0..n : 0..n` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `quant_ids` | list of records | Quants | `stock.quant` | `1 : 0..n` | `mirror of the target column` | derived |
| Lot/Serial | `stock.lot` | `repair_line_ids` | list on both sides | Repair Order | `repair.order` | `0..n : 0..n` | `not declared` | derived |
| Lot/Serial | `stock.lot` | `sale_order_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `Product Category` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `allowed_location_ids` | list of records | Inventory Locations | `stock.location` | `1 : 0..n` | `mirror of the target column` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `allowed_replenishment_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `available_vendor` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `effective_bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `effective_route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `effective_vendor_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `cascade` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `cascade` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `product_uom` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `replenishment_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `not declared` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `rule_ids` | list on both sides | Stock Rule | `stock.rule` | `0..n : 0..n` | `not declared` | derived |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `supplier_id` | link to one record | Supplier Pricelist | `product.supplierinfo` | `n : 0..1` | `not declared` | stored |
| Minimum Inventory Rule | `stock.warehouse.orderpoint` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 1` | `cascade` | stored |
| Package | `stock.package` | `all_children_package_ids` | list of records | Package | `stock.package` | `1 : 0..n` | `mirror of the target column` | derived |
| Package | `stock.package` | `child_package_dest_ids` | list of records | Package | `stock.package` | `1 : 0..n` | `mirror of the target column` | derived |
| Package | `stock.package` | `child_package_ids` | list of records | Package | `stock.package` | `1 : 0..n` | `mirror of the target column` | derived |
| Package | `stock.package` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Package | `stock.package` | `contained_quant_ids` | list of records | Quants | `stock.quant` | `1 : 0..n` | `mirror of the target column` | derived |
| Package | `stock.package` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | derived |
| Package | `stock.package` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Package | `stock.package` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Package | `stock.package` | `outermost_package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | derived |
| Package | `stock.package` | `owner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Package | `stock.package` | `package_dest_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Package | `stock.package` | `package_type_id` | link to one record | Stock package type | `stock.package.type` | `n : 0..1` | `not declared` | stored |
| Package | `stock.package` | `parent_package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Package | `stock.package` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | derived |
| Package | `stock.package` | `quant_ids` | list of records | Quants | `stock.quant` | `1 : 0..n` | `mirror of the target column` | derived |
| Picking Type | `stock.picking.type` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_location_src_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_product_location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_product_location_src_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_recycle_location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `default_remove_location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `dock_ids` | list on both sides | Inventory Locations | `stock.location` | `0..n : 0..n` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `l10n_ar_document_type_id` | link to one record | Latam Document Type | `l10n_latam.document.type` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `l10n_ar_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `l10n_it_ddt_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `return_picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `cascade` | stored |
| Picking Type | `stock.picking.type` | `wave_category_ids` | list on both sides | Product Category | `product.category` | `0..n : 0..n` | `not declared` | stored |
| Picking Type | `stock.picking.type` | `wave_location_ids` | list on both sides | Inventory Locations | `stock.location` | `0..n : 0..n` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Product Moves (Stock Move Line) | `stock.move.line` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `consume_line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `owner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `package_history_id` | link to one record | Stock Package History | `stock.package.history` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `restrict` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | derived |
| Product Moves (Stock Move Line) | `stock.move.line` | `produce_line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `quant_id` | link to one record | Quants | `stock.quant` | `n : 0..1` | `not declared` | derived |
| Product Moves (Stock Move Line) | `stock.move.line` | `result_package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `restrict` | stored |
| Product Moves (Stock Move Line) | `stock.move.line` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Product Replenish | `product.replenish` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Product Replenish | `product.replenish` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Product Replenish | `product.replenish` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Product Replenish | `product.replenish` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `not declared` | stored |
| Product Replenish | `product.replenish` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Product Replenish | `product.replenish` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 1` | `not declared` | stored |
| Product Replenish Mixin | `stock.replenish.mixin` | `allowed_route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | derived |
| Product Replenish Mixin | `stock.replenish.mixin` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | derived |
| Product Replenish Mixin | `stock.replenish.mixin` | `route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `not declared` | derived |
| Product Replenish Mixin | `stock.replenish.mixin` | `supplier_id` | link to one record | Supplier Pricelist | `product.supplierinfo` | `n : 0..1` | `not declared` | derived |
| Put In Pack Wizard | `stock.put.in.pack` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Put In Pack Wizard | `stock.put.in.pack` | `move_line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Put In Pack Wizard | `stock.put.in.pack` | `origin_package_ids` | list on both sides | Package | `stock.package` | `0..n : 0..n` | `not declared` | stored |
| Put In Pack Wizard | `stock.put.in.pack` | `package_ids` | list on both sides | Package | `stock.package` | `0..n : 0..n` | `not declared` | stored |
| Put In Pack Wizard | `stock.put.in.pack` | `package_type_id` | link to one record | Stock package type | `stock.package.type` | `n : 0..1` | `not declared` | stored |
| Put In Pack Wizard | `stock.put.in.pack` | `result_package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Putaway Rule | `stock.putaway.rule` | `category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `cascade` | stored |
| Putaway Rule | `stock.putaway.rule` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Putaway Rule | `stock.putaway.rule` | `location_in_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `cascade` | stored |
| Putaway Rule | `stock.putaway.rule` | `location_out_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `cascade` | stored |
| Putaway Rule | `stock.putaway.rule` | `package_type_ids` | list on both sides | Stock package type | `stock.package.type` | `0..n : 0..n` | `not declared` | stored |
| Putaway Rule | `stock.putaway.rule` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Putaway Rule | `stock.putaway.rule` | `storage_category_id` | link to one record | Storage Category | `stock.storage.category` | `n : 0..1` | `cascade` | stored |
| Quants | `stock.quant` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Quants | `stock.quant` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `restrict` | stored |
| Quants | `stock.quant` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `restrict` | stored |
| Quants | `stock.quant` | `owner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Quants | `stock.quant` | `package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `restrict` | stored |
| Quants | `stock.quant` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `restrict` | stored |
| Quants | `stock.quant` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Quants | `stock.quant` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Quants | `stock.quant` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Quants | `stock.quant` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Reference between stock documents | `stock.reference` | `move_ids` | list on both sides | Stock Move | `stock.move` | `0..n : 0..n` | `not declared` | stored |
| Reference between stock documents | `stock.reference` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | derived |
| Reference between stock documents | `stock.reference` | `pos_order_ids` | list on both sides | Point of Sale Orders | `pos.order` | `0..n : 0..n` | `not declared` | stored |
| Reference between stock documents | `stock.reference` | `production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Reference between stock documents | `stock.reference` | `purchase_ids` | list on both sides | Purchase Order | `purchase.order` | `0..n : 0..n` | `not declared` | stored |
| Reference between stock documents | `stock.reference` | `sale_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | stored |
| Return Picking | `stock.return.picking` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Return Picking | `stock.return.picking` | `product_return_moves` | list of records | Return Picking Line | `stock.return.picking.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Return Picking Line | `stock.return.picking.line` | `move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Return Picking Line | `stock.return.picking.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Return Picking Line | `stock.return.picking.line` | `uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Return Picking Line | `stock.return.picking.line` | `wizard_id` | link to one record | Return Picking | `stock.return.picking` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Scrap | `stock.scrap` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Scrap | `stock.scrap` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Scrap | `stock.scrap` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Scrap | `stock.scrap` | `owner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Scrap | `stock.scrap` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Scrap | `stock.scrap` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Scrap | `stock.scrap` | `scrap_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Scrap | `stock.scrap` | `scrap_reason_tag_ids` | list on both sides | Scrap Reason Tag | `stock.scrap.reason.tag` | `0..n : 0..n` | `not declared` | stored |
| Scrap | `stock.scrap` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Snooze Orderpoint | `stock.orderpoint.snooze` | `orderpoint_ids` | list on both sides | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `allowed_operation_ids` | list of records | Work Center Usage | `mrp.routing.workcenter` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Move | `stock.move` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Stock Move | `stock.move` | `analytic_account_line_ids` | list on both sides | Analytic Line | `account.analytic.line` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `bom_line_id` | link to one record | Bill of Material Line | `mrp.bom.line` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `byproduct_id` | link to one record | Byproduct | `mrp.bom.byproduct` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Stock Move | `stock.move` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Stock Move | `stock.move` | `consume_unbuild_id` | link to one record | Unbuild Order | `mrp.unbuild` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `created_production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `created_purchase_line_ids` | list on both sides | Purchase Order Line | `purchase.order.line` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `ewaybill_tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Stock Move | `stock.move` | `location_final_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Stock Move | `stock.move` | `lot_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | derived |
| Stock Move | `stock.move` | `move_dest_ids` | list on both sides | Stock Move | `stock.move` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Move | `stock.move` | `move_orig_ids` | list on both sides | Stock Move | `stock.move` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `never_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `operation_id` | link to one record | Work Center Usage | `mrp.routing.workcenter` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `order_finished_lot_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | derived |
| Stock Move | `stock.move` | `orderpoint_id` | link to one record | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `origin_returned_move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `package_ids` | list of records | Package | `stock.package` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Move | `stock.move` | `packaging_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `product_category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | derived |
| Stock Move | `stock.move` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Stock Move | `stock.move` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Stock Move | `stock.move` | `product_uom` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Stock Move | `stock.move` | `production_group_id` | link to one record | Production Group | `mrp.production.group` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `cascade` | stored |
| Stock Move | `stock.move` | `purchase_line_id` | link to one record | Purchase Order Line | `purchase.order.line` | `n : 0..1` | `set null` | stored |
| Stock Move | `stock.move` | `raw_material_production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `cascade` | stored |
| Stock Move | `stock.move` | `reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `repair_id` | link to one record | Repair Order | `repair.order` | `n : 0..1` | `cascade` | stored |
| Stock Move | `stock.move` | `requisition_line_ids` | list of records | Purchase Requisition Line | `purchase.requisition.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Move | `stock.move` | `restrict_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `returned_move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Move | `stock.move` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Stock Move | `stock.move` | `rule_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `restrict` | stored |
| Stock Move | `stock.move` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `scrap_id` | link to one record | Scrap | `stock.scrap` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `unbuild_id` | link to one record | Unbuild Order | `mrp.unbuild` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Stock Move | `stock.move` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Stock Package Destination | `stock.package.destination` | `filtered_location` | list of records | Inventory Locations | `stock.location` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Package Destination | `stock.package.destination` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Stock Package Destination | `stock.package.destination` | `move_line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Package History | `stock.package.history` | `outermost_dest_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `package_id` | link to one record | Package | `stock.package` | `n : 1` | `cascade` | stored |
| Stock Package History | `stock.package.history` | `package_type_id` | link to one record | Stock package type | `stock.package.type` | `n : 0..1` | `not declared` | derived |
| Stock Package History | `stock.package.history` | `parent_dest_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `parent_orig_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Stock Package History | `stock.package.history` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Stock Quantity Relocation | `stock.quant.relocate` | `dest_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Stock Quantity Relocation | `stock.quant.relocate` | `dest_package_id` | link to one record | Package | `stock.package` | `n : 0..1` | `not declared` | stored |
| Stock Quantity Relocation | `stock.quant.relocate` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Stock Request an Inventory Count | `stock.request.count` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | stored |
| Stock Request an Inventory Count | `stock.request.count` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `location_src_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `partner_address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Stock Rule | `stock.rule` | `route_id` | link to one record | Inventory Routes | `stock.route` | `n : 1` | `cascade` | stored |
| Stock Rule | `stock.rule` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Stock Rules report | `stock.rules.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Stock Rules report | `stock.rules.report` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `not declared` | stored |
| Stock Rules report | `stock.rules.report` | `so_route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Stock Rules report | `stock.rules.report` | `warehouse_ids` | list on both sides | Warehouse | `stock.warehouse` | `0..n : 0..n` | `not declared` | stored |
| Stock package type | `stock.package.type` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Stock package type | `stock.package.type` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Stock package type | `stock.package.type` | `sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Stock package type | `stock.package.type` | `storage_category_capacity_ids` | list of records | Storage Category Capacity | `stock.storage.category.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock supplier replenishment information | `stock.replenishment.info` | `bom_ids` | list on both sides | Bill of Material | `mrp.bom` | `0..n : 0..n` | `not declared` | stored |
| Stock supplier replenishment information | `stock.replenishment.info` | `orderpoint_id` | link to one record | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `n : 0..1` | `not declared` | stored |
| Stock supplier replenishment information | `stock.replenishment.info` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Stock supplier replenishment information | `stock.replenishment.info` | `supplierinfo_ids` | list on both sides | Supplier Pricelist | `product.supplierinfo` | `0..n : 0..n` | `not declared` | stored |
| Stock supplier replenishment information | `stock.replenishment.info` | `wh_replenishment_option_ids` | list of records | Stock warehouse replenishment option | `stock.replenishment.option` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock warehouse replenishment option | `stock.replenishment.option` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | derived |
| Stock warehouse replenishment option | `stock.replenishment.option` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Stock warehouse replenishment option | `stock.replenishment.option` | `replenishment_info_id` | link to one record | Stock supplier replenishment information | `stock.replenishment.info` | `n : 0..1` | `not declared` | stored |
| Stock warehouse replenishment option | `stock.replenishment.option` | `route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `not declared` | stored |
| Stock warehouse replenishment option | `stock.replenishment.option` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Storage Category | `stock.storage.category` | `capacity_ids` | list of records | Storage Category Capacity | `stock.storage.category.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Storage Category | `stock.storage.category` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Storage Category | `stock.storage.category` | `location_ids` | list of records | Inventory Locations | `stock.location` | `1 : 0..n` | `mirror of the target column` | derived |
| Storage Category | `stock.storage.category` | `package_capacity_ids` | list of records | Storage Category Capacity | `stock.storage.category.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Storage Category | `stock.storage.category` | `product_capacity_ids` | list of records | Storage Category Capacity | `stock.storage.category.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Storage Category Capacity | `stock.storage.category.capacity` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Storage Category Capacity | `stock.storage.category.capacity` | `package_type_id` | link to one record | Stock package type | `stock.package.type` | `n : 0..1` | `cascade` | stored |
| Storage Category Capacity | `stock.storage.category.capacity` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Storage Category Capacity | `stock.storage.category.capacity` | `storage_category_id` | link to one record | Storage Category | `stock.storage.category` | `n : 1` | `cascade` | stored |
| Transfer | `stock.picking` | `allowed_carrier_ids` | list on both sides | Shipping Methods | `delivery.carrier` | `0..n : 0..n` | `not declared` | derived |
| Transfer | `stock.picking` | `backorder_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `backorder_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `batch_id` | link to one record | Batch Transfer | `stock.picking.batch` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `carrier_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_in_ewaybill_ids` | list of records | Electronic Waybill | `l10n.in.ewaybill` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `l10n_ro_edi_stock_document_ids` | list of records | Document object for tracking CIUS-RO extensible markup language sent to E-Factura | `l10n_ro_edi.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `l10n_tr_nilvera_buyer_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_nilvera_buyer_originator_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_nilvera_carrier_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_nilvera_driver_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_nilvera_seller_supplier_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_nilvera_trailer_plate_ids` | list on both sides | GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `0..n : 0..n` | `not declared` | stored |
| Transfer | `stock.picking` | `l10n_tr_vehicle_plate` | link to one record | GİB Plate numbers | `l10n_tr.nilvera.trailer.plate` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Transfer | `stock.picking` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Transfer | `stock.picking` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `owner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `package_history_ids` | list on both sides | Stock Package History | `stock.package.history` | `0..n : 0..n` | `not declared` | stored |
| Transfer | `stock.picking` | `partner_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Transfer | `stock.picking` | `pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `pos_session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `production_group_id` | link to one record | Production Group | `mrp.production.group` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `production_ids` | list of records | Manufacturing Order | `mrp.production` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `purchase_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | derived |
| Transfer | `stock.picking` | `repair_ids` | list of records | Repair Order | `repair.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `return_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `return_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `return_label_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Transfer | `stock.picking` | `sale_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Transfer | `stock.picking` | `warehouse_address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Transfer | `stock.picking` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `buy_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `delivery_route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `restrict` | stored |
| Warehouse | `stock.warehouse` | `in_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `int_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `lot_stock_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `manu_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `manufacture_mto_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `manufacture_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `mto_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `opening_hours` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `out_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pack_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pbm_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pbm_mto_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pbm_route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `restrict` | stored |
| Warehouse | `stock.warehouse` | `pbm_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pick_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `pos_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `qc_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `reception_route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `restrict` | stored |
| Warehouse | `stock.warehouse` | `repair_mto_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `repair_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `resupply_route_ids` | list of records | Inventory Routes | `stock.route` | `1 : 0..n` | `mirror of the target column` | derived |
| Warehouse | `stock.warehouse` | `resupply_wh_ids` | list on both sides | Warehouse | `stock.warehouse` | `0..n : 0..n` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `sam_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `sam_rule_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `sam_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `store_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_dropshipping_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_mto_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_pull_id` | link to one record | Stock Rule | `stock.rule` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_resupply_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `restrict` | stored |
| Warehouse | `stock.warehouse` | `subcontracting_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `view_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `wh_input_stock_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `wh_output_stock_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `wh_pack_stock_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `wh_qc_stock_loc_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Warehouse | `stock.warehouse` | `xdock_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | derived |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | derived |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | `quant_ids` | list on both sides | Quants | `stock.quant` | `0..n : 0..n` | `not declared` | derived |
| Warn Insufficient Scrap Quantity | `stock.warn.insufficient.qty.scrap` | `scrap_id` | link to one record | Scrap | `stock.scrap` | `n : 0..1` | `not declared` | stored |
| Wave Transfer Lines | `stock.add.to.wave` | `line_ids` | list on both sides | Product Moves (Stock Move Line) | `stock.move.line` | `0..n : 0..n` | `not declared` | stored |
| Wave Transfer Lines | `stock.add.to.wave` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Wave Transfer Lines | `stock.add.to.wave` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Wave Transfer Lines | `stock.add.to.wave` | `wave_id` | link to one record | Batch Transfer | `stock.picking.batch` | `n : 0..1` | `not declared` | stored |

### 3.20 Inventory Valuation and Costing

Valuation layers, costing methods, automated and manual valuation, stock accounts, cost of goods sold, landed costs, price difference handling and revaluation.

Specified in [`../domains/inventory-valuation-and-costing/`](../domains/inventory-valuation-and-costing/).

#### Persistent entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Product Value | `product.value` | `product_value` | Persistent record with 8 stored columns; belongs to Companies, User; company scoped. | WMS Accounting |
| Stock Landed Cost | `stock.landed.cost` | `stock_landed_cost` | Persistent record with 10 stored columns; belongs to Companies, Journal; owns Stock Landed Cost Line, Valuation Adjustment Lines; lifecycle states Draft, Posted, Cancelled; company scoped; referenced by 2 relation fields. | WMS Landed Costs |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `stock_landed_cost_lines` | Persistent record with 6 stored columns; belongs to Product Variant, Stock Landed Cost; referenced by 1 relation field. | WMS Landed Costs |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `stock_valuation_adjustment_lines` | Persistent record with 11 stored columns; belongs to Product Variant, Stock Landed Cost. | WMS Landed Costs |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Stock average cost Justifier | `stock.avco.report` | `none` | Shared behaviour definition reused through composition. | WMS Accounting |
| Stock Valuation | `stock_account.stock.valuation.report` | `none` | Shared behaviour definition reused through composition. | WMS Accounting |

#### Relationships (28)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Product Value | `product.value` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Product Value | `product.value` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product Value | `product.value` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | stored |
| Product Value | `product.value` | `move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Product Value | `product.value` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Product Value | `product.value` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `account_journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `cost_lines` | list of records | Stock Landed Cost Line | `stock.landed.cost.lines` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Landed Cost | `stock.landed.cost` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Stock Landed Cost | `stock.landed.cost` | `mrp_production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Stock Landed Cost | `stock.landed.cost` | `valuation_adjustment_lines` | list of records | Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `1 : 0..n` | `mirror of the target column` | derived |
| Stock Landed Cost | `stock.landed.cost` | `vendor_bill_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `cost_id` | link to one record | Stock Landed Cost | `stock.landed.cost` | `n : 1` | `cascade` | stored |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Stock Landed Cost Line | `stock.landed.cost.lines` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Stock average cost Justifier | `stock.avco.report` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Stock average cost Justifier | `stock.avco.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Stock average cost Justifier | `stock.avco.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Stock average cost Justifier | `stock.avco.report` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `cost_id` | link to one record | Stock Landed Cost | `stock.landed.cost` | `n : 1` | `cascade` | stored |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `cost_line_id` | link to one record | Stock Landed Cost Line | `stock.landed.cost.lines` | `n : 0..1` | `not declared` | stored |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Valuation Adjustment Lines | `stock.valuation.adjustment.lines` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |

### 3.21 Manufacturing

Bills of materials, manufacturing orders, work orders, work centres, operations, productivity records, by-products, unbuild orders and subcontracting.

Specified in [`../domains/manufacturing/`](../domains/manufacturing/).

#### Persistent entities (14)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Add tag for the workcenter | `mrp.workcenter.tag` | `mrp_workcenter_tag` | Persistent record with 2 stored columns; referenced by 1 relation field. | Manufacturing |
| Bill of Material | `mrp.bom` | `mrp_bom` | Persistent record with 18 stored columns; belongs to Product, Product Unit of Measure; owns Bill of Material Line, Byproduct, Work Center Usage; company scoped; referenced by 15 relation fields. | Manufacturing |
| Bill of Material Line | `mrp.bom.line` | `mrp_bom_line` | Persistent record with 9 stored columns; belongs to Bill of Material, Product Unit of Measure, Product Variant; owns Bill of Material Line, Work Center Usage; referenced by 1 relation field. | Manufacturing |
| Byproduct | `mrp.bom.byproduct` | `mrp_bom_byproduct` | Persistent record with 8 stored columns; belongs to Product Unit of Measure, Product Variant; owns Work Center Usage; referenced by 1 relation field. | Manufacturing |
| Manufacturing Order | `mrp.production` | `mrp_production` | Persistent record with 36 stored columns; belongs to Companies, Inventory Locations, Picking Type and 2 further required links; owns Product Moves (Stock Move Line), Scrap, Stock Move and 2 further collections; lifecycle states Draft, Confirmed, In Progress, To Close, Done, Cancelled; 3 state fields in all; company scoped; referenced by 24 relation fields. | Manufacturing |
| manufacturing Workorder productivity losses | `mrp.workcenter.productivity.loss.type` | `mrp_workcenter_productivity_loss_type` | Persistent record with 1 stored column; referenced by 1 relation field. | Manufacturing |
| Production Group | `mrp.production.group` | `mrp_production_group` | Persistent record with 1 stored column; owns Manufacturing Order; referenced by 5 relation fields. | Manufacturing |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | Persistent record with 11 stored columns; belongs to Companies, Inventory Locations, Product Unit of Measure and 1 further required links; owns Stock Move; lifecycle states Draft, Done; company scoped; referenced by 3 relation fields. | Manufacturing |
| Work Center | `mrp.workcenter` | `mrp_workcenter` | Persistent record with 17 stored columns; belongs to Currency; owns Work Center Capacity, Work Center Usage, Work Order and 1 further collections; states of `working_state`: Normal, Blocked, In Progress; referenced by 7 relation fields. | Manufacturing |
| Work Center Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | Persistent record with 6 stored columns; belongs to Product Unit of Measure, Work Center. | Manufacturing |
| Work Center Usage | `mrp.routing.workcenter` | `mrp_routing_workcenter` | Persistent record with 9 stored columns; belongs to Bill of Material, Work Center; owns Work Order; company scoped; referenced by 6 relation fields. | Manufacturing |
| Work Order | `mrp.workorder` | `mrp_workorder` | Persistent record with 20 stored columns; belongs to Manufacturing Order, Work Center; owns Product Moves (Stock Move Line), Scrap, Stock Move and 2 further collections; lifecycle states Blocked, To Do, In Progress, Finished, Cancelled; 3 state fields in all; referenced by 8 relation fields. | Manufacturing |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | Persistent record with 10 stored columns; belongs to Companies, Work Center, Workcenter Productivity Losses; company scoped. | Manufacturing |
| Workcenter Productivity Losses | `mrp.workcenter.productivity.loss` | `mrp_workcenter_productivity_loss` | Persistent record with 4 stored columns; referenced by 1 relation field. | Manufacturing |

#### Interactive assistant entities (11)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `mrp_account_wip_accounting_line` | Interactive assistant with 6 stored columns. | Accounting - manufacturing |
| Assign serial numbers to production order | `mrp.production.serials` | `mrp_production_serials` | Interactive assistant with 5 stored columns. | Manufacturing |
| Backorder Confirmation Line | `mrp.production.backorder.line` | `mrp_production_backorder_line` | Interactive assistant with 3 stored columns; belongs to Manufacturing Order, Wizard to mark as done or create back order. | Manufacturing |
| Line of issue consumption | `mrp.consumption.warning.line` | `mrp_consumption_warning_line` | Interactive assistant with 5 stored columns; belongs to Manufacturing Order, Product Variant, Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials). | Manufacturing |
| Split Production Detail | `mrp.production.split.line` | `mrp_production_split_line` | Interactive assistant with 4 stored columns; belongs to Wizard to Split a Production. | Manufacturing |
| Warn Insufficient Unbuild Quantity | `stock.warn.insufficient.qty.unbuild` | `stock_warn_insufficient_qty_unbuild` | Interactive assistant with 5 stored columns. | Manufacturing |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `mrp_consumption_warning` | Interactive assistant with 0 stored columns; owns Line of issue consumption; referenced by 1 relation field. | Manufacturing |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `mrp_production_backorder` | Interactive assistant with 0 stored columns; owns Backorder Confirmation Line; referenced by 1 relation field. | Manufacturing |
| Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `mrp_account_wip_accounting` | Interactive assistant with 4 stored columns; belongs to Journal; owns Account move line to be created when posting work in progress account move; referenced by 1 relation field. | Accounting - manufacturing |
| Wizard to Split a Production | `mrp.production.split` | `mrp_production_split` | Interactive assistant with 2 stored columns; owns Split Production Detail; referenced by 1 relation field. | Manufacturing |
| Wizard to Split Multiple Productions | `mrp.production.split.multi` | `mrp_production_split_multi` | Interactive assistant with 0 stored columns; owns Wizard to Split a Production; referenced by 1 relation field. | Manufacturing |

#### Relationships (148)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `wip_accounting_id` | link to one record | Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `n : 0..1` | `not declared` | stored |
| Assign serial numbers to production order | `mrp.production.serials` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Assign serial numbers to production order | `mrp.production.serials` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Backorder Confirmation Line | `mrp.production.backorder.line` | `mrp_production_backorder_id` | link to one record | Wizard to mark as done or create back order | `mrp.production.backorder` | `n : 1` | `cascade` | stored |
| Backorder Confirmation Line | `mrp.production.backorder.line` | `mrp_production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 1` | `cascade` | stored |
| Bill of Material | `mrp.bom` | `bom_line_ids` | list of records | Bill of Material Line | `mrp.bom.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Bill of Material | `mrp.bom` | `byproduct_ids` | list of records | Byproduct | `mrp.bom.byproduct` | `1 : 0..n` | `mirror of the target column` | derived |
| Bill of Material | `mrp.bom` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `operation_ids` | list of records | Work Center Usage | `mrp.routing.workcenter` | `1 : 0..n` | `mirror of the target column` | derived |
| Bill of Material | `mrp.bom` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `possible_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | derived |
| Bill of Material | `mrp.bom` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Bill of Material | `mrp.bom` | `subcontractor_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Bill of Material Line | `mrp.bom.line` | `allowed_operation_ids` | list of records | Work Center Usage | `mrp.routing.workcenter` | `1 : 0..n` | `mirror of the target column` | derived |
| Bill of Material Line | `mrp.bom.line` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 1` | `cascade` | stored |
| Bill of Material Line | `mrp.bom.line` | `bom_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Bill of Material Line | `mrp.bom.line` | `child_bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | derived |
| Bill of Material Line | `mrp.bom.line` | `child_line_ids` | list of records | Bill of Material Line | `mrp.bom.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Bill of Material Line | `mrp.bom.line` | `operation_id` | link to one record | Work Center Usage | `mrp.routing.workcenter` | `n : 0..1` | `not declared` | stored |
| Bill of Material Line | `mrp.bom.line` | `parent_product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Bill of Material Line | `mrp.bom.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Bill of Material Line | `mrp.bom.line` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | stored |
| Bill of Material Line | `mrp.bom.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Byproduct | `mrp.bom.byproduct` | `allowed_operation_ids` | list of records | Work Center Usage | `mrp.routing.workcenter` | `1 : 0..n` | `mirror of the target column` | derived |
| Byproduct | `mrp.bom.byproduct` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `cascade` | stored |
| Byproduct | `mrp.bom.byproduct` | `bom_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Byproduct | `mrp.bom.byproduct` | `operation_id` | link to one record | Work Center Usage | `mrp.routing.workcenter` | `n : 0..1` | `not declared` | stored |
| Byproduct | `mrp.bom.byproduct` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Byproduct | `mrp.bom.byproduct` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Line of issue consumption | `mrp.consumption.warning.line` | `mrp_consumption_warning_id` | link to one record | Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `n : 1` | `cascade` | stored |
| Line of issue consumption | `mrp.consumption.warning.line` | `mrp_production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 1` | `cascade` | stored |
| Line of issue consumption | `mrp.consumption.warning.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Line of issue consumption | `mrp.consumption.warning.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `all_move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `all_move_raw_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `bom_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `finished_move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `location_final_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `location_src_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `lot_producing_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `move_byproduct_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `move_dest_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `move_finished_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `move_line_raw_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `move_raw_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `never_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `orderpoint_id` | link to one record | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `product_variant_attributes` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `production_group_id` | link to one record | Production Group | `mrp.production.group` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `production_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `scrap_ids` | list of records | Scrap | `stock.scrap` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `subcontractor_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `unbuild_ids` | list of records | Unbuild Order | `mrp.unbuild` | `1 : 0..n` | `mirror of the target column` | derived |
| Manufacturing Order | `mrp.production` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `wip_move_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Manufacturing Order | `mrp.production` | `workcenter_id` | link to one record | Work Center | `mrp.workcenter` | `n : 0..1` | `not declared` | derived |
| Manufacturing Order | `mrp.production` | `workorder_ids` | list of records | Work Order | `mrp.workorder` | `1 : 0..n` | `mirror of the target column` | derived |
| Production Group | `mrp.production.group` | `child_ids` | list on both sides | Production Group | `mrp.production.group` | `0..n : 0..n` | `not declared` | stored |
| Production Group | `mrp.production.group` | `parent_ids` | list on both sides | Production Group | `mrp.production.group` | `0..n : 0..n` | `not declared` | stored |
| Production Group | `mrp.production.group` | `production_ids` | list of records | Manufacturing Order | `mrp.production` | `1 : 0..n` | `mirror of the target column` | derived |
| Split Production Detail | `mrp.production.split.line` | `mrp_production_split_id` | link to one record | Wizard to Split a Production | `mrp.production.split` | `n : 1` | `cascade` | stored |
| Split Production Detail | `mrp.production.split.line` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `consume_line_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Unbuild Order | `mrp.unbuild` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `lot_producing_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | derived |
| Unbuild Order | `mrp.unbuild` | `mo_bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | derived |
| Unbuild Order | `mrp.unbuild` | `mo_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `produce_line_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Unbuild Order | `mrp.unbuild` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Unbuild Order | `mrp.unbuild` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Warn Insufficient Unbuild Quantity | `stock.warn.insufficient.qty.unbuild` | `unbuild_id` | link to one record | Unbuild Order | `mrp.unbuild` | `n : 0..1` | `not declared` | stored |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `mrp_consumption_warning_line_ids` | list of records | Line of issue consumption | `mrp.consumption.warning.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard in case of consumption in warning/strict and more component has been used for a manufacturing order (related to the bill of materials) | `mrp.consumption.warning` | `mrp_production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Wizard to Split Multiple Productions | `mrp.production.split.multi` | `production_ids` | list of records | Wizard to Split a Production | `mrp.production.split` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard to Split a Production | `mrp.production.split` | `production_detailed_vals_ids` | list of records | Split Production Detail | `mrp.production.split.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard to Split a Production | `mrp.production.split` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | stored |
| Wizard to Split a Production | `mrp.production.split` | `production_split_multi_id` | link to one record | Wizard to Split Multiple Productions | `mrp.production.split.multi` | `n : 0..1` | `not declared` | stored |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `mrp_production_backorder_line_ids` | list of records | Backorder Confirmation Line | `mrp.production.backorder.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard to mark as done or create back order | `mrp.production.backorder` | `mrp_production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `journal_id` | link to one record | Journal | `account.journal` | `n : 1` | `not declared` | stored |
| Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `line_ids` | list of records | Account move line to be created when posting work in progress account move | `mrp.account.wip.accounting.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard to post Manufacturing work in progress account move | `mrp.account.wip.accounting` | `mo_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Work Center | `mrp.workcenter` | `alternative_workcenter_ids` | list on both sides | Work Center | `mrp.workcenter` | `0..n : 0..n` | `not declared` | stored |
| Work Center | `mrp.workcenter` | `capacity_ids` | list of records | Work Center Capacity | `mrp.workcenter.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Center | `mrp.workcenter` | `costs_hour_account_ids` | list on both sides | Analytic Account | `account.analytic.account` | `0..n : 0..n` | `not declared` | stored |
| Work Center | `mrp.workcenter` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | derived |
| Work Center | `mrp.workcenter` | `expense_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Work Center | `mrp.workcenter` | `order_ids` | list of records | Work Order | `mrp.workorder` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Center | `mrp.workcenter` | `routing_line_ids` | list of records | Work Center Usage | `mrp.routing.workcenter` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Center | `mrp.workcenter` | `tag_ids` | list on both sides | Add tag for the workcenter | `mrp.workcenter.tag` | `0..n : 0..n` | `not declared` | stored |
| Work Center | `mrp.workcenter` | `time_ids` | list of records | Workcenter Productivity Log | `mrp.workcenter.productivity` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Center Capacity | `mrp.workcenter.capacity` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Work Center Capacity | `mrp.workcenter.capacity` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Work Center Capacity | `mrp.workcenter.capacity` | `workcenter_id` | link to one record | Work Center | `mrp.workcenter` | `n : 1` | `not declared` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `blocked_by_operation_ids` | list on both sides | Work Center Usage | `mrp.routing.workcenter` | `0..n : 0..n` | `not declared` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 1` | `cascade` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `bom_product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Work Center Usage | `mrp.routing.workcenter` | `needed_by_operation_ids` | list on both sides | Work Center Usage | `mrp.routing.workcenter` | `0..n : 0..n` | `not declared` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `workcenter_id` | link to one record | Work Center | `mrp.workcenter` | `n : 1` | `not declared` | stored |
| Work Center Usage | `mrp.routing.workcenter` | `workorder_ids` | list of records | Work Order | `mrp.workorder` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `blocked_by_workorder_ids` | list on both sides | Work Order | `mrp.workorder` | `0..n : 0..n` | `not declared` | stored |
| Work Order | `mrp.workorder` | `finished_lot_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | derived |
| Work Order | `mrp.workorder` | `last_working_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Work Order | `mrp.workorder` | `leave_id` | link to one record | Resource Time Off Detail | `resource.calendar.leaves` | `n : 0..1` | `not declared` | stored |
| Work Order | `mrp.workorder` | `mo_analytic_account_line_ids` | list on both sides | Analytic Line | `account.analytic.line` | `0..n : 0..n` | `not declared` | stored |
| Work Order | `mrp.workorder` | `move_finished_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `move_line_ids` | list of records | Product Moves (Stock Move Line) | `stock.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `move_raw_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `needed_by_workorder_ids` | list on both sides | Work Order | `mrp.workorder` | `0..n : 0..n` | `not declared` | stored |
| Work Order | `mrp.workorder` | `operation_id` | link to one record | Work Center Usage | `mrp.routing.workcenter` | `n : 0..1` | `not declared` | stored |
| Work Order | `mrp.workorder` | `product_variant_attributes` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | derived |
| Work Order | `mrp.workorder` | `production_bom_id` | link to one record | Bill of Material | `mrp.bom` | `n : 0..1` | `not declared` | derived |
| Work Order | `mrp.workorder` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 1` | `not declared` | stored |
| Work Order | `mrp.workorder` | `scrap_ids` | list of records | Scrap | `stock.scrap` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `time_ids` | list of records | Workcenter Productivity Log | `mrp.workcenter.productivity` | `1 : 0..n` | `mirror of the target column` | derived |
| Work Order | `mrp.workorder` | `wc_analytic_account_line_ids` | list on both sides | Analytic Line | `account.analytic.line` | `0..n : 0..n` | `not declared` | stored |
| Work Order | `mrp.workorder` | `workcenter_id` | link to one record | Work Center | `mrp.workcenter` | `n : 1` | `not declared` | stored |
| Work Order | `mrp.workorder` | `working_user_ids` | list of records | User | `res.users` | `1 : 0..n` | `mirror of the target column` | derived |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `account_move_line_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | stored |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `loss_id` | link to one record | Workcenter Productivity Losses | `mrp.workcenter.productivity.loss` | `n : 1` | `restrict` | stored |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `production_id` | link to one record | Manufacturing Order | `mrp.production` | `n : 0..1` | `not declared` | derived |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `workcenter_id` | link to one record | Work Center | `mrp.workcenter` | `n : 1` | `not declared` | stored |
| Workcenter Productivity Log | `mrp.workcenter.productivity` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Workcenter Productivity Losses | `mrp.workcenter.productivity.loss` | `loss_id` | link to one record | manufacturing Workorder productivity losses | `mrp.workcenter.productivity.loss.type` | `n : 0..1` | `not declared` | stored |

### 3.22 Products and Catalog

Product templates and variants, attributes and values, categories, tags, documents, barcodes and nomenclatures, expiry and combos.

Specified in [`../domains/products-and-catalog/`](../domains/products-and-catalog/).

#### Persistent entities (19)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Attribute Value | `product.attribute.value` | `product_attribute_value` | Persistent record with 8 stored columns; belongs to Product Attribute; referenced by 3 relation fields. | Products & Pricelists |
| Barcode Nomenclature | `barcode.nomenclature` | `barcode_nomenclature` | Persistent record with 4 stored columns; owns Barcode Rule; referenced by 4 relation fields. | Barcode |
| Barcode Rule | `barcode.rule` | `barcode_rule` | Persistent record with 10 stored columns. | Barcode |
| Link between products and their Units of Measure | `product.uom` | `product_uom` | Persistent record with 4 stored columns; belongs to Product Unit of Measure, Product Variant; company scoped. | Products & Pricelists |
| Pricelist | `product.pricelist` | `product_pricelist` | Persistent record with 8 stored columns; belongs to Currency; owns Pricelist Rule; company scoped; referenced by 20 relation fields. | Products & Pricelists |
| Pricelist Rule | `product.pricelist.item` | `product_pricelist_item` | Persistent record with 22 stored columns; company scoped; referenced by 1 relation field. | Products & Pricelists |
| Product | `product.template` | `product_template` | Persistent record with 100 stored columns; belongs to Product Unit of Measure; owns Bill of Material, Bill of Material Line, Preferred myDATA classification combinations for a particular product and 6 further collections; company scoped; referenced by 32 relation fields. | Products & Pricelists |
| Product Attribute | `product.attribute` | `product_attribute` | Persistent record with 9 stored columns; owns Attribute Value, Product Template Attribute Line, Product Template Attribute Value; referenced by 2 relation fields. | Products & Pricelists |
| Product Attribute Custom Value | `product.attribute.custom.value` | `product_attribute_custom_value` | Persistent record with 4 stored columns; belongs to Product Template Attribute Value. | Products & Pricelists |
| Product Category | `product.category` | `product_category` | Persistent record with 15 stored columns; owns Product Category, Putaway Rule; referenced by 17 relation fields. | Products & Pricelists |
| Product Combo | `product.combo` | `product_combo` | Persistent record with 5 stored columns; owns Product Combo Item; company scoped; referenced by 3 relation fields. | Products & Pricelists |
| Product Combo Item | `product.combo.item` | `product_combo_item` | Persistent record with 4 stored columns; belongs to Product Combo, Product Variant; referenced by 2 relation fields. | Products & Pricelists |
| Product Document | `product.document` | `product_document` | Persistent record with 7 stored columns; belongs to Attachment; referenced by 3 relation fields. | Products & Pricelists |
| Product Tag | `product.tag` | `product_tag` | Persistent record with 6 stored columns; referenced by 8 relation fields. | Products & Pricelists |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_exclusion` | Persistent record with 2 stored columns; belongs to Product. | Products & Pricelists |
| Product Template Attribute Line | `product.template.attribute.line` | `product_template_attribute_line` | Persistent record with 5 stored columns; belongs to Product, Product Attribute; owns Product Template Attribute Value; referenced by 3 relation fields. | Products & Pricelists |
| Product Template Attribute Value | `product.template.attribute.value` | `product_template_attribute_value` | Persistent record with 7 stored columns; belongs to Attribute Value, Product Template Attribute Line; owns Product Template Attribute Exclusion; referenced by 16 relation fields. | Products & Pricelists |
| Product Variant | `product.product` | `product_product` | Persistent record with 18 stored columns; belongs to Product; owns Bill of Material, Bill of Material Line, Course and 11 further collections; states of `invoice_state`: Paid, Open and Paid, Draft, Open and Paid; referenced by 85 relation fields. | Products & Pricelists |
| Supplier Pricelist | `product.supplierinfo` | `product_supplierinfo` | Persistent record with 16 stored columns; belongs to Contact, Currency, Product and 1 further required links; company scoped; referenced by 4 relation fields. | Products & Pricelists |

#### Interactive assistant entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Choose the sheet layout to print the labels | `product.label.layout` | `product_label_layout` | Interactive assistant with 6 stored columns. | Products & Pricelists |
| Confirm Expiry | `expiry.picking.confirmation` | `expiry_picking_confirmation` | Interactive assistant with 1 stored column. | Products Expiration Date |
| Update product attribute value | `update.product.attribute.value` | `update_product_attribute_value` | Interactive assistant with 2 stored columns; belongs to Attribute Value. | Products & Pricelists |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Barcode Event Mixin | `barcodes.barcode_events_mixin` | `none` | Shared behaviour definition reused through composition. | Barcode |
| Product Catalog Mixin | `product.catalog.mixin` | `none` | Shared behaviour merged into 6 entities. | Products & Pricelists |

#### Relationships (156)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Attribute Value | `product.attribute.value` | `attribute_id` | link to one record | Product Attribute | `product.attribute` | `n : 1` | `cascade` | stored |
| Attribute Value | `product.attribute.value` | `pav_attribute_line_ids` | list on both sides | Product Template Attribute Line | `product.template.attribute.line` | `0..n : 0..n` | `not declared` | stored |
| Barcode Nomenclature | `barcode.nomenclature` | `rule_ids` | list of records | Barcode Rule | `barcode.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Barcode Rule | `barcode.rule` | `associated_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Barcode Rule | `barcode.rule` | `barcode_nomenclature_id` | link to one record | Barcode Nomenclature | `barcode.nomenclature` | `n : 0..1` | `not declared` | stored |
| Choose the sheet layout to print the labels | `product.label.layout` | `move_ids` | list on both sides | Stock Move | `stock.move` | `0..n : 0..n` | `not declared` | stored |
| Choose the sheet layout to print the labels | `product.label.layout` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Choose the sheet layout to print the labels | `product.label.layout` | `product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Choose the sheet layout to print the labels | `product.label.layout` | `product_tmpl_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Confirm Expiry | `expiry.picking.confirmation` | `lot_ids` | list on both sides | Lot/Serial | `stock.lot` | `0..n : 0..n` | `not declared` | stored |
| Confirm Expiry | `expiry.picking.confirmation` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Confirm Expiry | `expiry.picking.confirmation` | `production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | stored |
| Confirm Expiry | `expiry.picking.confirmation` | `workorder_id` | link to one record | Work Order | `mrp.workorder` | `n : 0..1` | `not declared` | stored |
| Link between products and their Units of Measure | `product.uom` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Link between products and their Units of Measure | `product.uom` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `cascade` | stored |
| Link between products and their Units of Measure | `product.uom` | `uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `cascade` | stored |
| Pricelist | `product.pricelist` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Pricelist | `product.pricelist` | `country_group_ids` | list on both sides | Country Group | `res.country.group` | `0..n : 0..n` | `not declared` | stored |
| Pricelist | `product.pricelist` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Pricelist | `product.pricelist` | `item_ids` | list of records | Pricelist Rule | `product.pricelist.item` | `1 : 0..n` | `mirror of the target column` | derived |
| Pricelist | `product.pricelist` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `restrict` | stored |
| Pricelist Rule | `product.pricelist.item` | `base_pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Pricelist Rule | `product.pricelist.item` | `categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `cascade` | stored |
| Pricelist Rule | `product.pricelist.item` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Pricelist Rule | `product.pricelist.item` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Pricelist Rule | `product.pricelist.item` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `cascade` | stored |
| Pricelist Rule | `product.pricelist.item` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Pricelist Rule | `product.pricelist.item` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `cascade` | stored |
| Product | `product.template` | `accessory_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `account_tag_ids` | list on both sides | Account Tag | `account.account.tag` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `alternative_product_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `attribute_line_ids` | list of records | Product Template Attribute Line | `product.template.attribute.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `base_unit_id` | link to one record | Unit of Measure for price per unit on Electronic Commerce products. | `website.base.unit` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `bom_ids` | list of records | Bill of Material | `mrp.bom` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `bom_line_ids` | list of records | Bill of Material Line | `mrp.bom.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `combo_ids` | list on both sides | Product Combo | `product.combo` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `cost_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product | `product.template` | `country_of_origin` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `cpv_code_id` | link to one record | CPV Code | `l10n_ro.cpv.code` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product | `product.template` | `email_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `gelato_image_ids` | list of records | Product Document | `product.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `grade_id` | link to one record | Partner Grade | `res.partner.grade` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `l10n_gr_edi_preferred_classification_ids` | list of records | Preferred myDATA classification combinations for a particular product | `l10n_gr_edi.preferred_classification` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `l10n_hr_kpd_category_id` | link to one record | Croatian KPD Category | `l10n_hr.kpd.category` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `l10n_id_product_code` | link to one record | Product categorization according to E-Faktur | `l10n_id_efaktur_coretax.product.code` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `l10n_tr_default_sales_return_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | derived |
| Product | `product.template` | `lot_sequence_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `optional_product_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `pos_categ_ids` | list on both sides | Point of Sale Category | `pos.category` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `pos_optional_product_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `pricelist_rule_ids` | list of records | Pricelist Rule | `product.pricelist.item` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `product_document_ids` | list of records | Product Document | `product.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `product_tag_ids` | list on both sides | Product Tag | `product.tag` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `product_template_image_ids` | list of records | Product Image | `product.image` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `product_variant_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Product | `product.template` | `product_variant_ids` | list of records | Product Variant | `product.product` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `project_template_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `property_account_expense_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product | `product.template` | `property_account_income_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product | `product.template` | `property_price_difference_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product | `product.template` | `property_stock_inventory` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `property_stock_production` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `public_categ_ids` | list on both sides | Website Product Category | `product.public.category` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `responsible_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `seller_ids` | list of records | Supplier Pricelist | `product.supplierinfo` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `supplier_taxes_id` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `task_template_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Product | `product.template` | `taxes_id` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Product | `product.template` | `uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | stored |
| Product | `product.template` | `valid_product_template_attribute_line_ids` | list on both sides | Product Template Attribute Line | `product.template.attribute.line` | `0..n : 0..n` | `not declared` | derived |
| Product | `product.template` | `variant_seller_ids` | list of records | Supplier Pricelist | `product.supplierinfo` | `1 : 0..n` | `mirror of the target column` | derived |
| Product | `product.template` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Product | `product.template` | `website_ribbon_id` | link to one record | Product ribbon | `product.ribbon` | `n : 0..1` | `not declared` | stored |
| Product Attribute | `product.attribute` | `attribute_line_ids` | list of records | Product Template Attribute Line | `product.template.attribute.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Attribute | `product.attribute` | `category_id` | link to one record | Product Attribute Category | `product.attribute.category` | `n : 0..1` | `not declared` | stored |
| Product Attribute | `product.attribute` | `product_tmpl_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Product Attribute | `product.attribute` | `template_value_ids` | list of records | Product Template Attribute Value | `product.template.attribute.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Attribute | `product.attribute` | `value_ids` | list of records | Attribute Value | `product.attribute.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Attribute Custom Value | `product.attribute.custom.value` | `custom_product_template_attribute_value_id` | link to one record | Product Template Attribute Value | `product.template.attribute.value` | `n : 1` | `restrict` | stored |
| Product Attribute Custom Value | `product.attribute.custom.value` | `pos_order_line_id` | link to one record | Point of Sale Order Lines | `pos.order.line` | `n : 0..1` | `cascade` | stored |
| Product Attribute Custom Value | `product.attribute.custom.value` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `cascade` | stored |
| Product Category | `product.category` | `account_stock_variation_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | derived |
| Product Category | `product.category` | `child_id` | list of records | Product Category | `product.category` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Category | `product.category` | `parent_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `cascade` | stored |
| Product Category | `product.category` | `parent_route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | derived |
| Product Category | `product.category` | `property_account_expense_categ_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product Category | `product.category` | `property_account_income_categ_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product Category | `product.category` | `property_price_difference_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product Category | `product.category` | `property_stock_account_production_cost_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product Category | `product.category` | `property_stock_journal` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Product Category | `product.category` | `property_stock_valuation_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Product Category | `product.category` | `putaway_rule_ids` | list of records | Putaway Rule | `stock.putaway.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Category | `product.category` | `removal_strategy_id` | link to one record | Removal Strategy | `product.removal` | `n : 0..1` | `not declared` | stored |
| Product Category | `product.category` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | stored |
| Product Category | `product.category` | `total_route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `not declared` | derived |
| Product Combo | `product.combo` | `combo_item_ids` | list of records | Product Combo Item | `product.combo.item` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Combo | `product.combo` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Product Combo | `product.combo` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product Combo Item | `product.combo.item` | `combo_id` | link to one record | Product Combo | `product.combo` | `n : 1` | `cascade` | stored |
| Product Combo Item | `product.combo.item` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product Combo Item | `product.combo.item` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `restrict` | stored |
| Product Document | `product.document` | `form_field_ids` | list on both sides | Form fields of inside quotation documents. | `sale.pdf.form.field` | `0..n : 0..n` | `not declared` | stored |
| Product Document | `product.document` | `ir_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 1` | `cascade` | stored |
| Product Tag | `product.tag` | `product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | derived |
| Product Tag | `product.tag` | `product_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Product Tag | `product.tag` | `product_template_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_template_attribute_value_id` | link to one record | Product Template Attribute Value | `product.template.attribute.value` | `n : 0..1` | `cascade` | stored |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `cascade` | stored |
| Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | stored |
| Product Template Attribute Line | `product.template.attribute.line` | `attribute_id` | link to one record | Product Attribute | `product.attribute` | `n : 1` | `restrict` | stored |
| Product Template Attribute Line | `product.template.attribute.line` | `product_template_value_ids` | list of records | Product Template Attribute Value | `product.template.attribute.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Template Attribute Line | `product.template.attribute.line` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `cascade` | stored |
| Product Template Attribute Line | `product.template.attribute.line` | `value_ids` | list on both sides | Attribute Value | `product.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Product Template Attribute Value | `product.template.attribute.value` | `attribute_line_id` | link to one record | Product Template Attribute Line | `product.template.attribute.line` | `n : 1` | `cascade` | stored |
| Product Template Attribute Value | `product.template.attribute.value` | `exclude_for` | list of records | Product Template Attribute Exclusion | `product.template.attribute.exclusion` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Template Attribute Value | `product.template.attribute.value` | `product_attribute_value_id` | link to one record | Attribute Value | `product.attribute.value` | `n : 1` | `cascade` | stored |
| Product Template Attribute Value | `product.template.attribute.value` | `ptav_product_variant_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Product Variant | `product.product` | `additional_product_tag_ids` | list on both sides | Product Tag | `product.tag` | `0..n : 0..n` | `not declared` | stored |
| Product Variant | `product.product` | `all_product_tag_ids` | list on both sides | Product Tag | `product.tag` | `0..n : 0..n` | `not declared` | derived |
| Product Variant | `product.product` | `base_unit_id` | link to one record | Unit of Measure for price per unit on Electronic Commerce products. | `website.base.unit` | `n : 0..1` | `not declared` | stored |
| Product Variant | `product.product` | `bom_line_ids` | list of records | Bill of Material Line | `mrp.bom.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `channel_ids` | list of records | Course | `slide.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product Variant | `product.product` | `event_ticket_ids` | list of records | Event Ticket | `event.event.ticket` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `orderpoint_ids` | list of records | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `pricelist_rule_ids` | list of records | Pricelist Rule | `product.pricelist.item` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `product_document_ids` | list of records | Product Document | `product.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `product_template_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Product Variant | `product.product` | `product_template_variant_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Product Variant | `product.product` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `cascade` | stored |
| Product Variant | `product.product` | `product_uom_ids` | list of records | Link between products and their Units of Measure | `product.uom` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `product_variant_image_ids` | list of records | Product Image | `product.image` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `purchase_order_line_ids` | list of records | Purchase Order Line | `purchase.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `putaway_rule_ids` | list of records | Putaway Rule | `stock.putaway.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `stock_move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `stock_notification_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Product Variant | `product.product` | `stock_quant_ids` | list of records | Quants | `stock.quant` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `storage_category_capacity_ids` | list of records | Storage Category Capacity | `stock.storage.category.capacity` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `variant_bom_ids` | list of records | Bill of Material | `mrp.bom` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Variant | `product.product` | `variant_ribbon_id` | link to one record | Product ribbon | `product.ribbon` | `n : 0..1` | `not declared` | stored |
| Supplier Pricelist | `product.supplierinfo` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Supplier Pricelist | `product.supplierinfo` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Supplier Pricelist | `product.supplierinfo` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Supplier Pricelist | `product.supplierinfo` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Supplier Pricelist | `product.supplierinfo` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 1` | `cascade` | stored |
| Supplier Pricelist | `product.supplierinfo` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 1` | `not declared` | stored |
| Supplier Pricelist | `product.supplierinfo` | `purchase_requisition_id` | link to one record | Purchase Requisition | `purchase.requisition` | `n : 0..1` | `not declared` | derived |
| Supplier Pricelist | `product.supplierinfo` | `purchase_requisition_line_id` | link to one record | Purchase Requisition Line | `purchase.requisition.line` | `n : 0..1` | `not declared` | stored |
| Update product attribute value | `update.product.attribute.value` | `attribute_value_id` | link to one record | Attribute Value | `product.attribute.value` | `n : 1` | `not declared` | stored |

### 3.23 Purchasing

Requests for quotation, purchase orders and lines, vendor prices, receipts, bill control policies, three-way matching and purchase agreements.

Specified in [`../domains/purchasing/`](../domains/purchasing/).

#### Persistent entities (8)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `none, read from a stored query` | Persistent record with 0 stored columns; company scoped. | Purchase |
| Purchase Order | `purchase.order` | `purchase_order` | Persistent record with 38 stored columns; belongs to Companies, Contact, Currency and 1 further required links; owns Purchase Order, Purchase Order Line; lifecycle states Request for Quotation, Request for Quotation Sent, To Approve, Purchase Order, Cancelled; 3 state fields in all; company scoped; referenced by 14 relation fields. | Purchase |
| Purchase Order Line | `purchase.order.line` | `purchase_order_line` | Persistent record with 30 stored columns; belongs to Purchase Order; owns Journal Item, Stock Move; carries the state field `state`; company scoped; referenced by 6 relation fields. | Purchase |
| Purchase Report | `purchase.report` | `none, read from a stored query` | Persistent record with 0 stored columns; lifecycle states Draft Request for Quotation, Request for Quotation Sent, To Approve, Purchase Order, Cancelled; company scoped. | Purchase |
| Purchase Requisition | `purchase.requisition` | `purchase_requisition` | Persistent record with 14 stored columns; belongs to Companies, Currency, Picking Type; owns Purchase Order, Purchase Requisition Line; lifecycle states Draft, Confirmed, Closed, Cancelled; company scoped; referenced by 3 relation fields. | Purchase Agreements |
| Purchase Requisition Line | `purchase.requisition.line` | `purchase_requisition_line` | Persistent record with 9 stored columns; belongs to Product Variant, Purchase Requisition; owns Supplier Pricelist; company scoped; referenced by 1 relation field. | Purchase Agreements |
| Purchases & Bills Union | `purchase.bill.union` | `purchase_bill_union` | Persistent record with 9 stored columns; company scoped; referenced by 1 relation field. | Purchase |
| Technical model to group purchase order for call to tenders | `purchase.order.group` | `purchase_order_group` | Persistent record with 0 stored columns; owns Purchase Order; referenced by 1 relation field. | Purchase Agreements |

#### Interactive assistant entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Bill to Purchase Order | `bill.to.po.wizard` | `bill_to_po_wizard` | Interactive assistant with 2 stored columns. | Purchase |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `purchase_requisition_alternative_warning` | Interactive assistant with 0 stored columns. | Purchase Agreements |
| Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `purchase_requisition_create_alternative` | Interactive assistant with 2 stored columns. | Purchase Agreements |

#### Relationships (88)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Bill to Purchase Order | `bill.to.po.wizard` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Bill to Purchase Order | `bill.to.po.wizard` | `purchase_order_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | stored |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `aml_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `line_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `pol_id` | link to one record | Purchase Order Line | `purchase.order.line` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Purchase Line and Vendor Bill line matching view | `purchase.bill.line.match` | `purchase_order_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | derived |
| Purchase Order | `purchase.order` | `alternative_po_ids` | list of records | Purchase Order | `purchase.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Order | `purchase.order` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `dest_address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `duplicated_order_ids` | list on both sides | Purchase Order | `purchase.order` | `0..n : 0..n` | `not declared` | derived |
| Purchase Order | `purchase.order` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `grid_product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Purchase Order | `purchase.order` | `incoterm_id` | link to one record | Incoterms | `account.incoterms` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Purchase Order | `purchase.order` | `order_line` | list of records | Purchase Order Line | `purchase.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Order | `purchase.order` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `payment_term_id` | link to one record | Payment Terms | `account.payment.term` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `picking_ids` | list on both sides | Transfer | `stock.picking` | `0..n : 0..n` | `not declared` | stored |
| Purchase Order | `purchase.order` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Purchase Order | `purchase.order` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `purchase_group_id` | link to one record | Technical model to group purchase order for call to tenders | `purchase.order.group` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Purchase Order | `purchase.order` | `requisition_id` | link to one record | Purchase Requisition | `purchase.requisition` | `n : 0..1` | `not declared` | stored |
| Purchase Order | `purchase.order` | `tax_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Purchase Order | `purchase.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Purchase Order Line | `purchase.order.line` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `invoice_lines` | list of records | Journal Item | `account.move.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Order Line | `purchase.order.line` | `location_final_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `move_dest_ids` | list on both sides | Stock Move | `stock.move` | `0..n : 0..n` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Order Line | `purchase.order.line` | `order_id` | link to one record | Purchase Order | `purchase.order` | `n : 1` | `cascade` | stored |
| Purchase Order Line | `purchase.order.line` | `orderpoint_id` | link to one record | Minimum Inventory Rule | `stock.warehouse.orderpoint` | `n : 0..1` | `set null` | stored |
| Purchase Order Line | `purchase.order.line` | `parent_id` | link to one record | Purchase Order Line | `purchase.order.line` | `n : 0..1` | `not declared` | derived |
| Purchase Order Line | `purchase.order.line` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `restrict` | stored |
| Purchase Order Line | `purchase.order.line` | `product_no_variant_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Purchase Order Line | `purchase.order.line` | `product_template_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Purchase Order Line | `purchase.order.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `restrict` | stored |
| Purchase Order Line | `purchase.order.line` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Purchase Order Line | `purchase.order.line` | `selected_seller_id` | link to one record | Supplier Pricelist | `product.supplierinfo` | `n : 0..1` | `not declared` | derived |
| Purchase Order Line | `purchase.order.line` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Purchase Report | `purchase.report` | `category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `order_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `picking_type_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Purchase Report | `purchase.report` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Purchase Requisition | `purchase.requisition` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Purchase Requisition | `purchase.requisition` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Purchase Requisition | `purchase.requisition` | `line_ids` | list of records | Purchase Requisition Line | `purchase.requisition.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Requisition | `purchase.requisition` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Purchase Requisition | `purchase.requisition` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Purchase Requisition | `purchase.requisition` | `purchase_ids` | list of records | Purchase Order | `purchase.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchase Requisition | `purchase.requisition` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition | `purchase.requisition` | `vendor_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition | `purchase.requisition` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `move_dest_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `requisition_id` | link to one record | Purchase Requisition | `purchase.requisition` | `n : 1` | `cascade` | stored |
| Purchase Requisition Line | `purchase.requisition.line` | `supplier_info_ids` | list of records | Supplier Pricelist | `product.supplierinfo` | `1 : 0..n` | `mirror of the target column` | derived |
| Purchases & Bills Union | `purchase.bill.union` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Purchases & Bills Union | `purchase.bill.union` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Purchases & Bills Union | `purchase.bill.union` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Purchases & Bills Union | `purchase.bill.union` | `purchase_order_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | stored |
| Purchases & Bills Union | `purchase.bill.union` | `vendor_bill_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Technical model to group purchase order for call to tenders | `purchase.order.group` | `order_ids` | list of records | Purchase Order | `purchase.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `alternative_po_ids` | list on both sides | Purchase Order | `purchase.order` | `0..n : 0..n` | `not declared` | stored |
| Wizard in case purchase order still has open alternative requests for quotation | `purchase.requisition.alternative.warning` | `po_ids` | list on both sides | Purchase Order | `purchase.order` | `0..n : 0..n` | `not declared` | stored |
| Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `origin_po_id` | link to one record | Purchase Order | `purchase.order` | `n : 0..1` | `not declared` | stored |
| Wizard to preset values for alternative purchase order | `purchase.requisition.create.alternative` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |

### 3.24 Repair and Maintenance

Repair orders and their lines, equipment, maintenance requests, teams and stages, and preventive maintenance scheduling.

Specified in [`../domains/repair-and-maintenance/`](../domains/repair-and-maintenance/).

#### Persistent entities (7)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Maintenance Equipment | `maintenance.equipment` | `maintenance_equipment` | Persistent record with 26 stored columns; owns Maintenance Request; referenced by 1 relation field. | Maintenance |
| Maintenance Equipment Category | `maintenance.equipment.category` | `maintenance_equipment_category` | Persistent record with 7 stored columns; owns Maintenance Equipment, Maintenance Request; company scoped; referenced by 2 relation fields. | Maintenance |
| Maintenance Request | `maintenance.request` | `maintenance_request` | Persistent record with 29 stored columns; belongs to Companies, Maintenance Teams; states of `kanban_state`: In Progress, Blocked, Ready for next stage; company scoped. | Maintenance |
| Maintenance Stage | `maintenance.stage` | `maintenance_stage` | Persistent record with 4 stored columns; referenced by 1 relation field. | Maintenance |
| Maintenance Teams | `maintenance.team` | `maintenance_team` | Persistent record with 5 stored columns; owns Maintenance Equipment, Maintenance Request; company scoped; referenced by 2 relation fields. | Maintenance |
| Repair Order | `repair.order` | `repair_order` | Persistent record with 27 stored columns; belongs to Companies, Inventory Locations, Picking Type; owns Lot/Serial, Product Variant, Stock Move; lifecycle states New, Confirmed, Under Repair, Repaired, Cancelled; 2 state fields in all; company scoped; referenced by 3 relation fields. | Repairs |
| Repair Tags | `repair.tags` | `repair_tags` | Persistent record with 2 stored columns; referenced by 1 relation field. | Repairs |

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Warn Insufficient Repair Quantity | `stock.warn.insufficient.qty.repair` | `stock_warn_insufficient_qty_repair` | Interactive assistant with 5 stored columns. | Repairs |

#### Shared behaviour entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Maintenance Maintained Item | `maintenance.mixin` | `none` | Shared behaviour merged into 1 entity. | Maintenance |

#### Relationships (52)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Maintenance Equipment | `maintenance.equipment` | `category_id` | link to one record | Maintenance Equipment Category | `maintenance.equipment.category` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment | `maintenance.equipment` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment | `maintenance.equipment` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment | `maintenance.equipment` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment | `maintenance.equipment` | `maintenance_ids` | list of records | Maintenance Request | `maintenance.request` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Equipment | `maintenance.equipment` | `owner_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment | `maintenance.equipment` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment Category | `maintenance.equipment.category` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Maintenance Equipment Category | `maintenance.equipment.category` | `equipment_ids` | list of records | Maintenance Equipment | `maintenance.equipment` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Equipment Category | `maintenance.equipment.category` | `maintenance_ids` | list of records | Maintenance Request | `maintenance.request` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Equipment Category | `maintenance.equipment.category` | `technician_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Maintenance Maintained Item | `maintenance.mixin` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Maintenance Maintained Item | `maintenance.mixin` | `maintenance_ids` | list of records | Maintenance Request | `maintenance.request` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Maintained Item | `maintenance.mixin` | `maintenance_team_id` | link to one record | Maintenance Teams | `maintenance.team` | `n : 0..1` | `not declared` | derived |
| Maintenance Maintained Item | `maintenance.mixin` | `technician_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Maintenance Request | `maintenance.request` | `category_id` | link to one record | Maintenance Equipment Category | `maintenance.equipment.category` | `n : 0..1` | `not declared` | stored |
| Maintenance Request | `maintenance.request` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Maintenance Request | `maintenance.request` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Maintenance Request | `maintenance.request` | `equipment_id` | link to one record | Maintenance Equipment | `maintenance.equipment` | `n : 0..1` | `restrict` | stored |
| Maintenance Request | `maintenance.request` | `maintenance_team_id` | link to one record | Maintenance Teams | `maintenance.team` | `n : 1` | `not declared` | stored |
| Maintenance Request | `maintenance.request` | `owner_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Maintenance Request | `maintenance.request` | `stage_id` | link to one record | Maintenance Stage | `maintenance.stage` | `n : 0..1` | `restrict` | stored |
| Maintenance Request | `maintenance.request` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Maintenance Teams | `maintenance.team` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Maintenance Teams | `maintenance.team` | `equipment_ids` | list of records | Maintenance Equipment | `maintenance.equipment` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Teams | `maintenance.team` | `member_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Maintenance Teams | `maintenance.team` | `request_ids` | list of records | Maintenance Request | `maintenance.request` | `1 : 0..n` | `mirror of the target column` | derived |
| Maintenance Teams | `maintenance.team` | `todo_request_ids` | list of records | Maintenance Request | `maintenance.request` | `1 : 0..n` | `mirror of the target column` | derived |
| Repair Order | `repair.order` | `allowed_lot_ids` | list of records | Lot/Serial | `stock.lot` | `1 : 0..n` | `mirror of the target column` | derived |
| Repair Order | `repair.order` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Repair Order | `repair.order` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `lot_id` | link to one record | Lot/Serial | `stock.lot` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `move_id` | link to one record | Stock Move | `stock.move` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Repair Order | `repair.order` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `parts_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `picking_id` | link to one record | Transfer | `stock.picking` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `picking_product_ids` | list of records | Product Variant | `product.product` | `1 : 0..n` | `mirror of the target column` | derived |
| Repair Order | `repair.order` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `product_location_dest_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `product_location_src_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `product_uom` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `recycle_location_id` | link to one record | Inventory Locations | `stock.location` | `n : 1` | `not declared` | stored |
| Repair Order | `repair.order` | `reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Repair Order | `repair.order` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Repair Order | `repair.order` | `tag_ids` | list on both sides | Repair Tags | `repair.tags` | `0..n : 0..n` | `not declared` | stored |
| Repair Order | `repair.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Warn Insufficient Repair Quantity | `stock.warn.insufficient.qty.repair` | `repair_id` | link to one record | Repair Order | `repair.order` | `n : 0..1` | `not declared` | stored |

### 3.25 Replenishment and Procurement

Routes and rules, reordering rules, make-to-order, lead times, the scheduler, forecasted quantities and drop shipping.

Specified in [`../domains/replenishment-and-procurement/`](../domains/replenishment-and-procurement/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Vendor Delay Report | `vendor.delay.report` | `vendor_delay_report` | Persistent record with 7 stored columns. | Purchase Stock |

#### Relationships (3)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Vendor Delay Report | `vendor.delay.report` | `category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Vendor Delay Report | `vendor.delay.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Vendor Delay Report | `vendor.delay.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |

### 3.26 Units of Measure and Packaging

Unit of measure categories, conversion factors and rounding, product packagings and package types with dimensions and weight.

Specified in [`../domains/units-of-measure-and-packaging/`](../domains/units-of-measure-and-packaging/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Product Unit of Measure | `uom.uom` | `uom_uom` | Persistent record with 17 stored columns; owns Link between products and their Units of Measure, Product Unit of Measure; referenced by 57 relation fields. | Units of measure |

#### Relationships (6)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Product Unit of Measure | `uom.uom` | `l10n_eg_unit_code_id` | link to one record | Estimated Time of Arrival code for the unit of measures | `l10n_eg_edi.uom.code` | `n : 0..1` | `not declared` | stored |
| Product Unit of Measure | `uom.uom` | `l10n_id_uom_code` | link to one record | unit of measure categorization according to E-Faktur | `l10n_id_efaktur_coretax.uom.code` | `n : 0..1` | `not declared` | stored |
| Product Unit of Measure | `uom.uom` | `package_type_id` | link to one record | Stock package type | `stock.package.type` | `n : 0..1` | `not declared` | stored |
| Product Unit of Measure | `uom.uom` | `product_uom_ids` | list of records | Link between products and their Units of Measure | `product.uom` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Unit of Measure | `uom.uom` | `related_uom_ids` | list of records | Product Unit of Measure | `uom.uom` | `1 : 0..n` | `mirror of the target column` | derived |
| Product Unit of Measure | `uom.uom` | `relative_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `cascade` | stored |

### 3.27 Customer Relationship Management

Leads and opportunities, pipeline stages, sales teams and members, predictive lead scoring, lead assignment and enrichment, lost reasons, merging and partnerships.

Specified in [`../domains/customer-relationship-management/`](../domains/customer-relationship-management/).

#### Persistent entities (25)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Campaign Stage | `utm.stage` | `utm_stage` | Persistent record with 2 stored columns; referenced by 1 relation field. | campaign tracking parameter Trackers |
| campaign tracking parameter Campaign | `utm.campaign` | `utm_campaign` | Persistent record with 13 stored columns; belongs to Campaign Stage, User; owns Mass Mailing; company scoped; referenced by 8 relation fields. | campaign tracking parameter Trackers |
| campaign tracking parameter Medium | `utm.medium` | `utm_medium` | Persistent record with 2 stored columns; referenced by 5 relation fields. | campaign tracking parameter Trackers |
| campaign tracking parameter Source | `utm.source` | `utm_source` | Persistent record with 1 stored column; referenced by 4 relation fields. | campaign tracking parameter Trackers |
| campaign tracking parameter Tag | `utm.tag` | `utm_tag` | Persistent record with 2 stored columns; referenced by 1 relation field. | campaign tracking parameter Trackers |
| customer relationship management Activity Analysis | `crm.activity.report` | `crm_activity_report` | Persistent record with 19 stored columns; states of `won_status`: Won, Lost, Pending; company scoped. | customer relationship management |
| customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry` | Customer Relationship Management In Application Purchase Lead Industry. | Lead Generation |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `crm_reveal_rule` | Persistent record with 18 stored columns; owns Lead; referenced by 2 relation fields. | Lead Generation From Website Visits |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request` | Persistent record with 15 stored columns; owns Country state, Lead; lifecycle states Draft, Error, Done; referenced by 1 relation field. | Lead Generation |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `none, read from a stored query` | Persistent record with 0 stored columns. | Resellers |
| customer relationship management Recurring revenue plans | `crm.recurring.plan` | `crm_recurring_plan` | Persistent record with 4 stored columns; referenced by 1 relation field. | customer relationship management |
| customer relationship management Reveal View | `crm.reveal.view` | `crm_reveal_view` | Persistent record with 3 stored columns; states of `reveal_state`: To Process, Not Found. | Lead Generation From Website Visits |
| customer relationship management Stages | `crm.stage` | `crm_stage` | Persistent record with 7 stored columns; referenced by 2 relation fields. | customer relationship management |
| Event Lead Request | `event.lead.request` | `event_lead_request` | Persistent record with 2 stored columns; belongs to Event. | Event customer relationship management |
| Event Lead Rules | `event.lead.rule` | `event_lead_rule` | Persistent record with 10 stored columns; owns Lead; company scoped; referenced by 2 relation fields. | Event customer relationship management |
| Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `crm_lead_scoring_frequency_field` | Persistent record with 2 stored columns; belongs to Fields; referenced by 2 relation fields. | customer relationship management |
| Helper methods for crm_iap_mine modules | `crm.iap.lead.helpers` | `crm_iap_lead_helpers` | Persistent record with 0 stored columns. | Lead Generation |
| Lead | `crm.lead` | `crm_lead` | Persistent record with 69 stored columns; owns Calendar Event, Sales Order; states of `phone_state`: Correct, Incorrect; 3 state fields in all; company scoped; referenced by 14 relation fields. | customer relationship management |
| Lead Scoring Frequency | `crm.lead.scoring.frequency` | `crm_lead_scoring_frequency` | Persistent record with 5 stored columns. | customer relationship management |
| Opp. Lost Reason | `crm.lost.reason` | `crm_lost_reason` | Persistent record with 2 stored columns; referenced by 2 relation fields. | customer relationship management |
| Partner Activation | `res.partner.activation` | `res_partner_activation` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Resellers |
| Partner Grade | `res.partner.grade` | `res_partner_grade` | Persistent record with 7 stored columns; company scoped; referenced by 4 relation fields. | Partnership / Membership |
| People Role | `crm.iap.lead.role` | `crm_iap_lead_role` | Persistent record with 3 stored columns; referenced by 4 relation fields. | Lead Generation |
| People Seniority | `crm.iap.lead.seniority` | `crm_iap_lead_seniority` | Persistent record with 2 stored columns; referenced by 2 relation fields. | Lead Generation |
| Phone Blacklist | `phone.blacklist` | `phone_blacklist` | Persistent record with 2 stored columns. | Phone Numbers Validation |

#### Interactive assistant entities (8)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `crm_lead2opportunity_partner_mass` | Interactive assistant with 9 stored columns. | customer relationship management |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `crm_lead2opportunity_partner` | Interactive assistant with 8 stored columns; belongs to Lead. | customer relationship management |
| Get Lost Reason | `crm.lead.lost` | `crm_lead_lost` | Interactive assistant with 2 stored columns. | customer relationship management |
| Lead Assignation | `crm.lead.assignation` | `crm_lead_assignation` | Interactive assistant with 6 stored columns. | Resellers |
| Lead forward to partner | `crm.lead.forward.to.partner` | `crm_lead_forward_to_partner` | Interactive assistant with 3 stored columns; owns Lead Assignation; referenced by 1 relation field. | Resellers |
| Merge Opportunities | `crm.merge.opportunity` | `crm_merge_opportunity` | Interactive assistant with 2 stored columns. | customer relationship management |
| Remove phone from blacklist | `phone.blacklist.remove` | `phone_blacklist_remove` | Interactive assistant with 2 stored columns. | Phone Numbers Validation |
| Update the probabilities | `crm.lead.pls.update` | `crm_lead_pls_update` | Interactive assistant with 1 stored column. | customer relationship management |

#### Shared behaviour entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| campaign tracking parameter Mixin | `utm.mixin` | `none` | Shared behaviour merged into 5 entities. | campaign tracking parameter Trackers |
| campaign tracking parameter Source Mixin | `utm.source.mixin` | `none` | Shared behaviour merged into 3 entities. | campaign tracking parameter Trackers |
| Phone Blacklist Mixin | `mail.thread.phone` | `none` | Shared behaviour merged into 4 entities. | Phone Numbers Validation |

#### Relationships (110)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `lead_tomerge_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Convert Lead to Opportunity (in mass) | `crm.lead2opportunity.partner.mass` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `duplicated_lead_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `lead_id` | link to one record | Lead | `crm.lead` | `n : 1` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | stored |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Event Lead Request | `event.lead.request` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `cascade` | stored |
| Event Lead Request | `event.lead.request` | `event_lead_rule_ids` | list on both sides | Event Lead Rules | `event.lead.rule` | `0..n : 0..n` | `not declared` | stored |
| Event Lead Rules | `event.lead.rule` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Event Lead Rules | `event.lead.rule` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Event Lead Rules | `event.lead.rule` | `event_type_ids` | list on both sides | Event Template | `event.type` | `0..n : 0..n` | `not declared` | stored |
| Event Lead Rules | `event.lead.rule` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Lead Rules | `event.lead.rule` | `lead_sales_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Event Lead Rules | `event.lead.rule` | `lead_tag_ids` | list on both sides | customer relationship management Tag | `crm.tag` | `0..n : 0..n` | `not declared` | stored |
| Event Lead Rules | `event.lead.rule` | `lead_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 1` | `cascade` | stored |
| Get Lost Reason | `crm.lead.lost` | `lead_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Get Lost Reason | `crm.lead.lost` | `lost_reason_id` | link to one record | Opp. Lost Reason | `crm.lost.reason` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `calendar_event_ids` | list of records | Calendar Event | `calendar.event` | `1 : 0..n` | `mirror of the target column` | derived |
| Lead | `crm.lead` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Lead | `crm.lead` | `company_currency` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Lead | `crm.lead` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `duplicate_lead_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | derived |
| Lead | `crm.lead` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `event_lead_rule_id` | link to one record | Event Lead Rules | `event.lead.rule` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `lead_mining_request_id` | link to one record | customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `lost_reason_id` | link to one record | Opp. Lost Reason | `crm.lost.reason` | `n : 0..1` | `restrict` | stored |
| Lead | `crm.lead` | `order_ids` | list of records | Sales Order | `sale.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Lead | `crm.lead` | `origin_channel_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `origin_survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `set null` | stored |
| Lead | `crm.lead` | `partner_assigned_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `partner_declined_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Lead | `crm.lead` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `recurring_plan` | link to one record | customer relationship management Recurring revenue plans | `crm.recurring.plan` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `registration_ids` | list on both sides | Event Registration | `event.registration` | `0..n : 0..n` | `not declared` | stored |
| Lead | `crm.lead` | `reveal_rule_id` | link to one record | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `stage_id` | link to one record | customer relationship management Stages | `crm.stage` | `n : 0..1` | `restrict` | stored |
| Lead | `crm.lead` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `tag_ids` | list on both sides | customer relationship management Tag | `crm.tag` | `0..n : 0..n` | `not declared` | stored |
| Lead | `crm.lead` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Lead | `crm.lead` | `user_company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | derived |
| Lead | `crm.lead` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Lead | `crm.lead` | `visitor_ids` | list on both sides | Website Visitor | `website.visitor` | `0..n : 0..n` | `not declared` | stored |
| Lead Assignation | `crm.lead.assignation` | `forward_id` | link to one record | Lead forward to partner | `crm.lead.forward.to.partner` | `n : 0..1` | `not declared` | stored |
| Lead Assignation | `crm.lead.assignation` | `lead_id` | link to one record | Lead | `crm.lead` | `n : 0..1` | `not declared` | stored |
| Lead Assignation | `crm.lead.assignation` | `partner_assigned_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Lead Scoring Frequency | `crm.lead.scoring.frequency` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `cascade` | stored |
| Lead forward to partner | `crm.lead.forward.to.partner` | `assignation_lines` | list of records | Lead Assignation | `crm.lead.assignation` | `1 : 0..n` | `mirror of the target column` | derived |
| Lead forward to partner | `crm.lead.forward.to.partner` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Merge Opportunities | `crm.merge.opportunity` | `opportunity_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Merge Opportunities | `crm.merge.opportunity` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | stored |
| Merge Opportunities | `crm.merge.opportunity` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Partner Grade | `res.partner.grade` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Partner Grade | `res.partner.grade` | `default_pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Update the probabilities | `crm.lead.pls.update` | `pls_fields` | list on both sides | Fields that can be used for predictive lead scoring computation | `crm.lead.scoring.frequency.field` | `0..n : 0..n` | `not declared` | stored |
| campaign tracking parameter Campaign | `utm.campaign` | `ab_testing_winner_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| campaign tracking parameter Campaign | `utm.campaign` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| campaign tracking parameter Campaign | `utm.campaign` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| campaign tracking parameter Campaign | `utm.campaign` | `mailing_mail_ids` | list of records | Mass Mailing | `mailing.mailing` | `1 : 0..n` | `mirror of the target column` | derived |
| campaign tracking parameter Campaign | `utm.campaign` | `mailing_sms_ids` | list of records | Mass Mailing | `mailing.mailing` | `1 : 0..n` | `mirror of the target column` | derived |
| campaign tracking parameter Campaign | `utm.campaign` | `stage_id` | link to one record | Campaign Stage | `utm.stage` | `n : 1` | `restrict` | stored |
| campaign tracking parameter Campaign | `utm.campaign` | `tag_ids` | list on both sides | campaign tracking parameter Tag | `utm.tag` | `0..n : 0..n` | `not declared` | stored |
| campaign tracking parameter Campaign | `utm.campaign` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| campaign tracking parameter Mixin | `utm.mixin` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `not declared` | derived |
| campaign tracking parameter Mixin | `utm.mixin` | `medium_id` | link to one record | campaign tracking parameter Medium | `utm.medium` | `n : 0..1` | `not declared` | derived |
| campaign tracking parameter Mixin | `utm.mixin` | `source_id` | link to one record | campaign tracking parameter Source | `utm.source` | `n : 0..1` | `not declared` | derived |
| campaign tracking parameter Source Mixin | `utm.source.mixin` | `source_id` | link to one record | campaign tracking parameter Source | `utm.source` | `n : 1` | `restrict` | derived |
| customer relationship management Activity Analysis | `crm.activity.report` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `lead_id` | link to one record | Lead | `crm.lead` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `mail_activity_type_id` | link to one record | Activity Type | `mail.activity.type` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `stage_id` | link to one record | customer relationship management Stages | `crm.stage` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `subtype_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | stored |
| customer relationship management Activity Analysis | `crm.activity.report` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `industry_tag_ids` | list on both sides | customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `other_role_ids` | list on both sides | People Role | `crm.iap.lead.role` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `preferred_role_id` | link to one record | People Role | `crm.iap.lead.role` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `seniority_id` | link to one record | People Seniority | `crm.iap.lead.seniority` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `tag_ids` | list on both sides | customer relationship management Tag | `crm.tag` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Generation Rules | `crm.reveal.rule` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `available_state_ids` | list of records | Country state | `res.country.state` | `1 : 0..n` | `mirror of the target column` | derived |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `industry_ids` | list on both sides | customer relationship management in-app purchase Lead Industry | `crm.iap.lead.industry` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `preferred_role_id` | link to one record | People Role | `crm.iap.lead.role` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `role_ids` | list on both sides | People Role | `crm.iap.lead.role` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `seniority_id` | link to one record | People Seniority | `crm.iap.lead.seniority` | `n : 0..1` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `tag_ids` | list on both sides | customer relationship management Tag | `crm.tag` | `0..n : 0..n` | `not declared` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| customer relationship management Lead Mining Request | `crm.iap.lead.mining.request` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `activation` | link to one record | Partner Activation | `res.partner.activation` | `n : 0..1` | `not declared` | derived |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `grade_id` | link to one record | Partner Grade | `res.partner.grade` | `n : 0..1` | `not declared` | derived |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| customer relationship management Partnership Analysis | `crm.partner.report.assign` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| customer relationship management Reveal View | `crm.reveal.view` | `reveal_rule_id` | link to one record | customer relationship management Lead Generation Rules | `crm.reveal.rule` | `n : 0..1` | `not declared` | stored |
| customer relationship management Stages | `crm.stage` | `team_ids` | list on both sides | Sales Team | `crm.team` | `0..n : 0..n` | `restrict` | stored |

### 3.28 Loyalty and Promotions

Loyalty programs, rules, rewards, cards, coupons, gift cards and electronic wallets.

Specified in [`../domains/loyalty-and-promotions/`](../domains/loyalty-and-promotions/).

#### Persistent entities (7)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| History for Loyalty cards and Electronic Wallets | `loyalty.history` | `loyalty_history` | Persistent record with 6 stored columns; belongs to Loyalty Coupon. | Coupons & Loyalty |
| Loyalty Communication | `loyalty.mail` | `loyalty_mail` | Persistent record with 6 stored columns; belongs to Email Templates, Loyalty Program. | Coupons & Loyalty |
| Loyalty Coupon | `loyalty.card` | `loyalty_card` | Persistent record with 9 stored columns; owns History for Loyalty cards and Electronic Wallets; referenced by 7 relation fields. | Coupons & Loyalty |
| Loyalty Program | `loyalty.program` | `loyalty_program` | Persistent record with 18 stored columns; belongs to Currency; owns Loyalty Communication, Loyalty Coupon, Loyalty Reward and 1 further collections; company scoped; referenced by 6 relation fields. | Coupons & Loyalty |
| Loyalty Reward | `loyalty.reward` | `loyalty_reward` | Persistent record with 18 stored columns; belongs to Loyalty Program; referenced by 5 relation fields. | Coupons & Loyalty |
| Loyalty Rule | `loyalty.rule` | `loyalty_rule` | Persistent record with 16 stored columns; belongs to Loyalty Program; referenced by 1 relation field. | Coupons & Loyalty |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `sale_order_coupon_points` | Persistent record with 3 stored columns; belongs to Loyalty Coupon, Sales Order. | Sale Loyalty |

#### Interactive assistant entities (5)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `coupon_share` | Interactive assistant with 4 stored columns; belongs to Loyalty Program, Website. | Coupons, Promotions, Gift Card and Loyalty for Electronic Commerce |
| Generate Coupons | `loyalty.generate.wizard` | `loyalty_generate_wizard` | Interactive assistant with 6 stored columns; belongs to Loyalty Program. | Coupons & Loyalty |
| Sale Loyalty - Apply Coupon Wizard | `sale.loyalty.coupon.wizard` | `sale_loyalty_coupon_wizard` | Interactive assistant with 2 stored columns; belongs to Sales Order. | Sale Loyalty |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `sale_loyalty_reward_wizard` | Interactive assistant with 3 stored columns; belongs to Sales Order. | Sale Loyalty |
| Update Loyalty Card Points | `loyalty.card.update.balance` | `loyalty_card_update_balance` | Interactive assistant with 3 stored columns; belongs to Loyalty Coupon. | Coupons & Loyalty |

#### Relationships (52)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `coupon_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 0..1` | `not declared` | stored |
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 1` | `not declared` | stored |
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `program_website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | derived |
| Create links that apply a coupon and redirect to a specific page | `coupon.share` | `website_id` | link to one record | Website | `website` | `n : 1` | `not declared` | stored |
| Generate Coupons | `loyalty.generate.wizard` | `customer_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Generate Coupons | `loyalty.generate.wizard` | `customer_tag_ids` | list on both sides | Partner Tags | `res.partner.category` | `0..n : 0..n` | `not declared` | stored |
| Generate Coupons | `loyalty.generate.wizard` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 1` | `not declared` | stored |
| History for Loyalty cards and Electronic Wallets | `loyalty.history` | `card_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 1` | `cascade` | stored |
| Loyalty Communication | `loyalty.mail` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 1` | `cascade` | stored |
| Loyalty Communication | `loyalty.mail` | `pos_report_print_id` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | stored |
| Loyalty Communication | `loyalty.mail` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 1` | `cascade` | stored |
| Loyalty Coupon | `loyalty.card` | `history_ids` | list of records | History for Loyalty cards and Electronic Wallets | `loyalty.history` | `1 : 0..n` | `mirror of the target column` | derived |
| Loyalty Coupon | `loyalty.card` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Loyalty Coupon | `loyalty.card` | `order_id_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Loyalty Coupon | `loyalty.card` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Loyalty Coupon | `loyalty.card` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 0..1` | `restrict` | stored |
| Loyalty Coupon | `loyalty.card` | `source_pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Loyalty Coupon | `loyalty.card` | `source_pos_order_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Loyalty Program | `loyalty.program` | `communication_plan_ids` | list of records | Loyalty Communication | `loyalty.mail` | `1 : 0..n` | `mirror of the target column` | derived |
| Loyalty Program | `loyalty.program` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Loyalty Program | `loyalty.program` | `coupon_ids` | list of records | Loyalty Coupon | `loyalty.card` | `1 : 0..n` | `mirror of the target column` | derived |
| Loyalty Program | `loyalty.program` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Loyalty Program | `loyalty.program` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | derived |
| Loyalty Program | `loyalty.program` | `payment_program_discount_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Loyalty Program | `loyalty.program` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Program | `loyalty.program` | `pos_report_print_id` | link to one record | Report Action | `ir.actions.report` | `n : 0..1` | `not declared` | derived |
| Loyalty Program | `loyalty.program` | `pricelist_ids` | list on both sides | Pricelist | `product.pricelist` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Program | `loyalty.program` | `reward_ids` | list of records | Loyalty Reward | `loyalty.reward` | `1 : 0..n` | `mirror of the target column` | derived |
| Loyalty Program | `loyalty.program` | `rule_ids` | list of records | Loyalty Rule | `loyalty.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Loyalty Reward | `loyalty.reward` | `all_discount_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `discount_line_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `restrict` | stored |
| Loyalty Reward | `loyalty.reward` | `discount_product_category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `discount_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `discount_product_tag_id` | link to one record | Product Tag | `product.tag` | `n : 0..1` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 1` | `cascade` | stored |
| Loyalty Reward | `loyalty.reward` | `reward_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `reward_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `reward_product_tag_id` | link to one record | Product Tag | `product.tag` | `n : 0..1` | `not declared` | stored |
| Loyalty Reward | `loyalty.reward` | `reward_product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Loyalty Rule | `loyalty.rule` | `product_category_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | stored |
| Loyalty Rule | `loyalty.rule` | `product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Loyalty Rule | `loyalty.rule` | `product_tag_id` | link to one record | Product Tag | `product.tag` | `n : 0..1` | `not declared` | stored |
| Loyalty Rule | `loyalty.rule` | `program_id` | link to one record | Loyalty Program | `loyalty.program` | `n : 1` | `cascade` | stored |
| Loyalty Rule | `loyalty.rule` | `valid_product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | stored |
| Sale Loyalty - Apply Coupon Wizard | `sale.loyalty.coupon.wizard` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `not declared` | stored |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `not declared` | stored |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `reward_ids` | list on both sides | Loyalty Reward | `loyalty.reward` | `0..n : 0..n` | `not declared` | derived |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `selected_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Sale Loyalty - Reward Selection Wizard | `sale.loyalty.reward.wizard` | `selected_reward_id` | link to one record | Loyalty Reward | `loyalty.reward` | `n : 0..1` | `not declared` | stored |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `coupon_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 1` | `cascade` | stored |
| Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `cascade` | stored |
| Update Loyalty Card Points | `loyalty.card.update.balance` | `card_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 1` | `not declared` | stored |

### 3.29 Payment Providers

Payment providers, payment methods, tokens and transactions with their state machine, capture and refund flows.

Specified in the domain folder `../domains/payment-providers/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Payment Method | `payment.method` | `payment_method` | Persistent record with 10 stored columns; owns Payment Method; referenced by 5 relation fields. | Payment Engine |
| Payment Provider | `payment.provider` | `payment_provider` | Persistent record with 100 stored columns; belongs to Companies; lifecycle states Disabled, Enabled, Test Mode; 2 state fields in all; company scoped; referenced by 7 relation fields. | Payment Engine |
| Payment Token | `payment.token` | `payment_token` | Persistent record with 14 stored columns; belongs to Contact, Payment Method, Payment Provider; owns Payment Transaction; states of `demo_simulated_state`: Pending, Confirmed, Canceled, Error; referenced by 5 relation fields. | Payment Engine |
| Payment Transaction | `payment.transaction` | `payment_transaction` | Persistent record with 32 stored columns; belongs to Contact, Currency, Payment Method and 1 further required links; owns Payment Transaction; lifecycle states Draft, Pending, Authorized, Confirmed, Canceled, Error; referenced by 7 relation fields. | Payment Engine |

#### Interactive assistant entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Generate Payment Link | `payment.link.wizard` | `payment_link_wizard` | Interactive assistant with 11 stored columns; company scoped. | Payment Engine |
| Payment Capture Wizard | `payment.capture.wizard` | `payment_capture_wizard` | Interactive assistant with 2 stored columns. | Payment Engine |

#### Relationships (41)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Generate Payment Link | `payment.link.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Generate Payment Link | `payment.link.wizard` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Generate Payment Link | `payment.link.wizard` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Payment Capture Wizard | `payment.capture.wizard` | `transaction_ids` | list on both sides | Payment Transaction | `payment.transaction` | `0..n : 0..n` | `not declared` | stored |
| Payment Method | `payment.method` | `brand_ids` | list of records | Payment Method | `payment.method` | `1 : 0..n` | `mirror of the target column` | derived |
| Payment Method | `payment.method` | `l10n_ec_sri_payment_id` | link to one record | SRI Payment Method | `l10n_ec.sri.payment` | `n : 0..1` | `not declared` | stored |
| Payment Method | `payment.method` | `primary_payment_method_id` | link to one record | Payment Method | `payment.method` | `n : 0..1` | `not declared` | stored |
| Payment Method | `payment.method` | `provider_ids` | list on both sides | Payment Provider | `payment.provider` | `0..n : 0..n` | `not declared` | stored |
| Payment Method | `payment.method` | `supported_country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Payment Method | `payment.method` | `supported_currency_ids` | list on both sides | Currency | `res.currency` | `0..n : 0..n` | `not declared` | stored |
| Payment Provider | `payment.provider` | `available_country_ids` | list on both sides | Country | `res.country` | `0..n : 0..n` | `not declared` | stored |
| Payment Provider | `payment.provider` | `available_currency_ids` | list on both sides | Currency | `res.currency` | `0..n : 0..n` | `not declared` | stored |
| Payment Provider | `payment.provider` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Payment Provider | `payment.provider` | `express_checkout_form_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `restrict` | stored |
| Payment Provider | `payment.provider` | `inline_form_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `restrict` | stored |
| Payment Provider | `payment.provider` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Payment Provider | `payment.provider` | `mercado_pago_account_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Payment Provider | `payment.provider` | `module_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `not declared` | stored |
| Payment Provider | `payment.provider` | `payment_method_ids` | list on both sides | Payment Method | `payment.method` | `0..n : 0..n` | `not declared` | stored |
| Payment Provider | `payment.provider` | `paymob_account_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Payment Provider | `payment.provider` | `redirect_form_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `restrict` | stored |
| Payment Provider | `payment.provider` | `token_inline_form_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `restrict` | stored |
| Payment Provider | `payment.provider` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `restrict` | stored |
| Payment Token | `payment.token` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Payment Token | `payment.token` | `payment_method_id` | link to one record | Payment Method | `payment.method` | `n : 1` | `not declared` | stored |
| Payment Token | `payment.token` | `provider_id` | link to one record | Payment Provider | `payment.provider` | `n : 1` | `not declared` | stored |
| Payment Token | `payment.token` | `transaction_ids` | list of records | Payment Transaction | `payment.transaction` | `1 : 0..n` | `mirror of the target column` | derived |
| Payment Transaction | `payment.transaction` | `child_transaction_ids` | list of records | Payment Transaction | `payment.transaction` | `1 : 0..n` | `mirror of the target column` | derived |
| Payment Transaction | `payment.transaction` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `partner_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `restrict` | stored |
| Payment Transaction | `payment.transaction` | `partner_state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `payment_method_id` | link to one record | Payment Method | `payment.method` | `n : 1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `primary_payment_method_id` | link to one record | Payment Method | `payment.method` | `n : 0..1` | `not declared` | derived |
| Payment Transaction | `payment.transaction` | `provider_id` | link to one record | Payment Provider | `payment.provider` | `n : 1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `sale_order_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `source_transaction_id` | link to one record | Payment Transaction | `payment.transaction` | `n : 0..1` | `not declared` | stored |
| Payment Transaction | `payment.transaction` | `token_id` | link to one record | Payment Token | `payment.token` | `n : 0..1` | `restrict` | stored |

### 3.30 Point of Sale

Point of sale configurations, sessions, orders and order lines, payments and payment methods, cash control, receipts, restaurant floors and tables, presets, self-ordering and payment terminals.

Specified in [`../domains/point-of-sale/`](../domains/point-of-sale/).

#### Persistent entities (17)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Coins/Bills | `pos.bill` | `pos_bill` | Persistent record with 2 stored columns; referenced by 1 relation field. | Point of Sale |
| Custom links that the restaurant can configure to be displayed on the self order screen | `pos_self_order.custom_link` | `pos_self_order_custom_link` | Persistent record with 5 stored columns. | point of sale Self Order |
| Easily load a set of configuration options | `pos.preset` | `pos_preset` | Persistent record with 14 stored columns; referenced by 5 relation fields. | Point of Sale |
| Point of Sale Category | `pos.category` | `pos_category` | Persistent record with 6 stored columns; owns Point of Sale Category; referenced by 7 relation fields. | Point of Sale |
| Point of Sale Configuration | `pos.config` | `pos_config` | Persistent record with 92 stored columns; belongs to Companies, Picking Type; owns Point of Sale Session; states of `status`: Inactive, Active; company scoped; referenced by 15 relation fields. | Point of Sale |
| Point of Sale Note | `pos.note` | `pos_note` | Persistent record with 3 stored columns; referenced by 1 relation field. | Point of Sale |
| Point of Sale Order Lines | `pos.order.line` | `pos_order_line` | Persistent record with 36 stored columns; belongs to Point of Sale Orders, Product Variant; owns Event Registration, Point of Sale Order Lines, Product Attribute Custom Value and 1 further collections; company scoped; referenced by 6 relation fields. | Point of Sale |
| Point of Sale Orders | `pos.order` | `pos_order` | Persistent record with 71 stored columns; belongs to Companies; owns Journal Entry, Point of Sale Order Lines, Point of Sale Payments and 3 further collections; lifecycle states New, Cancelled, Paid, Posted; 7 state fields in all; company scoped; referenced by 16 relation fields. | Point of Sale |
| Point of Sale Payment Methods | `pos.payment.method` | `pos_payment_method` | Persistent record with 76 stored columns; company scoped; referenced by 10 relation fields. | Point of Sale |
| Point of Sale Payments | `pos.payment` | `pos_payment` | Persistent record with 30 stored columns; belongs to Point of Sale Orders, Point of Sale Payment Methods; company scoped. | Point of Sale |
| Point of Sale Printer | `pos.printer` | `pos_printer` | Persistent record with 5 stored columns; belongs to Companies; company scoped; referenced by 1 relation field. | Point of Sale |
| point of sale Restaurant Order Course | `restaurant.order.course` | `restaurant_order_course` | Persistent record with 5 stored columns; belongs to Point of Sale Orders; owns Point of Sale Order Lines; referenced by 1 relation field. | Restaurant |
| Point of Sale Session | `pos.session` | `pos_session` | Persistent record with 17 stored columns; belongs to Point of Sale Configuration, User; owns Bank Statement Line, Payments, Point of Sale Orders and 1 further collections; lifecycle states x; company scoped; referenced by 9 relation fields. | Point of Sale |
| Restaurant Floor | `restaurant.floor` | `restaurant_floor` | Persistent record with 4 stored columns; owns Restaurant Table; referenced by 2 relation fields. | Restaurant |
| Restaurant Table | `restaurant.table` | `restaurant_table` | Persistent record with 12 stored columns; referenced by 3 relation fields. | Restaurant |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `pos_pack_operation_lot` | Persistent record with 2 stored columns. | Point of Sale |
| Transaction Lipa na M-PESA | `transaction.lipa.na.mpesa` | `transaction_lipa_na_mpesa` | Persistent record with 5 stored columns. | point of sale Safaricom |

#### Interactive assistant entities (6)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Close Session Wizard | `pos.close.session.wizard` | `pos_close_session_wizard` | Interactive assistant with 4 stored columns. | Point of Sale |
| Confirmation Wizard | `pos.confirmation.wizard` | `pos_confirmation_wizard` | Interactive assistant with 1 stored column. | Point of Sale |
| Multiple order invoice creation | `pos.make.invoice` | `pos_make_invoice` | Interactive assistant with 1 stored column. | Point of Sale |
| Point of Sale Daily Report | `pos.daily.sales.reports.wizard` | `pos_daily_sales_reports_wizard` | Interactive assistant with 2 stored columns; belongs to Point of Sale Session. | Point of Sale |
| Point of Sale Details Report | `pos.details.wizard` | `pos_details_wizard` | Interactive assistant with 2 stored columns. | Point of Sale |
| Point of Sale Make Payment Wizard | `pos.make.payment` | `pos_make_payment` | Interactive assistant with 5 stored columns; belongs to Point of Sale Configuration, Point of Sale Payment Methods. | Point of Sale |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Bus Mixin | `pos.bus.mixin` | `none` | Shared behaviour merged into 3 entities. | Point of Sale |
| Point of Sale data loading mixin | `pos.load.mixin` | `none` | Shared behaviour merged into 68 entities. | Point of Sale |

#### Relationships (162)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Close Session Wizard | `pos.close.session.wizard` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Coins/Bills | `pos.bill` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Custom links that the restaurant can configure to be displayed on the self order screen | `pos_self_order.custom_link` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Easily load a set of configuration options | `pos.preset` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Easily load a set of configuration options | `pos.preset` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Easily load a set of configuration options | `pos.preset` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Easily load a set of configuration options | `pos.preset` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Point of Sale Category | `pos.category` | `child_ids` | list of records | Point of Sale Category | `pos.category` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Category | `pos.category` | `parent_id` | link to one record | Point of Sale Category | `pos.category` | `n : 0..1` | `not declared` | stored |
| Point of Sale Category | `pos.category` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `advanced_employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `available_preset_ids` | list on both sides | Easily load a set of configuration options | `pos.preset` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `available_pricelist_ids` | list on both sides | Pricelist | `product.pricelist` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `basic_employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `crm_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Point of Sale Configuration | `pos.config` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `current_session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | derived |
| Point of Sale Configuration | `pos.config` | `current_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Point of Sale Configuration | `pos.config` | `default_bill_ids` | list on both sides | Coins/Bills | `pos.bill` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `default_fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `default_preset_id` | link to one record | Easily load a set of configuration options | `pos.preset` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `device_seq_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `discount_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `down_payment_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `fallback_nomenclature_id` | link to one record | Barcode Nomenclature | `barcode.nomenclature` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `fast_payment_method_ids` | list on both sides | Point of Sale Payment Methods | `pos.payment.method` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `fiscal_position_ids` | list on both sides | Fiscal Position | `account.fiscal.position` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `floor_ids` | list on both sides | Restaurant Floor | `restaurant.floor` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `group_pos_manager_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `group_pos_user_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `iface_available_categ_ids` | list on both sides | Point of Sale Category | `pos.category` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `invoice_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `restrict` | stored |
| Point of Sale Configuration | `pos.config` | `l10n_es_simplified_invoice_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `l10n_vn_pos_symbol` | link to one record | SInvoice symbol | `l10n_vn_edi_viettel.sinvoice.symbol` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `minimal_employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `note_ids` | list on both sides | Point of Sale Note | `pos.note` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `order_backend_seq_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `order_line_seq_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `order_seq_id` | link to one record | Sequence | `ir.sequence` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `payment_method_ids` | list on both sides | Point of Sale Payment Methods | `pos.payment.method` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 1` | `restrict` | stored |
| Point of Sale Configuration | `pos.config` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `printer_ids` | list on both sides | Point of Sale Printer | `pos.printer` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `rounding_method` | link to one record | Account Cash Rounding | `account.cash.rounding` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `route_id` | link to one record | Inventory Routes | `stock.route` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_order_online_payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_ordering_available_language_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_ordering_default_language_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_ordering_default_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_ordering_image_background_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `self_ordering_image_home_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `session_ids` | list of records | Point of Sale Session | `pos.session` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Configuration | `pos.config` | `simplified_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Point of Sale Configuration | `pos.config` | `sms_receipt_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `tip_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `trusted_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Configuration | `pos.config` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `restrict` | stored |
| Point of Sale Daily Report | `pos.daily.sales.reports.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | derived |
| Point of Sale Daily Report | `pos.daily.sales.reports.wizard` | `pos_session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 1` | `not declared` | stored |
| Point of Sale Details Report | `pos.details.wizard` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Make Payment Wizard | `pos.make.payment` | `config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 1` | `not declared` | stored |
| Point of Sale Make Payment Wizard | `pos.make.payment` | `payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `combo_id` | link to one record | Product Combo | `product.combo` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `combo_item_id` | link to one record | Product Combo Item | `product.combo.item` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `combo_line_ids` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Order Lines | `pos.order.line` | `combo_parent_id` | link to one record | Point of Sale Order Lines | `pos.order.line` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `coupon_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 0..1` | `restrict` | stored |
| Point of Sale Order Lines | `pos.order.line` | `course_id` | link to one record | point of sale Restaurant Order Course | `restaurant.order.course` | `n : 0..1` | `set null` | stored |
| Point of Sale Order Lines | `pos.order.line` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Point of Sale Order Lines | `pos.order.line` | `custom_attribute_value_ids` | list of records | Product Attribute Custom Value | `product.attribute.custom.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Order Lines | `pos.order.line` | `event_registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Order Lines | `pos.order.line` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 1` | `cascade` | stored |
| Point of Sale Order Lines | `pos.order.line` | `pack_lot_ids` | list of records | Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Order Lines | `pos.order.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Point of Sale Order Lines | `pos.order.line` | `refund_orderline_ids` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Order Lines | `pos.order.line` | `refunded_orderline_id` | link to one record | Point of Sale Order Lines | `pos.order.line` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `reward_id` | link to one record | Loyalty Reward | `loyalty.reward` | `n : 0..1` | `restrict` | stored |
| Point of Sale Order Lines | `pos.order.line` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `sale_order_origin_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Order Lines | `pos.order.line` | `tax_ids_after_fiscal_position` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `Consolidated Invoices` | list on both sides | MyInvois Document | `myinvois.document` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `account_move` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `available_payment_method_ids` | list on both sides | Point of Sale Payment Methods | `pos.payment.method` | `0..n : 0..n` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `course_ids` | list of records | point of sale Restaurant Order Course | `restaurant.order.course` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `crm_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Point of Sale Orders | `pos.order` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `l10n_es_edi_verifactu_document_ids` | list of records | Veri*Factu Document | `l10n_es_edi_verifactu.document` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `l10n_es_tbai_post_document_id` | link to one record | TicketBAI Document | `l10n_es_edi_tbai.document` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `l10n_id_qris_transaction_ids` | list on both sides | Record of QRIS transactions | `l10n_id.qris.transaction` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `l10n_jo_edi_pos_xml_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `lines` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `online_payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 0..1` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `payment_ids` | list of records | Point of Sale Payments | `pos.payment` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `picking_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `picking_type_id` | link to one record | Picking Type | `stock.picking.type` | `n : 0..1` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `preset_id` | link to one record | Easily load a set of configuration options | `pos.preset` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `previous_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `refunded_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `reversed_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Orders | `pos.order` | `sale_journal` | link to one record | Journal | `account.journal` | `n : 0..1` | `restrict` | stored |
| Point of Sale Orders | `pos.order` | `self_ordering_table_id` | link to one record | Restaurant Table | `restaurant.table` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `session_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | derived |
| Point of Sale Orders | `pos.order` | `stock_reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `table_id` | link to one record | Restaurant Table | `restaurant.table` | `n : 0..1` | `not declared` | stored |
| Point of Sale Orders | `pos.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `restrict` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `mollie_payment_provider_id` | link to one record | Payment Provider | `payment.provider` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `online_payment_provider_ids` | list on both sides | Payment Provider | `payment.provider` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `open_session_ids` | list on both sides | Point of Sale Session | `pos.session` | `0..n : 0..n` | `not declared` | derived |
| Point of Sale Payment Methods | `pos.payment.method` | `outstanding_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Point of Sale Payment Methods | `pos.payment.method` | `receivable_account_id` | link to one record | Account | `account.account` | `n : 0..1` | `restrict` | stored |
| Point of Sale Payments | `pos.payment` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Point of Sale Payments | `pos.payment` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `online_account_payment_id` | link to one record | Payments | `account.payment` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Point of Sale Payments | `pos.payment` | `payment_method_id` | link to one record | Point of Sale Payment Methods | `pos.payment.method` | `n : 1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `pos_order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 1` | `cascade` | stored |
| Point of Sale Payments | `pos.payment` | `session_id` | link to one record | Point of Sale Session | `pos.session` | `n : 0..1` | `not declared` | stored |
| Point of Sale Payments | `pos.payment` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Point of Sale Printer | `pos.printer` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Point of Sale Printer | `pos.printer` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Printer | `pos.printer` | `product_categories_ids` | list on both sides | Point of Sale Category | `pos.category` | `0..n : 0..n` | `not declared` | stored |
| Point of Sale Session | `pos.session` | `bank_payment_ids` | list of records | Payments | `account.payment` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Session | `pos.session` | `cash_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Point of Sale Session | `pos.session` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Point of Sale Session | `pos.session` | `config_id` | link to one record | Point of Sale Configuration | `pos.config` | `n : 1` | `not declared` | stored |
| Point of Sale Session | `pos.session` | `crm_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | derived |
| Point of Sale Session | `pos.session` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Point of Sale Session | `pos.session` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Point of Sale Session | `pos.session` | `move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Point of Sale Session | `pos.session` | `order_ids` | list of records | Point of Sale Orders | `pos.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Session | `pos.session` | `payment_method_ids` | list on both sides | Point of Sale Payment Methods | `pos.payment.method` | `0..n : 0..n` | `not declared` | derived |
| Point of Sale Session | `pos.session` | `picking_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Session | `pos.session` | `statement_line_ids` | list of records | Bank Statement Line | `account.bank.statement.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Point of Sale Session | `pos.session` | `user_id` | link to one record | User | `res.users` | `n : 1` | `restrict` | stored |
| Restaurant Floor | `restaurant.floor` | `pos_config_ids` | list on both sides | Point of Sale Configuration | `pos.config` | `0..n : 0..n` | `not declared` | stored |
| Restaurant Floor | `restaurant.floor` | `table_ids` | list of records | Restaurant Table | `restaurant.table` | `1 : 0..n` | `mirror of the target column` | derived |
| Restaurant Table | `restaurant.table` | `floor_id` | link to one record | Restaurant Floor | `restaurant.floor` | `n : 0..1` | `not declared` | stored |
| Restaurant Table | `restaurant.table` | `parent_id` | link to one record | Restaurant Table | `restaurant.table` | `n : 0..1` | `not declared` | stored |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 0..1` | `not declared` | derived |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `pos_order_line_id` | link to one record | Point of Sale Order Lines | `pos.order.line` | `n : 0..1` | `not declared` | stored |
| Specify product lot/serial number in point of sale order line | `pos.pack.operation.lot` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| point of sale Restaurant Order Course | `restaurant.order.course` | `line_ids` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| point of sale Restaurant Order Course | `restaurant.order.course` | `order_id` | link to one record | Point of Sale Orders | `pos.order` | `n : 1` | `cascade` | stored |

### 3.31 Pricing and Pricelists

Pricelists, pricelist rules, computation bases, minimum quantities, date validity and discount policies.

Specified in [`../domains/pricing-and-pricelists/`](../domains/pricing-and-pricelists/).

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Product Margin | `product.margin` | `product_margin` | Interactive assistant with 3 stored columns; states of `invoice_state`: Paid, Open and Paid, Draft, Open and Paid. | Margins by Products |

### 3.32 Sales

Quotations, sales orders and order lines, quotation templates, invoicing policies, down payments, margins and quotation documents.

Specified in [`../domains/sales/`](../domains/sales/).

#### Persistent entities (10)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| customer relationship management Tag | `crm.tag` | `crm_tag` | Persistent record with 2 stored columns; referenced by 5 relation fields. | Sales Teams |
| Form fields of inside quotation documents. | `sale.pdf.form.field` | `sale_pdf_form_field` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Sales Portable Document Format Quotation Builder |
| Quotation Template | `sale.order.template` | `sale_order_template` | Persistent record with 11 stored columns; owns Quotation Template Line; company scoped; referenced by 4 relation fields. | Sales |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_line` | Persistent record with 9 stored columns; belongs to Quotation Template; referenced by 1 relation field. | Sales |
| Quotation's Headers & Footers | `quotation.document` | `quotation_document` | Persistent record with 5 stored columns; belongs to Attachment; referenced by 4 relation fields. | Sales Portable Document Format Quotation Builder |
| Sales Analysis Report | `sale.report` | `none, read from a stored query` | Persistent record with 0 stored columns; lifecycle states x; 3 state fields in all; company scoped. | Sales |
| Sales Order | `sale.order` | `sale_order` | Persistent record with 73 stored columns; belongs to Companies, Contact; owns Event Booth, Expense, Point of Sale Order Lines and 4 further collections; lifecycle states x; 3 state fields in all; company scoped; referenced by 26 relation fields. | Sales |
| Sales Order Line | `sale.order.line` | `sale_order_line` | Persistent record with 59 stored columns; belongs to Sales Order; owns Analytic Line, Event Booth, Event Booth Registration and 8 further collections; carries the state field `state`; 2 state fields in all; referenced by 25 relation fields. | Sales |
| Sales Team | `crm.team` | `crm_team` | Persistent record with 13 stored columns; owns Point of Sale Configuration, Sales Team Member, Survey and 1 further collections; company scoped; referenced by 23 relation fields. | Sales Teams |
| Sales Team Member | `crm.team.member` | `crm_team_member` | Persistent record with 7 stored columns; belongs to Sales Team, User; company scoped. | Sales Teams |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Cancel multiple quotations | `sale.mass.cancel.orders` | `sale_mass_cancel_orders` | Interactive assistant with 0 stored columns. | Sales |
| Create new or use existing Customer on new Quotation | `crm.quotation.partner` | `crm_quotation_partner` | Interactive assistant with 3 stored columns; belongs to Lead. | Opportunity to Quotation |
| Discount Wizard | `sale.order.discount` | `sale_order_discount` | Interactive assistant with 4 stored columns; belongs to Sales Order. | Sales |
| Sales Advance Payment Invoice | `sale.advance.payment.inv` | `sale_advance_payment_inv` | Interactive assistant with 10 stored columns; company scoped. | Sales |

#### Relationships (146)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Cancel multiple quotations | `sale.mass.cancel.orders` | `sale_order_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | stored |
| Create new or use existing Customer on new Quotation | `crm.quotation.partner` | `lead_id` | link to one record | Lead | `crm.lead` | `n : 1` | `not declared` | stored |
| Create new or use existing Customer on new Quotation | `crm.quotation.partner` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Discount Wizard | `sale.order.discount` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `not declared` | stored |
| Form fields of inside quotation documents. | `sale.pdf.form.field` | `product_document_ids` | list on both sides | Product Document | `product.document` | `0..n : 0..n` | `not declared` | stored |
| Form fields of inside quotation documents. | `sale.pdf.form.field` | `quotation_document_ids` | list on both sides | Quotation's Headers & Footers | `quotation.document` | `0..n : 0..n` | `not declared` | stored |
| Quotation Template | `sale.order.template` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Quotation Template | `sale.order.template` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Quotation Template | `sale.order.template` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Quotation Template | `sale.order.template` | `quotation_document_ids` | list on both sides | Quotation's Headers & Footers | `quotation.document` | `0..n : 0..n` | `not declared` | stored |
| Quotation Template | `sale.order.template` | `sale_order_template_line_ids` | list of records | Quotation Template Line | `sale.order.template.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Quotation Template Line | `sale.order.template.line` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Quotation Template Line | `sale.order.template.line` | `parent_id` | link to one record | Quotation Template Line | `sale.order.template.line` | `n : 0..1` | `not declared` | derived |
| Quotation Template Line | `sale.order.template.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Quotation Template Line | `sale.order.template.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Quotation Template Line | `sale.order.template.line` | `sale_order_template_id` | link to one record | Quotation Template | `sale.order.template` | `n : 1` | `cascade` | stored |
| Quotation's Headers & Footers | `quotation.document` | `form_field_ids` | list on both sides | Form fields of inside quotation documents. | `sale.pdf.form.field` | `0..n : 0..n` | `not declared` | stored |
| Quotation's Headers & Footers | `quotation.document` | `ir_attachment_id` | link to one record | Attachment | `ir.attachment` | `n : 1` | `cascade` | stored |
| Quotation's Headers & Footers | `quotation.document` | `quotation_template_ids` | list on both sides | Quotation Template | `sale.order.template` | `0..n : 0..n` | `not declared` | stored |
| Sales Advance Payment Invoice | `sale.advance.payment.inv` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Sales Advance Payment Invoice | `sale.advance.payment.inv` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Sales Advance Payment Invoice | `sale.advance.payment.inv` | `sale_order_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | stored |
| Sales Analysis Report | `sale.report` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `categ_id` | link to one record | Product Category | `product.category` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `commercial_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `industry_id` | link to one record | Industry | `res.partner.industry` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `medium_id` | link to one record | campaign tracking parameter Medium | `utm.medium` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `source_id` | link to one record | campaign tracking parameter Source | `utm.source` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | derived |
| Sales Analysis Report | `sale.report` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `applied_coupon_ids` | list on both sides | Loyalty Coupon | `loyalty.card` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `assigned_grade_id` | link to one record | Partner Grade | `res.partner.grade` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `authorized_transaction_ids` | list on both sides | Payment Transaction | `payment.transaction` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `available_quotation_document_ids` | list on both sides | Quotation's Headers & Footers | `quotation.document` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `carrier_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `code_enabled_rule_ids` | list on both sides | Loyalty Rule | `loyalty.rule` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Sales Order | `sale.order` | `coupon_point_ids` | list of records | Sale Order Coupon Points - Keeps track of how a sale order impacts a coupon | `sale.order.coupon.points` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `restrict` | stored |
| Sales Order | `sale.order` | `disabled_auto_rewards` | list on both sides | Loyalty Reward | `loyalty.reward` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `duplicated_order_ids` | list on both sides | Sales Order | `sale.order` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `event_booth_ids` | list of records | Event Booth | `event.booth` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `expense_ids` | list of records | Expense | `hr.expense` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `fiscal_position_id` | link to one record | Fiscal Position | `account.fiscal.position` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `grid_product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `incoterm` | link to one record | Incoterms | `account.incoterms` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `invoice_ids` | list on both sides | Journal Entry | `account.move` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `l10n_ec_sri_payment_id` | link to one record | SRI Payment Method | `l10n_ec.sri.payment` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `l10n_in_reseller_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `l10n_it_edi_doi_id` | link to one record | Declaration of Intent | `l10n_it_edi_doi.declaration_of_intent` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `mrp_production_ids` | list on both sides | Manufacturing Order | `mrp.production` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `opportunity_id` | link to one record | Lead | `crm.lead` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `order_line` | list of records | Sales Order Line | `sale.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Sales Order | `sale.order` | `partner_invoice_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Sales Order | `sale.order` | `partner_shipping_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Sales Order | `sale.order` | `payment_term_id` | link to one record | Payment Terms | `account.payment.term` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `pending_email_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `set null` | stored |
| Sales Order | `sale.order` | `picking_ids` | list of records | Transfer | `stock.picking` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `pos_order_line_ids` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `preferred_payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `project_account_id` | link to one record | Analytic Account | `account.analytic.account` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `project_ids` | list on both sides | Project | `project.project` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `quotation_document_ids` | list on both sides | Quotation's Headers & Footers | `quotation.document` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `repair_order_ids` | list of records | Repair Order | `repair.order` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order | `sale.order` | `sale_order_template_id` | link to one record | Quotation Template | `sale.order.template` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `stock_reference_ids` | list on both sides | Reference between stock documents | `stock.reference` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `tag_ids` | list on both sides | customer relationship management Tag | `crm.tag` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `tasks_ids` | list on both sides | Task | `project.task` | `0..n : 0..n` | `not declared` | derived |
| Sales Order | `sale.order` | `tax_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Sales Order | `sale.order` | `timesheet_encode_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Sales Order | `sale.order` | `transaction_ids` | list on both sides | Payment Transaction | `payment.transaction` | `0..n : 0..n` | `not declared` | stored |
| Sales Order | `sale.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Sales Order | `sale.order` | `website_order_line` | list of records | Sales Order Line | `sale.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `allowed_uom_ids` | list on both sides | Product Unit of Measure | `uom.uom` | `0..n : 0..n` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `analytic_line_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `available_product_document_ids` | list on both sides | Product Document | `product.document` | `0..n : 0..n` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `combo_item_id` | link to one record | Product Combo Item | `product.combo.item` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `coupon_id` | link to one record | Loyalty Coupon | `loyalty.card` | `n : 0..1` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `event_booth_category_id` | link to one record | Event Booth Category | `event.booth.category` | `n : 0..1` | `set null` | stored |
| Sales Order Line | `sale.order.line` | `event_booth_ids` | list of records | Event Booth | `event.booth` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `event_booth_pending_ids` | list on both sides | Event Booth | `event.booth` | `0..n : 0..n` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `event_booth_registration_ids` | list of records | Event Booth Registration | `event.booth.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `expense_id` | link to one record | Expense | `hr.expense` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `expense_ids` | list of records | Expense | `hr.expense` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `invoice_lines` | list on both sides | Journal Item | `account.move.line` | `0..n : 0..n` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `linked_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `cascade` | stored |
| Sales Order Line | `sale.order.line` | `linked_line_ids` | list of records | Sales Order Line | `sale.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `move_ids` | list of records | Stock Move | `stock.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `cascade` | stored |
| Sales Order Line | `sale.order.line` | `parent_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `pos_order_line_ids` | list of records | Point of Sale Order Lines | `pos.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `pricelist_item_id` | link to one record | Pricelist Rule | `product.pricelist.item` | `n : 0..1` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `product_custom_attribute_value_ids` | list of records | Product Attribute Custom Value | `product.attribute.custom.value` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `product_document_ids` | list on both sides | Product Document | `product.document` | `0..n : 0..n` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `product_no_variant_attribute_value_ids` | list on both sides | Product Template Attribute Value | `product.template.attribute.value` | `0..n : 0..n` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `product_template_id` | link to one record | Product | `product.template` | `n : 0..1` | `not declared` | derived |
| Sales Order Line | `sale.order.line` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `purchase_line_ids` | list of records | Purchase Order Line | `purchase.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `reached_milestones_ids` | list of records | Project Milestone | `project.milestone` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `reward_id` | link to one record | Loyalty Reward | `loyalty.reward` | `n : 0..1` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `route_ids` | list on both sides | Inventory Routes | `stock.route` | `0..n : 0..n` | `restrict` | stored |
| Sales Order Line | `sale.order.line` | `task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Sales Order Line | `sale.order.line` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Order Line | `sale.order.line` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Sales Team | `crm.team` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Sales Team | `crm.team` | `crm_team_member_all_ids` | list of records | Sales Team Member | `crm.team.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Team | `crm.team` | `crm_team_member_ids` | list of records | Sales Team Member | `crm.team.member` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Team | `crm.team` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Sales Team | `crm.team` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Sales Team | `crm.team` | `member_company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | derived |
| Sales Team | `crm.team` | `member_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Sales Team | `crm.team` | `origin_survey_ids` | list of records | Survey | `survey.survey` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Team | `crm.team` | `pos_config_ids` | list of records | Point of Sale Configuration | `pos.config` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Team | `crm.team` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Sales Team | `crm.team` | `website_ids` | list of records | Website | `website` | `1 : 0..n` | `mirror of the target column` | derived |
| Sales Team Member | `crm.team.member` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Sales Team Member | `crm.team.member` | `crm_team_id` | link to one record | Sales Team | `crm.team` | `n : 1` | `cascade` | stored |
| Sales Team Member | `crm.team.member` | `user_company_ids` | list on both sides | Companies | `res.company` | `0..n : 0..n` | `not declared` | derived |
| Sales Team Member | `crm.team.member` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Sales Team Member | `crm.team.member` | `user_in_teams_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |

### 3.33 Website and Storefront

Websites, pages and menus, themes, content blocks, the editor, forms, visitors and tracking, blogs, search-engine metadata, rewrites, the online shop, the cart and checkout, wishlists and comparison.

Specified in [`../domains/website-and-storefront/`](../domains/website-and-storefront/).

#### Persistent entities (37)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| All Website Route | `website.route` | `website_route` | Persistent record with 1 stored column; referenced by 1 relation field. | Website |
| Blog | `blog.blog` | `blog_blog` | Persistent record with 13 stored columns; owns Blog Post; referenced by 1 relation field. | Blog |
| Blog Post | `blog.post` | `blog_post` | Persistent record with 22 stored columns; belongs to Blog; referenced by 1 relation field. | Blog |
| Blog Tag | `blog.tag` | `blog_tag` | Persistent record with 9 stored columns; referenced by 1 relation field. | Blog |
| Blog Tag Category | `blog.tag.category` | `blog_tag_category` | Persistent record with 1 stored column; owns Blog Tag; referenced by 1 relation field. | Blog |
| Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `website_sale_extra_field` | Persistent record with 3 stored columns; belongs to Fields. | Electronic Commerce |
| Forum | `forum.forum` | `forum_forum` | Persistent record with 55 stored columns; owns Course, Forum Post, Forum Tag; referenced by 4 relation fields. | Forum |
| Forum Post | `forum.post` | `forum_post` | Persistent record with 27 stored columns; belongs to Forum; owns Forum Post, Post Vote; lifecycle states Active, Waiting Validation, Closed, Offensive, Flagged; referenced by 4 relation fields. | Forum |
| Forum Tag | `forum.tag` | `forum_tag` | Persistent record with 10 stored columns; belongs to Forum; referenced by 1 relation field. | Forum |
| Model Page | `website.controller.page` | `website_controller_page` | Persistent record with 8 stored columns; belongs to View; owns Website Menu; referenced by 1 relation field. | Website |
| Multi Website Published Mixin | `website.published.multi.mixin` | `none in the observed installation` | Persistent record with 0 stored columns. | Website |
| Page | `website.page` | `website_page` | Persistent record with 13 stored columns; belongs to View; owns Website Menu; referenced by 6 relation fields. | Website |
| Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `res_partner_tag` | Persistent record with 4 stored columns; referenced by 1 relation field. | Customer References |
| Post Closing Reason | `forum.post.reason` | `forum_post_reason` | Persistent record with 2 stored columns; referenced by 1 relation field. | Forum |
| Post Vote | `forum.post.vote` | `forum_post_vote` | Persistent record with 5 stored columns; belongs to Forum Post, User. | Forum |
| Product Feed | `product.feed` | `product_feed` | Persistent record with 8 stored columns; belongs to Languages, Website. | Electronic Commerce |
| Product Image | `product.image` | `product_image` | Persistent record with 6 stored columns. | Electronic Commerce |
| Product ribbon | `product.ribbon` | `product_ribbon` | Persistent record with 8 stored columns; referenced by 2 relation fields. | Electronic Commerce |
| Product Wishlist | `product.wishlist` | `product_wishlist` | Persistent record with 6 stored columns; belongs to Product Variant, Website. | Shopper's Wishlist |
| Rich Text Editor Converter Subtest | `html_editor.converter.test.sub` | `html_editor_converter_test_sub` | Persistent record with 1 stored column; referenced by 1 relation field. | hypertext markup language Editor |
| Rich Text Editor Converter Test | `html_editor.converter.test` | `html_editor_converter_test` | Persistent record with 11 stored columns. | hypertext markup language Editor |
| Theme Asset | `theme.ir.asset` | `theme_ir_asset` | Persistent record with 8 stored columns; owns Asset; referenced by 1 relation field. | Website |
| Theme Attachments | `theme.ir.attachment` | `theme_ir_attachment` | Persistent record with 3 stored columns; owns Attachment; referenced by 1 relation field. | Website |
| Theme user interface View | `theme.ir.ui.view` | `theme_ir_ui_view` | Persistent record with 10 stored columns; owns View; referenced by 2 relation fields. | Website |
| Unit of Measure for price per unit on Electronic Commerce products. | `website.base.unit` | `website_base_unit` | Persistent record with 1 stored column; referenced by 2 relation fields. | Electronic Commerce |
| Visited Pages | `website.track` | `website_track` | Persistent record with 5 stored columns; belongs to Website Visitor. | Website |
| Website | `website` | `website` | Persistent record with 79 stored columns; belongs to Companies, Languages, User; owns Electronic Commerce Extra Info Shown on product page, Pricelist; company scoped; referenced by 27 relation fields. | Website |
| Website Checkout Step | `website.checkout.step` | `website_checkout_step` | Persistent record with 7 stored columns. | Electronic Commerce |
| Website Configurator Feature | `website.configurator.feature` | `website_configurator_feature` | Persistent record with 11 stored columns. | Website |
| Website Menu | `website.menu` | `website_menu` | Persistent record with 12 stored columns; owns Website Menu; referenced by 4 relation fields. | Website |
| Website Product Category | `product.public.category` | `product_public_category` | Persistent record with 16 stored columns; owns Website Product Category; referenced by 4 relation fields. | Electronic Commerce |
| Website rewrite | `website.rewrite` | `website_rewrite` | Persistent record with 8 stored columns. | Website |
| Website Snippet Filter | `website.snippet.filter` | `website_snippet_filter` | Persistent record with 9 stored columns. | Website |
| Website Technical Page | `website.technical.page` | `none, read from a stored query` | Persistent record with 0 stored columns. | Website |
| Website Theme Menu | `theme.website.menu` | `theme_website_menu` | Persistent record with 9 stored columns; owns Website Menu; referenced by 2 relation fields. | Website |
| Website Theme Page | `theme.website.page` | `theme_website_page` | Persistent record with 10 stored columns; belongs to Theme user interface View; owns Page; referenced by 2 relation fields. | Website |
| Website Visitor | `website.visitor` | `website_visitor` | Persistent record with 9 stored columns; owns Discussion Channel, Event Registration, Track / Visitor Link and 1 further collections; referenced by 6 relation fields. | Website |

#### Interactive assistant entities (7)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Grant Portal Access | `portal.wizard` | `portal_wizard` | Interactive assistant with 1 stored column; owns Portal User Config; referenced by 1 relation field. | Customer Portal |
| Page Properties | `website.page.properties` | `website_page_properties` | Interactive assistant with 3 stored columns. | Website |
| Page Properties Base | `website.page.properties.base` | `website_page_properties_base` | Interactive assistant with 3 stored columns; belongs to Website; owns Website Menu. | Website |
| Portal Sharing | `portal.share` | `portal_share` | Interactive assistant with 3 stored columns. | Customer Portal |
| Portal User Config | `portal.wizard.user` | `portal_wizard_user` | Interactive assistant with 3 stored columns; belongs to Contact, Grant Portal Access; states of `email_state`: Valid, Invalid, Already Registered. | Customer Portal |
| Robots.txt Editor | `website.robots` | `website_robots` | Interactive assistant with 1 stored column. | Website |
| User list of blocked 3rd-party domains | `website.custom_blocked_third_party_domains` | `website_custom_blocked_third_party_domains` | Interactive assistant with 1 stored column. | Website |

#### Shared behaviour entities (12)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Assets Utils | `website.assets` | `none` | Shared behaviour definition reused through composition. | Website |
| Cover Properties Website Mixin | `website.cover_properties.mixin` | `none` | Shared behaviour merged into 4 entities. | Website |
| Field rich text History | `html.field.history.mixin` | `none` | Shared behaviour merged into 1 entity. | hypertext markup language Editor |
| hypertext markup language Text Processor Abstract Model | `website.html.text.processor` | `none` | Shared behaviour definition reused through composition. | Website |
| Multi Website Mixin | `website.multi.mixin` | `none` | Shared behaviour merged into 6 entities. | Website |
| Portal Mixin | `portal.mixin` | `none` | Shared behaviour merged into 8 entities. | Customer Portal |
| search engine optimization metadata | `website.seo.metadata` | `none` | Shared behaviour merged into 16 entities. | Website |
| Theme Utils | `theme.utils` | `none` | Shared behaviour definition reused through composition. | Website |
| Website page/record specific options | `website.page_options.mixin` | `none` | Shared behaviour merged into 2 entities. | Website |
| Website page/record specific visibility options | `website.page_visibility_options.mixin` | `none` | Shared behaviour merged into 3 entities. | Website |
| Website Published Mixin | `website.published.mixin` | `none` | Shared behaviour merged into 8 entities. | Website |
| Website Searchable Mixin | `website.searchable.mixin` | `none` | Shared behaviour merged into 15 entities. | Website |

#### Relationships (131)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Blog | `blog.blog` | `blog_post_ids` | list of records | Blog Post | `blog.post` | `1 : 0..n` | `mirror of the target column` | derived |
| Blog Post | `blog.post` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Blog Post | `blog.post` | `blog_id` | link to one record | Blog | `blog.blog` | `n : 1` | `cascade` | stored |
| Blog Post | `blog.post` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Blog Post | `blog.post` | `tag_ids` | list on both sides | Blog Tag | `blog.tag` | `0..n : 0..n` | `not declared` | stored |
| Blog Post | `blog.post` | `write_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Blog Tag | `blog.tag` | `category_id` | link to one record | Blog Tag Category | `blog.tag.category` | `n : 0..1` | `not declared` | stored |
| Blog Tag | `blog.tag` | `post_ids` | list on both sides | Blog Post | `blog.post` | `0..n : 0..n` | `not declared` | stored |
| Blog Tag Category | `blog.tag.category` | `tag_ids` | list of records | Blog Tag | `blog.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 1` | `cascade` | stored |
| Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Forum | `forum.forum` | `authorized_group_id` | link to one record | Access Groups | `res.groups` | `n : 0..1` | `not declared` | stored |
| Forum | `forum.forum` | `last_post_id` | link to one record | Forum Post | `forum.post` | `n : 0..1` | `not declared` | derived |
| Forum | `forum.forum` | `post_ids` | list of records | Forum Post | `forum.post` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum | `forum.forum` | `slide_channel_id` | link to one record | Course | `slide.channel` | `n : 0..1` | `not declared` | stored |
| Forum | `forum.forum` | `slide_channel_ids` | list of records | Course | `slide.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum | `forum.forum` | `tag_ids` | list of records | Forum Tag | `forum.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum | `forum.forum` | `tag_most_used_ids` | list of records | Forum Tag | `forum.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum | `forum.forum` | `tag_unused_ids` | list of records | Forum Tag | `forum.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum Post | `forum.post` | `child_ids` | list of records | Forum Post | `forum.post` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum Post | `forum.post` | `closed_reason_id` | link to one record | Post Closing Reason | `forum.post.reason` | `n : 0..1` | `not declared` | stored |
| Forum Post | `forum.post` | `closed_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Forum Post | `forum.post` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Forum Post | `forum.post` | `favourite_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Forum Post | `forum.post` | `flag_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Forum Post | `forum.post` | `forum_id` | link to one record | Forum | `forum.forum` | `n : 1` | `not declared` | stored |
| Forum Post | `forum.post` | `moderator_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Forum Post | `forum.post` | `parent_id` | link to one record | Forum Post | `forum.post` | `n : 0..1` | `cascade` | stored |
| Forum Post | `forum.post` | `tag_ids` | list on both sides | Forum Tag | `forum.tag` | `0..n : 0..n` | `not declared` | stored |
| Forum Post | `forum.post` | `vote_ids` | list of records | Post Vote | `forum.post.vote` | `1 : 0..n` | `mirror of the target column` | derived |
| Forum Post | `forum.post` | `write_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Forum Tag | `forum.tag` | `forum_id` | link to one record | Forum | `forum.forum` | `n : 1` | `not declared` | stored |
| Forum Tag | `forum.tag` | `post_ids` | list on both sides | Forum Post | `forum.post` | `0..n : 0..n` | `not declared` | stored |
| Grant Portal Access | `portal.wizard` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Grant Portal Access | `portal.wizard` | `user_ids` | list of records | Portal User Config | `portal.wizard.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Model Page | `website.controller.page` | `menu_ids` | list of records | Website Menu | `website.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Model Page | `website.controller.page` | `record_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `cascade` | stored |
| Model Page | `website.controller.page` | `view_id` | link to one record | View | `ir.ui.view` | `n : 1` | `cascade` | stored |
| Multi Website Mixin | `website.multi.mixin` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `restrict` | derived |
| Page | `website.page` | `menu_ids` | list of records | Website Menu | `website.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Page | `website.page` | `theme_template_id` | link to one record | Website Theme Page | `theme.website.page` | `n : 0..1` | `not declared` | stored |
| Page | `website.page` | `view_id` | link to one record | View | `ir.ui.view` | `n : 1` | `cascade` | stored |
| Page | `website.page` | `view_write_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Page Properties | `website.page.properties` | `target_model_id` | link to one record | Page | `website.page` | `n : 0..1` | `not declared` | stored |
| Page Properties Base | `website.page.properties.base` | `menu_ids` | list of records | Website Menu | `website.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Page Properties Base | `website.page.properties.base` | `website_id` | link to one record | Website | `website` | `n : 1` | `not declared` | stored |
| Partner Tags - These tags can be used on website to find customers by sector, or ... | `res.partner.tag` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Portal Sharing | `portal.share` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Portal User Config | `portal.wizard.user` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Portal User Config | `portal.wizard.user` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Portal User Config | `portal.wizard.user` | `wizard_id` | link to one record | Grant Portal Access | `portal.wizard` | `n : 1` | `cascade` | stored |
| Post Vote | `forum.post.vote` | `forum_id` | link to one record | Forum | `forum.forum` | `n : 0..1` | `not declared` | stored |
| Post Vote | `forum.post.vote` | `post_id` | link to one record | Forum Post | `forum.post` | `n : 1` | `cascade` | stored |
| Post Vote | `forum.post.vote` | `recipient_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Post Vote | `forum.post.vote` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Product Feed | `product.feed` | `lang_id` | link to one record | Languages | `res.lang` | `n : 1` | `not declared` | stored |
| Product Feed | `product.feed` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Product Feed | `product.feed` | `product_category_ids` | list on both sides | Website Product Category | `product.public.category` | `0..n : 0..n` | `not declared` | stored |
| Product Feed | `product.feed` | `website_id` | link to one record | Website | `website` | `n : 1` | `not declared` | stored |
| Product Image | `product.image` | `product_tmpl_id` | link to one record | Product | `product.template` | `n : 0..1` | `cascade` | stored |
| Product Image | `product.image` | `product_variant_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Product Wishlist | `product.wishlist` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Product Wishlist | `product.wishlist` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Product Wishlist | `product.wishlist` | `pricelist_id` | link to one record | Pricelist | `product.pricelist` | `n : 0..1` | `not declared` | stored |
| Product Wishlist | `product.wishlist` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Product Wishlist | `product.wishlist` | `website_id` | link to one record | Website | `website` | `n : 1` | `cascade` | stored |
| Rich Text Editor Converter Test | `html_editor.converter.test` | `many2one` | link to one record | Rich Text Editor Converter Subtest | `html_editor.converter.test.sub` | `n : 0..1` | `not declared` | stored |
| Theme Asset | `theme.ir.asset` | `copy_ids` | list of records | Asset | `ir.asset` | `1 : 0..n` | `mirror of the target column` | derived |
| Theme Attachments | `theme.ir.attachment` | `copy_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Theme user interface View | `theme.ir.ui.view` | `copy_ids` | list of records | View | `ir.ui.view` | `1 : 0..n` | `mirror of the target column` | derived |
| Visited Pages | `website.track` | `page_id` | link to one record | Page | `website.page` | `n : 0..1` | `cascade` | stored |
| Visited Pages | `website.track` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `cascade` | stored |
| Visited Pages | `website.track` | `visitor_id` | link to one record | Website Visitor | `website.visitor` | `n : 1` | `cascade` | stored |
| Website | `website` | `cart_recovery_mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `channel_id` | link to one record | Livechat Channel | `im_livechat.channel` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Website | `website` | `confirmation_email_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `crm_default_team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `crm_default_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Website | `website` | `default_lang_id` | link to one record | Languages | `res.lang` | `n : 1` | `not declared` | stored |
| Website | `website` | `in_store_dm_id` | link to one record | Shipping Methods | `delivery.carrier` | `n : 0..1` | `not declared` | derived |
| Website | `website` | `language_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | stored |
| Website | `website` | `menu_id` | link to one record | Website Menu | `website.menu` | `n : 0..1` | `not declared` | derived |
| Website | `website` | `newsletter_id` | link to one record | Mailing List | `mailing.list` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `pricelist_ids` | list of records | Pricelist | `product.pricelist` | `1 : 0..n` | `mirror of the target column` | derived |
| Website | `website` | `salesperson_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `salesteam_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Website | `website` | `shop_extra_field_ids` | list of records | Electronic Commerce Extra Info Shown on product page | `website.sale.extra.field` | `1 : 0..n` | `mirror of the target column` | derived |
| Website | `website` | `theme_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `not declared` | stored |
| Website | `website` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Website | `website` | `warehouse_id` | link to one record | Warehouse | `stock.warehouse` | `n : 0..1` | `not declared` | stored |
| Website Checkout Step | `website.checkout.step` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |
| Website Configurator Feature | `website.configurator.feature` | `module_id` | link to one record | Module | `ir.module.module` | `n : 0..1` | `cascade` | stored |
| Website Configurator Feature | `website.configurator.feature` | `page_view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `cascade` | stored |
| Website Menu | `website.menu` | `child_id` | list of records | Website Menu | `website.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Menu | `website.menu` | `controller_page_id` | link to one record | Model Page | `website.controller.page` | `n : 0..1` | `cascade` | stored |
| Website Menu | `website.menu` | `group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Website Menu | `website.menu` | `page_id` | link to one record | Page | `website.page` | `n : 0..1` | `cascade` | stored |
| Website Menu | `website.menu` | `parent_id` | link to one record | Website Menu | `website.menu` | `n : 0..1` | `cascade` | stored |
| Website Menu | `website.menu` | `theme_template_id` | link to one record | Website Theme Menu | `theme.website.menu` | `n : 0..1` | `not declared` | stored |
| Website Menu | `website.menu` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |
| Website Product Category | `product.public.category` | `child_id` | list of records | Website Product Category | `product.public.category` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Product Category | `product.public.category` | `parent_id` | link to one record | Website Product Category | `product.public.category` | `n : 0..1` | `cascade` | stored |
| Website Product Category | `product.public.category` | `parents_and_self` | list on both sides | Website Product Category | `product.public.category` | `0..n : 0..n` | `not declared` | derived |
| Website Product Category | `product.public.category` | `product_tmpl_ids` | list on both sides | Product | `product.template` | `0..n : 0..n` | `not declared` | stored |
| Website Snippet Filter | `website.snippet.filter` | `action_server_id` | link to one record | Server Actions | `ir.actions.server` | `n : 0..1` | `cascade` | stored |
| Website Snippet Filter | `website.snippet.filter` | `filter_id` | link to one record | Filters | `ir.filters` | `n : 0..1` | `cascade` | stored |
| Website Snippet Filter | `website.snippet.filter` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |
| Website Theme Menu | `theme.website.menu` | `copy_ids` | list of records | Website Menu | `website.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Theme Menu | `theme.website.menu` | `page_id` | link to one record | Website Theme Page | `theme.website.page` | `n : 0..1` | `cascade` | stored |
| Website Theme Menu | `theme.website.menu` | `parent_id` | link to one record | Website Theme Menu | `theme.website.menu` | `n : 0..1` | `cascade` | stored |
| Website Theme Page | `theme.website.page` | `copy_ids` | list of records | Page | `website.page` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Theme Page | `theme.website.page` | `view_id` | link to one record | Theme user interface View | `theme.ir.ui.view` | `n : 1` | `cascade` | stored |
| Website Visitor | `website.visitor` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Website Visitor | `website.visitor` | `discuss_channel_ids` | list of records | Discussion Channel | `discuss.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Visitor | `website.visitor` | `event_registered_ids` | list on both sides | Event | `event.event` | `0..n : 0..n` | `not declared` | derived |
| Website Visitor | `website.visitor` | `event_registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Visitor | `website.visitor` | `event_track_visitor_ids` | list of records | Track / Visitor Link | `event.track.visitor` | `1 : 0..n` | `mirror of the target column` | derived |
| Website Visitor | `website.visitor` | `event_track_wishlisted_ids` | list on both sides | Event Track | `event.track` | `0..n : 0..n` | `not declared` | derived |
| Website Visitor | `website.visitor` | `lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | stored |
| Website Visitor | `website.visitor` | `last_visited_page_id` | link to one record | Page | `website.page` | `n : 0..1` | `not declared` | derived |
| Website Visitor | `website.visitor` | `lead_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Website Visitor | `website.visitor` | `livechat_operator_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Website Visitor | `website.visitor` | `page_ids` | list on both sides | Page | `website.page` | `0..n : 0..n` | `not declared` | derived |
| Website Visitor | `website.visitor` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Website Visitor | `website.visitor` | `product_ids` | list on both sides | Product Variant | `product.product` | `0..n : 0..n` | `not declared` | derived |
| Website Visitor | `website.visitor` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | stored |
| Website Visitor | `website.visitor` | `website_track_ids` | list of records | Visited Pages | `website.track` | `1 : 0..n` | `mirror of the target column` | derived |
| Website rewrite | `website.rewrite` | `route_id` | link to one record | All Website Route | `website.route` | `n : 0..1` | `not declared` | stored |
| Website rewrite | `website.rewrite` | `website_id` | link to one record | Website | `website` | `n : 0..1` | `cascade` | stored |

### 3.34 Projects and Tasks

Projects, tasks, stages, milestones, recurrences, collaborators, project updates, profitability, tags and to-do items.

Specified in [`../domains/projects-and-tasks/`](../domains/projects-and-tasks/).

#### Persistent entities (12)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Collaborators in project shared | `project.collaborator` | `project_collaborator` | Persistent record with 3 stored columns; belongs to Contact, Project. | Project |
| Personal Task Stage | `project.task.stage.personal` | `project_task_user_rel` | Persistent record with 3 stored columns; belongs to Task, User; referenced by 1 relation field. | Project |
| Project | `project.project` | `project_project` | Persistent record with 30 stored columns; owns Analytic Line, Collaborators in project shared, Project Milestone and 3 further collections; states of `last_update_status`: On Track, At Risk, Off Track, On Hold, Set Status, Complete; 2 state fields in all; company scoped; referenced by 25 relation fields. | Project |
| Project Milestone | `project.milestone` | `project_milestone` | Persistent record with 8 stored columns; belongs to Project; owns Task; referenced by 6 relation fields. | Project |
| Project Role | `project.role` | `project_role` | Persistent record with 4 stored columns; referenced by 2 relation fields. | Project |
| Project Stage | `project.project.stage` | `project_project_stage` | Persistent record with 8 stored columns; company scoped; referenced by 2 relation fields. | Project |
| Project Tags | `project.tags` | `project_tags` | Persistent record with 2 stored columns; referenced by 4 relation fields. | Project |
| Project Update | `project.update` | `project_update` | Persistent record with 13 stored columns; belongs to Project, User; states of `status`: On Track, At Risk, Off Track, On Hold, Complete; referenced by 1 relation field. | Project |
| Rating | `rating.rating` | `rating_rating` | Persistent record with 21 stored columns; referenced by 2 relation fields. | Customer Rating |
| Task | `project.task` | `project_task` | Persistent record with 45 stored columns; owns Analytic Line, Attachment, Skill level for employee and 1 further collections; lifecycle states x; 2 state fields in all; company scoped; referenced by 17 relation fields. | Project |
| Task Recurrence | `project.task.recurrence` | `project_task_recurrence` | Persistent record with 4 stored columns; owns Task; referenced by 1 relation field. | Project |
| Task Stage | `project.task.type` | `project_task_type` | Persistent record with 15 stored columns; states of `rating_status`: when reaching this stage, on a periodic basis; referenced by 9 relation fields. | Project |

#### Interactive assistant entities (8)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Create activity and to-do at the same time | `mail.activity.todo.create` | `mail_activity_todo_create` | Interactive assistant with 4 stored columns; belongs to User. | To-Do |
| Project role to users mapping | `project.template.role.to.users.map` | `project_template_role_to_users_map` | Interactive assistant with 2 stored columns; belongs to Project Role. | Project |
| Project Sharing | `project.share.wizard` | `project_share_wizard` | Interactive assistant with 3 stored columns; owns Project Sharing Collaborator Wizard; referenced by 1 relation field. | Project |
| Project Sharing Collaborator Wizard | `project.share.collaborator.wizard` | `project_share_collaborator_wizard` | Interactive assistant with 4 stored columns; belongs to Contact. | Project |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `project_project_stage_delete_wizard` | Interactive assistant with 0 stored columns. | Project |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `project_task_type_delete_wizard` | Interactive assistant with 0 stored columns. | Project |
| Project Template create Wizard | `project.template.create.wizard` | `project_template_create_wizard` | Interactive assistant with 7 stored columns; owns Project role to users mapping; referenced by 1 relation field. | Project |
| Task Sharing | `task.share.wizard` | `task_share_wizard` | Interactive assistant with 4 stored columns. | Project |

#### Shared behaviour entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Burndown Chart | `project.task.burndown.chart.report` | `none` | Shared behaviour definition reused through composition. | Project |
| Rating Mixin | `rating.mixin` | `none` | Shared behaviour merged into 4 entities. | Customer Rating |
| Rating Parent Mixin | `rating.parent.mixin` | `none` | Shared behaviour merged into 2 entities. | Customer Rating |

#### Relationships (98)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Burndown Chart | `project.task.burndown.chart.report` | `milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | derived |
| Burndown Chart | `project.task.burndown.chart.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Burndown Chart | `project.task.burndown.chart.report` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | derived |
| Burndown Chart | `project.task.burndown.chart.report` | `stage_id` | link to one record | Task Stage | `project.task.type` | `n : 0..1` | `not declared` | derived |
| Burndown Chart | `project.task.burndown.chart.report` | `tag_ids` | list on both sides | Project Tags | `project.tags` | `0..n : 0..n` | `not declared` | stored |
| Burndown Chart | `project.task.burndown.chart.report` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Collaborators in project shared | `project.collaborator` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Collaborators in project shared | `project.collaborator` | `project_id` | link to one record | Project | `project.project` | `n : 1` | `not declared` | stored |
| Create activity and to-do at the same time | `mail.activity.todo.create` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Personal Task Stage | `project.task.stage.personal` | `stage_id` | link to one record | Task Stage | `project.task.type` | `n : 0..1` | `set null` | stored |
| Personal Task Stage | `project.task.stage.personal` | `task_id` | link to one record | Task | `project.task` | `n : 1` | `cascade` | stored |
| Personal Task Stage | `project.task.stage.personal` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Project | `project.project` | `account_id` | link to one record | Analytic Account | `account.analytic.account` | `n : 0..1` | `set null` | stored |
| Project | `project.project` | `collaborator_ids` | list of records | Collaborators in project shared | `project.collaborator` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Project | `project.project` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Project | `project.project` | `last_update_id` | link to one record | Project Update | `project.update` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `milestone_ids` | list of records | Project Milestone | `project.milestone` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `next_milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | derived |
| Project | `project.project` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `reinvoiced_sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | derived |
| Project | `project.project` | `sale_line_employee_ids` | list of records | Project Sales line, employee mapping | `project.sale.line.employee.map` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `stage_id` | link to one record | Project Stage | `project.project.stage` | `n : 0..1` | `restrict` | stored |
| Project | `project.project` | `tag_ids` | list on both sides | Project Tags | `project.tags` | `0..n : 0..n` | `not declared` | stored |
| Project | `project.project` | `task_ids` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `tasks` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `timesheet_encode_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | derived |
| Project | `project.project` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `timesheet_product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Project | `project.project` | `type_ids` | list on both sides | Task Stage | `project.task.type` | `0..n : 0..n` | `not declared` | stored |
| Project | `project.project` | `update_ids` | list of records | Project Update | `project.update` | `1 : 0..n` | `mirror of the target column` | derived |
| Project | `project.project` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Project Milestone | `project.milestone` | `project_id` | link to one record | Project | `project.project` | `n : 1` | `cascade` | stored |
| Project Milestone | `project.milestone` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Project Milestone | `project.milestone` | `task_ids` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Project Sharing | `project.share.wizard` | `collaborator_ids` | list of records | Project Sharing Collaborator Wizard | `project.share.collaborator.wizard` | `1 : 0..n` | `mirror of the target column` | derived |
| Project Sharing | `project.share.wizard` | `existing_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Project Sharing Collaborator Wizard | `project.share.collaborator.wizard` | `parent_wizard_id` | link to one record | Project Sharing | `project.share.wizard` | `n : 0..1` | `not declared` | stored |
| Project Sharing Collaborator Wizard | `project.share.collaborator.wizard` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Project Stage | `project.project.stage` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Project Stage | `project.project.stage` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Project Stage | `project.project.stage` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Project Stage Delete Wizard | `project.project.stage.delete.wizard` | `stage_ids` | list on both sides | Project Stage | `project.project.stage` | `0..n : 0..n` | `cascade` | stored |
| Project Tags | `project.tags` | `project_ids` | list on both sides | Project | `project.project` | `0..n : 0..n` | `not declared` | stored |
| Project Tags | `project.tags` | `task_ids` | list on both sides | Task | `project.task` | `0..n : 0..n` | `not declared` | stored |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `project_ids` | list on both sides | Project | `project.project` | `0..n : 0..n` | `cascade` | stored |
| Project Task Stage Delete Wizard | `project.task.type.delete.wizard` | `stage_ids` | list on both sides | Task Stage | `project.task.type` | `0..n : 0..n` | `cascade` | stored |
| Project Template create Wizard | `project.template.create.wizard` | `alias_domain_id` | link to one record | Email Domain | `mail.alias.domain` | `n : 0..1` | `not declared` | stored |
| Project Template create Wizard | `project.template.create.wizard` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Project Template create Wizard | `project.template.create.wizard` | `role_to_users_ids` | list of records | Project role to users mapping | `project.template.role.to.users.map` | `1 : 0..n` | `mirror of the target column` | derived |
| Project Template create Wizard | `project.template.create.wizard` | `template_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Project Update | `project.update` | `project_id` | link to one record | Project | `project.project` | `n : 1` | `not declared` | stored |
| Project Update | `project.update` | `uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Project Update | `project.update` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Project role to users mapping | `project.template.role.to.users.map` | `role_id` | link to one record | Project Role | `project.role` | `n : 1` | `not declared` | stored |
| Project role to users mapping | `project.template.role.to.users.map` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Project role to users mapping | `project.template.role.to.users.map` | `wizard_id` | link to one record | Project Template create Wizard | `project.template.create.wizard` | `n : 0..1` | `not declared` | stored |
| Rating | `rating.rating` | `message_id` | link to one record | Message | `mail.message` | `n : 0..1` | `cascade` | stored |
| Rating | `rating.rating` | `parent_res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Rating | `rating.rating` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Rating | `rating.rating` | `publisher_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `set null` | stored |
| Rating | `rating.rating` | `rated_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Rating | `rating.rating` | `res_model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Rating Parent Mixin | `rating.parent.mixin` | `rating_ids` | list of records | Rating | `rating.rating` | `1 : 0..n` | `mirror of the target column` | derived |
| Task | `project.task` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Task | `project.task` | `child_ids` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Task | `project.task` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `depend_on_ids` | list on both sides | Task | `project.task` | `0..n : 0..n` | `not declared` | stored |
| Task | `project.task` | `dependent_ids` | list on both sides | Task | `project.task` | `0..n : 0..n` | `not declared` | stored |
| Task | `project.task` | `displayed_image_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `last_sol_of_customer` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | derived |
| Task | `project.task` | `milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `parent_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `personal_stage_id` | link to one record | Personal Task Stage | `project.task.stage.personal` | `n : 0..1` | `not declared` | derived |
| Task | `project.task` | `personal_stage_type_id` | link to one record | Task Stage | `project.task.type` | `n : 0..1` | `not declared` | derived |
| Task | `project.task` | `personal_stage_type_ids` | list on both sides | Task Stage | `project.task.type` | `0..n : 0..n` | `restrict` | stored |
| Task | `project.task` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `project_sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | derived |
| Task | `project.task` | `recurrence_id` | link to one record | Task Recurrence | `project.task.recurrence` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `role_ids` | list on both sides | Project Role | `project.role` | `0..n : 0..n` | `not declared` | stored |
| Task | `project.task` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Task | `project.task` | `stage_id` | link to one record | Task Stage | `project.task.type` | `n : 0..1` | `restrict` | stored |
| Task | `project.task` | `tag_ids` | list on both sides | Project Tags | `project.tags` | `0..n : 0..n` | `not declared` | stored |
| Task | `project.task` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Task | `project.task` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Task | `project.task` | `user_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Task Recurrence | `project.task.recurrence` | `task_ids` | list of records | Task | `project.task` | `1 : 0..n` | `mirror of the target column` | derived |
| Task Sharing | `task.share.wizard` | `task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Task Stage | `project.task.type` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Task Stage | `project.task.type` | `project_ids` | list on both sides | Project | `project.project` | `0..n : 0..n` | `not declared` | stored |
| Task Stage | `project.task.type` | `rating_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Task Stage | `project.task.type` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `not declared` | stored |
| Task Stage | `project.task.type` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |

### 3.35 Timesheets

Timesheet lines on tasks and projects, employee hourly cost, timesheet billing to customers and the comparison with attendances.

Specified in [`../domains/timesheets/`](../domains/timesheets/).

#### Persistent entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Personal Filters on Employees for the Calendar view | `account.analytic.line.calendar.employee` | `account_analytic_line_calendar_employee` | Persistent record with 4 stored columns; belongs to User. | Task Logs |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `project_sale_line_employee_map` | Persistent record with 7 stored columns; belongs to Employee, Project; company scoped. | Sales Timesheet |
| Timesheets Analysis Report | `timesheets.analysis.report` | `timesheets_analysis_report` | Persistent record with 22 stored columns; company scoped. | Task Logs |

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Employee Delete Wizard | `hr.employee.delete.wizard` | `hr_employee_delete_wizard` | Interactive assistant with 0 stored columns. | Task Logs |

#### Relationships (24)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Employee Delete Wizard | `hr.employee.delete.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Personal Filters on Employees for the Calendar view | `account.analytic.line.calendar.employee` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Personal Filters on Employees for the Calendar view | `account.analytic.line.calendar.employee` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `cost_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `existing_employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | derived |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `project_id` | link to one record | Project | `project.project` | `n : 1` | `not declared` | stored |
| Project Sales line, employee mapping | `project.sale.line.employee.map` | `sale_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `message_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Timesheets Analysis Report | `timesheets.analysis.report` | `milestone_id` | link to one record | Project Milestone | `project.milestone` | `n : 0..1` | `not declared` | derived |
| Timesheets Analysis Report | `timesheets.analysis.report` | `order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `parent_task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `project_id` | link to one record | Project | `project.project` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `so_line` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `task_id` | link to one record | Task | `project.task` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `timesheet_invoice_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Timesheets Analysis Report | `timesheets.analysis.report` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |

### 3.36 Attendances and Working Time

Working schedules and their attendance lines, resource calendars and leaves, resources, check-in and check-out records, overtime rules and rulesets.

Specified in [`../domains/attendances-and-working-time/`](../domains/attendances-and-working-time/).

#### Persistent entities (9)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Attendance | `hr.attendance` | `hr_attendance` | Persistent record with 21 stored columns; belongs to Employee; states of `overtime_status`: To Approve, Approved, Refused; referenced by 2 relation fields. | Attendances |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `hr_attendance_overtime_line` | Persistent record with 9 stored columns; belongs to Employee; states of `status`: To Approve, Approved, Refused; referenced by 1 relation field. | Attendances |
| Overtime Rule | `hr.attendance.overtime.rule` | `hr_attendance_overtime_rule` | Persistent record with 17 stored columns; belongs to Overtime Ruleset; referenced by 1 relation field. | Attendances |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `hr_attendance_overtime_ruleset` | Persistent record with 6 stored columns; owns Overtime Rule; company scoped; referenced by 2 relation fields. | Attendances |
| Resource Time Off Detail | `resource.calendar.leaves` | `resource_calendar_leaves` | Persistent record with 10 stored columns; owns Analytic Line; company scoped; referenced by 2 relation fields. | Resource |
| Resource Working Time | `resource.calendar` | `resource_calendar` | Persistent record with 11 stored columns; owns Resource Time Off Detail, Work Detail; company scoped; referenced by 18 relation fields. | Resource |
| Resources | `resource.resource` | `resource_resource` | Persistent record with 9 stored columns; owns Employee; company scoped; referenced by 4 relation fields. | Resource |
| Timesheet Attendance Report | `hr.timesheet.attendance.report` | `hr_timesheet_attendance_report` | Persistent record with 9 stored columns; company scoped. | Timesheets/attendances reporting |
| Work Detail | `resource.calendar.attendance` | `resource_calendar_attendance` | Persistent record with 12 stored columns; belongs to Resource Working Time. | Resource |

#### Shared behaviour entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Resource Mixin | `resource.mixin` | `none` | Shared behaviour merged into 2 entities. | Resource |

#### Relationships (36)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Attendance | `hr.attendance` | `attendance_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Attendance | `hr.attendance` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | derived |
| Attendance | `hr.attendance` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Attendance | `hr.attendance` | `linked_overtime_ids` | list on both sides | Attendance Overtime Line | `hr.attendance.overtime.line` | `0..n : 0..n` | `not declared` | derived |
| Attendance | `hr.attendance` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | derived |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Attendance Overtime Line | `hr.attendance.overtime.line` | `rule_ids` | list on both sides | Overtime Rule | `hr.attendance.overtime.rule` | `0..n : 0..n` | `not declared` | stored |
| Overtime Rule | `hr.attendance.overtime.rule` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Overtime Rule | `hr.attendance.overtime.rule` | `ruleset_id` | link to one record | Overtime Ruleset | `hr.attendance.overtime.ruleset` | `n : 1` | `not declared` | stored |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Overtime Ruleset | `hr.attendance.overtime.ruleset` | `rule_ids` | list of records | Overtime Rule | `hr.attendance.overtime.rule` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Mixin | `resource.mixin` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Resource Mixin | `resource.mixin` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | derived |
| Resource Mixin | `resource.mixin` | `resource_id` | link to one record | Resources | `resource.resource` | `n : 1` | `restrict` | derived |
| Resource Time Off Detail | `resource.calendar.leaves` | `calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Resource Time Off Detail | `resource.calendar.leaves` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Resource Time Off Detail | `resource.calendar.leaves` | `holiday_id` | link to one record | Time Off | `hr.leave` | `n : 0..1` | `not declared` | stored |
| Resource Time Off Detail | `resource.calendar.leaves` | `resource_id` | link to one record | Resources | `resource.resource` | `n : 0..1` | `not declared` | stored |
| Resource Time Off Detail | `resource.calendar.leaves` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Time Off Detail | `resource.calendar.leaves` | `work_entry_type_id` | link to one record | human resources Work Entry Type | `hr.work.entry.type` | `n : 0..1` | `not declared` | stored |
| Resource Working Time | `resource.calendar` | `attendance_ids` | list of records | Work Detail | `resource.calendar.attendance` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Working Time | `resource.calendar` | `attendance_ids_1st_week` | list of records | Work Detail | `resource.calendar.attendance` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Working Time | `resource.calendar` | `attendance_ids_2nd_week` | list of records | Work Detail | `resource.calendar.attendance` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Working Time | `resource.calendar` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Resource Working Time | `resource.calendar` | `global_leave_ids` | list of records | Resource Time Off Detail | `resource.calendar.leaves` | `1 : 0..n` | `mirror of the target column` | derived |
| Resource Working Time | `resource.calendar` | `leave_ids` | list of records | Resource Time Off Detail | `resource.calendar.leaves` | `1 : 0..n` | `mirror of the target column` | derived |
| Resources | `resource.resource` | `calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Resources | `resource.resource` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Resources | `resource.resource` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | derived |
| Resources | `resource.resource` | `employee_id` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Resources | `resource.resource` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Timesheet Attendance Report | `hr.timesheet.attendance.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Timesheet Attendance Report | `hr.timesheet.attendance.report` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Work Detail | `resource.calendar.attendance` | `calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 1` | `cascade` | stored |
| Work Detail | `resource.calendar.attendance` | `work_entry_type_id` | link to one record | human resources Work Entry Type | `hr.work.entry.type` | `n : 0..1` | `not declared` | stored |

### 3.37 Expenses

Employee expenses, expense reports, approval, reimbursement or company payment and re-invoicing to customers.

Specified in [`../domains/expenses/`](../domains/expenses/).

#### Persistent entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Expense | `hr.expense` | `hr_expense` | Persistent record with 32 stored columns; belongs to Companies, Currency, Employee; owns Attachment; lifecycle states Draft, Submitted, Approved, Posted, In Payment, Paid, Refused; 2 state fields in all; company scoped; referenced by 10 relation fields. | Expenses |

#### Interactive assistant entities (5)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Expense Approve Duplicate | `hr.expense.approve.duplicate` | `hr_expense_approve_duplicate` | Interactive assistant with 0 stored columns. | Expenses |
| Expense Posting Wizard | `hr.expense.post.wizard` | `hr_expense_post_wizard` | Interactive assistant with 3 stored columns; company scoped. | Expenses |
| Expense Refuse Reason Wizard | `hr.expense.refuse.wizard` | `hr_expense_refuse_wizard` | Interactive assistant with 1 stored column. | Expenses |
| Expense Split | `hr.expense.split` | `hr_expense_split` | Interactive assistant with 14 stored columns; belongs to Employee, Product Variant; states of `approval_state`: x; company scoped. | Expenses |
| Expense Split Wizard | `hr.expense.split.wizard` | `hr_expense_split_wizard` | Interactive assistant with 1 stored column; belongs to Expense; owns Expense Split; referenced by 1 relation field. | Expenses |

#### Relationships (37)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Expense | `hr.expense` | `account_id` | link to one record | Account | `account.account` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `account_move_id` | link to one record | Journal Entry | `account.move` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Expense | `hr.expense` | `company_currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Expense | `hr.expense` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Expense | `hr.expense` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Expense | `hr.expense` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `duplicate_expense_ids` | list on both sides | Expense | `hr.expense` | `0..n : 0..n` | `not declared` | derived |
| Expense | `hr.expense` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| Expense | `hr.expense` | `journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | derived |
| Expense | `hr.expense` | `manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `payment_method_line_id` | link to one record | Payment Methods | `account.payment.method.line` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `restrict` | stored |
| Expense | `hr.expense` | `product_uom_id` | link to one record | Product Unit of Measure | `uom.uom` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `same_receipt_expense_ids` | list on both sides | Expense | `hr.expense` | `0..n : 0..n` | `not declared` | derived |
| Expense | `hr.expense` | `selectable_payment_method_line_ids` | list on both sides | Payment Methods | `account.payment.method.line` | `0..n : 0..n` | `not declared` | derived |
| Expense | `hr.expense` | `split_expense_origin_id` | link to one record | Expense | `hr.expense` | `n : 0..1` | `not declared` | stored |
| Expense | `hr.expense` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Expense | `hr.expense` | `vendor_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Expense Approve Duplicate | `hr.expense.approve.duplicate` | `expense_ids` | list on both sides | Expense | `hr.expense` | `0..n : 0..n` | `not declared` | stored |
| Expense Posting Wizard | `hr.expense.post.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Expense Posting Wizard | `hr.expense.post.wizard` | `employee_journal_id` | link to one record | Journal | `account.journal` | `n : 0..1` | `not declared` | stored |
| Expense Refuse Reason Wizard | `hr.expense.refuse.wizard` | `expense_ids` | list on both sides | Expense | `hr.expense` | `0..n : 0..n` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `expense_id` | link to one record | Expense | `hr.expense` | `n : 0..1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `tax_ids` | list on both sides | Tax | `account.tax` | `0..n : 0..n` | `not declared` | stored |
| Expense Split | `hr.expense.split` | `wizard_id` | link to one record | Expense Split Wizard | `hr.expense.split.wizard` | `n : 0..1` | `not declared` | stored |
| Expense Split Wizard | `hr.expense.split.wizard` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Expense Split Wizard | `hr.expense.split.wizard` | `expense_id` | link to one record | Expense | `hr.expense` | `n : 1` | `not declared` | stored |
| Expense Split Wizard | `hr.expense.split.wizard` | `expense_split_line_ids` | list of records | Expense Split | `hr.expense.split` | `1 : 0..n` | `mirror of the target column` | derived |

### 3.38 Fleet

Vehicles, models, brands and categories, contracts, services, odometer readings, assignment logs, states, tags and cost reporting.

Specified in the domain folder `../domains/fleet/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (13)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Brand of the vehicle | `fleet.vehicle.model.brand` | `fleet_vehicle_model_brand` | Persistent record with 3 stored columns; owns Model of a vehicle; referenced by 3 relation fields. | Fleet |
| Category of the model | `fleet.vehicle.model.category` | `fleet_vehicle_model_category` | Persistent record with 4 stored columns; referenced by 3 relation fields. | Fleet |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `fleet_vehicle_assignation_log` | Persistent record with 5 stored columns; belongs to Contact, Vehicle. | Fleet |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | `fleet_vehicle_cost_report` | Persistent record with 9 stored columns; company scoped. | Fleet |
| Fleet Odometer Analysis Report | `fleet.vehicle.odometer.report` | `fleet_vehicle_odometer_report` | Persistent record with 4 stored columns. | Fleet |
| Fleet Service Type | `fleet.service.type` | `fleet_service_type` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Fleet |
| Model of a vehicle | `fleet.vehicle.model` | `fleet_vehicle_model` | Persistent record with 23 stored columns; belongs to Brand of the vehicle; referenced by 2 relation fields. | Fleet |
| Odometer log for a vehicle | `fleet.vehicle.odometer` | `fleet_vehicle_odometer` | Persistent record with 5 stored columns; belongs to Vehicle; referenced by 1 relation field. | Fleet |
| Services for vehicles | `fleet.vehicle.log.services` | `fleet_vehicle_log_services` | Persistent record with 18 stored columns; belongs to Fleet Service Type, Vehicle; lifecycle states New, Running, Done, Cancelled; 2 state fields in all; company scoped. | Fleet |
| Vehicle | `fleet.vehicle` | `fleet_vehicle` | Persistent record with 48 stored columns; belongs to Model of a vehicle; owns Drivers history on a vehicle, Journal Entry, Services for vehicles and 1 further collections; states of `contract_state`: Incoming, In Progress, Expired, Closed; company scoped; referenced by 9 relation fields. | Fleet |
| Vehicle Contract | `fleet.vehicle.log.contract` | `fleet_vehicle_log_contract` | Persistent record with 16 stored columns; belongs to Vehicle; lifecycle states New, Running, Expired, Cancelled; company scoped. | Fleet |
| Vehicle Status | `fleet.vehicle.state` | `fleet_vehicle_state` | Persistent record with 3 stored columns; referenced by 1 relation field. | Fleet |
| Vehicle Tag | `fleet.vehicle.tag` | `fleet_vehicle_tag` | Persistent record with 2 stored columns; referenced by 1 relation field. | Fleet |

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Send mails to Drivers | `fleet.vehicle.send.mail` | `fleet_vehicle_send_mail` | Interactive assistant with 5 stored columns; belongs to Contact. | Fleet |

#### Relationships (52)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Brand of the vehicle | `fleet.vehicle.model.brand` | `model_ids` | list of records | Model of a vehicle | `fleet.vehicle.model` | `1 : 0..n` | `mirror of the target column` | derived |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `driver_employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `driver_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 1` | `not declared` | stored |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | `driver_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 0..1` | `not declared` | stored |
| Fleet Odometer Analysis Report | `fleet.vehicle.odometer.report` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 0..1` | `not declared` | stored |
| Model of a vehicle | `fleet.vehicle.model` | `brand_id` | link to one record | Brand of the vehicle | `fleet.vehicle.model.brand` | `n : 1` | `not declared` | stored |
| Model of a vehicle | `fleet.vehicle.model` | `category_id` | link to one record | Category of the model | `fleet.vehicle.model.category` | `n : 0..1` | `not declared` | stored |
| Model of a vehicle | `fleet.vehicle.model` | `vendors` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Odometer log for a vehicle | `fleet.vehicle.odometer` | `driver_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Odometer log for a vehicle | `fleet.vehicle.odometer` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 1` | `not declared` | stored |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `author_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Send mails to Drivers | `fleet.vehicle.send.mail` | `vehicle_ids` | list on both sides | Vehicle | `fleet.vehicle` | `0..n : 0..n` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `account_move_line_id` | link to one record | Journal Item | `account.move.line` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `brand_id` | link to one record | Brand of the vehicle | `fleet.vehicle.model.brand` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Services for vehicles | `fleet.vehicle.log.services` | `manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `model_id` | link to one record | Model of a vehicle | `fleet.vehicle.model` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `odometer_id` | link to one record | Odometer log for a vehicle | `fleet.vehicle.odometer` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `purchaser_employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `purchaser_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `service_type_id` | link to one record | Fleet Service Type | `fleet.service.type` | `n : 1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 1` | `not declared` | stored |
| Services for vehicles | `fleet.vehicle.log.services` | `vendor_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `account_move_ids` | list of records | Journal Entry | `account.move` | `1 : 0..n` | `mirror of the target column` | derived |
| Vehicle | `fleet.vehicle` | `brand_id` | link to one record | Brand of the vehicle | `fleet.vehicle.model.brand` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `category_id` | link to one record | Category of the model | `fleet.vehicle.model.category` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Vehicle | `fleet.vehicle` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Vehicle | `fleet.vehicle` | `driver_employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `driver_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `future_driver_employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `future_driver_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `log_contracts` | list of records | Vehicle Contract | `fleet.vehicle.log.contract` | `1 : 0..n` | `mirror of the target column` | derived |
| Vehicle | `fleet.vehicle` | `log_drivers` | list of records | Drivers history on a vehicle | `fleet.vehicle.assignation.log` | `1 : 0..n` | `mirror of the target column` | derived |
| Vehicle | `fleet.vehicle` | `log_services` | list of records | Services for vehicles | `fleet.vehicle.log.services` | `1 : 0..n` | `mirror of the target column` | derived |
| Vehicle | `fleet.vehicle` | `manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `model_id` | link to one record | Model of a vehicle | `fleet.vehicle.model` | `n : 1` | `not declared` | stored |
| Vehicle | `fleet.vehicle` | `state_id` | link to one record | Vehicle Status | `fleet.vehicle.state` | `n : 0..1` | `set null` | stored |
| Vehicle | `fleet.vehicle` | `tag_ids` | list on both sides | Vehicle Tag | `fleet.vehicle.tag` | `0..n : 0..n` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `cost_subtype_id` | link to one record | Fleet Service Type | `fleet.service.type` | `n : 0..1` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Vehicle Contract | `fleet.vehicle.log.contract` | `insurer_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `service_ids` | list on both sides | Fleet Service Type | `fleet.service.type` | `0..n : 0..n` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Vehicle Contract | `fleet.vehicle.log.contract` | `vehicle_id` | link to one record | Vehicle | `fleet.vehicle` | `n : 1` | `not declared` | stored |

### 3.39 Human Resources Core

Employees and employee versions, departments, job positions, work locations, skills and resumes, the organization chart, presence, remote work, departure reasons and hourly cost.

Specified in [`../domains/human-resources-core/`](../domains/human-resources-core/).

#### Persistent entities (22)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Contract Type | `hr.contract.type` | `hr_contract_type` | Persistent record with 4 stored columns; referenced by 2 relation fields. | Employees |
| Department | `hr.department` | `hr_department` | Persistent record with 9 stored columns; owns Activity Plan, Department, Employee and 1 further collections; company scoped; referenced by 26 relation fields. | Employees |
| Departure Reason | `hr.departure.reason` | `hr_departure_reason` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Employees |
| Employee | `hr.employee` | `hr_employee` | Persistent record with 58 stored columns; belongs to Companies, Resources, Version; owns Applicant, Attendance, Attendance Overtime Line and 8 further collections; states of `hr_presence_state`: Present, Absent, Archived, Off-Hours; 4 state fields in all; company scoped; referenced by 67 relation fields. | Employees |
| Employee Category | `hr.employee.category` | `hr_employee_category` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Employees |
| Employee Certification Report | `hr.employee.certification.report` | `hr_employee_certification_report` | Persistent record with 8 stored columns; company scoped. | Skills Management |
| Employee Location | `hr.employee.location` | `hr_employee_location` | Persistent record with 3 stored columns; belongs to Employee, Work Location. | Remote Work |
| Employee Skills Report | `hr.employee.skill.history.report` | `hr_employee_skill_history_report` | Persistent record with 5 stored columns. | Skills Management |
| Employee Skills Report | `hr.employee.skill.report` | `hr_employee_skill_report` | Persistent record with 8 stored columns; company scoped. | Skills Management |
| Job Position | `hr.job` | `hr_job` | Persistent record with 31 stored columns; owns Applicant, Attachment, Employee and 2 further collections; company scoped; referenced by 10 relation fields. | Employees |
| Public Employee | `hr.employee.public` | `hr_employee_public` | Persistent record with 34 stored columns; owns Gamification User Badge, Public Employee, Resume line of an employee and 1 further collections; states of `hr_presence_state`: Present, Absent, Archived, Off-Hours; 2 state fields in all; company scoped; referenced by 2 relation fields. | Employees |
| Resume line of an employee | `hr.resume.line` | `hr_resume_line` | Persistent record with 16 stored columns; belongs to Employee; states of `expiration_status`: Expired, Expiring, Valid. | Skills Management |
| Salary Structure Type | `hr.payroll.structure.type` | `hr_payroll_structure_type` | Persistent record with 3 stored columns; referenced by 1 relation field. | Employees |
| Skill | `hr.skill` | `hr_skill` | Persistent record with 3 stored columns; belongs to Skill Type; referenced by 9 relation fields. | Skills Management |
| Skill Level | `hr.skill.level` | `hr_skill_level` | Persistent record with 4 stored columns; referenced by 1 relation field. | Skills Management |
| Skill level for an applicant | `hr.applicant.skill` | `hr_applicant_skill` | Persistent record with 7 stored columns; belongs to Applicant. | Recruitment - Skills Management |
| Skill level for employee | `hr.employee.skill` | `hr_employee_skill` | Persistent record with 7 stored columns; belongs to Employee. | Skills Management |
| Skill Type | `hr.skill.type` | `hr_skill_type` | Persistent record with 6 stored columns; owns Skill, Skill Level; referenced by 6 relation fields. | Skills Management |
| Skills for job positions | `hr.job.skill` | `hr_job_skill` | Persistent record with 7 stored columns; belongs to Job Position. | Skills Management |
| Type of a resume line | `hr.resume.line.type` | `hr_resume_line_type` | Persistent record with 4 stored columns; referenced by 1 relation field. | Skills Management |
| Version | `hr.version` | `hr_version` | Persistent record with 53 stored columns; belongs to User; company scoped; referenced by 5 relation fields. | Employees |
| Work Location | `hr.work.location` | `hr_work_location` | Persistent record with 6 stored columns; belongs to Companies, Contact; company scoped; referenced by 26 relation fields. | Employees |

#### Interactive assistant entities (6)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Bank Account Allocation Line (Wizard) | `hr.bank.account.allocation.wizard.line` | `hr_bank_account_allocation_wizard_line` | Interactive assistant with 6 stored columns; belongs to Bank Account Allocation Wizard, Bank Accounts. | Employees |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `hr_bank_account_allocation_wizard` | Interactive assistant with 1 stored column; belongs to Employee; owns Bank Account Allocation Line (Wizard); referenced by 1 relation field. | Employees |
| Contract Template Wizard | `hr.version.wizard` | `hr_version_wizard` | Interactive assistant with 1 stored column; belongs to Version. | Employees |
| Departure Wizard | `hr.departure.wizard` | `hr_departure_wizard` | Interactive assistant with 7 stored columns; belongs to Departure Reason. | Employees |
| Print Resume | `hr.employee.cv.wizard` | `hr_employee_cv_wizard` | Interactive assistant with 5 stored columns. | Skills Management |
| Set Homework Location Wizard | `homework.location.wizard` | `homework_location_wizard` | Interactive assistant with 4 stored columns; belongs to Employee, Work Location. | Remote Work with calendar |

#### Shared behaviour entities (2)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Human Resources Manager Department Report | `hr.manager.department.report` | `none` | Shared behaviour merged into 4 entities. | Employees |
| Skill level | `hr.individual.skill.mixin` | `none` | Shared behaviour merged into 3 entities. | Skills Management |

#### Relationships (165)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Bank Account Allocation Line (Wizard) | `hr.bank.account.allocation.wizard.line` | `bank_account_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 1` | `not declared` | stored |
| Bank Account Allocation Line (Wizard) | `hr.bank.account.allocation.wizard.line` | `wizard_id` | link to one record | Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `n : 1` | `cascade` | stored |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `allocation_ids` | list of records | Bank Account Allocation Line (Wizard) | `hr.bank.account.allocation.wizard.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Bank Account Allocation Wizard | `hr.bank.account.allocation.wizard` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| Contract Template Wizard | `hr.version.wizard` | `contract_template_id` | link to one record | Version | `hr.version` | `n : 1` | `not declared` | stored |
| Contract Type | `hr.contract.type` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Department | `hr.department` | `child_ids` | list of records | Department | `hr.department` | `1 : 0..n` | `mirror of the target column` | derived |
| Department | `hr.department` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Department | `hr.department` | `jobs_ids` | list of records | Job Position | `hr.job` | `1 : 0..n` | `mirror of the target column` | derived |
| Department | `hr.department` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Department | `hr.department` | `master_department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Department | `hr.department` | `member_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Department | `hr.department` | `parent_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Department | `hr.department` | `plan_ids` | list of records | Activity Plan | `mail.activity.plan` | `1 : 0..n` | `mirror of the target column` | derived |
| Departure Reason | `hr.departure.reason` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Departure Wizard | `hr.departure.wizard` | `departure_reason_id` | link to one record | Departure Reason | `hr.departure.reason` | `n : 1` | `not declared` | stored |
| Departure Wizard | `hr.departure.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Employee | `hr.employee` | `applicant_ids` | list of records | Applicant | `hr.applicant` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `attendance_ids` | list of records | Attendance | `hr.attendance` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `attendance_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `badge_ids` | list of records | Gamification User Badge | `gamification.badge.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `bank_account_ids` | list on both sides | Bank Accounts | `res.partner.bank` | `0..n : 0..n` | `not declared` | stored |
| Employee | `hr.employee` | `car_ids` | list of records | Vehicle | `fleet.vehicle` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `category_ids` | list on both sides | Employee Category | `hr.employee.category` | `0..n : 0..n` | `not declared` | stored |
| Employee | `hr.employee` | `certification_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `child_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `coach_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `company_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Employee | `hr.employee` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Employee | `hr.employee` | `country_of_birth` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Employee | `hr.employee` | `current_employee_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `current_leave_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | derived |
| Employee | `hr.employee` | `current_version_id` | link to one record | Version | `hr.version` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `direct_badge_ids` | list of records | Gamification User Badge | `gamification.badge.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `employee_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `equipment_ids` | list of records | Maintenance Equipment | `maintenance.equipment` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `exceptional_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | derived |
| Employee | `hr.employee` | `expense_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `friday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `goal_ids` | list of records | Gamification Goal | `gamification.goal` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `last_attendance_id` | link to one record | Attendance | `hr.attendance` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `leave_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `monday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `overtime_ids` | list of records | Attendance Overtime Line | `hr.attendance.overtime.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `parent_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `primary_bank_account_id` | link to one record | Bank Accounts | `res.partner.bank` | `n : 0..1` | `not declared` | derived |
| Employee | `hr.employee` | `resource_id` | link to one record | Resources | `resource.resource` | `n : 1` | `not declared` | stored |
| Employee | `hr.employee` | `resume_line_ids` | list of records | Resume line of an employee | `hr.resume.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `saturday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `skill_ids` | list on both sides | Skill | `hr.skill` | `0..n : 0..n` | `not declared` | stored |
| Employee | `hr.employee` | `subordinate_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `subscribed_courses` | list on both sides | Course | `slide.channel` | `0..n : 0..n` | `not declared` | derived |
| Employee | `hr.employee` | `sunday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `thursday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `tuesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `restrict` | stored |
| Employee | `hr.employee` | `version_id` | link to one record | Version | `hr.version` | `n : 1` | `cascade` | derived |
| Employee | `hr.employee` | `version_ids` | list of records | Version | `hr.version` | `1 : 0..n` | `mirror of the target column` | derived |
| Employee | `hr.employee` | `wednesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Employee | `hr.employee` | `work_contact_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Employee Category | `hr.employee.category` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Employee Certification Report | `hr.employee.certification.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Employee Certification Report | `hr.employee.certification.report` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Employee Certification Report | `hr.employee.certification.report` | `skill_id` | link to one record | Skill | `hr.skill` | `n : 0..1` | `not declared` | stored |
| Employee Certification Report | `hr.employee.certification.report` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 0..1` | `not declared` | stored |
| Employee Location | `hr.employee.location` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Employee Location | `hr.employee.location` | `work_location_id` | link to one record | Work Location | `hr.work.location` | `n : 1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.history.report` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.history.report` | `skill_id` | link to one record | Skill | `hr.skill` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.history.report` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.report` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.report` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.report` | `skill_id` | link to one record | Skill | `hr.skill` | `n : 0..1` | `not declared` | stored |
| Employee Skills Report | `hr.employee.skill.report` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 0..1` | `not declared` | stored |
| Human Resources Manager Department Report | `hr.manager.department.report` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | derived |
| Job Position | `hr.job` | `address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `allowed_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Job Position | `hr.job` | `application_ids` | list of records | Applicant | `hr.applicant` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `contract_type_id` | link to one record | Contract Type | `hr.contract.type` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `current_job_skill_ids` | list of records | Skills for job positions | `hr.job.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `document_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `employee_ids` | list of records | Employee | `hr.employee` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `expected_degree` | link to one record | Applicant Degree | `hr.recruitment.degree` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `extended_interviewer_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Job Position | `hr.job` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Job Position | `hr.job` | `industry_id` | link to one record | Industry | `res.partner.industry` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `interviewer_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Job Position | `hr.job` | `job_skill_ids` | list of records | Skills for job positions | `hr.job.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `job_source_ids` | list of records | Source of Applicants | `hr.recruitment.source` | `1 : 0..n` | `mirror of the target column` | derived |
| Job Position | `hr.job` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `skill_ids` | list on both sides | Skill | `hr.skill` | `0..n : 0..n` | `not declared` | stored |
| Job Position | `hr.job` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `not declared` | stored |
| Job Position | `hr.job` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Print Resume | `hr.employee.cv.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `badge_ids` | list of records | Gamification User Badge | `gamification.badge.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `certification_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `child_ids` | list of records | Public Employee | `hr.employee.public` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `coach_id` | link to one record | Public Employee | `hr.employee.public` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `current_employee_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `employee_skill_ids` | list of records | Skill level for employee | `hr.employee.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `expense_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `friday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `leave_manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `monday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `parent_id` | link to one record | Public Employee | `hr.employee.public` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `resource_id` | link to one record | Resources | `resource.resource` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `resume_line_ids` | list of records | Resume line of an employee | `hr.resume.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Public Employee | `hr.employee.public` | `saturday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `sunday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `thursday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `tuesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `wednesday_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `work_contact_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Public Employee | `hr.employee.public` | `work_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Resume line of an employee | `hr.resume.line` | `channel_id` | link to one record | Course | `slide.channel` | `n : 0..1` | `not declared` | stored |
| Resume line of an employee | `hr.resume.line` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Resume line of an employee | `hr.resume.line` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Resume line of an employee | `hr.resume.line` | `line_type_id` | link to one record | Type of a resume line | `hr.resume.line.type` | `n : 0..1` | `not declared` | stored |
| Resume line of an employee | `hr.resume.line` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `not declared` | stored |
| Salary Structure Type | `hr.payroll.structure.type` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Salary Structure Type | `hr.payroll.structure.type` | `default_resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Set Homework Location Wizard | `homework.location.wizard` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Set Homework Location Wizard | `homework.location.wizard` | `work_location_id` | link to one record | Work Location | `hr.work.location` | `n : 1` | `not declared` | stored |
| Skill | `hr.skill` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 1` | `cascade` | stored |
| Skill Level | `hr.skill.level` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 0..1` | `cascade` | stored |
| Skill Type | `hr.skill.type` | `skill_ids` | list of records | Skill | `hr.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Skill Type | `hr.skill.type` | `skill_level_ids` | list of records | Skill Level | `hr.skill.level` | `1 : 0..n` | `mirror of the target column` | derived |
| Skill level | `hr.individual.skill.mixin` | `skill_id` | link to one record | Skill | `hr.skill` | `n : 1` | `cascade` | derived |
| Skill level | `hr.individual.skill.mixin` | `skill_level_id` | link to one record | Skill Level | `hr.skill.level` | `n : 1` | `cascade` | derived |
| Skill level | `hr.individual.skill.mixin` | `skill_type_id` | link to one record | Skill Type | `hr.skill.type` | `n : 1` | `cascade` | derived |
| Skill level for an applicant | `hr.applicant.skill` | `applicant_id` | link to one record | Applicant | `hr.applicant` | `n : 1` | `cascade` | stored |
| Skill level for employee | `hr.employee.skill` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `cascade` | stored |
| Skills for job positions | `hr.job.skill` | `job_id` | link to one record | Job Position | `hr.job` | `n : 1` | `cascade` | stored |
| Version | `hr.version` | `address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `allowed_country_state_ids` | list on both sides | Country state | `res.country.state` | `0..n : 0..n` | `not declared` | derived |
| Version | `hr.version` | `company_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Version | `hr.version` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `contract_template_id` | link to one record | Version | `hr.version` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `contract_type_id` | link to one record | Contract Type | `hr.contract.type` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `departure_reason_id` | link to one record | Departure Reason | `hr.departure.reason` | `n : 0..1` | `restrict` | stored |
| Version | `hr.version` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `hr_responsible_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Version | `hr.version` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `last_modified_uid` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Version | `hr.version` | `private_country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `private_state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `ruleset_id` | link to one record | Overtime Ruleset | `hr.attendance.overtime.ruleset` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `structure_type_id` | link to one record | Salary Structure Type | `hr.payroll.structure.type` | `n : 0..1` | `not declared` | stored |
| Version | `hr.version` | `work_location_id` | link to one record | Work Location | `hr.work.location` | `n : 0..1` | `not declared` | stored |
| Work Location | `hr.work.location` | `address_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Work Location | `hr.work.location` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |

### 3.40 Lunch Ordering

Meal suppliers, products and categories, locations, orders, cash movements and alerts.

Specified in the domain folder `../domains/lunch-ordering/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (9)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Cashmoves report | `lunch.cashmove.report` | `lunch_cashmove_report` | Persistent record with 5 stored columns. | Lunch |
| Lunch Alert | `lunch.alert` | `lunch_alert` | Persistent record with 17 stored columns; belongs to Scheduled Actions. | Lunch |
| Lunch Cashmove | `lunch.cashmove` | `lunch_cashmove` | Persistent record with 5 stored columns; belongs to Currency. | Lunch |
| Lunch Extras | `lunch.topping` | `lunch_topping` | Persistent record with 5 stored columns; company scoped; referenced by 3 relation fields. | Lunch |
| Lunch Locations | `lunch.location` | `lunch_location` | Persistent record with 3 stored columns; company scoped; referenced by 5 relation fields. | Lunch |
| Lunch Order | `lunch.order` | `lunch_order` | Persistent record with 15 stored columns; belongs to Lunch Product; lifecycle states To Order, Ordered, Sent, Received, Cancelled; company scoped. | Lunch |
| Lunch Product | `lunch.product` | `lunch_product` | Persistent record with 8 stored columns; belongs to Lunch Product Category, Lunch Supplier; company scoped; referenced by 2 relation fields. | Lunch |
| Lunch Product Category | `lunch.product.category` | `lunch_product_category` | Persistent record with 3 stored columns; company scoped; referenced by 1 relation field. | Lunch |
| Lunch Supplier | `lunch.supplier` | `lunch_supplier` | Persistent record with 24 stored columns; belongs to Contact, Scheduled Actions; owns Lunch Extras; company scoped; referenced by 2 relation fields. | Lunch |

#### Relationships (35)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Cashmoves report | `lunch.cashmove.report` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | stored |
| Cashmoves report | `lunch.cashmove.report` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Lunch Alert | `lunch.alert` | `cron_id` | link to one record | Scheduled Actions | `ir.cron` | `n : 1` | `cascade` | stored |
| Lunch Alert | `lunch.alert` | `location_ids` | list on both sides | Lunch Locations | `lunch.location` | `0..n : 0..n` | `not declared` | stored |
| Lunch Cashmove | `lunch.cashmove` | `currency_id` | link to one record | Currency | `res.currency` | `n : 1` | `not declared` | stored |
| Lunch Cashmove | `lunch.cashmove` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Lunch Extras | `lunch.topping` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Extras | `lunch.topping` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Lunch Extras | `lunch.topping` | `supplier_id` | link to one record | Lunch Supplier | `lunch.supplier` | `n : 0..1` | `cascade` | stored |
| Lunch Locations | `lunch.location` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Order | `lunch.order` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Order | `lunch.order` | `lunch_location_id` | link to one record | Lunch Locations | `lunch.location` | `n : 0..1` | `not declared` | stored |
| Lunch Order | `lunch.order` | `product_id` | link to one record | Lunch Product | `lunch.product` | `n : 1` | `not declared` | stored |
| Lunch Order | `lunch.order` | `topping_ids_1` | list on both sides | Lunch Extras | `lunch.topping` | `0..n : 0..n` | `not declared` | stored |
| Lunch Order | `lunch.order` | `topping_ids_2` | list on both sides | Lunch Extras | `lunch.topping` | `0..n : 0..n` | `not declared` | stored |
| Lunch Order | `lunch.order` | `topping_ids_3` | list on both sides | Lunch Extras | `lunch.topping` | `0..n : 0..n` | `not declared` | stored |
| Lunch Order | `lunch.order` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Lunch Product | `lunch.product` | `category_id` | link to one record | Lunch Product Category | `lunch.product.category` | `n : 1` | `not declared` | stored |
| Lunch Product | `lunch.product` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Product | `lunch.product` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Lunch Product | `lunch.product` | `favorite_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Lunch Product | `lunch.product` | `is_available_at` | link to one record | Lunch Locations | `lunch.location` | `n : 0..1` | `not declared` | derived |
| Lunch Product | `lunch.product` | `supplier_id` | link to one record | Lunch Supplier | `lunch.supplier` | `n : 1` | `not declared` | stored |
| Lunch Product Category | `lunch.product.category` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Product Category | `lunch.product.category` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Lunch Supplier | `lunch.supplier` | `available_location_ids` | list on both sides | Lunch Locations | `lunch.location` | `0..n : 0..n` | `not declared` | stored |
| Lunch Supplier | `lunch.supplier` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Lunch Supplier | `lunch.supplier` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Lunch Supplier | `lunch.supplier` | `cron_id` | link to one record | Scheduled Actions | `ir.cron` | `n : 1` | `cascade` | stored |
| Lunch Supplier | `lunch.supplier` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Lunch Supplier | `lunch.supplier` | `responsible_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Lunch Supplier | `lunch.supplier` | `state_id` | link to one record | Country state | `res.country.state` | `n : 0..1` | `not declared` | derived |
| Lunch Supplier | `lunch.supplier` | `topping_ids_1` | list of records | Lunch Extras | `lunch.topping` | `1 : 0..n` | `mirror of the target column` | derived |
| Lunch Supplier | `lunch.supplier` | `topping_ids_2` | list of records | Lunch Extras | `lunch.topping` | `1 : 0..n` | `mirror of the target column` | derived |
| Lunch Supplier | `lunch.supplier` | `topping_ids_3` | list of records | Lunch Extras | `lunch.topping` | `1 : 0..n` | `mirror of the target column` | derived |

### 3.41 Recruitment

Job openings, candidates, applicants, recruitment stages, sources, degrees, refuse reasons, interviews and the job board.

Specified in [`../domains/recruitment/`](../domains/recruitment/).

#### Persistent entities (8)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Applicant | `hr.applicant` | `hr_applicant` | Persistent record with 42 stored columns; owns Attachment, Calendar Event, Skill level for an applicant and 1 further collections; states of `kanban_state`: In Progress, Ready for Next Stage, Waiting, Blocked; 2 state fields in all; company scoped; referenced by 11 relation fields. | Recruitment |
| Applicant Degree | `hr.recruitment.degree` | `hr_recruitment_degree` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Recruitment |
| Category of applicant | `hr.applicant.category` | `hr_applicant_category` | Persistent record with 2 stored columns; referenced by 3 relation fields. | Recruitment |
| Job Platforms | `hr.job.platform` | `hr_job_platform` | Persistent record with 3 stored columns. | Recruitment |
| Recruitment Stages | `hr.recruitment.stage` | `hr_recruitment_stage` | Persistent record with 11 stored columns; referenced by 2 relation fields. | Recruitment |
| Refuse Reason of Applicant | `hr.applicant.refuse.reason` | `hr_applicant_refuse_reason` | Persistent record with 4 stored columns; referenced by 2 relation fields. | Recruitment |
| Source of Applicants | `hr.recruitment.source` | `hr_recruitment_source` | Persistent record with 5 stored columns. | Recruitment |
| Talent Pool | `hr.talent.pool` | `hr_talent_pool` | Persistent record with 6 stored columns; company scoped; referenced by 2 relation fields. | Recruitment |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Add applicants to a job | `job.add.applicants` | `job_add_applicants` | Interactive assistant with 0 stored columns. | Recruitment |
| Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_add_applicants` | Interactive assistant with 0 stored columns. | Recruitment |
| Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_get_refuse_reason` | Interactive assistant with 8 stored columns; belongs to Refuse Reason of Applicant. | Recruitment |
| Send mails to applicants | `applicant.send.mail` | `applicant_send_mail` | Interactive assistant with 5 stored columns; belongs to Contact. | Recruitment |

#### Relationships (47)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Add applicants to a job | `job.add.applicants` | `applicant_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |
| Add applicants to a job | `job.add.applicants` | `job_ids` | list on both sides | Job Position | `hr.job` | `0..n : 0..n` | `not declared` | stored |
| Add applicants to talent pool | `talent.pool.add.applicants` | `applicant_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |
| Add applicants to talent pool | `talent.pool.add.applicants` | `categ_ids` | list on both sides | Category of applicant | `hr.applicant.category` | `0..n : 0..n` | `not declared` | stored |
| Add applicants to talent pool | `talent.pool.add.applicants` | `talent_pool_ids` | list on both sides | Talent Pool | `hr.talent.pool` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `applicant_skill_ids` | list of records | Skill level for an applicant | `hr.applicant.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Applicant | `hr.applicant` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Applicant | `hr.applicant` | `categ_ids` | list on both sides | Category of applicant | `hr.applicant.category` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `current_applicant_skill_ids` | list of records | Skill level for an applicant | `hr.applicant.skill` | `1 : 0..n` | `mirror of the target column` | derived |
| Applicant | `hr.applicant` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `interviewer_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `last_stage_id` | link to one record | Recruitment Stages | `hr.recruitment.stage` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `matching_skill_ids` | list on both sides | Skill | `hr.skill` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `meeting_ids` | list of records | Calendar Event | `calendar.event` | `1 : 0..n` | `mirror of the target column` | derived |
| Applicant | `hr.applicant` | `missing_skill_ids` | list on both sides | Skill | `hr.skill` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `pool_applicant_id` | link to one record | Applicant | `hr.applicant` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `refuse_reason_id` | link to one record | Refuse Reason of Applicant | `hr.applicant.refuse.reason` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `response_ids` | list of records | Survey User Input | `survey.user_input` | `1 : 0..n` | `mirror of the target column` | derived |
| Applicant | `hr.applicant` | `skill_ids` | list on both sides | Skill | `hr.skill` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `stage_id` | link to one record | Recruitment Stages | `hr.recruitment.stage` | `n : 0..1` | `restrict` | stored |
| Applicant | `hr.applicant` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `not declared` | derived |
| Applicant | `hr.applicant` | `talent_pool_ids` | list on both sides | Talent Pool | `hr.talent.pool` | `0..n : 0..n` | `not declared` | stored |
| Applicant | `hr.applicant` | `type_id` | link to one record | Applicant Degree | `hr.recruitment.degree` | `n : 0..1` | `not declared` | stored |
| Applicant | `hr.applicant` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Get Refuse Reason | `applicant.get.refuse.reason` | `applicant_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |
| Get Refuse Reason | `applicant.get.refuse.reason` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Get Refuse Reason | `applicant.get.refuse.reason` | `duplicate_applicant_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |
| Get Refuse Reason | `applicant.get.refuse.reason` | `refuse_reason_id` | link to one record | Refuse Reason of Applicant | `hr.applicant.refuse.reason` | `n : 1` | `not declared` | stored |
| Get Refuse Reason | `applicant.get.refuse.reason` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Recruitment Stages | `hr.recruitment.stage` | `job_ids` | list on both sides | Job Position | `hr.job` | `0..n : 0..n` | `not declared` | stored |
| Recruitment Stages | `hr.recruitment.stage` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Refuse Reason of Applicant | `hr.applicant.refuse.reason` | `template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Send mails to applicants | `applicant.send.mail` | `applicant_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |
| Send mails to applicants | `applicant.send.mail` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Send mails to applicants | `applicant.send.mail` | `author_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Source of Applicants | `hr.recruitment.source` | `alias_id` | link to one record | Email Aliases | `mail.alias` | `n : 0..1` | `restrict` | stored |
| Source of Applicants | `hr.recruitment.source` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `not declared` | stored |
| Source of Applicants | `hr.recruitment.source` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `cascade` | stored |
| Source of Applicants | `hr.recruitment.source` | `medium_id` | link to one record | campaign tracking parameter Medium | `utm.medium` | `n : 0..1` | `not declared` | stored |
| Talent Pool | `hr.talent.pool` | `categ_ids` | list on both sides | Category of applicant | `hr.applicant.category` | `0..n : 0..n` | `not declared` | stored |
| Talent Pool | `hr.talent.pool` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Talent Pool | `hr.talent.pool` | `pool_manager` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Talent Pool | `hr.talent.pool` | `talent_ids` | list on both sides | Applicant | `hr.applicant` | `0..n : 0..n` | `not declared` | stored |

### 3.42 Time Off

Time off types, requests, allocations, accrual plans and levels, approval flows, public holidays and mandatory days.

Specified in [`../domains/time-off/`](../domains/time-off/).

#### Persistent entities (11)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Accrual Plan | `hr.leave.accrual.plan` | `hr_leave_accrual_plan` | Persistent record with 12 stored columns; owns Accrual Plan Level, Time Off Allocation; company scoped; referenced by 3 relation fields. | Time Off |
| Accrual Plan Level | `hr.leave.accrual.level` | `hr_leave_accrual_level` | Persistent record with 27 stored columns; belongs to Accrual Plan. | Time Off |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `hr_leave_attendance_report` | Persistent record with 7 stored columns. | human resources Attendance Holidays |
| Mandatory Day | `hr.leave.mandatory.day` | `hr_leave_mandatory_day` | Persistent record with 6 stored columns; belongs to Companies; company scoped. | Time Off |
| Optional Holidays | `l10n.in.hr.leave.optional.holiday` | `l10n_in_hr_leave_optional_holiday` | Persistent record with 3 stored columns; belongs to Companies; company scoped. | India - Time Off |
| Time Off | `hr.leave` | `hr_leave` | Persistent record with 29 stored columns; belongs to Employee, Time Off Type; owns Analytic Line, Attachment; lifecycle states To Approve, Refused, Second Approval, Approved, Cancelled; company scoped; referenced by 7 relation fields. | Time Off |
| Time Off Allocation | `hr.leave.allocation` | `hr_leave_allocation` | Persistent record with 24 stored columns; belongs to Employee, Time Off Type; lifecycle states To Approve, Refused, Second Approval, Approved; referenced by 1 relation field. | Time Off |
| Time Off Calendar | `hr.leave.report.calendar` | `hr_leave_report_calendar` | Persistent record with 15 stored columns; lifecycle states Cancelled, To Approve, Refused, Second Approval, Approved; company scoped. | Time Off |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `hr_leave_employee_type_report` | Persistent record with 11 stored columns; lifecycle states Cancelled, To Approve, Refused, Second Approval, Approved; 2 state fields in all; company scoped. | Time Off |
| Time Off Summary / Report | `hr.leave.report` | `hr_leave_report` | Persistent record with 13 stored columns; lifecycle states Cancelled, To Approve, Refused, Second Approval, Approved; company scoped. | Time Off |
| Time Off Type | `hr.leave.type` | `hr_leave_type` | Persistent record with 28 stored columns; owns Accrual Plan; company scoped; referenced by 11 relation fields. | Time Off |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | `hr_holidays_cancel_leave` | Interactive assistant with 2 stored columns; belongs to Time Off. | Time Off |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `hr_leave_allocation_generate_multi_wizard` | Interactive assistant with 12 stored columns; belongs to Companies, Time Off Type; company scoped. | Time Off |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `hr_leave_generate_multi_wizard` | Interactive assistant with 8 stored columns; belongs to Companies, Time Off Type; company scoped. | Time Off |
| human resources Time Off Summary Report By Employee | `hr.holidays.summary.employee` | `hr_holidays_summary_employee` | Interactive assistant with 2 stored columns. | Time Off |

#### Relationships (70)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Accrual Plan | `hr.leave.accrual.plan` | `allocation_ids` | list of records | Time Off Allocation | `hr.leave.allocation` | `1 : 0..n` | `mirror of the target column` | derived |
| Accrual Plan | `hr.leave.accrual.plan` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Accrual Plan | `hr.leave.accrual.plan` | `level_ids` | list of records | Accrual Plan Level | `hr.leave.accrual.level` | `1 : 0..n` | `mirror of the target column` | derived |
| Accrual Plan | `hr.leave.accrual.plan` | `time_off_type_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | stored |
| Accrual Plan Level | `hr.leave.accrual.level` | `accrual_plan_id` | link to one record | Accrual Plan | `hr.leave.accrual.plan` | `n : 1` | `cascade` | stored |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `attendance_ids` | list on both sides | Attendance | `hr.attendance` | `0..n : 0..n` | `not declared` | derived |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `leave_ids` | list on both sides | Time Off | `hr.leave` | `0..n : 0..n` | `not declared` | derived |
| Attendance and Leave Analysis Report | `hr.leave.attendance.report` | `schedule_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Cancel Time Off Wizard | `hr.holidays.cancel.leave` | `leave_id` | link to one record | Time Off | `hr.leave` | `n : 1` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `accrual_plan_id` | link to one record | Accrual Plan | `hr.leave.accrual.plan` | `n : 0..1` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `category_id` | link to one record | Employee Category | `hr.employee.category` | `n : 0..1` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Generate time off allocations for multiple employees | `hr.leave.allocation.generate.multi.wizard` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 1` | `not declared` | stored |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `category_id` | link to one record | Employee Category | `hr.employee.category` | `n : 0..1` | `not declared` | stored |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Generate time off for multiple employees | `hr.leave.generate.multi.wizard` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 1` | `not declared` | stored |
| Mandatory Day | `hr.leave.mandatory.day` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Mandatory Day | `hr.leave.mandatory.day` | `department_ids` | list on both sides | Department | `hr.department` | `0..n : 0..n` | `not declared` | stored |
| Mandatory Day | `hr.leave.mandatory.day` | `job_ids` | list on both sides | Job Position | `hr.job` | `0..n : 0..n` | `not declared` | stored |
| Mandatory Day | `hr.leave.mandatory.day` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Optional Holidays | `l10n.in.hr.leave.optional.holiday` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| Time Off | `hr.leave` | `attachment_ids` | list of records | Attachment | `ir.attachment` | `1 : 0..n` | `mirror of the target column` | derived |
| Time Off | `hr.leave` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `restrict` | stored |
| Time Off | `hr.leave` | `first_approver_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 1` | `not declared` | stored |
| Time Off | `hr.leave` | `meeting_id` | link to one record | Calendar Event | `calendar.event` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `resource_calendar_id` | link to one record | Resource Working Time | `resource.calendar` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `second_approver_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off | `hr.leave` | `supported_attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | derived |
| Time Off | `hr.leave` | `timesheet_ids` | list of records | Analytic Line | `account.analytic.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Time Off | `hr.leave` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `accrual_plan_id` | link to one record | Accrual Plan | `hr.leave.accrual.plan` | `n : 0..1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `approver_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `restrict` | stored |
| Time Off Allocation | `hr.leave.allocation` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `manager_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off Allocation | `hr.leave.allocation` | `second_approver_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `job_id` | link to one record | Job Position | `hr.job` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `leave_id` | link to one record | Time Off | `hr.leave` | `n : 0..1` | `not declared` | stored |
| Time Off Calendar | `hr.leave.report.calendar` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.employee.type.report` | `leave_type` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.report` | `allocation_id` | link to one record | Time Off Allocation | `hr.leave.allocation` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.report` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.report` | `holiday_status_id` | link to one record | Time Off Type | `hr.leave.type` | `n : 0..1` | `not declared` | stored |
| Time Off Summary / Report | `hr.leave.report` | `leave_id` | link to one record | Time Off | `hr.leave` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `accruals_ids` | list of records | Accrual Plan | `hr.leave.accrual.plan` | `1 : 0..n` | `mirror of the target column` | derived |
| Time Off Type | `hr.leave.type` | `allocation_notif_subtype_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `icon_id` | link to one record | Attachment | `ir.attachment` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `leave_notif_subtype_id` | link to one record | Message subtypes | `mail.message.subtype` | `n : 0..1` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `responsible_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Time Off Type | `hr.leave.type` | `work_entry_type_id` | link to one record | human resources Work Entry Type | `hr.work.entry.type` | `n : 0..1` | `not declared` | stored |
| human resources Time Off Summary Report By Employee | `hr.holidays.summary.employee` | `emp` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |

### 3.43 Work Entries

Work entry types, generated work entries and their conflicts.

Specified in [`../domains/work-entries/`](../domains/work-entries/).

#### Persistent entities (3)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| human resources Work Entry | `hr.work.entry` | `hr_work_entry` | Persistent record with 13 stored columns; belongs to Companies, Employee, Version; lifecycle states New, In Conflict, In Payslip, Cancelled; 2 state fields in all; company scoped. | Work Entries |
| human resources Work Entry Type | `hr.work.entry.type` | `hr_work_entry_type` | Persistent record with 11 stored columns; owns Time Off Type; referenced by 4 relation fields. | Work Entries |
| Work Entries Employees | `hr.user.work.entry.employee` | `hr_user_work_entry_employee` | Persistent record with 4 stored columns; belongs to Employee, User. | Work Entries |

#### Interactive assistant entities (1)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Regenerate Employee Work Entries | `hr.work.entry.regeneration.wizard` | `hr_work_entry_regeneration_wizard` | Interactive assistant with 2 stored columns. | Work Entries |

#### Relationships (13)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Regenerate Employee Work Entries | `hr.work.entry.regeneration.wizard` | `employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Regenerate Employee Work Entries | `hr.work.entry.regeneration.wizard` | `validated_work_entry_employee_ids` | list on both sides | Employee | `hr.employee` | `0..n : 0..n` | `not declared` | stored |
| Work Entries Employees | `hr.user.work.entry.employee` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| Work Entries Employees | `hr.user.work.entry.employee` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| human resources Work Entry | `hr.work.entry` | `company_id` | link to one record | Companies | `res.company` | `n : 1` | `not declared` | stored |
| human resources Work Entry | `hr.work.entry` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| human resources Work Entry | `hr.work.entry` | `department_id` | link to one record | Department | `hr.department` | `n : 0..1` | `not declared` | stored |
| human resources Work Entry | `hr.work.entry` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 1` | `not declared` | stored |
| human resources Work Entry | `hr.work.entry` | `leave_id` | link to one record | Time Off | `hr.leave` | `n : 0..1` | `not declared` | stored |
| human resources Work Entry | `hr.work.entry` | `version_id` | link to one record | Version | `hr.version` | `n : 1` | `not declared` | stored |
| human resources Work Entry | `hr.work.entry` | `work_entry_type_id` | link to one record | human resources Work Entry Type | `hr.work.entry.type` | `n : 0..1` | `not declared` | stored |
| human resources Work Entry Type | `hr.work.entry.type` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| human resources Work Entry Type | `hr.work.entry.type` | `leave_type_ids` | list of records | Time Off Type | `hr.leave.type` | `1 : 0..n` | `mirror of the target column` | derived |

### 3.44 Events

Events and event types, tickets, registrations and answers, booths and booth categories, tracks and track stages, sponsors, tags, stages and event communications.

Specified in [`../domains/events/`](../domains/events/).

#### Persistent entities (33)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Content Quiz Question | `event.quiz.question` | `event_quiz_question` | Persistent record with 3 stored columns; belongs to Quiz; owns Question's Answer; referenced by 1 relation field. | Quizzes on Tracks |
| Event | `event.event` | `event_event` | Persistent record with 45 stored columns; owns Event Automated Mailing, Event Booth, Event Registration and 7 further collections; states of `kanban_state`: In Progress, Ready for Next Stage, Blocked, Cancelled; company scoped; referenced by 21 relation fields. | Events Organization |
| Event Automated Mailing | `event.mail` | `event_mail` | Persistent record with 11 stored columns; belongs to Event; owns Registration Mail Scheduler, Slot Mail Scheduler; states of `mail_state`: Running, Scheduled, Sent, Error, Cancelled; referenced by 2 relation fields. | Events Organization |
| Event Booth | `event.booth` | `event_booth` | Persistent record with 14 stored columns; belongs to Event; owns Event Booth Registration; lifecycle states Available, Unavailable; referenced by 3 relation fields. | Events Booths |
| Event Booth Category | `event.booth.category` | `event_booth_category` | Persistent record with 9 stored columns; belongs to Product Variant; owns Event Booth; referenced by 5 relation fields. | Events Booths |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | Persistent record with 11 stored columns; belongs to Event Booth, Sales Order Line. | Events Booths Sales |
| Event Booth Template | `event.type.booth` | `event_type_booth` | Persistent record with 4 stored columns; belongs to Event Booth Category, Event Template. | Events Booths |
| Event Question | `event.question` | `event_question` | Persistent record with 8 stored columns; owns Event Question Answer; referenced by 6 relation fields. | Events Organization |
| Event Question Answer | `event.question.answer` | `event_question_answer` | Persistent record with 3 stored columns; belongs to Event Question; referenced by 1 relation field. | Events Organization |
| Event Registration | `event.registration` | `event_registration` | Persistent record with 22 stored columns; belongs to Event; owns Event Registration Answer, Registration Mail Scheduler; lifecycle states Unconfirmed, Registered, Attended, Cancelled; 2 state fields in all; company scoped; referenced by 7 relation fields. | Events Organization |
| Event Registration Answer | `event.registration.answer` | `event_registration_answer` | Persistent record with 4 stored columns; belongs to Event Question, Event Registration. | Events Organization |
| Event Sales Report | `event.sale.report` | `event_sale_report` | Persistent record with 25 stored columns; states of `event_registration_state`: Unconfirmed, Cancelled, Confirmed, Attended; 3 state fields in all; company scoped. | Events Sales |
| Event Slot | `event.slot` | `event_slot` | Persistent record with 7 stored columns; belongs to Event; owns Event Registration; referenced by 6 relation fields. | Events Organization |
| Event Sponsor | `event.sponsor` | `event_sponsor` | Persistent record with 16 stored columns; belongs to Contact, Event, Event Sponsor Level; referenced by 1 relation field. | Event Exhibitors |
| Event Sponsor Level | `event.sponsor.type` | `event_sponsor_type` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Event Exhibitors |
| Event Stage | `event.stage` | `event_stage` | Persistent record with 5 stored columns; referenced by 1 relation field. | Events Organization |
| Event Tag | `event.tag` | `event_tag` | Persistent record with 7 stored columns; belongs to Event Tag Category; referenced by 2 relation fields. | Events Organization |
| Event Tag Category | `event.tag.category` | `event_tag_category` | Persistent record with 4 stored columns; owns Event Tag; referenced by 1 relation field. | Events Organization |
| Event Template | `event.type` | `event_type` | Persistent record with 13 stored columns; owns Event Booth Template, Event Template Ticket, Mail Scheduling on Event Category; referenced by 7 relation fields. | Events Organization |
| Event Template Ticket | `event.type.ticket` | `event_type_ticket` | Persistent record with 8 stored columns; belongs to Event Template, Product Variant. | Events Organization |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | Persistent record with 13 stored columns; belongs to Event; owns Event Registration; company scoped; referenced by 6 relation fields. | Events Organization |
| Event Track | `event.track` | `event_track` | Persistent record with 38 stored columns; belongs to Event, Event Track Stage; owns Quiz, Track / Visitor Link; states of `kanban_state`: Grey, Green, Red; company scoped; referenced by 4 relation fields. | Advanced Events |
| Event Track Location | `event.track.location` | `event_track_location` | Persistent record with 2 stored columns; referenced by 1 relation field. | Advanced Events |
| Event Track Stage | `event.track.stage` | `event_track_stage` | Persistent record with 12 stored columns; referenced by 1 relation field. | Advanced Events |
| Event Track Tag | `event.track.tag` | `event_track_tag` | Persistent record with 4 stored columns; referenced by 3 relation fields. | Advanced Events |
| Event Track Tag Category | `event.track.tag.category` | `event_track_tag_category` | Persistent record with 2 stored columns; owns Event Track Tag; referenced by 1 relation field. | Advanced Events |
| Mail Scheduling on Event Category | `event.type.mail` | `event_type_mail` | Persistent record with 5 stored columns; belongs to Event Template. | Events Organization |
| Question's Answer | `event.quiz.answer` | `event_quiz_answer` | Persistent record with 6 stored columns; belongs to Content Quiz Question. | Quizzes on Tracks |
| Quiz | `event.quiz` | `event_quiz` | Persistent record with 4 stored columns; owns Content Quiz Question; referenced by 2 relation fields. | Quizzes on Tracks |
| Registration Mail Scheduler | `event.mail.registration` | `event_mail_registration` | Persistent record with 4 stored columns; belongs to Event Automated Mailing, Event Registration. | Events Organization |
| Slot Mail Scheduler | `event.mail.slot` | `event_mail_slot` | Persistent record with 6 stored columns; belongs to Event Automated Mailing, Event Slot. | Events Organization |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | Persistent record with 7 stored columns; belongs to Event Track. | Advanced Events |
| Website Event Menu | `website.event.menu` | `website_event_menu` | Persistent record with 10 stored columns. | Events |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Edit Attendee Details on Sales Confirmation | `registration.editor` | `registration_editor` | Interactive assistant with 1 stored column; belongs to Sales Order; owns Edit Attendee Line on Sales Confirmation; referenced by 1 relation field. | Events Sales |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `registration_editor_line` | Interactive assistant with 9 stored columns; belongs to Event. | Events Sales |
| Event Booth Configurator | `event.booth.configurator` | `event_booth_configurator` | Interactive assistant with 4 stored columns; belongs to Event, Event Booth Category. | Events Booths Sales |
| Event Configurator | `event.event.configurator` | `event_event_configurator` | Interactive assistant with 4 stored columns. | Events Sales |

#### Relationships (159)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Content Quiz Question | `event.quiz.question` | `answer_ids` | list of records | Question's Answer | `event.quiz.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Content Quiz Question | `event.quiz.question` | `correct_answer_id` | list of records | Question's Answer | `event.quiz.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Content Quiz Question | `event.quiz.question` | `quiz_id` | link to one record | Quiz | `event.quiz` | `n : 1` | `cascade` | stored |
| Edit Attendee Details on Sales Confirmation | `registration.editor` | `event_registration_ids` | list of records | Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Edit Attendee Details on Sales Confirmation | `registration.editor` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 1` | `cascade` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `editor_id` | link to one record | Edit Attendee Details on Sales Confirmation | `registration.editor` | `n : 0..1` | `not declared` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `not declared` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 0..1` | `not declared` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `not declared` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `registration_id` | link to one record | Event Registration | `event.registration` | `n : 0..1` | `not declared` | stored |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `address_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `address_search` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Event | `event.event` | `allowed_track_tag_ids` | list on both sides | Event Track Tag | `event.track.tag` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `booth_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `community_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `currency_id` | link to one record | Currency | `res.currency` | `n : 0..1` | `not declared` | derived |
| Event | `event.event` | `event_booth_category_available_ids` | list on both sides | Event Booth Category | `event.booth.category` | `0..n : 0..n` | `not declared` | derived |
| Event | `event.event` | `event_booth_category_ids` | list on both sides | Event Booth Category | `event.booth.category` | `0..n : 0..n` | `not declared` | derived |
| Event | `event.event` | `event_booth_ids` | list of records | Event Booth | `event.booth` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `event_mail_ids` | list of records | Event Automated Mailing | `event.mail` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `event_slot_ids` | list of records | Event Slot | `event.slot` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `event_ticket_ids` | list of records | Event Ticket | `event.event.ticket` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `event_type_id` | link to one record | Event Template | `event.type` | `n : 0..1` | `set null` | stored |
| Event | `event.event` | `exhibitor_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `general_question_ids` | list on both sides | Event Question | `event.question` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `introduction_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `menu_id` | link to one record | Website Menu | `website.menu` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `organizer_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event | `event.event` | `other_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `question_ids` | list on both sides | Event Question | `event.question` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `register_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `sale_order_lines_ids` | list of records | Sales Order Line | `sale.order.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `specific_question_ids` | list on both sides | Event Question | `event.question` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `sponsor_ids` | list of records | Event Sponsor | `event.sponsor` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `stage_id` | link to one record | Event Stage | `event.stage` | `n : 0..1` | `restrict` | stored |
| Event | `event.event` | `tag_ids` | list on both sides | Event Tag | `event.tag` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `track_ids` | list of records | Event Track | `event.track` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `track_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `track_proposal_menu_ids` | list of records | Website Event Menu | `website.event.menu` | `1 : 0..n` | `mirror of the target column` | derived |
| Event | `event.event` | `tracks_tag_ids` | list on both sides | Event Track Tag | `event.track.tag` | `0..n : 0..n` | `not declared` | stored |
| Event | `event.event` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Event Automated Mailing | `event.mail` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `cascade` | stored |
| Event Automated Mailing | `event.mail` | `last_registration_id` | link to one record | Event Registration | `event.registration` | `n : 0..1` | `not declared` | stored |
| Event Automated Mailing | `event.mail` | `mail_registration_ids` | list of records | Registration Mail Scheduler | `event.mail.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Automated Mailing | `event.mail` | `mail_slot_ids` | list of records | Slot Mail Scheduler | `event.mail.slot` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Booth | `event.booth` | `event_booth_registration_ids` | list of records | Event Booth Registration | `event.booth.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Booth | `event.booth` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `cascade` | stored |
| Event Booth | `event.booth` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Booth | `event.booth` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `set null` | stored |
| Event Booth | `event.booth` | `sale_order_line_registration_ids` | list on both sides | Sales Order Line | `sale.order.line` | `0..n : 0..n` | `not declared` | stored |
| Event Booth | `event.booth` | `sponsor_id` | link to one record | Event Sponsor | `event.sponsor` | `n : 0..1` | `not declared` | stored |
| Event Booth Category | `event.booth.category` | `booth_ids` | list of records | Event Booth | `event.booth` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Booth Category | `event.booth.category` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Event Booth Category | `event.booth.category` | `sponsor_type_id` | link to one record | Event Sponsor Level | `event.sponsor.type` | `n : 0..1` | `not declared` | stored |
| Event Booth Configurator | `event.booth.configurator` | `event_booth_category_id` | link to one record | Event Booth Category | `event.booth.category` | `n : 1` | `not declared` | stored |
| Event Booth Configurator | `event.booth.configurator` | `event_booth_ids` | list on both sides | Event Booth | `event.booth` | `0..n : 0..n` | `not declared` | stored |
| Event Booth Configurator | `event.booth.configurator` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `not declared` | stored |
| Event Booth Configurator | `event.booth.configurator` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Event Booth Configurator | `event.booth.configurator` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Event Booth Registration | `event.booth.registration` | `event_booth_id` | link to one record | Event Booth | `event.booth` | `n : 1` | `not declared` | stored |
| Event Booth Registration | `event.booth.registration` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Booth Registration | `event.booth.registration` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 1` | `cascade` | stored |
| Event Booth Template | `event.type.booth` | `booth_category_id` | link to one record | Event Booth Category | `event.booth.category` | `n : 1` | `restrict` | stored |
| Event Booth Template | `event.type.booth` | `event_type_id` | link to one record | Event Template | `event.type` | `n : 1` | `cascade` | stored |
| Event Configurator | `event.event.configurator` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Event Configurator | `event.event.configurator` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 0..1` | `not declared` | stored |
| Event Configurator | `event.event.configurator` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `not declared` | stored |
| Event Configurator | `event.event.configurator` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Event Question | `event.question` | `answer_ids` | list of records | Event Question Answer | `event.question.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Question | `event.question` | `event_ids` | list on both sides | Event | `event.event` | `0..n : 0..n` | `not declared` | stored |
| Event Question | `event.question` | `event_type_ids` | list on both sides | Event Template | `event.type` | `0..n : 0..n` | `not declared` | stored |
| Event Question Answer | `event.question.answer` | `question_id` | link to one record | Event Question | `event.question` | `n : 1` | `cascade` | stored |
| Event Registration | `event.registration` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Event Registration | `event.registration` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `not declared` | stored |
| Event Registration | `event.registration` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 0..1` | `restrict` | stored |
| Event Registration | `event.registration` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `restrict` | stored |
| Event Registration | `event.registration` | `lead_ids` | list on both sides | Lead | `crm.lead` | `0..n : 0..n` | `not declared` | stored |
| Event Registration | `event.registration` | `mail_registration_ids` | list of records | Registration Mail Scheduler | `event.mail.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Registration | `event.registration` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Registration | `event.registration` | `pos_order_line_id` | link to one record | Point of Sale Order Lines | `pos.order.line` | `n : 0..1` | `cascade` | stored |
| Event Registration | `event.registration` | `registration_answer_choice_ids` | list of records | Event Registration Answer | `event.registration.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Registration | `event.registration` | `registration_answer_ids` | list of records | Event Registration Answer | `event.registration.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Registration | `event.registration` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `cascade` | stored |
| Event Registration | `event.registration` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `cascade` | stored |
| Event Registration | `event.registration` | `utm_campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `set null` | stored |
| Event Registration | `event.registration` | `utm_medium_id` | link to one record | campaign tracking parameter Medium | `utm.medium` | `n : 0..1` | `set null` | stored |
| Event Registration | `event.registration` | `utm_source_id` | link to one record | campaign tracking parameter Source | `utm.source` | `n : 0..1` | `set null` | stored |
| Event Registration | `event.registration` | `visitor_id` | link to one record | Website Visitor | `website.visitor` | `n : 0..1` | `set null` | stored |
| Event Registration Answer | `event.registration.answer` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | derived |
| Event Registration Answer | `event.registration.answer` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Event Registration Answer | `event.registration.answer` | `question_id` | link to one record | Event Question | `event.question` | `n : 1` | `restrict` | stored |
| Event Registration Answer | `event.registration.answer` | `registration_id` | link to one record | Event Registration | `event.registration` | `n : 1` | `cascade` | stored |
| Event Registration Answer | `event.registration.answer` | `value_answer_id` | link to one record | Event Question Answer | `event.question.answer` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `event_registration_id` | link to one record | Event Registration | `event.registration` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `event_ticket_id` | link to one record | Event Ticket | `event.event.ticket` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `event_type_id` | link to one record | Event Template | `event.type` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `invoice_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `sale_order_id` | link to one record | Sales Order | `sale.order` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `sale_order_line_id` | link to one record | Sales Order Line | `sale.order.line` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `sale_order_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Sales Report | `event.sale.report` | `sale_order_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Event Slot | `event.slot` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `cascade` | stored |
| Event Slot | `event.slot` | `registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Sponsor | `event.sponsor` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | derived |
| Event Sponsor | `event.sponsor` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `not declared` | stored |
| Event Sponsor | `event.sponsor` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `not declared` | stored |
| Event Sponsor | `event.sponsor` | `sponsor_type_id` | link to one record | Event Sponsor Level | `event.sponsor.type` | `n : 1` | `not declared` | stored |
| Event Tag | `event.tag` | `category_id` | link to one record | Event Tag Category | `event.tag.category` | `n : 1` | `cascade` | stored |
| Event Tag Category | `event.tag.category` | `tag_ids` | list of records | Event Tag | `event.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Template | `event.type` | `event_type_booth_ids` | list of records | Event Booth Template | `event.type.booth` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Template | `event.type` | `event_type_mail_ids` | list of records | Mail Scheduling on Event Category | `event.type.mail` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Template | `event.type` | `event_type_ticket_ids` | list of records | Event Template Ticket | `event.type.ticket` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Template | `event.type` | `question_ids` | list on both sides | Event Question | `event.question` | `0..n : 0..n` | `not declared` | stored |
| Event Template | `event.type` | `tag_ids` | list on both sides | Event Tag | `event.tag` | `0..n : 0..n` | `not declared` | stored |
| Event Template Ticket | `event.type.ticket` | `event_type_id` | link to one record | Event Template | `event.type` | `n : 1` | `cascade` | stored |
| Event Template Ticket | `event.type.ticket` | `product_id` | link to one record | Product Variant | `product.product` | `n : 1` | `not declared` | stored |
| Event Ticket | `event.event.ticket` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Event Ticket | `event.event.ticket` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `cascade` | stored |
| Event Ticket | `event.event.ticket` | `registration_ids` | list of records | Event Registration | `event.registration` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Track | `event.track` | `company_id` | link to one record | Companies | `res.company` | `n : 0..1` | `not declared` | derived |
| Event Track | `event.track` | `event_id` | link to one record | Event | `event.event` | `n : 1` | `not declared` | stored |
| Event Track | `event.track` | `event_track_visitor_ids` | list of records | Track / Visitor Link | `event.track.visitor` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Track | `event.track` | `location_id` | link to one record | Event Track Location | `event.track.location` | `n : 0..1` | `not declared` | stored |
| Event Track | `event.track` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Event Track | `event.track` | `quiz_id` | link to one record | Quiz | `event.quiz` | `n : 0..1` | `not declared` | stored |
| Event Track | `event.track` | `quiz_ids` | list of records | Quiz | `event.quiz` | `1 : 0..n` | `mirror of the target column` | derived |
| Event Track | `event.track` | `stage_id` | link to one record | Event Track Stage | `event.track.stage` | `n : 1` | `restrict` | stored |
| Event Track | `event.track` | `tag_ids` | list on both sides | Event Track Tag | `event.track.tag` | `0..n : 0..n` | `not declared` | stored |
| Event Track | `event.track` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Event Track | `event.track` | `wishlist_visitor_ids` | list on both sides | Website Visitor | `website.visitor` | `0..n : 0..n` | `not declared` | derived |
| Event Track Stage | `event.track.stage` | `mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Event Track Tag | `event.track.tag` | `category_id` | link to one record | Event Track Tag Category | `event.track.tag.category` | `n : 0..1` | `set null` | stored |
| Event Track Tag | `event.track.tag` | `track_ids` | list on both sides | Event Track | `event.track` | `0..n : 0..n` | `not declared` | stored |
| Event Track Tag Category | `event.track.tag.category` | `tag_ids` | list of records | Event Track Tag | `event.track.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Mail Scheduling on Event Category | `event.type.mail` | `event_type_id` | link to one record | Event Template | `event.type` | `n : 1` | `cascade` | stored |
| Question's Answer | `event.quiz.answer` | `question_id` | link to one record | Content Quiz Question | `event.quiz.question` | `n : 1` | `cascade` | stored |
| Quiz | `event.quiz` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `not declared` | stored |
| Quiz | `event.quiz` | `event_track_id` | link to one record | Event Track | `event.track` | `n : 0..1` | `not declared` | stored |
| Quiz | `event.quiz` | `question_ids` | list of records | Content Quiz Question | `event.quiz.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Registration Mail Scheduler | `event.mail.registration` | `registration_id` | link to one record | Event Registration | `event.registration` | `n : 1` | `cascade` | stored |
| Registration Mail Scheduler | `event.mail.registration` | `scheduler_id` | link to one record | Event Automated Mailing | `event.mail` | `n : 1` | `cascade` | stored |
| Slot Mail Scheduler | `event.mail.slot` | `event_slot_id` | link to one record | Event Slot | `event.slot` | `n : 1` | `cascade` | stored |
| Slot Mail Scheduler | `event.mail.slot` | `last_registration_id` | link to one record | Event Registration | `event.registration` | `n : 0..1` | `not declared` | stored |
| Slot Mail Scheduler | `event.mail.slot` | `scheduler_id` | link to one record | Event Automated Mailing | `event.mail` | `n : 1` | `cascade` | stored |
| Track / Visitor Link | `event.track.visitor` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `set null` | stored |
| Track / Visitor Link | `event.track.visitor` | `track_id` | link to one record | Event Track | `event.track` | `n : 1` | `cascade` | stored |
| Track / Visitor Link | `event.track.visitor` | `visitor_id` | link to one record | Website Visitor | `website.visitor` | `n : 0..1` | `cascade` | stored |
| Website Event Menu | `website.event.menu` | `event_id` | link to one record | Event | `event.event` | `n : 0..1` | `cascade` | stored |
| Website Event Menu | `website.event.menu` | `menu_id` | link to one record | Website Menu | `website.menu` | `n : 0..1` | `cascade` | stored |
| Website Event Menu | `website.event.menu` | `view_id` | link to one record | View | `ir.ui.view` | `n : 0..1` | `cascade` | stored |

### 3.45 Learning, Surveys and Gamification

Surveys, questions and answers, participations and scoring, courses, slides and content, quizzes, certifications, forums and posts, badges, challenges, goals and karma.

Specified in the domain folder `../domains/learning-surveys-and-gamification/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (25)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Additional resource for a particular slide | `slide.slide.resource` | `slide_slide_resource` | Persistent record with 6 stored columns; belongs to Slides. | Electronic Learning |
| Channel / Partners (Members) | `slide.channel.partner` | `slide_channel_partner` | Persistent record with 8 stored columns; belongs to Contact, Course; states of `member_status`: Invite Sent, Joined, Ongoing, Finished. | Electronic Learning |
| Channel/Course Groups | `slide.channel.tag.group` | `slide_channel_tag_group` | Persistent record with 3 stored columns; owns Channel/Course Tag; referenced by 1 relation field. | Electronic Learning |
| Channel/Course Tag | `slide.channel.tag` | `slide_channel_tag` | Persistent record with 5 stored columns; belongs to Channel/Course Groups; referenced by 1 relation field. | Electronic Learning |
| Content Quiz Question | `slide.question` | `slide_question` | Persistent record with 3 stored columns; belongs to Slides; owns Slide Question's Answer; referenced by 1 relation field. | Electronic Learning |
| Course | `slide.channel` | `slide_channel` | Persistent record with 48 stored columns; owns Channel / Partners (Members), Slide / Partner decorated m2m, Slides; referenced by 11 relation fields. | Electronic Learning |
| Embedded Slides View Counter | `slide.embed` | `slide_embed` | Persistent record with 3 stored columns; belongs to Slides. | Electronic Learning |
| Gamification Badge | `gamification.badge` | `gamification_badge` | Persistent record with 9 stored columns; owns Gamification Challenge, Gamification User Badge, Survey; referenced by 8 relation fields. | Gamification |
| Gamification Challenge | `gamification.challenge` | `gamification_challenge` | Persistent record with 22 stored columns; belongs to Email Templates; owns Gamification generic goal for challenge; lifecycle states Draft, In Progress, Done; referenced by 2 relation fields. | Gamification |
| Gamification generic goal for challenge | `gamification.challenge.line` | `gamification_challenge_line` | Persistent record with 4 stored columns; belongs to Gamification Challenge, Gamification Goal Definition; referenced by 1 relation field. | Gamification |
| Gamification Goal | `gamification.goal` | `gamification_goal` | Persistent record with 13 stored columns; belongs to Gamification Goal Definition, User; lifecycle states Draft, In progress, Reached, Failed, Cancelled; referenced by 1 relation field. | Gamification |
| Gamification Goal Definition | `gamification.goal.definition` | `gamification_goal_definition` | Persistent record with 17 stored columns; referenced by 3 relation fields. | Gamification |
| Gamification User Badge | `gamification.badge.user` | `gamification_badge_user` | Persistent record with 7 stored columns; belongs to Gamification Badge, User. | Gamification |
| Product Attribute Category | `product.attribute.category` | `product_attribute_category` | Persistent record with 2 stored columns; owns Product Attribute; referenced by 1 relation field. | Product Comparison |
| Rank based on karma | `gamification.karma.rank` | `gamification_karma_rank` | Persistent record with 4 stored columns; owns User; referenced by 2 relation fields. | Gamification |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_slide_partner` | Persistent record with 7 stored columns; belongs to Contact, Slides; owns Survey User Input; referenced by 2 relation fields. | Electronic Learning |
| Slide Question's Answer | `slide.answer` | `slide_answer` | Persistent record with 5 stored columns; belongs to Content Quiz Question. | Electronic Learning |
| Slide Tag | `slide.tag` | `slide_tag` | Persistent record with 1 stored column; referenced by 1 relation field. | Electronic Learning |
| Slides | `slide.slide` | `slide_slide` | Persistent record with 41 stored columns; belongs to Course; owns Additional resource for a particular slide, Content Quiz Question, Embedded Slides View Counter and 2 further collections; referenced by 8 relation fields. | Electronic Learning |
| Survey | `survey.survey` | `survey_survey` | Persistent record with 35 stored columns; owns Course, Job Position, Lead and 3 further collections; states of `session_state`: Ready, In Progress; referenced by 9 relation fields. | Surveys |
| Survey Label | `survey.question.answer` | `survey_question_answer` | Persistent record with 8 stored columns; referenced by 3 relation fields. | Surveys |
| Survey Question | `survey.question` | `survey_question` | Persistent record with 41 stored columns; owns Survey Label, Survey Question, Survey User Input Line; referenced by 9 relation fields. | Surveys |
| Survey User Input | `survey.user_input` | `survey_user_input` | Persistent record with 22 stored columns; belongs to Survey; owns Survey User Input Line; lifecycle states New, In Progress, Completed; referenced by 1 relation field. | Surveys |
| Survey User Input Line | `survey.user_input.line` | `survey_user_input_line` | Persistent record with 16 stored columns; belongs to Survey Question, Survey User Input. | Surveys |
| Track Karma Changes | `gamification.karma.tracking` | `gamification_karma_tracking` | Persistent record with 8 stored columns; belongs to User. | Gamification |

#### Interactive assistant entities (4)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Channel Invitation Wizard | `slide.channel.invite` | `slide_channel_invite` | Interactive assistant with 7 stored columns; belongs to Course. | Electronic Learning |
| Gamification Goal Wizard | `gamification.goal.wizard` | `gamification_goal_wizard` | Interactive assistant with 2 stored columns; belongs to Gamification Goal. | Gamification |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `gamification_badge_user_wizard` | Interactive assistant with 4 stored columns; belongs to Gamification Badge, User. | Gamification |
| Survey Invitation Wizard | `survey.invite` | `survey_invite` | Interactive assistant with 11 stored columns; belongs to Survey. | Surveys |

#### Relationships (143)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Additional resource for a particular slide | `slide.slide.resource` | `slide_id` | link to one record | Slides | `slide.slide` | `n : 1` | `cascade` | stored |
| Channel / Partners (Members) | `slide.channel.partner` | `channel_id` | link to one record | Course | `slide.channel` | `n : 1` | `cascade` | stored |
| Channel / Partners (Members) | `slide.channel.partner` | `channel_user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | derived |
| Channel / Partners (Members) | `slide.channel.partner` | `channel_website_id` | link to one record | Website | `website` | `n : 0..1` | `not declared` | derived |
| Channel / Partners (Members) | `slide.channel.partner` | `next_slide_id` | link to one record | Slides | `slide.slide` | `n : 0..1` | `not declared` | derived |
| Channel / Partners (Members) | `slide.channel.partner` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Channel Invitation Wizard | `slide.channel.invite` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Channel Invitation Wizard | `slide.channel.invite` | `channel_id` | link to one record | Course | `slide.channel` | `n : 1` | `not declared` | stored |
| Channel Invitation Wizard | `slide.channel.invite` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Channel/Course Groups | `slide.channel.tag.group` | `tag_ids` | list of records | Channel/Course Tag | `slide.channel.tag` | `1 : 0..n` | `mirror of the target column` | derived |
| Channel/Course Tag | `slide.channel.tag` | `channel_ids` | list on both sides | Course | `slide.channel` | `0..n : 0..n` | `not declared` | stored |
| Channel/Course Tag | `slide.channel.tag` | `group_id` | link to one record | Channel/Course Groups | `slide.channel.tag.group` | `n : 1` | `cascade` | stored |
| Content Quiz Question | `slide.question` | `answer_ids` | list of records | Slide Question's Answer | `slide.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Content Quiz Question | `slide.question` | `slide_id` | link to one record | Slides | `slide.slide` | `n : 1` | `cascade` | stored |
| Course | `slide.channel` | `channel_partner_all_ids` | list of records | Channel / Partners (Members) | `slide.channel.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `channel_partner_ids` | list of records | Channel / Partners (Members) | `slide.channel.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `completed_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `enroll_group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Course | `slide.channel` | `forum_id` | link to one record | Forum | `forum.forum` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Course | `slide.channel` | `prerequisite_channel_ids` | list on both sides | Course | `slide.channel` | `0..n : 0..n` | `not declared` | stored |
| Course | `slide.channel` | `prerequisite_of_channel_ids` | list on both sides | Course | `slide.channel` | `0..n : 0..n` | `not declared` | stored |
| Course | `slide.channel` | `product_id` | link to one record | Product Variant | `product.product` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `promoted_slide_id` | link to one record | Slides | `slide.slide` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `publish_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `share_channel_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `share_slide_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Course | `slide.channel` | `slide_category_ids` | list of records | Slides | `slide.slide` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `slide_content_ids` | list of records | Slides | `slide.slide` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `slide_ids` | list of records | Slides | `slide.slide` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `slide_partner_ids` | list of records | Slide / Partner decorated m2m | `slide.slide.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Course | `slide.channel` | `tag_ids` | list on both sides | Channel/Course Tag | `slide.channel.tag` | `0..n : 0..n` | `not declared` | stored |
| Course | `slide.channel` | `upload_group_ids` | list on both sides | Access Groups | `res.groups` | `0..n : 0..n` | `not declared` | stored |
| Course | `slide.channel` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Embedded Slides View Counter | `slide.embed` | `slide_id` | link to one record | Slides | `slide.slide` | `n : 1` | `cascade` | stored |
| Gamification Badge | `gamification.badge` | `challenge_ids` | list of records | Gamification Challenge | `gamification.challenge` | `1 : 0..n` | `mirror of the target column` | derived |
| Gamification Badge | `gamification.badge` | `goal_definition_ids` | list on both sides | Gamification Goal Definition | `gamification.goal.definition` | `0..n : 0..n` | `not declared` | stored |
| Gamification Badge | `gamification.badge` | `owner_ids` | list of records | Gamification User Badge | `gamification.badge.user` | `1 : 0..n` | `mirror of the target column` | derived |
| Gamification Badge | `gamification.badge` | `rule_auth_badge_ids` | list on both sides | Gamification Badge | `gamification.badge` | `0..n : 0..n` | `not declared` | stored |
| Gamification Badge | `gamification.badge` | `rule_auth_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Gamification Badge | `gamification.badge` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `not declared` | stored |
| Gamification Badge | `gamification.badge` | `survey_ids` | list of records | Survey | `survey.survey` | `1 : 0..n` | `mirror of the target column` | derived |
| Gamification Badge | `gamification.badge` | `unique_owner_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | derived |
| Gamification Challenge | `gamification.challenge` | `invited_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `line_ids` | list of records | Gamification generic goal for challenge | `gamification.challenge.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Gamification Challenge | `gamification.challenge` | `manager_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `report_message_group_id` | link to one record | Discussion Channel | `discuss.channel` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `report_template_id` | link to one record | Email Templates | `mail.template` | `n : 1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `reward_first_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `reward_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `reward_second_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `reward_third_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 0..1` | `not declared` | stored |
| Gamification Challenge | `gamification.challenge` | `user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Gamification Goal | `gamification.goal` | `definition_id` | link to one record | Gamification Goal Definition | `gamification.goal.definition` | `n : 1` | `cascade` | stored |
| Gamification Goal | `gamification.goal` | `line_id` | link to one record | Gamification generic goal for challenge | `gamification.challenge.line` | `n : 0..1` | `cascade` | stored |
| Gamification Goal | `gamification.goal` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Gamification Goal | `gamification.goal` | `user_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Gamification Goal Definition | `gamification.goal.definition` | `action_id` | link to one record | Action Window | `ir.actions.act_window` | `n : 0..1` | `not declared` | stored |
| Gamification Goal Definition | `gamification.goal.definition` | `batch_distinctive_field` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Gamification Goal Definition | `gamification.goal.definition` | `field_date_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Gamification Goal Definition | `gamification.goal.definition` | `field_id` | link to one record | Fields | `ir.model.fields` | `n : 0..1` | `not declared` | stored |
| Gamification Goal Definition | `gamification.goal.definition` | `model_id` | link to one record | Models | `ir.model` | `n : 0..1` | `cascade` | stored |
| Gamification Goal Definition | `gamification.goal.definition` | `model_inherited_ids` | list on both sides | Models | `ir.model` | `0..n : 0..n` | `not declared` | derived |
| Gamification Goal Wizard | `gamification.goal.wizard` | `goal_id` | link to one record | Gamification Goal | `gamification.goal` | `n : 1` | `not declared` | stored |
| Gamification User Badge | `gamification.badge.user` | `badge_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 1` | `cascade` | stored |
| Gamification User Badge | `gamification.badge.user` | `challenge_id` | link to one record | Gamification Challenge | `gamification.challenge` | `n : 0..1` | `not declared` | stored |
| Gamification User Badge | `gamification.badge.user` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Gamification User Badge | `gamification.badge.user` | `sender_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Gamification User Badge | `gamification.badge.user` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |
| Gamification User Badge | `gamification.badge.user` | `user_partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | derived |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `badge_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 1` | `not declared` | stored |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `employee_id` | link to one record | Employee | `hr.employee` | `n : 0..1` | `not declared` | stored |
| Gamification User Badge Wizard | `gamification.badge.user.wizard` | `user_id` | link to one record | User | `res.users` | `n : 1` | `not declared` | stored |
| Gamification generic goal for challenge | `gamification.challenge.line` | `challenge_id` | link to one record | Gamification Challenge | `gamification.challenge` | `n : 1` | `cascade` | stored |
| Gamification generic goal for challenge | `gamification.challenge.line` | `definition_id` | link to one record | Gamification Goal Definition | `gamification.goal.definition` | `n : 1` | `cascade` | stored |
| Product Attribute Category | `product.attribute.category` | `attribute_ids` | list of records | Product Attribute | `product.attribute` | `1 : 0..n` | `mirror of the target column` | derived |
| Rank based on karma | `gamification.karma.rank` | `user_ids` | list of records | User | `res.users` | `1 : 0..n` | `mirror of the target column` | derived |
| Slide / Partner decorated m2m | `slide.slide.partner` | `channel_id` | link to one record | Course | `slide.channel` | `n : 0..1` | `cascade` | stored |
| Slide / Partner decorated m2m | `slide.slide.partner` | `partner_id` | link to one record | Contact | `res.partner` | `n : 1` | `cascade` | stored |
| Slide / Partner decorated m2m | `slide.slide.partner` | `slide_id` | link to one record | Slides | `slide.slide` | `n : 1` | `cascade` | stored |
| Slide / Partner decorated m2m | `slide.slide.partner` | `user_input_ids` | list of records | Survey User Input | `survey.user_input` | `1 : 0..n` | `mirror of the target column` | derived |
| Slide Question's Answer | `slide.answer` | `question_id` | link to one record | Content Quiz Question | `slide.question` | `n : 1` | `cascade` | stored |
| Slides | `slide.slide` | `category_id` | link to one record | Slides | `slide.slide` | `n : 0..1` | `not declared` | stored |
| Slides | `slide.slide` | `channel_id` | link to one record | Course | `slide.channel` | `n : 1` | `cascade` | stored |
| Slides | `slide.slide` | `embed_ids` | list of records | Embedded Slides View Counter | `slide.embed` | `1 : 0..n` | `mirror of the target column` | derived |
| Slides | `slide.slide` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Slides | `slide.slide` | `question_ids` | list of records | Content Quiz Question | `slide.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Slides | `slide.slide` | `slide_ids` | list of records | Slides | `slide.slide` | `1 : 0..n` | `mirror of the target column` | derived |
| Slides | `slide.slide` | `slide_partner_ids` | list of records | Slide / Partner decorated m2m | `slide.slide.partner` | `1 : 0..n` | `mirror of the target column` | derived |
| Slides | `slide.slide` | `slide_resource_ids` | list of records | Additional resource for a particular slide | `slide.slide.resource` | `1 : 0..n` | `mirror of the target column` | derived |
| Slides | `slide.slide` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `not declared` | stored |
| Slides | `slide.slide` | `tag_ids` | list on both sides | Slide Tag | `slide.tag` | `0..n : 0..n` | `not declared` | stored |
| Slides | `slide.slide` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Slides | `slide.slide` | `user_membership_id` | link to one record | Slide / Partner decorated m2m | `slide.slide.partner` | `n : 0..1` | `not declared` | derived |
| Survey | `survey.survey` | `certification_badge_id` | link to one record | Gamification Badge | `gamification.badge` | `n : 0..1` | `not declared` | stored |
| Survey | `survey.survey` | `certification_mail_template_id` | link to one record | Email Templates | `mail.template` | `n : 0..1` | `not declared` | stored |
| Survey | `survey.survey` | `hr_job_ids` | list of records | Job Position | `hr.job` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `lang_ids` | list on both sides | Languages | `res.lang` | `0..n : 0..n` | `not declared` | stored |
| Survey | `survey.survey` | `lead_ids` | list of records | Lead | `crm.lead` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `page_ids` | list of records | Survey Question | `survey.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `question_and_page_ids` | list of records | Survey Question | `survey.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `question_ids` | list of records | Survey Question | `survey.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `restrict_user_ids` | list on both sides | User | `res.users` | `0..n : 0..n` | `not declared` | stored |
| Survey | `survey.survey` | `session_question_id` | link to one record | Survey Question | `survey.question` | `n : 0..1` | `not declared` | stored |
| Survey | `survey.survey` | `slide_channel_ids` | list of records | Course | `slide.channel` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `slide_ids` | list of records | Slides | `slide.slide` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey | `survey.survey` | `team_id` | link to one record | Sales Team | `crm.team` | `n : 0..1` | `set null` | stored |
| Survey | `survey.survey` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Survey | `survey.survey` | `user_input_ids` | list of records | Survey User Input | `survey.user_input` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey Invitation Wizard | `survey.invite` | `applicant_id` | link to one record | Applicant | `hr.applicant` | `n : 0..1` | `not declared` | stored |
| Survey Invitation Wizard | `survey.invite` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Survey Invitation Wizard | `survey.invite` | `author_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `set null` | stored |
| Survey Invitation Wizard | `survey.invite` | `existing_partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | derived |
| Survey Invitation Wizard | `survey.invite` | `mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Survey Invitation Wizard | `survey.invite` | `partner_ids` | list on both sides | Contact | `res.partner` | `0..n : 0..n` | `not declared` | stored |
| Survey Invitation Wizard | `survey.invite` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 1` | `not declared` | stored |
| Survey Label | `survey.question.answer` | `matrix_question_id` | link to one record | Survey Question | `survey.question` | `n : 0..1` | `cascade` | stored |
| Survey Label | `survey.question.answer` | `question_id` | link to one record | Survey Question | `survey.question` | `n : 0..1` | `cascade` | stored |
| Survey Question | `survey.question` | `allowed_triggering_question_ids` | list on both sides | Survey Question | `survey.question` | `0..n : 0..n` | `not declared` | derived |
| Survey Question | `survey.question` | `matrix_row_ids` | list of records | Survey Label | `survey.question.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey Question | `survey.question` | `page_id` | link to one record | Survey Question | `survey.question` | `n : 0..1` | `not declared` | stored |
| Survey Question | `survey.question` | `question_ids` | list of records | Survey Question | `survey.question` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey Question | `survey.question` | `suggested_answer_ids` | list of records | Survey Label | `survey.question.answer` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey Question | `survey.question` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 0..1` | `cascade` | stored |
| Survey Question | `survey.question` | `triggering_answer_ids` | list on both sides | Survey Label | `survey.question.answer` | `0..n : 0..n` | `not declared` | stored |
| Survey Question | `survey.question` | `triggering_question_ids` | list on both sides | Survey Question | `survey.question` | `0..n : 0..n` | `not declared` | derived |
| Survey Question | `survey.question` | `user_input_line_ids` | list of records | Survey User Input Line | `survey.user_input.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey User Input | `survey.user_input` | `applicant_id` | link to one record | Applicant | `hr.applicant` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `last_displayed_page_id` | link to one record | Survey Question | `survey.question` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `lead_id` | link to one record | Lead | `crm.lead` | `n : 0..1` | `set null` | stored |
| Survey User Input | `survey.user_input` | `partner_id` | link to one record | Contact | `res.partner` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `predefined_question_ids` | list on both sides | Survey Question | `survey.question` | `0..n : 0..n` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `slide_id` | link to one record | Slides | `slide.slide` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `slide_partner_id` | link to one record | Slide / Partner decorated m2m | `slide.slide.partner` | `n : 0..1` | `not declared` | stored |
| Survey User Input | `survey.user_input` | `survey_id` | link to one record | Survey | `survey.survey` | `n : 1` | `cascade` | stored |
| Survey User Input | `survey.user_input` | `user_input_line_ids` | list of records | Survey User Input Line | `survey.user_input.line` | `1 : 0..n` | `mirror of the target column` | derived |
| Survey User Input Line | `survey.user_input.line` | `lang_id` | link to one record | Languages | `res.lang` | `n : 0..1` | `not declared` | derived |
| Survey User Input Line | `survey.user_input.line` | `matrix_row_id` | link to one record | Survey Label | `survey.question.answer` | `n : 0..1` | `not declared` | stored |
| Survey User Input Line | `survey.user_input.line` | `question_id` | link to one record | Survey Question | `survey.question` | `n : 1` | `cascade` | stored |
| Survey User Input Line | `survey.user_input.line` | `suggested_answer_id` | link to one record | Survey Label | `survey.question.answer` | `n : 0..1` | `not declared` | stored |
| Survey User Input Line | `survey.user_input.line` | `user_input_id` | link to one record | Survey User Input | `survey.user_input` | `n : 1` | `cascade` | stored |
| Track Karma Changes | `gamification.karma.tracking` | `user_id` | link to one record | User | `res.users` | `n : 1` | `cascade` | stored |

### 3.46 Marketing and Mass Mailing

Mass mailings, mailing lists, contacts and subscriptions, traces and trace statistics, link tracking, campaign tracking, marketing cards and social links.

Specified in the domain folder `../domains/marketing-and-mass-mailing/`, listed in the domain index [`../domains/README.md`](../domains/README.md).

#### Persistent entities (15)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Link Tracker | `link.tracker` | `link_tracker` | Persistent record with 8 stored columns; owns Link Tracker Click, Link Tracker Code; referenced by 3 relation fields. | Link Tracker |
| Link Tracker Click | `link.tracker.click` | `link_tracker_click` | Persistent record with 6 stored columns; belongs to Link Tracker. | Link Tracker |
| Link Tracker Code | `link.tracker.code` | `link_tracker_code` | Persistent record with 2 stored columns; belongs to Link Tracker. | Link Tracker |
| Mailing Contact | `mailing.contact` | `mailing_contact` | Persistent record with 11 stored columns; owns Mailing List Subscription; referenced by 3 relation fields. | Email Marketing |
| Mailing Favorite Filters | `mailing.filter` | `mailing_filter` | Persistent record with 3 stored columns; belongs to Models; referenced by 1 relation field. | Email Marketing |
| Mailing List | `mailing.list` | `mailing_list` | Persistent record with 3 stored columns; owns Mailing List Subscription; referenced by 9 relation fields. | Email Marketing |
| Mailing List Subscription | `mailing.subscription` | `mailing_subscription` | Persistent record with 5 stored columns; belongs to Mailing Contact, Mailing List. | Email Marketing |
| Mailing Statistics | `mailing.trace` | `mailing_trace` | Persistent record with 20 stored columns; owns Link Tracker Click, Link text message to mailing/text message tracking models; states of `trace_status`: Outgoing, Processing, Sent, Delivered, Opened, Replied, Bounced, Exception, Cancelled; referenced by 2 relation fields. | Email Marketing |
| Mailing Subscription Reason | `mailing.subscription.optout` | `mailing_subscription_optout` | Persistent record with 3 stored columns; referenced by 2 relation fields. | Email Marketing |
| Marketing Card | `card.card` | `card_card` | Persistent record with 5 stored columns; belongs to Marketing Card Campaign; states of `share_status`: Shared, Visited. | Marketing Card |
| Marketing Card Campaign | `card.campaign` | `card_campaign` | Persistent record with 35 stored columns; belongs to Marketing Card Template; owns Marketing Card, Mass Mailing; referenced by 2 relation fields. | Marketing Card |
| Marketing Card Campaign Tag | `card.campaign.tag` | `card_campaign_tag` | Persistent record with 2 stored columns; referenced by 1 relation field. | Marketing Card |
| Marketing Card Template | `card.template` | `card_template` | Persistent record with 6 stored columns; referenced by 1 relation field. | Marketing Card |
| Mass Mailing | `mailing.mailing` | `mailing_mailing` | Persistent record with 36 stored columns; belongs to Models; owns Mailing Statistics; lifecycle states Draft, In Queue, Sending, Sent; referenced by 12 relation fields. | Email Marketing |
| Mass Mailing Statistics | `mailing.trace.report` | `mailing_trace_report` | Persistent record with 17 stored columns; lifecycle states Draft, Tested, Sent. | Email Marketing |

#### Interactive assistant entities (6)

| Entity | Transport name | Table | Purpose | Defining capability package |
|---|---|---|---|---|
| Add Contacts to Mailing List | `mailing.contact.to.list` | `mailing_contact_to_list` | Interactive assistant with 1 stored column; belongs to Mailing List. | Email Marketing |
| Mailing Contact Import | `mailing.contact.import` | `mailing_contact_import` | Interactive assistant with 1 stored column. | Email Marketing |
| Merge Mass Mailing List | `mailing.list.merge` | `mailing_list_merge` | Interactive assistant with 4 stored columns. | Email Marketing |
| Sample Mail Wizard | `mailing.mailing.test` | `mailing_mailing_test` | Interactive assistant with 2 stored columns; belongs to Mass Mailing. | Email Marketing |
| schedule a mailing | `mailing.mailing.schedule.date` | `mailing_mailing_schedule_date` | Interactive assistant with 2 stored columns; belongs to Mass Mailing. | Email Marketing |
| Test text message Mailing | `mailing.sms.test` | `mailing_sms_test` | Interactive assistant with 2 stored columns; belongs to Mass Mailing. | text message Marketing |

#### Relationships (52)

| From entity | Transport name | Relation field | Kind | To entity | Transport name | Cardinality | On delete | Storage |
|---|---|---|---|---|---|---|---|---|
| Add Contacts to Mailing List | `mailing.contact.to.list` | `contact_ids` | list on both sides | Mailing Contact | `mailing.contact` | `0..n : 0..n` | `not declared` | stored |
| Add Contacts to Mailing List | `mailing.contact.to.list` | `mailing_list_id` | link to one record | Mailing List | `mailing.list` | `n : 1` | `not declared` | stored |
| Link Tracker | `link.tracker` | `link_click_ids` | list of records | Link Tracker Click | `link.tracker.click` | `1 : 0..n` | `mirror of the target column` | derived |
| Link Tracker | `link.tracker` | `link_code_ids` | list of records | Link Tracker Code | `link.tracker.code` | `1 : 0..n` | `mirror of the target column` | derived |
| Link Tracker | `link.tracker` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| Link Tracker Click | `link.tracker.click` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `set null` | stored |
| Link Tracker Click | `link.tracker.click` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Link Tracker Click | `link.tracker.click` | `link_id` | link to one record | Link Tracker | `link.tracker` | `n : 1` | `cascade` | stored |
| Link Tracker Click | `link.tracker.click` | `mailing_trace_id` | link to one record | Mailing Statistics | `mailing.trace` | `n : 0..1` | `not declared` | stored |
| Link Tracker Click | `link.tracker.click` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `not declared` | stored |
| Link Tracker Code | `link.tracker.code` | `link_id` | link to one record | Link Tracker | `link.tracker` | `n : 1` | `cascade` | stored |
| Mailing Contact | `mailing.contact` | `country_id` | link to one record | Country | `res.country` | `n : 0..1` | `not declared` | stored |
| Mailing Contact | `mailing.contact` | `list_ids` | list on both sides | Mailing List | `mailing.list` | `0..n : 0..n` | `not declared` | stored |
| Mailing Contact | `mailing.contact` | `subscription_ids` | list of records | Mailing List Subscription | `mailing.subscription` | `1 : 0..n` | `mirror of the target column` | derived |
| Mailing Contact | `mailing.contact` | `tag_ids` | list on both sides | Partner Tags | `res.partner.category` | `0..n : 0..n` | `not declared` | stored |
| Mailing Contact Import | `mailing.contact.import` | `mailing_list_ids` | list on both sides | Mailing List | `mailing.list` | `0..n : 0..n` | `not declared` | stored |
| Mailing Favorite Filters | `mailing.filter` | `create_uid` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Mailing Favorite Filters | `mailing.filter` | `mailing_model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Mailing List | `mailing.list` | `contact_ids` | list on both sides | Mailing Contact | `mailing.contact` | `0..n : 0..n` | `not declared` | stored |
| Mailing List | `mailing.list` | `mailing_ids` | list on both sides | Mass Mailing | `mailing.mailing` | `0..n : 0..n` | `not declared` | stored |
| Mailing List | `mailing.list` | `subscription_ids` | list of records | Mailing List Subscription | `mailing.subscription` | `1 : 0..n` | `mirror of the target column` | derived |
| Mailing List Subscription | `mailing.subscription` | `contact_id` | link to one record | Mailing Contact | `mailing.contact` | `n : 1` | `cascade` | stored |
| Mailing List Subscription | `mailing.subscription` | `list_id` | link to one record | Mailing List | `mailing.list` | `n : 1` | `cascade` | stored |
| Mailing List Subscription | `mailing.subscription` | `opt_out_reason_id` | link to one record | Mailing Subscription Reason | `mailing.subscription.optout` | `n : 0..1` | `restrict` | stored |
| Mailing Statistics | `mailing.trace` | `links_click_ids` | list of records | Link Tracker Click | `link.tracker.click` | `1 : 0..n` | `mirror of the target column` | derived |
| Mailing Statistics | `mailing.trace` | `mail_mail_id` | link to one record | Outgoing Mails | `mail.mail` | `n : 0..1` | `not declared` | stored |
| Mailing Statistics | `mailing.trace` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 0..1` | `cascade` | stored |
| Mailing Statistics | `mailing.trace` | `sms_id` | link to one record | Outgoing text message | `sms.sms` | `n : 0..1` | `not declared` | derived |
| Mailing Statistics | `mailing.trace` | `sms_tracker_ids` | list of records | Link text message to mailing/text message tracking models | `sms.tracker` | `1 : 0..n` | `mirror of the target column` | derived |
| Marketing Card | `card.card` | `campaign_id` | link to one record | Marketing Card Campaign | `card.campaign` | `n : 1` | `cascade` | stored |
| Marketing Card Campaign | `card.campaign` | `card_ids` | list of records | Marketing Card | `card.card` | `1 : 0..n` | `mirror of the target column` | derived |
| Marketing Card Campaign | `card.campaign` | `card_template_id` | link to one record | Marketing Card Template | `card.template` | `n : 1` | `not declared` | stored |
| Marketing Card Campaign | `card.campaign` | `link_tracker_id` | link to one record | Link Tracker | `link.tracker` | `n : 0..1` | `restrict` | stored |
| Marketing Card Campaign | `card.campaign` | `mailing_ids` | list of records | Mass Mailing | `mailing.mailing` | `1 : 0..n` | `mirror of the target column` | derived |
| Marketing Card Campaign | `card.campaign` | `tag_ids` | list on both sides | Marketing Card Campaign Tag | `card.campaign.tag` | `0..n : 0..n` | `not declared` | stored |
| Marketing Card Campaign | `card.campaign` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `attachment_ids` | list on both sides | Attachment | `ir.attachment` | `0..n : 0..n` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `campaign_id` | link to one record | campaign tracking parameter Campaign | `utm.campaign` | `n : 0..1` | `set null` | stored |
| Mass Mailing | `mailing.mailing` | `card_campaign_id` | link to one record | Marketing Card Campaign | `card.campaign` | `n : 0..1` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `contact_list_ids` | list on both sides | Mailing List | `mailing.list` | `0..n : 0..n` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `mail_server_id` | link to one record | Mail Server | `ir.mail_server` | `n : 0..1` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `mailing_filter_id` | link to one record | Mailing Favorite Filters | `mailing.filter` | `n : 0..1` | `not declared` | stored |
| Mass Mailing | `mailing.mailing` | `mailing_model_id` | link to one record | Models | `ir.model` | `n : 1` | `cascade` | stored |
| Mass Mailing | `mailing.mailing` | `mailing_trace_ids` | list of records | Mailing Statistics | `mailing.trace` | `1 : 0..n` | `mirror of the target column` | derived |
| Mass Mailing | `mailing.mailing` | `medium_id` | link to one record | campaign tracking parameter Medium | `utm.medium` | `n : 0..1` | `restrict` | stored |
| Mass Mailing | `mailing.mailing` | `sms_template_id` | link to one record | text message Templates | `sms.template` | `n : 0..1` | `set null` | stored |
| Mass Mailing | `mailing.mailing` | `user_id` | link to one record | User | `res.users` | `n : 0..1` | `not declared` | stored |
| Merge Mass Mailing List | `mailing.list.merge` | `dest_list_id` | link to one record | Mailing List | `mailing.list` | `n : 0..1` | `not declared` | stored |
| Merge Mass Mailing List | `mailing.list.merge` | `src_list_ids` | list on both sides | Mailing List | `mailing.list` | `0..n : 0..n` | `not declared` | stored |
| Sample Mail Wizard | `mailing.mailing.test` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 1` | `cascade` | stored |
| Test text message Mailing | `mailing.sms.test` | `mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 1` | `cascade` | stored |
| schedule a mailing | `mailing.mailing.schedule.date` | `mass_mailing_id` | link to one record | Mass Mailing | `mailing.mailing` | `n : 1` | `not declared` | stored |
## 4. Cross-domain relationship map

Each row counts the materialized relation fields declared by entities of the source domain that point at entities of the target domain. Lists of records are excluded, because they are the mirror of a link already counted on the other side. A non-zero row means the source domain cannot be built without the target domain, because its records carry links that must resolve.

### 4.1 Outgoing references per domain

| Source domain | Target domain | Relation fields |
|---|---|---|
| Automation and Integration | Contacts and Organizations | 5 |
| Automation and Integration | Identity and Access | 4 |
| Automation and Integration | Platform Foundation | 3 |
| Identity and Access | Human Resources Core | 9 |
| Identity and Access | Contacts and Organizations | 7 |
| Identity and Access | Platform Foundation | 7 |
| Identity and Access | Messaging and Activities | 4 |
| Identity and Access | Learning, Surveys and Gamification | 2 |
| Identity and Access | Lunch Ordering | 2 |
| Identity and Access | Sales | 2 |
| Identity and Access | Attendances and Working Time | 1 |
| Identity and Access | Inventory Operations | 1 |
| Identity and Access | Projects and Tasks | 1 |
| Identity and Access | Website and Storefront | 1 |
| Platform Foundation | General Ledger | 25 |
| Platform Foundation | Identity and Access | 21 |
| Platform Foundation | Contacts and Organizations | 13 |
| Platform Foundation | Products and Catalog | 12 |
| Platform Foundation | Point of Sale | 10 |
| Platform Foundation | Messaging and Activities | 9 |
| Platform Foundation | Website and Storefront | 9 |
| Platform Foundation | Projects and Tasks | 8 |
| Platform Foundation | Attendances and Working Time | 2 |
| Platform Foundation | Inventory Operations | 2 |
| Platform Foundation | Multi-Currency | 2 |
| Platform Foundation | Sales | 2 |
| Platform Foundation | Customer Relationship Management | 1 |
| Platform Foundation | Electronic Invoicing and Document Exchange | 1 |
| Platform Foundation | Fiscal Localizations | 1 |
| Platform Foundation | Human Resources Core | 1 |
| Platform Foundation | Manufacturing | 1 |
| Platform Foundation | Payment Providers | 1 |
| Platform Foundation | Time Off | 1 |
| Platform Foundation | Units of Measure and Packaging | 1 |
| Calendar and Scheduling | Contacts and Organizations | 7 |
| Calendar and Scheduling | Identity and Access | 4 |
| Calendar and Scheduling | Messaging and Activities | 3 |
| Calendar and Scheduling | Platform Foundation | 2 |
| Calendar and Scheduling | Customer Relationship Management | 1 |
| Calendar and Scheduling | Recruitment | 1 |
| Contacts and Organizations | General Ledger | 68 |
| Contacts and Organizations | Fiscal Localizations | 12 |
| Contacts and Organizations | Platform Foundation | 11 |
| Contacts and Organizations | Electronic Invoicing and Document Exchange | 8 |
| Contacts and Organizations | Inventory Operations | 7 |
| Contacts and Organizations | Multi-Currency | 5 |
| Contacts and Organizations | Products and Catalog | 5 |
| Contacts and Organizations | Identity and Access | 4 |
| Contacts and Organizations | Messaging and Activities | 4 |
| Contacts and Organizations | Customer Relationship Management | 2 |
| Contacts and Organizations | Manufacturing | 2 |
| Contacts and Organizations | Projects and Tasks | 2 |
| Contacts and Organizations | Units of Measure and Packaging | 2 |
| Contacts and Organizations | Website and Storefront | 2 |
| Contacts and Organizations | Attendances and Working Time | 1 |
| Contacts and Organizations | Calendar and Scheduling | 1 |
| Contacts and Organizations | Delivery and Shipping | 1 |
| Contacts and Organizations | Human Resources Core | 1 |
| Contacts and Organizations | Learning, Surveys and Gamification | 1 |
| Contacts and Organizations | Sales | 1 |
| Contacts and Organizations | Time Off | 1 |
| Messaging and Activities | Contacts and Organizations | 57 |
| Messaging and Activities | Platform Foundation | 28 |
| Messaging and Activities | Identity and Access | 26 |
| Messaging and Activities | Marketing and Mass Mailing | 7 |
| Messaging and Activities | Automation and Integration | 3 |
| Messaging and Activities | General Ledger | 3 |
| Messaging and Activities | Human Resources Core | 3 |
| Messaging and Activities | Calendar and Scheduling | 2 |
| Messaging and Activities | Customer Relationship Management | 2 |
| Messaging and Activities | Projects and Tasks | 2 |
| Messaging and Activities | Multi-Currency | 1 |
| Messaging and Activities | Sales | 1 |
| Messaging and Activities | Website and Storefront | 1 |
| Multi-Currency | Contacts and Organizations | 1 |
| Spreadsheets and Dashboards | Identity and Access | 2 |
| Spreadsheets and Dashboards | Contacts and Organizations | 1 |
| Spreadsheets and Dashboards | Platform Foundation | 1 |
| Accounts Receivable | General Ledger | 3 |
| Analytic Accounting | Contacts and Organizations | 10 |
| Analytic Accounting | General Ledger | 4 |
| Analytic Accounting | Products and Catalog | 4 |
| Analytic Accounting | Projects and Tasks | 4 |
| Analytic Accounting | Human Resources Core | 3 |
| Analytic Accounting | Manufacturing | 3 |
| Analytic Accounting | Units of Measure and Packaging | 2 |
| Analytic Accounting | Attendances and Working Time | 1 |
| Analytic Accounting | Identity and Access | 1 |
| Analytic Accounting | Sales | 1 |
| Analytic Accounting | Time Off | 1 |
| Electronic Invoicing and Document Exchange | Contacts and Organizations | 7 |
| Electronic Invoicing and Document Exchange | General Ledger | 3 |
| Electronic Invoicing and Document Exchange | Platform Foundation | 1 |
| Fiscal Localizations | Contacts and Organizations | 31 |
| Fiscal Localizations | General Ledger | 30 |
| Fiscal Localizations | Platform Foundation | 7 |
| Fiscal Localizations | Multi-Currency | 5 |
| Fiscal Localizations | Point of Sale | 4 |
| Fiscal Localizations | Electronic Invoicing and Document Exchange | 3 |
| Fiscal Localizations | Inventory Operations | 3 |
| Fiscal Localizations | Identity and Access | 1 |
| Fiscal Localizations | Products and Catalog | 1 |
| General Ledger | Contacts and Organizations | 75 |
| General Ledger | Fiscal Localizations | 31 |
| General Ledger | Platform Foundation | 22 |
| General Ledger | Multi-Currency | 21 |
| General Ledger | Payment Providers | 8 |
| General Ledger | Point of Sale | 6 |
| General Ledger | Purchasing | 5 |
| General Ledger | Electronic Invoicing and Document Exchange | 4 |
| General Ledger | Units of Measure and Packaging | 4 |
| General Ledger | Identity and Access | 3 |
| General Ledger | Products and Catalog | 3 |
| General Ledger | Sales | 3 |
| General Ledger | Expenses | 2 |
| General Ledger | Payments and Bank Reconciliation | 2 |
| General Ledger | Fleet | 1 |
| General Ledger | Human Resources Core | 1 |
| General Ledger | Inventory Operations | 1 |
| General Ledger | Manufacturing | 1 |
| General Ledger | Messaging and Activities | 1 |
| General Ledger | Website and Storefront | 1 |
| Payments and Bank Reconciliation | General Ledger | 7 |
| Payments and Bank Reconciliation | Contacts and Organizations | 3 |
| Taxes | General Ledger | 5 |
| Taxes | Contacts and Organizations | 2 |
| Taxes | Multi-Currency | 2 |
| Delivery and Shipping | Contacts and Organizations | 6 |
| Delivery and Shipping | Products and Catalog | 3 |
| Delivery and Shipping | Inventory Operations | 2 |
| Delivery and Shipping | Multi-Currency | 1 |
| Delivery and Shipping | Sales | 1 |
| Inventory Operations | Contacts and Organizations | 40 |
| Inventory Operations | Products and Catalog | 30 |
| Inventory Operations | Manufacturing | 22 |
| Inventory Operations | Units of Measure and Packaging | 15 |
| Inventory Operations | Identity and Access | 7 |
| Inventory Operations | Purchasing | 5 |
| Inventory Operations | Platform Foundation | 4 |
| Inventory Operations | Sales | 4 |
| Inventory Operations | Fiscal Localizations | 3 |
| Inventory Operations | General Ledger | 3 |
| Inventory Operations | Multi-Currency | 3 |
| Inventory Operations | Point of Sale | 3 |
| Inventory Operations | Delivery and Shipping | 2 |
| Inventory Operations | Fleet | 2 |
| Inventory Operations | Repair and Maintenance | 2 |
| Inventory Operations | Analytic Accounting | 1 |
| Inventory Operations | Attendances and Working Time | 1 |
| Inventory Operations | Projects and Tasks | 1 |
| Inventory Operations | Website and Storefront | 1 |
| Inventory Valuation and Costing | Multi-Currency | 5 |
| Inventory Valuation and Costing | General Ledger | 4 |
| Inventory Valuation and Costing | Inventory Operations | 4 |
| Inventory Valuation and Costing | Products and Catalog | 4 |
| Inventory Valuation and Costing | Contacts and Organizations | 3 |
| Inventory Valuation and Costing | Identity and Access | 2 |
| Inventory Valuation and Costing | Manufacturing | 1 |
| Manufacturing | Products and Catalog | 19 |
| Manufacturing | Inventory Operations | 15 |
| Manufacturing | Units of Measure and Packaging | 8 |
| Manufacturing | Contacts and Organizations | 7 |
| Manufacturing | General Ledger | 5 |
| Manufacturing | Identity and Access | 4 |
| Manufacturing | Analytic Accounting | 3 |
| Manufacturing | Multi-Currency | 2 |
| Manufacturing | Projects and Tasks | 2 |
| Manufacturing | Attendances and Working Time | 1 |
| Manufacturing | Sales | 1 |
| Products and Catalog | General Ledger | 14 |
| Products and Catalog | Inventory Operations | 12 |
| Products and Catalog | Contacts and Organizations | 10 |
| Products and Catalog | Multi-Currency | 8 |
| Products and Catalog | Website and Storefront | 6 |
| Products and Catalog | Units of Measure and Packaging | 5 |
| Products and Catalog | Fiscal Localizations | 3 |
| Products and Catalog | Projects and Tasks | 3 |
| Products and Catalog | Manufacturing | 2 |
| Products and Catalog | Platform Foundation | 2 |
| Products and Catalog | Point of Sale | 2 |
| Products and Catalog | Purchasing | 2 |
| Products and Catalog | Sales | 2 |
| Products and Catalog | Customer Relationship Management | 1 |
| Products and Catalog | Identity and Access | 1 |
| Products and Catalog | Learning, Surveys and Gamification | 1 |
| Products and Catalog | Messaging and Activities | 1 |
| Purchasing | Contacts and Organizations | 19 |
| Purchasing | Products and Catalog | 12 |
| Purchasing | Inventory Operations | 10 |
| Purchasing | General Ledger | 9 |
| Purchasing | Units of Measure and Packaging | 6 |
| Purchasing | Multi-Currency | 5 |
| Purchasing | Identity and Access | 3 |
| Purchasing | Projects and Tasks | 1 |
| Purchasing | Sales | 1 |
| Repair and Maintenance | Inventory Operations | 12 |
| Repair and Maintenance | Contacts and Organizations | 7 |
| Repair and Maintenance | Identity and Access | 7 |
| Repair and Maintenance | Human Resources Core | 3 |
| Repair and Maintenance | Sales | 2 |
| Repair and Maintenance | Units of Measure and Packaging | 2 |
| Repair and Maintenance | Products and Catalog | 1 |
| Replenishment and Procurement | Products and Catalog | 2 |
| Replenishment and Procurement | Contacts and Organizations | 1 |
| Units of Measure and Packaging | Fiscal Localizations | 2 |
| Units of Measure and Packaging | Inventory Operations | 1 |
| Customer Relationship Management | Contacts and Organizations | 26 |
| Customer Relationship Management | Sales | 13 |
| Customer Relationship Management | Identity and Access | 10 |
| Customer Relationship Management | Events | 5 |
| Customer Relationship Management | Messaging and Activities | 3 |
| Customer Relationship Management | Multi-Currency | 2 |
| Customer Relationship Management | Website and Storefront | 2 |
| Customer Relationship Management | Learning, Surveys and Gamification | 1 |
| Customer Relationship Management | Marketing and Mass Mailing | 1 |
| Customer Relationship Management | Platform Foundation | 1 |
| Customer Relationship Management | Products and Catalog | 1 |
| Loyalty and Promotions | Products and Catalog | 15 |
| Loyalty and Promotions | Contacts and Organizations | 6 |
| Loyalty and Promotions | Sales | 4 |
| Loyalty and Promotions | Messaging and Activities | 2 |
| Loyalty and Promotions | Platform Foundation | 2 |
| Loyalty and Promotions | Point of Sale | 2 |
| Loyalty and Promotions | Website and Storefront | 2 |
| Loyalty and Promotions | Multi-Currency | 1 |
| Loyalty and Promotions | Units of Measure and Packaging | 1 |
| Payment Providers | Contacts and Organizations | 11 |
| Payment Providers | Platform Foundation | 5 |
| Payment Providers | Multi-Currency | 4 |
| Payment Providers | General Ledger | 3 |
| Payment Providers | Fiscal Localizations | 1 |
| Payment Providers | Point of Sale | 1 |
| Payment Providers | Sales | 1 |
| Payment Providers | Website and Storefront | 1 |
| Point of Sale | General Ledger | 21 |
| Point of Sale | Products and Catalog | 13 |
| Point of Sale | Contacts and Organizations | 12 |
| Point of Sale | Human Resources Core | 7 |
| Point of Sale | Identity and Access | 7 |
| Point of Sale | Platform Foundation | 7 |
| Point of Sale | Inventory Operations | 5 |
| Point of Sale | Multi-Currency | 5 |
| Point of Sale | Sales | 5 |
| Point of Sale | Fiscal Localizations | 4 |
| Point of Sale | Loyalty and Promotions | 2 |
| Point of Sale | Messaging and Activities | 2 |
| Point of Sale | Payment Providers | 2 |
| Point of Sale | Attendances and Working Time | 1 |
| Point of Sale | Events | 1 |
| Point of Sale | Units of Measure and Packaging | 1 |
| Sales | Contacts and Organizations | 19 |
| Sales | Products and Catalog | 15 |
| Sales | General Ledger | 9 |
| Sales | Identity and Access | 7 |
| Sales | Customer Relationship Management | 6 |
| Sales | Projects and Tasks | 6 |
| Sales | Units of Measure and Packaging | 6 |
| Sales | Events | 5 |
| Sales | Inventory Operations | 5 |
| Sales | Loyalty and Promotions | 5 |
| Sales | Multi-Currency | 4 |
| Sales | Fiscal Localizations | 2 |
| Sales | Messaging and Activities | 2 |
| Sales | Payment Providers | 2 |
| Sales | Website and Storefront | 2 |
| Sales | Analytic Accounting | 1 |
| Sales | Delivery and Shipping | 1 |
| Sales | Expenses | 1 |
| Sales | Manufacturing | 1 |
| Sales | Platform Foundation | 1 |
| Website and Storefront | Identity and Access | 17 |
| Website and Storefront | Contacts and Organizations | 14 |
| Website and Storefront | Platform Foundation | 9 |
| Website and Storefront | Products and Catalog | 8 |
| Website and Storefront | Messaging and Activities | 3 |
| Website and Storefront | Events | 2 |
| Website and Storefront | Multi-Currency | 2 |
| Website and Storefront | Sales | 2 |
| Website and Storefront | Customer Relationship Management | 1 |
| Website and Storefront | Delivery and Shipping | 1 |
| Website and Storefront | Inventory Operations | 1 |
| Website and Storefront | Learning, Surveys and Gamification | 1 |
| Website and Storefront | Marketing and Mass Mailing | 1 |
| Projects and Tasks | Contacts and Organizations | 13 |
| Projects and Tasks | Identity and Access | 9 |
| Projects and Tasks | Messaging and Activities | 7 |
| Projects and Tasks | Sales | 7 |
| Projects and Tasks | Platform Foundation | 3 |
| Projects and Tasks | Units of Measure and Packaging | 2 |
| Projects and Tasks | Analytic Accounting | 1 |
| Projects and Tasks | Attendances and Working Time | 1 |
| Projects and Tasks | Multi-Currency | 1 |
| Projects and Tasks | Products and Catalog | 1 |
| Timesheets | Human Resources Core | 6 |
| Timesheets | Projects and Tasks | 5 |
| Timesheets | Contacts and Organizations | 4 |
| Timesheets | Multi-Currency | 3 |
| Timesheets | Sales | 3 |
| Timesheets | Identity and Access | 2 |
| Timesheets | General Ledger | 1 |
| Attendances and Working Time | Contacts and Organizations | 7 |
| Attendances and Working Time | Human Resources Core | 6 |
| Attendances and Working Time | Identity and Access | 2 |
| Attendances and Working Time | Work Entries | 2 |
| Attendances and Working Time | Time Off | 1 |
| Expenses | General Ledger | 8 |
| Expenses | Contacts and Organizations | 4 |
| Expenses | Multi-Currency | 4 |
| Expenses | Human Resources Core | 3 |
| Expenses | Sales | 3 |
| Expenses | Identity and Access | 2 |
| Expenses | Products and Catalog | 2 |
| Expenses | Units of Measure and Packaging | 1 |
| Fleet | Contacts and Organizations | 15 |
| Fleet | Human Resources Core | 4 |
| Fleet | Identity and Access | 3 |
| Fleet | Multi-Currency | 3 |
| Fleet | General Ledger | 1 |
| Fleet | Platform Foundation | 1 |
| Human Resources Core | Contacts and Organizations | 28 |
| Human Resources Core | Identity and Access | 14 |
| Human Resources Core | Attendances and Working Time | 7 |
| Human Resources Core | Learning, Surveys and Gamification | 4 |
| Human Resources Core | Recruitment | 2 |
| Human Resources Core | Events | 1 |
| Human Resources Core | Multi-Currency | 1 |
| Human Resources Core | Time Off | 1 |
| Lunch Ordering | Contacts and Organizations | 9 |
| Lunch Ordering | Identity and Access | 5 |
| Lunch Ordering | Multi-Currency | 5 |
| Lunch Ordering | Platform Foundation | 2 |
| Recruitment | Human Resources Core | 9 |
| Recruitment | Contacts and Organizations | 4 |
| Recruitment | Messaging and Activities | 4 |
| Recruitment | Identity and Access | 3 |
| Recruitment | Customer Relationship Management | 2 |
| Recruitment | Platform Foundation | 2 |
| Recruitment | Learning, Surveys and Gamification | 1 |
| Time Off | Human Resources Core | 25 |
| Time Off | Contacts and Organizations | 11 |
| Time Off | Attendances and Working Time | 4 |
| Time Off | Identity and Access | 3 |
| Time Off | Messaging and Activities | 2 |
| Time Off | Platform Foundation | 2 |
| Time Off | Calendar and Scheduling | 1 |
| Time Off | Work Entries | 1 |
| Work Entries | Human Resources Core | 6 |
| Work Entries | Contacts and Organizations | 3 |
| Work Entries | Identity and Access | 1 |
| Work Entries | Time Off | 1 |
| Events | Contacts and Organizations | 19 |
| Events | Sales | 10 |
| Events | Products and Catalog | 5 |
| Events | Website and Storefront | 5 |
| Events | Customer Relationship Management | 4 |
| Events | Identity and Access | 3 |
| Events | Messaging and Activities | 1 |
| Events | Multi-Currency | 1 |
| Events | Platform Foundation | 1 |
| Events | Point of Sale | 1 |
| Learning, Surveys and Gamification | Identity and Access | 17 |
| Learning, Surveys and Gamification | Contacts and Organizations | 14 |
| Learning, Surveys and Gamification | Platform Foundation | 9 |
| Learning, Surveys and Gamification | Messaging and Activities | 7 |
| Learning, Surveys and Gamification | Human Resources Core | 2 |
| Learning, Surveys and Gamification | Recruitment | 2 |
| Learning, Surveys and Gamification | Website and Storefront | 2 |
| Learning, Surveys and Gamification | Customer Relationship Management | 1 |
| Learning, Surveys and Gamification | Products and Catalog | 1 |
| Learning, Surveys and Gamification | Sales | 1 |
| Marketing and Mass Mailing | Platform Foundation | 4 |
| Marketing and Mass Mailing | Contacts and Organizations | 3 |
| Marketing and Mass Mailing | Customer Relationship Management | 3 |
| Marketing and Mass Mailing | Identity and Access | 3 |
| Marketing and Mass Mailing | Messaging and Activities | 3 |

### 4.2 Dependency summary per domain

| Domain | Depends on | Referenced by |
|---|---|---|
| Automation and Integration | Contacts and Organizations (5); Identity and Access (4); Platform Foundation (3) | Messaging and Activities (3) |
| Identity and Access | Attendances and Working Time (1); Contacts and Organizations (7); Human Resources Core (9); Inventory Operations (1); Learning, Surveys and Gamification (2); Lunch Ordering (2); Messaging and Activities (4); Platform Foundation (7); Projects and Tasks (1); Sales (2); Website and Storefront (1) | Analytic Accounting (1); Attendances and Working Time (2); Automation and Integration (4); Calendar and Scheduling (4); Contacts and Organizations (4); Customer Relationship Management (10); Events (3); Expenses (2); Fiscal Localizations (1); Fleet (3); General Ledger (3); Human Resources Core (14); Inventory Operations (7); Inventory Valuation and Costing (2); Learning, Surveys and Gamification (17); Lunch Ordering (5); Manufacturing (4); Marketing and Mass Mailing (3); Messaging and Activities (26); Platform Foundation (21); Point of Sale (7); Products and Catalog (1); Projects and Tasks (9); Purchasing (3); Recruitment (3); Repair and Maintenance (7); Sales (7); Spreadsheets and Dashboards (2); Time Off (3); Timesheets (2); Website and Storefront (17); Work Entries (1) |
| Platform Foundation | Attendances and Working Time (2); Contacts and Organizations (13); Customer Relationship Management (1); Electronic Invoicing and Document Exchange (1); Fiscal Localizations (1); General Ledger (25); Human Resources Core (1); Identity and Access (21); Inventory Operations (2); Manufacturing (1); Messaging and Activities (9); Multi-Currency (2); Payment Providers (1); Point of Sale (10); Products and Catalog (12); Projects and Tasks (8); Sales (2); Time Off (1); Units of Measure and Packaging (1); Website and Storefront (9) | Automation and Integration (3); Calendar and Scheduling (2); Contacts and Organizations (11); Customer Relationship Management (1); Electronic Invoicing and Document Exchange (1); Events (1); Fiscal Localizations (7); Fleet (1); General Ledger (22); Identity and Access (7); Inventory Operations (4); Learning, Surveys and Gamification (9); Loyalty and Promotions (2); Lunch Ordering (2); Marketing and Mass Mailing (4); Messaging and Activities (28); Payment Providers (5); Point of Sale (7); Products and Catalog (2); Projects and Tasks (3); Recruitment (2); Sales (1); Spreadsheets and Dashboards (1); Time Off (2); Website and Storefront (9) |
| Calendar and Scheduling | Contacts and Organizations (7); Customer Relationship Management (1); Identity and Access (4); Messaging and Activities (3); Platform Foundation (2); Recruitment (1) | Contacts and Organizations (1); Messaging and Activities (2); Time Off (1) |
| Contacts and Organizations | Attendances and Working Time (1); Calendar and Scheduling (1); Customer Relationship Management (2); Delivery and Shipping (1); Electronic Invoicing and Document Exchange (8); Fiscal Localizations (12); General Ledger (68); Human Resources Core (1); Identity and Access (4); Inventory Operations (7); Learning, Surveys and Gamification (1); Manufacturing (2); Messaging and Activities (4); Multi-Currency (5); Platform Foundation (11); Products and Catalog (5); Projects and Tasks (2); Sales (1); Time Off (1); Units of Measure and Packaging (2); Website and Storefront (2) | Analytic Accounting (10); Attendances and Working Time (7); Automation and Integration (5); Calendar and Scheduling (7); Customer Relationship Management (26); Delivery and Shipping (6); Electronic Invoicing and Document Exchange (7); Events (19); Expenses (4); Fiscal Localizations (31); Fleet (15); General Ledger (75); Human Resources Core (28); Identity and Access (7); Inventory Operations (40); Inventory Valuation and Costing (3); Learning, Surveys and Gamification (14); Loyalty and Promotions (6); Lunch Ordering (9); Manufacturing (7); Marketing and Mass Mailing (3); Messaging and Activities (57); Multi-Currency (1); Payment Providers (11); Payments and Bank Reconciliation (3); Platform Foundation (13); Point of Sale (12); Products and Catalog (10); Projects and Tasks (13); Purchasing (19); Recruitment (4); Repair and Maintenance (7); Replenishment and Procurement (1); Sales (19); Spreadsheets and Dashboards (1); Taxes (2); Time Off (11); Timesheets (4); Website and Storefront (14); Work Entries (3) |
| Messaging and Activities | Automation and Integration (3); Calendar and Scheduling (2); Contacts and Organizations (57); Customer Relationship Management (2); General Ledger (3); Human Resources Core (3); Identity and Access (26); Marketing and Mass Mailing (7); Multi-Currency (1); Platform Foundation (28); Projects and Tasks (2); Sales (1); Website and Storefront (1) | Calendar and Scheduling (3); Contacts and Organizations (4); Customer Relationship Management (3); Events (1); General Ledger (1); Identity and Access (4); Learning, Surveys and Gamification (7); Loyalty and Promotions (2); Marketing and Mass Mailing (3); Platform Foundation (9); Point of Sale (2); Products and Catalog (1); Projects and Tasks (7); Recruitment (4); Sales (2); Time Off (2); Website and Storefront (3) |
| Multi-Currency | Contacts and Organizations (1) | Contacts and Organizations (5); Customer Relationship Management (2); Delivery and Shipping (1); Events (1); Expenses (4); Fiscal Localizations (5); Fleet (3); General Ledger (21); Human Resources Core (1); Inventory Operations (3); Inventory Valuation and Costing (5); Loyalty and Promotions (1); Lunch Ordering (5); Manufacturing (2); Messaging and Activities (1); Payment Providers (4); Platform Foundation (2); Point of Sale (5); Products and Catalog (8); Projects and Tasks (1); Purchasing (5); Sales (4); Taxes (2); Timesheets (3); Website and Storefront (2) |
| Spreadsheets and Dashboards | Contacts and Organizations (1); Identity and Access (2); Platform Foundation (1) | nothing |
| Accounts Payable | nothing | nothing |
| Accounts Receivable | General Ledger (3) | nothing |
| Analytic Accounting | Attendances and Working Time (1); Contacts and Organizations (10); General Ledger (4); Human Resources Core (3); Identity and Access (1); Manufacturing (3); Products and Catalog (4); Projects and Tasks (4); Sales (1); Time Off (1); Units of Measure and Packaging (2) | Inventory Operations (1); Manufacturing (3); Projects and Tasks (1); Sales (1) |
| Electronic Invoicing and Document Exchange | Contacts and Organizations (7); General Ledger (3); Platform Foundation (1) | Contacts and Organizations (8); Fiscal Localizations (3); General Ledger (4); Platform Foundation (1) |
| Financial Reporting | nothing | nothing |
| Fiscal Localizations | Contacts and Organizations (31); Electronic Invoicing and Document Exchange (3); General Ledger (30); Identity and Access (1); Inventory Operations (3); Multi-Currency (5); Platform Foundation (7); Point of Sale (4); Products and Catalog (1) | Contacts and Organizations (12); General Ledger (31); Inventory Operations (3); Payment Providers (1); Platform Foundation (1); Point of Sale (4); Products and Catalog (3); Sales (2); Units of Measure and Packaging (2) |
| General Ledger | Contacts and Organizations (75); Electronic Invoicing and Document Exchange (4); Expenses (2); Fiscal Localizations (31); Fleet (1); Human Resources Core (1); Identity and Access (3); Inventory Operations (1); Manufacturing (1); Messaging and Activities (1); Multi-Currency (21); Payment Providers (8); Payments and Bank Reconciliation (2); Platform Foundation (22); Point of Sale (6); Products and Catalog (3); Purchasing (5); Sales (3); Units of Measure and Packaging (4); Website and Storefront (1) | Accounts Receivable (3); Analytic Accounting (4); Contacts and Organizations (68); Electronic Invoicing and Document Exchange (3); Expenses (8); Fiscal Localizations (30); Fleet (1); Inventory Operations (3); Inventory Valuation and Costing (4); Manufacturing (5); Messaging and Activities (3); Payment Providers (3); Payments and Bank Reconciliation (7); Platform Foundation (25); Point of Sale (21); Products and Catalog (14); Purchasing (9); Sales (9); Taxes (5); Timesheets (1) |
| Payments and Bank Reconciliation | Contacts and Organizations (3); General Ledger (7) | General Ledger (2) |
| Taxes | Contacts and Organizations (2); General Ledger (5); Multi-Currency (2) | nothing |
| Delivery and Shipping | Contacts and Organizations (6); Inventory Operations (2); Multi-Currency (1); Products and Catalog (3); Sales (1) | Contacts and Organizations (1); Inventory Operations (2); Sales (1); Website and Storefront (1) |
| Inventory Operations | Analytic Accounting (1); Attendances and Working Time (1); Contacts and Organizations (40); Delivery and Shipping (2); Fiscal Localizations (3); Fleet (2); General Ledger (3); Identity and Access (7); Manufacturing (22); Multi-Currency (3); Platform Foundation (4); Point of Sale (3); Products and Catalog (30); Projects and Tasks (1); Purchasing (5); Repair and Maintenance (2); Sales (4); Units of Measure and Packaging (15); Website and Storefront (1) | Contacts and Organizations (7); Delivery and Shipping (2); Fiscal Localizations (3); General Ledger (1); Identity and Access (1); Inventory Valuation and Costing (4); Manufacturing (15); Platform Foundation (2); Point of Sale (5); Products and Catalog (12); Purchasing (10); Repair and Maintenance (12); Sales (5); Units of Measure and Packaging (1); Website and Storefront (1) |
| Inventory Valuation and Costing | Contacts and Organizations (3); General Ledger (4); Identity and Access (2); Inventory Operations (4); Manufacturing (1); Multi-Currency (5); Products and Catalog (4) | nothing |
| Manufacturing | Analytic Accounting (3); Attendances and Working Time (1); Contacts and Organizations (7); General Ledger (5); Identity and Access (4); Inventory Operations (15); Multi-Currency (2); Products and Catalog (19); Projects and Tasks (2); Sales (1); Units of Measure and Packaging (8) | Analytic Accounting (3); Contacts and Organizations (2); General Ledger (1); Inventory Operations (22); Inventory Valuation and Costing (1); Platform Foundation (1); Products and Catalog (2); Sales (1) |
| Products and Catalog | Contacts and Organizations (10); Customer Relationship Management (1); Fiscal Localizations (3); General Ledger (14); Identity and Access (1); Inventory Operations (12); Learning, Surveys and Gamification (1); Manufacturing (2); Messaging and Activities (1); Multi-Currency (8); Platform Foundation (2); Point of Sale (2); Projects and Tasks (3); Purchasing (2); Sales (2); Units of Measure and Packaging (5); Website and Storefront (6) | Analytic Accounting (4); Contacts and Organizations (5); Customer Relationship Management (1); Delivery and Shipping (3); Events (5); Expenses (2); Fiscal Localizations (1); General Ledger (3); Inventory Operations (30); Inventory Valuation and Costing (4); Learning, Surveys and Gamification (1); Loyalty and Promotions (15); Manufacturing (19); Platform Foundation (12); Point of Sale (13); Projects and Tasks (1); Purchasing (12); Repair and Maintenance (1); Replenishment and Procurement (2); Sales (15); Website and Storefront (8) |
| Purchasing | Contacts and Organizations (19); General Ledger (9); Identity and Access (3); Inventory Operations (10); Multi-Currency (5); Products and Catalog (12); Projects and Tasks (1); Sales (1); Units of Measure and Packaging (6) | General Ledger (5); Inventory Operations (5); Products and Catalog (2) |
| Repair and Maintenance | Contacts and Organizations (7); Human Resources Core (3); Identity and Access (7); Inventory Operations (12); Products and Catalog (1); Sales (2); Units of Measure and Packaging (2) | Inventory Operations (2) |
| Replenishment and Procurement | Contacts and Organizations (1); Products and Catalog (2) | nothing |
| Units of Measure and Packaging | Fiscal Localizations (2); Inventory Operations (1) | Analytic Accounting (2); Contacts and Organizations (2); Expenses (1); General Ledger (4); Inventory Operations (15); Loyalty and Promotions (1); Manufacturing (8); Platform Foundation (1); Point of Sale (1); Products and Catalog (5); Projects and Tasks (2); Purchasing (6); Repair and Maintenance (2); Sales (6) |
| Customer Relationship Management | Contacts and Organizations (26); Events (5); Identity and Access (10); Learning, Surveys and Gamification (1); Marketing and Mass Mailing (1); Messaging and Activities (3); Multi-Currency (2); Platform Foundation (1); Products and Catalog (1); Sales (13); Website and Storefront (2) | Calendar and Scheduling (1); Contacts and Organizations (2); Events (4); Learning, Surveys and Gamification (1); Marketing and Mass Mailing (3); Messaging and Activities (2); Platform Foundation (1); Products and Catalog (1); Recruitment (2); Sales (6); Website and Storefront (1) |
| Loyalty and Promotions | Contacts and Organizations (6); Messaging and Activities (2); Multi-Currency (1); Platform Foundation (2); Point of Sale (2); Products and Catalog (15); Sales (4); Units of Measure and Packaging (1); Website and Storefront (2) | Point of Sale (2); Sales (5) |
| Payment Providers | Contacts and Organizations (11); Fiscal Localizations (1); General Ledger (3); Multi-Currency (4); Platform Foundation (5); Point of Sale (1); Sales (1); Website and Storefront (1) | General Ledger (8); Platform Foundation (1); Point of Sale (2); Sales (2) |
| Point of Sale | Attendances and Working Time (1); Contacts and Organizations (12); Events (1); Fiscal Localizations (4); General Ledger (21); Human Resources Core (7); Identity and Access (7); Inventory Operations (5); Loyalty and Promotions (2); Messaging and Activities (2); Multi-Currency (5); Payment Providers (2); Platform Foundation (7); Products and Catalog (13); Sales (5); Units of Measure and Packaging (1) | Events (1); Fiscal Localizations (4); General Ledger (6); Inventory Operations (3); Loyalty and Promotions (2); Payment Providers (1); Platform Foundation (10); Products and Catalog (2) |
| Pricing and Pricelists | nothing | nothing |
| Sales | Analytic Accounting (1); Contacts and Organizations (19); Customer Relationship Management (6); Delivery and Shipping (1); Events (5); Expenses (1); Fiscal Localizations (2); General Ledger (9); Identity and Access (7); Inventory Operations (5); Loyalty and Promotions (5); Manufacturing (1); Messaging and Activities (2); Multi-Currency (4); Payment Providers (2); Platform Foundation (1); Products and Catalog (15); Projects and Tasks (6); Units of Measure and Packaging (6); Website and Storefront (2) | Analytic Accounting (1); Contacts and Organizations (1); Customer Relationship Management (13); Delivery and Shipping (1); Events (10); Expenses (3); General Ledger (3); Identity and Access (2); Inventory Operations (4); Learning, Surveys and Gamification (1); Loyalty and Promotions (4); Manufacturing (1); Messaging and Activities (1); Payment Providers (1); Platform Foundation (2); Point of Sale (5); Products and Catalog (2); Projects and Tasks (7); Purchasing (1); Repair and Maintenance (2); Timesheets (3); Website and Storefront (2) |
| Website and Storefront | Contacts and Organizations (14); Customer Relationship Management (1); Delivery and Shipping (1); Events (2); Identity and Access (17); Inventory Operations (1); Learning, Surveys and Gamification (1); Marketing and Mass Mailing (1); Messaging and Activities (3); Multi-Currency (2); Platform Foundation (9); Products and Catalog (8); Sales (2) | Contacts and Organizations (2); Customer Relationship Management (2); Events (5); General Ledger (1); Identity and Access (1); Inventory Operations (1); Learning, Surveys and Gamification (2); Loyalty and Promotions (2); Messaging and Activities (1); Payment Providers (1); Platform Foundation (9); Products and Catalog (6); Sales (2) |
| Projects and Tasks | Analytic Accounting (1); Attendances and Working Time (1); Contacts and Organizations (13); Identity and Access (9); Messaging and Activities (7); Multi-Currency (1); Platform Foundation (3); Products and Catalog (1); Sales (7); Units of Measure and Packaging (2) | Analytic Accounting (4); Contacts and Organizations (2); Identity and Access (1); Inventory Operations (1); Manufacturing (2); Messaging and Activities (2); Platform Foundation (8); Products and Catalog (3); Purchasing (1); Sales (6); Timesheets (5) |
| Timesheets | Contacts and Organizations (4); General Ledger (1); Human Resources Core (6); Identity and Access (2); Multi-Currency (3); Projects and Tasks (5); Sales (3) | nothing |
| Attendances and Working Time | Contacts and Organizations (7); Human Resources Core (6); Identity and Access (2); Time Off (1); Work Entries (2) | Analytic Accounting (1); Contacts and Organizations (1); Human Resources Core (7); Identity and Access (1); Inventory Operations (1); Manufacturing (1); Platform Foundation (2); Point of Sale (1); Projects and Tasks (1); Time Off (4) |
| Expenses | Contacts and Organizations (4); General Ledger (8); Human Resources Core (3); Identity and Access (2); Multi-Currency (4); Products and Catalog (2); Sales (3); Units of Measure and Packaging (1) | General Ledger (2); Sales (1) |
| Fleet | Contacts and Organizations (15); General Ledger (1); Human Resources Core (4); Identity and Access (3); Multi-Currency (3); Platform Foundation (1) | General Ledger (1); Inventory Operations (2) |
| Human Resources Core | Attendances and Working Time (7); Contacts and Organizations (28); Events (1); Identity and Access (14); Learning, Surveys and Gamification (4); Multi-Currency (1); Recruitment (2); Time Off (1) | Analytic Accounting (3); Attendances and Working Time (6); Contacts and Organizations (1); Expenses (3); Fleet (4); General Ledger (1); Identity and Access (9); Learning, Surveys and Gamification (2); Messaging and Activities (3); Platform Foundation (1); Point of Sale (7); Recruitment (9); Repair and Maintenance (3); Time Off (25); Timesheets (6); Work Entries (6) |
| Lunch Ordering | Contacts and Organizations (9); Identity and Access (5); Multi-Currency (5); Platform Foundation (2) | Identity and Access (2) |
| Recruitment | Contacts and Organizations (4); Customer Relationship Management (2); Human Resources Core (9); Identity and Access (3); Learning, Surveys and Gamification (1); Messaging and Activities (4); Platform Foundation (2) | Calendar and Scheduling (1); Human Resources Core (2); Learning, Surveys and Gamification (2) |
| Time Off | Attendances and Working Time (4); Calendar and Scheduling (1); Contacts and Organizations (11); Human Resources Core (25); Identity and Access (3); Messaging and Activities (2); Platform Foundation (2); Work Entries (1) | Analytic Accounting (1); Attendances and Working Time (1); Contacts and Organizations (1); Human Resources Core (1); Platform Foundation (1); Work Entries (1) |
| Work Entries | Contacts and Organizations (3); Human Resources Core (6); Identity and Access (1); Time Off (1) | Attendances and Working Time (2); Time Off (1) |
| Events | Contacts and Organizations (19); Customer Relationship Management (4); Identity and Access (3); Messaging and Activities (1); Multi-Currency (1); Platform Foundation (1); Point of Sale (1); Products and Catalog (5); Sales (10); Website and Storefront (5) | Customer Relationship Management (5); Human Resources Core (1); Point of Sale (1); Sales (5); Website and Storefront (2) |
| Learning, Surveys and Gamification | Contacts and Organizations (14); Customer Relationship Management (1); Human Resources Core (2); Identity and Access (17); Messaging and Activities (7); Platform Foundation (9); Products and Catalog (1); Recruitment (2); Sales (1); Website and Storefront (2) | Contacts and Organizations (1); Customer Relationship Management (1); Human Resources Core (4); Identity and Access (2); Products and Catalog (1); Recruitment (1); Website and Storefront (1) |
| Marketing and Mass Mailing | Contacts and Organizations (3); Customer Relationship Management (3); Identity and Access (3); Messaging and Activities (3); Platform Foundation (4) | Customer Relationship Management (1); Messaging and Activities (7); Website and Storefront (1) |

### 4.3 Most referenced entities

The entities that the largest number of relation fields point at. These are the records that must exist, and must be seeded, before any other domain can store a complete record. Every entity referenced by at least ten relation fields is listed.

| Entity | Transport name | Specified in | Relation fields pointing at it |
|---|---|---|---|
| Contact | `res.partner` | Contacts and Organizations | 232 |
| Companies | `res.company` | Contacts and Organizations | 211 |
| User | `res.users` | Identity and Access | 188 |
| Account | `account.account` | General Ledger | 104 |
| Currency | `res.currency` | Multi-Currency | 97 |
| Product Variant | `product.product` | Products and Catalog | 85 |
| Journal Entry | `account.move` | General Ledger | 73 |
| Employee | `hr.employee` | Human Resources Core | 67 |
| Inventory Locations | `stock.location` | Inventory Operations | 66 |
| Country | `res.country` | Contacts and Organizations | 62 |
| Journal | `account.journal` | General Ledger | 62 |
| Product Unit of Measure | `uom.uom` | Units of Measure and Packaging | 57 |
| Attachment | `ir.attachment` | Platform Foundation | 47 |
| Models | `ir.model` | Platform Foundation | 38 |
| Product | `product.template` | Products and Catalog | 32 |
| Email Templates | `mail.template` | Messaging and Activities | 30 |
| Tax | `account.tax` | General Ledger | 30 |
| Picking Type | `stock.picking.type` | Inventory Operations | 29 |
| Access Groups | `res.groups` | Identity and Access | 28 |
| Website | `website` | Website and Storefront | 27 |
| Department | `hr.department` | Human Resources Core | 26 |
| Sales Order | `sale.order` | Sales | 26 |
| Warehouse | `stock.warehouse` | Inventory Operations | 26 |
| Work Location | `hr.work.location` | Human Resources Core | 26 |
| Project | `project.project` | Projects and Tasks | 25 |
| Sales Order Line | `sale.order.line` | Sales | 25 |
| Manufacturing Order | `mrp.production` | Manufacturing | 24 |
| Transfer | `stock.picking` | Inventory Operations | 24 |
| Sales Team | `crm.team` | Sales | 23 |
| Country state | `res.country.state` | Contacts and Organizations | 22 |
| Event | `event.event` | Events | 21 |
| Fields | `ir.model.fields` | Platform Foundation | 21 |
| Inventory Routes | `stock.route` | Inventory Operations | 21 |
| Fiscal Position | `account.fiscal.position` | General Ledger | 20 |
| Pricelist | `product.pricelist` | Products and Catalog | 20 |
| Sequence | `ir.sequence` | Platform Foundation | 20 |
| View | `ir.ui.view` | Platform Foundation | 20 |
| Message | `mail.message` | Messaging and Activities | 19 |
| Resource Working Time | `resource.calendar` | Attendances and Working Time | 18 |
| Journal Item | `account.move.line` | General Ledger | 17 |
| Module | `ir.module.module` | Platform Foundation | 17 |
| Payments | `account.payment` | General Ledger | 17 |
| Product Category | `product.category` | Products and Catalog | 17 |
| Task | `project.task` | Projects and Tasks | 17 |
| Languages | `res.lang` | Contacts and Organizations | 16 |
| Point of Sale Orders | `pos.order` | Point of Sale | 16 |
| Product Template Attribute Value | `product.template.attribute.value` | Products and Catalog | 16 |
| Activity Type | `mail.activity.type` | Messaging and Activities | 15 |
| Bank Accounts | `res.partner.bank` | Contacts and Organizations | 15 |
| Bill of Material | `mrp.bom` | Manufacturing | 15 |
| Package | `stock.package` | Inventory Operations | 15 |
| Point of Sale Configuration | `pos.config` | Point of Sale | 15 |
| Discussion Channel | `discuss.channel` | Messaging and Activities | 14 |
| Lead | `crm.lead` | Customer Relationship Management | 14 |
| Purchase Order | `purchase.order` | Purchasing | 14 |
| Lot/Serial | `stock.lot` | Inventory Operations | 13 |
| Mass Mailing | `mailing.mailing` | Marketing and Mass Mailing | 12 |
| Payment Methods | `account.payment.method.line` | General Ledger | 12 |
| Stock Move | `stock.move` | Inventory Operations | 12 |
| Stock Rule | `stock.rule` | Inventory Operations | 12 |
| Applicant | `hr.applicant` | Recruitment | 11 |
| Course | `slide.channel` | Learning, Surveys and Gamification | 11 |
| Time Off Type | `hr.leave.type` | Time Off | 11 |
| text message Templates | `sms.template` | Messaging and Activities | 11 |
| Expense | `hr.expense` | Expenses | 10 |
| Job Position | `hr.job` | Human Resources Core | 10 |
| Point of Sale Payment Methods | `pos.payment.method` | Point of Sale | 10 |

## 5. Structural summary of the model

### 5.1 Totals

| Measure | Count |
|---|---|
| Entities in total | 983 |
| Persistent entities | 600 |
| Interactive assistant entities | 222 |
| Shared behaviour entities | 161 |
| Domains | 46 |
| Relation fields in total | 3,985 |
| Relations of kind link to one record | 2,659 |
| Relations of kind list of records | 622 |
| Relations of kind list on both sides | 704 |
| Relations of kind polymorphic link | 0 |
| Relations of kind polymorphic reference | 0 |
| Relations materialized in storage | 2,754 |
| Relations derived at read time | 1,231 |
| Relations that cross a domain boundary | 2,063 |
| Entities that embed a parent record | 12 |
| Self-referencing relation fields | 190 |

### 5.2 Delegation: entities that embed a parent record

A delegating entity stores a link to a record of another entity and exposes every field of that record as if it were its own. Reading such a field follows the link; writing it writes on the linked record; creating a delegating record without a link creates the linked record first; and deleting the delegating record deletes the embedded parent with it. These are the delegations in the model.

| Entity | Transport name | Embedded parent entity | Transport name | Link field |
|---|---|---|---|---|
| Bank setup manual config | `account.setup.bank.manual.config` | Bank Accounts | `res.partner.bank` | `res_partner_bank_id` |
| Bank Statement Line | `account.bank.statement.line` | Journal Entry | `account.move` | `move_id` |
| Email Aliases Mixin | `mail.alias.mixin` | Email Aliases | `mail.alias` | `alias_id` |
| Employee | `hr.employee` | Version | `hr.version` | `version_id` |
| Model Page | `website.controller.page` | View | `ir.ui.view` | `view_id` |
| Outgoing Mails | `mail.mail` | Message | `mail.message` | `mail_message_id` |
| Page | `website.page` | View | `ir.ui.view` | `view_id` |
| Product Document | `product.document` | Attachment | `ir.attachment` | `ir_attachment_id` |
| Product Variant | `product.product` | Product | `product.template` | `product_tmpl_id` |
| Quotation's Headers & Footers | `quotation.document` | Attachment | `ir.attachment` | `ir_attachment_id` |
| Scheduled Actions | `ir.cron` | Server Actions | `ir.actions.server` | `ir_actions_server_id` |
| User | `res.users` | Contact | `res.partner` | `partner_id` |

### 5.3 Shared behaviours and the entities that carry them

Every shared behaviour with at least one carrier, with the number of entities that merge it. A replacement must implement each behaviour once and attach it to the same entities; the per-entity lists are in the domain folders and on the reference page of each entity.

| Shared behaviour | Transport name | Specified in | Entities that carry it |
|---|---|---|---|
| Email Thread | `mail.thread` | Messaging and Activities | 82 |
| Point of Sale data loading mixin | `pos.load.mixin` | Point of Sale | 68 |
| Activity Mixin | `mail.activity.mixin` | Messaging and Activities | 60 |
| Qweb Field | `ir.qweb.field` | Platform Foundation | 17 |
| search engine optimization metadata | `website.seo.metadata` | Website and Storefront | 16 |
| Website Searchable Mixin | `website.searchable.mixin` | Website and Storefront | 15 |
| Can send messages via bus.bus | `bus.listener.mixin` | Messaging and Activities | 13 |
| Image Mixin | `image.mixin` | Platform Foundation | 13 |
| Multi Website Published Mixin | `website.published.multi.mixin` | Website and Storefront | 13 |
| Universal Business Language BIS Billing 3.0.12 | `account.edi.xml.ubl_bis3` | Electronic Invoicing and Document Exchange | 13 |
| Analytic Mixin | `analytic.mixin` | Analytic Accounting | 10 |
| Mail Composer Mixin | `mail.composer.mixin` | Messaging and Activities | 8 |
| Mail Main Attachment management | `mail.thread.main.attachment` | Messaging and Activities | 8 |
| Portal Mixin | `portal.mixin` | Website and Storefront | 8 |
| Universal Business Language 2.1 | `account.edi.xml.ubl_21` | Electronic Invoicing and Document Exchange | 8 |
| Website Published Mixin | `website.published.mixin` | Website and Storefront | 8 |
| Actions | `ir.actions.actions` | Platform Foundation | 6 |
| Multi Website Mixin | `website.multi.mixin` | Website and Storefront | 6 |
| Product Catalog Mixin | `product.catalog.mixin` | Products and Catalog | 6 |
| Avatar Mixin | `avatar.mixin` | Platform Foundation | 5 |
| Email Aliases Mixin | `mail.alias.mixin` | Messaging and Activities | 5 |
| Email Carbon Copy management | `mail.thread.cc` | Messaging and Activities | 5 |
| Mail Render Mixin | `mail.render.mixin` | Messaging and Activities | 5 |
| campaign tracking parameter Mixin | `utm.mixin` | Customer Relationship Management | 5 |
| Cover Properties Website Mixin | `website.cover_properties.mixin` | Website and Storefront | 4 |
| Human Resources Manager Department Report | `hr.manager.department.report` | Human Resources Core | 4 |
| Mail Blacklist mixin | `mail.thread.blacklist` | Messaging and Activities | 4 |
| Mixin to compute the time a record has spent in each value a many2one field can take | `mail.tracking.duration.mixin` | Messaging and Activities | 4 |
| Phone Blacklist Mixin | `mail.thread.phone` | Customer Relationship Management | 4 |
| Rating Mixin | `rating.mixin` | Projects and Tasks | 4 |
| Address Format | `format.address.mixin` | Contacts and Organizations | 3 |
| Base helpers for Universal Business Language | `account.edi.ubl` | Electronic Invoicing and Document Exchange | 3 |
| Bus Mixin | `pos.bus.mixin` | Point of Sale | 3 |
| Business document import mixin | `account.document.import.mixin` | General Ledger | 3 |
| Common functions for electronic data interchange documents: generate the data, the constraints, etc | `account.edi.common` | Electronic Invoicing and Document Exchange | 3 |
| Skill level | `hr.individual.skill.mixin` | Human Resources Core | 3 |
| Universal Business Language 2.0 | `account.edi.xml.ubl_20` | Electronic Invoicing and Document Exchange | 3 |
| Warn Insufficient Quantity | `stock.warn.insufficient.qty` | Inventory Operations | 3 |
| Website page/record specific visibility options | `website.page_visibility_options.mixin` | Website and Storefront | 3 |
| campaign tracking parameter Source Mixin | `utm.source.mixin` | Customer Relationship Management | 3 |
| hr.mixin | `hr.mixin` | Platform Foundation | 3 |
| Account Move Send | `account.move.send` | General Ledger | 2 |
| Analytic Plan Fields | `analytic.plan.fields.mixin` | Analytic Accounting | 2 |
| Automatic sequence | `sequence.mixin` | General Ledger | 2 |
| Country Specific value-added tax Label | `format.vat.label.mixin` | Contacts and Organizations | 2 |
| Email Aliases Mixin (light) | `mail.alias.mixin.optional` | Messaging and Activities | 2 |
| Google Gmail Mixin | `google.gmail.mixin` | Calendar and Scheduling | 2 |
| Microsoft Outlook Mixin | `microsoft.outlook.mixin` | Calendar and Scheduling | 2 |
| Portal Sharing | `portal.share` | Website and Storefront | 2 |
| Properties Base Definition Mixin | `properties.base.definition.mixin` | Platform Foundation | 2 |
| Qweb Field Many to One | `ir.qweb.field.many2one` | Platform Foundation | 2 |
| Rating Parent Mixin | `rating.parent.mixin` | Projects and Tasks | 2 |
| Resource Mixin | `resource.mixin` | Attendances and Working Time | 2 |
| Spreadsheet mixin | `spreadsheet.mixin` | Spreadsheets and Dashboards | 2 |
| Synchronize a record with Google Calendar | `google.calendar.sync` | Calendar and Scheduling | 2 |
| Synchronize a record with Microsoft Calendar | `microsoft.calendar.sync` | Calendar and Scheduling | 2 |
| Template Reset Mixin | `template.reset.mixin` | Messaging and Activities | 2 |
| Website page/record specific options | `website.page_options.mixin` | Website and Storefront | 2 |
| withholding line | `account.withholding.line` | Taxes | 2 |
| Account report without payment lines | `report.account.report_invoice` | Platform Foundation | 1 |
| Base helpers for Cross Industry Invoice | `account.edi.cii` | Electronic Invoicing and Document Exchange | 1 |
| Convert Lead to Opportunity (not in mass) | `crm.lead2opportunity.partner` | Customer Relationship Management | 1 |
| Device Log | `res.device.log` | Identity and Access | 1 |
| Event Booth Template | `event.type.booth` | Events | 1 |
| Event Template Ticket | `event.type.ticket` | Events | 1 |
| Field rich text History | `html.field.history.mixin` | Website and Storefront | 1 |
| Maintenance Maintained Item | `maintenance.mixin` | Repair and Maintenance | 1 |
| Page Properties Base | `website.page.properties.base` | Website and Storefront | 1 |
| Point of Sale Details | `report.point_of_sale.report_saledetails` | Platform Foundation | 1 |
| Point of Sale Order Universal Business Language 2.1 builder | `pos.edi.xml.ubl_21` | Electronic Invoicing and Document Exchange | 1 |
| Product Replenish Mixin | `stock.replenish.mixin` | Inventory Operations | 1 |
| Qweb Field Image | `ir.qweb.field.image` | Platform Foundation | 1 |
| Stock Replenishment Report | `stock.forecasted_product_product` | Inventory Operations | 1 |
| Universal Business Language CEN-EN16931 | `account.edi.ubl_cen_en16931` | Electronic Invoicing and Document Exchange | 1 |
| Universal Business Language Peppol International Invoice | `account.edi.ubl_pint` | Electronic Invoicing and Document Exchange | 1 |
| Universal Business Language Peppol International Invoice-European Union Layer | `account.edi.ubl_pint_eu` | Electronic Invoicing and Document Exchange | 1 |
| Users application programming interface Keys | `res.users.apikeys` | Identity and Access | 1 |

### 5.4 Self-referencing relations

Relations whose target entity is the source entity. They form hierarchies, where a record has a parent record of the same kind; chains, where a record supersedes, reverses, returns or continues another record of the same kind; and pairings, where two records of the same kind reference each other.

| Entity | Transport name | Relation field | Kind | Required | On delete | Materialized ancestor path |
|---|---|---|---|---|---|---|
| Access Groups | `res.groups` | `all_implied_by_ids` | list on both sides | no | `not declared` | no |
| Access Groups | `res.groups` | `all_implied_ids` | list on both sides | no | `not declared` | no |
| Access Groups | `res.groups` | `disjoint_ids` | list on both sides | no | `not declared` | no |
| Access Groups | `res.groups` | `implied_by_ids` | list on both sides | no | `not declared` | no |
| Access Groups | `res.groups` | `implied_ids` | list on both sides | no | `not declared` | no |
| Account | `account.account` | `account_stock_expense_id` | link to one record | no | `not declared` | no |
| Account | `account.account` | `account_stock_variation_id` | link to one record | no | `not declared` | no |
| Account codes first 2 digits | `account.root` | `parent_id` | link to one record | no | `not declared` | no |
| Account Group | `account.group` | `parent_id` | link to one record | no | `cascade` | no |
| Accounting Report | `account.report` | `root_report_id` | link to one record | no | `not declared` | no |
| Accounting Report | `account.report` | `section_main_report_ids` | list on both sides | no | `not declared` | no |
| Accounting Report | `account.report` | `section_report_ids` | list on both sides | no | `not declared` | no |
| Accounting Report | `account.report` | `variant_report_ids` | list of records | no | `mirror of the target column` | no |
| Accounting Report Line | `account.report.line` | `children_ids` | list of records | no | `mirror of the target column` | no |
| Accounting Report Line | `account.report.line` | `parent_id` | link to one record | no | `set null` | no |
| Activity Type | `mail.activity.type` | `previous_type_ids` | list on both sides | no | `not declared` | no |
| Activity Type | `mail.activity.type` | `suggested_next_type_ids` | list on both sides | no | `not declared` | no |
| Activity Type | `mail.activity.type` | `triggered_next_type_id` | link to one record | no | `restrict` | no |
| Analytic Plans | `account.analytic.plan` | `children_ids` | list of records | no | `mirror of the target column` | yes |
| Analytic Plans | `account.analytic.plan` | `parent_id` | link to one record | no | `cascade` | yes |
| Analytic Plans | `account.analytic.plan` | `root_id` | link to one record | no | `not declared` | yes |
| Applicant | `hr.applicant` | `pool_applicant_id` | link to one record | no | `not declared` | no |
| Application | `ir.module.category` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Application | `ir.module.category` | `parent_id` | link to one record | no | `not declared` | no |
| Attachment | `ir.attachment` | `original_id` | link to one record | no | `not declared` | no |
| Bank | `res.bank` | `intermediary_bank_id` | link to one record | no | `not declared` | no |
| Bill of Material Line | `mrp.bom.line` | `child_line_ids` | list of records | no | `mirror of the target column` | no |
| Certificate | `certificate.certificate` | `issuer_cert_id` | link to one record | no | `not declared` | no |
| Companies | `res.company` | `all_child_ids` | list of records | no | `mirror of the target column` | yes |
| Companies | `res.company` | `child_ids` | list of records | no | `mirror of the target column` | yes |
| Companies | `res.company` | `parent_id` | link to one record | no | `restrict` | yes |
| Companies | `res.company` | `parent_ids` | list on both sides | no | `not declared` | yes |
| Companies | `res.company` | `peppol_parent_company_id` | link to one record | no | `not declared` | yes |
| Companies | `res.company` | `root_id` | link to one record | no | `not declared` | yes |
| Contact | `res.partner` | `assigned_partner_id` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Contact | `res.partner` | `commercial_partner_id` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `implemented_partner_ids` | list of records | no | `mirror of the target column` | no |
| Contact | `res.partner` | `l10n_pl_parent_lgu` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `parent_id` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `same_company_registry_partner_id` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `same_vat_partner_id` | link to one record | no | `not declared` | no |
| Contact | `res.partner` | `self` | link to one record | no | `not declared` | no |
| Course | `slide.channel` | `prerequisite_channel_ids` | list on both sides | no | `not declared` | no |
| Course | `slide.channel` | `prerequisite_of_channel_ids` | list on both sides | no | `not declared` | no |
| Department | `hr.department` | `child_ids` | list of records | no | `mirror of the target column` | yes |
| Department | `hr.department` | `master_department_id` | link to one record | no | `not declared` | yes |
| Department | `hr.department` | `parent_id` | link to one record | no | `not declared` | yes |
| Discussion Channel | `discuss.channel` | `parent_channel_id` | link to one record | no | `cascade` | no |
| Discussion Channel | `discuss.channel` | `sub_channel_ids` | list of records | no | `mirror of the target column` | no |
| Employee | `hr.employee` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Employee | `hr.employee` | `coach_id` | link to one record | no | `not declared` | no |
| Employee | `hr.employee` | `parent_id` | link to one record | no | `not declared` | no |
| Employee | `hr.employee` | `subordinate_ids` | list of records | no | `mirror of the target column` | no |
| Expense | `hr.expense` | `duplicate_expense_ids` | list on both sides | no | `not declared` | no |
| Expense | `hr.expense` | `same_receipt_expense_ids` | list on both sides | no | `not declared` | no |
| Expense | `hr.expense` | `split_expense_origin_id` | link to one record | no | `not declared` | no |
| Fields | `ir.model.fields` | `related_field_id` | link to one record | no | `cascade` | no |
| Fields | `ir.model.fields` | `relation_field_id` | link to one record | no | `cascade` | no |
| Fields | `ir.model.fields` | `serialization_field_id` | link to one record | no | `cascade` | no |
| Forum Post | `forum.post` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Forum Post | `forum.post` | `parent_id` | link to one record | no | `cascade` | no |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `initial_flow_id` | link to one record | no | `not declared` | no |
| French Approved Dematerialization Platform Flow | `l10n.fr.pdp.reports.flow` | `rectificative_flow_ids` | list of records | no | `mirror of the target column` | no |
| Gamification Badge | `gamification.badge` | `rule_auth_badge_ids` | list on both sides | no | `not declared` | no |
| Inventory Locations | `stock.location` | `child_ids` | list of records | no | `mirror of the target column` | yes |
| Inventory Locations | `stock.location` | `child_internal_location_ids` | list on both sides | no | `not declared` | yes |
| Inventory Locations | `stock.location` | `location_id` | link to one record | no | `not declared` | yes |
| Journal Entry | `account.move` | `adjusting_entries_move_ids` | list on both sides | no | `not declared` | no |
| Journal Entry | `account.move` | `adjusting_entry_origin_move_ids` | list on both sides | no | `not declared` | no |
| Journal Entry | `account.move` | `auto_post_origin_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `debit_note_ids` | list of records | no | `mirror of the target column` | no |
| Journal Entry | `account.move` | `debit_origin_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `duplicated_ref_ids` | list on both sides | no | `not declared` | no |
| Journal Entry | `account.move` | `invoice_vendor_bill_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_es_edi_verifactu_substituted_entry_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_es_edi_verifactu_substitution_move_ids` | list of records | no | `mirror of the target column` | no |
| Journal Entry | `account.move` | `l10n_es_tbai_reversed_ids` | list on both sides | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_gr_edi_correlation_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_in_withhold_move_ids` | list of records | no | `mirror of the target column` | no |
| Journal Entry | `account.move` | `l10n_in_withholding_ref_move_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_sa_edi_chain_head_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `l10n_vn_edi_replacement_origin_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `pos_refunded_invoice_ids` | list on both sides | no | `not declared` | no |
| Journal Entry | `account.move` | `reversal_move_ids` | list of records | no | `mirror of the target column` | no |
| Journal Entry | `account.move` | `reversed_entry_id` | link to one record | no | `not declared` | no |
| Journal Entry | `account.move` | `tax_cash_basis_created_move_ids` | list of records | no | `mirror of the target column` | no |
| Journal Entry | `account.move` | `tax_cash_basis_origin_move_id` | link to one record | no | `not declared` | no |
| Journal Item | `account.move.line` | `cogs_origin_id` | link to one record | no | `not declared` | no |
| Journal Item | `account.move.line` | `first_reconciled_lines_excluding_exchange_diff_id` | link to one record | no | `not declared` | no |
| Journal Item | `account.move.line` | `first_reconciled_lines_id` | link to one record | no | `not declared` | no |
| Journal Item | `account.move.line` | `parent_id` | link to one record | no | `not declared` | no |
| Journal Item | `account.move.line` | `reconciled_lines_excluding_exchange_diff_ids` | list on both sides | no | `not declared` | no |
| Journal Item | `account.move.line` | `reconciled_lines_ids` | list on both sides | no | `not declared` | no |
| Lead | `crm.lead` | `duplicate_lead_ids` | list on both sides | no | `not declared` | no |
| Mailing List Message | `mail.group.message` | `group_message_child_ids` | list of records | no | `mirror of the target column` | no |
| Mailing List Message | `mail.group.message` | `group_message_parent_id` | link to one record | no | `not declared` | no |
| Menu | `ir.ui.menu` | `child_id` | list of records | no | `mirror of the target column` | yes |
| Menu | `ir.ui.menu` | `parent_id` | link to one record | no | `restrict` | yes |
| Message | `mail.message` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Message | `mail.message` | `linked_message_ids` | list on both sides | no | `not declared` | no |
| Message | `mail.message` | `parent_id` | link to one record | no | `set null` | no |
| Message subtypes | `mail.message.subtype` | `parent_id` | link to one record | no | `set null` | no |
| Models | `ir.model` | `inherited_model_ids` | list on both sides | no | `not declared` | no |
| Package | `stock.package` | `all_children_package_ids` | list of records | no | `mirror of the target column` | yes |
| Package | `stock.package` | `child_package_dest_ids` | list of records | no | `mirror of the target column` | yes |
| Package | `stock.package` | `child_package_ids` | list of records | no | `mirror of the target column` | yes |
| Package | `stock.package` | `outermost_package_id` | link to one record | no | `not declared` | yes |
| Package | `stock.package` | `package_dest_id` | link to one record | no | `not declared` | yes |
| Package | `stock.package` | `parent_package_id` | link to one record | no | `not declared` | yes |
| Partner Tags | `res.partner.category` | `child_ids` | list of records | no | `mirror of the target column` | yes |
| Partner Tags | `res.partner.category` | `parent_id` | link to one record | no | `cascade` | yes |
| Payment Method | `payment.method` | `brand_ids` | list of records | no | `mirror of the target column` | no |
| Payment Method | `payment.method` | `primary_payment_method_id` | link to one record | no | `not declared` | no |
| Payment Transaction | `payment.transaction` | `child_transaction_ids` | list of records | no | `mirror of the target column` | no |
| Payment Transaction | `payment.transaction` | `source_transaction_id` | link to one record | no | `not declared` | no |
| Payments | `account.payment` | `duplicate_payment_ids` | list on both sides | no | `not declared` | no |
| Payments | `account.payment` | `paired_internal_transfer_payment_id` | link to one record | no | `not declared` | no |
| Payments | `account.payment` | `source_payment_id` | link to one record | no | `not declared` | no |
| Picking Type | `stock.picking.type` | `return_picking_type_id` | link to one record | no | `not declared` | no |
| Point of Sale Category | `pos.category` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Point of Sale Category | `pos.category` | `parent_id` | link to one record | no | `not declared` | no |
| Point of Sale Configuration | `pos.config` | `trusted_config_ids` | list on both sides | no | `not declared` | no |
| Point of Sale Order Lines | `pos.order.line` | `combo_line_ids` | list of records | no | `mirror of the target column` | no |
| Point of Sale Order Lines | `pos.order.line` | `combo_parent_id` | link to one record | no | `not declared` | no |
| Point of Sale Order Lines | `pos.order.line` | `refund_orderline_ids` | list of records | no | `mirror of the target column` | no |
| Point of Sale Order Lines | `pos.order.line` | `refunded_orderline_id` | link to one record | no | `not declared` | no |
| Point of Sale Orders | `pos.order` | `previous_order_id` | link to one record | no | `not declared` | no |
| Point of Sale Orders | `pos.order` | `refunded_order_id` | link to one record | no | `not declared` | no |
| Product | `product.template` | `alternative_product_ids` | list on both sides | no | `not declared` | no |
| Product | `product.template` | `optional_product_ids` | list on both sides | no | `not declared` | no |
| Product | `product.template` | `pos_optional_product_ids` | list on both sides | no | `not declared` | no |
| Product Category | `product.category` | `child_id` | list of records | no | `mirror of the target column` | yes |
| Product Category | `product.category` | `parent_id` | link to one record | no | `cascade` | yes |
| Product Moves (Stock Move Line) | `stock.move.line` | `consume_line_ids` | list on both sides | no | `not declared` | no |
| Product Moves (Stock Move Line) | `stock.move.line` | `produce_line_ids` | list on both sides | no | `not declared` | no |
| Product Unit of Measure | `uom.uom` | `related_uom_ids` | list of records | no | `mirror of the target column` | yes |
| Product Unit of Measure | `uom.uom` | `relative_uom_id` | link to one record | no | `cascade` | yes |
| Production Group | `mrp.production.group` | `child_ids` | list on both sides | no | `not declared` | no |
| Production Group | `mrp.production.group` | `parent_ids` | list on both sides | no | `not declared` | no |
| Public Employee | `hr.employee.public` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Public Employee | `hr.employee.public` | `coach_id` | link to one record | no | `not declared` | no |
| Public Employee | `hr.employee.public` | `parent_id` | link to one record | no | `not declared` | no |
| Purchase Order | `purchase.order` | `alternative_po_ids` | list of records | no | `mirror of the target column` | no |
| Purchase Order | `purchase.order` | `duplicated_order_ids` | list on both sides | no | `not declared` | no |
| Purchase Order Line | `purchase.order.line` | `parent_id` | link to one record | no | `not declared` | no |
| Quotation Template Line | `sale.order.template.line` | `parent_id` | link to one record | no | `not declared` | no |
| Restaurant Table | `restaurant.table` | `parent_id` | link to one record | no | `not declared` | no |
| Sales Order | `sale.order` | `duplicated_order_ids` | list on both sides | no | `not declared` | no |
| Sales Order Line | `sale.order.line` | `linked_line_id` | link to one record | no | `cascade` | no |
| Sales Order Line | `sale.order.line` | `linked_line_ids` | list of records | no | `mirror of the target column` | no |
| Sales Order Line | `sale.order.line` | `parent_id` | link to one record | no | `not declared` | no |
| Server Actions | `ir.actions.server` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Server Actions | `ir.actions.server` | `parent_id` | link to one record | no | `cascade` | no |
| Slides | `slide.slide` | `category_id` | link to one record | no | `not declared` | no |
| Slides | `slide.slide` | `slide_ids` | list of records | no | `mirror of the target column` | no |
| Stock Move | `stock.move` | `move_dest_ids` | list on both sides | no | `not declared` | no |
| Stock Move | `stock.move` | `move_orig_ids` | list on both sides | no | `not declared` | no |
| Stock Move | `stock.move` | `origin_returned_move_id` | link to one record | no | `not declared` | no |
| Stock Move | `stock.move` | `returned_move_ids` | list of records | no | `mirror of the target column` | no |
| Survey Question | `survey.question` | `allowed_triggering_question_ids` | list on both sides | no | `not declared` | no |
| Survey Question | `survey.question` | `page_id` | link to one record | no | `not declared` | no |
| Survey Question | `survey.question` | `question_ids` | list of records | no | `mirror of the target column` | no |
| Survey Question | `survey.question` | `triggering_question_ids` | list on both sides | no | `not declared` | no |
| Task | `project.task` | `child_ids` | list of records | no | `mirror of the target column` | no |
| Task | `project.task` | `depend_on_ids` | list on both sides | no | `not declared` | no |
| Task | `project.task` | `dependent_ids` | list on both sides | no | `not declared` | no |
| Task | `project.task` | `parent_id` | link to one record | no | `not declared` | no |
| Tax | `account.tax` | `children_tax_ids` | list on both sides | no | `not declared` | no |
| Tax | `account.tax` | `original_tax_ids` | list on both sides | no | `cascade` | no |
| Tax | `account.tax` | `replacing_tax_ids` | list on both sides | no | `not declared` | no |
| Transfer | `stock.picking` | `backorder_id` | link to one record | no | `not declared` | no |
| Transfer | `stock.picking` | `backorder_ids` | list of records | no | `mirror of the target column` | no |
| Transfer | `stock.picking` | `return_id` | link to one record | no | `not declared` | no |
| Transfer | `stock.picking` | `return_ids` | list of records | no | `mirror of the target column` | no |
| Version | `hr.version` | `contract_template_id` | link to one record | no | `not declared` | no |
| View | `ir.ui.view` | `inherit_children_ids` | list of records | no | `mirror of the target column` | no |
| View | `ir.ui.view` | `inherit_id` | link to one record | no | `restrict` | no |
| Warehouse | `stock.warehouse` | `resupply_wh_ids` | list on both sides | no | `not declared` | no |
| Website Menu | `website.menu` | `child_id` | list of records | no | `mirror of the target column` | yes |
| Website Menu | `website.menu` | `parent_id` | link to one record | no | `cascade` | yes |
| Website Product Category | `product.public.category` | `child_id` | list of records | no | `mirror of the target column` | yes |
| Website Product Category | `product.public.category` | `parent_id` | link to one record | no | `cascade` | yes |
| Website Product Category | `product.public.category` | `parents_and_self` | list on both sides | no | `not declared` | yes |
| Website Theme Menu | `theme.website.menu` | `parent_id` | link to one record | no | `cascade` | no |
| Work Center | `mrp.workcenter` | `alternative_workcenter_ids` | list on both sides | no | `not declared` | no |
| Work Center Usage | `mrp.routing.workcenter` | `blocked_by_operation_ids` | list on both sides | no | `not declared` | no |
| Work Center Usage | `mrp.routing.workcenter` | `needed_by_operation_ids` | list on both sides | no | `not declared` | no |
| Work Order | `mrp.workorder` | `blocked_by_workorder_ids` | list on both sides | no | `not declared` | no |
| Work Order | `mrp.workorder` | `needed_by_workorder_ids` | list on both sides | no | `not declared` | no |
## 6. Reconciliation notes

The target branch carried no file for this topic, so the structure comes from the working branch. Every enumeration was regenerated from the machine-readable catalogues of this repository, and the following differences were resolved against them and against the source tree.

| Point | Working branch | Resolution |
|---|---|---|
| Entity identifiers | The identifier column carried a name derived from the full name of the entity, for example `contact` for the party entity, `bank_account` for the bank account entity and `country_subdivision` for the country subdivision entity. One row carried a whole sentence in place of an identifier. | Replaced by the transport name and the table name that the registry and the observed schema report, for example `res.partner` and `res_partner`. A map whose identifiers cannot be used to address the entity is not usable by a rebuild, and rule three of the documentation rules requires identifiers to be reproduced exactly. Every entity row now carries the full name, the transport name and the table. |
| Field identifiers in the relationship tables | Written as de-pluralized paraphrases, for example `companys`, `tags`, `implied_bys`. | Replaced by the field names the registry reports, for example `company_ids`, `tag_ids`, `implied_ids`. |
| Domain names | The forty-five keys of the working taxonomy, whose names differ from the folder names of this repository. | Replaced by the forty-five domain folders of this repository plus the platform foundation, which owns the generic platform entities. Each section states the folder that specifies the domain, and links to it where the folder exists. |
| Domain attribution of the foundation entities | The foundation entities were split across an identity domain, a platform domain and a contacts domain by hand. | The repository's own attribution assigns every entity of the foundation package to the first domain whose scope lists that package, which is not the folder that specifies them. The writing charter overrides it: entities whose transport name begins with `ir.`, `base.`, `base_import.`, `report.`, `format.`, `properties.` or `change.` are attributed to the platform foundation, the party, company, bank, country, country subdivision, country group, city, language, partner tag and industry entities to contacts and organizations, the currency and currency rate entities to multi-currency, and the user, group, privilege, device, application key, settings and deletion-request entities to identity and access. |
| Entity totals | 601 persistent, 222 assistants, 160 shared behaviours, 983 in total. | 600 persistent, 222 assistants, 161 shared behaviours, 983 in total. One entity counted as persistent is a shared behaviour that carries no table of its own; the total is unchanged. |
| Relation totals | 3,632 relation fields, of which 2,493 materialized and 1,139 derived, and 2,244 crossing a domain boundary. | 3,985 relation fields, of which 2,754 materialized and 1,231 derived, and 2,063 crossing a domain boundary. The working branch counted only the relations declared by persistent entities; this map counts every relation field of every entity, because an assistant entity that links to a business record still needs the link to exist. The crossing count also changed with the domain attribution above. |
| Cross-domain traffic | Counted every relation field, including lists of records. | Counts only the materialized side. A list of records is the mirror of a link already counted on the other side, so counting both double-counts the dependency. |
| Delegation | Twelve delegations were listed, with paraphrased link fields such as `journal_entry` and `partner`. | The registry reports the same delegations with their real link fields, for example `move_id` and `partner_id`; every one is listed with both transport names. |
| Self-referencing relations | Listed with paraphrased field names and with `set_null` written as one word. | Regenerated with the real field names and with the deletion behaviour spelled as the registry reports it. |
