# Canned Response (`mail.canned.response`)

**Transport name:** `mail.canned.response`  
**Storage name:** `mail_canned_response`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Canned Response

## Identity and behavior

- Default ordering: `id desc`
- Display name field: `source`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `source` | Shortcut | single line text |  | required; indexed (trigram); Help: Canned response that will automatically be substituted with longer content in your messages. Type '::' followed by the name of your shortcut (e.g. ::hello) to use in your messages. |
| `substitution` | Substitution | multi line text |  | required; Help: Content that will automatically replace the shortcut of your choosing. This content can still be adapted before sending your message. |
| `last_used` | Last Used | date and time |  | Help: Last time this canned_response was used |
| `group_ids` | Authorized Groups | many to many | `res.groups` | restricted by domain `lambda self: [('id', 'in', self.env.user.all_group_ids.ids)]` |
| `is_shared` | Determines if the canned_response is currently shared with other users | boolean |  | computed by rule `_compute_is_shared` and stored |
| `is_editable` | Determines if the canned response can be edited by the current user | boolean |  | computed by rule `_compute_is_editable` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_shared` | computation | self | `mail` | depends: `group_ids` |  |
| `_compute_is_editable` | computation | self | `mail` | depends_context: `uid`; depends: `create_uid` |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_broadcast` | internal rule | delete | `mail` |  |  |
| `_to_store_defaults` | internal rule | self, target | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Canned response: admin has all access on shared canned response | `[Command.link(ref('group_mail_canned_response_admin'))]` | `[('is_shared', '=', True)]` | True | True | False | True |
| Canned response: User read: own or in groups | `[Command.link(ref('base.group_user'))]` | `['\|', ('create_uid', '=', user.id), ('group_ids', 'in', user.all_group_ids.ids)]` | True | False | False | False |
| Canned response: User write/unlink: own only | `[Command.link(ref('base.group_user'))]` | `[('create_uid', '=', user.id)]` | False | True | False | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_canned_response_view_search` | search |  | `source`, `substitution` |  | `Private`, `Shared`, `Authorized Groups` | `mail` |
| `mail.mail_canned_response_view_tree` | list |  | `source`, `substitution`, `create_uid`, `group_ids`, `last_used`, `is_editable`, `is_shared` |  |  | `mail` |
| `mail.mail_canned_response_view_form` | form |  | `source`, `substitution`, `group_ids`, `is_editable` |  |  | `mail` |
| `mail.mail_canned_response_view_kanban` | kanban |  | `source`, `substitution`, `group_ids` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_canned_response_action` | Canned Responses | list,form,kanban |  | `{}` |  | `mail` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `im_livechat.canned_responses` | Canned Responses | `livechat_config` | `mail.mail_canned_response_action` | 15 | `im_livechat_group_user` |
| `mail.menu_canned_responses` | Canned Responses | `mail.menu_configuration` | `mail.mail_canned_response_action` | 15 |  |

Machine-readable definition: `../../../schemas/data/entities/mail.canned.response.json`; views: `../../../schemas/interfaces/views/mail.canned.response.json`.
