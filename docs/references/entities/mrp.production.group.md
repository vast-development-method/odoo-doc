# Production Group (`mrp.production.group`)

**Transport name:** `mrp.production.group`  
**Storage name:** `mrp_production_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Production Group

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; indexed (btree) |
| `production_ids` | Productions | one to many | `mrp.production` | inverse field `production_group_id` |
| `child_ids` | Child Manufacturing Orders | many to many | `mrp.production.group` | association table `mrp_production_group_rel` |
| `parent_ids` | Parent Manufacturing Orders | many to many | `mrp.production.group` | association table `mrp_production_group_rel` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.group.json`.
