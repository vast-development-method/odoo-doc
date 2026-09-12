# Glossary of Calendar and Scheduling

Every term this domain uses, defined. Terms are listed alphabetically. Where a term names a stored
value, an identifier or a user-visible label, that string is reproduced in code font or in quotation
marks beside the definition.

---

**Acknowledgement instant.** The instant at which a person last dismissed their in-application
reminders, stored on their Contact as `calendar_last_notif_ack`. Reminders whose firing instant is at
or before it are never offered again. See
[business-rules.md](business-rules.md#6-reminders), CAL-120.

**Activity of the meeting category.** An Activity whose type carries the category value `meeting`.
Such an activity does not show a deadline, an assignee, a summary or a note on screen; instead it
offers a "Schedule" button that opens the calendar. See
[configuration.md, chapter 9](configuration.md#9-the-activity-type).

**All-day meeting.** A Calendar Event whose `allday` marker is true. It occupies whole calendar days
rather than a span of clock time. Its authoritative fields are the start day and the stop day; its
start and stop instants carry a conventional eight and eighteen o'clock so that the meeting falls on
the same days for every reader.

**Attendee.** A Contact invited to a meeting. The invitation itself is a separate record, the
Calendar Attendee Information.

**Attendee contact set.** The many-to-many set `partner_ids` on Calendar Event. Writing it is what
creates and deletes the attendee records; the two are always kept in step.

**Availability marker.** The field `show_as` on Calendar Event, holding `free` ("Available") or `busy`
("Busy"). Only a meeting marked busy makes its attendees unavailable.

**Base occurrence.** The occurrence of a series whose values are copied when the series generates the
others, held in `base_event_id`. When it is archived or deleted, the series selects its earliest
remaining occurrence that is not an outlier.

**Calendar-scoped identifier.** The field `microsoft_id`, holding the identifier the second external
calendar service assigns to an event inside one particular calendar. It differs between the
organiser's copy and each attendee's copy, and every request path needs it.

**Calendar interchange file.** The machine-readable representation of a meeting, attached to every
attendee notification under the name `invitation.ics` with the media type `text/calendar`. Its
contents are specified in [interfaces.md, section 7.1](interfaces.md#71-the-calendar-interchange-attachment).

**Change marker.** The value a synchronisation run stores so that the next run fetches only what
changed since it: `google_calendar_sync_token` for the first service and
`microsoft_calendar_sync_token` for the second. A rejected marker forces a full run.

**Common name.** The computed label of an invitation: the invited Contact's name, or, when it has
none, its electronic mail address.

**Confidential.** The visibility value `confidential`, labelled "Only internal users". A confidential
meeting is readable by internal users and is not offered to portal users.

**Consent.** The round trip in which a person authorises the platform to reach their calendar at an
external service. The platform sends the person to the service's own consent address and receives an
authorisation code at its callback, which it exchanges for credentials. See
[workflows.md, chapter 16](workflows.md#16-connect-a-users-account).

**Contact paragraph.** The block prepended to a new meeting's description, holding the organiser's
name, address and telephone number under the heading "Organized by" and, when there is exactly one
other attendee, that contact's details under the heading "Contact Details".

**Coordinated universal time.** The reference time in which every timed instant of this domain is
stored. Conversion to a reader's own clock happens when a value is shown, never in storage.

**Declined.** The response value `declined`, labelled "No". A declined attendee receives no reminder
of any channel.

**Default privacy.** The visibility applied to a person's meetings that carry no policy of their own,
held as `calendar_default_privacy` on their User Settings and mirrored on their User. A new user takes
it from the system parameter `calendar.default_privacy`, whose shipped value is `public`.

**Detached occurrence.** An occurrence that the pattern no longer produces. Detaching empties its
series reference and sets its repetition marker. Detached occurrences that are also being written are
archived; the rest are deleted, or archived when a remote copy exists.

**Display time.** The sentence describing when a meeting happens, in the reader's own time zone, in
three forms depending on whether the meeting is all-day, shorter than a day, or longer. Its
construction is in [calculations.md, chapter 5](calculations.md#5-the-display-time).

**Duration.** The length of a meeting in decimal hours, held in `duration`, derived from the start and
the stop and rounded to two decimals, or supplied and used to derive the stop.

**Effective visibility.** The visibility actually applied to a meeting: its own policy when it has
one, and otherwise its organiser's default privacy. Held in `effective_privacy`, computed, not stored.

**End day.** The last day on which a series may place an occurrence, held in `until`, used when the
termination mode is `end_date`.

**Endless repetition.** The termination mode `forever`, labelled "Forever". It is nevertheless bounded
by the horizon and the hard cap; see [calculations.md, chapter 11](calculations.md#11-the-occurrence-horizon-and-its-caps).

**Event Alarm.** The reusable reminder definition: a channel, a lead time and a template. Meetings
point at reminders; a reminder points at no meeting.

**Event Meeting Type.** A free tag classifying meetings and giving them a colour, unique by name.

**Firing instant.** The instant at which a reminder is due: the occurrence's start less the reminder's
lead time in minutes.

**First-connection cut-off.** The system parameter
`microsoft_calendar.sync.first_synchronization_date`, set once at the very first run on a platform that
has never synchronised, to the current instant less one minute. Meetings created before it are never
pushed, which prevents a flood of invitations for meetings that pre-date the connection.

**First external calendar service.** The service whose stored selection value is `google` and whose
capability-package identifiers begin with `google_`. See
[external-calendar-synchronisation.md](external-calendar-synchronisation.md).

**Follow marker.** The field `follow_recurrence` on Calendar Event, true while the occurrence still
sits exactly where the pattern puts it. Lowering it makes the occurrence an outlier.

**Full run.** A synchronisation run made with no change marker, which fetches a bounded window of the
remote calendar rather than only the changes. A full run suppresses outward attendee notifications, so
that a first connection does not announce old meetings.

**Guest-modification restriction.** The field `guests_readonly`, set from the first external calendar
service, which forbids anybody but the organiser to change the meeting.

**Horizon.** The number of years an endless series may reach, read from the system parameter
`calendar.max_recurrence_years` and defaulting to fifteen.

**Immediate-sending threshold.** The number of notified attendees above which messages are queued
rather than sent at once, read from the platform parameter `mail.mail_force_send_limit` and defaulting
to one hundred.

**In-application reminder.** A reminder of the channel `notification`, delivered by the notification
bus to open screens and by the polling route, never by electronic mail.

**Interval.** How many periods separate two occurrences of a series, held in `interval`. It must be
strictly positive.

**Invitation token.** The unguessable value `access_token` on Calendar Attendee Information, generated
as a thirty-two character hexadecimal value, which authorises answering an invitation without signing
in.

**Inward.** The direction from an external calendar service into the platform.

**Lead time.** How long before the start a reminder fires, held as a duration and a unit and derived
into whole minutes in `duration_minutes`, which is the only unit any comparison uses.

**Linked document.** The business record a meeting was scheduled from, named by `res_model_id` and
`res_id` on Calendar Event, for example an opportunity, an applicant or a contact.

**Long-lived credential.** The value that lets the platform obtain fresh short-lived credentials
without asking the person again. Stored as `google_calendar_rtoken` and `microsoft_calendar_rtoken`
and visible only to the settings-administration group.

**Meeting.** A Calendar Event. Every occurrence of a repeating meeting is a meeting in its own right.

**Meeting token.** The unguessable value `access_token` on Calendar Event, used in the public
video-call address. Each occurrence of a series has its own, deliberately.

**Notification bus.** The platform mechanism by which the reminder engine pushes a message of the kind
`calendar.alarm` to a contact's channel. The payload is specified in
[interfaces.md, chapter 11](interfaces.md#11-the-notification-bus-contract).

**Occurrence.** One meeting of a series.

**Old-event guard.** The rule that a change reported by the second external calendar service on a
meeting older than a configured bound is applied only when the difference between the remote and the
local modification instants is at least one hour. The bound is
`microsoft_calendar.sync.lower_bound_range`, expressed in days.

**Organiser.** The user who owns a meeting, held in `user_id`. Their contact appears as "Scheduled by".
The organiser's connection is preferred for outward calls, and only the organiser may remove a meeting
from an external service.

**Organiser-alone marker.** The computed field `is_organizer_alone`, true when a meeting has more than
one attendee and every attendee other than the organiser has declined.

**Outlier.** An occurrence whose follow marker is false, because it was moved or edited on its own. An
outlier is excluded when the series looks for a reference occurrence and survives a regeneration.

**Outward.** The direction from the platform into an external calendar service.

**Outward scope.** The condition selecting which local records a given user is responsible for pushing.
For a meeting it is the meetings that user attends whose stop is after the lower bound and whose start
is before the upper bound, excluding occurrences that still follow their series.

**Pattern.** The repetition rule of a series: a frequency, an interval, a termination mode and the
parameters that go with them. It is stored twice, as separate fields and as the interchange text in
`rrule`, each derived from the other.

**Pattern sentence.** The readable description of a pattern stored as the series' name, for example
"Every 2 Weeks on Tuesday, Wednesday for 3 events". The fifteen forms are in
[calculations.md, chapter 18](calculations.md#18-the-pattern-sentence).

**Pending flag.** The field that marks a record as still owing a push to an external service:
`need_sync` for the first service and `need_sync_m` for the second. There is no separate outbox; the
flag is the queue.

**Period start.** The first instant of the period a pattern counts in, from which occurrences are
generated: the first day of the week for a weekly series, the first day of the month for a monthly
series, and the day itself otherwise.

**Portal user.** An external person with limited access. A portal user reads only the meetings they
attend, may not write them, and may not read attendee records at all.

**Position in the month.** For a monthly series counted by weekday, which occurrence of that weekday
is meant: `1` "First", `2` "Second", `3` "Third", `4` "Fourth" or `-1` "Last". The fourth and the fifth
occurrence of a weekday both map to "Last".

**Private.** The visibility value `private`, labelled "Private". A private meeting hides its subject
and its other sensitive fields from internal users who are neither its organiser nor its attendees, and
cannot be written by them.

**Public.** The visibility value `public`, labelled "Public". Every internal user may read every field.

**Registration identifier and secret.** The pair of values that identifies the platform to an external
service, stored as `google_calendar_client_id` and `google_calendar_client_secret` for the first
service and `microsoft_calendar_client_id` and `microsoft_calendar_client_secret` for the second.

**Reminder channel.** The delivery method of a reminder, held in `alarm_type`: `notification`
("Notification"), `email` ("Email") and, when the text-message package is installed, `sms` ("SMS Text
Message").

**Reminder job.** The scheduled job named "Calendar: Event Reminder" that runs once a day and whenever
a wake-up fires, and sends the electronic-mail and text-message reminders.

**Repetition marker.** The field `recurrency` on Calendar Event, true when the meeting belongs to a
series or is about to create one.

**Response.** The value `state` on Calendar Attendee Information: `needsAction` ("Needs Action"),
`accepted` ("Yes"), `declined` ("No") or `tentative` ("Maybe").

**Scope selector.** The transient field `recurrence_update` on Calendar Event, carried with a change to
say how far it reaches: `self_only` ("This event"), `future_events` ("This and following events") or
`all_events` ("All events").

**Second external calendar service.** The service whose stored selection value is `microsoft` and whose
capability-package identifiers begin with `microsoft_`.

**Series.** An Event Recurrence Rule together with the meetings it produced.

**Series head.** In the second external calendar service, the entry that carries the pattern of a
repeating event. Local records that belong to one carry its identifier in
`microsoft_recurrence_master_id`.

**Short-lived credential.** The value that authorises one burst of calls to an external service, with
its own expiry, stored as `google_calendar_token` and `microsoft_calendar_token`. It is refreshed from
the long-lived credential when it is within one minute of expiring.

**Side list.** The list of colleagues whose calendars a person overlays on their own. Each entry is a
Calendar Filters record holding the person, the colleague's contact and whether the colleague's box is
ticked.

**Suspension.** The platform-wide switch that stops every exchange with one external service, held in
`google_calendar_sync_paused` and `microsoft_calendar_sync_paused`. Local records keep raising their
pending flag while a service is suspended, so the backlog is replayed when it is lifted.

**Synchronisation run.** One pass of fetching remote changes and pushing local ones, for one user
against one service. The two runs are specified in
[workflows.md, chapters 17 and 18](workflows.md#17-a-synchronisation-run-against-the-first-service).

**Synchronised field.** A field whose change marks a record as owing a push. The two sets differ
between the services; they are listed in
[state-machines.md, chapters 7 and 8](state-machines.md#7-record-pending-synchronisation-flag-first-service).

**Systray meeting group.** The entry the domain inserts into the platform's activity indicator, titled
"Today's Meetings", holding the reader's meetings that fall inside the remainder of the reader's own
day.

**Tag.** See Event Meeting Type.

**Termination mode.** How a series ends, held in `end_type`: `count` ("Number of repetitions"),
`end_date` ("End date") or `forever` ("Forever").

**Time zone of a series.** The zone in which a pattern is evaluated, held in `event_tz`, so that a
meeting keeps its wall-clock time across a change of daylight saving time.

**Unavailable attendee.** An attendee counted as occupied during a meeting, either because another
meeting marked busy overlaps it or, when the working-hours package is installed, because the meeting
falls outside their working schedule.

**Universal identifier.** The field `ms_universal_event_id`, holding the identifier the second external
calendar service gives an event that is the same in every calendar. It is the reliable matching key.

**Video call.** A meeting held over a web address, held in `videocall_location`. The address may be
hosted by the platform's own conversation channel, typed by a person, or supplied by an external
service.

**Visibility policy.** See Private, Public, Confidential and Effective visibility. The field is
`privacy` on Calendar Event and may be empty, in which case the organiser's default applies.

**Wake-up.** A one-shot trigger planted on the reminder job so that it runs at a reminder's firing
instant rather than waiting for its daily schedule. A series keeps exactly one.

**Working schedule.** The pattern of working hours attached to an employee, or, failing that, to the
current company. Used to decide whether an attendee is available and to shade the calendar. Owned by
[../attendances-and-working-time/](../attendances-and-working-time/).

**Working-hours shading.** The grey overlay the calendar screen draws outside the hours common to every
selected attendee, built from the payload of
[interfaces.md, section 12.2](interfaces.md#122-the-working-hours-payload).
