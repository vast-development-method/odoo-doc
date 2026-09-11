# Documentation index

Every document in the specification, grouped by what it is for. Read the [repository charter](../README.md) first for the conventions, the architecture and the capability map, then the [documentation rules](references/documentation-rules.md) so you know what the text will and will not contain.

## Understand the platform

The behaviour every business domain relies on, described once.

| Document | Content |
|---|---|
| [Architecture](overview/architecture.md) | Layers, tenancy, the entity registry, the environment carried by every operation, record sets, the unit of work, the path from a call to a committed transaction |
| [Package system](overview/package-system.md) | Manifests, dependency resolution, installation order, data loading, automatic installation, lifecycles and hooks, the category tree |
| [Entity and field system](overview/entity-and-field-system.md) | Entity kinds, every field type, computed and related fields, recomputation, defaults, constraints, ordering, display names, the archive flag, the filter grammar |
| [Inheritance and extension](overview/inheritance-and-extension.md) | Extending in place, copying, embedding, extension of views, data and security, and the resolution order |
| [Security model](overview/security-model.md) | Users, groups, privileges, access rights, record rules, field restrictions, privilege elevation, company scope, external access |
| [Views and actions](overview/views-and-actions.md) | Every view kind and its grammar, view inheritance, every action kind, menus, the search grammar |
| [Messaging model](overview/messaging-model.md) | Threads, followers, subtypes, notifications, tracking and activities as a capability any entity can adopt |
| [Design principles](overview/design-principles.md) | The recurring choices that give the system its behaviour, and the trade-off each implies |

## Understand the data

| Document | Content |
|---|---|
| [Domain model](data/domain-model.md) | The whole data model as one picture: the master data backbone, the transactional backbone, the aggregates and the cross-domain relations |
| [Identity and values](data/persistence-identity-and-values.md) | Identifiers, value semantics for every type, precision and rounding, dates and zones, audit fields, sequences, uniqueness |
| [Physical data catalogue](data/physical-data-catalog.md) | The tables, columns, indexes, foreign keys and constraints a full installation creates, and how each field type maps to storage |
| [Data loading and exchange](data/data-loading-and-exchange.md) | The record declaration grammar, reload semantics, import matching and batching, export |
| [Reference data](data/reference-data.md) | The records a fresh installation must contain, enumerated |

## Rebuild a business domain

Each folder holds the same eleven documents. The list of domains, with their entity and field counts, is in [the domain index](domains/README.md). The event traces that cross several domains at once are in [cross-domain transactions](domains/cross-domain-transactions.md).

## Rebuild the interfaces

| Document | Content |
|---|---|
| [Desktop workflows](interfaces/desktop-workflows.md) | How a working client composes views, actions and menus into usable screens |
| [Endpoint catalogue](interfaces/endpoint-catalog.md) | Every route with its authentication, inputs, outputs and errors |
| [External integrations](interfaces/external-integrations.md) | Every outside service as a contract: what is sent, what is received, how authenticity is established, how failure is handled |
| [Remote transport contracts](interfaces/remote-transport-contracts.md) | The envelope, the sessions, and every generic entity operation with its exact arguments and return shape |
| [Printable documents and exports](interfaces/report-and-export-documents.md) | Every document the system produces, section by section, and the export formats |
| [Service layer](interfaces/service-layer.md) | The named business operations on entities, with inputs, preconditions, effects and failures |

## Understand the runtime

| Document | Content |
|---|---|
| [Request lifecycle](runtime/request-lifecycle.md) | Routing, sessions, authentication levels, context resolution, error mapping |
| [Transactions and concurrency](runtime/transactions-and-concurrency.md) | The unit of work, flush ordering, isolation, retry on conflict, locking, recomputation protection |
| [Caching](runtime/caching.md) | The record cache, prefetching, invalidation, and what must never be cached |
| [Scheduled jobs](runtime/scheduled-jobs.md) | Job definitions, intervals, locking, failure handling, and the catalogue of shipped jobs |
| [Notification bus](runtime/notification-bus.md) | Channels, delivery, presence, fan-out across workers |
| [Mail gateway](runtime/mail-gateway.md) | The outgoing queue, server selection, bounces, inbound routing, loop prevention |
| [Attachments and file store](runtime/attachments-and-file-store.md) | Content addressing, deduplication, access control, collection of unreferenced content, image handling |
| [Report rendering](runtime/report-rendering.md) | From a report definition to a finished document: templates, layouts, page furniture, codes |
| [Translation](runtime/translation.md) | Per-language values, fallback order, and the formatting of dates, numbers and amounts |
| [Background work](runtime/background-workers.md) | Queue semantics, idempotency, retry, progress and resumption |

## Look something up

| Document | Content |
|---|---|
| [Entity index](references/entity-index.md) | Every entity with its names, kind, package and counts |
| [Entity reference pages](references/entities/) | One exhaustive page per entity |
| [Entity name dictionary](references/entity-name-dictionary.md) | The canonical full name of every entity |
| [Capability packages](references/capability-packages.md) | Every package with its category, dependencies and contents |
| [Routes](references/routes.md) | Every route |
| [Views](references/views.md) | Every view, by entity |
| [Actions and menus](references/actions-and-menus.md) | Every window action, server action and menu |
| [Printable reports](references/reports.md) | Every report definition |
| [Scheduled jobs](references/scheduled-jobs.md) | Every job |
| [Sequences](references/sequences.md) | Every numbering sequence |
| [Groups and access](references/groups-and-access.md) | Every group, access right and record rule |
| [Validation messages](references/validation-messages.md) | Every message with the entity and operation that raises it |
| [Reference data](references/reference-data.md) | Every shipped data set |
| [Documentation rules](references/documentation-rules.md) | The rules every file in this repository obeys, and why |

## Plan and verify a rebuild

| Document | Content |
|---|---|
| [Build sequence](reimplementation/build-sequence.md) | The dependency-ordered plan, what each step contains, the decisions to make first, and what closes it |
| [Milestones](reimplementation/milestones.md) | The acceptance gate for every step, item by item |
| [Conformance profiles](reimplementation/conformance-profiles.md) | Four levels of claim, what each obliges and how each is proved |
| [Equivalence test plan](reimplementation/equivalence-test-plan.md) | Eight test layers, the data sets, the running order and what to report |
| [Traceability rules](reimplementation/traceability-rules.md) | The identifiers, the maps, the ownership rules and the stability promises |
| [Coverage and evidence](reimplementation/coverage-and-evidence.md) | What is covered, how strong the evidence is, and what remains uncertain |

## Reading order for a complete rebuild

Platform, then data, then runtime, then interfaces, then the domains in the order of the build sequence, consulting the reference pages as needed, with the reimplementation folder open throughout.
