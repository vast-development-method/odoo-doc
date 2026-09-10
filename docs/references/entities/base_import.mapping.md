# Base Import Mapping (`base_import.mapping`)

**Transport name:** `base_import.mapping`  
**Storage name:** `base_import_mapping`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base_import`

Description: Base Import Mapping

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model` | Resource Model | single line text |  | indexed |
| `column_name` | Column Name | single line text |  |  |
| `field_name` | Field Name | single line text |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `base_import` |

Machine-readable definition: `../../../schemas/data/entities/base_import.mapping.json`.
