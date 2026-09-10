# Accounts Mapping of Fiscal Position (`account.fiscal.position.account`)

**Transport name:** `account.fiscal.position.account`  
**Storage name:** `account_fiscal_position_account`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Accounts Mapping of Fiscal Position

## Identity and behavior

- Display name field: `position_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `position_id` | Fiscal Position | many to one | `account.fiscal.position` | required; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | related through path `position_id.company_id` and stored |
| `account_src_id` | Account on Product | many to one | `account.account` | required; must belong to the same company |
| `account_dest_id` | Account to Use Instead | many to one | `account.account` | required; must belong to the same company |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_account_src_dest_uniq` | Constraint | `unique (position_id,account_src_id,account_dest_id)` | An account fiscal position could be defined only one time on same accounts. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `base.group_user` | no | yes | no | no | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.fiscal.position.account.json`.
