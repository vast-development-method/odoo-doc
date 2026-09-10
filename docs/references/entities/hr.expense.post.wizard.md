# Expense Posting Wizard (`hr.expense.post.wizard`)

**Transport name:** `hr.expense.post.wizard`  
**Storage name:** `hr_expense_post_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_expense`

Description: Expense Posting Wizard

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | read only; default computed dynamically (lambda self: self.env.company) |
| `accounting_date` | Accounting Date | date |  | default computed dynamically (fields.Date.context_today); Help: Specify the bill date of the related vendor bill. |
| `employee_journal_id` | Journal | many to one | `account.journal` | default computed dynamically (_default_journal_id); restricted by domain `[["type", "=", "purchase"]]`; must belong to the same company; Help: The journal used when the expense is paid by employee. |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_journal_id` | preparation rule | self | `hr_expense` | model | The journal is determining the company of the accounting entries generated from expense. We need to force journal company and expense company to be the same. |
| `action_post_entry` | user action | self | `hr_expense` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_post_entry` | UserError | You don't have the rights to create accounting entries. | `hr_expense` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `hr_expense` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_expense.hr_expense_post_wizard_view` | form |  | `employee_journal_id`, `accounting_date` | `Post Expenses`, `Cancel` |  | `hr_expense` |

Machine-readable definition: `../../../schemas/data/entities/hr.expense.post.wizard.json`; views: `../../../schemas/interfaces/views/hr.expense.post.wizard.json`.
