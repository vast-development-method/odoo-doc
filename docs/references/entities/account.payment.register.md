# Pay (`account.payment.register`)

**Transport name:** `account.payment.register`  
**Storage name:** `account_payment_register`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account`  
**Extended by packages:** `account_payment`, `hr_expense`, `l10n_account_withholding_tax`, `l10n_latam_check`, `l10n_ar_withholding`, `l10n_pl_bank_verification`

Description: Pay

## Identity and behavior

- Company consistency is checked automatically on company-bound relations

## Fields (68)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `payment_date` | Payment Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `amount` | Amount | monetary |  | computed by rule `_compute_amount` and stored; currency taken from `currency_id` |
| `hide_writeoff_section` | Hide Writeoff Section | boolean |  | computed by rule `_compute_hide_writeoff_section` (not stored) |
| `communication` | Memo | single line text |  | computed by rule `_compute_communication` and stored |
| `group_payment` | Group Payments | boolean |  | computed by rule `_compute_group_payment` and stored; Help: Only one payment will be created by partner (bank), instead of one per bill. |
| `early_payment_discount_mode` | Early Payment Discount Mode | boolean |  | computed by rule `_compute_early_payment_discount_mode` (not stored) |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored; precomputed before insertion; Help: The payment's currency. |
| `journal_id` | Journal | many to one | `account.journal` | computed by rule `_compute_journal_id` and stored; restricted by domain `[('id', 'in', available_journal_ids)]`; must belong to the same company; precomputed before insertion |
| `available_journal_ids` | Available Journal | many to many | `account.journal` | computed by rule `_compute_available_journal_ids` (not stored) |
| `available_partner_bank_ids` | Available Partner Bank | many to many | `res.partner.bank` | computed by rule `_compute_available_partner_bank_ids` (not stored) |
| `partner_bank_id` | Recipient Bank Account | many to one | `res.partner.bank` | computed by rule `_compute_partner_bank_id` and stored; restricted by domain `[('id', 'in', available_partner_bank_ids)]` |
| `company_currency_id` | Company Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `qr_code` | quick response Code uniform resource locator | rich text |  | computed by rule `_compute_qr_code` (not stored) |
| `batches` | Batches | binary |  | computed by rule `_compute_batches` (not stored) |
| `installments_mode` | Installments Mode | selection |  | computed by rule `_compute_installments_mode` and stored |
| `installments_switch_html` | Installments Switch Hypertext markup language | rich text |  | computed by rule `_compute_installments_switch_values` (not stored) |
| `installments_switch_amount` | Installments Switch Amount | monetary |  | computed by rule `_compute_installments_switch_values` (not stored); currency taken from `currency_id` |
| `custom_user_amount` | Custom User Amount | monetary |  | currency taken from `currency_id` |
| `custom_user_currency_id` | Custom User Currency | many to one | `res.currency` |  |
| `line_ids` | Journal items | many to many | `account.move.line` | read only; not copied on duplication; association table `account_payment_register_move_line_rel` |
| `payment_type` | Payment Type | selection |  | computed by rule `_compute_from_lines` and stored; not copied on duplication |
| `partner_type` | Partner Type | selection |  | computed by rule `_compute_from_lines` and stored; not copied on duplication |
| `source_amount` | Amount to Pay (company currency) | monetary |  | computed by rule `_compute_from_lines` and stored; not copied on duplication; currency taken from `company_currency_id` |
| `source_amount_currency` | Amount to Pay (foreign currency) | monetary |  | computed by rule `_compute_from_lines` and stored; not copied on duplication; currency taken from `source_currency_id` |
| `source_currency_id` | Source Currency | many to one | `res.currency` | computed by rule `_compute_from_lines` and stored; not copied on duplication |
| `can_edit_wizard` | Can Edit Wizard | boolean |  | computed by rule `_compute_from_lines` and stored; not copied on duplication |
| `can_group_payments` | Can Group Payments | boolean |  | computed by rule `_compute_can_group_payments` and stored; not copied on duplication |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_from_lines` and stored; not copied on duplication |
| `partner_id` | Customer/Vendor | many to one | `res.partner` | computed by rule `_compute_from_lines` and stored; not copied on duplication; on delete of the target: restrict |
| `payment_method_line_id` | Payment Method | many to one | `account.payment.method.line` | computed by rule `_compute_payment_method_line_id` and stored; restricted by domain `[('id', 'in', available_payment_method_line_ids)]`; Help: Manual: Pay or Get paid by any method outside of the system. Payment Providers: Each payment provider has its own Payment Method. Request a transaction on/to a card thanks to a payment token saved by the partner when buying or subscribing online. Check: Pay bills by check and print it from the system. Batch Deposit: Collect several customer checks at once generating and submitting a batch deposit to your bank. Module account_batch_payment is necessary. SEPA Credit Transfer: Pay in the SEPA zone by submitting a SEPA Credit Transfer file to your bank. Module account_sepa is necessary. SEPA Direct Debit: Get paid in the SEPA zone thanks to a mandate your partner will have granted to you. Module account_sepa is necessary. |
| `available_payment_method_line_ids` | Available Payment Method Line | many to many | `account.payment.method.line` | computed by rule `_compute_payment_method_line_fields` (not stored) |
| `payment_method_code` | Payment Method Code | single line text |  | related through path `payment_method_line_id.code` |
| `payment_difference` | Payment Difference | monetary |  | computed by rule `_compute_payment_difference` (not stored) |
| `payment_difference_handling` | Payment Difference Handling | selection |  | computed by rule `_compute_payment_difference_handling` and stored |
| `writeoff_account_id` | Difference Account | many to one | `account.account` | not copied on duplication; must belong to the same company |
| `writeoff_label` | Journal Item Label | single line text |  | default `Write-Off`; Help: Change label of the counterpart that will hold the payment difference |
| `writeoff_is_exchange_account` | Writeoff Is Exchange Account | boolean |  | computed by rule `_compute_writeoff_is_exchange_account` (not stored) |
| `show_payment_difference` | Show Payment Difference | boolean |  | computed by rule `_compute_show_payment_difference` (not stored) |
| `show_partner_bank_account` | Show Partner Bank Account | boolean |  | computed by rule `_compute_show_require_partner_bank` (not stored) |
| `require_partner_bank_account` | Require Partner Bank Account | boolean |  | computed by rule `_compute_show_require_partner_bank` (not stored) |
| `country_code` | Country Code | single line text |  | read only; related through path `company_id.account_fiscal_country_id.code` |
| `duplicate_payment_ids` | Duplicate Payment | many to many | `account.payment` | computed by rule `_compute_duplicate_moves` (not stored) |
| `is_register_payment_on_draft` | Is Register Payment On Draft | boolean |  | computed by rule `_compute_is_register_payment_on_draft` (not stored) |
| `actionable_errors` | Actionable Errors | structured document |  | computed by rule `_compute_actionable_errors` (not stored) |
| `untrusted_bank_ids` | Untrusted Bank | many to many | `res.partner.bank` | computed by rule `_compute_trust_values` (not stored) |
| `total_payments_amount` | Total Payments Amount | integer |  | computed by rule `_compute_trust_values` (not stored) |
| `untrusted_payments_count` | Untrusted Payments Count | integer |  | computed by rule `_compute_trust_values` (not stored) |
| `missing_account_partners` | Missing Account Partners | many to many | `res.partner` | computed by rule `_compute_trust_values` (not stored) |
| `payment_token_id` | Saved payment token | many to one | `payment.token` | computed by rule `_compute_payment_token_id` and stored; restricted by domain `[             ('id', 'in', suitable_payment_token_ids),         ]`; Help: Note that tokens from providers set to only authorize transactions (instead of capturing the amount) are not available. |
| `suitable_payment_token_ids` | Suitable Payment Token | many to many | `payment.token` | computed by rule `_compute_suitable_payment_token_ids` (not stored) |
| `use_electronic_payment_method` | Use Electronic Payment Method | boolean |  | computed by rule `_compute_use_electronic_payment_method` (not stored) |
| `display_withholding` | Display Withholding | boolean |  | computed by rule `_compute_display_withholding` (not stored) |
| `should_withhold_tax` | Withhold Tax Amounts | boolean |  | computed by rule `_compute_should_withhold_tax` and stored; not copied on duplication |
| `withholding_line_ids` | Withholding Lines | one to many | `account.payment.register.withholding.line` | computed by rule `_compute_withholding_line_ids` and stored; inverse field `payment_register_id` |
| `withholding_net_amount` | Net Amount | monetary |  | computed by rule `_compute_withholding_net_amount` and stored; Help: Net amount after deducting the withholding lines |
| `withholding_default_account_id` | Withholding Default Account | many to one |  | related through path `journal_id.default_account_id` |
| `withholding_outstanding_account_id` | Outstanding Account | many to one | `account.account` | computed by rule `_compute_withholding_outstanding_account_id` and stored; not copied on duplication; restricted by domain `['\|', ('account_type', 'in', ('asset_current', 'liability_current')), ('id', '=', withholding_default_account_id)]`; must belong to the same company |
| `withholding_payment_account_id` | Withholding Payment Account | many to one |  | related through path `payment_method_line_id.payment_account_id` |
| `withholding_hide_tax_base_account` | Withholding Hide Tax Base Account | boolean |  | computed by rule `_compute_withholding_hide_tax_base_account` (not stored) |
| `l10n_latam_new_check_ids` | New Checks | one to many | `l10n_latam.payment.register.check` | inverse field `payment_register_id` |
| `l10n_latam_move_check_ids` | Checks | many to many | `l10n_latam.check` |  |
| `l10n_ar_withholding_ids` | Withholdings | one to many | `l10n_ar.payment.register.withholding` | computed by rule `_compute_l10n_ar_withholding_ids` and stored; inverse field `payment_register_id` |
| `l10n_ar_net_amount` | Localization Ar Net Amount | monetary |  | read only; computed by rule `_compute_l10n_ar_net_amount` (not stored); Help: Net amount after withholdings |
| `l10n_ar_adjustment_warning` | Localization Ar Adjustment Warning | boolean |  | computed by rule `_compute_l10n_ar_adjustment_warning` (not stored) |
| `l10n_pl_bank_verification_ids` | Localization Pl Bank Verification | many to many | `l10n_pl.bank.account.verification` | computed by rule `_compute_l10n_pl_bank_verification` (not stored) |
| `l10n_pl_bank_verification_invalid_bank_account_ids` | Localization Pl Bank Verification Invalid Bank Account | many to many | `res.partner.bank` | computed by rule `_compute_l10n_pl_bank_verification` (not stored) |
| `l10n_pl_not_found_partner_ids` | Localization Pl Not Found Partner | many to many | `res.partner` | computed by rule `_compute_l10n_pl_bank_verification` (not stored) |
| `l10n_pl_incomplete_data_partner_ids` | Localization Pl Incomplete Data Partner | many to many | `res.partner` | computed by rule `_compute_l10n_pl_bank_verification` (not stored) |

## Selection values

### `installments_mode` (Installments Mode)

| Value | Label |
|---|---|
| `next` | Next Installment |
| `overdue` | Overdue Amount |
| `before_date` | Before Next Payment Date |
| `full` | Full Amount |

### `payment_type` (Payment Type)

| Value | Label |
|---|---|
| `outbound` | Send Money |
| `inbound` | Receive Money |

### `partner_type` (Partner Type)

| Value | Label |
|---|---|
| `customer` | Customer |
| `supplier` | Vendor |

### `payment_difference_handling` (Payment Difference Handling)

| Value | Label |
|---|---|
| `open` | Keep open |
| `reconcile` | Mark as fully paid |

## Operations (72)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_communication` | preparation rule | self, lines | `account` | model | Helper to compute the communication based on lines. :param lines:           A recordset of the `account.move.line`'s that will be reconciled. :return:                A string representing a communication to be set on payment. |
| `_get_batch_available_journals` | preparation rule | self, batch_result | `account` | model | Helper to compute the available journals based on the batch.  :param batch_result:    A batch computed by '_compute_batches'. :return:                A recordset of account.journal. |
| `_get_batch_journal` | preparation rule | self, batch_result | `account` | model | Helper to compute the journal based on the batch.  :param batch_result:    A batch computed by '_compute_batches'. :return:                An account.journal record. |
| `_get_batch_available_partner_banks` | preparation rule | self, batch_result, journal | `account` | model |  |
| `_get_line_batch_key` | preparation rule | self, line | `account`, `hr_expense` | model | Turn the line passed as parameter to a dictionary defining on which way the lines will be grouped together. :return: A python dictionary. |
| `_get_wizard_values_from_batch` | preparation rule | self, batch_result | `account` | model | Extract values from the batch passed as parameter (see '_compute_batches') to be mounted in the wizard view. :param batch_result:    A batch computed by '_compute_batches'. :return:                A dictionary containing valid fields |
| `_from_sibling_companies` | internal rule | self, lines | `account` | model |  |
| `_compute_show_payment_difference` | computation | self | `account` | depends: `early_payment_discount_mode`, `can_edit_wizard`, `can_group_payments`, `group_payment`, `payment_method_line_id` |  |
| `_compute_batches` | computation | self | `account` | depends: `line_ids` | Group the account.move.line linked to the wizard together. Lines are grouped if they share 'partner_id','account_id','currency_id' & 'partner_type' and if 0 or 1 partner_bank_id can be determined for the group.  Computes a list of batches, each one containing:     * payment_values:   A dictionary of payment values.     * moves:        An account.move recordset. |
| `_compute_trust_values` | computation | self | `account` | depends: `payment_method_line_id`, `line_ids`, `group_payment`, `partner_bank_id` |  |
| `_compute_from_lines` | computation | self | `account` | depends: `line_ids` | Load initial values from the account.moves passed through the context. |
| `_compute_can_group_payments` | computation | self | `account` | depends: `batches`, `amount` |  |
| `_compute_communication` | computation | self | `account` | depends: `can_edit_wizard`, `amount` |  |
| `_compute_group_payment` | computation | self | `account` | depends: `can_edit_wizard` |  |
| `_compute_currency_id` | computation | self | `account`, `l10n_latam_check` | depends: `journal_id`; depends: `l10n_latam_move_check_ids.currency_id` |  |
| `_compute_available_journal_ids` | computation | self | `account` | depends: `payment_type`, `company_id`, `can_edit_wizard` |  |
| `_compute_journal_id` | computation | self | `account` | depends: `available_journal_ids` |  |
| `_compute_available_partner_bank_ids` | computation | self | `account` | depends: `can_edit_wizard`, `journal_id` |  |
| `_compute_partner_bank_id` | computation | self | `account` | depends: `journal_id`, `available_partner_bank_ids` |  |
| `_compute_payment_method_line_fields` | computation | self | `account` | depends: `payment_type`, `journal_id`, `currency_id` |  |
| `_compute_payment_method_line_id` | computation | self | `account` | depends: `payment_type`, `journal_id` |  |
| `_compute_show_require_partner_bank` | computation | self | `account` | depends: `payment_method_line_id` | Computes if the destination bank account must be displayed in the payment form view. By default, it won't be displayed but some modules might change that, depending on the payment type. |
| `_compute_actionable_errors` | computation | self | `account` | depends: `line_ids` |  |
| `_convert_to_wizard_currency` | internal rule | self, installments | `account` |  |  |
| `_get_total_amounts_to_pay` | preparation rule | self, batch_results | `account` |  |  |
| `_onchange_amount` | on change | self | `account` | onchange: `amount` |  |
| `_onchange_currency_id` | on change | self | `account` | onchange: `currency_id` |  |
| `_onchange_payment_date` | on change | self | `account` | onchange: `payment_date` |  |
| `_compute_amount` | computation | self | `account`, `l10n_ar_withholding`, `l10n_latam_check` | depends: `can_edit_wizard`, `source_amount`, `source_amount_currency`, `source_currency_id`, `company_id`, `currency_id`, `payment_date`, `installments_mode`; depends: `l10n_latam_move_check_ids.amount`, `l10n_latam_new_check_ids.amount`, `payment_method_code`; depends: `can_edit_wizard`, `source_amount`, `source_amount_currency`, `source_currency_id`, `company_id`, `currency_id`, `payment_date`, `installments_mode`, `l10n_latam_move_check_ids.amount`, `l10n_latam_new_check_ids.amount`, `payment_method_code` |  |
| `_compute_installments_mode` | computation | self | `account` | depends: `amount` |  |
| `_compute_installments_switch_values` | computation | self | `account` | depends: `installments_mode` |  |
| `_compute_early_payment_discount_mode` | computation | self | `account` | depends: `can_edit_wizard`, `payment_date`, `currency_id`, `amount` |  |
| `_compute_payment_difference` | computation | self | `account` | depends: `can_edit_wizard`, `amount`, `installments_mode` |  |
| `_compute_writeoff_is_exchange_account` | computation | self | `account` | depends: `can_edit_wizard`, `writeoff_account_id`, `payment_difference_handling`, `currency_id` |  |
| `_compute_payment_difference_handling` | computation | self | `account` | depends: `early_payment_discount_mode` |  |
| `_compute_hide_writeoff_section` | computation | self | `account` | depends: `early_payment_discount_mode` |  |
| `_compute_qr_code` | computation | self | `account` | depends: `partner_bank_id`, `amount`, `currency_id`, `payment_method_line_id`, `payment_type`, `communication` |  |
| `_compute_duplicate_moves` | computation | self | `account` | depends: `partner_id`, `amount`, `payment_date`, `payment_type`, `line_ids` |  |
| `_compute_is_register_payment_on_draft` | computation | self | `account` | depends: `line_ids` |  |
| `_fetch_duplicate_reference` | internal rule | self, matching_states | `account` |  | Retrieve move ids for possible duplicates of payments. Duplicates moves: - Have the same partner_id, amount and date as the payment - Are not reconciled - Represent a credit in the same account receivable or a debit in the same account payable as the payment, or - Represent a credit in outstanding receipts or debit in outstanding payments, so bank statement lines with an  outstanding counterpart can be matched, or - Are in the suspense account |
| `default_get` | lifecycle override | self, fields | `account` | model |  |
| `_create_payment_vals_from_wizard` | internal rule | self, batch_result | `account_payment`, `account`, `l10n_account_withholding_tax`, `l10n_ar_withholding`, `l10n_latam_check`, `l10n_pl_bank_verification` |  | Update the computation of the payment vals in order to correctly set the outstanding account as well as the withholding line when needed. |
| `_create_payment_vals_from_batch` | internal rule | self, batch_result | `account`, `l10n_pl_bank_verification` |  |  |
| `_init_payments` | internal rule | self, to_process, edit_mode | `account`, `hr_expense` |  | Create the payments.  :param to_process:  A list of python dictionary, one for each payment to create, containing:                     * create_vals:  The values used for the 'create' method.                     * to_reconcile: The journal items to perform the reconciliation.                     * batch:        A python dict containing everything you want about the source journal items                                     to which a payment will be created (see '_compute_batches'). :param edit_mode:   Is the wizard in edition mode. |
| `_post_payments` | internal rule | self, to_process, edit_mode | `account` |  | Post the newly created payments.  :param to_process:  A list of python dictionary, one for each payment to create, containing:                     * create_vals:  The values used for the 'create' method.                     * to_reconcile: The journal items to perform the reconciliation.                     * batch:        A python dict containing everything you want about the source journal items                                     to which a payment will be created (see '_compute_batches'). :param edit_mode:   Is the wizard in edition mode. |
| `_reconcile_payments` | internal rule | self, to_process, edit_mode | `account` |  | Reconcile the payments.  :param to_process:  A list of python dictionary, one for each payment to create, containing:                     * create_vals:  The values used for the 'create' method.                     * to_reconcile: The journal items to perform the reconciliation.                     * batch:        A python dict containing everything you want about the source journal items                                     to which a payment will be created (see '_compute_batches'). :param edit_mode:   Is the wizard in edition mode. |
| `_create_payments` | internal rule | self | `account` |  |  |
| `_get_next_payment_date_in_context` | preparation rule | self | `account` |  |  |
| `action_create_payments` | user action | self | `account`, `l10n_ar_withholding`, `l10n_latam_check` |  |  |
| `_get_batch_account` | preparation rule | self, batch_result | `account` |  |  |
| `action_open_untrusted_bank_accounts` | user action | self | `account` |  |  |
| `action_open_missing_account_partners` | user action | self | `account` |  |  |
| `_compute_suitable_payment_token_ids` | computation | self | `account_payment` | depends: `payment_method_line_id` |  |
| `_compute_use_electronic_payment_method` | computation | self | `account_payment` | depends: `payment_method_line_id` |  |
| `_compute_payment_token_id` | computation | self | `account_payment` | depends: `can_edit_wizard`, `suitable_payment_token_ids`, `journal_id` |  |
| `_compute_withholding_net_amount` | computation | self | `l10n_account_withholding_tax` | depends: `withholding_line_ids.amount`, `amount` | The net amount is the one that will actually be paid by the payer. It is simply the payment amount - the sum of withholding taxes. |
| `_compute_withholding_outstanding_account_id` | computation | self | `l10n_account_withholding_tax` | depends: `withholding_payment_account_id`, `should_withhold_tax` | We propose a default account by getting one from the latest payment which:  - Has the same payment method line id (and thus indirectly the same journal, and thus the same company)  - That payment method has no payment_account_id  - Yet the payment has an outstanding_account_id |
| `_compute_display_withholding` | computation | self | `l10n_account_withholding_tax` | depends: `company_id`, `can_edit_wizard`, `can_group_payments`, `group_payment` | The withholding feature should not show on companies which does not contain any withholding taxes. |
| `_compute_withholding_line_ids` | computation | self | `l10n_account_withholding_tax` | depends: `can_edit_wizard`, `display_withholding` | When opening the wizard, we want to compute the default withholding lines by looking at the invoice lines to see if they have withholding taxes set on them. |
| `_compute_should_withhold_tax` | computation | self | `l10n_account_withholding_tax` | depends: `withholding_line_ids` | Ensures that we display the line table if any withholding line has been added to the payment. |
| `_compute_withholding_hide_tax_base_account` | computation | self | `l10n_account_withholding_tax` | depends: `company_id` | When the withholding tax base account is set in the setting, simplify the view by hiding the account column on the lines as we will default to that tax base account. |
| `_onchange_withholding_line_ids` | on change | self | `l10n_account_withholding_tax` | onchange: `withholding_line_ids` | Any time a line is edited, we want to check if we need to recompute the placeholders. The idea is to try and display accurate placeholders on lines whose tax have a sequence set. |
| `_get_total_amount_in_wizard_currency` | preparation rule | self | `l10n_account_withholding_tax` |  | Returns the total amount of the first batch, in the currency of the wizard. This information can be used to determine if we are doing a partial payment or not. |
| `_is_latam_check_payment` | internal rule | self, check_subtype | `l10n_latam_check` |  |  |
| `_compute_l10n_ar_adjustment_warning` | computation | self | `l10n_ar_withholding` | depends: `amount`, `l10n_latam_move_check_ids`, `l10n_latam_new_check_ids`, `payment_method_code` |  |
| `_compute_l10n_ar_net_amount` | computation | self | `l10n_ar_withholding` | depends: `amount`, `l10n_ar_withholding_ids.amount` |  |
| `_get_conversion_rate` | preparation rule | self | `l10n_ar_withholding` |  |  |
| `_compute_l10n_ar_withholding_ids` | computation | self | `l10n_ar_withholding` | depends: `partner_id`, `payment_date` |  |
| `_compute_l10n_pl_bank_verification` | computation | self | `l10n_pl_bank_verification` | depends: `line_ids`, `partner_bank_id` |  |
| `_batch_need_check` | internal rule | self, batch | `l10n_pl_bank_verification` | model | Does batch need a government API call to check if the partner vat is linked to its account number |
| `_update_payment_vals` | internal rule | self, payment_vals, batch_result | `l10n_pl_bank_verification` |  |  |
| `_get_partner_bank_from_batch` | preparation rule | self, batch | `l10n_pl_bank_verification` | model |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_batches` | UserError | You can't create payments for entries belonging to different companies. | `account` |
| `_compute_batches` | UserError | You can't open the register payment wizard without at least one receivable/payable line. | `account` |
| `default_get` | UserError | There's nothing left to pay for the selected journal items, so no payment registration is necessary. You've got your finances under control like a boss! | `account` |
| `default_get` | UserError | You can't create payments for entries belonging to different companies. | `account` |
| `default_get` | UserError | You can't create payments for entries belonging to different branches without access to parent company. | `account` |
| `default_get` | UserError | You can't register payments for both inbound and outbound moves at the same time. | `account` |
| `default_get` | UserError | You cannot register payments for blocked invoices. | `account` |
| `default_get` | UserError | The register payment wizard should only be called on account.move or account.move.line records. | `account` |
| `_create_payments` | UserError | To record payments with %(payment_method)s, the recipient bank account must be manually validated. You should go on the partner bank account in order to validate it. | `account` |
| `_create_payment_vals_from_wizard` | UserError | The withholding net amount cannot be negative. | `l10n_account_withholding_tax` |
| `action_create_payments` | ValidationError | You can't mix checks of different currencies in one payment, and you can't change the payment's currency if checks are already created in that currency. Please create separate payments for each currency. | `l10n_latam_check` |
| `_create_payment_vals_from_wizard` | UserError | Please enter withholding number for tax %s | `l10n_ar_withholding` |
| `action_create_payments` | ValidationError | A payment cannot have withholding if the payment method has no outstanding accounts | `l10n_ar_withholding` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | no | `account` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_payment_register_form` | form |  | `line_ids`, `can_edit_wizard`, `can_group_payments`, `early_payment_discount_mode`, `installments_mode`, `installments_switch_amount`, `installments_switch_html`, `payment_type`, `partner_type`, `source_amount`, `source_amount_currency`, `source_currency_id`, `company_id`, `partner_id`, `country_code`, `currency_id`, `custom_user_amount`, `custom_user_currency_id`, `show_partner_bank_account`, `require_partner_bank_account`, `available_journal_ids`, `available_payment_method_line_ids`, `available_partner_bank_ids`, `company_currency_id`, `hide_writeoff_section`, `writeoff_is_exchange_account`, `untrusted_bank_ids`, `missing_account_partners`, `payment_difference`, `untrusted_payments_count`, `total_payments_amount`, `duplicate_payment_ids`, `actionable_errors`, `journal_id`, `payment_method_line_id`, `partner_bank_id`, `group_payment`, `payment_difference`, `payment_difference_handling`, `writeoff_account_id`, `writeoff_label`, `amount`, `currency_id`, `installments_switch_html`, `payment_date`, `communication`, `qr_code`, `qr_code` | `action_open_untrusted_bank_accounts`, `action_open_missing_account_partners`, `Create Payments`, `Create Payment`, `Discard` |  | `account` |
| `account_payment.view_account_payment_register_form_inherit_payment` | field | `account.view_account_payment_register_form` | `payment_method_line_id`, `payment_method_code`, `suitable_payment_token_ids`, `use_electronic_payment_method`, `payment_token_id` |  |  | `account_payment` |
| `l10n_account_withholding_tax.view_account_payment_register_form` | group | `account.view_account_payment_register_form` | `should_withhold_tax` |  |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_account_payment_register_form` | page | `l10n_latam_check.view_account_payment_register_form` | `l10n_ar_withholding_ids`, `withholding_sequence_id`, `company_id`, `currency_id`, `tax_id`, `name`, `base_amount`, `amount` |  |  | `l10n_ar_withholding` |
| `l10n_latam_check.view_account_payment_register_form` | group | `account.view_account_payment_register_form` | `l10n_latam_new_check_ids`, `company_id`, `currency_id`, `name`, `bank_id`, `issuer_vat`, `payment_date`, `amount`, `l10n_latam_move_check_ids`, `company_id`, `currency_id`, `name`, `bank_id`, `issuer_vat`, `payment_date`, `amount` |  |  | `l10n_latam_check` |
| `l10n_pl_bank_verification.l10n_pl_view_account_payment_register_form` | xpath | `account.view_account_payment_register_form` | `l10n_pl_bank_verification_invalid_bank_account_ids`, `l10n_pl_not_found_partner_ids`, `l10n_pl_incomplete_data_partner_ids` |  |  | `l10n_pl_bank_verification` |

Machine-readable definition: `../../../schemas/data/entities/account.payment.register.json`; views: `../../../schemas/interfaces/views/account.payment.register.json`.
