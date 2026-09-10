# User, Change Password Wizard (`change.password.user`)

**Transport name:** `change.password.user`  
**Storage name:** `change_password_user`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: User, Change Password Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `change.password.wizard` | required; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | required; on delete of the target: cascade |
| `user_login` | User Login | single line text |  | read only |
| `new_passwd` | New Password | single line text |  | default  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `change_password_button` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| change user password rule | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_password_policy.change_password_multi` | xpath | `base.change_password_wizard_user_tree_view` |  |  |  | `auth_password_policy` |
| `base.change_password_wizard_user_tree_view` | list |  | `user_id`, `user_login`, `new_passwd` |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/change.password.user.json`; views: `../../../schemas/interfaces/views/change.password.user.json`.
