# Bank Account Allocation Wizard (`hr.bank.account.allocation.wizard`)

**Transport name:** `hr.bank.account.allocation.wizard`  
**Storage name:** `hr_bank_account_allocation_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr`

Description: Bank Account Allocation Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | required |
| `allocation_ids` | Allocations | one to many | `hr.bank.account.allocation.wizard.line` | inverse field `wizard_id` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_prepare_allocations_from_employee` | preparation rule | self | `hr` |  |  |
| `create` | lifecycle override | self, vals_list | `hr` | model_create_multi |  |
| `action_save` | user action | self | `hr` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_prepare_allocations_from_employee` | ValidationError | Bank account %s not found within the salary distribution of the employee | `hr` |
| `action_save` | ValidationError | Total percentage allocation must equal 100%. | `hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | no | `hr` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_bank_account_allocation_wizard` | form |  | `employee_id`, `allocation_ids` | `Save`, `Cancel` |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.action_bank_account_allocation_wizard` | Bank Account Allocations | form |  | `{'active_id': active_id}` | new | `hr` |

Machine-readable definition: `../../../schemas/data/entities/hr.bank.account.allocation.wizard.json`; views: `../../../schemas/interfaces/views/hr.bank.account.allocation.wizard.json`.
