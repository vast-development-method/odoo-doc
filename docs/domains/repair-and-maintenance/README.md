# Repair and Maintenance

## Scope

This domain specifies two related but structurally independent capabilities of the system.

The first is **repair**: the handling of a physical item that a customer, or the company itself, wants restored to working order. A Repair Order names the product to repair (optionally a specific lot or serial number), the parts that will be consumed into it, the parts that will be taken out of it and thrown away, and the parts that will be taken out of it and returned to stock. It moves through a five-state lifecycle, it reserves its parts from stock exactly as an ordinary warehouse document does, it can be priced and billed through a quotation and a sales order, it can be declared "under warranty" so that nothing is charged, it can be created automatically from a customer return, from a sales order line whose product is configured as a repair service, or by hand, and when it is completed it produces a precise set of inventory movements whose valuation and accounting consequences are owned by the inventory-valuation domain.

The second is **maintenance**: the register of Equipment owned or used by the company, the Maintenance Teams responsible for it, and the Maintenance Requests raised against it. A request is either *corrective* — something broke, fix it — or *preventive* — the calendar says it is time to service this machine. Requests travel across a configurable pipeline of stages, one or more of which are marked as closing stages. Preventive requests may be *recurrent*: when a recurrent preventive request reaches a closing stage the system immediately generates its successor, dated by adding the configured repeat interval to the current scheduled date. Equipment carries effectiveness measurements — mean time between failures, mean time to repair, latest failure date and estimated next failure — computed from the closed corrective requests raised against it.

The two capabilities share no records. A Repair Order does not reference Equipment, and a Maintenance Request does not create inventory movements. They are documented together because they answer the same operational question — *this thing is broken, what do we do about it* — and because the bridge packages that extend them (to manufacturing, purchasing, the point of sale, inventory, people management and subcontracting) sit between the same set of neighbouring domains.

Everything in this folder is derived from the behaviour of the following capability packages: the Repair package; the Maintenance package; the Repair–Manufacturing bridge; the Repair–Purchasing bridge; the Repair–Point of Sale bridge; the Inventory–Maintenance bridge; the People–Maintenance bridge; and the Subcontracting–Repair bridge.

The domain does **not** cover: how Stock Moves reserve, execute and merge (see `../inventory-operations/`); how a validated Stock Move is valued and which journal entries it creates (see `../inventory-valuation-and-costing/`); how a Sales Order is confirmed, invoiced and paid (see `../sales/`); how a Purchase Order is raised and received (see `../purchasing/`); how a bill of materials is exploded during manufacturing (see `../manufacturing/`); how activities, followers, message subtypes and incoming-mail aliases work (see `../messaging-and-activities/`); how employees and departments are modelled (see `../human-resources-core/`).

Where this domain has to name a mechanism owned by another domain — a reservation pass, a unit conversion, a costing method — it states exactly which inputs it passes and which result it expects, and links to the owning domain rather than restating it.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Repair Orders: reference numbering per operation type, customer, product to repair, lot or serial, quantity and unit, scheduling, priority, tags, responsible user, internal notes, properties | `entities.md`, `configuration.md` |
| The five repair locations (component source, product source, product destination, added-parts destination, removed-parts destination, recycled-parts destination) and how each is defaulted from the operation type | `entities.md`, `calculations.md` |
| Repair parts: the three line kinds (add, remove, recycle), the source and destination location each kind implies, quantity and picked flag, catalog-driven entry | `entities.md`, `calculations.md`, `workflows.md` |
| The Repair Order state machine: new, confirmed, under repair, repaired, cancelled — with every transition, guard and side effect, including reset to draft | `state-machines.md` |
| Confirmation: the insufficient-quantity check on the product to repair, the warning wizard, procurement method adjustment, move confirmation, scheduler trigger | `workflows.md`, `business-rules.md` |
| Completion: cancellation of zero-quantity parts, the serial-number requirement, owner detection, the generated product move with its consumption links, backorder suppression, delivered-quantity feedback to the sales order line | `workflows.md`, `calculations.md` |
| Parts availability: the three availability states, the forecast comparison, the readiness and lateness booleans, the operation-type dashboard counters | `calculations.md` |
| The inventory movements a repair produces, enumerated move by move with source and destination location, quantity and unit | `workflows.md`, `accounting-effects.md` |
| The journal items a repair produces, through the valuation of its moves and through the invoice raised on its sales order, with account selection rules and amount formulas | `accounting-effects.md` |
| Warranty: what the flag does to prices at line creation, what it does when toggled afterwards, and what is left uncharged | `workflows.md`, `calculations.md`, `business-rules.md` |
| The sales order link: quotation creation from a repair, line creation and update from parts, line cancellation, delivered quantity derivation, repair creation from a confirmed sales order line, cancellation propagation in both directions | `workflows.md`, `business-rules.md` |
| The return link: creating a repair from a validated customer return, restriction of the product and lot choice to what came back, quantity derivation, the warehouse-mismatch warning | `workflows.md`, `business-rules.md` |
| Repair operation types: the repair code, the four extra default locations, the four dashboard counters, the property definition, the sequence and barcode created per warehouse | `configuration.md`, `entities.md` |
| The make-to-order rule created per warehouse for repair components | `configuration.md` |
| Kit explosion of repair parts when a phantom bill of materials exists | `workflows.md`, `calculations.md` |
| Purchase orders and manufacturing orders raised from repair components, and the counters that surface them | `interfaces.md`, `workflows.md` |
| Lot and serial traceability through a repair: repaired counts, in-repair counts, the parts used, the consumption links that make the traceability tree | `calculations.md`, `interfaces.md` |
| Equipment: identity, category, vendor and vendor reference, model, serial number, assignment to an employee or a department or neither, owner, technician, team, cost, warranty expiry, scrap date, location, properties | `entities.md` |
| Equipment effectiveness: mean time between failures, mean time to repair, latest failure date, estimated next failure, expected mean time between failures, with exact formulas and worked examples | `calculations.md` |
| Equipment categories: responsible technician, colour, folding rule, equipment and request counters, property definitions, deletion guard | `entities.md`, `business-rules.md` |
| Maintenance Teams: members, company, the request dashboard counters, the incoming-mail alias that creates requests | `entities.md`, `configuration.md`, `interfaces.md` |
| Maintenance Stages: sequence, folding, the closing flag, the shipped stage set | `entities.md`, `configuration.md` |
| Maintenance Requests: subject, description, kind, request date, schedule window and duration, technician, team, stage, kanban state, priority, close date, archive flag, instructions in three media, employee link | `entities.md` |
| The maintenance request pipeline as a state machine over stages, with the closing-stage side effects | `state-machines.md` |
| The recurring generation algorithm for preventive work, its date arithmetic, its termination rule and its copy semantics, as numbered steps | `calculations.md`, `workflows.md` |
| Equipment downtime computed across stoppages | `calculations.md`, `acceptance-criteria.md` |
| The activity that is scheduled, rescheduled, closed and removed as a request changes | `workflows.md`, `interfaces.md` |
| Followers and tracked fields on equipment and requests; the assignment notification subtype | `interfaces.md`, `configuration.md` |
| Departure handling: freeing the equipment assigned to a leaving employee | `workflows.md` |
| Equipment and stock: the location field, the serial-number match and the jump to the matching lot record | `workflows.md`, `interfaces.md` |
| Settings, security groups, the complete access rights matrix, record rules, sequences and shipped records | `configuration.md` |
| Every validation, invariant and error message of both capabilities, with exact message text | `business-rules.md` |
| Numbered acceptance scenarios with concrete numbers | `acceptance-criteria.md` |

## Entities

### Repair side

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Repair Order | `repair.order` | `repair_order` | One item to restore: what it is, which parts go in and out, when, for whom, and at what price |
| Repair Tag | `repair.tags` | `repair_tags` | Free classification label attachable to Repair Orders |
| Insufficient Repair Quantity Warning | `stock.warn.insufficient.qty.repair` | `stock_warn_insufficient_qty_repair` | Transient confirmation shown when the product to repair is not on hand at its source location |
| Stock Move (extended) | `stock.move` | `stock_move` | Gains a Repair Order link and a part kind; one Stock Move is one repair part line |
| Operation Type (extended) | `stock.picking.type` | `stock_picking_type` | Gains the repair code, four default repair locations, four repair counters and a repair property definition |
| Warehouse (extended) | `stock.warehouse` | `stock_warehouse` | Gains a repair operation type and a repair make-to-order rule |
| Transfer (extended) | `stock.picking` | `stock_picking` | Gains the list of repairs raised from it and their count |
| Lot or Serial Number (extended) | `stock.lot` | `stock_lot` | Gains repaired count, in-repair count, and the repairs in which the lot was used as a part |
| Sales Order (extended) | `sale.order` | `sale_order` | Gains the repairs it generated and their count |
| Sales Order Line (extended) | `sale.order.line` | `sale_order_line` | Gains repair-aware delivered-quantity derivation and repair creation and cancellation on quantity change |
| Product Template (extended) | `product.template` | `product_template` | Gains the repair value of the service tracking selector |

### Maintenance side

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Maintained Item | `maintenance.mixin` | *(abstract, no table)* | Shared behaviour of anything that can be maintained: company, effective date, team, technician, request counters and the four effectiveness measurements |
| Equipment | `maintenance.equipment` | `maintenance_equipment` | One machine, tool, vehicle or device that is registered, assigned and maintained |
| Equipment Category | `maintenance.equipment.category` | `maintenance_equipment_category` | Grouping of equipment with a default responsible technician and a property definition |
| Maintenance Team | `maintenance.team` | `maintenance_team` | Group of users that receives and works maintenance requests; owns an incoming-mail alias |
| Maintenance Stage | `maintenance.stage` | `maintenance_stage` | One column of the maintenance pipeline; may be marked as a closing stage |
| Maintenance Request | `maintenance.request` | `maintenance_request` | One piece of work to be done on a piece of equipment, corrective or preventive, possibly recurrent |
| Location (extended) | `stock.location` | `stock_location` | Gains an equipment counter and the jump to the equipment stored there |
| Employee (extended) | `hr.employee` | `hr_employee` | Gains the equipment assigned to the person and its count |
| Public Employee (extended) | `hr.employee.public` | `hr_employee_public` | Gains the equipment count, readable without private-data rights |
| Departure Wizard (extended) | `hr.departure.wizard` | *(transient)* | Gains the option to free the departing person's equipment |

## Reading order

1. `README.md` — this file: scope, capability map, entity list, dependencies.
2. `glossary.md` — every term used below, defined in full. Read it first if any term is unfamiliar.
3. `entities.md` — the complete field tables, relations, defaults, computed rules, ordering, display names, archival and multi-company behaviour of each entity.
4. `state-machines.md` — the Repair Order lifecycle and the Maintenance Request pipeline, with transition tables, guards, side effects and diagrams.
5. `workflows.md` — the end-to-end operational sequences: raising a repair by hand, from a return, from a sales order; confirming, starting, completing, cancelling and resetting one; quoting and invoicing one; raising, working, closing and repeating a maintenance request; assigning and freeing equipment.
6. `calculations.md` — every formula and algorithm: availability, quantity derivation, location mapping, warranty pricing, the recurring generation date arithmetic, the effectiveness measurements, downtime, duration, kit explosion, with worked numeric examples.
7. `business-rules.md` — every validation, constraint, invariant, permission check and edge case, with exact error messages.
8. `accounting-effects.md` — the journal entries the domain causes, directly and indirectly, with account selection rules and amount formulas.
9. `configuration.md` — settings, sequences and numbering formats, shipped records, security groups, the access rights matrix, record rules and scheduled jobs.
10. `interfaces.md` — menus, window actions, views, buttons, filters, groupings, named remote operations, printable documents, notifications and integration points.
11. `acceptance-criteria.md` — numbered Given/When/Then scenarios with concrete numbers, to be used as the conformance suite.

## Dependencies on other domains

| Depends on | For what | Link |
|---|---|---|
| Inventory operations | Stock Moves, Stock Move Lines, reservation, the done pass, locations, operation types, warehouses, lots and serial numbers, the transfer that returns goods from a customer | `../inventory-operations/` |
| Inventory valuation and costing | The valuation layers and journal entries produced by the moves a repair validates, and the cost of the product to repair | `../inventory-valuation-and-costing/` |
| Sales | Sales Orders, Sales Order Lines, their confirmation, invoicing and cancellation; the service-tracking selector; delivered quantity | `../sales/` |
| Purchasing | Purchase Orders raised for repair components through the make-to-order rule | `../purchasing/` |
| Manufacturing | Bills of materials of the kit kind used when exploding repair parts; manufacturing orders raised for repair components | `../manufacturing/` |
| Products and catalog | Products, service tracking, product tracking mode, lot sequences, the catalog data contract | `../products-and-catalog/` |
| Units of measure and packaging | The unit of a repair quantity and of each part, and the conversion between a part's unit and the product's reference unit | `../units-of-measure-and-packaging/` |
| Pricing and pricelists | The unit price placed on a repair sales order line when the repair is not under warranty | `../pricing-and-pricelists/` |
| Messaging and activities | The discussion thread on Repair Orders, Equipment and Maintenance Requests; the maintenance activity type; the message subtypes; the incoming-mail alias on a Maintenance Team | `../messaging-and-activities/` |
| Human resources core | Employees, departments and department managers used to assign equipment; the departure wizard | `../human-resources-core/` |
| Identity and access | Users, user companies, groups and record rules | `../identity-and-access/` |
| Contacts and organizations | Partners used as repair customers and as equipment vendors | `../contacts-and-organizations/` |
| Point of sale | The suppression of duplicate delivery moves when a repair-backed sales order line is settled at a point-of-sale terminal | `../point-of-sale/` |

## Domains that depend on this one

| Domain | What it takes from here |
|---|---|
| Inventory operations | The repair operation-type code and its dashboard counters; the repair reference shown on a Stock Move; the traceability node a repair contributes |
| Inventory valuation and costing | The move set a completed repair validates, which it values |
| Sales | The repair count badge on a Sales Order, the delivered quantity of a repair-backed line, and the cancellation of repairs when an order is cancelled |
| Purchasing | The repair count badge on a Purchase Order raised from repair components |
| Manufacturing | The repair count badge on a Manufacturing Order raised from repair components |
| Human resources core | The equipment count badge on an employee and the equipment-freeing option in the departure wizard |
