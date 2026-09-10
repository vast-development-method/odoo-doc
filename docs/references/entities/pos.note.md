# PoS Note (`pos.note`)

**Transport name:** `pos.note`  
**Storage name:** `pos_note`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`

Description: PoS Note

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `sequence` | Sequence | integer |  | default `1` |
| `color` | Color | integer |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_unique` | Constraint | `unique (name)` | A note with this name already exists | `point_of_sale` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_note_tree` | list |  | `sequence`, `name`, `color` |  |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_note_model` | Note Models | list |  |  |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.note.json`; views: `../../../schemas/interfaces/views/pos.note.json`.
