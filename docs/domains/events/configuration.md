# Configuration of the Events domain

Every switch, parameter, default, piece of shipped master data, scheduled job and access group of the domain, with its data type, its default value and its effect. A replacement must be able to reach the same behaviour by setting the same things.

## 1. Capability packages

The domain is delivered as a core package plus bridges. Each bridge only adds behaviour; nothing is removed when a bridge is absent.

| Capability package | Adds |
|---|---|
| Events Organization | The whole core: events, slots, tickets, registrations, questions and answers, automatic communications, stages, tags, templates, badges and tickets, the registration desk, the answer breakdown report. |
| Events Product | The product on a ticket, the ticket price, the tax-included and discounted prices, the event value of the product service tracking, and the derived sale status of a registration. |
| Events Sales | Selling tickets on a sales order: the event configurator, the attendee editor, the registration creation on confirmation, the revenue analysis and the total sales figure of an event. |
| Events Booths | The booth catalogue: booth categories, booths, booth templates and the booth counters of an event. |
| Events Booths Sales | Selling booths on a sales order: the booth product, the booth prices, the booth configurator, pending reservations and the paid flag. |
| Text Message on Events | A second kind of automatic communication, sent as a text message. |
| Point of Sale - Event | Selling tickets at the shop counter, with live seat broadcasting, attendee capture and badge printing. |
| Point of Sale - Event Sale | Refines the sale status of a counter registration using the counter order state. |
| Events (website) | The public event list and page, the registration form, the per-event menu tree, the event search, the visitor link and the publication rules. |
| Online Event Ticketing | Buying tickets through the cart and the checkout, with the seat caps of the cart and the payment-time verification. |
| Online Event Booths | The public booth catalogue and the booth booking form. |
| Online Event Booth Sale | Booking booths through the cart and the checkout. |
| Event Exhibitors | Sponsors and exhibitors, sponsorship levels, exhibitor pages and opening hours. |
| Booths/Exhibitors Bridge | Creating a sponsor automatically when a booth of a sponsoring category is booked. |
| Booths Sale/Exhibitors Bridge | Carrying the sponsor details through a booth reservation made online. |
| Advanced Events | The programme: talks, review stages, rooms, tags, the agenda, wish lists, the progressive application name and icon. |
| Live Event Tracks | Embedded video on a talk, the live and replay states and the next-talk suggestion. |
| Quizzes on Tracks | Quizzes on talks, points, the visitor link fields and the community leaderboard. |
| Quiz on Live Event Tracks | Shows the quiz invitation inside the next-talk suggestion of a live talk. |
| Events Lead Generation | Turning attendees into leads, with rules, triggers and batch regeneration. |
| Spreadsheet dashboard for events | A shipped analysis dashboard for events, restricted to the Event Administrator group. |

## 2. Settings shown on the configuration screen

| Setting | Type | Default | Effect |
|---|---|---|---|
| Tickets with Sale | boolean (installs a package) | off | Installs Events Sales. |
| Online Ticketing | boolean (installs a package) | off | Installs Online Event Ticketing. |
| Tickets with Point of Sale | boolean (installs a package) | off | Installs Point of Sale - Event. |
| Booth Management | boolean (installs a package) | off | Installs Events Booths. |
| Tracks and Agenda | boolean (installs a package) | off | Installs Advanced Events. Switching it **off** in the form also clears Live Mode and Quiz on Tracks, because those depend on it and would otherwise bring it back. |
| Live Mode | boolean (installs a package) | off | Installs Live Event Tracks. |
| Quiz on Tracks | boolean (installs a package) | off | Installs Quizzes on Tracks. |
| Advanced Sponsors | boolean (installs a package) | off | Installs Event Exhibitors. |
| Use Event Barcode | boolean | off | Stored in the parameter `event.use_event_barcode`. When off, the linear barcode is omitted from badges and tickets; the quick response code is always printed. |
| Barcode Nomenclature | many_to_one to Barcode Nomenclature | the one of the company | Read and written through the company. Governs how a scanned code is interpreted. |
| Google Maps static Application Programming Interface | boolean | true when both the key and the secret are already stored, false otherwise | Enables the signed static map picture of a venue address. Switching it off clears both the key and the secret. |
| Google Maps Application Programming Interface key | text | empty | Stored in the parameter `google_maps.signed_static_api_key`. |
| Google Maps Application Programming Interface secret | text | empty | Stored in the parameter `google_maps.signed_static_api_secret`. Must decode as a web-safe base-encoded value (`EV-RULE-091`). |
| Events App Name | text | `<website name> Events` | Read and written through the website. The name of the progressive web application of the event site; required (`EV-RULE-090`). |

## 3. Configuration parameters

| Parameter | Type | Default when absent | Effect |
|---|---|---|---|
| `event.use_event_barcode` | text, read as the literal `True` | absent, read as off | Prints the linear barcode on badges and tickets, and turns on the barcode elements of the registration desk. |
| `event.event_mail_async` | text, any non-empty value is true | absent, read as off | When on, creating a registration only wakes the communication job and the outgoing-message job instead of sending the attendee-based messages immediately. Use it when registrations arrive in bursts. |
| `mail.batch_size` | integer | 50 (also used when the stored value is zero) | Number of attendees processed between two commits inside one run of one schedule. |
| `mail.render.cron.limit` | integer | 1000 (also used when the stored value is zero) | Number of attendees processed by one run of one schedule before it stops and re-wakes the job. |
| `google_maps.signed_static_api_key` | text | absent | The key used to build the signed static map address of a contact. Without it, no map picture is produced. |
| `google_maps.signed_static_api_secret` | text | absent | The signing secret. Without it, no map picture is produced. |
| `website_event_sale.require_billing_details_for_events` | text read as a boolean | `False` | When true, an order that contains only ticket lines still asks for the full billing address. |

## 4. Scheduled and periodic jobs

| Job | Frequency | Runs as | What it does |
|---|---|---|---|
| Event: Mail Scheduler | every 24 hours; first call 15 minutes after installation; additionally woken whenever a schedule date is recomputed, whenever a global run hits its render limit, and whenever a registration is created while the asynchronous parameter is on | the platform root user | Selects and executes the due communication schedules (see [workflows.md](workflows.md#16-run-the-communication-scheduler)). Commits after each schedule. |
| Generate Leads based on Rules (external identifier `event_crm.ir_cron_generate_leads`) | every day; woken immediately when a large regeneration is requested and when a batch remains unfinished | the job user | Processes at most 100 lead-generation requests, each in batches of at most 200 attendees, resuming from the stored identifier. |
| Housekeeping: move ended events | with the periodic housekeeping pass | the job user | Moves every event whose end date has passed and whose stage is not an ending stage into the first ending stage by sequence. |
| Housekeeping: purge inactive visitors | owned by the website domain | the job user | Never deletes a visitor that has at least one registration or at least one talk link (`EV-RULE-095`). |

## 5. Access groups

The domain defines one privilege named **Events**, placed in the Marketing category with sequence 18, holding three groups. Each group includes the previous one.

| Group | Sequence | Includes | Intended holder |
|---|---|---|---|
| Registration Desk | 10 | the internal user group | Staff at the desk and at the counter. |
| User | 20 | Registration Desk | Event organisers. |
| Administrator | 30 | User | Event configuration owners. The platform root user and the default administrator hold it. |

Two further links exist:

- the Salesperson group of the sales domain **includes** Registration Desk, so that every salesperson can read events and manage attendees;
- the Event Administrator group **includes** the restricted website editor group, so that an event administrator can edit the event pages.

### 5.1 What each group may do

| Entity | Registration Desk | User | Administrator | Other groups |
|---|---|---|---|---|
| Event Template | read | read | create, read, update, delete | — |
| Event Template Ticket | read | read | create, read, update, delete | — |
| Event Booth Template | read | read | create, read, update, delete | — |
| Event | read | create, read, update | create, read, update, delete | shop counter operator: read. Anonymous, portal and internal users: read (website package). |
| Event Slot | read | create, read, update, delete | inherited | Anonymous, portal and internal users: read (website package). |
| Event Ticket | read | create, read, update, delete | inherited | shop counter operator: read. Anonymous, portal and internal users: read (website package). |
| Event Registration | create, read, update | inherited | create, read, update, delete | shop counter operator: create, read, update. |
| Event Registration Answer | create, read, update, delete | inherited | inherited | — |
| Event Question | — | create, read, update, delete | create, read, update, delete | Anonymous, portal and internal users: read (website package). |
| Event Question Answer | read, update | create, read, update, delete | inherited | Anonymous, portal and internal users: read (website package). |
| Event Automated Mailing | read | create, read, update, delete | inherited | — |
| Registration Mail Scheduler | read | read | create, read, update, delete | — |
| Slot Mail Scheduler | read | read | create, read, update, delete | — |
| Mail Scheduling on Event Category | read | read | create, read, update, delete | — |
| Event Stage | read | read | create, read, update, delete | — |
| Event Tag Category | read | create, read, update, delete | inherited | Anonymous, portal and internal users: read. |
| Event Tag | read | create, read, update | create, read, update, delete | Anonymous, portal and internal users: read. |
| Event Booth Category | read | read | create, read, update, delete | Anonymous users: read (website booth package). |
| Event Booth | read | create, read, update, delete | create, read, update, delete | Anonymous, portal and internal users: read (website booth package). |
| Event Booth Registration | read | create, read, update, delete | inherited | Salesperson: create, read, update, delete. |
| Event Sponsor Level | — | — | create, read, update, delete | — |
| Event Sponsor | — | — | create, read, update, delete | Anonymous, portal and internal users: read. |
| Event Track | — | create, read, update | create, read, update, delete | Anonymous, portal and internal users: read. |
| Event Track Stage | — | — | create, read, update, delete | Anonymous, portal and internal users: read. |
| Event Track Tag | — | create, read, update | create, read, update, delete | Anonymous, portal and internal users: read. |
| Event Track Tag Category | — | create, read, update, delete | inherited | — |
| Event Track Location | — | create, read, update | create, read, update, delete | — |
| Track / Visitor Link | — | — | create, read, update, delete | — |
| Quiz, Content Quiz Question, Question's Answer | — | create, read, update, delete | inherited | — |
| Website Event Menu | — | create, read, update, delete | inherited | Anonymous, portal and internal users: read. |
| Website Visitor | read, update | inherited | inherited | — |
| Event Sales Report | — | — | read | — |
| Event Configurator | — | — | — | Salesperson: create, read, update. |
| Event Booth Configurator | — | — | — | Salesperson: create, read, update. |
| Edit Attendee Details on Sales Confirmation | — | — | — | Salesperson: create, read, update. |
| Edit Attendee Line on Sales Confirmation | — | — | — | Salesperson: create, read, update, delete. |
| Text Message Template | — | — | create, read, update, delete | — |

Two rights are worth noting because they are exceptions:

- a Registration Desk user may **post a message on an Event** they can only read (`EV-RULE-099`);
- an Event Administrator may create, update and delete **text message templates**, but only those written on Event or Event Registration.

### 5.2 Record rules

| Rule | Entity | Condition | Applies to |
|---|---|---|---|
| Event: multi-company | Event | `company_id IN (companies of the reader) OR company_id IS NULL` | everyone |
| Event/Registration: multi-company | Event Registration | `company_id IN (companies of the reader) OR company_id IS NULL` | everyone |
| Event/Ticket: multi-company | Event Ticket | `event_id.company_id IN (companies of the reader) OR event_id.company_id IS NULL` | everyone |
| Event Sales Report multi-company | Event Sales Report | `company_id IN (companies of the reader) OR company_id IS NULL` | everyone |
| Event: published read | Event | `website_published = true` | anonymous and portal readers, read only |
| Event Ticket: published read | Event Ticket | `event_id.website_published = true` | anonymous and portal readers, read only |
| Event Slot: published read | Event Slot | `event_id.website_published = true` | anonymous and portal readers, read only |
| Event Tag: colour and published category | Event Tag | `category_id.website_published = true AND color NOT IN (empty, 0)` | anonymous and portal readers, read only |
| Event Question: published event | Event Question | at least one linked event is published | anonymous, portal and internal readers, read only |
| Event Question: event user | Event Question | always true | Registration Desk and above, read only through this rule |
| Event Question Answer: published event | Event Question Answer | at least one event of the question is published | anonymous, portal and internal readers, read only |
| Event Question Answer: event user | Event Question Answer | always true | Registration Desk and above, read only through this rule |
| Event Booth: published read | Event Booth | `event_id.website_published = true` | anonymous and portal readers, read only |
| Event Sponsor: published read | Event Sponsor | `website_published = true` | anonymous and portal readers, read only |
| Event Tracks: published | Event Track | `website_published = true` | anonymous and portal readers, read only |
| Event Track Tag: coloured | Event Track Tag | `color NOT IN (empty, 0)` | anonymous and portal readers, read only |
| Text Message Template: event manager | Text Message Template | `model IN ("event.event", "event.registration")` | Event Administrator, for create, update and delete only |

## 6. Shipped master data

### 6.1 Event stages

| Name | Sequence | Description | End stage | Folded |
|---|---|---|---|---|
| New | 1 | `Freshly created` | no | no |
| Booked | 2 | empty | no | no |
| Announced | 3 | `The event has been publicly announced` | no | no |
| Ended | 5 | `Finished events. the system will automatically move them to this stage once their end date has passed.` | yes | yes |

### 6.2 Default questions

| Title | Type | Mandatory | Default | Reusable |
|---|---|---|---|---|
| Name | Name | yes | yes | yes |
| Email | Email | yes | yes | yes |
| Phone | Phone | no | yes | yes |

Every new event and every new template links these three questions by default.

### 6.3 Default communication schedules of a new template and a new event

| Interval | Unit | Trigger | Template |
|---|---|---|---|
| 0 | Immediately | After each registration | Event: Registration Confirmation |
| 1 | Hours | Before the event starts | Event: Reminder |
| 3 | Days | Before the event starts | Event: Reminder |

### 6.4 Booth categories

| Name | Sequence | Price with the sales bridge | Creates a sponsor | Sponsorship level | Sponsor kind |
|---|---|---|---|---|---|
| Standard Booth | 1 | 100 | no | — | — |
| Premium Booth | 2 | 500 | yes | Silver | Exhibitor |
| Very Important Person Booth | 3 | 1000 | yes | Gold | Online Exhibitor |

All three point at the shipped Event Booth product and carry an illustration.

### 6.5 Sponsorship levels

| Name | Sequence | Ribbon |
|---|---|---|
| Gold | 1 | Gold |
| Silver | 2 | Silver |
| Bronze | 3 | Bronze |

A **lower** sequence is a **higher** level; the default level proposed for a new sponsor is the one with the **highest** sequence, that is the lowest level.

### 6.6 Talk review stages

| Name | Sequence | Colour | Visible in agenda | Fully accessible | Cancelled | Folded | Template sent |
|---|---|---|---|---|---|---|---|
| Proposal | 1 | 1 | no | no | no | no | — |
| Confirmed | 2 | 2 | no | no | no | no | Event: Track Confirmation |
| Announced | 3 | 3 | yes | no | no | no | — |
| Published | 4 | 4 | yes | yes | no | no | — |
| Refused | 5 | 5 | no | no | no | yes | — |
| Cancelled | 6 | — | no | no | yes | yes | — |

### 6.7 Message templates

| Template | Written on | Subject or body | Attachment | Notes |
|---|---|---|---|---|
| Event: Registration Badge | Event Registration | `Your badge for {{ event name }}` | the Badge report | Sent after someone registers; the attachment is deleted after sending. |
| Event: Registration Confirmation | Event Registration | `Your registration at {{ event name }}` | the Full Page Ticket report | The default "after each registration" message. |
| Event: Reminder | Event Registration | `{{ event name }}: {{ relative date phrase }}` | none | The default "before the event" message. |
| Event: Track Confirmation | Event Track | `Confirmation of {{ talk title }}` | none | Sent to the speaker when a talk reaches the Confirmed stage; deleted after sending. |
| Add reminder via email | Event Track | `Add talk reminder: {{ talk title }}` | none | Sent on demand from the talk page, carrying the calendar links. |

Every one of them takes its sender from the first of: the organiser of the event, the company of the event, the acting user; and renders in the language of the event, falling back to the language of the contact. The first three address the default recipients of the registration rather than its contact, which matters when one contact books several seats.

| Text message template | Written on | Body |
|---|---|---|
| Event: Registration | Event Registration | `<organiser or company name>: We are happy to confirm your registration for the <event name> event.` |
| Event: Reminder | Event Registration | `Ready for "<event name>" <relative date phrase>?` then either `It starts at <start time in the event time zone>`, plus `, at <venue on one line>` when a venue exists, plus `. See you there!`, or `Join us on <base address>/event/<event identifier>!` when the event has no venue and the website package is installed. |

### 6.8 Products and product categories

| Record | Values |
|---|---|
| Product category "Events" | child of the Services category |
| Product "Event Registration" | service, sales price 30.00, cost 10.00, unit of measure Units, category Events, service tracking = event, invoicing policy = ordered quantities, no sales description; with the counter bridge: available at the counter, in the counter category Events |
| Product "Event Booth" | service, sales price 100.00, category Events, service tracking = event booth, invoicing policy = ordered quantities, not purchasable, no sales description |
| Counter category "Events" | used to group the ticket products on the counter screen |

### 6.9 Paper formats

| Format | Used by | Page | Margins | Resolution |
|---|---|---|---|---|
| Event badge | the Badge reports | A4 portrait, shrinking disabled | all margins 0 | 96 dots per inch |
| Event full page ticket | the Full Page Ticket reports | A4 portrait, shrinking disabled | top 0, bottom 8, left 0, right 0 | 96 dots per inch |
| Event full page ticket, with exhibitors installed | the same | the bottom margin becomes 29, to leave room for the sponsor strip | | |

### 6.10 Thread message subtypes

| Subtype | Written on | Default | Meaning |
|---|---|---|---|
| Booth Booked | Event | off | A booth of the event has been booked. |
| Event published | Event | off | The event became publicly visible. |
| Event unpublished | Event | off | The event left public visibility. |
| New Track | Event | off | A talk was proposed or created. |
| Track Blocked | Event Track | off, internal | The talk was marked blocked. |
| Track Ready | Event Track | on, internal | The talk was marked ready for the next stage. |

### 6.11 Website data

| Record | Value |
|---|---|
| Top menu entry | `Events`, address `/event`, sequence 30, under the main menu |
| Saved filter "Upcoming Events" | events whose start date is in the future and which are visible on the website, ordered by start date ascending |
| Saved filter "Upcoming and Ongoing Events" | events that are not finished and which are visible on the website, ordered by start date ascending |
| Content block "Upcoming Events" | shows the name and the subtitle, at most 16 events, from the first filter |
| Content block "Upcoming and Ongoing Events" | the same from the second filter |
| Guided tour | one tour named `event_tour`, sequence 210, ending with the message *"Great! Now all you have to do is wait for your attendees to show up!"* |
| Opening action | after installing the website package, the back office opens the public event list once |

### 6.12 Dashboard

One shipped analysis dashboard named **Events**, placed in the Marketing dashboard group with sequence 60, built on the Event entity, published, and restricted to the Event Administrator group.

## 7. Master-data prerequisites

A replacement needs the following before the domain can be used at all:

1. **At least one Event Stage.** A new event takes the first stage by sequence; with no stage the field stays empty and the pipeline view is unusable. At least one stage must be flagged as an ending stage for the automatic closing to work.
2. **At least one Event Question flagged as default**, when the shipped defaults are wanted on every new event; otherwise new events start with no question.
3. **A ticket product** whose service tracking is the event value, as soon as tickets must be sold. A ticket cannot be created without a product once the product bridge is installed.
4. **A booth product** whose service tracking is the booth value, as soon as booths must be sold.
5. **At least one Booth Category**, since a booth template preselects the single existing category when exactly one exists, and a booth always needs one.
6. **At least one Sponsorship Level**, since a sponsor requires one and the default is the level with the highest sequence.
7. **At least one Talk Review Stage**, since a talk requires one and takes the first by sequence.
8. **A company contact**, used as the default organiser and the default venue of a new event.
9. **An outgoing message channel**, because every automatic communication is queued rather than sent synchronously.
10. **A website**, when the public pages are used; the website of an event must belong to the company of the event.

## 8. Menu structure

| Menu | Parent | Sequence | Opens | Restricted to |
|---|---|---|---|---|
| Events | root | 125 | — | Registration Desk |
| Events > Events | Events | 1 | the event list, calendar, kanban, form, pivot, graph and activity views | Registration Desk |
| Events > Registration Desk | Events | 30 | the full-screen barcode interface | Registration Desk |
| Events > Tracks | Events | 40 | the talk list | hidden by default |
| Events > Reporting | Events | 50 | — | User |
| Events > Reporting > Attendees | Reporting | 4 | the attendee analysis, opened on graph, pivot, kanban, list and form, filtered on the last month of creation, on taken seats and grouped by day and by event | User |
| Events > Reporting > Revenues | Reporting | 5 | the revenue analysis, opened on graph and pivot, filtered on priced tickets and on the event start date, measuring the count, the untaxed revenue and the revenue | User |
| Events > Configuration | Events | 99 | — | User |
| Configuration > Settings | Configuration | 0 | the configuration screen of the domain | system administrator |
| Configuration > Event Templates | Configuration | 1 | the template list | User |
| Configuration > Event Stages | Configuration | 2 | the stage list | User |
| Configuration > Event Tags Categories | Configuration | 3 | the tag category list | User |
| Configuration > Event Questions | Configuration | 4 | the question list with its filters | User |
| Configuration > Mail Schedulers | Configuration | 10 | every communication schedule, read only for creation | hidden by default |
| Configuration > Booth Categories | Configuration | 20 | the booth category list | User |
| Configuration > Booths | Configuration | 21 | every booth | hidden by default |
| Configuration > Track Stages | Configuration | 30 | the talk stage list | hidden by default |
| Configuration > Track Locations | Configuration | 32 | the room list | User |
| Configuration > Track Tag Categories | Configuration | 33 | the talk tag category list | hidden by default |
| Configuration > Track Tags | Configuration | 34 | the talk tag list | hidden by default |
| Configuration > Track Visitors | Configuration | 38 | the talk and visitor links, read only for creation | hidden by default |
| Configuration > Sponsor Levels | Configuration | 40 | the sponsorship level list | hidden by default |
| Configuration > Quizzes | Configuration | 50 | the quiz list, read only for creation | hidden by default |
| Configuration > Quiz Questions | Configuration | 55 | the quiz question list, read only for creation | hidden by default |
| Configuration > Website Menus | Configuration | 99 | every per-event menu entry, read only for creation | hidden by default |
| Website content > Events | the website content menu | 40 | the event page list, with a shortcut that creates an event through the quick form | website editor |

"Hidden by default" means the menu exists but is reserved for the technical role; it is not shown to an ordinary Event User.

---

## Reconciliation notes

1. **Provenance.** The settings, parameters, jobs, groups, record rules and shipped master data come
   from version M, which was the only version that carried a configuration document. Version P
   announced the same subjects in its reading order and its dependency table, and every one of them
   is present here.
2. **Names.** The entities are named with the full names of the entity dictionary of this repository,
   so the access matrix reads Event Automated Mailing, Registration Mail Scheduler, Event Sponsor
   Level and Track / Visitor Link where version M read Event Communication, Event Communication
   Registration, Event Sponsor Type and Event Track Visitor.
3. **The lead-generation job.** Its label is given without the abbreviation version M used, together
   with its external identifier `event_crm.ir_cron_generate_leads`, which is the contractual string.
