# Exports (`ir.exports`)

**Transport name:** `ir.exports`  
**Storage name:** `ir_exports`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Exports

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Export Name | single line text |  |  |
| `resource` | Resource | single line text |  | indexed |
| `export_fields` | Export identifier | one to many | `ir.exports.line` | inverse field `export_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_allow_export` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.exports.json`.
