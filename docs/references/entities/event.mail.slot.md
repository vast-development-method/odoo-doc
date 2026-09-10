# Slot Mail Scheduler (`event.mail.slot`)

**Transport name:** `event.mail.slot`  
**Storage name:** `event_mail_slot`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`

Description: Slot Mail Scheduler

## Identity and behavior

- Default ordering: `scheduled_date DESC, id ASC`
- Display name field: `scheduler_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_slot_id` | Slot | many to one | `event.slot` | required; on delete of the target: cascade |
| `scheduled_date` | Schedule Date | date and time |  | computed by rule `_compute_scheduled_date` and stored |
| `scheduler_id` | Mail Scheduler | many to one | `event.mail` | required; indexed; on delete of the target: cascade |
| `last_registration_id` | Last Attendee | many to one | `event.registration` |  |
| `mail_count_done` | # Sent | integer |  | read only; not copied on duplication |
| `mail_done` | Sent | boolean |  | read only; not copied on duplication |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_scheduled_date` | computation | self | `event` | depends: `event_slot_id.start_datetime`, `event_slot_id.end_datetime`, `scheduler_id.interval_unit`, `scheduler_id.interval_type` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |

Machine-readable definition: `../../../schemas/data/entities/event.mail.slot.json`.
