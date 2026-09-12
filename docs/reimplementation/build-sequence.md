# Build sequence

The dependency-ordered plan for rebuilding the platform from nothing: twenty steps, grouped into ten stages, each step a self-contained delivery with named entities, named operations, named reports, named scheduled jobs and named interfaces, ordered so that every step depends only on what the steps before it already deliver.

Nothing here prescribes a programming language, a framework, a database product or a deployment arrangement. It prescribes what must exist, in what order, and what "finished" means for each part. A step is finished when its gate in [milestones](milestones.md) passes; the scenarios that prove it are in the [equivalence test plan](equivalence-test-plan.md); the level of conformance the gates are read against is chosen from [conformance profiles](conformance-profiles.md).

---

## 1. How to read this plan

### 1.1 Twenty steps and ten stages

The plan is written at two grains and the two agree.

- A **step** is a delivery unit: one team, one coherent body of behavior, one gate. There are twenty of them, numbered 1 to 20, and their order is forced by dependency.
- A **stage** is a coarse phase of the rebuild that groups the steps which share one risk. There are ten of them. A stage exists so that a program can be reported and financed at a level above the step, and so that the properties that must hold across several steps — recomputation, access enforcement, rounding discipline, quantity conservation — have somewhere to be asserted.

Step 1 is one delivery unit but closes three stage gates, because the platform, identity and access, and the presentation and transport contracts are mutually dependent — the shipped access rules are records loaded by the package loader, and the loader stores its own records through the registry it defines — while each of the three carries an acceptance property that has to be asserted on its own.

Steps 1 to 3, which are stages one to five up to and including master data, build a platform and a set of shared records that contain no business decision at all. They are the largest source of risk, because every later step assumes their semantics exactly. Steps 4 to 20 add business capability, much of it in parallel once the master data of step 3 exists.

### 1.2 The shape of a step

Every step is described with the same eight sections.

| Section | Content |
|---|---|
| Purpose | What business capability exists at the end of the step that did not exist before. |
| Prerequisites | The steps that must be complete. A step never reads a specification that a later step delivers. |
| Entities to deliver | The persistent, abstract and transient entities that the step brings into existence, by canonical name. Transient entities are the short-lived records behind assisted dialogs; they are listed because their validations and defaults are part of observable behavior. |
| Operations to deliver | The business operations a user or an integration can invoke, by full-word identifier. Every operation is specified in the workflow file of its domain with actors, guards, records written and messages. |
| Reports to deliver | The printed and downloadable documents produced by the step. |
| Scheduled jobs to deliver | The background jobs, with their firing interval and the effect of one run. |
| Interfaces to deliver | The request endpoints, service operations, screens and exchange files. |
| Domain folders to read | The exact folders under `../domains/` that specify the step, in reading order. |

Three further sections appear where the step needs them: **Decisions to make first**, which names the choices that pervade the code and cannot be changed cheaply afterwards; **Notes on order inside the step**, which orders the work within the delivery; and **Gate**, which names the milestone that closes the step.

### 1.3 Ordering rules that produced this sequence

1. **A step may only depend on delivered behavior.** Every cross-domain rule cited by a step is owned by a domain delivered in the same step or an earlier one.
2. **Entities before operations, operations before automation.** Inside a step, deliver the persistent shape (fields, constraints, defaults, derivations), then the state machine and operations, then the scheduled jobs and interfaces that drive them.
3. **Money after measurement.** Units of measure and product identity precede pricing; pricing precedes any document that carries a price; the ledger precedes any document that posts.
4. **Physical before financial.** Inventory movements are delivered before inventory valuation, because valuation reads movement quantities, ordering and states, and never the reverse.
5. **Source documents after their consequences.** Purchasing is delivered after receipts and vendor bills exist; sales after deliveries and customer invoices exist; manufacturing after both; the point of sale after sales, inventory and payments.
6. **The collaboration substrate is split.** The parts that every business document needs (the discussion thread, followers, tracked field changes, activities, outgoing electronic mail) are delivered early, in step 2. The parts that are themselves an application (live chat, chat bots, text messages, postal mail, the incoming mail gateway, mailing lists) are delivered in step 17. This split is deliberate: without it, either every document entity would be blocked behind a chat application, or the chat application would be rebuilt twice.
7. **Country packages last.** Fiscal localizations are delivered after the electronic exchange layer, because most country packages ship electronic document formats, and after every document type they touch exists.

### 1.4 The two costly mistakes

A common and expensive mistake is to start at step 4 or step 7, because accounting and inventory are what the business asked for, and to reimplement the platform incidentally underneath them. The platform's behaviors — dependency-driven recomputation, the record cache and its invalidation, record rules, company scoping, extension resolution — are pervasive. Retrofitting them after domain code exists means rewriting the domain code.

A second mistake is to treat the platform as something to be bought rather than built. Most of the platform's requirements can be met by an existing foundation in the chosen language. What cannot be bought is the exact semantics: the ordering of a flush, the moment a derived field is recalculated, the way two record rules combine, the precision at which a monetary amount is rounded. Those must be implemented deliberately against this specification, whatever foundation underlies them.

### 1.5 Parallel tracks

After step 6 the plan admits five tracks that can be built by separate teams. They share the platform, the catalog and the ledger, and nothing else; they re-converge at step 20.

| Track | Steps | What it needs before it starts | Coordination it still owes |
|---|---|---|---|
| Financial track | 4, 5, 6 | Steps 2 and 3. | It owns the tax engine and the rounding primitives that every other track calls. |
| Supply chain track | 7, 8, 9, 10, 12 | Steps 3 and 4 for its physical part, step 6 before purchasing. | It owns quantity conservation and valuation. |
| Sales and service track | 11, 13, 14 | Steps 6, 7 and 9. | The counter of step 13 must call the same tax arithmetic as the financial track. |
| People track | 15, 16 | Steps 2 and 6. | Working-time arithmetic is shared with manufacturing scheduling. |
| Presentation track | 17, 18, 19 | Step 2, and step 11 before the storefront. | It owns the blacklists that the marketing step of the same track depends on. |

Step 20 is final and depends on nearly everything.

### 1.6 Step map

| Step | Name | Stage | Domains delivered | Requires |
|---|---|---|---|---|
| 1 | Platform foundation | One, two, three | Platform Foundation, Identity and Access | none |
| 2 | Shared entities and the collaboration substrate | Four, five | Contacts and Organizations, Multi-Currency, Calendar and Scheduling, the document-thread part of Messaging and Activities | 1 |
| 3 | Products, units of measure and pricing | Five | Units of Measure and Packaging, Products and Catalog, Pricing and Pricelists | 2 |
| 4 | General ledger | Six | General Ledger | 2 |
| 5 | Taxes, fiscal positions and analytic accounting | Six | Taxes, Analytic Accounting, Financial Reporting | 4 |
| 6 | Receivables, payables, payments, payment providers | Six | Accounts Receivable, Accounts Payable, Payments and Bank Reconciliation, Payment Providers | 5, and 3 for the product-driven line defaults |
| 7 | Inventory operations | Seven | Inventory Operations, Delivery and Shipping | 3, 4 |
| 8 | Inventory valuation and costing | Seven | Inventory Valuation and Costing | 7, 5 |
| 9 | Replenishment and procurement | Seven | Replenishment and Procurement | 7 |
| 10 | Purchasing | Seven | Purchasing | 6, 8, 9 |
| 11 | Sales and promotions | Eight | Sales, Loyalty and Promotions | 6, 7, 9 |
| 12 | Manufacturing, repair and maintenance | Seven | Manufacturing, Repair and Maintenance | 8, 9 |
| 13 | Point of sale | Eight | Point of Sale | 6, 7, 11 |
| 14 | Customer relationship management | Eight | Customer Relationship Management | 11 |
| 15 | The human resources family | Nine | Human Resources Core, Work Entries, Attendances and Working Time, Time Off, Expenses, Recruitment, Fleet, Lunch Ordering, and the recognition part of Learning, Questionnaires and Recognition | 2, 6 |
| 16 | Projects and timesheets | Nine | Projects and Tasks, Timesheets | 11, 15 |
| 17 | Communication | Four | Messaging and Activities, completed | 2 |
| 18 | Marketing, events, questionnaires and courses | Ten | Marketing and Mass Mailing, Events, Learning, Questionnaires and Recognition | 17, 11 |
| 19 | Website, storefront and portal | Eight | Website and Storefront, Customer Portal | 11, 6, 17 |
| 20 | Spreadsheets, automation, electronic invoicing, localizations and industry blueprints | Ten, and Six for the country and exchange rules | Spreadsheets and Dashboards, Automation and Integration, Electronic Invoicing and Document Exchange, Fiscal Localizations | all |

### 1.7 Stage map

Each stage closes with a stage gate in [milestones](milestones.md): a short list of properties that must be demonstrably true across the whole stage, each mapped to a test layer of the [equivalence test plan](equivalence-test-plan.md). The step gates prove that each delivery is correct; the stage gates prove that the deliveries together hold the property the stage exists for.

| Stage | Name | What exists at the end of the stage | Steps | Stage gate |
|---|---|---|---|---|
| One | Platform foundation | A running application that can define entities, persist them, query them, extend them and load data into them, with no business domain present. | 1, its registry, field, recomputation, query, unit-of-work, numbering, data-loading and translation parts | Stage gate one |
| Two | Identity and access | Authenticated users with groups, enforced access rights, enforced record rules and enforced company scope. | 1, its identity and access part | Stage gate two |
| Three | Presentation contracts and transport | A client can obtain a view definition, open an action, navigate menus, and call the generic entity operations over the transport. | 1, its presentation, transport, attachment and report parts | Stage gate three |
| Four | Messaging, activities and scheduled work | Any entity can carry a discussion thread, followers, tracked field changes and activities; the system sends and receives mail; scheduled jobs run. | 2 for the substrate, 17 for the complete communication application | Stage gate four |
| Five | Master data | The records every later domain refers to: parties, companies, countries, currencies, languages, units, packagings, products, variants, categories, price lists and vendor prices. | 2, 3 | Stage gate five |
| Six | Money | A complete accounting system: chart of accounts, journals, entries, taxes, analytic attribution, receivables, payables, payments, reconciliation, statements, and then the country and exchange rules built on them. | 4, 5, 6, and the electronic exchange and localization parts of 20 | Stage gate six |
| Seven | Goods | A complete inventory, procurement and manufacturing system with valuation. | 7, 8, 9, 10, 12 | Stage gate seven |
| Eight | Commerce | Selling, in every channel: quotations and orders, the counter, the pipeline, the storefront and the portal. | 11, 13, 14, 19 | Stage gate eight |
| Nine | People and services | Employment, time, absence, expense, project and time-billing capability. | 15, 16 | Stage gate nine |
| Ten | Remaining capabilities | Calendar and events, marketing, learning and questionnaires, spreadsheets and dashboards, automation and integration. | 18, and the spreadsheet and automation parts of 20; the calendar part is delivered early, in step 2 | Stage gate ten |

### 1.8 Which step delivers which domain folder

The forty-seven domain folders of `../domains/` are distributed over the twenty steps as follows. A folder appears once, against the step that delivers the behavior it specifies; where a second step completes it, both are named.

| Domain folder | Step | Stage |
|---|---:|---|
| [accounts-payable](../domains/accounts-payable/) | 6 | Six |
| [accounts-receivable](../domains/accounts-receivable/) | 6 | Six |
| [analytic-accounting](../domains/analytic-accounting/) | 5 | Six |
| [attendances-and-working-time](../domains/attendances-and-working-time/) | 15 | Nine |
| [automation-and-integration](../domains/automation-and-integration/) | 20 | Ten |
| [calendar-and-scheduling](../domains/calendar-and-scheduling/) | 2 | Five |
| [contacts-and-organizations](../domains/contacts-and-organizations/) | 2 | Five |
| [customer-portal](../domains/customer-portal/) | 19 | Eight |
| [customer-relationship-management](../domains/customer-relationship-management/) | 14 | Eight |
| [delivery-and-shipping](../domains/delivery-and-shipping/) | 7 | Seven |
| [electronic-invoicing-and-document-exchange](../domains/electronic-invoicing-and-document-exchange/) | 20 | Six |
| [events](../domains/events/) | 18 | Ten |
| [expenses](../domains/expenses/) | 15 | Nine |
| [financial-reporting](../domains/financial-reporting/) | 5 | Six |
| [fiscal-localizations](../domains/fiscal-localizations/) | 20 | Six |
| [fleet](../domains/fleet/) | 15 | Nine |
| [general-ledger](../domains/general-ledger/) | 4 | Six |
| [human-resources-core](../domains/human-resources-core/) | 15 | Nine |
| [identity-and-access](../domains/identity-and-access/) | 1 | Two |
| [inventory-operations](../domains/inventory-operations/) | 7 | Seven |
| [inventory-valuation-and-costing](../domains/inventory-valuation-and-costing/) | 8 | Seven |
| [learning-surveys-and-gamification](../domains/learning-surveys-and-gamification/) | 18, and its recognition part in 15 | Ten |
| [loyalty-and-promotions](../domains/loyalty-and-promotions/) | 11 | Eight |
| [lunch-ordering](../domains/lunch-ordering/) | 15 | Nine |
| [manufacturing](../domains/manufacturing/) | 12 | Seven |
| [marketing-and-mass-mailing](../domains/marketing-and-mass-mailing/) | 18 | Ten |
| [messaging-and-activities](../domains/messaging-and-activities/) | 2 and 17 | Four |
| [multi-currency](../domains/multi-currency/) | 2 | Five |
| [payment-providers](../domains/payment-providers/) | 6 | Six |
| [payments-and-bank-reconciliation](../domains/payments-and-bank-reconciliation/) | 6 | Six |
| [platform-foundation](../domains/platform-foundation/) | 1 | One |
| [point-of-sale](../domains/point-of-sale/) | 13 | Eight |
| [pricing-and-pricelists](../domains/pricing-and-pricelists/) | 3 | Five |
| [products-and-catalog](../domains/products-and-catalog/) | 3 | Five |
| [projects-and-tasks](../domains/projects-and-tasks/) | 16 | Nine |
| [purchasing](../domains/purchasing/) | 10 | Seven |
| [recruitment](../domains/recruitment/) | 15 | Nine |
| [repair-and-maintenance](../domains/repair-and-maintenance/) | 12 | Seven |
| [replenishment-and-procurement](../domains/replenishment-and-procurement/) | 9 | Seven |
| [sales](../domains/sales/) | 11 | Eight |
| [spreadsheets-and-dashboards](../domains/spreadsheets-and-dashboards/) | 20 | Ten |
| [taxes](../domains/taxes/) | 5 | Six |
| [time-off](../domains/time-off/) | 15 | Nine |
| [timesheets](../domains/timesheets/) | 16 | Nine |
| [units-of-measure-and-packaging](../domains/units-of-measure-and-packaging/) | 3 | Five |
| [website-and-storefront](../domains/website-and-storefront/) | 19 | Eight |
| [work-entries](../domains/work-entries/) | 15 | Nine |

The cross-domain traces of [cross-domain transactions](../domains/cross-domain-transactions.md) span several of these folders; they are exercised from step 10 onward, as soon as a trace's first and last document both exist.

### 1.9 Common substrate every step uses

The following behaviors are delivered once, in step 1, and are assumed by every later step without repetition: the record model with its field kinds and derivations, the query and filter notation, the unit of work with its flush and cache invalidation rules, inheritance and extension, the package registry and the loading of shipped records under stable external identifiers, translations, access groups and record rules, screens and actions, numbering sequences, attachments, scheduled execution and the report renderer. The runtime semantics of transactions, concurrency, caching and background execution are specified in [`../runtime/`](../runtime/) and are entry conditions for step 1, not deliverables of later steps.

Every formula a step delivers has a machine-readable counterpart in the mathematics catalogs under [`../../schemas/mathematics/`](../../schemas/mathematics/), whose worked examples are the arithmetic tests of that step.

---

## Step 1: Platform foundation

### Purpose

At the end of step 1 an empty but complete application server exists: it can define entities at runtime, store and query records, enforce access, render screens and printed documents, install feature packages with their shipped records, number documents, run background jobs, and authenticate users. No business object exists yet.

### Prerequisites

None. The runtime contracts in [`../runtime/transactions-and-concurrency.md`](../runtime/transactions-and-concurrency.md), [`../runtime/caching.md`](../runtime/caching.md), [`../runtime/scheduled-jobs.md`](../runtime/scheduled-jobs.md), [`../runtime/translation.md`](../runtime/translation.md) and [`../runtime/request-lifecycle.md`](../runtime/request-lifecycle.md) are inputs.

### Platform behaviors to deliver

The entities listed below are the visible part of step 1. The behaviors that carry them are the harder part, and they are what every later step assumes.

1. **The entity registry.** Entity definitions assembled from installed packages, with the three entity kinds, extension in place, prototype copying and delegation embedding, and a deterministic resolution order when several packages extend the same entity.
2. **The field system.** Every field type with its storage mapping and value semantics; declarable attributes; derived fields with declared dependencies; related fields; company-dependent values; translatable values; defaults and their resolution order; the archive flag; audit fields.
3. **The recomputation engine.** When a stored field is written, every derived field that declares a dependency on it, directly or through a relation, is marked for recomputation and recalculated before it is next read or written to storage. This is the single hardest piece of the platform to get right and it must be built first, because every later correctness property rests on it.
4. **Record sets.** An ordered collection of records of one entity that is both the value passed around and the receiver of every operation, supporting creation, reading, updating, deletion, searching, grouped reading with aggregation, name searching, copying, set operations, mapping, filtering and sorting.
5. **The filter notation.** Leaf conditions of field path, operator and value, combined with prefix logical operators, evaluated across related fields and hierarchies, with every operator's exact semantics.
6. **The unit of work.** A pending-write buffer with a defined flush order, a record cache with defined invalidation, a database transaction per request at the specified isolation level, and automatic retry on a serialization conflict.
7. **Sequences and numbering**, including the per-period reset and the gap rules.
8. **External identifiers**, the data loading grammar, the reload semantics and the no-update marking.
9. **Translatable values** with their fallback order, and the formatting of dates, numbers and monetary amounts per language.
10. **Identity and access.** Users and their kinds; groups, privilege families and the implied-group closure; per-entity access rights with the checking algorithm and its refusal messages; record rules with the combination rules for global and for group rules; field-level restrictions; the unrestricted actor and the elevate-privileges contract; company scope and the consistency checks; sessions and every authentication method; the settings mechanism that turns a configuration choice into installed packages, group membership and default values.
11. **Presentation and transport.** Every view kind and its grammar; view inheritance; window, server, client, address and report actions; menus; the generic operations with their exact arguments and return shapes; the error envelope; attachments with content addressing and access control; report rendering to a printable document.

**Why access control belongs here and not later.** Access control cannot be added after the queries are written without auditing every one of them. A record read that bypasses record rules is a data leak, and the only way to be sure that none exists is for the enforcement to predate the queries.

### Entities to deliver

**Registry and persistence.** Model Definition, Field Definition, Selection Value Definition, Model Constraint Registry, Model Inheritance Registry, Relation Table Registry, Properties Base Definition, Properties Base Definition Mixin, Base Model, Unknown, Field Value Converter, Decimal Precision.

**Packaging and data loading.** Module, Module Category, Module Dependency, Module Exclusion, Module Installation Request, Module Uninstall Wizard, Module Upgrade Wizard, Module Activation Review Wizard, Module List Update Wizard, Import Module Wizard, External Identifier, Demonstration Data Loader, Demonstration Data Failure, Demonstration Data Failure Wizard.

**Presentation.** View Definition, User Customized View, Reset View Architecture Wizard, Window Action, Window Action View Mode, Server Action, Server Action History, Server Action History Wizard, Client Action, Close Window Action, Web Address Action, Report Action, Report Layout, Paper Format, Embedded Action, Menu Item, Create Menu Wizard, Saved Filter, Action, Guided Tour, Tour's step, Onboarding Panel, Onboarding Step, Onboarding Progress, Onboarding Progress Step, Configuration Step, Web Asset.

**Rendering primitives.** Template Engine, Template Field Renderer and the field renderers for barcode, contact, date, datetime, duration, decimal, duration-as-time, rich text, image, integer, many to many, many to one, monetary, one to many, nested template, relative time, selection and text.

**Operations and housekeeping.** Sequence, Sequence Date Range, System Parameter, User Default Value, Scheduled Action, Scheduled Action Trigger, Scheduled Action Progress, Log Entry, Attachment, Binary Content Service, Automatic Cleanup, Performance Profile, Profiling Enablement Wizard, Export Template, Export Template Line, Translation Export Wizard, Translation Import Wizard, Language Installation Wizard, Configuration Settings, Configuration Wizard Base, Request Routing, Web Socket Handler, Key Performance Indicator Provider, Outgoing Mail Server, Avatar Mixin, Image Mixin, Address Format Mixin, Tax Identification Label Mixin, Module Reference Report.

**Identity and access.** User, User Preferences, User Settings Volumes, User Settings for Embedded Actions, Access Group, Access Privilege, Model Access Rule, Record Rule, User Application Key, User Application Key Creation Wizard, User Application Key Display Wizard, Password Change Wizard, Password Change User Line, Own Password Change Wizard, User Identity Check Wizard, Two-Factor Enrollment Wizard, Two-Factor Trusted Device, Time Based One Time Password rate limit logs Wizard, Passkey, Passkey Creation Wizard, Open Authorization Provider, Directory Server Configuration, User Login Log, User Device, User Device Log, User Deletion Request.

### Operations to deliver

`create_record`, `read_records`, `write_record`, `delete_record`, `duplicate_record`, `search_records`, `search_and_count`, `read_grouped`, `name_search`, `default_values_for_new_record`, `recompute_on_change`, `install_package`, `upgrade_package`, `uninstall_package`, `load_shipped_records`, `resolve_external_identifier`, `next_sequence_number`, `render_report`, `render_screen`, `export_records`, `import_records`, `run_scheduled_action`, `authenticate_user`, `change_own_password`, `enrol_second_factor`, `create_application_key`, `revoke_application_key`, `check_model_access`, `apply_record_rules`, `impersonate_for_scheduled_job`.

### Reports to deliver

Model Overview, Technical guide, Preview Internal Report, Preview External Report, Report Layout Preview.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Automatic data vacuum | daily | Deletes expired transient records, orphan attachments and expired one-time tokens, according to the retention rules of [`../runtime/`](../runtime/) and the scheduled actions of [platform foundation configuration](../domains/platform-foundation/configuration.md). |
| Portal user deletion | daily | Processes accepted User Deletion Request records: anonymizes or removes the account and writes a log entry. |
| Unregistered user reminder | daily | Sends one reminder message per user account that was invited but never signed in. |

### Interfaces to deliver

The remote transport contract of [`../interfaces/remote-transport-contracts.md`](../interfaces/remote-transport-contracts.md): the call envelope, the error envelope, batching, session handling and authentication levels. The service layer of [`../interfaces/service-layer.md`](../interfaces/service-layer.md): the generic entity operations exposed to clients, with the desktop behavior they support in [`../interfaces/desktop-workflows.md`](../interfaces/desktop-workflows.md) and the printable and exported documents of [`../interfaces/report-and-export-documents.md`](../interfaces/report-and-export-documents.md). The seventy-eight foundation routes of [`../../schemas/interfaces/routes.json`](../../schemas/interfaces/routes.json) whose domain is the platform foundation, covering session establishment, screen and action loading, binary content download and upload, report download, data export, language selection and health probes, as narrated in the [endpoint catalog](../interfaces/endpoint-catalog.md). The screen grammar of [platform foundation interfaces](../domains/platform-foundation/interfaces.md) for list, form, kanban, calendar, pivot, graph, activity, hierarchy and cohort presentations.

### Domain folders to read

1. [`../overview/`](../overview/) in full, in the order given by [`../overview/README.md`](../overview/README.md).
2. [platform foundation](../domains/platform-foundation/) in the order README, entities, business rules, state machines, workflows, calculations, configuration — which carries the numbering sequences, the attachment store and the scheduled actions — interfaces, which carries the screen grammar, then acceptance criteria and glossary.
3. [identity and access](../domains/identity-and-access/) in the order README, entities, business rules, state machines, workflows, configuration, interfaces, acceptance criteria.
4. [`../runtime/`](../runtime/) in full.
5. [`../data/persistence-identity-and-values.md`](../data/persistence-identity-and-values.md), [`../data/physical-data-catalog.md`](../data/physical-data-catalog.md) and [`../data/data-loading-and-exchange.md`](../data/data-loading-and-exchange.md).

### Notes on order inside the step

Deliver the registry and persistence before anything else, because the package loader stores its own records through it. Deliver access control before the package loader completes, because shipped access rules are records. Deliver sequences before any numbered document exists. Deliver attachments before reports, because a rendered report may be stored as an attachment.

### Decisions to make first

- **Eager or lazy recomputation.** Whether derived fields are recalculated at write time or at read time. The observable behavior specified here is lazy with a flush barrier: a derived value is correct whenever it is read and whenever it is written to storage. Either strategy can satisfy that, but the choice pervades the code and cannot be changed cheaply afterwards.
- **The flush order of the pending-write buffer.** The order decides which constraint fails first, and therefore which message a user sees.
- **How company-dependent values are stored**, as a per-company map on the record or as separate rows. Both are visible only through the same read and write contract, but the choice affects every query that filters on such a field.
- **Whether to serve the transport contract of this specification exactly, or to serve a new contract behind an adapter.** Serving it exactly lets existing integrations and existing clients keep working, at the cost of carrying its shape. The specification gives enough detail for either. Decide before writing the first endpoint: the conformance level claimed in [conformance profiles](conformance-profiles.md) follows from this choice.

### Gate

Milestone M1 of [milestones](milestones.md), and with it stage gates one, two and three. At the gate the platform can define an entity with stored, derived, related, company-dependent and translatable fields; create, read, update, delete and search its records; extend it from a second package; load and reload data by external identifier; refuse a user the records the rules exclude and say so in the specified words; serve menus, actions, views and the generic operations to a client; store and serve an attachment; render a printable document; and survive a concurrent write conflict by retrying.

---

## Step 2: Shared entities and the collaboration substrate

### Purpose

At the end of step 2 the platform knows who the business deals with, in which company, in which currency, at which address and in which language; every business record can carry a discussion thread with followers, tracked changes and planned activities; and the system can send electronic mail.

### Prerequisites

Step 1.

### Entities to deliver

**Contacts and organizations.** Contact, Company, Contact Tag, Industry, Bank, Bank Account, Country, Country Group, Country Subdivision, City, Language, Partner Grade, Partner Activation, Contact Merge Wizard, Contact Merge Line, Document Layout Wizard, Contact Enrichment, Geolocation Provider, Geocoder, Contact Autocomplete Service.

**Multi-currency.** Currency, Currency Rate.

**Calendar and scheduling.** Calendar Event, Calendar Recurrence, Calendar Attendee, Calendar Reminder, Calendar Event Tag, Calendar Filter, Calendar Provider Configuration, Calendar Event Deletion Wizard, Event Alarm Manager, and the two external calendar synchronization mixins with their account reset wizards.

**Collaboration substrate.** Discussion Thread Mixin, Message, Message Subtype, Message Reaction, Message Translation, Follower, Notification, Notification Bus, Notification Listener Mixin, Field Change Tracking Value, Tracking Duration Mixin, Activity, Activity Type, Activity Mixin, Activity Plan, Activity plan template, Activity schedule plan Wizard, Mail Activity Schedule Line Wizard, Email Template, Email Template Preview Wizard, Mail Template Reset Wizard, Template Reset Mixin, Rendering Mixin, Composer Mixin, Message Composer Wizard, Outgoing Email, Canned Response, Link Preview, Main Attachment Thread Mixin, Guest, User/Guest Presence.

### Operations to deliver

`create_contact`, `merge_contacts`, `validate_tax_identification_number`, `compute_company_hierarchy`, `convert_amount_between_currencies`, `refresh_currency_rates`, `post_message_on_thread`, `subscribe_follower`, `unsubscribe_follower`, `track_field_change`, `schedule_activity`, `mark_activity_done`, `cancel_activity`, `render_email_template`, `queue_outgoing_email`, `send_outgoing_email`, `create_calendar_event`, `expand_recurrence`, `send_event_reminder`.

### Reports to deliver

None of its own. The document layout, headers, footers and page formats configured here are used by every printed document from step 4 onward.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Electronic mail queue manager | hourly | Sends every Outgoing Email in the `outgoing` state whose scheduled moment has passed, marks each `sent` or `exception`, and writes the failure reason. |
| Calendar event reminder | daily | Creates the notifications and electronic mail for every Calendar Reminder that becomes due before the next run. |
| External calendar synchronization | every twelve hours, one job per external provider | Pulls and pushes changed events for every account that authorized synchronization. |
| Tax identification number verification refresh | daily | Re-checks contact tax identification numbers whose verification is older than the configured interval. |
| Notification cleanup | monthly | Deletes notification records older than six months. |

### Interfaces to deliver

The contact autocomplete and enrichment service contracts and the calendar synchronization connectors, both specified in [`../interfaces/external-integrations.md`](../interfaces/external-integrations.md). The notification bus channel contract of [`../runtime/notification-bus.md`](../runtime/notification-bus.md), which every later screen that waits for events uses.

### Domain folders to read

1. [`../domains/contacts-and-organizations/`](../domains/contacts-and-organizations/) in full.
2. [`../domains/multi-currency/`](../domains/multi-currency/) in full.
3. [messaging and activities](../domains/messaging-and-activities/): the README, the entities, the business rules, the workflows and the activity behavior, then the interfaces file, leaving discussion channels and live chat, the incoming electronic mail gateway, and text messages and postal mail for step 17.
4. [`../domains/calendar-and-scheduling/`](../domains/calendar-and-scheduling/) in full.
5. [`../overview/multi-company.md`](../overview/multi-company.md) for the company scoping rules that every later entity inherits.

### Notes on order inside the step

Deliver Currency and Currency Rate before Company, because a company requires a currency. Deliver Contact before Company, because a company record points at a contact record for its address and identity. Deliver the discussion thread before activities, because an activity posts a message when it is marked done.

**Why the collaboration substrate is here and not later.** Almost every business document of the later steps carries a thread, notifies its followers on a state change, tracks field changes for audit, and schedules activities. Building those documents first and adding messaging afterwards means revisiting all of them.

**Why currencies are here.** A currency carries the rounding discipline that every later amount depends on: its decimal places, its rounding step and the dated rate used to convert. Prove the conversion and the rounding against the worked examples of [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md) before any document carries an amount.

### Gate

Milestone M2, and the substrate half of stage gate four. A message posted on a document notifies exactly the right followers by the channel each follower's settings select; a tracked field change produces a tracking entry; an activity falls due, is marked done and chains its successor; a queued electronic mail is sent, marked and, on failure, marked with its reason; a scheduled job acquires its lock, runs and records its outcome; a recurring calendar event expands into the specified occurrences.

---

## Step 3: Products, units of measure and pricing

### Purpose

At the end of step 3 the catalog exists: things that can be bought, sold, stocked and priced, measured in convertible units, packed in packagings, described by attributes that generate variants, and priced by pricelists with rules.

### Prerequisites

Step 2.

### Entities to deliver

**Units of measure and packaging.** Unit of Measure, the unit of measure category grouping, Product Unit of Measure Link, Package Type.

**Products and catalog.** Product Template, Product Variant, Product Category, Product Attribute, Product Attribute Value, Product Template Attribute Line, Product Template Attribute Value, Product Attribute Custom Value, Product Attribute Exclusion, Product Tag, Product Document, Product Image, Product Combo, Product Combo Item, Barcode Nomenclature, Barcode Rule, Barcode Event Mixin, Product Catalog Mixin, Product Label Layout Wizard, Update product attribute value Wizard, Product Margin Wizard, Expiry Confirmation Wizard.

**Pricing.** Pricelist, Pricelist Rule, Vendor Price.

### Operations to deliver

`convert_quantity_between_units`, `round_quantity_to_unit_precision`, `create_product_template`, `generate_product_variants`, `archive_product_variant`, `compute_variant_extra_price`, `assign_barcode`, `parse_barcode_with_nomenclature`, `compute_price_from_pricelist`, `select_applicable_pricelist_rule`, `compute_vendor_price`, `print_product_labels`, `print_pricelist_report`.

### Reports to deliver

Product Label in the four shipped layouts (two by seven, four by seven, four by twelve, four by twelve without price), Packaging Barcodes, Pricelist.

### Scheduled jobs to deliver

None.

### Interfaces to deliver

The catalog screens: product list, product form with the variant matrix, attribute configuration, the price computation preview, and the label printing dialog. The four catalog endpoints for barcode lookup and product image download.

### Domain folders to read

1. [`../domains/units-of-measure-and-packaging/`](../domains/units-of-measure-and-packaging/) in full, first. Every quantity in the system is a quantity in a unit of measure, and the conversion and rounding rules of this domain are used by inventory, purchasing, sales, manufacturing and the point of sale.
2. [`../domains/products-and-catalog/`](../domains/products-and-catalog/) in full.
3. [`../domains/pricing-and-pricelists/`](../domains/pricing-and-pricelists/) in full.
4. [`../data/reference-data.md`](../data/reference-data.md) for the shipped reference records this step and step 2 must load, with their external identifiers.

### Notes on order inside the step

Deliver units of measure before products: a product carries a stock unit of measure and a purchase unit of measure, and the constraint that both belong to the same category is a product-level validation that needs the category to exist. Deliver attributes and variant generation before pricing, because a pricelist rule may target a variant. Deliver the vendor price before purchasing but after the product: it is catalog data, not a purchasing document.

**Why the arithmetic of this step is the riskiest part of it.** Units carry the rounding discipline that every later quantity depends on, as currencies do for every later amount. Getting the conversion and the rounding right here, with the worked examples of [`../domains/units-of-measure-and-packaging/calculations.md`](../domains/units-of-measure-and-packaging/calculations.md) and [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md) as tests, prevents a class of penny errors that is very hard to find later.

### Gate

Milestone M3, and with it stage gate five. Every shipped reference record loads with its specified values; a quantity converts between units in both directions with the specified rounding; an amount converts between currencies at the rate in force on a stated date; variant generation produces exactly the specified combinations; and a pricelist produces the specified price for a given product, quantity, date and currency.

---

## Step 4: General ledger

### Purpose

At the end of step 4 the platform keeps double-entry books: a chart of accounts, journals, balanced journal entries with items, numbering per journal with gap detection, posting and reversal, lock dates, an audit trail with inalterability hashing, and reconciliation of items.

### Prerequisites

Step 2. Step 3 is not required: the ledger itself does not know products.

### Entities to deliver

Account, Account Group, Account Root, Account Code Mapping, Account Merge Wizard, Account merge wizard line, Journal, Journal Group, Journal Entry, Journal Item, Sequence Mixin, Resequencing Wizard, Lock Date Exception, Entry Securing Wizard, Journal Entry Validation Wizard, Automatic Entry Wizard, Accrued Orders Wizard, Opening Balance of Financial Year Wizard, Chart of Accounts Template, Document Import Mixin, Accounting Assert Test, Invoices Statistics, Account Test Report, the two account report variants with and without payment lines, and the hash integrity report.

### Operations to deliver

`post_journal_entry`, `reset_entry_to_draft`, `cancel_journal_entry`, `reverse_journal_entry`, `duplicate_journal_entry`, `resequence_entries`, `check_sequence_gaps`, `set_lock_date`, `grant_lock_date_exception`, `revoke_lock_date_exception`, `hash_posted_entries`, `verify_hash_chain`, `reconcile_items`, `unreconcile_items`, `create_automatic_entry`, `accrue_orders`, `load_chart_of_accounts_template`, `merge_accounts`, `open_financial_year`, `run_accounting_assert_tests`.

### Reports to deliver

Invoice document, Original Bills, Document without Payment, Payment Receipt, Statement, Hash integrity result, Accounting Tests.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Post scheduled draft entries | daily | Posts every draft Journal Entry flagged for automatic posting whose accounting date is today or earlier; failures are reported on the entry and in the discussion thread. |
| Send scheduled documents | daily | Sends the documents queued for automatic sending by the sending service. |

### Interfaces to deliver

The twelve ledger endpoints, covering the audit trail download, the hash integrity download and the document preview by access token. The screens: chart of accounts list with the code mapping per company, journal configuration, the journal entry form with its item grid and its status bar, the reconciliation view for items of one account and one partner, the lock date dialog.

### Domain folders to read

1. [`../domains/general-ledger/`](../domains/general-ledger/) in full, in the order README, entities, business-rules, calculations, workflows, accounting-effects, configuration, interfaces, acceptance-criteria, glossary.
2. The accounting identity section of [`../data/domain-model.md`](../data/domain-model.md).

### Notes on order inside the step

Deliver the account and the journal before the entry. Deliver numbering, with its per-journal and per-period sequence derivation, before posting, because posting assigns the number. Deliver the balance constraint, the currency consistency constraints and the lock date guards before reversal, because reversal is a posting. Deliver reconciliation last inside the step: it depends on posted items and on the partial reconciliation entity that step 6 extends with payment semantics.

### Gate

Milestone M4. Every posted entry balances, per entry and per currency, with no exception; posting assigns a number in the specified format with gaps only where they are permitted; a lock date refuses a posting inside the locked period with the specified message and an exception grants exactly what it says; reversal produces the exact opposite entry; and reconciling two items leaves the specified residual.

---

## Step 5: Taxes, fiscal positions and analytic accounting

### Purpose

At the end of step 5 every amount can carry tax. The tax engine computes tax on a line, in a document, price-included or price-excluded, in chains, in groups, with fixed and percentage and division amounts, with distribution to accounts and tags, and with a fiscal position that substitutes taxes and accounts per customer or supplier. Analytic accounting can attribute any amount to one or several analytic accounts across several plans.

### Prerequisites

Step 4.

### Entities to deliver

**Taxes.** Tax, Tax Group, Tax Distribution Line, Account Tag, Fiscal Position, Fiscal Position Account Mapping, Update Tax Tags Wizard, Financial Report, Financial Report Line, Financial Report Expression, Financial Report Column, Financial Report External Value.

**Analytic accounting.** Analytic Plan, Analytic Account, Analytic Line, Analytic Distribution Model, Analytic Applicability Rule, Analytic Mixin, Analytic Plan Fields Mixin.

### Operations to deliver

`compute_taxes_for_line`, `compute_taxes_for_document`, `flatten_tax_hierarchy`, `apply_fiscal_position_to_taxes`, `apply_fiscal_position_to_account`, `distribute_tax_amount_to_lines`, `compute_cash_basis_entries`, `aggregate_tax_report`, `apply_analytic_distribution`, `create_analytic_lines_from_items`, `select_analytic_distribution_model`, `validate_mandatory_analytic_plan`.

### Reports to deliver

The tax return structures of the Financial Report family, rendered as a printed document and as a spreadsheet export. The generic financial statements built on the same engine: balance sheet, income statement, partner ledger, aged receivable and aged payable, general ledger, trial balance.

### Scheduled jobs to deliver

None of its own.

### Interfaces to deliver

The tax configuration screens, the fiscal position screen with its tax and account mapping grids, the analytic plan and analytic account screens, the analytic distribution widget contract used by every document line from step 6 onward, and the financial report viewer contract with its expansion, comparison and export behavior.

### Domain folders to read

1. [taxes](../domains/taxes/) in full. Read [`calculations.md`](../domains/taxes/calculations.md) twice: the rounding primitives, the batching rules and the two rounding methods are the single most-cited behavior in the whole specification.
2. [`../domains/analytic-accounting/`](../domains/analytic-accounting/) in full.
3. [`../domains/financial-reporting/`](../domains/financial-reporting/) in full: the report engine delivered here renders the tax returns and the generic financial statements alike.
4. The date and rounding conventions used by both, in [`../domains/multi-currency/calculations.md`](../domains/multi-currency/calculations.md) and [`../domains/general-ledger/calculations.md`](../domains/general-ledger/calculations.md).

### Notes on order inside the step

Deliver the rounding primitives before anything else in the step, and prove them against the worked examples before continuing. Deliver the single-line computation before the document computation. Deliver the distribution lines before the tax report, because report lines aggregate by tag. Deliver analytic plans before analytic accounts and both before the distribution widget.

**Why the tax engine comes before invoices.** An invoice's totals are the tax engine's output. Building invoices against an approximate tax engine produces an invoice model shaped around the approximation, and every later document inherits that shape.

### Gate

Milestone M5. The tax engine reproduces every worked example — percentage, fixed, division and group taxes, price-included extraction, base-amount chaining, and per-line against global rounding on the three-line case, to the cent; a fiscal position substitutes taxes and accounts on a document and on its items; and an analytic distribution produces lines whose signed amounts sum to the originating item.

---

## Step 6: Receivables, payables, payments and payment providers

### Purpose

At the end of step 6 the business can invoice a customer, receive a vendor bill, send documents, take and make payments, register them against documents, import bank statements and reconcile them, and accept online payments through a provider.

### Prerequisites

Step 5. Step 3 for the product-driven invoice line defaults.

### Entities to deliver

**Receivables.** The customer-facing document types of the Journal Entry entity (customer invoice, credit note, receipt), Cash Rounding, International Commercial Term, Document Sending Service, Document Sending Wizard, Batch Document Sending Wizard, Journal Entry Reversal Wizard, Payment Refund Wizard.

**Payables.** The supplier-facing document types of the Journal Entry entity (vendor bill, refund, debit note), Add Debit Note wizard, Automatic Bill Posting Wizard, Print Pre-numbered Checks Wizard.

**Payments and reconciliation.** Payment, Payment Method, Payment Method Line, Payment Terms, Payment Terms Line, Payment Registration Wizard, Payment register withholding line Wizard, Payment withholding line, Bank Statement, Bank Statement Line, Reconciliation Model, Reconciliation Model Line, Partial Reconciliation, Full Reconciliation, Bank Account Setup Wizard.

**Payment providers.** Payment Provider, Payment Method, Payment Transaction, Payment Token, Payment Link Wizard, Payment Capture Wizard.

### Operations to deliver

`create_customer_invoice`, `validate_invoice`, `send_and_print_document`, `create_credit_note`, `reverse_and_modify_invoice`, `create_vendor_bill`, `import_bill_from_document`, `match_bill_to_purchase_order`, `create_debit_note`, `register_payment`, `post_payment`, `cancel_payment`, `refund_payment`, `void_check`, `print_checks`, `import_bank_statement`, `reconcile_statement_line`, `apply_reconciliation_model`, `undo_statement_reconciliation`, `compute_payment_terms_schedule`, `compute_early_payment_discount`, `apply_cash_rounding`, `create_payment_transaction`, `authorize_transaction`, `capture_transaction`, `void_transaction`, `refund_transaction`, `tokenize_payment_method`, `generate_payment_link`.

### Reports to deliver

Invoice with payments, Payment Receipt, Statement, printed checks in the shipped check layouts.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Post-process payment transactions | every ten minutes | Finalizes transactions left in an intermediate state by an interrupted callback: confirms, cancels or marks them in error, and posts the resulting payment. |
| Send invoices automatically | daily | Sends the invoices queued by the sending service with the chosen channels. |

### Interfaces to deliver

The payment provider connector contract of [`../domains/payment-providers/interfaces.md`](../domains/payment-providers/interfaces.md): the redirect form, the direct charge, the tokenization, the webhook signature verification and the state mapping, for each shipped provider family. The sixty-two payment endpoints covering checkout, redirect return, webhook receipt and token management. The five receivable endpoints for portal document viewing and online payment of an invoice. The invoice, payment and reconciliation screens, including the reconciliation workbench.

### Domain folders to read

1. [`../domains/accounts-receivable/`](../domains/accounts-receivable/) in full.
2. [`../domains/accounts-payable/`](../domains/accounts-payable/) in full.
3. [payments and bank reconciliation](../domains/payments-and-bank-reconciliation/) in full, including the matching engine of its workflows and the quick response codes and structured payment references of its interfaces.
4. [payment providers](../domains/payment-providers/) in full, including the provider connector contracts of its [`interfaces.md`](../domains/payment-providers/interfaces.md).
5. [customer portal](../domains/customer-portal/) may be read now or deferred to step 19; the portal access token rules are used by the payment links delivered here.

### Notes on order inside the step

Deliver payment terms before invoices, because the due date schedule of an invoice comes from its payment terms. Deliver the dynamic line synchronization that keeps a document's product lines, tax lines and term lines consistent before any document is posted, because every later correction of a posted document depends on it. Deliver the payment and its outstanding accounts before reconciliation. Deliver reconciliation before early payment discounts and before exchange differences, because both are write-off lines created during reconciliation. Deliver the provider abstraction before any concrete provider.

### Gate

Milestone M6, and with it stage gate six, whose country and exchange part is re-checked at step 20. An invoice's term lines distribute the total exactly, with the remainder on the last instalment; a payment reconciles and leaves the specified residual; a foreign-currency payment produces the specified exchange difference; a provider callback that arrives twice has its effect once; and the statements balance.

---

## Step 7: Inventory operations

### Purpose

At the end of step 7 the business moves goods: warehouses with locations, operation types, transfers with moves and move lines, reservation against quantities on hand, lots and serial numbers, packages, batch and wave transfers, putaway and removal strategies, scrap, inventory adjustments, and delivery with shipping methods.

### Prerequisites

Steps 3 and 4.

### Entities to deliver

Warehouse, Location, Storage Category, Storage Category Capacity, Putaway Rule, Operation Type, Transfer, Stock Move, Stock Move Line, Stock Quantity Record, Lot or Serial Number, Package, Stock Package History, Package Type (extended), Batch Transfer, Batch Transfer Lines Wizard, Wave Transfer Lines Wizard, Scrap Order, Scrap Reason Tag, Shipping Method, Shipping Price Rule, Shipping Method Selection Wizard, Delivery Postal Code Prefix, Put In Pack Wizard, Package Destination Wizard, Stock Relocation Wizard, Return Wizard, Return Wizard Line, Backorder Confirmation Wizard, Backorder Confirmation Line Wizard, Inventory Adjustment Naming Wizard, Inventory Conflict Wizard, Inventory Warning Wizard, Count Request Wizard, Quantity History Wizard, Insufficient Quantity Warning, Insufficient Quantity Warning for Scrap, Product Replenish Wizard, Product Replenish Mixin, Confirm Stock Text Message Wizard, Reference between stock documents, Stock Quantity Report, Lot Label Report, Product Label Report, Stock Reception Report, Stock rule report, Traceability Report.

### Operations to deliver

`confirm_transfer`, `check_availability`, `unreserve_transfer`, `validate_transfer`, `create_backorder`, `cancel_transfer`, `return_transfer`, `split_transfer`, `put_in_pack`, `assign_package_destination`, `scrap_quantity`, `apply_inventory_adjustment`, `request_count`, `relocate_quantities`, `generate_lot_numbers`, `assign_lot_to_move_line`, `apply_putaway_rule`, `apply_removal_strategy`, `merge_moves`, `split_move`, `compute_forecast_availability`, `rate_shipment`, `send_shipment`, `cancel_shipment`, `get_return_label`, `print_delivery_slip`, `confirm_batch_transfer`, `validate_batch_transfer`.

### Reports to deliver

Reception Report and Reception Report Label, Picking Operations, Delivery Slip, Return slip, Batch Transfer, Packages, Count Sheet, Package Barcode and Package Barcode with Contents, Location Barcode, Lot and Serial Number label, Operation type label, Product Routes Report, Product Label, Packaging Barcodes.

### Scheduled jobs to deliver

The scheduler job is delivered in step 9; step 7 delivers its hook only.

### Interfaces to deliver

The three inventory endpoints for barcode screens and label download. The operational screens: transfer list grouped by operation type with the availability status bar, the detailed operation grid with lots and packages, the quantity-on-hand view with editable counted quantity, the traceability report, the forecast report. The shipping connector contract: rate request, shipment creation, label retrieval, tracking link and cancellation.

### Domain folders to read

1. [inventory operations](../domains/inventory-operations/) in full, including warehouse configuration, lots and serial numbers, packages, inventory adjustments, and batch and wave transfers.
2. [delivery and shipping](../domains/delivery-and-shipping/) in full, for the shipping methods, their price rules and the carrier connectors.
3. [`../domains/units-of-measure-and-packaging/calculations.md`](../domains/units-of-measure-and-packaging/calculations.md) again, for the quantity rounding rules applied to every move.

### Notes on order inside the step

Deliver locations and the quantity record before moves: reservation writes reserved quantity on the quantity record. Deliver the move state machine and its propagation to the transfer state before backorders. Deliver lots before removal strategies that order by lot date. Deliver packages before batch transfers. Deliver shipping methods last inside the step.

### Gate

Milestone M7. Quantity is conserved: for every product and location, the quantity on hand equals the signed sum of completed movements; reservation follows the specified removal ordering; reserved quantity never exceeds the quantity on hand and never goes negative; and a partial validation produces the specified backorder or none, according to the operation type's policy.

---

## Step 8: Inventory valuation and costing

### Purpose

At the end of step 8 every stock movement has a value, every product has a cost maintained by its costing method, and, when the valuation is automated, every movement posts a journal entry.

### Prerequisites

Steps 7 and 5.

### Entities to deliver

Stock Valuation, Product Value, Stock Average Cost Justifier, Landed Cost, Landed Cost Line, Valuation Adjustment Line.

### Operations to deliver

`classify_movement_for_valuation`, `value_incoming_movement`, `value_outgoing_movement`, `run_first_in_first_out_consumption`, `update_average_cost`, `replay_average_cost`, `maintain_product_cost`, `revalue_product`, `compute_landed_cost`, `validate_landed_cost`, `post_valuation_entry`, `post_closing_entry`, `justify_average_cost`.

### Reports to deliver

The stock valuation statement, printable and exportable as a spreadsheet.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Inventory valuation closing | daily | For every company using periodic valuation, computes the difference between the physical value and the accounting value per product category and posts the closing entry. |

### Interfaces to deliver

The valuation screens: the layer list per product with quantity, unit cost and remaining quantity; the landed cost form with its adjustment grid; the revaluation dialog.

### Domain folders to read

1. [inventory valuation and costing](../domains/inventory-valuation-and-costing/) in full, including landed costs and lot valuation.
2. [`../domains/general-ledger/accounting-effects.md`](../domains/general-ledger/accounting-effects.md) for the posting conventions the valuation entries follow.

### Notes on order inside the step

Deliver the classification of a movement (incoming, outgoing, internal, dropship) before any valuation formula. Deliver standard costing first, then average cost, then first in first out, in that order: each adds one mechanism to the previous. Deliver the account selection precedence (product, product category, company default) before posting. Deliver landed costs last.

### Gate

Milestone M8. Each costing method produces the specified value and unit cost for every worked example, including the negative-stock correction; the balance of the valuation account equals the sum of the valuation layers, continuously; and a landed cost apportions by each split method exactly as specified.

---

## Step 9: Replenishment and procurement

### Purpose

At the end of step 9 the system decides by itself what to buy, make or move: routes and rules propagate demand, reordering rules trigger replenishment, make-to-order chains a supply to each demand, lead times schedule the dates, and the scheduler runs the whole loop.

### Prerequisites

Step 7. The buy rule needs step 10 and the manufacture rule needs step 12; deliver the rule framework here and the two action types when their target document exists.

### Entities to deliver

Route, Stock Rule, Reordering Rule, Reordering Rule Snooze Wizard, Replenishment Information, Replenishment Option, Forecasted Stock Report, Stock Replenishment Report, Stock Rules Report, Vendor Delay Report.

### Operations to deliver

`run_procurement`, `select_rule_for_demand`, `run_pull_rule`, `run_push_rule`, `run_buy_rule`, `run_manufacture_rule`, `run_scheduler`, `compute_reordering_quantity`, `snooze_reordering_rule`, `compute_forecasted_quantity`, `compute_lead_times`, `link_supply_to_demand`, `cancel_propagated_supply`, `open_replenishment_report`.

### Reports to deliver

The replenishment report and the forecast report, both as screens with a printable form.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Procurement scheduler | daily | Processes every reordering rule and every open procurement in the order specified by the domain, creates the supply documents, reschedules the existing ones and writes the exception messages on documents it cannot satisfy. |

### Interfaces to deliver

The replenishment screen with its suggested quantities and the ordering actions, the forecast screen with its period breakdown, the route configuration screen, and the rules diagram.

### Domain folders to read

1. [`../domains/replenishment-and-procurement/`](../domains/replenishment-and-procurement/) in full.
2. [`../domains/inventory-operations/workflows.md`](../domains/inventory-operations/workflows.md) again, for the move creation contract that a rule uses.

### Notes on order inside the step

Deliver the rule resolution order before any rule action. Deliver lead time arithmetic before scheduling. Deliver the exception message and the scheduler's ordering guarantees before the automatic run, because the observable behavior of the scheduler is defined by the order in which it processes demands.

**Why quantities come before rules.** The purpose of the rule engine is to create moves. Building it before moves behave correctly means debugging two unfinished systems at once, and every defect is attributed to the wrong one.

### Gate

Milestone M9. A reordering rule proposes the specified quantity, rounded up to its multiple; the scheduler groups procurements into the specified number of documents with the specified lines; lead times produce the specified dates for a pick, pack and ship chain and for a purchase; and a second run of the scheduler with nothing changed creates nothing.

---

## Step 10: Purchasing

### Purpose

At the end of step 10 the business can ask suppliers for prices, order from them, receive the goods, control the bill against the order and the receipt, and analyze what it bought.

### Prerequisites

Steps 6, 8 and 9.

### Entities to deliver

Purchase Order, Purchase Order Line, Purchase Agreement, Purchase Agreement Line, Purchase Bill Union, Purchase Bill Line Matching, Bill to Purchase Order Wizard, Wizard to preset values for alternative Purchase Order, Wizard for open alternative requests, the grouping entity for calls to tender, Purchase Analysis Report.

### Operations to deliver

`create_request_for_quotation`, `send_request_for_quotation`, `confirm_purchase_order`, `approve_purchase_order`, `lock_purchase_order`, `unlock_purchase_order`, `cancel_purchase_order`, `reset_purchase_order_to_draft`, `create_alternative_quotation`, `compare_alternatives`, `create_vendor_bill_from_order`, `match_bill_lines`, `receive_purchase_order`, `compute_received_quantity`, `compute_billed_quantity`, `apply_vendor_price`, `confirm_purchase_agreement`, `close_purchase_agreement`, `send_purchase_reminder`.

### Reports to deliver

Purchase Order, Request for Quotation, Purchase Agreements.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Purchase reminder | daily | Sends the confirmation-date reminder to the vendor of every confirmed order whose receipt date approaches, and records the answer. |

### Interfaces to deliver

The six purchasing endpoints for vendor acknowledgement and date confirmation by token. The purchasing screens: the order form with its line grid, the catalog picker, the receipt smart button, the billing status, the alternative comparison view, and the purchase analysis pivot.

### Domain folders to read

1. [`../domains/purchasing/`](../domains/purchasing/) in full.
2. [accounts payable](../domains/accounts-payable/) again for the bill control policy consequences.
3. The purchase-order source of the incoming value chain, in [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md).

### Notes on order inside the step

Deliver the order and its lines before the receipt link, and the receipt link before the bill control policy, because the policy reads received quantities. Deliver the three-way matching last inside the step.

### Gate

Milestone M10. Each bill control policy proposes the specified quantity; three-way matching marks an order fully billed only when the billed quantity equals the received quantity on every line; and a bill posted at a price other than the receipt price posts the specified price difference.

---

## Step 11: Sales and promotions

### Purpose

At the end of step 11 the business can quote, sell, deliver and invoice: quotations with templates and optional lines, orders with delivery and invoicing policies, down payments, margins, sales teams, and loyalty programs, coupons, gift cards and discounts applied to an order.

### Prerequisites

Steps 6, 7 and 9.

### Entities to deliver

**Sales.** Sales Order, Sales Order Line, Quotation Template, Quotation Template Line, Quotation Document, Quotation Document Form Field, Down Payment Invoice Wizard, Sales Order Discount Wizard, Quotation Partner Wizard, Mass Order Cancellation Wizard, Sales Analysis Report.

**Loyalty and promotions.** Loyalty Program, Loyalty Rule, Loyalty Reward, Loyalty Card, Loyalty History, Loyalty Communication, Loyalty Coupon Generation Wizard, Loyalty Reward Selection Wizard, Update Loyalty Card Points Wizard, the coupon-link creation wizard, the order-to-coupon points link entity, and the coupon application wizard.

### Operations to deliver

`create_quotation`, `apply_quotation_template`, `send_quotation`, `confirm_sales_order`, `lock_sales_order`, `unlock_sales_order`, `cancel_sales_order`, `reset_quotation_to_draft`, `create_invoice_from_order`, `create_down_payment_invoice`, `apply_order_discount`, `update_prices_from_pricelist`, `update_taxes_from_fiscal_position`, `compute_delivered_quantity`, `compute_invoiced_quantity`, `compute_margin`, `assign_sales_team`, `apply_loyalty_program`, `claim_reward`, `generate_coupons`, `redeem_gift_card`, `expire_loyalty_points`.

### Reports to deliver

Quotation or Order, Pro-forma Invoice, the quotation document built from uploaded pages, Coupon Code, Gift Card.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Automatic invoicing | daily | Creates and sends the invoices of confirmed orders configured for automatic invoicing whose conditions are met. |
| Send pending quotation electronic mail | daily | Sends the quotations queued for delayed sending. |

### Interfaces to deliver

The nineteen sales endpoints covering the customer preview of a quotation, online signature, online payment and the acceptance or refusal of an order. The sales screens: the quotation form with its section and note lines, the optional-line grid, the product catalog picker, the delivery and invoice smart buttons, the sales analysis pivot.

### Domain folders to read

1. [`../domains/sales/`](../domains/sales/) in full.
2. [loyalty and promotions](../domains/loyalty-and-promotions/) in full, including its counter and storefront applications, which are re-read in steps 13 and 19.
3. [`../domains/pricing-and-pricelists/calculations.md`](../domains/pricing-and-pricelists/calculations.md) again for the price resolution order used on every line.

### Notes on order inside the step

Deliver the order line quantity and price derivations before the delivery link, and both before the invoicing policy. Deliver down payments after ordinary invoicing. Deliver loyalty last inside the step: its rewards create order lines and therefore depend on the complete line behavior.

### Gate

Milestone M11. Confirming an order creates exactly the specified records and statuses; invoicing on ordered and on delivered quantities each produce the specified lines and quantities; an advance invoice deducts exactly on the final invoice, with the specified line and tax treatment; and a promotion and a gift card on one order produce the specified reward lines, prices and tax treatment.

---

## Step 12: Manufacturing, repair and maintenance

### Purpose

At the end of step 12 the business can make what it sells: bills of materials with components, by-products and operations, manufacturing orders that consume and produce, work orders on work centers with time tracking, unbuilding, subcontracting, and the repair and maintenance of items.

### Prerequisites

Steps 8 and 9.

### Entities to deliver

**Manufacturing.** Bill of Materials, Bill of Materials Line, Bill of Materials By-Product, Operation, Manufacturing Order, Work Order, Work Center, Work Center Capacity, Work Center Tag, Work Center Productivity Record, Work Center Productivity Loss, Work Center Productivity Loss Type, Unbuild Order, Production Group, Manufacturing Order Split Wizard, Manufacturing Order Split Line, Multiple Manufacturing Order Split Wizard, Manufacturing Backorder Wizard, Manufacturing Backorder Line, Production Quantity Change Wizard, Consumption Warning Wizard, Consumption Warning Line, Assign serial numbers to production order Wizard, Warn Insufficient Unbuild Quantity Wizard, the two work-in-progress posting wizards, Bill of Materials Overview Report, Manufacturing Order Overview Report.

**Repair and maintenance.** Repair Order, Repair Tag, Insufficient Quantity Warning for Repair, Maintenance Request, Maintenance Stage, Maintenance Team, Equipment, Equipment Category, Maintenance Maintained Item.

### Operations to deliver

`explode_bill_of_materials`, `confirm_manufacturing_order`, `plan_manufacturing_order`, `unplan_manufacturing_order`, `reserve_components`, `start_work_order`, `pause_work_order`, `finish_work_order`, `record_production`, `produce_and_close`, `create_manufacturing_backorder`, `split_manufacturing_order`, `merge_manufacturing_orders`, `unbuild_order`, `scrap_from_production`, `post_work_in_progress`, `confirm_repair_order`, `start_repair`, `end_repair`, `invoice_repair`, `create_maintenance_request`, `close_maintenance_request`, `compute_next_preventive_maintenance`.

### Reports to deliver

Production Order, Bill of Materials Overview, Manufacturing Order Overview, Finished Product Label in both layouts, Work Order, Work in Progress report, Repair Order.

### Scheduled jobs to deliver

None of its own; the manufacture rule is driven by the procurement scheduler of step 9.

### Interfaces to deliver

The four manufacturing endpoints for shop-floor screens and label download. The screens: the bill of materials form with its structure and cost overview, the manufacturing order form with component and by-product grids, the work order list per work center, the shop floor board, the equipment and maintenance request boards.

### Domain folders to read

1. [manufacturing](../domains/manufacturing/) in full, including the explosion of a bill of materials, work orders and their scheduling, and subcontracting.
2. [`../domains/repair-and-maintenance/`](../domains/repair-and-maintenance/) in full.
3. The production source of the incoming value chain, in [`../domains/inventory-valuation-and-costing/calculations.md`](../domains/inventory-valuation-and-costing/calculations.md).

### Notes on order inside the step

Deliver the explosion algorithm before the manufacturing order, because the order's components come from it. Deliver the consumption of components before the production of the finished item, because the valuation of the finished item is the sum of the consumed values plus the operation costs. Deliver work orders after the order is complete without them: a manufacturing order must work when no operations are defined.

### Gate

Milestone M12, and with it stage gate seven. A bill of materials explodes with correct unit conversion and a cycle is refused with the specified message; a production consumes, produces and values as specified, including by-product cost shares and a backorder; and the production account of a completed order returns to zero.

---

## Step 13: Point of sale

### Purpose

At the end of step 13 a cashier can open a session, sell to walk-in customers with a touch interface that keeps working without a network connection, take payments in cash, by card through a terminal and on customer account, print or send receipts, and close the session with a cash count and a correct set of journal entries.

### Prerequisites

Steps 6, 7 and 11.

### Entities to deliver

Point of Sale Configuration, Point of Sale Session, Point of Sale Order, Point of Sale Order Line, Point of Sale Payment, Point of Sale Payment Method, Point of Sale Category, Point of Sale Note, Point of Sale Preset, Point of Sale Printer, Point of Sale Cash Denomination, Point of Sale Lot Operation, Point of Sale Session Closing Wizard, Point of Sale Payment Wizard, Point of Sale Daily Sales Report Wizard, Point of Sale Sales Details Wizard, Multiple order invoice creation Wizard, Point of Sale Data Loading Mixin, Point of Sale Orders Report, Point of Sale Invoice Report, Point of Sale Details, the per-employee session sales detail, Restaurant Floor, Restaurant Table, Point of Sale Restaurant Order Course, the self-order custom links entity, and the preparation display entities.

### Operations to deliver

`open_session`, `load_session_data`, `create_order_offline`, `synchronize_orders`, `add_payment_to_order`, `validate_order`, `invoice_order`, `refund_order`, `settle_customer_account`, `close_session`, `count_cash`, `post_session_entry`, `create_session_transfers`, `open_rescue_session`, `print_receipt`, `send_receipt`, `open_terminal_payment`, `cancel_terminal_payment`, `assign_table`, `split_bill`, `transfer_order_between_tables`, `fire_course`, `mark_preparation_done`.

### Reports to deliver

Sales Details, User Labels, Quick Response Codes for self-ordering.

### Scheduled jobs to deliver

None.

### Interfaces to deliver

The forty-five point of sale endpoints: session loading, order synchronization, self-ordering, kiosk mode, customer display, preparation display and payment terminal callbacks. The offline contract: what the client caches, how it reconciles conflicting order numbers, and what happens when the connection returns. The payment terminal connector contract for each shipped family.

### Domain folders to read

1. [point of sale](../domains/point-of-sale/) in full, including session closing accounting, restaurant operations, self-ordering and the payment terminal connectors.
2. The counter application of [loyalty and promotions](../domains/loyalty-and-promotions/).
3. [`../domains/inventory-operations/workflows.md`](../domains/inventory-operations/workflows.md) for the delivery created at session closing.

### Notes on order inside the step

Deliver the session and its state machine before orders. Deliver the order, its lines and its taxes before payments. Deliver the closing entry last inside the step and prove it against the accounting scenarios before any restaurant or self-ordering feature is added.

**Note on the counter channel.** The counter application computes prices, taxes, discounts and rounding on the operator's device, often with no network, and reconciles later. Its arithmetic must match the server's exactly, or a session will not close. Treat the counter's calculation rules and the tax engine as one requirement with two implementations that are tested against each other.

### Gate

Milestone M13. A counter session closes with the specified entry, line by line, including the cash difference and the excluded invoiced orders; orders created while the counter is offline synchronize once, with no duplicate on replay; and the counter's price, tax, discount and rounding results match the server's for every worked example.

---

## Step 14: Customer relationship management

### Purpose

At the end of step 14 the business tracks demand before it becomes an order: leads, opportunities, pipelines, teams, assignment rules, scoring, lost reasons, merging and recurring revenue plans.

### Prerequisites

Step 11.

### Entities to deliver

Lead, Pipeline Stage, Sales Team, Sales Team Member, Sales Tag, Lost Reason, Lead to Opportunity Conversion Wizard, Mass Lead to Opportunity Conversion Wizard, Opportunity Merge Wizard, Lead Lost Wizard, Lead Assignment Wizard, Lead Forward to Partner Wizard, Recurring Revenue Plan, Lead Scoring Frequency, Lead Scoring Frequency Field, Predictive Lead Scoring Update Wizard, Lead Mining Request, Lead Mining Industry, Lead Mining Role, Lead Mining Seniority, the lead generation rules entity, the reveal view entity, Event Lead Rules, Event Lead Request, Activity Analysis Report, the partnership analysis report.

### Operations to deliver

`create_lead`, `convert_lead_to_opportunity`, `merge_opportunities`, `mark_opportunity_won`, `mark_opportunity_lost`, `restore_lost_opportunity`, `assign_leads_by_rule`, `compute_predictive_probability`, `recompute_scoring_frequencies`, `request_lead_mining`, `forward_lead_to_partner`, `create_quotation_from_opportunity`, `compute_expected_revenue`.

### Reports to deliver

None of its own; the pipeline is analyzed through the shipped pivot and graph screens.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Recompute automated probabilities | daily | Rebuilds the scoring frequency table from won and lost leads and refreshes the automated probability of every open lead. |
| Lead assignment | daily | Distributes unassigned leads to teams and salespeople according to the assignment rules and their maximum capacity. |
| Lead enrichment | daily | Enriches leads that requested enrichment, using the contact enrichment service. |
| Generate leads from event rules | daily | Creates leads from event registrations that match an event lead rule. |
| Lead generation from mining requests | daily | Produces leads for every active mining request within its quota. |

### Interfaces to deliver

The fourteen endpoints of the domain, covering the lead capture forms and the partner assignment pages. The pipeline kanban with its drag-and-drop stage change, the lead form with its conversion action, the assignment configuration screen, and the scoring explanation panel.

### Domain folders to read

1. [customer relationship management](../domains/customer-relationship-management/) in full, including lead assignment and predictive lead scoring.

### Notes on order inside the step

Deliver the lead and its stages before teams, and teams before assignment. Deliver conversion before merging, because merging reuses the conversion rules for the surviving record. Deliver scoring last.

### Gate

Milestone M14. Conversion, merging, winning, losing and restoring behave as specified; assignment respects each member's maximum and does not reassign an assigned lead; and the predictive probability reproduces the specified value from the stated frequency counts.

---

## Step 15: The human resources family

### Purpose

At the end of step 15 the business manages people: employees with versions and contracts, departments and job positions, working schedules, attendances and overtime, time off with accrual plans, expenses with their reimbursement, recruitment, vehicles, and the employee services that keep a workplace running.

### Prerequisites

Steps 2 and 6.

### Entities to deliver

**Human resources core.** Employee, Public Employee Profile, Employee Version, Contract Type, Contract Template Wizard, Department, Job Position, Employee Tag, Employee Location, Work Location, Departure Reason, Departure Wizard, Set Homework Location Wizard, Resume Line, Resume Line Type, Employee Resume, Print Resume Wizard, Skill, Skill Type, Skill Level, Employee Skill, the skills requirement of a job position, Salary Structure Type, Work Entry, Work Entry Type, Work Entries Employees, Work Entry Regeneration Wizard, Bank Account Allocation Wizard, Bank Account Allocation Line, the employee skills and certification reports, and the department manager report.

**Attendances and working time.** Resource, Resource Mixin, Working Schedule, Resource Time Off, Attendance, Attendance Overtime Rule, Attendance Overtime Ruleset, Attendance Overtime Line.

**Time off.** Time Off Type, Time Off Request, Time Off Allocation, Accrual Plan, Accrual Plan Level, Mandatory Working Day, Multiple Time Off Generation Wizard, Multiple Allocation Generation Wizard, Time Off Cancellation Wizard, the summary report wizard and the analysis reports.

**Expenses.** Expense, Expense Split Wizard, Expense Posting Wizard, Expense Refusal Wizard, Duplicate Expense Approval Wizard.

**Recruitment.** Applicant, Applicant Tag, Recruitment Stage, Recruitment Source, Job Platform, Degree, Refuse Reason, Get Refuse Reason Wizard, Send mails to applicants Wizard, Talent Pool, Add applicants to talent pool Wizard, Add applicants to a job Wizard, the applicant skill level entity.

**Fleet.** Vehicle, Vehicle Model, Vehicle Brand, Vehicle Category, Vehicle State, Vehicle Tag, Vehicle Contract, Vehicle Service Log, Vehicle Service Type, Vehicle Odometer Reading, Vehicle Assignment Log, Send mails to Drivers Wizard, the cost and odometer analysis reports.

**Meals, recognition and digests.** Lunch Product, Lunch Product Category, Lunch Topping, Lunch Vendor, Lunch Location, Lunch Alert, Lunch Order, Lunch Cash Move, Lunch Cash Move Report, Challenge, Challenge Line, Goal, Goal Definition, Goal Update Wizard, Badge, Badge Award, Badge Award Wizard, Karma Tracking, Karma Rank, Digest Email, Digest Tip.

### Operations to deliver

`create_employee`, `create_employee_version`, `activate_employee_version`, `register_departure`, `compute_working_hours`, `check_in`, `check_out`, `compute_overtime`, `request_time_off`, `approve_time_off`, `refuse_time_off`, `cancel_time_off`, `allocate_time_off`, `run_accrual_plan`, `compute_remaining_leaves`, `generate_work_entries`, `regenerate_work_entries`, `submit_expense`, `approve_expense`, `post_expense`, `reimburse_expense`, `refuse_expense`, `create_applicant`, `move_applicant_stage`, `hire_applicant`, `refuse_applicant`, `add_to_talent_pool`, `assign_vehicle`, `record_odometer`, `close_vehicle_contract`, `order_lunch`, `confirm_lunch_order`, `grant_badge`, `evaluate_challenge`, `send_digest`.

### Reports to deliver

Print Badge, Employee Resume, Time Off Summary, Expenses Report, Expense Report Image.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Notify expiring contract or work permit | daily | Creates an activity for every employee version or work permit that expires within the configured horizon. |
| Update current employee version | daily | Switches the current version of every employee whose next version starts today. |
| Presence refresh | hourly | Recomputes the presence state of every employee from attendances, time off and system activity. |
| Certification reminder | daily | Creates an activity for every employee whose certification is missing or expiring. |
| Generate missing work entries | daily | Generates the work entries of every employee for the open period. |
| Automatic check-out | every four hours | Closes attendances left open beyond the configured maximum duration. |
| Absence detection | every four hours | Creates the absence records of employees who did not check in on a working day. |
| Accrual time off update | daily | Advances every accrual plan level and credits the earned allocation. |
| Cancel invalid time off | daily | Cancels approved time off that became invalid because the contract or schedule changed. |
| Submitted expense reminder | weekly | Reminds approvers of expenses waiting for approval. |
| Fleet contract cost generation | daily | Creates the recurring cost entries of vehicle contracts according to their frequency. |
| Digest electronic mail | daily | Sends each digest to its subscribers with the computed indicators. |
| Challenge evaluation | daily | Recomputes goals, closes reached goals and awards the badges. |
| Karma consolidation | monthly | Consolidates the karma tracking rows into monthly totals. |

### Interfaces to deliver

The thirteen attendance endpoints (the kiosk, the personal identification number check, the barcode check-in), the four human resources endpoints, the six recruitment endpoints (the public job page and the application form), and the nine employee-service endpoints. The screens: the employee form with its versions, the attendance kiosk, the time off calendar with its allocation summary, the expense list with its submission flow, the recruitment kanban, the vehicle form, the lunch ordering board.

### Domain folders to read

1. [human resources core](../domains/human-resources-core/) in full, including employee versions and contracts, and [work entries](../domains/work-entries/) in full.
2. [`../domains/attendances-and-working-time/`](../domains/attendances-and-working-time/) in full.
3. [time off](../domains/time-off/) in full, including accrual plans.
4. [`../domains/expenses/`](../domains/expenses/) in full.
5. [`../domains/recruitment/`](../domains/recruitment/) in full.
6. [`../domains/fleet/`](../domains/fleet/) in full.
7. [lunch ordering](../domains/lunch-ordering/) in full for meals; the recognition part of [learning, questionnaires and recognition](../domains/learning-surveys-and-gamification/) for challenges, goals, badges and ranks; and the digest sections of [human resources core](../domains/human-resources-core/).

### Notes on order inside the step

Deliver the resource and the working schedule before employees: an employee is a resource with a schedule, and every duration computation in time off, attendances, projects, timesheets and manufacturing scheduling reads the schedule. Deliver the interval arithmetic of the schedule — working hours and working days between two moments, in a named time zone, across a public holiday — before any absence, overtime, work entry or task metric is computed from it. Deliver employee versions before any behavior that depends on the contract. Deliver time off before work entries, because a work entry of the time-off type is created from an approved request. Deliver expenses after step 6, because posting an expense writes a journal entry and may create a payment.

### Gate

Milestone M15. A working schedule yields the specified hours and days across a period containing a public holiday; an absence request in days, half days and hours each computes the specified duration; an accrual produces the specified balance across multiple periods, including proration and carry-over; work entry generation produces exactly the specified entries with a validated absence overlaid and a conflict raised for an overlap; and an expense report posts the specified entry for each payment mode.

---

## Step 16: Projects and timesheets

### Purpose

At the end of step 16 work can be organized and measured: projects with stages and milestones, tasks with their own stages, recurrences, dependencies and sharing, and time logged against tasks, billed to customers and compared with planned effort.

### Prerequisites

Steps 11 and 15.

### Entities to deliver

**Projects and tasks.** Project, Project Stage, Project Template create Wizard, Project Update, Project Tag, Project Role, Project Collaborator, Project Sharing Wizard, Project Sharing Collaborator Wizard, the role-to-user mapping wizard, Project Stage Delete Wizard, Task, Task Stage, Personal Task Stage, Task Recurrence, Task Sharing Wizard, Project Task Stage Delete Wizard, Milestone, Tasks Analysis, Burndown Chart Report.

**Timesheets.** The timesheet line as an extension of the Analytic Line entity, the project and sales line mapping entity, the employee deletion wizard, Timesheet Analysis Report, Timesheet Attendance Report.

### Operations to deliver

`create_project`, `create_project_from_template`, `move_task_stage`, `assign_task`, `create_recurring_task`, `close_task`, `share_project`, `share_task`, `create_milestone`, `reach_milestone`, `log_timesheet`, `validate_timesheet`, `bill_timesheet`, `compute_remaining_hours`, `compute_project_profitability`, `send_project_rating_request`.

### Reports to deliver

The six timesheet report layouts.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Project stage rating request | daily | Sends the customer satisfaction request for tasks that entered a stage configured to ask for a rating. |

### Interfaces to deliver

The twelve project endpoints for shared project and task pages. The three timesheet endpoints. The screens: the task kanban with its stages, the gantt-style planning list, the timesheet grid by day and week, the project profitability panel.

### Domain folders to read

1. [`../domains/projects-and-tasks/`](../domains/projects-and-tasks/) in full.
2. [`../domains/timesheets/`](../domains/timesheets/) in full.
3. [`../domains/analytic-accounting/entities.md`](../domains/analytic-accounting/entities.md) again: a timesheet line is an analytic line and inherits its constraints.

### Notes on order inside the step

Deliver projects before tasks and both before timesheets. Deliver the analytic account link of a project before billing, because billable time reaches the invoice through the analytic account and the sales order line.

### Gate

Milestone M16, and with it stage gate nine. A timesheet line costs and bills at the specified rates, a validated line refuses edits with the specified message, and the project visibility rules admit and refuse exactly the specified users for each privacy setting.

---

## Step 17: Communication

### Purpose

At the end of step 17 the collaboration substrate of step 2 becomes a complete communication application: discussion channels with membership and presence, live chat with visitors and chat bots, text messages, postal letters, the incoming electronic mail gateway with aliases, mailing lists and moderation, and blacklists.

### Prerequisites

Step 2.

### Entities to deliver

Discussion Channel, Discussion Channel Member, the channel member history entity, Real Time Communication Session, the call history entity, Interactive Connectivity Server, Voice Message Metadata, Live Chat Channel, Live Chat Channel Rule, Live Chat Conversation Tags, Live Chat Expertise, Live Chat Channel Report, Chatbot Script, Chatbot Script Step, Chatbot Script Answer, Chatbot Message, Rating, Rating Mixin, Rating Parent Mixin, Email Alias, Email Alias Domain, Email Alias Mixin, Email Aliases Mixin (light), Mail Gateway Allowed Sender, Mail Group, Mailing List Message, Mailing List Member, the mailing list allow and block list entity, Reject Group Message Wizard, Email Blacklist, Remove email from blacklist wizard, Phone Blacklist, Remove phone from blacklist Wizard, Discussion Thread Blacklist Mixin, Discussion Thread Phone Mixin, Discussion Thread Carbon Copy Mixin, Outgoing Text Message, Text Message Template, Text Message Template Preview, Text Message Template Reset Wizard, Text Message Composer Wizard, Text Message Tracker, Text Message Account Verification Code Wizard, Text Message Account Registration Phone Number Wizard, Text Message Account Sender Name Wizard, Text Message Twilio Connection Wizard, Twilio Number, Postal Letter, Push Notification, Push Notification Device, Scheduled Message, Scheduled Messages, the user role mention entity, Mail Bot, Publisher Warranty Contract.

### Operations to deliver

`create_channel`, `join_channel`, `leave_channel`, `mute_channel`, `pin_message`, `react_to_message`, `translate_message`, `start_call`, `join_call`, `route_incoming_email`, `create_record_from_email`, `bounce_email`, `moderate_group_message`, `subscribe_to_mailing_list`, `unsubscribe_from_mailing_list`, `blacklist_email`, `blacklist_phone`, `send_text_message`, `queue_text_message`, `send_postal_letter`, `start_chatbot_script`, `answer_chatbot_step`, `escalate_to_operator`, `rate_conversation`, `schedule_message`, `send_push_notification`.

### Reports to deliver

Live Chat Conversation transcript.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Incoming mail fetch | every five minutes | Polls every configured incoming mail server and routes each message through the gateway. |
| Text message queue manager | daily | Sends queued text messages, records the delivery state and handles the credit failures. |
| Postal letter queue | daily | Sends queued letters and records their state. |
| Post scheduled messages | daily | Posts messages whose scheduled moment has arrived. |
| Notify scheduled messages | hourly | Sends the notifications of the scheduled messages posted since the last run. |
| Push notification delivery | daily | Delivers the queued push notifications to registered devices and removes the expired registrations. |
| Channel member unmute | daily | Clears expired mute settings. |
| Mailing list moderator notification | daily | Notifies moderators of messages waiting for moderation. |
| Publisher notification refresh | weekly | Refreshes the publisher notification state. |

### Interfaces to deliver

The one hundred and forty-eight endpoints of the domain: the message bus subscriptions, channel operations, attachment upload, live chat session handling, chat bot steps, rating pages, mailing list subscription pages and gateway health probes.

### Domain folders to read

1. [messaging and activities](../domains/messaging-and-activities/) in full, this time including discussion channels and live chat, the incoming electronic mail gateway, and text messages and postal mail.

### Notes on order inside the step

Deliver channels before live chat: a live chat conversation is a channel with a visitor and an operator. Deliver the gateway before mailing lists and before any alias-driven document creation used by recruitment or projects. Deliver blacklists before mass mailing in step 18.

### Gate

Milestone M17, and with it the remainder of stage gate four. An inbound message routed by reply reference appends to the existing thread; one routed by an alias creates a record with the specified field mapping; one that matches nothing is handled as specified; and a bounced message increments the counter on the party and suppresses further sending as specified.

---

## Step 18: Marketing, events, questionnaires and courses

### Purpose

At the end of step 18 the business can address an audience: mass mailings with templates, statistics and split testing, campaigns with automated steps, marketing cards, events with tickets, booths, tracks and badges, surveys with scoring and certification, courses with content and quizzes, and forums.

### Prerequisites

Steps 17 and 11.

### Entities to deliver

**Marketing.** Mass Mailing, Mailing List, Mailing Contact, Mailing Subscription, Mailing Trace, Mailing Trace Report, Mailing Filter, Mailing Opt-Out Reason, Mailing List Merge Wizard, Mailing Contact Import Wizard, Mailing Contact to List Wizard, Mailing Schedule Wizard, Mailing Test Wizard, Test Text Message Mailing Wizard, Campaign, Campaign Stage, Campaign Tag, Campaign Medium, Campaign Source, Campaign Tracking Mixin, Campaign Tracking Source Mixin, Link Tracker, Link Tracker Code, Link Tracker Click, Marketing Card, Marketing Card Campaign, Marketing Card Campaign Tag, Marketing Card Template.

**Events.** Event, Event Template, Event Stage, Event Tag, Event Tag Category, Event Ticket, Event Template Ticket, Event Registration, Event Registration Answer, Event Question, Event Question Answer, Event Slot, Slot Mail Scheduler, Event Communication, Event Communication Registration, the mail scheduling template on the event category, Event Track, Event Track Stage, Event Track Tag, Event Track Tag Category, Event Track Location, Event Track Visitor, Event Booth, Event Booth Category, Event Booth Template, Event Booth Registration, Event Booth Configurator Wizard, Event Configurator Wizard, Event Sponsor, Event Sponsor Type, the two attendee-editing wizards used on sales confirmation, Event Sales Report, Website Event Menu, Quiz, Quiz question and answer entities.

**Surveys, courses and forums.** Survey, Survey Question, Survey Answer Option, Survey Participation, Survey Participation Answer, Survey Invitation Wizard, Course, Course Content, Content Resource, Content Embed, Content Progress, Content Tag, Course Enrollment, Course Invitation Wizard, Course Tag, Course Tag Group, Quiz Question, Quiz Answer, Forum, Forum Post, Forum Post Vote, Forum Post Closing Reason, Forum Tag.

### Operations to deliver

`create_mailing`, `send_mailing_test`, `schedule_mailing`, `send_mailing`, `record_mailing_trace`, `run_split_test`, `track_link_click`, `advance_campaign_participant`, `generate_marketing_card`, `create_event`, `create_event_from_template`, `open_event_registrations`, `register_attendee`, `confirm_registration`, `cancel_registration`, `scan_badge`, `sell_event_ticket`, `reserve_booth`, `publish_track`, `send_event_communication`, `start_survey`, `answer_survey_question`, `score_survey`, `issue_certification`, `enroll_in_course`, `complete_content`, `post_forum_question`, `accept_forum_answer`, `vote_forum_post`.

### Reports to deliver

Full Page Ticket in its three layouts, Badge in its two layouts, Attendee List in its two layouts, Certifications.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Mass mailing queue | daily | Sends the mailings whose scheduled moment has arrived, in batches, and records one trace per recipient. |
| Split test evaluation | daily | Compares the variants of a split test, picks the winner according to the configured criterion and sends it to the remaining audience. |
| Event mail scheduler | daily | Sends the event communications that became due (before the event, after the event, on registration) and records them. |

### Interfaces to deliver

The thirty-one marketing endpoints (tracked links, unsubscribe pages, mailing preview), the forty-one event endpoints (registration pages, ticket selection, badge scanning), and the one hundred and five endpoints of surveys, courses and forums (survey taking, certification download, course player, forum pages).

### Domain folders to read

1. [`../domains/marketing-and-mass-mailing/`](../domains/marketing-and-mass-mailing/) in full.
2. [`../domains/events/`](../domains/events/) in full.
3. [`../domains/learning-surveys-and-gamification/`](../domains/learning-surveys-and-gamification/) in full.

### Notes on order inside the step

Deliver mailing lists and blacklists before mass mailing. Deliver events before event tickets sold through a sales order, which reuses the order line behavior of step 11. Deliver questionnaires before certification, and courses after questionnaires because a course quiz reuses the questionnaire scoring rules.

### Gate

Milestone M18. A mailing sends to the specified recipients, suppresses the specified ones and reports the stated counts; event seat availability and the communication schedule match the specified values; and a questionnaire scores as specified, including partial credit, conditional display and attempt limits.

---

## Step 19: Website, storefront and portal

### Purpose

At the end of step 19 the business has a public face: pages built from reusable blocks, menus, blogs, a search engine metadata layer, a storefront with a catalog, a cart, a checkout with delivery and payment, and a customer portal where a signed-in customer sees documents, pays invoices and signs quotations.

### Prerequisites

Steps 11, 6 and 17.

### Entities to deliver

**Website and content management.** Website, Website Menu, Website Page, Website Model Page, Website Technical Page, Website Route, Web Address Rewrite, Robots Exclusion Wizard, Search Engine Metadata Mixin, Website Published Mixin, Multi Website Mixin, Multi Website Published Mixin, Website Searchable Mixin, Cover Properties Mixin, Website Visitor, Website Visit Track, Website Configurator Feature, Website Content Block Filter, the page property wizards, the page and record visibility option entities, Theme View, Theme Page, Theme Menu, Theme Asset, Theme Attachment, Theme Utilities, Assets Utils, Blog, Blog Post, Blog Tag, Blog Tag Category, the rich text conversion, history and processing entities, the blocked third-party domain wizard.

**Commerce storefront.** Website Product Category, Product Attribute Category, Product Ribbon, Product Image, Product Feed, Product Wishlist, Website Checkout Step, Storefront Extra Field, Base Unit Display.

**Customer portal.** Portal Access Mixin, Portal Share Wizard, Portal Access Wizard, Portal Access Wizard User.

### Operations to deliver

`publish_record`, `unpublish_record`, `create_page`, `duplicate_page`, `apply_theme`, `track_visitor`, `search_website`, `add_to_cart`, `update_cart_line`, `apply_coupon_on_cart`, `select_delivery_method`, `compute_delivery_rate_on_cart`, `start_checkout`, `confirm_cart_as_order`, `pay_cart`, `subscribe_to_stock_notification`, `add_to_wishlist`, `compare_products`, `grant_portal_access`, `revoke_portal_access`, `share_document_by_token`, `sign_quotation_online`, `pay_invoice_online`.

### Reports to deliver

None of its own; portal pages reuse the printed documents of steps 6, 10 and 11.

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Abandoned cart reminder | hourly | Sends the recovery message for carts left unpaid beyond the configured delay. |
| Back-in-stock notification | hourly | Notifies the customers who subscribed to a product whose available quantity became positive. |
| Inactive visitor cleanup | daily | Deletes visitor records older than the retention period and their tracking rows. |
| Unused asset cleanup | weekly | Removes generated presentation assets that no page references. |

### Interfaces to deliver

The ninety-six website endpoints, the seventy-seven storefront endpoints and the nineteen portal endpoints. The cart and checkout contract: step order, validation at each step, the recomputation of prices and taxes on address change, and the transition from cart to sales order.

### Domain folders to read

1. [website and storefront](../domains/website-and-storefront/) in full: the page, menu, theme and visitor material first, then the catalog and its variants, stock availability and pickup, checkout and payment, and engagement and merchandising.
2. [customer portal](../domains/customer-portal/) in full.
3. The storefront application of [loyalty and promotions](../domains/loyalty-and-promotions/).

### Notes on order inside the step

Deliver the page and menu model before the storefront, because storefront pages are website pages. Deliver the cart as a draft sales order before checkout. Deliver the portal access token rules before any shared document link.

### Gate

Milestone M19, and with it stage gate eight. A storefront checkout reserves, authorizes a payment, confirms the order and produces the specified documents; a duplicate payment notification is processed once; and a portal user sees exactly their own documents and no others.

---

## Step 20: Spreadsheets, automation, electronic invoicing, localizations and industry blueprints

### Purpose

At the end of step 20 the platform is complete: users build live spreadsheets and dashboards over their data, administrators automate rules without programming, invoices leave and enter the system as structured electronic documents over a document exchange network, country packages configure the chart of accounts and tax rules per jurisdiction, and a vertical can be configured in one activation from an industry blueprint.

### Prerequisites

Every earlier step.

### Entities to deliver

**Spreadsheets and dashboards.** Spreadsheet Mixin, Spreadsheet Dashboard, Spreadsheet Dashboard Group, Spreadsheet Dashboard Share, Dashboard Board.

**Automation and integration services.** Automation Rule, Data Recycling Rule, Data Recycling Record, Privacy Lookup Wizard, Privacy Lookup Line, Privacy Log, Data Import Session, Data Import Column Mapping, Code Translation, Transifex Translation, In-Application Purchase Account, In-Application Purchase Service, Lead Enrichment Service, Google Service, Google Gmail Mixin, Microsoft Service, Microsoft Outlook Mixin, Fetchmail Server, Cloud Storage Migration Report, Import Module Wizard.

**Electronic invoicing and document interchange.** Electronic Document, Electronic Document Format, Electronic Document Common Base, the universal business language base and its country profiles, the cross industry invoice base and its formats, the ordering profiles for purchase and sales, the point of sale document builder, Digital Certificate, Digital Key, Electronic Interchange Proxy User, Peppol Service, Peppol Registration Wizard, Peppol Configuration Wizard, Peppol Rejection wizard, the business level response entities and the rejection clarification entity.

**Fiscal localizations.** The ninety-six country-specific entities and their chart templates, tax templates, fiscal position templates, tax report structures and country document rules.

### Operations to deliver

`create_spreadsheet`, `insert_pivot_into_spreadsheet`, `refresh_spreadsheet_data`, `share_dashboard`, `evaluate_automation_rule`, `execute_automation_action`, `run_data_recycling_rule`, `merge_duplicate_records`, `run_privacy_lookup`, `import_data_file`, `map_import_columns`, `generate_electronic_document`, `validate_electronic_document`, `send_electronic_document`, `receive_electronic_document`, `import_bill_from_electronic_document`, `register_on_exchange_network`, `update_participant_status`, `load_country_package`, `load_chart_template`, `generate_country_tax_report`, `activate_industry_blueprint`.

### Reports to deliver

The electronic invoice rendering, the country-specific printed documents of the localization packages (delivery guides, vouchers, waybills, commercial invoices, compliance letters, disbursement vouchers, hash integrity statements).

### Scheduled jobs to deliver

| Job | Interval | Effect of one run |
|---|---|---|
| Automation rule evaluation | every four hours | Evaluates every time-based automation rule and executes its actions on the matching records. |
| Data recycling | daily | Applies the recycling rules and lists the records proposed for merge or deletion. |
| Code translation reload | weekly | Refreshes the translated terms from the translation service. |
| Cloud storage migration | on demand | Moves stored binaries to the configured cloud storage. |
| Exchange network document retrieval | every four hours, one job per network | Fetches new inbound documents, creates the draft vendor bills and records the acknowledgements. |
| Exchange network status update | daily, one job per network | Refreshes the delivery state of sent documents. |
| Exchange network participant status | weekly | Refreshes the registration state of participants. |
| Exchange network keep-alive | every two weeks | Renews the webhook registration. |
| Country regulatory submissions | daily, monthly or annually depending on the country | Produces and submits the periodic declarations of the country package: daily, monthly and annual sales closings, invoice status updates, third-party invoice retrieval, signed document archiving. |

### Interfaces to deliver

The six spreadsheet endpoints, the twenty-two automation and integration endpoints, the five electronic document endpoints and the seventeen localization endpoints. The exchange network connector contract: registration, sending, receiving, acknowledging, rejecting and status polling.

### Domain folders to read

1. [`../domains/spreadsheets-and-dashboards/`](../domains/spreadsheets-and-dashboards/) in full.
2. [`../domains/automation-and-integration/`](../domains/automation-and-integration/) in full.
3. [`../domains/electronic-invoicing-and-document-exchange/`](../domains/electronic-invoicing-and-document-exchange/) in full.
4. [fiscal localizations](../domains/fiscal-localizations/) in full, including every per-country file that folder publishes under `countries/`, one file per jurisdiction, because each country package differs from the base only in its data and in a small number of rule overrides that the per-country file states.

### Notes on order inside the step

Deliver the electronic document abstraction before any concrete format, and the formats before the country packages that use them. Deliver the chart template loader before any country package. Deliver the country packages last among the accounting work, because each one is a data set plus a small number of rule overrides on a base that must already be correct; they are independent of one another and can be built one country per team, in any order. Deliver industry blueprints last of all.

**Industry configuration blueprints.** A blueprint is a configuration bundle for one vertical. Activating it installs exactly the capability packages it declares, seeds exactly the master data records it declares under their stable external identifiers, and leaves the system in a state where the vertical's first document can be created without further configuration. A blueprint contains no behavior of its own: every capability it names must already exist, which is why blueprints are the last thing built. Activating a blueprint twice does not duplicate its master data.

### Gate

Milestone M20, and with it stage gate ten and the country and exchange part of stage gate six. A spreadsheet recomputes over live records; an automation rule fires on exactly the specified condition and does not retrigger itself; an import of two hundred rows with five invalid ones reports exactly those five and commits the rest, or none, according to the specified mode; a generated electronic document validates against its format; a received one creates the specified draft bill; each country package loads its chart, taxes, fiscal positions and report structure; and every gate of milestones M1 to M19 still passes on the complete system.

---

## 2. What to build in parallel and what never to parallelize

| Never split across teams | Reason |
|---|---|
| The tax engine and the document totals | The document total is defined by the tax engine's rounding sequence; two teams produce two roundings. |
| The move state machine and the transfer state machine | The transfer state is derived from its moves; splitting them produces states that disagree. |
| Valuation and the costing methods | The three costing methods share the layer and the replay algorithm. |
| The session closing entry and the point of sale order | The closing entry is the aggregation of order lines and payments; the aggregation rules live in both. |
| Reconciliation and exchange differences | An exchange difference is a write-off created inside reconciliation. |

| Safe to parallelize after its prerequisite | Prerequisite |
|---|---|
| Recruitment, fleet, meal ordering, recognition and digests | Human resources core |
| Marketing, events, questionnaires and courses | Communication |
| Website, storefront and portal | Sales, payments and communication |
| Localizations, one country per team | Electronic invoicing and the chart template loader |
| Industry blueprints, one vertical per team | Every capability package the blueprint activates |

---

## 3. Entry conditions that apply to every step

1. **No step starts without its fixtures.** The reference company of the [equivalence test plan](equivalence-test-plan.md), section 3, must be loadable before the step's scenarios can run. Each step extends the fixture; it never redefines it.
2. **Every rule cited by the step has an identifier.** Before coding an operation, list the rule identifiers it must enforce from the domain's `business-rules.md`, and record them in the coverage matrix described in [traceability rules](traceability-rules.md).
3. **Every numeric rule has a worked example.** If the domain document gives one, reproduce its numbers exactly. If it does not, the rule is under-specified: raise it rather than inventing a rounding.
4. **Error messages are part of the contract.** A step is not complete while an operation refuses an action with a different message than the one in the domain document.
5. **The step's scheduled jobs are proven with a controlled clock.** Every job is run twice at a fixed moment: the second run must not repeat the effect of the first unless the job is specified as repeating.

---

## 4. Seed data to load at each step

A step is not complete until its shipped records exist, because later steps, the acceptance criteria and the industry blueprints all resolve them by their stable external identifier. The counts below are the records the reference behavior ships; a replacement must ship the same records with the same identifiers and the same values.

| Step | Entity | Records | What the data is |
|---|---:|---:|---|
| 1 | Report Layout | 8 | The printed-document layouts a company can choose. |
| 1 | Module | 21 | The package descriptors loaded before any installation. |
| 1 | Access Group | 143 across all steps, of which 12 belong to the foundation | Administration, access rights management, settings, multi-company, multi-currency and the internal-user and portal-user base groups. |
| 1 | Access Privilege | 29 | The named privilege levels grouped by category and used to build the group matrix. |
| 1 | Group Category | 28 | The application categories the groups are presented under. |
| 1 | System Parameter | 3 of the foundation's own | The database identifier, the web base address and the report layout default. |
| 2 | Country | 261 | Every country with its code, currency, address format, state requirement and tax identification label. |
| 2 | Country Subdivision | 39 | The shipped states and provinces. |
| 2 | City | 2 850 | The shipped city list used by address completion. |
| 2 | Country Group | 19 | Economic and tax zones used by fiscal positions. |
| 2 | Currency | 186 | Every currency with its symbol, decimal places, rounding step and display position. |
| 2 | Bank | 115 | The shipped bank directory. |
| 2 | Language | 6 | The languages activated by default. |
| 2 | Contact | 23 | The shipped demonstration and system contacts, including the default company contact. |
| 2 | Company | 3 | The default company and the two companies used by multi-company behavior. |
| 2 | Contact Tag | 21, Industry 23, Partner Grade 3, Partner Activation 3 | Segmentation data for contacts. |
| 2 | Calendar Reminder | 7 | The reminder offsets offered on an event. |
| 2 | Activity Type | 16 across all steps, of which 6 belong to the collaboration substrate | Electronic mail, call, meeting, to-do, upload document and reminder. |
| 2 | Message Subtype | 103 across all steps | The subscription categories of the discussion thread, per entity. |
| 2 | Email Template | 70 across all steps | The rendered messages of every document type. |
| 3 | Unit of Measure | 121 | The shipped units with their category, ratio and rounding. |
| 3 | Product Attribute Value | 10, Barcode Nomenclature 2, Barcode Rule 39 | The demonstration attributes and the two barcode nomenclatures with their rules. |
| 3 | Decimal Precision | 7 | Payment Terms 6 places, Percentage Analytic 2, Product Price 2, Discount 2, Stock Weight 2, Volume 2, Product Unit 2. |
| 4 | Accounting Assert Test | 6 | The shipped consistency tests of the books. |
| 4 | Sequence | 16 across all steps, of which 1 belongs to the ledger | The numbering series that are not derived from a journal. |
| 5 | Account Tag | 519 | The tax grid tags referenced by distribution lines and tax reports. |
| 5 | Financial Report | 165 | The shipped report structures, including the generic statements and the country tax returns. |
| 5 | Analytic Plan | 1 | The default plan every analytic account belongs to. |
| 6 | Payment Terms | 10 | Immediate, fifteen days, thirty days, forty-five days, sixty days, end of following month, two installments, thirty days end of month, ninety days end of month on the fifteenth, and the early-discount term. |
| 6 | International Commercial Term | 15 | The delivery terms selectable on a document. |
| 6 | Cash Rounding | 2 | The shipped nearest-five-cent and nearest-unit rules. |
| 6 | Payment Provider | 49, Payment Method 237 | The provider descriptors and the payment brands they accept. |
| 7 | Warehouse | 4, Location 3, Shipping Method 10, Shipping Price Rule 8 | The default warehouse with its locations and the shipped delivery methods with their price grids. |
| 7 | Removal Strategy | 5 | First in first out, last in first out, first expired first out, closest location, least packages. |
| 9 | Route | 6 | Buy, manufacture, deliver in one step, deliver in two steps, deliver in three steps, receive in two steps. |
| 11 | Loyalty Program | 2, Loyalty Rule 1, Loyalty Reward 1 | The shipped promotion and loyalty examples. |
| 12 | Work Center Productivity Loss | 7, Loss Type 4 | The blocking reasons and their categories. |
| 12 | Maintenance Stage | 4, Maintenance Team 1 | The default maintenance pipeline. |
| 13 | Point of Sale Cash Denomination | 14, Point of Sale Category 13, Point of Sale Preset 6, Point of Sale Note 4, Restaurant Floor 2, Restaurant Table 24 | The counter configuration data of the shipped scenarios. |
| 14 | Sales Team | 9, Pipeline Stage 4, Lost Reason 3, Sales Tag 3, Recurring Revenue Plan 4, Lead Scoring Frequency Field 7 | The pipeline configuration. |
| 15 | Work Entry Type | 118, Skill 48, Skill Level 36, Skill Type 6, Contract Type 12, Salary Structure Type 4, Departure Reason 3, Work Location 3, Department 6, Job Position 5, Resume Line Type 4 | The employment master data. |
| 15 | Time Off Type | 132 | The shipped leave types across the supported jurisdictions. |
| 15 | Working Schedule | 3, Attendance Overtime Rule 6, Attendance Overtime Ruleset 2 | The default forty-hour week and the overtime rules. |
| 15 | Vehicle Brand | 67, Vehicle State 4, Vehicle Service Type 3 | The fleet master data. |
| 15 | Goal Definition | 47, Challenge 37, Challenge Line 41, Badge 38, Karma Rank 5, Digest Email 9 | The engagement master data. |
| 15 | Recruitment Stage | 6, Refuse Reason 6, Applicant Tag 5, Degree 4, Job Platform 3 | The recruitment pipeline. |
| 16 | Project Stage | 4, Task Stage 1 | The default project and task pipelines. |
| 17 | Chatbot Script | 2 with 14 steps and 3 answers, Live Chat Channel 1, Discussion Channel 3, Canned Response 1, Email Alias 1, Text Message Template 7 | The communication master data. |
| 18 | Marketing Card Template | 14, Campaign Source 12, Campaign Medium 11, Mailing Opt-Out Reason 5 | The campaign master data. |
| 18 | Event Booth Category | 8, Event Track Stage 6, Event Stage 4, Event Question 3, Event Sponsor Type 3 | The event master data. |
| 18 | Forum Post Closing Reason | 13, Forum 1, Course Tag 3, Course Tag Group 2 | The community master data. |
| 19 | Website | 4, Website Menu 9, Website Page 5, Website Configurator Feature 13, Website Content Block Filter 11, Blog 1, Website Checkout Step 5, Product Ribbon 4 | The public site and storefront master data. |
| 20 | Spreadsheet Dashboard | 14 in 7 groups | The shipped dashboards. |
| 20 | In-Application Purchase Service | 5 | The metered service descriptors. |
| 20 | Electronic Document Format | 3, Peppol rejection clarifications 21 | The exchange master data. |
| 20 | Localization identification types | 102, Latam Document Type 60, Electronic Waybill Document Type 17, Responsibility Type 16, earnings scales 18, tax expense categories 13 | The country master data. |

---

## 5. Access groups per step

The group matrix is built incrementally: a step adds its own groups and the privileges that compose them, and never edits a group of an earlier step except to add an access rule. The one hundred and forty-three shipped groups are distributed as follows.

| Step | Groups added | Examples |
|---|---:|---|
| 1 | 12 | Administration Settings, Access Rights Manager, Multi Company, Multi Currency, Internal User, Portal User, Public User. |
| 3 | 5 | Product Variants, Units of Measure, Product Packagings, Product Price Comparison, Product Pricelists. |
| 4 | 10 | Show Accounting Features, Billing, Billing Administrator, Accountant, Bank Statement Import, Audit Trail, Cash Rounding, Analytic Accounting. |
| 6 | included in the ledger groups | Payment methods and reconciliation are privileges inside the billing groups. |
| 7 | 13 | Inventory User, Inventory Administrator, Storage Locations, Multi-Step Routes, Lots and Serial Numbers, Packages, Consignment, Delivery Methods, Barcode. |
| 10 | 6 | Purchase User, Purchase Administrator, Purchase Agreements, Purchase Alternatives, Purchase Warnings, Bill Control. |
| 11 | included in the sales groups | Sales User own documents, Sales User all documents, Sales Administrator, Discounts on lines, Quotation Templates, Sales Warnings. |
| 12 | 7 | Manufacturing User, Manufacturing Administrator, Work Orders, By-Products, Work Order Dependencies, Shop Floor, Maintenance User, Maintenance Administrator. |
| 13 | 6 | Point of Sale User, Point of Sale Administrator, Restaurant, Self-Ordering, Preparation Display. |
| 14 | 5 | Sales Team Member, Sales Team Leader, Lead Stage, Lead Mining, Predictive Scoring. |
| 15 | 5 in attendances, and the employee, officer and administrator groups of each people domain | Attendance User, Attendance Officer, Attendance Administrator, Employee Officer, Employee Administrator, Time Off Approver, Time Off Administrator, Expense Approver, Expense Administrator, Recruitment Interviewer, Recruitment Officer, Recruitment Administrator, Fleet User, Fleet Administrator. |
| 16 | 11 | Project User, Project Administrator, Project Stages, Task Dependencies, Milestones, Project Sharing, Timesheet User, Timesheet Approver, Timesheet Administrator, Billing Rates. |
| 17 | 8 | Discussion User, Live Chat Operator, Live Chat Manager, Mail Group Moderator, Blacklist Manager, Text Message Sender. |
| 18 | 5 events plus the marketing, survey and course groups | Event User, Event Administrator, Event Booth Manager, Mailing User, Mailing Administrator, Campaign Manager, Survey User, Survey Administrator, Course Officer, Course Administrator, Forum Moderator. |
| 19 | 6 | Website Editor, Website Designer, Website Administrator, Storefront Manager, Portal Manager. |
| 20 | remaining | Spreadsheet Editor, Dashboard Manager, Automation Manager, Data Recycling Manager, Privacy Officer, Electronic Exchange Manager, Localization Administrator. |

---

## 6. Numeric precision to configure before step 4

These seven precisions are configuration records, not constants in the source of a program: an administrator may change them, and every formula that reads them must observe the change. They are loaded in step 3 and are entry conditions for every monetary and quantity computation afterwards.

| Precision | Decimal places | Applies to |
|---|---:|---|
| Product Unit | 2 | Every quantity field on a document line, a stock move, a manufacturing component or a point of sale line, unless the unit of measure imposes a coarser rounding step. |
| Product Price | 2 | Unit prices on documents, pricelist computed prices, the cost of a product. |
| Discount | 2 | The percentage discount on a document line. |
| Payment Terms | 6 | The percentage of a payment term installment, so that three equal installments distribute exactly. |
| Percentage Analytic | 2 | The percentage of an analytic distribution. |
| Stock Weight | 2 | The weight of a product, a package and a shipment. |
| Volume | 2 | The volume of a product and a package. |

Currency amounts do not use these precisions: they use the decimal places and the rounding step of their currency, which is data on the Currency record.

---

## 7. Work that runs alongside every step

- **The equivalence suite.** Every worked example in a `calculations.md` file, every record of the mathematics catalogs under [`../../schemas/mathematics/`](../../schemas/mathematics/) and every scenario in an `acceptance-criteria.md` file is a test. Encode them as the behavior is implemented, never afterwards: a suite written after the fact tests what was built instead of what was specified. The plan is in the [equivalence test plan](equivalence-test-plan.md).
- **The invariant check.** From step 4 onward, run the invariants of the equivalence test plan after every test: that every posted entry balances, that the valuation account equals the sum of the layers, that reconciled amounts net to zero, that quantities are conserved. A rebuild that drifts here drifts silently, and the drift is attributed to the wrong change weeks later.
- **The conformance level.** Decide early which level of [conformance profiles](conformance-profiles.md) the program is claiming, per domain, because the transport, identifier and storage decisions of steps 1 and 6 follow from it, and because the test layers that a gate requires depend on it.
- **Coverage tracking.** Record which specified artifacts are implemented and which are verified, using the structure of [coverage and evidence](coverage-and-evidence.md), and keep the record current. An overstated claim costs more to correct than a modest one costs to raise.
- **The rule identifier list.** Every operation a step delivers names, before it is built, the numbered rules of its domain's `business-rules.md` that it must enforce. That list is the input to the coverage matrix of [traceability rules](traceability-rules.md).

---

## 8. What the plan deliberately omits

It does not schedule the work in time, because duration depends on team size, on the language chosen and on how much of the platform an existing foundation provides. It does not prescribe a code structure for the rebuild, because the folder structure of this specification reflects the business, not a code layout: a rebuild is free to organize itself differently while making the same decisions. It does not order the country packages among themselves; step 20 states why. It says nothing about hosting, process supervision, scaling or server sizing, which are outside the scope of this repository.

