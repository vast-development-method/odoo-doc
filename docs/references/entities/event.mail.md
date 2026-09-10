# Event Automated Mailing (`event.mail`)

**Transport name:** `event.mail`  
**Storage name:** `event_mail`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_sms`

Description: Event Automated Mailing

## Identity and behavior

- Display name field: `event_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Event | many to one | `event.event` | required; indexed; on delete of the target: cascade |
| `sequence` | Display order | integer |  |  |
| `interval_nbr` | Interval | integer |  | default `1` |
| `interval_unit` | Unit | selection |  | required; default `hours` |
| `interval_type` | Trigger | selection |  | required; default `before_event`; Help: Indicates when the communication is sent. If the event has multiple slots, the interval is related to each time slot instead of the whole event. |
| `scheduled_date` | Schedule Date | date and time |  | computed by rule `_compute_scheduled_date` and stored |
| `error_datetime` | Last Error | date and time |  |  |
| `last_registration_id` | Last Attendee | many to one | `event.registration` |  |
| `mail_registration_ids` | Mail Registration | one to many | `event.mail.registration` | inverse field `scheduler_id`; Help: Communication related to event registrations |
| `mail_slot_ids` | Mail Slot | one to many | `event.mail.slot` | inverse field `scheduler_id`; Help: Slot-based communication |
| `mail_done` | Sent | boolean |  | read only; not copied on duplication |
| `mail_state` | Global communication Status | selection |  | computed by rule `_compute_mail_state` (not stored) |
| `mail_count_done` | # Sent | integer |  | read only; not copied on duplication |
| `notification_type` | Send | selection |  | computed by rule `_compute_notification_type` (not stored); extended by packages `event_sms` |
| `template_ref` | Template | reference |  | required; on delete of the target: {"sms.template": "cascade"}; extended by packages `event_sms` |

## Selection values

### `interval_unit` (Unit)

| Value | Label |
|---|---|
| `now` | Immediately |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

### `interval_type` (Trigger)

| Value | Label |
|---|---|
| `after_sub` | After each registration |
| `before_event` | Before the event starts |
| `after_event_start` | After the event started |
| `after_event` | After the event ended |
| `before_event_end` | Before the event ends |

### `mail_state` (Global communication Status)

| Value | Label |
|---|---|
| `running` | Running |
| `scheduled` | Scheduled |
| `sent` | Sent |
| `error` | Error |
| `cancelled` | Cancelled |

### `notification_type` (Send)

| Value | Label |
|---|---|
| `mail` | Mail |
| `sms` | SMS |

### `template_ref` (Template)

| Value | Label |
|---|---|
| `mail.template` | Mail |
| `sms.template` | SMS |

## State fields

State machine fields of this entity: `mail_state`. Transitions are specified in the domain documents.

## Operations (18)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_scheduled_date` | computation | self | `event` | depends: `event_id.date_begin`, `event_id.date_end`, `interval_type`, `interval_unit`, `interval_nbr` |  |
| `_compute_mail_state` | computation | self | `event` | depends: `error_datetime`, `interval_type`, `mail_done`, `event_id` |  |
| `_compute_notification_type` | computation | self | `event_sms`, `event` | depends: `template_ref` | Assigns the type of template in use, if any is set. |
| `execute` | operation | self | `event` |  |  |
| `_execute_event_based` | internal rule | self, mail_slot | `event` |  | Main scheduler method when running in event-based mode aka 'after_event' or 'before_event' (and their negative counterparts). This is a global communication done once i.e. we do not track each registration individually.  :param mail_slot: optional <event.mail.slot> slot-specific event communication,   when event uses slots. In that case, it works like the classic event   communication (iterative, ...) but information is specific to each   slot (last registration, scheduled datetime, ...) |
| `_execute_event_based_for_registrations` | internal rule | self, registrations | `event_sms`, `event` |  | Method doing notification and recipients specific implementation of contacting attendees globally.  :param registrations: a recordset of registrations to contact |
| `_execute_slot_based` | internal rule | self | `event` |  | Main scheduler method when running in slot-based mode aka 'after_event' or 'before_event' (and their negative counterparts) on events with slots. This is a global communication done once i.e. we do not track each registration individually. |
| `_execute_attendee_based` | internal rule | self | `event` |  | Main scheduler method when running in attendee-based mode aka 'after_sub'. This relies on a sub model allowing to know which registrations have been contacted.  It currently does two main things   * generate missing 'event.mail.registrations' which are scheduled     communication linked to registrations;   * launch registration-based communication, splitting in batches as     it may imply a lot of computation. When having more than given     limit to handle, schedule another call of cron to avoid having to     wait another cron interval check; |
| `_create_missing_mail_registrations` | internal rule | self, registrations | `event` |  |  |
| `_refresh_mail_count_done` | internal rule | self, mail_slot | `event` |  |  |
| `_filter_template_ref` | internal rule | self | `event` |  | Check for valid template reference: existing, working template |
| `_send_mail` | internal rule | self, registrations | `event` |  | Mail action: send mail to attendees |
| `_template_model_by_notification_type` | internal rule | self | `event_sms`, `event` |  |  |
| `_prepare_event_mail_values` | preparation rule | self | `event` |  |  |
| `_warn_error` | internal rule | self, exception | `event` |  |  |
| `run` | operation | self, autocommit | `event` | model | Backward compatible method, notably if crons are not updated when migrating for some reason. |
| `schedule_communications` | operation | self, autocommit | `event` | model |  |
| `_send_sms` | internal rule | self, registrations | `event_sms` |  | SMS action: send SMS to attendees |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.view_event_mail_form` | form |  | `event_id`, `template_ref`, `mail_state`, `interval_nbr`, `interval_unit`, `interval_type`, `scheduled_date`, `mail_registration_ids`, `registration_id`, `scheduled_date`, `mail_sent` |  |  | `event` |
| `event.view_event_mail_tree` | list |  | `event_id`, `template_ref`, `scheduled_date`, `mail_count_done`, `mail_state` |  |  | `event` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.action_event_mail` | Events Mail Schedulers |  |  | `{'create': False}` |  | `event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.menu_event_mail_schedulers` |  |  | `event.action_event_mail` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `event.event_mail_scheduler` | Event: Mail Scheduler | 24 hours | `schedule_communications` |  |

Machine-readable definition: `../../../schemas/data/entities/event.mail.json`; views: `../../../schemas/interfaces/views/event.mail.json`.
