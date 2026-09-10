# Reference between stock documents (`stock.reference`)

**Transport name:** `stock.reference`  
**Storage name:** `stock_reference`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`, `point_of_sale`, `purchase_stock`, `mrp`

Description: Reference between stock documents

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | required; read only |
| `move_ids` | Stock Moves | many to many | `stock.move` | association table `stock_reference_move_rel` |
| `picking_ids` | Transfers | many to many | `stock.picking` | read only; computed by rule `_compute_picking_ids` (not stored) |
| `sale_ids` | Sales | many to many | `sale.order` | association table `stock_reference_sale_rel` |
| `pos_order_ids` | PoS Orders | many to many | `pos.order` | association table `stock_reference_pos_order_rel` |
| `purchase_ids` | Purchases | many to many | `purchase.order` | not copied on duplication; association table `stock_reference_purchase_rel` |
| `production_ids` | Productions | many to many | `mrp.production` | association table `stock_reference_production_rel` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_picking_ids` | computation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `stock` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.stock_reference_pos_view_form` | xpath | `stock.stock_reference_form_view` | `pos_order_ids` |  |  | `point_of_sale` |
| `purchase_stock.stock_reference_purchase_view_form` | xpath | `stock.stock_reference_form_view` | `purchase_ids` |  |  | `purchase_stock` |
| `sale_stock.stock_reference_sale_view_form` | xpath | `stock.stock_reference_form_view` | `sale_ids` |  |  | `sale_stock` |
| `stock.stock_reference_search_view` | search |  | `name` |  |  | `stock` |
| `stock.stock_reference_form_view` | form |  | `name`, `picking_ids` |  |  | `stock` |
| `stock.stock_reference_tree_view` | list |  | `name` |  |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_reference` | References | list,form |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.reference.json`; views: `../../../schemas/interfaces/views/stock.reference.json`.
