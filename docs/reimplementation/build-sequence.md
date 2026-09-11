# Build sequence

This document orders the rebuild. The order is forced by dependency: each step can be built and proved with only the steps before it in place. Nothing here prescribes a language, a framework, a database product or a deployment arrangement; it prescribes what must exist, in what order, and what "done" means for each part.

Each step names its deliverable, the specification documents that define it, the decisions the team must make before starting, and the gate that closes it. The gates themselves, with their acceptance scenarios, are in [milestones](milestones.md).

## How to read this plan

The plan has ten steps. Steps one to four build a platform that contains no business domain at all; they are the largest source of risk, because every later step assumes their semantics exactly. Steps five to ten add business capability, mostly in parallel once the master data of step five exists.

A common and costly mistake is to start at step six or seven, because accounting and inventory are what the business asked for, and to reimplement the platform incidentally underneath them. The platform's behaviours — dependency-driven recomputation, the record cache and its invalidation, record rules, company scoping, extension resolution — are pervasive. Retrofitting them after domain code exists means rewriting the domain code.

A second mistake is to treat the platform as a framework to be bought rather than built. Most of the platform's requirements can be met by an existing framework in the chosen language. What cannot be bought is the exact semantics: the ordering of a flush, the moment a computed field is recalculated, the way two record rules combine, the precision at which a monetary amount is rounded. Those must be implemented deliberately against this specification regardless of what framework underlies them.

## Step one: platform foundation

**Deliverable.** A running application that can define entities, persist them, query them, extend them and load data into them, with no business domain present.

**Specified by.** [Architecture](../overview/architecture.md), [entity and field system](../overview/entity-and-field-system.md), [inheritance and extension](../overview/inheritance-and-extension.md), [package system](../overview/package-system.md), [identity and values](../data/persistence-identity-and-values.md), [physical data catalogue](../data/physical-data-catalog.md), [data loading and exchange](../data/data-loading-and-exchange.md), [transactions and concurrency](../runtime/transactions-and-concurrency.md), [caching](../runtime/caching.md), [translation](../runtime/translation.md).

**Contains.**

1. The entity registry: entity definitions assembled from installed packages, with the three entity kinds, classical extension in place, prototype copying and delegation embedding, and a deterministic resolution order when several packages extend the same entity.
2. The field system: every field type with its storage mapping and value semantics; declarable attributes; computed fields with declared dependencies; related fields; company-dependent values; translatable values; defaults and their resolution order; the archive flag; audit fields.
3. The recomputation engine: when a stored field is written, every computed field that declares a dependency on it, directly or through a relation, is marked for recomputation and recalculated before it is next read or written to storage. This is the single hardest piece of the platform to get right and it must be built first, because every later correctness property rests on it.
4. Record sets: an ordered collection of records of one entity that is both the value passed around and the receiver of every operation, supporting creation, reading, updating, deletion, searching, grouped reading with aggregation, name searching, copying, set operations, mapping, filtering and sorting.
5. The filter grammar: leaf conditions of field path, operator and value, combined with prefix logical operators, evaluated against related fields and hierarchies, with every operator's exact semantics.
6. The unit of work: a pending-write buffer with a defined flush order, a record cache with defined invalidation, a database transaction per request with the specified isolation level, and automatic retry on serialisation conflict.
7. Sequences and numbering, including per-period reset and the gap rules.
8. External identifiers, the data loading grammar, reload semantics and the no-update marking.
9. Translatable values with their fallback order, and the formatting of dates, numbers and monetary amounts per language.

**Decisions to make first.**

- Whether computed fields are recalculated eagerly at write time or lazily at read time. The specification's observable behaviour is lazy-with-a-flush-barrier: a computed value is correct whenever it is read and whenever it is written to storage. Either strategy can satisfy that, but the choice pervades the code and cannot be changed cheaply.
- How the pending-write buffer orders its flush. Ordering affects which constraint fails first, and therefore which error message a user sees.
- Whether company-dependent values are stored as a per-company map on the record or as separate rows. Both are visible only through the same read and write contract, but the choice affects every query that filters on such a field.

**Gate.** The platform can define an entity with computed, related, company-dependent and translatable fields; create, read, update, delete and search its records; extend it from a second package; load and reload data by external identifier; and survive a concurrent write conflict by retrying. Milestone one.

## Step two: identity and access

**Deliverable.** Authenticated users with groups, enforced access rights, enforced record rules and enforced company scope.

**Specified by.** [Security model](../overview/security-model.md), [identity and access](../domains/identity-and-access/), [request lifecycle](../runtime/request-lifecycle.md).

**Contains.** Users and their kinds; groups, privilege families and implied-group closure; per-entity access rights with the checking algorithm and its refusal messages; record rules with the combination rules for global and group rules; field-level restrictions; the unrestricted actor and the elevate-privileges contract; company scope and the consistency checks; sessions and every authentication method; the settings mechanism that turns a configuration choice into installed packages, group membership and default values.

**Why here.** Access control cannot be added later without auditing every query already written. A record read that bypasses record rules is a data leak, and the only way to be sure none exists is for the enforcement to predate the queries.

**Gate.** A user in one group sees exactly the records the rules permit; a user in two groups sees the union permitted by their group rules intersected with every global rule; a refusal produces the specified message; switching the active company changes visibility as specified. Milestone two.

## Step three: presentation contracts and transport

**Deliverable.** A client can obtain a view definition, open an action, navigate menus, and call the generic entity operations over the transport.

**Specified by.** [Views and actions](../overview/views-and-actions.md), [remote transport contracts](../interfaces/remote-transport-contracts.md), [endpoint catalogue](../interfaces/endpoint-catalog.md), [desktop workflows](../interfaces/desktop-workflows.md), [request lifecycle](../runtime/request-lifecycle.md), [attachments and file store](../runtime/attachments-and-file-store.md), [report rendering](../runtime/report-rendering.md).

**Contains.** Every view kind and its grammar; view inheritance; window, server, client, address and report actions; menus; the generic operations with their exact arguments and return shapes; the error envelope; attachments with content addressing and access control; report rendering to a printable document.

**Decision to make first.** Whether to serve the original transport contract exactly, or to serve a new contract and provide an adapter. Serving it exactly lets existing integrations and existing clients keep working, at the cost of carrying its shape. The specification gives enough detail for either. Decide before writing the first endpoint.

**Gate.** A client can list menus, open an action, receive a form and a list view, read and save a record through the generic operations, upload and download an attachment, and print a document. Milestone three.

## Step four: messaging, activities and scheduled work

**Deliverable.** Any entity can carry a discussion thread, followers, tracked field changes and activities; the system sends and receives mail; scheduled jobs run.

**Specified by.** [Messaging model](../overview/messaging-model.md), [messaging and activities](../domains/messaging-and-activities/), [mail gateway](../runtime/mail-gateway.md), [scheduled jobs](../runtime/scheduled-jobs.md), [notification bus](../runtime/notification-bus.md), [background work](../runtime/background-workers.md).

**Why here.** Almost every business document in later steps carries a thread, notifies followers on a state change, tracks field changes for audit, and schedules activities. Building those documents first and adding messaging later means revisiting all of them.

**Gate.** A message posted on a document notifies the right followers by the right channel; a tracked field change produces a tracking entry; an inbound message creates or updates a record through an alias; an activity falls due and chains its successor; a scheduled job acquires its lock, runs and records its outcome. Milestone four.

## Step five: master data

**Deliverable.** The records every later domain refers to.

**Specified by.** [Contacts and organisations](../domains/contacts-and-organizations/), [multi-currency](../domains/multi-currency/), [units of measure and packaging](../domains/units-of-measure-and-packaging/), [products and catalogue](../domains/products-and-catalog/), [pricing and price lists](../domains/pricing-and-pricelists/), [reference data](../data/reference-data.md).

**Contains.** Parties and companies with the address and commercial-party synchronisation; countries, states, currencies and languages with their shipped data; units of measure with the conversion arithmetic and packagings; products, variants, attributes and categories; price lists and the price computation; vendor prices.

**Why here.** Units and currencies carry the rounding discipline that every later amount depends on. Getting the conversion and rounding right here, with the worked examples in [units of measure and packaging](../domains/units-of-measure-and-packaging/calculations.md) and [multi-currency](../domains/multi-currency/calculations.md) as tests, prevents a class of penny errors that is very hard to find later.

**Gate.** Every shipped reference record loads; a quantity converts between units in both directions with the specified rounding; an amount converts between currencies at a dated rate; a price list produces the specified price for a given product, quantity, date and currency. Milestone five.

## Step six: money

**Deliverable.** A complete accounting system.

**Specified by.** [General ledger](../domains/general-ledger/), [taxes](../domains/taxes/), [analytic accounting](../domains/analytic-accounting/), [accounts receivable](../domains/accounts-receivable/), [accounts payable](../domains/accounts-payable/), [payments and bank reconciliation](../domains/payments-and-bank-reconciliation/), [financial reporting](../domains/financial-reporting/), then [fiscal localisations](../domains/fiscal-localizations/) and [electronic invoicing and document exchange](../domains/electronic-invoicing-and-document-exchange/).

**Order within the step.** The chart of accounts and journals; the entry and item structure with the balance invariant; posting, numbering and the lock rules; the tax engine; the dynamic line synchronisation that keeps an invoice's product lines, tax lines and term lines consistent; reconciliation; payments; then reporting. Localisations last, because each one is a data set plus a small number of rule overrides on a base that must already be correct.

**Why the tax engine before invoices.** An invoice's totals are the tax engine's output. Building invoices against an approximate tax engine produces an invoice model shaped around the approximation.

**Gate.** A posted entry balances; a tax computes identically under per-line and global rounding for the documented cases; an invoice's terms distribute to the cent with the balance on the last instalment; a payment reconciles and leaves the specified residual; a foreign-currency payment produces the specified exchange difference; the statements balance. Milestone six.

## Step seven: goods

**Deliverable.** A complete inventory and manufacturing system with valuation.

**Specified by.** [Inventory operations](../domains/inventory-operations/), [inventory valuation and costing](../domains/inventory-valuation-and-costing/), [replenishment and procurement](../domains/replenishment-and-procurement/), [purchasing](../domains/purchasing/), [delivery and shipping](../domains/delivery-and-shipping/), [manufacturing](../domains/manufacturing/), [repair and maintenance](../domains/repair-and-maintenance/).

**Order within the step.** Locations and quantities; moves and their state machine; reservation; transfers and validation with backorders; valuation layers; then the rules and routes that generate moves; then purchasing; then manufacturing.

**Why quantities before rules.** The rule engine's purpose is to create moves. Building it before moves behave correctly means debugging two unfinished systems at once.

**Gate.** A quantity is conserved across every movement; a reservation follows the specified removal ordering; a partial validation produces the specified backorder; each costing method produces the specified value and unit cost for the worked examples; the valuation account balance equals the sum of the layers at all times. Milestone seven.

## Step eight: commerce

**Deliverable.** Selling, in every channel.

**Specified by.** [Sales](../domains/sales/), [loyalty and promotions](../domains/loyalty-and-promotions/), [point of sale](../domains/point-of-sale/), [customer relationship management](../domains/customer-relationship-management/), [payment providers](../domains/payment-providers/), [website and storefront](../domains/website-and-storefront/).

**Note on the counter channel.** The counter application computes prices, taxes, discounts and rounding on the operator's device, often without a network, and reconciles later. Its arithmetic must match the server's exactly, or a session will not close. Treat the counter's calculation rules and the tax engine as one requirement with two implementations that are tested against each other.

**Gate.** An order confirms, delivers, invoices and settles; an advance invoice deducts correctly from the final one; a counter session closes with the specified entry; a storefront checkout authorises a payment and confirms the order; a promotion and a gift card apply to one order with the specified lines. Milestone eight.

## Step nine: people and services

**Deliverable.** Employment, time, absence, expense, project and time-billing capability.

**Specified by.** [Human resources core](../domains/human-resources-core/), [attendances and working time](../domains/attendances-and-working-time/), [time off](../domains/time-off/), [work entries](../domains/work-entries/), [recruitment](../domains/recruitment/), [expenses](../domains/expenses/), [fleet](../domains/fleet/), [meal ordering](../domains/lunch-ordering/), [projects and tasks](../domains/projects-and-tasks/), [timesheets](../domains/timesheets/).

**Order within the step.** Working schedules and the interval arithmetic first, because absence duration, attendance overtime, work entry generation and task metrics all derive from it.

**Gate.** A working schedule yields the specified hours and days across a period containing a public holiday; an absence request of each unit computes the specified duration; an accrual produces the specified balance over multiple periods; an expense report posts the specified entry; a timesheet line bills at the specified rate. Milestone nine.

## Step ten: remaining capabilities

**Deliverable.** Calendar, events, marketing, learning and questionnaires, spreadsheets and dashboards, automation and integration.

**Specified by.** [Calendar and scheduling](../domains/calendar-and-scheduling/), [events](../domains/events/), [marketing and mass mailing](../domains/marketing-and-mass-mailing/), [learning, questionnaires and recognition](../domains/learning-surveys-and-gamification/), [spreadsheets and dashboards](../domains/spreadsheets-and-dashboards/), [automation and integration](../domains/automation-and-integration/).

**Gate.** Milestone ten.

## Work that runs alongside every step

- **The equivalence suite.** Every worked example in a `calculations.md` file and every scenario in an `acceptance-criteria.md` file is a test. Encode them as they are implemented, not at the end. The plan is in [equivalence test plan](equivalence-test-plan.md).
- **The ledger invariant check.** From step six onward, run a check after every test that the ledger balances, that the valuation account equals the sum of layers, and that reconciled amounts net to zero. A rebuild that drifts here drifts silently.
- **Conformance level.** Decide early which level in [conformance profiles](conformance-profiles.md) is being targeted, because the transport and identifier decisions in steps one and three follow from it.
- **Coverage tracking.** Record which specified artefacts are implemented and which are verified, using the structure in [coverage and evidence](coverage-and-evidence.md).

## What the plan deliberately omits

It does not schedule the work in time, because duration depends on team size, language and how much of the platform an existing framework provides. It does not prescribe a module structure for the rebuild, because the specification's folder structure reflects the business, not a code layout. It does not order the country localisations among themselves: they are independent of one another and depend only on step six.
