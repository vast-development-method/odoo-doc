# Point of Sale Session (`pos.session`)

**Transport name:** `pos.session`  
**Storage name:** `pos_session`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `l10n_ar_pos`, `pos_restaurant`, `pos_sale`, `l10n_fr_pos_cert`, `l10n_in_pos`, `l10n_pe_pos`, `l10n_tw_edi_ecpay_pos`, `l10n_vn_edi_viettel_pos`, `pos_event`, `pos_hr`, `pos_loyalty`, `pos_mercado_pago`, `pos_online_payment`, `pos_self_order`

Description: Point of Sale Session

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `pos.bus.mixin`, `pos.load.mixin`
- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (32)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | read only; related through path `config_id.company_id` |
| `config_id` | Point of Sale | many to one | `pos.config` | required; indexed |
| `name` | Session identifier | single line text |  | read only; default `/` |
| `user_id` | Opened By | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.uid); indexed; on delete of the target: restrict |
| `currency_id` | Currency | many to one | `res.currency` | related through path `config_id.currency_id` |
| `start_at` | Opening Date | date and time |  | read only |
| `stop_at` | Closing Date | date and time |  | read only; not copied on duplication |
| `state` | Status | selection |  | required; read only; default `opening_control`; indexed; not copied on duplication |
| `opening_notes` | Opening Notes | multi line text |  |  |
| `closing_notes` | Closing Notes | multi line text |  |  |
| `cash_control` | Has Cash Control | boolean |  | computed by rule `_compute_cash_control` (not stored) |
| `cash_journal_id` | Cash Journal | many to one | `account.journal` | computed by rule `_compute_cash_journal` and stored |
| `cash_register_balance_end_real` | Ending Balance | monetary |  | read only |
| `cash_register_balance_start` | Starting Balance | monetary |  | read only |
| `cash_register_balance_end` | Theoretical Closing Balance | monetary |  | read only; computed by rule `_compute_cash_balance` (not stored); Help: Opening balance summed to all cash transactions. |
| `cash_register_difference` | Before Closing Difference | monetary |  | read only; computed by rule `_compute_cash_balance` (not stored); Help: Difference between the theoretical closing balance and the real closing balance. |
| `cash_real_transaction` | Transaction | monetary |  | read only |
| `order_ids` | Orders | one to many | `pos.order` | inverse field `session_id` |
| `order_count` | Order Count | integer |  | computed by rule `_compute_order_count` (not stored) |
| `statement_line_ids` | Cash Lines | one to many | `account.bank.statement.line` | read only; inverse field `pos_session_id` |
| `failed_pickings` | Failed Pickings | boolean |  | computed by rule `_compute_picking_count` (not stored) |
| `picking_count` | Picking Count | integer |  | computed by rule `_compute_picking_count` (not stored) |
| `picking_ids` | Picking | one to many | `stock.picking` | inverse field `pos_session_id` |
| `rescue` | Recovery Session | boolean |  | read only; not copied on duplication; Help: Auto-generated session for orphan orders, ignored in constraints |
| `move_id` | Journal Entry | many to one | `account.move` | indexed |
| `payment_method_ids` | Payment Methods | many to many | `pos.payment.method` | related through path `config_id.payment_method_ids` |
| `total_payments_amount` | Total Payments Amount | float |  | computed by rule `_compute_total_payments_amount` (not stored) |
| `is_in_company_currency` | Is Using Company Currency | boolean |  | computed by rule `_compute_is_in_company_currency` (not stored) |
| `update_stock_at_closing` | Stock should be updated at closing | boolean |  |  |
| `bank_payment_ids` | Bank Payments | one to many | `account.payment` | inverse field `pos_session_id`; Help: Account payments representing aggregated and bank split payments. |
| `crm_team_id` | Sales Team | many to one | `crm.team` | read only; related through path `config_id.crm_team_id` |
| `employee_id` | Cashier | many to one | `hr.employee` | changes are tracked in the message thread; Help: The employee who currently uses the cash register |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (118)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `write` | lifecycle override | self, vals | `point_of_sale` |  |  |
| `_load_pos_data_relations` | internal rule | self, model, fields | `point_of_sale`, `pos_event` | model |  |
| `_load_pos_data_models` | internal rule | self, config | `l10n_ar_pos`, `l10n_pe_pos`, `point_of_sale`, `pos_event`, `pos_hr`, `pos_loyalty`, `pos_restaurant`, `pos_sale`, `pos_self_order` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `load_data` | operation | self, models_to_load | `point_of_sale` |  |  |
| `load_data_params` | operation | self | `point_of_sale` |  |  |
| `filter_local_data` | operation | self, models_to_filter | `point_of_sale` |  |  |
| `delete_opening_control_session` | operation | self | `point_of_sale` |  |  |
| `_delete_session` | internal rule | self | `point_of_sale` |  |  |
| `get_pos_ui_product_pricelist_item_by_product` | operation | self, product_tmpl_ids, product_ids, config_id | `point_of_sale` |  |  |
| `_compute_is_in_company_currency` | computation | self | `point_of_sale` | depends: `currency_id`, `company_id.currency_id` |  |
| `_compute_cash_balance` | computation | self | `point_of_sale` | depends: `payment_method_ids`, `order_ids`, `cash_register_balance_start` |  |
| `_compute_total_payments_amount` | computation | self | `point_of_sale` | depends: `order_ids.payment_ids.amount` |  |
| `_compute_order_count` | computation | self | `point_of_sale` |  |  |
| `_compute_picking_count` | computation | self | `point_of_sale` | depends: `picking_ids`, `picking_ids.state` |  |
| `action_stock_picking` | user action | self | `point_of_sale` |  |  |
| `_compute_cash_control` | computation | self | `point_of_sale` | depends: `cash_journal_id` |  |
| `_compute_cash_journal` | computation | self | `point_of_sale` | depends: `config_id`, `payment_method_ids` |  |
| `_check_pos_config` | validation | self | `point_of_sale` | constrains: `config_id` |  |
| `_check_start_date` | validation | self | `point_of_sale` | constrains: `start_at` |  |
| `_check_invoices_are_posted` | validation | self | `point_of_sale` |  |  |
| `create` | lifecycle override | self, vals_list | `point_of_sale` | model_create_multi |  |
| `unlink` | lifecycle override | self | `point_of_sale` |  |  |
| `action_pos_session_open` | user action | self | `point_of_sale` |  |  |
| `get_session_orders` | operation | self | `point_of_sale` |  |  |
| `action_pos_session_closing_control` | user action | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |  |
| `action_pos_session_validate` | user action | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |  |
| `action_pos_session_close` | user action | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |  |
| `_validate_session` | internal rule | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |  |
| `_post_statement_difference` | internal rule | self, amount | `point_of_sale` |  |  |
| `_prepare_cash_diff_line_ids` | preparation rule | self, amount, counterpart_account, taxes, name, date | `point_of_sale` |  |  |
| `_close_session_action` | internal rule | self, amount_to_balance | `point_of_sale` |  |  |
| `close_session_from_ui` | operation | self, bank_payment_method_diff_pairs | `point_of_sale` |  | Calling this method will try to close the session.  param bank_payment_method_diff_pairs: list[(int, float)]     Pairs of payment_method_id and diff_amount which will be used to post     loss/profit when closing the session.  If successful, it returns {'successful': True} Otherwise, it returns {'successful': False, 'message': str, 'redirect': bool}. 'redirect' is a boolean used to know whether we redirect the user to the back end or not. When necessary, error (i.e. UserError, AccessError) is raised which should redirect the user to the back end. |
| `post_close_register_message` | operation | self | `point_of_sale`, `pos_hr` |  |  |
| `update_closing_control_state_session` | operation | self, notes | `point_of_sale` |  |  |
| `post_closing_cash_details` | operation | self, counted_cash | `point_of_sale` |  | Calling this method will try store the cash details during the session closing.  :param counted_cash: float, the total cash the user counted from its cash register  If successful, it returns `{'successful': True}`. Otherwise, it returns `{'successful': False, 'message': str, 'redirect': bool}` where `'redirect'` is a boolean used to know whether we redirect the user to the back end or not. When necessary, error (i.e. UserError, AccessError) is raised which should redirect the user to the back end. |
| `_create_diff_account_move_for_split_payment_method` | internal rule | self, payment_method, diff_amount | `point_of_sale` |  |  |
| `_get_diff_account_move_ref` | preparation rule | self, payment_method | `point_of_sale` |  |  |
| `_get_diff_vals` | preparation rule | self, payment_method_id, diff_amount, outstanding_account | `point_of_sale` |  |  |
| `_cannot_close_session` | internal rule | self, bank_payment_method_diffs | `point_of_sale` |  | Add check in this method if you want to return or raise an error when trying to either post cash details or close the session. Raising an error will always redirect the user to the back end. It should return {'successful': False, 'message': str, 'redirect': bool} if we can't close the session |
| `get_cash_in_out_list` | operation | self | `point_of_sale`, `pos_hr` |  |  |
| `get_closing_control_data` | operation | self | `point_of_sale`, `pos_hr` |  |  |
| `_create_picking_at_end_of_session` | internal rule | self | `point_of_sale` |  |  |
| `_create_balancing_line` | internal rule | self, data, balancing_account, amount_to_balance | `point_of_sale` |  |  |
| `_prepare_balancing_line_vals` | preparation rule | self, imbalance_amount, move, balancing_account | `point_of_sale` |  |  |
| `_get_balancing_account` | preparation rule | self | `point_of_sale` |  |  |
| `_create_account_move` | internal rule | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  | Create account.move and account.move.line records for this session.  Side-effects include:     - setting self.move_id to the created account.move record     - reconciling cash receivable lines, invoice receivable lines and stock output lines |
| `_accumulate_amounts` | internal rule | self, data | `point_of_sale`, `pos_online_payment` |  |  |
| `_create_non_reconciliable_move_lines` | internal rule | self, data | `point_of_sale` |  |  |
| `_create_bank_payment_moves` | internal rule | self, data | `point_of_sale`, `pos_online_payment` |  |  |
| `_create_pay_later_receivable_lines` | internal rule | self, data | `point_of_sale` |  |  |
| `_ensure_payment_outstanding_account` | internal rule | self, payment, payment_amount | `point_of_sale` |  |  |
| `_create_combine_account_payment` | internal rule | self, payment_method, amounts, diff_amount | `point_of_sale` |  |  |
| `_apply_diff_on_account_payment_move` | internal rule | self, account_payment, payment_method, diff_amount | `point_of_sale` |  |  |
| `_create_split_account_payment` | internal rule | self, payment, amounts | `point_of_sale` |  |  |
| `_get_split_account_payment_vals` | preparation rule | self, payment, amounts, accounting_partner, destination_account | `point_of_sale` |  |  |
| `_create_split_account_payments` | internal rule | self, payment_amounts_list | `point_of_sale` |  |  |
| `_create_cash_statement_lines_and_cash_move_lines` | internal rule | self, data | `point_of_sale` |  |  |
| `_create_invoice_receivable_lines` | internal rule | self, data | `point_of_sale` |  |  |
| `_create_stock_valuation_lines` | internal rule | self, data | `point_of_sale` |  |  |
| `_reconcile_account_move_lines` | internal rule | self, data | `point_of_sale`, `pos_online_payment` |  |  |
| `_get_rounding_difference_vals` | preparation rule | self, amount, amount_converted | `point_of_sale` |  |  |
| `_get_split_receivable_vals` | preparation rule | self, payment, amount, amount_converted | `point_of_sale` |  |  |
| `_get_combine_receivable_vals` | preparation rule | self, payment_method, amount, amount_converted | `point_of_sale` |  |  |
| `_get_invoice_receivable_vals` | preparation rule | self, amount, amount_converted | `point_of_sale` |  |  |
| `_get_sale_key` | preparation rule | self, base_line | `l10n_in_pos`, `point_of_sale` |  |  |
| `_get_sale_vals` | preparation rule | self, key, sale_vals | `l10n_in_pos`, `point_of_sale` |  |  |
| `_get_tax_vals` | preparation rule | self, key, amount, amount_converted, base_amount_converted | `point_of_sale` |  |  |
| `_get_stock_expense_vals` | preparation rule | self, exp_account, amount, amount_converted | `point_of_sale` |  |  |
| `_get_stock_valuation_vals` | preparation rule | self, stock_val_account, amount, amount_converted | `point_of_sale` |  |  |
| `_get_combine_statement_line_vals` | preparation rule | self, journal, amount, payment_method | `point_of_sale` |  |  |
| `_get_split_statement_line_vals` | preparation rule | self, journal, amount, payment | `point_of_sale` |  |  |
| `_prepare_statement_line_amount_values` | preparation rule | self, journal, amount | `point_of_sale` |  |  |
| `_update_amounts` | internal rule | self, old_amounts, amounts_to_add, date, round, force_company_currency | `point_of_sale` |  | Responsible for adding `amounts_to_add` to `old_amounts` considering the currency of the session.      old_amounts {                                                       new_amounts {         amount                         amounts_to_add {                     amount         amount_converted        +          amount               ->          amount_converted        [base_amount                       [base_amount]                    [base_amount         base_amount_converted]        }                                     base_amount_converted]     }                                                |
| `_round_amounts` | internal rule | self, amounts | `point_of_sale` |  |  |
| `_credit_amounts` | internal rule | self, partial_move_line_vals, amount, amount_converted, force_company_currency | `point_of_sale` |  | `partial_move_line_vals` is completed by `credit`ing the given amounts.  NOTE Amounts in PoS are in the currency of journal_id in the session.config_id. This means that amount fields in any pos record are actually equivalent to amount_currency in account module. Understanding this basic is important in correctly assigning values for 'amount' and 'amount_currency' in the account.move.line record.  :param dict partial_move_line_vals:     initial values in creating account.move.line :param float amount:     amount derived from pos.payment, pos.order, or pos.order.line records :param float amount_ |
| `_debit_amounts` | internal rule | self, partial_move_line_vals, amount, amount_converted, force_company_currency | `point_of_sale` |  | `partial_move_line_vals` is completed by `debit`ing the given amounts.  See _credit_amounts docs for more details. |
| `_amount_converter` | internal rule | self, amount, date, round | `point_of_sale` |  |  |
| `show_cash_register` | operation | self | `point_of_sale` |  |  |
| `show_journal_items` | operation | self | `point_of_sale` |  |  |
| `_get_other_related_moves` | preparation rule | self | `point_of_sale` |  |  |
| `_get_related_account_moves` | preparation rule | self | `point_of_sale` |  |  |
| `_get_receivable_account` | preparation rule | self, payment_method | `point_of_sale` |  | Returns the default pos receivable account if no receivable_account_id is set on the payment method. |
| `action_show_payments_list` | user action | self | `point_of_sale` |  |  |
| `_get_captured_payments_domain` | preparation rule | self | `point_of_sale` |  |  |
| `open_frontend_cb` | operation | self | `l10n_fr_pos_cert`, `point_of_sale` |  | Open the pos interface with config_id as an extra argument.  In vanilla PoS each user can only have one active session, therefore it was not needed to pass the config_id on opening a session. It is also possible to login to sessions created by other users.  :returns: dict |
| `_set_opening_control_data` | internal rule | self, cashbox_value, notes | `point_of_sale`, `pos_hr` |  | Internal logic for opening the session. Inherit this method to add custom logic before the sequence is assigned. |
| `set_opening_control` | operation | self, cashbox_value, notes | `point_of_sale` |  | Public method to open the session. This calls the internal logic and, if successful, assigns the sequence name.  DO NOT INHERIT THIS METHOD. Inherit _set_opening_control_data instead. |
| `_post_cash_details_message` | internal rule | self, state, expected, difference, notes | `point_of_sale` |  |  |
| `action_view_order` | user action | self | `point_of_sale` |  |  |
| `_alert_old_session` | internal rule | self | `point_of_sale` | model |  |
| `_check_if_no_draft_orders` | validation | self | `point_of_sale` |  |  |
| `_prepare_account_bank_statement_line_vals` | preparation rule | self, session, sign, amount, reason, partner_id, extras | `point_of_sale`, `pos_hr` |  |  |
| `try_cash_in_out` | operation | self, _type, amount, reason, partner_id, extras | `point_of_sale` |  |  |
| `delete_cash_in_out` | operation | self, absl_id, partner_id | `point_of_sale` |  |  |
| `_get_attributes_by_ptal_id` | preparation rule | self | `point_of_sale` |  |  |
| `_get_partners_domain` | preparation rule | self | `point_of_sale` |  |  |
| `find_product_by_barcode` | operation | self, barcode, config_id | `point_of_sale` |  |  |
| `get_total_discount` | operation | self | `point_of_sale` |  |  |
| `_get_invoice_total_list` | preparation rule | self | `point_of_sale` |  |  |
| `_get_total_invoice` | preparation rule | self | `point_of_sale` |  |  |
| `log_partner_message` | operation | self, partner_id, action, message_type | `point_of_sale` |  |  |
| `_pos_has_valid_product` | internal rule | self | `point_of_sale` |  |  |
| `_get_closed_orders` | preparation rule | self | `point_of_sale` |  |  |
| `_set_last_order_preparation_change` | internal rule | self, order_ids | `pos_restaurant` | model |  |
| `_check_session_timing` | validation | self | `l10n_fr_pos_cert` |  |  |
| `set_missing_hsn_codes_in_pos_orders` | operation | self | `l10n_in_pos` |  |  |
| `l10n_tw_edi_check_mobile_barcode` | operation | self, text | `l10n_tw_edi_ecpay_pos` |  |  |
| `l10n_tw_edi_check_love_code` | operation | self, text | `l10n_tw_edi_ecpay_pos` |  |  |
| `_load_pos_data_read` | internal rule | self, records, config | `l10n_vn_edi_viettel_pos`, `pos_self_order` | model |  |
| `_get_message_author` | preparation rule | self | `pos_hr` |  |  |
| `_aggregate_payments_amounts_by_employee` | internal rule | self, all_payments | `pos_hr` |  |  |
| `_aggregate_moves_by_employee` | internal rule | self | `pos_hr` |  |  |
| `_loader_params_pos_payment_method` | internal rule | self | `pos_mercado_pago` |  |  |
| `_get_split_receivable_op_vals` | preparation rule | self, payment, amount, amount_converted | `pos_online_payment` |  |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |

## Validation and error messages (24)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `delete_opening_control_session` | UserError | You can only cancel a session that is in opening control state and has no orders. | `point_of_sale` |
| `_check_pos_config` | ValidationError | Another session is already opened for this point of sale. | `point_of_sale` |
| `_check_start_date` | ValidationError | You cannot create a session starting before: %(lock_date_info)s | `point_of_sale` |
| `_check_invoices_are_posted` | UserError | You cannot close the POS when invoices are not posted. Invoices: %s | `point_of_sale` |
| `create` | UserError | You should assign a Point of Sale to your session. | `point_of_sale` |
| `action_pos_session_closing_control` | UserError | You cannot close the POS while there are still draft orders for the day. | `point_of_sale` |
| `action_pos_session_closing_control` | UserError | This session is already closed. | `point_of_sale` |
| `_validate_session` | UserError | This session is already closed. | `point_of_sale` |
| `_post_statement_difference` | UserError | Please go on the %s journal and define a Loss Account. This account will be used to record cash difference. | `point_of_sale` |
| `_post_statement_difference` | UserError | Please go on the %s journal and define a Profit Account. This account will be used to record cash difference. | `point_of_sale` |
| `update_closing_control_state_session` | UserError | This session is already closed. | `point_of_sale` |
| `post_closing_cash_details` | UserError | There is no cash register in this session. | `point_of_sale` |
| `get_cash_in_out_list` | AccessError | You don't have the access rights to get the cash in/out list. | `point_of_sale` |
| `get_closing_control_data` | AccessError | You don't have the access rights to get the point of sale closing control data. | `point_of_sale` |
| `_create_non_reconciliable_move_lines` | UserError | Unable to close and validate the session. Please set corresponding tax account in each repartition line of the following taxes:  %s | `point_of_sale` |
| `_get_split_receivable_vals` | UserError | You have enabled the "Identify Customer" option for %(payment_method)s payment method,but the order %(order)s does not contain a customer. | `point_of_sale` |
| `_check_if_no_draft_orders` | UserError | There are still orders in draft state in the session. Pay or cancel the following orders to validate the session: %s | `point_of_sale` |
| `try_cash_in_out` | AccessError | You don't have the access rights to perform a cash in/out. | `point_of_sale` |
| `try_cash_in_out` | UserError | There is no cash payment method for this PoS Session | `point_of_sale` |
| `delete_cash_in_out` | AccessError | You don't have the access rights to delete a cash in/out. | `point_of_sale` |
| `delete_cash_in_out` | AccessError | You cannot delete a cash move that is not linked to this session. | `point_of_sale` |
| `l10n_tw_edi_check_mobile_barcode` | UserError | Mobile barcode is invalid! | `l10n_tw_edi_ecpay_pos` |
| `l10n_tw_edi_check_love_code` | UserError | Love code is invalid! | `l10n_tw_edi_ecpay_pos` |
| `_get_split_receivable_op_vals` | UserError | The partner of the POS online payment (id=%d) could not be found | `pos_online_payment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | no | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Point Of Sale Session | global (all users) | `[('config_id.company_id', 'in', company_ids)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_session_form` | form |  | `state`, `failed_pickings`, `rescue`, `order_count`, `picking_count`, `picking_count`, `total_payments_amount`, `name`, `cash_control`, `user_id`, `currency_id`, `config_id`, `move_id`, `start_at`, `stop_at`, `cash_register_balance_start`, `cash_register_balance_end_real` | `Continue Selling`, `Close Session & Post Entries`, `action_view_order`, `action_stock_picking`, `action_show_payments_list`, `show_journal_items`, `show_cash_register` |  | `point_of_sale` |
| `point_of_sale.view_pos_session_tree` | list |  | `name`, `config_id`, `user_id`, `start_at`, `stop_at`, `cash_register_balance_start`, `cash_register_balance_end_real`, `cash_register_balance_end`, `state` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_session_kanban` | kanban |  | `config_id`, `state`, `name`, `start_at`, `user_id` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_session_search` | search |  | `name`, `config_id`, `user_id` |  | `My Sessions`, `In Progress`, `Opening Date`, `Opened By`, `Point of Sale`, `Status`, `Opening Date`, `Closing Date` | `point_of_sale` |
| `pos_sale.view_pos_session_search_inherit_pos_sale` | xpath | `point_of_sale.view_pos_session_search` | `crm_team_id` |  |  | `pos_sale` |
| `pos_self_order.pos_self_view_pos_session_form` | xpath | `point_of_sale.view_pos_session_form` |  |  |  | `pos_self_order` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_session` | Sessions | list,kanban,form |  |  |  | `point_of_sale` |
| `point_of_sale.action_pos_session_filtered` | Sessions | list,form |  | `{             'search_default_config_id': [active_id],             'default_config_id': active_id}` |  | `point_of_sale` |
| `pos_sale.pos_session_action_from_crm_team` | Open Sessions | list,form |  | `{'search_default_open_sessions': True, 'search_default_crm_team_id': active_id}` |  | `pos_sale` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `point_of_sale.sale_details_report` | Sales Details | qweb-pdf | `point_of_sale.report_saledetails` |  |  |

Machine-readable definition: `../../../schemas/data/entities/pos.session.json`; views: `../../../schemas/interfaces/views/pos.session.json`.
