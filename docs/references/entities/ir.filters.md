# Filters (`ir.filters`)

**Transport name:** `ir.filters`  
**Storage name:** `ir_filters`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Filters

## Identity and behavior

- Default ordering: `model_id, name, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Filter Name | single line text |  | required |
| `user_ids` | Users | many to many | `res.users` | on delete of the target: cascade; Help: The users the filter is shared with. If empty, the filter is shared with all users. |
| `domain` | Domain | multi line text |  | required; default `[]` |
| `context` | Context | multi line text |  | required; default `{}` |
| `sort` | Sort | single line text |  | required; default `[]` |
| `model_id` | Model | selection |  | required; values provided by rule `_list_all_models` |
| `is_default` | Default Filter | boolean |  |  |
| `action_id` | Action | many to one | `ir.actions.actions` | on delete of the target: cascade; Help: The menu action this filter applies to. When left empty the filter applies to all menus for this model. |
| `embedded_action_id` | Embedded Action | many to one | `ir.embedded.actions` | indexed (btree_not_null); on delete of the target: cascade; Help: The embedded action this filter is applied to |
| `embedded_parent_res_id` | Embedded Parent Resource | integer |  | Help: id of the record the filter should be applied to. Only used in combination with embedded actions |
| `active` | Active | boolean |  | default `True` |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_get_filters_index` | Index | `(model_id, action_id, embedded_action_id, embedded_parent_res_id)` |  | `base` |
| `_check_res_id_only_when_embedded_action` | Constraint | `CHECK(NOT (embedded_parent_res_id IS NOT NULL AND embedded_action_id IS NULL))` | Constraint to ensure that the embedded_parent_res_id is only defined when a top_action_id is defined. | `base` |
| `_check_sort_json` | Constraint | `CHECK(sort IS NULL OR jsonb_typeof(sort::jsonb) = 'array')` | Invalid sort definition | `base` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_list_all_models` | internal rule | self | `base` | model |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_get_eval_domain` | preparation rule | self | `base` |  |  |
| `_get_action_domain` | preparation rule | self, action_id, embedded_action_id, embedded_parent_res_id | `base` | model | Return a domain component for matching filters that are visible in the same context (menu/view) as the given action. |
| `get_filters` | operation | self, model, action_id, embedded_action_id, embedded_parent_res_id | `base` | model | Obtain the list of filters available for the user on the given model.  :param int model: id of model to find filters for :param action_id: optional ID of action to restrict filters to this action     plus global filters. If missing only global filters are returned.     The action does not have to correspond to the model, it may only be     a contextual action. :return: list of :meth:`~osv.read`-like dicts containing the     `name`, `is_default`, `domain`, `user_ids` (m2m),     `action_id` (m2o tuple), `embedded_action_id` (m2o tuple), `embedded_parent_res_id`     and `context`  |
| `create_filter` | operation | self, vals | `base` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `group_user` | yes | yes | yes | yes | `base` |
| `group_portal` | yes | yes | yes | yes | `base` |
| `group_public` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| ir.filters.admin.all.rights | `[Command.link(ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| ir.filter: owner or global | `[Command.link(ref('base.group_user'))]` | `[('user_ids','in',[False,user.id])]` | True | True | True | True |
| ir.filter: portal/public | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('user_ids', 'in', user.ids)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_filters_view_form` | form |  | `model_id`, `name`, `user_ids`, `is_default`, `model_id`, `action_id`, `domain`, `context`, `sort` |  |  | `base` |
| `base.ir_filters_view_edit_form` | xpath | `base.ir_filters_view_form` |  |  |  | `base` |
| `base.ir_filters_view_tree` | list |  | `name`, `model_id`, `user_ids`, `is_default`, `action_id`, `domain`, `context`, `sort` |  |  | `base` |
| `base.ir_filters_view_search` | search |  | `name`, `model_id`, `user_ids` |  | `Global`, `My filters`, `Archived`, `User`, `Model` | `base` |
| `mail.ir_filters_view_form` | xpath | `base.ir_filters_view_form` |  |  |  | `mail` |
| `mail.ir_filters_view_tree` | xpath | `base.ir_filters_view_tree` |  |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.actions_ir_filters_view` | User-defined Filters |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.filters.json`; views: `../../../schemas/interfaces/views/ir.filters.json`.
