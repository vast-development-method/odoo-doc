# Bank Account Allocation Line (Wizard) (`hr.bank.account.allocation.wizard.line`)

**Transport name:** `hr.bank.account.allocation.wizard.line`  
**Storage name:** `hr_bank_account_allocation_wizard_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr`

Description: Bank Account Allocation Line (Wizard)

## Identity and behavior

- Default ordering: `sequence, id`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `wizard_id` | Wizard | many to one | `hr.bank.account.allocation.wizard` | required; on delete of the target: cascade |
| `bank_account_id` | Bank Account | many to one | `res.partner.bank` | required; read only |
| `acc_number` | Acc Number | single line text |  | read only; related through path `bank_account_id.acc_number` |
| `amount` | Amount | float |  | precision `[16, 2]` |
| `amount_type` | Amount Type | selection |  | values provided by rule `_get_amount_type_selection_vals` |
| `symbol` | Symbol | single line text |  | read only; computed by rule `_compute_symbol` (not stored) |
| `trusted` | Trusted | boolean |  |  |
| `sequence` | Sequence | integer |  | default `10` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_symbol` | computation | self | `hr` | depends: `amount_type`, `bank_account_id.symbol` |  |
| `_get_amount_type_selection_vals` | preparation rule | self | `hr` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_bank_account_allocation_line_list` | list |  | `sequence`, `acc_number`, `amount`, `symbol`, `amount_type`, `trusted` |  |  | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.bank.account.allocation.wizard.line.json`; views: `../../../schemas/interfaces/views/hr.bank.account.allocation.wizard.line.json`.
