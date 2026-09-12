# Incoming Mail Server (`fetchmail.server`)

**Transport name:** `fetchmail.server`  
**Storage name:** `fetchmail_server`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `google_gmail`, `microsoft_outlook`

Description: Incoming Mail Server

## Identity and behavior

- Mixins (classical inheritance): `google.gmail.mixin`, `microsoft.outlook.mixin`
- Default ordering: `priority`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `state` | Status | selection |  | read only; default `draft`; indexed; not copied on duplication |
| `server` | Server Name | single line text |  | Help: Hostname or IP of the mail server |
| `port` | Port | integer |  |  |
| `server_type` | Server Type | selection |  | required; default `imap`; indexed; on delete of the target: {"outlook": "set default"}; extended by packages `google_gmail`, `microsoft_outlook` |
| `server_type_info` | Server Type Info | multi line text |  | computed by rule `_compute_server_type_info` (not stored) |
| `is_ssl` | SSL/TLS | boolean |  | Help: Connections are encrypted with SSL/TLS through a dedicated port (default: IMAPS=993, POP3S=995) |
| `attach` | Keep Attachments | boolean |  | default `True`; Help: Whether attachments should be downloaded. If not enabled, incoming emails will be stripped of any attachments before being processed |
| `original` | Keep Original | boolean |  | Help: Whether a full original copy of each email should be kept for reference and attached to each processed message. This will usually double the size of your message database. |
| `date` | Last Fetch Date | date and time |  | read only |
| `error_date` | Last Error Date | date and time |  | read only; Help: Date of last failure, reset on success. |
| `error_message` | Last Error Message | multi line text |  | read only |
| `user` | Username | single line text |  |  |
| `password` | Password | single line text |  |  |
| `object_id` | Create a New Record | many to one | `ir.model` | Help: Process each incoming mail as part of a conversation corresponding to this document type. This will create new documents for new conversations, or attach follow-up emails to the existing conversations (documents). |
| `priority` | Server Priority | integer |  | default `5`; Help: Defines the order of processing, lower values mean higher priority |
| `message_ids` | Messages | one to many | `mail.mail` | read only; inverse field `fetchmail_server_id` |
| `configuration` | Configuration | multi line text |  | read only |
| `script` | Script | single line text |  | read only; default `/mail/static/scripts/system-mailgate.py` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Not Confirmed |
| `done` | Confirmed |

### `server_type` (Server Type)

| Value | Label |
|---|---|
| `imap` | IMAP Server |
| `pop` | POP Server |
| `local` | Local Server |
| `gmail` | Gmail OAuth Authentication |
| `outlook` | Outlook OAuth Authentication |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_server_type_info` | computation | self | `google_gmail`, `mail`, `microsoft_outlook` | depends: `server_type` |  |
| `onchange_server_type` | on change | self | `google_gmail`, `mail`, `microsoft_outlook` | onchange: `server_type`, `is_ssl`, `object_id`; onchange: `server_type` | Set the default configuration for a IMAP Gmail server. |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `set_draft` | operation | self | `mail` |  |  |
| `_connect__` | internal rule | self, allow_archived | `mail` |  | :param bool allow_archived: by default (False), an exception is raised when calling this method on an    archived record. It can be set to True for testing so that the exception is no longer raised. |
| `_imap_login__` | internal rule | self, connection | `google_gmail`, `mail`, `microsoft_outlook` |  | Authenticate the IMAP connection.  Can be overridden in other module for different authentication methods.  :param connection: The IMAP connection to authenticate |
| `button_confirm_login` | user action | self | `mail` |  |  |
| `fetch_mail` | operation | self | `mail` |  | Action to fetch the mail from the current server. |
| `_fetch_mails` | internal rule | self, **kw | `mail` | model | Method called by cron to fetch mails from servers |
| `_fetch_mail` | internal rule | self, batch_limit | `mail` |  | Fetch e-mails from multiple servers.  Commit after each message. |
| `_get_connection_type` | preparation rule | self | `google_gmail`, `mail`, `microsoft_outlook` |  | Return which connection must be used for this mail server (IMAP or POP). Can be overridden in sub-module to define which connection to use for a specific "server_type" (e.g. Gmail server). |
| `_update_cron` | internal rule | self | `mail` | model |  |
| `_check_use_google_gmail_service` | validation | self | `google_gmail` | constrains: `server_type`, `is_ssl` |  |
| `_check_use_microsoft_outlook_service` | validation | self | `microsoft_outlook` | constrains: `server_type`, `is_ssl` |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_connect__` | UserError | The server "%s" cannot be used because it is archived. | `mail` |
| `button_confirm_login` | UserError | Invalid server name!  %s | `mail` |
| `button_confirm_login` | UserError | No response received. Check server information.  %s | `mail` |
| `button_confirm_login` | UserError | Server replied with following exception:  %s | `mail` |
| `button_confirm_login` | UserError | An SSL exception occurred. Check SSL/TLS configuration on server port.  %s | `mail` |
| `button_confirm_login` | UserError | Connection test failed: %s | `mail` |
| `_check_use_google_gmail_service` | UserError | SSL is required for server “%s”. | `google_gmail` |
| `_check_use_microsoft_outlook_service` | UserError | SSL is required for server “%s”. | `microsoft_outlook` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `google_gmail.fetchmail_server_view_form` | field | `mail.view_email_server_form` | `user` | `open_google_gmail_uri`, `open_google_gmail_uri` |  | `google_gmail` |
| `mail.view_email_server_tree` | list |  | `name`, `server_type`, `user`, `date`, `state` |  |  | `mail` |
| `mail.view_email_server_form` | form |  | `state`, `active`, `name`, `server_type`, `date`, `server_type_info`, `server`, `port`, `is_ssl`, `user`, `password`, `object_id`, `configuration`, `script`, `priority`, `attach`, `original`, `error_date`, `error_message` | `Test & Confirm`, `Fetch Now`, `Reset Confirmation` |  | `mail` |
| `mail.view_email_server_search` | search |  | `name`, `user` |  | `IMAP`, `POP`, `SSL`, `Archived` | `mail` |
| `microsoft_outlook.fetchmail_server_view_form` | field | `mail.view_email_server_form` | `user` | `open_microsoft_outlook_uri`, `open_microsoft_outlook_uri` |  | `microsoft_outlook` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_email_server_tree` | Incoming Mail Servers | list,form |  |  |  | `mail` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail.ir_cron_mail_gateway_action` | Mail: Fetchmail Service | 5 minutes | `_fetch_mails` |  |

Machine-readable definition: `../../../schemas/data/entities/fetchmail.server.json`; views: `../../../schemas/interfaces/views/fetchmail.server.json`.
