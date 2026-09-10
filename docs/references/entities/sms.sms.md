# Outgoing text message (`sms.sms`)

**Transport name:** `sms.sms`  
**Storage name:** `sms_sms`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sms`  
**Extended by packages:** `mass_mailing_sms`, `sms_twilio`

Description: Outgoing SMS

## Identity and behavior

- Default ordering: `id DESC`
- Display name field: `number`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `uuid` | UUID | single line text |  | read only; default computed dynamically (lambda self: uuid4().hex); not copied on duplication; Help: Alternate way to identify a SMS record, used for delivery reports |
| `number` | Number | single line text |  |  |
| `body` | Body | multi line text |  |  |
| `partner_id` | Customer | many to one | `res.partner` |  |
| `mail_message_id` | Mail Message | many to one | `mail.message` | indexed |
| `state` | text message Status | selection |  | required; read only; default `outgoing`; not copied on duplication |
| `failure_type` | Failure Type | selection |  | not copied on duplication; extended by packages `sms_twilio` |
| `sms_tracker_id` | text message trackers | many to one | `sms.tracker` | computed by rule `_compute_sms_tracker_id` (not stored) |
| `to_delete` | Marked for deletion | boolean |  | default ; Help: Will automatically be deleted, while notifications will not be deleted in any case. |
| `mailing_id` | Mass Mailing | many to one | `mailing.mailing` |  |
| `mailing_trace_ids` | Statistics | one to many | `mailing.trace` | inverse field `sms_id_int` |
| `sms_twilio_sid` | Text message Twilio Sid | single line text |  | related through path `sms_tracker_id.sms_twilio_sid` |
| `record_company_id` | Company | many to one | `res.company` | on delete of the target: set null |

## Selection values

### `state` (text message Status)

| Value | Label |
|---|---|
| `outgoing` | In Queue |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `error` | Error |
| `canceled` | Cancelled |

### `failure_type` (Failure Type)

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_credit` | Insufficient Credit |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_blacklist` | Blacklisted |
| `sms_duplicate` | Duplicate |
| `sms_optout` | Opted Out |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uuid_unique` | Constraint | `unique(uuid)` | UUID must be unique | `sms` |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `sms_twilio`, `sms` | model_create_multi |  |
| `_compute_sms_tracker_id` | computation | self | `sms` | depends: `uuid` |  |
| `action_set_canceled` | user action | self | `sms` |  |  |
| `action_set_error` | user action | self, failure_type | `sms` |  |  |
| `action_set_outgoing` | user action | self | `sms` |  |  |
| `send` | operation | self, unlink_failed, unlink_sent, raise_exception | `sms` |  | Main API method to send SMS.  This contacts an external server. If the transaction fails, it may be retried which can result in sending multiple SMS messages!    :param unlink_failed: unlink failed SMS after IAP feedback;   :param unlink_sent: unlink sent SMS after IAP feedback;   :param raise_exception: raise if there is an issue contacting IAP; |
| `_split_by_api` | internal rule | self | `sms_twilio`, `sms` |  |  |
| `resend_failed` | operation | self | `sms` |  |  |
| `_process_queue` | background operation | self | `sms` | model | CRON job to send queued SMS messages. |
| `_get_send_batch_size` | preparation rule | self | `sms_twilio`, `sms` |  |  |
| `_get_sms_company` | preparation rule | self | `sms_twilio`, `sms` |  |  |
| `_split_batch` | internal rule | self | `sms` |  |  |
| `_send` | internal rule | self, unlink_failed, unlink_sent, raise_exception | `sms` |  | Send SMS after checking the number (presence and formatting). |
| `_send_with_api` | internal rule | self, sms_api, unlink_failed, unlink_sent, raise_exception | `sms` |  | Send SMS after checking the number (presence and formatting). |
| `_update_sms_state_and_trackers` | internal rule | self, new_state, failure_type | `sms` |  | Update sms state update and related tracking records (notifications, traces). |
| `_handle_call_result_hook` | internal rule | self, results | `sms_twilio`, `sms` |  | Further process SMS sending API results. |
| `_gc_device` | background operation | self | `sms` | autovacuum |  |
| `_update_body_short_links` | internal rule | self | `mass_mailing_sms` |  | Override to tweak shortened URLs by adding statistics ids, allowing to find customer back once clicked. |
| `fields_get` | lifecycle override | self, allfields, attributes | `sms_twilio` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `sms` |
| `base.group_system` | yes | yes | yes | yes | `sms` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_tsms_view_form` | form |  | `to_delete`, `state`, `partner_id`, `mail_message_id`, `number`, `failure_type`, `body` | `Send Now`, `Retry`, `Cancel` |  | `sms` |
| `sms.sms_sms_view_tree` | list |  | `number`, `partner_id`, `failure_type`, `state` | `Send Now`, `Retry`, `Cancel` |  | `sms` |
| `sms.sms_sms_view_search` | search |  | `number`, `partner_id` |  |  | `sms` |
| `sms_twilio.sms_sms_view_form` | xpath | `sms.sms_tsms_view_form` | `sms_twilio_sid` |  |  | `sms_twilio` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sms.sms_sms_action` | SMS | list,form | `[('to_delete', '!=', True)]` |  |  | `sms` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `sms.ir_actions_server_sms_sms_resend` | Resend | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `sms.ir_cron_sms_scheduler_action` | SMS: SMS Queue Manager | 24 hours | `_process_queue` |  |

Machine-readable definition: `../../../schemas/data/entities/sms.sms.json`; views: `../../../schemas/interfaces/views/sms.sms.json`.
