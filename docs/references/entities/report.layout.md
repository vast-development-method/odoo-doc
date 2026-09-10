# Report Layout (`report.layout`)

**Transport name:** `report.layout`  
**Storage name:** `report_layout`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Report Layout

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `view_id` | Document Template | many to one | `ir.ui.view` | required |
| `image` | Preview image src | single line text |  |  |
| `pdf` | Preview pdf src | single line text |  |  |
| `sequence` | Sequence | integer |  | default `50` |
| `name` | Name | single line text |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/report.layout.json`.
