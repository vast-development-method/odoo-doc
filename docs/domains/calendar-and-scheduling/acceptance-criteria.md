# Acceptance criteria of Calendar and Scheduling

Every scenario below is numbered `CAL-AC-nnn`, states concrete starting records, a concrete operation
with concrete inputs, and the exact resulting records, instants, counts and states. Unless a scenario
says otherwise:

- the platform holds no default meeting duration, so the default is one hour;
- the reader's language formats a date as month, day and year separated by solidi and a time as hours,
  minutes and seconds separated by colons, and begins the week on Monday;
- the reader's time zone is coordinated universal time;
- no external calendar service is installed;
- the system parameter `calendar.max_recurrence_years` is absent, so the horizon is fifteen years.

Instants written without a zone are in coordinated universal time.

| Group | Scenarios |
|---|---|
| [1. Creating a meeting](#1-creating-a-meeting) | 001 to 014 |
| [2. Times, durations and all-day meetings](#2-times-durations-and-all-day-meetings) | 020 to 032 |
| [3. Attendees and invitations](#3-attendees-and-invitations) | 040 to 059 |
| [4. Responses](#4-responses) | 070 to 082 |
| [5. Repetition: generating a series](#5-repetition-generating-a-series) | 090 to 110 |
| [6. Repetition: changing a series](#6-repetition-changing-a-series) | 120 to 139 |
| [7. Archiving and deleting](#7-archiving-and-deleting) | 150 to 161 |
| [8. Reminders](#8-reminders) | 170 to 186 |
| [9. Privacy and access](#9-privacy-and-access) | 200 to 214 |
| [10. Video calls](#10-video-calls) | 220 to 226 |
| [11. Availability and working hours](#11-availability-and-working-hours) | 230 to 239 |
| [12. Activities and linked documents](#12-activities-and-linked-documents) | 250 to 258 |
| [13. Tags, side lists and configuration](#13-tags-side-lists-and-configuration) | 270 to 278 |
| [14. The first external calendar service](#14-the-first-external-calendar-service) | 300 to 329 |
| [15. The second external calendar service](#15-the-second-external-calendar-service) | 340 to 366 |
| [16. Text-message reminders](#16-text-message-reminders) | 380 to 384 |

---

## 1. Creating a meeting

### CAL-AC-001 A meeting created with only a subject gets defaults

**Given** the current instant is 16 April 2024 at 10:07:23 and no default duration is stored anywhere.
**When** a meeting is created with the subject "Kick-off" and nothing else.
**Then** exactly one Calendar Event exists with the subject "Kick-off", a start of
16 April 2024 at 10:30:00, a stop of 16 April 2024 at 11:30:00, a duration of 1.0, the availability
marker `busy`, the activity flag true, no visibility policy, the acting user as organiser, and one
Calendar Attendee Information record whose contact is the acting user's contact and whose response is
`accepted`.

### CAL-AC-002 The default start does not move an instant already on a boundary

**Given** the current instant is 16 April 2024 at 10:30:00.
**When** a meeting is created with only a subject.
**Then** its start is 16 April 2024 at 10:30:00 and its stop is 11:30:00.

### CAL-AC-003 A personal default duration wins over the platform default

**Given** a user has a stored default duration of 2 for the field `duration` on Calendar Event, for no
particular company, and the current instant is 16 April 2024 at 10:07:23.
**When** that user creates a meeting with only a subject.
**Then** the start is 10:30:00, the stop is 12:30:00 and the duration is 2.0.

### CAL-AC-004 A company default applies to readers acting in that company

**Given** a second company has a stored default duration of 8, and the reader has no personal default.
**When** the reader, acting in that second company, asks for the default duration.
**Then** the answer is 8.

### CAL-AC-005 Creating a meeting without a subject is refused

**Given** nothing.
**When** a meeting is created with a start of 16 April 2024 at 09:00:00, a stop of 10:00:00 and no
subject.
**Then** the creation is refused by the platform's required-field check naming "Meeting Subject", and
no Calendar Event exists.

### CAL-AC-006 A meeting created from a contact invites both parties

**Given** a contact named "Wood Corner".
**When** the meeting button on that contact is pressed and a meeting is created with the subject
"Review".
**Then** the meeting has two attendee contacts, the reader's own contact and "Wood Corner", two
attendee records, the reader's own with the response `accepted` and "Wood Corner" with the response
`needsAction`.

### CAL-AC-007 The organiser paragraph is prepended for a one-to-one meeting

**Given** the organiser is a user named "Colleen Diaz" with the address `colleen@example.com` and no
telephone number, and the only other attendee is a contact named "Wood Corner" with the address
`wood@example.com` and the telephone number `+32 2 555 01 01`.
**When** the meeting is created with an empty description.
**Then** the description holds a block whose first line is "Organized by" in bold, then "Colleen
Diaz", then a link to `colleen@example.com`, then a blank line, then "Contact Details" in bold, then
"Wood Corner", then a link to `wood@example.com`, then a link to the telephone number.

### CAL-AC-008 The contact-details block is omitted when there are two counterparties

**Given** the same organiser and two other attendee contacts.
**When** the meeting is created.
**Then** the description holds only the "Organized by" block.

### CAL-AC-009 A meeting created against a document creates an activity

**Given** an activity type of the meeting category exists and applies to every entity, and a contact
whose identifier is 42 supports activities.
**When** a meeting is created with the subject "Call back", the description "Ask about the quote", a
start of 16 April 2024 at 14:00:00, the linked entity Contact and the linked identifier 42, and no
activity of its own.
**Then** one Activity exists on that contact, of that type, with the summary "Call back", the note
"Ask about the quote", the deadline 16 April 2024 and the meeting's organiser as assignee, and the
meeting's activity set holds exactly that activity.

### CAL-AC-010 A linked identifier of zero is treated as absent

**Given** the same setup.
**When** a meeting is created with the linked entity Contact and the linked identifier 0.
**Then** the meeting's linked identifier is the value the defaults supply, and when the defaults
supply none it is empty; no Activity is created.

### CAL-AC-011 Duplicating a meeting recreates its attendees

**Given** a meeting with two attendee contacts whose responses are `accepted` and `declined`, and one
attendee record each.
**When** the meeting is duplicated.
**Then** the copy has the same two attendee contacts, two new attendee records with new invitation
tokens, the response of the copy's own reader's record being `accepted` and the other `needsAction`,
and the copy's description holds no second contact paragraph.

### CAL-AC-012 Duplicating an all-day meeting keeps its days

**Given** an all-day meeting with a start day of 16 October 2018 and a stop day of 18 October 2018.
**When** it is duplicated.
**Then** the copy's start day is 16 October 2018 and its stop day is 18 October 2018.

### CAL-AC-013 A duplicated attendee record is refused

**Given** an attendee record.
**When** it is duplicated.
**Then** the operation is refused with "You cannot duplicate a calendar attendee."

### CAL-AC-014 Ordering is by start, descending

**Given** four meetings, "foo" on 1 April 2011 at 12:00:00, "bar" on 1 May 2011 at 12:00:00, "foo" on
1 June 2011 at 12:00:00 and "bar" on 1 June 2011 at 12:00:00.
**When** they are read with no explicit order.
**Then** the two meetings of 1 June come first, then 1 May, then 1 April.

---

## 2. Times, durations and all-day meetings

### CAL-AC-020 The stop follows the start when the duration is kept

**Given** a meeting starting 16 April 2024 at 09:00:00 with a duration of 1.75.
**When** nothing else is written.
**Then** the stop is 16 April 2024 at 10:45:00.

### CAL-AC-021 A halfway duration rounds to the nearer even minute, upwards

**Given** a meeting starting 16 April 2024 at 09:00:00.
**When** its duration is written as 0.125.
**Then** its stop is 16 April 2024 at 09:08:00.

### CAL-AC-022 A halfway duration rounds to the nearer even minute, downwards

**Given** the same meeting.
**When** its duration is written as 0.375.
**Then** its stop is 16 April 2024 at 09:22:00.

### CAL-AC-023 The duration follows the stop

**Given** a meeting starting 16 April 2024 at 09:00:00.
**When** its stop is written as 16 April 2024 at 12:19:59.
**Then** its duration is 3.33.

### CAL-AC-024 A stop before a start is refused for a timed meeting

**Given** a meeting named "Doom's day".
**When** its start is written as 16 April 2024 at 12:00:00 and its stop as 16 April 2024 at 11:00:00.
**Then** the write is refused with "The ending date and time cannot be earlier than the starting date
and time.\nMeeting “Doom's day” starts at 2024-04-16 12:00:00 and ends at 2024-04-16 11:00:00", and no
change is stored.

### CAL-AC-025 A stop day before a start day is refused for an all-day meeting

**Given** an all-day meeting named "Conference".
**When** its start day is written as 18 October 2018 and its stop day as 16 October 2018.
**Then** the write is refused with "The ending date cannot be earlier than the starting date.\nMeeting
“Conference” starts on 2018-10-18 and ends on 2018-10-16".

### CAL-AC-026 An all-day meeting rewrites its instants from its days

**Given** the reader's time zone is ten hours behind coordinated universal time.
**When** an all-day meeting is created with a start of 16 October 2018 at 00:00:00, a start day of
16 October 2018, a stop of 18 October 2018 at 00:00:00 and a stop day of 18 October 2018.
**Then** after the values are reloaded, the start is 16 October 2018 at 08:00:00, the stop is
18 October 2018 at 18:00:00 and the duration is 58.0.

### CAL-AC-027 Pushing the start of an all-day meeting works

**Given** an all-day meeting whose start day and stop day are both today, with one attendee contact.
**When** the start day is written as tomorrow.
**Then** the meeting's start day and its start move by one day and no validation refuses the change.

### CAL-AC-028 The display time of a short timed meeting

**Given** a meeting from 16 April 2024 at 08:00:00 to 10:00:00 with a duration of 2.0, read in a zone
two hours ahead named `Europe/Brussels`.
**When** the display time is asked for.
**Then** it reads "04/16/2024 at (10:00:00 To 12:00:00) (Europe/Brussels)".

### CAL-AC-029 The display time of a long meeting

**Given** a meeting from 21 October 2019 at 08:00:00 to 23 October 2019 at 18:00:00, read in a zone two
hours ahead named `Europe/Brussels`.
**When** the display time is asked for.
**Then** it reads "10/21/2019 at 10:00:00 To", then a line break, then " 10/23/2019 at 20:00:00
(Europe/Brussels)".

### CAL-AC-030 The display time of an all-day meeting

**Given** an all-day meeting starting 31 July 2013.
**When** the display time is asked for.
**Then** it reads "All Day, 07/31/2013".

### CAL-AC-031 A forced time zone changes the display time

**Given** the meeting of CAL-AC-028.
**When** the display time is asked for with the zone forced to coordinated universal time.
**Then** it reads "04/16/2024 at (08:00:00 To 10:00:00) (UTC)".

### CAL-AC-032 A meeting with no dates cannot produce an interchange file

**Given** a meeting whose stop has been emptied by a caller writing directly.
**When** the calendar interchange file is produced.
**Then** the operation is refused with "First you have to specify the date of the invitation."

---

## 3. Attendees and invitations

### CAL-AC-040 Adding a contact creates an attendee record

**Given** a meeting created by user "Xavier" starting 25 October 2019 at 08:00:00 with no attendees.
**When** the attendee contacts are written as Xavier's own contact.
**Then** one attendee record exists whose contact is Xavier's contact, and Xavier's contact is a
follower of the meeting's discussion thread.

### CAL-AC-041 An attendee supplied at creation does not become a follower

**Given** nothing.
**When** a meeting is created with the attendee contact Xavier supplied in the creation values.
**Then** one attendee record exists and Xavier's contact is not a follower.

### CAL-AC-042 Explicit responses at creation are honoured

**Given** a contact "orga" with the address `orga@example.com`.
**When** a meeting is created with two attendee records supplied explicitly, Xavier's with the response
`needsAction` and "orga"'s with `accepted`, and both contacts in the attendee contacts.
**Then** the meeting has two attendee records with exactly those responses.

### CAL-AC-043 Adding the same contact twice creates no second record

**Given** a meeting whose attendee contacts already include Xavier.
**When** Xavier is added again.
**Then** the same attendee record is still the only one for Xavier.

### CAL-AC-044 The acting user's own invitation is accepted

**Given** nothing.
**When** user Xavier writes the attendee contacts of a meeting as Xavier's own contact.
**Then** the attendee record's response is `accepted`.

### CAL-AC-045 Removing a contact deletes its record and unsubscribes it

**Given** a meeting with attendee contacts Xavier and "Bis", and one attendee record each.
**When** Xavier is removed from the attendee contacts.
**Then** the attendee record of "Bis" is unchanged and is still the only remaining one, Xavier has no
attendee record, and Xavier's contact is no longer a follower.

### CAL-AC-046 Writing plain identifiers instead of instructions is handled

**Given** a meeting with no attendees and a contact whose identifier is 7.
**When** the attendee contacts are written as the plain list holding 7.
**Then** one attendee record exists whose contact is 7.

### CAL-AC-047 An attendee with no address is flagged

**Given** a contact with no electronic mail address.
**When** it is added to a meeting.
**Then** an attendee record exists, and the meeting's invalid-address set holds exactly that contact.

### CAL-AC-048 An attendee with a malformed address is flagged

**Given** a contact whose address is the text "I'm an invalid email".
**When** it is added to a meeting.
**Then** the meeting's invalid-address set holds exactly that contact.

### CAL-AC-049 A new attendee of a future meeting is invited immediately

**Given** a meeting starting in the future and the immediate-sending threshold left at its default.
**When** a contact with the address `bob.invitee@example.com` is added.
**Then** one message is produced for that contact from the invitation template, and it is sent rather
than queued.

### CAL-AC-050 More than the threshold means queued

**Given** the immediate-sending threshold is 100 and a meeting starting in the future.
**When** 101 contacts, each with an address, are added at once.
**Then** 101 notifications are produced and none is sent immediately; all leave with the ordinary mail
run.

### CAL-AC-051 The reader is not notified of their own invitation

**Given** user Xavier and a meeting.
**When** Xavier adds Xavier's own contact as attendee.
**Then** no notification of any kind is produced.

### CAL-AC-052 An archived meeting invites nobody

**Given** a meeting whose activity flag is false.
**When** a contact is added.
**Then** no notification is produced.

### CAL-AC-053 Moving a future meeting notifies the attendees who stay

**Given** a meeting with one attendee contact, starting in the past relative to nothing but with a new
start one day in the future.
**When** the start is written to one day from now.
**Then** exactly one notification is produced for that attendee, rendered from the rescheduling
template.

### CAL-AC-054 Moving a meeting into the past notifies nobody

**Given** an all-day meeting whose start day is today and stop day tomorrow, with one attendee.
**When** the start day is written as yesterday.
**Then** no notification is produced.

### CAL-AC-055 Moving somebody else's meeting resets the organiser's answer

**Given** a meeting organised by user A, whose attendee records are A with `accepted` and B with
`accepted`.
**When** user B writes a new start.
**Then** A's response is `needsAction` and B's response is still `accepted`.

### CAL-AC-056 Contacting the attendees of a meeting with none is refused

**Given** a meeting with no attendee contacts.
**When** the composer action is invoked.
**Then** it is refused with "There are no attendees on these events".

### CAL-AC-057 Every invitation carries the interchange attachment

**Given** a meeting from 1 January 2024 at 10:00:00 to 11:00:00 with the video-call address
`https://meet.example.com/test-meeting` and one attendee.
**When** the attendee is invited.
**Then** the message carries one attachment named `invitation.ics`, whose address property is
`https://meet.example.com/test-meeting` and whose location property is absent.

### CAL-AC-058 The interchange file keeps both a location and a video address

**Given** the same meeting with the location "Conference Room A" and the video-call address
`https://meet.example.com/room-a`.
**When** the attendee is invited.
**Then** the attachment's address property is `https://meet.example.com/room-a` and its location
property is "Conference Room A".

### CAL-AC-059 Joining by link adds the joiner

**Given** a meeting whose token is `t0ken` and a signed-in user whose contact is not an attendee.
**When** that user opens `/calendar/meeting/join` with the token `t0ken`.
**Then** the user's contact is now an attendee, an attendee record exists for it, and the user is
redirected to `/calendar/meeting/view` with that record's own invitation token and the meeting
identifier.

---

## 4. Responses

### CAL-AC-070 Accepting from the application posts a message

**Given** an attendee record whose common name is "Wood Corner" and whose response is `needsAction`.
**When** the acceptance operation runs.
**Then** the response is `accepted` and one message exists on the meeting, authored by the contact
"Wood Corner", under the subtype "Invitation", whose body reads "Wood Corner has accepted the
invitation".

### CAL-AC-071 Declining from the application posts a message

**Given** the same record.
**When** the decline operation runs.
**Then** the response is `declined` and the message body reads "Wood Corner has declined the
invitation".

### CAL-AC-072 Answering "Maybe" posts no message

**Given** the same record.
**When** the tentative operation runs.
**Then** the response is `tentative` and no message is posted.

### CAL-AC-073 Accepting from the link without signing in

**Given** an attendee record whose token is `abc` on the meeting whose identifier is 5, with the
response `needsAction`, and no open session.
**When** `/calendar/meeting/accept` is called with the token `abc` and the identifier 5.
**Then** the response becomes `accepted` and the reader is shown the public invitation page with the
badge "Yes I'm going.".

### CAL-AC-074 An unknown token is refused

**Given** no attendee record carries the token `zzz`.
**When** `/calendar/meeting/accept` is called with the token `zzz`.
**Then** the request is refused with "Invalid Invitation Token."

### CAL-AC-075 A forwarded link is refused

**Given** an attendee record whose token is `abc` and whose contact's address is
`wood@example.com`, and a signed-in user whose address is `other@example.com` and whose contact is
not that attendee's contact.
**When** that user calls `/calendar/meeting/accept` with the token `abc`.
**Then** the request is refused with "Invitation cannot be forwarded via email. This event/meeting
belongs to wood@example.com and you are logged in as other@example.com. Please ask organizer to add
you."

### CAL-AC-076 An internal user is redirected rather than shown the page

**Given** an internal user whose contact is the attendee's contact and whose invitation token is `abc`
on the meeting 5.
**When** that user calls `/calendar/meeting/view` with the token `abc` and the identifier 5.
**Then** the reader is redirected to `/app/calendar.event/5` with the database name as a query value.

### CAL-AC-077 Accepting a whole series from the link

**Given** a weekly series of three occurrences, each with an attendee record for the same contact, all
`needsAction`, and the token of the second occurrence's record is `abc`.
**When** `/calendar/recurrence/accept` is called with `abc`.
**Then** all three records are `accepted`.

### CAL-AC-078 Answering with the scope "This and following events"

**Given** a series of three occurrences starting 22 October, 29 October and 5 November 2019, whose
attendee records for the reader are all `needsAction`.
**When** the reader answers "Yes" on the second occurrence with the scope "This and following events".
**Then** the reader's records on the second and third occurrences are `accepted` and the record on the
first is still `needsAction`.

### CAL-AC-079 Answering with the scope "All events"

**Given** the same series.
**When** the reader answers "No" on the second occurrence with the scope "All events".
**Then** all three of the reader's records are `declined`.

### CAL-AC-080 Only the reader's own record changes

**Given** the same series with two attendee contacts.
**When** the reader answers "Yes" with the scope "All events".
**Then** only the three records of the reader's own contact change; the other contact's three records
stay `needsAction`.

### CAL-AC-081 Answering an already-answered invitation from the link changes nothing

**Given** an attendee record already `accepted` with the token `abc`.
**When** `/calendar/meeting/accept` is called with `abc`.
**Then** no record changes and the invitation page is shown.

### CAL-AC-082 The organiser-alone marker

**Given** a meeting whose attendee records are the organiser with `accepted` and two others with
`declined`.
**When** the marker is read.
**Then** it is true. **And when** one of the two others is changed to `tentative`, the marker becomes
false. **And when** the meeting has only the organiser as attendee, the marker is false.

---

## 5. Repetition: generating a series

All scenarios in this group use a base meeting created with the subject "Recurrent Event" and the
series time zone stated in each case.

### CAL-AC-090 Weekly on Tuesdays for three occurrences

**Given** a meeting from 21 October 2019 at 08:00:00 to 23 October 2019 at 18:00:00, marked recurrent.
**When** a weekly pattern on Tuesdays with an interval of 1, a count of 3 and the time zone
coordinated universal time is applied.
**Then** a series exists with three occurrences: 22 October 08:00:00 to 24 October 18:00:00,
29 October 08:00:00 to 31 October 18:00:00 and 5 November 08:00:00 to 7 November 18:00:00. The
original meeting is detached from the series and is archived.

### CAL-AC-091 Weekly with an interval of two

**Given** the same meeting.
**When** a weekly pattern on Tuesdays with an interval of 2 and a count of 2 is applied.
**Then** the occurrences are 22 October 08:00:00 to 24 October 18:00:00 and 5 November 08:00:00 to
7 November 18:00:00.

### CAL-AC-092 The first day of the week does not change the result here

**Given** the same meeting and a language whose week begins on Sunday.
**When** the pattern of CAL-AC-091 is applied.
**Then** the occurrences are the same two.

### CAL-AC-093 Weekly until a day

**Given** the same meeting.
**When** a weekly pattern on Tuesdays with an interval of 2 and an end day of 15 November 2019 is
applied.
**Then** two occurrences exist: 22 October and 5 November, each 08:00:00 to the corresponding
18:00:00.

### CAL-AC-094 Monthly by number, every two months

**Given** the same meeting.
**When** a monthly pattern by number on the twenty-seventh with an interval of 2 and a count of 3 is
applied.
**Then** the occurrences are 27 October 2019 08:00:00 to 29 October 18:00:00, 27 December 2019
08:00:00 to 29 December 18:00:00 and 27 February 2020 08:00:00 to 29 February 18:00:00.

### CAL-AC-095 A day of the month that does not exist is skipped

**Given** the same meeting.
**When** a monthly pattern by number on the thirty-first with an interval of 1 and a count of 3 is
applied.
**Then** the occurrences are 31 October 2019 08:00:00 to 2 November 18:00:00, 31 December 2019
08:00:00 to 2 January 2020 18:00:00 and 31 January 2020 08:00:00 to 2 February 18:00:00; November has
no thirty-first and is skipped.

### CAL-AC-096 Monthly by position, on the third Tuesday

**Given** a meeting from 1 October 2019 at 08:00:00 to 3 October 2019 at 18:00:00.
**When** a monthly pattern by position on the third Tuesday with an interval of 2 and an end day of
27 March 2020 is applied.
**Then** the occurrences are 15 October 2019, 17 December 2019 and 18 February 2020, each 08:00:00 to
two days later at 18:00:00.

### CAL-AC-097 Monthly by position, on the last Wednesday

**Given** a meeting from 21 October 2019 at 08:00:00 to 23 October 2019 at 18:00:00.
**When** a monthly pattern by position on the last Wednesday with an interval of 2 and an end day of
15 January 2020 is applied.
**Then** two occurrences exist: 30 October 2019 08:00:00 to 1 November 18:00:00 and 25 December 2019
08:00:00 to 27 December 18:00:00.

### CAL-AC-098 Yearly, every two years

**Given** the same meeting.
**When** a yearly pattern with an interval of 2 and a count of 2 is applied.
**Then** two occurrences exist, the original instants and the same instants two years later.

### CAL-AC-099 Daily with a fixed count

**Given** nothing.
**When** a meeting is created from 13 April 2011 at 11:04:00 to 12:04:00 with a duration of 1.0, marked
recurrent, with a daily pattern and a count of 5.
**Then** five meetings exist between 13 March 2011 and 13 May 2011.

### CAL-AC-100 Weekly on five weekdays until a day

**Given** nothing.
**When** a meeting is created from 18 April 2011 at 11:47:00 to 12:47:00, marked recurrent, weekly on
Monday, Tuesday, Wednesday, Thursday and Friday, ending on 30 April 2011.
**Then** ten meetings exist between 13 March 2011 and 13 May 2011.

### CAL-AC-101 The daylight-saving rule keeps the clock time

**Given** a meeting from 28 October 2002 at 10:00:00 to 12:00:00.
**When** a weekly pattern on Mondays with an interval of 2 and a count of 2 is applied with a time zone
that left daylight saving time on 27 October 2002.
**Then** the occurrences are 28 October 2002 10:00:00 to 12:00:00 and 11 November 2002 10:00:00 to
12:00:00.

### CAL-AC-102 An ambiguous clock time resolves to standard time

**Given** a meeting whose start is 20 October 2002 at 05:30:00 and whose stop is 06:30:00, which is
01:30 to 02:30 in a zone that leaves daylight saving time on 27 October 2002.
**When** a weekly pattern on Sundays with an interval of 1 and a count of 2 is applied in that zone.
**Then** the occurrences are 20 October 05:30:00 to 06:30:00 and 27 October 06:30:00 to 07:30:00, and
both have a duration of 1.0.

### CAL-AC-103 A clock time that does not exist is pushed forward

**Given** a meeting whose start is 31 March 2002 at 07:30:00 and whose stop is 08:30:00, which is 02:30
to 03:30 in a zone that enters daylight saving time on 7 April 2002 by skipping 02:00 to 03:00.
**When** a weekly pattern on Sundays with an interval of 1 and a count of 2 is applied in that zone.
**Then** the occurrences are 31 March 07:30:00 to 08:30:00 and 7 April 07:30:00 to 08:30:00, and both
have a duration of 1.0.

### CAL-AC-104 An all-day series ignores the clock change

**Given** an all-day meeting from 23 March 2020 at 00:00:00 to 23:59:00.
**When** a weekly pattern on Mondays with a count of 2 is applied in a zone that changes on
23 March 2020.
**Then** the occurrences are 23 March 00:00:00 to 23:59:00 and 30 March 00:00:00 to 23:59:00.

### CAL-AC-105 A backward move across a clock change is abandoned

**Given** the current instant is 27 March 2023, and a meeting whose start is 27 March 2023 at 07:00:00
and whose stop is 08:00:00, which is 09:00 to 10:00 in a zone that changed on 26 March 2023.
**When** a monthly pattern by number on the twenty-seventh with an interval of 1 and a count of 2 is
applied in that zone.
**Then** the occurrences are 27 March 07:00:00 to 08:00:00 and 27 April 07:00:00 to 08:00:00.

### CAL-AC-106 An all-day series keeps the same day each week

**Given** nothing.
**When** an all-day meeting is created on 22 October 2019 with a weekly pattern on Tuesdays, an
interval of 1 and a count of 2.
**Then** the first occurrence's start day is 22 October 2019 and the second's is 29 October 2019.

### CAL-AC-107 An endless daily series is capped at 720

**Given** the horizon is fifteen years.
**When** a meeting from 1 April 2026 at 05:00:00 to 06:00:00 is created with an endless daily pattern.
**Then** 720 meetings exist with that subject.

### CAL-AC-108 An endless monthly series is bounded by the horizon

**Given** the same setup.
**When** the pattern is endless monthly.
**Then** 180 meetings exist. **And when** the horizon is set to five years and the same is repeated,
60 meetings exist.

### CAL-AC-109 An endless yearly series is bounded by the horizon

**Given** the same setup.
**When** the pattern is endless yearly.
**Then** 15 meetings exist. **And when** the horizon is five years, 5 meetings exist.

### CAL-AC-110 An endless weekly series scales with the weekdays and the interval

**Given** the horizon is two years and a meeting from Wednesday 1 April 2026 at 05:00:00 to 06:00:00.
**When** an endless weekly pattern on Wednesdays alone is applied, 104 occurrences exist. **When** the
pattern names Monday, Wednesday and Friday, 311 occurrences exist, the Monday before the start being
dropped. **When** the pattern names Wednesdays with an interval of 2, 52 occurrences exist. **And
when** the horizon is five years and the pattern names all seven weekdays in a language whose week
begins on Sunday, 717 occurrences exist: the generated count is capped at 720 and the three days of
the first week that precede the Wednesday start are dropped.

---

## 6. Repetition: changing a series

Unless stated otherwise the starting series is: a weekly series on Tuesdays with an interval of 1, a
count of 3 and the time zone four hours ahead of coordinated universal time, holding the occurrences
22 October 01:00:00 to 24 October 18:00:00, 29 October 01:00:00 to 31 October 18:00:00 and 5 November
01:00:00 to 7 November 18:00:00, all in 2019.

### CAL-AC-120 Writing a pattern field with the scope "This event" is refused

**Given** a meeting with a pattern of ten daily occurrences.
**When** the pattern text is written on one occurrence with no scope.
**Then** the write is refused with "Unable to save the recurrence with \"This Event\"".

### CAL-AC-121 Shifting from the second occurrence with the scope "This and following events"

**Given** the starting series.
**When** the second occurrence is written with the scope "This and following events", a start four days
later and a stop five days later.
**Then** the original series has the termination mode by end day with the end day 27 October 2019 and
holds one occurrence, 22 October 01:00:00 to 24 October 18:00:00. A new series exists with a count of
2, a first instant of 2 November 01:00:00, the Tuesday marker off and the Saturday marker on, holding
2 November 01:00:00 to 5 November 18:00:00 and 9 November 01:00:00 to 12 November 18:00:00.

### CAL-AC-122 Shifting from the first occurrence replaces the series

**Given** the starting series.
**When** the first occurrence is written with the scope "This and following events", a start four days
later and a stop five days later.
**Then** the original series no longer exists. A new series has a count of 3, a first instant of
26 October 01:00:00, the Saturday marker on, and holds 26 October 01:00:00 to 29 October 18:00:00,
2 November 01:00:00 to 5 November 18:00:00 and 9 November 01:00:00 to 12 November 18:00:00.

### CAL-AC-123 Reapplying the trimmed series is idempotent

**Given** the starting series with the third occurrence shifted by "This and following events".
**When** the original series is applied again.
**Then** it still holds exactly 22 October 01:00:00 to 24 October 18:00:00 and 29 October 01:00:00 to
31 October 18:00:00.

### CAL-AC-124 Shifting the whole series

**Given** the starting series.
**When** the second occurrence is written with the scope "All events", the Tuesday and Friday markers
off, the Saturday marker on, a start four days later and a stop five days later.
**Then** the series holds 26 October 01:00:00 to 29 October 18:00:00, 2 November 01:00:00 to
5 November 18:00:00 and 9 November 01:00:00 to 12 November 18:00:00.

### CAL-AC-125 Shifting only the stop shifts the start too

**Given** the starting series.
**When** the first occurrence is written with the scope "All events" and a stop one hour later.
**Then** the series holds 22 October 02:00:00 to 24 October 19:00:00, 29 October 02:00:00 to
31 October 19:00:00 and 5 November 02:00:00 to 7 November 19:00:00.

### CAL-AC-126 Adding a weekday with the scope "All events"

**Given** the starting series.
**When** the second occurrence is written with the scope "All events" and the Monday marker on.
**Then** the series holds 22 October 01:00:00 to 24 October 18:00:00, 28 October 01:00:00 to
30 October 18:00:00 and 29 October 01:00:00 to 31 October 18:00:00.

### CAL-AC-127 Adding a weekday with the scope "This and following events"

**Given** the starting series.
**When** the second occurrence is written with the scope "This and following events", the Friday marker
on and a count of 4.
**Then** the original series holds one occurrence, 22 October 01:00:00 to 24 October 18:00:00. The new
series has a count of 4, both the Tuesday and the Friday markers on, and holds 29 October, 1 November,
5 November and 8 November, each 01:00:00 to two days later at 18:00:00.

### CAL-AC-128 Renaming with the scope "This and following events"

**Given** the starting series.
**When** the second occurrence is written with the subject "New name", the scope "This and following
events", a daily frequency and a count of 5.
**Then** the original series still exists, the new series has a count of 5, every occurrence of the new
series is named "New name", and the third occurrence of the original series is no longer active.
**And when** the first occurrence of the new series is then written with the subject "Old name" and the
same scope, every occurrence of the new series is named "Old name" and the series still exists.

### CAL-AC-129 Renaming with the scope "All events" replaces the series

**Given** the starting series.
**When** the first occurrence is written with the subject "New name", the scope "All events" and a
count of 5.
**Then** the original series no longer exists, a new series with a count of 5 exists, all of its
occurrences are named "New name" and the other two original occurrences are no longer active.

### CAL-AC-130 Changing the day number of a monthly series with the scope "All events"

**Given** a monthly series by number on the twenty-second with a count of 3 and a time zone four hours
ahead, holding 22 October, 22 November and 22 December 2019, each 01:00:00 to two days later at
18:00:00.
**When** the second occurrence is written with the scope "All events" and the day number 25.
**Then** a series with the day number 25 holds 25 October 01:00:00 to 27 October 18:00:00,
25 November 01:00:00 to 27 November 18:00:00 and 25 December 01:00:00 to 27 December 18:00:00.

### CAL-AC-131 Trimming a monthly series by number

**Given** the same monthly series.
**When** the second occurrence is written with the scope "This and following events", a start four days
later and a stop five days later.
**Then** the original series holds only 22 October 01:00:00 to 24 October 18:00:00, and the new series
holds 26 November 01:00:00 to 29 November 18:00:00 and 26 December 01:00:00 to 29 December 18:00:00.

### CAL-AC-132 Shifting a monthly series by position

**Given** a monthly series by position on the third Tuesday with a count of 3, holding 15 October,
19 November and 17 December 2019, each 01:00:00 to the next day at 18:00:00.
**When** the second occurrence is written with the scope "All events" and both instants five hours
later.
**Then** the series holds 15 October 06:00:00 to 16 October 23:00:00, 19 November 06:00:00 to
20 November 23:00:00 and 17 December 06:00:00 to 18 December 23:00:00.

### CAL-AC-133 A multi-weekday series moved wholesale keeps its spacing

**Given** a weekly series on Tuesdays and Fridays with a count of 3, holding 22 October, 25 October and
29 October 2019, each 01:00:00 to two days later at 18:00:00.
**When** the first occurrence is written with the scope "All events", the Tuesday and Friday markers
off, the Thursday marker on, and both instants two days later.
**Then** the series holds 24 October 01:00:00 to 26 October 18:00:00, 31 October 01:00:00 to
2 November 18:00:00 and 7 November 01:00:00 to 9 November 18:00:00.

### CAL-AC-134 A multi-weekday series moved from its second occurrence

**Given** the same series.
**When** the second occurrence, the Friday, is written with the scope "This and following events" and
both instants three days later.
**Then** the original series still has both the Tuesday and the Friday markers; the new series has the
Tuesday and Monday markers, not the Friday marker, and a count of 2.

### CAL-AC-135 An outlier survives a whole-series shift

**Given** the starting series with the second occurrence already moved on its own to 31 October
01:00:00 to 18:00:00 with the scope "This event".
**When** the first occurrence is written with the scope "All events", the Tuesday and Friday markers
off, the Saturday marker on, and both instants four days later.
**Then** the series holds 26 October 01:00:00 to 28 October 18:00:00, 2 November 01:00:00 to
4 November 18:00:00 and 9 November 01:00:00 to 11 November 18:00:00, and the moved occurrence still
exists.

### CAL-AC-136 A series whose base is archived generates nothing

**Given** the starting series with its base occurrence archived.
**When** the second occurrence is written with the scope "All events" and both instants shifted.
**Then** the series no longer exists and no new one is created.

### CAL-AC-137 A series whose first generated occurrence is archived picks a new base

**Given** a meeting created on Tuesday 22 October 2019 at 01:00:00 to 02:00:00, marked recurrent,
weekly on Wednesdays and Fridays with a count of 3 and a time zone four hours ahead.
**Then** the created meeting is archived, the series' base occurrence is the earliest active
occurrence, and the series holds 23 October, 25 October and 30 October, each 01:00:00 to 02:00:00.
**And when** the first of those is written with the scope "All events" and the Friday marker off,
the series holds 23 October 01:00:00 to 02:00:00, 30 October 01:00:00 to 02:00:00 and
6 November 01:00:00 to 02:00:00.

### CAL-AC-138 Breaking a repetition from the second occurrence

**Given** the starting series.
**When** the second occurrence is written with the scope "This and following events" and the
repetition marker false.
**Then** that occurrence has no series and is active, the first occurrence is active and still in the
series, the third no longer exists, and the series has the termination mode by end day with the end
day 27 October 2019 and holds only 22 October 01:00:00 to 24 October 18:00:00.

### CAL-AC-139 Breaking a repetition for the whole series

**Given** the starting series.
**When** the second occurrence is written with the scope "All events", the repetition marker false and
a count of 0.
**Then** the first and the third occurrences no longer exist, the second is active and has no series,
and the series no longer exists.

---

## 7. Archiving and deleting

### CAL-AC-150 Mass archiving a whole series

**Given** the starting series of group 6.
**When** the second occurrence is mass-archived with the scope "All events".
**Then** all three occurrences have the activity flag false.

### CAL-AC-151 Mass archiving from an occurrence

**Given** the same series.
**When** the second occurrence is mass-archived with the scope "This and following events".
**Then** the first occurrence is still active and the second and third are not.

### CAL-AC-152 Mass deleting a whole series

**Given** the same series.
**When** the second occurrence is mass-deleted with the scope "All events".
**Then** the series and all three occurrences no longer exist.

### CAL-AC-153 Mass deleting from an occurrence

**Given** the same series.
**When** the second occurrence is mass-deleted with the scope "This and following events".
**Then** only the first occurrence remains and the series still exists.

### CAL-AC-154 Deleting the following occurrences from the panel

**Given** the same series.
**When** the first panel is opened on the second occurrence with the scope "Delete this and following
events" and confirmed, and then the confirmation panel is confirmed with "Send and delete".
**Then** the cancellation message is sent, only the first occurrence remains and the series still
exists.

### CAL-AC-155 Deleting the whole series from the panel

**Given** the same series.
**When** the same is done with the scope "Delete all the events".
**Then** the cancellation message is sent and neither the series nor any occurrence remains.

### CAL-AC-156 Deleting a single occurrence of a series

**Given** the same series.
**When** the second occurrence alone is deleted.
**Then** the series holds the first and third occurrences and its base occurrence is unchanged.

### CAL-AC-157 Deleting the base occurrence reselects a base

**Given** the same series.
**When** the first occurrence, which is the base, is deleted.
**Then** the series' base occurrence is the earliest remaining occurrence, 29 October 01:00:00.

### CAL-AC-158 Deleting the last occurrence deletes the series

**Given** a series with one occurrence.
**When** that occurrence is deleted.
**Then** the series no longer exists.

### CAL-AC-159 Archiving with the scope "This event" reselects a base

**Given** the starting series.
**When** the first occurrence is mass-archived with the scope "This event".
**Then** the first occurrence is inactive, the series still exists and its base occurrence is the
second occurrence.

### CAL-AC-160 A reminder in use cannot be deleted

**Given** a reminder attached to at least one meeting.
**When** the reminder record is deleted.
**Then** the deletion is refused by the platform's reference check and the record still exists.

### CAL-AC-161 Deleting a meeting notifies the remaining reminders

**Given** two meetings sharing an attendee, both with an in-application reminder, the second reminder
being the next one due.
**When** the second meeting is deleted.
**Then** a bus message of the kind `calendar.alarm` is pushed to that attendee's contact, holding the
payload of the remaining meeting only.

---

## 8. Reminders

### CAL-AC-170 The lead time in minutes

**Given** nothing.
**When** reminders are created with 15 minutes, 3 hours and 1 day.
**Then** their lead times in minutes are 15, 180 and 1440.

### CAL-AC-171 Choosing the electronic-mail channel attaches the shipped template

**Given** a new reminder.
**When** its channel is set to electronic mail and no template is given.
**Then** its template is the shipped reminder template. **And when** its channel is then set to the
in-application channel, its template is empty.

### CAL-AC-172 The reminder name is rewritten on screen

**Given** a reminder being edited.
**When** the channel is the in-application channel, the duration 30 and the unit minutes.
**Then** the name becomes "Notification - 30 Minutes". **And when** the channel is text message with
the organiser flag on, the name becomes "SMS Text Message - 30 Minutes - Notify Responsible".

### CAL-AC-173 Searching a lead time crosses the units

**Given** a reminder of 1 hour.
**When** reminders with a lead time of at most 90 minutes are searched for.
**Then** that reminder is returned.

### CAL-AC-174 A wake-up is planted at the firing instant

**Given** the current instant is 13 April 2022 at 10:00:00 and the reminder job has never run.
**When** a meeting is created from 10:15:00 to 10:20:00 with a five-minute electronic-mail reminder.
**Then** exactly one wake-up is planted, at 13 April 2022 at 10:10:00.

### CAL-AC-175 A series plants exactly one wake-up

**Given** the same instant and a five-minute electronic-mail reminder.
**When** a meeting is created from 10:15:00 to 10:20:00, marked recurrent, monthly by number on the
thirteenth with a count of 5, carrying that reminder.
**Then** exactly one wake-up is planted, at 13 April 2022 at 10:14:00 when the reminder is one minute,
and running the reminder job at that moment plants no second wake-up.

### CAL-AC-176 The next wake-up of a monthly series

**Given** the series of CAL-AC-175 with a one-minute reminder, whose wake-ups have been cleared, and
the current instant is 16 May 2022 at 10:00:00.
**When** the reminder job runs.
**Then** exactly one wake-up is planted, at 13 June 2022 at 10:14:00.

### CAL-AC-177 The next wake-up of a monthly series with an hour of lead

**Given** the current instant is 16 April 2024 at 10:00:00.
**When** a meeting is created from 12:00:00 to 13:00:00, marked recurrent, monthly by number on the
sixteenth with a count of 2, carrying a one-hour electronic-mail reminder.
**Then** exactly one wake-up is planted, at 16 April 2024 at 11:00:00. **And when** the wake-ups are
cleared and the reminder job runs on 22 April 2024 at 10:00:00, exactly one wake-up is planted, at
16 May 2024 at 11:00:00.

### CAL-AC-178 The next wake-up of a daily series

**Given** the current instant is 13 April 2022 at 10:00:00 and a five-minute electronic-mail reminder.
**When** a meeting is created from 10:15:00 to 10:20:00, marked recurrent, daily with a count of 3.
**Then** one wake-up is planted at 13 April 2022 at 10:10:00. **And when** the reminder job runs at
13 April 2022 at 10:11:00, exactly one wake-up is planted. **And when** it runs again at 14 April 2022
at 10:11:00, exactly one wake-up is planted, at 15 April 2022 at 10:10:00.

### CAL-AC-179 The reminder job sends to the organiser too

**Given** the current instant is a fixed instant, a meeting named "test event" from that instant plus
fifteen minutes to plus eighteen minutes, with an attendee and a twenty-minute electronic-mail
reminder, and the previous run was that instant less twenty-five minutes.
**When** the reminder job runs.
**Then** a message exists on that meeting with the subject "test event - Reminder", and the organiser's
contact is among its recipients.

### CAL-AC-180 A reminder attached after the previous run is skipped for the past

**Given** the previous run was one hour ago and a meeting starting in five minutes.
**When** a thirty-minute electronic-mail reminder is attached now and the reminder job runs.
**Then** no reminder message is produced for that meeting, because the firing instant lies before the
previous run.

### CAL-AC-181 A meeting that has already ended sends no reminder

**Given** a meeting whose stop is one minute ago and whose firing instant falls inside the window.
**When** the reminder job runs.
**Then** no message is produced for it.

### CAL-AC-182 A declined attendee receives no reminder

**Given** a meeting with two attendees, one `accepted` and one `declined`, and a due reminder.
**When** the reminder job runs.
**Then** exactly one message is produced, for the accepted attendee.

### CAL-AC-183 The in-application reminder payload

**Given** the current instant is a fixed instant, a meeting named "Doom's day" from that instant plus
fifty minutes to plus fifty-five minutes, with the reader as attendee and a thirty-minute
in-application reminder.
**When** the meeting is written.
**Then** a bus message of the kind `calendar.alarm` is pushed to the reader's contact whose single
entry holds the reminder's identifier, the meeting's identifier, the title "Doom's day", the meeting's
display time as the message, a timer of 1200 seconds and a firing instant equal to the fixed instant
plus twenty minutes.

### CAL-AC-184 The organiser also receives the bus payload

**Given** an administrator who is both organiser and only attendee of a meeting from the current
instant plus fifty minutes to plus fifty-five minutes.
**When** a thirty-minute in-application reminder is attached.
**Then** the bus payload is pushed to that administrator's contact with a timer of 1200 seconds.

### CAL-AC-185 The acknowledgement suppresses a reminder

**Given** the payload of CAL-AC-183 has been delivered.
**When** the reader calls the acknowledgement route and then the polling route.
**Then** the acknowledgement instant on the reader's contact is the current instant and the polling
route returns nothing for that reminder.

### CAL-AC-186 The systray shows today's meetings in the reader's own day

**Given** the reader's time zone is one hour ahead and two meetings exist: one from 15 November 2023 at
18:00:00 to 19:00:00 and one from 15 November 2023 at 23:00:00 to 16 November at 00:00:00.
**When** the systray query runs at 17:30:00, at 18:00:00, at 18:30:00 and at 19:00:00 on 15 November.
**Then** each time it returns exactly the first meeting. **And when** it runs at 19:30:00, it returns
nothing.

---

## 9. Privacy and access

Group 9 uses four internal users, John, Raoul, George and an administrator, and one portal user. John
organises the meetings and George is always an attendee.

### CAL-AC-200 The visibility policy itself is public

**Given** a private meeting organised by John.
**When** John, George and Raoul read the visibility policy.
**Then** all three read `private`. **When** the portal user reads it, the platform's access refusal is
raised.

### CAL-AC-201 A private subject is substituted for uninvited readers

**Given** a private meeting named "my private event" organised by John with George as attendee.
**When** John and George read the subject, they read "my private event". **When** Raoul reads it, he
reads "Busy". **When** the portal user reads it, the platform's access refusal is raised.

### CAL-AC-202 The display name is substituted too

**Given** the same meeting.
**When** Raoul reads the display name.
**Then** he reads "Busy".

### CAL-AC-203 A private non-substituted field reads empty

**Given** the same meeting with the location "in the Sky".
**When** John and George read the location, they read "in the Sky". **When** Raoul reads it, he reads
an empty value.

### CAL-AC-204 A private attendee set reads empty

**Given** the same meeting.
**When** Raoul reads the attendee contacts.
**Then** he reads an empty set.

### CAL-AC-205 A public field of a private meeting is not obfuscated

**Given** a private meeting with the location "in the Sky" and a public meeting with the location "In
Hell", both organised by John.
**When** Raoul reads the locations of both in one read.
**Then** the first is empty and the second is "In Hell".

### CAL-AC-206 Grouped reads hide other people's private meetings

**Given** a private meeting organised by John.
**When** Raoul groups meetings by subject over that meeting alone.
**Then** the result is empty. **When** Raoul groups a public meeting by subject, by month or by week,
the result is not empty.

### CAL-AC-207 An organiser whose calendar default is private makes the meeting private

**Given** John's calendar default privacy is `private` and a meeting organised by John carries no
policy.
**When** Raoul reads its subject.
**Then** he reads "Busy".

### CAL-AC-208 An administrator may not read another person's private meeting

**Given** a private meeting organised by John to which the administrator is not invited.
**When** the administrator reads the subject.
**Then** the administrator reads "Busy" and the sensitive fields read empty.

### CAL-AC-209 An administrator may not edit another person's private meeting

**Given** the same meeting.
**When** the administrator writes its location.
**Then** the write is refused with the platform's access refusal for the write operation.

### CAL-AC-210 An administrator may edit a public or confidential meeting they are not invited to

**Given** a public meeting and a confidential meeting organised by John.
**When** the administrator writes the location of each.
**Then** both writes succeed.

### CAL-AC-211 An uninvited reader may not add attendees to a private meeting

**Given** a private meeting organised by John.
**When** Raoul writes its attendee contacts.
**Then** the write is refused with the platform's access refusal.

### CAL-AC-212 A user may not change another user's default privacy

**Given** two users.
**When** one writes the other's calendar default privacy.
**Then** the write is refused with "You are not allowed to change the calendar default privacy of
another user due to privacy constraints."

### CAL-AC-213 A user may change their own default privacy

**Given** an ordinary internal user.
**When** that user writes their own calendar default privacy to `private`.
**Then** the value is stored on their settings record and applies to their meetings that carry no
policy.

### CAL-AC-214 A non-administrator may change a repeating meeting with reminders

**Given** a series created by an ordinary internal user with a reminder attached.
**When** that user writes a new subject with the scope "All events".
**Then** the change succeeds and every occurrence of the rebuilt series carries the new subject and the
same reminder.

---

## 10. Video calls

### CAL-AC-220 Asking for a platform-hosted call fills the address

**Given** a meeting with no video-call address and no token.
**When** the platform-hosted call is requested.
**Then** the meeting has a thirty-two character hexadecimal token and its address is the platform's
base address followed by `/calendar/join_videocall/` and that token, and its hosting selector reads
`discuss`.

### CAL-AC-221 No channel exists until somebody joins

**Given** the meeting of CAL-AC-220.
**When** nothing else happens.
**Then** the meeting has no conversation channel.

### CAL-AC-222 The first join creates the channel

**Given** the same meeting.
**When** the joining route is called with its token.
**Then** the meeting has a conversation channel and the caller is redirected to that channel's
invitation address.

### CAL-AC-223 Adding attendees adds them to the channel

**Given** a meeting with a live channel and two new contacts.
**When** the two contacts are added as attendees.
**Then** both are members of the channel.

### CAL-AC-224 Every occurrence of a series has its own address

**Given** a meeting with a platform-hosted call.
**When** a weekly pattern on Mondays with a count of 2 is applied.
**Then** every occurrence has a different video-call address and none has a channel.

### CAL-AC-225 The channel is shared across the series

**Given** the series of CAL-AC-224.
**When** the channel of one occurrence is created.
**Then** every other occurrence of the series points at the same channel.

### CAL-AC-226 A meeting's channel never rings

**Given** a meeting with a channel.
**When** a member joins the call.
**Then** the other members are not rung.

---

## 11. Availability and working hours

### CAL-AC-230 One meeting makes nobody unavailable

**Given** contacts A and B and one meeting from 13 December 2020 at 17:00:00 to 22:00:00 with both as
attendees.
**When** the unavailable set is read.
**Then** it is empty.

### CAL-AC-231 A second overlapping meeting makes the shared attendee unavailable

**Given** the meeting of CAL-AC-230 and a second meeting over the same span with contact A alone.
**When** the unavailable sets of both meetings are read.
**Then** both hold exactly contact A.

### CAL-AC-232 Meetings that merely touch do not overlap

**Given** contact A attending a meeting from 09:00:00 to 10:00:00 and another from 10:00:00 to
11:00:00.
**When** the unavailable sets are read.
**Then** both are empty.

### CAL-AC-233 A meeting marked available does not occupy anybody

**Given** the pair of CAL-AC-231 with the second meeting's availability marker set to `free`.
**When** the unavailable sets are read.
**Then** both are empty.

### CAL-AC-234 The interval of a timed meeting is its own span

**Given** a company working Monday to Friday from 08:00 to 12:00 and 13:00 to 16:00 in a zone two hours
ahead.
**When** the interval of a meeting from 12 July 2024 at 08:30:00 to 09:30:00 is computed.
**Then** it is exactly 08:30:00 to 09:30:00 in coordinated universal time.

### CAL-AC-235 The interval of an all-day meeting is the company's working hours

**Given** the same company.
**When** the interval of an all-day meeting on Friday 12 July 2024 is computed.
**Then** it is 08:00 to 12:00 and 13:00 to 16:00 local time, seven hours in all.

### CAL-AC-236 An all-day meeting on a closed day has an empty interval

**Given** the same company.
**When** the interval is computed for an all-day meeting on Saturday 13 July 2024, for one from Friday
12 to Saturday 13 July, for one from Sunday 14 to Monday 15 July and for one from Friday 12 to Monday
15 July.
**Then** all four intervals are empty, because at least one day of each range has no working hours.

### CAL-AC-237 A meeting with no length has an empty interval

**Given** the same company.
**When** the interval is computed for a meeting whose start and stop are both 12 July 2024 at
00:00:00.
**Then** it is empty. **And when** the same instants are used with the all-day marker, the interval is
the company's seven working hours of that day.

### CAL-AC-238 An attendee outside the working hours is unavailable

**Given** the interval of CAL-AC-235 and two attendees, one working Tuesday to Friday from 08:00 to
12:00 and 13:00 to 16:00 and one working Monday to Friday from 15:00 to 22:00, both in the same zone.
**When** the unavailable set is read.
**Then** it holds exactly the second attendee.

### CAL-AC-239 The shading payload of two identical schedules

**Given** two employees sharing the schedule Monday to Friday from 08:00 to 12:00 and 13:00 to 16:00 in
a zone two hours ahead, and the reader in that same zone.
**When** the working hours common to both are asked for over one week.
**Then** ten entries are returned; the Monday morning entry holds the weekday number 1, the start time
"08:00" and the end time "12:00". **And when** the two employees have wholly disjoint schedules, one
entry is returned holding the weekday number 7 and both times "00:00".

---

## 12. Activities and linked documents

### CAL-AC-250 A meeting created from an activity carries that activity

**Given** an activity of the meeting category on a contact, with the note "Discuss pricing", the
deadline 20 April 2024 and the assignee user A.
**When** the scheduling button is pressed and the meeting is saved with the start 20 April 2024 at
14:00:00.
**Then** the meeting's activity set holds that activity and no second activity exists.

### CAL-AC-251 The deadline follows the start, in the reader's zone

**Given** the reader's time zone is thirteen hours ahead and a meeting whose start is 20 April 2024 at
14:00:00 in coordinated universal time, linked to an activity.
**When** the deadline is derived.
**Then** it is 21 April 2024, because the start falls on the twenty-first in the reader's zone.

### CAL-AC-252 The deadline of an all-day meeting is its start day

**Given** an all-day meeting starting 20 April 2024 linked to an activity, and any reader time zone.
**When** the deadline is derived.
**Then** it is 20 April 2024.

### CAL-AC-253 Moving the activity moves the meeting

**Given** a timed meeting starting 20 April 2024 at 14:00:00 linked to an activity whose deadline is
20 April 2024, with the reader in coordinated universal time.
**When** the activity's deadline is written as 22 April 2024.
**Then** the meeting's start becomes 22 April 2024 at 14:00:00.

### CAL-AC-254 Moving an all-day meeting's activity moves it by the day difference

**Given** an all-day meeting whose start day is 20 April 2024 linked to an activity with the same
deadline.
**When** the deadline is written as 23 April 2024.
**Then** the meeting's start moves forward by three days.

### CAL-AC-255 Moving the meeting moves the activity

**Given** the pair of CAL-AC-253.
**When** the meeting's start is written as 25 April 2024 at 09:00:00.
**Then** the activity's deadline becomes 25 April 2024 and no further change is triggered.

### CAL-AC-256 Completing the activity records the feedback on the meeting

**Given** the same pair, with the meeting's internal notes empty.
**When** the activity is marked done with the feedback "Agreed on the discount".
**Then** the meeting's internal notes read a line break followed by "Feedback: Agreed on the discount"
turned into markup, and the meeting's description is unchanged.

### CAL-AC-257 Scheduling a meeting on more than one document is refused

**Given** the scheduling wizard addressing two documents.
**When** the meeting-creating action is invoked.
**Then** it is refused with "Scheduling an activity using the calendar is not possible on more than
one record."

### CAL-AC-258 The next activity's meeting is exposed on the document

**Given** a document with two activities, the first of which points at a meeting.
**When** the document's next-activity meeting is read.
**Then** it is that meeting. **And when** no activity points at a meeting, it is empty.

---

## 13. Tags, side lists and configuration

### CAL-AC-270 A tag gets a colour

**Given** nothing.
**When** a tag named "Internal" is created with no colour.
**Then** its colour index is a whole number between 1 and 11 inclusive.

### CAL-AC-271 A duplicate tag is refused

**Given** a tag named "Internal".
**When** a second tag named "Internal" is created.
**Then** the creation is refused with "Tag name already exists!"

### CAL-AC-272 A side-list row is unique per user and contact

**Given** a side-list row for user A and contact B.
**When** a second row for the same pair is created.
**Then** the creation is refused with "A user cannot have the same contact twice."

### CAL-AC-273 Removing a contact clears every side list

**Given** three users each holding a side-list row for contact B.
**When** the clearing operation is called for contact B.
**Then** all three rows are deleted.

### CAL-AC-274 The provider panel installs and configures the first service

**Given** the first service's capability package is not installed, and an administrator.
**When** the panel is confirmed with the first service chosen, the identifier "abc" and the secret
"xyz".
**Then** the package is installed, `google_calendar_client_id` is "abc",
`google_calendar_client_secret` is "xyz", `google_calendar_sync_paused` holds the switch's value, and
none of the second service's parameters changed.

### CAL-AC-275 The provider panel refuses a non-administrator

**Given** an ordinary internal user.
**When** the panel's action is invoked.
**Then** it is refused with the platform's administrative-access refusal.

### CAL-AC-276 A new user inherits the platform default privacy

**Given** the parameter `calendar.default_privacy` holds `private`.
**When** a new internal user is created without a calendar default privacy.
**Then** that user's calendar default privacy is `private`.

### CAL-AC-277 A portal user gets no settings record

**Given** the same parameter.
**When** a portal user is created.
**Then** no user settings record is created for it and its calendar default privacy falls back to the
platform default.

### CAL-AC-278 A portal user only sees their own meetings

**Given** two meetings, one with the portal user as attendee and one without.
**When** the portal user reads meetings.
**Then** exactly the first is returned, and writing it is refused.

---

## 14. The first external calendar service

Group 14 assumes the first service's package is installed, its registration is complete, the reader
holds a valid credential and the service is not suspended, unless a scenario says otherwise.

### CAL-AC-300 Creating a meeting inserts it outwards

**Given** the reader is connected.
**When** a meeting from 16 April 2024 at 09:00:00 to 10:00:00 is created.
**Then** one insertion is made after the transaction commits, the returned identifier is stored on the
meeting and its pending flag is lowered.

### CAL-AC-301 Creating a meeting for a user who has stopped does not insert it

**Given** the organiser has stopped synchronising.
**When** a meeting is created for that organiser.
**Then** the meeting's pending flag is false and no insertion is made.

### CAL-AC-302 Writing a synchronised field patches the remote copy

**Given** a meeting with a stored identifier.
**When** its subject is written.
**Then** its pending flag is raised, a patch is made after the commit, and the flag is lowered again.

### CAL-AC-303 Writing an unsynchronised field patches nothing

**Given** the same meeting.
**When** its internal notes are written.
**Then** the pending flag stays false and no call is made.

### CAL-AC-304 Deleting a synchronised meeting archives it

**Given** a meeting with a stored identifier.
**When** it is deleted.
**Then** the record still exists with the activity flag false, and the next run removes the remote
copy.

### CAL-AC-305 Deleting an unsynchronised meeting removes it

**Given** a meeting with no stored identifier.
**When** it is deleted.
**Then** the record no longer exists and no call is made.

### CAL-AC-306 A recurring meeting is pushed as a series

**Given** the reader is connected.
**When** a meeting is created with a weekly pattern and a count of 3.
**Then** the three occurrences all have their pending flag false, and exactly one insertion is made,
for the series.

### CAL-AC-307 An occurrence added later is pushed as part of the series

**Given** the series of CAL-AC-306.
**When** the pattern's count is raised to 4 with the scope "All events".
**Then** the series is marked pending and one patch is made; no individual occurrence is inserted.

### CAL-AC-308 Only synchronised fields raise the flag

**Given** a meeting with a stored identifier and a pending flag of false.
**When** the description, the location, the availability marker, the visibility policy, the reminders,
the attendees, the subject, the start, the stop, the all-day marker, the activity flag or the
video-call address is written, the flag is raised. **When** any other field is written, it is not.

### CAL-AC-309 A guest may not change a protected meeting

**Given** a meeting whose guest-modification marker is set and whose organiser is another user.
**When** the reader writes its subject.
**Then** the write is refused with "The following event can only be updated by the organizer according
to the event permissions set on Google Calendar."

### CAL-AC-310 Restarting synchronisation marks everything pending

**Given** the reader has stopped synchronising and holds five meetings inside the outward scope with
the pending flag false.
**When** the reader restarts synchronisation.
**Then** the stopped flag is false and all five have the pending flag true.

### CAL-AC-311 Suspension stops every call

**Given** `google_calendar_sync_paused` is true.
**When** a meeting is created, then updated, then deleted.
**Then** the created meeting keeps the pending flag true and no insertion is made; the update keeps the
flag true and no patch is made; the deletion archives the meeting locally and no deletion is made.
**And when** the parameter is set to false and a run happens, the backlog is pushed.

### CAL-AC-312 A new remote meeting is created locally

**Given** the reader's calendar holds a new remote meeting with the subject "Remote", a start of
16 April 2024 at 09:00:00 with a clock and an end of 10:00:00.
**When** a run happens.
**Then** a local meeting exists with that subject, that start, that stop, the all-day marker false, the
stored remote identifier and the pending flag false.

### CAL-AC-313 A new remote all-day meeting is created locally

**Given** a remote all-day meeting whose start day is 16 April 2024 and whose end day is
18 April 2024.
**When** a run happens.
**Then** the local meeting has the all-day marker true, a start of 16 April 2024 at 08:00:00 and a stop
of 17 April 2024 at 18:00:00, because the remote end day is exclusive.

### CAL-AC-314 An inclusive remote end day is tolerated

**Given** a remote all-day meeting whose start day and end day are both 16 April 2024.
**When** a run happens.
**Then** the local stop is 16 April 2024 at 18:00:00, because subtracting one day would put the stop
before the start.

### CAL-AC-315 A remote meeting with no title gets a fallback

**Given** a remote meeting with no summary and no matching local meeting.
**When** a run happens.
**Then** the local subject is "(No title)".

### CAL-AC-316 A cancellation by the organiser deletes the meeting locally

**Given** a synchronised meeting the reader organises.
**When** the run reports it as cancelled.
**Then** the local record no longer exists and no outward deletion is attempted.

### CAL-AC-317 A cancellation of somebody else's meeting declines instead

**Given** a synchronised meeting organised by another user, with the reader as attendee.
**When** the run reports it as cancelled.
**Then** the meeting still exists and the reader's own attendee record is `declined`.

### CAL-AC-318 A birthday entry is ignored

**Given** the remote answer holds one entry whose kind is a birthday.
**When** a run happens.
**Then** no local meeting is created for it.

### CAL-AC-319 An older remote change does not overwrite a newer local one

**Given** a synchronised meeting whose local write instant is 16 April 2024 at 12:00:00 and a remote
change whose modification instant is 16 April 2024 at 11:00:00.
**When** a run happens.
**Then** the local meeting is unchanged.

### CAL-AC-320 A newer remote change wins

**Given** the same meeting and a remote modification instant of 16 April 2024 at 13:00:00.
**When** a run happens.
**Then** the remote values are written locally and the pending flag is lowered.

### CAL-AC-321 A remote series becomes a local series

**Given** a remote series repeating weekly on Mondays four times, starting 15 April 2024 at 09:00:00.
**When** a run happens.
**Then** one local series exists with four occurrences, whose base occurrence is the first, and each
occurrence's derived identifier is the series identifier, an underscore and the compact start instant
with a trailing `Z`.

### CAL-AC-322 A remote occurrence removed from a series is removed locally

**Given** the series of CAL-AC-321.
**When** the run reports the second occurrence as cancelled.
**Then** that occurrence no longer exists and the other three remain.

### CAL-AC-323 A single meeting turned into a series remotely is reused

**Given** a synchronised single meeting whose identifier is `xyz`.
**When** the run reports a series whose identifier is also `xyz`.
**Then** the existing meeting is reused as the base occurrence of the new local series, its own stored
identifier is cleared and the occurrences are generated.

### CAL-AC-324 A synchronised meeting that becomes a local series is withdrawn as a single meeting

**Given** a synchronised meeting whose identifier is `xyz`.
**When** a weekly pattern is applied to it.
**Then** an inactive copy carrying the identifier `xyz` is created with the pending flag true, the
remote copy is deleted at once, and the meeting's own identifier is cleared so that it takes the
series-derived one.

### CAL-AC-325 An attendee does not insert the organiser's meeting

**Given** a meeting whose organiser is another user who is not connected.
**When** the reader, who is an attendee, runs a synchronisation.
**Then** no insertion is made and the meeting stays pending.

### CAL-AC-326 A rejected call posts a note and stops retrying

**Given** a meeting whose insertion the service rejects with a forbidden answer mentioning a
non-organiser restriction.
**When** the call runs.
**Then** the meeting's pending flag is lowered and a note is posted on it reading "The following event
could not be synced with Google Calendar." then a line break, "It will not be synced as long at it is
not updated." then a line break, then "you don't seem to have permission to modify this event on
Google Calendar".

### CAL-AC-327 Resetting an account with the policy to leave events and synchronise all

**Given** a user with three synchronised meetings and a stored credential.
**When** the reset panel is confirmed with "Leave them untouched" and "Synchronize all existing
events".
**Then** the three meetings still exist with their identifiers, all three have the pending flag true,
and the user's credentials, change marker and last-calendar marker are empty.

### CAL-AC-328 Resetting an account deleting on both sides

**Given** the same user.
**When** the panel is confirmed with "Delete from both" and "Synchronize only new events".
**Then** each of the three meetings is deleted at the service, all three have their identifier cleared
and are then deleted locally, and the credentials are cleared.

### CAL-AC-329 A meeting that is over sends no update invitations

**Given** a synchronised meeting whose stop is one hour in the past.
**When** its subject is written.
**Then** the patch is made with the service asked not to notify the attendees. **And when** the stop is
one hour in the future, the patch asks the service to notify them.

---

## 15. The second external calendar service

Group 15 assumes the second service's package is installed, its registration is complete, the reader
holds a valid credential and the service is not suspended, unless a scenario says otherwise.

### CAL-AC-340 Creating a meeting inserts it and stores both identifiers

**Given** the reader is connected.
**When** a meeting is created.
**Then** one insertion is made after the commit, both the calendar-scoped and the shared identifier are
stored and the pending flag is lowered.

### CAL-AC-341 Creating a meeting without the service enabled makes no call

**Given** the reader holds no credential.
**When** a meeting is created.
**Then** the local meeting exists and no call is made.

### CAL-AC-342 An attendee without an address blocks the batch

**Given** a meeting with two attendees, one of whom has no address.
**When** an outward call is attempted.
**Then** it is refused with the message of CAL-145, listing that meeting with its display time and its
display name, preceded by a tab and a hyphen, and the count in parentheses.

### CAL-AC-343 Creating a series from the platform is refused

**Given** the reader is connected.
**When** a meeting with a repetition marker is created.
**Then** the creation is refused with "Due to an Outlook Calendar limitation, recurrent events must be
created directly in Outlook Calendar."

### CAL-AC-344 Creating a series while not connected is allowed

**Given** the reader holds no credential.
**When** a meeting with a weekly pattern and a count of 3 is created.
**Then** the series and its three occurrences exist and no call is made.

### CAL-AC-345 Changing a synchronised series is refused

**Given** a synchronised series whose occurrences carry the series-head reference.
**When** an occurrence is written with the scope "All events".
**Then** the write is refused with "Due to an Outlook Calendar limitation, recurrence updates must be
done directly in Outlook Calendar."

### CAL-AC-346 The longer refusal appears when an occurrence is unsynchronised

**Given** the same series with one occurrence carrying no shared identifier.
**When** the same write is attempted.
**Then** the refusal reads "Due to an Outlook Calendar limitation, recurrence updates must be done
directly in Outlook Calendar.\nIf this recurrence is not shown in Outlook Calendar, you must delete it
in Odoo Calendar and recreate it in Outlook Calendar."

### CAL-AC-347 Deleting occurrences of a synchronised series from the list is refused

**Given** the same series.
**When** two of its occurrences are deleted at once.
**Then** the deletion is refused with the message of CAL-AC-345.

### CAL-AC-348 An occurrence may not jump over its neighbours

**Given** a synchronised series with occurrences on 1, 8 and 15 May 2024.
**When** the occurrence of 8 May is written with the scope "This event" and a start of 16 May 2024.
**Then** the write is refused with "Outlook limitation: in a recurrence, an event cannot be moved to or
before the day of the previous event, and cannot be moved to or after the day of the following event."
**And when** it is written with a start of 10 May 2024, the write succeeds.

### CAL-AC-349 A meeting that becomes recurrent is withdrawn first

**Given** a synchronised single meeting.
**When** its repetition marker is set.
**Then** its remote copy is deleted with a three-second budget and both identifiers are cleared before
the series is built.

### CAL-AC-350 A new remote meeting is created locally

**Given** the reader's calendar holds a new remote meeting with the subject "Remote", a start of
16 April 2024 at 09:00:00 in the zone the entry names and an end of 10:00:00.
**When** a run happens.
**Then** a local meeting exists with that subject and those instants read in that zone, both
identifiers stored and the pending flag false.

### CAL-AC-351 A remote all-day meeting loses one day at the end

**Given** a remote all-day meeting whose start is 16 April 2024 and whose end is 18 April 2024.
**When** a run happens.
**Then** the local stop is 17 April 2024, because the remote end is exclusive.

### CAL-AC-352 A remote meeting is matched by its shared identifier

**Given** a local meeting whose shared identifier is `u1` and whose calendar-scoped identifier is `a1`.
**When** the run returns an entry whose shared identifier is `u1` and whose scoped identifier is `b1`.
**Then** the entry is matched to that meeting and its values are applied.

### CAL-AC-353 A remote meeting matched by its scoped identifier gains the shared one

**Given** a local meeting whose scoped identifier is `a1` and whose shared identifier is empty.
**When** the run returns an entry whose scoped identifier is `a1` and whose shared identifier is `u1`.
**Then** the local meeting's shared identifier becomes `u1`, its scoped identifier stays `a1` and its
pending flag is false.

### CAL-AC-354 An entry that matches nothing is created

**Given** no local meeting carries either identifier of a returned entry.
**When** a run happens.
**Then** a new local meeting is created from it.

### CAL-AC-355 A removed entry cancels the local record

**Given** a synchronised meeting the reader organises.
**When** the run returns that entry with a removal marker whose reason is deletion.
**Then** both identifiers are cleared and the local record is deleted, with no outward call.

### CAL-AC-356 A removed entry the reader neither owns nor attends declines instead

**Given** a synchronised meeting organised by somebody else, with the reader neither organiser nor
attendee contact.
**When** the run reports it as cancelled.
**Then** the meeting survives and the reader's own attendee record, if any, is `declined`.

### CAL-AC-357 A remote series creates a local series with its occurrences

**Given** a remote series head repeating weekly with four instances.
**When** a run happens.
**Then** one local series and four occurrences exist, the base occurrence is the first, the pattern's
frequency, interval, termination mode and weekday markers match the remote pattern, and every record
carries both identifiers.

### CAL-AC-358 A single meeting turned into a series remotely leaves no duplicate

**Given** a synchronised single meeting whose shared identifier is `u1`.
**When** the run returns a series head whose shared identifier is also `u1`.
**Then** the local series is created and the old single meeting is removed after having both its
identifiers cleared, so no duplicate remains beside the first occurrence.

### CAL-AC-359 An attendee accepts from the platform

**Given** a synchronised meeting organised by another user, with the reader as attendee.
**When** the reader accepts.
**Then** the reader's own scoped identifier is fetched by the shared identifier, and the answer
`accept` is sent with an empty comment and a request to send a response.

### CAL-AC-360 The organiser does not answer through the service

**Given** a synchronised meeting the reader organises.
**When** the reader accepts.
**Then** the local response changes and no answer call is made.

### CAL-AC-361 Changing the organiser requires the new organiser to be connected

**Given** a synchronised meeting and a proposed organiser who is not connected while the reader is.
**When** the organiser is written.
**Then** the write is refused with "For having a different organizer in your event, it is necessary
that the organizer have its Odoo Calendar synced with Outlook Calendar."

### CAL-AC-362 Changing the organiser requires them to be an attendee

**Given** the same meeting and a proposed organiser who is connected but is not an attendee.
**When** the organiser is written.
**Then** the write is refused with "It is necessary adding the proposed organizer as attendee before
saving the event."

### CAL-AC-363 A valid organiser change recreates the meeting

**Given** the same meeting, a proposed organiser who is connected and is an attendee.
**When** the organiser is written.
**Then** a new meeting is created under the new organiser with the same values and no scoped
identifier, the original is archived, and the original's remote copy is deleted.

### CAL-AC-364 Suspension stops every call

**Given** `microsoft_calendar_sync_paused` is true.
**When** a meeting is created, then updated, then deleted.
**Then** the created meeting keeps the pending flag true and no insertion is made; the updated meeting
keeps the flag true; the deleted meeting is removed locally with no remote deletion.

### CAL-AC-365 A first connection ignores earlier meetings

**Given** no user has ever synchronised with the second service, the parameter
`microsoft_calendar.sync.first_synchronization_date` is unset, and the reader already holds ten
meetings.
**When** the reader runs a synchronisation at 16 April 2024 at 10:00:00.
**Then** the parameter becomes 16 April 2024 at 09:59:00 and none of the ten meetings, all created
earlier, is pushed.

### CAL-AC-366 Resetting an account leaves series alone

**Given** a user with two single synchronised meetings and one synchronised series of three
occurrences.
**When** the reset panel is confirmed with "Delete from the current Microsoft Calendar account" and
"Synchronize only new events".
**Then** the two single meetings are deleted at the service, the three occurrences of the series are
not, and the credentials, the change marker and the last-synchronisation instant are cleared.

---

## 16. Text-message reminders

### CAL-AC-380 Only contacts with a usable number are messaged

**Given** a meeting with two attendee contacts, one with the telephone number `0477777777` in a country
whose code makes it valid and one with no number, and a text-message reminder.
**When** the reminder is produced for that meeting.
**Then** exactly one text message is produced.

### CAL-AC-381 Each reminder uses its own template

**Given** the current instant is a fixed instant; a one-hour reminder whose template body reads
"Reminder: Your event is starting in 1 hour!" and a twenty-four-hour reminder whose template body
reads "Reminder: Your event is starting in 24 hour!"; a meeting starting in thirty minutes with
contact one, a copy of it with contact three, and a meeting starting in twenty-three hours and thirty
minutes with contact two; all three meetings carrying both reminders.
**When** the reminder job runs with a previous run one hour ago.
**Then** exactly three text messages are produced: to contact one and to contact three with the
one-hour body, and to contact two with the twenty-four-hour body.

### CAL-AC-382 A declined attendee gets no text message

**Given** a meeting with two attendees with numbers, one of whom declined.
**When** the text-message reminder is produced.
**Then** exactly one message is produced, for the attendee who did not decline.

### CAL-AC-383 The organiser is excluded unless the flag is set

**Given** a meeting whose organiser's contact has a number, plus one other attendee with a number, and
a text-message reminder whose organiser flag is off.
**When** the reminder is produced.
**Then** exactly one message is produced, for the other attendee. **And when** the flag is on, two
messages are produced.

### CAL-AC-384 A reminder with no template uses the fallback text

**Given** a text-message reminder whose template has been cleared, and a meeting named "Standup".
**When** the reminder is produced.
**Then** the message body reads "Event reminder: Standup, " followed by the meeting's display time and
a full stop.
