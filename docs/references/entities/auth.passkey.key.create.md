# Create a Passkey (`auth.passkey.key.create`)

**Transport name:** `auth.passkey.key.create`  
**Storage name:** `auth_passkey_key_create`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `auth_passkey`

Description: Create a Passkey

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `make_key` | operation | self, registration | `auth_passkey` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `auth_passkey` |
| `base.group_portal` | yes | yes | yes | yes | `auth_passkey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Passkeys: Users can only modify their own Passkey creation requests | `[             (4, ref('base.group_portal')),             (4, ref('base.group_user')),         ]` | `[('create_uid', '=', user.id)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `auth_passkey.auth_passkey_key_create_view_form` | form |  | `name` | `Create`, `Cancel` |  | `auth_passkey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `auth_passkey.action_auth_passkey_key_create` | Create Passkey Wizard | form |  |  | new | `auth_passkey` |

Machine-readable definition: `../../../schemas/data/entities/auth.passkey.key.create.json`; views: `../../../schemas/interfaces/views/auth.passkey.key.create.json`.
