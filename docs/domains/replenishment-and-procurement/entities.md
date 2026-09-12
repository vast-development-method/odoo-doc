# Entities

The Replenishment and Procurement domain owns three persistent entities (Route, Stock Rule, Reordering Rule), six transient wizards and reports, and one in-memory structure, the procurement request, which is never persisted but whose exact shape a replacement must reproduce. It also adds fields to seventeen entities owned by other domains. Every one of those entities is specified below, field by field.

## 0. Conventions that apply to every entity in this file

### 0.1 Shared persistent fields

Every persistent entity carries the following fields, which are not repeated in the per-entity tables:

| Field | Type | Meaning |
|---|---|---|
| `identifier` | integer | Surrogate primary key, assigned by the system, never reused. |
| `created_on` | datetime | Moment the record was inserted. |
| `created_by_user` | many_to_one to User | The user who inserted the record. On records created by the scheduler this is the superuser account. |
| `last_updated_on` | datetime | Moment of the most recent modification. |
| `last_updated_by_user` | many_to_one to User | The user who made the most recent modification. |

Entities that support archiving additionally carry `active` (boolean, default true). Archived records are excluded from every default query; a query may opt in to archived records explicitly.

### 0.2 Quantity precision

Every quantity field in this domain is a `decimal` stored with the precision of the shared decimal precision setting named "Product Unit" (default two decimal places). Comparisons between quantities are never exact equality comparisons: they use the rounding of the unit of measure in question, as specified in `../units-of-measure-and-packaging/calculations.md`. Throughout this file, "compare *a* with *b* in unit *u*" means: round both values to the rounding step of *u* and compare the rounded values.

### 0.3 Company scoping

Route, Stock Rule and Reordering Rule are company-scoped. A Route may have an empty company, which means it is shared by every company. A Stock Rule always belongs to the company of its Route when the Route has one. Records of this domain are visible only to users whose active company set includes the record's company, or whose company is empty.

### 0.4 Notation for relation deletion behavior

- `restrict`: deleting the referenced record is refused while a referring record exists.
- `cascade`: deleting the referenced record deletes the referring record.
- `set null`: deleting the referenced record empties the reference.

### 0.5 Change tracking in the discussion thread

None of the three persistent entities this domain owns (Route, Stock Rule, Reordering Rule) carries a discussion thread, so no field of them is tracked: changing a field writes no message and raises no notification. The per-entity field tables state this explicitly in their `Tracked` column so that a replacement does not have to infer it. The transient entities of sections 4 to 9 are wizards; they live for the duration of one dialog, are never followed and are never tracked. For the fields this domain adds to entities owned by other domains (sections 11 to 27), the tracking behavior is the one the owning domain defines for that entity; this domain adds no tracked field to any of them. In particular the `destination_address` it adds to Purchase Order is not tracked, even though Purchase Order carries a discussion thread and tracks its vendor.

### 0.6 Field-level access

Unless a per-entity "Field visibility and editing rights" subsection says otherwise, every field of an entity is readable by every access group listed as a reader of that entity in `configuration.md`, section 11, and writable by every access group listed there as a writer. Three hidden access groups narrow that default per field, and each per-entity subsection names the fields they hide:

| Hidden access group | Effect |
|---|---|
| Manage Multiple Stock Locations | Without it, the location fields are hidden from the form and the list, and the value the system computed is used. |
| Manage Multiple Warehouses | Without it, the warehouse fields are hidden and the single warehouse is used. Granted automatically as soon as a second active warehouse exists. |
| Multi-Step Routes | Without it, the Routes and Stock Rules screens are hidden altogether, and the route selection fields are hidden on products, product categories and sales order lines. |

---

## 1. Route

A Route is an ordered, named collection of Stock Rules. Routes are the unit of selection: a user attaches a Route to a product, a product category, a warehouse, a package type, a sales order line or a shipping method, and the rules inside it then become candidates for rule selection.

Identifier: `route`. Kind: persistent, archivable, company-scoped (company may be empty).

### 1.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text, translatable | yes | none | stored | yes, with the suffix " (copy)" appended | no | The route name shown everywhere. |
| `active` | boolean | no | true | stored | yes | no | When false the route is hidden. Writing `active` also archives or unarchives every rule of the route whose destination location is itself active (see rule RP-RULE-011). |
| `sequence` | integer | no | 0 | stored | yes | no | Ordering key. Lower values are evaluated first during rule selection. Also the default ordering of route lists. |
| `rules` | one_to_many to Stock Rule (inverse `route`) | no | empty | stored | yes (rules are duplicated with the route) | no | The rules that make up the route. |
| `product_selectable` | boolean | no | true | stored | yes | no | When true the route may be chosen on a product. |
| `product_category_selectable` | boolean | no | false | stored | yes | no | When true the route may be chosen on a product category. |
| `warehouse_selectable` | boolean | no | false | stored | yes | no | When true the route may be attached to warehouses, and it then acts as a default route for goods passing through those warehouses. |
| `package_type_selectable` | boolean | no | false | stored | yes | no | When true the route may be chosen on a package type. |
| `sale_selectable` | boolean | no | false | stored | yes | no | When true the route may be chosen on a sales order line. Added by the Sales Inventory capability package. |
| `shipping_selectable` | boolean | no | false | stored | yes | no | When true the route may be chosen on a shipping method. Added by the Delivery Methods capability package. |
| `supplied_warehouse` | many_to_one to Warehouse | no | empty | stored, indexed (index skips empty values) | no | no | For an automatically generated inter-warehouse resupply route: the warehouse that receives the goods. |
| `supplier_warehouse` | many_to_one to Warehouse | no | empty | stored | no | no | For an automatically generated inter-warehouse resupply route: the warehouse that ships the goods. |
| `company` | many_to_one to Company | no | the active company | stored, indexed | yes | no | Empty means the route is shared between all companies. |
| `products` | many_to_many to Product Template | no | empty | stored | no | no | The products on which this route is selected. Constrained to products of the route company. |
| `product_categories` | many_to_many to Product Category | no | empty | stored | no | no | The product categories on which this route is selected. (Naming choice: the mechanically derived plural of "category" is ungrammatical, so the name `product_categories` is used instead, spelled in full and consistently throughout this folder.) |
| `allowed_warehouses` | one_to_many to Warehouse | no | computed | derived, not stored | no | no | The list of warehouses that may be selected in `warehouses`: every warehouse of `company`, or every warehouse when `company` is empty. Recomputed when `company` changes. |
| `warehouses` | many_to_many to Warehouse | no | empty | stored | no | no | The warehouses for which this route is a default route. Restricted to `allowed_warehouses`. |

### 1.2 Identity, ordering and display

- There is no uniqueness constraint on the route name. Two routes may share a name; they are distinguished by identifier.
- Default ordering: by `sequence` ascending. Records with an equal sequence are returned in insertion order.
- Display name: `name`.
- No parent/child hierarchy.

### 1.3 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Company consistency | For every rule of the route, if the route has a company then the rule company must equal the route company. Checked when `company` changes. | `Rule <rule name> belongs to <rule company> while the route belongs to <route company>.` |

### 1.4 On-change behavior in a form

| Field edited | Effect |
|---|---|
| `company` | `warehouses` is filtered down to the warehouses that belong to the newly chosen company; warehouses of other companies are removed from the selection. |
| `warehouse_selectable` | When it becomes false, `warehouses` is emptied. |

### 1.5 Duplication

Duplicating a route copies `rules` (each rule is duplicated with the copy suffix on its name), sets the new `name` to the original name followed by " (copy)", and clears `products`, `product_categories`, `warehouses`, `supplied_warehouse` and `supplier_warehouse`.

### 1.6 Lifecycle

1. Created manually by an inventory administrator, or generated automatically by warehouse configuration (see `configuration.md`, section "Routes and rules generated by a warehouse").
2. Used by rule selection while `active` is true.
3. Archived: `active` set to false, which archives its rules.
4. Deleted: deleting a route cascades to its rules (the rule's `route` relation has deletion behavior `cascade`).

### 1.7 Resupply eligibility test

Every Route answers the question "is this route a valid resupply route for this product?". The answer is computed as follows and is used by the Product Replenish Wizard and by warehouse route filtering:

1. If any rule of the route has action `buy`: the answer is true when the product has at least one Vendor Price, false otherwise.
2. Otherwise, if any rule of the route has action `manufacture`: the answer is true when the product has at least one bill of materials of type `normal`, false otherwise.
3. Otherwise: the answer is false.

### 1.8 Field visibility and editing rights

| Fields | Who may read | Who may edit |
|---|---|---|
| `name`, `active`, `sequence`, `rules`, `product_selectable`, `product_category_selectable`, `warehouse_selectable`, `package_type_selectable`, `sale_selectable`, `shipping_selectable`, `supplied_warehouse`, `supplier_warehouse`, `company`, `products`, `product_categories`, `allowed_warehouses`, `warehouses` | every internal user, because rule selection runs on behalf of users who have no inventory rights | Inventory / Administrator only |

The Routes screen itself is hidden from users who do not hold the hidden access group Multi-Step Routes, so in practice only a holder of both Multi-Step Routes and Inventory / Administrator edits a route through the screen. `supplied_warehouse` and `supplier_warehouse` are written only by the resupply-route generator and are shown read-only. `warehouses` is additionally hidden from users without the hidden access group Manage Multiple Warehouses.

---

## 2. Stock Rule

A Stock Rule is one step of a route. It states what to do when a need for a product appears at a location (pull, buy, manufacture) or what to do when goods arrive at a location (push).

Identifier: `stock_rule`. Kind: persistent, archivable, company-scoped.

### 2.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text, translatable | yes | none | stored | yes, with " (copy)" appended | no | Fills the origin of the documents the rule creates and the name of its moves. |
| `active` | boolean | no | true | stored | yes | no | When false the rule is never selected. |
| `action` | selection | yes | `pull` | stored, indexed | yes | no | See the closed list below. |
| `sequence` | integer | no | 20 | stored | yes | no | Ordering key inside a route; lower is evaluated first. |
| `company` | many_to_one to Company | no | the active company | stored, indexed | yes | no | Restricted to the route company when the route has one. |
| `route` | many_to_one to Route | yes | none | stored, indexed, deletion behavior `cascade` | yes | no | The route this rule belongs to. |
| `route_company` | many_to_one to Company | no | derived from `route.company` | derived, not stored | not applicable | no | Used only to restrict the `company` selection. |
| `route_sequence` | integer | no | derived from `route.sequence` | derived and stored (kept in step with the route sequence) | not applicable | no | Denormalized copy of the route sequence, used to sort candidate rules without joining. |
| `destination_location` | many_to_one to Location | yes | none | stored, indexed | yes | no | The location where the need appears (pull, buy, manufacture) or the location goods are sent to (push). Must belong to the rule company. |
| `location_source` | many_to_one to Location | no | empty | stored, indexed | yes | no | The location the goods are taken from (pull) or the location where goods arrive (push). Must belong to the rule company. Not used by `buy`. |
| `location_destination_from_rule` | boolean | no | false | stored | yes | no | When true, the destination location written on the created stock move is the rule's `destination_location`. When false, the destination location is taken from the operation type's default destination location, and the rule's `destination_location` is written on the move as its final location instead. |
| `procure_method` | selection | yes | `make_to_stock` | stored | yes | no | See the closed list below. |
| `operation_type` | many_to_one to Operation Type | yes | none | stored | yes | no | The operation type stamped on the documents the rule creates. Must belong to the rule company. Restricted by `allowed_operation_type_codes`. |
| `allowed_operation_type_codes` | structured_data | no | derived | derived, not stored | not applicable | no | The list of operation type codes allowed for the current `action`: empty (meaning every code) for `pull`, `push` and `pull_push`; `["incoming", "dropship"]` for `buy` when drop shipping is installed, `["incoming"]` for `buy` otherwise; `["manufacturing"]` for `manufacture`. |
| `lead_time_days` | integer | no | 0 | stored | yes | no | The rule lead time in calendar days. Used to shift the scheduled date of the documents this rule creates. |
| `partner_address` | many_to_one to Contact | no | empty | stored | yes | no | Optional address where the goods should be delivered. When set, it overrides the partner taken from the procurement request. Must belong to the rule company. |
| `propagate_cancel` | boolean | no | false | stored | yes | no | When true, cancelling the move created by this rule also cancels the next move in the chain. |
| `propagate_carrier` | boolean | no | false | stored | yes | no | When true, the shipping method of the downstream document is propagated to the document created by this rule. |
| `warehouse` | many_to_one to Warehouse | no | empty | stored, indexed | yes | no | The warehouse this rule belongs to. An empty warehouse means the rule applies whatever the warehouse of the need. Must belong to the rule company. |
| `auto` | selection | yes | `manual` | stored | yes | no | `manual` = "Manual Operation": a push rule creates a new stock move after the current one. `transparent` = "Automatic No Step Added": a push rule rewrites the destination location of the existing move instead of creating a new one. |
| `rule_message` | rich_text | no | derived | derived, not stored | not applicable | no | A human-readable sentence describing what the rule does. Formula in `calculations.md`, section "Rule description message". |
| `push_condition` | text | no | empty | stored | yes | no | An optional condition expression. A push rule with a non-empty `push_condition` applies only to moves that satisfy the condition; otherwise rule selection continues with the next candidate push rule. |

Closed list for `action`:

| Value | Label | Meaning |
|---|---|---|
| `pull` | Pull From | When a need appears in `destination_location`, create a stock move from `location_source`. |
| `push` | Push To | When goods arrive in `location_source`, create a stock move to `destination_location` (or rewrite the destination when `auto` is `transparent`). |
| `pull_push` | Pull & Push | Both behaviors on the same rule. During need resolution it behaves as `pull`; during arrival handling it behaves as `push`. |
| `buy` | Buy | When a need appears in `destination_location`, create or extend a draft purchase order line. Added by the Purchase Inventory capability package. Deleting the selection value cascades: rules holding it are deleted when the capability package is removed. |
| `manufacture` | Manufacture | When a need appears in `destination_location`, create or extend a manufacturing order. Added by the Manufacturing capability package. Deleting the selection value cascades. |

Closed list for `procure_method`:

| Value | Label | Meaning |
|---|---|---|
| `make_to_stock` | Take From Stock | The goods are taken from the stock available in `location_source`. No supply need is created. |
| `make_to_order` | Trigger Another Rule | The available stock in `location_source` is ignored; a new need is created in `location_source` and rule selection runs again for it. |
| `mts_else_mto` (make to stock, else make to order) | Take From Stock, if unavailable, Trigger Another Rule | The goods are taken from the free stock of `location_source`; only the missing quantity creates a new need in `location_source`. |

### 2.2 Identity, ordering and display

- No uniqueness constraint on the rule name.
- Default ordering: by `sequence` ascending, then by `identifier` ascending.
- Display name: `name`.
- Candidate rules during rule selection are sorted by `route_sequence` ascending, then `sequence` ascending (see `workflows.md`, section "Rule selection").

### 2.3 Defaults on creation

When a rule is created through a form and the company is not supplied, the company defaults to the active company.

### 2.4 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Company consistency | If the route has a company, the rule company must equal it. Checked when `company` changes. | `Rule <rule name> belongs to <rule company> while the route belongs to <route company>.` |

### 2.5 On-change behavior in a form

| Field edited | Effect |
|---|---|
| `action` | When the new value is `buy`, `location_source` is emptied (a buy rule has no source location; the source of the receipt is the vendor location of the vendor chosen at run time). |
| `operation_type` | `location_source` is set to the operation type's default source location and `destination_location` is set to the operation type's default destination location. |
| `route` or `company` | If the route has a company, `company` is set to it. If the warehouse of the currently selected operation type belongs to a different company than the route, `operation_type` is emptied. |

### 2.6 Duplication

Duplicating a rule appends " (copy)" to `name`; every other field is copied unchanged.

### 2.7 Lifecycle

1. Created manually inside a route, or generated by warehouse configuration.
2. Selected by the rule-selection algorithm while `active` is true and its route is active.
3. Archived directly, or indirectly when its route is archived.
4. Deleted when its route is deleted.

### 2.8 Field visibility and editing rights

| Fields | Who may read | Who may edit |
|---|---|---|
| `name`, `active`, `action`, `sequence`, `company`, `route`, `route_company`, `route_sequence`, `location_destination_from_rule`, `procure_method`, `operation_type`, `allowed_operation_type_codes`, `lead_time_days`, `partner_address`, `propagate_cancel`, `propagate_carrier`, `auto`, `rule_message`, `push_condition` | every internal user, for the same reason as Route | Inventory / Administrator only |
| `destination_location`, `location_source` | every internal user | Inventory / Administrator, and only when the user also holds the hidden access group Manage Multiple Stock Locations; otherwise the two locations are hidden and keep the values the warehouse generator wrote |
| `warehouse` | every internal user | Inventory / Administrator, and only when the user also holds the hidden access group Manage Multiple Warehouses |

`route_company`, `route_sequence`, `allowed_operation_type_codes` and `rule_message` are derived and are never editable by anyone. The Stock Rules screen is hidden from users without the hidden access group Multi-Step Routes.

---

## 3. Reordering Rule

A Reordering Rule (also called a minimum stock rule) states that a given product at a given location must never be forecast below a minimum quantity, and that when it is, the system must order enough to reach a maximum quantity.

Identifier: `reordering_rule`. Kind: persistent, archivable, company-scoped.

### 3.1 Fields

| Canonical name | Type | Required | Default | Stored or derived | Copied on duplicate | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | text | yes | next value of the sequence with code `reordering_rule` (prefix `OP/`, five digits, starting at 1, incremented by 1, shared by all companies) | stored, read-only | no | no | The reference of the rule. Rules created by the replenishment report are named `Replenishment Report` instead. |
| `trigger` | selection (`auto` = Auto, `manual` = Manual) | yes | `auto` | stored | yes | no | `auto`: the scheduler runs the rule automatically. `manual`: the rule only appears on the replenishment report and waits for a user to press Order. |
| `active` | boolean | no | true | stored | yes | no | When false the rule is hidden and never evaluated. Archiving a product archives its reordering rules; unarchiving a product unarchives them. |
| `snoozed_until` | date | no | empty | stored | no | no | While this date is in the future the rule is hidden from the replenishment report. Only allowed on manual rules. |
| `warehouse` | many_to_one to Warehouse | yes | derived from `source_location` and `company`, user-editable, precomputed before insertion, deletion behavior `cascade`, indexed | stored | yes | no | The warehouse of the rule. |
| `source_location` | many_to_one to Location | yes | derived from `warehouse` and `company`, user-editable, precomputed before insertion, deletion behavior `cascade`, indexed | stored | yes | no | The location whose stock is watched and where the replenished goods must arrive. |
| `product` | many_to_one to Product Variant | yes | none, deletion behavior `cascade`, indexed | stored | yes | no | The product watched by the rule. Restricted to inventory-tracked goods. |
| `product_template` | many_to_one to Product Template | no | derived from `product.product_template` | derived, not stored | not applicable | no | Convenience relation. |
| `product_category` | many_to_one to Product Category | no | derived from `product.product_category` | derived, not stored | not applicable | no | Used for grouping on the replenishment report. |
| `unit_of_measure` | many_to_one to Unit of Measure | no | derived from `product.unit_of_measure` | derived, not stored | not applicable | no | The unit in which the minimum, maximum and quantity to order are expressed. |
| `product_unit_of_measure_name` | text | no | derived from `unit_of_measure.display_name` | derived, read-only, not stored | not applicable | no | Label shown next to quantities. |
| `product_minimum_quantity` | decimal (Product Unit) | yes | 0.0 | stored | yes | no | The minimum stock level that triggers a replenishment. |
| `product_maximum_quantity` | decimal (Product Unit) | yes | 0.0, then derived from `product_minimum_quantity`, user-editable | stored | yes | no | The stock level to reach when replenishing. Whenever `product_minimum_quantity` changes, if `product_maximum_quantity` is empty or lower than the new minimum, it is raised to the minimum. |
| `allowed_replenishment_unit_of_measures` | many_to_many to Unit of Measure | no | derived | derived, not stored | not applicable | no | The units that may be chosen as `replenishment_unit_of_measure`: every unit of the product, plus the units of the product's Vendor Prices when any selected rule has action `buy`, plus the unit of every matching bill of materials when any selected rule has action `manufacture`. |
| `replenishment_unit_of_measure` | many_to_one to Unit of Measure | no | empty | stored | yes | no | The multiple. The quantity to order is rounded up to a whole number of this unit. When empty no rounding to a multiple is applied, but a fallback multiple may still be derived (see `replenishment_unit_of_measure_identifier_placeholder`). |
| `replenishment_unit_of_measure_identifier_placeholder` | text | no | derived | derived, not stored | not applicable | no | The display name of the fallback multiple that would be used when `replenishment_unit_of_measure` is empty, shown as a grey placeholder in the form. Empty when no fallback exists. |
| `company` | many_to_one to Company | yes | the active company | stored, indexed | yes | no | The company of the rule. May never be changed after creation. |
| `allowed_locations` | one_to_many to Location | no | derived from `warehouse` | derived, not stored | not applicable | no | The locations that may be chosen as `source_location`: internal and view locations that either belong to this rule's warehouse or belong to no warehouse at all, restricted to the rule company or to no company. |
| `rules` | many_to_many to Stock Rule | no | derived | derived, not stored | not applicable | no | The chain of stock rules that would be walked to supply this product at this location, computed by the rule-chain walk of `calculations.md`, section "The rule chain from a location". |
| `lead_days` | decimal | no | derived | derived, not stored | not applicable | no | The total lead time in calendar days contributed by `rules` (rule lead times, vendor lead time, days to purchase, manufacturing lead time, days to supply components), excluding the replenishment horizon. |
| `lead_horizon_date` | date | no | derived | derived, not stored | not applicable | no | Today plus `lead_days` plus the replenishment horizon of the company. This is the date at which the forecast is read. |
| `route` | many_to_one to Route | no | empty | stored | yes | no | The preferred route for this rule. Restricted to routes that are selectable on products or that contain a rule with action `buy` or `manufacture`. Setting it to empty also clears `vendor_price` (see below). |
| `route_identifier_placeholder` | text | no | derived | derived, not stored | not applicable | no | The display name of the route that would be used if `route` were empty. |
| `effective_route` | many_to_one to Route | no | derived | derived, not stored, searchable | not applicable | no | `route` when set, otherwise the default route computed by the algorithm in `calculations.md`, section "Default route of a reordering rule". |
| `quantity_on_hand` | decimal (Product Unit) | no | derived, read-only | derived, not stored | not applicable | no | The physical quantity of the product currently in `source_location` and its children. |
| `quantity_forecast` | decimal (Product Unit) | no | derived, read-only | derived, not stored | not applicable | no | The forecast quantity at `lead_horizon_date` in `source_location`, plus the quantity already in progress that the forecast does not see (draft and pending purchase order lines, and pending manufacturing orders). |
| `quantity_to_order_computed` | decimal (Product Unit) | no | derived and stored | derived, stored | no | no | The quantity the system computes. Formula in `calculations.md`, section "Quantity to order". |
| `quantity_to_order_manual` | decimal (Product Unit) | no | 0 | stored | no | no | A quantity typed by the user that overrides the computed one. |
| `quantity_to_order` | decimal (Product Unit) | no | derived, writable through an inverse rule, searchable | derived, not stored | no | no | `quantity_to_order_manual` when it is non-zero, otherwise `quantity_to_order_computed`. |
| `days_to_order` | decimal | no | derived | derived, not stored | not applicable | no | The number of days in advance that the demand is created. Zero by default; set to the company's days to purchase when any selected rule has action `buy`; set to the bill of materials' "days to supply components" when any selected rule has action `manufacture`. |
| `unwanted_replenish` | boolean | no | derived | derived, not stored | not applicable | no | True when ordering `quantity_to_order` would push the forecast above `product_maximum_quantity`. Used to warn the user. |
| `show_supply_warning` | boolean | no | derived | derived, not stored | not applicable | no | True when no stock rule at all was found for this product and location, or (when a `buy` rule was found) when the product has no Vendor Price. Used to show the "no supply method" warning on the replenishment report. |
| `deadline_date` | date | no | derived and stored, read-only | derived, stored | no | no | The last date on which an order must be placed to avoid falling below the minimum. Formula in `calculations.md`, section "Deadline date". Empty when no shortage is foreseen inside the horizon. |
| `vendor_price` | many_to_one to Vendor Price | no | empty | stored | yes | no | The specific vendor price line to use for this rule. When set, it bypasses vendor selection at run time. Restricted to vendor prices of this product or of its product template, and to the rule company. Setting it while `route` is empty sets `route` to the first route containing a `buy` rule. |
| `vendor_price_identifier_placeholder` | text | no | derived | derived, not stored | not applicable | no | The display name of the vendor price that would be selected if `vendor_price` were empty. |
| `vendors` | one_to_many to Vendor Price | no | derived from `product.vendor_prices` | derived, not stored | not applicable | no | Every vendor price of the product. |
| `effective_vendor` | many_to_one to Contact | no | derived, searchable | derived, not stored | not applicable | no | The contact of `vendor_price` when set, otherwise the contact of the vendor price that would be selected. |
| `available_vendor` | many_to_one to Contact | no | search-only | not stored | not applicable | no | A search-only field that matches any reordering rule whose product has a vendor price for the searched contact. |
| `show_vendor` | boolean | no | derived | derived, not stored | not applicable | no | True when `effective_route` is a route containing a rule with action `buy`. Controls whether the vendor column is shown. |
| `bill_of_materials` | many_to_one to Bill of Materials | no | empty | stored | yes | no | The specific bill of materials to use for this rule. Restricted to bills of type `normal`, of this company or of no company, and for this product or product template. Setting it while `route` is empty sets `route` to the first route containing a `manufacture` rule. |
| `bill_of_materials_identifier_placeholder` | text | no | derived | derived, not stored | not applicable | no | The display name of the bill of materials that would be used if `bill_of_materials` were empty. |
| `effective_bill_of_materials` | many_to_one to Bill of Materials | no | derived, searchable | derived, not stored | not applicable | no | `bill_of_materials` when set, otherwise the bill that would be found. |
| `show_bill_of_materials` | boolean | no | derived | derived, not stored | not applicable | no | True when `effective_route` is a route containing a rule with action `manufacture`. |

**Naming choices.** Three field names of this entity were chosen rather than derived mechanically, and are used with these names throughout this folder. `vendor_price` names the many_to_one link to a Vendor Price record, because "Vendor Price" is the canonical display name of that entity and because the field holds a price line, not a contact; the contact behind it is `effective_vendor`. `vendor_price_identifier_placeholder` follows the same reading for the grey placeholder that shows what vendor price would be selected if the field were left empty. `show_vendor` keeps the word "vendor" rather than "supplier" so that one word names one concept across the whole folder: every contact this domain buys from is a vendor.

### 3.2 Identity and uniqueness

- Database uniqueness constraint on the triple (`product`, `source_location`, `company`). Violation message: `A replenishment rule already exists for this product on this location.`
- The constraint is enforced across archived records as well, therefore an archived rule for the same product and location blocks the creation of a new one. Code that creates rules automatically (the replenishment report) searches including archived records for this reason.
- Default ordering: `source_location` ascending, then `company` ascending, then `identifier` ascending.
- Display name: `name`.

### 3.3 Validation rules

| Rule | Condition | Message |
|---|---|---|
| Minimum not above maximum | Checked when `product_minimum_quantity` or `product_maximum_quantity` changes. | `The minimum quantity must be less than or equal to the maximum quantity.` |
| Product must not be a kit | Checked when `product` changes. Refused when the product has a bill of materials of type "kit". | `A product with a kit-type bill of materials can not have a reordering rule.` |
| Snooze only on manual rules, at creation | Refused when a record is created with a non-empty `snoozed_until` while `trigger` is `auto` (including the case where `trigger` is not supplied and defaults to `auto`). | `You can not create a snoozed orderpoint that is not manually triggered.` |
| Snooze only on manual rules, at modification | Refused when `snoozed_until` is written on any record whose `trigger` is `auto`. | `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.` |
| Company is immutable | Refused when `company` is written with a value different from the current one. | `Changing the company of this record is forbidden at this point, you should rather archive it and create a new one.` |
| Warehouse must exist | When computing `warehouse` no warehouse can be found for the company, the system raises the shared "no warehouse configured" redirect warning owned by `../inventory-operations/`. | (Owned by the Inventory Operations domain.) |

### 3.4 On-change behavior in a form

| Field edited | Effect |
|---|---|
| `product` | `unit_of_measure` is set to the product's unit of measure. |
| `warehouse` | `source_location` is recomputed as the warehouse's stock location. If no warehouse is set, the first warehouse of the company is used. |
| `source_location` | `warehouse` is recomputed as the warehouse of that location; if the location belongs to no warehouse, the first warehouse of the company is used. |
| `product_minimum_quantity` | `product_maximum_quantity` is raised to the new minimum when it is empty or lower. |
| `quantity_to_order` | Writing this field runs the inverse rule: if `trigger` is `auto`, `quantity_to_order_manual` is reset to 0 (an automatic rule may not carry a manual override). Otherwise, if both the manual override and the new value are empty, the field falls back to `quantity_to_order_computed`. Otherwise, if the new value differs from `quantity_to_order_computed`, `quantity_to_order_manual` is set to the new value. |
| `route` | Writing an empty route also empties `vendor_price`. |
| `vendor_price` | Writing a non-empty vendor price while `route` is empty sets `route` to the route of the first rule with action `buy`. |
| `bill_of_materials` | Writing a non-empty bill of materials while `route` is empty sets `route` to the route of the first rule with action `manufacture`. |

### 3.5 Search behavior of derived fields

- Searching on `quantity_to_order` with any operator matches: records whose `quantity_to_order_manual` is non-zero and satisfies the operator, or records whose `quantity_to_order_manual` is zero or empty and whose `quantity_to_order_computed` satisfies the operator.
- Searching on `effective_route`, `effective_vendor` or `effective_bill_of_materials` evaluates the derived value on every reordering rule and matches those whose value is in the searched set.
- Searching on `available_vendor` matches reordering rules whose product has at least one vendor price for a contact in the searched set.

### 3.6 Automatic recomputation triggers

`quantity_to_order_computed` and `quantity_forecast` are invalidated and queued for recomputation whenever a stock move is written whose `product`, `state`, `date`, `demand_quantity`, `source_location` or `destination_location` changed. Only the reordering rules for that product in the warehouses of the move's source and destination locations are queued, not every rule of the product.

### 3.7 Automatic cleanup

A background vacuum task deletes every reordering rule that was created by the superuser, has `trigger` equal to `manual`, and whose `quantity_to_order` is zero or negative. These are the temporary rules created by the replenishment report; the deletion removes suggestions that have been satisfied.

### 3.8 Lifecycle

| From | Trigger | Guard | To | Side effects |
|---|---|---|---|---|
| (none) | A user creates a rule | Uniqueness, minimum/maximum and kit constraints pass | Active, `trigger` as chosen | A sequence value is consumed for `name`. |
| (none) | The replenishment report detects a negative forecast | No rule (active or archived) exists for that product and location | Active, `trigger` = `manual`, `name` = `Replenishment Report`, minimum = 0, maximum = 0, created by the superuser | The rule appears on the replenishment report with the shortage as quantity to order. |
| Active manual | A user snoozes the rule | `trigger` = `manual` | Active manual, `snoozed_until` set | The rule disappears from the replenishment report until that date. |
| Active manual, created by the superuser | The quantity to order drops to zero or below | | Deleted | Removed by the vacuum task or by opening the replenishment report. |
| Active | A user presses Order | `quantity_to_order` greater than zero | Active | A procurement request is run; the manual quantity override is cleared. |
| Active | A user presses Order to Max | | Active | `quantity_to_order` is first forced to the multiple-rounded value of maximum minus forecast, then the procurement request is run. |
| Active | A user archives the rule | | Archived | The rule is no longer evaluated. |
| Active | The product is archived | | Archived | Archiving a product archives all of its reordering rules. |

### 3.9 Field visibility and editing rights

| Fields | Who may read | Who may edit |
|---|---|---|
| `name`, `trigger`, `active`, `snoozed_until`, `product`, `product_template`, `product_category`, `unit_of_measure`, `product_unit_of_measure_name`, `product_minimum_quantity`, `product_maximum_quantity`, `allowed_replenishment_unit_of_measures`, `replenishment_unit_of_measure`, `replenishment_unit_of_measure_identifier_placeholder`, `company`, `rules`, `lead_days`, `lead_horizon_date`, `route`, `route_identifier_placeholder`, `effective_route`, `quantity_on_hand`, `quantity_forecast`, `quantity_to_order_computed`, `quantity_to_order_manual`, `quantity_to_order`, `days_to_order`, `unwanted_replenish`, `show_supply_warning`, `deadline_date`, `vendor_price`, `vendor_price_identifier_placeholder`, `vendors`, `effective_vendor`, `available_vendor`, `show_vendor`, `bill_of_materials`, `bill_of_materials_identifier_placeholder`, `effective_bill_of_materials`, `show_bill_of_materials` | Inventory / User, Inventory / Administrator, Purchase / User, Purchase / Administrator | Inventory / Administrator only |
| `source_location`, `allowed_locations` | the same groups | Inventory / Administrator, and only when the user also holds the hidden access group Manage Multiple Stock Locations; otherwise the location column is hidden and the warehouse stock location is used |
| `warehouse` | the same groups | Inventory / Administrator, and only when the user also holds the hidden access group Manage Multiple Warehouses; otherwise the column is hidden and the single warehouse is used |

`name`, `quantity_on_hand`, `quantity_forecast`, `deadline_date` and every field whose row in section 3.1 says "derived" are shown read-only to every group. `product_minimum_quantity`, `product_maximum_quantity` and `quantity_to_order` are additionally writable from the Replenishment Information dialog of section 5, which writes them back with the acting user's own rights, not with elevated rights, so a user who may not edit a reordering rule cannot edit it through that dialog either.

---

## 4. Reordering Rule Snooze Wizard

A transient record used to postpone one or more manual reordering rules.

Identifier: `reordering_rule_snooze_wizard`. Kind: transient (discarded by the periodic transient-record cleanup).

| Canonical name | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `orderpoints` | many_to_many to Reordering Rule | no | the selected records | The rules to snooze. |
| `predefined_date` | selection (`day` = 1 Day, `week` = 1 Week, `month` = 1 Month, `custom` = Custom) | no | `day` | Shortcut for computing the snooze date. |
| `snoozed_until` | date | no | empty | The date until which the rules are hidden. |

On-change: when `predefined_date` changes, `snoozed_until` is set to today plus one day, today plus one week, or today plus one month respectively; when the value is `custom` the date is left untouched for the user to type.

Operation `action_snooze`: writes `snoozed_until` on every rule in `orderpoints`. Because the snooze validation rule of the Reordering Rule applies, the operation fails with `You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered.` when any selected rule is automatic.

---

## 5. Replenishment Information

A transient record that explains, for one reordering rule, where the lead times come from, what the forecast date is, what the historic demand looks like, which supplying warehouses could serve the need, and which vendor prices exist.

Identifier: `replenishment_information`. Kind: transient. Display name is taken from `orderpoint`.

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `orderpoint` | many_to_one to Reordering Rule | yes in practice | supplied by the opening action | stored | The rule being explained. |
| `product` | many_to_one to Product Variant | no | derived from `orderpoint.product` | derived | |
| `product_unit_of_measure_name` | text | no | derived from `orderpoint.product_unit_of_measure_name` | derived | |
| `product_minimum_quantity` | decimal | yes | derived from `orderpoint.product_minimum_quantity`, writable (writes back to the rule with the current user's rights, not elevated rights) | derived, writable | |
| `product_maximum_quantity` | decimal | yes | derived from `orderpoint.product_maximum_quantity`, writable (writes back to the rule) | derived, writable | |
| `quantity_to_order` | decimal | no | derived from `orderpoint.quantity_to_order` | derived | |
| `structured_data_lead_days` | text holding a structured data document | no | derived | derived | The lead-time breakdown. Content specified in `interfaces.md`, section "Replenishment Information payload". |
| `structured_data_replenishment_graph` | text holding a structured data document | no | derived | derived | The demand graph. Content and formulas in `calculations.md`, section "Replenishment demand graph". |
| `based_on` | selection | yes | `one_month` | stored | The historic period used to estimate daily demand. Closed list: `one_week` = Last 7 days, `one_month` = Last 30 days, `three_months` = Last 3 months, `one_year` = Last 12 months, `last_year` = Same month last year, `last_year_2` = Next month last year, `last_year_3` = After next month last year, `last_year_quarter` = Last year quarter. |
| `percent_factor` | integer | yes | 100 | stored | A percentage applied to the estimated daily demand. |
| `resupply_routes` | one_to_many to Route | no | derived from `orderpoint.warehouse.resupply_routes` | derived | The inter-warehouse resupply routes that feed the rule's warehouse. |
| `warehouse_replenishment_options` | one_to_many to Replenishment Option | no | derived | derived | One option per resupply route, created on the fly and sorted by free-to-use quantity descending. |
| `vendor_price` | many_to_one to Vendor Price | no | derived from `orderpoint.vendor_price` | derived, writable (writes back to the rule) | The single vendor price the reordering rule currently holds, that is the one that bypasses vendor selection at run time. Empty when the rule leaves the choice to vendor selection. The vendor tab of the dialog marks this row as the selected one, and the `action_set_vendor_price` operation of section 20 writes a different row into it. |
| `vendor_prices` | many_to_many to Vendor Price | no | derived and stored | derived | Every vendor price of the product. |
| `show_vendor_tab` | boolean | no | derived | derived | True when the rule has no preferred route, or when the rule has a preferred route and at least one selected rule has action `buy`. |
| `bill_of_materials` | many_to_one to Bill of Materials | no | derived from `orderpoint.bill_of_materials` | derived | |
| `bills_of_materials` | many_to_many to Bill of Materials | no | derived and stored | derived | Every bill of materials that could produce the product. |
| `show_bill_of_materials_tab` | boolean | no | derived | derived | True when the bills-of-materials tab should be shown. |

**Naming choices.** Four field names of this entity were chosen rather than derived mechanically, and are used with these names throughout this folder: `vendor_price` and `vendor_prices` (the singular link the rule holds and the list of every candidate), because "Vendor Price" is the canonical display name of the entity they point at; `bill_of_materials` and `bills_of_materials` (the singular link and the list), because "Bill of Materials" is the canonical display name and its plural is irregular. The same two pairs appear on Reordering Rule (section 3.1), where `vendor_price`, `vendor_price_identifier_placeholder` and `show_vendor` carry the same reading: `vendor_price` is the chosen Vendor Price record, not the vendor contact, which is `effective_vendor`.

---

## 6. Replenishment Option

A transient record representing one candidate supplying warehouse inside the Replenishment Information wizard.

Identifier: `replenishment_option`. Kind: transient.

| Canonical name | Type | Required | Default | Stored or derived | Meaning |
|---|---|---|---|---|---|
| `route` | many_to_one to Route | no | supplied at creation | stored | The inter-warehouse resupply route this option would use. |
| `product` | many_to_one to Product Variant | no | supplied at creation | stored | |
| `replenishment_information` | many_to_one to Replenishment Information | no | supplied at creation | stored | The parent wizard. |
| `warehouse` | many_to_one to Warehouse | no | derived from `route.supplier_warehouse` | derived | The supplying warehouse. |
| `source_location` | many_to_one to Location | no | derived from `warehouse.lot_stock` | derived | The stock location of the supplying warehouse. |
| `unit_of_measure` | text | no | derived from `product.unit_of_measure_name` | derived | |
| `quantity_to_order` | decimal | no | derived from `replenishment_information.quantity_to_order` | derived | |
| `free_to_use_quantity` | decimal | no | derived | derived | The unreserved quantity of the product available in `source_location`. |
| `lead_time` | text | no | derived | derived | The string "<n> days", where <n> is the total lead time of the rule that would be selected for this product at `source_location` with this route, or 0 when no rule is found. |
| `warning_message` | text | no | derived | derived | Empty when `free_to_use_quantity` is at least `quantity_to_order`; otherwise `<warehouse name> can only provide <free quantity> <unit>, while the quantity to order is <quantity to order> <unit>.` |

Operations:

- `select_route`: when `free_to_use_quantity` is lower than `quantity_to_order`, opens the warning form of this same record titled "Quantity available too low". Otherwise it behaves as `order_all`. When the wizard was opened from the Product Replenish Wizard (that wizard's identifier is carried in the opening context), it instead writes the route on that wizard and reopens it.
- `order_available` ("Order available quantity"): writes `route` on the parent reordering rule and sets the rule's `quantity_to_order` to `free_to_use_quantity`, then closes the window.
- `order_all`: writes `route` on the parent reordering rule and closes the window; the quantity to order is unchanged.

---

## 7. Stock Rules Report wizard

A transient record used to print the routes diagram of a product.

Identifier: `stock_rules_report`. Kind: transient.

| Canonical name | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `product` | many_to_one to Product Variant | yes | from the opening context: the product when opened on a variant, the first variant when opened on a template | The product whose routes are drawn. |
| `product_template` | many_to_one to Product Template | yes | from the opening context | The template of `product`. |
| `product_has_variants` | boolean | yes | false; true when the template has more than one variant | Controls whether a variant selector is shown. |
| `warehouses` | many_to_many to Warehouse | yes | the first warehouse of the product's company, or of the active company | The warehouses whose rules are drawn. |
| `sales_order_routes` | many_to_many to Route | no | empty | Extra routes to include in the drawing, restricted to routes selectable on sales order lines. Added by the Sales Inventory capability package. |

If no warehouse exists for the company, opening the wizard raises the shared "no warehouse configured" redirect warning owned by `../inventory-operations/`.

Operation `print_report`: renders the routes diagram document described in `interfaces.md`, section "Routes diagram".

---

## 8. Forecasted Stock Report and Stock Replenishment Report

Two abstract report entities. They hold no fields of their own; they expose read operations that build the forecast data structure for one or more products.

| Identifier | Input | Purpose |
|---|---|---|
| `forecasted_stock_report` | a list of product variant identifiers | Builds the forecast payload for those variants. |
| `stock_replenishment_report` | a list of product template identifiers | Builds the same payload for every variant of those templates. It reuses every operation of `forecasted_stock_report`. |

The payload structure and the reconciliation algorithm are specified in `calculations.md`, section "Forecast reconciliation", and in `interfaces.md`, section "Forecast report payload".

---

## 9. Vendor Delay Report

A read-only database view that measures vendor punctuality. One row per purchase order line.

Identifier: `vendor_delay_report`. Kind: read-only derived view; it has no shared persistent fields other than `identifier`, which equals the identifier of the purchase order line.

| Canonical name | Type | Meaning |
|---|---|---|
| `partner` | many_to_one to Contact, read-only | The vendor of the purchase order line. |
| `product` | many_to_one to Product Variant, read-only | The product of the purchase order line. |
| `category` | many_to_one to Product Category, read-only | The category of that product. |
| `date` | datetime, read-only | The earliest date among the stock moves linked to the purchase order line. |
| `quantity_total` | decimal, read-only | The ordered quantity of the purchase order line, in the line's unit. |
| `quantity_on_time` | decimal, read-only | The sum, over the stock move lines of the completed moves of that purchase order line whose move date (date part only) is not later than the line's planned date (date part only), of the move line quantity converted into the product's reference unit. |
| `on_time_rate` | decimal, read-only | A derived aggregate, not a stored column: see below. |

Row construction: one row per purchase order line that has at least one stock move. Grouping is by purchase order line. `quantity_on_time` sums only the move lines of moves in state `done` whose date is on or before the line's planned date; every other move line contributes zero.

Aggregation of `on_time_rate`: when a report query asks for the sum of `on_time_rate`, the value returned is a weighted average, not a sum:

```
on_time_rate_percentage =
    when sum(quantity_total) ≠ 0
        then sum(quantity_on_time) ÷ sum(quantity_total) × 100
        else 100
```

Groups whose `sum(quantity_total)` is not greater than zero are removed from the result.

---

## 10. The procurement request structure

A procurement request is the unit of work of this domain. It is never stored as a record; it is an ordered tuple passed to the run operation. Its values are serialized onto the stock moves that the run operation creates, in the `procurement_values` field of Stock Move, so that later steps of a chain can read them.

### 10.1 Positional members

| Position | Member | Type | Meaning |
|---|---|---|---|
| 1 | `product` | reference to a Product Variant | The product that is needed. |
| 2 | `product_quantity` | decimal | The quantity needed, expressed in `product_unit_of_measure`. May be negative, which expresses a return. |
| 3 | `product_unit_of_measure` | reference to a Unit of Measure | The unit of `product_quantity`. |
| 4 | `source_location` | reference to a Location | The location where the need exists. Named `location` in prose; rule selection walks up from this location. |
| 5 | `name` | text | A short name; becomes the name of the created move. |
| 6 | `origin` | text | A source-document label; becomes the `origin` of the created documents. |
| 7 | `company` | reference to a Company | The company of the need. |
| 8 | `values` | key-value map | Everything else, listed below. |

### 10.2 The `values` map

The run operation guarantees three defaults before anything else happens:

| Key | Default applied when absent or empty |
|---|---|
| `company` | the company of `source_location` |
| `priority` | `"0"` |
| `date_planned` | the current moment |

The complete set of keys recognized anywhere in this domain:

| Key | Type | Set by | Used by |
|---|---|---|---|
| `company` | reference to Company | run operation default; sales order line; reordering rule | Rule selection (restricting rules to the company tree when running with elevated rights). |
| `priority` | text (`"0"` normal, `"1"` urgent) | run operation default; stock move; sales order line | Written on the created move. |
| `date_planned` | datetime | every producer | The date the goods are needed at `source_location`. Every downstream date is computed backwards from it. |
| `date_order` | datetime | reordering rule; stock move | The date the supplying document should be placed. Used as the purchase order date. |
| `date_deadline` | datetime | sales order line; reordering rule; stock move | The date the goods must be available to keep a downstream promise. Propagated along the chain. |
| `move_destinations` | list of references to Stock Move | stock move (when its supply method is make to order); push rule | The downstream moves that the created document must feed. |
| `routes` | list of references to Route | sales order line; reordering rule; stock move; replenish wizard | Routes offered first to rule selection. |
| `warehouse` | reference to Warehouse | sales order line; reordering rule; stock move; replenish wizard | Restricts candidate rules to that warehouse or to rules with no warehouse. |
| `packaging_unit_of_measure` | reference to Unit of Measure | sales order line; stock move | The packaging unit; its package type's routes are offered to rule selection after the explicit routes. |
| `partner` | reference to Contact | sales order line; stock move; subcontracting resupply | The delivery address written on the created move or purchase order. |
| `orderpoint` | reference to Reordering Rule | reordering rule; stock move | Written on the created move and on the created purchase order line; used as a merge key. |
| `references` | list of references to Reference between stock documents | sales order line; reordering rule; stock move; purchase order | Links the created documents back to the originating documents. |
| `sales_order_line` | reference to Sales Order Line | sales order line | A merge key: procurements from different sales order lines never share a purchase order line. |
| `product_description_variants` | text | sales order line; stock move | Extra description appended to the created purchase order line name; also a merge key. |
| `never_product_template_attribute_values` | list of references to Product Template Attribute Value | sales order line; stock move | Attribute values excluded from the created documents. |
| `forced_vendor_price` | reference to Vendor Price | reordering rule; replenish wizard | Forces the vendor price instead of running vendor selection. |
| `vendor_contact` | reference to Contact | purchase requisition flows | Restricts vendor selection to that contact. |
| `chosen_vendor_price` | reference to Vendor Price | the buy action, internally | The vendor price chosen; read when preparing the purchase order and its lines. |
| `force_unit_of_measure` | boolean | the Product Replenish Wizard | When true, the unit of the request is kept instead of being converted to the vendor's purchase unit. |
| `propagate_cancel` | boolean | the buy action, internally | Copied from the rule onto the purchase order line; also a merge key. |
| `bill_of_materials` | reference to Bill of Materials | manufacturing flows | Forces the bill of materials. |
| `bill_of_materials_line` | reference to Bill of Materials Line | kit explosion | Written on the created move. |
| `production_group` | reference to Production Group | manufacturing flows | Written on the created move and manufacturing order. |
| `location_final` | reference to Location | sales order line | The ultimate destination of the chain. |
| `sequence` | integer | sales order line | Ordering of the created lines. |
| `procurement_values` | key-value map | stock move | The values map carried over from an earlier step of the chain. |
| `to_refund` | boolean | set by the pull action when the quantity is negative | Marks the created move as a refund-bearing return. |

**Naming note.** Every key of this map is written with full words and without relational suffixes, and is used with that name consistently throughout this folder. The keys whose name was chosen rather than taken verbatim are, with the meaning that fixes the choice:

| Key | Why the name was chosen |
|---|---|
| `move_destinations` | A list of references to the downstream Stock Moves the created document must feed. Plural, because the relation holds many records. |
| `routes` | A list of references to Route. Plural, because the relation holds many records. |
| `warehouse` | A single reference to a Warehouse. |
| `partner` | A single reference to a Contact, the delivery address. |
| `orderpoint` | A single reference to a Reordering Rule. The same name is used for the corresponding field on Stock Move and on Purchase Order Line, so that one concept carries one name. |
| `references` | A list of references to the Reference records that tie stock documents together. Plural, because the relation holds many records. |
| `sales_order_line` | A single reference to a Sales Order Line, spelled in full rather than shortened. |
| `never_product_template_attribute_values` | A list of references to Product Template Attribute Value. Plural, because the relation holds many records. |
| `production_group` | A single reference to a Production Group. |
| `location_final` | A single reference to a Location, the ultimate destination of the chain. |
| `forced_vendor_price` | The Vendor Price forced by the caller. |
| `chosen_vendor_price` | The Vendor Price the buy action selected. |
| `vendor_contact` | A Contact that restricts vendor selection. |
| `bill_of_materials` and `bill_of_materials_line` | The Bill of Materials and one of its lines, spelled in full. |
| `packaging_unit_of_measure` and `force_unit_of_measure` | The packaging unit of measure, and the switch that keeps the request unit. |

### 10.3 Serialization onto a stock move

When a pull rule creates a move it stores the whole `values` map on the move, after converting each entry as follows:

1. A reference or list of references becomes the list of referenced identifiers.
2. A date or datetime becomes its text representation in the standard year-month-day (and hour-minute-second) form.
3. Every other value is stored unchanged.

---

## 11. Fields added to Warehouse (owned by `../inventory-operations/`)

| Canonical name | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `routes` | many_to_many to Route | empty | stored, not copied | The default routes of the warehouse. Restricted to routes with `warehouse_selectable` true and belonging to the warehouse company or to no company. |
| `reception_route` | many_to_one to Route | empty | stored, deletion behavior `restrict`, not copied | The route holding the receipt steps of this warehouse. |
| `delivery_route` | many_to_one to Route | empty | stored, deletion behavior `restrict`, not copied | The route holding the delivery steps of this warehouse. |
| `make_to_order_pull` | many_to_one to Stock Rule | empty | stored, not copied | The rule of the global "Replenish on Order" route belonging to this warehouse. |
| `resupply_warehouses` | many_to_many to Warehouse | empty | stored | The warehouses that may resupply this one. Writing this field creates or archives inter-warehouse resupply routes. |
| `resupply_routes` | one_to_many to Route (inverse `supplied_warehouse`) | derived from the routes | stored on the Route side, not copied | The inter-warehouse resupply routes generated for this warehouse. |
| `buy_to_resupply` | boolean | true | derived from whether this warehouse is listed on the global Buy route, writable through an inverse rule | When true the warehouse may be supplied by purchasing. Writing it adds or removes this warehouse from the `warehouses` list of the Buy route. |
| `buy_pull` | many_to_one to Stock Rule | empty | stored, not copied | The `buy` rule of the global Buy route belonging to this warehouse. |
| `subcontracting_dropshipping_pull` | many_to_one to Stock Rule | empty | stored, not copied | The pull rule that moves components from the subcontracting location to the production location, inside the Dropship route. Added by the Dropship and Subcontracting Management capability package. |

The complete generation logic for these routes and rules is specified in `configuration.md`, section "Routes and rules generated by a warehouse", and in `workflows.md`, section "Warehouse creation and reconfiguration".

---

## 12. Fields added to Stock Move (owned by `../inventory-operations/`)

| Canonical name | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `rule` | many_to_one to Stock Rule | empty | stored | The stock rule that created this move. |
| `procure_method` | selection (`make_to_stock` = "Default: Take From Stock", `make_to_order` = "Advanced: Apply Procurement Rules") | `make_to_stock` | stored, required | How this move gets its goods. A make-to-order move creates a procurement request at its source location when it is confirmed and waits for the resulting move. |
| `move_destinations` | many_to_many to Stock Move | empty | stored | The downstream moves this move feeds. |
| `move_origins` | many_to_many to Stock Move (inverse of `move_destinations`) | empty | stored | The origin moves that feed this move. |
| `location_final` | many_to_one to Location | empty | stored, writable | The ultimate destination of the chain this move belongs to. The move brings goods to `destination_location`, which may be an intermediate location on the way to `location_final`. |
| `deadline` | datetime | empty | stored, read-only, writable through an inverse rule | The date by which the move must be completed to keep a downstream promise. Writing it propagates the same shift to origin and destination moves (see `calculations.md`, section "Deadline propagation"). |
| `delay_alert_date` | datetime | empty | derived and stored | The latest scheduled date among the not-yet-completed origin moves, when that date is later than this move's own scheduled date; empty otherwise, and always empty for completed or cancelled moves. |
| `orderpoint` | many_to_one to Reordering Rule | empty | stored | The reordering rule that caused this move. |
| `routes` | many_to_many to Route | empty | stored | Preferred routes carried by the move, offered first to rule selection when this move creates a procurement request or applies a push rule. |
| `warehouse` | many_to_one to Warehouse | empty | stored | The warehouse to consider for rule selection on the next procurement. |
| `propagate_cancel` | boolean | true | stored | When true, cancelling this move cancels the downstream move. |
| `procurement_values` | structured_data | empty | not stored (kept only in memory for the duration of the operation) | The serialized procurement values, propagated to later steps. |
| `purchase_line` | many_to_one to Purchase Order Line | empty | stored, read-only, indexed (index skips empty values), deletion behavior `set null` | The purchase order line that generated this move. |
| `created_purchase_lines` | many_to_many to Purchase Order Line | empty | stored, not copied | The purchase order lines that were created to supply this move. A make-to-order move that triggered a buy rule holds the created line here until the purchase order is confirmed and the receipt move is created. |
| `demand_quantity_in_reference_unit` | decimal | derived from `demand_quantity` | derived | The move's demand quantity converted into the reference unit of the product's unit category. Owned by `../inventory-operations/`; listed here because the confirm-time split and the forecast reconciliation of `calculations.md` compare quantities in that unit. |

---

## 13. Fields added to Transfer (owned by `../inventory-operations/`)

| Canonical name | Type | Stored or derived | Meaning |
|---|---|---|---|
| `purchase` | many_to_one to Purchase Order | derived from `stock_moves.purchase_line.purchase_order`, read-only | The purchase order this transfer receives. |
| `days_to_arrive` | datetime | derived, searchable (searching it searches the completion date), not copied | The completion date of the transfer when the transfer is completed and its destination location is not a vendor location; empty otherwise. Used by the purchase analysis report. |
| `linked_purchase_order_date` | datetime | derived, searchable (searching it searches the purchase order date), indexed, not copied | The order date of the linked purchase order, or the current moment when there is none. |
| `is_dropship` | boolean | derived | True when the source location is a vendor location, or a transit location with no company, **and** the destination location is a customer location, or a transit location with no company. Added by the Drop Shipping capability package. |

Behavior added: completing a transfer acknowledges its purchase order (a purchase-domain operation that records vendor acknowledgement) before the normal completion processing runs.

---

## 14. Fields added to Operation Type (owned by `../inventory-operations/`)

| Change | Detail |
|---|---|
| `code` gains the value `dropship` ("Dropship") | Added by the Drop Shipping capability package. Removing that package rewrites affected operation types to `outgoing` and archives them. |
| Default source location | For an operation type with code `dropship` the default source location is forced to the shared vendor location. |
| Default destination location | For an operation type with code `dropship` the default destination location is forced to the shared customer location. |
| `warehouse` | Forced to empty for an operation type with code `dropship`. |
| `show_in_operations_overview` | Forced to true for an operation type with code `dropship`, so that it always appears in the operations overview. |

---

## 15. Fields added to Location (owned by `../inventory-operations/`)

| Canonical name | Type | Default | Meaning |
|---|---|---|---|
| `replenish_location` | boolean | derived from `usage`, writable, stored, not copied | When true, the replenishment report watches this location and creates temporary reordering rules for negative forecasts in it. Forced to false for any location whose usage is not `internal`. The stock location of each warehouse is created with this flag set to true. |

Validation: a location may not be a replenish location while one of its ancestors or descendants is also one. Message: `Another parent/sub replenish location <name> exists, if you wish to change it, uncheck it first`.

---

## 16. Fields added to Purchase Order (owned by `../purchasing/`)

| Canonical name | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `operation_type` | many_to_one to Operation Type | the first incoming operation type of the order company, else the first incoming operation type with no warehouse, else the first such archived one | stored, required | Labelled "Deliver To". Determines the operation type of the receipt and, for a drop shipping operation type, that the goods never enter the company's warehouse. Restricted to operation types with no warehouse or whose warehouse belongs to the order company. |
| `default_location_destination_identifier_usage` | selection | derived from `operation_type.default_location_destination.usage`, read-only | derived | Used to decide whether the drop-ship address field is shown. |
| `destination_address` | many_to_one to Contact | derived, writable, stored | derived | Emptied whenever the operation type's default destination location is not a customer location. For a drop shipping order it holds the customer. |
| `transfers` | many_to_many to Transfer | derived from `order_lines.stock_moves.transfer`, stored | derived | The receipts (or drop shipments) of the order. |
| `incoming_transfer_count` | integer | derived | derived | The number of transfers of the order, minus the number of drop shipments. |
| `dropship_transfer_count` | integer | derived | derived | The number of transfers of the order that are drop shipments. |
| `references` | many_to_many to Reference between stock documents | empty, not copied | stored | The stock references that link this order to the documents that caused it. |
| `is_shipped` | boolean | derived | derived | True when the order has at least one transfer and every transfer is completed or cancelled. |
| `effective_date` | datetime | derived and stored, not copied | derived | Labelled "Arrival". The earliest completion date among the order's completed transfers whose destination location is not a vendor location. |
| `receipt_status` | selection (`pending` = Not Received, `partial` = Partially Received, `full` = Fully Received) | derived and stored | derived | Empty when the order has no transfer or every transfer is cancelled; `full` when every transfer is completed or cancelled; `partial` when at least one is completed; `pending` otherwise. |
| `on_time_rate` | decimal | derived from `partner.on_time_rate` | derived | The vendor's on-time delivery rate. |
| `incoterm_location` | text | empty | stored | The named place that completes the international commercial term. Copied onto the vendor bill. |

---

## 17. Fields added to Purchase Order Line (owned by `../purchasing/`)

| Canonical name | Type | Default | Stored or derived | Meaning |
|---|---|---|---|---|
| `stock_moves` | one_to_many to Stock Move (inverse `purchase_line`) | empty, read-only, not copied | stored | The receipt moves created from this line. |
| `orderpoint` | many_to_one to Reordering Rule | empty, not copied, indexed (index skips empty values), deletion behavior `set null` | stored | The reordering rule that generated this line. Used as a merge key and to compute the quantity in progress. |
| `move_destinations` | many_to_many to Stock Move | empty | stored | The downstream moves this line must feed (the make-to-order chain). |
| `product_description_variants` | text | empty | stored | The extra description that was appended to the line name; a merge key. |
| `propagate_cancel` | boolean | true | stored | When true, deleting or cancelling this line cancels the downstream moves; when false the downstream moves fall back to taking from stock. |
| `forecasted_issue` | boolean | derived | derived | True when the forecast of the product in the receiving warehouse at the line's planned date is negative (counting this line's quantity when the order is still a draft). Shown as a warning icon. |
| `is_storable` | boolean | derived from `product.is_storable` | derived | |
| `location_final` | many_to_one to Location | empty | stored | The ultimate destination of the goods, taken from the procurement request. |
| `quantity_received_method` gains the value `stock_moves` ("Stock Moves") | selection | | stored | For a goods product the received quantity is computed from the stock moves instead of being typed. Removing the Purchase Inventory capability package converts affected lines to the manual method, copying the current received quantity into the manual field. |

---

## 18. Fields added to Sales Order and Sales Order Line (owned by `../sales/`)

| Entity | Canonical name | Type | Meaning |
|---|---|---|---|
| Sales Order | `dropship_transfer_count` | integer, derived | The number of transfers of the order that are drop shipments. Computing it also subtracts that number from the order's delivery count. |
| Sales Order Line | `is_make_to_order` | boolean, derived | Extended by drop shipping: the line is additionally flagged as make to order when any rule of the line's routes (or, when the line has no route, of the product's and category's routes) has an operation type whose default source location is a vendor location and whose default destination location is a customer location. |

Behavior added by drop shipping to Sales Order Line:

- The delivered quantity of a line whose purchase order lines are drop shipped is the sum of the ordered quantities of those purchase order lines (excluding cancelled ones), converted into the line unit with half-up rounding, instead of the quantity taken from stock moves. This only applies when the purchase order line product equals the sales order line product, which excludes kits with drop-shipped components.
- A line whose purchase order line count is greater than zero may not have its product changed by a user who belongs to the purchase user group.
- When a service product is subcontracted and a drop shipping operation type exists for the company, the purchase order created for the service is given that operation type and the customer's shipping address as destination address.

---

## 19. Fields added to Product Variant, Product Template and Product Category (owned by `../products-and-catalog/`)

| Entity | Canonical name | Type | Meaning |
|---|---|---|---|
| Product Variant | `purchase_order_lines` | one_to_many to Purchase Order Line | Every purchase order line for this variant. |
| Product Variant | `monthly_demand` | decimal, derived | The outgoing demand over a configurable past window, used by the purchase suggestion feature. Formula in `calculations.md`, section "Purchase suggestion quantities". |
| Product Variant | `suggested_quantity` | integer, derived | The quantity the purchase catalogue suggests for this product. |
| Product Variant | `suggest_estimated_price` | decimal, derived | The estimated price of `suggested_quantity` at the vendor's price. |
| Product Category | `routes` | many_to_many to Route | The routes selected on the category. Owned by Inventory Operations; used here as a rule-selection source. |
| Product Category | `parent_routes` | many_to_many to Route, derived | The routes of every ancestor category, excluding those already on this category. |
| Product Category | `total_routes` | many_to_many to Route, derived, searchable | `routes` union `parent_routes`. This is the set used by rule selection. |

Behavior added to Product Variant by this domain:

- "Total routes" (an extension point used by rule selection and by the reordering rule's rule chain) returns, in addition to the base empty set, the routes of every rule with action `buy` when the product has at least one Vendor Price.
- The forecast quantities of a product are increased by the quantity in progress on draft, sent and to-approve purchase order lines (see `calculations.md`, section "Quantity in progress").
- Product Template gains an on-change behavior: selecting or deselecting the Buy route on a product template triggers the "purchase method" consistency behavior owned by `../purchasing/`.

---

## 20. Fields added to Vendor Price (owned by `../pricing-and-pricelists/`)

| Canonical name | Type | Stored or derived | Meaning |
|---|---|---|---|
| `last_purchase_date` | date | derived | The date of the most recent confirmed purchase order for a variant of this vendor price's product template placed with this vendor price's contact. |
| `show_set_vendor_price_button` | boolean | derived | True when the wizard context carries a reordering rule identifier and this vendor price is not already the rule's `vendor_price`. |

Behavior added: the operation `action_set_vendor_price` writes this vendor price on the reordering rule named by the context, which also sets the rule's preferred route to a Buy route when the rule had none, and closes the wizard.

The field `lead_time_days` of Vendor Price is owned by `../pricing-and-pricelists/` and is read by this domain as the vendor lead time.

---

## 21. Fields added to Company (owned by the Contacts and Organizations domain)

| Canonical name | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `replenishment_horizon_days` | decimal | yes | 365 | The replenishment horizon, in calendar days. Reordering rules look this many days further ahead than the pure lead time when reading the forecast. |
| `days_to_purchase` | decimal | no | 0 | The number of days the vendor needs to receive and confirm a request for quotation. Added to the lead time of any `buy` rule. |
| `sales_security_lead_days` | decimal | yes | 0.0 | Labelled "Sales Safety Days". The number of days by which the scheduled date of a procurement coming from a sales order line is moved earlier relative to the promised delivery date. |
| `dropship_subcontractor_operation_type` | many_to_one to Operation Type | no | empty | The operation type used to drop ship components straight to a subcontractor. Added by the Dropship and Subcontracting Management capability package. |

---

## 22. Fields added to Contact (owned by the Contacts and Organizations domain)

| Canonical name | Type | Required | Default | Meaning |
|---|---|---|---|---|
| `purchase_lines` | one_to_many to Purchase Order Line | no | empty | Every purchase order line placed with this contact. |
| `on_time_rate` | decimal | no | derived | The share of products received on time over the past window. Formula in `calculations.md`, section "On-time delivery rate". |
| `group_request_for_quotation` | selection | yes | `default` | How replenishment needs for this vendor are grouped into requests for quotation. Closed list: `default` = "On Order", `day` = "Daily", `week` = "Weekly", `all` = "Always". |
| `grouping_weekday` | selection | yes | `default` | The target weekday when `group_request_for_quotation` is `week`. Closed list: `default` = "Expected Date", `1` = Monday, `2` = Tuesday, `3` = Wednesday, `4` = Thursday, `5` = Friday, `6` = Saturday, `7` = Sunday. |
| `suggest_based_on` | text | no | `30_days` | The historic window used by the purchase suggestion feature. |
| `suggest_days` | integer | no | 7 | The number of days of demand the purchase suggestion covers. |
| `suggest_percent` | integer | no | 100 | A percentage applied to the suggested quantity. |

---

## 23. Fields added to Reference between stock documents (owned by `../inventory-operations/`)

| Canonical name | Type | Meaning |
|---|---|---|
| `purchases` | many_to_many to Purchase Order | The purchase orders created for this reference. |

---

## 24. Fields added to Lot or Serial Number (owned by `../inventory-operations/`)

| Canonical name | Type | Stored or derived | Meaning |
|---|---|---|---|
| `purchase_orders` | many_to_many to Purchase Order | derived, read-only, not stored | The purchase orders through which this lot entered: for every completed move line carrying this lot, the purchase order of the move's purchase order line, when the move's transfer had a vendor or transit source location. |
| `purchase_order_count` | integer | derived | The number of such purchase orders. |

Behavior added by drop shipping: the customer contacts of a lot are computed from the delivery transfers; for a drop shipment the customer is the shipping address of the linked sales order rather than the transfer partner. The "outgoing" test used for that computation is widened to include any move going from a vendor location to a customer location.

---

## 25. Fields added to Product Replenish Mixin and Product Replenish Wizard (owned by `../inventory-operations/`)

Product Replenish Mixin gains:

| Canonical name | Type | Stored or derived | Meaning |
|---|---|---|---|
| `vendor_price` | many_to_one to Vendor Price | stored | The vendor price to use for the replenishment. The name follows the entity's canonical display name, so that one concept carries one name across this folder; the vendor contact behind it is read through that record. |
| `show_vendor` | boolean | derived from `route` | True when the chosen route contains a rule with action `buy`. |

The allowed-route condition of the mixin is narrowed twice: the Drop Shipping capability package removes the global Dropship route from the list, and the Dropship and Subcontracting Management capability package removes it again (the two conditions are combined, the net effect being that the Dropship route is never offered in the replenish wizard).

Product Replenish Wizard gains, from the Purchase Inventory capability package:

- The defaults: when a reordering rule already exists for the chosen product and warehouse, its `route` and `vendor_price` are proposed as defaults for the wizard's `route` and `vendor_price`.
- An on-change on `route`: when the chosen route is a buy route and no vendor price is chosen yet, the first vendor price of the product template is proposed; when the chosen route is not a buy route the vendor price is cleared.
- The scheduled date computation: for a buy route with a chosen vendor price, the scheduled date is the base scheduled date plus the vendor lead time plus the company's days to purchase. The base scheduled date is the current moment plus the sum of the lead times of every rule of the chosen route.
- The run values gain `forced_vendor_price` when a vendor price is chosen.
- The allowed-route condition is widened to include the routes of `buy` rules of the company whose operation type has code `incoming`, when the product has at least one vendor price.
- The notification after launching a replenishment points at the created purchase order when one was created, otherwise at the created transfer.
- An operation that opens the Replenishment Information wizard: it finds or creates a reordering rule for the product and warehouse and opens the wizard on it, carrying this wizard's identifier so that choosing a supplying warehouse writes the route back onto this wizard instead of onto the rule.

---

## 26. Fields added to Configuration Settings (owned by `../platform-foundation/`)

| Canonical name | Type | Meaning |
|---|---|---|
| `replenishment_horizon_days` | decimal, mirrors `company.replenishment_horizon_days` | The replenishment horizon. |
| `days_to_purchase` | decimal, mirrors `company.days_to_purchase` | Days to purchase. |
| `sales_security_lead_days` | decimal, mirrors `company.sales_security_lead_days` | Sales safety days. |
| `use_security_lead_time` | boolean, backed by the stored parameter `sales.use_security_lead_time` | Turns the sales safety days on; turning it off resets the value to 0.0. |
| `multi_step_routes_enabled` | boolean, grants the "Multi-Step Routes" access group | Turns on multi-step routes; turning it on also turns on storage locations. |
| `replenish_on_order` | boolean, derived and writable | Mirrors the `active` flag of the global "Replenish on Order" route. |
| `install_drop_shipping` | boolean | Installs the Drop Shipping capability package. |
| `is_sales_capability_installed` | boolean | True when the Sales capability package is installed; controls the visibility of the sales safety days setting. |

---

## 27. Fields added to Purchase Analysis Report (owned by `../purchasing/`)

The Purchase Analysis Report is a read-only analytical view over purchase order lines. It is owned by `../purchasing/`, which defines its measures and its base groupings. This domain adds the three columns that tie a purchase to the warehouse that received it and to the day the goods actually arrived, and it widens the query that feeds the view.

Identifier: `purchase_analysis_report`. Kind: read-only derived view, one row per purchase order line, never written by a user, never archived, company-scoped through the purchase order.

### 27.1 Fields added

| Canonical name | Type | Label | Required | Stored or derived | Copied on duplicate | Tracked | Access | Meaning |
|---|---|---|---|---|---|---|---|---|
| `operation_type` | many_to_one to Warehouse | Warehouse | no | derived by the query, read-only | not applicable, the view cannot be duplicated | no | every group that may read the report: Purchase / User, Purchase / Administrator, Inventory / Administrator | The warehouse of the operation type of the purchase order. Empty for an order whose operation type has no warehouse, which is the case for every drop shipping order. Although the column is named after the operation type it carries the warehouse, because that is the level at which buyers group their analysis. |
| `effective_date` | datetime | Effective Date | no | derived by the query, read-only | not applicable | no | the same groups | The `effective_date` of the purchase order, that is the earliest completion date among the order's completed transfers whose destination location is not a vendor location (section 16). Empty while nothing has been received. |
| `days_to_arrival` | decimal, two decimal places | Effective Days To Arrival | no | derived by the query, read-only, aggregated as an average | not applicable | no | the same groups | The number of days between the order date and the day the goods arrived. Formula and worked example in `calculations.md`, section 30. |

### 27.2 How the query is widened

1. The selected columns gain the three above.
2. The source of the query gains two joins: the operation type of the purchase order, which yields its warehouse; and a sub-query that gives, per purchase order, the earliest completion date among the completed transfers that carry a move of one of the order's lines and whose destination location is not a vendor location. A transfer with no completion date is ignored by that sub-query.
3. The grouping gains the warehouse, the effective date and the sub-query's earliest completion date, so that two lines of the same order that arrived on different days stay on separate rows.

### 27.3 Reading rules

- `days_to_arrival` is aggregated as a plain average over the rows in the group, not weighted by quantity or by amount.
- A line whose order has been received counts the arrival day of the whole order, not of that line, because the sub-query groups by order.
- A returned receipt does not move the effective date, because the sub-query ignores transfers whose destination location is a vendor location.

---

## 28. Return Wizard and Return Wizard Line (owned by `../inventory-operations/`)

The Return Wizard turns a completed transfer into a new transfer in the opposite direction. It is owned by `../inventory-operations/`, which defines its fields, its line list and the three operations "Return", "Return all" and "Exchange". This domain extends it so that a return of a receipt is recognised as a return to the vendor: the returned move is linked back to the purchase order line that brought the goods in, the vendor becomes the counterparty of the return, and the received quantity of the purchase order line is reduced.

Identifiers: `return_wizard` and `return_wizard_line`. Kind: transient.

### 28.1 The fields this domain reads

| Entity | Canonical name | Type | Read for |
|---|---|---|---|
| Return Wizard | `transfer` | many_to_one to Transfer | The completed transfer being returned. Its destination location becomes the source location of the return, which is what decides whether the return is a return to the vendor. |
| Return Wizard | `product_return_moves` | one_to_many to Return Wizard Line | One line per returnable move of the transfer. |
| Return Wizard | `company` | many_to_one to Company, mirroring the transfer's | The company the created transfer belongs to. |
| Return Wizard Line | `stock_move` | many_to_one to Stock Move | The completed move being returned. |
| Return Wizard Line | `quantity` | decimal (Product Unit) | How much to return. Zero by default, so a user must type a quantity unless the "Return all" operation is used. |
| Return Wizard Line | `unit_of_measure` | many_to_one to Unit of Measure | The unit of the returned move, falling back to the product's reference unit. |
| Return Wizard Line | `to_refund` | boolean, true by default | Labelled "Update Quantities on Purchase Order". When true the returned quantity is subtracted from the received quantity of the purchase order line; when false the received quantity is left alone and the goods are simply moved back. |

The source location of the return is the destination location of the returned transfer, and the destination location of the return is the default destination location of the return operation type when that operation type receives goods, and the source location of the returned transfer otherwise. For a receipt from a vendor, that source location is the vendor location, which is what makes the test below true.

### 28.2 The fields this domain writes on the created return move

| Canonical name | Value written |
|---|---|
| `purchase_line` | The purchase order line found by walking the move chain upwards from the returned move (algorithm in `calculations.md`, section 31). Written only when the source location of the return has usage `supplier`. |
| `partner` | The counterparty of the transfer that carries the purchase order line found by the same walk. Written under the same condition. |

### 28.3 Consequences on the purchase order line

The received quantity of a purchase order line is recomputed from its moves whenever one of them changes. Each non-cancelled move of the line whose destination location usage is not `inventory` and whose product is the line's product is classified as follows.

| Case | Classification |
|---|---|
| The move is a purchase return (see below) **and** (its `to_refund` is true **or** it has no originating returned move) | Counted as outgoing: it subtracts from the received quantity. |
| Otherwise, the move's destination location usage is not `supplier`, **and** (it has no originating returned move **or** its `to_refund` is true) | Counted as incoming: it adds to the received quantity. |
| Every other move | Not counted at all. |

A move is a **purchase return** when its destination location usage is `supplier`, or when it has an originating returned move and either its destination location is the shared inter-company transit location or the originating returned move's source location usage is `supplier`.

The quantity taken from each counted move is its completed quantity when the move is completed and its demand quantity otherwise, converted into the purchase order line unit with half-up rounding.

### 28.4 Worked example

A purchase order line orders 10 units. The receipt is validated for 10, so the received quantity is 10. The buyer returns 2 units to the vendor with "Update Quantities on Purchase Order" ticked: the return move is a purchase return with `to_refund` true, it counts as outgoing, and the received quantity becomes 10 − 2 = 8. The 2 units are then received again from the vendor with the option left unticked on that second return: the new incoming move has an originating returned move and `to_refund` false, so it is not counted, and the received quantity stays 8. Had the option been ticked on the second return as well, the new move would have counted as incoming and the received quantity would have returned to 10.

---

## 29. Cross-domain relation summary

| This domain reads or writes | Owner |
|---|---|
| Location, Warehouse, Operation Type, Stock Move, Transfer, Stock Quantity Record, Reference between stock documents, Product Replenish Wizard | `../inventory-operations/` |
| Purchase Order, Purchase Order Line, purchase analysis report | `../purchasing/` |
| Vendor Price and its lead time, price and minimum quantity | `../pricing-and-pricelists/` |
| Sales Order, Sales Order Line, customer lead time | `../sales/` |
| Manufacturing Order, Bill of Materials, manufacturing lead time, days to supply components, subcontracting locations | `../manufacturing/` |
| Product Variant, Product Template, Product Category, routes on products and categories | `../products-and-catalog/` |
| Unit of Measure conversion and rounding | `../units-of-measure-and-packaging/` |
| Company, Contact | the Contacts and Organizations domain |
| Activity, chatter message | `../messaging-and-activities/` |
| Scheduled Action, sequence, access groups, stored parameters | `../platform-foundation/` |
