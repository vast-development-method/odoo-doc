# Business rules of the Events domain

The complete rule catalogue: validations, guards, permissions, consistency rules, uniqueness rules,
date rules, rounding rules and the exact message the system shows when it refuses. Each rule carries
a stable identifier of the form `EV-RULE-nnn`, so that the other documents of this folder, and other
domains, can cite it.

Conventions: a verbatim user-facing message is reproduced between quotation marks; a stored value,
an identifier or a short interface label is reproduced in code font; a placeholder written between
angle brackets is replaced by the value named inside it. Where the text of the system contains an
irregularity — a double space, a space before a colon, a missing article — the irregularity is
reproduced exactly, because a replacement must produce the same text.

Rules are grouped as follows: `001`–`010` the event record, `011`–`014` time slots, `015`–`017`
tickets, `018`–`020` questions, `021`–`031` registrations, `032`–`040` automatic communications,
`041`–`049` products and prices, `050`–`059` selling on a sales order, `060`–`068` the online shop
and the configurators, `070`–`079` booths, `080`–`089` the programme, `090`–`099` access, website
and cross-cutting rules. The identifier `069` is not used; every other number in those ranges is.
A complete index is in section [Rule identifier index](#rule-identifier-index), and the mapping of
the identifiers used by the two source versions is in section
[Mapping of former rule identifiers](#mapping-of-former-rule-identifiers).

---

## The event record

### EV-RULE-001

**An event always has a name, a start date, an end date, a display time zone and a seat-limit flag.** `name`, `date_begin`, `date_end`, `date_tz` and `seats_limited` are required. `date_tz` is pre-computed on creation, so that a record created through an integration without naming a zone still gets one: the zone of the template, then the zone of the acting user, then coordinated universal time.

### EV-RULE-002

**The end date cannot precede the start date.** Checked on every change of `date_begin` or `date_end`. Message: *"The closing date cannot be earlier than the beginning date."*

### EV-RULE-003

**The online-event link must be a complete web address.** When `event_url` is filled, it must parse into an address that has both a scheme and a host. Message: *"Please enter a valid event URL."*

### EV-RULE-004

**The online-event link belongs to events without a venue.** Whenever `address_id` is set, `event_url` is emptied automatically. While typing in the form, a link whose scheme is missing or is neither of the two web schemes is prefixed with `https://`.

### EV-RULE-005

**The website of an event must belong to the company of the event.** Checked when the website link is set. Message: *"The website must be from the same company as the event."*

### EV-RULE-006

**Lowering the seat maximum below the seats already taken is allowed but warned about.** While the user edits `seats_max` in the form, and the event limits seats, and the new maximum is non-zero, and the resulting available seats are zero or less, and (for a multi-slot event) at least one slot exists, a non-blocking warning is shown, titled *"Update the limit of registrations?"* with the body *"There are more registrations than this limit, the event will be sold out and the extra registrations will remain."* Saving is not prevented: the existing registrations are kept and the event simply becomes sold out.

### EV-RULE-007

**Registrations are open only when every one of these conditions holds.**

```formula
event.kanban_state <> "cancel"
AND event_registrations_started
AND (date_end, read in date_tz, >= now, read in date_tz)
AND (NOT seats_limited OR seats_max = 0 OR seats_available > 0)
AND (
      NOT is_multi_slots
        AND (there is no ticket OR at least one ticket has sale_available)
      OR
      is_multi_slots
        AND event_slot_count > 0
        AND (there is no ticket
             OR at least one ticket is launched, not expired, and has at least one slot
                whose (slot, ticket) availability is unlimited or strictly positive)
    )
```

`event_registrations_started` is true when no ticket defines a start of sale, and otherwise when the earliest such start, read in the display time zone, is at or before the current moment read in the same zone.

### EV-RULE-008

**An event is sold out when its own seats are exhausted, or when every sellable combination is exhausted.**

```formula
event_registrations_sold_out =
     (seats_limited AND seats_max > 0 AND seats_available <= 0)
  OR (there is at least one ticket AND
        multi-slot event  → no (slot, ticket) combination has an unlimited
                            or strictly positive availability
        single-slot event → every ticket has is_sold_out = true
     )
```

The event maximum and the sum of the ticket maximums are deliberately independent: an event may cap 20 seats while offering a 20-seat ticket A and a 20-seat ticket B, and it may also leave its own cap empty while capping each ticket.

### EV-RULE-009

**Multi-company visibility.** Three record rules restrict reading:

| Entity | Condition |
|---|---|
| Event | `company_id IN (companies of the reader) OR company_id IS NULL` |
| Event Registration | `company_id IN (companies of the reader) OR company_id IS NULL` |
| Event Ticket | `event_id.company_id IN (companies of the reader) OR event_id.company_id IS NULL` |
| Event Sales Report | `company_id IN (companies of the reader) OR company_id IS NULL` |

An event without a company is therefore visible to every company.

### EV-RULE-010

**The organiser and the venue must belong to the company of the event, or to no company.** Enforced by the standard company-consistency check on the two contact links.

---

## Time slots

### EV-RULE-011

**A slot hour must lie inside a day.** Both `start_hour` and `end_hour` must satisfy `0 <= hour <= 23.99`. Message: *"A slot hour must be between 0:00 and 23:59."*

### EV-RULE-012

**A slot must end after it starts.** `end_hour` must be strictly greater than `start_hour`. Message: *"A slot end hour must be later than its start hour."* followed by a new line and the display name of the slot.

### EV-RULE-013

**A slot must lie inside the time range of its event.** Checked from both sides.

- On the slot, when its date or hours change: the computed start and the computed end must both satisfy `event.date_begin <= value <= event.date_end`. Message: *"A slot cannot be scheduled outside of its event time range."* then a blank line, then `Event:` and a tab and the event start and end formatted in medium format in the display time zone separated by ` - `, then a new line, then `Slot:` and a tab and the display name of the slot.
- On the event, when its dates, its slot list or its multi-slot flag change: for every multi-slot event, the **earliest** slot start and the **latest** slot end must both lie within `[date_begin, date_end]`. Message: *"These events cannot have slots scheduled outside of their time range:"* followed by one line `- <event name>` per offending event.

### EV-RULE-014

**A slot with registrations cannot be deleted.** Message: *"The following slots cannot be deleted while they have one or more registrations linked to them:"* followed by one line `- <slot display name>` per slot.

---

## Tickets

### EV-RULE-015

**The sales window of a ticket must be coherent.** When both `start_sale_datetime` and `end_sale_datetime` are set, the start must not be after the end. Message: *"The stop date cannot be earlier than the start date. Please check ticket <ticket name>"*

### EV-RULE-016

**The per-order limit of a ticket is bounded.** Checked in this order on every change of `limit_max_per_order` or `seats_max`:

1. When the ticket caps its seats and the per-order limit exceeds that cap: *"The limit per order cannot be greater than the maximum seats number. Please check ticket <ticket name>"*
2. When the per-order limit exceeds the absolute ceiling of **30** tickets per order: *"The limit per order cannot be greater than 30. Please check ticket <ticket name>"*
3. When the per-order limit is negative: *"The limit per order must be positive. Please check ticket <ticket name>"*

A limit of zero switches the rule off and lets the absolute ceiling of 30 apply.

### EV-RULE-017

**A ticket with registrations cannot be deleted.** Message: *"The following tickets cannot be deleted while they have one or more registrations linked to them:"* followed by one line `- <ticket name>` per ticket.

---

## Questions

### EV-RULE-018

**A default question must be reusable.** Database check: `is_default IS DISTINCT FROM TRUE OR is_reusable IS TRUE`. Message: *"A default question must be reusable."* The reusable flag is also forced to true for every default question when it is computed.

### EV-RULE-019

**The type of an answered question cannot change.** When a write would change `question_type` on a question that already has at least one recorded answer, the write is refused. Message: *"You cannot change the question type of a question that already has answers!"*

### EV-RULE-020

**An answered question cannot be deleted, and a default question cannot be deleted at all.**

- With at least one recorded answer: *"You cannot delete a question that has already been answered by attendees. You can archive it instead."*
- When the question is one of the shipped default questions: *"You cannot delete a default question."*
- A suggested answer that has already been chosen by an attendee cannot be deleted either: *"You cannot delete an answer that has already been selected by attendees."*

---

## Registrations

### EV-RULE-021

**A slot must belong to the event, and a multi-slot event requires a slot.** Two messages, checked on every change of `event_id` or `event_slot_id`:

- *"Invalid event / slot choice"* when the chosen slot belongs to another event;
- *"Slot choice is mandatory on multi-slots events."* when the event uses slots and no slot is chosen.

Changing the event in the form automatically clears a slot belonging to another event.

### EV-RULE-022

**A ticket must belong to the event.** Message: *"Invalid event / ticket choice"*. Changing the event in the form automatically clears a ticket belonging to another event.

### EV-RULE-023

**An answer must carry a value.** Database check on Event Registration Answer: `value_answer_id IS NOT NULL OR COALESCE(value_text_box, '') <> ''`. Message: *"There must be a suggested value or a text value."*

### EV-RULE-024

**An answer may only reference a question of its event.** The selectable questions of an answer are restricted to the questions linked to the event of its registration.

### EV-RULE-025

**Barcodes are unique across the whole system.** Database constraint `unique(barcode)` on Event Registration. Message: *"Barcode should be unique"*. The default value is the decimal text of a pseudo-random eight-byte number, and the barcode is **not** carried over when a registration is duplicated.

### EV-RULE-026

**The public registration form may write only eight fields on a registration:** `name`, `phone`, `email`, `company_name`, `event_id`, `partner_id`, `event_slot_id`, `event_ticket_id`. Any other posted field is silently ignored.

### EV-RULE-027

**A ticket posted to the public registration form must be sellable.** Every posted ticket identifier must belong to the event, and its ticket must be launched and not expired. Otherwise the request is refused with *"This ticket is not available for sale for this event"*. A ticket identifier of zero is accepted only when the event has no ticket at all.

### EV-RULE-028

**Contact details are filled from the contact but never overwritten.** `name`, `email`, `phone` and `company_name` take their value from the **contact address** of the chosen contact only while they are empty. A value typed by a user or posted by a form always wins, and writing a registration never writes back on the contact.

### EV-RULE-029

**Telephone numbers are reformatted against a country, best effort.** On creation, a supplied telephone number is formatted for the first country found among: the country of the chosen contact, the country of the event, the country of the active company. When formatting fails, the raw value is kept unchanged. The same rule is applied while editing the form whenever `phone`, `event_id` or `partner_id` changes.

### EV-RULE-030

**Seats are verified for every combination of slot and ticket, and overbooking is refused.**

Scope: the check runs whenever `active`, `state`, `event_id`, `event_slot_id` or `event_ticket_id` changes on a registration, and it is also called explicitly before creating registrations from the public site and before accepting an online payment. Only **active** registrations in state `open` or `done` are counted.

Procedure:

1. Group the registrations being verified by event.
2. Inside each event, group them by the pair `(event_slot, event_ticket)` and count them.
3. For each pair, compute the availability:
   - start with "no limit";
   - when the event limits seats and has a non-zero maximum, the availability becomes the available seats of the slot when a slot is given, and the available seats of the event otherwise;
   - when the availability is not already zero and a ticket is given and that ticket has a non-zero maximum, the ticket availability is `ticket.seats_max − (registrations of that slot and that ticket)` when a slot is given, and `ticket.seats_available` otherwise; the final availability is the ticket availability when there was no limit yet, and the smaller of the two otherwise.
4. A pair whose availability is "no limit" always passes. A pair whose availability is strictly smaller than the requested count fails, and contributes a line *"<name>: missing <n> seat(s)"* where `n = count − availability` and the name is:
   - `<ticket name> - <slot display name>` when both are given;
   - the slot display name when only a slot is given;
   - the ticket name when only a ticket is given;
   - the event name when neither is given.
5. When at least one pair failed, the operation is rejected with *"There are not enough seats available for <event name>:"* followed by a new line and the failing lines, one per line.

The count passed for a plain state change or an un-archiving is **zero**, which turns the check into "the current figures must not already overflow the limits".

**Worked example.** An event caps 100 seats, has no ticket and no slot; 60 registrations are `open`, 10 are `done`, 5 are `cancel`. Then `seats_reserved = 60`, `seats_used = 10`, `seats_taken = 70`, `seats_available = 100 − 70 = 30`. A request for 31 seats fails with *"There are not enough seats available for <event name>:"* and the line *"<event name>: missing 1 seat(s)"*. A request for 30 seats passes and leaves zero seats available, which makes the event sold out.

### EV-RULE-031

**Deleting an order deletes its registrations; deleting a registration never deletes the order.** `sale_order_id`, `sale_order_line_id` and `pos_order_line_id` all cascade from the order side to the registration. `event_id`, `event_slot_id` and `event_ticket_id` restrict, that is they prevent the deletion of a used event, slot or ticket.

---

## Automatic communications

### EV-RULE-032

**A schedule must reference a template of the model matching its notification type.** The reference is required. `notification_type` is derived from the target model: an Email Template gives `mail`, a Text Message Template gives `sms`. Before a run, a schedule whose reference points at the wrong model, or at a record that no longer exists, is dropped from that run and a warning is written to the technical log; the schedule is neither marked done nor marked in error.

### EV-RULE-033

**Deleting a template deletes the schedules that used it.** Deleting an Email Template or a Text Message Template also deletes, with elevated rights, every Event Automated Mailing and every template communication line whose reference pointed at it. Deleting an Email Template additionally cascades through the reference itself.

### EV-RULE-034

**The scheduler only picks live work.** A schedule is executed only when the event is active, the event kanban state is not `cancel`, the due date is at or before the current moment, the schedule is not already done, and either the schedule is not attendee based or the event has not ended.

### EV-RULE-035

**A message scheduled before the start is never sent after the end.** In event-based and slot-based modes the sending is allowed only when the due date has passed **and** either the trigger is not "before the event starts" nor "after the event started", or the event (respectively the slot) has not ended yet.

### EV-RULE-036

**Attendee-based messages ignore the mass-mailing exclusion list; global messages honour it.** Registering to an event is treated as an explicit subscription to the messages of that event, including the one that carries the ticket.

### EV-RULE-037

**A failing schedule is reported at most once per hour.** On a failure, when `error_datetime` is empty or older than one hour, a message is posted on the event, addressed to the organiser, the event responsible and the last author of the template, restricted to the active contacts among them, and queued rather than sent immediately. Its body is assembled from the three parts listed in [workflows.md](workflows.md#step-4-failure). Then `error_datetime` is stamped. A later successful run clears `error_datetime`.

### EV-RULE-038

**Traces of attendees that fell back to `draft` or `cancel` are deleted, not sent.** During an attendee-based batch, every trace whose registration is no longer `open` or `done` is deleted before the batch is sent.

### EV-RULE-039

**A slot-based schedule creates its per-slot traces lazily.** The first time a global schedule runs on a multi-slot event, one trace is created for every slot that has none. Each trace keeps its own due date, its own resume point, its own counter and its own done flag.

### EV-RULE-040

**Two configuration parameters govern batching, and one governs synchronous execution.** `mail.batch_size` (default 50 when missing or zero) is the number of attendees processed between two commits. `mail.render.cron.limit` (default 1000 when missing or zero) is the number of attendees processed in one run of one schedule before the job re-wakes itself. `event.event_mail_async`, when set, makes a new registration merely wake the jobs instead of sending its attendee-based messages immediately.

---

## Products and prices

### EV-RULE-041

**A product used by a ticket must have its service tracking set to the event value.** Message: *"Products linked to an event ticket must have "<service tracking label>" set to "<Event Registration>"."* The two placeholders are the readable label of the service-tracking field and the readable label of its event value.

### EV-RULE-042

**A product used by a booth category must have its service tracking set to the booth value.** Message: *"The product, <product name> , is used for Event Booth, it must have service_tracking set to "Event Booth"."*

### EV-RULE-043

**The service tracking of a product already used by a booth category cannot be changed.** Two messages, one per level:

- On a variant: *"You cannot change the service_tracking of the product <product name> because it is already assigned to <booth category name>. The service_tracking must remain 'event_booth'."*
- On a product template whose variant is used: *"The "service_tracking" for the product template, <product name> cannot be changed because one of its variants is assigned to the Event Booth Category, <booth category name>. The service_tracking must remain "Event Booth"."*

### EV-RULE-044

**Event and booth products invoice on ordered quantities.** Selecting the event or the booth value for the service tracking of a product sets its invoicing policy to "ordered quantities" in the form.

### EV-RULE-045

**Event and booth products may be sold at price zero online.** Both service-tracking values are added to the list of product kinds for which a zero price is acceptable in the storefront.

### EV-RULE-046

**Event and booth products are excluded from the product catalogue of a sales order.** The catalogue picker filters out products whose service tracking is the event value or the booth value, because those lines must be created through their configurator.

### EV-RULE-047

**A pricelist rule with a positive minimum quantity does not apply to ticket products.** While editing such a rule, a non-blocking warning titled `Warning` is shown:

- when the rule applies globally or to a product category: *"A pricelist item with a positive min. quantity will not be applied to the event tickets products."*
- when the rule applies to a product template or a variant whose service tracking is the event value: *"A pricelist item with a positive min. quantity cannot be applied to this event tickets product."*

### EV-RULE-048

**Anonymous visitors may read the image of an unpublished ticket product.** The image fields of a product variant in the sizes 1920, 1024, 512, 256 and 128 are readable by anyone when that variant is used by at least one event ticket, even if the product is not published. When such a product has no image, the placeholder used is the event ticket placeholder rather than the generic one.

### EV-RULE-049

**A product used by a ticket or by a booth category cannot be deleted.** The product link of an
Event Template Ticket, of an Event Ticket and of an Event Booth Category is required, and a required
link refuses the deletion of its target. Deleting such a product is therefore refused by the
platform with its standard referential message naming the records that still point at it. To retire
a ticket product, archive it instead: an archived product makes every ticket that uses it
unavailable for sale (`sale_available` becomes false) without destroying anything.

---

## Selling tickets on a sales order

### EV-RULE-050

**The configurator refuses an incoherent choice.** When the chosen ticket does not belong to the chosen event, or the chosen slot does not belong to it, the wizard is refused with one line per problem, joined by new lines:

- *"Invalid ticket choice "<ticket name>" for event "<event name>"."*
- *"Invalid slot choice "<slot name>" for event "<event name>"."*

### EV-RULE-051

**An order line selling a ticket product must name an event, a ticket, and a slot when the event uses slots.** Message: *"The sale order line with the product <product name> needs an event, a ticket and a slot in case the event has multiple time slots."*

### EV-RULE-052

**The price of a ticket line comes from the ticket, not from the product.** When the line carries both an event and a ticket, the display price is the ticket price converted into the currency of the order line, using the company of the ticket (or the active company): the **discounted** ticket price read in the pricing context of the line when the matching pricelist rule is not allowed to show a discount, and the plain ticket price when it is. The same rule applies to booths with the sum of the category prices.

### EV-RULE-053

**A sales order carrying event lines cannot be confirmed while a line is unconfigured.** Message: *"Please make sure all your event related lines are configured before confirming this order:"* followed by one line `- <line description>` per offending line. The booth equivalent is *"Please make sure all your event-booth related lines are configured before confirming this order:"* with the same list.

### EV-RULE-054

**Confirming an order tops the registrations up to the ordered quantity.** For each event line, `ordered quantity − number of non-cancelled registrations of that line` new registrations are created. The count is therefore idempotent: confirming twice does not double the attendees, while confirming after a cancellation recreates the missing seats.

### EV-RULE-055

**Paid seats of a single confirmed order start unconfirmed.** When exactly one order is being confirmed and the line total including tax is not zero, the new registrations are created in state `draft`, so that the attendee details can be captured before the seats are taken. In every other case the state derivation applies immediately, which makes them `open`.

### EV-RULE-056

**Changing the customer of an order rewrites the contact of its registrations.** Every registration whose sales order is among the written orders receives the new customer, with elevated rights, whenever the order carries at least one event line.

### EV-RULE-057

**Changing the slot or the ticket of a registration attached to an order raises a warning activity.** The activity is scheduled on the sales order, assigned to the event responsible, or failing that the salesperson of the order, or failing that the administrator, and describes the move from the old display name to the new one, naming the changed record as `Ticket` or `Slot`.

### EV-RULE-058

**The unit of measure of an event line is read-only.** Any order line carrying an event has its unit-of-measure field locked.

### EV-RULE-059

**A configured line description is never overwritten by the product template description.** For a line carrying a ticket or pending booths, the default description of the product template is not applied; the ticket or booth description computed by the domain wins.

---

## The online shop

### EV-RULE-060

**Adding tickets to a cart is capped by the remaining seats.** Let `existing` be the current quantity of the matched line (zero when there is none) and `added = requested − existing`. Let `availability` be the `(slot, ticket)` availability when a slot is given, and the ticket available seats otherwise.

- When the ticket limits its seats and `availability <= 0`: the quantity stays at `existing` and the message is *"Sorry, The <ticket name> tickets for the <event name> event are sold out."*
- When the ticket limits its seats and `added > availability`: the quantity becomes `existing + availability` and the message is *"Sorry, only <availability> seats are still available for the <ticket name> ticket for the <event name> event<, on <slot name> when a slot is given>."*

### EV-RULE-061

**The quantity of a ticket line can never be raised by hand from the cart.** When a cart update without a ticket identifier would raise the quantity of a line that carries a ticket, the quantity is kept and the message is *"You cannot raise manually the event ticket quantity in your cart"*.

### EV-RULE-062

**A cart line is reused only for the exact same combination.** For tickets, an existing line is matched only when its slot and its ticket are both identical to the requested ones, which produces one line per `(slot, ticket)` pair. For booths, an existing line is matched only when it already contains one of the requested booths.

### EV-RULE-063

**Seats are verified again before a payment is accepted.** The non-cancelled ticket registrations of the order are grouped by event, then by `(slot, ticket)`, and `EV-RULE-030` is applied with those counts. A failure refuses the transaction.

### EV-RULE-064

**Abandoned-cart reminders skip carts whose tickets are no longer sellable.** A cart is eligible only when every ticket it carries is still available for sale.

### EV-RULE-065

**A ticket-only order does not require a full billing address.** When every line of the order carries a ticket, the full address step is skipped unless the configuration parameter `website_event_sale.require_billing_details_for_events` is set to a true value.

### EV-RULE-066

**A ticket order whose tickets are all free is confirmed without checkout.** When every line of the cart carries a ticket whose price is zero (compared with two decimal places), the order is confirmed immediately, the cart is reset and the visitor is sent to the order confirmation page. When the visitor chose only free tickets and no cart exists at all, no order is created: the registrations are created directly.

### EV-RULE-067

**Lowering the quantity of a ticket cart line cancels the surplus seats.** The non-cancelled registrations of that order, that slot and that ticket are ordered by creation date ascending; the first `new quantity` are kept and the next `old quantity − new quantity` are cancelled.

### EV-RULE-068

**A booth configuration must name at least one booth.** The booth configurator refuses to close
while its booth list is empty. Message: *"You have to select at least one booth."* The same
configurator clears the chosen category whenever the event changes, and clears the chosen booths
whenever the event or the category changes, so an empty list is the normal state after either
change.

---

## Booths

### EV-RULE-070

**A booth category always names a product.** The product is required and defaults to the shipped Event Booth product. The category price defaults to `product sales price + product extra price` whenever the product has a non-zero sales price, and stays editable, because one product may serve several categories at different prices.

### EV-RULE-071

**All reservations of one order line must belong to a single event.** Message: *"Registrations from the same Order Line must belong to a single event."*

### EV-RULE-072

**A booth linked to a sales order cannot be deleted.** Message: *"You can't delete the following booths as they are linked to sales orders: <comma separated booth names>"*

### EV-RULE-073

**One reservation per order line and booth.** Database constraint `unique(sale_order_line_id, event_booth_id)` with the message *"There can be only one registration for a booth by sale order line"*.

### EV-RULE-074

**A booth must still be available when the order that reserves it is confirmed.** Message: *"The following booths are unavailable, please remove them to continue : "* followed by one line per booth, each indented and prefixed with `- `.

### EV-RULE-075

**Confirming a booth cancels the competing reservations and their orders.** Every other pending reservation on the same booths is handled as follows: a message is posted on its order, addressed to the salesperson of that order, reading *"Your order has been cancelled because the following booths have been reserved"* followed by a bulleted list of the booth display names; the order is then cancelled and the reservation is deleted.

### EV-RULE-076

**The public booth form validates three situations before booking.** The answers are error codes, not sentences, because the page renders its own text:

| Code | Situation |
|---|---|
| `boothError` | the requested booths are not exactly the booths of this event that are still available, or they span more than one category |
| `boothCategoryError` | the requested category does not exist (only checked when online booth sales are installed) |
| `existingPartnerError` | the visitor is anonymous and the posted electronic mail address already belongs to a known contact |

### EV-RULE-077

**A booth line always sells exactly one unit.** Raising the quantity of a booth line above one is refused with *"You cannot manually change the quantity of an Event Booth product."* and the quantity is forced back to one.

### EV-RULE-078

**Paying the invoice stamps the booths as paid.** When an invoice whose lines come from sales order lines is paid, every confirmed booth of those lines receives `is_paid = true`.

### EV-RULE-079

**Booking a booth of a sponsoring category creates or reuses a sponsor.** The sponsor is looked up by the quadruple (contact, sponsorship level, sponsor kind, event); when none matches, one is created with those four values plus every value posted with a `sponsor_` prefix, and with the contact name as the sponsor name when none was posted.

---

## The programme

### EV-RULE-080

**The three stage flags of a talk stage form a ladder.** A cancelling stage is neither visible in the agenda nor fully accessible; a fully accessible stage is always visible in the agenda; a stage that is not visible in the agenda is never fully accessible. Entering a fully accessible stage publishes the talk; entering a cancelling stage unpublishes it.

### EV-RULE-081

**Talk tag names are unique.** Database constraint `unique(name)` on Event Track Tag. Message: *"Tag name already exists!"* A tag whose colour index is zero or empty is never offered on the public site.

### EV-RULE-082

**A quiz question must have exactly one correct answer and at least one incorrect answer.** Two messages:

- *"Question "<question name>" must have 1 correct answer to be valid."* when the number of correct answers is not exactly one;
- *"Question "<question name>" must have 1 correct answer and at least 1 incorrect answer to be valid."* when the question has fewer than two answers.

### EV-RULE-083

**A quiz submission must cover every question exactly once.** The submitted answers are read restricted to the quiz of the talk; when the number of distinct questions they cover differs from the number of questions of the quiz, the submission is refused with the code `quiz_incomplete`. A second submission by the same visitor is refused with the code `track_quiz_done`.

### EV-RULE-084

**Resetting a quiz requires either unlimited tries or the administrator role.** Any other reader is forbidden.

### EV-RULE-085

**A talk reminder is only sent for an upcoming, visible talk to a valid address.** Refusals, in this order: *"Invalid data."* when the talk is not visible to the reader; *"Invalid email."* when the address does not normalise; *"The talk is already finished."* when the talk or its event has finished; *"The talk has already begun."* when the talk is not upcoming.

### EV-RULE-086

**A talk proposal with a separate contact block needs a contact address or a telephone.** Otherwise the answer is the code `invalidFormInputs`. A proposal whose event cannot be reached from the current website gives the code `forbidden`.

### EV-RULE-087

**A talk proposal may only carry existing, readable tags.** The posted tag identifiers are passed through a search, which silently drops unknown tags and, for an anonymous visitor, colourless tags.

### EV-RULE-088

**"Not in" searches on visitor wish lists are refused.** Two messages: *"Unsupported 'Not In' operation on track wishlist visitors"* when searching talks by wish-listing visitor or visitors by wish-listed talk, and *"Unsupported 'Not In' operation on visitors registrations"* when searching visitors by registered event.

### EV-RULE-089

**A talk is shown in the public agenda when it is published or when its stage is visible in the agenda; an anonymous visitor additionally requires publication.** Readers who belong to the Registration Desk group see both published and unpublished talks whose stage is visible in the agenda.

---

## Access, website and cross-cutting rules

### EV-RULE-090

**The name of the event application on the website is required.** Message: *""Events App Name" field is required."* It defaults to `<website name> Events`.

### EV-RULE-091

**The static-map signing secret must be a valid base-encoded value.** Checked on creation and whenever the secret is written. Message: *"Please enter a valid base64 secret"*. Switching the static-map option off clears both the key and the secret.

### EV-RULE-092

**Only an Event Administrator may regenerate the leads of an event.** Message: *"Only Event Managers are allowed to re-generate all leads."*

### EV-RULE-093

**One lead-generation request per event at a time.** Database constraint `unique(event_id)` on Event Lead Request. Message: *"You can only have one generation request per event at a time."*

### EV-RULE-094

**Lead generation never duplicates a lead for the same rule and the same attendee.** Before applying a rule, every existing lead of that rule linked to any of the candidate registrations, **including archived leads**, is read and the registrations it already covers are excluded.

### EV-RULE-095

**A visitor who registered to an event, or wish-listed a talk, is never purged.** The housekeeping that removes inactive visitors excludes visitors having at least one registration and visitors having at least one talk link.

### EV-RULE-096

**Merging two visitors moves their registrations and their talk links to the surviving visitor.** Registrations and talk links without a contact additionally receive the contact of the surviving visitor. The display name of an anonymous visitor with registrations is the attendee name of its latest registration; its electronic mail address and mobile number are taken from the earliest of its registrations that carries one.

### EV-RULE-097

**Rounding rules.** Monetary comparisons on an order total use the rounding of the currency of that order. Ticket and booth prices are carried with the product price precision. Seat counts are whole numbers and are never rounded. Durations of talks and opening hours of sponsors are fractional hours, where `0.5` means thirty minutes and `8.25` means eight fifteen.

### EV-RULE-098

**Date and time zone rules.**

- Every datetime is stored in coordinated universal time.
- The back-office form of an event shows `date_begin` and `date_end` in the time zone of the reader.
- The public page, the printed ticket, the badge, the agenda, the calendar files and the automatic communications all show the dates in `date_tz`, the display time zone of the event.
- Slot hours are entered and displayed in `date_tz`.
- Sponsor opening hours are read in `date_tz`.
- The comparisons that decide whether registrations are open, whether a ticket is launched and whether a ticket is expired are all made after converting both sides into `date_tz`.
- The comparisons that decide `is_ongoing`, `is_done`, `start_today`, `start_remaining` and every talk time flag are made in coordinated universal time, because they only involve differences.

### EV-RULE-099

**Registration Desk users may post messages on an event they may only read.** A member of the Registration Desk group is allowed to create a message on an Event with read access only. This exception does not extend to anonymous or portal readers, who can read published events but never post on them.

---

## Rule identifier index

Every rule of this file, in order, with the property it protects.

| Rule | Subject |
|---|---|
| `EV-RULE-001` | An event always has a name, a start date, an end date, a display time zone and a seat-limit flag |
| `EV-RULE-002` | The end date cannot precede the start date |
| `EV-RULE-003` | The online-event link must be a complete web address |
| `EV-RULE-004` | The online-event link belongs to events without a venue |
| `EV-RULE-005` | The website of an event must belong to the company of the event |
| `EV-RULE-006` | Lowering the seat maximum below the seats already taken is allowed but warned about |
| `EV-RULE-007` | Registrations are open only when every one of these conditions holds |
| `EV-RULE-008` | An event is sold out when its own seats are exhausted, or when every sellable combination is exhausted |
| `EV-RULE-009` | Multi-company visibility |
| `EV-RULE-010` | The organiser and the venue must belong to the company of the event, or to no company |
| `EV-RULE-011` | A slot hour must lie inside a day |
| `EV-RULE-012` | A slot must end after it starts |
| `EV-RULE-013` | A slot must lie inside the time range of its event |
| `EV-RULE-014` | A slot with registrations cannot be deleted |
| `EV-RULE-015` | The sales window of a ticket must be coherent |
| `EV-RULE-016` | The per-order limit of a ticket is bounded |
| `EV-RULE-017` | A ticket with registrations cannot be deleted |
| `EV-RULE-018` | A default question must be reusable |
| `EV-RULE-019` | The type of an answered question cannot change |
| `EV-RULE-020` | An answered question cannot be deleted, and a default question cannot be deleted at all |
| `EV-RULE-021` | A slot must belong to the event, and a multi-slot event requires a slot |
| `EV-RULE-022` | A ticket must belong to the event |
| `EV-RULE-023` | An answer must carry a value |
| `EV-RULE-024` | An answer may only reference a question of its event |
| `EV-RULE-025` | Barcodes are unique across the whole system |
| `EV-RULE-026` | The public registration form may write only eight fields on a registration: |
| `EV-RULE-027` | A ticket posted to the public registration form must be sellable |
| `EV-RULE-028` | Contact details are filled from the contact but never overwritten |
| `EV-RULE-029` | Telephone numbers are reformatted against a country, best effort |
| `EV-RULE-030` | Seats are verified for every combination of slot and ticket, and overbooking is refused |
| `EV-RULE-031` | Deleting an order deletes its registrations; deleting a registration never deletes the order |
| `EV-RULE-032` | A schedule must reference a template of the model matching its notification type |
| `EV-RULE-033` | Deleting a template deletes the schedules that used it |
| `EV-RULE-034` | The scheduler only picks live work |
| `EV-RULE-035` | A message scheduled before the start is never sent after the end |
| `EV-RULE-036` | Attendee-based messages ignore the mass-mailing exclusion list; global messages honour it |
| `EV-RULE-037` | A failing schedule is reported at most once per hour |
| `EV-RULE-038` | Traces of attendees that fell back to `draft` or `cancel` are deleted, not sent |
| `EV-RULE-039` | A slot-based schedule creates its per-slot traces lazily |
| `EV-RULE-040` | Two configuration parameters govern batching, and one governs synchronous execution |
| `EV-RULE-041` | A product used by a ticket must have its service tracking set to the event value |
| `EV-RULE-042` | A product used by a booth category must have its service tracking set to the booth value |
| `EV-RULE-043` | The service tracking of a product already used by a booth category cannot be changed |
| `EV-RULE-044` | Event and booth products invoice on ordered quantities |
| `EV-RULE-045` | Event and booth products may be sold at price zero online |
| `EV-RULE-046` | Event and booth products are excluded from the product catalogue of a sales order |
| `EV-RULE-047` | A pricelist rule with a positive minimum quantity does not apply to ticket products |
| `EV-RULE-048` | Anonymous visitors may read the image of an unpublished ticket product |
| `EV-RULE-049` | A product used by a ticket or by a booth category cannot be deleted |
| `EV-RULE-050` | The configurator refuses an incoherent choice |
| `EV-RULE-051` | An order line selling a ticket product must name an event, a ticket, and a slot when the event uses slots |
| `EV-RULE-052` | The price of a ticket line comes from the ticket, not from the product |
| `EV-RULE-053` | A sales order carrying event lines cannot be confirmed while a line is unconfigured |
| `EV-RULE-054` | Confirming an order tops the registrations up to the ordered quantity |
| `EV-RULE-055` | Paid seats of a single confirmed order start unconfirmed |
| `EV-RULE-056` | Changing the customer of an order rewrites the contact of its registrations |
| `EV-RULE-057` | Changing the slot or the ticket of a registration attached to an order raises a warning activity |
| `EV-RULE-058` | The unit of measure of an event line is read-only |
| `EV-RULE-059` | A configured line description is never overwritten by the product template description |
| `EV-RULE-060` | Adding tickets to a cart is capped by the remaining seats |
| `EV-RULE-061` | The quantity of a ticket line can never be raised by hand from the cart |
| `EV-RULE-062` | A cart line is reused only for the exact same combination |
| `EV-RULE-063` | Seats are verified again before a payment is accepted |
| `EV-RULE-064` | Abandoned-cart reminders skip carts whose tickets are no longer sellable |
| `EV-RULE-065` | A ticket-only order does not require a full billing address |
| `EV-RULE-066` | A ticket order whose tickets are all free is confirmed without checkout |
| `EV-RULE-067` | Lowering the quantity of a ticket cart line cancels the surplus seats |
| `EV-RULE-068` | A booth configuration must name at least one booth |
| `EV-RULE-070` | A booth category always names a product |
| `EV-RULE-071` | All reservations of one order line must belong to a single event |
| `EV-RULE-072` | A booth linked to a sales order cannot be deleted |
| `EV-RULE-073` | One reservation per order line and booth |
| `EV-RULE-074` | A booth must still be available when the order that reserves it is confirmed |
| `EV-RULE-075` | Confirming a booth cancels the competing reservations and their orders |
| `EV-RULE-076` | The public booth form validates three situations before booking |
| `EV-RULE-077` | A booth line always sells exactly one unit |
| `EV-RULE-078` | Paying the invoice stamps the booths as paid |
| `EV-RULE-079` | Booking a booth of a sponsoring category creates or reuses a sponsor |
| `EV-RULE-080` | The three stage flags of a talk stage form a ladder |
| `EV-RULE-081` | Talk tag names are unique |
| `EV-RULE-082` | A quiz question must have exactly one correct answer and at least one incorrect answer |
| `EV-RULE-083` | A quiz submission must cover every question exactly once |
| `EV-RULE-084` | Resetting a quiz requires either unlimited tries or the administrator role |
| `EV-RULE-085` | A talk reminder is only sent for an upcoming, visible talk to a valid address |
| `EV-RULE-086` | A talk proposal with a separate contact block needs a contact address or a telephone |
| `EV-RULE-087` | A talk proposal may only carry existing, readable tags |
| `EV-RULE-088` | "Not in" searches on visitor wish lists are refused |
| `EV-RULE-089` | A talk is shown in the public agenda when it is published or when its stage is visible in the agenda; an anonymous visitor additionally requires publication |
| `EV-RULE-090` | The name of the event application on the website is required |
| `EV-RULE-091` | The static-map signing secret must be a valid base-encoded value |
| `EV-RULE-092` | Only an Event Administrator may regenerate the leads of an event |
| `EV-RULE-093` | One lead-generation request per event at a time |
| `EV-RULE-094` | Lead generation never duplicates a lead for the same rule and the same attendee |
| `EV-RULE-095` | A visitor who registered to an event, or wish-listed a talk, is never purged |
| `EV-RULE-096` | Merging two visitors moves their registrations and their talk links to the surviving visitor |
| `EV-RULE-097` | Rounding rules |
| `EV-RULE-098` | Date and time zone rules |
| `EV-RULE-099` | Registration Desk users may post messages on an event they may only read |

---

## Mapping of former rule identifiers

Two independently written versions of this folder were merged. Version M numbered its rules
`EV-RULE-001` to `EV-RULE-099`; version P carried no rule catalogue at all, only a reading order
that announced one. The consolidated scheme therefore keeps the identifiers of version M unchanged,
so that every citation already written elsewhere in this folder still resolves, and adds two rules
that neither version had numbered.

| Identifier in version P | Identifier in version M | Identifier here | Note |
|---|---|---|---|
| none: version P had no `business-rules.md` | `EV-RULE-001` … `EV-RULE-048` | unchanged | Same rule, same wording, messages moved from code font into quotation marks. |
| none | `EV-RULE-050` … `EV-RULE-067` | unchanged | Same rule. |
| none | `EV-RULE-070` … `EV-RULE-099` | unchanged | Same rule. |
| none | none | `EV-RULE-049` | Added here: a product used by a ticket or a booth category cannot be deleted. Version M described the required product link in `entities.md` but never numbered the consequence. |
| none | none | `EV-RULE-068` | Added here: a booth configuration must name at least one booth. Version M carried the refusal *"You have to select at least one booth."* in `workflows.md` and in `booths-and-exhibitors.md` without a rule number. |
| none | `069` | not used | The number was already unused in version M and is left unused, so that every existing citation keeps its meaning. |

No identifier was reused for a different rule, and no rule of either version was dropped.

---

## Reconciliation notes

1. **Rule scheme.** Only version M carried numbered rules, so the consolidation is a superset: the
   numbering is unchanged and two previously unnumbered refusals were given the free numbers `049`
   and `068`. The mapping table above records that.
2. **Message formatting.** Version M reproduced every message in code font. The documentation rules
   of this repository reserve code font for stored values and identifiers and require quotation
   marks for verbatim user-facing text, so every message in this file is now quoted. The text
   itself, including its irregular spacing, is untouched: for example the refusal
   *"The following booths are unavailable, please remove them to continue : "* keeps the space
   before the colon and the trailing space.
3. **Field identifiers.** Version M used readable substitutes for several storage names. This file
   now names the fields exactly as the database does — `seats_max`, `date_tz`, `event_url`,
   `interval_nbr`, `template_ref`, `limit_max_per_order`, `pos_order_line_id`, `color`,
   `is_in_opening_hours`, `website_cta` and the rest — because a rule that names a field is only
   testable if the name is the real one. The full name of each field is given in
   [`entities.md`](entities.md).
4. **Seat verification.** Both versions agree that the check counts only active registrations in
   state `open` or `done`, and that a plain state change or an un-archiving passes a requested count
   of zero. The source confirms it: the counters are read in one grouped query restricted to those
   two states and to active records, and the effective maximum of a multi-slot event is the seat
   maximum multiplied by the number of slots. `EV-RULE-030` and
   [`calculations.md`](calculations.md#1-seat-counters) state it in the same terms.
5. **The absolute per-order ceiling.** Version M stated thirty tickets per order both as the upper
   bound of the per-order limit and as the fallback limit of the public form. The source confirms a
   single constant used in both places, so `EV-RULE-016` and
   [`calculations.md`](calculations.md#9-per-order-ticket-limits) quote the same number.
