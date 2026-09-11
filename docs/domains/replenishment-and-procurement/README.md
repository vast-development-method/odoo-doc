# Replenishment and Procurement

The Replenishment and Procurement domain decides *how* a need for a product at a location is satisfied. A need (called a procurement request) arises when a sales order line is confirmed, when a stock move that must be supplied by another operation is confirmed, when a reordering rule detects that the forecasted stock will fall below its minimum, when a user launches a manual replenishment, or when a subcontracted or manufactured product requires components. The domain owns the configuration objects that describe the supply chain (routes and stock rules), the algorithm that selects the rule applicable to a need (the "get rule" search up the location hierarchy with route precedence), the "run" algorithm that converts needs into chained stock moves, purchase orders or manufacturing orders (with merging into existing draft documents), the reordering rules and the daily scheduler that evaluates them, the lead time and date planning logic that positions every document backwards from the date the goods are needed, the forecast report that explains where every unit of forecasted stock comes from and goes to, and the drop shipping and inter-warehouse resupply flows.

## Business scope

| In scope | Out of scope (owned elsewhere) |
|---|---|
| Routes and stock rules (pull, push, pull and push, buy, manufacture) and their selection | Physical execution of transfers, reservation, move lines, lots, packages: see `../inventory-operations/` |
| The procurement request structure and the run algorithm | Purchase order lifecycle, approval, billing, vendor pricelists: see `../purchasing/` and `../pricing-and-pricelists/` |
| Merging of procurements into draft purchase orders, vendor selection for a need, purchase date computation | Sales order lifecycle and invoicing: see `../sales/` |
| Reordering rules, the replenishment report, the replenishment information wizard, the product replenish wizard | Manufacturing order lifecycle, bills of materials, work orders: see `../manufacturing/` |
| The scheduler (scheduled action) and the exception activities it creates | Inventory valuation and cost of goods sold: see `../inventory-valuation-and-costing/` |
| Lead time propagation, deadline versus scheduled date, delay alerts, late indicators as they relate to chained documents | Warehouse and location master data as such (creation of the warehouse structure is described here only for the routes and rules it generates) |
| Drop shipping (route, operation type, purchase order to the customer address, cross-links to accounting) | Subcontracting production itself (only the resupply and drop-ship-to-subcontractor rules are described here) |
| Inter-warehouse resupply routes and the transit location | Vendor delay statistics are described here as a report because they are computed from receipts created by procurement |

## Capabilities delivered

1. Declarative supply chain modelling: a route is an ordered set of stock rules; a rule says "when a product is needed in location D, take it from location S with operation type T" (pull), "when a product arrives in S, send it to D" (push), "buy it" or "manufacture it".
2. Rule selection: for any (product, location, context) the system finds the single applicable rule by walking from the location up to its root and applying a strict precedence of route sources (routes given on the request, routes of the packaging type, routes of the product and its category, routes of the warehouse).
3. Procurement execution: the run operation turns a list of procurement requests into stock moves (pull rules), draft purchase order lines (buy rules) or manufacturing orders (manufacture rules), chaining them through destination move links, and merging into existing draft documents when the merge keys match.
4. Push propagation: when a move is completed, the push rule of its destination location creates the next move (or rewrites the destination when the rule is transparent).
5. Reordering rules with minimum, maximum, replenishment multiple, automatic or manual trigger, snoozing, per-rule preferred route, vendor price or bill of materials, and a computed deadline date.
6. Just-in-time forecasting: the quantity to order is computed from the forecasted quantity at the lead horizon date (today plus the cumulative lead time of the rules, plus the company replenishment horizon), so that orders are created neither too early nor too late.
7. The replenishment report: automatic generation of manual reordering rules for every product whose forecast is negative at a replenishment location, and their removal once satisfied.
8. The scheduler: a daily scheduled action that runs every automatic reordering rule, then reserves every waiting move whose reservation date has been reached, then merges duplicated stock quantity records; failures are converted into warning activities on the product.
9. Lead time and date planning: scheduled dates are computed backwards from the need date using rule lead times, vendor lead time, days to purchase, manufacturing lead time and days to prepare the manufacturing order; deadlines propagate along chains and delay alerts flag late upstream documents.
10. Drop shipping: a global route whose buy rule ships from the vendor directly to the customer through a dedicated "Dropship" operation type, with the purchase order carrying the customer as delivery address, and a variant for shipping components directly to a subcontractor.
11. Inter-warehouse resupply: automatically generated routes "X: Supply Product from Y" that pull from the supplying warehouse through the inter-warehouse transit location into the supplied warehouse.
12. Forecast reporting: the forecasted stock report reconciles every outgoing move with reserved stock, free stock, in-transit stock and incoming moves, in priority and date order, and lets a user reserve or unreserve the linked upstream moves for a specific outgoing document.

## Actors

| Actor | Role in this domain |
|---|---|
| Inventory user (group "Inventory / User") | Reads routes and rules; reads reordering rules; uses the replenishment report, snoozes manual rules, opens the replenishment information wizard and the forecast report; launches manual replenishment from the product form. |
| Inventory administrator (group "Inventory / Administrator") | Creates and edits routes, stock rules, reordering rules and the settings of this domain (replenishment horizon, days to purchase, sales security lead time, multi-step routes, drop shipping). |
| Purchase user and purchase manager | Read reordering rules; act on the request for quotation documents created by buy rules (owned by `../purchasing/`). |
| Salesperson | Confirms sales orders that create procurements; selects a route on a sales order line when routes are selectable on sales order lines. |
| Product responsible (the user set as responsible on the product) | Receives the warning activity created when a procurement launched by a reordering rule cannot be fulfilled. |
| Scheduler (automated) | Runs the scheduled action "Procurement: run scheduler" once a day as the superuser. |
| System (automated) | Creates warehouse routes and rules when a warehouse is created or reconfigured; applies push rules when moves are completed; triggers automatic reordering rules when a transfer is confirmed. |

## Entities owned by this domain

| Canonical name | Identifier | Kind | Purpose |
|---|---|---|---|
| Route | `route` | persistent | An ordered, named set of stock rules that can be selected on products, product categories, warehouses, package types or sales order lines. |
| Stock Rule | `stock_rule` | persistent | One step of a route: pull, push, pull and push, buy or manufacture, with source and destination locations, operation type, supply method and lead time. |
| Reordering Rule | `reordering_rule` | persistent | Minimum and maximum stock levels for a product at a location, with the computed quantity to order and deadline. |
| Replenishment Information | `replenishment_information` | transient | Wizard that explains the lead times, the forecasted date and the demand graph of one reordering rule and lets the user pick an alternative supplying warehouse or vendor. |
| Replenishment Option | `replenishment_option` | transient | One supplying-warehouse option shown in the Replenishment Information wizard. |
| Reordering Rule Snooze Wizard | `reordering_rule_snooze_wizard` | transient | Hides manual reordering rules from the replenishment report until a date. |
| Stock Rules Report | `stock_rules_report` | transient | Wizard that prints the routes diagram of a product for selected warehouses. |
| Forecasted Stock Report | `forecasted_stock_report` | abstract report | The forecast report per product variant: quantities, lead time, and the reconciliation lines between outgoing and incoming documents. |
| Stock Replenishment Report | `stock_replenishment_report` | abstract report | The same report opened for a product template (all its variants). |
| Vendor Delay Report | `vendor_delay_report` | database view | On-time delivery rate of vendors computed from purchase order lines and their receipts. |
| Procurement request | `procurement_request` | in-memory structure | The need passed to the run operation (product, quantity, unit, location, name, origin, company, values). It is never stored as such; its values are serialized onto the moves it creates. |

## Entities from other domains extended by this domain

| Entity (owner) | What this domain adds |
|---|---|
| Warehouse (`../inventory-operations/`) | Route creation and maintenance (reception route, delivery route, replenish-on-order rule, buy rule, resupply routes), the fields `buy_to_resupply`, `buy_pull`, `resupply_warehouses`, `resupply_routes`, `reception_route`, `delivery_route`, `make_to_order_pull`, and the subcontracting-dropshipping rule. |
| Stock Move (`../inventory-operations/`) | Procurement-related fields: `rule`, `procure_method`, `move_destinations`, `move_origins`, `location_final`, `deadline`, `delay_alert_date`, `orderpoint`, `routes`, `warehouse`, `propagate_cancel`, `procurement_values`, `purchase_line`, `created_purchase_lines`; the confirm-time procurement creation, push application, deadline propagation, cancel propagation and the scheduler trigger. |
| Transfer (`../inventory-operations/`) | `purchase`, `is_dropship`, `days_to_arrive`, `linked_purchase_order_date`; the delay-alert pop-over and the "late" availability state derived from forecast data; acknowledgement of the purchase order when the transfer is completed. |
| Operation Type (`../inventory-operations/`) | The `dropship` operation type code and its default locations. |
| Location (`../inventory-operations/`) | The `replenish_location` flag used by the replenishment report. |
| Purchase Order and Purchase Order Line (`../purchasing/`) | Operation type ("Deliver To"), destination address, receipts, arrival, receipt status, dropship counters, stock references, the moves created at confirmation, the reordering rule and destination moves on lines, cancel propagation, forecasted issue flag, final location. |
| Sales Order and Sales Order Line (`../sales/`) | Routes selectable on lines, the make-to-order flag, the customer lead time, the procurement values built from a line, the dropship counter. |
| Product Variant, Product Template, Product Category (`../products-and-catalog/`) | Routes on products and categories, total routes of a category (inherited from parents), customer lead time, the rule-chain walk and the dates-information helper, quantity in progress from draft purchase orders, the buy-route warning. |
| Vendor Price (`../pricing-and-pricelists/`) | Lead time (delay), last purchase date, the "Set as Supplier" action used from the replenishment information wizard. |
| Company (owned by the Contacts and Organizations domain) | Replenishment horizon, days to purchase, sales security lead time, the internal transit location, the dropship subcontractor operation type. |
| Contact (owned by the Contacts and Organizations domain) | Request-for-quotation grouping options (`group_request_for_quotation`, `grouping_weekday`), the on-time delivery rate, purchase suggestion preferences. |
| Reference between stock documents (`../inventory-operations/`) | The link to the purchase orders created for the reference. |
| Lot or Serial Number (`../inventory-operations/`) | The purchase orders that received the lot; the customer of a drop-shipped lot. |
| Product Replenish Wizard and Product Replenish Mixin (`../inventory-operations/`) | Vendor selection, buy-route date planning, exclusion of the dropship route, notification links to the created purchase order. |
| Purchase Analysis Report (`../purchasing/`) | The warehouse column, the effective date and the effective days to arrival, plus the three widenings of the query that feeds the view. |
| Return Wizard and Return Wizard Line (`../inventory-operations/`) | Recognition of a return to the vendor: the link from the return move back to its purchase order line, the vendor as counterparty of the return, and the effect of the "Update Quantities on Purchase Order" flag on the received quantity. |
| Configuration Settings (`../platform-foundation/`) | The settings listed in `configuration.md`. |
| Journal Entry and Journal Item (`../general-ledger/`) | No new fields; the drop-ship valuation hooks are summarized in `accounting-effects.md`. |

## Cross-domain dependencies (must exist first)

1. `../inventory-operations/`: locations, warehouses, operation types, stock moves, transfers, reservation, stock quantity records, references between stock documents. Every rule of this domain produces or rewrites stock moves.
2. `../products-and-catalog/` and `../units-of-measure-and-packaging/`: products, product categories, units of measure and packagings (quantities are converted between the request unit, the vendor unit and the product unit).
3. `../purchasing/` and `../pricing-and-pricelists/`: purchase orders, purchase order lines, vendor prices (buy rules create purchase order lines and select vendor prices).
4. `../sales/`: sales orders and lines (the main external source of procurement requests).
5. `../manufacturing/`: manufacturing orders and bills of materials (manufacture rules create manufacturing orders; kit bills of materials explode procurements into component procurements; subcontracting resupply).
6. The Contacts and Organizations domain: companies (multi-company scoping of routes and rules) and contacts (vendors, delivery addresses, buyers). Every field this domain adds to Company and to Contact is nevertheless specified in full in `entities.md`, sections 21 and 22, so that this folder can be read on its own.
7. `../messaging-and-activities/`: activities (warning activities on procurement failure), chatter notes (deadline updates, origin links, vendor-not-found notifications).
8. `../platform-foundation/`: scheduled actions, sequences, company-dependent defaults, access groups, record rules.
9. `../inventory-valuation-and-costing/` and `../accounts-payable/`: the valuation and the vendor bill that later price the documents this domain creates. Nothing of this domain posts; the boundary is stated in `accounting-effects.md`.

## Navigation

| File | Content |
|---|---|
| `entities.md` | Every field of Route, Stock Rule, Reordering Rule, the wizards and reports, the procurement request structure, and every field added to entities owned elsewhere; constraints, defaults, on-change behaviors, lifecycle. |
| `workflows.md` | Rule selection, the run algorithm (pull, buy, manufacture), push application, chained move creation, make to order versus multi-step routes, reordering rule evaluation and the scheduler, manual replenishment, the replenishment report, drop shipping, inter-warehouse resupply, subcontractor resupply, warehouse creation and reconfiguration, cancellation propagation, date propagation, return to the vendor; state tables. |
| `business-rules.md` | The numbered rule catalog (prefix `RP-RULE-`): validations, guards, company consistency, uniqueness, rounding, date rules, exact messages. |
| `calculations.md` | Every formula: quantity to order, replenishment multiple rounding, lead days, lead horizon date, deadline date, purchase order dates, grouping windows, vendor selection, forecast reconciliation, daily demand graph, on-time delivery rate, purchase suggestion quantities, effective days to arrival, the walk that finds the purchase order line behind a move, worked examples. |
| `accounting-effects.md` | This domain creates no journal entries itself; the boundary with valuation, and the drop-ship and dropship-to-subcontractor valuation hooks. |
| `configuration.md` | Settings, shipped routes and rules, warehouse-generated routes and rules, sequences, operation types, scheduled action, access groups and record rules. |
| `interfaces.md` | Service operations (run, get rule, run scheduler, procure orderpoints, replenish, snooze, order to max), screens as workflows on views, reports, notifications and scheduled jobs. |
| `acceptance-criteria.md` | Given / When / Then scenarios derived from the automated test scenarios of the reference and from the rules and formulas documented here. |
| `glossary.md` | Domain terms with definitions. |
