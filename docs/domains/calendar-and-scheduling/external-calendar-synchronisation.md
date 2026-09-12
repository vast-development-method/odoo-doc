# External calendar synchronisation

## 1. Why this document exists

Two of the nine capability packages of this domain exist only to keep the platform's meetings in step
with an external calendar service, and two more supply the delegated-authorisation plumbing they use.
Together they are larger than the calendar itself, and their behaviour is not a variation on the
calendar's behaviour: it is a distributed-state problem with its own identity model, its own conflict
rule, its own failure handling and two incompatible remote data models.

This document specifies that behaviour completely. The eleven ordinary documents of the folder refer
to it rather than repeat it.

Throughout, **the first external calendar service** is the service whose stored selection value is
`google` and whose capability-package identifiers all begin with `google_`, and **the second external
calendar service** is the one whose stored selection value is `microsoft` and whose identifiers begin
with `microsoft_`. Addresses, path fragments and property names of those services are reproduced in
code font because they are part of the integration contract; changing one of them would break the
integration.

Contents:

1. [Why this document exists](#1-why-this-document-exists)
2. [The shared contract](#2-the-shared-contract)
3. [Identity](#3-identity)
4. [Outward to the first service](#4-outward-to-the-first-service)
5. [Inward from the first service](#5-inward-from-the-first-service)
6. [Series in the first service](#6-series-in-the-first-service)
7. [Outward to the second service](#7-outward-to-the-second-service)
8. [Inward from the second service](#8-inward-from-the-second-service)
9. [Series in the second service](#9-series-in-the-second-service)
10. [Ownership and blocked insertions](#10-ownership-and-blocked-insertions)
11. [Errors and how they are reported](#11-errors-and-how-they-are-reported)
12. [The two services side by side](#12-the-two-services-side-by-side)
13. [Transport addresses](#13-transport-addresses)

---

## 2. The shared contract

Both packages express themselves as a behaviour that Calendar Event and Event Recurrence Rule adopt.
The behaviour supplies four things and demands seven.

**What the behaviour supplies**

| Supplied | First service | Second service |
|---|---|---|
| A remote identifier | `google_id` | `microsoft_id` and `ms_universal_event_id` |
| A pending flag | `need_sync` | `need_sync_m` |
| An activity flag | `active` | `active` |
| Wrapped creation, writing, deletion and cancellation | yes | yes |

**What each adopting entity must supply**

1. The values to write locally from a remote entry.
2. The values to send outwards.
3. The outward scope, that is which local records this user is responsible for pushing.
4. The set of fields whose change marks the record pending.
5. How to raise the pending flag on everything, when a user restarts synchronisation.
6. Which user's connection to use for a given record.
7. Whether an insertion must be blocked because the record's organiser is somebody else.

**Three invariants hold for both services.**

*Calls are made after the transaction commits.* An insertion, a patch or a deletion is queued and
executed only once the database transaction has succeeded, so the remote service never learns about a
change the platform rolled back. Everything the deferred call needs is captured when the call is
queued, because the record may have changed or disappeared by the time it runs. A failure in a
deferred call is logged and swallowed.

*The pending flag is the queue.* There is no separate outbox. A record that has changed carries the
flag; a run pushes every flagged record inside the scope and lowers the flag.

*The newest change wins.* An inward change is applied only when the remote modification instant is at
or after the local write instant. The comparison is on instants, not on versions, so it is only as
good as the two clocks; this is a deliberate, documented compromise.

---

## 3. Identity

### 3.1 The first service

One identifier, `google_id`, per record.

- A single meeting carries the identifier the service assigned it at insertion, or the identifier the
  platform generated and sent, which is a thirty-two character hexadecimal value.
- A series carries the identifier of the remote series.
- An occurrence of a series does not carry an identifier of its own in the database unless it has been
  moved: it derives one from its series, by
  [calculations.md, chapter 19](calculations.md#19-the-identifier-of-an-occurrence-in-the-first-external-calendar-service).
  The derivation deliberately does not depend on the occurrence's current start.

Matching a remote entry to a local record happens in two passes:

1. **By identifier.** Look for local records whose identifier is one of the identifiers in the batch.
2. **By stored property.** For the entries still unmatched, read the local identifier the platform
   wrote into the remote entry's shared or private extended properties under the key
   `<database name>_odoo_id`, where the prefix is the database name. Accept the match only when the
   local record exists and carries no identifier of its own. That last condition matters because the
   service copies extended properties when a series is split, so two remote series can name the same
   local identifier.

A cancelled remote entry carries only its identifier and its cancelled status, so nothing in it says
whether it was a meeting or a series. Such entries are resolved by looking them up among the local
series: those that match are treated as series, the rest as meetings.

A remote occurrence that has been rescheduled changes identifier: an identifier of the shape
*series* + `_` + *range marker* becomes *series* + `_` + *instant*. The run therefore also removes the
local record created under the combined shape *series* + `_` + *range marker* + `_` + *instant*, which
it reconstructs from the entry's own identifier and its series reference by taking the leading
identifier segment common to both, the range marker from the series reference and the instant from the
entry's identifier. When the two leading segments differ, or either pattern is absent, no
reconstruction is attempted.

### 3.2 The second service

Two identifiers per record, because that service gives an event a different identifier in every
calendar it appears in.

- `ms_universal_event_id` is the same in every calendar and is the reliable key.
- `microsoft_id` is the identifier inside one particular calendar, and is the one every request path
  needs.

Matching a batch of remote entries to local records happens in two passes:

1. **By the shared identifier.** Match unmatched entries against local records whose shared identifier
   equals the entry's.
2. **By the calendar-scoped identifier.** Match the rest against local records whose scoped identifier
   equals the entry's. When such a match is found, the shared identifier is also written onto the local
   record, together with the scoped one and a lowered pending flag, so that later runs match on the
   first pass.

Series are matched before meetings, because a removed series carries nothing that says it was a
series: every removed entry is therefore offered to the series matcher first.

An entry counts as a series head when its kind is the series-head kind; as belonging to a series when
it carries a series-head reference; as an exception when its kind is the exception kind; and as
removed when it carries a removal marker whose reason is deletion.

---

## 4. Outward to the first service

### 4.1 The value map for a meeting

| Sent property | Value |
|---|---|
| `id` | The record's identifier, or a generated thirty-two character hexadecimal value when it has none. |
| `start` and `end` | For an all-day meeting, `date` holds the start day and the day after the stop day, and `dateTime` is explicitly empty. For a timed meeting, `dateTime` holds the start and the stop as instants in coordinated universal time, and `date` is explicitly empty. Both keys are always present, so that a meeting can change between the two kinds. |
| `summary` | The subject. |
| `description` | The description, cleaned of unsafe markup, or an empty text when it is empty. |
| `location` | The location, or an empty text. |
| `guestsCanModify` | The negation of the guest-modification restriction. |
| `organizer` | The organiser's electronic mail address, and whether the organiser is the acting user. |
| `attendees` | One entry per attendee that has a normalised address, holding that address and the response, defaulting to needing action. The list is sorted by address so that repeated runs produce the same request. |
| `extendedProperties` | Under `shared`, the key `<database name>_odoo_id` holding the local identifier. |
| `reminders` | The overrides of [calculations.md, section 20.1](calculations.md#201-outward-first-service), with `useDefault` false. |
| `conferenceData` | A creation request carrying a fresh thirty-two character hexadecimal request identifier, when the record has no remote identifier, no video-call address and no location. Explicitly empty when the record has a remote identifier and no video-call address. |
| `visibility` | The visibility policy, when one is set. |
| `transparency` | `opaque` when the availability marker is busy, `transparent` when it is free. |
| `status` | `cancelled`, when the record is archived. |

Two ownership variations apply.

*The organiser is a platform user who does not synchronise.* An extra shared property
`<database name>_owner_id` carries that user's identifier, so that a later run by that user can
recognise the meeting as theirs.

*There is no organiser at all.* The service refuses shared properties in that case, so the request is
reduced to the identifier, the subject, the attendees, the start, the end and the reminders, and the
local identifier is carried under `private` rather than `shared`.

### 4.2 The value map for a series

The series sends the value map of its first occurrence, with four changes: the identifier is the
series' own; for a timed series the start and the end carry the series' time zone, defaulting to
`Etc/UTC`; the pattern is sent as a `recurrence` list holding a single rule line; and the local
identifier is carried under `shared` when the first occurrence has an organiser and under `private`
otherwise.

Preparing the rule line takes two edits. The start line is removed, because the service takes the
start from the start property instead. An end instant that carries no trailing `Z` gains one, so that
it is unambiguously in coordinated universal time; an end instant that already has one is left alone.

### 4.3 The calls

| Call | Path | Notes |
|---|---|---|
| Fetch | `/calendar/v3/calendars/primary/events`, with the change marker, or with a window when there is none | Followed page by page until no page marker remains. A rejected marker raises the full-synchronisation condition. |
| Fetch one | the same path with the entry identifier appended | Returns one entry, no marker and no default reminders. |
| Insert | `/calendar/v3/calendars/primary/events?conferenceDataVersion=%d&sendUpdates=%s` | The first value is 1 when the meeting wants a video call and 0 when it does not; the second is `all` or `none`. |
| Patch | `/calendar/v3/calendars/primary/events/%s?sendUpdates=%s&conferenceDataVersion=1` | The first value is the entry identifier. |
| Delete | `/calendar/v3/calendars/primary/events/%s?sendUpdates=all` | For a series, the extra parameter `singleEvents` set to `true` asks the service to remove every occurrence itself, so that it sends one cancellation rather than one per occurrence. |

Whether attendees are told is decided per call: an insertion tells them unless the caller asked
otherwise or the meeting is already over; a patch tells them unless the meeting is over; a deletion
always tells them. A run that is a full run suppresses the telling, so that a first connection does
not flood everybody.

### 4.4 After a successful insertion

The identifier the service returned is written onto the record and the pending flag is lowered, in one
write that suppresses further notification.

---

## 5. Inward from the first service

### 5.1 The value map for a meeting

| Local field | Source |
|---|---|
| activity flag | Set to false, and nothing else is mapped, when the remote entry is cancelled. |
| subject | The remote summary; failing that the existing subject; failing that "(No title)". |
| description | The remote description, cleaned of unsafe markup. |
| location | The remote location. |
| organiser | The owner resolved by 5.2. |
| visibility | The remote visibility, or empty. |
| attendees | The instructions of 5.3. |
| reminders | The instructions of [calculations.md, section 20.2](calculations.md#202-inward-first-service). |
| repetition marker | Whether the entry belongs to a series or is one. |
| video-call address | The address of the first conferencing entry point whose kind is video. Removed from the map entirely when there is none, so that a locally set address is not wiped. |
| availability marker | `free` when the remote transparency is `transparent`, `busy` otherwise. |
| guest-modification restriction | The negation of the remote guest-modification permission. |
| identifier | The remote identifier, unless the entry is a series. |
| follow marker | For an entry that belongs to a series, whether the entry still sits at its original start. |
| all-day marker, start, stop | 5.4. |

### 5.2 Resolving the owner

1. When the entry carries the shared property `<database name>_owner_id` and it names an existing
   user, that user is the owner.
2. Otherwise, when the entry's organiser is marked as the reader, the reader is the owner.
3. Otherwise, when the entry's organiser carries an address, the first user whose normalised address
   matches is the owner.
4. Otherwise there is no owner.

### 5.3 Attendee instructions

1. Take the remote attendee list. When it is empty and the entry's organiser is marked as the reader,
   add a synthetic entry for the owner's own address with the response `accepted`.
2. Collect the addresses. Find or create a contact for each, in the order the service gave them, and
   note the creation as coming from a calendar synchronisation so that the contact's creation message
   reads "Contact created through Calendar sync.".
3. For each remote attendee:
   - when a local attendee record already has that exact address, update its response;
   - otherwise pick the contact: the reader's own contact when the entry marks the attendee as the
     reader; else the contact whose normalised address matches; else the contact whose raw address
     matches; else skip the attendee entirely;
   - create an attendee record with the response and that contact, and add the contact to the attendee
     set;
   - when the remote attendee carries a display name and the contact has none, write it.
4. For each existing local attendee record whose normalised address is neither in the remote list nor
   equal to the reader's own address, delete the record and remove the contact from the attendee set.

The exception for the reader's own address is what stops a run made by an attendee from removing that
attendee from a meeting the service does not list them on.

### 5.4 Times

```formula
when the remote entry carries a start with a clock:
    start = the remote start, converted to coordinated universal time, with the zone marker removed
    stop  = the remote end,   converted the same way
    all-day marker = false

otherwise:
    start = the remote start day at 08:00
    stop  = ( the remote end day − 1 day ) at 18:00
    when that stop is earlier than that start:
        stop = the remote end day at 18:00
    all-day marker = true
```

The subtraction of one day is there because the remote end day of an all-day entry is exclusive. The
guard that follows it exists because older entries sometimes carry an inclusive end day.

A start or a stop equal to the value the local record already holds is left out of the map, so that
writing it does not mark the record pending for no reason.

### 5.5 The run

For a batch of remote entries:

1. Split them into those that already match a local record and those that do not, discarding cancelled
   entries from the second group.
2. Create the new ones with the pending flag lowered, with notification suppressed and without the
   contact paragraph.
3. Cancel the ones that are cancelled: clear the identifier and delete the record. When the reader is
   not the organiser, decline instead (CAL-138 in [business-rules.md](business-rules.md)).
4. For an entry that belongs to a series but no longer sits at its original start, also remove the
   local record created under the combined identifier described in [3.1](#31-the-first-service).
5. For each remaining matched entry, compare the remote modification instant with the write instant
   captured before the run began, and apply the change only when the remote one is at or after it. A
   local record with no write instant at all is always overwritten.

---

## 6. Series in the first service

### 6.1 Creating a series inward

For each new remote series:

1. Build the meeting value map from the series entry itself.
2. Look for a local meeting whose identifier equals the series' identifier. That happens when a single
   meeting was turned into a series remotely, because the service reuses the identifier. When one is
   found, overwrite it with the series values and clear its identifier, because it is about to be
   recomputed from the series. When none is found, create a new meeting.
3. Make that meeting the series' base occurrence and its first occurrence.
4. Record the series' time zone from the remote start's zone; the platform stores the zone on the
   series while the service stores it on the entry.
5. Create the series and apply it, passing the base occurrence's attendee instructions so that every
   generated occurrence carries the same attendees.

### 6.2 Updating a series inward

1. Remember the current pattern text and its parsed form.
2. Write the incoming values, including the time zone.
3. Rebuild the attendee state across every occurrence: update the responses of attendee records whose
   address is in the remote list; create records and add contacts for remote attendees not yet
   present; and remove attendee contacts that are no longer in the remote list, except the contacts of
   the occurrences' organisers, because removing an organiser would make the occurrences disappear.
4. Compare the base occurrence's start, stop and all-day marker with the incoming ones. When any of
   them differs, the series must be rebuilt: every occurrence except the base has its identifier
   cleared and is deleted, the base is written with the new values and no identifier, and the series is
   applied again — but only when the pattern itself did not change, because a changed pattern is
   handled by step 5 anyway.
5. When the pattern's parsed form changed, apply the series: the occurrences it no longer produces are
   detached, their identifiers are cleared and they are deleted.
6. When the times did not change, write every non-time, non-pattern value onto every occurrence, with
   the pending flag lowered.

### 6.3 Pushing a series outward

Applying a series lowers the pending flag on every occurrence, because the series carries them. Before
that, every occurrence that already had an identifier of its own and whose derived series identifier
now differs is dealt with as in CAL-149: an inactive copy carrying the old identifier is created so
that the next run removes it, the remote copy is deleted at once, and the occurrence's own identifier
is cleared.

Writing values onto the occurrences of a series always removes the identifier from the values and
lowers the pending flag, so that a series-wide change produces one request rather than one per
occurrence.

---

## 7. Outward to the second service

### 7.1 The value map for a meeting

The map is built field by field, and only the fields being synchronised are included, so a patch sends
exactly what changed.

| Sent property | Included when | Value |
|---|---|---|
| `seriesMasterId` and `type` | The meeting carries a series-head reference and the caller did not already set a kind | The series-head reference, and the kind `exception`. |
| `subject` | the subject is synchronised | The subject, or an empty text. |
| `body` | the description is synchronised | The description cleaned of unsafe markup, with the content kind `html`. |
| `start`, `end`, `isAllDay` | any time field is synchronised | For an all-day meeting, the start day and the day after the stop day, both marked as being in coordinated universal time. For a timed meeting, the start and the stop as instants in coordinated universal time. |
| `location` | the location is synchronised | The location, or an empty text, under the display-name key. |
| `isOnlineMeeting`, `onlineMeetingProvider` | the video-call address is synchronised | True and the service's own conferencing product, when the meeting has no location and wants a video call. False otherwise. |
| `isReminderOn`, `reminderMinutesBeforeStart` | the reminders are synchronised | [calculations.md, section 20.3](calculations.md#203-outward-second-service). |
| `organizer`, `isOrganizer` | the organiser is synchronised | The organiser's address and display name, and whether the organiser is the acting user. |
| `attendees` | the attendees are synchronised | One entry per attendee that is not the organiser's own contact, holding the address, the display name and the mapped response. |
| `showAs`, `sensitivity` | the visibility or the availability marker is synchronised | The availability marker as it stands, and the visibility mapped by 7.2. |
| `isCancelled` | the activity flag is synchronised and the record is archived | True. |
| `recurrence` | the caller set the kind to the series-head kind | The pattern of [chapter 9](#9-series-in-the-second-service). |

### 7.2 The two mappings

**Responses, outward.** An attendee who is the organiser's own user is sent as `organizer`. Otherwise
`needsAction` is sent as `notresponded`, `tentative` as `tentativelyaccepted`, `declined` as
`declined` and `accepted` as `accepted`. An unmapped value is sent as the text `None`.

**Visibility, outward.** `public` is sent as `normal`, `private` as `private` and `confidential` as
`confidential`. An empty policy is resolved through the organiser's default privacy and, when there is
no organiser, sent as `normal`.

### 7.3 The calls

| Call | Path | Notes |
|---|---|---|
| Fetch changes | `/v1.0/me/calendarView/delta`, with the change marker when there is one | Requested with a page size of fifty and a preference for markup bodies. Followed page by page. The new marker is read from the query value `$deltatoken` of the final link. Entries of the occurrence kind are dropped. |
| Fetch the instances of a series | `/v1.0/me/events/%s/instances` | The value is the series head's calendar-scoped identifier. Followed page by page. |
| Fetch one by shared identifier | `/v1.0/me/events?$filter=iCalUId eq '%s'` | Used before answering an invitation, because the path of the answer needs the reader's own scoped identifier. |
| Insert | `/v1.0/me/calendar/events` | Returns both identifiers. |
| Patch | `/v1.0/me/calendar/events/%s` | Answers false when the service says the event is absent, and the record stays pending. |
| Delete | `/v1.0/me/calendar/events/%s` | An answer that the event is gone or forbidden is treated as success. |
| Answer | `/v1.0/me/calendar/events/%s/%s` | The second value is `accept`, `tentativelyAccept` or `decline`. The body carries an empty comment and asks the service to send a response. |

The window of a fetch with no marker runs from the current instant less the configured number of days
to the current instant plus twice that number.

---

## 8. Inward from the second service

### 8.1 The value map for a meeting

| Local field | Source |
|---|---|
| activity flag | Set to false, and nothing else is mapped, when the entry is cancelled or removed. |
| subject | The remote subject, or "(No title)". |
| description | The content of the remote body. |
| location | The remote location's display name, or empty. |
| organiser | 8.2. |
| visibility | The remote sensitivity mapped as `normal` to `public`, `private` to `private`, `confidential` to `confidential`, anything else to empty. |
| attendees | 8.3. |
| all-day marker | The remote all-day marker. |
| start | The remote start read in the zone the remote start names, with the zone marker removed. |
| stop | The remote end read the same way, less one day when the entry is all-day. |
| availability marker | `free` when the remote availability is `free`, `busy` otherwise. |
| repetition marker | Whether the entry belongs to a series or is one. |
| follow marker | For an entry that belongs to a series, the negation of its being an exception. |
| video-call address | The join address of the remote online meeting when there is one. Otherwise, when the location matches the pattern of an address beginning `https://teams.microsoft.com`, the location is moved into the video-call address and the location is emptied. |
| both identifiers | Included only when the caller asked for them. |
| series-head reference | The remote series-head reference, when the entry belongs to a series. |
| reminders | [calculations.md, section 20.4](calculations.md#204-inward-second-service). |

An entry of the occurrence kind maps only its two identifiers, its series-head reference, its start
and its stop, because everything else comes from the series head.

### 8.2 Resolving the owner

1. When the entry marks the reader as the organiser, the reader is the owner.
2. Otherwise, when the entry has no organiser, there is no owner.
3. Otherwise the first user whose address matches the organiser's normalised address is the owner;
   when none matches, there is no owner, and the meeting can then only be changed from the service.

### 8.3 Attendee instructions

1. Collect the normalised addresses of the remote attendees.
2. When the entry already matches a local record, collect the existing attendee records of that record
   whose contact's normalised address is one of them. When it does not, and the reader's own address is
   not among the remote addresses, add an instruction creating an accepted attendee record for the
   reader and adding the reader's contact.
3. Find or create a contact for each remote address.
4. For each remote attendee: the response is taken from the entry's own response property when the
   attendee is the reader, because the service hides other people's responses from an attendee's copy,
   and from the attendee's own status otherwise. It is mapped by 8.4. An existing record is updated; a
   contact that exists gets a new record and joins the attendee set; a display name is written onto a
   nameless contact.
5. Existing records whose contact's address is no longer in the remote list are deleted and their
   contacts leave the attendee set.

### 8.4 Responses, inward

`none` and `notResponded` map to `needsAction`; `tentativelyAccepted` to `tentative`; `declined` to
`declined`; `accepted` to `accepted`; `organizer` to `accepted`. An unmapped value maps to
`needsAction`.

### 8.5 The run

1. Match the batch against local records ([3.2](#32-the-second-service)).
2. Split the batch into matched, cancelled and new; the new that belong to a series are handled by
   [chapter 9](#9-series-in-the-second-service).
3. Create the new single meetings with the pending flag lowered, without the contact paragraph.
4. Delete the cancelled ones: series first, then the meetings that are not themselves a cancelled
   series' head. Cancelling clears both identifiers and then deletes, so no outward call is made.
   A meeting the reader neither owns nor attends is declined instead of deleted.
5. For every remaining matched entry that carries a modification instant, compare it with the local
   write instant and apply the change when it is at or after it and the old-event guard passes.
6. For a series head, updating also refreshes every occurrence
   ([9.3](#93-updating-a-series-inward)).

---

## 9. Series in the second service

### 9.1 The pattern, outward

The pattern is sent as a structure of two parts.

**The repetition part**

| Sent key | Value |
|---|---|
| `interval` | The interval. |
| `type` | `daily` or `weekly` for those frequencies. For a monthly or yearly frequency, the word `absolute` when the monthly mode is by number and `relative` when it is by position, followed by the frequency with its first letter capitalised, giving `absoluteMonthly`, `relativeMonthly`, `absoluteYearly` or `relativeYearly`. |
| `dayOfMonth` | The day number, when the monthly mode is by number. |
| `daysOfWeek` | The selected weekdays written in full and in lower case, when the monthly mode is by position or the frequency is weekly. |
| `firstDayOfWeek` | `sunday`, always, when the previous key is present. |
| `index` | The position, mapped as 1 to `first`, 2 to `second`, 3 to `third`, 4 to `fourth` and −1 to `last`, when the frequency is monthly and the mode is by position. |

**The range part**

| Termination | Sent keys |
|---|---|
| by count | `numberOfOccurrences` = the smaller of the count and 720; `type` = `numbered`. |
| endless | `numberOfOccurrences` = 720; `type` = `numbered`. |
| by end day | `endDate` = the end day; `type` = `endDate`. |
| always | `startDate` = the day part of the series' first instant, or of the current instant when the series has none. |

### 9.2 The pattern, inward

| Local field | Source |
|---|---|
| frequency | `absoluteMonthly` and `relativeMonthly` map to monthly; `absoluteYearly` and `relativeYearly` to yearly; anything else keeps its own name, giving daily or weekly. |
| termination mode | `endDate` maps to by end day; `noEnd` to endless; `numbered` to by count. |
| interval | The remote interval. |
| count | The remote number of occurrences. |
| day number | The remote day of the month. |
| position | The remote index mapped back through the same table. |
| end day | The remote end day, only when the range kind is the end-day kind. |
| monthly mode | `absoluteMonthly` and `absoluteYearly` map to by number; `relativeMonthly` and `relativeYearly` to by position. |
| weekday markers | Each of the seven is set when the first three letters of its name appear in the remote list of days. |
| weekday | The first three letters of the first remote day, in upper case. |

### 9.3 Creating and updating a series inward

For each new remote series head:

1. Build the series values from the head, including both identifiers, with the pending flag lowered.
2. Collect the remote entries that name that head. When the termination mode is by count or endless,
   at most seven hundred and twenty of them are kept.
3. Build the base values from the head, and then, for each collected entry, either the occurrence value
   map when it is of the occurrence kind or the full value map otherwise.
4. Create the series together with all of its occurrences in one operation, and make the first of them
   the base occurrence.
5. Remove any local single meeting whose shared identifier equals the new series' shared identifier and
   that belongs to no series: the service reuses the identifier of a single event that has been turned
   into a series, and leaving the single meeting in place would duplicate the series' first occurrence.

For an existing series:

1. Find it by its shared identifier among the batch.
2. For each remote entry that names the head, build the values, find the occurrence whose start and
   stop match exactly, and write the values onto it without the start and the stop.
3. When the series head itself changed, first write the head's values onto the series, and then update
   every occurrence: exceptions with the full value map, occurrences with the occurrence value map.
   Occurrences whose start or stop appear in those values are collected as creation values, the series
   is applied with them, and the occurrences the pattern no longer produces are cancelled.
4. Before rebuilding, compare the base occurrence's start, stop and all-day marker with the incoming
   ones, comparing only the day parts for an all-day entry. The destructive rebuild only runs when they
   differ, the new start is not earlier than the current base start, and the base occurrence still
   follows the pattern; the last condition stops an exception's own shifted time from being mistaken
   for a change to the head.

### 9.4 Why the platform refuses to change a series

The service sends one message per changed occurrence. A platform-side change to a series of two
hundred occurrences would therefore produce two hundred messages to every attendee. The platform
consequently refuses to create a series while the service is connected (CAL-141), to change one
(CAL-142), and to delete or archive one (CAL-143), and directs the reader to the service's own
screens.

The one place the platform does write a series outward is when a series is split: the new series' base
occurrence may already exist in the service, so it is deleted there first, to avoid a duplicate.

---

## 10. Ownership and blocked insertions

Both services carry the same problem: a meeting has one owner in the remote calendar, and the platform
lets anybody edit a meeting they attend.

**Whose connection is used.** For the first service, the organiser's connection when the organiser
holds a short-lived credential, and the acting user's otherwise. For the second, the organiser's
connection when the acting user holds a credential and the organiser's synchronisation is active, and
the acting user's otherwise.

**When an insertion is skipped.** For both services, a record whose organiser exists and is not the
user making the call is not inserted. For a series, the test is made on the base occurrence's
organiser. The record stays pending, so the organiser's own next run inserts it under the right owner.

**When invitations are not sent by the platform.** A meeting whose chosen user is connected and valid
is left to the service to announce, because the service sends its own invitations; the rule is CAL-170.

**Changing the organiser.** The first service simply sends the new organiser in the value map. The
second service cannot change an organiser at all, so the platform copies the meeting, creates the copy
under the new organiser, archives the original and deletes the original's remote copy (CAL-147), after
checking that the new organiser is connected and is an attendee (CAL-146).

---

## 11. Errors and how they are reported

### 11.1 Delegated-authorisation failures

| Situation | Behaviour |
|---|---|
| The registration is missing | The synchronisation route answers that configuration is needed and, for an administrator, includes the general settings action. |
| No consent yet | The route answers that consent is needed and returns the consent address. |
| The consent round trip returns an error | The reader is redirected to the return address with the error appended as a query value, or with the value `Unknown_error` when the service gave none. |
| The code cannot be exchanged | Refused with the message of CAL-162. |
| A refresh is rejected as an invalid grant or an invalid client | The credentials are cleared and committed and the message of CAL-161 is raised. |
| A call is attempted with no short-lived credential | The call refuses immediately with the internal condition "An authentication token is required"; nothing is sent. |

### 11.2 Request failures, first service

A rejected insertion or patch whose answer is a bad-request or a forbidden answer is handled: the
pending flag is lowered so that the record is not retried until it changes again, and a note is posted
on the meeting with the wording of CAL-163. When the rejected record is a series, every occurrence has
its pending flag lowered and the note goes on the base occurrence, or on the earliest occurrence
including outliers when there is none. When the record no longer exists, only a log line is written.

A deletion that is answered with "gone" or "forbidden" is treated as a success, because the entry was
already removed. An answer with no content is read as an empty answer rather than an error.

### 11.3 Request failures, second service

A patch that fails is logged and answers false, which leaves the record pending so that the next run
tries again. A deletion answered with "gone" or "forbidden" is treated as a success. Answers with no
content and not-found answers are read as an empty answer.

### 11.4 The change marker

The first service signals an expired marker with a "gone" answer whose body mentions that a full
synchronisation is required; the second signals it with a "gone" answer whose error code mentions a
full synchronisation or a missing synchronisation state. Both cause an immediate full fetch, and the
marker returned by that fetch replaces the expired one.

---

## 12. The two services side by side

| Aspect | First external calendar service | Second external calendar service |
|---|---|---|
| Identifiers per record | one | two: one shared across calendars, one per calendar |
| Where the time zone of a series lives | on the entry, read into the series | on the entry, read into the series |
| Series created from the platform | allowed | refused |
| Series changed from the platform | allowed | refused |
| Series deleted from the platform | allowed, in one request | refused |
| Deleting a synchronised meeting | archived locally; removed remotely at the next run | removed remotely first, then locally |
| Reminders crossing outward | every reminder, as electronic-mail or pop-up overrides | only the first in-application reminder |
| Reminders crossing inward | every remote reminder | only the single reminder switch |
| Attendee responses outward | inside the meeting's own value map | through a dedicated answer call |
| Organiser change | sent in the value map | the meeting is recreated under the new organiser |
| Attendees without an address | tolerated; they are skipped | refused for the whole batch |
| Outward batch size | at most two hundred records per transaction | unbounded |
| Scope narrowed by a last-run instant | no | yes, with five minutes of tolerance |
| A first connection ignores older meetings | no | yes, through the first-connection cut-off |
| Video call | a conferencing creation request | the service's own online-meeting switch |
| Guest editing | carried as a restriction and enforced locally | not carried |

---

## 13. Transport addresses

These addresses are part of the integration contract and are reproduced exactly.

### 13.1 First external calendar service

| Purpose | Address |
|---|---|
| Consent | `https://accounts.google.com/o/oauth2/auth` |
| Credential exchange and refresh | `https://accounts.google.com/o/oauth2/token` |
| Application base | `https://www.googleapis.com` |
| Requested scope | `https://www.googleapis.com/auth/calendar` |
| Platform callback | `/google_account/authentication` |

The consent request also carries a response kind of `code`, the registration identifier, the scope,
the callback address, the state value, a forced approval prompt and an offline access request. The
default address to return to, when the caller gives none, is the vendor's own site; this
specification writes it as `https://example.com/`. Every request is checked to be going to one of the
two hosts above before it leaves.

### 13.2 Second external calendar service

| Purpose | Address |
|---|---|
| Consent | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`, overridable through the system parameter `microsoft_account.auth_endpoint` |
| Credential exchange and refresh | `https://login.microsoftonline.com/common/oauth2/v2.0/token`, overridable through the system parameter `microsoft_account.token_endpoint` |
| Application base | `https://graph.microsoft.com` |
| Requested scope | `offline_access openid Calendars.ReadWrite` |
| Platform callback | `/microsoft_account/authentication` |
| Stored redirect value | the system parameter `microsoft_redirect_uri`, shipped with the value `urn:ietf:wg:oauth:2.0:oob` |

The consent request also carries a response kind of `code`, the registration identifier, the state
value, the scope, the callback address and an offline access request. Every request is checked to be
going to one of the two hosts above before it leaves. A refresh returns a new long-lived credential
each time, and the platform stores it, because that service rotates the credential on every refresh.

The video-call addresses this service produces are recognised locally by the prefix
`https://teams.microsoft.com`.
