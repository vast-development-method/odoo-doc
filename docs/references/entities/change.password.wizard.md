# Change Password Wizard (`change.password.wizard`)

**Transport name:** `change.password.wizard`  
**Storage name:** `change_password_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Change Password Wizard

## Identity and behavior

- Transient maximum hours: 0.2

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_ids` | Users | one to many | `change.password.user` | default computed dynamically (_default_user_ids); inverse field `wizard_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_user_ids` | preparation rule | self | `base` |  |  |
| `change_password_button` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.change_password_wizard_view` | form |  | `user_ids` | `Change Password`, `Cancel` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.change_password_wizard_action` | Change Password | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/change.password.wizard.json`; views: `../../../schemas/interfaces/views/change.password.wizard.json`.
