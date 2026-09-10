# Stock warehouse replenishment option (`stock.replenishment.option`)

**Transport name:** `stock.replenishment.option`  
**Storage name:** `stock_replenishment_option`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `purchase_stock`

Description: Stock warehouse replenishment option

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `route_id` | Route | many to one | `stock.route` |  |
| `product_id` | Product | many to one | `product.product` |  |
| `replenishment_info_id` | Replenishment Info | many to one | `stock.replenishment.info` |  |
| `location_id` | Location | many to one | `stock.location` | related through path `warehouse_id.lot_stock_id` |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | related through path `route_id.supplier_wh_id` |
| `uom` | Unit of measure | single line text |  | related through path `product_id.uom_name` |
| `qty_to_order` | Qty To Order | float |  | related through path `replenishment_info_id.qty_to_order` |
| `free_qty` | Free Qty | float |  | computed by rule `_compute_free_qty` (not stored) |
| `lead_time` | Lead Time | single line text |  | computed by rule `_compute_lead_time` (not stored) |
| `warning_message` | Warning Message | single line text |  | computed by rule `_compute_warning_message` (not stored) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_free_qty` | computation | self | `stock` | depends: `product_id`, `route_id` |  |
| `_compute_lead_time` | computation | self | `stock` | depends: `replenishment_info_id` |  |
| `_compute_warning_message` | computation | self | `stock` | depends: `warehouse_id`, `free_qty`, `uom`, `qty_to_order` |  |
| `select_route` | operation | self | `purchase_stock`, `stock` |  |  |
| `order_avbl` | operation | self | `stock` |  |  |
| `order_all` | operation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.replenishment_option_tree_view` | list |  | `qty_to_order`, `warehouse_id`, `location_id`, `free_qty`, `uom`, `route_id`, `lead_time` | `Select Route` |  | `stock` |
| `stock.replenishment_option_warning_view` | form |  | `warning_message`, `free_qty`, `qty_to_order` | `order_avbl`, `order_all`, `Cancel` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.replenishment.option.json`; views: `../../../schemas/interfaces/views/stock.replenishment.option.json`.
