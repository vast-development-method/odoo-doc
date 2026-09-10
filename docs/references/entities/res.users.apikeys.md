# Users application programming interface Keys (`res.users.apikeys`)

**Transport name:** `res.users.apikeys`  
**Storage name:** `res_users_apikeys`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Users API Keys

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required; read only |
| `user_id` | User | many to one | `res.users` | required; read only; indexed; on delete of the target: cascade |
| `scope` | Scope | single line text |  | read only |
| `create_date` | Creation Date | date and time |  | read only |
| `expiration_date` | Expiration Date | date and time |  | read only |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `base` |  |  |
| `remove` | operation | self | `base` |  |  |
| `_remove` | internal rule | self | `base` |  | Use the remove() method to remove an API Key. This method implement logic, but won't check the identity (mainly used to remove trusted devices) |
| `_check_credentials` | validation | self, scope, key | `base` |  |  |
| `_check_expiration_date` | validation | self, date | `base` |  |  |
| `_generate` | internal rule | self, scope, name, expiration_date | `base` |  | Generates an api key. :param str\|None scope: the scope of the key. If None, the key will give access to any rpc. :param str name: the name of the key, mainly intended to be displayed in the UI. :param datetime.datetime expiration_date: the expiration date of the key. :returns: the key. :rtype: str  Note: This method must be called in sudo to use a duration greater than that allowed by the user's privileges. For a persistent key (infinite duration), no value for expiration date. |
| `_ensure_can_manage_keys_programmatically` | internal rule | self | `base` |  |  |
| `generate` | operation | self, key, scope, name, expiration_date | `base` | model | Generate a new API key with an existing API key. The provided `key` must be an existing API key that belongs to the current user. Its scope must be compatible with `scope`. The `expiration_date` must be allowed for the user's group.  To renew a key, generate the new one, store it, and then call `revoke` on the previous one.  :param str key: an active API key belonging to the current user :param str\|None scope: the scope of the key. If None, the key will give access to any rpc. :param str name: the name of the key, mainly intended to be displayed in the UI. :param str\|datetime.datetime expira |
| `revoke` | operation | self, key | `base` | model | Revoke an existing API key. If it exists, the `key` will be removed from the server.  :param str key: the API key that has to be revoked |
| `_gc_user_apikeys` | background operation | self | `base` | autovacuum |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_remove` | AccessError | You can not remove API keys unless they're yours or you are a system user | `base` |
| `_check_expiration_date` | ValidationError | The API key must have an expiration date | `base` |
| `_check_expiration_date` | ValidationError | You cannot exceed %(duration)s days. | `base` |
| `_check_expiration_date` | ValidationError | You cannot set an expiration date in the past. | `base` |
| `_ensure_can_manage_keys_programmatically` | UserError | Programmatic API keys are not enabled | `base` |
| `generate` | UserError | Limit of %s API keys is reached for programmatic creation | `base` |
| `generate` | AccessDenied | The provided API key is invalid or does not belong to the current user. | `base` |
| `revoke` | AccessDenied | The provided API key is invalid. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_portal` | no | yes | no | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Public users can't interact with keys at all | `[Command.link(ref('base.group_public'))]` | `[(0, '=', 1)]` | True | True | True | True |
| Users can read and delete their own keys | `[                 Command.link(ref('base.group_portal')),                 Command.link(ref('base.group_user')),             ]` | `[('user_id', '=', user.id)]` | True | True | True | True |
| Administrators can view user keys to revoke them | `[Command.link(ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_apikeys` | list |  | `user_id`, `name`, `scope`, `create_date` | `remove` |  | `base` |
| `base.res_users_apikeys_view_kanban` | kanban |  | `name`, `create_date`, `scope`, `expiration_date` | `Delete` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_apikeys_admin` | API Keys Listing | list |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.users.apikeys.json`; views: `../../../schemas/interfaces/views/res.users.apikeys.json`.
