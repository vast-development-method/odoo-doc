# Configuration

A replenishment installation is configured through company settings, contact settings, product and warehouse settings, stored parameters, and a body of master data that the domain ships or generates: the shipped routes and rules, the routes and rules a warehouse generates, the sequences, the operation types, the scheduled action, the access groups and the record rules. Each item below states its data type, its default value and its effect.

---

## 1. Company settings

All of the following are fields of Company. They are edited on the settings screen but stored per company; a multi-company database holds one value per company.

| Setting | Data type | Required | Default | Effect |
|---|---|---|---|---|
| `replenishment_horizon_days` ("Replenishment Horizon") | decimal | yes | 365 | The number of extra calendar days that a reordering rule looks ahead beyond the pure lead time when it reads the forecast. A value of 0 gives strict just-in-time behavior: a rule orders only what is missing within the lead time. A large value makes rules fire long in advance, which reduces the risk of a stock-out and increases the average stock. The horizon widens the forecast window only; it never moves the date of the documents that are created, because the procurement date subtracts it again (`RP-RULE-229`). |
| `days_to_purchase` ("Days to Purchase") | decimal | no | 0 | The number of calendar days the vendor needs to review and accept a request for quotation. It is added to `total_delay` of every `buy` chain and therefore widens the forecast window of every reordering rule that buys, but it is deliberately not added to `purchase_delay`, so it never moves the order deadline of a purchase order. |
| `sales_security_lead_days` ("Sales Safety Days") | decimal | yes | 0.0 | The number of calendar days by which a procurement created from a sales order line is scheduled earlier than the promised delivery date. The promised date stays the deadline; only the scheduled date moves. |
| `dropship_subcontractor_operation_type` | many_to_one to Operation Type | no | created by the Dropship and Subcontracting Management capability package | The operation type used to ship purchased components straight from a vendor to a subcontractor. |
| `internal_transit_location` (owned by `../inventory-operations/`) | many_to_one to Location | yes | created with the company | The transit location used between two warehouses of the same company. Read by this domain when it builds a resupply route and when it stamps warehouse partners on inter-warehouse moves. |

---

## 2. Settings screen entries

| Entry | Data type | Default | Backed by | Effect |
|---|---|---|---|---|
| Replenishment Horizon | decimal | 365 | `company.replenishment_horizon_days` | see section 1 |
| Days to Purchase | decimal | 0 | `company.days_to_purchase` | see section 1 |
| Security Lead Time for Sales | boolean | false | the stored parameter `sales.use_security_lead_time` | Turns the sales safety days on. Turning it off resets `sales_security_lead_days` to 0.0. Visible only when the Sales capability package is installed. |
| Sales Safety Days | decimal | 0.0 | `company.sales_security_lead_days` | see section 1. Editable only while the switch above is on. |
| Multi-Step Routes | boolean | false | grants the access group "Multi-Step Routes" | Turns on multi-step routes: it reveals the routes and rules screens, the route selection on products, categories and sales order lines, and the reception and delivery step selectors on the warehouse. Turning it on also turns on Storage Locations, because a multi-step route needs intermediate locations. |
| Replenish on Order (Make To Order) | boolean | false | the `active` flag of the global "Replenish on Order" route | Activates or archives the global make-to-order route. |
| Dropshipping | boolean | false | installs the Drop Shipping capability package | Adds the `dropship` operation type code, the global Dropship route, the per-company dropship sequence, operation type and `buy` rule. |
| Storage Locations (owned by `../inventory-operations/`) | boolean | false | grants the access group "Manage Multiple Stock Locations" | Prerequisite of multi-step routes. |

---

## 3. Contact settings (vendors)

| Setting | Data type | Required | Default | Effect |
|---|---|---|---|---|
| `group_request_for_quotation` ("Group Request for Quotation") | selection: `default` = "On Order", `day` = "Daily", `week` = "Weekly", `all` = "Always" | yes | `default` | How replenishment needs for this vendor are grouped into requests for quotation. `default`: needs are grouped by originating reference, so needs that come from different documents never merge. `day`: needs whose expected arrival falls on the same calendar day are grouped. `week`: needs whose expected arrival falls in the same week, or on the same target weekday, are grouped. `all`: every need for this vendor is grouped. Drop shipping operations always keep the reference component whatever the mode. |
| `grouping_weekday` ("Week Day") | selection: `default` = "Expected Date", `1` = Monday, `2` = Tuesday, `3` = Wednesday, `4` = Thursday, `5` = Friday, `6` = Saturday, `7` = Sunday | yes | `default` | Only used when `group_request_for_quotation` is `week`. `default` groups by calendar week; a named weekday moves every line's expected arrival forward to the next occurrence of that weekday and groups on it. |
| `suggest_based_on` | text | no | `30_days` | The historic window used by the purchase suggestion feature. Accepted values: `30_days`, `actual_demand`, `one_week`, `three_months`, `one_year`, `last_year`, `last_year_next_month`, `last_year_month_after_next`, `last_year_quarter`. |
| `suggest_days` | integer | no | 7 | The number of days of demand the purchase suggestion covers. |
| `suggest_percent` | integer | no | 100 | A percentage applied to the suggested quantity. |
| `buyer` (owned by `../purchasing/`) | many_to_one to User | no | empty | The buyer written on the purchase orders created for this vendor, and a component of the purchase order grouping key. |
| `purchase_currency` (owned by `../purchasing/`) | many_to_one to Currency, per company | no | empty | The fallback currency of a purchase order when the chosen Vendor Price has none. |
| `vendor_payment_terms` (owned by `../accounts-payable/`) | many_to_one to Payment Term, per company | no | empty | Written on the purchase orders created for this vendor. |
| `vendor_location` and `customer_location` (owned by `../inventory-operations/`) | many_to_one to Location, per company | yes | the shared vendor location, the shared customer location | The vendor location a receipt takes from and the customer location a delivery sends to. For a warehouse partner they are both set to the company's internal transit location. |
| `subcontractor_location` (owned by `../manufacturing/`) | many_to_one to Location, per company | no | empty | The subcontracting location of a subcontractor; read by the drop-ship-to-subcontractor recognition rule. |

---

## 4. Product and product category settings

| Setting | Entity | Data type | Default | Effect |
|---|---|---|---|---|
| `routes` | Product Template | many_to_many to Route | empty | The routes offered first to rule selection after the routes carried by the request. Restricted to routes whose `product_selectable` is true. |
| `routes` | Product Category | many_to_many to Route | empty | Restricted to routes whose `product_category_selectable` is true. |
| `total_routes` | Product Category | derived many_to_many to Route | derived | The category's own routes plus the routes of every ancestor category. This is the set that rule selection uses. |
| `vendor_prices` (owned by `../pricing-and-pricelists/`) | Product Template | one_to_many to Vendor Price | empty | The vendor prices among which vendor selection chooses. Each carries a contact, a minimum quantity, a unit, a price, a currency, validity dates and a lead time in days. |
| `responsible_user` | Product Template | many_to_one to User | the creating user | The person who receives the warning activity when a reordering rule of this product fails. |
| `is_storable` | Product Template | boolean | false | A reordering rule may only watch a storable goods product. |
| `can_be_purchased` (owned by `../purchasing/`) | Product Template | boolean | true | Selecting a Buy route on a product template that cannot be purchased raises the warning `This product has the "Buy" route checked but is not purchasable.` |

---

## 5. Location and warehouse settings

| Setting | Entity | Data type | Default | Effect |
|---|---|---|---|---|
| `replenish_location` | Location | boolean | true for the stock location of each warehouse, false elsewhere; forced to false when the usage is not `internal` | The replenishment report watches this location and creates temporary reordering rules for the negative forecasts found in it. A location may not have this flag while an ancestor or a descendant also has it. |
| `reception_steps` | Warehouse | selection: `one_step` = "Receive and Store (1 step)", `two_steps` = "Receive then Store (2 steps)", `three_steps` = "Receive, Quality Control, then Store (3 steps)" | `one_step` | Determines the rules of the warehouse's reception route (section 8) and whether the Input and Quality Control locations are active. |
| `delivery_steps` | Warehouse | selection: `ship_only` = "Deliver (1 step)", `pick_ship` = "Pick then Deliver (2 steps)", `pick_pack_ship` = "Pick, Pack, then Deliver (3 steps)" | `ship_only` | Determines the rules of the warehouse's delivery route and whether the Packing Zone and Output locations are active. |
| `routes` | Warehouse | many_to_many to Route | the warehouse's own reception and delivery routes | The routes offered last to rule selection. Restricted to routes whose `warehouse_selectable` is true and that belong to the warehouse company or to no company. |
| `resupply_warehouses` | Warehouse | many_to_many to Warehouse | empty | The warehouses that may resupply this one. Writing the field creates or archives inter-warehouse resupply routes (section 9). |
| `buy_to_resupply` | Warehouse | boolean, derived from the global Buy route's warehouse list, writable | true | When true the warehouse may be supplied by purchasing; the warehouse is listed on the global Buy route and its `buy` rule is active. |
| `manufacture_to_resupply` (owned by `../manufacturing/`) | Warehouse | boolean | true | When true the warehouse may be supplied by manufacturing. |
| `subcontracting_to_resupply` (owned by `../manufacturing/`) | Warehouse | boolean | true | When true the warehouse holds the rules that send components to subcontractors, and, when the Dropship and Subcontracting Management capability package is installed, the rule that moves components from the subcontracting location to the production location. |
| `partner` | Warehouse | many_to_one to Contact | the company's partner | Stamped on inter-warehouse moves so that both transfers show the two warehouses as counterparties. Setting it also rewrites that contact's vendor and customer locations to the company's internal transit location. |

---

## 6. Stored parameters

| Parameter | Data type | Default when absent | Effect |
|---|---|---|---|
| `inventory.disable_automatic_scheduler` | text, treated as a switch | absent | While present, the event-driven trigger that runs automatic reordering rules when moves are confirmed is disabled entirely. The daily scheduled action still runs. |
| `inventory.cancel_originating_moves` | text, treated as a switch | absent | While present, cancelling a move whose `propagate_cancel` is true also cancels its not-yet-completed origin moves. |
| `purchasing.on_time_delivery_days` | integer as text | 365 | The length in days of the window over which a vendor's on-time delivery rate is measured. |
| `sales.use_security_lead_time` | boolean as text | false | Backs the "Security Lead Time for Sales" switch. |

The four keys above are full-word keys chosen for this specification. A replacement may use any stable key, provided the four switches exist with the stated defaults and the stated effects.

---

## 7. Shipped master data

### 7.1 Locations (owned by `../inventory-operations/`, listed because every rule refers to them)

| Record | Usage | Company | Active by default |
|---|---|---|---|
| Vendors | `supplier` | none | yes |
| Customers | `customer` | none | yes |
| Inter-company transit | `transit` | none | no; activated the first time an inter-company resupply route is created |
| Internal transit location | `transit` | one per company | yes |

### 7.2 Routes

| Route | Sequence | Company | Active by default | Selectable on | Created by |
|---|---|---|---|---|---|
| Replenish on Order | 5 | none | no | product categories | the Inventory capability package. Deliberately given a lower sequence than the resupply routes, so that a make-to-order rule wins over a resupply rule when both apply. |
| Buy | 10 | one per company (resolved or duplicated per company) | yes | warehouses | the Purchase Inventory capability package; the first warehouse is listed on it. |
| Dropship | 20 | none | yes | sales order lines, products, product categories | the Drop Shipping capability package. |
| `<warehouse name>: <reception step label>` | 9 when the Purchase Inventory capability package is installed, 50 otherwise | the warehouse company | follows the warehouse | product categories, warehouses | generated per warehouse (section 8). |
| `<warehouse name>: <delivery step label>` | 60 | the warehouse company | follows the warehouse | product categories, warehouses | generated per warehouse (section 8). |
| `<supplied warehouse name>: Supply Product from <supplying warehouse name>` | 0 | the intersection of the two warehouses' companies | yes | warehouses, products, product categories | generated per resupply pair (section 9). |

The reception and delivery step labels are: `Receive in 1 step (stock)`, `Receive in 2 steps (input + stock)`, `Receive in 3 steps (input + quality + stock)`, `Deliver in 1 step (ship)`, `Deliver in 2 steps (pick + ship)`, `Deliver in 3 steps (pick + pack + ship)`.

**Resolving a global route.** When a warehouse needs one of the global routes (Replenish on Order, Buy, Dropship, and the manufacturing and subcontracting routes owned by `../manufacturing/`), the route is resolved as follows:

1. Take the route registered under the well-known marker of that route.
2. When it does not exist, or when it has a company that is not the warehouse's company, search for an active or archived route whose name is like the given label and whose company is the warehouse's company or empty, ordered by company, and take the first.
3. When nothing is found and the marker route exists, duplicate it with the given name, the warehouse's company and no rules.
4. When nothing is found and the marker route does not exist either, the slot is skipped entirely: no rule is created for it and no error is raised. A user who deleted a global route therefore simply loses that capability.

### 7.3 Rules shipped per company

| Rule | Route | Action | Source | Destination | Supply method | Operation type |
|---|---|---|---|---|---|---|
| `<vendor location name> → <customer location name>` | Dropship | `buy` | the shared vendor location | the shared customer location | `make_to_stock` | the company's Dropship operation type |
| `<vendor location name> → <subcontracting location name>` | Dropship | `buy` | the shared vendor location | the company's subcontracting location | `make_to_stock` | the company's Dropship Subcontractor operation type |

Both are created when the company is created and are repaired for existing companies when the capability package is installed.

### 7.4 Sequences

| Sequence | Code | Prefix | Padding | First value | Increment | Company |
|---|---|---|---|---|---|---|
| Reordering Rule | `reordering_rule` | `OP/` | 5 | 1 | 1 | none (shared) |
| Dropship (`<company name>`) | `dropship_transfer` | `DS/` | 5 | 1 | 1 | one per company |
| Dropship Subcontractor (`<company name>`) | `dropship_subcontractor_transfer` | `DSC/` | 5 | 1 | 1 | one per company |

The three codes above are the stable keys under which the sequences are looked up. They are full-word keys chosen for this specification; a replacement may use any stable key, provided the same three sequences exist and are found by the reordering rule, the drop shipping operation type and the drop-ship-to-subcontractor operation type respectively.

### 7.5 Operation types created by this domain

| Operation type | Code | Warehouse | Sequence prefix | Default source | Default destination | Use existing lots |
|---|---|---|---|---|---|---|
| Dropship | `dropship` | none | `DS` | the shared vendor location | the shared customer location | off |
| Dropship Subcontractor | `dropship` | none | `DSC` | the shared vendor location | the company's subcontracting location | off |

An operation type whose code is `dropship` always has those default locations, never has a warehouse, and is always shown in the operations overview. Removing the Drop Shipping capability package rewrites such operation types to code `outgoing` and archives them.

### 7.6 Screens and menus shipped by this domain

| Item | Purpose |
|---|---|
| Menu "Dropships" under Operations, sequence 30 | Opens the list of transfers whose operation type code is `dropship`, with the "Dropships" filter applied. Visible to the inventory user group and the inventory administrator group. |
| Action "On-time Delivery" | Opens the Vendor Delay Report as a graph, with the filter "later than a year ago" applied. Reachable from a vendor's form and from a purchase order. |

---

## 8. Routes and rules generated by a warehouse

When a warehouse is created, and whenever one of the declared dependency fields is written, the warehouse rebuilds its routes and its global rules.

### 8.1 The two step routes

For each of the two route slots (`reception_route` and `delivery_route`):

1. When the slot already holds a route, the route is updated and **all** of its rules are archived; otherwise the route is created.
2. The routing list for the current step value is looked up and one rule per entry is created or unarchived.
3. When the route is selectable on warehouses, it is added to the warehouse's `routes`.

**Route values.**

| Field | Reception route | Delivery route |
|---|---|---|
| `name` | `<warehouse name>: <reception step label>` | `<warehouse name>: <delivery step label>` |
| `active` | follows the warehouse | follows the warehouse |
| `product_category_selectable` | true | true |
| `warehouse_selectable` | true | true |
| `product_selectable` | false | false |
| `company` | the warehouse company | the warehouse company |
| `sequence` | 50, or 9 when the Purchase Inventory capability package is installed | 60 |

**Routing lists.**

| Step value | Routings, in order: source → destination, operation type, action |
|---|---|
| `one_step` | shared vendor location → warehouse stock, Receipt, pull |
| `two_steps` | shared vendor location → warehouse stock, Receipt, pull; Input → warehouse stock, Storage, push |
| `three_steps` | shared vendor location → warehouse stock, Receipt, pull; Input → Quality Control, Quality Control, push; Quality Control → warehouse stock, Storage, push |
| `ship_only` | warehouse stock → shared customer location, Delivery, pull |
| `pick_ship` | warehouse stock → shared customer location, Pick, pull; Output → shared customer location, Delivery, push |
| `pick_pack_ship` | warehouse stock → shared customer location, Pick, pull; Packing Zone → Output, Pack, push; Output → shared customer location, Delivery, push |

When the Purchase Inventory capability package is installed, the reception routing list loses its first entry, because the `buy` rule now starts the chain, and every remaining rule is created with supply method `make_to_order`. The lists then become: `one_step` → no rule at all; `two_steps` → Input → warehouse stock, Storage, push; `three_steps` → Input → Quality Control, Quality Control, push and Quality Control → warehouse stock, Storage, push.

**Rule values.**

| Field | Value |
|---|---|
| `name` | `<warehouse code>: <source location name> → <destination location name>`, plus ` (<suffix>)` when a suffix is given |
| `location_source`, `destination_location`, `action`, `operation_type` | from the routing |
| `auto` | `manual` |
| `procure_method` | `make_to_stock` for the first routing of the list, `make_to_order` for every later one; overridden to `make_to_order` for every reception rule when the Purchase Inventory capability package is installed |
| `warehouse` | the warehouse |
| `company` | the warehouse company |
| `active` | true |
| `propagate_cancel` | true for the reception route's rules, then forced to false on the **last** rule of the list |
| `propagate_carrier` | true for the delivery route's rules |

**Rule reuse.** Before a rule is created, an archived rule with the same `operation_type`, `location_source`, `destination_location`, `route` and `action` is looked for; when one exists it is unarchived instead of a duplicate being created.

### 8.2 The global rules

For each global rule slot, when the slot is empty a rule is created from the create values plus the update values; otherwise the existing rule is written with the update values only.

| Slot | Route | Dependencies | Create values | Update values |
|---|---|---|---|---|
| `make_to_order_pull` | Replenish on Order | `delivery_steps` | active; `procure_method` `make_to_order`; the warehouse company; `action` `pull`; `auto` `manual`; `propagate_carrier` true | `name` = `<warehouse code>: <stock location name> → <destination name> (Make To Order)`; `destination_location` and `operation_type` from the first delivery routing that starts at the warehouse stock location; `location_source` = the warehouse stock location |
| `buy_pull` | Buy | `reception_steps`, `buy_to_resupply` | `action` `buy`; the warehouse's incoming operation type; the warehouse company; `propagate_cancel` = (`reception_steps` is not `one_step`) | `active` = `buy_to_resupply`; `name` = `<warehouse code>: <stock location name> (Buy)`; `destination_location` = the warehouse stock location; `propagate_cancel` = (`reception_steps` is not `one_step`) |
| `subcontracting_dropshipping_pull` | Dropship | `subcontracting_to_resupply`, `active` | `procure_method` `make_to_order`; the warehouse company; `action` `pull`; `auto` `manual`; `name` from the subcontracting and production locations; `destination_location` = the production location; `location_source` = the company's subcontracting location; the warehouse's subcontracting operation type | `active` = `subcontracting_to_resupply` |
| `manufacture_pull`, `manufacture_make_to_order_pull`, `pick_before_manufacturing_make_to_order_pull`, `store_after_manufacturing_rule`, `subcontracting_pull`, `subcontracting_make_to_order_pull` | the manufacturing and subcontracting global routes | owned by `../manufacturing/` | owned by `../manufacturing/` | owned by `../manufacturing/` |

A slot whose route cannot be resolved is skipped (see section 7.2).

### 8.3 Reconfiguration

| Written field | Consequence |
|---|---|
| `reception_steps` | The Input location is active when the value is not `one_step`; the Quality Control location is active when the value is `three_steps`. The reception route is rebuilt. The inter-warehouse resupply check runs. The `buy_pull` rule's `propagate_cancel` is updated. |
| `delivery_steps` | The Output location is active when the value is not `ship_only`; the Packing Zone is active when the value is `pick_pack_ship`. The delivery route is rebuilt. The `make_to_order_pull` rule is updated. The inter-warehouse resupply check runs (section 9.3). |
| `buy_to_resupply` | True adds the warehouse to the Buy route's warehouse list; false removes it. The `buy_pull` rule's `active` follows. |
| `resupply_warehouses` | Removed warehouses have their resupply routes archived; added warehouses have their archived routes unarchived when they exist, and new routes created otherwise. |
| `name` | In every route name of the warehouse, in every rule name of those routes, in the make-to-order rule name and in the `buy` rule name, the first occurrence of the old warehouse name is replaced by the new one. |
| `code` | The warehouse's view location is renamed and the sequences of its operation types are renamed. Rule names are not rewritten. |
| `active` | Archiving a warehouse is refused while any of its operation types has a move that is neither completed nor cancelled, with the message `You still have ongoing operations for operation types <operation type names> in warehouse <warehouse name>`; and while another operation type outside the warehouse uses a location of the warehouse as default source or destination, with the message `<operation type names> have default source or destination locations within warehouse <warehouse name>, therefore you cannot archive it.` Otherwise every operation type, the view location, every rule of the warehouse and every route that applies only to this warehouse follow the flag. Unarchiving rewrites every dependency field onto itself, which rebuilds the routes and rules. |
| `company` | Refused: `Changing the company of this record is forbidden at this point, you should rather archive it and create a new one.` |

---

## 9. Inter-warehouse resupply routes

### 9.1 What is created

For each pair (supplied warehouse, supplying warehouse):

1. The transit location is the company's internal transit location when both warehouses share a company, and the shared inter-company transit location otherwise. When neither exists the pair is skipped. The chosen transit location is activated.
2. The supplying warehouse's output location is its stock location when it delivers in one step, and its Output location otherwise.
3. When the supplying warehouse delivers in one step, an extra rule is added to the Replenish on Order route: from the output location to the transit location, with the supplying warehouse's outgoing operation type, the create values of that warehouse's make-to-order rule, and the name suffix "Make To Order".
4. The resupply route is created with: `name` = `<supplied warehouse name>: Supply Product from <supplying warehouse name>`, `warehouse_selectable` true, `product_selectable` true, `product_category_selectable` true, `supplied_warehouse` = the supplied warehouse, `supplier_warehouse` = the supplying warehouse, `company` = the intersection of the two warehouses' companies.
5. The route's rules are created active, in this order:

| Order | Source | Destination | Operation type | Supply method | Other |
|---|---|---|---|---|---|
| 1 | the supplying warehouse's output location | the transit location | the supplying warehouse's outgoing operation type | `make_to_stock` when the source is the supplying warehouse's stock location, `make_to_order` otherwise | `location_destination_from_rule` true |
| 2, only when the supplying warehouse does not deliver in one step | the supplying warehouse's stock location | its output location | its picking operation type | `make_to_stock` | |
| 3 | the transit location | the supplied warehouse's stock location | the supplied warehouse's incoming operation type | `make_to_order` | |

### 9.2 Partner stamping

When the pull action creates the move into the transit location and the request location is the company's internal transit location, the new move receives the partner of the destination warehouse (when the downstream moves resolve to exactly one warehouse partner) and the downstream moves receive the partner of the warehouse of the rule's source location, or, when that warehouse has none, the partner of the rule's company.

### 9.3 Changing the delivery steps of a supplying warehouse

The check runs only when the change crosses the boundary between one step and several steps.

1. Every rule of every route supplied by this warehouse, that is not a push rule and whose destination is a transit location, has its `location_source` rewritten to the new output location and its `procure_method` set to `make_to_order` when moving to several steps or `make_to_stock` when moving back to one step.
2. **Moving back to one step:** the extra "stock to Output" rules of those routes, identified by destination equal to the Output location and the picking operation type, are archived; and one new rule per transit destination is created from the stock location with the outgoing operation type and the create values of the make-to-order rule, with the name suffix "Make To Order".
3. **Moving to several steps:** those extra rules are unarchived; for every supplied route that had none, a rule "stock location → new output location" with the picking operation type is created; and every rule of the Replenish on Order route whose destination is a transit location and whose source is this warehouse's stock location is archived, so that it can no longer be selected.

---

## 10. The scheduled action

| Property | Value |
|---|---|
| Name | `Procurement: run scheduler` |
| Runs as | the superuser account |
| Interval | every 1 day |
| Active by default | yes |
| Effect | Runs the three scheduler tasks of `workflows.md`, section 12, in batch mode: recompute and run every automatic reordering rule, reserve every waiting move whose reservation date has been reached, merge duplicated stock quantity records. |
| Progress reporting | Three tasks are announced; each commits its progress when it finishes. |
| Failure | Any exception is logged with its stack trace and re-raised, which aborts that run. |

---

## 11. Access groups and what each may do

| Group | Reads | Writes |
|---|---|---|
| Any internal user | Routes and Stock Rules (needed because rule selection runs on behalf of users with no inventory rights) | nothing in this domain |
| Inventory / User | Routes, Stock Rules, Reordering Rules, Purchase Orders, Purchase Order Lines | creates, reads and changes the Stock Rules Report wizard, the Product Replenish wizard and the Replenishment Option wizard; creates, reads, changes and deletes the Reordering Rule Snooze wizard |
| Inventory / Administrator | everything of the Inventory / User group, plus the Replenishment Information wizard | creates, changes and deletes Routes, Stock Rules and Reordering Rules; creates, reads and changes the Replenishment Information wizard |
| Purchase / User | Locations, Warehouses, Reordering Rules, the Vendor Delay Report | creates, reads, changes and deletes Transfers; creates, reads and changes Stock Moves |
| Purchase / Administrator | the same as Purchase / User | the same as Purchase / User, plus deleting Stock Moves |
| Manage Multiple Stock Locations (hidden group) | none | Reveals the location fields on reordering rules and the multi-location screens. |
| Manage Multiple Warehouses (hidden group) | none | Reveals the warehouse fields. Granted automatically as soon as a second active warehouse exists, and revoked automatically when only one remains. |
| Multi-Step Routes (hidden group) | none | Reveals the routes and rules screens, the route selection on products, categories and sales order lines, and the reception and delivery step selectors. |

---

## 12. Record rules (row-level visibility)

| Entity | Condition |
|---|---|
| Reordering Rule | `company IN <the user's active companies>` |
| Stock Rule | `company IN <the user's active companies> OR company IS EMPTY` |
| Route | `company IN <the user's active companies> OR company IS EMPTY` |

A Route or Stock Rule with no company is visible to every user, which is what lets the global Replenish on Order and Dropship routes work across companies.

---

## 13. Master-data prerequisites

Before this domain can function, the following must exist. Each row states what fails when the prerequisite is missing.

| Prerequisite | Owner | Failure when missing |
|---|---|---|
| At least one warehouse per company, with a stock location | `../inventory-operations/` | Creating a reordering rule raises the redirect warning `Please create a warehouse for company <company name>.`; opening the Stock Rules Report wizard raises the same. |
| The shared vendor location and the shared customer location | `../inventory-operations/` | Generating a warehouse's routes fails with `Can't find any customer or supplier location.` |
| An internal transit location per company | `../inventory-operations/` | Inter-warehouse resupply routes for warehouses of the same company cannot be created; the pair is skipped silently. |
| The shared inter-company transit location | `../inventory-operations/` | Inter-company resupply routes cannot be created; the pair is skipped silently. |
| The sequence with code `reordering_rule` | this domain | A reordering rule cannot be created, because `name` is mandatory. |
| The Replenish on Order route | this domain | Warehouses stop maintaining their make-to-order rule; make-to-order flows must then be configured by hand. |
| The Buy route | the Purchase Inventory capability package | Warehouses stop maintaining their `buy` rule; no reordering rule can buy. |
| At least one Vendor Price per product that must be bought | `../pricing-and-pricelists/` | A reordering rule fails with the "no matching vendor price" message; a lead-time computation adds a penalty of 365 days and shows `No Vendor Found`. |
| At least one bill of materials of type `normal` per product that must be manufactured | `../manufacturing/` | A lead-time computation adds a penalty of 365 days and shows `No Bill of Materials Found`. |
| The decimal precision setting named "Product Unit" | `../platform-foundation/` | Quantity rounding falls back to the platform default and the quantities to order become inconsistent. |
