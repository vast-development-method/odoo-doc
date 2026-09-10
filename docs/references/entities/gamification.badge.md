# Gamification Badge (`gamification.badge`)

**Transport name:** `gamification.badge`  
**Storage name:** `gamification_badge`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`  
**Extended by packages:** `hr_gamification`, `survey`, `website_profile`

Description: Gamification Badge

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `image.mixin`, `website.published.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (23)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Badge | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `description` | Description | rich text |  | translatable |
| `level` | Forum Badge Level | selection |  | default `bronze` |
| `rule_auth` | Allowance to Grant | selection |  | required; default `everyone`; Help: Who can grant this badge |
| `rule_auth_user_ids` | Authorized Users | many to many | `res.users` | association table `rel_badge_auth_users`; Help: Only these people can give this badge |
| `rule_auth_badge_ids` | Required Badges | many to many | `gamification.badge` | association table `gamification_badge_rule_badge_rel`; Help: Only the people having these badges can give this badge |
| `rule_max` | Monthly Limited Sending | boolean |  | Help: Check to set a monthly limit per person of sending this badge |
| `rule_max_number` | Limitation Number | integer |  | Help: The maximum number of time this badge can be sent per month per person. |
| `challenge_ids` | Reward of Challenges | one to many | `gamification.challenge` | inverse field `reward_id` |
| `goal_definition_ids` | Rewarded by | many to many | `gamification.goal.definition` | association table `badge_unlocked_definition_rel`; Help: The users that have succeeded these goals will receive automatically the badge. |
| `owner_ids` | Owners | one to many | `gamification.badge.user` | inverse field `badge_id`; Help: The list of instances of this badge granted to users |
| `granted_count` | Total | integer |  | computed by rule `_get_owners_info` (not stored); Help: The number of time this badge has been received. |
| `granted_users_count` | Number of users | integer |  | computed by rule `_get_owners_info` (not stored); Help: The number of time this badge has been received by unique users. |
| `unique_owner_ids` | Unique Owners | many to many | `res.users` | computed by rule `_get_owners_info` (not stored); Help: The list of unique users having received this badge. |
| `stat_this_month` | Monthly total | integer |  | computed by rule `_get_badge_user_stats` (not stored); Help: The number of time this badge has been received this month. |
| `stat_my` | My Total | integer |  | computed by rule `_get_badge_user_stats` (not stored); Help: The number of time the current user has received this badge. |
| `stat_my_this_month` | My Monthly Total | integer |  | computed by rule `_get_badge_user_stats` (not stored); Help: The number of time the current user has received this badge this month. |
| `stat_my_monthly_sending` | My Monthly Sending Total | integer |  | computed by rule `_get_badge_user_stats` (not stored); Help: The number of time the current user has sent this badge this month. |
| `remaining_sending` | Remaining Sending Allowed | integer |  | computed by rule `_remaining_sending_calc` (not stored); Help: If a maximum is set |
| `granted_employees_count` | Granted Employees Count | integer |  | computed by rule `_compute_granted_employees_count` (not stored) |
| `survey_ids` | Survey Ids | one to many | `survey.survey` | inverse field `certification_badge_id` |
| `survey_id` | Survey | many to one | `survey.survey` | computed by rule `_compute_survey_id` and stored |

## Selection values

### `level` (Forum Badge Level)

| Value | Label |
|---|---|
| `bronze` | Bronze |
| `silver` | Silver |
| `gold` | Gold |

### `rule_auth` (Allowance to Grant)

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `users` | A selected list of users |
| `having` | People having some badges |
| `nobody` | No one, assigned through challenges |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_owners_info` | computation | self | `gamification` | depends: `owner_ids` | Return: the list of unique res.users ids having received this badge the total number of time this badge was granted the total number of users this badge was granted to |
| `_get_badge_user_stats` | computation | self | `gamification` | depends: `owner_ids.badge_id`, `owner_ids.create_date`, `owner_ids.user_id` | Return stats related to badge users |
| `_remaining_sending_calc` | computation | self | `gamification` | depends: `rule_auth`, `rule_auth_user_ids`, `rule_auth_badge_ids`, `rule_max`, `rule_max_number`, `stat_my_monthly_sending` | Computes the number of badges remaining the user can send  0 if not allowed or no remaining integer if limited sending -1 if infinite (should not be displayed) |
| `check_granting` | operation | self | `gamification` |  | Check the user 'uid' can grant the badge 'badge_id' and raise the appropriate exception if not  Do not check for SUPERUSER_ID |
| `_can_grant_badge` | internal rule | self | `gamification` |  | Check if a user can grant a badge to another user  :return: integer representing the permission. |
| `_compute_granted_employees_count` | computation | self | `hr_gamification` | depends: `owner_ids.employee_id` |  |
| `get_granted_employees` | operation | self | `hr_gamification` |  |  |
| `_compute_survey_id` | computation | self | `survey` | depends: `survey_ids.certification_badge_id` |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_granting` | UserError | This badge can not be sent by users. | `gamification` |
| `check_granting` | UserError | You are not in the user allowed list. | `gamification` |
| `check_granting` | UserError | You do not have the required badges. | `gamification` |
| `check_granting` | UserError | You have already sent this badge too many time this month. | `gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | no | yes | no | no | `gamification` |
| `base.group_public` | no | yes | no | no | `gamification` |
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |
| `group_survey_user` | yes | yes | yes | yes | `survey` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.gamification_badge_view_search` | search |  | `name` |  | `Archived` | `gamification` |
| `gamification.badge_list_view` | list |  | `name`, `granted_count`, `stat_this_month`, `stat_my`, `rule_auth` |  |  | `gamification` |
| `gamification.badge_form_view` | form |  | `image_1920`, `name`, `description`, `active`, `rule_auth`, `rule_auth_user_ids`, `rule_auth_badge_ids`, `rule_max`, `rule_max_number`, `stat_my_monthly_sending`, `remaining_sending`, `challenge_ids`, `level`, `granted_count`, `stat_this_month`, `granted_users_count`, `stat_my`, `stat_my_this_month` | `Grant this Badge` |  | `gamification` |
| `gamification.badge_kanban_view` | kanban |  | `remaining_sending`, `image_1024`, `name`, `stat_my_monthly_sending`, `rule_max_number`, `stat_my_monthly_sending`, `granted_count`, `stat_this_month`, `description`, `unique_owner_ids` | `%(action_grant_wizard)d` |  | `gamification` |
| `hr_gamification.hr_badge_form_view` | div | `gamification.badge_form_view` | `granted_employees_count` | `get_granted_employees` |  | `hr_gamification` |
| `survey.gamification_badge_form_view_simplified` | form |  | `image_1920`, `name`, `description`, `level` |  |  | `survey` |
| `website_profile.gamification_badge_view_form` | xpath | `gamification.badge_form_view` | `is_published` | `website_publish_button` |  | `website_profile` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.badge_list_action` | Badges | kanban,list,form |  |  |  | `gamification` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_gamification.gamification_badge_menu_hr` |  | `menu_hr_gamification` | `gamification.badge_list_action` |  |  |
| `website_forum.menu_forum_badges` | Badges | `menu_website_forum_global` | `gamification.badge_list_action` | 40 |  |

Machine-readable definition: `../../../schemas/data/entities/gamification.badge.json`; views: `../../../schemas/interfaces/views/gamification.badge.json`.
