# Access Groups (`res.groups`)

**Transport name:** `res.groups`  
**Storage name:** `res_groups`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `bus`, `mail`, `account`, `auth_timeout`, `im_livechat`, `website_slides`

Description: Access Groups

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`
- Default ordering: `privilege_id, sequence, name, id`
- Display name field: `full_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (32)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `user_ids` | User | many to many | `res.users` | association table `res_groups_users_rel`; Help: Users explicitly in this group |
| `all_user_ids` | Users and implied users | many to many | `res.users` | computed by rule `_compute_all_user_ids` (not stored); writable through an inverse rule; searchable through a search rule |
| `all_users_count` | # Users | integer |  | computed by rule `_compute_all_users_count` (not stored); Help: Number of users having this group (implicitly or explicitly) |
| `model_access` | Access Controls | one to many | `ir.model.access` | inverse field `group_id` |
| `rule_groups` | Rules | many to many | `ir.rule` | restricted by domain `[('global', '=', False)]`; association table `rule_group_rel` |
| `menu_access` | Access Menu | many to many | `ir.ui.menu` | association table `ir_ui_menu_group_rel` |
| `view_access` | Views | many to many | `ir.ui.view` | association table `ir_ui_view_group_rel` |
| `comment` | Comment | multi line text |  | translatable |
| `full_name` | Group Name | single line text |  | computed by rule `_compute_full_name` (not stored); searchable through a search rule |
| `share` | Share Group | boolean |  | Help: Group created to set access rights for sharing data with some users. |
| `api_key_duration` | application programming interface Keys maximum duration days | float |  | Help: Determines the maximum duration of an api key created by a user belonging to this group. |
| `sequence` | Sequence | integer |  |  |
| `privilege_id` | Privilege | many to one | `res.groups.privilege` | indexed |
| `view_group_hierarchy` | Technical field for default group setting | structured document |  | computed by rule `_compute_view_group_hierarchy` (not stored) |
| `implied_ids` | Implied Groups | many to many | `res.groups` | association table `res_groups_implied_rel`; Help: Users of this group are also implicitly part of those groups |
| `all_implied_ids` | Transitively Implied Groups | many to many | `res.groups` | computed by rule `_compute_all_implied_ids` (not stored); searchable through a search rule; recursive dependency; Help: The group itself with all its implied groups. |
| `implied_by_ids` | Implying Groups | many to many | `res.groups` | association table `res_groups_implied_rel`; Help: Users in those groups are implicitly part of this group. |
| `all_implied_by_ids` | Transitively Implying Groups | many to many | `res.groups` | computed by rule `_compute_all_implied_by_ids` (not stored); searchable through a search rule; recursive dependency |
| `disjoint_ids` | Disjoint Groups | many to many | `res.groups` | computed by rule `_compute_disjoint_ids` (not stored); Help: A user may not belong to this group and one of those.  For instance, users may not be portal users and internal users. |
| `lock_timeout` | Session timeout | integer |  | Help: Time interval (in minutes) after which re-authentication is required, regardless of inactivity. |
| `lock_timeout_mfa` | Require MFA on session timeout | boolean |  | Help: Enable two-factor authentication when the session timeout is reached. |
| `lock_timeout_inactivity` | Inactivity timeout | integer |  | Help: Time (in minutes) of user inactivity after which re-authentication is required. |
| `lock_timeout_inactivity_mfa` | Require MFA on inactivity timeout | boolean |  | Help: Enable two-factor authentication when the inactivity timeout is reached. |
| `has_lock_timeout` | Has Lock Timeout | boolean |  | computed by rule `_compute_has_lock_timeout` (not stored); Help: Requires re-authentication after the user's last connection |
| `lock_timeout_delay_unit` | Lock Timeout Delay Unit | selection |  | computed by rule `_compute_lock_timeout_delay_unit` (not stored) |
| `lock_timeout_delay_in_unit` | Lock Timeout Delay In Unit | integer |  | computed by rule `_compute_lock_timeout_delay_unit` (not stored) |
| `lock_timeout_2fa_selection` | Lock Timeout Two-factor authentication Selection | selection |  | computed by rule `_compute_lock_timeout_2fa_selection` (not stored); writable through an inverse rule |
| `has_lock_timeout_inactivity` | Has Lock Timeout Inactivity | boolean |  | computed by rule `_compute_lock_timeout_inactivity_bool` (not stored); Help: Requires re-authentication after a period of user inactivity |
| `lock_timeout_inactivity_delay_unit` | Lock Timeout Inactivity Delay Unit | selection |  | computed by rule `_compute_lock_timeout_inactivity_delay_unit` (not stored) |
| `lock_timeout_inactivity_delay_in_unit` | Lock Timeout Inactivity Delay In Unit | integer |  | computed by rule `_compute_lock_timeout_inactivity_delay_unit` (not stored) |
| `lock_timeout_inactivity_2fa_selection` | Lock Timeout Inactivity Two-factor authentication Selection | selection |  | computed by rule `_compute_lock_timeout_inactivity_2fa_selection` (not stored); writable through an inverse rule |

## Selection values

### `lock_timeout_2fa_selection` (Lock Timeout Two-factor authentication Selection)

| Value | Label |
|---|---|
| `without_2fa` | Logout |
| `with_2fa` | Logout with two-factor authentication |

### `lock_timeout_inactivity_2fa_selection` (Lock Timeout Inactivity Two-factor authentication Selection)

| Value | Label |
|---|---|
| `without_2fa` | Screen lock |
| `with_2fa` | Screen lock with two-factor authentication |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `UNIQUE (privilege_id, name)` | The name of the group must be unique within a group privilege! | `base` |
| `_check_api_key_duration` | Constraint | `CHECK(api_key_duration >= 0)` | The api key duration cannot be a negative value. | `base` |

## Operations (44)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_disjoint_groups` | validation | self | `base` | constrains: `implied_ids`, `implied_by_ids` |  |
| `_check_inherited_view_groups` | validation | self | `base` | constrains: `view_access` |  |
| `_check_user_disjoint_groups` | validation | self | `base` | constrains: `user_ids` |  |
| `_unlink_except_settings_group` | internal rule | self | `base` | ondelete |  |
| `_compute_full_name` | computation | self | `base` | depends: `privilege_id.name`, `name`; depends_context: `short_display_name` |  |
| `_search_full_name` | search rule | self, operator, operand | `base` |  |  |
| `_search` | search rule | self, domain, offset, limit, order, **kwargs | `base` | model |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `write` | lifecycle override | self, vals | `auth_timeout`, `base`, `im_livechat`, `mail`, `website_slides` |  | Override to invalidate `_get_lock_timeouts` cache if timeout fields are updated. |
| `_ensure_xml_id` | internal rule | self | `base` |  | Return the groups external identifiers, creating the external identifier for groups missing one |
| `_compute_all_user_ids` | computation | self | `base` | depends: `all_implied_by_ids.user_ids` |  |
| `_inverse_all_user_ids` | inverse computation | self | `base` |  |  |
| `_search_all_user_ids` | search rule | self, operator, value | `base` |  |  |
| `_compute_all_implied_ids` | computation | self | `base` | depends: `implied_ids.all_implied_ids` | Compute the reflexive transitive closure of implied_ids. |
| `_search_all_implied_ids` | search rule | self, operator, value | `base` |  | Compute the search on the reflexive transitive closure of implied_ids. |
| `_compute_all_implied_by_ids` | computation | self | `base` | depends: `implied_by_ids.all_implied_by_ids` | Compute the reflexive transitive closure of implied_by_ids. |
| `_search_all_implied_by_ids` | search rule | self, operator, value | `base` |  | Compute the search on the reflexive transitive closure of implied_by_ids. |
| `_get_user_type_groups` | preparation rule | self | `base` |  | Return the (disjoint) user type groups (employee, portal, public). |
| `_compute_disjoint_ids` | computation | self | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `auth_timeout`, `base` | model_create_multi | Override to invalidate `_get_lock_timeouts` cache if timeout fields are set on creation. |
| `unlink` | lifecycle override | self | `auth_timeout`, `base` |  | Override to invalidate `_get_lock_timeouts` cache if timeout fields exist on deleted records. |
| `_apply_group` | internal rule | self, implied_group | `base` |  | Add the given group to the groups implied by the current group :param implied_group: the implied group to add |
| `_remove_group` | internal rule | self, implied_group | `base` |  | Remove the given group from the implied groups of the current group :param implied_group: the implied group to remove |
| `_compute_view_group_hierarchy` | computation | self | `base` |  |  |
| `_get_view_group_hierarchy` | preparation rule | self | `base` | model |  |
| `_get_group_definitions` | preparation rule | self | `base` | model | Return the definition of all the groups as a :class:`~system.tools.SetDefinitions`. |
| `_is_feature_enabled` | internal rule | self, group_reference | `base` | model |  |
| `_compute_all_users_count` | computation | self | `base` | depends: `all_user_ids` |  |
| `action_show_all_users` | user action | self | `base` |  |  |
| `get_application_groups` | operation | self, domain | `account` | model |  |
| `_activate_group_account_secured` | internal rule | self | `account` | model |  |
| `_compute_has_lock_timeout` | computation | self | `auth_timeout` | depends: `lock_timeout` |  |
| `_compute_lock_timeout_delay_unit` | computation | self | `auth_timeout` | depends: `lock_timeout` |  |
| `_compute_lock_timeout_2fa_selection` | computation | self | `auth_timeout` | depends: `lock_timeout_mfa` |  |
| `_inverse_lock_timeout_2fa_selection` | inverse computation | self | `auth_timeout` |  |  |
| `_compute_lock_timeout_inactivity_bool` | computation | self | `auth_timeout` | depends: `lock_timeout_inactivity` |  |
| `_compute_lock_timeout_inactivity_delay_unit` | computation | self | `auth_timeout` | depends: `lock_timeout_inactivity` |  |
| `_compute_lock_timeout_inactivity_2fa_selection` | computation | self | `auth_timeout` | depends: `lock_timeout_inactivity_mfa` |  |
| `_inverse_lock_timeout_inactivity_2fa_selection` | inverse computation | self | `auth_timeout` |  |  |
| `_onchange_has_lock_timeout` | on change | self | `auth_timeout` | onchange: `has_lock_timeout` |  |
| `_onchange_lock_timeout_delay_unit` | on change | self | `auth_timeout` | onchange: `lock_timeout_delay_unit`, `lock_timeout_delay_in_unit` |  |
| `_onchange_has_lock_timeout_inactivity` | on change | self | `auth_timeout` | onchange: `has_lock_timeout_inactivity` |  |
| `_onchange_lock_timeout_inactivity_delay_unit` | on change | self | `auth_timeout` | onchange: `lock_timeout_inactivity_delay_unit`, `lock_timeout_inactivity_delay_in_unit` |  |
| `_get_lock_timeouts` | preparation rule | self | `auth_timeout` |  | Compute the session and inactivity timeout settings for the user.  This method returns the shortest configured timeouts (in seconds) across all groups implied by the user's group membership. For each type of timeout, it distinguishes between those that require MFA and those that do not.  :return: A dictionary with timeout types as keys and a list of tuples as values.     Each tuple is of the form (timeout_in_seconds, requires_mfa), ordered from shortest to longest.      Example::          {             'lock_timeout': [(43200, False), (86400, True)],             'lock_timeout_inactivity': [(90 |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_settings_group` | ValidationError | You cannot delete a group linked with a settings field. | `base` |
| `write` | UserError | The name of the group can not start with "-" | `base` |
| `_inverse_all_user_ids` | UserError | It is not possible to remove implied group %(group)s from users %(users)s | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `group_user` | no | yes | no | no | `base` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_timeout.auth_passkey_groups_form` | notebook | `base.view_groups_form` | `has_lock_timeout_inactivity`, `lock_timeout_inactivity`, `lock_timeout_inactivity_2fa_selection`, `lock_timeout_inactivity_delay_in_unit`, `lock_timeout_inactivity_delay_unit`, `has_lock_timeout`, `lock_timeout`, `lock_timeout_2fa_selection`, `lock_timeout_delay_in_unit`, `lock_timeout_delay_unit` |  |  | `auth_timeout` |
| `base.view_groups_search` | search |  | `full_name`, `implied_by_ids`, `implied_ids`, `share` |  | `Internal Groups` | `base` |
| `base.view_groups_list` | list |  | `privilege_id`, `name`, `comment`, `implied_ids`, `implied_by_ids` |  |  | `base` |
| `base.view_groups_form` | form |  | `all_users_count`, `name`, `privilege_id`, `share`, `api_key_duration`, `user_ids`, `implied_ids`, `implied_by_ids`, `disjoint_ids`, `menu_access`, `view_access`, `model_access`, `name`, `model_id`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink`, `rule_groups`, `name`, `model_id`, `domain_force`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink`, `comment` | `action_show_all_users` |  | `base` |
| `base.view_default_groups_form` | form |  | `implied_ids` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_groups` | Groups |  |  | `{'search_default_filter_no_share': 1}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.groups.json`; views: `../../../schemas/interfaces/views/res.groups.json`.
