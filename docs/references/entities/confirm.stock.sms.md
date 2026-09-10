# Confirm Stock text message (`confirm.stock.sms`)

**Transport name:** `confirm.stock.sms`  
**Storage name:** `confirm_stock_sms`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock_sms`

Description: Confirm Stock SMS

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pick_ids` | Pick | many to many | `stock.picking` | association table `stock_picking_sms_rel` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `send_sms` | operation | self | `stock_sms` |  |  |
| `dont_send_sms` | operation | self | `stock_sms` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock_sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock_sms.view_confirm_stock_sms` | form |  |  | `Confirm`, `Disable SMS`, `Cancel` |  | `stock_sms` |

Machine-readable definition: `../../../schemas/data/entities/confirm.stock.sms.json`; views: `../../../schemas/interfaces/views/confirm.stock.sms.json`.
