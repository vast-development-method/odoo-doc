# Business rules of Calendar and Scheduling

## 1. How rules are identified

Every rule in this document carries a stable identifier of the form `CAL-nnn`. The identifier never
changes once assigned, so acceptance scenarios, tests and support procedures can refer to it. Rules
are grouped by subject; the numbering is contiguous inside the document and the full index is in
[chapter 12](#12-rule-index).

A rule states three things: the condition under which it fires, what the system does, and, when the
system refuses, the exact text it shows. Placeholders inside a quoted message are described in
words immediately after the message. Quoted messages are reproduced character for character,
including any spelling the system itself uses; where the reproduced text is defective this is
recorded as a **compatibility finding**.

---

## 2. Timing and duration

### CAL-001 A meeting must have a subject

Condition: the subject is empty when the record is written.
Effect: the write is refused by the platform's required-field check.
Message: the platform's own required-field message, naming the field "Meeting Subject".

### CAL-002 A meeting must have a start and a stop

Condition: the start or the stop is empty when the record is written.
Effect: refused by the platform's required-field check. Both fields carry defaults, so this can only
happen when a caller writes an empty value explicitly.

### CAL-003 A timed meeting may not end before it starts

Condition: the meeting is not all-day, both the start and the stop are set, and the stop is strictly
earlier than the start.
Effect: the write is refused.
Message: "The ending date and time cannot be earlier than the starting date and time.\nMeeting
“%(name)s” starts at %(start_time)s and ends at %(end_time)s". The first placeholder is the meeting
subject, shown between typographic double quotation marks; the second is the start instant; the
third is the stop instant.

### CAL-004 An all-day meeting may not end before it starts

Condition: the meeting is all-day, both the start day and the stop day are set, and the stop day is
strictly earlier than the start day.
Effect: the write is refused.
Message: "The ending date cannot be earlier than the starting date.\nMeeting “%(name)s” starts on
%(start_date)s and ends on %(end_date)s". The placeholders are the subject, the start day and the
stop day.

### CAL-005 The stop follows from the start and the duration

Condition: the start or the duration changes and the stop is not written in the same operation.
Effect: the stop becomes the start plus the duration in hours multiplied by sixty and rounded to the
nearest whole minute; when the duration is empty, one hour is used. For an all-day meeting one
second is then subtracted, so that the meeting ends at the last second of its final day rather than
at the first instant of the next one. The arithmetic is worked in
[calculations.md, chapter 3](calculations.md#3-duration-and-stop).

### CAL-006 The duration follows from the start and the stop

Condition: the start or the stop changes and the duration is not written in the same operation.
Effect: the duration becomes the number of hours between the two, rounded to two decimals. A missing
start or stop yields zero.

### CAL-007 The default start is the next half hour

Condition: a meeting is created without a start.
Effect: the start is the current instant advanced to the next multiple of thirty minutes; an instant
already on a multiple is left alone.

### CAL-008 The default stop is the default start plus the default duration

Condition: a meeting is created without a stop.
Effect: the stop is the default start plus the default duration in hours. The default duration is
resolved in four steps, the first that yields a value winning: the reader's own default for the
current company; the reader's own default regardless of company; the current company's default; the
platform-wide default. When none is set, one hour is used.

### CAL-009 All-day days and instants are kept consistent

Condition: the all-day marker, the start or the stop changes.
Effect: for an all-day meeting the start day becomes the day part of the start and the stop day the
day part of the stop; for a timed meeting both days are emptied. Writing the two days on an all-day
meeting writes back a start at eight o'clock and a stop at eighteen o'clock on those days.

### CAL-010 Setting the two days on an unsaved all-day meeting fills the instants

Condition: on screen, both the start day and the stop day are set.
Effect: the start becomes that start day at eight o'clock and the stop that stop day at eighteen
o'clock. This is needed because the two day fields feed no other derivation, so nothing else would
fill the instants.

### CAL-011 An all-day meeting reads its dates as calendar dates, not as instants

Condition: any reading of an all-day meeting.
Effect: the start and stop of an all-day meeting are deliberately not held in coordinated universal
time. They carry the conventional eight and eighteen o'clock so that the meeting occupies the same
calendar days for every reader, wherever they are. This is a **compatibility finding**: the same two
fields carry a different meaning depending on the all-day marker. A corrected behaviour would hold
the two day fields as the only truth for an all-day meeting and derive the instants for display
only; the specification records the observed behaviour because integrations read the instants.

---

## 3. Recurrence

### CAL-020 A pattern field may not be written with the scope "This event"

Condition: at least one pattern field is written, and neither the scope is "This and following
events" or "All events" on exactly one occurrence of a series, nor the recurrent marker is being set
to true in the same operation.
Effect: the write is refused.
Message: "Unable to save the recurrence with \"This Event\"". The inner quotation marks are part of
the message.
The pattern fields are: the position inside the month, the end day, the frequency, the monthly mode,
the time zone, the pattern text, the interval, the count, the termination mode, the seven weekday
markers, the day number and the weekday.

### CAL-021 A change with the scope "All events" or "This and following events" only applies to one occurrence at a time

Condition: a scope other than "This event" is supplied while more than one event is addressed.
Effect: the scope is ignored and each addressed event is written directly.

### CAL-022 Trimming a series at its own head is the same as replacing the series

Condition: the scope is "This and following events" and the addressed occurrence is the base
occurrence of its series.
Effect: the system behaves as if the scope were "All events".

### CAL-023 A recurrence cannot be updated without a base occurrence

Condition: a time shift is computed for a series whose base occurrence is missing.
Effect: the operation is refused.
Message: "You can't update a recurrence without base event."

### CAL-024 The interval must be strictly positive

Condition: the pattern text is rebuilt while the interval is zero or negative.
Effect: the operation is refused.
Message: "The interval cannot be negative."

### CAL-025 The number of repetitions must be strictly positive

Condition: the pattern text is rebuilt while the termination mode is by count and the count is zero
or negative.
Effect: the operation is refused.
Message: "The number of repetitions cannot be negative."

### CAL-026 A weekly series must name at least one weekday

Condition: the occurrences of a weekly series are generated and none of the seven weekday markers is
set.
Effect: the operation is refused.
Message: "You have to choose at least one day in the week"

### CAL-027 A monthly series counted by number uses a day between 1 and 31

Condition: the frequency is monthly, the monthly mode is by number, and the day number is outside
the range 1 to 31 while the weekday and position are not both valid.
Effect: the database constraint `_month_day` refuses the write.
Message: "The day must be between 1 and 31"

### CAL-028 A day number that does not exist in a month simply produces no occurrence there

Condition: a monthly series counted by number uses a day number that some months do not have.
Effect: those months are skipped; the count is honoured across the months that do have that day. A
series of three occurrences on the thirty-first starting in October 2019 therefore falls on 31
October 2019, 31 December 2019 and 31 January 2020, skipping November.

### CAL-029 The occurrences of a series are real records

Condition: any series exists.
Effect: every occurrence is a Calendar Event with its own identifier, attendees, reminders, video
call address and responses. Nothing is computed at read time. Applying a series creates the missing
occurrences and detaches the ones the pattern no longer produces.

### CAL-030 A detached occurrence keeps the recurrent marker but loses the series

Condition: an occurrence no longer matches the pattern when the series is applied.
Effect: its series reference is emptied and its recurrent marker is set to true, so that the screen
still offers repetition controls. Detached occurrences that are also in the set being written are
archived; the others are deleted, archiving instead of deleting when a remote copy exists.

### CAL-031 An occurrence edited on its own becomes an outlier

Condition: a time field is written on an occurrence with the scope "This event", and the follow
marker is not written in the same operation.
Effect: the follow marker is lowered. An outlier is excluded when the series looks for its first
occurrence, so it does not become the pattern for the others, and it survives a later regeneration
of the series.

### CAL-032 A series with no remaining occurrence is deleted

Condition: the last occurrence leaves a series, whether by trimming, deletion or archiving with the
scope "This event".
Effect: the series record is removed. When the removal cannot proceed because a remote copy exists,
the series is archived instead.

### CAL-033 A series always has a base occurrence when it can

Condition: the base occurrence is archived or deleted.
Effect: the series selects its earliest remaining occurrence that is not an outlier as the new base.
When every remaining occurrence is an outlier, the earliest of those is chosen only if the caller
asked to include outliers; otherwise the base is emptied and the series generates nothing.

### CAL-034 A series that would generate nothing generates nothing

Condition: the base occurrence of a series is archived and a change with the scope "All events" is
attempted.
Effect: the rewrite archives everything, deletes the series, reactivates the base occurrence and
applies the new pattern. When the base occurrence was inactive to begin with, no series is created.

### CAL-035 Repetition is bounded even when it is stated to be endless

Condition: the termination mode is "Forever".
Effect: the number of generated occurrences is capped, first by a horizon in years read from the
system parameter `calendar.max_recurrence_years` with a shipped default of fifteen, and then by a
hard cap of seven hundred and twenty occurrences. The per-frequency arithmetic is in
[calculations.md, chapter 11](calculations.md#11-the-occurrence-horizon-and-its-caps).

### CAL-036 A count is also capped

Condition: the termination mode is by count.
Effect: at most seven hundred and twenty occurrences are generated, whatever the count says.

### CAL-037 Occurrences in the past are not created

Condition: a series is applied whose period start falls before the start of the reference
occurrence, which happens for a weekly series starting mid-week and for a monthly series starting
mid-month.
Effect: candidate occurrences whose start day is earlier than the reference occurrence's start day
are dropped. When the termination mode is by count and dropping them would leave too few, the count
is temporarily raised, the candidates are regenerated, the past ones are dropped again and the count
is restored. The arithmetic is in
[calculations.md, chapter 12](calculations.md#12-range-calculation-and-count-inflation).

### CAL-038 A series keeps the wall-clock time across a daylight-saving change

Condition: a series spans a change of daylight saving time in its own time zone.
Effect: each occurrence is generated in the series' time zone and only then converted, so a meeting
at six in the morning stays at six in the morning and its instant shifts by one hour instead. When
the wall-clock time is ambiguous, the interpretation that is not in daylight saving time is chosen.
When the wall-clock time does not exist at all, the occurrence lands the same number of minutes
after midnight, which pushes it one hour later on the clock.

### CAL-039 Trimming a series sets its end day to the day before the period start

Condition: a series is stopped at an occurrence and at least one earlier occurrence remains.
Effect: the termination mode becomes "End date" and the end day becomes the day before the start of
the period that contains the stopping occurrence: for a weekly series the first day of that week in
the reader's language, for a monthly series the first day of that month, and otherwise the
occurrence's own day.

### CAL-040 A split series inherits the old pattern and drops the old start weekday

Condition: a series is split at an occurrence whose weekday differs from the weekday of the
occurrence being moved.
Effect: the new series copies every value of the old one, then removes the weekday marker of the old
start day and adds the marker of the new one, and sets its count to the supplied count or, failing
that, to the number of detached occurrences, never below one.

### CAL-041 Shifting a whole series shifts every occurrence by the same amount

Condition: a time field is written with the scope "All events".
Effect: the shift is computed against the base occurrence and applied to it, and the whole series is
rebuilt from the shifted base. The arithmetic is in
[calculations.md, chapter 14](calculations.md#14-the-time-shift-of-a-recurrence-update).

### CAL-042 Breaking a repetition detaches instead of deleting

Condition: the recurrent marker is written as false.
Effect: with the scope "This and following events" the series is stopped at this occurrence and the
later ones are detached; with the scope "All events" every occurrence is detached and the series
itself is removed. Detached occurrences other than the ones addressed are deleted, or archived when
a remote copy exists.

### CAL-043 Applying a series is idempotent

Condition: a series is applied twice with no change in between.
Effect: the second application creates nothing and detaches nothing, because every generated range
already matches an existing occurrence exactly on both the start and the stop.

---

## 4. Visibility and privacy

### CAL-050 Every event has an effective visibility

Condition: any reading of an event.
Effect: the effective visibility is the event's own policy when set, and otherwise the organiser's
calendar default privacy.

### CAL-051 A private event hides its sensitive fields from uninvited readers

Condition: an internal user who is neither the organiser nor an attendee reads an event whose
effective visibility is private.
Effect: the subject reads "Busy"; the attendee set reads empty; every other field that is not in the
public set reads empty. The public set is: the identifier, the activity flag, the all-day marker,
the duration, the organiser, the interval, the organiser's contact, the count, the pattern text, the
series reference, the availability marker, the visibility policy, every pattern field, every time
field, and every field added to the entity by a customisation.

### CAL-052 A private event hides its name in the display name too

Condition: the same reader asks for the display name.
Effect: the display name reads "Busy".

### CAL-053 Grouped reads exclude other people's private events

Condition: an internal user runs a grouped read whose grouping or aggregation touches at least one
field outside the public set.
Effect: the query is narrowed to events that are public or confidential, or organised by the reader,
or that carry no policy while their organiser's default is not private, or that carry no policy and
have no organiser. Private events of other people therefore contribute nothing to the totals.

### CAL-054 Writing another person's private event is refused

Condition: an internal user writes an event whose organiser is somebody else and whose effective
visibility is private, and the reader is not among the attendees.
Effect: the platform's access refusal for the write operation is raised. Administrators are not
exempt: the check is made before any record rule.

### CAL-055 Reading the messages of another person's private event requires write access

Condition: a reader asks to read the messages of an event whose policy is private and to whose
attendee set the reader's contact does not belong.
Effect: the required permission for that read is raised from read to write, so the reader is refused
under CAL-054.

### CAL-056 Only attendees, the organiser and uninvited administrators may edit

Condition: the screen decides whether to allow editing.
Effect: the reader may edit when the reader is the organiser, or is a user linked to one of the
attendee contacts, or was a user linked to one of the attendee contacts before the current unsaved
change, or belongs to the settings-administration group while the event's policy is not private.

### CAL-057 A user may only change their own calendar default privacy

Condition: the calendar default privacy is written on a user record that is not the acting user.
Effect: the write is refused.
Message: "You are not allowed to change the calendar default privacy of another user due to privacy
constraints."

### CAL-058 A new user inherits the platform's default privacy

Condition: a user is created without a calendar default privacy.
Effect: the value of the system parameter `calendar.default_privacy` is applied; when the parameter
is absent, `public` is applied.

### CAL-059 Only internal users receive a settings record

Condition: the calendar default privacy is written back to the user's settings record.
Effect: only internal users are considered; a portal or public user gets no settings record, so
their default remains the platform-wide one.

### CAL-060 A portal user only sees the meetings they attend

Condition: a portal user reads meetings.
Effect: the record rule for the portal group narrows the set to events whose attendee contacts
include the reader's own contact. The portal group may read but not write, create or delete.

### CAL-061 An internal user may read every meeting, subject to privacy

Condition: an internal user reads meetings.
Effect: the record rule for the internal group is unrestricted; privacy is enforced by CAL-051 to
CAL-055 instead of by the rule.

### CAL-062 The private-event rule applies to writing, creating and deleting only

Condition: any write, create or delete of an event.
Effect: the record rule named "Private events" allows the operation when the policy is not private,
or when it is private and the reader is either the organiser or an attendee. The rule deliberately
does not apply to reading, because reading is handled by field obfuscation instead.

### CAL-063 A reader who cannot see a private event cannot add attendees to it

Condition: a reader who is neither organiser nor attendee writes the attendee set of a private event.
Effect: refused under CAL-054 before any attendee record is touched.

---

## 5. Attendees and invitations

### CAL-070 The attendee records follow the attendee contacts

Condition: the attendee contact set is written.
Effect: the system turns the contact instructions into attendee instructions. Removing a contact
deletes its attendee record; adding a contact that is not already an attendee creates one; adding a
contact that is already an attendee creates nothing. Replacing the whole set removes the attendee
records of the contacts that leave and creates records for the contacts that arrive. Instructions
that create or update a contact in place are not supported on this path.

### CAL-071 The acting user's own invitation starts as accepted

Condition: an attendee record is created for the acting user's own contact with no explicit response.
Effect: the response is `accepted`.

### CAL-072 Every other invitation starts as needing an answer

Condition: an attendee record is created for any other contact with no explicit response.
Effect: the response is `needsAction`.

### CAL-073 An attendee record may not be duplicated

Condition: a duplication of an attendee record is attempted.
Effect: refused.
Message: "You cannot duplicate a calendar attendee."

### CAL-074 Duplicating an event recreates its attendees

Condition: an event is duplicated.
Effect: the copy is created with no attendee contacts and no attendee records, and the original's
attendee contacts are then written onto it, which builds fresh attendee records with fresh tokens
and fresh responses. The contact-details paragraph is not prepended a second time.

### CAL-075 Creating or changing an attendee re-checks write access on the event

Condition: an attendee record is created or written.
Effect: immediately afterwards the system checks that the reader may write the parent event, and
raises the platform's access refusal when they may not.

### CAL-076 Removing an attendee unsubscribes the contact from the thread

Condition: an attendee record is deleted.
Effect: for each affected event, the contacts that are both leaving and following the event's
discussion thread are unsubscribed from it.

### CAL-077 A new attendee of a future meeting is invited by electronic mail

Condition: an attendee record is created, and the parent event starts strictly after the current
instant.
Effect: the shipped invitation template is rendered against the attendee record and sent
immediately. Attendees of a meeting that has already started are not invited.

### CAL-078 The reader is not notified of their own action

Condition: a notification is about to be sent to an attendee whose contact is the acting user's own
contact.
Effect: no message and no electronic mail is produced for that attendee, unless the caller asked
explicitly for the author to be notified. The reminder job asks for exactly that, so an organiser
does receive their own reminder.

### CAL-079 An attendee without an address is not notified

Condition: an attendee's electronic mail address is empty.
Effect: no message is produced for that attendee. The address is shown as invalid on the event
screen so that somebody can fix it.

### CAL-080 An address that is not a single well-formed address counts as invalid

Condition: an attendee's address is empty or does not match the single-address pattern.
Effect: the contact appears in the invalid-address set of the event, which the attendee widget
highlights.

### CAL-081 Notifications may be blocked platform-wide

Condition: the system parameter `calendar.block_mail` holds a value that reads as true, or the
caller passes the no-notification instruction.
Effect: no attendee notification of any kind is produced.

### CAL-082 Sending is immediate below a threshold and queued above it

Condition: a batch of attendee notifications is produced with the immediate-sending request.
Effect: the messages are created first and are then sent immediately only when the number of
notified attendees is strictly below the threshold read from the system parameter
`mail.mail_force_send_limit`, whose platform default is one hundred. Above the threshold the
messages stay in the queue and leave with the ordinary mail run.

### CAL-083 Every invitation carries a calendar interchange attachment

Condition: an attendee notification is produced for an event.
Effect: a file named `invitation.ics` is attached, holding the event in the calendar interchange
format, with the creation instant, the start, the end, the summary, the description as plain text,
the location, the video-call address as the address property, the pattern when the event repeats,
one alarm block per reminder, one attendee line per invitation and, when the organiser's contact has
an address, an organiser line. The attachment is created against the composer entity with the
document identifier zero, so that it is not filed against the event.

### CAL-084 A calendar interchange file cannot be produced without dates

Condition: the file is produced for an event whose start or stop is empty.
Effect: refused.
Message: "First you have to specify the date of the invitation."

### CAL-085 A rescheduling of a future meeting notifies the attendees who stay

Condition: the start is written, the operation is not the creation of the event, the caller did not
ask to skip attendee notification, and the new start is at or after the current instant.
Effect: the attendees that were already invited and are still invited receive the shipped
rescheduling template, sent immediately. Attendees that have just arrived receive the invitation
template instead, under CAL-077, and attendees that have just left receive nothing.

### CAL-086 Moving a meeting into the past notifies nobody

Condition: the new start is strictly earlier than the current instant.
Effect: no rescheduling notification is produced.

### CAL-087 Moving a meeting somebody else organises resets the organiser's answer

Condition: at least one time field is written, the event has an organiser and that organiser is not
the acting user.
Effect: the organiser's own attendee record is set back to `needsAction`.

### CAL-088 A recurrence-wide time change resets every answer

Condition: a series is rewritten or trimmed with new time values.
Effect: within the affected occurrences, the acting user's own attendee record is set to `accepted`
and every other attendee record is set to `needsAction`.

### CAL-089 Contacting the attendees requires attendees

Condition: the "Send email" or "Send Mail" action is invoked on events with no attendee contacts.
Effect: refused.
Message: "There are no attendees on these events"

### CAL-090 The same refusal guards the text-message action

Condition: the "Send SMS" action is invoked on events with no attendee contacts.
Effect: refused.
Message: "There are no attendees on these events"

### CAL-091 An invitation is only sent when the organiser has an address

Condition: the "Send Invitations" action is invoked.
Effect: the invitation template is sent to the attendees only when the acting user has an electronic
mail address; otherwise nothing is sent and the action reports success.

### CAL-092 The organiser and a single counterpart are described in the meeting description

Condition: a meeting is created and the caller did not ask to skip the contact paragraph.
Effect: a paragraph is prepended to the description holding, under the heading "Organized by", the
organiser's name, address and telephone number, each present only when filled, and, when exactly one
attendee other than the organiser and the platform's own automated user exists, a second block under
the heading "Contact Details" with that contact's name, address and telephone number. The platform's
automated user is never described.

### CAL-093 A meeting with several counterparts gets no contact-details block

Condition: the meeting has two or more attendees besides the organiser.
Effect: only the organiser block is prepended.

### CAL-094 The organiser is alone when every other attendee has declined

Condition: the event has more than one attendee and every attendee other than the organiser's own
contact has declined.
Effect: the organiser-alone marker becomes true, so that the screen can warn the organiser.

### CAL-095 Joining a meeting by link adds the joiner as an attendee

Condition: a signed-in user opens the joining route with a valid event token.
Effect: when the user's contact is not already an attendee, it is added, which creates an attendee
record. The user is then redirected to the invitation page for their own invitation.

---

## 6. Reminders

### CAL-100 A reminder is defined once and shared

Condition: a reminder is attached to an event.
Effect: the reminder record is shared; changing it changes every event that uses it.

### CAL-101 A reminder still in use cannot be deleted

Condition: a reminder record is deleted while at least one event points at it.
Effect: the association refuses the removal, and the platform reports that the record is still
referenced.

### CAL-102 The lead time in minutes is the only comparison unit

Condition: any comparison of reminders.
Effect: the lead time in minutes is derived from the duration and the unit and is stored, so that
reminders expressed in different units sort and compare correctly.

### CAL-103 Searching a lead time searches the two stored fields

Condition: a search filters on the lead time in minutes.
Effect: the filter is rewritten into three alternatives combined with "or": the unit is minutes and
the duration satisfies the operator against the value; the unit is hours and the duration satisfies
it against the value divided by sixty; the unit is days and the duration satisfies it against the
value divided by sixty and again by twenty-four. A filter for the empty value is rewritten into a
filter on an empty duration. A filter that lists several values is rewritten into the alternation of
the equality filters. Operators other than equality, membership and the four inequalities are not
supported.

### CAL-104 An electronic-mail reminder always has a template

Condition: the channel is set to electronic mail and no template is attached.
Effect: the shipped reminder template is attached. Choosing any other channel, or clearing the
template, empties it.

### CAL-105 A text-message reminder always has a template

Condition: the channel is set to text message and no template is attached.
Effect: the shipped text-message reminder template is attached. Any other channel empties it.

### CAL-106 The organiser flag only applies to text messages

Condition: the organiser flag is set while the channel is electronic mail or in-application
notification.
Effect: the flag is forced back to false on screen.

### CAL-107 The reminder name is rewritten from its own values

Condition: the duration, the unit, the channel or the organiser flag changes on screen.
Effect: the name becomes the channel label, then " - ", then the duration, then a space, then the
unit label; when the organiser flag is on, " - Notify Responsible" is appended.

### CAL-108 The reminder job wakes only for reminders that can still fire

Condition: an event with at least one reminder of a triggering channel is created or changed.
Effect: for each such reminder a one-shot wake-up is planted at the event's start minus the lead
time, unless that instant is not after the last run of the reminder job, and unless the series
already holds a wake-up at exactly that instant. The triggering channels are electronic mail and,
when the text-message package is installed, text message.

### CAL-109 A series keeps exactly one wake-up

Condition: an event that belongs to a series plants a wake-up.
Effect: the wake-up is recorded on the series, replacing whatever the series held. A series of two
hundred occurrences therefore holds one wake-up, not two hundred.

### CAL-110 After a reminder fires, the series plants the next one

Condition: the reminder job has just sent the reminders of a series.
Effect: the next wake-up is computed as follows. If the series' wake-up instant is at or before now,
the next instant is the current occurrence's start minus the shortest lead time when the event still
has an earlier reminder to fire, and otherwise the current occurrence's start. If the series holds
no wake-up at all and the event has at least one reminder, the next instant is the start of the next
occurrence minus the shortest lead time. When neither case yields an instant, no wake-up is planted.

### CAL-111 Reminders created after the previous run are skipped for the past

Condition: a reminder is attached to an event whose firing instant already lies before the previous
run of the reminder job.
Effect: no reminder is sent for it. The attendees have just received an invitation, so a reminder
would be redundant. This is deliberate.

### CAL-112 The reminder job selects by firing instant, not by event

Condition: the reminder job runs.
Effect: for each channel it selects the pairs of reminder and event where the event is active and
the event's start minus the reminder's duration in its own unit falls at or after the previous run
and strictly before now.

### CAL-113 A reminder is not sent for an event that is over

Condition: the reminder job has selected an event whose stop is at or before the current instant.
Effect: the event is dropped before any message is produced.

### CAL-114 A declined attendee receives no reminder

Condition: the reminder job produces messages for an event.
Effect: attendees whose response is `declined` are excluded, on every channel.

### CAL-115 An externally synchronised event sends no electronic-mail reminder from the platform

Condition: the reminder job selects electronic-mail reminders while at least one external calendar
package is installed.
Effect: events that carry an identifier of the first service are excluded, and events that carry an
identifier of the second service are excluded, because those services send their own reminders. The
exclusion applies only to the electronic-mail channel.

### CAL-116 The immediate-sending threshold counts attendees, not reminders

Condition: the reminder job produces messages for several reminders at once.
Effect: the threshold is compared against the total number of attendees to be notified across the
whole run, not against the number per reminder.

### CAL-117 A reminder message notifies the author too

Condition: the reminder job notifies attendees.
Effect: the author-notification request is set, so the organiser receives their own reminder,
contrary to CAL-078.

### CAL-118 A text-message reminder needs a usable number

Condition: the text-message reminder is produced.
Effect: only contacts with a sanitised telephone number receive it. Contacts that declined are
excluded. The organiser's contact is excluded unless the reminder's organiser flag is set. When the
reminder has no template, the fallback text is "Event reminder: %(name)s, %(time)s." where the first
placeholder is the meeting subject and the second is the display time. Text messages are sent
immediately rather than queued.

### CAL-119 In-application reminders are polled, not pushed on a schedule

Condition: a signed-in reader's screen asks for the next notification.
Effect: the reminder engine returns every in-application reminder of the reader's own contact whose
firing instant falls within the next twenty-four hours and which has not already been acknowledged.

### CAL-120 Acknowledgement is remembered per contact

Condition: a reader acknowledges the reminders.
Effect: the acknowledgement instant is written on the reader's contact, and reminders whose firing
instant is at or before it are never offered again.

### CAL-121 Adding, changing or removing a reminder pushes the next notification

Condition: an event with reminders is created, changed or deleted, and the caller did not ask to
suppress notification.
Effect: the reminder engine recomputes the next notification for every attendee contact of the
affected events and pushes it over the notification bus, so that open screens update their
countdown. Only non-shared users are addressed.

### CAL-122 A reminder notification is only pushed for a meeting that has not ended

Condition: the engine decides which events to announce.
Effect: only events that carry at least one reminder and whose stop is at or after the current
instant are announced.

---

## 7. External synchronisation

### CAL-130 Synchronisation is a per-user connection

Condition: any outward or inward call.
Effect: the call is made under one user's credential. A user with no credential synchronises
nothing, and nothing is synchronised on their behalf.

### CAL-131 A suspended service exchanges nothing

Condition: the suspension parameter of a service reads true.
Effect: no insertion, patch or deletion is attempted, and the scheduled run returns immediately.
Records still raise their pending flag, so the backlog is replayed once the suspension is lifted.

### CAL-132 A change is only pushed when it touches a synchronised field

Condition: a record is written.
Effect: the pending flag is raised only when the written fields intersect the service's synchronised
set. The two sets are listed in
[state-machines.md, chapters 7 and 8](state-machines.md#7-record-pending-synchronisation-flag-first-service).

### CAL-133 Outward calls happen after the transaction commits

Condition: an insertion, a patch or a deletion is queued.
Effect: the call is deferred until the database transaction has committed, so that the external
service never learns about a change the platform rolled back. A failure of the deferred call is
logged and does not fail the transaction.

### CAL-134 The organiser's connection is preferred

Condition: an outward call is made for an event whose organiser is not the acting user.
Effect: for the first service the organiser's connection is used when the organiser holds a
short-lived credential, and the acting user's otherwise. For the second service the organiser's
connection is used when the acting user holds a credential and the organiser's synchronisation is
active, and the acting user's otherwise.

### CAL-135 An attendee does not create the organiser's event in the service

Condition: a record whose organiser is somebody other than the acting user is about to be inserted.
Effect: the insertion is skipped, for both services, so that the remote copy is not created under
the wrong owner. The record stays pending until the organiser's own run inserts it.

### CAL-136 An event with a remote copy is archived rather than deleted, in the first service

Condition: an event that carries an identifier of the first service is deleted.
Effect: it is archived instead. Archiving is what the service reads as a removal instruction. Once
the service confirms the removal, the inward run deletes the local record.

### CAL-137 An event with a remote copy is deleted remotely first, in the second service

Condition: an event that carries the universal identifier of the second service is deleted, and
synchronisation is not suspended.
Effect: the remote copy is deleted first, under the organiser's connection, and the local record is
then removed.

### CAL-138 Only the organiser may remove a meeting from the first service

Condition: an inward run reports an event as cancelled.
Effect: events the acting user organises are cancelled locally, which clears the remote identifier
and deletes the record. For the other events, only the acting user's own attendee record is set to
`declined`; the event survives.

### CAL-139 Cancellation in the second service depends on who owns the event

Condition: an inward run reports an event as cancelled.
Effect: events with no organiser, events the acting user organises and events the acting user
attends are cancelled locally. For the rest, the acting user's own attendee record is declined.

### CAL-140 A guest may not change an event the first service protects

Condition: an event whose guest-modification marker is set is written by somebody who is not its
organiser, the change touches a synchronised field, it is not a restart of synchronisation, and the
caller did not ask to skip the permission check.
Effect: refused.
Message: "The following event can only be updated by the organizer according to the event
permissions set on Google Calendar."

### CAL-141 A series may not be created from the platform while the second service is connected

Condition: a batch that contains at least one recurrent event is created while the acting user's
connection to the second service is live and the creation does not come from that service.
Effect: refused.
Message: "Due to an Outlook Calendar limitation, recurrent events must be created directly in
Outlook Calendar."

### CAL-142 A synchronised series may not be changed from the platform

Condition: a change is attempted on a recurrent event that carries the series-head identifier of the
second service, the change is not a pure activity-flag change, the change does not come from that
service, and the organiser's connection is live.
Effect: refused.
Message: "Due to an Outlook Calendar limitation, recurrence updates must be done directly in Outlook
Calendar." When at least one addressed record carries no universal identifier the message instead
reads "Due to an Outlook Calendar limitation, recurrence updates must be done directly in Outlook
Calendar.\nIf this recurrence is not shown in Outlook Calendar, you must delete it in Odoo Calendar
and recreate it in Outlook Calendar."

### CAL-143 The same refusal guards deleting and archiving a synchronised series

Condition: occurrences of a synchronised series are deleted from the list screen, or a synchronised
event is mass-archived, while the connection to the second service is live and the request does not
come from that service.
Effect: refused with the message of CAL-142.

### CAL-144 An occurrence may not jump over its neighbours in the second service

Condition: the start of a single occurrence is written with the scope "This event".
Effect: the number of occurrences whose day is strictly before the current day is compared with the
number whose day is strictly before the proposed day, excluding the occurrence itself. When the two
counts differ the change is refused.
Message: "Outlook limitation: in a recurrence, an event cannot be moved to or before the day of the
previous event, and cannot be moved to or after the day of the following event."

### CAL-145 Every attendee must have an address before the second service is called

Condition: an outward call to the second service is about to be made for events, and at least one
attendee of at least one of them has no electronic mail address.
Effect: refused.
Message: "For a correct synchronization between Odoo and Outlook Calendar, all attendees must have
an email address. However, some events do not respect this condition. As long as the events are
incorrect, the calendars will not be synchronized.\nEither update the events/attendees or archive
these events %(details)s:\n%(invalid_events)s". The first placeholder is either the number of listed
events in parentheses, or the listed count and the total count separated by a solidus in
parentheses when more than fifty events are affected; the second is one line per affected event,
each beginning with a tab and a hyphen, then the display time, then a colon and the display name,
ordered by start.

### CAL-146 Changing the organiser requires the new organiser to be connected and invited

Condition: the organiser of a synchronised event is changed, or an event is created with an
organiser other than the acting user, and the change does not come from the second service.
Effect: when the proposed organiser is not connected to the second service while the acting user is,
the change is refused with "For having a different organizer in your event, it is necessary that the
organizer have its Odoo Calendar synced with Outlook Calendar." When the proposed organiser is
connected but is not among the attendees, the change is refused with "It is necessary adding the
proposed organizer as attendee before saving the event."

### CAL-147 Changing the organiser of a synchronised event recreates it

Condition: the organiser of an event that already carries an identifier of the second service is
changed and the two checks of CAL-146 pass.
Effect: the event's values are copied, the copy is created under the new organiser with the new
values and no remote identifier, the original is archived, and the original's remote copy is
deleted.

### CAL-148 Answers are pushed to the second service through its dedicated operation

Condition: an attendee answers an event that is synchronised with the second service.
Effect: the answer is pushed only when the acting user is not the organiser and is among the
attendees. The event's identifier inside the acting user's own calendar is fetched by its universal
identifier first, because the identifier differs per calendar. The answer carries an empty comment
and asks the service to send a response. Answering an occurrence of a series is refused under
CAL-142.

### CAL-149 A single event that becomes a series is withdrawn as a single event first

Condition: an event that already has a remote copy becomes part of a newly applied series.
Effect: for the first service, an inactive copy carrying the old remote identifier is created so the
next run removes it, the remote copy is deleted at once, and the event's own identifier is cleared.
For the second service, the same is done with both identifiers, and the event is deleted remotely
under the organiser's connection.

### CAL-150 An event of the second service that becomes recurrent is withdrawn first

Condition: the recurrent marker is written on an event that carries an identifier of the second
service and belongs to no series, while synchronisation is not suspended.
Effect: the remote copy is deleted with a three-second request budget and both identifiers are
cleared before the series is built.

### CAL-151 Inward changes only win when they are newer

Condition: an inward run finds an event that already exists locally.
Effect: the remote change is applied only when the remote modification instant is at or after the
locally recorded write instant. For the first service the write instants are captured before the run
begins, so that several updates inside one run compare against the same baseline.

### CAL-152 A small time difference does not reopen an old event

Condition: the second service reports a change on an event whose stop lies before a configured lower
bound.
Effect: the change is applied only when the difference between the remote modification instant and
the local write instant is at least one hour. The lower bound is read from the system parameter
`microsoft_calendar.sync.lower_bound_range`, expressed in days; when the parameter is absent the
check is not made. For a series the bound is tested against every occurrence, and any occurrence
inside the bound is enough.

### CAL-153 Birthday entries are not imported

Condition: the first service returns an entry whose kind is a birthday.
Effect: it is discarded before anything else happens.

### CAL-154 A full run is bounded in time

Condition: a run with no change marker is made.
Effect: the first service is asked for the window from the current instant less a number of days to
the current instant plus the same number of days, read from the system parameter
`google_calendar.sync.range_days` with a default of three hundred and sixty-five. The second service
is asked for the window from the current instant less that number of days, read from
`microsoft_calendar.sync.range_days` with the same default, to the current instant plus twice that
number of days.

### CAL-155 An outward run is bounded in count

Condition: records are selected to be pushed to the first service.
Effect: at most two hundred records are taken in one transaction. The rest are pushed by later runs.

### CAL-156 An expired change marker forces a full run

Condition: the service rejects the change marker.
Effect: the run is repeated without a marker, which makes it a full run, and the new marker returned
by that run is stored.

### CAL-157 Occurrences that follow their series are not pushed individually

Condition: the outward scope is computed.
Effect: events that are recurrent, belong to a series and still follow it are excluded; the series
carries them.

### CAL-158 The outward scope is limited to the reader's own meetings

Condition: the outward scope is computed.
Effect: only events at least one of whose attendee contacts is linked to the acting user are
considered, and only those whose stop is after the lower bound and whose start is before the upper
bound.

### CAL-159 A series is only pushed to the first service when it has a pattern

Condition: the outward scope of a series is computed.
Effect: only series whose pattern text is not empty and at least one of whose occurrences is
organised by the acting user are considered.

### CAL-160 A series is never pushed to the second service on its own

Condition: the outward scope of a series is computed for the second service.
Effect: the scope is empty. Series reach that service only through the events that make them up.

### CAL-161 Refreshing a credential that has been revoked disconnects the user

Condition: the service answers a refresh with a client error or an authorisation error.
Effect: the transaction is rolled back, the stored credentials are cleared and committed, and the
operation is refused with, for the first service, "An error occurred while generating the token.
Your authorization code may be invalid or has already expired [%s]. You should check your Client ID
and secret on the Google APIs plateform or try to stop and restart your calendar synchronization."
and, for the second service, "An error occurred while generating the token. Your authorization code
may be invalid or has already expired [%s]. You should check your Client ID and secret on the
Microsoft Azure portal or try to stop and restart your calendar synchronisation." The placeholder is
the error code the service returned, or the two letters `nc` when it returned none.
**Compatibility finding**: the first message contains the misspelling "plateform" and the two
messages spell "synchronization" and "synchronisation" differently. Both are reproduced because
support procedures and automated tests key on them; a corrected behaviour would spell both words the
same way and correct the misspelling.

### CAL-162 Exchanging an authorisation code that has expired is refused

Condition: the code returned by the consent round trip is rejected.
Effect: refused, with, for the first service, "Something went wrong during your token generation.
Maybe your Authorization Code is invalid or already expired" and, for the second service, "Something
went wrong during your token generation. Maybe your Authorization Code is invalid".

### CAL-163 A rejected outward call is reported on the event

Condition: the first service rejects an insertion or a patch with a client error or a forbidden
answer.
Effect: the record's pending flag is lowered so that it is not retried, and a note is posted on the
event whose body reads "The following event could not be synced with Google Calendar." then a line
break, then "It will not be synced as long at it is not updated." then a line break, then the
reason. The reason is "you don't seem to have permission to modify this event on Google Calendar"
when the answer is a forbidden answer mentioning a non-organiser restriction, and otherwise "Google
gave the following explanation: " followed by the message the service returned.
**Compatibility finding**: the second sentence reads "as long at it is not updated" where "as long
as" is meant. The text is reproduced; a corrected behaviour would read "It will not be synced as
long as it is not updated."

### CAL-164 A rejection on a series stops the whole series being retried

Condition: the rejected record is a series.
Effect: in addition to CAL-163, every occurrence of that series has its pending flag lowered, and
the note is posted on the base occurrence, or on the earliest occurrence including outliers when
there is no base.

### CAL-165 A run refuses to overlap itself

Condition: a run starts for a user whose row is already locked by another run.
Effect: the run returns immediately without doing anything.

### CAL-166 The consent round trip must name a service

Condition: the callback of either service is reached without a service name in the returned state,
or with an authorisation code but no return address.
Effect: the request is refused as a malformed request.

### CAL-167 The consent link is only offered to an administrator

Condition: the platform holds no application registration for a service.
Effect: the screen is told that configuration is needed, and is given the general settings action
only when the reader belongs to the settings-administration group.

### CAL-168 A first connection ignores previously created meetings, in the second service

Condition: the very first run against the second service happens on a platform where no user has
ever synchronised.
Effect: the system parameter `microsoft_calendar.sync.first_synchronization_date` is set to the
current instant less one minute, and from then on the outward scope only contains events created at
or after it. This prevents a flood of invitations for meetings that pre-date the connection.

### CAL-169 A meeting the platform believes is over sends no update invitations

Condition: an outward call is made to the first service for an event that is over.
Effect: the service is asked not to notify the attendees. An event is over when, being all-day, its
stop day is strictly before today, or, being timed, its stop is strictly before the current instant.
A series is over when every one of its occurrences is over.

### CAL-170 A meeting whose organiser is connected does not also receive platform invitations

Condition: an attendee notification is about to be produced for an event.
Effect: for the first service, the notification is skipped when the event's chosen user is connected
and their short-lived credential is still valid. For the second service, it is skipped when the
chosen user's connection is live, their state is active, and the event either already carries a
remote identifier or is pending. The service sends its own invitations instead.

---

## 8. Activities and linked documents

### CAL-180 A meeting created against a document creates an activity

Condition: a meeting is created carrying a linked document whose entity supports activities, that
entity is not on the exclusion list, the meeting carries no complete activity instruction of its
own, and at least one activity type of the meeting category exists.
Effect: an activity is created on that document, of the first matching activity type, with the
meeting's description as its note, the meeting's subject as its summary, the deadline derived from
the start, and the meeting's organiser as its assignee.

### CAL-181 The deadline of the activity is the day of the start in the reader's time zone

Condition: the deadline is derived.
Effect: for an all-day meeting the day part of the start is used directly; for a timed meeting the
start is first converted from coordinated universal time into the reader's time zone and the day
part of the converted value is used.

### CAL-182 Rescheduling a meeting moves its activity, and moving the activity moves the meeting

Condition: the subject, the description, the start or the organiser of a meeting changes, or the
deadline of an activity changes.
Effect: the corresponding field of the other record is written. Both directions carry a marker that
suppresses the opposite direction, so the two cannot loop. For a timed meeting the activity's new
deadline is turned back into a shift in whole days, computed in the reader's time zone, and applied
to the start; for an all-day meeting the difference between the new deadline and the start day is
applied directly.

### CAL-183 An activity of the meeting category may only be scheduled on one document

Condition: the scheduling wizard is asked to create a meeting while it addresses more than one
document.
Effect: refused.
Message: "Scheduling an activity using the calendar is not possible on more than one record."

### CAL-184 Completing an activity records its feedback on the meeting

Condition: an activity that points at a meeting is marked done with feedback.
Effect: the meeting's internal notes gain a line break followed by "Feedback: " and the feedback
turned into markup. The description, which is synchronised outwards, is not touched.

### CAL-185 An activity may be deleted together with its meeting

Condition: the combined deletion operation is invoked.
Effect: the activity is deleted first and the meeting afterwards.

### CAL-186 A meeting created from an existing activity of the meeting category reuses that type

Condition: the creation context names exactly one originating activity, that activity already points
at a meeting, and its type is of the meeting category.
Effect: the new meeting's activity is created with that same type, when the type applies to the
document's entity or to no entity in particular.

### CAL-187 A document identifier of zero is treated as absent

Condition: a meeting is created with a linked document identifier of zero.
Effect: the default identifier is used instead, so that a zero never becomes a dangling reference.

---

## 9. Public access to an invitation

### CAL-190 The invitation routes authenticate by token

Condition: any of the accept, decline, recurrence-accept, recurrence-decline and view routes is
called.
Effect: the token in the request is matched against the invitation tokens. When no invitation
matches, the request is refused with the text "Invalid Invitation Token." When a session is open and
its login is not the anonymous one, the session's contact must be the invitation's contact;
otherwise the request is refused with "Invitation cannot be forwarded via email. This event/meeting
belongs to %s and you are logged in as %s. Please ask organizer to add you." The first placeholder
is the invitation's electronic mail address and the second the signed-in user's address. When the
checks pass, the request continues as a public request.

### CAL-191 The view route needs the event identifier as well as the token

Condition: the view route is called.
Effect: the invitation must match both the token and the event identifier; otherwise the request
answers "not found".

### CAL-192 An internal user is redirected to the full screen

Condition: the view route is reached by an internal user.
Effect: the reader is sent to `/app/calendar.event/` followed by the event identifier, with the
database name as a query value.

### CAL-193 Everybody else sees the simplified page

Condition: the view route is reached by anybody else.
Effect: the simplified invitation page is rendered in the invitation contact's own language and time
zone, and shows the accept and decline buttons, the current answer, the invitation's common name and
address, the display time, the location or a hyphen, the list of attendee common names, and the
description or a hyphen. The two buttons submit to the accept and decline routes and carry the
platform's request-forgery token.

### CAL-194 The video-call route is public

Condition: the video-call joining route is called with an event token.
Effect: when no event carries that token the request answers "not found". Otherwise the conversation
channel is created if the event has none, and the reader is redirected to that channel's invitation
address.

### CAL-195 A recurring meeting gets one token per occurrence

Condition: a platform-hosted video call is requested on a recurring meeting.
Effect: every occurrence receives its own token and therefore its own address, deliberately, so that
deleting the base occurrence does not lock the other occurrences out of their call.

### CAL-196 A conversation channel is shared across a series

Condition: the channel of an occurrence of a series is created.
Effect: when any occurrence of the same series already has a channel, that channel is reused;
otherwise a new group channel is created with the meeting subject as its name, the attendee contacts
as its members and full-screen video as its display mode, and every other occurrence of the series
that has no channel is pointed at it. The channel's description becomes the series' pattern
sentence, or the meeting's display time when the meeting does not repeat.

### CAL-197 Adding an attendee to a meeting with a live channel adds them to the channel

Condition: the attendee contact set is written on an event that already has a conversation channel.
Effect: the contacts named by the linking and replacing instructions are added as channel members.

---

## 10. Configuration, tags and filters

### CAL-200 A tag name is unique

Condition: a second tag with the same name is created.
Effect: refused by the constraint `_name_uniq`.
Message: "Tag name already exists!"

### CAL-201 A tag gets a random colour

Condition: a tag is created without a colour.
Effect: a whole number drawn at random between one and eleven inclusive is stored.

### CAL-202 A user may list a colleague only once

Condition: a second calendar-filter row is created for the same user and contact.
Effect: refused by the constraint `_user_id_partner_id_unique`.
Message: "A user cannot have the same contact twice."

### CAL-203 Removing a contact removes it from every side list

Condition: the named operation that clears a contact from the side lists is called.
Effect: every calendar-filter row pointing at that contact is deleted, for every user.

### CAL-204 Only an administrator may configure a service from the provider panel

Condition: the provider panel's action is invoked.
Effect: the platform's administrative-access assertion is applied and the call is written to the
audit log. A reader outside the settings-administration group is refused with the platform's own
message.

### CAL-205 The provider panel installs the package it needs

Condition: the chosen service's capability package is not installed.
Effect: it is installed immediately, before the parameters are written.

### CAL-206 The provider panel only writes the chosen service's parameters

Condition: the panel is confirmed.
Effect: exactly three parameters are written, all belonging to the chosen service. The other
service's parameters are untouched, even though the panel holds values for them.

---

## 11. Concurrency and locking

### CAL-210 One synchronisation per user at a time

Condition: a run begins for a user.
Effect: the user's row is locked for update, allowing other rows to reference it. When the lock
cannot be taken the run is abandoned silently, because the transaction could not have been committed
anyway.

### CAL-211 Each user's run commits on its own

Condition: the scheduled synchronisation job iterates over the connected users.
Effect: after each user the transaction is committed. A failure for one user is logged and rolled
back, and the job continues with the next user.

### CAL-212 Reminder wake-ups are planted under the platform's own authority

Condition: a wake-up is planted.
Effect: the scheduled job record is read and triggered with elevated rights, because an ordinary
user may not write scheduled jobs.

### CAL-213 An outward call carries a request budget

Condition: an outward call is made.
Effect: calls made from a create or write operation carry three seconds for the first service, and
for the second service the number of seconds read from the system parameter
`microsoft_calendar.graph_timeout`, defaulting to five and never below one. Calls made from a
scheduled run carry twenty seconds.

### CAL-214 A record deleted between the queueing and the sending of a call is tolerated

Condition: a deferred outward call runs for a record that no longer exists.
Effect: the failure is logged and swallowed. For the first service a rejection on a record that no
longer exists is logged with the reason and nothing is posted anywhere.

---

## 12. Rule index

| Identifier | Subject | Refusal message, if any |
|---|---|---|
| CAL-001 | Subject required | platform required-field message |
| CAL-002 | Start and stop required | platform required-field message |
| CAL-003 | Timed meeting order | "The ending date and time cannot be earlier than the starting date and time.\nMeeting “%(name)s” starts at %(start_time)s and ends at %(end_time)s" |
| CAL-004 | All-day meeting order | "The ending date cannot be earlier than the starting date.\nMeeting “%(name)s” starts on %(start_date)s and ends on %(end_date)s" |
| CAL-005 | Stop from start and duration | — |
| CAL-006 | Duration from start and stop | — |
| CAL-007 | Default start | — |
| CAL-008 | Default stop and default duration | — |
| CAL-009 | All-day day and instant consistency | — |
| CAL-010 | Filling instants from days on screen | — |
| CAL-011 | All-day instants are conventional | — |
| CAL-020 | Pattern field with the scope "This event" | "Unable to save the recurrence with \"This Event\"" |
| CAL-021 | Scope ignored on several events | — |
| CAL-022 | Trimming at the head | — |
| CAL-023 | Missing base occurrence | "You can't update a recurrence without base event." |
| CAL-024 | Interval positive | "The interval cannot be negative." |
| CAL-025 | Count positive | "The number of repetitions cannot be negative." |
| CAL-026 | Weekly needs a weekday | "You have to choose at least one day in the week" |
| CAL-027 | Monthly day range | "The day must be between 1 and 31" |
| CAL-028 | Missing day of month | — |
| CAL-029 | Occurrences are records | — |
| CAL-030 | Detachment | — |
| CAL-031 | Outliers | — |
| CAL-032 | Empty series deleted | — |
| CAL-033 | Base reselection | — |
| CAL-034 | Inactive base | — |
| CAL-035 | Endless repetition bounded | — |
| CAL-036 | Count capped | — |
| CAL-037 | Past occurrences dropped | — |
| CAL-038 | Daylight saving | — |
| CAL-039 | Trim end day | — |
| CAL-040 | Split inherits pattern | — |
| CAL-041 | Whole-series shift | — |
| CAL-042 | Breaking repetition | — |
| CAL-043 | Idempotent application | — |
| CAL-050 | Effective visibility | — |
| CAL-051 | Private field obfuscation | — |
| CAL-052 | Private display name | "Busy" |
| CAL-053 | Grouped reads | — |
| CAL-054 | Writing a private event | platform access refusal |
| CAL-055 | Reading private messages | platform access refusal |
| CAL-056 | Who may edit | — |
| CAL-057 | Another user's default privacy | "You are not allowed to change the calendar default privacy of another user due to privacy constraints." |
| CAL-058 | New user default privacy | — |
| CAL-059 | Settings records for internal users only | — |
| CAL-060 | Portal record rule | — |
| CAL-061 | Internal record rule | — |
| CAL-062 | Private-events record rule | — |
| CAL-063 | Adding attendees to a private event | platform access refusal |
| CAL-070 | Attendees follow contacts | — |
| CAL-071 | Own invitation accepted | — |
| CAL-072 | Other invitations pending | — |
| CAL-073 | No duplication | "You cannot duplicate a calendar attendee." |
| CAL-074 | Duplicating an event | — |
| CAL-075 | Access re-check | platform access refusal |
| CAL-076 | Unsubscription | — |
| CAL-077 | Invitation on creation | — |
| CAL-078 | No self-notification | — |
| CAL-079 | No address, no message | — |
| CAL-080 | Invalid address set | — |
| CAL-081 | Blocking notifications | — |
| CAL-082 | Immediate versus queued | — |
| CAL-083 | Calendar interchange attachment | — |
| CAL-084 | Attachment needs dates | "First you have to specify the date of the invitation." |
| CAL-085 | Rescheduling notification | — |
| CAL-086 | No notification for the past | — |
| CAL-087 | Organiser's answer reset | — |
| CAL-088 | Series-wide answer reset | — |
| CAL-089 | Contacting with no attendees | "There are no attendees on these events" |
| CAL-090 | Text message with no attendees | "There are no attendees on these events" |
| CAL-091 | Invitation needs an organiser address | — |
| CAL-092 | Contact-details paragraph | — |
| CAL-093 | No paragraph for a group | — |
| CAL-094 | Organiser alone | — |
| CAL-095 | Joining by link | — |
| CAL-100 | Shared reminders | — |
| CAL-101 | Reminder in use | platform reference refusal |
| CAL-102 | Lead time in minutes | — |
| CAL-103 | Searching a lead time | — |
| CAL-104 | Electronic-mail template | — |
| CAL-105 | Text-message template | — |
| CAL-106 | Organiser flag | — |
| CAL-107 | Reminder name | — |
| CAL-108 | Wake-up planting | — |
| CAL-109 | One wake-up per series | — |
| CAL-110 | Next wake-up | — |
| CAL-111 | Late reminders skipped | — |
| CAL-112 | Selection window | — |
| CAL-113 | Events that are over | — |
| CAL-114 | Declined attendees | — |
| CAL-115 | Externally synchronised events | — |
| CAL-116 | Threshold counts attendees | — |
| CAL-117 | Author notified | — |
| CAL-118 | Text-message recipients | "Event reminder: %(name)s, %(time)s." |
| CAL-119 | Polling | — |
| CAL-120 | Acknowledgement | — |
| CAL-121 | Bus push | — |
| CAL-122 | Only future meetings announced | — |
| CAL-130 | Per-user connection | — |
| CAL-131 | Suspension | — |
| CAL-132 | Synchronised fields | — |
| CAL-133 | After-commit calls | — |
| CAL-134 | Whose connection | — |
| CAL-135 | Attendee insertion blocked | — |
| CAL-136 | Archive instead of delete | — |
| CAL-137 | Remote delete first | — |
| CAL-138 | Cancellation, first service | — |
| CAL-139 | Cancellation, second service | — |
| CAL-140 | Guest modification | "The following event can only be updated by the organizer according to the event permissions set on Google Calendar." |
| CAL-141 | Series creation forbidden | "Due to an Outlook Calendar limitation, recurrent events must be created directly in Outlook Calendar." |
| CAL-142 | Series update forbidden | "Due to an Outlook Calendar limitation, recurrence updates must be done directly in Outlook Calendar." and its longer variant |
| CAL-143 | Series deletion forbidden | as CAL-142 |
| CAL-144 | Occurrence overlap | "Outlook limitation: in a recurrence, an event cannot be moved to or before the day of the previous event, and cannot be moved to or after the day of the following event." |
| CAL-145 | Attendee addresses required | "For a correct synchronization between Odoo and Outlook Calendar, all attendees must have an email address. …" |
| CAL-146 | Organiser change | "For having a different organizer in your event, it is necessary that the organizer have its Odoo Calendar synced with Outlook Calendar." / "It is necessary adding the proposed organizer as attendee before saving the event." |
| CAL-147 | Organiser change recreates | — |
| CAL-148 | Answers to the second service | — |
| CAL-149 | Single event becomes a series | — |
| CAL-150 | Recurrent marker withdrawal | — |
| CAL-151 | Newest wins | — |
| CAL-152 | Old-event guard | — |
| CAL-153 | Birthdays ignored | — |
| CAL-154 | Full-run window | — |
| CAL-155 | Outward batch size | — |
| CAL-156 | Expired change marker | — |
| CAL-157 | Following occurrences excluded | — |
| CAL-158 | Outward scope | — |
| CAL-159 | Series scope, first service | — |
| CAL-160 | Series scope, second service | — |
| CAL-161 | Revoked credential | "An error occurred while generating the token. …" in two variants |
| CAL-162 | Expired authorisation code | "Something went wrong during your token generation. …" in two variants |
| CAL-163 | Rejected call reported | "The following event could not be synced with Google Calendar." and the rest of the note |
| CAL-164 | Rejected series | — |
| CAL-165 | Overlapping runs | — |
| CAL-166 | Malformed consent return | platform malformed-request refusal |
| CAL-167 | Consent link for administrators | — |
| CAL-168 | First-connection cut-off | — |
| CAL-169 | No updates for past meetings | — |
| CAL-170 | No double invitations | — |
| CAL-180 | Activity creation | — |
| CAL-181 | Activity deadline | — |
| CAL-182 | Two-way synchronisation with activities | — |
| CAL-183 | One document at a time | "Scheduling an activity using the calendar is not possible on more than one record." |
| CAL-184 | Feedback into notes | "Feedback: %s" |
| CAL-185 | Combined deletion | — |
| CAL-186 | Reusing an activity type | — |
| CAL-187 | Zero document identifier | — |
| CAL-190 | Token authentication | "Invalid Invitation Token." / "Invitation cannot be forwarded via email. …" |
| CAL-191 | View route arguments | platform not-found answer |
| CAL-192 | Internal redirect | — |
| CAL-193 | Simplified page | — |
| CAL-194 | Video-call route | platform not-found answer |
| CAL-195 | One token per occurrence | — |
| CAL-196 | Shared channel | — |
| CAL-197 | Channel membership | — |
| CAL-200 | Unique tag | "Tag name already exists!" |
| CAL-201 | Random colour | — |
| CAL-202 | Unique side-list row | "A user cannot have the same contact twice." |
| CAL-203 | Contact removal | — |
| CAL-204 | Administrative panel | platform administrative-access refusal |
| CAL-205 | Package installation | — |
| CAL-206 | Parameter isolation | — |
| CAL-210 | Run locking | — |
| CAL-211 | Per-user commit | — |
| CAL-212 | Elevated wake-up planting | — |
| CAL-213 | Request budgets | — |
| CAL-214 | Vanished records tolerated | — |
