# Interfaces of the Events domain

The operations a client or an integration invokes, the request endpoints the public site exposes, the printed documents, the exported files, the notifications and the screens described as workflows on views. Nothing here refers to a client technology: a screen is described by the data it shows, the buttons it offers and the guards on those buttons.

## 1. Service operations

Every operation below is invoked on a set of records unless it is marked "model level", in which case it is invoked without records. Errors are the exact messages of [business-rules.md](business-rules.md).

### 1.1 On Event

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `action_set_done` | the events | none | writes the first stage by sequence flagged as an ending stage; does nothing when no such stage exists | none |
| `action_open_slot_calendar` | one event | a screen description opening the slots of that event on a calendar, a list and a form | none | none |
| `get_seats_availability` | one event, a list of `(slot, ticket)` pairs where each element may be empty | one availability per pair, in the same order; "no limit" is a possible value | none | *"Input should be a list of tuples containing slot, ticket"* when a pair is malformed |
| `verify_seats_availability` | one event, a list of `(slot, ticket, count)` triples | nothing on success | none | *"Input should be a list of tuples containing slot, ticket, count"* when a triple is malformed; otherwise the sold-out message of `EV-RULE-030` |
| `action_generate_leads` | the events, optionally a set of rules | a notification description | either applies the rules at once or creates one lead request per event and wakes the batch job | *"Only Event Managers are allowed to re-generate all leads."* |
| `action_view_linked_orders` | the events | a screen description listing the confirmed sales orders having a line on those events, creation disabled | none | none |
| `get_ics_file` | the events, optionally one slot | one calendar file per event, as bytes | none | returns nothing when the calendar library is unavailable |
| `get_event_resource_urls` | one event, optionally one slot | two addresses: an external calendar link and a calendar file link | none | none |
| `get_tickets_access_hash` | one event, a list of registration identifiers | the keyed digest of the pair | none | none |
| `get_kiosk_url` | one event | the address of the registration desk screen | none | none |
| `toggle_website_menu`, `toggle_booth_menu`, `toggle_exhibitor_menu`, `toggle_website_track`, `toggle_website_track_proposal` | the events, a boolean | none | writes the matching switch, which creates or deletes the matching menu entries | none |
| `action_invite_contacts` | the events | a screen description titled "Mass Mail Invitation" opening a mailing prepared on Contact, subject pre-filled as `Event: <event name>`, no default selection | none | none |
| `action_mass_mailing_attendees` | the events | a screen description titled "Mass Mail Attendees" opening a mailing prepared on Event Registration, subject pre-filled as `Event: <event name>`, default selection restricted to the attendees of those events whose state is neither `cancel` nor `draft` | none | none |
| `action_mass_mailing_track_speakers` | the events | a screen description opening a mailing prepared on Event Track, subject pre-filled as `Event: <event name>`, default selection restricted to the talks of those events whose stage is not a cancelling stage; the screen title is "Mass Mail Attendees", which is a **compatibility finding** recorded in [workflows.md](workflows.md#28-send-a-mass-mailing-to-attendees) | none | none |
| `get_slot_tickets_availability_pos` | one event, a list of `(slot identifier, ticket identifier)` pairs | one availability per pair | none | none |

### 1.2 On Event Registration

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `action_confirm` | the registrations | none | state becomes `open`; communications are woken; "registered" lead rules run | the sold-out message of `EV-RULE-030` |
| `action_set_done` | the registrations | none | state becomes `done`; `date_closed` is stamped; the note *"Attended on <date>"* is logged; "attended" lead rules run | the sold-out message when seats do not allow it |
| `action_cancel` | the registrations | none | state becomes `cancel`; seats released | none |
| `action_set_draft` | the registrations | none | state becomes `draft`; seats released | none |
| `action_send_badge_email` | one registration | a screen description opening a message composer pre-loaded with the badge template | none | none |
| `register_attendee` (model level) | a barcode, optionally an event identifier | the registration summary plus one of the seven statuses of [workflows.md](workflows.md#18-check-an-attendee-in-at-the-registration-desk) | sets the state to `done` in the `confirmed_registration` case only | none: every abnormal case is reported as a status, not as an error |
| `get_registration_summary` | one registration | identifier, attendee name, contact, slot display name, ticket name, event identifier and display name, the display texts of the selection answers, company name, badge format, attendance date in short format, whether that date is today in the event time zone; with the product bridge also the sale status, its label and the "has to pay" flag | none | none |
| `action_view_sale_order` | one registration | a screen description opening the sales order form | none | none |
| `action_view_pos_order` | one registration | a screen description opening the counter order form | none | none |
| `apply_lead_generation_rules` | the registrations, optionally a set of rules | the created leads | creates or updates leads | none |

### 1.3 On Event Automated Mailing

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `schedule_communications` (model level) | optionally a flag asking for automatic committing | always true | selects the due schedules and executes them one by one; catches, logs and reports every failure | none: failures are reported on the event, not raised |
| `execute` | one or several schedules | always true | runs the schedule in attendee-based, slot-based or event-based mode; clears the error stamp on success | the failure is raised to the caller when invoked directly |

### 1.4 On Event Booth and Event Booth Registration

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `action_confirm` (booth) | the booths, optionally a dictionary of values to write with the state | none | state becomes `unavailable`; the sponsor is created or reused when the category asks for it; the booking message is posted on the event | none |
| `action_set_paid` (booth) | the booths | none | `is_paid` becomes true | none |
| `action_view_sale_order` (booth) | exactly one booth with an order | a screen description opening that order | none | fails when the booths do not share exactly one order |
| `action_view_sponsor` (booth) | one booth | a screen description opening the sponsor form | none | none |
| `action_confirm` (reservation) | the reservations | none | confirms the booths with the collected values, then cancels and deletes the competing reservations and cancels their orders | the booth availability message of `EV-RULE-074` when raised by the caller |

### 1.5 On the sales side

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `action_confirm` (sales order) | the orders | the standard confirmation result, or, for a single order carrying tickets, a screen description opening the attendee editor | creates the registrations of every event line; confirms the booths of every booth line | `EV-RULE-053`, `EV-RULE-051`, `EV-RULE-074`, `EV-RULE-030` |
| `action_view_attendee_list` (sales order) | the orders | a screen description listing the registrations of those orders | none | none |
| `action_view_booth_list` (sales order) | the orders | a screen description listing the booths of those orders | none | none |
| `action_make_registration` (attendee editor) | the wizard | a close instruction | writes the edited registrations, creates the missing ones, forces the state derivation | the sold-out message of `EV-RULE-030` |
| `print_event_tickets`, `print_event_badges` (counter order) | the orders | a printed document of the registrations of the order lines | none | none |
| `action_view_attendee_list` (counter order) | the orders | a screen description listing the registrations | none | none |

### 1.6 On the programme

| Operation | Inputs | Outputs | Side effects | Errors |
|---|---|---|---|---|
| `get_track_suggestions` | one talk, an optional restricting condition, a wanted count | the ordered candidate talks | none | none |
| `get_ics_file` (talk) | the talks | one calendar file per talk | none | returns nothing when the calendar library is unavailable |
| `get_track_calendar_urls` | one talk | the external calendar link and the calendar file link | none | none |
| `open_track_speakers_list` | the talks | a screen description listing the speaker contacts | none | none |
| `action_add_quiz`, `action_view_quiz` | one talk | a screen description opening the quiz form, creation disabled | none | none |
| `get_event_track_visitors` | one talk, a flag asking to create the link | the visitor links of the reader for that talk | creates the site visitor and the link when asked; refreshes the last-visit stamp | none |

## 2. Request endpoints

"Page" means the response is a rendered page; "call" means the response is a structured answer for a client script; "document" means the response is a file with a content disposition. "Public" means no sign-in is required.

### 2.1 Core

| Path | Method | Access | Purpose | Request | Response |
|---|---|---|---|---|---|
| `/event/<event>/ics` | page request | public | Download the calendar file of an event, optionally for one slot | optional `slot_id`; an unknown slot of that event gives "not found" | document named `<event name>.ics` |
| `/event/<event identifier>/my_tickets` | page request | public | Printed tickets or badges of a set of attendees | `registration_ids` as a structured list, `tickets_hash`, optional `badge_mode`, optional `responsive_html` | printed document, or a responsive page; "not found" when a parameter is missing, the hash does not match, or no registration matches |
| `/event/init_barcode_interface` | call | signed-in user | Initialise the registration desk screen | `event_id`, possibly empty | the event name, the venue country and city, the company name and identifier; or the label `Event Registrations` with the active company when no event is given |

### 2.2 Public event site

| Path | Method | Access | Purpose |
|---|---|---|---|
| `/event`, `/events`, `/event/page/<n>`, `/events/page/<n>`, `/event/tags/<slugs>`, `/event/tags/<slugs>/page/<n>` (and the `events` variants) | page request | public | The event list, with the text search, the date filter, the country filter, the template filter and the tag filters, 12 events per page. A request carrying more than one tag through a plain page request is permanently redirected to `/event`, to stop crawlers from exploring every tag combination. |
| `/event/<event>` | page request | public | Redirects to the first child menu entry of the event, or to `/event/<identifier>/register`. |
| `/event/<event>/register` | page request | public | The registration page: description, practical information, calendar links, open slots and the ticket panel. |
| `/event/<event>/page/<path>` | page request | public | A custom page of the event. The view is found by matching the menu of the event on the page key, falling back to the page-not-found view. |
| `/event/<event>/registration/slot/<slot identifier>/tickets` | call, POST | public | Renders the ticket panel for one slot, with the remaining seats of each slot-and-ticket combination. |
| `/event/<event>/registration/new` | call, POST | public | Renders the attendee form for the chosen quantities; returns nothing when no quantity was chosen. |
| `/event/<event>/registration/confirm` | page request, POST | public | Creates the registrations; see [workflows.md](workflows.md#10-register-from-the-public-website). Redirects back with `registration_error_code=recaptcha_failed` or `registration_error_code=insufficient_seats` on the two refusals. |
| `/event/<event>/registration/success` | page request, GET | public | The confirmation page; requires a matching visitor, otherwise "not found". |
| `/event/<event>/community` | page request | public | The community page; the base package answers with the page-not-found view and the quiz package replaces it with the leaderboard. |

### 2.3 Booths

| Path | Method | Access | Purpose |
|---|---|---|---|
| `/event/<event>/booth` | page request | public | The booth catalogue: the categories that still have free booths, the booths, the exhibition map. Forbidden when the reader cannot read the event. |
| `/event/<event>/booth/register` | page request, POST | public | Collects the checked booths and redirects to the contact form with them in the address. |
| `/event/<event>/booth/register_form` | page request, GET | public | The contact form; "not found" when the booths or the category are missing from the address. |
| `/event/<event>/booth/confirm` | page request, POST | public | Confirms the booking, or, with online booth sales, puts it in the cart. Answers `success` with the event name and the contact block, or one of the codes of `EV-RULE-076`, or a redirect to the cart. |
| `/event/booth/check_availability` | call, POST | public | Returns the identifiers of the requested booths that are no longer available. |
| `/event/booth_category/get_available_booths` | call | public | Returns the free booths of a category as pairs of identifier and name. |

### 2.4 Exhibitors

| Path | Method | Access | Purpose |
|---|---|---|---|
| `/event/<event>/exhibitors`, `/event/<event>/exhibitor` | page request, GET and POST | public | The exhibitor list, filtered by text, by country and by sponsorship level, grouped by level, each level shuffled at random. A Registration Desk reader also sees the unpublished sponsors, shown after the published ones. |
| `/event/<event>/exhibitor/<sponsor>` | page request | public | The sponsor page; the event must have its exhibitor menu on and the sponsor must belong to the event. Forbidden when the reader cannot read the sponsor. The sidebar lists at most 30 other exhibitors, ordered by published, then open now, then same country, then sponsorship level, then at random. |
| `/event_sponsor/<sponsor identifier>/read` | call | public | Returns the sponsor card for the "event not started" dialog: name, slogan, website, electronic mail address, telephone, description, logo address, opening hours and their formatted texts, the "open now" flag, the event display time zone, the country flag address, the country name and identifier, the sponsorship level name and identifier, the event name, and the event flags "ongoing", "done", "starts today", "minutes before start" and start date. |

### 2.5 Programme

| Path | Method | Access | Purpose |
|---|---|---|---|
| `/event/<event>/track`, `/event/<event>/track/tag/<tag>` | page request | public | The talk list, with the text search, the tag filters and a wish-list filter, grouped as live, starting soon, then one group per day, then a group `Coming soon` for the undated talks. A day group whose talks are all finished is collapsed by default when the event still has running or upcoming talks. More than one tag in a plain page request is permanently redirected to `/event/<event>/track`. |
| `/event/<event>/agenda` | page request | public | The agenda grid, computed as described in [calculations.md](calculations.md#11-talk-times-the-action-button-window-and-the-agenda-grid). |
| `/event/<event>/track/<talk>` | page request | public | The talk page, with the video when one is set, the action button, the quiz and the suggested next talks. The event must have its talk menu on. |
| `/event/track/toggle_reminder` | call | public | Switches the reminder of the reader on a talk; answers the new state or the code `ignored`. |
| `/event/track/send_email_reminder` | call | public | Sends the reminder message; answers success, or one of the four refusals of `EV-RULE-085`, or `missing_template`. |
| `/event/<event>/track_proposal` | page request | public | The proposal form. |
| `/event/<event>/track_proposal/post` | page request, POST | public | Creates the proposed talk; answers `success`, `forbidden` or `invalidFormInputs`. |
| `/event/track_tag/search_read` | call | public | Reads talk tags for the filter panel, since an anonymous reader cannot query the tags directly. |
| `/event/<event>/track/<talk>/ics` | page request | public | The calendar file of one talk, named `<event name>-<talk name>.ics`; "not found" when the file cannot be produced. |
| `/event_track/get_track_suggestion` | call | public | The next talk to watch, with the name and the picture of the current talk and the identifier, name, speaker name and address of the suggestion. With the quiz bridge, the answer also says whether the quiz of the current talk should be shown. |
| `/event_track/quiz/submit` | call | public | Submits the quiz answers; see [workflows.md](workflows.md#27-answer-a-quiz). |
| `/event_track/quiz/reset` | call | public | Resets the quiz; forbidden unless the quiz allows unlimited tries or the reader is an Event Administrator. |
| `/event/<event>/community/leaderboard`, `/event/<event>/community/leaderboard/results`, `/event/<event>/community/leaderboard/results/page/<n>` | page request | public | The leaderboard, 30 visitors per page, at most 5 page links, opened on the page containing the reader when the reader is ranked. |
| `/event/manifest.webmanifest` | page request, GET | public | The web application manifest of the event site: the application name, the icon derived from the site icon, and the start address. |
| `/event/service-worker.js` | page request, GET | public | The background worker of the event site, scoped to the event pages. |
| `/event/offline` | page request, GET | public | The page shown when the event site is opened without a connection. |

## 3. Reports and printed documents

| Report | Bound to | Layout | Document name |
|---|---|---|---|
| Badge | Event Registration | the layout chosen by `badge_format` of the event: A4 foldable, A6, or four per sheet | `Badge - <event name> - <attendee name>` |
| Badge Example | Event | the same layouts with placeholder data | `Badge - <event name>` |
| Full Page Ticket | Event Registration | one page per attendee, with the repeated organiser footer | `Full Page Ticket - <event name> - <attendee name>` |
| Full Page Ticket Example | Event | the same with placeholder data | `Full Page Ticket - <event name>` |
| Responsive Rich Text Full Page Ticket | Event Registration | a responsive page rather than a printed sheet, with a download button | not applicable |
| Attendee List | Event | one table per event | `Attendee List - <event name>` |
| Attendee List | Event Registration | one table per event, grouping the given attendees | `Attendee List` |

Slashes are removed from the event name and the attendee name when they are put into a document name.

**Badge content.** Event name over the optional background image; the start and end date and time in the display time zone (the slot dates when the attendee has one, the event dates otherwise, with the end date hidden when the event lasts one day); the venue block; the attendee name; the attendee company name; the organiser logo; a quick response code of the barcode; the linear barcode when the barcode option is on; a coloured strip carrying the ticket name, using the ticket colour or `#875A7B`. In the foldable layout the barcode block, the attendee name and the selection answers move to the bottom-left quarter and the four folding illustrations fill the bottom-right quarter.

**Full page ticket content.** Event name; ticket name; attendee name; the selection answers as chips; the venue block with a map marker; the date block; a quick response code; the linear barcode when the option is on; the ticket instructions of the event; a footer repeated on every page carrying the organiser name, telephone, electronic mail address and website, or the event name when there is no organiser.

**Attendee list content.** Heading `Attendee list`; the event name; the event start and end in the display time zone in medium format; then the table `Name`, `Company`, `Ticket type`, `Phone number`, quick response code. A page break follows each event.

**Answer breakdown.** Not a printed document but an analysis screen over Event Registration Answer, opened from a question. For a selection question it opens on a graph with a pivot and a list; for a text question it opens on a list only. It is filtered on that question and, when opened from an event, on that event, and restricted to answers whose event still links the question.

## 4. Exported files and outgoing links

| File or link | Produced by | Content |
|---|---|---|
| Event calendar file | `/event/<event>/ics` and the calendar links | one calendar entry with the creation stamp, the start and end converted into the display time zone, the event name as the summary, the shortened description (see below) as the description in plain and rich form, and the one-line venue address as the location when a venue exists. With a slot, the slot start and end replace the event dates. |
| Talk calendar file | `/event/<event>/track/<talk>/ics` | one calendar entry with the talk times, or the event times when the talk has none, the talk title as the summary, a description carrying a link to the talk page, the talk title, the shortened abstract and, when the talk has no time of its own, the warning *"Note: The start and end times of the talk were not specified when you asked to add them to your calendar, therefore the times indicated in this reminder correspond to those of the event."*; the location is the venue address followed by the room name. |
| External calendar link of an event | the registration page and the confirmation page | an address carrying the template action, the event name, the start and end formatted as `<year><month><day>T<hour><minute><second>` in the display time zone, the display time zone itself, the shortened description and, when a venue exists, the one-line address. |
| External calendar link of a talk | the talk page | the same, with the text `<event name>: <talk title>` and the talk times. |
| Shortened description | both calendar outputs | a link to the shared event address followed by the plain text of the description, shortened to 1900 characters, which leaves room for the additions of the bridges inside the address length limits of browsers. |
| Signed static map picture of a venue | the contact of the venue | an address built from the street, city, postal code and country of the contact, with a marker, a size, a zoom level and the key, signed with the secret. When the key or the secret is missing, or the secret does not decode, no address is produced. The validity flag performs a real request with a two-second timeout and is true only when the answer succeeds and carries no warning header. |
| Web application manifest | `/event/manifest.webmanifest` | the application name of the event site and the square icon derived from the site icon: the icon is cropped to a square on its larger side and resized to 512 by 512; an icon in a vector format is skipped and no application icon is produced. |

## 5. Notifications

| Notification | Trigger | Recipients | Channel |
|---|---|---|---|
| Registration confirmation | the "after each registration" schedule with a zero interval | the attendee | the message template of the schedule |
| Event reminder | a "before the event starts" schedule | every attendee not `draft` and not `cancel` | the message template of the schedule |
| Registration badge | the Send by Email button, and every paid counter order | the attendee | the badge template, carrying the badge document |
| Communication failure | a failing schedule, at most once per hour | the organiser, the event responsible and the last author of the template | a message on the event, queued |
| Booth booked | a booth becomes unavailable | the followers of the event | a message on the event under the subtype "Booth Booked" |
| Event published or unpublished | the publication flag changes | the followers of the event | a message under the matching subtype |
| New talk | a talk is created or proposed | the followers of the event | a message on the event under the subtype "New Track" |
| Talk blocked, talk ready | the kanban state of a talk changes | the followers of the talk | internal messages under the matching subtype |
| Talk stage message | a talk enters a stage carrying a template | the speaker | an internal note using the light notification layout |
| Talk reminder | the visitor asks for it | the posted address, or the address of the signed-in user | the reminder template with the calendar links |
| Losing booth reservation | a competing reservation wins | the salesperson of the losing order | a message on the losing order, followed by its cancellation |
| Slot or ticket changed on a sold seat | the slot or ticket of a registration attached to an order changes | the event responsible, or the salesperson, or the administrator | a warning activity on the order |
| Remaining seats changed | a counter registration is created or written | every open counter session | a live broadcast carrying, per event, the remaining seats of the event, of each ticket and of each slot |
| Order origin link | a registration is created with a sales order | the followers of the registration | an internal note linking to the order |

## 6. Screens

Each screen is described by the data it shows and the buttons it offers.

### 6.1 Event list

Columns: name, venue, organiser (hidden by default), responsible, company (multi-company only), start date, end date, tags (hidden by default), total attendees with a sum, attendees present with a sum, maximum seats with a sum (hidden by default), reserved seats with a sum (hidden by default), stage, and the activity exception marker. Multiple records may be edited at once. A sample data set is shown when the list is empty.

### 6.2 Event kanban

Grouped by stage by default, every stage shown. Each card carries a coloured date block on the left showing the day, the month and year, the start time and, when the end differs, the end day and month; and on the right the name, the venue, the attendee figures and the tags. New records are created through a quick form asking only for the name and the date range.

### 6.3 Event form

- **Header:** a `Registration Desk` button, visible only when the event has at least one taken seat, opening the desk screen for that event; and the stage status bar, clickable.
- **Buttons above the sheet:** `Registration`, opening the attendee statistics of that event; and an attendee counter showing `seats_taken` and opening the attendee list.
- **Top-right marker:** the kanban state, editable.
- **Title:** the event name.
- **Left column:** the date range with the display time zone; the multi-slot switch with a link showing the slot count and opening the slot calendar; the language; the template; the tags.
- **Right column:** the organiser; the responsible (internal users only); the company, described as "Visible to all" when empty; the venue, described as "Online if not set"; the online-event link, shown only when there is no venue; and the seat limit, phrased as `Limit Registrations` then `to <maximum> Attendees`, with the suffix `per slot` when the event uses slots. The maximum becomes required as soon as the limit is switched on.
- **Tabs:**
  - *Tickets*: the ticket lines, as a list and a kanban.
  - *Communication*: the schedule lines, editable inline, ordered by a drag handle, showing the template, the interval number (read-only when the unit is Immediately), the unit, the trigger, the number sent and a status icon; a line in error is highlighted. The scheduled date is visible to the technical role only.
  - *Questions*: the linked questions, restricted to reusable ones, ordered by a drag handle, showing the title, the mandatory flag, the once-per-order flag, the type, the suggested answers (only for a selection question) and a `Stats` button opening the answer breakdown for that question and that event.
  - *Notes & Documents*: the badge layout, the badge background image, the ticket instructions and the internal note.
- **Discussion thread** at the bottom.

### 6.4 Event search panel

Filters: `My Events`; `Upcoming/Running`; `Start Date` as a date filter; `Archived`; `Online` (events with no venue). Groupings: responsible, template, stage (Registration Desk and above), start date, venue. Searchable fields include the name, the venue through the address search helper, the tags, the responsible and the organiser.

### 6.5 Attendee list

Ordered by creation date descending, expanded by default. Columns: registration date, attendee name, contact (hidden by default), electronic mail address, telephone, company name (hidden by default), event (hidden when the list is opened from one event), slot, ticket, activities, status as a coloured badge, company (multi-company, hidden by default, editable only while unconfirmed). Buttons per row, with their guards:

| Button | Visible when |
|---|---|
| `Registered` | the record is active and the state is `draft` |
| `Mark as Attending` | the record is active and the state is `open` |
| `Cancel` | the record is active and the state is `open` or `draft` |

Multiple records may be edited at once.

### 6.6 Attendee form

- **Header buttons:** `Send by Email` (active record, state `open` or `done`), `Registered` (active, `draft`), `Attended` (active, `open`), `Cancel Registration` (active, state not already `cancel`); then the status bar showing `Registered` and `Attended`, clickable.
- **Attendee group:** name, electronic mail address, telephone with a text-message action, company name.
- **Event group:** event, slot, ticket, and the event dates.
- **Questions page:** one row per answer.
- **Archived ribbon** when the record is archived.

### 6.7 Attendee search panel

Filters: `Ongoing Events`; `Taken` (`state IN ("open","done")`); `Unconfirmed`; `Registered`; `Attended`; the date filters `Registration Date`, `Event Start Date` and `Attended Date`; `Last 30 days`; `Archived`; plus the activity filters. Groupings: contact, event, slot, ticket, status, registration date by week or by day, campaign, medium, source, and the user-defined attendee fields. Searchable: the registration identifier, a combined search over the attendee name, electronic mail address and company name, the company, the contact, the ticket and the event.

### 6.8 Attendee kanban of the registration desk

One card per attendee showing the name, the event, the company, the ticket and the status badge. Clicking a card opens a dialog that states the outcome of the check-in and offers `Continue`, which marks the attendee as attended.

### 6.9 Registration desk screen

A full-screen screen with two choices: scan a badge with the camera, or pick an attendee from the list. A scan produces one of the seven outcomes of [workflows.md](workflows.md#18-check-an-attendee-in-at-the-registration-desk); an invalid code raises a transient error notice. The heading shows the event name, the city and the country when the screen was opened for one event.

### 6.10 Slot calendar

A calendar of the slots of one event, with a list and a form. The selectable days are limited to the day range of the event in the display time zone. Creating a slot asks for the start time, the end time, the time zone and an optional colour. The list shows the date, the start hour, the end hour and the colour.

### 6.11 Ticket views

Inside the event form, the ticket list shows the name, the description, the sales window, the seat maximum (labelled per slot on a multi-slot event), the per-order limit, the price and the derived counters (reserved, used, available). A kanban variant shows one card per ticket with the colour strip.

### 6.12 Attendee analysis

Opened on graph, pivot, kanban, list and form, filtered on the last thirty days of creation and on taken seats, grouped by day and by event, with the weekly grouping hidden. Opened from an event, it is restricted to that event and keeps the same filters.

### 6.13 Revenue analysis

Opened on graph and pivot over Event Sales Report, filtered on priced tickets and on the event start date, measuring the row count, the untaxed revenue and the revenue. Dimensions available: event template, event, event start and end date, slot, ticket, ticket price, registration date, registration status, attendee name, product, order, order date, customer, order status, salesperson, invoice address, payment status, company and, with online ticketing, whether the event is published.

### 6.14 Answer breakdown

Over Event Registration Answer: a list of the answers with the question, the attendee, the contact, the event and the value; a graph and a pivot counting the chosen suggestions of selection questions. Filters by question, by event and by question type.

### 6.15 Booth views

- *Kanban*, grouped by state by default, one card per booth with its name, its category, its renter and its state.
- *List* with the name, the category, the renter, the contact block, the state and, for a salesperson, the order.
- *Form* with the renter block, the state, the sponsor block when the category creates sponsors, and the buttons `Confirm`, `Set to paid` and `View Sales Order`.
- *Graph* and *pivot* for counting booths per state and per category.

### 6.16 Programme views

- *Talk kanban*, grouped by stage, every stage shown, cards carrying the title, the speaker, the room, the date, the tags, the kanban state and its legend.
- *Talk list* with the title, the speaker, the date, the duration, the room, the stage and the publication flag.
- *Talk calendar* on the start date.
- *Talk form* with the speaker block, the timing block, the room, the tags, the website block (picture, action button, video link), the quiz button and the wish-list counter.
- *Talk graph* and *activity* views.

### 6.17 Public screens

| Screen | Content | Actions |
|---|---|---|
| Event list | the published, visible events with their cover, name, subtitle, dates and venue; the search box; the date, country, template and tag filters with their counts | open an event, page through the list |
| Event page | the cover, the description, the practical block, the calendar links, the slots and the tickets | choose a slot, choose quantities, register |
| Attendee form | one block per seat plus the order-level block, with the questions of the event | submit |
| Confirmation page | the created attendees, the calendar links, the ticket download link | download the tickets |
| Booth page | the categories with their price and description, the booths of the selected category, the exhibition map | select booths, continue to the contact form |
| Exhibitor list | the sponsors grouped by level, with their logo, name, slogan, country flag and "open now" marker | filter, open a sponsor |
| Sponsor page | the logo, the description, the opening hours, the contact block and the other exhibitors | contact the sponsor |
| Talk list | the talks grouped as live, soon, per day and `Coming soon`, with the speaker, the room, the time and the reminder control | search, filter by tag, filter by wish list, toggle a reminder |
| Agenda | the grid of quarter-hour rows by room, per day | open a talk |
| Talk page | the abstract, the speaker block, the video, the action button, the quiz and the suggested talks | watch, take the quiz, set a reminder, add to a calendar |
| Proposal form | the talk fields, the speaker fields, the optional contact block and the tags | submit a proposal |
| Leaderboard | the ranked visitors with their points, the top three highlighted, the reader highlighted | search a name, page through |

---

## Reconciliation notes

1. **Provenance.** The operations, endpoints, documents, exported files, notifications and screens
   come from version M. Version P announced the same subjects — menus, views, named operations,
   routes, printable documents, message templates, external integrations, import and export — and
   each is covered: the menu tree is in [`configuration.md`](configuration.md), the views are
   described here as screens, and the printable documents and calendar exports are in sections 3 and
   4 of this file.
2. **Identifiers.** Operation names and route paths are reproduced exactly; the fields they read and
   write are named by their storage names.
3. **Prepared mailings.** The three operations that open a prepared mailing were listed together
   without their defaults. They are now one row each, with the screen title, the entity the mailing
   is prepared on, the pre-filled subject and the default selection.

