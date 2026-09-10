# Password Check Wizard (`res.users.identitycheck`)

**Transport name:** `res.users.identitycheck`  
**Storage name:** `res_users_identitycheck`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `auth_passkey`

Description: Password Check Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `request` | Request | single line text |  | read only; visible only to groups `{"expression": "fields.NO_ACCESS"}` |
| `auth_method` | Auth Method | selection |  | default computed dynamically (lambda self: self._get_default_auth_method()); extended by packages `auth_passkey` |
| `password` | Password | single line text |  |  |

## Selection values

### `auth_method` (Auth Method)

| Value | Label |
|---|---|
| `password` | Password |
| `webauthn` | Passkey |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_auth_method` | preparation rule | self | `auth_passkey`, `base` | model |  |
| `_check_identity` | validation | self | `auth_passkey`, `base` |  |  |
| `run_check` | operation | self | `base` |  |  |
| `action_use_password` | user action | self | `auth_passkey` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_identity` | UserError | Incorrect Password, try again or click on Forgot Password to reset your password. | `base` |
| `_check_identity` | UserError | Incorrect Passkey. Please provide a valid passkey or use a different authentication method. | `auth_passkey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | yes | no | `base` |
| `group_portal` | yes | yes | yes | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| users can only access their own id check | global (all users) | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_passkey.res_users_identitycheck_view_form_passkey` | xpath | `base.res_users_identitycheck_view_form` |  |  |  | `auth_passkey` |
| `base.res_users_identitycheck_view_form` | form |  | `password` | `Confirm Password`, `Discard` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.users.identitycheck.json`; views: `../../../schemas/interfaces/views/res.users.identitycheck.json`.
