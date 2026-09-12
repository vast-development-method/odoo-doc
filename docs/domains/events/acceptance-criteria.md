# Acceptance criteria of the Events domain

Behavioural scenarios a replacement must pass, written as Given / When / Then with concrete values. Each criterion is independently verifiable and cites the rule, the workflow or the formula it exercises. Unless stated otherwise, the acting user is an Event User, the event display time zone is `Europe/Brussels` and every amount is expressed in a currency with two decimal places.

---

## 1. Creating and configuring an event

### EV-AC-001 Default dates of a new event

**Given** the current moment is `2026-03-04 08:17:41`
**When** a user opens a new Event form
**Then** the proposed start is `2026-03-04 08:30:00`, the proposed end is `2026-03-05 08:30:00`, the proposed responsible is the acting user, the proposed organiser and venue are the contact of the active company, the proposed stage is the first stage by sequence, the proposed badge layout is `A6`, and the proposed display time zone is the time zone of the acting user.

### EV-AC-002 Rounding of the proposed start

**Given** the current moment is `2026-03-04 08:37:00`
**When** a user opens a new Event form
**Then** the proposed start is `2026-03-04 09:00:00`.

### EV-AC-003 The end date cannot precede the start date

**Given** an event with the start `2026-06-15 09:00`
**When** the user sets the end to `2026-06-14 18:00` and saves
**Then** the save is rejected with *"The closing date cannot be earlier than the beginning date."* and nothing is written (`EV-RULE-002`).

### EV-AC-004 Applying a template copies the seat configuration

**Given** a template with the limited-seats flag on and a maximum of 30
**When** a user creates an event and chooses that template
**Then** the event has the limited-seats flag on and a maximum of 30, and its available seats are 30 while no attendee exists (`EV-RULE-001`, workflow 2).

### EV-AC-005 Applying a template copies the tickets

**Given** a template holding two ticket lines named `First Ticket` and `Second Ticket`
**When** a user creates an event with that template
**Then** the event holds exactly two tickets with those names, each copied with its sequence, description and seat maximum, and with its product and price when the product bridge is installed.

### EV-AC-006 Changing the template keeps the tickets that are already used

**Given** an event created from template A holding a ticket that has one registration, and a second ticket with no registration
**When** the user changes the template to template B, which holds one ticket line
**Then** the ticket carrying the registration is kept, the unused ticket is removed, and one ticket is created from the line of template B.

### EV-AC-007 Changing the template keeps the communications that already ran

**Given** an event with three schedules, one of which is already marked sent and one of which has per-attendee traces
**When** the user changes the template
**Then** the sent schedule and the schedule carrying traces are kept, the third is removed, and one schedule is created for each template line whose interval number, interval unit, interval type and template reference are not already present among the kept schedules.

### EV-AC-008 Changing the template keeps the answered questions

**Given** an event whose question `Question1` already has a recorded answer
**When** the user changes the template
**Then** `Question1` stays linked, every other current question is unlinked, and the questions of the new template are linked.

### EV-AC-009 The note of the template replaces an empty note only when the template has one

**Given** a template with an empty note and an event with the note `Keep this`
**When** the user applies that template
**Then** the note stays `Keep this`. **And when** the template note is set to `Template note` and the template is applied again, the event note becomes `Template note`.

### EV-AC-010 The display time zone falls back correctly

**Given** a template with no default time zone and a user whose time zone is `Europe/Brussels`
**When** the user creates an event with that template and the display time zone is still empty
**Then** the display time zone becomes `Europe/Brussels`. **And** when the user has no time zone either, it becomes coordinated universal time.

### EV-AC-011 Default questions are linked to every new event

**Given** the three shipped default questions Name, Email and Phone
**When** a user creates an event without a template
**Then** the event links exactly those three questions, Name and Email are mandatory and Phone is not.

### EV-AC-012 A default question cannot be deleted or made non-reusable

**When** a user tries to delete a shipped default question
**Then** the deletion is refused with *"You cannot delete a default question."*
**And when** a user tries to save a default question with the reusable flag off, the database refuses it with *"A default question must be reusable."* (`EV-RULE-018`, `EV-RULE-020`).

### EV-AC-013 The type of an answered question cannot change

**Given** a selection question with at least one recorded answer
**When** a user changes its type to Text Input
**Then** the write is refused with *"You cannot change the question type of a question that already has answers!"* (`EV-RULE-019`).

### EV-AC-014 A question that has been answered can only be archived

**Given** a question with at least one recorded answer
**When** a user deletes it
**Then** the deletion is refused with *"You cannot delete a question that has already been answered by attendees. You can archive it instead."*

### EV-AC-015 Duplicating an event

**Given** an event named `Summer Conference` in the stage `Announced`, with a cancelled kanban state, two tickets, three booths, two communications and a website menu tree
**When** the user duplicates it
**Then** the copy is named `Summer Conference (copy)`, sits in the first stage with the kanban state `normal`, holds two tickets, three booths and two communications, the copied booths are `available` with no renter and no order, the copied communications are not marked sent and carry no counter, and a full copy of the website menu tree exists under a new root menu named after the copy.

---

## 2. Dates, time zones and slots

### EV-AC-020 A one-day event is detected in the display time zone

**Given** an event running from `2026-06-15 22:00` to `2026-06-15 23:30` in coordinated universal time, displayed in `Europe/Brussels` (two hours ahead in June)
**Then** the start reads `2026-06-16 00:00` and the end `2026-06-16 01:30` locally, therefore the "one day" flag is **true** because both fall on 16 June locally, even though the stored dates suggest 15 June.

### EV-AC-021 A slot hour must lie inside a day

**When** a user saves a slot with a start hour of `24.5`
**Then** the save is rejected with *"A slot hour must be between 0:00 and 23:59."* (`EV-RULE-011`).

### EV-AC-022 A slot must end after it starts

**When** a user saves a slot from `14.0` to `14.0`
**Then** the save is rejected with *"A slot end hour must be later than its start hour."* followed by the display name of the slot (`EV-RULE-012`).

### EV-AC-023 A slot must lie inside the event range

**Given** an event running from `2026-04-21 06:30` to `2026-08-21 17:45`
**When** a user creates a slot on `2026-04-20` from `9.0` to `12.0`
**Then** the save is rejected with *"A slot cannot be scheduled outside of its event time range."* followed by the event range and the slot name (`EV-RULE-013`).

### EV-AC-024 Shrinking an event outside its slots is refused

**Given** a multi-slot event whose earliest slot starts on `2026-04-21 07:00`
**When** the user moves the event start to `2026-04-22 08:00`
**Then** the save is rejected with *"These events cannot have slots scheduled outside of their time range:"* followed by `- <event name>`.

### EV-AC-025 Slot datetimes are derived from the date, the hours and the display time zone

**Given** the display time zone `Europe/Paris` and a slot on `2026-06-12` from `9.5` to `12.0`
**Then** the stored start is `2026-06-12 07:30` and the stored end `2026-06-12 10:00`, both in coordinated universal time (calculation 16).

### EV-AC-026 A slot with registrations cannot be deleted

**Given** a slot with one registration
**When** the user deletes it
**Then** the deletion is refused with *"The following slots cannot be deleted while they have one or more registrations linked to them:"* followed by the slot display name (`EV-RULE-014`).

### EV-AC-027 Searching attendees by event start date honours the slot

**Given** an event without slots starting `2026-04-21 06:30` with one attendee, and a multi-slot event with a slot starting `2026-04-21 07:00` holding three attendees and a second slot starting `2026-04-21 11:00` holding one attendee
**When** the user searches attendees whose event start date is at or after `2026-04-21 07:00`
**Then** only the four attendees of the two slots are returned; searching at or after `2026-04-21 11:00` returns only the attendee of the second slot.

---

## 3. Seats

### EV-AC-030 The canonical seat computation

**Given** an event capped at 100 seats, without slots and without tickets, holding 60 registrations in `open`, 10 in `done` and 5 in `cancel`
**Then** the reserved seats are 60, the used seats are 10, the taken seats are 70 and the available seats are 30 (calculation 1).

### EV-AC-031 Archiving releases a seat and un-archiving takes it back

**Given** the state of EV-AC-030
**When** one of the 60 open registrations is archived
**Then** the reserved seats become 59, the taken seats 69 and the available seats 31.
**When** it is un-archived and seats remain
**Then** the figures return to 60, 70 and 30.

### EV-AC-032 Un-archiving is refused when the event is full

**Given** an event capped at 5 seats with 5 taken and one archived registration in state `open`
**When** the user un-archives that registration
**Then** the operation is rejected with *"There are not enough seats available for <event name>:"* followed by *"<event name>: missing 1 seat(s)"* (`EV-RULE-030`).

### EV-AC-033 Confirming a draft registration is refused when the event is full

**Given** an event capped at 6 seats with 6 taken and one registration in state `draft`
**When** the user confirms it
**Then** the operation is rejected with the same message and the registration stays `draft`.

### EV-AC-034 Creating an open registration is refused when the event is full

**Given** an event capped at 6 seats with 6 taken
**When** a user creates a registration with the default state
**Then** the creation is rejected with the sold-out message and nothing is written.

### EV-AC-035 Lowering the maximum below the seats taken is allowed with a warning

**Given** an event capped at 100 with 70 taken
**When** the user types a new maximum of 50 in the form
**Then** a non-blocking warning appears, titled *"Update the limit of registrations?"* with the body *"There are more registrations than this limit, the event will be sold out and the extra registrations will remain."*, the save succeeds, the available seats become `50 − 70 = −20`, and the event reports itself sold out (`EV-RULE-006`, `EV-RULE-008`).

### EV-AC-036 Lowering the event maximum does not change the ticket counters

**Given** an event with two tickets, each holding registrations
**When** the user lowers the event maximum
**Then** the reserved-seat figure of each ticket is unchanged; only the event figures move.

### EV-AC-037 Seats per slot and per ticket

**Given** a multi-slot event capped at 5 seats per slot, with the tickets `Classic` (no cap), `Better` (cap 3) and `VIP` (cap 1), a first slot holding 3 `Classic` registrations and a second slot holding 1 `Better` registration
**When** the availabilities of the six combinations are requested in the order first-slot Classic, first-slot Better, first-slot VIP, second-slot Classic, second-slot Better, second-slot VIP
**Then** the answer is `[2, 2, 1, 4, 2, 1]` (calculation 2).

### EV-AC-038 Creating registrations inside the availability

**Given** the state of EV-AC-037
**When** two registrations are created on the first slot with the ticket `Better`
**Then** they are created successfully.

### EV-AC-039 Creating registrations beyond the slot availability

**Given** the state of EV-AC-037
**When** five registrations are created on the first slot with the ticket `Classic`
**Then** the creation is rejected with the sold-out message naming the missing seats.

### EV-AC-040 Creating registrations beyond the ticket availability

**Given** the state of EV-AC-037
**When** two registrations are created on the second slot with the ticket `VIP`
**Then** the creation is rejected, because `VIP` allows only one seat per slot.

### EV-AC-041 An archived registration does not consume a slot-and-ticket seat

**Given** the state of EV-AC-037
**When** two `VIP` registrations are created on the second slot, one of them archived
**Then** the creation succeeds; **and when** the archived one is later set to active, the write is rejected with the sold-out message.

### EV-AC-042 An event without a seat limit reports zero available seats

**Given** an event whose limited-seats flag is off
**Then** the available seats are zero and the event is **not** sold out, because the limit flag is tested before the figure.

---

## 4. Registrations opening and selling

### EV-AC-050 An event with no ticket and free seats is open

**Given** an event that is not cancelled, ends in the future, has no seat limit and no ticket
**Then** registrations are open.

### EV-AC-051 A cancelled event closes registrations

**Given** the state of EV-AC-050
**When** the kanban state is set to `cancel`
**Then** registrations are closed, whatever the dates and the seats (`EV-RULE-007`).

### EV-AC-052 A finished event closes registrations

**Given** an event whose end date has passed in the display time zone
**Then** registrations are closed.

### EV-AC-053 Sales windows drive the opening

**Given** an event with a single ticket whose sale starts on `2026-03-01 09:00` in the display time zone
**Then** before that moment registrations are not started and therefore not open; at that exact moment they are open; after the end of sale they are closed again.

### EV-AC-054 One ticket without a start date opens the whole event

**Given** an event with one ticket starting to sell in the future and one ticket with no start date, both non-expired
**Then** the earliest start-of-sale figure of the event is empty and the event counts as already selling.

### EV-AC-055 An event with free seats but every ticket sold out is sold out

**Given** an event capped at 100 with 70 taken, and one ticket capped at 50 with 55 taken
**Then** the event has 30 available seats, the ticket is sold out, registrations are **closed** and the event reports itself **sold out** (calculation 3).

### EV-AC-056 A multi-slot event is open when at least one combination has room

**Given** a multi-slot event with three slots and one ticket, where two slots are full and one has room
**Then** registrations are open.
**When** the last slot fills up
**Then** registrations are closed and the event reports itself sold out.

### EV-AC-057 A multi-slot event with no slot is closed

**Given** a multi-slot event with tickets but zero slots
**Then** registrations are closed.

### EV-AC-058 Ticket sales-window coherency

**When** a user saves a ticket whose start of sale is after its end of sale
**Then** the save is rejected with *"The stop date cannot be earlier than the start date. Please check ticket <ticket name>"* (`EV-RULE-015`).

### EV-AC-059 Per-order limit bounds

**Given** a ticket capped at 4 seats
**When** the user sets a per-order limit of 5
**Then** the save is rejected with *"The limit per order cannot be greater than the maximum seats number. Please check ticket <ticket name>"*.
**When** the user sets a per-order limit of 31 on an uncapped ticket
**Then** the save is rejected with *"The limit per order cannot be greater than 30. Please check ticket <ticket name>"*.
**When** the user sets a per-order limit of −1
**Then** the save is rejected with *"The limit per order must be positive. Please check ticket <ticket name>"* (`EV-RULE-016`).

### EV-AC-060 A ticket with registrations cannot be deleted

**Given** a ticket with one registration
**When** the user deletes it
**Then** the deletion is refused with *"The following tickets cannot be deleted while they have one or more registrations linked to them:"* followed by the ticket name (`EV-RULE-017`).

### EV-AC-061 Display names carrying availability

**Given** an event named `Summer Conference` capped at 100 with 70 taken
**When** the caller asks for names with availability
**Then** the event reads `Summer Conference (30 seats remaining)`, and once 100 seats are taken it reads `Summer Conference (Sold out)`.
**And** a ticket of a **multi-slot** event always reads its plain name, whatever its seats (calculation 18).

---

## 5. Attendees

### EV-AC-070 Contact details are filled from the contact only while empty

**Given** a contact named `Alice Smith` with the address `alice@example.com` and the telephone `+32 456 11 22 33`
**When** a registration is created with that contact and no other value
**Then** the attendee name, electronic mail address and telephone are copied from the contact.
**When** a registration is created with that contact and the name `Bob`
**Then** the name stays `Bob` and only the other two are copied (`EV-RULE-028`).

### EV-AC-071 The contact address of a company is used

**Given** a company contact whose designated contact address is a child record named `Reception`
**When** a registration is created with the company as contact
**Then** the attendee values are copied from `Reception`, not from the company record.

### EV-AC-072 Telephone formatting falls back through three countries

**Given** a contact with no country, an event whose venue is in Belgium, and the telephone `0456112233`
**When** the registration is created
**Then** the telephone is formatted for Belgium. **And** when neither the contact nor the event has a country, the country of the active company is used; when formatting fails, the raw value is kept (`EV-RULE-029`).

### EV-AC-073 A barcode is generated and is unique

**When** a registration is created without a barcode
**Then** a barcode is generated as the decimal text of a pseudo-random eight-byte number.
**When** a second registration is saved with the same barcode
**Then** the save is rejected with *"Barcode should be unique"* (`EV-RULE-025`).

### EV-AC-074 Duplicating a registration does not copy the barcode

**When** a registration is duplicated
**Then** the copy receives a fresh barcode, keeps the attendee values and the answers, and loses the sales and counter links.

### EV-AC-075 The slot is mandatory on a multi-slot event

**When** a registration is saved on a multi-slot event without a slot
**Then** the save is rejected with *"Slot choice is mandatory on multi-slots events."*
**When** it is saved with a slot of another event
**Then** the save is rejected with *"Invalid event / slot choice"* (`EV-RULE-021`).

### EV-AC-076 A ticket of another event is refused

**When** a registration is saved with a ticket belonging to another event
**Then** the save is rejected with *"Invalid event / ticket choice"* (`EV-RULE-022`).

### EV-AC-077 An answer must carry a value

**When** an answer is saved with neither a chosen suggestion nor a typed text
**Then** the database refuses it with *"There must be a suggested value or a text value."* (`EV-RULE-023`).

### EV-AC-078 Attendance stamps the date and logs a note

**Given** a registration in state `open` and no attendance date
**When** it is set to `done` on `2026-06-15`
**Then** the attendance date becomes the current moment and the note *"Attended on 06/15/2026"* is logged in its thread, formatted in the short date format of the reader.
**When** it is set back to `open`
**Then** the attendance date is **not** cleared automatically, because it is only cleared while it is empty.

### EV-AC-079 Cancelling releases the seat

**Given** an event capped at 10 with 10 taken
**When** one registration is cancelled
**Then** the available seats become 1 and a new registration can be created.

---

## 6. The registration desk

### EV-AC-085 An unknown barcode

**When** the desk scans a barcode that no registration carries
**Then** the outcome is `invalid_ticket`, the answer carries that word alone and no registration summary, and nothing is written.

### EV-AC-086 A cancelled attendee

**When** the desk scans the badge of a cancelled registration
**Then** the outcome is `canceled_registration` and the state stays `cancel`.

### EV-AC-087 An unconfirmed attendee

**When** the desk scans the badge of a registration in state `draft`
**Then** the outcome is `unconfirmed_registration` and the state stays `draft`.

### EV-AC-088 A finished event

**Given** an event whose end date has passed
**When** the desk scans the badge of one of its attendees in state `open`
**Then** the outcome is `not_ongoing_event` and the state stays `open`.

### EV-AC-089 A badge of another event

**Given** the desk opened for event A
**When** it scans the badge of an attendee of event B, in state `open`, event B not finished
**Then** the outcome is `need_manual_confirmation` and nothing is written.

### EV-AC-090 A successful check-in

**Given** the desk opened for event A and an attendee of event A in state `open`, event A not finished
**When** the badge is scanned
**Then** the outcome is `confirmed_registration`, the state becomes `done`, the attendance date is stamped and the attendance note is logged.

### EV-AC-091 A second scan

**When** the same badge is scanned again
**Then** the outcome is `already_registered` and nothing changes.

### EV-AC-092 An attended badge scanned at the desk of another event

**Given** the desk opened for event A, and an attendee of event B already in state `done`, event B not finished
**When** the badge of that attendee is scanned
**Then** the outcome is `already_registered`, not `need_manual_confirmation`, and nothing is written, because the attendance test is evaluated before the event test.

### EV-AC-093 The summary returned by a scan

**When** any scan other than `invalid_ticket` returns
**Then** the answer carries the registration identifier, the attendee name, the contact, the slot display name, the ticket name, the event identifier and display name, the display texts of the selection answers, the company name, the badge layout, the attendance date in short format and whether it falls on the current day in the display time zone; with the product bridge it also carries the sale status, its label and the "has to pay" flag.

---

## 7. Automatic communications

### EV-AC-100 The schedule date of a "two days before" communication

**Given** an event starting `2026-06-15 09:00:00`
**When** a schedule is created with interval 2, unit `days`, trigger `before_event`
**Then** its due date is `2026-06-13 09:00:00` (calculation 5).

### EV-AC-101 A "before the event" message is not sent after the event

**Given** the schedule of EV-AC-100 and an event that ended on `2026-06-15 18:00`
**When** the job runs on `2026-06-16 03:00`
**Then** nothing is sent, the schedule is not marked sent, and no error is reported (`EV-RULE-035`).

### EV-AC-102 An "after the event ended" message is still sent late

**Given** a schedule with interval 1, unit `days`, trigger `after_event` on an event that ended on `2026-06-15 18:00`
**When** the job runs on `2026-06-18 03:00`
**Then** the message is sent to every attendee not `draft` and not `cancel`.

### EV-AC-103 A per-attendee message due immediately

**Given** the shipped schedule with interval 0, unit `now`, trigger `after_sub`
**When** an attendee is created at `2026-05-02 14:22:37.812` while the asynchronous parameter is off
**Then** a trace is created with the due date `2026-05-02 14:22:37`, the message is sent immediately, and the trace is marked sent.

### EV-AC-104 A per-attendee message due later

**Given** a schedule with interval 1, unit `hours`, trigger `after_sub`
**When** an attendee is created at `2026-05-02 14:22:37`
**Then** a trace is created with the due date `2026-05-02 15:22:37` and no message is sent until a job run at or after that moment.

### EV-AC-105 The asynchronous parameter defers the work

**Given** the configuration parameter `event.event_mail_async` set
**When** an attendee is created
**Then** no message is sent during the creation; only the communication job and the outgoing-message job are woken (`EV-RULE-040`).

### EV-AC-106 Traces of attendees that fell back are deleted

**Given** a per-attendee trace that is not sent yet and whose registration has been cancelled
**When** the job processes that batch
**Then** the trace is deleted and no message is sent (`EV-RULE-038`).

### EV-AC-107 Slot-based schedules fire once per slot

**Given** a multi-slot event with two slots, on `2026-06-12` 10:00–12:00 and 14:00–16:00 in `Europe/Paris`, and a schedule of 1 hour `before_event`
**When** the job runs after `2026-06-12 07:00` in coordinated universal time
**Then** a trace exists per slot with the due dates `07:00` and `11:00`, only the first one is executed at that moment, and only the attendees of the first slot are contacted (`EV-RULE-039`).

### EV-AC-108 Progress and completion of a global communication

**Given** an event with 130 attendees in state `open`, a batch size of 50 and a render limit of 1000
**When** the job runs the "before the event" schedule
**Then** it processes the attendees in batches of 50, 50 and 30, sets the sent counter to 130, and marks the schedule sent because `130 >= 130` (calculation 6).

### EV-AC-109 A global run that hits the render limit resumes

**Given** the same event with a render limit of 100
**When** the job runs
**Then** 100 attendees are processed, the counter is 100, the schedule is **not** marked sent, the job is woken again, and the next run processes the remaining 30 and marks it sent.

### EV-AC-110 A global communication on an event with no attendee

**Given** a "before the event" schedule and an event with no attendee at all
**When** the job runs
**Then** nothing is sent and the schedule is immediately marked sent.

### EV-AC-111 Archived and cancelled events are skipped

**Given** a due schedule on an archived event, and a due schedule on an event whose kanban state is `cancel`
**When** the job runs
**Then** neither is executed; the readable status of the second is `Cancelled` (`EV-RULE-034`).

### EV-AC-112 An attendee-based schedule stops when the event has ended

**Given** an attendee-based schedule on an event that has ended
**When** the job runs
**Then** the schedule is not selected.

### EV-AC-113 A missing template is skipped, not reported

**Given** a schedule whose referenced template has been deleted
**When** the job runs
**Then** the schedule is dropped from the run, a warning is written to the technical log, the schedule is neither marked sent nor marked in error, and no message is posted on the event (`EV-RULE-032`).

### EV-AC-114 A failing template is reported once per hour

**Given** a schedule whose template raises while rendering
**When** the job runs
**Then** a message is posted on the event addressed to the organiser, the event responsible and the last author of the template, whose body reads *"Communication for <event name> scheduled on <due date> failed."* then *"This is due to an error in template <template link>."* then the rendering error; the error stamp is set and the readable status becomes `Error`.
**When** the job runs again within the hour with the same failure
**Then** no second message is posted.
**When** a later run succeeds
**Then** the error stamp is cleared (`EV-RULE-037`).

### EV-AC-115 Deleting a template deletes its schedules

**Given** an Email Template used by two event schedules and one template schedule line
**When** the template is deleted
**Then** the two event schedules and the template line are deleted as well (`EV-RULE-033`).

### EV-AC-116 The exclusion list

**Given** an attendee whose address is on the mass-mailing exclusion list
**When** an attendee-based schedule runs
**Then** the message is sent anyway.
**When** a global event-based schedule runs
**Then** the message is not sent to that attendee (`EV-RULE-036`).

### EV-AC-117 A text message schedule

**Given** the text message package installed and a schedule whose reference points at a Text Message Template
**Then** its notification type reads `sms`, and running it schedules a mass text message on the attendees rather than an electronic mail message.

---

## 8. Selling tickets on a sales order

### EV-AC-125 The configurator refuses an incoherent choice

**When** a salesperson picks an event and a ticket belonging to another event
**Then** the wizard is refused with *"Invalid ticket choice "<ticket name>" for event "<event name>"."* (`EV-RULE-050`).

### EV-AC-126 A line selling a ticket product must be configured

**Given** an order line with a ticket product but no event
**When** the salesperson confirms the order
**Then** the confirmation is refused with *"Please make sure all your event related lines are configured before confirming this order:"* followed by the line description (`EV-RULE-053`).

### EV-AC-127 A single paid order confirms into draft registrations

**Given** a draft order with one ticket line of quantity 1 and a unit price of `10.00`
**When** the salesperson confirms it
**Then** one registration is created in state `draft`, the attendee editor opens, and the seats are not yet consumed (`EV-RULE-055`).

### EV-AC-128 Validating the attendee editor confirms the seats

**Given** the state of EV-AC-127
**When** the salesperson validates the editor
**Then** exactly one registration exists, its state is `open`, its sale status is `sold` and the seat is consumed.

### EV-AC-129 Free seats are confirmed immediately

**Given** an event capped at 5 seats and an order line of 3 free tickets
**When** the attendee editor is validated
**Then** three registrations exist and all three are in state `open`.

### EV-AC-130 Free seats are still refused when the event is full

**Given** an event capped at 2 seats and an order carrying two free ticket lines of 2 and 1 seats
**When** the salesperson confirms the order
**Then** the confirmation is rejected with the sold-out message; **and when** the attendee editor is validated instead, it is rejected the same way.

### EV-AC-131 Free seats are refused when the ticket is full

**Given** an uncapped event with a ticket capped at 2 seats and an order for 3 free seats of that ticket
**When** the order is confirmed
**Then** the operation is rejected with the sold-out message naming the ticket.

### EV-AC-132 Confirming twice does not duplicate the attendees

**Given** a confirmed order line of 3 seats with 3 registrations
**When** the confirmation logic runs again
**Then** no new registration is created, because the count of non-cancelled registrations already equals the ordered quantity (`EV-RULE-054`).

### EV-AC-133 Cancelling a seat and re-confirming creates a replacement

**Given** a confirmed order line of 3 seats where one registration has been cancelled
**When** the confirmation logic runs again
**Then** one new registration is created, giving two active seats plus one cancelled.

### EV-AC-134 Cancelling the order cancels the seats

**Given** a confirmed order with one registration
**When** the order is cancelled
**Then** the registration state becomes `cancel` and the registration is kept.

### EV-AC-135 Resetting to draft and confirming again creates new seats

**Given** the state of EV-AC-134
**When** the order is set back to draft and confirmed again
**Then** two registrations exist, the first `cancel` and the second `draft`.

### EV-AC-136 Deleting the order deletes the seats

**Given** a confirmed order with one registration
**When** the order is deleted
**Then** the registration is deleted as well; the same happens when only the order line is deleted (`EV-RULE-031`).

### EV-AC-137 Reducing the quantity leaves the surplus to be cancelled by hand

**Given** a confirmed order line of 3 seats with 3 registrations
**When** the salesperson cancels one registration
**Then** two remain active, the order attendee counter shows 2, and the three registrations read `draft`, `draft`, `cancel` in creation order.

### EV-AC-138 Changing the customer rewrites the attendees

**Given** a confirmed order with three registrations and the customer `Alpha`
**When** the customer is changed to `Beta`
**Then** the contact of all three registrations becomes `Beta` (`EV-RULE-056`).

### EV-AC-139 Changing the ticket of a sold seat raises an activity

**Given** a registration attached to an order, holding the ticket `Standard`
**When** its ticket is changed to `Premium`
**Then** a warning activity is scheduled on the order, assigned to the event responsible, describing the move from `Standard` to `Premium` and naming the changed record as `Ticket` (`EV-RULE-057`).

### EV-AC-140 The sale status follows the order

**Given** a registration attached to a draft order whose total is `10.00`
**Then** its sale status is `to_pay` and its state is `draft`.
**When** the line total is set to zero while the order total stays non-zero
**Then** the sale status stays `to_pay`.
**When** the order total becomes zero
**Then** the sale status becomes `free` and the state becomes `open`.
**When** the order and line links are removed
**Then** the sale status stays `free`.
**When** the order total becomes `0.01`
**Then** the sale status becomes `to_pay`.
**When** the order is confirmed
**Then** the sale status becomes `sold` and the state becomes `open`.

### EV-AC-141 A registration without any order is free

**When** a registration is created with no sales order and no counter order
**Then** its sale status is `free` and its state is `open`.

### EV-AC-142 The price of a ticket line comes from the ticket

**Given** a ticket priced `1,000.00` whose product has a sales price of `10.00`, and a pricelist that fixes the product at `6.00` and a tax of 10 percent
**When** a line is created with that product, event and ticket
**Then** the order total is `660.00`, that is the pricelist price `6.00` scaled to the ticket, giving `600.00` plus 10 percent (`EV-RULE-052`).

### EV-AC-143 Currency conversion of a ticket line

**Given** a company whose currency is `USD` (United States dollar), a ticket priced `1,000.00`, and a pricelist in `VEF` (Venezuelan bolívar) with the rate `5.0`
**When** the order uses the `USD` pricelist
**Then** the order total is `1,000.00`.
**When** the order switches to the `VEF` pricelist and the prices are refreshed
**Then** the order total is `5,000.00`.

### EV-AC-144 The total sales figure of an event

**Given** two confirmed orders selling tickets of one event, for `1,200.00` and `3,000.00` including tax, in the company currency
**Then** the total sales of the event is `4,200.00`; with a second currency involved, each currency group is converted at today's rate before the sum (calculation 14).

---

## 9. The public website

### EV-AC-150 Parsing the attendee form

**Given** an event with the identification questions Name, Email and Phone, a second Phone question, a Company question, one per-attendee selection question and two order-level questions, and the posted form:

```text
1-name-<name question>          = Pixis
1-email-<email question>        = pixis@gmail.com
1-phone-<phone question>        = +32444444444
1-phone-<second phone question> = +32555555555
1-event_ticket_id               = <ticket 1>
2-name-<name question>          = Geluchat
2-email-<email question>        = geluchat@gmail.com
2-phone-<phone question>        = +32777777777
2-company_name-<company question> = My Company
2-event_ticket_id               = <ticket 2>
1-simple_choice-<question 1>    = 5
2-simple_choice-<question 1>    = 9
0-simple_choice-<question 2>    = 7
0-text_box-<question 3>         = Free Text
custom-field                    = custom-value
```

**Then** two attendee value sets are produced. The first carries the name `Pixis`, the address `pixis@gmail.com`, the telephone `+32444444444`, the first ticket, and seven answers: the four identification answers including the **second** telephone stored as an answer only, the selection answer `5`, and the two order-level answers `7` and `Free Text`. The second carries `Geluchat`, `geluchat@gmail.com`, `+32777777777`, the company name `My Company`, the second ticket, and the matching seven answers. The posted `custom-field` is ignored (`EV-RULE-026`).

### EV-AC-151 Only the first answer of an identification type writes the field

**Given** the form of EV-AC-150
**Then** the telephone field of the first attendee is `+32444444444`, the answer to the second telephone question is stored but does not overwrite the field.

### EV-AC-152 Registering on an event with no ticket

**Given** an event with no ticket
**When** a visitor submits the attendee form with a ticket identifier of zero
**Then** the registrations are created without a ticket.

### EV-AC-153 A ticket outside its sales window is refused

**Given** an event whose only ticket ended its sale yesterday
**When** a visitor posts that ticket
**Then** the request is refused with *"This ticket is not available for sale for this event"* (`EV-RULE-027`).

### EV-AC-154 Insufficient seats redirect back with a code

**Given** an event capped at 2 free seats
**When** a visitor submits three attendees
**Then** no registration is created and the visitor is redirected to `/event/<identifier>/register?registration_error_code=insufficient_seats`.

### EV-AC-155 A failed anti-robot check redirects back with a code

**When** the anti-robot verification fails
**Then** the visitor is redirected to `/event/<identifier>/register?registration_error_code=recaptcha_failed` and nothing is created.

### EV-AC-156 The success page requires the right visitor

**Given** registrations created by visitor A
**When** visitor B opens the success page with those identifiers
**Then** nothing is shown, because the registrations are re-read restricted to the current visitor and the event.

### EV-AC-157 Campaign attribution is captured

**Given** a visiting session carrying a campaign, a source and a medium
**When** the visitor registers
**Then** the created registrations carry those three values.

### EV-AC-158 Website visibility for an anonymous reader

**Given** an unpublished event
**When** an anonymous visitor opens the event list
**Then** the event does not appear, and opening its address is refused. **And** a published event with the visibility `public` appears for everyone.

### EV-AC-159 Website visibility `logged_users`

**Given** a published event with the visibility `logged_users`
**Then** an anonymous visitor does not see it in the list, a signed-in user does, and both can open it through its direct link.

### EV-AC-160 Website visibility `link`

**Given** a published event with the visibility `link`
**Then** neither an anonymous visitor nor an ordinary signed-in user sees it in the list, and both can open it through its direct link.

### EV-AC-161 A participant always sees the event

**Given** a published event with the visibility `link` and a visitor holding a registration in state `open` on it
**Then** that visitor sees the event in the list (calculation 12).

### EV-AC-162 A draft registration does not make a participant

**Given** a visitor whose only registration on an event is in state `draft`
**Then** that visitor is not a participant of the event.

### EV-AC-163 Event users see unpublished events on the site

**Given** an unpublished event
**Then** an Event User and an Event Administrator can open its public page; a portal user and an anonymous visitor cannot.

### EV-AC-164 The event address redirects

**Given** an event with a website menu whose first child points at `/event/<slug>/page/home`
**When** a visitor opens `/event/<slug>`
**Then** the visitor is redirected to that first child address. **And** when the event has no menu, the visitor is redirected to `/event/<identifier>/register`.

### EV-AC-165 Tag filtering on the event list

**Given** the tags `age 10-12` and `age 12-15` in the category `Age`, and `football` in the category `Activity`
**When** a visitor filters on `age 10-12` and `football`
**Then** only events tagged with both are returned.
**When** `age 12-15` is added
**Then** events tagged `football` and tagged with either age are returned.

### EV-AC-166 Multiple tags in a plain page request are redirected

**When** a plain page request carries more than one tag
**Then** the answer is a permanent redirect to `/event`.

### EV-AC-167 The venue is searchable

**Given** an event whose venue is in the city `Brussels`
**When** a visitor searches `Brussels` in the event list
**Then** the event is returned, because the search also matches the venue name, street, second street line, city, postal code, subdivision and country.

### EV-AC-168 Publishing notifies the followers

**When** an event is published
**Then** a message is posted on it under the subtype "Event published"; unpublishing posts under "Event unpublished".

### EV-AC-169 Per-event menus are created and removed

**Given** an event with the website menu switched off
**When** the user switches it on and saves
**Then** a root menu named after the event is created with the entries Home (a page), Practical and, depending on the installed packages, Talks, Agenda, Propose a talk, Exhibitors list, Rooms and Become exhibitor.
**When** the user switches the website menu off
**Then** the root menu and all of its children are deleted.

### EV-AC-170 Deleting a menu entry switches the flag off

**Given** an event whose booth menu is on
**When** the website editor deletes the "Become exhibitor" entry
**Then** the booth menu flag of the event becomes false.

### EV-AC-171 Each per-event page has its own view and its own search metadata

**Given** two events, each with a Home page
**Then** each page carries its own view with a unique key, and editing one does not change the other; the search metadata of a page is that of its own menu entry.

---

## 10. The online shop

### EV-AC-180 Only free tickets and no cart: no order is created

**Given** an event with two free tickets and a visitor with no cart
**When** the visitor registers for both
**Then** two registrations are created in state `open`, and no sales order exists.

### EV-AC-181 A mix of free and paid tickets creates an order

**Given** an event with one free ticket and one paid ticket
**When** a visitor selects one of each
**Then** a cart is created with one line per ticket, both registrations carry the order and the line, and the visitor is sent to the checkout.

### EV-AC-182 An order of only free tickets is confirmed without checkout

**Given** a cart in which every line carries a ticket whose price is zero
**When** the visitor confirms the registration
**Then** the order is confirmed at once, the cart is reset and the visitor is sent to the order confirmation page (`EV-RULE-066`).

### EV-AC-183 The cart cap when a ticket is sold out

**Given** a ticket capped at 2 seats with 2 taken
**When** a visitor adds 1 seat to the cart
**Then** the quantity stays at its previous value and the message is *"Sorry, The <ticket name> tickets for the <event name> event are sold out."* (`EV-RULE-060`).

### EV-AC-184 The cart cap when fewer seats remain than requested

**Given** a ticket with 3 seats remaining and a cart line already holding 1 seat
**When** the visitor asks for 6 seats
**Then** the quantity becomes `1 + 3 = 4` and the message is *"Sorry, only 3 seats are still available for the <ticket name> ticket for the <event name> event."* With a slot, the message ends with ` on <slot name>`.

### EV-AC-185 The quantity of a ticket line cannot be raised by hand

**When** a visitor raises the quantity of a ticket line from the cart page without passing a ticket
**Then** the quantity is kept and the message is *"You cannot raise manually the event ticket quantity in your cart"* (`EV-RULE-061`).

### EV-AC-186 Lowering the quantity cancels the surplus seats

**Given** a cart line of 5 seats with 5 registrations
**When** the visitor sets the quantity to 2
**Then** the 2 oldest registrations of that order, slot and ticket are kept and the next 3 are cancelled (`EV-RULE-067`).

### EV-AC-187 One line per slot and ticket pair

**Given** a cart already holding 2 seats of ticket `Standard` on slot A
**When** the visitor adds 1 seat of `Standard` on slot B
**Then** a **second** line is created, because the slot differs (`EV-RULE-062`).

### EV-AC-188 Seats are verified again at payment

**Given** an order holding 3 seats of a ticket whose remaining seats have fallen to 1 since the cart was filled
**When** the payment is attempted
**Then** the transaction is refused with the sold-out message (`EV-RULE-063`).

### EV-AC-189 Buying the last ticket succeeds

**Given** a ticket with exactly 1 seat remaining
**When** a visitor buys 1 seat and pays
**Then** the order is confirmed, the registration moves to `open` with the sale status `sold`, and the ticket becomes sold out.

### EV-AC-190 Abandoned-cart reminders skip unsellable tickets

**Given** an abandoned cart whose ticket is now sold out
**When** the abandoned-cart reminder runs
**Then** that cart is skipped (`EV-RULE-064`).

### EV-AC-191 A ticket-only order skips the full address

**Given** an order whose lines all carry tickets and the configuration parameter left at its default
**When** the visitor reaches the checkout
**Then** the full billing address is not requested (`EV-RULE-065`).

### EV-AC-192 A gift voucher covering the whole amount

**Given** an order of paid tickets whose amount is fully covered by a voucher
**Then** the order is **not** confirmed automatically: it still goes through the ordinary confirmation, because the automatic confirmation applies only when every line carries a ticket priced zero.

### EV-AC-193 The cart shows the ticket name

**Given** a cart line carrying a ticket
**Then** the short label of that line is the ticket display name, the strike-through original price is hidden, and the line cannot be re-ordered from the order history.

---

## 11. The shop counter

### EV-AC-200 Selling a ticket at the counter

**Given** a counter session with the event data loaded
**When** the operator sells one ticket and collects the attendee details
**Then** one registration is created carrying the counter order line, the event, the ticket and (when applicable) the slot, its contact is the customer of the counter order when the registration has none, and its sale status follows the counter order state.

### EV-AC-201 Selling a ticket of a multi-slot event

**Given** a multi-slot event at the counter
**When** the operator sells one seat
**Then** the counter asks for the slot and the created registration carries it.

### EV-AC-202 Attendee questions are optional at the counter

**Given** an event whose questions are not mandatory
**When** the operator skips them
**Then** the sale completes and the registrations carry no answer.

### EV-AC-203 The line price stays at the ticket price

**Given** a ticket priced `30.00`
**When** the operator adds it to a counter order
**Then** the line price is `30.00`, even after the customer is set.

### EV-AC-204 Paying the counter order sends the badges

**Given** a counter order carrying two attendees with electronic mail addresses
**When** the order is paid
**Then** a badge message is sent to each of them.

### EV-AC-205 Refunding cancels the right number of seats

**Given** a counter line that sold 4 seats and no cancelled registration
**When** 2 seats are refunded
**Then** exactly 2 registrations of that line are cancelled.
**When** 1 further seat is refunded
**Then** exactly 1 more is cancelled, leaving 1 active.

### EV-AC-206 Remaining seats are broadcast live

**Given** two open counter sessions
**When** a registration is created anywhere, including on the public website
**Then** both sessions receive the remaining seats of the event, of each of its tickets and of each of its slots.

### EV-AC-207 The counter and sales bridge refines the sale status

**Given** the counter-and-sales bridge installed
**Then** a counter order in state `paid`, `done` or `invoiced` gives the sale status `sold` and the state `open`; any other counter state gives `to_pay` and `draft`.

---

## 12. Booths and exhibitors

### EV-AC-215 Booth contact details are filled from the renter

**Given** a contact named `Gamma` with an address and a telephone
**When** a booth is confirmed with that renter and no other value
**Then** the renter name, electronic mail address and telephone are copied from the contact, and a value already typed is kept.

### EV-AC-216 Confirming a booth posts one message

**Given** an available booth
**When** it is confirmed
**Then** its state becomes `unavailable` and one message is posted on the **event** under the subtype "Booth Booked".
**When** it is written with the state `unavailable` again
**Then** no second message is posted.

### EV-AC-217 The booth price on an order line

**Given** two Premium booths at `500.00` each and one Standard booth at `100.00`, reserved on one line
**Then** the line carries quantity 1 and the price `1,100.00`, and its description reads `<event display name> : ` followed by the three booth names.

### EV-AC-218 Confirming an order confirms its booths

**Given** an order line with three pending booths, all still available
**When** the order is confirmed
**Then** the three booths become `unavailable`, carry the contact values of the reservation, and the booking messages are posted.

### EV-AC-219 A booth that became unavailable blocks the confirmation

**Given** an order line reserving a booth that has since been booked by someone else
**When** the order is confirmed
**Then** the confirmation is refused with *"The following booths are unavailable, please remove them to continue : "* followed by the booth name (`EV-RULE-074`).

### EV-AC-220 The losing reservations are cancelled

**Given** booth `B12` claimed by order A and by order B
**When** order A is confirmed
**Then** `B12` belongs to order A, the reservation of order B is deleted, order B is cancelled, and a message is posted on order B for its salesperson reading *"Your order has been cancelled because the following booths have been reserved"* followed by `B12` (`EV-RULE-075`).

### EV-AC-221 A booth linked to an order cannot be deleted

**When** a user deletes a booth that carries a sales order
**Then** the deletion is refused with *"You can't delete the following booths as they are linked to sales orders: <booth name>"* (`EV-RULE-072`).

### EV-AC-222 Paying the invoice stamps the booths

**Given** a confirmed order with a booth line, invoiced
**When** the invoice is paid
**Then** the confirmed booths of that line carry `is_paid = true`; no journal entry is written by this domain (`EV-RULE-078`).

### EV-AC-223 Booking a sponsoring booth creates a sponsor

**Given** the shipped Premium Booth category, which creates a Silver exhibitor
**When** a renter books one Premium booth of an event
**Then** one sponsor is created for that renter, that event, the Silver level and the exhibitor kind, and it is linked to the booth.
**When** the same renter books a second Premium booth of the same event
**Then** the **same** sponsor is reused (`EV-RULE-079`).

### EV-AC-224 A different level creates a different sponsor

**Given** the renter of EV-AC-223
**When** the same renter books a Very Important Person booth, which creates a Gold online exhibitor
**Then** a second sponsor is created, because the level and the kind differ.

### EV-AC-225 The public booth form refuses a taken booth

**Given** a booth that was booked while the visitor was filling the form
**When** the visitor submits
**Then** the answer is the code `boothError` and nothing is booked (`EV-RULE-076`).

### EV-AC-226 The public booth form refuses an anonymous known address

**Given** an anonymous visitor whose posted address already belongs to a contact
**When** the visitor submits
**Then** the answer is the code `existingPartnerError`.

### EV-AC-227 Online booth sales go through the cart

**Given** online booth sales installed and a category priced `500.00`
**When** a visitor books a booth
**Then** the booth is **not** confirmed; a cart line is created carrying the booth as a pending reservation, and the visitor is sent to the cart. The booth is confirmed only when the order is confirmed.

### EV-AC-228 A free online booth is confirmed immediately

**Given** online booth sales installed and a category priced zero, with an empty cart
**When** a visitor books a booth
**Then** the order is confirmed at once, the booth becomes unavailable and the success answer is returned.

### EV-AC-229 A booth line always sells one unit

**When** a visitor raises the quantity of a booth line to 2
**Then** the quantity is forced back to 1 with the message *"You cannot manually change the quantity of an Event Booth product."* (`EV-RULE-077`).

### EV-AC-230 Booth prices in another currency

**Given** a booth category priced `500.00` in the company currency and a pricelist in a second currency with the rate `2.0`
**When** a visitor books one booth with that pricelist
**Then** the line price is `1,000.00` in the second currency.

### EV-AC-231 The exhibitor list shows only exhibitors

**Given** an event with one sponsor of kind `sponsor`, one of kind `exhibitor` and one of kind `online`, all published
**Then** the exhibitor list shows the second and the third; the first appears only as a footer logo.

### EV-AC-232 Sponsor opening hours

**Given** an event running `2026-06-12 08:00` to `2026-06-14 20:00` in `Europe/Paris` and a sponsor open from `9.5` to `17.0`
**Then** at `2026-06-12 09:15` local the sponsor is closed, at `10:00` it is open, and on `2026-06-14` with a closing hour of `22.0` it closes at `20:00` with the event (calculation 10).

### EV-AC-233 A sponsor with no hours is always open during the event

**Given** an ongoing event and a sponsor with no opening hours set
**Then** the sponsor is open.
**Given** an event that is not ongoing
**Then** every sponsor is closed, whatever its hours.

---

## 13. The programme

### EV-AC-240 Duration, start and end form a triangle

**Given** a talk starting `2026-06-12 14:00` with a duration of `1.5`
**Then** its end is `15:30`.
**When** the end is set to `16:00`
**Then** the duration becomes `2.0`.
**When** the duration is set to `0.75`
**Then** the end becomes `14:45`.

### EV-AC-241 A talk with no date has no time flags

**Given** a talk with neither a start nor an end
**Then** the live, soon, today, upcoming and done flags are all false and both counters are zero, and the "one day" flag is false.

### EV-AC-242 Speaker fields are filled from the contact only while empty

**Given** a contact named `Ada Lovelace` with a job position and a biography
**When** a talk is created with that contact and the speaker name `A. Lovelace`
**Then** the name stays `A. Lovelace` and the job position and biography are copied.

### EV-AC-243 The operational contact fields are overwritten

**Given** a talk whose operational address is `old@example.com`
**When** a contact whose address is `new@example.com` is set on the talk
**Then** the operational address becomes `new@example.com`.

### EV-AC-244 The speaker tag line

**Given** the speaker name `Ada Lovelace`, the job position `Chief Analyst` and the company `Analytical Engines`
**Then** the tag line reads `Ada Lovelace, Chief Analyst at Analytical Engines`; without a company it reads `Ada Lovelace, Chief Analyst`; without a job position it reads `Ada Lovelace from Analytical Engines`; with neither it reads `Ada Lovelace`; with no name it is empty.

### EV-AC-245 Entering a fully accessible stage publishes the talk

**Given** an unpublished talk in the stage `Announced`
**When** it is moved to `Published`
**Then** it becomes published and its kanban state is reset to `normal`.

### EV-AC-246 Entering a cancelling stage unpublishes the talk

**Given** a published talk
**When** it is moved to `Cancelled`
**Then** it becomes unpublished.

### EV-AC-247 A stage template is sent to the speaker

**Given** the stage `Confirmed`, which carries the confirmation template
**When** a talk is moved into it
**Then** the confirmation message is sent to the speaker as an internal note.

### EV-AC-248 Proposing a talk from the public site

**Given** an event whose proposal page is active
**When** a visitor submits a proposal with a title, an abstract, a speaker block and two known tags
**Then** a talk is created in the first stage, with no responsible user, with the two tags, with the abstract and biography converted to rich text, and a message is posted on the event under the subtype "New Track".

### EV-AC-249 A proposal with an unknown tag drops it silently

**When** a proposal posts an unknown tag identifier and a valid one
**Then** the created talk carries only the valid tag (`EV-RULE-087`).

### EV-AC-250 A proposal with a contact block but no way to reach the speaker

**When** a visitor asks to add contact information and fills neither a contact address nor a contact telephone
**Then** the answer is the code `invalidFormInputs` and nothing is created (`EV-RULE-086`).

### EV-AC-251 The talk list shows announced talks to the organiser only

**Given** a talk in the stage `Announced`, which is visible in the agenda but not published
**Then** an Event User sees it in the public talk list and in the agenda; an anonymous visitor does not (`EV-RULE-089`).

### EV-AC-252 Grouping the talk list

**Given** an event with one running talk, one starting in 20 minutes, two on 12 June, one on 13 June and one with no date
**Then** the list shows a live group, a soon group, a group per day ordered ascending, and a final group `Coming soon`.
**And** a day group whose talks are all finished is collapsed when the event still has a talk that is not finished.

### EV-AC-253 The agenda grid rounds down to quarter hours

**Given** a talk starting `10:07` lasting `1.5` hours in the room `Main Hall`
**Then** it occupies six quarter-hour rows starting at `10:00`, and the displayed times are the real ones, `10:07` to `11:37`.

### EV-AC-254 A talk without a room spans every column

**Given** a talk with no room in a grid with two rooms
**Then** the talk occupies both room columns on every row it spans.

### EV-AC-255 Switching the reminder of an ordinary talk

**Given** an ordinary talk and a visitor who never touched it
**Then** the reminder state is false.
**When** the visitor switches it on
**Then** a link is created with the wish-list flag true and the state becomes true.
**When** the visitor asks for the same state again
**Then** the answer is `ignored`.

### EV-AC-256 Switching the reminder of a key talk

**Given** a key talk and a visitor who never touched it
**Then** the reminder state is true.
**When** the visitor switches it off
**Then** a link is created with the opt-out flag true and the state becomes false.
**When** the visitor switches it on again
**Then** the opt-out flag becomes false and the state returns to true.

### EV-AC-257 The reminder message is refused for a finished talk

**When** a visitor asks for the reminder of a talk that has ended
**Then** the answer is *"The talk is already finished."* and nothing is sent (`EV-RULE-085`).

### EV-AC-258 The reminder message falls back to the event times

**Given** a talk with no date
**When** the reminder is sent
**Then** the calendar entry carries the event dates and the message carries the warning that the times shown are those of the event.

### EV-AC-259 Talk suggestions prefer the live talk

**Given** three published candidates: one started four minutes ago and still running, one starting in twenty minutes, one finished yesterday
**Then** the suggestion order is the running one, then the upcoming one, then the finished one (calculation 17).

### EV-AC-260 A quiz question must have exactly one correct answer

**When** a question is saved with two correct answers
**Then** the save is rejected with *"Question "<question name>" must have 1 correct answer to be valid."*
**When** it is saved with a single answer
**Then** the save is rejected with *"Question "<question name>" must have 1 correct answer and at least 1 incorrect answer to be valid."* (`EV-RULE-082`).

### EV-AC-261 Quiz scoring

**Given** a quiz of three questions where the visitor picks answers worth 10, 5 and 15 points
**Then** the visitor link records `quiz_completed = true` and 30 points, and the answer returns per question the points, the correct answer text, whether the pick was correct and the explanatory comment.

### EV-AC-262 An incomplete quiz is refused

**When** a visitor submits answers covering only two of three questions
**Then** the answer is `quiz_incomplete` and nothing is written (`EV-RULE-083`).

### EV-AC-263 A second submission is refused

**When** a visitor who already completed the quiz submits again
**Then** the answer is `track_quiz_done`.

### EV-AC-264 Resetting a quiz

**Given** a quiz without unlimited tries
**When** an ordinary visitor asks to reset it
**Then** the request is forbidden.
**When** an Event Administrator asks to reset it
**Then** the completion flag becomes false and the points become zero (`EV-RULE-084`).

### EV-AC-265 The leaderboard ordering and tie-break

**Given** five visitors scoring 40, 30, 30, 10 and 5, where the two 30-point visitors have the identifiers 87 and 42
**Then** the ranking is 40, then visitor 42, then visitor 87, then 10, then 5.
**When** a name search matches only visitor 87
**Then** the single returned row keeps the position 3 (calculation 13).

### EV-AC-266 Visitors that wish-listed a talk are never purged

**Given** a visitor with no recent visit but with one talk link
**When** the inactive-visitor housekeeping runs
**Then** the visitor is kept (`EV-RULE-095`).

### EV-AC-267 Merging visitors moves the talk links

**Given** two visitors, one of which has talk links and no contact
**When** they are merged
**Then** the links move to the surviving visitor and receive its contact.

---

## 14. Lead generation

### EV-AC-275 A per-attendee rule at creation

**Given** an active rule with the basis "per attendee" and the trigger "Attendees are created"
**When** three attendees are created on a matching event
**Then** three leads are created, each carrying the event, the rule, the attendee, the lead type, the salesperson, the sales team and the tags of the rule.

### EV-AC-276 A per-order rule groups the attendees

**Given** an active rule with the basis "per order" and the trigger "Attendees are created", and three attendees created in one batch on the same event
**Then** exactly **one** lead is created, carrying the three attendees, and its description lists them as a numbered list under `Participants`.

### EV-AC-277 A per-order rule groups by sales order

**Given** the sales bridge installed, a per-order rule, and two attendees on order A plus one on order B, all on the same event
**Then** two leads are created, one per order.

### EV-AC-278 A per-order rule updates an existing lead

**Given** the state of EV-AC-277 and a further attendee added to order A
**When** the rule runs again
**Then** no new lead is created for order A: the existing one is updated, gains the new attendee and its description gains a paragraph `New registrations` with the new attendee.

### EV-AC-279 Duplicate protection

**Given** a rule with the trigger "Attendees are registered" that already produced a lead for an attendee
**When** the attendee moves from `open` to `draft` and back to `open`
**Then** no second lead is created (`EV-RULE-094`).

### EV-AC-280 A rule filter restricts the attendees

**Given** a rule whose registration filter keeps only attendees whose address contains `@example.com`
**When** two attendees are created, one matching and one not
**Then** only the matching one produces a lead.

### EV-AC-281 A rule restricted to an event or a template

**Given** a rule naming event A and template T
**Then** an attendee of event A matches, an attendee of an event using template T matches, and an attendee of an unrelated event does not.

### EV-AC-282 The contact of a per-attendee lead is dropped on mismatch

**Given** one attendee whose contact is `Alice` with the address `alice@example.com`, while the attendee address is `bob@example.com`
**When** a per-attendee lead is created
**Then** the lead carries no contact and instead carries the attendee name, address and telephone.

### EV-AC-283 The lead name

**Given** an event named `Summer Conference` and an attendee named `Alice Smith`
**Then** the lead name is `Summer Conference - Alice Smith`; with no attendee name the electronic mail address is used instead.

### EV-AC-284 Updating an attendee updates its leads

**Given** an attendee with a per-attendee lead
**When** the attendee telephone changes
**Then** the lead description gains a paragraph `Updated registrations` with the new line of that attendee.

### EV-AC-285 Answers are added to the lead description

**Given** the website bridge installed and an attendee with two answers
**Then** the lead description of that attendee carries a `Questions` block listing the question titles and the answer values.

### EV-AC-286 Regeneration permission

**When** an Event User presses "Generate Leads"
**Then** the operation is refused with *"Only Event Managers are allowed to re-generate all leads."* (`EV-RULE-092`).

### EV-AC-287 Small-volume regeneration is immediate

**Given** an event with 120 attendees that are neither `draft` nor `cancel`
**When** an Event Administrator regenerates the leads
**Then** the rules are applied at once and the notification reads *"Yee-ha, <n> Leads have been created!"*, or *"Aww! No Leads created, check your Lead Generation Rules and try again."* when nothing was created.

### EV-AC-288 Large-volume regeneration is batched

**Given** an event with 500 attendees that are neither `draft` nor `cancel`
**When** an Event Administrator regenerates the leads
**Then** a lead request is created, the batch job is woken, and the notification reads *"Got it! We've noted your request. Your leads will be created soon!"*
**And** the job processes 200 attendees per pass, stores the last processed identifier, wakes itself while work remains, and deletes the request when it is complete.

### EV-AC-289 One request per event

**When** a second regeneration of the same event is requested while the first is still pending
**Then** the database refuses it with *"You can only have one generation request per event at a time."* (`EV-RULE-093`).

### EV-AC-290 Imports do not trigger rules

**When** attendees are created or written during a data import
**Then** no lead rule runs.

---

## 15. Printed documents and links

### EV-AC-295 The badge layout follows the event

**Given** an event whose badge layout is `four_per_sheet`
**When** the badges of five attendees are printed
**Then** the document holds four badges on the first sheet and one on the second, each carrying the event name, the dates in the display time zone, the venue, the attendee name and company, the organiser logo, the quick response code of the barcode and the coloured ticket strip.

### EV-AC-296 The barcode option hides the linear barcode

**Given** the barcode option switched off
**Then** the printed badge and ticket carry only the quick response code, not the linear barcode.

### EV-AC-297 The ticket footer names the organiser

**Given** an event with an organiser carrying a telephone, an address and a website
**Then** the full-page ticket footer repeats that block on every page; without an organiser the footer carries the event name instead.

### EV-AC-298 The ticket link is protected by a hash

**Given** a link built for the attendees `{4, 9}` of event A
**When** the identifier `7` is added to the address
**Then** the request answers "not found", because the hash no longer matches.
**When** the two identifiers are reordered in the address
**Then** the request still succeeds, because the hash is computed on the sorted list.

### EV-AC-299 The document name

**Given** an event named `Summer/Conference` starting `2026-06-15 09:00` and one attendee named `Alice Smith`
**Then** the produced document is named `Tickets - SummerConference (Jun 15, 2026, 11:00:00 AM) - Alice Smith` when the start is read in a display time zone two hours ahead; the slash is removed from the event name.

### EV-AC-300 The calendar file of an event

**When** the calendar file of an event is requested
**Then** it holds one entry whose start and end are the event dates converted into the display time zone, whose summary is the event name, whose description is a link to the event followed by the description shortened to 1900 characters, and whose location is the one-line venue address when a venue exists.

### EV-AC-301 The calendar file of a slot

**When** the calendar file is requested with a slot of that event
**Then** the entry carries the slot start and end instead of the event dates.
**When** it is requested with a slot of another event
**Then** the answer is "not found".

### EV-AC-302 The attendee list document

**When** the attendee list of an event is printed
**Then** it holds the heading `Attendee list`, the event name, the event range in the display time zone, and a table with the columns Name, Company, Ticket type, Phone number and a quick response code, with a page break after the event.

---

## 16. Access control

### EV-AC-310 A Registration Desk user cannot create an event

**When** a Registration Desk user creates an event
**Then** the operation is refused for lack of rights; the same user can read events, create and update attendees, and scan badges.

### EV-AC-311 An Event User cannot delete an event

**When** an Event User deletes an event
**Then** the operation is refused; an Event Administrator may delete it.

### EV-AC-312 Multi-company isolation

**Given** an event belonging to company A
**When** a user whose active companies are only B reads the event list
**Then** the event is not returned; an event with **no** company is returned to every user (`EV-RULE-009`).

### EV-AC-313 A Registration Desk user may post on an event

**When** a Registration Desk user posts a message on an event they can only read
**Then** the message is created (`EV-RULE-099`).

### EV-AC-314 An anonymous visitor cannot post on an event

**When** an anonymous visitor tries to post a message on a published event
**Then** the operation is refused.

### EV-AC-315 Colourless tags are hidden from the public

**Given** a tag whose colour index is zero
**Then** an anonymous visitor cannot read it, and it never appears in the public filter panel.

### EV-AC-316 The public may read the image of an unpublished ticket product

**Given** an unpublished product used by an event ticket
**When** an anonymous visitor requests its image
**Then** the image is served (`EV-RULE-048`).

### EV-AC-317 A "not in" search on wish lists is refused

**When** a search asks for the talks **not** wish-listed by a visitor
**Then** the search is refused with *"Unsupported 'Not In' operation on track wishlist visitors"* (`EV-RULE-088`).

---

## 17. Products and pricing guards

### EV-AC-325 A ticket product must have the right service tracking

**When** a product used by an event ticket has its service tracking changed away from the event value
**Then** the save is rejected with *"Products linked to an event ticket must have "<service tracking label>" set to "<Event Registration>"."* (`EV-RULE-041`).

### EV-AC-326 A booth category product must have the right service tracking

**When** a booth category is saved with a product whose service tracking is not the booth value
**Then** the save is rejected with *"The product, <product name> , is used for Event Booth, it must have service_tracking set to "Event Booth"."* (`EV-RULE-042`).

### EV-AC-327 A product used by a booth category is locked

**When** the service tracking of a variant used by a booth category is changed
**Then** the save is rejected with *"You cannot change the service_tracking of the product <product name> because it is already assigned to <booth category name>. The service_tracking must remain 'event_booth'."* (`EV-RULE-043`).

### EV-AC-328 Choosing the event service tracking sets the invoicing policy

**When** a user sets the service tracking of a product to the event value in the form
**Then** the invoicing policy becomes "ordered quantities" (`EV-RULE-044`).

### EV-AC-329 A pricelist rule with a minimum quantity warns

**When** a user sets a positive minimum quantity on a rule that applies globally
**Then** a non-blocking warning appears titled `Warning` with the body *"A pricelist item with a positive min. quantity will not be applied to the event tickets products."* (`EV-RULE-047`).

### EV-AC-330 Tax-included and discounted ticket prices

**Given** a ticket priced `100.00`, a product tax of 21 percent and a reader discount of 10 percent
**Then** the tax-included price is `121.00`, the discounted price is `90.00` and the discounted tax-included price is `108.90` (calculation 8).

### EV-AC-331 The public strike-through price

**Given** the state of EV-AC-330 and a pricelist rule allowed to show a discount
**Then** the storefront shows `100.00` crossed out next to `90.00`.
**Given** a rule not allowed to show a discount
**Then** no crossed-out price is shown and the order line takes `90.00`.

---

## 18. Closing an event

### EV-AC-340 Ended events move to the ending stage

**Given** an event whose end date has passed and whose stage is not an ending stage
**When** the housekeeping runs
**Then** the event is written into the first ending stage by sequence, which is the shipped stage `Ended`.

### EV-AC-341 Moving stage resets the kanban state except when cancelled

**Given** an event whose kanban state is `blocked`
**When** it is moved to another stage
**Then** the kanban state becomes `normal`.
**Given** an event whose kanban state is `cancel`
**When** it is moved to another stage
**Then** the kanban state stays `cancel`.

### EV-AC-342 An event with no ending stage is left alone

**Given** no stage flagged as an ending stage
**When** the housekeeping runs
**Then** nothing is written.

---

## Reconciliation notes

1. **Provenance.** Every scenario comes from version M, which was the only version that carried
   acceptance criteria. Version P announced numbered scenarios covering the ordinary path, every
   validation failure, every transition, rounding edges, several currencies and several companies;
   those are present here as sections 1 to 18, with `EV-AC-143` and `EV-AC-230` covering the
   second-currency cases and `EV-AC-312` covering the multi-company case.
2. **Messages.** The expected texts are reproduced between quotation marks and are unchanged from the
   source, including their irregular spacing.
3. **Identifiers.** The fields named in the scenarios use the storage names the database carries, so
   that a scenario can be executed against a replacement without a translation step.
4. **Two scenarios added.** `EV-AC-092` was added for the badge scanned at the desk of another event
   when the attendee is already attended, because the order of the two tests is observable and
   neither version had a scenario for it; the former `EV-AC-092`, which covers the summary returned
   by a scan, is now `EV-AC-093`. No other identifier moved.

