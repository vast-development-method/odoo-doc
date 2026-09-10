# Lunch Alert (`lunch.alert`)

**Transport name:** `lunch.alert`  
**Storage name:** `lunch_alert`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Alert

## Identity and behavior

- Default ordering: `write_date desc, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (19)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Alert Name | single line text |  | required; translatable |
| `message` | Message | rich text |  | required; translatable |
| `mode` | Display | selection |  | default `alert` |
| `recipients` | Recipients | selection |  | default `everyone` |
| `notification_time` | Notification Time | float |  | default `10.0` |
| `notification_moment` | Notification Moment | selection |  | required; default `am` |
| `tz` | Timezone | selection |  | required; default computed dynamically (lambda self: self.env.user.tz or 'UTC') |
| `cron_id` | Cron | many to one | `ir.cron` | required; read only; on delete of the target: cascade |
| `until` | Show Until | date |  |  |
| `mon` | Mon | boolean |  | default `True` |
| `tue` | Tue | boolean |  | default `True` |
| `wed` | Wed | boolean |  | default `True` |
| `thu` | Thu | boolean |  | default `True` |
| `fri` | Fri | boolean |  | default `True` |
| `sat` | Sat | boolean |  | default `True` |
| `sun` | Sun | boolean |  | default `True` |
| `available_today` | Is Displayed Today | boolean |  | computed by rule `_compute_available_today` (not stored); searchable through a search rule |
| `active` | Active | boolean |  | default `True` |
| `location_ids` | Location | many to many | `lunch.location` |  |

## Selection values

### `mode` (Display)

| Value | Label |
|---|---|
| `alert` | Alert in app |
| `chat` | Chat notification |

### `recipients` (Recipients)

| Value | Label |
|---|---|
| `everyone` | Everyone |
| `last_week` | Employee who ordered last week |
| `last_month` | Employee who ordered last month |
| `last_year` | Employee who ordered last year |

### `notification_moment` (Notification Moment)

| Value | Label |
|---|---|
| `am` | AM |
| `pm` | PM |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_notification_time_range` | Constraint | `CHECK(notification_time >= 0 and notification_time <= 12)` | Notification time must be between 0 and 12 | `lunch` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_available_today` | computation | self | `lunch` | depends: `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun` |  |
| `_search_available_today` | search rule | self, operator, value | `lunch` |  |  |
| `_sync_cron` | internal rule | self | `lunch` |  | Synchronise the related cron fields to reflect this alert |
| `create` | lifecycle override | self, vals_list | `lunch` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `lunch` |  |  |
| `unlink` | lifecycle override | self | `lunch` |  |  |
| `_notify_chat` | internal rule | self | `lunch` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_alert_view_search` | search |  | `message` |  | `Currently inactive`, `Active`, `Archived` | `lunch` |
| `lunch.lunch_alert_view_tree` | list |  | `name`, `mode`, `message`, `available_today`, `active` |  |  | `lunch` |
| `lunch.lunch_alert_view_form` | form |  | `name`, `mode`, `recipients`, `location_ids`, `until`, `active`, `notification_time`, `notification_moment`, `tz`, `message` |  |  | `lunch` |
| `lunch.lunch_alert_view_kanban` | kanban |  | `name`, `mode`, `recipients`, `notification_time`, `notification_moment`, `location_ids` |  |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_alert_action` | Lunch Alerts | list,form,kanban | `['\|', ('active', '=', True), ('active', '=', False)]` | `{}` |  | `lunch` |

Machine-readable definition: `../../../schemas/data/entities/lunch.alert.json`; views: `../../../schemas/interfaces/views/lunch.alert.json`.
