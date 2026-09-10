# Stock package type (`stock.package.type`)

**Transport name:** `stock.package.type`  
**Storage name:** `stock_package_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_delivery`

Description: Stock package type

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (19)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Package Type | single line text |  | required |
| `sequence` | Sequence | integer |  | default `1`; Help: The first in the sequence is the default one. |
| `sequence_id` | Reference Sequence | many to one | `ir.sequence` | not copied on duplication; must belong to the same company |
| `sequence_code` | Sequence Prefix | single line text |  | related through path `sequence_id.code` |
| `height` | Height | float |  | Help: Packaging Height |
| `width` | Width | float |  | Help: Packaging Width |
| `packaging_length` | Length | float |  | Help: Packaging Length |
| `base_weight` | Weight | float |  | Help: Weight of the package type |
| `max_weight` | Max Weight | float |  | Help: Maximum weight shippable in this packaging |
| `barcode` | Barcode | single line text |  | not copied on duplication |
| `weight_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_uom_name` (not stored); default computed dynamically (_get_default_weight_uom) |
| `length_uom_name` | Length unit of measure label | single line text |  | computed by rule `_compute_length_uom_name` (not stored); default computed dynamically (_get_default_length_uom) |
| `company_id` | Company | many to one | `res.company` | indexed |
| `package_use` | Package Use | selection |  | required; default `disposable`; Help: Reusable boxes are used for batch picking and emptied afterwards to be reused. In the barcode application, scanning a reusable box will add the products in this box.         Disposable boxes aren't reused, when scanning a disposable box in the barcode application, the contained products are added to the transfer. |
| `has_quants` | Has Contents | boolean |  | computed by rule `_compute_has_quants` (not stored) |
| `storage_category_capacity_ids` | Storage Category Capacity | one to many | `stock.storage.category.capacity` | inverse field `package_type_id` |
| `route_ids` | Routes | many to many | `stock.route` | restricted by domain `[('package_type_selectable', '=', True)]` |
| `shipper_package_code` | Carrier Code | single line text |  |  |
| `package_carrier_type` | Carrier | selection |  | default `none` |

## Selection values

### `package_use` (Package Use)

| Value | Label |
|---|---|
| `disposable` | Disposable Box |
| `reusable` | Reusable Box (totes) |

### `package_carrier_type` (Carrier)

| Value | Label |
|---|---|
| `none` | No carrier integration |

## Database constraints and indexes (5)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_barcode_uniq` | Constraint | `unique(barcode)` | A barcode can only be assigned to one package type! | `stock` |
| `_positive_height` | Constraint | `CHECK(height>=0.0)` | Height must be positive | `stock` |
| `_positive_width` | Constraint | `CHECK(width>=0.0)` | Width must be positive | `stock` |
| `_positive_length` | Constraint | `CHECK(packaging_length>=0.0)` | Length must be positive | `stock` |
| `_positive_max_weight` | Constraint | `CHECK(max_weight>=0.0)` | Max Weight must be positive | `stock` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_length_uom` | preparation rule | self | `stock` |  |  |
| `_get_default_weight_uom` | preparation rule | self | `stock` |  |  |
| `_compute_display_name` | computation | self | `stock` | depends: `name`, `packaging_length`, `width`, `height`; depends_context: `formatted_display_name` |  |
| `_compute_has_quants` | computation | self | `stock` |  |  |
| `_compute_length_uom_name` | computation | self | `stock_delivery`, `stock` | depends: `package_carrier_type` |  |
| `_compute_weight_uom_name` | computation | self | `stock` |  |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `_get_next_name_by_sequence` | preparation rule | self | `stock` |  |  |
| `_onchange_carrier_type` | on change | self | `stock_delivery` | onchange: `package_carrier_type` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_stock` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_package_type_form` | form |  | `name`, `barcode`, `sequence_id`, `sequence_code`, `route_ids`, `company_id`, `packaging_length`, `width`, `height`, `length_uom_name`, `base_weight`, `weight_uom_name`, `max_weight`, `weight_uom_name`, `storage_category_capacity_ids`, `storage_category_id`, `quantity` |  |  | `stock` |
| `stock.stock_package_type_tree` | list |  | `sequence`, `name`, `height`, `width`, `packaging_length`, `max_weight`, `has_quants`, `barcode` |  |  | `stock` |
| `stock_delivery.stock_package_type_form_delivery` | xpath | `stock.stock_package_type_form` | `package_carrier_type`, `shipper_package_code` |  |  | `stock_delivery` |
| `stock_delivery.stock_package_type_tree_delivery` | xpath | `stock.stock_package_type_tree` | `package_carrier_type` |  |  | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_package_type_view` | Package Types |  |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.package.type.json`; views: `../../../schemas/interfaces/views/stock.package.type.json`.
