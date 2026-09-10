# Batch Transfer Lines (`stock.picking.to.batch`)

**Transport name:** `stock.picking.to.batch`  
**Storage name:** `stock_picking_to_batch`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock_picking_batch`

Description: Batch Transfer Lines

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `batch_id` | Batch Transfer | many to one | `stock.picking.batch` | restricted by domain `[('is_wave', '=', False), ('state', 'in', ('draft', 'in_progress'))]` |
| `mode` | Mode | selection |  | default `new` |
| `user_id` | Responsible | many to one | `res.users` |  |
| `is_create_draft` | Draft | boolean |  | Help: When checked, create the batch in draft status |
| `description` | Description | single line text |  |  |

## Selection values

### `mode` (Mode)

| Value | Label |
|---|---|
| `existing` | an existing batch transfer |
| `new` | a new batch transfer |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `attach_pickings` | operation | self | `stock_picking_batch` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `attach_pickings` | UserError | The selected pickings should belong to an unique company. | `stock_picking_batch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock_picking_batch` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock_picking_batch.stock_picking_to_batch_form` | form |  | `mode`, `description`, `batch_id`, `user_id`, `is_create_draft` | `Confirm`, `Cancel` |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_picking_batch.stock_picking_to_batch_action_stock_picking` | Add to batch | form |  |  | new | `stock_picking_batch` |

Machine-readable definition: `../../../schemas/data/entities/stock.picking.to.batch.json`; views: `../../../schemas/interfaces/views/stock.picking.to.batch.json`.
