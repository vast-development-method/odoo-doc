# Mailing List black/white list (`mail.group.moderation`)

**Transport name:** `mail.group.moderation`  
**Storage name:** `mail_group_moderation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail_group`

Description: Mailing List black/white list

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email` | Email | single line text |  | required |
| `status` | Status | selection |  | required; default `ban` |
| `mail_group_id` | Group | many to one | `mail.group` | required; indexed; on delete of the target: cascade |

## Selection values

### `status` (Status)

| Value | Label |
|---|---|
| `allow` | Always Allow |
| `ban` | Permanent Ban |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_mail_group_email_uniq` | Constraint | `UNIQUE(mail_group_id, email)` | You can create only one rule for a given email address in a group. | `mail_group` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail_group` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail_group` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | UserError | Invalid email address “%s” | `mail_group` |
| `write` | UserError | Invalid email address “%s” | `mail_group` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail_group` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mail Group Moderation: Moderation rules are accessible only by moderators | `[(4, ref('base.group_user'))]` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | True | True | True | True |
| Mail Group Moderation: Administrator have access to all moderation rules | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_moderation_view_tree` | list |  | `mail_group_id`, `email`, `status` |  |  | `mail_group` |
| `mail_group.mail_group_moderation_view_search` | search |  | `mail_group_id`, `email`, `status` |  | `Is Banned`, `Is Allowed`, `Status` | `mail_group` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_moderation_action` | Moderation | list,form |  |  |  | `mail_group` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website_mail_group.mail_group_moderation_menu_website` | Moderation Rules | `website_mail_group.mail_group_menu_website_root` | `mail_group.mail_group_moderation_action` | 51 | `mail_group.group_mail_group_manager` |

Machine-readable definition: `../../../schemas/data/entities/mail.group.moderation.json`; views: `../../../schemas/interfaces/views/mail.group.moderation.json`.
