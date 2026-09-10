# Track Karma Changes (`gamification.karma.tracking`)

**Transport name:** `gamification.karma.tracking`  
**Storage name:** `gamification_karma_tracking`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`  
**Extended by packages:** `website_slides`, `website_forum`

Description: Track Karma Changes

## Identity and behavior

- Default ordering: `tracking_date desc, id desc`
- Display name field: `user_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; indexed; on delete of the target: cascade |
| `old_value` | Old Karma Value | integer |  | read only |
| `new_value` | New Karma Value | integer |  | required |
| `gain` | Gain | integer |  | computed by rule `_compute_gain` (not stored) |
| `consolidated` | Consolidated | boolean |  |  |
| `tracking_date` | Tracking Date | date and time |  | read only; default computed dynamically (fields.Datetime.now); indexed |
| `reason` | Description | multi line text |  | default computed dynamically (lambda self: _('Add Manually')) |
| `origin_ref` | Source | reference |  | default computed dynamically (lambda self: f'res.users,{self.env.user.id}') |
| `origin_ref_model_name` | Source Type | selection |  | computed by rule `_compute_origin_ref_model_name` and stored |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_origin_selection_values` | preparation rule | self | `gamification`, `website_forum`, `website_slides` |  |  |
| `_compute_gain` | computation | self | `gamification` | depends: `old_value`, `new_value` |  |
| `_compute_origin_ref_model_name` | computation | self | `gamification` | depends: `origin_ref` |  |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi |  |
| `_consolidate_cron` | internal rule | self | `gamification` | model | Consolidate the trackings 2 months ago. Used by a cron to cleanup tracking records. |
| `_process_consolidate` | background operation | self, from_date, end_date | `gamification` |  | Consolidate the karma trackings.  The consolidation keeps, for each user, the oldest "old_value" and the most recent "new_value", creates a new karma tracking with those values and removes all karma trackings between those dates. The origin / reason is changed on the consolidated records, so this information is lost in the process. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `gamification` |
| `base.group_system` | yes | yes | yes | yes | `gamification` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.gamification_karma_tracking_view_search` | search |  | `user_id`, `tracking_date`, `origin_ref_model_name` |  | `Consolidated`, `My Karma`, `Manual`, `Granted By`, `Source Type`, `Karma Owner` | `gamification` |
| `gamification.gamification_karma_tracking_view_tree` | list |  | `tracking_date`, `origin_ref`, `reason`, `user_id`, `old_value`, `gain`, `new_value` |  |  | `gamification` |
| `gamification.gamification_karma_tracking_view_form` | form |  | `user_id`, `tracking_date`, `gain`, `old_value`, `new_value`, `consolidated`, `origin_ref`, `reason` |  |  | `gamification` |
| `website_forum.gamification_karma_tracking_view_search` | xpath | `gamification.gamification_karma_tracking_view_search` |  |  | `Forum` | `website_forum` |
| `website_slides.gamification_karma_tracking_view_search` | xpath | `gamification.gamification_karma_tracking_view_search` |  |  | `Course`, `Quiz` | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.gamification_karma_tracking_action` | Karma Tracking | list,form |  |  |  | `gamification` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `gamification.ir_cron_consolidate` | Gamification: Karma tracking consolidation | 1 months | `_consolidate_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/gamification.karma.tracking.json`; views: `../../../schemas/interfaces/views/gamification.karma.tracking.json`.
