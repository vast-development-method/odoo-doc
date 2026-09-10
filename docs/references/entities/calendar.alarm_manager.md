# Event Alarm Manager (`calendar.alarm_manager`)

**Transport name:** `calendar.alarm_manager`  
**Storage name:** `calendar_alarm_manager`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `calendar`  
**Extended by packages:** `calendar_sms`, `google_calendar`, `microsoft_calendar`

Description: Event Alarm Manager

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_next_potential_limit_alarm` | preparation rule | self, alarm_type, seconds, partners | `calendar` |  |  |
| `do_check_alarm_for_one_date` | user action | self, one_date, event, event_maxdelta, in_the_next_X_seconds, alarm_type, after, missing | `calendar` |  | Search for some alarms in the interval of time determined by some parameters (after, in_the_next_X_seconds, ...) :param one_date: date of the event to check (not the same that in the event browse if recurrent) :param event: Event browse record :param event_maxdelta: biggest duration from alarms for this event :param in_the_next_X_seconds: looking in the future (in seconds) :param after: if not False: will return alert if after this date (date as string - todo: change in master) :param missing: if not False: will return alert even if we are too late :param notif: Looking for type notification : |
| `_get_notify_alert_extra_conditions` | preparation rule | self | `calendar`, `google_calendar`, `microsoft_calendar` | model | To be overriden on inherited modules adding extra conditions to extract only the unsynced events |
| `_get_events_by_alarm_to_notify` | preparation rule | self, alarm_type | `calendar` |  | Get the events with an alarm of the given type between the cron last call and now.  Please note that all new reminders created since the cron last call with an alarm prior to the cron last call are skipped by design. The attendees receive an invitation for any new event already. |
| `_send_reminder` | internal rule | self | `calendar_sms`, `calendar` | model | Cron method, overridden here to send SMS reminders as well |
| `get_next_notif` | operation | self | `calendar` | model |  |
| `do_notif_reminder` | user action | self, alert | `calendar` |  |  |
| `_notify_next_alarm` | internal rule | self, partner_ids | `calendar` |  | Sends through the bus the next alarm of given partners |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `calendar` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `calendar.ir_cron_scheduler_alarm` | Calendar: Event Reminder | 1 days | `_send_reminder` |  |

Machine-readable definition: `../../../schemas/data/entities/calendar.alarm_manager.json`.
