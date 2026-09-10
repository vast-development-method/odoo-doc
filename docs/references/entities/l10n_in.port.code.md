# Indian port code (`l10n_in.port.code`)

**Transport name:** `l10n_in.port.code`  
**Storage name:** `l10n_in_port_code`  
**Kind:** persistent entity (one table)  
**Defined by package:** `l10n_in`

Description: Indian port code

## Identity and behavior

- Display name field: `code`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Port Code | single line text |  | required |
| `name` | Port | single line text |  | required |
| `state_id` | State | many to one | `res.country.state` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code_uniq` | Constraint | `unique (code)` | The Port Code must be unique! | `l10n_in` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `l10n_in` |
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_in` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_in.l10n_in_port_code_form_view` | form |  | `name`, `code`, `state_id` |  |  | `l10n_in` |
| `l10n_in.l10n_in_port_code_tree_view` | list |  | `name`, `code`, `state_id` |  |  | `l10n_in` |
| `l10n_in.l10n_in_port_code_search_view` | search |  | `name`, `state_id` |  | `State` | `l10n_in` |

Machine-readable definition: `../../../schemas/data/entities/l10n_in.port.code.json`; views: `../../../schemas/interfaces/views/l10n_in.port.code.json`.
