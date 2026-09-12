# Customer Relationship Management

This domain specifies the complete pre-sale capability of the system: the capture of a commercial
interest as a Lead, its qualification into an Opportunity, the pipeline of Stages it travels
through, the Sales Team and Sales Team Member structure that owns it, the automatic allocation and
assignment of leads to teams and to salespeople according to capacity, the predictive probability
that a deal will be won, the revenue expectations attached to a deal (one-off, recurring and
prorated), the detection and merging of duplicates, the conversion into a customer record, the won
and lost outcomes with their reasons, and every channel through which a lead can enter the system
(electronic mail, a public web form, a live chat conversation, an electronic mail client plug-in, a
text message, an event registration, a survey participation, a lead generation service, an
enrichment service, identified website traffic and a partner network).

The domain is the head of the order-to-cash chain. It does not itself price anything, ship anything
or post anything to the ledger. It produces two kinds of outcome: a qualified Opportunity that the
Sales domain turns into a quotation, and a Contact record that the Contacts domain owns from then
on. Every one of those hand-offs is specified here field by field so that a re-implementation
produces the same downstream records.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Lead capture | Creation of a Lead from a form, from an incoming electronic mail addressed to a team alias, from a public website form, from a live chat conversation, from an electronic mail client plug-in, from a text message reply, from an event registration, from a survey participation, from a lead generation service request, and from identified website traffic. |
| Lead qualification | Contact and company name, job position, electronic mail address, telephone number, postal address, language, website, tags, priority, referral source, free notes, and per-team user-defined properties. |
| Quality signals | Automatic classification of the electronic mail address and of the telephone number as correct or incorrect, derived from format validation and from the country, each specified as a state machine. |
| Pipeline | An ordered set of Stages, optionally restricted to given teams, with a folding flag, a won flag, a rotting threshold and internal requirements text. |
| Priorities | A four-level priority scale used as the primary sort key of the pipeline. |
| Sales organisation | Sales Teams with a leader, members, an electronic mail alias, an assignment domain, a capacity derived from members, and per-team lead property definitions; Sales Team Members with individual capacity, an assignment domain, a preferred assignment domain and a pause switch. |
| Rule-based allocation | A weighted random allocation of unassigned leads to teams proportional to team capacity, with duplicate merging performed during allocation. |
| Rule-based assignment | A round-robin assignment of team leads to members respecting each member's daily quota, honouring preferred domains first, and converting each assigned lead into an Opportunity. |
| Predictive probability | A naive Bayesian classifier over a configurable set of lead variables, fed by a per-team frequency table of won and lost counts, with live increments on every won and lost transition and a full rebuild by a scheduled job. |
| Manual probability | A user-entered probability that detaches the record from the automatic computation until it is realigned. |
| Revenue expectation | Expected revenue, recurring revenue with a plan expressed in months, the monthly equivalent of the recurring revenue, and the prorated variants of all three weighted by the probability. |
| Date metrics | Assignment date, days to assign, closed date, days to close, last stage update, conversion date, expected closing date, rotting days, time spent in each stage. |
| Duplicate detection | Two distinct detectors: a display-oriented one based on the electronic mail domain criterion, the sanitized telephone number and the commercial entity; and a merge-oriented one based on the normalized electronic mail address and the linked contact. |
| Merge | A deterministic merge of two or more leads or opportunities into the most trustworthy one, with field-by-field precedence, address block precedence, history, attachment, meeting and follower transfer, and a summary note. |
| Conversion | Single-record and mass conversion wizards that create or link a Contact, set the type to Opportunity, stamp the conversion date, and allocate salespeople round-robin. |
| Won and lost | A won flow that moves the record to the appropriate won stage and forces the probability to one hundred; a lost flow that archives the record, forces the probability to zero and records a lost reason with an optional closing note. |
| Celebration message | A ranked set of congratulation messages chosen from historical statistics when a deal is won from the form view. |
| Enrichment | Automatic or on-demand completion of a lead from a company data service keyed on the electronic mail domain. |
| Lead generation | A request-based service that creates leads from company or contact criteria (country, state, size, industry, role, seniority). |
| Website identification | Rules that turn identified website traffic into leads, with one pending view per network address and per rule. |
| Partner network | Grades, activations, geographic assignment of an opportunity to the nearest suitable partner, a forwarding message, a public partner directory, and a partner-facing portal with interest and desinterest actions. |
| Membership programme | A product whose sale grants a partner level to the customer's commercial entity. |
| Campaign tracking | Campaign, source and medium parameters carried from the web visit through to the lead and onward to the quotation, with roll-up counters on the campaign and on the mass mailing. |
| Communication | Message composition with suggested recipients, batch messages, single and batch text messages, mass mailing on leads, and the electronic mail and telephone blacklists. |
| Recognition | Sales-related badge goals and challenges fed from won opportunity counts and amounts. |
| Analysis | A pipeline analysis built directly on the lead table, a leads analysis, a forecast, an activity analysis built on the message table, and a partnership analysis. |

---

## 2. Entities of the domain

### 2.1 Entities this folder owns

| Entity | Transport name | Storage | One-line purpose |
|---|---|---|---|
| Lead | `crm.lead` | `crm_lead` | A commercial interest, before (type Lead) or after (type Opportunity) qualification; the central record of the domain. |
| Stage | `crm.stage` | `crm_stage` | One column of the pipeline, with an order, a won flag and an optional team restriction. |
| Tag | `crm.tag` | `crm_tag` | A free classification label attachable to leads, opportunities and sales documents. |
| Lost Reason | `crm.lost.reason` | `crm_lost_reason` | A reusable explanation recorded when a deal is lost. |
| Recurring Plan | `crm.recurring.plan` | `crm_recurring_plan` | A named duration in months used to convert a recurring revenue into a monthly figure. |
| Scoring Frequency | `crm.lead.scoring.frequency` | `crm_lead_scoring_frequency` | One cell of the frequency table: for one team, one variable and one value, the won count and the lost count. |
| Scoring Frequency Field | `crm.lead.scoring.frequency.field` | `crm_lead_scoring_frequency_field` | The catalogue of lead fields that may be selected as scoring variables. |
| Sales Team | `crm.team` | `crm_team` | A group of salespeople sharing a pipeline, an alias, an allocation domain and a capacity. |
| Sales Team Member | `crm.team.member` | `crm_team_member` | The membership of one user in one team, carrying the individual assignment capacity and domains. |
| Activity Analysis | `crm.activity.report` | database view `crm_activity_report` | Read-only aggregation of completed activities logged on leads. |
| Convert to Opportunity Wizard | `crm.lead2opportunity.partner` | transient | Converts one lead, creating or linking a Contact, or merges it with detected duplicates. |
| Mass Convert Wizard | `crm.lead2opportunity.partner.mass` | transient | Converts many leads at once, with optional deduplication and round-robin salesperson allocation. |
| Merge Wizard | `crm.merge.opportunity` | transient | Merges the selected leads and opportunities into one. |
| Lost Reason Wizard | `crm.lead.lost` | transient | Applies a lost reason and an optional closing note to one or many records. |
| Probability Rebuild Wizard | `crm.lead.pls.update` | transient | Changes the scoring start date and the scoring variables and rebuilds the frequency table. |
| Quotation Contact Wizard | `crm.quotation.partner` | transient | Asks for or creates the Contact required before a quotation can be started from an opportunity. |
| Lead Generation Request | `crm.iap.lead.mining.request` | `crm_iap_lead_mining_request` | A parameterised request to a company data service that produces leads. |
| Industry | `crm.iap.lead.industry` | `crm_iap_lead_industry` | A classification value offered by the lead generation service. |
| Role | `crm.iap.lead.role` | `crm_iap_lead_role` | A contact role value offered by the lead generation service. |
| Seniority | `crm.iap.lead.seniority` | `crm_iap_lead_seniority` | A contact seniority value offered by the lead generation service. |
| Lead Enrichment Helpers | `crm.iap.lead.helpers` | `crm_iap_lead_helpers` | A stateless service entity shared by the lead generation and website identification capabilities. |
| Partner Assignment Analysis | `crm.partner.report.assign` | database view `crm_partner_report_assign` | Read-only aggregation of partners with their assigned opportunity counts and invoiced turnover. |
| Forward to Partner Wizard | `crm.lead.forward.to.partner` | transient | Sends an opportunity to one or more reselling partners by electronic mail. |
| Lead Assignation Line | `crm.lead.assignation` | transient | One proposed lead-to-partner pairing inside the forwarding wizard. |
| Event Lead Rule | `event.lead.rule` | `event_lead_rule` | A rule that creates leads from event registrations, per attendee or per order, at creation, confirmation or attendance. |
| Event Lead Request | `event.lead.request` | `event_lead_request` | The resumable bookkeeping record of a batched lead generation run for one event. |
| Lead Generation Rule | `crm.reveal.rule` | `crm_reveal_rule` | A rule that turns identified website traffic into leads, filtered by country, state, page pattern, industry and size. |
| Reveal View | `crm.reveal.view` | `crm_reveal_view` | One visit of one network address to a page matched by a Lead Generation Rule, waiting to be resolved into a company. |
| Campaign | `utm.campaign` | `utm_campaign` | A named marketing effort that leads and later documents are attributed to. |
| Source | `utm.source` | `utm_source` | The origin of the link that produced the visit and then the lead. |
| Medium | `utm.medium` | `utm_medium` | The delivery method of the link that produced the visit and then the lead. |
| Campaign Stage | `utm.stage` | `utm_stage` | One column of the campaign pipeline. |
| Campaign Tag | `utm.tag` | `utm_tag` | A free classification label attachable to campaigns. |
| Attribution mixin | `utm.mixin` | abstract | The three attribution links contributed to every record that can be attributed. |
| Source mixin | `utm.source.mixin` | abstract | The required source link contributed to records that are themselves a traffic source. |

### 2.2 Entities of the scope that another folder owns

Three entities of this domain's package scope are shared with, or belong to, another folder. They
are specified here only for the part this domain contributes, and the owning folder is named.

| Entity | Transport name | Owning folder | What this folder specifies |
|---|---|---|---|
| Partner Grade | `res.partner.grade` | [Contacts and organisations](../contacts-and-organizations/) | The level weight used by the geographic assignment, the default price list it imposes, and its public directory page. |
| Partner Activation | `res.partner.activation` | [Contacts and organisations](../contacts-and-organizations/) | The activity level of a reselling partnership, shipped by this domain's partner network capability. |
| Telephone Blacklist and its removal wizard | `phone.blacklist`, `phone.blacklist.remove` | [Messaging and activities](../messaging-and-activities/) | The uniqueness, activation and access rules a Lead depends on when a number is blacklisted. |
| Telephone blacklist mixin | `mail.thread.phone` | [Messaging and activities](../messaging-and-activities/) | The four fields it contributes to the Lead and the searches it enables. |

### 2.3 Entities owned elsewhere that this domain extends

| Entity | Owning domain | What this domain adds |
|---|---|---|
| Contact (`res.partner`) | [Contacts and organisations](../contacts-and-organizations/) | The opportunity list and the hierarchical opportunity count; the reseller fields — level, level weight, level sequence, activation, partnership date, latest review, next review, implemented by, implementation references and their count. |
| Company (`res.company`) | [Contacts and organisations](../contacts-and-organizations/) | The label used to name affiliates. |
| Price List (`product.pricelist`) | [Pricing and price lists](../pricing-and-pricelists/) | The count of partners using the price list and the affiliate label. |
| Product Template (`product.template`) | [Products and catalogue](../products-and-catalog/) | A membership value of the service tracking selection and the level the product grants. |
| User (`res.users`) | [Identity and access](../identity-and-access/) | The sales team list, the membership list and the main sales team; archiving a user archives the memberships. |
| Meeting (`calendar.event`) | [Calendar and scheduling](../calendar-and-scheduling/) | The opportunity link, the defaults taken from the opportunity, the highlighting of its meetings and the note logged at creation. |
| Discussion Channel, Chatbot Script and Chatbot Script Step | [Messaging and activities](../messaging-and-activities/) | The lead list on a channel, the stored flag that grants salespeople read access, the conversation command that creates a lead, and two script step types that create and optionally forward a lead. |
| Live chat channel report | [Messaging and activities](../messaging-and-activities/) | The count of leads created from the conversations of the channel. |
| Sales Order (`sale.order`) | [Sales](../sales/) | The opportunity link, the expected-revenue feedback on confirmation, and the level granted by a membership product. |
| Event, Event Registration and Event Question Answer | [Events](../events/) | The lead list and count, the execution of the lead rules, and the control that creates a rule from an answer option. |
| Survey, Survey Question, Survey Answer Option and Survey Participation | [Learning, surveys and gamification](../learning-surveys-and-gamification/) | The lead-generating flags, the team to assign, the lead list and count, and lead creation when a participation is completed. |
| Campaign and Mass Mailing | [Marketing and mass mailing](../marketing-and-mass-mailing/) | The lead usage flag, the lead counts, the navigation controls, the split-test criterion that counts leads, and the lead figure in the statistics message. |
| Website and Website Visitor | [Website and storefront](../website-and-storefront/) | The default team and salesperson of the contact form; the lead list and count on a visitor, the fallback address and telephone, the protection from cleanup and the merge behaviour. |
| Periodic Digest | [Human resources core](../human-resources-core/) | Two indicators: new leads and opportunities won. |
| Configuration Settings and System Parameter | [Platform foundation](../platform-foundation/) | Every setting of [configuration.md](configuration.md); writing the scoring variable list reloads the Lead entity. |

Generic platform entities whose transport name begins with a reserved prefix — the model, field,
attachment, translation, action, view, scheduled job and property definition entities — belong to
the platform foundation and to the overview documents, not to this folder; this folder only states
what it stores in them.

---

## 3. Reading order

1. **[entities.md](entities.md)** — every entity with its complete field table, defaults, computed
   rules, relations, uniqueness rules, ordering, display rule, archival behaviour and company
   scoping. Read this first; every other file refers to field storage names defined here.
2. **[state-machines.md](state-machines.md)** — the lead type machine, the pipeline stage machine,
   the won and lost outcome machine, the activity and archival flags, the two quality machines, the
   lead generation request machine, the reveal view machine and the partner assignment machine,
   each with a state table, a transition table and a diagram.
3. **[calculations.md](calculations.md)** — every formula: the revenue formulas, the date metrics,
   the predictive probability with exact arithmetic over the frequency tables, the capacity and
   quota arithmetic, the allocation weighting, the confidence ordering, the merge precedence, the
   geographic distance selection, the credit estimate and the celebration selection.
4. **[predictive-lead-scoring.md](predictive-lead-scoring.md)** — the frequency table, the scoring
   variables, the naive Bayesian formula with its smoothing and its clamping, the full rebuild and
   the live increment, the explanation panel, the configuration dialogue and four fully worked
   examples.
5. **[lead-assignment.md](lead-assignment.md)** — the configuration and the schedule, the
   allocation to teams by weighted draw, the distribution to members, the quota arithmetic, the
   preference pass, the deduplication during allocation, the result messages and four worked
   examples.
6. **[workflows.md](workflows.md)** — the end-to-end operational sequences: capture through every
   channel, qualification, conversion, merge, assignment, win, loss, restore, enrichment, partner
   forwarding, communication and campaign attribution, with the actor performing each step and the
   records created or updated.
7. **[business-rules.md](business-rules.md)** — every validation, constraint, invariant, exact
   error message, permission check and edge case, numbered from `LEAD-001` to `LEAD-189`, with the
   mapping of the former identifiers.
8. **[configuration.md](configuration.md)** — settings, system parameters, numbering series,
   default records, security groups, the access rights matrix, the record rules, the scheduled jobs
   and the menus.
9. **[interfaces.md](interfaces.md)** — named operations, request endpoints, screens, reports,
   notifications, message templates and external service contracts.
10. **[accounting-effects.md](accounting-effects.md)** — the reasoned statement that this domain
    posts no journal entry, and the precise description of how it nonetheless reaches the ledger
    through other domains.
11. **[acceptance-criteria.md](acceptance-criteria.md)** — numbered Given / When / Then scenarios
    with concrete numbers.
12. **[glossary.md](glossary.md)** — every term used in the domain, defined in full.

---

## 4. Every file in this folder

| File | Content |
|---|---|
| [README.md](README.md) | This file: scope, capabilities, entities, reading order, dependencies and conventions. |
| [entities.md](entities.md) | Every entity in full: purpose, lifecycle, complete field table, relations, uniqueness, ordering, display rule, archival, company behaviour and the extensions other packages contribute. |
| [state-machines.md](state-machines.md) | Every state-bearing field with its states, transitions, guards, refusal messages and a diagram. |
| [workflows.md](workflows.md) | End-to-end operational procedures with the records each step creates or changes. |
| [business-rules.md](business-rules.md) | The numbered rule catalogue with the exact user-facing messages, the access matrix and the identifier mapping. |
| [calculations.md](calculations.md) | Every formula and algorithm with rounding, precision, currency handling and worked numeric examples. |
| [predictive-lead-scoring.md](predictive-lead-scoring.md) | The probability model in full, as a topic of its own. |
| [lead-assignment.md](lead-assignment.md) | The two-phase assignment algorithm in full, as a topic of its own. |
| [accounting-effects.md](accounting-effects.md) | Why this domain posts nothing, and where the boundary lies. |
| [configuration.md](configuration.md) | Settings, parameters, sequences, delivered records, groups, record rules, scheduled jobs, message templates and menus. |
| [interfaces.md](interfaces.md) | Operations, endpoints, screens, reports, notifications and external integrations. |
| [acceptance-criteria.md](acceptance-criteria.md) | Numbered Given, When and Then scenarios. |
| [glossary.md](glossary.md) | Domain vocabulary. |

---

## 5. Actors

| Actor | Group | What they may do in this domain |
|---|---|---|
| Salesperson (own documents only) | *User: Own Documents Only* | Create, read and modify the leads that are theirs or unassigned; create Contacts; run the conversion, merge and loss wizards; **may not delete** a lead. |
| Salesperson (all documents) | *User: All Documents* | Everything above, on every lead of every team. |
| Sales administrator | *Administrator* | Everything above, plus delete leads, configure stages, tags, lost reasons, recurring plans, teams, memberships, assignment rules, activity plans and text message templates, and trigger the assignment run. |
| Platform administrator | *Settings* | Everything, plus the probability rebuild wizard and the scoring configuration. |
| Portal partner (reseller) | portal, with a partner level | Read and act on the opportunities assigned to their commercial family through the partner portal. |
| Website visitor | public | Submit the public contact form, converse with the live chat script, answer a survey, register for an event. |
| Scheduled job runner | the system identity | The allocation job, the probability rebuild, the enrichment job, the event lead generation job and the website identification job. |

---

## 6. Dependencies on other domains

| Domain | What this domain needs from it | What it gives back |
|---|---|---|
| [Platform foundation](../platform-foundation/) | Persistence, access groups, record rules, scheduled jobs, system parameters, numbering series, actions, views and property definitions. | The settings of this domain and the rule that writing the scoring variable list reloads the Lead entity. |
| [Identity and access](../identity-and-access/) | Users, the internal and shared distinction, the companies allowed for a user, portal users, groups and record rules. | Team membership drives the record rules that restrict lead visibility; the main sales team of a user. |
| [Contacts and organisations](../contacts-and-organizations/) | The Contact record, its company and individual distinction, its parent hierarchy and commercial entity, its address fields, its language, its electronic mail and telephone normalisation, countries, states, languages, currencies and geolocation. | Newly created Contacts produced by conversion; updates of the contact's electronic mail address and telephone number propagated from the lead; the reseller fields and the partner activation catalogue. |
| [Messaging and activities](../messaging-and-activities/) | The discussion thread, followers, activities and activity plans, electronic mail aliases and the incoming gateway, tracking of field changes and stage durations, the electronic mail and telephone blacklists, telephone formatting, text messages, live chat and chatbots. | Five message subtypes on the Lead and five parent subtypes on the Sales Team; an alias per team; the lead list on a discussion channel. |
| [Calendar and scheduling](../calendar-and-scheduling/) | Meetings linked to opportunities. | The opportunity link on a meeting and the defaults taken from the opportunity. |
| [Marketing and mass mailing](../marketing-and-mass-mailing/) | Campaigns, sources, mediums and the attribution mixin; mass mailings. | Lead counters on a campaign and on a mailing, and the split-test criterion that counts leads. |
| [Sales](../sales/) | The Sales Order entity, to count quotations and confirmed orders on an opportunity and to feed the expected revenue back from a confirmed order. | The opportunity reference stamped on every quotation created from it, and the campaign, source, medium, tags, salesperson and team defaults carried over. |
| [Products and catalogue](../products-and-catalog/) and [Pricing and price lists](../pricing-and-pricelists/) | Membership products and the price list attached to a level. | The level a confirmed order grants and the partner count on a price list. |
| [Events](../events/) and [Learning, surveys and gamification](../learning-surveys-and-gamification/) | Registrations and participations that feed the lead rules; the recognition goals and challenges. | The lead rules, the lead lists and the goal definitions fed from lead counts. |
| [Website and storefront](../website-and-storefront/) | The public contact form, visitors, page tracking and the partner directory pages. | The default team and salesperson of the form and the lead list on a visitor. |
| [Human resources core](../human-resources-core/) | The periodic digest. | Two digest indicators. |
| [Accounts receivable](../accounts-receivable/) | The customer invoice analysis rows read by the partnership analysis view. | Nothing. |
| [General ledger](../general-ledger/) | The currency conversion rule used when the confirmed orders of an opportunity are summed. | Nothing; the domain posts no journal entry. See [accounting-effects.md](accounting-effects.md). |
| [Analytic accounting](../analytic-accounting/) | Nothing. | Nothing; a Lead carries no analytic distribution. |
| [Units of measure and packaging](../units-of-measure-and-packaging/) | Nothing. | Nothing. |

---

## 7. Conventions used in these documents

- Entity names are written in full words and in title case: Lead, Opportunity, Sales Team, Sales
  Team Member, Stage, Lost Reason, Recurring Plan, Scoring Frequency.
- On first mention in each file an entity is followed by its transport name and storage name in
  code font, for example Lead (`crm.lead`, table `crm_lead`).
- Field storage names are reproduced exactly in code font because external contracts depend on
  them, and each is accompanied on first use by its full name in words, for example
  `expected_revenue` (expected revenue).
- Operation names written in code font in [interfaces.md](interfaces.md) are the stable names this
  specification assigns to each operation so that the documents can cite them; they are not strings
  taken from an external contract, and a rebuild may choose its own.
- Formulas are written as plain arithmetic in fenced blocks labelled `formula`, with named
  quantities in words and explicit rounding.
- Algorithms are written as numbered steps with preconditions, postconditions and failure
  conditions.
- Rule identifiers are stable within [business-rules.md](business-rules.md) and are cited from the
  other files of this folder.
- All amounts in worked examples are expressed in a single currency unless the example is
  explicitly about currency conversion.
- British spelling is used in this file, in [entities.md](entities.md),
  [state-machines.md](state-machines.md), [workflows.md](workflows.md) and
  [calculations.md](calculations.md); American spelling is used in the remaining files. Each file is
  internally consistent.

---

## 8. What this domain deliberately does not cover

- The pricing of a deal. An Opportunity carries only an *expectation* of revenue typed by a
  salesperson or fed back from a confirmed order; it has no lines, no products, no taxes and no
  price list. Pricing belongs to the Sales domain.
- The customer record itself. The domain creates Contacts and synchronises a small set of fields
  with them, but the Contact entity, its addresses, its bank details and its accounting settings
  belong to the Contacts domain.
- The marketing automation that generates visits. The domain stores the campaign, source and medium
  attribution of a lead but does not run campaigns.
- Any ledger movement. See [accounting-effects.md](accounting-effects.md).

---

## 9. Reconciliation notes

This folder was consolidated from two independently written descriptions of the same domain. Where
they agreed, the more precise wording was kept once. Where they differed, the source behaviour
decided, and the resolution is recorded at the end of the file that carries the subject:
[entities.md](entities.md) section 22, [state-machines.md](state-machines.md) section 13,
[workflows.md](workflows.md) section 20, [business-rules.md](business-rules.md) section 13,
[calculations.md](calculations.md) section 17, [configuration.md](configuration.md) section 13,
[interfaces.md](interfaces.md) section 10, [acceptance-criteria.md](acceptance-criteria.md) section
19 and [glossary.md](glossary.md) at the end.

The differences that affected this file are:

| Subject | The two statements | Resolution |
|---|---|---|
| Which entities the folder owns | One description listed the core pipeline entities; the other added the event, website identification, telephone blacklist and campaign mixin entities. | All of them are listed, split into the entities this folder owns, the entities of the package scope another folder owns, and the entities this folder only extends. |
| Naming of entity identifiers | One description used descriptive canonical names; the other reproduced the transport and storage names. | The transport and storage names are reproduced, because the generated reference pages and every integration key on them. |
| Sibling folder names | One description used the folder names of the working branch. | Every cross-reference uses the folder keys of this repository. |
