# Custom View (`ir.ui.view.custom`)

**Transport name:** `ir.ui.view.custom`  
**Storage name:** `ir_ui_view_custom`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Custom View

## Identity and behavior

- Default ordering: `create_date desc, id desc`
- Display name field: `user_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `ref_id` | Original View | many to one | `ir.ui.view` | required; indexed; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | required; indexed; on delete of the target: cascade |
| `arch` | View Architecture | multi line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_id_ref_id` | Index | `(user_id, ref_id)` |  | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| ir.ui.view_custom rule | global (all users) | `[('user_id','=',user.id)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_view_custom_form` | form |  | `user_id`, `ref_id`, `arch` |  |  | `base` |
| `base.view_view_custom_tree` | list |  | `user_id`, `ref_id` |  |  | `base` |
| `base.view_view_custom_search` | search |  | `user_id`, `ref_id` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_ui_view_custom` | Customized Views |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.ui.view.custom.json`; views: `../../../schemas/interfaces/views/ir.ui.view.custom.json`.
