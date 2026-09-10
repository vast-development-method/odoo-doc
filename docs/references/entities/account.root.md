# Account codes first 2 digits (`account.root`)

**Transport name:** `account.root`  
**Storage name:** `account_root`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Account codes first 2 digits

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_root` (not stored) |
| `parent_id` | Parent | many to one | `account.root` | computed by rule `_compute_root` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `browse` | operation | self, ids | `account` | private |  |
| `_search` | search rule | self, domain, offset, limit, order, **kw | `account` |  |  |
| `_from_account_code` | internal rule | self, code | `account` | model |  |
| `_compute_root` | computation | self | `account` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search` | UserError | Filter on the Account or its Display Name instead | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.root.json`.
