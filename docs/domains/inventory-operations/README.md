# Inventory Operations

## Scope

This domain specifies the physical goods engine of the system: how warehouses and their locations are structured, how operation types and the routes and rules attached to them are generated from a warehouse step configuration, how transfers are created, confirmed, reserved, validated, split into backorders, cancelled and returned, how every stock movement is recorded as a Stock Move with its detailed Stock Move Lines, how on-hand quantities are stored as Stock Quantity records and mutated, how reservation picks quantities out of those records under a removal strategy, how put-away redirects incoming goods to a sublocation subject to storage-capacity checks, how lots and serial numbers are created and traced, how packages and containers are built and moved, how goods are scrapped, and how physical inventory counts are requested, entered, reconciled and applied.

Everything in this folder is derived from the behavior of the following capability packages: the core Inventory package (all of its models, wizards, reports, shipped data and security files), the Batch and Wave Transfers package, the Inventory Text Message Confirmation package, the Dispatch Management package that links batches to vehicles and docks, and the Inventory–Maintenance bridge package that links equipment to locations and serial numbers.

The domain does **not** cover: valuation, costing and the journal entries produced by stock movements (see `../inventory-valuation-and-costing/`); reordering rules, the procurement scheduler, lead-time arithmetic, the forecast report and the replenish wizard (see `../replenishment-and-procurement/`); products, product categories, tracking configuration, barcodes and barcode nomenclatures (see `../products-and-catalog/`); units of measure, their conversion arithmetic and packaging units (see `../units-of-measure-and-packaging/`); delivery methods, carriers and shipping labels (see `../delivery-and-shipping/`); manufacturing consumption and production moves (see `../manufacturing/`); repair orders and maintenance requests themselves (see `../repair-and-maintenance/`).

Where this domain has to name a mechanism owned by another domain — a unit conversion, a rounding precision, a route matching rule used by procurement — it states exactly which inputs it passes and which result it expects, and links to the owning domain.

## Capabilities covered

| Capability | Where specified |
|---|---|
| Warehouses: view location, stock location, input, quality control, output and packing locations, short name, addresses, resupply links | `entities.md`, `workflows.md`, `configuration.md` |
| Warehouse step configuration: one-, two- and three-step receipts; one-, two- and three-step deliveries; the exact operation types, locations, routes and rules created for each; what changes when the configuration is changed | `workflows.md`, `calculations.md`, `configuration.md` |
| Locations: the seven usages, the hierarchy and materialised path, barcodes, removal strategy, storage category, cyclic counting frequency, next count date, weights, emptiness, archival cascade | `entities.md`, `business-rules.md`, `calculations.md` |
| Routes and Stock Rules: selectability flags, rule actions (pull, push, pull and push), supply methods, lead time, cancel and carrier propagation, automatic-move mode, push applicability domain | `entities.md`, `workflows.md` |
| Operation types: code, default locations, reservation methods, backorder policy, lot settings, automatic printing options, package-type prompting, dashboard counters and the dashboard graph | `entities.md`, `calculations.md`, `interfaces.md` |
| Transfers: reference numbering, contact, scheduled date and deadline, shipping policy, state computation from the moves, availability text, lock flag, signature, printing flag, properties, next transfers, returns | `entities.md`, `state-machines.md`, `calculations.md` |
| Transfer operations: confirmation, availability check, unreservation, validation, the backorder decision, split, cancellation, put in pack, add entire packages, scrap from a transfer, return and exchange | `workflows.md`, `business-rules.md` |
| Stock Moves: every field, the state machine, the merge key and merge algorithm including negative-quantity absorption, chaining upstream and downstream, deadline propagation, delay alerts, splitting, the extra-move and backorder creation, push application, cancellation propagation | `entities.md`, `state-machines.md`, `calculations.md`, `workflows.md` |
| Reservation: the full assignment algorithm, chained-move distribution, bypass rules, partial availability, serial-number expansion, forced quantities and the freeing of other reservations | `calculations.md`, `workflows.md` |
| Removal strategies: first in first out, last in first out, closest location, least packages (with its search), and the ordering each imposes | `calculations.md` |
| Stock Move Lines: quantity in two units, picked flag, source and destination package, lot name versus lot record, quantity synchronisation with Stock Quantity records, the done algorithm, aggregation for printed documents | `entities.md`, `calculations.md` |
| Stock Quantity records: available quantity, reservation counter, incoming date, gathering with strict and loose matching, negative quantities, merging duplicates, cleaning stale reservations, deletion of empty records | `entities.md`, `calculations.md`, `business-rules.md` |
| Inventory counting: counted quantity, difference, scheduled count date, conflict detection, request-a-count, apply, reset, relocate, revert, the annual and cyclic count date arithmetic | `workflows.md`, `calculations.md`, `state-machines.md` |
| Lots and serial numbers: creation rules, name generation from a template, uniqueness across companies, the single-location field, traceability upstream and downstream, delivery discovery | `entities.md`, `calculations.md`, `business-rules.md` |
| Packages and containers: naming sequences, nesting, destination containers, entire-package detection, put in pack, unpack, package history, weights | `entities.md`, `workflows.md`, `calculations.md` |
| Put-away rules and storage categories: rule specificity ordering, sublocation modes, capacity by product and by package type, maximum weight, mixed-product policy | `calculations.md`, `entities.md` |
| Scrap: creation from a transfer or standalone, quantity availability check, the scrap move, replenishment trigger, reason tags | `entities.md`, `workflows.md` |
| Batch and wave transfers: composition rules, automatic batching and waving criteria, limits, merge, validation of a whole batch, detachment of empty transfers | `entities.md`, `workflows.md`, `state-machines.md` |
| Reception report: the allocation matching algorithm, assignment and unassignment of incoming moves to outgoing moves, labels | `calculations.md`, `interfaces.md` |
| Traceability report: upstream and downstream line discovery, the printable tree | `calculations.md`, `interfaces.md` |
| Text-message confirmation on delivery and the one-time warning wizard | `workflows.md`, `interfaces.md` |
| Dispatch management: vehicle, vehicle category capacity, dock, driver, load percentages, ordering by postal code | `entities.md`, `calculations.md` |
| Settings, security groups, access rights, record rules, sequences, shipped records and the scheduled job | `configuration.md` |

## Entities

| Entity | Transport name | Storage name | Purpose |
|---|---|---|---|
| Warehouse | `stock.warehouse` | `stock_warehouse` | One physical site; owns its locations, operation types, routes and rules |
| Location | `stock.location` | `stock_location` | One node of the location tree; holds stock when its usage is internal or transit |
| Route | `stock.route` | `stock_route` | Named ordered collection of Stock Rules selectable on products, categories, warehouses or package types |
| Stock Rule | `stock.rule` | `stock_rule` | One pull or push step: from a source location to a destination location through an operation type |
| Operation Type | `stock.picking.type` | `stock_picking_type` | Template and counter holder for transfers of one kind in one warehouse |
| Transfer | `stock.picking` | `stock_picking` | One document grouping moves that travel together between two locations |
| Stock Move | `stock.move` | `stock_move` | One product's demand to travel from a source to a destination location |
| Stock Move Line | `stock.move.line` | `stock_move_line` | One concrete reservation or execution detail of a Stock Move: quantity, lot, source and destination package, exact locations |
| Stock Quantity | `stock.quant` | `stock_quant` | On-hand quantity of one product in one location with one lot, package and owner |
| Lot or Serial Number | `stock.lot` | `stock_lot` | One identified batch or one identified unit of a tracked product |
| Package | `stock.package` | `stock_package` | One physical container holding quantities and/or other packages |
| Package Type | `stock.package.type` | `stock_package_type` | Reusable container specification: dimensions, weights, sequence, routes |
| Package History | `stock.package.history` | `stock_package_history` | Immutable trace of one package as it was at the moment a transfer was validated |
| Put-away Rule | `stock.putaway.rule` | `stock_putaway_rule` | Redirection of arriving goods from a parent location to a sublocation |
| Removal Strategy | `product.removal` | `product_removal` | Named method used to order Stock Quantity records when taking goods out |
| Storage Category | `stock.storage.category` | `stock_storage_category` | Capacity and mixing policy attachable to locations |
| Storage Category Capacity | `stock.storage.category.capacity` | `stock_storage_category_capacity` | One capacity limit of a storage category, by product or by package type |
| Scrap | `stock.scrap` | `stock_scrap` | Removal of damaged goods to a scrap location |
| Scrap Reason Tag | `stock.scrap.reason.tag` | `stock_scrap_reason_tag` | Free tag qualifying why goods were scrapped |
| Document Reference | `stock.reference` | `stock_reference` | Shared key linking moves that belong to the same originating need |
| Batch Transfer | `stock.picking.batch` | `stock_picking_batch` | Group of transfers processed together; a wave when the flag is set |
| Traceability Report | `stock.traceability.report` | `stock_traceability_report` | Transient holder of one user's traceability tree |
| Stock Quantity Report | `report.stock.quantity` | database view `report_stock_quantity` | Read-only daily series of forecast, incoming and outgoing quantities |

Transient entities of this domain (all specified in `entities.md`): Backorder Confirmation and its line, Inventory Adjustment Reference, Inventory Conflict, Inventory Warning, Label Type Chooser, Lot Label Layout, Package Destination Chooser, Put in Pack, Quantity Relocation, Quantity History, Request a Count, Routes Report, Insufficient Quantity Warning (abstract) and its scrap variant, Return Transfer and its line, Batch Assignment, Wave Assignment.

Entities of other domains that carry fields specified here: Company (`res.company`), Contact (`res.partner`), Product (`product.product`), Product Template (`product.template`), Product Category (`product.category`), Unit of Measure (`uom.uom`), Barcode Rule (`barcode.rule`), Settings (`res.config.settings`), Vehicle Model Category (`fleet.vehicle.model.category`), Equipment (`maintenance.equipment`).

## Reading order

1. `glossary.md` — the vocabulary (transfer, move, move line, quantity record, gather, reserve, pick, backorder, put-away, entire package, wave).
2. `entities.md` — the data model, entity by entity, with every field.
3. `state-machines.md` — the four state fields of the domain and how they interact.
4. `calculations.md` — read in order; the reservation algorithm depends on the gather algorithm, the completion algorithm depends on both.
5. `workflows.md` — the end-to-end sequences that call those algorithms.
6. `business-rules.md` — validations, messages, locking and permission checks.
7. `configuration.md` and `interfaces.md`.
8. `accounting-effects.md` — short; this domain produces no journal entries by itself.
9. `acceptance-criteria.md` — to validate an implementation.

## Dependencies on other domains

| Domain | Dependency |
|---|---|
| `../products-and-catalog/` | Product, whether it is storable, its tracking mode, its default unit of measure, its category, its barcode, its weight and volume, its lot-number sequence, its lot property definitions, its routes |
| `../units-of-measure-and-packaging/` | The conversion of a quantity between two units, the rounding of a quantity to a unit, the comparison of two quantities at a unit's precision, the packaging units used on moves, package types as they appear on packaging units |
| `../inventory-valuation-and-costing/` | Consumes every validated Stock Move; supplies the unit price field carried on the move; produces the journal entries this domain does not produce |
| `../replenishment-and-procurement/` | Owns reordering rules, the scheduler entry point, lead times, the forecast report and the rule-matching algorithm used when a move must be supplied by another rule; this domain supplies the rules, routes and moves it operates on |
| `../purchasing/`, `../sales/`, `../manufacturing/` | Create moves and transfers through the rule engine and read back received and delivered quantities |
| `../delivery-and-shipping/` | Adds carrier fields to transfers and packages and consumes the shipping weight computed here |
| `../repair-and-maintenance/` | Equipment records point at a location and match a serial number by name |
| `../fleet/` | Vehicles and vehicle model categories used by dispatch management on batch transfers |
| `../messaging-and-activities/` | The discussion thread, the activity scheduling and the text-message sending used by transfers, lots and batches |
| `../automation-and-integration/` | Sequences, defaults per company, system parameters and scheduled jobs used by this domain |

## Behavior not present in the source set

- The domain defines no accounting entries of its own. `accounting-effects.md` explains what it hands over and to whom.
- Expiry dates on lots and the first-expired-first-out removal ordering are referenced by a help text and by an optional context switch (`with_expiration`) but the dates themselves and the strategy record are supplied by a companion package; only the hooks are specified here and marked as such.
- Barcode-driven screens are described only through the data contract they use (barcode fields, barcode rule types, aggregate barcode generation, reusable versus disposable package behavior); the interactive screens themselves belong to a companion package.

## How to read the algorithms

Five algorithms are the heart of the domain and everything else is scaffolding around them. Read them in this order:

1. **Gathering** (`calculations.md`, section 3) — turns "I need product X at location L" into an ordered list of Stock Quantity records. Everything that takes goods out of anywhere goes through it.
2. **The reservation-quantity computation** (section 3.3) — decides how much of that list may be taken, including the clamping, the unit re-expression and the negative-pocket absorption.
3. **The reservation algorithm** (section 5) — turns the result into Stock Move Lines and reserved counters, with three branches: bypassing sources, unchained moves, chained moves.
4. **The completion algorithm** (section 16) and **the line completion** (section 17) — actually move the goods.
5. **The validation algorithm** (section 20) — the sequence a person triggers, which wraps the completion in the sanity check, the backorder decision and the follow-up actions.

Three more are needed to understand where goods end up rather than where they come from: **put-away selection** (section 9), the **whole-container** passes (section 10) and **move merging** (section 13).

## What an implementation must get exactly right

In order of how much damage an error does:

| Rank | Thing | Why |
|---|---|---|
| 1 | The reserved counter invariant (`business-rules.md`, invariant 14.2) | Every promise the system makes to a document rests on it. |
| 2 | The gathering order per removal strategy | It decides which physical goods leave, and therefore the cost the valuation domain computes. |
| 3 | The completion order (`calculations.md`, section 16) | Backorders, pushes and re-reservations all depend on happening at the right step. |
| 4 | The status derivations | Every screen, every filter and every counter is built on them. |
| 5 | The rounding table (`calculations.md`, section 35) | The difference between a backorder and no backorder is often one comparison at the wrong precision. |
| 6 | The merge key | Getting it wrong either fragments documents or silently fuses different costs. |
| 7 | The put-away specificity sort | It decides which shelf, and a wrong shelf is a lost item. |

## Conventions in this folder

- Entity names are written in full and in title case. The transport name and the storage name are given in code font at the first mention in each file.
- Quantities always name the unit they are expressed in: the *product unit*, the *line unit* or the *packaging unit*.
- Algorithms are numbered steps with explicit preconditions and failure conditions. Every failure names the exact message.
- Formulas are plain mathematics in fenced blocks labelled `formula`, with the rounding rule stated next to them.
- Reproduced identifiers — storage names, transport names, selection values, route paths — are in code font and are the only abbreviations used; each file that contains one glosses it at the top.
