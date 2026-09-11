# Entities

This file specifies every entity of the repair and maintenance domain: its purpose, its lifecycle, its complete field table, its relations, its uniqueness rules, its defaults, its computed fields with the rules that produce them, its ordering, its display-name rule, its archival behaviour and its multi-company behaviour.

Conventions used in every field table:

- **Field (storage name)** gives the business name of the field followed by the reproduced storage name in code font. The storage name is the column name in the entity's table, or — for a many-to-many relation — the name of the relation used in the transport contract.
- **Type** gives the abstract data type. `reference to one` means a single-valued reference to another entity (a foreign key). `collection of` means a multi-valued reference. `selection` means a closed list of stored values each with a display label.
- **Meaning and rules** states, in order: what the field means; whether it is required; its default; whether it is computed and from what; whether the computed value is stored; whether it is read-only; whether it is carried into a duplicate; whether changes to it are recorded in the discussion thread; whether it is restricted by company; whether it is indexed; and, for references, what happens to this record when the referenced record is deleted.

Throughout, *the current company* means the company the acting user is operating in, and *the acting user* means the user performing the operation.

---

## 1. Repair Order

**Repair Order** (`repair.order`, table `repair_order`).

### 1.1 Purpose

A Repair Order is the document that governs the restoration of one product. It answers six questions:

1. **What is being repaired?** The product to repair, optionally a specific lot or serial number, in a quantity expressed in a unit of measure.
2. **What goes into it and what comes out of it?** A collection of part lines, each of which is a Stock Move carrying a part kind: *add*, *remove* or *recycle*.
3. **Where do the goods come from and go to?** Six locations: the component source location, the product source and destination locations, the added-parts destination location, the removed-parts destination location, and the recycled-parts destination location.
4. **For whom and when?** A customer, a responsible user, a scheduled date and a priority.
5. **Is it chargeable?** A warranty flag, and an optional link to a Sales Order on which the added parts are billed.
6. **What is its progress?** A state among *new*, *confirmed*, *under repair*, *repaired* and *cancelled*.

### 1.2 Inherited behaviour

The Repair Order participates in three shared behaviours specified in other domains:

- **Discussion thread** — it carries messages, followers, attachment counters and tracked-field logging. See `../messaging-and-activities/`. Message posting on a Repair Order always notifies the author when the author is mentioned, which differs from the platform default of suppressing self-notification.
- **Scheduled activities** — it carries planned activities with their deadlines, types, summaries and an exception decoration. See `../messaging-and-activities/`.
- **Product catalog** — it can be filled from a product catalog screen in which products are picked and quantities typed directly. The catalog contract is specified in `../products-and-catalog/`; the repair-specific bindings are given in section 1.7 below and in `interfaces.md`.

### 1.3 Ordering, display name and identity

- **Default ordering**: by priority descending, then by creation timestamp descending. Urgent repairs therefore sort above normal ones, and within one priority the most recently created repair comes first.
- **Display name**: the repair reference (`name`).
- **Uniqueness**: no database uniqueness constraint is declared on the reference. Uniqueness is achieved in practice because the reference is drawn from the numbering sequence attached to the repair operation type (see `configuration.md`).
- **Archival**: the Repair Order has no archive flag. A repair that must be taken out of the working set is cancelled, not archived.
- **Deletion**: deleting a Repair Order first cancels it (see `business-rules.md`, rule R-14). Part lines are deleted with the order because the part line's reference to the order is declared to cascade.

### 1.4 Complete field table

#### 1.4.1 Identification and progress

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repair Reference (`name`) | text | The human-readable reference of the repair. Required. Default at creation time is the literal text `New`; the creation routine immediately replaces it with the next value of the numbering sequence attached to the chosen operation type (see section 1.6.1). Read-only in the user interface. Not carried into a duplicate — a duplicate draws a fresh number. Indexed for partial-text search. Rewritten when the operation type is changed on a repair that is neither cancelled nor repaired (see section 1.6.2). |
| Company (`company_id`) | reference to one Company | The company that owns the repair. Required. Default is the current company. Read-only. Indexed. Deletion of the company is restricted while repairs reference it. Every company-scoped reference on the order — customer, product, operation type, all six locations, lot, part lines, sales order — is validated against this company automatically. |
| Status (`state`) | selection | The lifecycle state. Values and labels: `draft` "New"; `confirmed` "Confirmed"; `under_repair` "Under Repair"; `done` "Repaired"; `cancel` "Cancelled". Required in practice; default `draft`. Read-only — changed only through the transition operations of `state-machines.md`. Changes are recorded in the discussion thread. Indexed. Not carried into a duplicate: a duplicate starts at `draft`. |
| Priority (`priority`) | selection | Values and labels: `0` "Normal"; `1` "Urgent". Default `0`. Drives the default ordering. |
| Responsible (`user_id`) | reference to one User | The user accountable for the repair. Default is the acting user. Restricted to users of the order's company. Deletion of the user sets this field empty. Also used, together with the company, to pick the default operation type (see section 1.5.1). |
| Customer (`partner_id`) | reference to one Partner | The party for whom the repair is performed, to whom the quotation is addressed and to whom the repaired item is delivered. Computed from the originating return transfer (see section 1.5.2), stored, and freely overridable. Indexed. Restricted to partners visible to the order's company. Deletion of the partner sets this field empty. Changing the customer changes the defaults offered on dependent fields. |
| Tags (`tag_ids`) | collection of Repair Tag | Free classification labels. No constraint. |
| Internal Notes (`internal_notes`) | rich text | Free text for the technician. Not printed on the customer-facing document. |
| Properties (`repair_properties`) | property bag | Ad-hoc fields whose definition is held on the operation type in `repair_properties_definition`. Carried into a duplicate. Changing the operation type changes which properties exist. |
| Scheduled Date (`schedule_date`) | date and time | When the repair is planned to be carried out. Required. Default is the moment of creation. Indexed. Not carried into a duplicate. Writing this field pushes the same value onto the date of every part line and onto the generated product move, for those moves that are neither done nor cancelled (see section 1.6.2). |
| Date Category (`search_date_category`) | selection | Not stored; exists only to offer date buckets in the search panel. Values and labels: `before` "Before"; `yesterday` "Yesterday"; `today` "Today"; `day_1` "Tomorrow"; `day_2` "The day after tomorrow"; `after` "After". Read-only. Searching on it translates into a range condition on the scheduled date; the translation is specified in `calculations.md`, section 12. |

#### 1.4.2 The product to repair

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product to Repair (`product_id`) | reference to one Product Variant | The item being restored. Restricted to goods (products whose type is `consu`, that is, physical goods as opposed to services), belonging to the order's company or to no company, and — when the repair was raised from a return transfer — to the products that actually appear on that transfer or that are the transfer's own product. Restricted to products visible to the order's company. Deletion of the product sets this field empty. May be left empty: a repair with no product to repair is legal and produces no product move at completion. |
| Product Quantity (`product_qty`) | decimal | How many units of the product are being repaired. Computed (see section 1.5.3), stored, overridable. Default `1.0`. Decimal precision follows the *Product Unit* precision setting. Forced to `1.0` whenever the product is changed on a repair whose product is tracked by unique serial number. |
| Allowed Units (`allowed_uom_ids`) | collection of Unit of Measure | Not stored. The units the user may pick for the repair: the product's reference unit, plus every additional unit declared on the product, plus every unit used by the product's vendor price entries. Read-only; exists only to bound the unit selector. |
| Unit (`product_uom`) | reference to one Unit of Measure | The unit in which the repair quantity is expressed. Computed (see section 1.5.4), stored, precomputed on creation, overridable. Restricted to the allowed units above. Deletion of the unit sets this field empty. |
| Lot/Serial (`lot_id`) | reference to one Lot or Serial Number | The specific batch or unit being repaired. Computed (see section 1.5.5), stored, overridable. Restricted to the allowed lots (see below) and to lots visible to the order's company. Deletion of the lot sets this field empty. Not carried into a duplicate. Required at completion when the product is tracked (see `business-rules.md`, rule R-08). |
| Allowed Lots (`allowed_lot_ids`) | collection of Lot or Serial Number | Not stored; read-only. The lots the user may pick: all lots of the chosen product, further narrowed — when the repair was raised from a return transfer — to the lots that appear on that transfer's moves. |
| Product Tracking (`tracking`) | selection | Mirrors the tracking mode of the chosen product: `none` (no tracking), `lot` (tracked by batch) or `serial` (tracked by unique serial number). Not stored. Writable, which writes through to the product. Used by the user interface to show or hide the lot selector and to force the quantity to one. |
| Inventory Move (`move_id`) | reference to one Stock Move | The single Stock Move that carries the product to repair from its source to its destination location. Empty until completion; created by the completion routine (see `workflows.md`, section 4). Read-only. Changes are recorded in the discussion thread. Not carried into a duplicate. Restricted to moves of the order's company. Deletion of the move sets this field empty. This move is deliberately *not* part of the part-line collection, because that collection is filtered to lines that carry a part kind and this move carries none. |

#### 1.4.3 Operation type and the six locations

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Operation Type (`picking_type_id`) | reference to one Operation Type | The repair operation type that supplies the numbering sequence, the default locations and the property definition. Required. Computed from the company and responsible user (see section 1.5.1), stored, precomputed, overridable. Carried into a duplicate. Restricted to operation types whose code is `repair_operation` and whose company is the order's company. Restricted to operation types visible to the order's company. Indexed. Deletion is restricted while repairs reference it. Changing it on a live repair renumbers the order and re-reserves its parts (see section 1.6.2). |
| Operation Type Selector Visible (`picking_type_visible`) | true or false | Not stored; read-only. True when the order's company owns more than one repair operation type, in which case the selector is shown to the user; false when there is only one, in which case it is hidden. |
| Component Source Location (`location_id`) | reference to one Location | Where the parts that will be added come *from*. Required. Computed as the operation type's default source location, stored, overridable, precomputed. Indexed. Restricted to locations visible to the order's company. Deletion is restricted while repairs reference it. |
| Product Source Location (`product_location_src_id`) | reference to one Location | Where the product to repair is taken *from* at completion. Required. Computed as the operation type's default product source location, stored, overridable, precomputed. Indexed. Restricted to the order's company. Deletion restricted. |
| Product Destination Location (`product_location_dest_id`) | reference to one Location | Where the repaired product is placed at completion. Required. Computed as the operation type's default product destination location, stored, overridable, precomputed. Indexed. Restricted to the order's company. Deletion restricted. |
| Added Parts Destination Location (`location_dest_id`) | reference to one Location | Where added parts go: the virtual location that represents the item under repair. Required. A mirror of the operation type's default destination location, stored, **read-only** — it cannot be overridden on the order, only on the operation type. Indexed. Restricted to the order's company. Precomputed. |
| Removed Parts Destination Location (`parts_location_id`) | reference to one Location | Where removed parts go: by default the inventory-loss location, so that removing a part writes it off. Required. A mirror of the operation type's default removal destination, stored, **read-only** on the order. Indexed. Restricted to the order's company. Precomputed. Not carried into a duplicate. |
| Recycled Parts Destination Location (`recycle_location_id`) | reference to one Location | Where recycled parts go: by default the warehouse stock location, so that recycling a part puts it back into inventory. Required. Computed as the operation type's default recycle destination, stored, **overridable**, precomputed. Indexed. Restricted to the order's company. Deletion restricted. |

The asymmetry is deliberate and must be reproduced exactly: of the three part destinations, only the recycle destination and the component source can be overridden per order; the added-parts destination and the removed-parts destination are fixed by the operation type.

#### 1.4.4 Parts

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Parts (`move_ids`) | collection of Stock Move | The part lines. The inverse reference on the Stock Move is `repair_id`. The collection is filtered to moves whose part kind is set, which excludes the product move created at completion. Carried into a duplicate (the duplicate's lines are fresh draft moves with no sales-order link). Restricted to moves of the order's company. |
| Component Status (`parts_availability`) | text | Not stored; read-only. A short human-readable availability statement: "Available", "Not Available", or "Exp " followed by the formatted expected date. Computed by the algorithm of `calculations.md`, section 3. Empty unless the order is confirmed or under repair. |
| Component Status Code (`parts_availability_state`) | selection | Not stored; read-only. The machine-readable counterpart of the text above. Values and labels: `available` "Available"; `expected` "Expected"; `late` "Late". Empty unless the order is confirmed or under repair. Drives the colour of the status badge. |
| All Parts are available (`is_parts_available`) | true or false | Stored; computed from the component status code; default false. True exactly when the code is `available`. Stored so that the operation-type dashboard can count ready repairs without recomputing availability. |
| Any Part is late (`is_parts_late`) | true or false | Stored; computed from the component status code; default false. True exactly when the code is `late`. |
| Incomplete Parts Present (`has_uncomplete_moves`) | true or false | Not stored; read-only. True when at least one part line has recorded a quantity strictly smaller than its demand, compared in the line's unit at that unit's precision. Used to decide whether ending the repair must ask the user for confirmation. |
| Unreserve Allowed (`unreserve_visible`) | true or false | Not stored; read-only. True when the order is neither new, repaired nor cancelled **and** at least one detail line of at least one part line carries a non-zero reserved quantity. |
| Reserve Allowed (`reserve_visible`) | true or false | Not stored; read-only. True when the order is confirmed or under repair **and** at least one part line is not picked, has a non-zero demand, and is in the `confirmed` or `partially_available` move state. |

#### 1.4.5 Links to other documents

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sale Order (`sale_order_id`) | reference to one Sales Order | The order on which this repair is billed. Read-only — set either by the quotation-creation operation (which creates the order) or by the repair-creation routine driven from a confirmed sales order line. Indexed, with the index skipping empty values. Not carried into a duplicate. Restricted to orders of the repair's company. Deletion of the order sets this field empty. |
| Sale Order Line (`sale_order_line_id`) | reference to one Sales Order Line | The specific line whose product is configured as a repair service and whose confirmation produced this repair. Read-only. Not carried into a duplicate. Restricted to the repair's company. Deletion of the line sets this field empty. |
| Repair Request (`repair_request`) | long text | Not stored; read-only. Mirrors the description of the originating sales order line, so the technician sees what the customer asked for. Not carried into a duplicate. |
| Transfer (`picking_id`) | reference to one Transfer | The return transfer through which the broken item came back. Restricted to transfers that are themselves returns (their return-origin reference is set) and, when a product is already chosen on the repair, to transfers carrying that product. Indexed, skipping empty values. Not carried into a duplicate. Restricted to the repair's company. Deletion of the transfer sets this field empty. |
| Transfer Products (`picking_product_ids`) | collection of Product Variant | Not stored; read-only. Every product appearing on the return transfer's moves. Bounds the product selector. |
| Transfer Product (`picking_product_id`) | reference to one Product Variant | Not stored; read-only. Mirrors the return transfer's own headline product, used as an additional permitted value in the product selector. Not carried into a duplicate. |
| References (`reference_ids`) | collection of Stock Reference | The cross-document reference tokens shared by every move this order generates, so that downstream documents (purchase orders, manufacturing orders) can be traced back to the repair. Relation name `stock_reference_repair_rel`, with `repair_id` pointing at the repair and `reference_id` at the reference. Not carried into a duplicate. One reference record carrying the repair's own number is created with the order. |
| Count of Manufacturing Orders (`production_count`) | whole number | Present only when the manufacturing capability is installed. Not stored; read-only. The number of distinct manufacturing orders reachable through the order's references. Visible only to users of the manufacturing user group. |
| Count of Purchase Orders (`purchase_count`) | whole number | Present only when the purchasing capability is installed. Not stored; read-only. The number of distinct purchase orders generated by the order's part lines. Visible only to users of the purchasing user group. |

### 1.5 Computed field rules

#### 1.5.1 Operation type

Depends on the company.

1. Build a map from (company, user) pairs to repair operation types by the following procedure. For each distinct (company, responsible user) pair present in the records being computed, read the user's default warehouse *as seen from that company*. If that warehouse exists and has a repair operation type, record the pair with that operation type.
2. Collect the companies for which step 1 found nothing.
3. For those companies, search for every operation type whose code is `repair_operation` and whose warehouse belongs to one of them, and record each under the pair (company, *no user*), keeping the first one found per company.
4. For each record, take the operation type recorded for its own (company, responsible user) pair; if there is none, take the one recorded for (company, *no user*).

When the map is asked for a default on a brand-new record — where no company or user is yet bound to a record — the procedure short-circuits: it reads the acting user's default warehouse in the current company and returns that warehouse's repair operation type if there is one.

#### 1.5.2 Customer

Depends on the return transfer. The customer is set to the return transfer's contact. When no return transfer is linked, the customer is set empty by the computation — but because the field is stored and overridable, a value typed by the user or supplied by the creating routine survives until the return transfer changes.

#### 1.5.3 Product quantity

Depends on the product, the return transfer and the lot.

1. If no return transfer is linked, the quantity is `1.0`.
2. If a return transfer is linked and the product's tracking mode is `serial` or `lot` and a lot is chosen: sum the recorded quantity of every detail line of the transfer whose product is the repair's product and whose lot is the repair's lot.
3. Otherwise (a return transfer is linked but the product is untracked or no lot is chosen): sum the recorded quantity of every move of the transfer whose product is the repair's product.

#### 1.5.4 Unit

Depends on the product and the product's reference unit. If no product is chosen, the unit is cleared. If a product is chosen and no unit is yet set, the unit becomes the product's reference unit. An already-set unit is never overwritten by the computation.

#### 1.5.5 Lot or serial number

Depends on the product, the lot, the lot's product and the return transfer.

1. If a product and a lot are both set and the lot belongs to a different product, or if no product is set, clear the lot.
2. Otherwise, if the linked return transfer's moves reference exactly one lot in total, set the lot to that lot.
3. Otherwise leave the lot as it is.

#### 1.5.6 Allowed units

Depends on the product, its reference unit, its additional units, its vendor price entries and the units on those entries. The result is the union of the product's reference unit, the product's additional units, and the units named on the product's vendor price entries.

#### 1.5.7 Allowed lots

Depends on the product, the company, the return transfer, the transfer's moves and the lots on those moves. Search all lots of the chosen product; when a return transfer is linked, intersect that set with the lots appearing on the transfer's moves.

#### 1.5.8 Parts availability

Specified as a numbered algorithm in `calculations.md`, section 3.

#### 1.5.9 Readiness and lateness booleans

Both are set to false for every record first. Then, for each record whose component status code is non-empty: if the code is `available`, the readiness boolean becomes true; if the code is `late`, the lateness boolean becomes true. A record whose code is `expected` therefore has both booleans false.

#### 1.5.10 Incomplete parts present

For each order, true when any part line satisfies: the line has a unit, and comparing the recorded quantity against the demanded quantity at that unit's rounding precision yields "less than".

#### 1.5.11 Unreserve and reserve visibility

Both computed together; see the field table in section 1.4.4 for the two conditions.

#### 1.5.12 Operation type selector visibility

Group the repair operation types by company, counting them. For each order, the selector is visible when the count for the order's company exceeds one.

### 1.6 Creation, modification and deletion behaviour

#### 1.6.1 Creation

For each set of values submitted:

1. Determine the operation type: the value supplied, or — when absent — the default computed by section 1.5.1. Write the determined operation type into the values if it was absent.
2. If no reference is supplied, or the supplied reference is the literal text `New`, draw the next value from the numbering sequence attached to that operation type and use it as the reference.
3. If no references collection is supplied, create one Stock Reference record whose name is the reference determined in step 2, and link it.
4. Create the record.

When the manufacturing capability is installed, the creation routine additionally runs the kit explosion pass over the new orders' part lines (see `calculations.md`, section 9).

A default supplied through the surrounding context is honoured for two fields that are not ordinary defaults: when the context carries a repair transfer identifier and the transfer field is being defaulted, it becomes the return transfer; when the context carries a repair lot identifier and the lot field is being defaulted, it becomes the lot. This is how the "create a repair" button on a return transfer and the "repairs of this lot" action on a lot record pre-fill the form.

#### 1.6.2 Modification

Writing to a Repair Order has the following ordered side effects.

1. **Operation type change.** Before the write is applied: for each order that is neither cancelled nor repaired, if the new operation type differs from the current one, the order's reference is redrawn from the new operation type's numbering sequence, and every part line of the order is collected for re-reservation.
2. The write is applied.
3. **Serial-tracked product change.** If the product was among the written fields and the (single) order's tracking mode is `serial`, the product quantity is forced to `1.0`.
4. **Location change.** For each order, if any of the four order-level location fields that drive part-line locations was written — the component source location, the product destination location, the removed-parts destination location or the recycled-parts destination location — every part line's source and destination location is recomputed from its part kind through the mapping of `calculations.md`, section 2.
5. **Schedule change.** If the scheduled date was written, the same date is written onto the generated product move and onto every part line, restricted to those that are neither done nor cancelled.
6. **Warranty change.** If the warranty flag was written, the prices of the sales-order lines backing the *add* part lines are rewritten (see `calculations.md`, section 6).
7. **Re-reservation.** The part lines collected in step 1 are unreserved. Of those, the ones that are now in the `confirmed` or `partially_available` move state **and** that either bypass reservation entirely, or belong to an operation type that reserves at confirmation, or carry a reservation date that has already arrived, are re-reserved.

When the manufacturing capability is installed, every write is followed by the kit explosion pass.

#### 1.6.3 Deletion

Deletion is guarded: before a Repair Order is deleted, every order in the set that is not already cancelled is cancelled through the ordinary cancellation operation, with all its side effects (see `state-machines.md`, transition T-07). The deletion then proceeds. Part lines are removed with the order because their reference to it cascades.

### 1.7 Product-catalog bindings

The Repair Order exposes the following to the shared product-catalog screen:

- The catalog is restricted to goods (products whose type is `consu`).
- The catalog shows stock figures for each product.
- The price shown for a product is its list price.
- The existing lines for a product are its part lines carrying that product.
- Setting a quantity for a product that already has a part line writes the quantity onto that line, or deletes the line when the quantity is zero. Setting a positive quantity for a product with no part line creates a new part line with kind *add*, the order's component source location as source, the order's added-parts destination location as destination, and the typed quantity. The operation returns the product's list price.
- The catalog search screen is replaced by a repair-specific one, and the products already present on the order are searchable through a dedicated flag (see `interfaces.md`).
- When the manufacturing capability is installed, the catalog is additionally seeded with the components of the bill of materials found for the product to repair, and a corresponding filter is switched on by default.

### 1.8 Multi-company behaviour

The company is required, read-only after creation and defaulted to the current company. A record rule restricts visibility to repairs whose company is among the acting user's allowed companies. Every company-scoped reference is checked against the order's company automatically, both on the order and on its part lines; a mismatch raises the platform's company-consistency error.

---

## 2. Repair part line (Stock Move extension)

A repair part line is not a separate entity: it is a **Stock Move** (`stock.move`, table `stock_move`) that carries a reference to a Repair Order and a part kind. The full Stock Move field set, state machine and execution semantics are specified in `../inventory-operations/`. This section specifies only what the repair capability adds and changes.

### 2.1 Added fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repair Order (`repair_id`) | reference to one Repair Order | The order this move belongs to. Indexed, skipping empty values. Not carried into a duplicate of the move. Restricted to the move's company. Deletion of the repair deletes the move (cascade). |
| Type (`repair_line_type`) | selection | The part kind. Values and labels: `add` "Add"; `remove` "Remove"; `recycle` "Recycle". Stored. Indexed. Empty on the product move created at completion, which is how that move is excluded from the part collection. |

### 2.2 Changed computations

| Computation | Repair behaviour |
|---|---|
| Forecast information | For part lines whose kind is *remove* or *recycle*, the forecast availability is set equal to the move's quantity expressed in the product's reference unit, and the forecast expected date is cleared. These parts come out of the item being repaired, not out of stock, so they are always "available". For part lines of kind *add*, and for moves with no kind, the ordinary forecast computation of the inventory domain applies. |
| Operation type | A move attached to a repair takes the repair's operation type. Other moves keep the ordinary computation. |
| Source location | A move attached to a repair **and** carrying a part kind takes its source location from the repair through the part-kind mapping of `calculations.md`, section 2. Other moves keep the ordinary computation. |
| Destination location | Same rule, for the destination. |
| Reference | A move attached to a repair that has a reference takes the repair's reference as its own document reference. |
| Source document | The repair is returned as the move's source document in preference to whatever the inventory domain would return. |
| Should be assigned | A move attached to a repair is never picked up by the inventory domain's automatic assignment sweep; reservation of repair parts is driven from the repair. |
| Splitting | A move attached to a repair is never split. This is what allows a repair to be completed with part lines whose recorded quantity is below their demand without leaving a residual move behind. |
| Is consuming | A move attached to a repair with part kind *add* counts as consuming, in addition to whatever the inventory domain already treats as consuming. |
| Lot shown on the invoice | A detail line whose move carries any part kind is eligible to have its lot printed on the invoice. |
| Details screen | For a part line of kind *recycle*, the details screen hides the on-hand column and shows the destination-location column. |

### 2.3 Creation of a part line

For each set of values submitted:

1. If the values name a repair **and** name a part kind, set the move's origin document text to the repair's reference.
2. Create the moves.
3. For every created move that names a repair: link the repair's cross-document references onto the move, and set the move's operation type to the repair's operation type.
4. Partition the repair-bound moves into those that are in the `draft` move state whose repair is confirmed or under repair — call these the *late-added* moves — and all the rest.
5. For the late-added moves: validate company consistency; adjust the procurement method with the repair operation code (this decides whether the part will be taken from stock or procured to order); confirm them; and trigger the procurement scheduler on the result.
6. For the union of the confirmed late-added moves and the other repair-bound moves, create the backing sales-order lines (see section 2.5).
7. Return the union of that set with the non-repair moves.

#### 2.3.1 Kit explosion on creation

When the manufacturing capability is installed, creating or writing a Repair Order runs the explosion pass described in `calculations.md`, section 9: any part line whose product has a kit-type bill of materials is replaced by part lines for the bill's components, each inheriting the original line's part kind, unit price, source location and destination location, and each created in the `draft` move state. Components whose product is a service are skipped.

### 2.4 Modification of a part line

After the write is applied, for each repair-bound move:

- If the move has no backing sales-order line, the write did not itself set one, and the part kind is *add*, the move is collected for line creation.
- If the move has a backing sales-order line and the write touched either the part kind or the demanded quantity, the move is collected for line update.

Collected updates run first, then collected creations. The update rule is in section 2.6.

### 2.5 Creating the backing sales-order line

For each move in the set: skip it when it already has a backing line, when its part kind is not *add*, or when its repair has no sales order. For the remainder, prepare one line with the following values and create them all at once.

| Sales order line value | Source |
|---|---|
| Order | The repair's sales order. |
| Product | The move's product. |
| Ordered quantity | The move's demanded quantity when the repair is not yet repaired; the move's recorded quantity when the repair is already repaired. |
| Unit | The move's unit. |
| Moves | The move itself is linked to the line. |
| Delivered quantity | The move's recorded quantity when the move is in the `done` state; otherwise zero. |
| Unit price | Zero when the repair is under warranty. Otherwise the move's unit price, when the move carries one; when it does not, the field is omitted and the sales domain's ordinary price derivation applies. |

### 2.6 Updating and clearing the backing sales-order line

- **Clearing.** Every move in the set that belongs to a repair and has a backing line has that line's ordered quantity set to zero. This is the mechanism by which cancelling a repair, cancelling a part line, deleting a part line, or changing a part line's kind away from *add* removes the charge from the quotation without deleting the line.
- **Updating.** For each backing line reached from a move whose kind is still *add*, the line's ordered quantity is recomputed as the sum of the demanded quantities of every move linked to that line. This makes a line that backs several merged moves carry their total.

Both `_action_cancel` on a repair-bound move and deletion of a repair-bound move run the clearing pass first.

### 2.7 Deletion of a part line

Deleting a part line first cancels it if it is attached to a repair (which itself clears the backing sales-order line's quantity), then applies the inventory domain's ordinary "only draft or cancelled moves may be deleted" guard.

---

## 3. Repair Tag

**Repair Tag** (`repair.tags`, table `repair_tags`).

### 3.1 Purpose

A free classification label attachable to Repair Orders — for example "Water damage", "Screen", "Out of warranty claim". Tags carry no behaviour; they exist for filtering, grouping and colour-coding.

### 3.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Tag Name (`name`) | text | The label. Required. |
| Color Index (`color`) | whole number | The colour slot used to render the tag. Default is a pseudo-random whole number drawn uniformly from the range one to eleven inclusive, computed once when the record is created. |

### 3.3 Rules

- **Uniqueness**: the tag name is unique across the whole installation. Violating it produces the message **"Tag name already exists!"**.
- **Ordering**: by identifier.
- **Display name**: the tag name.
- **Multi-company**: tags are not company-scoped; one tag set is shared by all companies.
- **Archival**: tags have no archive flag; they are deleted when no longer wanted.

---

## 4. Insufficient Repair Quantity Warning

**Insufficient Repair Quantity Warning** (`stock.warn.insufficient.qty.repair`, table `stock_warn_insufficient_qty_repair`). A transient record: it exists only for the duration of one dialogue and is discarded afterwards.

### 4.1 Purpose

When a user confirms a Repair Order whose product to repair is a storable good that is not on hand in sufficient quantity at its product source location, the system does not refuse. It shows this dialogue, which lists the on-hand records for that product and location and asks the user to confirm anyway.

### 4.2 Field table

This record specialises the shared insufficient-quantity warning of the inventory domain.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | reference to one Product Variant | The product that is short. Required. Deletion of the product deletes the dialogue record. |
| Location (`location_id`) | reference to one Location | The location that is short. Required. Deletion of the location deletes the dialogue record. |
| Quantity (`quantity`) | decimal | The quantity the repair needs, expressed in the product's reference unit. Required. |
| Unit Name (`product_uom_name`) | text | The display name of the unit in which the quantity is expressed. Required. |
| On-hand Records (`quant_ids`) | collection of Stock Quantity | Not stored; read-only. The on-hand records for the product and location, shown to the user so they can see where the goods actually are. |
| Repair (`repair_id`) | reference to one Repair Order | The order being confirmed. Deletion of the repair sets this field empty. |

### 4.3 Behaviour

- The company used for the shared warning's own bookkeeping is the repair's company.
- Confirming the dialogue clears the surrounding context of any defaults that might pollute subsequently created records, then runs the repair's confirmation routine (see `state-machines.md`, transition T-01).
- Dismissing the dialogue leaves the repair in the `draft` state.

---

## 5. Operation Type (repair extension)

**Operation Type** (`stock.picking.type`, table `stock_picking_type`). The base entity is specified in `../inventory-operations/`. The repair capability adds one selection value and eight fields.

### 5.1 The repair code

The operation-type code gains the value `repair_operation` with the label "Repair". Deleting the repair capability cascades: operation types carrying this code are removed with it.

An operation type whose code is `repair_operation` behaves differently in four base computations:

| Base computation | Repair behaviour |
|---|---|
| Default source location | The warehouse's stock location. |
| Default destination location | The lowest-numbered location of usage *production* belonging to the same company. This is a virtual location: sending an added part there means the part has been consumed into the item under repair. |
| Aggregated dashboard dates | Instead of transfers, the confirmed Repair Orders of this operation type are aggregated by scheduled date under the label "Confirmed". |
| Dashboard graph action | Clicking the graph opens the repair graph action rather than the transfer graph action. |

### 5.2 Added fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product Source Location (`default_product_location_src_id`) | reference to one Location | Default source of the product to repair for orders of this type. Computed from the code: for repair operation types it becomes the warehouse's stock location. Stored, precomputed, overridable. Restricted to the operation type's company. |
| Product Destination Location (`default_product_location_dest_id`) | reference to one Location | Default destination of the repaired product. Computed identically: the warehouse's stock location. Stored, precomputed, overridable. Restricted to the company. |
| Remove Destination Location (`default_remove_location_dest_id`) | reference to one Location | Default destination for removed parts. Computed from the code: the lowest-numbered location of usage *inventory* — the inventory-loss location — belonging to the operation type's company or to no company. Stored, precomputed, overridable. Restricted to the company. |
| Recycle Destination Location (`default_recycle_location_dest_id`) | reference to one Location | Default destination for recycled parts. Computed from the code: the warehouse's stock location. Stored, precomputed, overridable. Restricted to the company. |
| Repair Properties (`repair_properties_definition`) | property definition | The definition of the ad-hoc property fields offered on Repair Orders of this type. |
| Number of Repair Orders Confirmed (`count_repair_confirmed`) | whole number | Not stored; see section 5.3. |
| Number of Repair Orders Under Repair (`count_repair_under_repair`) | whole number | Not stored; see section 5.3. |
| Number of Repair Orders to Process (`count_repair_ready`) | whole number | Not stored; see section 5.3. |
| Number of Late Repair Orders (`count_repair_late`) | whole number | Not stored; see section 5.3. |

### 5.3 The four counters

All four are computed together and are false (zero) for every operation type whose code is not `repair_operation`.

1. Group the Repair Orders whose operation type is among the repair operation types being computed and whose state is `confirmed` or `under_repair`, by operation type, readiness boolean and state, counting.
2. Separately, count the Repair Orders whose operation type is among them, whose state is `confirmed`, and which are either scheduled before today or flagged as having a late part.
3. For each operation type: the *confirmed* counter is the total of the groups whose state is `confirmed`; the *under repair* counter is the total of the groups whose state is `under_repair`; the *ready* counter is the total of the groups whose state is `confirmed` **and** whose readiness boolean is true; the *late* counter is the count from step 2, defaulting to zero.

Note that an order that is under repair is never counted as ready, however available its parts are.

---

## 6. Warehouse (repair extension)

**Warehouse** (`stock.warehouse`, table `stock_warehouse`). The base entity is specified in `../inventory-operations/`.

### 6.1 Added fields

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repair Operation Type (`repair_type_id`) | reference to one Operation Type | The warehouse's repair operation type. Restricted to the warehouse's company. Not carried into a duplicate of the warehouse. |
| Repair Make-to-Order Rule (`repair_mto_pull_id`) | reference to one Stock Rule | The rule that procures repair components to order. Not carried into a duplicate. |

### 6.2 What a warehouse creates for repair

When a warehouse is created or updated, the repair capability contributes:

- **A numbering sequence** named "*warehouse name* Sequence repair", with prefix formed as the warehouse short code, then a slash, then the repair operation type's sequence code (falling back to the literal `RO` when none is set), then a slash; padded to five digits; owned by the warehouse's company.
- **A repair operation type** named "Repairs", with code `repair_operation`, default source location the warehouse stock location, default destination location the company's production location, default removal destination the inventory-loss location, default recycle destination the warehouse stock location, sequence code `RO`, both lot options enabled (new lots may be created and existing lots may be used), owned by the warehouse's company, and placed one slot after the last operation type the base warehouse creates. The warehouse's operation-type sequence counter advances by two rather than one to leave room for it.
- **An update pass** that keeps the repair operation type's active flag in step with the warehouse's, and sets its barcode to the warehouse short code with spaces removed and letters upper-cased, followed by the letters `RO`.
- **A global route rule** — the repair make-to-order rule — with procurement method *make to order*, action *pull*, automatic mode *manual*, on the installation-wide replenish-on-order route (created if missing, named "Replenish on Order (MTO)"), from the repair operation type's default source location to its default destination location, through the repair operation type, in the warehouse's company. Its name is formatted by the warehouse's standard rule-naming convention from the stock location, the production location and the token `MTO`, and it is activated. The rule depends on the repair operation type existing.
- **A production location check**: when the warehouse's missing locations are created, if the warehouse's company has no location of usage *production*, one is created for it.

### 6.3 Failure conditions

- If no location of usage *inventory* can be found for the warehouse's company or for no company when the repair operation type is being prepared, the operation fails with **"No location of type Inventory Loss found"**.
- If no location of usage *production* can be found for the warehouse's company when the production location is requested, the operation fails with **"Can't find any production location."**.

---

## 7. Transfer, Lot, Sales Order, Sales Order Line, Product (repair extensions)

### 7.1 Transfer

**Transfer** (`stock.picking`, table `stock_picking`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repairs (`repair_ids`) | collection of Repair Order | The repairs raised from this transfer. The inverse reference on the repair is `picking_id`. |
| Number of repairs linked to this picking (`nbr_repairs`) | whole number | Not stored; the length of the collection above. |

### 7.2 Lot or Serial Number

**Lot or Serial Number** (`stock.lot`, table `stock_lot`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repair Orders (`repair_line_ids`) | collection of Repair Order | Not stored; read-only. The repairs in which this lot was used **as a part** — that is, the repairs reachable from completed part lines whose detail lines carry this lot. |
| Repair part count (`repair_part_count`) | whole number | Not stored. The length of the collection above. |
| In repair count (`in_repair_count`) | whole number | Not stored. The number of Repair Orders whose *product to repair* carries this lot and whose state is neither `done` nor `cancel`. |
| Repaired count (`repaired_count`) | whole number | Not stored. The number of Repair Orders whose *product to repair* carries this lot and whose state is `done`. |

The first two are computed by searching every Stock Move that is attached to a repair, carries a part kind, has a detail line referencing one of the lots being computed, and is in the `done` state; each such move contributes its repair to every lot it touches.

A lot's creation is additionally guarded: when a lot is created from within a Repair Order form — signalled by an active-repair marker in the surrounding context — and that repair's operation type does not allow creating new lots, the creation fails with **"You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers"."**

### 7.3 Sales Order

**Sales Order** (`sale.order`, table `sale_order`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Repair Order (`repair_order_ids`) | collection of Repair Order | The repairs linked to this order. Inverse reference `sale_order_id`. Visible only to users of the inventory user group. |
| Repair Order(s) (`repair_count`) | whole number | Not stored; the length of the collection above. Visible only to users of the inventory user group. |

Two lifecycle hooks are added:

- **Confirmation.** After the order's ordinary confirmation completes, the repair-creation pass runs over its lines (see section 7.4).
- **Cancellation.** After the order's ordinary cancellation completes, the repair-cancellation pass runs over its lines (see section 7.4).

### 7.4 Sales Order Line

**Sales Order Line** (`sale.order.line`, table `sale_order_line`).

The repair capability changes five behaviours.

1. **Delivered quantity.** When preparing delivered quantities, a line is handled specially if exactly one of its moves is attached to a repair and is in the `done` state: the delivered quantity of that line is the recorded quantity of that move. Lines not matching that test fall through to the sales domain's ordinary derivation. The test requires *exactly one* such move; a line backed by several completed repair moves falls through.
2. **Creation.** After lines are created, every created line whose order is in the `sale` or `done` state runs the repair-creation pass.
3. **Quantity change.** When the ordered quantity is written: the previous quantities are captured, the write is applied, and then for each line whose order is in the `sale` or `done` state and which has a product: if the previous quantity was zero or less and the new quantity is greater than zero, the repair-creation pass runs; if the previous quantity was greater than zero and the new quantity is zero or less, the repair-cancellation pass runs. Comparisons are made at the precision of the line's unit.
4. **Stock rule launch.** Lines whose moves are attached to a repair are excluded from the pass that launches procurement from a sales order, because the repair already created and confirmed those moves. Only lines with no repair-bound move go through the ordinary pass.
5. **Valued moves.** A line reports that it has valued moves only when the base rule says so **and** none of its moves is attached to a repair. This prevents the sales-margin and anglo-saxon-recognition machinery from double-counting a repair part whose valuation has already been recognised by the repair itself.

**The repair-creation pass.** For each line in the set:

1. If the line's order already has a repair whose originating line is this line, and the line's ordered quantity is greater than zero: take the repairs of that order whose originating line is this line and whose state is `cancel`; reset them to draft; confirm them; and move on to the next line. (This is the "re-open a cancelled repair when the line comes back to life" path.)
2. Otherwise, skip the line when its product's service-tracking setting is not `repair`, or when the line already has a repair-bound move, or when its ordered quantity is zero or less.
3. For each surviving line, prepare a Repair Order with state `confirmed`, the order's customer, the order as sales order, the line as originating line, and the operation type taken from the order's warehouse's repair operation type.
4. Create the prepared orders with elevated rights, so that a salesperson without inventory rights can still confirm an order that spawns repairs.

Note that quantity is not multiplied: a line with ordered quantity five produces exactly one Repair Order, not five.

**The repair-cancellation pass.** For each line, collect the repairs of the line's order whose originating line is this line and whose state is not `done`; cancel them all through the ordinary cancellation operation.

When the point-of-sale capability is installed, a further field is added: **Is linked to repair** (`is_repair_line`), not stored, true when any of the line's moves is attached to a repair. This field is shipped to the point-of-sale terminal, and the terminal's delivery-creation pass skips lines for which it is true, so that settling a repair-backed order at a terminal does not create a second delivery move for a part the repair already moved.

### 7.5 Product Template and Product Variant

**Product Template** (`product.template`, table `product_template`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Service Tracking (`service_tracking`) | selection | Gains the value `repair` with label "Repair Order". Deleting the repair capability resets affected products to the selector's default. The value is added to the set of saleable tracking types, so it is offered on saleable service products. A sales order line for a product carrying this value creates a Repair Order on confirmation. |

**Product Variant** (`product.product`, table `product_product`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| In this repair (`product_catalog_product_is_in_repair`) | true or false | Not stored; always false when read. It exists only to support a search condition: searching for products "in this repair" resolves the repair identified in the surrounding context and returns the products on its part lines. |

Two further behaviours change:

- **Returned serial counting.** When counting how many units of a serial-tracked product have come back, the count additionally includes detail lines whose move carries part kind *remove* or *recycle* and whose destination location is internal — that is, a part taken out of a repaired item and put back into stock counts as returned.
- **Unit change.** Changing a product's reference unit is refused when any Repair Order uses the product with a different unit. The operation fails with **"As other units of measure (ex : *the other unit*) than *the product unit* have already been used for this product, the change of unit of measure can not be done. If you want to change it, please archive the product and create a new one."** When every Repair Order already uses the product's own reference unit, those orders are rewritten to the new unit and the change proceeds.

### 7.6 Manufacturing Order and Purchase Order (repair extensions)

**Manufacturing Order** (`mrp.production`, table `mrp_production`) gains **Count of source repairs** (`repair_count`), not stored, the number of distinct repairs reachable through the order's downstream moves, visible only to users of the inventory user group.

**Purchase Order** (`purchase.order`, table `purchase_order`) gains **Count of source repairs** (`repair_count`), not stored, the number of distinct repairs reachable through the order lines' downstream moves, visible only to users of the inventory user group.

**Stock Reference** (`stock.reference`, table `stock_reference`) is the join that makes those counts possible: it carries a reference text, a set of Stock Moves (relation `stock_reference_move_rel`) and — through the repair capability — a set of Repair Orders (relation `stock_reference_repair_rel`); when the manufacturing capability is installed it also carries a set of Manufacturing Orders. Every move a repair creates is linked to the repair's reference records, so any downstream document that inherits the reference is reachable from the repair.

### 7.7 Traceability report (repair extension)

The traceability report of the inventory domain is extended so that a detail line whose move belongs to a repair reports the repair as its source document: the referenced entity becomes the Repair Order, the referenced identifier becomes the repair's identifier, and the displayed reference becomes the repair's reference. Furthermore, when the base report finds no upstream lines for a detail line, and the line's move belongs to a repair, the line's *consumed* links are used as the upstream set; and when the base report finds no downstream usage, and the move belongs to a repair, the line's *produced* links are used. This is what makes the product move created at completion the parent node of the parts consumed into it (see `calculations.md`, section 10).

### 7.8 Forecast report (repair extension)

Two changes:

- A move that belongs to a repair and carries a part kind contributes no reservation data to the forecast report, because repair parts are reserved by the repair rather than by a transfer.
- When a move is bound both to a Repair Order and to a Sales Order Line with part kind *add*, the sales-order side of the forecast is suppressed for that line, so the demand is counted once — as a repair — rather than twice.

---

## 8. Maintained Item (shared behaviour)

**Maintained Item** (`maintenance.mixin`) is an abstract behaviour with no table of its own. Anything that can be maintained mixes it in; in the shipped system, Equipment does.

### 8.1 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Company (`company_id`) | reference to one Company | The owning company. Default is the current company. Not required — a record with no company is visible to every company. Deletion of the company sets this field empty. |
| Effective Date (`effective_date`) | date | The date from which the item is considered to be in service. Required. Default is today in the acting user's time zone. This is the origin date of the mean-time-between-failures computation. |
| Maintenance Team (`maintenance_team_id`) | reference to one Maintenance Team | The team responsible. Computed from the company (see section 8.2), stored, overridable. Restricted to the item's company. Indexed, skipping empty values. Deletion of the team sets this field empty. |
| Technician (`technician_user_id`) | reference to one User | The person responsible. Changes are recorded in the discussion thread. Deletion of the user sets this field empty. |
| Maintenance (`maintenance_ids`) | collection of Maintenance Request | The requests raised against this item. The concrete entity must supply the inverse reference; Equipment supplies `equipment_id`. |
| Maintenance Count (`maintenance_count`) | whole number | Stored; computed. The total number of requests raised against this item. |
| Current Maintenance (`maintenance_open_count`) | whole number | Stored; computed. The number of requests raised against this item that are neither in a closing stage nor archived. |
| Expected MTBF (`expected_mtbf`) | whole number | The mean time between failures the operator expects, in days. Entered by hand; used for comparison against the measured value. Written in full: expected mean time between failures. |
| MTBF (`mtbf`) | whole number | Not stored; read-only. The measured mean time between failures, in days, computed from the closed corrective requests. Formula in `calculations.md`, section 14. |
| MTTR (`mttr`) | whole number | Not stored; read-only. The measured mean time to repair, in days, computed from the closed corrective requests. Formula in `calculations.md`, section 13. |
| Latest Failure Date (`latest_failure_date`) | date | Not stored; read-only. The latest request date among the closed corrective requests. |
| Estimated time before next failure (in days) (`estimated_next_failure`) | date | Not stored; read-only. The latest failure date advanced by the measured mean time between failures. Formula in `calculations.md`, section 15. |

### 8.2 Team computation

Depends on the company. For each record: if the currently chosen team has a company and that company differs from the record's company, the team is cleared. The computation never *assigns* a team; it only removes one that has become invalid. A team is assigned by hand, or — on a Maintenance Request — inherited from the equipment.

### 8.3 Effectiveness computation

All four effectiveness fields are computed together; they depend on the effective date and, through the request collection, on each request's stage, close date and request date. The algorithm and its worked examples are in `calculations.md`, sections 13 to 15.

### 8.4 Counter computation

Depends on the closing flag of each request's stage and on each request's archive flag. The total counter is the number of requests. The open counter is the number of requests whose stage is not a closing stage and which are not archived.

---

## 9. Equipment

**Equipment** (`maintenance.equipment`, table `maintenance_equipment`).

### 9.1 Purpose

Equipment is the register of the physical things the company maintains: machines on a production line, vehicles, tools, laptops, printers. An equipment record identifies the thing, records who supplied it and at what cost, records who uses it and who services it, holds its warranty and scrap dates, and accumulates the maintenance history from which its effectiveness measurements are derived.

### 9.2 Inherited behaviour

Equipment mixes in the Maintained Item behaviour of section 8, the discussion thread, and scheduled activities.

### 9.3 Ordering, display name, identity and archival

- **Default ordering**: by identifier.
- **Display name**: when a serial number is present, the equipment name, a slash, and the serial number — for example `Drill press/SN-4471`. When no serial number is present, the equipment name alone.
- **Uniqueness**: the serial number is unique across the whole installation. Violating it produces **"Another asset already exists with this serial number!"**. The equipment name is *not* unique.
- **Archival**: equipment carries an archive flag (`active`), default true. Archived equipment disappears from the ordinary lists but keeps its history.
- **Deletion**: deletion of a piece of equipment is restricted while any Maintenance Request references it, because the request's reference to the equipment is declared to restrict.

### 9.4 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Equipment Name (`name`) | text | The name of the thing. Required. Translatable. |
| Active (`active`) | true or false | The archive flag. Default true. |
| Serial Number (`serial_no`) | text | The manufacturer's serial number. Unique across the installation. Not carried into a duplicate. Part of the display name. |
| Model (`model`) | text | The model designation. |
| Equipment Category (`category_id`) | reference to one Equipment Category | The grouping. Changes are recorded in the discussion thread. Indexed, skipping empty values. Deletion of the category sets this field empty. The kanban grouping expands to show every category, including empty ones. Choosing a category proposes the category's responsible technician as the equipment's technician (an interactive proposal, not a stored computation). |
| Vendor (`partner_id`) | reference to one Partner | Who supplied the equipment. Restricted to the equipment's company. Deletion sets this field empty. |
| Vendor Reference (`partner_ref`) | text | The vendor's own reference for the item. |
| Owner (`owner_user_id`) | reference to one User | The user considered to own the equipment. Changes are recorded in the discussion thread. Indexed, skipping empty values. Not carried into a duplicate. When the people capability is installed this becomes a stored computed field (see section 9.7). Deletion of the user sets this field empty. |
| Technician (`technician_user_id`) | reference to one User | Inherited from the Maintained Item behaviour: the person who services this equipment. Tracked. |
| Maintenance Team (`maintenance_team_id`) | reference to one Maintenance Team | Inherited: the responsible team. |
| Assigned Date (`assign_date`) | date | When the equipment was handed over. Changes are recorded in the discussion thread. When the people capability is installed this becomes a stored computed field that is refreshed to today whenever the assignment changes, and it *is* carried into a duplicate. |
| Cost (`cost`) | decimal | The acquisition cost. Plain number: no currency is attached, so the figure is read in the company's own currency by convention. |
| Note (`note`) | rich text | Free description. |
| Warranty Expiration Date (`warranty_date`) | date | When the manufacturer's warranty runs out. |
| Scrap Date (`scrap_date`) | date | When the equipment was, or is to be, scrapped. Purely informational: setting it neither archives the record nor blocks new requests. |
| Color Index (`color`) | whole number | The colour slot used to render the card. |
| Effective Date (`effective_date`) | date | Inherited: required, default today. |
| Expected MTBF (`expected_mtbf`) | whole number | Inherited. |
| MTBF (`mtbf`), MTTR (`mttr`), Latest Failure Date (`latest_failure_date`), Estimated next failure (`estimated_next_failure`) | see section 8.1 | Inherited, not stored. |
| Maintenance (`maintenance_ids`) | collection of Maintenance Request | The requests raised against this equipment; inverse reference `equipment_id`. |
| Maintenance Count (`maintenance_count`), Current Maintenance (`maintenance_open_count`) | whole number | Inherited, stored. |
| Properties (`equipment_properties`) | property bag | Ad-hoc fields whose definition lives on the category in `equipment_properties_definition`. Carried into a duplicate. |
| Company (`company_id`) | reference to one Company | Inherited: optional, default the current company. |

### 9.5 Fields added by the inventory bridge

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Location (`location_id`) | reference to one Location | Where the equipment physically stands. Restricted to internal locations. Deletion sets this field empty. |
| Serial Match Found (`match_serial`) | true or false | Not stored; read-only. True when at least one Lot or Serial Number record carries exactly the equipment's serial number. Computed only for users who may read lot records **and** belong to the lot-tracking group; for anyone else it is false without a query. Drives the visibility of the jump-to-lot button. |

The reciprocal field on the Location entity is **Equipment Count** (`equipment_count`), not stored, the number of equipment records standing at that location.

### 9.6 Fields added by the people bridge

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Used By (`equipment_assign_to`) | selection | Who uses the equipment. Values and labels: `department` "Department"; `employee` "Employee"; `other` "Other". Required. Default `employee`. |
| Assigned Employee (`employee_id`) | reference to one Employee | The person using the equipment. Computed from the assignment mode (see section 9.7), stored, overridable. Changes recorded in the discussion thread. Indexed, skipping empty values. Deletion sets this field empty. |
| Assigned Department (`department_id`) | reference to one Department | The department using the equipment. Computed from the assignment mode, stored, overridable. Changes recorded in the discussion thread. Deletion sets this field empty. |

The reciprocal fields are: on Employee, **Equipment** (`equipment_ids`, inverse `employee_id`, readable only by users of the people user group) and **Equipment Count** (`equipment_count`, not stored); on Public Employee, **Equipment Count** (`equipment_count`, mirroring the private employee's counter so it can be shown without private-data rights).

### 9.7 Computations added by the people bridge

**Assignment normalisation.** Depends on the assignment mode.

1. When the mode is `employee`: clear the department and keep the employee.
2. When the mode is `department`: clear the employee and keep the department.
3. When the mode is `other`: keep both as they are.
4. In every case, set the assigned date to today in the acting user's time zone.

**Owner derivation.** Depends on the employee, the department and the assignment mode. Start by setting the owner to the acting user. Then: when the mode is `employee`, the owner becomes the user account of the assigned employee; when the mode is `department`, the owner becomes the user account of the assigned department's manager. When the mode is `other`, the owner remains the acting user.

### 9.8 Creation and modification side effects

- **On creation (base):** any equipment with an owner subscribes that owner's contact to the discussion thread.
- **On creation (people bridge):** any equipment with an assigned employee who has a user account subscribes that account's contact; any equipment with an assigned department whose manager has a user account subscribes that account's contact.
- **On modification (base):** writing a non-empty owner subscribes the new owner's contact.
- **On modification (people bridge):** writing a non-empty employee subscribes that employee's user's contact; writing a non-empty department subscribes that department's manager's user's contact.
- **Notification subtype:** a change of owner — or, with the people bridge, a change of assigned employee or assigned department to a non-empty value — posts under the "Equipment Assigned" subtype rather than the generic tracking subtype, so followers who only want assignment news receive it.

### 9.9 Multi-company behaviour

The company is optional. A record rule restricts visibility to equipment whose company is among the acting user's allowed companies **or is empty** — equipment with no company is shared. Company-scoped references on the record (the vendor) are validated against the equipment's company.

---

## 10. Equipment Category

**Equipment Category** (`maintenance.equipment.category`, table `maintenance_equipment_category`).

### 10.1 Purpose

A grouping of equipment — "Computers", "Monitors", "Software", "Production line A" — carrying a default responsible technician, a colour, a free comment, and the definition of the ad-hoc properties offered on the equipment it groups.

### 10.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Category Name (`name`) | text | Required. Translatable. |
| Company (`company_id`) | reference to one Company | Optional. Default is the current company. Deletion sets this field empty. |
| Responsible (`technician_user_id`) | reference to one User | The technician proposed for equipment in this category, and — when a request's equipment has no technician of its own — the fallback technician for requests. Default is the acting user. Deletion sets this field empty. |
| Color Index (`color`) | whole number | Colour slot for the card. |
| Comments (`note`) | rich text | Free text. Translatable. |
| Equipment (`equipment_ids`) | collection of Equipment | The equipment in this category; inverse reference `category_id`. Not carried into a duplicate. |
| Equipment Count (`equipment_count`) | whole number | Not stored. The number of equipment records in the category, obtained by a grouped count. |
| Maintenance (`maintenance_ids`) | collection of Maintenance Request | Every request whose equipment is in this category; inverse reference `category_id`. Not carried into a duplicate. |
| Maintenance Count (`maintenance_count`) | whole number | Not stored. The total number of requests in the category, archived and not. |
| Current Maintenance (`maintenance_open_count`) | whole number | Not stored. The number of requests in the category that are **not archived**. Note that, unlike the equipment counter, this one does not consider the stage's closing flag: a closed but unarchived request still counts as current here. |
| Folded in Maintenance Pipe (`fold`) | true or false | Stored; computed. True when the category holds no equipment, false when it holds at least one. Not carried into a duplicate. A folded category's column is collapsed in the grouped views. |
| Equipment Properties (`equipment_properties_definition`) | property definition | The definition of the ad-hoc property fields offered on equipment in this category. |

### 10.3 The counter computation

One grouped count over the requests of the categories being computed, grouped by category and by archive flag. For each category, the current counter is the count of the (category, *not archived*) group, defaulting to zero; the total counter is that plus the count of the (category, *archived*) group, defaulting to zero.

### 10.4 The folding computation

The folding flag depends on the equipment collection and is derived from the equipment count. Every category in the set is first set to *not folded*, and then each is set to folded exactly when its equipment count is zero. The initial blanket assignment exists to break the circular dependency that would otherwise arise, because the grouped count that produces the equipment count itself reads the folding flag.

### 10.5 Rules

- **Ordering**: by identifier.
- **Display name**: the category name.
- **Deletion guard**: a category holding any equipment or any maintenance request cannot be deleted. The attempt fails with **"You can't delete an equipment category if some equipment or maintenance requests are linked to it."**
- **Archival**: categories have no archive flag.
- **Multi-company**: a record rule restricts visibility to categories whose company is among the acting user's allowed companies or is empty.

---

## 11. Maintenance Team

**Maintenance Team** (`maintenance.team`, table `maintenance_team`).

### 11.1 Purpose

A team is the queue that receives maintenance requests. It owns an incoming-mail alias, so that sending a message to the team's address creates a request assigned to it, and it exposes a dashboard of counters over its open requests.

### 11.2 Inherited behaviour

The team mixes in the incoming-mail alias behaviour and the discussion thread. The alias contract — the alias name, its domain, its contact policy, its bounce content, its default values — is specified in `../messaging-and-activities/`; the fields it contributes are mirrored onto the team and are all non-stored mirrors of the underlying alias record.

### 11.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Team Name (`name`) | text | Required. Translatable. |
| Active (`active`) | true or false | The archive flag. Default true. |
| Company (`company_id`) | reference to one Company | Optional. Default is the current company. Deletion sets this field empty. |
| Team Members (`member_ids`) | collection of User | The users on the team. Relation `maintenance_team_users_rel`. Restricted to users who belong to the team's company. |
| Color Index (`color`) | whole number | Colour slot. Default zero. |
| Requests (`request_ids`) | collection of Maintenance Request | Every request assigned to this team; inverse reference `maintenance_team_id`. Not carried into a duplicate. |
| Equipment (`equipment_ids`) | collection of Equipment | Every piece of equipment whose responsible team is this one; inverse reference `maintenance_team_id`. Not carried into a duplicate. |
| Email Alias (`alias_id`) | reference to one Mail Alias | Required; the incoming-mail alias. Deletion is restricted while a team references it. Not carried into a duplicate. Its help text reads "Email alias for this maintenance team." |
| Requests (`todo_request_ids`) | collection of Maintenance Request | Not stored; read-only. The open requests of the team: assigned to it, not in a closing stage, and not archived. |
| Number of Requests (`todo_request_count`) | whole number | Not stored. The number of open requests. |
| Number of Requests Scheduled (`todo_request_count_date`) | whole number | Not stored. The number of open requests that carry a scheduled date. |
| Number of Requests in High Priority (`todo_request_count_high_priority`) | whole number | Not stored. The number of open requests whose priority is `3` (High). |
| Number of Requests Blocked (`todo_request_count_block`) | whole number | Not stored. The number of open requests whose kanban state is `blocked`. |
| Number of Requests Unscheduled (`todo_request_count_unscheduled`) | whole number | Not stored. The number of open requests with no scheduled date, computed as the total minus the scheduled count. |

### 11.4 The dashboard computation

Depends on the closing flag of the stages of the team's requests. For each team:

1. Search the open requests — team equal to this team, stage not a closing stage, not archived — and assign them to the request collection.
2. Group the same set by year of scheduled date, priority and kanban state, counting.
3. The total is the sum of all counts. The scheduled count is the sum of the counts of the groups whose scheduled-date year is non-empty. The high-priority count is the sum of the counts of the groups whose priority is `3`. The blocked count is the sum of the counts of the groups whose kanban state is `blocked`. The unscheduled count is the total minus the scheduled count.

### 11.5 The alias contract

When the team's alias is created, it is configured with:

- the entity to create set to Maintenance Request;
- when the team already exists, a default-values map containing the team's own identifier under the team field, so that every request created from an incoming message is assigned to this team.

### 11.6 Rules

- **Ordering**: by identifier.
- **Display name**: the team name.
- **Multi-company**: a record rule restricts visibility to teams whose company is among the acting user's allowed companies or is empty.
- **Shipped record**: one team named "Internal Maintenance" is shipped with the system.

---

## 12. Maintenance Stage

**Maintenance Stage** (`maintenance.stage`, table `maintenance_stage`).

### 12.1 Purpose

One column of the maintenance pipeline. Stages are fully configurable: an operator may rename them, reorder them, add them and remove them. Exactly one property matters behaviourally — the closing flag.

### 12.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | Required. Translatable. |
| Sequence (`sequence`) | whole number | Ordering weight. Default `20`. |
| Folded in Maintenance Pipe (`fold`) | true or false | When true, the stage's column is collapsed by default in the grouped view. |
| Request Done (`done`) | true or false | **The closing flag.** When true, a request that reaches this stage is considered finished: its close date is stamped, its pending maintenance activity is marked done, its equipment's open counter drops, and — if it is a recurrent preventive request — its successor is generated. |

### 12.3 Rules

- **Ordering**: by sequence ascending, then by identifier ascending. The *first* stage in this ordering is the default stage for new requests and the stage a reset request returns to.
- **Display name**: the stage name.
- **Deletion**: a stage's reference from a request restricts deletion, so a stage that is in use cannot be removed.
- **Multi-company**: stages are not company-scoped; one pipeline is shared by all companies.
- **Grouped views** expand to show every stage, including empty ones.

### 12.4 Shipped stages

| Name | Sequence | Folded | Closing |
|---|---|---|---|
| New Request | 1 | no | no |
| In Progress | 2 | no | no |
| Repaired | 3 | yes | **yes** |
| Scrap | 4 | yes | **yes** |

Two closing stages are shipped, which matters: the closing behaviour is keyed on the flag, not on a particular stage, so a request moved to "Scrap" closes exactly as one moved to "Repaired" does, including generating the successor of a recurrent preventive request.

---

## 13. Maintenance Request

**Maintenance Request** (`maintenance.request`, table `maintenance_request`).

### 13.1 Purpose

One piece of maintenance work. A request states what has to be done, to which equipment, by whom, by when, and how long it is expected to take; it records whether the work is corrective (reacting to a failure) or preventive (following a plan); and, when preventive, it may be recurrent, in which case closing it produces its successor.

### 13.2 Inherited behaviour

The request mixes in the discussion thread *with carbon-copy handling* — meaning it carries an extra `email_cc` text field holding the carbon-copy addresses captured from an incoming message — and scheduled activities.

### 13.3 Ordering, display name and archival

- **Default ordering**: by identifier descending — newest first.
- **Display name**: the subject (`name`).
- **Archival**: the request does **not** use the platform's standard archive flag. It carries its own boolean named `archive`, default false. Records with it set are hidden by the views' default filter but are *not* hidden by the platform's automatic active-record filtering, because that mechanism keys on a field named `active` which this entity deliberately does not define. Every count and dashboard query that wants to exclude archived requests states the condition explicitly.

### 13.4 Field table

#### 13.4.1 Identity and classification

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Subjects (`name`) | text | The short description of the work. Required. |
| Description (`description`) | rich text | The long description. |
| Company (`company_id`) | reference to one Company | Required. Default is the current company. Deletion is restricted while requests reference it. |
| Maintenance Type (`maintenance_type`) | selection | Values and labels: `corrective` "Corrective"; `preventive` "Preventive". Default `corrective`. Corrective requests feed the effectiveness measurements; preventive ones do not. Only preventive requests may be recurrent. |
| Equipment (`equipment_id`) | reference to one Equipment | The thing being worked on. Optional — a request may be raised with no equipment. Indexed. Restricted to the request's company. Deletion of the equipment is **restricted** while requests reference it. When the people capability is installed, the choice is narrowed to equipment assigned to the request's employee or to no employee at all. |
| Category (`category_id`) | reference to one Equipment Category | Mirrors the equipment's category. Stored so it can be grouped and filtered on. Read-only. Changes recorded in the discussion thread. Indexed, skipping empty values. Not carried into a duplicate. |
| Priority (`priority`) | selection | Values and labels: `0` "Very Low"; `1` "Low"; `2` "Normal"; `3` "High". No default — a new request has no priority until one is set. |
| Color Index (`color`) | whole number | Colour slot for the card. |
| Archive (`archive`) | true or false | The hiding flag described in section 13.3. Default false. Its help text reads "Set archive to true to hide the maintenance request without deleting it." |

#### 13.4.2 People

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Created by User (`owner_user_id`) | reference to one User | Who raised the request. Default is the acting user. Deletion sets this field empty. Not carried into a duplicate. When the people capability is installed it becomes a stored computed field derived from the employee (see section 13.6). |
| Technician (`user_id`) | reference to one User | Who will do the work. Computed from the company and equipment (see section 13.5.2), stored, overridable. Changes recorded in the discussion thread. Deletion sets this field empty. |
| Team (`maintenance_team_id`) | reference to one Maintenance Team | The queue. **Required.** Default is the first team of the current company, or — if that company has none — the first team of any company. Computed from the company and equipment (see section 13.5.1), stored, overridable. Indexed. Restricted to the request's company. Deletion restricted. |
| Employee (`employee_id`) | reference to one Employee | Present only with the people capability. The person the request concerns. Default is the acting user's own employee record. Deletion sets this field empty. |

#### 13.4.3 Stage and progress

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Stage (`stage_id`) | reference to one Maintenance Stage | The pipeline column. Default is the first stage in the stage ordering. Changes recorded in the discussion thread. Deletion of the stage is restricted. Not carried into a duplicate. Grouped views expand to show every stage. |
| Kanban State (`kanban_state`) | selection | The sub-status within a stage. Values and labels: `normal` "In Progress"; `blocked` "Blocked"; `done` "Ready for next stage". Required. Default `normal`. Changes recorded in the discussion thread. Reset to `normal` automatically whenever the stage changes and the same write does not itself set a kanban state. |
| Done (`done`) | true or false | Not stored; read-only. Mirrors the closing flag of the request's stage. |
| Close Date (`close_date`) | date | The day the work was finished. Maintained automatically by the stage logic (see `state-machines.md`, section 4): stamped with today when the request enters a closing stage; cleared when it leaves one. May also be supplied at creation, where it is corrected: supplied together with a non-closing stage it is cleared, and omitted together with a closing stage it is set to today. |

#### 13.4.4 Dates and duration

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Request Date (`request_date`) | date | The day the maintenance was asked for. Default is today in the acting user's time zone. Changes recorded in the discussion thread. This is the date used by the effectiveness measurements as the failure date. |
| Scheduled Date (`schedule_date`) | date and time | When the team plans to do the work. No default: a request may be unscheduled, and the team dashboard counts unscheduled requests separately. |
| Scheduled End (`schedule_end`) | date and time | The planned end. Computed from the scheduled date (see section 13.5.4), stored, overridable. |
| Duration (`duration`) | decimal | The planned length in hours. Stored; computed from the scheduled window (see section 13.5.5). Not carried into a duplicate. Read-only. |

#### 13.4.5 Recurrence

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Recurrent (`recurring_maintenance`) | true or false | Whether closing this request should generate its successor. Computed from the maintenance kind (see section 13.5.3), stored, overridable. Forced false for any request that is not preventive. |
| Repeat Every (`repeat_interval`) | whole number | How many repeat units to add when generating the successor. Default `1`. Must be at least one (see `business-rules.md`, rule M-02). |
| Repeat Every, unit (`repeat_unit`) | selection | Values and labels: `day` "Days"; `week` "Weeks"; `month` "Months"; `year` "Years". Default `week`. |
| Until (`repeat_type`) | selection | Values and labels: `forever` "Forever"; `until` "Until". Default `forever`. |
| End Date (`repeat_until`) | date | The last date for which a successor may be generated, honoured only when the repeat type is `until`. |

#### 13.4.6 Instructions

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Instruction (`instruction_type`) | selection | Which instruction medium is used. Values and labels: `pdf` "PDF" (a Portable Document Format attachment); `google_slide` "Google Slide" (an externally hosted slide deck); `text` "Text". Default `text`. |
| PDF (`instruction_pdf`) | binary | The attached instruction document, used when the medium is `pdf`. |
| Google Slide (`instruction_google_slide`) | text | The address of the externally hosted slide deck, used when the medium is `google_slide`. Its help text reads "Paste the url of your Google Slide. Make sure the access to the document is public." |
| Text (`instruction_text`) | rich text | The written instructions, used when the medium is `text`. |

### 13.5 Computed field rules

#### 13.5.1 Team

Depends on the company and the equipment.

1. If the request has equipment and that equipment has a responsible team, the request's team becomes that team.
2. Then, if the resulting team has a company and that company differs from the request's company, the team is cleared.

Because the field is required, step 2 can leave the record invalid; the user must then pick a team belonging to the request's company.

#### 13.5.2 Technician

Depends on the company and the equipment.

1. If the request has equipment: the technician becomes the equipment's technician, or — when the equipment has none — the technician of the equipment's category.
2. Then, if the resulting technician is set and the request's company is not among that user's allowed companies, the technician is cleared.

#### 13.5.3 Recurrence flag

Depends on the maintenance kind. Any request whose kind is not `preventive` has its recurrence flag set to false. A preventive request's flag is left as it stands, so it can be switched on by hand and survives.

#### 13.5.4 Scheduled end

Depends on the scheduled date. The scheduled end becomes the scheduled date advanced by exactly one hour; when there is no scheduled date, the scheduled end is cleared. Because the field is stored and overridable, a user-supplied end survives until the scheduled date is changed again.

#### 13.5.5 Duration

Depends on the scheduled date and the scheduled end. When both are set, the duration is the difference between them expressed in hours and rounded to two decimal places. When either is missing, the duration is zero. The formula and worked examples are in `calculations.md`, section 16.

#### 13.5.6 Created-by user, with the people capability

Depends on the employee. When the request's equipment is assigned to an employee (its assignment mode is `employee`), the created-by user becomes the user account of the request's employee; otherwise the created-by user is cleared.

### 13.6 Creation side effects

For each created request, in order:

1. If it has a created-by user or a technician, subscribe both of their contacts to the discussion thread.
2. If it has equipment but no team, re-assign the team field to itself (a no-op write that forces the stored computation to settle).
3. If it has a close date but its stage is not a closing stage, clear the close date.
4. If it has no close date and its stage is a closing stage, set the close date to today.
5. With the people capability: if the request's employee has a user account, subscribe that account's contact.

After every request in the batch has been created, the activity update pass runs over all of them (see section 13.8).

Creation also posts a message under the "Request Created" subtype, which is hidden by default and not subscribed to by default.

### 13.7 Modification side effects

Writing to a Maintenance Request has the following ordered effects. Note carefully that the recurrence generation happens **before** the write is applied, using the record's *current* values.

1. **Kanban reset.** If the write is non-empty, does not itself set a kanban state, and does set a stage, a kanban state of `normal` is added to the write.
2. **Recurrence generation.** If the write sets a stage and that stage is a closing stage, then for each request in the set that is preventive **and** recurrent, generate the successor by the algorithm of `calculations.md`, section 17.
3. The write is applied.
4. **Follower subscription.** If the write set a non-empty created-by user or a non-empty technician, subscribe both contacts.
5. **Stage consequences.** If the write set a stage: every request now in a closing stage has its close date set to today; every request now *not* in a closing stage has its close date cleared; the pending maintenance activity of every request in the set is marked as done with feedback; and every request now not in a closing stage runs the activity update pass.
6. **Activity refresh.** If the write set a non-empty technician or a non-empty scheduled date, the activity update pass runs.
7. **Activity replacement.** If the write set a non-empty equipment reference, the pending maintenance activity is deleted and the activity update pass runs, so that the activity's note — which names the equipment — is rebuilt.
8. With the people capability: if the write set a non-empty employee, that employee's user's contact is subscribed.

### 13.8 The activity update pass

1. Every request in the set that has **no** scheduled date has its pending maintenance activity deleted.
2. For every request that **has** a scheduled date:
   a. Convert the scheduled date into the acting user's time zone and take its calendar date; call this the deadline.
   b. Attempt to reschedule the existing maintenance activity to that deadline, assigning it to the request's technician, or — when there is none — the request's created-by user, or — when there is none — the acting user.
   c. If there was no existing activity to reschedule, schedule a new maintenance activity with that deadline and that assignee, and with a note reading **"Request planned for "** followed by a link to the equipment. When the request has no equipment, no note is attached.

### 13.9 Archive and reset operations

- **Archive.** Sets the archive flag to true **and** the recurrence flag to false in one write. Archiving a recurrent preventive request therefore also stops the series. It does not change the stage, so it does not itself generate a successor.
- **Reset.** Finds the stage with the lowest sequence and writes the archive flag to false and that stage onto the request. Because the stage changes, the ordinary stage consequences apply: the close date is cleared (the first stage is not a closing stage in the shipped configuration) and the kanban state is reset to `normal`.

### 13.10 Multi-company behaviour

The company is required, defaulted to the current company. A record rule restricts visibility to requests whose company is among the acting user's allowed companies or is empty. Company-scoped references on the record — equipment, team — are checked against the request's company.

### 13.11 Access-visibility rule

An ordinary internal user sees only the requests they are involved with: those they created, those on which they are a follower, and those for which they are the technician. A user in the equipment-manager group sees all of them. The exact rule domains are in `configuration.md`, section 6.

### 13.12 Creation from an incoming message

When a message arrives at a Maintenance Team's alias, a request is created with the alias's default values — which include the team — and with the message's subject as the request subject and its body as the description, following the general incoming-routing contract of `../messaging-and-activities/`. With the people capability installed, the creation is additionally seeded: the sender's address is normalised and matched against user login names; when a user matches, the *acting* user's own employee record is placed on the request.
