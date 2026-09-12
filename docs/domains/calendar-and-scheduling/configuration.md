# Configuration of Calendar and Scheduling

## 1. Overview

This document lists everything an administrator can set, everything the domain ships as data, and
everything that governs who may do what. A rebuild that reproduces this chapter reproduces the
domain's administrative surface exactly.

| Chapter | Content |
|---|---|
| [2](#2-capability-packages) | The nine capability packages and what installing each one changes |
| [3](#3-system-parameters) | Every stored setting, with its shipped value and its effect |
| [4](#4-the-default-meeting-duration) | The per-user and per-company default duration |
| [5](#5-shipped-reminders) | The seven reminder records |
| [6](#6-message-templates) | The five electronic-mail templates |
| [7](#7-the-text-message-template) | The one text-message template |
| [8](#8-the-message-subtype) | The subtype used to log responses |
| [9](#9-the-activity-type) | The meeting activity category |
| [10](#10-access-groups-and-access-rights) | Entity-level rights per group |
| [11](#11-record-rules) | Row-level rules |
| [12](#12-field-level-visibility) | Fields restricted to a group |
| [13](#13-scheduled-jobs) | The three scheduled jobs |
| [14](#14-settings-screens-and-preferences) | The settings block, the provider panel and the per-user preference |
| [15](#15-sequences) | Why the domain has none |
| [16](#16-demonstration-data) | What the demonstration set adds |

---

## 2. Capability packages

| Package | Technical name | Depends on | What installing it adds |
|---|---|---|---|
| Calendar | `calendar` | the platform foundation and messaging | Everything in [entities.md](entities.md) except the provider-specific entities: meetings, attendees, series, reminders, tags, side-list rows, the reminder engine, the two panels, the routes, the templates and the reminder job. It is an application and appears in the main menu. |
| Calendar - Text Message | `calendar_sms` | Calendar and text messaging | The `sms` reminder channel, the text-message template, the text-message reminder path in the reminder job, and the two text-message buttons. Installed automatically when both of its dependencies are present. |
| Google Users | `google_account` | the base setup package | The delegated-authorisation helper for the first external calendar service family and its callback route. Owned by [../automation-and-integration/](../automation-and-integration/). |
| Google Calendar | `google_calendar` | Google Users and Calendar | The synchronisation behaviour for the first service, its fields on meetings, series, users and user settings, its reset panel, its scheduled job, its settings controls and its synchronisation route. |
| Google Gmail | `google_gmail` | the mail server packages | Unrelated to calendars; it shares the same registration family. Owned by [../automation-and-integration/](../automation-and-integration/). |
| Display Working Hours in Calendar | `hr_calendar` | Human resources and Calendar | The working-schedule intersection, the unusual-day query, the working-hours shading and the "My Team" filter on contacts. Installed automatically when both dependencies are present. |
| Microsoft Users | `microsoft_account` | the base setup package | The delegated-authorisation helper for the second service family, its callback route and the stored redirect value. Owned by [../automation-and-integration/](../automation-and-integration/). |
| Outlook Calendar | `microsoft_calendar` | Microsoft Users and Calendar | The synchronisation behaviour for the second service and everything that mirrors the fourth row. Installing it also generates a unique platform marker, once, into a system parameter. |
| Microsoft Outlook | `microsoft_outlook` | the mail server packages | Unrelated to calendars. Owned by [../automation-and-integration/](../automation-and-integration/). |

---

## 3. System parameters

| Key | Shipped value | Read by | Effect |
|---|---|---|---|
| `calendar.default_privacy` | `public` | user creation and the default-privacy computation | The visibility applied to a new user's meetings that carry no policy of their own. |
| `calendar.max_recurrence_years` | not shipped; read as 15 | the occurrence generator | The horizon in years for an endless series. |
| `calendar.block_mail` | not shipped; read as absent | attendee notification | Any value that reads as true suppresses every attendee notification. |
| `mail.mail_force_send_limit` | not shipped; read as 100 | attendee notification and the reminder job | Above this many notified attendees, messages are queued instead of sent immediately. Owned by [../messaging-and-activities/](../messaging-and-activities/). |
| `google_calendar_client_id` | empty | the first service | Its application registration identifier. |
| `google_calendar_client_secret` | empty | the first service | Its secret. Never returned to a screen in clear. |
| `google_calendar_sync_paused` | not shipped; read as false | the first service | Suspends every exchange with the first service. |
| `google_calendar.sync.range_days` | not shipped; read as 365 | the first service | The half-width in days of the window a full run fetches, and of the outward scope. |
| `microsoft_calendar_client_id` | empty | the second service | Its application registration identifier. |
| `microsoft_calendar_client_secret` | empty | the second service | Its secret. |
| `microsoft_calendar_sync_paused` | not shipped; read as false | the second service | Suspends every exchange with the second service. |
| `microsoft_calendar.sync.range_days` | not shipped; read as 365 | the second service | The number of days a full run reaches back, and half the number it reaches forward. The outward scope uses it symmetrically. |
| `microsoft_calendar.sync.lower_bound_range` | not shipped | the second service | When set, expressed in days, it both narrows the outward scope's lower bound and enables the old-event guard of [business-rules.md](business-rules.md#7-external-synchronisation). |
| `microsoft_calendar.sync.first_synchronization_date` | not shipped | the second service | Written once, at the very first run on a platform that never synchronised, to the current instant less one minute. From then on the outward scope only contains meetings created at or after it. |
| `microsoft_calendar.graph_timeout` | not shipped; read as 5 | the second service | The request budget in seconds for calls made from a create or write operation. A value that is not a whole number is ignored; a value below one is raised to one. |
| `microsoft_calendar.microsoft_guid` | generated once at installation | the second service | A universally unique value identifying this platform to the service. |
| `microsoft_account.auth_endpoint` | not shipped; read as the default consent address | the second service | Overrides the consent address. |
| `microsoft_account.token_endpoint` | not shipped; read as the default credential address | the second service | Overrides the credential address. |
| `microsoft_redirect_uri` | `urn:ietf:wg:oauth:2.0:oob` | the second service family | The stored redirect value. |
| `database.uuid` | generated by the platform | both services | Carried inside the state value of the consent round trip. Owned by [../platform-foundation/](../platform-foundation/). |

---

## 4. The default meeting duration

The default length of a new meeting is not a system parameter. It is a default value in the
platform's default-values register, stored against the entity Calendar Event and the field
`duration`, and it may be set per user, per company, per user and company together, or globally. The
four lookups and their order are in
[calculations.md, section 2.2](calculations.md#22-the-default-duration). No default is shipped, so an
untouched platform uses one hour.

---

## 5. Shipped reminders

Seven Event Alarm records are shipped. They are shipped as non-updatable data, so an administrator may
edit them freely without an upgrade putting them back.

| External identifier | Name | Channel | Duration | Unit | Lead time in minutes | Template |
|---|---|---|---|---|---|---|
| `alarm_notif_1` | "Notification - 15 Minutes" | `notification` | 15 | `minutes` | 15 | — |
| `alarm_notif_2` | "Notification - 30 Minutes" | `notification` | 30 | `minutes` | 30 | — |
| `alarm_notif_3` | "Notification - 1 Hours" | `notification` | 1 | `hours` | 60 | — |
| `alarm_notif_4` | "Notification - 2 Hours" | `notification` | 2 | `hours` | 120 | — |
| `alarm_notif_5` | "Notification - 1 Days" | `notification` | 1 | `days` | 1440 | — |
| `alarm_mail_1` | "Email - 3 Hours" | `email` | 3 | `hours` | 180 | the shipped reminder template |
| `alarm_mail_2` | "Email - 6 Hours" | `email` | 6 | `hours` | 360 | the shipped reminder template |

No reminder is attached to a new meeting by default; a person or an inward synchronisation attaches
them.

---

## 6. Message templates

Five templates are shipped. All five carry the same sender expression, which is the organiser's
formatted address, falling back to the acting user's formatted address, falling back to an empty
value, which makes the platform use its own fallback sender. All five let the recipient be derived
from the record rather than listing addresses.

| External identifier | Name | Rendered against | Subject | Deleted after sending | Purpose |
|---|---|---|---|---|---|
| `calendar_template_meeting_invitation` | "Calendar: Meeting Invitation" | Calendar Attendee Information | "Invitation to " followed by the meeting subject | yes | "Invitation email to new attendees" |
| `calendar_template_meeting_changedate` | "Calendar: Date Updated" | Calendar Attendee Information | the meeting subject followed by ": Date updated" | yes | "Sent to all attendees if the schedule change" |
| `calendar_template_meeting_reminder` | "Calendar: Reminder" | Calendar Attendee Information | the meeting subject followed by " - Reminder" | yes | "Sent to all attendees if a reminder is set" |
| `calendar_template_meeting_update` | "Calendar: Event Update" | Calendar Event | the meeting subject followed by ": Event update" | no | "Used to manually notify attendees" |
| `calendar_template_delete_event` | "Calendar: Event Deleted" | Calendar Event | "Deleted event: " followed by the meeting subject | no | "Used to manually notify attendees" |

### 6.1 What the three attendee templates contain

All three are rendered against one invitation, so each recipient gets their own copy, in their own
language and with the times in their own time zone.

1. A heading: "Invitation", "Date Updated" or "Reminder", the last two followed by a small icon.
2. A greeting addressing the invitation's common name.
3. A sentence. The invitation template says who invited the recipient to which meeting, or, when the
   organiser's user record is archived, that a customer did; when the recipient is the organiser it
   says the meeting has been booked. The rescheduling template says the appointment has been updated.
   The reminder template says "This is a reminder for the event below."
4. A "View" button pointing at `/calendar/meeting/view` with the invitation token and the meeting
   identifier as query values.
5. A "Details" block holding: the date and time, written as the weekday, the day number, the month
   and year and, for a timed meeting, the short time followed by the time-zone name in parentheses;
   the repetition sentence under the label "When", when the meeting belongs to a series and the caller
   did not ask to ignore repetition; the duration under the label "Duration", written as the whole
   hours, the letter H and the remaining minutes in two digits, for a timed meeting with a non-zero
   duration; the location, with a map link, when there is one; and the video-call address under the
   label "Join with" when it is a platform-hosted call and "Join" otherwise.
6. An "Attendees" block listing every invitation's common name, each preceded by a small image whose
   name carries the response, with the recipient's own line reading "You".
7. A "Description of the event" block, when the description is not empty.
8. A closing "Thank you!" and the organiser's signature when they have one.

The reminder template is the one the reminder job uses, and the caller asks it to ignore repetition,
so the "When" line does not appear on a reminder.

### 6.2 What the two event templates contain

Both are rendered against the meeting rather than the invitation, so one message serves every
recipient, and the times are rendered in the time zone the caller forces.

The update template heads "Event updated", says "This meeting has been updated." and then repeats the
"Details" and "Attendees" blocks. The deletion template heads "Event canceled" and says "This is to
inform you that the event " followed by the meeting subject, then, when there is an organiser,
"organized by " and the organiser's name, then " has been canceled and removed from your calendar."

---

## 7. The text-message template

| External identifier | Name | Rendered against | Body |
|---|---|---|---|
| `sms_template_data_calendar_reminder` | "Calendar Event: Reminder" | Calendar Event | "Event reminder: " then the meeting subject, then ", " then the display time forced into the organiser's contact's time zone |

---

## 8. The message subtype

| External identifier | Name | Entity | Default | Purpose |
|---|---|---|---|---|
| `calendar.subtype_invitation` | "Invitation" | Calendar Event | not a default subscription | The subtype under which an acceptance or a decline is logged on the meeting, authored by the responding contact. Because it is not a default, followers do not receive those messages unless they subscribe to it. |

---

## 9. The activity type

The activity-type category gains the value `meeting` = "Meeting", and the platform's shipped "Meeting"
activity type is placed in that category. An activity of that category behaves differently on screen:
the deadline, the assignee, the summary and the note are hidden, a large calendar illustration and the
sentence "Schedule a meeting in your calendar" are shown, the ordinary scheduling and done buttons are
hidden, and a "Schedule" button appears that opens the calendar. The same changes are made in the
scheduling wizard.

---

## 10. Access groups and access rights

The domain defines no group of its own. It uses four platform groups: the internal-user group, the
portal group, the contact-management group and the settings-administration group.

| Entity | Group | Read | Write | Create | Delete |
|---|---|---|---|---|---|
| Calendar Event | portal | yes | no | no | no |
| Calendar Event | internal user | yes | yes | yes | yes |
| Calendar Event | contact management | yes | yes | yes | yes |
| Calendar Attendee Information | portal | no | no | no | no |
| Calendar Attendee Information | internal user | yes | yes | yes | yes |
| Event Alarm | internal user | yes | yes | yes | yes |
| Event Alarm Manager | internal user | yes | yes | yes | yes |
| Event Meeting Type | internal user | yes | no | no | no |
| Event Meeting Type | settings administration | yes | yes | yes | yes |
| Calendar Filters | internal user | yes | yes | yes | yes |
| Calendar Filters | settings administration | yes | yes | yes | yes |
| Event Recurrence Rule | internal user | yes | yes | yes | yes |
| Calendar Popover Delete Wizard | internal user | yes | yes | yes | yes |
| Calendar Provider Configuration Wizard | everybody | no | no | no | no |
| Calendar Provider Configuration Wizard | settings administration | yes | yes | yes | yes |
| Google Calendar Account Reset | settings administration | yes | yes | yes | no |
| Microsoft Calendar Account Reset | settings administration | yes | yes | yes | no |

A portal user therefore reads meetings but never attendee records; the invitation page reads them with
elevated rights instead.

---

## 11. Record rules

| External identifier | Name | Entity | Groups | Operations | Condition |
|---|---|---|---|---|---|
| `calendar_event_rule_my` | "Own events" | Calendar Event | portal | all four | The reader's own contact is among the meeting's attendee contacts. |
| `calendar_event_rule_employee` | "All Calendar Event for employees" | Calendar Event | internal user | all four | Unrestricted. Privacy is enforced by field obfuscation instead. |
| `calendar_attendee_rule_my` | "Own attendees" | Calendar Attendee Information | portal | all four | Unrestricted. The entity-level rights of chapter 10 already deny the portal group everything, so the rule has no effect on its own. |
| `calendar_event_rule_private` | "Private events" | Calendar Event | everybody | write, create and delete only | The visibility policy is not private, or it is private and either the reader is the organiser or the reader's contact is among the attendee contacts. |

The last rule deliberately does not apply to reading. Reading a private meeting succeeds and returns
obfuscated values, which is what lets a calendar show somebody else's private meeting as a "Busy"
block.

---

## 12. Field-level visibility

| Entity | Fields | Restricted to |
|---|---|---|
| User | `google_calendar_rtoken`, `google_calendar_token`, `google_calendar_token_validity`, `google_calendar_sync_token`, `google_calendar_cal_id`, `google_synchronization_stopped`, `microsoft_calendar_rtoken`, `microsoft_calendar_token`, `microsoft_calendar_sync_token`, `microsoft_synchronization_stopped`, `microsoft_last_sync_date` | the settings-administration group |
| User Settings | the same set, plus `google_calendar_cal_id` | the settings-administration group |
| Activity Mixin | `activity_calendar_event_id` | internal users |

A user may read and write their own `calendar_default_privacy` even though they are not an
administrator, because that field is added to the two self-service lists. Every credential field of
both services is excluded from the payload a client receives when a session opens.

---

## 13. Scheduled jobs

| External identifier | Name | Runs | Interval | Runs as | What it does |
|---|---|---|---|---|---|
| `ir_cron_scheduler_alarm` | "Calendar: Event Reminder" | on the Event Alarm Manager | every 1 day, and additionally whenever a planted wake-up fires | the platform's own automated user | Sends the electronic-mail reminders and, when the text-message package is installed, the text-message reminders, then plants the next wake-up for each series. |
| `ir_cron_sync_all_cals` of the first service | "Google Calendar: synchronization" | on the User entity | every 12 hours | the platform's own automated user | Runs a synchronisation for every user who holds a long-lived credential and has not stopped, ordered by the expiry of their short-lived credential. |
| `ir_cron_sync_all_cals` of the second service | "Outlook: synchronization" | on the User entity | every 12 hours | the platform's own automated user | The same for the second service. |

The reminder job is shipped active and is created even when a record with the same external identifier
is missing. Its last-call instant is the boundary the selection window of
[calculations.md, section 7.3](calculations.md#73-the-selection-window-of-the-reminder-job) uses.

The two synchronisation jobs iterate user by user and commit after each one, so that a failure for one
user neither loses the work of the previous users nor stops the following ones.

---

## 14. Settings screens and preferences

### 14.1 The calendar settings block

The general settings screen gains an application block titled "Calendar", visible to the
settings-administration group, holding two settings:

| Setting | Controls |
|---|---|
| "Outlook Calendar", helped by "Synchronize your calendar with Outlook" | The installation switch of the second service's package. When it is on and the page has not been saved, the warning "Save this page and come back here to set up the feature." is shown, with "Save" in bold. Once installed, the block is replaced by three controls: "Client ID", "Client Secret" masked, and "Pause Synchronization". |
| "Google Calendar", helped by "Synchronize your calendar with Google Calendar" | The same for the first service. |

### 14.2 The provider panel

A one-screen panel offering the two services as radio buttons laid out horizontally, and, for the
chosen one, its identifier labelled "Client ID", its secret labelled "Client Secret" and masked, and
its suspension switch, all three required when that service is chosen. The panel's footer holds the
connect control and a "Cancel" button. Its behaviour is CAL-204 to CAL-206.

### 14.3 Per-user preference

The preferences screen and the user screen both gain a field labelled "Privacy" next to the time zone,
hidden for a shared user, holding the calendar default privacy. A user may change their own; changing
anybody else's is refused by CAL-057.

### 14.4 The credentials block on a user

The user screen gains an empty container on its calendar page, into which each installed service adds
a block, visible to the settings-administration group and hidden for a shared user:

| Service | Fields shown, all read only | Button |
|---|---|---|
| first | the long-lived credential, the short-lived credential, its validity, the change marker and the last synchronised calendar | "Reset Account", opening the reset panel with the user pre-filled |
| second | the long-lived credential labelled "Refresh Token", the short-lived credential labelled "User Token", its validity labelled "Token Validity", the change marker labelled "Next Sync Token" and the last-synchronisation instant labelled "Last Sync Time", the last of which is writable | "Reset Account", opening the reset panel with the user pre-filled |

---

## 15. Sequences

The domain defines no numbering sequence. Meetings are identified by their subject and their time, not
by a reference. The only generated values are the invitation token and the meeting token, each a
thirty-two character hexadecimal value drawn at random, and the conferencing request identifier sent to
the first external calendar service, also a thirty-two character hexadecimal value.

---

## 16. Demonstration data

The Calendar package ships a demonstration set. It is loaded only when a platform is created with
demonstration data and is never part of an ordinary installation; it exists so that the calendar
screen is not empty on a freshly created demonstration platform. It contains sample meetings spread
around the installation date, with sample attendees drawn from the demonstration contacts. No
behaviour depends on it.
