# Embedded Actions (`ir.embedded.actions`)

**Transport name:** `ir.embedded.actions`  
**Storage name:** `ir_embedded_actions`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Embedded Actions

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `sequence` | Sequence | integer |  |  |
| `parent_action_id` | Parent Action | many to one | `ir.actions.act_window` | required; on delete of the target: cascade |
| `parent_res_id` | Active Parent Id | integer |  |  |
| `parent_res_model` | Active Parent Model | single line text |  | required |
| `action_id` | Action | many to one | `ir.actions.actions` | on delete of the target: cascade |
| `python_method` | Python Method | single line text |  | Help: Python method returning an action |
| `user_id` | User | many to one | `res.users` | on delete of the target: cascade; Help: User specific embedded action. If empty, shared embedded action |
| `is_deletable` | Is Deletable | boolean |  | computed by rule `_compute_is_deletable` (not stored) |
| `default_view_mode` | Default View | single line text |  | Help: Default view (if none, default view of the action is taken) |
| `filter_ids` | Filter | one to many | `ir.filters` | inverse field `embedded_action_id`; Help: Default filter of the embedded action (if none, no filters) |
| `is_visible` | Embedded visibility | boolean |  | computed by rule `_compute_is_visible` (not stored); Help: Computed field to check if the record should be visible according to the domain |
| `domain` | Domain | single line text |  | default `[]`; Help: Domain applied to the active id of the parent model |
| `context` | Context | single line text |  | default `{}`; Help: Context dictionary as Python expression, empty by default (Default: {}) |
| `groups_ids` | Groups | many to many | `res.groups` | Help: Groups that can execute the embedded action. Leave empty to allow everybody. |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_only_one_action_defined` | Constraint | `CHECK(             (action_id IS NOT NULL AND python_method IS NULL)             OR (action_id IS NULL AND python_method IS NOT NULL)         )` | Constraint to ensure that either an XML action or a python_method is defined, but not both. | `base` |
| `_check_python_method_requires_name` | Constraint | `CHECK(NOT (python_method IS NOT NULL AND name IS NULL))` | Constraint to ensure that if a python_method is defined, then the name must also be defined. | `base` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `_compute_display_name` | computation | self | `base` | depends: `action_id.display_name`, `name` |  |
| `_compute_is_deletable` | computation | self | `base` |  |  |
| `_compute_is_visible` | computation | self | `base` |  |  |
| `_unlink_if_action_deletable` | internal rule | self | `base` | ondelete |  |
| `_get_readable_fields` | preparation rule | self | `base` |  | return the list of fields that are safe to read |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_if_action_deletable` | UserError | You cannot delete a default embedded action | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users can modify or delete embedded actions that they have created or that are shared | `[Command.link(ref('base.group_user'))]` | `[('user_id', 'in', [user.id, False])]` | False | True | False | True |
| Admins have all the rights on embedded actions | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | False | True | False | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.embedded_action_form` | form |  | `name`, `parent_action_id`, `action_id`, `user_id`, `filter_ids`, `sequence`, `is_deletable`, `default_view_mode`, `parent_res_model`, `domain`, `groups_ids` |  |  | `base` |
| `base.embedded_action_tree` | list |  | `name`, `parent_action_id`, `action_id`, `user_id` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_embedded_action` | Embedded Actions |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.embedded.actions.json`; views: `../../../schemas/interfaces/views/ir.embedded.actions.json`.
