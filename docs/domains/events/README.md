# Events

This domain specifies the complete event organisation capability of the system: the definition of
an event with its dates, time zone, venue, seat limits and lifecycle stages; the ticket types that
may be taken for it; the time slots an event may be split into; the attendee registrations with
their own lifecycle, their answers to registration questions, their barcodes and their attendance
recording; the automated communication schedule that contacts attendees by electronic mail or by
text message at configurable offsets around registration, around the start of the event and around
the end of the event; the exhibition booths that may be rented to sponsors; the programme of talks
with their speakers, rooms, live broadcasting and quizzes; the sponsors and exhibitors shown on the
public pages; the selling of tickets and of booths through sales orders and through the shop
counter; and the automatic creation of leads from registrations. It owns the whole life cycle from
the moment an organiser drafts an event until the moment an attendee is scanned at the registration
desk and the event is moved into an ended stage.

The domain owns no journal entries of its own. Every financial consequence of an event reaches the
ledger through the selling domain that carried the transaction — a sales order and the customer
invoice derived from it, or a counter order and the session closing entry derived from it. Those
consequences are nevertheless fully specified here, in
[`accounting-effects.md`](accounting-effects.md), because the account selection depends on the
product that stands behind the ticket or the booth category, and because the moment at which the
seat becomes occupied differs from the moment at which the revenue is recognised.

---

## 1. The questions this domain answers

1. What events are we running, when, where, in which time zone, with which responsible person and
   which organiser?
2. How many seats do we have, how many are booked, how many people actually turned up, and is the
   event or a single ticket sold out?
3. Which ticket types exist, at which price, inside which sales window, with which per-order limit?
4. Who is attending, under which ticket, in which time slot, with which answers to the registration
   questions, and with which barcode on their badge?
5. What do we send them automatically, when, and did it go out?
6. Which organisations rent which booths, at which price, and which of them become published
   sponsors or exhibitors?
7. What is the programme: which talks, by which speaker, in which room, at which hour, visible to
   whom, replayed from which video service, with which quiz and how many points?
8. How much money did the event bring in, through the sales pipeline or through the shop counter?

---

## 2. Capabilities covered

| Capability | Summary |
|---|---|
| Event definition | Name, subtitle, description, internal note, responsible user, organiser, venue, country, online address, language, display time zone, start and end instant, tags, company. |
| Event templates | A reusable skeleton that pre-fills seat limitation, time zone, tickets, booths, communication schedule, questions, tags, note, ticket instructions and website switches on a newly created event. |
| Lifecycle stages | An ordered, user-definable pipeline of stages with an end marker; ended events are moved automatically into the first end stage. A separate cancellation marker suspends registrations and communications. |
| Seat limits and availability | A maximum attendee count at event level, at slot level and at ticket level, with an exact availability formula per combination and a validation that refuses to overbook. |
| Time slots | An event may be declared multi-slot; each slot carries a date and a start and end hour expressed in the display time zone, and each slot then carries its own seat limit and its own communication schedule. |
| Ticket types | Named ticket types with an optional sale window, an optional seat maximum, a per-order quantity limit, a price (when the product bridge is installed) and derived availability flags. |
| Registrations | One record per attendee seat, with a four-state lifecycle, contact fields synchronised from the booked contact, a unique barcode, answers to questions, free-form properties, campaign attribution and an attendance instant. |
| Registration questions | Reusable or event-specific questions of six kinds, mandatory or optional, asked once per order or once per attendee, with suggested answers for the selection kind, and an answer breakdown analysis. |
| Attendance recording | A scanning operation that looks a registration up by its barcode and moves it to the attended state, returning a status word per outcome. |
| Automated communications | A schedule of communications per event, each with a trigger kind, an offset number and an offset unit, and a template reference which is either an electronic mail template or a text message template; a scheduled job walks the schedules and sends. |
| Badges and tickets | Printable full-page tickets, badges in three layouts, an attendee list and a responsive page form, reachable from a signed address so that an attendee may fetch them without an account. |
| Calendar files | A calendar attachment per event, per slot and per talk, and external calendar service links for the public pages. |
| Exhibition booths | Booths grouped in categories, each booth available or unavailable, each carrying a renter with contact details, sold either manually or through an order line that may reserve several competing candidates. |
| Programme | Talks with a speaker, a room, a start instant and a duration, a review pipeline of stages, publication on the public pages, wish-listing by visitors, an action button, live broadcasting and a quiz. |
| Quizzes | A quiz attached to a talk, with questions, answers, per-answer point awards, a completion flag, a points total carried on the visitor link and a per-event leaderboard. |
| Sponsors and exhibitors | Sponsorship levels with ribbon styles and ordering, sponsors attached to an event, exhibitors with an online presence, opening hours and country flags. |
| Selling through an order | An order line whose product is marked as an event registration carries the event, the slot and the ticket; confirming the order creates one registration per unit; a dialogue collects attendee details; the registration lifecycle follows the order lifecycle. |
| Selling at the shop counter | A counter order line whose product is an event ticket carries the ticket; paying the order creates confirmed registrations, sends the badge message and refreshes the seat counters of every open counter session. |
| Lead generation | Rules that turn registrations into leads, one per attendee or one per order, triggered at creation, at registration or at attendance, with filters and default values, plus a batch regeneration job. |
| Public pages | Event list and detail pages, the registration form with ticket and slot choice, the question form, the confirmation page, the talk list, the agenda, talk pages, the booth catalogue, the exhibitor list, the sponsor page, the community leaderboard and an installable application manifest. |
| Reporting | An attendee analysis, an answer breakdown and a revenue analysis that combines registrations with their order lines. |

---

## 3. Actors

| Actor | Description |
|---|---|
| Registration Desk operator | Lowest event role. Reads events, tickets, booths, templates, stages and tags; creates and updates registrations; scans badges; may post messages on an event. |
| Event User | Full operational role. Creates and updates events, tickets, slots, tags, booths, talks, rooms and questions; runs the reporting. |
| Event Administrator | Configuration role. Manages templates, stages, booth categories, sponsorship levels, talk review stages, quizzes and communication schedules, and may regenerate leads. |
| Salesperson | Sells tickets and booths on sales orders, opens the attendee editor, sees the sales information carried on booths and events. |
| Shop counter operator | Sells tickets at the counter, collects attendee details, prints badges and tickets. |
| Website visitor (anonymous) | Browses published events, selects tickets, fills the registration form, proposes talks, wish-lists talks, answers quizzes, books booths. |
| Portal user | The same as the anonymous visitor, with an identified contact. |
| Speaker | Proposes a talk from the public site and is contacted through the discussion thread of the talk. |
| Attendee | Receives the communications, the ticket and the badge, and is checked in at the desk. |
| Scheduled job runner | Executes the communication scheduler, the lead-generation batches and the automatic move of ended events. |

---

## 4. Entities this folder owns

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Event | `event.event` | `event_event` | The occasion itself: dates, venue, limits, stage, tickets, slots, communications. |
| Event Template | `event.type` | `event_type` | A reusable configuration skeleton applied when an event is created. |
| Event Stage | `event.stage` | `event_stage` | One position in the event pipeline, optionally marked as an end position. |
| Event Tag | `event.tag` | `event_tag` | A public or internal classification label for events. |
| Event Tag Category | `event.tag.category` | `event_tag_category` | A named group of tags, giving them display order and publication. |
| Event Slot | `event.slot` | `event_slot` | One time slot of a multi-slot event, with its own seat counters. |
| Event Ticket | `event.event.ticket` | `event_event_ticket` | A ticket type available for one event, with its sale window and seat maximum. |
| Event Template Ticket | `event.type.ticket` | `event_type_ticket` | A ticket type defined on a template and copied onto events. |
| Event Registration | `event.registration` | `event_registration` | One attendee seat, with its lifecycle, contact data, barcode and attendance. |
| Event Question | `event.question` | `event_question` | A question asked when registering, of one of six kinds. |
| Event Question Answer | `event.question.answer` | `event_question_answer` | One selectable answer of a selection question. |
| Event Registration Answer | `event.registration.answer` | `event_registration_answer` | The answer one attendee gave to one question. |
| Event Automated Mailing | `event.mail` | `event_mail` | One planned communication of an event: trigger, offset, template. |
| Registration Mail Scheduler | `event.mail.registration` | `event_mail_registration` | The per-attendee instance of an after-registration communication. |
| Slot Mail Scheduler | `event.mail.slot` | `event_mail_slot` | The per-slot instance of an event-based communication on a multi-slot event. |
| Mail Scheduling on Event Category | `event.type.mail` | `event_type_mail` | A communication defined on a template and copied onto events. |
| Event Booth | `event.booth` | `event_booth` | One rentable exhibition booth of an event, available or unavailable. |
| Event Booth Category | `event.booth.category` | `event_booth_category` | A named class of booths with a shared description, image, product and price. |
| Event Booth Template | `event.type.booth` | `event_type_booth` | A booth defined on a template and copied onto events. |
| Event Booth Registration | `event.booth.registration` | `event_booth_registration` | A pending claim of a booth by one order line, before the booth is awarded. |
| Event Track | `event.track` | `event_track` | One talk of the programme, from proposal to publication. |
| Event Track Stage | `event.track.stage` | `event_track_stage` | One position in the talk review pipeline, with its publication and cancellation markers. |
| Event Track Location | `event.track.location` | `event_track_location` | A named room or stage where talks take place. |
| Event Track Tag | `event.track.tag` | `event_track_tag` | A classification label for talks. |
| Event Track Tag Category | `event.track.tag.category` | `event_track_tag_category` | A named group of talk tags. |
| Track / Visitor Link | `event.track.visitor` | `event_track_visitor` | The relation between one public visitor and one talk: wish-listed, opted out, quiz points. |
| Quiz | `event.quiz` | `event_quiz` | A quiz attached to one talk. |
| Content Quiz Question | `event.quiz.question` | `event_quiz_question` | One question of a quiz. |
| Question's Answer | `event.quiz.answer` | `event_quiz_answer` | One possible answer of a quiz question, correct or not, with points and an explanation. |
| Event Sponsor | `event.sponsor` | `event_sponsor` | A sponsoring or exhibiting organisation attached to an event. |
| Event Sponsor Level | `event.sponsor.type` | `event_sponsor_type` | A named sponsorship tier with a ribbon style and a display order. |
| Website Event Menu | `website.event.menu` | `website_event_menu` | One entry of the per-event website menu, with its page view. |
| Event Lead Rules | `event.lead.rule` | `event_lead_rule` | A rule turning registrations into leads. |
| Event Lead Request | `event.lead.request` | `event_lead_request` | A background job record used to re-run lead rules over a large event. |
| Event Sales Report | `event.sale.report` | database view `event_sale_report` | Read-only aggregation of registrations joined to their order lines. |
| Event Configurator | `event.event.configurator` | transient `event_event_configurator` | Chooses the event, slot and ticket behind an order line. |
| Event Booth Configurator | `event.booth.configurator` | transient `event_booth_configurator` | Chooses the event, booth category and booths behind an order line. |
| Edit Attendee Details on Sales Confirmation | `registration.editor` | transient `registration_editor` | Collects attendee names, addresses and telephone numbers after an order is confirmed. |
| Edit Attendee Line on Sales Confirmation | `registration.editor.line` | transient `registration_editor_line` | One attendee row of that dialogue. |

Those thirty-nine entities are the whole scope of this folder. Every one of them is specified field
by field in [`entities.md`](entities.md), and each carries a link to its generated reference page
under `../../references/entities/`.

## 5. Entities of other domains that this folder extends

The extensions are specified here; the entities themselves belong to the domain named.

| Entity | Owning domain | What this domain adds |
|---|---|---|
| Contact (`res.partner`) | [Contacts and organizations](../contacts-and-organizations/README.md) | An event counter, the signed static map address of the contact address with its validity flag, and an action opening the events of that contact. |
| Product Template and Product Variant (`product.template`, `product.product`) | [Products and catalog](../products-and-catalog/README.md) | Two new values of the service-tracking selection, `event` and `event_booth`; the list of event tickets that use the variant; validations that forbid changing that tracking while a ticket or a booth category uses the product; permission for anonymous visitors to read the image of an unpublished ticket product. |
| Sales Order (`sale.order`) | [Sales](../sales/README.md) | An attendee counter, a booth list and booth counter, confirmation rules that create registrations and confirm booths, customer propagation to registrations, cart rules for tickets and booths. |
| Sales Order Line (`sale.order.line`) | [Sales](../sales/README.md) | The event, the slot, the ticket, the registrations, the pending booths, the confirmed booth registrations and the booths; the price taken from the ticket or the booth category; the description built from the ticket or the booths. |
| Customer Invoice (`account.move`) | [Accounts receivable](../accounts-receivable/README.md) | When an invoice that carries booth lines is paid, the booths are stamped as paid. |
| Counter Order and Counter Order Line (`pos.order`, `pos.order.line`) | [Point of sale](../point-of-sale/README.md) | The ticket on the line, the registrations on the line, an attendee counter, badge and ticket printing, the refund that cancels registrations, and the paid order that sends the badge message. |
| Counter Configuration and Counter Session (`pos.config`, `pos.session`) | [Point of sale](../point-of-sale/README.md) | The loading of event data into the counter and the live broadcast of the remaining seats. |
| Pricelist Rule (`product.pricelist.item`) | [Pricing and pricelists](../pricing-and-pricelists/README.md) | A warning when a rule with a positive minimum quantity is aimed at ticket products. |
| Electronic Mail Template and Text Message Template (`mail.template`, `sms.template`) | [Messaging and activities](../messaging-and-activities/README.md) | The filtering of templates to those written on Event Registration, and the deletion of the schedules that referenced a deleted template. |
| Site, Site Menu and Public Visitor (`website`, `website.menu`, `website.visitor`) | [Website and storefront](../website-and-storefront/README.md) | Per-event menu trees and pages, the event search, the application name and icon, the visitor registrations and wish-listed talks, the visitor merge and the retention rule. |
| Configuration Settings (`res.config.settings`) | [Platform foundation](../platform-foundation/README.md) | The feature switches of the domain and the static map credentials. |
| Lead (`crm.lead`) | [Customer relationship management](../customer-relationship-management/README.md) | The source event, the originating rule, the source registrations and their counter, and the merge behaviour of those links. |

Generic platform entities are not owned here: records and their audit fields, sequences, scheduled
jobs, configuration parameters, reports, access rules and request routing belong to the platform
documents listed in section 8.

---

## 6. Reading order

1. **[`README.md`](README.md)** — this file: scope, capabilities, actors, entity list,
   dependencies, and the list of every file in the folder.
2. **[`glossary.md`](glossary.md)** — every term defined; read it first if any word above is
   unfamiliar. It also reconciles the two vocabularies used for the same records.
3. **[`entities.md`](entities.md)** — the complete field tables. Everything else refers back to it.
4. **[`state-machines.md`](state-machines.md)** — the registration lifecycle and its payment
   situation, the event stage pipeline and its cancellation marker, the booth availability states,
   the talk pipeline and its capability ladder, the communication status.
5. **[`calculations.md`](calculations.md)** — the seat availability formulas, the communication
   scheduling arithmetic, the price derivations, the time-zone arithmetic, the quiz point awards.
6. **[`business-rules.md`](business-rules.md)** — every validation with its exact message, the
   permission checks and the deletion protections, numbered `EV-RULE-nnn`.
7. **[`workflows.md`](workflows.md)** — the end-to-end operational sequences.
8. **[`booths-and-exhibitors.md`](booths-and-exhibitors.md)** — the booth catalogue, the competition
   between candidates, the sponsor creation and the public exhibitor pages, in depth.
9. **[`tracks-and-agenda.md`](tracks-and-agenda.md)** — the programme: proposals, review stages, the
   agenda grid, wish lists and reminders, live video, quizzes and the leaderboard, in depth.
10. **[`accounting-effects.md`](accounting-effects.md)** — the boundary with the ledger, and the one
    place where an accounting event writes on a record of this domain.
11. **[`configuration.md`](configuration.md)** — settings, parameters, shipped records, security
    groups, access rights, record rules, scheduled jobs, message templates and menus.
12. **[`interfaces.md`](interfaces.md)** — service operations, request endpoints, printed documents,
    exported files, notifications and screens.
13. **[`acceptance-criteria.md`](acceptance-criteria.md)** — numbered Given, When and Then scenarios
    with concrete numbers.

---

## 7. Files in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | Scope, capabilities, actors, the entities owned and extended, reading order, dependencies, and this list. |
| [`entities.md`](entities.md) | Every entity of the domain, field by field, with types, defaults, derivations, constraints, validation messages, on-change behaviour and life cycle. |
| [`state-machines.md`](state-machines.md) | Every state field: states with stored value, label and meaning; transition tables with triggers, guards and side effects; the exact refusals; a diagram per machine. |
| [`workflows.md`](workflows.md) | End-to-end procedures: creating an event from a template, registering on the public site, selling tickets on an order and at the counter, checking in at the desk, running the communication scheduler, generating leads, booking booths, running the programme, closing an event. |
| [`business-rules.md`](business-rules.md) | The numbered rule catalogue with exact messages, guards, permissions, uniqueness, date and rounding rules, and the mapping of the former rule identifiers. |
| [`calculations.md`](calculations.md) | Every formula: seat counters, availability, sold-out detection, communication schedule dates, ticket and booth pricing with taxes and discounts, quiz scoring, leaderboard ranking, agenda grid placement, revenue conversion. |
| [`accounting-effects.md`](accounting-effects.md) | Where the money lands: this domain posts no journal entry of its own, the boundary is stated precisely, and the one inbound hook is specified. |
| [`configuration.md`](configuration.md) | Capability packages, settings, configuration parameters, scheduled jobs, access groups and rights, record rules, shipped master data, message templates, paper formats, thread subtypes and menus. |
| [`interfaces.md`](interfaces.md) | Service operations, request endpoints, printed documents, exported files and outgoing links, notifications and screens. |
| [`acceptance-criteria.md`](acceptance-criteria.md) | Numbered Given, When and Then scenarios a replacement must pass. |
| [`glossary.md`](glossary.md) | Every term of the domain, defined, plus the reconciliation of the two vocabularies. |
| [`booths-and-exhibitors.md`](booths-and-exhibitors.md) | Extra topic file: the booth catalogue, booking from the back office, from an order and from the public site, paid booths, sponsors and exhibitor pages. |
| [`tracks-and-agenda.md`](tracks-and-agenda.md) | Extra topic file: the programme, from proposal to leaderboard. |

---

## 8. Dependencies on other domains

| Domain | What this domain relies on |
|---|---|
| [Platform foundation](../platform-foundation/README.md) | Records and their audit fields, sequences, scheduled jobs, configuration parameters, reports, access rules and request routing. The platform documents are [`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md), [`../../overview/security-model.md`](../../overview/security-model.md), [`../../runtime/scheduled-jobs.md`](../../runtime/scheduled-jobs.md), [`../../runtime/configuration-parameters.md`](../../runtime/configuration-parameters.md) and [`../../runtime/report-rendering.md`](../../runtime/report-rendering.md). |
| [Identity and access](../identity-and-access/README.md) | The user groups, the access rights matrix and the record rules described in [`configuration.md`](configuration.md), and the anonymous and portal roles. |
| [Contacts and organizations](../contacts-and-organizations/README.md) | The contact behind an organiser, a venue, a renter, a speaker, a sponsor and a booked-by attendee, and the telephone number formatting applied to registration telephone numbers. |
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread and activity mixins carried by events, registrations, booths, talks and sponsors; electronic mail templates and their rendering; the mass-mailing composer used to send scheduled communications; text message templates and the mass text message dispatch; the outgoing message queue. |
| [Products and catalog](../products-and-catalog/README.md) | The product behind a ticket type and behind a booth category, its sale description, its list price, its taxes and its service-tracking marker. |
| [Taxes](../taxes/README.md) | The tax computation applied to a ticket price and to a booth price to produce the tax-inclusive display price. |
| [Pricing and pricelists](../pricing-and-pricelists/README.md) | The contextual discount applied to a ticket price and to a booth price when a price list is active for the reader or on the order. |
| [Sales](../sales/README.md) | The order, the order line, the confirmation algorithm the events bridge extends, the invoicing policy forced to ordered quantities, the cart, the checkout and the customer portal. The events domain adds fields and hooks; it does not restate the order behaviour. |
| [Point of sale](../point-of-sale/README.md) | The counter configuration, the counter session and its closing entry, the counter order and its lines, the data set loaded at session opening, the refund mechanism. The events domain adds the ticket line and the registration creation; it does not restate the session behaviour. |
| [Accounts receivable](../accounts-receivable/README.md) | The customer invoice produced from an order carrying ticket lines or booth lines, and the payment hook that marks booths as paid. |
| [General ledger](../general-ledger/README.md) | The journal entry produced when such an invoice is posted, and the account selection precedence. |
| [Multi-currency](../multi-currency/README.md) | The conversion of order line totals into the currency of the event company for the sales total shown on an event and for the measures of the revenue analysis. |
| [Customer relationship management](../customer-relationship-management/README.md) | The lead entity, its stages and teams, and the merge algorithm the events bridge extends. |
| [Website and storefront](../website-and-storefront/README.md) | The site, the public visitor, the published-record mixin, the search mixin, the menu system, the page editing, the cart and the checkout used by the public event pages. |
| [Marketing and mass mailing](../marketing-and-mass-mailing/README.md) | The mailing entity, its recipient selection and its exclusion list, used by the "Invite", "Contact Attendees" and "Contact Speakers" buttons of an event. This domain declares Event Registration and Event Track as mailing targets and prepares the selection; it does not restate the mailing behaviour. |
| [Learning, surveys and gamification](../learning-surveys-and-gamification/README.md) | The karma and profile mechanics that the quiz points feed when the community profile capability is present. |
| [Analytic accounting](../analytic-accounting/README.md) | The analytic distribution an organisation may set on the order lines of an event; this domain sets none of its own. |

---

## 9. What this domain deliberately does not cover

- The order lifecycle, the invoicing of an order and the customer portal acceptance flow: see
  [Sales](../sales/README.md).
- The counter session lifecycle, cash control and the closing journal entry line by line: see
  [Point of sale](../point-of-sale/README.md).
- Tax computation itself: see [Taxes](../taxes/README.md).
- The lead pipeline, its probability model and its assignment algorithm: see
  [Customer relationship management](../customer-relationship-management/README.md).
- The mass-mailing engine, the outgoing message queue and the bounce handling: see
  [Messaging and activities](../messaging-and-activities/README.md).
- The page editor, the theme system and the site search engine: see
  [Website and storefront](../website-and-storefront/README.md).

---

## Reconciliation notes

Two independently written versions of this folder were merged into the text you are reading.

1. **Scope.** Version P listed thirty-eight owned entities and version M listed thirty-nine; the
   difference was Website Event Menu, which version M owned and version P omitted. The consolidated
   scope is the union, thirty-nine entities, which matches the target taxonomy exactly.
2. **Names.** The two versions used different words for the same records — stand against booth,
   session against talk, opportunity against lead, communication schedule against automated
   mailing. The consolidated text uses the full name each entity carries in the entity dictionary of
   this repository, and [`glossary.md`](glossary.md) lists every alternative wording so that a
   reader of either version finds the entry.
3. **Identifiers.** Version M rewrote the stored field names into a readable form, for example
   `date_time_zone` for the display time zone and `seats_maximum` for the seat cap. Storage names
   are contractual, so the consolidated text reproduces them exactly (`date_tz`, `seats_max`) and
   gives the full name of every field in its own column of the field tables, as the documentation
   rules require. The behaviour described is unchanged.
4. **Files.** Version M had ten files and no `state-machines.md`; the reading order of version P
   announced eleven. The consolidated folder holds the eleven prescribed documents plus the two
   extra topic files of version M, [`booths-and-exhibitors.md`](booths-and-exhibitors.md) and
   [`tracks-and-agenda.md`](tracks-and-agenda.md), both linked from section 7. The state machines
   that version M kept inside `workflows.md` now live in
   [`state-machines.md`](state-machines.md), completed with stored values, labels, guards, refusals
   and diagrams; `workflows.md` points at them.
5. **Message formatting.** Version M reproduced user-facing messages in code font. The consolidated
   folder reproduces them between quotation marks and keeps code font for stored values,
   identifiers and short interface labels, which is what the documentation rules of this repository
   require.
