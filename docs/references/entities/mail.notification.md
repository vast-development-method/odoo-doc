# Message Notifications (`mail.notification`)

**Transport name:** `mail.notification`  
**Storage name:** `mail_notification`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `sms`, `sms_twilio`, `snailmail`

Description: Message Notifications

## Identity and behavior

- Display name field: `res_partner_id`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `author_id` | Author | many to one | `res.partner` | on delete of the target: set null |
| `mail_message_id` | Message | many to one | `mail.message` | required; indexed; on delete of the target: cascade |
| `mail_mail_id` | Mail | many to one | `mail.mail` | indexed; Help: Optional mail_mail ID. Used mainly to optimize searches. |
| `res_partner_id` | Recipient | many to one | `res.partner` | indexed; on delete of the target: cascade |
| `mail_email_address` | Mail Email Address | single line text |  | Help: Recipient email address |
| `notification_type` | Notification Type | selection |  | required; default `inbox`; indexed; on delete of the target: {"snail": "cascade"}; extended by packages `sms`, `snailmail` |
| `notification_status` | Status | selection |  | default `ready`; indexed |
| `is_read` | Is Read | boolean |  | indexed |
| `read_date` | Read Date | date and time |  | not copied on duplication |
| `failure_type` | Failure type | selection |  | extended by packages `sms`, `sms_twilio`, `snailmail` |
| `failure_reason` | Failure reason | multi line text |  | not copied on duplication |
| `sms_id_int` | text message identifier | integer |  | indexed (btree_not_null) |
| `sms_id` | text message | many to one | `sms.sms` | computed by rule `_compute_sms_id` (not stored) |
| `sms_tracker_ids` | text message Trackers | one to many | `sms.tracker` | inverse field `mail_notification_id` |
| `sms_number` | text message Number | single line text |  | visible only to groups `base.group_user` |
| `letter_id` | Snailmail Letter | many to one | `snailmail.letter` | indexed (btree_not_null); on delete of the target: cascade |

## Selection values

### `notification_type` (Notification Type)

| Value | Label |
|---|---|
| `inbox` | Inbox |
| `email` | Email |
| `sms` | SMS |
| `snail` | Snailmail |

### `notification_status` (Status)

| Value | Label |
|---|---|
| `ready` | Ready to Send |
| `process` | Processing |
| `pending` | Sent |
| `sent` | Delivered |
| `bounce` | Bounced |
| `exception` | Exception |
| `canceled` | Cancelled |

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
| `mail_optout` | Opted Out |
| `mail_dup` | Duplicated Email |
| `sms_number_missing` | Missing Number |
| `sms_number_format` | Wrong Number Format |
| `sms_credit` | Insufficient Credit |
| `sms_country_not_supported` | Country Not Supported |
| `sms_registration_needed` | Country-specific Registration Required |
| `sms_server` | Server Error |
| `sms_acc` | Unregistered Account |
| `sms_expired` | Expired |
| `sms_invalid_destination` | Invalid Destination |
| `sms_not_allowed` | Not Allowed |
| `sms_not_delivered` | Not Delivered |
| `sms_rejected` | Rejected |
| `twilio_authentication` | Authentication Error" |
| `twilio_callback` | Incorrect callback URL |
| `twilio_from_missing` | Missing From Number |
| `twilio_from_to` | From / To identic |
| `sn_credit` | Snailmail Credit Error |
| `sn_trial` | Snailmail Trial Error |
| `sn_price` | Snailmail No Price Available |
| `sn_fields` | Snailmail Missing Required Fields |
| `sn_format` | Snailmail Format Error |
| `sn_error` | Snailmail Unknown Error |

## State fields

State machine fields of this entity: `notification_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (5)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_notification_partner_required` | Constraint | `CHECK(notification_type != 'inbox' OR res_partner_id IS NOT NULL)` | Customer is required for inbox notification | `mail` |
| `_notification_partner_or_email_required` | Constraint | `CHECK(notification_type != 'email' OR failure_type IS NOT NULL OR res_partner_id IS NOT NULL OR COALESCE(mail_email_address, '') != '')` | Customer or email is required for inbox / email notification | `mail` |
| `_res_partner_id_is_read_notification_status_mail_message_id` | Index | `(res_partner_id, is_read, notification_status, mail_message_id)` |  | `mail` |
| `_author_id_notification_status_failure` | Index | `(author_id, notification_status) WHERE notification_status IN ('bounce', 'exception')` |  | `mail` |
| `_unique_mail_message_id_res_partner_id_` | UniqueIndex | `(mail_message_id, res_partner_id) WHERE res_partner_id IS NOT NULL` |  | `mail` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `_gc_notifications` | background operation | self, max_age_days | `mail` | model |  |
| `format_failure_reason` | operation | self | `mail` |  |  |
| `_filtered_for_web_client` | internal rule | self | `mail` |  | Returns only the notifications to show on the web client. |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |
| `_compute_sms_id` | computation | self | `sms` | depends: `sms_id_int`, `notification_type` |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `sms_twilio` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | AccessError | Can not update the message or recipient of a notification. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `mail` |
| `base.group_user` | yes | yes | yes | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| mail.notifications: group_user: write its own entries | `[Command.link(ref('base.group_user')), Command.link(ref('base.group_portal'))]` | `[('res_partner_id', '=', user.partner_id.id)]` | False | True | False | False |
| mail.notifications: group_portal: own entries | `[Command.link(ref('base.group_portal'))]` | `['\|', ('res_partner_id', '=', user.partner_id.id), ('author_id', '=', user.partner_id.id)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_notification_view_tree` | list |  | `mail_message_id`, `notification_type`, `res_partner_id`, `is_read`, `failure_type` |  |  | `mail` |
| `mail.mail_notification_view_form` | form |  | `notification_status`, `mail_message_id`, `notification_type`, `mail_mail_id`, `res_partner_id`, `is_read`, `read_date`, `failure_type`, `failure_reason` |  |  | `mail` |
| `sms.mail_notification_view_tree` | xpath | `mail.mail_notification_view_tree` | `sms_number` |  |  | `sms` |
| `sms.mail_notification_view_form` | xpath | `mail.mail_notification_view_form` | `sms_number` |  |  | `sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_notification_action` | Notifications | list,form |  |  |  | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_delete_notification` | Notification: Delete Notifications older than 6 Months | 1 months | `_gc_notifications` |  |

Machine-readable definition: `../../../schemas/data/entities/mail.notification.json`; views: `../../../schemas/interfaces/views/mail.notification.json`.
