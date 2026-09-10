# Authentication Device (`auth_totp.device`)

**Transport name:** `auth_totp.device`  
**Storage name:** `auth_totp_device`  
**Kind:** persistent entity (one table)  
**Defined by package:** `auth_totp`  
**Extended by packages:** `auth_totp_mail`, `auth_timeout`

Description: Authentication Device

## Identity and behavior

- Mixins (classical inheritance): `res.users.apikeys`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_credentials_for_uid` | validation | self, scope, key, uid | `auth_totp` |  | Return True if device key matches given `scope` for user ID `uid` |
| `_get_trusted_device_age` | preparation rule | self | `auth_timeout`, `auth_totp` |  |  |
| `unlink` | lifecycle override | self | `auth_totp_mail` |  | Notify users when trusted devices are removed from their account. |
| `_classify_by_user` | internal rule | self | `auth_totp_mail` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `auth_totp` |
| `base.group_portal` | no | yes | no | no | `auth_totp` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Public users can't interact with keys at all | `[Command.link(ref('base.group_public'))]` | `[(0, '=', 1)]` | True | True | True | True |
| Users can read and delete their own keys | `[                 Command.link(ref('base.group_portal')),                 Command.link(ref('base.group_user')),             ]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Administrators can view user keys to revoke them | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/auth_totp.device.json`.
