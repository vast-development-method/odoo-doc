# Add tag for the workcenter (`mrp.workcenter.tag`)

**Transport name:** `mrp.workcenter.tag`  
**Storage name:** `mrp_workcenter_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Add tag for the workcenter

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_tag_name_unique` | Constraint | `unique(name)` | The tag name must be unique. | `mrp` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.tag.json`.
