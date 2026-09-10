# 2-Factor Setup Wizard (`auth_totp.wizard`)

**Transport name:** `auth_totp.wizard`  
**Storage name:** `auth_totp_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `auth_totp`

Description: 2-Factor Setup Wizard

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; read only |
| `secret` | Secret | single line text |  | required; read only |
| `url` | Uniform resource locator | single line text |  | read only; computed by rule `_compute_qrcode` and stored |
| `qrcode` | Qrcode | binary |  | read only; computed by rule `_compute_qrcode` and stored |
| `code` | Verification Code | single line text |  | maximum length 7 |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_qrcode` | computation | self | `auth_totp` | depends: `user_id.login`, `user_id.company_id.display_name`, `secret` |  |
| `enable` | operation | self | `auth_totp` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `enable` | UserError | Verification failed, please double-check the 6-digit code | `auth_totp` |
| `enable` | UserError | The verification code should only contain numbers | `auth_totp` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users can only access their own wizard | global (all users) | `[('user_id', '=', user.id)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_totp.view_totp_wizard` | form |  | `qrcode`, `secret`, `secret`, `code` | `Enable Two-Factor Authentication`, `Discard` |  | `auth_totp` |

Machine-readable definition: `../../../schemas/data/entities/auth_totp.wizard.json`; views: `../../../schemas/interfaces/views/auth_totp.wizard.json`.
