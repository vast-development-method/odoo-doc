# Workflows of the Events domain

Every operational procedure of the domain, end to end: who starts it, what must be true before it starts, the numbered steps, the branches, the records written with their field values, the messages posted, the notifications sent and the state of the world afterwards. The state machine table of every state field of the domain closes the file.

Rules cited as `EV-RULE-nnn` are defined in [business-rules.md](business-rules.md). Formulas cited by name are in [calculations.md](calculations.md). Booth-specific depth is in [booths-and-exhibitors.md](booths-and-exhibitors.md); programme-specific depth is in [tracks-and-agenda.md](tracks-and-agenda.md).

## Contents

1. [Create an event](#1-create-an-event)
2. [Apply an event template to an existing event](#2-apply-an-event-template-to-an-existing-event)
3. [Define the time slots of an event](#3-define-the-time-slots-of-an-event)
4. [Define the tickets of an event](#4-define-the-tickets-of-an-event)
5. [Define the questions of an event](#5-define-the-questions-of-an-event)
6. [Define the automatic communications of an event](#6-define-the-automatic-communications-of-an-event)
7. [Publish an event on the public website](#7-publish-an-event-on-the-public-website)
8. [Duplicate an event](#8-duplicate-an-event)
9. [Register an attendee from the back office](#9-register-an-attendee-from-the-back-office)
10. [Register from the public website](#10-register-from-the-public-website)
11. [Sell tickets on a sales order](#11-sell-tickets-on-a-sales-order)
12. [Change the quantity of a ticket line](#12-change-the-quantity-of-a-ticket-line)
13. [Cancel, reset or delete a sales order that sold tickets](#13-cancel-reset-or-delete-a-sales-order-that-sold-tickets)
14. [Buy tickets in the online shop](#14-buy-tickets-in-the-online-shop)
15. [Sell tickets at the shop counter](#15-sell-tickets-at-the-shop-counter)
16. [Run the communication scheduler](#16-run-the-communication-scheduler)
17. [Send a badge or a ticket to an attendee](#17-send-a-badge-or-a-ticket-to-an-attendee)
18. [Check an attendee in at the registration desk](#18-check-an-attendee-in-at-the-registration-desk)
19. [Generate leads from attendees](#19-generate-leads-from-attendees)
20. [Move ended events to the ending stage](#20-move-ended-events-to-the-ending-stage)
21. [Cancel an event](#21-cancel-an-event)
22. [Book a booth from the back office](#22-book-a-booth-from-the-back-office)
23. [Book a booth from the public website](#23-book-a-booth-from-the-public-website)
24. [Propose a talk from the public website](#24-propose-a-talk-from-the-public-website)
25. [Review and publish a talk](#25-review-and-publish-a-talk)
26. [Set a reminder on a talk and receive it](#26-set-a-reminder-on-a-talk-and-receive-it)
27. [Answer a quiz](#27-answer-a-quiz)
28. [Send a mass mailing to attendees](#28-send-a-mass-mailing-to-attendees)
29. [State machines](#29-state-machines)

---

# 1. Create an event

**Actors:** Event User, Event Administrator.
**Preconditions:** at least one Event Stage exists; the acting user belongs to the Event User group (creation is denied to the Registration Desk group).

## Steps

1. The user opens a new Event form. The following defaults are proposed:
   - `date_begin` = the current moment with seconds and sub-seconds cleared, rounded **up** to the next half hour. A current moment of 08:17 gives 08:30; 08:37 gives 09:00; 08:30 gives 08:30.
   - `date_end` = `date_begin` plus exactly one day.
   - `user_id` = the acting user.
   - `company_id` = the active company of the acting user.
   - `organizer_id` = the contact of the active company.
   - `address_id` = the contact of the active company.
   - `stage_id` = the first Event Stage by sequence.
   - `kanban_state` = `normal`.
   - `badge_format` = `A6`.
   - `website_visibility` = `public`.
   - `date_tz` = the time zone of the acting user, or coordinated universal time when the user has none.
   - `description` = the shipped default description layout, rendered without editor branding.
   - `question_ids` = every Event Question flagged as a default question and not archived.
   - `event_mail_ids` = the three shipped default schedules (see step 5).
2. The user types the `name` (required) and adjusts the dates. Entering an end date earlier than the start date is rejected by `EV-RULE-002`.
3. Optionally the user chooses an `event_type_id`. Everything the template carries is then applied; see [workflow 2](#2-apply-an-event-template-to-an-existing-event).
4. Optionally the user sets `seats_limited` and `seats_max`. While typing a new maximum, the non-blocking warning of `EV-RULE-006` may appear.
5. The three shipped default communication schedules are, in order:

   | Interval | Unit | Trigger | Template |
   |---|---|---|---|
   | 0 | Immediately | After each registration | Event: Registration Confirmation |
   | 1 | Hours | Before the event starts | Event: Reminder |
   | 3 | Days | Before the event starts | Event: Reminder |

6. The user saves. On save:
   - Every constraint of `EV-RULE-001` to `EV-RULE-005` is evaluated.
   - When the website capability package is installed, the website menu tree is created or removed according to `website_menu` and the sub-menu switches; see [workflow 7](#7-publish-an-event-on-the-public-website).
   - The scheduled date of every communication schedule is computed, and the communication scheduled job is woken for the earliest of those dates.

**Postconditions:** one Event record exists, in the first stage, with its ticket list, question list, communication list and (when configured) booth list; the public page exists but is not published until the user publishes it.

**Records written:** Event; Event Ticket (one per template ticket); Event Automated Mailing (one per template schedule or per default schedule); Event Booth (one per template booth); Website Event Menu and website menu entries when the website package is installed.

---

# 2. Apply an event template to an existing event

**Actor:** Event User.
**Trigger:** the `event_type_id` field of an Event is set or changed.

Every rule below reacts **only** to a change of the template link itself. Editing the content of a template afterwards never rewrites the events already created from it.

## Steps

1. **Seat limit.** `seats_limited` takes the "limited seats" flag of the template whenever that flag differs from the current value. A false result stays false.
2. **Seat maximum.** With no template, the current maximum is kept (zero when empty). With a template, the maximum of the template is taken (zero when empty).
3. **Time zone.** When the template defines a default time zone, that value is taken. If the field is still empty afterwards, the time zone of the acting user is taken, and finally coordinated universal time.
4. **Note.** When the note of the template is not empty, it replaces the note of the event.
5. **Ticket instructions.** When the event has empty instructions and the template has non-empty instructions, the template instructions are copied.
6. **Tags.** When the event has no tag yet and the template has tags, the template tags are copied.
7. **Tickets.** Every existing ticket that has **no registration** is deleted; then one ticket is created for each ticket line of the template, copying exactly the whitelisted fields: `sequence`, `name`, `description`, `seats_max`, and, with the product bridge installed, `product_id` and `price`. Tickets that already have registrations are kept.
8. **Communications.** Every existing schedule that is **not** already sent and has **no** per-attendee trace is deleted. Then, for every schedule line of the template whose signature `(interval_nbr, interval_unit, interval_type, template_ref)` is not already present among the kept schedules, a new schedule is created with that signature.
9. **Questions.** If any question of the event already has an answer recorded for this event, those questions are kept and every other current question is unlinked; otherwise the whole current list is emptied. Then every question of the template is linked. When the event has no template and nothing had to be kept, the list falls back to the default questions.
10. **Booths.** Every existing booth that is still `available` is deleted; then one booth is created for each booth line of the template, copying `name` and `booth_category_id`, plus `product_id` and `price` when the booth sales bridge is installed. Booths already `unavailable` are kept.
11. **Website switches.** `website_menu` takes the value of the template. The sub-switches `community_menu`, `booth_menu`, `exhibitor_menu`, `website_track` and `website_track_proposal` each take the value of the matching template switch. See [workflow 7](#7-publish-an-event-on-the-public-website) for the exact precedence.

**Postcondition:** the event carries the configuration of the template without having lost anything that was already used (sold tickets, sent communications, answered questions, booked booths).

---

# 3. Define the time slots of an event

**Actor:** Event User.
**Precondition:** the event exists and has start and end dates.

## Steps

1. The user ticks `is_multi_slots` on the event. From that moment:
   - `seats_max` is read as a maximum **per slot**, not for the whole event;
   - the ticket maximum is likewise read per slot;
   - global communications fire once per slot;
   - every registration must name a slot (`EV-RULE-021`).
2. The user opens the slot calendar. The calendar is opened with:
   - domain: slots of this event;
   - default start hour: the hour following the current hour in the time zone of the user; default end hour: one hour later;
   - the selectable day range limited to the calendar days from the start date to the end date of the event, both read in the event time zone;
   - the initial displayed date: the later of the current moment and the event start, but never after the event end.
3. For each slot the user gives a `date`, a `start_hour` and an `end_hour`, both expressed as fractional hours in the **event** time zone, plus an optional colour.
4. On save, `start_datetime` and `end_datetime` are computed by combining the date with the fractional hour in the event time zone and converting the result to coordinated universal time (see the slot datetime formula in [calculations.md](calculations.md#16-slot-datetimes-from-date-and-hours)).
5. The constraints of `EV-RULE-011`, `EV-RULE-012` and `EV-RULE-013` are evaluated.

**Postconditions:** the event has one or more slots; `event_slot_count` is the number of slots; the effective event-wide capacity becomes `seats_max × event_slot_count` for the purpose of the event-level available-seat figure.

**Deletion:** a slot with at least one registration cannot be deleted (`EV-RULE-014`).

---

# 4. Define the tickets of an event

**Actor:** Event User.

## Steps

1. In the Tickets tab of the event, the user adds a line. When created from the event form and the proposed name would be the shipped default `Registration`, the name becomes `Registration for <event name>`.
2. The user sets, for each ticket: `name`, optional `description`, optional `seats_max`, optional `limit_max_per_order`, optional `start_sale_datetime` and `end_sale_datetime`, optional `color`, and, with the product bridge installed, the `product_id` and the `price`.
3. `seats_limited` on the ticket is derived: it is true exactly when `seats_max` is non-zero.
4. With the product bridge installed:
   - `price` is proposed from the sales price of the product whenever a product is chosen and its sales price is non-zero; otherwise it stays at its current value, or zero when empty;
   - `description` is proposed from the sales description of the product when the product has one;
   - `price_incl` is the price with the taxes of the product applied for one unit;
   - `price_reduce` is the price after the discount that applies to the reading context (the pricelist of the reader);
   - `price_reduce_taxinc` is `price_reduce` with the taxes of the event company applied;
   - the chosen product must have its service tracking set to the event value (`EV-RULE-041`).
5. On save the constraints `EV-RULE-015` (coherent sales window) and `EV-RULE-016` (per-order limit) are evaluated.

**Postconditions:** the ticket list drives `start_sale_datetime` on the event (the lowest start date of the non-expired tickets, and only when every one of them has a start date), and therefore drives `event_registrations_started`.

**Deletion:** a ticket with at least one registration cannot be deleted (`EV-RULE-017`).

---

# 5. Define the questions of an event

**Actors:** Event User, Event Administrator.

## Steps

1. In the Questions tab the user links existing reusable questions or creates new ones. A question carries: `title`, `question_type`, `sequence`, `once_per_order`, `is_mandatory_answer`, `is_default`, `is_reusable` and, for a selection question, its list of suggested answers.
2. The six question types are `simple_choice` (Selection), `text_box` (Text Input), `name` (Name), `email` (Email), `phone` (Phone) and `company_name` (Company). The last four both store an answer **and** write the matching field of the registration.
3. `once_per_order` splits the list in two derived views: `general_question_ids` (asked once per order) and `specific_question_ids` (asked for each attendee).
4. `is_reusable` is forced to true for every default question, and the database refuses a default question that is not reusable (`EV-RULE-018`).
5. Changing the `question_type` of a question that already has at least one recorded answer is refused (`EV-RULE-019`).
6. A question that already has a recorded answer cannot be deleted, only archived (`EV-RULE-020`). A default question cannot be deleted at all.

**Postcondition:** the public and back-office registration forms ask exactly the linked questions, in `sequence` order, marking the mandatory ones.

---

# 6. Define the automatic communications of an event

**Actors:** Event User (create, update, delete), Registration Desk (read only).

## Steps

1. In the Communication tab the user adds a line and sets:
   - `interval_nbr` (default 1), `interval_unit` (`now`, `hours`, `days`, `weeks`, `months`, default `hours`), `interval_type` (`after_sub`, `before_event`, `after_event_start`, `after_event`, `before_event_end`, default `before_event`);
   - `template_ref`, which points either at an Email Template or, when the text message package is installed, at a Text Message Template. The selector only offers templates written on Event Registration.
2. `notification_type` is derived from the target model of the reference: `mail` for an Email Template, `sms` for a Text Message Template.
3. On save, `scheduled_date` is computed (see [calculations.md](calculations.md#5-communication-schedule-dates)) and the communication scheduled job is woken for the earliest new date.
4. When the event uses slots and the schedule is event based, one Slot Mail Scheduler is created per slot the first time the schedule runs, each with its own due date and its own resume point.

**Postcondition:** the schedule appears with a readable status: Running for an attendee-based schedule, Scheduled while a global one is pending, Sent once finished, Cancelled while the event is cancelled, Error after a failure.

---

# 7. Publish an event on the public website

**Actor:** Event User (menus), Event Administrator (also a restricted website editor).

## Steps

1. The user ticks `website_menu` on the event. On the next save:
   - a root website menu entry is created, named after the event, on the website of the event;
   - `introduction_menu` and `register_menu` both take the value of `website_menu`;
   - `community_menu` takes the value of `website_menu` only when the quiz package is installed (the base website package forces it to false);
   - `booth_menu`, `exhibitor_menu`, `website_track` and `website_track_proposal` each become true when `website_menu` has just been switched on, or when `website_menu` is on and the specific switch is still off.
2. For every switch that is on and has no menu entry yet, the matching entries are created. The full catalogue of entries, with sequence and target, is:

   | Sequence | Label | Target | Menu type | Parent |
   |---|---|---|---|---|
   | 1 | Home | a **page** built from the introduction layout | `introduction` | root |
   | 10 | Talks | `#` (a container with no page of its own) | `track` | root |
   | 10 | Talks | `/event/<event>/track` | `track` | the Talks container |
   | 15 | Agenda | `/event/<event>/agenda` | `track` | the Talks container |
   | 20 | Propose a talk | `/event/<event>/track_proposal` | `track_proposal` | the Talks container |
   | 60 | Exhibitors list | `/event/<event>/exhibitors` | `exhibitor` | root |
   | 80 | Rooms | `/event/<event>/community` | `community` | root |
   | 90 | Become exhibitor | `/event/<event>/booth` | `booth` | root |
   | 100 | Practical | `/event/<event>/register` | `register` | root |

3. An entry described by a **page layout** rather than by a target address causes a new page to be created from that layout, embedded in the event site layout, with a unique view key; the menu then points at `/event/<event>/page/<last part of the view key>`.
4. Switching a flag off deletes the matching menu entries; deleting a menu entry from the website editor switches the matching flag off on the event (the two directions are kept synchronised).
5. Switching `website_menu` off deletes the root menu and all of its children.
6. The user publishes the event. Publishing and unpublishing are tracked in the discussion thread under the subtypes "Event published" and "Event unpublished".
7. `website_visibility` controls who finds the event:

   | Value | Who sees the event in lists and searches |
   |---|---|
   | `public` | everyone |
   | `link` | only people who already have the link, plus participants |
   | `logged_users` | any signed-in user, plus participants |

   In every case the event page itself remains reachable through its direct link, and a participant always sees the event (see the participation rule in [calculations.md](calculations.md#12-participation-detection)).

**Postconditions:** the event appears at `/event/<event slug>`; opening that address redirects to the first child menu entry when a menu exists, otherwise to `/event/<identifier>/register`.

---

# 8. Duplicate an event

**Actor:** Event User.

## Steps

1. The user duplicates the event. The copy is named `<name> (copy)`.
2. Copied: the ticket list, the booth list, the communication list, the slot list, the multi-slot flag, the questions link, the dates, the venue, the organiser, the responsible, the seat configuration, the badge configuration.
3. Not copied: the stage, the kanban state, the root website menu link, and every sales or counter link on the copied children (booths lose their renter, their order links and their paid flag; communications lose their sent flag, their sent counter and their per-attendee traces).
4. When the website package is installed, after the copy is written the whole menu tree of the source event is duplicated: a new root menu named after the copy, then the introduction menus and the other menus, each with a fresh copy of its page view (the view key gets a timestamp suffix to stay unique) and a fresh copy of its page; every copied menu is re-parented under the new root.

**Postcondition:** a new event in the first stage, ready to be dated and published, with the same catalogue and the same programme skeleton.

---

# 9. Register an attendee from the back office

**Actors:** Registration Desk operator, Event User.
**Preconditions:** the event exists; when the event uses slots, a slot must be chosen; seats must be available.

## Steps

1. The user opens the attendee list of the event and creates a line. `event_id` is preset from the context.
2. The user fills `partner_id` and/or the direct contact fields. When `partner_id` is set, the empty ones among `name`, `email`, `phone` and `company_name` are filled from the **contact address** of that partner; a value already typed is never overwritten.
3. A typed telephone number is reformatted for the first country found among: the country of the partner, the country of the event, the country of the active company. If formatting fails the raw value is kept.
4. The user chooses `event_slot_id` (mandatory on a multi-slot event) and `event_ticket_id`. Choosing an event that does not own the current slot or the current ticket clears them.
5. The user answers the questions of the event in the Questions page of the form. Each answer becomes an Event Registration Answer with either a chosen suggestion or a typed text; a row with neither is refused by the database check.
6. On save:
   - a unique `barcode` is generated if none exists (`EV-RULE-025`);
   - `state` becomes `open` unless an order derivation says otherwise (see section 4.5 of [entities.md](entities.md));
   - the seat check of `EV-RULE-030` runs; when it fails the save is rejected with the sold-out message and nothing is written;
   - attendee-based communications are scheduled for the new attendee, and in synchronous mode they are executed immediately (see [workflow 16](#16-run-the-communication-scheduler));
   - lead generation rules whose trigger is "Attendees are created" run, and, if the new registration is already `open` or `done`, the rules for "Attendees are registered" and "Attendees attended" run as well.

**Postconditions:** one Event Registration exists in state `open`; the seat counters of the event, of the slot and of the ticket have increased by one; the registration confirmation message is queued when a schedule of type "After each registration" with a zero interval exists.

**Buttons available on the form:** `action_confirm` (Registered), `action_set_done` (Attended), `action_cancel` (Cancel Registration), `action_set_draft`, `action_send_badge_email` (Send by Email).

---

# 10. Register from the public website

**Actor:** website visitor (anonymous or signed in).
**Precondition:** the event is published, or the reader belongs to the Event group; registrations are open (`EV-RULE-007`).

## Steps

1. The visitor opens `/event/<event slug>`. The system redirects to the first child of the event menu, or to `/event/<identifier>/register` when the event has no menu.
2. The registration page shows the event description, the practical information, the calendar links (an external calendar link and a calendar file link) and, for a multi-slot event, the list of **open slots** grouped by date. A slot is open when it starts in the future **and** at least one of its slot-and-ticket combinations still has room (or has no limit at all).
3. **Slot choice (multi-slot events only).** The visitor picks a slot; the ticket panel is then re-rendered for that slot, with the remaining seats of each slot-and-ticket combination as the upper bound of each quantity selector.
4. **Ticket choice.** The visitor sets a quantity per ticket. An event with no ticket at all offers one generic line named `Registration`. The upper bound of each selector is the per-order limit computed by the rule in [calculations.md](calculations.md#9-per-order-ticket-limits).
5. The visitor submits. The attendee form is rendered with one block per seat, plus one block for the order-level questions:
   - each per-attendee block asks the specific questions and the identification questions;
   - the order-level block asks the questions flagged "ask once per order";
   - the first attendee block is pre-filled from the signed-in user, or from the identified visitor when one exists.
   - Two checks are performed before rendering: the per-order limit of every ticket, and, when the event limits seats, that the total requested quantity does not exceed the remaining seats of the event (or of the chosen slot). A failure is reported in the form rather than blocking it.
6. The visitor submits the attendee form. The confirmation endpoint then:
   1. verifies the anti-robot token; on failure it redirects back to the registration page with the code `recaptcha_failed`;
   2. parses the posted fields. Only `name`, `phone`, `email`, `company_name`, `event_id`, `partner_id`, `event_slot_id` and `event_ticket_id` may be written on a registration; everything else is ignored. Answers are parsed as `<attendee index>-<question type>-<question identifier>`; an index of zero means an order-level answer, which is then copied onto **every** attendee. An identification answer (`name`, `email`, `phone`, `company_name`) also writes the matching registration field, but only the **first** answer of each of those types is used per attendee;
   3. refuses any posted ticket that does not belong to the event, or that is not launched, or that is expired, with *"This ticket is not available for sale for this event"*;
   4. counts the requested seats per `(slot, ticket)` pair and calls the seat verification of `EV-RULE-030`. On failure it redirects back to the registration page with the code `insufficient_seats`;
   5. creates or reuses the site visitor record, then creates one Event Registration per attendee block with `visitor_id` set, and with `partner_id` taken from the visitor contact, or from the signed-in user, or left empty for an anonymous visitor;
   6. redirects to `/event/<identifier>/registration/success?registration_ids=<comma separated identifiers>`.
7. The success page re-reads the registrations, checking that they belong to this event **and** to the current visitor, and shows the attendee list, the calendar links for the chosen slot and a download link for the tickets.

**Branch: online ticket sales installed.** When at least one chosen ticket carries a product and any chosen ticket has a non-zero price, or a cart already exists, the flow changes; see [workflow 14](#14-buy-tickets-in-the-online-shop).

**Postconditions:** one registration per seat in state `open` (free flow) or `draft` (paid flow until the order is confirmed); the visitor is marked as participating in the event; campaign attribution values are taken from the visiting session.

---

# 11. Sell tickets on a sales order

**Actors:** Salesperson, Event User.
**Preconditions:** the ticket product exists with its service tracking set to the event value; the event has at least one ticket using that product.

## Steps

1. The salesperson adds an order line and picks the ticket **product**. Because the product is flagged as an event product, the Event Configurator opens immediately with that product.
2. In the configurator the salesperson chooses:
   - `event_id`. The picker shows availability in the name: `<event name> (Sold out)` when the event is sold out, `<event name> (<n> seats remaining)` when it has a limit, otherwise the plain name;
   - `event_slot_id`, shown only for a multi-slot event; when the event has exactly one slot it is preselected;
   - `event_ticket_id`; when exactly one ticket of that event uses the chosen product it is preselected.
   The configurator refuses a ticket or a slot that belongs to another event (`EV-RULE-050`).
3. The chosen values are written on the order line as `event_id`, `event_slot_id` and `event_ticket_id`. The line description becomes the multi-line ticket description: `<ticket display name>` (or the sales description of the product when it has one), then the event display name, then the slot display name when a slot is set, then the variant description. The template description of the product does **not** overwrite it.
4. The unit price of the line becomes the ticket price, converted into the order currency: the discounted ticket price when the matching pricelist rule is not allowed to show a discount, the plain ticket price otherwise (`EV-RULE-052`).
5. The unit of measure of the line becomes read-only.
6. The salesperson sets the quantity, which is the number of seats to sell on that line.
7. The salesperson confirms the order. On confirmation:
   1. every line whose product is an event product but which has no event is refused: *"Please make sure all your event related lines are configured before confirming this order:"* followed by one line `- <line description>` per offending line;
   2. `EV-RULE-051` also refuses any line that has an event but no ticket, or a multi-slot event and no slot;
   3. for every event line, registrations are created so that the number of non-cancelled registrations of the line equals the ordered quantity. Each new registration carries only `sale_order_line_id` and `sale_order_id`; the event, the slot, the ticket and the customer are copied from the line by the order synchronisation rule;
   4. when exactly one order is being confirmed and the line total including tax is not zero, the new registrations are created directly in state `draft` so that the attendee details can be filled before the seats are taken. In every other case (several orders at once, or a zero line total) the state is left to the derivation, which makes them `open` and therefore consumes the seats immediately;
   5. when exactly one order is being confirmed, the Attendee Details wizard opens.
8. In the Attendee Details wizard, one line is proposed per seat: the already existing registrations of each order line first, then as many new lines as are still missing to reach the ordered quantity. Each new line is pre-filled with the name, the electronic mail address and the telephone of the order customer.
9. The salesperson edits the names and contacts and validates. The wizard then:
   - writes `partner_id`, `name`, `phone` and `email` on every existing registration (falling back to the order customer value when a field is left empty);
   - creates the missing registrations with those values plus `event_id`, `event_slot_id`, `event_ticket_id`, `sale_order_id` and `sale_order_line_id`;
   - forces the state derivation on all of them, which moves the paid ones from `draft` to `open` when the order is confirmed, stamps the sale status, and therefore performs the seat check at that exact moment.

**Postconditions:** the order carries `attendee_count`, the count of its non-cancelled registrations; every registration links back to its order and line; the revenue of the event includes the line totals of confirmed orders.

**Failure path:** when the seats are not sufficient, both the order confirmation and the wizard validation raise the sold-out error of `EV-RULE-030` and nothing is written.

---

# 12. Change the quantity of a ticket line

**Actor:** Salesperson.

## Steps

1. **Increase.** Raising the quantity of a confirmed line does not create registrations by itself. Re-opening the Attendee Details wizard on the order proposes the missing lines and creates them on validation, subject to the seat check.
2. **Decrease in the back office.** Lowering the quantity leaves the existing registrations untouched; the salesperson cancels the surplus registrations by hand. The order attendee counter then follows, because it ignores cancelled registrations.
3. **Decrease in the online cart.** Lowering the quantity of a cart line that carries a ticket cancels the surplus registrations automatically: the registrations of that order, that slot and that ticket which are not already cancelled are ordered by creation date ascending, the first `new quantity` of them are kept, and exactly `old quantity − new quantity` of the following ones are cancelled.
4. **Increase in the online cart** is bounded by the remaining seats; see `EV-RULE-060`.

**Worked example.** A cart line holds 5 seats of ticket "Standard" for a single-slot event; 5 registrations exist. The visitor sets the quantity to 2. The 5 registrations are ordered by creation date; the first 2 are kept; the next 3 are cancelled. The event loses 3 reserved seats.

---

# 13. Cancel, reset or delete a sales order that sold tickets

**Actor:** Salesperson.

| Action on the order | Effect on the registrations |
|---|---|
| Cancel the order | Every registration of the order is set to state `cancel` by the state derivation; the seats are released; the registrations are kept for the record. |
| Set the cancelled order back to draft, then confirm it again | The cancelled registrations stay cancelled; the confirmation creates **new** registrations, because the count of non-cancelled registrations of each line is below the ordered quantity. A line of one seat therefore ends with one cancelled and one fresh registration. |
| Delete an order line | Every registration of that line is deleted (the link cascades). |
| Delete the whole order | Every registration of the order is deleted (the link cascades). |
| Change the customer of the order | The contact of every registration attached to the order is rewritten to the new customer. |
| Change the slot or the ticket of a registration attached to an order | A warning activity is scheduled on the order, assigned to the event responsible, or the order salesperson, or the administrator, describing the change from the old value to the new one. |

---

# 14. Buy tickets in the online shop

**Actor:** website visitor.
**Precondition:** the online ticketing package is installed; the chosen tickets carry products.

## Steps

1. The visitor goes through steps 1 to 6 of [workflow 10](#10-register-from-the-public-website). At the attendee-creation step the behaviour splits:
   - **All chosen tickets are free and no cart exists:** the registrations are created directly, with no order at all, and the flow ends on the success page.
   - **Otherwise:** a cart is created (or the existing one reused). For each `(slot, ticket)` pair the matching quantity is added to the cart, producing one order line per pair; the created line identifier is then written on each registration together with the order.
2. The registrations are created with `sale_order_id` and `sale_order_line_id` set, which makes the state derivation apply: with a non-zero order total they are `draft` and `to_pay`.
3. After creation:
   - when the cart contains no ticket line at all, the free confirmation page is shown;
   - when **every** line of the cart is a ticket line whose ticket price is zero, the order is confirmed immediately, the cart is reset and the visitor is sent to the order confirmation page;
   - otherwise, if the cart is still anonymous, a customer contact is created (or matched) from the details of the first attendee and set on the order, and the visitor is sent to the checkout with the address step skipped when possible.
4. At payment time, before a transaction is accepted, the seats are verified once more: the non-cancelled ticket registrations of the order are grouped per event and per `(slot, ticket)` pair and the seat verification of `EV-RULE-030` is applied. A failure refuses the payment.
5. Once the order is confirmed, the state derivation moves every registration of the order from `draft` to `open` and sets its sale status to `sold`, which consumes the seats and triggers the attendee-based communications.
6. The shop confirmation page lists, per event, the attendees created by the order (grouped per slot for a multi-slot event) with the calendar links of each slot.

**Cart rules specific to tickets.**
- A cart line is matched to an existing line only when the slot **and** the ticket are identical, which keeps one line per `(slot, ticket)` pair.
- The quantity of a ticket line can never be raised by hand from the cart page: *"You cannot raise manually the event ticket quantity in your cart"*.
- Adding tickets is capped by the remaining seats; see `EV-RULE-060` for the two messages.
- Abandoned-cart reminders are not sent for a cart whose tickets are no longer available for sale.
- The full billing address is not requested when every line of the order is a ticket line, unless the configuration parameter `website_event_sale.require_billing_details_for_events` is set to a true value.
- The line label shown in the cart is the ticket display name, not the product name; the strike-through original price is hidden for event lines; an event line cannot be re-ordered from the order history.

---

# 15. Sell tickets at the shop counter

**Actor:** shop counter operator.
**Precondition:** the counter and event bridge is installed; the ticket product is available at the counter.

## Steps

1. When a counter session opens, the following data are loaded into the counter: the tickets whose event is not finished, whose event belongs to the company of the counter, whose product is loaded, and whose sales window is open at that moment; their events; the future slots of those events; the questions and suggested answers of those events; and the existing registrations and answers needed to display them.
2. The operator adds a ticket product to the order. The counter asks for the event, the slot (when the event has slots) and the ticket, then for the attendee details and the answers to the event questions. Answers to questions that are not mandatory may be skipped.
3. Validating the order creates one Event Registration per sold seat, each carrying `pos_order_line_id`. The customer of the counter order becomes the contact of the registration when the registration has none; empty name, electronic mail, telephone and company values posted by the counter are dropped so that the computed values apply.
4. The state derivation then applies the counter rules: a cancelled counter order gives `cancel`; a zero-total counter order gives sale status `free` and state `open`; any other counter order gives sale status `sold` and state `open`. With the counter-and-sales bridge installed the rule becomes: counter order in state `paid`, `done` or `invoiced` gives `sold` and `open`; any other counter state gives `to_pay` and `draft`.
5. Every creation or update of a counter registration pushes the new seat figures to **every** open counter session: for each affected event, the remaining seats of the event, of each of its tickets and of each of its slots are broadcast under the message `UPDATE_AVAILABLE_SEATS`.
6. When the order is paid, a badge message is sent to every attendee of the order who has an electronic mail address.
7. The operator can print the full-page tickets or the badges of the order from the order screen.
8. **Refund.** When a counter order line that carries registrations is refunded, the number of registrations to cancel is `refunded quantity − already cancelled registrations`; that many non-cancelled registrations of the line are set to `cancel`.

**Worked example.** A line sold 4 seats. Two seats are refunded, and no registration of the line is cancelled yet: `4 − 0 = 2`... the refund quantity is 2 and the already cancelled count is 0, therefore 2 registrations are cancelled. A second refund of 1 seat gives `3 − 2 = 1`, therefore 1 more registration is cancelled, leaving 1 active.

---

# 16. Run the communication scheduler

**Actor:** the scheduled job runner. The job "Event: Mail Scheduler" runs every 24 hours, starting 15 minutes after installation, and is additionally woken whenever a schedule date is recomputed or a new attendee is registered.

## Step 0: selection

The job selects every Event Automated Mailing matching **all** of:

```
event.active = true
AND event.kanban_state <> "cancel"
AND scheduled_date <= now
AND mail_done = false
AND (interval_type <> "after_sub" OR event.date_end > now)
```

Each selected schedule is then executed on its own; an exception in one schedule is caught, logged, reported (see step 4) and does not stop the others. When the job is run with automatic committing, the work of each schedule is committed before the next one starts.

## Step 1: template validity

Before anything is sent, a schedule whose `template_ref` does not point at the model matching its notification type, or whose referenced record no longer exists, is removed from this run and a warning is written to the technical log naming the schedule, the event and the missing or invalid template. Such a schedule is simply skipped; it is not marked done and not marked in error.

## Step 2: dispatch

| Condition | Mode |
|---|---|
| `interval_type = after_sub` | attendee based |
| the event uses slots | slot based |
| otherwise | event based |

### Event-based mode (one global communication)

1. Skip when `mail_done` is already true.
2. Skip unless `scheduled_date <= now` **and** (the trigger is not "before the event starts" or "after the event started", **or** the event has not ended yet). A reminder scheduled before the start is therefore never sent after the event is over, while an "after the event ended" message still goes out.
3. Read the attendees to contact: registrations of the event whose state is neither `draft` nor `cancel`, with identifier strictly greater than the resume point when one exists, ordered by identifier ascending, limited to `render limit + 1` records.
4. When no attendee is found: mark `mail_done = true` and stop. An event with no attendee is therefore finished in one pass.
5. When more attendees were found than the render limit, keep the first `render limit` of them and wake the job again immediately.
6. Process the kept attendees in batches of `batch size`. For each batch:
   - send the message to that batch (electronic mail or text message, see step 3);
   - move the resume point to the last attendee of the batch;
   - refresh the progress counters (see step 5);
   - commit and empty the working memory when automatic committing is on.

`batch size` is the configuration parameter `mail.batch_size`, defaulting to 50 when absent or zero. `render limit` is the configuration parameter `mail.render.cron.limit`, defaulting to 1000 when absent or zero.

### Slot-based mode (one global communication per slot)

1. Create a Slot Mail Scheduler for every slot of the event that does not have one yet.
2. For each slot trace, skip when it is already done; otherwise, when its own due date has passed **and** (the trigger is not "before the event starts" or "after the event started", **or** the slot has not ended yet), run the event-based procedure above, but with the attendee list further restricted to that slot and with the resume point, the counters and the done flag kept on the slot trace.

### Attendee-based mode (one communication per attendee)

1. Find attendees of the event whose state is neither `cancel` nor `draft` and which have no trace yet for this schedule, ordered by identifier ascending, limited to twice the render limit. When the run was started for specific attendees (the synchronous path of a new registration), the list is further restricted to those attendees.
2. Create the missing traces in chunks of 500. The due date of each trace is `registration.created_on` (sub-seconds cleared) plus the interval of the schedule.
3. Select the traces to send: not sent, with a due date set, and due at or before the current moment, ordered by identifier ascending, limited to `render limit + 1`. When more than the limit are found, keep the first `render limit` and wake the job again.
4. Process the selected traces in batches of `batch size`. In each batch:
   - traces whose registration has fallen back to `draft` or `cancel` are **deleted** without sending anything;
   - the remaining traces are sent, grouped per schedule, and marked `mail_sent = true`;
   - the progress counters are refreshed, then the work is committed when automatic committing is on.
5. Attendee-based messages deliberately **ignore** the mass-mailing exclusion list, because registering to an event is itself a subscription to the messages of that event. Global event-based messages do apply that exclusion list.

## Step 3: sending

- **Electronic mail.** The author is the first of: the organiser of the event when it has an electronic mail address, the active company when it has one, the acting user when they have one, the platform root contact. The message is composed in mass mode with sending deferred to the outgoing queue, on the registration records, from the referenced template; the sender address is the one of the template when it has one, otherwise the formatted address of the author. Contacts are **not** created for the recipients: the recipient addresses stay on the outgoing message.
- **Text message.** A mass text message is scheduled on the registrations from the referenced text message template, keeping a log of the sending.

## Step 4: failure

When the execution of a schedule raises, the run of that schedule is abandoned, the working memory is emptied and the failure is reported **at most once per hour** per schedule:

1. If `error_datetime` is empty or older than one hour, a message is posted on the **event**, addressed to the organiser, the event responsible and the last author of the template (only the active ones among them), and sent through the outgoing queue. Its body is built in three parts:
   - *"Communication for <event name> scheduled on <scheduled date> failed."* The scheduled date is the current moment for an attendee-based schedule, and the stored due date otherwise.
   - Either *"This is due to an error in template <template link>."* when the failure came from rendering the template, or *"This may be linked to template <template link>."* otherwise.
   - Either *"There is an issue with dynamic placeholder. Actual error received is: <error>."* when a dynamic placeholder failed, or *"Rendering of template failed with error: <error>."* for any other rendering failure, or *"It failed with error <error>."* for a non-rendering failure.
2. `error_datetime` is stamped with the current moment with sub-seconds cleared, which turns the readable status of the schedule into Error.
3. A successful execution of the same schedule clears `error_datetime`.

## Step 5: progress counters

| Mode | `mail_count_done` | `mail_done` |
|---|---|---|
| attendee based | the number of traces of this schedule with `mail_sent = true` | never set by the counters; the schedule stays Running |
| slot based | on the slot trace: the number of registrations of that slot, of that event, not `draft` and not `cancel`, whose identifier is at most the slot resume point. On the schedule: the sum over its slot traces | on the slot trace: `count >= slot.seats_taken`. On the schedule: `sum of slot counts >= event.seats_taken` |
| event based | the number of registrations of the event, not `draft` and not `cancel`, whose identifier is at most the resume point | `count >= event.seats_taken` |
| nothing processed yet | 0 | false |

## Synchronous path on registration

When a registration is created, or moves into state `open` from `draft` or `cancel`:

1. Only registrations that are `open` are considered.
2. The attendee-based schedules of their events are read with elevated rights.
3. If none exists, nothing happens.
4. If the configuration parameter `event.event_mail_async` is set, or the work happens during a file import, the communication job and the outgoing-message job are simply woken and the work is left to them.
5. Otherwise each attendee-based schedule is executed immediately, with elevated rights, restricted to those registrations. A failure is caught, logged and reported through the same one-per-hour rule as above.
6. During a module installation this whole path is skipped.

---

# 17. Send a badge or a ticket to an attendee

**Actors:** Registration Desk operator, Event User, attendee.

## From the back office

1. The user selects one registration and presses "Send by Email". A message composer opens on the registration, pre-loaded with the template "Event: Registration Badge", in comment mode.
2. That template carries the Badge report as an attachment, is addressed to the default recipients of the record, uses the language of the event (or of the contact), takes the organiser (or the company, or the user) as sender, and its attachment is deleted after sending.
3. The user may also print the Badge report or the Full Page Ticket report directly from the list or the form; both are bound to Event Registration, and the equivalent example reports are bound to Event.

## From a message link

The endpoint `/event/<event identifier>/my_tickets` returns the printed tickets of a set of registrations without requiring a login. It is protected by a keyed hash:

1. The caller passes the event identifier, the list of registration identifiers and the hash.
2. Missing parameters, or an unknown event, or a hash that does not match the ground truth for that exact event and that exact sorted list of registrations, all give "not found".
3. The registrations are filtered to those belonging to the event; an empty result gives "not found".
4. The document name is `<prefix> - <event name> (<start date of the first registration, medium format, in the event time zone>)`, where the prefix is `Ticket` for the responsive page, `Badges` in badge mode and `Tickets` otherwise; when exactly one registration is requested, ` - <attendee name>` is appended.
5. The response is either a responsive rich-text page or a printed document, in badge layout or in full-page-ticket layout.

## Content of the printed documents

- **Badge**, in one of three layouts chosen by `badge_format`:
  - `A6`: one card per page half.
  - `four_per_sheet`: four cards per sheet, two per row.
  - `A4_french_fold`: a foldable sheet with two front cards on the top row, the barcode block and the selection answers bottom-left, and the four folding illustrations bottom-right.
  Every card shows the event name over the optional badge background image, the start and end date and time in the event time zone, the venue block, the attendee name, the attendee company name, the organiser logo, a quick response code of the barcode, the linear barcode when the barcode feature is enabled, and a coloured strip with the ticket name using the ticket colour (defaulting to `#875A7B`).
- **Full Page Ticket**: the event name, the ticket name, the attendee name, the selection answers as chips, the venue block, the date block (the slot dates when the attendee has a slot, otherwise the event dates, shortened to one line when the event lasts one day), a quick response code, the linear barcode when enabled, the ticket instructions, and a repeated footer with the organiser name, telephone, electronic mail address and website.
- **Attendee List**: a heading, the event name, the event date range in the event time zone, then a table with the columns Name, Company, Ticket type, Phone number and a quick response code of the barcode, with a page break after each event.

---

# 18. Check an attendee in at the registration desk

**Actor:** Registration Desk operator.
**Precondition:** the operator belongs to the Registration Desk group.

## Steps

1. The operator opens the Registration Desk screen. The screen is initialised for an optional event:
   - with an event: it shows the event name, the country and city of the venue, and the company name and identifier of the event;
   - without an event: it shows the label `Event Registrations`, no country, no city, and the active company.
2. The operator scans a badge. The scanned value is matched against the barcodes of the registrations, taking the first match.
3. The outcome is one of:

   | Situation | Outcome | Effect |
   |---|---|---|
   | No registration carries that barcode | `invalid_ticket` | none |
   | The registration is cancelled | `canceled_registration` | none |
   | The registration is unconfirmed | `unconfirmed_registration` | none |
   | The event of the registration has finished | `not_ongoing_event` | none |
   | The desk is opened for a given event and the registration belongs to another event | `need_manual_confirmation` | none |
   | The registration is already attended | `already_registered` | none |
   | Otherwise | `confirmed_registration` | the registration is set to `done` |

   The order of the tests is exactly the order of the rows.
4. Whatever the outcome, the answer carries the registration summary: identifier, attendee name, contact, slot display name, ticket name, event identifier and display name, the display text of every selection answer, the company name, the badge format, the attendance date in short format, and whether that date falls on the current day in the event time zone. With the product bridge it also carries the sale status, its readable label and a "has to pay" flag that is true when the sale status is `to_pay`.
5. Alternatively the operator picks an attendee from the kanban or list view of the desk and presses "Mark as Attending", which performs the same state change without a scan.

**Effect of setting the state to `done`:** `date_closed` is stamped with the current moment when it is still empty; a note *"Attended on <date, short format>"* is logged in the discussion thread of the registration; the seat moves from reserved to used, leaving `seats_taken` unchanged; lead rules with the trigger "Attendees attended" run.

---

# 19. Generate leads from attendees

**Actors:** the system (automatic triggers), Event Administrator (manual regeneration), the scheduled job runner (batch regeneration).

## Automatic triggers

| Moment | Rules that run |
|---|---|
| A registration is created | every rule whose trigger is "Attendees are created"; plus, when the new registration is already `open`, the rules whose trigger is "Attendees are registered"; plus, when it is already `done`, the rules whose trigger is "Attendees attended" |
| A registration is written with state `open` | every rule whose trigger is "Attendees are registered" |
| A registration is written with state `done` | every rule whose trigger is "Attendees attended" |
| A registration is created or written during a data import | none: the rules are deliberately skipped |

## Rule application procedure

For a given set of rules and a given set of registrations:

1. The registrations are ordered by identifier ascending, so that the first created wins any tie.
2. **Duplicate protection.** Every existing lead, including archived ones, that is linked both to one of those registrations and to one of those rules is read; for each rule the registrations already covered are excluded.
3. **Filtering.** For each rule, the remaining registrations are filtered:
   - when the rule carries a registration filter that is not the empty filter, only registrations matching it are kept;
   - when the rule names a company, only registrations whose company is that company are kept;
   - when the rule names an event or event templates, only registrations whose event is that event **or** whose event uses one of those templates are kept; when the rule names neither, this test passes.
4. **Grouping (order-based rules only).** Registrations are grouped:
   - base rule: by event and by creation moment. Registrations created in one batch, such as the several attendees of one website registration, therefore form one group;
   - with the sales bridge: registrations that carry a sales order are grouped **by sales order** instead, and for each group the existing leads of that rule whose registrations belong to that same order are looked up so that they can be updated rather than duplicated.
5. **Creation.**
   - Per-attendee rules create one lead per remaining registration.
   - Per-order rules, for each group: if an existing lead was found it is **updated**, its description gaining a paragraph `New registrations` followed by a numbered list of the new registrations, and the new registrations being linked to it; otherwise one lead is created per event represented in the group.
6. Leads are created with elevated rights, because the people running the rules may not have access to the sales pipeline.

## Values written on a lead

| Lead field | Value |
|---|---|
| type | the lead type of the rule (`lead` or `opportunity`) |
| salesperson | the salesperson of the rule |
| sales team | the sales team of the rule |
| tags | the tags of the rule |
| originating rule | the rule |
| source event | the event of the registrations |
| referred | the name of the event |
| source registrations | the registrations of the group |
| campaign, source, medium | the first non-empty value found among the registrations, ordered by identifier |
| contact block | see below |
| description | `Participants` followed by a numbered list, one entry per registration |
| visitor and language | with the website bridge: the site visitor of the registration and its language |

**Contact block.** The first contact among the registrations that is not the anonymous public contact is the candidate.

- For a **single** registration with a candidate contact, the contact is kept only when it really matches the attendee: the electronic mail addresses must match after normalisation (or literally when the contact has no normalised address), and the telephone numbers must match after formatting in the country of the contact (or literally when either cannot be formatted). A mismatch drops the contact.
- With a kept contact, the lead contact values are prepared from that contact; the electronic mail address is forced from the registrations only when the contact has none, and likewise for the telephone.
- With no kept contact, the lead carries `contact_name`, `email_from` and `phone` taken as the first non-empty value among the registrations, and no language.
- In both cases the lead name is `<event name> - <contact name>`, where the contact name is the contact name, or the first non-empty attendee name, or the first non-empty attendee electronic mail address.

**Description line of one registration:** `<attendee name or contact name or electronic mail address> (<electronic mail address> - <telephone>)` with only the non-empty parts joined, plus an optional suffix such as `(updated)`. With the website bridge, the questions and answers of the registration are appended under the heading `Questions`, one block per answer with the question title and the answer value.

## Updating leads when attendee data change

When a registration that already has leads is written:

1. The tracked values before the write are captured for the union of the contact fields (`name`, `email`, `phone`, `partner_id`) and the description fields (`name`, `email`, `phone`, and, with the website bridge, the answers).
2. After the write, for every **per-attendee** lead of that registration:
   - when the contact changed, the contact fields of the registration are recomputed and taken into account;
   - when any contact field really changed, the lead contact block is rebuilt;
   - when any description field really changed, the description of each lead gains a paragraph `Updated registrations` with the new line of that registration.
3. For every **per-order** lead, only a change of contact matters: the contact block is rebuilt from the whole group, and the description gains either `Participants` (when the lead had no contact yet) or `Updated registrations` with the suffix `(updated)`.

## Manual regeneration

1. An Event Administrator presses "Generate Leads" on one or several events. Any other user is refused with *"Only Event Managers are allowed to re-generate all leads."*
2. The registrations of those events that are neither `draft` nor `cancel` are counted.
3. **Small volume** (at most 200 registrations): the rules are applied immediately and a notification is shown: *"Yee-ha, <n> Leads have been created!"* when at least one lead was created, *"Aww! No Leads created, check your Lead Generation Rules and try again."* otherwise.
4. **Large volume**: one Event Lead Request is created per event (at most one request per event at a time, enforced by a unique constraint whose message is *"You can only have one generation request per event at a time."*), the batch job is woken, and the notification is *"Got it! We've noted your request. Your leads will be created soon!"*
5. The batch job "Generate Leads based on Rules", whose external identifier is `event_crm.ir_cron_generate_leads`, runs daily and, at each run, takes at most 100 requests. For each request it reads at most 200 registrations of the event that are neither `draft` nor `cancel` and whose identifier is greater than the stored resume point, ordered by identifier ascending, applies the rules of the request (or every matching rule when the request names none), and then either marks the request fulfilled (when fewer than a full batch was read) or stores the last processed identifier. Each completed batch is committed. Fulfilled requests are deleted, and if any request remains unfinished the job wakes itself again.

---

# 20. Move ended events to the ending stage

**Actor:** the periodic housekeeping job.

1. The job selects every event whose `date_end` is strictly before the current moment and whose current stage is **not** flagged as an ending stage.
2. Each selected event is written into the **first** stage by sequence that is flagged as an ending stage. When no such stage exists, nothing happens.
3. Writing the stage resets `kanban_state` to `normal`, unless it is `cancel`, which is preserved.

The same move can be triggered by hand with the "Set as Done" operation on one or several events.

---

# 21. Cancel an event

**Actor:** Event User.

1. The user sets `kanban_state` to `cancel` on the event.
2. From that moment:
   - registrations are no longer open, whatever the dates and the seats;
   - the communication scheduler skips every schedule of that event;
   - the readable status of every unfinished schedule becomes Cancelled;
   - moving the event to another stage keeps the cancelled kanban state, because only non-cancelled states are reset to `normal`.
3. Archiving the event (`active = false`) has the same effect on the scheduler and additionally removes the event from every default list.

---

# 22. Book a booth from the back office

**Actors:** Event User, Salesperson.

## Direct booking (no order)

1. The user opens the booth of the event and fills the renter: `partner_id`, and optionally `contact_name`, `contact_email` and `contact_phone` (each filled from the renter contact only while empty).
2. The user presses "Confirm", which writes `state = unavailable` together with the collected values in a single write.
3. The post-confirmation rule runs on the booths that were `available` before the write:
   - when the booth category asks for a sponsor and the booth has a renter, a Sponsor is created or reused (see [booths-and-exhibitors.md](booths-and-exhibitors.md#creating-the-sponsor-of-a-booth));
   - a message built from the booking layout is posted **on the event** with the subtype "Booth Booked".

## Booking through a sales order

1. The salesperson adds an order line with a booth product. The Booth Configurator opens with that product and asks for the event, then the booth category (only categories that still have free booths are offered), then one or more booths of that category. Selecting nothing is refused with *"You have to select at least one booth."*
2. The chosen booths are written on the line as pending booths, which creates one Event Booth Registration per booth, each carrying the order line, the booth and the customer.
3. The line description becomes `<event display name> : ` followed by one line `- <booth name>` per booth; the line price becomes the sum of the category prices (discounted when the pricelist rule may not show a discount), converted into the order currency.
4. Changing the product of the line clears the event when the product no longer matches the pending booths; changing the event clears the pending booths.
5. All reservations of one order line must belong to a single event (`EV-RULE-071`).
6. On order confirmation:
   - a booth line with no pending booth is refused with *"Please make sure all your event-booth related lines are configured before confirming this order:"* followed by one line per offending line;
   - for each booth line that has pending booths and no confirmed booth yet, the booths that are no longer available are refused with *"The following booths are unavailable, please remove them to continue : "* followed by one indented line per booth;
   - otherwise every reservation of the line is confirmed: its collected values are written on its booth through the confirm operation, which marks the booth unavailable and runs the post-confirmation rule; then every **other** pending reservation on the same booths loses: a message is posted on each losing order for its salesperson, reading *"Your order has been cancelled because the following booths have been reserved"* followed by the booth names, each losing order is cancelled, and the losing reservations are deleted.
7. When the invoice that carries the booth lines is paid, every confirmed booth of those lines is stamped `is_paid = true`.

**Deletion guard:** a booth linked to a sales order cannot be deleted (`EV-RULE-072`).

---

# 23. Book a booth from the public website

**Actor:** website visitor.
**Precondition:** the booth menu of the event is active; the visitor can read the event.

## Steps

1. The visitor opens `/event/<event slug>/booth`. The page shows the booth categories that still have at least one free booth, with the default category preselected, the booths of the event, and the exhibition map when one is set. Without read access the request is forbidden.
2. The visitor selects a category and one or several booths of that category, then submits; the browser is redirected to the contact form with the chosen booth identifiers and category in the address.
3. The contact form pre-fills the name, the electronic mail address and the telephone from the signed-in user, or from the identified visitor.
4. The visitor submits the contact form. The confirmation endpoint then:
   1. re-reads the requested booths, keeping only those of this event that are still `available`; if the set read back differs from the set requested, or if the booths span more than one category, the answer is the error code `boothError`;
   2. with online booth sales installed, an unknown category gives `boothCategoryError`;
   3. for an anonymous visitor whose electronic mail address already belongs to a known contact, the answer is `existingPartnerError`, so that the visitor signs in instead of creating a duplicate;
   4. resolves the contact: for an anonymous visitor a contact is found or created from the normalised electronic mail address, with the posted name and telephone; for a signed-in visitor the contact of the user is used;
   5. **without** online booth sales: confirms the booths immediately with the collected contact values, which marks them unavailable, creates the sponsor when the category asks for one, and posts the booking message on the event. The answer carries `success`, the event name and the contact block;
   6. **with** online booth sales: does **not** confirm. Instead a cart is created or reused, the customer is set on an anonymous cart, and one cart line is added for the booth product with quantity one, the chosen booths as pending booths and the contact values. When the cart total is non-zero the answer redirects to the cart; when it is zero the order is confirmed straight away (which confirms the booths through the order confirmation rule), the cart is reset and the success answer is returned.
5. Two helper endpoints support the page: one returns the identifiers of the requested booths that are no longer available, the other returns the free booths of a category as identifier and name pairs.

**Online cart rules for booths.** A cart line is matched to an existing line only when it already contains one of the requested booths, which prevents two lines competing for the same booth. The quantity of a booth line can never exceed one: *"You cannot manually change the quantity of an Event Booth product."* Updating a booth line deletes its reservations and creates new ones from the new selection.

---

# 24. Propose a talk from the public website

**Actor:** website visitor (speaker).
**Precondition:** `website_track_proposal` is on for the event; the event is reachable from the current website.

## Steps

1. The visitor opens `/event/<event slug>/track_proposal` and fills the proposal form: the talk title, the abstract, the speaker name, electronic mail address, telephone, job position and biography, an optional photograph, the chosen tags, and optionally a separate contact block.
2. The posted tags are filtered through a search, so that only existing tags the visitor is allowed to see are kept; colourless tags are therefore refused for anonymous visitors.
3. Contact resolution:
   - when the visitor asked to add contact information, at least one of the contact electronic mail address and the contact telephone must be filled, otherwise the answer is the error code `invalidFormInputs`. If the normalised contact address equals that of the identified visitor, the contact of that visitor is reused; otherwise a new contact is created with that address, the posted contact name and the posted telephone;
   - when the visitor did not ask for a separate contact block, the contact of the identified visitor is reused when the normalised speaker address matches it; otherwise no contact is set.
4. A talk is created with: the title, the contact, the speaker block, the operational contact electronic mail address and telephone taken from the resolved contact, the event, the tags, the abstract and the biography converted from plain text to rich text, the photograph, and **no responsible user**. The creator is not automatically subscribed to the thread.
5. When the visitor is signed in as a real user, that user is subscribed to the talk thread so that they receive the review messages.
6. On creation, a message built from the "new talk" layout is posted on the **event** under the subtype "New Track", and the stage synchronisation of the first stage is applied.
7. The answer is `success`, and the page shows the confirmation block.

**Postcondition:** the talk sits in the first review stage (shipped: "Proposal"), invisible in the agenda and unpublished.

---

# 25. Review and publish a talk

**Actors:** Event User, Event Administrator.

## Steps

1. The reviewer opens the talk kanban grouped by stage. Every stage is shown, including empty ones.
2. Moving a talk to another stage:
   - resets `kanban_state` to `normal` unless a new kanban state is written in the same operation;
   - publishes the talk when the target stage is flagged "fully accessible";
   - unpublishes the talk when the target stage is flagged "cancelled stage";
   - sends the electronic mail template of the target stage, when it carries one, to the speaker as an internal note using the light notification layout.
3. The shipped stages are:

   | Sequence | Name | Visible in agenda | Fully accessible | Cancelled | Folded | Template |
   |---|---|---|---|---|---|---|
   | 1 | Proposal | no | no | no | no | none |
   | 2 | Confirmed | no | no | no | no | Event: Track Confirmation |
   | 3 | Announced | yes | no | no | no | none |
   | 4 | Published | yes | yes | no | no | none |
   | 5 | Refused | no | no | no | yes | none |
   | 6 | Cancelled | no | no | yes | yes | none |

4. The reviewer schedules the talk: a date, a duration in hours and a room. Setting the start recomputes the end as `start + duration`; setting the end recomputes the duration from the two datetimes.
5. The reviewer may mark the talk as a key talk (`wishlisted_by_default`), attach a video link, enable the action button with its title, target and delay, and attach a quiz.
6. The talk appears on the public agenda as soon as its stage is visible in the agenda; it becomes readable by anonymous visitors only once published.

Details of the agenda grid, the suggestions, the reminders, the video and the quiz are in [tracks-and-agenda.md](tracks-and-agenda.md).

---

# 26. Set a reminder on a talk and receive it

**Actor:** website visitor.

## Toggle

1. The visitor presses the reminder control on a talk. The request carries the talk and the desired state.
2. The talk is fetched; a visitor that cannot read it is served through elevated rights only for this operation, and a talk whose event is not reachable from the current website gives "not found".
3. A link record between the visitor and the talk is created when it is missing and either the visitor is switching the reminder on, or the talk is a key talk. For an anonymous visitor a site visitor record is force-created; the last-visit stamp of the visitor is refreshed.
4. **Ordinary talk:** the request is ignored (answer `ignored`) when no link exists or the wish-listed flag already equals the desired state; otherwise `is_wishlisted` takes the desired state.
5. **Key talk:** the request is ignored when no link exists or the blacklisted flag differs from the desired state; otherwise `is_blacklisted` becomes the opposite of the desired state. A key talk is therefore "on" for everybody until a visitor explicitly opts out.
6. The answer carries the new reminder state.

## Send the reminder by electronic mail

1. The visitor asks for the talk reminder to be sent to an address.
2. The recipient is the posted address for an anonymous visitor, and the address of the signed-in user otherwise, after normalisation.
3. The request is refused, with a message and no sending, when:
   - the talk is not visible to that visitor: *"Invalid data."*;
   - the address is not a valid one: *"Invalid email."*;
   - the talk is already finished, or the event is finished: *"The talk is already finished."*;
   - the talk has already begun: *"The talk has already begun."*
4. Otherwise the "Add reminder via email" template is sent to that address, rendered in the language of the visitor session for an anonymous visitor and in the language of the user otherwise. The message carries the calendar links of the talk.

---

# 27. Answer a quiz

**Actor:** website visitor.
**Precondition:** the quiz package is installed and the talk carries a quiz.

## Submit

1. The visitor answers every question of the quiz and submits the chosen answer identifiers.
2. The talk is fetched and a visitor link is force-created.
3. When the link already records a completed quiz, the answer is the error `track_quiz_done` and nothing changes.
4. The submitted answers are read with elevated rights, restricted to answers belonging to the quiz of this talk. When the number of **distinct questions** covered by the submitted answers differs from the number of questions of the quiz, the answer is the error `quiz_incomplete`.
5. Otherwise the points are summed over the submitted answers and written on the visitor link together with `quiz_completed = true`.
6. The answer returns, per question: the points awarded by the chosen answer, the text of the correct answer, whether the chosen answer was correct, and the extra comment of the chosen answer; plus the completion flag and the total points.

## Reset

1. The visitor asks to reset the quiz.
2. The request is forbidden unless the quiz allows unlimited tries **or** the reader is an Event Administrator (administrators may always reset, for testing).
3. The visitor link is force-created if missing, then `quiz_completed` becomes false and `quiz_points` becomes zero.

**Leaderboard.** See [tracks-and-agenda.md](tracks-and-agenda.md#leaderboard).

---

# 28. Send a mass mailing to attendees

**Actor:** Event User, with the mass mailing capability package installed.

1. From the event, the user presses "Mass Mail Attendees" (electronic mail) or the text message equivalent. A mailing is prepared on Event Registration with a default selection restricted to the attendees of that event.
2. From the event, the user may also press "Invite Contacts", which prepares a mailing on contacts.
3. With the programme package, "Mass Mail Speakers" prepares a mailing on Event Track restricted to the talks of that event.
4. Unlike the automatic communications of the event, a mass mailing **does** honour the mass-mailing exclusion list.

---

# 29. State machines

Every state field of this domain — the event stage and its progress marker, the registration life
cycle and its payment situation, the readable status of an automatic communication, the booth
availability state, the talk stage and its progress marker, the derived flags of a talk stage and
the website visibility of an event — is specified state by state and transition by transition, with
its guards, its refusal messages, its side effects and a diagram, in
[`state-machines.md`](state-machines.md). The procedures above name the operation that triggers each
transition; the state machine document states what the transition is allowed to do and what it
writes.
