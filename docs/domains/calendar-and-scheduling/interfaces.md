# Interfaces of Calendar and Scheduling

## 1. Overview

This document lists every way into and out of the domain: the menus a person navigates, the screens
they see, the operations those screens invoke, the addresses a browser or another system may call, the
documents the domain produces, and the payloads it pushes.

| Chapter | Content |
|---|---|
| [2](#2-menus) | Menus |
| [3](#3-screens) | Screens |
| [4](#4-named-operations) | Named operations |
| [5](#5-routes) | Routes |
| [6](#6-the-public-invitation-page) | The public invitation page |
| [7](#7-produced-documents) | Produced documents |
| [8](#8-message-templates-as-an-interface) | Message templates as an interface |
| [9](#9-external-integrations) | External integrations |
| [10](#10-import-and-export) | Import and export |
| [11](#11-the-notification-bus-contract) | The notification bus contract |
| [12](#12-client-data-contracts) | Client data contracts |

---

## 2. Menus

| External identifier | Label | Parent | Order | Visible to | Opens |
|---|---|---|---|---|---|
| `mail_menu_calendar` | "Calendar" | the main menu | 10 | internal users | nothing; it is the application entry |
| `calendar_event_menu` | "Calendar" | the application entry | 1 | internal users | the meetings action |
| `calendar_menu_config` | "Configuration" | the application entry | 40 | settings administration and the developer group | the meetings action |
| `menu_calendar_settings` | "Settings" | Configuration | 45 | settings administration | the calendar settings action |
| `calendar_submenu_reminders` | "Reminders" | Configuration | 50 | the developer group | the reminders action |
| `menu_calendar_configuration` | "Calendar" | the platform's technical menu | 30 | the developer group | nothing; it is a container |
| `menu_calendar_event_type` | the action's own label, "Meeting Types" | the technical container | — | the developer group | the meeting-types action |
| `menu_calendar_alarm` | the action's own label, "Calendar Alarm" | the technical container | — | the developer group | the reminders action |

**Compatibility finding.** The "Configuration" menu opens the meetings action rather than a
configuration screen. A corrected behaviour would give it no action, since it exists only to hold the
two entries below it. The observed behaviour is reproduced because a rebuild that changes it changes
what a reader sees when they click.

### 2.1 Actions

| External identifier | Label | Entity | Screens offered | Notes |
|---|---|---|---|---|
| `action_calendar_event` | "Meetings" | Calendar Event | calendar, list, form | Reached at the path `calendar`. Opens on the calendar screen. Its empty-state text reads "No meetings found. Let's schedule one!" followed by "The calendar is shared between employees and fully integrated with other applications such as the employee leaves or the business opportunities." The screen order is fixed by three ordering records: calendar first, list second, form third. |
| `action_calendar_alarm` | "Calendar Alarm" | Event Alarm | list, form | — |
| `action_calendar_event_type` | "Meeting Types" | Event Meeting Type | list | — |
| `calendar_settings_action` | "Settings" | Config Settings | form | Opens the settings screen scrolled to the calendar block. |
| `action_event_delete_wizard` | "Event Cancel Wizard" | Calendar Popover Delete Wizard | form, in a dialog | The confirmation panel of the deletion procedure. |
| the first service's reset action | untitled | Google Calendar Account Reset | form, in a dialog | Opened from the user screen. |
| the second service's reset action | untitled | Microsoft Calendar Account Reset | form, in a dialog | Opened from the user screen. |

---

## 3. Screens

### 3.1 The calendar screen

The default screen of the meetings action. Its start field is the start, its stop field is the stop,
its length field is the duration and its all-day field is the all-day marker. Events open in a popover
rather than a full screen, at most five are shown per day before a "more" link appears, and quick
creation is enabled and uses the quick-creation screen. Meetings are coloured by attendee contact.

The popover shows, each with its own icon and only when filled: the location with a map pin; the
attendee contacts, as an expandable list of avatars, which doubles as the side-list filter and writes
its ticks into Calendar Filters through the contact and ticked fields; the video-call address as a
copyable link labelled "Join Video Call"; the linked document's entity name as a link that opens the
document; the reminders with a bell; the tags with their colours; the organiser's contact; and the
description.

Six further fields are loaded invisibly because the client needs them: the attendee count, the
accepted count, the declined count, the may-edit marker, the highlight marker, the visibility policy,
the effective visibility, the repetition marker and the scope selector.

When the working-hours package is installed the screen also shades unusual days and passes the
contact search screen that offers the "My Team" filter. When a synchronisation package is installed
the screen also loads that service's identifier invisibly, so that the client can tell a synchronised
meeting from a local one.

### 3.2 The meeting form

A full screen with a header, a body and a discussion thread.

| Region | Content |
|---|---|
| Header | A "Send email" button, hidden when the reader may not edit. |
| Banner | When the meeting belongs to a series, an information banner reading "Edit recurring event" together with the scope selector as radio buttons. |
| Button box | A button that opens the linked document, showing that document's entity name, hidden when there is none. |
| Ribbon | An "Archived" ribbon when the meeting is hidden. |
| Title | The subject, with the placeholder "e.g. Business Lunch". |
| Left column | The start and stop as a range, in two variants depending on the all-day marker; the duration in hours with the word "hours", read only for a saved repeating meeting; the all-day switch; the location with the placeholder "Online Meeting"; the video-call address as a copyable value with three controls — a cross that clears it, an arrow that opens it, and a control that creates a platform-hosted call; and the linked document, read only. |
| Right column | The reader's own response as a badge selector labelled "Going?", shown only when it is worth showing; the availability marker and the visibility policy side by side under the label "Status", the second with the placeholder "User default" and the help text "Set whether you're available for other simultaneous events and the privacy of this event to others. Manage your default privacy in your user preferences."; the attendee count with the word "guests" and the four response counters, each shown only when non-zero, labelled "Yes", "Maybe", "No" and "Awaiting"; an envelope button that opens the composer; and the attendee contacts through the attendee widget, which highlights invalid addresses and unavailable attendees. |
| Notes page | The internal notes, with the placeholder "Add notes about this meeting…". |
| Options page | The organiser; the tags; the description under the label "Calendar description" with the placeholder "Add description"; the reminders; the repetition switch with the word "Repeat:" and the simplified frequency; for a custom repetition, the interval labelled "Repeat every" and the frequency; for a weekly repetition, the seven weekday toggles under "Repeat on"; for a monthly repetition, the monthly mode under "Day of Month" together with either the day number or the position and the weekday; the termination mode under "Until" together with either the count or the end day, the latter with the placeholder "e.g: 12/31/2023"; and the time zone. |
| Invitations page | Visible to the developer group and only when the reader may edit. A "Send Invitations" button and the list of invitations showing the contact, the address, the telephone number, the response, and three buttons — "Uncertain", "Accept" and "Decline" — each hidden when the invitation already holds that answer. |
| Footer | The discussion thread. |

### 3.3 The quick-creation screen

A compact panel with an icon per line: the subject with the placeholder "Add Title"; the dates as a
range with an all-day switch; the attendee contacts as tags with the placeholder "Participants"; the
location with the placeholder "Location" and a control that adds a video call; the video-call address
as a copyable value with a control that removes it; the visibility policy with the placeholder
"Visibility"; and the notes with the placeholder "Notes".

### 3.4 The meeting list

Columns: the subject, the start labelled "Start Date", the stop labelled "End Date", the organiser,
the attendee contacts, the reminders, the tags, the repetition marker, the visibility policy, the
availability marker, the location, the duration as a time, and the description. Several are hidden by
default. Every column except the dates and the duration becomes read only when the meeting repeats,
because a series may not be edited from a list. Its header carries a "Send Mail" button that opens the
composer in mass mode; the text-message package adds a "Send SMS" button beside it.

### 3.5 The meeting search screen

| Kind | Name | Effect |
|---|---|---|
| field | the subject, matched loosely | — |
| field | the attendee contacts | — |
| field | the organiser | — |
| field | the location | — |
| field | the availability marker | — |
| field | the tags | — |
| field | the description | — |
| filter | `mymeetings`, "My Meetings" | Meetings whose attendee contacts are linked to the reader. |
| filter | `filter_start_date`, "Date" | The platform's date filter on the start. |
| filter | `busy`, "Busy" | The availability marker is busy. |
| filter | `free`, "Free" | The availability marker is free. |
| filter | `default`, "Default Privacy" | No visibility policy is set. |
| filter | `public`, "Public" | The policy is public. |
| filter | `private`, "Private" | The policy is private. |
| filter | `confidential`, "Only Internal Users" | The policy is confidential. |
| filter | `recurrent`, "Recurrent" | The meeting repeats. |
| filter | `inactive`, "Archived" | The meeting is hidden. |
| grouping | `responsible`, "Responsible" | By organiser. |
| grouping | `group_by_start`, "Date" | By start. |

### 3.6 The reminder screens

The list shows the channel, the duration and the unit. The form shows the channel; the electronic-mail
template, shown and required only for that channel; the additional message, shown only for the
in-application channel; the duration and the unit side by side; and, added by the text-message
package, the organiser flag, shown only for the text-message channel, and the text-message template.

### 3.7 The tag list

A single editable column holding the tag.

### 3.8 The two deletion panels

The first is titled "Delete Event" and shows the scope as radio buttons, with a "Submit" button and a
"Cancel" button. The second is also titled "Delete Event" and shows a warning reading "Are you sure
you want to delete this event?", the recipients as address tags, the subject, the body, and three
buttons: "Send and delete", "Delete" and "Discard".

### 3.9 The two reset panels

Each is titled "Reset Google Calendar Account" or "Reset Outlook Calendar Account" and shows the
deletion policy and the next-run policy as radio buttons, with a "Confirm" button and a "Cancel"
button.

### 3.10 Screens this domain changes elsewhere

| Screen | Change |
|---|---|
| The contact form | A statistics button showing the meeting count with a calendar icon and the label "Meetings", which opens the meetings action filtered to that contact's meetings and pre-filled with that contact as an attendee. |
| The contact search screen | The working-hours package adds a "My Team" filter matching contacts whose employees report to the reader. |
| The user form | A container on the calendar page into which each installed service adds its credential block and reset button; and the calendar default privacy next to the time zone. |
| The preferences screen | The calendar default privacy next to the time zone, labelled "Privacy" and hidden for a shared user. |
| The activity dialog | For an activity of the meeting category, the deadline, the assignee, the summary and the note are hidden, an illustration and the sentence "Schedule a meeting in your calendar" appear, and a "Schedule" button opens the calendar. |
| The activity scheduling wizard | The same changes and the same "Schedule" button. |
| The general settings screen | The calendar block of [configuration.md, section 14.1](configuration.md#141-the-calendar-settings-block). |

---

## 4. Named operations

### 4.1 On Calendar Event

| Operation | Arguments | Effect |
|---|---|---|
| `get_state_selections` | none | Returns the four response values and their labels, so the client can render the response control. |
| `get_default_duration` | none | Returns the default length in hours. |
| `action_open_calendar_event` | none | Returns the action that opens the linked document, or nothing when there is none. |
| `action_sendmail` | none | Sends the invitation template to the attendees, when the acting user has an address. |
| `action_open_composer` | none | Opens the composer titled "Contact Attendees" against this meeting, pre-filled with the update template, with the attendee contacts as recipients and the reader's time zone forced. Refuses with CAL-089 when there are no attendees. |
| `action_join_video_call` | none | Returns an action that opens the video-call address in a new window. |
| `action_join_meeting` | a contact identifier | Adds that contact to the attendee contacts when it is not already there. |
| `action_unlink_event` | an invitation identifier and a repetition marker | Deletes at once and returns to `/app/calendar` when the organiser has a live external connection or several meetings are addressed; otherwise opens the confirmation panel. |
| `action_mass_deletion` | a scope | Deletes according to the scope. |
| `action_mass_archive` | a scope | Archives according to the scope. |
| `change_attendee_status` | an answer and a scope | Changes the reader's own response over that scope. |
| `set_discuss_videocall_location` | none | Intercepted by the client, which sets the address locally; the operation itself only reports success. |
| `clear_videocall_location` | none | The same, for clearing. |
| `get_discuss_videocall_location` | none | Returns a fresh platform-hosted address without touching any record. |
| `get_display_time_tz` | a time zone | Returns the display time forced into that zone. Called from message templates so that they need no elevated rights. |
| `find_partner_customer` | none | Returns the first attendee contact that is not the organiser's own contact. |
| `get_next_alarm_date` | the reminders selected by a run | Returns the instant of the next wake-up for this occurrence's series. |
| `action_send_sms` | none | Opens the text-message composer in mass mode against the attendee contacts, keeping a log. Refuses with CAL-090 when there are no attendees. Added by the text-message package. |
| `get_unusual_days` | a first day and an optional last day | Returns the days in that window that are not working days for the reader. Added by the working-hours package. |

### 4.2 On Calendar Attendee Information

| Operation | Effect |
|---|---|
| `do_tentative` | Sets the response to "Maybe" and tells the two services. |
| `do_accept` | Sets it to "Yes", posts the acceptance message and tells the two services. |
| `do_decline` | Sets it to "No", posts the decline message and tells the two services. |

### 4.3 On Event Alarm Manager

| Operation | Effect |
|---|---|
| `get_next_notif` | Returns the payloads of chapter 11 for the reader. |
| `do_notif_reminder` | Turns one due reminder into one payload. |

### 4.4 On Contact

| Operation | Effect |
|---|---|
| `schedule_meeting` | Returns the meetings action, pre-filled with this contact and the reader as attendees, restricted to the meetings of this contact and its children or to meetings this contact attends. |
| `get_attendee_detail` | Returns the payload of [section 12.1](#121-the-attendee-detail-payload). |
| `get_working_hours_for_all_attendees` | Returns the payload of [section 12.2](#122-the-working-hours-payload). Added by the working-hours package. |

### 4.5 On User

| Operation | Effect |
|---|---|
| `check_calendar_credentials` | Returns, per installed service, whether both registration parameters are filled. |
| `check_synchronization_status` | Returns, per installed service, one of `missing_credentials`, `sync_active`, `sync_paused` and `sync_stopped`. |
| `get_selected_calendars_partner_ids` | Returns the contacts ticked in the reader's side list, optionally including the reader's own contact. |
| `is_google_calendar_synced` | True when the reader holds a short-lived credential for the first service and their derived state is active. |
| `stop_google_synchronization`, `restart_google_synchronization`, `pause_google_synchronization`, `unpause_google_synchronization` | The four transitions of [state-machines.md, chapter 5](state-machines.md#5-per-user-synchronisation-state-first-external-calendar-service). |
| `stop_microsoft_synchronization`, `restart_microsoft_synchronization`, `pause_microsoft_synchronization`, `unpause_microsoft_synchronization` | The same for the second service. |

### 4.6 On the other entities

| Entity | Operation | Effect |
|---|---|---|
| Calendar Filters | `unlink_from_partner_id` | Deletes every side-list row pointing at a contact. |
| Calendar Popover Delete Wizard | `close`, `action_delete`, `action_send_mail_and_delete` | [entities.md, section 9.3](entities.md#93-operations). |
| Calendar Provider Configuration Wizard | `action_calendar_prepare_external_provider_sync` | [entities.md, section 10.3](entities.md#103-operation). |
| Google Calendar Account Reset | `reset_account` | [workflows.md, section 19.1](workflows.md#191-first-external-calendar-service). |
| Microsoft Calendar Account Reset | `reset_account` | [workflows.md, section 19.2](workflows.md#192-second-external-calendar-service). |
| Activity | `action_create_calendar_event` | Opens the calendar pre-filled from the activity. |
| Activity | `unlink_w_meeting` | Deletes the activity and then its meeting. |
| Activity Schedule Wizard | `action_create_calendar_event` | Schedules the activity and then opens the calendar. Refuses with CAL-183 on more than one document. |

---

## 5. Routes

| Path | Kind | Authentication | Arguments | Answer |
|---|---|---|---|---|
| `/calendar/meeting/accept` | browser request | invitation token | the token and the meeting identifier | Accepts every matching invitation that is not already accepted, then renders the view route. |
| `/calendar/meeting/decline` | browser request | invitation token | the same | Declines every matching invitation that is not already declined, then renders the view route. |
| `/calendar/recurrence/accept` | browser request | invitation token | the same | Accepts the same contact's invitations on every occurrence of the series, then renders the view route. |
| `/calendar/recurrence/decline` | browser request | invitation token | the same | The mirror. |
| `/calendar/meeting/view` | browser request | invitation token | the token and the meeting identifier | For an internal user, a redirect to `/app/calendar.event/` followed by the identifier with the database name as a query value. For anybody else, the page of chapter 6. Answers "not found" when the token and the identifier do not match one invitation. |
| `/calendar/meeting/join` | browser request | signed-in user, rendered as a site page | a meeting token | Adds the reader's contact as an attendee when it is absent, then redirects to the view route with that reader's own invitation token. Answers "not found" when no meeting carries the token. |
| `/calendar/join_videocall/<string:access_token>` | browser request | public | the meeting token in the path | Creates the conversation channel when the meeting has none and redirects to the channel's invitation address. Answers "not found" when no meeting carries the token. |
| `/calendar/notify` | remote procedure call | signed-in user | none | The payloads of chapter 11. |
| `/calendar/notify_ack` | remote procedure call | signed-in user | none | Writes the acknowledgement instant onto the reader's contact. |
| `/calendar/check_credentials` | remote procedure call | signed-in user | none | Per installed service, whether its registration is complete. |
| `/google_calendar/sync_data` | remote procedure call | signed-in user | an entity name, and optionally the address to return to | For the entity Calendar Event, the connection procedure of [workflows.md, chapter 16](workflows.md#16-connect-a-users-account) and then a run. The answer holds a status and an address, and, for a missing registration, an action identifier. The statuses are `need_config_from_admin`, `need_auth`, `need_refresh`, `no_new_event_from_google`, `sync_paused`, `sync_stopped` and, for any other entity, `success`. |
| `/microsoft_calendar/sync_data` | remote procedure call | signed-in user | the same | The same, with the status `no_new_event_from_microsoft` in place of the fourth. |
| `/google_account/authentication` | browser request | public | the consent result | The callback of the first service family. Owned by [../automation-and-integration/](../automation-and-integration/). |
| `/microsoft_account/authentication` | browser request | public | the consent result | The callback of the second service family. Owned by the same folder. |

The authentication method named `calendar` is defined by this domain and is specified in
[business-rules.md, chapter 9](business-rules.md#9-public-access-to-an-invitation).

---

## 6. The public invitation page

The page is rendered in the invitation contact's own language, with the times in that contact's own
time zone, and is not lazy, because the rendering must finish before the database connection closes.

| Region | Content |
|---|---|
| Header | The company's logotype, taken from the organiser's company or, when there is no organiser, from the creator's company. |
| Title | "Calendar Invitation" followed by the meeting subject in smaller type. |
| Buttons | "Accept" and "Decline", each shown only when the invitation does not already hold that answer, each submitting to its route with the query values preserved, each carrying the platform's request-forgery token. |
| Badge | Shown when the invitation has been answered: "Yes I'm going." for accepted, "No I'm not going." for declined, "Tentative" for a maybe, and "No feedback yet" for an unanswered invitation. |
| Table | Five rows: "Invitation for", holding the common name and the address in parentheses; "Date", holding the display time; "Location", holding the location or a hyphen; "Attendees", holding one list entry per invitation with a style that carries the response; and "Description", holding the description or a hyphen. |

---

## 7. Produced documents

The domain produces no printable document. It produces one machine-readable document.

### 7.1 The calendar interchange attachment

Produced whenever an attendee notification is sent, one per meeting, named `invitation.ics`, of the
media type `text/calendar`, attached to the composer entity with the document identifier zero so that
it is not filed against the meeting.

| Property | Value |
|---|---|
| creation instant | The current instant, marked as coordinated universal time. |
| start | The start, marked as coordinated universal time for a timed meeting and left as a plain value for an all-day meeting. |
| end | The stop, marked the same way. |
| summary | The meeting subject, or an empty text. |
| description | The description cleaned of unsafe markup and then flattened to plain text; omitted when that is empty. |
| location | The location; omitted when empty. |
| address | The video-call address; omitted when empty. Present in addition to the location when both are set. |
| repetition rule | The last line of the pattern text with any leading rule marker removed; omitted when the meeting does not repeat. |
| alarm blocks | One per reminder, each with a trigger relative to the start whose value is the negative of the reminder's own duration in its own unit, and a description holding the reminder's name or, when it has none, the platform's own name. |
| attendee lines | One per invitation, holding a mail address reference built from the invitation's address, or from an empty text when it has none. |
| organiser line | The organiser's contact address, present only when that contact has an address, with the contact's display name as its common-name parameter, with double quotation marks replaced by single ones. |

Producing the document for a meeting with no start or no stop is refused with CAL-084. When the
platform's calendar-interchange library is unavailable, no attachment is produced at all and
notifications are sent without one.

---

## 8. Message templates as an interface

The five electronic-mail templates and the one text-message template of
[configuration.md, chapters 6 and 7](configuration.md#6-message-templates) are part of the domain's
observable surface: their subjects, their headings and their block labels are what a recipient sees.
Three of them are rendered once per invitation, so each recipient receives a copy in their own
language and time zone; two are rendered once per meeting.

The invitation, rescheduling and reminder templates all embed a link to the view route carrying the
invitation token and the meeting identifier. That link is the entry point of the whole public
invitation flow.

---

## 9. External integrations

The two external calendar services are specified in
[external-calendar-synchronisation.md](external-calendar-synchronisation.md), which lists the
transport addresses, the value maps in both directions, the identity model, the error handling and the
differences between the two services.

Two further integrations touch this domain from outside:

- **Messaging.** Every notification travels through the platform's messaging layer; see
  [../messaging-and-activities/](../messaging-and-activities/) and
  [../../runtime/mail-gateway.md](../../runtime/mail-gateway.md).
- **The notification bus.** Reminder countdowns are pushed over the platform's bus; see
  [../../runtime/notification-bus.md](../../runtime/notification-bus.md) and chapter 11 below.

---

## 10. Import and export

The domain adds no import or export format of its own. Its entities are importable and exportable
through the platform's generic facilities described in
[../../data/data-loading-and-exchange.md](../../data/data-loading-and-exchange.md), with three
cautions that follow from the rules of this domain:

1. Importing meetings with pattern fields and the scope "This event" is refused by CAL-020. A bulk
   import must either set the repetition marker in the same row or leave the pattern fields out.
2. Importing attendee records directly bypasses the derivation of the attendee contacts, so the
   waiting counter can go wrong; see the compatibility finding in
   [calculations.md, chapter 15](calculations.md#15-attendee-counters).
3. Importing meetings while an external calendar service is connected pushes every one of them
   outwards. Stopping synchronisation for the importing user first, and restarting it afterwards, is
   the way to control that.

---

## 11. The notification bus contract

The reminder engine pushes a message of the kind `calendar.alarm` to one contact's channel. The
payload is a list; each entry describes one due in-application reminder.

| Key | Value |
|---|---|
| `alarm_id` | The identifier of the reminder record. |
| `event_id` | The identifier of the meeting. |
| `title` | The meeting subject. |
| `message` | The meeting's display time, followed, when the reminder carries an additional message, by that message turned into a paragraph of markup. |
| `timer` | The number of seconds from now to the firing instant, computed as the seconds part of the difference plus its days part multiplied by 86 400. |
| `notify_at` | The firing instant as text. |

**Worked example.** The current instant is a Monday at 12:00:00. A meeting titled "Doom's day" starts
at 12:50:00 and carries a thirty-minute in-application reminder, so the firing instant is 12:20:00. The
payload holds the reminder's identifier, the meeting's identifier, the title "Doom's day", the
meeting's display time as the message, a timer of 20 × 60 = **1200** seconds, and a firing instant of
that Monday at 12:20:00.

The push is addressed to every non-shared user among the contacts concerned, each in their own
company selection.

---

## 12. Client data contracts

### 12.1 The attendee detail payload

Returned for a set of meetings and a set of contacts; one entry per invitation whose contact is in the
set.

| Key | Value |
|---|---|
| `id` | The contact's identifier. |
| `name` | The contact's display name. |
| `status` | The response. |
| `event_id` | The meeting's identifier. |
| `attendee_id` | The invitation's identifier. |
| `is_alone` | True when the meeting's organiser-alone marker is set and the reader is both the organiser and this contact. |
| `is_organizer` | 1 when this contact is the organiser's own contact, 0 otherwise. The client sorts on this key. |

### 12.2 The working-hours payload

Returned for a set of attendee contacts and a window; one entry per interval of the intersection of
their schedules.

| Key | Value |
|---|---|
| `daysOfWeek` | A one-element list holding the weekday number of [calculations.md, section 17.4](calculations.md#174-the-working-hours-shown-behind-the-calendar). |
| `startTime` | The interval's start in the reader's time zone, as hours and minutes separated by a colon. |
| `endTime` | The interval's stop, the same way. |

When the intersection is empty a single entry is returned with the weekday number 7 and both times
00:00, which shades the whole week.

### 12.3 The systray meeting group

The platform's activity indicator gains a group for today's meetings, inserted first.

| Key | Value |
|---|---|
| `id` | The identifier of the Calendar Event entity definition. |
| `type` | `meeting`. |
| `name` | "Today's Meetings". |
| `model` | `calendar.event`. |
| `icon` | The icon of the package that defines the entity. |
| `domain` | A condition matching both hidden and visible meetings. |
| `meetings` | One entry per meeting, holding its identifier, its start, its subject and its all-day marker, ordered by start. |
| `view_type` | `calendar`. |

The meetings shown are those that satisfy all of the following: the reader has an invitation that is
not declined; and either the meeting starts or stops at or after the current instant while starting at
or before the end of the reader's own day, or the meeting is all-day and its start day is the reader's
own today. Both bounds are computed in the reader's time zone and then converted, so a reader far from
coordinated universal time does not see tomorrow's meetings.

**Worked example.** The reader's time zone is one hour ahead. A meeting runs from 18:00 to 19:00 in
coordinated universal time on 15 November 2023, that is 19:00 to 20:00 locally. At 17:30, 18:00, 18:30
and 19:00 in coordinated universal time the meeting is listed; at 19:30 it is not, because its stop has
passed. A reader five hours behind, with a meeting from 21:00 to 22:00 in coordinated universal time
on 16 November, sees nothing at 19:00 on 15 November, because the meeting starts after the end of that
reader's 15 November.
