# Interfaces

Everything a client, an integration or a user touches: the service operations invoked on the entities of this domain, the request endpoints, the screens described as workflows on views, the analytical reports, the notifications, the printed and exported documents, and the scheduled jobs. No client technology is named; a screen is described by the fields it shows, the buttons it offers, the guard of each button, the filters, the groupings and the status bar.

## 1. Service operations on the Lead

Operation names are given as stable full-word identifiers. They are the names **this specification
assigns** so that the other documents can cite an operation unambiguously; they are not strings
taken from an external contract, and a rebuild may choose its own. Every operation applies to a set
of records unless stated otherwise.

### 1.1 Life-cycle operations

| Operation | Inputs | Effect | Output | Errors |
|---|---|---|---|---|
| `create_lead` | the field values | Creates one or several Leads, cleans the web address, computes the derived fields, sets the closing date when the record lands in a won stage, links the commercial entity as customer when the conditions of [entities.md](entities.md) section 1.12 hold, runs the won and lost bookkeeping, posts the creation message. | the created records | the storage-level refusals of `LEAD-001`, `LEAD-005`, `LEAD-006`, `LEAD-009`, `LEAD-119` |
| `update_lead` | the field values | Writes the values, applying the date rules `LEAD-043` to `LEAD-045`, the won stage forcing of `LEAD-041`, and the won and lost bookkeeping. | true | the same refusals, plus `LEAD-007` |
| `duplicate_lead` | none | Copies a record. `stage_id`, `probability` and `date_closed` are not copied; `type` and `team_id` are re-imposed from the source; `date_open` becomes the current instant for an opportunity whose salesperson is active and is empty otherwise; an inactive salesperson is dropped; the recurring amount and plan are dropped for a user outside the recurring revenue group. | the copy | none |
| `archive_lead` | none | Sets `active = false`. Does not by itself make the record lost. | true | none |
| `unarchive_lead` | none | Sets `active = true`, clears the lost reason, recomputes the automated probability. Does not realign the probability. | true | none |
| `restore_lead` | none | Performs `unarchive_lead` and then writes `probability` set equal to `automated_probability`. | true | none |
| `set_lead_lost` | optionally a lost reason and additional values | Archives, then writes a probability of zero, an automated probability of zero, and the additional values. | true | `LEAD-006` |
| `set_lead_won` | none | Unarchives, chooses a won stage per record, writes the stage and a probability of one hundred grouped by target stage. | true | none |
| `set_lead_won_with_celebration` | none, single record | Performs `set_lead_won` and returns the celebration effect when a message applies. | a celebration effect or true | none |
| `get_celebration_message` | none, single record | Returns the celebration message when the record currently sits in a won stage, and nothing otherwise. | text or nothing | none |
| `delete_lead` | none | Detaches the meetings that point at the record through the generic document reference, then deletes. | true | reserved to the sales administrator |
| `use_automated_probability` | none, single record | Recomputes the record and writes the automated value into the probability. | true | none |

### 1.2 Conversion and merging

| Operation | Inputs | Effect | Output | Errors |
|---|---|---|---|---|
| `convert_lead_to_opportunity` | the customer to link, optionally a list of salespeople, optionally a team | Per record, skipping archived and won records: writes `type = "opportunity"`, the conversion instant, the customer when it differs, and a stage when the record had none. Then distributes the salespeople round robin and writes the team. | true | none |
| `merge_leads` | optionally a salesperson, optionally a team, optionally a flag telling whether to delete the merged-away records | Orders by confidence, computes the merged values, moves followers, logs the summary, moves history, activities, attachments and meetings, repairs the stage, writes the survivor, deletes the merged-away records unless asked not to. | the surviving record | `LEAD-066`, `LEAD-067` |
| `assign_customers` | optionally a customer to force, a flag telling whether to create the missing customers, optionally a parent company | Writes the forced customer on every record; creates a customer from the record data for the records that still have none, when creation is allowed. | true | none |
| `assign_salespeople` | a list of salespeople, optionally a team | Distributes the salespeople over the records in strides, as described in [lead-assignment.md](lead-assignment.md) section 11. | true | none |
| `find_lead_duplicates` | optionally a customer, optionally an email, a flag telling whether to include lost records | Returns the records matching the search of `LEAD-061`. | a set of records | none |
| `find_matching_contact` | none, single record | Returns the customer of the record, or the contact found from its email, without creating one. | a contact or nothing | none |
| `create_customer_from_lead` | optionally a parent company | Creates the contact described in [entities.md](entities.md) section 8.8 and returns it. | a contact | none |

### 1.3 Scoring operations

| Operation | Inputs | Effect | Output | Errors |
|---|---|---|---|---|
| `compute_probabilities` | none | Recomputes `automated_probability` for the records in scope and realigns `probability` where the record is active and automatic. | true | none |
| `prepare_scoring_explanation` | none, single record | Recomputes the record, writes the automated value and the probability when automatic, and returns the explanation structure of [predictive-lead-scoring.md](predictive-lead-scoring.md) section 5. | a structure with the probability, the team name, the three positive factors and the three negative factors | none |
| `rebuild_scoring_frequency_table` | none | Empties and rebuilds the frequency table. | true | `LEAD-112` |
| `update_automated_probabilities` | none | Recomputes every open record created on or after the scoring start date, in batches. | true | none |
| `recompute_scoring` | none | Performs the rebuild and then the recomputation. | true | `LEAD-112` |

### 1.4 Navigation operations

These return a screen definition rather than changing data.

| Operation | Returns |
|---|---|
| `open_meeting_calendar` | The meetings screen, pre-filled with the opportunity, its customer, the reader, the team and the record title, and positioned on the most relevant period (see [workflows.md](workflows.md) section 3). |
| `reschedule_meeting` | The same screen without the period heuristic, positioned on the meeting of the reader's own activity. |
| `show_potential_duplicates` | The opportunities screen restricted to the potential duplicates of the record, archived records included, creation disabled. |
| `open_lead_or_opportunity` | The form of the record, with a condition restricting the screen to records of the same type. |
| `open_live_chat_conversation` | The conversation that produced the record. |
| `open_page_views` | The page view screen restricted to the visitors linked to the record, grouped by page when there are more than fifteen views over more than one page. |
| `open_live_chat_sessions` | The conversation screen restricted to the conversations of the visitors linked to the record. |
| `open_new_quotation` | The quotation creation screen pre-filled from the opportunity; when the opportunity has no customer, the customer is asked for first. |
| `open_quotations` | The quotation screen restricted to the linked orders that are draft or sent. |
| `open_sales_orders` | The order screen restricted to the linked orders that are neither draft, nor sent, nor cancelled. |
| `open_event_registrations` | The registration screen restricted to the registrations behind the record, creation disabled. |

## 2. Service operations on the other entities

### 2.1 Sales Team

| Operation | Inputs | Effect | Output | Errors |
|---|---|---|---|---|
| `assign_leads` | none | Runs both phases of the assignment over the teams in scope, with the quota forced and no creation window, logs a note per team and returns a notification. | a notification | `LEAD-098` |
| `assign_leads_scheduled` | optionally a force flag, optionally a creation window in days | Runs both phases over every team that uses leads or opportunities and is not opted out. | true | `LEAD-098` |
| `allocate_leads_to_teams` | optionally a creation window in days | Phase one only. | the per-team result structure | none |
| `distribute_leads_to_members` | optionally a force flag | Phase two only. | the per-member result structure | none |
| `open_my_pipeline` | none | Returns the pipeline screen positioned on the reader's own team. | a screen | none |
| `open_forecast` | none | Returns the forecast screen positioned on the reader's own team. | a screen | none |
| `open_team_leads` | none | Returns the Leads screen of the team, or the opportunities screen when the team does not use leads. | a screen | none |
| `open_unassigned_leads` | none | Returns the Leads screen of the team restricted to records with no salesperson. | a screen | none |
| `update_invoicing_target` | an amount | Writes the monthly revenue target of the team. | true | reserved to the sales administrator |

### 2.2 Lost Reason

| Operation | Effect |
|---|---|
| `open_lost_leads` | Returns the Lead screen restricted to the records carrying the reason, archived records included, creation disabled. |

### 2.3 Campaign and Mass Mailing

| Operation | Effect |
|---|---|
| `open_campaign_leads` | Returns the Leads screen, or the opportunities screen when the reader is outside the group Show Lead Menu, restricted to the records of the campaign, archived records included, creation disabled. |
| `open_mailing_leads` | Returns the same kind of screen restricted to the records whose source is the source of the mailing. |

### 2.4 Contact

| Operation | Effect |
|---|---|
| `open_contact_opportunities` | Returns the opportunity screen restricted to the contact and all its descendants, archived records included, with the won, ongoing and lost filters pre-selected, list view first. |

### 2.5 Lead Mining Request

| Operation | Effect | Errors |
|---|---|---|
| `submit_mining_request` | Assigns a number when the request is still named `New`, builds the payload, calls the service, creates the Leads, posts a message per Lead, and sets the state. | `LEAD-125` |
| `reset_mining_request_to_draft` | Sets the state to draft and resets the number to the literal text `New`. | none |
| `open_generated_leads` | Returns the Leads screen restricted to the records of the request. | none |
| `open_generated_opportunities` | Returns the opportunities screen restricted to the records of the request. | none |
| `buy_credits` | Returns the external credit purchase screen. | none |

### 2.6 Enrichment

| Operation | Effect | Errors |
|---|---|---|
| `enrich_leads` | Calls the enrichment service for the records in scope and applies the answer as described in [workflows.md](workflows.md) section 2.8. | `LEAD-129` |
| `enrich_leads_scheduled` | Selects the eligible records created in the last twenty-four hours and enriches them in batches. | the same, suppressed |

### 2.7 Website identification

| Operation | Effect |
|---|---|
| `generate_leads_from_visits` | Deletes the Reveal View records whose address already produced a Lead within the retention window, groups the remaining ones by address, resolves them and creates the Leads. |
| `clean_reveal_views` | Deletes the Reveal View records older than one month. |
| `open_rule_leads` / `open_rule_opportunities` | Returns the Lead or opportunity screen restricted to the records of the rule. |

### 2.8 Event lead rules

| Operation | Effect | Errors |
|---|---|---|
| `run_event_lead_rules` | Applies the rules in scope to a set of registrations, creating or updating the Leads. | none |
| `regenerate_event_leads` | Regenerates the Leads of an event, synchronously below the volume threshold and through a generation request above it. | `LEAD-123`, `LEAD-122` |
| `generate_event_leads_scheduled` | Processes the pending generation requests in batches, resuming from the last processed registration. | none |
| `add_lead_rule_from_answer` | Called on one Event Question Answer. Returns the creation dialogue of an Event Lead Rules record, opened as a modal and pre-filled with the label of the answer as the rule name, the acting user as the salesperson written on the created records, and the registration condition `registration_answers.question IN (the question of this answer) AND registration_answer_options.answer_option IN (this answer)`. Nothing is written until the user saves the dialogue. | none |

### 2.9 Reseller operations

| Operation | Inputs | Effect | Errors |
|---|---|---|---|
| `assign_partner_geographically` | none | Geolocates the records that have a country, searches candidates in widening windows, draws one weighted by level weight, writes it, and writes the salesperson of the partner. | `LEAD-140` (a warning, not a refusal) |
| `assign_partner` | optionally a partner to force | The same, with the search skipped when a partner is forced. | none |
| `forward_leads_to_partner` | the forwarding mode, the partner or the proposed pairs, the message body | Groups the records by recipient, renders and sends the template once per recipient, writes the assigned partner and the salesperson without notifying, subscribes the partner. | `LEAD-144` |
| `partner_accepts_lead` | optionally a comment | Posts the acceptance message and converts the record into an opportunity. | `LEAD-136` |
| `partner_declines_lead` | optionally a comment, a contacted flag, a spam flag | Posts the refusal message, unsubscribes the partner family, clears the assigned partner, records the family as having declined, adds the spam tag when asked. | `LEAD-136` |
| `partner_updates_lead` | expected revenue, probability, priority, expected closing date, activity type, activity summary, activity deadline | Writes the four record values and updates or creates the portal user's own activity. | `LEAD-136` |
| `partner_updates_contact_details` | a map of field values | Writes only the allowed fields. | `LEAD-136`, `LEAD-138` |
| `partner_updates_stage` | a stage | Writes the stage. | `LEAD-136` |
| `partner_creates_opportunity` | contact name, description, title | Creates a record with priority `2`, the reader's commercial entity as assigned partner and the shipped tag, assigns the salesperson of that partner and converts it. | `LEAD-139` |

### 2.10 Activities and meetings

| Operation | Inputs | Effect | Errors |
|---|---|---|---|
| `create_calendar_event_from_activity` | none | Called on an Activity. When the meeting already attached to that activity points at an opportunity, the meeting defaults of that opportunity are merged into the screen definition returned by the messaging domain: the opportunity, the customer of the opportunity as default customer, the acting user's contact plus that customer as default attendees, the team of the opportunity, the title of the opportunity as default title, and the start of the existing meeting as the initial date. When the meeting points at no opportunity, the screen definition of the messaging domain is returned unchanged. | none |

## 3. Request endpoints

### 3.1 Action links sent by email

| Path | Method | Authentication | Purpose | Response |
|---|---|---|---|---|
| `/lead/case_mark_won` | read | an authenticated internal user | Marks the record won, with the celebration computation. Requires a record identifier and a signed token that matches it. | a redirection to the record, or to the generic fallback screen when the operation fails |
| `/lead/case_mark_lost` | read | an authenticated internal user | Marks the record lost with no reason. | the same |
| `/lead/convert` | read | an authenticated internal user | Converts the record into an opportunity keeping its current customer. | the same |

All three verify the token against the record before acting and silently fall back when the operation raises.

### 3.2 Mail plugin endpoints

| Path | Method | Authentication | Inputs | Output |
|---|---|---|---|---|
| `/mail_plugin/lead/create` | remote call | the mail plugin identity | the contact identifier, the email body, the email subject | the identifier of the created record, or the error code `partner_not_found` |

The created record carries the company of the contact, the plain text of the subject as its title, the contact as customer and the email body as description.

### 3.3 Reseller portal endpoints

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/my/leads` and `/my/leads/page/<page>` | read | an authenticated portal user | The paginated list of the Leads assigned to the reader's commercial family, with a date range filter and a sort order. |
| `/my/opportunities` and `/my/opportunities/page/<page>` | read | an authenticated portal user | The same for opportunities, with an additional filter by state. |
| `/my/lead/<lead_id>` | read | an authenticated portal user | The detail page of one Lead. |
| `/my/opportunity/<lead_id>` | read | an authenticated portal user | The detail page of one opportunity, with the acceptance, refusal, update and stage operations. |

Every one of them is restricted by the portal visibility rule of `LEAD-135` and by the write guard of `LEAD-136`.

### 3.4 Public reseller directory

| Path | Method | Authentication | Purpose |
|---|---|---|---|
| `/partners` and `/partners/page/<page>` | read | public | The published reseller directory, ordered by grade sequence then by implementation reference count. |
| `/partners/grade/<grade_name>` and its paginated form | read | public | The directory filtered on one grade. |
| `/partners/country/<country_name>` and its paginated form | read | public | The directory filtered on one country. |
| `/partners/grade/<grade_name>/country/<country_name>` and its paginated form | read | public | The directory filtered on both. |
| the detail path of one published reseller | read | public | The public page of one reseller, with its address, its grade, its description and its implementation references. |

**Which resellers the directory lists.** A reseller appears only when it is a company contact, carries a grade, is itself published, and its grade is not archived. A reader who is not a website editor additionally sees only resellers whose grade is published (`LEAD-154` and the grade rule of [configuration.md](configuration.md) section 6). A free-text search matches the name, the public description, the two street lines, the city, the postal code, the country subdivision and the country. An industry filter keeps the resellers having at least one implementation reference in that industry.

**Ordering and paging.** Grade sequence ascending, then implementation reference count descending, then complete name ascending, then identifier ascending. Forty resellers per page. When no reseller at all matches, the page is still rendered but answered with the not-found status, so that an empty directory is not indexed as a valid page.

**Country of the visitor.** When neither a country nor the explicit "all countries" choice is part of the request, the country is inferred from the network address of the visitor. When the inferred country holds no listable reseller, the country filter is dropped and the whole directory is shown instead, with the country selector back on **All Countries** (`LEAD-153`). The country selector always shows every country that holds at least one listable reseller with its count, plus an **All Countries** entry carrying the total; the grade selector is built the same way over the grades.

**Publication.** The publication control of a reseller page is offered only to a reader who may write on the Contact record behind it (`LEAD-152`). Website editing rights alone do not grant it.

**Page inventory.** The directory path is registered as listable website content under the name `Partners`, so that it appears among the site pages and in the site map together with one entry per grade and per country that holds a listable reseller.


### 3.5 Website contact form

The Lead entity is registered as a public form target under the key `create_lead`, with `description` as the default free-text field and the label `Create an Opportunity`. The form handler applies `LEAD-118`, the telephone reformatting and the subdivision inference described in [workflows.md](workflows.md) section 2.3.

## 4. Screens

### 4.1 The Lead form

**Status bar.** The stage, clickable, restricted to the stages with no team or with the team of the record, hidden for a record of type `lead`, read only when the record is lost or archived. It also shows the time spent in each stage and highlights a rotting record.

**Header buttons.**

| Button | Guard |
|---|---|
| Won | visible when the won status is not `won`, the type is `opportunity` and the record is active |
| Convert to Opportunity | visible when the type is `lead` and the record is active |
| Restore | visible when the won status is `lost` |
| Lost | visible when the won status is `pending` and the record is active |

**Ribbons.** `Archived` when the record is archived and neither won nor lost; `Lost` when the won status is `lost`; `Won` when the won status is `won`.

**Counter buttons.**

| Button | Shows | Guard |
|---|---|---|
| Meetings | the meeting label (`Next Meeting`, `Last Meeting` or `No Meeting`) and the date | visible on a saved record of type `opportunity` |
| Similar Leads | the duplicate count, labelled `Similar Lead` in the singular and `Similar Leads` in the plural | visible when the duplicate count is at least one |
| Quotations, Orders, Registrations, Page Views, Live Chat Sessions | the matching counter | visible when the matching bridge is installed and the counter is not zero |

**Title block.** The title, then, for an opportunity, the expected revenue in the company currency, then, for a user in the recurring revenue group, the recurring revenue and the recurring plan (the plan becomes required as soon as the recurring revenue is not zero), then the probability with:

- a button that switches back to the automated value, visible when the probability is manual and the record is not lost;
- an explanation button, visible when the record is pending and the probability is automatic;
- a small read-only display of the automated value, visible when the probability is manual and the record is not lost;
- the probability itself, read only unless the record is pending.

**Main fields, for a record of type `lead`.** Customer (only when set or in the technical mode), company name, the six address fields, web address, language (hidden when only one language is installed), contact name, email, additional copy addresses (technical mode only), job position, telephone, salesperson, team, expected closing date, priority, tags, properties.

**Main fields, for a record of type `opportunity`.** Customer, the address block as a read-only summary, email, telephone, lost reason (visible only when the won status is `lost`), salesperson, team, expected closing date shown next to the priority, tags, properties.

**Blacklist buttons.** Next to the email and next to the telephone, a button appears when the address or the number is blacklisted, or when the customer is blacklisted, and removes it from the blacklist.

**Warnings.** A note appears next to the email and next to the telephone when saving will also modify the customer record (`LEAD-028`).

**Notebook.**

| Page | Content |
|---|---|
| Notes | The description, as collaborative rich text. |
| Extra Info | The delivery failure counter, the company, the campaign, the medium, the source, the referrer, the assignment date and the closing date. |
| Contacts | For a record of type `lead`, the full contact and address block, the campaign tracking fields and the company. |

**Discussion thread.** Messages, activities, followers and attachments, with the reply address equal to the alias of the team.

### 4.2 The pipeline (kanban of opportunities)

- Grouped by stage by default, with the column set expanded by `LEAD-169`.
- Quick creation opens a reduced form asking for the title, the customer or the company, the expected revenue and the recurring figures.
- Cards are coloured by the colour index of the record and highlighted when the record is rotting.
- Archiving from the kanban is disabled; losing a record goes through the Lost dialogue.
- A card shows the title, the customer, the expected revenue, the tags, the priority stars, the next activity indicator and the salesperson avatar.
- Dragging a card into a column flagged as won triggers `LEAD-041`.

### 4.3 The Leads list

**Header button.** Mark Lost, which opens the Lost dialogue on the selection.

**Columns**, with the optional ones marked: creation date (optional, hidden), title, contact name (optional, hidden), company name (optional, hidden), email (optional, shown), telephone (optional, hidden), company (optional, hidden, multi-company only), city (optional, shown), state (optional, hidden), country (optional, shown), salesperson (optional, shown), team (optional, shown), campaign (optional, hidden), medium (optional, hidden), source (optional, hidden), probability (optional, hidden), tags (optional, hidden), priority (optional, hidden).

The list supports multiple-record editing.

### 4.4 The opportunities list

Same structure, with Mark Lost, Send Email in single mode and Send Email in batch mode as header buttons.

When the text message capability package is installed, two more controls appear on this screen and on the pipeline. The first is a header button that opens the text message composer in batch mode over the selected records, with the option that keeps a log of what was sent already enabled. The second sits on each row of the opportunities list, next to the email button, carries a speech-bubble icon, and opens the composer in single mode on that record; it is hidden when the record is lost, and it is absent from the reporting variant of the list. Both controls are labelled with the short user-facing name of a text message.

### 4.5 The Leads search panel

**Free-text field.** Typing in the search box matches the company name, the email, the contact name or the title.

**Fields offered.** tags, salesperson, team, country, city, telephone (matching both the telephone number and its sanitized form), language, creation date, source, medium, campaign, activity state, properties, activity responsible, activity type.

**Filters.** My Leads; Unassigned (no salesperson and type `lead`); Lost (won status `lost` and archived); Creation Date, defaulting to the current month; Closed Date; Archived; and four hidden activity filters (My Activities, Late Activities, Today Activities, Future Activities).

**Groupings.** Salesperson, Sales Team, City, Country, Company (multi-company only), Campaign, Medium, Source, Creation Date by month, Closed Date, Properties.

### 4.6 The opportunities search panel

**Free-text field.** Matches the customer, the company name, the email, the title or the contact name.

**Fields offered.** customer (matching the customer and all its descendants), tags, salesperson, team, stage, country, city, telephone, activity state, properties.

**Filters.** My Pipeline; Unassigned; Open Opportunities (probability below one hundred, type `opportunity`, active); Unread Messages; Creation Date; Closed Date; Won; Ongoing (pending and active); Rotting; Lost; plus the hidden filters Overdue Opportunities (no closing date and an expected closing date in the past) and the three activity filters.

**Groupings.** Salesperson, Sales Team, Stage, City, Country, Lost Reason, Company, Campaign, Medium, Source, Creation Date by month or by day, Conversion Date (only for a reader in the group Show Lead Menu), Expected Closing, Closed Date, Properties.

### 4.7 The forecast

A variant of the opportunity screens built around the expected closing date.

- The extra filter **Upcoming Closings** keeps the records whose expected closing falls in the current or a future period.
- The default grouping is by expected closing month.
- The kanban shows one column per period and a running total of the prorated revenue.
- The graph and the pivot measure the prorated revenue rather than the expected revenue.

### 4.8 The calendar

Leads placed on the deadline of their next activity, coloured by salesperson, month mode, creation disabled, at most five events shown per day.

### 4.9 The activity screen

Leads in rows, activity types in columns, each cell showing the state of that activity type on that record.

### 4.10 The graph and the pivot

| Screen | Rows | Columns | Measure |
|---|---|---|---|
| Pipeline Analysis, graph | stage, then salesperson | | the record count by default |
| Pipeline Analysis, pivot | creation date by month | stage | expected revenue |
| Forecast, graph | expected closing date | | prorated revenue |
| Forecast, pivot | expected closing date by month | stage | prorated revenue |
| Leads Analysis | as configured | | the record count |
| Activity Analysis, pivot | message date by month | message subtype, then activity type | the row count |
| Activity Analysis, graph | message date by month, split by subtype | | the row count, as a bar chart |

The measures available on every one of them include the expected revenue, the prorated revenue, the probability, the automated probability, the days to assign, the days to close, the delivery failure counter and, for a user in the recurring revenue group, the four recurring figures.

### 4.11 The Sales Team card

A card per team showing the name, the leader, the members, the invoicing target progress, and a main button whose label is `Pipeline` when the team uses opportunities and `Sales Analysis` inside the sales application. The card also offers, when rule-based assignment is on, the unassigned count, the monthly assigned count with a warning when the capacity is exceeded, and an assignment button.

### 4.12 The Sales Team form

| Section | Fields |
|---|---|
| Header | name, active, company, leader, sequence, colour |
| Options | uses leads, uses opportunities, email alias |
| Members | the membership list with the user, the capacity, the pause flag, the assignment condition and the preference condition |
| Assignment | the assignment condition of the team, the pause flag, the derived capacity, the unassigned count, the monthly assigned count, and the assignment button |
| Properties | the definition of the dynamic properties available on the Leads of the team |

A warning line appears under the member list when single-membership mode is active and a chosen user already belongs to another team.

### 4.13 The dialogues

| Dialogue | Fields | Buttons |
|---|---|---|
| Convert to opportunity | conversion action (`Convert to opportunity` or `Merge with existing opportunities`), customer handling (`Create a new customer` or `Link to an existing customer`), the customer, the company, the similar records, the salesperson, the team, the force flag | `Create Opportunity`, `Cancel` |
| Convert to opportunities, in batch | the selected records, the salespeople, the deduplication flag, the customer handling, the team, the force flag | `Convert to Opportunities`, `Cancel` |
| Merge | the records to merge, the salesperson, the team | `Merge`, `Cancel` |
| Mark Lost | the records, the lost reason, the closing note | `Mark as Lost`, `Discard` |
| Update Probabilities | the scoring start date, the scoring variables | `Update`, `Discard` |
| Forward to Partner | the forwarding mode, the partner in single mode, the proposed pairs in automatic mode, the message body | `Send`, `Cancel` |

### 4.14 The help text of an empty Lead screen

When a Lead screen is opened and no record matches, the screen shows a help block instead of an empty list. The block is assembled as follows.

1. When the screen definition already carries its own help text, that text is shown unchanged and nothing below applies. The Leads Analysis screen reached from a mass mailing is such a case: it carries `No Leads yet!` or `No Opportunities yet!` according to the reader's groups.
2. Otherwise the title is `Create a new lead` when the screen is opened with a default type of lead, and `Create an opportunity to start playing with your pipeline.` in every other case.
3. The subtitle offers the email gateway of one Sales Team: `Use the New button, or send an email to <alias address> to test the email gateway.` where the alias address is rendered as a link that opens a new message to it.
4. The team whose alias is shown is chosen among the teams that have a non-empty alias name, whose alias creates Leads, and whose company is the company active in the session or is empty. Those teams are ordered by two keys, both descending: first the teams that qualify incoming requests as leads, then the teams the reader is a member of. The first team of that ordering wins.
5. When no team satisfies those conditions, the subtitle is empty and only the title is shown.

## 5. Reports

### 5.1 Pipeline Analysis

Source: the Lead entity itself. Default condition: type `opportunity`, records of the current period. Default grouping and measures as in section 4.10. Typical uses: revenue by stage and by month, conversion rate by source, average days to close by salesperson.

### 5.2 Leads Analysis

Source: the Lead entity. Default condition: none, with the active and archived filters both pre-selected so that lost records are visible, and the creation date filter pre-selected. Typical uses: incoming volume by source, by medium and by campaign.

### 5.3 Forecast

Source: the Lead entity, restricted to opportunities, grouped by expected closing month, measuring the prorated revenue. It answers "how much do we expect to invoice next month".

### 5.4 Activity Analysis

Source: the Activity Analysis Report database view, one row per message on a Lead carrying an activity type. Columns are listed in [entities.md](entities.md) section 11.2. Typical uses: number of calls per salesperson per month, distribution of activity types by stage.

### 5.5 Partner Assignment Analysis

Source: the Partner Assignment Analysis database view, one row per contact carrying a grade or an activation level, joined with the opportunities forwarded to it and with the customer invoice analysis. Columns: partner, grade, activation, salesperson, latest review, partnership date, country, number of opportunities, invoiced turnover, invoice accounting date. Default condition: a grade is set. Default presentation: a graph.

### 5.6 Live chat channel report

The live chat bridge adds one column, the number of Leads created from the conversations of the channel, to the report owned by the messaging domain.

## 6. Printed documents and exported files

This domain ships no printed document. The exchanges that leave the system are:

| Exchange | Direction | Content |
|---|---|---|
| Lead import template | out | A spreadsheet template listing the importable columns of the Lead, offered from the import screen under the label `Import Template for Leads & Opportunities`. |
| Forwarding email | out | The rendered forwarding template, one message per recipient partner, listing the forwarded records with their portal link and telling whether the partner already has a portal account. |
| Lead mining request | out and in | The filter payload sent to the external mining service and the company and contact data received back. |
| Enrichment request | out and in | The email domain sent to the enrichment service and the company data received back. |
| Website identification request | out and in | The grouped page-view addresses with their rule parameters, and the resolved company data. |
| Digest email | out | The two indicators of this domain, embedded in the digest owned by the employee services domain. |
| Mass mailing | out | Messages addressed to the email of the Leads selected as the mailing target; the email blacklist applies. |
| Mass text message | out | Messages addressed to the telephone number of the Leads selected; the telephone blacklist applies. |

## 7. Notifications

### 7.1 Discussion-thread messages

| Event | Subtype | Body |
|---|---|---|
| A Lead is created | Opportunity Created | `A new lead has been created for the team "<team name>".` or `A new lead has been created and is not assigned to any team.` |
| The stage changes | Stage Changed | the tracked change |
| The record becomes won | Opportunity Won | the tracked change |
| The record becomes lost | Opportunity Lost | the tracked change, plus the closing note rendered as `Lost Comment:` followed by the note when one was typed |
| The record is restored | Opportunity Restored | the tracked change |
| A meeting is created on the opportunity, outside an activity | plain note | `Meeting scheduled at <date and time>` then `Subject: <link to the meeting>` then `Duration: <duration or "unknown">` |
| Records are merged | internal note | the merge summary listing every merged-away record, its main fields, its properties and the followers carried over |
| A Lead is assigned to the team leader automatically | internal note | `This new lead created by <creation source> was automatically assigned to team leader <user name>` |
| The expected revenue is raised by a confirmed order | tracked change | `Expected revenue has been updated based on the linked Sales Orders.` |
| A Lead is created by mining | plain note | `Opportunity created by Lead Generation` followed by the company data |
| A Lead is enriched | plain note | `Lead enriched based on email address` followed by the company data |
| Enrichment found nothing | plain note | the "not found" template of the enrichment package |
| Enrichment had no usable address | plain note | the "no email" template of the enrichment package |
| A reseller accepts | plain message | `I am interested by this lead.` plus the optional comment |
| A reseller declines | plain message | `I am not interested by this lead. I contacted the lead.` or `I am not interested by this lead. I have not contacted the lead.` plus the optional comment |
| An assignment is requested manually | internal note on each team | `Lead Assignment requested by <user name>` followed by the result lines |

### 7.2 Transient notifications

| Event | Kind | Message |
|---|---|---|
| A manual assignment finishes | success | title `Leads Assigned`, body built as in [lead-assignment.md](lead-assignment.md) section 6.1 |
| An enrichment finishes | information | `The leads/opportunities have successfully been enriched` |
| An enrichment runs out of credits | warning | `Not enough credits for Lead Enrichment` |
| An enrichment fails otherwise | warning | `An error occurred during lead enrichment` |
| Partner assignment skips records with no country | danger | title `Warning`, body `There is no country set in addresses for <lead names>.` |
| The live chat command creates a record | transient chat message | `Created a new lead: <link to the record>` |
| The live chat command is used with no title | transient chat message | `Create a new lead with: /lead <lead title>` |
| An opportunity is won from the form | celebration effect | the message chosen by [calculations.md](calculations.md) section 11, with the picture of the team leader when they have one and a default picture otherwise |

### 7.3 Emails

| Event | Recipients | Template |
|---|---|---|
| A Lead is forwarded to a reseller | the reseller | Lead Forward: Send to partner |
| An external service runs out of credits | the buyers of the credit account | Lead Generation Notification |
| Any message posted on a Lead | the followers, according to their subtype subscriptions | the platform notification layout, with the expected closing date as a subtitle when set, and the team alias as the reply address |

## 8. Scheduled jobs

| Job | Default state | Frequency | Behaviour on failure |
|---|---|---|---|
| Recompute Automated Probabilities | inactive | daily | The rebuild is atomic; the recomputation commits per batch and skips a batch whose write fails, logging it. |
| Lead Assignment | inactive | daily | Both phases commit at batch boundaries; a failure late in the run keeps the earlier assignments. |
| Enrich leads | active with the package | every twenty-four hours | Each batch is locked; a batch that cannot be locked reschedules the job five minutes later. A credit exhaustion stops the whole run. |
| Generate event leads | active with the package | daily | The generation request remembers the last processed registration and resumes there. |
| Generate leads from website visits | active with the package | daily | Views that fail to resolve become `not_found` and are cleaned up after one month. |

## 9. Integration contracts with external services

Three external services are used. Each is described by what is sent and what is expected back, so that a replacement can plug a different provider behind the same contract.

### 9.1 Lead enrichment by email domain

**Sent.** A map from record identifier to the domain part of the normalized email address of that record.

**Expected back.** A map from record identifier to either nothing, or a company description holding: the company name, the external company identifier, the postal address (street, city, postal code), a telephone number, a country code, a subdivision code, and any additional descriptive data the provider offers.

**Applied.** Only onto empty fields, as stated in `LEAD-128`.

### 9.2 Lead mining

**Sent.** The number of companies wanted; the target (companies, or companies and their contacts); the list of countries, each with the selected subdivisions of that country; optionally the company size range; optionally the flattened list of industry codes; in contact mode the number of contacts per company, the filter kind (role or seniority) and the matching codes; and the list of external company identifiers already present in the database so that they are not returned again.

**Expected back.** A list of company descriptions, each with the external company identifier, the company name, the domain, an email address, a telephone number, the postal address, the country code and the subdivision code; and, in contact mode, a list of people per company, each with a name, an email address and a job position.

### 9.3 Website visitor identification

**Sent.** A map from network address to the list of matching rule identifiers, together with the parameters of each rule: the industry codes, the size range, the contact filter kind and codes, and the number of contacts to track.

**Expected back.** Per network address, either nothing, or a company description of the same shape as the mining answer, plus the number of credits consumed.

In all three cases, an answer stating that credits are exhausted must be distinguishable from an empty answer, because the two lead to different states (`LEAD-125`).

## 10. Menus, window actions and views

The menu tree, with its parents, its sequences and the groups each entry is visible to, is in
[configuration.md](configuration.md), section 9. Each menu entry opens one window action; the window
actions of this domain, and the presentations they offer, are:

| Window action | Entity | Presentations | Restriction and defaults |
|---|---|---|---|
| My pipeline | Lead | kanban, list, form, calendar, pivot, graph, activity, map | type `opportunity`; the reader's own records first; grouped by stage. |
| Leads | Lead | list, kanban, form, calendar, pivot, graph, activity | type `lead`; visible only with the group *Show Lead Menu*. |
| Opportunities (configuration entry point) | Lead | kanban, list, form | type `opportunity`. |
| Forecast | Lead | kanban, list, pivot, graph | type `opportunity`; grouped by expected closing month; measures the prorated revenue. |
| Pipeline analysis | Lead | graph, pivot, list | type `opportunity`; the current period. |
| Leads analysis | Lead | graph, pivot, list | both the active and the archived filters pre-selected. |
| Activities analysis | Activity Analysis | pivot, graph | none. |
| Partnership analysis | Partner Assignment Analysis | graph | restricted to rows whose level is set. |
| Teams | Sales Team | kanban, list, form | the dashboard presentation of the team cards. |
| Team members | Sales Team Member | list, form | reserved to the technical group. |
| Stages | Stage | list, form | reserved to the technical group. |
| Tags | Tag | list, form | none. |
| Lost reasons | Lost Reason | list, form | none. |
| Recurring plans | Recurring Plan | list, form | visible only with the group *Show Recurring Revenues Menu*. |
| Levels and Partner activations | Partner Grade, Partner Activation | list, form | present with the membership capability. |
| Lead generation requests | Lead Generation Request | list, form | present with the lead generation capability. |
| Visits to leads rules | Lead Generation Rule | list, form | present with the website identification capability. |
| Lead generation views | Reveal View | list, form | reserved to the technical group. |
| Blacklisted telephone numbers | Telephone Blacklist | list, form | reserved to the system administration group. |
| Forward to partner | Forward to Partner Wizard | form, in a dialogue | one variant defaults the composition to the batch mode. |

Every list presentation of the Lead supports editing several records at once; every kanban
presentation groups by stage by default and expands the column set by `LEAD-169`.

---

## 11. Reconciliation notes

| Subject | The two statements | Resolution |
|---|---|---|
| Operation names | One description named the operations in prose; the other gave each a stable identifier. | The stable identifiers are kept, with the sentence at the head of section 1 saying plainly that they are assigned by this specification and are not contractual strings. |
| Menus | One description put the menu tree in the interface document; the other in the configuration document. | The tree is in [configuration.md](configuration.md), section 9, and section 10 here lists the window actions the entries open. |
| The celebration animation | One description mentioned only the message; the other described the picture. | Both are here: the message is chosen by [calculations.md](calculations.md) section 11, and the animation shows the team leader's portrait when the leader has one and a generic smiling face otherwise. |
| The activity-deadline ordering | One description called it a two-pass search; the other stated it as a rule. | It is stated once as `LEAD-167` and referred to from section 4 here and from [workflows.md](workflows.md). |
| Reports | Both descriptions listed the same five reports. | Kept once, in section 5, with the measures and the default groupings of each. |
