# Mailing List Message (`mail.group.message`)

**Transport name:** `mail.group.message`  
**Storage name:** `mail_group_message`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail_group`

Description: Mailing List Message

## Identity and behavior

- Default ordering: `create_date DESC`
- Display name field: `subject`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `attachment_ids` | Attachment | many to many |  | related through path `mail_message_id.attachment_ids` |
| `author_id` | Author | many to one |  | related through path `mail_message_id.author_id` |
| `email_from` | Email From | single line text |  | related through path `mail_message_id.email_from` |
| `email_from_normalized` | Normalized From | single line text |  | computed by rule `_compute_email_from_normalized` and stored |
| `body` | Body | rich text |  | related through path `mail_message_id.body` |
| `subject` | Subject | single line text |  | related through path `mail_message_id.subject` |
| `mail_group_id` | Group | many to one | `mail.group` | required; indexed; on delete of the target: cascade |
| `mail_message_id` | Mail Message | many to one | `mail.message` | required; indexed; not copied on duplication; on delete of the target: cascade |
| `group_message_parent_id` | Parent | many to one | `mail.group.message` | indexed |
| `group_message_child_ids` | Children | one to many | `mail.group.message` | inverse field `group_message_parent_id` |
| `author_moderation` | Author Moderation Status | selection |  | computed by rule `_compute_author_moderation` (not stored) |
| `is_group_moderated` | Is Group Moderated | boolean |  | related through path `mail_group_id.moderation` |
| `moderation_status` | Status | selection |  | required; default `pending_moderation`; indexed; not copied on duplication |
| `moderator_id` | Moderated By | many to one | `res.users` |  |
| `create_date` | Posted | date and time |  |  |

## Selection values

### `author_moderation` (Author Moderation Status)

| Value | Label |
|---|---|
| `ban` | Banned |
| `allow` | Whitelisted |

### `moderation_status` (Status)

| Value | Label |
|---|---|
| `pending_moderation` | Pending Moderation |
| `accepted` | Accepted |
| `rejected` | Rejected |

## State fields

State machine fields of this entity: `moderation_status`. Transitions are specified in the domain documents.

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_email_from_normalized` | computation | self | `mail_group` | depends: `email_from` |  |
| `_compute_author_moderation` | computation | self | `mail_group` | depends: `email_from_normalized`, `mail_group_id` |  |
| `_constrains_mail_message_id` | validation | self | `mail_group` | constrains: `mail_message_id` |  |
| `create` | lifecycle override | self, vals_list | `mail_group` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `mail_group` |  |  |
| `action_moderate_accept` | user action | self | `mail_group` |  | Accept the incoming email.  Will send the incoming email to all members of the group. |
| `action_moderate_reject_with_comment` | user action | self, reject_subject, reject_comment | `mail_group` |  |  |
| `action_moderate_reject` | user action | self | `mail_group` |  |  |
| `action_moderate_allow` | user action | self | `mail_group` |  |  |
| `action_moderate_ban` | user action | self | `mail_group` |  |  |
| `action_moderate_ban_with_comment` | user action | self, ban_subject, ban_comment | `mail_group` |  |  |
| `_get_pending_same_author_same_group` | preparation rule | self | `mail_group` |  | Return the pending messages of the same authors in the same groups. |
| `_create_moderation_rule` | internal rule | self, status | `mail_group` |  | Create a moderation rule <mail.group.moderation> with the given status.  Update existing moderation rule for the same email address if found, otherwise create a new rule. |
| `_assert_moderable` | internal rule | self | `mail_group` |  | Raise an error if one of the current message can not be moderated.  A <mail.group.message> can only be moderated if it's moderation status is "pending_moderation". |
| `_moderate_send_reject_email` | internal rule | self, subject, comment | `mail_group` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_mail_message_id` | AccessError | Group message can only be linked to mail group. Current model is %s. | `mail_group` |
| `_constrains_mail_message_id` | AccessError | The record of the message should be the group. | `mail_group` |
| `_create_moderation_rule` | UserError | The email "%s" is not valid. | `mail_group` |
| `_assert_moderable` | UserError | Those messages can not be moderated: %s. | `mail_group` |
| `_assert_moderable` | UserError | This message can not be moderated | `mail_group` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `mail_group` |
| `base.group_portal` | no | yes | no | no | `mail_group` |
| `base.group_user` | yes | yes | yes | yes | `mail_group` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mail Group Message: Only accepted message are accessible | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[             '&',                 ('moderation_status', '=', 'accepted'),                 '\|',                 '\|',                 '\|',                     ('mail_group_id.moderator_ids', 'in', user.id),                     ('mail_group_id.access_mode', '=', 'public'),                     '&',                         ('mail_group_id.access_mode', '=', 'groups'),                         ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                     '&',                         ('mail_group_id.access_mode', '=', 'members'),                         ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| Mail Group Message: Non-accepted messages are accessible only by moderators | `[(4, ref('base.group_user'))]` | `[                 '&',                     '\|',                         ('moderation_status', '=', 'accepted'),                         ('mail_group_id.moderator_ids', 'in', user.id),                     '\|',                     '\|',                     '\|',                         ('mail_group_id.moderator_ids', 'in', user.id),                         ('mail_group_id.access_mode', '=', 'public'),                         '&',                             ('mail_group_id.access_mode', '=', 'groups'),                             ('mail_group_id.access_group_id', 'in', user.all_group_ids.ids),                         '&',                             ('mail_group_id.access_mode', '=', 'members'),                             ('mail_group_id.member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | True | True | True |
| Mail Group Message: Administrator have access to all messages | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_message_view_list` | list |  | `create_date`, `author_id`, `email_from`, `subject`, `mail_group_id`, `moderation_status`, `is_group_moderated` | `Accept`, `Reject`, `Whitelist`, `Ban`, `Send` |  | `mail_group` |
| `mail_group.mail_group_message_view_form` | form |  | `mail_message_id`, `moderation_status`, `is_group_moderated`, `subject`, `author_id`, `email_from`, `author_moderation`, `mail_group_id`, `create_date`, `attachment_ids`, `body` | `Accept`, `Reject`, `Whitelist`, `Ban`, `Send` |  | `mail_group` |
| `mail_group.mail_group_message_view_search` | search |  | `mail_group_id`, `email_from`, `author_id`, `moderation_status` |  | `group` | `mail_group` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_message_action` | Messages | list,form |  |  |  | `mail_group` |

Machine-readable definition: `../../../schemas/data/entities/mail.group.message.json`; views: `../../../schemas/interfaces/views/mail.group.message.json`.
