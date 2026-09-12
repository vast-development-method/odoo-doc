# Model Access (`ir.model.access`)

**Transport name:** `ir.model.access`  
**Storage name:** `ir_model_access`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Model Access

## Identity and behavior

- Default ordering: `model_id,group_id,name,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; indexed |
| `active` | Active | boolean |  | default `True`; Help: If you uncheck the active field, it will disable the ACL without deleting it (if you delete a native ACL, it will be re-created when you reload the module). |
| `model_id` | Model | many to one | `ir.model` | required; indexed; on delete of the target: cascade |
| `group_id` | Group | many to one | `res.groups` | indexed; on delete of the target: restrict |
| `perm_read` | Read Access | boolean |  |  |
| `perm_write` | Write Access | boolean |  |  |
| `perm_create` | Create Access | boolean |  |  |
| `perm_unlink` | Delete Access | boolean |  |  |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `group_names_with_access` | operation | self, model_name, access_mode | `base` | model | Return the names of visible groups which have been granted  `access_mode` on the model `model_name`.  :rtype: list |
| `_get_access_groups` | preparation rule | self, model_name, access_mode | `base` | model | Return the group expression object that represents the users who have `access_mode` to the model `model_name`. |
| `_get_allowed_models` | preparation rule | self, mode | `base` |  |  |
| `check` | operation | self, model, mode, raise_exception | `base` | model |  |
| `_make_access_error` | internal rule | self, model, mode | `base` |  | Return the exception corresponding to an access error. |
| `call_cache_clearing_methods` | operation | self | `base` | model |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_access_view_tree` | list |  | `name`, `model_id`, `group_id`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink` |  |  | `base` |
| `base.ir_access_view_tree_edition` | list |  | `name`, `model_id`, `group_id`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink` |  |  | `base` |
| `base.ir_access_view_form` | form |  | `name`, `model_id`, `group_id`, `active`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink` |  |  | `base` |
| `base.ir_access_view_search` | search |  | `name`, `model_id`, `group_id` |  | `Global`, `Full Access`, `Read Access`, `Write Access`, `Archived`, `Group`, `Model` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_access_act` | Access Rights |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.access.json`; views: `../../../schemas/interfaces/views/ir.model.access.json`.
