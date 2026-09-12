# Link text message to mailing/sms tracking models (`sms.tracker`)

**Transport name:** `sms.tracker`  
**Storage name:** `sms_tracker`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sms`  
**Extended by packages:** `mass_mailing_sms`, `sms_twilio`

Description: Link SMS to mailing/sms tracking models

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sms_uuid` | text message uuid | single line text |  | required |
| `mail_notification_id` | Mail Notification | many to one | `mail.notification` | indexed (btree_not_null); on delete of the target: cascade |
| `mailing_trace_id` | Mailing Trace | many to one | `mailing.trace` | indexed (btree_not_null); on delete of the target: cascade |
| `sms_twilio_sid` | Twilio text message SID | single line text |  | read only |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_sms_uuid_unique` | Constraint | `unique(sms_uuid)` | A record for this UUID already exists | `sms` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_action_update_from_provider_error` | internal rule | self, provider_error | `mass_mailing_sms`, `sms` |  | :param str provider_error: value returned by SMS service provider (IAP) or any string.     If provided, notification values will be derived from it.     (see `_get_tracker_values_from_provider_error`) |
| `_action_update_from_sms_state` | internal rule | self, sms_state, failure_type, failure_reason | `mass_mailing_sms`, `sms` |  |  |
| `_update_sms_notifications` | internal rule | self, notification_status, failure_type, failure_reason | `sms` |  |  |
| `_update_sms_traces` | internal rule | self, trace_status, failure_type, failure_reason | `mass_mailing_sms` |  |  |
| `_update_sms_mailings` | internal rule | self, trace_status, traces | `mass_mailing_sms` |  |  |
| `_action_update_from_twilio_error` | internal rule | self, sms_status, error_code, error_message | `sms_twilio` |  | Update the SMS tracker with the Twilio Status and Error code/msg |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing_sms` |
| all internal users | no | no | no | no | `sms` |
| `base.group_system` | yes | yes | yes | yes | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.tracker.json`.
