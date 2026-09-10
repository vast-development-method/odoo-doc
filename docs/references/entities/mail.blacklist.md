# Mail Blacklist (`mail.blacklist`)

**Transport name:** `mail.blacklist`  
**Storage name:** `mail_blacklist`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `mass_mailing`

Description: Mail Blacklist

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Display name field: `email`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email` | Email Address | single line text |  | required; changes are tracked in the message thread; indexed (trigram); Help: This field is case insensitive. |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `opt_out_reason_id` | Opt-out Reason | many to one | `mailing.subscription.optout` | changes are tracked in the message thread; on delete of the target: restrict |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_email` | Constraint | `unique (email)` | Email address already exists! | `mail` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `_search` | search rule | self, domain, *args, **kwargs | `mail` |  | Override _search in order to grep search on email field and make it lower-case and sanitized |
| `_add` | internal rule | self, email, message | `mail` |  |  |
| `_remove` | internal rule | self, email, message | `mail` |  |  |
| `mail_action_blacklist_remove` | operation | self | `mail` |  |  |
| `action_add` | user action | self | `mail` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `mass_mailing` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | UserError | Invalid email address “%s” | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_blacklist_view_tree` | list |  | `create_date`, `email` |  |  | `mail` |
| `mail.mail_blacklist_view_form` | form |  | `email` | `Unblacklist`, `Blacklist` |  | `mail` |
| `mail.mail_blacklist_view_search` | search |  | `email` |  | `Archived` | `mail` |
| `mass_mailing.mail_blacklist_view_tree` | xpath | `mail.mail_blacklist_view_tree` | `opt_out_reason_id` |  |  | `mass_mailing` |
| `mass_mailing.mail_blacklist_view_form` | xpath | `mail.mail_blacklist_view_form` | `opt_out_reason_id` |  |  | `mass_mailing` |
| `mass_mailing.mail_blacklist_view_search` | xpath | `mail.mail_blacklist_view_search` |  |  | `Reason` | `mass_mailing` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_blacklist_action` | Blacklisted Email Addresses |  |  |  |  | `mail` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `mass_mailing.mail_blacklist_mm_menu` | Blacklisted Email Addresses | `mass_mailing_configuration` | `mail.mail_blacklist_action` | 20 |  |

Machine-readable definition: `../../../schemas/data/entities/mail.blacklist.json`; views: `../../../schemas/interfaces/views/mail.blacklist.json`.
