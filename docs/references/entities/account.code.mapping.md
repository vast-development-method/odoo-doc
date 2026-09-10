# Mapping of account codes per company (`account.code.mapping`)

**Transport name:** `account.code.mapping`  
**Storage name:** `account_code_mapping`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Mapping of account codes per company

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Account | many to one | `account.account` | computed by rule `_compute_account_id` (not stored); searchable through a search rule |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` (not stored) |
| `code` | Code | single line text |  | computed by rule `_compute_code` (not stored); writable through an inverse rule |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `_search` | search rule | self, domain, offset, limit, order, **kw | `account` |  |  |
| `_compute_account_id` | computation | self | `account` |  |  |
| `_compute_company_id` | computation | self | `account` |  |  |
| `_compute_code` | computation | self | `account` | depends: `account_id.code` |  |
| `_inverse_code` | inverse computation | self | `account` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search` | UserError | Account Code Mapping cannot be accessed directly. It is designed to be used only through the Chart of Accounts. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | no | yes | yes | no | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.code.mapping.json`.
