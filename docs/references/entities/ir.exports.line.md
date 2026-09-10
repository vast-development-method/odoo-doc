# Exports Line (`ir.exports.line`)

**Transport name:** `ir.exports.line`  
**Storage name:** `ir_exports_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Exports Line

## Identity and behavior

- Default ordering: `id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Field Name | single line text |  |  |
| `export_id` | Export | many to one | `ir.exports` | indexed; on delete of the target: cascade |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.exports.line.json`.
