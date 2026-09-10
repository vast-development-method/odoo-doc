# Mailing Statistics (`mailing.trace`)

**Transport name:** `mailing.trace`  
**Storage name:** `mailing_trace`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mass_mailing`  
**Extended by packages:** `mass_mailing_sms`

Description: Mailing Statistics

## Identity and behavior

- Default ordering: `create_date DESC`
- Display name field: `id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `trace_type` | Type | selection |  | required; default `mail`; on delete of the target: {"sms": "set default"}; extended by packages `mass_mailing_sms` |
| `is_test_trace` | Generated for testing | boolean |  |  |
| `mail_mail_id` | Mail | many to one | `mail.mail` | indexed (btree_not_null) |
| `mail_mail_id_int` | Mail identifier (tech) | integer |  | indexed (btree_not_null); Help: ID of the related mail_mail. This field is an integer field because the related mail_mail can be deleted separately from its statistics. However the ID is needed for several action and controllers. |
| `email` | Email | single line text |  | Help: Normalized email address |
| `message_id` | Message-identifier | single line text |  |  |
| `medium_id` | Medium | many to one |  | related through path `mass_mailing_id.medium_id` |
| `source_id` | Source | many to one |  | related through path `mass_mailing_id.source_id` |
| `model` | Document model | single line text |  | required |
| `res_id` | Document identifier | many to one by reference |  |  |
| `mass_mailing_id` | Mailing | many to one | `mailing.mailing` | indexed; on delete of the target: cascade |
| `campaign_id` | Campaign | many to one |  | read only; related through path `mass_mailing_id.campaign_id` and stored; indexed (btree_not_null) |
| `sent_datetime` | Sent On | date and time |  |  |
| `open_datetime` | Opened On | date and time |  |  |
| `reply_datetime` | Replied On | date and time |  |  |
| `trace_status` | Status | selection |  | default `outgoing` |
| `failure_type` | Failure type | selection |  | extended by packages `mass_mailing_sms` |
| `failure_reason` | Failure reason | multi line text |  | read only; not copied on duplication |
| `links_click_ids` | Links click | one to many | `link.tracker.click` | inverse field `mailing_trace_id` |
| `links_click_datetime` | Clicked On | date and time |  | Help: Stores last click datetime in case of multi clicks. |
| `sms_id` | text message | many to one | `sms.sms` | computed by rule `_compute_sms_id` (not stored) |
| `sms_id_int` | text message identifier | integer |  | indexed (btree_not_null) |
| `sms_tracker_ids` | text message Trackers | one to many | `sms.tracker` | inverse field `mailing_trace_id` |
| `sms_number` | Number | single line text |  |  |
| `sms_code` | Code | single line text |  |  |

## Selection values

### `trace_type` (Type)

| Value | Label |
|---|---|
| `mail` | Email |
| `sms` | SMS |

### `trace_status` (Status)

| Value | Label |
|---|---|
| `outgoing` | Outgoing |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `open` | Opened |
| `reply` | Replied |
| `bounce` | Bounced |
| `error` | Exception |
| `cancel` | Cancelled |

### `failure_type` (Failure type)

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_bounce` | Bounce |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email address |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_dup` | Duplicated Email |
| `mail_optout` | Opted Out |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_credit` | Insufficient Credit |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_blacklist` | Blacklisted |
| `sms_duplicate` | Duplicate |
| `sms_optout` | Opted Out |
| `sms_expired` | Expired |
| `sms_invalid_destination` | Invalid Destination |
| `sms_not_allowed` | Not Allowed |
| `sms_not_delivered` | Not Delivered |
| `sms_rejected` | Rejected |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |

## State fields

State machine fields of this entity: `trace_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_res_id_is_set` | Constraint | `CHECK(res_id IS NOT NULL AND res_id !=0 )` | Traces have to be linked to records with a not null res_id. | `mass_mailing` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `mass_mailing` | depends: `trace_type`, `mass_mailing_id` |  |
| `create` | lifecycle override | self, vals_list | `mass_mailing_sms`, `mass_mailing` | model_create_multi |  |
| `action_view_contact` | user action | self | `mass_mailing` |  |  |
| `set_sent` | operation | self, domain | `mass_mailing` |  |  |
| `set_opened` | operation | self, domain | `mass_mailing` |  | Reply / Open are a bit shared in various processes: reply implies open, click implies open. Let us avoid status override by skipping traces that are not already opened or replied. |
| `set_clicked` | operation | self, domain | `mass_mailing` |  |  |
| `set_replied` | operation | self, domain | `mass_mailing` |  |  |
| `set_bounced` | operation | self, domain, bounce_message | `mass_mailing` |  |  |
| `set_failed` | operation | self, domain, failure_type | `mass_mailing` |  |  |
| `set_canceled` | operation | self, domain | `mass_mailing` |  |  |
| `_compute_sms_id` | computation | self | `mass_mailing_sms` | depends: `sms_id_int`, `trace_type` |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `mass_mailing_sms` | model |  |
| `_get_random_code` | preparation rule | self | `mass_mailing_sms` |  | Generate a random code for trace. Uniqueness is not really necessary as it serves as obfuscation when unsubscribing. A valid trio code / mailing_id / number will be requested. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_view_search` | search |  | `mail_mail_id_int`, `message_id`, `email`, `mass_mailing_id` |  | `Scheduled`, `Cancelled`, `Processing`, `Sent`, `Clicked`, `Delivered`, `Opened`, `Replied`, `Bounced`, `Failed`, `Test Traces`, `State`, `Open Date`, `Reply Date`, `Last State Update`, `Mass Mailing` | `mass_mailing` |
| `mass_mailing.mailing_trace_view_tree` | list |  | `mass_mailing_id`, `email`, `message_id`, `sent_datetime`, `links_click_datetime`, `trace_status`, `failure_type`, `open_datetime`, `reply_datetime`, `is_test_trace` | `Open Recipient` |  | `mass_mailing` |
| `mass_mailing.mailing_trace_view_tree_mail` | list |  | `mass_mailing_id`, `email`, `message_id`, `sent_datetime`, `links_click_datetime`, `trace_status`, `failure_type`, `open_datetime`, `reply_datetime`, `is_test_trace` | `Open Recipient` |  | `mass_mailing` |
| `mass_mailing.mailing_trace_view_form` | form |  | `trace_status`, `is_test_trace`, `failure_type`, `sent_datetime`, `links_click_datetime`, `open_datetime`, `failure_reason`, `reply_datetime`, `trace_type`, `email`, `mass_mailing_id`, `mail_mail_id_int`, `message_id`, `campaign_id`, `medium_id`, `source_id` | `action_view_contact` |  | `mass_mailing` |
| `mass_mailing.view_mail_mail_statistics_graph` | graph |  | `write_date`, `trace_status` |  |  | `mass_mailing` |
| `mass_mailing_sms.mailing_trace_view_search` | xpath | `mass_mailing.mailing_trace_view_search` | `sms_id_int`, `sms_id`, `sms_number` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_tree` | xpath | `mass_mailing.mailing_trace_view_tree` | `trace_type` |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_tree_sms` | list |  | `mass_mailing_id`, `sms_number`, `sent_datetime`, `links_click_datetime`, `trace_status`, `failure_type`, `open_datetime`, `reply_datetime`, `is_test_trace` | `Open Recipient` |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_form` | xpath | `mass_mailing.mailing_trace_view_form` |  |  |  | `mass_mailing_sms` |
| `mass_mailing_sms.mailing_trace_view_form_sms` | form |  | `is_test_trace`, `trace_status`, `failure_type`, `sent_datetime`, `links_click_datetime`, `open_datetime`, `reply_datetime`, `trace_type`, `sms_number`, `mass_mailing_id`, `sms_id_int`, `sms_code`, `campaign_id`, `medium_id`, `source_id`, `sms_id` | `action_view_contact` |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing.mailing_trace_action` | Mailing Traces | list,form,graph,pivot | `[]` |  |  | `mass_mailing` |
| `mass_mailing.action_view_mail_mail_statistics_mailing` | Mail Statistics | graph,list,form,pivot | `[]` | `{'search_default_mass_mailing_id': active_id}` |  | `mass_mailing` |

Machine-readable definition: `../../../schemas/data/entities/mailing.trace.json`; views: `../../../schemas/interfaces/views/mailing.trace.json`.
