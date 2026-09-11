# Inventory Operations — Entities

This file specifies every entity owned by the inventory operations domain: its purpose, its lifecycle, its complete field table, its relations, its uniqueness rules, its defaults, its computed fields and the exact rule behind each, its ordering, its display rule, its archival behavior and its multi-company behavior.

Reading conventions used throughout:

- Each entity is introduced as **Name** (`transport.name`, table `storage_name`). The transport name is the identifier used by remote operations; the storage name is the database table.
- Field tables have three columns: `Field (storage name)`, `Type`, `Meaning and rules`.
- "Required" means the record cannot be saved without a value. "Stored" means the value is persisted; "computed, not stored" means it is derived on read.
- "Company scoped" means the value participates in the multi-company consistency check described in `business-rules.md`, section "Company consistency".
- Quantities are always stated with the unit they are expressed in. Three units occur: the product's own unit of measure (called the *product unit*), the unit chosen on the document line (called the *line unit*), and the packaging unit.
- Where a numeric field says "precision: Product Unit", the number of decimal digits comes from the decimal-precision setting named `Product Unit`; where it says "precision: Stock Weight", it comes from the decimal-precision setting named `Stock Weight`.

---

# 1. Warehouse

**Warehouse** (`stock.warehouse`, table `stock_warehouse`) is one physical site. It owns a tree of Locations, a set of Operation Types, a receipt Route, a delivery Route, a supply-on-order Stock Rule and, optionally, resupply Routes from other warehouses.

## 1.1 Purpose and lifecycle

1. A Warehouse is created with a name and a short name. Creating it creates its whole location tree and its whole set of Operation Types and sequences, then its Routes and Stock Rules, in that order.
2. Changing the receipt step configuration or the delivery step configuration rewrites the corresponding Route and its Stock Rules and activates or archives the intermediate Locations and Operation Types.
3. Archiving a Warehouse archives its Operation Types, its view Location subtree, its Stock Rules and the Routes that apply only to it. Archiving is refused while any move of the Warehouse's Operation Types is still open.
4. A Warehouse is never deleted while stock or open documents refer to its Locations; deletion is possible but leaves the Locations behind.

## 1.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | Required. Default: when the company has no Warehouse yet, the company name; otherwise the company name followed by " - warehouse # " and the count of existing Warehouses of that company plus one. Unique per company (constraint message: "The name of the warehouse must be unique per company!"). Renaming rewrites the names of the Warehouse's Routes and Stock Rules by textual substitution of the old name by the new one, first occurrence only, and rewrites the names of the numbering sequences of every Operation Type of the Warehouse. |
| Active (`active`) | boolean | Default true. Archiving cascades as described in 1.1 step 3. |
| Company (`company_id`) | link to Company | Required, read-only, default: the active company. Changing it after creation is refused with "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| Address (`partner_id`) | link to Contact | Default: the company's contact. Company scoped. Setting it rewrites that contact's customer stock location and vendor stock location to the company's internal transit Location. |
| View Location (`view_location_id`) | link to Location | Required, company scoped, indexed. Domain: usage must be virtual and the company must match. Created automatically at Warehouse creation, named with the short name. |
| Stock Location (`lot_stock_id`) | link to Location | Required, company scoped. Domain: usage must be internal and the company must match. Created automatically, named "Stock", flagged as a replenishment location, with barcode equal to the short name followed by `STOCK` when that barcode is free. |
| Short Name (`code`) | text, at most 5 characters | Required. Unique per company (constraint message: "The short name of the warehouse must be unique per company!"). Used as the prefix of every numbering sequence and of every Location barcode of the Warehouse. Changing it renames the view Location's parent of the stock Location and rewrites all sequence prefixes. On copy the short name becomes the literal text "COPY". |
| Routes (`route_ids`) | many-to-many to Route | The Routes that are selectable for this Warehouse. Domain: only Routes flagged as selectable on a warehouse, and either company-less or of the same company. Not copied. |
| Incoming Shipments (`reception_steps`) | selection | Required, default `one_step`. Values: `one_step` "Receive and Store (1 step)", `two_steps` "Receive then Store (2 steps)", `three_steps` "Receive, Quality Control, then Store (3 steps)". |
| Outgoing Shipments (`delivery_steps`) | selection | Required, default `ship_only`. Values: `ship_only` "Deliver (1 step)", `pick_ship` "Pick then Deliver (2 steps)", `pick_pack_ship` "Pick, Pack, then Deliver (3 steps)". |
| Input Location (`wh_input_stock_loc_id`) | link to Location | Company scoped. Created automatically, named "Input", barcode short name plus `INPUT`. Active only when receipts take more than one step. |
| Quality Control Location (`wh_qc_stock_loc_id`) | link to Location | Company scoped. Created automatically, named "Quality Control", barcode short name plus `QUALITY`. Active only when receipts take three steps. |
| Output Location (`wh_output_stock_loc_id`) | link to Location | Company scoped. Created automatically, named "Output", barcode short name plus `OUTPUT`. Active only when deliveries take more than one step. |
| Packing Location (`wh_pack_stock_loc_id`) | link to Location | Company scoped. Created automatically, named "Packing Zone", barcode short name plus `PACKING`. Active only when deliveries take three steps. |
| Supply-on-order Rule (`mto_pull_id`) | link to Stock Rule | Not copied. The single rule of the global "Replenish on Order" Route that belongs to this Warehouse. Rebuilt whenever the delivery step configuration changes. |
| Pick Type (`pick_type_id`) | link to Operation Type | Not copied, company scoped. |
| Pack Type (`pack_type_id`) | link to Operation Type | Not copied, company scoped. |
| Out Type (`out_type_id`) | link to Operation Type | Not copied, company scoped. |
| In Type (`in_type_id`) | link to Operation Type | Not copied, company scoped. |
| Internal Type (`int_type_id`) | link to Operation Type | Not copied, company scoped. |
| Quality Control Type (`qc_type_id`) | link to Operation Type | Not copied, company scoped. |
| Storage Type (`store_type_id`) | link to Operation Type | Not copied, company scoped. |
| Cross Dock Type (`xdock_type_id`) | link to Operation Type | Not copied, company scoped. |
| Receipt Route (`reception_route_id`) | link to Route | Not copied. Deletion of the Route is restricted while the Warehouse points at it. |
| Delivery Route (`delivery_route_id`) | link to Route | Not copied. Deletion of the Route is restricted while the Warehouse points at it. |
| Resupply From (`resupply_wh_ids`) | many-to-many to Warehouse | The Warehouses that may supply this one. Adding one creates (or un-archives) a resupply Route; removing one archives it. |
| Resupply Routes (`resupply_route_ids`) | one-to-many of Route (through the supplied-warehouse link) | Not copied. The Routes generated by the resupply configuration. |
| Sequence (`sequence`) | integer | Default 10. Orders Warehouses in lists. |

Ordering: by sequence, then by identifier.

## 1.3 What creation builds

Creating a Warehouse performs, in order:

1. Create the view Location named with the short name, usage virtual, company of the Warehouse.
2. Create the five sublocations under it (stock, input, quality control, output, packing) with the names, usages, active flags and barcodes described in the field table. A barcode is only applied when no other Location of the same company already carries it.
3. Insert the Warehouse row.
4. Create the eight numbering sequences and the eight Operation Types (see section 5.6 and `configuration.md`). Set the return Operation Type of the delivery type to the receipt type and the return Operation Type of the receipt type to the delivery type.
5. Create or update the receipt Route and the delivery Route with their Stock Rules.
6. Create or update the supply-on-order Stock Rule inside the global "Replenish on Order" Route.
7. Create resupply Routes for each Warehouse listed as a supplier.
8. If an address was given, rewrite that contact's customer and vendor stock locations to the company's internal transit Location.
9. Stamp the Warehouse identifier on the view Location and all its direct children.
10. Re-evaluate the multi-warehouse security group: if any company has more than one active Warehouse, the multi-warehouse group and the multi-location group are granted to all internal users; if no company has more than one, the multi-warehouse group is withdrawn.

---

# 2. Location

**Location** (`stock.location`, table `stock_location`) is one node of the storage tree.

## 2.1 Purpose and lifecycle

A Location is a place goods can be, or a virtual counterpart used to balance movements. Locations form a tree through the parent link, and the tree is materialised in a path column so that ancestor and descendant tests are string-prefix tests.

## 2.2 Usages

| Value | Label | Meaning |
|---|---|---|
| `supplier` | Vendor | Virtual source of goods bought from outside. Reservation is bypassed. |
| `view` | Virtual | Grouping node only. May never hold goods. |
| `internal` | Internal | A real place inside a warehouse. Holds counted stock. |
| `customer` | Customer | Virtual destination of goods sent outside. Reservation is bypassed. |
| `inventory` | Inventory Loss | Virtual counterpart of counts and scraps. Reservation is bypassed. |
| `production` | Production | Virtual counterpart of manufacturing consumption and production. Reservation is bypassed. |
| `transit` | Transit | Holding point between two warehouses or two companies. Holds counted stock; reservation is **not** bypassed. |

## 2.3 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Location Name (`name`) | text | Required. On copy becomes the original name followed by " (copy)". |
| Full Location Name (`complete_name`) | text | Computed and stored, recursive. Equal to the parent's full name, a slash, and this name, unless the Location has no parent or its own usage is virtual, in which case it is just the name. Depends on the name, the parent's full name and the usage. |
| Active (`active`) | boolean | Default true. |
| Location Type (`usage`) | selection | Required, default `internal`, indexed. Values as in 2.2. |
| Parent Location (`location_id`) | link to Location | Indexed, company scoped. |
| Contains (`child_ids`) | one-to-many of Location | The direct children. |
| Internal locations among descendants (`child_internal_location_ids`) | many-to-many of Location, computed, not stored, recursive | All descendants of this Location, including itself, whose usage is internal. |
| Path (`parent_path`) | text, indexed | The materialised tree path: the slash-separated list of ancestor identifiers ending with this Location's own identifier and a trailing slash. A Location A is a descendant of B exactly when A's path starts with B's path. |
| Company (`company_id`) | link to Company | Default: the active company. Indexed. May be left empty, which makes the Location shared between companies. Changing it after creation is refused with "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| Replenishments (`replenish_location`) | boolean | Computed, stored, editable. Forced to false whenever the usage is not internal. Not copied. Marks the Location as the one replenishment suggestions target. |
| Removal Strategy (`removal_strategy_id`) | link to Removal Strategy | Optional. When empty, the strategy is inherited from the nearest ancestor that defines one; the product category takes precedence over all Locations. |
| Putaway Rules (`putaway_rule_ids`) | one-to-many of Put-away Rule (through the arrival-location link) | The rules that redirect goods arriving here. |
| Barcode (`barcode`) | text | Not copied. Unique per company; violation message: "The barcode for a location must be unique per company!". When creating a Location through a form that already computed a full name and no barcode was given, the barcode defaults to the full name. |
| Quantities (`quant_ids`) | one-to-many of Stock Quantity | The quantity records stored here. |
| Inventory Frequency (`cyclic_inventory_frequency`) | integer, days | Default 0, must be greater than or equal to zero; violation message: "The inventory frequency (days) for a location must be non-negative". Zero means no cyclic counting. |
| Last Inventory (`last_inventory_date`) | date | Read-only. Stamped with today's date each time an adjustment is applied on a quantity record of this Location. |
| Next Expected (`next_inventory_date`) | date | Computed and stored. See `calculations.md`, section "Next count date". Empty when the Location has no company, or its usage is neither internal nor transit, or the frequency is zero. |
| Warehouse views (`warehouse_view_ids`) | one-to-many of Warehouse | The Warehouses whose view Location is this Location. |
| Warehouse (`warehouse_id`) | link to Warehouse, computed and stored | The Warehouse whose view Location is the nearest ancestor of this Location. Computed by scanning the Location's path from the deepest ancestor upwards against the set of warehouse view Locations sorted by decreasing path length. |
| Storage Category (`storage_category_id`) | link to Storage Category | Company scoped, indexed when not empty. |
| Outgoing move lines (`outgoing_move_line_ids`) | one-to-many of Stock Move Line (through the source-location link) | Used only for the weight computation. |
| Incoming move lines (`incoming_move_line_ids`) | one-to-many of Stock Move Line (through the destination-location link) | Used only for the weight computation. |
| Net Weight (`net_weight`) | decimal, computed, not stored | See `calculations.md`, section "Location weight". |
| Forecasted Weight (`forecast_weight`) | decimal, computed, not stored | See `calculations.md`, section "Location weight". |
| Is Empty (`is_empty`) | boolean, computed, not stored, searchable | True when the sum of on-hand quantities of the quantity records directly in this Location (restricted to internal and transit Locations) is less than or equal to zero. |

Ordering: by full location name, then by identifier. Text search matches the full location name or the barcode. An extra database index exists on the pair (path, identifier).

## 2.4 Display rule

The displayed name is the full name of the parent, a slash, and this Location's name, when the Location has a parent and its own usage is not virtual; otherwise it is just the name. In the "formatted" display context the same string is produced but each of the two parts is prefixed with two hyphens.

## 2.5 Creating by name

When a Location is created by typing a name that contains slashes, the text before the last slash is looked up as a full location name; if a Location with that full name exists it becomes the parent, and the text after the last slash becomes the new Location's name.

## 2.6 Archival behavior

- Archiving is refused when an active Warehouse uses the Location as its stock Location or its view Location. Message: "You cannot archive location *the location name* because it is used by warehouse *the warehouse name*".
- Archiving a Location archives all of its descendants, unless one of the internal descendants still holds a non-zero on-hand or reserved quantity, in which case the whole operation is refused with "You can't disable locations *the comma-separated list of location display names* because they still contain products."
- Deleting a Location deletes all of its descendants. Deleting the shared inter-company Location is refused with "The *the location name* location is required by the Inventory app and cannot be deleted, but you can archive it."

## 2.7 Other write-time rules

- Changing the usage to virtual is refused when the Location holds quantity records: "This location's usage cannot be changed to view as it contains products."
- Changing the usage at all is refused when any of the Locations being changed holds a quantity record with a positive on-hand quantity: "Internal locations having stock can't be converted".
- Marking a Location as a replenishment Location is refused when an ancestor or a descendant is already marked: "Another parent/sub replenish location *the other location's name* exists, if you wish to change it, uncheck it first".
- Setting the usage to inventory loss is refused when the Location is the default destination of a manufacturing Operation Type: "You cannot set a location as a scrap location when it is assigned as a destination location for a manufacturing type operation."

---

# 3. Route

**Route** (`stock.route`, table `stock_route`) is a named, ordered collection of Stock Rules.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Route (`name`) | text, translated | Required. On copy becomes the original followed by " (copy)". |
| Active (`active`) | boolean | Default true. Archiving a Route archives all of its Stock Rules whose destination Location is still active; un-archiving un-archives them. |
| Sequence (`sequence`) | integer | Default 0. Routes are ordered by it; the order matters for rule selection (see `calculations.md`, section "Rule matching"). |
| Rules (`rule_ids`) | one-to-many of Stock Rule | Copied with the Route. |
| Applicable on Product (`product_selectable`) | boolean | Default true. |
| Applicable on Product Category (`product_categ_selectable`) | boolean | Default false. |
| Applicable on Warehouse (`warehouse_selectable`) | boolean | Default false. Clearing it clears the Warehouses list. |
| Applicable on Package Type (`package_type_selectable`) | boolean | Default false. |
| Supplied Warehouse (`supplied_wh_id`) | link to Warehouse | Set on generated resupply Routes: the Warehouse being replenished. |
| Supplying Warehouse (`supplier_wh_id`) | link to Warehouse | Set on generated resupply Routes: the Warehouse doing the supplying. |
| Company (`company_id`) | link to Company | Default: the active company. May be empty, which shares the Route between companies. |
| Products (`product_ids`) | many-to-many to Product Template | Not copied, company scoped. |
| Product Categories (`categ_ids`) | many-to-many to Product Category | Not copied. |
| Warehouses (`warehouse_ids`) | many-to-many to Warehouse | Not copied. Restricted to the Warehouses of the Route's company. |
| Allowed warehouses (`warehouse_domain_ids`) | one-to-many of Warehouse, computed, not stored | The Warehouses of the Route's company, or all Warehouses when the Route has no company. |

Ordering: by sequence.

Validation: every Stock Rule of a Route with a company must have that same company, else "Rule *the rule name* belongs to *the rule's company name* while the route belongs to *the route's company name*."

---

# 4. Stock Rule

**Stock Rule** (`stock.rule`, table `stock_rule`) is one step of a Route: it says how goods move from a source Location to a destination Location through an Operation Type.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text, translated | Required. Fills the origin of the documents it creates. Generated rules are named "*short name*: *source location name* → *destination location name*" with an optional suffix in parentheses. On copy becomes the original followed by " (copy)". |
| Active (`active`) | boolean | Default true. |
| Action (`action`) | selection | Required, default `pull`, indexed. Values: `pull` "Pull From" (a need at the destination creates a document sourcing from the source), `push` "Push To" (an arrival at the source creates a document sending to the destination), `pull_push` "Pull & Push" (both). |
| Sequence (`sequence`) | integer | Default 20. Rules are ordered by sequence then identifier; the order decides which rule wins when several match. |
| Company (`company_id`) | link to Company | Default: the active company. Must equal the Route's company when the Route has one. |
| Destination Location (`location_dest_id`) | link to Location | Required, company scoped, indexed. |
| Source Location (`location_src_id`) | link to Location | Company scoped, indexed. A pull rule without a source Location raises "No source location defined on stock rule: *the rule name*!" when it runs. |
| Destination location origin from rule (`location_dest_from_rule`) | boolean | Default false. When true, the Stock Move created by this rule takes its intermediate destination Location from the rule instead of from the Operation Type. |
| Route (`route_id`) | link to Route | Required, indexed; deleting the Route deletes the rule. |
| Route Company (`route_company_id`) | link to Company, related to the Route | Read-only. |
| Supply Method (`procure_method`) | selection | Required, default `make_to_stock`. Values: `make_to_stock` "Take From Stock", `make_to_order` "Trigger Another Rule", `mts_else_mto` "Take From Stock, if unavailable, Trigger Another Rule". |
| Route Sequence (`route_sequence`) | integer, related to the Route's sequence, stored | Used for ordering candidate rules. |
| Operation Type (`picking_type_id`) | link to Operation Type | Required, company scoped. Choosing one copies its default source and destination Locations onto the rule. |
| Lead Time (`delay`) | integer, days | Default 0. Subtracted from the planned date when a pull rule creates a move; added to the date when a push rule creates a move. |
| Partner Address (`partner_address_id`) | link to Contact | Company scoped. Optional delivery address forced on the created move. |
| Cancel Next Move (`propagate_cancel`) | boolean | Default false. When true, cancelling the move created by this rule cancels the next move of the chain. |
| Propagation of carrier (`propagate_carrier`) | boolean | Default false. Used by the delivery domain. |
| Warehouse (`warehouse_id`) | link to Warehouse | Company scoped, indexed. |
| Automatic Move (`auto`) | selection | Required, default `manual`. Values: `manual` "Manual Operation" (create a second move after the current one), `transparent` "Automatic No Step Added" (rewrite the destination of the current move instead of creating a second one). |
| Rule description (`rule_message`) | rich text, computed, not stored | A human-readable sentence describing what the rule does. See `interfaces.md`, section "Rule description sentences". |
| Push Applicability (`push_domain`) | text | An optional filter expression. A push rule whose filter does not match the arriving move is skipped and the search continues excluding it. |
| Allowed operation kinds (`picking_type_code_domain`) | structured data, computed, not stored | Empty in this domain; extended by other domains to restrict the choice of Operation Type per action. |

Ordering: by sequence, then by identifier.

---

# 5. Operation Type

**Operation Type** (`stock.picking.type`, table `stock_picking_type`) is the template and counter holder for Transfers of one kind in one Warehouse.

## 5.1 Field table — identity and defaults

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Operation Type (`name`) | text, translated | Required. On copy becomes the original followed by " (copy)". |
| Color (`color`) | integer | Chosen at Warehouse creation as the lowest number from 0 to 11 not already used by another Warehouse's Operation Types, or 0 when all are used. |
| Sequence (`sequence`) | integer | Orders the operation cards in the overview. |
| Reference Sequence (`sequence_id`) | link to Numbering Sequence | Company scoped, not copied. Created automatically: when the Operation Type belongs to a Warehouse, the sequence is named "*warehouse name* Sequence *sequence prefix*" with prefix "*short name*/*sequence prefix*/" and padding 5; otherwise it is named "Sequence *sequence prefix*" with prefix equal to the sequence prefix and padding 5. |
| Sequence Prefix (`sequence_code`) | text | Required. On copy becomes the original followed by " (copy)". Changing it rewrites the name, prefix and padding of the numbering sequence. A warning is shown when another Operation Type of the same company (or a company-less one) already uses the same prefix with a different sequence: "This sequence prefix is already being used by another operation type. It is recommended that you select a unique prefix to avoid issues and/or repeated reference values or assign the existing reference sequence to this operation type." |
| Source Location (`default_location_src_id`) | link to Location | Required, company scoped, computed and stored, editable. Default rule: for a receipt, the shared vendor Location; otherwise the Warehouse's stock Location. When there is no Warehouse, the user is redirected to the Warehouse screen with "Please create a warehouse for company *the company name*."; a user without the inventory manager group instead gets "Please contact your administrator to configure your warehouse." |
| Destination Location (`default_location_dest_id`) | link to Location | Required, company scoped, computed and stored, editable. Default rule: for a delivery, the shared customer Location; otherwise the Warehouse's stock Location. |
| Type of Operation (`code`) | selection | Required, default `incoming`. Values: `incoming` "Receipt", `outgoing` "Delivery", `internal` "Internal Transfer". Choosing internal without the multi-location group shows "You need to activate storage locations to be able to do internal operation types." |
| Operation Type for Returns (`return_picking_type_id`) | link to Operation Type | Company scoped, indexed when not empty. |
| Warehouse (`warehouse_id`) | link to Operation Type's Warehouse | Computed and stored, editable, company scoped. Default: the first Warehouse of the Operation Type's company. Deleting the Warehouse deletes the Operation Type. |
| Active (`active`) | boolean | Default true. |
| Barcode (`barcode`) | text | Not copied. Generated Operation Types get the Warehouse short name (spaces removed, upper-cased) followed by `IN`, `OUT`, `PICK`, `PACK`, `QC`, `STOR`, `INT` or `XD`. |
| Company (`company_id`) | link to Company | Required, default: the active company, indexed. Changing it after creation is refused with "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| Show Operation in Overview (`is_favorite`) | boolean, computed, not stored, searchable and writable | True when the current user is in the favourite-users list. Writing it adds or removes the current user. Operation Types are ordered with favourites first. |
| Favourite users (`favorite_user_ids`) | many-to-many to User | The users who pinned this Operation Type. |
| Picking Properties (`picking_properties_definition`) | properties definition | The schema of the free properties available on Transfers of this Operation Type. |
| Shipping Policy (`move_type`) | selection | Required, default `direct`. Values: `direct` "As soon as possible", `one` "When all products are ready". Copied onto each new Transfer. |

## 5.2 Field table — behavior switches

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Move Entire Packages (`show_entire_packs`) | boolean | Default false. When true, containers rather than their contents are presented in scanning screens. |
| Set Package Type (`set_package_type`) | boolean | Default false. When true, the put-in-pack action first asks which container or container type to use. |
| Create New Lots/Serial Numbers (`use_create_lots`) | boolean | Computed, stored, editable. Default true; forced to true for receipts. When true, a person may type a new lot or serial number, and the system creates the Lot record at completion. |
| Use Existing Lots/Serial Numbers (`use_existing_lots`) | boolean | Computed, stored, editable. Default true; forced to true for deliveries. When true, a person may select an existing Lot. |
| Generate Shipping Labels (`print_label`) | boolean | Computed, stored, editable. Forced to false for receipts and internal transfers, true for deliveries. |
| Show Detailed Operations (`show_operations`) | boolean | Default false. When true the Transfer screen lists Stock Move Lines instead of Stock Moves. |
| Reservation Method (`reservation_method`) | selection | Required, default `at_confirm`. Values: `at_confirm` "At Confirmation", `manual` "Manually", `by_date` "Before scheduled date". |
| Days (`reservation_days_before`) | integer | Number of days before the scheduled date at which an ordinary move becomes eligible for automatic reservation. |
| Days when starred (`reservation_days_before_priority`) | integer | The same for moves whose priority is urgent. |
| Create Backorder (`create_backorder`) | selection | Required, default `ask`. Values: `ask` "Ask", `always` "Always", `never` "Never". |
| Hide reservation method (`hide_reservation_method`) | boolean, computed, not stored | True for receipts, where reservation is meaningless. |
| Show operation kind (`show_picking_type`) | boolean, computed, not stored | True when the kind is receipt, delivery or internal transfer. |

## 5.3 Field table — automatic printing

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Show Reception Report at Validation (`auto_show_reception_report`) | boolean | Opens the Reception Report after a successful validation when there is something to allocate. |
| Auto Print Delivery Slip (`auto_print_delivery_slip`) | boolean | Queues the delivery document for printing at validation. |
| Auto Print Return Slip (`auto_print_return_slip`) | boolean | Queues the return label document. |
| Auto Print Product Labels (`auto_print_product_labels`) | boolean | Queues product labels. |
| Product Label Format to auto-print (`product_label_format`) | selection | Default `2x7xprice`. Values: `dymo` "Dymo", `2x7xprice` "2 x 7 with price", `4x7xprice` "4 x 7 with price", `4x12` "4 x 12", `4x12xprice` "4 x 12 with price", `zpl` "ZPL Labels", `zplxprice` "ZPL Labels with price". |
| Auto Print Lot/SN Labels (`auto_print_lot_labels`) | boolean | Queues lot and serial-number labels. |
| Lot Label Format to auto-print (`lot_label_format`) | selection | Default `4x12_lots`. Values: `4x12_lots` "4 x 12 - One per lot/SN", `4x12_units` "4 x 12 - One per unit", `zpl_lots` "ZPL Labels - One per lot/SN", `zpl_units` "ZPL Labels - One per unit". |
| Auto Print Reception Report (`auto_print_reception_report`) | boolean | Queues the Reception Report. Only for non-delivery kinds and only when the Transfer's moves have destination moves. |
| Auto Print Reception Report Labels (`auto_print_reception_report_labels`) | boolean | Queues one label per destination move, in a count equal to the demand rounded up to the next whole number. |
| Auto Print Packages (`auto_print_packages`) | boolean | Queues the container-content document. |
| Auto Print Package Label (`auto_print_package_label`) | boolean | Prints the container label immediately when a container is created by the put-in-pack action. |
| Package Label to Print (`package_label_to_print`) | selection | Default `pdf`. Values: `pdf` "PDF", `zpl` "ZPL". |

## 5.4 Field table — counters

All counters restrict to Transfers of this Operation Type whose state is neither done nor cancelled.

| Field (storage name) | Type | Counting rule |
|---|---|---|
| `count_picking_draft` | integer, computed, not stored | Transfers in state draft. |
| `count_picking_waiting` | integer, computed, not stored | Transfers in state waiting-another-operation or waiting. |
| `count_picking_ready` | integer, computed, not stored | Transfers in state ready. |
| `count_picking` | integer, computed, not stored | Transfers in state ready, waiting-another-operation or waiting. |
| `count_picking_late` | integer, computed, not stored | Transfers in those same three states whose scheduled date is earlier than today, or which are flagged late. |
| `count_picking_backorders` | integer, computed, not stored | Transfers in state waiting, ready or waiting-another-operation that have a back-order link. |
| `count_move_ready` | integer, computed, not stored | Stock Moves of this Operation Type in state assigned (no state filter on the Transfer). |
| `kanban_dashboard_graph` | text, computed, not stored | A serialised six-bucket bar series. See `calculations.md`, section "Operation overview graph". |

## 5.5 Report title per kind

The printable title of a Transfer depends on the Operation Type kind: delivery gives "Delivery Note", receipt gives "Goods Receipt Note", internal transfer gives "Internal Move".

## 5.6 The eight generated Operation Types

For every Warehouse the following Operation Types are created. `M` denotes the highest sequence in use at creation time.

| Warehouse field | Name | Kind | Sequence prefix | Sequence | Create lots | Use existing lots | Default source | Default destination | Active when |
|---|---|---|---|---|---|---|---|---|
| `in_type_id` | Receipts | incoming | `IN` | M + 1 | yes | no | shared vendor Location | input Location (stock Location when receipts take one step) | always |
| `qc_type_id` | Quality Control | internal | `QC` | M + 2 | no | yes | input Location | quality control Location | receipts take three steps |
| `store_type_id` | Storage | internal | `STOR` | M + 3 | no | yes | input Location for two steps, quality control Location for three steps | stock Location | receipts take more than one step |
| `int_type_id` | Internal Transfers | internal | `INT` | M + 4 | no | yes | stock Location | stock Location | the multi-location group is active |
| `pick_type_id` | Pick | internal | `PICK` | M + 5 | no | yes | stock Location | output Location for two-step delivery, packing Location for three-step delivery | deliveries take more than one step |
| `pack_type_id` | Pack | internal | `PACK` | M + 6 | no | yes | packing Location | output Location | deliveries take three steps |
| `out_type_id` | Delivery Orders | outgoing | `OUT` | M + 7 | no | yes | output Location (stock Location when deliveries take one step) | shared customer Location | always |
| `xdock_type_id` | Cross Dock | internal | `XD` | M + 8 | no | yes | input Location | output Location | receipts take more than one step **and** deliveries take more than one step |

The delivery type also has the shipping-label flag set. The receipt type and the delivery type point at each other as return Operation Types.

---

# 6. Transfer

**Transfer** (`stock.picking`, table `stock_picking`) is one document grouping the Stock Moves that travel together between two Locations. It carries a discussion thread and activity scheduling.

## 6.1 Lifecycle

1. Created in state draft, either by hand or by a Stock Rule that grouped moves.
2. Confirmed: every draft move is confirmed; moves that need supplying trigger their rules.
3. Reserved, automatically or on demand, which creates Stock Move Lines and raises reserved counters.
4. Validated: checked, optionally split into a backorder, completed. The state becomes done and the completion date is stamped.
5. Or cancelled, which cancels its moves.

## 6.2 Field table — identity

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Reference (`name`) | text | Read-only, not copied, trigram-indexed. Default the single character `/`; replaced at creation by the next number of the Operation Type's numbering sequence. Unique per company; violation message: "Reference must be unique per company!". Changing the Operation Type of an open Transfer draws a new number from the new sequence. |
| Source Document (`origin`) | text, trigram-indexed | Free text naming the document that caused the Transfer. When moves are grouped into an existing Transfer the origins are merged as a comma-separated list preserving order and removing duplicates; when a Transfer is created from a group of moves the first five distinct origins are joined by commas and, if there were more, three dots are appended. |
| Notes (`note`) | rich text | Free text. |
| Back Order of (`backorder_id`) | link to Transfer | Read-only, not copied, indexed when not empty, company scoped. |
| Back Orders (`backorder_ids`) | one-to-many of Transfer | The Transfers created as backorders of this one. |
| Return of (`return_id`) | link to Transfer | Read-only, not copied, indexed when not empty, company scoped. |
| Returns (`return_ids`) | one-to-many of Transfer | The Transfers created as returns of this one. |
| Number of returns (`return_count`) | integer, computed, not stored | The number of returns. |
| Operation Type (`picking_type_id`) | link to Operation Type | Required, indexed, tracked in the discussion thread. Default: when the screen restricts to one kind, the first Operation Type of that kind belonging to the active company. Changing it on a done or cancelled Transfer is refused with "Changing the operation type of this record is forbidden at this point." Changing it on an open Transfer redraws the reference and resets both Locations to the new type's defaults. |
| Operation kind (`picking_type_code`) | selection, related to the Operation Type's kind | Read-only. |
| Move entire packages (`picking_type_entire_packs`) | boolean, related | Read-only. |
| Create lots allowed (`use_create_lots`) | boolean, related | Read-only. |
| Use existing lots allowed (`use_existing_lots`) | boolean, related | Read-only. |
| Warehouse address (`warehouse_address_id`) | link to Contact, related to the Operation Type's Warehouse address | Read-only. |
| Contact (`partner_id`) | link to Contact | Company scoped, indexed when not empty. Writing it also writes it on every move whose destination usage is not inventory loss. |
| Company (`company_id`) | link to Company, related to the Operation Type, stored | Read-only, indexed. |
| Responsible (`user_id`) | link to User | Tracked. Default: the current user. Not copied. Restricted to users in the inventory user group. A backorder is created with no responsible. |
| Assign Owner (`owner_id`) | link to Contact | Company scoped, indexed when not empty. At validation, every move gets this contact as its owner restriction and every move line gets it as its owner. |
| Printed (`printed`) | boolean | Not copied. Set to true when the transfer document is printed. A printed Transfer is no longer a candidate for absorbing newly created moves. |
| Signature (`signature`) | image, stored as an attachment | Not copied. Writing it renders the delivery document to a portable-document-format file and posts it in the discussion thread with the message "Order signed by *the contact name*" or "Order signed" when there is no contact. |
| Is Signed (`is_signed`) | boolean, computed, not stored | True when a signature is present. |
| Properties (`picking_properties`) | properties | Schema taken from the Operation Type. Copied. |
| Instructions (`picking_warning_text`) | text, computed, not stored | Concatenation of the contact's transfer warning message and its parent contact's transfer warning message, each followed by a line break. Empty unless the stock-warning group is active. |
| Contact country (`partner_country_id`) | link to Country, related | Read-only. |

## 6.3 Field table — state, dates and policy

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Status (`state`) | selection, computed and stored, tracked, indexed, not copied | Values: `draft` "Draft", `waiting` "Waiting Another Operation", `confirmed` "Waiting", `assigned` "Ready", `done` "Done", `cancel` "Cancelled". Full computation in `state-machines.md`. |
| Shipping Policy (`move_type`) | selection, computed and stored, editable, required | Default from the Operation Type. Values: `direct` "As soon as possible", `one` "When all products are ready". |
| Priority (`priority`) | selection | Default `0`. Values: `0` "Normal", `1` "Urgent". Reset to normal at validation. |
| Scheduled Date (`scheduled_date`) | date and time, computed and stored, writable, indexed, tracked | Default now. Computation: over the moves that are neither done nor cancelled, the minimum of their dates when the shipping policy is as-soon-as-possible, the maximum when it is all-at-once; when there are no such moves the previous value or now. Writing it writes the same date on every move; writing it on a cancelled Transfer is refused with "You cannot change the Scheduled Date on a cancelled transfer."; writing it on a done Transfer is ignored. |
| Deadline (`date_deadline`) | date and time, computed and stored | Over the moves that are not cancelled and have a deadline: the minimum when the shipping policy is as-soon-as-possible, the maximum when it is all-at-once. |
| Is late (`has_deadline_issue`) | boolean, computed and stored | True when a deadline exists and is earlier than the scheduled date. |
| Date of Transfer (`date_done`) | date and time | Not copied. Stamped with the current instant at validation. Writing it also writes it as the date of every done move. |
| Delay Alert Date (`delay_alert_date`) | date and time, computed, not stored, searchable | The maximum delay alert date among the Transfer's moves. |
| Rescheduling popover (`json_popover`) | text, computed, not stored | Empty for done and cancelled Transfers and when there is no delay alert date; otherwise a structure naming the alert date and the upstream documents that are late. |
| Is Scheduled Date Editable (`is_date_editable`) | boolean, computed, not stored | For done and cancelled Transfers: true only when the Transfer is unlocked. Otherwise always true. |
| Locked (`is_locked`) | boolean | Default true, not copied. While the Transfer is open it forbids editing the demand; once done it forbids editing the processed quantities. Toggled by the lock action. Cancelling a Transfer forces it to true. |

## 6.4 Field table — locations, content and weights

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Source Location (`location_id`) | link to Location | Required, company scoped, computed and stored, editable. Computation: the Operation Type's default source; but when that default has vendor usage and the Transfer has a contact whose own vendor stock Location differs from the system default, that contact Location is used instead. Not recomputed for done, cancelled or return Transfers. Writing it writes it on every move whose destination usage is not inventory loss. |
| Destination Location (`location_dest_id`) | link to Location | Required, company scoped, computed and stored, editable. Same rule mirrored: the Operation Type's default destination, overridden by the contact's own customer stock Location when the default has customer usage. |
| Stock Moves (`move_ids`) | one-to-many of Stock Move | Copied. Adding moves to an existing Transfer re-runs the automatic confirmation. |
| Operations (`move_line_ids`) | one-to-many of Stock Move Line | The detail lines of all the moves plus any line created directly on the Transfer. |
| Product (`product_id`) | link to Product, related through the moves | Read-only; exists so that Transfers can be searched by product. |
| Lot/Serial Number (`lot_id`) | link to Lot, related through the move lines | Read-only; exists so that Transfers can be searched by lot. |
| Has Scrap Moves (`has_scrap_move`) | boolean, computed, not stored | True when at least one move of the Transfer has a destination Location with inventory-loss usage. |
| Packages Count (`packages_count`) | integer, computed, not stored | For a done Transfer: the number of Package History records linked to it. Otherwise: the number of containers that name the Transfer among their Transfers. |
| Transferred Packages (`package_history_ids`) | many-to-many to Package History | Not copied. Filled at completion. |
| Bulk Weight (`weight_bulk`) | decimal, computed, not stored | See `calculations.md`, section "Transfer weights". |
| Weight for Shipping (`shipping_weight`) | decimal, computed and stored, editable, precision Stock Weight | See `calculations.md`, section "Transfer weights". |
| Volume for Shipping (`shipping_volume`) | decimal, computed, not stored | The sum over the moves of the processed quantity converted to the product unit multiplied by the product's volume. |
| Show detailed operations (`show_operations`) | boolean, related to the Operation Type | Read-only. |
| Show lot text box (`show_lots_text`) | boolean, computed, not stored | True when the lot group is active, the Operation Type allows creating lots but not using existing ones, and the Transfer is not done. False when the Transfer has no move lines and the Operation Type does not allow creating lots. |
| Has tracking (`has_tracking`) | boolean, computed, not stored | True when at least one move carries a tracked product. |
| Product Availability (`products_availability`) | text, computed, not stored | See `calculations.md`, section "Transfer availability text". |
| Availability state (`products_availability_state`) | selection, computed, not stored, searchable | Values: `available` "Available", `expected` "Expected", `late` "Late". |
| Show check availability button (`show_check_availability`) | boolean, computed, not stored | False unless the state is waiting, waiting-another-operation or ready; false when every move is picked or already fully processed; otherwise true when at least one move is in state waiting-another-operation, waiting or partially available and has a non-zero demand. |
| Show allocation button (`show_allocation`) | boolean, computed, not stored | See `calculations.md`, section "Reception report visibility". |
| Show next transfers (`show_next_pickings`) | boolean, computed, not stored | True when the destination moves of this Transfer's moves belong to at least one Transfer that is not one of this Transfer's returns. |
| References (`reference_ids`) | many-to-many to Document Reference, related through the moves | Read-only. |
| Date Category (`search_date_category`) | selection, not stored, search only | Values: `before` "Before", `yesterday` "Yesterday", `today` "Today", `day_1` "Tomorrow", `day_2` "The day after tomorrow", `after` "After". See `calculations.md`, section "Date categories". |

Ordering: by priority descending, then scheduled date ascending, then identifier descending.

## 6.5 Deletion

Deleting a Transfer first cancels then deletes its moves; deleting a move whose state is not draft or cancelled and which is chained is refused with "You can not delete moves linked to another operation".

---

# 7. Stock Move

**Stock Move** (`stock.move`, table `stock_move`) is one product's planned travel from a source Location to a destination Location.

## 7.1 Field table — product and quantities

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sequence (`sequence`) | integer | Default 10. Moves are ordered by sequence then identifier. |
| Product (`product_id`) | link to Product | Required, indexed, company scoped. Restricted to goods (not services). |
| Product Category (`product_category_id`) | link to Product Category, related | Read-only. |
| Product Template (`product_tmpl_id`) | link to Product Template, related | Read-only. |
| Never-attribute values (`never_product_template_attribute_value_ids`) | many-to-many to Product Attribute Value | Attribute values that must not appear in the printed description. |
| Description Of Picking (`description_picking`) | long text, computed, not stored, writable | The product's transfer description for this Operation Type; when a manual override exists it is used instead. Writing it stores the manual override. |
| Manual description (`description_picking_manual`) | long text | Read-only from screens; holds the override. |
| Demand (`product_uom_qty`) | decimal, precision Product Unit | Required, default 0, in the line unit. Lowering it never creates a backorder. |
| Real Quantity (`product_qty`) | decimal, computed and stored | The demand converted to the product unit, rounding half away from zero. Writing it directly is refused with "The requested operation cannot be processed because of a programming error setting the `product_qty` field instead of the `product_uom_qty`." |
| Unit (`product_uom`) | link to Unit of Measure | Required, computed and stored, editable. Default: the product's own unit. Restricted to the product's own unit, its alternative units and the units of its vendor price lines. Changing it on a done move is refused with "You cannot change the UoM for a stock move that has been set to 'Done'." |
| Allowed units (`allowed_uom_ids`) | many-to-many to Unit of Measure, computed, not stored | The product's own unit, its alternative units and its vendor units. |
| Quantity (`quantity`) | decimal, computed and stored, writable, precision Product Unit | The sum of the move lines' quantities converted to the line unit without intermediate rounding. Writing it distributes the difference over the move lines. See `calculations.md`, section "Setting a processed quantity". |
| Picked (`picked`) | boolean, computed and stored, writable | Not copied, default false. True when the move is done or any of its lines is picked; false when it has lines and none is picked. Writing it writes the same value on all its lines. |
| Packaging (`packaging_uom_id`) | link to Unit of Measure, computed and stored | Default: the line unit. Carries the packaging unit coming from a sales or purchase document. |
| Packaging Quantity (`packaging_uom_qty`) | decimal, computed and stored | The demand converted from the line unit into the packaging unit. |
| Unit Price (`price_unit`) | decimal | Not copied. Technical field carrying the unit cost in the company currency and in the product unit; consumed by the valuation domain. |
| Forecasted Quantity (`availability`) | decimal, computed, not stored | For a done move, the real quantity. Otherwise the smaller of the real quantity and the total available quantity of the product at the source Location. |
| Forecast Availability (`forecast_availability`) | decimal, computed, not stored, precision Product Unit | See `calculations.md`, section "Move forecast information". |
| Forecasted Expected date (`forecast_expected_date`) | date and time, computed, not stored | See `calculations.md`, section "Move forecast information". |
| Is storable (`is_storable`) | boolean, related to the product | Read-only. |
| Product with Tracking (`has_tracking`) | selection, related to the product's tracking mode | Read-only. Values are `none`, `lot`, `serial`. |

## 7.2 Field table — routing and chaining

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Source Location (`location_id`) | link to Location | Required, indexed, company scoped, computed and stored, editable. Computation: skipped when the move is picked; otherwise the Transfer's source Location if there is a Transfer, else the Operation Type's default source. |
| Intermediate Location (`location_dest_id`) | link to Location | Required, indexed, computed and stored, editable. Computation: the Transfer's destination Location if there is a Transfer; else the rule's destination when the rule says so; else the Operation Type's default destination. Then, if a final Location is set and it is a descendant of that destination — or the destination is a descendant of the shared customer Location while the final Location is the shared inter-company Location — the final Location replaces it. Writing it re-applies put-away on the move lines whose current destination is not already inside the new destination. |
| Final Location (`location_final_id`) | link to Location | Indexed, company scoped. The Location the whole chain is aiming at. |
| Source Location Type (`location_usage`) | selection, related | Read-only. |
| Destination Location Type (`location_dest_usage`) | selection, related | Read-only. |
| Destination Address (`partner_id`) | link to Contact, computed and stored, editable, indexed when not empty | Default: the Transfer's contact. |
| Destination Moves (`move_dest_ids`) | many-to-many to Stock Move | Not copied. The moves that must happen after this one. |
| Original Move (`move_orig_ids`) | many-to-many to Stock Move | Not copied. The moves that must happen before this one. The two relations are two directions of the same link table. |
| Transfer (`picking_id`) | link to Transfer | Indexed, company scoped. |
| Operation Type (`picking_type_id`) | link to Operation Type | Computed and stored, editable, company scoped. Default: the Transfer's Operation Type. |
| Operation kind (`picking_code`) | selection, related | Read-only. |
| Supply Method (`procure_method`) | selection | Required, default `make_to_stock`, not copied. Values: `make_to_stock` "Default: Take From Stock", `make_to_order` "Advanced: Apply Procurement Rules". |
| Stock Rule (`rule_id`) | link to Stock Rule | Company scoped. Deleting the rule is restricted while moves point at it. |
| Preferred route (`route_ids`) | many-to-many to Route | Routes to prefer when the move must be supplied. |
| Warehouse (`warehouse_id`) | link to Warehouse | The Warehouse to consider when selecting the next rule. Recomputed whenever a Location changes: the source Location's Warehouse, or, when it has none, the destination Location's Warehouse. |
| Propagate cancel and split (`propagate_cancel`) | boolean | Default true. |
| Original return move (`origin_returned_move_id`) | link to Stock Move | Not copied, indexed, company scoped. The move this one reverses. |
| All returned moves (`returned_move_ids`) | one-to-many of Stock Move | The moves that reverse this one. |
| References (`reference_ids`) | many-to-many to Document Reference | The shared keys grouping moves that come from the same originating need. Filled from the Transfer when empty. |
| Original Reordering Rule (`orderpoint_id`) | link to Reordering Rule | Indexed. Set when the move was created by the replenishment scheduler. |
| Procurement values (`procurement_values`) | structured data, not stored | Carries the supply request values forward to later steps of the chain. |

## 7.3 Field table — dates, state and flags

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Date Scheduled (`date`) | date and time | Required, default now, indexed. Before completion it is the planned date; from completion it is the actual processing date. Writing it on a done move also writes it on the move lines. |
| Deadline (`date_deadline`) | date and time | Read-only, not copied. Writing it propagates along the chain: see `calculations.md`, section "Deadline propagation". |
| Delay Alert Date (`delay_alert_date`) | date and time, computed and stored | Empty for done and cancelled moves. Otherwise the maximum date among the originating moves that are neither done nor cancelled, when that maximum is later than this move's own date; empty when it is not. |
| Date to Reserve (`reservation_date`) | date, computed and stored | For an Operation Type reserving before the scheduled date and a move in state draft, waiting, waiting-another-operation or partially available: the move's date reduced by the Operation Type's number of days (the urgent number when the move's priority is urgent). For an Operation Type reserving manually: empty. For an Operation Type reserving at confirmation the value is stamped as today's date when the move is confirmed. |
| Priority (`priority`) | selection, computed and stored | Default `0`. Values: `0` "Normal", `1` "Urgent". Taken from the Transfer. |
| Status (`state`) | selection | Read-only, not copied, indexed, default `draft`. Values: `draft` "New", `waiting` "Waiting Another Move", `confirmed` "Waiting", `partially_available` "Partially Available", `assigned` "Available", `done` "Done", `cancel` "Cancelled". See `state-machines.md`. |
| Inventory (`is_inventory`) | boolean | True for the moves generated by an inventory adjustment or by a relocation. |
| Inventory name (`inventory_name`) | text | Read-only. The label shown as the reference of an adjustment move. |
| Owner (`restrict_partner_id`) | link to Contact | Company scoped, indexed when not empty. Restricts which owner's goods the move may consume. |
| Scrap operation (`scrap_id`) | link to Scrap | Read-only, company scoped, indexed when not empty. |
| Whether the move was added after the picking's confirmation (`additional`) | boolean | Default false. |
| Company (`company_id`) | link to Company | Required, default: the active company, indexed. |
| Source Document (`origin`) | text | Free text. |
| Reference (`reference`) | text, computed and stored | For a scrap move, the scrap document name; for an adjustment move, the inventory name when set, otherwise "Product Quantity Confirmed" when the processed quantity is zero or "Product Quantity Updated" when it is not, followed by the creating user's display name in parentheses unless the creator is the system user; otherwise the Transfer's reference. |
| Move lines (`move_line_ids`) | one-to-many of Stock Move Line | The detail lines. |
| Move line count (`move_lines_count`) | integer, computed, not stored | The number of detail lines. |
| Packages (`package_ids`) | one-to-many of Package, computed, not stored | For a done or cancelled move with package history, the outermost destination containers recorded in that history; otherwise the outermost destination containers of the move lines. |
| Serial Numbers (`lot_ids`) | many-to-many to Lot, computed, not stored, writable | The lots of the move lines that have a lot and a non-zero quantity. Writing it rebuilds the move lines: see `calculations.md`, section "Assigning lots on a move". |
| First SN/Lot (`next_serial`) | text | The first serial number of a generated series. |
| Number of SN/Lots (`next_serial_count`) | integer | How many serial numbers to generate. |
| Locked (`is_locked`) | boolean, computed, not stored | The Transfer's lock flag, or false when there is no Transfer. |
| Is initial demand editable (`is_initial_demand_editable`) | boolean, computed, not stored | True when the Transfer is not locked or the move is in draft. |
| Is Date Editable (`is_date_editable`) | boolean, computed, not stored | The Transfer's value, or true when there is no Transfer. |
| Is quantity done editable (`is_quantity_done_editable`) | boolean, computed, not stored | True when a product is set. |
| Details Visible (`show_details_visible`) | boolean, computed, not stored | See `calculations.md`, section "Detail button visibility". |
| Show quantity picker (`show_quant`) | boolean, computed, not stored | True when the Operation Type kind is not receipt and the product is storable. |
| Show lot selector (`show_lots_m2o`) | boolean, computed, not stored | True when the quantity picker is hidden, the lot text box is hidden, the product is tracked, and either existing lots may be used, the move is done, or the move is a return. |
| Show lot text box (`show_lots_text`) | boolean, computed, not stored | True when the product is tracked, the Operation Type allows creating lots but not using existing ones, the move is not done and the move is not a return. |
| Show assign serial button (`display_assign_serial`) and Show import lot button (`display_import_lot`) | boolean, computed, not stored | Both true when the product is tracked, a product is set, the Operation Type allows creating lots, the move is not a return and its state is neither done nor cancelled. |
| Lines without destination container (`has_lines_without_result_package`) | boolean, computed, not stored | True when at least one line has a destination container and at least one has none. |

An extra database index exists on the tuple (product, source Location, destination Location, company, state).

## 7.4 Direction predicates

- A move is **incoming** when its source Location usage is customer or vendor, or its source Location usage is transit and that Location has no company.
- A move is **outgoing** when its destination Location usage is customer or vendor, or its destination Location usage is transit and that Location has no company.
- A move **counts for received quantity** when its source Location usage is vendor or transit.
- A move is **consuming** when its Operation Type kind is internal transfer or delivery, or its source Warehouse and destination Warehouse both exist and differ.
- A move **bypasses reservation** when its source Location bypasses reservation (usage vendor, customer, inventory loss or production) or its product is not storable.

---

# 8. Stock Move Line

**Stock Move Line** (`stock.move.line`, table `stock_move_line`) is one concrete reservation or execution detail of a Stock Move.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Transfer (`picking_id`) | link to Transfer | Indexed, company scoped. |
| Stock Operation (`move_id`) | link to Stock Move | Indexed, company scoped. When a line is created with a Transfer but no move, the system links it to an existing move of the same Transfer and product — preferring one whose processed quantity is still below its real quantity — or creates a new move for it. |
| Company (`company_id`) | link to Company | Required, read-only, indexed. Taken from the move, else from the Transfer. |
| Product (`product_id`) | link to Product | Indexed, company scoped. Deleting the product deletes the line. Not a service. Changing it is only allowed while the line is in draft: "Changing the product is only allowed in 'Draft' state." |
| Unit (`product_uom_id`) | link to Unit of Measure | Required, computed and stored, editable. Default: the move's unit, else the product's own unit. Restricted to the allowed units of the product. |
| Allowed units (`allowed_uom_ids`) | many-to-many to Unit of Measure, computed, not stored | Same rule as on the move. |
| Product Category (`product_category_name`) | text, related | Read-only. |
| Quantity (`quantity`) | decimal, computed and stored, writable, precision Product Unit | Not copied. In the line unit. Must be greater than or equal to zero: "You can not enter negative quantities." Computed from a chosen quantity record when one is picked: see `calculations.md`, section "Quantity from a picked quantity record". |
| Quantity in Product UoM (`quantity_product_uom`) | decimal, computed and stored, precision Product Unit | Not copied. The quantity converted to the product unit, rounding half away from zero. |
| Picked (`picked`) | boolean, computed and stored, writable | Not copied. Forced true when the move is done or the screen asked for automatic picking. On creation it defaults to the move's own picked flag. |
| Source Package (`package_id`) | link to Package | Company scoped. Deletion of the container is restricted. Restricted to containers located in the line's source Location. |
| Lot/Serial Number (`lot_id`) | link to Lot | Indexed, company scoped. Restricted to lots of the line's product. |
| Lot/Serial Number Name (`lot_name`) | text | A typed lot or serial number that does not yet exist as a record. |
| Destination Package (`result_package_id`) | link to Package | Company scoped. Deletion of the container is restricted. Restricted to containers located in the destination Location, or the source container itself, or a container with no Location that is either unused or already destined to the same destination Location. |
| Destination Package Name (`result_package_dest_name`) | text, related to the destination container's destination full name | Read-only. |
| Package History (`package_history_id`) | link to Package History | Indexed when not empty. Filled at completion. |
| Is added through entire package (`is_entire_pack`) | boolean | True when the line exists because a whole container was added to the Transfer. |
| Date (`date`) | date and time | Required, default now. Re-stamped with the current instant when the quantity is increased, when the line becomes picked, and at completion. |
| Scheduled Date (`scheduled_date`) | date and time, related to the move's date | Read-only. |
| From Owner (`owner_id`) | link to Contact | Company scoped, indexed when not empty. |
| From (`location_id`) | link to Location | Required, indexed, company scoped, computed and stored, editable. Default: the move's source Location, else the Transfer's. Usage may not be virtual. |
| To (`location_dest_id`) | link to Location | Required, indexed, company scoped, computed and stored, editable. Default: the move's destination Location, else the Transfer's. Usage may not be virtual. |
| Source Location Type (`location_usage`) and Destination Location Type (`location_dest_usage`) | selection, related | Read-only. |
| Show lot fields (`lots_visible`) | boolean, computed, not stored | When the Transfer has an Operation Type and the product is tracked: true when the Operation Type allows existing or new lots. Otherwise: true when the product is tracked. |
| Operation type (`picking_type_id`) | link to Operation Type, computed, not stored, searchable | The Transfer's Operation Type. |
| Operation kind (`picking_code`), create-lots allowed (`picking_type_use_create_lots`), existing-lots allowed (`picking_type_use_existing_lots`) | related | Read-only. |
| Status (`state`) | selection, related to the move's state, stored | Read-only. |
| Scrap operation (`scrap_id`), Inventory (`is_inventory`), Locked (`is_locked`), Reference (`reference`), Tracking (`tracking`), Source (`origin`), Description (`description_picking`) | related | Read-only mirrors of the move's fields. |
| Consumed lines (`consume_line_ids`) and Produced lines (`produce_line_ids`) | many-to-many to Stock Move Line | The two directions of the production genealogy link used by traceability. |
| Pick From (`quant_id`) | link to Stock Quantity, not stored | A helper used by the detail screen: choosing a quantity record copies its product, lot, container, Location and owner onto the line. |
| Transfer source (`picking_location_id`) and Transfer destination (`picking_location_dest_id`) | related | Read-only. |

Ordering: by destination container descending, then identifier ascending.

A partial database index exists on the tuple (identifier, company, product, lot, source Location, owner, source container) restricted to lines that are neither cancelled nor done, have a positive quantity in the product unit and are not picked; it serves the reservation-freeing search.

Uniqueness: none enforced. A line is identified by the combination of its characteristics only for the purpose of matching quantity records.

---

# 9. Stock Quantity

**Stock Quantity** (`stock.quant`, table `stock_quant`) records the on-hand quantity of one product in one Location with one lot, one container and one owner.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | link to Product | Required, indexed, company scoped. Deletion of the product is restricted. Must be storable: "Quants cannot be created for consumables or services." |
| Product Template (`product_tmpl_id`), Unit (`product_uom_id`), Tracking (`tracking`), Product Category (`product_categ_id`), Favourite (`is_favorite`) | related | Read-only. |
| Location (`location_id`) | link to Location | Required, indexed. Deletion of the Location is restricted. Usage may not be virtual: "You cannot take products from or deliver products to a location of type \"view\" (*the location name*)." |
| Company (`company_id`) | link to Company, related to the Location, stored | Read-only, indexed. |
| Warehouse (`warehouse_id`), Storage Category (`storage_category_id`), Inventory Frequency (`cyclic_inventory_frequency`) | related to the Location | Read-only. |
| Lot/Serial Number (`lot_id`) | link to Lot | Indexed, company scoped. Deletion of the lot is restricted. The lot's product, when set, must equal the record's product: "The Lot/Serial number (*the lot name*) is linked to another product." |
| Lot properties (`lot_properties`) | properties, related | Read-only. |
| Duplicated Serial Number (`sn_duplicated`) | boolean, computed, not stored | True when the same serial number appears on more than one quantity record with a positive quantity in an internal or transit Location. |
| Package (`package_id`) | link to Package | Indexed, company scoped. Deletion of the container is restricted. Restricted to containers in the same Location, or containers with no Location and no contents. |
| Owner (`owner_id`) | link to Contact | Company scoped, indexed when not empty. |
| Quantity (`quantity`) | decimal, read-only, precision Product Unit | The physical quantity, in the product unit. May be negative. |
| Reserved Quantity (`reserved_quantity`) | decimal, required, read-only, default 0, precision Product Unit | How much of the quantity is promised to open documents, in the product unit. |
| Available Quantity (`available_quantity`) | decimal, computed, not stored, precision Product Unit | Quantity minus reserved quantity. |
| Incoming Date (`in_date`) | date and time, required, read-only, default now | The arrival instant used by the first in first out and last in first out orderings. |
| On Hand (`on_hand`) | boolean, not stored, search only | Selects the records located in the Locations the product-quantity computation considers on hand. |
| Counted (`inventory_quantity`) | decimal, precision Product Unit | The quantity a person counted. |
| Inventoried Quantity (`inventory_quantity_auto_apply`) | decimal, computed, writable, precision Product Unit | Reads back the on-hand quantity; writing it sets the counted quantity and applies the adjustment immediately. Restricted to the inventory user group. |
| Difference (`inventory_diff_quantity`) | decimal, computed and stored, read-only, precision Product Unit | Counted quantity minus on-hand quantity when a count has been entered; zero otherwise. |
| Scheduled (`inventory_date`) | date, computed and stored, editable | The next date this record should be counted. Computed only for records with no date yet in internal or transit Locations. |
| Last count date (`last_count_date`) | date, computed, not stored | See `calculations.md`, section "Last count date". |
| Counted flag (`inventory_quantity_set`) | boolean, computed and stored, editable | Turned true whenever a counted quantity is written. |
| Quantity has been moved since last count (`is_outdated`) | boolean, computed, not stored, searchable | True when a count has been entered and the counted quantity minus the recorded difference no longer equals the on-hand quantity. |
| Assigned To (`user_id`) | link to User | Restricted to users in the inventory user group. |

Ordering: default order by identifier. Display name: the Location display name, then the lot name, the container display name and the owner name when present, joined by " - ". Text search matches the Location, the lot, the container or the owner.

Duplication is refused outright: "You cannot duplicate stock quants." Creating a record by typing a name is refused.

Deletion by a non-manager is refused: "Quants are auto-deleted when appropriate. If you must manually delete them, please ask a stock manager to do it." A manager deleting a record instead sets its counted quantity to zero and applies the adjustment, so that the removal is traced by moves.

Constraint on serial numbers: for every serial-tracked product, the sum of quantities over the descendants of a Location for one serial number may not exceed one in absolute value, else "The serial number has already been assigned: \n Product: *the product display name*, Serial Number: *the serial number*". Records in inventory-loss Locations are excluded from the check.

---

# 10. Lot or Serial Number

**Lot** (`stock.lot`, table `stock_lot`) identifies a batch, or — when only one unit exists — a single item. It carries a discussion thread and activity scheduling.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Lot/Serial Number (`name`) | text | Required, computed and stored, editable, trigram-indexed. When left empty and the product has a lot numbering sequence, the next number of that sequence is drawn. On copy becomes "(copy of) " followed by the original. |
| Internal Reference (`ref`) | text | The manufacturer's own number when it differs. |
| Product (`product_id`) | link to Product | Required, indexed, company scoped, tracked. Restricted to storable tracked products. Changing it is refused when move lines already exist for the lot with a different product: "You are not allowed to change the product linked to a serial or lot number if some stock moves have already been created with that number. This would lead to inconsistencies in your stock." |
| Unit (`product_uom_id`) | link to Unit of Measure, related | Read-only. |
| Quantities (`quant_ids`) | one-to-many of Stock Quantity | Read-only. |
| On Hand Quantity (`product_qty`) | decimal, computed, not stored, searchable | See `calculations.md`, section "Lot on-hand quantity". |
| Description (`note`) | rich text | Free text. |
| Company (`company_id`) | link to Company | Computed and stored, editable, indexed. Rule: when the active company is a descendant of the product's company and the product's company is not among the allowed companies, the active company; otherwise the product's company (possibly empty). Changing it is refused when the lot currently sits in a Location belonging to a different company: "You cannot change the company of a lot/serial number currently in a location belonging to another company." |
| Transfers (`delivery_ids`) and Delivery order count (`delivery_count`) | computed, not stored | The completed outgoing Transfers that carried this lot, following production genealogy upwards. See `calculations.md`, section "Delivery discovery". |
| Contacts (`partner_ids`) | many-to-many to Contact, computed, not stored, searchable | The contacts of those Transfers, newest completion date first. |
| Properties (`lot_properties`) | properties | Schema taken from the product. Copied. |
| Location (`location_id`) | link to Location | Computed and stored, editable, grouped in lists by partner Locations and warehouse stock Locations. Computed as the single Location holding a positive quantity of this lot, or empty when there is none or more than one. Writing it relocates every positive quantity record of the lot to the new Location, unpacking when the source container also holds other lots; writing it when the lot is in more than one Location is refused with "You can only move a lot/serial to a new location if it exists in a single location." |
| Show all fields (`display_complete`) | boolean, computed, not stored | True once the record exists. |

Ordering: by name, then identifier.

Uniqueness: the combination of product and name must be unique within a company, and also unique against company-less lots. Violation message: "The combination of lot/serial number and product must be unique within a company including when no company is defined.\nThe following combinations contain duplicates:\n" followed by one line per duplicate of the form " - Product: *the product display name*, Lot/Serial Number: *the name*".

Creation restriction: when the creation happens from a Transfer screen whose Operation Type does not allow creating lots, it is refused with "You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box \"Create New Lots/Serial Numbers\"."

---

# 11. Package

**Package** (`stock.package`, table `stock_package`) is one physical container. Containers nest: a container may hold quantity records, other containers, or both.

Two distinct parent links exist and must not be confused:

- **Container** (`parent_package_id`) — where the container *is now*. Materialised in a path column.
- **Destination Container** (`package_dest_id`) — where the container *will be* once the Transfer that is moving it is completed. Not materialised; walked explicitly.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Package Reference (`name`) | text | Required, not copied, trigram-indexed. When created without a name, the next number of the Package Type's numbering sequence is drawn; when the name is cleared on write, it is redrawn the same way. |
| Full Package Name (`complete_name`) | text, computed and stored, recursive | The parent container's full name, " > ", and this name; or just this name when there is no parent. |
| Package Name At Destination (`dest_complete_name`) | text, computed, not stored, recursive | The same construction along the destination-container chain. |
| Bulk Content (`quant_ids`) | one-to-many of Stock Quantity | Read-only. Restricted to records with a non-zero quantity or a non-zero reservation. |
| Contents (`contained_quant_ids`) | one-to-many of Stock Quantity, computed, not stored, searchable | The container's own quantity records plus those of all its descendants. |
| Contents description (`content_description`) | text, computed, not stored | A human list of "quantity unit product" entries, one per pair of unit and product, the unit omitted when the multiple-units group is off. |
| Package Type (`package_type_id`) | link to Package Type | Indexed. |
| Location (`location_id`) | link to Location | Computed and stored, editable, indexed, recursive. The Location of the first quantity record with a positive quantity; when there is none, the Location of the first child container. Writing it relocates the contents: clearing it on a non-empty container is refused with "Cannot remove the location of a non empty package"; setting it on an empty container is refused with "Cannot move an empty package". |
| Destination location (`location_dest_id`) | link to Location, computed, not stored, searchable | The destination Location of the container's first open move line. |
| Company (`company_id`) | link to Company, computed and stored, read-only, indexed, recursive | The common company of the contents when they all agree, otherwise empty. |
| Owner (`owner_id`) | link to Contact, computed, not stored, searchable | The common owner of the quantity records when they all agree, otherwise empty. |
| Container (`parent_package_id`) | link to Package | Indexed when not empty. |
| Contained Packages (`child_package_ids`) | one-to-many of Package | The direct children. |
| All contained packages (`all_children_package_ids`) | one-to-many of Package, computed, not stored, searchable | Every descendant. |
| Destination Container (`package_dest_id`) | link to Package | Indexed when not empty. Setting a descendant of the destination chain as destination is refused with "A package can't have one of its contained packages as destination container." |
| Assigned Contained Packages (`child_package_dest_ids`) | one-to-many of Package | The containers that name this one as their destination container. |
| Outermost Destination Container (`outermost_package_id`) | link to Package, computed, not stored, searchable, recursive | The last container of the destination chain; the container itself when it has no destination container. |
| Operations (`move_line_ids`) | one-to-many of Stock Move Line, computed, not stored, searchable | Every open move line whose destination container is this container or any container that has it (directly or transitively) as destination container. |
| Transfers (`picking_ids`) | many-to-many to Transfer, computed, not stored, searchable | The Transfers of those move lines. |
| Shipping Weight (`shipping_weight`) | decimal | A manually entered total weight that overrides the computed one. |
| Package name is valid SSCC (`valid_sscc`) | boolean, computed, not stored | True when the name passes the serial shipping container code check. |
| Pack Date (`pack_date`) | date | Default today. |
| Path (`parent_path`) | text, indexed | The materialised containment path. |
| Issues popover (`json_popover`) | text, computed, not stored | Present when the container's open move lines point at more than one destination Location; carries the title "Multiple destinations" and the message "This package is currently set to be sent in *the list of location names*." |

Ordering: by name, then identifier. The record is named by its full name; the displayed name is the plain name, except in the contexts that ask for the source path or the destination path.

---

# 12. Package Type

**Package Type** (`stock.package.type`, table `stock_package_type`) is a reusable container specification.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Package Type (`name`) | text | Required. Unique; violation message: "A package type already exists with this name". |
| Sequence (`sequence`) | integer | Default 1. Orders the list. |
| Height (`height`), Width (`width`), Length (`packaging_length`) | integer | Each must be positive; violation messages "Height must be positive", "Width must be positive", "Length must be positive". |
| Base Weight (`base_weight`) | decimal | The empty container's own weight. |
| Max Weight (`max_weight`) | decimal | The heaviest total the container may carry. |
| Barcode (`barcode`) | text | Not copied. Unique; violation message: "A barcode can only be assigned to one package type!". |
| Company (`company_id`) | link to Company | Indexed. |
| Routes (`route_ids`) | many-to-many to Route | Restricted to Routes flagged selectable on a package type. |
| Package Use (`package_use`) | selection | Required, default `disposable`. Values: `disposable` "Disposable Box" (the container travels with the goods and is consumed), `reusable` "Reusable Box" (the container stays; goods are taken out of it). A reusable container is never taken over as an entire package. |
| Sequence for names (`sequence_id`) | link to Numbering Sequence | The numbering used when a container of this type is created without a name. |
| Dimension unit (`length_uom_name`) and Weight unit (`weight_uom_name`) | text, computed, not stored | Display names for the configured length and weight units. |

---

# 13. Package History

**Package History** (`stock.package.history`, table `stock_package_history`) freezes what a container was at the moment a Transfer that moved it was completed. It exists so that a completed Transfer can still be printed and audited after the container has been re-used or dismantled.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Package (`package_id`) | link to Package | The container the record describes. |
| Package name (`package_name`) | text | The container's full name at that instant. |
| Source location (`location_id`) and Destination location (`location_dest_id`) | link to Location | Where the container came from and went to. |
| Parent before (`parent_orig_id`) and its name (`parent_orig_name`) | link to Package, text | The container's parent container before the move. |
| Parent after (`parent_dest_id`) and its name (`parent_dest_name`) | link to Package, text | The container's destination container before it was applied. |
| Outermost destination (`outermost_dest_id`) | link to Package | The last container of the destination chain at that instant. |
| Move lines (`move_line_ids`) | one-to-many of Stock Move Line | The lines whose destination container was this container. |
| Transfers (`picking_ids`) | many-to-many to Transfer | The Transfers those lines belonged to. |

---

# 14. Put-away Rule

**Put-away Rule** (`stock.putaway.rule`, table `stock_putaway_rule`) redirects goods arriving in one Location to a sublocation.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Product (`product_id`) | link to Product | Indexed when not empty, company scoped. Deleting the product deletes the rule. Default: the product in context when the rule is created from a product screen. |
| Product Category (`category_id`) | link to Product Category | Indexed when not empty. Deleting the category deletes the rule. Restricted to categories flagged as usable for put-away. Default: the category in context. |
| When product arrives in (`location_in_id`) | link to Location | Required, indexed, company scoped. Deleting the Location deletes the rule. Restricted to Locations that have children. Default: the Location in context; when the user has no multi-warehouse group, the input Location of the company's first Warehouse. |
| Store to sublocation (`location_out_id`) | link to Location | Required, company scoped. Deleting the Location deletes the rule. Restricted to descendants of the arrival Location. When the arrival Location changes and the current target is not inside it, the target is reset to the arrival Location. |
| Priority (`sequence`) | integer | Rules are ordered by priority then product. |
| Company (`company_id`) | link to Company | Required, default: the active company, indexed. Changing it is refused with "Changing the company of this record is forbidden at this point, you should rather archive it and create a new one." |
| Package Type (`package_type_ids`) | many-to-many to Package Type | Company scoped. When set, the rule only applies to goods travelling in a container of one of those types. |
| Storage Category (`storage_category_id`) | link to Storage Category | Computed and stored, editable, company scoped. Deleting the category deletes the rule. Forced empty unless the sublocation mode is closest-location. |
| Sublocation mode (`sublocation`) | selection | Default `no`. Values: `no` "No", `last_used` "Last Used", `closest_location` "Closest Location". |
| Active (`active`) | boolean | Default true. |

Warning: choosing the closest-location mode with a storage category that no descendant of the target Location carries shows "Selected storage category does not exist in the 'store to' location or any of its sublocations".

---

# 15. Removal Strategy

**Removal Strategy** (`product.removal`, table `product_removal`) names a method for ordering quantity records when goods leave.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text, translated | Required. The label shown to users. |
| Method (`method`) | text, translated | Required. The reproduced key used by the ordering algorithm. The shipped values are `fifo`, `lifo`, `closest` and `least_packages`; a companion capability adds `fefo`. |

The ordering each method imposes is specified in `calculations.md`, section "Removal strategies".

---

# 16. Storage Category and Storage Category Capacity

**Storage Category** (`stock.storage.category`, table `stock_storage_category`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Storage Category (`name`) | text | Required. On copy becomes the original followed by " (copy)". |
| Max Weight (`max_weight`) | decimal, precision Stock Weight | Must be greater than or equal to zero; violation message: "Max weight should be a positive number." |
| Capacities (`capacity_ids`) | one-to-many of Storage Category Capacity | Copied. |
| Product capacities (`product_capacity_ids`) and Package capacities (`package_capacity_ids`) | computed, not stored, writable | The capacities that name a product and those that name a Package Type. Writing either rebuilds the full capacity list as the union of the two. |
| Allow new product (`allow_new_product`) | selection | Required, default `mixed`. Values: `empty` "If the location is empty", `same` "If all products are same", `mixed` "Allow mixed products". |
| Locations (`location_ids`) | one-to-many of Location | The Locations that use this category. |
| Company (`company_id`) | link to Company | Optional. |
| Weight unit (`weight_uom_name`) | text, computed, not stored | Display name of the configured weight unit. |

Ordering: by name.

**Storage Category Capacity** (`stock.storage.category.capacity`, table `stock_storage_category_capacity`)

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Storage category (`storage_category_id`) | link to Storage Category | Required, indexed. Deleting the category deletes the capacity. |
| Product (`product_id`) | link to Product | Company scoped, indexed when not empty. Deleting the product deletes the capacity. Restricted to storable products. |
| Package Type (`package_type_id`) | link to Package Type | Company scoped, indexed when not empty. Deleting the type deletes the capacity. |
| Quantity (`quantity`) | decimal | Required, must be strictly positive: "Quantity should be a positive number." |
| Unit (`product_uom_id`) | link to Unit of Measure, related | Read-only. |
| Company (`company_id`) | link to Company, related | Read-only. |

Uniqueness: at most one capacity per product and category ("Multiple capacity rules for one product.") and at most one per Package Type and category ("Multiple capacity rules for one package type.").

Ordering: by storage category.

---

# 17. Scrap and Scrap Reason Tag

**Scrap** (`stock.scrap`, table `stock_scrap`) records the removal of damaged goods.

**Scrap Reason Tag** (`stock.scrap.reason.tag`, table `stock_scrap_reason_tag`) is a free tag qualifying the reason.

A Scrap carries a discussion thread. Ordering: by identifier descending.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Reference (`name`) | text | Required, read-only, not copied. Default: the literal word "New". Replaced at validation by the next number of the scrap numbering sequence, or by "New" when no sequence exists. |
| Company (`company_id`) | link to Company | Required, default: the active company. |
| Source Document (`origin`) | text | Free text. |
| Product (`product_id`) | link to Product | Required, company scoped. Goods only (not services). |
| Unit (`product_uom_id`) | link to Unit of Measure | Required, computed and stored, editable. Default: the product's own unit. Restricted to the allowed units of the product. |
| Allowed units (`allowed_uom_ids`) | many-to-many to Unit of Measure, computed, not stored | The product's own unit, its alternative units and its vendor units. |
| Product Tracking (`tracking`) | selection, related | Read-only. |
| Lot/Serial (`lot_id`) | link to Lot | Company scoped, restricted to lots of the product. |
| Package (`package_id`) | link to Package | Company scoped. |
| Owner (`owner_id`) | link to Contact | Company scoped. |
| Moves (`move_ids`) | one-to-many of Stock Move | The single move created at validation. |
| Picking (`picking_id`) | link to Transfer | Company scoped. Set when the scrap was started from a Transfer. |
| Source Location (`location_id`) | link to Location | Required, computed and stored, editable, company scoped. Restricted to internal usage. Rule: when a Transfer is set, that Transfer's destination Location if the Transfer is done, otherwise its source Location; when no Transfer is set, the stock Location of the company's first Warehouse. When the company has no Warehouse the user is redirected to the Warehouse screen. |
| Scrap Location (`scrap_location_id`) | link to Location | Required, computed and stored, editable, company scoped. Restricted to inventory-loss usage. Default: the lowest-numbered inventory-loss Location of the company. |
| Quantity (`scrap_qty`) | decimal, precision Product Unit | Required, computed and stored, editable, default 1. When a move already exists the value is read back from that move's processed quantity. |
| Status (`state`) | selection | Read-only, tracked, default `draft`. Values: `draft` "Draft", `done` "Done". |
| Date (`date_done`) | date and time | Read-only. Stamped at validation. |
| Replenish Quantities (`should_replenish`) | boolean | When true, a supply request for the scrapped quantity at the source Location is run right after the scrap. |
| Scrap Reason (`scrap_reason_tag_ids`) | many-to-many to Scrap Reason Tag | Free qualification. |

Deleting a completed Scrap is refused with "You cannot delete a scrap which is done."

**Scrap Reason Tag** fields: Name (`name`, required, translated, unique — violation message "Tag name already exists!"), Sequence (`sequence`, default 10), Color (`color`, text, default `#3C3C3C`). Ordering: by sequence then identifier.

---

# 18. Document Reference

**Document Reference** (`stock.reference`, table `stock_reference`) is the shared key that links every Stock Move created to satisfy the same originating need, across as many Transfers as the chain requires. Moves with the same reference are grouped into one Transfer when their Locations and Operation Type also agree.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Name (`name`) | text | The human label of the originating need, usually the document number that created it. |
| Moves (`move_ids`) | many-to-many to Stock Move | The moves that share the reference. |

---

# 19. Batch Transfer

**Batch Transfer** (`stock.picking.batch`, table `stock_picking_batch`) groups Transfers so that they are picked, validated and printed together. A batch whose wave flag is set was built from a selection of moves rather than of whole Transfers.

It carries a discussion thread and activity scheduling. Ordering: by name descending.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Batch Transfer (`name`) | text | Required, read-only, not copied. Default: the literal word "New". At creation, when an Operation Type is given, the name is built by the naming rule below. |
| Description (`description`) | text | Free text. Automatic batches and waves receive a generated description naming the grouping criteria that produced them. |
| Responsible (`user_id`) | link to User | Tracked, company scoped. Writing it re-assigns the responsible on every Transfer of the batch and posts a note on each: "Assigned to *link to the batch* Responsible" or "Unassigned responsible from *link to the batch*". |
| Company (`company_id`) | link to Company | Required, read-only, indexed, default: the active company. |
| Transfers (`picking_ids`) | one-to-many of Transfer (through the batch link on the Transfer) | Company scoped. Restricted to the allowed Transfers. |
| Allowed transfers (`allowed_picking_ids`) | one-to-many of Transfer, computed, not stored | The Transfers of the same company whose state is waiting-another-operation, waiting or ready — plus draft ones when the batch itself is draft — and, when the batch has an Operation Type, of that Operation Type. |
| Show Check Availability (`show_check_availability`) | boolean, computed, not stored | True when at least one move of the batch is not ready, cancelled or done. |
| Show Allocation Button (`show_allocation`) | boolean, computed, not stored | Same rule as on a Transfer, evaluated over all the Transfers of the batch. |
| Stock moves (`move_ids`) | one-to-many of Stock Move, computed, not stored | The moves of the Transfers. |
| Stock move lines (`move_line_ids`) | one-to-many of Stock Move Line, computed, not stored, writable, searchable | The lines of the Transfers. Writing the list re-dispatches each line to its own Transfer and deletes the lines that were removed. |
| Status (`state`) | selection, computed and stored, required, read-only, tracked, indexed, not copied | Default `draft`. Values: `draft` "Draft", `in_progress` "In progress", `done` "Done", `cancel` "Cancelled". See `state-machines.md`. |
| Operation Type (`picking_type_id`) | link to Operation Type | Company scoped, indexed, not copied. Set from the first Transfer when Transfers are added and none was set. Changing it renames the batch and re-runs the composition check. |
| Warehouse (`warehouse_id`), Operation kind (`picking_type_code`) | related | Read-only. |
| Scheduled Date (`scheduled_date`) | date and time, computed and stored, editable, not copied | The earliest scheduled date among the Transfers. Setting it by hand pushes the same date onto every Transfer of the batch. |
| This batch is a wave (`is_wave`) | boolean | True for waves. A wave and a batch can never be merged with each other. |
| Show lot text box (`show_lots_text`) | boolean, computed, not stored | Taken from the first Transfer. |
| Estimated shipping weight (`estimated_shipping_weight`) | decimal, computed, not stored, precision Product Unit | See `calculations.md`, section "Batch load". |
| Estimated shipping volume (`estimated_shipping_volume`) | decimal, computed, not stored, precision Product Unit | See `calculations.md`, section "Batch load". |
| Properties (`properties`) | properties | Schema taken from the Operation Type's batch property definition. Copied. |

Naming rule: draw the next number of the sequence whose code is `picking.batch` for a batch or `picking.wave` for a wave, in the batch's company. Split that number at its last slash. When there is no slash, the name is the Operation Type's sequence prefix, a slash, and the number, and a note is posted on the batch: "The sequence '*the sequence code*' is misconfigured. Its prefix should end with a '/' separator." Otherwise the name is the part before the last slash, a slash, the Operation Type's sequence prefix, a slash, and the part after the last slash.

Deleting a completed batch is refused with "You cannot delete Done batch transfers."

Composition check: every Transfer of the batch must be among the allowed Transfers, else "The following transfers cannot be added to batch transfer *the batch name*. Please check their states and operation types.\n\nIncompatibilities: *the list of transfer references*".

## 19.1 Fields this capability adds to other entities

On **Transfer**: Batch Transfer (`batch_id`, link to Batch Transfer, company scoped, indexed, not copied) and Sequence (`batch_sequence`, integer) which orders the Transfers inside the batch.

On **Stock Move Line**: Batch Transfer (`batch_id`, related through the Transfer, read-only).

On **Operation Type**:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| `count_picking_batch` | integer, computed | Open batches of this Operation Type that are not waves. |
| `count_picking_wave` | integer, computed | Open waves of this Operation Type. |
| Automatic Batches (`auto_batch`) | boolean | Turns automatic grouping on. |
| Contact (`batch_group_by_partner`) | boolean | Group Transfers by contact. |
| Destination Country (`batch_group_by_destination`) | boolean | Group Transfers by the contact's country. |
| Group by Source Location (`batch_group_by_src_loc`) | boolean | Group Transfers by source Location. |
| Group by Destination Location (`batch_group_by_dest_loc`) | boolean | Group Transfers by destination Location. |
| Product (`wave_group_by_product`) | boolean | Split Transfers per product and group the pieces by product. |
| Product Category (`wave_group_by_category`) | boolean | Split per product category and group by category. |
| Wave Product Categories (`wave_category_ids`) | many-to-many to Product Category | The categories considered when grouping waves. |
| Location (`wave_group_by_location`) | boolean | Split per configured Location and group by Location. |
| Wave Locations (`wave_location_ids`) | many-to-many to Location | The internal Locations considered when grouping waves. |
| Maximum lines (`batch_max_lines`) | integer | A Transfer is not added automatically to a batch when doing so would push the number of moves above this. Zero means no limit. |
| Maximum transfers (`batch_max_pickings`) | integer | The same for the number of Transfers. Zero means no limit. |
| Auto-confirm (`batch_auto_confirm`) | boolean | Default true. Confirms a newly created automatic batch or wave immediately. |
| Batch Properties (`batch_properties_definition`) | properties definition | The schema of the batch's free properties. |

Validation: when automatic batching is on, at least one grouping option (batch or wave) must be chosen, else "If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected."

---

# 20. Dispatch management

The dispatch capability adds vehicle and dock handling to Batch Transfers.

On **Operation Type**: Dispatch Management (`dispatch_management`, boolean) and Docks (`dock_ids`, many-to-many to Location, computed and stored, editable, restricted to internal Locations of the Operation Type's Warehouse; cleared when the Warehouse changes). Warehouse generation sets dispatch management on the receipt type, on the delivery type, and on the pick type for a two-step delivery or the pack type for a three-step delivery; it also links the output Location as a dock of the delivery type when deliveries take more than one step.

On **Transfer**: Zip (`zip`, text, related to the contact's postal code, searchable).

On **Batch Transfer**:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Vehicle (`vehicle_id`) | link to Vehicle | The vehicle carrying the batch. |
| Vehicle Category (`vehicle_category_id`) | link to Vehicle Model Category, computed and stored, editable | Default: the vehicle's category. |
| Allowed Docks (`allowed_dock_ids`) | many-to-many to Location, related to the Operation Type's docks | Read-only. |
| Dock (`dock_id`) | link to Location, computed and stored, editable | Restricted to descendants of the allowed docks. Cleared when the Operation Type changes. Set automatically to the Transfers' common source Location when that Location is an allowed dock. |
| Vehicle Payload Capacity (`vehicle_weight_capacity`) | decimal, related to the category's maximum weight | Read-only. |
| Max Volume (`vehicle_volume_capacity`) | decimal, related to the category's maximum volume | Read-only. |
| Driver (`driver_id`) | link to Contact, computed and stored, editable | Default: the vehicle's driver. |
| Weight percentage (`used_weight_percentage`) | decimal, computed, not stored | See `calculations.md`, section "Batch load". |
| Volume percentage (`used_volume_percentage`) | decimal, computed, not stored | See `calculations.md`, section "Batch load". |
| End Date (`end_date`) | date and time, computed and stored | The scheduled date plus one hour, recomputed whenever the current value is empty or earlier than the scheduled date. |
| Dispatch Management (`has_dispatch_management`) | boolean, related | Read-only. |

On **Vehicle Model Category** (owned by the fleet domain): Max Weight (`weight_capacity`) and Max Volume (`volume_capacity`), plus their unit labels. The displayed name of a category with capacities is the plain name followed by the capacities in parentheses.

Setting a dock rewrites the moves of every Transfer of the batch: for receipt and internal kinds the destination Location becomes the dock; for delivery kinds the source Location becomes the dock. Clearing the dock restores each move's Location to its Transfer's own Location when the move's Location is no longer inside it. Creating a batch, or changing its Transfers, re-sorts the Transfers by the contact's postal code ascending (empty postal codes first) and stamps the resulting index into each Transfer's batch sequence.

---

# 21. Text-message confirmation

On **Company**: Text Message Template (`stock_sms_confirmation_template_id`, link to a text-message template restricted to Transfers, default: the shipped delivery template) and a one-time warning flag (`has_received_warning_stock_sms`).

Transient entity **Text Message Confirmation** (`confirm.stock.sms`): Transfers (`pick_ids`, many-to-many to Transfer). It offers two choices; both mark the company as warned, and the "do not send" choice additionally turns the company's text-message validation setting off. Both then re-run the validation of the Transfers named in the validation context.

---

# 22. Maintenance bridge

On **Equipment** (owned by the repair-and-maintenance domain): Location (`location_id`, link to Location restricted to internal usage) and Serial number match (`match_serial`, boolean, computed, not stored) which is true when at least one Lot exists whose name equals the equipment's serial number; the flag is forced false when the current user may not read Lots or is not in the lot group.

On **Location**: Equipment Count (`equipment_count`, integer, computed, not stored) — the number of equipment records pointing at the Location.

---

# 23. Fields this domain adds to entities of other domains

| Entity | Field (storage name) | Meaning |
|---|---|---|
| Company | Annual Inventory Month (`annual_inventory_month`) | Selection of the twelve months, values `1` "January" through `12` "December"; default `12` "December". The month of the yearly count. Leaving it empty switches the yearly count off. |
| Company | Day of the month (`annual_inventory_day`) | Integer, default 31. Clamped into the valid range of the chosen month. |
| Company | Internal Transit Location (`internal_transit_location_id`) | The transit Location used between the company's warehouses. Created with the company. |
| Company | Email confirmation on delivery (`stock_move_email_validation`) and its template (`stock_mail_confirmation_template_id`) | Whether a delivery confirmation message is posted at validation and with which template. |
| Contact | Customer Location (`property_stock_customer`) and Vendor Location (`property_stock_supplier`) | Per-company Locations overriding the shared customer and vendor Locations for this contact. |
| Contact | Transfer warning (`picking_warn`, `picking_warn_msg`) | The message shown when the contact is used on a Transfer. |
| Product Template | Storable (`is_storable`), Tracking (`tracking`), Routes (`route_ids`), Inventory Loss Location (`property_stock_inventory`), Production Location (`property_stock_production`), Lot numbering sequence (`lot_sequence_id`), Lot property definition (`lot_properties_definition`), Responsible (`responsible_id`) | Consumed by this domain; owned by the products domain. |
| Product Category | Removal Strategy (`removal_strategy_id`), Force Removal Strategy on put-away rules (`filter_for_stock_putaway_rule`), Reserve full packaging (`packaging_reserve_method`) | Consumed by this domain. |
| Unit of Measure | Package Type (`package_type_id`) | Links a packaging unit to a container specification. |
| Barcode Rule | Type values used by this domain: `location`, `package`, `lot` | Consumed by scanning. |

---

# 24. Transient entities

These records exist only for the duration of one user interaction. Each is listed with its fields and the exact effect of its actions.

## 24.1 Backorder Confirmation (`stock.backorder.confirmation`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Transfers (`pick_ids`) | many-to-many to Transfer | The Transfers awaiting a decision. |
| Show transfers (`show_transfers`) | boolean | True when more than one Transfer is being validated, in which case the screen lists them individually. |
| Lines (`backorder_confirmation_line_ids`) | one-to-many of Backorder Confirmation Line | One line per Transfer, each with a "to backorder" switch. |

**Backorder Confirmation Line** (`stock.backorder.confirmation.line`): Transfer (`picking_id`), Confirmation (`backorder_confirmation_id`), To Backorder (`to_backorder`, boolean, default true).

Actions: *Create backorder* re-runs the validation with the backorder decision taken; *No backorder* re-runs it declaring every Transfer as not to be backordered. When the list view is used, only the Transfers whose switch is off are declared as not to be backordered.

## 24.2 Inventory Adjustment Reference (`stock.inventory.adjustment.name`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Quantities (`quant_ids`) | many-to-many to Stock Quantity | The records whose counts will be applied. |
| Inventory Reference (`inventory_adjustment_name`) | text | A free label stamped as the reference of every adjustment move created. |

Action: apply the counts of the listed quantity records, passing the label along.

## 24.3 Inventory Conflict (`stock.inventory.conflict`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Quantities (`quant_ids`) | many-to-many to Stock Quantity | All the records being applied. |
| Conflicting quantities (`quant_to_fix_ids`) | many-to-many to Stock Quantity | The subset whose on-hand quantity moved since the count was entered. |

Actions: *Keep counted quantity* applies the counts as entered; *Discard* clears the counted quantities of the conflicting records.

## 24.4 Inventory Warning (`stock.inventory.warning`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Quantities (`quant_ids`) | many-to-many to Stock Quantity | The records concerned. |

Two variants are shown: one warns that counted quantities are already set before re-setting them, the other warns before clearing them.

## 24.5 Request a Count (`stock.request.count`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Inventory Date (`inventory_date`) | date | Required. The date to schedule. |
| User (`user_id`) | link to User | The person asked to count. |
| Quantities (`quant_ids`) | many-to-many to Stock Quantity | The records to schedule. |
| Set Count (`set_count`) | selection, default `empty` | `empty` "Leave Empty", `set` "Set Current Value". |

Action: write the date and the user on every listed record; when the choice is "Set Current Value" also copy each record's on-hand quantity into its counted quantity; when it is "Leave Empty" clear the counted quantity and its flag.

## 24.6 Quantity Relocation (`stock.quant.relocate`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Quantities (`quant_ids`) | many-to-many to Stock Quantity | The records to move. |
| Destination Location (`dest_location_id`) | link to Location | Where to move them. |
| Destination Package (`dest_package_id`) | link to Package | Which container to put them in. |
| Destination package name (`dest_package_id_domain`) | text, computed | The allowed containers. |
| Message (`message`) | text | The label written as the reference of the generated moves; default "Quantity Relocated". |
| Partial container warning (`partial_package_names`) | text, computed | Names the containers only part of whose content is being moved. |

Action: create and immediately complete one adjustment-style move per quantity record, from its current Location and container to the chosen Location and container.

## 24.7 Quantity History (`stock.quantity.history`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Inventory at Date (`inventory_datetime`) | date and time | Required, default now. |

Action: open the quantity list evaluated as of that instant.

## 24.8 Return Transfer (`stock.return.picking`) and its line (`stock.return.picking.line`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Transfer (`picking_id`) | link to Transfer | The completed Transfer being returned. |
| Lines (`product_return_moves`) | one-to-many of Return Transfer Line | One line per returnable move. |
| Return Location (`location_id`) | link to Location | Where the returned goods go. |
| Operation Type (`picking_type_id`) | link to Operation Type | Which document kind to create. |
| Original Location (`original_location_id`) | link to Location, computed | Used to restrict the choice of return Location. |
| Parent Location (`parent_location_id`) | link to Location, computed | Used to restrict the choice of return Location. |
| Company (`company_id`) | link to Company, related | Read-only. |

**Return Transfer Line**: Product (`product_id`), Quantity (`quantity`), Unit (`uom_id`), Original move (`move_id`), Return (`wizard_id`), and a flag telling whether the line is to be returned.

Actions: *Return* creates the return Transfer and opens it; *Return and Exchange* also re-issues the goods.

## 24.9 Put in Pack (`stock.package.destination` and the put-in-pack action)

**Package Destination** (`stock.package.destination`): Move lines (`move_line_ids`), Destination location (`location_dest_id`), Filtered destination locations (`filtered_location`, computed). Shown when the lines being packed point at more than one destination Location; choosing one rewrites all of them before the container is created.

The put-in-pack wizard proper offers: an existing container, a container type, or a free name. See `workflows.md`, section "Put in pack".

## 24.10 Insufficient Quantity Warning

**Abstract base** (`stock.warn.insufficient.qty`): Product (`product_id`), Location (`location_id`), Quantity (`quantity`), Unit name (`product_uom_name`), Quantity records (`quant_ids`, computed from the product and the Location).

**Scrap variant** (`stock.warn.insufficient.qty.scrap`): adds Scrap (`scrap_id`). Its confirmation action performs the scrap regardless of the shortage.

## 24.11 Label wizards

**Label Type Chooser** (`picking.label.type`): Transfers (`picking_ids`), Print (`label_type`, selection `products` "Products Labels" or `lots` "Lot/SN Labels").

**Lot Label Layout** (`lot.label.layout`): Move lines (`move_line_ids`), Quantity to print (`label_quantity`, selection `lots` "One per lot/SN" or `units` "One per unit"), Format (`print_format`, selection `4x12` or `zpl`).

**Product Label Layout** (extended here): Moves (`move_ids`), Quantity to print (`move_quantity`, selection `move` "Operation Quantities" or `custom` "Custom", default `custom`), added print formats `zpl` "ZPL Labels" and `zplxprice` "ZPL Labels with price", template choice (`zpl_template`, selection `normal` "Normal (2.25\" x 1.25\")", `small` "Small (1.25\" x 1.00\")", `alternative` "Alternative (2.00\" x 1.00\")", `jewelry` "Jewelry (2.20\" x 0.50\")", default `normal`) and a preview image.

## 24.12 Routes Report (`stock.rules.report`)

| Field (storage name) | Type | Meaning |
|---|---|---|
| Product (`product_id`) | link to Product | Required. |
| Product Template (`product_tmpl_id`) | link to Product Template | Required. |
| Warehouses (`warehouse_ids`) | many-to-many to Warehouse | Required. The Warehouses whose rules are drawn. |
| Print Variant Grids (`product_has_variants`) | boolean | |

Action: render the rule diagram for the chosen product across the chosen Warehouses.

## 24.13 Traceability Report (`stock.traceability.report`)

A transient holder with no meaningful stored fields; it renders the upstream and downstream tree of a lot, a product, a container or a move line. See `calculations.md`, section "Traceability tree".

## 24.14 Batch composition wizards

**Add to Batch** (`stock.picking.to.batch`): Batch Transfer (`batch_id`), Add to (`mode`, selection `existing` "an existing batch transfer" or `new` "a new batch transfer", default `existing`), Responsible (`user_id`), Description (`description`).

**Add to Wave** (`stock.add.to.wave`): Wave Transfer (`wave_id`), Add to (`mode`, same two values), Responsible (`user_id`), Description (`description`).

Both create or extend the target grouping document and then open it.
