# Backorder Confirmation Line (`stock.backorder.confirmation.line`)

**Transport name:** `stock.backorder.confirmation.line`  
**Storage name:** `stock_backorder_confirmation_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Backorder Confirmation Line

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `backorder_confirmation_id` | Immediate Transfer | many to one | `stock.backorder.confirmation` |  |
| `picking_id` | Transfer | many to one | `stock.picking` |  |
| `to_backorder` | To Backorder | boolean |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.backorder.confirmation.line.json`.
