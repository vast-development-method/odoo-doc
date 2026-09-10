# User Settings for Embedded Actions (`res.users.settings.embedded.action`)

**Transport name:** `res.users.settings.embedded.action`  
**Storage name:** `res_users_settings_embedded_action`  
**Kind:** persistent entity (one table)  
**Defined by package:** `web`

Description: User Settings for Embedded Actions

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_setting_id` | User Setting | many to one | `res.users.settings` | required; indexed (btree_not_null); on delete of the target: cascade |
| `action_id` | Action | many to one | `ir.actions.act_window` | required; on delete of the target: cascade |
| `res_model` | Resource Model | single line text |  | required |
| `res_id` | Resource | integer |  |  |
| `embedded_actions_order` | List order of embedded action ids | single line text |  |  |
| `embedded_actions_visibility` | List visibility of embedded actions ids | single line text |  |  |
| `embedded_visibility` | Is top bar visible | boolean |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_res_user_settings_embedded_action_unique` | Constraint | `UNIQUE (user_setting_id, action_id, res_id)` | The user should have one unique embedded action setting per user setting, action and record id. | `web` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_embedded_actions_order` | validation | self | `web` | constrains: `embedded_actions_order` |  |
| `_check_embedded_actions_visibility` | validation | self | `web` | constrains: `embedded_actions_visibility` |  |
| `_check_embedded_actions_field_format` | validation | self, field_name | `web` |  |  |
| `_embedded_action_settings_format` | internal rule | self | `web` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_embedded_actions_field_format` | ValidationError | The ids in %(field_name)s must not be duplicated: “%(action_ids)s” | `web` |
| `_check_embedded_actions_field_format` | ValidationError | The ids in %(field_name)s must only be integers or "false": “%(action_ids)s” | `web` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `web` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| res.users.settings.embedded.action: access their own entries | `[Command.link(ref('base.group_user'))]` | `[('user_setting_id.user_id', '=', user.id)]` | True | True | True | True |
| Administrators can access all User Settings embedded actions | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/res.users.settings.embedded.action.json`.
