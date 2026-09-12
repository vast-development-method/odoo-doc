# Manufacturing — Glossary

Every term used in this domain, defined in full. Terms are listed alphabetically. A term
whose meaning in this domain differs from its everyday meaning carries a note saying so.

Where a definition names a storage or transport identifier, it is reproduced exactly in
code font and accompanied by its full name in words.

---

## A

**Alternative work centre.** A work centre that may be substituted for another when a Work
Order is planned. The planner evaluates the Work Order's own work centre and every
alternative, computing the expected duration for each and asking each for its first
available slot, and keeps the one whose slot ends earliest. A work centre may not be its
own alternative. The relation is not automatically symmetric: declaring B as an alternative
of A does not make A an alternative of B, although the Work Orders screen of A offers to
show the Work Orders of the work centres that name A as an alternative.

**Allocation.** The act of earmarking newly produced goods for the documents waiting for
them. Manufacturing exposes it through the allocation control and the allocation report,
both gated by the allocation security group. The report is offered when a finished move is
storable, not cancelled, and a demand exists in the same warehouse that could be served by
it.

**Assistant.** A short-lived record that collects a user's answer to a question and then
acts. Seven exist in this domain: change production quantity, backorder confirmation,
consumption warning, split production (single and multiple), serial number assignment,
insufficient unbuild quantity, and work-in-progress accounting.

**Automatic filling.** The pass, run just before a Manufacturing Order is closed, that
generates a missing lot or serial number, sets the producing quantity to the remainder and
distributes it over the component and by-product moves. It runs only for an order that
qualifies: one where nothing is tracked, or whose total quantity is exactly 1, or whose
product is not serial-tracked and whose readiness is assigned, waiting or confirmed.

**Availability (of a component).** Whether the components of a Manufacturing Order can be
obtained. Reported as one of four states — available, expected, late, not available — and
as a text. Not to be confused with the readiness of the order, which is derived from the
reservation state of the component moves rather than from the forecast.

---

## B

**Backorder.** A Manufacturing Order created to carry the part of another order's quantity
that was not produced when that order was closed. It belongs to the same production group
as its parent, carries the next backorder sequence, and takes its share of the components,
the finished goods and the Work Order progress. See *split*.

**Backorder policy.** The setting on an operation type that decides what happens to an
unproduced remainder: *always* creates the backorder without asking, *ask* opens the
Backorder Confirmation assistant, and *never* drops the remainder and closes the order at
the produced quantity.

**Backorder sequence.** An integer on a Manufacturing Order. Zero means the order has no
backorder chain. The first split sets the original to 1 and gives each backorder the next
number; the number is rendered into the reference as a hyphen and three zero-padded digits.

**Batch size.** A quantity on a recipe. When batch sizing is enabled, every automatically
generated Manufacturing Order for the product carries exactly that quantity, repeatedly,
until the procured quantity is covered. It also seeds the default maximum batch size of the
split assistant.

**Bill of Materials.** See *recipe*.

**Blocked (of a work centre).** The state of a work centre that carries an open Productivity
Log whose loss category is availability or quality. A blocked work centre refuses to start
or validate any of its Work Orders.

**Blocked (of a Work Order).** The state of a Work Order whose ready quantity is zero,
because a predecessor has not produced enough. It is computed, not set.

**By-product.** A secondary output of a recipe: a product produced alongside the finished
product by the same production. Each by-product line carries a quantity per recipe output,
a producing operation, a variant restriction and a percentage cost share. A by-product may
not be the recipe's own finished product.

---

## C

**Capacity.** The number of pieces of a given product a work centre processes in parallel.
Resolved from the work centre's capacity lines by a three-key preference: the line naming
exactly this product in the product's own unit, then the product-agnostic line in the
requested unit, then the product-agnostic line in the product's own unit. A capacity line
of zero means "use the default capacity" while still contributing its setup and cleanup
times.

**Carried quantity.** On a Work Order, the quantity already produced earlier in the
backorder chain and awaiting allocation at this Work Order. Written by the split algorithm
so that a backorder's Work Orders do not re-produce what their predecessors already
produced.

**Cleanup time.** The minutes a work centre needs after the last cycle of an operation. It
is added once to the expected duration, not once per cycle.

**Component.** A product consumed by a Manufacturing Order. Represented on the order by a
component Stock Move that runs from the components location to the production location. A
component line of a recipe carries the quantity per recipe output, the unit, the consuming
operation and the variant restriction.

**Components location.** The location a Manufacturing Order looks in for its components.
Taken from the operation type's default source, which in a one-step configuration is the
stock location and in a two- or three-step configuration the pre-production location.

**Consumption policy.** The recipe setting, copied onto the order at confirmation, that
decides what happens when the consumed quantities differ from the expected ones: *Allowed*
skips the check; *Allowed with warning* raises the warning and lets any manufacturing user
confirm; *Blocked* raises the warning and lets only a manufacturing administrator confirm.

**Cost mode.** The operation setting, copied once onto each Work Order at confirmation,
deciding whether the operation's cost uses the actual tracked time or the estimated
expected duration.

**Cost share.** The percentage of a production's total cost attributed to one by-product
line, divided among the quantity produced. Held with two decimal places, which matters for
the rounding of the complement. The sum over the applicable by-product lines must not
exceed 100 for any variant.

**Cycle.** One pass of an operation through a full work centre capacity. The number of
cycles is the quantity divided by the capacity, rounded up to a whole number.

---

## D

**Days to prepare.** A recipe setting saying how many days in advance a Manufacturing Order
must be created and confirmed so that its components can be replenished or its
sub-assemblies manufactured. It is part of the lead time a rule contributes and can be
recomputed from the recipe structure report.

**Dependency (between operations or Work Orders).** A declaration that one step must be
completed before another may start. Declared on operations and copied onto Work Orders at
confirmation. Dependencies drive both planning (a Work Order is planned after its
predecessors finish) and readiness (a Work Order's ready quantity is bounded by what its
predecessors have produced). A cycle is refused.

**Distribution (of the producing quantity).** The pass that writes the consumed quantity of
every component move, and of every by-product move and serial-tracked finished move, as
`(producing quantity − produced quantity) × unit factor`, rounded at the move's unit,
subject to the supply cap and skipping picked manual-consumption and by-product
moves.

**Duration deviation.** The signed percentage by which a Work Order's real duration differs
from its expected duration:
`100 × (expected − real) ÷ expected`, clamped to a signed 32-bit range and stored as an
integer. A negative value means the Work Order overran.

**Duration per unit.** A Work Order's real duration divided by the greater of its produced
quantity and 1, rounded to two decimals.

---

## E

**Effectiveness.** See *overall equipment effectiveness*.

**Efficiency.** See *time efficiency*.

**Expected duration.** The planned length of a Work Order in minutes. For a Work Order with
an operation, it is `setup + cleanup + cycles × cycle time × 100 ÷ efficiency`. For a
manually added Work Order it is the stored value rescaled by the quantity ratio.

**Explosion.** The recursive expansion of a recipe into its leaf component lines, following
nested kit recipes, converting units at every level and rounding each leaf quantity upward
at its own unit. It returns two lists: the recipes visited and the leaf lines with their
quantities.

**Extra unit cost.** A per-unit amount added to the production cost of a Manufacturing
Order without any counterpart entry. It is carried over to backorders. For a subcontracted
order it is overwritten by the subcontracting service cost derived from the purchase side.

---

## F

**Finished move.** A Stock Move of a Manufacturing Order that runs from the production
location to the finished-products location. There is one for the order's own product and
one per applicable by-product.

**Finished-products location.** Where a Manufacturing Order puts what it produces. Taken
from the operation type's default destination, which in a one- or two-step configuration is
the stock location and in a three-step configuration the post-production location.

**Flexible consumption.** See *consumption policy*.

---

## K

**Kit.** A recipe of kind `phantom` (kit). A kit is never manufactured: wherever a Stock
Move for the kit product appears on a document, that move is replaced at confirmation by
one move per leaf component. The kit product itself never moves, never holds stock, is
never valued, cannot be counted and cannot have a reordering rule. A kit's on-hand,
forecast, incoming, outgoing and free quantities are derived from its components.

---

## L

**Leaf line.** A component line of an exploded recipe whose component has no kit recipe of
its own, and which therefore produces a real component move.

**Lead time (manufacturing).** The number of days a recipe declares between the start and
the finish of a production. It shifts the planned start of an automatically created order
backwards and contributes to the cumulative delay a rule reports.

**Loss category.** One of four classifications of recorded work centre time:
*availability*, *performance*, *quality*, *productive*. The categories drive two different
things and must not be confused:

- for the **working state** of the work centre, *productive* and *performance* mean "in
  use" and the other two mean "blocked";
- for the **effectiveness formula**, only *productive* counts as productive time and the
  other three, including *performance*, count as blocked time.

**Loss reason.** A named reason attached to a loss category. A reason marked as a blocking
reason is offered to users; the two non-blocking shipped reasons — *Fully Productive Time*
and *Reduced Speed* — are assigned by the system.

---

## M

**Make to order.** A procurement method under which a move's supply is created specifically
for it rather than taken from stock. A component move whose product is made to order causes
its own supplying chain — a purchase, a transfer or a child Manufacturing Order — when the
order is confirmed.

**Manual consumption.** A flag on a component move meaning that its consumed quantity is
registered by hand only and is never overwritten by the distribution pass. It is set
automatically when the generating recipe line names a consuming operation, and it becomes
true whenever a user edits a component move's consumed quantity so that it differs from the
demand.

**Manufacturing Order.** The order to build a stated quantity of a stated product: the
components to consume, the finished goods and by-products to produce, the operations to
execute, the dates, the state and the results. Also called *the order* in this folder.

**Manufacturing readiness (of a recipe).** The recipe setting deciding when an order is
considered ready while its components are only partially reserved: *when all components are
available*, or *when components for the first operation are available*.

**Merge.** Combining several Manufacturing Orders of the same product, recipe, state and
operation type into one order for the sum of their quantities. The merged orders are
cancelled and their moves' production group is re-stamped onto the survivor.

---

## O

**Operation.** A named step of a recipe performed at a work centre, with a cycle duration,
optional predecessors and a cost mode. Operations are the templates from which Work Orders
are instantiated on an order. In this domain the word never means an operation type or a
transfer.

**Operation type.** The document category that governs a Manufacturing Order: its sequence,
its default locations, its backorder policy, its reservation method, its lot settings and
its automatic-printing switches. A manufacturing operation type has the code
`mrp_operation` (manufacturing).

**Order unit.** The unit of measure of a Manufacturing Order. Distinct from the recipe unit
and from the product's reference unit; conversions between the three occur constantly.

**Outdated recipe.** A flag on a Manufacturing Order saying that its recipe changed after
the order was created. Set when the recipe's components, by-products, product or quantity
change, or when one of its operations is created, written, archived or unarchived. Cleared
by applying the update, and cleared automatically on a confirmed order whose product no
longer matches the recipe.

**Overall equipment effectiveness.** A percentage over the last month:
`productive minutes × 100 ÷ (productive minutes + blocked minutes)`, rounded to two
decimals, and zero when there is no productive time. Blocked minutes include the
performance category.

---

## P

**Performance (of a work centre).** A percentage over the last month:
`100 × sum of expected durations ÷ sum of real durations` over the finished Work Orders of
that work centre, stored as an integer and zero when the real-duration sum is zero. Above
100 means the work centre is faster than expected.

**Phantom.** The stored value of the kit recipe kind. See *kit*.

**Picked.** A flag on a Stock Move meaning "this quantity has really been handled". A
component move that is picked is posted when the order closes; one that is not picked is
cancelled. Picking is also what moves an order into the `progress` state.

**Post-production location.** The internal location a three-step warehouse puts finished
goods in before a push rule stores them in stock.

**Pre-production location.** The internal location a two- or three-step warehouse gathers
components in before they are consumed.

**Production capacity (of an order).** How many units the components currently on hand
allow: the minimum, over the storable component moves, of the on-hand quantity divided by
the unit factor, rounded at the product's reference unit and capped by the quantity to
produce.

**Production cost.** The sum of the values of the consumed component moves, the costs of
the Work Orders and the extra unit cost times the produced quantity. It is allocated to the
by-products by their cost shares and to the finished product by the complement.

**Production group.** The technical grouping that binds a Manufacturing Order to its
backorders and, through its parent and child links, to the orders above and below it in a
multi-level chain. It is stamped on every Stock Move of its orders, which is how an order's
transfers are found.

**Production location.** The virtual location of usage `production` that stands for "inside
the production process". Components move into it; finished goods move out of it. It is the
location that carries the Production account in the accounting model.

**Productivity Log.** One recorded time interval on a work centre, optionally on a Work
Order, carrying a loss reason and a computed duration in minutes.

**Producing quantity.** The quantity being produced in the current pass of a Manufacturing
Order. It is the input of the distribution and of the completion; it is not the quantity to
produce and not the quantity produced.

**Push rule.** A rule that moves goods onward after they arrive somewhere. In this domain,
the three-step configuration uses one to move finished goods from the post-production
location to stock.

---

## Q

**Quantity produced.** The sum of the quantities of the picked, non-cancelled finished
moves for the order's own product. It grows as backorders of the chain are closed.

**Quantity to produce.** The order's target quantity, expressed in the order unit. It is
rescaled by the change-quantity assistant and by the split.

---

## R

**Ready quantity.** On a Work Order, how many units it may process now:
its remaining quantity when it has no live predecessor, and otherwise the running minimum
over its predecessors of what they have produced and carried, minus what this Work Order
has already produced and carried.

**Recipe.** The word used throughout this folder for a Bill of Materials: the statement
that a given quantity of a product, expressed in a given unit, is made of a set of
components, optionally produces by-products, and optionally passes through operations. Its
kinds are *manufacture this product*, *kit* and, with the subcontracting capability,
*subcontracting*.

**Recipe unit.** The unit of measure of a recipe, in which its quantity and the explosion
multiplier are expressed.

**Reference unit.** A product's own unit of measure, in which its on-hand quantity and
every quantity comparison are expressed.

**Remaining quantity (of a Work Order).** `max(round(production quantity − carried quantity
− produced quantity), 0)`.

**Reservation.** Earmarking stock for a component move. Manufacturing does not define its
own reservation: it uses the inventory domain's, subject to the order's priority and to the
operation type's reservation method.

**Resupply (of a subcontractor).** Sending components from the company's stock to a
subcontractor's location, either through the make-to-order resupply route or by hand. A
component that is not resupplied is assumed to be owned by the subcontractor.

---

## S

**Setup time.** The minutes a work centre needs before the first cycle of an operation. It
is added once to the expected duration.

**Should-consume quantity.** On a component move, the quantity the current producing
quantity implies: `(producing quantity − produced quantity) × unit factor`, rounded at the
move's unit. It is what the readiness upgrade compares the reserved quantity against.

**Skip rule.** The test that decides whether a component line, a by-product line or an
operation applies to a given finished product, given the line's attribute-value restriction
and the never-variant values chosen on the order. See [entities.md](entities.md) §2.2.

**Slot.** The reservation of a work centre's calendar that represents a Work Order's planned
window. Creating it is what makes a Work Order planned; deleting it is what unplans it. A
slot spans the closed hours of the calendar, while the duration is counted only over
working time.

**Split.** Dividing one Manufacturing Order into several, either explicitly through the
split assistant or implicitly when a backorder is created. The quantities, the moves, the
reservations and the Work Order progress are all divided.

**Subcontracting.** Manufacturing performed by an external partner. The finished product is
received rather than produced in house, and the corresponding Manufacturing Order is created
and closed automatically by that receipt. A subcontracting recipe may have no operation and
no by-product.

**Subcontracting location.** The internal location that stands for "at the subcontractor".
One per company by default, optionally one per partner. Components sent there are still
owned and valued by the company.

---

## T

**Time efficiency.** A percentage on a work centre. Below 100 it lengthens every expected
duration proportionally; above 100 it shortens it. It falls back to 100 when absent.

**To close.** The state of a Manufacturing Order whose production is complete in the sense
that every Work Order is finished or cancelled, or — for an order without Work Orders — the
producing quantity has reached the quantity to produce. The order still has to be closed
explicitly.

**Tracked (product).** A product whose individual units or batches are identified by a lot
or serial number. The tracking mode is *none*, *lot* (one number for many units) or *serial*
(one number per unit). Tracking drives the producing-lot rules, the uniqueness checks, the
distribution cap and the unbuild lot matching.

---

## U

**Unbuild.** The reversal of a build: a record that consumes a quantity of a finished
product (and its by-products) and returns the corresponding components to stock. Its factor
is always computed against the source order's produced quantity, never against what is left,
so repeated unbuilds of the same order are consistent.

**Unit factor.** On a Stock Move of a Manufacturing Order, the move's demand divided by the
greater of the order's outstanding quantity and 1:
`demand ÷ max(quantity to produce − quantity produced, 1)`. It is the multiplier that turns
a producing quantity into a consumed or produced quantity.

**Unreserve.** Releasing the stock earmarked for an order's component moves. Offered only
when no component move is picked; by-product moves are never unreserved.

---

## V

**Variant restriction.** The set of product attribute values on a component line, a
by-product line or an operation that limits it to particular variants of the finished
product. Values of an attribute that creates variants are matched against the product's own
values; values of a *no variant* attribute are matched against the never-variant values
chosen on the order.

---

## W

**Waiting another operation.** The readiness value of a Manufacturing Order whose component
moves are waiting for an supplying move — typically a child Manufacturing Order or an
incoming transfer.

**Work centre.** A production resource: a machine, a bench or a cell. It owns a working-time
calendar through the shared resource mechanism, a time efficiency, setup and cleanup times,
an hourly cost, per-product capacities, alternatives, tags and a productivity-logging
apparatus.

**Work centre load.** The sum of the expected durations, in minutes, of the Work Orders of a
work centre in state blocked, ready or progress.

**Work Order.** One operation of one Manufacturing Order: its own state, its planned window
in the work centre's calendar, its dependencies, its time logs, its expected and real
durations and its produced quantity.

**Work in progress.** Production started but not closed. Its value — the components already
consumed plus the work-centre time already spent — is recognised at a reporting date by a
manual entry and its automatic reversal.

**Working state.** The state of a work centre derived from its open time logs: *normal*
(idle), *done* (in use, shown as "In Progress"), or *blocked*.

---

## Identifier index

Storage and transport identifiers used in this folder, with their full names.

| Identifier | Full name |
|---|---|
| `BoM` (inside reproduced messages and labels only) | Bill of Materials |
| `MO` (inside reproduced messages, labels and references only) | Manufacturing Order |
| `WIP` (inside reproduced journal item labels only) | work in progress |
| `MTO` (inside reproduced rule names only) | make to order |
| `pdf` | the portable-document label format |
| `zpl` | the label-printer format |
| `mrp.bom`, `mrp_bom` | Bill of Materials |
| `mrp.bom.line`, `mrp_bom_line` | Bill of Materials Line |
| `mrp.bom.byproduct`, `mrp_bom_byproduct` | Bill of Materials By-Product |
| `mrp.routing.workcenter`, `mrp_routing_workcenter` | Operation |
| `mrp.workcenter`, `mrp_workcenter` | Work Centre |
| `mrp.workcenter.tag`, `mrp_workcenter_tag` | Work Centre Tag |
| `mrp.workcenter.capacity`, `mrp_workcenter_capacity` | Work Centre Capacity |
| `mrp.workcenter.productivity`, `mrp_workcenter_productivity` | Productivity Log |
| `mrp.workcenter.productivity.loss`, `mrp_workcenter_productivity_loss` | Productivity Loss Reason |
| `mrp.workcenter.productivity.loss.type`, `mrp_workcenter_productivity_loss_type` | Productivity Loss Category |
| `mrp.production`, `mrp_production` | Manufacturing Order |
| `mrp.production.group`, `mrp_production_group` | Production Group |
| `mrp.workorder`, `mrp_workorder` | Work Order |
| `mrp.unbuild`, `mrp_unbuild` | Unbuild Order |
| `change.production.qty` | Change Production Quantity assistant |
| `mrp.production.backorder`, `mrp.production.backorder.line` | Backorder Confirmation assistant and its lines |
| `mrp.consumption.warning`, `mrp.consumption.warning.line` | Consumption Warning assistant and its lines |
| `mrp.production.split`, `mrp.production.split.line`, `mrp.production.split.multi` | Split Production assistant, its details and the multiple-split assistant |
| `mrp.production.serials` | Serial Number Assignment assistant |
| `stock.warn.insufficient.qty.unbuild` | Insufficient Unbuild Quantity warning |
| `mrp.account.wip.accounting`, `mrp.account.wip.accounting.line` | Work-in-progress accounting assistant and its lines |
| `report.mrp.report_bom_structure` | Recipe Structure report |
| `report.mrp.report_mo_overview` | Order Overview report |
| `mrp_operation` | the operation type code meaning "manufacturing" |
| `normal`, `phantom`, `subcontract` | the recipe kinds: manufacture this product, kit, subcontracting |
| `flexible`, `warning`, `strict` | the consumption policies: Allowed, Allowed with warning, Blocked |
| `all_available`, `asap` | the manufacturing readiness modes |
| `draft`, `confirmed`, `progress`, `to_close`, `done`, `cancel` | the Manufacturing Order states |
| `confirmed`, `assigned`, `waiting` | the Manufacturing Order readiness values: Waiting, Ready, Waiting Another Operation |
| `blocked`, `ready`, `progress`, `done`, `cancel` | the Work Order states: Blocked, To Do, In Progress, Finished, Cancelled |
| `normal`, `blocked`, `done` | the work centre working states: Normal, Blocked, In Progress |
| `availability`, `performance`, `quality`, `productive` | the productivity loss categories |
| `actual`, `estimated` | the operation cost modes |
| `mrp_one_step`, `pbm`, `pbm_sam` | the warehouse manufacturing step configurations |
| `manufacture` | the rule action that creates a Manufacturing Order |
| `mrp.unbuild` | the sequence code of the unbuild numbering |
| `stock.lot.serial` | the shared sequence code for lot and serial numbers |
| `mrp.workcenter_max_planning_iterations` | The system parameter bounding the slot search |
| `property_stock_account_production_cost_id` | the product category's Production Account |
| `account_production_wip_account_id` | the company's Production Work In Progress Account |
| `account_production_wip_overhead_account_id` | the company's Production Work In Progress Overhead Account |
| `property_stock_subcontractor` | the partner's Subcontractor Location |
| `is_kits` | the product flag meaning "a kit recipe exists for this product" |
| `unit_factor` | the move's Unit Factor |
| `should_consume_qty` | the move's Quantity To Consume |
| `cost_share` | the by-product Cost Share percentage |
| `qty_producing` | the order's Quantity Producing |
| `qty_produced` | the order's Quantity Produced |
| `qty_reported_from_previous_wo` | the Work Order's Carried Quantity |
| `duration_percent` | the Work Order's Duration Deviation percentage |
| `oee` | the work centre's Overall Equipment Effectiveness |
| `produce_delay` | the recipe's Manufacturing Lead Time |
| `days_to_prepare_mo` | the recipe's Days to prepare Manufacturing Order |

---

## Terms deliberately avoided

The following everyday manufacturing words are **not** used as defined terms in this
folder, because the system uses a different word for the same thing. They are listed so
that a reader coming from another system can map them.

| Common word | Word used here | Why |
|---|---|---|
| Routing | the set of operations of a recipe | The system has no separate routing entity; operations belong directly to a recipe. |
| Work order operation | Operation | The template; a Work Order is the instance. |
| Phantom bill | Kit | The stored value is `phantom`; the user-visible word is Kit. |
| Co-product | By-product | Only one term exists, and it carries the cost share. |
| Scrap factor | — | No scrap factor exists on a recipe; over-consumption is handled by the consumption policy. |
| Yield | — | No yield percentage exists; a lower output is expressed by producing less and backordering or cancelling the rest. |
| Lot size | Batch size | The recipe field is called batch size. |
| Shop order, job, work ticket | Manufacturing Order, Work Order | — |
| Teardown, disassembly | Unbuild | The entity is the Unbuild Order. |
| Standard routing time | Cycle duration | The per-cycle time of an operation. |
| Machine hour rate | Cost per hour | The work centre's hourly cost. |
