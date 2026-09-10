# Account move line to be created when posting work in progress account move (`mrp.account.wip.accounting.line`)

**Transport name:** `mrp.account.wip.accounting.line`  
**Storage name:** `mrp_account_wip_accounting_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp_account`

Description: Account move line to be created when posting WIP account move

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Account | many to one | `account.account` |  |
| `label` | Label | single line text |  |  |
| `debit` | Debit | monetary |  | computed by rule `_compute_debit` and stored |
| `credit` | Credit | monetary |  | computed by rule `_compute_credit` and stored |
| `currency_id` | Currency | many to one | `res.currency` | default computed dynamically (lambda self: self.env.company.currency_id) |
| `wip_accounting_id` | work in progress accounting wizard | many to one | `mrp.account.wip.accounting` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_debit_credit` | Constraint | `CHECK ( debit = 0 OR credit = 0 )` | A single line cannot be both credit and debit. | `mrp_account` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_debit` | computation | self | `mrp_account` | depends: `credit` |  |
| `_compute_credit` | computation | self | `mrp_account` | depends: `debit` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `mrp_account` |

Machine-readable definition: `../../../schemas/data/entities/mrp.account.wip.accounting.line.json`.
