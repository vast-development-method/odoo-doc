# Customer Relationship Management

This domain specifies the complete pre-sale capability of the system: the capture of a commercial
interest as a Lead, its qualification into an Opportunity, the pipeline of Stages it travels
through, the Sales Team and Sales Team Member structure that owns it, the automatic allocation and
assignment of leads to teams and to salespeople according to capacity, the predictive probability
that a deal will be won, the revenue expectations attached to a deal (one-off, recurring and
prorated), the detection and merging of duplicates, the conversion into a customer record, the won
and lost outcomes with their reasons, and every channel through which a lead can enter the system
(electronic mail, a public web form, a live chat conversation, an electronic mail client plug-in, a
text message, a lead generation service, an enrichment service, and a partner network).

The domain is the head of the order-to-cash chain. It does not itself price anything, ship anything
or post anything to the ledger. It produces two kinds of outcome: a qualified Opportunity that the
Sales domain turns into a quotation, and a Contact record that the Contacts domain owns from then
on. Every one of those hand-offs is specified here field by field so that a re-implementation
produces the same downstream records.

---

## 1. Capabilities covered

| Capability | Summary |
|---|---|
| Lead capture | Creation of a Lead from a form, from an incoming electronic mail addressed to a team alias, from a public website form, from a live chat conversation, from an electronic mail client plug-in, from a text message reply, from a lead generation service request, and from a partner network portal. |
| Lead qualification | Contact and company name, job position, electronic mail address, telephone number, postal address, language, website, tags, priority, referral source, free notes, and per-team user-defined properties. |
| Quality signals | Automatic classification of the electronic mail address and of the telephone number as correct or incorrect, derived from format validation and from the country. |
| Pipeline | An ordered set of Stages, optionally restricted to given teams, with a folding flag, a won flag, a rotting threshold and internal requirements text. |
| Priorities | A four-level priority scale used as the primary sort key of the pipeline. |
| Sales organisation | Sales Teams with a leader, members, an electronic mail alias, an assignment domain, a capacity derived from members, and per-team lead property definitions; Sales Team Members with individual capacity, an assignment domain, a preferred assignment domain and a pause switch. |
| Rule-based allocation | A weighted random allocation of unassigned leads to teams proportional to team capacity, with duplicate merging performed during allocation. |
| Rule-based assignment | A round-robin assignment of team leads to members respecting each member's daily quota, honouring preferred domains first, and converting each assigned lead into an Opportunity. |
| Predictive probability | A naive Bayesian classifier over a configurable set of lead variables, fed by a per-team frequency table of won and lost counts, with live increments on every won and lost transition and a full rebuild by a scheduled job. |
| Manual probability | A user-entered probability that detaches the record from the automatic computation until it is realigned. |
| Revenue expectation | Expected revenue, recurring revenue with a plan expressed in months, the monthly equivalent of the recurring revenue, and the prorated variants of all three weighted by the probability. |
| Date metrics | Assignment date, days to assign, closed date, days to close, last stage update, conversion date, expected closing date, rotting days. |
| Duplicate detection | Two distinct detectors: a display-oriented one based on electronic mail domain, sanitized telephone number and commercial entity; and a merge-oriented one based on normalized electronic mail address and linked contact. |
| Merge | A deterministic merge of two or more leads or opportunities into the most trustworthy one, with field-by-field precedence, address block precedence, history, attachment, meeting and follower transfer, and a summary note. |
| Conversion | Single-record and mass conversion wizards that create or link a Contact, set the type to Opportunity, stamp the conversion date, and allocate salespeople round-robin. |
| Won and lost | A won flow that moves the record to the appropriate won stage and forces the probability to one hundred; a lost flow that archives the record, forces the probability to zero and records a lost reason with an optional closing note. |
| Celebration message | A ranked set of congratulation messages chosen from historical statistics when a deal is won from the form view. |
| Enrichment | Automatic or on-demand completion of a lead from a company data service keyed on the electronic mail domain. |
| Lead generation | A request-based service that creates leads from company or contact criteria (country, size, industry, role, seniority). |
| Partner network | Grades, activations, geographic assignment of an opportunity to the nearest suitable partner, a forwarding message, and a partner-facing portal with interest and desinterest actions. |
| Campaign tracking | Campaign, source and medium parameters carried from the web visit through to the lead and onward to the quotation. |
| Recognition | Sales-related badge goals fed from won opportunity counts and amounts. |
| Analysis | A pipeline analysis built directly on the lead table and an activity analysis built on the message table. |

---

## 2. Entities of the domain

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
| Pipeline Analysis | `crm.lead` (list, pivot and graph presentations) | `crm_lead` | Aggregation of leads and opportunities by stage, team, salesperson, country, campaign and date. |
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
| Partner Grade | `res.partner.grade` | `res_partner_grade` | A ranking level of a reselling partner, with a weight used in geographic assignment. |
| Partner Activation | `res.partner.activation` | `res_partner_activation` | The activity level of a reselling partner. |
| Partner Assignment Analysis | `crm.partner.report.assign` | database view `crm_partner_report_assign` | Read-only aggregation of partners with their assigned opportunity counts. |
| Forward to Partner Wizard | `crm.lead.forward.to.partner` | transient | Sends an opportunity to one or more reselling partners by electronic mail. |
| Lead Assignation Line | `crm.lead.assignation` | transient | One proposed lead-to-partner pairing inside the forwarding wizard. |
| Campaign | `utm.campaign` | `utm_campaign` | A named marketing effort that leads and later documents are attributed to. |
| Source | `utm.source` | `utm_source` | The origin of the link that produced the visit and then the lead. |
| Medium | `utm.medium` | `utm_medium` | The delivery method of the link that produced the visit and then the lead. |
| Campaign Stage | `utm.stage` | `utm_stage` | One column of the campaign pipeline. |
| Campaign Tag | `utm.tag` | `utm_tag` | A free classification label attachable to campaigns. |

Entities that this domain extends rather than owns — and whose extensions are specified here — are
the Contact (`res.partner`), the User (`res.users`), the Company (`res.company`), the Meeting
(`calendar.event`), the Sales Order (`sale.order`), the Discussion Channel (`discuss.channel`), the
Website Visitor (`website.visitor`), the Periodic Digest (`digest.digest`) and the Configuration
Settings (`res.config.settings`).

---

## 3. Reading order

1. **`entities.md`** — every entity with its complete field table, defaults, computed rules,
   relations, uniqueness rules, ordering, display rule, archival behaviour and company scoping.
   Read this first; every other file refers to field storage names defined here.
2. **`state-machines.md`** — the lead type machine, the pipeline stage machine, the won and lost
   outcome machine, the activity and archival flags, the lead generation request machine and the
   partner assignment machine, each with a state table, a transition table and a diagram.
3. **`calculations.md`** — every formula: the predictive probability with exact arithmetic over the
   frequency tables, the revenue formulas, the date metrics, the capacity and quota arithmetic, the
   allocation weighting, the confidence ordering, the geographic distance selection.
4. **`workflows.md`** — the end-to-end operational sequences: capture, qualification, conversion,
   merge, assignment, win, loss, restore, enrichment, partner forwarding, with the actor performing
   each step and the records created or updated.
5. **`business-rules.md`** — every validation, constraint, invariant, exact error message,
   permission check and edge case.
6. **`configuration.md`** — settings, system parameters, default records, security groups, the
   access rights matrix, the record rules and the scheduled jobs.
7. **`interfaces.md`** — menus, window actions, views, named remote operations, routes, reports,
   electronic mail templates and external service contracts.
8. **`accounting-effects.md`** — the statement that this domain posts no journal entries, and the
   precise description of how it nonetheless reaches the ledger through other domains.
9. **`acceptance-criteria.md`** — numbered Given / When / Then scenarios with concrete numbers.
10. **`glossary.md`** — every term used in the domain, defined in full.

---

## 4. Dependencies on other domains

| Domain | What this domain needs from it | What it gives back |
|---|---|---|
| [Contacts and organisations](../contacts-and-organizations/README.md) | The Contact record, its company and individual distinction, its parent hierarchy and commercial entity, its address fields, its language and its electronic mail and telephone normalisation. | Newly created Contacts produced by conversion; updates of the contact's electronic mail address and telephone number propagated from the lead. |
| [Identity and access](../identity-and-access/README.md) | Users, groups, record rules, multi-company scoping. | Team membership drives the record rules that restrict lead visibility. |
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread on a record, followers, activities, activity plans, electronic mail aliases, the incoming electronic mail gateway, tracking of field changes, the blacklist and the telephone mixins, stage duration tracking, rotting. | Message subtypes specific to lead creation, stage change, win, loss and restoration; an alias per team. |
| [Sales](../sales/README.md) | The Sales Order entity, to count quotations and confirmed orders on an opportunity and to feed the expected revenue back from a confirmed order. | The Opportunity reference stamped on every quotation created from it, and the campaign, source, medium, tags, salesperson and team defaults carried over. |
| [Products and catalogue](../products-and-catalog/README.md) | Nothing directly. | Nothing. |
| [Units of measure and packaging](../units-of-measure-and-packaging/README.md) | Nothing. | Nothing. |
| [General ledger](../general-ledger/README.md) | Nothing directly; the domain posts no journal entries. | Indirectly, through the Sales domain, the confirmed order that a won opportunity produced. |

---

## 5. Conventions used in these documents

- Entity names are written in full words and in title case: Lead, Opportunity, Sales Team, Sales
  Team Member, Stage, Lost Reason, Recurring Plan, Scoring Frequency.
- On first mention in each file an entity is followed by its transport name and storage name in
  code font, for example Lead (`crm.lead`, table `crm_lead`).
- Field storage names are reproduced exactly in code font because external contracts depend on
  them, and each is accompanied on first use by its full name in words, for example
  `expected_revenue` (expected revenue).
- Formulas are written as plain mathematics in fenced blocks labelled `formula`, with named
  quantities in words and explicit rounding.
- Algorithms are written as numbered steps with preconditions, postconditions and failure
  conditions.
- All amounts in worked examples are expressed in a single currency unless the example is
  explicitly about currency conversion.

---

## 6. What this domain deliberately does not cover

- The pricing of a deal. An Opportunity carries only an *expectation* of revenue typed by a
  salesperson or fed back from a confirmed order; it has no lines, no products, no taxes and no
  price list. Pricing belongs to the Sales domain.
- The customer record itself. The domain creates Contacts and synchronises a small set of fields
  with them, but the Contact entity, its addresses, its bank details and its accounting settings
  belong to the Contacts domain.
- The marketing automation that generates visits. The domain stores the campaign, source and medium
  attribution of a lead but does not run campaigns.
- Any ledger movement. See `accounting-effects.md`.
