# Mail Gateway Allowed (`mail.gateway.allowed`)

**Transport name:** `mail.gateway.allowed`  
**Storage name:** `mail_gateway_allowed`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: Mail Gateway Allowed

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email` | Email Address | single line text |  | required |
| `email_normalized` | Normalized Email | single line text |  | computed by rule `_compute_email_normalized` and stored; indexed |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_email_normalized` | computation | self | `mail` | depends: `email` |  |
| `get_empty_list_help` | operation | self, help_message | `mail` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_gateway_allowed_view_tree` | list |  | `email` |  |  | `mail` |
| `mail.mail_gateway_allowed_view_search` | search |  | `email` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_gateway_allowed_action` | Mail Gateway Allowed | list |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.gateway.allowed.json`; views: `../../../schemas/interfaces/views/mail.gateway.allowed.json`.
