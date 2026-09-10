# Split Production Detail (`mrp.production.split.line`)

**Transport name:** `mrp.production.split.line`  
**Storage name:** `mrp_production_split_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Split Production Detail

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mrp_production_split_id` | Split Production | many to one | `mrp.production.split` | required; on delete of the target: cascade |
| `quantity` | Quantity To Produce | float |  | required; precision `Product Unit` |
| `user_id` | Responsible | many to one | `res.users` | restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('mrp.group_mrp_user').id)]` |
| `date` | Schedule Date | date and time |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.split.line.json`.
