# Remove email from blacklist wizard (`mail.blacklist.remove`)

**Transport name:** `mail.blacklist.remove`  
**Storage name:** `mail_blacklist_remove`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`

Description: Remove email from blacklist wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `Email` | Email | single line text |  | required; read only |
| `Reason` | Reason | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_unblacklist_apply` | user action | self | `mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_blacklist_remove_view_form` | form |  | `email`, `reason` | `Remove address from blacklist`, `Discard` |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.blacklist.remove.json`; views: `../../../schemas/interfaces/views/mail.blacklist.remove.json`.
