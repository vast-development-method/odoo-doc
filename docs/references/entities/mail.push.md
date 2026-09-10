# Push Notifications (`mail.push`)

**Transport name:** `mail.push`  
**Storage name:** `mail_push`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Push Notifications

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mail_push_device_id` | devices | many to one | `mail.push.device` | required; on delete of the target: cascade |
| `payload` | Payload | multi line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_push_notification_to_endpoint` | internal rule | self, batch_size | `mail` | model | Send to web browser endpoint computed notification |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_web_push_notification` | Mail: send web push notification | 1 days | `_push_notification_to_endpoint` |  |

Machine-readable definition: `../../../schemas/data/entities/mail.push.json`.
