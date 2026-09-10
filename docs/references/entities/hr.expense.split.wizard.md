# Expense Split Wizard (`hr.expense.split.wizard`)

**Transport name:** `hr.expense.split.wizard`  
**Storage name:** `hr_expense_split_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_expense`

Description: Expense Split Wizard

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `expense_id` | Expense | many to one | `hr.expense` | required |
| `expense_split_line_ids` | Expense Split Line | one to many | `hr.expense.split` | inverse field `wizard_id` |
| `total_amount_currency` | Total Amount | monetary |  | computed by rule `_compute_total_amount_currency` (not stored); currency taken from `currency_id` |
| `total_amount_currency_original` | Total amount original | monetary |  | related through path `expense_id.total_amount_currency`; currency taken from `currency_id`; Help: Total amount of the original Expense that we are splitting |
| `tax_amount_currency` | Taxes | monetary |  | computed by rule `_compute_tax_amount_currency` (not stored); currency taken from `currency_id` |
| `split_possible` | Split Possible | boolean |  | computed by rule `_compute_split_possible` (not stored); Help: The sum of after split shut remain the same |
| `currency_id` | Currency | many to one | `res.currency` | related through path `expense_id.currency_id` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_total_amount_currency` | computation | self | `hr_expense` | depends: `expense_split_line_ids.total_amount_currency` |  |
| `_compute_tax_amount_currency` | computation | self | `hr_expense` | depends: `expense_split_line_ids.tax_amount_currency` |  |
| `_compute_split_possible` | computation | self | `hr_expense` | depends: `total_amount_currency_original`, `total_amount_currency` |  |
| `action_split_expense` | user action | self | `hr_expense` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `hr_expense` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee Expense Split | `[(4, ref('base.group_user'))]` | `[                 ('expense_id.state', '=', 'draft'),                 '\|', ('expense_id.employee_id.user_id', '=', user.id), ('expense_id.manager_id', '=', user.id),             ]` | True | True | True | True |
| Approver Expense Split | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[                 ('expense_id.state', 'in', ['draft', 'submitted']),                 ('expense_id.manager_id', 'in', [user.id, False])             ]` | True | True | True | True |
| All approver Expense Split | `[(4, ref('hr_expense.group_hr_expense_user'))]` | `[                 ('expense_id.state', 'in', ['draft', 'submitted']),                 '\|', ('expense_id.employee_id.user_id', '!=', user.id), ('expense_id.manager_id', 'in', [user.id, False])             ]` | True | True | True | True |
| Manager Expense Split | `[(4, ref('hr_expense.group_hr_expense_manager'))]` | `[('expense_id.state', 'in', ['draft', 'submitted'])]` | True | True | True | True |
| Accountant Expense Split | `[(4, ref('account.group_account_invoice'))]` | `[('expense_id.state', 'in', ('draft', 'submitted', 'approved'))]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_split` | form |  | `total_amount_currency_original`, `expense_id`, `expense_split_line_ids`, `currency_id`, `expense_id`, `company_id`, `product_has_cost`, `name`, `product_id`, `employee_id`, `tax_ids`, `tax_amount_currency`, `analytic_distribution`, `total_amount_currency`, `currency_id`, `total_amount_currency`, `total_amount_currency`, `total_amount_currency_original`, `tax_amount_currency`, `split_possible` | `Split Expense`, `Split Expense`, `Cancel` |  | `hr_expense` |
| `sale_expense.hr_expense_split_view_inherit_sale_expense` | xpath | `hr_expense.hr_expense_split` | `can_be_reinvoiced`, `sale_order_id` |  |  | `sale_expense` |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.split.wizard.json`; views: `../../../schemas/interfaces/views/hr.expense.split.wizard.json`.
