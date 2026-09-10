# Gamification User Badge (`gamification.badge.user`)

**Transport name:** `gamification.badge.user`  
**Storage name:** `gamification_badge_user`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`  
**Extended by packages:** `hr_gamification`

Description: Gamification User Badge

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Default ordering: `create_date desc`
- Display name field: `badge_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; indexed; on delete of the target: cascade |
| `user_partner_id` | User Partner | many to one | `res.partner` | related through path `user_id.partner_id` |
| `sender_id` | Sender | many to one | `res.users` |  |
| `badge_id` | Badge | many to one | `gamification.badge` | required; indexed; on delete of the target: cascade |
| `challenge_id` | Challenge | many to one | `gamification.challenge` |  |
| `comment` | Comment | multi line text |  |  |
| `badge_name` | Badge Name | single line text |  | related through path `badge_id.name` |
| `level` | Badge Level | selection |  | read only; related through path `badge_id.level` and stored |
| `employee_id` | Employee | many to one | `hr.employee` | indexed |
| `has_edit_delete_access` | Has Edit Delete Access | boolean |  | computed by rule `_compute_has_edit_delete_access` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_send_badge` | internal rule | self | `gamification` |  | Send a notification to a user for receiving a badge  Does not verify constrains on badge granting. The users are added to the owner_ids (create badge_user if needed) The stats counters are incremented :param ids: list(int) of badge users that will receive the badge |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `gamification`, `hr_gamification` |  |  |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi |  |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `gamification` |  |  |
| `_check_employee_related_user` | validation | self | `hr_gamification` | constrains: `employee_id` |  |
| `_compute_has_edit_delete_access` | computation | self | `hr_gamification` |  |  |
| `action_open_badge` | user action | self | `hr_gamification` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_employee_related_user` | ValidationError | The selected employee does not correspond to the selected user. | `hr_gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | yes | yes | yes | no | `gamification` |
| `base.group_public` | no | yes | no | no | `gamification` |
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `base.group_user` | yes | yes | yes | yes | `hr_gamification` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Base group user granted badge write/unlink access | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Base group user not granted badge write/unlink access | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '!=', user.id)]` | True | False | True | False |
| Officer: Manage all employees- Badge Access | `[Command.link(ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.badge_user_kanban_view` | kanban |  | `create_date`, `badge_name`, `create_uid`, `create_uid`, `badge_id`, `comment` |  |  | `gamification` |
| `hr_gamification.view_current_badge_form` | form |  | `badge_id`, `create_uid`, `create_date`, `badge_id`, `comment` | `Save`, `Discard`, `Delete` |  | `hr_gamification` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `gamification.email_template_badge_received` | Gamification: Badge Received | New badge {{ object.badge_id.name }} granted |

Machine-readable definition: `../../../schemas/data/entities/gamification.badge.user.json`; views: `../../../schemas/interfaces/views/gamification.badge.user.json`.
