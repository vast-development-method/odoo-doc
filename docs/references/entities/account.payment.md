# Payments (`account.payment`)

**Transport name:** `account.payment`  
**Storage name:** `account_payment`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_check_printing`, `account_payment`, `hr_expense`, `l10n_account_withholding_tax`, `point_of_sale`, `website_payment`, `l10n_latam_check`, `l10n_ar_withholding`, `l10n_au`, `l10n_ch`, `l10n_in`, `l10n_nz`, `l10n_ph`, `l10n_pl_bank_verification`, `pos_online_payment`

Description: Payments

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.main.attachment`, `mail.activity.mixin`
- Default ordering: `date desc, name desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (80)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Number | single line text |  | computed by rule `_compute_name` and stored |
| `date` | Date | date |  | required; default computed dynamically (fields.Date.context_today); changes are tracked in the message thread |
| `move_id` | Journal Entry | many to one | `account.move` | indexed; not copied on duplication; must belong to the same company |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal_id` and stored; must belong to the same company; precomputed before insertion |
| `company_id` | Company | many to one | `res.company` | required; computed by rule `_compute_company_id` and stored; precomputed before insertion |
| `state` | State | selection |  | required; computed by rule `_compute_state` and stored; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `is_reconciled` | Is Reconciled | boolean |  | computed by rule `_compute_reconciliation_status` and stored |
| `is_matched` | Is Matched With a Bank Statement | boolean |  | computed by rule `_compute_reconciliation_status` and stored |
| `is_sent` | Is Sent | boolean |  | read only; not copied on duplication |
| `available_partner_bank_ids` | Available Partner Bank | many to many | `res.partner.bank` | computed by rule `_compute_available_partner_bank_ids` (not stored) |
| `partner_bank_id` | Recipient Bank Account | many to one | `res.partner.bank` | computed by rule `_compute_partner_bank_id` and stored; changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `[('id', 'in', available_partner_bank_ids)]`; must belong to the same company |
| `qr_code` | quick response Code uniform resource locator | rich text |  | computed by rule `_compute_qr_code` (not stored) |
| `paired_internal_transfer_payment_id` | Paired Internal Transfer Payment | many to one | `account.payment` | indexed (btree_not_null); not copied on duplication; Help: When an internal transfer is posted, a paired payment is created. They are cross referenced through this field |
| `payment_method_line_id` | Payment Method | many to one | `account.payment.method.line` | computed by rule `_compute_payment_method_line_id` and stored; indexed; not copied on duplication; restricted by domain `[('id', 'in', available_payment_method_line_ids)]`; Help: Manual: Pay or Get paid by any method outside of the system. Payment Providers: Each payment provider has its own Payment Method. Request a transaction on/to a card thanks to a payment token saved by the partner when buying or subscribing online. Check: Pay bills by check and print it from the system. Batch Deposit: Collect several customer checks at once generating and submitting a batch deposit to your bank. Module account_batch_payment is necessary. SEPA Credit Transfer: Pay in the SEPA zone by submitting a SEPA Credit Transfer file to your bank. Module account_iso20022 is necessary. SEPA Direct Debit: Get paid in the SEPA zone thanks to a mandate your partner will have granted to you. Module account_iso20022 is necessary. U.S. ISO20022: Pay in the US by submitting an ISO20022 file to your bank. Module account_iso20022 is necessary.; extended by packages `account_check_printing` |
| `available_payment_method_line_ids` | Available Payment Method Line | many to many | `account.payment.method.line` | computed by rule `_compute_payment_method_line_fields` (not stored) |
| `payment_method_id` | Method | many to one |  | related through path `payment_method_line_id.payment_method_id` and stored; changes are tracked in the message thread |
| `available_journal_ids` | Available Journal | many to many | `account.journal` | computed by rule `_compute_available_journal_ids` (not stored) |
| `amount` | Amount | monetary |  | computed by rule `_compute_amount` and stored; currency taken from `currency_id`; extended by packages `l10n_latam_check` |
| `payment_type` | Payment Type | selection |  | required; default `inbound`; changes are tracked in the message thread |
| `partner_type` | Partner Type | selection |  | required; default `customer`; changes are tracked in the message thread |
| `memo` | Memo | single line text |  | writable through an inverse rule; changes are tracked in the message thread |
| `payment_reference` | Payment Reference | single line text |  | changes are tracked in the message thread; not copied on duplication; Help: Reference of the document used to issue this payment. Eg. check number, file name, etc. |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored; precomputed before insertion; Help: The payment's currency. |
| `company_currency_id` | Company Currency | many to one |  | related through path `company_id.currency_id` |
| `partner_id` | Customer/Vendor | many to one | `res.partner` | changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `['\|', ('parent_id','=', False), ('is_company','=', True)]`; must belong to the same company |
| `outstanding_account_id` | Outstanding Account | many to one | `account.account` | computed by rule `_compute_outstanding_account_id` and stored; indexed (btree_not_null); must belong to the same company; extended by packages `l10n_account_withholding_tax` |
| `destination_account_id` | Destination Account | many to one | `account.account` | computed by rule `_compute_destination_account_id` and stored; indexed (btree_not_null); restricted by domain `[('account_type', 'in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company |
| `invoice_ids` | Invoices | many to many | `account.move` | not copied on duplication; association table `account_move__account_payment` |
| `reconciled_invoice_ids` | Reconciled Invoices | many to many | `account.move` | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored); searchable through a search rule; Help: Invoices whose journal items have been reconciled with these payments. |
| `reconciled_invoices_count` | # Reconciled Invoices | integer |  | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored) |
| `reconciled_invoices_type` | Reconciled Invoices Type | selection |  | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored) |
| `reconciled_bill_ids` | Reconciled Bills | many to many | `account.move` | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored); searchable through a search rule; Help: Invoices whose journal items have been reconciled with these payments. |
| `reconciled_bills_count` | # Reconciled Bills | integer |  | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored) |
| `reconciled_statement_line_ids` | Reconciled Statement Lines | many to many | `account.bank.statement.line` | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored); Help: Statements lines matched to this payment |
| `reconciled_statement_lines_count` | # Reconciled Statement Lines | integer |  | computed by rule `_compute_stat_buttons_from_reconciliation` (not stored) |
| `payment_method_code` | Payment Method Code | single line text |  | related through path `payment_method_line_id.code` |
| `payment_receipt_title` | Payment Receipt Title | single line text |  | computed by rule `_compute_payment_receipt_title` (not stored) |
| `need_cancel_request` | Need Cancel Request | boolean |  | related through path `move_id.need_cancel_request` |
| `show_partner_bank_account` | Show Partner Bank Account | boolean |  | computed by rule `_compute_show_require_partner_bank` (not stored) |
| `require_partner_bank_account` | Require Partner Bank Account | boolean |  | computed by rule `_compute_show_require_partner_bank` (not stored) |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `amount_signed` | Amount Signed | monetary |  | computed by rule `_compute_amount_signed` (not stored); changes are tracked in the message thread; currency taken from `currency_id`; Help: Negative value of amount field if payment_type is outbound |
| `amount_company_currency_signed` | Amount Company Currency Signed | monetary |  | computed by rule `_compute_amount_company_currency_signed` and stored; currency taken from `company_currency_id` |
| `duplicate_payment_ids` | Duplicate Payment | many to many | `account.payment` | computed by rule `_compute_duplicate_payment_ids` (not stored) |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | inverse field `res_id` |
| `check_amount_in_words` | Amount in Words | single line text |  | computed by rule `_compute_check_amount_in_words` and stored |
| `check_manual_sequencing` | Check Manual Sequencing | boolean |  | related through path `journal_id.check_manual_sequencing` |
| `check_number` | Check Number | single line text |  | computed by rule `_compute_check_number` and stored; writable through an inverse rule; not copied on duplication; Help: The selected journal is configured to print check numbers. If your pre-printed check paper already has numbers or if the current numbering is wrong, you can change it in the journal configuration page. |
| `show_check_number` | Show Check Number | boolean |  | computed by rule `_compute_show_check_number` (not stored) |
| `check_layout_available` | Has Check Layout | boolean |  | default computed dynamically (lambda self: len(self.env['res.company']._fields['account_check_printing_layout'].selection) > 1) |
| `payment_transaction_id` | Payment Transaction | many to one | `payment.transaction` | read only |
| `payment_token_id` | Saved Payment Token | many to one | `payment.token` | restricted by domain `[             ('id', 'in', suitable_payment_token_ids),         ]`; Help: Note that only tokens from providers allowing to capture the amount are available. |
| `amount_available_for_refund` | Amount Available For Refund | monetary |  | computed by rule `_compute_amount_available_for_refund` (not stored) |
| `suitable_payment_token_ids` | Suitable Payment Token | many to many | `payment.token` | computed by rule `_compute_suitable_payment_token_ids` (not stored) |
| `use_electronic_payment_method` | Use Electronic Payment Method | boolean |  | computed by rule `_compute_use_electronic_payment_method` (not stored) |
| `source_payment_id` | Source Payment | many to one | `account.payment` | read only; related through path `payment_transaction_id.source_transaction_id.payment_id` and stored; indexed (btree_not_null); Help: The source payment of related refund payments |
| `refunds_count` | Refunds Count | integer |  | computed by rule `_compute_refunds_count` (not stored) |
| `expense_ids` | Expense | one to many |  | related through path `move_id.expense_ids` |
| `display_withholding` | Display Withholding | boolean |  | computed by rule `_compute_display_withholding` (not stored) |
| `should_withhold_tax` | Withhold Tax Amounts | boolean |  | computed by rule `_compute_should_withhold_tax` and stored; not copied on duplication; Help: Withhold tax amounts from the payment amount. |
| `withholding_line_ids` | Withholding Lines | one to many | `account.payment.withholding.line` | inverse field `payment_id` |
| `withholding_payment_account_id` | Withholding Payment Account | many to one |  | related through path `payment_method_line_id.payment_account_id` |
| `withholding_hide_tax_base_account` | Withholding Hide Tax Base Account | boolean |  | computed by rule `_compute_withholding_hide_tax_base_account` (not stored) |
| `pos_payment_method_id` | point of sale Payment Method | many to one | `pos.payment.method` |  |
| `force_outstanding_account_id` | Forced Outstanding Account | many to one | `account.account` | indexed (btree_not_null); must belong to the same company |
| `pos_session_id` | point of sale Session | many to one | `pos.session` | indexed (btree_not_null) |
| `is_donation` | Is Donation | boolean |  | related through path `payment_transaction_id.is_donation` |
| `l10n_latam_new_check_ids` | Checks | one to many | `l10n_latam.check` | inverse field `payment_id` |
| `l10n_latam_move_check_ids` | Checks Operations | many to many | `l10n_latam.check` | required; not copied on duplication; association table `l10n_latam_check_account_payment_rel` |
| `l10n_latam_check_warning_msg` | Localization Latam Check Warning Msg | multi line text |  | computed by rule `_compute_l10n_latam_check_warning_msg` (not stored) |
| `l10n_ar_withholding_ids` | Localization Ar Withholding | one to many |  | related through path `move_id.l10n_ar_withholding_ids` |
| `l10n_ch_reference_warning_msg` | Localization Ch Reference Warning Msg | single line text |  | computed by rule `_compute_l10n_ch_reference_warning_msg` (not stored) |
| `l10n_in_withhold_move_ids` | Indian Payment tax deducted at source Entries | one to many | `account.move` | inverse field `l10n_in_withholding_ref_payment_id` |
| `l10n_in_total_withholding_amount` | Localization In Total Withholding Amount | monetary |  | computed by rule `_compute_l10n_in_total_withholding_amount` (not stored) |
| `l10n_in_tds_feature_enabled` | Localization In Tax deducted at source Feature Enabled | boolean |  | related through path `company_id.l10n_in_tds_feature` |
| `l10n_pl_verification_id` | PL Bank Verification | many to one | `l10n_pl.bank.account.verification` | read only; computed by rule `_compute_l10n_pl_verification_id` and stored; not copied on duplication |
| `l10n_pl_verification_status` | Localization Pl Verification Status | selection |  | related through path `l10n_pl_verification_id.verification_status` |
| `l10n_pl_verification_timestamp` | Localization Pl Verification Timestamp | date and time |  | related through path `l10n_pl_verification_id.verification_timestamp` |
| `l10n_pl_verification_request_id` | Localization Pl Verification Request | single line text |  | related through path `l10n_pl_verification_id.verification_request_id` |
| `pos_order_id` | point of sale Order | many to one | `pos.order` | read only; Help: The Point of Sale order linked to this payment |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_process` | In Process |
| `paid` | Paid |
| `canceled` | Canceled |
| `rejected` | Rejected |

### `payment_type` (Payment Type)

| Value | Label |
|---|---|
| `outbound` | Send |
| `inbound` | Receive |

### `partner_type` (Partner Type)

| Value | Label |
|---|---|
| `customer` | Customer |
| `supplier` | Vendor |

### `reconciled_invoices_type` (Reconciled Invoices Type)

| Value | Label |
|---|---|
| `credit_note` | Credit Note |
| `invoice` | Invoice |

## State fields

State machine fields of this entity: `state`, `l10n_pl_verification_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_amount_not_negative` | Constraint | `CHECK(amount >= 0.0)` | The payment amount cannot be negative. | `account` |
| `_journal_id_company_id_idx` | Index | `(journal_id, company_id)` |  | `account` |
| `_unmatched_idx` | Index | `(journal_id, company_id) WHERE is_matched IS NOT TRUE` |  | `account` |

## Operations (111)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_valid_payment_account_types` | preparation rule | self | `account` | model |  |
| `_seek_for_lines` | internal rule | self | `account` |  | Helper used to dispatch the journal items between: - The lines using the temporary liquidity account. - The lines using the counterpart account. - The lines being the write-off lines. :return: (liquidity_lines, counterpart_lines, writeoff_lines) |
| `_get_valid_liquidity_accounts` | preparation rule | self | `account` |  |  |
| `_valid_payment_states` | internal rule | self | `account` |  | This method is used to know in which edition we are: Community or Enterprise and fetch the payment states accordingly. |
| `_get_aml_default_display_name_list` | preparation rule | self | `account_check_printing`, `account` |  | Hook allowing custom values when constructing the default label to set on the journal items.  :return: A list of terms to concatenate all together. E.g.     [         ('label', "Greg's Card"),         ('sep', ": "),         ('memo', "New Computer"),     ] |
| `_prepare_move_withholding_lines` | preparation rule | self, default_values | `account`, `l10n_account_withholding_tax` |  |  |
| `_prepare_move_liquidity_lines` | preparation rule | self, default_values | `account` |  |  |
| `_prepare_move_counterpart_lines` | preparation rule | self, default_values | `account` |  |  |
| `_prepare_move_lines_per_type` | preparation rule | self, write_off_line_vals, force_balance | `account` |  | Prepare the dictionary containing default vals for account.move.lines for the current payment. returns a dictionary of list of python dictionary containing liquidity, counterpart and writeoff lines.     E.g.     {         'liquidity_lines': [...],         'counterpart_lines': [...],         'writeoff_lines': [...],     } |
| `_prepare_move_line_default_vals` | preparation rule | self, write_off_line_vals, force_balance | `account`, `l10n_latam_check` |  | Prepare the dictionary to create the default account.move.lines for the current payment. :param write_off_line_vals: Optional list of dictionaries to create a write-off account.move.line easily containing:     * amount:       The amount to be added to the counterpart amount.     * name:         The label to set on the line.     * account_id:   The account on which create the write-off. :param force_balance: Optional balance. :return: A list of python dictionary to be passed to the account.move.line's 'create' method. |
| `_compute_name` | computation | self | `account` | depends: `move_id.name`, `state` |  |
| `_compute_journal_id` | computation | self | `account` | depends: `company_id`, `partner_id` |  |
| `_compute_company_id` | computation | self | `account` | depends: `journal_id` |  |
| `_compute_state` | computation | self | `account` | depends: `reconciled_invoice_ids.payment_state`, `reconciled_bill_ids.payment_state`, `move_id.line_ids.amount_residual` |  |
| `_compute_reconciliation_status` | computation | self | `account` | depends: `move_id.line_ids.amount_residual`, `move_id.line_ids.amount_residual_currency`, `move_id.line_ids.account_id`, `state` | Compute the field indicating if the payments are already reconciled with something. This field is used for display purpose (e.g. display the 'reconcile' button redirecting to the reconciliation widget). |
| `_get_method_codes_using_bank_account` | preparation rule | self | `account` | model |  |
| `_get_method_codes_needing_bank_account` | preparation rule | self | `account` | model |  |
| `action_open_business_doc` | user action | self | `account` |  |  |
| `_compute_show_require_partner_bank` | computation | self | `account`, `hr_expense` | depends: `payment_method_code` | Computes if the destination bank account must be displayed in the payment form view. By default, it won't be displayed but some modules might change that, depending on the payment type. |
| `_compute_amount_company_currency_signed` | computation | self | `account` | depends: `move_id.amount_total_signed`, `amount`, `payment_type`, `currency_id`, `date`, `company_id`, `company_currency_id` |  |
| `_compute_amount_signed` | computation | self | `account` | depends: `amount`, `payment_type` |  |
| `_compute_available_partner_bank_ids` | computation | self | `account` | depends: `partner_id`, `company_id`, `payment_type` |  |
| `_compute_partner_bank_id` | computation | self | `account` | depends: `available_partner_bank_ids`, `journal_id` | The default partner_bank_id will be the first available on the partner. |
| `_compute_payment_method_line_id` | computation | self | `account` | depends: `available_payment_method_line_ids` | Compute the 'payment_method_line_id' field. This field is not computed in '_compute_payment_method_line_fields' because it's a stored editable one. |
| `_compute_payment_method_line_fields` | computation | self | `account` | depends: `payment_type`, `journal_id`, `currency_id` |  |
| `_compute_available_journal_ids` | computation | self | `account` | depends: `payment_type` | Get all journals having at least one payment method for inbound/outbound depending on the payment_type. |
| `_get_payment_method_codes_to_exclude` | preparation rule | self | `account`, `point_of_sale` |  |  |
| `_compute_currency_id` | computation | self | `account` | depends: `journal_id` |  |
| `_compute_outstanding_account_id` | computation | self | `account`, `hr_expense`, `l10n_account_withholding_tax`, `point_of_sale` | depends: `payment_method_line_id`; depends: `should_withhold_tax`; depends: `force_outstanding_account_id` | Update the computation to reset the account when should_withhold_tax is unchecked. |
| `_compute_destination_account_id` | computation | self | `account`, `l10n_latam_check` | depends: `journal_id`, `partner_id`, `partner_type`; depends: `l10n_latam_move_check_ids` |  |
| `_compute_qr_code` | computation | self | `account` | depends: `partner_bank_id`, `amount`, `memo`, `currency_id`, `journal_id`, `move_id.state`, `payment_method_line_id`, `payment_type` |  |
| `_compute_stat_buttons_from_reconciliation` | computation | self | `account` | depends: `move_id.line_ids.matched_debit_ids`, `move_id.line_ids.matched_credit_ids` | Retrieve the invoices reconciled to the payments through the reconciliation (account.partial.reconcile). |
| `_compute_payment_receipt_title` | computation | self | `account`, `l10n_au`, `l10n_nz` | depends: `country_code`, `partner_type` | To override in order to change the title displayed on the payment receipt report |
| `_compute_duplicate_payment_ids` | computation | self | `account` | depends: `partner_id`, `amount`, `date`, `payment_type` | Retrieve move ids with same partner_id, amount and date as the current payment |
| `_search_reconciled_invoice_ids` | search rule | self, operator, value | `account` |  |  |
| `_fetch_duplicate_reference` | internal rule | self, matching_states | `account` |  | Retrieve move ids for possible duplicates of payments. Duplicates moves: - Have the same partner_id, amount and date as the payment - Are not reconciled - Represent a credit in the same account receivable or a debit in the same account payable as the payment, or - Represent a credit in outstanding receipts or debit in outstanding payments, so bank statement lines with an  outstanding counterpart can be matched, or - Are in the suspense account |
| `_inverse_memo` | inverse computation | self | `account` |  |  |
| `_check_payment_method_line_id` | validation | self | `account` | constrains: `payment_method_line_id` | Ensure the 'payment_method_line_id' field is not null. Can't be done using the regular 'required=True' because the field is a computed editable stored one. |
| `_check_move_id` | validation | self | `account`, `l10n_latam_check` | constrains: `state`, `move_id` |  |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `_get_outstanding_account` | preparation rule | self, payment_type | `account` |  |  |
| `write` | lifecycle override | self, vals | `account`, `hr_expense` |  |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `_compute_display_name` | computation | self | `account` | depends: `move_id.name` |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |
| `_message_mail_after_hook` | messaging hook | self, mails | `account` |  |  |
| `_synchronize_to_moves` | internal rule | self, changed_fields | `account`, `l10n_ar_withholding` |  | Update the account.move regarding the modified account.payment. :param changed_fields: A list containing all modified fields on account.payment. |
| `_get_trigger_fields_to_synchronize` | preparation rule | self | `account_check_printing`, `account`, `l10n_account_withholding_tax`, `l10n_latam_check` | model |  |
| `_generate_journal_entry` | internal rule | self, write_off_line_vals, force_balance, line_ids | `account` |  |  |
| `_generate_move_vals` | internal rule | self, write_off_line_vals, force_balance, line_ids | `account` |  | Prepare the values needed to create a move for self. |
| `_get_payment_receipt_report_values` | preparation rule | self | `account` |  | Get the extra values when rendering the Payment Receipt PDF report.  :return: A dictionary:     * display_invoices: Display the invoices table.     * display_payment_method: Display the payment method value. |
| `mark_as_sent` | operation | self | `account` |  |  |
| `unmark_as_sent` | operation | self | `account` |  |  |
| `action_post` | user action | self | `account_check_printing`, `account_payment`, `account`, `l10n_latam_check` |  | draft -> posted |
| `action_validate` | user action | self | `account` |  |  |
| `action_reject` | user action | self | `account` |  |  |
| `action_cancel` | user action | self | `account`, `l10n_latam_check` |  |  |
| `button_request_cancel` | user action | self | `account` |  |  |
| `action_draft` | user action | self | `account`, `l10n_latam_check` |  |  |
| `button_open_invoices` | user action | self | `account` |  | Redirect the user to the invoice(s) paid by this payment. :return:    An action on account.move. |
| `button_open_bills` | user action | self | `account` |  | Redirect the user to the bill(s) paid by this payment. :return:    An action on account.move. |
| `button_open_statement_lines` | user action | self | `account` |  | Redirect the user to the statement line(s) reconciled to this payment. :return:    An action on account.move. |
| `button_open_journal_entry` | user action | self | `account` |  | Redirect the user to this payment journal. :return:    An action on account.move. |
| `_compute_show_check_number` | computation | self | `account_check_printing` | depends: `payment_method_line_id.code`, `check_number` |  |
| `_constrains_check_number` | validation | self | `account_check_printing` | constrains: `check_number` |  |
| `_auto_init` | lifecycle override | self | `account_check_printing`, `l10n_pl_bank_verification` |  | Create compute stored field check_number here to avoid MemoryError on large databases. |
| `_constrains_check_number_unique` | validation | self | `account_check_printing` | constrains: `check_number`, `journal_id` |  |
| `_compute_check_amount_in_words` | computation | self | `account_check_printing` | depends: `payment_method_line_id`, `currency_id`, `amount` |  |
| `_compute_check_number` | computation | self | `account_check_printing` | depends: `journal_id`, `payment_method_code` |  |
| `_inverse_check_number` | inverse computation | self | `account_check_printing` |  |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `account_check_printing` | model |  |
| `print_checks` | operation | self | `account_check_printing` |  | Check that the recordset is valid, set the payments state to sent and call print_checks() |
| `action_void_check` | user action | self | `account_check_printing` |  |  |
| `do_print_checks` | user action | self | `account_check_printing` |  |  |
| `_check_fill_line` | validation | self, amount_str | `account_check_printing` |  |  |
| `_check_build_page_info` | validation | self, i, p | `account_check_printing` |  |  |
| `_check_get_pages` | validation | self | `account_check_printing` |  | Returns the data structure used by the template: a list of dicts containing what to print on pages. |
| `_check_make_stub_pages` | validation | self | `account_check_printing` |  | The stub is the summary of paid invoices. It may spill on several pages, in which case only the check on first page is valid. This function returns a list of stub lines per page. |
| `_compute_amount_available_for_refund` | computation | self | `account_payment` |  |  |
| `_compute_suitable_payment_token_ids` | computation | self | `account_payment` | depends: `payment_method_line_id` |  |
| `_compute_use_electronic_payment_method` | computation | self | `account_payment` | depends: `payment_method_line_id` |  |
| `_compute_refunds_count` | computation | self | `account_payment` |  |  |
| `_onchange_set_payment_token_id` | on change | self | `account_payment` | onchange: `partner_id`, `payment_method_line_id`, `journal_id` |  |
| `action_refund_wizard` | user action | self | `account_payment` |  |  |
| `action_view_refunds` | user action | self | `account_payment` |  |  |
| `_create_payment_transaction` | internal rule | self, **extra_create_values | `account_payment` |  |  |
| `_prepare_payment_transaction_vals` | preparation rule | self, **extra_create_values | `account_payment` |  |  |
| `_get_payment_refund_wizard_values` | preparation rule | self | `account_payment` |  |  |
| `action_open_expense` | user action | self | `hr_expense` |  |  |
| `_creation_message` | internal rule | self | `hr_expense` |  |  |
| `_compute_display_withholding` | computation | self | `l10n_account_withholding_tax` | depends: `company_id` | The withholding feature should not show on companies which does not contain any withholding taxes. |
| `_compute_should_withhold_tax` | computation | self | `l10n_account_withholding_tax` | depends: `withholding_line_ids` | Ensures that we display the line table if any withholding line has been added to the payment. |
| `_compute_withholding_hide_tax_base_account` | computation | self | `l10n_account_withholding_tax` | depends: `company_id` | When the withholding tax base account is set in the setting, simplify the view by hiding the account column on the lines as we will default to that tax base account. |
| `_onchange_withholding_line_ids` | on change | self | `l10n_account_withholding_tax` | onchange: `withholding_line_ids` | Any time a line is edited, we want to check if we need to recompute the placeholders. The idea is to try and display accurate placeholders on lines whose tax have a sequence set. |
| `_compute_amount` | computation | self | `l10n_latam_check` | depends: `l10n_latam_move_check_ids.amount`, `l10n_latam_new_check_ids.amount`, `payment_method_code` |  |
| `_is_latam_check_payment` | internal rule | self, check_subtype | `l10n_latam_check` |  |  |
| `_get_latam_checks` | preparation rule | self | `l10n_latam_check` |  |  |
| `_get_blocking_l10n_latam_warning_msg` | preparation rule | self | `l10n_latam_check` |  |  |
| `_get_reconciled_checks_error` | preparation rule | self | `l10n_latam_check` |  |  |
| `_l10n_latam_check_split_move` | internal rule | self | `l10n_latam_check` |  |  |
| `_l10n_latam_check_unlink_split_move` | internal rule | self | `l10n_latam_check` |  |  |
| `_compute_l10n_latam_check_warning_msg` | computation | self | `l10n_latam_check` | depends: `payment_method_line_id`, `state`, `date`, `amount`, `currency_id`, `company_id`, `l10n_latam_move_check_ids.issuer_vat`, `l10n_latam_move_check_ids.bank_id`, `l10n_latam_move_check_ids.payment_id.date`, `l10n_latam_new_check_ids.amount`, `l10n_latam_new_check_ids.name` | Compute warning message for latam checks checks We use l10n_latam_check_number as de dependency because on the interface this is the field the user is using. Another approach could be to add an onchange on _inverse_l10n_latam_check_number method |
| `_is_latam_check_transfer` | internal rule | self | `l10n_latam_check` |  |  |
| `_compute_l10n_ch_reference_warning_msg` | on change | self | `l10n_ch` | onchange: `partner_id`, `memo`, `payment_type` |  |
| `_l10n_ch_reference_is_valid` | internal rule | self, payment_reference | `l10n_ch` |  | Check if this invoice has a valid reference (for Switzerland) e.g. 000000000000000000000012371 210000000003139471430009017 21 00000 00003 13947 14300 09017 |
| `_compute_l10n_in_total_withholding_amount` | computation | self | `l10n_in` |  |  |
| `action_l10n_in_withholding_entries` | user action | self | `l10n_in` |  |  |
| `action_open_l10n_ph_2307_wizard` | user action | self | `l10n_ph` |  |  |
| `_payment_need_check` | internal rule | self, partner, payment_type, amounts, currency | `l10n_pl_bank_verification` | model | :param amounts: list of amounts in case values are coming from a batch |
| `_compute_l10n_pl_verification_id` | computation | self | `l10n_pl_bank_verification` | depends: `state`, `date`, `partner_id`, `partner_bank_id` |  |
| `action_view_pos_order` | user action | self | `pos_online_payment` |  | Return the action for the view of the pos order linked to the payment. |

## Validation and error messages (20)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_prepare_move_lines_per_type` | UserError | You can't create a new payment without an outstanding payments/receipts account set either on the company or the %(payment_method)s payment method in the %(journal)s journal. | `account` |
| `_check_payment_method_line_id` | ValidationError | Please define a payment method line on your payment. | `account` |
| `_check_payment_method_line_id` | ValidationError | The selected payment method is not available for this payment, please select the payment method again. | `account` |
| `_check_move_id` | ValidationError | A payment with an outstanding account cannot be confirmed without having a journal entry. | `account` |
| `_get_outstanding_account` | UserError | No outstanding account could be found to make the payment | `account` |
| `_synchronize_to_moves` | UserError | You cannot change the amount of a payment with multiple liquidity lines. | `account` |
| `action_post` | UserError | To record payments with %(method_name)s, the recipient bank account must be manually validated. You should go on the partner bank account of %(partner)s in order to validate it. | `account` |
| `_constrains_check_number` | ValidationError | Check numbers can only consist of digits | `account_check_printing` |
| `_constrains_check_number_unique` | ValidationError | The following numbers are already used: %s | `account_check_printing` |
| `print_checks` | UserError | Payments to print as a checks must have 'Check' selected as payment method and not have already been reconciled | `account_check_printing` |
| `print_checks` | UserError | In order to print multiple checks at once, they must belong to the same bank journal. | `account_check_printing` |
| `do_print_checks` | RedirectWarning | msg | `account_check_printing` |
| `do_print_checks` | RedirectWarning | msg | `account_check_printing` |
| `_create_payment_transaction` | ValidationError | A payment transaction with reference %s already exists. | `account_payment` |
| `_create_payment_transaction` | ValidationError | A token is required to create a new payment transaction. | `account_payment` |
| `write` | UserError | You cannot do this modification since the payment is linked to an expense. | `hr_expense` |
| `_check_move_id` | ValidationError | A payment with any Third Party Check or Own Check payment methods needs an outstanding account | `l10n_latam_check` |
| `action_post` | ValidationError | error_msg | `l10n_latam_check` |
| `_get_reconciled_checks_error` | UserError | You can't cancel or re-open a payment with checks if some check has been debited or been voided. Checks: %s | `l10n_latam_check` |
| `action_open_l10n_ph_2307_wizard` | UserError | Only Outbound Payment is available. | `l10n_ph` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account payment company rule | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (19)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_payment_tree` | list |  | `company_currency_id`, `available_payment_method_line_ids`, `date`, `name`, `journal_id`, `company_id`, `payment_method_line_id`, `partner_id`, `amount_signed`, `amount_signed`, `currency_id`, `activity_ids`, `amount_company_currency_signed`, `state` | `Confirm` |  | `account` |
| `account.view_account_supplier_payment_tree` | field | `account.view_account_payment_tree` | `partner_id` |  |  | `account` |
| `account.view_account_various_payment_tree` | field | `account.view_account_payment_tree` | `partner_id` |  |  | `account` |
| `account.view_account_payment_kanban` | kanban |  | `currency_id`, `partner_id`, `journal_id`, `amount`, `name`, `date`, `activity_ids`, `state` |  |  | `account` |
| `account.view_account_payment_search` | search |  | `name`, `partner_id`, `journal_id`, `company_id` |  | `Customer Payments`, `Vendor Payments`, `Draft`, `In Process`, `Sent`, `Not Sent`, `No Bank Matching`, `Reconciled`, `Payment Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Partner`, `Journal`, `Payment Method Line`, `Status`, `Payment Date`, `Currency`, `Company` | `account` |
| `account.view_account_payment_form` | form |  | `state`, `duplicate_payment_ids`, `id`, `is_sent`, `need_cancel_request`, `is_reconciled`, `is_matched`, `payment_method_code`, `show_partner_bank_account`, `require_partner_bank_account`, `available_payment_method_line_ids`, `available_partner_bank_ids`, `country_code`, `partner_type`, `reconciled_invoices_type`, `company_id`, `paired_internal_transfer_payment_id`, `available_journal_ids`, `currency_id`, `reconciled_invoices_count`, `reconciled_bills_count`, `reconciled_statement_lines_count`, `name`, `payment_type`, `partner_id`, `partner_id`, `amount`, `currency_id`, `date`, `memo`, `journal_id`, `payment_method_line_id`, `partner_bank_id`, `partner_bank_id`, `partner_bank_id`, `qr_code`, `qr_code` | `Confirm`, `Validate`, `Reject`, `Reset to Draft`, `Request Cancel`, `Mark as Sent`, `Unmark as Sent`, `Cancel`, `button_open_invoices`, `button_open_bills`, `button_open_statement_lines`, `button_open_journal_entry` |  | `account` |
| `account.view_account_payment_graph` | graph |  | `payment_type`, `journal_id`, `amount` |  |  | `account` |
| `account_check_printing.view_account_payment_form_inherited` | xpath | `account.view_account_payment_form` |  | `Print Check`, `Unmark Sent`, `Void Check` |  | `account_check_printing` |
| `account_check_printing.view_payment_check_printing_search` | xpath | `account.view_account_payment_search` |  |  | `Checks to Print` | `account_check_printing` |
| `account_payment.view_account_payment_form_inherit_payment` | xpath | `account.view_account_payment_form` | `amount_available_for_refund` | `Refund` |  | `account_payment` |
| `hr_expense.view_payment_form_inherit_expense` | xpath | `account.view_account_payment_form` | `expense_ids` | `action_open_expense` |  | `hr_expense` |
| `l10n_account_withholding_tax.view_account_payment_form` | field | `account.view_account_payment_form` | `memo`, `should_withhold_tax` |  |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_account_payment_form` | page | `l10n_latam_check.view_account_payment_form_inherited` | `l10n_ar_withholding_ids`, `move_name`, `tax_line_id`, `name`, `tax_base_amount`, `amount_currency`, `currency_id` |  |  | `l10n_ar_withholding` |
| `l10n_ch.l10n_ch_account_payment_form` | header | `account.view_account_payment_form` | `l10n_ch_reference_warning_msg` |  |  | `l10n_ch` |
| `l10n_in.view_account_payment_form_inherit_l10n_in_withholding` | xpath | `account.view_account_payment_form` |  | `TDS Entry` |  | `l10n_in` |
| `l10n_latam_check.view_account_payment_form_inherited` | group | `account.view_account_payment_form` | `l10n_latam_new_check_ids`, `company_id`, `currency_id`, `name`, `bank_id`, `issuer_vat`, `payment_date`, `amount`, `l10n_latam_move_check_ids`, `company_id`, `currency_id`, `name`, `bank_id`, `issuer_vat`, `payment_date`, `amount` | `get_formview_action`, `get_formview_action` |  | `l10n_latam_check` |
| `l10n_latam_check.view_account_third_party_check_operations_tree` | list |  | `date`, `name`, `payment_type`, `journal_id`, `partner_id`, `state` |  |  | `l10n_latam_check` |
| `l10n_pl_bank_verification.view_account_payment_form_inherit_l10n_pl` | xpath | `account.view_account_payment_form` | `l10n_pl_verification_status`, `l10n_pl_verification_timestamp`, `l10n_pl_verification_request_id` |  |  | `l10n_pl_bank_verification` |
| `pos_online_payment.view_account_payment_form_inherit_pos_online_payment` | xpath | `account.view_account_payment_form` | `pos_order_id`, `pos_order_id` | `action_view_pos_order` |  | `pos_online_payment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_all_payments` | Payments | list,kanban,form,graph,activity |  |  |  | `account` |
| `account.action_account_payments` | Customer Payments | list,kanban,form,graph,activity |  | `{                 'default_payment_type': 'inbound',                 'default_partner_type': 'customer',                 'search_default_inbound_filter': 1,                 'default_move_journal_types': ('bank', 'cash'),                 'display_account_trust': True,             }` |  | `account` |
| `account.action_account_payments_payable` | Vendor Payments | list,kanban,form,graph,activity |  | `{                 'default_payment_type': 'outbound',                 'default_partner_type': 'supplier',                 'search_default_outbound_filter': 1,                 'default_move_journal_types': ('bank', 'cash'),                 'display_account_trust': True,             }` |  | `account` |
| `account.action_account_payments_transfer` | Internal Transfers | list,kanban,form,graph | `[]` | `{'default_payment_type': 'outbound', 'search_default_transfers_filter': 1, 'display_account_trust': True}` |  | `account` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account.action_account_confirm_payments` | Post Payments | code |  | yes |
| `account_check_printing.action_account_print_checks` | Print Checks | code |  | yes |
| `l10n_ph.action_account_payment_bir_2307` | Download BIR 2307 XLS | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `account.action_report_payment_receipt` | Payment Receipt | qweb-pdf | `account.report_payment_receipt` |  |  |
| `l10n_ph.action_report_disbursement_voucher_ph` | Philippines : Disbursement Voucher | qweb-pdf | `l10n_ph.report_disbursement_voucher` |  |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `account.mail_template_data_payment_receipt` | Payment: Payment Receipt | {{ object.company_id.name }} Payment Receipt (Ref {{ object.name or 'n/a' }}) |

Machine-readable definition: `../../../schemas/data/entities/account.payment.json`; views: `../../../schemas/interfaces/views/account.payment.json`.
