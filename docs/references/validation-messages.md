# Validation messages

Every user-facing validation or error message, with the entity and operation that raises it.

| Entity | Operation | Kind | Message |
|---|---|---|---|
| [`account.account`](entities/account.account.md) | `_check_reconcile` | ValidationError | You cannot have a receivable/payable account that is not reconcilable. (account code: %s) |
| [`account.account`](entities/account.account.md) | `_constrains_reconcile` | UserError | An Off-Balance account can not be reconcilable |
| [`account.account`](entities/account.account.md) | `_constrains_reconcile` | UserError | An Off-Balance account can not have taxes |
| [`account.account`](entities/account.account.md) | `_check_journal_consistency` | ValidationError | The foreign currency set on the journal '%(journal)s' and the account '%(account)s' must be the same. |
| [`account.account`](entities/account.account.md) | `_check_company_consistency` | ValidationError | The following accounts must be assigned to at least one company: %(accounts)s |
| [`account.account`](entities/account.account.md) | `_check_company_consistency` | ValidationError | Bank & Cash accounts cannot be shared between companies. |
| [`account.account`](entities/account.account.md) | `_check_company_consistency` | UserError | You can't unlink this company from this account since there are some journal items linked to it. |
| [`account.account`](entities/account.account.md) | `_check_account_type_sales_purchase_journal` | ValidationError | The account is already in use in a 'sale' or 'purchase' journal. This means that the account's type couldn't be 'receivable' or 'payable'. |
| [`account.account`](entities/account.account.md) | `_check_account_code` | ValidationError | The account code can only contain alphanumeric characters and dots. (account code: %s) |
| [`account.account`](entities/account.account.md) | `_check_account_is_bank_journal_bank_account` | ValidationError | You cannot change the type of an account set as Bank Account on a journal to Receivable or Payable. |
| [`account.account`](entities/account.account.md) | `_search_new_account_code` | UserError | Cannot generate an unused account code. |
| [`account.account`](entities/account.account.md) | `_toggle_reconcile_to_false` | UserError | You cannot switch an account to prevent the reconciliation if some partial reconciliations are still pending. |
| [`account.account`](entities/account.account.md) | `name_create` | ValidationError | Please create new accounts from the Chart of Accounts menu. |
| [`account.account`](entities/account.account.md) | `write` | UserError | You cannot deprecate an account that is used in a tax distribution. |
| [`account.account`](entities/account.account.md) | `write` | UserError | You cannot set a currency on this account as it already has some journal entries having a different foreign currency. |
| [`account.account`](entities/account.account.md) | `_ensure_code_is_unique` | ValidationError | Account codes must be unique. You can't create accounts with these duplicate codes: %s |
| [`account.account`](entities/account.account.md) | `_ensure_code_is_unique` | ValidationError | The code must be set for every company to which this account belongs. |
| [`account.account`](entities/account.account.md) | `_unlink_except_contains_journal_items` | UserError | You cannot perform this action on an account that contains journal items. |
| [`account.account`](entities/account.account.md) | `_unlink_except_linked_to_fiscal_position` | UserError | You cannot remove/deactivate the accounts "%s" which are set on the account mapping of a fiscal position. |
| [`account.account`](entities/account.account.md) | `_unlink_except_linked_to_tax_repartition_line` | UserError | You cannot remove/deactivate the accounts "%s" which are set on a tax repartition line. |
| [`account.account`](entities/account.account.md) | `_merge_method` | UserError | You cannot merge accounts. |
| [`account.account`](entities/account.account.md) | `_check_action_unmerge_possible` | UserError | You do not have the right to perform this operation as you do not have access to the following companies: %s. |
| [`account.account`](entities/account.account.md) | `_check_action_unmerge_possible` | UserError | Account %s cannot be unmerged as it already belongs to a single company. The unmerge operation only splits an account based on its companies. |
| [`account.account`](entities/account.account.md) | `_action_unmerge_get_user_confirmation` | RedirectWarning | msg |
| [`account.account`](entities/account.account.md) | `write` | UserError | You can not change the code of an account. |
| [`account.account`](entities/account.account.md) | `_unlink_bank_cash_accounts` | UserError | You must keep at least one bank and cash account for %(company)s! |
| [`account.account.tag`](entities/account.account.tag.md) | `_unlink_except_master_tags` | UserError | You cannot delete this account tag (%s), it is used on the chart of account definition. |
| [`account.accrued.orders.wizard`](entities/account.accrued.orders.wizard.md) | `_compute_move_vals` | UserError | Entries can only be created for a single company at a time. |
| [`account.accrued.orders.wizard`](entities/account.accrued.orders.wizard.md) | `_compute_move_vals` | UserError | Cannot create an accrual entry with orders in different currencies. |
| [`account.accrued.orders.wizard`](entities/account.accrued.orders.wizard.md) | `create_entries` | UserError | Reversal date must be posterior to date. |
| [`account.analytic.account`](entities/account.analytic.account.md) | `_check_company_consistency` | UserError | You can't change the company of an analytic account that already has analytic items! It's a recipe for an analytical disaster! |
| [`account.analytic.account`](entities/account.analytic.account.md) | `_unlink_except_account_in_analytic_distribution` | UserError | You cannot delete an analytic account that is used in an expense. |
| [`account.analytic.account`](entities/account.analytic.account.md) | `_unlink_except_existing_tasks` | UserError | Before we can bid farewell to these accounts, you need to tidy up the projects linked to them by removing their existing tasks! |
| [`account.analytic.distribution.model`](entities/account.analytic.distribution.model.md) | `_check_company_accounts` | UserError | You defined a distribution with analytic account(s) belonging to a specific company but a model shared between companies or with a different company |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_general_account_id` | ValidationError | The journal item is not linked to the correct financial account |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_can_write` | AccessError | You cannot access timesheets that are not yours. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `create` | ValidationError | error_msg |
| [`account.analytic.line`](entities/account.analytic.line.md) | `create` | ValidationError | error_msg |
| [`account.analytic.line`](entities/account.analytic.line.md) | `create` | ValidationError | Timesheets cannot be created on a private task. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `write` | ValidationError | Timesheets cannot be created on a private task. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `write` | UserError | You cannot set an archived employee on existing timesheets. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_timesheet_preprocess_get_accounts` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the timesheet. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_timesheet_postprocess_values` | ValidationError | Timesheets must be created with at least an active analytic account defined in the plan '%(plan_name)s'. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_timesheet_postprocess_values` | ValidationError | The project, the task and the analytic accounts of the timesheet must belong to the same company. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_unlink_except_linked_leave` | UserError | You cannot delete timesheets that are linked to global time off. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_unlink_except_linked_leave` | RedirectWarning | error_message |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_unlink_except_linked_leave` | UserError | error_message |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_can_write` | UserError | Timesheets linked to public holidays cannot be modified. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_can_write` | UserError | You cannot modify timesheets that are linked to time off requests. Please use the Time Off application to modify your time off requests instead. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_can_create` | UserError | You cannot create timesheets for a task that is linked to a time off type. Please use the Time Off application to request new time off instead. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_check_can_write` | UserError | You cannot modify timesheets that are already invoiced. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_unlink_except_invoiced` | UserError | You cannot remove a timesheet that has already been invoiced. |
| [`account.analytic.line`](entities/account.analytic.line.md) | `_timesheet_preprocess_get_accounts` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the analytic distribution of the sale order item '%(so_line_name)s' linked to the timesheet. |
| [`account.analytic.plan`](entities/account.analytic.plan.md) | `__get_all_plans` | UserError | A 'Project' plan needs to exist and its id needs to be set as `analytic.project_plan` in the system variables |
| [`account.analytic.plan`](entities/account.analytic.plan.md) | `_onchange_parent_id` | UserError | You cannot add a parent to the base plan '%s' |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `_constraint_percentage` | UserError | Percentage must be between 0 and 100 |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `_check_date` | ValidationError | The date selected is protected by: %(lock_date_info)s. |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `default_get` | UserError | This can only be used on journal items |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `default_get` | UserError | Oops! You can only change the period or account for posted entries! Other ones aren't up for an adventure like that! |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `default_get` | UserError | Oops! You can only change the period or account for items that are not yet reconciled! Other ones aren't up for an adventure like that! |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `default_get` | UserError | You cannot use this wizard on journal entries belonging to different companies. |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `default_get` | UserError | No possible action found with the selected lines. |
| [`account.automatic.entry.wizard`](entities/account.automatic.entry.wizard.md) | `_compute_move_data` | UserError | All accounts on the lines must be of the same type. |
| [`account.bank.statement`](entities/account.bank.statement.md) | `default_get` | UserError | A statement should only contain lines from the same journal. |
| [`account.bank.statement`](entities/account.bank.statement.md) | `default_get` | UserError | Unable to create a statement due to missing transactions. You may want to reorder the transactions before proceeding. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_check_amounts_currencies` | ValidationError | The foreign currency must be different than the journal one: %s |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_check_amounts_currencies` | ValidationError | You can't provide an amount in foreign currency without specifying a foreign currency. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_check_amounts_currencies` | ValidationError | You can't provide a foreign currency without specifying an amount in 'Amount in Currency' field. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `action_undo_reconciliation` | ValidationError | Validated entries can only be changed by your accountant. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_check_allow_unlink` | UserError | You can not delete a transaction from a valid statement. If you want to delete it, please remove the statement first. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_prepare_move_line_default_vals` | UserError | You can't create a new statement line without a suspense account set on the %s journal. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_synchronize_from_moves` | UserError | The journal entry %s reached an invalid state regarding its related statement line. To be consistent, the journal entry must always have exactly one journal item involving the bank/cash account. |
| [`account.bank.statement.line`](entities/account.bank.statement.line.md) | `_synchronize_from_moves` | UserError | %(move)s reached an invalid state regarding its related statement line. To be consistent, the journal entry must always have exactly one suspense line. |
| [`account.cash.rounding`](entities/account.cash.rounding.md) | `validate_rounding` | ValidationError | Please set a strictly positive rounding value. |
| [`account.cash.rounding`](entities/account.cash.rounding.md) | `_unlink_except_pos_config` | UserError | You cannot delete a rounding method that is used in a Point of Sale configuration. |
| [`account.cash.rounding`](entities/account.cash.rounding.md) | `_check_session_state` | ValidationError | You are not allowed to change the cash rounding configuration while a pos session using it is already opened. |
| [`account.chart.template`](entities/account.chart.template.md) | `try_loading` | UserError | The %s chart template shouldn't be selected directly. Instead, you should directly select the chart template related to your country. |
| [`account.chart.template`](entities/account.chart.template.md) | `_load` | AccessError | Only administrators can install chart templates |
| [`account.code.mapping`](entities/account.code.mapping.md) | `_search` | UserError | Account Code Mapping cannot be accessed directly. It is designed to be used only through the Chart of Accounts. |
| [`account.debit.note`](entities/account.debit.note.md) | `default_get` | UserError | You can only debit posted moves. |
| [`account.debit.note`](entities/account.debit.note.md) | `default_get` | UserError | You can't make a debit note for an invoice that is already linked to a debit note. |
| [`account.debit.note`](entities/account.debit.note.md) | `default_get` | UserError | You can make a debit note only for a Customer Invoice, a Customer Credit Note, a Vendor Bill or a Vendor Credit Note. |
| [`account.edi.common`](entities/account.edi.common.md) | `_validate_taxes` | ValidationError | error_msg |
| [`account.edi.document`](entities/account.edi.document.md) | `_process_documents_web_services` | UserError | This document is being sent by another process already. |
| [`account.edi.xml.ubl.tr`](entities/account.edi.xml.ubl.tr.md) | `_add_invoice_header_nodes` | UserError | Nilvera portal cannot process negative quantity nor negative price on invoice lines |
| [`account.financial.year.op`](entities/account.financial.year.op.md) | `_check_fiscalyear` | ValidationError | Incorrect fiscal year date: day is out of range for month. Month: %(month)s; Day: %(day)s |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_check_zip` | ValidationError | Invalid "Zip Range", You have to configure both "From" and "To" values for the zip range and "To" should be greater than "From". |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_validate_foreign_vat_country` | ValidationError | The country of the foreign VAT number could not be detected. Please assign a country to the fiscal position. |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_validate_foreign_vat_country` | ValidationError | A fiscal position with a foreign VAT already exists in this country. |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_validate_foreign_vat_country` | ValidationError | You cannot create a fiscal position with a country outside of the selected country group. |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_validate_foreign_vat_country` | ValidationError | You cannot create a fiscal position with a foreign VAT within your fiscal country without assigning it a state. |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `write` | UserError | You cannot modify a fiscal position used in a POS order. You should archive it and create a new one. |
| [`account.fiscal.position`](entities/account.fiscal.position.md) | `_never_unlink_declaration_of_intent_fiscal_position` | UserError | You cannot delete the special fiscal position for Declarations of Intent. |
| [`account.group`](entities/account.group.md) | `_constraint_prefix_overlap` | ValidationError | Account Groups with the same granularity can't overlap |
| [`account.group`](entities/account.group.md) | `_check_parent_not_circular` | ValidationError | You cannot create recursive groups. |
| [`account.journal`](entities/account.journal.md) | `action_create_vendor_bill` | UserError | self._build_no_journal_error_msg(self.env.company.display_name, ['purchase']) |
| [`account.journal`](entities/account.journal.md) | `action_create_vendor_bill` | UserError | You may only use samples in demo mode, try uploading one of your invoices instead. |
| [`account.journal`](entities/account.journal.md) | `_check_bank_account` | ValidationError | The bank account of a bank journal must belong to the same company (%s). |
| [`account.journal`](entities/account.journal.md) | `_check_bank_account` | ValidationError | The holder of a journal's bank account must be the company (%s). |
| [`account.journal`](entities/account.journal.md) | `_check_company_consistency` | UserError | You can't change the company of your journal since there are some journal entries linked to it. |
| [`account.journal`](entities/account.journal.md) | `_check_type_default_account_id_type` | ValidationError | The type of the journal's default credit/debit account shouldn't be 'receivable' or 'payable'. |
| [`account.journal`](entities/account.journal.md) | `_check_payment_method_line_ids_multiplicity` | ValidationError | Some payment methods supposed to be unique already exists somewhere else. (%s) |
| [`account.journal`](entities/account.journal.md) | `_check_payment_method_line_ids_multiplicity` | ValidationError | You can't have two payment method lines of the same payment type (%(payment_type)s) and with the same name (%(name)s) on a single journal. |
| [`account.journal`](entities/account.journal.md) | `_check_auto_post_draft_entries` | ValidationError | You can not archive a journal containing draft journal entries.  To proceed: 1/ go to Accounting > Accounting > Journal Entries 2/ filter on this journal and on 'Unposted' entries 3/ select them all and post or delete them through the action menu |
| [`account.journal`](entities/account.journal.md) | `copy_data` | UserError | Could not compute any code for the copy automatically. Please create it manually. |
| [`account.journal`](entities/account.journal.md) | `write` | UserError | You cannot modify the field %s of a journal that already has accounting entries. |
| [`account.journal`](entities/account.journal.md) | `write` | UserError | The partners of the journal's company and the related bank account mismatch. |
| [`account.journal`](entities/account.journal.md) | `_fill_missing_values` | UserError | Cannot generate an unused journal code. Please change the name for journal %s. |
| [`account.journal`](entities/account.journal.md) | `_create_document_from_attachment` | UserError | No attachment was provided |
| [`account.journal`](entities/account.journal.md) | `_create_document_from_attachment` | UserError | self.env['account.journal']._build_no_journal_error_msg(self.env.company.display_name, [journal_type]) |
| [`account.journal`](entities/account.journal.md) | `_create_document_from_attachment` | UserError | The journal in which to upload the invoice is not specified. |
| [`account.journal`](entities/account.journal.md) | `_inverse_check_next_number` | ValidationError | Next Check Number should only contains numbers. |
| [`account.journal`](entities/account.journal.md) | `_inverse_check_next_number` | ValidationError | The last check number was %s. In order to avoid a check being rejected by the bank, you can only use a greater number. |
| [`account.journal`](entities/account.journal.md) | `_inverse_check_next_number` | ValidationError | The check number you entered (%(num)s) exceeds the maximum allowed value of %(max)d. Please enter a smaller number. |
| [`account.journal`](entities/account.journal.md) | `write` | UserError | Cannot deactivate (%s) on this journal because not all documents are synchronized |
| [`account.journal`](entities/account.journal.md) | `_unlink_except_linked_to_payment_provider` | UserError | You must first deactivate a payment provider before deleting its journal. Linked providers: %s |
| [`account.journal`](entities/account.journal.md) | `_check_type_for_peppol_journal` | ValidationError | You can't change the type of a journal used for Peppol invoice reception toa type different than 'Purchase'. Please change the journal used for Peppol reception before changing the type of this journal. |
| [`account.journal`](entities/account.journal.md) | `_check_type` | ValidationError | This journal is associated with a payment method. You cannot modify its type |
| [`account.journal`](entities/account.journal.md) | `_check_no_active_payments` | ValidationError | You can not archive this journal because it is set on the following payment method : %s. |
| [`account.journal`](entities/account.journal.md) | `check_use_document` | ValidationError | You can not modify the field "Use Documents?" if there are validated invoices in this journal! |
| [`account.journal`](entities/account.journal.md) | `_get_journal_letter` | RedirectWarning | msg |
| [`account.journal`](entities/account.journal.md) | `_check_afip_pos_system` | ValidationError | '\n'.join((_('The pos system %(system)s can not be used on a purchase journal (id %(id)s)', system=x.l10n_ar_afip_pos_system, id=x.id) for x in journals)) |
| [`account.journal`](entities/account.journal.md) | `_check_afip_pos_number` | ValidationError | Please define an ARCA POS number |
| [`account.journal`](entities/account.journal.md) | `_check_afip_pos_number` | ValidationError | Please define a valid ARCA POS number (5 digits max) |
| [`account.journal`](entities/account.journal.md) | `write` | UserError | You can not change %s journal's configuration if it already has validated invoices |
| [`account.journal`](entities/account.journal.md) | `_check_fik_creditor_number` | ValidationError | FIK Creditor Number must be exactly 8 digits. |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_api_onboard_sanity_checks` | UserError | Oops! The journal is stuck. Please submit the pending invoices to ZATCA and try again. |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_generate_csr` | UserError | Please set the following on %(company_name)s: %(fields)s |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_get_compliance_CSID` | UserError | Please check the details below and onboard the journal again: %s |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_get_production_CSID` | UserError | str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_get_production_CSID` | UserError | Could not obtain Production CSID: %s |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_get_production_CSID` | UserError | The Journal is valid until (%s) and can only be renewed upon expiry. |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_run_compliance_checks` | UserError | Please change the (%s)'s country to Saudi Arabia and try again. |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_run_compliance_checks` | UserError | str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_run_compliance_checks` | UserError | Markup("<p class='mb-0'>%s</p>") % str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_run_compliance_checks` | UserError | Markup("<p class='mb-0'>%s</p>") % str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_api_get_compliance_CSID` | UserError | The OTP is invalid. Please try again. |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_api_get_compliance_CSID` | UserError | str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_api_get_pcsid` | UserError | str(ERROR_MESSAGE) |
| [`account.journal`](entities/account.journal.md) | `_l10n_sa_api_get_pcsid` | UserError | The Journal is not valid anymore. Please Renew it. |
| [`account.journal`](entities/account.journal.md) | `_check_l10n_se_invoice_ocr_length` | ValidationError | OCR Reference Number length need to be greater than 5. Please correct settings under invoice journal settings. |
| [`account.journal`](entities/account.journal.md) | `_check_api_key` | RedirectWarning | Please configure your Nilvera API key |
| [`account.lock_exception`](entities/account.lock_exception.md) | `create` | ValidationError | A single exception must change exactly one lock date field. |
| [`account.lock_exception`](entities/account.lock_exception.md) | `copy` | UserError | You cannot duplicate a Lock Date Exception. |
| [`account.lock_exception`](entities/account.lock_exception.md) | `action_revoke` | UserError | You cannot revoke Lock Date Exceptions. Ask someone with the 'Adviser' role. |
| [`account.merge.wizard`](entities/account.merge.wizard.md) | `default_get` | UserError | This can only be used on accounts. |
| [`account.merge.wizard`](entities/account.merge.wizard.md) | `default_get` | UserError | You must select at least 2 accounts. |
| [`account.merge.wizard`](entities/account.merge.wizard.md) | `_check_access_rights` | UserError | You do not have the right to perform this operation as you do not have access to the following companies: %s. |
| [`account.move`](entities/account.move.md) | `_search_default_journal` | UserError | error_msg |
| [`account.move`](entities/account.move.md) | `_inverse_company_id` | ValidationError | We can't leave this document without any company. Please select a company for this document. |
| [`account.move`](entities/account.move.md) | `_onchange_partner_id` | RedirectWarning | msg |
| [`account.move`](entities/account.move.md) | `_check_balanced` | UserError | error_msg |
| [`account.move`](entities/account.move.md) | `_check_balanced` | UserError | The entry is not balanced. |
| [`account.move`](entities/account.move.md) | `_check_fiscal_lock_dates` | UserError | message |
| [`account.move`](entities/account.move.md) | `_require_bill_date_for_autopost` | ValidationError | For this entry to be automatically posted, it required a bill date. |
| [`account.move`](entities/account.move.md) | `_check_journal_move_type` | ValidationError | Cannot create a purchase document in a non purchase journal |
| [`account.move`](entities/account.move.md) | `_check_journal_move_type` | ValidationError | Cannot create a sale document in a non sale journal |
| [`account.move`](entities/account.move.md) | `_validate_taxes_country` | ValidationError | This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration. |
| [`account.move`](entities/account.move.md) | `_validate_taxes_country` | ValidationError | This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration. |
| [`account.move`](entities/account.move.md) | `_check_invoice_currency_rate` | ValidationError | The currency rate must be strictly positive. |
| [`account.move`](entities/account.move.md) | `create` | UserError | You cannot create a move already in the posted state. Please create a draft move and post it after. |
| [`account.move`](entities/account.move.md) | `write` | AccessError | You don't have the access rights to perform this action. |
| [`account.move`](entities/account.move.md) | `write` | ValidationError | Validated entries can only be changed by your accountant. |
| [`account.move`](entities/account.move.md) | `write` | UserError | This document is protected by a hash. Therefore, you cannot edit the following fields: %s. |
| [`account.move`](entities/account.move.md) | `write` | UserError | You cannot edit the journal of an account move if it has been posted once, unless the name is removed or set to "/". This might create a gap in the sequence. |
| [`account.move`](entities/account.move.md) | `write` | UserError | You cannot edit the journal of an account move with a sequence number assigned, unless the name is removed or set to "/". This might create a gap in the sequence. |
| [`account.move`](entities/account.move.md) | `write` | UserError | You cannot modify the following readonly fields on the posted move %(move)s: %(fields)s |
| [`account.move`](entities/account.move.md) | `write` | UserError | The Journal Entry sequence is not conform to the current format. Only the Accountant can change it. |
| [`account.move`](entities/account.move.md) | `_unlink_forbid_parts_of_chain` | UserError | You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead. |
| [`account.move`](entities/account.move.md) | `_unlink_account_audit_trail_except_once_post` | UserError | To keep the restrictive audit trail, you can not delete journal entries once they have been posted. Instead, you can cancel the journal entry. |
| [`account.move`](entities/account.move.md) | `_get_invoice_computed_reference` | UserError | The combination of reference model and reference type on the journal is not implemented |
| [`account.move`](entities/account.move.md) | `_get_chains_to_hash` | UserError | An error occurred when computing the inalterability. All entries have to be reconciled. |
| [`account.move`](entities/account.move.md) | `_get_chains_to_hash` | UserError | This move could not be locked either because some move with the same sequence prefix has a higher number. You may need to resequence it. |
| [`account.move`](entities/account.move.md) | `_get_chains_to_hash` | UserError | An error occurred when computing the inalterability. A gap has been detected in the sequence. |
| [`account.move`](entities/account.move.md) | `_post` | AccessError | You don't have the access rights to post an invoice. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | msg |
| [`account.move`](entities/account.move.md) | `_post` | UserError | You cannot post an entry with an archived analytic account: %s |
| [`account.move`](entities/account.move.md) | `_post` | RedirectWarning | The company bank account (%(account_number)s) linked to this invoice is not trusted. Go to the Bank Settings, double-check that it is yours or correct the number, and click on Send Money to trust it. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | The bank account of your company is not trusted. Please ask an admin or someone with approval rights to check it. |
| [`account.move`](entities/account.move.md) | `action_switch_move_type` | ValidationError | You cannot switch the type of a document with an existing sequence number. |
| [`account.move`](entities/account.move.md) | `action_switch_move_type` | ValidationError | This action isn't available for this document. |
| [`account.move`](entities/account.move.md) | `action_register_payment` | UserError | You can only register payment for posted journal entries. |
| [`account.move`](entities/account.move.md) | `action_force_register_payment` | UserError | You cannot register payments for miscellaneous entries. |
| [`account.move`](entities/account.move.md) | `action_force_register_payment` | UserError | You cannot register payments for blocked invoices. |
| [`account.move`](entities/account.move.md) | `action_validate_moves_with_confirmation` | UserError | There are no journal items in the draft state to post. |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | Only posted/cancelled journal entries can be reset to draft. |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | You can't reset to draft those journal entries. You need to request a cancellation instead. |
| [`account.move`](entities/account.move.md) | `_check_draftable` | UserError | You cannot reset to draft an exchange difference journal entry. |
| [`account.move`](entities/account.move.md) | `_check_draftable` | UserError | You cannot reset to draft a tax cash basis journal entry. |
| [`account.move`](entities/account.move.md) | `_check_draftable` | UserError | You cannot reset to draft a locked journal entry. |
| [`account.move`](entities/account.move.md) | `button_request_cancel` | UserError | You can only request a cancellation for invoice sent to the government. |
| [`account.move`](entities/account.move.md) | `button_cancel` | UserError | Only draft journal entries can be cancelled. |
| [`account.move`](entities/account.move.md) | `action_toggle_block_payment` | UserError | You can't block a paid invoice. |
| [`account.move`](entities/account.move.md) | `_generate_qr_code` | UserError | error_msg |
| [`account.move`](entities/account.move.md) | `_get_available_invoice_template_pdf_report_ids` | UserError | There is no template that applies to invoices. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | Invalid invoice configuration:  %s |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | You can't edit the following journal entry %s because an electronic document has already been sent. Please use the 'Request EDI Cancellation' button instead. |
| [`account.move`](entities/account.move.md) | `_ungroup_lines` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_ungroup_lines` | UserError | Cannot decode origin file, try by importing it again |
| [`account.move`](entities/account.move.md) | `_ungroup_lines` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_group_lines_by_tax` | UserError | You can only group lines of an invoice |
| [`account.move`](entities/account.move.md) | `_check_move_for_group_ungroup_lines_by_tax` | UserError | You can only (un)group lines of a draft invoice |
| [`account.move`](entities/account.move.md) | `action_cancel_peppol_documents` | UserError | Cannot cancel an entry that has already been sent to PEPPOL |
| [`account.move`](entities/account.move.md) | `_check_expense_ids` | ValidationError | Each expense paid by the company must have a distinct and dedicated journal entry. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | We do not accept the usage of document types on receipts yet. |
| [`account.move`](entities/account.move.md) | `_check_l10n_latam_documents` | ValidationError | The journal require a document type but not document type has been selected on invoices %s. |
| [`account.move`](entities/account.move.md) | `_check_l10n_latam_documents` | ValidationError | Please set the document number on the following invoices %s. |
| [`account.move`](entities/account.move.md) | `_check_invoice_type_document_type` | ValidationError | You can not use a %s document type with a refund invoice |
| [`account.move`](entities/account.move.md) | `_check_invoice_type_document_type` | ValidationError | You can not use a %s document type with a invoice |
| [`account.move`](entities/account.move.md) | `_check_moves_use_documents` | ValidationError | The selected Journal can't be used in this transaction, please select one that doesn't use documents as these are just for Invoices. |
| [`account.move`](entities/account.move.md) | `_check_argentinean_invoice_taxes` | UserError | There should be a single tax from the “VAT“ tax group per line, but this is not the case for line “%s”. Please add a tax to this line or check the tax configuration's advanced options for the corresponding field “Tax Group”. |
| [`account.move`](entities/account.move.md) | `_check_argentinean_invoice_taxes` | UserError | On invoice id “%s” you must use VAT Not Applicable on every line. |
| [`account.move`](entities/account.move.md) | `_check_argentinean_invoice_taxes` | UserError | On invoice id “%s” you must use a VAT tax that is not VAT Not Applicable |
| [`account.move`](entities/account.move.md) | `_onchange_partner_journal` | RedirectWarning | msg |
| [`account.move`](entities/account.move.md) | `_inverse_l10n_latam_document_number` | UserError | The document number can not be changed for this journal, you can only modify the POS number if there is not posted (or posted before) invoices |
| [`account.move`](entities/account.move.md) | `l10n_ch_action_print_qr` | UserError | Only customers invoices can be QR-printed. |
| [`account.move`](entities/account.move.md) | `_check_l10n_latam_document_number_is_numeric` | ValidationError | The DTE document number (folio) must contain only digits. |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | Tax payer type and vat number are mandatory for this type of document. Please set the current tax payer type of this customer |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | The DIN document is intended to be used only with RUT 60805000-0 (Tesorería General de La República) |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | The tax payer type of this supplier is incorrect for the selected type of document. |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | You need a journal without the use of documents for foreign suppliers |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | Document types for foreign customers must be export type (codes 110, 111 or 112) or you should define the customer as an end consumer and use receipts (codes 39 or 41) |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | Tax payer type and vat number are mandatory for this type of document. Please set the current tax payer type of this supplier |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | The tax payer type of this supplier is not entitled to deliver fees documents |
| [`account.move`](entities/account.move.md) | `_check_document_types_post` | ValidationError | The tax payer type of this supplier is not entitled to deliver imports documents |
| [`account.move`](entities/account.move.md) | `_check_fapiao` | ValidationError | Fapiao number is an 8-digit number. Please enter a correct one. |
| [`account.move`](entities/account.move.md) | `_get_invoice_reference_dk_fik` | ValidationError | FIK %(prefix)s reference cannot be generated: invoice number '%(invoice)s' has more than %(max_digits)s digits. |
| [`account.move`](entities/account.move.md) | `action_cancel_nemhandel_documents` | UserError | Cannot cancel an entry that has already been sent to Nemhandel |
| [`account.move`](entities/account.move.md) | `action_post_sign_invoices` | UserError | Please only sign invoices from one company at a time |
| [`account.move`](entities/account.move.md) | `action_post_sign_invoices` | ValidationError | Please setup a personal drive for company %s |
| [`account.move`](entities/account.move.md) | `action_post_sign_invoices` | ValidationError | Please setup the certificate on the thumb drive menu |
| [`account.move`](entities/account.move.md) | `_l10n_es_edi_facturae_get_corrective_data` | UserError | The credit note/refund appears to have been issued manually. For the purpose of generating a Facturae document, it's necessary that the credit note/refund is created directly from the associated invoice/bill. |
| [`account.move`](entities/account.move.md) | `_l10n_es_edi_facturae_export_facturae` | UserError | The company needs a set tax identification number or VAT number |
| [`account.move`](entities/account.move.md) | `_l10n_es_edi_facturae_export_facturae` | UserError | The partner needs a set tax identification number or VAT number |
| [`account.move`](entities/account.move.md) | `_l10n_es_edi_facturae_export_facturae` | UserError | The partner needs a set country |
| [`account.move`](entities/account.move.md) | `_l10n_es_facturae_sign_xml` | UserError | No valid certificate found |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | You cannot reset to draft an entry that has been posted to TicketBAI's chain |
| [`account.move`](entities/account.move.md) | `_l10n_es_tbai_unlink_except_in_chain` | UserError | You cannot delete a move that has a TicketBAI chain id. |
| [`account.move`](entities/account.move.md) | `_l10n_es_tbai_lock_move` | UserError | Cannot send this entry as it is already being processed. |
| [`account.move`](entities/account.move.md) | `l10n_es_tbai_resend_bill` | UserError | error |
| [`account.move`](entities/account.move.md) | `l10n_es_tbai_send_bill` | UserError | error |
| [`account.move`](entities/account.move.md) | `l10n_es_tbai_cancel` | UserError | You cannot reset to draft a locked journal entry. |
| [`account.move`](entities/account.move.md) | `l10n_es_tbai_cancel` | UserError | error |
| [`account.move`](entities/account.move.md) | `l10n_es_tbai_cancel` | UserError | edi_document.response_message |
| [`account.move`](entities/account.move.md) | `number2numeric` | UserError | Invoice number must contain numeric characters |
| [`account.move`](entities/account.move.md) | `action_pdp_open_response_wizard` | UserError | Cannot send response for any of the journal entries. |
| [`account.move`](entities/account.move.md) | `l10n_gr_edi_try_send_expense_classification` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `l10n_gr_edi_try_send_expense_classification` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_l10n_gr_edi_try_send_batch` | UserError | You should use Send & Print wizard for sending customer invoices to myDATA. |
| [`account.move`](entities/account.move.md) | `_l10n_gr_edi_try_send_batch` | UserError | Some of the selected moves does not meet the requirements to be sent to myDATA. |
| [`account.move`](entities/account.move.md) | `_check_draftable` | UserError | You cannot reset this invoice to draft. |
| [`account.move`](entities/account.move.md) | `_check_l10n_hr_process_type` | ValidationError | Business Process Type P9 can only be used with credit notes. |
| [`account.move`](entities/account.move.md) | `_check_l10n_hr_process_type` | ValidationError | Credit notes must use Business Process Type P9 or P10. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | This vendor bill is already rejected according to the Tax Authority. |
| [`account.move`](entities/account.move.md) | `_check_posted_if_active` | ValidationError | Cannot reset to draft or cancel invoice %s because an electronic document was already sent to NAV! |
| [`account.move`](entities/account.move.md) | `l10n_hu_edi_button_update_status` | UserError | error_text |
| [`account.move`](entities/account.move.md) | `_l10n_hu_edi_acquire_lock` | UserError | Could not acquire lock on invoices - is another user performing operations on them? |
| [`account.move`](entities/account.move.md) | `_l10n_hu_edi_get_invoice_values` | UserError | Please create a sales tax with type ATK (outside the scope of the VAT Act). |
| [`account.move`](entities/account.move.md) | `download_efaktur` | UserError | You are not allowed to generate e-Faktur document from invoices coming from different companies |
| [`account.move`](entities/account.move.md) | `download_efaktur` | ValidationError | '\n - '.join(err_messages) |
| [`account.move`](entities/account.move.md) | `download_efaktur` | RedirectWarning | msg |
| [`account.move`](entities/account.move.md) | `_post` | UserError | Please set a valid TIN Number on the Place of Supply %s |
| [`account.move`](entities/account.move.md) | `_post` | RedirectWarning | msg |
| [`account.move`](entities/account.move.md) | `_post` | ValidationError | Partner %(partner_name)s (%(partner_id)s) GSTIN is required under GST Treatment %(name)s |
| [`account.move`](entities/account.move.md) | `_l10n_in_lock_invoice` | UserError | This electronic document is being processed already. |
| [`account.move`](entities/account.move.md) | `action_l10n_in_ewaybill_create` | UserError | Ewaybill already created for this move. |
| [`account.move`](entities/account.move.md) | `action_check_l10n_it_edi` | UserError | This move is not waiting for updates from the SdI. |
| [`account.move`](entities/account.move.md) | `_l10n_it_edi_send` | UserError | This document is being sent by another process already. |
| [`account.move`](entities/account.move.md) | `_l10n_it_edi_update_send_state` | UserError | An error occurred while downloading updates from the Proxy Server: (%(code)s) %(message)s |
| [`account.move`](entities/account.move.md) | `_l10n_it_edi_update_send_state` | UserError | An error occurred while downloading updates from the Proxy Server: (%(code)s) %(message)s |
| [`account.move`](entities/account.move.md) | `_check_l10n_it_edi_doi_id` | UserError | '\n'.join(validity_errors) |
| [`account.move`](entities/account.move.md) | `_post` | UserError | '\n'.join(errors) |
| [`account.move`](entities/account.move.md) | `download_l10n_jo_edi_computed_xml` | ValidationError | The following errors have to be fixed in order to create an XML: |
| [`account.move`](entities/account.move.md) | `l10n_ke_action_cu_post` | UserError | An OSCU has been initialized for this company. Please send the e-invoice via Send and Print -> Send to eTIMS instead. |
| [`account.move`](entities/account.move.md) | `l10n_ke_action_cu_post` | UserError | error_msg |
| [`account.move`](entities/account.move.md) | `_constrains_l10n_lk_sequence_length` | UserError | Invoice number exceeds %(max)d characters: %(name)s |
| [`account.move`](entities/account.move.md) | `_get_last_sequence` | ValidationError | %(field_name)s is not a stored field |
| [`account.move`](entities/account.move.md) | `action_invoice_sent` | UserError | You cannot send invoices that are currently being validated. Please wait for the validation to complete. |
| [`account.move`](entities/account.move.md) | `action_open_l10n_ph_2307_wizard` | UserError | Only Vendor Bills are available. |
| [`account.move`](entities/account.move.md) | `action_l10n_pl_edi_get_invoice_UPO` | UserError | This invoice does not have a KSeF Invoice Reference Number. It may not have been sent yet. |
| [`account.move`](entities/account.move.md) | `action_l10n_pl_edi_get_invoice_UPO` | UserError | You can only download a UPO for an 'Accepted' invoice. Please update the status first. |
| [`account.move`](entities/account.move.md) | `action_l10n_pl_edi_get_invoice_UPO` | UserError | The KSeF service returned empty UPO content. |
| [`account.move`](entities/account.move.md) | `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Purchase tax corresponding to '%s' required for the KSeF import was not found in the system. |
| [`account.move`](entities/account.move.md) | `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Currency '%s' from the KSeF bill was not found. |
| [`account.move`](entities/account.move.md) | `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Purchase tax corresponding to '%s' required for the KSeF import was not found in the system. |
| [`account.move`](entities/account.move.md) | `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Tax corresponding to '%s' required to derive the net unit price from gross price during KSeF import could not be interpreted. |
| [`account.move`](entities/account.move.md) | `_handle_download_bills_from_ksef_error` | UserError | error.get('message') |
| [`account.move`](entities/account.move.md) | `_l10n_ro_edi_fetch_invoices` | UserError | result['error'] |
| [`account.move`](entities/account.move.md) | `_get_normalized_l10n_sa_confirmation_datetime` | UserError | Please set the Invoice Date to be either less than or equal to today as per the Asia/Riyadh time zone, since ZATCA does not allow future-dated invoicing. |
| [`account.move`](entities/account.move.md) | `_prevent_zatca_rejected_invoice_deletion` | UserError | The Invoice(s) are linked to a validated EDI document and cannot be modified according to ZATCA rules |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | The Invoice(s) are linked to a validated EDI document and cannot be modified according to ZATCA rules |
| [`account.move`](entities/account.move.md) | `_get_invoice_reference_se_ocr4` | UserError | OCR Reference Number length is greater than allowed. Allowed length in invoice journal setting is %s. |
| [`account.move`](entities/account.move.md) | `_l10n_se_check_payment_reference` | ValidationError | Vendor require OCR Number as payment reference. Payment reference isn't a valid OCR Number. |
| [`account.move`](entities/account.move.md) | `button_draft` | UserError | You cannot reset to draft an entry that has been sent to Nilvera. |
| [`account.move`](entities/account.move.md) | `_post` | UserError | To preserve accounting integrity and comply with legal requirements, invoices cannot be reused once an error occurs. Please create a new invoice to continue. |
| [`account.move`](entities/account.move.md) | `_l10n_tr_nilvera_submit_document` | UserError | Oops, seems like you're unauthorised to do this. Try another API key with more rights or contact Nilvera. |
| [`account.move`](entities/account.move.md) | `_l10n_tr_nilvera_submit_document` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_l10n_tr_nilvera_submit_document` | UserError | Server error from Nilvera, please try again later. |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_check_before_generate_invoice_json` | UserError | 'Error:\n' + '\n'.join(errors) |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | You cannot issue an allowance for invoice %(invoice_number)s as it was not sent to Ecpay. |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | Customer email is needed for notification |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | Customer %(notify_way)s is needed for notification |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_update_ecpay_invoice_info` | UserError | The invoice: %(invoice_name)s has no related number |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_run_invoice_invalid` | UserError | You cannot invalidate an invoice that was not sent to Ecpay. |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_run_invoice_invalid` | UserError | The invoice: %(invoice_id)s has already been invalidated |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_run_invoice_invalid` | UserError | Fail to invalidate invoice. Error message: %(error_message)s |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_issue_allowance` | UserError | Fail to issue allowance for ECpay invoice. Error message: %(error_message)s |
| [`account.move`](entities/account.move.md) | `_l10n_tw_edi_print_invoice` | UserError | You cannot print an invoice that was not sent to Ecpay, without the print flag, or that is invalid. |
| [`account.move`](entities/account.move.md) | `_l10n_vn_edi_fetch_invoice_files` | UserError | Please send the invoice to SInvoice before fetching the tax invoice files. |
| [`account.move`](entities/account.move.md) | `action_l10n_vn_edi_update_payment_status` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `action_l10n_vn_edi_update_payment_status` | UserError | error |
| [`account.move`](entities/account.move.md) | `action_l10n_vn_edi_update_payment_status` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_l10n_vn_edi_cancel_invoice` | UserError | error |
| [`account.move`](entities/account.move.md) | `_l10n_vn_edi_cancel_invoice` | UserError | error_message |
| [`account.move`](entities/account.move.md) | `_check_move_for_group_ungroup_lines_by_tax` | UserError | You can only (un)group lines of an invoice not linked to a purchase order |
| [`account.move.line`](entities/account.move.line.md) | `_check_constrains_account_id_journal_id` | UserError | The account %(name)s (%(code)s) is archived. |
| [`account.move.line`](entities/account.move.line.md) | `_check_constrains_account_id_journal_id` | UserError | The account selected on your journal entry forces to provide a secondary currency. You should remove the secondary currency on the account. |
| [`account.move.line`](entities/account.move.line.md) | `_check_off_balance` | UserError | If you want to use "Off-Balance Sheet" accounts, all the accounts of the journal entry must be of this type |
| [`account.move.line`](entities/account.move.line.md) | `_check_off_balance` | UserError | You cannot use taxes on lines with an Off-Balance account |
| [`account.move.line`](entities/account.move.line.md) | `_check_off_balance` | UserError | Lines from "Off-Balance Sheet" accounts cannot be reconciled |
| [`account.move.line`](entities/account.move.line.md) | `_check_payable_receivable` | UserError | Account %s is of payable type, but is used in a sale operation. |
| [`account.move.line`](entities/account.move.line.md) | `_check_payable_receivable` | UserError | Any journal item on a receivable account must have a due date and vice versa. |
| [`account.move.line`](entities/account.move.line.md) | `_check_payable_receivable` | UserError | Account %s is of receivable type, but is used in a purchase operation. |
| [`account.move.line`](entities/account.move.line.md) | `_check_payable_receivable` | UserError | Any journal item on a payable account must have a due date and vice versa. |
| [`account.move.line`](entities/account.move.line.md) | `_check_tax_lock_date` | UserError | The operation is refused as it would impact an already issued tax statement. Please change the journal entry date or the following lock dates to proceed: %(lock_date_info)s. |
| [`account.move.line`](entities/account.move.line.md) | `_check_reconciliation` | UserError | You cannot do this modification on a reconciled journal entry. You can just change some non legal fields or you must unreconcile first. Journal Entry (id): %(entry)s (%(id)s) |
| [`account.move.line`](entities/account.move.line.md) | `_check_caba_non_caba_shared_tags` | ValidationError | Taxes exigible on payment and on invoice cannot be mixed on the same journal item if they share some tag. |
| [`account.move.line`](entities/account.move.line.md) | `_constrains_matching_number` | ValidationError | A temporary number can not be used in a real matching |
| [`account.move.line`](entities/account.move.line.md) | `_constrains_deductible_amount` | ValidationError | Only vendor bills allow for deductibility of product/services. |
| [`account.move.line`](entities/account.move.line.md) | `_constrains_deductible_amount` | ValidationError | The deductibility must be a value between 0 and 100. |
| [`account.move.line`](entities/account.move.line.md) | `write` | UserError | You cannot use an archived account. |
| [`account.move.line`](entities/account.move.line.md) | `write` | UserError | You cannot edit the following fields: %(fields)s. The following entries are already hashed: %(entries)s |
| [`account.move.line`](entities/account.move.line.md) | `write` | UserError | You cannot modify the taxes related to a posted journal item, you should reset the journal entry to draft to do so. |
| [`account.move.line`](entities/account.move.line.md) | `_unlink_except_posted` | UserError | You can't delete a posted journal item. Don’t play games with your accounting records; reset the journal entry to draft before deleting it. |
| [`account.move.line`](entities/account.move.line.md) | `_prevent_automatic_line_deletion` | ValidationError | You cannot delete a tax line as it would impact the tax report |
| [`account.move.line`](entities/account.move.line.md) | `_prevent_automatic_line_deletion` | ValidationError | You cannot delete a payable/receivable line as it would not be consistent with the payment terms |
| [`account.move.line`](entities/account.move.line.md) | `_except_hashed_entry_lines` | UserError | You cannot delete journal items belonging to a locked journal entry. |
| [`account.move.line`](entities/account.move.line.md) | `_check_amls_exigibility_for_reconciliation` | UserError | You are trying to reconcile some entries that are already reconciled. |
| [`account.move.line`](entities/account.move.line.md) | `_check_amls_exigibility_for_reconciliation` | UserError | You can not reconcile cancelled entries. |
| [`account.move.line`](entities/account.move.line.md) | `_check_amls_exigibility_for_reconciliation` | UserError | Entries are not from the same account: %s |
| [`account.move.line`](entities/account.move.line.md) | `_check_amls_exigibility_for_reconciliation` | UserError | Entries don't belong to the same company: %s |
| [`account.move.line`](entities/account.move.line.md) | `_check_amls_exigibility_for_reconciliation` | UserError | Account %s does not allow reconciliation. First change the configuration of this account to allow it. |
| [`account.move.line`](entities/account.move.line.md) | `_create_exchange_difference_moves` | UserError | You have to configure the 'Exchange Gain or Loss Journal' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |
| [`account.move.line`](entities/account.move.line.md) | `_create_exchange_difference_moves` | UserError | You should configure the 'Loss Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |
| [`account.move.line`](entities/account.move.line.md) | `_create_exchange_difference_moves` | UserError | You should configure the 'Gain Exchange Rate Account' in your company settings, to manage automatically the booking of accounting entries related to differences between exchange rates. |
| [`account.move.line`](entities/account.move.line.md) | `_validate_analytic_distribution` | ValidationError | msg |
| [`account.move.line`](entities/account.move.line.md) | `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced must be validated before registering expenses. |
| [`account.move.line`](entities/account.move.line.md) | `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced is cancelled. You cannot register an expense on a cancelled Sales Order. |
| [`account.move.line`](entities/account.move.line.md) | `_sale_create_reinvoice_sale_line` | UserError | The Sales Order %(order)s to be reinvoiced is currently locked. You cannot register an expense on a locked Sales Order. |
| [`account.move.line`](entities/account.move.line.md) | `_l10n_id_coretax_build_invoice_line_vals` | ValidationError | Price for line '%s' cannot be a negative amount. Please check again. |
| [`account.move.line`](entities/account.move.line.md) | `_check_l10n_tr_ctsp_number` | ValidationError | CTSP Number must be 12 digits or fewer. |
| [`account.move.reversal`](entities/account.move.reversal.md) | `_check_journal_type` | UserError | Journal should be the same type as the reversed entry. |
| [`account.move.reversal`](entities/account.move.reversal.md) | `default_get` | UserError | All selected moves for reversal must belong to the same company. |
| [`account.move.reversal`](entities/account.move.reversal.md) | `default_get` | UserError | To reverse a journal entry, it has to be posted first. |
| [`account.move.reversal`](entities/account.move.reversal.md) | `_compute_documents_info` | UserError | You can only reverse documents with legal invoicing documents from Latin America one at a time. Problematic documents: %s |
| [`account.move.reversal`](entities/account.move.reversal.md) | `_compute_l10n_es_tbai_is_required` | UserError | Reversals mixing invoices with and without TicketBAI are not allowed. |
| [`account.move.reversal`](entities/account.move.reversal.md) | `reverse_moves` | UserError | You cannot adjust/replace invoice %s, it has not been approved by the tax authorities. Please cancel/reverse it and create a new invoice instead. |
| [`account.move.send`](entities/account.move.send.md) | `_get_default_pdf_report_id` | UserError | There is no template that applies to this move type. |
| [`account.move.send`](entities/account.move.send.md) | `_raise_danger_alerts` | UserError | '\n'.join(danger_alert_messages) |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | UserError | next(iter(move_constraints.values()), None) |
| [`account.move.send`](entities/account.move.send.md) | `_check_invoice_report` | UserError | The sending of invoices is not set up properly, make sure the report used is set for invoices. |
| [`account.move.send`](entities/account.move.send.md) | `_prepare_invoice_pdf_report` | ValidationError | Cannot identify the invoices in the generated PDF: %s |
| [`account.move.send`](entities/account.move.send.md) | `_hook_if_errors` | UserError | self._format_error_text(error) |
| [`account.move.send`](entities/account.move.send.md) | `_hook_if_errors` | RedirectWarning | '\n'.join(error['errors']) |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | UserError | Operator label is required for sending invoices in Croatia. |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | UserError | Operator OIB is required for sending invoices in Croatia. |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | UserError | KPD categories must be defined on every invoice line for any Business Process Type other than P4. |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | UserError | Name of custom business process is required for Business Process Type P99. |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | ValidationError | For Croatia, all VAT taxes on an invoice should either be cash basis or not. |
| [`account.move.send`](entities/account.move.send.md) | `_check_move_constraints` | ValidationError | For Croatia, Legal Notes should be provided for all cash basis taxes. |
| [`account.move.send`](entities/account.move.send.md) | `_call_web_service_after_invoice_pdf_render` | UserError | Failed to send invoice via MojEracun: check configuration. |
| [`account.move.send.batch.wizard`](entities/account.move.send.batch.wizard.md) | `action_send_and_print` | UserError | Batch invoice sending is unavailable. Please, contact your system administrator to activate the cron to enable batch sending of invoices. |
| [`account.move.send.batch.wizard`](entities/account.move.send.batch.wizard.md) | `action_send_and_print` | RedirectWarning | Batch invoice sending is unavailable. Please, activate the cron to enable batch sending of invoices. |
| [`account.move.send.wizard`](entities/account.move.send.wizard.md) | `create_mail_template` | UserError | Template creation from composer requires a valid model. |
| [`account.move.send.wizard`](entities/account.move.send.wizard.md) | `action_send_and_print` | UserError | Partner doesn't have a valid Peppol configuration. |
| [`account.move.send.wizard`](entities/account.move.send.wizard.md) | `action_send_and_print` | UserError | Partner doesn't have a valid Nemhandel configuration. |
| [`account.move.send.wizard`](entities/account.move.send.wizard.md) | `action_send_and_print` | UserError | self._get_l10n_ke_edi_tremol_warning_message(warning_moves) |
| [`account.partial.reconcile`](entities/account.partial.reconcile.md) | `_check_required_computed_currencies` | ValidationError | Missing foreign currencies on partials having ids: %s |
| [`account.partial.reconcile`](entities/account.partial.reconcile.md) | `_collect_tax_cash_basis_values` | UserError | There is no tax cash basis journal defined for the '%s' company. Configure it in Accounting/Configuration/Settings |
| [`account.payment`](entities/account.payment.md) | `_prepare_move_lines_per_type` | UserError | You can't create a new payment without an outstanding payments/receipts account set either on the company or the %(payment_method)s payment method in the %(journal)s journal. |
| [`account.payment`](entities/account.payment.md) | `_check_payment_method_line_id` | ValidationError | Please define a payment method line on your payment. |
| [`account.payment`](entities/account.payment.md) | `_check_payment_method_line_id` | ValidationError | The selected payment method is not available for this payment, please select the payment method again. |
| [`account.payment`](entities/account.payment.md) | `_check_move_id` | ValidationError | A payment with an outstanding account cannot be confirmed without having a journal entry. |
| [`account.payment`](entities/account.payment.md) | `_get_outstanding_account` | UserError | No outstanding account could be found to make the payment |
| [`account.payment`](entities/account.payment.md) | `_synchronize_to_moves` | UserError | You cannot change the amount of a payment with multiple liquidity lines. |
| [`account.payment`](entities/account.payment.md) | `action_post` | UserError | To record payments with %(method_name)s, the recipient bank account must be manually validated. You should go on the partner bank account of %(partner)s in order to validate it. |
| [`account.payment`](entities/account.payment.md) | `_constrains_check_number` | ValidationError | Check numbers can only consist of digits |
| [`account.payment`](entities/account.payment.md) | `_constrains_check_number_unique` | ValidationError | The following numbers are already used: %s |
| [`account.payment`](entities/account.payment.md) | `print_checks` | UserError | Payments to print as a checks must have 'Check' selected as payment method and not have already been reconciled |
| [`account.payment`](entities/account.payment.md) | `print_checks` | UserError | In order to print multiple checks at once, they must belong to the same bank journal. |
| [`account.payment`](entities/account.payment.md) | `do_print_checks` | RedirectWarning | msg |
| [`account.payment`](entities/account.payment.md) | `do_print_checks` | RedirectWarning | msg |
| [`account.payment`](entities/account.payment.md) | `_create_payment_transaction` | ValidationError | A payment transaction with reference %s already exists. |
| [`account.payment`](entities/account.payment.md) | `_create_payment_transaction` | ValidationError | A token is required to create a new payment transaction. |
| [`account.payment`](entities/account.payment.md) | `write` | UserError | You cannot do this modification since the payment is linked to an expense. |
| [`account.payment`](entities/account.payment.md) | `_check_move_id` | ValidationError | A payment with any Third Party Check or Own Check payment methods needs an outstanding account |
| [`account.payment`](entities/account.payment.md) | `action_post` | ValidationError | error_msg |
| [`account.payment`](entities/account.payment.md) | `_get_reconciled_checks_error` | UserError | You can't cancel or re-open a payment with checks if some check has been debited or been voided. Checks: %s |
| [`account.payment`](entities/account.payment.md) | `action_open_l10n_ph_2307_wizard` | UserError | Only Outbound Payment is available. |
| [`account.payment.method.line`](entities/account.payment.method.line.md) | `_unlink_except_active_provider` | UserError | You can't delete a payment method that is linked to a provider in the enabled or test state. Linked providers(s): %s |
| [`account.payment.register`](entities/account.payment.register.md) | `_compute_batches` | UserError | You can't create payments for entries belonging to different companies. |
| [`account.payment.register`](entities/account.payment.register.md) | `_compute_batches` | UserError | You can't open the register payment wizard without at least one receivable/payable line. |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | There's nothing left to pay for the selected journal items, so no payment registration is necessary. You've got your finances under control like a boss! |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | You can't create payments for entries belonging to different companies. |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | You can't create payments for entries belonging to different branches without access to parent company. |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | You can't register payments for both inbound and outbound moves at the same time. |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | You cannot register payments for blocked invoices. |
| [`account.payment.register`](entities/account.payment.register.md) | `default_get` | UserError | The register payment wizard should only be called on account.move or account.move.line records. |
| [`account.payment.register`](entities/account.payment.register.md) | `_create_payments` | UserError | To record payments with %(payment_method)s, the recipient bank account must be manually validated. You should go on the partner bank account in order to validate it. |
| [`account.payment.register`](entities/account.payment.register.md) | `_create_payment_vals_from_wizard` | UserError | The withholding net amount cannot be negative. |
| [`account.payment.register`](entities/account.payment.register.md) | `action_create_payments` | ValidationError | You can't mix checks of different currencies in one payment, and you can't change the payment's currency if checks are already created in that currency. Please create separate payments for each currency. |
| [`account.payment.register`](entities/account.payment.register.md) | `_create_payment_vals_from_wizard` | UserError | Please enter withholding number for tax %s |
| [`account.payment.register`](entities/account.payment.register.md) | `action_create_payments` | ValidationError | A payment cannot have withholding if the payment method has no outstanding accounts |
| [`account.payment.term`](entities/account.payment.term.md) | `_check_lines` | ValidationError | The Payment Term must have at least one percent line and the sum of the percent must be 100%. |
| [`account.payment.term`](entities/account.payment.term.md) | `_check_lines` | ValidationError | The Early Payment Discount functionality can only be used with payment terms using a single 100% line. |
| [`account.payment.term`](entities/account.payment.term.md) | `_check_lines` | ValidationError | The Early Payment Discount must be strictly positive. |
| [`account.payment.term`](entities/account.payment.term.md) | `_check_lines` | ValidationError | The Early Payment Discount days must be strictly positive. |
| [`account.payment.term`](entities/account.payment.term.md) | `_unlink_except_referenced_terms` | UserError | Uh-oh! Those payment terms are quite popular and can't be deleted since there are still some records referencing them. How about archiving them instead? |
| [`account.payment.term.line`](entities/account.payment.term.line.md) | `_check_valid_char_value` | ValidationError | The days added must be a number and has to be between 0 and 31. |
| [`account.payment.term.line`](entities/account.payment.term.line.md) | `_check_valid_char_value` | ValidationError | The days added must be between 0 and 31. |
| [`account.payment.term.line`](entities/account.payment.term.line.md) | `_check_percent` | ValidationError | Percentages on the Payment Terms lines must be between 0 and 100. |
| [`account.peppol.rejection.wizard`](entities/account.peppol.rejection.wizard.md) | `button_send` | ValidationError | At least one reason must be given when rejecting a Peppol invoice. |
| [`account.reconcile.model`](entities/account.reconcile.model.md) | `_check_match_label_param` | UserError | The regex is not valid |
| [`account.reconcile.model.line`](entities/account.reconcile.model.line.md) | `_validate_amount` | UserError | The amount is not a number |
| [`account.reconcile.model.line`](entities/account.reconcile.model.line.md) | `_validate_amount` | UserError | Statement line percentage can't be 0 |
| [`account.reconcile.model.line`](entities/account.reconcile.model.line.md) | `_validate_amount` | UserError | Balance percentage can't be 0 |
| [`account.reconcile.model.line`](entities/account.reconcile.model.line.md) | `_validate_amount` | UserError | The regex is not valid |
| [`account.report`](entities/account.report.md) | `_validate_root_report_id` | ValidationError | Only a report without a root report of its own can be selected as root report. |
| [`account.report`](entities/account.report.md) | `_validate_parent_sequence` | ValidationError | Line "%(line)s" defines line "%(parent_line)s" as its parent, but appears before it in the report. The parent must always come first. |
| [`account.report`](entities/account.report.md) | `_validate_section_report_ids` | ValidationError | The sections defined on a report cannot have sections themselves. |
| [`account.report`](entities/account.report.md) | `_validate_availability_condition` | ValidationError | The Availability is set to 'Country Matches' but the field Country is not set. |
| [`account.report`](entities/account.report.md) | `_unlink_if_no_variant` | UserError | You can't delete a report that has variants. |
| [`account.report.expression`](entities/account.report.expression.md) | `_check_carryover_target` | UserError | You cannot use the field carryover_target in an expression that does not have the label starting with _carryover_ |
| [`account.report.expression`](entities/account.report.expression.md) | `_check_carryover_target` | UserError | When targeting an expression for carryover, the label of that expression must start with _applied_carryover_ |
| [`account.report.expression`](entities/account.report.expression.md) | `_check_formula` | ValidationError | Invalid formula for expression '%(label)s' of line '%(line)s': %(formula)s |
| [`account.report.expression`](entities/account.report.expression.md) | `_validate_engine` | ValidationError | Groupby feature isn't supported by '%(engine)s' engine. Please remove the groupby value on '%(report_line)s' |
| [`account.report.expression`](entities/account.report.expression.md) | `_expand_aggregations` | UserError | In report '%(report_name)s', on line '%(line_name)s', with label '%(label)s', The format of the cross report expression is invalid.  Expected: cross_report(<report_id>\|<xml_id>)Example:  cross_report(my_module.my_report) or cross_report(123) |
| [`account.report.expression`](entities/account.report.expression.md) | `_expand_aggregations` | UserError | In report '%(report_name)s', on line '%(line_name)s', with label '%(label)s', Failed to parse the cross report id or xml_id. |
| [`account.report.expression`](entities/account.report.expression.md) | `_expand_aggregations` | UserError | You cannot use cross report on itself |
| [`account.report.expression`](entities/account.report.expression.md) | `_get_aggregation_terms_details` | UserError | Cannot get aggregation details from a line not using 'aggregation' engine |
| [`account.report.expression`](entities/account.report.expression.md) | `_get_carryover_target_expression` | UserError | Could not determine carryover target automatically for expression %s. |
| [`account.report.line`](entities/account.report.line.md) | `_validate_groupby_no_child` | ValidationError | A line cannot have both children and a groupby value (line '%s'). |
| [`account.report.line`](entities/account.report.line.md) | `_check_parent_line` | ValidationError | Line "%s" defines itself as its parent. |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | `default_get` | UserError | You can only resequence items from the same journal |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | `default_get` | UserError | The sequences of this journal are different for Invoices and Refunds but you selected some of both types. |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | `default_get` | UserError | The sequences of this journal are different for Payments and non-Payments but you selected some of both types. |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | `resequence` | UserError | You can not reorder sequence by date when the journal is locked with a hash. |
| [`account.resequence.wizard`](entities/account.resequence.wizard.md) | `resequence` | UserError | The following documents have already been sent and cannot be resequenced: %s |
| [`account.root`](entities/account.root.md) | `_search` | UserError | Filter on the Account or its Display Name instead |
| [`account.sale.closing`](entities/account.sale.closing.md) | `write` | UserError | Sale Closings are not meant to be written or deleted under any circumstances. |
| [`account.sale.closing`](entities/account.sale.closing.md) | `_unlink_never` | UserError | Sale Closings are not meant to be written or deleted under any circumstances. |
| [`account.secure.entries.wizard`](entities/account.secure.entries.wizard.md) | `action_secure_entries` | UserError | Set a date. The moves will be secured up to including this date. |
| [`account.tax`](entities/account.tax.md) | `_constrains_name` | ValidationError | Tax names must be unique! %(taxes)s |
| [`account.tax`](entities/account.tax.md) | `validate_tax_group_id` | ValidationError | The tax group must have the same country_id as the tax using it. |
| [`account.tax`](entities/account.tax.md) | `_constrains_cash_basis_transition_account` | ValidationError | The cash basis transition account needs to allow reconciliation. |
| [`account.tax`](entities/account.tax.md) | `_check_repartition_lines` | ValidationError | Invoice and credit note distribution should each contain exactly one line for the base. |
| [`account.tax`](entities/account.tax.md) | `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have the same number of lines. |
| [`account.tax`](entities/account.tax.md) | `_validate_repartition_lines` | ValidationError | Invoice and credit note repartition should have at least one tax repartition line. |
| [`account.tax`](entities/account.tax.md) | `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have a total factor (+) equals to 100. |
| [`account.tax`](entities/account.tax.md) | `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have a total factor (-) equals to 100. |
| [`account.tax`](entities/account.tax.md) | `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should match (same percentages, in the same order). |
| [`account.tax`](entities/account.tax.md) | `_check_children_scope` | ValidationError | Recursion found for tax “%s”. |
| [`account.tax`](entities/account.tax.md) | `_check_children_scope` | ValidationError | The application scope of taxes in a group must be either the same as the group or left empty. |
| [`account.tax`](entities/account.tax.md) | `_check_children_scope` | ValidationError | Nested group of taxes are not allowed. |
| [`account.tax`](entities/account.tax.md) | `_check_company_consistency` | UserError | You can't change the company of your tax since there are some journal items linked to it. |
| [`account.tax`](entities/account.tax.md) | `unlink_except_tax_used` | ValidationError | You cannot delete taxes that are currently in use. Consider archiving them instead. |
| [`account.tax`](entities/account.tax.md) | `_eval_tax_amount_formula` | ValidationError | Only primitive types are allowed in python tax formula context. |
| [`account.tax`](entities/account.tax.md) | `_check_amount_type` | UserError | Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations. |
| [`account.tax`](entities/account.tax.md) | `write` | UserError | It is forbidden to modify a tax used in a POS order not posted. You must close the POS sessions before modifying the tax. |
| [`account.tax`](entities/account.tax.md) | `_l10n_it_edi_check_exoneration_with_no_tax` | ValidationError | If the tax amount is 0%, you must enter the exoneration code and the related legal notes. |
| [`account.tax`](entities/account.tax.md) | `_l10n_it_edi_check_exoneration_with_no_tax` | UserError | Split Payment is not compatible with exoneration of kind 'N6' |
| [`account.tax`](entities/account.tax.md) | `_validate_withholding` | ValidationError | Tax '%s' has a withholding type so the amount must be negative. |
| [`account.tax`](entities/account.tax.md) | `_validate_withholding` | ValidationError | Tax '%s' has a withholding type, so the withholding reason must also be specified |
| [`account.tax`](entities/account.tax.md) | `_validate_withholding` | ValidationError | Tax '%s' has a withholding reason, so the withholding type must also be specified |
| [`account.tax`](entities/account.tax.md) | `_validate_withholding` | ValidationError | Tax '%s' has one of withholding and pension fund types that do not relate to ENASARCO, and one that does. |
| [`account.tax`](entities/account.tax.md) | `_validate_withholding` | ValidationError | Tax '%s' has withholding type ENASARCO, the withholding reason should be [ZO] - Other reason. |
| [`account.tax`](entities/account.tax.md) | `_never_unlink_declaration_of_intent_tax` | UserError | You cannot delete the special tax for Declarations of Intent. |
| [`account.tax`](entities/account.tax.md) | `_l10n_sa_constrain_is_retention` | UserError | The tax is unable to be set as Retention as the Amount is greater than or equal to 0. |
| [`account.tax`](entities/account.tax.md) | `_check_special_tax_type_constrains` | UserError | Invalid special tax type for Duty free tax type. |
| [`account.tax`](entities/account.tax.md) | `_check_special_tax_type_constrains` | UserError | Zero tax rate and Duty free tax type must have a tax amount of 0. |
| [`account.tax.group`](entities/account.tax.group.md) | `check_uninstall_required` | UserError | The tax group '%s' can't be removed, since it is required in the Argentinian localization. |
| [`account.update.tax.tags.wizard`](entities/account.update.tax.tags.wizard.md) | `update_amls_tax_tags` | UserError | Update with children taxes that are child of multiple parents is not supported. |
| [`account.withholding.line`](entities/account.withholding.line.md) | `_constrains_base_amount` | UserError | The base amount of a withholding tax line must be above 0. |
| [`account.withholding.line`](entities/account.withholding.line.md) | `_constrains_account_id` | UserError | The account "%(account_name)s" is not valid to use on withholding lines. |
| [`account.withholding.line`](entities/account.withholding.line.md) | `_prepare_withholding_amls_create_values` | UserError | Please enter the withholding number for the tax %(tax_name)s |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | A user already exists with this identification. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | response['error'] |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | e.message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | A user already exists with theses credentials on our server. Please check your information. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | EDI user should be of one of the following types: %s |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | token_out_of_sync_error_message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | error_message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | e.message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | We could not find a user with this information on our server. Please check your information. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_peppol_proxy` | UserError | token_out_of_sync_error_message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_mark_connection_out_of_sync` | UserError | This connection has been superseded by another database. Register again. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_get_proxy_identification` | UserError | Please fill in the EAS code and the Participant ID code. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_peppol_get_new_documents` | UserError | msg |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_peppol_register_sender_as_receiver` | UserError | Cannot register a user with a %s application |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_peppol_register_sender_as_receiver` | UserError | error_msg |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_peppol_send_response` | ValidationError | At least one reason must be given when rejecting a Peppol invoice. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_nemhandel_proxy` | UserError | EDI user should be of type Nemhandel |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_nemhandel_proxy` | UserError | errors.get(error_code) or error_message or _('Connection error, please try again later.') |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_call_nemhandel_proxy` | UserError | e.message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_check_user_on_alternative_service` | UserError | error_msg |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_get_proxy_identification` | UserError | Please fill in the Identifier Type and Value. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | e.message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_nemhandel_get_new_documents` | UserError | msg |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_nemhandel_register_as_receiver` | UserError | Cannot register a user with a %s application |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_nemhandel_register_as_receiver` | ValidationError | If you try to register with your CVR, please make sure your company has the same VAT |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_get_proxy_identification` | UserError | Please fill the Peppol Endpoint field with scheme '%s' on the company partner. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | error_message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_register_proxy_user` | UserError | e.message |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_pdp_register_receiver` | UserError | This is only possible for the 'Approved Platform'. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_pdp_register_receiver` | UserError | Cannot register a user with a '%(proxy_state)s' application. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_pdp_send_response` | UserError | Unsupported response status: '%s'. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_get_proxy_identification` | UserError | Please fill your codice fiscale to be able to receive invoices from FatturaPA |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_get_proxy_identification` | UserError | Please fill the TIN of company "%(company_name)s" before enabling the integration with MyInvois. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_l10n_my_edi_contact_proxy` | UserError | The MyInvois server is temporarily unreachable. Please try again later. |
| [`account_edi_proxy_client.user`](entities/account_edi_proxy_client.user.md) | `_l10n_my_edi_contact_proxy` | UserError | You have reached the maximum number of requests allowed in a short period of time. Please wait a few minutes before trying again. |
| [`analytic.mixin`](entities/analytic.mixin.md) | `_search_analytic_distribution` | UserError | Operation not supported |
| [`analytic.mixin`](entities/analytic.mixin.md) | `_validate_distribution` | ValidationError | One or more lines require a 100% analytic distribution. |
| [`analytic.plan.fields.mixin`](entities/analytic.plan.fields.mixin.md) | `_check_account_id` | ValidationError | At least one analytic account must be set |
| [`applicant.get.refuse.reason`](entities/applicant.get.refuse.reason.md) | `action_refuse_reason_apply` | UserError | Unable to post message, please configure the sender's email address. |
| [`applicant.get.refuse.reason`](entities/applicant.get.refuse.reason.md) | `action_refuse_reason_apply` | UserError | At least one applicant doesn't have a email; you can't use send email option. |
| [`auth.passkey.key`](entities/auth.passkey.key.md) | `_get_session_challenge` | AccessDenied | Cannot find a challenge for this session |
| [`auth_totp.wizard`](entities/auth_totp.wizard.md) | `enable` | UserError | Verification failed, please double-check the 6-digit code |
| [`auth_totp.wizard`](entities/auth_totp.wizard.md) | `enable` | UserError | The verification code should only contain numbers |
| [`barcode.nomenclature`](entities/barcode.nomenclature.md) | `_unlink_except_default` | UserError | You cannot delete '%(name)s' because it's the default barcode nomenclature. |
| [`barcode.nomenclature`](entities/barcode.nomenclature.md) | `_check_pattern` | ValidationError | The FNC1 Separator Alternative is not a valid Regex: %(error)s |
| [`barcode.nomenclature`](entities/barcode.nomenclature.md) | `gs1_date_to_date` | ValidationError | A GS1 barcode nomenclature pattern was matched. However, the barcode failed to be converted to a valid date: '%(error_message)s' |
| [`barcode.nomenclature`](entities/barcode.nomenclature.md) | `parse_gs1_rule_pattern` | ValidationError | There is something wrong with the barcode rule "%s" pattern. If this rule uses decimal, check it can't get sometime else than a digit as last char for the Application Identifier. Check also the possible matched values can only be digits, otherwise the value can't be casted as a measure. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: braces can only contain N's followed by D's. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: a rule can only contain one pair of braces. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | The barcode pattern %(pattern)s does not lead to a valid regular expression. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: empty braces. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | '*' is not a valid Regex Barcode Pattern. Did you mean '.*'? |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | The rule pattern "%s" is not valid, it needs two groups: 	- A first one for the Application Identifier (usually 2 to 4 digits); 	- A second one to catch the value. |
| [`barcode.rule`](entities/barcode.rule.md) | `_check_pattern` | ValidationError | The rule pattern '%(rule)s' is not a valid Regex: %(error)s |
| [`base`](entities/base.md) | `_get_default_calendar_view` | UserError | Insufficient fields for Calendar View! |
| [`base`](entities/base.md) | `_get_default_calendar_view` | UserError | Insufficient fields to generate a Calendar View for %s, missing a date_stop or a date_delay |
| [`base`](entities/base.md) | `_get_view` | UserError | No default view of type '%s' could be found! |
| [`base`](entities/base.md) | `search_panel_select_range` | UserError | Only types %(supported_types)s are supported for category (found type %(field_type)s) |
| [`base`](entities/base.md) | `search_panel_select_multi_range` | UserError | Only types %(supported_types)s are supported for filter (found type %(field_type)s) |
| [`base`](entities/base.md) | `_find_value_from_field_path` | UserError | %(model_name)s.%(field_path)s does not seem to be a valid field path |
| [`base`](entities/base.md) | `_find_value_from_field_path` | UserError | We were not able to fetch value of field '%(field)s' |
| [`base.automation`](entities/base.automation.md) | `_check_trigger` | ValidationError | Mail event can not be configured on model %s. Only models with discussion feature can be used. |
| [`base.automation`](entities/base.automation.md) | `_check_action_server_model` | ValidationError | Target model of actions %(action_names)s are different from rule model. |
| [`base.automation`](entities/base.automation.md) | `_check_time_trigger` | ValidationError | Delay must be positive. Set 'Delay mode' to 'Before' to negate the delay. |
| [`base.automation`](entities/base.automation.md) | `_check_trigger_state` | ValidationError | Following child actions have warnings: %(children)s |
| [`base.automation`](entities/base.automation.md) | `_check_trigger_state` | ValidationError | "On live update" automation rules can only be used with "Execute Python Code" action type. |
| [`base.automation`](entities/base.automation.md) | `_check_trigger_state` | ValidationError | Email, follower or activity action types cannot be used when deleting records, as there are no more records to apply these changes to! |
| [`base.automation`](entities/base.automation.md) | `action_open_scheduled_action` | MissingError | message |
| [`base.automation`](entities/base.automation.md) | `_execute_webhook` | ValidationError | No record to run the automation on was found. |
| [`base.geocoder`](entities/base.geocoder.md) | `geo_find` | UserError | Provider %s is not implemented for geolocation service. |
| [`base.geocoder`](entities/base.geocoder.md) | `_call_openstreetmap_reverse` | UserError | OpenStreetMap calls disabled in testing environment. |
| [`base.geocoder`](entities/base.geocoder.md) | `_call_googlemap` | UserError | API key for GeoCoding (Places) required. Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information. |
| [`base.geocoder`](entities/base.geocoder.md) | `_call_googlemap` | UserError | error_msg |
| [`base.geocoder`](entities/base.geocoder.md) | `_raise_query_error` | UserError | Error with geolocation server: %s |
| [`base.language.import`](entities/base.language.import.md) | `import_lang` | UserError | File "%(file_name)s" not imported due to format mismatch or a malformed file. (Valid formats are .csv, .po)  Technical Details: %(error_message)s |
| [`base.module.install.review`](entities/base.module.install.review.md) | `_get_depending_apps` | UserError | No module selected. |
| [`base.module.install.review`](entities/base.module.install.review.md) | `_get_depending_apps` | UserError | The module is already installed. |
| [`base.module.upgrade`](entities/base.module.upgrade.md) | `upgrade_module` | UserError | The following modules are not installed or unknown: %s |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_merge` | UserError | For safety reasons, you cannot merge more than 3 contacts together. You can re-open the wizard several times if needed. |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_merge` | UserError | You cannot merge a contact with one of his parent. |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_merge` | UserError | You cannot merge contacts linked to more than one user even if only one is active. |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_merge` | UserError | All contacts must have the same email. Only the Administrator can merge contacts with different emails. |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_compute_selected_groupby` | UserError | You have to specify a filter for your selection. |
| [`base.partner.merge.automatic.wizard`](entities/base.partner.merge.automatic.wizard.md) | `_update_foreign_keys` | UserError | You cannot merge these contacts because multiple contacts are enrolled in the same courses: %s |
| [`base_import.import`](entities/base_import.import.md) | `_read_file` | UserError | Unsupported file format "{}", import only supports CSV, ODS, XLS and XLSX |
| [`base_import.import`](entities/base_import.import.md) | `_read_file` | UserError | Unable to load "{extension}" file: requires Python module "{modname}" |
| [`bill.to.po.wizard`](entities/bill.to.po.wizard.md) | `action_add_to_po` | UserError | There are no products to add to the Purchase Order. Are these Down Payments? |
| [`calendar.attendee`](entities/calendar.attendee.md) | `copy` | UserError | You cannot duplicate a calendar attendee. |
| [`calendar.event`](entities/calendar.event.md) | `_check_closing_date` | ValidationError | The ending date and time cannot be earlier than the starting date and time. Meeting “%(name)s” starts at %(start_time)s and ends at %(end_time)s |
| [`calendar.event`](entities/calendar.event.md) | `_check_closing_date` | ValidationError | The ending date cannot be earlier than the starting date. Meeting “%(name)s” starts on %(start_date)s and ends on %(end_date)s |
| [`calendar.event`](entities/calendar.event.md) | `write` | UserError | Unable to save the recurrence with "This Event" |
| [`calendar.event`](entities/calendar.event.md) | `action_open_composer` | UserError | There are no attendees on these events |
| [`calendar.event`](entities/calendar.event.md) | `_get_time_update_dict` | UserError | You can't update a recurrence without base event. |
| [`calendar.event`](entities/calendar.event.md) | `_get_ics_file` | UserError | First you have to specify the date of the invitation. |
| [`calendar.event`](entities/calendar.event.md) | `action_send_sms` | UserError | There are no attendees on these events |
| [`calendar.event`](entities/calendar.event.md) | `_check_modify_event_permission` | ValidationError | The following event can only be updated by the organizer according to the event permissions set on Google Calendar. |
| [`calendar.event`](entities/calendar.event.md) | `_check_organizer_validation` | ValidationError | For having a different organizer in your event, it is necessary that the organizer have its the system Calendar synced with Outlook Calendar. |
| [`calendar.event`](entities/calendar.event.md) | `_check_organizer_validation` | ValidationError | It is necessary adding the proposed organizer as attendee before saving the event. |
| [`calendar.event`](entities/calendar.event.md) | `_check_recurrence_overlapping` | UserError | Outlook limitation: in a recurrence, an event cannot be moved to or before the day of the previous event, and cannot be moved to or after the day of the following event. |
| [`calendar.event`](entities/calendar.event.md) | `_forbid_recurrence_update` | UserError | error_msg |
| [`calendar.event`](entities/calendar.event.md) | `_forbid_recurrence_creation` | UserError | Due to an Outlook Calendar limitation, recurrent events must be created directly in Outlook Calendar. |
| [`calendar.event`](entities/calendar.event.md) | `_ensure_attendees_have_email` | ValidationError | For a correct synchronization between the system and Outlook Calendar, all attendees must have an email address. However, some events do not respect this condition. As long as the events are incorrect, the calendars will not be synchronized. Either update the events/attendees or archive these events %(details)s: %(invalid_events)s |
| [`calendar.recurrence`](entities/calendar.recurrence.md) | `_rrule_serialize` | UserError | The interval cannot be negative. |
| [`calendar.recurrence`](entities/calendar.recurrence.md) | `_rrule_serialize` | UserError | The number of repetitions cannot be negative. |
| [`calendar.recurrence`](entities/calendar.recurrence.md) | `_get_rrule` | UserError | You have to choose at least one day in the week |
| [`card.campaign`](entities/card.campaign.md) | `write` | ValidationError | Model of campaign %(campaign)s may not be changed as it already has cards |
| [`card.campaign`](entities/card.campaign.md) | `_get_image_b64` | UserError | An error occured while rendering a card for %(record_name)s. Try again or check the server logs for more details. |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_constrains_certificate_key_compatibility` | ValidationError | certificate.private_key_id.loading_error |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_constrains_certificate_key_compatibility` | ValidationError | The certificate and private key are not compatible. |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_constrains_certificate_key_compatibility` | ValidationError | certificate.public_key_id.loading_error |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_constrains_certificate_key_compatibility` | ValidationError | The certificate and public key are not compatible. |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_constrains_certificate_loaded` | ValidationError | certificate.loading_error or _('This certificate could not be loaded. Please provide the certificate password.') |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_get_fingerprint_bytes` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_get_public_key_bytes` | UserError | The public key from the certificate could not be loaded. |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_sign` | UserError | self.loading_error or _('This certificate is not valid, its validity has expired.') |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_sign` | UserError | No private key linked to the certificate, it is required to sign documents. |
| [`certificate.certificate`](entities/certificate.certificate.md) | `_l10n_sa_validate_csr_vals` | UserError | Please make sure the following fields are shorter than %(max_length)d bytes (note that Arabic or special characters take more space): %(error_fields_msg)s |
| [`certificate.key`](entities/certificate.key.md) | `_sign` | UserError | Make sure to use a private key to sign documents. |
| [`certificate.key`](entities/certificate.key.md) | `_sign` | UserError | The private key could not be loaded. |
| [`certificate.key`](entities/certificate.key.md) | `_sign` | UserError | self.name + ' - ' + self.loading_error |
| [`certificate.key`](entities/certificate.key.md) | `_verify` | UserError | Make sure to use a public key to verify the signature of documents. |
| [`certificate.key`](entities/certificate.key.md) | `_verify` | UserError | self.name + ' - ' + self.loading_error |
| [`certificate.key`](entities/certificate.key.md) | `_decrypt` | UserError | A private key is required to decrypt data. |
| [`certificate.key`](entities/certificate.key.md) | `_decrypt` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." |
| [`certificate.key`](entities/certificate.key.md) | `_decrypt` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for decryption: RSA. |
| [`certificate.key`](entities/certificate.key.md) | `_sign_with_key` | UserError | f"Unsupported hashing algorithm '{hashing_algorithm}'. Currently supported: sha1 and sha256." |
| [`certificate.key`](entities/certificate.key.md) | `_sign_with_key` | UserError | The private key could not be loaded. |
| [`certificate.key`](entities/certificate.key.md) | `_sign_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: ED25519, EC and RSA. |
| [`certificate.key`](entities/certificate.key.md) | `_verify_with_key` | UserError | f"Unsupported signature algorithm '{signature_algorithm}'. Currently supported: sha1 and sha256." |
| [`certificate.key`](entities/certificate.key.md) | `_verify_with_key` | UserError | The public key could not be loaded. |
| [`certificate.key`](entities/certificate.key.md) | `_verify_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported for signature: EC and RSA. |
| [`certificate.key`](entities/certificate.key.md) | `_numbers_public_key_bytes_with_key` | UserError | The public key could not be loaded. |
| [`certificate.key`](entities/certificate.key.md) | `_numbers_public_key_bytes_with_key` | UserError | Unsupported asymmetric cryptography algorithm '%s'. Currently supported: EC, RSA. |
| [`certificate.key`](entities/certificate.key.md) | `_generate_ec_private_key` | UserError | f"Unsupported curve algorithm '{curve}'. Currently supported: SECP256R1." |
| [`certificate.key`](entities/certificate.key.md) | `_generate_rsa_private_key` | UserError | The public exponent should be 65537 (or 3 for legacy purposes). |
| [`certificate.key`](entities/certificate.key.md) | `_generate_rsa_private_key` | UserError | The key size should be at least 512 bytes. |
| [`change.password.own`](entities/change.password.own.md) | `_check_password_confirmation` | ValidationError | The new password and its confirmation must be identical. |
| [`chatbot.script`](entities/chatbot.script.md) | `_check_question_selection` | ValidationError | Step of type 'Question' must have answers. |
| [`chatbot.script.step`](entities/chatbot.script.step.md) | `_process_answer` | ValidationError | "%s" is not a valid email. |
| [`choose.delivery.carrier`](entities/choose.delivery.carrier.md) | `update_price` | UserError | vals.get('error_message') |
| [`choose.delivery.carrier`](entities/choose.delivery.carrier.md) | `button_confirm` | ValidationError | Please, choose a Parcel Point |
| [`compliance.letter.wizard`](entities/compliance.letter.wizard.md) | `generate_letter` | UserError | Compliance letters can only be created for companies registered in Malta. Please ensure the company's country is set to Malta. |
| [`coupon.share`](entities/coupon.share.md) | `_check_program` | ValidationError | A coupon is needed for coupon programs. |
| [`coupon.share`](entities/coupon.share.md) | `_check_website` | ValidationError | The shared website should correspond to the website of the program. |
| [`coupon.share`](entities/coupon.share.md) | `create_share_action` | UserError | Provide either a coupon or a program. |
| [`crm.iap.lead.mining.request`](entities/crm.iap.lead.mining.request.md) | `_perform_request` | UserError | Your request could not be executed: %s |
| [`crm.lead`](entities/crm.lead.md) | `_check_won_validity` | ValidationError | A lead in a Won stage cannot be lost. Move it to another stage first. |
| [`crm.lead`](entities/crm.lead.md) | `_handle_won_lost` | ValidationError | The lead %s cannot be won and lost at the same time. |
| [`crm.lead`](entities/crm.lead.md) | `_merge_opportunity` | UserError | Select at least two Leads/Opportunities from the list to merge them. |
| [`crm.lead`](entities/crm.lead.md) | `_merge_opportunity` | UserError | To prevent data loss, Leads and Opportunities can only be merged by groups of %(max_length)s. |
| [`crm.lead`](entities/crm.lead.md) | `_rebuild_pls_frequency_table` | UserError | You don't have the access needed to run this cron. |
| [`crm.lead`](entities/crm.lead.md) | `create` | AccessError | You cannot create leads linked to channels you don't have access to. |
| [`crm.lead`](entities/crm.lead.md) | `write` | AccessError | You cannot update a lead and link it to a channel you don't have access to. |
| [`crm.lead`](entities/crm.lead.md) | `_assert_portal_write_access` | AccessError | Only users with commercial partner which is a parent of the assigned partner can edit this lead. |
| [`crm.lead`](entities/crm.lead.md) | `update_contact_details_from_portal` | UserError | Not allowed to update the following field(s): %s. |
| [`crm.lead.forward.to.partner`](entities/crm.lead.forward.to.partner.md) | `action_forward` | UserError | The Forward Email Template is not in the database |
| [`crm.lead.forward.to.partner`](entities/crm.lead.forward.to.partner.md) | `action_forward` | UserError | Set an email address for the partner %s |
| [`crm.lead.forward.to.partner`](entities/crm.lead.forward.to.partner.md) | `action_forward` | UserError | Set an email address for the partner(s): %s |
| [`crm.lead2opportunity.partner`](entities/crm.lead2opportunity.partner.md) | `default_get` | UserError | Closed/Dead leads cannot be converted into opportunities. |
| [`crm.quotation.partner`](entities/crm.quotation.partner.md) | `default_get` | UserError | You can only apply this action from a lead. |
| [`crm.reveal.rule`](entities/crm.reveal.rule.md) | `_check_regex_url` | ValidationError | Enter Valid Regex. |
| [`crm.team`](entities/crm.team.md) | `_constrains_company_members` | UserError | The following team members are not allowed in company '%(company)s' of the Sales Team '%(team)s': %(users)s |
| [`crm.team`](entities/crm.team.md) | `_unlink_except_default` | UserError | Cannot delete default team "%s" |
| [`crm.team`](entities/crm.team.md) | `_constrains_assignment_domain` | ValidationError | Assignment domain for team %(team)s is incorrectly formatted |
| [`crm.team`](entities/crm.team.md) | `_action_assign_leads` | UserError | Lead/Opportunities automatic assignment is limited to managers or administrators |
| [`crm.team`](entities/crm.team.md) | `_unlink_except_used_for_sales` | UserError | Team %(team_name)s has %(sale_order_count)s active sale orders. Consider cancelling them or archiving the team instead. |
| [`crm.team.member`](entities/crm.team.member.md) | `_constrains_membership` | ValidationError | You are trying to create duplicate membership(s). We found that %(duplicates)s already exist(s). |
| [`crm.team.member`](entities/crm.team.member.md) | `_constrains_company_membership` | UserError | User '%(user)s' is not allowed in the company '%(company)s' of the Sales Team '%(team)s'. |
| [`crm.team.member`](entities/crm.team.member.md) | `_constrains_assignment_domain` | ValidationError | Member assignment domain for user %(user)s and team %(team)s is incorrectly formatted |
| [`crm.team.member`](entities/crm.team.member.md) | `_constrains_assignment_domain_preferred` | ValidationError | Member preferred assignment domain for user %(user)s and team %(team)s is incorrectly formatted |
| [`data_recycle.model`](entities/data_recycle.model.md) | `_check_recycle_action` | UserError | This model doesn't manage archived records. Only deletion is possible. |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_check_tags` | UserError | Carrier %s cannot have the same tag in both Must Have Tags and Excluded Tags. |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_match_must_have_tags` | UserError | Invalid source document type |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_match_excluded_tags` | UserError | Invalid source document type |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_match_weight` | UserError | Invalid source document type |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_match_volume` | UserError | Invalid source document type |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_get_price_from_picking` | UserError | Not available for current order |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_get_packages_from_order` | UserError | The package cannot be created because the total weight of the products in the picking is 0.0 %s |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_get_packages_from_picking` | UserError | The package cannot be created because the total weight of the products in the picking is 0.0 %s |
| [`delivery.carrier`](entities/delivery.carrier.md) | `base_on_rule_send_shipping` | ValidationError | There is no matching delivery rule. |
| [`delivery.carrier`](entities/delivery.carrier.md) | `available_carriers` | UserError | Invalid source document type |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_check_in_store_dm_has_warehouses_when_published` | ValidationError | The delivery method must have at least one warehouse to be published. |
| [`delivery.carrier`](entities/delivery.carrier.md) | `_check_warehouses_have_same_company` | ValidationError | The delivery method and a warehouse must share the same company |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_account_total_revenue_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_crm_lead_created_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_crm_opportunities_won_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_sale_total_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_hr_recruitment_new_colleagues_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_project_task_opened_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_pos_total_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`digest.digest`](entities/digest.digest.md) | `_compute_kpi_website_sale_total_value` | AccessError | Do not have access, skip this data for user's digest email |
| [`discuss.channel`](entities/discuss.channel.md) | `_constraint_from_message_id` | ValidationError | Cannot create %(channels)s: initial message should belong to parent channel or one of its sub-channels. |
| [`discuss.channel`](entities/discuss.channel.md) | `_constraint_parent_channel_id` | ValidationError | Cannot create %(channels)s: parent should not be a sub-channel and should be of type 'channel' or 'group'. The sub-channel should have the same type as the parent. |
| [`discuss.channel`](entities/discuss.channel.md) | `_constraint_partners_chat` | ValidationError | A channel of type 'chat' cannot have more than two users. |
| [`discuss.channel`](entities/discuss.channel.md) | `_constraint_group_id_channel` | ValidationError | For %(channels)s, channel_type should be 'channel' to have the group-based authorization or group auto-subscription. |
| [`discuss.channel`](entities/discuss.channel.md) | `create` | ValidationError | Invalid value when creating a channel with members, only 4 or 6 are allowed. |
| [`discuss.channel`](entities/discuss.channel.md) | `create` | ValidationError | Invalid value when creating a channel with memberships, only 0 is allowed. |
| [`discuss.channel`](entities/discuss.channel.md) | `create` | ValidationError | Invalid field “%(field_name)s” when creating a channel with members. |
| [`discuss.channel`](entities/discuss.channel.md) | `_unlink_except_all_employee_channel` | UserError | You cannot delete those groups, as the Whole Company group is required by other modules. |
| [`discuss.channel`](entities/discuss.channel.md) | `write` | UserError | Cannot change initial message nor parent channel of: %(channels)s. |
| [`discuss.channel`](entities/discuss.channel.md) | `write` | UserError | Cannot change the channel type of: %(channel_names)s |
| [`discuss.channel`](entities/discuss.channel.md) | `write` | UserError | Cannot change authorized group of sub-channel: %(channels)s. |
| [`discuss.channel`](entities/discuss.channel.md) | `invite_by_email` | AccessError | You don't have access to invite users to this channel. |
| [`discuss.channel`](entities/discuss.channel.md) | `invite_by_email` | UserError | Inviting by email is not allowed for this channel type (%s). |
| [`discuss.channel`](entities/discuss.channel.md) | `invite_by_email` | UserError | error_msg |
| [`discuss.channel`](entities/discuss.channel.md) | `_check_can_update_message_content` | UserError | Only messages type comment can have their content updated on model 'discuss.channel' |
| [`discuss.channel`](entities/discuss.channel.md) | `_message_subscribe` | UserError | Adding followers on channels is not possible. Consider adding members instead. |
| [`discuss.channel`](entities/discuss.channel.md) | `_get_or_create_chat` | UserError | A chat should not be created with more than 2 persons. Create a group instead. |
| [`discuss.channel`](entities/discuss.channel.md) | `_constraint_subscription_department_ids_channel` | ValidationError | For %(channels)s, channel_type should be 'channel' to have the department auto-subscription. |
| [`discuss.channel.member`](entities/discuss.channel.member.md) | `_contrains_no_public_member` | ValidationError | Channel members cannot include public users. |
| [`discuss.channel.member`](entities/discuss.channel.member.md) | `create` | UserError | It appears you're trying to create a channel member, but it seems like you forgot to specify the related channel. To move forward, please make sure to provide the necessary channel information. |
| [`discuss.channel.member`](entities/discuss.channel.member.md) | `create` | UserError | Adding more members to this chat isn't possible; it's designed for just two people. |
| [`discuss.channel.member`](entities/discuss.channel.member.md) | `write` | AccessError | You can not write on %(field_name)s. |
| [`event.booth`](entities/event.booth.md) | `_unlink_except_linked_sale_order` | UserError | You can't delete the following booths as they are linked to sales orders: %(booths)s |
| [`event.booth.category`](entities/event.booth.category.md) | `_check_service_tracking` | ValidationError | The product, %(product_name)s , is used for Event Booth, it must have service_tracking set to "Event Booth". |
| [`event.booth.configurator`](entities/event.booth.configurator.md) | `_check_if_no_booth_ids` | ValidationError | You have to select at least one booth. |
| [`event.event`](entities/event.event.md) | `_check_slots_dates` | ValidationError | These events cannot have slots scheduled outside of their time range: %(event_names)s |
| [`event.event`](entities/event.event.md) | `_check_closing_date` | ValidationError | The closing date cannot be earlier than the beginning date. |
| [`event.event`](entities/event.event.md) | `_check_event_url` | ValidationError | Please enter a valid event URL. |
| [`event.event`](entities/event.event.md) | `_verify_seats_availability` | ValidationError | There are not enough seats available for %(event_name)s: %(sold_out_info)s |
| [`event.event`](entities/event.event.md) | `action_generate_leads` | UserError | Only Event Managers are allowed to re-generate all leads. |
| [`event.event`](entities/event.event.md) | `_check_website_id` | ValidationError | The website must be from the same company as the event. |
| [`event.event.configurator`](entities/event.event.configurator.md) | `check_event_id` | ValidationError | '\n'.join(error_messages) |
| [`event.event.ticket`](entities/event.event.ticket.md) | `_constrains_dates_coherency` | UserError | The stop date cannot be earlier than the start date. Please check ticket %(ticket_name)s |
| [`event.event.ticket`](entities/event.event.ticket.md) | `_constrains_limit_max_per_order` | UserError | The limit per order cannot be greater than the maximum seats number. Please check ticket %(ticket_name)s |
| [`event.event.ticket`](entities/event.event.ticket.md) | `_constrains_limit_max_per_order` | UserError | The limit per order cannot be greater than %(limit_orderable)s. Please check ticket %(ticket_name)s |
| [`event.event.ticket`](entities/event.event.ticket.md) | `_constrains_limit_max_per_order` | UserError | The limit per order must be positive. Please check ticket %(ticket_name)s |
| [`event.event.ticket`](entities/event.event.ticket.md) | `_unlink_except_if_registrations` | UserError | The following tickets cannot be deleted while they have one or more registrations linked to them: - %s |
| [`event.question`](entities/event.question.md) | `write` | UserError | You cannot change the question type of a question that already has answers! |
| [`event.question`](entities/event.question.md) | `_unlink_except_answered_question` | UserError | You cannot delete a question that has already been answered by attendees. You can archive it instead. |
| [`event.question`](entities/event.question.md) | `_unlink_except_default_question` | UserError | You cannot delete a default question. |
| [`event.question.answer`](entities/event.question.answer.md) | `_unlink_except_selected_answer` | UserError | You cannot delete an answer that has already been selected by attendees. |
| [`event.quiz.question`](entities/event.quiz.question.md) | `_check_answers_integrity` | ValidationError | Question "%s" must have 1 correct answer to be valid. |
| [`event.quiz.question`](entities/event.quiz.question.md) | `_check_answers_integrity` | ValidationError | Question "%s" must have 1 correct answer and at least 1 incorrect answer to be valid. |
| [`event.registration`](entities/event.registration.md) | `_check_event_slot` | ValidationError | Invalid event / slot choice |
| [`event.registration`](entities/event.registration.md) | `_check_event_slot` | ValidationError | Slot choice is mandatory on multi-slots events. |
| [`event.registration`](entities/event.registration.md) | `_check_event_ticket` | ValidationError | Invalid event / ticket choice |
| [`event.slot`](entities/event.slot.md) | `_check_hours` | ValidationError | A slot hour must be between 0:00 and 23:59. |
| [`event.slot`](entities/event.slot.md) | `_check_hours` | ValidationError | A slot end hour must be later than its start hour. %s |
| [`event.slot`](entities/event.slot.md) | `_check_time_range` | ValidationError | A slot cannot be scheduled outside of its event time range.  Event:		%(event_start)s - %(event_end)s Slot:		%(slot_name)s |
| [`event.slot`](entities/event.slot.md) | `_unlink_except_if_registrations` | UserError | The following slots cannot be deleted while they have one or more registrations linked to them: - %s |
| [`event.track`](entities/event.track.md) | `_search_wishlist_visitor_ids` | UserError | Unsupported 'Not In' operation on track wishlist visitors |
| [`fetchmail.server`](entities/fetchmail.server.md) | `_connect__` | UserError | The server "%s" cannot be used because it is archived. |
| [`fetchmail.server`](entities/fetchmail.server.md) | `button_confirm_login` | UserError | Invalid server name!  %s |
| [`fetchmail.server`](entities/fetchmail.server.md) | `button_confirm_login` | UserError | No response received. Check server information.  %s |
| [`fetchmail.server`](entities/fetchmail.server.md) | `button_confirm_login` | UserError | Server replied with following exception:  %s |
| [`fetchmail.server`](entities/fetchmail.server.md) | `button_confirm_login` | UserError | An SSL exception occurred. Check SSL/TLS configuration on server port.  %s |
| [`fetchmail.server`](entities/fetchmail.server.md) | `button_confirm_login` | UserError | Connection test failed: %s |
| [`fetchmail.server`](entities/fetchmail.server.md) | `_check_use_google_gmail_service` | UserError | SSL is required for server “%s”. |
| [`fetchmail.server`](entities/fetchmail.server.md) | `_check_use_microsoft_outlook_service` | UserError | SSL is required for server “%s”. |
| [`fleet.vehicle`](entities/fleet.vehicle.md) | `write` | UserError | The odometer value cannot be lower than the previous one. |
| [`fleet.vehicle.log.services`](entities/fleet.vehicle.log.services.md) | `_set_odometer` | UserError | Emptying the odometer value of a vehicle is not allowed. |
| [`fleet.vehicle.log.services`](entities/fleet.vehicle.log.services.md) | `_inverse_amount` | UserError | You cannot modify amount of services linked to an account move line. Do it on the related accounting entry instead. |
| [`fleet.vehicle.log.services`](entities/fleet.vehicle.log.services.md) | `_unlink_if_no_linked_bill` | UserError | You cannot delete log services records because one or more of them were bill created. |
| [`forum.post`](entities/forum.post.md) | `_check_parent_id` | ValidationError | You cannot create recursive forum posts. |
| [`forum.post`](entities/forum.post.md) | `create` | UserError | Posting answer on a [Deleted] or [Closed] question is not possible. |
| [`forum.post`](entities/forum.post.md) | `create` | AccessError | %d karma required to create a new question. |
| [`forum.post`](entities/forum.post.md) | `create` | AccessError | %d karma required to answer a question. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to edit a post. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to delete or reactivate a post. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to accept or refuse an answer. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to retag. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to close or reopen a post. |
| [`forum.post`](entities/forum.post.md) | `write` | AccessError | %d karma required to flag a post. |
| [`forum.post`](entities/forum.post.md) | `_unlink_if_enough_karma` | AccessError | %d karma required to unlink a post. |
| [`forum.post`](entities/forum.post.md) | `_update_content` | AccessError | %d karma required to post an image or link. |
| [`forum.post`](entities/forum.post.md) | `validate` | AccessError | %d karma required to validate a post. |
| [`forum.post`](entities/forum.post.md) | `_refuse` | AccessError | %d karma required to refuse a post. |
| [`forum.post`](entities/forum.post.md) | `_flag` | AccessError | %d karma required to flag a post. |
| [`forum.post`](entities/forum.post.md) | `_mark_as_offensive` | AccessError | %d karma required to mark a post as offensive. |
| [`forum.post`](entities/forum.post.md) | `convert_answer_to_comment` | AccessError | %d karma required to convert an answer to a comment. |
| [`forum.post`](entities/forum.post.md) | `convert_comment_to_answer` | AccessError | %d karma required to convert your comment to an answer. |
| [`forum.post`](entities/forum.post.md) | `convert_comment_to_answer` | AccessError | %d karma required to convert a comment to an answer. |
| [`forum.post`](entities/forum.post.md) | `unlink_comment` | AccessError | %d karma required to delete a comment. |
| [`forum.post`](entities/forum.post.md) | `message_post` | AccessError | %d karma required to comment. |
| [`forum.post.vote`](entities/forum.post.vote.md) | `_check_general_rights` | UserError | It is not allowed to vote for its own post. |
| [`forum.post.vote`](entities/forum.post.vote.md) | `_check_general_rights` | UserError | It is not allowed to modify someone else's vote. |
| [`forum.post.vote`](entities/forum.post.vote.md) | `_check_karma_rights` | AccessError | %d karma required to upvote. |
| [`forum.post.vote`](entities/forum.post.vote.md) | `_check_karma_rights` | AccessError | %d karma required to downvote. |
| [`forum.tag`](entities/forum.tag.md) | `create` | AccessError | %d karma required to create a new Tag. |
| [`gamification.badge`](entities/gamification.badge.md) | `check_granting` | UserError | This badge can not be sent by users. |
| [`gamification.badge`](entities/gamification.badge.md) | `check_granting` | UserError | You are not in the user allowed list. |
| [`gamification.badge`](entities/gamification.badge.md) | `check_granting` | UserError | You do not have the required badges. |
| [`gamification.badge`](entities/gamification.badge.md) | `check_granting` | UserError | You have already sent this badge too many time this month. |
| [`gamification.badge.user`](entities/gamification.badge.user.md) | `_check_employee_related_user` | ValidationError | The selected employee does not correspond to the selected user. |
| [`gamification.badge.user.wizard`](entities/gamification.badge.user.wizard.md) | `action_grant_badge` | UserError | You can not grant a badge to yourself. |
| [`gamification.badge.user.wizard`](entities/gamification.badge.user.wizard.md) | `action_grant_badge` | UserError | You can not send a badge to yourself. |
| [`gamification.challenge`](entities/gamification.challenge.md) | `write` | UserError | You can not reset a challenge with unfinished goals. |
| [`gamification.challenge`](entities/gamification.challenge.md) | `_get_serialized_challenge_lines` | UserError | Retrieving progress for personal challenge without user information |
| [`gamification.goal`](entities/gamification.goal.md) | `write` | UserError | Can not modify the configuration of a started goal |
| [`gamification.goal.definition`](entities/gamification.goal.definition.md) | `_check_domain_validity` | UserError | The domain for the definition %(definition)s seems incorrect, please check it.  %(error_message)s |
| [`gamification.goal.definition`](entities/gamification.goal.definition.md) | `_check_model_validity` | UserError | The model configuration for the definition %(name)s seems incorrect, please check it.  %(field_name)s not stored |
| [`gamification.goal.definition`](entities/gamification.goal.definition.md) | `_check_model_validity` | UserError | The model configuration for the definition %(name)s seems incorrect, please check it.  %(error)s not found |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `open_google_gmail_uri` | AccessError | Only the administrator can link a Gmail mail server. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `open_google_gmail_uri` | UserError | Please enter a valid email address. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `open_google_gmail_uri` | UserError | Please configure your Gmail credentials. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `open_google_gmail_uri` | UserError | Please configure your Gmail credentials. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `open_google_gmail_uri` | UserError | Oops, we could not authenticate you. Please try again later. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `_fetch_gmail_token` | UserError | An error occurred when fetching the access token. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `_fetch_gmail_access_token_iap` | UserError | Oops, we could not authenticate you. Please try again later. |
| [`google.gmail.mixin`](entities/google.gmail.mixin.md) | `_raise_iap_error` | UserError | get_iap_error_message(self.env, error) |
| [`hr.applicant`](entities/hr.applicant.md) | `_check_talent_pool_required` | ValidationError | Talent must belong to at least one Talent Pool. |
| [`hr.applicant`](entities/hr.applicant.md) | `_inverse_partner_email` | UserError | You must define a Contact Name for this applicant. |
| [`hr.applicant`](entities/hr.applicant.md) | `copy` | UserError | You cannot duplicate the talent(s). |
| [`hr.applicant`](entities/hr.applicant.md) | `action_create_meeting` | UserError | You must define a Contact Name for this applicant. |
| [`hr.applicant`](entities/hr.applicant.md) | `create_employee_from_applicant` | UserError | Please provide an applicant name. |
| [`hr.applicant`](entities/hr.applicant.md) | `_check_interviewer_access` | UserError | You are not allowed to perform this action. |
| [`hr.applicant`](entities/hr.applicant.md) | `action_send_survey` | UserError | Please provide an applicant name. |
| [`hr.applicant`](entities/hr.applicant.md) | `website_form_input_filter` | UserError | The job offer has been closed. |
| [`hr.attendance`](entities/hr.attendance.md) | `_check_validity_check_in_check_out` | ValidationError | "Check Out" time cannot be earlier than "Check In" time. |
| [`hr.attendance`](entities/hr.attendance.md) | `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s |
| [`hr.attendance`](entities/hr.attendance.md) | `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee hasn't checked out since %(datetime)s |
| [`hr.attendance`](entities/hr.attendance.md) | `_check_validity` | ValidationError | Cannot create new attendance record for %(empl_name)s, the employee was already checked in on %(datetime)s |
| [`hr.attendance`](entities/hr.attendance.md) | `write` | AccessError | Do not have access, user cannot edit the attendances that are not their own or if they are not the attendance manager of the employee. |
| [`hr.attendance`](entities/hr.attendance.md) | `copy` | UserError | You cannot duplicate an attendance. |
| [`hr.attendance.overtime.rule`](entities/hr.attendance.overtime.rule.md) | `_check_expected_hours` | ValidationError | Rule '%(name)s' is based off quantity, but the usual amount of work hours is not specified |
| [`hr.attendance.overtime.rule`](entities/hr.attendance.overtime.rule.md) | `_check_expected_hours` | ValidationError | Rule '%(name)s' is based off quantity, but the period is not specified |
| [`hr.attendance.overtime.rule`](entities/hr.attendance.overtime.rule.md) | `_check_work_schedule` | ValidationError | Rule '%(name)s' is based off timing, but the work schedule is not specified |
| [`hr.bank.account.allocation.wizard`](entities/hr.bank.account.allocation.wizard.md) | `_prepare_allocations_from_employee` | ValidationError | Bank account %s not found within the salary distribution of the employee |
| [`hr.bank.account.allocation.wizard`](entities/hr.bank.account.allocation.wizard.md) | `action_save` | ValidationError | Total percentage allocation must equal 100%. |
| [`hr.department`](entities/hr.department.md) | `_check_parent_id` | ValidationError | You cannot create recursive departments. |
| [`hr.departure.reason`](entities/hr.departure.reason.md) | `_unlink_except_default_departure_reasons` | UserError | Default departure reasons cannot be deleted. |
| [`hr.departure.wizard`](entities/hr.departure.wizard.md) | `action_register_departure` | UserError | Departure date can't be earlier than the start date of current contract. |
| [`hr.employee`](entities/hr.employee.md) | `_check_salary_distribution` | ValidationError | Total salary distribution on bank accounts must be exactly 100%. |
| [`hr.employee`](entities/hr.employee.md) | `_check_salary_distribution` | ValidationError | Each amount percentage must be a number between 0 and 100. |
| [`hr.employee`](entities/hr.employee.md) | `check_no_existing_contract` | ValidationError | The employee is already in contract on %s. Please select a date outside existing contracts |
| [`hr.employee`](entities/hr.employee.md) | `_get_first_versions_filtered` | AccessError | Only HR users can access first version date on an employee. |
| [`hr.employee`](entities/hr.employee.md) | `_create_work_contacts` | UserError | Some employee already have a work contact |
| [`hr.employee`](entities/hr.employee.md) | `action_create_user` | ValidationError | This employee already has an user. |
| [`hr.employee`](entities/hr.employee.md) | `_check_private_fields` | AccessError | The fields “%s”, which you are trying to read, are not available for employee public profiles. |
| [`hr.employee`](entities/hr.employee.md) | `_search` | AccessError | You do not have access to this document. |
| [`hr.employee`](entities/hr.employee.md) | `_search` | AccessError | You do not have access to this document. |
| [`hr.employee`](entities/hr.employee.md) | `_verify_pin` | ValidationError | The PIN must be a sequence of digits. |
| [`hr.employee`](entities/hr.employee.md) | `_verify_barcode` | ValidationError | The Badge ID must be alphanumeric without any accents and no longer than 18 characters. |
| [`hr.employee`](entities/hr.employee.md) | `_get_version_periods` | UserError | This field %(field_name)s doesn't exist on this model (hr.version). |
| [`hr.employee`](entities/hr.employee.md) | `_attendance_action_change` | UserError | Cannot perform check out on %(empl_name)s, could not find corresponding check in. Your attendances have probably been modified manually by human resources. |
| [`hr.employee`](entities/hr.employee.md) | `_check_work_contact_id` | ValidationError | Cannot remove address from employees with linked cars. |
| [`hr.employee`](entities/hr.employee.md) | `write` | ValidationError | Changing this working schedule results in the affected employee(s) not having enough leaves allocated to accomodate for their leaves already taken in the future. Please review this employee's leaves and adjust their allocation accordingly. |
| [`hr.employee`](entities/hr.employee.md) | `_action_set_manual_presence` | UserError | You don't have the right to do this. Please contact an Administrator. |
| [`hr.employee`](entities/hr.employee.md) | `action_send_sms` | UserError | You don't have the right to do this. Please contact an Administrator. |
| [`hr.employee`](entities/hr.employee.md) | `action_send_log` | UserError | You don't have the right to do this. Please contact an Administrator. |
| [`hr.employee`](entities/hr.employee.md) | `get_internal_resume_lines` | AccessError | You cannot access the resume of this employee. |
| [`hr.employee`](entities/hr.employee.md) | `action_unlink_wizard` | UserError | You cannot delete employees who have timesheets. |
| [`hr.employee`](entities/hr.employee.md) | `_unlink_except_active_pos_session` | UserError | error_msg |
| [`hr.expense`](entities/hr.expense.md) | `_default_employee_id` | ValidationError | The current user has no related employee. Please, create one. |
| [`hr.expense`](entities/hr.expense.md) | `_check_non_zero` | ValidationError | Only draft expenses can have a total of 0. |
| [`hr.expense`](entities/hr.expense.md) | `_check_o2o_payment` | ValidationError | Only one expense can be linked to a particular payment |
| [`hr.expense`](entities/hr.expense.md) | `_inverse_total_amount_currency` | UserError | Uh-oh! You can’t edit this expense.  Reach out to the administrators, flash your best smile, and see if they'll grant you the magical access you seek. |
| [`hr.expense`](entities/hr.expense.md) | `_unlink_except_approved` | UserError | You cannot delete a posted or approved expense. |
| [`hr.expense`](entities/hr.expense.md) | `write` | UserError | You cannot edit the security fields of an expense manually |
| [`hr.expense`](entities/hr.expense.md) | `write` | UserError | Uh-oh! You can’t edit this expense.  Reach out to the administrators, flash your best smile, and see if they'll grant you the magical access you seek. |
| [`hr.expense`](entities/hr.expense.md) | `action_submit` | UserError | You do not have the required permission to submit this expense. |
| [`hr.expense`](entities/hr.expense.md) | `action_submit` | UserError | You can not submit an expense without a category. |
| [`hr.expense`](entities/hr.expense.md) | `action_post` | UserError | You can't post simultaneously employee-paid expenses belonging to different companies |
| [`hr.expense`](entities/hr.expense.md) | `action_post` | UserError | The vendor is required for expenses using SEPA Credit Transfer as the payment method. Please set a vendor on the following expenses: %s |
| [`hr.expense`](entities/hr.expense.md) | `attach_document` | AccessError | You don't have the access rights to modify this expense. |
| [`hr.expense`](entities/hr.expense.md) | `attach_document` | UserError | You can't add an attachment to an expense once it has been approved. |
| [`hr.expense`](entities/hr.expense.md) | `create_expense_from_attachments` | UserError | No attachment was provided |
| [`hr.expense`](entities/hr.expense.md) | `create_expense_from_attachments` | UserError | Invalid attachments! |
| [`hr.expense`](entities/hr.expense.md) | `create_expense_from_attachments` | UserError | You need to have at least one category that can be expensed in your database to proceed! |
| [`hr.expense`](entities/hr.expense.md) | `action_split_wizard` | UserError | You cannot split an expense that is already posted. |
| [`hr.expense`](entities/hr.expense.md) | `action_split_wizard` | UserError | You do not have the rights to edit this expense. |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_approve` | UserError | reasons |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_refuse` | UserError | reasons |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_reset_approval` | UserError | Only HR Officers, accountants, or the concerned employee can reset to draft. |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_reset_approval` | UserError | You cannot reset to draft an expense linked to a posted journal entry. |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_create_move` | UserError | You can only generate an accounting entry for approved expense(s). |
| [`hr.expense`](entities/hr.expense.md) | `_check_can_create_move` | UserError | Please specify if the expenses were paid by the company, or the employee. |
| [`hr.expense`](entities/hr.expense.md) | `_do_refuse` | UserError | You cannot cancel an expense linked to a posted journal entry |
| [`hr.expense`](entities/hr.expense.md) | `_post_wizard` | UserError | Only expense paid by the employee can be posted with the wizard |
| [`hr.expense`](entities/hr.expense.md) | `_prepare_payments_vals` | UserError | You need to add a manual payment method on the journal (%s) |
| [`hr.expense`](entities/hr.expense.md) | `_get_base_account` | UserError | the system had a look at your expense, its product, your company and the journal but came back with empty hands. Give the system a hand to find an account by setting up an expense account. %(expense)s %(expense_name)s. |
| [`hr.expense`](entities/hr.expense.md) | `_get_expense_account_destination` | UserError | The following expenses payment method leads to several accounts payable and this isn't supported: %(expenses)s |
| [`hr.expense`](entities/hr.expense.md) | `_get_expense_account_destination` | UserError | No work contact found for the employee %(name)s, please configure one. |
| [`hr.expense.post.wizard`](entities/hr.expense.post.wizard.md) | `action_post_entry` | UserError | You don't have the rights to create accounting entries. |
| [`hr.individual.skill.mixin`](entities/hr.individual.skill.mixin.md) | `_check_not_overlapping_regular_skill` | ValidationError | error_msg |
| [`hr.individual.skill.mixin`](entities/hr.individual.skill.mixin.md) | `_check_date` | ValidationError | The following skills have their valid stop date prior to their valid start date: |
| [`hr.individual.skill.mixin`](entities/hr.individual.skill.mixin.md) | `_check_skill_type` | ValidationError | The skill %(name)s and skill type %(type)s don't match |
| [`hr.individual.skill.mixin`](entities/hr.individual.skill.mixin.md) | `_check_skill_level` | ValidationError | The skill level %(level)s is not valid for skill type: %(type)s |
| [`hr.leave`](entities/hr.leave.md) | `_check_contracts` | ValidationError | A leave cannot be set across multiple versions with different working schedules.  Please create one time off for each version period.  Time off: %(time_off)s  Versions: %(versions)s |
| [`hr.leave`](entities/hr.leave.md) | `_check_date` | ValidationError | holiday.dashboard_warning_message |
| [`hr.leave`](entities/hr.leave.md) | `_check_date_state` | ValidationError | This modification is not allowed in the current state. |
| [`hr.leave`](entities/hr.leave.md) | `_check_validity` | ValidationError | You are not allowed to request time off on a Mandatory Day |
| [`hr.leave`](entities/hr.leave.md) | `_check_validity` | ValidationError | You do not have any allocation for this time off type. Please request an allocation before submitting your time off request. |
| [`hr.leave`](entities/hr.leave.md) | `_check_validity` | ValidationError | %(name)s does not have a valid allocation for the leave type %(leave_type)s to cover that request. |
| [`hr.leave`](entities/hr.leave.md) | `_check_validity` | ValidationError | You do not have any allocation for this time off type. Please request an allocation before submitting your time off request. |
| [`hr.leave`](entities/hr.leave.md) | `_check_validity` | ValidationError | %(name)s does not have a valid allocation for the leave type %(leave_type)s to cover that request. |
| [`hr.leave`](entities/hr.leave.md) | `_check_double_validation_rules` | AccessError | You cannot first approve a time off for %s, because you are not his time off manager |
| [`hr.leave`](entities/hr.leave.md) | `_check_double_validation_rules` | AccessError | You don't have the rights to apply second approval on a time off request |
| [`hr.leave`](entities/hr.leave.md) | `create` | UserError | There is no employee set on the time off. Please make sure you're logged in the correct company. |
| [`hr.leave`](entities/hr.leave.md) | `write` | UserError | You must have manager rights to modify/validate a time off that already begun |
| [`hr.leave`](entities/hr.leave.md) | `write` | UserError | Only a manager can modify a canceled leave. |
| [`hr.leave`](entities/hr.leave.md) | `_unlink_if_correct_states` | UserError | error_message % {'state': state_description_values.get(self[:1].state)} |
| [`hr.leave`](entities/hr.leave.md) | `_unlink_if_correct_states` | UserError | You can't delete a time off request that is in the past. |
| [`hr.leave`](entities/hr.leave.md) | `_unlink_if_correct_states` | UserError | error_message % {'state': state_description_values.get(holiday.state)} |
| [`hr.leave`](entities/hr.leave.md) | `copy_data` | UserError | A time off cannot be duplicated. |
| [`hr.leave`](entities/hr.leave.md) | `action_approve` | UserError | You cannot approve this leave. |
| [`hr.leave`](entities/hr.leave.md) | `_action_validate` | UserError | You can't validate this leave. |
| [`hr.leave`](entities/hr.leave.md) | `_action_validate` | ValidationError | The following employees are not supposed to work during that period:  %s |
| [`hr.leave`](entities/hr.leave.md) | `action_refuse` | UserError | Time off request must be confirmed or validated in order to refuse it. |
| [`hr.leave`](entities/hr.leave.md) | `_action_user_cancel` | ValidationError | This time off cannot be cancelled. |
| [`hr.leave`](entities/hr.leave.md) | `_check_approval_update` | UserError | error_message |
| [`hr.leave`](entities/hr.leave.md) | `_check_approval_update` | UserError | e |
| [`hr.leave`](entities/hr.leave.md) | `_check_overtime_deductible` | ValidationError | The employee does not have enough extra hours to request this leave. |
| [`hr.leave`](entities/hr.leave.md) | `_check_overtime_deductible` | ValidationError | You do not have enough extra hours to request this leave |
| [`hr.leave`](entities/hr.leave.md) | `_get_fr_date_from_to` | UserError | An employee can't take paid time off in a period without any work hours. |
| [`hr.leave`](entities/hr.leave.md) | `_l10n_in_check_optional_holiday_request_dates` | ValidationError | The following leaves are not on Optional Holidays:  - %s |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | `_check_dates` | ValidationError | error_message |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | `_check_maximum_leaves` | UserError | You cannot have a balance cap on accrued time set to 0. |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | `_get_next_date` | ValidationError | Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly. |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | `_get_previous_date` | ValidationError | Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly. |
| [`hr.leave.accrual.level`](entities/hr.leave.accrual.level.md) | `_check_worked_hours` | ValidationError | You can't base accrued time on hours worked, because time is accrued at the start of the period. |
| [`hr.leave.accrual.plan`](entities/hr.leave.accrual.plan.md) | `_prevent_used_plan_unlink` | ValidationError | Some of the accrual plans you're trying to delete are linked to an existing allocation. Delete or cancel them first. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_check_date_from_date_to` | UserError | The Start Date of the Validity Period must be anterior to the End Date. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `create` | UserError | Incorrect state for new allocation |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `write` | ValidationError | You cannot reduce the duration below the duration of leaves already taken by the employee. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_unlink_if_correct_states` | UserError | You cannot delete an allocation request which is in %s state. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_unlink_if_no_leaves` | UserError | You cannot delete an allocation request which has some validated leaves. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `action_approve` | UserError | Allocation must be "To Approve" in order to approve it. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `action_refuse` | UserError | Allocation request must be confirmed, second approval or validated in order to refuse it. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_check_approval_update` | UserError | error_message |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_check_approval_update` | UserError | e |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `write` | ValidationError | Only an Officer or Administrator is allowed to edit the allocation duration in this status. |
| [`hr.leave.allocation`](entities/hr.leave.allocation.md) | `_check_employee_overtime_balance` | ValidationError | The employee does not have enough overtime hours to request this leave. |
| [`hr.leave.allocation.generate.multi.wizard`](entities/hr.leave.allocation.generate.multi.wizard.md) | `_check_allocation_mode` | AccessError | As Time Off Responsible, you can only use the allocation mode 'By Employee'. |
| [`hr.leave.generate.multi.wizard`](entities/hr.leave.generate.multi.wizard.md) | `action_generate_time_off` | UserError | Some employees already have time off requests in hours that overlap with the selected period, the system cannot automatically adjust or split hourly leaves during batch generation. Conflicting time off: %s |
| [`hr.leave.generate.multi.wizard`](entities/hr.leave.generate.multi.wizard.md) | `_check_allocation_mode` | AccessError | As Time Off Responsible, you can only use the allocation mode 'By Employee'. |
| [`hr.leave.report.calendar`](entities/hr.leave.report.calendar.md) | `action_approve` | ValidationError | You are not allowed to approve this leave request. |
| [`hr.leave.report.calendar`](entities/hr.leave.report.calendar.md) | `action_refuse` | ValidationError | You are not allowed to refuse this leave request. |
| [`hr.leave.type`](entities/hr.leave.type.md) | `_check_allow_request_on_top` | ValidationError | You cannot allow requests on top of leaves of type 'Absence'. |
| [`hr.leave.type`](entities/hr.leave.type.md) | `_check_elligible_for_accrual_rate` | ValidationError | leaves of type 'Worked Time' should be always eligible for accrual rate. |
| [`hr.leave.type`](entities/hr.leave.type.md) | `_check_overlapping_public_holidays` | ValidationError | You cannot modify the 'Public Holiday Included' setting since one or more leaves for that                         time off type are overlapping with public holidays, meaning that the balance of those employees would be affected by this change. |
| [`hr.leave.type`](entities/hr.leave.type.md) | `check_allocation_requirement_edit_validity` | UserError | The allocation requirement of a time off type cannot be changed once leaves of that type have been taken. You should create a new time off type instead. |
| [`hr.skill.type`](entities/hr.skill.type.md) | `_check_no_null_skill_or_skill_level` | ValidationError | The following skills type must contain at least one skill and one level: %s |
| [`hr.version`](entities/hr.version.md) | `_check_dates` | ValidationError | Start date (%(start)s) must be earlier than contract end date (%(end)s). |
| [`hr.version`](entities/hr.version.md) | `_check_dates` | ValidationError | %s already has a contract running during the selected period.  Please either:  - Change the start date so that it doesn't overlap with the existing contract, or - Create a new employee if this employee should have multiple active contracts. |
| [`hr.version`](entities/hr.version.md) | `check_contract_finished` | ValidationError | Before creating a new contract, close the current one by setting an end date. |
| [`hr.version`](entities/hr.version.md) | `_unlink_except_last_version` | ValidationError | Employee %s must always have at least one active version. |
| [`hr.version`](entities/hr.version.md) | `write` | ValidationError | Cannot unassign all the active versions of an employee. |
| [`hr.version`](entities/hr.version.md) | `write` | ValidationError | Cannot archive all the active versions of an employee. |
| [`hr.version`](entities/hr.version.md) | `write` | ValidationError | Cannot modify multiple versions contract dates with different contracts at once. |
| [`hr.version`](entities/hr.version.md) | `create` | ValidationError | Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.  This error has been triggered by: |
| [`hr.version`](entities/hr.version.md) | `write` | ValidationError | Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.  This error has been triggered by: |
| [`hr.version`](entities/hr.version.md) | `_generate_work_entries_postprocess` | UserError | Missing timezone for work entries generation. |
| [`hr.version`](entities/hr.version.md) | `_generate_work_entries_postprocess` | UserError | Missing date or duration on work entry |
| [`hr.work.entry`](entities/hr.work.entry.md) | `_check_duration` | ValidationError | Duration must be positive and cannot exceed 24 hours. |
| [`hr.work.entry`](entities/hr.work.entry.md) | `action_split` | UserError | You can't split a work entry with less than 1 hour. |
| [`hr.work.entry`](entities/hr.work.entry.md) | `action_split` | UserError | Split work entry duration has to be less than the existing work entry duration. |
| [`hr.work.entry`](entities/hr.work.entry.md) | `_unlink_except_validated_work_entries` | UserError | This work entry is validated. You can't delete it. |
| [`hr.work.entry.regeneration.wizard`](entities/hr.work.entry.regeneration.wizard.md) | `regenerate_work_entries` | ValidationError | In order to regenerate the work entries, you need to provide the wizard with an employee_id, a date_from and a date_to. |
| [`hr.work.entry.regeneration.wizard`](entities/hr.work.entry.regeneration.wizard.md) | `regenerate_work_entries` | ValidationError | The from date must be >= '%(earliest_available_date)s' and the to date must be <= '%(latest_available_date)s', which correspond to the generated work entries time interval. |
| [`hr.work.entry.regeneration.wizard`](entities/hr.work.entry.regeneration.wizard.md) | `regenerate_work_entries` | ValidationError | No work entry can be regenerated in this range of dates and these employees. |
| [`hr.work.entry.type`](entities/hr.work.entry.type.md) | `_check_work_entry_type_country` | UserError | You can't change the country of this specific work entry type. |
| [`hr.work.entry.type`](entities/hr.work.entry.type.md) | `_check_work_entry_type_country` | UserError | You can't change the Country of this work entry type cause it's currently used by the system. You need to delete related working entries first. |
| [`hr.work.entry.type`](entities/hr.work.entry.type.md) | `_check_code_unicity` | UserError | The same code cannot be associated to multiple work entry types (%s) |
| [`hr.work.location`](entities/hr.work.location.md) | `_unlink_except_used_by_employee` | UserError | You cannot delete locations that are being used by your employees |
| [`html.field.history.mixin`](entities/html.field.history.mixin.md) | `write` | ValidationError | 'Ensure all versioned fields ( %s ) in model %s are declared as sanitize=True' % (str(versioned_fields), rec._name) |
| [`iap.account`](entities/iap.account.md) | `validate_warning_alerts` | UserError | Please set a positive email alert threshold. |
| [`iap.account`](entities/iap.account.md) | `validate_warning_alerts` | UserError | One of the email alert recipients doesn't have an email address set. Users: %s |
| [`iap.account`](entities/iap.account.md) | `get` | UserError | No service exists with the provided technical name |
| [`iap.account`](entities/iap.account.md) | `_hash_iap_token` | UserError | The IAP token provided is invalid or empty. |
| [`iap.autocomplete.api`](entities/iap.autocomplete.api.md) | `_contact_iap` | ValidationError | Test mode |
| [`im_livechat.channel`](entities/im_livechat.channel.md) | `_check_review_link` | ValidationError | Invalid URL '%s'. The Review Link must start with 'http://' or 'https://'. |
| [`im_livechat.channel`](entities/im_livechat.channel.md) | `action_join` | AccessError | Only Live Chat operators can join Live Chat channels |
| [`im_livechat.channel.member.history`](entities/im_livechat.channel.member.history.md) | `_constraint_channel_id` | ValidationError | Cannot create history as it is only available for live chats: %(histories)s. |
| [`ir.actions.act_window`](entities/ir.actions.act_window.md) | `_check_model` | ValidationError | Invalid model name “%s” in action definition. |
| [`ir.actions.act_window`](entities/ir.actions.act_window.md) | `_check_model` | ValidationError | Invalid model name “%s” in action definition. |
| [`ir.actions.act_window`](entities/ir.actions.act_window.md) | `_check_view_mode` | ValidationError | The modes in view_mode must not be duplicated: %s |
| [`ir.actions.act_window`](entities/ir.actions.act_window.md) | `_check_view_mode` | ValidationError | No spaces allowed in view_mode: “%s” |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | `_check_path` | ValidationError | The path should contain only lowercase alphanumeric characters, underscore, and dash, and it should start with a letter. |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | `_check_path` | ValidationError | 'm-' is a reserved prefix. |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | `_check_path` | ValidationError | 'action-' is a reserved prefix. |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | `_check_path` | ValidationError | 'new' is reserved, and can not be used as path. |
| [`ir.actions.actions`](entities/ir.actions.actions.md) | `_check_path` | ValidationError | Path to show in the URL must be unique! Please choose another one. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_run_wkhtmltoimage` | UserError | wkhtmltoimage 0.12.0^ is required in order to render images from html |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_run_wkhtmltopdf` | UserError | message |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_run_wkhtmltopdf` | UserError | Tried to convert multiple documents in wkhtmltopdf using unpatched QT |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_handle_merge_pdfs_error` | UserError | the system is unable to merge the generated PDFs. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_merge_pdfs` | UserError | the system is unable to merge the generated PDFs. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_render_qweb_pdf_prepare_streams` | UserError | Unable to find Wkhtmltopdf on this system. The PDF can not be created. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_render_qweb_pdf_prepare_streams` | UserError | Report template “%s” has an issue, please contact your administrator.   Cannot separate file to save as attachment because the report's template does not contain the attributes 'data-oe-model' and 'data-oe-id' as part of the div with 'article' classname. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_render_qweb_pdf_prepare_streams` | UserError | No original purchase document could be found for any of the selected purchase documents. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_pre_render_qweb_pdf` | UserError | Only invoices could be printed. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_unlink_except_master_tags` | UserError | You cannot delete this report (%s), it is used by the accounting PDF generation engine. |
| [`ir.actions.report`](entities/ir.actions.report.md) | `_pre_render_qweb_pdf` | UserError | Only invoices could be printed. |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_get_relation_chain` | ValidationError | The path contained by the field '%(searched_field)s' contains a non-relational field (%(current_field)s) that is not the last field in the path. You can't traverse non-relational fields (even in the quantum realm). Make sure only the last field in the path is non-relational. |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_check_python_code` | ValidationError | msg |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_check_children` | ValidationError | Recursion found in child server actions |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_check_children` | ValidationError | Following child actions have warnings: %(children)s |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_run_action_webhook` | UserError | I'll be happy to send a webhook for you, but you really need to give me a URL to reach out to... |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. |
| [`ir.actions.server`](entities/ir.actions.server.md) | `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. |
| [`ir.attachment`](entities/ir.attachment.md) | `force_storage` | AccessError | Only administrators can execute this action. |
| [`ir.attachment`](entities/ir.attachment.md) | `_get_path` | UserError | The attachment collides with an existing file. |
| [`ir.attachment`](entities/ir.attachment.md) | `_check_serving_attachments` | ValidationError | Sorry, you are not allowed to write on this document |
| [`ir.attachment`](entities/ir.attachment.md) | `_check_circular_attachment` | ValidationError | You cannot attach an attachment to itself. Attachment %(record)s cannot have res_id: %(res_id)s |
| [`ir.attachment`](entities/ir.attachment.md) | `check` | AccessError | Sorry, you are not allowed to access this document. |
| [`ir.attachment`](entities/ir.attachment.md) | `check` | AccessError | Sorry, you are not allowed to access this document. |
| [`ir.attachment`](entities/ir.attachment.md) | `write` | AccessError | Sorry, you are not allowed to access this document. |
| [`ir.attachment`](entities/ir.attachment.md) | `create` | AccessError | Sorry, you are not allowed to access this document. |
| [`ir.attachment`](entities/ir.attachment.md) | `create_unique` | UserError | Attachment is not encoded in base64. |
| [`ir.attachment`](entities/ir.attachment.md) | `_can_return_content` | AccessError | Invalid access token |
| [`ir.attachment`](entities/ir.attachment.md) | `_migrate_remote_to_local` | ValidationError | URL attachment (%s) shouldn't be migrated to local. |
| [`ir.attachment`](entities/ir.attachment.md) | `_has_attachments_ownership` | UserError | An access token must be provided for each attachment. |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_government_document` | UserError | You can't unlink an attachment being an EDI document sent to the government. |
| [`ir.attachment`](entities/ir.attachment.md) | `_post_add_create` | UserError | Cloud Storage is not enabled |
| [`ir.attachment`](entities/ir.attachment.md) | `_migrate_remote_to_local` | ValidationError | Failed to download attachment (%(id)s) from cloud: %(code)s - %(reason)s |
| [`ir.attachment`](entities/ir.attachment.md) | `_get_cloud_storage_azure_info` | ValidationError | %s is not a valid Azure Blob Storage URL. |
| [`ir.attachment`](entities/ir.attachment.md) | `_get_cloud_storage_google_info` | ValidationError | %s is not a valid Google Cloud Storage URL. |
| [`ir.attachment`](entities/ir.attachment.md) | `_migrate_local_to_cloud_storage` | ValidationError | Attachment (%s) is not a binary attachment and cannot be migrated to cloud storage. |
| [`ir.attachment`](entities/ir.attachment.md) | `_migrate_local_to_cloud_storage` | ValidationError | Attachment (%s) does not have a stored filename and cannot be migrated to cloud storage. |
| [`ir.attachment`](entities/ir.attachment.md) | `_migrate_local_to_cloud_storage` | ValidationError | Failed to upload attachment %(id)s to cloud storage: %(code)s |
| [`ir.attachment`](entities/ir.attachment.md) | `_cron_migrate_local_to_cloud_storage` | UserError | Cloud storage provider is not configured |
| [`ir.attachment`](entities/ir.attachment.md) | `_cron_migrate_local_to_cloud_storage` | UserError | No model for cloud storage migration |
| [`ir.attachment`](entities/ir.attachment.md) | `_prevent_delete_from_submitted_expense` | AccessError | You can't delete attachments from an expense once it has been submitted. |
| [`ir.attachment`](entities/ir.attachment.md) | `create` | AccessError | You can't add attachments to an expense once it has been approved. |
| [`ir.attachment`](entities/ir.attachment.md) | `_l10n_fr_pdp_no_delete_fro_sent_flow` | UserError | You can't delete an attachment linked to a sent Flow. |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_l10n_in_government_document` | UserError | You can't unlink an attachment that you received from the government |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_ewaybill_government_document` | UserError | You can't unlink an attachment that you received from the government |
| [`ir.attachment`](entities/ir.attachment.md) | `_except_submitted_invoices_pdfs` | UserError | You cannot delete this Invoice PDF as it has been submitted to JoFotara |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_posted_pdf_invoices` | UserError | The Invoice PDF(s) cannot be deleted according to ZATCA rules: %s |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_rejected_zatca_document` | UserError | You can't unlink an attachment being an EDI document refused by the government. |
| [`ir.attachment`](entities/ir.attachment.md) | `_unlink_except_validated_pdf_invoices` | UserError | Oops! The invoice PDF(s) are linked to a validated EDI document and cannot be deleted according to ZATCA rules: %s |
| [`ir.binary`](entities/ir.binary.md) | `_find_record` | MissingError | f'No record found for xmlid={xmlid}, res_model={res_model}, id={res_id}' |
| [`ir.binary`](entities/ir.binary.md) | `_record_to_stream` | MissingError | The related attachment does not exist. |
| [`ir.binary`](entities/ir.binary.md) | `_get_stream_from` | UserError | f'Field {field_def!r} is type {field_def.type!r} but it is only possible to stream Binary or Image fields.' |
| [`ir.binary`](entities/ir.binary.md) | `_get_stream_from` | UserError | f'Record has no field {field_name!r}.' |
| [`ir.config_parameter`](entities/ir.config_parameter.md) | `write` | ValidationError | You cannot rename config parameters with keys %s |
| [`ir.config_parameter`](entities/ir.config_parameter.md) | `unlink_default_parameters` | ValidationError | You cannot delete the %s record. |
| [`ir.config_parameter`](entities/ir.config_parameter.md) | `write` | UserError | The value for %s must be the ID to a valid analytic plan that is not a subplan |
| [`ir.cron`](entities/ir.cron.md) | `method_direct_trigger` | UserError | Job '%s' already executing |
| [`ir.cron`](entities/ir.cron.md) | `write` | UserError | Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes |
| [`ir.cron`](entities/ir.cron.md) | `_unlink_unless_running` | UserError | Record cannot be modified right now: This cron task is currently being executed and may not be modified Please try again in a few minutes |
| [`ir.default`](entities/ir.default.md) | `_check_json_format` | ValidationError | Invalid JSON format in Default Value field. |
| [`ir.default`](entities/ir.default.md) | `_check_json_format` | ValidationError | Invalid value in Default Value field. Expected type '%(field_type)s' for '%(model_name)s.%(field_name)s'. |
| [`ir.default`](entities/ir.default.md) | `set` | ValidationError | Invalid value for %(model)s.%(field)s: %(value)s is out of bounds (integers should be between -2,147,483,648 and 2,147,483,647) |
| [`ir.default`](entities/ir.default.md) | `set` | ValidationError | Invalid field %(model)s.%(field)s |
| [`ir.default`](entities/ir.default.md) | `set` | ValidationError | Invalid value for %(model)s.%(field)s: %(value)s |
| [`ir.embedded.actions`](entities/ir.embedded.actions.md) | `_unlink_if_action_deletable` | UserError | You cannot delete a default embedded action |
| [`ir.http`](entities/ir.http.md) | `_auth_method_bearer` | AccessDenied | e |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | ValidationError | The reCaptcha private key is invalid. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | ValidationError | The reCaptcha token is invalid. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | Your request has timed out, please retry. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | The request is invalid or malformed. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | Suspicious activity detected by google reCAPTCHA. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | ValidationError | The Cloudflare turnstile private key is invalid. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | ValidationError | The CloudFlare human validation failed. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | Your request has timed out, please retry. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | The request is invalid or malformed. |
| [`ir.http`](entities/ir.http.md) | `_verify_request_recaptcha_token` | UserError | Suspicious activity detected by Turnstile CAPTCHA. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_smtp_ssl_files` | UserError | SSL private key is missing for %s. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_smtp_ssl_files` | UserError | SSL certificate is missing for %s. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `write` | UserError | You cannot archive this Outgoing Mail Server (%(server_usage)s) because it is still used in the following case(s): %(usage_details)s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `write` | UserError | You cannot archive these Outgoing Mail Servers (%(server_usage)s) because they are still used in the following case(s): %(usage_details)s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_get_test_email_from` | UserError | Please configure an email on the current user to simulate sending an email message via this outgoing server |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | The server refused the sender address (%(email_from)s) with error %(repl)s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | The server refused the test recipient (%(email_to)s) with error %(repl)s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | The server refused the test connection with error %(repl)s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | Invalid server name!  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | No response received. Check server address and port number.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | The server has closed the connection unexpectedly. Check configuration served on this port number.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | Server replied with following exception:  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | An option is not supported by the server:  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | An SMTP exception occurred. Check port number and connection security type.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | An SSL exception occurred. Check connection security type.  CertificateError: %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | An SSL exception occurred. Check connection security type.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | Connection Test Failed! Here is what we got instead:  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `test_smtp_connection` | UserError | The server "%(server_name)s" doesn't return the maximum email size. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_connect__` | UserError | Missing SMTP Server Please define at least one SMTP server, or provide the SMTP parameters explicitly. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_connect__` | UserError | The private key or the certificate is not a valid file.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_connect__` | UserError | Could not load your certificate / private key.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_connect__` | UserError | The private key or the certificate is not a valid file.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_connect__` | UserError | Could not load your certificate / private key.  %s |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_forced_mail_server` | UserError | The server "%s" cannot be used because it is archived. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as it belongs to a user. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as it belongs to a user and is archived. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_forced_mail_server` | UserError | The server "%s" cannot be forced as the owner does not use it anymore. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_google_gmail_service` | UserError | Please leave the password field empty for Gmail mail server “%s”. The OAuth process does not require it |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_google_gmail_service` | UserError | Incorrect Connection Security for Gmail mail server “%s”. Please set it to "TLS (STARTTLS)". |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_google_gmail_service` | UserError | Please fill the "Username" field with your Gmail username (your email address). This should be the same account as the one used for the Gmail OAuthentication Token. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_owner_user_id_not_mass_mailing` | ValidationError | Cannot set an owner on '%(server)s': it is configured as the dedicated Email Marketing server. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_owner_user_id_not_mass_mailing` | ValidationError | Cannot set an owner on '%(server)s': it is used by mailing '%(mailing)s'. |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_microsoft_outlook_service` | UserError | Please leave the password field empty for Outlook mail server “%s”. The OAuth process does not require it |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_microsoft_outlook_service` | UserError | Incorrect Connection Security for Outlook mail server “%s”. Please set it to "TLS (STARTTLS)". |
| [`ir.mail_server`](entities/ir.mail_server.md) | `_check_use_microsoft_outlook_service` | UserError | Please fill the "Username" field with your Outlook/Office365 username (your email address). This should be the same account as the one used for the Outlook OAuthentication Token. |
| [`ir.model`](entities/ir.model.md) | `_check_model_name` | ValidationError | The model name can only contain lowercase characters, digits, underscores and dots. |
| [`ir.model`](entities/ir.model.md) | `_check_order` | ValidationError | str(e) |
| [`ir.model`](entities/ir.model.md) | `_check_order` | ValidationError | Unable to order by %s: fields used for ordering must be present on the model and stored. |
| [`ir.model`](entities/ir.model.md) | `_check_fold_name` | ValidationError | The value of 'Fold Field' should be a field name of the model. |
| [`ir.model`](entities/ir.model.md) | `_unlink_if_manual` | UserError | Model “%s” contains module data and cannot be removed. |
| [`ir.model`](entities/ir.model.md) | `write` | UserError | Field %s cannot be modified on models. |
| [`ir.model`](entities/ir.model.md) | `_check_manual_name` | ValidationError | The model name must start with 'x_'. |
| [`ir.model`](entities/ir.model.md) | `write` | UserError | Only custom models can be modified. |
| [`ir.model`](entities/ir.model.md) | `write` | UserError | Field "Mail Thread" cannot be changed to "False". |
| [`ir.model`](entities/ir.model.md) | `write` | UserError | Field "Mail Activity" cannot be changed to "False". |
| [`ir.model`](entities/ir.model.md) | `write` | UserError | Field "Mail Blacklist" cannot be changed to "False". |
| [`ir.model.data`](entities/ir.model.data.md) | `check_object_reference` | AccessError | Not enough access rights on the external ID "%(module)s.%(xml_id)s" |
| [`ir.model.data`](entities/ir.model.data.md) | `_module_data_uninstall` | AccessError | Administrator access is required to uninstall a module |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_domain` | ValidationError | An error occurred while evaluating the domain: %(error)s |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_name` | ValidationError | msg |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_related_field` | UserError | Unknown field name "%(field_name)s" in related field "%(related_field)s" |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_related_field` | UserError | Non-relational field name "%(field_name)s" in related field "%(related_field)s" |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_related_field` | UserError | Field "%(field_name)s" in related path "%(related_field)s" is not searchable. Non-searchable fields cannot be used in related fields. |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_related` | ValidationError | Related field "%(related_field)s" does not have type "%(type)s" |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_related` | ValidationError | Related field "%(related_field)s" does not have comodel "%(comodel)s" |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_relation` | ValidationError | Unknown model name '%s' in Related Model |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_depends` | UserError | Empty dependency in “%s” |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_depends` | UserError | Compute method cannot depend on field 'id' |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_depends` | UserError | Unknown field “%(field)s” in dependency “%(dependency)s” |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_depends` | UserError | Non-relational field “%(field)s” in dependency “%(dependency)s” |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_currency_field` | ValidationError | Currency field does not have type many2one |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_currency_field` | ValidationError | Currency field should have a res.currency relation |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_currency_field` | ValidationError | Currency field is empty and there is no fallback field in the model |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_currency_field` | ValidationError | Unknown field specified “%s” in currency_field |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_on_delete_required_m2o` | ValidationError | The m2o field %s is required but declares its ondelete policy as being 'set null'. Only 'restrict' and 'cascade' make sense. |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_prepare_update` | UserError | This column contains module data and cannot be removed! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_prepare_update` | UserError | The field '%(field)s' cannot be removed because the field '%(other_field)s' depends on it. |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_prepare_update` | UserError | Cannot rename/delete fields that are still present in views: Fields: %(fields)s View: %(view)s |
| [`ir.model.fields`](entities/ir.model.fields.md) | `create` | UserError | Model %s does not exist! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `create` | UserError | Many2one %(field)s on model %(model)s does not exist! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Changing the model of a field is forbidden! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Changing the type of a field is not yet supported. Please drop it and create it again! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Can only rename one field at a time! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Changing the storing system for field "%s" is not allowed. |
| [`ir.model.fields`](entities/ir.model.fields.md) | `write` | UserError | Renaming sparse field "%s" is not allowed |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_reflect_fields` | UserError | Serialization field "%(serialization_field)s" not found for sparse field %(sparse_field)s! |
| [`ir.model.fields`](entities/ir.model.fields.md) | `_check_if_used_in_website_form` | ValidationError | The field '%(field)s' cannot be deleted because it is referenced in a website view. Model: %(model)s View: %(view)s |
| [`ir.model.fields.selection`](entities/ir.model.fields.selection.md) | `_reflect_selections` | ValidationError | Fields %s contain a non-str value/label in selection |
| [`ir.model.fields.selection`](entities/ir.model.fields.selection.md) | `create` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! |
| [`ir.model.fields.selection`](entities/ir.model.fields.selection.md) | `write` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! |
| [`ir.model.fields.selection`](entities/ir.model.fields.selection.md) | `_unlink_if_manual` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! |
| [`ir.model.relation`](entities/ir.model.relation.md) | `_module_data_uninstall` | AccessError | Administrator access is required to uninstall a module |
| [`ir.module.category`](entities/ir.module.category.md) | `_check_parent_not_circular` | ValidationError | Error ! You cannot create recursive categories. |
| [`ir.module.module`](entities/ir.module.module.md) | `_unlink_except_installed` | UserError | You are trying to remove a module that is installed or will be installed. |
| [`ir.module.module`](entities/ir.module.module.md) | `check_external_dependencies` | UserError | msg |
| [`ir.module.module`](entities/ir.module.module.md) | `_state_update` | UserError | Recursion error in modules dependencies! |
| [`ir.module.module`](entities/ir.module.module.md) | `_state_update` | UserError | You try to install module "%(module)s" that depends on module "%(dependency)s". But the latter module is not available in your system. |
| [`ir.module.module`](entities/ir.module.module.md) | `button_install` | UserError | You are trying to install incompatible modules in category "%(category)s":%(module_list)s |
| [`ir.module.module`](entities/ir.module.module.md) | `button_install` | UserError | Modules "%(module)s" and "%(incompatible_module)s" are incompatible. |
| [`ir.module.module`](entities/ir.module.module.md) | `_button_immediate_function` | UserError | The method _button_immediate_install cannot be called on init or non loaded registries. Please use button_install instead. |
| [`ir.module.module`](entities/ir.module.module.md) | `_button_immediate_function` | UserError | the system is currently processing another module operation. Please try again later or contact your system administrator. |
| [`ir.module.module`](entities/ir.module.module.md) | `_button_immediate_function` | UserError | the system is currently processing another module operation. Please try again later or contact your system administrator. |
| [`ir.module.module`](entities/ir.module.module.md) | `_button_immediate_function` | UserError | the system is currently processing a scheduled action. Module operations are not possible at this time, please try again later or contact your system administrator. |
| [`ir.module.module`](entities/ir.module.module.md) | `button_uninstall` | UserError | Those modules cannot be uninstalled: %s |
| [`ir.module.module`](entities/ir.module.module.md) | `button_uninstall` | UserError | One or more of the selected modules have already been uninstalled, if you believe this to be an error, you may try again later or contact support. |
| [`ir.module.module`](entities/ir.module.module.md) | `button_upgrade` | UserError | Cannot upgrade module “%s”. It is not installed. |
| [`ir.module.module`](entities/ir.module.module.md) | `button_upgrade` | UserError | You try to upgrade the module %(module)s that depends on the module: %(dependency)s. But this module is not available in your system. |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_module` | UserError | err |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_module` | UserError | Studio customizations require the the system Studio app. |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_module` | UserError | The assets path in the manifest of imported module '%(module_name)s' cannot contain glob wildcards (e.g., *, **). |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_zipfile` | AccessError | Only administrators can install data modules. |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_zipfile` | UserError | Only zip files are supported. |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_zipfile` | UserError | File '%s' exceed maximum allowed file size |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_zipfile` | UserError | No manifest found in '%(modules)s'. Can't import the zip file. |
| [`ir.module.module`](entities/ir.module.module.md) | `_import_zipfile` | UserError | Error while importing module '%(module)s'.   %(error_message)s |
| [`ir.module.module`](entities/ir.module.module.md) | `_get_modules_from_apps` | UserError | The list of industry applications cannot be fetched. Please try again later |
| [`ir.module.module`](entities/ir.module.module.md) | `_get_modules_from_apps` | UserError | Connection to %s failed The list of industry modules cannot be fetched |
| [`ir.module.module`](entities/ir.module.module.md) | `button_immediate_install_app` | UserError | missing_dependencies_description |
| [`ir.module.module`](entities/ir.module.module.md) | `button_immediate_install_app` | UserError | The module %s cannot be downloaded |
| [`ir.module.module`](entities/ir.module.module.md) | `button_immediate_install_app` | UserError | Connection to %(url)s failed, the module %(module)s cannot be downloaded. |
| [`ir.module.module`](entities/ir.module.module.md) | `_get_missing_dependencies_modules` | UserError | File '%s' exceed maximum allowed file size |
| [`ir.module.module`](entities/ir.module.module.md) | `_update_records` | MissingError | error |
| [`ir.profile`](entities/ir.profile.md) | `_generate_speedscope` | UserError | All profiles must have the same initial stack trace to be displayed together. |
| [`ir.profile`](entities/ir.profile.md) | `set_profiling` | UserError | Profiling is not enabled on this database. Please contact an administrator. |
| [`ir.qweb.field.datetime`](entities/ir.qweb.field.datetime.md) | `from_html` | ValidationError | The datetime %(value)s does not match the format %(format)s |
| [`ir.rule`](entities/ir.rule.md) | `_check_model_name` | ValidationError | Rules can not be applied on the Record Rules model. |
| [`ir.rule`](entities/ir.rule.md) | `_check_domain` | ValidationError | Invalid domain: %s |
| [`ir.sequence`](entities/ir.sequence.md) | `_get_prefix_suffix` | UserError | Invalid prefix or suffix for sequence “%s” |
| [`ir.sequence`](entities/ir.sequence.md) | `_unlink_sequence` | UserError | You cannot delete a sequence used in an active POS config: %s |
| [`ir.ui.menu`](entities/ir.ui.menu.md) | `_check_parent_id` | ValidationError | Error! You cannot create recursive menus. |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_check_xml` | ValidationError | Invalid view %(name)s definition in %(file)s |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_check_xml` | ValidationError | Error while validating view (%(view)s):  %(error)s |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_check_groups` | ValidationError | Inherited view cannot have '%(attr)s' defined on the record. Use '%(attr)s' attributes inside the view definition |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_check_000_inheritance` | ValidationError | You cannot create recursive inherited views. |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_validate_xml_encoding` | UserError | Unicode strings with encoding declaration are not supported in XML. Remove the encoding declaration. |
| [`ir.ui.view`](entities/ir.ui.view.md) | `create` | ValidationError | Missing view architecture. |
| [`ir.ui.view`](entities/ir.ui.view.md) | `create` | ValidationError | Invalid view type: '%(view_type)s'. You might have used an invalid starting tag in the architecture. Allowed types are: %(valid_types)s |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_check_view_access` | AccessError | error |
| [`ir.ui.view`](entities/ir.ui.view.md) | `save_embedded_field` | ValidationError | Invalid field value for %(field_name)s: %(value)s |
| [`ir.ui.view`](entities/ir.ui.view.md) | `_copy_custom_snippet_translations` | ValidationError | str(e) |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `_ondelete_flow` | UserError | You cannot delete sent flows. |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `_build_payload` | UserError | Flow %(name)s has already been sent. |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `_get_pdp_proxy_user` | UserError | No active PDP proxy user is configured for company %(company)s. |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `_send_to_proxy` | UserError | The flow payload is missing. Build the payload before sending. |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `_send_to_proxy` | UserError | The PDP proxy did not return a flow tracking identifier. |
| [`l10n.fr.pdp.reports.flow`](entities/l10n.fr.pdp.reports.flow.md) | `action_send_from_ui` | UserError | This flow still contains invoices with validation errors. Fix them or use the 'Send without invalid invoices' button. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_generate_ewaybill` | UserError | '\n'.join(errors) |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_reset_to_pending` | UserError | Only Cancelled E-waybill can be resent. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_print` | UserError | Please generate the E-Waybill to print it. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `_lock_ewaybill` | UserError | This document is being sent by another process already. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `_unlink_l10n_in_ewaybill_prevent` | UserError | You cannot delete a generated E-waybill. Instead, you should cancel it. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_generate_ewaybill` | UserError | '\n'.join(errors) |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_reset_to_pending` | UserError | Only Delivery Challan and Cancelled E-waybill can be reset to pending. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_set_to_challan` | UserError | The challan can only be generated in the Pending state. |
| [`l10n.in.ewaybill`](entities/l10n.in.ewaybill.md) | `action_print` | UserError | Please generate the E-Waybill or mark the document as a Challan to print it. |
| [`l10n.in.hr.leave.optional.holiday`](entities/l10n.in.hr.leave.optional.holiday.md) | `default_get` | UserError | You must be logged in an Indian company to use this feature |
| [`l10n.in.hr.leave.optional.holiday`](entities/l10n.in.hr.leave.optional.holiday.md) | `_unlink_except_optional_holidays` | ValidationError | You cannot delete an Optional Holiday that is linked to a leave request. |
| [`l10n_ar.partner.tax`](entities/l10n_ar.partner.tax.md) | `check_partner_tax_dates` | ValidationError | "From date" must be lower than "To date" on Withholding (AR) taxes. |
| [`l10n_br.zip.range`](entities/l10n_br.zip.range.md) | `_check_range` | ValidationError | Invalid zip range format: %(start)s %(end)s. It should follow this format: 01000-001 |
| [`l10n_br.zip.range`](entities/l10n_br.zip.range.md) | `_check_range` | ValidationError | Start should be less than end: %(start)s %(end)s |
| [`l10n_ch.qr_invoice.wizard`](entities/l10n_ch.qr_invoice.wizard.md) | `default_get` | UserError | No invoice was found to be printed. |
| [`l10n_ch.qr_invoice.wizard`](entities/l10n_ch.qr_invoice.wizard.md) | `default_get` | UserError | All selected invoices must belong to the same Switzerland company |
| [`l10n_eg_edi.thumb.drive`](entities/l10n_eg_edi.thumb.drive.md) | `_get_host` | ValidationError | Please define the host of sign tool. |
| [`l10n_es_edi_tbai.document`](entities/l10n_es_edi_tbai.document.md) | `_generate_sale_document_xml` | UserError | No valid certificate found for this company, TicketBAI file will not be signed. |
| [`l10n_es_edi_tbai.document`](entities/l10n_es_edi_tbai.document.md) | `_sign_sale_document` | UserError | No certificate found |
| [`l10n_es_edi_verifactu.document`](entities/l10n_es_edi_verifactu.document.md) | `_never_unlink_chained_documents` | UserError | You cannot delete Veri*Factu Documents that are part of the chain of all Veri*Factu Documents. |
| [`l10n_hu_edi.cancellation`](entities/l10n_hu_edi.cancellation.md) | `button_request_cancel` | UserError | self.env['account.move.send']._format_error_text(self.invoice_id.l10n_hu_edi_messages) |
| [`l10n_hu_edi.tax_audit_export`](entities/l10n_hu_edi.tax_audit_export.md) | `action_export` | UserError | No invoice to export! |
| [`l10n_hu_edi_receive.bills.wizard`](entities/l10n_hu_edi_receive.bills.wizard.md) | `action_receive_bills` | UserError | The length of the interval specified by the query parameter can be up to 35 days. |
| [`l10n_id.qris.transaction`](entities/l10n_id.qris.transaction.md) | `_constraint_model` | ValidationError | QRIS capability is not extended to model %s yet! |
| [`l10n_id_efaktur_coretax.document`](entities/l10n_id_efaktur_coretax.document.md) | `_generate_xml` | UserError | Some documents don't have a transaction code: %s |
| [`l10n_id_efaktur_coretax.document`](entities/l10n_id_efaktur_coretax.document.md) | `_generate_xml` | UserError | Some documents are not Customer Invoices: %s |
| [`l10n_in.pan.entity`](entities/l10n_in.pan.entity.md) | `_check_pan_name` | ValidationError | The entered PAN %s seems invalid. Please enter a valid PAN. |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `default_get` | UserError | TDS must be created from an Invoice or a Payment. |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `default_get` | UserError | You can only create a withhold for only one record at a time. |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `default_get` | UserError | TDS must be created from Posted Customer Invoices, Customer Credit Notes, Vendor Bills or Vendor Refunds. |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `default_get` | UserError | Please set a partner on the %s before creating a withhold. |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `_check_amounts` | ValidationError | Negative or zero values are not allowed in Base Amount for withhold |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `_check_amounts` | ValidationError | Negative or zero values are not allowed in TDS Amount for withhold |
| [`l10n_in.withhold.wizard`](entities/l10n_in.withhold.wizard.md) | `_validate_withhold_data_on_post` | UserError | Please configure the withholding account from the settings |
| [`l10n_it.document.type`](entities/l10n_it.document.type.md) | `_check_code_unique` | ValidationError | Document Type code must be unique. |
| [`l10n_it_edi_doi.declaration_of_intent`](entities/l10n_it_edi_doi.declaration_of_intent.md) | `_unlink_except_linked_to_document` | UserError | You cannot delete Declarations of Intents that are already used on at least one Invoice or Sales Order. |
| [`l10n_latam.check`](entities/l10n_latam.check.md) | `_constrains_min_amount` | ValidationError | The amount of the check must be greater than 0 |
| [`l10n_latam.check`](entities/l10n_latam.check.md) | `_unlink_if_payment_is_draft` | UserError | Can't delete a check if payment is In Process! |
| [`l10n_latam.document.type`](entities/l10n_latam.document.type.md) | `_format_document_number` | UserError | %(value)s is not a valid value for %(field)s. The document number must be entered with a dash (-) and a maximum of 5 characters for the first part and 8 for the second. The following are examples of valid numbers: * 1-1 * 0001-00000001 * 00001-00000001 |
| [`l10n_latam.document.type`](entities/l10n_latam.document.type.md) | `_format_document_number` | UserError | %(value)s is not a valid value for %(field)s. The number of import Dispatch must be 16 characters. |
| [`l10n_latam.document.type`](entities/l10n_latam.document.type.md) | `_format_document_number` | UserError | Ecuadorian Document %s must be like 001-001-123456789 |
| [`l10n_latam.document.type`](entities/l10n_latam.document.type.md) | `_format_document_number` | UserError | %(document_number)s is not a valid value for %(document_type)s. The document number must be entered with a maximum of 2 letters for the first part and 7 numbers for the second. The following are examples of valid document numbers: - XX0000001  - YY0000123  - A0000001 |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | `_compute_journal_company` | UserError | All selected checks must be on the same journal and on hand |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | `default_get` | UserError | The register payment wizard should only be called on account.payment records. |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | `default_get` | UserError | You have selected payments which are not checks. Please call this action from the Third Party Checks menu |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | `default_get` | UserError | All the selected checks must use the same currency |
| [`l10n_latam.payment.mass.transfer`](entities/l10n_latam.payment.mass.transfer.md) | `default_get` | UserError | All the selected checks must be posted |
| [`l10n_sa_edi.otp.wizard`](entities/l10n_sa_edi.otp.wizard.md) | `validate` | UserError | Please provide an OTP to complete the onboarding process |
| [`l10n_tw_edi.invoice.cancel`](entities/l10n_tw_edi.invoice.cancel.md) | `button_request_cancel` | UserError | You must provide a reason for canceling the invoice. |
| [`l10n_tw_edi.invoice.print`](entities/l10n_tw_edi.invoice.print.md) | `button_print` | UserError | Error: %(error)s |
| [`l10n_vn_edi_viettel.sinvoice.symbol`](entities/l10n_vn_edi_viettel.sinvoice.symbol.md) | `_constrains_changes` | UserError | You cannot change the symbol value or template of the symbol %s because it has already been used to send invoices. |
| [`link.tracker`](entities/link.tracker.md) | `_compute_short_url` | UserError | Please enter valid short URL code. |
| [`link.tracker`](entities/link.tracker.md) | `_check_unicity` | UserError | Combinations of Link Tracker values (URL, campaign, medium, source, and label) must be unique. The following combinations are already used:  - %(error_lines)s |
| [`link.tracker`](entities/link.tracker.md) | `create` | UserError | “%s” is not a valid link, links cannot redirect to the current page. |
| [`link.tracker`](entities/link.tracker.md) | `search_or_create` | UserError | '\n'.join(errors) |
| [`loyalty.card`](entities/loyalty.card.md) | `_contrains_code` | ValidationError | A trigger with the same code as one of your coupon already exists. |
| [`loyalty.card`](entities/loyalty.card.md) | `_restrict_expiration_on_loyalty` | ValidationError | Expiration date cannot be set on a loyalty card. |
| [`loyalty.card.update.balance`](entities/loyalty.card.update.balance.md) | `action_update_card_point` | ValidationError | New Balance should be positive and different then old balance. |
| [`loyalty.generate.wizard`](entities/loyalty.generate.wizard.md) | `generate_coupons` | ValidationError | Can not generate coupon, no program is set. |
| [`loyalty.generate.wizard`](entities/loyalty.generate.wizard.md) | `generate_coupons` | ValidationError | Invalid quantity. |
| [`loyalty.program`](entities/loyalty.program.md) | `_check_pricelist_currency` | UserError | The loyalty program's currency must be the same as all it's pricelists ones. |
| [`loyalty.program`](entities/loyalty.program.md) | `_check_date_from_date_to` | UserError | The validity period's start date must be anterior or equal to its end date. |
| [`loyalty.program`](entities/loyalty.program.md) | `_constrains_reward_ids` | ValidationError | A program must have at least one reward. |
| [`loyalty.program`](entities/loyalty.program.md) | `_unlink_except_active` | UserError | You can not delete a program in an active state |
| [`loyalty.program`](entities/loyalty.program.md) | `_inverse_pos_report_print_id` | UserError | You must set '%(mail_template)s' before setting '%(report)s'. |
| [`loyalty.reward`](entities/loyalty.reward.md) | `_check_reward_product_id_no_combo` | ValidationError | A reward product can't be of type "combo". |
| [`loyalty.rule`](entities/loyalty.rule.md) | `_constraint_trigger_multi` | ValidationError | Split per unit is not allowed for Loyalty and eWallet programs. |
| [`loyalty.rule`](entities/loyalty.rule.md) | `_constrains_code` | ValidationError | The promo code must be unique. |
| [`loyalty.rule`](entities/loyalty.rule.md) | `_constrains_code` | ValidationError | A coupon with the same code was found. |
| [`loyalty.rule`](entities/loyalty.rule.md) | `_constrains_code` | ValidationError | A coupon with the same code was found. |
| [`loyalty.rule`](entities/loyalty.rule.md) | `_constrains_code` | ValidationError | The promo code must be unique. |
| [`lunch.order`](entities/lunch.order.md) | `_check_topping_quantity` | ValidationError | errors[quantity] % label |
| [`lunch.order`](entities/lunch.order.md) | `_check_wallet` | ValidationError | Oh no! You don’t have enough money in your wallet to order your selected lunch! Contact your lunch manager to add some money to your wallet. |
| [`lunch.order`](entities/lunch.order.md) | `action_order` | ValidationError | Product is no longer available. |
| [`lunch.order`](entities/lunch.order.md) | `action_order` | UserError | The vendor related to this order is not available at the selected date. |
| [`lunch.order`](entities/lunch.order.md) | `action_reorder` | UserError | The vendor related to this order is not available today. |
| [`lunch.product`](entities/lunch.product.md) | `_check_active_categories` | UserError | The following product categories are archived. You should either unarchive the categories or change the category of the product. %s |
| [`lunch.product`](entities/lunch.product.md) | `_check_active_suppliers` | UserError | The following suppliers are archived. You should either unarchive the suppliers or change the supplier of the product. %s |
| [`lunch.supplier`](entities/lunch.supplier.md) | `_send_auto_email` | UserError | Cannot send an email to this supplier! |
| [`mail.activity.plan`](entities/mail.activity.plan.md) | `_check_compatibility_with_model` | UserError | Plan %(plan_names)s cannot use a department as it is used only for some HR plans. |
| [`mail.activity.plan`](entities/mail.activity.plan.md) | `_check_compatibility_with_model` | UserError | Plan activities %(template_names)s cannot use coach, manager or employee responsible as it is used only for employee plans. |
| [`mail.activity.plan.template`](entities/mail.activity.plan.template.md) | `_check_activity_type_res_model` | ValidationError | The activity type "%(activity_type_name)s" is not compatible with the plan "%(plan_name)s" because it is limited to the model "%(activity_type_model)s". |
| [`mail.activity.plan.template`](entities/mail.activity.plan.template.md) | `_check_responsible` | ValidationError | When selecting "Default user" assignment, you must specify a responsible. |
| [`mail.activity.plan.template`](entities/mail.activity.plan.template.md) | `_check_responsible_hr` | ValidationError | Those responsible types are limited to Employee plans. |
| [`mail.activity.plan.template`](entities/mail.activity.plan.template.md) | `_check_responsible_hr_fleet` | ValidationError | Fleet Manager is limited to Employee plans. |
| [`mail.activity.schedule`](entities/mail.activity.schedule.md) | `_check_consistency` | ValidationError | html2plaintext(scheduler.error) |
| [`mail.activity.schedule`](entities/mail.activity.schedule.md) | `_onchange_activity_user_id` | UserError | Selected user '%(user)s' cannot upload documents on model '%(model)s' |
| [`mail.activity.schedule`](entities/mail.activity.schedule.md) | `action_create_calendar_event` | UserError | Scheduling an activity using the calendar is not possible on more than one record. |
| [`mail.activity.type`](entities/mail.activity.type.md) | `write` | UserError | You cannot modify %(activities_names)s target model as they are are required in various apps. |
| [`mail.activity.type`](entities/mail.activity.type.md) | `_unlink_except_todo` | UserError | You cannot delete %(activity_names)s as it is required in various apps. |
| [`mail.activity.type`](entities/mail.activity.type.md) | `action_archive` | UserError | The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted. |
| [`mail.alias`](entities/mail.alias.md) | `_check_alias_domain_id_mc` | ValidationError | We could not create alias %(alias_name)s because domain %(alias_domain_name)s belongs to company %(alias_company_names)s while the owner document belongs to company %(company_name)s. |
| [`mail.alias`](entities/mail.alias.md) | `_check_alias_domain_id_mc` | ValidationError | We could not create alias %(alias_name)s because domain %(alias_domain_name)s belongs to company %(alias_company_names)s while the target document belongs to company %(company_name)s. |
| [`mail.alias`](entities/mail.alias.md) | `_check_alias_is_ascii` | ValidationError | You cannot use anything else than unaccented latin characters in the alias address %(alias_name)s. |
| [`mail.alias`](entities/mail.alias.md) | `_check_alias_defaults` | ValidationError | Invalid expression, it must be a literal python dictionary definition e.g. "{'field': 'value'}" |
| [`mail.alias`](entities/mail.alias.md) | `_check_alias_domain_clash` | ValidationError | Aliases %(alias_names)s is already used as bounce or catchall address. Please choose another alias. |
| [`mail.alias`](entities/mail.alias.md) | `_check_unique` | UserError | f'{msg_begin} {msg_end}' |
| [`mail.alias`](entities/mail.alias.md) | `_check_unique` | UserError | Email aliases %(alias_name)s cannot be used on several records at the same time. Please update records one by one. |
| [`mail.alias`](entities/mail.alias.md) | `_sanitize_allowed_domains` | ValidationError | Value %(allowed_domains)s for `mail.catchall.domain.allowed` cannot be validated. It should be a comma separated list of domains e.g. example.com,example.org. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_bounce_catchall_uniqueness` | ValidationError | Bounce/Catchall '%(matching_alias_name)s' is already used. Choose another alias or change it on the linked model. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_bounce_catchall_uniqueness` | ValidationError | Bounce alias %(bounce)s is already used for another domain with same name. Use another bounce or simply use the other alias domain. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_bounce_catchall_uniqueness` | ValidationError | Catchall alias %(catchall)s is already used for another domain with same name. Use another catchall or simply use the other alias domain. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_bounce_catchall_uniqueness` | ValidationError | Bounce/Catchall '%(matching_alias_name)s' is already used by %(document_name)s. Choose another alias or change it on the other document. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_name` | ValidationError | You cannot assign an empty domain name. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_name` | ValidationError | You cannot use anything else than unaccented latin characters in the domain name %(domain_name)s. |
| [`mail.alias.domain`](entities/mail.alias.domain.md) | `_check_default_from_not_used_by_users` | UserError | A personal mail server is using that address, you can not use it. |
| [`mail.blacklist`](entities/mail.blacklist.md) | `create` | UserError | Invalid email address “%s” |
| [`mail.compose.message`](entities/mail.compose.message.md) | `_action_schedule_message` | UserError | A message can only be scheduled in monocomment mode |
| [`mail.compose.message`](entities/mail.compose.message.md) | `_action_schedule_message` | UserError | A scheduled date is needed to schedule a message |
| [`mail.compose.message`](entities/mail.compose.message.md) | `_action_send_mail_comment` | UserError | No recipient found. |
| [`mail.compose.message`](entities/mail.compose.message.md) | `create_mail_template` | UserError | Template creation from composer requires a valid model. |
| [`mail.compose.message`](entities/mail.compose.message.md) | `_evaluate_res_domain` | ValidationError | Invalid domain “%(domain)s” (type “%(domain_type)s”) |
| [`mail.followers.edit`](entities/mail.followers.edit.md) | `edit_followers` | UserError | No documents found for the selected records. |
| [`mail.followers.edit`](entities/mail.followers.edit.md) | `edit_followers` | UserError | Unable to post message, please configure the sender's email address. |
| [`mail.group`](entities/mail.group.md) | `_check_moderator_email` | ValidationError | Moderators must have an email address. |
| [`mail.group`](entities/mail.group.md) | `_check_moderation_notify` | ValidationError | The notification message is missing. |
| [`mail.group`](entities/mail.group.md) | `_check_moderation_guidelines` | ValidationError | The guidelines description is missing. |
| [`mail.group`](entities/mail.group.md) | `_check_moderator_existence` | ValidationError | Moderated group must have moderators. |
| [`mail.group`](entities/mail.group.md) | `_check_access_mode` | ValidationError | The "Authorized Group" is missing. |
| [`mail.group`](entities/mail.group.md) | `action_send_guidelines` | UserError | Only an administrator or a moderator can send guidelines to group members. |
| [`mail.group`](entities/mail.group.md) | `action_send_guidelines` | UserError | The guidelines description is empty. |
| [`mail.group`](entities/mail.group.md) | `action_send_guidelines` | UserError | You can not send guidelines for a closed group. |
| [`mail.group`](entities/mail.group.md) | `action_send_guidelines` | UserError | Template "mail_group.mail_template_guidelines" was not found. No email has been sent. Please contact an administrator to fix this issue. |
| [`mail.group`](entities/mail.group.md) | `_notify_members` | UserError | The group of the message do not match. |
| [`mail.group`](entities/mail.group.md) | `action_join` | UserError | You can not join a closed group. |
| [`mail.group`](entities/mail.group.md) | `_join_group` | ValidationError | The partner can not be found. |
| [`mail.group`](entities/mail.group.md) | `_generate_action_token` | UserError | Email %s is invalid |
| [`mail.group.message`](entities/mail.group.message.md) | `_constrains_mail_message_id` | AccessError | Group message can only be linked to mail group. Current model is %s. |
| [`mail.group.message`](entities/mail.group.message.md) | `_constrains_mail_message_id` | AccessError | The record of the message should be the group. |
| [`mail.group.message`](entities/mail.group.message.md) | `_create_moderation_rule` | UserError | The email "%s" is not valid. |
| [`mail.group.message`](entities/mail.group.message.md) | `_assert_moderable` | UserError | Those messages can not be moderated: %s. |
| [`mail.group.message`](entities/mail.group.message.md) | `_assert_moderable` | UserError | This message can not be moderated |
| [`mail.group.moderation`](entities/mail.group.moderation.md) | `create` | UserError | Invalid email address “%s” |
| [`mail.group.moderation`](entities/mail.group.moderation.md) | `write` | UserError | Invalid email address “%s” |
| [`mail.guest`](entities/mail.guest.md) | `_update_name` | UserError | Guest's name cannot be empty. |
| [`mail.guest`](entities/mail.guest.md) | `_update_name` | UserError | Guest's name is too long. |
| [`mail.mail`](entities/mail.mail.md) | `_check_mail_server_id` | ValidationError | You may not create a message using another user's mail server. |
| [`mail.mail`](entities/mail.mail.md) | `_send` | UserError | Unauthorized server for some of the sending mails. |
| [`mail.message`](entities/mail.message.md) | `write` | AccessError | Only administrators can modify 'model' and 'res_id' fields. |
| [`mail.message`](entities/mail.message.md) | `export_data` | AccessError | Only administrators are allowed to export mail message |
| [`mail.message`](entities/mail.message.md) | `_except_audit_log` | UserError | You cannot remove parts of a restricted audit trail. Archive the record instead. |
| [`mail.notification`](entities/mail.notification.md) | `write` | AccessError | Can not update the message or recipient of a notification. |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_check_access_right_dynamic_template` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_render_template_qweb` | UserError | Failed to render QWeb template for %(template_label)s Target Model: %(model_name)s Language context: %(lang_context)s Error: %(error_details)s  Template Source Snippet: %(template_src)s |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_render_template_qweb` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_render_template_qweb_view` | UserError | Failed to render template: %(view_ref)s |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_render_template_inline_template` | AccessError | Only members of %(group_name)s group are allowed to edit templates containing sensible placeholders |
| [`mail.render.mixin`](entities/mail.render.mixin.md) | `_render_template_inline_template` | UserError | Failed to render inline_template template: %(template_txt)s Error details: %(error)s |
| [`mail.scheduled.message`](entities/mail.scheduled.message.md) | `_check_model` | ValidationError | A message cannot be scheduled on a model that does not have a mail thread. |
| [`mail.scheduled.message`](entities/mail.scheduled.message.md) | `_check_scheduled_date` | ValidationError | A Scheduled Message cannot be scheduled in the past |
| [`mail.scheduled.message`](entities/mail.scheduled.message.md) | `write` | UserError | You are not allowed to change the target record of a scheduled message. |
| [`mail.scheduled.message`](entities/mail.scheduled.message.md) | `post_message` | UserError | You are not allowed to send this scheduled message |
| [`mail.template`](entities/mail.template.md) | `_check_abstract_models` | ValidationError | You may not define a template on an abstract model: %s |
| [`mail.template`](entities/mail.template.md) | `_check_can_be_rendered` | ValidationError | Oops! We couldn't save your template due to an issue.  Error: %(error_details)s  Correct it and try again. |
| [`mail.template`](entities/mail.template.md) | `_generate_template_attachments` | UserError | Unsupported report type %s found. |
| [`mail.template`](entities/mail.template.md) | `_unlink_except_master_mail_template` | UserError | You cannot delete this mail template, it is used in the invoice sending flow. |
| [`mail.thread`](entities/mail.thread.md) | `_search_message_partner_ids` | AccessError | Portal users can only filter threads by themselves as followers. |
| [`mail.thread`](entities/mail.thread.md) | `_check_can_update_message_content` | UserError | Messages with tracking values cannot be modified |
| [`mail.thread`](entities/mail.thread.md) | `_check_can_update_message_content` | UserError | Only messages type comment can have their content updated |
| [`mail.thread`](entities/mail.thread.md) | `notify_cancel_by_type` | AccessError | Access Denied |
| [`mail.thread.blacklist`](entities/mail.thread.blacklist.md) | `_assert_primary_email` | UserError | Invalid primary email field on model %s |
| [`mail.thread.blacklist`](entities/mail.thread.blacklist.md) | `_assert_primary_email` | UserError | Invalid primary email field on model %s |
| [`mail.thread.blacklist`](entities/mail.thread.blacklist.md) | `mail_action_blacklist_remove` | AccessError | You do not have the access right to unblacklist emails. Please contact your administrator. |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | `_search_phone_mobile_search` | UserError | Missing definition of phone fields. |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | `_search_phone_mobile_search` | UserError | Please enter at least 3 characters when searching a Phone number. |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | `_assert_phone_field` | UserError | Invalid primary phone field on model %s |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | `_assert_phone_field` | UserError | Invalid primary phone field on model %s |
| [`mail.thread.phone`](entities/mail.thread.phone.md) | `phone_action_blacklist_remove` | AccessError | You do not have the access right to unblacklist phone numbers. Please contact your administrator. |
| [`mail.tracking.duration.mixin`](entities/mail.tracking.duration.mixin.md) | `_search_is_rotting` | UserError | Model configuration does not support the rotting feature |
| [`mailing.contact`](entities/mailing.contact.md) | `create` | UserError | You should give either list_ids, either subscription_ids to create new contacts. |
| [`mailing.filter`](entities/mailing.filter.md) | `_check_mailing_domain` | ValidationError | The filter domain is not valid for this recipients. |
| [`mailing.list`](entities/mailing.list.md) | `write` | UserError | At least one of the mailing list you are trying to archive is used in an ongoing mailing campaign. |
| [`mailing.list.merge`](entities/mailing.list.merge.md) | `default_get` | UserError | You can only apply this action from Mailing Lists. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `_check_mailing_filter_model` | ValidationError | The saved filter targets different recipients and is incompatible with this mailing. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `write` | ValidationError | A campaign should be set when A/B test is enabled |
| [`mailing.mailing`](entities/mailing.mailing.md) | `action_send_winner_mailing` | ValidationError | No mailing for this A/B testing campaign has been sent yet! Send one first and try again later. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `_action_send_mail` | UserError | There are no recipients selected. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `_check_mailing_domain` | ValidationError | Card Campaign Mailing should target model %(model_name)s |
| [`mailing.mailing`](entities/mailing.mailing.md) | `action_put_in_queue` | UserError | You should update all the cards for %(mailing)s before scheduling a mailing. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `action_send_mail` | UserError | You should update all the cards for %(mailing)s before scheduling a mailing. |
| [`mailing.mailing`](entities/mailing.mailing.md) | `_get_seen_list_sms` | UserError | Unsupported %s for mass SMS |
| [`maintenance.equipment.category`](entities/maintenance.equipment.category.md) | `_unlink_except_contains_maintenance_requests` | UserError | You can’t delete an equipment category if some equipment or maintenance requests are linked to it. |
| [`maintenance.request`](entities/maintenance.request.md) | `_check_schedule_end` | ValidationError | End date cannot be earlier than start date. |
| [`maintenance.request`](entities/maintenance.request.md) | `_check_repeat_interval` | ValidationError | The repeat interval cannot be less than 1. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `open_microsoft_outlook_uri` | AccessError | Only the administrator can link an Outlook mail server. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `open_microsoft_outlook_uri` | UserError | Please enter a valid email address. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `open_microsoft_outlook_uri` | UserError | Please configure your Outlook credentials. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `open_microsoft_outlook_uri` | UserError | Please configure your Outlook credentials. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `open_microsoft_outlook_uri` | UserError | Oops, we could not authenticate you. Please try again later. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `_fetch_outlook_token` | UserError | An error occurred when fetching the access token. %s |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `_fetch_outlook_access_token_iap` | UserError | Oops, we could not authenticate you. Please try again later. |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `_raise_iap_error` | UserError | get_iap_error_message(self.env, error) |
| [`microsoft.outlook.mixin`](entities/microsoft.outlook.mixin.md) | `_generate_outlook_oauth2_string` | UserError | Please connect with your Outlook account before using it. |
| [`mrp.account.wip.accounting`](entities/mrp.account.wip.accounting.md) | `confirm` | UserError | Please make sure the total credit amount equals the total debit amount. |
| [`mrp.account.wip.accounting`](entities/mrp.account.wip.accounting.md) | `confirm` | UserError | Reversal date must be after the posting date. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_cycle` | ValidationError | The current configuration is incorrect because it would create a cycle between these products: %s. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | ValidationError | You cannot use the 'Apply on Variant' functionality and simultaneously create a BoM for a specific variant. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | ValidationError | The attribute value %(attribute)s set on product %(product)s does not match the BoM product %(bom_product)s. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | ValidationError | By-product %s should not be the same as BoM product. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | ValidationError | By-products cost shares must be positive. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | ValidationError | The total cost share for a BoM's by-products cannot exceed 100. |
| [`mrp.bom`](entities/mrp.bom.md) | `name_create` | UserError | You cannot create a new Bill of Material from here. |
| [`mrp.bom`](entities/mrp.bom.md) | `check_kit_has_not_orderpoint` | ValidationError | You can not create a kit-type bill of materials for products that have at least one reordering rule. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_valid_batch_size` | ValidationError | The batch size must be positive! |
| [`mrp.bom`](entities/mrp.bom.md) | `_unlink_except_running_mo` | UserError | You can not delete a Bill of Material with running manufacturing orders. Please close or cancel it first. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_subcontracting_no_operation` | ValidationError | You can not set a Bill of Material with operations or by-product line as subcontracting. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | UserError | Components cost share have to be positive or equals to zero. |
| [`mrp.bom`](entities/mrp.bom.md) | `_check_bom_lines` | UserError | The total cost share for a BoM's component have to be 100 |
| [`mrp.bom`](entities/mrp.bom.md) | `_ensure_bom_is_free` | UserError | As long as there are some sale order lines that must be delivered/invoiced and are related to these bills of materials, you can not remove them. The error concerns these products: %s |
| [`mrp.consumption.warning`](entities/mrp.consumption.warning.md) | `action_set_qty` | UserError | Values cannot be set and validated because a Lot/Serial Number needs to be specified for a tracked product that is having its consumed amount increased:%(products)s |
| [`mrp.production`](entities/mrp.production.md) | `_check_byproducts` | ValidationError | By-products cost shares must be positive. |
| [`mrp.production`](entities/mrp.production.md) | `_check_byproducts` | ValidationError | The total cost share for a manufacturing order's by-products cannot exceed 100. |
| [`mrp.production`](entities/mrp.production.md) | `_check_lot_producing_ids` | UserError | You cannot set more than 1 lot |
| [`mrp.production`](entities/mrp.production.md) | `write` | UserError | You cannot move a manufacturing order once it is cancelled or done. |
| [`mrp.production`](entities/mrp.production.md) | `_unlink_if_not_done` | UserError | You cannot delete a manufacturing order that is already done. |
| [`mrp.production`](entities/mrp.production.md) | `_get_moves_finished_values` | UserError | You cannot have %s  as the finished product and in the Byproducts |
| [`mrp.production`](entities/mrp.production.md) | `_unlink_except_done` | UserError | Cannot delete a manufacturing order in done state. |
| [`mrp.production`](entities/mrp.production.md) | `_unlink_except_done` | UserError | %s cannot be deleted. Try to cancel them before. |
| [`mrp.production`](entities/mrp.production.md) | `_prepare_stock_lot_values` | UserError | Please set the first Serial Number or a default sequence |
| [`mrp.production`](entities/mrp.production.md) | `action_generate_serial` | UserError | You cannot set more than 1 lot per product |
| [`mrp.production`](entities/mrp.production.md) | `button_unplan` | UserError | Some work orders are already done, so you cannot unplan this manufacturing order.  It’d be a shame to waste all that progress, right? |
| [`mrp.production`](entities/mrp.production.md) | `button_unplan` | UserError | Some work orders have already started, so you cannot unplan this manufacturing order.  It’d be a shame to waste all that progress, right? |
| [`mrp.production`](entities/mrp.production.md) | `action_cancel` | UserError | You cannot cancel a manufacturing order that is already done. |
| [`mrp.production`](entities/mrp.production.md) | `_split_productions` | UserError | Unable to split with more than the quantity to produce. |
| [`mrp.production`](entities/mrp.production.md) | `pre_button_mark_done` | UserError | You need to generate Lot/Serial Number(s) to mark as done some productions |
| [`mrp.production`](entities/mrp.production.md) | `_check_sn_uniqueness` | UserError | Serial number(s) for product %(product_name)s already produced |
| [`mrp.production`](entities/mrp.production.md) | `_check_sn_uniqueness` | UserError | sn_error_msg[sn_id] |
| [`mrp.production`](entities/mrp.production.md) | `_check_sn_uniqueness` | UserError | The serial number %(number)s used for byproduct %(product_name)s has already been produced |
| [`mrp.production`](entities/mrp.production.md) | `_check_sn_uniqueness` | UserError | message |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | Only manufacturing orders in either a draft or confirmed state can be %s. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | Only manufacturing orders with a Bill of Materials can be %s. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | You need at least two production orders to merge them. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing orders of identical products with same BoM. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing orders with no additional components or by-products. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing with the same state. |
| [`mrp.production`](entities/mrp.production.md) | `_pre_action_split_merge_hook` | UserError | You can only merge manufacturing with the same operation type |
| [`mrp.production`](entities/mrp.production.md) | `button_unbuild` | UserError | You can't unbuild a subcontracted Manufacturing Order. |
| [`mrp.production`](entities/mrp.production.md) | `write` | AccessError | You cannot write on fields %s in mrp.production. |
| [`mrp.production`](entities/mrp.production.md) | `action_merge` | ValidationError | Subcontracted manufacturing orders cannot be merged. |
| [`mrp.production`](entities/mrp.production.md) | `action_split_subcontracting` | UserError | Please set a lot/serial for the currently opened subcontracting MO first. |
| [`mrp.production`](entities/mrp.production.md) | `action_split_subcontracting` | UserError | The subcontracted goods have already been received. |
| [`mrp.production`](entities/mrp.production.md) | `_validate_analytic_distribution` | ValidationError | The Project linked to the Manufacturing Order is missing a mandatory distribution for the analytic plan(s) %(missing_plan_names)s. |
| [`mrp.production.serials`](entities/mrp.production.serials.md) | `_parse_serial_numbers` | UserError | There is no serial numbers to apply. |
| [`mrp.production.serials`](entities/mrp.production.serials.md) | `_parse_serial_numbers` | UserError | No valid serial numbers provided. |
| [`mrp.routing.workcenter`](entities/mrp.routing.workcenter.md) | `_check_no_cyclic_dependencies` | ValidationError | You cannot create cyclic dependency. |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | `_unlink_except_done` | UserError | You cannot delete an unbuild order if the state is 'Done'. |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | `action_unbuild` | UserError | You should provide a lot number for the final product. |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | `action_unbuild` | UserError | You cannot unbuild a undone manufacturing order. |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | `action_unbuild` | UserError | error_message |
| [`mrp.unbuild`](entities/mrp.unbuild.md) | `action_unbuild` | UserError | error_message |
| [`mrp.workcenter`](entities/mrp.workcenter.md) | `_check_alternative_workcenter` | ValidationError | Workcenter %s cannot be an alternative of itself. |
| [`mrp.workcenter`](entities/mrp.workcenter.md) | `unblock` | UserError | It has already been unblocked. |
| [`mrp.workcenter.productivity`](entities/mrp.workcenter.productivity.md) | `_check_open_time_ids` | ValidationError | The Workorder (%s) cannot be started twice! |
| [`mrp.workcenter.productivity`](entities/mrp.workcenter.productivity.md) | `_close` | UserError | You need to define at least one unactive productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_set_dates` | UserError | It is not possible to unplan one single Work Order. You should unplan the Manufacturing Order instead in order to unplan all the linked operations. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_check_no_cyclic_dependencies` | ValidationError | You cannot create cyclic dependency. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_onchange_date_finished` | UserError | It is not possible to unplan one single Work Order. You should unplan the Manufacturing Order instead in order to unplan all the linked operations. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `write` | UserError | You cannot link this work order to another manufacturing order. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `write` | UserError | You cannot change the quantity produced of a work order that is in done or cancel state. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `write` | UserError | The planned end date of the work order cannot be prior to the planned start date, please correct this to save the work order. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `write` | UserError | The quantity produced must be positive. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `write` | UserError | You cannot change the workcenter of a work order that is done. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_plan_workorder` | UserError | Impossible to plan the workorder. Please check the workcenter availabilities. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_plan_workorder` | UserError | There is no defined calendar on workcenter %s. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `button_start` | UserError | Please unblock the work center to start the work order. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `button_start` | UserError | You cannot start a work order that is already done or cancelled |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_prepare_timeline_vals` | UserError | You need to define at least one productivity loss in the category 'Productivity'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `_prepare_timeline_vals` | UserError | You need to define at least one productivity loss in the category 'Performance'. Create one from the Manufacturing app, menu: Configuration / Productivity Losses. |
| [`mrp.workorder`](entities/mrp.workorder.md) | `action_mark_as_done` | UserError | Please unblock the work center to validate the work order |
| [`myinvois.consolidate.invoice.wizard`](entities/myinvois.consolidate.invoice.wizard.md) | `_get_myinvois_document_vals` | ValidationError | Invalid Operation. No order to consolidate. |
| [`myinvois.document`](entities/myinvois.document.md) | `_unlink_check` | UserError | You cannot delete a document that is active on MyInvois. You must cancel it first. |
| [`myinvois.document`](entities/myinvois.document.md) | `action_submit_to_myinvois` | UserError | You cannot send this document to MyInvois because the related invoice(s) %s are in draft or canceled state. |
| [`myinvois.document`](entities/myinvois.document.md) | `action_generate_xml_file` | UserError | Error when generating the documents' files:  - %(errors)s |
| [`myinvois.document`](entities/myinvois.document.md) | `_myinvois_get_proxy_user` | UserError | Please register for the E-Invoicing service in the settings first. |
| [`myinvois.document`](entities/myinvois.document.md) | `_submit_to_myinvois` | UserError | errors[self.id]['plain_text_error'] |
| [`myinvois.document`](entities/myinvois.document.md) | `_myinvois_check_can_update_status` | UserError | It has been more than 72h since the document validation, you can no longer cancel it. Instead, you should issue a debit or credit note. |
| [`myinvois.document`](entities/myinvois.document.md) | `_myinvois_check_can_update_status` | UserError | You can only change the state of a document in the valid or rejected states. |
| [`myinvois.document`](entities/myinvois.document.md) | `_myinvois_single_status_update` | UserError | self._myinvois_map_error(result['error']) |
| [`myinvois.document.status.update.wizard`](entities/myinvois.document.status.update.wizard.md) | `button_request_update` | UserError | You must provide a reason for updating the document. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_nemhandel_registration_sms` | UserError | Cannot register a user with a %s application |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_nemhandel_registration_sms` | ValidationError | Please enter a phone number to verify your application. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_nemhandel_registration_sms` | ValidationError | Please enter a primary contact email to verify your application. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_nemhandel_registration_sms` | RedirectWarning | Please fill in your company's VAT |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_update_nemhandel_user_data` | ValidationError | Contact email and phone number are required. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_check_nemhandel_verification_code` | ValidationError | Please first verify your phone number by clicking on 'Send a registration code by SMS'. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_check_nemhandel_verification_code` | ValidationError | The verification code should contain six digits. |
| [`nemhandel.registration`](entities/nemhandel.registration.md) | `button_check_nemhandel_verification_code` | UserError | errors.get(error_code) or _('Connection error, please try again later.') |
| [`onboarding.onboarding.step`](entities/onboarding.onboarding.step.md) | `check_step_on_onboarding_has_action` | ValidationError | An "Opening Action" is required for the following steps to be linked to an onboarding panel: %(step_titles)s |
| [`payment.capture.wizard`](entities/payment.capture.wizard.md) | `_check_amount_to_capture_within_boundaries` | ValidationError | The amount to capture must be positive and cannot be superior to %s. |
| [`payment.capture.wizard`](entities/payment.capture.wizard.md) | `_check_amount_to_capture_within_boundaries` | ValidationError | Some of the transactions you intend to capture can only be captured in full. Handle the transactions individually to capture a partial amount. |
| [`payment.method`](entities/payment.method.md) | `_check_manual_capture_supported_by_providers` | ValidationError | The following payment methods cannot be enabled because their payment provider has manual capture activated: %s |
| [`payment.method`](entities/payment.method.md) | `write` | UserError | This payment method needs a partner in crime; you should enable a payment provider supporting this method first. |
| [`payment.method`](entities/payment.method.md) | `_unlink_if_not_default_payment_method` | UserError | You cannot delete the default payment method. |
| [`payment.provider`](entities/payment.provider.md) | `_onchange_company_block_if_existing_transactions` | UserError | You cannot change the company of a payment provider with existing transactions. |
| [`payment.provider`](entities/payment.provider.md) | `_check_manual_capture_supported_by_payment_methods` | ValidationError | The following payment methods must be disabled in order to enable manual capture: %s |
| [`payment.provider`](entities/payment.provider.md) | `_check_required_if_provider` | ValidationError | The following fields must be filled: %s |
| [`payment.provider`](entities/payment.provider.md) | `_unlink_except_master_data` | UserError | You cannot delete the payment provider %s; disable it or uninstall it instead. |
| [`payment.provider`](entities/payment.provider.md) | `action_toggle_is_published` | UserError | You cannot publish a disabled provider. |
| [`payment.provider`](entities/payment.provider.md) | `_send_api_request` | ValidationError | Could not establish the connection to the payment provider. |
| [`payment.provider`](entities/payment.provider.md) | `_send_api_request` | ValidationError | The payment provider rejected the request. %s |
| [`payment.provider`](entities/payment.provider.md) | `_parse_proxy_response` | ValidationError | The payment provider rejected the request. %s |
| [`payment.provider`](entities/payment.provider.md) | `_remove_provider` | UserError | You cannot uninstall this module as payments using this payment method already exist. |
| [`payment.provider`](entities/payment.provider.md) | `_limit_available_currency_ids` | ValidationError | Only one currency can be selected by AsiaPay account. |
| [`payment.provider`](entities/payment.provider.md) | `_limit_available_currency_ids` | ValidationError | AsiaPay does not support the following currencies: %(currencies)s. |
| [`payment.provider`](entities/payment.provider.md) | `_limit_available_currency_ids` | ValidationError | Only one currency can be selected by Authorize.Net account. |
| [`payment.provider`](entities/payment.provider.md) | `action_update_merchant_details` | UserError | This action cannot be performed while the provider is disabled. |
| [`payment.provider`](entities/payment.provider.md) | `action_update_merchant_details` | UserError | Failed to authenticate. %s |
| [`payment.provider`](entities/payment.provider.md) | `action_update_merchant_details` | UserError | Could not fetch merchant details: %s |
| [`payment.provider`](entities/payment.provider.md) | `_check_provider_state` | UserError | Demo providers should never be enabled. |
| [`payment.provider`](entities/payment.provider.md) | `_check_currency_is_supported` | ValidationError | ECPay only supports TWD. |
| [`payment.provider`](entities/payment.provider.md) | `_parse_response_content` | ValidationError | The payment provider rejected the request. %s |
| [`payment.provider`](entities/payment.provider.md) | `_check_currency_is_supported` | ValidationError | Only the currency %s is available for this account. |
| [`payment.provider`](entities/payment.provider.md) | `_check_mercado_pago_credentials_are_set_before_enabling` | ValidationError | Mercado Pago credentials are missing. Click the "Connect" button to set up your account. |
| [`payment.provider`](entities/payment.provider.md) | `_check_mercado_pago_credentials_are_set_before_allowing_tokenization` | ValidationError | Connect your account before enabling tokenization. |
| [`payment.provider`](entities/payment.provider.md) | `action_start_onboarding` | RedirectWarning | Mercado Pago is not available in your country; please use another payment provider. |
| [`payment.provider`](entities/payment.provider.md) | `action_start_onboarding` | ValidationError | Set the account country before connecting the account. |
| [`payment.provider`](entities/payment.provider.md) | `_check_available_country_currency_ids` | ValidationError | Only one currency can be selected per Paymob account. |
| [`payment.provider`](entities/payment.provider.md) | `_check_available_country_currency_ids` | ValidationError | Only currencies supported by Paymob can be selected. |
| [`payment.provider`](entities/payment.provider.md) | `_paymob_fetch_access_token` | ValidationError | Could not generate a new access token. |
| [`payment.provider`](entities/payment.provider.md) | `action_paypal_create_webhook` | UserError | 'PayPal: ' + _('You must have an HTTPS connection to generate a webhook.') |
| [`payment.provider`](entities/payment.provider.md) | `_paypal_fetch_access_token` | ValidationError | Could not generate a new access token. |
| [`payment.provider`](entities/payment.provider.md) | `_check_payu_credentials_are_set_before_enabling` | ValidationError | PayU credentials are missing. Click the "Connect" button to set up your account. |
| [`payment.provider`](entities/payment.provider.md) | `action_start_onboarding` | RedirectWarning | PayU is not available in your country; please use another payment provider. |
| [`payment.provider`](entities/payment.provider.md) | `_check_razorpay_credentials_are_set_before_enabling` | ValidationError | Razorpay credentials are missing. Click the "Connect" button to set up your account. |
| [`payment.provider`](entities/payment.provider.md) | `action_start_onboarding` | RedirectWarning | Razorpay is not available in your country; please use another payment provider. |
| [`payment.provider`](entities/payment.provider.md) | `_check_state_of_connected_account_is_never_test` | ValidationError | You cannot set the provider to Test Mode while it is linked with your Stripe account. |
| [`payment.provider`](entities/payment.provider.md) | `_check_onboarding_of_enabled_provider_is_completed` | ValidationError | You cannot set the provider state to Enabled until your onboarding to Stripe is completed. |
| [`payment.provider`](entities/payment.provider.md) | `action_start_onboarding` | RedirectWarning | Stripe Connect is not available in your country, please use another payment provider. |
| [`payment.provider`](entities/payment.provider.md) | `action_stripe_verify_apple_pay_domain` | UserError | Please use live credentials to enable Apple Pay. |
| [`payment.provider`](entities/payment.provider.md) | `_check_available_currency_ids_only_contains_supported_currencies` | ValidationError | Currencies other than KRW are not supported. |
| [`payment.refund.wizard`](entities/payment.refund.wizard.md) | `_check_amount_to_refund_within_boundaries` | ValidationError | The amount to be refunded must be positive and cannot be superior to %s. |
| [`payment.token`](entities/payment.token.md) | `write` | UserError | You can't unarchive tokens linked to inactive payment methods or disabled providers. |
| [`payment.token`](entities/payment.token.md) | `_check_partner_is_never_public` | ValidationError | No token can be assigned to the public partner. |
| [`payment.token`](entities/payment.token.md) | `_stripe_sca_migrate_customer` | ValidationError | Unable to convert payment token to new API. |
| [`payment.transaction`](entities/payment.transaction.md) | `_check_state_authorized_supported` | ValidationError | Transaction authorization is not supported by the following payment providers: %s |
| [`payment.transaction`](entities/payment.transaction.md) | `_check_token_is_active` | ValidationError | Creating a transaction from an archived token is forbidden. |
| [`payment.transaction`](entities/payment.transaction.md) | `action_void` | ValidationError | Only authorized transactions can be voided. |
| [`payment.transaction`](entities/payment.transaction.md) | `action_refund` | ValidationError | Only confirmed transactions can be refunded. |
| [`payment.transaction`](entities/payment.transaction.md) | `_ensure_provider_is_not_disabled` | UserError | Making a request to the provider is not possible because the provider is disabled. |
| [`payment.transaction`](entities/payment.transaction.md) | `_apply_updates` | ValidationError | Received data with missing success code. |
| [`payment.transaction`](entities/payment.transaction.md) | `_get_specific_rendering_values` | UserError | 'Nuvei: ' + _('%(payment_method)s requires both a first and last name.', payment_method=self.payment_method_id.name) |
| [`payment.transaction`](entities/payment.transaction.md) | `_validate_phone_number` | ValidationError | The phone number is missing. |
| [`payment.transaction`](entities/payment.transaction.md) | `_validate_phone_number` | ValidationError | The phone number is invalid. |
| [`payment.transaction`](entities/payment.transaction.md) | `_send_void_request` | UserError | Transactions processed by Razorpay can't be manually voided from the system. |
| [`payment.transaction`](entities/payment.transaction.md) | `_razorpay_create_refund_tx_from_payment_data` | ValidationError | Received incomplete refund data. |
| [`payment.transaction`](entities/payment.transaction.md) | `_process_pos_online_payment` | ValidationError | The payment transaction (%d) has a negative amount. |
| [`payment.transaction`](entities/payment.transaction.md) | `_process_pos_online_payment` | ValidationError | The POS online payment (tx.id=%d) could not be saved correctly |
| [`payment.transaction`](entities/payment.transaction.md) | `_process_pos_online_payment` | ValidationError | The POS online payment (tx.id=%d) could not be saved correctly because the online payment method could not be found |
| [`pdp.registration`](entities/pdp.registration.md) | `_ensure_mandatory_fields` | ValidationError | The contact email is required. |
| [`pdp.registration`](entities/pdp.registration.md) | `button_trigger_authentication` | ValidationError | Invalid email address '%s' |
| [`pdp.registration`](entities/pdp.registration.md) | `button_trigger_authentication` | UserError | error |
| [`pdp.registration`](entities/pdp.registration.md) | `button_trigger_authentication` | UserError | Something wrong happened. |
| [`pdp.registration`](entities/pdp.registration.md) | `button_open_authentication_link` | UserError | error |
| [`pdp.registration`](entities/pdp.registration.md) | `button_open_authentication_link` | UserError | Something wrong happened. |
| [`pdp.registration`](entities/pdp.registration.md) | `button_register_pdp_participant` | UserError | Cannot register a user with a '%s' application |
| [`pdp.registration`](entities/pdp.registration.md) | `button_register_pdp_participant` | UserError | There is a connection to Peppol (non-PA) already |
| [`pdp.registration`](entities/pdp.registration.md) | `button_register_pdp_participant` | UserError | The Identifier is not valid. The expected format is: SIREN, SIREN_SIRET, SIREN_SIRET_CodeRoutage or SIREN_SuffixeAdressage |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `_compute_currency_id` | UserError | The EUR currency is missing. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `_compute_available_statuses` | UserError | All journal entries must either be purchase or sale documents. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Please select a Status. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Some of the journal entries were not sent to the Approved Platform yet: %s |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | To refuse an invoice please select a Reason Code. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | To refuse an invoice please enter a Note. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | To suspend an invoice please select a Reason Code. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | To suspend an invoice please enter a Note. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Some of the journal entries are not cancelled: %s |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Some of the journal entries are not posted: %s |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Only journal entries in currency EUR are supported. |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Some of the journal entries are without tax: %s |
| [`pdp.response.wizard`](entities/pdp.response.wizard.md) | `button_send` | UserError | Some of the journal entries have no payments to send: %s |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_mandatory_fields` | ValidationError | Please select a country for your company. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_mandatory_fields` | ValidationError | Contact email and phone number are required. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_mandatory_fields` | ValidationError | Peppol Address should be provided. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_mandatory_fields` | ValidationError | Peppol ID should be different from main company. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_mandatory_fields` | ValidationError | Cannot register a user with a %s application |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | Could not connect to Proxy Server. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | Your identifier is invalid. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | The database you are trying to connect to is not suitable for Peppol. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | You need to authenticate to continue. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | Selected authentication method is not available. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | Your identifier you entered is invalid for Peppol. |
| [`peppol.registration`](entities/peppol.registration.md) | `_ensure_can_connect` | UserError | Your identifier does not have a valid format.%s |
| [`peppol.registration`](entities/peppol.registration.md) | `button_register_peppol_participant` | UserError | A connection to '%s' already exists. |
| [`phone.blacklist`](entities/phone.blacklist.md) | `create` | UserError | %(error)s Please correct the number and try again. |
| [`phone.blacklist`](entities/phone.blacklist.md) | `write` | UserError | %(error)s Please correct the number and try again. |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `action_grant_access` | UserError | The partner "%s" already has the portal access. |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `action_revoke_access` | UserError | The partner "%s" has no portal access or is internal. |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `action_invite_again` | UserError | You should first grant the portal access to the partner "%s". |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `_send_email` | UserError | The template "Portal: new user" not found for sending email to the portal user. |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `_assert_user_email_uniqueness` | UserError | The contact "%s" does not have a valid email. |
| [`portal.wizard.user`](entities/portal.wizard.user.md) | `_assert_user_email_uniqueness` | UserError | The contact "%s" has the same email as an existing user |
| [`pos.bill`](entities/pos.bill.md) | `name_create` | UserError | The name of the Coins/Bills must be a number. |
| [`pos.category`](entities/pos.category.md) | `_check_category_recursion` | ValidationError | Error! You cannot create recursive categories. |
| [`pos.category`](entities/pos.category.md) | `_unlink_except_session_open` | UserError | You cannot delete a point of sale category while a session is still opened. |
| [`pos.category`](entities/pos.category.md) | `_check_hour` | ValidationError | The Availability Until must be set between 00:00 and 24:00 |
| [`pos.category`](entities/pos.category.md) | `_check_hour` | ValidationError | The Availability After must be set between 00:00 and 24:00 |
| [`pos.category`](entities/pos.category.md) | `_check_hour` | ValidationError | The Availability Until must be greater than Availability After. |
| [`pos.config`](entities/pos.config.md) | `_check_rounding_method_strategy` | ValidationError | The cash rounding strategy of the point of sale %(pos)s must be: '%(value)s' |
| [`pos.config`](entities/pos.config.md) | `_check_profit_loss_cash_journal` | ValidationError | You need a loss and profit account on your cash journal. |
| [`pos.config`](entities/pos.config.md) | `_check_company_payment` | ValidationError | The payment methods for the point of sale %s must belong to its company. |
| [`pos.config`](entities/pos.config.md) | `_check_currencies` | ValidationError | The default pricelist must be included in the available pricelists. |
| [`pos.config`](entities/pos.config.md) | `_check_currencies` | ValidationError | All available pricelists must be in the same currency as the company or as the Sales Journal set on this point of sale if you use the Accounting application. |
| [`pos.config`](entities/pos.config.md) | `_check_currencies` | ValidationError | The invoice journal must be in the same currency as the Sales Journal or the company currency if that is not set. |
| [`pos.config`](entities/pos.config.md) | `_check_currencies` | ValidationError | All payment methods must be in the same currency as the Sales Journal or the company currency if that is not set. |
| [`pos.config`](entities/pos.config.md) | `_check_payment_method_ids` | ValidationError | You must have at least one payment method configured to launch a session. |
| [`pos.config`](entities/pos.config.md) | `_check_pricelists` | ValidationError | The default pricelist must belong to no company or the company of the point of sale. |
| [`pos.config`](entities/pos.config.md) | `_check_companies` | ValidationError | The selected pricelists must belong to no company or the company of the point of sale. |
| [`pos.config`](entities/pos.config.md) | `_check_company_has_template` | ValidationError | No chart of account configured, go to the "configuration / settings" menu, and install one from the Invoicing tab. |
| [`pos.config`](entities/pos.config.md) | `_check_payment_method_ids_journal` | ValidationError | This cash payment method is already used in another Point of Sale. A new cash payment method should be created for this Point of Sale. |
| [`pos.config`](entities/pos.config.md) | `_check_payment_method_ids_journal` | ValidationError | You cannot use the same journal on multiples cash payment methods. |
| [`pos.config`](entities/pos.config.md) | `_check_trusted_config_ids_currency` | ValidationError | You cannot share open orders with configuration that does not use the same currency. |
| [`pos.config`](entities/pos.config.md) | `_check_header_footer` | AccessError | Only administrators can edit receipt headers and footers |
| [`pos.config`](entities/pos.config.md) | `_check_company_has_fiscal_country` | ValidationError | The company must have a fiscal country set. |
| [`pos.config`](entities/pos.config.md) | `_reset_default_on_vals` | UserError | The default tip product is missing. Please manually specify the tip product. (See Tips field.) |
| [`pos.config`](entities/pos.config.md) | `write` | UserError | Unable to modify this PoS Configuration because you can't modify %s while a session is open. |
| [`pos.config`](entities/pos.config.md) | `open_ui` | UserError | You do not have permission to open a POS session. Please try opening a session with a different user |
| [`pos.config`](entities/pos.config.md) | `_create_journal_and_payment_methods` | UserError | Ensure that there is an existing bank journal. Check if chart of accounts is installed in your company. |
| [`pos.config`](entities/pos.config.md) | `open_ui` | UserError | You have to set a country in your company setting. |
| [`pos.config`](entities/pos.config.md) | `open_ui` | UserError | You have to set a country in your company setting. |
| [`pos.config`](entities/pos.config.md) | `open_ui` | RedirectWarning | msg |
| [`pos.config`](entities/pos.config.md) | `_check_adyen_ask_customer_for_tip` | ValidationError | Please configure a tip product for POS %s to support tipping with Adyen. |
| [`pos.config`](entities/pos.config.md) | `open_ui` | UserError | A discount product is needed to use the Global Discount feature. Go to Point of Sale > Configuration > Settings to set it. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | f'{prefix_error_msg}\n{invalid_reward_products_msg}' |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | Invalid gift card program. More than one reward. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | Invalid gift card program rule. Use 1 point per currency spent. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | Invalid gift card program reward. Use 1 currency per point discount. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | There is no email template on the gift card program and your pos is set to print them. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | There is no print report on the gift card program and your pos is set to print them. |
| [`pos.config`](entities/pos.config.md) | `_check_before_creating_new_session` | UserError | Invalid gift card program. More than one rule. |
| [`pos.config`](entities/pos.config.md) | `_check_online_payment_methods` | ValidationError | A POS config cannot have more than one online payment method. |
| [`pos.config`](entities/pos.config.md) | `_check_online_payment_methods` | ValidationError | To use an online payment method in a POS config, it must have at least one published payment provider supporting the currency of that POS config. |
| [`pos.config`](entities/pos.config.md) | `_check_default_user` | UserError | The Self-Order default user must be a POS user |
| [`pos.config`](entities/pos.config.md) | `_onchange_payment_method_ids` | ValidationError | You cannot add cash payment methods in kiosk mode. |
| [`pos.config`](entities/pos.config.md) | `_check_self_order_online_payment_method_id` | ValidationError | The online payment method used for self-order in a POS config must have at least one published payment provider supporting the currency of that POS config. |
| [`pos.make.invoice`](entities/pos.make.invoice.md) | `action_create_invoices` | UserError | No valid orders were selected. No new invoices could be generated |
| [`pos.make.invoice`](entities/pos.make.invoice.md) | `action_create_invoices` | UserError | The following refund orders can't be part of a consolidated invoice because they refunded invoiced orders. Each refund order should be handled separately.  %s |
| [`pos.make.invoice`](entities/pos.make.invoice.md) | `action_create_invoices` | UserError | Kindly ensure that each order contains a customer. |
| [`pos.make.payment`](entities/pos.make.payment.md) | `check` | UserError | Customer is required for %s payment method. |
| [`pos.order`](entities/pos.order.md) | `_get_valid_session` | UserError | No open session available. Please open a new session to capture the order. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | No invoice journal configured for this POS session. |
| [`pos.order`](entities/pos.order.md) | `_process_payment_lines` | UserError | No cash statement found for this session. Unable to record returned cash. |
| [`pos.order`](entities/pos.order.md) | `_compute_prices` | UserError | You can't: create a pos order from the backend interface, or unset the pricelist, or create a pos.order in a python test with Form tool, or edit the form view in studio if no PoS order exist |
| [`pos.order`](entities/pos.order.md) | `_unlink_except_draft_or_cancel` | UserError | In order to delete a sale, it must be new or cancelled. |
| [`pos.order`](entities/pos.order.md) | `write` | UserError | This order has already been paid. You cannot set it back to draft or edit it. |
| [`pos.order`](entities/pos.order.md) | `write` | UserError | The paid amount is different from the total amount of the order. |
| [`pos.order`](entities/pos.order.md) | `write` | UserError | You cannot change the payment of a printed order. |
| [`pos.order`](entities/pos.order.md) | `action_pos_order_paid` | UserError | Order %s is not fully paid. |
| [`pos.order`](entities/pos.order.md) | `action_pos_order_paid` | UserError | Order %s is not fully paid. |
| [`pos.order`](entities/pos.order.md) | `_generate_pos_order_invoice` | UserError | Some orders are already being invoiced. Please try again later. |
| [`pos.order`](entities/pos.order.md) | `action_pos_order_cancel` | UserError | The order delivery / pickup date is in the future. You cannot cancel it. |
| [`pos.order`](entities/pos.order.md) | `action_pos_order_cancel` | UserError | This order has already been paid. You cannot set it back to draft or edit it. |
| [`pos.order`](entities/pos.order.md) | `sync_from_ui` | ValidationError | You can only refund products from the same order. |
| [`pos.order`](entities/pos.order.md) | `_refund` | UserError | To return product(s), you need to open a session in the POS %s |
| [`pos.order`](entities/pos.order.md) | `action_send_receipt` | UserError | The mail template with xmlid %s has been deleted. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | Please create an invoice for an amount over %s. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | You cannot invoice a refund whose linked order hasn't been invoiced. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | Please invoice the refund as the linked order has been invoiced. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | You cannot mix orders that require TicketBAI with those that don't. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | You cannot consolidate orders with different TicketBAI refund reasons. |
| [`pos.order`](entities/pos.order.md) | `l10n_es_tbai_retry_post` | UserError | error |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | The order needs to be invoiced since its total amount is above %s€. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | You have to specify a refund reason. |
| [`pos.order`](entities/pos.order.md) | `_process_saved_order` | UserError | A partner has to be specified for the selected Veri*Factu Refund Reason. |
| [`pos.order`](entities/pos.order.md) | `_generate_pos_order_invoice` | UserError | The order can not be invoiced. It is waiting to send a Veri*Factu record to the AEAT already. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | With Veri*Factu enabled, POS orders cannot be consolidated into one invoice. |
| [`pos.order`](entities/pos.order.md) | `_compute_previous_order` | UserError | An error occurred when computing the inalterability. Impossible to get the unique previous posted point of sale order. |
| [`pos.order`](entities/pos.order.md) | `write` | UserError | According to the French law, you cannot modify a point of sale order. Forbidden fields: %s. |
| [`pos.order`](entities/pos.order.md) | `write` | UserError | You cannot overwrite the values ensuring the inalterability of the point of sale. |
| [`pos.order`](entities/pos.order.md) | `_unlink_except_pos_so` | UserError | According to French law, you cannot delete a point of sale order. |
| [`pos.order`](entities/pos.order.md) | `download_l10n_jo_edi_pos_computed_xml` | ValidationError | The following errors have to be fixed in order to create an XML: |
| [`pos.order`](entities/pos.order.md) | `_process_order` | UserError | You must invoice a refund for an order that has been submitted to MyInvois. |
| [`pos.order`](entities/pos.order.md) | `_process_order` | UserError | You cannot invoice a refund for an order that has not been submitted to MyInvois yet. |
| [`pos.order`](entities/pos.order.md) | `_generate_pos_order_invoice` | UserError | This order has been included in a consolidated invoice and cannot be invoiced separately. |
| [`pos.order`](entities/pos.order.md) | `_generate_pos_order_invoice` | UserError | You must set the identification information on the commercial partner. |
| [`pos.order`](entities/pos.order.md) | `_generate_pos_order_invoice` | UserError | You must set a TIN number on the commercial partner. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | You cannot create a consolidated invoice for POS orders with different ZATCA refund reasons. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | POS orders from different companies cannot be consolidated into one invoice. |
| [`pos.order`](entities/pos.order.md) | `_prepare_invoice_vals` | UserError | With EcPay enabled, POS orders cannot be consolidated into one invoice. |
| [`pos.order`](entities/pos.order.md) | `action_send_self_order_receipt` | UserError | The mail template with xmlid %s has been deleted. |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order_lines` | UserError | Invalid product attribute |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order_lines` | UserError | Invalid quantity |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order` | UserError | Invalid preset |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order` | UserError | Preset is not available in self-ordering |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order` | UserError | Preset is not available in this configuration |
| [`pos.order`](entities/pos.order.md) | `_check_pos_order` | UserError | The order ID isn't linked to the order UUID. This is a sign of a tampered payload. |
| [`pos.order`](entities/pos.order.md) | `_check_combo_lines` | UserError | Invalid combo line |
| [`pos.order`](entities/pos.order.md) | `_check_combo_lines` | UserError | Invalid combo line |
| [`pos.order.line`](entities/pos.order.line.md) | `get_existing_lots` | UserError | No PoS configuration found |
| [`pos.order.line`](entities/pos.order.line.md) | `_unlink_except_order_state` | UserError | You can only unlink PoS order lines that are related to orders in new or cancelled state. |
| [`pos.order.line`](entities/pos.order.line.md) | `_onchange_qty` | ValidationError | You cannot refund more than the outstanding quantity for this product. |
| [`pos.order.line`](entities/pos.order.line.md) | `_prepare_base_line_for_taxes_computation` | UserError | Please define income account for this product: '%(product)s' (id:%(id)d). |
| [`pos.order.line`](entities/pos.order.line.md) | `write` | UserError | According to the French law, you cannot modify a point of sale order line. Forbidden fields: %s. |
| [`pos.payment`](entities/pos.payment.md) | `_check_amount` | ValidationError | You cannot edit a payment for a posted order. |
| [`pos.payment`](entities/pos.payment.md) | `_check_payment_method_id` | ValidationError | The payment method selected is not allowed in the config of the POS session. |
| [`pos.payment`](entities/pos.payment.md) | `create` | UserError | Cannot create a POS online payment without an accounting payment. |
| [`pos.payment`](entities/pos.payment.md) | `create` | UserError | Cannot create a POS online payment without an accounting payment. |
| [`pos.payment`](entities/pos.payment.md) | `create` | UserError | Cannot create a POS payment with a not online payment method and an online accounting payment. |
| [`pos.payment`](entities/pos.payment.md) | `write` | UserError | Cannot edit a POS online payment essential data. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_onchange_journal_id` | UserError | Only journals of type 'Cash' or 'Bank' could be used with payment methods. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `write` | UserError | Please close and validate the following open PoS Sessions before modifying this payment method. Open sessions: %s |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_payment_method` | ValidationError | At least one bank account must be defined on the journal to allow registering QR code payments with Bank apps. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_payment_method` | ValidationError | You must select a QR-code method to generate QR-codes for this payment method. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_payment_method` | ValidationError | error_msg |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_company_config` | ValidationError | The points of sale for the payment method %s must belong to its company. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_cash_method_single_shop` | ValidationError | Validation Error: You cannot assign the same Cash payment method to multiple POS Shops. Please create a separate Cash payment method for each shop. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `get_qr_code` | UserError | This payment method is not configured to generate QR codes. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `l10n_id_verify_qris_status` | UserError | No QRIS transaction record is found based on this order |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_adyen_terminal_identifier` | ValidationError | Terminal %(terminal)s is already used on payment method %(payment_method)s. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_adyen_terminal_identifier` | ValidationError | Terminal %(terminal)s is already used in company %(company)s on payment method %(payment_method)s. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `proxy_adyen_request` | UserError | Invalid Adyen request |
| [`pos.payment.method`](entities/pos.payment.method.md) | `proxy_adyen_request` | UserError | Invalid Adyen request |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_generate_dpopay_token` | UserError | Unable to retrieve DPO Pay bearer token: check Client ID and Client Secret. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_execute_dpopay_api_request` | UserError | Invalid endpoint |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_special_access` | AccessError | Do not have access to fetch token from Mercado Pago |
| [`pos.payment.method`](entities/pos.payment.method.md) | `force_pdv` | UserError | Unexpected Mercado Pago response: %s |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_find_terminal` | UserError | Please verify your production user token as it was rejected |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_find_terminal` | UserError | The terminal serial number is not registered on Mercado Pago |
| [`pos.payment.method`](entities/pos.payment.method.md) | `mollie_create_payment` | ValidationError | Please set the API key on the Mollie payment provider before making a payment. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_online_payment_providers` | ValidationError | All payment providers configured for an online payment method must use the same currency as the Sales Journal, or the company currency if that is not set, of the POS config. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_pos_config_online_payment` | ValidationError | The %s already has one online payment. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_or_create_online_payment_method` | ValidationError | Could not create an online payment method (company_id=%(company_id)d, pos_config_id=%(pos_config_id)d) |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_pine_labs_terminal` | UserError | This Payment Terminal is only valid for INR Currency |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_qfpay_terminal` | UserError | QFPay is only valid for HKD Currency |
| [`pos.payment.method`](entities/pos.payment.method.md) | `qfpay_sign_request` | UserError | This method can only be used with QFPay payment terminal. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_razorpay_terminal` | UserError | This Payment Terminal is only valid for INR Currency |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_stripe_serial_number` | ValidationError | Terminal %(terminal)s is already used on payment method %(payment_method)s. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_stripe_payment_provider` | UserError | Stripe payment provider for company %s is missing |
| [`pos.payment.method`](entities/pos.payment.method.md) | `stripe_connection_token` | AccessError | Do not have access to fetch token from Stripe |
| [`pos.payment.method`](entities/pos.payment.method.md) | `stripe_payment_intent` | AccessError | Do not have access to fetch token from Stripe |
| [`pos.payment.method`](entities/pos.payment.method.md) | `stripe_refund` | AccessError | Do not have access to refund Stripe payment |
| [`pos.payment.method`](entities/pos.payment.method.md) | `stripe_capture_payment` | AccessError | Do not have access to fetch token from Stripe |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_business_short_code` | ValidationError | validation_error |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_bearer_token` | UserError | Consumer Key and Consumer Secret are required for Safaricom M-Pesa |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_bearer_token` | UserError | Failed to retrieve access token from Safaricom |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_get_bearer_token` | UserError | Failed to retrieve access token from Safaricom |
| [`pos.payment.method`](entities/pos.payment.method.md) | `lipa_na_mpesa_register_urls` | UserError | Could not find base url. Please set up web.base.url to a valid https address |
| [`pos.payment.method`](entities/pos.payment.method.md) | `lipa_na_mpesa_register_urls` | UserError | Failed to register URLs: %s |
| [`pos.payment.method`](entities/pos.payment.method.md) | `lipa_na_mpesa_register_urls` | UserError | Failed to register URLs. Check your credentials and try again. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_bearer_token` | UserError | Unable to retrieve Viva.com Bearer Token: Please verify that the Client ID and Client Secret are correct |
| [`pos.payment.method`](entities/pos.payment.method.md) | `viva_com_send_payment_request` | AccessError | Only 'group_pos_user' are allowed to send a Viva.com payment request |
| [`pos.payment.method`](entities/pos.payment.method.md) | `viva_com_send_refund_request` | AccessError | Only 'group_pos_user' are allowed to send a Viva.com refund request |
| [`pos.payment.method`](entities/pos.payment.method.md) | `viva_com_send_payment_cancel` | AccessError | Only 'group_pos_user' are allowed to cancel a Viva.com payment |
| [`pos.payment.method`](entities/pos.payment.method.md) | `viva_com_get_payment_status` | AccessError | Only 'group_pos_user' are allowed to get the payment status from Viva.com |
| [`pos.payment.method`](entities/pos.payment.method.md) | `write` | UserError | Can't update payment method. Please check the data and update it. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `create` | UserError | Can't create payment method. Please check the data and update it. |
| [`pos.payment.method`](entities/pos.payment.method.md) | `_check_viva_com_credentials` | UserError | It is essential to provide API key for the use of Viva.com |
| [`pos.preset`](entities/pos.preset.md) | `_check_slots` | ValidationError | The start time must be before the end time. |
| [`pos.preset`](entities/pos.preset.md) | `_unlink_except_used_preset` | UserError | You cannot delete a preset that is linked to a POS configuration. |
| [`pos.preset`](entities/pos.preset.md) | `_unlink_except_master_presets` | UserError | You cannot delete the master preset(s). |
| [`pos.printer`](entities/pos.printer.md) | `_constrains_epson_printer_ip` | ValidationError | Epson Printer IP Address cannot be empty. |
| [`pos.session`](entities/pos.session.md) | `delete_opening_control_session` | UserError | You can only cancel a session that is in opening control state and has no orders. |
| [`pos.session`](entities/pos.session.md) | `_check_pos_config` | ValidationError | Another session is already opened for this point of sale. |
| [`pos.session`](entities/pos.session.md) | `_check_start_date` | ValidationError | You cannot create a session starting before: %(lock_date_info)s |
| [`pos.session`](entities/pos.session.md) | `_check_invoices_are_posted` | UserError | You cannot close the POS when invoices are not posted. Invoices: %s |
| [`pos.session`](entities/pos.session.md) | `create` | UserError | You should assign a Point of Sale to your session. |
| [`pos.session`](entities/pos.session.md) | `action_pos_session_closing_control` | UserError | You cannot close the POS while there are still draft orders for the day. |
| [`pos.session`](entities/pos.session.md) | `action_pos_session_closing_control` | UserError | This session is already closed. |
| [`pos.session`](entities/pos.session.md) | `_validate_session` | UserError | This session is already closed. |
| [`pos.session`](entities/pos.session.md) | `_post_statement_difference` | UserError | Please go on the %s journal and define a Loss Account. This account will be used to record cash difference. |
| [`pos.session`](entities/pos.session.md) | `_post_statement_difference` | UserError | Please go on the %s journal and define a Profit Account. This account will be used to record cash difference. |
| [`pos.session`](entities/pos.session.md) | `update_closing_control_state_session` | UserError | This session is already closed. |
| [`pos.session`](entities/pos.session.md) | `post_closing_cash_details` | UserError | There is no cash register in this session. |
| [`pos.session`](entities/pos.session.md) | `get_cash_in_out_list` | AccessError | You don't have the access rights to get the cash in/out list. |
| [`pos.session`](entities/pos.session.md) | `get_closing_control_data` | AccessError | You don't have the access rights to get the point of sale closing control data. |
| [`pos.session`](entities/pos.session.md) | `_create_non_reconciliable_move_lines` | UserError | Unable to close and validate the session. Please set corresponding tax account in each repartition line of the following taxes:  %s |
| [`pos.session`](entities/pos.session.md) | `_get_split_receivable_vals` | UserError | You have enabled the "Identify Customer" option for %(payment_method)s payment method,but the order %(order)s does not contain a customer. |
| [`pos.session`](entities/pos.session.md) | `_check_if_no_draft_orders` | UserError | There are still orders in draft state in the session. Pay or cancel the following orders to validate the session: %s |
| [`pos.session`](entities/pos.session.md) | `try_cash_in_out` | AccessError | You don't have the access rights to perform a cash in/out. |
| [`pos.session`](entities/pos.session.md) | `try_cash_in_out` | UserError | There is no cash payment method for this PoS Session |
| [`pos.session`](entities/pos.session.md) | `delete_cash_in_out` | AccessError | You don't have the access rights to delete a cash in/out. |
| [`pos.session`](entities/pos.session.md) | `delete_cash_in_out` | AccessError | You cannot delete a cash move that is not linked to this session. |
| [`pos.session`](entities/pos.session.md) | `l10n_tw_edi_check_mobile_barcode` | UserError | Mobile barcode is invalid! |
| [`pos.session`](entities/pos.session.md) | `l10n_tw_edi_check_love_code` | UserError | Love code is invalid! |
| [`pos.session`](entities/pos.session.md) | `_get_split_receivable_op_vals` | UserError | The partner of the POS online payment (id=%d) could not be found |
| [`print.prenumbered.checks`](entities/print.prenumbered.checks.md) | `_check_next_check_number` | ValidationError | Next Check Number should only contains numbers. |
| [`privacy.lookup.wizard`](entities/privacy.lookup.wizard.md) | `_get_query` | UserError | Invalid email address “%s” |
| [`privacy.lookup.wizard.line`](entities/privacy.lookup.wizard.line.md) | `action_unlink` | UserError | The record is already unlinked. |
| [`product.attribute`](entities/product.attribute.md) | `write` | UserError | You cannot change the Variants Creation Mode of the attribute %(attribute)s because it is used on the following products: %(products)s |
| [`product.attribute`](entities/product.attribute.md) | `_unlink_except_used_on_product` | UserError | You cannot delete the attribute %(attribute)s because it is used on the following products: %(products)s |
| [`product.attribute`](entities/product.attribute.md) | `action_archive` | UserError | You cannot archive this attribute as there are still products linked to it |
| [`product.attribute.value`](entities/product.attribute.value.md) | `write` | UserError | You cannot change the attribute of the value %(value)s because it is used on the following products: %(products)s |
| [`product.attribute.value`](entities/product.attribute.value.md) | `_unlink_except_used_on_product` | UserError | is_used_on_products |
| [`product.category`](entities/product.category.md) | `_check_category_recursion` | ValidationError | You cannot create recursive categories. |
| [`product.category`](entities/product.category.md) | `_unlink_except_delivery_category` | UserError | You cannot delete the deliveries product category as it is used on the delivery carriers products. |
| [`product.combo`](entities/product.combo.md) | `_check_combo_item_ids_not_empty` | ValidationError | A combo choice must contain at least 1 product. |
| [`product.combo`](entities/product.combo.md) | `_check_combo_item_ids_no_duplicates` | ValidationError | A combo choice can't contain duplicate products. |
| [`product.combo`](entities/product.combo.md) | `_check_qty_max` | ValidationError | The maximum quantity of a combo must be greater or equal to 1. |
| [`product.combo`](entities/product.combo.md) | `_check_qty_free` | ValidationError | The free quantity of a combo must be greater or equal to 0. |
| [`product.combo`](entities/product.combo.md) | `_check_qty_max_greater_than_qty_free` | ValidationError | The free quantity must be smaller or equal to the maximum quantity. |
| [`product.combo.item`](entities/product.combo.item.md) | `_check_product_id_no_combo` | ValidationError | A combo choice can't contain products of type "combo". |
| [`product.document`](entities/product.document.md) | `_onchange_url` | ValidationError | Please enter a valid URL. Example: https://www.the system.com  Invalid URL: %s |
| [`product.document`](entities/product.document.md) | `_unsupported_product_product_document_on_ecommerce` | ValidationError | Documents shown on product page cannot be restricted to a specific variant |
| [`product.document`](entities/product.document.md) | `_gelato_prepare_file_payload` | UserError | Print images must be set on products before they can be ordered. |
| [`product.document`](entities/product.document.md) | `_check_attached_on_and_datas_compatibility` | ValidationError | When attached inside a quote, the document must be a file, not a URL. |
| [`product.document`](entities/product.document.md) | `_check_attached_on_and_datas_compatibility` | ValidationError | Only PDF documents can be attached inside a quote. |
| [`product.document`](entities/product.document.md) | `_check_product_is_unpublished_before_removing_print_images` | ValidationError | Products must be unpublished before print images can be removed. |
| [`product.feed`](entities/product.feed.md) | `_check_product_limit` | ValidationError | A single feed cannot contain more than %(limit)s products. Please separate products with Categories. |
| [`product.image`](entities/product.image.md) | `_check_valid_video_url` | ValidationError | Provided video URL for '%s' is not valid. Please enter a valid video URL. |
| [`product.label.layout`](entities/product.label.layout.md) | `_prepare_report_data` | UserError | You need to set a positive quantity. |
| [`product.label.layout`](entities/product.label.layout.md) | `_prepare_report_data` | UserError | No product to print, if the product is archived please unarchive it before printing its label. |
| [`product.label.layout`](entities/product.label.layout.md) | `process` | UserError | Unable to find report template for %s format |
| [`product.pricelist`](entities/product.pricelist.md) | `_unlink_except_used_as_rule_base` | UserError | You cannot delete pricelist(s): (%(pricelists)s) They are used within pricelist(s): %(other_pricelists)s |
| [`product.pricelist`](entities/product.pricelist.md) | `_check_websites_in_company` | ValidationError | Only the company's websites are allowed. Leave the Company field empty or select a website from that company. |
| [`product.pricelist`](entities/product.pricelist.md) | `action_archive` | UserError | This pricelist may not be archived. It is being used for active promotion programs: %s |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_base_pricelist_id` | ValidationError | A pricelist item with "Other Pricelist" as base must have a base_pricelist_id. |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_pricelist_recursion` | ValidationError | Recursive pricelist rules detected: %s |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_date_range` | ValidationError | %(item_name)s: end date (%(end_date)s) should be after start date (%(start_date)s) |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_margin` | ValidationError | The minimum margin should be lower than the maximum margin. |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_product_consistency` | ValidationError | Please specify the category for which this rule should be applied |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_product_consistency` | ValidationError | Please specify the product for which this rule should be applied |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_check_product_consistency` | ValidationError | Please specify the product variant for which this rule should be applied |
| [`product.pricelist.item`](entities/product.pricelist.item.md) | `_onchange_price_round` | ValidationError | The rounding method must be strictly positive. |
| [`product.product`](entities/product.product.md) | `_check_duplicated_product_barcodes` | ValidationError | Barcode(s) already assigned:  %s |
| [`product.product`](entities/product.product.md) | `_check_duplicated_packaging_barcodes` | ValidationError | A packaging already uses the barcode |
| [`product.product`](entities/product.product.md) | `_onchange_standard_price` | ValidationError | The cost of a product can't be negative. |
| [`product.product`](entities/product.product.md) | `_load_records_write` | ValidationError | The exitings product has different attribute value. "%(imported_values)s" is not equivalent to "%(existing_values)s" for "%(external_id)s", "%(id)s" |
| [`product.product`](entities/product.product.md) | `action_open_label_layout` | ValidationError | Labels cannot be printed for products of service type |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_get_rules_from_location` | UserError | Invalid rule's configuration, the following rule causes an endless loop: %s |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_get_standard_price_at_date` | ValidationError | You can only get the standard price at a given date for products with 'Standard Price' as cost method. |
| [`product.product`](entities/product.product.md) | `_check_event_ticket_service_tracking` | ValidationError | Products linked to an event ticket must have "%(tracking)s" set to "%(event)s". |
| [`product.product`](entities/product.product.md) | `_check_service_tracking_for_event_booths` | ValidationError | You cannot change the service_tracking of the product %(product_name)s because it is already assigned to %(booth_category_name)s. The service_tracking must remain 'event_booth'. |
| [`product.product`](entities/product.product.md) | `_unlink_except_active_pos_session` | UserError | To delete a product, make sure all point of sale sessions are closed.  Deleting a product available in a session would be like attempting to snatch a hamburger from a customer’s hand mid-bite; chaos will ensue as ketchup and mayo go flying everywhere! |
| [`product.product`](entities/product.product.md) | `_check_base_unit_count` | ValidationError | The value of Base Unit Count must be greater than 0. Use 0 to hide the price per unit on this product. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_check_l10n_tr_ctsp_number` | ValidationError | CTSP Number must be 12 digits or fewer. |
| [`product.product`](entities/product.product.md) | `write` | ValidationError | This product may not be archived. It is being used for an active promotion program. |
| [`product.product`](entities/product.product.md) | `_unlink_except_loyalty_products` | UserError | You cannot delete %(name)s as it is used in 'Coupons & Loyalty'. Please archive it instead. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_update_uom` | UserError | As other units of measure (ex : %(problem_uom)s) than %(uom)s have already been used for this product, the change of unit of measure can not be done.If you want to change it, please archive the product and create a new one. |
| [`product.product`](entities/product.product.md) | `_unlink_except_master_data` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. |
| [`product.product`](entities/product.product.md) | `write` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. |
| [`product.replenish`](entities/product.replenish.md) | `launch_replenishment` | UserError | error |
| [`product.ribbon`](entities/product.ribbon.md) | `_check_assign` | ValidationError | Only one ribbon with the assign %s is allowed. |
| [`product.template`](entities/product.template.md) | `_onchange_standard_price` | ValidationError | The cost of a product can't be negative. |
| [`product.template`](entities/product.template.md) | `_onchange_type` | UserError | Combo products can't have attributes. |
| [`product.template`](entities/product.template.md) | `_onchange_type` | UserError | This product is part of a combo, so its type can't be changed to "combo". |
| [`product.template`](entities/product.template.md) | `_check_combo_ids_not_empty` | ValidationError | A combo product must contain at least 1 combo choice. |
| [`product.template`](entities/product.template.md) | `_check_sale_combo_ids` | ValidationError | A sellable combo product can only contain sellable products. |
| [`product.template`](entities/product.template.md) | `action_open_label_layout` | ValidationError | Labels cannot be printed for products of service type |
| [`product.template`](entities/product.template.md) | `_create_variant_ids` | UserError | This configuration of product attributes, values, and exclusions would lead to no possible variant. Please archive or delete your product directly if intended. |
| [`product.template`](entities/product.template.md) | `_create_variant_ids` | UserError | The number of variants to generate is above allowed limit. You should either not generate variants for each combination or generate them on demand from the sales order. To do so, open the form view of attributes and change the mode of *Create Variants*. |
| [`product.template`](entities/product.template.md) | `_check_uom_not_in_invoice` | ValidationError | This product is already being used in posted Journal Entries. If you want to change its Unit of Measure, please archive this product and create a new one. |
| [`product.template`](entities/product.template.md) | `_check_sale_product_company` | ValidationError | The following products cannot be restricted to the company %(company)s because they have already been used in quotations or sales orders in another company: %(used_products)s You can archive these products and recreate them with your company restriction instead, or leave them as shared product. |
| [`product.template`](entities/product.template.md) | `_check_incompatible_types` | ValidationError | The product (%(product)s) has incompatible values: %(value_list)s |
| [`product.template`](entities/product.template.md) | `_inverse_qty_available` | UserError | Save the product form before updating the Quantity On Hand. |
| [`product.template`](entities/product.template.md) | `write` | UserError | This product's company cannot be changed as long as there are stock moves of it belonging to another company. |
| [`product.template`](entities/product.template.md) | `write` | UserError | This product's company cannot be changed as long as there are quantities of it belonging to another company. |
| [`product.template`](entities/product.template.md) | `_search_valuation` | UserError | You can only use the '=' operator to search on valuation field. |
| [`product.template`](entities/product.template.md) | `_search_valuation` | UserError | Only the value 'periodic' and 'real_time' are accepted to search on valuation field. |
| [`product.template`](entities/product.template.md) | `write` | UserError | You cannot enable lot valuation because the following products have on-hand quantities without a lot/serial number: %s |
| [`product.template`](entities/product.template.md) | `_check_service_tracking_for_event_booths` | ValidationError | The "service_tracking" for the product template, %(product_template_name)s cannot be changed because one of its variants is assigned to the Event Booth Category, %(event_booth_category_name)s. The service_tracking must remain "Event Booth". |
| [`product.template`](entities/product.template.md) | `_unlink_except_open_session` | UserError | To delete a product, make sure all point of sale sessions are closed.  Deleting a product available in a session would be like attempting to snatch a hamburger from a customer’s hand mid-bite; chaos will ensue as ketchup and mayo go flying everywhere! |
| [`product.template`](entities/product.template.md) | `_ensure_unused_in_pos` | UserError | Hold up! Archiving products while POS sessions are active is like pulling a plate mid-meal. Make sure to close all sessions first to avoid any issues. |
| [`product.template`](entities/product.template.md) | `_check_is_special_product` | UserError | You cannot archive a product that is set as a special product in a Point of Sale configuration. Please change the configuration first. |
| [`product.template`](entities/product.template.md) | `_check_combo_inclusions` | UserError | You must first remove this product from the %s combo |
| [`product.template`](entities/product.template.md) | `_unlink_except_loyalty_products` | UserError | You cannot delete %(name)s as it is used in 'Coupons & Loyalty'. Please archive it instead. |
| [`product.template`](entities/product.template.md) | `write` | UserError | You cannot change the product type or disable landed cost option because the product is used in an account move line. |
| [`product.template`](entities/product.template.md) | `_check_service_to_purchase` | ValidationError | Product that is not a service can not create RFQ. |
| [`product.template`](entities/product.template.md) | `_check_vendor_for_service_to_purchase` | ValidationError | Please define the vendor from whom you would like to purchase this service automatically. |
| [`product.template`](entities/product.template.md) | `_check_project_and_template` | ValidationError | The product %s should not have a project nor a project template since it will not generate project. |
| [`product.template`](entities/product.template.md) | `_check_project_and_template` | ValidationError | The product %s should not have a project template since it will generate a task in a global project. |
| [`product.template`](entities/product.template.md) | `_check_project_and_template` | ValidationError | The product %s should not have a global project since it will generate a project. |
| [`product.template`](entities/product.template.md) | `_unlink_except_master_data` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. |
| [`product.template`](entities/product.template.md) | `write` | ValidationError | The %s product is required by the Timesheets app and cannot be archived, deleted nor linked to a company. |
| [`product.template`](entities/product.template.md) | `_check_print_images_are_set_before_publishing` | ValidationError | Print images must be set on products before they can be published. |
| [`product.template.attribute.line`](entities/product.template.attribute.line.md) | `_check_valid_values` | ValidationError | The attribute %(attribute)s must have at least one value for the product %(product)s. |
| [`product.template.attribute.line`](entities/product.template.attribute.line.md) | `_check_valid_values` | ValidationError | On the product %(product)s you cannot associate the value %(value)s with the attribute %(attribute)s because they do not match. |
| [`product.template.attribute.line`](entities/product.template.attribute.line.md) | `write` | UserError | You cannot move the attribute %(attribute)s from the product %(product_src)s to the product %(product_dest)s. |
| [`product.template.attribute.line`](entities/product.template.attribute.line.md) | `write` | UserError | On the product %(product)s you cannot transform the attribute %(attribute_src)s into the attribute %(attribute_dest)s. |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | `_check_valid_values` | ValidationError | The value %(value)s is not defined for the attribute %(attribute)s on the product %(product)s. |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | `create` | UserError | You cannot update related variants from the values. Please update related values from the variants. |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | `write` | UserError | You cannot update related variants from the values. Please update related values from the variants. |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | `write` | UserError | You cannot change the value of the value %(value)s set on product %(product)s. |
| [`product.template.attribute.value`](entities/product.template.attribute.value.md) | `write` | UserError | You cannot change the product of the value %(value)s set on product %(product)s. |
| [`product.uom`](entities/product.uom.md) | `_check_barcode_uniqueness` | ValidationError | A product already uses the barcode |
| [`project.project`](entities/project.project.md) | `_inverse_company_id` | UserError | The project and the associated partner must be linked to the same company. |
| [`project.project`](entities/project.project.md) | `_inverse_company_id` | UserError | The project's company cannot be changed if its analytic account has analytic lines or if more than one project is linked to it. |
| [`project.project`](entities/project.project.md) | `_ensure_stage_has_same_company` | UserError | _('This project is associated with %(project_company)s, whereas the selected stage belongs to %(stage_company)s. There are a couple of options to consider: either remove the company designation from the project or from the stage. Alternatively, you can update the company information for these record |
| [`project.project`](entities/project.project.md) | `_check_allow_timesheet` | ValidationError | To use the timesheets feature, you need an analytic account for your project. Please set one up in the plan '%(plan_name)s' or turn off the timesheets feature. |
| [`project.project`](entities/project.project.md) | `_unlink_except_contains_entries` | RedirectWarning | warning_msg |
| [`project.project`](entities/project.project.md) | `_check_sale_line_type` | ValidationError | You cannot link a billable project to a sales order item that is not a service. |
| [`project.project`](entities/project.project.md) | `_check_sale_line_type` | ValidationError | You cannot link a billable project to a sales order item that comes from an expense or a vendor bill. |
| [`project.project.stage`](entities/project.project.stage.md) | `write` | UserError | You are not able to switch the company of this stage to %(company_name)s since it currently includes projects associated with %(project_company_name)s. Please ensure that this stage exclusively consists of projects linked to %(company_name)s. |
| [`project.task`](entities/project.task.md) | `_ensure_company_consistency_with_partner` | ValidationError | The task and the associated partner must be linked to the same company. |
| [`project.task`](entities/project.task.md) | `_ensure_super_task_is_not_private` | ValidationError | This task has sub-tasks, so it can't be private. |
| [`project.task`](entities/project.task.md) | `_check_no_cyclic_dependencies` | ValidationError | Two tasks cannot depend on each other. |
| [`project.task`](entities/project.task.md) | `_check_parent_id` | ValidationError | Error! You cannot create a recursive hierarchy of tasks. |
| [`project.task`](entities/project.task.md) | `write` | UserError | Sorry. You can't set a task as its parent task. |
| [`project.task`](entities/project.task.md) | `write` | UserError | You can only set a personal stage on a private task. |
| [`project.task`](entities/project.task.md) | `_check_project_root` | UserError | This task cannot be private because there are some timesheets linked to it. |
| [`project.task`](entities/project.task.md) | `_unlink_except_contains_entries` | RedirectWarning | warning_msg |
| [`project.task`](entities/project.task.md) | `_unlink_except_contains_entries` | UserError | This task can’t be deleted because it’s linked to timesheets. Please contact someone with higher access to remove the timesheets first, and then you’ll be able to delete the task. |
| [`project.task`](entities/project.task.md) | `_check_sale_line_type` | ValidationError | You cannot link the order item %(order_id)s - %(product_id)s to this task because it is a re-invoiced expense. |
| [`project.task.burndown.chart.report`](entities/project.task.burndown.chart.report.md) | `_validate_group_by` | UserError | The view must be grouped by date and by Stage - Burndown chart or Is Closed - Burnup chart |
| [`project.task.recurrence`](entities/project.task.recurrence.md) | `_check_repeat_interval` | ValidationError | The interval should be greater than 0 |
| [`project.task.recurrence`](entities/project.task.recurrence.md) | `_check_repeat_until_date` | ValidationError | The end date should be in the future |
| [`project.task.type`](entities/project.task.type.md) | `_unlink_if_remaining_personal_stages` | UserError | Each user should have at least one personal stage. Create a new stage to which the tasks can be transferred after the selected ones are deleted. |
| [`project.task.type`](entities/project.task.type.md) | `_check_personal_stage_not_linked_to_projects` | UserError | A personal stage cannot be linked to a project because it is only visible to its corresponding user. |
| [`properties.base.definition`](entities/properties.base.definition.md) | `_check_properties_field_id` | ValidationError | The definition needs to be linked to a properties field. Those fields are not: %s. |
| [`properties.base.definition`](entities/properties.base.definition.md) | `write` | AccessError | You can not change the field of a base definition |
| [`properties.base.definition`](entities/properties.base.definition.md) | `get_properties_base_definition` | AccessError | You can not read that field definition. |
| [`publisher_warranty.contract`](entities/publisher_warranty.contract.md) | `update_notification` | UserError | Error during communication with the publisher warranty server. |
| [`purchase.bill.line.match`](entities/purchase.bill.line.match.md) | `action_match_lines` | UserError | You must select at least one Purchase Order line to match or create bill. |
| [`purchase.bill.line.match`](entities/purchase.bill.line.match.md) | `action_add_to_po` | UserError | Select Vendor Bill lines to add to a Purchase Order |
| [`purchase.bill.line.match`](entities/purchase.bill.line.match.md) | `action_add_to_po` | UserError | Please select bill lines with the same vendor. |
| [`purchase.bill.line.match`](entities/purchase.bill.line.match.md) | `action_add_to_po` | UserError | Vendor Bill lines can only be added to one Purchase Order. |
| [`purchase.order`](entities/purchase.order.md) | `_check_order_line_company_id` | ValidationError | Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s.   Please change the company of your quotation or remove the products from other companies (%(bad_products)s). |
| [`purchase.order`](entities/purchase.order.md) | `_search_is_late` | ValidationError | Unsupported operator |
| [`purchase.order`](entities/purchase.order.md) | `_unlink_if_cancelled` | UserError | In order to delete a purchase order, you must cancel it first. |
| [`purchase.order`](entities/purchase.order.md) | `button_confirm` | UserError | error_msg |
| [`purchase.order`](entities/purchase.order.md) | `button_cancel` | UserError | Unable to cancel purchase order(s): %s. You must first unlock them. |
| [`purchase.order`](entities/purchase.order.md) | `button_cancel` | UserError | Unable to cancel purchase order(s): %s. You must first cancel their related vendor bills. |
| [`purchase.order`](entities/purchase.order.md) | `action_create_invoice` | ValidationError | You can only upload a bill for a single vendor at a time. |
| [`purchase.order`](entities/purchase.order.md) | `action_merge` | UserError | Please select at least two purchase orders with state RFQ and RFQ sent to merge. |
| [`purchase.order`](entities/purchase.order.md) | `action_merge` | UserError | In selected purchase order to merge these details must be same Vendor, currency, destination, dropship address and agreement |
| [`purchase.order`](entities/purchase.order.md) | `create_document_from_attachment` | UserError | No attachment was provided |
| [`purchase.order`](entities/purchase.order.md) | `_prepare_picking` | UserError | You must set a Vendor Location for this partner %s |
| [`purchase.order`](entities/purchase.order.md) | `_apply_grid` | ValidationError | You cannot change the quantity of a product present in multiple purchase lines. |
| [`purchase.order.line`](entities/purchase.order.line.md) | `write` | UserError | You cannot change the type of a purchase order line. Instead you should delete the current line and create a new line of the proper type. |
| [`purchase.order.line`](entities/purchase.order.line.md) | `_unlink_except_purchase` | UserError | Cannot delete a purchase order line which is in state “%s”. |
| [`purchase.order.line`](entities/purchase.order.line.md) | `_check_orderpoint_picking_type` | UserError | The warehouse of operation type (%(operation_type)s) is inconsistent with location (%(location)s) of reordering rule (%(reordering_rule)s) for product %(product)s. Change the operation type or cancel the request for quotation. |
| [`purchase.requisition`](entities/purchase.requisition.md) | `_check_dates` | ValidationError | End date cannot be earlier than start date. Please check dates for agreements: %s |
| [`purchase.requisition`](entities/purchase.requisition.md) | `write` | UserError | You cannot change the Agreement Type or Company of a not draft purchase agreement. |
| [`purchase.requisition`](entities/purchase.requisition.md) | `action_confirm` | UserError | You cannot confirm agreement '%(agreement)s' because it does not contain any product lines. |
| [`purchase.requisition`](entities/purchase.requisition.md) | `action_confirm` | UserError | You cannot confirm a blanket order with lines missing a price. |
| [`purchase.requisition`](entities/purchase.requisition.md) | `action_confirm` | UserError | You cannot confirm a blanket order with lines missing a quantity. |
| [`purchase.requisition`](entities/purchase.requisition.md) | `action_done` | UserError | To close this purchase requisition, cancel related Requests for Quotation.  Imagine the mess if someone confirms these duplicates: double the order, double the trouble :) |
| [`purchase.requisition`](entities/purchase.requisition.md) | `_unlink_if_draft_or_cancel` | UserError | You can only delete draft or cancelled requisitions. |
| [`purchase.requisition.line`](entities/purchase.requisition.line.md) | `create` | UserError | You cannot have a negative or unit price of 0 for an already confirmed blanket order. |
| [`purchase.requisition.line`](entities/purchase.requisition.line.md) | `write` | UserError | You cannot have a negative or unit price of 0 for an already confirmed blanket order. |
| [`quotation.document`](entities/quotation.document.md) | `_check_pdf_validity` | ValidationError | Only PDF documents can be used as header or footer. |
| [`rating.rating`](entities/rating.rating.md) | `_check_synchronize_publisher_values` | AccessError | Updating rating comment require write access on related record |
| [`repair.order`](entities/repair.order.md) | `action_generate_serial` | UserError | Please set the first Serial Number or a default sequence |
| [`repair.order`](entities/repair.order.md) | `action_repair_cancel` | UserError | You cannot cancel a Repair Order that's already been completed |
| [`repair.order`](entities/repair.order.md) | `action_repair_done` | ValidationError | Serial number is required for product to repair : %s |
| [`repair.order`](entities/repair.order.md) | `action_repair_end` | UserError | Repair must be under repair in order to end reparation. |
| [`repair.order`](entities/repair.order.md) | `action_validate` | UserError | You can not enter negative quantities. |
| [`repair.order`](entities/repair.order.md) | `_create_sale_order` | UserError | You cannot create a quotation for a repair order that is already linked to an existing sale order. Concerned repair order(s): %(ref_str)s |
| [`repair.order`](entities/repair.order.md) | `_create_sale_order` | UserError | You need to define a customer for a repair order in order to create an associated quotation. Concerned repair order(s): %(ref_str)s |
| [`report.mrp.report_bom_structure`](entities/report.mrp.report_bom_structure.md) | `_simulate_operation_planning` | UserError | Impossible to plan. Please check the workcenter availabilities. |
| [`report.mrp.report_bom_structure`](entities/report.mrp.report_bom_structure.md) | `_simulate_operation_planning` | UserError | There is no defined calendar on workcenter %s. |
| [`report.paperformat`](entities/report.paperformat.md) | `_check_format_or_page` | ValidationError | You can select either a format or a specific page width/height, but not both. |
| [`report.point_of_sale.report_invoice`](entities/report.point_of_sale.report_invoice.md) | `_get_report_values` | UserError | No link to an invoice for %s. |
| [`report.stock.label_product_product_view`](entities/report.stock.label_product_product_view.md) | `_get_report_values` | UserError | Product model not defined, Please contact your administrator. |
| [`res.bank`](entities/res.bank.md) | `_constrains_intermediary_bank_id` | ValidationError | A bank cannot be its own intermediary bank. |
| [`res.company`](entities/res.company.md) | `copy` | UserError | Duplicating a company is not allowed. Please create a new company instead. |
| [`res.company`](entities/res.company.md) | `write` | UserError | The company hierarchy cannot be changed. |
| [`res.company`](entities/res.company.md) | `_check_active` | ValidationError | The company %(company_name)s cannot be archived because it is still used as the default company of %(active_users)s users. |
| [`res.company`](entities/res.company.md) | `_check_root_delegated_fields` | ValidationError | The %s of a subsidiary must be the same as it's root company. |
| [`res.company`](entities/res.company.md) | `_check_audit_trail_restriction` | ValidationError | Can't disable restricted audit trail: forced by localization. |
| [`res.company`](entities/res.company.md) | `_check_set_account_price_include` | ValidationError | Cannot change Price Tax computation method on a company that has already started invoicing. |
| [`res.company`](entities/res.company.md) | `_check_fiscalyear_last_day` | ValidationError | Invalid fiscal year last day |
| [`res.company`](entities/res.company.md) | `_validate_locks` | RedirectWarning | error_msg |
| [`res.company`](entities/res.company.md) | `_validate_locks` | RedirectWarning | error_msg |
| [`res.company`](entities/res.company.md) | `_validate_locks` | UserError | The Hard Lock Date cannot be removed. |
| [`res.company`](entities/res.company.md) | `_validate_locks` | UserError | A new Hard Lock Date must be posterior (or equal) to the previous one. |
| [`res.company`](entities/res.company.md) | `write` | UserError | You cannot change the currency of the company since some journal items already exist |
| [`res.company`](entities/res.company.md) | `_get_default_opening_move_values` | UserError | Please install a chart of accounts or create a miscellaneous journal before proceeding. |
| [`res.company`](entities/res.company.md) | `_update_opening_move` | UserError | You cannot import the "openning_balance" if the opening move (%s) is already posted.                 If you are absolutely sure you want to modify the opening balance of your accounts, reset the move to draft. |
| [`res.company`](entities/res.company.md) | `get_chart_of_accounts_or_fail` | RedirectWarning | msg |
| [`res.company`](entities/res.company.md) | `_check_hash_integrity` | UserError | Please contact your accountant to print the Hash integrity result. |
| [`res.company`](entities/res.company.md) | `_with_locked_records` | UserError | Some documents are being sent by another process already. |
| [`res.company`](entities/res.company.md) | `_check_phonenumbers_import` | ValidationError | Please install the phonenumbers library. |
| [`res.company`](entities/res.company.md) | `_sanitize_peppol_phone_number` | ValidationError | error_message |
| [`res.company`](entities/res.company.md) | `_sanitize_peppol_phone_number` | ValidationError | error_message |
| [`res.company`](entities/res.company.md) | `_check_peppol_endpoint` | ValidationError | The Peppol endpoint identification number is not correct. |
| [`res.company`](entities/res.company.md) | `_check_peppol_purchase_journal_id` | ValidationError | A purchase journal must be used to receive Peppol documents. |
| [`res.company`](entities/res.company.md) | `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. |
| [`res.company`](entities/res.company.md) | `action_close_stock_valuation` | UserError | It exists closing entries after the selected date. Cancel them before generate an entry prior to them |
| [`res.company`](entities/res.company.md) | `action_close_stock_valuation` | UserError | Everything is correctly closed |
| [`res.company`](entities/res.company.md) | `action_close_stock_valuation` | UserError | Please set the Journal for Inventory Valuation in the settings. |
| [`res.company`](entities/res.company.md) | `action_close_stock_valuation` | UserError | Please set the Valuation Account for Inventory Valuation in the settings. |
| [`res.company`](entities/res.company.md) | `_check_country_change_holidays` | ValidationError | The company country cannot be changed while time off leaves or allocations with the country exist. |
| [`res.company`](entities/res.company.md) | `_check_active` | ValidationError | The company “%(company_name)s” cannot be archived because it has a linked website “%(website_name)s”. Change that website's company first. |
| [`res.company`](entities/res.company.md) | `_check_internal_project_id_company` | ValidationError | The Internal Project of a company should be in that company. |
| [`res.company`](entities/res.company.md) | `validate_lock_dates` | ValidationError | Please close all the point of sale sessions in this period before closing it. Open sessions are: %s |
| [`res.company`](entities/res.company.md) | `write` | UserError | Could not change the ARCA Responsibility of this company because there are already accounting entries. |
| [`res.company`](entities/res.company.md) | `write` | ValidationError | You cannot change the fiscal country. |
| [`res.company`](entities/res.company.md) | `get_l10n_de_stnr_national` | ValidationError | Your company's SteuerNummer is not compatible with your state |
| [`res.company`](entities/res.company.md) | `get_l10n_de_stnr_national` | ValidationError | Your company's SteuerNummer is not valid |
| [`res.company`](entities/res.company.md) | `_check_phonenumbers_import` | ValidationError | Please install the phonenumbers library. |
| [`res.company`](entities/res.company.md) | `_sanitize_nemhandel_phone_number` | ValidationError | error_message |
| [`res.company`](entities/res.company.md) | `_sanitize_nemhandel_phone_number` | ValidationError | error_message |
| [`res.company`](entities/res.company.md) | `_check_nemhandel_purchase_journal_id` | ValidationError | A purchase journal must be used to receive Nemhandel documents. |
| [`res.company`](entities/res.company.md) | `_map_eu_taxes` | RedirectWarning | To properly configure OSS tax mapping, the domestic tax group you are using must have the necessary accounts defined. |
| [`res.company`](entities/res.company.md) | `_get_fr_reference_leave_type` | ValidationError | You must first define a reference time off type for the company. |
| [`res.company`](entities/res.company.md) | `_inverse_pdp_identifier` | UserError | The identifier %s is not valid. The expected format is: SIREN, SIREN_SIRET, SIREN_SIRET_CodeRoutage or SIREN_SuffixeAdressage |
| [`res.company`](entities/res.company.md) | `_check_pos_hash_integrity` | UserError | Accounting is not unalterable for the company %s. This mechanism is designed for companies where accounting is unalterable. |
| [`res.company`](entities/res.company.md) | `_check_pos_hash_integrity` | UserError | msg_alert |
| [`res.company`](entities/res.company.md) | `_check_l10n_hr_mer_purchase_journal_id` | ValidationError | A purchase journal must be used to receive eRacun document via MojEracun. |
| [`res.company`](entities/res.company.md) | `_l10n_hu_edi_get_credentials_dict` | UserError | Missing NAV credentials for company %s |
| [`res.company`](entities/res.company.md) | `_l10n_hu_edi_test_credentials` | UserError | NAV Credentials: Please set the hungarian vat number on the company first! |
| [`res.company`](entities/res.company.md) | `_l10n_hu_edi_test_credentials` | UserError | Incorrect NAV Credentials! Check that your company VAT number is set correctly.  Error details: %s |
| [`res.company`](entities/res.company.md) | `_check_l10n_it_edi_purchase_journal_id` | ValidationError | The Italian default purchase journal requires a default account. |
| [`res.company`](entities/res.company.md) | `_check_eco_admin_index` | ValidationError | All fields about the Economic and Administrative Index must be completed. |
| [`res.company`](entities/res.company.md) | `_check_eco_incorporated` | ValidationError | If one of Share Capital or Sole Shareholder is present, then they must be both filled out. |
| [`res.company`](entities/res.company.md) | `_check_tax_representative` | ValidationError | You must select a tax representative. |
| [`res.company`](entities/res.company.md) | `_check_tax_representative` | ValidationError | Your tax representative partner must have a tax number. |
| [`res.company`](entities/res.company.md) | `_check_tax_representative` | ValidationError | Your tax representative partner must have a country. |
| [`res.company`](entities/res.company.md) | `_l10n_ro_edi_process_token_response` | ValidationError | Token not found. Response: %s |
| [`res.company`](entities/res.company.md) | `_l10n_ro_edi_refresh_access_token` | UserError | Client ID and Client Secret field must be filled. |
| [`res.company`](entities/res.company.md) | `_l10n_ro_edi_refresh_access_token` | UserError | Refresh token not found |
| [`res.company`](entities/res.company.md) | `write` | UserError | ZATCA API Mode cannot be changed after an invoice has been successfully submitted under the Production Mode. |
| [`res.company`](entities/res.company.md) | `_assert_twilio_sid` | UserError | Invalid Twilio Account SID: must start with 'AC' and be 34 characters long. |
| [`res.company`](entities/res.company.md) | `_assert_twilio_sid` | UserError | Invalid Twilio Account SID: must only contain alphanumeric characters after 'AC'. |
| [`res.company.ldap`](entities/res.company.ldap.md) | `_get_or_create_user` | AccessDenied | No local user found for LDAP login and not configured to create one |
| [`res.config.settings`](entities/res.config.settings.md) | `copy` | UserError | Cannot duplicate configuration! |
| [`res.config.settings`](entities/res.config.settings.md) | `execute` | AccessError | Only administrators can change the settings |
| [`res.config.settings`](entities/res.config.settings.md) | `action_open_template_user` | UserError | Invalid template user. It seems it has been deleted. |
| [`res.config.settings`](entities/res.config.settings.md) | `open_email_layout` | UserError | This layout seems to no longer exist. |
| [`res.config.settings`](entities/res.config.settings.md) | `set_values` | UserError | Please configure the Cloud Storage before enabling it |
| [`res.config.settings`](entities/res.config.settings.md) | `_setup_cloud_storage_provider` | ValidationError | The connection string is not allowed to upload blobs to the container. %s |
| [`res.config.settings`](entities/res.config.settings.md) | `_setup_cloud_storage_provider` | ValidationError | The connection string is not allowed to download blobs from the container. %s |
| [`res.config.settings`](entities/res.config.settings.md) | `_check_cloud_storage_uninstallable` | UserError | Some Azure attachments are in use, please migrate their cloud storages before disable this module |
| [`res.config.settings`](entities/res.config.settings.md) | `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to upload blobs to the bucket. %s |
| [`res.config.settings`](entities/res.config.settings.md) | `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to download blobs from the bucket. %s |
| [`res.config.settings`](entities/res.config.settings.md) | `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to set the bucket's CORS. %s |
| [`res.config.settings`](entities/res.config.settings.md) | `_check_cloud_storage_uninstallable` | UserError | Some Google attachments are in use, please migrate cloud storages before disable the provider |
| [`res.config.settings`](entities/res.config.settings.md) | `_onchange_crm_auto_assignment_run_datetime` | UserError | Repeat frequency should be positive. |
| [`res.config.settings`](entities/res.config.settings.md) | `_onchange_crm_auto_assignment_run_datetime` | UserError | Invalid repeat frequency. Consider changing frequency type instead of using large numbers. |
| [`res.config.settings`](entities/res.config.settings.md) | `set_values` | UserError | You can't deactivate the multi-location if you have more than once warehouse by company |
| [`res.config.settings`](entities/res.config.settings.md) | `set_values` | UserError | You have product(s) in stock that have lot/serial number tracking enabled.  Switch off tracking on all the products before switching off this setting. |
| [`res.config.settings`](entities/res.config.settings.md) | `_check_google_maps_static_api_secret` | UserError | Please enter a valid base64 secret |
| [`res.config.settings`](entities/res.config.settings.md) | `button_update_nemhandel_user_data` | ValidationError | Contact email is required |
| [`res.config.settings`](entities/res.config.settings.md) | `l10n_in_edi_buy_iap` | ValidationError | Please ensure that at least one Indian service and production environment is enabled, and save the configuration to proceed with purchasing credits. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_in_check_gst_number` | RedirectWarning | Please set a valid GST number on company. |
| [`res.config.settings`](entities/res.config.settings.md) | `l10n_in_edi_test` | UserError | '\n'.join(['[%s] %s' % (e.get('code'), e.get('message')) for e in response['error']]) |
| [`res.config.settings`](entities/res.config.settings.md) | `l10n_in_edi_test` | UserError | Incorrect username or password, or the GST number on company does not match. |
| [`res.config.settings`](entities/res.config.settings.md) | `l10n_in_ewaybill_test` | UserError | Incorrect username or password, or the GST number on company does not match. |
| [`res.config.settings`](entities/res.config.settings.md) | `l10n_in_ewaybill_test` | UserError | e.get_all_error_message() |
| [`res.config.settings`](entities/res.config.settings.md) | `action_l10n_my_edi_unregister` | UserError | An unexpected error occurred while unregistering. Please try again later. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | A polish VAT number must be set on your company. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | Please set up a valid KSeF Certificate, with its Private Key set |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | The selected certificate record (%(name)s) is missing a private key. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | UserError | KSeF certificate and private key are not set. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | Failed to initiate KSeF authentication. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | Authentication with KSeF failed. |
| [`res.config.settings`](entities/res.config.settings.md) | `_l10n_pl_edi_ksef_authenticate` | ValidationError | Failed to retrieve access or refresh tokens. |
| [`res.config.settings`](entities/res.config.settings.md) | `_onchange_default_user` | ValidationError | The user must be a POS user |
| [`res.config.settings`](entities/res.config.settings.md) | `_onchange_pos_payment_method_ids` | ValidationError | You cannot add cash payment methods in kiosk mode. |
| [`res.config.settings`](entities/res.config.settings.md) | `_onchange_pos_self_order_pay_after` | ValidationError | Only pay after each is available with kiosk mode. |
| [`res.config.settings`](entities/res.config.settings.md) | `generate_qr_codes_zip` | ValidationError | QR codes can only be generated in mobile or consultation mode. |
| [`res.config.settings`](entities/res.config.settings.md) | `generate_qr_codes_zip` | ValidationError | In Self-Order mode, you must have at least one table to generate QR codes |
| [`res.config.settings`](entities/res.config.settings.md) | `generate_qr_codes_page` | ValidationError | In Self-Order mode, you must have at least one table to generate QR codes |
| [`res.country`](entities/res.country.md) | `_check_address_format` | UserError | The layout contains an invalid format key |
| [`res.currency`](entities/res.currency.md) | `_check_company_currency_stays_active` | UserError | This currency is set on a company and therefore cannot be deactivated. |
| [`res.currency`](entities/res.currency.md) | `write` | UserError | You cannot reduce the number of decimal places of a currency which has already been used to make accounting entries. |
| [`res.currency.rate`](entities/res.currency.rate.md) | `_get_latest_rate` | UserError | The name for the current rate is empty. Please set it. |
| [`res.currency.rate`](entities/res.currency.rate.md) | `_check_company_id` | ValidationError | Currency rates should only be created for main companies |
| [`res.groups`](entities/res.groups.md) | `_unlink_except_settings_group` | ValidationError | You cannot delete a group linked with a settings field. |
| [`res.groups`](entities/res.groups.md) | `write` | UserError | The name of the group can not start with "-" |
| [`res.groups`](entities/res.groups.md) | `_inverse_all_user_ids` | UserError | It is not possible to remove implied group %(group)s from users %(users)s |
| [`res.lang`](entities/res.lang.md) | `_check_active` | ValidationError | At least one language must be active. |
| [`res.lang`](entities/res.lang.md) | `_check_format` | ValidationError | Invalid date/time format directive specified. Please refer to the list of allowed directives, displayed when you edit a language. |
| [`res.lang`](entities/res.lang.md) | `_get_active_by` | UserError | Field "%s" is not cached |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | Language code cannot be modified. |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | Cannot deactivate a language that is currently used by users. |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | Cannot deactivate a language that is currently used by contacts. |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | You cannot archive the language in which the system was setup as it is used by automated processes. |
| [`res.lang`](entities/res.lang.md) | `_unlink_except_default_lang` | UserError | Base Language 'en_US' can not be deleted. |
| [`res.lang`](entities/res.lang.md) | `_unlink_except_default_lang` | UserError | You cannot delete the language which is the user's preferred language. |
| [`res.lang`](entities/res.lang.md) | `_unlink_except_default_lang` | UserError | You cannot delete the language which is Active! Please de-activate the language first. |
| [`res.lang`](entities/res.lang.md) | `format` | UserError | The language %s is not installed. |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | error |
| [`res.lang`](entities/res.lang.md) | `write` | UserError | Cannot deactivate a language that is currently used on a website. |
| [`res.partner`](entities/res.partner.md) | `_check_parent_id` | ValidationError | You cannot create recursive Partner hierarchies. |
| [`res.partner`](entities/res.partner.md) | `_check_partner_company` | ValidationError | The company assigned to this partner does not match the company this partner represents. |
| [`res.partner`](entities/res.partner.md) | `_check_barcode_unicity` | ValidationError | Another partner already has this barcode |
| [`res.partner`](entities/res.partner.md) | `write` | RedirectWarning | error_msg |
| [`res.partner`](entities/res.partner.md) | `write` | ValidationError | You cannot archive contacts linked to an active user. Ask an administrator to archive their associated user first.  Linked active users : %(names)s |
| [`res.partner`](entities/res.partner.md) | `write` | UserError | The selected company is not compatible with the companies of the related user(s) |
| [`res.partner`](entities/res.partner.md) | `_unlink_except_user` | RedirectWarning | error_msg |
| [`res.partner`](entities/res.partner.md) | `_unlink_except_user` | ValidationError | You cannot delete contacts linked to an active user. Ask an administrator to archive their associated user first.  Linked active users : %(names)s |
| [`res.partner`](entities/res.partner.md) | `name_create` | ValidationError | Couldn't create contact without email address! |
| [`res.partner`](entities/res.partner.md) | `_signup_retrieve_partner` | UserError | Signup token '%s' is not valid or expired |
| [`res.partner`](entities/res.partner.md) | `write` | UserError | You cannot set a partner as an invoicing address of another if they have a different %(vat_label)s. |
| [`res.partner`](entities/res.partner.md) | `_unlink_if_partner_in_account_move` | UserError | The partner cannot be deleted because it is used in Accounting |
| [`res.partner`](entities/res.partner.md) | `_merge_method` | UserError | Partners that are used in hashed entries cannot be merged. |
| [`res.partner`](entities/res.partner.md) | `_check_peppol_fields` | ValidationError | error |
| [`res.partner`](entities/res.partner.md) | `_run_vat_checks` | ValidationError | To explicitly indicate no (valid) VAT, use '/' instead. |
| [`res.partner`](entities/res.partner.md) | `_run_vat_checks` | ValidationError | msg |
| [`res.partner`](entities/res.partner.md) | `_run_vat_checks` | ValidationError | msg + '\n\n' + _('If you are trying to input a European number, this is the expected format: ') + _ref_vat[country_code.lower()] |
| [`res.partner`](entities/res.partner.md) | `_get_iap_vies_endpoint` | UserError | Invalid IAP VIES endpoint |
| [`res.partner`](entities/res.partner.md) | `_unlink_contact_rel_employee` | UserError | You cannot delete contact that are linked to an employee, please archive them instead. |
| [`res.partner`](entities/res.partner.md) | `_unlink_contact_rel_employee` | RedirectWarning | error_msg |
| [`res.partner`](entities/res.partner.md) | `_ensure_same_company_than_projects` | UserError | Partner company cannot be different from its assigned projects' company |
| [`res.partner`](entities/res.partner.md) | `_ensure_same_company_than_tasks` | UserError | Partner company cannot be different from its assigned tasks' company |
| [`res.partner`](entities/res.partner.md) | `_unlink_if_pos_no_orders` | ValidationError | You cannot delete a customer that has point of sales orders. You can archive it instead. |
| [`res.partner`](entities/res.partner.md) | `ensure_vat` | UserError | No VAT configured for partner [%i] %s |
| [`res.partner`](entities/res.partner.md) | `_l10n_ar_identification_validation` | ValidationError | The validation digit is not valid for "%s" |
| [`res.partner`](entities/res.partner.md) | `_l10n_ar_identification_validation` | ValidationError | Invalid length for "%s" |
| [`res.partner`](entities/res.partner.md) | `_l10n_ar_identification_validation` | ValidationError | Only numbers allowed for "%s" |
| [`res.partner`](entities/res.partner.md) | `_l10n_ar_identification_validation` | ValidationError | CUIT number must be prefixed with one of the following: %s |
| [`res.partner`](entities/res.partner.md) | `_l10n_ar_identification_validation` | ValidationError | repr(error) |
| [`res.partner`](entities/res.partner.md) | `_ar_unlink_except_master_data` | UserError | Deleting this partner is not allowed. |
| [`res.partner`](entities/res.partner.md) | `_run_check_identification` | ValidationError | The format of your RUN is not valid.  It should be like 76086428-5. |
| [`res.partner`](entities/res.partner.md) | `_check_nemhandel_send_oioubl` | ValidationError | On Nemhandel, only OIOUBL 2.1 is supported. |
| [`res.partner`](entities/res.partner.md) | `_run_check_identification` | ValidationError | If your identification type is %s, it must be 10 digits |
| [`res.partner`](entities/res.partner.md) | `_validate_l10n_es_edi_facturae_ac_physical_gln` | ValidationError | The Physical GLN entered is not valid. |
| [`res.partner`](entities/res.partner.md) | `_validate_l10n_es_edi_facturae_ac_logical_operational_point` | ValidationError | The Logical Operational Point entered is not valid. |
| [`res.partner`](entities/res.partner.md) | `_check_pdp_send_ubl_21_fr` | ValidationError | For French regulated invoices, only %(format_name)s is supported. |
| [`res.partner`](entities/res.partner.md) | `_check_mojeracun_send_ubl_hr` | ValidationError | On MojEracun, only the Croatian UBL format is supported. |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | UserError | You must be logged in an Indian company to use this feature |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | ValidationError | Please enter the GSTIN |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | ValidationError | This feature is not activated. Go to Settings to activate this feature. |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | UserError | Unable to connect with GST network |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | UserError | error_messages and '\n'.join(error_messages) or default_error_message |
| [`res.partner`](entities/res.partner.md) | `action_l10n_in_verify_gstin_status` | UserError | The provided GSTIN is invalid. Please check the GSTIN and try again. |
| [`res.partner`](entities/res.partner.md) | `validate_codice_fiscale` | UserError | Invalid Codice Fiscale '%s': should be like 'MRTMTT91D08F205J' for physical person and '12345670546' for businesses. |
| [`res.partner`](entities/res.partner.md) | `_check_company_registry_ma` | ValidationError | ICE number should have exactly 15 digits. |
| [`res.partner`](entities/res.partner.md) | `action_validate_tin` | UserError | In order to validate the TIN, you must provide the Identification type and number. |
| [`res.partner`](entities/res.partner.md) | `action_validate_tin` | UserError | Please register for the E-Invoicing service in the settings first. |
| [`res.partner`](entities/res.partner.md) | `_pe_unlink_except_master_data` | UserError | Deleting the partner %s is not allowed because it is required by the Peruvian point of sale. |
| [`res.partner`](entities/res.partner.md) | `_check_l10n_rs_edi_public_funds` | ValidationError | Public Funds ID(JBKJS) must be exactly five digits |
| [`res.partner`](entities/res.partner.md) | `_check_l10n_rs_edi_registration_number` | ValidationError | Customer identification number should be 8 or 13 digits |
| [`res.partner`](entities/res.partner.md) | `_run_check_identification` | ValidationError | self._l10n_uy_build_vat_error_message(partner) |
| [`res.partner`](entities/res.partner.md) | `write` | UserError | You are trying to assign two different pricelists (one directly and one from grade (%(grade_name)s)). |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_find_or_create_bank_account` | UserError | Please add your own bank account manually: %(account_number)s (%(partner)s) |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_journal_id` | ValidationError | A bank account can belong to only one journal. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_allow_out_payment` | ValidationError | You do not have the right to trust or un-trust a bank account. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_build_qr_code_vals` | UserError | Currency must always be provided in order to generate a QR-code |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_build_qr_code_vals` | UserError | The following error prevented '%(candidate)s' QR-code to be generated though it was detected as eligible: |
| [`res.partner.bank`](entities/res.partner.bank.md) | `create` | UserError | A bank account with Account Number %(number)s already exists for Partner %(partner)s, but is archived. Please unarchive it instead. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `write` | UserError | You cannot modify the account number or partner of an account that has been trusted. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `write` | UserError | You do not have the rights to trust or un-trust accounts. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `get_bban` | UserError | Cannot compute the BBAN because the account number is not an IBAN. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_validate_aba_bsb` | ValidationError | BSB is not valid (expected format is "NNN-NNN"). Please rectify. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_br_proxy` | ValidationError | The proxy type must be Email Address, Mobile Number, CPF/CNPJ (BR) or Random Key (BR) for Pix code generation. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_br_proxy` | ValidationError | %s is not a valid email. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_br_proxy` | ValidationError | %s is not a valid CPF or CNPJ (don't include periods or dashes). |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_br_proxy` | ValidationError | The mobile number %s is invalid. It must start with +55, contain a 2 digit territory or state code followed by a 9 digit number. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_br_proxy` | ValidationError | The random key %s is invalid, the format looks like this: 71d6c6e1-64ea-4a11-9560-a10870c40ca2 |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_hk_proxy` | ValidationError | The FPS Type must be either ID, Mobile or Email to generate a FPS QR code for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_hk_proxy` | ValidationError | Invalid FPS ID! Please enter a valid FPS ID with length 7 or 9 for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_hk_proxy` | ValidationError | Invalid Mobile! Please enter a valid mobile number with format +852-67891234 for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_hk_proxy` | ValidationError | Invalid Email! Please enter a valid email address for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_get_qr_vals` | ValidationError | response.get('data') |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_kh_proxy` | ValidationError | The proxy type must be Bakong Account ID |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_kh_proxy` | ValidationError | Please enter a valid Bakong Account ID. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_kh_proxy` | ValidationError | Merchant ID is missing. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_sg_proxy` | ValidationError | The PayNow Type must be either Mobile or UEN to generate a PayNow QR code for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_th_proxy` | ValidationError | The QR Code Type must be either Ewallet ID, Merchant Tax ID or Mobile Number to generate a Thailand Bank QR code for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_th_proxy` | ValidationError | The Merchant Tax ID must be in the format 1234567890123 for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_th_proxy` | ValidationError | The Mobile Number must be in the format 0812345678 for account number %s. |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_clearing_number_us` | ValidationError | ABA/Routing should only contain numbers (maximum 9 digits). |
| [`res.partner.bank`](entities/res.partner.bank.md) | `_check_vn_proxy` | ValidationError | The QR Code Type must be either Merchant ID, ATM Card Number or Bank Account to generate a Vietnam Bank QR code for account number %s. |
| [`res.partner.category`](entities/res.partner.category.md) | `_check_parent_id` | ValidationError | You can not create recursive tags. |
| [`res.partner.category`](entities/res.partner.category.md) | `_unlink_l10n_tr_official_category` | UserError | The Contact Tag(s) cannot be deleted because it is used in Türkiye electronic integrations. |
| [`res.users`](entities/res.users.md) | `_set_new_password` | UserError | Please use the change password wizard (in User Preferences or User menu) to change your own password. |
| [`res.users`](entities/res.users.md) | `_check_user_company` | ValidationError | Company %(company_name)s is not in the allowed companies for user %(user_name)s (%(company_allowed)s). |
| [`res.users`](entities/res.users.md) | `_check_action_id` | ValidationError | The "App Switcher" action cannot be selected as home action. |
| [`res.users`](entities/res.users.md) | `_check_action_id` | ValidationError | The "%s" action cannot be selected as home action. |
| [`res.users`](entities/res.users.md) | `_check_action_id` | ValidationError | The action "%s" cannot be set as the home action because it requires a record to be selected beforehand. |
| [`res.users`](entities/res.users.md) | `_check_disjoint_groups` | ValidationError | User %(user)s cannot be at the same time in exclusive groups %(groups)s. |
| [`res.users`](entities/res.users.md) | `_check_at_least_one_administrator` | ValidationError | You must have at least an administrator user. |
| [`res.users`](entities/res.users.md) | `write` | UserError | You cannot activate the superuser. |
| [`res.users`](entities/res.users.md) | `write` | UserError | You cannot deactivate the user you're currently logged in as. |
| [`res.users`](entities/res.users.md) | `_unlink_except_master_data` | UserError | You can not remove the admin user as it is used internally for resources created by the system (updates, module installation, ...) |
| [`res.users`](entities/res.users.md) | `_unlink_except_master_data` | UserError | You cannot delete the admin user because it is utilized in various places (such as security configurations,...). Instead, archive it. |
| [`res.users`](entities/res.users.md) | `_unlink_except_master_data` | UserError | Deleting the template users is not allowed. Deleting this profile will compromise critical functionalities. |
| [`res.users`](entities/res.users.md) | `_unlink_except_master_data` | UserError | Deleting the public user is not allowed. Deleting this profile will compromise critical functionalities. |
| [`res.users`](entities/res.users.md) | `_change_password` | UserError | Setting empty passwords is not allowed for security reasons! |
| [`res.users`](entities/res.users.md) | `_deactivate_portal_user` | AccessDenied | Only the portal users can delete their accounts. The user(s) %s can not be deleted. |
| [`res.users`](entities/res.users.md) | `has_group` | AccessError | You can ony call user.has_group() with your current user. |
| [`res.users`](entities/res.users.md) | `_assert_can_auth` | AccessDenied | Too many login failures, please wait a bit before trying again. |
| [`res.users`](entities/res.users.md) | `web_create_users` | UserError | You have to install the Discuss application to use this feature. |
| [`res.users`](entities/res.users.md) | `action_setup_outgoing_mail_server` | UserError | You are not allowed to create a personal mail server. |
| [`res.users`](entities/res.users.md) | `action_setup_outgoing_mail_server` | UserError | Only internal users can configure a personal mail server. |
| [`res.users`](entities/res.users.md) | `action_setup_outgoing_mail_server` | UserError | Please set your email before connecting your mail server. |
| [`res.users`](entities/res.users.md) | `action_setup_outgoing_mail_server` | UserError | Wrong email address %s. |
| [`res.users`](entities/res.users.md) | `action_setup_outgoing_mail_server` | UserError | Your email address is used by an alias domain, and so you can not create a mail server for it. |
| [`res.users`](entities/res.users.md) | `action_test_outgoing_mail_server` | UserError | You are not allowed to test personal mail servers. |
| [`res.users`](entities/res.users.md) | `action_test_outgoing_mail_server` | UserError | Only internal users can configure personal mail servers. |
| [`res.users`](entities/res.users.md) | `action_test_outgoing_mail_server` | UserError | No mail server configured |
| [`res.users`](entities/res.users.md) | `_signup_create_user` | UserError | Another user is already registered using this email address. |
| [`res.users`](entities/res.users.md) | `action_reset_password` | UserError | Could not contact the mail server, please check your outgoing email server configuration |
| [`res.users`](entities/res.users.md) | `action_reset_password` | UserError | There was an error when trying to deliver your Email, please check your configuration |
| [`res.users`](entities/res.users.md) | `_action_reset_password` | UserError | You cannot perform this action on an archived user. |
| [`res.users`](entities/res.users.md) | `_action_reset_password` | UserError | Cannot send email: user %s has no email address. |
| [`res.users`](entities/res.users.md) | `remove_oauth_access_token` | AccessError | You do not have permissions to remove the access token |
| [`res.users`](entities/res.users.md) | `_auth_oauth_validate` | AccessDenied | Missing subject identity |
| [`res.users`](entities/res.users.md) | `_login` | AccessDenied | Unknown passkey |
| [`res.users`](entities/res.users.md) | `_check_credentials` | AccessDenied | Unknown passkey |
| [`res.users`](entities/res.users.md) | `_check_credentials` | AccessDenied | e.args[0] |
| [`res.users`](entities/res.users.md) | `_check_password_policy` | UserError | u'\n\n '.join(failures) |
| [`res.users`](entities/res.users.md) | `_check_credentials` | AccessDenied | Verification failed, please double-check the 6-digit code |
| [`res.users`](entities/res.users.md) | `_check_credentials` | AccessDenied | Verification failed, please use the latest 6-digit code |
| [`res.users`](entities/res.users.md) | `_totp_rate_limit` | AccessDenied | description |
| [`res.users`](entities/res.users.md) | `action_totp_enable_wizard` | UserError | Two-factor authentication can only be enabled for yourself |
| [`res.users`](entities/res.users.md) | `action_totp_enable_wizard` | UserError | Two-factor authentication already enabled |
| [`res.users`](entities/res.users.md) | `_check_credentials` | AccessDenied | Verification failed, please double-check the 6-digit code |
| [`res.users`](entities/res.users.md) | `_send_totp_mail_code` | UserError | Cannot send email: user %s has no email address. |
| [`res.users`](entities/res.users.md) | `write` | AccessError | You are not allowed to change the calendar default privacy of another user due to privacy constraints. |
| [`res.users`](entities/res.users.md) | `action_create_employee` | AccessError | You are not allowed to create an employee because the user does not have access rights for %s |
| [`res.users`](entities/res.users.md) | `_check_login` | ValidationError | You can not have two users with the same login! |
| [`res.users`](entities/res.users.md) | `_check_disjoint_groups` | ValidationError | Remove website on related partner before they become internal user. |
| [`res.users`](entities/res.users.md) | `_refresh_microsoft_calendar_token` | UserError | error_msg |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `_remove` | AccessError | You can not remove API keys unless they're yours or you are a system user |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `_check_expiration_date` | ValidationError | The API key must have an expiration date |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `_check_expiration_date` | ValidationError | You cannot exceed %(duration)s days. |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `_check_expiration_date` | ValidationError | You cannot set an expiration date in the past. |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `_ensure_can_manage_keys_programmatically` | UserError | Programmatic API keys are not enabled |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `generate` | UserError | Limit of %s API keys is reached for programmatic creation |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `generate` | AccessDenied | The provided API key is invalid or does not belong to the current user. |
| [`res.users.apikeys`](entities/res.users.apikeys.md) | `revoke` | AccessDenied | The provided API key is invalid. |
| [`res.users.apikeys.description`](entities/res.users.apikeys.description.md) | `check_access_make_key` | AccessError | Only internal users can create API keys |
| [`res.users.apikeys.description`](entities/res.users.apikeys.description.md) | `check_access_make_key` | AccessError | Only internal and portal users can create API keys |
| [`res.users.identitycheck`](entities/res.users.identitycheck.md) | `_check_identity` | UserError | Incorrect Password, try again or click on Forgot Password to reset your password. |
| [`res.users.identitycheck`](entities/res.users.identitycheck.md) | `_check_identity` | UserError | Incorrect Passkey. Please provide a valid passkey or use a different authentication method. |
| [`res.users.settings`](entities/res.users.settings.md) | `_refresh_google_calendar_token` | UserError | error_msg |
| [`res.users.settings.embedded.action`](entities/res.users.settings.embedded.action.md) | `_check_embedded_actions_field_format` | ValidationError | The ids in %(field_name)s must not be duplicated: “%(action_ids)s” |
| [`res.users.settings.embedded.action`](entities/res.users.settings.embedded.action.md) | `_check_embedded_actions_field_format` | ValidationError | The ids in %(field_name)s must only be integers or "false": “%(action_ids)s” |
| [`reset.view.arch.wizard`](entities/reset.view.arch.wizard.md) | `default_get` | ValidationError | Can't compare more than two views. |
| [`resource.calendar`](entities/resource.calendar.md) | `_check_attendance_ids` | ValidationError | In a calendar with 2 weeks mode, all periods need to be in the sections. |
| [`resource.calendar`](entities/resource.calendar.md) | `_onchange_attendance_ids` | ValidationError | You can't delete section between weeks. |
| [`resource.calendar`](entities/resource.calendar.md) | `_check_overlap` | ValidationError | Attendances can't overlap. |
| [`resource.calendar.attendance`](entities/resource.calendar.attendance.md) | `_check_day_period` | UserError | %(att)s is a break attendance, You should not have such record on duration based calendar |
| [`resource.calendar.leaves`](entities/resource.calendar.leaves.md) | `check_dates` | ValidationError | The start date of the time off must be earlier than the end date. |
| [`resource.calendar.leaves`](entities/resource.calendar.leaves.md) | `_check_compare_dates` | ValidationError | Two public holidays cannot overlap each other for the same working hours. |
| [`restaurant.floor`](entities/restaurant.floor.md) | `_unlink_except_active_pos_session` | UserError | error_msg |
| [`restaurant.floor`](entities/restaurant.floor.md) | `write` | UserError | Please close and validate the following open PoS Session before modifying this floor. Open session: %(session_names)s |
| [`restaurant.floor`](entities/restaurant.floor.md) | `deactivate_floor` | UserError | You cannot delete a floor when orders are still in draft for this floor. |
| [`restaurant.table`](entities/restaurant.table.md) | `are_orders_still_in_draft` | UserError | You cannot delete a table when orders are still in draft for this table. |
| [`restaurant.table`](entities/restaurant.table.md) | `_unlink_except_active_pos_session` | UserError | error_msg |
| [`sale.advance.payment.inv`](entities/sale.advance.payment.inv.md) | `_check_amount_is_positive` | UserError | The value of the down payment amount must be positive. |
| [`sale.advance.payment.inv`](entities/sale.advance.payment.inv.md) | `_check_amount_is_positive` | UserError | The value of the down payment amount must be positive. |
| [`sale.loyalty.coupon.wizard`](entities/sale.loyalty.coupon.wizard.md) | `action_apply` | ValidationError | Invalid sales order. |
| [`sale.loyalty.coupon.wizard`](entities/sale.loyalty.coupon.wizard.md) | `action_apply` | ValidationError | status['error'] |
| [`sale.loyalty.reward.wizard`](entities/sale.loyalty.reward.wizard.md) | `action_apply` | ValidationError | No reward selected. |
| [`sale.loyalty.reward.wizard`](entities/sale.loyalty.reward.wizard.md) | `action_apply` | ValidationError | Coupon not found while trying to add the following reward: %s |
| [`sale.order`](entities/sale.order.md) | `_check_order_line_company_id` | ValidationError | Your quotation contains products from company %(product_company)s whereas your quotation belongs to company %(quote_company)s.   Please change the company of your quotation or remove the products from other companies (%(bad_products)s). |
| [`sale.order`](entities/sale.order.md) | `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. |
| [`sale.order`](entities/sale.order.md) | `_onchange_company_id` | ValidationError | The company is required, please select one before making any other changes to the sale order. |
| [`sale.order`](entities/sale.order.md) | `_onchange_order_line` | ValidationError | The number of selected combo items must match the number of available combo choices. |
| [`sale.order`](entities/sale.order.md) | `_unlink_except_draft_or_cancel` | UserError | You can not delete a sent quotation or a confirmed sales order. You must first cancel it. |
| [`sale.order`](entities/sale.order.md) | `write` | UserError | You cannot change the pricelist of a confirmed order ! |
| [`sale.order`](entities/sale.order.md) | `action_quotation_sent` | UserError | Only draft orders can be marked as sent directly. |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | UserError | error_msg |
| [`sale.order`](entities/sale.order.md) | `action_cancel` | UserError | You cannot cancel a locked order. Please unlock it first. |
| [`sale.order`](entities/sale.order.md) | `_create_invoices` | UserError | self._nothing_to_invoice_error_message() |
| [`sale.order`](entities/sale.order.md) | `create_document_from_attachment` | UserError | No attachment was provided |
| [`sale.order`](entities/sale.order.md) | `_remove_delivery_line` | UserError | You can not update the shipping costs on an order where it was already invoiced!  The following delivery lines (product, invoiced quantity and price) have already been processed: |
| [`sale.order`](entities/sale.order.md) | `_check_warehouse` | UserError | You must have a warehouse for line using a delivery in different company. |
| [`sale.order`](entities/sale.order.md) | `_check_warehouse` | UserError | You must set a warehouse on your sale order to proceed. |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | UserError | error |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | ValidationError | Please make sure all your event related lines are configured before confirming this order:%s |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | ValidationError | Please make sure all your event-booth related lines are configured before confirming this order:%s |
| [`sale.order`](entities/sale.order.md) | `_cart_add` | ValidationError | This product is not available (anymore) in this unit of measure. |
| [`sale.order`](entities/sale.order.md) | `_prepare_order_line_values` | UserError | The given combination does not exist therefore it cannot be added to cart. |
| [`sale.order`](entities/sale.order.md) | `_prepare_order_line_values` | UserError | Invalid request parameters. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | Your cart is not ready to be paid, please verify previous steps. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | No shipping method is selected. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | The delivery method is not compatible with your delivery address. |
| [`sale.order`](entities/sale.order.md) | `number2numeric` | UserError | Reference must contain numeric characters |
| [`sale.order`](entities/sale.order.md) | `_l10n_it_edi_doi_check_configuration` | UserError | '\n'.join(errors) |
| [`sale.order`](entities/sale.order.md) | `_check_l10n_it_edi_doi_id` | ValidationError | '\n'.join(errors) |
| [`sale.order`](entities/sale.order.md) | `_constraint_unique_assigned_grade` | ValidationError | You cannot confirm Sale Order %(sale_order_name)s because there are products assigning different grades. |
| [`sale.order`](entities/sale.order.md) | `get_first_service_line` | UserError | The Sales Order must contain at least one service product. |
| [`sale.order`](entities/sale.order.md) | `_prevent_mixing_gelato_and_non_gelato_products` | ValidationError | You cannot mix Gelato products with non-Gelato products in the same order. |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | ValidationError | message |
| [`sale.order`](entities/sale.order.md) | `_create_order_on_gelato` | UserError | The order with reference %(order_reference)s was not sent to Gelato. Reason: %(error_message)s |
| [`sale.order`](entities/sale.order.md) | `action_confirm` | ValidationError | One or more rewards on the sale order is invalid. Please check them. |
| [`sale.order`](entities/sale.order.md) | `_get_reward_values_product` | UserError | Invalid product to claim. |
| [`sale.order`](entities/sale.order.md) | `_get_reward_values_discount` | UserError | There is nothing to discount |
| [`sale.order`](entities/sale.order.md) | `_apply_grid` | ValidationError | You cannot change the quantity of a product present in multiple sale lines. |
| [`sale.order`](entities/sale.order.md) | `create` | UserError | The Sales Order must contain at least one service product. |
| [`sale.order`](entities/sale.order.md) | `_verify_updated_quantity` | UserError | The provided ticket doesn't exist |
| [`sale.order`](entities/sale.order.md) | `_verify_updated_quantity` | UserError | The provided ticket slot doesn't exist |
| [`sale.order`](entities/sale.order.md) | `_prepare_order_line_values` | UserError | The ticket doesn't match with this product. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | ' '.join(values) |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | Some products are not available in the selected store. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | Point Relais® can only be used with the delivery method Mondial Relay. |
| [`sale.order`](entities/sale.order.md) | `_check_cart_is_ready_to_be_paid` | ValidationError | Delivery method Mondial Relay can only ship to Point Relais®. |
| [`sale.order.discount`](entities/sale.order.discount.md) | `_check_discount_amount` | ValidationError | Invalid discount amount |
| [`sale.order.discount`](entities/sale.order.discount.md) | `_get_discount_product` | ValidationError | There does not seem to be any discount product configured for this company yet. You can either use a per-line discount, or ask an administrator to grant the discount the first time. |
| [`sale.order.line`](entities/sale.order.line.md) | `_check_combo_item_id` | ValidationError | A sale order line's combo item must be among its linked line's available combo items. |
| [`sale.order.line`](entities/sale.order.line.md) | `_check_combo_item_id` | ValidationError | A sale order line's product must match its combo item's product. |
| [`sale.order.line`](entities/sale.order.line.md) | `write` | UserError | You cannot modify the product of this order line. |
| [`sale.order.line`](entities/sale.order.line.md) | `write` | UserError | You cannot change the type of a sale order line. Instead you should delete the current line and create a new line of the proper type. |
| [`sale.order.line`](entities/sale.order.line.md) | `write` | UserError | It is forbidden to modify the following fields in a locked order: %s |
| [`sale.order.line`](entities/sale.order.line.md) | `_unlink_except_confirmed` | UserError | Once a sales order is confirmed, you can't remove one of its lines (we need to track if something gets invoiced or delivered).                 Set the quantity to 0 instead. |
| [`sale.order.line`](entities/sale.order.line.md) | `_update_line_quantity` | UserError | The ordered quantity of a sale order line cannot be decreased below the amount already delivered. Instead, create a return in your inventory. |
| [`sale.order.line`](entities/sale.order.line.md) | `_check_event_registration_ticket` | ValidationError | The sale order line with the product %(product_name)s needs an event, a ticket and a slot in case the event has multiple time slots. |
| [`sale.order.line`](entities/sale.order.line.md) | `_check_event_booth_registration_ids` | ValidationError | Registrations from the same Order Line must belong to a single event. |
| [`sale.order.line`](entities/sale.order.line.md) | `_update_event_booths` | ValidationError | The following booths are unavailable, please remove them to continue : %(booth_names)s |
| [`sale.order.line`](entities/sale.order.line.md) | `_check_validity` | UserError | The given product does not have a price therefore it cannot be added to cart. |
| [`sale.order.line`](entities/sale.order.line.md) | `_purchase_service_match_supplier` | UserError | There is no vendor associated to the product %s. Please define a vendor for this product. |
| [`sale.order.line`](entities/sale.order.line.md) | `_timesheet_service_generation` | UserError | A project must be defined on the quotation %(order)s or on the form of products creating a task on order. The following product need a project in which to put its task: %(product_name)s |
| [`sale.order.template`](entities/sale.order.template.md) | `_check_company_id` | ValidationError | Your template cannot contain products from specific companies if it's shared between companies. Please restrict the template access, or remove those products. |
| [`sale.order.template`](entities/sale.order.template.md) | `_check_company_id` | ValidationError | Your template belongs to company %(template_company)s but contains products from company (%(product_company)s) that are not accessible to %(template_company)s. Please change the company of your template or remove the products from other companies. |
| [`sale.order.template`](entities/sale.order.template.md) | `_check_company_id` | ValidationError | Your template belongs to company %(template_company)s but contains products from other companies (%(product_company)s) that are not accessible to %(template_company)s. Please change the company of your template or remove the products from other companies. |
| [`sale.order.template`](entities/sale.order.template.md) | `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. |
| [`sale.order.template.line`](entities/sale.order.template.line.md) | `write` | UserError | You cannot change the type of a sale quote line. Instead you should delete the current line and create a new line of the proper type. |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_form_field_name_follows_pattern` | ValidationError | Invalid form field name %(field_name)s. It should only contain alphanumerics, hyphens or underscores. |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_form_field_name_follows_pattern` | ValidationError | Invalid form field name %(field_name)s. A form field name in a header or a footer can not start with "sol_id_". |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_valid_and_existing_paths` | ValidationError | Invalid path %(path)s. It should only contain alphanumerics, hyphens, underscores or points. |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_valid_and_existing_paths` | ValidationError | Please use only relational fields until the last value of your path. |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_valid_and_existing_paths` | ValidationError | The field %(field_name)s doesn't exist on model %(model_name)s |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_document_type_and_document_linked_compatibility` | ValidationError | A form field set as used in product documents can't be linked to a quotation document. |
| [`sale.pdf.form.field`](entities/sale.pdf.form.field.md) | `_check_document_type_and_document_linked_compatibility` | ValidationError | A form field set as used in quotation documents can't be linked to a product document. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_constrains_date_sequence` | ValidationError | The %(date_field)s (%(date)s) you've entered isn't aligned with the existing sequence number (%(sequence)s). Clear the sequence number to proceed. To maintain date-based sequences, select entries and use the resequence option from the actions menu, available in developer mode. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_deduce_sequence_number_reset` | ValidationError | The sequence regex should at least contain the seq grouping keys. For instance: ^(?P<prefix1>.*?)(?P<seq>\d*)(?P<suffix>\D*?)$ |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_last_sequence` | ValidationError | %s is not a stored field |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_sequence_format_param` | ValidationError | Journal is not set for this invoice. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_sequence_format_param` | ValidationError | Business premises label is not set on the journal. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_sequence_format_param` | ValidationError | Issuing device label is not set on the journal. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_sequence_format_param` | ValidationError | Invalid sequence number format. |
| [`sequence.mixin`](entities/sequence.mixin.md) | `_get_sequence_format_param` | ValidationError | Invalid year format. |
| [`slide.channel`](entities/slide.channel.md) | `message_post` | AccessError | Not enough karma to review |
| [`slide.channel`](entities/slide.channel.md) | `message_post` | ValidationError | Only a single review can be posted per course. |
| [`slide.channel`](entities/slide.channel.md) | `_filter_add_members` | AccessError | You are not allowed to add members to this course. Please contact the course responsible or an administrator. |
| [`slide.channel`](entities/slide.channel.md) | `_send_share_email` | UserError | Impossible to send emails. Select a "Channel Share Template" for courses %(course_names)s first |
| [`slide.channel.invite`](entities/slide.channel.invite.md) | `action_invite` | UserError | Unable to post message, please configure the sender's email address. |
| [`slide.channel.invite`](entities/slide.channel.invite.md) | `action_invite` | UserError | Please select at least one recipient. |
| [`slide.question`](entities/slide.question.md) | `_check_answers_integrity` | ValidationError | All questions must have at least one correct answer and one incorrect answer:  %s |
| [`slide.slide`](entities/slide.slide.md) | `message_post` | AccessError | Not enough karma to comment |
| [`slide.slide`](entities/slide.slide.md) | `_send_share_email` | UserError | Impossible to send emails. Select a "Share Template" for courses %(course_names)s first |
| [`slide.slide`](entities/slide.slide.md) | `action_set_viewed` | UserError | You cannot mark a slide as viewed if you are not among its members. |
| [`slide.slide`](entities/slide.slide.md) | `action_mark_completed` | UserError | You cannot mark a slide as completed if you are not among its members. |
| [`slide.slide`](entities/slide.slide.md) | `action_mark_uncompleted` | UserError | You cannot mark a slide as uncompleted if you are not among its members. |
| [`slide.slide`](entities/slide.slide.md) | `_action_set_quiz_done` | UserError | _('You cannot mark a slide quiz as completed if you are not among its members or it is unpublished.') if completed else _('You cannot mark a slide quiz as not completed if you are not among its members or it is unpublished.') |
| [`slide.slide.resource`](entities/slide.slide.resource.md) | `_check_link_type` | ValidationError | Resource %(resource_name)s is a link and should not contain a data file |
| [`sms.account.code`](entities/sms.account.code.md) | `action_register` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) |
| [`sms.account.phone`](entities/sms.account.phone.md) | `action_send_verification_code` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) |
| [`sms.account.sender`](entities/sms.account.sender.md) | `_check_sender_name` | ValidationError | Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters. |
| [`sms.account.sender`](entities/sms.account.sender.md) | `action_set_sender_name` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) |
| [`sms.composer`](entities/sms.composer.md) | `_compute_sanitized_numbers` | UserError | Following numbers are not correctly encoded: %s |
| [`sms.composer`](entities/sms.composer.md) | `action_send_sms` | UserError | Invalid recipient number. Please update it. |
| [`sms.composer`](entities/sms.composer.md) | `action_send_sms` | UserError | %s invalid recipients |
| [`sms.twilio.account.manage`](entities/sms.twilio.account.manage.md) | `action_send_test` | UserError | Please set the number to which you want to send a test SMS. |
| [`snailmail.letter`](entities/snailmail.letter.md) | `_fetch_attachment` | UserError | Please use an A4 Paper format. |
| [`spreadsheet.dashboard.group`](entities/spreadsheet.dashboard.group.md) | `_unlink_except_spreadsheet_data` | UserError | You cannot delete %s as it is used in another module. |
| [`spreadsheet.mixin`](entities/spreadsheet.mixin.md) | `_check_spreadsheet_data` | ValidationError | Uh-oh! Looks like the spreadsheet file contains invalid data.  %(errors)s |
| [`spreadsheet.mixin`](entities/spreadsheet.mixin.md) | `_check_spreadsheet_data` | ValidationError | Uh-oh! Looks like the spreadsheet file contains invalid data. |
| [`stock.add.to.wave`](entities/stock.add.to.wave.md) | `default_get` | UserError | The selected transfers should belong to the same operation type |
| [`stock.add.to.wave`](entities/stock.add.to.wave.md) | `attach_pickings` | UserError | Cannot create wave transfers |
| [`stock.add.to.wave`](entities/stock.add.to.wave.md) | `attach_pickings` | UserError | The selected operations should belong to a unique company. |
| [`stock.add.to.wave`](entities/stock.add.to.wave.md) | `attach_pickings` | UserError | The selected transfers should belong to a unique company. |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | `button_cancel` | UserError | Validated landed costs cannot be cancelled, but you could create negative landed costs to reverse them |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | `button_validate` | UserError | Cost and adjustments lines do not match. You should maybe recompute the landed costs. |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | `get_valuation_lines` | UserError | You cannot apply landed costs on the chosen %s(s). Landed costs can only be applied for products with FIFO or average costing method. |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | `_check_can_validate` | UserError | Only draft landed costs can be validated |
| [`stock.landed.cost`](entities/stock.landed.cost.md) | `_check_can_validate` | UserError | Please define %s on which those additional costs should apply. |
| [`stock.location`](entities/stock.location.md) | `_compute_next_inventory_date` | UserError | The selected Inventory Frequency (Days) creates a date too far into the future. |
| [`stock.location`](entities/stock.location.md) | `_check_replenish_location` | ValidationError | Another parent/sub replenish location %s exists, if you wish to change it, uncheck it first |
| [`stock.location`](entities/stock.location.md) | `_check_scrap_location` | ValidationError | You cannot set a location as a scrap location when it is assigned as a destination location for a manufacturing type operation. |
| [`stock.location`](entities/stock.location.md) | `_unlink_except_master_data` | ValidationError | The %s location is required by the Inventory app and cannot be deleted, but you can archive it. |
| [`stock.location`](entities/stock.location.md) | `write` | UserError | This location's usage cannot be changed to view as it contains products. |
| [`stock.location`](entities/stock.location.md) | `write` | UserError | Internal locations having stock can't be converted |
| [`stock.location`](entities/stock.location.md) | `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. |
| [`stock.location`](entities/stock.location.md) | `write` | UserError | You can't disable locations %s because they still contain products. |
| [`stock.location`](entities/stock.location.md) | `write` | UserError | You cannot archive location %(location)s because it is used by warehouse %(warehouse)s |
| [`stock.location`](entities/stock.location.md) | `_check_subcontracting_location` | ValidationError | You cannot alter the company's subcontracting location |
| [`stock.location`](entities/stock.location.md) | `_check_subcontracting_location` | ValidationError | In order to manage stock accurately, subcontracting locations must be type Internal, linked to the appropriate company. |
| [`stock.lot`](entities/stock.lot.md) | `_check_unique_lot` | ValidationError | The combination of lot/serial number and product must be unique within a company including when no company is defined. The following combinations contain duplicates: %(error_lines)s |
| [`stock.lot`](entities/stock.lot.md) | `_check_create` | UserError | You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers". |
| [`stock.lot`](entities/stock.lot.md) | `_set_single_location` | UserError | You can only move a lot/serial to a new location if it exists in a single location. |
| [`stock.lot`](entities/stock.lot.md) | `write` | UserError | You are not allowed to change the product linked to a serial or lot number if some stock moves have already been created with that number. This would lead to inconsistencies in your stock. |
| [`stock.lot`](entities/stock.lot.md) | `write` | UserError | You cannot change the company of a lot/serial number currently in a location belonging to another company. |
| [`stock.lot`](entities/stock.lot.md) | `_check_create` | UserError | You are not allowed to create a lot or serial number with this operation type. To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers". |
| [`stock.lot`](entities/stock.lot.md) | `_check_create` | UserError | You are not allowed to create or edit a lot or serial number for the components with the operation type "Manufacturing". To change this, go on the operation type and tick the box "Create New Lots/Serial Numbers for Components". |
| [`stock.move`](entities/stock.move.md) | `_set_quantity` | UserError | '\n'.join(err) |
| [`stock.move`](entities/stock.move.md) | `_set_product_qty` | UserError | The requested operation cannot be processed because of a programming error setting the `product_qty` field instead of the `product_uom_qty`. |
| [`stock.move`](entities/stock.move.md) | `write` | UserError | You cannot change the UoM for a stock move that has been set to 'Done'. |
| [`stock.move`](entities/stock.move.md) | `write` | UserError | You cannot change a cancelled stock move, create a new line instead. |
| [`stock.move`](entities/stock.move.md) | `action_add_packages` | UserError | You need a transfer to add these packages to. |
| [`stock.move`](entities/stock.move.md) | `_do_unreserve` | UserError | You cannot unreserve a stock move that has been set to 'Done'. |
| [`stock.move`](entities/stock.move.md) | `_generate_serial_numbers` | ValidationError | The number of Serial Numbers to generate must be greater than zero. |
| [`stock.move`](entities/stock.move.md) | `action_generate_lot_line_vals` | UserError | No product found to generate Serials/Lots for. |
| [`stock.move`](entities/stock.move.md) | `action_generate_lot_line_vals` | UserError | The quantity per lot should always be a positive value. |
| [`stock.move`](entities/stock.move.md) | `_action_cancel` | UserError | You cannot cancel a stock move that has been set to 'Done'. Create a return in order to reverse the moves which took place. |
| [`stock.move`](entities/stock.move.md) | `_action_done` | UserError | error_msg + package_msg |
| [`stock.move`](entities/stock.move.md) | `_unlink_if_draft_or_cancel` | UserError | You can not delete moves linked to another operation |
| [`stock.move`](entities/stock.move.md) | `_split` | UserError | You cannot split a stock move that has been set to 'Done' or 'Cancel'. |
| [`stock.move`](entities/stock.move.md) | `_split` | UserError | You cannot split a draft move. It needs to be confirmed first. |
| [`stock.move`](entities/stock.move.md) | `_match_searched_availability` | UserError | Search not supported without a value. |
| [`stock.move`](entities/stock.move.md) | `_match_searched_availability` | UserError | Operation not supported |
| [`stock.move`](entities/stock.move.md) | `_match_searched_availability` | UserError | Selection not supported. |
| [`stock.move`](entities/stock.move.md) | `search_remaining_qty` | UserError | Only is set (= True) is supported in search for remaining_qty. |
| [`stock.move`](entities/stock.move.md) | `action_adjust_valuation` | UserError | You can only adjust valuation for one move at a time. |
| [`stock.move`](entities/stock.move.md) | `_set_value` | UserError | A lot/serial number is required for product '%s' as it has lot valuation enabled. |
| [`stock.move`](entities/stock.move.md) | `_add_mls_related_to_order` | UserError | '\n'.join(error_message_lines) |
| [`stock.move`](entities/stock.move.md) | `_check_negative_quantity` | ValidationError | Please enter a positive quantity. |
| [`stock.move`](entities/stock.move.md) | `_check_access_if_subcontractor` | AccessError | Portal users cannot create a stock move with a state 'Done' or change the current state to 'Done'. |
| [`stock.move`](entities/stock.move.md) | `_get_valuation_price_and_qty` | UserError | the system is not able to generate the anglo saxon entries. The total valuation of %s is zero. |
| [`stock.move`](entities/stock.move.md) | `_prepare_analytic_lines` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the manufacturing order. |
| [`stock.move`](entities/stock.move.md) | `_prepare_analytic_lines` | ValidationError | '%(missing_plan_names)s' analytic plan(s) required on the project '%(project_name)s' linked to the stock picking. |
| [`stock.move.line`](entities/stock.move.line.md) | `_check_lot_product` | ValidationError | This lot %(lot_name)s is incompatible with this product %(product_name)s |
| [`stock.move.line`](entities/stock.move.line.md) | `_check_positive_quantity` | ValidationError | You can not enter negative quantities. |
| [`stock.move.line`](entities/stock.move.line.md) | `_onchange_quantity` | UserError | You can only process 1.0 %s of products with unique serial number. |
| [`stock.move.line`](entities/stock.move.line.md) | `write` | UserError | Changing the product is only allowed in 'Draft' state. |
| [`stock.move.line`](entities/stock.move.line.md) | `write` | UserError | Changing the Lot/Serial number for move lines with different products is not allowed. |
| [`stock.move.line`](entities/stock.move.line.md) | `write` | UserError | Reserving a negative quantity is not allowed. |
| [`stock.move.line`](entities/stock.move.line.md) | `_unlink_except_done_or_cancel` | UserError | Deleting product moves after the transfer is done?  That would be like going back in time to revert all operations triggered after this move. Who knows what the end result would be, So let's not do it.  Try changing the “done” quantity to 0 instead. |
| [`stock.move.line`](entities/stock.move.line.md) | `_action_done` | UserError | You need to supply a Lot/Serial Number for product: %(products)s |
| [`stock.move.line`](entities/stock.move.line.md) | `_action_done` | UserError | The quantity done for the product "%(product)s" doesn't respect the rounding precision defined on the unit of measure "%(unit)s". Please change the quantity done or the rounding precision of your unit of measure. |
| [`stock.move.line`](entities/stock.move.line.md) | `_action_done` | UserError | No negative quantities allowed |
| [`stock.move.line`](entities/stock.move.line.md) | `_get_lines_and_packages_to_pack` | UserError | You cannot pack products into the same package when they are from different transfers with different operation types |
| [`stock.move.line`](entities/stock.move.line.md) | `_get_package_carrier_type_for_pack` | UserError | You cannot pack products into the same package when they have different carriers (i.e. check that all of their transfers have a carrier assigned and are using the same carrier). |
| [`stock.package`](entities/stock.package.md) | `write` | UserError | Cannot remove the location of a non empty package |
| [`stock.package`](entities/stock.package.md) | `write` | ValidationError | A package can't have one of its contained packages as destination container. |
| [`stock.package`](entities/stock.package.md) | `write` | UserError | Cannot move an empty package |
| [`stock.package`](entities/stock.package.md) | `_apply_dest_to_package` | UserError | Packages %(duplicate_names)s are moved to different locations while being in the same container %(container_name)s. |
| [`stock.package`](entities/stock.package.md) | `_apply_dest_to_package` | UserError | Can't move a container having packages in another location (%(old_location)s) to a different location (%(new_location)s). |
| [`stock.picking`](entities/stock.picking.md) | `_set_scheduled_date` | UserError | You cannot change the Scheduled Date on a cancelled transfer. |
| [`stock.picking`](entities/stock.picking.md) | `write` | UserError | Changing the operation type of this record is forbidden at this point. |
| [`stock.picking`](entities/stock.picking.md) | `action_assign` | UserError | Nothing to check the availability for. |
| [`stock.picking`](entities/stock.picking.md) | `_sanity_check` | UserError | You can’t validate an empty transfer. Please add some products to move before proceeding. |
| [`stock.picking`](entities/stock.picking.md) | `_sanity_check` | UserError | self._get_without_quantities_error_message() |
| [`stock.picking`](entities/stock.picking.md) | `_sanity_check` | UserError | You need to supply a Lot/Serial number for products %s. |
| [`stock.picking`](entities/stock.picking.md) | `_sanity_check` | UserError | message.lstrip() |
| [`stock.picking`](entities/stock.picking.md) | `action_split_transfer` | UserError | %s: Nothing to split. Fill the quantities you want in a new transfer in the done quantities |
| [`stock.picking`](entities/stock.picking.md) | `action_split_transfer` | UserError | %s: Nothing to split, all demand is done. For split you need at least one line not fully fulfilled |
| [`stock.picking`](entities/stock.picking.md) | `action_split_transfer` | UserError | %s: Can't split: quantities done can't be above demand |
| [`stock.picking`](entities/stock.picking.md) | `_check_backdate_allowed` | ValidationError | You cannot modify the scheduled date of operation %s because it falls within a locked fiscal period. |
| [`stock.picking`](entities/stock.picking.md) | `open_website_url` | UserError | Your delivery method has no redirect on courier provider's website to track this order. |
| [`stock.picking`](entities/stock.picking.md) | `l10n_ar_action_create_delivery_guide` | UserError | The delivery guide number %s exceeds the range specified in the CAI. Please update the range or use a different CAI with a different range. |
| [`stock.picking`](entities/stock.picking.md) | `l10n_ar_action_send_delivery_guide` | UserError | The partner does not have an email address. |
| [`stock.picking`](entities/stock.picking.md) | `action_l10n_in_ewaybill_create` | UserError | Please set HSN code in below products:  %s |
| [`stock.picking`](entities/stock.picking.md) | `action_l10n_in_ewaybill_create` | UserError | Ewaybill already created for this picking. |
| [`stock.picking`](entities/stock.picking.md) | `_l10n_ro_edi_stock_validate_carrier` | UserError | The picking %(picking_name)s is missing a delivery carrier. |
| [`stock.picking`](entities/stock.picking.md) | `_l10n_ro_edi_stock_validate_carrier` | UserError | The delivery carrier of %(picking_name)s is missing the partner field value. |
| [`stock.picking`](entities/stock.picking.md) | `action_generate_l10n_tr_edispatch_xml` | UserError | Error occurred in generating XML for following records: - %s |
| [`stock.picking`](entities/stock.picking.md) | `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s must be validated before validating the stock picking. |
| [`stock.picking`](entities/stock.picking.md) | `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s is cancelled. You cannot validate a stock picking on a cancelled Sales Order. |
| [`stock.picking`](entities/stock.picking.md) | `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s is currently locked. You cannot validate a stock picking on a locked Sales Order. Please create a new SO linked to this Project. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `_unlink_if_not_done` | UserError | You cannot delete Done batch transfers. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_confirm` | UserError | You have to set some pickings to batch. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_merge` | UserError | Please select at least two batch/wave transfers to merge. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_merge` | UserError | Batch/Wave transfers with different operation types cannot be merged. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_merge` | UserError | Batch transfers cannot be merged with wave transfers and vice versa. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_merge` | UserError | Batch/Wave transfers with different states cannot be merged. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_merge` | UserError | You cannot merge done or cancelled batch/wave transfers. |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `_sanity_check` | UserError | The following transfers cannot be added to batch transfer %(batch)s. Please check their states and operation types.  Incompatibilities: %(incompatible_transfers)s |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_done` | UserError | All Pickings in a Batch Transfer should have the same Carrier |
| [`stock.picking.batch`](entities/stock.picking.batch.md) | `action_done` | UserError | All Pickings in a Batch Transfer should have the same Commercial Partner |
| [`stock.picking.to.batch`](entities/stock.picking.to.batch.md) | `attach_pickings` | UserError | The selected pickings should belong to an unique company. |
| [`stock.picking.type`](entities/stock.picking.type.md) | `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. |
| [`stock.picking.type`](entities/stock.picking.type.md) | `_validate_auto_batch_group_by` | ValidationError | If the Automatic Batches feature is enabled, at least one 'Group by' option must be selected. |
| [`stock.picking.type`](entities/stock.picking.type.md) | `_check_active` | ValidationError | You cannot archive '%(picking_type)s' as it is used by POS configuration '%(config)s'. |
| [`stock.picking.type`](entities/stock.picking.type.md) | `_constrains_l10n_ar_sequence_number` | ValidationError | %(sequence_number)s is not a valid sequence number. Sequence numbers should contain exactly 8 digits (e.g. 00012345). |
| [`stock.picking.type`](entities/stock.picking.type.md) | `_onchange_sequence_code` | UserError | Only 3 characters are allowed in the Sequence Prefix by GİB |
| [`stock.picking.type`](entities/stock.picking.type.md) | `_check_default_location` | ValidationError | You cannot set a scrap location as the destination location for a manufacturing type operation. |
| [`stock.putaway.rule`](entities/stock.putaway.rule.md) | `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. |
| [`stock.quant`](entities/stock.quant.md) | `copy` | UserError | You cannot duplicate stock quants. |
| [`stock.quant`](entities/stock.quant.md) | `create` | UserError | Quant's creation is restricted, you can't do this operation. |
| [`stock.quant`](entities/stock.quant.md) | `write` | UserError | Quant's editing is restricted, you can't do this operation. |
| [`stock.quant`](entities/stock.quant.md) | `_unlink_except_wrong_permission` | UserError | Quants are auto-deleted when appropriate. If you must manually delete them, please ask a stock manager to do it. |
| [`stock.quant`](entities/stock.quant.md) | `action_stock_quant_relocate` | UserError | You can only move positive quantities stored in locations used by a single company per relocation. |
| [`stock.quant`](entities/stock.quant.md) | `check_product_id` | ValidationError | Quants cannot be created for consumables or services. |
| [`stock.quant`](entities/stock.quant.md) | `check_quantity` | ValidationError | The serial number has already been assigned:   Product: %(product)s, Serial Number: %(serial_number)s |
| [`stock.quant`](entities/stock.quant.md) | `check_location_id` | ValidationError | You cannot take products from or deliver products to a location of type "view" (%s). |
| [`stock.quant`](entities/stock.quant.md) | `check_lot_id` | ValidationError | The Lot/Serial number (%s) is linked to another product. |
| [`stock.quant`](entities/stock.quant.md) | `_get_removal_strategy_order` | UserError | Removal strategy %s not implemented. |
| [`stock.quant`](entities/stock.quant.md) | `_get_reserve_quantity` | UserError | It is not possible to unreserve more products of %s than you have in stock. |
| [`stock.quant`](entities/stock.quant.md) | `_update_available_quantity` | ValidationError | Quantity or Reserved Quantity should be set. |
| [`stock.quant`](entities/stock.quant.md) | `_check_kits` | UserError | You should update the components quantity instead of directly updating the quantity of the kit product. |
| [`stock.return.picking`](entities/stock.return.picking.md) | `default_get` | UserError | You may only return one picking at a time. |
| [`stock.return.picking`](entities/stock.return.picking.md) | `_compute_moves_locations` | UserError | You may only return Done pickings. |
| [`stock.return.picking`](entities/stock.return.picking.md) | `_compute_moves_locations` | UserError | No products to return (only lines in Done state and not fully returned yet can be returned). |
| [`stock.return.picking`](entities/stock.return.picking.md) | `_create_return` | UserError | Please specify at least one non-zero quantity. |
| [`stock.route`](entities/stock.route.md) | `_check_company_consistency` | ValidationError | Rule %(rule)s belongs to %(rule_company)s while the route belongs to %(route_company)s. |
| [`stock.rule`](entities/stock.rule.md) | `_check_company_consistency` | ValidationError | Rule %(rule)s belongs to %(rule_company)s while the route belongs to %(route_company)s. |
| [`stock.rule`](entities/stock.rule.md) | `run` | UserError | '\n'.join(errors) |
| [`stock.scrap`](entities/stock.scrap.md) | `_unlink_except_done` | UserError | You cannot delete a scrap which is done. |
| [`stock.scrap`](entities/stock.scrap.md) | `action_validate` | UserError | You can only enter positive quantities. |
| [`stock.valuation.adjustment.lines`](entities/stock.valuation.adjustment.lines.md) | `_create_accounting_entries` | UserError | Please configure Stock Expense Account for product: %s. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_warehouse_redirect_warning` | RedirectWarning | msg |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_warehouse_redirect_warning` | UserError | Please contact your administrator to configure your warehouse. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `write` | UserError | You still have ongoing operations for operation types %(operations)s in warehouse %(warehouse)s |
| [`stock.warehouse`](entities/stock.warehouse.md) | `write` | UserError | %(operations)s have default source or destination locations within warehouse %(warehouse)s, therefore you cannot archive it. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_find_or_create_global_route` | UserError | Can't find any generic route %s. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_get_partner_locations` | UserError | Can't find any customer or supplier location. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_get_picking_type_create_values` | UserError | No location of type Inventory Loss found |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_get_production_location` | UserError | Can't find any production location. |
| [`stock.warehouse`](entities/stock.warehouse.md) | `_get_production_location` | UserError | Can't find any production location. |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `_check_min_max_qty` | ValidationError | The minimum quantity must be less than or equal to the maximum quantity. |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `create` | UserError | You can not create a snoozed orderpoint that is not manually triggered. |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `write` | UserError | You can only snooze manual orderpoints. You should rather archive 'auto-trigger' orderpoints if you do not want them to be triggered. |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `action_replenish` | RedirectWarning | e |
| [`stock.warehouse.orderpoint`](entities/stock.warehouse.orderpoint.md) | `check_product_is_not_kit` | ValidationError | A product with a kit-type bill of materials can not have a reordering rule. |
| [`survey.invite`](entities/survey.invite.md) | `_onchange_emails` | UserError | This survey does not allow external people to participate. You should create user accounts or update survey access mode accordingly. |
| [`survey.invite`](entities/survey.invite.md) | `_onchange_emails` | UserError | Some emails you just entered are incorrect: %s |
| [`survey.invite`](entities/survey.invite.md) | `_onchange_partner_ids` | UserError | The following recipients have no user account: %s. You should create user accounts for them or allow external signup in configuration. |
| [`survey.invite`](entities/survey.invite.md) | `_send_mail` | UserError | Unable to post message, please configure the sender's email address. |
| [`survey.invite`](entities/survey.invite.md) | `action_invite` | UserError | Please enter at least one valid recipient. |
| [`survey.question`](entities/survey.question.md) | `_check_question_type_for_pages` | ValidationError | Question type should be empty for these pages: %s |
| [`survey.question`](entities/survey.question.md) | `_unlink_except_live_sessions_in_progress` | UserError | You cannot delete questions from surveys "%(survey_names)s" while live sessions are in progress. |
| [`survey.question.answer`](entities/survey.question.answer.md) | `_check_question_not_empty` | ValidationError | A label must be attached to only one question. |
| [`survey.survey`](entities/survey.survey.md) | `_check_scoring_after_page_availability` | ValidationError | Combining roaming and "Scoring with answers after each page" is not possible; please update the following surveys: - %(survey_names)s |
| [`survey.survey`](entities/survey.survey.md) | `_check_survey_responsible_access` | ValidationError | The access of the following surveys is restricted. Make sure their responsible still has access to it:  %(survey_names)s |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | Creating token for closed/archived surveys is not allowed. |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | Creating token for anybody else than employees is not allowed for internal surveys. |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | No attempts left. |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | Creating test token is not allowed for you. |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | Creating token for external people is not allowed for surveys requesting authentication. |
| [`survey.survey`](entities/survey.survey.md) | `_check_answer_creation` | UserError | Creating token for external people is not allowed for surveys requesting authentication. |
| [`survey.survey`](entities/survey.survey.md) | `check_validity` | UserError | You cannot send an invitation for a survey that has no questions. |
| [`survey.survey`](entities/survey.survey.md) | `check_validity` | UserError | A scored survey needs at least one question that gives points. Please check answers and their scores. |
| [`survey.survey`](entities/survey.survey.md) | `check_validity` | UserError | You cannot send invitations for closed surveys. |
| [`survey.survey`](entities/survey.survey.md) | `check_validity` | UserError | You cannot send an invitation for a "One page per section" survey if the survey has no sections. |
| [`survey.survey`](entities/survey.survey.md) | `check_validity` | UserError | You cannot send an invitation for a "One page per section" survey if the survey only contains empty sections. |
| [`survey.survey`](entities/survey.survey.md) | `action_start_session` | AccessError | Only survey users can manage sessions. |
| [`survey.survey`](entities/survey.survey.md) | `action_end_session` | AccessError | Only survey users can manage sessions. |
| [`survey.survey`](entities/survey.survey.md) | `_unlink_except_linked_to_course` | ValidationError | Uh-oh! You can’t delete surveys used as a Course Certification! Otherwise, students might think diplomas just grow on trees. The courses that need them are: %s |
| [`survey.user_input`](entities/survey.user_input.md) | `_save_lines` | UserError | This answer cannot be overwritten. |
| [`survey.user_input.line`](entities/survey.user_input.line.md) | `_check_answer_type_skipped` | ValidationError | A question can either be skipped or answered, not both. |
| [`survey.user_input.line`](entities/survey.user_input.line.md) | `_check_answer_type_skipped` | ValidationError | The answer must be in the right type |
| [`template.reset.mixin`](entities/template.reset.mixin.md) | `reset_template` | UserError | The following email templates could not be reset because their related source files could not be found: - %s |
| [`uom.uom`](entities/uom.uom.md) | `_check_factor` | UserError | Reference unit of measure is missing. |
| [`uom.uom`](entities/uom.uom.md) | `_unlink_except_master_data` | UserError | The following units of measure are used by the system and cannot be deleted: %s You can archive them instead. |
| [`uom.uom`](entities/uom.uom.md) | `write` | UserError | error_msg |
| [`uom.uom`](entities/uom.uom.md) | `write` | UserError | error_msg |
| [`uom.uom`](entities/uom.uom.md) | `write` | UserError | error_msg |
| [`utm.campaign`](entities/utm.campaign.md) | `_unlink_except_utm_campaign_job` | UserError | The UTM campaign '%s' cannot be deleted as it is used in the recruitment process. |
| [`utm.medium`](entities/utm.medium.md) | `_unlink_except_utm_medium_record` | UserError | Oops, you can't delete the Medium '%s'. Doing so would be like tearing down a load-bearing wall — not the best idea. |
| [`utm.medium`](entities/utm.medium.md) | `_unlink_except_linked_mailings` | UserError | You cannot delete these UTM Mediums as they are linked to the following mailings in Mass Mailing: %(mailing_names)s |
| [`utm.medium`](entities/utm.medium.md) | `_unlink_except_utm_medium_sms` | UserError | The UTM medium '%s' cannot be deleted as it is used in some main functional flows, such as the SMS Marketing. |
| [`utm.source`](entities/utm.source.md) | `_unlink_except_referral` | ValidationError | You cannot delete the 'Referral' UTM source record. |
| [`utm.source`](entities/utm.source.md) | `_unlink_except_linked_recruitment_sources` | UserError | You cannot delete these UTM Sources as they are linked to the following recruitment sources in Recruitment: %(recruitment_sources)s |
| [`utm.source`](entities/utm.source.md) | `_unlink_except_linked_mailings` | UserError | You cannot delete these UTM Sources as they are linked to the following mailings in Mass Mailing: %(mailing_names)s |
| [`utm.source`](entities/utm.source.md) | `_unlink_except_utm_source_marketing_card` | UserError | The UTM source '%s' cannot be deleted as it is used to promote marketing cards campaigns. |
| [`validate.account.move`](entities/validate.account.move.md) | `default_get` | UserError | There are no journal items in the draft state to post. |
| [`validate.account.move`](entities/validate.account.move.md) | `default_get` | UserError | Missing 'active_model' in context. |
| [`website`](entities/website.md) | `_check_domain` | ValidationError | The domain path cannot contain relative path segments like '/./' or '/../'. |
| [`website`](entities/website.md) | `_check_domain` | ValidationError | The provided website domain is not a valid URL. |
| [`website`](entities/website.md) | `_check_homepage_url` | ValidationError | The homepage URL should be relative and start with '/'. |
| [`website`](entities/website.md) | `_unlink_except_default_website` | UserError | You cannot delete default website %s. Try to change its settings instead |
| [`website`](entities/website.md) | `get_website_page_ids` | AccessError | Access Denied |
| [`website`](entities/website.md) | `action_dashboard_redirect` | AccessError | You don't have the necessary access rights to access this dashboard. |
| [`website`](entities/website.md) | `_check_events_app_name` | ValidationError | "Events App Name" field is required. |
| [`website.configurator.feature`](entities/website.configurator.feature.md) | `_check_module_xor_page_view` | ValidationError | One and only one of the two fields 'page_view_id' and 'module_id' should be set |
| [`website.controller.page`](entities/website.controller.page.md) | `_check_user_has_model_access` | ValidationError | A page must be set to display a concrete model. |
| [`website.custom_blocked_third_party_domains`](entities/website.custom_blocked_third_party_domains.md) | `action_save` | ValidationError | _('The following domain is not valid:') + '\n' + domain |
| [`website.menu`](entities/website.menu.md) | `_validate_parent_menu` | UserError | A mega menu cannot have a parent or child menu. |
| [`website.menu`](entities/website.menu.md) | `_validate_parent_menu` | UserError | Menus with child menus cannot be added as a submenu. |
| [`website.menu`](entities/website.menu.md) | `_validate_parent_menu` | UserError | Menus cannot have more than two levels of hierarchy. |
| [`website.menu`](entities/website.menu.md) | `_unlink_except_master_tags` | UserError | You cannot delete this website menu as this serves as the default parent menu for new websites (e.g., /shop, /event, ...). |
| [`website.published.mixin`](entities/website.published.mixin.md) | `create` | AccessError | self._get_can_publish_error_message() |
| [`website.published.mixin`](entities/website.published.mixin.md) | `write` | AccessError | self._get_can_publish_error_message() |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" can not be empty. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL from" can not be empty. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | URL must not start with '#'. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | base URL of 'URL to' should not be same as 'URL from'. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" must start with a leading slash. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" cannot be set to "/". To change the homepage content, use the "Homepage URL" field in the website settings or the page properties on any custom page. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" cannot be set to an existing page. |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" must contain parameter %s used in "URL from". |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" cannot contain parameter %s which is not used in "URL from". |
| [`website.rewrite`](entities/website.rewrite.md) | `_check_url_to` | ValidationError | "URL to" is invalid: %s |
| [`website.snippet.filter`](entities/website.snippet.filter.md) | `_check_data_source_is_provided` | ValidationError | Either action_server_id or filter_id must be provided. |
| [`website.snippet.filter`](entities/website.snippet.filter.md) | `_check_limit` | ValidationError | The limit must be between 1 and 16. |
| [`website.snippet.filter`](entities/website.snippet.filter.md) | `_check_field_names` | ValidationError | Empty field name in “%s” |
| [`website.visitor`](entities/website.visitor.md) | `action_send_mail` | UserError | There are no contact and/or no email linked to this visitor. |
| [`website.visitor`](entities/website.visitor.md) | `_search_event_registered_ids` | UserError | Unsupported 'Not In' operation on visitors registrations |
| [`website.visitor`](entities/website.visitor.md) | `_search_event_track_wishlisted_ids` | UserError | Unsupported 'Not In' operation on track wishlist visitors |
| [`website.visitor`](entities/website.visitor.md) | `action_send_chat_request` | UserError | Recipients are not available. Please refresh the page to get latest visitors status. |
| [`website.visitor`](entities/website.visitor.md) | `action_send_chat_request` | UserError | No Livechat Channel allows you to send a chat request for website %s. |
| [`website.visitor`](entities/website.visitor.md) | `action_send_sms` | UserError | There are no contact and/or no phone or mobile numbers linked to this visitor. |
