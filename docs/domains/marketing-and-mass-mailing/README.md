# Marketing and Mass Mailing

This domain designs, schedules, sends and measures one-to-many commercial communications. It owns
the Mass Mailing record — an electronic mail or a text message written once and delivered to a
computed audience — together with the audience itself (Mailing List, Mailing Contact and the
Mailing Subscription that joins them with an opt-out flag, a moment and a reason), the per-recipient
delivery record (Mailing Trace) that carries the whole delivery, open, click, reply, bounce and
failure history, the shortened-link machinery that measures clicks (Link Tracker, Link Tracker Code,
Link Tracker Click), the campaign-tracking vocabulary that attributes every business record to a
marketing effort (Campaign, Campaign Source, Campaign Medium, Campaign Stage, Campaign Tag and the
two abstract behaviours that stamp those values on other records), the shareable personalised image
feature (Marketing Card Campaign, Marketing Card, Marketing Card Template) and the social-network
account fields of the company. It also owns the public pages a recipient reaches from a delivered
message: the subscription-management page, the confirmation pages, the click-redirection endpoint,
the open-tracking endpoint and the text-message opt-out pages.

The domain posts no journal entry of its own. It nevertheless displays three monetary or
near-monetary indicators that belong to other domains, and it consumes a paid external sending
service; both boundaries are specified precisely in [accounting-effects.md](accounting-effects.md).

---

## 1. The questions this domain answers

1. What are we sending, to whom, from which address, with which reply behaviour, and when does it
   leave?
2. Which records are the recipients: the contacts of one or more Mailing Lists, or any other entity
   the system allows to be mailed, restricted by a stored condition that may be saved and reused?
3. Who must never be contacted — globally blocked addresses and numbers — who has opted out of
   which list, and who has already received the same campaign?
4. How many messages were queued, sent, delivered, opened, clicked, replied to, bounced, cancelled
   and failed, and what percentage does each of those represent?
5. Which of two or more competing versions of a message performs best on a chosen indicator, and
   how is the winning version then sent to the rest of the audience automatically?
6. Which links inside a message were clicked, how many times, from which country, and by which
   recipient?
7. Which marketing effort produced a given business record — a lead, a quotation, an invoice, a
   registration — through the campaign, source and medium stamped on it?
8. How do recipients unsubscribe themselves, block themselves entirely, re-subscribe, and tell us
   why?
9. How do website visitors subscribe to a public list from a page block, a pop-up or the online-shop
   checkout?
10. How do we turn a record — a contact, a talk, a booth, a registration — into a personalised
    shareable image that the person can post on a social network, and how many of those images were
    viewed and shared?

---

## 2. Capabilities covered

| Capability | Summary |
|---|---|
| Mailing design | Subject, preview sentence, rich-text body built from a chosen theme and a library of building blocks, attachments, inline images converted to stored files, mobile preview. |
| Audience selection | A recipient entity chosen from the set of mailing-enabled entities, a stored condition, reusable saved conditions, or a set of Mailing Lists that resolve to their Mailing Contacts. |
| Exclusion rules | Globally blocked addresses and numbers, per-list opt-out, already-contacted detection inside the same mailing or the same comparison-test campaign, duplicate-address suppression inside a single run. |
| Scheduling | Send immediately or at a stated moment; a queue state; a scheduled job that drains the queue in batches and can be woken early. |
| Sending algorithm | Fixed-size batches, per-record rendering, per-record delivery record, exclusion checks with an exact failure code, automatic commit between batches, resumability after an interruption. |
| Delivery measurement | Nine delivery statuses per recipient, timestamps for sent, opened, replied and last click, bounce reason, twenty-nine failure codes, aggregated counters and five percentage indicators. |
| Link tracking | Every outgoing link is replaced by a short coded address; each visit is recorded with the network address, the resolved country, the campaign and, when the visitor came from a message, the exact recipient. |
| Comparison testing | Two or more versions of a message each addressed to a random percentage of the audience, with a winner chosen manually or automatically on an indicator at a stated moment, and then sent to the whole remaining audience. |
| Campaign tracking | A campaign with stages, tags, a responsible person and aggregated indicators; a source and a medium; parameters captured from a web address into cookies and stamped on newly created records. |
| Text message marketing | The same mailing life cycle with a plain-text body, phone-number sanitising, per-message opt-out link and code, and delivery reports from the sending service. |
| Subscription management | A public page listing the recipient's subscriptions, public lists they may join, a self-exclusion button, a re-inclusion button, and a feedback form with a closed list of reasons. |
| Website subscription | A page block and a pop-up that subscribe a visitor to a chosen public list, plus an option on the online-shop checkout. |
| Marketing cards | A campaign that renders a personalised image per record from a template, a preview page, a redirection page that serves social-preview metadata to crawlers, and share and visit counters. |
| Reporting | A read-only analytical view over delivery records, an opt-out-reason report, per-campaign indicators, and a statistics message sent to the person responsible one day after sending. |

---

## 3. Actors

| Actor | Description |
|---|---|
| Marketing User | Member of the Email Marketing user group. Creates, edits, tests, schedules, cancels and duplicates mailings; manages Mailing Lists, Mailing Contacts, Mailing Subscriptions, Mailing Filters, opt-out reasons, campaigns, sources, mediums, stages and link trackers; reads the analytical view. |
| Campaign Manager | A Marketing User who also belongs to the campaign-management group. Sees and edits the campaign field on mailings, manages Campaign Stages and Campaign Tags, and reaches the campaign screens. |
| Settings Administrator | Opens the marketing settings page; chooses the dedicated outgoing mail server, enables campaigns, the self-exclusion buttons, the statistics message and the split contact name; manages Marketing Card Templates. |
| Marketing Card User | Creates and edits their own Marketing Card Campaigns and sends them. |
| Marketing Card Manager | Reads and edits every Marketing Card Campaign, manages Marketing Card Campaign Tags and Marketing Cards. |
| Website Designer | Places the subscription block and the subscription pop-up on pages, chooses the target list, and reaches the shortened-link creation page. |
| Recipient (anonymous) | Receives a message; opens it; clicks its links; opens the subscription-management page from the message; unsubscribes, excludes themselves, re-subscribes and gives a reason; views the message in a browser; opts out of text messages. |
| Portal or internal user | The same as the recipient, and may reach the subscription-management page without a token because the system identifies them by their own address. |
| Scheduled job runner | Drains the mailing queue, selects and sends comparison-test winners, sends the statistics messages, and removes cancelled outgoing mail and stale marketing card images. |
| External sending service | The outgoing mail relay, the text-message service and the social-network crawlers that fetch preview images. |

---

## 4. Entities owned by this domain

Twenty-eight entities are in scope, of which twenty-two are persistent, two are abstract behaviours
contributed to other entities and six are transient assistants. Every one of them is owned by this
domain: none of them is a shared platform entity.

| Canonical name | Transport name | Kind | Purpose |
|---|---|---|---|
| Mass Mailing | `mailing.mailing` | persistent | One designed message with its audience, schedule, state, comparison-test settings and aggregated indicators. |
| Mailing List | `mailing.list` | persistent | A named audience of Mailing Contacts, optionally shown on the public subscription page. |
| Mailing Contact | `mailing.contact` | persistent | A lightweight addressee holding a name, an address, a mobile number, a country, tags and user-defined fields. |
| Mailing Subscription | `mailing.subscription` | persistent | Membership of one Mailing Contact in one Mailing List with its opt-out flag, moment and reason. |
| Mailing Opt-Out Reason | `mailing.subscription.optout` | persistent | A selectable reason for leaving a list or for self-exclusion, optionally asking for free text. |
| Mailing Trace | `mailing.trace` | persistent | The delivery record of one message to one recipient record, with its status, timestamps and failure code. |
| Mailing Trace Report | `mailing.trace.report` | derived view | Read-only aggregation of delivery records by mailing, campaign, source, state and creation moment. |
| Mailing Filter | `mailing.filter` | persistent | A named, reusable recipient condition bound to one recipient entity. |
| Link Tracker | `link.tracker` | persistent | A target web address plus campaign attribution, exposed through one or more short codes, with a click counter. |
| Link Tracker Code | `link.tracker.code` | persistent | One unique short code pointing at one Link Tracker. |
| Link Tracker Click | `link.tracker.click` | persistent | One recorded visit of a short code, with network address, country, campaign, mailing and recipient. |
| Campaign | `utm.campaign` | persistent | A named marketing effort with a stage, tags, a responsible person, comparison-test settings and aggregated indicators. |
| Campaign Source | `utm.source` | persistent | Where a contact or a click came from; unique by name. |
| Campaign Medium | `utm.medium` | persistent | How a message was delivered; unique by name. |
| Campaign Stage | `utm.stage` | persistent | An ordered step in the campaign board. |
| Campaign Tag | `utm.tag` | persistent | A coloured label on campaigns. |
| Campaign Tracking Mixin | `utm.mixin` | abstract | Adds campaign, source and medium to any entity and fills them from request cookies. |
| Campaign Tracking Source Mixin | `utm.source.mixin` | abstract | Makes an entity own a required Campaign Source whose name is generated and kept unique. |
| Marketing Card Campaign | `card.campaign` | persistent | A personalised-image campaign: design, content mapping, target entity, share link, reward message and counters. |
| Marketing Card | `card.card` | persistent | The rendered image for one record of one Marketing Card Campaign, with its share status. |
| Marketing Card Template | `card.template` | persistent | A shipped image layout with a background and four colours. |
| Marketing Card Campaign Tag | `card.campaign.tag` | persistent | A coloured label on marketing card campaigns. |
| Mailing Contact Import Wizard | `mailing.contact.import` | transient | Pastes many addresses at once into one or more lists. |
| Mailing Contact to List Wizard | `mailing.contact.to.list` | transient | Adds selected contacts to a list, optionally continuing into a new mailing. |
| Mailing List Merge Wizard | `mailing.list.merge` | transient | Merges several lists into a new or existing one, optionally archiving the sources. |
| Mailing Test Wizard | `mailing.mailing.test` | transient | Sends a rendered sample of the mailing to typed addresses. |
| Mailing Schedule Wizard | `mailing.mailing.schedule.date` | transient | Collects a future moment and puts the mailing in the queue. |
| Test Text Message Mailing Wizard | `mailing.sms.test` | transient | Sends a rendered sample text message to typed numbers. |

Every field of every one of these entities is in [entities.md](entities.md), and each entity there
links to its generated reference page under `docs/references/entities/`.

### 4.1 Entities this domain uses but does not own

| Entity | Owning domain | Why this domain needs it |
|---|---|---|
| Model Definition, Outgoing Mail Server, Request Routing, Configuration Settings, Attachment, Scheduled Job, System Parameter | [platform-foundation](../platform-foundation/) | The entity registry consulted to list mailable entities, the mail servers, the cookie capture, the settings screen, the store for inline images, the job runner and the parameter table. |
| Outgoing Email, Outgoing Text Message, Message Composer Wizard, Text Message Composer Wizard, Text Message Tracker, Email Blacklist, Phone Blacklist, Rendering Mixin, Discussion Thread Mixin | [messaging-and-activities](../messaging-and-activities/) | The transports, the fan-out composer, the rendering engine, the blocked-address and blocked-number registers and the incoming gateway. |
| Contact, Company, Contact Category, Country | [contacts-and-organizations](../contacts-and-organizations/) | The most common recipient entity, the sender identity, the tag catalogue and the country of a click. |
| User | [identity-and-access](../identity-and-access/) | The responsible person, the access groups and the record rules. |
| Website | [website-and-storefront](../website-and-storefront/) | The host used to build per-recipient links, the page blocks and the human-verification service. |
| Lead, Sales Order, Customer Invoice | [customer-relationship-management](../customer-relationship-management/), [sales](../sales/), [accounts-receivable](../accounts-receivable/) | Additional mailable entities and the three business indicators shown on a mailing and on a campaign. |
| Event, Event Registration, Event Track, Event Booth | [events](../events/) | Additional mailable entities, additional marketing card targets and the buttons that open a prepared mailing. |
| Course | [learning-surveys-and-gamification](../learning-surveys-and-gamification/) | The button that opens a prepared mailing addressed to the contacts of the course members. |

The fields and behaviours this domain adds to those entities are specified in section 28 of
[entities.md](entities.md).

---

## 5. Capability packages

The domain is delivered as eighteen capability packages. The names below are the business names; the
technical package keys appear only in the machine-readable catalogues under `schemas/`.

| Package | What it adds |
|---|---|
| Campaign Tracking Trackers | Campaign, Campaign Source, Campaign Medium, Campaign Stage, Campaign Tag, the two abstract behaviours and the cookie capture. |
| Link Tracker | Link Tracker, Link Tracker Code, Link Tracker Click, the redirection endpoint and the two link-shortening operations. |
| Email Marketing | Mass Mailing, Mailing List, Mailing Contact, Mailing Subscription, Mailing Opt-Out Reason, Mailing Trace, Mailing Trace Report, Mailing Filter, the queue, the public subscription pages and the statistics message. |
| Mass Mailing Themes | Eleven shipped body designs. |
| Text Message Marketing | The text-message channel: plain-text body, opt-out code and pages, provider trackers and the text-message failure codes. |
| Marketing Card | Marketing Card Campaign, Marketing Card, Marketing Card Template, Marketing Card Campaign Tag and the three public card endpoints. |
| Social Media | The eight social-network account fields of the company. |
| Link Tracker on the website | The shortened-link creation page and its statistics page. |
| Newsletter Subscribe Button | The subscription page block, the subscription pop-up and the public subscription endpoints. |
| Newsletter Subscribe Text Message Template | The text-message variant of the subscription block. |
| Checkout Newsletter | The newsletter option on the online-shop checkout and the newsletter list of a website. |
| Mass mailing on attendees | Makes Event Registration mailable and adds the attendee and invitee buttons on an event. |
| Event Attendees Text Message Marketing | The text-message variant of those buttons. |
| Mass mailing on track speakers | Makes Event Track mailable and adds the speaker button on an event. |
| Track Speakers Text Message Marketing | The text-message variant of that button. |
| Mass mailing on sale orders | Makes Sales Order mailable, adds the quotation-count and invoiced-amount indicators and their winner criteria. |
| Mass mailing text message on sale orders | Those criteria for the text-message channel. |
| Mass mailing on course members | The button that opens a prepared mailing addressed to the contacts of course members. |

---

## 6. Dependencies on other domains

| Domain | Dependency |
|---|---|
| [messaging-and-activities](../messaging-and-activities/) | Owns the outgoing mail queue, the text-message queue, the composer that performs the actual fan-out, the template rendering engine, the blocked-address and blocked-number registers, the incoming-mail gateway that detects bounces and replies, and the discussion thread on which marketing logs its notes. Marketing cannot send anything without it. |
| [platform-foundation](../platform-foundation/) | Owns the entity registry consulted to list mailable entities, the outgoing mail server records, the scheduled job runner, the system parameter table, the attachment store used for inline images, and the rendering service used to rasterise a marketing card. |
| [identity-and-access](../identity-and-access/) | Owns the User record, the access groups listed in [configuration.md](configuration.md) and the record-level rules restricting marketing card campaigns to their owner. |
| [contacts-and-organizations](../contacts-and-organizations/) | Owns Contact, the most common recipient entity, and Company, which carries the social-network fields and the sender identity. |
| [website-and-storefront](../website-and-storefront/) | Owns the public site whose host is used to build per-recipient links, the page-block mechanism used by the subscription block and pop-up, the online-shop checkout and the human-verification service protecting the subscription endpoint. |
| [customer-relationship-management](../customer-relationship-management/), [sales](../sales/), [events](../events/), [learning-surveys-and-gamification](../learning-surveys-and-gamification/) | Provide additional mailable entities, additional winner criteria and the business indicators shown in the statistics message. Each is optional. |
| [accounts-receivable](../accounts-receivable/), [general-ledger](../general-ledger/), [accounts-payable](../accounts-payable/), [multi-currency](../multi-currency/) | Read-only boundaries only: the invoiced-amount indicator, the currency used to format it and the path by which the purchase of sending credit reaches the ledger. See [accounting-effects.md](accounting-effects.md). |
| [recruitment](../recruitment/) | Holds a deletion guard on a Campaign used by a recruitment source; the message is reproduced in [business-rules.md](business-rules.md). |

None of these domains is required for the campaign-tracking vocabulary alone, which depends only on
the platform foundation.

Platform behaviour that this domain relies on but does not define is specified in
[../../runtime/scheduled-jobs.md](../../runtime/scheduled-jobs.md) (the job runner and its progress
reporting), [../../runtime/mail-gateway.md](../../runtime/mail-gateway.md) (the incoming gateway
that detects bounces and answers), [../../runtime/request-lifecycle.md](../../runtime/request-lifecycle.md)
(how a public endpoint is dispatched and how cookies are written),
[../../runtime/attachments-and-file-store.md](../../runtime/attachments-and-file-store.md) (where
inline images are stored), [../../runtime/report-rendering.md](../../runtime/report-rendering.md)
(how a marketing card is rasterised),
[../../runtime/configuration-parameters.md](../../runtime/configuration-parameters.md) (the system
parameter table), [../../overview/security-model.md](../../overview/security-model.md) (groups,
access rights and record rules) and
[../../overview/record-operations-and-query-notation.md](../../overview/record-operations-and-query-notation.md)
(how a stored recipient condition is written and evaluated).

---

## 7. Reading order

1. [README.md](README.md) — this file: scope, entities, dependencies.
2. [glossary.md](glossary.md) — the vocabulary. Read it before anything else if the words *trace*,
   *medium*, *source*, *seen list* or *comparison test* are new.
3. [entities.md](entities.md) — every field of every entity, its derivation, its constraints and its
   life cycle.
4. [state-machines.md](state-machines.md) — the six state machines, their guards and their refusal
   messages.
5. [workflows.md](workflows.md) — the twenty-seven end-to-end procedures, including the sending
   algorithm, which is the core of the domain.
6. [business-rules.md](business-rules.md) — the numbered rule catalogue that the other documents
   cite.
7. [calculations.md](calculations.md) — every formula with its rounding and a worked example.
8. [configuration.md](configuration.md) — settings, parameters, jobs, shipped records, groups,
   access rights and record rules.
9. [interfaces.md](interfaces.md) — menus, screens, operations, endpoints, produced documents and
   external integrations.
10. [accounting-effects.md](accounting-effects.md) — the ledger boundary.
11. [acceptance-criteria.md](acceptance-criteria.md) — the numbered scenarios a rebuild must pass.

---

## 8. Every file in this folder

| File | Contents |
|---|---|
| [README.md](README.md) | Scope, questions answered, capabilities, actors, owned entities with their transport names, borrowed entities, capability packages, dependencies, reading order and this file list. |
| [entities.md](entities.md) | Every field of every owned entity and of every field added to entities owned elsewhere, with identifier, full name, type, requirement, default, storage, derivation, copy behaviour, tracking, constraints, validations, on-change behaviour and life cycle. |
| [state-machines.md](state-machines.md) | The six state machines: states with stored value, label and meaning; transition tables with triggers, guards and side effects; the exact refusal message of every guard; a diagram per machine. |
| [workflows.md](workflows.md) | End-to-end procedures: designing, testing, scheduling, sending, retrying, comparison testing, importing and merging audiences, unsubscribing, click and open tracking, marketing card production and sharing, bounce and answer handling. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with guards, permissions, uniqueness, rounding, dates and the exact user-facing messages. |
| [calculations.md](calculations.md) | Every formula and algorithm with worked examples: the five percentage indicators, the click ratio, the audience-size and sample-size computations, the batching algorithm, the winner selection, the list quality percentages, the short-code generation, the token computations and the text-message length estimate. |
| [accounting-effects.md](accounting-effects.md) | The reasoned statement that this domain posts no journal entry, the three monetary figures it reads, and the path by which the cost of sending reaches the ledger. |
| [configuration.md](configuration.md) | Every setting, system parameter, scheduled job, shipped record, sequence, group, access right, record rule and message template. |
| [interfaces.md](interfaces.md) | Menus, views, named operations, public and internal endpoints, produced documents, notifications, import and export, and external integrations. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete records, inputs, amounts and states. |
| [glossary.md](glossary.md) | Every term of the domain, defined. |

Machine-readable catalogues for the entities of this domain are under
[`../../../schemas/data/entities/`](../../../schemas/data/entities/); the generated reference page of
each entity is under [`../../references/entities/`](../../references/entities/).
