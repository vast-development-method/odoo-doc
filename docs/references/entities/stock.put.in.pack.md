# Put In Pack Wizard (`stock.put.in.pack`)

**Transport name:** `stock.put.in.pack`  
**Storage name:** `stock_put_in_pack`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_delivery`

Description: Put In Pack Wizard

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `location_dest_id` | Destination | many to one | `stock.location` |  |
| `move_line_ids` | Move lines | many to many | `stock.move.line` |  |
| `package_ids` | Packages | many to many | `stock.package` |  |
| `package_type_id` | Package Type | many to one | `stock.package.type` |  |
| `package_type_sequence_id` | Package Type Sequence | many to one |  | related through path `package_type_id.sequence_id` |
| `result_package_id` | Package | many to one | `stock.package` |  |
| `origin_package_ids` | Origin Package | many to many | `stock.package` | computed by rule `_compute_origin_package_ids` (not stored) |
| `shipping_weight` | Shipping Weight | float |  | computed by rule `_compute_shipping_weight` and stored |
| `weight_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_uom_name` (not stored) |
| `package_carrier_type` | Carrier Type | single line text |  |  |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_origin_package_ids` | computation | self | `stock` |  |  |
| `_onchange_package_type_id` | on change | self | `stock` | onchange: `package_type_id` |  |
| `action_put_in_pack` | user action | self | `stock` |  |  |
| `_get_put_in_pack_context` | preparation rule | self | `stock_delivery`, `stock` |  |  |
| `_compute_weight_uom_name` | computation | self | `stock_delivery` |  |  |
| `_compute_shipping_weight` | computation | self | `stock_delivery` | depends: `package_type_id`, `result_package_id` |  |
| `_onchange_package_type_weight` | on change | self | `stock_delivery` | onchange: `package_type_id`, `result_package_id`, `shipping_weight` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_put_in_pack_form` | form |  | `package_type_id`, `result_package_id` | `Put in Pack`, `Discard` |  | `stock` |
| `stock_delivery.stock_put_in_pack_form` | field | `stock.stock_put_in_pack_form` | `result_package_id`, `move_line_ids`, `package_ids`, `package_carrier_type`, `shipping_weight`, `weight_uom_name` |  |  | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_put_in_pack_wizard` | Put in Pack | form |  | `{'dialog_size': 'medium'}` | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.put.in.pack.json`; views: `../../../schemas/interfaces/views/stock.put.in.pack.json`.
