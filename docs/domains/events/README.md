# Events

This domain specifies the complete event organisation capability of the system: the definition of an
event with its dates, time zone, venue, seat limits and lifecycle stages; the ticket types that may
be taken for it; the time slots an event may be split into; the attendee registrations with their
own lifecycle, their answers to registration questions, their barcodes and their attendance
recording; the automated communication schedule that contacts attendees by electronic mail or by
text message at configurable offsets around registration, around the start of the event and around
the end of the event; the exhibition stands (booths) that may be rented to sponsors; the agenda of
sessions (tracks) with their speakers, stages, live broadcasting and quizzes; the sponsors and
exhibitors shown on the public pages; the selling of tickets and of stands through sales orders and
through the counter (point of sale); and the automatic creation of opportunities from registrations.

The domain owns no journal entries of its own. Every financial consequence of an event reaches the
ledger through the selling domain that carried the transaction — a sales order and the customer
invoice derived from it, or a counter order and the session closing entry derived from it. Those
consequences are nevertheless fully specified here, in
[`accounting-effects.md`](accounting-effects.md), because the account selection depends on the
product that stands behind the ticket or the stand category, and because the moment at which the
seat becomes occupied differs from the moment at which the revenue is recognised.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Event definition | Name, description, note, responsible user, organiser, venue, country, online address, language, display time zone, start and end instant, tags, company. |
| Event templates | A reusable skeleton that pre-fills seat limitation, time zone, tickets, stands, communication schedule, questions, tags, note and ticket instructions on a newly created event. |
| Lifecycle stages | An ordered, user-definable pipeline of stages with an end marker; ended events are moved automatically into the first end stage. A separate cancellation marker suspends communications. |
| Seat limits and availability | A maximum attendee count at event level, at slot level and at ticket level, with an exact availability formula per combination and a validation that refuses to overbook. |
| Time slots | An event may be declared multi-slot; each slot carries a date and a start and end hour expressed in the display time zone, and each slot then carries its own seat limit and its own communication schedule. |
| Ticket types | Named ticket types with an optional sale window, an optional seat maximum, a per-order quantity limit, a price (when the product bridge is installed) and derived availability flags. |
| Registrations | One record per attendee seat, with a four-state lifecycle, contact fields synchronised from the booked contact, a unique barcode, answers to questions, free-form properties, campaign attribution and an attendance instant. |
| Registration questions | Reusable or event-specific questions of six kinds, mandatory or optional, asked once per order or once per attendee, with suggested answers for the selection kind. |
| Attendance recording | A scanning operation that looks a registration up by its barcode and moves it to the attended state, returning a status word per outcome. |
| Automated communications | A schedule of communications per event, each with a trigger kind, an offset number and an offset unit, and a template reference which is either an electronic mail template or a text message template; a scheduled job walks the schedules and sends. |
| Badges and tickets | Printable full-page tickets, foldable badges and a responsive page form, reachable from a signed address so that an attendee may fetch them without an account. |
| Calendar files | A calendar attachment per event or per slot, and calendar service links for the public pages. |
| Exhibition stands | Stands grouped in categories, each stand available or unavailable, each carrying a renter with contact details, sold either manually or through an order line that may reserve several competing candidates. |
| Agenda sessions | Sessions (tracks) with a speaker, a location, a start instant and a duration, a kanban pipeline of stages, publication on the public pages, wish-listing by visitors, a call-to-action button, live broadcasting and a quiz. |
| Quizzes | A quiz attached to a session, with questions, answers, per-answer point awards, an attempt counter, a decreasing award for later attempts, and a points total carried on the visitor link. |
| Sponsors and exhibitors | Sponsor levels with ordering, sponsors attached to an event, exhibitors with an online presence and a chat window during the event. |
| Selling through an order | An order line whose product is marked as an event registration carries the event, the slot and the ticket; confirming the order creates one registration per unit; a dialogue collects attendee details; the registration lifecycle follows the order lifecycle. |
| Selling at the counter | A counter order line whose product is an event ticket carries the ticket; paying the order creates confirmed registrations, sends the badge message and refreshes the seat counters of every open counter session. |
| Opportunity generation | Rules that turn registrations into opportunities, one per attendee or one per order, triggered at creation, at confirmation or at attendance, with filters and default values. |
| Public pages | Event list and detail pages, the registration form with ticket and slot choice, the question form, the confirmation page, the agenda, session pages, the exhibitor list and the community page. |
| Reporting | An attendee analysis, an answer analysis and a revenue analysis that combines registrations with their order lines. |

---

## 2. Entities of the domain

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Event | `event.event` | `event_event` | The occasion itself: dates, venue, limits, stage, tickets, slots, communications. |
| Event Template | `event.type` | `event_type` | A reusable configuration skeleton applied when an event is created. |
| Event Stage | `event.stage` | `event_stage` | One position in the event pipeline, optionally marked as an end position. |
| Event Tag | `event.tag` | `event_tag` | A public or internal classification label for events. |
| Event Tag Category | `event.tag.category` | `event_tag_category` | A named group of tags, giving them display order. |
| Event Slot | `event.slot` | `event_slot` | One time slot of a multi-slot event, with its own seat counters. |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | A ticket type available for one event, with its sale window and seat maximum. |
| Event Template Ticket | `event.type.ticket` | `event_type_ticket` | A ticket type defined on a template and copied onto events. |
| Registration | `event.registration` | `event_registration` | One attendee seat, with its lifecycle, contact data, barcode and attendance. |
| Registration Question | `event.question` | `event_question` | A question asked when registering, of one of six kinds. |
| Question Suggested Answer | `event.question.answer` | `event_question_answer` | One selectable answer of a selection question. |
| Registration Answer | `event.registration.answer` | `event_registration_answer` | The answer one attendee gave to one question. |
| Communication Schedule | `event.mail` | `event_mail` | One planned communication of an event: trigger, offset, template. |
| Communication per Attendee | `event.mail.registration` | `event_mail_registration` | The per-attendee instance of an after-registration communication. |
| Communication per Slot | `event.mail.slot` | `event_mail_slot` | The per-slot instance of an event-based communication on a multi-slot event. |
| Template Communication | `event.type.mail` | `event_type_mail` | A communication defined on a template and copied onto events. |
| Stand | `event.booth` | `event_booth` | One rentable exhibition stand of an event, available or unavailable. |
| Stand Category | `event.booth.category` | `event_booth_category` | A named class of stands with a shared description, image, product and price. |
| Template Stand | `event.type.booth` | `event_type_booth` | A stand defined on a template and copied onto events. |
| Stand Reservation | `event.booth.registration` | `event_booth_registration` | A pending claim of a stand by one order line, before the stand is awarded. |
| Session | `event.track` | `event_track` | One agenda item of an event: talk, workshop or session, with its speaker. |
| Session Stage | `event.track.stage` | `event_track_stage` | One position in the session pipeline, with publication and cancellation markers. |
| Session Location | `event.track.location` | `event_track_location` | A named room or place where sessions take place. |
| Session Tag | `event.track.tag` | `event_track_tag` | A classification label for sessions. |
| Session Tag Category | `event.track.tag.category` | `event_track_tag_category` | A named group of session tags. |
| Session Visitor Link | `event.track.visitor` | `event_track_visitor` | The relation between one public visitor and one session: wish-listed, blacklisted, quiz points. |
| Quiz | `event.quiz` | `event_quiz` | A quiz attached to one session. |
| Quiz Question | `event.quiz.question` | `event_quiz_question` | One question of a quiz. |
| Quiz Answer | `event.quiz.answer` | `event_quiz_answer` | One possible answer of a quiz question, correct or not, with an explanation. |
| Sponsor | `event.sponsor` | `event_sponsor` | A sponsoring or exhibiting organisation attached to an event. |
| Sponsor Level | `event.sponsor.type` | `event_sponsor_type` | A named sponsoring tier with a display order. |
| Opportunity Rule | `event.lead.rule` | `event_lead_rule` | A rule turning registrations into opportunities. |
| Opportunity Generation Request | `event.lead.request` | `event_lead_request` | A background job record used to re-run opportunity rules over a large event. |
| Attendee Detail Dialogue | `registration.editor` | transient | Collects attendee names, addresses and telephone numbers after an order is confirmed. |
| Attendee Detail Line | `registration.editor.line` | transient | One attendee row of that dialogue. |
| Ticket Configuration Dialogue | `event.event.configurator` | transient | Chooses the event, slot and ticket behind an order line. |
| Stand Configuration Dialogue | `event.booth.configurator` | transient | Chooses the event, stand category and stands behind an order line. |
| Revenue Analysis | `event.sale.report` | database view `event_sale_report` | Read-only aggregation of registrations joined to their order lines. |

Entities that this domain extends rather than owns — and whose extensions are specified here — are
the Contact (`res.partner`), the Product Template and Product Variant (`product.template`,
`product.product`), the Sales Order and Sales Order Line (`sale.order`, `sale.order.line`), the
Counter Order and Counter Order Line (`pos.order`, `pos.order.line`), the Counter Configuration and
Counter Session (`pos.config`, `pos.session`), the Customer Invoice (`account.move`), the
Opportunity (`crm.lead`), the Electronic Mail Template (`mail.template`), the Text Message Template
(`sms.template`), the Public Visitor (`website.visitor`) and the Site (`website`).

---

## 3. Reading order

1. **[`README.md`](README.md)** — this file: scope, entity list, dependencies.
2. **[`glossary.md`](glossary.md)** — every term defined; read it first if any word below is
   unfamiliar.
3. **[`entities.md`](entities.md)** — the complete field tables. Everything else refers back to it.
4. **[`state-machines.md`](state-machines.md)** — the registration lifecycle, the event stage
   pipeline and its cancellation marker, the stand availability states, the session pipeline, the
   communication status.
5. **[`calculations.md`](calculations.md)** — the seat availability formulas, the communication
   scheduling arithmetic, the price derivations, the time-zone arithmetic, the quiz point awards.
6. **[`business-rules.md`](business-rules.md)** — every validation with its exact message, the
   permission checks and the deletion protections.
7. **[`workflows.md`](workflows.md)** — the end-to-end operational sequences.
8. **[`accounting-effects.md`](accounting-effects.md)** — the journal items produced when a ticket
   or a stand is sold through each channel.
9. **[`configuration.md`](configuration.md)** — settings, parameters, shipped records, security
   groups, access rights, record rules, scheduled jobs.
10. **[`interfaces.md`](interfaces.md)** — menus, views, routes, reports, templates, data contracts.
11. **[`acceptance-criteria.md`](acceptance-criteria.md)** — numbered scenarios with concrete
    numbers, including the seven mandatory ones.

---

## 4. Dependencies on other domains

| Domain | What this domain relies on |
|---|---|
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread and activity mixins carried by events, registrations, stands and sessions; electronic mail templates and their rendering; the mass-mailing composer used to send scheduled communications; text message templates and the mass text message dispatch; the scheduled job infrastructure. |
| [Products and catalog](../products-and-catalog/README.md) | The product behind a ticket type and behind a stand category, its sale description, its list price, its taxes and its service tracking marker. |
| [Pricing and pricelists](../pricing-and-pricelists/README.md) | The contextual discount applied to a ticket price and to a stand price when a price list is active on the order. |
| [Sales](../sales/README.md) | The order, the order line, the confirmation algorithm the events bridge extends, the invoicing policy forced to *ordered quantities*, the customer portal. The events domain adds fields and hooks; it does not restate the order behaviour. |
| [Point of sale](../point-of-sale/README.md) | The counter configuration, the counter session and its closing entry, the counter order and its lines, the data set loaded at session opening, the refund mechanism. The events domain adds the ticket line and the registration creation; it does not restate the session behaviour. |
| [Accounts receivable](../accounts-receivable/README.md) | The customer invoice produced from an order carrying ticket lines or stand lines, and the payment hook that marks stands as paid. |
| [Taxes](../taxes/README.md) | The tax computation applied to a ticket price to produce the tax-inclusive display price. |
| [Customer relationship management](../customer-relationship-management/README.md) | The opportunity entity, its stages and teams, the merge algorithm the events bridge extends. |
| [Multi-currency](../multi-currency/README.md) | The conversion of order line totals into the event company currency for the sales total shown on an event. |
| [Website and storefront](../website-and-storefront/README.md) | The site, the public visitor, the published-record mixin, the search mixin, the menu system and the page editing used by the public event pages. |
| [Contacts and organizations](../contacts-and-organizations/README.md) | The contact behind an organiser, a venue, a renter, a speaker and a booked-by attendee, and the telephone number formatting applied to registration telephone numbers. |
| [Identity and access](../identity-and-access/README.md) | The user groups, the access rights matrix and the record rules described in [`configuration.md`](configuration.md). |
| [Learning, surveys and gamification](../learning-surveys-and-gamification/README.md) | The karma and profile mechanics that the session quiz points feed when the community profile capability is present. |

---

## 5. What this domain deliberately does not cover

- The order lifecycle, the invoicing of an order and the customer portal acceptance flow: see
  [Sales](../sales/README.md).
- The counter session lifecycle, cash control and the closing journal entry line by line: see
  [Point of sale](../point-of-sale/README.md).
- Tax computation itself: see [Taxes](../taxes/README.md).
- The opportunity pipeline, its probability model and its assignment algorithm: see
  [Customer relationship management](../customer-relationship-management/README.md).
- The mass-mailing engine, the outgoing message queue and the bounce handling: see
  [Messaging and activities](../messaging-and-activities/README.md).
