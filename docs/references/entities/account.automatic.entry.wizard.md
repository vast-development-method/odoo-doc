# Create Automatic Entries (`account.automatic.entry.wizard`)

**Transport name:** `account.automatic.entry.wizard`  
**Storage name:** `account_automatic_entry_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `account_fleet`

Description: Create Automatic Entries

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `action` | Action | selection |  | required |
| `move_data` | Move Data | multi line text |  | computed by rule `_compute_move_data` (not stored) |
| `preview_move_data` | Preview Move Data | multi line text |  | computed by rule `_compute_preview_move_data` (not stored) |
| `move_line_ids` | Move Line | many to many | `account.move.line` |  |
| `date` | Date | date |  | required; default computed dynamically (lambda self: fields.Date.context_today(self)) |
| `company_id` | Company | many to one | `res.company` | required; read only |
| `company_currency_id` | Company Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `percentage` | Percentage | float |  | computed by rule `_compute_percentage` and stored; Help: Percentage of each line to execute the action on. |
| `total_amount` | Total Amount | monetary |  | computed by rule `_compute_total_amount` and stored; currency taken from `company_currency_id`; Help: Total amount impacted by the automatic entry. |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal_id` (not stored); writable through an inverse rule; restricted by domain `[('type', '=', 'general')]`; must belong to the same company; Help: Journal where to create the entry. |
| `account_type` | Account Type | selection |  | computed by rule `_compute_account_type` and stored |
| `expense_accrual_account` | Expense Accrual Account | many to one | `account.account` | computed by rule `_compute_expense_accrual_account` (not stored); writable through an inverse rule; restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'off_balance'))]`; must belong to the same company |
| `revenue_accrual_account` | Revenue Accrual Account | many to one | `account.account` | computed by rule `_compute_revenue_accrual_account` (not stored); writable through an inverse rule; restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable', 'off_balance'))]`; must belong to the same company |
| `lock_date_message` | Lock Date Message | single line text |  | computed by rule `_compute_lock_date_message` (not stored) |
| `destination_account_id` | To | many to one | `account.account` | must belong to the same company; Help: Account to transfer to. |
| `display_currency_helper` | Currency Conversion Helper | boolean |  | computed by rule `_compute_display_currency_helper` (not stored) |

## Selection values

### `action` (Action)

| Value | Label |
|---|---|
| `change_period` | Change Period |
| `change_account` | Change Account |

### `account_type` (Account Type)

| Value | Label |
|---|---|
| `income` | Revenue |
| `expense` | Expense |

## Operations (28)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_expense_accrual_account` | computation | self | `account` | depends: `company_id` |  |
| `_inverse_expense_accrual_account` | inverse computation | self | `account` |  |  |
| `_compute_revenue_accrual_account` | computation | self | `account` | depends: `company_id` |  |
| `_inverse_revenue_accrual_account` | inverse computation | self | `account` |  |  |
| `_compute_journal_id` | computation | self | `account` | depends: `company_id` |  |
| `_inverse_journal_id` | inverse computation | self | `account` |  |  |
| `_constraint_percentage` | validation | self | `account` | constrains: `percentage`, `action` |  |
| `_compute_total_amount` | computation | self | `account` | depends: `percentage`, `move_line_ids` |  |
| `_compute_percentage` | computation | self | `account` | depends: `total_amount`, `move_line_ids` |  |
| `_compute_account_type` | computation | self | `account` | depends: `move_line_ids` |  |
| `_compute_lock_date_message` | computation | self | `account` | depends: `action`, `move_line_ids` |  |
| `_compute_display_currency_helper` | computation | self | `account` | depends: `destination_account_id` |  |
| `_check_date` | validation | self | `account` | constrains: `date`, `move_line_ids` |  |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_get_cut_off_label_format` | preparation rule | self | `account` |  | Get the translated format string used in cut-off labels |
| `_get_move_dict_vals_change_account` | preparation rule | self | `account` |  |  |
| `_get_move_line_dict_vals_change_period` | preparation rule | self, aml, date | `account_fleet`, `account` |  |  |
| `_get_lock_safe_date` | preparation rule | self, date | `account` |  |  |
| `_get_move_dict_vals_change_period` | preparation rule | self | `account` |  |  |
| `_compute_move_data` | computation | self | `account` | depends: `move_line_ids`, `journal_id`, `revenue_accrual_account`, `expense_accrual_account`, `percentage`, `date`, `account_type`, `action`, `destination_account_id` |  |
| `_compute_preview_move_data` | computation | self | `account` | depends: `move_data` |  |
| `do_action` | user action | self | `account` |  |  |
| `_do_action_change_period` | internal rule | self, move_vals | `account` |  |  |
| `_do_action_change_account` | internal rule | self, move_vals | `account` |  |  |
| `_format_new_transfer_move_log` | internal rule | self, acc_transfer_per_move | `account` |  |  |
| `_format_transfer_source_log` | internal rule | self, balances_per_account, transfer_move | `account` |  |  |
| `_format_move_link` | internal rule | self, move | `account` |  |  |
| `_format_strings` | internal rule | self, string, move, amount, account_source_name | `account` |  |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constraint_percentage` | UserError | Percentage must be between 0 and 100 | `account` |
| `_check_date` | ValidationError | The date selected is protected by: %(lock_date_info)s. | `account` |
| `default_get` | UserError | This can only be used on journal items | `account` |
| `default_get` | UserError | Oops! You can only change the period or account for posted entries! Other ones aren't up for an adventure like that! | `account` |
| `default_get` | UserError | Oops! You can only change the period or account for items that are not yet reconciled! Other ones aren't up for an adventure like that! | `account` |
| `default_get` | UserError | You cannot use this wizard on journal entries belonging to different companies. | `account` |
| `default_get` | UserError | No possible action found with the selected lines. | `account` |
| `_compute_move_data` | UserError | All accounts on the lines must be of the same type. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_user` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.account_automatic_entry_wizard_form` | form |  | `account_type`, `company_id`, `move_line_ids`, `display_currency_helper`, `action`, `lock_date_message`, `date`, `expense_accrual_account`, `revenue_accrual_account`, `date`, `destination_account_id`, `percentage`, `total_amount`, `total_amount`, `journal_id`, `preview_move_data` | `Create Journal Entries`, `Cancel` |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.account_automatic_entry_wizard_action` | Transfer Journal Items | form |  |  | new | `account` |

Machine-readable definition: `../../../schemas/data/entities/account.automatic.entry.wizard.json`; views: `../../../schemas/interfaces/views/account.automatic.entry.wizard.json`.
