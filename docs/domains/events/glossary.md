# Events — Glossary

Every term this domain uses, defined. Terms are listed alphabetically. Where a reproduced
identifier belongs to the term, it is given in code font; where two vocabularies exist for the same
thing, the alternative wording is recorded so that a reader coming from either one finds the
entry.

Entities of the domain are defined field by field in [`entities.md`](entities.md); this glossary
gives the business meaning only.

---

**Absolute per-order ceiling.** The largest number of tickets one order line may hold, whatever the
configuration: thirty. It bounds the per-order limit of a ticket and is also the fallback limit
offered on the public registration form when nothing else constrains the quantity. See
[`calculations.md`](calculations.md#9-per-order-ticket-limits).

**Action button.** An optional button shown on the public talk page while the talk plays. It has a
title, a target address and a delay in minutes after the start of the talk; it is live from the
start plus the delay until the end of the talk. Also called the magic button.

**Agenda.** The public grid of the programme: one table per calendar day, quarter-hour rows down
the side and rooms across the top, each talk drawn over the rows it spans. A talk without a room
spans every room column.

**Answer.** Two different records carry this word. A **suggested answer** (Event Question Answer)
is one of the choices offered by a selection question. A **recorded answer** (Event Registration
Answer) is what one attendee actually answered to one question, either a chosen suggestion or a
typed text.

**Answer breakdown analysis.** The reporting screen over Event Registration Answer, opened on
pivot and graph, that counts the recorded answers per question and per suggested answer, so that an
organiser can read how the attendees answered each registration question. It is described in
[`interfaces.md`](interfaces.md).

**Attendance.** The fact that an attendee physically turned up. It is recorded by moving the
registration to state `done`, which stamps the closing moment in `date_closed` and logs a note. The
operation that records it is the badge scan of the registration desk.

**Attendee.** The person who occupies a seat. The record that represents the seat is the
Registration; the attendee name, electronic mail address, telephone and company name are fields of
that record and are not necessarily those of the contact who booked it.

**Attendee analysis.** The reporting screen over registrations, opened on graph, pivot, kanban, list
and form, filtered by default on the last thirty days of creation and on taken seats.

**Attendee editor.** The dialogue that opens when a single sales order carrying tickets is
confirmed. It shows one line per seat, pre-filled from the order customer, and creates or updates
the registrations when it is validated. Its records are Edit Attendee Details on Sales Confirmation
and Edit Attendee Line on Sales Confirmation.

**Automatic communication.** A planned message of an event, described by an interval number, an
interval unit, a trigger kind and a reference to an electronic mail template or a text message
template. The record is Event Automated Mailing; the scheduled job walks the due ones and sends
them. Also called a communication schedule.

**Available seats.** The remaining capacity of a scope, computed as the effective maximum of that
scope minus the taken seats, and reported as zero when the scope has no maximum at all. A zero
figure therefore means "no limit" as often as it means "full", which is why every decision tests
the limit flag before the figure.

**Badge.** The printed card an attendee wears, produced in one of three layouts: A4 foldable, A6, or
four per sheet. It carries the event name, the dates in the display time zone, the venue, the
attendee name and company, the organiser logo, a quick response code of the barcode, the linear
barcode when that option is on, and a coloured strip with the ticket name.

**Barcode.** The code printed on the badge and read at the registration desk. It is generated as the
decimal text of a pseudo-random eight-byte number, is unique across the whole database, and is not
carried over when a registration is duplicated.

**Booth.** One rentable piece of floor space at an event, belonging to one booth category, with a
two-value availability state. Called a **stand** in some vocabularies; the two words mean the same
thing here.

**Booth category.** A class of booth sharing a description, a picture, a product, a price and the
sponsor options. Three are shipped: Standard Booth, Premium Booth and Very Important Person Booth.

**Booth configurator.** The dialogue that opens when a booth product is put on a sales order line.
It asks for the event, then the booth category, then the booths.

**Booth reservation.** A pending claim on a booth carried by one sales order line. Several customers
may hold a claim on the same booth at the same time; the first order to be confirmed wins the booth
and every competing claim is destroyed with its order. The record is Event Booth Registration.

**Cancelling stage.** A talk review stage flagged as a refusal or a cancellation. A talk entering
such a stage is unpublished, and the stage can be neither visible in the agenda nor fully
accessible.

**Capability package.** A named unit of behaviour that can be installed or left out. The domain is
delivered as a core package plus bridges to the selling, counter, website, programme, exhibitor and
lead-generation capabilities; a bridge only adds behaviour and never removes any.

**Communication trace.** The record that says a given attendee is due to receive, or has received, a
given attendee-based message: Registration Mail Scheduler. Its slot-based counterpart, which lets
one global schedule fire once per time slot, is Slot Mail Scheduler.

**Contact address.** The address a contact designates as the one to write to, which for a company is
usually a child record. Attendee values copied from a contact are read from the contact address, not
from the company record.

**Display time zone.** The time zone in which the dates of an event are shown publicly, on printed
tickets, in communications, in the agenda and in the calendar files. Slot hours and sponsor opening
hours are expressed in it. It is `date_tz` on the event.

**Effective maximum.** The capacity a scope really has: the seat maximum of a single-slot event, the
seat maximum multiplied by the number of slots for a multi-slot event, the seat maximum of the event
for each slot taken separately, and the seat maximum of a ticket for that ticket.

**Ending stage.** An event stage flagged as the end of the pipeline. The housekeeping pass moves
every event whose end date has passed into the first ending stage by sequence.

**Event.** The occasion itself: a name, a start and an end, a display time zone, a venue or an online
address, a seat policy, a ticket catalogue, a question set, a communication schedule, a booth
catalogue, a programme and a public page.

**Event configurator.** The dialogue that opens when a ticket product is put on a sales order line.
It asks for the event, the slot when the event uses slots, and the ticket.

**Event stage.** One position of the ordered pipeline an event travels through. A stage carries a
sequence, a description, a folding marker for the pipeline view and an ending marker. Stages are
free master data: an organisation may rename them, reorder them and add its own. See
[`state-machines.md`](state-machines.md).

**Event template.** A reusable bundle of defaults — seat limitation, time zone, tickets, booths,
communications, questions, tags, note, ticket instructions and website switches — applied to an
event when its template link is set or changed. Applying a template never destroys anything that is
already in use.

**Exhibitor.** A sponsor whose kind is `exhibitor` or `online`, which is what makes it appear in the
public exhibitor list. A sponsor of kind `sponsor` is shown only as a logo in the page footer.

**Full page ticket.** The printed document one attendee receives: the event name, the ticket name,
the attendee name, the selection answers, the venue, the dates, a quick response code, the linear
barcode when that option is on, the ticket instructions and a footer repeating the organiser
details.

**Global communication.** An automatic communication that is sent once to every attendee of an event
rather than once per attendee arrival. Its trigger is one of "before the event starts", "after the
event started", "after the event ended" or "before the event ends". On a multi-slot event it fires
once per slot.

**Installable event application.** The public event site packaged so that a visitor can add it to
the home screen of a device. It is made of an application name, an application icon derived from
the site icon, a manifest, a background worker and an offline page. See
[`tracks-and-agenda.md`](tracks-and-agenda.md).

**Karma.** The reputation points a site visitor accumulates in the community capability of the
platform. Quiz points earned on a talk are added to the karma of the reader when that capability is
present; the mechanism itself belongs to
[Learning, surveys and gamification](../learning-surveys-and-gamification/README.md).

**Key talk.** A talk flagged as always wish-listed: every attendee has a reminder on it unless they
explicitly opt out. The opt-out is stored separately from the ordinary wish-list flag, because a key
talk cannot simply be removed from the list.

**Kiosk.** The full-screen registration desk screen, reached from an event or from the menu, offering
a badge scan and an attendee list.

**Lead.** The record produced from attendees by the lead-generation rules. The lead itself belongs to
the [Customer relationship management](../customer-relationship-management/README.md) domain; this
domain owns the rules that create and update it. Also called an **opportunity** when its type is
`opportunity`.

**Lead request.** A background job ticket created when an administrator asks to regenerate the leads
of an event that has too many attendees to process in one pass. At most one exists per event at a
time.

**Lead-generation rule.** A rule that turns attendees into leads: a creation basis (one lead per
attendee, or one lead per order), a trigger (at creation, at registration, at attendance), optional
filters on the event, the event templates, the company and a stored condition, and the default
values written on the produced lead.

**Leaderboard.** The public ranking of site visitors by the quiz points they collected across the
talks of one event, ordered by points descending then by visitor identifier ascending.

**Live talk.** A talk whose start has passed and whose end has not, and which carries a video
address. While a talk is live the public page shows the video and, when the video is not marked as
a recording, the accompanying chat. See **Replay**.

**Multi-slot event.** An event that repeats in several dated time slots. The seat maximum, the ticket
maximum and the communications then apply per slot, and every registration must name a slot.

**Online event.** An event with no venue. Its `event_url` carries the address where it takes place;
setting a venue clears that address.

**Organiser.** The contact that organises the event. It is the sender of every automatic
communication and is printed in the footer of the full page ticket.

**Participant.** A reader with a registration in state `open` or `done` on an event. A participant
always sees the event in the public lists, whatever its visibility setting.

**Per-event menu.** The tree of website menu entries created for one event when its website
switch is turned on: the introduction page, the registration page, the talk pages, the booth page,
the exhibitor page and the community page, each one an entry of Website Event Menu tied to the page
it opens.

**Per-order limit.** The largest number of one ticket a single order may contain. Zero switches the
rule off; the value may never exceed the seat maximum of the ticket, nor the absolute ceiling of
thirty, nor be negative.

**Progress marker.** The coloured state shown on a card in a pipeline view, called the kanban state.
An event has four values, one of which cancels the event; a talk has three.

**Public visitor.** The site visitor record that identifies a browser, signed in or not. It links a
person to their registrations and to their wish-listed talks, survives across visits, and is merged
into the surviving record when two visitors turn out to be the same person.

**Question.** A question asked on the registration form, of one of six kinds: selection, text input,
name, electronic mail address, telephone and company. The last four also write the matching field of
the registration. A question is asked once per order or once per attendee.

**Quick response code.** The two-dimensional code printed next to the barcode on badges and tickets.
It is always printed; the linear barcode next to it appears only when the barcode option is on.

**Quiz.** A set of questions attached to one talk. Each question has exactly one correct answer and
at least one incorrect one; each answer carries a number of points, which may be granted for an
incorrect answer as well.

**Registration.** One attendee seat: the unit in which seats are counted, communications are traced,
badges are printed and attendance is recorded. Its four-state life cycle is in
[`state-machines.md`](state-machines.md).

**Registration desk.** The place, and the screen, where badges are scanned and attendees are marked
as having attended. The lowest access group of the domain is named after it.

**Replay.** A talk whose video is marked as a recording rather than a broadcast. A replay hides
the live-only elements of the public talk page, in particular the chat, and it is never counted as
live in the grouping of the talk list.

**Reserved seats.** The number of active registrations of a scope in state `open`. Together with the
used seats they make the taken seats.

**Revenue analysis.** The read-only analysis that joins registrations with their tickets, orders and
order lines to show how many seats were sold, at which price, by whom and to whom. Its two measures
are per seat and expressed in the company currency.

**Ribbon style.** The visual style attached to a sponsorship level and shown on the sponsor cards of
the public exhibitor list. It is a presentation attribute only and has no effect on any
calculation.

**Sale status.** The payment situation of a seat: not sold, sold or free. It is derived from the
order that carries the seat and never set by hand.

**Seat.** The unit of capacity. One registration occupies one seat, and only while it is active and
in state `open` or `done`.

**Service tracking.** The marker on a product that says what selling that product creates. This
domain adds two values to it: the event value, which makes the product usable as a ticket product,
and the booth value, which makes it usable as a booth category product. The marker itself belongs to
[Products and catalog](../products-and-catalog/README.md).

**Session.** An alternative word for a **talk**: one item of the programme. This folder says talk,
and the entity is Event Track.

**Signed access address.** A public address that carries a keyed digest of the records it exposes,
so that an attendee can fetch a badge or a ticket without an account and without being able to
reach the documents of anybody else. The digest is produced by `get_tickets_access_hash`.

**Slot.** One dated occurrence of a multi-slot event, carrying a calendar date, a start hour and an
end hour expressed as fractional hours in the display time zone, and its own seat counters.

**Sold out.** The condition in which no seat can be taken any more: either the event has a positive
maximum and no available seat, or it has at least one ticket and every sellable combination of slot
and ticket is exhausted. An event may be sold out while it still shows free seats, because its only
ticket is exhausted.

**Speaker.** The person who gives a talk. A talk carries a speaker block filled from the contact only
while empty, and an operational contact block that the contact always overwrites.

**Sponsor.** An organisation attached to an event with its own public page, a sponsorship level, a
kind and, for an online exhibitor, daily opening hours. A sponsor may be created automatically when a
booth of a sponsoring category is booked.

**Sponsorship level.** A named tier with a ribbon style and a ranking sequence. A **lower** sequence
means a **higher** level; the shipped levels are Gold, Silver and Bronze.

**Stand.** See **Booth**.

**Taken seats.** The reserved seats plus the used seats: the total number of seats a scope has given
away. Marking an attendee as having attended moves a seat from reserved to used and leaves the taken
figure unchanged.

**Talk.** One item of the programme: a title, an abstract, a speaker, a room, a start, a duration,
tags, a review stage, a publication state, an optional video, an optional action button and an
optional quiz. The entity is Event Track. Also called a session.

**Talk review stage.** One column of the pipeline through which a talk passes from proposal to
publication, carrying the three capability flags that decide whether the talk appears in the agenda
and whether it is published.

**Template communication.** A communication line defined on an event template and copied onto the
events created from it, matched by the tuple of interval number, interval unit, trigger kind and
template reference.

**Ticket.** A named kind of seat of one event: a sale window, an optional seat maximum, a per-order
limit, a colour and, with the product bridge, a product and a price. An event may have no ticket at
all, in which case the public form offers one generic line.

**Ticket instructions.** The text printed at the bottom of the full page ticket, taken from the
event template when the event has none.

**Time slot.** See **Slot**.

**Trigger kind.** What an automatic communication is measured against: each registration, the start
of the event, the end of the event, or their negative counterparts. The stored values are
`after_sub`, `before_event`, `after_event_start`, `after_event` and `before_event_end`.

**Used seats.** The number of active registrations of a scope in state `done`, that is the attendees
who actually turned up.

**Venue.** The contact that represents the place where the event happens. An event with no venue is
an online event.

**Wish list.** The set of talks for which a site visitor has asked for a reminder. It is stored on
the link between the visitor and the talk, together with the opt-out flag used for key talks and the
quiz result.

---

---

## Vocabularies reconciled

The two source versions of this folder used different words for the same records. The consolidated
text uses the full names of the entity dictionary of this repository; the other wording is kept
here so that either reader finds the entry.

| Wording of version P | Wording used here | Entity |
|---|---|---|
| Stand | Booth | `event.booth` |
| Stand Category | Event Booth Category | `event.booth.category` |
| Stand Reservation | Event Booth Registration | `event.booth.registration` |
| Template Stand | Event Booth Template | `event.type.booth` |
| Session | Talk (Event Track) | `event.track` |
| Session Stage | Event Track Stage | `event.track.stage` |
| Session Location | Event Track Location | `event.track.location` |
| Session Visitor Link | Track / Visitor Link | `event.track.visitor` |
| Communication Schedule | Event Automated Mailing | `event.mail` |
| Communication per Attendee | Registration Mail Scheduler | `event.mail.registration` |
| Communication per Slot | Slot Mail Scheduler | `event.mail.slot` |
| Template Communication | Mail Scheduling on Event Category | `event.type.mail` |
| Opportunity Rule | Event Lead Rules | `event.lead.rule` |
| Opportunity Generation Request | Event Lead Request | `event.lead.request` |
| Attendee Detail Dialogue | Edit Attendee Details on Sales Confirmation | `registration.editor` |
| Attendee Detail Line | Edit Attendee Line on Sales Confirmation | `registration.editor.line` |
| Ticket Configuration Dialogue | Event Configurator | `event.event.configurator` |
| Stand Configuration Dialogue | Event Booth Configurator | `event.booth.configurator` |
| Sponsor Level | Event Sponsor Level | `event.sponsor.type` |
| Registration Question | Event Question | `event.question` |
| Question Suggested Answer | Event Question Answer | `event.question.answer` |
| Registration Answer | Event Registration Answer | `event.registration.answer` |
| Revenue Analysis | Event Sales Report | `event.sale.report` |

---

## Reconciliation notes

1. **Provenance.** Neither version carried a glossary file; both reading orders announced one. Every
   term defined above is drawn from the vocabulary the two versions actually used, so that a reader
   arriving from either one finds the entry under the word that version used.
2. **Two vocabularies.** Where the versions disagreed on the readable name of a record, the table
   above maps the wording of version P onto the full name used throughout this folder, and the
   entries **Session** and **Stand** are cross-references so that the alphabetical list works from
   either vocabulary.
3. **Identifiers.** Where a term corresponds to a stored field or a stored value, the identifier is
   given in code font next to the definition and its full name is in
   [`entities.md`](entities.md).

