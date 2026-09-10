# Expense Approve Duplicate (`hr.expense.approve.duplicate`)

**Transport name:** `hr.expense.approve.duplicate`  
**Storage name:** `hr_expense_approve_duplicate`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_expense`

Description: Expense Approve Duplicate

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `expense_ids` | Expense | many to many | `hr.expense` | read only |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_expense` | model |  |
| `action_approve` | user action | self | `hr_expense` |  |  |
| `action_refuse` | user action | self | `hr_expense` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | no | `hr_expense` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_approve_duplicate_view_form` | form |  | `expense_ids`, `date`, `employee_id`, `product_id`, `total_amount`, `name`, `manager_id`, `approval_date` | `Refuse`, `Approve`, `Cancel` |  | `hr_expense` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_approve_duplicate_action` | Validate Duplicate Expenses | form |  |  | new | `hr_expense` |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.approve.duplicate.json`; views: `../../../schemas/interfaces/views/hr.expense.approve.duplicate.json`.
