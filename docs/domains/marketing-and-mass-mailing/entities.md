# Marketing entities

Every persistent entity of this domain is specified below: its identity rules, its default ordering, its display-name rule, its company scoping, its archiving behavior, every field with its type, requirement, default, storage, derivation, copy behavior and tracking, its database constraints, its validations with the exact messages, its on-change behavior and its life cycle. Fields that this domain adds to entities owned by other domains are in section 28.

## 0. Conventions

**Shared fields.** Every persistent entity has `id` (the surrogate integer primary key, called the identifier in prose), `create_date` (created on), `create_uid` (created by user), `write_date` (last updated on) and `write_uid` (last updated by user). Entities that support archiving also have `active` (boolean, default true; archived records are hidden from default queries). These are not repeated in the field tables below, except where a specific behavior is attached to them, for example the mailing list archiving guard, or the fact that Mailing Filter shows `create_uid` under the label *Saved by*.

**Copy behavior.** Unless a field row says otherwise, a field is copied when the record is duplicated.

**Tracking.** "Tracked" means a change of the field is written as a change entry in the discussion thread of the record. Only Mass Mailing, Mailing Contact, Marketing Card Campaign and the blocked-address record of the messaging domain have a discussion thread in this domain.

**Address normalisation.** Wherever this document says an address is *normalised*, it means the lower-cased, comment-free, single-address form produced by the messaging domain (see [../messaging-and-activities/calculations.md](../messaging-and-activities/calculations.md)). Comparison of addresses is always done on the normalised form.

**Number sanitising.** Wherever this document says a telephone number is *sanitised*, it means the strict international form produced by the telephone-validation service of the messaging domain, computed against the country of the record, of the visitor or of the company.

**Identifiers and full names.** The first column of every field table holds the identifier exactly as it is stored, in code font, because a rebuild that has to import an existing database or serve an existing integration depends on it character for character. The second column holds the full name of the same field in words. Prose always uses the full name or the canonical entity name; an identifier appears in prose only in code font.

**Abbreviated identifiers.** Several stored identifiers are abbreviated in ways this specification never uses in prose. `sms` inside an identifier always means text message; `kpi_mail_required` is the flag that asks for the statistics message sent one day after a mailing; `mail_mail_id_int` and `sms_id_int` are plain integer copies of the keys of the outgoing electronic mail and of the outgoing text message, kept after those records are deleted; `ab_testing` inside an identifier always means comparison testing; `pc` inside an identifier means percentage; `crm_lead_count` counts leads and opportunities; `utm` inside an identifier always means campaign tracking. Each is expanded in the full-name column of the table where it appears.

---

## 1. Mass Mailing

**Transport name** `mailing.mailing` · **storage name** `mailing_mailing` · **generated reference page** [`mailing.mailing`](../../references/entities/mailing.mailing.md)

One designed message with its audience, schedule, delivery state and measured results. It is the central entity of the domain.

- **Identity and uniqueness.** No natural key. The record owns a Campaign Source (see section 18) whose name is unique across all sources; the mailing therefore has a unique `name` in practice, generated from the subject when not given.
- **Default ordering.** `calendar_date` descending.
- **Display name.** The `subject`.
- **Company scoping.** None. A mailing is not company-scoped; the sender address, the outgoing mail server and the statistics message are resolved from the responsible user's company.
- **Archiving.** Supported (`active`, default true, tracked). Archived mailings stay visible in the favorite-design gallery.
- **Discussion thread.** Yes, with activities. Notes are logged by the test wizard and by the text-message test wizard.
- **Rendering.** The mailing is a template: `subject`, `preview`, `body_html` and `body_plaintext` contain placeholders evaluated against each recipient record. The rendering entity is `mailing_model_real`.

### 1.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Copied | Tracked | Meaning and derivation |
|---|---|---|---|---|---|---|---|---|
| `subject` | Subject | text | yes | empty | stored, not translated | yes | no | The line the recipient sees in the inbox. For a text-message mailing it is the internal title only. Placeholders allowed. |
| `preview` | Preview | text | no | empty | stored, not translated | yes | no | Catchy sentence displayed next to the subject in most inboxes. Rendered as an inline template with post-processing. When empty, the first characters of the body appear instead. |
| `name` | Name | text | yes | derived | related to the owned Campaign Source name, writable | yes | no | Internal name of the mailing, unique across all Campaign Sources. Generated from the subject when not supplied (see section 18.2). |
| `source_id` | Source | many_to_one to Campaign Source | yes | created on the fly | stored, delete restricted | no | no | The Campaign Source that carries the mailing name and attributes clicks and created records to this mailing. |
| `email_from` | Email from | text | conditional | derived | stored, derived with manual override, pre-computed | yes | no | Sender address. Required when `mailing_type` is `mail` (database check). Derivation in section 1.5. |
| `reply_to_mode` | Reply to mode | selection: `update`, `new` | no | derived | stored, derived with manual override | yes | no | `update` = "Recipient Followers": answers go back to the recipient record's discussion thread. `new` = "Specified Email Address": answers are routed to `reply_to`. |
| `reply_to` | Reply to | text | conditional | derived | stored, derived with manual override | yes | no | Preferred answer address. Required in the form when `reply_to_mode` is `new`. Cleared when the mode is `update`. |
| `favorite` | Favorite | boolean | no | false | stored | **no** | yes | Marks the design as a reusable template in the gallery. |
| `favorite_date` | Favorite date | datetime | no | empty | stored, derived | **no** | no | Set to the current moment the first time `favorite` becomes true; cleared when `favorite` becomes false; never overwritten while it stays true. |
| `sent_date` | Sent date | datetime | no | empty | stored | **no** | no | Moment the mailing finished sending. |
| `schedule_type` | Schedule type | selection: `now`, `scheduled` | yes | `now` | stored | yes | no | `now` = "Send now"; `scheduled` = "Send on". |
| `schedule_date` | Schedule date | datetime | no | derived | stored, derived with manual override | yes | yes | Moment at which the queue may pick the mailing up. Forced empty when `schedule_type` is `now`. |
| `calendar_date` | Calendar date | datetime | no | derived | stored, derived | **no** | no | The moment shown in the calendar view. Derivation in section 1.5. |
| `body_arch` | Body as designed | rich_text | no | empty | stored, not translated, sanitised for outgoing mail | yes | no | The editable body as built in the designer, including the building-block markup. |
| `body_html` | Body rich text | rich_text | no | empty | stored, sanitised for outgoing mail, rendered with the template engine | yes | no | The same body converted to the inline form actually sent. This is the field rendered per recipient. |
| `is_body_empty` | Is body empty | boolean | no | derived | derived, not stored | n/a | no | True when `body_arch` has no visible content. |
| `attachment_ids` | Attachments | many_to_many to Attachment | no | empty | stored, access-check bypassed on search | yes | no | Files attached to every outgoing message. On create and on write the attachments are re-owned by the mailing. |
| `keep_archives` | Keep archives | boolean | no | false | stored | yes | no | When true the outgoing mail records are kept after sending instead of being deleted. Default true for text-message mailings. |
| `campaign_id` | Campaign | many_to_one to Campaign | no | empty | stored, indexed, delete sets empty | yes | no | The marketing effort this mailing belongs to. Visible only to the campaign-management group. |
| `medium_id` | Medium | many_to_one to Campaign Medium | yes in the form | derived | stored, derived with manual override, delete restricted | yes | no | Delivery method. Derivation in section 1.5. |
| `state` | State | selection: `draft`, `in_queue`, `sending`, `done` | yes | `draft` | stored | **no** | yes | Life cycle; see the state table in [workflows.md](workflows.md). Empty groups are shown in the board. |
| `color` | Color index | integer | no | empty | stored | yes | no | Card color in the board. |
| `user_id` | User | many_to_one to User | no | the current user | stored | yes | yes | The person responsible; receives the statistics message; their language and time zone are used when sending. |
| `mailing_type` | Mailing type | selection: `mail`, `sms` | yes | `mail` | stored | yes | no | The channel. `sms` exists only when the Text Message Marketing capability package is installed; removing that package resets the value to the default. |
| `mailing_type_description` | Mailing type description | text | no | derived | derived, not stored | n/a | no | The label of the current `mailing_type`. |
| `mailing_model_id` | Mailing model | many_to_one to Model Definition | yes | the Mailing List entity | stored, delete cascades | yes | no | The entity the recipients are chosen from. Restricted to entities whose `is_mailing_enabled` flag is true. |
| `mailing_model_name` | Mailing model name | text | no | derived | related to the entity's technical name, read-only, read with elevated rights | yes | no | Used by conditions and by the on-change rules. |
| `mailing_model_real` | Mailing model real | text | no | derived | derived, not stored | n/a | no | The entity actually addressed: the Mailing Contact entity when `mailing_model_id` is the Mailing List entity, otherwise `mailing_model_name` itself. |
| `mailing_on_mailing_list` | Mailing on mailing list | boolean | no | derived | derived, not stored | n/a | no | True when `mailing_model_id` is the Mailing List entity. Drives which recipient controls the form shows. |
| `contact_list_ids` | Contact lists | many_to_many to Mailing List | no | empty | stored | **yes, explicitly re-copied on duplication** | no | The lists whose contacts are addressed. Only meaningful when `mailing_on_mailing_list` is true. |
| `mailing_domain` | Mailing domain | text (condition expression) | no | derived | stored, derived with manual override, evaluated without elevated rights | yes | no | The stored recipient condition. Derivation in section 1.5. |
| `mailing_filter_id` | Mailing filter | many_to_one to Mailing Filter | no | derived | stored, derived with manual override | yes | no | The saved condition currently loaded. Choices are limited to filters of the same recipient entity. |
| `mailing_filter_domain` | Mailing filter domain | text | no | derived | related to the filter's condition | yes | no | Shown to compare the stored condition with the saved one. |
| `mailing_filter_count` | Mailing filter count | integer | no | derived | derived, not stored | n/a | no | Number of saved filters available for the current recipient entity. |
| `mail_server_available` | Mail server available | boolean | no | derived | derived, not stored | n/a | no | True when the "Dedicated Server" setting is enabled. Controls visibility of `mail_server_id`. |
| `mail_server_id` | Mail server | many_to_one to Outgoing Mail Server | no | the configured dedicated server | stored, partially indexed | yes | no | Server used in priority. When empty the ordinary server selection applies, except that servers owned by a person are excluded. |
| `use_exclusion_list` | Use exclusion list | boolean | no | true | stored | **no** | no | When false the globally blocked addresses are not filtered out. The form hides the control when no dedicated server is set and the value is true. |
| `warning_message` | Warning message | text | no | derived | derived, not stored | n/a | no | Non-blocking warning shown on the form; see section 1.5. |
| `ab_testing_enabled` | Comparison testing enabled | boolean | no | false | stored | yes | no | Turns the mailing into one version of a comparison test. |
| `ab_testing_pc` | Comparison test percentage | integer | yes | 10 | stored | yes | no | Percentage of the audience this version is sent to. Database check: between 0 and 100 inclusive. |
| `ab_testing_winner_selection` | Comparison test winner criterion | selection: `manual`, `opened_ratio`, `clicks_ratio`, `replied_ratio` (plus `crm_lead_count`, `sale_quotation_count`, `sale_invoiced_amount` with the matching capability packages) | no | `opened_ratio` | related to the Campaign, writable | **yes, explicitly** | no | Criterion used to pick the winning version. |
| `ab_testing_schedule_datetime` | Comparison test decision moment | datetime | no | current moment plus one day | related to the Campaign, writable | **yes, explicitly re-copied on duplication** | no | Moment at which the winner is picked and sent automatically. |
| `ab_testing_completed` | Comparison test completed | boolean | no | derived | related to the Campaign, read-only | no | no | True once a winner has been chosen for the campaign. |
| `ab_testing_is_winner_mailing` | Is the winning version of its campaign | boolean | no | derived | derived, not stored | n/a | no | True when this mailing is the campaign's recorded winner. |
| `ab_testing_mailings_count` | Comparison test electronic mail version count | integer | no | derived | related to the Campaign | n/a | no | Number of electronic-mail mailings of the campaign. |
| `is_ab_test_sent` | Is comparison test sent | boolean | no | derived | derived, not stored | n/a | no | True when at least one sibling version of the campaign has reached `done`. |
| `ab_testing_description` | Comparison test description | rich_text | no | derived | derived, not stored | n/a | no | Human sentence describing the test: number of versions, total percentage covered, criterion, moment. Empty when `ab_testing_enabled` is false. |
| `kpi_mail_required` | Statistics message required | boolean | no | false | stored | **no** | no | Set to true when the mailing finishes its first sending; consumed by the statistics job. |
| `mailing_trace_ids` | Mailing traces | one_to_many to Mailing Trace | no | empty | stored inverse | no | no | One record per addressed recipient record. |
| `total` | Total | integer | no | derived | derived, not stored | n/a | no | Size of the audience, reduced to the test percentage when a comparison test is running. |
| `expected` | Expected | integer | no | derived | derived, not stored | n/a | no | Number of delivery records of any status. |
| `scheduled` | Scheduled | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `outgoing`. |
| `process` | Process | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `process`. |
| `pending` | Pending | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `pending` (text messages handed to the service, delivery report not yet received). |
| `sent` | Sent | integer | no | derived | derived, not stored | n/a | no | Delivery records whose sent moment is filled. |
| `delivered` | Delivered | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `sent`, `open` or `reply`. |
| `opened` | Opened | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `open` or `reply`. |
| `clicked` | Clicked | integer | no | derived | derived, not stored | n/a | no | Delivery records whose last-click moment is filled. |
| `replied` | Replied | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `reply`. |
| `bounced` | Bounced | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `bounce`. |
| `failed` | Failed | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `error`. |
| `canceled` | Canceled | integer | no | derived | derived, not stored | n/a | no | Delivery records with status `cancel`. |
| `received_ratio` | Received ratio | decimal (2 places) | no | derived | derived, not stored | n/a | no | See [calculations.md](calculations.md) section 1. |
| `opened_ratio` | Opened ratio | decimal (2 places) | no | derived | derived, not stored | n/a | no | See [calculations.md](calculations.md) section 1. |
| `replied_ratio` | Replied ratio | decimal (2 places) | no | derived | derived, not stored | n/a | no | See [calculations.md](calculations.md) section 1. |
| `bounced_ratio` | Bounced ratio | decimal (2 places) | no | derived | derived, not stored | n/a | no | See [calculations.md](calculations.md) section 1. |
| `clicks_ratio` | Clicks ratio | decimal (2 places) | no | derived | derived, not stored | n/a | no | See [calculations.md](calculations.md) section 2. |
| `link_trackers_count` | Link trackers count | integer | no | derived | derived, not stored | n/a | no | Number of Link Tracker records created for this mailing. |
| `next_departure` | Next departure | datetime | no | derived | derived, not stored | n/a | no | The later of `schedule_date` and the current moment; the current moment when no schedule date is set. |
| `next_departure_is_past` | Next departure is past | boolean | no | derived | derived, not stored | n/a | no | True when the state is `in_queue` and `next_departure` is already past; drives the "will be sent as soon as possible" warning and its refresh button. |

### 1.2 Fields added by the Text Message Marketing capability package

| Field | Full name | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|---|
| `sms_subject` | Text message title | text | no | derived | related to `subject`, writable, not translated | The same value as `subject`, labelled "Title", because for a text message the subject is an internal name only. |
| `body_plaintext` | Body as plain text | long_text | no | derived | stored, derived with manual override | The text-message body. Set from the chosen text-message template when one is selected and the type is `sms`. |
| `sms_template_id` | Text message template | many_to_one to Text Message Template | no | empty | stored, delete sets empty | Template whose body seeds `body_plaintext`. |
| `sms_force_send` | Text message force send | boolean | no | false | stored | Sends immediately instead of queueing. |
| `sms_allow_unsubscribe` | Text message allow unsubscribe | boolean | no | false | stored | Appends the opt-out sentence and link to each message and creates the matching trace code. |
| `sms_has_insufficient_credit` | Text message has insufficient credit | boolean | no | derived | derived, not stored | True when at least one trace of this mailing failed with `sms_credit`. |
| `sms_has_unregistered_account` | Text message has unregistered account | boolean | no | derived | derived, not stored | True when at least one trace failed with `sms_acc`. |
| `ab_testing_sms_winner_selection` | Comparison test text message winner criterion | selection: `manual`, `clicks_ratio` (plus `crm_lead_count`, `sale_quotation_count`, `sale_invoiced_amount` with the matching capability packages) | no | `clicks_ratio` | related to the Campaign, writable, copied | Winner criterion used when the mailing type is `sms`. |
| `ab_testing_mailings_sms_count` | Comparison test text message version count | integer | no | derived | related to the Campaign | Number of text-message mailings of the campaign. |

### 1.3 Fields added by other capability packages

| Field | Full name | Package | Type | Derivation |
|---|---|---|---|---|
| `card_campaign_id` | Card campaign | Marketing Card | many_to_one to Marketing Card Campaign, partially indexed | When set, the mailing sends personalised images; it forces `mailing_model_id` to the campaign's target entity. |
| `card_requires_sync_count` | Card requires sync count | Marketing Card | integer, derived | Number of recipients of the current condition that have no up-to-date card. Computed only while the state is `draft`; zero otherwise. |
| `use_leads` | Use leads | Lead bridge | boolean, derived | True when the current user may use leads rather than only opportunities; decides the wording of the lead button. |
| `crm_lead_count` | Lead and opportunity count | Lead bridge | integer, derived | Number of leads and opportunities, including archived ones, whose source is this mailing's source. |
| `sale_quotation_count` | Quotation count attributed to this mailing | Sales bridge | integer, derived | Number of sales orders with at least one line whose source is this mailing's source. |
| `sale_invoiced_amount` | Invoiced amount attributed to this mailing | Sales bridge | integer, derived | Sum of the untaxed signed amounts of the customer invoices whose source is this mailing's source and whose state is neither draft nor cancelled. |

### 1.4 Constraints, indexes and validations

| Kind | Statement | Message |
|---|---|---|
| Database check | `ab_testing_pc >= 0 AND ab_testing_pc <= 100` | *"The A/B Testing Percentage needs to be between 0 and 100%"* |
| Database check | `email_from IS NOT NULL OR mailing_type != 'mail'` | *"email from is required for mailing"* |
| Index | `campaign_id` indexed; `mail_server_id` and `card_campaign_id` partially indexed (only non-empty values) | — |
| Validation on (`mailing_model_id`, `mailing_filter_id`) | The saved filter must target the same recipient entity as the mailing. | *"The saved filter targets different recipients and is incompatible with this mailing."* |
| Validation on (`card_campaign_id`, `mailing_domain`, `mailing_model_id`) | When a marketing card campaign is set, the recipient entity must be the campaign's target entity. | *"Card Campaign Mailing should target model %(model_name)s"* where the placeholder is the display name of the required entity. |
| Validation on write | Removing the campaign while comparison testing stays enabled is refused. | *"A campaign should be set when A/B test is enabled"* |
| Validation on send | Sending with an empty audience is refused. | *"There are no recipients selected."* |
| Validation on queueing and sending | With a marketing card campaign, every recipient must have an up-to-date card. | *"You should update all the cards for %(mailing)s before scheduling a mailing."* |
| Validation on winner sending | All selected mailings must share one campaign. | *"To send the winner mailing the same campaign should be used by the mailings"* |
| Validation on winner sending | The campaign must not already be completed. | *"To send the winner mailing the campaign should not have been completed."* |
| Validation on winner sending | At least one version must have been sent. | *"No mailing for this A/B testing campaign has been sent yet! Send one first and try again later."* |
| Validation on winner selection | Comparison testing must be enabled on the mailing being promoted. | *"A/B test option has not been enabled"* |
| Validation on version comparison | A campaign must exist. | *"No mailing campaign has been found"* |

### 1.5 Derivation rules

**`email_from`.** Inputs: the creating user, the selected `mail_server_id`, the system notification address.

```
notification_address = default sender address of the system
user_address         = formatted address of created_by_user, or of the current user when absent
```

The five cases are evaluated in this order and the first one that applies decides the value.

| Order | Case | Resulting `email_from` |
|---|---|---|
| 1 | `mail_server_id` is empty | the value already stored when there is one, otherwise `user_address` |
| 2 | `email_from` is set and matches the sender filter of `mail_server_id` | the value already stored, unchanged |
| 3 | `user_address` matches the sender filter of `mail_server_id` | `user_address` |
| 4 | `notification_address` matches the sender filter of `mail_server_id` | `notification_address` |
| 5 | none of the above | the value already stored when there is one, otherwise `user_address` |

**`warning_message`.** For mailings of type `mail` only: when a server is selected and `email_from` does not match its sender filter, the message is *"This email from can not be used with this mail server."* followed by a line break and *"Your emails might be marked as spam on the mail clients."* Otherwise empty.

**`medium_id`.** When the type is `mail` and no medium is set, the medium named `Email` is fetched or created. When the Text Message Marketing package is installed: for type `sms`, if the medium is empty or still the `Email` medium, the medium named `Text Message` is fetched or created; for type `mail`, if the medium is empty or still the `Text Message` medium, the `Email` medium is used.

**`reply_to_mode`.** `new` when the recipient entity is Contact, Mailing List or Mailing Contact; `update` for every other entity.

**`reply_to`.** When the mode is `new` and the field is empty, the formatted address of the current user. When the mode is `update`, empty.

**`mailing_model_real`.** The Mailing Contact entity when `mailing_model_id` is the Mailing List entity, otherwise the technical name of `mailing_model_id`.

**`mailing_domain`.**

```
if mailing_model is empty:            mailing_domain = ""
else if mailing_filter is set:        mailing_domain = mailing_filter.mailing_domain
else:                                 mailing_domain = default condition of the recipient entity
```

The default condition of an entity is the condition that entity publishes for mailings, or the always-true condition when it publishes none. Known default conditions: Mailing List gives `list_ids IN contact_lists`; Sales Order gives `state != "cancel"`; Event Registration gives `state NOT IN ("cancel", "draft")` unless a prepared condition was supplied by the calling screen; Event Track gives `stage.is_cancel = false`.

**`mailing_filter_id`.** Recomputed to empty whenever `mailing_model_name` changes. The stored `mailing_domain` is *not* cleared when the filter record is deleted: the condition survives its filter.

**`schedule_date`.** Forced empty when `schedule_type` is `now` or when no date is set.

**`calendar_date`.**

```
state = "done"      -> calendar_date = sent_date
state = "in_queue"  -> calendar_date = next_departure
state = "sending"   -> calendar_date = current moment
state = "draft"     -> calendar_date = empty
```

**`ab_testing_description`.** Rendered from: the mailing, the number of versions (`ab_testing_mailings_count`, or `ab_testing_mailings_sms_count` for text messages), the label of the winner criterion, and the sum of `ab_testing_pc` over all comparison-test versions of the campaign.

### 1.6 On-change behavior in the form

| The user changes | The system recomputes |
|---|---|
| `mailing_model_id` | `mailing_model_name`, `mailing_model_real`, `mailing_on_mailing_list`, `mailing_filter_id` (cleared), `mailing_domain` (reset to the entity default), `reply_to_mode` and therefore `reply_to`, `mailing_filter_count`. With a marketing card campaign, `mailing_model_id` itself is forced back to the campaign's entity. |
| `contact_list_ids` | `mailing_domain` (when no filter is loaded). |
| `mailing_filter_id` | `mailing_domain` (replaced by the filter's condition); the compatibility validation runs. |
| `mailing_domain` | `mailing_filter_count`; the loaded filter is **not** cleared. |
| `mailing_type` | `medium_id`, `mailing_type_description`, `body_plaintext`, `ab_testing_description`. |
| `schedule_type` | `schedule_date` (emptied when `now`), then `calendar_date` and `next_departure`. |
| `reply_to_mode` | `reply_to`. |
| `mail_server_id` or `email_from` | `email_from` (re-derived), `warning_message`. |
| `favorite` | `favorite_date`. |
| `body_arch` | `is_body_empty`. |
| `sms_template_id` | `body_plaintext`. |
| `ab_testing_enabled`, `ab_testing_pc`, `ab_testing_schedule_datetime`, `ab_testing_winner_selection`, `campaign_id` | `ab_testing_description`. |
| Opening the calendar view on a future day and creating there | `schedule_type` becomes `scheduled` and `schedule_date` becomes that day, provided the day is in the future. |

### 1.7 Creation, write, duplication and deletion

**On creation.**
1. When a comparison-test moment is supplied, the comparison-test job is asked to wake at that moment.
2. When `mailing_type` is `sms` and a title was supplied, the title is copied into `subject` before the name is generated.
3. The record is created; the owned Campaign Source is created first and its name is made unique (section 18.2). A `name` supplied only through screen defaults is ignored, in order to avoid a uniqueness violation.
4. For every mailing with comparison testing enabled and no campaign, a campaign is created (section 1.8).
5. Attachments are re-owned by the mailing.
6. Inline images inside `body_arch` and `body_html` are converted to stored files and replaced by addresses (see [calculations.md](calculations.md) section 12).
7. When `mailing_type` is `sms`, the archive-keeping flag defaults to true.

**On write.**
1. Inline images in the written bodies are converted as above.
2. Clearing the campaign while comparison testing stays enabled is refused.
3. The write happens.
4. When comparison testing has just been enabled, a campaign is created for every mailing that has none.
5. Attachments are re-owned.
6. When any of the written records has a comparison-test moment, the comparison-test job is asked to wake at the earliest such moment.

**On duplication.** The copy keeps the subject, bodies, condition, lists and settings. It does not keep `favorite`, `favorite_date`, `sent_date`, `state` (restarts at `draft`), `calendar_date`, `kpi_mail_required`, `use_exclusion_list`, the traces or the campaign's winner marker. It explicitly re-copies `contact_list_ids`, and, when comparison testing is enabled, `ab_testing_schedule_datetime`. When the selected mail server is archived, the copy falls back to the configured dedicated server. A new Campaign Source is created with an incremented counter.

**On deletion.** Traces cascade (they are deleted with the mailing). Link Tracker records survive with an empty mailing reference. Outgoing mail records survive unless deleted by their own life cycle.

---

## 2. Mailing List

**Transport name** `mailing.list` · **storage name** `mailing_list` · **generated reference page** [`mailing.list`](../../references/entities/mailing.list.md)

A named audience. Membership is carried by Mailing Subscription records, therefore a contact can be a member of a list and opted out of it at the same time.

- **Identity.** No natural key; duplicate names are allowed.
- **Default ordering.** `created_on` descending.
- **Display name.** `"<name> (<contact_count>)"`, for example `Newsletter (1204)`.
- **Company scoping.** None.
- **Archiving.** Supported, with a guard (section 2.3).
- **Duplication.** The copy is named `"<name> (copy)"`. Subscriptions are copied; the direct member association is not.
- **Mailing enabled.** Yes: the Mailing List entity may be chosen as a mailing's recipient entity, in which case the mailing actually addresses Mailing Contacts.
- **Generic record merging is disabled** for this entity because it has its own merge assistant (section 25).

### 2.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored | yes | The list name. |
| `active` | Active | boolean | no | true | stored | yes | Archiving flag. |
| `is_public` | Is public | boolean | no | false | stored | yes | When true the list is offered on the public subscription-management page and its name may be shown in unsubscribe confirmations. |
| `contact_ids` | Contacts | many_to_many to Mailing Contact through the subscription table | no | empty | stored | **no** | Direct member view; writing it creates subscriptions. |
| `subscription_ids` | Subscriptions | one_to_many to Mailing Subscription | no | empty | stored inverse | **yes** | The memberships, each with its own opt-out data. |
| `mailing_ids` | Mailings | many_to_many to Mass Mailing | no | empty | stored | **no** | Mailings that address this list. |
| `mailing_count` | Mailing count | integer | no | derived | derived, not stored | n/a | Number of mailings that address this list. |
| `contact_count` | Contact count | integer | no | derived | derived, not stored | n/a | Number of subscriptions. |
| `contact_count_email` | Contact count email | integer | no | derived | derived, not stored | n/a | Members that have a normalised address, are not opted out of this list, and are not globally blocked. |
| `contact_count_opt_out` | Contact count opt out | integer | no | derived | derived, not stored | n/a | Members opted out of this list. |
| `contact_count_blacklisted` | Contact count blacklisted | integer | no | derived | derived, not stored | n/a | Members whose address is globally blocked (with the Text Message Marketing package: or whose sanitised number is globally blocked). |
| `contact_pct_opt_out` | Contact percentage opt out | decimal | no | derived | derived, not stored | n/a | See [calculations.md](calculations.md) section 5. |
| `contact_pct_blacklisted` | Contact percentage blacklisted | decimal | no | derived | derived, not stored | n/a | See [calculations.md](calculations.md) section 5. |
| `contact_pct_bounce` | Contact percentage bounce | decimal | no | derived | derived, not stored | n/a | See [calculations.md](calculations.md) section 5. |
| `contact_count_sms` | Contact count text message | integer | no | derived | derived, not stored | n/a | Added by the Text Message Marketing package: members with a sanitised number, not opted out of this list, not globally blocked for telephone. |

### 2.2 Statistics computation

All counters are produced by one grouped read over the subscription table joined to Mailing Contact, to the blocked-address register and (with the Text Message Marketing package) to the blocked-number register, grouped by list. Lists with no subscription get zeros. The bounce percentage uses a second grouped read counting distinct contacts with a bounce counter greater than zero. Before computing, all pending writes are flushed, because the normalised address of a contact must be up to date.

### 2.3 Validations

| Rule | Message |
|---|---|
| Archiving a list that is referenced by at least one mailing whose state is not `done` is refused. | *"At least one of the mailing list you are trying to archive is used in an ongoing mailing campaign."* |

### 2.4 Life cycle

A list is created, filled (manually, by import, by the add-to-list assistant, by a website subscription or by a merge), used by mailings, and finally archived. It is never automatically deleted. Deleting a list cascades to its subscriptions.

---

## 3. Mailing Contact

**Transport name** `mailing.contact` · **storage name** `mailing_contact` · **generated reference page** [`mailing.contact`](../../references/entities/mailing.contact.md)

A lightweight addressee, kept separate from the general Contact entity in order to hold very large audiences cheaply.

- **Identity.** No unique constraint on the address: two contacts may share one address, deliberately, because the same person may be an opted-in member of one list and an opted-out member of another.
- **Default ordering.** `name` ascending, then `identifier` descending.
- **Display name.** The name.
- **Discussion thread.** Yes, with the blocked-address mixin: the contact carries `email_normalized`, `is_blacklisted` and `message_bounce` from that mixin.
- **User-defined fields.** Supported through the shared property-definition mechanism; the definition is owned by the marketing user group.
- **Mailing enabled.** Yes.

### 3.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Tracked | Meaning |
|---|---|---|---|---|---|---|---|
| `name` | Name | text | no | derived | stored, derived with manual override | yes | The display name. Derived as `first_name` and `last_name` joined by a space whenever either is set. |
| `first_name` | First name | text | no | empty | stored | no | Only usable when the split-name setting is on; otherwise it is not searchable. |
| `last_name` | Last name | text | no | empty | stored | no | Same. |
| `company_name` | Company name | text | no | empty | stored | no | Free text, not a link to a Contact. |
| `email` | Email | text | no | empty | stored | no | Raw address. The normalised form comes from the blocked-address mixin. |
| `mobile` | Mobile phone | text | no | empty | stored | no | Added by the Text Message Marketing package, together with the telephone mixin that computes the sanitised number and the blocked-number flag. |
| `country_id` | Country | many_to_one to Country | no | empty | stored | no | Used for number sanitising and for segmentation. |
| `tag_ids` | Tags | many_to_many to Contact Category | no | empty | stored | no | Reuses the contact tag catalogue. |
| `list_ids` | Lists | many_to_many to Mailing List through the subscription table | no | empty | stored | no | Membership view. |
| `subscription_ids` | Subscriptions | one_to_many to Mailing Subscription | no | empty | stored inverse | no | Memberships with their opt-out data. |
| `opt_out` | Opt out | boolean | no | derived | derived, not stored, context-dependent | no | True when the contact is opted out **of the single list given by the screen context**. False in every other situation. Searching on it is only meaningful in that same context. |

### 3.2 Context-dependent opt-out

`opt_out` and its search rule are only defined when exactly one target list is present in the screen context. Outside that situation the derived value is false and a search on it returns nothing. This is why the list column showing it is hidden unless the screen was opened from a list.

### 3.3 Creation rules

- Supplying both the membership view and the subscription list in the same creation values is refused with *"You should give either list_ids, either subscription_ids to create new contacts."*
- When the screen supplies a default set of lists and the creation values do not set the membership view, one subscription is added for every default list that is not already present in the supplied subscriptions. The default set is then cleared for the actual creation, and the membership caches are invalidated.
- Duplicating a contact from inside a list screen does not add the screen's default lists again: the copied subscriptions already carry them.
- Creating a contact from a typed string parses it as `"Name" <address>`: the name part becomes `name`, the address part becomes `email`. The same parsing is used by the add-to-list shortcut, which additionally links the given list.

### 3.4 Life cycle

Created manually, by import, by a website subscription or by a merge. Never archived by the system (the entity has no archiving flag of its own; the blocked-address mixin provides the global blocking instead). Deleting a contact cascades to its subscriptions.

---

## 4. Mailing Subscription

**Transport name** `mailing.subscription` · **storage name** `mailing_subscription` · **generated reference page** [`mailing.subscription`](../../references/entities/mailing.subscription.md)

The membership of one contact in one list, and the only place where opting out is recorded.

- **Table.** A dedicated association table that also serves as the many-to-many table behind `Mailing List.contacts` and `Mailing Contact.lists`.
- **Default ordering.** `list_id` descending, then `contact_id` descending.
- **Display name.** The contact.

| Field | Full name | Type | Required | Default | Stored / derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `contact_id` | Contact | many_to_one to Mailing Contact | yes | empty | stored, delete cascades | yes | The member. |
| `list_id` | List | many_to_one to Mailing List | yes | empty | stored, indexed, delete cascades | yes | The list. |
| `opt_out` | Opt out | boolean | no | false | stored | yes | True when the contact refused further messages from this list. |
| `opt_out_reason_id` | Opt out reason | many_to_one to Mailing Opt-Out Reason | no | empty | stored, delete restricted | yes | Why they left. |
| `opt_out_datetime` | Opt out datetime | datetime | no | derived | stored, derived with manual override | yes | Moment of the opt-out; cleared when `opt_out` returns to false. |
| `message_bounce` | Message bounce | integer | no | derived | related to the contact, not stored, writable | n/a | Bounce counter of the contact, shown in the subscription list. |
| `is_blacklisted` | Is blacklisted | boolean | no | derived | related to the contact, not stored, writable | n/a | Global blocking flag of the contact. |

**Constraint.** Unique (`contact_id`, `list_id`). Message: *"A mailing contact cannot subscribe to the same mailing list multiple times."*

**Derivation of `opt_out_datetime`.** When `opt_out` is false the moment is cleared. When `opt_out` is true and the moment is empty, it is set to the current database moment.

**Creation and write coupling.** Writing either `opt_out_datetime` or `opt_out_reason_id` forces `opt_out` to true, on creation and on write. Therefore recording a reason is by itself an opt-out.

**Navigation.** From one or many subscriptions the user can open the underlying contacts; with a single subscription the contact form opens directly.

---

## 5. Mailing Opt-Out Reason

**Transport name** `mailing.subscription.optout` · **storage name** `mailing_subscription_optout` · **generated reference page** [`mailing.subscription.optout`](../../references/entities/mailing.subscription.optout.md)

A selectable reason, shown as radio buttons on the public subscription page and on the self-exclusion flow.

- **Default ordering.** `sequence` ascending, then `created_on` descending, then `identifier` descending.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | no | empty | stored, translated | The sentence shown to the recipient. |
| `sequence` | Sequence | integer | no | 10 | stored | Display order. |
| `is_feedback` | Is feedback | boolean | no | false | stored | When true, choosing this reason reveals a free-text box whose content is logged. |

Reasons are also referenced by the blocked-address record of the messaging domain, which gains an `opt_out_reason_id` field from this domain.

---

## 6. Mailing Trace

**Transport name** `mailing.trace` · **storage name** `mailing_trace` · **generated reference page** [`mailing.trace`](../../references/entities/mailing.trace.md)

The delivery record of one message to one recipient record. Traces are kept in their own table in order that outgoing mail can be deleted without losing the measurement.

- **Default ordering.** `created_on` descending.
- **Display name.** `"<trace_type>: <mailing name> (<identifier>)"`.
- **No archiving.**

### 6.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `trace_type` | Trace type | selection: `mail`, `sms` | yes | `mail` | stored | yes | Channel of this delivery. The `sms` value comes with the Text Message Marketing package and falls back to the default if that package is removed. |
| `is_test_trace` | Is test trace | boolean | no | false | stored | yes | True for traces created by a test send, in order to exclude them from the real measurement screens. |
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing | no | empty | stored, indexed, delete cascades | yes | The mailing. |
| `campaign_id` | Campaign | many_to_one to Campaign | no | derived | related to the mailing, stored, read-only, partially indexed | yes | Copied at creation for fast per-campaign queries and for already-contacted detection. |
| `medium_id` | Medium | many_to_one to Campaign Medium | no | derived | related to the mailing, not stored | n/a | Convenience. |
| `source_id` | Source | many_to_one to Campaign Source | no | derived | related to the mailing, not stored | n/a | Convenience. |
| `model` | Model name | text | yes | empty | stored | yes | Technical name of the recipient entity. |
| `res_id` | Related record identifier | integer reference to a record of `model` | yes in practice | empty | stored | yes | The recipient record. |
| `email` | Email | text | no | empty | stored | yes | Normalised recipient address, kept even when the outgoing mail is deleted. |
| `message_id` | Message | text | no | empty | stored | yes | The message key of the outgoing electronic mail, used to match answers and bounces. |
| `mail_mail_id` | Mail mail | many_to_one to Outgoing Email | no | empty | stored, partially indexed | yes | The outgoing mail, while it exists. |
| `mail_mail_id_int` | Outgoing mail identifier as a plain integer | integer | no | derived at creation | stored, partially indexed | yes | A plain integer copy of the outgoing mail key, kept after that record is deleted; used by the open-tracking endpoint. |
| `sent_datetime` | Sent datetime | datetime | no | empty | stored | yes | Moment the message was handed over successfully. |
| `open_datetime` | Open datetime | datetime | no | empty | stored | yes | First open. |
| `reply_datetime` | Reply datetime | datetime | no | empty | stored | yes | First answer. |
| `links_click_datetime` | Links click datetime | datetime | no | empty | stored | yes | **Last** click; overwritten on every further click. |
| `links_click_ids` | Links clicks | one_to_many to Link Tracker Click | no | empty | stored inverse | no | Every recorded click of this recipient. |
| `trace_status` | Trace status | selection of nine values (section 6.2) | no | `outgoing` | stored | yes | The delivery status. |
| `failure_type` | Failure type | selection (section 6.3) | no | empty | stored | yes | Why the delivery did not succeed. |
| `failure_reason` | Failure reason | long_text | no | empty | stored, read-only | **no** | Free text captured from the bounce message or the provider answer. |

### 6.2 Fields added by the Text Message Marketing capability package

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `sms_id` | Text message | many_to_one to Outgoing Text Message | derived, not stored | Resolved from `sms_id_int` when the outgoing record still exists and is not marked for deletion. |
| `sms_id_int` | Outgoing text message identifier as a plain integer | integer | stored, partially indexed | Plain integer copy of the outgoing text-message key, kept after deletion. |
| `sms_tracker_ids` | Text message trackers | one_to_many to Text Message Tracker | stored inverse | The provider-side trackers that will report delivery. |
| `sms_number` | Text message number | text | stored | The sanitised destination number. |
| `sms_code` | Text message code | text | stored | A three-character random code, generated at creation when absent, that obfuscates the opt-out link. Uniqueness is not required: the triple (code, mailing, number) is what is verified. |

### 6.3 Statuses

| Value | Label | Meaning |
|---|---|---|
| `outgoing` | Outgoing | Created, not yet handed to the transport. |
| `process` | Processing | Text message accepted by the service and held before actual sending. |
| `pending` | Sent | Text message sent, delivery report not yet received. |
| `sent` | Delivered | Electronic mail handed over to the relay without error, or text message confirmed delivered. |
| `open` | Opened | The tracking image was fetched, a link was clicked, or an answer arrived. |
| `reply` | Replied | An answer arrived through the incoming gateway. |
| `bounce` | Bounced | A bounce notification arrived, or the number format was rejected. |
| `error` | Exception | Sending failed with a technical failure type. |
| `cancel` | Cancelled | The message was suppressed before sending (blocked, opted out, duplicate, invalid address when archives are not kept). |

Note the two deliberately confusing labels: the stored value `pending` is displayed as "Sent" and the stored value `sent` is displayed as "Delivered". Percentage formulas use the stored values.

### 6.4 Failure types

| Value | Label | Channel |
|---|---|---|
| `unknown` | Unknown error | both |
| `mail_bounce` | Bounce | electronic mail |
| `mail_spam` | Detected As Spam | electronic mail |
| `mail_email_invalid` | Invalid email address | electronic mail |
| `mail_email_missing` | Missing email address | electronic mail |
| `mail_from_invalid` | Invalid from address | electronic mail |
| `mail_from_missing` | Missing from address | electronic mail |
| `mail_smtp` | Connection failed (outgoing mail server problem) | electronic mail |
| `mail_bl` | Blacklisted Address | electronic mail, mass mode only |
| `mail_dup` | Duplicated Email | electronic mail, mass mode only |
| `mail_optout` | Opted Out | electronic mail, mass mode only |
| `sms_number_missing` | Missing Number | text message |
| `sms_number_format` | Wrong Number Format | text message |
| `sms_credit` | Insufficient Credit | text message |
| `sms_country_not_supported` | Country Not Supported | text message |
| `sms_registration_needed` | Country-specific Registration Required | text message |
| `sms_server` | Server Error | text message |
| `sms_acc` | Unregistered Account | text message |
| `sms_blacklist` | Blacklisted | text message, mass mode only |
| `sms_duplicate` | Duplicate | text message, mass mode only |
| `sms_optout` | Opted Out | text message, mass mode only |
| `sms_expired` | Expired | text message, delivery report |
| `sms_invalid_destination` | Invalid Destination | text message, delivery report |
| `sms_not_allowed` | Not Allowed | text message, delivery report |
| `sms_not_delivered` | Not Delivered | text message, delivery report |
| `sms_rejected` | Rejected | text message, delivery report |
| `twilio_authentication` | Authentication Error" | text message, Twilio telephony provider |
| `twilio_callback` | Incorrect callback URL | text message, Twilio telephony provider |
| `twilio_from_missing` | Missing From Number | text message, Twilio telephony provider |
| `twilio_from_to` | From / To identic | text message, Twilio telephony provider |

Twilio is a third-party telephony service that can replace the built-in credit-based text-message service.

**Compatibility finding.** The label of `twilio_authentication` ends with a stray double quotation
mark: the text stored and displayed is exactly `Authentication Error"`. It is reproduced here
because it is what the screen shows today and what an automated test asserts. A corrected behaviour
would display "Authentication Error" without the trailing mark; a rebuild that corrects it must
expect any test that compares the label literally to need updating.

**Industry-standard default.** Three failure types have no observable difference in the screens:
`sms_server`, `unknown` and, for a delivery report that names no reason, `sms_not_delivered`. A
rebuild that has to choose one for an unclassified provider answer should store `unknown` and put
the provider's own wording into the failure reason, which is the resolution used everywhere else in
this domain.

### 6.5 Constraints

| Kind | Statement | Message |
|---|---|---|
| Database check | `related_record_identifier IS NOT NULL AND related_record_identifier != 0` | *"Traces have to be linked to records with a not null res_id."* |
| Index | `mass_mailing_id` indexed; `campaign_id`, `mail_mail_id`, `mail_mail_id_int` and `sms_id_int` partially indexed | — |

### 6.6 Status operations

Each of the following is an operation on a set of traces, optionally extended by a search condition. They are the only way statuses change.

| Operation | Effect |
|---|---|
| `set_sent` | `trace_status = "sent"`, `sent_datetime = now`, `failure_type` cleared. |
| `set_opened` | For traces whose status is neither `open` nor `reply`: `trace_status = "open"`, `open_datetime = now`. Traces already open or replied are left untouched. |
| `set_clicked` | `links_click_datetime = now` (always overwritten). |
| `set_replied` | `trace_status = "reply"`, `reply_datetime = now`. |
| `set_bounced` | `trace_status = "bounce"`, `failure_type = "mail_bounce"`, `failure_reason` = the plain-text bounce body. |
| `set_failed` | `trace_status = "error"`, `failure_type` = the supplied code. |
| `set_canceled` | `trace_status = "cancel"`. |

---

## 7. Mailing Trace Report

**Transport name** `mailing.trace.report` · **storage name** `mailing_trace_report` · **generated reference page** [`mailing.trace.report`](../../references/entities/mailing.trace.report.md)

A read-only analytical view over delivery records, joined to their mailing, campaign and source. It has no table of its own and is rebuilt when the capability package is installed or updated.

- **Grain.** One row per distinct combination of (trace creation moment, source name, campaign name, mailing type, mailing state, sender address). The row key is the smallest trace identifier of the group.

| Field | Full name | Type | Meaning |
|---|---|---|---|
| `name` | Name | text | Name of the mailing, taken from its Campaign Source. |
| `campaign_id` | Campaign | text | Name of the campaign. |
| `mailing_type` | Mailing type | selection: `mail` (plus `sms` with that package) | Channel. |
| `scheduled_date` | Scheduled date | datetime | The creation moment of the traces in the group. |
| `state` | State | selection: `draft`, `test`, `done` | The mailing state as exposed by the report. |
| `email_from` | Email from | text | Sender address of the mailing. |
| `scheduled` | Scheduled | integer | Count of traces in the group. |
| `sent` | Sent | integer | Traces whose sent moment is filled. |
| `delivered` | Delivered | integer | Traces whose status is **not** one of `outgoing`, `pending`, `process`, `error`, `bounce`, `cancel`. |
| `processing` | Processing | integer | Traces with status `process`. |
| `pending` | Pending | integer | Traces with status `pending`. |
| `error` | Error | integer | Traces with status `error`. |
| `bounced` | Bounced | integer | Traces with status `bounce`. |
| `canceled` | Canceled | integer | Traces with status `cancel`. |
| `opened` | Opened | integer | Traces with status `open`. |
| `replied` | Replied | integer | Traces with status `reply`. |
| `clicked` | Clicked | integer | Traces whose last-click moment is filled. |

Note that in this report `opened` counts only the status `open`, whereas the counter on the Mass Mailing counts `open` and `reply` together. A replacement must reproduce both definitions.

---

## 8. Mailing Filter

**Transport name** `mailing.filter` · **storage name** `mailing_filter` · **generated reference page** [`mailing.filter`](../../references/entities/mailing.filter.md)

A named, reusable recipient condition.

- **Default ordering.** `created_on` descending.
- **Display name.** The filter name.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored | Filter name. |
| `mailing_domain` | Mailing domain | text (condition expression) | yes | empty | stored | The condition. |
| `mailing_model_id` | Mailing model | many_to_one to Model Definition | yes | empty | stored, delete cascades | The entity the condition applies to. |
| `mailing_model_name` | Mailing model name | text | no | derived | related | Technical name of that entity. |
| `create_uid` | Created by user | many_to_one to User | yes | the current user | stored, indexed, read-only | Labelled "Saved by"; the default screen filter shows only the current user's filters. |

**Validation on (`mailing_domain`, `mailing_model_id`).** Unless the condition is exactly the empty condition, the system counts the records matching it on the stated entity; if that evaluation fails for any reason the write is refused with *"The filter domain is not valid for this recipients."*

**Deletion.** Deleting a filter clears `mailing_filter_id` on the mailings that referenced it, but leaves their stored `mailing_domain` untouched.

---

## 9. Link Tracker

**Transport name** `link.tracker` · **storage name** `link_tracker` · **generated reference page** [`link.tracker`](../../references/entities/link.tracker.md)

A target address with campaign attribution, reachable through one or more short codes.

- **Default ordering.** `count` descending.
- **Display name.** The short address.
- **Campaign tracking.** Carries `campaign_id`, `source_id` and `medium_id` from the Campaign Tracking Mixin, each with delete-sets-empty behavior.

### 9.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|---|
| `url` | Web address | text | yes | empty | stored | Target address, labelled "Target Web Address". Validated and completed at creation (section 9.3). |
| `absolute_url` | Absolute web address | text | no | derived | derived, not stored | The target address when it already carries a scheme; otherwise the site base address joined with it. |
| `code` | Code | text | no | derived | derived with manual override, not stored | The most recent short code of this tracker. Writing it renames that code. |
| `link_code_ids` | Link codes | one_to_many to Link Tracker Code | no | empty | stored inverse | Every code pointing here. |
| `short_url` | Short web address | text | no | derived | derived, not stored, searchable | `short_url_host` joined with `code`. |
| `short_url_host` | Short web address host | text | no | derived | derived, not stored | The base address of the current website when one is resolved and it belongs to the current company, otherwise the company base address, in both cases followed by `/r/`. Without the website package, the record base address followed by `/r/`. |
| `redirected_url` | Redirected web address | text | no | derived | derived, not stored | The address the visitor is finally sent to, with campaign parameters appended (section 9.4). |
| `title` | Title | text | no | derived at creation | stored | Page title, fetched from the target page's preview metadata at creation when not supplied; otherwise the address itself. |
| `label` | Label | text | no | empty | stored | The clickable text of the link, truncated to 40 characters, captured when the link was shortened. Part of the uniqueness rule. |
| `link_click_ids` | Link clicks | one_to_many to Link Tracker Click | no | empty | stored inverse | Recorded visits. |
| `count` | Count | integer | no | derived | stored, derived | Number of recorded visits. |
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing | no | empty | stored | Added by the Email Marketing package: the mailing whose body contained the link. |

### 9.2 Uniqueness

The combination (`url`, `campaign_id`, `medium_id`, `source_id`, `label`) must be unique. This cannot be a database constraint because empty values must compare equal; it is checked on create and on write by searching for every candidate combination and detecting repeats, including repeats inside the batch being written. An empty `label` and an empty-string `label` are treated as the same value. On violation:

```
Combinations of Link Tracker values (URL, campaign, medium, source, and label) must be unique.
The following combinations are already used: 
- <tuple>
```

where each `<tuple>` is `(address, campaign name, medium name, source name, label or "")`.

### 9.3 Creation

1. Creating without an address is refused with *"Creating a Link Tracker without URL is not possible"*.
2. An address starting with `?` or `#` is refused with *"“%s” is not a valid link, links cannot redirect to the current page."* (the placeholder is the offending value).
3. The address is validated and completed: a bare host such as `example.com` becomes `http://example.com`.
4. When no title is supplied, the target page's preview title is fetched; on failure the address itself becomes the title.
5. Campaign, source and medium supplied by request cookies are explicitly discarded: any of the three not present in the creation values is forced empty, in order that a visitor's cookies cannot silently attribute a new link.
6. The record is created, then one Link Tracker Code is created for it with a freshly generated code.

**Find-or-create.** A bulk operation takes a list of value sets and returns one tracker per entry, in the same order, reusing existing trackers that match the five unique fields and creating the missing ones once even when the input repeats them. The same two refusals apply; several invalid addresses are reported together, one per line.

### 9.4 Redirection address

```
no_external_tracking = system parameter "link_tracker.no_external_tracking"
```

1. When `no_external_tracking` is set, the target address has a host, and that host differs from the site host, the redirection address is the target address unchanged. No campaign parameter is appended to a foreign site.
2. Otherwise the existing query parameters of the target address are taken as the starting point, and three parameters are written into them, each only when its value is not empty: `utm_campaign` receives the campaign name, `utm_source` receives the source name, `utm_medium` receives the medium name.
3. The redirection address is the target address carrying that query, with every literal sequence of three dots replaced by its escaped form.

The escaping of three consecutive dots is required because some reverse proxies reject that sequence.

### 9.5 Search on the short address

Searching `short_url` strips the site base address and the `/r/` prefix from the searched value and then searches the codes. Therefore both a full short address and a bare code match.

### 9.6 Recent-links query

A named query returns trackers in one of three orders: `newest` (creation moment descending), `most-clicked` (visit count descending, restricted to trackers with at least one visit), `recently-used` (last write descending, same restriction). Any other value returns `{"Error": "This filter doesn't exist."}`.

---

## 10. Link Tracker Code

**Transport name** `link.tracker.code` · **storage name** `link_tracker_code` · **generated reference page** [`link.tracker.code`](../../references/entities/link.tracker.code.md)

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `code` | Code | text | yes | generated | stored | The short code appearing after `/r/`. |
| `link_id` | Link | many_to_one to Link Tracker | yes | empty | stored, indexed, delete cascades | The tracker it resolves to. |

- **Display name.** The code.
- **Constraint.** Unique `code`. Message: *"Code must be unique."*
- **Generation.** See [calculations.md](calculations.md) section 9: random alphanumeric strings of a length that starts at 3 and grows until the whole requested batch is unique and unused.
- A tracker may own several codes; the newest one is the tracker's `code`, and all of them resolve to the same target.

---

## 11. Link Tracker Click

**Transport name** `link.tracker.click` · **storage name** `link_tracker_click` · **generated reference page** [`link.tracker.click`](../../references/entities/link.tracker.click.md)

One recorded visit.

- **Display name.** The link.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `link_id` | Link | many_to_one to Link Tracker | yes | empty | stored, indexed, delete cascades | The tracker visited. |
| `campaign_id` | Campaign | many_to_one to Campaign | no | derived | related to the link, stored, partially indexed, delete sets empty | Campaign attribution of the visit. |
| `ip` | Internet protocol address | text | no | empty | stored | Network address of the visitor as seen by the server. |
| `country_id` | Country | many_to_one to Country | no | empty | stored | Resolved from the visitor's country code. |
| `mailing_trace_id` | Mailing trace | many_to_one to Mailing Trace | no | empty | stored, partially indexed | Added by the Email Marketing package: the exact recipient, when the visit came from a message. |
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing | no | empty | stored, partially indexed | Added by the Email Marketing package. |

**Recording a visit.** See [workflows.md](workflows.md) section 12. In short: the code is resolved; if no code matches, nothing is recorded and nothing is returned. Otherwise a visit is created with the link, the network address, the resolved country and, when a trace was named, the trace, its campaign and its mailing. When a trace is attached, that trace is marked opened and clicked.

---

## 12. Campaign

**Transport name** `utm.campaign` · **storage name** `utm_campaign` · **generated reference page** [`utm.campaign`](../../references/entities/utm.campaign.md)

A named marketing effort. It is the parent of comparison tests and the unit of attribution.

- **Default ordering.** Model default.
- **Display name.** The `title`.
- **Archiving.** Supported.
- **Company scoping.** A company field exists with the current company as default, and the currency follows that company. Records are not filtered by company by default.

### 12.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `title` | Title | text | yes | empty | stored, translated | yes | The human name. |
| `name` | Name | text | yes | derived | stored, derived with manual override, not translated, pre-computed | yes | The unique technical name, derived from the title and made unique with a bracketed counter (section 18.2). This is the value written into link parameters. |
| `user_id` | User | many_to_one to User | yes | the current user | stored | yes | Responsible. |
| `stage_id` | Stage | many_to_one to Campaign Stage | yes | the first stage by sequence | stored, delete restricted | **no** | Board column; empty columns are shown. |
| `tag_ids` | Tags | many_to_many to Campaign Tag | no | empty | stored | yes | Labels. |
| `is_auto_campaign` | Is auto campaign | boolean | no | false | stored | yes | True for campaigns created implicitly from a link parameter, in order to filter them out of the campaign screens. |
| `color` | Color index | integer | no | empty | stored | yes | Card color. |
| `company_id` | Company | many_to_one to Company | no | the current company | stored | yes | Owner company. |
| `currency_id` | Currency | many_to_one to Currency | no | derived | related to the company | n/a | Used to format the revenue indicator. |
| `click_count` | Click count | integer | no | derived | derived, not stored | n/a | Number of Link Tracker Click records attributed to the campaign. |
| `mailing_mail_ids` | Mailing mails | one_to_many to Mass Mailing restricted to type `mail` | no | empty | stored inverse, marketing group only | no | Electronic-mail mailings of the campaign. |
| `mailing_mail_count` | Mailing mail count | integer | no | derived | derived, marketing group only | n/a | Their number. |
| `mailing_sms_ids` | Mailing text messages | one_to_many to Mass Mailing restricted to type `sms` | no | empty | stored inverse, marketing group only | no | Text-message mailings. |
| `mailing_sms_count` | Mailing text message count | integer | no | derived | derived, marketing group only | n/a | Their number. |
| `ab_testing_mailings_count` | Comparison test electronic mail version count | integer | no | derived | derived | n/a | Electronic-mail mailings of the campaign that have comparison testing enabled. |
| `ab_testing_mailings_sms_count` | Comparison test text message version count | integer | no | derived | derived | n/a | The same for text messages. |
| `ab_testing_winner_mailing_id` | Comparison test winner mailing | many_to_one to Mass Mailing | no | empty | stored | **no** | The version chosen as winner. |
| `ab_testing_completed` | Comparison test completed | boolean | no | derived | stored, derived, read-only | **no** | True as soon as a winner mailing is recorded. |
| `ab_testing_schedule_datetime` | Comparison test decision moment | datetime | no | current moment plus one day | stored | yes | When the automatic winner selection must run. |
| `ab_testing_winner_selection` | Comparison test winner criterion | selection: `manual`, `opened_ratio`, `clicks_ratio`, `replied_ratio` (extended by bridges with `crm_lead_count`, `sale_quotation_count`, `sale_invoiced_amount`) | no | `opened_ratio` | stored | yes | Criterion for electronic mail. |
| `ab_testing_sms_winner_selection` | Comparison test text message winner criterion | selection: `manual`, `clicks_ratio` (extended by bridges with `crm_lead_count`, `sale_quotation_count`, `sale_invoiced_amount`) | no | `clicks_ratio` | stored | yes | Criterion for text messages. |
| `is_mailing_campaign_activated` | Is mailing campaign activated | boolean | no | derived | derived, not stored | n/a | True when the current user belongs to the campaign-management group. |
| `received_ratio` | Received ratio | decimal (2 places) | no | derived | derived, not stored | n/a | See [calculations.md](calculations.md) section 3. |
| `opened_ratio` | Opened ratio | decimal (2 places) | no | derived | derived, not stored | n/a | Same. |
| `replied_ratio` | Replied ratio | decimal (2 places) | no | derived | derived, not stored | n/a | Same. |
| `bounced_ratio` | Bounced ratio | decimal (2 places) | no | derived | derived, not stored | n/a | Same. |
| `crm_lead_count` | Lead and opportunity count | integer | no | derived | derived, sales-team group only | n/a | Added by the Lead bridge. |
| `use_leads` | Use leads | boolean | no | derived | derived | n/a | Added by the Lead bridge. |
| `quotation_count` | Quotation count | integer | no | derived | derived, sales-team group only | n/a | Added by the Sales package. Note that the Campaign spells this indicator without a package prefix, while the Mass Mailing spells the equivalent indicator `sale_quotation_count`. |
| `invoiced_amount` | Invoiced amount | integer | no | derived | derived, sales-team group only | n/a | Added by the Sales package. Note that the Campaign spells this indicator without a package prefix, while the Mass Mailing spells the equivalent indicator `sale_invoiced_amount`. |

### 12.2 Constraints and rules

| Kind | Statement | Message |
|---|---|---|
| Database constraint | Unique `name`. | *"The name must be unique"* |
| Deletion guard (Recruitment package) | A campaign used by the recruitment process cannot be deleted. | *"The UTM campaign '%s' cannot be deleted as it is used in the recruitment process."* |

**Creation.** When only `name` is supplied, `title` is set to the same value. Names are then made unique in batch (section 18.2).

**Already-contacted set.** For a campaign, the set of recipient records already addressed is the set of `res_id` values of its traces, optionally restricted to one recipient entity. This set is what makes a comparison test address each person only once.

---

## 13. Campaign Source

**Transport name** `utm.source` · **storage name** `utm_source` · **generated reference page** [`utm.source`](../../references/entities/utm.source.md)

Where a contact or a click came from.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored | The source name, unique. |

- **Constraint.** Unique `name`, message *"The name must be unique"*.
- **Creation.** Names are made unique in batch (section 18.2).
- **Deletion guards.** The shipped `Referral` source cannot be deleted: *"You cannot delete the 'Referral' UTM source record."* A source used by a marketing card campaign cannot be deleted: *"The UTM source '%s' cannot be deleted as it is used to promote marketing cards campaigns."* A source linked to mailings cannot be deleted: *"You cannot delete these UTM Sources as they are linked to the following mailings in Mass Mailing:"* followed by a line break and the comma-separated quoted mailing subjects. A source linked to recruitment sources cannot be deleted: *"You cannot delete these UTM Sources as they are linked to the following recruitment sources in Recruitment:"* followed by a line break and the list.
- **Generated name.** For a record that owns a source, the generated name is
  `"<content> (<entity display name> created on <creation date>)"`, where `<content>` is the owner's naming field with line breaks replaced by spaces and, when 24 characters or longer, truncated to 20 characters followed by three dots. When the content is empty no name is generated.

---

## 14. Campaign Medium

**Transport name** `utm.medium` · **storage name** `utm_medium` · **generated reference page** [`utm.medium`](../../references/entities/utm.medium.md)

How a message was delivered.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored, not translated | The medium name, unique. |
| `active` | Active | boolean | no | true | stored | Archiving flag. |

- **Default ordering.** `name`.
- **Constraint.** Unique `name`, message *"The name must be unique"*.
- **Protected mediums.** `Email`, `Direct`, `Website`, `X`, `Facebook`, `LinkedIn`, and, with the Text Message Marketing package, `Text Message`, cannot be deleted: *"Oops, you can't delete the Medium '%s'."* followed by a line break and *"Doing so would be like tearing down a load-bearing wall — not the best idea."*
- **Additional guards.** A medium linked to mailings cannot be deleted: *"You cannot delete these UTM Mediums as they are linked to the following mailings in Mass Mailing:"* followed by a line break and the quoted subjects. The text-message medium has its own message: *"The UTM medium '%s' cannot be deleted as it is used in some main functional flows, such as the SMS Marketing."*
- **Fetch-or-create.** Given a name and an owning capability package, the medium is looked up by the external key `utm_medium_<normalised name>` where normalisation lower-cases the name and replaces every space and dot by an underscore. When the key does not exist, a medium is created (using the protected label when the key is one of the protected ones) and the external key is registered. Therefore `New Medium`, `new medium`, `new_medium` and `new.Medium` all resolve to the same record.

---

## 15. Campaign Stage

**Transport name** `utm.stage` · **storage name** `utm_stage` · **generated reference page** [`utm.stage`](../../references/entities/utm.stage.md)

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored, translated | Column title. |
| `sequence` | Sequence | integer | no | 1 | stored | Order. |

- **Default ordering.** `sequence`.
- Stages with no campaign are still shown as columns in the campaign board.

---

## 16. Campaign Tag

**Transport name** `utm.tag` · **storage name** `utm_tag` · **generated reference page** [`utm.tag`](../../references/entities/utm.tag.md)

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored, translated | Label. |
| `color` | Color index | integer | no | a random integer between 1 and 11 inclusive | stored | Card color; no color means the tag is not displayed on cards. |

- **Default ordering.** `name`.
- **Constraint.** Unique `name`, message *"Tag name already exists!"*.

---

## 17. Campaign Tracking Mixin

**Transport name** `utm.mixin` · **storage name** `utm_mixin` · **generated reference page** [`utm.mixin`](../../references/entities/utm.mixin.md)

An abstract behavior added to any entity that must remember which marketing effort produced it (leads, quotations, registrations, link trackers, and more).

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `campaign_id` | Campaign | many_to_one to Campaign | no | from cookie | stored, partially indexed | "This is a name that helps you keep track of your different campaign efforts, e.g. Fall_Drive, Christmas_Special". |
| `source_id` | Source | many_to_one to Campaign Source | no | from cookie | stored, partially indexed | "This is the source of the link, e.g. Search Engine, another domain, or name of email list". |
| `medium_id` | Medium | many_to_one to Campaign Medium | no | from cookie | stored, partially indexed | "This is the method of delivery, e.g. Postcard, Email, or Banner Ad". |

### 17.1 The three tracked parameters

| Request parameter | Field | Cookie name |
|---|---|---|
| `utm_campaign` | `campaign_id` | `odoo_utm_campaign` |
| `utm_source` | `source_id` | `odoo_utm_source` |
| `utm_medium` | `medium_id` | `odoo_utm_medium` |

### 17.2 Capture

After every request is answered, the response is inspected: for each of the three parameters present in the request whose value differs from the current cookie value, a cookie of that name is set on the request host, classified as optional (therefore subject to the visitor's consent choices), with a lifetime of 31 days expressed in seconds (`31 × 24 × 3600`).

### 17.3 Stamping

When a record of a tracking-enabled entity is created and the field is among those being defaulted:

1. If the current user is not the system user **and** belongs to the salesperson group, nothing is stamped. Salespeople creating records by hand must not inherit a visitor's cookies.
2. Otherwise, for each of the three fields, the cookie value is read. When the value is a name rather than a key, the matching record is found or created (section 17.4). When a value is found, it becomes the default.

### 17.4 Find or create

Given an entity and a name: the name is trimmed; a case-insensitive equality search is made including archived records; if a record is found it is returned; otherwise a record is created with that name, and, on the Campaign entity, with `is_auto_campaign` set to true. The public front-end variant of this operation applies the same logic for the three tracking entities and falls back to a plain creation on the naming field for any other entity, with ordinary access checks.

---

## 18. Campaign Tracking Source Mixin

**Transport name** `utm.source.mixin` · **storage name** `utm_source_mixin` · **generated reference page** [`utm.source.mixin`](../../references/entities/utm.source.mixin.md)

An abstract behavior making an entity own exactly one Campaign Source, whose name doubles as the record's own name.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning |
|---|---|---|---|---|---|---|---|
| `source_id` | Source | many_to_one to Campaign Source | yes | created on the fly | stored, delete restricted | **no** | The owned source. |
| `name` | Name | text | no | derived | related to the source name, writable | n/a | The record's name. |

### 18.1 Creation, write and duplication

- **Defaults.** The `name` field is excluded from screen defaults, in order that a stale default name cannot violate uniqueness.
- **Creation.** For each value set without a source, a source is created whose name is: the supplied `name`, else the screen default name, else the generated name built from the record's naming field (section 13). The created source is linked and `name` is removed from the values.
- **Write.** Writing the naming field or the name on more than one record at once is refused with *"You cannot update multiple records with the same name. The name should be unique!"* Writing the naming field without a name regenerates the name. Any name written is then made unique, ignoring the record's own source when checking.
- **Duplication.** The copy receives the next free counter of the original name.

### 18.2 Unique-name algorithm

Given an entity and a list of wanted names, the algorithm returns one final name per input, in order:

1. Strip the counter part of each wanted name: `"<base> [<n>]"` becomes `<base>` and `n`; a name without brackets becomes itself and 1.
2. Search the entity for every existing name matching each base (case-insensitive containment), excluding records explicitly skipped (the record's own source when updating).
3. For each base, collect the counters already used by the existing names that are exactly the base or start with `"<base> ["`.
4. For each wanted name in order: when an explicit counter was given and is free, take it; otherwise take the lowest free counter starting at 1, filling holes. Mark it used.
5. The final name is `<base>` when the counter is 1, otherwise `"<base> [<counter>]"`.
6. An empty wanted name produces an empty result.

Worked example: the name `Spring new` already exists. Wanted names `Spring 1`, `Spring 2`, `Spring new`, `Spring new` and `Spring new [0]`, submitted in that order, produce `Spring 1`, `Spring 2`, `Spring new [2]`, `Spring new [3]` and `Spring new [4]`. After deleting the holders of `Spring new`, `Spring new [2]` and `Spring new [3]`, four further requests for `Spring new` produce `Spring new`, `Spring new [2]`, `Spring new [3]` and `Spring new [5]`, because counter 4 is still taken by the record created earlier.

---

## 19. Marketing Card Campaign

**Transport name** `card.campaign` · **storage name** `card_campaign` · **generated reference page** [`card.campaign`](../../references/entities/card.campaign.md)

A campaign that renders one personalised image per record and gives each person a page from which to share it.

- **Default ordering.** `identifier` descending.
- **Display name.** The name.
- **Archiving.** Supported.
- **Discussion thread.** Yes, with activities.
- **Rendering.** The body is rendered with the template engine against the target entity, with the campaign available to the template. Rendering of this entity is unrestricted, because the rendered field is a read-only related field of the shipped template and therefore only a settings administrator can change it.

### 19.1 Fields

| Field | Full name | Type | Required | Default | Stored / derived | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored | Campaign name. |
| `active` | Active | boolean | no | true | stored | Archiving flag. |
| `user_id` | User | many_to_one to User | no | the current user | stored | Responsible; restricted to non-shared users. |
| `card_template_id` | Card template | many_to_one to Marketing Card Template | yes | the first template | stored | The chosen design. |
| `body_html` | Body rich text | rich_text | no | derived | related to the template body, writable, rendered with the template engine | The layout actually rendered. |
| `res_model` | Related record model | selection of allowed entities | yes | derived | stored, derived, read-only, pre-computed | The entity the cards are produced for. Allowed values are the intersection of the entities present in the system with the fixed list: Contact, Event Track, Event Booth, Event Registration. |
| `preview_record_ref` | Preview record reference | reference (entity plus key) | yes | empty | stored | The record used to render the preview; its entity decides `res_model`. |
| `tag_ids` | Tags | many_to_many to Marketing Card Campaign Tag | no | empty | stored | Labels. |
| `image_preview` | Image preview | image | no | derived | stored, derived, read-only, not stored as an attachment | The rendered preview image. |
| `link_tracker_id` | Link tracker | many_to_one to Link Tracker | no | created at creation | stored, delete restricted | Measures visits to the campaign's target address. |
| `target_url` | Target web address | text | no | empty | stored | Where the share button finally sends people. Empty means the site base address. |
| `target_url_click_count` | Target web address click count | integer | no | derived | related to the tracker's count | Visits to the target address. |
| `post_suggestion` | Post suggestion | long_text | no | empty | stored | Text shown under the card and used as the default post text when sharing on the X social network. |
| `request_title` | Request title | text | no | *"Help us share the news"* | stored | Heading of the sharing page. |
| `request_description` | Request description | long_text | no | empty | stored | Explanation on the sharing page. |
| `reward_message` | Reward message | rich_text | no | empty | stored | Thank-you message shown after sharing. |
| `reward_target_url` | Reward target web address | text | no | empty | stored | Link offered as a reward. |
| `card_ids` | Cards | one_to_many to Marketing Card | no | empty | stored inverse | The produced images. |
| `card_count` | Card count | integer | no | derived | derived, not stored | Number of cards. |
| `card_click_count` | Card click count | integer | no | derived | derived, not stored | Cards whose share status is `visited` or `shared`. |
| `card_share_count` | Card share count | integer | no | derived | derived, not stored | Cards whose share status is `shared`. |
| `mailing_ids` | Mailings | one_to_many to Mass Mailing | no | empty | stored inverse | Mailings that distribute these cards. |
| `mailing_count` | Mailing count | integer | no | derived | derived, not stored | Their number. |

### 19.2 Content-mapping fields

The design is filled from a fixed set of slots. Each textual slot has three fields: a static value, a dynamic flag and a field path.

| Slot | Static value | Dynamic flag | Field path | Color |
|---|---|---|---|---|
| Header | `content_header` | `content_header_dyn` | `content_header_path` | `content_header_color` |
| Sub-header | `content_sub_header` | `content_sub_header_dyn` | `content_sub_header_path` | `content_sub_header_color` |
| Section | `content_section` | `content_section_dyn` | `content_section_path` | — |
| Sub-section 1 | `content_sub_section1` | `content_sub_section1_dyn` | `content_sub_section1_path` | — |
| Sub-section 2 | `content_sub_section2` | `content_sub_section2_dyn` | `content_sub_section2_path` | — |

Two image slots exist and are always dynamic: `content_image1_path` and `content_image2_path`. One static image slot exists: `content_background`. One static text slot exists: `content_button`.

**Slot resolution for one record.**

For each textual slot, in this order:

1. When the dynamic flag is false, the value is the static value of the slot and no further step runs.
2. When the dynamic flag is true and the field path is empty, the value is the record itself.
3. When the dynamic flag is true and a field path is given, the path is walked segment by segment, each segment being fetched from the value produced by the previous one; the value is the first value obtained by mapping the record over the whole path, or empty when the walk produces nothing. Any attribute error or value error during the walk makes the value the field path itself, shown literally, so that a mistyped path is visible on the card rather than silently blank.
4. When the value produced by step 2 or step 3 is a date or a moment and the record exposes a time zone, the value is converted from coordinated universal time into that time zone and the zone marker is dropped.

For each image slot the value is the first value obtained by mapping the record over the path, when the path names a field of the record, and empty otherwise.

### 19.3 Creation, write and deletion

**Creation.** Before creating the campaign, one Link Tracker is created with elevated rights, with: the target address, or the site base address when empty; the campaign name as title (supplied explicitly to avoid a network fetch); the shipped `Marketing Card` source; and a label of the form `marketing_card_campaign_<name>_<current moment>`. The campaign is then created with that tracker.

**Write.**
1. When any rendering-relevant field is written (the body, the template, any content slot, the background, the button, any dynamic flag, any path, the header colors), every card of the campaign, including archived ones, is marked as needing regeneration.
2. When the target address is written, the tracker's address is updated to the new value or to the site base address when empty.
3. The write happens.
4. If the write changed `res_model` on a campaign that already has cards, the write is refused with *"Model of campaign %(campaign)s may not be changed as it already has cards"*.

**Deletion.** Removing an entity from the registry deletes the marketing card campaigns targeting it.

### 19.4 Operations

| Operation | Behavior |
|---|---|
| Preview | Fetch or create the card of the preview record, store the current preview image on it, force it archived (in order that it is regenerated before a real send), and open its preview page in a new tab. |
| Share | Open a new mailing form pre-filled with: this campaign, the campaign name as subject, the campaign's entity as recipient entity, and a default body containing an introductory text block and a centred clickable preview image. |
| View cards / clicked / shared | Open the card list filtered on the campaign, optionally with the visited or shared filter pre-selected. |
| View mailings | Open the mailings of the campaign, titled `<name> Mailings`. |
| Update cards (from a mailing) | For every draft mailing of the campaign, produce or refresh the cards of the mailing's recipients in batches of 100, committing between batches. |

**Image production for one record.** When the template body is empty the result is empty. Otherwise the body is rendered for that record and rasterised to an image of 600 by 315 pixels (the ratio recommended by social networks for large preview images). When rasterising fails outside automated tests, the operation is refused with *"An error occured while rendering a card for %(record_name)s. Try again or check the server logs for more details."*

**Card refresh algorithm.** Given a condition:
1. Search the target entity with that condition to obtain the recipient keys.
2. Load the campaign's cards for those keys, including archived ones, and set them all active.
3. Create a card for every key that has none.
4. Repeatedly take up to 100 cards of this campaign that are flagged as needing regeneration and whose record is in the key set; render each one, store the image, clear the flag and set it active. When automatic committing is requested and at least one batch has already been done, commit and drop the cached images before the next batch.

---

## 20. Marketing Card

**Transport name** `card.card` · **storage name** `card_card` · **generated reference page** [`card.card`](../../references/entities/card.card.md)

The rendered image for one record of one campaign.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `campaign_id` | Campaign | many_to_one to Marketing Card Campaign | yes | empty | stored, indexed, delete cascades | Owner campaign. |
| `res_model` | Related record model | selection | no | derived | related to the campaign, computed once | The target entity; computed from the campaign and never updated afterwards. |
| `res_id` | Related record identifier | integer reference to a record of `res_model` | yes | empty | stored | The record the card was produced for. |
| `image` | Image | image | no | empty | stored | The produced image. |
| `requires_sync` | Requires sync | boolean | no | true | stored | True while the image does not match the current campaign design. |
| `share_status` | Share status | selection: empty, `visited`, `shared` | no | empty | stored | Empty = never opened; `visited` = the person opened their preview page; `shared` = a social-network crawler fetched the image. |
| `active` | Active | boolean | no | true | stored | Archived while the card is only a preview. |

- **Constraint.** Unique (`campaign_id`, `res_id`). Message: *"Each record should be unique for a campaign"*.
- **Display name.** The display name of the referenced record, read with elevated rights; empty when the entity is unknown.
- **Addresses.** The image address is `<base>/cards/<slug>/card.jpg`, the preview address is `<base>/cards/<slug>/preview` and the redirection address is `<base>/cards/<slug>/redirect`, where `<slug>` is the readable-key form of the card.
- **Automatic cleanup.** A cleanup routine deletes every card, archived ones included, whose last write is older than the number of days in the system parameter `marketing_card.card_image_cleanup_interval_days` (default 60). A value of zero or empty disables the cleanup. Social networks are expected to have cached the images by then.

---

## 21. Marketing Card Template

**Transport name** `card.template` · **storage name** `card_template` · **generated reference page** [`card.template`](../../references/entities/card.template.md)

A shipped image layout.

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored, translated | Template name. |
| `body` | Body | rich_text | no | empty | stored, not sanitised | The layout. Only a settings administrator may change it. |
| `default_background` | Default background | image | no | empty | stored | Background image proposed with the template. |
| `primary_color` | Primary color | text | yes | `#f9f9f9` | stored | Colors are stored as hexadecimal strings. |
| `secondary_color` | Secondary color | text | yes | `#000000` | stored | |
| `primary_text_color` | Primary text color | text | yes | `#000000` | stored | |
| `secondary_text_color` | Secondary text color | text | yes | `#ffffff` | stored | |

Rendered images are 600 by 315 pixels, a ratio of 40 to 21.

---

## 22. Marketing Card Campaign Tag

**Transport name** `card.campaign.tag` · **storage name** `card_campaign_tag` · **generated reference page** [`card.campaign.tag`](../../references/entities/card.campaign.tag.md)

| Field | Full name | Type | Required | Default | Stored | Meaning |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | empty | stored | Label. |
| `color` | Color index | integer | no | a random integer between 1 and 11 inclusive | stored | Card color. |

**Constraint.** Unique `name`. Message: *"Tags may not reuse existing names."*

---

## 23. Mailing Contact Import Wizard

**Transport name** `mailing.contact.import` · **storage name** `mailing_contact_import` · **generated reference page** [`mailing.contact.import`](../../references/entities/mailing.contact.import.md)

Pastes many addresses at once.

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `mailing_list_ids` | Mailing lists | many_to_many to Mailing List | no | the lists of the calling screen | Lists to subscribe the imported contacts to. |
| `contact_list` | Contact list | long_text | no | empty | One contact per line, each either a bare address or `"Name" <address>`. |

Behavior is specified in [workflows.md](workflows.md) section 8. The wizard also offers to open the general file-import assistant for the Mailing Contact entity, which is the path recommended above 5000 addresses.

---

## 24. Mailing Contact to List Wizard

**Transport name** `mailing.contact.to.list` · **storage name** `mailing_contact_to_list` · **generated reference page** [`mailing.contact.to.list`](../../references/entities/mailing.contact.to.list.md)

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `contact_ids` | Contacts | many_to_many to Mailing Contact | no | the selected contacts | Contacts to add. |
| `mailing_list_id` | Mailing list | many_to_one to Mailing List | yes | empty | Destination list. |

Two operations: add and close, or add and open a new mailing pre-filled with that list. Both add only the contacts that are not already members, then show *"<n> Mailing Contacts have been added. "* followed by the chosen next action.

---

## 25. Mailing List Merge Wizard

**Transport name** `mailing.list.merge` · **storage name** `mailing_list_merge` · **generated reference page** [`mailing.list.merge`](../../references/entities/mailing.list.merge.md)

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `src_list_ids` | Source lists | many_to_many to Mailing List | no | the selected lists | Lists to merge. |
| `dest_list_id` | Destination list | many_to_one to Mailing List | no | the first selected list | Target list when merging into an existing one. |
| `merge_options` | Merge options | selection: `new`, `existing` | yes | `new` | `new` = "Merge into a new mailing list"; `existing` = "Merge into an existing mailing list". |
| `new_list_name` | New list name | text | no | empty | Name of the list created when the option is `new`. |
| `archive_src_lists` | Archive source lists | boolean | no | true | Archive the sources after the merge, except the destination. |

**Default loading.** When the source lists are not supplied and the calling screen is not a Mailing List screen, the wizard refuses to open with *"You can only apply this action from Mailing Lists."*

---

## 26. Mailing Test Wizard

**Transport name** `mailing.mailing.test` · **storage name** `mailing_mailing_test` · **generated reference page** [`mailing.mailing.test`](../../references/entities/mailing.mailing.test.md)

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `email_to` | Email to | long_text | yes | the last value used by this user, otherwise the user's formatted address | Line-separated list of addresses. |
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing | yes | the current mailing | The mailing to sample. |

The wizard's records are kept for ten hours instead of the usual one hour, in order that the last used address can be proposed again.

---

## 27. Mailing Schedule Wizard and Test Text Message Mailing Wizard

**Transport names** `mailing.mailing.schedule.date` and `mailing.sms.test` · **storage names** `mailing_mailing_schedule_date` and `mailing_sms_test` · **generated reference pages** [`mailing.mailing.schedule.date`](../../references/entities/mailing.mailing.schedule.date.md) and [`mailing.sms.test`](../../references/entities/mailing.sms.test.md)

**Mailing Schedule Wizard**

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `schedule_date` | Schedule date | datetime | no | empty | The chosen moment. |
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing | yes | the current mailing | The mailing to schedule. |

Its single operation writes `schedule_type = "scheduled"` and the chosen moment on the mailing, then queues it.

**Test Text Message Mailing Wizard**

| Field | Full name | Type | Required | Default | Meaning |
|---|---|---|---|---|---|
| `numbers` | Numbers | long_text | yes | the last value used by this user, otherwise the user's own sanitised number | Line-separated list of telephone numbers. |
| `mailing_id` | Mailing | many_to_one to Mass Mailing | yes | the current mailing | The mailing to sample. |

---

## 28. Fields and behavior added to entities owned by other domains

### 28.1 Model Definition (Platform Foundation)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `is_mailing_enabled` | Is mailing enabled | boolean | derived, not stored, searchable | True when the entity declares itself mailable. |

Searching on it with the operators "in" and "not in" is supported and is resolved by listing every entity of the registry that exists in the running system, is not a temporary assistant entity, and declares the flag. Any other operator is unsupported.

Entities that declare the flag: Contact, Mailing Contact, Mailing List, and, with the matching capability packages, Lead, Sales Order, Event Registration and Event Track.

### 28.2 Outgoing Mail Server (Platform Foundation)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `active_mailing_ids` | Active mailings | one_to_many to Mass Mailing restricted to state other than `done` and non-archived | stored inverse, read-only | Mailings that would break if the server were removed. |

**Usage description.** Before deletion the system lists the usages of a server. This domain adds: *"Email Marketing uses it as its default mail server to send mass mailings"* when the server is the configured dedicated server, and, for each non-finished mailing, `Mass Mailing "<mailing display name>"`, followed by ` (scheduled for <date>)` when the mailing has a schedule date.

**Validation on the server owner.** Giving a personal owner to a server is refused when the server is the configured dedicated marketing server: *"Cannot set an owner on '%(server)s': it is configured as the dedicated Email Marketing server."* It is also refused when at least one mailing whose state is not `done` uses the server: *"Cannot set an owner on '%(server)s': it is used by mailing '%(mailing)s'."* A server used only by finished mailings may receive an owner.

**Server selection when sending.** When an outgoing mail belongs to a mailing, servers that have a personal owner are removed from the candidate list, even when their sender filter matches. Therefore a marketing message is never sent through somebody's personal server.

### 28.3 Request Routing (Platform Foundation)

After every dispatched request the response is post-processed to capture the three campaign-tracking parameters into cookies (section 17.2). The front-end translation catalogue of the Email Marketing package is added to the public pages.

### 28.4 Email Blacklist (Messaging and Activities)

| Field | Full name | Type | Stored | Tracked | Meaning |
|---|---|---|---|---|---|
| `opt_out_reason_id` | Opt out reason | many_to_one to Mailing Opt-Out Reason | stored, delete restricted | weight 10 | Why the address was blocked. |

When the reason changes and becomes non-empty, the change entry is posted as a visible message rather than as a silent note.

### 28.5 Message Composer Wizard (Messaging and Activities)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `mass_mailing_id` | Mass mailing | many_to_one to Mass Mailing, delete cascades | stored | The mailing on whose behalf the composer runs. |
| `campaign_id` | Campaign | many_to_one to Campaign, delete sets empty | stored | Campaign attribution. |
| `mass_mailing_name` | Mass mailing name | text | stored | When set and no mailing is linked, a mailing is created at send time in order that the results can be tracked. |
| `mailing_list_ids` | Mailing lists | many_to_many to Mailing List | stored | Lists carried into the created mailing. |

Behavior added:

1. **Implicit mailing creation.** In mass mode, when a name is given, no mailing is linked and the recipient entity has a discussion thread, a mailing is created before sending with: the composer attachments, the composer body as rich-text body, the campaign, the recipient entity, the condition (the composer condition, or `id IN (<the explicit keys>)`), the name, the answer address and mode, the current moment as sent moment, the state `done`, the subject and the exclusion-list flag.
2. **Trace generation.** In mass mode with a linked mailing on a thread-enabled entity, every prepared outgoing mail receives one trace with: the first normalised recipient address (or the raw address when none normalises), the mailing, the message key, the recipient entity and the recipient key. When the prepared mail is already cancelled the trace status is `cancel`; when it is already in error the status is `error`; the failure type is copied when present.
3. **Body wrapping.** The rich-text body of each outgoing mail is wrapped in the marketing layout together with the marketing stylesheet. The thread-visible body is prefixed with `Received the mailing <b><mailing name></b>` and the original body is quoted below it.
4. **Invalid addresses are cancelled.** For marketing sends, a missing or unusable address always produces the state `cancel` rather than an exception, because the number of such failures is expected to be untrackable.
5. **No per-recipient notifications.** Notification records are not created for marketing sends; the traces replace them.
6. **Already-contacted set.** The composer's set of already-contacted addresses is extended with the mailing's own set (section 1 of [calculations.md](calculations.md), step 3).
7. **Opted-out set.** The composer's set of opted-out addresses is extended with the mailing's own set.
8. **Cancelled mails are filtered out.** After the exclusion checks, the values whose state is `cancel` are removed from the batch and their traces are created directly, with elevated rights, in order that a cancelled recipient still appears in the measurement.
9. **With a marketing card campaign**, the already-contacted set is emptied, because every recipient receives a different image and duplicate suppression by address would wrongly remove people; and the generic card addresses in the body are replaced by the recipient's own card addresses.

### 28.6 Outgoing Email (Messaging and Activities)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `mailing_id` | Mailing | many_to_one to Mass Mailing | stored | The mailing that produced this message. |
| `mailing_trace_ids` | Mailing traces | one_to_many to Mailing Trace | stored inverse | Its traces (one in practice). |

Behavior added:

1. **Open tracking.** When the mail belongs to a mailing and has a trace, an image tag pointing at `<recipient base>/mail/track/<mail key>/<token>/blank.gif` is appended to the body. The token is a keyed hash of the mail key under the purpose `mass_mailing-mail_mail-open`. The base address is the one of the recipient record, in order that the image stays on the recipient's website.
2. **Trace key inside short links.** Every link of the body whose path starts with `/r/` is rewritten to `<short address>/m/<trace key>`, in order that a click identifies the exact recipient.
3. **Per-recipient links.** For each outgoing address, the placeholder `<base>/unsubscribe_from_list` in the body is replaced by the recipient's unsubscribe address and the placeholder `<base>/view` by the recipient's view-in-browser address, both built on the recipient's own base address. The unsubscribe substitution is skipped for test sends.
4. **Unsubscribe headers.** Four headers are added to every outgoing marketing message: `List-Unsubscribe` set to the one-click unsubscribe address between angle brackets, `List-Unsubscribe-Post` set to `List-Unsubscribe=One-Click`, `Precedence` set to `list_id`, and `X-Auto-Response-Suppress` set to `OOF` in order to prevent out-of-office answers.
5. **Result propagation.** After an attempt, the traces of mails belonging to a mailing are marked failed with the reported failure type, or marked sent when no failure was reported.
6. **Cleanup.** A cleanup routine deletes, in pages of at most 10 000 records ordered by key, the cancelled outgoing mails whose last write is older than the number of months in the system parameter `mass_mailing.cancelled_mails_months_limit` (default 6). A value of zero or less disables the cleanup.

### 28.7 Rendering Mixin (Messaging and Activities)

Two operations are added.

**Shorten links in a rich-text body.** Inputs: the body, the values to copy onto created trackers (campaign, source, medium, mailing), an optional list of addresses to skip, an optional base address.

1. Empty or visually empty bodies are returned unchanged.
2. The base address defaults to the system parameter `web.base.url`; the short prefix is that base followed by `/r/`.
3. Every anchor element with a non-empty target is examined. Its address is made absolute by prefixing the base when it starts with `/`, `?` or `#`. It is skipped when it matches the protocol skip pattern (electronic mail, telephone and text-message links), when it already starts with the short prefix, or when it matches one of the skipped addresses followed by `#`, `?`, `/` or the end of the address.
4. The label of the link is the trimmed text of the anchor, truncated to 40 characters. When the anchor has no text, the label is taken from its children: for an image, the alternative text prefixed by `[media] `, or the last path segment of its source prefixed by `[media] `, or an empty string; comments are skipped; a paragraph marked as an outgoing-mail compatibility wrapper is descended into.
5. All (address, label) pairs are turned into trackers in one find-or-create call, and each anchor target is replaced by the tracker's short address.

**Shorten links in a plain-text body.** Every distinct address found in the text is examined; addresses already shortened and addresses pointing at the text-message opt-out prefix `<base>/sms/` are skipped, as are addresses whose path matches a skipped item followed by `#`, `?`, `/` or the end. Each remaining address is replaced, wherever it appears and only when not followed by another address character, by the short address of a found-or-created tracker.

**Post-processing.** After a template field is rendered, if the rendering context carries link-tracker values, the rendered rich text is shortened with the skip list `/unsubscribe_from_list`, `/view`, `/cards`.

### 28.8 Discussion Thread Mixin (Messaging and Activities)

1. **Answers.** When an incoming message is routed to a record, the message keys found in its reference headers (or its in-reply-to header) are looked up among the traces. Every matching trace is marked opened and then replied.
2. **Bounces.** When an incoming message is a bounce, the traces whose message key is among the bounced keys are marked bounced with the plain-text body as reason. For every trace whose stored address differs from the bounced address, the underlying recipient record is additionally told about the bounce, because the ordinary bounce handling would have missed it.
3. **Automatic blocking.** After handling a bounce for an address, the traces with status `bounce`, written in the last thirteen weeks, whose address matches the bounced address case-insensitively, are counted. When there are at least **5** of them, and either no contact was resolved or every resolved contact already has a bounce counter of at least 5, and the newest of those traces is more than one week after the oldest, the address is added to the blocked-address register with the note *"This email has been automatically added in blocklist because of too much bounced."* The one-week spread requirement prevents a temporary server outage from blocking an address.
4. **Attribution of records created from an answer.** When a new record is created by the incoming gateway on a tracking-enabled entity, the message keys in the reference headers are looked up among the traces; the first match provides the campaign, the source of its mailing and the medium of its mailing as defaults.
5. **Guard.** Messages posted through the ordinary posting operations explicitly clear the mailing defaults from the context, in order that an ordinary note is never mistaken for a mass mailing.

### 28.9 Text Message Composer Wizard (Messaging and Activities)

| Field | Full name | Type | Default | Meaning |
|---|---|---|---|---|
| `mailing_id` | Mailing | many_to_one to Mass Mailing | empty | The mailing on whose behalf the composer runs. |
| `mass_sms_allow_unsubscribe` | Mass text message allow unsubscribe | boolean | true | Whether the opt-out sentence is appended. |
| `utm_campaign_id` | Campaign tracking campaign | many_to_one to Campaign, delete sets empty | empty | Campaign attribution. |

Behavior added: opted-out records come from the mailing; already-contacted records come from the mailing (because a comparison test may already have reached them); the body is shortened with the mailing's tracker values before sending; each prepared message receives a trace; cancelled messages are filtered out and their traces are created directly; after creation the short links receive the message key.

**Trace preparation for one recipient.** A three-character code is generated. The trace carries the mailing, the recipient entity and key, the code, the sanitised number, a new provider tracker and the type `sms`. When the prepared message is in error, the trace takes the status `error` and the failure type; when it is cancelled, the status `cancel` and the failure type. Otherwise, when the opt-out option is on, the message body is extended with a line break and *"STOP SMS: <base>/sms/<mailing key>/<code>"*, where the base is the recipient's own base address.

### 28.10 Outgoing Text Message (Messaging and Activities)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `mailing_id` | Mailing | many_to_one to Mass Mailing | stored | The mailing. |
| `mailing_trace_ids` | Mailing traces | one_to_many to Mailing Trace, joined on the plain integer key | stored inverse | Its traces. |

Behavior added: before sending, every address in the body that starts with `<base>/r/` is rewritten to `<address>/s/<message key>`, wherever it appears and only when not followed by another address character, in order that a click identifies the exact message and therefore the exact recipient.

### 28.11 Text Message Tracker (Messaging and Activities)

| Field | Full name | Type | Stored | Meaning |
|---|---|---|---|---|
| `mailing_trace_id` | Mailing trace | many_to_one to Mailing Trace, delete cascades | stored, partially indexed | The trace to update when the provider reports. |

**Status mapping.**

| Provider-side state | Trace status |
|---|---|
| `error` | `error` |
| `process` | `process` |
| `outgoing` | `outgoing` |
| `canceled` | `cancel` |
| `pending` | `pending` |
| `sent` | `sent` |

**Update rule.** A new status is ignored when the trace is already in a status at least as advanced. The ignore sets are: for `cancel`, the statuses `cancel`, `process`, `pending`, `sent`; for `outgoing`, the statuses `outgoing`, `process`, `pending`, `sent`; for `process`, the statuses `process`, `pending`, `sent`; for `pending`, the statuses `pending`, `sent`; for `bounce`, the status `bounce`; for `sent`, the status `sent`; for `error`, the status `error`. When the trace is updated, the failure type and reason are written too, and the sent moment is filled with the current database moment for every trace whose new status is not one of `outgoing`, `process`, `error`, `cancel` and whose sent moment was empty.

**Mailing closing.** When the reported status is `process`, the mailings of the updated traces move to state `sending`. Otherwise, every mailing of the updated traces that has no trace left in status `process` and is not already `done` is written to `done`, with the current moment as sent moment and the statistics flag set when it had no previous sent moment. When the update arrives from an unauthenticated provider callback, the change entries are attributed to the system contact.

### 28.12 Company (Contacts and Organizations)

Added by the Social Media capability package:

| Field | Full name | Type | Meaning |
|---|---|---|---|
| `social_twitter` | Account on the X network | text | Account address on the X social network. |
| `social_facebook` | Account on Facebook | text | Account address on Facebook. |
| `social_github` | Account on GitHub | text | Account address on GitHub. |
| `social_linkedin` | Account on LinkedIn | text | Account address on LinkedIn. |
| `social_youtube` | Account on YouTube | text | Account address on YouTube. |
| `social_instagram` | Account on Instagram | text | Account address on Instagram. |
| `social_tiktok` | Account on TikTok | text | Account address on TikTok. |
| `social_discord` | Account on Discord | text | Account address on Discord. |

**Accessor used by mailing footers.** The Email Marketing package adds an operation returning the five addresses used by the footer building blocks: Facebook, LinkedIn, X, Instagram and TikTok. The Newsletter Subscribe Button package overrides it so that, when a website is resolved, each value is taken from the website when the website defines it and from the company otherwise.

### 28.13 Configuration Settings (Platform Foundation)

| Setting | Type | Storage | Effect |
|---|---|---|---|
| `group_mass_mailing_campaign` | boolean | grants the campaign-management group | Shows campaigns, campaign stages and campaign tags; enables and disables the comparison-test job. |
| `mass_mailing_outgoing_mail_server` | boolean | system parameter `mass_mailing.outgoing_mail_server` | Reveals the dedicated-server choice on every mailing. |
| `mass_mailing_mail_server_id` | many_to_one to Outgoing Mail Server | system parameter `mass_mailing.mail_server_id` | The dedicated server, used as the default of every new mailing. Cleared when the boolean above is turned off. |
| `show_blacklist_buttons` | boolean | system parameter `mass_mailing.show_blacklist_buttons` | Shows the self-exclusion and re-inclusion buttons on the public subscription page. |
| `mass_mailing_reports` | boolean | system parameter `mass_mailing.mass_mailing_reports` | Enables the statistics message sent one day after each mailing. |
| `mass_mailing_split_contact_name` | boolean | activates two alternative Mailing Contact screens | Splits the contact name into a first name and a last name. |
| `newsletter_id` | many_to_one to Mailing List | field on the Website record | The list offered at online-shop checkout. Added by the Checkout Newsletter package. |
| `is_newsletter_enabled` | boolean | activates the checkout block of the current website | Derived from whether that block is active on the selected website. |

**Saving behavior.** Saving the settings also enables or disables the comparison-test scheduled job so that it matches the campaign setting, and activates or deactivates the two split-name screens so that they match the split-name setting. Reading the settings reports the split-name value by inspecting whether the alternative list screen is active.

### 28.14 Contact, User, Lead, Sales Order, Event entities, Course, Website

| Entity | Addition |
|---|---|
| Contact | Declares itself mailable. |
| User | The activity tray group for mailings is renamed to `Email Marketing`; with the Text Message Marketing package the single group is replaced by two groups, one per mailing type, obtained by a separate grouped read. |
| Lead | Declares itself mailable; supplies the lead-count indicator and winner criterion. |
| Sales Order | Declares itself mailable; supplies the default condition `state != "cancel"`; supplies the quotation-count and invoiced-amount indicators and winner criteria. |
| Event | Gains three buttons: mail the attendees (recipient entity Event Registration, condition `event IN (selected events) AND state NOT IN ("cancel", "draft")`, subject `Event: <name>`), invite contacts (recipient entity Contact, subject `Event: <name>`), and mail the speakers (recipient entity Event Track, condition `event IN (selected events) AND stage.is_cancel != true`, subject `Event: <name>`). With the text-message packages the same buttons open the mixed form that offers both channels. |
| Event Registration | Declares itself mailable; default condition `state NOT IN ("cancel", "draft")`, unless the calling screen supplied a prepared condition for the same recipient entity, in which case that condition is used. |
| Event Track | Declares itself mailable; default condition `stage.is_cancel = false`. |
| Course | Gains a button that opens a mailing addressed to Contact with the condition `slide_channel_ids IN (selected courses)`. |
| Website | Gains the `newsletter_id` list used by the checkout subscription option. |
