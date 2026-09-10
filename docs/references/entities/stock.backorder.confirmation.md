# Backorder Confirmation (`stock.backorder.confirmation`)

**Transport name:** `stock.backorder.confirmation`  
**Storage name:** `stock_backorder_confirmation`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Backorder Confirmation

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pick_ids` | Pick | many to many | `stock.picking` | association table `stock_picking_backorder_rel` |
| `show_transfers` | Show Transfers | boolean |  |  |
| `backorder_confirmation_line_ids` | Backorder Confirmation Lines | one to many | `stock.backorder.confirmation.line` | inverse field `backorder_confirmation_id` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `_check_less_quantities_than_expected` | validation | self, pickings | `stock` |  |  |
| `process` | operation | self | `stock` |  |  |
| `process_cancel_backorder` | operation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_backorder_confirmation` | form |  | `pick_ids`, `show_transfers`, `backorder_confirmation_line_ids`, `picking_id`, `to_backorder` | `Create Backorder`, `No Backorder`, `Discard` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.backorder.confirmation.json`; views: `../../../schemas/interfaces/views/stock.backorder.confirmation.json`.
