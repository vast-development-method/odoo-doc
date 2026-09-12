# Workflows of Calendar and Scheduling

## 1. Overview

This document gives the end-to-end procedures of the domain, step by step. Each step names the
records it creates or changes, the operation it invokes and the conditions under which it fails.
Rules quoted by identifier are specified in [business-rules.md](business-rules.md); arithmetic quoted
by chapter is in [calculations.md](calculations.md).

| Chapter | Procedure |
|---|---|
| [2](#2-create-a-meeting) | Create a meeting |
| [3](#3-invite-and-disinvite-attendees) | Invite and disinvite attendees |
| [4](#4-respond-to-an-invitation) | Respond to an invitation |
| [5](#5-reschedule-a-meeting) | Reschedule a meeting |
| [6](#6-make-a-meeting-repeat) | Make a meeting repeat |
| [7](#7-change-a-series) | Change a series |
| [8](#8-break-a-repetition) | Break a repetition |
| [9](#9-archive-and-delete) | Archive and delete |
| [10](#10-reminders-scheduling-and-sending) | Reminders: scheduling and sending |
| [11](#11-in-application-reminders) | In-application reminders |
| [12](#12-hold-a-video-call) | Hold a video call |
| [13](#13-see-when-people-are-available) | See when people are available |
| [14](#14-turn-an-activity-into-a-meeting) | Turn an activity into a meeting |
| [15](#15-configure-an-external-calendar-service) | Configure an external calendar service |
| [16](#16-connect-a-users-account) | Connect a user's account |
| [17](#17-a-synchronisation-run-against-the-first-service) | A synchronisation run against the first service |
| [18](#18-a-synchronisation-run-against-the-second-service) | A synchronisation run against the second service |
| [19](#19-reset-an-account) | Reset an account |
| [20](#20-pause-stop-and-restart-synchronisation) | Pause, stop and restart synchronisation |
| [21](#21-send-a-text-message-reminder) | Send a text-message reminder |

---

## 2. Create a meeting

### 2.1 Entry points

| Entry point | What it pre-fills |
|---|---|
| The calendar screen, by dragging over a time slot | The start and the stop from the slot, the reader's own contact as the only attendee. |
| The quick-creation panel | The same, plus the subject, the location, the visibility and the notes. |
| The meeting button on a Contact | The reader's own contact and the contact whose screen it was opened from, both as attendees. |
| An Activity of the meeting category | The linked document, its display name as the subject, the activity note as the description, the activity assignee as organiser and only attendee, the activity's deadline as the day to open the calendar on, and the activity itself as the meeting's activity. |
| An inward synchronisation | Everything the remote copy carries; see chapters [17](#17-a-synchronisation-run-against-the-first-service) and [18](#18-a-synchronisation-run-against-the-second-service). |

### 2.2 Procedure

1. **Resolve the linked document.** When the creation context names an entity but no entity
   definition, the definition is looked up; when it names a definition but no entity, the transport
   name is read from the definition. When neither is given but the screen was opened from another
   document, that document's entity and identifier are used. A document identifier of zero is
   treated as absent (CAL-187).
2. **Apply the defaults.** The activity set, the all-day marker, the description, the subject, the
   linked document identifier and entity, the start and the organiser are filled from the defaults
   when the caller supplied none. The organiser falls back to the acting user.
3. **Decide whether an activity must be created.** The procedure of CAL-180 runs. When an activity is
   to be created, its values are placed in the meeting's own activity instruction, so that the
   activity and the meeting are created in one step.
4. **Complete the video-call address.** An address that carries no scheme is completed with the
   platform's own scheme and host, so that a relative address becomes absolute.
5. **Derive the attendee instructions.** When the caller gave no attendee instruction, the attendee
   contacts are turned into attendee instructions by CAL-070. When the caller gave none of either,
   the reader's own contact is used.
6. **Prepend the contact paragraph.** Unless the caller asked to skip it, the paragraph of CAL-092 is
   built and placed before the existing description.
7. **Create the records in two batches.** Meetings that carry no recurrent marker are created first.
   Meetings that do are created next, each with the follow marker already set.
8. **Build the series.** For each recurrent meeting, the pattern fields are removed from its values
   and applied as a series. The occurrences the pattern produces are created; the meeting that
   started the process is detached when it does not itself sit on the pattern, and, being detached,
   is archived.
9. **Invite the attendees.** Every attendee record just created whose meeting starts in the future
   receives the invitation template, sent immediately (CAL-077, CAL-082).
10. **Keep the activity in step.** For meetings whose activity instruction was not a complete fresh
    creation, the activity's summary, note, deadline and assignee are written from the meeting.
11. **Plant the reminder wake-ups.** Unless the caller asked to suppress notification, wake-ups are
    planted for the meetings that are not all-day; an all-day meeting plants its wake-up through the
    write that fills its instants instead. Meetings that belong to a series plant through the series.
12. **Announce the next reminder.** The reminder engine recomputes and pushes the next in-application
    reminder for every attendee contact concerned (CAL-121).

### 2.3 Failure conditions

| Condition | Result |
|---|---|
| No subject | CAL-001 |
| Stop before start | CAL-003 or CAL-004 |
| A recurrent meeting while the second external calendar service is connected | CAL-141 |
| An organiser other than the acting user, while the second service is connected | CAL-146 |
| A weekly pattern with no weekday | CAL-026 |
| A monthly pattern with an impossible day | CAL-027 |

---

## 3. Invite and disinvite attendees

1. The reader edits the attendee contacts on the meeting, or a synchronisation writes them.
2. The contact instructions are turned into attendee instructions (CAL-070): contacts leaving have
   their attendee records deleted, contacts arriving get new ones.
3. When the meeting already has a conversation channel, the arriving contacts are added to it
   (CAL-197).
4. The meeting is written. Deleting an attendee record unsubscribes its contact from the discussion
   thread when the contact was following (CAL-076).
5. The set of attendee records that are new, that is the records present now and absent before, is
   invited with the invitation template, sent immediately (CAL-077). The reader's own record is not
   notified (CAL-078); records with no address are skipped (CAL-079).
6. Because the attendee set changed, the reminder wake-ups are replanted and the next in-application
   reminder is pushed.

Failure conditions: the reader may not write a private meeting they are not part of (CAL-063); an
attendee record may not be duplicated (CAL-073); every attendee must have an address before the
second external calendar service is called (CAL-145).

---

## 4. Respond to an invitation

### 4.1 From inside the application

1. The reader opens the meeting and uses the response control, or opens the invitation list on the
   options page and uses one of the three buttons.
2. The named operation for the chosen answer runs. Accepting and declining post a message on the
   meeting under the "Invitation" subtype, authored by the responding contact.
3. The two external services are told, each in its own way (CAL-148 and the response propagation of
   [entities.md, chapter 3](entities.md#3-calendar-attendee-information-calendarattendee)).

### 4.2 From an invitation message, without signing in

1. The recipient follows the accept or decline link in the invitation message. The link carries the
   invitation token.
2. The request is authenticated by token (CAL-190). A signed-in reader whose contact is not the
   invitation's contact is refused.
3. Every invitation record matching that token and not already in the target answer is changed.
4. The reader is redirected to the invitation view route, which shows the simplified page or, for an
   internal user, the full meeting screen (CAL-192, CAL-193).

### 4.3 Answering a whole series at once

The recurrence-accept and recurrence-decline routes resolve the token to one invitation, walk to that
invitation's meeting, walk to the series, and change every invitation of the same contact on every
occurrence of the series that is not already in the target answer.

### 4.4 Answering with a chosen scope from the application

The named operation `change_attendee_status` takes an answer and a scope. With "All events" it
addresses every occurrence of the series; with "This and following events" every occurrence starting
at or after this one; otherwise this one alone. Within that set it changes only the invitation whose
contact is the reader's.

---

## 5. Reschedule a meeting

1. The reader changes the start, the stop, the start day or the stop day.
2. The derivations of [calculations.md, chapter 3](calculations.md#3-duration-and-stop) fill whichever
   of the duration and the stop was not written.
3. The order check of CAL-003 or CAL-004 runs.
4. When the meeting belongs to a series and the scope is "This event", the follow marker is lowered,
   making the occurrence an outlier (CAL-031).
5. When the organiser is not the acting user, the organiser's own answer is reset to needing action
   (CAL-087).
6. The reminder wake-ups are replanted and the next in-application reminder is pushed.
7. When the new start is in the future and the caller did not ask to skip notification, the attendees
   who were already invited and are still invited receive the rescheduling template, sent immediately
   (CAL-085). Attendees who have just arrived receive the invitation template instead.
8. When the meeting is linked to an activity, that activity's deadline is rewritten from the new start
   (CAL-182).
9. When the second external calendar service is connected and the meeting is an occurrence of a
   series moved with the scope "This event", the neighbour check of CAL-144 runs first and may refuse
   the change.

---

## 6. Make a meeting repeat

1. The reader ticks the repetition marker and chooses a simplified frequency, or chooses "Custom" and
   fills the pattern fields.
2. The simplified frequency and the pattern fields are mirrored onto the meeting from the series when
   one exists, and from the series defaults when none does. Choosing an interval other than one shows
   "Custom".
3. On saving, the pattern fields are removed from the values and applied as a series:
   - When the meeting has no series, a new series is created with this meeting as its base occurrence
     and as its only occurrence so far.
   - When the meeting already has a series and the scope is "This and following events", the existing
     series is split at this meeting.
4. The meeting is marked recurrent and following.
5. The series is applied: the ranges are computed
   ([calculations.md, chapters 10 to 12](calculations.md#10-occurrence-generation-and-time-zones)),
   the occurrences that already match are kept, the ones that no longer match are detached, and the
   missing ones are created by copying the base occurrence.
6. Copies are created with attendee notification suppressed and with no log entry, so that a
   two-hundred-occurrence series does not send two hundred invitations.
7. Detached occurrences that are also being written are archived; the others are deleted, or archived
   when a remote copy exists (CAL-030).
8. When the first external calendar service holds a single-event copy of a meeting that has just
   become part of a series, that copy is withdrawn (CAL-149).
9. One reminder wake-up is planted for the whole series (CAL-109).

Failure conditions: CAL-020 when a pattern field is written with the scope "This event" and the
repetition marker is not being set at the same time; CAL-024, CAL-025, CAL-026 and CAL-027 for
impossible patterns; CAL-141 when the second external calendar service is connected.

---

## 7. Change a series

The behaviour depends entirely on the scope carried with the change.

### 7.1 Scope "This event"

The change is written to that occurrence alone. Writing a time field lowers the follow marker. Writing
a pattern field is refused (CAL-020).

### 7.2 Scope "This and following events"

1. The time shift is computed against this occurrence
   ([calculations.md, chapter 14](calculations.md#14-the-time-shift-of-a-recurrence-update)); the
   supplied time values are replaced by the shifted ones.
2. The values of the current series are copied, and the weekday marker of this occurrence's current
   start day is removed when the new start day falls on a different weekday.
3. The series is stopped at this occurrence: this occurrence and every later one are detached, and the
   series' end day is set by
   [calculations.md, chapter 13](calculations.md#13-the-end-day-of-a-trimmed-series).
4. Every detached occurrence except this one is archived; for the first external calendar service
   their remote identifiers are also cleared.
5. This occurrence receives the change, together with the identifier-clearing values and the
   no-further-push values of the first service.
6. When a time value changed, the answers are reset: the acting user's own answer becomes accepted and
   every other answer becomes needing action (CAL-088).
7. The new pattern is assembled from the copied values, the weekday and monthly parameters recomputed
   from the new start day, and the supplied pattern values. Its count is the supplied count or, failing
   that, the number of detached occurrences.
8. A new series is created from this occurrence with that pattern and applied.

When this occurrence is the base occurrence of its series, the procedure of 7.3 is used instead
(CAL-022).

### 7.3 Scope "All events"

1. The base occurrence is the series' base, or, when there is none, the series' earliest occurrence
   that is not an outlier.
2. The time shift is computed against this occurrence and applied to the base occurrence.
3. When nothing that the outside world cares about changed and no time value and no pattern value
   changed, the change is simply written onto every occurrence of the series, with attendee
   notification suppressed and the scope forced to "This event". The procedure ends here.
4. Otherwise the values of the current series are copied and the weekday marker of the old start day
   is dropped when the weekday changes.
5. Every occurrence of the series is archived and the series is deleted.
6. The base occurrence is reactivated, detached from the deleted series and given the change and the
   shifted times, with attendee notification suppressed.
7. When a time value changed, the answers are reset as in 7.2.
8. The new pattern is assembled and a new series is created from the base occurrence and applied.
   Occurrences detached by that application are archived.

### 7.4 What the two external services do

The first service marks the new series pending when the change touched one of its synchronised
fields, and the occurrences of the rebuilt series are not pushed one by one. The second service
refuses the whole procedure when the series is already synchronised (CAL-142).

---

## 8. Break a repetition

1. The reader unticks the repetition marker on an occurrence and picks a scope.
2. With the scope "This and following events" the series is stopped at this occurrence: this
   occurrence and the later ones are detached and the series keeps the earlier ones with an end day.
3. With the scope "All events" every occurrence is detached, the series reference is cleared on all of
   them and the series is deleted.
4. Detached occurrences other than the one addressed are deleted, or archived when a remote copy
   exists.
5. The addressed occurrence stays active and keeps its repetition marker set, so that the screen still
   offers repetition controls, but has no series.

**Worked outcome.** A weekly series of three occurrences, broken at the second with the scope "This
and following events": the first occurrence stays in the series, which now ends on the day before the
start of the second occurrence's week; the second occurrence stays active and detached; the third is
deleted. Broken at the second with the scope "All events": the first and the third are deleted, the
second stays active and detached, and the series is deleted.

---

## 9. Archive and delete

### 9.1 Deleting a single meeting

1. The reader deletes the meeting.
2. The attendee contacts of meetings that carry reminders are remembered, so that their next
   in-application reminder can be recomputed afterwards.
3. The series whose base occurrence is being deleted are remembered.
4. The records are removed. Attendee records and activities go with them. A meeting that carries an
   identifier of the first external calendar service is archived instead (CAL-136). A meeting that
   carries the universal identifier of the second service is deleted remotely first (CAL-137).
5. The remembered series pick a new base occurrence (CAL-033).
6. The next in-application reminder is pushed to the remembered contacts.

### 9.2 Deleting from the calendar screen

1. The reader deletes an occurrence from the calendar. The screen calls the meeting's deletion action.
2. When the organiser has any live external connection, or more than one meeting is addressed, the
   meetings are deleted at once and the reader is sent to `/app/calendar`.
3. Otherwise the shipped deletion template is looked up. When it is missing, the meeting is deleted at
   once and a warning is logged.
4. Otherwise the confirmation panel opens, pre-filled with the template, the reader's chosen scope,
   the meeting, and the recipients, which are the organiser's contact together with every attendee's
   contact.
5. The reader either deletes, which applies the scope and returns to `/app/calendar`, or sends and
   deletes, which first sends the deletion template to the meeting with the light notification layout,
   immediately, and then deletes.

### 9.3 Choosing how much of a series to delete

1. The first panel offers the three scopes of
   [state-machines.md, chapter 10](state-machines.md#10-deletion-scope-selector).
2. On confirmation, when the meeting has exactly one attendee and that attendee is the organiser's own
   contact, the scope is applied directly: "Delete this event" deletes the occurrence; the two others
   run the mass-deletion operation.
3. Otherwise the deletion action of 9.2 runs, so the attendees can be told.

### 9.4 Mass deletion

| Scope | Effect |
|---|---|
| "All events" | Every occurrence of the series is collected, the series is deleted, and then the occurrences are deleted. |
| "This and following events" | Every occurrence whose start is at or after this one's is deleted. The series survives and keeps the earlier occurrences. |

### 9.5 Mass archiving

Mass archiving exists because the two external services sometimes make deletion impossible.

| Scope | Effect |
|---|---|
| "All events" | Every occurrence is written with the archive values, which lower the activity flag and, for the first service, also lower the pending flag. Before that, the first service deletes the whole series remotely in a single request, which triggers a single cancellation message rather than one per occurrence. |
| "This and following events" | The series is stopped at this occurrence and the detached occurrences are archived. When this occurrence is the base occurrence, the first service treats the case as "All events" for efficiency. |
| "This event" | This occurrence alone is archived with the scope "This event". When the series is then empty it is deleted; when this occurrence was the base, the series picks a new base. |

The second service refuses mass archiving of a synchronised meeting (CAL-143).

---

## 10. Reminders: scheduling and sending

### 10.1 Planting the wake-ups

Whenever a meeting with reminders is created or changed, or its attendee set or reminder set changes:

1. The scheduled reminder job is read with elevated rights (CAL-212).
2. For each occurrence and each of its reminders of a triggering channel, the firing instant is
   computed and the wake-up conditions of
   [calculations.md, chapter 7](calculations.md#7-the-reminder-firing-instant-and-the-wake-up) are
   evaluated. A wake-up is planted when they hold.
3. Occurrences that belong to a series record their wake-up on the series, which keeps exactly one.
4. Occurrences that carry at least one in-application reminder and whose stop has not passed are
   collected, and the next in-application reminder is pushed to their attendee contacts.

For a series, the planting is done for the earliest occurrence that starts after the reference
instant, found in one query per series.

### 10.2 The reminder job

The job runs once a day and whenever a wake-up fires.

1. Select the pairs of reminder and occurrence for the electronic-mail channel, using the window of
   [calculations.md, section 7.3](calculations.md#73-the-selection-window-of-the-reminder-job). When
   the selection is empty, the job ends.
2. Read the immediate-sending threshold from the platform parameter.
3. Collect the distinct occurrences, drop the ones whose stop has passed (CAL-113), collect their
   attendee records and drop the ones that declined (CAL-114).
4. For each selected reminder, notify the attendee records that belong to that reminder's occurrences,
   using the reminder's own template, with the author included (CAL-117), and sending immediately only
   when the total number of attendees does not exceed the threshold (CAL-116).
5. Ask the occurrences that belong to a series to plant the next wake-up
   ([calculations.md, chapter 8](calculations.md#8-the-next-reminder-instant-in-a-series)).
6. When the text-message package is installed, repeat steps 1, 3 and 4 for the text-message channel
   with the procedure of [chapter 21](#21-send-a-text-message-reminder), and plant the next wake-ups
   again.

Failure conditions: none of the steps refuses; a meeting that cannot be notified is simply skipped.
Meetings that carry an identifier of either external service are excluded from the electronic-mail
selection (CAL-115).

---

## 11. In-application reminders

1. The reader's screen calls the polling route every few minutes.
2. The reminder engine collects, for the reader's own contact, every active meeting that carries an
   in-application reminder and whose reminder window straddles the present: the earliest firing
   instant of the meeting must be earlier than the upper bound and its latest firing instant later
   than now. The upper bound is the earliest future firing instant plus three minutes, or the current
   instant when there is none.
3. Meetings the reader may not read are dropped.
4. For each remaining meeting, the reminders of the in-application channel whose firing instant falls
   within the next twenty-four hours and strictly after the reader's acknowledgement instant are
   turned into payloads.
5. The payload of each is described in
   [interfaces.md, chapter 11](interfaces.md#11-the-notification-bus-contract).
6. The reader's screen shows the countdown. When the reader dismisses the reminders, the
   acknowledgement route writes the current instant onto the reader's contact, and those reminders are
   never offered again.

The same payload is pushed over the notification bus, without polling, whenever a meeting with
reminders changes (CAL-121).

---

## 12. Hold a video call

1. The reader asks for a platform-hosted call on the meeting. A token is generated when the meeting
   has none, and the address becomes the platform's base address followed by `/calendar/join_videocall/`
   and the token.
2. On a repeating meeting, each occurrence receives its own token and therefore its own address
   (CAL-195).
3. Nothing else happens until somebody follows the address.
4. The first person to follow it reaches the public joining route. The meeting is found by its token;
   when none matches, the request answers "not found".
5. When the meeting has no conversation channel, one is created: a group channel named after the
   meeting, with the attendee contacts as members and full-screen video as its display mode, whose
   description is the series' pattern sentence for a repeating meeting and the meeting's display time
   otherwise. Every other occurrence of the same series that has no channel is pointed at it; when one
   already had a channel, that one is reused instead (CAL-196).
6. The reader is redirected to the channel's invitation address.
7. Adding attendees later adds them to the channel (CAL-197). The channel never rings its members,
   because the meeting is the invitation.
8. A reader may clear the address, which empties it and leaves the token in place.

---

## 13. See when people are available

1. The reader adds attendees to a meeting, or moves it in time.
2. The unavailability of the attendees is recomputed
   ([calculations.md, chapter 16](calculations.md#16-unavailability-from-overlapping-meetings)) and
   the attendee widget marks the unavailable ones.
3. When the working-hours package is installed, the schedule check of
   [calculations.md, chapter 17](calculations.md#17-unavailability-from-working-schedules) also runs,
   so an attendee outside their working hours is marked too.
4. The calendar screen additionally asks for the working hours common to the selected attendees over
   the visible window, and shades the rest of the grid
   ([calculations.md, section 17.4](calculations.md#174-the-working-hours-shown-behind-the-calendar)).
5. The calendar screen asks which days are unusual for the reader, so that non-working days are shaded
   as well; the answer comes from the reader's own employee record.
6. The reader ticks and unticks colleagues in the side list. Each tick writes a calendar-filter row for
   the reader and that colleague; unticking clears its ticked marker rather than deleting the row.

---

## 14. Turn an activity into a meeting

### 14.1 From an existing activity

1. The reader opens an activity whose type is of the meeting category and presses the scheduling
   button.
2. The calendar action opens with the activity's type, the linked document and its identifier, the
   document's display name as the subject, the activity note as the description, the activity itself
   as the meeting's activity, the assignee's contact as the only attendee, the assignee as organiser,
   the activity's deadline as the day to open on, and the activity's existing meeting when it has one.
3. The reader saves the meeting. Because the activity instruction is a complete link rather than a
   fresh creation, no second activity is created; the meeting's values are instead written back onto
   the activity.

### 14.2 From the scheduling wizard

1. The reader opens the scheduling wizard on a document and picks a type of the meeting category.
2. The wizard refuses when it addresses more than one document (CAL-183).
3. When the wizard has no document, a personal activity is scheduled and its meeting action is
   returned. Otherwise the activity is scheduled against the document and its meeting action is
   returned.

### 14.3 Keeping the two in step afterwards

Writing the subject, the description, the start or the organiser of the meeting writes the summary,
the note, the deadline and the assignee of the activity. Writing the deadline of the activity moves
the meeting by the same number of days. Both directions carry a marker that stops the other from
firing, so the pair cannot oscillate (CAL-182). Marking the activity done appends its feedback to the
meeting's internal notes (CAL-184). Deleting the activity together with its meeting is a single
operation (CAL-185).

---

## 15. Configure an external calendar service

### 15.1 From the general settings screen

1. An administrator opens the settings screen and finds the calendar block, which offers one switch
   per service.
2. Turning a switch on marks the corresponding capability package for installation. The screen shows
   the warning "Save this page and come back here to set up the feature.", with the word "Save" in
   bold.
3. After saving and reopening, the block shows three controls per service: the registration identifier
   labelled "Client ID", the secret labelled "Client Secret", masked, and a switch labelled "Pause
   Synchronization".
4. Saving writes the three system parameters of that service.

### 15.2 From the provider panel

1. A reader who is not yet connected sees a connect control on the calendar screen and opens the
   provider panel.
2. The panel offers the two services as radio buttons and shows, for the chosen one, its registration
   identifier, its secret and its suspension switch, each pre-filled from the stored parameter.
3. Confirming runs the panel's action, which is restricted to administrators and written to the audit
   log (CAL-204). The action installs the chosen service's capability package when it is absent
   (CAL-205) and then writes exactly that service's three parameters (CAL-206).

---

## 16. Connect a user's account

1. The reader presses the synchronise control on the calendar screen, which calls the service's own
   synchronisation route with the entity name of Calendar Event.
2. The route reads the registration identifier. When it is missing or empty, it answers that
   configuration by an administrator is needed, and includes the general settings action only when the
   reader belongs to the settings-administration group (CAL-167).
3. When the reader holds no long-lived credential, the route builds the consent address and answers
   that consent is needed, together with that address. The consent address carries the registration
   identifier, the requested scope, the return address of the platform's own callback, and a state
   value holding the database name, the service name, the address to return to and the database's
   unique identifier. The first service also asks for offline access and forces the consent screen to
   be shown again.
4. The reader consents at the service. The service calls the platform's callback with an authorisation
   code and the state value.
5. The callback reads the service name from the state; when it is missing, or a code is present with
   no return address, the request is refused as malformed (CAL-166).
6. The code is exchanged for a long-lived credential, a short-lived credential and a lifetime. Failure
   is refused with the message of CAL-162.
7. The three values are written: for the first service onto the reader's settings record, for the
   second onto the reader's own record.
8. The reader is redirected to the address held in the state.
9. The reader presses the synchronise control again; this time a run happens.

---

## 17. A synchronisation run against the first service

The run happens for one user, either from the synchronisation route or from the scheduled job.

1. **Check the state.** When the derived state is not active, the run ends and reports nothing.
2. **Take the lock.** The user's row is locked for update. When the lock cannot be taken, the run ends
   silently (CAL-210).
3. **Decide the mode.** The run is a full run when the user holds no change marker.
4. **Fetch.** The events changed since the marker are fetched under the user's short-lived credential,
   following the service's page links until there are no more. A full run fetches the window of
   CAL-154 instead. A rejected marker causes an immediate full fetch (CAL-156). The new marker and the
   service's default reminders are kept.
5. **Store the marker.** The new marker is written onto the user's settings record.
6. **Filter.** Entries whose kind is a birthday are discarded (CAL-153).
7. **Stop when there is nothing to do.** When no remote entry survived and no local record is pending,
   the run ends and reports that nothing happened.
8. **Resolve ambiguous cancellations.** A cancelled entry carries only its identifier and its status,
   so the system cannot tell a cancelled meeting from a cancelled series. Each such entry is looked up
   among the local series; the ones that match are treated as series and the rest as meetings.
9. **Capture the local write instants** of the meetings and series about to be touched, so that
   several remote updates inside one run all compare against the same baseline (CAL-151).
10. **Bring the series inward.** New remote series create a local series and its base occurrence;
    cancelled ones are cancelled locally; the rest are updated when the remote change is newer. The
    detail is in
    [external-calendar-synchronisation.md, chapter 5](external-calendar-synchronisation.md#5-inward-from-the-first-service).
11. **Bring the meetings inward**, the same way, using the service's default reminders for entries
    that ask for them.
12. **Push the series outward.** The series inside the outward scope, minus the ones just brought
    inward, are pushed: cancelled ones deleted, new ones inserted, known ones patched. Their
    occurrences, minus their outliers, are added to the set considered already handled.
13. **Push the meetings outward**, the same way, skipping the ones already handled.
14. **Report.** The run reports that the screen should refresh when anything was fetched or pushed.

The scheduled job repeats the whole procedure for every user who holds a long-lived credential and
has not stopped synchronising, ordered by the expiry of their short-lived credential so that the ones
nearest expiry go first, committing after each user and rolling back and logging on failure
(CAL-211).

---

## 18. A synchronisation run against the second service

1. **Check the state.** When the derived state is not active, the run ends.
2. **Record the first-connection cut-off.** When the platform has never synchronised with this service
   and the parameter is unset, it is set to the current instant less one minute (CAL-168).
3. **Decide the mode.** The run is a full run when the user holds no change marker.
4. **Fetch the changes.** The service is asked for the events added, changed or removed since the
   marker, page by page. Entries of the occurrence kind are dropped from that answer, because they
   arrive without the identifier that is the same in every calendar.
5. **Fetch the occurrences.** For every series head in the answer, the service is asked for that
   series' instances, page by page, and those are added.
6. **Handle a rejected marker.** When the service reports that a full synchronisation is required, the
   fetch is repeated without a marker.
7. **Store the marker.**
8. **Bring everything inward.** Remote entries are matched against local records first by the
   identifier that is the same in every calendar and then by the calendar-scoped identifier; matching
   by the second also writes the first onto the local record, so that later runs match on the first.
   New entries are created, cancelled ones are cancelled, and the rest are updated when the remote
   change is newer and passes the old-event guard (CAL-151, CAL-152). Series are handled through their
   heads and their instances. The detail is in
   [external-calendar-synchronisation.md, chapter 8](external-calendar-synchronisation.md#8-inward-from-the-second-service).
9. **Push the series outward.** The outward scope of a series is empty for this service (CAL-160), so
   in practice nothing is pushed here; the occurrences carry the series.
10. **Push the meetings outward**, skipping those just brought inward: cancelled ones deleted, new ones
    inserted, pending known ones patched. Every attendee must have an address first (CAL-145).
11. **Record the run.** The user's last-synchronisation instant is set to the current instant, which
    narrows the outward scope of later runs.
12. **Report.**

The scheduled job repeats the procedure for every user who holds a long-lived credential and has not
stopped, committing after each and rolling back and logging on failure.

---

## 19. Reset an account

### 19.1 First external calendar service

1. The administrator opens the user's screen and presses the reset control, which opens the panel with
   the user pre-filled.
2. The administrator chooses what happens to the user's events and what the next run does, and
   confirms.
3. The events the user organises that carry a remote identifier are collected, and the series whose
   base occurrence is one of them and that carry a remote identifier are collected.
4. When the deletion policy is "Delete from the current Google Calendar account" or "Delete from
   both", each collected event is deleted at the service under the user's own credential.
5. When the deletion policy is any of the three deleting policies, the remote identifier is cleared on
   every collected event and series, with the permission check of CAL-140 skipped. When the policy is
   not "Delete from the current Google Calendar account", the events are then deleted locally.
6. The next-run policy is turned into a value: "Synchronize all existing events" raises the pending
   flag, "Synchronize only new events" lowers it.
7. When the deletion policy left the events in place, that value is written onto them.
8. The three credential values are cleared, and the change marker and the last-calendar marker are
   emptied.

### 19.2 Second external calendar service

1. The same panel, with the second option labelled "Delete from the current Microsoft Calendar
   account".
2. The events the user organises that carry the universal identifier are collected; separately, the
   ones among them that belong to no series are collected, because a series is never touched here, to
   avoid a flood of cancellations.
3. When the deletion policy deletes at the service, each of those single events is deleted remotely
   with a three-second request budget.
4. When the next-run policy is "Synchronize all existing events", every collected event has its
   calendar-scoped identifier cleared and its pending flag raised.
5. When the deletion policy deletes locally, every collected event has its calendar-scoped identifier
   cleared and is then deleted.
6. The transaction is committed at this point, deliberately, so that the deferred remote deletions run
   while the user still holds a credential.
7. The credentials are cleared, and the change marker and the last-synchronisation instant are
   emptied.

---

## 20. Pause, stop and restart synchronisation

| Action | Who | Effect |
|---|---|---|
| Suspend a service | An administrator, from the settings screen, the provider panel or the named operation | The service's suspension parameter becomes true. No call of any kind is made for anybody. Local records keep raising their pending flag. |
| Resume a service | The same | The parameter becomes false. The next run pushes the whole backlog. |
| Stop synchronising | A user, for themself | The user's stopped flag is raised. For the second service the last-synchronisation instant is also cleared. New records created by that user are created with the pending flag already lowered. |
| Restart synchronising | A user, for themself | The stopped flag is lowered. For the second service the last-synchronisation instant is set to the current instant, so everything written before the restart is ignored. Every series and every meeting inside the outward scope has its pending flag raised, so the next run pushes them all. |

An event created while a service is suspended keeps its pending flag raised and is pushed once the
suspension is lifted. An event updated while a service is suspended is modified locally and keeps its
pending flag. An event deleted while the first service is suspended is archived locally and its remote
copy survives until the suspension is lifted; an event deleted while the second service is suspended
is removed locally without any remote call at all.

---

## 21. Send a text-message reminder

1. The reminder job selects the pairs of reminder and occurrence for the text-message channel, using
   the same window as the electronic-mail channel.
2. For each reminder, its occurrences are collected.
3. For each occurrence:
   - the contacts that declined are collected;
   - the recipients are the meeting's message recipients that have a sanitised telephone number and
     are not among the decliners;
   - when the meeting has an organiser and the reminder does not carry the organiser flag, the
     organiser's contact is removed from the recipients;
   - the reminder's text-message template is rendered against the meeting and sent to those
     recipients, immediately rather than queued. When the reminder has no template, the fallback text
     of CAL-118 is used.
4. The next wake-up is planted for the occurrences that belong to a series.

**Worked outcome.** A meeting has two attendee contacts, one with a telephone number and one without.
The reminder is sent to exactly one recipient. A second meeting one hour away and a third meeting
twenty-four hours away, each with a one-hour and a twenty-four-hour reminder, produce three messages
in a single run: the one-hour reminder for the two meetings that start in about half an hour, and the
twenty-four-hour reminder for the meeting that starts in about twenty-three and a half hours.
