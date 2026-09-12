# Configuration

Every setting, system parameter, scheduled action, access group, record visibility rule, numbering series and shipped master-data record that the Customer Relationship Management domain needs, with data type, default value and effect. A replacement must ship the same master data with the same values, because workflows and messages reference them by name.

## 1. Application settings

These settings live on the configuration screen of the application. Settings whose effect is to install a capability package are marked as such; in a replacement they correspond to enabling the matching feature.

### 1.1 Pipeline

| Setting | Type | Default | Effect |
|---|---|---|---|
| Leads | boolean, backed by the access group **Show Lead Menu** | false | When true, incoming requests are qualified as Leads before being converted into opportunities: the Leads menu appears, the default `type` of a new record becomes `lead`, and the alias of every team that uses opportunities is switched to create Leads. When switched off, the same teams are switched back to creating opportunities. |
| Recurring Revenues | boolean, backed by the access group **Show Recurring Revenues Menu** | false | When true, the recurring revenue amount, the recurring plan and the four derived recurring figures are visible and the Recurring Plans menu appears. |
| Multi Teams | boolean, stored in the system parameter for multiple memberships | false | When true a user may belong to several Sales Teams; when false, creating or activating a membership archives the other active memberships of that user (`LEAD-RULE-105`). |
| Membership / Partnership | capability package | not installed | Adds partner grades, the membership product kind and the grade granted by a confirmed Sales Order. |

Switching the Leads setting has an immediate side effect on every team: every team whose `use_opportunities` is true receives `use_leads` equal to the new value, and the alias of every team is rewritten (`LEAD-RULE-113`).

### 1.2 Lead assignment

| Setting | Type | Default | Effect |
|---|---|---|---|
| Rule-Based Assignment | boolean, stored in a system parameter | false | Master switch of the assignment algorithm. See [lead-assignment.md](lead-assignment.md) section 1.1. |
| Auto Assignment Action | selection `manual` "Manually", `auto` "Repeatedly" | `manual` | `auto` activates the assignment scheduled action. |
| Auto Assignment Interval Unit | selection `minutes`, `hours`, `days`, `weeks` | `days` | Unit of the repetition interval. |
| Repeat every | integer | 1 | Number of units between two runs. Must be strictly positive and strictly below one hundred. |
| Auto Assignment Next Execution Date | datetime | empty | Instant of the next scheduled run. Recomputed as `now + number × unit` whenever the unit or the number changes; kept as typed when the user sets it directly. |

### 1.3 Lead generation and enrichment

| Setting | Type | Default | Effect |
|---|---|---|---|
| Generate new leads based on their country, industries and size | capability package | not installed | Adds the Lead Mining Request entity and its menus. The label shown on the settings screen is `Generate new leads based on their country, industries, size, etc.` |
| Enrich your leads automatically with company data based on their email address | capability package | not installed | Adds the enrichment operation, its scheduled action and the enrichment fields. |
| Create Leads/Opportunities from your website's traffic | capability package | not installed | Adds the Customer Relationship Management Lead Generation Rules and the Customer Relationship Management Reveal View entities. |
| Enrich lead automatically | selection `manual` "Enrich leads on demand only", `auto` "Enrich all leads automatically", stored in a system parameter | `auto` | With `auto`, creating any Lead triggers the enrichment job; with `manual`, only the button on the record triggers it. |
| Create a lead mining request directly from the opportunity pipeline | boolean, stored in a system parameter | false | Adds the mining action to the pipeline screen. |

### 1.4 Predictive lead scoring

| Setting | Type | Default | Effect |
|---|---|---|---|
| Lead Scoring Starting Date | date, derived from a text system parameter | the date eight days before installation | Records created before this date are ignored by the model. A parameter that cannot be parsed as a date is displayed as the date eight days before today and makes the model inactive. |
| Lead Scoring Frequency Fields | many_to_many to Lead Scoring Frequency Field, derived from a text system parameter | the seven shipped catalogue entries | The scored variables. |
| The variables in force | text, derived, read only | `Stage` plus the selected entries | Label listing the variables actually used, always beginning with `Stage`. |

The settings screen also offers the buttons **Manage Recurring Plans**, **Update Probabilities** (which opens the probability update dialogue) and the assignment button, which runs a manual assignment over every team that is not opted out.

## 2. System parameters

| Key meaning | Type of the stored value | Default | Read by |
|---|---|---|---|
| Scoring variable list | text: field names separated by commas | `phone_state,email_state,state_id,country_id,source_id,lang_id,tag_ids` | the scoring model; writing it reloads the Lead entity (`LEAD-RULE-135`) |
| Scoring start date | text: a date written as year, month and day separated by hyphens | the date eight days before installation | the scoring model and the settings screen |
| Rule-based assignment master switch | text read as a truth value | absent, which reads as false | the assignment algorithm and the team screen |
| Multiple memberships allowed | text read as a truth value | absent, which reads as false | the membership rules |
| Automatic enrichment mode | text: `manual` or `auto` | `auto` | the enrichment job |
| Lead mining from the pipeline | text read as a truth value | absent | the pipeline screen |
| Assignment delay, in hours | text read as a number | `0` | phase one of the assignment |
| Assignment commit batch size | text read as a whole number | `100` | both phases of the assignment |
| Enrichment notification throttle | text: an instant | set by the enrichment job | limits how often the "not enough credits" notification is repeated |
| Website identification retention, in months | text read as a whole number | `6` | the cleanup of the Customer Relationship Management Reveal View records that already produced a Lead |

## 3. Scheduled actions

| Name | Shipped state | Interval | Runs as | What it does |
|---|---|---|---|---|
| Predictive Lead Scoring: Recompute Automated Probabilities | inactive | every 1 day | the record owner of the action | Rebuilds the frequency table completely and recomputes the automated probability of every open record created on or after the scoring start date. |
| Customer Relationship Management: Lead Assignment | inactive | every 1 day | the system identity | Runs both phases of the assignment over every team that uses leads or opportunities and is not opted out, with the quota reduced by the last twenty-four hours and a creation window of seven days. |
| Customer Relationship Management: enrich leads | active once the enrichment package is installed | every 24 hours | the system identity | Enriches the eligible Leads created in the last twenty-four hours, in batches of fifty, each batch locked and committed separately. |
| Event Customer Relationship Management: Generate Leads based on Rules | active once the event bridge is installed | every 1 day | the record owner of the action | Processes the pending event lead generation requests in batches, resuming from the last processed registration. |
| Lead Generation: Leads/Opportunities Generation | active once the website identification package is installed | every 1 day | the system identity | Resolves the Customer Relationship Management Reveal View records into companies and creates the Leads. |

Every scheduled action of this domain is idempotent in the sense that a second run immediately after the first produces no additional record: the assignment finds no unclaimed Lead, the enrichment finds every record already marked, the event generation finds every registration already processed, and the scoring rebuild produces the same table.

## 4. Numbering series

| Series | Prefix | Padding | Scope | Used by |
|---|---|---|---|---|
| Lead Mining Request | `LMR` | 3 digits | all companies | The request number, assigned when the request is submitted. Until then the number is the literal text `New`. |

No other entity of this domain is numbered; a Lead is identified by its title and its surrogate identifier.

## 5. Access groups

| Group | Implies | Purpose |
|---|---|---|
| User: Own Documents Only | Internal user | A salesperson who sees only the Leads assigned to them or unassigned. |
| User: All Documents | User: Own Documents Only | A salesperson who sees every Lead of every team. |
| Administrator | User: All Documents, plus the canned response administration group of the messaging domain | Configures teams, stages, tags, lost reasons, recurring plans, assignment rules, activity plans and text message templates; may delete Leads; may run the assignment. |
| Show Lead Menu | none | Presentation group: makes the Leads menu visible and changes the default record type and the alias defaults. |
| Show Recurring Revenues Menu | none | Presentation group: makes the recurring revenue fields and the Recurring Plans menu visible. |

The three first groups belong to one privilege named **Sales**, of which a user holds at most one level. The two last groups are independent toggles.

The following identities of other domains also matter here: the platform administrator (who alone may confirm the probability update dialogue), the system identity (under which the scheduled actions run), the contact manager (who administers partner activation levels), the event manager (who administers event lead rules and may regenerate the leads of an event), and the portal user (an external reseller).

## 6. Record visibility rules

| Entity | Rule name | Applies to | Condition |
|---|---|---|---|
| Lead | Personal Leads | User: Own Documents Only | `user_id` is the reading user or is empty |
| Lead | All Leads | User: All Documents | everything |
| Lead | Multi-company | everyone | `company_id` is among the reader's allowed companies or is empty |
| Lead | Portal Graded Partner | Portal user | `partner_assigned_id` is the reader's commercial entity or one of its descendants, read only |
| Activity Analysis Report | Personal Activities | User: Own Documents Only | `user_id` is the reading user or is empty |
| Activity Analysis Report | All Activities | User: All Documents | everything |
| Activity Analysis Report | Multi-company | everyone | `company_id` is among the reader's allowed companies or is empty |
| Sales Team | All Salesteam | User: All Documents | everything |
| Sales Team | Multi-company | everyone | `company_id` is among the reader's allowed companies or is empty |
| Activity Plan | Manager can manage lead plans | Administrator | `related_record_model = Lead`, for create, update and delete |
| Activity Plan Template | Manager can manage lead plan templates | Administrator | the plan's model is Lead, for create, update and delete |
| Discussion Channel | Sales users read the channel of a lead | User: Own Documents Only | `has_crm_lead` is true, read only |
| Discussion Channel Membership | Sales users read and create members on the channel of a lead | User: Own Documents Only | the channel produced a Lead; read and create only |
| Text Message Template | Administrator on lead and contact templates | Administrator | the template's model is Lead or Contact, for create, update and delete |
| Event Lead Rules | Multi-company | Multi-company users | `company_id` is among the reader's allowed companies or is empty |
| Customer Relationship Management Lead Generation Rules | All Rules | User: All Documents | everything |
| Customer Relationship Management Lead Generation Rules | Personal or Global Rules | User: Own Documents Only | `user_id` is the reading user or is empty |
| Customer Relationship Management Reveal View | All Views | User: All Documents | everything |
| Customer Relationship Management Reveal View | Personal or Global Views | User: Own Documents Only | the rule of the view has `user_id` is the reading user or is empty |
| Partnership Analysis | All Assignations | User: All Documents | everything |
| Partnership Analysis | Personal or Global Assignations | User: Own Documents Only | `user_id` is the reading user or is empty |
| Partner Grade | Published grades only | Portal user, Public user | `published = true`, read only |

The per-entity create, read, update and delete matrix is in [business-rules.md](business-rules.md), rule `LEAD-RULE-180`.

## 7. Shipped master data

### 7.1 Pipeline stages

| Name | Sequence | Won flag | Folded | Colour index |
|---|---|---|---|---|
| New | 1 | no | no | 11 |
| Qualified | 2 | no | no | 5 |
| Proposition | 3 | no | no | 8 |
| Won | 70 | **yes** | no | 10 |

None of them is restricted to a team, which matters for the scoring model (see [predictive-lead-scoring.md](predictive-lead-scoring.md) section 3.3). The large gap between the sequence of `Proposition` and that of `Won` leaves room for intermediate stages.

### 7.2 Lost reasons

| Name |
|---|
| Too expensive |
| We don't have people/skills |
| Not enough stock |

### 7.3 Recurring revenue plans

| Name | Months |
|---|---|
| Monthly | 1 |
| Yearly | 12 |
| Over 3 years | 36 |
| Over 5 years | 60 |

### 7.4 Sales teams

| Name | Sequence | Company | Leader | Active | Uses opportunities | Alias |
|---|---|---|---|---|---|---|
| Sales | 0 | none | the shipped administrator user | yes | yes | `info` |
| Website | not set | none | none | **no** | **no** | none |
| Point of Sale | not set | none | none | **no** | **no** | none |

The two inactive teams exist so that the storefront and the point of sale have a team to attach their documents to; they cannot be deleted (`LEAD-RULE-109`).

### 7.5 Sales team memberships

| Team | User | Capacity |
|---|---|---|
| Sales | the shipped administrator user | 30, the default |

### 7.6 Scoring variable catalogue

| Catalogue entry | Field of the Lead |
|---|---|
| Country | `country_id` |
| State | `state_id` |
| Phone Quality | `phone_state` |
| Email Quality | `email_state` |
| Source | `source_id` |
| Language | `lang_id` |
| Tags | `tag_ids` |

### 7.7 Tags shipped by the reseller package

| Name | Colour index | Used by |
|---|---|---|
| No more partner available | 3 | added when the geographic search finds no partner (`LEAD-RULE-168`) |
| Spam | 3 | added when a reseller declines a lead as spam (`LEAD-RULE-170`) |
| Created by Partner | 4 | added to an opportunity created by a reseller on the portal (`LEAD-RULE-164`) |

### 7.8 Partner grades and activation levels

| Entity | Name | Sequence | Level weight |
|---|---|---|---|
| Partner Grade | Gold | 1 | 1, the default |
| Partner Grade | Silver | 2 | 1 |
| Partner Grade | Bronze | 3 | 1 |
| Partner Activation | Fully Operational | 1 | not applicable |
| Partner Activation | Ramp-up | 2 | not applicable |
| Partner Activation | First Contact | 3 | not applicable |

The shipped weights are all one, which makes the geographic draw uniform among the candidates until an administrator differentiates them.

### 7.9 Message subtypes

Five subtypes are defined on the Lead and five parent subtypes on the Sales Team, so that a follower of a team is notified of what happens on its Leads.

| Subtype on the Lead | Description | Default subscription | Parent subtype on the Sales Team | Sequence |
|---|---|---|---|---|
| Opportunity Created | Lead/Opportunity created | hidden, not default | Opportunity Created | 10 |
| Stage Changed | Stage changed | not default | Opportunity Stage Changed | 11 |
| Opportunity Won | Opportunity won | not default | Opportunity Won | 12 |
| Opportunity Lost | Opportunity lost | not default | Opportunity Lost | 13 |
| Opportunity Restored | Opportunity restored | not default | Opportunity Restored | 14 |

The team-side subtypes are linked to their Lead-side counterpart through the team reference of the Lead. Only the first team-side subtype is a default subscription.

### 7.10 Campaign source shipped by the live chat bridge

| Entity | Name |
|---|---|
| Campaign Source | Livechat |

### 7.11 The lead generation chatbot script

| Step | Sequence | Kind | Message |
|---|---|---|---|
| 1 | 1 | free multi-line input | `Hi there, what brings you to our website today? 👋` |
| 2 | 2 | forward to an operator | none |
| 3 | 3 | text | `Hu-ho, it looks like none of our operators are available 🙁` |
| 4 | 4 | email question | `Would you mind leaving your email address so that we can reach you back?` |
| 5 | 5 | create a lead | `Thank you, you should hear back from us very soon!` |

The script is named `Lead Generation Bot`.

### 7.12 Email templates

| Template | Model | Purpose |
|---|---|---|
| Lead Forward: Send to partner | Lead Forward to Partner Wizard | Sent to a reseller when one or several Leads are forwarded. The subject is `Fwd: Lead: <partner name>`; the sender is the acting user; the recipient is the partner; the language is the language of the partner; the message is deleted after sending. |
| Lead Generation Notification | the external credit account entity | Sent when a lead mining or lead generation request runs out of credits. |

### 7.13 Sales goal definitions

The gamification bridge ships ten goal definitions and two challenges. A replacement that offers sales goals must reproduce the definitions; a replacement that does not offer them may omit this section entirely without affecting any other rule of this domain.

| Definition | Mode | Measured on | Date field | Condition | Unit |
|---|---|---|---|---|---|
| Total Invoiced | sum of the untaxed amount | customer invoice analysis | invoice date | state is not cancelled and the document is a customer invoice | currency |
| New Leads | count | Lead | creation date | `type` is `lead` or `opportunity` | leads |
| Time to Qualify a Lead | sum of the days to close, lower is better | Lead | closing date | `type = "lead"` | days |
| Days to Close a Deal | sum of the days to assign, lower is better | Lead | assignment date | none | days |
| New Opportunities | count | Lead | assignment date | `type = "opportunity"` | opportunities |
| New Sales Orders | count | Sales Order | order date | state is not draft, sent or cancelled | orders |
| Paid Sales Orders | count | customer invoice analysis | invoice date | payment state is paid or in payment, and the document is a customer invoice | orders |
| Total Paid Sales Orders | count of the untaxed amount | customer invoice analysis | invoice date | same as above | currency |
| Customer Credit Notes | count, lower is better | customer invoice analysis | invoice date | state is not cancelled and the document is a customer credit note | invoices |
| Total Customer Credit Notes | sum of the untaxed amount, higher is better | customer invoice analysis | invoice date | same as above | currency |

Every definition is measured per salesperson.

| Challenge | Period | Visibility | Audience | Report frequency | Lines |
|---|---|---|---|---|---|
| Monthly Sales Targets | monthly | ranking | users in the group User: Own Documents Only | weekly | Total Invoiced, target 20 000 |
| Lead Acquisition | monthly | ranking | users in the group User: Own Documents Only | weekly | New Leads target 7; Time to Qualify a Lead target 15; New Opportunities target 5 |

### 7.14 Digest tips

Eight tips are shipped for the digest email of the employee services domain, in this order of appearance:

| Sequence | Title | Audience |
|---|---|---|
| 200 | Tip: Convert incoming emails into opportunities | User: All Documents |
| 1500 | Tip: Did you know the system has built-in lead mining? | User: All Documents |
| 1700 | Tip: Opportunity win rate is predicted with Artificial Intelligence | User: All Documents |
| 2600 | Tip: Manage your pipeline | User: All Documents |
| 2800 | Tip: Do not waste time recording customers' data | User: All Documents |
| 3000 | Tip: Turn a selection of opportunities into a map | User: All Documents |
| 3600 | Tip: Identify Bottlenecks at a glance | User: Own Documents Only |
| 4400 | Tip: Turn Sales Forecasting into Child's Play | User: Own Documents Only |

The shipped digest activates the two indicators of this domain, New Leads and Opportunities Won.

### 7.15 Lead mining reference data

The lead mining package ships no industry, role or seniority record: those lists are filled from the external service catalogue on installation of the data set of the deployment. The three entities exist with a uniqueness constraint on their name (`Industry name already exists!`, `Role name already exists!`, `Name already exists!`).

## 8. Master-data prerequisites owned by other domains

| Prerequisite | Owning domain | Why this domain needs it |
|---|---|---|
| Countries and country subdivisions | [Contacts and Organizations](../contacts-and-organizations/README.md) | Address of a Lead, telephone parsing, geographic partner search, scoring variables. |
| Languages | [Contacts and Organizations](../contacts-and-organizations/README.md) | Language of a Lead, scoring variable, language of the forwarding email. |
| Currencies and companies | [Contacts and Organizations](../contacts-and-organizations/README.md) and [General Ledger](../general-ledger/README.md) | Currency of the monetary fields, company scoping. |
| Email aliases and the incoming mail gateway | [Messaging, Activities and Collaboration](../messaging-and-activities/README.md) | The alias of a Sales Team that creates Leads from incoming email. |
| Activity types and activity plans | [Messaging, Activities and Collaboration](../messaging-and-activities/README.md) | Next activity of a Lead; the shipped activity type list is filtered to those with no model or with the model Lead or Contact. |
| Email and telephone blacklists, telephone formatting | [Messaging, Activities and Collaboration](../messaging-and-activities/README.md) | Quality flags, blacklist flags, telephone reformatting. |
| Campaigns, sources and mediums | [Marketing](../marketing-and-mass-mailing/README.md) | Campaign tracking fields of a Lead and the shipped source `Livechat`. |
| Meetings | [Calendar and Scheduling](../calendar-and-scheduling/README.md) | Meetings of an opportunity. |
| Contact grades and activation levels | [Contacts and Organizations](../contacts-and-organizations/README.md) | Reseller programme; the entities are shipped by this domain's reseller package but belong to the contacts domain. |

## 9. Menus

| Menu | Parent | Sequence | Visible to |
|---|---|---|---|
| Customer Relationship Management | root | 25 | User: Own Documents Only, Administrator |
| Sales | the application root | 1 | |
| My Pipeline | Sales | 1 | |
| My Activities | Sales | 2 | User: Own Documents Only |
| Teams | Sales | 4 | |
| Customers | Sales | 5 | |
| Leads | the application root | 5 | Show Lead Menu |
| Reporting | the application root | 20 | User: Own Documents Only |
| Forecast | Reporting | 1 | |
| Pipeline | Reporting | 2 | |
| Leads | Reporting | 3 | |
| Activities | Reporting | 4 | |
| Partnerships | Reporting | 5 | with the reseller package |
| Lead Generation Views | Reporting | not set | technical group only |
| Configuration | the application root | 25 | Administrator |
| Settings | Configuration | 0 | Platform administrator |
| Opportunities | Configuration | 1 | Administrator |
| Sales Teams | Configuration | 5 | |
| Teams Members | Configuration | 6 | technical group only |
| Activities | Configuration | 8 | |
| Activity Types | Configuration, Activities | 10 | |
| Activity Plans | Configuration, Activities | 11 | Administrator |
| Recurring Plans | Configuration | 12 | Show Recurring Revenues Menu |
| Pipeline | Configuration | 15 | Administrator |
| Stages | Configuration, Pipeline | 0 | technical group only |
| Tags | Configuration, Pipeline | 1 | |
| Lost Reasons | Configuration, Pipeline | 6 | |
| Members / Partners | Configuration | 16 | with the membership package; the label is the company's affiliate label |
| Levels | Configuration, Members / Partners | 1 | |
| Partner Activations | Configuration, Members / Partners | 2 | |
| Lead Generation | Configuration | 20 | with the mining package |
| Lead Mining Requests | Configuration, Lead Generation | 0 | |
| Visits to Leads Rules | Configuration, Lead Generation | 5 | with the website identification package |
| Import and Synchronize | the application root | not set | |

The Configuration menu carries the pipeline action itself, so that opening it lands on the pipeline rather than on an empty page.

## 10. Company-level settings

| Setting | Entity | Type | Default | Effect |
|---|---|---|---|---|
| Affiliate label | Company | text, translatable | `Members` | The word used to name affiliates in menus and screens. A deployment chooses whichever word fits its programme, for instance `Partners`, `Members` or `Alumni`. It is editable from the application settings and propagates to the affiliate menu, to the grade screens and to the pricelist screens. |

## 11. Website-level settings

| Setting | Entity | Type | Default | Effect |
|---|---|---|---|---|
| Default Sales Teams | Website | many_to_one to Sales Team | empty | The team written on a Lead created from the contact form when the form does not carry one. The selector offers teams that use leads when the reader belongs to the group Show Lead Menu, and teams that use opportunities otherwise. |
| Default Salesperson | Website | many_to_one to User | empty | The salesperson written on a Lead created from the contact form when the form does not carry one. |

The Lead entity is published as a form target with the key `create_lead`, the default text field `description`, public submission allowed, and the label `Create an Opportunity`.

## 12. Survey-level and event-level settings

| Setting | Entity | Type | Effect |
|---|---|---|---|
| Assign Leads to | Survey | many_to_one to Sales Team | The team written on the Leads created from the participations of that survey. |
| Lead creation | Survey Answer Option | boolean | Choosing this answer creates a Lead. |
| Sales Team | Chatbot Script Step | many_to_one to Sales Team | The team written on the Lead created by a step of kind "create a lead" or "create a lead and forward". |

Event lead rules are full records rather than settings; see [entities.md](entities.md) section 13.1.
