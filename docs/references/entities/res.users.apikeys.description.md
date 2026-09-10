# application programming interface Key Description (`res.users.apikeys.description`)

**Transport name:** `res.users.apikeys.description`  
**Storage name:** `res_users_apikeys_description`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `portal`

Description: API Key Description

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | single line text |  | required |
| `duration` | Duration | selection |  | required; default computed dynamically (lambda self: self._selection_duration()[0][0]); values provided by rule `_selection_duration` |
| `expiration_date` | Expiration Date | date and time |  | computed by rule `_compute_expiration_date` and stored |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_selection_duration` | internal rule | self | `base` |  |  |
| `_compute_expiration_date` | computation | self | `base` | depends: `duration` |  |
| `_onchange_expiration_date` | on change | self | `base` | onchange: `expiration_date` |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `make_key` | operation | self | `base` |  |  |
| `check_access_make_key` | operation | self | `base`, `portal` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_access_make_key` | AccessError | Only internal users can create API keys | `base` |
| `check_access_make_key` | AccessError | Only internal and portal users can create API keys | `portal` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | yes | yes | no | no | `base` |
| `group_portal` | yes | yes | no | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.form_res_users_key_description` | form |  | `name`, `duration`, `expiration_date` | `Generate key`, `Cancel` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_users_keys_description` | API Key: description input wizard | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/res.users.apikeys.description.json`; views: `../../../schemas/interfaces/views/res.users.apikeys.description.json`.
