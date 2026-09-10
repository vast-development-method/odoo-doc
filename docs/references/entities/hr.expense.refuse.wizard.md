# Expense Refuse Reason Wizard (`hr.expense.refuse.wizard`)

**Transport name:** `hr.expense.refuse.wizard`  
**Storage name:** `hr_expense_refuse_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_expense`

Description: Expense Refuse Reason Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `reason` | Reason | single line text |  | required |
| `expense_ids` | Expense | many to many | `hr.expense` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `hr_expense` | model |  |
| `action_refuse` | user action | self | `hr_expense` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_expense.group_hr_expense_team_approver` | yes | yes | yes | no | `hr_expense` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_refuse_wizard_view_form` | form |  | `expense_ids`, `reason` | `Refuse`, `Cancel` |  | `hr_expense` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_refuse_wizard_action` | Refuse Expense | form |  | `{'dialog_size': 'medium'}` | new | `hr_expense` |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.refuse.wizard.json`; views: `../../../schemas/interfaces/views/hr.expense.refuse.wizard.json`.
