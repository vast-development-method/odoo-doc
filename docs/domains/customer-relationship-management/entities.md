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
| Electronic mail domain criterion (`email_domain_criterion`) | single line text | Computed and stored from `email_normalized`, read-only, indexed with a partial index that skips empty values. Holds an at sign followed by the organisational domain of the address; for an address whose domain is a well-known free public provider, and for a value that contains no at sign at all, it holds the whole normalised address instead. Used for duplicate detection by company. See section 1.4.13. |
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
| Allow manual enrichment (`show_enrich_button`) | boolean | Enrichment | Not stored; depends on `email_from`, `probability`, `iap_enrich_done` and `reveal_id`. False when the record is archived, **or** has no electronic mail address, **or** its electronic mail quality is `incorrect`, **or** it has already been enriched, **or** it carries a service company identifier (it came from the identification service), **or** its probability is exactly one hundred. True otherwise. The manual enrichment control on the form is shown when this flag is true; the enrichment mode setting does not take part in the flag. |
| Lead generation request (`lead_mining_request_id`) | link to Lead Generation Request | Lead generation | Indexed. The request that produced this lead. On delete: set to empty. |
| Assigned partner (`partner_assigned_id`) | link to Contact | Partner network | Tracked. Indexed. The reselling partner the opportunity has been forwarded to. On delete: set to empty. |
| Partner assignment date (`date_partner_assign`) | date | Partner network | Computed from `partner_assigned_id`, writable, stored, **copied** on duplication. Set to today's date in the reader's time zone whenever an assigned partner is present, and cleared when the assigned partner is cleared. |
| Partners not interested (`partner_declined_ids`) | many-sided link to Contact | Partner network | Partners that explicitly declined the opportunity. |
| Geographic latitude (`partner_latitude`) | decimal | Partner network | The latitude used for the nearest-partner search. |
| Geographic longitude (`partner_longitude`) | decimal | Partner network | The longitude used for the nearest-partner search. |
| Source event (`event_id`) | link to Event | Events coupling | The event whose registration rule produced this lead. |
| Registration rule (`event_lead_rule_id`) | link to Event Lead Rule | Events coupling | The rule that produced this lead. |
| Source registrations (`registration_ids`) | many-sided link to Event Registration | Events coupling | The registrations that triggered the rule. |
| Registration count (`registration_count`) | integer | Events coupling | Not stored. The number of those registrations. |
| Originating survey (`origin_survey_id`) | link to Survey | Survey coupling | The survey whose completion produced this lead. |
| Service company identifier (`reveal_id`) | single line text | Lead generation and website identification | Indexed with a partial index that skips empty values. The identifier the company data service assigned to the company. It is used to avoid buying the same company twice and it suppresses the manual enrichment control. |
| Originating network address (`reveal_ip`) | single line text | Website identification | The address of the visit that produced the lead. |
| Identification credits consumed (`reveal_iap_credits`) | integer | Website identification | The number of service credits the identification consumed for this lead. Available as a measure in the pivot and graph presentations. |
| Lead generation rule (`reveal_rule_id`) | link to Lead Generation Rule | Website identification | Indexed with a partial index that skips empty values. The rule that produced this lead. |

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

Depends on `email_normalized`. The rule is the domain-preparation routine:

1. When `email_normalized` is empty, the criterion is empty.
2. Otherwise take the normalised address; when the value cannot be normalised, take the raw address
   folded to lower case.
3. If that text contains no at sign, the criterion is the whole text.
4. Otherwise the domain is the part after the **last** at sign. When that domain is **not** one of
   the well-known free public provider domains, the criterion is an at sign followed by the domain,
   for example `@northwind-parts.example`.
5. When the domain **is** a free public provider domain, the criterion is the whole address, so that
   two different individuals at the same public provider are never grouped, while two Leads carrying
   the *same* public address still are.

Worked examples: `robert.poilvert@mycompany.example` yields `@mycompany.example`;
`accounting@mycompany.example` yields `@mycompany.example` as well, so the two Leads are potential
duplicates of each other; `robert.poilvert@gmail.example` yields `robert.poilvert@gmail.example` in
full; a value with no at sign, such as `not-an-address`, yields `not-an-address`.

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

### 1.11 Behaviour of the form as the user types

These reactions happen in the form before anything is saved.

| Trigger | Effect |
|---|---|
| The user sets `commercial_partner_id` (customer company) while a Contact is already linked and the chosen company is **not** the commercial entity of that Contact | `partner_id`, `email_from` and `phone` are cleared, and the chosen company is written back into `commercial_partner_id`, so that the user can pick a fresh contact under that company. |
| The user sets `commercial_partner_id` while `name` is still empty | `name` becomes "*the company name*'s opportunity". |
| The user edits `phone`, `country_id` or `company_id` | `phone` is rewritten in international notation when it can be parsed with the record's country; when it cannot be parsed it is left exactly as typed. |
| The user types a recurring revenue different from zero | `recurring_plan` becomes required and the record cannot be saved until a plan is chosen. |

### 1.12 Life cycle summary of a write

| Step | What happens |
|---|---|
| Create | The website text is cleaned. The derived fields are computed in this order: team, company, stage, the contact block taken from the Contact, then the probability. When the record lands directly in a stage flagged as won and carries no closed date, the closed date is stamped. When the calling context imposed no Contact and only a customer company was supplied, the rule of section 1.12.1 decides whether that company becomes the Contact or only fills the company name. The outcome bookkeeping then runs with an empty previous state, so a record created won or created lost adjusts the frequency table immediately. A creation message is posted. |
| Write | The date rules of `state-machines.md` section 9 apply, then the won-stage forcing, then the outcome bookkeeping and the frequency adjustments. |
| Duplicate | See section 1.9. |
| Archive | Only sets `active` to false. Combined with a probability of zero this means lost. |
| Unarchive | Clears `lost_reason_id` and recomputes `automated_probability` for every record that was actually inactive. |
| Delete | Every Meeting that points at the record through the generic document reference is detached first. |

#### 1.12.1 Creating with a customer company but no Contact

When a Lead is created with a value in `commercial_partner_id` and no `partner_id`, and the calling
context did not impose a default Contact:

1. If the lead has neither a telephone nor an electronic mail address, **or** both of them match the
   company's telephone and address, the company itself becomes `partner_id`.
2. Otherwise only `partner_name` is filled from the company's name and the record stays without a
   Contact.

---

## 2. Stage

**Stage** (`crm.stage`, table `crm_stage`).

### 2.1 Purpose

A Stage is one column of the pipeline. It expresses how far a deal has progressed. Stages are
ordered by a sequence number; the further along the sequence, the closer to a decision. A stage
may be flagged as a won stage, which makes every record in it a won deal.

A stage may be global (available to every team) or restricted to a set of teams.

### 2.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Stage name (`name`) | single line text | **Required.** Translatable. |
| Sequence (`sequence`) | integer | Default 1. Lower means earlier in the pipeline. |
| Is won stage (`is_won`) | boolean | Default false. When true, every lead in this stage is treated as won and must carry probability 100. |
| Days to rot (`rotting_threshold_days`) | integer | Default 0. A record that has not been updated for more than this many days is highlighted as rotting. Zero disables the feature. Changing the value does not retroactively alter the rotting state of records last updated before the change. |
| Requirements (`requirements`) | long text | Internal guidance shown as a tooltip on the stage title, for example "Offer sent to customer". |
| Sales Teams (`team_ids`) | many-sided link to Sales Team | Empty means the stage is available to every team. On delete of a team: **restricted**. |
| Folded in pipeline (`fold`) | boolean | When true the column is collapsed in the pipeline view when it holds no record. |
| Team count (`team_count`) | integer | Not stored, interface only. The total number of teams in the database (used to decide whether to display the team restriction widget). |
| Colour (`color`) | integer | Presentation only. |

### 2.3 Ordering and display

- **Default ordering**: sequence ascending, then name, then identifier.
- **Display name**: the value of `name`.

### 2.4 Side effects of writing

Writing the won flag has a cascading effect, because a lead in a won stage must have probability
100:

1. Perform the write.
2. If the won flag was part of the write, collect **all** leads whose stage is one of the written
   stages.
3. If the flag was set to true, force those leads to probability 100 and automated probability 100.
4. If the flag was set to false, recompute the probability of those leads from the automatic
   computation.

Because this can touch a large number of records, the interface warns the user before saving with
the title "Do you really want to update this stage?" and the message "Changing the value of 'Is Won
Stage' may induce a large number of operations, as the probabilities of opportunities in this stage
will be recomputed on saving." The warning is suppressed when the stage is being created.

### 2.5 Archival, multi-company, deletion

A Stage has no active flag and no company: stages are global to the database and are shared across
companies. A stage that is referenced by at least one lead cannot be deleted, because the reference
from the lead is restrictive.

---

## 3. Tag

**Tag** (`crm.tag`, table `crm_tag`).

### 3.1 Purpose

A free classification label attached to leads and opportunities (and, where the sales capability is
present, to sales documents as well). Tags are used for reporting and, importantly, as a predictive
variable: a tag can raise or lower the computed probability.

### 3.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Tag name (`name`) | single line text | **Required.** Translatable. **Unique** across the database. |
| Colour (`color`) | integer | Default: a pseudo-random integer between 1 and 11 inclusive, drawn at creation. |

### 3.3 Uniqueness

A stored uniqueness rule on the name rejects a duplicate with the message "Tag name already
exists!".

### 3.4 Ordering and display

- **Default ordering**: identifier.
- **Display name**: the value of `name`.

---

## 4. Lost Reason

**Lost Reason** (`crm.lost.reason`, table `crm_lost_reason`).

### 4.1 Purpose

A reusable explanation of why a deal was lost. Recording it turns the loss into a measurable fact.

### 4.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Description (`name`) | single line text | **Required.** Translatable. |
| Active (`active`) | boolean | Default true. Archiving a reason keeps it on the historical records but removes it from the selection list. |
| Leads count (`leads_count`) | integer | Not stored. The number of leads carrying this reason, counted **including archived leads** — which is essential, since a lost lead is archived. |

### 4.3 Default records

Three reasons are created on installation and are not overwritten afterwards: "Too expensive", "We
don't have people/skills" and "Not enough stock".

### 4.4 Ordering, display, deletion

- **Default ordering**: identifier.
- **Display name**: the value of `name`.
- A reason that is referenced by a lead cannot be deleted, because the reference is restrictive.

---

## 5. Recurring Plan

**Recurring Plan** (`crm.recurring.plan`, table `crm_recurring_plan`).

### 5.1 Purpose

A named duration expressed in months. It is the divisor that turns a recurring revenue quoted over
a whole term into a monthly figure.

### 5.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Plan name (`name`) | single line text | **Required.** Translatable. |
| Number of months (`number_of_months`) | integer | **Required.** Must be greater than or equal to zero (stored check). A plan with zero months behaves, in the monthly conversion, as a plan of one month — see `calculations.md`. |
| Active (`active`) | boolean | Default true. |
| Sequence (`sequence`) | integer | Default 10. Determines the order of the selection list. |

### 5.3 Constraint

The stored check on the number of months rejects a negative value with the message "The number of
month can't be negative."

### 5.4 Default records

Four plans are created on installation and are not overwritten afterwards:

| Name | Months |
|---|---|
| Monthly | 1 |
| Yearly | 12 |
| Over 3 years | 36 |
| Over 5 years | 60 |

### 5.5 Ordering and display

- **Default ordering**: sequence.
- **Display name**: the value of `name`.

---

## 6. Scoring Frequency

**Scoring Frequency** (`crm.lead.scoring.frequency`, table `crm_lead_scoring_frequency`).

### 6.1 Purpose

One cell of the frequency table that drives the predictive probability. A cell records, for one
Sales Team (or for "no team"), one variable and one value of that variable, how many won deals and
how many lost deals carried that value.

Counts are stored as decimals, not integers, because the table is deliberately seeded with a
fractional increment of one tenth to avoid zero frequencies (see `calculations.md`, section on the
zero-frequency correction).

### 6.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Variable (`variable`) | single line text | The storage name of the lead field the cell is about, for example `stage_id`, `country_id`, `email_state`, or the special value `tag_id` used for individual tags. Indexed. |
| Value (`value`) | single line text | The value of that variable, **always stored as text**: a numeric identifier is stored as its decimal representation, a selection value as its technical value, a boolean absence as the text `False`. |
| Won count (`won_count`) | decimal with one decimal place | The number of won deals carrying this variable and value. |
| Lost count (`lost_count`) | decimal with one decimal place | The number of lost deals carrying this variable and value. |
| Sales Team (`team_id`) | link to Sales Team | The team the cell belongs to. Empty means the cell is the aggregate "no team" bucket. On delete of the team: **cascade** — but see the team deletion routine in section 9.5, which first folds the counts into the "no team" bucket. |

### 6.3 Uniqueness

There is no stored uniqueness rule on the triple (team, variable, value). Uniqueness is maintained
procedurally by the update routine, which looks up an existing cell before creating one.

### 6.4 Ordering and display

- **Default ordering**: identifier. The probability computation explicitly reads the table ordered
  by team ascending, then identifier, so that the per-team grouping is contiguous.
- **Display name**: the identifier.

---

## 7. Scoring Frequency Field

**Scoring Frequency Field** (`crm.lead.scoring.frequency.field`, table
`crm_lead_scoring_frequency_field`).

### 7.1 Purpose

The catalogue of Lead fields that an administrator may select as predictive variables. Selecting a
field here does not by itself activate it; activation happens when the field's storage name is
written into the configuration parameter listing the scoring variables.

### 7.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Field (`field_id`) | link to Model Field | **Required.** Restricted to fields of the Lead model. On delete: **cascade**. |
| Field label (`name`) | single line text | Related to the field's description; read-only; translatable. |
| Colour (`color`) | integer | Default: a pseudo-random integer between 1 and 11 inclusive. |

### 7.3 Default records

Seven catalogue entries are created on installation, for the Lead fields `state_id` (state),
`country_id` (country), `phone_state` (telephone quality), `email_state` (electronic mail quality),
`source_id` (source), `lang_id` (language) and `tag_ids` (tags).

### 7.4 Ordering and display

- **Default ordering**: identifier.
- **Display name**: the field label.

---

## 8. Shared routines referenced by the Lead computations

These routines are used by more than one computed field and by the wizards. They are specified once
here and referred to by number elsewhere.

### 8.1 Stage search

**Inputs**: an optional team identifier, an optional extra filter, an ordering (default: sequence
then identifier) and a limit (default: one).

1. Build the set of relevant team identifiers: the supplied team identifier if any, plus the team
   of every record in the current selection.
2. If the set is non-empty, the base filter is "the stage has no team restriction **or** the
   stage's teams include one of the relevant teams". If the set is empty, the base filter is "the
   stage has no team restriction".
3. Add the extra filter if supplied.
4. Search stages with that filter, that ordering and that limit and return them.

### 8.2 Default team selection

**Inputs**: an optional user (default: the acting user) and an optional extra filter.

Let the *valid company set* be the empty company plus every company that is both among the target
user's allowed companies and among the companies currently enabled for the acting user. Let the
*context default team* be the team named by a default supplied in the calling context, if any.

1. Search the teams whose company is in the valid company set and where the target user is either
   the leader or a member. Call this set *my teams*, ordered by the entity's default ordering
   (sequence ascending, then creation date descending, then identifier descending).
2. If *my teams* is non-empty **and** an extra filter was supplied: narrow *my teams* by the
   filter. If the context default team is in the narrowed set, return it; otherwise return the
   first of the narrowed set.
3. Otherwise, if *my teams* is non-empty: if the context default team is in *my teams*, return it;
   otherwise return the first of *my teams*.
4. Otherwise, if a context default team exists, return it.
5. Otherwise, search all teams whose company is in the valid company set. If an extra filter was
   supplied, return the first team matching it. If none matches or no filter was supplied, return
   the first team of that search.
6. If nothing is found, return nothing.

### 8.3 Address block synchronisation

**Input**: a Contact (possibly empty).

1. If **any** of the six address fields of the Contact is non-empty, the result is the Contact's
   values for all six fields — including the empty ones, so that an address is never mixed between
   two sources.
2. Otherwise the result is the lead's own current values for all six fields — that is, nothing
   changes.

### 8.4 Contact electronic mail update test

**Input**: a flag "force empty" (default: true). **Output**: true when the Contact's address ought
to be replaced by the lead's.

1. If the lead has no Contact, the answer is false.
2. If "force empty" is false **and** the lead's address is empty, the answer is false. (This guard
   stops an empty lead value from erasing a valid Contact value.)
3. If the lead's raw address text equals the Contact's raw address text, the answer is false.
4. Normalise both addresses (strip the display name, lower-case); if either cannot be normalised,
   fall back to the raw text, and treat an empty string as an absence.
5. The answer is true when the two normalised values differ.

### 8.5 Contact telephone update test

Identical in shape to section 8.4, with one difference: instead of normalising, both numbers are
formatted using the telephone formatting routine (which uses the record's country and the
company's country), falling back to the raw text when formatting fails, and treating an empty
string as an absence.

### 8.6 Values to copy from a Contact onto a Lead

**Input**: a Contact. **Output**: a set of lead values.

1. Start from the address block of section 8.3.
2. For each of the fields telephone, job position and website: take the Contact's value if it is
   non-empty, otherwise keep the lead's current value.
3. If the Contact has a language code, resolve it to a Language record and set the lead's language
   to it.
4. Apply the contact-name rule of section 1.4.6 and the company-name rule of section 1.4.7.

### 8.7 Values to create a Contact from a Lead

**Inputs**: a name, a flag "is a company", a parent identifier.

The created Contact receives:

| Contact field | Value taken from the lead |
|---|---|
| `name` | the supplied name |
| `user_id` | the salesperson named as a default in the calling context, otherwise the lead's salesperson |
| `comment` | the lead's notes |
| `phone` | the lead's telephone |
| `email` | the **first** address extracted from the lead's electronic mail field |
| `function` | the lead's job position |
| `street`, `street2`, `zip`, `city`, `country_id`, `state_id` | the lead's address block |
| `website` | the lead's website |
| `parent_id` | the supplied parent identifier |
| `is_company` | the supplied flag |
| `company_name` | the lead's company name, but **only** when the new Contact is not itself a company and has no parent |
| `type` | the fixed value `contact` |
| `lang` | the lead's language code, but only when that language is active |
| `partner_latitude`, `partner_longitude` | the lead's geographic coordinates, when the partner network capability is present |

### 8.8 Customer creation from a Lead

**Input**: an optional parent Contact. **Output**: the Contact to link.

1. Determine the individual's name: the lead's contact name if set; otherwise the name part parsed
   out of the lead's electronic mail address; otherwise nothing.
2. Determine the organisation Contact:
   - if a parent was supplied, that parent;
   - otherwise, if the lead has a company name, **create** a Contact flagged as a company with that
     name using the values of section 8.7;
   - otherwise, if the lead already has a Contact, that Contact;
   - otherwise nothing.
3. If an individual's name was determined, the answer is a newly **created** Contact with that name,
   not flagged as a company, whose parent is the organisation Contact determined above (possibly
   empty).
4. Otherwise, if an organisation Contact exists, that organisation Contact is the answer.
5. Otherwise the answer is a newly **created** Contact named after the lead's title and not flagged
   as a company.

### 8.9 Matching an existing Contact

1. If the lead already has a Contact, that Contact is the answer.
2. Otherwise, if the lead has a normalised or raw electronic mail address, look up a Contact by
   that address **without creating one**; the outcome of that lookup is the answer.
3. Otherwise the answer is empty.

---

## 9. Sales Team

**Sales Team** (`crm.team`, table `crm_team`).

### 9.1 Purpose

A Sales Team groups salespeople who share a pipeline, an electronic mail alias, a dashboard and a
set of record-visibility rules. In the assignment machinery a team is both a *bucket* (leads are
first allocated to teams) and a *capacity* (the sum of its members' capacities).

### 9.2 Field table — core

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Team (`name`) | single line text | **Required.** Translatable. |
| Sequence (`sequence`) | integer | Default 10. First key of the default ordering. |
| Active (`active`) | boolean | Default true. Archiving hides the team without deleting it. |
| Company (`company_id`) | link to Company | Indexed. May be empty, which makes the team available to every company. |
| Currency (`currency_id`) | link to Currency | Related to the company's currency; read-only. |
| Team leader (`user_id`) | link to User | Restricted to non-shared users. Company-checked. |

### 9.3 Field table — membership

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Multiple memberships allowed (`is_membership_multi`) | boolean | Not stored. Reads the configuration parameter `sales_team.membership_multi`. When false, a user may belong to at most one team at a time. |
| Salespersons (`member_ids`) | many-sided link to User | Not stored; computed from the active memberships, with an inverse that creates and archives memberships, and a search that delegates to the membership table. Restricted to non-shared users whose companies include the team's company (or any company when the team has none). |
| Member companies (`member_company_ids`) | many-sided link to Company | Not stored, interface only. The team's company if set, otherwise every company. |
| Membership issue warning (`member_warning`) | long text | Not stored. In single-membership mode, lists the users being added who already belong to another team, as "*the user names* already in other teams (*the team names*)." |
| Sales Team Members (`crm_team_member_ids`) | reverse link to Sales Team Member | The **active** memberships. |
| Sales Team Members including inactive (`crm_team_member_all_ids`) | reverse link to Sales Team Member | All memberships, active and archived. |

### 9.4 Field table — assignment

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Lead assign (`assignment_enabled`) | boolean | Not stored. True when the configuration parameter `crm.lead.auto.assignment` is set. |
| Auto assignment (`assignment_auto_enabled`) | boolean | Not stored. True when rule-based assignment is enabled **and** the lead-assignment scheduled job is active. |
| Skip auto assignment (`assignment_optout`) | boolean | When true the team is excluded from the scheduled allocation run. |
| Lead average capacity (`assignment_max`) | integer | Not stored. The **sum** of the capacities of the team's active members. |
| Assignment domain (`assignment_domain`) | single line text | A stored filter expression used to restrict which unassigned leads the team may take. Tracked. Empty means no restriction. |
| Unassigned leads (`lead_unassigned_count`) | integer | Not stored. The number of leads belonging to this team that have no salesperson. |
| Leads and opportunities assigned this month (`lead_all_assigned_month_count`) | integer | Not stored. The sum over active members of their thirty-day assigned counts. |
| Exceeded monthly assignment (`lead_all_assigned_month_exceeded`) | boolean | Not stored. True when the previous figure is strictly greater than the team capacity. |
| Lead properties definition (`lead_properties_definition`) | properties definition | The schema of the user-defined fields that leads of this team carry. |

### 9.5 Field table — pipeline usage and alias

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Leads (`use_leads`) | boolean | When true, the team qualifies incoming requests as unqualified leads before converting them. |
| Pipeline (`use_opportunities`) | boolean | Default true. When true the team manages a pre-sales pipeline of opportunities. |
| Alias (`alias_id`) | link to Electronic Mail Alias | **Required**, created automatically. Incoming electronic mail to this address creates a lead attached to the team. On delete: restricted. Not copied. |
| Alias name, domain, full address, contact policy, default values, status | various | All related to the alias record; see the Messaging domain. |

Interaction rules:

- Clearing **both** usage flags in the form clears the alias name.
- On writing either usage flag, the alias creation values are recomputed and written back: the
  aliased model is forced to the Lead model; if neither flag is set the alias name is cleared; the
  alias defaults are merged with `type` set to `lead` when the acting user holds the group *Show
  Lead Menu* and the team uses leads, otherwise `opportunity`, and with `team_id` set to the team.

### 9.6 Field table — dashboard and favourites

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Colour index (`color`) | integer | Default: a pseudo-random integer between 1 and 11. |
| Favourite members (`favorite_user_ids`) | many-sided link to User | Default: the creating user. Stored in association table `team_favorite_user_rel`. |
| Show on dashboard (`is_favorite`) | boolean | Not stored; computed as "the acting user is among the favourite members", with an inverse that adds or removes the acting user. |
| Dashboard button (`dashboard_button_name`) | single line text | Not stored. "Pipeline" when the team uses opportunities. When the team is viewed from inside the sales application, "Sales Analysis" instead. |

### 9.7 Ordering, display, defaults

- **Default ordering**: sequence ascending, then creation date descending, then identifier
  descending.
- **Display name**: the value of `name`.
- Three teams are created on installation: "Sales" (sequence 0, no company, led by the
  administrator, with a membership for the administrator), "Website" (archived by default) and
  "Point of Sale" (archived by default).

### 9.8 Constraints

| Constraint | Condition | Message |
|---|---|---|
| Company of members | For a team that has a company, every active member must have that company among their allowed companies. | "The following team members are not allowed in company '*the company*' of the Sales Team '*the team*': *the user names*" |
| Assignment domain | The stored filter expression must parse and must be usable as a search filter on leads. | "Assignment domain for team *the team* is incorrectly formatted" |
| Default teams | The teams "Website" and "Point of Sale" may not be deleted. | "Cannot delete default team "*the team name*"" |
| Team in active use by sales | Where the sales capability is installed, a team whose count of active sales orders is **five or more** may not be deleted. | "Team *the team name* has *the count* active sale orders. Consider cancelling them or archiving the team instead." |

### 9.9 Deletion behaviour — folding the frequency table

Deleting a team must not silently destroy the statistical history it accumulated. Before the
record is removed:

1. Collect every Scoring Frequency cell belonging to the deleted teams.
2. Collect the existing "no team" cells whose variable appears among those cells.
3. For each cell of the deleted team:
   - Skip it entirely when **both** its won count and its lost count are less than or equal to one
     tenth when compared at two decimals — these are pure seeding artefacts with no information.
   - Look for a "no team" cell with the same variable and the same value.
     - **If found**: round the existing won count, the existing lost count, the incoming won count
       and the incoming lost count each to the nearest whole number using half-up rounding; add
       them; store the sums, replacing any sum that is not strictly greater than one tenth by one
       tenth.
     - **If not found**: create a new "no team" cell with the same variable and value, whose won
       count is the incoming won count when that is strictly greater than one tenth and one tenth
       otherwise, and likewise for the lost count.
4. Delete the team. The remaining cells that still point at it are removed by the cascading
   reference.

### 9.10 Multi-company behaviour

A record rule restricts teams to those whose company is among the reader's enabled companies or is
empty. Writing a company on a team re-checks the membership company constraint for every member.

---

## 10. Sales Team Member

**Sales Team Member** (`crm.team.member`, table `crm_team_member`).

### 10.1 Purpose

The membership of one user in one team. It is a first-class record because it carries the
per-person assignment parameters: capacity, filter, preferred filter and pause switch.

### 10.2 Field table

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sales Team (`crm_team_id`) | link to Sales Team | **Required.** Indexed. On delete: **cascade**. Default: empty. Group expansion shows every team even when empty. Deliberately **not** company-checked, because the company check is done against the user instead. |
| Salesperson (`user_id`) | link to User | **Required.** Indexed. Company-checked. On delete: **cascade**. Restricted to non-shared users who are not already in the team (in single-membership mode) and whose companies include the team's company. |
| Active (`active`) | boolean | Default true. Archiving a membership is how a user leaves a team without losing the history. |
| Multiple memberships allowed (`is_membership_multi`) | boolean | Not stored. Reads the configuration parameter. |
| Member warning (`member_warning`) | long text | Not stored. In single-membership mode, "*the user name* already in other teams (*the team names*)." |
| Users already in teams (`user_in_teams_ids`) | many-sided link to User | Not stored, interface only. In multi-membership mode always empty. Otherwise, the users already holding a membership, so that the selection list excludes them. |
| User companies (`user_company_ids`) | many-sided link to Company | Not stored, interface only. The team's company if set, otherwise every company. |
| Image, image small (`image_1920`, `image_128`) | image | Related to the user. |
| Name (`name`) | single line text | Related to the user's display name; writable. |
| Electronic mail address (`email`) | single line text | Related to the user; read-only. |
| Telephone (`phone`) | single line text | Related to the user; read-only. |
| Company (`company_id`) | link to Company | Related to the user's company; read-only. |
| Lead assign (`assignment_enabled`) | boolean | Related to the team's flag; read-only. |
| Assignment domain (`assignment_domain`) | single line text | Tracked. A stored filter expression restricting which of the team's leads this member may receive. Empty means no restriction. |
| Preference assignment domain (`assignment_domain_preferred`) | single line text | Tracked. A second stored filter expression; leads matching **both** this and the ordinary domain are assigned to this member **before** any other lead is assigned. |
| Pause assignment (`assignment_optout`) | boolean | When true the member receives nothing from the assignment run. |
| Average leads capacity on thirty days (`assignment_max`) | integer | Default 30. The number of leads the member can absorb over thirty days. |
| Leads in the last twenty-four hours (`lead_day_count`) | integer | Not stored. The number of leads whose assignment date falls within the last twenty-four hours and that belong to this pair of user and team. Archived leads are excluded from the count. |
| Leads in the last thirty days (`lead_month_count`) | integer | Not stored. The same count over the last thirty days. |

### 10.3 Constraints

| Constraint | Condition | Message |
|---|---|---|
| Duplicate membership | In single-membership mode, the pair (team, user) must be unique **among active memberships**; archived duplicates are allowed, which is why this is a programmed check and not a stored uniqueness rule. | "You are trying to create duplicate membership(s). We found that *the user name (the team name)*, … already exist(s)." |
| Company of membership | If the team has a company, that company must be among the user's allowed companies. | "User '*the user*' is not allowed in the company '*the company*' of the Sales Team '*the team*'." |
| Assignment domain | Must parse and be usable as a lead filter. | "Member assignment domain for user *the user* and team *the team* is incorrectly formatted" |
| Preferred assignment domain | Must parse and be usable as a lead filter. | "Member preferred assignment domain for user *the user* and team *the team* is incorrectly formatted" |

### 10.4 Creation and write behaviour in single-membership mode

On creation, and on any write that activates a membership, the system **archives** every other
active membership of the same users. The procedure is:

1. Search the active memberships of the affected users.
2. Group them by user.
3. For each incoming pair (user, team), select that user's active memberships whose team differs
   from the incoming team.
4. Archive the selected memberships.

Creating a membership does not subscribe the member to the record's discussion thread; the thread
exists only to record tracked changes.

Manual re-pointing of an existing membership to another user or another team is explicitly
unsupported: the intended operations are create, archive and activate.

### 10.5 Ordering and display

- **Default ordering**: creation date ascending, then identifier. This ordering is load-bearing:
  the *main* team of a user is defined as the team of that user's **first-created** membership.
- **Display name**: the user's display name.

### 10.6 Effect on the User record

| Field on User (storage name) | Meaning |
|---|---|
| Sales Teams (`crm_team_ids`) | Not stored; the teams of the user's active memberships. Searching on it is rewritten into an explicit list of user identifiers when the result set is smaller than ten thousand, because the field is used inside record rules and an inline list is far cheaper. |
| Sales team memberships (`crm_team_member_ids`) | The reverse link to the membership records, ordered by creation date. |
| User sales team (`sale_team_id`) | Stored and computed: the team of the user's **first** membership in creation order, or empty when the user has none. Used as the default team for the pipeline and for invoicing. |

Archiving a user archives all of that user's memberships first.

---

## 11. Activity Analysis

**Activity Analysis** (`crm.activity.report`, database view `crm_activity_report`).

### 11.1 Purpose

A read-only reporting entity. Each row is one **completed activity** logged on a lead: it joins
the message that recorded the activity completion with the lead it belongs to.

### 11.2 Row definition

One row exists for every message whose model is the Lead model and whose activity type is set.
Each row exposes:

| Field (storage name) | Source |
|---|---|
| identifier | the message identifier |
| Creation date (`lead_create_date`) | the lead's creation date |
| Conversion date (`date_conversion`) | the lead's conversion date |
| Expected closing (`date_deadline`) | the lead's expected closing date |
| Closed date (`date_closed`) | the lead's closed date |
| Subtype (`subtype_id`) | the message subtype |
| Activity type (`mail_activity_type_id`) | the message's activity type |
| Assigned to (`author_id`) | the message author |
| Completion date (`date`) | the message date |
| Activity description (`body`) | the message body |
| Opportunity (`lead_id`) | the lead |
| Salesperson (`user_id`), Sales Team (`team_id`), Country (`country_id`), Company (`company_id`), Stage (`stage_id`), Customer (`partner_id`), Type (`lead_type`), Active (`active`), Won or lost (`won_status`) | the corresponding lead fields |
| Tags (`tag_ids`) | related to the lead's tags |

### 11.3 Access and rules

The view is readable by salespeople only. Two record rules apply: users in the group *User: All
Documents* see every row; users in the group *User: Own Documents Only* see only rows whose
salesperson is themselves or is empty. A third rule restricts rows to the reader's enabled
companies or to rows with no company.

---

## 12. Transient entities (wizards)

### 12.1 Convert to Opportunity Wizard

**Convert to Opportunity Wizard** (`crm.lead2opportunity.partner`, transient).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Conversion action (`name`) | selection | `convert` "Convert to opportunity", `merge` "Merge with existing opportunities". Computed from the detected duplicates and writable: when empty it becomes `merge` if at least two duplicates were detected, otherwise `convert`. |
| Related customer (`action`) | selection | **Required.** `create` "Create a new customer", `exist` "Link to an existing customer". Computed from the lead and writable: `exist` when a matching Contact is found by section 8.9, otherwise `create`. |
| Associated lead (`lead_id`) | link to Lead | **Required.** Defaulted from the active record of the calling context when not supplied. |
| Company name (`lead_partner_name`) | single line text | Related to the lead's company name. |
| Contact name (`lead_contact_name`) | single line text | Related to the lead's contact name. |
| Opportunities (`duplicated_lead_ids`) | many-sided link to Lead | Computed from the lead and the chosen Contact, writable, archived records included. The merge-oriented duplicate search of `calculations.md`, run with lost records **included**. |
| Company (`commercial_partner_id`) | link to Contact | Restricted to Contacts flagged as companies. Computed from the chosen Contact, writable: when the current value is empty or is not an ancestor of itself, and the chosen Contact has a parent, take that parent. |
| Customer (`partner_id`) | link to Contact | Computed from the action and the lead, writable: the matched Contact when the action is `exist`, nothing when it is `create`. |
| Salesperson (`user_id`) | link to User | Computed from the lead, writable: the lead's salesperson. |
| Sales Team (`team_id`) | link to Sales Team | Computed from the salesperson, writable, following the same rule as the lead's own team computation but with no extra filter. |
| Force assignment (`force_assignment`) | boolean | Default true. When true the salesperson is overwritten even on records that already have one. |

**Opening guard.** When the wizard is opened on a lead whose probability is exactly 100, it
refuses to open with the message "Closed/Dead leads cannot be converted into opportunities."

### 12.2 Mass Convert Wizard

**Mass Convert Wizard** (`crm.lead2opportunity.partner.mass`, transient). It inherits every field
of the single wizard and changes the following:

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Associated lead (`lead_id`) | link to Lead | No longer required. |
| Active leads (`lead_tomerge_ids`) | many-sided link to Lead | Default: the active records of the calling context. Archived records included. Stored in association table `crm_convert_lead_mass_lead_rel`. |
| Salespersons (`user_ids`) | many-sided link to User | The pool of salespeople to distribute the converted opportunities among, round-robin. |
| Apply deduplication (`deduplicate`) | boolean | Default true. When true, before converting, each lead is merged with its detected duplicates. |
| Related customer (`action`) | selection | Extended with `each_exist_or_create` "Use existing partner or create", which becomes the computed default. On removal of the extending capability the value falls back to `exist`. |
| Force assignment (`force_assignment`) | boolean | Default **false** in mass mode. |
| Conversion action (`name`) | selection | Always computed to `convert`. |
| Customer (`partner_id`) | link to Contact | Always computed to empty: a single customer cannot apply to many leads. |
| Company (`commercial_partner_id`) | link to Contact | Always empty: setting one company per lead is not supported in mass mode. |
| Opportunities (`duplicated_lead_ids`) | many-sided link to Lead | Computed from the selected leads: those leads for which the merge-oriented duplicate search (lost records **excluded**) returns more than one record. |

### 12.3 Merge Wizard

**Merge Wizard** (`crm.merge.opportunity`, transient).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Leads and opportunities (`opportunity_ids`) | many-sided link to Lead | Default: the active records of the calling context, filtered to those whose outcome is **not** won. Archived records included. Stored in association table `merge_opportunity_rel`. |
| Salesperson (`user_id`) | link to User | Restricted to non-shared users. Optional; when set it overrides the merged salesperson. |
| Sales Team (`team_id`) | link to Sales Team | Computed from the salesperson, writable: when a salesperson is chosen and the current team does not have that salesperson as leader or member, take the first team that does. |

### 12.4 Lost Reason Wizard

**Lost Reason Wizard** (`crm.lead.lost`, transient).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Leads (`lead_ids`) | many-sided link to Lead | The records to mark lost. Archived records included. |
| Lost reason (`lost_reason_id`) | link to Lost Reason | Optional. |
| Closing note (`lost_feedback`) | rich text | Optional, sanitised. When non-empty it is posted as the log message accompanying the tracked change. |

### 12.5 Probability Rebuild Wizard

**Probability Rebuild Wizard** (`crm.lead.pls.update`, transient).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Scoring start date (`pls_start_date`) | date | **Required.** Default: the current value of the configuration parameter `crm.pls_start_date`. |
| Scoring variables (`pls_fields`) | many-sided link to Scoring Frequency Field | Default: the catalogue entries matching the field names currently listed in the configuration parameter `crm.pls_fields`. |

Applying the wizard is restricted to administrators; see `business-rules.md`.

### 12.6 Quotation Contact Wizard

**Quotation Contact Wizard** (`crm.quotation.partner`, transient). Present only when the sales
capability is installed. It asks, before a quotation can be started from an opportunity that has
no Contact, whether to create a new Contact from the lead information or to link an existing one,
then performs the customer assignment and opens the new quotation.

---

## 13. Partner network entities

These entities exist when the partner network capability is installed. They turn the Contact
entity into a reselling partner directory and let an opportunity be forwarded to the nearest
suitable partner.

### 13.1 Partner Grade

**Partner Grade** (`res.partner.grade`, table `res_partner_grade`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sequence (`sequence`) | integer | Default 10. Determines the ranking order; the lowest sequence is the entry level. |
| Active (`active`) | boolean | Default true. |
| Level name (`name`) | single line text | Translatable. |
| Company (`company_id`) | link to Company | Default: the acting company. |
| Default price list (`default_pricelist_id`) | link to Price List | When set, assigning this level to a Contact also assigns this price list to that Contact. |
| Members count (`partners_count`) | integer | Not stored. The number of Contacts carrying this level. |
| Members label (`partners_label`) | single line text | Related to the company's configurable label for affiliates. |
| Level weight (`partner_weight`) | integer | Default 1. The relative chance of this partner being chosen by the geographic assignment. Zero means never assign. |
| Published (`is_published`) | boolean | Default true; makes the level visible on the public partner directory. |
| Public address (`website_url`) | single line text | Computed: the public directory path for this level. |

- **Default ordering**: sequence.
- **Default records**: three levels are supplied — see `configuration.md`.

### 13.2 Partner Activation

**Partner Activation** (`res.partner.activation`, table `res_partner_activation`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Sequence (`sequence`) | integer | Ordering key. |
| Name (`name`) | single line text | **Required.** |
| Active (`active`) | boolean | Default true. |

- **Default ordering**: sequence.

### 13.3 Fields added to the Contact

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Partner level (`grade_id`) | link to Partner Grade | Tracked. Group expansion shows every level. Writing it also writes the level's default price list onto the Contact, and rejects the write when a different price list is written at the same time. |
| Level weight (`partner_weight`) | integer | Computed from the level's weight, writable, stored, tracked. Zero means the Contact is never selected by geographic assignment. |
| Level sequence (`grade_sequence`) | integer | Related to the level's sequence; stored; read-only. |
| Activation (`activation`) | link to Partner Activation | Indexed, tracked. |
| Partnership date (`date_partnership`) | date | When the partnership started. |
| Latest review (`date_review`) | date | When the partnership was last reviewed. |
| Next review (`date_review_next`) | date | When the next review is due. |
| Implemented by (`assigned_partner_id`) | link to Contact | Indexed. The partner that implemented this customer. |
| Implementation references (`implemented_partner_ids`) | reverse link to Contact | The customers this partner implemented. |
| Implementation reference count (`implemented_partner_count`) | integer | Computed and stored: the number of **published** implementation references. |
| Opportunities (`opportunity_ids`) | reverse link to Lead | Restricted to records of type `opportunity`. |
| Opportunity count (`opportunity_count`) | integer | Not stored, visible only to salespeople. Counts opportunities of the Contact **and of every descendant Contact**, walking up the parent chain so that a parent accumulates its children's counts. With the partner network capability present the count also includes opportunities assigned to the Contact as a reselling partner, and a seen-set prevents double counting when the same Contact appears in both roles. |

When a Contact is created on the fly from a many-to-one selection with the appropriate context
flag, it receives the lowest-sequence level and the lowest-sequence activation by default, so that
it remains visible in the same selection list afterwards.

### 13.4 Partner Assignment Analysis

**Partner Assignment Analysis** (`crm.partner.report.assign`, database view
`crm_partner_report_assign`). A read-only aggregation built from the Contacts that carry a level or
an activation, joined with the opportunities forwarded to them and with the customer invoice
analysis. One row is one partner. Every column is read-only.

| Column (storage name) | Full name | Source |
|---|---|---|
| `partner_id` | Partner | the Contact |
| `grade_id` | Level | its partner level |
| `activation` | Activation | its activation, indexed |
| `user_id` | Salesperson | the salesperson of the Contact |
| `date_review` | Latest partner review | the latest review date |
| `date_partnership` | Partnership date | the date the partnership started |
| `country_id` | Country | the country of the Contact |
| `nbr_opportunities` | Number of opportunities | the number of opportunities forwarded to the partner |
| `turnover` | Turnover | the invoiced amount attached to the partner, read from the customer invoice analysis |
| `date` | Invoice accounting date | the accounting date of the invoice rows |

The delivered navigation restricts the view to rows whose level is set and presents it as a graph.

### 13.5 Forward to Partner Wizard

**Forward to Partner Wizard** (`crm.lead.forward.to.partner`, transient). Sends one or more
opportunities to one or more reselling partners by electronic mail, using a template, and records
the forwarding on each opportunity.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Forward selected leads to (`forward_type`) | selection | Values `single` labelled "a single partner: manual selection of partner" and `assigned` labelled "several partners: automatic assignment, using GPS coordinates and partner's grades". Default: the value imposed by the calling context, otherwise `single`. The two labels are reproduced verbatim because they are what the screen shows. |
| Forward leads to (`partner_id`) | link to Contact | The single recipient in `single` mode. Defaulted from the assigned partner of the first selected opportunity. |
| Partner assignment (`assignation_lines`) | reverse link to Lead Assignation Line | One line per selected opportunity. |
| Contents (`body`) | rich text | The message body, sanitised. Defaulted from the body of the forwarding message template. |

**Lead Assignation Line** (`crm.lead.assignation`, transient). One proposed pairing of an
opportunity with a partner inside that wizard.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Partner assignment (`forward_id`) | link to Forward to Partner Wizard | The parent wizard record. |
| Lead (`lead_id`) | link to Lead | The opportunity being forwarded. |
| Lead location (`lead_location`) | single line text | Computed at creation and refreshed when the lead changes: the country name and the city of the opportunity, separated by a comma. |
| Assigned partner (`partner_assigned_id`) | link to Contact | The proposed partner. In `assigned` mode it comes from the geographic search; in `single` mode it is the partner already carried by the opportunity. |
| Partner location (`partner_location`) | single line text | Refreshed when the proposed partner changes: the country name and the city of the partner, separated by a comma. |
| Link to lead (`lead_link`) | single line text | Computed at creation: the public address of the opportunity in the partner portal, built as the base web address followed by `/my/lead/` or `/my/opportunity/` and the record identifier. |

---

## 14. Lead generation entities

These entities exist when the lead generation capability is installed.

### 14.1 Lead Generation Request

**Lead Generation Request** (`crm.iap.lead.mining.request`, table `crm_iap_lead_mining_request`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Request number (`name`) | single line text | **Required**, read-only, **not copied**. Default: the text "New"; replaced by the next value of the numbering series when the request is submitted. |
| Status (`state`) | selection | **Required.** `draft` "Draft", `error` "Error", `done` "Done". Default `draft`. |
| Number of leads (`lead_number`) | integer | **Required.** Default 3. Clamped in the form to the range one to two hundred inclusive. |
| Target (`search_type`) | selection | **Required.** `companies` "Companies", `people` "Companies and their Contacts". Default `companies`. |
| Error type (`error_type`) | selection | Read-only, **not copied**. `credits` "Insufficient Credits", `no_result` "No Result". |
| Type (`lead_type`) | selection | **Required.** `lead` "Leads", `opportunity` "Opportunities". Default: `lead` when the acting user holds the group *Show Lead Menu*, otherwise `opportunity`. |
| Sales Team (`team_id`) | link to Sales Team | Computed from the salesperson and the type, writable, stored. Restricted to teams that use opportunities. On delete: set to empty. |
| Salesperson (`user_id`) | link to User | Default: the acting user. |
| Tags (`tag_ids`) | many-sided link to Tag | Applied to every generated lead. |
| Generated leads (`lead_ids`) | reverse link to Lead | The leads this request produced. |
| Number of generated leads (`lead_count`) | integer | Not stored. |
| Filter on size (`filter_on_size`) | boolean | Default false. |
| Size minimum (`company_size_min`) | integer | Default 1. Clamped in the form to at least one and at most the maximum. |
| Size maximum (`company_size_max`) | integer | Default 1000. Clamped in the form to at least the minimum. |
| Countries (`country_ids`) | many-sided link to Country | Default: the country of the acting user's company. |
| States (`state_ids`) | many-sided link to Country State | Cleared whenever the countries change. |
| Available states (`available_state_ids`) | reverse link to Country State | Not stored. Only the states of the countries that appear on a whitelist of countries for which the data service actually carries state information; offering the others would silently shrink the result set. |
| Industries (`industry_ids`) | many-sided link to Industry | Optional filter. |
| Number of contacts (`contact_number`) | integer | Default 10. Clamped in the form to the range one to five inclusive. |
| Filter on (`contact_filter_type`) | selection | `role` "Role", `seniority` "Seniority". Default `role`. |
| Preferred role (`preferred_role_id`) | link to Role | Used when filtering on role. |
| Other roles (`role_ids`) | many-sided link to Role | Used when filtering on role. |
| Seniority (`seniority_id`) | link to Seniority | Used when filtering on seniority. |
| Credit tooltips (`lead_credits`, `lead_contacts_credits`, `lead_total_credits`) | single line text | Not stored. Human-readable estimates of the service credits the request will consume. |

### 14.2 Industry, Role, Seniority

| Entity | Fields | Uniqueness |
|---|---|---|
| Industry (`crm.iap.lead.industry`) | Industry name (`name`, required, translatable), service identifiers (`reveal_ids`, required, a comma-separated list), colour (`color`), sequence (`sequence`); ordered by sequence then identifier. | Name unique — "Industry name already exists!" |
| Role (`crm.iap.lead.role`) | Role name (`name`, required, translatable), service identifier (`reveal_id`, required), colour (`color`). Display name is the name with underscores replaced by spaces and title-cased. | Name unique — "Role name already exists!" |
| Seniority (`crm.iap.lead.seniority`) | Name (`name`, required, translatable), service identifier (`reveal_id`, required). Display name is the name with underscores replaced by spaces and title-cased. | Name unique — "Name already exists!" |

### 14.3 Fields added to the Lead

| Field (storage name) | Type | Meaning |
|---|---|---|
| Lead generation request (`lead_mining_request_id`) | link to Lead Generation Request | Indexed. |
| Service company identifier (`reveal_id`) | single line text | Indexed. The identifier the data service assigned to the company; used to avoid asking for the same company twice. |

---

## 15. Campaign attribution entities

### 15.1 Campaign

**Campaign** (`utm.campaign`, table `utm_campaign`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Active (`active`) | boolean | Default true. |
| Campaign identifier (`name`) | single line text | **Required**, **unique**, not translatable. Computed from the title, writable, stored, precomputed. The uniqueness is achieved by appending a bracketed counter: see the unique-naming algorithm in `calculations.md`. |
| Campaign name (`title`) | single line text | **Required**, translatable. This is the human label; the record's display name is the title. |
| Responsible (`user_id`) | link to User | **Required.** Default: the acting user. |
| Stage (`stage_id`) | link to Campaign Stage | **Required**, **not copied**. Default: the first stage in sequence order. On delete: restricted. Group expansion shows every stage. |
| Tags (`tag_ids`) | many-sided link to Campaign Tag | Stored in association table `utm_tag_rel`. |
| Automatically generated (`is_auto_campaign`) | boolean | Default false. Set when the campaign was created implicitly from a tracking parameter rather than by a person. |
| Colour index (`color`) | integer | Presentation only. |
| Use leads (`use_leads`) | boolean | Not stored. True when the acting user holds the group *Show Lead Menu*; decides which navigation target the campaign's lead button opens. |
| Leads and opportunities count (`crm_lead_count`) | integer | Not stored, visible to salespeople. Counts leads attributed to this campaign, **including archived ones**. |

On creation, a campaign supplied with an identifier but no title takes the identifier as its title;
identifiers are then made unique.

### 15.2 Source

**Source** (`utm.source`, table `utm_source`). One field: Source name (`name`), required and
**unique**, uniqueness enforced both by a stored rule ("The name must be unique") and by the
automatic bracketed counter applied on creation. The record delivered as "Referral" may not be
deleted: "You cannot delete the 'Referral' UTM source record."

### 15.3 Medium

**Medium** (`utm.medium`, table `utm_medium`). Fields: Medium name (`name`), required, **unique**,
not translatable; Active (`active`), default true. Ordered by name. Six delivered records may not
be deleted — Email, Direct, Website, X, Facebook and LinkedIn — with the message "Oops, you can't
delete the Medium '*the name*'. Doing so would be like tearing down a load-bearing wall — not the
best idea."

### 15.4 Campaign Stage, Campaign Tag and the source mixin

| Entity | Fields |
|---|---|
| Campaign Stage (`utm.stage`) | Name (`name`, required, translatable), Sequence (`sequence`, default 1). Ordered by sequence. |
| Campaign Tag (`utm.tag`) | Name (`name`, required, translatable, **unique** — "Tag name already exists!"), Colour (`color`, default a pseudo-random integer between 1 and 11). Ordered by name. |

**Source mixin** (`utm.source.mixin`) is an abstract entity reused by records that are themselves a
traffic source, such as a mass mailing. It carries `source_id` (source), which is **required**, is
**not copied** on duplication and is restrictive on delete, and `name` (name), related to the name
of that Source. Creating such a record creates the Source when it is missing and names it after the
record's own content; duplicating one increments the bracketed counter of the source name using the
unique-name counter of `calculations.md`.

### 15.5 The attribution mixin (`utm.mixin`)

The campaign attribution mixin (`utm.mixin`) is an abstract entity: it has no table of its own and
contributes its three fields to every entity that reuses it, the Lead among them.

| Field (storage name) | Full name | Type | Meaning |
|---|---|---|---|
| `campaign_id` | Campaign | link to Campaign | The named effort that produced the record. |
| `source_id` | Source | link to Source | The origin of the link that produced the visit. |
| `medium_id` | Medium | link to Medium | The delivery method of that link. |

Every record that can be attributed carries three links — campaign, source and medium — each
indexed with a partial index that skips empty values. On the Lead these three are additionally
declared to clear themselves when the referenced record is deleted.

The default values of the three fields are taken from the visitor's browser cookies, **except**
when the acting user is a salesperson and is not acting with elevated rights: in that case no
attribution default is applied at all, because a salesperson creating a record by hand should not
inherit the attribution of their own browsing session.

When the cookie holds a text value for a link field, the corresponding record is looked up by a
case-insensitive exact name match among all records including archived ones, and is created if
absent; a campaign created this way is flagged as automatically generated.

---

## 16. Extensions to entities owned by other domains

### 16.1 Contact

| Field (storage name) | Type | Meaning |
|---|---|---|
| Opportunities (`opportunity_ids`) | reverse link to Lead | Leads of type `opportunity` whose Contact is this one. |
| Opportunity count (`opportunity_count`) | integer | Not stored, visible only to salespeople; see section 13.3 for the full hierarchical rule. |

A Contact also gains an application statistic entry showing the opportunity count with a star icon
when that count is non-zero.

### 16.2 Meeting

| Field (storage name) | Type | Meaning |
|---|---|---|
| Opportunity (`opportunity_id`) | link to Lead | Restricted to records of type `opportunity`. Indexed. On delete: set to empty. |

Behaviour:

- When a meeting is created with an opportunity and it has no linked activity, a note is posted on
  the opportunity recording the scheduled time, the subject as a link and the duration; a missing
  duration is rendered as the word "unknown".
- When the calendar is opened from a lead, the generic document reference of the new meeting is
  pre-filled with that lead, and the opportunity link is derived from it.
- A meeting whose opportunity is the record currently being viewed is highlighted.

### 16.3 Sales Order

| Field (storage name) | Type | Meaning |
|---|---|---|
| Opportunity (`opportunity_id`) | link to Lead | Restricted to records of type `opportunity` whose company is empty or equal to the order's company. Indexed with a partial index that skips empty values. Company-checked. |

Behaviour: confirming an order feeds the expected revenue back to the opportunity — see
`calculations.md`.

### 16.4 Website

| Field (storage name) | Type | Meaning |
|---|---|---|
| Default sales team (`crm_default_team_id`) | link to Sales Team | The team applied to leads created through the public contact form. Restricted to teams that use leads when the acting user holds the group *Show Lead Menu*, otherwise to teams that use opportunities. |
| Default salesperson (`crm_default_user_id`) | link to User | The salesperson applied to leads created through the public contact form. Restricted to non-shared users. |

### 16.5 Website Visitor

| Field (storage name) | Type | Meaning |
|---|---|---|
| Leads (`lead_ids`) | many-sided link to Lead | Visible to salespeople. |
| Leads count (`lead_count`) | integer | Not stored. |

Behaviour: a visitor's electronic mail address and telephone fall back to those of its most
recently created lead that has one; a visitor tied to at least one lead is never purged as
inactive; merging two visitors moves the leads onto the surviving visitor.

### 16.6 Discussion Channel

| Field (storage name) | Type | Meaning |
|---|---|---|
| Leads (`lead_ids`) | reverse link to Lead | Visible to salespeople. The leads created out of this live chat conversation. |
| Has lead (`has_crm_lead`) | boolean | Computed and stored. Indexed with a partial index restricted to true values. It is the flag a record rule uses to grant salespeople read access to the conversations behind their leads. |

### 16.7 Periodic Digest

| Field (storage name) | Type | Meaning |
|---|---|---|
| New leads (`kpi_crm_lead_created`) | boolean | Whether the digest includes the count of leads created in the period. |
| New leads value (`kpi_crm_lead_created_value`) | integer | The count, computed per company. |
| Opportunities won (`kpi_crm_opportunities_won`) | boolean | Whether the digest includes the count of opportunities won in the period. |
| Opportunities won value (`kpi_crm_opportunities_won_value`) | integer | The count of records of type `opportunity` with probability 100, dated by closed date, computed per company. |

Both computations raise an access error for a user who is not a salesperson, which the digest
machinery treats as "omit this figure for this recipient".

### 16.8 Company

| Field (storage name) | Type | Meaning |
|---|---|---|
| Partnership label (`partnership_label`) | single line text | Translatable. Default "Members". The word a deployment uses to name its affiliates — for example "Partners", "Members" or "Alumni". It is shown in the affiliate menu, on the level screens and on the price list screen. |

### 16.9 Product Template and Price List

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Product Template | Create on order (`service_tracking`) | selection | Extended with the value `partnership`, labelled "Membership / Partnership". Selling such a product grants a partner level to the customer. When the membership capability is removed the value falls back to the default. |
| Product Template | Assigned level (`grade_id`) | link to Partner Grade | The level the product grants. |
| Price List | Partners count (`partners_count`) | integer | Not stored. The number of Contacts using this price list as their own. |
| Price List | Partners label (`partners_label`) | single line text | Related to the company's partnership label. |

### 16.10 Sales Order — the level granted by an order

| Field (storage name) | Type | Meaning |
|---|---|---|
| Assigned level (`assigned_grade_id`) | link to Partner Grade | Not stored. Computed from the order lines: the single level carried by the lines whose service tracking is `partnership`. Confirming the order writes that level onto the commercial entity of the customer. |

A validation on the order lines refuses an order that would grant two different levels: "You cannot
confirm Sale Order *the order reference* because there are products assigning different grades."

### 16.11 Discussion Channel, Chatbot Script and Chatbot Script Step

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Chatbot Script | Leads (`lead_count`) | integer | Not stored. The number of Leads whose source is the source of the script. A navigation control opens them. |
| Chatbot Script Step | Step type (`step_type`) | selection | Extended with `create_lead` labelled "Create Lead" and `create_lead_and_forward` labelled "Create Lead & Forward". Deleting the referenced record cascades for both values. |
| Chatbot Script Step | Sales Team (`crm_team_id`) | link to Sales Team | Indexed with a partial index that skips empty values. On delete: set to empty. The team written on the Lead the step creates. |

The step type `create_lead_and_forward` also counts as an operator-forwarding step, so the
conversation is handed to a human afterwards.

### 16.12 Survey, Survey Question, Survey Answer Option and Survey Participation

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Survey | Lead generating (`generate_lead`) | boolean | Computed and stored. True when at least one answer option of the survey generates a Lead. |
| Survey | Leads (`lead_ids`) | reverse link to Lead | The Leads produced by the survey. |
| Survey | Leads (`lead_count`) | integer | Not stored. Their number. |
| Survey | Sales Team (`team_id`) | link to Sales Team | The team written on the Leads produced by the survey. |
| Survey Question | Survey type (`survey_type`) | selection | Related to the survey. |
| Survey Question | Lead generating (`generate_lead`) | boolean | Not stored. True when at least one answer option of the question generates a Lead. |
| Survey Answer Option | Lead creation (`generate_lead`) | boolean | Choosing this answer creates a Lead. |
| Survey Participation | Lead (`lead_id`) | link to Lead | On delete: set to empty. The Lead the completed participation produced. |

### 16.13 Event, Event Registration and Event Question Answer

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Event | Leads (`lead_ids`) | reverse link to Lead | The Leads produced by the registrations of the event. |
| Event | Leads count (`lead_count`) | integer | Not stored. Their number. |
| Event Registration | Leads (`lead_ids`) | many-sided link to Lead | Read-only. The Leads produced from this registration. |
| Event Registration | Leads count (`lead_count`) | integer | Not stored. |

Behaviour: creating, confirming and marking attended a registration runs the matching Event Lead
Rules; changing the contact data or the description of a registration updates the Leads already
produced; the rules do **not** run while registrations are created by an import. An event manager
may regenerate the Leads of an event; any other user is refused with "Only Event Managers are
allowed to re-generate all leads."

An Event Question Answer offers an **Add rules** control that opens the creation dialogue of an
Event Lead Rule, pre-filled with the label of the answer option as the rule name, the acting user as
the salesperson written on the created records, and a registration condition selecting the
registrations that chose that answer option to that question. Nothing is written until the user
saves the dialogue.

### 16.14 Campaign and Mass Mailing

| Entity | Field (storage name) | Type | Meaning |
|---|---|---|---|
| Campaign | Use leads (`use_leads`) | boolean | Not stored. True when the reader holds the group *Show Lead Menu*; decides which navigation target the lead control opens. |
| Campaign | Leads and opportunities count (`crm_lead_count`) | integer | Not stored, visible to salespeople. Counts the Leads attributed to the campaign, archived ones included. |
| Campaign | Winner selection (`ab_testing_winner_selection`) | selection | Extended with a value that picks the variant whose source produced the most Leads. The same extra value is added to the split-test criterion of the electronic mail variants and to the split-test criterion of the text message variants. |
| Mass Mailing | Use leads (`use_leads`) | boolean | Not stored. |
| Mass Mailing | Leads count (`crm_lead_count`) | integer | Not stored. Counts, with elevated rights and archived records included, the Leads whose source is the source of the mailing. |

### 16.15 Live chat channel report

| Field (storage name) | Type | Meaning |
|---|---|---|
| Leads created (`leads_created`) | integer | Read-only, aggregated as a sum. The number of Leads created from the conversations of the channel. |

### 16.16 Activity

An Activity gains no field. When a Meeting is created from an Activity whose meeting already points
at an opportunity, the meeting defaults of that opportunity are merged into the creation context:
the opportunity itself, the Contact of the opportunity as the default customer, the acting user's
Contact plus that customer as the default attendees, the team of the opportunity and the title of
the opportunity as the default title. The initial date offered is the start of the meeting that
already exists, and the relevant-period heuristic of `interfaces.md` is deliberately not applied,
because the period is already known.

---

## 17. Event lead generation entities

These entities exist when the events coupling is installed.

### 17.1 Event Lead Rule

**Event Lead Rule** (`event.lead.rule`, table `event_lead_rule`). A rule that turns event
registrations into Leads.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Rule name (`name`) | single line text | **Required.** Translatable. |
| Active (`active`) | boolean | Default true. |
| Created leads (`lead_ids`) | reverse link to Lead | Visible to salespeople. The Leads the rule produced. |
| Create (`lead_creation_basis`) | selection | **Required.** Default `attendee`. Values `attendee` labelled "Per Attendee" — one Lead per registration — and `order` labelled "Per Order" — one Lead per batch of registrations. |
| When (`lead_creation_trigger`) | selection | **Required.** Default `create`. Values `create` labelled "Attendees are created", `confirm` labelled "Attendees are registered", `done` labelled "Attendees attended". |
| Event templates (`event_type_ids`) | many-sided link to Event Template | Restricts the rule to events of these categories. Empty means no category restriction. |
| Event (`event_id`) | link to Event | Restricts the rule to one event. Restricted to events of the rule's company or of no company. |
| Company (`company_id`) | link to Company | Restricts the rule to events of one company. Empty means no company restriction. |
| Registrations condition (`event_registration_filter`) | long text | An extra stored filter expression applied to the registrations. |
| Lead type (`lead_type`) | selection | **Required.** Values `lead` and `opportunity`. Default: `lead` when the acting user holds the group *Show Lead Menu*, otherwise `opportunity`. |
| Sales Team (`lead_sales_team_id`) | link to Sales Team | On delete: set to empty. Written on the created Leads. |
| Salesperson (`lead_user_id`) | link to User | Written on the created Leads. |
| Tags (`lead_tag_ids`) | many-sided link to Tag | Added to the created Leads. |

- **Ordering**: identifier. **Display name**: the rule name.
- **On change**: choosing a Sales Team whose leader is set proposes that leader as the salesperson
  written on the created records.
- **Company scoping**: a record rule restricts a multi-company reader to rules whose company is
  among the reader's enabled companies or is empty.

**Which registrations one rule keeps.** A registration is kept when all three hold:

1. the registrations condition is empty, is the empty filter, or the registration matches it;
2. the rule has no company, or the registration's company is that company;
3. the rule has neither an event nor an event template, **or** the registration's event is the
   rule's event, **or** the category of the registration's event is one of the rule's event
   templates.

### 17.2 Event Lead Request

**Event Lead Request** (`event.lead.request`, table `event_lead_request`). The bookkeeping record of
a resumable batch run that regenerates the Leads of one event. It has no audit fields.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Event (`event_id`) | link to Event | **Required.** On delete: cascade. The event being processed. It is also the display name of the record. |
| Lead rules (`event_lead_rule_ids`) | many-sided link to Event Lead Rule | The rules to apply. |
| Processed registration (`processed_registration_id`) | integer | The identifier of the last processed registration, used to resume the run. |

- **Ordering**: identifier ascending.
- **Uniqueness**: one request per event, enforced by a stored rule — "You can only have one
  generation request per event at a time."

---

## 18. Website identification entities

These entities exist when the website identification capability is installed. They turn identified
website traffic into Leads.

### 18.1 Lead Generation Rule

**Lead Generation Rule** (`crm.reveal.rule`, table `crm_reveal_rule`).

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Rule name (`name`) | single line text | **Required.** |
| Active (`active`) | boolean | Default true. |
| Countries (`country_ids`) | many-sided link to Country | Only visitors located in these countries are converted. Empty means every country. |
| Website (`website_id`) | link to Website | Restricts the rule to one website. |
| States (`state_ids`) | many-sided link to Country State | Only visitors located in these states are converted. |
| Address expression (`regex_url`) | single line text | A pattern matched against the visited path. Empty tracks the whole site; a single slash targets the home page; a pattern such as a prefix followed by a star tracks every path beginning with that prefix. |
| Sequence (`sequence`) | integer | Orders the rules that share a path pattern and a country; the lowest sequence is evaluated first. |
| Industries (`industry_tag_ids`) | many-sided link to Industry | Empty always matches; otherwise no Lead is created when the identified company does not match. |
| Filter on size (`filter_on_size`) | boolean | Default true. |
| Size minimum (`company_size_min`) | integer | Default 0. |
| Size maximum (`company_size_max`) | integer | Default 1000. |
| Filter on (`contact_filter_type`) | selection | **Required.** Default `role`. Values `role` "Role" and `seniority` "Seniority". |
| Preferred role (`preferred_role_id`) | link to Role | Used when filtering on role. |
| Other roles (`other_role_ids`) | many-sided link to Role | Used when filtering on role. |
| Seniority (`seniority_id`) | link to Seniority | Used when filtering on seniority. |
| Number of contacts (`extra_contacts`) | integer | Default 1. The number of contacts tracked per identified company; each one consumes one service credit. |
| Data tracking (`lead_for`) | selection | **Required.** Default `companies`. Values `companies` "Companies" and `people` "Companies and their Contacts". |
| Type (`lead_type`) | selection | **Required.** Default `opportunity`. Values `lead` and `opportunity`. |
| Suffix (`suffix`) | single line text | Appended to the title of the created records so that the rule can be recognised. |
| Sales Team (`team_id`) | link to Sales Team | On delete: set to empty. Written on the created records. |
| Tags (`tag_ids`) | many-sided link to Tag | Written on the created records. |
| Salesperson (`user_id`) | link to User | Written on the created records. |
| Priority (`priority`) | selection | Written on the created records. |
| Generated leads (`lead_ids`) | reverse link to Lead | The records the rule produced. |
| Number of generated leads (`lead_count`) | integer | Not stored. |
| Number of generated opportunities (`opportunity_count`) | integer | Not stored. |

- **Ordering**: sequence.
- **Stored check** on the number of contacts: at least one and at most five — "Maximum 5 contacts
  are allowed!"
- **Programmed validation** on the address expression: the pattern must compile — "Enter Valid
  Regex."
- Creating, writing or deleting a rule clears the cached rule table used while serving pages.

### 18.2 Reveal View

**Reveal View** (`crm.reveal.view`, table `crm_reveal_view`). One visit of one network address to a
page matched by a Lead Generation Rule, waiting to be resolved into a company.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Network address (`reveal_ip`) | single line text | The address the visit came from. It is the display name of the record. |
| Lead generation rule (`reveal_rule_id`) | link to Lead Generation Rule | Indexed with a partial index that skips empty values. |
| State (`reveal_state`) | selection | Default `to_process`. Values `to_process` "To Process" and `not_found` "Not Found". Indexed. |
| Creation date (`create_date`) | date and time | Indexed; used to expire old rows. |

- **Ordering**: identifier descending.
- **Unique index** on the pair (`reveal_rule_id`, `reveal_ip`), so one address produces at most one
  pending row per rule.
- **Index** on the pair (`reveal_state`, `create_date`), which is what the scheduled job scans.

---

## 19. Telephone validation and blacklist entities

These entities come with the telephone validation capability and are used by the Lead through the
telephone mixin.

### 19.1 Telephone blacklist

**Telephone Blacklist** (`phone.blacklist`, table `phone_blacklist`). One blacklisted number.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Phone number (`number`) | single line text | **Required.** Stored in international notation with no separators. Tracked. Searchable through a dedicated search rule. It is the display name of the record. |
| Active (`active`) | boolean | Default true. Tracked. Removing a number from the blacklist archives the record rather than deleting it. |

- **Uniqueness**: a stored rule on the number — "Number already exists".
- **Creation behaviour**: creating a number that already exists as an archived record reactivates
  that record instead of creating a second one; creating one that already exists and is active
  returns the existing record. A number that cannot be sanitised is refused with "*the parsing
  error* Please correct the number and try again."
- **Access**: only the system administration group may read or write it directly; every other path
  goes through the add and remove operations.

### 19.2 Remove a telephone number from the blacklist

**Remove Telephone From Blacklist** (`phone.blacklist.remove`, transient). Asks for a reason before
a number is taken off the blacklist.

| Field (storage name) | Type | Meaning and rules |
|---|---|---|
| Phone number (`phone`) | single line text | **Required**, read-only. The number being removed. |
| Reason (`reason`) | single line text | Free text recorded with the removal. |

Applying the wizard removes the number and posts the reason. A user without the right to
un-blacklist is refused with "You do not have the access right to unblacklist phone numbers. Please
contact your administrator."

### 19.3 The telephone mixin

**Telephone blacklist mixin** (`mail.thread.phone`) is an abstract entity. It contributes to the
Lead:

| Field (storage name) | Type | Meaning |
|---|---|---|
| `phone_sanitized` | single line text | Computed and stored. The primary telephone field of the record reduced to international notation with no separators, using the record's country and then the company's country as the parsing hint. Empty when the number cannot be parsed. |
| `phone_sanitized_blacklisted` | boolean | Not stored, searchable, visible to internal users only. True when the sanitised number is on the telephone blacklist. |
| `phone_blacklisted` | boolean | Not stored, visible to internal users only. Tells which of the record's number fields is the blacklisted one, for records that carry both a landline and a mobile number. |
| `phone_mobile_search` | single line text | Not stored, searchable. A search over this field matches the raw number fields **and** the sanitised number, so that a search for a number written in international notation also finds records storing it in a local form. |

Two refusals belong to the mixin: a search on the telephone with fewer than three characters is
refused with "Please enter at least 3 characters when searching a Phone number."; a record that
declares no primary telephone field is refused with "Missing definition of phone fields."

---

## 20. Lead enrichment helpers

**Lead Enrichment Helpers** (`crm.iap.lead.helpers`, table `crm_iap_lead_helpers`). A stateless
service entity with no field of its own, shared by the lead generation capability and by the website
identification capability. Internal users have no access to it; it is only reachable from the
operations that use it.

| Operation | Inputs | Behaviour |
|---|---|---|
| Notify that credits are exhausted | the service name, the entity name, the name of the throttling parameter | Sends the delivered notification message to the buyers of the external credit account and records the instant in a system parameter, so that the same notification is not repeated inside the throttling window. |
| Build record values from a service answer | the record type, the team, the tags, the salesperson, the company data and optionally the data of the first person | Produces the field values of a new Lead: the type, the team, the tags and the salesperson as supplied; the service company identifier; the title and the company name taken from the company name and falling back on the domain; the electronic mail address; the telephone; the website built as `https://` followed by the domain; and the six address fields with the country and the state resolved from their codes. When person data is supplied, the contact name, the electronic mail address and the job position of the first person override the company values. |
| Resolve a state code | a state code and a country | Returns the state of that country whose code matches, and nothing when no state matches. |

---

## 21. Generated reference pages

Every entity named above has a generated reference page carrying its complete field list, its
constraints, its access rights and its views.

| Entity | Reference page |
|---|---|
| Lead | [../../references/entities/crm.lead.md](../../references/entities/crm.lead.md) |
| Stage | [../../references/entities/crm.stage.md](../../references/entities/crm.stage.md) |
| Tag | [../../references/entities/crm.tag.md](../../references/entities/crm.tag.md) |
| Lost Reason | [../../references/entities/crm.lost.reason.md](../../references/entities/crm.lost.reason.md) |
| Recurring Plan | [../../references/entities/crm.recurring.plan.md](../../references/entities/crm.recurring.plan.md) |
| Scoring Frequency | [../../references/entities/crm.lead.scoring.frequency.md](../../references/entities/crm.lead.scoring.frequency.md) |
| Scoring Frequency Field | [../../references/entities/crm.lead.scoring.frequency.field.md](../../references/entities/crm.lead.scoring.frequency.field.md) |
| Sales Team | [../../references/entities/crm.team.md](../../references/entities/crm.team.md) |
| Sales Team Member | [../../references/entities/crm.team.member.md](../../references/entities/crm.team.member.md) |
| Activity Analysis | [../../references/entities/crm.activity.report.md](../../references/entities/crm.activity.report.md) |
| Convert to Opportunity Wizard | [../../references/entities/crm.lead2opportunity.partner.md](../../references/entities/crm.lead2opportunity.partner.md) |
| Mass Convert Wizard | [../../references/entities/crm.lead2opportunity.partner.mass.md](../../references/entities/crm.lead2opportunity.partner.mass.md) |
| Merge Wizard | [../../references/entities/crm.merge.opportunity.md](../../references/entities/crm.merge.opportunity.md) |
| Lost Reason Wizard | [../../references/entities/crm.lead.lost.md](../../references/entities/crm.lead.lost.md) |
| Probability Rebuild Wizard | [../../references/entities/crm.lead.pls.update.md](../../references/entities/crm.lead.pls.update.md) |
| Quotation Contact Wizard | [../../references/entities/crm.quotation.partner.md](../../references/entities/crm.quotation.partner.md) |
| Lead Generation Request | [../../references/entities/crm.iap.lead.mining.request.md](../../references/entities/crm.iap.lead.mining.request.md) |
| Industry | [../../references/entities/crm.iap.lead.industry.md](../../references/entities/crm.iap.lead.industry.md) |
| Role | [../../references/entities/crm.iap.lead.role.md](../../references/entities/crm.iap.lead.role.md) |
| Seniority | [../../references/entities/crm.iap.lead.seniority.md](../../references/entities/crm.iap.lead.seniority.md) |
| Lead Enrichment Helpers | [../../references/entities/crm.iap.lead.helpers.md](../../references/entities/crm.iap.lead.helpers.md) |
| Partner Grade | [../../references/entities/res.partner.grade.md](../../references/entities/res.partner.grade.md) |
| Partner Activation | [../../references/entities/res.partner.activation.md](../../references/entities/res.partner.activation.md) |
| Partner Assignment Analysis | [../../references/entities/crm.partner.report.assign.md](../../references/entities/crm.partner.report.assign.md) |
| Forward to Partner Wizard | [../../references/entities/crm.lead.forward.to.partner.md](../../references/entities/crm.lead.forward.to.partner.md) |
| Lead Assignation Line | [../../references/entities/crm.lead.assignation.md](../../references/entities/crm.lead.assignation.md) |
| Event Lead Rule | [../../references/entities/event.lead.rule.md](../../references/entities/event.lead.rule.md) |
| Event Lead Request | [../../references/entities/event.lead.request.md](../../references/entities/event.lead.request.md) |
| Lead Generation Rule | [../../references/entities/crm.reveal.rule.md](../../references/entities/crm.reveal.rule.md) |
| Reveal View | [../../references/entities/crm.reveal.view.md](../../references/entities/crm.reveal.view.md) |
| Telephone Blacklist | [../../references/entities/phone.blacklist.md](../../references/entities/phone.blacklist.md) |
| Remove Telephone From Blacklist | [../../references/entities/phone.blacklist.remove.md](../../references/entities/phone.blacklist.remove.md) |
| Telephone blacklist mixin | [../../references/entities/mail.thread.phone.md](../../references/entities/mail.thread.phone.md) |
| Campaign | [../../references/entities/utm.campaign.md](../../references/entities/utm.campaign.md) |
| Source | [../../references/entities/utm.source.md](../../references/entities/utm.source.md) |
| Medium | [../../references/entities/utm.medium.md](../../references/entities/utm.medium.md) |
| Campaign Stage | [../../references/entities/utm.stage.md](../../references/entities/utm.stage.md) |
| Campaign Tag | [../../references/entities/utm.tag.md](../../references/entities/utm.tag.md) |
| Attribution mixin | [../../references/entities/utm.mixin.md](../../references/entities/utm.mixin.md) |
| Source mixin | [../../references/entities/utm.source.mixin.md](../../references/entities/utm.source.mixin.md) |

---

## 22. Reconciliation notes

Two independently written descriptions of this domain were consolidated into this file. Where they
agreed, the more precise wording was kept once. The differences that had to be resolved against the
behaviour of the system are recorded here.

| Subject | The two statements | Resolution |
|---|---|---|
| `email_domain_criterion` for a free public provider | One version said the criterion is **empty** for an address at a free public provider; the other said it is the **whole address**. | The whole address is correct. The domain-preparation routine returns the entire normalised address when the domain is a free public provider, and the entire text when there is no at sign at all. Section 1.4.13 now states this, and `calculations.md` was corrected in the same way. Consequence: two Leads carrying the *same* public address are still detected as duplicates, while two different individuals at that provider are not. |
| `show_enrich_button` | One version made the flag depend on the enrichment mode setting; the other listed six record-level conditions. | The six record-level conditions are correct and the enrichment mode plays no part in the flag. Section 1.3.11 now lists them; the mode only decides whether the scheduled job runs. |
| `date_partner_assign` | One version described a plain date field; the other described a stored computed field driven by the assigned partner. | The stored computed field is correct: it is recomputed from `partner_assigned_id`, remains writable, is copied on duplication, and is cleared when the assigned partner is cleared. |
| Deleting a Sales Team that has sales orders | One version listed only the two delivered teams as undeletable; the other added a rule about active sales orders and described the threshold as "more than five". | Both rules exist. The threshold is **five or more** active sales orders, not more than five. Section 9.8 now carries both constraints with the corrected threshold. |
| Naming of the frequency-table variables | One version wrote the variables as `stage`, `country`, `tag`; the other as `stage_id`, `country_id`, `tag_id`. | The storage names are contractual, so `stage_id`, `country_id`, `state_id`, `source_id`, `lang_id`, `email_state`, `phone_state` and the singular `tag_id` are used everywhere. |
| Field naming convention | One version used descriptive canonical names such as `sales_team` and `probability_percentage`; the other reproduced the storage names. | The storage names are reproduced, because a rebuild that must import an existing database or serve an existing integration depends on them character for character. Every field table therefore carries the storage name and the full name in words. |
