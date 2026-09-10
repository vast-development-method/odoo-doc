# Event Alarm (`calendar.alarm`)

**Transport name:** `calendar.alarm`  
**Storage name:** `calendar_alarm`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`  
**Extended by packages:** `calendar_sms`

Description: Event Alarm

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `alarm_type` | Type | selection |  | required; default `email`; on delete of the target: {"sms": "set default"}; extended by packages `calendar_sms` |
| `duration` | Remind Before | integer |  | required; default `1` |
| `interval` | Unit | selection |  | required; default `hours` |
| `duration_minutes` | Duration in minutes | integer |  | computed by rule `_compute_duration_minutes` and stored; searchable through a search rule |
| `mail_template_id` | Email Template | many to one | `mail.template` | computed by rule `_compute_mail_template_id` and stored; restricted by domain `[["model", "in", ["calendar.attendee"]]]`; Help: Template used to render mail reminder content. |
| `body` | Additional Message | multi line text |  | Help: Additional message that would be sent with the notification for the reminder |
| `notify_responsible` | Notify Responsible | boolean |  | default  |
| `sms_template_id` | text message Template | many to one | `sms.template` | computed by rule `_compute_sms_template_id` and stored; restricted by domain `[["model", "in", ["calendar.event"]]]`; Help: Template used to render SMS reminder content. |

## Selection values

### `alarm_type` (Type)

| Value | Label |
|---|---|
| `notification` | Notification |
| `email` | Email |
| `sms` | SMS Text Message |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_duration_minutes` | computation | self | `calendar` | depends: `interval`, `duration` |  |
| `_compute_mail_template_id` | computation | self | `calendar` | depends: `alarm_type`, `mail_template_id` |  |
| `_search_duration_minutes` | search rule | self, operator, value | `calendar` |  |  |
| `_onchange_duration_interval` | on change | self | `calendar` | onchange: `duration`, `interval`, `alarm_type`, `notify_responsible` |  |
| `_compute_sms_template_id` | computation | self | `calendar_sms` | depends: `alarm_type`, `sms_template_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `calendar` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.view_calendar_alarm_tree` | list |  | `name`, `alarm_type`, `duration`, `interval` |  |  | `calendar` |
| `calendar.calendar_alarm_view_form` | form |  | `name`, `alarm_type`, `mail_template_id`, `body`, `duration`, `interval`, `notify_responsible` |  |  | `calendar` |
| `calendar_sms.calendar_alarm_view_form` | xpath | `calendar.calendar_alarm_view_form` | `sms_template_id` |  |  | `calendar_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `calendar.action_calendar_alarm` | Calendar Alarm | list,form |  |  |  | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.alarm.json`; views: `../../../schemas/interfaces/views/calendar.alarm.json`.
