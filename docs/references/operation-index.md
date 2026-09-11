# Operation index

Every operation catalogued on every entity: 18,344 in total. Operations are classified by what they are for, and the classification is how a rebuild decides which to expose and which are internal.

| Kind | Meaning | Count |
|---|---|---|
| internal rule | Internal helper that other operations call | 5,485 |
| preparation rule | Assembles the values for a record that another operation is about to create | 3,636 |
| computation | Derives the value of one or more computed fields from declared dependencies | 3,589 |
| operation | General operation | 1,387 |
| user action | An operation a person invokes, usually from a button on a form or a list | 1,294 |
| lifecycle override | Extends a generic operation such as creation, update, deletion, copying or reading | 926 |
| validation | Refuses a change that would break an invariant, with a message | 724 |
| on change | Adjusts other values in a form while a person edits, before anything is saved | 437 |
| search rule | Makes a non-stored field searchable by translating a condition into one over stored fields | 337 |
| messaging hook | Participates in threads, followers, notifications or tracking | 191 |
| inverse computation | Makes a computed field writable by deciding what to store when it is set | 176 |
| background operation | Runs from a scheduled job or a queue rather than from a person | 162 |

## By domain

| Domain | Operations | Entities | Validation messages raised |
|---|---|---|---|
| accounts-payable | 2 | 1 | 1 |
| accounts-receivable | 11 | 2 | 4 |
| analytic-accounting | 168 | 7 | 29 |
| attendances-and-working-time | 194 | 10 | 15 |
| automation-and-integration | 216 | 23 | 30 |
| calendar-and-scheduling | 328 | 17 | 37 |
| contacts-and-organizations | 22 | 2 | 0 |
| customer-relationship-management | 328 | 31 | 31 |
| delivery-and-shipping | 77 | 4 | 14 |
| electronic-invoicing-and-document-exchange | 974 | 27 | 82 |
| events | 396 | 32 | 29 |
| expenses | 113 | 6 | 32 |
| financial-reporting | 2 | 1 | 0 |
| fiscal-localizations | 712 | 78 | 94 |
| fleet | 95 | 11 | 4 |
| general-ledger | 2,940 | 51 | 462 |
| human-resources-core | 476 | 29 | 45 |
| identity-and-access | 33 | 7 | 6 |
| inventory-operations | 1,260 | 52 | 147 |
| inventory-valuation-and-costing | 35 | 6 | 6 |
| learning-surveys-and-gamification | 429 | 24 | 54 |
| loyalty-and-promotions | 123 | 10 | 23 |
| lunch-ordering | 67 | 8 | 8 |
| manufacturing | 492 | 23 | 78 |
| marketing-and-mass-mailing | 237 | 19 | 18 |
| messaging-and-activities | 1,277 | 82 | 142 |
| multi-currency | 2,851 | 112 | 524 |
| payment-providers | 272 | 6 | 59 |
| payments-and-bank-reconciliation | 25 | 3 | 7 |
| point-of-sale | 713 | 28 | 192 |
| pricing-and-pricelists | 1 | 1 | 0 |
| products-and-catalog | 784 | 30 | 110 |
| projects-and-tasks | 529 | 21 | 24 |
| purchasing | 265 | 11 | 30 |
| recruitment | 96 | 10 | 10 |
| repair-and-maintenance | 104 | 8 | 10 |
| replenishment-and-procurement | 3 | 1 | 0 |
| sales | 697 | 14 | 83 |
| spreadsheets-and-dashboards | 25 | 5 | 3 |
| taxes | 48 | 4 | 4 |
| time-off | 285 | 15 | 56 |
| timesheets | 25 | 3 | 0 |
| units-of-measure-and-packaging | 25 | 1 | 5 |
| website-and-storefront | 547 | 46 | 72 |
| work-entries | 42 | 3 | 10 |

## Operations a person invokes

These are the named business operations a client binds to a button. A rebuild must expose each of them, because they are part of the observable contract.

| Entity | Operation | Arguments | Packages | Purpose |
|---|---|---|---|---|
| `account.account` | `action_open_related_taxes` | self | `account` |  |
| `account.account` | `action_unmerge` | self | `account` | Split the account `self` into several accounts, one per company. The original account's codes are assigned respectively to the account created in each company.  From an accounting perspective, this does not change anything to the journal items, since their account codes will remain unchanged. |
| `account.analytic.account` | `action_view_invoice` | self | `account` |  |
| `account.analytic.account` | `action_view_mrp_bom` | self | `mrp_account` |  |
| `account.analytic.account` | `action_view_mrp_production` | self | `mrp_account` |  |
| `account.analytic.account` | `action_view_projects` | self | `project` |  |
| `account.analytic.account` | `action_view_purchase_orders` | self | `purchase` |  |
| `account.analytic.account` | `action_view_vendor_bill` | self | `account` |  |
| `account.analytic.account` | `action_view_workorder` | self | `mrp_account` |  |
| `account.analytic.line` | `action_invoice_from_timesheet` | self | `sale_timesheet` |  |
| `account.analytic.line` | `action_open_timesheet_view_portal` | self | `hr_timesheet` |  |
| `account.analytic.line` | `action_sale_order_from_timesheet` | self | `sale_timesheet` |  |
| `account.analytic.plan` | `action_view_analytical_accounts` | self | `analytic` |  |
| `account.analytic.plan` | `action_view_children_plans` | self | `analytic` |  |
| `account.automatic.entry.wizard` | `do_action` | self | `account` |  |
| `account.autopost.bills.wizard` | `action_ask_later` | self | `account` |  |
| `account.autopost.bills.wizard` | `action_automate_partner` | self | `account` |  |
| `account.autopost.bills.wizard` | `action_never_automate_partner` | self | `account` |  |
| `account.bank.statement.line` | `action_undo_reconciliation` | self | `account` | Undo the reconciliation made on the statement line and reset their journal items to their original states. |
| `account.edi.document` | `action_export_xml` | self | `account_edi` |  |
| `account.financial.year.op` | `action_save_onboarding_fiscal_year` | self | `account` |  |
| `account.fiscal.position` | `action_create_foreign_taxes` | self | `account` |  |
| `account.fiscal.position` | `action_open_related_taxes` | self | `account` |  |
| `account.journal` | `action_checks_to_print` | self | `account_check_printing` |  |
| `account.journal` | `action_configure_bank_journal` | self | `account` | This function is called by the "configure" button of bank journals, visible on dashboard if no bank statement source has been defined yet |
| `account.journal` | `action_create_new` | self | `account` |  |
| `account.journal` | `action_create_vendor_bill` | self | `account` | This function is called by the "try our sample" button of Vendor Bills, visible on dashboard if no bill has been created yet. |
| `account.journal` | `action_post_all_entries` | self | `account` |  |
| `account.journal` | `button_fetch_in_einvoices` | self | `account`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_pl_edi`, `l10n_tr_nilvera_einvoice` | Abstract method to fetch e-invoices. Should fetch vendor bill invoices synchronously and doesn't return anything. |
| `account.journal` | `button_refresh_out_einvoices_status` | self | `account`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_tr_nilvera_einvoice` | Abstract method to fetch e-invoice statuses. Should fetch customer invoices statuses synchronously and doesn't return anything. |
| `account.journal` | `button_unsubscribe_from_invoice_notifications` | self | `account` |  |
| `account.lock_exception` | `action_revoke` | self | `account` | Revokes an active exception. |
| `account.lock_exception` | `action_show_audit_trail_during_exception` | self | `account` |  |
| `account.merge.wizard` | `action_merge` | self | `account` | Merge each group of accounts in `self.wizard_line_ids`. |
| `account.move` | `action_activate_currency` | self | `account` |  |
| `account.move` | `action_add_from_catalog` | self | `account` |  |
| `account.move` | `action_cancel_nemhandel_documents` | self | `l10n_dk_nemhandel` |  |
| `account.move` | `action_cancel_peppol_documents` | self | `account_peppol` |  |
| `account.move` | `action_check_l10n_it_edi` | self | `l10n_it_edi` |  |
| `account.move` | `action_debit_note` | self | `account_debit_note` |  |
| `account.move` | `action_delete_duplicates` | self | `account` |  |
| `account.move` | `action_duplicate` | self | `account` |  |
| `account.move` | `action_export_l10n_in_edi_content_json` | self | `l10n_in_edi` |  |
| `account.move` | `action_force_register_payment` | self | `account` |  |
| `account.move` | `action_get_eta_invoice_pdf` | self | `l10n_eg_edi_eta` | This is a pdf with the structure from the government.  While we can use our own format, some clients appreciate this to verify that all the data is there in case of confusion. |
| `account.move` | `action_group_ungroup_lines_by_tax` | self | `account_edi_ubl_cii` | This action allows the user to reload an imported move, grouping or not lines by tax |
| `account.move` | `action_invoice_download_facturae` | self | `l10n_es_edi_facturae` |  |
| `account.move` | `action_invoice_download_fatturapa` | self | `l10n_it_edi` |  |
| `account.move` | `action_invoice_download_pdf` | self, target | `account` |  |
| `account.move` | `action_invoice_download_ubl` | self | `account_edi_ubl_cii` |  |
| `account.move` | `action_invoice_sent` | self | `account`, `l10n_my_edi` | Open a window to compose an email, with the edi invoice template message loaded by default |
| `account.move` | `action_l10n_id_update_payment_status` | self | `l10n_id` | This action will:     - Get all invoices that are not paid, and have details about QRIS qr codes.     - For each invoices, get information about the payment state of the QR using the API.     - If the QR is not paid and it has been more than 30m, we discard that qr id (no longer valid)     - If it i |
| `account.move` | `action_l10n_in_apply_higher_tax` | self | `l10n_in` |  |
| `account.move` | `action_l10n_in_edi_force_cancel` | self | `l10n_in_edi` |  |
| `account.move` | `action_l10n_in_ewaybill_create` | self | `l10n_in_ewaybill` |  |
| `account.move` | `action_l10n_in_withholding_entries` | self | `l10n_in` |  |
| `account.move` | `action_l10n_it_edi_send` | self | `l10n_it_edi` | Checks that the invoice data is coherent. Attaches the XML file to the invoice. Sends the invoice to the SdI. |
| `account.move` | `action_l10n_my_edi_send_invoice` | self | `l10n_my_edi` | This action will create the MyInvois Document for this invoice if it does not already exist. Once done, it will trigger the sending of said document to the platform. |
| `account.move` | `action_l10n_my_edi_update_status` | self | `l10n_my_edi` |  |
| `account.move` | `action_l10n_pl_edi_get_invoice_UPO` | self | `l10n_pl_edi` |  |
| `account.move` | `action_l10n_pl_edi_update_invoice_status` | self | `l10n_pl_edi` |  |
| `account.move` | `action_l10n_ro_edi_fetch_invoices` | self | `l10n_ro_edi` |  |
| `account.move` | `action_l10n_vn_edi_update_payment_status` | self | `l10n_vn_edi_viettel` | Send a request to update the payment status of the invoice. |
| `account.move` | `action_move_download_all` | self | `account` |  |
| `account.move` | `action_nemhandel_open_rejection_wizard` | self | `l10n_dk_nemhandel_response` |  |
| `account.move` | `action_nemhandel_send_approval_response` | self | `l10n_dk_nemhandel_response` |  |
| `account.move` | `action_open_business_doc` | self | `account` |  |
| `account.move` | `action_open_declaration_of_intent` | self | `l10n_it_edi_doi` |  |
| `account.move` | `action_open_expense` | self | `hr_expense` |  |
| `account.move` | `action_open_l10n_in_ewaybill` | self | `l10n_in_ewaybill` |  |
| `account.move` | `action_open_l10n_ph_2307_wizard` | self | `l10n_ph` |  |
| `account.move` | `action_open_nemhandel_reponses` | self | `l10n_dk_nemhandel_response` |  |
| `account.move` | `action_open_peppol_reponses` | self | `account_peppol_response` |  |
| `account.move` | `action_pdp_open_response_wizard` | self, **wizard_kwargs | `l10n_fr_pdp` |  |
| `account.move` | `action_peppol_cancel_and_remove_sequence` | self | `account_peppol` |  |
| `account.move` | `action_peppol_open_rejection_wizard` | self | `account_peppol_response` |  |
| `account.move` | `action_peppol_reset_documents` | self, ids_to_delete | `account_peppol` |  |
| `account.move` | `action_peppol_send_approval_response` | self | `account_peppol_response` |  |
| `account.move` | `action_post` | self | `account`, `pos_sale`, `sale`, `sale_timesheet` |  |
| `account.move` | `action_post_sign_invoices` | self | `l10n_eg_edi_eta` |  |
| `account.move` | `action_print_pdf` | self | `account` |  |
| `account.move` | `action_process_edi_web_services` | self, with_commit | `account_edi` |  |
| `account.move` | `action_purchase_matching` | self | `purchase` |  |
| `account.move` | `action_register_payment` | self | `account` |  |
| `account.move` | `action_retry_edi_documents_error` | self | `account_edi` |  |
| `account.move` | `action_reverse` | self | `account` |  |
| `account.move` | `action_send_and_print` | self | `account`, `account_peppol`, `l10n_dk_nemhandel` |  |
| `account.move` | `action_show_chain_head` | self | `l10n_sa_edi` | Action to show the chain head of the invoice |
| `account.move` | `action_show_myinvois_documents` | self | `l10n_my_edi` |  |
| `account.move` | `action_switch_move_type` | self | `account` |  |
| `account.move` | `action_toggle_block_payment` | self | `account` |  |
| `account.move` | `action_update_fpos_values` | self | `account` |  |
| `account.move` | `action_validate_moves_with_confirmation` | self | `account` | If 'restrict_mode_hash_table' is enabled or future-dated moves, open a confirmation wizard; otherwise, validate moves directly. |
| `account.move` | `action_view_debit_notes` | self | `account_debit_note` |  |
| `account.move` | `action_view_landed_costs` | self | `stock_landed_costs` |  |
| `account.move` | `action_view_payment_transactions` | self | `account_payment` |  |
| `account.move` | `action_view_source_pos_orders` | self | `point_of_sale` |  |
| `account.move` | `action_view_source_purchase_orders` | self | `purchase` |  |
| `account.move` | `action_view_source_sale_orders` | self | `sale` |  |
| `account.move` | `action_view_timesheet` | self | `sale_timesheet` |  |
| `account.move` | `action_view_wip_production` | self | `mrp_account` |  |
| `account.move` | `button_abandon_cancel_posted_posted_moves` | self | `account_edi` | Cancel the request for cancellation of the EDI. |
| `account.move` | `button_cancel` | self | `account`, `account_edi`, `account_peppol_response`, `hr_expense`, `l10n_dk_nemhandel_response`, `l10n_fr_pdp`, `pos_sale`, `sale`, `stock_account` |  |
| `account.move` | `button_cancel_posted_moves` | self | `account_edi` | Mark the edi.document related to this move to be canceled. |
| `account.move` | `button_create_landed_costs` | self | `stock_landed_costs` | Create a `stock.landed.cost` record associated to the account move of `self`, each `stock.landed.costs` lines mirroring the current `account.move.line` of self. |
| `account.move` | `button_draft` | self | `account`, `account_edi`, `l10n_eg_edi_eta`, `l10n_es_edi_tbai`, `l10n_fr_pdp`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_latam_check`, `l10n_pl_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `point_of_sale`, `purchase_stock`, `sale`, `sale_expense`, `stock_account` | When going from canceled => draft, we ensure to clear the edi fields so that the invoice can be resent if required. |
| `account.move` | `button_force_cancel` | self | `account_edi` | Cancel the invoice without waiting for the cancellation request to succeed. |
| `account.move` | `button_hash` | self | `account` |  |
| `account.move` | `button_process_edi_web_services` | self | `account_edi` |  |
| `account.move` | `button_request_cancel` | self | `account`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_my_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | Hook allowing the localizations to request a cancellation from the government before cancelling the invoice. |
| `account.move` | `button_set_checked` | self | `account` |  |
| `account.move.line` | `action_add_from_catalog` | self | `account` | Will open the catalog view |
| `account.move.line` | `action_automatic_entry` | self, default_action | `account` |  |
| `account.move.line` | `action_open_business_doc` | self | `account` |  |
| `account.move.line` | `action_payment_items_register_payment` | self | `account` |  |
| `account.move.line` | `action_register_payment` | self, ctx | `account` | Open the account.payment.register wizard to pay the selected journal items. :return: An action opening the account.payment.register wizard. |
| `account.move.line` | `action_unreconcile_match_entries` | self | `account` | This method will do the unreconcile action in the list view of the moves |
| `account.move.send` | `action_what_is_peppol_activate` | self, moves | `account_peppol`, `l10n_fr_pdp` |  |
| `account.move.send.batch.wizard` | `action_send_and_print` | self, force_synchronous, allow_fallback_pdf | `account`, `account_peppol`, `l10n_dk_nemhandel` | Launch asynchronously the generation and sending of invoices. |
| `account.move.send.wizard` | `action_send_and_print` | self, allow_fallback_pdf | `account`, `account_peppol`, `l10n_dk_nemhandel`, `l10n_es_edi_facturae`, `l10n_ke_edi_tremol` | Create invoice documents and send them. |
| `account.payment` | `action_cancel` | self | `account`, `l10n_latam_check` |  |
| `account.payment` | `action_draft` | self | `account`, `l10n_latam_check` |  |
| `account.payment` | `action_l10n_in_withholding_entries` | self | `l10n_in` |  |
| `account.payment` | `action_open_business_doc` | self | `account` |  |
| `account.payment` | `action_open_expense` | self | `hr_expense` |  |
| `account.payment` | `action_open_l10n_ph_2307_wizard` | self | `l10n_ph` |  |
| `account.payment` | `action_post` | self | `account`, `account_check_printing`, `account_payment`, `l10n_latam_check` | draft -> posted |
| `account.payment` | `action_refund_wizard` | self | `account_payment` |  |
| `account.payment` | `action_reject` | self | `account` |  |
| `account.payment` | `action_validate` | self | `account` |  |
| `account.payment` | `action_view_pos_order` | self | `pos_online_payment` | Return the action for the view of the pos order linked to the payment. |
| `account.payment` | `action_view_refunds` | self | `account_payment` |  |
| `account.payment` | `action_void_check` | self | `account_check_printing` |  |
| `account.payment` | `button_open_bills` | self | `account` | Redirect the user to the bill(s) paid by this payment. :return:    An action on account.move. |
| `account.payment` | `button_open_invoices` | self | `account` | Redirect the user to the invoice(s) paid by this payment. :return:    An action on account.move. |
| `account.payment` | `button_open_journal_entry` | self | `account` | Redirect the user to this payment journal. :return:    An action on account.move. |
| `account.payment` | `button_open_statement_lines` | self | `account` | Redirect the user to the statement line(s) reconciled to this payment. :return:    An action on account.move. |
| `account.payment` | `button_request_cancel` | self | `account` |  |
| `account.payment` | `do_print_checks` | self | `account_check_printing` |  |
| `account.payment.method.line` | `action_open_provider_form` | self | `account_payment` |  |
| `account.payment.register` | `action_create_payments` | self | `account`, `l10n_ar_withholding`, `l10n_latam_check` |  |
| `account.payment.register` | `action_open_missing_account_partners` | self | `account` |  |
| `account.payment.register` | `action_open_untrusted_bank_accounts` | self | `account` |  |
| `account.peppol.rejection.wizard` | `button_send` | self | `account_peppol_response` |  |
| `account.reconcile.model` | `action_reconcile_stat` | self | `account` |  |
| `account.reconcile.model` | `action_set_auto_reconcile` | self | `account` |  |
| `account.reconcile.model` | `action_set_manual` | self | `account` |  |
| `account.secure.entries.wizard` | `action_secure_entries` | self | `account` |  |
| `account.secure.entries.wizard` | `action_show_draft_moves_in_hashed_period` | self | `account` |  |
| `account.secure.entries.wizard` | `action_show_moves` | self, moves | `account` |  |
| `applicant.get.refuse.reason` | `action_refuse_reason_apply` | self | `hr_recruitment` |  |
| `applicant.send.mail` | `action_send` | self | `hr_recruitment` |  |
| `auth.passkey.key` | `action_delete_passkey` | self | `auth_passkey` |  |
| `auth.passkey.key` | `action_rename_passkey` | self | `auth_passkey` |  |
| `base.automation` | `action_open_scheduled_action` | self | `base_automation` |  |
| `base.automation` | `action_rotate_webhook_uuid` | self | `base_automation` |  |
| `base.automation` | `action_view_webhook_logs` | self | `base_automation` |  |
| `base.import.module` | `action_module_open` | self | `base_import_module` |  |
| `base.module.install.request` | `action_send_request` | self | `base_install_request` |  |
| `base.module.install.review` | `action_install_module` | self | `base_install_request` |  |
| `base.module.uninstall` | `action_uninstall` | self | `base` |  |
| `base.module.update` | `action_module_open` | self | `base` |  |
| `base.partner.merge.automatic.wizard` | `action_merge` | self | `base` | Merge Contact button. Merge the selected partners, and redirect to the end screen (since there is no other wizard line to process. |
| `base.partner.merge.automatic.wizard` | `action_skip` | self | `base` | Skip this wizard line. Don't compute any thing, and simply redirect to the new step. |
| `base.partner.merge.automatic.wizard` | `action_start_automatic_process` | self | `base` | Start the process 'Merge Automatically'. This will fill the wizard with the same mechanism as 'Merge with Manual Check', but instead of refreshing wizard with the current line, it will automatically process all lines by merging partner grouped according to the checked options. |
| `base.partner.merge.automatic.wizard` | `action_start_manual_process` | self | `base` | Start the process 'Merge with Manual Check'. Fill the wizard according to the group_by and exclude options, and redirect to the first step (treatment of first wizard line). After, for each subset of partner to merge, the wizard will be actualized.      - Compute the selected groups (with duplication |
| `base.partner.merge.automatic.wizard` | `action_update_all_process` | self | `base` |  |
| `bill.to.po.wizard` | `action_add_downpayment` | self | `purchase` |  |
| `bill.to.po.wizard` | `action_add_to_po` | self | `purchase` |  |
| `calendar.alarm_manager` | `do_check_alarm_for_one_date` | self, one_date, event, event_maxdelta, in_the_next_X_seconds, alarm_type, after, missing | `calendar` | Search for some alarms in the interval of time determined by some parameters (after, in_the_next_X_seconds, ...) :param one_date: date of the event to check (not the same that in the event browse if recurrent) :param event: Event browse record :param event_maxdelta: biggest duration from alarms for  |
| `calendar.alarm_manager` | `do_notif_reminder` | self, alert | `calendar` |  |
| `calendar.attendee` | `do_accept` | self | `calendar`, `google_calendar`, `microsoft_calendar` | Marks event invitation as Accepted. |
| `calendar.attendee` | `do_decline` | self | `calendar`, `google_calendar`, `microsoft_calendar` | Marks event invitation as Declined. |
| `calendar.attendee` | `do_tentative` | self | `calendar`, `google_calendar`, `microsoft_calendar` | Makes event invitation as Tentative. |
| `calendar.event` | `action_join_meeting` | self, partner_id | `calendar` | Method used when an existing user wants to join |
| `calendar.event` | `action_join_video_call` | self | `calendar` |  |
| `calendar.event` | `action_mass_archive` | self, recurrence_update_setting | `calendar`, `google_calendar`, `microsoft_calendar` | The aim of this action purpose is to be called from sync calendar module when mass deletion is not possible. |
| `calendar.event` | `action_mass_deletion` | self, recurrence_update_setting | `calendar` |  |
| `calendar.event` | `action_open_calendar_event` | self | `calendar` |  |
| `calendar.event` | `action_open_composer` | self | `calendar` |  |
| `calendar.event` | `action_send_sms` | self | `calendar_sms` |  |
| `calendar.event` | `action_sendmail` | self | `calendar` |  |
| `calendar.event` | `action_unlink_event` | self, attendee_id, recurrence | `calendar` | Delete the event after displaying the delete wizard if necessary.  :param attendee_id: The ID of the attendee for the event :param recurrence: Boolean indicating if the event is recurring :return: Action to delete the event |
| `calendar.popover.delete.wizard` | `action_delete` | self | `calendar` | Delete the event based on the specified deletion type.  :return: Action URL to redirect to the calendar view |
| `calendar.popover.delete.wizard` | `action_send_mail_and_delete` | self | `calendar` | Send email notification and delete the event based on the specified deletion type. |
| `calendar.provider.config` | `action_calendar_prepare_external_provider_sync` | self | `calendar` | Called by the wizard to configure an external calendar provider without requiring users to access the general settings page. Make sure that the provider calendar module is installed or install it. Then, set the API keys into the applicable config parameters. |
| `card.campaign` | `action_preview` | self | `marketing_card` |  |
| `card.campaign` | `action_share` | self | `marketing_card` |  |
| `card.campaign` | `action_view_cards` | self | `marketing_card` |  |
| `card.campaign` | `action_view_cards_clicked` | self | `marketing_card` |  |
| `card.campaign` | `action_view_cards_shared` | self | `marketing_card` |  |
| `card.campaign` | `action_view_mailings` | self | `marketing_card` |  |
| `chatbot.script` | `action_test_script` | self | `website_livechat` |  |
| `chatbot.script` | `action_view_leads` | self | `crm_livechat` |  |
| `chatbot.script` | `action_view_livechat_channels` | self | `im_livechat` |  |
| `choose.delivery.carrier` | `button_confirm` | self | `delivery`, `delivery_mondialrelay` |  |
| `coupon.share` | `action_generate_short_link` | self | `website_sale_loyalty` |  |
| `crm.iap.lead.mining.request` | `action_buy_credits` | self | `crm_iap_mine` |  |
| `crm.iap.lead.mining.request` | `action_draft` | self | `crm_iap_mine` |  |
| `crm.iap.lead.mining.request` | `action_get_lead_action` | self | `crm_iap_mine` |  |
| `crm.iap.lead.mining.request` | `action_get_opportunity_action` | self | `crm_iap_mine` |  |
| `crm.iap.lead.mining.request` | `action_submit` | self | `crm_iap_mine` |  |
| `crm.lead` | `action_assign_partner` | self | `website_crm_partner_assign` | While assigning a partner, geo-localization is performed only for leads having country set (see method 'assign_geo_localize' and 'search_geo_partner'). So for leads that does not have country set, we show the notification, and for the rest, we geo-localize them. |
| `crm.lead` | `action_generate_leads` | self | `crm_iap_mine` |  |
| `crm.lead` | `action_new_quotation` | self | `sale_crm` |  |
| `crm.lead` | `action_open_livechat` | self | `crm_livechat` |  |
| `crm.lead` | `action_redirect_to_livechat_sessions` | self | `website_crm_livechat` |  |
| `crm.lead` | `action_redirect_to_page_views` | self | `website_crm` |  |
| `crm.lead` | `action_reschedule_meeting` | self | `crm` |  |
| `crm.lead` | `action_restore` | self | `crm` | Restoring a lost lead means that it should go back to its normal life cycle. This should reactivate the lead but also force the recompute of its probability, for the stage where the lead is currently at. During toggle_active, when reactivating a lost lead,only the automated probability will be recom |
| `crm.lead` | `action_sale_quotations_new` | self | `sale_crm` |  |
| `crm.lead` | `action_schedule_meeting` | self, smart_calendar | `crm` | Open meeting's calendar view to schedule meeting on current opportunity.  :param bool smart_calendar: to set to False if the view should not try to choose relevant   mode and initial date for calendar view, see ``_get_opportunity_meeting_view_parameters`` :returns: dictionary value for created Meeti |
| `crm.lead` | `action_set_automated_probability` | self | `crm` | Update the automated probability and align probability to that value |
| `crm.lead` | `action_set_lost` | self, **additional_values | `crm` | Lost semantic: probability = 0 AND active = False |
| `crm.lead` | `action_set_won` | self | `crm` | Won semantic: stage.is_won (AND probability = 100 but implied) |
| `crm.lead` | `action_set_won_rainbowman` | self | `crm` |  |
| `crm.lead` | `action_show_potential_duplicates` | self | `crm` | Open kanban view to display duplicate leads or opportunity. :return dict: dictionary value for created kanban view |
| `crm.lead` | `action_view_sale_order` | self | `sale_crm` |  |
| `crm.lead` | `action_view_sale_quotation` | self | `sale_crm` |  |
| `crm.lead.forward.to.partner` | `action_forward` | self | `website_crm_partner_assign` |  |
| `crm.lead.lost` | `action_lost_reason_apply` | self | `crm` | Mark lead as lost and apply the loss reason |
| `crm.lead.pls.update` | `action_update_crm_lead_probabilities` | self | `crm` |  |
| `crm.lead2opportunity.partner` | `action_apply` | self | `crm` |  |
| `crm.lead2opportunity.partner.mass` | `action_mass_convert` | self | `crm` |  |
| `crm.lost.reason` | `action_lost_leads` | self | `crm` |  |
| `crm.merge.opportunity` | `action_merge` | self | `crm` |  |
| `crm.quotation.partner` | `action_apply` | self | `sale_crm` | Convert lead to opportunity or merge lead and opportunity and open the freshly created opportunity view. |
| `crm.reveal.rule` | `action_get_lead_tree_view` | self | `website_crm_iap_reveal` |  |
| `crm.reveal.rule` | `action_get_opportunity_tree_view` | self | `website_crm_iap_reveal` |  |
| `crm.team` | `action_assign_leads` | self | `crm` | Manual (direct) leads assignment. This method both    * assigns leads to teams given by self;   * assigns leads to salespersons belonging to self;  See sub methods for more details about assign process.  :returns: action, a client notification giving some insights on assign   process; |
| `crm.team` | `action_open_leads` | self | `crm` |  |
| `crm.team` | `action_open_unassigned_leads` | self | `crm` |  |
| `crm.team` | `action_opportunity_forecast` | self | `crm` |  |
| `crm.team` | `action_primary_channel_button` | self | `crm`, `sale`, `sale_crm`, `sales_team` | Skeleton function to be overloaded It will return the adequate action depending on the Sales Team's options. |
| `crm.team` | `action_your_pipeline` | self | `crm` |  |
| `data_recycle.model` | `action_recycle_records` | self | `data_recycle` |  |
| `data_recycle.record` | `action_discard` | self | `data_recycle` |  |
| `data_recycle.record` | `action_validate` | self | `data_recycle` |  |
| `digest.digest` | `action_activate` | self | `digest` |  |
| `digest.digest` | `action_deactivate` | self | `digest` |  |
| `digest.digest` | `action_send` | self | `digest` | Send digests emails to all the registered users. |
| `digest.digest` | `action_send_manual` | self | `digest` | Manually send digests emails to all registered users. In that case do not update periodicity as this is not an automation rule that could be considered as unwanted spam. |
| `digest.digest` | `action_set_periodicity` | self, periodicity | `digest` |  |
| `digest.digest` | `action_subscribe` | self | `digest` |  |
| `digest.digest` | `action_unsubscribe` | self | `digest` |  |
| `discuss.channel` | `action_unfollow` | self | `mail` |  |
| `discuss.channel.rtc.session` | `action_disconnect` | self | `mail` |  |
| `event.booth` | `action_confirm` | self, additional_values | `event_booth` |  |
| `event.booth` | `action_set_paid` | self | `event_booth_sale` |  |
| `event.booth` | `action_view_sale_order` | self | `event_booth_sale` |  |
| `event.booth` | `action_view_sponsor` | self | `website_event_booth_exhibitor` |  |
| `event.booth.registration` | `action_confirm` | self | `event_booth_sale` |  |
| `event.event` | `action_generate_leads` | self, event_lead_rules | `event_crm` | Re-generate leads based on event.lead.rules. The method is ran synchronously if there is a low amount of registrations, otherwise it goes through a CRON job that runs in batches. |
| `event.event` | `action_invite_contacts` | self | `mass_mailing_event`, `mass_mailing_event_sms` |  |
| `event.event` | `action_mass_mailing_attendees` | self | `mass_mailing_event`, `mass_mailing_event_sms` |  |
| `event.event` | `action_mass_mailing_track_speakers` | self | `mass_mailing_event_track`, `mass_mailing_event_track_sms` |  |
| `event.event` | `action_open_slot_calendar` | self | `event` |  |
| `event.event` | `action_set_done` | self | `event` | Action which will move the events into the first next (by sequence) stage defined as "Ended" (if they are not already in an ended stage) |
| `event.event` | `action_view_linked_orders` | self | `event_sale` | Redirects to only the confirmed orders linked to the current events |
| `event.lead.rule` | `action_execute_rule` | self | `event_crm` |  |
| `event.question` | `action_event_view` | self | `event` |  |
| `event.question` | `action_view_question_answers` | self | `event` | Allow analyzing the attendees answers to event questions in a convenient way:  - A graph view showing counts of each suggestion for simple_choice questions   (Along with secondary pivot and list views) - A list view showing textual answers values for text_box questions. |
| `event.question.answer` | `action_add_rule_button` | self | `event_crm` |  |
| `event.registration` | `action_cancel` | self | `event` |  |
| `event.registration` | `action_confirm` | self | `event` |  |
| `event.registration` | `action_send_badge_email` | self | `event` | Open a window to compose an email, with the template - 'event_badge' message loaded by default |
| `event.registration` | `action_set_done` | self | `event` | Close Registration |
| `event.registration` | `action_set_draft` | self | `event` |  |
| `event.registration` | `action_view_pos_order` | self | `pos_event` |  |
| `event.registration` | `action_view_sale_order` | self | `event_sale` |  |
| `event.track` | `action_add_quiz` | self | `website_event_track_quiz` |  |
| `event.track` | `action_view_quiz` | self | `website_event_track_quiz` |  |
| `fetchmail.server` | `button_confirm_login` | self | `mail` |  |
| `fleet.vehicle` | `action_accept_driver_change` | self | `fleet` |  |
| `fleet.vehicle` | `action_open_employee` | self | `hr_fleet` |  |
| `fleet.vehicle` | `action_open_odometer_report` | self | `fleet` |  |
| `fleet.vehicle` | `action_send_email` | self | `fleet` |  |
| `fleet.vehicle` | `action_view_bills` | self | `account_fleet` |  |
| `fleet.vehicle.assignation.log` | `action_get_attachment_view` | self | `hr_fleet` |  |
| `fleet.vehicle.log.contract` | `action_close` | self | `fleet` |  |
| `fleet.vehicle.log.contract` | `action_draft` | self | `fleet` |  |
| `fleet.vehicle.log.contract` | `action_expire` | self | `fleet` |  |
| `fleet.vehicle.log.contract` | `action_open` | self | `fleet` |  |
| `fleet.vehicle.log.contract` | `action_open_employee` | self | `hr_fleet` |  |
| `fleet.vehicle.log.services` | `action_open_account_move` | self | `account_fleet` |  |
| `fleet.vehicle.model` | `action_model_vehicle` | self | `fleet` |  |
| `fleet.vehicle.model.brand` | `action_brand_model` | self | `fleet` |  |
| `fleet.vehicle.model.brand` | `action_open_brand_form` | self | `fleet` |  |
| `fleet.vehicle.send.mail` | `action_save_as_template` | self | `fleet` |  |
| `fleet.vehicle.send.mail` | `action_send` | self | `fleet` |  |
| `gamification.badge.user` | `action_open_badge` | self | `hr_gamification` |  |
| `gamification.badge.user.wizard` | `action_grant_badge` | self | `gamification`, `hr_gamification` | Wizard action for sending a badge to a chosen user |
| `gamification.challenge` | `action_check` | self | `gamification` | Check a challenge  Create goals that haven't been created yet (eg: if added users) Recompute the current value for each goal related |
| `gamification.challenge` | `action_report_progress` | self | `gamification` | Manual report of a goal, does not influence automatic report frequency |
| `gamification.challenge` | `action_start` | self | `gamification` | Start a challenge |
| `gamification.challenge` | `action_view_users` | self | `gamification` | Redirect to the participants (users) list. |
| `gamification.goal` | `action_cancel` | self | `gamification` | Reset the completion after setting a goal as reached or failed.  This is only the current state, if the date and/or target criteria match the conditions for a change of state, this will be applied at the next goal update. |
| `gamification.goal` | `action_fail` | self | `gamification` | Set the state of the goal to failed.  A failed goal will be ignored in future checks. |
| `gamification.goal` | `action_reach` | self | `gamification` | Mark a goal as reached.  If the target goal condition is not met, the state will be reset to In Progress at the next goal update until the end date. |
| `gamification.goal` | `action_start` | self | `gamification` | Mark a goal as started.  This should only be used when creating goals manually (in draft state) |
| `gamification.goal.wizard` | `action_update_current` | self | `gamification` | Wizard action for updating the current value |
| `hr.applicant` | `action_add_to_job` | self | `hr_recruitment_skills` |  |
| `hr.applicant` | `action_create_meeting` | self | `hr_recruitment` | This opens Meeting's calendar view to schedule meeting on current applicant @return: Dictionary value for created Meeting view |
| `hr.applicant` | `action_job_add_applicants` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_open_applications` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_open_attachments` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_open_employee` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_print_survey` | self | `hr_recruitment_survey` | If response is available then print this response otherwise print survey form (print template of the survey) |
| `hr.applicant` | `action_send_email` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_send_sms` | self | `hr_recruitment_sms` |  |
| `hr.applicant` | `action_send_survey` | self | `hr_recruitment_survey` |  |
| `hr.applicant` | `action_talent_pool_add_applicants` | self | `hr_recruitment` |  |
| `hr.applicant` | `action_talent_pool_stat_button` | self | `hr_recruitment` |  |
| `hr.attendance` | `action_approve_overtime` | self | `hr_attendance` |  |
| `hr.attendance` | `action_in_attendance_maps` | self | `hr_attendance` |  |
| `hr.attendance` | `action_out_attendance_maps` | self | `hr_attendance` |  |
| `hr.attendance` | `action_refuse_overtime` | self | `hr_attendance` |  |
| `hr.attendance` | `action_try_kiosk` | self | `hr_attendance` |  |
| `hr.attendance.overtime.line` | `action_approve` | self | `hr_attendance` |  |
| `hr.attendance.overtime.line` | `action_refuse` | self | `hr_attendance` |  |
| `hr.attendance.overtime.ruleset` | `action_regenerate_overtimes` | self | `hr_attendance` |  |
| `hr.bank.account.allocation.wizard` | `action_save` | self | `hr` |  |
| `hr.department` | `action_employee_from_department` | self | `hr` |  |
| `hr.department` | `action_open_allocation_department` | self | `hr_holidays` |  |
| `hr.department` | `action_open_leave_department` | self | `hr_holidays` |  |
| `hr.department` | `action_open_view_child_departments` | self | `hr` |  |
| `hr.department` | `action_plan_from_department` | self | `hr` |  |
| `hr.departure.wizard` | `action_register_departure` | self | `hr`, `hr_fleet`, `hr_holidays`, `hr_maintenance` |  |
| `hr.employee` | `action_create_user` | self | `hr` |  |
| `hr.employee` | `action_create_users` | self | `hr` |  |
| `hr.employee` | `action_create_users_confirmation` | self | `hr` |  |
| `hr.employee` | `action_open_allocation_wizard` | self | `hr` |  |
| `hr.employee` | `action_open_courses` | self | `hr_skills_slides` |  |
| `hr.employee` | `action_open_employee_cars` | self | `hr_fleet` |  |
| `hr.employee` | `action_open_last_month_attendances` | self | `hr_attendance` |  |
| `hr.employee` | `action_open_leave_request` | self | `hr_presence` |  |
| `hr.employee` | `action_open_versions` | self | `hr` |  |
| `hr.employee` | `action_open_work_entries` | self, initial_date | `hr_work_entry` |  |
| `hr.employee` | `action_related_contacts` | self | `hr` |  |
| `hr.employee` | `action_send_log` | self | `hr_presence` |  |
| `hr.employee` | `action_send_sms` | self | `hr_presence` |  |
| `hr.employee` | `action_set_absent` | self | `hr_presence` |  |
| `hr.employee` | `action_set_present` | self | `hr_presence` |  |
| `hr.employee` | `action_time_off_dashboard` | self | `hr_holidays` |  |
| `hr.employee` | `action_timesheet_from_employee` | self | `hr_timesheet` |  |
| `hr.employee` | `action_toggle_primary_bank_account_trust` | self | `hr` |  |
| `hr.employee` | `action_unlink_wizard` | self | `hr_timesheet` |  |
| `hr.employee.cv.wizard` | `action_validate` | self | `hr_skills` |  |
| `hr.employee.delete.wizard` | `action_confirm_delete` | self | `hr_timesheet` |  |
| `hr.employee.delete.wizard` | `action_open_timesheets` | self | `hr_timesheet` |  |
| `hr.employee.public` | `action_open_courses` | self | `hr_skills_slides` |  |
| `hr.employee.public` | `action_open_last_month_attendances` | self | `hr_attendance` |  |
| `hr.employee.public` | `action_open_time_off_calendar` | self | `hr_holidays` | Open the time off calendar filtered on this employee. |
| `hr.employee.public` | `action_time_off_dashboard` | self | `hr_holidays` |  |
| `hr.employee.public` | `action_timesheet_from_employee` | self | `hr_timesheet` |  |
| `hr.employee.skill` | `action_save` | self | `hr_skills` |  |
| `hr.expense` | `action_approve` | self | `hr_expense` | Approve an expense, pops a wizard if a duplicated expense is found to confirm they are all valid expenses |
| `hr.expense` | `action_approve_duplicates` | self | `hr_expense` |  |
| `hr.expense` | `action_open_account_move` | self | `hr_expense` |  |
| `hr.expense` | `action_open_sale_order` | self | `sale_expense` |  |
| `hr.expense` | `action_open_split_expense` | self | `hr_expense` |  |
| `hr.expense` | `action_pay` | self | `hr_expense` | Register payment shortcut on the expense form view |
| `hr.expense` | `action_post` | self | `hr_expense`, `project_sale_expense`, `sale_expense` | Post the expense, following one of those two options:     - Company-paid expenses: Create and post a payment, with an accounting entry     - Employee-paid expenses: Through a wizard, create and post a receipt |
| `hr.expense` | `action_refuse` | self | `hr_expense` | Refuse an expense with a reason |
| `hr.expense` | `action_reset` | self | `hr_expense` | Reset an expense to draft state, reversing the accounting entries if needed |
| `hr.expense` | `action_show_same_receipt_expense_ids` | self | `hr_expense` |  |
| `hr.expense` | `action_split_wizard` | self | `hr_expense` |  |
| `hr.expense` | `action_submit` | self | `hr_expense` | Submit a draft expense to an approve, may skip to the approval step if no approver on the employee nor the expense |
| `hr.expense.approve.duplicate` | `action_approve` | self | `hr_expense` |  |
| `hr.expense.approve.duplicate` | `action_refuse` | self | `hr_expense` |  |
| `hr.expense.post.wizard` | `action_post_entry` | self | `hr_expense` |  |
| `hr.expense.refuse.wizard` | `action_refuse` | self | `hr_expense` |  |
| `hr.expense.split.wizard` | `action_split_expense` | self | `hr_expense` |  |
| `hr.holidays.cancel.leave` | `action_cancel_leave` | self | `hr_holidays` |  |
| `hr.job` | `action_new_survey` | self | `hr_recruitment_survey` |  |
| `hr.job` | `action_open_activities` | self | `hr_recruitment` |  |
| `hr.job` | `action_open_attachments` | self | `hr_recruitment` |  |
| `hr.job` | `action_open_employees` | self | `hr_recruitment` |  |
| `hr.job` | `action_search_matching_applicants` | self | `hr_recruitment_skills` |  |
| `hr.job` | `action_test_survey` | self | `hr_recruitment_survey` |  |
| `hr.leave` | `action_approve` | self, check_state | `hr_holidays`, `hr_holidays_attendance`, `l10n_in_hr_holidays` |  |
| `hr.leave` | `action_back_to_approval` | self | `hr_holidays` |  |
| `hr.leave` | `action_cancel` | self | `hr_holidays` |  |
| `hr.leave` | `action_documents` | self | `hr_holidays` |  |
| `hr.leave` | `action_refuse` | self | `hr_holidays`, `hr_work_entry_holidays`, `l10n_in_hr_holidays`, `project_timesheet_holidays` | Override to archive linked work entries and recreate attendance work entries where the refused leave was. |
| `hr.leave` | `action_reset_confirm` | self | `hr_holidays_attendance` |  |
| `hr.leave.accrual.level` | `action_save_new` | self | `hr_holidays` |  |
| `hr.leave.accrual.plan` | `action_create_accrual_plan_level` | self | `hr_holidays` |  |
| `hr.leave.accrual.plan` | `action_open_accrual_plan_employees` | self | `hr_holidays` |  |
| `hr.leave.accrual.plan` | `action_open_accrual_plan_level` | self, level_id | `hr_holidays` |  |
| `hr.leave.allocation` | `action_approve` | self | `hr_holidays` |  |
| `hr.leave.allocation` | `action_refuse` | self | `hr_holidays`, `hr_holidays_attendance` |  |
| `hr.leave.allocation.generate.multi.wizard` | `action_generate_allocations` | self | `hr_holidays` |  |
| `hr.leave.employee.type.report` | `action_time_off_analysis` | self | `hr_holidays` |  |
| `hr.leave.generate.multi.wizard` | `action_generate_time_off` | self | `hr_holidays` |  |
| `hr.leave.report` | `action_open_record` | self | `hr_holidays` |  |
| `hr.leave.report.calendar` | `action_approve` | self | `hr_holidays` |  |
| `hr.leave.report.calendar` | `action_refuse` | self | `hr_holidays` |  |
| `hr.leave.type` | `action_see_accrual_plans` | self | `hr_holidays` |  |
| `hr.leave.type` | `action_see_days_allocated` | self | `hr_holidays` |  |
| `hr.leave.type` | `action_see_group_leaves` | self | `hr_holidays` |  |
| `hr.talent.pool` | `action_talent_pool_add_talents` | self | `hr_recruitment` |  |
| `hr.version` | `action_open_version` | self | `hr` |  |
| `hr.version` | `action_open_version_form_view` | self | `hr` |  |
| `hr.version.wizard` | `action_load_template` | self | `hr` |  |
| `hr.work.entry` | `action_approve_leave` | self | `hr_work_entry_holidays` |  |
| `hr.work.entry` | `action_refuse_leave` | self | `hr_work_entry_holidays` |  |
| `hr.work.entry` | `action_split` | self, vals | `hr_work_entry` |  |
| `hr.work.entry` | `action_validate` | self | `hr_work_entry` | Try to validate work entries. If some errors are found, set `state` to conflict for conflicting work entries and validation fails. :return: True if validation succeeded |
| `iap.account` | `action_buy_credits` | self | `iap` |  |
| `iap.account` | `action_open_registration_wizard` | self | `sms` |  |
| `iap.account` | `action_open_sender_name_wizard` | self | `sms` |  |
| `im_livechat.channel` | `action_join` | self | `im_livechat` |  |
| `im_livechat.channel` | `action_quit` | self | `im_livechat` |  |
| `im_livechat.channel` | `action_view_chatbot_scripts` | self | `im_livechat` |  |
| `im_livechat.channel` | `action_view_rating` | self | `im_livechat` | Action to display the rating relative to the channel, so all rating of the sessions of the current channel :returns : the ir.action 'action_view_rating' with the correct context |
| `im_livechat.channel.member.history` | `action_open_discuss_channel_view` | self, domain | `im_livechat` |  |
| `im_livechat.report.channel` | `action_open_discuss_channel_view` | self, domain | `im_livechat` |  |
| `ir.actions.server` | `action_open_automation` | self | `base_automation` |  |
| `ir.actions.server` | `action_open_parent_action` | self | `base` |  |
| `ir.actions.server` | `action_open_scheduled_action` | self | `base` |  |
| `ir.actions.todo` | `action_launch` | self | `base` | Launch Action of Wizard |
| `ir.actions.todo` | `action_open` | self | `base` | Sets configuration wizard in TODO state |
| `ir.attachment` | `action_get` | self | `base` |  |
| `ir.attachment` | `action_preview_attachment` | self | `hr_fleet` |  |
| `ir.cron` | `action_open_automation` | self | `base_automation` |  |
| `ir.cron` | `action_open_parent_action` | self | `base` |  |
| `ir.cron` | `action_open_scheduled_action` | self | `base` |  |
| `ir.mail_server` | `action_retrieve_max_email_size` | self | `base` |  |
| `ir.module.module` | `action_open_install_request` | self | `base_install_request` |  |
| `ir.module.module` | `action_view_delivery_methods` | self | `delivery` |  |
| `ir.module.module` | `button_choose_theme` | self | `website` | Remove any existing theme on the current website and install the theme ``self`` instead.  The actual loading of the theme on the current website will be done automatically on ``write`` thanks to the upgrade and/or install.  When installating a new theme, upgrade the upstream chain first to make sure |
| `ir.module.module` | `button_immediate_install` | self | `base` | Installs the selected module(s) immediately and fully, returns the next res.config action to execute  :returns: next res.config item to execute :rtype: dict[str, object] |
| `ir.module.module` | `button_immediate_install_app` | self | `base_import_module` |  |
| `ir.module.module` | `button_immediate_uninstall` | self | `base` | Uninstall the selected module(s) immediately and fully, returns the next res.config action to execute |
| `ir.module.module` | `button_immediate_upgrade` | self | `base` | Upgrade the selected module(s) immediately and fully, return the next res.config action to execute |
| `ir.module.module` | `button_install` | self | `base` |  |
| `ir.module.module` | `button_refresh_theme` | self | `website` | Refresh the current theme of the current website.  To refresh it, we only need to upgrade the modules. Indeed the (re)loading of the theme will be done automatically on ``write``. |
| `ir.module.module` | `button_remove_theme` | self | `website` | Remove the current theme of the current website. |
| `ir.module.module` | `button_reset_state` | self | `base` |  |
| `ir.module.module` | `button_uninstall` | self | `base` |  |
| `ir.module.module` | `button_uninstall_wizard` | self | `base` | Launch the wizard to uninstall the given module. |
| `ir.module.module` | `button_upgrade` | self | `base`, `base_import_module` |  |
| `ir.profile` | `action_view_speedscope` | self | `base` |  |
| `job.add.applicants` | `action_add_applicants_to_job` | self | `hr_recruitment` |  |
| `l10n.fr.pdp.reports.flow` | `action_build_payload_manual` | self | `l10n_fr_pdp` | Manual trigger for payload building. |
| `l10n.fr.pdp.reports.flow` | `action_open_send_wizard` | self | `l10n_fr_pdp` | Open send wizard if errors exist, otherwise send directly. |
| `l10n.fr.pdp.reports.flow` | `action_send` | self, check_totp | `l10n_fr_pdp` | Send flow payload to transport gateway. The parameter check totp is no longer useful and will be remove in master |
| `l10n.fr.pdp.reports.flow` | `action_send_from_ui` | self | `l10n_fr_pdp` | Send flow from UI with error checking. |
| `l10n.fr.pdp.reports.flow` | `action_view_error_moves` | self | `l10n_fr_pdp` | Open list view of invalid invoices. |
| `l10n.fr.pdp.reports.flow` | `action_view_initial` | self | `l10n_fr_pdp` |  |
| `l10n.fr.pdp.reports.flow` | `action_view_moves` | self | `l10n_fr_pdp` | Open list view of related invoices. |
| `l10n.fr.pdp.reports.send.wizard` | `action_send_anyway` | self | `l10n_fr_pdp` | Send flow excluding invalid invoices. |
| `l10n.fr.pdp.reports.send.wizard` | `action_view_errors` | self | `l10n_fr_pdp` | Open list of invalid invoices. |
| `l10n.in.ewaybill` | `action_cancel_ewaybill` | self | `l10n_in_ewaybill` |  |
| `l10n.in.ewaybill` | `action_export_content_json` | self | `l10n_in_ewaybill` |  |
| `l10n.in.ewaybill` | `action_generate_ewaybill` | self | `l10n_in_ewaybill`, `l10n_in_ewaybill_irn` |  |
| `l10n.in.ewaybill` | `action_print` | self | `l10n_in_ewaybill`, `l10n_in_ewaybill_stock` |  |
| `l10n.in.ewaybill` | `action_reset_to_pending` | self | `l10n_in_ewaybill`, `l10n_in_ewaybill_irn`, `l10n_in_ewaybill_stock` |  |
| `l10n.in.ewaybill` | `action_set_to_challan` | self | `l10n_in_ewaybill_stock` |  |
| `l10n_ch.qr_invoice.wizard` | `action_view_faulty_invoices` | self | `l10n_ch` | Open a list view of all the invoices that could not be printed in the QR format. |
| `l10n_eg_edi.thumb.drive` | `action_set_certificate_from_usb` | self | `l10n_eg_edi_eta` |  |
| `l10n_eg_edi.thumb.drive` | `action_sign_invoices` | self, invoice_ids | `l10n_eg_edi_eta` |  |
| `l10n_gr_edi.document` | `action_download` | self | `l10n_gr_edi` | Download the XML file linked to the document. |
| `l10n_hr_edi.mojeracun_reject_wizard` | `button_reject_invoice` | self | `l10n_hr_edi` |  |
| `l10n_hu_edi.cancellation` | `button_request_cancel` | self | `l10n_hu_edi` |  |
| `l10n_hu_edi.tax_audit_export` | `action_export` | self | `l10n_hu_edi` |  |
| `l10n_hu_edi_receive.bills.wizard` | `action_receive_bills` | self | `l10n_hu_edi_receive` |  |
| `l10n_id_efaktur_coretax.document` | `action_download` | self | `l10n_id_efaktur_coretax` | Download E-Faktur of related attachment |
| `l10n_id_efaktur_coretax.document` | `action_regenerate` | self | `l10n_id_efaktur_coretax` |  |
| `l10n_in.withhold.wizard` | `action_create_and_post_withhold` | self | `l10n_in` |  |
| `l10n_it_edi_doi.declaration_of_intent` | `action_open_invoice_ids` | self | `l10n_it_edi_doi` |  |
| `l10n_it_edi_doi.declaration_of_intent` | `action_open_sale_order_ids` | self | `l10n_it_edi_doi` |  |
| `l10n_it_edi_doi.declaration_of_intent` | `action_reactivate` | self | `l10n_it_edi_doi` | Resets a not 'active' Declaration of Intent back to 'active'. |
| `l10n_it_edi_doi.declaration_of_intent` | `action_reset_to_draft` | self | `l10n_it_edi_doi` | Resets an 'active' Declaration of Intent back to 'draft'. |
| `l10n_it_edi_doi.declaration_of_intent` | `action_revoke` | self | `l10n_it_edi_doi` | Called by the 'revoke' button of the form view. |
| `l10n_it_edi_doi.declaration_of_intent` | `action_terminate` | self | `l10n_it_edi_doi` | Called by the 'terminated' button of the form view. |
| `l10n_it_edi_doi.declaration_of_intent` | `action_validate` | self | `l10n_it_edi_doi` | Move a 'draft' Declaration of Intent to 'active'. |
| `l10n_latam.check` | `action_show_journal_entry` | self | `l10n_latam_check` |  |
| `l10n_latam.check` | `action_show_reconciled_move` | self | `l10n_latam_check` |  |
| `l10n_latam.check` | `action_void` | self | `l10n_latam_check` |  |
| `l10n_latam.check` | `button_open_check_operations` | self | `l10n_latam_check` | Redirect the user to the invoice(s) paid by this payment. :return:    An action on account.move. |
| `l10n_latam.check` | `button_open_payment` | self | `l10n_latam_check` |  |
| `l10n_latam.payment.mass.transfer` | `action_create_payments` | self | `l10n_latam_check` |  |
| `l10n_ph_2307.wizard` | `action_generate` | self | `l10n_ph` | Generate a xls format file for importing to https://bir-excel-uploader.com/excel-file-to-bir-dat-format/#bir-form-2307-settings. This website will then generate a BIR 2307 format excel file for uploading to the PH government. |
| `l10n_ro_edi.document` | `action_l10n_ro_edi_download_attachment` | self | `l10n_ro_edi` | Download the sent attachment in case if no status have been received from ANAF. Otherwise, download the received successful signature XML file from E-Factura. |
| `l10n_ro_edi.document` | `action_l10n_ro_edi_fetch_status` | self | `l10n_ro_edi` | Fetch the latest response from E-Factura about the XML sent |
| `l10n_tw_edi.invoice.cancel` | `button_request_cancel` | self | `l10n_tw_edi_ecpay` |  |
| `l10n_tw_edi.invoice.print` | `button_print` | self | `l10n_tw_edi_ecpay` |  |
| `l10n_vn_edi_viettel.cancellation` | `button_request_cancel` | self | `l10n_vn_edi_viettel` |  |
| `link.tracker` | `action_view_statistics` | self | `link_tracker` |  |
| `link.tracker` | `action_visit_page` | self | `link_tracker` |  |
| `link.tracker` | `action_visit_page_statistics` | self | `website_links` |  |
| `loyalty.card` | `action_coupon_send` | self | `loyalty` | Open a window to compose an email, with the default template returned by `_get_default_template` message loaded by default |
| `loyalty.card` | `action_coupon_share` | self | `website_sale_loyalty` |  |
| `loyalty.card` | `action_loyalty_update_balance` | self | `loyalty` |  |
| `loyalty.card.update.balance` | `action_update_card_point` | self | `loyalty` |  |
| `loyalty.program` | `action_open_loyalty_cards` | self | `loyalty` |  |
| `loyalty.program` | `action_program_share` | self | `website_sale_loyalty` |  |
| `lunch.order` | `action_cancel` | self | `lunch` |  |
| `lunch.order` | `action_confirm` | self | `lunch` |  |
| `lunch.order` | `action_notify` | self | `lunch` |  |
| `lunch.order` | `action_order` | self | `lunch` |  |
| `lunch.order` | `action_reorder` | self | `lunch` |  |
| `lunch.order` | `action_reset` | self | `lunch` |  |
| `lunch.order` | `action_send` | self | `lunch` |  |
| `lunch.supplier` | `action_confirm_orders` | self | `lunch` |  |
| `lunch.supplier` | `action_send_orders` | self | `lunch` |  |
| `mail.activity` | `action_cancel` | self | `mail` |  |
| `mail.activity` | `action_close_dialog` | self | `mail` |  |
| `mail.activity` | `action_create_calendar_event` | self | `calendar`, `crm` | Small override of the action that creates a calendar.  If the activity is linked to a crm.lead through the "opportunity_id" field, we include in the action context the default values used when scheduling a meeting from the crm.lead form view. e.g: It will set the partner_id of the crm.lead as defaul |
| `mail.activity` | `action_done` | self | `mail` | Wrapper without feedback because web button add context as parameter, therefore setting context to feedback |
| `mail.activity` | `action_done_redirect_to_other` | self | `mail` | Mark activity as done and return action mail.mail_activity_without_access_action.  Goal: Unless "keep done" activity is enabled, when marking an activity as done, the activity is deleted and can no more be displayed. To overcome this, we return an action that will launch the list view displaying the |
| `mail.activity` | `action_done_schedule_next` | self | `mail` | Wrapper without feedback because web button add context as parameter, therefore setting context to feedback |
| `mail.activity` | `action_feedback` | self, feedback, attachment_ids | `mail` |  |
| `mail.activity` | `action_feedback_schedule_next` | self, feedback, attachment_ids | `mail` |  |
| `mail.activity` | `action_notify` | self | `mail` |  |
| `mail.activity` | `action_open_document` | self | `mail` | Opens the related record based on the model and ID, or activity if user has no access to the related record. |
| `mail.activity` | `action_reschedule_nextweek` | self | `mail` |  |
| `mail.activity` | `action_reschedule_today` | self | `mail` |  |
| `mail.activity` | `action_reschedule_tomorrow` | self | `mail` |  |
| `mail.activity.mixin` | `action_reschedule_my_next_nextweek` | self | `mail` |  |
| `mail.activity.mixin` | `action_reschedule_my_next_today` | self | `mail` |  |
| `mail.activity.mixin` | `action_reschedule_my_next_tomorrow` | self | `mail` |  |
| `mail.activity.schedule` | `action_create_calendar_event` | self | `calendar` |  |
| `mail.activity.schedule` | `action_schedule_activities` | self | `mail` |  |
| `mail.activity.schedule` | `action_schedule_activities_done` | self | `mail` |  |
| `mail.activity.schedule` | `action_schedule_plan` | self | `mail` |  |
| `mail.blacklist` | `action_add` | self | `mail` |  |
| `mail.blacklist.remove` | `action_unblacklist_apply` | self | `mail` |  |
| `mail.compose.message` | `action_schedule_message` | self | `mail` |  |
| `mail.compose.message` | `action_send_mail` | self | `mail` | Used for action button that do not accept arguments. |
| `mail.group` | `action_close` | self | `mail_group` |  |
| `mail.group` | `action_go_to_website` | self | `website_mail_group` |  |
| `mail.group` | `action_join` | self | `mail_group` |  |
| `mail.group` | `action_leave` | self | `mail_group` |  |
| `mail.group` | `action_open` | self | `mail_group` |  |
| `mail.group` | `action_send_guidelines` | self, members | `mail_group` | Send guidelines to given members. |
| `mail.group.message` | `action_moderate_accept` | self | `mail_group` | Accept the incoming email.  Will send the incoming email to all members of the group. |
| `mail.group.message` | `action_moderate_allow` | self | `mail_group` |  |
| `mail.group.message` | `action_moderate_ban` | self | `mail_group` |  |
| `mail.group.message` | `action_moderate_ban_with_comment` | self, ban_subject, ban_comment | `mail_group` |  |
| `mail.group.message` | `action_moderate_reject` | self | `mail_group` |  |
| `mail.group.message` | `action_moderate_reject_with_comment` | self, reject_subject, reject_comment | `mail_group` |  |
| `mail.group.message.reject` | `action_send_mail` | self | `mail_group` |  |
| `mail.mail` | `action_open_document` | self | `mail` | Opens the related record based on the model and ID |
| `mail.mail` | `action_retry` | self | `mail` |  |
| `mail.mail` | `action_send_and_close` | self | `mail` | An action sending the selected mail and redirecting to mail.mail list view. |
| `mail.message` | `action_open_document` | self | `mail` | Opens the related record based on the model and ID |
| `mail.template` | `action_open_mail_preview` | self | `mail` |  |
| `mailing.contact` | `action_add_to_mailing_list` | self | `mass_mailing` |  |
| `mailing.contact` | `action_import` | self | `mass_mailing` |  |
| `mailing.contact.import` | `action_import` | self | `mass_mailing` | Import each lines of "contact_list" as a new contact. |
| `mailing.contact.import` | `action_open_base_import` | self | `mass_mailing` | Open the base import wizard to import mailing list contacts with a xlsx file. |
| `mailing.contact.to.list` | `action_add_contacts` | self | `mass_mailing` | Simply add contacts to the mailing list and close wizard. |
| `mailing.contact.to.list` | `action_add_contacts_and_send_mailing` | self | `mass_mailing` | Add contacts to the mailing list and redirect to a new mailing on this list. |
| `mailing.list` | `action_merge` | self, src_lists, archive | `mass_mailing` | Insert all the contact from the mailing lists 'src_lists' to the mailing list in 'self'. Possibility to archive the mailing lists 'src_lists' after the merge except the destination mailing list 'self'. |
| `mailing.list` | `action_open_import` | self | `mass_mailing` | Open the mailing list contact import wizard. |
| `mailing.list` | `action_send_mailing` | self | `mass_mailing` | Open the mailing form view, with the current lists set as recipients. |
| `mailing.list` | `action_send_mailing_sms` | self | `mass_mailing_sms` |  |
| `mailing.list` | `action_view_contacts` | self | `mass_mailing` |  |
| `mailing.list` | `action_view_contacts_blacklisted` | self | `mass_mailing` |  |
| `mailing.list` | `action_view_contacts_bouncing` | self | `mass_mailing` |  |
| `mailing.list` | `action_view_contacts_email` | self | `mass_mailing` |  |
| `mailing.list` | `action_view_contacts_opt_out` | self | `mass_mailing` |  |
| `mailing.list` | `action_view_contacts_sms` | self | `mass_mailing_sms` |  |
| `mailing.list` | `action_view_mailings` | self | `mass_mailing`, `mass_mailing_sms` |  |
| `mailing.list.merge` | `action_mailing_lists_merge` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_buy_sms_credits` | self | `mass_mailing_sms` |  |
| `mailing.mailing` | `action_cancel` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_compare_versions` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_duplicate` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_fetch_favorites` | self, extra_domain | `mass_mailing` | Return all mailings set as favorite and skip mailings with empty body.  Return archived mailing templates as well, so the user can archive the templates while keeping using it, without cluttering the Kanban view if they're a lot of templates. |
| `mailing.mailing` | `action_launch` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_put_in_queue` | self | `marketing_card`, `mass_mailing` | Detect mismatches before scheduling. |
| `mailing.mailing` | `action_redirect_to_invoiced` | self | `mass_mailing_sale` |  |
| `mailing.mailing` | `action_redirect_to_leads_and_opportunities` | self | `mass_mailing_crm` |  |
| `mailing.mailing` | `action_redirect_to_quotations` | self | `mass_mailing_sale` |  |
| `mailing.mailing` | `action_reload` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_remove_favorite` | self | `mass_mailing` | Remove the current mailing from the favorites list. |
| `mailing.mailing` | `action_retry_failed` | self | `mass_mailing`, `mass_mailing_sms` | Remove all failed emails and their traces, and try sending them again. |
| `mailing.mailing` | `action_retry_failed_sms` | self | `mass_mailing_sms` |  |
| `mailing.mailing` | `action_schedule` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_select_as_winner` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_send_mail` | self, res_ids | `marketing_card`, `mass_mailing` |  |
| `mailing.mailing` | `action_send_sms` | self, res_ids | `mass_mailing_sms` |  |
| `mailing.mailing` | `action_send_winner_mailing` | self | `mass_mailing` | Send the winner mailing based on the winner selection field. This action is used in 2 cases:  - When the user clicks on a button to send the winner mailing. There is only one mailing in self - When the cron is executed to send winner mailing based on the A/B testing schedule datetime. In this   case |
| `mailing.mailing` | `action_set_favorite` | self | `mass_mailing` | Add the current mailing in the favorites list. |
| `mailing.mailing` | `action_test` | self | `mass_mailing`, `mass_mailing_sms` |  |
| `mailing.mailing` | `action_update_cards` | self | `marketing_card` | Update the cards in batches, commiting after each batch. |
| `mailing.mailing` | `action_view_bounced` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_clicked` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_delivered` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_link_trackers` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_mailing_contacts` | self | `mass_mailing` | Show the mailing contacts who are in a mailing list selected for this mailing. |
| `mailing.mailing` | `action_view_opened` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_replied` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_traces_canceled` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_traces_failed` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_traces_process` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_traces_scheduled` | self | `mass_mailing` |  |
| `mailing.mailing` | `action_view_traces_sent` | self | `mass_mailing` |  |
| `mailing.mailing.schedule.date` | `action_schedule_date` | self | `mass_mailing` |  |
| `mailing.sms.test` | `action_send_sms` | self | `mass_mailing_sms` |  |
| `mailing.trace` | `action_view_contact` | self | `mass_mailing` |  |
| `maintenance.equipment` | `action_open_matched_serial` | self | `stock_maintenance` |  |
| `mrp.bom` | `action_compute_bom_days` | self | `mrp` |  |
| `mrp.bom` | `action_copy_existing_operations` | self | `mrp` |  |
| `mrp.bom` | `action_open_operation_form` | self | `mrp` |  |
| `mrp.bom` | `action_set_bom_on_orderpoint` | self | `mrp` |  |
| `mrp.bom.byproduct` | `action_add_from_catalog` | self | `mrp` |  |
| `mrp.bom.line` | `action_add_from_catalog` | self | `mrp` |  |
| `mrp.bom.line` | `action_see_attachments` | self | `mrp` |  |
| `mrp.consumption.warning` | `action_cancel` | self | `mrp` |  |
| `mrp.consumption.warning` | `action_confirm` | self | `mrp` |  |
| `mrp.consumption.warning` | `action_set_qty` | self | `mrp` |  |
| `mrp.production` | `action_assign` | self | `mrp` |  |
| `mrp.production` | `action_cancel` | self | `mrp` | Cancels production order, unfinished stock moves and set procurement orders in exception |
| `mrp.production` | `action_clear_lot_producing_ids` | self | `mrp` |  |
| `mrp.production` | `action_confirm` | self | `mrp`, `project_mrp_account`, `sale_mrp` |  |
| `mrp.production` | `action_generate_bom` | self | `mrp`, `project_mrp` | Generates a new Bill of Material based on the Manufacturing Order's product, components, workorders and by-products, and assigns it to the MO. Returns a new BoM's form view action. |
| `mrp.production` | `action_generate_serial` | self, workorder | `mrp` |  |
| `mrp.production` | `action_merge` | self | `mrp`, `mrp_subcontracting` |  |
| `mrp.production` | `action_open_label_layout` | self | `mrp` |  |
| `mrp.production` | `action_open_label_type` | self | `mrp` |  |
| `mrp.production` | `action_open_project` | self | `project_mrp` |  |
| `mrp.production` | `action_plan_with_components_availability` | self | `mrp` |  |
| `mrp.production` | `action_product_forecast_report` | self | `mrp` |  |
| `mrp.production` | `action_see_move_scrap` | self | `mrp` |  |
| `mrp.production` | `action_split` | self | `mrp` |  |
| `mrp.production` | `action_split_subcontracting` | self | `mrp_subcontracting` |  |
| `mrp.production` | `action_start` | self | `mrp` |  |
| `mrp.production` | `action_toggle_is_locked` | self | `mrp` |  |
| `mrp.production` | `action_update_bom` | self | `mrp` |  |
| `mrp.production` | `action_view_analytic_accounts` | self | `project_mrp_account` |  |
| `mrp.production` | `action_view_mo_delivery` | self | `mrp` | Returns an action that display picking related to manufacturing order. It can either be a list view or in a form view (if there is only one picking to show). |
| `mrp.production` | `action_view_move_wip` | self | `mrp_account` |  |
| `mrp.production` | `action_view_mrp_production_backorders` | self | `mrp` |  |
| `mrp.production` | `action_view_mrp_production_childs` | self | `mrp` |  |
| `mrp.production` | `action_view_mrp_production_sources` | self | `mrp` |  |
| `mrp.production` | `action_view_mrp_production_unbuilds` | self | `mrp` |  |
| `mrp.production` | `action_view_purchase_orders` | self | `purchase_mrp` |  |
| `mrp.production` | `action_view_reception_report` | self | `mrp` |  |
| `mrp.production` | `action_view_repair_orders` | self | `mrp_repair` |  |
| `mrp.production` | `action_view_sale_orders` | self | `sale_mrp` |  |
| `mrp.production` | `action_view_serial_numbers` | self | `mrp` |  |
| `mrp.production` | `button_mark_done` | self | `mrp` |  |
| `mrp.production` | `button_plan` | self | `mrp` | Create work orders. And probably do stuff, like things. |
| `mrp.production` | `button_scrap` | self | `mrp` |  |
| `mrp.production` | `button_unbuild` | self | `mrp`, `mrp_subcontracting` |  |
| `mrp.production` | `button_unplan` | self | `mrp` |  |
| `mrp.production` | `do_unreserve` | self | `mrp` |  |
| `mrp.production.backorder` | `action_backorder` | self | `mrp` |  |
| `mrp.production.backorder` | `action_close_mo` | self | `mrp` |  |
| `mrp.production.serials` | `action_apply` | self | `mrp`, `mrp_subcontracting` |  |
| `mrp.production.serials` | `action_generate_serial_numbers` | self | `mrp` |  |
| `mrp.production.serials` | `action_split_and_assign_serials` | self | `mrp` |  |
| `mrp.production.split` | `action_prepare_split` | self | `mrp` |  |
| `mrp.production.split` | `action_return_to_list` | self | `mrp` |  |
| `mrp.production.split` | `action_split` | self | `mrp` |  |
| `mrp.routing.workcenter` | `action_open_operation_form` | self | `mrp` |  |
| `mrp.unbuild` | `action_unbuild` | self | `mrp` |  |
| `mrp.unbuild` | `action_validate` | self | `mrp` |  |
| `mrp.workcenter` | `action_show_operations` | self | `mrp` |  |
| `mrp.workcenter` | `action_work_order` | self | `mrp` |  |
| `mrp.workcenter` | `action_work_order_alternatives` | self | `mrp` |  |
| `mrp.workcenter.productivity` | `button_block` | self | `mrp` |  |
| `mrp.workorder` | `action_cancel` | self | `mrp`, `mrp_account` |  |
| `mrp.workorder` | `action_mark_as_done` | self | `mrp` |  |
| `mrp.workorder` | `action_open_wizard` | self | `mrp` |  |
| `mrp.workorder` | `action_replan` | self | `mrp` | Replan a work order.  It actually replans every  "ready" or "blocked" work orders of the linked manufacturing orders. |
| `mrp.workorder` | `action_see_move_scrap` | self | `mrp` |  |
| `mrp.workorder` | `button_finish` | self | `mrp` |  |
| `mrp.workorder` | `button_pending` | self | `mrp` |  |
| `mrp.workorder` | `button_scrap` | self | `mrp` |  |
| `mrp.workorder` | `button_start` | self, raise_on_invalid_state | `mrp` |  |
| `mrp.workorder` | `button_unblock` | self | `mrp` |  |
| `myinvois.consolidate.invoice.wizard` | `button_consolidate` | self | `l10n_my_edi` | By default, we only consolidated orders that are in the range, done and not invoiced. We do allow to also consolidate invoices linked to a cancelled consolidated invoice.  Note that doing so lock the cancelled invoice into its cancelled state. |
| `myinvois.document` | `action_cancel_submission` | self | `l10n_my_edi` | Cancel the document on the platform. |
| `myinvois.document` | `action_generate_xml_file` | self | `l10n_my_edi` | Generate a xml file for each of the MyInvois documents in self. If the document already as a file, the previous file's name is updated to include an (old) tag to avoid confusion in the attachment list. |
| `myinvois.document` | `action_open_consolidate_invoice_wizard` | self | `l10n_my_edi_pos` | Open the wizard, and set a default date_from/date_to based on the current date as well as already existing consolidated invoices. |
| `myinvois.document` | `action_show_myinvois_documents` | self | `l10n_my_edi`, `l10n_my_edi_pos` | Open the documents in self in the correct view based on the amount of records. |
| `myinvois.document` | `action_submit_to_myinvois` | self | `l10n_my_edi` | Submit all new documents in self to MyInvois. This can also be used on invalid documents to re-submit them after correcting the error. |
| `myinvois.document` | `action_update_submission_status` | self | `l10n_my_edi` | Fetches the status of all the documents in self. Note that the endpoint reached to do so will differ based on the amount of documents in the recordset. |
| `myinvois.document` | `action_view_linked_orders` | self | `l10n_my_edi_pos` | Return the action used to open the order(s) linked to the selected consolidated invoice. |
| `myinvois.document.status.update.wizard` | `button_request_update` | self | `l10n_my_edi` |  |
| `nemhandel.registration` | `button_check_nemhandel_verification_code` | self | `l10n_dk_nemhandel` | Calls /verify_phone_number to compare user's input and the code generated on the IAP server |
| `nemhandel.registration` | `button_deregister_nemhandel_participant` | self | `l10n_dk_nemhandel` | Deregister the edi user from Nemhandel network |
| `nemhandel.registration` | `button_nemhandel_receiver_registration` | self | `l10n_dk_nemhandel` | The user is registered on the Nemhandel network, i.e. can receive documents from other Nemhandel participants. |
| `nemhandel.registration` | `button_nemhandel_registration_sms` | self | `l10n_dk_nemhandel` | The first step of the Nemhandel onboarding. - Creates an EDI proxy user on the iap side, then the client side - Calls /activate_participant to mark the EDI user as nemhandel user - Sends an SMS code |
| `nemhandel.registration` | `button_update_nemhandel_user_data` | self | `l10n_dk_nemhandel` | Action for the user to be able to update their contact details any time Calls /update_user on the iap server |
| `nemhandel.rejection.wizard` | `button_send` | self | `l10n_dk_nemhandel_response` |  |
| `onboarding.onboarding` | `action_close` | self | `onboarding` | Close the onboarding panel. |
| `onboarding.onboarding` | `action_close_panel` | self, xmlid | `onboarding` | Close the onboarding panel identified by its `xmlid`.  If not found, quietly do nothing. |
| `onboarding.onboarding` | `action_close_panel_account_dashboard` | self | `account` |  |
| `onboarding.onboarding` | `action_close_panel_account_invoice` | self | `account` |  |
| `onboarding.onboarding` | `action_refresh_progress_ids` | self | `onboarding` | Re-initialize onboarding progress records (after step is_per_company change).  Meant to be called when `is_per_company` of linked steps is modified (or per-company steps are added to an onboarding). |
| `onboarding.onboarding` | `action_toggle_visibility` | self | `onboarding` |  |
| `onboarding.onboarding.step` | `action_open_step_bank_account` | self | `account` |  |
| `onboarding.onboarding.step` | `action_open_step_base_document_layout` | self | `account` |  |
| `onboarding.onboarding.step` | `action_open_step_chart_of_accounts` | self | `account` | Called by the 'Chart of Accounts' button of the dashboard onboarding panel. |
| `onboarding.onboarding.step` | `action_open_step_company_data` | self | `account` | Set company's basic information. |
| `onboarding.onboarding.step` | `action_open_step_create_invoice` | self | `account` |  |
| `onboarding.onboarding.step` | `action_open_step_fiscal_year` | self | `account` |  |
| `onboarding.onboarding.step` | `action_open_step_sales_tax` | self | `account` |  |
| `onboarding.onboarding.step` | `action_set_just_done` | self | `onboarding` |  |
| `onboarding.onboarding.step` | `action_validate_step` | self, xml_id | `onboarding` |  |
| `onboarding.onboarding.step` | `action_validate_step_base_document_layout` | self | `account` | Set the onboarding(s) step as done only if layout is set. |
| `onboarding.progress` | `action_close` | self | `onboarding` |  |
| `onboarding.progress` | `action_toggle_visibility` | self | `onboarding` |  |
| `onboarding.progress.step` | `action_consolidate_just_done` | self | `onboarding` |  |
| `onboarding.progress.step` | `action_set_just_done` | self | `onboarding` |  |
| `payment.capture.wizard` | `action_capture` | self | `payment` |  |
| `payment.provider` | `action_paypal_create_webhook` | self | `payment_paypal` | Create a new webhook.  Note: This action only works for instances using a public URL.  :return: None :raise UserError: If the base URL is not in HTTPS. |
| `payment.provider` | `action_razorpay_create_webhook` | self | `payment_razorpay` | Create a webhook and display a toast notification.  Note: `self.ensure_one()`  :return: The feedback notification. :rtype: dict |
| `payment.provider` | `action_recompute_pending_msg` | self | `payment_custom` | Recompute the pending message to include the existing bank accounts. |
| `payment.provider` | `action_reset_credentials` | self | `payment` | Reset the credentials of the provider, disable it, and unpublish it.  Note: self.ensure_one()  :return: The result of the write operation. :rtype: bool |
| `payment.provider` | `action_start_onboarding` | self, menu_id | `payment`, `payment_mercado_pago`, `payment_payu`, `payment_razorpay`, `payment_stripe` | Start the provider-specific onboarding.  Providers implementing a specific onboarding must override this method and return the action to run the onboarding.  :param int menu_id: The menu from which the onboarding is started, as an `ir.ui.menu` id. :return: The onboarding action. :rtype: dict |
| `payment.provider` | `action_stripe_create_webhook` | self | `payment_stripe` | Create a webhook and return a feedback notification.  Note: This action only works for instances using a public URL  :return: The feedback notification :rtype: dict |
| `payment.provider` | `action_stripe_verify_apple_pay_domain` | self | `payment_stripe` | Verify the web domain with Stripe to enable Apple Pay.  The domain is sent to Stripe API for them to verify that it is valid by making a request to the `/.well-known/apple-developer-merchantid-domain-association` route. If the domain is valid, it is registered to use with Apple Pay. See https://stri |
| `payment.provider` | `action_sync_paymob_payment_methods` | self | `payment_paymob` | Synchronize the payment methods with the ones on the Paymob portal, the integration_name needs to be set to be able to communicate with the `payment_method.code` when the intention is created.  :return: A notification with the status of the action. :rtype: dict |
| `payment.provider` | `action_toggle_is_published` | self | `payment` | Toggle the field `is_published`.  :return: None :raise UserError: If the provider is disabled. |
| `payment.provider` | `action_update_merchant_details` | self | `payment_authorize` | Fetch the merchant details to update the client key and the account currency. |
| `payment.provider` | `action_view_payment_methods` | self | `payment` |  |
| `payment.provider` | `button_immediate_install` | self | `payment` | Install the module and reload the page.  Note: `self.ensure_one()`  :return: The action to reload the page. :rtype: dict |
| `payment.refund.wizard` | `action_refund` | self | `account_payment` |  |
| `payment.transaction` | `action_capture` | self | `payment` | Open the partial capture wizard if it is supported by the related providers, otherwise capture the transactions immediately.  :return: The action to open the partial capture wizard, if supported. :rtype: action.act_window\|None |
| `payment.transaction` | `action_demo_set_canceled` | self | `payment_demo` | Set the state of the demo transaction to 'cancel'.  Note: self.ensure_one()  :return: None |
| `payment.transaction` | `action_demo_set_done` | self | `payment_demo` | Set the state of the demo transaction to 'done'.  Note: self.ensure_one()  :return: None |
| `payment.transaction` | `action_demo_set_error` | self | `payment_demo` | Set the state of the demo transaction to 'error'.  Note: self.ensure_one()  :return: None |
| `payment.transaction` | `action_post_process` | self | `payment` | Trigger the post-processing of the transactions.  :return: A client action to soft-reload the view. :rtype: dict |
| `payment.transaction` | `action_refund` | self, amount_to_refund | `payment` | Check the state of the transactions and request their refund.  :param float amount_to_refund: The amount to be refunded. :return: None |
| `payment.transaction` | `action_view_invoices` | self | `account_payment` | Return the action for the views of the invoices linked to the transaction.  Note: self.ensure_one()  :return: The action :rtype: dict |
| `payment.transaction` | `action_view_pos_order` | self | `pos_online_payment` | Return the action for the view of the pos order linked to the transaction. |
| `payment.transaction` | `action_view_refunds` | self | `payment` | Return the windows action to browse the refund transactions linked to the transaction.  Note: `self.ensure_one()`  :return: The window action to browse the refund transactions. :rtype: dict |
| `payment.transaction` | `action_view_sales_orders` | self | `sale` |  |
| `payment.transaction` | `action_void` | self | `payment` | Check the state of the transaction and request to have them voided. |
| `pdp.config.wizard` | `button_peppol_unregister` | self | `l10n_fr_pdp` | Unregister the user from Peppol network. |
| `pdp.config.wizard` | `button_sync_form_with_peppol_proxy` | self | `l10n_fr_pdp` | Update the peppol contact email on IAP. Note: The service configuration is DEPRECATED / hidden in the view. Disabling services can lead to complicance issues and is not necessary since all existing services should just work. |
| `pdp.registration` | `button_cancel_authentication` | self | `l10n_fr_pdp` |  |
| `pdp.registration` | `button_deregister_pdp_participant` | self | `l10n_fr_pdp` | Deregister the edi user from PDP network |
| `pdp.registration` | `button_open_authentication_link` | self | `l10n_fr_pdp` |  |
| `pdp.registration` | `button_refresh_authentication` | self | `l10n_fr_pdp` |  |
| `pdp.registration` | `button_register_pdp_participant` | self | `l10n_fr_pdp` |  |
| `pdp.registration` | `button_trigger_authentication` | self | `l10n_fr_pdp` |  |
| `pdp.response.wizard` | `button_send` | self | `l10n_fr_pdp` |  |
| `peppol.config.wizard` | `button_peppol_register_sender_as_receiver` | self | `account_peppol` | Reset the participant back to sender and unregister it from the SMP |
| `peppol.config.wizard` | `button_peppol_reset_to_sender` | self | `account_peppol` | Reset the participant back to sender and unregister it from the SMP |
| `peppol.config.wizard` | `button_peppol_unregister` | self | `account_peppol` | Unregister the user from Peppol network. |
| `peppol.config.wizard` | `button_sync_form_with_peppol_proxy` | self | `account_peppol` | Update the peppol contact email on IAP. Note: The service configuration is DEPRECATED / hidden in the view. Disabling services can lead to complicance issues and is not necessary since all existing services should just work. |
| `peppol.registration` | `button_register_peppol_participant` | self, selected_auth | `account_peppol` |  |
| `peppol.registration` | `button_register_with_itsme` | self | `account_peppol` |  |
| `phone.blacklist` | `action_add` | self | `phone_validation` |  |
| `phone.blacklist.remove` | `action_unblacklist_apply` | self | `phone_validation` |  |
| `portal.mixin` | `action_share` | self | `portal` |  |
| `portal.share` | `action_send_mail` | self | `portal`, `project` |  |
| `portal.wizard` | `action_open_wizard` | self | `portal` | Create a "portal.wizard" and open the form view.  We need a server action for that because the one2many "user_ids" records need to exist to be able to execute an a button action on it. If they have no ID, the buttons will be disabled and we won't be able to click on them.  That's why we need a serve |
| `portal.wizard.user` | `action_grant_access` | self | `portal` | Grant the portal access to the partner.  If the partner has no linked user, we will create a new one in the same company as the partner (or in the current company if not set).  An invitation email will be sent to the partner. |
| `portal.wizard.user` | `action_invite_again` | self | `portal` | Re-send the invitation email to the partner. |
| `portal.wizard.user` | `action_refresh_modal` | self | `portal` | Refresh the portal wizard modal and keep it open. Used as fallback action of email state icon buttons, required as they must be non-disabled buttons to fire mouse events to show tooltips on email state. |
| `portal.wizard.user` | `action_revoke_access` | self | `portal` | Archive the portal user of the partner.  User is kept in `group_portal` as `group_public` should only be used for automated tasks and guest interactions. |
| `pos.config` | `action_close_kiosk_session` | self | `pos_self_order` |  |
| `pos.config` | `action_open_wizard` | self | `pos_self_order` |  |
| `pos.config` | `action_pos_config_modal_edit` | self | `point_of_sale` |  |
| `pos.confirmation.wizard` | `action_confirm` | self | `point_of_sale` |  |
| `pos.make.invoice` | `action_create_invoices` | self | `point_of_sale` |  |
| `pos.order` | `action_create_invoices` | self | `point_of_sale` |  |
| `pos.order` | `action_pos_order_cancel` | self | `point_of_sale`, `pos_self_order` |  |
| `pos.order` | `action_pos_order_invoice` | self | `point_of_sale` |  |
| `pos.order` | `action_pos_order_paid` | self | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_jo_edi_pos`, `point_of_sale`, `pos_event`, `pos_restaurant_adyen`, `pos_sale` | Once an order is paid, sync it with JoFotara if possible |
| `pos.order` | `action_send_mail` | self | `point_of_sale` |  |
| `pos.order` | `action_send_receipt` | self, email, ticket_image, basic_image | `point_of_sale` |  |
| `pos.order` | `action_send_self_order_receipt` | self, email, mail_template_id, ticket_image, basic_image | `pos_self_order` |  |
| `pos.order` | `action_sent_message_on_sms` | self, phone, _, basic_image | `pos_sms` |  |
| `pos.order` | `action_show_myinvois_documents` | self | `l10n_my_edi_pos` |  |
| `pos.order` | `action_stock_picking` | self | `point_of_sale` |  |
| `pos.order` | `action_view_attendee_list` | self | `pos_event` |  |
| `pos.order` | `action_view_invoice` | self | `point_of_sale` |  |
| `pos.order` | `action_view_refund_orders` | self | `point_of_sale` |  |
| `pos.order` | `action_view_refunded_order` | self | `point_of_sale` |  |
| `pos.order` | `action_view_sale_order` | self | `pos_sale` |  |
| `pos.order` | `button_l10n_jo_edi_pos` | self | `l10n_jo_edi_pos` |  |
| `pos.payment.method` | `action_stripe_key` | self | `pos_stripe` |  |
| `pos.preset` | `action_open_linked_config` | self | `point_of_sale` |  |
| `pos.preset` | `action_open_linked_orders` | self | `point_of_sale` |  |
| `pos.session` | `action_pos_session_close` | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |
| `pos.session` | `action_pos_session_closing_control` | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |
| `pos.session` | `action_pos_session_open` | self | `point_of_sale` |  |
| `pos.session` | `action_pos_session_validate` | self, balancing_account, amount_to_balance, bank_payment_method_diffs | `point_of_sale` |  |
| `pos.session` | `action_show_payments_list` | self | `point_of_sale` |  |
| `pos.session` | `action_stock_picking` | self | `point_of_sale` |  |
| `pos.session` | `action_view_order` | self | `point_of_sale` |  |
| `privacy.lookup.wizard` | `action_lookup` | self | `privacy_lookup` |  |
| `privacy.lookup.wizard` | `action_open_lines` | self | `privacy_lookup` |  |
| `privacy.lookup.wizard.line` | `action_archive_all` | self | `privacy_lookup` |  |
| `privacy.lookup.wizard.line` | `action_open_record` | self | `privacy_lookup` |  |
| `privacy.lookup.wizard.line` | `action_unlink` | self | `privacy_lookup` |  |
| `privacy.lookup.wizard.line` | `action_unlink_all` | self | `privacy_lookup` |  |
| `product.attribute` | `action_open_product_template_attribute_lines` | self | `product` |  |
| `product.attribute.value` | `action_add_to_products` | self | `product` |  |
| `product.attribute.value` | `action_update_prices` | self | `product` |  |
| `product.catalog.mixin` | `action_add_from_catalog` | self | `product` |  |
| `product.document` | `action_open_pdf_form_fields` | self | `sale_pdf_quote_builder` |  |
| `product.feed` | `action_invalidate_cache` | self | `website_sale` |  |
| `product.margin` | `action_open_window` | self | `product_margin` |  |
| `product.pricelist` | `action_open_pricelist_report` | self | `product` |  |
| `product.product` | `action_bom_cost` | self | `mrp_account` |  |
| `product.product` | `action_open_documents` | self | `product` |  |
| `product.product` | `action_open_label_layout` | self | `product` |  |
| `product.product` | `action_open_product_lot` | self | `stock` |  |
| `product.product` | `action_open_quants` | self | `mrp`, `stock` |  |
| `product.product` | `action_product_forecast_report` | self | `stock` |  |
| `product.product` | `action_used_in_bom` | self | `mrp` |  |
| `product.product` | `action_view_bom` | self | `mrp` |  |
| `product.product` | `action_view_mos` | self | `mrp` |  |
| `product.product` | `action_view_orderpoints` | self | `stock` |  |
| `product.product` | `action_view_po` | self | `purchase` |  |
| `product.product` | `action_view_related_putaway_rules` | self | `stock` |  |
| `product.product` | `action_view_routes` | self | `stock` |  |
| `product.product` | `action_view_sales` | self | `sale` |  |
| `product.product` | `action_view_stock_move_lines` | self | `stock` |  |
| `product.product` | `action_view_storage_category_capacity` | self | `stock` |  |
| `product.product` | `button_bom_cost` | self | `mrp_account` |  |
| `product.replenish` | `action_stock_replenishment_info` | self | `purchase_stock` |  |
| `product.supplierinfo` | `action_set_supplier` | self | `purchase_stock` |  |
| `product.template` | `action_bom_cost` | self | `mrp_account` |  |
| `product.template` | `action_create_product_variants_from_gelato_template` | self | `website_sale_gelato` | Override of `sale_gelato` to unpublish products for which the synchronization with Gelato led to new print images being created. |
| `product.template` | `action_open_documents` | self | `product` |  |
| `product.template` | `action_open_label_layout` | self | `product` |  |
| `product.template` | `action_open_product_lot` | self | `stock` |  |
| `product.template` | `action_open_quants` | self | `stock` |  |
| `product.template` | `action_open_routes_diagram` | self | `stock` |  |
| `product.template` | `action_product_tmpl_forecast_report` | self | `stock` |  |
| `product.template` | `action_sync_gelato_template_info` | self | `sale_gelato` | Fetch the template information from Gelato and update the product template accordingly.  :return: The action to display a toast notification to the user. :rtype: dict |
| `product.template` | `action_used_in_bom` | self | `mrp` |  |
| `product.template` | `action_view_mos` | self | `mrp` |  |
| `product.template` | `action_view_orderpoints` | self | `stock` |  |
| `product.template` | `action_view_po` | self | `purchase` |  |
| `product.template` | `action_view_related_putaway_rules` | self | `stock` |  |
| `product.template` | `action_view_sales` | self | `sale` |  |
| `product.template` | `action_view_stock_move_lines` | self | `stock` |  |
| `product.template` | `action_view_storage_category_capacity` | self | `stock` |  |
| `product.template` | `button_bom_cost` | self | `mrp_account` |  |
| `product.template.attribute.line` | `action_open_attribute_values` | self | `product` |  |
| `project.milestone` | `action_view_sale_order` | self | `sale_project` |  |
| `project.milestone` | `action_view_tasks` | self | `project` |  |
| `project.project` | `action_billable_time_button` | self | `sale_timesheet` |  |
| `project.project` | `action_create_from_template` | self, values, role_to_users_mapping | `project` |  |
| `project.project` | `action_create_invoice` | self | `sale_project` |  |
| `project.project` | `action_create_template_from_project` | self | `project` |  |
| `project.project` | `action_customer_preview` | self | `sale_project` |  |
| `project.project` | `action_get_list_view` | self | `project`, `sale_project` |  |
| `project.project` | `action_open_all_pickings` | self | `project_stock` |  |
| `project.project` | `action_open_analytic_items` | self | `project_account` |  |
| `project.project` | `action_open_deliveries` | self | `project_stock` |  |
| `project.project` | `action_open_project_expenses` | self | `project_hr_expense` |  |
| `project.project` | `action_open_project_invoices` | self | `sale_project` |  |
| `project.project` | `action_open_project_purchase_orders` | self | `project_purchase` |  |
| `project.project` | `action_open_project_vendor_bills` | self | `sale_project` |  |
| `project.project` | `action_open_receipts` | self | `project_stock` |  |
| `project.project` | `action_open_share_project_wizard` | self | `project` |  |
| `project.project` | `action_profitability_items` | self, section_name, domain, res_id | `project`, `project_account`, `project_hr_expense`, `project_purchase`, `sale_project`, `sale_timesheet` |  |
| `project.project` | `action_project_task_burndown_chart_report` | self | `project` |  |
| `project.project` | `action_project_timesheets` | self | `hr_timesheet`, `sale_timesheet` |  |
| `project.project` | `action_toggle_project_template_mode` | self | `project` |  |
| `project.project` | `action_undo_convert_to_template` | self | `project` |  |
| `project.project` | `action_view_all_rating` | self | `project` | return the action to see all the rating of the project and activate default filters |
| `project.project` | `action_view_mrp_bom` | self | `project_mrp` |  |
| `project.project` | `action_view_mrp_production` | self | `project_mrp` |  |
| `project.project` | `action_view_sols` | self | `sale_project` |  |
| `project.project` | `action_view_sos` | self | `sale_project` |  |
| `project.project` | `action_view_tasks` | self | `hr_timesheet`, `project`, `sale_project` |  |
| `project.project` | `action_view_tasks_analysis` | self | `project` | return the action to see the tasks analysis report of the project |
| `project.project` | `action_view_tasks_from_project_milestone` | self | `project` |  |
| `project.project` | `action_view_timesheet` | self | `sale_timesheet` |  |
| `project.project.stage.delete.wizard` | `action_unarchive_project` | self | `project` |  |
| `project.project.stage.delete.wizard` | `action_unlink` | self | `project` |  |
| `project.share.wizard` | `action_send_mail` | self | `project` |  |
| `project.share.wizard` | `action_share_record` | self | `project` |  |
| `project.task` | `action_convert_to_subtask` | self | `project` |  |
| `project.task` | `action_convert_to_task` | self | `project_todo` |  |
| `project.task` | `action_convert_to_template` | self | `project` |  |
| `project.task` | `action_create_from_template` | self, values | `project` |  |
| `project.task` | `action_dependent_tasks` | self | `project` |  |
| `project.task` | `action_open_parent_task` | self | `project` |  |
| `project.task` | `action_open_ratings` | self | `project` |  |
| `project.task` | `action_open_subtasks` | self | `project` |  |
| `project.task` | `action_open_task` | self | `project` |  |
| `project.task` | `action_project_sharing_open_blocking` | self | `project` |  |
| `project.task` | `action_project_sharing_open_subtasks` | self | `project` |  |
| `project.task` | `action_project_sharing_open_task` | self | `project` |  |
| `project.task` | `action_project_sharing_recurring_tasks` | self | `project` |  |
| `project.task` | `action_project_sharing_view_parent_task` | self | `project` |  |
| `project.task` | `action_project_sharing_view_so` | self | `sale_project` |  |
| `project.task` | `action_recurring_tasks` | self | `project` |  |
| `project.task` | `action_redirect_to_project_task_form` | self | `project` |  |
| `project.task` | `action_undo_convert_to_template` | self | `project` |  |
| `project.task` | `action_unlink_recurrence` | self | `project` |  |
| `project.task` | `action_view_so` | self | `sale_project` |  |
| `project.task` | `action_view_subtask_timesheet` | self | `hr_timesheet` |  |
| `project.task.type.delete.wizard` | `action_confirm` | self | `project` |  |
| `project.task.type.delete.wizard` | `action_unarchive_task` | self | `project` |  |
| `project.task.type.delete.wizard` | `action_unlink` | self | `project` |  |
| `project.template.create.wizard` | `action_create_project_from_so` | self | `sale_project` | Create a project either from template or directly if no template is set. |
| `project.template.create.wizard` | `action_open_template_view` | self | `project`, `sale_project` |  |
| `purchase.bill.line.match` | `action_add_to_po` | self | `purchase` |  |
| `purchase.bill.line.match` | `action_match_lines` | self | `purchase` |  |
| `purchase.bill.line.match` | `action_open_line` | self | `purchase` |  |
| `purchase.order` | `action_acknowledge` | self | `purchase` |  |
| `purchase.order` | `action_add_from_catalog` | self | `purchase`, `purchase_stock` |  |
| `purchase.order` | `action_bill_matching` | self | `purchase` |  |
| `purchase.order` | `action_compare_alternative_lines` | self | `purchase_requisition` |  |
| `purchase.order` | `action_create_alternative` | self | `purchase_requisition` |  |
| `purchase.order` | `action_create_invoice` | self, attachment_ids | `purchase` | Create the invoice associated to the PO. |
| `purchase.order` | `action_merge` | self | `purchase` |  |
| `purchase.order` | `action_open_business_doc` | self | `purchase` |  |
| `purchase.order` | `action_purchase_comparison` | self | `purchase` |  |
| `purchase.order` | `action_purchase_order_suggest` | self | `purchase_stock` | Adds suggested products to PO, removing products with no suggested_qty, and collapsing existing po_lines into at most 1 orderline. Saves suggestion params (eg. number_of_days) to partner table. |
| `purchase.order` | `action_rfq_send` | self | `purchase` | This function opens a window to compose an email, with the edi purchase template message loaded by default |
| `purchase.order` | `action_view_dropship` | self | `stock_dropshipping` |  |
| `purchase.order` | `action_view_invoice` | self, invoices | `purchase` | This function returns an action that display existing vendor bills of given purchase order ids. When only one found, show the vendor bill immediately. |
| `purchase.order` | `action_view_mrp_productions` | self | `purchase_mrp` |  |
| `purchase.order` | `action_view_picking` | self | `purchase_stock`, `stock_dropshipping` |  |
| `purchase.order` | `action_view_repair_orders` | self | `purchase_repair` |  |
| `purchase.order` | `action_view_sale_orders` | self | `sale_purchase` |  |
| `purchase.order` | `action_view_subcontracting_resupply` | self | `mrp_subcontracting_purchase` |  |
| `purchase.order` | `button_approve` | self, force | `purchase`, `purchase_stock` |  |
| `purchase.order` | `button_cancel` | self | `purchase`, `purchase_stock`, `sale_purchase` |  |
| `purchase.order` | `button_confirm` | self | `purchase`, `purchase_requisition` |  |
| `purchase.order` | `button_draft` | self | `purchase` |  |
| `purchase.order` | `button_lock` | self | `purchase` |  |
| `purchase.order` | `button_unlock` | self | `purchase` |  |
| `purchase.order.line` | `action_add_from_catalog` | self | `purchase` |  |
| `purchase.order.line` | `action_choose` | self | `purchase_requisition` |  |
| `purchase.order.line` | `action_clear_quantities` | self | `purchase_requisition` |  |
| `purchase.order.line` | `action_open_order` | self | `purchase` |  |
| `purchase.order.line` | `action_product_forecast_report` | self | `purchase_stock` |  |
| `purchase.requisition` | `action_cancel` | self | `purchase_requisition` |  |
| `purchase.requisition` | `action_confirm` | self | `purchase_requisition` |  |
| `purchase.requisition` | `action_done` | self | `purchase_requisition` | Generate all purchase order based on selected lines, should only be called on one agreement at a time |
| `purchase.requisition` | `action_draft` | self | `purchase_requisition` |  |
| `purchase.requisition.alternative.warning` | `action_cancel_alternatives` | self | `purchase_requisition` |  |
| `purchase.requisition.alternative.warning` | `action_keep_alternatives` | self | `purchase_requisition` |  |
| `purchase.requisition.create.alternative` | `action_create_alternative` | self | `purchase_requisition` |  |
| `quotation.document` | `action_open_pdf_form_fields` | self | `sale_pdf_quote_builder` |  |
| `rating.rating` | `action_open_rated_object` | self | `im_livechat`, `rating` |  |
| `registration.editor` | `action_make_registration` | self | `event_sale` |  |
| `repair.order` | `action_add_from_catalog` | self | `repair` |  |
| `repair.order` | `action_assign` | self | `repair` |  |
| `repair.order` | `action_create_sale_order` | self | `repair` |  |
| `repair.order` | `action_explode` | self | `mrp_repair` |  |
| `repair.order` | `action_generate_serial` | self | `repair` |  |
| `repair.order` | `action_repair_cancel` | self | `repair` |  |
| `repair.order` | `action_repair_cancel_draft` | self | `repair` |  |
| `repair.order` | `action_repair_done` | self | `repair` | Creates stock move for final product of repair order. Writes move_id and move_ids state to 'done'. Writes repair order state to 'Repaired'. @return: True |
| `repair.order` | `action_repair_end` | self | `repair` | Checks before action_repair_done. @return: True |
| `repair.order` | `action_repair_start` | self | `repair` | Writes repair order state to 'Under Repair' |
| `repair.order` | `action_unreserve` | self | `repair` |  |
| `repair.order` | `action_validate` | self | `repair` |  |
| `repair.order` | `action_view_mrp_productions` | self | `mrp_repair` |  |
| `repair.order` | `action_view_purchase_orders` | self | `purchase_repair` |  |
| `repair.order` | `action_view_sale_order` | self | `repair` |  |
| `report.stock.report_reception` | `action_assign` | self, move_ids, qtys, in_ids | `stock` | Assign picking move(s) [i.e. link] to other moves (i.e. make them MTO) :param move_id ids: the ids of the moves to make MTO :param qtys list: the quantities that are being assigned to the move_ids (in same order as move_ids) :param in_ids ids: the ids of the moves that are to be assigned to move_ids |
| `report.stock.report_reception` | `action_unassign` | self, move_id, qty, in_ids | `stock` | Unassign moves [i.e. unlink] from a move (i.e. make non-MTO) :param move_id id: the id of the move to make non-MTO :param qty float: the total quantity that is being unassigned from move_id :param in_ids ids: the ids of the moves that are to be unassigned from move_id |
| `res.company` | `action_all_company_branches` | self | `base` |  |
| `res.company` | `action_close_stock_valuation` | self, at_date, auto_post | `stock_account` |  |
| `res.company` | `action_open_website_theme_selector` | self | `website` |  |
| `res.company` | `action_save_onboarding_company_data` | self | `account` |  |
| `res.company` | `action_save_onboarding_sale_tax` | self | `account` | Set the onboarding step as done |
| `res.company` | `action_update_state_as_per_gstin` | self | `l10n_in` |  |
| `res.config` | `action_cancel` | self | `base` | Action handler for the ``cancel`` event. That event isn't generated by the res.config.view.base inheritable view, the inherited view has to overload one of the buttons (or add one more).  Sets the status of the todo the event was sent from to ``cancel``, calls ``cancel`` and -- unless ``cancel`` ret |
| `res.config` | `action_next` | self | `base` | Action handler for the ``next`` event.  Sets the status of the todo the event was sent from to ``done``, calls ``execute`` and -- unless ``execute`` returned an action dictionary -- executes the action provided by calling ``next``. |
| `res.config` | `action_skip` | self | `base` | Action handler for the ``skip`` event.  Sets the status of the todo the event was sent from to ``skip``, calls ``cancel`` and -- unless ``cancel`` returned an action dictionary -- executes the action provided by calling ``next``. |
| `res.config.settings` | `action_crm_assign_leads` | self | `crm` |  |
| `res.config.settings` | `action_eu_oss_tax_mapping` | self | `account` |  |
| `res.config.settings` | `action_l10n_my_edi_allow_processing` | self | `l10n_my_edi` | We always expect the user to give his consent by pressing the button, in any mode, to enable the edi. |
| `res.config.settings` | `action_l10n_my_edi_unregister` | self | `l10n_my_edi` | Send a notification to the proxy to free the ID (vat) of the user, and archive the local proxy user. Useful if there has been a misconfiguration or the user wishes to use a new database/... |
| `res.config.settings` | `action_open_abandoned_cart_mail_template` | self | `website_sale` |  |
| `res.config.settings` | `action_open_blocked_third_party_domains` | self | `website` |  |
| `res.config.settings` | `action_open_cloud_storage_migration_configurations` | self | `cloud_storage_migration` |  |
| `res.config.settings` | `action_open_company_form` | self | `l10n_my_edi` | This will be used to ease the configuration by allowing to quickly access the company. |
| `res.config.settings` | `action_open_extra_info` | self | `website_sale` |  |
| `res.config.settings` | `action_open_nemhandel_form` | self | `l10n_dk_nemhandel` |  |
| `res.config.settings` | `action_open_pdp_form` | self | `l10n_fr_pdp` |  |
| `res.config.settings` | `action_open_peppol_form` | self | `account_peppol`, `l10n_fr_pdp` |  |
| `res.config.settings` | `action_open_product_feeds` | self | `website_sale` | Open the list view to manage the feed specific to the current website. |
| `res.config.settings` | `action_open_robots` | self | `website` |  |
| `res.config.settings` | `action_open_sale_mail_templates` | self | `website_sale` |  |
| `res.config.settings` | `action_open_sms_twilio_account_manage` | self | `sms_twilio` |  |
| `res.config.settings` | `action_open_template_user` | self | `base` |  |
| `res.config.settings` | `action_pos_config_create_new` | self | `point_of_sale` |  |
| `res.config.settings` | `action_pos_printer_dialog` | self | `point_of_sale` |  |
| `res.config.settings` | `action_sale_start_payment_onboarding` | self | `sale` |  |
| `res.config.settings` | `action_update_terms` | self | `account` |  |
| `res.config.settings` | `action_view_active_provider` | self | `payment` |  |
| `res.config.settings` | `action_view_delivery_provider_modules` | self | `website_sale` |  |
| `res.config.settings` | `action_view_in_store_delivery_methods` | self | `website_sale_collect` | Return an action to browse pickup delivery methods in list view, or in form view if there is only one. |
| `res.config.settings` | `action_w_payment_start_payment_onboarding` | self | `website_payment` |  |
| `res.config.settings` | `action_website_create_new` | self | `website` |  |
| `res.config.settings` | `button_deregister_nemhandel_participant` | self | `l10n_dk_nemhandel` | Deregister the edi user from Nemhandel network |
| `res.config.settings` | `button_disconnect_this_database` | self | `account_peppol` | Disconnect the current database from the Peppol network. This does not delete or affect the IAP connection, which will remain intact. So don't use this to deregister the participant/connection. |
| `res.config.settings` | `button_l10n_hr_activate_mojeracun` | self | `l10n_hr_edi` |  |
| `res.config.settings` | `button_l10n_hr_deactivate_mojeracun` | self | `l10n_hr_edi` |  |
| `res.config.settings` | `button_l10n_ro_edi_generate_token` | self | `l10n_ro_edi` | Redirects to controllers/main.py ~ `authorize` method |
| `res.config.settings` | `button_open_pdp_config_wizard` | self | `l10n_fr_pdp` |  |
| `res.config.settings` | `button_open_peppol_config_wizard` | self | `account_peppol`, `l10n_fr_pdp` |  |
| `res.config.settings` | `button_peppol_deregister` | self | `account_peppol` | Unregister the user from Peppol network. |
| `res.config.settings` | `button_peppol_disconnect_branch_from_parent` | self | `account_peppol` |  |
| `res.config.settings` | `button_peppol_register_sender_as_receiver` | self | `account_peppol` | Register the existing user as a receiver. |
| `res.config.settings` | `button_peppol_reregister` | self | `account_peppol` |  |
| `res.config.settings` | `button_reconnect_this_database` | self | `account_peppol` | Re-establish an out-of-sync connection |
| `res.config.settings` | `button_update_nemhandel_user_data` | self | `l10n_dk_nemhandel` | Action for the user to be able to update their contact details any time Calls /update_user on the iap server |
| `res.groups` | `action_show_all_users` | self | `base` |  |
| `res.lang` | `action_activate_langs` | self | `base`, `website` | Activate the selected languages |
| `res.partner` | `action_event_view` | self | `event` |  |
| `res.partner` | `action_l10n_in_verify_gstin_status` | self | `l10n_in` |  |
| `res.partner` | `action_open_business_doc` | self | `account` |  |
| `res.partner` | `action_open_employees` | self | `hr` |  |
| `res.partner` | `action_privacy_lookup` | self | `privacy_lookup` |  |
| `res.partner` | `action_signup_prepare` | self | `auth_signup` |  |
| `res.partner` | `action_update_state_as_per_gstin` | self | `l10n_in` |  |
| `res.partner` | `action_validate_tin` | self | `l10n_my_edi` | Calling this action will reach our EDI proxy in order to validate the TIN against the provided identification information. |
| `res.partner` | `action_view_certifications` | self | `survey` |  |
| `res.partner` | `action_view_courses` | self | `website_slides` | View partners courses. In singleton mode, return courses followed by all its contacts (if company) or by themselves (if not a company). Otherwise simply set a domain on required partners. The courses to which the partner(s) is not enrolled (e.g. invited) are not shown. |
| `res.partner` | `action_view_livechat_sessions` | self | `im_livechat` |  |
| `res.partner` | `action_view_loyalty_cards` | self | `loyalty` |  |
| `res.partner` | `action_view_opportunity` | self | `crm` |  |
| `res.partner` | `action_view_partner_invoices` | self | `account` |  |
| `res.partner` | `action_view_pos_order` | self | `point_of_sale` | This function returns an action that displays the pos orders from partner. |
| `res.partner` | `action_view_stock_serial` | self | `stock` |  |
| `res.partner` | `action_view_tasks` | self | `project` |  |
| `res.partner` | `button_account_peppol_check_partner_endpoint` | self, company | `account_peppol`, `account_peppol_response`, `l10n_fr_pdp` | A basic check for whether a participant is reachable at the given Peppol participant ID - peppol_eas:peppol_endpoint (ex: '9999:test') The SML (Service Metadata Locator) assigns a DNS name to each peppol participant. This DNS name resolves into the SMP (Service Metadata Publisher) of the participant |
| `res.partner` | `button_nemhandel_check_partner_endpoint` | self, company | `l10n_dk_nemhandel`, `l10n_dk_nemhandel_response` | A basic check for whether a participant is reachable at the given identifier_type and identifier_value |
| `res.partner.bank` | `action_archive_bank` | self | `base` | Custom archive function because the basic action_archive don't trigger a re-rendering of the page, so the archived value is still visible in the view. |
| `res.partner.bank` | `action_open_allocation_wizard` | self | `hr` |  |
| `res.partner.bank` | `action_open_business_doc` | self | `account` |  |
| `res.users` | `action_change_password_wizard` | self | `base` |  |
| `res.users` | `action_create_employee` | self | `hr` |  |
| `res.users` | `action_create_passkey` | self | `auth_passkey` |  |
| `res.users` | `action_get` | self | `base`, `hr` |  |
| `res.users` | `action_karma_report` | self | `gamification` |  |
| `res.users` | `action_open_employees` | self | `hr` |  |
| `res.users` | `action_open_my_account_settings` | self | `auth_totp_mail` |  |
| `res.users` | `action_related_contact` | self | `hr` |  |
| `res.users` | `action_reset_password` | self | `auth_signup` |  |
| `res.users` | `action_revoke_all_devices` | self | `base` |  |
| `res.users` | `action_setup_outgoing_mail_server` | self, server_type | `mail` | Configure the outgoing mail servers. |
| `res.users` | `action_show_accesses` | self | `base` |  |
| `res.users` | `action_show_groups` | self | `base` |  |
| `res.users` | `action_show_rules` | self | `base` |  |
| `res.users` | `action_test_outgoing_mail_server` | self | `mail` |  |
| `res.users` | `action_totp_disable` | self | `auth_totp` |  |
| `res.users` | `action_totp_enable_wizard` | self | `auth_totp` |  |
| `res.users` | `action_totp_invite` | self | `auth_totp_mail` |  |
| `res.users.identitycheck` | `action_use_password` | self | `auth_passkey` |  |
| `sale.loyalty.coupon.wizard` | `action_apply` | self | `sale_loyalty` |  |
| `sale.loyalty.reward.wizard` | `action_apply` | self | `sale_loyalty` |  |
| `sale.loyalty.reward.wizard` | `action_cancel` | self | `sale_loyalty` |  |
| `sale.mass.cancel.orders` | `action_mass_cancel` | self | `sale` |  |
| `sale.order` | `action_cancel` | self | `sale` | Cancel sales order and related draft invoices. |
| `sale.order` | `action_confirm` | self | `delivery_mondialrelay`, `event_booth_sale`, `event_sale`, `l10n_it_edi_doi`, `partnership`, `sale`, `sale_crm`, `sale_gelato`, `sale_loyalty`, `sale_management`, `sale_project`, `website_sale` | Confirm the given quotation(s) and set their confirmation date.  If the corresponding setting is enabled, also locks the Sale Order.  :return: True :rtype: bool :raise: UserError if trying to confirm cancelled SO's |
| `sale.order` | `action_create_project` | self | `sale_project` |  |
| `sale.order` | `action_draft` | self | `sale` |  |
| `sale.order` | `action_lock` | self | `sale` |  |
| `sale.order` | `action_open_business_doc` | self | `sale` |  |
| `sale.order` | `action_open_declaration_of_intent` | self | `l10n_it_edi_doi` |  |
| `sale.order` | `action_open_delivery_wizard` | self | `delivery`, `sale_gelato` | Override of `delivery` to set a Gelato delivery method by default in the wizard. |
| `sale.order` | `action_open_discount_wizard` | self | `sale` |  |
| `sale.order` | `action_open_reward_wizard` | self | `sale_loyalty` |  |
| `sale.order` | `action_preview_sale_order` | self | `sale`, `website_sale` |  |
| `sale.order` | `action_quotation_send` | self | `l10n_it_edi_doi`, `sale` | Opens a wizard to compose an email, with relevant mail template loaded by default |
| `sale.order` | `action_quotation_sent` | self | `l10n_it_edi_doi`, `sale` | Mark the given draft quotation(s) as sent.  :raise: UserError if any given SO is not in draft state. |
| `sale.order` | `action_recovery_email_send` | self | `website_sale` |  |
| `sale.order` | `action_show_repair` | self | `repair` |  |
| `sale.order` | `action_unlock` | self | `sale` |  |
| `sale.order` | `action_update_prices` | self | `sale` |  |
| `sale.order` | `action_update_taxes` | self | `sale` |  |
| `sale.order` | `action_view_attendee_list` | self | `event_sale` |  |
| `sale.order` | `action_view_booth_list` | self | `event_booth_sale` |  |
| `sale.order` | `action_view_delivery` | self | `sale_stock`, `stock_dropshipping` |  |
| `sale.order` | `action_view_dropship` | self | `stock_dropshipping` |  |
| `sale.order` | `action_view_gift_cards` | self | `sale_loyalty` |  |
| `sale.order` | `action_view_invoice` | self, invoices | `sale` |  |
| `sale.order` | `action_view_milestone` | self | `sale_project` |  |
| `sale.order` | `action_view_mrp_production` | self | `sale_mrp` |  |
| `sale.order` | `action_view_pos_order` | self | `pos_sale` |  |
| `sale.order` | `action_view_project_ids` | self | `sale_project` |  |
| `sale.order` | `action_view_purchase_orders` | self | `sale_purchase` |  |
| `sale.order` | `action_view_timesheet` | self | `sale_timesheet` |  |
| `sale.order.discount` | `action_apply_discount` | self | `sale` |  |
| `sale.order.line` | `action_add_from_catalog` | self | `sale` |  |
| `sale.report` | `action_open_order` | self | `sale` |  |
| `slide.channel` | `action_channel_enroll` | self | `website_slides` |  |
| `slide.channel` | `action_channel_invite` | self | `website_slides` |  |
| `slide.channel` | `action_grant_access` | self, partner_id | `website_slides` |  |
| `slide.channel` | `action_mass_mailing_attendees` | self | `mass_mailing_slides` |  |
| `slide.channel` | `action_redirect_to_certified_members` | self | `website_slides_survey` |  |
| `slide.channel` | `action_redirect_to_completed_members` | self | `website_slides` |  |
| `slide.channel` | `action_redirect_to_engaged_members` | self | `website_slides` |  |
| `slide.channel` | `action_redirect_to_forum` | self | `website_slides_forum` |  |
| `slide.channel` | `action_redirect_to_invited_members` | self | `website_slides` |  |
| `slide.channel` | `action_redirect_to_members` | self, status_filter | `website_slides` | Redirects to attendees of the course. If status_filter is set to 'invited' / 'engaged' ('joined' + 'ongoing') / 'completed', attendees are filtered accordingly. |
| `slide.channel` | `action_refuse_access` | self, partner_id | `website_slides` |  |
| `slide.channel` | `action_request_access` | self | `website_slides` | Request access to the channel. Returns a dict with keys being either 'error' (specific error raised) or 'done' (request done or not). |
| `slide.channel` | `action_view_ratings` | self | `website_slides` |  |
| `slide.channel` | `action_view_sales` | self | `website_sale_slides` |  |
| `slide.channel` | `action_view_slides` | self | `website_slides` |  |
| `slide.channel.invite` | `action_invite` | self | `website_slides` | Process the wizard content and proceed with sending the related email(s), rendering any template patterns on the fly if needed. This method is used both to add members as 'joined' (when adding attendees) and as 'invited' (on invitation), depending on the value of enroll_mode. Archived members can be |
| `slide.slide` | `action_dislike` | self | `website_slides` |  |
| `slide.slide` | `action_like` | self | `website_slides` |  |
| `slide.slide` | `action_mark_completed` | self | `website_slides` |  |
| `slide.slide` | `action_mark_uncompleted` | self | `website_slides` |  |
| `slide.slide` | `action_set_viewed` | self, quiz_attempts_inc | `website_slides` |  |
| `slide.slide` | `action_view_embeds` | self | `website_slides` |  |
| `sms.account.code` | `action_register` | self | `sms` |  |
| `sms.account.phone` | `action_send_verification_code` | self | `sms` |  |
| `sms.account.sender` | `action_set_sender_name` | self | `sms` |  |
| `sms.composer` | `action_send_sms` | self | `sms` |  |
| `sms.composer` | `action_send_sms_mass_now` | self | `sms` |  |
| `sms.sms` | `action_set_canceled` | self | `sms` |  |
| `sms.sms` | `action_set_error` | self, failure_type | `sms` |  |
| `sms.sms` | `action_set_outgoing` | self | `sms` |  |
| `sms.template` | `action_create_sidebar_action` | self | `sms` |  |
| `sms.template` | `action_unlink_sidebar_action` | self | `sms` |  |
| `sms.twilio.account.manage` | `action_reload_numbers` | self | `sms_twilio` | Fetch the available numbers from Twilio account |
| `sms.twilio.account.manage` | `action_save` | self | `sms_twilio` |  |
| `sms.twilio.account.manage` | `action_send_test` | self | `sms_twilio` |  |
| `sms.twilio.number` | `action_unlink` | self | `sms_twilio` |  |
| `spreadsheet.dashboard` | `action_toggle_favorite` | self | `spreadsheet_dashboard` |  |
| `spreadsheet.dashboard.share` | `action_get_share_url` | self, vals | `spreadsheet_dashboard` |  |
| `stock.forecasted_product_product` | `action_reserve_linked_picks` | self, move_id | `stock` |  |
| `stock.forecasted_product_product` | `action_unreserve_linked_picks` | self, move_id | `stock` |  |
| `stock.inventory.adjustment.name` | `action_apply` | self | `stock` |  |
| `stock.inventory.conflict` | `action_keep_counted_quantity` | self | `stock` |  |
| `stock.inventory.conflict` | `action_keep_difference` | self | `stock` |  |
| `stock.inventory.warning` | `action_reset` | self | `stock` |  |
| `stock.inventory.warning` | `action_set` | self | `stock` |  |
| `stock.landed.cost` | `button_cancel` | self | `stock_landed_costs` |  |
| `stock.landed.cost` | `button_validate` | self | `stock_landed_costs` |  |
| `stock.location` | `action_view_equipments_records` | self | `stock_maintenance` |  |
| `stock.lot` | `action_lot_open_quants` | self | `stock` |  |
| `stock.lot` | `action_lot_open_repairs` | self | `repair` |  |
| `stock.lot` | `action_lot_open_transfers` | self | `stock` |  |
| `stock.lot` | `action_view_po` | self | `purchase_stock` |  |
| `stock.lot` | `action_view_ro` | self | `repair` |  |
| `stock.lot` | `action_view_so` | self | `sale_stock` |  |
| `stock.move` | `action_add_from_catalog_byproduct` | self | `mrp` |  |
| `stock.move` | `action_add_from_catalog_raw` | self | `mrp` |  |
| `stock.move` | `action_add_from_catalog_repair` | self | `repair` |  |
| `stock.move` | `action_add_packages` | self | `stock` | Opens a list of suitable packages to add to a picking. |
| `stock.move` | `action_adjust_valuation` | self | `stock_account` |  |
| `stock.move` | `action_explode` | self | `mrp` | Explodes pickings |
| `stock.move` | `action_generate_lot_line_vals` | self, context_data, mode, first_lot, count, lot_text | `product_expiry`, `stock` |  |
| `stock.move` | `action_open_reference` | self | `mrp`, `stock` | Open the form view of the move's reference document, if one exists, otherwise open form view of self |
| `stock.move` | `action_product_forecast_report` | self | `stock` |  |
| `stock.move` | `action_show_details` | self | `mrp`, `mrp_subcontracting`, `repair`, `stock`, `stock_picking_batch` | Returns an action that will open a form view (in a popup) allowing to work on all the move lines of a particular move. This form view is used when "show operations" is not checked on the picking type. |
| `stock.move` | `action_show_subcontract_details` | self, lot_id | `mrp_subcontracting` | Display moves raw for subcontracted product self. |
| `stock.move.line` | `action_open_add_to_wave` | self | `stock_picking_batch` |  |
| `stock.move.line` | `action_open_reference` | self | `stock` |  |
| `stock.move.line` | `action_put_in_pack` | self, package_id, package_type_id, package_name | `stock` |  |
| `stock.move.line` | `action_revert_inventory` | self | `stock` |  |
| `stock.orderpoint.snooze` | `action_snooze` | self | `stock` |  |
| `stock.package` | `action_add_to_picking` | self | `stock` |  |
| `stock.package` | `action_put_in_pack` | self, package_id, package_type_id, package_name | `stock` |  |
| `stock.package` | `action_remove_package` | self | `stock` | Removes all packages in self from the destination container tree. For move lines directly linked to a package (through result_package_id) - If the entire package is moved, remove the move lines entirely from the picking - Otherwise, just unset the packages as destination package |
| `stock.package` | `action_view_picking` | self | `stock` |  |
| `stock.package.destination` | `action_done` | self | `stock` |  |
| `stock.package.history` | `action_show_package` | self | `stock` |  |
| `stock.picking` | `action_add_entire_packs` | self, package_ids | `stock` |  |
| `stock.picking` | `action_add_operations` | self | `stock_picking_batch` |  |
| `stock.picking` | `action_assign` | self | `stock` | Check availability of picking moves. This has the effect of changing the state and reserve quants on available moves, and may also impact the state of the picking as it is computed based on move's states. @return: True |
| `stock.picking` | `action_cancel` | self | `stock`, `stock_picking_batch` |  |
| `stock.picking` | `action_confirm` | self | `stock`, `stock_picking_batch` |  |
| `stock.picking` | `action_detailed_operations` | self | `mrp`, `stock` |  |
| `stock.picking` | `action_generate_l10n_tr_edispatch_xml` | self, is_list | `l10n_tr_nilvera_edispatch` |  |
| `stock.picking` | `action_l10n_in_ewaybill_create` | self | `l10n_in_ewaybill_stock` |  |
| `stock.picking` | `action_l10n_ro_edi_stock_fetch_status` | self | `l10n_ro_edi_stock` |  |
| `stock.picking` | `action_l10n_ro_edi_stock_send_etransport` | self | `l10n_ro_edi_stock` |  |
| `stock.picking` | `action_mark_l10n_tr_edispatch_status` | self | `l10n_tr_nilvera_edispatch` |  |
| `stock.picking` | `action_next_transfer` | self | `stock` |  |
| `stock.picking` | `action_open_l10n_in_ewaybill` | self | `l10n_in_ewaybill_stock` |  |
| `stock.picking` | `action_open_label_layout` | self | `stock` |  |
| `stock.picking` | `action_open_label_type` | self | `stock` |  |
| `stock.picking` | `action_picking_move_tree` | self | `stock` |  |
| `stock.picking` | `action_put_in_pack` | self, package_id, package_type_id, package_name | `stock` |  |
| `stock.picking` | `action_repair_return` | self | `repair` |  |
| `stock.picking` | `action_see_move_scrap` | self | `stock` |  |
| `stock.picking` | `action_see_package_histories` | self | `stock` |  |
| `stock.picking` | `action_see_packages` | self | `stock` |  |
| `stock.picking` | `action_see_returns` | self | `stock` |  |
| `stock.picking` | `action_show_subcontract_details` | self | `mrp_subcontracting` |  |
| `stock.picking` | `action_split_transfer` | self | `stock` |  |
| `stock.picking` | `action_toggle_is_locked` | self | `stock` |  |
| `stock.picking` | `action_view_batch` | self | `stock_picking_batch` |  |
| `stock.picking` | `action_view_mrp_production` | self | `mrp` |  |
| `stock.picking` | `action_view_reception_report` | self | `stock` |  |
| `stock.picking` | `action_view_repairs` | self | `repair` |  |
| `stock.picking` | `action_view_subcontracting_source_purchase` | self | `mrp_subcontracting_purchase` |  |
| `stock.picking` | `button_scrap` | self | `stock` |  |
| `stock.picking` | `button_validate` | self | `l10n_ro_edi_stock`, `l10n_tr_nilvera_edispatch`, `sale_project_stock`, `stock`, `stock_delivery`, `stock_picking_batch` |  |
| `stock.picking` | `do_print_picking` | self | `stock` |  |
| `stock.picking` | `do_unreserve` | self | `stock` |  |
| `stock.picking.batch` | `action_assign` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_batch_detailed_operations` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_cancel` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_confirm` | self | `stock_picking_batch` | Sanity checks, confirm the pickings and mark the batch as confirmed. |
| `stock.picking.batch` | `action_done` | self | `l10n_ro_edi_stock_batch`, `stock_picking_batch` |  |
| `stock.picking.batch` | `action_l10n_ro_edi_stock_fetch_status` | self | `l10n_ro_edi_stock_batch` |  |
| `stock.picking.batch` | `action_l10n_ro_edi_stock_send_etransport` | self | `l10n_ro_edi_stock_batch` |  |
| `stock.picking.batch` | `action_merge` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_open_label_layout` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_print` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_put_in_pack` | self, package_id, package_type_id, package_name | `stock_picking_batch` | Action to put move lines with 'Done' quantities into a new pack This method follows same logic to stock.picking. |
| `stock.picking.batch` | `action_see_packages` | self | `stock_picking_batch` |  |
| `stock.picking.batch` | `action_view_reception_report` | self | `stock_picking_batch` |  |
| `stock.picking.type` | `action_batch` | self | `stock_picking_batch` |  |
| `stock.picking.type` | `action_redirect_to_barcode_installation` | self | `stock` |  |
| `stock.picking.type` | `action_wave` | self | `stock_picking_batch` |  |
| `stock.put.in.pack` | `action_put_in_pack` | self | `stock` |  |
| `stock.quant` | `action_apply_all` | self | `stock` |  |
| `stock.quant` | `action_apply_inventory` | self, date | `stock` |  |
| `stock.quant` | `action_clear_inventory_quantity` | self | `stock` |  |
| `stock.quant` | `action_inventory_history` | self | `stock` |  |
| `stock.quant` | `action_reset` | self | `stock` |  |
| `stock.quant` | `action_set_inventory_quantity` | self | `stock` |  |
| `stock.quant` | `action_set_inventory_quantity_zero` | self | `stock` |  |
| `stock.quant` | `action_stock_quant_relocate` | self | `stock` |  |
| `stock.quant` | `action_view_inventory` | self | `stock` | Similar to _get_quants_action except specific for inventory adjustments (i.e. inventory counts). |
| `stock.quant` | `action_view_orderpoints` | self | `stock` |  |
| `stock.quant` | `action_view_quants` | self | `stock` |  |
| `stock.quant` | `action_view_stock_moves` | self | `stock` |  |
| `stock.quant.relocate` | `action_relocate_quants` | self | `stock` |  |
| `stock.request.count` | `action_request_count` | self | `stock` |  |
| `stock.return.picking` | `action_create_exchanges` | self | `stock` | Create a return for the active picking, then create a return of the return for the exchange picking and open it. |
| `stock.return.picking` | `action_create_returns` | self | `stock` |  |
| `stock.return.picking` | `action_create_returns_all` | self | `stock` | Create a return matching the total delivered quantity and open it. |
| `stock.scrap` | `action_get_stock_move_lines` | self | `stock` |  |
| `stock.scrap` | `action_get_stock_picking` | self | `stock` |  |
| `stock.scrap` | `action_validate` | self | `stock` |  |
| `stock.scrap` | `do_replenish` | self, values | `mrp`, `stock` |  |
| `stock.scrap` | `do_scrap` | self | `mrp`, `stock` |  |
| `stock.warehouse` | `action_view_all_routes` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_open_orderpoints` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_product_forecast_report` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_remove_manual_qty_to_order` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_replenish` | self, force_to_max | `stock` |  |
| `stock.warehouse.orderpoint` | `action_replenish_auto` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_stock_replenishment_info` | self | `stock` |  |
| `stock.warehouse.orderpoint` | `action_view_purchase` | self | `purchase_stock` | This function returns an action that display existing purchase orders of given orderpoint. |
| `stock.warn.insufficient.qty` | `action_done` | self | `stock` |  |
| `stock.warn.insufficient.qty.repair` | `action_done` | self | `repair` |  |
| `stock.warn.insufficient.qty.scrap` | `action_cancel` | self | `stock` |  |
| `stock.warn.insufficient.qty.scrap` | `action_done` | self | `stock` |  |
| `stock.warn.insufficient.qty.unbuild` | `action_done` | self | `mrp` |  |
| `stock_account.stock.valuation.report` | `action_print_as_pdf` | self | `stock_account` |  |
| `stock_account.stock.valuation.report` | `action_print_as_xlsx` | self | `stock_account` |  |
| `survey.invite` | `action_invite` | self | `hr_recruitment_survey`, `survey` | Process the wizard content and proceed with sending the related email(s), rendering any template patterns on the fly if needed |
| `survey.survey` | `action_end_session` | self | `survey`, `survey_crm` | The write is sudo'ed because a survey user can end a session even if it's not their own survey. |
| `survey.survey` | `action_load_sample_custom` | self | `survey` |  |
| `survey.survey` | `action_load_survey_template_sample` | self, template_key | `survey` |  |
| `survey.survey` | `action_open_session_manager` | self | `survey` |  |
| `survey.survey` | `action_print_survey` | self, answer | `survey` | Open the website page with the survey printable view |
| `survey.survey` | `action_result_survey` | self | `survey` | Open the website page with the survey results view |
| `survey.survey` | `action_send_survey` | self | `survey` | Open a window to compose an email, pre-filled with the survey message |
| `survey.survey` | `action_show_sample` | self | `survey` |  |
| `survey.survey` | `action_start_session` | self | `survey` | Sets the necessary fields for the session to take place and starts it. The write is sudo'ed because a survey user can start a session even if it's not their own survey. |
| `survey.survey` | `action_start_survey` | self, answer | `survey` | Open the website page with the survey form |
| `survey.survey` | `action_survey_preview_certification_template` | self | `survey` |  |
| `survey.survey` | `action_survey_see_leads` | self | `survey_crm` | Shows the leads created from the current survey |
| `survey.survey` | `action_survey_user_input` | self | `survey` |  |
| `survey.survey` | `action_survey_user_input_certified` | self | `survey` |  |
| `survey.survey` | `action_survey_user_input_completed` | self | `hr_recruitment_survey`, `survey` |  |
| `survey.survey` | `action_survey_view_slide_channels` | self | `website_slides_survey` | Redirect to the channels using the survey as a certification. Open in no-create as link between those two comes through a slide, hard to keep as default values. |
| `survey.survey` | `action_test_survey` | self | `survey` | Open the website page with the survey form into test mode |
| `survey.user_input` | `action_print_answers` | self | `survey` | Open the website page with the survey form |
| `survey.user_input` | `action_redirect_lead` | self | `survey_crm` | Shows the lead associated, created from inputs |
| `survey.user_input` | `action_redirect_to_attempts` | self | `survey` |  |
| `survey.user_input` | `action_resend` | self | `survey` |  |
| `talent.pool.add.applicants` | `action_add_applicants_to_pool` | self | `hr_recruitment` |  |
| `uom.uom` | `action_open_packaging_barcodes` | self | `product` |  |
| `update.product.attribute.value` | `action_confirm` | self | `product` |  |
| `utm.campaign` | `action_create_mass_sms` | self | `mass_mailing_sms` |  |
| `utm.campaign` | `action_redirect_to_invoiced` | self | `sale` |  |
| `utm.campaign` | `action_redirect_to_leads_opportunities` | self | `crm` |  |
| `utm.campaign` | `action_redirect_to_mailing_sms` | self | `mass_mailing_sms` |  |
| `utm.campaign` | `action_redirect_to_quotations` | self | `sale` |  |
| `website` | `action_dashboard_redirect` | self | `website`, `website_sale` |  |
| `website` | `button_go_website` | self, path | `website` |  |
| `website.custom_blocked_third_party_domains` | `action_save` | self | `website` |  |
| `website.page` | `action_page_debug_view` | self | `website` |  |
| `website.robots` | `action_save` | self | `website` |  |
| `website.visitor` | `action_send_chat_request` | self | `website_livechat` | Send a chat request to website_visitor(s). This creates a chat_request and a discuss_channel with livechat active flag. But for the visitor to get the chat request, the operator still has to speak to the visitor. The visitor will receive the chat request the next time he navigates to a website page. |
| `website.visitor` | `action_send_mail` | self | `website` |  |
| `website.visitor` | `action_send_sms` | self | `website_sms` |  |

## Operations that run without a person

| Entity | Operation | Packages | Purpose |
|---|---|---|---|
| `account.edi.document` | `_cron_process_documents_web_services` | `account_edi` | Method called by the EDI cron processing all web-services.  :param job_count: Limit explicitely the number of web service calls. If not provided, process all. |
| `account.edi.document` | `_process_documents_no_web_services` | `account_edi` | Post and cancel all the documents that don't need a web service. |
| `account.edi.document` | `_process_documents_web_services` | `account_edi` | Post and cancel all the documents that need a web service.  :param job_count:   The maximum number of jobs to process if specified. :param with_commit: Flag indicating a commit should be made between each job. :return:            The number of remaining jobs to process. |
| `account.edi.document` | `_process_job` | `account_edi` | Post or cancel move_id by calling the related methods on edi_format_id.  :param job:  {     'documents': account.edi.document,     'method_to_call': str, } |
| `account.journal` | `_process_reference_for_sale_order` | `account`, `l10n_ch` | returns the order reference to be used for the payment. Hook to be overriden: see l10n_ch for an example. |
| `account.move` | `_cron_account_move_send` | `account` | Process invoices generation and sending asynchronously. :param job_count: maximum number of jobs to process if specified. |
| `account.move` | `_cron_l10n_pl_edi_check_invoice_status` | `l10n_pl_edi` | get all moves that are in state sent run action_update_invoice_status on all of them |
| `account.move` | `_cron_l10n_pl_edi_download_bills` | `l10n_pl_edi` |  |
| `account.move` | `_cron_nilvera_get_invoice_status` | `l10n_tr_nilvera_einvoice` |  |
| `account.move` | `_cron_nilvera_get_new_earchive_sale_documents` | `l10n_tr_nilvera_einvoice` |  |
| `account.move` | `_cron_nilvera_get_new_einvoice_purchase_documents` | `l10n_tr_nilvera_einvoice` |  |
| `account.move` | `_cron_nilvera_get_new_einvoice_sale_documents` | `l10n_tr_nilvera_einvoice` |  |
| `account.move` | `_cron_nilvera_get_sale_pdf` | `l10n_tr_nilvera_einvoice` | Fetches the Nilvera generated PDFs for the sales generated on the system. |
| `account.move` | `_process_attachments_for_template_post` | `account_edi` | Add Edi attachments to templates. |
| `account_edi_proxy_client.user` | `_cron_nemhandel_get_message_status` | `l10n_dk_nemhandel` |  |
| `account_edi_proxy_client.user` | `_cron_nemhandel_get_new_documents` | `l10n_dk_nemhandel` |  |
| `account_edi_proxy_client.user` | `_cron_nemhandel_get_participant_status` | `l10n_dk_nemhandel` |  |
| `account_edi_proxy_client.user` | `_cron_nemhandel_webhook_keepalive` | `l10n_dk_nemhandel` |  |
| `account_edi_proxy_client.user` | `_cron_pdp_get_regulatory_documents` | `l10n_fr_pdp` |  |
| `account_edi_proxy_client.user` | `_cron_pdp_send_lifecycles` | `l10n_fr_pdp` |  |
| `account_edi_proxy_client.user` | `_cron_peppol_auto_register_services` | `account_peppol_response` |  |
| `account_edi_proxy_client.user` | `_cron_peppol_get_message_status` | `account_peppol` |  |
| `account_edi_proxy_client.user` | `_cron_peppol_get_new_documents` | `account_peppol` |  |
| `account_edi_proxy_client.user` | `_cron_peppol_get_participant_status` | `account_peppol` |  |
| `account_edi_proxy_client.user` | `_cron_peppol_webhook_keepalive` | `account_peppol` |  |
| `base.automation` | `_cron_process_time_based_actions` | `base_automation` | Execute the time-based automations. |
| `base.automation` | `_process` | `base_automation` | Process automation ``self`` on the ``records`` that have not been done yet. |
| `base.partner.merge.automatic.wizard` | `_process_query` | `base` | Execute the select request and write the result in this wizard :param query : the SQL query used to fill the wizard line |
| `bus.bus` | `_gc_messages` | `bus` |  |
| `card.card` | `_gc_card` | `marketing_card` | Remove cards. Social networks are expected to cache the images on their side. |
| `chatbot.script.step` | `_process_answer` | `im_livechat` | Method called when the user reacts to the current chatbot.script step. For most chatbot.script.step#step_types it simply returns the next chatbot.script.step of the script (see '_fetch_next_step').  Some extra processing is done for steps of type 'question_email' and 'question_phone' where we store  |
| `chatbot.script.step` | `_process_step` | `crm_livechat`, `im_livechat` | When we reach a chatbot.step in the script we need to do some processing on behalf of the bot. Which is for most chatbot.script.step#step_types just posting the message field.  Some extra processing may be required for special step types such as 'forward_operator', 'create_lead', 'create_ticket' (in |
| `chatbot.script.step` | `_process_step_create_lead` | `crm_livechat` | When reaching a 'create_lead' step, we extract the relevant information: visitor's email, phone and conversation history to create a crm.lead.  We use the email and phone to update the environment partner's information (if not a public user) if they differ from the current values.  The whole convers |
| `chatbot.script.step` | `_process_step_create_lead_and_forward` | `crm_livechat` |  |
| `crm.lead` | `_cron_update_automated_probabilities` | `crm` | This cron will : - rebuild the lead scoring frequency table - recompute all the automated_probability and align probability if both were aligned |
| `crm.reveal.rule` | `_process_lead_generation` | `website_crm_iap_reveal` | Cron Job for lead generation from page view |
| `crm.team` | `_cron_assign_leads` | `crm` | Cron method assigning leads. Leads are allocated to all teams and assigned to their members.  The cron is designed to run at least once a day or more. A number of leads will be assigned each time depending on the daily leads already assigned. This allows the assignment process based on the cron to w |
| `data_recycle.model` | `_cron_recycle_records` | `data_recycle` |  |
| `digest.digest` | `_cron_send_digest_email` | `digest` |  |
| `discuss.channel` | `_gc_bot_only_ongoing_sessions` | `im_livechat` | Garbage collect bot-only livechat sessions with no activity for over 1 day. |
| `discuss.channel` | `_gc_empty_livechat_sessions` | `im_livechat` |  |
| `discuss.channel.member` | `_gc_unpin_livechat_sessions` | `im_livechat` | Unpin read livechat sessions with no activity for at least one day to clean the operator's interface |
| `discuss.channel.member` | `_gc_unpin_outdated_sub_channels` | `mail` |  |
| `discuss.channel.rtc.session` | `_gc_inactive_sessions` | `mail` | Garbage collect sessions that aren't active anymore, this can happen when the server or the user's browser crash or when the user's the system session ends. |
| `event.event` | `_gc_mark_events_done` | `event` | move every ended events in the next 'ended stage' |
| `event.lead.request` | `_cron_generate_leads` | `event_crm` | See class docstring for details.  :param job_limit: The maximum amount of 'event.lead.request' to process   Defaults to 100. :param registrations_batch_size: The amount of attendees processed at once.   Defaults to event.lead.request._REGISTRATIONS_BATCH_SIZE |
| `event.lead.rule` | `_run_on_registrations` | `event_crm` | Create or update leads based on rule configuration. Two main lead management type exists    * per attendee: each registration creates a lead;   * per order: registrations are grouped per group and one lead is created     or updated with the batch (used mainly with sale order configuration     in eve |
| `gamification.challenge` | `_cron_update` | `gamification` | Daily cron check.  - Start planned challenges (in draft and with start_date = today) - Create the missing goals (eg: modified the challenge to add lines) - Update every running challenge |
| `gamification.karma.tracking` | `_process_consolidate` | `gamification` | Consolidate the karma trackings.  The consolidation keeps, for each user, the oldest "old_value" and the most recent "new_value", creates a new karma tracking with those values and removes all karma trackings between those dates. The origin / reason is changed on the consolidated records, so this in |
| `hr.attendance` | `_cron_absence_detection` | `hr_attendance` | Objective is to create technical attendances on absence days to have negative overtime created for that day |
| `hr.attendance` | `_cron_auto_check_out` | `hr_attendance` |  |
| `hr.employee` | `_cron_update_current_version_id` | `hr` |  |
| `hr.expense` | `_cron_send_submitted_expenses_mail` | `hr_expense` |  |
| `hr.leave.allocation` | `_process_accrual_plan_level` | `hr_holidays` | Returns the added days for that level |
| `hr.leave.allocation` | `_process_accrual_plans` | `hr_holidays` | This method is part of the cron's process. The goal of this method is to retroactively apply accrual plan levels and progress from nextcall to date_to or today. If force_period is set, the accrual will run until date_to in a prorated way (used for end of year accrual actions). |
| `hr.version` | `_cron_generate_missing_work_entries` | `hr_work_entry` |  |
| `ir.actions.report` | `_run_wkhtmltoimage` | `base` | :param str bodies: valid html documents as strings :param int width: width in pixels :param int height: height in pixels :param image_format: format of the image :type image_format: typing.Literal['jpg', 'png'] |
| `ir.actions.report` | `_run_wkhtmltopdf` | `base` | Execute wkhtmltopdf as a subprocess in order to convert html given in input into a pdf document.  :param Iterable[str] bodies: The html bodies of the report, one per page. :param report_ref: report reference that is needed to get report paperformat. :param str header: The html header of the report c |
| `ir.actions.server` | `_run_action_code_multi` | `base`, `website` | Override to allow returning response the same way action is already returned by the basic server action behavior. Note that response has priority over action, avoid using both. |
| `ir.actions.server` | `_run_action_followers_multi` | `mail` |  |
| `ir.actions.server` | `_run_action_mail_post_multi` | `mail` |  |
| `ir.actions.server` | `_run_action_multi` | `base` |  |
| `ir.actions.server` | `_run_action_next_activity` | `mail` |  |
| `ir.actions.server` | `_run_action_object_copy` | `base` | Duplicate specified model object. If applicable, link active_id.<self.link_field_id> to the new record. |
| `ir.actions.server` | `_run_action_object_create` | `base` | Create specified model object with specified name contained in value.  If applicable, link active_id.<self.link_field_id> to the new record. |
| `ir.actions.server` | `_run_action_object_write` | `base` | Apply specified write changes to active_id. |
| `ir.actions.server` | `_run_action_remove_followers_multi` | `mail` |  |
| `ir.actions.server` | `_run_action_sms_multi` | `sms` |  |
| `ir.actions.server` | `_run_action_webhook` | `base` | Send a post request with a read of the selected field on active_id. |
| `ir.actions.server.history` | `_gc_histories` | `base` |  |
| `ir.asset` | `_process_command` | `base` | Parses a given command to return its directive, target and path definition. |
| `ir.asset` | `_process_path` | `base` | This sub function is meant to take a directive and a set of arguments and apply them to the current asset_paths list accordingly.  It is nested inside `_get_asset_paths` since we need the current list of addons, extensions and asset_paths.  :param directive: string :param target: string or None or F |
| `ir.attachment` | `_cron_migrate_local_to_cloud_storage` | `cloud_storage_migration` | The Http server only reschedules the cron job asap without migrating any attachment. The cron server will continue the migrating process stopped at the last time by using ``cloud_storage_migration_min_attachment_id`` |
| `ir.attachment` | `_gc_doc_index` | `api_doc` | Garbage collect the outdated /doc/index.json attachments. |
| `ir.attachment` | `_gc_file_store` | `base` | Perform the garbage collection of the filestore. |
| `ir.attachment` | `_gc_file_store_unsafe` | `base` |  |
| `ir.autovacuum` | `_gc_orm_signaling` | `base` |  |
| `ir.autovacuum` | `_run_vacuum_cleaner` | `base` | Perform a complete database cleanup by safely calling every ``@api.autovacuum`` decorated method. |
| `ir.cron` | `_process_job` | `base` | Execute the cron's server action in a dedicated transaction.  In case the previous process actually timed out, the cron's server action is not executed and the cron is considered ``'failed'``.  The server action can use the progress API via the method :meth:`_commit_progress` to report how many reco |
| `ir.cron` | `_process_jobs` | `base` | Execute every job ready to be run on this database. |
| `ir.cron` | `_process_jobs_loop` | `base` | Process ready jobs to run on this database.  The `cron_cr` is used to lock the currently processed job and relased by committing after each job. |
| `ir.cron` | `_run_job` | `base` | Execute the job's server action multiple times until it completes. The completion status is returned.  It is considered completed when either:  - the server action doesn't use the progress API, or returned   and notified that all records has been processed: ``'fully done'``;  - the server action ret |
| `ir.cron.progress` | `_gc_cron_progress` | `base` |  |
| `ir.cron.trigger` | `_gc_cron_triggers` | `base` |  |
| `ir.http` | `_gc_sessions` | `base` |  |
| `ir.model.data` | `_process_end` | `base` | Clear records removed from updated module data. This method is called at the end of the module loading process. It is meant to removed records that are no longer present in the updated data. Such records are recognised as the one with an xml id and a module in ir_model_data and noupdate set to false |
| `ir.model.data` | `_process_end_unlink_record` | `base`, `website` |  |
| `ir.model.fields.selection` | `_process_ondelete` | `base` | Process the 'ondelete' of the given selection values. |
| `ir.profile` | `_gc_profile` | `base` |  |
| `l10n.fr.pdp.reports.flow` | `_cron_process_company` | `l10n_fr_pdp` |  |
| `l10n.fr.pdp.reports.flow` | `_cron_update_and_send_flows` | `l10n_fr_pdp` |  |
| `l10n_es_edi_tbai.document` | `_process_post_response_xml_ar_gi` | `l10n_es_edi_tbai` | Government response processing for Araba and Gipuzkoa. |
| `l10n_es_edi_tbai.document` | `_process_post_response_xml_bi` | `l10n_es_edi_tbai` | Government response processing for Bizkaia. |
| `l10n_id.qris.transaction` | `_gc_remove_pointless_qris_transactions` | `l10n_id` | Removes unpaid transactions that have been for more than 35 minutes. These can no longer be paid and status will no longer change |
| `l10n_pl.bank.account.verification` | `_gc_bank_account_verification` | `l10n_pl_bank_verification` |  |
| `mail.activity` | `_gc_delete_old_overdue_activities` | `mail` | Delete old overdue activities - If the config_parameter is deleted or 0, the user doesn't want to run this gc routine - If the config_parameter is set to a negative number, it's an invalid value, we skip the gc routine - If the config_parameter is set to a positive number, we delete only overdue act |
| `mail.compose.message` | `_gc_lost_attachments` | `mail` | Garbage collect lost mail attachments. Those are attachments - linked to res_model 'mail.compose.message', the composer wizard - with res_id 0, because they were created outside of an existing     wizard (typically user input through Chatter or reports     created on-the-fly by the templates) - unus |
| `mail.compose.message` | `_process_generic_card_url_body` | `marketing_card` | Update the bodies with the specific card url for that res_id and create a card.  example: (1, "/cards/9/preview") -> (1, "/cards/9/1/abchashtoken/preview") + new card as side-effect  :return: processed bodies in the order they were received |
| `mail.compose.message` | `_process_mail_values_state` | `mail` | When being in mass mailing, avoid sending emails to void or invalid emails. For that purpose a processing of generated values allows to give a state and a failure type to mail.mail records that will be created at sending time.  :param dict mail_values_dict: as generated by '_prepare_mail_values';  : |
| `mail.group` | `_cron_notify_moderators` | `mail_group` |  |
| `mail.mail` | `_gc_canceled_mail_mail` | `mass_mailing` | Garbage collects old canceled mail.mail records as we consider nobody is going to look at them anymore, becoming noise. |
| `mail.message.translation` | `_gc_translations` | `mail` |  |
| `mail.notification` | `_gc_notifications` | `mail` |  |
| `mail.presence` | `_gc_bus_presence` | `mail` |  |
| `mail.render.mixin` | `_process_scheduled_date` | `mail` |  |
| `mail.thread` | `_process_attachments_for_post` | `mail` | Preprocess attachments for MailTread.message_post() or MailMail.create(). Purpose is to    * transfer attachments given by ``attachment_ids`` from the composer     to the record (if any);   * limit attachments manipulation when being a shared user: only those     created by the user and linked to th |
| `mail.thread` | `_process_attachments_for_template_post` | `mail` | Model specific management of attachments used with template attachments generation in addition to reports. Only usage currently is for EDI in accounting.  :param mail.template mail_template: a mail.template record used to generate   message or emails on self;  :return: a dictionary based on self.ids |
| `mailing.mailing` | `_process_mass_mailing_queue` | `mass_mailing` |  |
| `payment.transaction` | `_cron_post_process` | `payment` | Trigger the post-processing of the transactions that were not handled by the client in the `poll_status` controller method.  :return: None |
| `payment.transaction` | `_cron_send_invoice` | `sale` | Cron to send invoice that where not ready to be send directly after posting |
| `payment.transaction` | `_process` | `payment`, `pos_online_payment_self_order` | Process the payment data received from the provider and update the transaction.  :param str provider_code: The code of the provider handling the transaction. :param dict payment_data: The payment data sent by the provider. :return: The updated transaction. :rtype: payment.transaction |
| `payment.transaction` | `_process_pos_online_payment` | `pos_online_payment`, `pos_online_payment_self_order` |  |
| `pos.order` | `_process_existing_gift_cards` | `pos_loyalty` |  |
| `pos.order` | `_process_order` | `l10n_my_edi_pos`, `point_of_sale`, `pos_event`, `pos_online_payment` | Create or update an pos.order from a given dictionary.  :param dict order: dictionary representing the order. :param existing_order: order to be updated or False. :type existing_order: pos.order. :returns: id of created/updated pos.order :rtype: int |
| `pos.order` | `_process_payment_lines` | `point_of_sale` | Create account.bank.statement.lines from the dictionary given to the parent function.  If the payment_line is an updated version of an existing one, the existing payment_line will first be removed before making a new one. :param pos_order: dictionary representing the order. :type pos_order: dict. :p |
| `pos.order` | `_process_saved_order` | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `point_of_sale` |  |
| `product.product` | `_run_avco` | `stock_account` |  |
| `product.product` | `_run_average_batch` | `stock_account` |  |
| `product.product` | `_run_fifo` | `stock_account` | Returns the value for the next outgoing product base on the qty give as argument. |
| `product.product` | `_run_fifo_batch` | `stock_account` |  |
| `product.product` | `_run_fifo_get_stack` | `stock_account` |  |
| `product.product` | `_run_standard_batch` | `stock_account` |  |
| `product.template` | `_process_pos_self_ui_products` | `pos_self_order` |  |
| `product.template` | `_process_pos_ui_product_product` | `point_of_sale` |  |
| `product.wishlist` | `_gc_sessions` | `website_sale_wishlist` | Remove wishlists for unexisting sessions. |
| `res.company` | `_cron_l10n_gr_edi_fetch_invoices` | `l10n_gr_edi` | Receive issued myDATA Invoices and create draft Vendor Bills based on the received XML. |
| `res.company` | `_cron_l10n_pl_edi_refresh_tokens` | `l10n_pl_edi` | Automatically performs a full KSeF authentication to renew both the access token and the refresh token for active companies. |
| `res.company` | `_cron_l10n_ro_edi_refresh_access_token` | `l10n_ro_edi` | This CRON method will be run every 30 days to refresh the following fields on the company:   - ``l10n_ro_edi_access_token``  - ``l10n_ro_edi_refresh_token``  - ``l10n_ro_edi_access_expiry_date``  - ``l10n_ro_edi_refresh_expiry_date`` |
| `res.company` | `_cron_l10n_ro_edi_synchronize_invoices` | `l10n_ro_edi` | This CRON method will be run every 24 hours to synchronize the invoices and the bills with the ANAF |
| `res.company` | `_cron_mer_archive_signed_xmls` | `l10n_hr_edi` |  |
| `res.company` | `_cron_mer_get_new_documents` | `l10n_hr_edi` |  |
| `res.company` | `_cron_mer_update_document_status` | `l10n_hr_edi` |  |
| `res.company` | `_cron_post_stock_valuation` | `stock_account` |  |
| `res.device.log` | `_gc_device_log` | `base` |  |
| `res.partner` | `_cron_check_vies_iap` | `base_vat` | Called by cron to check if IAP has any update on a previously requested VAT that was pending |
| `res.partner` | `_process_enriched_response` | `partner_autocomplete` |  |
| `res.partner` | `_run_check_identification` | `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_latam_base`, `l10n_uy` | Since we validate more documents than the vat for Argentinean partners (CUIT - VAT AR, CUIL, DNI) we extend this method in order to process it. |
| `res.partner` | `_run_vat_checks` | `account`, `base_vat` | Checks a VAT number syntactically to ensure its validity upon saving.  :param country: a country to check for :param vat: a string with the VAT number to check. :param partner_name: to put into the error message :param validation: if False, it will only return the formatted vat without checking if i |
| `res.partner` | `_run_vies_test` | `l10n_hu_edi` | Convert back the hungarian format to EU format: 12345678-1-12 => HU12345678 |
| `res.users` | `_gc_personal_mail_servers` | `mail` | In case the user change their email, we need to delete the old personal servers. |
| `res.users` | `_process_profile_validation_token` | `website_profile` |  |
| `res.users.apikeys` | `_gc_user_apikeys` | `base` |  |
| `res.users.deletion` | `_gc_portal_users` | `base` | Remove the portal users that asked to deactivate their account.  (see <res.users>::_deactivate_portal_user)  Removing a user can be an heavy operation on large database (because of create_uid, write_uid on each models, which are not always indexed). Because of that, this operation is done in a CRON. |
| `res.users.log` | `_gc_user_logs` | `base` |  |
| `sale.order` | `_cron_send_pending_emails` | `sale` | Find and send pending order status emails asynchronously.  :return: None |
| `sale.order` | `_gc_abandoned_coupons` | `website_sale_loyalty` | Remove coupons from abandonned ecommerce order. |
| `sale.pdf.form.field` | `_cron_post_upgrade_assign_missing_form_fields` | `sale_pdf_quote_builder` |  |
| `slide.channel.partner` | `_gc_slide_channel_partner` | `website_slides` | The invitations of 'invited' attendees are only valid for 3 months. Remove outdated invitations with no completion. A missing last_invitation_date is also considered as expired. |
| `sms.sms` | `_gc_device` | `sms` |  |
| `sms.sms` | `_process_queue` | `sms` | CRON job to send queued SMS messages. |
| `stock.move` | `_run_procurement` | `mrp` |  |
| `stock.quant` | `_run_least_packages_removal_strategy_astar` | `stock` |  |
| `stock.return.picking.line` | `_process_line` | `stock` |  |
| `stock.rule` | `_run_buy` | `purchase_stock` |  |
| `stock.rule` | `_run_manufacture` | `mrp` |  |
| `stock.rule` | `_run_pull` | `stock` |  |
| `stock.rule` | `_run_push` | `stock` | Apply a push rule on a move. If the rule is 'no step added' it will modify the destination location on the move. If the rule is 'manual operation' it will generate a new move in order to complete the section define by the rule. Care this function is not call by method run. It is called explicitely i |
| `stock.rule` | `_run_scheduler_tasks` | `point_of_sale`, `product_expiry`, `stock` |  |
| `utm.campaign` | `_cron_process_mass_mailing_ab_testing` | `mass_mailing`, `mass_mailing_sms` | Cron that manages A/B testing and sends a winner mailing computed based on the value set on the A/B testing campaign. In case there is no mailing sent for an A/B testing campaign we ignore this campaign |
| `website.configurator.feature` | `_process_svg` | `website` |  |
| `website.html.text.processor` | `_process_snippet` | `website` | Process a snippet and its translation :param snippet: Snippet HTML string :param snippet_en: English snippet HTML string :type snippet_en: str :return: (updated_processor, placeholders) :rtype: tuple |
| `website.visitor` | `_cron_unlink_old_visitors` | `website` | Unlink inactive visitors (see '_inactive_visitors_domain' for details).  Visitors were previously archived but we came to the conclusion that archived visitors have very little value and bloat the database for no reason. |
