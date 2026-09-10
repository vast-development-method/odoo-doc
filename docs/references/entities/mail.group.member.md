# Mailing List Member (`mail.group.member`)

**Transport name:** `mail.group.member`  
**Storage name:** `mail_group_member`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail_group`

Description: Mailing List Member

## Identity and behavior

- Display name field: `email`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email` | Email | single line text |  | computed by rule `_compute_email` and stored |
| `email_normalized` | Normalized Email | single line text |  | computed by rule `_compute_email_normalized` and stored; indexed |
| `mail_group_id` | Group | many to one | `mail.group` | required; indexed; on delete of the target: cascade |
| `partner_id` | Partner | many to one | `res.partner` | on delete of the target: cascade |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_partner` | Constraint | `UNIQUE(partner_id, mail_group_id)` | This partner is already subscribed to the group | `mail_group` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_email` | computation | self | `mail_group` | depends: `partner_id.email` |  |
| `_compute_email_normalized` | computation | self | `mail_group` | depends: `email` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail_group` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mail Group Member: Members are accessible only by moderators | `[(4, ref('base.group_user'))]` | `[('mail_group_id.moderator_ids', 'in', user.id)]` | True | True | True | True |
| Mail Group Member: Administrator have access to all members | `[(4, ref('mail_group.group_mail_group_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_member_view_tree` | list |  | `email`, `email_normalized`, `partner_id`, `mail_group_id` | `Email` |  | `mail_group` |
| `mail_group.mail_group_member_view_search` | search |  | `email`, `partner_id`, `mail_group_id` |  |  | `mail_group` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_group.mail_group_member_action` | Members | list |  |  |  | `mail_group` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `mail_group.mail_template_guidelines` | Mail Group: Send Guidelines | Guidelines of group {{ object.mail_group_id.name }} |

Machine-readable definition: `../../../schemas/data/entities/mail.group.member.json`; views: `../../../schemas/interfaces/views/mail.group.member.json`.
