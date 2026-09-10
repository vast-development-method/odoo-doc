# time-based one-time password rate limit logs (`auth.totp.rate.limit.log`)

**Transport name:** `auth.totp.rate.limit.log`  
**Storage name:** `auth_totp_rate_limit_log`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `auth_totp`

Description: TOTP rate limit logs

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | required; read only |
| `ip` | Internet protocol | single line text |  | read only |
| `limit_type` | Limit Type | selection |  | read only |

## Selection values

### `limit_type` (Limit Type)

| Value | Label |
|---|---|
| `send_email` | Send Email |
| `code_check` | Code Checking |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_id_limit_type_create_date_idx` | Index | `(user_id, limit_type, create_date)` |  | `auth_totp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `auth_totp` |

Machine-readable definition: `../../../schemas/data/entities/auth.totp.rate.limit.log.json`.
