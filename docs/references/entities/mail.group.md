# Mail Group (`mail.group`)

**Transport name:** `mail.group`  
**Storage name:** `mail_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail_group`  
**Extended by packages:** `website_mail_group`

Description: Mail Group

## Identity and behavior

- Mixins (classical inheritance): `mail.alias.mixin`
- Default ordering: `is_closed ASC, create_date DESC, id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; translatable |
| `description` | Description | multi line text |  |  |
| `image_128` | Image | image |  |  |
| `is_closed` | Is Closed | boolean |  | not copied on duplication; Help: Closed groups might still be accessed, but emails sent to it will bounce |
| `mail_group_message_ids` | Pending Messages | one to many | `mail.group.message` | inverse field `mail_group_id` |
| `mail_group_message_last_month_count` | Messages Per Month | integer |  | computed by rule `_compute_mail_group_message_last_month_count` (not stored) |
| `mail_group_message_count` | Messages Count | integer |  | computed by rule `_compute_mail_group_message_count` (not stored); Help: Number of message in this group |
| `mail_group_message_moderation_count` | Pending Messages Count | integer |  | computed by rule `_compute_mail_group_message_moderation_count` (not stored); Help: Messages that need an action |
| `is_member` | Is Member | boolean |  | computed by rule `_compute_is_member` (not stored) |
| `member_ids` | Members | one to many | `mail.group.member` | inverse field `mail_group_id` |
| `member_partner_ids` | Partners Member | many to many | `res.partner` | computed by rule `_compute_member_partner_ids` (not stored); searchable through a search rule |
| `member_count` | Members Count | integer |  | computed by rule `_compute_member_count` (not stored) |
| `is_moderator` | Moderator | boolean |  | computed by rule `_compute_is_moderator` (not stored); Help: Current user is a moderator of the group |
| `moderation` | Moderate | boolean |  |  |
| `moderation_rule_count` | Moderated emails count | integer |  | computed by rule `_compute_moderation_rule_count` (not stored) |
| `moderation_rule_ids` | Moderated Emails | one to many | `mail.group.moderation` | inverse field `mail_group_id` |
| `moderator_ids` | Moderators | many to many | `res.users` | restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('base.group_user').id)]`; association table `mail_group_moderator_rel` |
| `moderation_notify` | Automatic notification | boolean |  | Help: People receive an automatic notification about their message being waiting for moderation. |
| `moderation_notify_msg` | Notification message | rich text |  |  |
| `moderation_guidelines` | Send guidelines to new members | boolean |  | Help: Newcomers on this moderated group will automatically receive the guidelines. |
| `moderation_guidelines_msg` | Guidelines | rich text |  |  |
| `access_mode` | Privacy | selection |  | required; default `public` |
| `access_group_id` | Authorized Group | many to one | `res.groups` | default computed dynamically (lambda self: self.env.ref('base.group_user')) |
| `can_manage_group` | Can Manage | boolean |  | computed by rule `_compute_can_manage_group` (not stored); Help: Can manage the members |

## Selection values

### `access_mode` (Privacy)

| Value | Label |
|---|---|
| `public` | Everyone |
| `members` | Members only |
| `groups` | Selected group of users |

## Operations (45)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail_group` | model |  |
| `_compute_mail_group_message_last_month_count` | computation | self | `mail_group` | depends: `mail_group_message_ids.create_date`, `mail_group_message_ids.moderation_status` |  |
| `_compute_mail_group_message_count` | computation | self | `mail_group` | depends: `mail_group_message_ids` |  |
| `_compute_mail_group_message_moderation_count` | computation | self | `mail_group` | depends: `mail_group_message_ids.moderation_status` |  |
| `_compute_member_count` | computation | self | `mail_group` | depends: `member_ids` |  |
| `_compute_is_member` | computation | self | `mail_group` | depends_context: `uid` |  |
| `_compute_member_partner_ids` | computation | self | `mail_group` | depends: `member_ids` |  |
| `_search_member_partner_ids` | search rule | self, operator, operand | `mail_group` |  |  |
| `_compute_is_moderator` | computation | self | `mail_group` | depends: `moderator_ids`; depends_context: `uid` |  |
| `_compute_moderation_rule_count` | computation | self | `mail_group` | depends: `moderation_rule_ids` |  |
| `_compute_can_manage_group` | computation | self | `mail_group` | depends: `is_moderator`; depends_context: `uid` |  |
| `_onchange_access_mode` | on change | self | `mail_group` | onchange: `access_mode` |  |
| `_onchange_moderation` | on change | self | `mail_group` | onchange: `moderation` |  |
| `_check_moderator_email` | validation | self | `mail_group` | constrains: `moderator_ids` |  |
| `_check_moderation_notify` | validation | self | `mail_group` | constrains: `moderation_notify`, `moderation_notify_msg` |  |
| `_check_moderation_guidelines` | validation | self | `mail_group` | constrains: `moderation_guidelines`, `moderation_guidelines_msg` |  |
| `_check_moderator_existence` | validation | self | `mail_group` | constrains: `moderator_ids`, `moderation` |  |
| `_check_access_mode` | validation | self | `mail_group` | constrains: `access_mode`, `access_group_id` |  |
| `_alias_get_creation_values` | internal rule | self | `mail_group` |  | Return the default values for the automatically created alias. |
| `action_close` | user action | self | `mail_group` |  |  |
| `action_open` | user action | self | `mail_group` |  |  |
| `_alias_get_error` | internal rule | self, message, message_dict, alias | `mail_group` |  | Checks for access errors related to sending email to the mailing list. Returns None if the mailing list is public or if no error cases are detected. |
| `message_new` | messaging hook | self, msg_dict, custom_values | `mail_group` | model | Add the method to make the mail gateway flow work with this model. |
| `message_update` | messaging hook | self, msg_dict, update_vals | `mail_group` | model | Add the method to make the mail gateway flow work with this model. |
| `message_post` | messaging hook | self, body, subject, email_from, author_id, **kwargs | `mail_group` |  | Custom posting process. This model does not inherit from ``mail.thread`` but uses the mail gateway so few methods should be defined.  This custom posting process works as follow    * create a ``mail.message`` based on incoming email;   * create linked ``mail.group.message`` that encapsulates message in a     format used in mail groups;   * apply moderation rules;  :returns: newly-created mail.message |
| `action_send_guidelines` | user action | self, members | `mail_group` |  | Send guidelines to given members. |
| `_notify_members` | internal rule | self, message | `mail_group` |  | Send the given message to all members of the mail group (except the author). |
| `_cron_notify_moderators` | background operation | self | `mail_group` | model |  |
| `_notify_moderators` | internal rule | self | `mail_group` |  | Push a notification (Inbox / Email) to the moderators whose an action is waiting. |
| `_clean_email_body` | internal rule | self, body_html | `mail_group` | model | When we receive an email, we want to clean it before storing it in the database. |
| `_routing_check_route` | internal rule | self, message, message_dict, route, raise_exception | `mail_group` | model | Bounce the incoming emails if the group is closed. |
| `action_join` | user action | self | `mail_group` |  |  |
| `action_leave` | user action | self | `mail_group` |  |  |
| `_join_group` | internal rule | self, email, partner_id | `mail_group` |  |  |
| `_leave_group` | internal rule | self, email, partner_id, all_members | `mail_group` |  | Remove the given email / partner from the group.  If the "all_members" parameter is set to True, remove all members with the given email address (multiple members might have the same email address).  Otherwise, remove the most appropriate. |
| `_send_subscribe_confirmation_email` | internal rule | self, email | `mail_group` |  | Send an email to the given address to subscribe / unsubscribe to the mailing list. |
| `_send_unsubscribe_confirmation_email` | internal rule | self, email | `mail_group` |  | Send an email to the given address to subscribe / unsubscribe to the mailing list. |
| `_generate_action_url` | internal rule | self, email, action | `mail_group` |  | Generate the confirmation URL to subscribe / unsubscribe from the mailing list. |
| `_generate_action_token` | internal rule | self, email, action | `mail_group` |  | Generate an action token to be able to subscribe / unsubscribe from the mailing list. |
| `_generate_email_access_token` | internal rule | self, email | `mail_group` |  | Generate an action token to be able to unsubscribe from the mailing list, while hashing the target email to avoid spoofind other emails.  :param str email: email included in hash, should be normalized |
| `_generate_group_access_token` | internal rule | self | `mail_group` |  | Generate an action token to be able to subscribe / unsubscribe from the mailing list. |
| `_get_email_unsubscribe_url` | preparation rule | self, email_to | `mail_group` |  |  |
| `_find_member` | internal rule | self, email, partner_id | `mail_group` |  | Return the <mail.group.member> corresponding to the given email address. |
| `_find_members` | internal rule | self, email, partner_id | `mail_group` |  | Get all the members record corresponding to the email / partner_id.  Can be called in batch and return a dictionary     {'group_id': <mail.group.member>}  Multiple members might have the same email address, but with different partner because there's no unique constraint on the email field of the <res.partner> model.  When a partner is given for the search, return in priority - The member whose partner match the given partner - The member without partner but whose email match the given email  When no partner is given for the search, return in priority - A member whose email match the given emai |
| `action_go_to_website` | user action | self | `website_mail_group` |  |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_moderator_email` | ValidationError | Moderators must have an email address. | `mail_group` |
| `_check_moderation_notify` | ValidationError | The notification message is missing. | `mail_group` |
| `_check_moderation_guidelines` | ValidationError | The guidelines description is missing. | `mail_group` |
| `_check_moderator_existence` | ValidationError | Moderated group must have moderators. | `mail_group` |
| `_check_access_mode` | ValidationError | The "Authorized Group" is missing. | `mail_group` |
| `action_send_guidelines` | UserError | Only an administrator or a moderator can send guidelines to group members. | `mail_group` |
| `action_send_guidelines` | UserError | The guidelines description is empty. | `mail_group` |
| `action_send_guidelines` | UserError | You can not send guidelines for a closed group. | `mail_group` |
| `action_send_guidelines` | UserError | Template "mail_group.mail_template_guidelines" was not found. No email has been sent. Please contact an administrator to fix this issue. | `mail_group` |
| `_notify_members` | UserError | The group of the message do not match. | `mail_group` |
| `action_join` | UserError | You can not join a closed group. | `mail_group` |
| `_join_group` | ValidationError | The partner can not be found. | `mail_group` |
| `_generate_action_token` | UserError | Email %s is invalid | `mail_group` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `mail_group` |
| `base.group_portal` | no | yes | no | no | `mail_group` |
| `base.group_user` | yes | yes | yes | yes | `mail_group` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mail Group: Access only public and joined groups | `[(4, ref('base.group_user')), (4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[             '\|',             '\|',             '\|',                 ('moderator_ids', 'in', user.id),                 ('access_mode', '=', 'public'),                 '&',                     ('access_mode', '=', 'groups'),                     ('access_group_id', 'in', user.all_group_ids.ids),                 '&',                     ('access_mode', '=', 'members'),                     ('member_partner_ids', 'in', [user.partner_id.id]),             ]` | True | False | False | False |
| Mail Group: Moderator have write access on their group | `[(4, ref('base.group_user'))]` | `[('moderator_ids', 'in', user.id)]` | False | True | False | True |
| Mail Group: Administrator have access to all mail group | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_view_list` | list |  | `name`, `member_count`, `mail_group_message_count`, `mail_group_message_moderation_count` |  |  | `mail_group` |
| `mail_group.mail_group_view_kanban` | kanban |  | `is_member`, `image_128`, `name`, `description`, `mail_group_message_moderation_count` | `action_join`, `action_leave` |  | `mail_group` |
| `mail_group.mail_group_view_form` | form |  | `member_count`, `mail_group_message_count`, `mail_group_message_moderation_count`, `moderation_rule_count`, `image_128`, `name`, `active`, `alias_id`, `alias_name`, `alias_domain_id`, `description`, `moderation`, `moderator_ids`, `is_moderator`, `can_manage_group`, `is_member`, `access_mode`, `access_group_id`, `alias_contact`, `moderation_notify`, `moderation_notify_msg`, `moderation_guidelines`, `moderation_guidelines_msg` | `Join`, `Leave`, `%(mail_group.mail_group_member_action)d`, `%(mail_group.mail_group_message_action)d`, `%(mail_group.mail_group_message_action)d`, `%(mail_group.mail_group_moderation_action)d`, `Choose or configure a custom domain` |  | `mail_group` |
| `mail_group.mail_group_view_search` | search |  | `name`, `alias_email` |  | `Archived`, `Moderated`, `Moderation` | `mail_group` |
| `website_mail_group.mail_group_view_form` | xpath | `mail_group.mail_group_view_form` |  | `action_go_to_website` |  | `website_mail_group` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_action` | Mail Groups | kanban,list,form |  |  |  | `mail_group` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website_mail_group.mail_group_menu_website` | Mailing Lists | `website_mail_group.mail_group_menu_website_root` | `mail_group.mail_group_action` | 50 |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `mail_group.ir_cron_mail_notify_group_moderators` | Mail List: Notify group moderators | 1 days | `_cron_notify_moderators` | 1000 |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `mail_group.mail_template_list_subscribe` | Mail Group: Mailing List Subscription | Confirm subscription to {{ object.name }} |
| `mail_group.mail_template_list_unsubscribe` | Mail Group: Mailing List Unsubscription | Confirm unsubscription to {{ object.name }} |

Machine-readable definition: `../../../schemas/data/entities/mail.group.json`; views: `../../../schemas/interfaces/views/mail.group.json`.
