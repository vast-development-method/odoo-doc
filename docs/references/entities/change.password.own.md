# User, change own password wizard (`change.password.own`)

**Transport name:** `change.password.own`  
**Storage name:** `change_password_own`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: User, change own password wizard

## Identity and behavior

- Transient maximum hours: 0.1

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `new_password` | New Password | single line text |  |  |
| `confirm_password` | New Password (Confirmation) | single line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_password_confirmation` | validation | self | `base` | constrains: `new_password`, `confirm_password` |  |
| `change_password` | operation | self | `base` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_password_confirmation` | ValidationError | The new password and its confirmation must be identical. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| change own password | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_password_policy.change_password` | xpath | `base.change_password_own_form` |  |  |  | `auth_password_policy` |
| `base.change_password_own_form` | form |  | `new_password`, `confirm_password` | `Change Password`, `Cancel` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/change.password.own.json`; views: `../../../schemas/interfaces/views/change.password.own.json`.
