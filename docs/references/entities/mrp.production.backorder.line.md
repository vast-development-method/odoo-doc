# Backorder Confirmation Line (`mrp.production.backorder.line`)

**Transport name:** `mrp.production.backorder.line`  
**Storage name:** `mrp_production_backorder_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Backorder Confirmation Line

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mrp_production_backorder_id` | manufacturing order Backorder | many to one | `mrp.production.backorder` | required; on delete of the target: cascade |
| `mrp_production_id` | Manufacturing Order | many to one | `mrp.production` | required; read only; on delete of the target: cascade |
| `to_backorder` | To Backorder | boolean |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.backorder.line.json`.
