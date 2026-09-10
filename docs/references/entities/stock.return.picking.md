# Return Picking (`stock.return.picking`)

**Transport name:** `stock.return.picking`  
**Storage name:** `stock_return_picking`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`, `sale_stock`, `stock_delivery`, `purchase_stock`, `mrp_subcontracting`

Description: Return Picking

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `picking_id` | Picking | many to one | `stock.picking` |  |
| `picking_type_code` | Picking Type Code | selection |  | read only; related through path `picking_id.picking_type_code` |
| `product_return_moves` | Moves | one to many | `stock.return.picking.line` | computed by rule `_compute_moves_locations` and stored; inverse field `wizard_id`; precomputed before insertion |
| `company_id` | Company | many to one |  | related through path `picking_id.company_id` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `_compute_moves_locations` | computation | self | `stock` | depends: `picking_id` |  |
| `_prepare_stock_return_picking_line_vals_from_move` | preparation rule | self, stock_move | `stock` | model |  |
| `_prepare_picking_default_values` | preparation rule | self | `mrp_subcontracting`, `stock` |  |  |
| `_prepare_picking_default_values_based_on` | preparation rule | self, picking | `sale_stock`, `stock` |  |  |
| `_create_return` | internal rule | self | `purchase_stock`, `stock_delivery`, `stock` |  |  |
| `_create_exchange` | internal rule | self, return_picking | `stock` |  |  |
| `action_create_returns` | user action | self | `stock` |  |  |
| `action_create_returns_all` | user action | self | `stock` |  | Create a return matching the total delivered quantity and open it. |
| `action_create_exchanges` | user action | self | `stock` |  | Create a return for the active picking, then create a return of the return for the exchange picking and open it. |
| `_get_proc_values` | preparation rule | self, line | `sale_stock`, `stock` |  |  |
| `_reset_carrier_id` | internal rule | self, picking | `stock_delivery` |  | Prevent copy of the carrier and carrier price when generating return picking (we have no integration of returns for now). |
| `_prepare_move_default_values` | preparation rule | self, return_line, new_picking | `purchase_stock` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | You may only return one picking at a time. | `stock` |
| `_compute_moves_locations` | UserError | You may only return Done pickings. | `stock` |
| `_compute_moves_locations` | UserError | No products to return (only lines in Done state and not fully returned yet can be returned). | `stock` |
| `_create_return` | UserError | Please specify at least one non-zero quantity. | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_stock_return_picking_form` | form |  | `picking_id`, `company_id`, `product_return_moves`, `move_quantity`, `product_id`, `quantity`, `uom_id`, `move_id` | `Return`, `Return All`, `Return for Exchange`, `Discard` |  | `stock` |
| `stock_account.view_stock_return_picking_form_inherit_stock_account` | xpath | `stock.view_stock_return_picking_form` | `to_refund` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.act_stock_return_picking` | Return | form |  |  | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.return.picking.json`; views: `../../../schemas/interfaces/views/stock.return.picking.json`.
