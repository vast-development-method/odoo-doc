# Inventory Operations — Glossary

Every term used anywhere in this domain folder is defined here. Terms are listed alphabetically. Where a term has a reproduced storage name or transport name, it is given in code font next to the term; those reproduced names are the only abbreviations allowed in this specification, and each is expanded in words on first use.

---

## A

**Additional move**
A Stock Move created on a Transfer after that Transfer was already confirmed. It carries the additional flag (`additional`), which causes the Transfer to re-run its confirmation step so that the new move leaves the new state and becomes part of the running document.

**Adjustment**
See *Inventory adjustment*.

**Aggregated product quantity**
A grouping of Stock Move Lines used when printing a delivery document. Lines are grouped by a composite key made of the product identifier, the product display name, the description on the move, the unit of measure of the move, the packaging unit of the move and, when the line has a destination container, the container identifier. Each group carries a done quantity and an ordered quantity. See `calculations.md`, section "Printed-document aggregation".

**Allocation**
The act of earmarking an incoming Stock Move to satisfy a specific outgoing Stock Move, performed through the Reception Report. Allocation creates a chain link (the incoming move becomes an originating move of the outgoing move) and switches the outgoing move to the advanced supply method.

**Annual inventory date**
A company-level day-and-month pair used to schedule a yearly count of every location that has no cyclic counting frequency of its own. See `calculations.md`, section "Next count date".

**Assigned**
The state value (`assigned`) of a Stock Move whose whole demand has been reserved (label "Available"), and the state value of a Transfer that is ready to be processed (label "Ready").

**Available quantity**
For one Stock Quantity record: the on-hand quantity minus the reserved quantity. For a product in a location: the sum of those differences over the matching Stock Quantity records, clamped at zero unless negative results are explicitly allowed. See `calculations.md`, section "Available quantity".

**A-star search**
The best-first search used by the least-packages removal strategy to pick the smallest set of containers whose combined free quantity covers a requested quantity. See `calculations.md`, section "Least packages removal strategy".

## B

**Backorder**
A new Transfer holding the part of the demand that was not processed when its parent Transfer was validated. The parent points at nothing; the backorder points at the parent through the back-order link (`backorder_id`).

**Backorder policy**
The setting on an Operation Type (`create_backorder`) deciding what happens to unprocessed demand at validation: ask the user, always create a backorder, or never create one (cancel the remainder).

**Barcode**
A short text stored on a Location, an Operation Type, a Package Type, a Product or a Lot, used to identify the record when scanning. Location barcodes are unique per company.

**Batch Transfer**
A grouping document (`stock.picking.batch`) that holds several Transfers so that they are picked, validated and printed together.

**Bulk weight**
The total weight of the goods on a Transfer that are not inside any container.

**Bypass reservation**
The property of a Location whose usage is vendor, customer, inventory loss or production: goods are considered infinitely available there and no reservation counters are maintained. Also true for any product that is not storable.

## C

**Capacity**
A limit belonging to a Storage Category, expressed either as a maximum quantity of one product or as a maximum number of containers of one Package Type.

**Chained move**
A Stock Move that has at least one originating move or one destination move. Chains implement multi-step receipts, multi-step deliveries and resupply between warehouses.

**Closest location strategy**
The removal strategy that orders candidate Stock Quantity records by the full location name ascending, then by record identifier descending.

**Company-scoped**
A record whose company field restricts visibility and usability to one company. A Location, a Route or a Lot may also be company-less, in which case it is shared.

**Complete name**
The slash-joined path of a Location from its top ancestor down to itself, skipping ancestors whose usage is virtual. Also, for a container, the greater-than-joined path of its parent containers.

**Confirmed**
The state value (`confirmed`) of a Stock Move that has been validated as a demand but has no reservation (label "Waiting"), and the state value of a Transfer none of whose moves could be reserved (label "Waiting").

**Consuming move**
A Stock Move whose Operation Type is of kind internal transfer or delivery, or whose source warehouse and destination warehouse differ. Consuming moves are the ones the forecast engine treats as taking goods out of a warehouse.

**Container**
A physical box, pallet or bin recorded as a Package. A container holds Stock Quantity records directly, other containers, or both.

**Counted quantity**
The quantity a person recorded during a physical count, stored on the Stock Quantity record (`inventory_quantity`).

**Cross dock**
The operation type created for every warehouse that both receives in more than one step and delivers in more than one step; it moves goods from the input location straight to the output location.

**Cyclic counting frequency**
A number of days on a Location. When greater than zero, every Stock Quantity record in that Location is given a next count date that far in the future after each count.

## D

**Deadline**
A date and time by which a Transfer or Stock Move should be completed to keep an external promise. Deadlines propagate along move chains in both directions.

**Delay alert date**
The latest scheduled date among the originating moves of a Stock Move when that date is later than the move's own scheduled date. A non-empty value means the move will be late because an upstream step is late.

**Delivery steps**
The warehouse configuration deciding how many Transfers a shipment needs: one (deliver), two (pick then deliver) or three (pick, pack, then deliver).

**Demand**
The planned quantity on a Stock Move, expressed in the move's unit of measure (`product_uom_qty`, label "Demand").

**Destination container**
The container a Stock Move Line will put its goods into (`result_package_id`). Distinct from the source container the goods came from.

**Destination move**
A Stock Move that must happen after this one; the link is the destination-moves relation (`move_dest_ids`).

**Difference**
On a Stock Quantity record in counting mode, the counted quantity minus the on-hand quantity.

**Dispatch management**
The optional capability that attaches a vehicle, a vehicle category, a driver and a dock to a Batch Transfer, and computes weight and volume load percentages.

**Done**
The terminal state value (`done`) of a Stock Move, a Stock Move Line or a Transfer, meaning the goods have actually changed location.

**Draft**
The initial state value (`draft`) of a Stock Move (label "New") or a Transfer (label "Draft"). Nothing is reserved and nothing is promised.

## E

**Entire package**
A container all of whose contents appear on one single Transfer with exactly the quantities the container holds. Such a container is moved as a unit: every one of its Stock Move Lines is given the container itself as destination container and is flagged as entire (`is_entire_pack`).

**Exchange**
A return that also re-issues the returned goods; produced by the Return Transfer wizard when the user asks for a replacement rather than a plain return.

**Extra move**
Historically, a move created at validation because more was processed than demanded. In the current behavior, over-processing does not create a move: the processed quantity simply exceeds the demand on the same move and no backorder is produced. See `calculations.md`, section "Over-processing".

## F

**First in first out**
The removal strategy that takes the goods with the oldest incoming date first. Ordering: incoming date ascending, then record identifier ascending.

**Forecast availability**
The quantity of a Stock Move that is expected to be coverable at the move's date, computed from the forecast engine of the replenishment domain.

**Free quantity**
The quantity of a product in a location that is on hand and not reserved.

## G

**Gather**
The operation that collects the candidate Stock Quantity records for a product at a location, ordered according to the applicable removal strategy. Gathering is either strict (exact match on every characteristic) or loose (match on product plus descendants of the location). See `calculations.md`, section "Gathering".

**Global Standards One barcode**
A structured barcode built from application identifiers. Stock Quantity records can generate one that carries the product code, the quantity and the lot or serial number.

## I

**Immediate transfer**
A Transfer validated without the user having reserved anything first: the validation step copies each move's demand into its done quantity before proceeding.

**Incoming date**
The date and time recorded on a Stock Quantity record when goods first arrived there (`in_date`). It is the key of the first in first out and last in first out orderings.

**Internal location**
A Location whose usage is internal. Only internal and transit locations hold counted stock.

**Inventory adjustment**
The act of writing a counted quantity onto Stock Quantity records and applying it, which creates and immediately completes Stock Moves between the counted location and the inventory loss location.

**Inventory loss location**
A virtual Location whose usage is inventory loss, used as the counterpart of every adjustment and of every scrap.

**Is outdated**
A flag on a Stock Quantity record meaning that the on-hand quantity changed after the counted quantity was entered, so the recorded difference no longer matches reality.

## L

**Last in first out**
The removal strategy that takes the goods with the newest incoming date first. Ordering: incoming date descending, then record identifier descending.

**Least packages**
The removal strategy that tries to satisfy the requested quantity by opening as few containers as possible, falling back to first in first out ordering inside the chosen set.

**Location**
A node of the storage tree (`stock.location`). Its usage decides whether it physically holds goods.

**Lock flag**
The flag on a Transfer (`is_locked`) that forbids editing the demand while the Transfer is open and the done quantities once it is completed.

**Lot**
An identified batch of a tracked product (`stock.lot`). A lot with exactly one unit is a serial number.

## M

**Merge key**
The tuple of fields that must be equal for two Stock Moves to be merged into one. See `calculations.md`, section "Move merging".

**Move line**
See *Stock Move Line*.

**Multi-step receipt**
A receipt configuration in which goods pass through an input location, optionally a quality control location, before reaching the stock location.

## O

**On hand quantity**
The physical quantity recorded on a Stock Quantity record (`quantity`), in the product's own unit of measure.

**Operation Type**
The template and counter holder for Transfers of one kind in one warehouse (`stock.picking.type`).

**Origin document**
A free text on a Transfer or a Stock Move naming the document that caused it.

**Originating move**
A Stock Move that must happen before this one; the link is the originating-moves relation (`move_orig_ids`).

**Outermost container**
For a container that is being placed into another container, the last container of that destination chain; for a container with no destination container, itself.

**Over-processing**
Recording a done quantity greater than the demand on a Stock Move.

**Owner**
A contact recorded on a Stock Quantity record or a Stock Move Line meaning that the goods belong to a third party (consignment).

## P

**Package**
See *Container*.

**Package History**
An immutable snapshot (`stock.package.history`) of one container as it was at the instant a Transfer that moved it was completed: its name, its parent before and after, its source and destination locations.

**Package Type**
A reusable container specification (`stock.package.type`): dimensions, base weight, maximum weight, numbering sequence, barcode, routes and reusability.

**Partially available**
The state value (`partially_available`) of a Stock Move for which some but not all of the demand has been reserved.

**Picked**
The flag on a Stock Move and on a Stock Move Line (`picked`) meaning that a person has physically handled that quantity. Only picked lines are completed at validation.

**Put-away rule**
A redirection rule (`stock.putaway.rule`) that sends goods arriving in one Location down to a specific sublocation.

**Put in pack**
The operation that creates a container and assigns it as destination container to a set of Stock Move Lines, or that nests an existing container into a new one.

## Q

**Quality control location**
The intermediate Location used by a three-step receipt between the input location and the stock location.

**Quantity record**
See *Stock Quantity*.

## R

**Reception Report**
A screen and printable document listing incoming quantities that are not yet allocated, next to the outgoing demands they could satisfy.

**Removal strategy**
The named method that decides in which order Stock Quantity records are consumed: first in first out, last in first out, closest location, least packages, and (when the companion capability is installed) first expired first out.

**Reservation**
The act of raising the reserved counter on Stock Quantity records and creating the matching Stock Move Lines, so that the goods cannot be taken by another document.

**Reservation date**
The date from which a Stock Move becomes eligible for automatic reservation, computed from the Operation Type's reservation method.

**Reservation method**
The Operation Type setting deciding when moves are reserved: at confirmation, manually, or a fixed number of days before the scheduled date.

**Resupply route**
A route created automatically between two warehouses so that one can be replenished from the other through a transit location.

**Return**
A Transfer that reverses a completed Transfer, created by the Return Transfer wizard. It points at the original through the return link (`return_id`).

**Route**
An ordered, named collection of Stock Rules (`stock.route`) selectable on products, product categories, warehouses or package types.

## S

**Scrap**
The removal of damaged goods from a storable location to a scrap (inventory loss) location, recorded as a Scrap document (`stock.scrap`) with one Stock Move.

**Serial number**
A Lot used to identify exactly one unit. Products tracked by serial number may only have one unit per Stock Move Line.

**Shipping policy**
The Transfer setting (`move_type`) deciding whether goods leave as soon as possible or only when every line is ready.

**Source container**
The container the goods are taken from on a Stock Move Line (`package_id`).

**Stock Move**
One product's planned travel from a source Location to a destination Location (`stock.move`).

**Stock Move Line**
One concrete detail of a Stock Move: an exact quantity, taken from an exact location, an exact lot, an exact container and an exact owner, put into an exact destination location and container (`stock.move.line`).

**Stock Quantity**
The record of an on-hand quantity for one product in one location with one lot, one container and one owner (`stock.quant`).

**Storage Category**
A reusable capacity and mixing policy (`stock.storage.category`) attachable to Locations.

**Strict matching**
Gathering with an exact match on location, lot, container and owner, as opposed to loose matching which accepts descendants of the location and ignores unset characteristics.

## T

**Transfer**
A document grouping the Stock Moves that travel together between two Locations (`stock.picking`).

**Transit location**
A Location whose usage is transit, used as the intermediate holding point between two warehouses or two companies.

**Traceability report**
A tree of completed Stock Move Lines showing where a lot, a product or a container came from and where it went.

## U

**Unreserve**
The operation that deletes the unpicked Stock Move Lines of a Stock Move and lowers the reserved counters on the corresponding Stock Quantity records.

**Usage**
The kind of a Location: vendor, virtual, internal, customer, inventory loss, production or transit.

## V

**Validation**
The act of completing a Transfer: checking it, deciding about a backorder, completing the moves, moving the quantities, creating the backorder and running the follow-up actions.

**View location**
A Location whose usage is virtual. It groups sublocations and may never hold goods.

## W

**Warehouse**
One physical site (`stock.warehouse`) owning a tree of Locations, a set of Operation Types, and the Routes and Stock Rules generated from its step configuration.

**Wave**
A Batch Transfer created from a selection of Stock Moves rather than whole Transfers; it carries the wave flag.
