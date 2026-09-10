# Reject Group Message (`mail.group.message.reject`)

**Transport name:** `mail.group.message.reject`  
**Storage name:** `mail_group_message_reject`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail_group`

Description: Reject Group Message

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `subject` | Subject | single line text |  | computed by rule `_compute_subject` and stored |
| `body` | Contents | rich text |  | default  |
| `email_from_normalized` | Email From | single line text |  | related through path `mail_group_message_id.email_from_normalized` |
| `mail_group_message_id` | Message | many to one | `mail.group.message` | required; read only |
| `action` | Action | selection |  | required |
| `send_email` | Send Email | boolean |  | computed by rule `_compute_send_email` (not stored); Help: Send an email to the author of the message |

## Selection values

### `action` (Action)

| Value | Label |
|---|---|
| `reject` | Reject |
| `ban` | Ban |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_subject` | computation | self | `mail_group` | depends: `mail_group_message_id` |  |
| `_compute_send_email` | computation | self | `mail_group` | depends: `body` |  |
| `action_send_mail` | user action | self | `mail_group` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail_group` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_message_reject_form` | form |  | `email_from_normalized`, `email_from_normalized`, `send_email`, `mail_group_message_id`, `subject`, `body`, `action` | `Reject Silently`, `Send & Reject`, `Ban`, `Send & Ban`, `Discard` |  | `mail_group` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_message_reject_action` | Message Rejection Explanation | form |  |  | new | `mail_group` |

Machine-readable definition: `../../../schemas/data/entities/mail.group.message.reject.json`; views: `../../../schemas/interfaces/views/mail.group.message.reject.json`.
