# Return Picking Line (`stock.return.picking.line`)

**Transport name:** `stock.return.picking.line`  
**Storage name:** `stock_return_picking_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `mrp_subcontracting`

Description: Return Picking Line

## Identity and behavior

- Display name field: `product_id`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required |
| `move_quantity` | Move Quantity | float |  | related through path `move_id.quantity` |
| `quantity` | Quantity | float |  | required; default `1`; precision `Product Unit` |
| `uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_uom_id` (not stored) |
| `wizard_id` | Wizard | many to one | `stock.return.picking` |  |
| `move_id` | Move | many to one | `stock.move` |  |
| `to_refund` | Update quantities on sales order/purchase order | boolean |  | default `True`; Help: Trigger a decrease of the delivered/received quantity in the associated Sale Order/Purchase Order |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_uom_id` | computation | self | `stock` | depends: `move_id.product_uom`, `product_id.uom_id` | Compute the UoM based on the move's product UoM or the product's default UoM. |
| `_prepare_move_default_values` | preparation rule | self, new_picking | `mrp_subcontracting`, `sale_stock`, `stock_account`, `stock` |  |  |
| `_process_line` | background operation | self, new_picking | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.return.picking.line.json`.
