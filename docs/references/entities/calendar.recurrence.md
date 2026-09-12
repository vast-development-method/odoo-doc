# Event Recurrence Rule (`calendar.recurrence`)

**Transport name:** `calendar.recurrence`  
**Storage name:** `calendar_recurrence`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`  
**Extended by packages:** `google_calendar`, `microsoft_calendar`

Description: Event Recurrence Rule

## Identity and behavior

- Mixins (classical inheritance): `google.calendar.sync`, `microsoft.calendar.sync`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored |
| `base_event_id` | Base Event | many to one | `calendar.event` | not copied on duplication; on delete of the target: set null |
| `calendar_event_ids` | Calendar Event | one to many | `calendar.event` | inverse field `recurrence_id` |
| `event_tz` | Timezone | selection |  | default computed dynamically (lambda self: self.env.context.get('tz') or self.env.user.tz) |
| `rrule` | Rrule | single line text |  | computed by rule `_compute_rrule` and stored; writable through an inverse rule |
| `dtstart` | Dtstart | date and time |  | computed by rule `_compute_dtstart` (not stored) |
| `rrule_type` | Rrule Type | selection |  | default `weekly` |
| `end_type` | End Type | selection |  | default `count` |
| `interval` | Interval | integer |  | default `1` |
| `count` | Count | integer |  | default `1` |
| `mon` | Mon | boolean |  |  |
| `tue` | Tue | boolean |  |  |
| `wed` | Wed | boolean |  |  |
| `thu` | Thu | boolean |  |  |
| `fri` | Fri | boolean |  |  |
| `sat` | Sat | boolean |  |  |
| `sun` | Sun | boolean |  |  |
| `month_by` | Month By | selection |  | default `date` |
| `day` | Day | integer |  | default `1` |
| `weekday` | Weekday | selection |  |  |
| `byday` | By day | selection |  |  |
| `until` | Repeat Until | date |  |  |
| `trigger_id` | Trigger | many to one | `ir.cron.trigger` |  |
| `need_sync_m` | Need Sync M | boolean |  | default  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_month_day` | Constraint | `{"expression": "\"CHECK (\\n        rrule_type != 'monthly'\\n        OR month_by != 'day'\\n        OR day >= 1 AND day <= 31\\n        OR weekday IN %s AND byday IN %s)\" % (tuple((wd[0] for wd in WEEKDAY_SELECTION)), tuple((bd[0] for bd in BYDAY_SELECTION)))"}` | The day must be between 1 and 31 | `calendar` |

## Operations (55)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_daily_recurrence_name` | preparation rule | self | `calendar` |  |  |
| `_get_weekly_recurrence_name` | preparation rule | self | `calendar` |  |  |
| `_get_monthly_recurrence_name` | preparation rule | self | `calendar` |  |  |
| `_get_yearly_recurrence_name` | preparation rule | self | `calendar` |  |  |
| `get_recurrence_name` | operation | self | `calendar` |  |  |
| `_compute_name` | computation | self | `calendar` | depends: `rrule` |  |
| `_compute_dtstart` | computation | self | `calendar` | depends: `calendar_event_ids.start` |  |
| `_compute_rrule` | computation | self | `calendar`, `microsoft_calendar` | depends: `byday`, `until`, `rrule_type`, `month_by`, `interval`, `count`, `end_type`, `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun`, `day`, `weekday` |  |
| `_inverse_rrule` | inverse computation | self | `calendar`, `microsoft_calendar` |  |  |
| `_reconcile_events` | internal rule | self, ranges | `calendar` |  | :param ranges: iterable of tuples (datetime_start, datetime_stop) :return: tuple (events of the recurrence already in sync with ranges,          and ranges not covered by any events) |
| `_select_new_base_event` | internal rule | self | `calendar` |  | when the base event is no more available (archived, deleted, etc.), a new one should be selected |
| `_apply_recurrence` | internal rule | self, specific_values_creation, no_send_edit, generic_values_creation | `calendar`, `google_calendar`, `microsoft_calendar` |  | Create missing events in the recurrence and detach events which no longer follow the recurrence rules. :return: detached events |
| `_setup_alarms` | internal rule | self, recurrence_update | `calendar` |  | Schedule cron triggers for future events Create one ir.cron.trigger per recurrence. :param recurrence_update: boolean: if true, update all recurrences in self, else only the recurrences        without trigger |
| `_split_from` | internal rule | self, event, recurrence_values | `calendar`, `microsoft_calendar` |  | Stops the current recurrence at the given event and creates a new one starting with the event. :param event: starting point of the new recurrence :param recurrence_values: values applied to the new recurrence :return: new recurrence |
| `_stop_at` | internal rule | self, event | `calendar` |  | Stops the recurrence at the given event. Detach the event and all following events from the recurrence.  :return: detached events from the recurrence |
| `_detach_events` | internal rule | self, events | `calendar` | model |  |
| `_write_events` | internal rule | self, values, dtstart | `calendar`, `google_calendar`, `microsoft_calendar` |  | Write values on events in the recurrence. :param values: event values :param dstart: if provided, only write events starting from this point in time |
| `_rrule_serialize` | internal rule | self | `calendar` |  | Compute rule string according to value type RECUR of iCalendar :return: string containing recurring rule (empty if no rule) |
| `_rrule_parse` | internal rule | self, rule_str, date_start | `calendar` | model |  |
| `_get_lang_week_start` | preparation rule | self | `calendar` |  |  |
| `_get_start_of_period` | preparation rule | self, dt | `calendar` |  |  |
| `_get_first_event` | preparation rule | self, include_outliers | `calendar` |  |  |
| `_get_outliers` | preparation rule | self | `calendar` |  |  |
| `_range_calculation` | internal rule | self, event, duration | `calendar` |  | Calculate the range of recurrence when applying the recurrence The following issues are taken into account:     start of period is sometimes in the past (weekly or monthly rule).     We can easily filter these range values but then the count value may be wrong...     In that case, we just increase the count value, recompute the ranges and dismiss the useless values |
| `_get_ranges` | preparation rule | self, start, event_duration | `calendar` |  |  |
| `_get_timezone` | preparation rule | self | `calendar` |  |  |
| `_get_occurrences` | preparation rule | self, dtstart | `calendar` |  | Get ocurrences of the rrule :param dtstart: start of the recurrence :return: iterable of datetimes |
| `_get_events_from` | preparation rule | self, dtstart | `calendar` |  |  |
| `_get_week_days` | preparation rule | self | `calendar` |  | :return: tuple of rrule weekdays for this recurrence. |
| `_is_allday` | internal rule | self | `calendar` |  | Returns whether a majority of events are allday or not (there might be some outlier events) |
| `_get_rrule` | preparation rule | self, dtstart | `calendar`, `microsoft_calendar` |  |  |
| `_is_event_over` | internal rule | self | `calendar` |  | Check if all events in this recurrence are in the past. :return: True if all events are over, False otherwise |
| `_get_event_google_id` | preparation rule | self, event | `google_calendar` |  | Return the Google id of recurring event. Google ids of recurrence instances are formatted as: {recurrence google_id}_{UTC starting time in compacted ISO8601} |
| `_cancel` | internal rule | self | `google_calendar` |  |  |
| `_get_google_synced_fields` | preparation rule | self | `google_calendar` |  |  |
| `_restart_google_sync` | internal rule | self | `google_calendar` | model |  |
| `_write_from_google` | internal rule | self, gevent, vals | `google_calendar` |  |  |
| `_create_from_google` | internal rule | self, gevents, vals_list | `google_calendar` |  |  |
| `_get_sync_domain` | preparation rule | self | `google_calendar` |  |  |
| `_system_values` | internal rule | self, google_recurrence, default_reminders | `google_calendar` | model |  |
| `_google_values` | internal rule | self | `google_calendar` |  |  |
| `_get_event_user` | preparation rule | self | `google_calendar` |  |  |
| `_is_google_insertion_blocked` | internal rule | self, sender_user | `google_calendar` |  |  |
| `_get_organizer` | preparation rule | self | `microsoft_calendar` |  |  |
| `_get_microsoft_synced_fields` | preparation rule | self | `microsoft_calendar` |  |  |
| `_restart_microsoft_sync` | internal rule | self | `microsoft_calendar` | model |  |
| `_has_base_event_time_fields_changed` | internal rule | self, new | `microsoft_calendar` |  | Indicates if at least one time field of the base event has changed, based on provided `new` values. Note: for all day event comparison, hours/minutes are ignored. |
| `_write_from_microsoft` | internal rule | self, microsoft_event, vals | `microsoft_calendar` |  |  |
| `_get_microsoft_sync_domain` | preparation rule | self | `microsoft_calendar` |  |  |
| `_cancel_microsoft` | internal rule | self | `microsoft_calendar` |  |  |
| `_microsoft_to_system_values` | internal rule | self, microsoft_recurrence, default_reminders, default_values, with_ids | `microsoft_calendar` | model |  |
| `_microsoft_values` | internal rule | self, fields_to_sync, initial_values | `microsoft_calendar` |  | Get values to update the whole Outlook event recurrence. (done through the first event of the Outlook recurrence). |
| `_ensure_attendees_have_email` | internal rule | self | `microsoft_calendar` |  |  |
| `_get_event_user_m` | preparation rule | self, user_id | `microsoft_calendar` |  | Get the user who will send the request to Microsoft (organizer if synchronized and current user otherwise). |
| `_is_microsoft_insertion_blocked` | internal rule | self, sender_user | `microsoft_calendar` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_rrule_serialize` | UserError | The interval cannot be negative. | `calendar` |
| `_rrule_serialize` | UserError | The number of repetitions cannot be negative. | `calendar` |
| `_get_rrule` | UserError | You have to choose at least one day in the week | `calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.recurrence.json`.
