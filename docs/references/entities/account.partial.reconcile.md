# Partial Reconcile (`account.partial.reconcile`)

**Transport name:** `account.partial.reconcile`  
**Storage name:** `account_partial_reconcile`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Partial Reconcile

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `debit_move_id` | Debit Move | many to one | `account.move.line` | required; indexed |
| `credit_move_id` | Credit Move | many to one | `account.move.line` | required; indexed |
| `full_reconcile_id` | Full Reconcile | many to one | `account.full.reconcile` | indexed (btree_not_null); not copied on duplication |
| `exchange_move_id` | Exchange Move | many to one | `account.move` | indexed (btree_not_null) |
| `draft_caba_move_vals` | Values that created the draft cash-basis entry | structured document |  |  |
| `company_currency_id` | Company Currency | many to one | `res.currency` | related through path `company_id.currency_id`; Help: Utility field to express amount currency |
| `debit_currency_id` | Currency of the debit journal item. | many to one | `res.currency` | related through path `debit_move_id.currency_id` and stored; precomputed before insertion |
| `credit_currency_id` | Currency of the credit journal item. | many to one | `res.currency` | related through path `credit_move_id.currency_id` and stored; precomputed before insertion |
| `amount` | Amount | monetary |  | currency taken from `company_currency_id`; Help: Always positive amount concerned by this matching expressed in the company currency. |
| `debit_amount_currency` | Debit Amount Currency | monetary |  | currency taken from `debit_currency_id`; Help: Always positive amount concerned by this matching expressed in the debit line foreign currency. |
| `credit_amount_currency` | Credit Amount Currency | monetary |  | currency taken from `credit_currency_id`; Help: Always positive amount concerned by this matching expressed in the credit line foreign currency. |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; precomputed before insertion |
| `max_date` | Max Date of Matched Lines | date |  | computed by rule `_compute_max_date` and stored; precomputed before insertion |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_required_computed_currencies` | validation | self | `account` | constrains: `debit_currency_id`, `credit_currency_id` |  |
| `_compute_max_date` | computation | self | `account` | depends: `debit_move_id.date`, `credit_move_id.date` |  |
| `_compute_company_id` | computation | self | `account` | depends: `debit_move_id`, `credit_move_id` |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `_get_to_update_payments` | preparation rule | self, from_state | `account` |  |  |
| `_update_matching_number` | internal rule | self, amls | `account` | model |  |
| `_collect_tax_cash_basis_values` | internal rule | self | `account` |  | Collect all information needed to create the tax cash basis journal entries on the current partials. :return:    A dictionary mapping each move_id to the result of 'account_move._collect_tax_cash_basis_values'.             Also, add the 'partials' keys being a list of dictionary, one for each partial to process:                 * partial:          The account.partial.reconcile record.                 * percentage:       The reconciled percentage represented by the partial.                 * payment_rate:     The applied rate of this partial.                 * settlement_date:  The date at whic |
| `_prepare_cash_basis_base_line_vals` | preparation rule | self, base_line, balance, amount_currency | `account` | model | Prepare the values to be used to create the cash basis journal items for the tax base line passed as parameter.  :param base_line:       An account.move.line being the base of some taxes. :param balance:         The balance to consider for this line. :param amount_currency: The balance in foreign currency to consider for this line. :return:                A python dictionary that could be passed to the create method of                         account.move.line. |
| `_prepare_cash_basis_counterpart_base_line_vals` | preparation rule | self, cb_base_line_vals | `account` | model | Prepare the move line used as a counterpart of the line created by _prepare_cash_basis_base_line_vals.  :param cb_base_line_vals:   The line returned by _prepare_cash_basis_base_line_vals. :return:                    A python dictionary that could be passed to the create method of                             account.move.line. |
| `_prepare_cash_basis_tax_line_vals` | preparation rule | self, tax_line, balance, amount_currency | `account` | model | Prepare the move line corresponding to a tax in the cash basis entry.  :param tax_line:        An account.move.line record being a tax line. :param balance:         The balance to consider for this line. :param amount_currency: The balance in foreign currency to consider for this line. :return:                A python dictionary that could be passed to the create method of                         account.move.line. |
| `_prepare_cash_basis_counterpart_tax_line_vals` | preparation rule | self, tax_line, cb_tax_line_vals | `account` | model | Prepare the move line used as a counterpart of the line created by _prepare_cash_basis_tax_line_vals.  :param tax_line:            An account.move.line record being a tax line. :param cb_tax_line_vals:    The result of _prepare_cash_basis_counterpart_tax_line_vals. :return:                    A python dictionary that could be passed to the create method of                             account.move.line. |
| `_get_cash_basis_base_line_grouping_key_from_vals` | preparation rule | self, base_line_vals | `account` | model | Get the grouping key of a cash basis base line that hasn't yet been created. :param base_line_vals:  The values to create a new account.move.line record. :return:                The grouping key as a tuple. |
| `_get_cash_basis_base_line_grouping_key_from_record` | preparation rule | self, base_line, account | `account` | model | Get the grouping key of a journal item being a base line. :param base_line:   An account.move.line record. :param account:     Optional account to shadow the current base_line one. :return:            The grouping key as a tuple. |
| `_get_cash_basis_tax_line_grouping_key_from_vals` | preparation rule | self, tax_line_vals | `account` | model | Get the grouping key of a cash basis tax line that hasn't yet been created. :param tax_line_vals:   The values to create a new account.move.line record. :return:                The grouping key as a tuple. |
| `_get_cash_basis_tax_line_grouping_key_from_record` | preparation rule | self, tax_line, account | `account` | model | Get the grouping key of a journal item being a tax line. :param tax_line:    An account.move.line record. :param account:     Optional account to shadow the current tax_line one. :return:            The grouping key as a tuple. |
| `_create_tax_cash_basis_moves` | internal rule | self | `account` |  | Create the tax cash basis journal entries. :return: The newly created journal entries. |
| `_get_draft_caba_move_vals` | preparation rule | self | `account` |  |  |
| `_set_draft_caba_move_vals` | internal rule | self | `account` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_required_computed_currencies` | ValidationError | Missing foreign currencies on partials having ids: %s | `account` |
| `_collect_tax_cash_basis_values` | UserError | There is no tax cash basis journal defined for the '%s' company. Configure it in Accounting/Configuration/Settings | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `account.group_account_user` | yes | yes | yes | yes | `account` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `sale_stock` |

Machine-readable definition: `../../../schemas/data/entities/account.partial.reconcile.json`.
