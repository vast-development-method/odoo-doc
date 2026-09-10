# Passkey (`auth.passkey.key`)

**Transport name:** `auth.passkey.key`  
**Storage name:** `auth_passkey_key`  
**Kind:** persistent entity (one table)  
**Defined by package:** `auth_passkey`

Description: Passkey

## Identity and behavior

- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `credential_identifier` | Credential Identifier | single line text |  | required; visible only to groups `base.group_system` |
| `public_key` | Public Key | single line text |  | required; computed by rule `_compute_public_key` (not stored); writable through an inverse rule; visible only to groups `base.group_system` |
| `sign_count` | Sign Count | integer |  | default ; visible only to groups `base.group_system` |
| `create_uid` | Create Uid | many to one | `res.users` | indexed |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_identifier` | Constraint | `UNIQUE(credential_identifier)` | The credential identifier should be unique. | `auth_passkey` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `auth_passkey` |  |  |
| `unlink` | lifecycle override | self | `auth_passkey` |  |  |
| `_compute_public_key` | computation | self | `auth_passkey` |  |  |
| `_inverse_public_key` | inverse computation | self | `auth_passkey` |  |  |
| `_get_session_challenge` | preparation rule | self | `auth_passkey` | model |  |
| `_start_auth` | internal rule | self | `auth_passkey` | model |  |
| `_verify_auth` | internal rule | self, auth, public_key, sign_count | `auth_passkey` | model |  |
| `_start_registration` | internal rule | self | `auth_passkey` | model |  |
| `_verify_registration_options` | internal rule | self, registration | `auth_passkey` | model |  |
| `action_delete_passkey` | user action | self | `auth_passkey` |  |  |
| `action_rename_passkey` | user action | self | `auth_passkey` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_session_challenge` | AccessDenied | Cannot find a challenge for this session | `auth_passkey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | yes | no | `auth_passkey` |
| `base.group_portal` | no | yes | yes | no | `auth_passkey` |
| `base.group_erp_manager` | no | yes | yes | yes | `auth_passkey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Passkeys: Users can only access own Passkeys | `[             (4, ref('base.group_portal')),             (4, ref('base.group_user')),         ]` | `[('create_uid', '=', user.id)]` | True | True | True | True |
| Passkeys: Admins can view and delete other peoples Passkeys | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | 1 | 0 | 0 | 1 |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_key_view_kanban` | kanban |  | `name`, `create_date`, `write_date` |  |  | `auth_passkey` |
| `auth_passkey.auth_passkey_key_rename` | form |  | `name` | `Save`, `Cancel` |  | `auth_passkey` |

Machine-readable definition: `../../../schemas/data/entities/auth.passkey.key.json`; views: `../../../schemas/interfaces/views/auth.passkey.key.json`.
