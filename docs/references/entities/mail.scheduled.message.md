# Scheduled Message (`mail.scheduled.message`)

**Transport name:** `mail.scheduled.message`  
**Storage name:** `mail_scheduled_message`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Scheduled Message

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `subject` | Subject | single line text |  |  |
| `body` | Contents | rich text |  |  |
| `scheduled_date` | Scheduled Date | date and time |  | required |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | association table `scheduled_message_attachment_rel` |
| `composition_comment_option` | Comment Options | selection |  |  |
| `model` | Related Document Model | single line text |  | required |
| `res_id` | Related Document Id | many to one by reference |  | required |
| `author_id` | Author | many to one | `res.partner` | required |
| `partner_ids` | Recipients | many to many | `res.partner` |  |
| `is_note` | Is a note | boolean |  | default ; Help: If the message will be posted as a Note. |
| `notification_parameters` | Notification parameters | multi line text |  |  |
| `send_context` | Sending Context | structured document |  |  |

## Selection values

### `composition_comment_option` (Comment Options)

| Value | Label |
|---|---|
| `reply_all` | Reply-All |
| `forward` | Forward |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_model` | validation | self | `mail` | constrains: `model` |  |
| `_check_scheduled_date` | validation | self | `mail` | constrains: `scheduled_date` |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `_search` | search rule | self, domain, offset, limit, order, bypass_access, **kwargs | `mail` | model | Override that add specific access rights to only get the ids of the messages that are scheduled on the records on which the user has mail_post (or read) access |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `open_edit_form` | operation | self | `mail` |  |  |
| `post_message` | operation | self | `mail` |  |  |
| `_message_created_hook` | messaging hook | self, message | `mail` |  | Hook called after scheduled messages have been posted. |
| `_post_message` | internal rule | self, raise_exception | `mail` |  | Post the scheduled messages. They are posted using their creator as user so that one can check that the creator has still post permission on the related record, and to allow for the attachments to be transferred to the messages (see _process_attachments_for_post in mail.thread) if raise_exception is set to False, the method will skip the posting of a message instead of raising an error, and send a notification to the author about the failure. This is useful when scheduled messages are sent from the _post_messages_cron. |
| `_check` | validation | self, values | `mail` | model | Restrict the access to a scheduled message. Access is based on the record on which the scheduled message will be posted to. :param values: dict with model and res_id on which to perform the check |
| `_notification_parameters_whitelist` | internal rule | self | `mail` | model | Parameters that can be used when posting the scheduled messages. |
| `_post_messages_cron` | internal rule | self, limit | `mail` | model | Posts past-due scheduled messages. |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_model` | ValidationError | A message cannot be scheduled on a model that does not have a mail thread. | `mail` |
| `_check_scheduled_date` | ValidationError | A Scheduled Message cannot be scheduled in the past | `mail` |
| `write` | UserError | You are not allowed to change the target record of a scheduled message. | `mail` |
| `post_message` | UserError | You are not allowed to send this scheduled message | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
|  | global (all users) | `[('create_uid', '=', user.id)]` | False | True | True | False |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_scheduled_message_view_form` | form |  | `composition_comment_option`, `res_id`, `model`, `partner_ids`, `subject`, `body`, `attachment_ids`, `attachment_ids`, `scheduled_date` | `Save`, `Send Now`, `Log Now`, `Discard` |  | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_post_scheduled_message` | Mail: Post scheduled messages | 1 days | `_post_messages_cron` |  |

Machine-readable definition: `../../../schemas/data/entities/mail.scheduled.message.json`; views: `../../../schemas/interfaces/views/mail.scheduled.message.json`.
