# Account Move Send (`account.move.send`)

**Transport name:** `account.move.send`  
**Storage name:** `account_move_send`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account`  
**Extended by packages:** `account_edi`, `account_edi_ubl_cii`, `account_peppol`, `l10n_ch`, `l10n_dk_nemhandel`, `l10n_es_edi_facturae`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_fr_pdp`, `l10n_gr_edi`, `l10n_gr_edi_e_invoo`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_ke_edi_tremol`, `l10n_my_edi`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `snailmail_account`

Description: Account Move Send

## Operations (88)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_sending_methods` | preparation rule | self, move | `account_peppol`, `account`, `l10n_fr_pdp` | model | By default, we use the sending method set on the partner or email. |
| `_get_all_extra_edis` | preparation rule | self | `account`, `l10n_es_edi_facturae`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | model | Returns a dict representing EDI data such as: { 'edi_key': {'label': 'EDI label', 'is_applicable': function, 'help': 'optional help'} } |
| `_get_default_extra_edis` | preparation rule | self, move | `account` | model | By default, we use all applicable extra EDIs. |
| `_get_default_invoice_edi_format` | preparation rule | self, move, **kwargs | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_hr_edi` | model | By default, we generate the EDI format set on partner. |
| `_get_default_pdf_report_id` | preparation rule | self, move | `account` | model |  |
| `_get_default_mail_template_id` | preparation rule | self, move | `account` | model |  |
| `_get_default_sending_settings` | preparation rule | self, move, from_cron, **custom_settings | `account` | model | Returns a dict with all the necessary data to generate and send invoices. Either takes the provided custom_settings, or the default value. |
| `_get_alerts` | preparation rule | self, moves, moves_data | `account_edi_ubl_cii`, `account_peppol`, `account`, `l10n_ch`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_ke_edi_tremol`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice`, `snailmail_account` | model | Returns a dict of all alerts corresponding to moves with the given context (sending method, edi format to generate, extra_edi to generate). An alert can have some information: - level (danger, info, warning, ...)  (! danger alerts are considered blocking and will be raised) - message to display - action_text for the text to show on the clickable link - action the action to run when the link is clicked |
| `_get_mail_default_field_value_from_template` | preparation rule | self, mail_template, lang, move, field, **kwargs | `account` | model |  |
| `_get_default_mail_lang` | preparation rule | self, move, mail_template | `account` | model |  |
| `_get_default_mail_body` | preparation rule | self, move, mail_template, mail_lang | `account` | model |  |
| `_get_default_mail_subject` | preparation rule | self, move, mail_template, mail_lang | `account` | model |  |
| `_get_default_mail_partner_ids` | preparation rule | self, move, mail_template, mail_lang | `account` | model |  |
| `_get_default_mail_attachments_widget` | preparation rule | self, move, mail_template, invoice_edi_format, extra_edis, pdf_report | `account` | model |  |
| `_get_placeholder_mail_attachments_data` | preparation rule | self, move, invoice_edi_format, extra_edis, pdf_report | `account_edi_ubl_cii`, `account`, `l10n_es_edi_facturae`, `l10n_es_edi_tbai`, `l10n_jo_edi`, `l10n_vn_edi_viettel` | model | Returns all the placeholder data. Should be extended to add placeholder based on the sending method. :param: move:       The current move. :returns: A list of dictionary for each placeholder. * id:               str: The (fake) id of the attachment, this is needed in rendering in t-key. * name:             str: The name of the attachment. * mimetype:         str: The mimetype of the attachment. * placeholder       bool: Should be true to prevent download / deletion. |
| `_get_placeholder_mail_template_dynamic_attachments_data` | preparation rule | self, move, mail_template, pdf_report | `account` | model | This method returns the placeholder data for the dynamic attachments. :param move:            The current move we are generating documents for. :param mail_template:   The mail template used to get dynamic attachments for the move. :param pdf_report:      The 'ir.actions.report' used for the move.                         Usually it will be the generic 'account.account_invoices' but the user can customize it                         from the Send Wizard interface. :return:                A list of dictionary, one for each placeholder. |
| `_get_invoice_extra_attachments` | preparation rule | self, move | `account_edi_ubl_cii`, `account_edi`, `account`, `l10n_es_edi_facturae`, `l10n_es_edi_tbai`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_my_edi`, `l10n_rs_edi`, `l10n_tr_nilvera_einvoice`, `l10n_vn_edi_viettel` | model | Sharing the XML file may be a requirement, as it doesn't hurt we will do so. |
| `_get_invoice_extra_attachments_data` | preparation rule | self, move | `account` | model |  |
| `_get_mail_template_attachments_data` | preparation rule | self, mail_template | `account` | model | Returns all mail template data. |
| `_raise_danger_alerts` | internal rule | self, alerts | `account` | model |  |
| `_check_move_constraints` | validation | self, moves | `account`, `l10n_hr_edi` | model |  |
| `_get_move_constraints` | preparation rule | self, move | `account_edi_ubl_cii`, `account` | model |  |
| `_check_invoice_report` | validation | self, moves, **custom_settings | `account` | model |  |
| `_format_error_text` | internal rule | self, error | `account` | model | Format the error that can be a dict (complex format needed)  :param error: the error to format. :return: a text formatted error. |
| `_format_error_html` | internal rule | self, error | `account` | model | Format the error that can be a dict (complex format needed)  :param error: the error to format. :return: a html formatted error. |
| `_display_attachments_widget` | internal rule | self, edi_format, sending_methods | `account_edi_ubl_cii`, `account` | model |  |
| `_is_applicable_to_company` | internal rule | self, method, company | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_hr_edi` | model | TO OVERRIDE - used to determine if we should display the sending method in the selection. |
| `_is_applicable_to_move` | internal rule | self, method, move, **move_data | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_hr_edi`, `snailmail_account` | model | TO OVERRIDE - |
| `_hook_invoice_document_before_pdf_report_render` | internal rule | self, invoice, invoice_data | `account_edi_ubl_cii`, `account`, `l10n_ch`, `l10n_es_edi_facturae`, `l10n_it_edi`, `l10n_ke_edi_tremol`, `l10n_ro_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | model | Hook allowing to add some extra data for the invoice passed as parameter before the rendering of the pdf report. :param invoice:         An account.move record. :param invoice_data:    The collected data for the invoice so far. |
| `_prepare_invoice_pdf_report` | preparation rule | self, invoices_data | `account` | model | Prepare the pdf report for the invoice passed as parameter. :param invoice:         An account.move record. :param invoice_data:    The collected data for the invoice so far. |
| `_prepare_invoice_proforma_pdf_report` | preparation rule | self, invoice, invoice_data | `account` | model | Prepare the proforma pdf report for the invoice passed as parameter. :param invoice:         An account.move record. :param invoice_data:    The collected data for the invoice so far. |
| `_hook_invoice_document_after_pdf_report_render` | internal rule | self, invoice, invoice_data | `account_edi_ubl_cii`, `account`, `l10n_it_edi` | model | Hook allowing to add some extra data for the invoice passed as parameter after the rendering of the (proforma) pdf report. :param invoice:         An account.move record. :param invoice_data:    The collected data for the invoice so far. |
| `_link_invoice_documents` | internal rule | self, invoices_data | `account_edi_ubl_cii`, `account`, `l10n_es_edi_facturae`, `l10n_it_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tw_edi_ecpay` | model | Create the attachments containing the pdf/electronic documents for the invoice passed as parameter. :param invoice:         An account.move record. :param invoice_data:    The collected data for the invoice so far. |
| `_hook_if_errors` | internal rule | self, moves_data, allow_raising | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_es_edi_verifactu`, `l10n_hr_edi` | model | Process errors found so far when generating the documents. |
| `_hook_if_success` | internal rule | self, moves_data, from_cron | `account`, `snailmail_account` | model | Process (typically send) successful documents. |
| `_send_notifications_to_partners` | internal rule | self, moves_grouped_by_author_partner_id, is_success | `account` | model |  |
| `_send_mail` | internal rule | self, move, mail_template, **kwargs | `account` | model | Send the journal entry passed as parameter by mail. |
| `_get_mail_layout` | preparation rule | self | `account_peppol`, `account` | model |  |
| `_get_mail_params` | preparation rule | self, move, move_data | `account` | model |  |
| `_generate_dynamic_reports` | internal rule | self, moves_data | `account` | model |  |
| `_send_mails` | internal rule | self, moves_data | `account` | model |  |
| `_can_commit` | internal rule | self | `account` | model | Helper to know if we can commit the current transaction or not. :return: True if commit is accepted, False otherwise. |
| `_call_web_service_before_invoice_pdf_render` | internal rule | self, invoices_data | `account`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_jo_edi`, `l10n_pl_edi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | model |  |
| `_call_web_service_after_invoice_pdf_render` | internal rule | self, invoices_data | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_gr_edi_e_invoo`, `l10n_hr_edi`, `l10n_it_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | model |  |
| `_generate_invoice_documents` | internal rule | self, invoices_data, allow_fallback_pdf | `account` | model | Generate the invoice PDF and electronic documents. :param invoices_data:   The collected data for invoices so far. :param allow_fallback_pdf:  In case of error when generating the documents for invoices, generate a                             proforma PDF report instead. |
| `_generate_invoice_fallback_documents` | internal rule | self, invoices_data | `account` | model | Generate the invoice PDF and electronic documents. :param invoices_data:   The collected data for invoices so far. |
| `_check_sending_data` | validation | self, moves, **custom_settings | `account` |  | Assert the data provided to _generate_and_send_invoices are correct. This is a security in case the method is called directly without going through the wizards. |
| `_generate_and_send_invoices` | internal rule | self, moves, from_cron, allow_raising, allow_fallback_pdf, **custom_settings | `account_peppol`, `account`, `l10n_hr_edi` | model | Generate and send the moves given custom_settings if provided, else their default configuration set on related partner/company. :param moves: account.move to process :param from_cron: whether the processing comes from a cron. :param allow_raising: whether the process can raise errors, or should log them on the move's chatter. :param allow_fallback_pdf:  In case of error when generating the documents for invoices, generate a proforma PDF report instead. :param custom_settings: settings to apply instead of related partner's defaults settings. |
| `_get_mail_attachment_from_doc` | preparation rule | self, doc | `account_edi`, `l10n_es_edi_sii` | model |  |
| `_get_ubl_available_attachments` | preparation rule | self, mail_attachments_widget, invoice_edi_format | `account_edi_ubl_cii` | model |  |
| `_needs_ubl_postprocessing` | internal rule | self, invoice_data | `account_edi_ubl_cii` | model |  |
| `_postprocess_invoice_ubl_xml` | internal rule | self, invoice, invoice_data | `account_edi_ubl_cii`, `l10n_tr_nilvera_einvoice` | model | Include the PDF in the UBL as an AdditionalDocumentReference element.  According to UBL 2.1 standard, the AdditionalDocumentReference element should be placed above ProjectReference which isn't usually in xml files. So usually it's set above AccountingSupplierParty. Here, we try to find a suitable anchor point among the available element to insert our PDF attachment. If none of these are found, we skip adding the attachment to avoid breaking the XML structure. Inside CreditNote, the ProjectReference element is not used in xml. So we look for OriginatorDocumentReference instead. |
| `_get_peppol_what_is_peppol_alert` | preparation rule | self, moves, moves_data, relevant_moves | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_peppol_what_is_peppol_message` | preparation rule | self, companies, moves, relevant_moves | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_peppol_partner_want_peppol_message` | preparation rule | self, partners, relevant_moves | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_peppol_what_is_pdp_message` | preparation rule | self, companies, moves, relevant_moves | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_peppol_document_params` | preparation rule | self, partner, invoice, invoice_data | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_do_peppol_pre_send` | internal rule | self, moves | `account_peppol` |  |  |
| `_send_peppol_documents` | internal rule | self, invoices_data_peppol, edi_user, params | `account_peppol` |  |  |
| `_get_peppol_attachments_linked_message` | preparation rule | self, edi_user | `account_peppol`, `l10n_fr_pdp` |  |  |
| `action_what_is_peppol_activate` | user action | self, moves | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_is_es_facturae_applicable` | internal rule | self, move | `l10n_es_edi_facturae` | model | Check if the Factura-e applies to the given move. |
| `_is_tbai_applicable` | internal rule | self, move | `l10n_es_edi_tbai` | model |  |
| `_l10n_es_edi_verifactu_get_move_info` | internal rule | self, moves | `l10n_es_edi_verifactu` | model |  |
| `_is_es_verifactu_applicable` | internal rule | self, move | `l10n_es_edi_verifactu` | model |  |
| `_is_gr_edi_applicable` | internal rule | self, move | `l10n_gr_edi` | model |  |
| `_l10n_gr_edi_try_upload_final_pdf` | internal rule | self, invoice, invoice_data | `l10n_gr_edi_e_invoo` | model |  |
| `_is_hu_edi_applicable` | internal rule | self, move | `l10n_hu_edi` | model |  |
| `_l10n_hu_edi_cron_update_status` | internal rule | self | `l10n_hu_edi` | model |  |
| `_is_in_edi_applicable` | internal rule | self, move | `l10n_in_edi` | model |  |
| `_is_it_edi_applicable` | internal rule | self, move | `l10n_it_edi` | model |  |
| `_l10n_jo_is_edi_applicable` | internal rule | self, move | `l10n_jo_edi` | model |  |
| `_get_l10n_ke_edi_tremol_warning_moves` | preparation rule | self, moves | `l10n_ke_edi_tremol` | model |  |
| `_get_l10n_ke_edi_tremol_warning_message` | preparation rule | self, warning_moves | `l10n_ke_edi_tremol` | model |  |
| `_l10n_pl_edi_try_status_fetch_from_ksef` | internal rule | self, invoices_data | `l10n_pl_edi` |  |  |
| `_is_ro_edi_applicable` | internal rule | self, move | `l10n_ro_edi` | model |  |
| `_is_rs_edi_applicable` | internal rule | self, move | `l10n_rs_edi` | model |  |
| `_is_sa_edi_applicable` | internal rule | self, move | `l10n_sa_edi` | model |  |
| `_is_tr_nilvera_applicable` | internal rule | self, move | `l10n_tr_nilvera_einvoice` | model |  |
| `_get_l10n_tr_tax_partner_address_alert` | preparation rule | self, moves | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  |  |
| `_get_l10n_tr_tax_partner_tax_office_alert` | preparation rule | self, moves | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  |  |
| `_get_l10n_tr_tax_company_tax_office_alert` | preparation rule | self, moves | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  |  |
| `_is_tw_edi_applicable` | internal rule | self, move | `l10n_tw_edi_ecpay` | model |  |
| `_is_tw_edi_issue_allowance_applicable` | internal rule | self, move | `l10n_tw_edi_ecpay` | model |  |
| `_l10n_tw_edi_generate_ecpay_json` | internal rule | self, invoice, invoice_data | `l10n_tw_edi_ecpay` | model |  |
| `_is_vn_edi_applicable` | internal rule | self, move | `l10n_vn_edi_viettel` | model |  |
| `_generate_sinvoice_file_date` | internal rule | self, invoice, invoice_data | `l10n_vn_edi_viettel` | model |  |
| `_prepare_snailmail_letter_values` | preparation rule | self, move | `snailmail_account` | model |  |

## Validation and error messages (14)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_default_pdf_report_id` | UserError | There is no template that applies to this move type. | `account` |
| `_raise_danger_alerts` | UserError | '\n'.join(danger_alert_messages) | `account` |
| `_check_move_constraints` | UserError | next(iter(move_constraints.values()), None) | `account` |
| `_check_invoice_report` | UserError | The sending of invoices is not set up properly, make sure the report used is set for invoices. | `account` |
| `_prepare_invoice_pdf_report` | ValidationError | Cannot identify the invoices in the generated PDF: %s | `account` |
| `_hook_if_errors` | UserError | self._format_error_text(error) | `account` |
| `_hook_if_errors` | RedirectWarning | '\n'.join(error['errors']) | `l10n_es_edi_verifactu` |
| `_check_move_constraints` | UserError | Operator label is required for sending invoices in Croatia. | `l10n_hr_edi` |
| `_check_move_constraints` | UserError | Operator OIB is required for sending invoices in Croatia. | `l10n_hr_edi` |
| `_check_move_constraints` | UserError | KPD categories must be defined on every invoice line for any Business Process Type other than P4. | `l10n_hr_edi` |
| `_check_move_constraints` | UserError | Name of custom business process is required for Business Process Type P99. | `l10n_hr_edi` |
| `_check_move_constraints` | ValidationError | For Croatia, all VAT taxes on an invoice should either be cash basis or not. | `l10n_hr_edi` |
| `_check_move_constraints` | ValidationError | For Croatia, Legal Notes should be provided for all cash basis taxes. | `l10n_hr_edi` |
| `_call_web_service_after_invoice_pdf_render` | UserError | Failed to send invoice via MojEracun: check configuration. | `l10n_hr_edi` |

Machine-readable definition: `../../../schemas/data/entities/account.move.send.json`.
