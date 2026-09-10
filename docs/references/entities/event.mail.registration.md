# Registration Mail Scheduler (`event.mail.registration`)

**Transport name:** `event.mail.registration`  
**Storage name:** `event_mail_registration`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_sms`

Description: Registration Mail Scheduler

## Identity and behavior

- Default ordering: `scheduled_date DESC, id ASC`
- Display name field: `scheduler_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `scheduler_id` | Mail Scheduler | many to one | `event.mail` | required; indexed; on delete of the target: cascade |
| `registration_id` | Attendee | many to one | `event.registration` | required; indexed; on delete of the target: cascade |
| `scheduled_date` | Scheduled Time | date and time |  | computed by rule `_compute_scheduled_date` and stored |
| `mail_sent` | Mail Sent | boolean |  |  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_scheduled_date` | computation | self | `event` | depends: `registration_id`, `scheduler_id.interval_unit`, `scheduler_id.interval_type` |  |
| `execute` | operation | self | `event` |  |  |
| `_execute_on_registrations` | internal rule | self | `event_sms`, `event` |  | Private mail registration execution. We consider input is already filtered at this point, allowing to let caller do optimizations when managing batches of registrations. |
| `_get_skip_domain` | preparation rule | self | `event` |  | Domain of mail registrations ot skip: not already done, linked to a valid registration, and scheduled in the past. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |

Machine-readable definition: `../../../schemas/data/entities/event.mail.registration.json`.
