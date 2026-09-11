# Manufacturing

This folder specifies the manufacturing domain of the platform: the definition of how
products are built from other products, the shop-floor resources that build them, the
orders that authorise and record production, the recording of consumption and output,
the reversal of production, and the cost and accounting consequences of all of it.

Everything stated here is behaviour of the current system. Where a behaviour is not
expressed explicitly by the system and a reimplementation must nevertheless choose
something, the text says "industry-standard default" and states the choice.

---

## 1. Scope

### 1.1 What this domain owns

| Capability | Summary |
|---|---|
| Bill of Materials definition | Recipes of two kinds: manufactured recipes that drive a Manufacturing Order, and kit recipes that are exploded away at the document that references them. Each recipe carries component lines, by-product lines, operations, a consumption policy, lead times and a batch size. |
| Bill of Materials explosion | The recursive expansion of a recipe into its leaf components, following nested kit recipes, converting units at every level, rounding component demand upward, and refusing cycles. |
| Bill of Materials selection | The deterministic rule for picking which recipe applies to a product, a company, an operation type and a recipe kind. |
| Variant-restricted lines | Component lines, by-product lines and operations may be restricted to particular attribute values of the finished product, including values of attributes that do not create separate product variants. |
| Work centres | Production resources with a working-time calendar, a time efficiency percentage, setup and cleanup times, an hourly cost, per-product parallel capacities, substitutable alternatives, tags, and a full productivity-logging apparatus that yields blocked time, productive time, performance and overall equipment effectiveness. |
| Operations | Named steps of a recipe bound to a work centre, with a fixed or learned cycle duration, dependency links between operations, and a cost attribution mode. |
| Manufacturing Orders | The order entity: a full state machine, automatic generation of component and finished Stock Moves from the recipe, reservation, partial production, consumption policies, backorders, splits, merges, serial and lot assignment, locking, multi-level chains, and every side effect on inventory. |
| Work Orders | The per-operation execution records: their own state machine, dependency-driven readiness, planning against the work centre calendar, time tracking through productivity logs, expected versus real duration, and deviation percentages. |
| Scheduling | The slot-finding algorithm that places a Work Order into the first interval of its work centre's calendar long enough to hold it and free of other Work Orders, forward or backward, with alternative work centre comparison. |
| Unbuild | The reversal entity that consumes a finished product and returns its components and by-products to stock, either from a recorded Manufacturing Order or from a recipe. |
| Subcontracting | Manufacturing performed by an external partner, triggered by a receipt of the finished product, with component resupply, dedicated locations, a partner portal, and a purchase-driven variant. |
| Manufacturing cost | The allocation of component value, operation cost and by-product cost share to the finished product, and the valuation layers and journal entries this produces. |
| Procurement integration | The manufacture rule: how a demand for a product is turned into a Manufacturing Order, how lead times accumulate, and how the multi-step warehouse configurations create the pick-components and store-finished transfers. |
| Reporting | The recipe structure report with cost and availability roll-up, the order overview report, the allocation report, and the printable order and label documents. |

### 1.2 What this domain does not own

- Generic Stock Move, Stock Move Line, Stock Quant, reservation and transfer mechanics
  live in [inventory operations](../inventory-operations/README.md). This folder describes
  only the manufacturing-specific fields and overrides on those entities.
- Costing methods, valuation layers and the general inventory accounting entries live in
  [inventory valuation and costing](../inventory-valuation-and-costing/README.md). This
  folder describes the manufacturing-specific production cost formula and the entries that
  production and unbuild add.
- Reordering rules, routes, rules in general, the scheduler and the forecast report live in
  [replenishment and procurement](../replenishment-and-procurement/README.md). This folder
  describes only the manufacture rule action and the manufacturing lead time components.
- Unit conversion arithmetic lives in
  [units of measure and packaging](../units-of-measure-and-packaging/README.md).
- Product templates, variants and attribute values live in
  [products and catalog](../products-and-catalog/README.md).
- Purchase orders and vendor bills live in [purchasing](../purchasing/README.md); the
  subcontracting purchase variant is described here only where it differs.
- Sales orders live in [sales](../sales/README.md); kit delivery and the manufacturing
  link from a sales order are described here only where they differ.

---

## 2. Entities

### 2.1 Core entities owned by this domain

| Entity | Transport name | Table | One-line purpose |
|---|---|---|---|
| Bill of Materials | `mrp.bom` | `mrp_bom` | A recipe: what a product is made of, in what quantity, through which operations, with which by-products. |
| Bill of Materials Line | `mrp.bom.line` | `mrp_bom_line` | One component of a recipe, with its quantity, unit, consuming operation and variant restriction. |
| Bill of Materials By-Product | `mrp.bom.byproduct` | `mrp_bom_byproduct` | One secondary output of a recipe, with its quantity, producing operation, variant restriction and percentage share of the production cost. |
| Operation | `mrp.routing.workcenter` | `mrp_routing_workcenter` | One named step of a recipe performed at a work centre, with a cycle duration and dependency links. |
| Work Centre | `mrp.workcenter` | `mrp_workcenter` | A production resource with a calendar, an efficiency, costs, capacities and alternatives. |
| Work Centre Tag | `mrp.workcenter.tag` | `mrp_workcenter_tag` | A free label grouping work centres. |
| Work Centre Capacity | `mrp.workcenter.capacity` | `mrp_workcenter_capacity` | How many pieces of one product (or of any product measured in one unit) a work centre processes in parallel, with the setup and cleanup times that apply. |
| Productivity Loss Category | `mrp.workcenter.productivity.loss.type` | `mrp_workcenter_productivity_loss_type` | One of the four effectiveness categories: availability, performance, quality, productive. |
| Productivity Loss Reason | `mrp.workcenter.productivity.loss` | `mrp_workcenter_productivity_loss` | A named reason attached to a category, either a blocking reason chosen by a user or a system reason. |
| Productivity Log | `mrp.workcenter.productivity` | `mrp_workcenter_productivity` | One time interval recorded against a work centre and optionally a Work Order, carrying a loss reason and a computed duration in minutes. |
| Manufacturing Order | `mrp.production` | `mrp_production` | The order to build a quantity of a product: components to consume, finished goods to produce, operations to execute, state, dates and results. |
| Production Group | `mrp.production.group` | `mrp_production_group` | The technical grouping that binds an order to its backorders and to the orders it generated or was generated by. |
| Work Order | `mrp.workorder` | `mrp_workorder` | One operation of one Manufacturing Order, with its own state, planned window, dependencies, time logs and produced quantity. |
| Unbuild Order | `mrp.unbuild` | `mrp_unbuild` | The reversal of a build: consumes finished goods and by-products, returns components. |

### 2.2 Assistant entities (transient, owned by this domain)

| Entity | Transport name | One-line purpose |
|---|---|---|
| Change Production Quantity assistant | `change.production.qty` | Changes the quantity to produce of a confirmed order and rescales every component move, finished move and Work Order duration. |
| Backorder Confirmation assistant | `mrp.production.backorder` | Asks, per order, whether the unproduced remainder becomes a backorder or is dropped. |
| Backorder Confirmation Line | `mrp.production.backorder.line` | One order inside that question. |
| Consumption Warning assistant | `mrp.consumption.warning` | Lists every component whose consumed quantity differs from the expected quantity, and offers to confirm anyway or to reset the quantities. |
| Consumption Warning Line | `mrp.consumption.warning.line` | One component difference inside that list. |
| Split Production assistant | `mrp.production.split` | Splits one order into several, by maximum batch size or by explicit detail lines. |
| Split Production Detail | `mrp.production.split.line` | One resulting quantity, responsible user and start date inside a split. |
| Split Multiple Productions assistant | `mrp.production.split.multi` | Holds several split assistants when splitting a selection of orders. |
| Serial Number Assignment assistant | `mrp.production.serials` | Generates or accepts a list of serial numbers for a tracked finished product, and either assigns them or splits the order one unit per number. |
| Insufficient Unbuild Quantity warning | `stock.warn.insufficient.qty.unbuild` | Warns that the on-hand quantity is below the quantity to unbuild and lets the user proceed. |
| Label Type choice | `picking.label.type` (extended) | Chooses between product labels and lot labels when printing from an order. |

### 2.3 Report entities (computed, owned by this domain)

| Entity | Transport name | One-line purpose |
|---|---|---|
| Recipe Structure report | `report.mrp.report_bom_structure` | Recursive roll-up of a recipe: component costs, operation costs, by-product cost shares, availability, lead times and the resupply path of each component. |
| Order Overview report | `report.mrp.report_mo_overview` | Recursive roll-up of one Manufacturing Order: real versus expected component cost, operation cost, by-product allocation, unit cost, and the replenishment documents behind every component. |

### 2.4 Entities of other domains extended here

| Entity | Extension |
|---|---|
| Stock Move (`stock.move`) | Links to the order it feeds (component side) or the order that produced it (finished side), to the recipe line or by-product line that generated it, to the consuming Work Order and Operation, to the Unbuild Order, plus the unit factor, the cost share, the manual-consumption flag and kit explosion. |
| Stock Move Line (`stock.move.line`) | Links to the Work Order and the Manufacturing Order; kit-aware aggregation for delivery documents. |
| Stock Picking (`stock.picking`) | Links to the orders in the same production group; subcontracting behaviour on receipts. |
| Stock Rule (`stock.rule`) | Adds the `manufacture` (manufacture) action and the matching order-creation algorithm, and contributes the manufacturing components of the lead time. |
| Stock Route (`stock.route`) | A route containing a manufacture rule only resupplies products that have a manufactured recipe. |
| Stock Warehouse (`stock.warehouse`) | Adds the manufacturing operation type, the manufacturing step configuration (one, two or three steps), the pre-production and post-production locations and their routes. |
| Reordering Rule (`stock.warehouse.orderpoint`) | Adds the chosen recipe, the manufacturing lead-time contribution and the "days to prepare" horizon. |
| Scrap (`stock.scrap`) | Adds the order and Work Order links, and kit explosion when scrapping a kit. |
| Lot / Serial Number (`stock.lot`) | Restricts creation of component lots when the operation type forbids it. |
| Stock Quant (`stock.quant`) | Forbids counting a kit product directly. |
| Product Template / Product Variant | Adds recipe collections, the kit flag, the manufactured quantity, and kit-aware on-hand and forecast quantities. |
| Company (`res.company`) | Creates the unbuild numbering sequence per company. |

---

## 3. Reading order

1. **[glossary.md](glossary.md)** — the vocabulary. Read the definitions of *recipe*,
   *kit*, *component*, *by-product*, *cost share*, *unit factor*, *consumption policy*,
   *backorder*, *cycle*, *capacity* and *effectiveness* before anything else.
2. **[entities.md](entities.md)** — every entity, every field, every default, every
   computed rule, every constraint, every relation.
3. **[state-machines.md](state-machines.md)** — the four state fields of the domain
   (order state, order readiness, Work Order state, Unbuild state) plus the work centre
   working state, with their transition tables and diagrams.
4. **[calculations.md](calculations.md)** — every formula: explosion, unit factor,
   expected duration, cycle count, capacity selection, effectiveness, cost roll-up,
   by-product allocation, backorder split, kit quantity from component moves.
5. **[workflows.md](workflows.md)** — the operational sequences from first draft to
   closed order, including planning, partial production, backorders, splits, merges,
   unbuild and subcontracting.
6. **[business-rules.md](business-rules.md)** — every validation, every error message,
   every permission check, every locking rule, every edge case.
7. **[accounting-effects.md](accounting-effects.md)** — the journal entries produced by
   production, by unbuild, by work-in-progress recognition and by subcontracted receipts.
8. **[configuration.md](configuration.md)** — settings, groups, access matrix, record
   rules, sequences, shipped default records, scheduled jobs and system parameters.
9. **[interfaces.md](interfaces.md)** — menus, views, buttons, named operations, routes,
   reports, labels and notification templates.
10. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given/When/Then
    scenarios with concrete numbers that a reimplementation must pass.

---

## 4. Dependencies on other domains

| Domain | What manufacturing needs from it |
|---|---|
| [inventory operations](../inventory-operations/README.md) | Stock Move and Stock Move Line lifecycle, reservation, quantity on hand, locations of usage `production` and `internal`, operation types, transfers, lots and serial numbers, scrap, the reception/allocation report. |
| [units of measure and packaging](../units-of-measure-and-packaging/README.md) | The conversion function between two units of the same category, the rounding methods (`UP`, `DOWN`, `HALF-UP`), the comparison and zero tests at a unit's precision. |
| [products and catalog](../products-and-catalog/README.md) | Product templates and variants, product attribute values, the "create variant" mode of an attribute (always / dynamic / never), the tracking mode (none / lot / serial), the production location property, the standard price. |
| [inventory valuation and costing](../inventory-valuation-and-costing/README.md) | Valuation layers, costing methods, the stock input/output/valuation accounts, the weighted-average update, the production cost posting hook. |
| [replenishment and procurement](../replenishment-and-procurement/README.md) | The procurement record, the rule matching algorithm, reordering rules, the scheduler, the delay computation contract. |
| [purchasing](../purchasing/README.md) | Purchase orders that resupply subcontractors and that carry the subcontracting service. |
| [sales](../sales/README.md) | Sales order lines whose delivered quantity is derived from kit component moves, and the make-to-order chain that creates orders from a sale. |
| [analytic accounting](../analytic-accounting/README.md) | The analytic distribution applied to work centre costs and component consumption. |
| [general ledger](../general-ledger/README.md) | Journal entries, journals and accounts used by production and unbuild postings. |
| [messaging and activities](../messaging-and-activities/README.md) | The discussion thread, the tracked-field log, the activity scheduling and the exception activities logged on supplying documents. |
| [attendances and working time](../attendances-and-working-time/README.md) | The working-time calendar, the resource, the leave interval and the interval arithmetic used by work centre planning. |

---

## 5. Companion capabilities catalogued here

The domain is delivered as a core capability plus a set of companions. Each companion is
described in the file where its behaviour belongs; this table says which file.

| Companion capability | Where described |
|---|---|
| Manufacturing accounting: work-in-progress account posting, production cost allocation, analytic lines for work centre time, the valuation report contribution | [accounting-effects.md](accounting-effects.md), [calculations.md](calculations.md) |
| Subcontracting: subcontracted recipes, subcontracting locations, receipt-driven production, component resupply, the partner portal | [workflows.md](workflows.md), [entities.md](entities.md), [business-rules.md](business-rules.md) |
| Subcontracting with purchase: purchase-order driven resupply, the subcontracted price on the bill | [workflows.md](workflows.md), [accounting-effects.md](accounting-effects.md) |
| Subcontracting with drop shipping: the subcontractor ships directly to the customer | [workflows.md](workflows.md) |
| Landed costs on production: allocating extra costs to a finished production | [accounting-effects.md](accounting-effects.md) |
| Expiry dates on production: propagating component expiry to the produced lot and warning on expired components | [business-rules.md](business-rules.md), [workflows.md](workflows.md) |
| Repair and manufacturing: consuming manufactured parts in repairs | [workflows.md](workflows.md) |
| Purchasing and manufacturing: receiving a kit, the received quantity of a kit line | [calculations.md](calculations.md) |
| Sales and manufacturing: delivering a kit, the delivered quantity of a kit line, the order chain from a sale, the kit margin | [calculations.md](calculations.md), [workflows.md](workflows.md) |
| Project and manufacturing: attributing an order to a project and its analytic account | [accounting-effects.md](accounting-effects.md) |
| Point of sale and manufacturing: the cost of goods sold of a kit sold at the counter | [accounting-effects.md](accounting-effects.md) |

---

## 6. Conventions used in this folder

- Entity names are written in full, in title case: Manufacturing Order, Bill of Materials,
  Work Order, Work Centre, Stock Move.
- Storage and transport names are reproduced exactly in code font and are always
  accompanied, on first use in each file, by their full name in words.
- Quantities are always qualified by the unit they are expressed in. Three units appear
  constantly and must not be confused:
  - the **order unit** — the unit of the Manufacturing Order (`product_uom_id`, the unit
    of measure of the order);
  - the **recipe unit** — the unit of the Bill of Materials (`product_uom_id` on the
    recipe);
  - the **reference unit** — the product's own unit of measure (`uom_id`), in which
    on-hand quantities and comparisons are expressed.
- "Round at the precision of unit *u*" means: apply the rounding of unit *u* (its
  `rounding` value, the smallest representable increment) with the named rounding method.
  The default rounding method is round-half-away-from-zero unless stated otherwise.
- Every error message is reproduced with its placeholders spelled out in words.
