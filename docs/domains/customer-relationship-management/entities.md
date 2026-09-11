# Entities

This file specifies every entity owned or extended by the Customer Relationship Management domain:
its purpose, its lifecycle, its complete field table, its relations, its uniqueness rules, its
defaults, its computed fields with their exact rules, its ordering, its display rule, its archival
behaviour and its multi-company behaviour.

Field tables use three columns: the field with its storage name, the type, and the meaning and
rules. Within the "meaning and rules" column the following shorthands are used consistently:

- **required** — the field must be non-empty at storage time.
- **default** — the value applied when the field is not supplied on creation.
- **computed from X** — the value is derived from X; when the field is also writable the phrase
  "writable" appears, meaning a user-supplied value survives until one of the dependencies changes.
- **stored** / **not stored** — whether the value is persisted in a column or recomputed on read.
- **tracked** — a change of value is written to the record's discussion thread as a tracked value;
  the number after "tracking order" is the display order of the tracked value in the message.
- **no copy** — the value is not carried over when the record is duplicated.
- **indexed** — a database index exists on the column; the index kind is named when it is not a
  plain one (trigram index for substring search, partial index restricted by a condition).
- **on delete** — what happens to this record when the referenced record is deleted.

---

## 1. Lead

**Lead** (`crm.lead`, table `crm_lead`).

### 1.1 Purpose

A Lead is a single commercial interest tracked from first contact to a won or lost outcome. The
same entity represents both stages of maturity, distinguished by the field `type` (type):

- `lead` — an unqualified interest. It may have nothing more than an electronic mail address. It is
  not expected to carry a Contact link, an expected revenue or a meaningful probability.
- `opportunity` — a qualified deal. It is expected to carry a Contact, a salesperson, a team, a
  stage, an expected revenue and a probability.

Whether the unqualified stage exists at all is a configuration decision: when the group *Show Lead
Menu* is not granted, records are created directly as opportunities and the separate lead menu is
hidden.

### 1.2 Lifecycle

1. **Creation.** By a salesperson in the interface, by the incoming electronic mail gateway on a
   team alias, by a public website form, by a live chat script or operator command, by an
   electronic mail client plug-in, by a text message reply, by a lead generation service request,
   or by an import.
2. **Qualification.** Fields are completed manually or by the enrichment service; the record is
   moved along the pipeline stages; the probability is recomputed at every relevant change.
3. **Allocation and assignment.** If rule-based assignment is enabled, an unassigned lead is first
   allocated to a team and then assigned to a member; assignment converts it to an opportunity.
4. **Conversion.** A wizard creates or links a Contact, sets `type` to `opportunity` and stamps
   `date_conversion` (conversion date).
5. **Selling.** Quotations are created from the opportunity; confirming one may raise the expected
   revenue.
6. **Outcome.** Either won — moved to a stage flagged as won, probability forced to one hundred,
   `date_closed` (closed date) stamped — or lost — archived, probability forced to zero, a lost
   reason recorded.
7. **Restoration.** A lost record can be restored: it is unarchived, the lost reason is cleared and
   the probability is realigned with the automatic computation.
8. **Merge or deletion.** Duplicates are merged into the most trustworthy record and the others are
   deleted. Only a manager may delete a lead outright.

### 1.3 Complete field table

#### 1.3.1 Description block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Title (`name`) | single line text | The label of the lead or opportunity. **Required.** Indexed with a trigram index so that substring search is fast. Computed from `partner_id` and writable: when the title is empty and a linked Contact with a name exists, the title becomes "*the contact name*'s opportunity". Stored. |
| Salesperson (`user_id`) | link to User | The person responsible. Default: the user performing the creation. Restricted to non-shared (internal) users. Company-checked against `company_id`. Indexed. Tracked. On delete of the user: set to empty. Setting it to empty clears `date_open`; setting it to a different user stamps `date_open` with the current instant. |
| User companies (`user_company_ids`) | many-sided link to Company | Not stored, presentation only. Equal to all companies when `company_id` is empty, otherwise to the single company of the lead. Used to restrict the selectable users in the interface. |
| Sales Team (`team_id`) | link to Sales Team | The owning team. Computed from `user_id` and `type`, writable, stored, precomputed at creation. Company-checked. Indexed. Tracked. On delete of the team: set to empty. The selectable stages are restricted to stages with no team restriction or with this team among their teams. |
| Properties (`lead_properties`) | properties | User-defined extra fields whose definition lives on the team (`team_id.lead_properties_definition`). Copied on duplication. |
| Company (`company_id`) | link to Company | The owning company. Computed from `user_id`, `team_id` and `partner_id`, writable, stored. Indexed. May legitimately be empty, which makes the record visible to every company. |
| Referred by (`referred`) | single line text | Free text naming the person or channel that referred the deal. |
| Notes (`description`) | rich text | Free internal notes. Used as the target for the live chat transcript and for the enrichment summary. |
| Active (`active`) | boolean | Default true. Tracked with tracking order 72. False means archived. A lost record is archived; an archived record is not necessarily lost (it is lost only if the probability is also zero). |
| Type (`type`) | selection | **Required.** Values: `lead` labelled "Lead", `opportunity` labelled "Opportunity". Tracked with tracking order 15. Indexed. Default: `lead` when the current user holds the group *Show Lead Menu*, otherwise `opportunity`. |

#### 1.3.2 Pipeline block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Priority (`priority`) | selection | Values `0` "Low", `1` "Medium", `2` "High", `3` "Very High". Default `0`. Indexed. It is the first sort key of the default ordering, descending. |
| Stage (`stage_id`) | link to Stage | The pipeline column. Computed from `team_id` and `type`, writable, stored, **not copied**. Indexed. Tracked. On delete of the stage: **restricted** — a stage that still holds leads cannot be deleted. Group expansion: when the pipeline is grouped by stage, empty columns are added for stages that are not folded and that are either unrestricted or attached to the relevant team. |
| Stage colour (`stage_id_color`) | integer | Related to `stage_id.color`; presentation only. |
| Tags (`tag_ids`) | many-sided link to Tag | Free classification labels. Stored in the association table `crm_tag_rel` with columns `lead_id` and `tag_id`. |
| Colour index (`color`) | integer | Default 0. Presentation only. |
| Rotting (`is_rotting`) | boolean | Not stored. True when the record has not been updated for longer than the stage's rotting threshold, and additionally only for records whose outcome is pending and whose type is `opportunity`. |
| Days rotting (`rotting_days`) | integer | Not stored. Number of days since the record was last updated. |
| Stage duration tracking (`duration_tracking`) | structured value | Not stored. Maps each stage identifier the record has occupied to the number of seconds spent in it. Maintained by the tracking machinery on every change of `stage_id`. |

#### 1.3.3 Revenue block

All monetary fields in this block are expressed in `company_currency` (company currency).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Expected revenue (`expected_revenue`) | monetary | The untaxed one-off amount the salesperson expects. Default 0. Tracked. Freely editable. |
| Prorated revenue (`prorated_revenue`) | monetary | Computed and stored from `expected_revenue` and `probability`; read-only. |
| Recurring revenue (`recurring_revenue`) | monetary | The untaxed amount expected over the whole recurring plan. Default 0. Tracked. Visible only when the group *Show Recurring Revenues Menu* is granted. |
| Recurring plan (`recurring_plan`) | link to Recurring Plan | The duration in months over which the recurring revenue is spread. |
| Expected monthly recurring revenue (`recurring_revenue_monthly`) | monetary | Computed and stored from `recurring_revenue` and `recurring_plan.number_of_months`; read-only. |
| Prorated monthly recurring revenue (`recurring_revenue_monthly_prorated`) | monetary | Computed and stored from `recurring_revenue_monthly` and `probability`; read-only. |
| Prorated recurring revenue (`recurring_revenue_prorated`) | monetary | Computed and stored from `recurring_revenue` and `probability`; read-only. |
| Currency (`company_currency`) | link to Currency | Not stored. The currency of `company_id`; when the lead has no company, the currency of the current user's company. Computed with elevated rights so that it is always readable. When a monetary field is aggregated in a grouped list the currency is resolved in the query itself: for each row, the currency of the row's company when the company is set, otherwise the current user's company currency. |

#### 1.3.4 Date block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Closed date (`date_closed`) | date and time | Read-only, **not copied**. Stamped when the record becomes won or lost; cleared when it returns to an open state. See the write rules in `business-rules.md`. |
| Last action (`date_automation_last`) | date and time | Read-only. Reserved for automation rules to record when they last acted on the record. |
| Assignment date (`date_open`) | date and time | Computed from `user_id`, stored, read-only, **not copied**. Set to the current instant the first time a salesperson is set; cleared when the salesperson is removed; re-stamped when the salesperson changes to a different user. |
| Days to assign (`day_open`) | decimal | Computed and stored from `create_date` and `date_open`. Empty when either is missing. |
| Days to close (`day_close`) | decimal | Computed and stored from `create_date` and `date_closed`. Empty when either is missing. |
| Last stage update (`date_last_stage_update`) | date and time | Computed from `stage_id`, stored, read-only, indexed, **not copied**. Set to the current instant on creation and re-stamped on every actual change of stage. |
| Conversion date (`date_conversion`) | date and time | Read-only. Stamped by the conversion routine. |
| Expected closing (`date_deadline`) | date | The date on which the opportunity is expected to be won. Free. Appears as a subtitle in notification electronic mail. |

#### 1.3.5 Customer and contact block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Customer company (`commercial_partner_id`) | link to Contact | **Not stored**; an interface convenience only, explicitly not to be relied on for business logic. Restricted to Contacts flagged as companies. Computed from `partner_id` and `partner_name`, writable. |
| Contact (`partner_id`) | link to Contact | The linked customer record. Optional. Company-checked. Indexed. Tracked with tracking order 10. On delete: set to empty. Usually created at conversion. |
| Contact is blacklisted (`partner_is_blacklisted`) | boolean | Related to `partner_id.is_blacklisted`; read-only. |
| Contact name (`contact_name`) | single line text | The name of the individual. Trigram-indexed. Tracked with tracking order 30. Computed from `partner_id`, writable, stored. |
| Company name (`partner_name`) | single line text | The name of the organisation that will be created as a Contact when the lead is converted. Trigram-indexed. Tracked with tracking order 20. Computed from `partner_id`, writable, stored. |
| Job position (`function`) | single line text | Computed from `partner_id`, writable, stored. |
| Electronic mail address (`email_from`) | single line text | Trigram-indexed. Tracked with tracking order 40. Computed from `partner_id.email`, writable, stored, with an inverse that writes back to the Contact. |
| Normalised electronic mail address (`email_normalized`) | single line text | Trigram-indexed. Maintained by the blacklist machinery from `email_from`: the address stripped of its display name and lower-cased. |
| Electronic mail domain criterion (`email_domain_criterion`) | single line text | Computed and stored from `email_normalized`, read-only, indexed with a partial index that skips empty values. Holds the organisational domain of the address, and is empty for addresses whose domain is a free public provider. Used for duplicate detection by company. |
| Telephone (`phone`) | single line text | Tracked with tracking order 50. Computed from `partner_id.phone`, writable, stored, with an inverse that writes back to the Contact. Reformatted to international notation when the telephone, the country or the company changes in the form. |
| Sanitised telephone (`phone_sanitized`) | single line text | Maintained by the telephone mixin: the number in international notation with no separators, or empty when it cannot be parsed. Indexed with a partial index that skips empty values. |
| Telephone quality (`phone_state`) | selection | Computed and stored from `phone` and `country_id.code`. Values `correct`, `incorrect`, or empty when there is no telephone number. |
| Electronic mail quality (`email_state`) | selection | Computed and stored from `email_from`. Values `correct`, `incorrect`, or empty when there is no address. |
| Website (`website`) | single line text | Computed from `partner_id`, writable, stored. Cleaned on write: a bare domain is completed into a full address. |
| Language (`lang_id`) | link to Language | Computed from `partner_id`, writable, stored. |
| Language code (`lang_code`) | single line text | Related to `lang_id.code`. |
| Active language count (`lang_active_count`) | integer | Not stored. The number of installed languages; used to decide whether to show the language selector. |

#### 1.3.6 Address block

The six address fields are always synchronised **as a block**: either all six are taken from the
Contact or none is. See `calculations.md` for the exact rule.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Street (`street`) | single line text | Computed from `partner_id`, writable, stored. |
| Street line two (`street2`) | single line text | Computed from `partner_id`, writable, stored. |
| Postal code (`zip`) | single line text | Computed from `partner_id`, writable, stored. Flagged as a default-changing field, meaning a user may store per-value defaults keyed on it. |
| City (`city`) | single line text | Computed from `partner_id`, writable, stored. |
| State (`state_id`) | link to Country State | Computed from `partner_id`, writable, stored. Restricted to states of `country_id` when a country is set. |
| Country (`country_id`) | link to Country | Computed from `partner_id`, writable, stored. |

#### 1.3.7 Probability block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Probability (`probability`) | decimal | The percentage chance of winning, between 0 and 100 inclusive (enforced by a stored check). Aggregated as an average in grouped views. **Not copied.** Computed, writable, stored: the computation aligns it with `automated_probability` only while the two were already equal. |
| Automated probability (`automated_probability`) | decimal | Read-only, stored. The output of the naive Bayesian computation. |
| Is automated probability (`is_automated_probability`) | boolean | Not stored. True when `probability` and `automated_probability` are equal when compared at two decimals. |

#### 1.3.8 Outcome block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Won or lost (`won_status`) | selection | Computed and stored from `active`, `probability` and `stage_id`. Values `won` "Won", `lost` "Lost", `pending` "Pending". Tracked with tracking order 70. **Not copied.** |
| Lost reason (`lost_reason_id`) | link to Lost Reason | Indexed. Tracked with tracking order 71. On delete: **restricted**. Cleared automatically when the record is unarchived. |

#### 1.3.9 Statistics and interface helper block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Meetings (`calendar_event_ids`) | reverse link to Meeting | The meetings scheduled against this opportunity. |
| Potential duplicates (`duplicate_lead_ids`) | many-sided link to Lead | Not stored, computed with elevated rights and with archived records included. See the detection algorithm in `calculations.md`. The set always contains the record itself. |
| Potential duplicate count (`duplicate_lead_count`) | integer | Not stored. The number of duplicates found, **excluding** the record itself. |
| Meeting display date (`meeting_display_date`) | date | Not stored. The start of the next future meeting if any, otherwise the start of the last past meeting. |
| Meeting display label (`meeting_display_label`) | single line text | Not stored. One of "No Meeting", "Next Meeting", "Last Meeting". |
| Contact electronic mail will update (`partner_email_update`) | boolean | Not stored. True when saving would overwrite the Contact's address with the lead's. |
| Contact telephone will update (`partner_phone_update`) | boolean | Not stored. True when saving would overwrite the Contact's telephone with the lead's. |
| Is contact visible (`is_partner_visible`) | boolean | Not stored, depends on the acting user. True when the type is `opportunity`, or a Contact is linked, or the user is in developer mode. Controls whether the customer field is shown on the lead form. |
| My activity deadline (`my_activity_date_deadline`) | date | Not stored. The earliest deadline among the current user's activities on the record. Sorting on it is supported through a two-pass search described in `interfaces.md`. |

#### 1.3.10 Campaign attribution block

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Campaign (`campaign_id`) | link to Campaign | Indexed. On delete: set to empty (overriding the generic behaviour). |
| Medium (`medium_id`) | link to Medium | Indexed. On delete: set to empty. |
| Source (`source_id`) | link to Source | Indexed. On delete: set to empty. |

#### 1.3.11 Fields contributed by companion capabilities

| Field (storage name) | Type | Contributed by | Meaning and rules |
|---|---|---|---|
| Orders (`order_ids`) | reverse link to Sales Order | Sales coupling | Quotations and confirmed orders whose opportunity reference points here. |
| Sum of orders (`sale_amount_total`) | monetary | Sales coupling | Not stored. The untaxed total of the **confirmed** orders, each converted into the company currency at the order date. |
| Number of quotations (`quotation_count`) | integer | Sales coupling | Not stored. The count of orders in state draft or sent. |
| Number of sales orders (`sale_order_count`) | integer | Sales coupling | Not stored. The count of orders **not** in state draft, sent or cancelled. |
| Web visitors (`visitor_ids`) | many-sided link to Website Visitor | Website form | The anonymous or identified visitors associated with the lead. Stored in association table `crm_lead_website_visitor_rel`. |
| Page views (`visitor_page_count`) | integer | Website form | Not stored. The total number of tracked page views of all associated visitors. |
| Sessions (`visitor_sessions_count`) | integer | Website form | Not stored. The number of visit sessions of all associated visitors. |
| Originating live chat channel (`origin_channel_id`) | link to Discussion Channel | Live chat | Read-only, indexed with a partial index that skips empty values. Set when the lead is produced by a chat script step or by the operator command. |
| Enrichment done (`iap_enrich_done`) | boolean | Enrichment | True once the enrichment service has been called for this lead, whether or not it returned data. |
| Allow manual enrichment (`show_enrich_button`) | boolean | Enrichment | Not stored. True when the lead is active, has an electronic mail address, is not yet enriched, and the enrichment setting is on demand. |
| Lead generation request (`lead_mining_request_id`) | link to Lead Generation Request | Lead generation | Indexed. The request that produced this lead. On delete: set to empty. |
| Assigned partner (`partner_assigned_id`) | link to Contact | Partner network | Tracked. Indexed. The reselling partner the opportunity has been forwarded to. On delete: set to empty. |
| Partner assignment date (`date_partner_assign`) | date | Partner network | The date the opportunity was last forwarded to a partner. |
| Partners not interested (`partner_declined_ids`) | many-sided link to Contact | Partner network | Partners that explicitly declined the opportunity. |
| Geographic latitude (`partner_latitude`) | decimal | Partner network | The latitude used for the nearest-partner search. |
| Geographic longitude (`partner_longitude`) | decimal | Partner network | The longitude used for the nearest-partner search. |
| Source event (`event_id`) | link to Event | Events coupling | The event whose registration rule produced this lead. |
| Registration rule (`event_lead_rule_id`) | link to Event Lead Rule | Events coupling | The rule that produced this lead. |
| Source registrations (`registration_ids`) | many-sided link to Event Registration | Events coupling | The registrations that triggered the rule. |
| Registration count (`registration_count`) | integer | Events coupling | Not stored. The number of those registrations. |
| Originating survey (`origin_survey_id`) | link to Survey | Survey coupling | The survey whose completion produced this lead. |

#### 1.3.12 Fields contributed by the discussion, activity and blacklist machinery

These fields exist on every record that carries a discussion thread. They are listed for
completeness; their semantics belong to the Messaging domain.

`message_ids`, `message_follower_ids`, `message_partner_ids`, `message_is_follower`,
`message_needaction`, `message_needaction_counter`, `message_has_error`,
`message_has_error_counter`, `message_has_sms_error`, `message_attachment_count`, `has_message`,
`website_message_ids`, `message_bounce`, `activity_ids`, `activity_state`, `activity_user_id`,
`activity_type_id`, `activity_type_icon`, `activity_date_deadline`, `activity_summary`,
`activity_exception_decoration`, `activity_exception_icon`, `activity_calendar_event_id`,
`is_blacklisted`, `phone_sanitized_blacklisted`, `phone_blacklisted`, `phone_mobile_search`,
`email_cc`, `rating_ids`.

The primary electronic mail field of the thread is declared to be `email_from`; this is what makes
the blacklist and the suggested-recipient machinery act on it.

### 1.4 Computed fields in full

#### 1.4.1 Sales Team (`team_id`)

Depends on `user_id` and `type`. For each record:

1. If there is no salesperson, leave the team unchanged (clearing the salesperson must not clear
   the team).
2. If a team is already set **and** the salesperson is one of its members or is its leader, leave
   it unchanged.
3. Otherwise compute a candidate domain: `[('use_leads', '=', True)]` when the type is `lead`,
   `[('use_opportunities', '=', True)]` when the type is `opportunity`.
4. Ask the team-selection routine (section 8.2) for the default team of that salesperson under that
   domain and store it if it differs from the current one.

#### 1.4.2 Company (`company_id`)

Depends on `user_id`, `team_id` and `partner_id`. For each record:

1. Start from the current company as a *proposal*.
2. Invalidate the proposal (set it to empty) if any of the following holds:
   - a salesperson is set and the proposal is not among that user's allowed companies;
   - the team has a company and the proposal differs from it;
   - a team is set, the team has no company, and no salesperson is set;
   - neither a team nor a salesperson is set, and either there is no Contact or the Contact's
     company differs from the proposal.
3. If the proposal is now empty, propose a new one in this order:
   - the team's company, if the team has one;
   - otherwise, if a salesperson is set: the current acting company when that company is among the
     salesperson's allowed companies, otherwise the intersection of the salesperson's own company
     with the set of companies currently enabled for the acting user;
   - otherwise, if a Contact is set: the Contact's company;
   - otherwise empty.

#### 1.4.3 Stage (`stage_id`)

Depends on `team_id` and `type`. For each record: if there is no stage, **or** the record has a
team and the current stage is team-restricted and the record's team is not among the stage's teams,
then find the first stage that is not folded using the stage-search routine (section 8.1).

#### 1.4.4 Title (`name`)

Depends on `partner_id`. If the title is empty and a Contact with a name is linked, set the title
to "*the contact name*'s opportunity".

#### 1.4.5 Customer company (`commercial_partner_id`)

Depends on `partner_id` and `partner_name`. Two passes:

1. For records that have a Contact: take the Contact's commercial entity; keep it only if it is
   flagged as a company **and** it is not the Contact itself; otherwise leave empty.
2. For the remaining records that have a company name: group them by that name, look up Contacts
   flagged as companies whose name is one of those names, and attach the **first** matching Contact
   identifier to every record carrying that name.

#### 1.4.6 Contact name (`contact_name`)

Depends on `partner_id`. Records without a Contact get an empty contact name. For the others, the
contact name becomes the Contact's name when the Contact is **not** flagged as a company, and
otherwise keeps the value already present on the lead.

#### 1.4.7 Company name (`partner_name`)

Depends on `partner_id`. Records without a Contact get an empty company name. For the others, the
candidate is, in order: the name of the Contact's parent; failing that, the Contact's own name when
the Contact is flagged as a company; failing that, the Contact's free-text company name. If the
candidate is empty the value already present on the lead is kept.

#### 1.4.8 Job position (`function`) and Website (`website`)

Both depend on `partner_id` and follow the same shape: if the lead's own value is empty **or** the
Contact has a value, take the Contact's value. In other words a non-empty Contact value always
wins, and an empty Contact value only wins when the lead was empty too.

#### 1.4.9 Language (`lang_id`)

Depends on `partner_id`. For every record that has a Contact, the language is forced to the
language matching the Contact's language code, **erasing** any previous value — including erasing
it to empty when the Contact has no language.

#### 1.4.10 Address block (`street`, `street2`, `zip`, `city`, `state_id`, `country_id`)

Depends on `partner_id`. All six are replaced at once by the block rule of section 8.3.

#### 1.4.11 Electronic mail address (`email_from`) and its inverse

Depends on `partner_id.email`. If the Contact has an address **and** the update test of section 8.4
passes with empty values forced, copy the Contact's address onto the lead.

The inverse direction runs when the field is written directly: if the update test passes **without**
forcing empty values (that is, a lead with an empty address never overwrites a Contact that has
one), write the lead's address onto the Contact.

#### 1.4.12 Telephone (`phone`) and its inverse

Identical in shape to the electronic mail address, using the telephone update test of section 8.5.

#### 1.4.13 Electronic mail domain criterion (`email_domain_criterion`)

Depends on `email_normalized`. Empty when there is no normalised address. Otherwise the result of
the domain-preparation routine, which returns the domain part of the address unless that domain is
a well-known free public provider, in which case it returns nothing.

#### 1.4.14 Telephone quality (`phone_state`)

Depends on `phone` and `country_id.code`. Empty when there is no telephone. Otherwise the number is
parsed using the lead's country code when it has one: parsing success yields `correct`, a parsing
failure yields `incorrect`.

#### 1.4.15 Electronic mail quality (`email_state`)

Depends on `email_from`. Empty when there is no address. Otherwise the value starts at `incorrect`
and becomes `correct` as soon as **any** of the addresses extracted from the field passes format
validation.

#### 1.4.16 Won or lost (`won_status`)

Depends on `active`, `probability` and `stage_id`:

- `won` when the probability is exactly 100 **and** the stage is flagged as a won stage;
- `lost` when the record is archived **and** the probability is exactly 0;
- `pending` otherwise.

#### 1.4.17 Is automated probability (`is_automated_probability`)

True when `probability` and `automated_probability` compare equal at two decimals.

#### 1.4.18 Meeting display (`meeting_display_date`, `meeting_display_label`)

One grouped read over meetings whose opportunity is one of the records collects, per record, the
list of start instants and the maximum start instant. Then, per record:

- no meetings at all: date empty, label "No Meeting";
- at least one start instant strictly after the current instant: date is the **smallest** such
  instant expressed in the reader's time zone, label "Next Meeting";
- otherwise: date is the **largest** start instant expressed in the reader's time zone, label
  "Last Meeting".

#### 1.4.19 Is contact visible (`is_partner_visible`)

True when the type is `opportunity`, or a Contact is set, or the acting user holds the technical
group that enables developer mode. The purpose is to hide a misleading empty customer field on
unqualified leads while still showing it whenever a Contact exists, because writing the lead's
electronic mail address or telephone then also writes the Contact's.

### 1.5 Uniqueness, constraints and indexes

| Kind | Definition | Effect |
|---|---|---|
| Stored check | `probability` between 0 and 100 inclusive | Rejects out-of-range probabilities with the message "The probability of closing the deal should be between 0% and 100%!" |
| Programmed constraint | on `probability` and `stage_id` | A record whose stage is flagged won must have probability exactly 100; otherwise "A lead in a Won stage cannot be lost. Move it to another stage first." |
| Programmed constraint | inside the outcome bookkeeping | A record may not be simultaneously won and lost; otherwise "The lead *the record* cannot be won and lost at the same time." |
| Index | on the triple (`user_id`, `team_id`, `type`) | Speeds the pipeline and dashboard queries. |
| Index | on the pair (`create_date`, `team_id`) | Speeds the allocation query. |
| Partial index | on (`priority` descending, identifier descending) restricted to active records | Matches the default ordering. |

There is **no** uniqueness rule on a Lead. Two leads may legitimately carry the same electronic mail
address, the same telephone number and the same Contact; that is exactly what duplicate detection
and merging exist to resolve.

### 1.6 Ordering and display

- **Default ordering**: priority descending, then identifier descending. In effect, the highest
  priority first and, within one priority, the most recently created first.
- **Display name**: the value of `name`.

### 1.7 Archival behaviour

- Archiving (setting `active` to false) is *not* by itself a loss. A record can be archived for any
  reason; it is only "lost" when the probability is also zero.
- Unarchiving clears the lost reason on every record that was actually inactive, and forces a
  recomputation of the probability.
- The **restore** operation goes one step further: it unarchives and then assigns the automated
  probability to the manual probability, which re-attaches the record to the automatic computation.
- Archived records are excluded from ordinary searches. Duplicate detection deliberately includes
  them; so does the lost-reason navigation action.

### 1.8 Multi-company behaviour

- `company_id` may be empty, in which case the record is visible from every company.
- A record rule restricts visibility to records whose company is among the reader's enabled
  companies or is empty.
- The company of the record is cross-checked against the companies of `user_id`, `team_id` and
  `partner_id`; a mismatch is rejected by the generic company-consistency check.
- Copying a record does not change the company.

### 1.9 Copy behaviour

When a Lead is duplicated:

- `stage_id`, `date_closed`, `date_open`, `probability`, `automated_probability`, `won_status`,
  `duplicate_lead_ids`, `duplicate_lead_count`, `email_domain_criterion`, `email_normalized`,
  `phone_sanitized`, `date_last_stage_update` and the whole discussion thread are **not** copied.
- `lead_properties` **is** copied.
- If the acting user does not hold the group *Show Recurring Revenues Menu*, the recurring revenue
  is forced to zero and the recurring plan is cleared on the copy.
- The type and the team of the source are carried over explicitly.
- The assignment date of the copy is set to the current instant when the source is an opportunity
  **and** its salesperson is an active user; otherwise it is cleared.
- If the source's salesperson is not an active user, the copy gets no salesperson.

### 1.10 Deletion behaviour

Deleting a Lead first detaches every Meeting that referenced it as a generic document (the
reference model and reference identifier are cleared) so that no meeting points at a record that no
longer exists. Only then is the record removed.

Deletion is restricted by access rights: salespeople may create and modify leads but may not delete
them; only the sales administrator group may.
