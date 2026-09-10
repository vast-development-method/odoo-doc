# Accrued Orders Wizard (`account.accrued.orders.wizard`)

**Transport name:** `account.accrued.orders.wizard`  
**Storage name:** `account_accrued_orders_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `sale_stock`

Description: Accrued Orders Wizard

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | default computed dynamically (_get_default_company) |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal_id` and stored; restricted by domain `[('type', '=', 'general')]`; must belong to the same company; precomputed before insertion |
| `date` | Date | date |  | required; default computed dynamically (_get_default_date) |
| `reversal_date` | Reversal Date | date |  | required; computed by rule `_compute_reversal_date` and stored; precomputed before insertion |
| `amount` | Amount | monetary |  | Help: Specify an arbitrary value that will be accrued on a         default account for the entire order, regardless of the products on the different lines. |
| `currency_id` | Company Currency | many to one |  | read only; related through path `company_id.currency_id` and stored; Help: Utility field to express amount currency |
| `account_id` | Accrual Account | many to one | `account.account` | required; restricted by domain `[('account_type', '=', 'liability_current')] if context.get('active_model') in ['purchase.order', 'purchase.order.line'] else [('account_type', '=', 'asset_current')]`; must belong to the same company |
| `preview_data` | Preview Data | multi line text |  | computed by rule `_compute_preview_data` (not stored) |
| `display_amount` | Display Amount | boolean |  | computed by rule `_compute_display_amount` (not stored) |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_company` | preparation rule | self | `account` |  |  |
| `_get_default_date` | preparation rule | self | `account` |  |  |
| `_compute_display_amount` | computation | self | `account` | depends: `date`, `amount` |  |
| `_compute_reversal_date` | computation | self | `account` | depends: `date` |  |
| `_compute_journal_id` | computation | self | `account` | depends: `company_id` |  |
| `_compute_preview_data` | computation | self | `account` | depends: `date`, `journal_id`, `account_id`, `amount` |  |
| `_get_computed_account` | preparation rule | self, order, product, is_purchase | `account` |  |  |
| `_compute_move_vals` | computation | self | `account` |  |  |
| `_get_accrual_message_body` | preparation rule | self, move, reverse_move | `account` |  |  |
| `create_entries` | operation | self | `account` |  |  |
| `_get_product_expense_and_stock_var_accounts` | preparation rule | self, product | `account`, `sale_stock` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_move_vals` | UserError | Entries can only be created for a single company at a time. | `account` |
| `_compute_move_vals` | UserError | Cannot create an accrual entry with orders in different currencies. | `account` |
| `create_entries` | UserError | Reversal date must be posterior to date. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_account_user` | yes | yes | yes | no | `account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_accrued_orders_wizard` | form |  | `company_id`, `journal_id`, `account_id`, `amount`, `display_amount`, `date`, `reversal_date`, `preview_data` | `Create Entry`, `Cancel` |  | `account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase.action_accrued_expense_entry` | Accrued Expense Entry | form |  |  | new | `purchase` |
| `sale.action_accrued_revenue_entry` | Accrued Revenue Entry | form |  |  | new | `sale` |
| `sale.action_accrued_revenue_entry_sale_order_line` | Accrued Revenue Entry | form |  |  | new | `sale` |

Machine-readable definition: `../../../schemas/data/entities/account.accrued.orders.wizard.json`; views: `../../../schemas/interfaces/views/account.accrued.orders.wizard.json`.
