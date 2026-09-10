# Bank Statement Line (`account.bank.statement.line`)

**Transport name:** `account.bank.statement.line`  
**Storage name:** `account_bank_statement_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_payment`, `point_of_sale`, `pos_hr`

Description: Bank Statement Line

## Identity and behavior

- Delegation inheritance: embeds `account.move` through field `move_id`
- Default ordering: `internal_index desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (27)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_id` | Journal Entry | many to one | `account.move` | required; read only; indexed; on delete of the target: cascade; must belong to the same company |
| `journal_id` | Journal | many to one | `account.journal` | required; related through path `move_id.journal_id` and stored; precomputed before insertion |
| `company_id` | Company | many to one | `res.company` | required; related through path `move_id.company_id` and stored; precomputed before insertion |
| `statement_id` | Statement | many to one | `account.bank.statement` | indexed |
| `payment_ids` | Auto-generated Payments | many to many | `account.payment` | association table `account_payment_account_bank_statement_line_rel` |
| `sequence` | Sequence | integer |  | default `1` |
| `partner_id` | Partner | many to one | `res.partner` | on delete of the target: restrict; restricted by domain `['\|', ('parent_id','=', False), ('is_company','=',True)]`; must belong to the same company |
| `account_number` | Bank Account Number | single line text |  |  |
| `partner_name` | Partner Name | single line text |  | indexed (btree_not_null) |
| `transaction_type` | Transaction Type | single line text |  |  |
| `payment_ref` | Label | single line text |  | indexed (trigram) |
| `currency_id` | Journal Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored |
| `amount` | Amount | monetary |  |  |
| `running_balance` | Running Balance | monetary |  | computed by rule `_compute_running_balance` (not stored) |
| `foreign_currency_id` | Foreign Currency | many to one | `res.currency` | Help: The optional other currency if it is a multi-currency entry. |
| `amount_currency` | Amount in Currency | monetary |  | computed by rule `_compute_amount_currency` and stored; currency taken from `foreign_currency_id`; Help: The amount expressed in an optional other currency if it is a multi-currency entry. |
| `amount_residual` | Residual Amount | float |  | computed by rule `_compute_is_reconciled` and stored |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `internal_index` | Internal Reference | single line text |  | computed by rule `_compute_internal_index` and stored |
| `is_reconciled` | Is Reconciled | boolean |  | computed by rule `_compute_is_reconciled` and stored |
| `statement_complete` | Statement Complete | boolean |  | related through path `statement_id.is_complete` |
| `statement_valid` | Statement Valid | boolean |  | related through path `statement_id.is_valid` |
| `statement_balance_end_real` | Statement Balance End Real | monetary |  | related through path `statement_id.balance_end_real` |
| `statement_name` | Statement Name | single line text |  | related through path `statement_id.name` |
| `transaction_details` | Transaction Details | structured document |  | read only |
| `pos_session_id` | Session | many to one | `pos.session` | indexed (btree_not_null); not copied on duplication |
| `employee_id` | Employee | many to one | `hr.employee` | Help: The employee who made the cash move. |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unreconciled_idx` | Index | `(journal_id, company_id, internal_index) WHERE is_reconciled IS NOT TRUE` |  | `account` |
| `_orphan_idx` | Index | `(journal_id, company_id, internal_index) WHERE statement_id IS NULL` |  | `account` |
| `_main_idx` | Index | `(journal_id, company_id, internal_index)` |  | `account` |

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_amount_currency` | computation | self | `account` | depends: `foreign_currency_id`, `date`, `amount`, `company_id` |  |
| `_compute_currency_id` | computation | self | `account` | depends: `journal_id.currency_id` |  |
| `_compute_running_balance` | computation | self | `account` |  |  |
| `_compute_internal_index` | computation | self | `account` | depends: `date`, `sequence` | Internal index is a field that holds the combination of the date, compliment of sequence and id of each line. Using this prevents us having a compound index, and extensive where clauses. Without this finding lines before current line (which we need for calculating the running balance) would need a query like this:   date < current date OR (date = current date AND sequence > current date) or (   date = current date AND sequence = current sequence AND id < current id) which needs to be repeated all over the code. This would be simply "internal index < current internal index" using this field. Al |
| `_compute_is_reconciled` | computation | self | `account` | depends: `journal_id`, `currency_id`, `amount`, `foreign_currency_id`, `amount_currency`, `move_id.checked`, `move_id.line_ids.account_id`, `move_id.line_ids.amount_currency`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.currency_id`, `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | Compute the field indicating if the statement lines are already reconciled with something. This field is used for display purpose (e.g. display the 'cancel' button on the statement lines). Also computes the residual amount of the statement line. |
| `_check_amounts_currencies` | validation | self | `account` | constrains: `amount`, `amount_currency`, `currency_id`, `foreign_currency_id`, `journal_id` | Ensure the consistency the specified amounts and the currencies. |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `new` | operation | self, values, origin, ref | `account` | model |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account` |  |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `formatted_read_group` | operation | self, domain, groupby, aggregates, having, offset, limit, order | `account` | model |  |
| `action_undo_reconciliation` | user action | self | `account` |  | Undo the reconciliation made on the statement line and reset their journal items to their original states. |
| `_check_allow_unlink` | validation | self | `account` | ondelete |  |
| `_find_or_create_bank_account` | internal rule | self | `account` |  |  |
| `_get_default_amls_matching_domain` | preparation rule | self, allow_draft | `account` |  |  |
| `_get_default_journal` | preparation rule | self | `account` | model |  |
| `_get_default_statement` | preparation rule | self, journal_id, date | `account` | model |  |
| `_get_accounting_amounts_and_currencies` | preparation rule | self | `account` |  | Retrieve the transaction amount, journal amount and the company amount with their corresponding currencies from the journal entry linked to the statement line. All returned amounts will be positive for an inbound transaction, negative for an outbound one.  :return: (     transaction_amount, transaction_currency,     journal_amount, journal_currency,     company_amount, company_currency, ) |
| `_prepare_counterpart_amounts_using_st_line_rate` | preparation rule | self, currency, balance, amount_currency | `account` |  | Convert the amounts passed as parameters to the statement line currency using the rates provided by the bank. The computed amounts are the one that could be set on the statement line as a counterpart journal item to fully paid the provided amounts as parameters.  :param currency:        The currency in which is expressed 'amount_currency'. :param balance:         The amount expressed in company currency. Only needed when the currency passed as                         parameter is neither the statement line's foreign currency, neither the journal's                         currency. :param amoun |
| `_prepare_move_line_default_vals` | preparation rule | self, counterpart_account_id | `account` |  | Prepare the dictionary to create the default account.move.lines for the current account.bank.statement.line record. :return: A list of python dictionary to be passed to the account.move.line's 'create' method. |
| `_seek_for_lines` | internal rule | self | `account` |  | Helper used to dispatch the journal items between: - The lines using the liquidity account. - The lines using the transfer account. - The lines being not in one of the two previous categories. :return: (liquidity_lines, suspense_lines, other_lines) |
| `_synchronize_from_moves` | internal rule | self, changed_fields | `account` |  | Update the account.bank.statement.line regarding its related account.move. Also, check both models are still consistent. :param changed_fields: A set containing all modified fields on account.move. |
| `_synchronize_to_moves` | internal rule | self, changed_fields | `account` |  | Update the account.move regarding the modified account.bank.statement.line. :param changed_fields: A list containing all modified fields on account.bank.statement.line. |
| `_get_partial_amounts` | preparation rule | self, current_balance, move_line, open_amount_currency, open_balance | `account_payment` |  |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_amounts_currencies` | ValidationError | The foreign currency must be different than the journal one: %s | `account` |
| `_check_amounts_currencies` | ValidationError | You can't provide an amount in foreign currency without specifying a foreign currency. | `account` |
| `_check_amounts_currencies` | ValidationError | You can't provide a foreign currency without specifying an amount in 'Amount in Currency' field. | `account` |
| `action_undo_reconciliation` | ValidationError | Validated entries can only be changed by your accountant. | `account` |
| `_check_allow_unlink` | UserError | You can not delete a transaction from a valid statement. If you want to delete it, please remove the statement first. | `account` |
| `_prepare_move_line_default_vals` | UserError | You can't create a new statement line without a suspense account set on the %s journal. | `account` |
| `_synchronize_from_moves` | UserError | The journal entry %s reached an invalid state regarding its related statement line. To be consistent, the journal entry must always have exactly one journal item involving the bank/cash account. | `account` |
| `_synchronize_from_moves` | UserError | %(move)s reached an invalid state regarding its related statement line. To be consistent, the journal entry must always have exactly one suspense line. | `account` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_basic` | yes | yes | yes | yes | `account` |
| `group_pos_user` | yes | yes | yes | no | `point_of_sale` |
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account bank statement line company rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Point Of Sale Bank Statement Line POS User | `[(4, ref('group_pos_user'))]` | `[('pos_session_id', '!=', False)]` | True | True | True | True |
| Point Of Sale Bank Statement Line Accountant | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/account.bank.statement.line.json`.
