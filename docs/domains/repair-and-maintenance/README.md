# Repair and Maintenance

## Scope

This domain specifies two related but structurally independent capabilities of the system.

The first is **repair**: the handling of a physical item that a customer, or the company itself, wants restored to working order. A Repair Order names the product to repair (optionally a specific lot or serial number), the parts that will be consumed into it, the parts that will be taken out of it and thrown away, and the parts that will be taken out of it and returned to stock. It moves through a five-state lifecycle, it reserves its parts from stock exactly as an ordinary warehouse document does, it can be priced and billed through a quotation and a Sales Order, it can be declared "under warranty" so that nothing is charged, it can be created automatically from a customer return, from a Sales Order Line whose product is configured as a repair service, or by hand, and when it is completed it produces a precise set of inventory movements whose valuation and accounting consequences are owned by the inventory valuation and costing domain.

The second is **maintenance**: the register of Equipment owned or used by the company, the Maintenance Teams responsible for it, and the Maintenance Requests raised against it. A request is either *corrective* — something broke, fix it — or *preventive* — the calendar says it is time to service this machine. Requests travel across a configurable pipeline of stages, one or more of which are marked as closing stages. Preventive requests may be *recurrent*: when a recurrent preventive request reaches a closing stage the system immediately generates its successor, dated by adding the configured repeat interval to the current scheduled date. Equipment carries effectiveness measurements — mean time between failures, mean time to repair, latest failure date and estimated next failure — computed from the closed corrective requests raised against it.

The two capabilities share no records. A Repair Order does not reference Equipment, and a Maintenance Request does not create Stock Moves. They are documented together because they answer the same operational question — *this thing is broken, what do we do about it* — from two directions, the outward-facing one (a customer's product) and the inward-facing one (the company's own assets), and because the bridge packages that extend them (to manufacturing, purchasing, the point of sale, inventory, people management and subcontracting) sit between the same set of neighbouring domains.

Everything in this folder is derived from the behaviour of the following capability packages: the Repair package; the Maintenance package; the Repair–Manufacturing bridge; the Repair–Purchasing bridge; the Repair–Point of Sale bridge; the Inventory–Maintenance bridge; the People–Maintenance bridge; the Subcontracting–Repair bridge; and the document-layout localization bridge that stamps a printing date on the printed Repair Order.

The domain does **not** cover: how Stock Moves reserve, execute and merge (see [inventory operations](../inventory-operations/)); how a validated Stock Move is valued and which journal entries it creates (see [inventory valuation and costing](../inventory-valuation-and-costing/)); how a Sales Order is confirmed, invoiced and paid (see [sales](../sales/)); how a Purchase Order is raised and received (see [purchasing](../purchasing/)); how a bill of materials is exploded during manufacturing (see [manufacturing](../manufacturing/)); how procurement rules, the replenish-on-order route and the forecast report work (see [replenishment and procurement](../replenishment-and-procurement/)); how activities, followers, message subtypes and incoming-mail aliases work (see [messaging and activities](../messaging-and-activities/)); how employees and departments are modelled (see [human resources core](../human-resources-core/)).

Where this domain has to name a mechanism owned by another domain — a reservation pass, a unit conversion, a costing method — it states exactly which inputs it passes and which result it expects, and links to the owning domain rather than restating it.

## Business scope

1. **Repair intake.** A Repair Order names one product to repair, optionally one lot or serial number of that product, the customer it belongs to, whether the repair is covered by warranty, the operation type that governs its locations and numbering, the scheduled date, a responsible user, free tags, internal notes and user-defined properties. It may be created by hand, from an incoming return transfer, from a lot or serial number, or automatically from a confirmed Sales Order Line whose product is configured to trigger a repair.
2. **Parts handling.** Every part touched during the repair is a Stock Move attached to the Repair Order and classified as *add*, *remove* or *recycle*. Each class has its own pair of source and destination locations, taken from the Repair Order, which themselves default from the operation type. Added parts leave stock and enter the production location, removed parts leave the production location for a loss location, recycled parts leave the production location and return to stock. Availability of the added parts is forecast and reported as a readiness state.
3. **Repair execution.** The Repair Order moves through New, Confirmed, Under Repair and Repaired, or is cancelled. Confirmation reserves the parts and warns when the product to repair is not physically available. Completion records every part movement as done, creates one further movement that carries the repaired product itself (which makes the repair appear in the traceability of the serial number), and closes the order.
4. **Billing.** A quotation can be created from the Repair Order; every *add* part becomes a Sales Order Line, while *remove* and *recycle* parts are never billed. When the Repair Order is marked under warranty the unit price of those lines is forced to zero. Quantities flow from the Repair Order to the Sales Order and never the other way. When the repair originates from a service line of a Sales Order, completing the repair also reports that service line as delivered.
5. **Replenishment and production links.** A part that is not in stock can be pulled through the replenishment rules: a Manufacturing Order or a Purchase Order may be created to feed the Repair Order, and both link back to it. A part that is a kit is exploded into its components on the Repair Order.
6. **Equipment register.** An Equipment record carries a name, a category, a vendor and vendor reference, a model designation, a serial number, an acquisition cost, a warranty expiration date, an effective date, a scrap date, an internal location, a Maintenance Team, a technician, an owner, an assignment to an employee or a department, notes, a colour and user-defined properties whose definition lives on the category.
7. **Maintenance requests.** A Maintenance Request names a subject, an equipment (optional), a kind (corrective or preventive), a request date, a scheduled start and end, a duration, a priority, a team, a technician, a kanban state, instructions in one of three media, and a recurrence rule for preventive work. Requests travel across configurable stages; a stage flagged as closing closes the request and stamps its close date, and a recurrent preventive request spawns its successor at that moment.
8. **Reliability statistics.** From the closed corrective requests of an item, the system derives the mean time between failures, the mean time to repair, the date of the latest failure and the estimated date of the next failure.
9. **Collaboration.** Both Repair Orders and Maintenance Requests carry a discussion thread, followers, scheduled activities and tracked field changes. A Maintenance Team publishes an incoming-mail alias that turns an inbound message into a Maintenance Request assigned to that team.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Repair Orders: reference numbering per operation type, customer, product to repair, lot or serial number, quantity and unit, scheduling, priority, tags, responsible user, internal notes, properties | [entities.md](entities.md), [configuration.md](configuration.md) |
| The six repair locations (component source, product source, product destination, added-parts destination, removed-parts destination, recycled-parts destination) and how each is defaulted from the operation type | [entities.md](entities.md), [calculations.md](calculations.md) |
| Repair parts: the three part kinds (add, remove, recycle), the source and destination location each kind implies, quantity and picked flag, catalog-driven entry | [entities.md](entities.md), [calculations.md](calculations.md), [workflows.md](workflows.md) |
| The Repair Order state machine: new, confirmed, under repair, repaired, cancelled — with every transition, guard and side effect, including reset to draft | [state-machines.md](state-machines.md) |
| Confirmation: the insufficient-quantity check on the product to repair, the warning dialogue, procurement method adjustment, move confirmation, scheduler trigger | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Completion: cancellation of zero-quantity parts, the serial-number requirement, owner detection, the generated product move with its consumption links, backorder suppression, delivered-quantity feedback to the Sales Order Line | [workflows.md](workflows.md), [calculations.md](calculations.md) |
| Parts availability: the three availability states, the forecast comparison, the readiness and lateness booleans, the operation-type dashboard counters | [calculations.md](calculations.md) |
| The inventory movements a repair produces, enumerated move by move with source and destination location, quantity and unit | [workflows.md](workflows.md), [accounting-effects.md](accounting-effects.md) |
| The journal items a repair produces, through the valuation of its moves and through the invoice raised on its Sales Order, with account selection rules and amount formulas | [accounting-effects.md](accounting-effects.md) |
| Warranty: what the flag does to prices at line creation, what it does when toggled afterwards, and what is left uncharged | [workflows.md](workflows.md), [calculations.md](calculations.md), [business-rules.md](business-rules.md) |
| The Sales Order link: quotation creation from a repair, line creation and update from parts, line cancellation, delivered quantity derivation, repair creation from a confirmed Sales Order Line, cancellation propagation in both directions | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| The return link: creating a repair from a validated customer return, restriction of the product and lot choice to what came back, quantity derivation, the warehouse-mismatch warning | [workflows.md](workflows.md), [business-rules.md](business-rules.md) |
| Repair operation types: the repair code, the four extra default locations, the four dashboard counters, the property definition, the sequence and barcode created per warehouse | [configuration.md](configuration.md), [entities.md](entities.md) |
| The make-to-order rule created per warehouse for repair components | [configuration.md](configuration.md) |
| Kit explosion of repair parts when a phantom bill of materials exists | [workflows.md](workflows.md), [calculations.md](calculations.md) |
| Purchase Orders and Manufacturing Orders raised from repair components, and the counters that surface them | [interfaces.md](interfaces.md), [workflows.md](workflows.md) |
| Lot and serial traceability through a repair: repaired counts, in-repair counts, the parts used, the consumption links that make the traceability tree | [calculations.md](calculations.md), [interfaces.md](interfaces.md) |
| Equipment: identity, category, vendor and vendor reference, model designation, serial number, assignment to an employee or a department or neither, owner, technician, team, cost, warranty expiry, scrap date, location, properties | [entities.md](entities.md) |
| Equipment effectiveness: mean time between failures, mean time to repair, latest failure date, estimated next failure, expected mean time between failures, with exact formulas and worked examples | [calculations.md](calculations.md) |
| Equipment categories: responsible technician, colour, folding rule, equipment and request counters, property definitions, deletion guard | [entities.md](entities.md), [business-rules.md](business-rules.md) |
| Maintenance Teams: members, company, the request dashboard counters, the incoming-mail alias that creates requests | [entities.md](entities.md), [configuration.md](configuration.md), [interfaces.md](interfaces.md) |
| Maintenance Stages: sequence, folding, the closing flag, the shipped stage set | [entities.md](entities.md), [configuration.md](configuration.md) |
| Maintenance Requests: subject, description, kind, request date, schedule window and duration, technician, team, stage, kanban state, priority, close date, archive flag, instructions in three media, employee link | [entities.md](entities.md) |
| The maintenance request pipeline as a state machine over stages, with the closing-stage side effects | [state-machines.md](state-machines.md) |
| The recurring generation algorithm for preventive work, its date arithmetic, its termination rule and its copy semantics, as numbered steps | [calculations.md](calculations.md), [workflows.md](workflows.md) |
| The projection of future occurrences of a recurring request onto the maintenance calendar | [calculations.md](calculations.md), [interfaces.md](interfaces.md) |
| The activity that is scheduled, rescheduled, closed and removed as a request changes | [workflows.md](workflows.md), [interfaces.md](interfaces.md) |
| Followers and tracked fields on Equipment and Maintenance Requests; the assignment notification subtype | [interfaces.md](interfaces.md), [configuration.md](configuration.md) |
| Departure handling: freeing the equipment assigned to a leaving employee | [workflows.md](workflows.md) |
| Equipment and stock: the location field, the serial-number match and the jump to the matching lot record | [workflows.md](workflows.md), [interfaces.md](interfaces.md) |
| Settings, security groups, the complete access rights matrix, record rules, sequences and shipped records | [configuration.md](configuration.md) |
| Every validation, invariant and error message of both capabilities, with exact message text | [business-rules.md](business-rules.md) |
| Numbered acceptance scenarios with concrete numbers | [acceptance-criteria.md](acceptance-criteria.md) |

## Capabilities delivered, in one line each

| Capability | Summary |
|---|---|
| Register a repair | Create a Repair Order for a product, optionally for one lot or serial number, for a customer, under an operation type. |
| Repair from a return | Turn a validated return transfer into a Repair Order that inherits the customer, the returned product, the return's destination location and the warehouse repair operation type. |
| Repair from a lot or serial number | Open, filter and create Repair Orders from the record of a lot or serial number. |
| Repair from a Sales Order | Confirming a Sales Order Line whose product is configured as a repair service creates one confirmed Repair Order per line. |
| Plan parts | List parts to add, remove or recycle, with demand quantity, actual quantity, unit of measure and lot or serial numbers. |
| Pick parts from a catalog | Browse goods in a catalog screen and set quantities directly on the Repair Order. |
| Explode kits | Replace a kit part by the components of its bill of materials on the Repair Order. |
| Forecast readiness | Report whether every added part is available, expected on a date, or late against the scheduled date. |
| Reserve and unreserve | Reserve the added parts from the component source location, and release those reservations. |
| Warn on missing product | Before confirming, verify that the product to repair is physically present, and let the user confirm anyway with an explicit warning. |
| Execute the repair | Move the order to Under Repair, record actual quantities, and end the repair. |
| Record the repair movements | Post every part movement and the movement of the repaired product itself, linking the consumed part movement lines to the repaired product movement line for traceability. |
| Bill the repair | Create a quotation carrying the added parts, keep its quantities in step with the Repair Order, and zero its prices when the repair is under warranty. |
| Cancel and reopen | Cancel a Repair Order (except when it is already completed) and put a cancelled one back to New. |
| Print the repair order | Produce a printed Repair Order listing customer, product, lot or serial number, status, responsible, the parts with their kind and quantity, and the repair notes. |
| Register equipment | Create Equipment with identity, vendor, warranty, location, category, team, technician and assignment. |
| Categorise equipment | Group Equipment in categories that carry a responsible user, comments, a colour and a property definition. |
| Raise a maintenance request | Create a corrective or preventive Maintenance Request by hand, from an equipment record, from the calendar, or by electronic mail to a team alias. |
| Progress a request | Move a request across stages, block or unblock it, archive it and reopen it. |
| Repeat preventive maintenance | Automatically create the successor of a recurring preventive request when it reaches a closing stage. |
| Schedule technician work | Keep one scheduled activity per request, aimed at the technician, deadlined on the scheduled date in the user's time zone. |
| Measure reliability | Compute mean time between failures, mean time to repair, latest failure date and estimated next failure per item. |
| Run team dashboards | Count, per Maintenance Team, the open requests, the scheduled ones, the high-priority ones, the blocked ones and the unscheduled ones. |
| Free equipment on departure | When an employee leaves, optionally unassign every piece of equipment held by that employee. |
| Match equipment to a serial number | Detect that an equipment's serial number is also a registered lot or serial number and open that record. |

## Actors

| Actor | Description |
|---|---|
| Repair Technician | An Inventory User. Creates, confirms, starts and ends Repair Orders, edits parts, reserves and unreserves, creates quotations, prints repair orders. |
| Repair Supervisor | An Inventory Administrator. Everything a Repair Technician may do, plus configuring repair operation types, Repair Tags and reporting. |
| Salesperson | Creates the Sales Order whose confirmation raises a Repair Order, and invoices the resulting quotation. Does not act on the Repair Order. |
| Accountant | Posts and consults the journal entries produced by the part movements and by the customer invoice for the repair. |
| Internal User | Any employee. May read Equipment they follow, and may create, read, update and delete their own Maintenance Requests. |
| Equipment Manager | Creates, edits and deletes Equipment, Equipment Categories, Maintenance Stages and Maintenance Teams, and sees every Maintenance Request and every Equipment record. |
| Maintenance Technician | An Internal User designated on a Maintenance Request or an Equipment record as the person who performs the work. Receives the scheduled activity. |
| Maintenance Team Member | An Internal User listed on a Maintenance Team; appears as a selectable technician on the maintenance calendar. |
| Human Resources Officer | Holds the Equipment Manager role implicitly, and decides on employee departure whether the departing employee's equipment is released. |

## Entities owned by this domain

| Entity | Transport name | Storage name | Reference page | Purpose |
|---|---|---|---|---|
| Repair Order | `repair.order` | `repair_order` | [repair.order.md](../../references/entities/repair.order.md) | One item to restore: what it is, which parts go in and out, when, for whom, and at what price |
| Repair Tag | `repair.tags` | `repair_tags` | [repair.tags.md](../../references/entities/repair.tags.md) | Free classification label attachable to Repair Orders |
| Insufficient Repair Quantity Warning | `stock.warn.insufficient.qty.repair` | `stock_warn_insufficient_qty_repair` | [stock.warn.insufficient.qty.repair.md](../../references/entities/stock.warn.insufficient.qty.repair.md) | Transient confirmation shown when the product to repair is not on hand at its source location |
| Maintained Item | `maintenance.mixin` | *(abstract, no table)* | [maintenance.mixin.md](../../references/entities/maintenance.mixin.md) | Shared behaviour of anything that can be maintained: company, effective date, team, technician, request counters and the four effectiveness measurements |
| Equipment | `maintenance.equipment` | `maintenance_equipment` | [maintenance.equipment.md](../../references/entities/maintenance.equipment.md) | One machine, tool, vehicle or device that is registered, assigned and maintained |
| Equipment Category | `maintenance.equipment.category` | `maintenance_equipment_category` | [maintenance.equipment.category.md](../../references/entities/maintenance.equipment.category.md) | Grouping of equipment with a default responsible technician and a property definition |
| Maintenance Team | `maintenance.team` | `maintenance_team` | [maintenance.team.md](../../references/entities/maintenance.team.md) | Group of users that receives and works Maintenance Requests; owns an incoming-mail alias |
| Maintenance Stage | `maintenance.stage` | `maintenance_stage` | [maintenance.stage.md](../../references/entities/maintenance.stage.md) | One column of the maintenance pipeline; may be marked as a closing stage |
| Maintenance Request | `maintenance.request` | `maintenance_request` | [maintenance.request.md](../../references/entities/maintenance.request.md) | One piece of work to be done on a piece of equipment, corrective or preventive, possibly recurrent |

The candidate entity list of the target taxonomy holds exactly these nine, and this folder owns all nine. No generic platform entity is claimed here; the platform foundation owns those, and the platform documents named under [Reading order](#reading-order) describe them.

## Entities owned by other domains and extended here

| Entity | Transport name | Owning domain | What this domain adds |
|---|---|---|---|
| Stock Move | `stock.move` | [inventory operations](../inventory-operations/) | The Repair Order link and the part kind (`add`, `remove`, `recycle`), the location derivation from that kind, the reference and origin taken from the Repair Order, the creation and maintenance of the backing Sales Order Line, the suppression of splitting and of automatic assignment, and the forecast override for removed and recycled parts |
| Stock Move Line | `stock.move.line` | [inventory operations](../inventory-operations/) | Lot and serial numbers of repair part movements are shown on the customer invoice |
| Transfer | `stock.picking` | [inventory operations](../inventory-operations/) | The list of Repair Orders sourced from the transfer, their count, the create-repair action, and the exclusion of repair-backed Sales Order Lines when a point-of-sale order builds its delivery |
| Operation Type | `stock.picking.type` | [inventory operations](../inventory-operations/) | The repair code, the four extra default locations, the repair property definition, the four repair counters of the overview screen, and the date aggregation used by the overview graph |
| Warehouse | `stock.warehouse` | [inventory operations](../inventory-operations/) | The repair operation type of the warehouse, its numbering sequence, its replenish-on-order rule, and the creation of the production location when missing |
| Location | `stock.location` | [inventory operations](../inventory-operations/) | The count of Equipment used in the location and the action that lists it |
| Lot or Serial Number | `stock.lot` | [inventory operations](../inventory-operations/) | The Repair Orders in which the lot was used as a part, the count of repairs in progress and completed for the lot, the actions that open or create Repair Orders from the lot, and the guard that forbids creating a lot from a repair whose operation type does not allow it |
| Stock Reference | `stock.reference` | [inventory operations](../inventory-operations/) | The association that ties a Repair Order to the documents created to replenish its parts |
| Traceability Report | `stock.traceability.report` | [inventory operations](../inventory-operations/) | A movement line of a repair reports the Repair Order as its source document, and the consumed and produced links of the repaired-product movement are followed |
| Forecasted Stock Report | `report.stock.report_product_product_replenishment` | [replenishment and procurement](../replenishment-and-procurement/) | Repair part movements are excluded from reservation data, and a movement bound both to a Repair Order and to a Sales Order Line is counted once, as a repair |
| Sales Order | `sale.order` | [sales](../sales/) | The list of Repair Orders raised by the order, their count, the action that opens them, and the creation or cancellation of those Repair Orders when the order is confirmed or cancelled |
| Sales Order Line | `sale.order.line` | [sales](../sales/) | Creation and cancellation of the bound Repair Order on quantity changes, the delivered quantity taken from the repair movement, the suppression of the ordinary delivery rule for repair-backed lines, the flag that marks a line as backed by a repair, and the exclusion of repair movements from value checks |
| Product Variant | `product.product` | [products and catalog](../products-and-catalog/) | The catalog membership flag used when picking parts, the counting of returned serial numbers through removed and recycled parts, and the guard that blocks a unit-of-measure change when repairs already use another unit |
| Product Template | `product.template` | [products and catalog](../products-and-catalog/) | The service-tracking value that makes a sold service raise a Repair Order, and its inclusion in the saleable tracking kinds |
| Journal Item | `account.move.line` | [general ledger](../general-ledger/) | The rule that suppresses the cost-of-goods-sold item on a customer invoice when the repair part movement already produced its own valuation entry |
| Manufacturing Order | `mrp.production` | [manufacturing](../manufacturing/) | The count of Repair Orders that this production feeds and the action that opens them |
| Purchase Order | `purchase.order` | [purchasing](../purchasing/) | The count of Repair Orders that this purchase feeds and the action that opens them |
| Employee | `hr.employee` | [human resources core](../human-resources-core/) | The Equipment assigned to the employee and its count |
| Public Employee Profile | `hr.employee.public` | [human resources core](../human-resources-core/) | The count of Equipment assigned to the employee, readable without private-data rights |
| Departure Wizard | `hr.departure.wizard` | [human resources core](../human-resources-core/) | The option that releases every piece of Equipment held by the departing employee |
| Configuration Settings | `res.config.settings` | [platform foundation](../platform-foundation/) | The switch that installs custom maintenance worksheets |

## Reading order

1. [README.md](README.md) — this file: scope, capability map, entity list, actors, dependencies.
2. [glossary.md](glossary.md) — every term used below, defined in full. Read it first if any term is unfamiliar.
3. [entities.md](entities.md) — the complete field tables, relations, defaults, computed rules, ordering, display names, archival and multi-company behaviour of each entity.
4. [state-machines.md](state-machines.md) — the Repair Order lifecycle, the readiness machine, the Stock Move states reached inside a repair, the Maintenance Request pipeline and its kanban state, with transition tables, guards, side effects and diagrams.
5. [workflows.md](workflows.md) — the end-to-end operational sequences: raising a repair by hand, from a return, from a Sales Order; confirming, starting, completing, cancelling and resetting one; quoting and invoicing one; raising, working, closing and repeating a Maintenance Request; assigning and freeing equipment.
6. [calculations.md](calculations.md) — every formula and algorithm: availability, quantity derivation, location mapping, warranty pricing, the recurring generation date arithmetic, the effectiveness measurements, duration, kit explosion, calendar projection, with worked numeric examples.
7. [business-rules.md](business-rules.md) — every validation, constraint, invariant, permission check and edge case, with exact error messages and the numbered rule catalogue.
8. [accounting-effects.md](accounting-effects.md) — the journal entries the domain causes, directly and indirectly, with account selection rules and amount formulas.
9. [configuration.md](configuration.md) — capability packages, settings, sequences and numbering formats, shipped records, security groups, the access rights matrix, record rules and scheduled execution.
10. [interfaces.md](interfaces.md) — menus, window actions, views, buttons, filters, groupings, named operations, printable documents, notifications, screen paths and integration points.
11. [acceptance-criteria.md](acceptance-criteria.md) — numbered Given, When and Then scenarios with concrete numbers, to be used as the conformance suite.

Platform-wide material that this folder assumes rather than restates: the record, field and access model in [../../overview/README.md](../../overview/README.md), the execution and scheduling model in [../../runtime/README.md](../../runtime/README.md), the storage and transport model in [../../data/README.md](../../data/README.md), and the client and integration surfaces in [../../interfaces/README.md](../../interfaces/README.md).

## Dependencies on other domains

| Depends on | For what | Link |
|---|---|---|
| Inventory operations | Stock Moves, Stock Move Lines, reservation, the done pass, locations, operation types, warehouses, lots and serial numbers, stock quantity records, and the transfer that returns goods from a customer | [../inventory-operations/](../inventory-operations/) |
| Replenishment and procurement | Procurement rules, the replenish-on-order route, the scheduler trigger and the forecast report; a repair part that is not in stock is replenished by those rules | [../replenishment-and-procurement/](../replenishment-and-procurement/) |
| Inventory valuation and costing | The valuation layers and journal entries produced by the moves a repair validates, and the cost of the product to repair | [../inventory-valuation-and-costing/](../inventory-valuation-and-costing/) |
| Sales | Sales Orders, Sales Order Lines, their confirmation, invoicing and cancellation; the service-tracking selector; delivered quantity | [../sales/](../sales/) |
| Purchasing | Purchase Orders raised for repair components through the make-to-order rule | [../purchasing/](../purchasing/) |
| Manufacturing | Bills of materials of the kit kind used when exploding repair parts; Manufacturing Orders raised for repair components | [../manufacturing/](../manufacturing/) |
| Products and catalog | Products, service tracking, product tracking mode, lot sequences, the catalog data contract | [../products-and-catalog/](../products-and-catalog/) |
| Units of measure and packaging | The unit of a repair quantity and of each part, and the conversion between a part's unit and the product's reference unit | [../units-of-measure-and-packaging/](../units-of-measure-and-packaging/) |
| Pricing and pricelists | The unit price placed on a repair Sales Order Line when the repair is not under warranty | [../pricing-and-pricelists/](../pricing-and-pricelists/) |
| General ledger | Journal Entries and Journal Items; this domain only suppresses one item in one case | [../general-ledger/](../general-ledger/) |
| Analytic accounting | The analytic distribution a repair reaches through its Sales Order Lines | [../analytic-accounting/](../analytic-accounting/) |
| Taxes | The taxes applied to the Sales Order Lines created from added parts | [../taxes/](../taxes/) |
| Payments and bank reconciliation | The reconciliation of the receivable produced by a repair invoice | [../payments-and-bank-reconciliation/](../payments-and-bank-reconciliation/) |
| Messaging and activities | The discussion thread on Repair Orders, Equipment, Maintenance Teams and Maintenance Requests; the maintenance activity type; the message subtypes; the incoming-mail alias on a Maintenance Team | [../messaging-and-activities/](../messaging-and-activities/) |
| Human resources core | Employees, departments and department managers used to assign equipment; the departure procedure | [../human-resources-core/](../human-resources-core/) |
| Identity and access | Users, user companies, groups and record rules | [../identity-and-access/](../identity-and-access/) |
| Contacts and organizations | Partners used as repair customers and as equipment vendors | [../contacts-and-organizations/](../contacts-and-organizations/) |
| Point of sale | The suppression of duplicate delivery moves when a repair-backed Sales Order Line is settled at a point-of-sale terminal, and the quantity that line contributes | [../point-of-sale/](../point-of-sale/) |
| Platform foundation | Companies, configuration settings, properties and property definitions, numbering sequences and the record duplication rules | [../platform-foundation/](../platform-foundation/) |
| Fiscal localizations | The commercial document layout that stamps a printing date on the printed Repair Order | [../fiscal-localizations/](../fiscal-localizations/) |

## Domains that depend on this one

| Domain | What it takes from here |
|---|---|
| Inventory operations | The repair operation-type code and its dashboard counters; the repair reference shown on a Stock Move; the traceability node a repair contributes |
| Inventory valuation and costing | The move set a completed repair validates, which it values |
| Sales | The repair count badge on a Sales Order, the delivered quantity of a repair-backed line, and the cancellation of repairs when an order is cancelled |
| Purchasing | The repair count badge on a Purchase Order raised from repair components |
| Manufacturing | The repair count badge on a Manufacturing Order raised from repair components, and the release of a unique serial number when a component is taken out of a repaired product |
| Point of sale | The flag that marks a Sales Order Line as repair-backed, and the settlement quantity of such a line |
| Human resources core | The equipment count badge on an employee and the equipment-freeing option in the departure procedure |
| General ledger | The suppression of the cost-of-goods-sold item on an invoice line whose repair movement already carries a journal entry |

## Files in this folder

| File | Contents |
|---|---|
| [README.md](README.md) | Scope, business scope, capability map, actors, the entities owned and extended, reading order, dependencies, and this file list. |
| [entities.md](entities.md) | Every field of every entity owned here and every field added to entities owned elsewhere, with identity and uniqueness rules, ordering, display names, defaults, computed rules, duplication behaviour, on-change behaviour, record lifecycles and multi-company behaviour. |
| [state-machines.md](state-machines.md) | Every state field of the domain: states with their stored values, labels and meanings; transition tables with triggers, guards and side effects; the exact refusal message of each guard; a diagram per machine. |
| [workflows.md](workflows.md) | Every end-to-end procedure with actors, preconditions, numbered steps, branches, records written and postconditions. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with guards, validations, permissions, consistency rules and exact messages, plus the mapping of the former rule identifiers. |
| [calculations.md](calculations.md) | Every formula and algorithm with inputs, outputs, precision, order of operations and worked examples. |
| [accounting-effects.md](accounting-effects.md) | The journal entries caused by repair movements and by the invoicing of repairs, with debit and credit tables and account selection, and the reasoned statement that maintenance produces none. |
| [configuration.md](configuration.md) | Capability packages, master-data prerequisites, every setting, default, shipped record, sequence, operation type, property definition, access group, access matrix, record rule, menu and scheduled execution. |
| [interfaces.md](interfaces.md) | Named operations, screens described as views with their fields and buttons, the deep-link path pattern of every screen, printed documents, the incoming-mail channel, notifications, counters and reporting surfaces. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given/When/Then scenarios with concrete values, covering every rule, every calculation, every workflow and every state transition. |
| [glossary.md](glossary.md) | The vocabulary of the domain, in full words. |
