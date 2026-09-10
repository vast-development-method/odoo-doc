# Wave Transfer Lines (`stock.add.to.wave`)

**Transport name:** `stock.add.to.wave`  
**Storage name:** `stock_add_to_wave`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock_picking_batch`

Description: Wave Transfer Lines

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wave_id` | Wave Transfer | many to one | `stock.picking.batch` | restricted by domain `[('is_wave', '=', True), ('state', 'in', ('draft', 'in_progress'))]` |
| `picking_ids` | Picking | many to many | `stock.picking` |  |
| `line_ids` | Line | many to many | `stock.move.line` |  |
| `mode` | Mode | selection |  | default `existing` |
| `user_id` | Responsible | many to one | `res.users` |  |

## Selection values

### `mode` (Mode)

| Value | Label |
|---|---|
| `existing` | an existing wave transfer |
| `new` | a new wave transfer |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock_picking_batch` | model |  |
| `attach_pickings` | operation | self | `stock_picking_batch` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | The selected transfers should belong to the same operation type | `stock_picking_batch` |
| `attach_pickings` | UserError | Cannot create wave transfers | `stock_picking_batch` |
| `attach_pickings` | UserError | The selected operations should belong to a unique company. | `stock_picking_batch` |
| `attach_pickings` | UserError | The selected transfers should belong to a unique company. | `stock_picking_batch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock_picking_batch` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock_picking_batch.stock_add_to_wave_form` | form |  | `mode`, `wave_id`, `user_id` | `Confirm`, `Cancel` |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_picking_batch.stock_add_to_wave_action_stock_picking` | Add to wave | form |  |  | new | `stock_picking_batch` |

Machine-readable definition: `../../../schemas/data/entities/stock.add.to.wave.json`; views: `../../../schemas/interfaces/views/stock.add.to.wave.json`.
