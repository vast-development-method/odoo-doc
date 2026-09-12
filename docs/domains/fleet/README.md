# Fleet

## Scope

This domain specifies how the system keeps the register of the vehicles a company owns, leases or lends, and everything that is recorded against a vehicle over its working life: who drives it, what it costs, what has been serviced on it, what coverage contracts protect it, how far it has travelled, and how those facts reach the ledger and the people register.

The centre of the domain is the **Vehicle**. A Vehicle is one physical machine — a car or a bicycle — identified by a licence plate and a chassis number, described by a **Vehicle Model** which itself belongs to a **Vehicle Manufacturer**, classified by a **Vehicle Category** and by free **Vehicle Tags**, placed in a configurable **Vehicle Status** that drives the pipeline board, and assigned to a driver who is a Contact and, where the people register is installed, also an Employee.

Around the Vehicle sit five record streams:

1. **Driver Assignment Logs** — an append-only history of who drove the vehicle and between which dates. Every change of driver opens a new entry automatically.
2. **Odometer Readings** — dated distance measurements, each carrying the value, the unit taken from the vehicle, the reading date and the driver at the time of the reading.
3. **Vehicle Services** — one record per piece of work done on the vehicle: its kind (a **Fleet Service Type** of the service category), its date, its cost, its vendor, its vendor reference, the odometer reading taken at the time, a four-stage progress and a discussion thread. A service can be typed by hand or created automatically when a vendor bill that names a vehicle is posted.
4. **Vehicle Contracts** — coverage agreements such as a lease or an all-risk insurance, each carrying a start date, an expiration date, a one-off activation cost, a recurring cost with its frequency, an insurer, a reference, the included service types, a four-state lifecycle and a renewal reminder that a daily scheduled job raises as an activity.
5. **Analysis results** — two read-only derived result sets: a monthly **Fleet Analysis Report** that spreads service costs and contract costs over calendar months, and a **Fleet Odometer Analysis Report** that interpolates a monthly distance travelled from sparse odometer readings.

One assistant completes the set: the **Send Mails to Drivers** helper, which composes one message per selected vehicle and posts it to the vehicle's thread addressed to its driver.

Everything in this folder is derived from the behaviour of four capability packages: the **Fleet** package, which owns every entity listed above; the **Accounting and Fleet bridge**, which puts a vehicle reference on journal items, turns posted vendor bill lines into Vehicle Services and carries the vehicle reference through the period-change assistant; the **Fleet History** bridge to the people register, which links drivers to Employees in both directions, publishes a mobility card, adds a fleet-manager responsible kind to activity plans and frees company vehicles when an employee leaves; and the **Transport Dispatch** bridge, which gives Vehicle Categories a weight and volume capacity and lets a batch transfer name the vehicle and the category that will carry it.

The domain does **not** cover: how a vendor bill is entered, taxed, posted, paid or reversed (see [accounts payable](../accounts-payable/) and [general ledger](../general-ledger/)); how a journal entry is balanced and what accounts it uses (see [general ledger](../general-ledger/)); how analytic distributions are computed (see [analytic accounting](../analytic-accounting/)); how Employees, work contacts and departure processing work (see [human resources core](../human-resources-core/)); how Contacts are modelled (see [contacts and organizations](../contacts-and-organizations/)); how discussion threads, followers, message subtypes, activities and activity plans work (see [messaging and activities](../messaging-and-activities/)); how batch transfers, operation types, warehouses and locations work (see [inventory operations](../inventory-operations/) and [delivery and shipping](../delivery-and-shipping/)); and how user groups, access rights and record rules are evaluated (see [identity and access](../identity-and-access/) and [../../overview/security-model.md](../../overview/security-model.md)).

Where this domain has to name a mechanism owned by another domain — posting a journal entry, scheduling an activity, resolving the current company — it states exactly which inputs it passes and which result it expects, and links to the owning domain instead of restating it.

## Business scope

1. **Keep a vehicle register.** Record every vehicle with its licence plate, chassis number, model, manufacturer, category, tags, registration date, order date, cancellation date, colour, location, seating capacity, number of doors, fuel kind, transmission, power, range, emissions figure and emission standard, together with user-defined properties whose definition lives on the model.
2. **Describe vehicles once, on the model.** Seventeen descriptive attributes are held on the Vehicle Model and copied onto a vehicle when its model is set or changed; each copied attribute stays editable on the vehicle afterwards.
3. **Track who drives what.** Assign a driver, plan a future driver, apply the planned change in one operation, and keep an assignment history with start and end dates. Where the people register is installed, the driver Contact and the driver Employee are kept in step in both directions.
4. **Track distance.** Record odometer readings by hand, from the vehicle form, or as a by-product of recording a service. Refuse a reading that would lower the vehicle's recorded distance.
5. **Track work done.** Record services with their kind, cost, vendor, reference, notes and progress stage, and create them automatically from posted vendor bill lines that name a vehicle.
6. **Track coverage.** Record contracts with their start and expiration dates, activation cost, recurring cost and frequency, insurer, reference and included services; move them automatically between the incoming, running, expired and cancelled states as dates pass; raise a renewal activity a configurable number of days before expiry.
7. **Warn about renewals.** Flag on each vehicle whether a contract renewal is due soon or already overdue, and let those flags be searched.
8. **Report costs.** Spread one-off and recurring contract costs and service costs over calendar months per vehicle, and analyse them by vehicle, driver, cost kind, fuel kind and period.
9. **Report distance.** Interpolate a monthly distance travelled per vehicle from sparse readings, filling gaps between readings proportionally to elapsed days, and present it as a timeline.
10. **Communicate with drivers.** Select vehicles and send one message per vehicle to its driver, optionally from a stored message template, with attachments, and save the composed text as a new template.
11. **Handle departures.** When an employee leaves, close the assignment history entries and release the vehicles they drove.
12. **Bridge to accounting.** Carry a vehicle reference on journal items, keep it through accrual period changes, count the vendor bills of a vehicle and open them.
13. **Bridge to transport dispatch.** Give categories a maximum weight and a maximum volume, let a batch transfer name the carrying vehicle and category, derive the driver from the vehicle, and report the share of capacity used.

## Capabilities covered

| Capability | Where specified |
|---|---|
| The Vehicle record: identity, description, dates, values, physical attributes, properties, counters and flags | [entities.md](entities.md) |
| The seventeen attributes copied from the Vehicle Model to the Vehicle, and the rule that decides when each is copied | [entities.md](entities.md), [calculations.md](calculations.md) |
| Vehicle Models, Manufacturers, Categories, Statuses and Tags with their counters, uniqueness rules and ordering | [entities.md](entities.md), [configuration.md](configuration.md) |
| Fleet Service Types and the two categories that decide where each type may be used | [entities.md](entities.md), [configuration.md](configuration.md) |
| Driver assignment: driver, future driver, the apply-new-driver operation, the assignment history and its automatic entries | [workflows.md](workflows.md), [business-rules.md](business-rules.md), [state-machines.md](state-machines.md) |
| The plan-to-change flags for cars and for bicycles, and every path that sets or clears them | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Odometer Readings: the three ways one is created, the never-decrease guard, the unit, the driver derivation and the display name | [entities.md](entities.md), [business-rules.md](business-rules.md), [calculations.md](calculations.md) |
| Vehicle Services: fields, the four-stage progress, the odometer link, the purchaser derivation and the archive cascade | [entities.md](entities.md), [state-machines.md](state-machines.md) |
| Vehicle Contracts: fields, the four-state lifecycle, the date-driven re-evaluation on write, the daily scheduler and the renewal activity | [entities.md](entities.md), [state-machines.md](state-machines.md), [workflows.md](workflows.md) |
| Contract renewal flags on the vehicle, their computation, and the two search rules that back them | [calculations.md](calculations.md), [business-rules.md](business-rules.md) |
| The monthly Fleet Analysis Report: its two halves, its month generation, its daily, monthly and yearly recurring-cost arithmetic | [calculations.md](calculations.md) |
| The Fleet Odometer Analysis Report: the fifteen-step interpolation with every formula and a worked example | [calculations.md](calculations.md) |
| The vendor-bill bridge: when a posted bill line becomes a Vehicle Service, what the service carries, what happens when the line changes or is deleted | [workflows.md](workflows.md), [accounting-effects.md](accounting-effects.md), [business-rules.md](business-rules.md) |
| The vehicle dimension on journal items, and its propagation through the accrual period-change assistant | [accounting-effects.md](accounting-effects.md) |
| Every ledger consequence the domain triggers, item by item, and the reasoned statement that the domain posts nothing itself | [accounting-effects.md](accounting-effects.md) |
| The people bridge: driver Employee, future driver Employee, mobility card, employee vehicle counters, licence plate aggregation, the work-contact guard | [entities.md](entities.md), [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Departure handling: closing assignment entries and releasing vehicles | [workflows.md](workflows.md) |
| The fleet-manager responsible kind on activity plans, its constraint and its resolution rules | [configuration.md](configuration.md), [business-rules.md](business-rules.md) |
| The transport dispatch bridge: category capacities, batch vehicle and category, dock handling, capacity percentages | [entities.md](entities.md), [calculations.md](calculations.md), [workflows.md](workflows.md) |
| The send-mail helper: recipients, rendering, the missing-address refusal, the save-as-template operation | [interfaces.md](interfaces.md), [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Settings, groups, access rights, record rules, the scheduled job, the activity type, the message subtype, the activity plan template and every shipped record | [configuration.md](configuration.md) |
| Menus, views, named operations, bound operations, analysis views and printable output | [interfaces.md](interfaces.md) |
| Every validation, invariant, guard and exact message text | [business-rules.md](business-rules.md) |
| Numbered acceptance scenarios with concrete records, inputs, amounts, dates and states | [acceptance-criteria.md](acceptance-criteria.md) |
| Every term of the domain | [glossary.md](glossary.md) |

## Capabilities delivered, in one line each

| Capability | Summary |
|---|---|
| Register a vehicle | Create a Vehicle from a Vehicle Model; seventeen attributes are copied from the model and remain editable. |
| Name a vehicle | The display name is built from manufacturer, model and licence plate, with a fallback text when there is no plate. |
| Classify a vehicle | Give it a category, tags, a configurable status and user-defined properties defined on its model. |
| Assign a driver | Set the driver; an assignment history entry opens automatically and a reminder activity asks for the previous driver's end date. |
| Plan a driver change | Set a future driver; vehicles of the same kind currently driven by that person are flagged as planned for change. |
| Apply a driver change | One operation clears the driver on the vehicles the incoming driver is giving up and promotes the future driver to driver. |
| Record a distance | Create an odometer reading by hand, through the vehicle's distance field, or through a service record. |
| Refuse a lower distance | Writing a lower distance through the vehicle's distance field is refused with an exact message. |
| Record a service | Log the work done with its kind, date, cost, vendor, reference, notes, progress stage and the distance at the time. |
| Bill a service | Posting a vendor bill line that names a vehicle creates the matching Vehicle Service and links the two. |
| Protect a billed service | The cost of a service that came from a bill cannot be typed, and such a service cannot be deleted. |
| Record a contract | Log a coverage agreement with dates, costs, frequency, insurer, reference and the service types it includes. |
| Move a contract through its life | Four states — incoming, running, expired, cancelled — reached by four operations, by writing dates, or by the daily job. |
| Warn before expiry | A daily job raises a renewal activity on every running contract expiring within the configured number of days. |
| Flag a vehicle that needs action | Two computed flags say whether a renewal is due soon or overdue; both are searchable. |
| Analyse costs | A monthly result set per vehicle splits contract and service costs across calendar months. |
| Analyse distance | A monthly result set per vehicle interpolates distance travelled between sparse readings. |
| Archive a vehicle | Archiving a vehicle archives all of its contracts and all of its services, after an explicit confirmation. |
| Mail the drivers | Compose one message per selected vehicle, rendered per vehicle when a template is used, and post it to the driver. |
| Save a message template | Turn the composed subject, body and attachments into a reusable template bound to the Vehicle. |
| Link drivers to employees | The driver Contact and the driver Employee are kept in step in both directions, per company. |
| Publish a mobility card | The employee's mobility card reference is mirrored onto every vehicle that employee drives. |
| Release vehicles on departure | The departure assistant closes assignment entries and clears the driver on every vehicle the leaver drove. |
| Route activity plans to the fleet manager | An activity plan line may name the fleet manager of the employee's first vehicle as the responsible user. |
| Dispatch with a vehicle | A batch transfer names the vehicle and the vehicle category that will carry it and reports the share of capacity used. |

## Entities this folder owns

| Full name | Transport name | Kind | Reference page |
|---|---|---|---|
| Vehicle | `fleet.vehicle` | persistent | [../../references/entities/fleet.vehicle.md](../../references/entities/fleet.vehicle.md) |
| Vehicle Model | `fleet.vehicle.model` | persistent | [../../references/entities/fleet.vehicle.model.md](../../references/entities/fleet.vehicle.model.md) |
| Vehicle Manufacturer | `fleet.vehicle.model.brand` | persistent | [../../references/entities/fleet.vehicle.model.brand.md](../../references/entities/fleet.vehicle.model.brand.md) |
| Vehicle Category | `fleet.vehicle.model.category` | persistent | [../../references/entities/fleet.vehicle.model.category.md](../../references/entities/fleet.vehicle.model.category.md) |
| Vehicle Status | `fleet.vehicle.state` | persistent | [../../references/entities/fleet.vehicle.state.md](../../references/entities/fleet.vehicle.state.md) |
| Vehicle Tag | `fleet.vehicle.tag` | persistent | [../../references/entities/fleet.vehicle.tag.md](../../references/entities/fleet.vehicle.tag.md) |
| Fleet Service Type | `fleet.service.type` | persistent | [../../references/entities/fleet.service.type.md](../../references/entities/fleet.service.type.md) |
| Vehicle Contract | `fleet.vehicle.log.contract` | persistent | [../../references/entities/fleet.vehicle.log.contract.md](../../references/entities/fleet.vehicle.log.contract.md) |
| Vehicle Service | `fleet.vehicle.log.services` | persistent | [../../references/entities/fleet.vehicle.log.services.md](../../references/entities/fleet.vehicle.log.services.md) |
| Odometer Reading | `fleet.vehicle.odometer` | persistent | [../../references/entities/fleet.vehicle.odometer.md](../../references/entities/fleet.vehicle.odometer.md) |
| Driver Assignment Log | `fleet.vehicle.assignation.log` | persistent | [../../references/entities/fleet.vehicle.assignation.log.md](../../references/entities/fleet.vehicle.assignation.log.md) |
| Fleet Analysis Report | `fleet.vehicle.cost.report` | derived result set | [../../references/entities/fleet.vehicle.cost.report.md](../../references/entities/fleet.vehicle.cost.report.md) |
| Fleet Odometer Analysis Report | `fleet.vehicle.odometer.report` | derived result set | [../../references/entities/fleet.vehicle.odometer.report.md](../../references/entities/fleet.vehicle.odometer.report.md) |
| Send Mails to Drivers | `fleet.vehicle.send.mail` | transient assistant | [../../references/entities/fleet.vehicle.send.mail.md](../../references/entities/fleet.vehicle.send.mail.md) |

The two analysis result sets are not stored tables that a user writes to. They are read-only derived sets rebuilt from the stored records; their derivation is specified step by step in [calculations.md](calculations.md).

## Entities this folder extends but does not own

| Full name | Transport name | Owning domain | What this domain adds |
|---|---|---|---|
| Journal Entry | `account.move` | [general ledger](../general-ledger/) | Posting a vendor bill creates one Vehicle Service per product line that names a vehicle |
| Journal Item | `account.move.line` | [general ledger](../general-ledger/) | A vehicle reference, a vehicle-required flag and the back reference to the generated Vehicle Service |
| Create Automatic Entries | `account.automatic.entry.wizard` | [general ledger](../general-ledger/) | The vehicle reference is carried onto the generated counterpart item that uses the same account |
| Employee | `hr.employee` | [human resources core](../human-resources-core/) | Vehicle counters, the vehicle collection, an aggregated licence plate, a mobility card, a work-contact guard and a two-way contact synchronisation |
| Public Employee Profile | `hr.employee.public` | [human resources core](../human-resources-core/) | A read-only mobility card |
| Departure Wizard | `hr.departure.wizard` | [human resources core](../human-resources-core/) | A release-the-company-vehicle option and the release procedure |
| Activity Plan Template | `mail.activity.plan.template` | [messaging and activities](../messaging-and-activities/) | A fleet-manager responsible kind, its constraint and its resolution rule |
| Activity Type | `mail.activity.type` | [messaging and activities](../messaging-and-activities/) | The renewal activity type is registered as generated once and never removed by the job that raises it |
| Attachment | `ir.attachment` | platform foundation, see [../../runtime/attachments-and-file-store.md](../../runtime/attachments-and-file-store.md) | A preview operation that opens the stored file in a new window |
| Configuration Settings | `res.config.settings` | platform foundation, see [../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md) | The contract-expiry alert delay |
| Batch Transfer | `stock.picking.batch` | [inventory operations](../inventory-operations/) | A vehicle, a vehicle category, a dock, a driver, an end date and two capacity-usage percentages |
| Operation Type | `stock.picking.type` | [inventory operations](../inventory-operations/) | A dispatch-management flag and the set of dock locations |
| Transfer | `stock.picking` | [inventory operations](../inventory-operations/) | The customer postal code, a dock-driven destination reset and the batch ordering by postal code |
| Warehouse | `stock.warehouse` | [inventory operations](../inventory-operations/) | Dispatch management is switched on for the outgoing, incoming and intermediate operation types |

Every addition above is specified in full in [entities.md](entities.md); the behaviour each one triggers is specified in [workflows.md](workflows.md) and [business-rules.md](business-rules.md).

## Entities this folder deliberately does not own

The scope list that produced this folder also names generic platform entities that appear in the domain's configuration files. They belong to the platform foundation, not to this domain: window actions, server actions, menus, views, record rules, access rights, scheduled jobs, groups and group privileges. Their structure is specified in [../../overview/views-and-actions.md](../../overview/views-and-actions.md), [../../overview/security-model.md](../../overview/security-model.md) and [../../runtime/scheduled-jobs.md](../../runtime/scheduled-jobs.md); the concrete records this domain ships of each kind are listed in [configuration.md](configuration.md) and [interfaces.md](interfaces.md).

## Reading order

1. [README.md](README.md) — this file: scope, capabilities, entity ownership and the map of the folder.
2. [glossary.md](glossary.md) — the vocabulary. Read it first if any term below is unfamiliar.
3. [entities.md](entities.md) — every entity, every field, every relation, every default and every computed rule.
4. [state-machines.md](state-machines.md) — the contract lifecycle, the service progress, the vehicle status pipeline and the driver-assignment machine.
5. [workflows.md](workflows.md) — the end-to-end procedures that create and change those records.
6. [business-rules.md](business-rules.md) — every validation, guard and permission check, with its exact message.
7. [calculations.md](calculations.md) — every formula, with rounding and worked examples.
8. [accounting-effects.md](accounting-effects.md) — what reaches the ledger, and by which route.
9. [configuration.md](configuration.md) — settings, groups, access rights, record rules, jobs, templates and shipped records.
10. [interfaces.md](interfaces.md) — menus, views, operations, analysis screens, printable output and integration points.
11. [acceptance-criteria.md](acceptance-criteria.md) — numbered scenarios that decide whether a rebuild is correct.

## Dependencies on other domains

| Domain | What this domain needs from it |
|---|---|
| [contacts and organizations](../contacts-and-organizations/) | The Contact used as driver, future driver, insurer, vendor and model vendor; the electronic mail address checked before a driver message is sent |
| [identity and access](../identity-and-access/) | Users, groups and group privileges; the fleet manager is a user, and the fleet officer and fleet administrator groups are defined against that model |
| [messaging and activities](../messaging-and-activities/) | Discussion threads, followers, tracked-field logging, message subtypes, message templates, scheduled activities, activity types and activity plans |
| [human resources core](../human-resources-core/) | Employees, work contacts, the private vehicle plate, the departure assistant and the offboarding activity plan |
| [general ledger](../general-ledger/) | Journal entries and journal items; the vehicle dimension is a field on the journal item, and the period-change assistant is extended |
| [accounts payable](../accounts-payable/) | Vendor bills; posting one is the event that creates Vehicle Services |
| [multi-currency](../multi-currency/) | The currency of monetary amounts, taken from the owning company |
| [inventory operations](../inventory-operations/) | Batch transfers, operation types, transfers, warehouses and locations used by the transport dispatch bridge |
| [delivery and shipping](../delivery-and-shipping/) | The shipping weight and shipping volume of a transfer, compared against the vehicle category's capacities |
| [units of measure and packaging](../units-of-measure-and-packaging/) | The weight and volume unit labels shown next to a category's capacities |
| [analytic accounting](../analytic-accounting/) | The analytic naming extension point a localization may use to turn a vehicle into an analytic dimension |

## Domains that depend on this one

| Domain | What it takes from here |
|---|---|
| [human resources core](../human-resources-core/) | The employee's vehicle list, vehicle counter, aggregated licence plate and mobility card, and the release of vehicles at departure |
| [general ledger](../general-ledger/) | The vehicle dimension on journal items and its survival through accrual period changes |
| [accounts payable](../accounts-payable/) | The vehicle column offered on vendor bill lines |
| [inventory operations](../inventory-operations/) | The vehicle and vehicle category named on a batch transfer, and the capacities the category publishes |
| [delivery and shipping](../delivery-and-shipping/) | The dock location and vehicle shown on the batch transfer document |

## Files in this folder

| File | Content |
|---|---|
| [README.md](README.md) | Scope, business scope, capabilities, entity ownership, reading order, dependencies and this file list |
| [entities.md](entities.md) | Every entity in full: purpose, lifecycle, complete field tables, relations, uniqueness, ordering, display names, archival, company behaviour and the fields other packages contribute |
| [state-machines.md](state-machines.md) | Every state field with its stored values, transition tables, guards, refusal messages, side effects and a diagram per machine |
| [workflows.md](workflows.md) | The end-to-end procedures, step by step, with the records each step creates or changes and the conditions that make it fail |
| [business-rules.md](business-rules.md) | Numbered validations, constraints, invariants, permission checks and exact messages, with a rule index |
| [calculations.md](calculations.md) | Every formula and algorithm with rounding, units, currency handling and worked numeric examples |
| [accounting-effects.md](accounting-effects.md) | The reasoned statement that this domain posts no journal entries itself, and every ledger effect it triggers elsewhere, item by item |
| [configuration.md](configuration.md) | Settings, system parameters, groups, access rights, record rules, the scheduled job, activity types, message subtypes, activity plan templates and every shipped record |
| [interfaces.md](interfaces.md) | Menus, views, named operations, bound operations, analysis screens, printable output, message templates, import and export |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete numbers |
| [glossary.md](glossary.md) | Every term of the domain, defined |
