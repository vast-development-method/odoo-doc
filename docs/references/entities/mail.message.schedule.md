# Scheduled Messages (`mail.message.schedule`)

**Transport name:** `mail.message.schedule`  
**Storage name:** `mail_message_schedule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Scheduled Messages

## Identity and behavior

- Default ordering: `scheduled_datetime DESC, id DESC`
- Display name field: `mail_message_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mail_message_id` | Message | many to one | `mail.message` | required; on delete of the target: cascade |
| `notification_parameters` | Notification Parameter | multi line text |  |  |
| `scheduled_datetime` | Scheduled Send Date | date and time |  | required; Help: Datetime at which notification should be sent. |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `_send_notifications_cron` | internal rule | self | `mail` | model |  |
| `force_send` | operation | self | `mail` |  | Launch notification process independently from the expected date. |
| `_send_notifications` | internal rule | self, default_notify_kwargs | `mail` |  | Send notification for scheduled messages.  :param dict default_notify_kwargs: optional parameters to propagate to   `notify_thread`. Those are default values overridden by content of   `notification_parameters` field. |
| `_send_message_notifications` | internal rule | self, messages, default_notify_kwargs | `mail` | model | Send scheduled notification for given messages.  :param <mail.message> messages: scheduled sending related to those messages   will be sent now; :param dict default_notify_kwargs: optional parameters to propagate to   `notify_thread`. Those are default values overridden by content of   `notification_parameters` field.  :returns: False if no schedule has been found, True otherwise :rtype: bool |
| `_update_message_scheduled_datetime` | internal rule | self, messages, new_datetime | `mail` | model | Update scheduled datetime for scheduled sending related to messages.  :param <mail.message> messages: scheduled sending related to those messages   will be updated. Missing one are skipped; :param datetime new_datetime: new datetime for sending. New triggers   are created based on it;  :returns: False if no schedule has been found, True otherwise :rtype: bool |
| `_group_by_model` | internal rule | self | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_message_schedule_view_form` | form |  | `mail_message_id`, `scheduled_datetime`, `notification_parameters` | `Force Send` |  | `mail` |
| `mail.mail_message_schedule_view_tree` | list |  | `mail_message_id`, `scheduled_datetime` |  |  | `mail` |
| `mail.mail_message_schedule_view_search` | search |  | `mail_message_id` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_message_schedule_action` | Scheduled Messages | list,form |  | `{}` |  | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_send_scheduled_message` | Notification: Notify scheduled messages | 1 hours | `_send_notifications_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/mail.message.schedule.json`; views: `../../../schemas/interfaces/views/mail.message.schedule.json`.
