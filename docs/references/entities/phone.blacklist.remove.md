# Remove phone from blacklist (`phone.blacklist.remove`)

**Transport name:** `phone.blacklist.remove`  
**Storage name:** `phone_blacklist_remove`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `phone_validation`

Description: Remove phone from blacklist

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `phone` | Phone Number | single line text |  | required; read only |
| `Reason` | Reason | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_unblacklist_apply` | user action | self | `phone_validation` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | yes | `mass_mailing_sms` |
| `base.group_system` | yes | yes | yes | yes | `phone_validation` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `phone_validation.phone_blacklist_remove_view_form` | form |  | `phone`, `reason` | `Remove phone from blacklist`, `Discard` |  | `phone_validation` |

Machine-readable definition: `../../../schemas/data/entities/phone.blacklist.remove.json`; views: `../../../schemas/interfaces/views/phone.blacklist.remove.json`.
