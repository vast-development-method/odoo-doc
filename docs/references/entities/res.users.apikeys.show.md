# Show application programming interface Key (`res.users.apikeys.show`)

**Transport name:** `res.users.apikeys.show`  
**Storage name:** `res_users_apikeys_show`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Show API Key

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `id` | Identifier | identifier |  |  |
| `key` | Key | single line text |  | read only |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | no | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.form_res_users_key_show` | form |  | `key` | `Done!` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.users.apikeys.show.json`; views: `../../../schemas/interfaces/views/res.users.apikeys.show.json`.
