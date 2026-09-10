# Outgoing Mails (`mail.mail`)

**Transport name:** `mail.mail`  
**Storage name:** `mail_mail`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `mass_mailing`

Description: Outgoing Mails

## Identity and behavior

- Delegation inheritance: embeds `mail.message` through field `mail_message_id`
- Default ordering: `id desc`
- Display name field: `subject`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mail_message_id` | Message | many to one | `mail.message` | required; indexed; on delete of the target: cascade |
| `mail_message_id_int` | Mail Message Identifier Int | integer |  | computed by rule `_compute_mail_message_id_int` (not stored) |
| `message_type` | Message Type | selection |  | related through path `mail_message_id.message_type`; default `email_outgoing` |
| `body_html` | Text Contents | multi line text |  | Help: Rich-text/HTML message |
| `body_content` | Rich-text Contents | rich text |  | computed by rule `_compute_body_content` (not stored); searchable through a search rule |
| `references` | References | multi line text |  | read only; Help: Message references, such as identifiers of previous messages |
| `headers` | Headers | multi line text |  | not copied on duplication |
| `restricted_attachment_count` | Restricted attachments | integer |  | computed by rule `_compute_restricted_attachments` (not stored) |
| `unrestricted_attachment_ids` | Unrestricted Attachments | many to many | `ir.attachment` | computed by rule `_compute_restricted_attachments` (not stored); writable through an inverse rule |
| `is_notification` | Notification Email | boolean |  | Help: Mail has been created to notify people of an existing mail.message |
| `email_to` | To | multi line text |  | Help: Message recipients (emails) |
| `email_cc` | Cc | single line text |  | Help: Carbon copy message recipients |
| `recipient_ids` | To (Partners) | many to many | `res.partner` |  |
| `state` | Status | selection |  | read only; default `outgoing`; not copied on duplication |
| `failure_type` | Failure type | selection |  |  |
| `failure_reason` | Failure Reason | multi line text |  | read only; not copied on duplication; Help: Failure reason. This is usually the exception thrown by the email server, stored to ease the debugging of mailing issues. |
| `auto_delete` | Auto Delete | boolean |  | Help: This option permanently removes any track of email after it's been sent, including from the Technical menu in the Settings, in order to preserve storage space of your Odoo database. |
| `scheduled_date` | Scheduled Send Date | date and time |  | Help: If set, the queue manager will send the email after the date. If not set, the email will be send as soon as possible. Unless a timezone is specified, it is considered as being in UTC timezone. |
| `fetchmail_server_id` | Inbound Mail Server | many to one | `fetchmail.server` | read only; indexed (btree_not_null) |
| `mailing_id` | Mass Mailing | many to one | `mailing.mailing` |  |
| `mailing_trace_ids` | Statistics | one to many | `mailing.trace` | inverse field `mail_mail_id` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `outgoing` | Outgoing |
| `sent` | Sent |
| `received` | Received |
| `exception` | Delivery Failed |
| `cancel` | Cancelled |

### `failure_type` (Failure type)

| Value | Label |
|---|---|
| `unknown` | Unknown error |
| `mail_spam` | Detected As Spam |
| `mail_email_invalid` | Invalid email address |
| `mail_email_missing` | Missing email |
| `mail_from_invalid` | Invalid from address |
| `mail_from_missing` | Missing from address |
| `mail_smtp` | Connection failed (outgoing mail server problem) |
| `mail_bl` | Blacklisted Address |
| `mail_optout` | Opted Out |
| `mail_dup` | Duplicated Email |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (33)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail` | model |  |
| `_check_mail_server_id` | validation | self | `mail` | constrains: `mail_message_id`, `mail_server_id` |  |
| `_compute_body_content` | computation | self | `mail` |  |  |
| `_compute_mail_message_id_int` | computation | self | `mail` |  |  |
| `_compute_restricted_attachments` | computation | self | `mail` | depends: `attachment_ids` | We might not have access to all the attachments of the emails. Compute the attachments we have access to, and the number of attachments we do not have access to. |
| `_inverse_unrestricted_attachment_ids` | inverse computation | self | `mail` |  | We can only remove the attachments we have access to. |
| `_search_body_content` | search rule | self, operator, value | `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `action_retry` | user action | self | `mail` |  |  |
| `action_open_document` | user action | self | `mail` |  | Opens the related record based on the model and ID |
| `mark_outgoing` | operation | self | `mail` |  |  |
| `cancel` | operation | self | `mail` |  |  |
| `process_email_queue` | operation | self, email_ids, batch_size | `mail` | model | Send immediately queued messages, committing after each    message is sent - this is not transactional and should    not be called during another transaction!  A maximum of 1K MailMail (configurable using 'mail.mail.queue.batch.size' optional ICP) are fetched in order to keep time under control.  :param list email_ids: optional list of emails ids to send. If given only                  scheduled and outgoing emails within this ids list                  are sent; |
| `_postprocess_sent_message` | internal rule | self, success_pids, success_emails, failure_reason, failure_type | `mail`, `mass_mailing` |  | Perform any post-processing necessary after sending ``mail`` successfully, including deleting it completely along with its attachment if the ``auto_delete`` flag of the mail was set. Overridden by subclasses for extra post-processing behaviors.  :return: True |
| `_parse_scheduled_datetime` | internal rule | self, scheduled_datetime | `mail` |  | Taking an arbitrary datetime (either as a date, a datetime or a string) try to parse it and return a datetime timezoned to UTC.  If no specific timezone information is given, we consider it as being given in UTC, as all datetime values given to the server. Trying to guess its timezone based on user or flow would be strange as this is not standard. When manually creating datetimes for mail.mail scheduled date, business code should ensure either a timezone info is set, either it is converted into UTC.  Using yearfirst when parsing str datetimes eases parser's job when dealing with the hard-to-pa |
| `_estimate_email_size` | internal rule | self, headers, body, attachments_size | `mail` | model | Estimate the email size, incorporating a small added security margin.  :param dict headers: email headers :param str body: email body :param list attachments_size: list of attachment size in bytes |
| `_filter_mail_mail_servers` | internal rule | self, mail_servers | `mail`, `mass_mailing` |  |  |
| `_prepare_outgoing_body` | preparation rule | self | `mail`, `mass_mailing` |  | Return a specific ir_email body. The main purpose of this method is to be inherited to add custom content depending on some module. |
| `_personalize_outgoing_body` | internal rule | self, body, partner, doc_to_followers | `mail` |  | Return a modified body based on the recipient (partner).  It must be called when using standard notification layouts even for message without partners.  :param str body: body to personalize for the recipient :param partner: <res.partner> recipient :param dict doc_to_followers: see ``Followers._get_mail_doc_to_followers()`` |
| `_prepare_outgoing_list` | preparation rule | self, mail_server, doc_to_followers | `mail`, `mass_mailing` |  | Return a list of emails to send based on current mail.mail. Each is a dictionary for specific email values, depending on a partner, or generic to the whole recipients given by mail.email_to.  :param mail_server: <ir.mail_server> mail server that will be used to send the mails,   False if it is the default one :param dict doc_to_followers: see ``Followers._get_mail_doc_to_followers()`` :returns: list of dicts used in IrMailServer._build_email__() :rtype: list[dict] |
| `_split_by_mail_configuration` | internal rule | self | `mail` |  | Group the <mail.mail> based on their "email_from", their "alias domain" and their "mail_server_id".  The <mail.mail> will have the "same sending configuration" if they have the same mail server, alias domain and mail from. For performance purpose, we can use an SMTP session in batch and therefore we need to group them by the parameter that will influence the mail server used.  The same "sending configuration" may repeat in order to limit batch size according to the `mail.session.batch.size` system parameter.  Return iterators over     mail_server_id, email_from, Records<mail.mail>.ids |
| `_split_by_delayed_batch` | internal rule | self, mail_server | `mail` |  | To not flag personal email servers as spam, we throttle them at X emails / minutes. |
| `send_after_commit` | operation | self | `mail` |  | Queues the email to be sent after the commit of the current cursor.  Useful to send an email only if a transaction is successful. |
| `send` | operation | self, auto_commit, raise_exception, post_send_callback | `mail` |  | Sends the selected emails immediately, ignoring their current state (mails that have already been sent should not be passed unless they should actually be re-sent). Emails successfully delivered are marked as 'sent', and those that fail to be deliver are marked as 'exception', and the corresponding error mail is output in the server logs.  :param bool auto_commit: whether to force a commit of the mail status     after sending each mail (meant only for scheduler processing);     should never be True during normal transactions (default: False) :param bool raise_exception: whether to raise an exc |
| `action_send_and_close` | user action | self | `mail` |  | An action sending the selected mail and redirecting to mail.mail list view. |
| `_send` | internal rule | self, auto_commit, raise_exception, smtp_session, alias_domain_id, mail_server, post_send_callback | `mail` |  |  |
| `_get_notification_values` | preparation rule | self | `mail` |  | Get list of base notification values to create a notification for existing emails.  Recipient-specific values should be added separately. |
| `_get_notification_status` | preparation rule | self | `mail` |  | Return the equivalent status for notifications based on state. |
| `_get_tracking_url` | preparation rule | self | `mass_mailing` |  |  |
| `_generate_mail_recipient_token` | internal rule | self, mail_id | `mass_mailing` | model |  |
| `_gc_canceled_mail_mail` | background operation | self | `mass_mailing` | autovacuum | Garbage collects old canceled mail.mail records as we consider nobody is going to look at them anymore, becoming noise. |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_mail_server_id` | ValidationError | You may not create a message using another user's mail server. | `mail` |
| `_send` | UserError | Unauthorized server for some of the sending mails. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.view_mail_form` | form |  | `message_type`, `state`, `model`, `res_id`, `mail_message_id_int`, `subject`, `author_id`, `date`, `email_from`, `email_to`, `recipient_ids`, `email_cc`, `reply_to`, `scheduled_date`, `body_content`, `auto_delete`, `is_notification`, `message_type`, `mail_server_id`, `model`, `res_id`, `message_id`, `references`, `fetchmail_server_id`, `headers`, `restricted_attachment_count`, `unrestricted_attachment_ids`, `failure_reason` | `Send & Close`, `Retry`, `Cancel`, `action_open_document`, `Reply` |  | `mail` |
| `mail.view_mail_tree` | list |  | `date`, `subject`, `author_id`, `message_id`, `recipient_ids`, `model`, `res_id`, `email_from`, `message_type`, `state` | `Retry`, `Send Now`, `Retry`, `Cancel Email` |  | `mail` |
| `mail.view_mail_search` | search |  | `email_from`, `date`, `author_id`, `recipient_ids`, `model`, `res_id` |  | `Received`, `Outgoing`, `Sent`, `Failed`, `Outgoing Email`, `Incoming Email`, `Comment`, `Notification`, `Status`, `Author`, `Thread`, `Date` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_view_mail_mail` | Emails | list,form |  | `{}` |  | `mail` |
| `mail.act_server_history` | Messages |  | `[('email_from', '!=', False), ('fetchmail_server_id', '=', active_id)]` | `{'search_default_server_id': active_id, 'default_fetchmail_server_id': active_id}` |  | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_mail_scheduler_action` | Mail: Email Queue Manager | 1 hours | `process_email_queue` | 6 |

Machine-readable definition: `../../../schemas/data/entities/mail.mail.json`; views: `../../../schemas/interfaces/views/mail.mail.json`.
