# Marketing acceptance criteria

Numbered scenarios in Given, When, Then form. Each one states concrete starting records, a concrete
operation with concrete inputs, and the exact resulting records, counts, amounts and states. They
cover the ordinary path, every validation failure, every state transition, the rounding edges of
every formula, the multi-company and multi-website cases that apply, and the permission boundaries.

Every scenario carries an identifier of the form `MKT-AC-nnn`, stable within this file. Where a
scenario asserts a message, the message is reproduced between quotation marks exactly as the system
shows it. Where a scenario asserts a stored value, the value is in code font. Unless a scenario says
otherwise, the acting user is a Marketing User, the current moment is 2026-03-02 09:00:00 in
coordinated universal time, and no comparison test is running.

Section index:

| Range | Topic |
|---|---|
| `001`–`019` | Creating and configuring a mailing |
| `020`–`034` | Audience, conditions and saved filters |
| `035`–`043` | Test sends |
| `044`–`062` | Queueing, sending and the exclusion checks |
| `063`–`072` | Cancelling, retrying and resuming |
| `073`–`089` | Comparison testing |
| `090`–`101` | Indicators, ratios and rounding |
| `102`–`115` | Link tracking and redirection |
| `116`–`125` | Campaign tracking and unique names |
| `126`–`143` | Lists, contacts, subscriptions, import and merge |
| `144`–`163` | Public pages, tokens and feedback |
| `164`–`175` | Text message mailings |
| `176`–`188` | Marketing cards |
| `189`–`200` | Configuration, permissions, deletion guards and multi-site |

---

## Creating and configuring a mailing

### MKT-AC-001 — Defaults of a new mailing

**Given** an installation where the dedicated-server setting is off and exactly one Mailing List
named `Newsletter` exists.
**When** a Marketing User creates a mailing from the Mailings menu without typing anything.
**Then** the new record has `state` = `draft`, `mailing_type` = `mail`, `mailing_model_id` pointing
at the Mailing List entity, `schedule_type` = `now`, `schedule_date` empty, `use_exclusion_list`
true, `ab_testing_pc` = 10, `ab_testing_winner_selection` = `opened_ratio`,
`ab_testing_schedule_datetime` = 2026-03-03 09:00:00, `user_id` = the acting user, `mail_server_id`
empty, `contact_list_ids` holding the single `Newsletter` list, and `calendar_date` empty.

### MKT-AC-002 — Default mail server when the dedicated-server setting is on

**Given** the parameter `mass_mailing.outgoing_mail_server` is true and
`mass_mailing.mail_server_id` holds the key of the server named `Bulk relay`.
**When** a Marketing User creates a mailing.
**Then** `mail_server_id` is `Bulk relay` and `mail_server_available` is true, so the server chooser
is visible on the form.

### MKT-AC-003 — Sender address derived from the creating user

**Given** a Marketing User whose formatted address is `Ada Byron <ada@example.com>` and a mailing
with no mail server.
**When** the mailing is created without a sender address.
**Then** `email_from` is `Ada Byron <ada@example.com>`, by case 1 of the derivation table in
[entities.md](entities.md#15-derivation-rules).

### MKT-AC-004 — Sender address corrected to the notification address

**Given** a mail server whose sender filter accepts only `news@example.com`, a Marketing User whose
address is `ada@example.com`, and a system notification address of `news@example.com`.
**When** a mailing is created with that server and no sender address.
**Then** `email_from` becomes `news@example.com` by case 4, and `warning_message` is empty.

### MKT-AC-005 — Warning when the sender does not match the server

**Given** the same mail server and a mailing whose `email_from` is typed by hand as
`ada@example.com`.
**When** the form recomputes.
**Then** `warning_message` is *"This email from can not be used with this mail server."* followed by
a line break and *"Your emails might be marked as spam on the mail clients."*, and the mailing may
still be saved: the warning does not block.

### MKT-AC-006 — Sender address is required for an electronic-mail mailing

**Given** a mailing of type `mail`.
**When** an integration writes an empty `email_from`.
**Then** the write is refused by the database check `email_from IS NOT NULL OR mailing_type != 'mail'`
with the message *"email from is required for mailing"*.

### MKT-AC-007 — Sender address is not required for a text-message mailing

**Given** a mailing of type `sms`.
**When** it is saved with an empty `email_from`.
**Then** the save succeeds, because the check exempts the text-message type.

### MKT-AC-008 — Medium derived for each channel

**Given** a new mailing of type `mail` with no medium.
**When** it is saved.
**Then** `medium_id` is the medium named `Email`.
**And when** its type is changed to `sms`.
**Then** `medium_id` becomes the medium named `Text Message`.
**And when** the type is changed back to `mail`.
**Then** `medium_id` becomes `Email` again, because the previous value was the text-message medium.

### MKT-AC-009 — Answer mode derived from the recipient entity

**Given** a mailing whose recipient entity is Mailing List.
**Then** `reply_to_mode` is `new` and `reply_to` is the formatted address of the acting user.
**And when** the recipient entity is changed to Sales Order.
**Then** `reply_to_mode` becomes `update` and `reply_to` is cleared.

### MKT-AC-010 — The mailing name is unique across every Campaign Source

**Given** a Campaign Source already named `Spring sale`.
**When** two mailings are created whose subject is `Spring sale`.
**Then** the first receives the name `Spring sale [2]` and the second `Spring sale [3]`, each
carrying its own newly created Campaign Source, and no uniqueness violation is raised.

### MKT-AC-011 — A default name coming from the screen is ignored

**Given** a screen that supplies `Spring sale` as a default name and a Campaign Source already
carrying that name.
**When** a mailing is created from that screen.
**Then** the supplied default is discarded and the name is generated from the subject instead, so
the creation cannot fail on the uniqueness constraint.

### MKT-AC-012 — Writing one name on several mailings at once is refused

**Given** two mailings.
**When** both are written in one operation with the subject `Autumn news`.
**Then** the write is refused with *"You cannot update multiple records with the same name. The name
should be unique!"*

### MKT-AC-013 — Comparison-test percentage bounds

**Given** a mailing.
**When** `ab_testing_pc` is written as 101.
**Then** the database check refuses the write with *"The A/B Testing Percentage needs to be between 0
and 100%"*.
**And when** it is written as −1, the same refusal occurs.
**And when** it is written as 0 or 100, the write succeeds.

### MKT-AC-014 — Favorite moment is stamped once

**Given** a mailing whose `favorite` is false and `favorite_date` empty.
**When** the author presses "Add to Templates" at 09:05:00.
**Then** `favorite` is true, `favorite_date` is 2026-03-02 09:05:00 and the notice *"Design added to
the Mailing Contacts Templates!"* is shown.
**And when** the mailing is saved again at 09:10:00 without touching the mark.
**Then** `favorite_date` is still 09:05:00.
**And when** "Remove from Templates" is pressed.
**Then** `favorite` is false, `favorite_date` is empty and the notice *"Design removed from the
Mailing Contacts Templates!"* is shown.

### MKT-AC-015 — Duplicating a mailing resets the right fields

**Given** a mailing in `state` = `done`, with `sent_date` 2026-03-01 10:00:00, `favorite` true,
`use_exclusion_list` false, 400 delivery records, two mailing lists and
`ab_testing_schedule_datetime` 2026-03-04 09:00:00 with comparison testing enabled.
**When** the author presses Duplicate.
**Then** the copy has `state` = `draft`, `sent_date` empty, `favorite` false, `favorite_date` empty,
`calendar_date` empty, `kpi_mail_required` false, `use_exclusion_list` true, no delivery record, the
same two mailing lists, `ab_testing_schedule_datetime` 2026-03-04 09:00:00, and a new Campaign Source
whose name is the original name with the next free counter.

### MKT-AC-016 — Duplicating falls back to the dedicated server

**Given** a mailing whose `mail_server_id` points at an archived server, and a configured dedicated
server `Bulk relay`.
**When** the mailing is duplicated.
**Then** the copy's `mail_server_id` is `Bulk relay`.

### MKT-AC-017 — Inline images become stored files

**Given** a body containing nineteen inline images, of which the first and the last are byte
identical.
**When** the mailing is saved.
**Then** eighteen attachment records are created, named `image_mailing_<mailing key>_1` to
`image_mailing_<mailing key>_18`, each attached to the mailing, and the saved body contains nineteen
addresses of the form `/web/image/<file key>?access_token=<token>`, two of which point at the same
file.

### MKT-AC-018 — An oversized fetched image is refused

**Given** an image address whose declared length exceeds the configured import maximum.
**When** the body is saved.
**Then** the operation is refused with *"File size exceeds configured maximum (%s bytes)"*, the
placeholder being the configured maximum in bytes.
**And given** an image of 45 million pixels, the refusal is *"Image size excessive, imported images
must be smaller than 42 million pixel"*.

### MKT-AC-019 — Calendar creation pre-fills the schedule

**Given** the calendar view open on 2026-03-10.
**When** the author creates a mailing on that day.
**Then** `schedule_type` is `scheduled` and `schedule_date` is 2026-03-10.
**And given** the calendar view open on 2026-03-01, which is in the past.
**Then** `schedule_type` stays `now` and `schedule_date` stays empty.

---

## Audience, conditions and saved filters

### MKT-AC-020 — Condition derived from the chosen lists

**Given** a mailing whose recipient entity is Mailing List.
**When** the author selects the lists `Newsletter` and `Beta testers`.
**Then** `mailing_domain` becomes the condition that a contact belongs to one of those two lists, and
`mailing_model_real` is the Mailing Contact entity.

### MKT-AC-021 — Changing the recipient entity resets the condition and clears the filter

**Given** a mailing with a loaded saved filter and a hand-edited condition.
**When** the recipient entity is changed to Sales Order.
**Then** `mailing_filter_id` is cleared, `mailing_domain` becomes the Sales Order default condition
excluding cancelled orders, `reply_to_mode` becomes `update`, `reply_to` is cleared and
`mailing_filter_count` is recomputed for the new entity.

### MKT-AC-022 — Editing the condition does not clear the loaded filter

**Given** a mailing with the saved filter `Active customers` loaded, so that `mailing_domain` equals
`mailing_filter_domain`.
**When** the author edits `mailing_domain` by hand.
**Then** `mailing_filter_id` still points at `Active customers`, the two values now differ, and the
form shows both so the difference is visible.

### MKT-AC-023 — A saved filter of another entity is refused

**Given** a mailing whose recipient entity is Contact and a saved filter bound to Sales Order.
**When** that filter is written on the mailing.
**Then** the write is refused with *"The saved filter targets different recipients and is
incompatible with this mailing."*

### MKT-AC-024 — An invalid saved condition is refused

**Given** a new Mailing Filter bound to Contact.
**When** its condition names a field that does not exist on Contact.
**Then** the save is refused with *"The filter domain is not valid for this recipients."*

### MKT-AC-025 — Deleting a filter keeps the condition

**Given** a mailing whose `mailing_filter_id` is `Active customers` and whose `mailing_domain` is the
condition of that filter.
**When** the filter record is deleted.
**Then** `mailing_filter_id` becomes empty and `mailing_domain` is unchanged, so the mailing still
addresses the same audience.

### MKT-AC-026 — An unparsable condition yields an empty audience

**Given** a mailing whose stored condition cannot be parsed.
**When** the sending algorithm computes the audience.
**Then** the audience is empty, no message is prepared, and the direct send path is refused with
*"There are no recipients selected."* while the queue job closes the mailing as `done` instead.

### MKT-AC-027 — Only mailing-enabled entities may be chosen

**Given** the recipient-entity chooser of a mailing.
**Then** the offered entities are exactly those whose mailing-enabled flag is true: Contact, Mailing
Contact, Mailing List and, with the matching packages, Lead, Sales Order, Event Registration and
Event Track.

### MKT-AC-028 — The mailing-enabled search supports only two operators

**Given** a query listing the entities whose mailing-enabled flag is in a set of values.
**Then** the query is resolved by listing the registered entities that exist in the running system,
are not transient assistants and declare the flag; a query using any other operator is unsupported.

### MKT-AC-029 — Default condition of Sales Order

**Given** a mailing whose recipient entity is Sales Order and no saved filter.
**Then** `mailing_domain` is the condition that the order state is not `cancel`.

### MKT-AC-030 — Default condition of Event Registration

**Given** a mailing whose recipient entity is Event Registration and no prepared condition supplied
by the calling screen.
**Then** `mailing_domain` is the condition that the registration state is neither `cancel` nor
`draft`.
**And given** the calling screen supplied a prepared condition for the same entity, that condition is
used instead.

### MKT-AC-031 — Default condition of Event Track

**Given** a mailing whose recipient entity is Event Track.
**Then** `mailing_domain` is the condition that the stage of the talk is not a cancellation stage.

### MKT-AC-032 — The event buttons prepare a mailing

**Given** two events selected in the event list, named `Spring conference` and `Autumn conference`.
**When** the organiser presses the button that mails the attendees.
**Then** a mailing form opens with recipient entity Event Registration, the condition restricting the
registrations to those two events and to states other than `cancel` and `draft`, and the subject
`Event: Spring conference`.

### MKT-AC-033 — The course button prepares a mailing

**Given** one course selected.
**When** the button that mails the members is pressed.
**Then** a mailing form opens with recipient entity Contact and the condition restricting contacts to
the members of that course.

### MKT-AC-034 — Saved-filter count

**Given** three Mailing Filter records bound to Contact and one bound to Sales Order, and a mailing
whose recipient entity is Contact.
**Then** `mailing_filter_count` is 3.

---

## Test sends

### MKT-AC-035 — Ordinary test send

**Given** a saved mailing whose subject renders to `Spring sale` and whose recipient entity is
Mailing Contact with at least one contact.
**When** the author opens the test assistant and confirms with the two addresses
`ada@example.com` and `bob@example.com`.
**Then** two outgoing mails are created with the subject `[TEST] Spring sale`, the body rendered
against the first Mailing Contact and wrapped in the marketing layout, the mailing's sender, answer
address, attachments and mail server, they are sent immediately, two notes are logged reading *"Test
mailing successfully sent to ada@example.com"* and *"Test mailing successfully sent to
bob@example.com"*, and **no delivery record is created**: the mailing's `expected` counter stays 0
and its state stays `draft`.

### MKT-AC-036 — Test send with an unusable line

**Given** the same mailing.
**When** the test assistant is confirmed with the three lines `ada@example.com`, `not an address`
and `bob@example.com`.
**Then** two outgoing mails are created and one further note is logged reading *"Mailing addresses
incorrect: not an address"*.

### MKT-AC-037 — Test send that fails at the relay

**Given** a relay that refuses `bob@example.com`.
**When** the test is sent to that address.
**Then** the note is *"Test mailing could not be sent to bob@example.com:"* followed by a line break
and the reported failure reason.

### MKT-AC-038 — The test address box remembers the last value

**Given** a Marketing User who used `qa@example.com` in a test 4 hours ago.
**When** they open the test assistant again.
**Then** the address box is pre-filled with `qa@example.com`.
**And given** the last use was 11 hours ago, so the transient record has expired, the box is
pre-filled with the user's own formatted address.

### MKT-AC-039 — The unsubscribe placeholder is not replaced in a test

**Given** a body containing the unsubscribe placeholder.
**When** a test is sent.
**Then** the placeholder is left untouched in the sent body, because the test flag suppresses the
per-recipient substitution.

### MKT-AC-040 — Test send when the recipient entity holds no record

**Given** a mailing whose condition matches no record.
**When** a test is sent.
**Then** the raw subject, body and preview are used without rendering, and the test still goes out.

### MKT-AC-041 — Ordinary text-message test

**Given** a text-message mailing whose plain-text body is `Sale starts today` and whose opt-out
option is off.
**When** the test assistant is confirmed with the number `+32470123456`.
**Then** one outgoing text message is created in state `outgoing` with a fresh provider key, the
batch is handed to the sending service with the delivery-report address `<base>/sms/status`, and the
note *"Test SMS successfully sent to +32470123456"* is logged. No delivery record is created.

### MKT-AC-042 — Text-message test with the opt-out option on

**Given** the same mailing with the opt-out option on and the mailing key 87.
**When** the test is sent to `+32470123456`.
**Then** one delivery record flagged as a test trace is created carrying the mailing, the first
recipient record, a fresh three-character code such as `k9Q`, the sanitised number, a provider
tracker and the type `sms`; and the body sent is `Sale starts today` followed by a line break and
*"STOP SMS: <base>/sms/87/k9Q"*.

### MKT-AC-043 — Text-message test with an unusable number

**Given** the lines `+32470123456` and `12`.
**When** the test is confirmed.
**Then** one message is sent and the note *"Test SMS skipped those numbers as they appear invalid:
12"* is logged.

---

## Queueing, sending and the exclusion checks

### MKT-AC-044 — Send now

**Given** a mailing in `draft` with a body and a condition matching 1000 contacts.
**When** the author presses Send and confirms the dialogue *"Ready to unleash emails?"* with the
button *"Send to all"*.
**Then** `schedule_type` is `now`, `schedule_date` is empty, `state` is `in_queue`, `next_departure`
is 2026-03-02 09:00:00, `next_departure_is_past` is false and the queue job carries a wake request
for that moment.

### MKT-AC-045 — Dismissing the confirmation changes nothing

**Given** the same mailing.
**When** the dialogue is dismissed.
**Then** the state stays `draft` and no wake request is made.

### MKT-AC-046 — Schedule with a future moment

**Given** a mailing in `draft` whose `schedule_date` is 2026-03-05 08:00:00 and whose
`schedule_type` is `scheduled`.
**When** the author presses Schedule.
**Then** the mailing goes straight to `in_queue`, `calendar_date` becomes 2026-03-05 08:00:00 and the
queue job carries a wake request for that moment; the schedule assistant does not open.

### MKT-AC-047 — Schedule with no moment opens the assistant

**Given** a mailing in `draft` with no schedule date.
**When** the author presses Schedule.
**Then** the schedule assistant opens and the state is still `draft`.
**And when** the author chooses 2026-03-06 07:30:00 and confirms.
**Then** the mailing is written with `schedule_type` = `scheduled` and that moment, and is queued.

### MKT-AC-048 — A moment in the past means "as soon as possible"

**Given** a queued mailing whose `schedule_date` is 2026-03-01 08:00:00.
**Then** `next_departure` is 2026-03-02 09:00:00, `next_departure_is_past` is true and the form shows
*"This mailing will be sent as soon as possible."* with a refresh button.

### MKT-AC-049 — Calendar moment per state

**Given** four mailings, one in each state, with `sent_date` 2026-03-01 10:00:00 on the finished one
and `schedule_date` 2026-03-05 08:00:00 on the queued one.
**Then** `calendar_date` is 2026-03-01 10:00:00 for the finished one, 2026-03-05 08:00:00 for the
queued one, 2026-03-02 09:00:00 for the sending one and empty for the draft one.

### MKT-AC-050 — The queue job selects the right mailings

**Given** five mailings: one `draft`, one `in_queue` with no schedule date, one `in_queue` scheduled
for 2026-03-05, one `in_queue` scheduled for 2026-03-01 and one `sending`.
**When** the queue job runs at 2026-03-02 09:00:00.
**Then** it selects exactly three of them: the queued one with no date, the queued one scheduled in
the past and the sending one; it reports 3 as the remaining work.

### MKT-AC-051 — The queue job closes an exhausted mailing

**Given** a queued mailing all of whose 200 recipients already have a delivery record, and whose
`sent_date` is empty.
**When** the queue job reaches it.
**Then** it is written with `state` = `done`, `sent_date` = 2026-03-02 09:00:00 and
`kpi_mail_required` = true, and no message is prepared.

### MKT-AC-052 — A second sending does not set the statistics flag again

**Given** the same mailing, now with `sent_date` 2026-03-02 09:00:00, queued a second time after a
retry.
**When** the queue job closes it again.
**Then** `sent_date` is updated to the new moment and `kpi_mail_required` is false, because there was
already a previous sent date.

### MKT-AC-053 — The batch loop

**Given** 1000 remaining recipients and `mail.batch_size` absent.
**When** the sending pass runs.
**Then** the batch size is 50, the loop runs 20 times, each pass prepares 50 messages, creates the
outgoing messages and their delivery records, reports progress and commits, and after the twentieth
pass the mailing is written to `done`.

### MKT-AC-054 — A batch size of zero is replaced

**Given** `mail.batch_size` set to `0`.
**Then** the batch size used is 50.

### MKT-AC-055 — Exclusion order: blocked address wins

**Given** a recipient whose address is in the blocked-address register, who is also opted out of the
mailing's list, and whose address was already contacted by this mailing.
**When** the message is prepared with `use_exclusion_list` true.
**Then** exactly one delivery record is created with `trace_status` = `cancel` and `failure_type` =
`mail_bl`; no outgoing message is created.

### MKT-AC-056 — Switching the exclusion list off

**Given** the same recipient with `use_exclusion_list` false.
**Then** the blocked check does not apply and the next matching rule decides: the opted-out check
produces `cancel` with `failure_type` = `mail_optout`.

### MKT-AC-057 — Missing and unusable addresses

**Given** a recipient with no address at all on a mailing whose `keep_archives` is false.
**Then** the delivery record is `cancel` with `failure_type` = `mail_email_missing`.
**And given** a recipient whose only address is `not an address`, the delivery record is `cancel`
with `failure_type` = `mail_email_invalid`.
**And given** the same two recipients on a mailing whose `keep_archives` is true, both delivery
records are `error` instead of `cancel`, with the same failure types.

### MKT-AC-058 — Two contacts sharing one address, one opted in

**Given** the list `Newsletter` holding the contacts `Gilberte` and `Gilberte En Mieux`, both with
the address `gilberte@example.com`, the first opted in and the second opted out.
**When** the mailing addressed to `Newsletter` is sent.
**Then** the address is **not** in the opted-out set, because the contact is opted in to at least one
of the mailing's lists; the first prepared message is kept, and the second is cancelled with
`failure_type` = `mail_dup` by the same-run duplicate rule, because the subject, the body and the
attachment count are identical.

### MKT-AC-059 — The same two contacts with a personalised body

**Given** the same two contacts and a body containing the contact name as a placeholder.
**When** the mailing is sent.
**Then** the two rendered bodies differ, the same-run duplicate rule does not match, and **two**
messages are sent to the same address.

### MKT-AC-060 — Already contacted by the same mailing

**Given** a mailing that already has a delivery record for the contact `ada@example.com`.
**When** the mailing is queued again and the pass runs.
**Then** that contact is not in the remaining recipients at all, so no second delivery record is
created for them.

### MKT-AC-061 — Cancelled messages still produce a delivery record

**Given** a pass in which 38 of 1000 recipients are cancelled by the exclusion checks.
**Then** 1000 delivery records exist afterwards, of which 38 have `trace_status` = `cancel`, and 962
outgoing messages were created; the mailing's `expected` counter is 1000 and its `canceled` counter
is 38.

### MKT-AC-062 — Sending with an empty remaining audience

**Given** a mailing whose remaining recipients are empty and a caller that supplied no explicit keys.
**When** the direct send operation is invoked.
**Then** it is refused with *"There are no recipients selected."*

---

## Cancelling, retrying and resuming

### MKT-AC-063 — Cancel returns the mailing to draft

**Given** a mailing in `in_queue` with `schedule_type` = `scheduled` and `schedule_date`
2026-03-05 08:00:00.
**When** the author presses Cancel.
**Then** `state` is `draft`, `schedule_date` is empty, `schedule_type` is `now` and the next
departure is cleared. Delivery records created by an earlier pass are **kept**.

### MKT-AC-064 — Cancelled then sent again skips the already-traced recipients

**Given** a cancelled mailing that already has 200 delivery records out of an audience of 1000.
**When** it is sent again.
**Then** the remaining recipients are 800 and 800 further delivery records are created.

### MKT-AC-065 — Retry deletes the failed deliveries

**Given** a mailing in `done` with 962 outgoing messages of which 38 are in the failure state, each
carrying one delivery record.
**When** the author presses Retry.
**Then** the 38 delivery records and the 38 outgoing messages are deleted, in pages of at most 1000,
the mailing is written to `in_queue`, and the queue job is woken for the current moment.
**And when** the queue job runs, those 38 recipients are prepared again because they no longer have a
delivery record.

### MKT-AC-066 — Retry is offered only when something failed

**Given** a mailing in `done` whose failure counter is 0.
**Then** the Retry button is not offered.

### MKT-AC-067 — Retry for text messages

**Given** a text-message mailing in `done` with 12 outgoing text messages in error.
**When** the author retries.
**Then** those 12 outgoing text messages and their delivery records are deleted and the mailing is
queued again.

### MKT-AC-068 — An interrupted pass resumes exactly

**Given** a pass over 1000 recipients that is interrupted by the runner's time budget after 7
committed batches of 50.
**Then** 350 delivery records exist, the mailing is still `sending`, and the next run computes 650
remaining recipients and continues from there. No recipient receives a second message.

### MKT-AC-069 — Cancel is offered only while queued

**Given** a mailing in `draft`, one in `sending` and one in `done`.
**Then** none of them offers the Cancel button; only a mailing in `in_queue` does.

### MKT-AC-070 — Duplicate is offered only when finished

**Given** a mailing in `draft`.
**Then** the header does not offer Duplicate; a mailing in `done` does.

### MKT-AC-071 — The acting identity of the queue job

**Given** a queued mailing whose responsible user has the language `fr_BE` and the time zone
`Europe/Brussels`.
**When** the queue job processes it.
**Then** the rendering, the number formatting and the date formatting use that language and that time
zone, not the job runner's.
**And given** the mailing has no responsible user, the last writer is used; when it has neither, the
job's own user is used.

### MKT-AC-072 — Editing guards while sending

**Given** a mailing in `sending`.
**Then** the subject, the preview, the sender address, the answer mode and address, the attachments,
the recipient entity, the mailing lists, the saved filter, the condition, the body, the campaign, the
medium, the name, the mail server and the archive-keeping flag are all read-only, and the
comparison-test controls and the exclusion-list flag are read-only as well.

---

## Comparison testing

### MKT-AC-073 — Enabling a comparison test creates a campaign

**Given** a draft mailing with the subject `Spring sale` and no campaign.
**When** comparison testing is enabled and the mailing is saved.
**Then** a Campaign is created named `A/B Test: Spring sale`, owned by the mailing's responsible,
carrying `ab_testing_schedule_datetime` 2026-03-03 09:00:00 and
`ab_testing_winner_selection` = `opened_ratio`; the mailing points at it; and the comparison-test job
receives a wake request for that moment.

### MKT-AC-074 — An existing campaign is never replaced

**Given** a draft mailing already attached to the campaign `Spring 2026`.
**When** comparison testing is enabled.
**Then** the mailing keeps `Spring 2026` and no campaign is created.

### MKT-AC-075 — Clearing the campaign while testing is enabled is refused

**Given** a mailing with comparison testing enabled and a campaign.
**When** the campaign is written empty while the flag stays true.
**Then** the write is refused with *"A campaign should be set when A/B test is enabled"*.

### MKT-AC-076 — Sampling of the first version

**Given** an audience of 1000 contacts, a campaign with no delivery record yet, and version A at 20
percent.
**When** version A is sent.
**Then** the sample size is `max(truncate(1000 ÷ 100 × 20), 1)` = 200, the remaining set is the whole
1000, and 200 delivery records are created.

### MKT-AC-077 — Sampling of the second version

**Given** the same campaign after version A, and version B at 20 percent.
**When** version B is sent.
**Then** the already-mailed set holds the 200 keys of version A, the remaining set holds 800, the
sample size stays 200, and 200 delivery records are created from those 800. Four hundred distinct
people have been contacted and nobody twice.

### MKT-AC-078 — A percentage too small for the audience

**Given** an audience of 10 contacts and one version at 2 percent.
**Then** the sample size is `max(truncate(10 ÷ 100 × 2), 1)` = `max(0, 1)` = 1 and exactly one
message is sent.

### MKT-AC-079 — A percentage larger than the remainder

**Given** an audience of 150 contacts of which 140 have already been traced by the campaign, and a
version at 20 percent.
**Then** the computed sample size 30 exceeds the remainder of 10, so the sample size is reduced to
10 and ten messages are sent.

### MKT-AC-080 — Percentages that do not divide evenly

**Given** an audience of 150 contacts, version 1 at 10 percent and version 2 at 20 percent, sent in
that order.
**Then** version 1 sends 15 messages, version 2 sends 30 drawn from the remaining 135, and 45
distinct contacts have been reached.

### MKT-AC-081 — Manual winner selection

**Given** the campaign of scenario MKT-AC-080 with both versions in `done`.
**When** the author presses "Send this as winner" on version 1.
**Then** a copy of version 1 is created with `ab_testing_pc` = 100 and the name *" A/B Testing V1
(final)"* built from the original name, the campaign records the copy as its winner,
`ab_testing_completed` becomes true, the copy is queued immediately and its form opens.

### MKT-AC-082 — The winner send reaches only the untouched remainder

**Given** the campaign of scenario MKT-AC-077, where 400 of 1000 contacts have been traced.
**When** the winner copy is sent.
**Then** its remaining recipients are 1000 − 400 = 600, and 600 delivery records are created. Across
the whole campaign each of the 1000 contacts received exactly one message.

### MKT-AC-083 — Automatic winner selection by open rate

**Given** version A with 96 opened out of 200 and version B with 70 opened out of 200, no bounce and
no error on either, and a decision moment of 2026-03-03 09:00:00.
**When** the comparison-test job runs at that moment.
**Then** version A's open rate is `round(100 × 96 ÷ 200, 2)` = 48.00 and version B's is
`round(100 × 70 ÷ 200, 2)` = 35.00, version A wins, and the winner copy is created and queued.

### MKT-AC-084 — Promoting a version that is not part of a test

**Given** a mailing whose comparison-test flag is false.
**When** "Select as winner" is invoked on it.
**Then** the operation is refused with *"A/B test option has not been enabled"*.

### MKT-AC-085 — Two campaigns in one winner operation

**Given** two mailings belonging to two different campaigns, selected together.
**When** the winner operation is invoked.
**Then** it is refused with *"To send the winner mailing the same campaign should be used by the
mailings"*.

### MKT-AC-086 — A completed campaign refuses a second winner

**Given** a campaign whose `ab_testing_completed` is true.
**When** the winner operation is invoked again.
**Then** it is refused with *"To send the winner mailing the campaign should not have been
completed."*

### MKT-AC-087 — No version has been sent

**Given** a campaign with two versions, both in `draft`, and a criterion of highest open rate.
**When** the winner operation is invoked by hand.
**Then** it is refused with *"No mailing for this A/B testing campaign has been sent yet! Send one
first and try again later."*
**And when** the comparison-test job reaches the same campaign, it skips it silently and raises
nothing.

### MKT-AC-088 — Comparing versions without a campaign

**Given** a mailing with comparison testing enabled but no campaign, reached through an integration
that bypassed the write guard.
**When** "Compare Version" is invoked.
**Then** it is refused with *"No mailing campaign has been found"*.

### MKT-AC-089 — Tie between two versions

**Given** two versions of one campaign, both in `done`, both with an open rate of exactly 40.00.
**Then** the winner is the first of the two in the campaign's reading order; no explicit tie-break
exists. **Industry-standard default:** a rebuild should break the tie on the lowest mailing key and
state that it does so, which is deterministic and reproduces the observed choice in the common case
where the versions were created in key order.

---

## Indicators, ratios and rounding

### MKT-AC-090 — The five indicators of a finished mailing

**Given** a mailing with 1000 delivery records: 550 with status `sent`, 370 `open`, 30 `reply`, 12
`bounce`, 38 `error`, none `cancel`, and 962 records carrying a sent moment.
**Then** `delivered` is 950, `opened` is 400, `replied` is 30, `bounced` is 12, `failed` is 38,
`canceled` is 0, `expected` is 1000, `sent` is 962, and the four ratios are
`received_ratio` = 95.00, `opened_ratio` = 42.11, `replied_ratio` = 3.16 and
`bounced_ratio` = 1.25.

### MKT-AC-091 — Denominators of the four ratios differ

**Given** the same mailing.
**Then** the denominators used are 1000 for reception, 950 for opening and replying, and 962 for
bouncing, so a rebuild that uses one denominator for all four produces different numbers.

### MKT-AC-092 — An empty mailing reports zero

**Given** a mailing with no delivery record at all.
**Then** every counter is 0 and every ratio is 0.00, because each zero denominator is replaced by 1
and no division fails.

### MKT-AC-093 — A mailing whose records were all cancelled

**Given** a mailing with 40 delivery records, all with status `cancel`.
**Then** `expected` is 40, `canceled` is 40, the reception denominator is 40 − 40 = 0 which is
replaced by 1, and `received_ratio` is 0.00.

### MKT-AC-094 — Rounding half away from zero

**Given** 1 opened out of 8 counted records.
**Then** `opened_ratio` is `round(100 × 1 ÷ 8, 2)` = `round(12.5, 2)` = 12.50.
**And given** 1 opened out of 1600, the value is `round(0.0625, 2)` = 0.06.
**And given** 1 opened out of 1600 counted the other way, 3 opened out of 1600 gives
`round(0.1875, 2)` = 0.19.

### MKT-AC-095 — The click ratio counts recipients, not clicks

**Given** 962 counted delivery records — those whose status is not `bounce`, `cancel` or `error` —
of which 173 have at least one recorded visit, one of them having clicked five links five times
each.
**Then** `clicks_ratio` is `round(100 × 173 ÷ 962, 2)` = 17.98, and the click counters of the five
Link Tracker records total 25.

### MKT-AC-096 — Campaign indicators use one denominator

**Given** a campaign holding the 1000 delivery records of scenario MKT-AC-090.
**Then** the campaign reports `received_ratio` 95.00, `opened_ratio` 40.00, `replied_ratio` 3.00 and
`bounced_ratio` 1.20, and the campaign open rate 40.00 differs from the mailing open rate 42.11
computed over the same records.

### MKT-AC-097 — Campaign delivered figure is derived by subtraction

**Given** the same campaign.
**Then** its delivered figure is `sent − bounce` = 962 − 12 = 950, whereas the mailing's delivered
figure is the sum of the statuses `sent`, `open` and `reply` = 950. Both must be reproduced even
though they agree here, because they diverge whenever a bounced record also carries a sent moment
that a status change later overwrote.

### MKT-AC-098 — Audience size shown before sending

**Given** an audience of 1000, comparison testing enabled and a percentage of 20.
**Then** the `total` shown on the form is 200.
**And given** the percentage is 100, `total` is 1000.
**And given** comparison testing is disabled, `total` is 1000.

### MKT-AC-099 — List quality percentages

**Given** a list with 1204 subscriptions, of which 37 are opted out and 11 have a blocked address,
1150 have a usable address that is neither opted out nor blocked, and 64 distinct contacts have at
least one bounce.
**Then** `contact_count` is 1204, `contact_count_email` is 1150, `contact_count_opt_out` is 37,
`contact_count_blacklisted` is 11, `contact_pct_opt_out` is 3.0731 percent,
`contact_pct_blacklisted` is 0.9136 percent and `contact_pct_bounce` is 5.3156 percent. The three
percentages are not rounded at computation time.

### MKT-AC-100 — An empty list reports zero quality

**Given** a list with no subscription.
**Then** all four counters and all three percentages are exactly 0 and no division is performed.

### MKT-AC-101 — The display name of a list carries its size

**Given** the list `Newsletter` with 1204 subscriptions.
**Then** its display name is `Newsletter (1204)`.

---

## Link tracking and redirection

### MKT-AC-102 — A body link becomes a short link

**Given** a body containing one anchor pointing at `https://example.com/spring` whose text is
`Discover the sale`.
**When** the mailing is sent.
**Then** one Link Tracker is created with that target address, the label `Discover the sale`, and the
campaign, source, medium and mailing of the mailing; one Link Tracker Code is created for it; and the
anchor in each outgoing body points at `<short host>/r/<code>/m/<delivery record key>`.

### MKT-AC-103 — Two links with the same target but different labels

**Given** a body containing two anchors pointing at `https://example.com/spring`, one labelled
`Discover the sale` and one labelled `Shop now`.
**Then** two Link Tracker records are created, with two different codes, counted separately.

### MKT-AC-104 — A long label is truncated

**Given** an anchor whose text is 57 characters long.
**Then** the stored label is the first 40 characters of the trimmed text.

### MKT-AC-105 — An image anchor

**Given** an anchor whose only child is an image whose alternative text is `Spring banner`.
**Then** the label is `[media] Spring banner`.
**And given** an image with no alternative text whose source ends in `banner.png`, the label is
`[media] banner.png`.

### MKT-AC-106 — Links that are never shortened

**Given** a body containing an electronic-mail link, a telephone link, a text-message link, a link
already pointing at the short prefix, and links pointing at `<base>/unsubscribe_from_list`,
`<base>/view` and `<base>/cards/9/preview`.
**Then** none of them is shortened and no Link Tracker is created for them.

### MKT-AC-107 — Uniqueness of a Link Tracker

**Given** an existing tracker with the target `https://example.com/spring`, no campaign, no medium,
no source and an empty label.
**When** a second tracker is created with the same target, no campaign, no medium, no source and the
label set to the empty string.
**Then** the creation is refused with *"Combinations of Link Tracker values (URL, campaign, medium,
source, and label) must be unique."* followed by a line break, *"The following combinations are
already used: "*, a line break and the line `- ('https://example.com/spring', '', '', '', '')`.

### MKT-AC-108 — Creating a tracker without a target

**Given** a creation with no target address.
**Then** it is refused with *"Creating a Link Tracker without URL is not possible"*.

### MKT-AC-109 — A target that loops on the current page

**Given** a target address of `#section-2`.
**Then** the creation is refused with *"“%s” is not a valid link, links cannot redirect to the
current page."*, the placeholder being `#section-2`.

### MKT-AC-110 — A bare host is completed

**Given** a target address of `example.com`.
**Then** the stored address is `http://example.com`.

### MKT-AC-111 — Visitor cookies never attribute a new tracker

**Given** a visitor whose cookies name a campaign, a source and a medium, creating a tracker through
the shortened-link page without naming any of the three.
**Then** the created tracker has an empty campaign, an empty source and an empty medium.

### MKT-AC-112 — A plain visit

**Given** a Link Tracker Code `aK7` pointing at a tracker whose target is
`https://example.com/spring`, whose campaign is `Spring 2026` and whose medium is `Email`.
**When** an ordinary visitor requests `/r/aK7` from a network address resolved to Belgium.
**Then** one Link Tracker Click is created with that tracker, that network address, the country
Belgium and the campaign `Spring 2026`; the tracker's `count` becomes 1; and the visitor is
redirected permanently to
`https://example.com/spring?utm_campaign=Spring+2026&utm_medium=Email`.

### MKT-AC-113 — A visit by an automated agent

**Given** the same code requested by a caller whose agent string matches a known preview fetcher.
**Then** no visit is recorded, the tracker's `count` stays as it was, and the redirection still
happens.

### MKT-AC-114 — A visit carrying a recipient

**Given** the delivery record 4711 of the mailing 87, whose status is `sent`.
**When** `/r/aK7/m/4711` is requested, even by an automated agent.
**Then** one visit is recorded carrying the tracker, the delivery record, its campaign and its
mailing; the delivery record becomes `open` with `open_datetime` = the current moment; and its
`links_click_datetime` is set to the current moment. A second click ten minutes later leaves the
status `open` and overwrites `links_click_datetime` with the later moment.

### MKT-AC-115 — External tracking suppressed

**Given** the parameter `link_tracker.no_external_tracking` set and a target host different from the
site host.
**Then** the redirection address is the target address unchanged, with no campaign parameter
appended.
**And given** a target on the site host, the parameters are appended as usual, and any literal
sequence of three dots in a parameter value is replaced by its escaped form.

---

## Campaign tracking and unique names

### MKT-AC-116 — Cookies are written after a tracked visit

**Given** a visitor requesting a page with the query parameters `utm_campaign=Spring`,
`utm_source=Newsletter` and `utm_medium=Email`, and no matching cookie yet.
**Then** three cookies are written on the request host, named `odoo_utm_campaign`,
`odoo_utm_source` and `odoo_utm_medium`, classified as optional and therefore subject to the
visitor's consent choices, each with a lifetime of 31 days, that is 2 678 400 seconds.

### MKT-AC-117 — An unchanged cookie is not rewritten

**Given** the same visitor whose `odoo_utm_campaign` cookie already holds `Spring`.
**When** they request another page carrying `utm_campaign=Spring`.
**Then** no cookie is written.

### MKT-AC-118 — Stamping a created record

**Given** a visitor carrying those three cookies who submits a public form that creates a Lead.
**Then** the created Lead carries the campaign `Spring`, the source `Newsletter` and the medium
`Email`, each resolved by a case-insensitive search that includes archived records, creating the
record when nothing matches.

### MKT-AC-119 — A salesperson never inherits a visitor's attribution

**Given** a signed-in salesperson creating a Lead by hand in the back office while their browser
carries the three cookies.
**Then** the created Lead carries no campaign, no source and no medium.

### MKT-AC-120 — An implicitly created campaign is hidden

**Given** a visitor cookie naming the campaign `Winter blast`, which does not exist.
**Then** a Campaign named `Winter blast` is created with `is_auto_campaign` true, and it does not
appear in the campaign menu, whose action excludes automatically generated campaigns.

### MKT-AC-121 — Unique names fill the lowest free counter

**Given** the name `Spring new` already taken.
**When** the names `Spring 1`, `Spring 2`, `Spring new`, `Spring new` and `Spring new [0]` are
requested in that order.
**Then** the results are `Spring 1`, `Spring 2`, `Spring new [2]`, `Spring new [3]` and
`Spring new [4]`.

### MKT-AC-122 — Holes are reused

**Given** that the holders of `Spring new`, `Spring new [2]` and `Spring new [3]` are then deleted
while `Spring new [4]` remains.
**When** four further `Spring new` are requested.
**Then** the results are `Spring new`, `Spring new [2]`, `Spring new [3]` and `Spring new [5]`,
because counter 4 is still taken.

### MKT-AC-123 — Campaign name derived from the title

**Given** a Campaign created with only the title `Spring sale`.
**Then** `name` is `Spring sale`, and a second campaign with the same title receives
`Spring sale [2]`.

### MKT-AC-124 — Campaign created with only a name

**Given** a Campaign created with only the name `Spring sale` and no title.
**Then** `title` is set to `Spring sale` as well.

### MKT-AC-125 — Medium fetch-or-create is insensitive to spelling

**Given** no medium registered under the external key derived from `New Medium`.
**When** the fetch-or-create operation is called successively with `New Medium`, `new medium`,
`new_medium` and `new.Medium`.
**Then** exactly one Campaign Medium is created and all four calls return it, because the key is the
lower-cased name with every space and dot replaced by an underscore.

---

## Lists, contacts, subscriptions, import and merge

### MKT-AC-126 — A contact may not be created with both membership views

**Given** a creation naming both the list membership and the subscription list.
**Then** it is refused with *"You should give either list_ids, either subscription_ids to create new
contacts."*

### MKT-AC-127 — Default lists of the calling screen are merged once

**Given** a contact screen opened from the list `Newsletter`, so that the screen supplies that list
as a default, and a creation that already supplies a subscription to `Newsletter`.
**Then** exactly one subscription to `Newsletter` is created, not two.

### MKT-AC-128 — Duplicating a contact inside a list

**Given** the contact `Ada` who is a member of `Newsletter` and `Beta testers`, duplicated from
inside the `Newsletter` screen.
**Then** the copy has exactly two subscriptions, to `Newsletter` and `Beta testers`, and the screen's
default list is not added a second time.

### MKT-AC-129 — Creating a contact from a typed string

**Given** the typed string `"Ada Byron" <ada@example.com>`.
**Then** a Mailing Contact is created with `name` = `Ada Byron` and `email` = `ada@example.com`.

### MKT-AC-130 — The contact opt-out column outside a list context

**Given** the Mailing Contact list opened from the main menu, with no single list in the screen
context.
**Then** the `opt_out` value of every row is false and a search on it returns nothing; the column is
hidden.

### MKT-AC-131 — The contact opt-out column inside a list context

**Given** the same list opened from the list `Newsletter`, and the contact `Ada` opted out of
`Newsletter` but opted in to `Beta testers`.
**Then** her `opt_out` value shows true.

### MKT-AC-132 — Subscription uniqueness

**Given** the contact `Ada` already subscribed to `Newsletter`.
**When** a second subscription of `Ada` to `Newsletter` is created.
**Then** it is refused with *"A mailing contact cannot subscribe to the same mailing list multiple
times."*

### MKT-AC-133 — Recording a reason is an opt-out

**Given** an opted-in subscription.
**When** only `opt_out_reason_id` is written.
**Then** `opt_out` becomes true and `opt_out_datetime` becomes the current database moment.

### MKT-AC-134 — Opting back in clears the moment

**Given** an opted-out subscription with `opt_out_datetime` 2026-02-20 11:00:00.
**When** `opt_out` is written false.
**Then** `opt_out_datetime` is cleared.

### MKT-AC-135 — Ordinary paste import

**Given** the lists `Newsletter` (first) and `Partners` (third) chosen, and the eleven pasted lines
of which four parse to nothing and the remaining seven are `alice@example.com`,
`bob@example.com`, `"Bob" <bob@EXAMPLE.com>`, `"Test" <bob@example.com>`,
`already_exists_list_1@example.com`, `already_exists_list_2@example.com` and
`"Test" <already_exists_list_1_and_2@example.com>`, where `already_exists_list_1@example.com` is
already a member of `Newsletter`, `already_exists_list_2@example.com` only of a second list and
`already_exists_list_1_and_2@example.com` of both.
**When** the import is confirmed.
**Then** five contacts are created and subscribed to both chosen lists; `Newsletter` gains
`alice@example.com`, `Bob <bob@example.com>` and `already_exists_list_2@example.com`; the existing
contact of `already_exists_list_1@example.com` gains `Partners` instead of being duplicated; the
second list is untouched; and the notice reads *"Contacts successfully imported. Number of contacts
imported: 5"* followed by *". Number of duplicates ignored: 2"* because seven pairs were parsed.

### MKT-AC-136 — Paste import with nothing parsable

**Given** three lines none of which yields an address.
**Then** nothing is created and the warning *"No valid email address found."* is shown.

### MKT-AC-137 — Paste import above the limit

**Given** 5001 parsed addresses.
**Then** nothing is created, the warning *"You have to much emails, please upload a file."* is shown,
and the general file-import assistant opens on Mailing Contact.

### MKT-AC-138 — Paste import where everybody is already a member

**Given** three addresses all of which already belong to the chosen list.
**Then** nothing is created and the warning *"No contacts were imported. All email addresses are
already in the mailing list."* is shown.

### MKT-AC-139 — The first non-empty name wins

**Given** the lines `bob@example.com`, `"Bob" <bob@example.com>` and `"Robert" <bob@example.com>`.
**Then** one contact is created whose name is `Bob`.

### MKT-AC-140 — Add selected contacts to a list

**Given** five selected contacts of which two are already members of `Partners`.
**When** the add-to-list assistant is confirmed with `Partners`.
**Then** three subscriptions are created and the notice *"3 Mailing Contacts have been added. "* is
shown, with the trailing space.
**And when** "Add and Send Mailing" was pressed instead, a new mailing form opens with `Partners`
pre-selected.

### MKT-AC-141 — Merging two lists

**Given** list A holding `sam@example.com` and `sam@example.net`, list B holding `sam@example.com`
and `sam@example.org`, and an empty list C.
**When** A and B are merged into C.
**Then** C receives three subscriptions: `sam@example.com` once, `sam@example.net` and
`sam@example.org`.

### MKT-AC-142 — The merge skips opted-out and blocked contacts

**Given** list A holding `ada@example.com` opted out of A, and `bob@example.com` whose address is in
the blocked-address register.
**When** A is merged into C.
**Then** neither contact is inserted into C.

### MKT-AC-143 — Archiving a list used by an unfinished mailing

**Given** the list `Newsletter` referenced by a mailing in `in_queue`.
**When** the list is archived, whether directly or as a source of a merge with the archive option on.
**Then** the operation is refused with *"At least one of the mailing list you are trying to archive is
used in an ongoing mailing campaign."* and, in the merge case, the whole merge fails.

---

## Public pages, tokens and feedback

### MKT-AC-144 — Unsubscribing from a list-based mailing

**Given** the mailing 87 addressed to the lists `Newsletter` and `Partners`, the recipient record
1337 and the address `jane@example.com`, which belongs to one contact opted in to both lists.
**When** the recipient opens
`/mailing/87/unsubscribe?document_id=1337&email=jane%40example.com&hash_token=<valid token>`.
**Then** both subscriptions become opted out with the current moment stamped, a note is posted on the
contact reading *"<contact display name> unsubscribed from the following mailing list(s)"* followed
by the two list names, and the page is rendered with the heading *"You are no longer part of the
Newsletter, Partners mailing list."* when both lists are public.

### MKT-AC-145 — Heading when no list is public

**Given** the same mailing whose two lists are both private.
**Then** the heading is *"You are no longer part of our mailing list(s)."*

### MKT-AC-146 — Heading with exactly one list

**Given** a mailing addressed to the single public list `Newsletter`.
**Then** the heading is *"You are no longer part of the Newsletter mailing list."*

### MKT-AC-147 — One-click unsubscribe

**Given** the same parameters as MKT-AC-144.
**When** a mail client posts to `/mailing/87/unsubscribe_oneclick` without a session.
**Then** the same unsubscription happens, the answer is an empty success, and the cross-site request
token is not required.

### MKT-AC-148 — The confirmation page

**Given** the same parameters.
**When** the recipient opens `/mailing/87/confirm_unsubscribe`.
**Then** nothing is written and the page asks *"Are you sure you want to unsubscribe from the mailing
list "Newsletter, Partners"?"*
**And when** the Unsubscribe button posts to `/mailing/confirm_unsubscribe`, the unsubscription
happens and the page reads *"Successfully unsubscribed!"* with a "Manage Subscriptions" button.

### MKT-AC-149 — Unsubscribing from a mailing addressed to another entity

**Given** the mailing 88 addressed to Sales Order, the record 4242 and the address
`jane@example.com`.
**When** the recipient unsubscribes.
**Then** the address is added to the blocked-address register with the note *"Blocklist request from
unsubscribe link of mailing <link to the mailing> (document <link to the record>)"*, and the page
heading is *"You are no longer part of our services and will not be contacted again."*

### MKT-AC-150 — An anonymous caller without a token

**Given** an anonymous visitor.
**When** they open `/mailing/87/unsubscribe` with no token.
**Then** the answer is a bad request.

### MKT-AC-151 — A token with a missing companion parameter

**Given** a token but no record key.
**Then** the answer is a bad request, because a token is only accepted together with a mailing key,
an address and a record key.

### MKT-AC-152 — A wrong token

**Given** a token that does not equal the keyed hash of the tuple made of the database name, the
mailing key 87, the record key 1337 and the address `jane@example.com`.
**Then** the answer is unauthorized, and the comparison is made in constant time.

### MKT-AC-153 — A missing mailing does not reveal itself

**Given** a page endpoint called with the mailing key 99 999, which does not exist.
**Then** the answer is unauthorized, not not-found, so a caller cannot learn which mailing keys
exist.

### MKT-AC-154 — A signed-in Marketing User needs no token

**Given** a signed-in Marketing User.
**When** they open `/mailing/87/unsubscribe` with no token.
**Then** the page opens and acts on **their own** normalised address; any address supplied in the
request is ignored.

### MKT-AC-155 — A signed-in ordinary user is refused

**Given** a signed-in portal user who is not a Marketing User.
**When** they open `/mailing/87/unsubscribe` with no token.
**Then** the answer is a bad request.

### MKT-AC-156 — The portal preferences page

**Given** a signed-in user whose normalised address is `jane@example.com`.
**When** they open `/mailing/my`.
**Then** the subscription-management page is rendered with no mailing, no record key, no token and
the feedback form disabled.
**And given** the signed-in user has no address at all, the answer is unauthorized.

### MKT-AC-157 — What the page shows

**Given** the address `jane@example.com` belonging to two contacts, the first opted in to
`Newsletter` and opted out of `Partners`, the second opted out of `Newsletter`; and three public
lists that the address belongs to none of.
**Then** the opted-in set holds `Newsletter`, the opted-out set holds `Partners` only — because a
list present in both sets counts as opted in — and the suggested set holds the three public lists,
capped at the ten most recently created.

### MKT-AC-158 — Applying checkbox changes

**Given** the same address, currently opted in to `Newsletter` and `Beta testers`.
**When** the page is applied with only `Beta testers` and the public list `Product news` ticked.
**Then** `Newsletter` is opted out, `Product news` is opted in — a subscription being created on the
first contact — and the answer is 1, the number of lists opted out of.

### MKT-AC-159 — A private list cannot be joined from the page

**Given** a private list `Internal` that the address has never belonged to, whose key is submitted
among the ticked lists.
**Then** no subscription to `Internal` is created.

### MKT-AC-160 — Feedback after an unsubscription

**Given** an unsubscription performed at 09:00:00 and the reason *"I changed my mind"* chosen at
09:04:00 with the free text `Too many messages`.
**Then** the reason is written on every subscription of the address opted out in the last ten
minutes, and the message *"Feedback from jane@example.com"* followed by a line break and
`Too many messages` is posted on the contacts.
**And given** the reason is submitted at 09:15:00, no subscription is touched, because the ten-minute
window has passed.

### MKT-AC-161 — Feedback without a reason

**Given** a feedback submission with no reason.
**Then** the answer is the string `error`.

### MKT-AC-162 — Excluding and re-including oneself

**Given** the subscription page with the parameter `mass_mailing.show_blacklist_buttons` true.
**When** "Exclude Me" is pressed with a mailing in context.
**Then** the address is added to the blocked-address register with the note *"Blocklist request from
portal of mailing <link> (document <link>)"* and the answer is true.
**And when** "Come Back" is pressed with both a mailing and a record key in context.
**Then** the register entry is archived with the note *"Blocklist removal request from portal of
mailing <link> (document <link>)"* and the answer is true.

### MKT-AC-163 — Open tracking

**Given** the outgoing mail 5150 belonging to a mailing, whose delivery record 4711 has the status
`sent`.
**When** `/mail/track/5150/<valid token>/blank.gif` is requested.
**Then** the delivery record becomes `open` with the current moment as `open_datetime`, and the
answer is a one-pixel transparent image with the image content type.
**And given** an invalid token, the answer is unauthorized and nothing is written.

---

## Text message mailings

### MKT-AC-164 — Creating a text-message mailing

**Given** a new mailing whose type is set to `sms` and whose title is `Flash sale`.
**Then** `subject` becomes `Flash sale`, `medium_id` becomes `Text Message`, `keep_archives`
defaults to true, and the body used is `body_plaintext`.

### MKT-AC-165 — Seeding the body from a template

**Given** a text-message template whose body is `Our sale starts now`.
**When** it is selected on the mailing.
**Then** `body_plaintext` becomes `Our sale starts now`.

### MKT-AC-166 — Text-message exclusions

**Given** four recipients: one whose sanitised number is in the blocked-number register, one opted
out of the mailing's list, one whose number was already used earlier in the same run, and one whose
number cannot be sanitised.
**Then** four delivery records are created, all cancelled, with the failure types `sms_blacklist`,
`sms_optout`, `sms_duplicate` and `sms_number_format` respectively; a fifth recipient with no number
at all produces `sms_number_missing`.

### MKT-AC-167 — An entity with no telephone field

**Given** a text-message mailing whose recipient entity exposes neither a telephone field nor a
linked contact.
**When** the already-contacted set is computed.
**Then** the operation is refused with *"Unsupported %s for mass SMS"*, the placeholder being the
recipient entity name.

### MKT-AC-168 — The opt-out sentence

**Given** a text-message mailing with the key 87 and the opt-out option on.
**When** a message is prepared for a recipient and the generated code is `k9Q`.
**Then** the delivery record carries `sms_code` = `k9Q` and the sent body ends with a line break and
*"STOP SMS: <base>/sms/87/k9Q"*.

### MKT-AC-169 — Short links carry the message key

**Given** a body containing one shortened link and an outgoing text message whose key is 4213.
**Then** the body actually sent contains `<short host>/r/<code>/s/4213`.

### MKT-AC-170 — Delivery reports drive the statuses

**Given** a delivery record in `outgoing`.
**When** the provider reports `process`, then `pending`, then delivery.
**Then** the record moves to `process`, then `pending` with `sent_datetime` filled, then `sent`; and
the mailing moves to `sending` on the first report and to `done` once no delivery record of the
mailing is left in `process`.

### MKT-AC-171 — A late report is ignored

**Given** a delivery record already in `sent`.
**When** a report arrives saying `pending`.
**Then** the record is unchanged, because `pending` is in the ignore set of a record already `sent`.

### MKT-AC-172 — Text-message opt-out, entry page

**Given** the mailing 87, the code `k9Q` and a delivery record whose stored number is
`+32470123456`.
**When** the visitor opens `/sms/87/k9Q` and submits `0470 12 34 56` while their network address
resolves to Belgium.
**Then** the number is sanitised to `+32470123456`, it matches the stored number, and the visitor is
redirected to the confirmation page.
**And when** they submit `12`, the page reloads with the error *"Oops! The phone number seems to be
incorrect. Please make sure to include the country code."*
**And when** they submit a valid but different number, the page reloads with *"Oops! Number not
found"*.

### MKT-AC-173 — Text-message opt-out, confirmation for a list-based mailing

**Given** the mailing 87 addressed to `Newsletter` and `Partners`, and a contact carrying
`+32470123456` subscribed to both.
**When** `/sms/87/unsubscribe/k9Q` completes.
**Then** both subscriptions become opted out and the page reports the two lists left, together with
the other lists that number is still opted in to.

### MKT-AC-174 — Text-message opt-out, confirmation for another entity

**Given** the mailing 88 addressed to Sales Order.
**When** the same page completes.
**Then** the number is added to the blocked-number register with the note *"Blacklist through SMS
Marketing unsubscribe (mailing ID: 88 - model: Sales Order)"*.

### MKT-AC-175 — An unknown mailing or code

**Given** `/sms/99999/k9Q` where the mailing does not exist, or `/sms/87/zzz` where no delivery
record carries that code.
**Then** the visitor is redirected to the back office rather than being told what failed.

---

## Marketing cards

### MKT-AC-176 — Creating a card campaign

**Given** a new Marketing Card Campaign named `Speaker cards`, whose preview record is the Event
Track `Opening keynote` and whose target address is empty.
**When** it is saved.
**Then** a Link Tracker is created with the site base address as target, the campaign name as title,
the shipped `Marketing Card` source, and a label of the form
`marketing_card_campaign_Speaker cards_2026-03-02 09:00:00`; `res_model` becomes the Event Track
entity; and the preview image is rendered and stored on the campaign.

### MKT-AC-177 — The target entity cannot change once cards exist

**Given** the same campaign with 40 cards produced.
**When** the preview record is changed to a Contact, which would change `res_model`.
**Then** the write is refused with *"Model of campaign %(campaign)s may not be changed as it already
has cards"*.

### MKT-AC-178 — Writing a design field invalidates every card

**Given** a campaign with 40 cards, 5 of them archived, all with `requires_sync` false.
**When** the header colour is changed.
**Then** all 40 cards, archived ones included, have `requires_sync` true.

### MKT-AC-179 — Refreshing the cards

**Given** a campaign whose current condition matches 250 records, of which 210 already have an
up-to-date card.
**When** "Update cards" is pressed.
**Then** 40 cards are created, every matching card is set active, and the rendering runs in batches
of at most 100 with a commit and a cache drop between batches.

### MKT-AC-180 — Queueing is refused while a card is missing

**Given** a mailing carrying a card campaign whose `card_requires_sync_count` is 3.
**When** the author presses Send.
**Then** the operation is refused with *"You should update all the cards for %(mailing)s before
scheduling a mailing."*, the placeholder being the mailing display name.

### MKT-AC-181 — Card recipients only

**Given** a mailing carrying a card campaign whose condition matches 250 records, of which 240 have a
card.
**Then** the audience is the 240 records that have a card.

### MKT-AC-182 — Duplicate suppression is disabled for cards

**Given** two recipients sharing the address `family@example.com`, each with their own card.
**When** the mailing is sent.
**Then** the already-contacted set is empty, both messages are kept, and each carries its own card
address, because every recipient receives a different image.

### MKT-AC-183 — The preview page marks the card visited

**Given** a card whose `share_status` is empty.
**When** the person opens `/cards/<slug>/preview`.
**Then** `share_status` becomes `visited`, written with elevated rights, and the page shows the
request title, the request description, the image, the post suggestion and the share buttons.

### MKT-AC-184 — A crawler marks the card shared

**Given** the same card, now `visited`.
**When** a caller whose agent string contains `LinkedInBot` requests `/cards/<slug>/card.jpg`.
**Then** `share_status` becomes `shared` and the answer is the image bytes with the image content
type and the download name `card.jpg`.
**And when** an ordinary person requests the same address, the status is unchanged.

### MKT-AC-185 — A card with no image

**Given** a card whose image is empty.
**When** `/cards/<slug>/card.jpg` is requested.
**Then** the answer is not found.

### MKT-AC-186 — The redirection endpoint

**Given** an active card of a campaign whose target address is `https://example.com/tickets`.
**When** a recognised crawler requests `/cards/<slug>/redirect`.
**Then** the answer is a page carrying only the preview metadata: the image address, the post text
and the target name.
**And when** an ordinary person requests it, they are redirected through the campaign's tracker short
address, which records a click, and land on `https://example.com/tickets`.

### MKT-AC-187 — Campaign counters

**Given** a campaign with 100 cards, of which 30 are `visited` and 12 are `shared`.
**Then** `card_count` is 100, `card_click_count` is 42 and `card_share_count` is 12.

### MKT-AC-188 — Card cleanup

**Given** `marketing_card.card_image_cleanup_interval_days` = 60 and a card last written on
2025-12-01.
**When** the cleanup routine runs on 2026-03-02.
**Then** the card is deleted.
**And given** the parameter set to 0, nothing is deleted.

---

## Configuration, permissions, deletion guards and multi-site

### MKT-AC-189 — Turning campaigns on and off

**Given** the campaign switch off and the comparison-test job inactive.
**When** the switch is turned on and the settings are saved.
**Then** every internal user gains the campaign-management group, the campaign menu, the Campaign
Stages menu and the Campaign Tags menu appear, and the comparison-test job becomes active.
**And when** the switch is turned off again, the group is revoked and the job becomes inactive.

### MKT-AC-190 — Turning the dedicated server off clears the choice

**Given** the dedicated-server switch on and `Bulk relay` chosen.
**When** the switch is turned off and the settings are saved.
**Then** `mass_mailing.mail_server_id` is emptied and new mailings are created with no mail server.

### MKT-AC-191 — A server used by an unfinished mailing cannot get an owner

**Given** the server `Bulk relay` used by a mailing in `in_queue`.
**When** a personal owner is set on it.
**Then** the write is refused with *"Cannot set an owner on '%(server)s': it is used by mailing
'%(mailing)s'."*
**And given** the server is the configured dedicated server, the refusal is *"Cannot set an owner on
'%(server)s': it is configured as the dedicated Email Marketing server."*
**And given** the server is used only by mailings in `done` and is not the dedicated server, the
owner may be set.

### MKT-AC-192 — A personal server is never used for a mailing

**Given** no dedicated server configured, one server with a personal owner whose sender filter
matches the mailing's sender, and one server without an owner.
**When** an outgoing mail belonging to a mailing is sent.
**Then** the server with the personal owner is removed from the candidate list and the other one is
used.

### MKT-AC-193 — Deletion guards on the campaign vocabulary

**Given** a Campaign Medium linked to a mailing, a Campaign Source linked to a mailing, the medium
`Email`, the medium `Text Message`, the source `Referral`, the source `Marketing Card` and a Campaign
used by a recruitment source.
**Then** deleting each is refused, with, respectively: *"You cannot delete these UTM Mediums as they
are linked to the following mailings in Mass Mailing:"* followed by the quoted subjects; *"You cannot
delete these UTM Sources as they are linked to the following mailings in Mass Mailing:"* followed by
the quoted subjects; *"Oops, you can't delete the Medium '%s'."* followed by a line break and *"Doing
so would be like tearing down a load-bearing wall — not the best idea."*; *"The UTM medium '%s'
cannot be deleted as it is used in some main functional flows, such as the SMS Marketing."*; *"You
cannot delete the 'Referral' UTM source record."*; *"The UTM source '%s' cannot be deleted as it is
used to promote marketing cards campaigns."*; and *"The UTM campaign '%s' cannot be deleted as it is
used in the recruitment process."*

### MKT-AC-194 — Deleting a mailing

**Given** a mailing with 1000 delivery records and 5 Link Tracker records.
**When** it is deleted.
**Then** the 1000 delivery records are deleted with it, and the 5 Link Tracker records survive with
an empty mailing reference and keep their recorded visits.

### MKT-AC-195 — Deleting a list and deleting a contact

**Given** a list with 1204 subscriptions.
**When** it is deleted.
**Then** the 1204 subscriptions are deleted and the 1204 contacts survive.
**And when** a contact is deleted, its subscriptions are deleted with it.

### MKT-AC-196 — A card campaign belongs to its owner

**Given** two Marketing Card Users, `Ada` and `Bob`, and a campaign whose responsible is `Ada`.
**Then** `Bob` may read the campaign and may create campaigns of his own, but a write or a delete on
`Ada`'s campaign is refused by the record rule; a Marketing Card Manager may do both.

### MKT-AC-197 — Anonymous access is limited to cards

**Given** an anonymous visitor.
**Then** they may read a Marketing Card, which is what makes the public card endpoints work, and they
may read nothing else of this domain: Link Tracker, Link Tracker Code and Link Tracker Click are all
denied to the public group.

### MKT-AC-198 — Per-recipient links use the recipient's own site

**Given** two websites, `shop.example.com` belonging to company A and `store.example.net` belonging
to company B, and a mailing addressed to contacts of both companies.
**When** the mailing is sent.
**Then** each recipient's unsubscribe address, view-in-browser address and open-tracking image are
built on the base address of the website resolved from that recipient's own record, so a contact of
company B receives links on `store.example.net`.

### MKT-AC-199 — The short-address host follows the website and the company

**Given** a current website belonging to the current company.
**Then** `short_url_host` is that website's base address followed by `/r/`.
**And given** no website is resolved, or the resolved website belongs to another company, the company
base address followed by `/r/` is used.

### MKT-AC-200 — The invoiced amount is not converted between currencies

**Given** a campaign whose company's currency is the euro, and two customer invoices attributed to
the campaign's source, one of 1 000.00 euro and one of 1 200.00 United States dollars posted in a
company whose currency is the dollar.
**Then** the campaign's invoiced amount is the unconverted sum of the two signed company-currency
amounts, formatted in euro, and the figure must be read as an indicator rather than as a ledger
balance. **Industry-standard default:** a rebuild that needs a comparable figure should convert each
invoice amount into the campaign's currency at the invoice date rate, following
[../multi-currency/calculations.md](../multi-currency/calculations.md), and state that it does so;
the reference behaviour performs no conversion.
