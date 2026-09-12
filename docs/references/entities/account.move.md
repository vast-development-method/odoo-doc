# Journal Entry (`account.move`)

**Transport name:** `account.move`  
**Storage name:** `account_move`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_debit_note`, `account_edi`, `account_edi_ubl_cii`, `account_fleet`, `account_payment`, `account_payment_interco`, `account_peppol`, `account_peppol_advanced_fields`, `account_peppol_response`, `sale`, `stock_account`, `sale_stock`, `event_booth_sale`, `hr_expense`, `point_of_sale`, `l10n_gcc_invoice`, `l10n_ae`, `l10n_anz_ubl_pint`, `l10n_latam_invoice_document`, `l10n_ar`, `website_sale`, `l10n_latam_check`, `l10n_ar_withholding`, `l10n_au`, `l10n_be`, `pos_sale`, `l10n_bg_ledger`, `l10n_br`, `l10n_ch`, `l10n_cl`, `l10n_cn`, `l10n_cz`, `l10n_de`, `purchase`, `l10n_dk_fik`, `l10n_dk_nemhandel`, `l10n_dk_nemhandel_response`, `l10n_dk_oioubl`, `l10n_ec`, `l10n_eg_edi_eta`, `l10n_es`, `l10n_es_edi_facturae`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_fi`, `l10n_fr_account`, `l10n_fr_facturx_chorus_pro`, `l10n_fr_pdp`, `l10n_fr_pdp_pos`, `l10n_gr_edi`, `l10n_gr_edi_e_invoo`, `l10n_hr_edi`, `l10n_hu`, `l10n_hu_edi`, `l10n_hu_edi_receive`, `l10n_id`, `l10n_id_efaktur_coretax`, `l10n_in`, `l10n_in_edi`, `l10n_in_ewaybill`, `l10n_in_pos`, `purchase_stock`, `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_it`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_it_stock_ddt`, `l10n_jo_edi`, `l10n_jo_edi_pos`, `l10n_jp_ubl_pint`, `l10n_ke`, `l10n_ke_edi_tremol`, `l10n_lk_invoice`, `l10n_mu_account`, `l10n_my_ubl_pint`, `l10n_my_edi`, `l10n_no`, `l10n_nz`, `l10n_pe`, `l10n_ph`, `l10n_pl`, `l10n_pl_edi`, `l10n_pl_edi_jst`, `l10n_ro_edi`, `l10n_rs`, `l10n_rs_edi`, `l10n_sa`, `l10n_sa_edi`, `l10n_sa_edi_pos`, `l10n_se`, `l10n_sg`, `l10n_sg_ubl_pint`, `l10n_si`, `l10n_sk`, `l10n_th`, `l10n_tr_nilvera_einvoice`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_uy`, `l10n_vn`, `l10n_vn_edi_viettel`, `l10n_vn_edi_viettel_pos`, `l10n_zm_account`, `mrp_account`, `stock_landed_costs`, `product_email_template`, `sale_project`, `sale_expense`, `purchase_edi_ubl_bis3`, `sale_timesheet`, `snailmail_account`

Description: Journal Entry

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `mail.thread.main.attachment`, `mail.activity.mixin`, `sequence.mixin`, `product.catalog.mixin`, `account.document.import.mixin`, `utm.mixin`, `pos.load.mixin`
- Default ordering: `date desc, name desc, invoice_date desc, id desc`
- Display name search fields: `["name", "partner_id.name", "ref"]`
- Company consistency is checked automatically on company-bound relations
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (508)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Number | single line text |  | computed by rule `_compute_name` and stored; writable through an inverse rule; changes are tracked in the message thread; indexed (trigram); not copied on duplication |
| `name_placeholder` | Name Placeholder | single line text |  | computed by rule `_compute_name_placeholder` (not stored) |
| `ref` | Reference | single line text |  | changes are tracked in the message thread; indexed (trigram); not copied on duplication |
| `date` | Date | date |  | required; computed by rule `_compute_date` and stored; changes are tracked in the message thread; indexed; not copied on duplication; precomputed before insertion |
| `state` | Status | selection |  | required; read only; default `draft`; changes are tracked in the message thread; not copied on duplication |
| `move_type` | Type | selection |  | required; read only; default `entry`; changes are tracked in the message thread; indexed |
| `is_storno` | Is Storno | boolean |  | computed by rule `_compute_is_storno` (not stored) |
| `journal_id` | Journal | many to one | `account.journal` | required; computed by rule `_compute_journal_id` and stored; writable through an inverse rule; restricted by domain `[('id', 'in', suitable_journal_ids)]`; must belong to the same company; precomputed before insertion |
| `journal_group_id` | Ledger | many to one | `account.journal.group` | searchable through a search rule |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; writable through an inverse rule; indexed; precomputed before insertion |
| `line_ids` | Journal Items | one to many | `account.move.line` | inverse field `move_id` |
| `journal_line_ids` | Journal Items (DEPRECATED) | one to many | `account.move.line` | not copied on duplication; restricted by domain `[["display_type", "not in", ["line_section", "line_subsection", "line_note"]]]`; inverse field `move_id` |
| `exchange_diff_partial_ids` | Related reconciliation | one to many | `account.partial.reconcile` | inverse field `exchange_move_id` |
| `origin_payment_id` | Payment | many to one | `account.payment` | indexed (btree_not_null); not copied on duplication; must belong to the same company |
| `matched_payment_ids` | Matched Payments | many to many | `account.payment` | not copied on duplication; association table `account_move__account_payment` |
| `reconciled_payment_ids` | Reconciled Payments | many to many | `account.payment` | computed by rule `_compute_reconciled_payment_ids` (not stored); searchable through a search rule; Help: Payments that have been reconciled with this invoice. |
| `payment_count` | Payment Count | integer |  | computed by rule `_compute_payment_count` (not stored) |
| `statement_line_id` | Statement Line | many to one | `account.bank.statement.line` | indexed (btree_not_null); not copied on duplication; must belong to the same company |
| `statement_id` | Statement | many to one |  | related through path `statement_line_id.statement_id` |
| `adjusting_entry_origin_move_ids` | Adjusting Entry Origin Moves | many to many | `account.move` | association table `adjusting_entries__account_move` |
| `adjusting_entry_origin_label` | Adjusting Entry Origin Label | single line text |  | computed by rule `_compute_adjusting_entry_origin_label` (not stored) |
| `adjusting_entry_origin_moves_count` | Adjusting Entry Origin Moves Count | integer |  | computed by rule `_compute_adjusting_entry_origin_moves_count` (not stored) |
| `adjusting_entries_move_ids` | Created Adjusting Entries | many to many | `account.move` | association table `adjusting_entries__account_move` |
| `adjusting_entries_count` | Adjusting Entries Count | integer |  | computed by rule `_compute_adjusting_entries_count` (not stored) |
| `tax_cash_basis_rec_id` | Tax Cash Basis Entry of | many to one | `account.partial.reconcile` | indexed (btree_not_null) |
| `tax_cash_basis_origin_move_id` | Cash Basis Origin | many to one | `account.move` | read only; indexed (btree_not_null); Help: The journal entry from which this tax cash basis journal entry has been created. |
| `tax_cash_basis_created_move_ids` | Cash Basis Entries | one to many | `account.move` | inverse field `tax_cash_basis_origin_move_id`; Help: The cash basis entries created from the taxes on this entry, when reconciling its lines. |
| `always_tax_exigible` | Always Tax Exigible | boolean |  | computed by rule `_compute_always_tax_exigible` and stored |
| `auto_post` | Auto-post | selection |  | required; default `no`; not copied on duplication; Help: Specify whether this entry is posted automatically on its accounting date, and any similar recurring invoices. |
| `auto_post_until` | Auto-post until | date |  | computed by rule `_compute_auto_post_until` and stored; not copied on duplication; Help: This recurring move will be posted up to and including this date. |
| `auto_post_origin_id` | First recurring entry | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication |
| `hide_post_button` | Hide Post Button | boolean |  | read only; computed by rule `_compute_hide_post_button` (not stored) |
| `checked` | Reviewed | boolean |  | computed by rule `_compute_checked` and stored; changes are tracked in the message thread; not copied on duplication |
| `posted_before` | Posted Before | boolean |  | not copied on duplication |
| `suitable_journal_ids` | Suitable Journal | many to many | `account.journal` | computed by rule `_compute_suitable_journal_ids` (not stored) |
| `highest_name` | Highest Name | single line text |  | computed by rule `_compute_highest_name` (not stored) |
| `made_sequence_gap` | Made Sequence Gap | boolean |  |  |
| `show_name_warning` | Show Name Warning | boolean |  |  |
| `type_name` | Type Name | single line text |  | computed by rule `_compute_type_name` (not stored) |
| `country_code` | Country Code | single line text |  | read only; related through path `company_id.account_fiscal_country_id.code` |
| `account_fiscal_country_group_codes` | Account Fiscal Country Group Codes | structured document |  | related through path `company_id.account_fiscal_country_group_codes` |
| `company_price_include` | Company Price Include | selection |  | read only; related through path `company_id.account_price_include` |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | restricted by domain `[["res_model", "=", "account.move"]]`; inverse field `res_id` |
| `audit_trail_message_ids` | Audit Trail Messages | one to many | `mail.message` | restricted by domain `[["model", "=", "account.move"], ["message_type", "=", "notification"]]`; inverse field `res_id` |
| `no_followup` | No Follow-Up | boolean |  | computed by rule `_compute_no_followup` (not stored); writable through an inverse rule; Help: Exclude this journal entry from follow-up reports. |
| `restrict_mode_hash_table` | Restrict Mode Hash Table | boolean |  | related through path `journal_id.restrict_mode_hash_table` |
| `secure_sequence_number` | Inalterability No Gap Sequence # | integer |  | read only; indexed; not copied on duplication |
| `inalterable_hash` | Inalterability Hash | single line text |  | read only; indexed (btree_not_null); not copied on duplication |
| `secured` | Secured | boolean |  | computed by rule `_compute_secured` (not stored); searchable through a search rule; Help: The entry is secured with an inalterable hash. |
| `invoice_line_ids` | Invoice lines | one to many | `account.move.line` | not copied on duplication; restricted by domain `[["display_type", "in", ["product", "line_section", "line_subsection", "line_note"]]]`; inverse field `move_id` |
| `invoice_date` | Invoice/Bill Date | date |  | indexed; not copied on duplication |
| `invoice_date_due` | Due Date | date |  | computed by rule `_compute_invoice_date_due` and stored; indexed; not copied on duplication |
| `delivery_date` | Delivery Date | date |  | computed by rule `_compute_delivery_date` and stored; writable through an inverse rule; not copied on duplication; precomputed before insertion |
| `show_delivery_date` | Show Delivery Date | boolean |  | computed by rule `_compute_show_delivery_date` (not stored) |
| `taxable_supply_date` | Taxable Supply Date | date |  | computed by rule `_compute_taxable_supply_date` and stored; not copied on duplication; precomputed before insertion |
| `show_taxable_supply_date` | Show Taxable Supply Date | boolean |  | computed by rule `_compute_show_taxable_supply_date` (not stored) |
| `taxable_supply_date_placeholder` | Taxable Supply Date Placeholder | single line text |  | computed by rule `_compute_taxable_supply_date_placeholder` (not stored) |
| `invoice_payment_term_id` | Payment Terms | many to one | `account.payment.term` | computed by rule `_compute_invoice_payment_term_id` and stored; writable through an inverse rule; must belong to the same company; precomputed before insertion |
| `needed_terms` | Needed Terms | binary |  | computed by rule `_compute_needed_terms` (not stored) |
| `needed_terms_dirty` | Needed Terms Dirty | boolean |  | computed by rule `_compute_needed_terms` (not stored) |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | read only; related through path `company_id.tax_calculation_rounding_method` |
| `show_journal` | Show Journal | boolean |  | computed by rule `_compute_show_journal` (not stored) |
| `partner_id` | Partner | many to one | `res.partner` | writable through an inverse rule; changes are tracked in the message thread; indexed; on delete of the target: restrict; must belong to the same company |
| `commercial_partner_id` | Commercial Entity | many to one | `res.partner` | read only; computed by rule `_compute_commercial_partner_id` and stored; on delete of the target: restrict; must belong to the same company |
| `partner_shipping_id` | Delivery Address | many to one | `res.partner` | computed by rule `_compute_partner_shipping_id` and stored; must belong to the same company; precomputed before insertion; Help: The delivery address will be used in the computation of the fiscal position. |
| `partner_bank_id` | Recipient Bank | many to one | `res.partner.bank` | computed by rule `_compute_partner_bank_id` and stored; changes are tracked in the message thread; indexed (btree_not_null); on delete of the target: restrict; must belong to the same company; Help: Bank Account Number to which the invoice will be paid. A Company bank account if this is a Customer Invoice or Vendor Credit Note, otherwise a Partner bank account number. |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` | computed by rule `_compute_fiscal_position_id` and stored; on delete of the target: restrict; must belong to the same company; precomputed before insertion; Help: Fiscal positions are used to adapt taxes and accounts for particular customers or sales orders/invoices. The default value comes from the customer. |
| `payment_reference` | Payment Reference | single line text |  | computed by rule `_compute_payment_reference` and stored; writable through an inverse rule; changes are tracked in the message thread; indexed (trigram); not copied on duplication; Help: The payment reference to set on journal items. |
| `sanitize_payment_reference` | Label sanitize | single line text |  | computed by rule `_compute_sanitize_payment_reference` (not stored) |
| `display_qr_code` | Display quick response-code | boolean |  | computed by rule `_compute_display_qr_code` (not stored) |
| `display_link_qr_code` | Display Link quick response-code | boolean |  | computed by rule `_compute_display_link_qr_code` (not stored) |
| `qr_code_method` | Payment quick response-code | selection |  | not copied on duplication; Help: Type of QR-code to be generated for the payment of this invoice, when printing it. If left blank, the first available and usable method will be used. |
| `invoice_outstanding_credits_debits_widget` | Invoice Outstanding Credits Debits Widget | binary |  | computed by rule `_compute_payments_widget_to_reconcile_info` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `invoice_has_outstanding` | Invoice Has Outstanding | boolean |  | computed by rule `_compute_invoice_has_outstanding` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `invoice_payments_widget` | Invoice Payments Widget | binary |  | computed by rule `_compute_payments_widget_reconciled_info` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `preferred_payment_method_line_id` | Preferred Payment Method Line | many to one | `account.payment.method.line` | computed by rule `_compute_preferred_payment_method_line_id` and stored |
| `company_currency_id` | Company Currency | many to one |  | read only; related through path `company_id.currency_id` |
| `currency_id` | Currency | many to one | `res.currency` | required; computed by rule `_compute_currency_id` and stored; writable through an inverse rule; changes are tracked in the message thread; precomputed before insertion |
| `expected_currency_rate` | Expected Currency Rate | float |  | computed by rule `_compute_expected_currency_rate` (not stored) |
| `invoice_currency_rate` | Currency Rate | float |  | computed by rule `_compute_invoice_currency_rate` and stored; not copied on duplication; precomputed before insertion; Help: Currency rate from company currency to document currency. |
| `direction_sign` | Direction Sign | integer |  | computed by rule `_compute_direction_sign` (not stored); Help: Multiplicator depending on the document type, to convert a price into a balance |
| `amount_untaxed` | Untaxed Amount | monetary |  | read only; computed by rule `_compute_amount` and stored; changes are tracked in the message thread |
| `amount_tax` | Tax | monetary |  | read only; computed by rule `_compute_amount` and stored |
| `amount_total` | Total | monetary |  | read only; computed by rule `_compute_amount` and stored; writable through an inverse rule |
| `amount_residual` | Amount Due | monetary |  | computed by rule `_compute_amount` and stored |
| `amount_untaxed_signed` | Untaxed Amount Signed | monetary |  | read only; computed by rule `_compute_amount` and stored; currency taken from `company_currency_id` |
| `amount_untaxed_in_currency_signed` | Untaxed Amount Signed Currency | monetary |  | read only; computed by rule `_compute_amount` and stored; currency taken from `currency_id` |
| `amount_tax_signed` | Tax Signed | monetary |  | read only; computed by rule `_compute_amount` and stored; currency taken from `company_currency_id` |
| `amount_total_signed` | Total Signed | monetary |  | read only; computed by rule `_compute_amount` and stored; currency taken from `company_currency_id` |
| `amount_total_in_currency_signed` | Total in Currency Signed | monetary |  | read only; computed by rule `_compute_amount` and stored; currency taken from `currency_id` |
| `amount_residual_signed` | Amount Due Signed | monetary |  | computed by rule `_compute_amount` and stored; currency taken from `company_currency_id` |
| `tax_totals` | Invoice Totals | binary |  | computed by rule `_compute_tax_totals` (not stored); writable through an inverse rule; Help: Edit Tax amounts if you encounter rounding issues. |
| `payment_state` | Payment Status | selection |  | read only; computed by rule `_compute_payment_state` and stored; changes are tracked in the message thread; not copied on duplication |
| `status_in_payment` | Status In Payment | selection |  | computed by rule `_compute_status_in_payment` (not stored); not copied on duplication |
| `amount_total_words` | Amount total in words | single line text |  | computed by rule `_compute_amount_total_words` (not stored) |
| `reversed_entry_id` | Reversal of | many to one | `account.move` | read only; changes are tracked in the message thread; indexed (btree_not_null); not copied on duplication; must belong to the same company; extended by packages `l10n_jo_edi` |
| `reversal_move_ids` | Reversal Move | one to many | `account.move` | inverse field `reversed_entry_id` |
| `invoice_vendor_bill_id` | Vendor Bill | many to one | `account.move` | must belong to the same company; Help: Auto-complete from a previous bill or refund. |
| `invoice_source_email` | Source Email | single line text |  | changes are tracked in the message thread |
| `invoice_partner_display_name` | Invoice Partner Display Name | single line text |  | computed by rule `_compute_invoice_partner_display_info` and stored |
| `is_manually_modified` | Is Manually Modified | boolean |  |  |
| `quick_edit_mode` | Quick Edit Mode | boolean |  | computed by rule `_compute_quick_edit_mode` (not stored) |
| `quick_edit_total_amount` | Total (Tax inc.) | monetary |  | Help: Use this field to encode the total amount of the invoice. The system will automatically create one invoice line with default values to match it. |
| `quick_encoding_vals` | Quick Encoding Vals | structured document |  | computed by rule `_compute_quick_encoding_vals` (not stored) |
| `narration` | Terms and Conditions | rich text |  | computed by rule `_compute_narration` and stored; translatable; extended by packages `l10n_gcc_invoice` |
| `is_move_sent` | Is Move Sent | boolean |  | read only; not copied on duplication; Help: It indicates that the invoice/payment has been sent or the PDF has been generated. |
| `is_being_sent` | Is Being Sent | boolean |  | computed by rule `_compute_is_being_sent` (not stored); Help: Is the move being sent asynchronously |
| `move_sent_values` | Sent | selection |  | computed by rule `compute_move_sent_values` (not stored); searchable through a search rule |
| `invoice_user_id` | Salesperson | many to one | `res.users` | computed by rule `_compute_invoice_default_sale_person` and stored; changes are tracked in the message thread; not copied on duplication |
| `user_id` | User | many to one |  | related through path `invoice_user_id` |
| `invoice_origin` | Origin | single line text |  | read only; changes are tracked in the message thread; not copied on duplication; Help: The document(s) that generated the invoice. |
| `invoice_incoterm_id` | Incoterm | many to one | `account.incoterms` | computed by rule `_compute_incoterm` and stored; Help: International Commercial Terms are a series of predefined commercial terms used in international transactions. |
| `incoterm_location` | Incoterm Location | single line text |  | computed by rule `_compute_incoterm_location` and stored |
| `invoice_cash_rounding_id` | Cash Rounding Method | many to one | `account.cash.rounding` | Help: Defines the smallest coinage of the currency that can be used to pay by cash. |
| `sending_data` | Sending Data | structured document |  | not copied on duplication |
| `invoice_pdf_report_id` | Portable Document Format Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='PDF Attachment', compute=lambda self: self._compute_linked_attachment_id('invoice_pdf_report_id', 'invoice_pdf_report_file'), depends=['invoice_pdf_report_file'])` (not stored) |
| `invoice_pdf_report_file` | Portable Document Format File | binary |  | not copied on duplication |
| `invoice_incoterm_placeholder` | Invoice Incoterm Placeholder | single line text |  | computed by rule `_compute_invoice_incoterm_placeholder` (not stored) |
| `invoice_filter_type_domain` | Invoice Filter Type Domain | single line text |  | computed by rule `_compute_invoice_filter_type_domain` (not stored) |
| `bank_partner_id` | Bank Partner | many to one | `res.partner` | computed by rule `_compute_bank_partner_id` (not stored); Help: Technical field to get the domain on the bank |
| `tax_lock_date_message` | Tax Lock Date Message | single line text |  | computed by rule `_compute_tax_lock_date_message` (not stored) |
| `display_inactive_currency_warning` | Display Inactive Currency Warning | boolean |  | computed by rule `_compute_display_inactive_currency_warning` (not stored) |
| `tax_country_id` | Tax Country | many to one | `res.country` | computed by rule `_compute_tax_country_id` (not stored) |
| `tax_country_code` | Tax Country Code | single line text |  | computed by rule `_compute_tax_country_code` (not stored) |
| `has_reconciled_entries` | Has Reconciled Entries | boolean |  | computed by rule `_compute_has_reconciled_entries` (not stored) |
| `show_reset_to_draft_button` | Show Reset To Draft Button | boolean |  | computed by rule `_compute_show_reset_to_draft_button` (not stored) |
| `partner_credit_warning` | Partner Credit Warning | multi line text |  | computed by rule `_compute_partner_credit_warning` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `duplicated_ref_ids` | Duplicated Ref | many to many | `account.move` | computed by rule `_compute_duplicated_ref_ids` (not stored) |
| `is_draft_duplicated_ref_ids` | Is Draft Duplicated Ref | boolean |  | computed by rule `_compute_is_draft_duplicated_ref_ids` (not stored) |
| `is_exact_move_duplicate` | Is Exact Move Duplicate | boolean |  | computed by rule `_compute_is_draft_duplicated_ref_ids` (not stored) |
| `need_cancel_request` | Need Cancel Request | boolean |  | computed by rule `_compute_need_cancel_request` (not stored) |
| `show_update_fpos` | Has Fiscal Position Changed | boolean |  |  |
| `payment_term_details` | Payment Term Details | binary |  | computed by rule `_compute_payment_term_details` (not stored) |
| `show_payment_term_details` | Show Payment Term Details | boolean |  | computed by rule `_compute_show_payment_term_details` (not stored) |
| `show_discount_details` | Show Discount Details | boolean |  | computed by rule `_compute_show_payment_term_details` (not stored) |
| `abnormal_amount_warning` | Abnormal Amount Warning | multi line text |  | computed by rule `_compute_abnormal_warnings` (not stored) |
| `abnormal_date_warning` | Abnormal Date Warning | multi line text |  | computed by rule `_compute_abnormal_warnings` (not stored) |
| `alerts` | Alerts | structured document |  | computed by rule `_compute_alerts` (not stored) |
| `taxes_legal_notes` | Taxes Legal Notes | rich text |  | computed by rule `_compute_taxes_legal_notes` (not stored) |
| `next_payment_date` | Next Payment Date | date |  | computed by rule `_compute_next_payment_date` (not stored); searchable through a search rule |
| `display_send_button` | Display Send Button | boolean |  | computed by rule `_compute_display_send_button` (not stored) |
| `highlight_send_button` | Highlight Send Button | boolean |  | computed by rule `_compute_highlight_send_button` (not stored) |
| `is_sale_installed` | Is Sale Installed | boolean |  | computed by rule `_compute_is_sale_installed` (not stored) |
| `statement_line_ids` | Statements | one to many | `account.bank.statement.line` | inverse field `move_id` |
| `payment_ids` | Payments | one to many | `account.payment` | inverse field `move_id` |
| `debit_origin_id` | Original Invoice Debited | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication |
| `debit_note_ids` | Debit Notes | one to many | `account.move` | inverse field `debit_origin_id`; Help: The debit notes created for this invoice |
| `debit_note_count` | Number of Debit Notes | integer |  | computed by rule `_compute_debit_count` (not stored) |
| `edi_document_ids` | Electronic data interchange Document | one to many | `account.edi.document` | inverse field `move_id` |
| `edi_state` | Electronic invoicing | selection |  | computed by rule `_compute_edi_state` and stored; Help: The aggregated state of all the EDIs with web-service of this move |
| `edi_error_count` | Electronic data interchange Error Count | integer |  | computed by rule `_compute_edi_error_count` (not stored); Help: How many EDIs are in error for this move? |
| `edi_blocking_level` | Electronic data interchange Blocking Level | selection |  | computed by rule `_compute_edi_error_message` (not stored) |
| `edi_error_message` | Electronic data interchange Error Message | rich text |  | computed by rule `_compute_edi_error_message` (not stored) |
| `edi_web_services_to_process` | Electronic data interchange Web Services To Process | multi line text |  | computed by rule `_compute_edi_web_services_to_process` (not stored) |
| `edi_show_cancel_button` | Electronic data interchange Show Cancel Button | boolean |  | computed by rule `_compute_edi_show_cancel_button` (not stored) |
| `edi_show_abandon_cancel_button` | Electronic data interchange Show Abandon Cancel Button | boolean |  | computed by rule `_compute_edi_show_abandon_cancel_button` (not stored) |
| `edi_show_force_cancel_button` | Electronic data interchange Show Force Cancel Button | boolean |  | computed by rule `_compute_edi_show_force_cancel_button` (not stored) |
| `ubl_cii_xml_id` | Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='Attachment', compute=lambda self: self._compute_linked_attachment_id('ubl_cii_xml_id', 'ubl_cii_xml_file'), depends=['ubl_cii_xml_file'])` (not stored) |
| `ubl_cii_xml_file` | Universal Business Language/Cross Industry Invoice File | binary |  | not copied on duplication |
| `ubl_cii_xml_filename` | Universal Business Language/Cross Industry Invoice Filename | single line text |  | computed by rule `_compute_filename` (not stored) |
| `transaction_ids` | Transactions | many to many | `payment.transaction` | read only; not copied on duplication; association table `account_invoice_transaction_rel` |
| `authorized_transaction_ids` | Authorized Transactions | many to many | `payment.transaction` | read only; computed by rule `_compute_authorized_transaction_ids` (not stored); not copied on duplication |
| `transaction_count` | Transaction Count | integer |  | computed by rule `_compute_transaction_count` (not stored) |
| `amount_paid` | Amount paid | monetary |  | computed by rule `_compute_amount_paid` (not stored) |
| `peppol_message_uuid` | PEPPOL message identifier | single line text |  | indexed (btree_not_null); not copied on duplication |
| `peppol_move_state` | E-Invoicing Status | selection |  | computed by rule `_compute_peppol_move_state` and stored; not copied on duplication; extended by packages `account_peppol_response`, `l10n_fr_pdp` |
| `peppol_is_sent` | the pan-European public procurement online network Is Sent | boolean |  | computed by rule `_compute_peppol_is_sent` (not stored) |
| `peppol_contract_document_reference` | [DEPRECATED] Contract Document Reference | single line text |  | Help: A reference to the contract document. |
| `peppol_project_reference` | [DEPRECATED] Project Reference | single line text |  | Help: A reference to the project. |
| `peppol_originator_document_reference` | [DEPRECATED] Originator Document Reference | single line text |  | Help: A reference to the document that originated the order. |
| `peppol_despatch_document_reference` | [DEPRECATED] Despatch Document Reference | single line text |  | Help: A reference to the despatch document. |
| `peppol_additional_document_reference` | [DEPRECATED] Additional Document Reference | single line text |  | Help: A reference to an additional supporting document. Only one document can be referenced. |
| `peppol_accounting_cost` | [DEPRECATED] Accounting Cost | single line text |  | Help: A textual description or a code to identify the accounting cost. |
| `peppol_delivery_location_id` | [DEPRECATED] Delivery Location global location number | single line text |  | Help: The Global Location Number (GLN) of the delivery location. |
| `peppol_response_ids` | the pan-European public procurement online network Response | one to many | `account.peppol.response` | inverse field `move_id` |
| `peppol_can_send_response` | the pan-European public procurement online network Can Send Response | boolean |  | computed by rule `_compute_peppol_can_send_response` (not stored) |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored; changes are tracked in the message thread; on delete of the target: set null; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `campaign_id` | Campaign | many to one |  | on delete of the target: set null |
| `medium_id` | Medium | many to one |  | on delete of the target: set null |
| `source_id` | Source | many to one |  | on delete of the target: set null |
| `sale_order_count` | Sale Order Count | integer |  | computed by rule `_compute_origin_so_count` (not stored) |
| `sale_warning_text` | Sale Warning | multi line text |  | computed by rule `_compute_sale_warning_text` (not stored); Help: Internal warning for the partner or the products as set by the user. |
| `stock_move_ids` | Stock Move | one to many | `stock.move` | inverse field `account_move_id` |
| `expense_ids` | Expense | one to many | `hr.expense` | inverse field `account_move_id` |
| `nb_expenses` | Number of Expenses | integer |  | computed by rule `_compute_nb_expenses` (not stored) |
| `pos_order_ids` | Point of sale Order | one to many | `pos.order` | inverse field `account_move` |
| `pos_payment_ids` | Point of sale Payment | one to many | `pos.payment` | inverse field `account_move_id` |
| `pos_refunded_invoice_ids` | Point of sale Refunded Invoice | many to many | `account.move` | association table `refunded_invoices` |
| `reversed_pos_order_id` | Reversed point of sale Order | many to one | `pos.order` | indexed (btree_not_null); Help: The pos order that was reverted after closing the session to create an invoice for it. |
| `pos_session_ids` | point of sale Sessions | one to many | `pos.session` | inverse field `move_id` |
| `pos_order_count` | point of sale Order Count | integer |  | computed by rule `_compute_origin_pos_count` (not stored) |
| `l10n_latam_available_document_type_ids` | Localization Latam Available Document Type | many to many | `l10n_latam.document.type` | computed by rule `_compute_l10n_latam_available_document_types` (not stored) |
| `l10n_latam_document_type_id` | Document Type | many to one | `l10n_latam.document.type` | computed by rule `_compute_l10n_latam_document_type` and stored; indexed (btree_not_null) |
| `l10n_latam_document_number` | Document Number | single line text |  | computed by rule `_compute_l10n_latam_document_number` (not stored); writable through an inverse rule |
| `l10n_latam_use_documents` | Localization Latam Use Documents | boolean |  | computed by rule `_compute_l10n_latam_use_documents` (not stored); searchable through a search rule |
| `l10n_latam_manual_document_number` | Manual Number | boolean |  | computed by rule `_compute_l10n_latam_manual_document_number` (not stored) |
| `l10n_latam_document_type_id_code` | Doc Type | single line text |  | related through path `l10n_latam_document_type_id.code` |
| `l10n_ar_afip_responsibility_type_id` | ARCA Responsibility Type | many to one | `l10n_ar.afip.responsibility.type` | Help: Defined by ARCA to identify the type of responsibilities that a person or a legal entity could have and that impacts in the type of operations and requirements they need. |
| `l10n_ar_afip_concept` | ARCA Concept | selection |  | computed by rule `_compute_l10n_ar_afip_concept` (not stored); values provided by rule `_get_afip_invoice_concepts`; Help: A concept is suggested regarding the type of the products on the invoice. |
| `l10n_ar_afip_service_start` | ARCA Service Start Date | date |  |  |
| `l10n_ar_afip_service_end` | ARCA Service End Date | date |  |  |
| `website_id` | Website | many to one | `website` | read only; computed by rule `_compute_website_id` and stored; changes are tracked in the message thread; Help: Website through which this invoice was created for eCommerce orders. |
| `l10n_ar_withholding_ids` | Withholdings | one to many | `account.move.line` | read only; computed by rule `_compute_l10n_ar_withholding_ids` (not stored); inverse field `move_id` |
| `l10n_bg_document_type` | Document Type (BG) | selection |  | computed by rule `_compute_l10n_bg_document_type` and stored; not copied on duplication; values provided by rule `_l10n_bg_document_type_selection_values` |
| `l10n_bg_document_number` | Document Number (BG) | single line text |  | computed by rule `_compute_l10n_bg_document_number` (not stored) |
| `l10n_bg_exemption_reason` | Exemption reason (BG) | selection |  |  |
| `l10n_ch_is_qr_valid` | Localization Ch Is Quick response Valid | boolean |  | computed by rule `_compute_l10n_ch_qr_is_valid` (not stored); Help: Determines whether an invoice can be printed as a QR or not |
| `partner_id_vat` | value-added tax No | single line text |  | related through path `partner_id.vat` |
| `l10n_latam_internal_type` | L10n Latam Internal Type | selection |  | related through path `l10n_latam_document_type_id.internal_type` |
| `fapiao` | Fapiao Number | single line text |  | changes are tracked in the message thread; not copied on duplication; maximum length 8 |
| `purchase_vendor_bill_id` | Auto-complete | many to one | `purchase.bill.union` | Help: Auto-complete from a previous bill, refund, or purchase order. |
| `purchase_id` | Purchase Order | many to one | `purchase.order` | Help: Auto-complete from a past purchase order. |
| `purchase_order_count` | Purchase Order Count | integer |  | computed by rule `_compute_origin_po_count` (not stored) |
| `purchase_order_name` | Purchase Order Name | single line text |  | computed by rule `_compute_purchase_order_name` (not stored) |
| `is_purchase_matched` | Is Purchase Matched | boolean |  | computed by rule `_compute_is_purchase_matched` (not stored) |
| `purchase_warning_text` | Purchase Warning | multi line text |  | computed by rule `_compute_purchase_warning_text` (not stored); Help: Internal warning for the partner or the products as set by the user. |
| `nemhandel_message_uuid` | Nemhandel message identifier | single line text |  | not copied on duplication |
| `nemhandel_move_state` | Nemhandel status | selection |  | computed by rule `_compute_nemhandel_move_state` and stored; not copied on duplication; extended by packages `l10n_dk_nemhandel_response` |
| `nemhandel_response_ids` | Nemhandel Response | one to many | `nemhandel.response` | inverse field `move_id` |
| `nemhandel_can_send_response` | Nemhandel Can Send Response | boolean |  | computed by rule `_compute_nemhandel_can_send_response` (not stored) |
| `l10n_ec_sri_payment_id` | Payment Method (SRI) | many to one | `l10n_ec.sri.payment` | default computed dynamically (lambda self: self.env['l10n_ec.sri.payment'].search([], limit=1)); Help: Ecuador: Payment Methods Defined by the SRI. |
| `l10n_eg_long_id` | ETA Long identifier | single line text |  | computed by rule `_compute_eta_long_id` (not stored) |
| `l10n_eg_qr_code` | ETA quick response Code | single line text |  | computed by rule `_compute_eta_qr_code_str` (not stored) |
| `l10n_eg_submission_number` | Submission identifier | single line text |  | computed by rule `_compute_eta_response_data` and stored; not copied on duplication |
| `l10n_eg_uuid` | Document UUID | single line text |  | computed by rule `_compute_eta_response_data` and stored; not copied on duplication |
| `l10n_eg_eta_json_doc_file` | ETA JavaScript Object Notation Document | binary |  | not copied on duplication |
| `l10n_eg_signing_time` | Signing Time | date and time |  | not copied on duplication |
| `l10n_eg_is_signed` | Localization Eg Is Signed | boolean |  | not copied on duplication |
| `l10n_es_is_simplified` | Is Simplified | boolean |  | computed by rule `_compute_l10n_es_is_simplified` and stored |
| `l10n_es_edi_facturae_xml_id` | Facturae Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='Facturae Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_es_edi_facturae_xml_id', 'l10n_es_edi_facturae_xml_file'), depends=['l10n_es_edi_facturae_xml_file'])` (not stored) |
| `l10n_es_edi_facturae_xml_file` | Facturae File | binary |  | not copied on duplication |
| `l10n_es_edi_facturae_reason_code` | Spanish Facturae electronic data interchange Reason Code | selection |  | computed by rule `_compute_l10n_es_edi_facturae_reason_code` and stored |
| `l10n_es_invoicing_period_start_date` | Invoice Period Start Date | date |  |  |
| `l10n_es_invoicing_period_end_date` | Invoice Period End Date | date |  |  |
| `l10n_es_payment_means` | Payment Means | selection |  | computed by rule `_compute_l10n_es_payment_means` and stored |
| `l10n_es_edi_is_required` | Is the Spanish electronic data interchange needed | boolean |  | computed by rule `_compute_l10n_es_edi_is_required` (not stored) |
| `l10n_es_edi_csv` | comma-separated values return code | single line text |  | changes are tracked in the message thread; not copied on duplication |
| `l10n_es_registration_date` | Registration Date | date |  | not copied on duplication |
| `l10n_es_tbai_state` | TicketBAI status | selection |  | computed by rule `_compute_l10n_es_tbai_state` (not stored) |
| `l10n_es_tbai_chain_index` | TicketBAI chain index | integer |  | related through path `l10n_es_tbai_post_document_id.chain_index`; Help: Invoice index in chain, set if and only if an in-chain XML was submitted and did not error |
| `l10n_es_tbai_post_document_id` | Localization Es electronic invoicing (Basque) Post Document | many to one | `l10n_es_edi_tbai.document` | read only; not copied on duplication |
| `l10n_es_tbai_cancel_document_id` | Localization Es electronic invoicing (Basque) Cancel Document | many to one | `l10n_es_edi_tbai.document` | read only; not copied on duplication |
| `l10n_es_tbai_post_file` | TicketBAI Post File | binary |  | related through path `l10n_es_tbai_post_document_id.xml_attachment_id.datas` |
| `l10n_es_tbai_post_file_name` | TicketBAI Post Attachment Name | single line text |  | related through path `l10n_es_tbai_post_document_id.xml_attachment_id.name` |
| `l10n_es_tbai_cancel_file` | TicketBAI Cancel File | binary |  | related through path `l10n_es_tbai_cancel_document_id.xml_attachment_id.datas` |
| `l10n_es_tbai_cancel_file_name` | TicketBAI Cancel File Name | single line text |  | related through path `l10n_es_tbai_cancel_document_id.xml_attachment_id.name` |
| `l10n_es_tbai_is_required` | TicketBAI required | boolean |  | computed by rule `_compute_l10n_es_tbai_is_required` (not stored); Help: Is the Basque EDI (TicketBAI) needed ? |
| `l10n_es_tbai_refund_reason` | Invoice Refund Reason Code (TicketBai) | selection |  | not copied on duplication; Help: BOE-A-1992-28740. Ley 37/1992, de 28 de diciembre, del Impuesto sobre el Valor Añadido. Artículo 80. Modificación de la base imponible. |
| `l10n_es_tbai_reversed_ids` | Refunded Vendor Bills | many to many | `account.move` | restricted by domain `[('move_type', '=', 'in_invoice'), ('commercial_partner_id', '=', commercial_partner_id)]`; association table `account_move_tbai_reversed_moves`; Help: In the case where a vendor refund has multiple original invoices, you can set them here. |
| `l10n_es_edi_verifactu_required` | Veri*Factu Required | boolean |  | related through path `company_id.l10n_es_edi_verifactu_required` |
| `l10n_es_edi_verifactu_document_ids` | Veri*Factu Documents | one to many | `l10n_es_edi_verifactu.document` | inverse field `move_id` |
| `l10n_es_edi_verifactu_state` | Veri*Factu Status | selection |  | computed by rule `_compute_l10n_es_edi_verifactu_state` and stored; Help: - Rejected: Successfully sent to the AEAT, but it was rejected during validation                 - Registered with Errors: Registered at the AEAT, but the AEAT has some issues with the sent document                 - Accepted: Registered by the AEAT without errors                 - Cancelled: Registered by the AEAT as cancelled |
| `l10n_es_edi_verifactu_warning_level` | Veri*Factu Warning Level | single line text |  | computed by rule `_compute_l10n_es_edi_verifactu_warning` (not stored) |
| `l10n_es_edi_verifactu_warning` | Veri*Factu Warning | rich text |  | computed by rule `_compute_l10n_es_edi_verifactu_warning` (not stored) |
| `l10n_es_edi_verifactu_qr_code` | Veri*Factu quick response Code | single line text |  | computed by rule `_compute_l10n_es_edi_verifactu_qr_code` (not stored) |
| `l10n_es_edi_verifactu_show_cancel_button` | Show Veri*Factu Cancel Button | boolean |  | computed by rule `_compute_l10n_es_edi_verifactu_show_cancel_button` (not stored) |
| `l10n_es_edi_verifactu_available_clave_regimens` | Available Veri*Factu Regime Key | single line text |  | computed by rule `_compute_l10n_es_edi_verifactu_available_clave_regimens` (not stored); Help: Technical field to enable a dynamic selection of the field "Veri*Factu Regime Key" |
| `l10n_es_edi_verifactu_clave_regimen` | Veri*Factu Regime Key | selection |  | computed by rule `_compute_l10n_es_edi_verifactu_clave_regimen` and stored; values provided by rule `_l10n_es_edi_verifactu_clave_regimen_selection` |
| `l10n_es_edi_verifactu_substituted_entry_id` | Substitution of | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication; must belong to the same company |
| `l10n_es_edi_verifactu_substitution_move_ids` | Substituted by | one to many | `account.move` | inverse field `l10n_es_edi_verifactu_substituted_entry_id` |
| `l10n_es_edi_verifactu_refund_reason` | Veri*Factu Refund Reason | selection |  | not copied on duplication |
| `l10n_fr_is_company_french` | Localization Fr Is Company French | boolean |  | computed by rule `_compute_l10n_fr_is_company_french` (not stored) |
| `buyer_reference` | Buyer Reference | single line text |  | Help: 'Code de Service' in Chorus PRO. |
| `contract_reference` | Contract Reference | single line text |  | Help: 'Numéro de Marché' in Chorus PRO. |
| `purchase_order_reference` | Purchase Order Reference | single line text |  | Help: 'Engagement Juridique' in Chorus PRO. |
| `pdp_ppf_move_state` | PPF Invoice Status | selection |  | computed by rule `_compute_pdp_ppf_state` and stored; not copied on duplication |
| `pdp_lifecycle_residual` | Lifecycle Residual | monetary |  | computed by rule `_compute_pdp_lifecycle_residual` and stored; not copied on duplication; Help: Technical field indicating the amount of collected money we have still to report to the PPF via a lifecycle. |
| `pdp_ppf_lifecycle_state` | PPF Lifeycle Status | selection |  | computed by rule `_compute_pdp_ppf_state` and stored; not copied on duplication |
| `pdp_can_send_response` | Pdp Can Send Response | boolean |  | computed by rule `_compute_pdp_can_send_response` (not stored) |
| `pdp_is_sent` | Pdp Is Sent | boolean |  | computed by rule `_compute_pdp_is_sent` (not stored) |
| `pdp_uses_pdp` | Pdp Uses Pdp | boolean |  | computed by rule `_compute_pdp_uses_pdp` (not stored) |
| `l10n_fr_pdp_sent_in_flow_ids` | Sent in PDP Flows | many to many | `l10n.fr.pdp.reports.flow` | not copied on duplication; association table `sent_account_move__pdp_flow` |
| `l10n_fr_pdp_last_flow_id` | Last PDP Flow | many to one | `l10n.fr.pdp.reports.flow` | computed by rule `_compute_l10n_fr_pdp_last_flow_id` and stored; changes are tracked in the message thread; not copied on duplication |
| `l10n_fr_pdp_status` | E-Reporting Status | selection |  | computed by rule `_compute_l10n_fr_pdp_status` and stored; changes are tracked in the message thread; not copied on duplication |
| `l10n_fr_pdp_display_info` | Localization Fr Pdp Display Info | boolean |  | related through path `company_id.l10n_fr_f10_enable_reporting` |
| `l10n_fr_pdp_flow_10_report_type` | Localization Fr Pdp Flow 10 Report Type | selection |  | computed by rule `_compute_l10n_fr_pdp_flow_10_report_type` and stored; not copied on duplication |
| `l10n_fr_pdp_flow_10_operation_type` | Localization Fr Pdp Flow 10 Operation Type | selection |  | computed by rule `_compute_l10n_fr_pdp_flow_10_operation_type` and stored; not copied on duplication |
| `l10n_fr_pdp_error_message` | Flow 10 blocking errors | multi line text |  | computed by rule `_compute_l10n_fr_pdp_error_message` (not stored) |
| `l10n_fr_pdp_has_error` | Localization Fr Pdp Has Error | boolean |  | read only; computed by rule `_compute_l10n_fr_pdp_has_error` and stored; not copied on duplication |
| `l10n_gr_edi_mark` | Mark | single line text |  | computed by rule `_compute_from_l10n_gr_edi_document_ids` and stored |
| `l10n_gr_edi_cls_mark` | Classification Mark | single line text |  | computed by rule `_compute_from_l10n_gr_edi_document_ids` and stored |
| `l10n_gr_edi_document_ids` | Localization Gr Electronic data interchange Document | one to many | `l10n_gr_edi.document` | read only; not copied on duplication; inverse field `move_id` |
| `l10n_gr_edi_state` | myDATA Status | selection |  | computed by rule `_compute_from_l10n_gr_edi_document_ids` and stored; changes are tracked in the message thread; on delete of the target: {"invoice_pending": "set null"}; extended by packages `l10n_gr_edi_e_invoo` |
| `l10n_gr_edi_available_inv_type` | Localization Gr Electronic data interchange Available Inv Type | single line text |  | computed by rule `_compute_l10n_gr_edi_available_inv_type` (not stored) |
| `l10n_gr_edi_correlation_id` | Correlated Invoice | many to one | `account.move` |  |
| `l10n_gr_edi_inv_type` | myDATA Invoice Type | selection |  | computed by rule `_compute_l10n_gr_edi_inv_type` and stored |
| `l10n_gr_edi_payment_method` | Payment Method | selection |  | computed by rule `_compute_l10n_gr_edi_payment_method` and stored |
| `l10n_gr_edi_alerts` | Localization Gr Electronic data interchange Alerts | structured document |  | computed by rule `_compute_l10n_gr_edi_alerts` (not stored) |
| `l10n_gr_edi_need_correlated` | Localization Gr Electronic data interchange Need Correlated | boolean |  | computed by rule `_compute_l10n_gr_edi_need_fields` (not stored) |
| `l10n_gr_edi_need_payment_method` | Localization Gr Electronic data interchange Need Payment Method | boolean |  | computed by rule `_compute_l10n_gr_edi_need_fields` (not stored) |
| `l10n_gr_edi_enable_view_mydata` | Localization Gr Electronic data interchange Enable View Mydata | boolean |  | computed by rule `_compute_l10n_gr_edi_enable_fields` (not stored) |
| `l10n_gr_edi_enable_send_invoices` | Localization Gr Electronic data interchange Enable Send Invoices | boolean |  | computed by rule `_compute_l10n_gr_edi_enable_fields` (not stored) |
| `l10n_gr_edi_enable_send_expense_classification` | Localization Gr Electronic data interchange Enable Send Expense Classification | boolean |  | computed by rule `_compute_l10n_gr_edi_enable_fields` (not stored) |
| `l10n_gr_edi_attachment_id` | Localization Gr Electronic data interchange Attachment | many to one | `ir.attachment` | computed by rule `_compute_from_l10n_gr_edi_document_ids` and stored |
| `l10n_hr_process_type` | Business Process Type | selection |  | computed by rule `_compute_l10n_hr_process_type` and stored; not copied on duplication |
| `l10n_hr_customer_defined_process_name` | Custom Process Name | single line text |  | Help: Required when Process Type is P99. Specify the name of your custom business process. This will appear in the UBL as P99:YourProcessName |
| `l10n_hr_fiscal_user_id` | Fiscal User | many to one | `res.partner` | restricted by domain `lambda self: self._get_l10n_hr_fiscal_user_id_domain()` |
| `l10n_hr_operator_name` | Operator Label | single line text |  | related through path `l10n_hr_fiscal_user_id.name` |
| `l10n_hr_operator_oib` | Operator OIB | single line text |  | related through path `l10n_hr_fiscal_user_id.l10n_hr_personal_oib` |
| `l10n_hr_edi_addendum_id` | human resources electronic data interchange Addendum | one to many | `l10n_hr_edi.addendum` | not copied on duplication; inverse field `move_id` |
| `l10n_hr_invoice_sending_time` | Localization Human resources Invoice Sending Time | date and time |  | related through path `l10n_hr_edi_addendum_id.invoice_sending_time` |
| `l10n_hr_business_document_status` | Localization Human resources Business Document Status | selection |  | related through path `l10n_hr_edi_addendum_id.business_document_status` |
| `l10n_hr_business_status_reason` | Localization Human resources Business Status Reason | single line text |  | related through path `l10n_hr_edi_addendum_id.business_status_reason` |
| `l10n_hr_fiscalization_number` | Localization Human resources Fiscalization Number | single line text |  | related through path `l10n_hr_edi_addendum_id.fiscalization_number` |
| `l10n_hr_fiscalization_status` | Localization Human resources Fiscalization Status | selection |  | related through path `l10n_hr_edi_addendum_id.fiscalization_status` |
| `l10n_hr_fiscalization_error` | Localization Human resources Fiscalization Error | single line text |  | related through path `l10n_hr_edi_addendum_id.fiscalization_error` |
| `l10n_hr_fiscalization_request` | Localization Human resources Fiscalization Request | single line text |  | related through path `l10n_hr_edi_addendum_id.fiscalization_request` |
| `l10n_hr_fiscalization_channel_type` | Localization Human resources Fiscalization Channel Type | selection |  | related through path `l10n_hr_edi_addendum_id.fiscalization_channel_type` |
| `l10n_hr_payment_reported_amount` | Localization Human resources Payment Reported Amount | monetary |  | related through path `l10n_hr_edi_addendum_id.payment_reported_amount`; currency taken from `currency_id` |
| `l10n_hr_payment_unreported` | Localization Human resources Payment Unreported | boolean |  | computed by rule `_compute_l10n_hr_payment_unreported` (not stored); searchable through a search rule |
| `l10n_hr_payment_method_type` | Localization Human resources Payment Method Type | selection |  | related through path `l10n_hr_edi_addendum_id.payment_method_type` |
| `l10n_hr_mer_document_eid` | Localization Human resources Mer Document Eid | single line text |  | related through path `l10n_hr_edi_addendum_id.mer_document_eid` |
| `l10n_hr_mer_document_status` | Localization Human resources Mer Document Status | selection |  | related through path `l10n_hr_edi_addendum_id.mer_document_status` |
| `l10n_hu_payment_mode` | Payment mode | selection |  | Help: NAV expected payment mode of the invoice. |
| `l10n_hu_edi_state` | NAV 3.0 status | selection |  | indexed (btree_not_null); not copied on duplication |
| `l10n_hu_edi_batch_upload_index` | Index of invoice within a batch upload | integer |  | not copied on duplication |
| `l10n_hu_edi_attachment` | Invoice extensible markup language file | binary |  | not copied on duplication |
| `l10n_hu_edi_send_time` | Invoice upload time | date and time |  | not copied on duplication |
| `l10n_hu_edi_transaction_code` | Transaction Code | single line text |  | changes are tracked in the message thread; indexed (trigram); not copied on duplication |
| `l10n_hu_edi_messages` | Transaction messages (JavaScript Object Notation) | structured document |  | not copied on duplication |
| `l10n_hu_invoice_chain_index` | Invoice Chain Index | integer |  | not copied on duplication; Help: Index in the chain of modification invoices:                 -1 for a base invoice;                 1, 2, 3, ... for modification invoices;                 0 for rejected/cancelled invoices or if it has not yet been set. |
| `l10n_hu_edi_attachment_filename` | Invoice extensible markup language filename | single line text |  | computed by rule `_compute_l10n_hu_edi_attachment_filename` (not stored) |
| `l10n_hu_edi_message_html` | Transaction messages | rich text |  | computed by rule `_compute_message_html` (not stored) |
| `l10n_id_qris_transaction_ids` | Localization Identifier Qris Transaction | many to many | `l10n_id.qris.transaction` | visible only to groups `account.group_account_invoice` |
| `l10n_id_coretax_add_info_07` | Localization Identifier Coretax Add Info 07 | selection |  | computed by rule `_compute_l10n_id_coretax_add_info` and stored |
| `l10n_id_coretax_facility_info_07` | Localization Identifier Coretax Facility Info 07 | selection |  | computed by rule `_compute_l10n_id_coretax_facility_info` and stored |
| `l10n_id_coretax_add_info_08` | Localization Identifier Coretax Add Info 08 | selection |  | computed by rule `_compute_l10n_id_coretax_add_info` and stored |
| `l10n_id_coretax_facility_info_08` | Localization Identifier Coretax Facility Info 08 | selection |  | computed by rule `_compute_l10n_id_coretax_facility_info` and stored |
| `l10n_id_coretax_efaktur_available` | Localization Identifier Coretax Efaktur Available | boolean |  | computed by rule `_compute_l10n_id_coretax_efaktur_available` (not stored) |
| `l10n_id_coretax_document` | e-Faktur Document (Coretax) | many to one | `l10n_id_efaktur_coretax.document` | read only; not copied on duplication |
| `l10n_id_coretax_custom_doc` | Localization Identifier Coretax Custom Doc | single line text |  | Help: Additional documentation when choosing kode 07 or 08 |
| `l10n_id_coretax_custom_doc_month_year` | Custom Document Month and Year | date |  |  |
| `l10n_id_kode_transaksi` | Kode Transaksi | selection |  | computed by rule `_compute_kode_transaksi` and stored; not copied on duplication; Help: The first 2 digits of tax code |
| `l10n_in_gst_treatment` | goods and services tax Treatment | selection |  | computed by rule `_compute_l10n_in_gst_treatment` and stored; precomputed before insertion |
| `l10n_in_state_id` | Place of supply | many to one | `res.country.state` | computed by rule `_compute_l10n_in_state_id` and stored; precomputed before insertion |
| `l10n_in_gstin` | GSTIN | single line text |  |  |
| `l10n_in_shipping_bill_number` | Shipping bill number | single line text |  |  |
| `l10n_in_shipping_bill_date` | Shipping bill date | date |  |  |
| `l10n_in_shipping_port_code_id` | Port code | many to one | `l10n_in.port.code` |  |
| `l10n_in_reseller_partner_id` | Reseller | many to one | `res.partner` | restricted by domain `[["vat", "!=", false]]`; Help: Only Registered Reseller |
| `l10n_in_journal_type` | Journal Type | selection |  | related through path `journal_id.type` |
| `l10n_in_warning` | Localization In Warning | structured document |  | computed by rule `_compute_l10n_in_warning` (not stored) |
| `l10n_in_is_gst_registered_enabled` | Localization In Is Goods and services tax Registered Enabled | boolean |  | related through path `company_id.l10n_in_is_gst_registered` |
| `l10n_in_tds_deduction` | tax deducted at source Deduction | selection |  | related through path `commercial_partner_id.l10n_in_pan_entity_id.tds_deduction` |
| `l10n_in_is_withholding` | Is Indian tax deducted at source Entry | boolean |  | not copied on duplication; Help: Technical field to identify Indian withholding entry |
| `l10n_in_withholding_ref_move_id` | Indian tax deducted at source Ref Move | many to one | `account.move` | read only; indexed (btree_not_null); not copied on duplication; Help: Reference move for withholding entry |
| `l10n_in_withholding_ref_payment_id` | Indian tax deducted at source Ref Payment | many to one | `account.payment` | read only; indexed (btree_not_null); not copied on duplication; Help: Reference Payment for withholding entry |
| `l10n_in_withhold_move_ids` | Indian tax deducted at source Entries | one to many | `account.move` | inverse field `l10n_in_withholding_ref_move_id` |
| `l10n_in_withholding_line_ids` | Indian tax deducted at source Lines | one to many | `account.move.line` | computed by rule `_compute_l10n_in_withholding_line_ids` (not stored); inverse field `move_id` |
| `l10n_in_total_withholding_amount` | Total Indian tax deducted at source Amount | monetary |  | computed by rule `_compute_l10n_in_total_withholding_amount` (not stored); Help: Total withholding amount for the move |
| `l10n_in_display_higher_tcs_button` | Display higher tax collected at source button | boolean |  | computed by rule `_compute_l10n_in_display_higher_tcs_button` (not stored) |
| `l10n_in_tds_feature_enabled` | Localization In Tax deducted at source Feature Enabled | boolean |  | related through path `company_id.l10n_in_tds_feature` |
| `l10n_in_tcs_feature_enabled` | Localization In Tax collected at source Feature Enabled | boolean |  | related through path `company_id.l10n_in_tcs_feature` |
| `l10n_in_partner_gstin_status` | goods and services tax Status | boolean |  | computed by rule `_compute_l10n_in_partner_gstin_status_and_date` (not stored) |
| `l10n_in_show_gstin_status` | Localization In Show Gstin Status | boolean |  | computed by rule `_compute_l10n_in_show_gstin_status` (not stored) |
| `l10n_in_gstin_verified_date` | Localization In Gstin Verified Date | date |  | computed by rule `_compute_l10n_in_partner_gstin_status_and_date` (not stored) |
| `l10n_in_edi_status` | India E-Invoice Status | selection |  | read only; changes are tracked in the message thread; not copied on duplication |
| `l10n_in_edi_attachment_id` | E-Invoice(IN) Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='E-Invoice(IN) Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_in_edi_attachment_id', 'l10n_in_edi_attachment_file'), depends=['l10n_in_edi_attachment_file'])` (not stored) |
| `l10n_in_edi_attachment_file` | E-Invoice(IN) File | binary |  | not copied on duplication |
| `l10n_in_edi_cancel_reason` | E-Invoice(IN) Cancel Reason | selection |  | not copied on duplication |
| `l10n_in_edi_cancel_remarks` | E-Invoice(IN) Cancel Remarks | single line text |  | not copied on duplication |
| `l10n_in_edi_content` | E-Invoice(IN) Content | binary |  | computed by rule `_compute_l10n_in_edi_content` (not stored) |
| `l10n_in_edi_error` | Localization In Electronic data interchange Error | rich text |  | read only; not copied on duplication |
| `l10n_in_ewaybill_ids` | E-Waybill | one to many | `l10n.in.ewaybill` | read only; inverse field `account_move_id` |
| `l10n_in_ewaybill_name` | Indian Ewaybill Number | single line text |  | computed by rule `_compute_l10n_in_ewaybill_details` (not stored) |
| `l10n_in_ewaybill_expiry_date` | Localization In Electronic waybill Expiry Date | date and time |  | computed by rule `_compute_l10n_in_ewaybill_details` (not stored) |
| `l10n_in_ewaybill_feature_enabled` | E-Waybill Feature Enabled | boolean |  | related through path `company_id.l10n_in_ewaybill_feature` |
| `l10n_it_edi_state` | SDI State | selection |  | writable through an inverse rule; changes are tracked in the message thread; not copied on duplication; Help: This state is updated by default, but you can force the value. |
| `l10n_it_edi_header` | Localization It Electronic data interchange Header | rich text |  | read only; not copied on duplication; Help: User description of the current state, with hints to make the flow progress |
| `l10n_it_edi_transaction` | FatturaPA Transaction | single line text |  | not copied on duplication |
| `l10n_it_edi_attachment_file` | Localization It Electronic data interchange Attachment File | binary |  | not copied on duplication |
| `l10n_it_edi_attachment_name` | FatturaPA Attachment | single line text |  |  |
| `l10n_it_edi_proxy_mode` | Localization It Electronic data interchange Proxy Mode | selection |  | related through path `company_id.l10n_it_edi_proxy_user_id.edi_mode` |
| `l10n_it_edi_button_label` | Localization It Electronic data interchange Button Label | single line text |  | computed by rule `_compute_l10n_it_edi_button_label` (not stored) |
| `l10n_it_edi_is_self_invoice` | Localization It Electronic data interchange Is Self Invoice | boolean |  | computed by rule `_compute_l10n_it_edi_is_self_invoice` (not stored) |
| `l10n_it_stamp_duty` | Dati Bollo | float |  |  |
| `l10n_it_ddt_id` | transport document | many to one | `l10n_it.ddt` | not copied on duplication |
| `l10n_it_origin_document_type` | Origin Document Type | selection |  | not copied on duplication |
| `l10n_it_origin_document_name` | Origin Document Name | single line text |  | not copied on duplication |
| `l10n_it_origin_document_date` | Origin Document Date | date |  | not copied on duplication |
| `l10n_it_cig` | CIG | single line text |  | not copied on duplication; Help: Tender Unique Identifier |
| `l10n_it_cup` | CUP | single line text |  | not copied on duplication; Help: Public Investment Unique Identifier |
| `l10n_it_partner_pa` | Localization It Partner Pa | boolean |  | computed by rule `_compute_l10n_it_partner_pa` (not stored) |
| `l10n_it_partner_is_public_administration` | Localization It Partner Is Public Administration | boolean |  | computed by rule `_compute_l10n_it_partner_is_public_administration` (not stored); Help: Only partners that have a 6-chars long l10n_it_pa_index actually belong to the Public Administration |
| `l10n_it_payment_method` | Localization It Payment Method | selection |  | computed by rule `_compute_l10n_it_payment_method` and stored |
| `l10n_it_document_type` | Localization It Document Type | many to one | `l10n_it.document.type` | computed by rule `_compute_l10n_it_document_type` and stored; not copied on duplication |
| `l10n_it_edi_doi_date` | Date on which Declaration of Intent is applied | date |  | computed by rule `_compute_l10n_it_edi_doi_date` (not stored) |
| `l10n_it_edi_doi_use` | Use Declaration of Intent | boolean |  | computed by rule `_compute_l10n_it_edi_doi_use` (not stored) |
| `l10n_it_edi_doi_id` | Declaration of Intent | many to one | `l10n_it_edi_doi.declaration_of_intent` | computed by rule `_compute_l10n_it_edi_doi_id` and stored; precomputed before insertion |
| `l10n_it_edi_doi_amount` | Declaration of Intent Amount | monetary |  | read only; computed by rule `_compute_l10n_it_edi_doi_amount` and stored; Help: Total amount of sales under the Declaration of Intent of this document |
| `l10n_it_edi_doi_warning` | Declaration of Intent Threshold Warning | multi line text |  | computed by rule `_compute_l10n_it_edi_doi_warning` (not stored) |
| `l10n_it_ddt_ids` | Localization It Transport document | many to many | `stock.picking` | computed by rule `_compute_ddt_ids` (not stored) |
| `l10n_it_ddt_count` | Localization It Transport document Count | integer |  | computed by rule `_compute_ddt_ids` (not stored) |
| `l10n_jo_edi_uuid` | Invoice UUID | single line text |  | computed by rule `_compute_l10n_jo_edi_uuid` and stored; not copied on duplication |
| `l10n_jo_edi_qr` | quick response | single line text |  | not copied on duplication |
| `l10n_jo_edi_is_needed` | Localization Jo Electronic data interchange Is Needed | boolean |  | computed by rule `_compute_l10n_jo_edi_is_needed` (not stored); Help: Jordan: technical field to determine if this invoice is eligible to be e-invoiced. |
| `l10n_jo_edi_state` | JoFotara State | selection |  | changes are tracked in the message thread; not copied on duplication |
| `l10n_jo_edi_error` | JoFotara Error | multi line text |  | read only; not copied on duplication; Help: Jordan: Error details. |
| `l10n_jo_edi_computed_xml` | Jordan E-Invoice computed extensible markup language File | binary |  | computed by rule `_compute_l10n_jo_edi_computed_xml` (not stored); Help: Jordan: technical field computing e-invoice XML data, useful at submission failure scenarios. |
| `l10n_jo_edi_xml_attachment_file` | Jordan E-Invoice extensible markup language File | binary |  | not copied on duplication; Help: Jordan: technical field holding the e-invoice XML data. |
| `l10n_jo_edi_xml_attachment_id` | Jordan E-Invoice extensible markup language | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='Jordan E-Invoice XML', compute=lambda self: self._compute_linked_attachment_id('l10n_jo_edi_xml_attachment_id', 'l10n_jo_edi_xml_attachment_file'), depends=['l10n_jo_edi_xml_attachment_file'], help='Jordan: e-invoice XML.')` (not stored); Help: Jordan: e-invoice XML. |
| `l10n_jo_edi_invoice_type` | Invoice Type | selection |  | computed by rule `_compute_l10n_jo_edi_invoice_type` and stored; changes are tracked in the message thread; precomputed before insertion; Help: Invoice Types as per the Income and Sales Tax Department for JoFotara |
| `l10n_ke_wh_certificate_number` | Withholding Certificate Number | single line text |  | Help: Customer withholding certificate number |
| `l10n_ke_wh_certificate_date` | Date of Certificate | date |  |  |
| `l10n_ke_cu_datetime` | CU Signing Date and Time | date and time |  | not copied on duplication |
| `l10n_ke_cu_serial_number` | CU Serial Number | single line text |  | not copied on duplication |
| `l10n_ke_cu_invoice_number` | CU Invoice Number | single line text |  | not copied on duplication |
| `l10n_ke_cu_qrcode` | CU quick response Code | single line text |  | not copied on duplication |
| `l10n_ke_cu_show_send_button` | Show Send to Tremol button | boolean |  | computed by rule `_compute_l10n_ke_cu_show_send_button` (not stored) |
| `MyInvois Documents` | MyInvois Documents | many to many | `myinvois.document` | not copied on duplication; association table `myinvois_document_invoice_rel` |
| `l10n_my_edi_display_tax_exemption_reason` | Display Tax Exemption Reason | boolean |  | computed by rule `_compute_l10n_my_edi_display_tax_exemption_reason` (not stored) |
| `l10n_my_invoice_need_edi` | Localization My Invoice Need Electronic data interchange | boolean |  | computed by rule `_compute_l10n_my_invoice_need_edi` (not stored) |
| `l10n_my_edi_state` | MyInvois State | selection |  | computed by rule `_compute_l10n_my_edi_state` and stored; changes are tracked in the message thread; Help: State of this document on the MyInvois portal. A document awaiting validation will be automatically updated once the validation status is available. |
| `l10n_my_edi_exemption_reason` | Tax Exemption Reason | single line text |  | Help: Buyer’s sales tax exemption certificate number, special exemption as per gazette orders, etc. Only applicable if you are using a tax with a type 'Exempt'. |
| `l10n_my_edi_custom_form_reference` | Customs Form Reference Number | single line text |  | Help: Reference Number of Customs Form No.1, 9, etc. |
| `l10n_pl_vat_b_spv` | B_SPV | boolean |  | Help: Transfer of a single-purpose voucher effected by a taxable person acting on his/its own behalf |
| `l10n_pl_vat_b_spv_dostawa` | B_SPV_Dostawa | boolean |  | Help: Supply of goods and/or services covered by a single-purpose voucher to a taxpayer |
| `l10n_pl_vat_b_mpv_prowizja` | B_MPV_Prowizja | boolean |  | Help: Supply of agency and other services pertaining to the transfer of a single-purpose voucher |
| `l10n_pl_edi_status` | KSeF Status | selection |  | read only; not copied on duplication |
| `l10n_pl_edi_ref` | KSeF Reference Number | single line text |  | read only; not copied on duplication |
| `l10n_pl_edi_register` | Localization Pl Electronic data interchange Register | boolean |  | related through path `company_id.l10n_pl_edi_register` |
| `l10n_pl_edi_header` | Localization Pl Electronic data interchange Header | rich text |  | read only; not copied on duplication; Help: User description of the current state, with hints to make the flow progress |
| `l10n_pl_edi_number` | KSeF Number | single line text |  | read only; indexed; not copied on duplication |
| `l10n_pl_edi_session_id` | KSeF Session Number used for sending | single line text |  | read only; not copied on duplication |
| `l10n_pl_edi_attachment_file` | Localization Pl Electronic data interchange Attachment File | binary |  | not copied on duplication |
| `l10n_pl_edi_attachment_id` | KSeF Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='KSeF Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_pl_edi_attachment_id', 'l10n_pl_edi_attachment_file'), depends=['l10n_pl_edi_attachment_file'])` (not stored) |
| `l10n_pl_edi_upo_file` | Localization Pl Electronic data interchange Upo File | binary |  | not copied on duplication |
| `l10n_pl_edi_upo_id` | UPO Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='UPO Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_pl_edi_upo_id', 'l10n_pl_edi_upo_file'), depends=['l10n_pl_edi_upo_file'])` (not stored) |
| `l10n_ro_edi_document_ids` | Localization Ro Electronic data interchange Document | one to many | `l10n_ro_edi.document` | inverse field `invoice_id` |
| `l10n_ro_edi_state` | E-Factura Status | selection |  | computed by rule `_compute_l10n_ro_edi_state` and stored; Help: - Not indexed: Invoice index was not received on time due to a server timeout                 - Sent: Successfully sent to the SPV, waiting for validation                 - Validated: Sent & validated by the SPV                 - Refused: Validation error from the SPV |
| `l10n_ro_edi_index` | E-Factura Index | single line text |  | read only; not copied on duplication |
| `l10n_rs_turnover_date` | Turnover Date | date |  |  |
| `l10n_rs_edi_uuid` | RS Invoice UUID | single line text |  | computed by rule `_compute_l10n_rs_edi_uuid` and stored; not copied on duplication; Help: Unique Identifier for an invoice used as request id |
| `l10n_rs_edi_is_eligible` | Localization Rs Electronic data interchange Is Eligible | boolean |  | computed by rule `_compute_l10n_rs_edi_is_eligible` and stored; Help: Technical field to determine if this invoice is eligible to be e-invoiced. |
| `l10n_rs_edi_attachment_file` | Serbian E-Invoice extensible markup language File | binary |  | not copied on duplication; Help: Serbia: technical field holding the e-invoice XML data. |
| `l10n_rs_edi_attachment_id` | eFaktura extensible markup language Attachment | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', string='eFaktura XML Attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_rs_edi_attachment_id', 'l10n_rs_edi_attachment_file'), depends=['l10n_rs_edi_attachment_file'])` (not stored) |
| `l10n_rs_edi_state` | Serbia E-Invoice state | selection |  | read only; changes are tracked in the message thread; not copied on duplication |
| `l10n_rs_edi_error` | Serbia E-Invoice error | multi line text |  | read only; not copied on duplication |
| `l10n_rs_tax_date_obligations_code` | Tax Date Obligations | selection |  | computed by rule `_compute_l10n_rs_tax_date_obligations_code` and stored |
| `l10n_rs_edi_invoice` | Invoice Id | single line text |  | not copied on duplication |
| `l10n_rs_edi_sales_invoice` | Sales Invoice Id | single line text |  | not copied on duplication |
| `l10n_rs_edi_purchase_invoice` | Purchase Invoice Id | single line text |  | not copied on duplication |
| `l10n_sa_qr_code_str` | Zatka quick response Code | single line text |  | computed by rule `_compute_qr_code_str` (not stored) |
| `l10n_sa_show_reason` | Localization Sa Show Reason | boolean |  | computed by rule `_compute_show_l10n_sa_reason` (not stored) |
| `l10n_sa_reason` | ZATCA Reason | selection |  | not copied on duplication |
| `l10n_sa_confirmation_datetime` | ZATCA Issue Date | date and time |  | read only; not copied on duplication; Help: Date on which the invoice is generated as final document (after securing all internal approvals). |
| `l10n_sa_uuid` | Document UUID (SA) | single line text |  | not copied on duplication; Help: Universally unique identifier of the Invoice |
| `l10n_sa_invoice_signature` | Unsigned extensible markup language Signature | single line text |  | not copied on duplication |
| `l10n_sa_chain_index` | ZATCA chain index | integer |  | read only; not copied on duplication; Help: Invoice index in chain, set if and only if an in-chain XML was submitted and did not error |
| `l10n_sa_edi_chain_head_id` | ZATCA chain stopping move | many to one | `account.move` | read only; not copied on duplication; Help: Technical field to know if the chain has been stopped by a previous invoice |
| `l10n_sg_permit_number` | Permit No. | single line text |  |  |
| `l10n_sg_permit_number_date` | Date of permit number | date |  |  |
| `l10n_tr_nilvera_uuid` | Nilvera Document UUID | single line text |  | read only; not copied on duplication; Help: Universally unique identifier of the Invoice |
| `l10n_tr_nilvera_send_status` | Nilvera Status | selection |  | read only; default `not_sent`; not copied on duplication |
| `l10n_tr_gib_invoice_scenario` | Invoice Scenario | selection |  | default `TEMELFATURA`; Help: The scenario of the invoice to be sent to GİB. |
| `l10n_tr_gib_invoice_type` | GIB Invoice Type | selection |  | computed by rule `_compute_l10n_tr_gib_invoice_type` and stored; default `SATIS`; Help: The type of invoice to be sent to GİB. |
| `l10n_tr_is_export_invoice` | Is GIB Export | boolean |  |  |
| `l10n_tr_shipping_type` | Shipping Method | selection |  | Help: The type of shipping. |
| `l10n_tr_exemption_code_id` | Exemption Reason | many to one | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | computed by rule `_compute_l10n_tr_exemption_code_id` and stored; Help: The exception reason of the invoice. |
| `l10n_tr_exemption_code_domain_list` | Localization Tr Exemption Code Domain List | binary |  | computed by rule `_compute_l10n_tr_exemption_code_domain_list` (not stored) |
| `l10n_tr_nilvera_customer_status` | Partner Nilvera Status | selection |  | related through path `partner_id.l10n_tr_nilvera_customer_status` |
| `l10n_tw_edi_file_id` | Localization Tw Electronic data interchange File | many to one | `ir.attachment` | computed by rule `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_tw_edi_file_id', 'l10n_tw_edi_file'), depends=['l10n_tw_edi_file'], copy=False, export_string_translation=False)` (not stored); not copied on duplication |
| `l10n_tw_edi_file` | Ecpay JavaScript Object Notation File | binary |  | read only; not copied on duplication |
| `l10n_tw_edi_ecpay_invoice_id` | Ecpay Invoice Number | single line text |  | read only; not copied on duplication |
| `l10n_tw_edi_related_number` | Related Number | single line text |  | read only; not copied on duplication |
| `l10n_tw_edi_state` | Invoice Status | selection |  | read only; changes are tracked in the message thread; not copied on duplication |
| `l10n_tw_edi_love_code` | Love Code | single line text |  | computed by rule `_compute_love_code` and stored |
| `l10n_tw_edi_is_print` | Get Printed Version | boolean |  | computed by rule `_compute_is_print` and stored |
| `l10n_tw_edi_carrier_type` | Carrier Type | selection |  | computed by rule `_compute_carrier_info` and stored; not copied on duplication; Help: - Citizen Digital Certificate: The carrier number format is 2 capital letters following 14 digits.     - Mobile Barcode: The carrier number format is / following 7 alphanumeric or +-. string.     - EasyCard or iPass: The carrier number is the card hidden code, the carrier number 2 is the card visible code. |
| `l10n_tw_edi_carrier_number` | Carrier Number | single line text |  | computed by rule `_compute_carrier_info` and stored; not copied on duplication |
| `l10n_tw_edi_carrier_number_2` | Carrier Number 2 | single line text |  | computed by rule `_compute_carrier_info` and stored; not copied on duplication |
| `l10n_tw_edi_invoice_type` | Ecpay Invoice Type | selection |  | computed by rule `_compute_l10n_tw_edi_invoice_type` and stored; not copied on duplication |
| `l10n_tw_edi_clearance_mark` | Clearance Mark | selection |  | not copied on duplication |
| `l10n_tw_edi_zero_tax_rate_reason` | Zero Tax Rate Reason | selection |  | not copied on duplication |
| `l10n_tw_edi_is_zero_tax_rate` | Is Zero Tax Rate | boolean |  | computed by rule `_compute_l10n_tw_edi_is_zero_tax_rate` (not stored); not copied on duplication |
| `l10n_tw_edi_invoice_create_date` | Creation Date | date and time |  | read only; not copied on duplication |
| `l10n_tw_edi_refund_state` | Refund State | selection |  | read only; not copied on duplication |
| `l10n_tw_edi_refund_agreement_type` | Refund invoice Agreement Type | selection |  | not copied on duplication |
| `l10n_tw_edi_allowance_notify_way` | Allowance Notify Way | selection |  | not copied on duplication |
| `l10n_tw_edi_invalidate_reason` | Invalidate Reason | single line text |  | read only; not copied on duplication |
| `l10n_tw_edi_refund_invoice_number` | Refund Invoice Number | single line text |  | read only; not copied on duplication |
| `l10n_tw_edi_is_b2b` | Is business to business | boolean |  | computed by rule `_compute_l10n_tw_edi_is_b2b` (not stored) |
| `l10n_vn_e_invoice_number` | eInvoice Number | single line text |  | not copied on duplication; Help: Electronic Invoicing number. |
| `l10n_vn_edi_invoice_state` | Sinvoice Status | selection |  | computed by rule `_compute_l10n_vn_edi_invoice_state` and stored; not copied on duplication |
| `l10n_vn_edi_invoice_transaction_id` | SInvoice Transaction identifier | single line text |  | not copied on duplication; Help: Technical field to store the transaction ID if needed |
| `l10n_vn_edi_invoice_symbol` | Invoice Symbol | many to one | `l10n_vn_edi_viettel.sinvoice.symbol` | computed by rule `_compute_l10n_vn_edi_invoice_symbol` and stored |
| `l10n_vn_edi_invoice_number` | SInvoice Number | single line text |  | read only; not copied on duplication; Help: Invoice Number as appearing on SInvoice. |
| `l10n_vn_edi_reservation_code` | Secret Code | single line text |  | read only; not copied on duplication; Help: Secret code that can be used by a customer to lookup an invoice on SInvoice. |
| `l10n_vn_edi_issue_date` | Issue Date | date and time |  | read only; not copied on duplication; Help: Date of issue of the invoice on the e-invoicing system. |
| `l10n_vn_edi_sinvoice_file_id` | Localization Vn Electronic data interchange Sinvoice File | many to one | `ir.attachment` | read only; computed by rule `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_file_id', 'l10n_vn_edi_sinvoice_file'), depends=['l10n_vn_edi_sinvoice_file'], copy=False, readonly=True, export_string_translation=False)` (not stored); not copied on duplication |
| `l10n_vn_edi_sinvoice_file` | SInvoice json File | binary |  | read only; not copied on duplication |
| `l10n_vn_edi_sinvoice_xml_file_id` | Localization Vn Electronic data interchange Sinvoice Extensible markup language File | many to one | `ir.attachment` | read only; computed by rule `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_xml_file_id', 'l10n_vn_edi_sinvoice_xml_file'), depends=['l10n_vn_edi_sinvoice_xml_file'], copy=False, readonly=True, export_string_translation=False)` (not stored); not copied on duplication |
| `l10n_vn_edi_sinvoice_xml_file` | SInvoice xml File | binary |  | read only; not copied on duplication |
| `l10n_vn_edi_sinvoice_pdf_file_id` | Localization Vn Electronic data interchange Sinvoice Portable Document Format File | many to one | `ir.attachment` | read only; computed by rule `fields.Many2one(comodel_name='ir.attachment', compute=lambda self: self._compute_linked_attachment_id('l10n_vn_edi_sinvoice_pdf_file_id', 'l10n_vn_edi_sinvoice_pdf_file'), depends=['l10n_vn_edi_sinvoice_pdf_file'], copy=False, readonly=True, export_string_translation=False)` (not stored); not copied on duplication |
| `l10n_vn_edi_sinvoice_pdf_file` | SInvoice pdf File | binary |  | read only; not copied on duplication |
| `l10n_vn_edi_agreement_document_name` | Agreement Name | single line text |  | not copied on duplication |
| `l10n_vn_edi_agreement_document_date` | Agreement Date | date and time |  | not copied on duplication |
| `l10n_vn_edi_adjustment_type` | Adjustment type | selection |  | not copied on duplication |
| `l10n_vn_edi_replacement_origin_id` | Replacement of | many to one | `account.move` | read only; not copied on duplication; must belong to the same company |
| `l10n_vn_edi_reversed_entry_invoice_number` | Revered Entry SInvoice Number | single line text |  | related through path `reversed_entry_id.l10n_vn_edi_invoice_number` |
| `wip_production_ids` | Relevant work in progress manufacturing orders | many to many | `mrp.production` | not copied on duplication; association table `wip_move_production_rel`; Help: The MOs that this WIP entry was based on. Expected to be set at time of WIP entry creation. |
| `wip_production_count` | Manufacturing Orders Count | integer |  | computed by rule `_compute_wip_production_count` (not stored) |
| `landed_costs_ids` | Landed Costs | one to many | `stock.landed.cost` | inverse field `vendor_bill_id` |
| `landed_costs_visible` | Landed Costs Visible | boolean |  | computed by rule `_compute_landed_costs_visible` (not stored) |
| `timesheet_ids` | Timesheets | one to many | `account.analytic.line` | read only; not copied on duplication; inverse field `timesheet_invoice_id` |
| `timesheet_count` | Number of timesheets | integer |  | computed by rule `_compute_timesheet_count` (not stored) |
| `timesheet_encode_uom_id` | Timesheet Encode Unit of measure | many to one | `uom.uom` | related through path `company_id.timesheet_encode_uom_id` |
| `timesheet_total_duration` | Timesheet Total Duration | integer |  | computed by rule `_compute_timesheet_total_duration` (not stored); Help: Total recorded duration, expressed in the encoding UoM, and rounded to the unit |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `posted` | Posted |
| `cancel` | Cancelled |

### `move_type` (Type)

| Value | Label |
|---|---|
| `entry` | Journal Entry |
| `out_invoice` | Customer Invoice |
| `out_refund` | Customer Credit Note |
| `in_invoice` | Vendor Bill |
| `in_refund` | Vendor Credit Note |
| `out_receipt` | Sales Receipt |
| `in_receipt` | Purchase Receipt |

### `auto_post` (Auto-post)

| Value | Label |
|---|---|
| `no` | No |
| `at_date` | At Date |
| `monthly` | Monthly |
| `quarterly` | Quarterly |
| `yearly` | Yearly |

### `move_sent_values` (Sent)

| Value | Label |
|---|---|
| `sent` | Sent |
| `not_sent` | Not Sent |

### `edi_state` (Electronic invoicing)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `to_cancel` | To Cancel |
| `cancelled` | Cancelled |

### `edi_blocking_level` (Electronic data interchange Blocking Level)

| Value | Label |
|---|---|
| `info` | Info |
| `warning` | Warning |
| `error` | Error |

### `peppol_move_state` (E-Invoicing Status)

| Value | Label |
|---|---|
| `ready` | Ready to send |
| `to_send` | Queued |
| `skipped` | Skipped |
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `AB` | Received |
| `AP` | Approved |
| `RE` | Rejected |

### `l10n_bg_exemption_reason` (Exemption reason (BG))

| Value | Label |
|---|---|
| `01` | 01 - A delivery under Part 1 of Appendix 2 of LVAT |
| `02` | 02 - A delivery under Part 2 of Appendix 2 of LVAT |
| `03` | 03 - Import under Appendix 3 of VAT act |

### `nemhandel_move_state` (Nemhandel status)

| Value | Label |
|---|---|
| `ready` | Ready to send |
| `to_send` | Queued |
| `processing` | Pending Reception |
| `done` | Done |
| `error` | Error |
| `BusinessAccept` | Approved |
| `BusinessReject` | Rejected |

### `l10n_es_edi_facturae_reason_code` (Spanish Facturae electronic data interchange Reason Code)

| Value | Label |
|---|---|
| `01` | Invoice number |
| `02` | Invoice serial number |
| `03` | Issue date |
| `04` | Name and surnames/Corporate name - Issuer (Sender) |
| `05` | Name and surnames/Corporate name - Receiver |
| `06` | Issuer's Tax Identification Number |
| `07` | Receiver's Tax Identification Number |
| `08` | Issuer's address |
| `09` | Receiver's address |
| `10` | Item line |
| `11` | Applicable Tax Rate |
| `12` | Applicable Tax Amount |
| `13` | Applicable Date/Period |
| `14` | Invoice Class |
| `15` | Legal literals |
| `16` | Taxable Base |
| `80` | Calculation of tax outputs |
| `81` | Calculation of tax inputs |
| `82` | Taxable Base modified due to return of packages and packaging materials |
| `83` | Taxable Base modified due to discounts and rebates |
| `84` | Taxable Base modified due to firm court ruling or administrative decision |
| `85` | Taxable Base modified due to unpaid outputs where there is a judgement opening insolvency proceedings |

### `l10n_es_payment_means` (Payment Means)

| Value | Label |
|---|---|
| `01` | In cash |
| `02` | Direct debit |
| `03` | Receipt |
| `04` | Credit transfer |
| `05` | Accepted bill of exchange |
| `06` | Documentary credit |
| `07` | Contract award |
| `08` | Bill of exchange |
| `09` | Transferable promissory note |
| `10` | Non transferable promissory note |
| `11` | Cheque |
| `12` | Open account reimbursement |
| `13` | Special payment |
| `14` | Set-off by reciprocal credits |
| `15` | Payment by postgiro |
| `16` | Certified cheque |
| `17` | Banker’s draft |
| `18` | Cash on delivery |
| `19` | Payment by card |

### `l10n_es_tbai_state` (TicketBAI status)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

### `l10n_es_edi_verifactu_state` (Veri*Factu Status)

| Value | Label |
|---|---|
| `rejected` | Rejected |
| `registered_with_errors` | Registered with Errors |
| `accepted` | Accepted |
| `cancelled` | Cancelled |

### `l10n_es_edi_verifactu_refund_reason` (Veri*Factu Refund Reason)

| Value | Label |
|---|---|
| `R1` | R1: Art 80.1 and 80.2 and error of law |
| `R2` | R2: Art. 80.3 |
| `R3` | R3: Art. 80.4 |
| `R4` | R4: Rest |
| `R5` | R5: Corrective invoices concerning simplified invoices |

### `pdp_ppf_move_state` (PPF Invoice Status)

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

### `pdp_ppf_lifecycle_state` (PPF Lifeycle Status)

| Value | Label |
|---|---|
| `in_progress` | In Progress |
| `sent` | Sent |
| `done` | Done |
| `error` | Error |

### `l10n_fr_pdp_flow_10_report_type` (Localization Fr Pdp Flow 10 Report Type)

| Value | Label |
|---|---|
| `transaction` | Transaction |
| `payment` | Payment |

### `l10n_fr_pdp_flow_10_operation_type` (Localization Fr Pdp Flow 10 Operation Type)

| Value | Label |
|---|---|
| `sale` | Sale |
| `purchase` | Purchase |

### `l10n_gr_edi_state` (myDATA Status)

| Value | Label |
|---|---|
| `invoice_sent` | Invoice sent |
| `bill_fetched` | Expense classification ready to send |
| `bill_sent` | Expense classification sent |
| `invoice_pending` | Invoice submission pending |

### `l10n_hr_process_type` (Business Process Type)

| Value | Label |
|---|---|
| `P1` | P1: Issuing invoices for deliveries of goods and services according to purchase orders, based on contracts |
| `P2` | P2: Periodic invoicing for deliveries of goods and services based on contracts |
| `P3` | P3: Issuing invoices for delivery according to an independent purchase order |
| `P4` | P4: Prepayment (advance payment) |
| `P5` | P5: Payment on the spot (Sport payment) |
| `P6` | P6: Payment before delivery, based on purchase order |
| `P7` | P7: Issuing invoices with references to the delivery note |
| `P8` | P8: Issuing invoices with references to the shipping and receipt notes |
| `P9` | P9: Credits or invoices with negative amounts, issued for various reasons, including empty returns packaging |
| `P10` | P10: Issuing a corrective invoice (reversal/correction of invoice) |
| `P11` | P11: Issuing partial and final invoices |
| `P12` | P12: Self-issuance of invoice |
| `P99` | P99: Customer-defined process |

### `l10n_hu_payment_mode` (Payment mode)

| Value | Label |
|---|---|
| `TRANSFER` | Transfer |
| `CASH` | Cash |
| `CARD` | Credit/debit card |
| `VOUCHER` | Voucher |
| `OTHER` | Other |

### `l10n_hu_edi_state` (NAV 3.0 status)

| Value | Label |
|---|---|
| `sent` | Sent, waiting for response |
| `send_timeout` | Timeout when sending |
| `confirmed` | Confirmed |
| `confirmed_warning` | Confirmed with warnings |
| `rejected` | Rejected |
| `cancel_sent` | Cancellation request sent |
| `cancel_timeout` | Timeout when requesting cancellation |
| `cancel_pending` | Cancellation request pending |
| `cancelled` | Cancelled |

### `l10n_id_coretax_add_info_07` (Localization Identifier Coretax Add Info 07)

| Value | Label |
|---|---|
| `TD.00501` | 1 - untuk Kawasan Bebas |
| `TD.00502` | 2 - untuk Tempat Penimbunan Berikat |
| `TD.00503` | 3 - untuk Hibah dan Bantuan Luar Negeri |
| `TD.00504` | 4 - untuk Avtur |
| `TD.00505` | 5 - untuk Lainnya |
| `TD.00506` | 6 - untuk Kontraktor Perjanjian Karya Pengusahaan Pertambangan Batubara Generasi I |
| `TD.00507` | 7 - untuk Penyerahan bahan bakar minyak untuk Kapal Angkutan Laut Luar Negeri |
| `TD.00508` | 8 - untuk Penyerahan jasa kena pajak terkait alat angkutan tertentu |
| `TD.00509` | 9 - untuk Penyerahan BKP Tertentu di KEK |
| `TD.00510` | 10 - untuk BKP tertentu yang bersifat strategis berupa anode slime |
| `TD.00511` | 11 - untuk Penyerahan alat angkutan tertentu dan/atau Jasa Kena Pajak terkait alat angkutan tertentu |
| `TD.00512` | 12 - untuk Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 27 Tahun 2017 |
| `TD.00513` | 13 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00514` | 14 - Penyerahan Jasa Sewa Ruangan atau Bangunan Kepada Pedagang Eceran yang Ditanggung Pemerintah Tahun Anggaran 2021 |
| `TD.00515` | 15 - Penyerahan Barang dan Jasa Dalam Rangka Penanganan Pandemi COVID-19 (PMK 239/PMK. 03/2020) |
| `TD.00516` | 16 - Insentif PMK-103/PMK.010/2021 berupa PPN atas Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2021 |
| `TD.00517` | 17 - Kawasan Ekonomi Khusus PP nomor 40 Tahun 2021 |
| `TD.00518` | 18 - Kawasan Bebas PP nomor 41 Tahun 2021 |
| `TD.00519` | 19 - Penyerahan Rumah Tapak dan Unit Hunian Rumah Susun yang Ditanggung Pemerintah Tahun Anggaran 2022 |
| `TD.00520` | 20 - PPN Ditanggung Pemerintah dalam rangka Penanganan Pandemi Corona Virus |
| `TD.00521` | 21 - Penyerahan kepada Kontraktor Kerja Sama Migas yang mengikuti ketentuan Peraturan Pemerintah Nomor 53 Tahun 2017 |
| `TD.00522` | 22 - BKP strategis tertentu dalam bentuk anode slime dan emas butiran |
| `TD.00523` | 23 - untuk penyerahan kertas koran dan/atau majalah |
| `TD.00524` | 24 - PPN Ditanggung Pemerintah |
| `TD.00525` | 25 - BKP dan JKP tertentu |
| `TD.00526` | 26 - Penyerahan BKP dan JKP di Ibu Kota Negara baru |
| `TD.00527` | 27 - Penyerahan kendaraan listrik berbasis baterai |
| `TD.00528` | 28 - Insentif Tambahan Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00529` | 29 - PPN atas Penyerahan Hewan Khusus Tertentu Berupa Kuda serta Perlengkapan Pendukungnya Pemerintah Tahun Anggaran 2025 |
| `TD.00530` | 30 - PPN atas Penyerahan Bekal Khusus Operasi Tertentu Yang Ditanggung Pemerintah Tahun Anggaran 2025 |
| `TD.00531` | 31 - Penyerahan Rumah Tapak dan Satuan Rumah Susun Rumah Susun Ditanggung Pemerintah Tahun Anggaran 2026 |

### `l10n_id_coretax_facility_info_07` (Localization Identifier Coretax Facility Info 07)

| Value | Label |
|---|---|
| `TD.01101` | 1 - Pajak Pertambahan Nilai Tidak Dipungut berdasarkan PP Nomor 10 Tahun 2012 |
| `TD.01102` | 2 - Pajak Pertambahan Nilai atau Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah tidak dipungut |
| `TD.01103` | 3 - Pajak Pertambahan Nilai dan Pajak Penjualan atas Barang Mewah Tidak Dipungut |
| `TD.01104` | 4 - Pajak Pertambahan Nilai Tidak Dipungut Sesuai PP Nomor 71 Tahun 2012 |
| `TD.01105` | 5 - (Tidak ada Cap) |
| `TD.01106` | 6 - PPN dan/atau PPnBM tidak dipungut berdasarkan PMK No. 194/PMK.03/2012 |
| `TD.01107` | 7 - PPN Tidak Dipungut Berdasarkan PP Nomor 15 Tahun 2015 |
| `TD.01108` | 8 - PPN Tidak Dipungut Berdasarkan PP Nomor 69 Tahun 2015 |
| `TD.01109` | 9 - PPN Tidak Dipungut Berdasarkan PP Nomor 96 Tahun 2015 |
| `TD.01110` | 10 - PPN Tidak Dipungut Berdasarkan PP Nomor 106 Tahun 2015 |
| `TD.01111` | 11 - PPN Tidak Dipungut Sesuai PP Nomor 50 Tahun 2019 |
| `TD.01112` | 12 - PPN atau PPN dan PPnBM Tidak Dipungut Sesuai Dengan PP Nomor 27 Tahun 2017 |
| `TD.01113` | 13 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 13 TAHUN 2025 |
| `TD.01114` | 14 - PPN DITANGGUNG PEMERINTAH EKS PMK 102/PMK.010/2021 |
| `TD.01115` | 15 - PPN DITANGGUNG PEMERINTAH EKS PMK 239/PMK.03/2020 |
| `TD.01116` | 16 - Insentif PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 103/PMK.010/2021 |
| `TD.01117` | 17 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 40 TAHUN 2021 |
| `TD.01118` | 18 - PAJAK PERTAMBAHAN NILAI TIDAK DIPUNGUT BERDASARKAN PP NOMOR 41 TAHUN 2021 |
| `TD.01119` | 19 - PPN DITANGGUNG PEMERINTAH EKS PMK 6/PMK.010/2022 |
| `TD.01120` | 20 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 226/PMK.03/2021 |
| `TD.01121` | 21 - PPN ATAU PPN DAN PPnBM TIDAK DIPUNGUT SESUAI DENGAN PP NOMOR 53 TAHUN 2017 |
| `TD.01122` | 22 - PPN tidak dipungut berdasarkan PP Nomor 70 Tahun 2021 |
| `TD.01123` | 23 - PPN ditanggung Pemerintah Ex PMK-125/PMK.01/2020 |
| `TD.01124` | 24 - (Tidak ada Cap) |
| `TD.01125` | 25 - PPN tidak dipungut berdasarkan PP Nomor 49 Tahun 2022 |
| `TD.01126` | 26 - PPN tidak dipungut berdasarkan PP Nomor 12 Tahun 2023 |
| `TD.01127` | 27 - PPN Ditanggung Pemerintah berdasarkan PMK Nomor 12 Tahun 2025 |
| `TD.01128` | 28 - PPN DITANGGUNG PEMERINTAH EKSEKUSI PMK NOMOR 60 TAHUN 2025 |
| `TD.01129` | 29 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 61 TAHUN 2025 |
| `TD.01130` | 30 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 44 TAHUN 2025 |
| `TD.01131` | 31 - PPN DITANGGUNG PEMERINTAH BERDASARKAN PMK NOMOR 90 TAHUN 2025 |

### `l10n_id_coretax_add_info_08` (Localization Identifier Coretax Add Info 08)

| Value | Label |
|---|---|
| `TD.00501` | 1 - untuk BKP dan JKP Tertentu |
| `TD.00502` | 2 - untuk BKP Tertentu yang Bersifat Strategis |
| `TD.00503` | 3 - untuk Jasa Kebandarudaraan |
| `TD.00504` | 4 - untuk Lainnya |
| `TD.00505` | 5 - untuk BKP Tertentu yang Bersifat Strategis sesuai PP Nomor 81 Tahun 2015 |
| `TD.00506` | 6 - untuk Penyerahan Jasa Kepelabuhan Tertentu untuk kegiatan angkutan laut Luar Negeri |
| `TD.00507` | 7 - untuk Penyerahan Air Bersih |
| `TD.00508` | 8 - Penyerahan BKP tertentu yang bersifat strategis berdasarkan PP 48 Tahun 2020 |
| `TD.00509` | 9 - Penyerahan kepada Perwakilan Negara Asing dan Badan Internasional serta Pejabatnya |
| `TD.00510` | 10 - BKP dan JKP tertentu |

### `l10n_id_coretax_facility_info_08` (Localization Identifier Coretax Facility Info 08)

| Value | Label |
|---|---|
| `TD.01101` | 1 - PPN Dibebaskan Sesuai PP Nomor 146 Tahun 2000 Sebagaimana Telah Diubah Dengan PP Nomor 38 Tahun 2003 |
| `TD.01102` | 2 - PPN Dibebaskan Sesuai PP Nomor 12 Tahun 2001 Sebagaimana Telah Beberapa Kali Diubah Terakhir Dengan PP Nomor 31 Tahun 2007 |
| `TD.01103` | 3 - PPN dibebaskan berdasarkan Peraturan Pemerintah Nomor 28 Tahun 2009 |
| `TD.01104` | 4 - (Tidak ada cap) |
| `TD.01105` | 5 - PPN Dibebaskan Sesuai Dengan PP Nomor 81 Tahun 2015 |
| `TD.01106` | 6 - PPN Dibebaskan Berdasarkan PP Nomor 74 Tahun 2015 |
| `TD.01107` | 7 - (tanpa cap) |
| `TD.01108` | 8 - PPN DIBEBASKAN SESUAI PP NOMOR 81 TAHUN 2015 SEBAGAIMANA TELAH DIUBAH DENGAN PP 48 TAHUN 2020 |
| `TD.01109` | 9 - PPN DIBEBASKAN BERDASARKAN PP NOMOR 47 TAHUN 2020 |
| `TD.01110` | 10 - PPN Dibebaskan berdasarkan PP Nomor 49 Tahun 2022 |

### `l10n_in_gst_treatment` (goods and services tax Treatment)

| Value | Label |
|---|---|
| `regular` | Registered Business - Regular |
| `composition` | Registered Business - Composition |
| `unregistered` | Unregistered Business |
| `consumer` | Consumer |
| `overseas` | Overseas |
| `special_economic_zone` | Special Economic Zone |
| `deemed_export` | Deemed Export |
| `uin_holders` | UIN Holders |

### `l10n_in_edi_status` (India E-Invoice Status)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `cancelled` | Cancelled |

### `l10n_it_edi_state` (SDI State)

| Value | Label |
|---|---|
| `being_sent` | Being Sent To SdI |
| `requires_user_signature` | Requires user signature |
| `processing` | SdI Processing |
| `rejected` | SdI Rejected |
| `forwarded` | SdI Accepted, Forwarded to Partner |
| `forward_failed` | SdI Accepted, Forward to Partner Failed |
| `forward_attempt` | SdI Accepted, Forwarding to Partner |
| `accepted_by_pa_partner` | SdI Accepted, Accepted by the PA Partner |
| `rejected_by_pa_partner` | SdI Accepted, Rejected by the PA Partner |
| `accepted_by_pa_partner_after_expiry` | SdI Accepted, PA Partner Expired Terms |

### `l10n_it_origin_document_type` (Origin Document Type)

| Value | Label |
|---|---|
| `purchase_order` | Purchase Order |
| `contract` | Contract |
| `agreement` | Agreement |

### `l10n_jo_edi_state` (JoFotara State)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |
| `demo` | Sent (Demo) |

### `l10n_jo_edi_invoice_type` (Invoice Type)

| Value | Label |
|---|---|
| `local` | Local |
| `export` | Export |
| `development` | Development Area |
| `transit` | Transit |
| `foreign` | Foreign Trade |
| `freezone` | Free Zone Transfer |

### `l10n_my_edi_state` (MyInvois State)

| Value | Label |
|---|---|
| `in_progress` | Validation In Progress |
| `valid` | Valid |
| `rejected` | Rejected |
| `invalid` | Invalid |
| `cancelled` | Cancelled |

### `l10n_pl_edi_status` (KSeF Status)

| Value | Label |
|---|---|
| `sent` | Sent (In Progress) |
| `accepted` | Accepted |
| `rejected` | Rejected |
| `fetch_ready` | Fetch Ready |
| `fetched` | Fetched |
| `fetch_failed` | Fetch Failed |

### `l10n_ro_edi_state` (E-Factura Status)

| Value | Label |
|---|---|
| `invoice_not_indexed` | Not indexed |
| `invoice_sent` | Sent |
| `invoice_refused` | Refused |
| `invoice_validated` | Validated |

### `l10n_rs_edi_state` (Serbia E-Invoice state)

| Value | Label |
|---|---|
| `sent` | Sent |
| `sending_failed` | Error |

### `l10n_rs_tax_date_obligations_code` (Tax Date Obligations)

| Value | Label |
|---|---|
| `35` | By Delivery Date |
| `3` | By Issuance Date |
| `432` | By Billing System |

### `l10n_tr_nilvera_send_status` (Nilvera Status)

| Value | Label |
|---|---|
| `error` | Error |
| `not_sent` | Not sent |
| `sent` | Sent and waiting response |
| `succeed` | Successful |
| `waiting` | Waiting |
| `unknown` | Unknown |

### `l10n_tr_gib_invoice_scenario` (Invoice Scenario)

| Value | Label |
|---|---|
| `TEMELFATURA` | Basic |
| `KAMU` | Public Sector |

### `l10n_tr_gib_invoice_type` (GIB Invoice Type)

| Value | Label |
|---|---|
| `SATIS` | Sales |
| `TEVKIFAT` | Withholding |
| `IHRACKAYITLI` | Registered for Export |
| `ISTISNA` | Tax Exempt |

### `l10n_tr_shipping_type` (Shipping Method)

| Value | Label |
|---|---|
| `1` | Sea Transportation |
| `2` | Railway Transportation |
| `3` | Road Transportation |
| `4` | Air Transportation |
| `5` | Post |
| `6` | Combined Transportation |
| `7` | Fixed Transportation |
| `8` | Domestic Water Transportation |
| `9` | Invalid Transportation Method |

### `l10n_tw_edi_state` (Invoice Status)

| Value | Label |
|---|---|
| `invoiced` | Invoiced |
| `valid` | Valid |
| `invalid` | Invalid |

### `l10n_tw_edi_carrier_type` (Carrier Type)

| Value | Label |
|---|---|
| `1` | ECpay e-invoice carrier |
| `2` | Citizen Digital Certificate |
| `3` | Mobile Barcode |
| `4` | EasyCard |
| `5` | iPass |

### `l10n_tw_edi_invoice_type` (Ecpay Invoice Type)

| Value | Label |
|---|---|
| `07` | General Invoice |
| `08` | Special Invoice |

### `l10n_tw_edi_clearance_mark` (Clearance Mark)

| Value | Label |
|---|---|
| `1` | NOT via the customs |
| `2` | Via the customs |

### `l10n_tw_edi_zero_tax_rate_reason` (Zero Tax Rate Reason)

| Value | Label |
|---|---|
| `71` | 71: No.1 export goods |
| `72` | 72: No.2 Services related to export sales, or services provided domestically but used abroad |
| `73` | 73: No.3 Duty-free shops established by law for the sale and transit or departure of passengers |
| `74` | 74: No.4 Sale of goods or services for operation by the operator of the FREE Trade Zone |
| `75` | 75: No.5 International transportation. However, foreign transport undertakings operating international transport business in Taiwan shall be limited to those whose countries shall give equal treatment to Taiwan's international transport undertakings or be exempt from similar taxes |
| `76` | 76: No.6 Ships, aircraft and distant-water fishing vessels for international transportation |
| `77` | 77: No.7 Goods or repair services used by ships, aircraft and distant-water fishing vessels for sale and international transport |
| `78` | 78: No.8 The bonded area operator sells goods that are not directly exported by the taxable area operator and the taxable area operator is not exported to the taxation area |
| `79` | 79: No.9 The bonded area operator sells the goods that the taxable area operator deposits into the bonded warehouse or logistics center managed by the free port area or customs administration for export |

### `l10n_tw_edi_refund_state` (Refund State)

| Value | Label |
|---|---|
| `to_be_agreed` | To be agreed |
| `agreed` | Agreed |
| `disagreed` | Disagreed |

### `l10n_tw_edi_refund_agreement_type` (Refund invoice Agreement Type)

| Value | Label |
|---|---|
| `offline` | Offline Agreement |
| `online` | Online Agreement |

### `l10n_tw_edi_allowance_notify_way` (Allowance Notify Way)

| Value | Label |
|---|---|
| `email` | Email |
| `phone` | Phone |

### `l10n_vn_edi_invoice_state` (Sinvoice Status)

| Value | Label |
|---|---|
| `ready_to_send` | Ready to send |
| `sent` | Sent |
| `payment_state_to_update` | Payment status to update |
| `canceled` | Canceled |
| `adjusted` | Adjusted |
| `replaced` | Replaced |

### `l10n_vn_edi_adjustment_type` (Adjustment type)

| Value | Label |
|---|---|
| `1` | Money adjustment |
| `2` | Information adjustment |

## State fields

State machine fields of this entity: `state`, `payment_state`, `edi_state`, `peppol_move_state`, `nemhandel_move_state`, `l10n_es_tbai_state`, `l10n_es_edi_verifactu_state`, `pdp_ppf_move_state`, `pdp_ppf_lifecycle_state`, `l10n_fr_pdp_status`, `l10n_gr_edi_state`, `l10n_hr_business_document_status`, `l10n_hr_fiscalization_status`, `l10n_hr_mer_document_status`, `l10n_hu_edi_state`, `l10n_in_edi_status`, `l10n_it_edi_state`, `l10n_jo_edi_state`, `l10n_my_edi_state`, `l10n_pl_edi_status`, `l10n_ro_edi_state`, `l10n_rs_edi_state`, `l10n_tr_nilvera_send_status`, `l10n_tr_nilvera_customer_status`, `l10n_tw_edi_state`, `l10n_tw_edi_refund_state`, `l10n_vn_edi_invoice_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (11)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_checked_idx` | Index | `(journal_id) WHERE (checked IS NOT TRUE)` |  | `account` |
| `_payment_idx` | Index | `(journal_id, state, payment_state, move_type, date)` |  | `account` |
| `_unique_name` | UniqueIndex | `(name, journal_id) WHERE (state = 'posted'AND name != '/')` | Another entry with the same name already exists. | `account` |
| `_journal_id_date_idx` | Index | `(journal_id, date)` |  | `account` |
| `_journal_id_company_id_idx` | Index | `(journal_id, company_id, date)` |  | `account` |
| `_made_gaps` | Index | `(journal_id, state, payment_state, move_type, date) WHERE (made_sequence_gap IS TRUE)` |  | `account` |
| `_duplicate_bills_idx` | Index | `(ref) WHERE (move_type IN ('in_invoice', 'in_refund'))` |  | `account` |
| `_account_move_sanitize_payment_ref_idx` | Index | `(regexp_replace(COALESCE(payment_reference, ''), '[^a-zA-Z0-9]', '', 'g'))` |  | `account` |
| `_unique_name` | UniqueIndex | `(name, journal_id) WHERE (state = 'posted' AND name != '/' AND (l10n_latam_document_type_id IS NULL OR move_type NOT IN ('in_invoice', 'in_refund', 'in_receipt')))` | Another entry with the same name already exists. | `l10n_latam_invoice_document` |
| `_unique_name_latam` | UniqueIndex | `(name, commercial_partner_id, l10n_latam_document_type_id, company_id) WHERE (state = 'posted' AND name != '/' AND (l10n_latam_document_type_id IS NOT NULL AND move_type IN ('in_invoice', 'in_refund', 'in_receipt')))` | Another entry with the same name already exists. | `l10n_latam_invoice_document` |
| `_l10n_pl_edi_number_company_id_move_type_uniq` | Constraint | `UNIQUE(l10n_pl_edi_number, company_id, move_type)` | The KSeF number must be unique per company per move_type | `l10n_pl_edi` |

## Operations (1065)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_sequence_monthly_regex` | internal rule | self | `account` |  |  |
| `_sequence_yearly_regex` | internal rule | self | `account` |  |  |
| `_sequence_year_range_regex` | internal rule | self | `account` |  |  |
| `_sequence_fixed_regex` | internal rule | self | `account` |  |  |
| `_sequence_year_range_monthly_regex` | internal rule | self | `account` |  |  |
| `_auto_init` | lifecycle override | self | `account`, `l10n_eg_edi_eta`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_latam_invoice_document`, `website_sale` |  | Create columns `l10n_es_edi_verifactu_state` and `l10n_es_edi_verifactu_clave_regimen` to avoid computing them during the installation of the module. |
| `_compute_invoice_default_sale_person` | computation | self | `account` | depends: `move_type`, `partner_id` |  |
| `_compute_is_being_sent` | computation | self | `account` | depends: `sending_data` |  |
| `compute_move_sent_values` | computation | self | `account` | depends: `is_move_sent` |  |
| `_search_move_sent_values` | search rule | self, operator, value | `account` |  |  |
| `_compute_payment_reference` | computation | self | `account` |  |  |
| `_compute_sanitize_payment_reference` | computation | self | `account` | depends: `payment_reference` |  |
| `_get_accounting_date_source` | preparation rule | self | `account`, `l10n_cz`, `l10n_de`, `l10n_pl` |  |  |
| `_compute_date` | computation | self | `account` | depends: `invoice_date`, `company_id`, `move_type`, `taxable_supply_date` |  |
| `_compute_auto_post_until` | computation | self | `account` | depends: `auto_post` |  |
| `_compute_hide_post_button` | computation | self | `account` | depends: `date`, `auto_post` |  |
| `_compute_company_id` | computation | self | `account` | depends: `journal_id` |  |
| `_compute_journal_id` | computation | self | `account` | depends: `move_type`, `origin_payment_id`, `statement_line_id` |  |
| `_get_valid_journal_types` | preparation rule | self | `account` |  |  |
| `_search_default_journal` | search rule | self | `account` |  |  |
| `_compute_is_storno` | computation | self | `account`, `point_of_sale` | depends: `move_type` |  |
| `_compute_suitable_journal_ids` | computation | self | `account` | depends: `company_id`, `invoice_filter_type_domain` |  |
| `_compute_name` | computation | self | `account`, `l10n_latam_invoice_document` | depends: `posted_before`, `state`, `journal_id`, `date`, `move_type`, `origin_payment_id`; depends: `l10n_latam_document_type_id` | Change the way that the use_document moves name is computed:  * If move use document but does not have document type selected then name = 'False' to do not show the name. * If move use document and are numbered manually do not compute name at all (will be set manually) * If move use document and is in draft state and has not been posted before we restart name to False (this is    when we change the document type) |
| `_compute_name_placeholder` | computation | self | `account`, `l10n_latam_invoice_document` | depends: `date`, `journal_id`, `move_type`, `name`, `posted_before`, `sequence_number`, `sequence_prefix`, `state` |  |
| `_compute_highest_name` | computation | self | `account`, `l10n_latam_invoice_document` | depends: `journal_id`, `date`; depends: `journal_id`, `l10n_latam_document_type_id` |  |
| `_compute_made_sequence_gap` | computation | self | `account`, `l10n_latam_invoice_document` |  |  |
| `_compute_type_name` | computation | self | `account` | depends_context: `lang`; depends: `move_type` |  |
| `_compute_secured` | computation | self | `account` | depends: `inalterable_hash` |  |
| `_search_secured` | search rule | self, operator, value | `account` |  |  |
| `_compute_always_tax_exigible` | computation | self | `account`, `point_of_sale` | depends: `line_ids.account_id.account_type`; depends: `tax_cash_basis_created_move_ids`, `pos_session_ids` |  |
| `_compute_commercial_partner_id` | computation | self | `account`, `hr_expense` | depends: `partner_id`; depends: `partner_id`, `expense_ids`, `company_id` |  |
| `_compute_partner_shipping_id` | computation | self | `account` | depends: `partner_id` |  |
| `_compute_fiscal_position_id` | computation | self | `account`, `l10n_in`, `l10n_it_edi_doi` | depends: `partner_id`, `partner_shipping_id`, `company_id`, `move_type`; depends: `l10n_in_state_id`, `l10n_in_gst_treatment`; depends: `l10n_it_edi_doi_id` |  |
| `_compute_partner_bank_id` | computation | self | `account` | depends: `bank_partner_id`, `currency_id`, `preferred_payment_method_line_id` |  |
| `_compute_invoice_payment_term_id` | computation | self | `account` | depends: `partner_id` |  |
| `_compute_invoice_date_due` | computation | self | `account` | depends: `needed_terms` |  |
| `_compute_delivery_date` | computation | self | `account`, `sale_stock` | depends: `line_ids.sale_line_ids.order_id.effective_date` |  |
| `_compute_show_delivery_date` | computation | self | `account`, `l10n_de`, `l10n_fr_account`, `l10n_hu`, `l10n_rs_edi`, `l10n_sa` | depends: `delivery_date`; depends: `country_code`, `move_type` |  |
| `_compute_taxable_supply_date` | computation | self | `account`, `l10n_cz` | depends: `country_code` |  |
| `_compute_show_taxable_supply_date` | computation | self | `account`, `l10n_cz`, `l10n_de`, `l10n_pl`, `l10n_sk` | depends: `country_code` |  |
| `_compute_taxable_supply_date_placeholder` | computation | self | `account`, `l10n_pl` | depends: `country_code` |  |
| `_compute_currency_id` | computation | self | `account` | depends: `journal_id`, `statement_line_id` |  |
| `_get_invoice_currency_rate_date` | preparation rule | self | `account`, `l10n_cz`, `l10n_de`, `l10n_hu_edi`, `l10n_pl` |  |  |
| `_get_expected_currency_rate_at` | preparation rule | self, date | `account` |  |  |
| `_compute_expected_currency_rate` | computation | self | `account`, `l10n_hu_edi` | depends: `currency_id`, `company_currency_id`, `company_id`, `invoice_date`, `taxable_supply_date`; depends: `delivery_date` |  |
| `_compute_invoice_currency_rate` | computation | self | `account` | depends: `currency_id`, `company_currency_id`, `company_id`, `invoice_date`, `taxable_supply_date` |  |
| `_compute_direction_sign` | computation | self | `account` | depends: `move_type` |  |
| `_compute_amount` | computation | self | `account`, `point_of_sale` | depends: `line_ids.matched_debit_ids.debit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_debit_ids.debit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.matched_credit_ids.credit_move_id.move_id.origin_payment_id.is_matched`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual`, `line_ids.matched_credit_ids.credit_move_id.move_id.line_ids.amount_residual_currency`, `line_ids.balance`, `line_ids.currency_id`, `line_ids.amount_currency`, `line_ids.amount_residual`, `line_ids.amount_residual_currency`, `line_ids.payment_id.state`, `line_ids.full_reconcile_id`, `state` |  |
| `_compute_payment_state` | computation | self | `account` | depends: `amount_residual`, `move_type`, `state`, `company_id`, `reconciled_payment_ids.state` |  |
| `_compute_status_in_payment` | computation | self | `account` | depends: `payment_state`, `state`, `is_move_sent` |  |
| `_field_to_sql` | internal rule | self, alias, fname, query | `account` |  |  |
| `_compute_payment_count` | computation | self | `account` | depends: `reconciled_payment_ids` |  |
| `_compute_adjusting_entries_count` | computation | self | `account` | depends: `adjusting_entries_move_ids` |  |
| `_compute_adjusting_entry_origin_moves_count` | computation | self | `account` | depends: `adjusting_entry_origin_move_ids` |  |
| `_compute_adjusting_entry_origin_label` | computation | self | `account` | depends_context: `lang`; depends: `adjusting_entry_origin_move_ids` |  |
| `_compute_needed_terms` | computation | self | `account`, `hr_expense` | depends: `invoice_payment_term_id`, `invoice_date`, `currency_id`, `amount_total_in_currency_signed`, `invoice_date_due`; depends: `expense_ids` |  |
| `_compute_show_journal` | computation | self | `account` | depends: `suitable_journal_ids` |  |
| `_compute_payments_widget_to_reconcile_info` | computation | self | `account` |  |  |
| `_compute_invoice_has_outstanding` | computation | self | `account` | depends: `invoice_outstanding_credits_debits_widget` |  |
| `_compute_preferred_payment_method_line_id` | computation | self | `account`, `l10n_jo_edi` | depends: `partner_id`, `company_id` |  |
| `_compute_payments_widget_reconciled_info` | computation | self | `account`, `point_of_sale` | depends: `move_type`, `line_ids.amount_residual` | Add pos_payment_name field in the reconciled vals to be able to show the payment method in the invoice. |
| `_get_product_base_line_currency_rate` | preparation rule | self, product_line | `account` |  |  |
| `_prepare_product_base_line_for_taxes_computation` | preparation rule | self, product_line | `account`, `hr_expense` |  | Convert an account.move.line having display_type='product' into a base line for the taxes computation.  :param product_line: An account.move.line. :return: A base line returned by '_prepare_base_line_for_taxes_computation'. |
| `_prepare_epd_base_line_for_taxes_computation` | preparation rule | self, epd_line | `account` |  | Convert an account.move.line having display_type='epd' into a base line for the taxes computation.  :param epd_line: An account.move.line. :return: A base line returned by '_prepare_base_line_for_taxes_computation'. |
| `_prepare_epd_base_lines_for_taxes_computation_from_base_lines` | preparation rule | self, base_lines | `account` |  | Anticipate the epd lines to be generated from the base lines passed as parameter. When the record is in draft (not saved), the accounting items are not there so we can't call '_prepare_epd_base_line_for_taxes_computation'.  :param base_lines: The base lines generated by '_prepare_product_base_line_for_taxes_computation'. :return: A list of base lines representing the epd lines. |
| `_prepare_cash_rounding_base_line_for_taxes_computation` | preparation rule | self, cash_rounding_line | `account` |  | Convert an account.move.line having display_type='rounding' into a base line for the taxes computation.  :param cash_rounding_line: An account.move.line. :return: A base line returned by '_prepare_base_line_for_taxes_computation'. |
| `_prepare_tax_line_for_taxes_computation` | preparation rule | self, tax_line | `account` |  | Convert an account.move.line having display_type='tax' into a tax line for the taxes computation.  :param tax_line: An account.move.line. :return: A tax line returned by '_prepare_tax_line_for_taxes_computation'. |
| `_prepare_non_deductible_base_line_for_taxes_computation` | preparation rule | self, non_deductible_line | `account` |  | Convert an account.move.line having display_type='non_deductible' into a base line for the taxes computation.  :param non_deductible_line: An account.move.line. :return: A base line returned by '_prepare_base_line_for_taxes_computation'. |
| `_prepare_non_deductible_base_lines_for_taxes_computation_from_base_lines` | preparation rule | self, base_lines | `account` |  | Anticipate the non deductible lines to be generated from the base lines passed as parameter. When the record is in draft (not saved), the accounting items are not there so we can't call '_prepare_non_deductible_base_line_for_taxes_computation'.  :param base_lines: The base lines generated by '_prepare_product_base_line_for_taxes_computation'. :return: A list of base lines representing the non deductible lines. |
| `_get_rounded_base_and_tax_lines` | preparation rule | self, round_from_tax_lines | `account` |  | Small helper to extract the base and tax lines for the taxes computation from the current move. The move could be stored or not and could have some features generating extra journal items acting as base lines for the taxes computation (e.g. epd, rounding lines).  :param round_from_tax_lines:    Indicate if the manual tax amounts of tax journal items should be kept or not.                                 It only works when the move is stored. :return:                        A tuple <base_lines, tax_lines> for the taxes computation. |
| `_compute_tax_totals` | computation | self | `account`, `l10n_cl`, `l10n_id`, `point_of_sale` | depends_context: `lang`; depends: `invoice_line_ids.currency_rate`, `invoice_line_ids.tax_base_amount`, `invoice_line_ids.tax_line_id`, `invoice_line_ids.price_total`, `invoice_line_ids.price_subtotal`, `invoice_payment_term_id`, `partner_id`, `currency_id` | Computed field used for custom widget's rendering. Only set on invoices. |
| `_compute_payment_term_details` | computation | self | `account` | depends: `show_payment_term_details` | Returns an [] containing the payment term's information to be displayed on the invoice's PDF. |
| `_compute_show_payment_term_details` | computation | self | `account` | depends: `move_type`, `payment_state`, `invoice_payment_term_id` | Determines : - whether or not an additional table should be added at the end of the invoice to display the various - whether or not there is an early pay discount in this invoice that should be displayed |
| `_need_cancel_request` | internal rule | self | `account`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_my_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` |  | Hook allowing a localization to prevent the user to reset draft an invoice that has been already sent to the government and thus, must remain untouched except if its cancellation is approved.  :return: True if the cancel button is displayed instead of draft button, False otherwise. |
| `_compute_need_cancel_request` | computation | self | `account`, `l10n_hu_edi`, `l10n_my_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | depends: `country_code`; depends: `l10n_hu_edi_state`; depends: `l10n_my_edi_state`; depends: `l10n_tw_edi_state`; depends: `l10n_vn_edi_invoice_state` |  |
| `_compute_invoice_partner_display_info` | computation | self | `account` | depends: `partner_id`, `invoice_source_email`, `partner_id.display_name` |  |
| `_compute_invoice_filter_type_domain` | computation | self | `account` | depends: `move_type` |  |
| `_compute_bank_partner_id` | computation | self | `account` | depends: `commercial_partner_id`, `company_id`, `move_type` |  |
| `_compute_tax_lock_date_message` | computation | self | `account` | depends: `date`, `line_ids.debit`, `line_ids.credit`, `line_ids.tax_line_id`, `line_ids.tax_ids`, `line_ids.tax_tag_ids`, `invoice_line_ids.debit`, `invoice_line_ids.credit`, `invoice_line_ids.tax_line_id`, `invoice_line_ids.tax_ids`, `invoice_line_ids.tax_tag_ids` |  |
| `_compute_display_inactive_currency_warning` | computation | self | `account` | depends: `currency_id` |  |
| `_compute_tax_country_id` | computation | self | `account` | depends: `company_id.account_fiscal_country_id`, `fiscal_position_id`, `fiscal_position_id.country_id`, `fiscal_position_id.foreign_vat` |  |
| `_compute_tax_country_code` | computation | self | `account` | depends: `tax_country_id` |  |
| `_compute_has_reconciled_entries` | computation | self | `account` | depends: `line_ids` |  |
| `_compute_show_reset_to_draft_button` | computation | self | `account_edi`, `account_peppol`, `account`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_gr_edi_e_invoo`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_my_edi`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` | depends: `restrict_mode_hash_table`, `state`, `inalterable_hash`; depends: `edi_document_ids.state`; depends: `peppol_is_sent`; depends: `l10n_es_tbai_post_document_id.chain_index`; depends: `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.json_attachment_id`; depends: `l10n_gr_edi_state`; depends: `l10n_hr_fiscalization_status`; depends: `l10n_hu_edi_state`, `state`; depends: `l10n_it_edi_transaction`; depends: `l10n_jo_edi_state`; depends: `l10n_my_edi_state`; depends: `l10n_pl_edi_status`; depends: `l10n_ro_edi_state`; depends: `l10n_rs_edi_state`; depends: `state`, `edi_document_ids.state`; depends: `l10n_tw_edi_state`, `l10n_tw_edi_refund_state`; depends: `l10n_vn_edi_invoice_state` | Disallow resetting to draft in the following cases: * The move is registered (accepted, regsitered_with_errors, cancelled) * We are waiting to sent a document to the AEAT |
| `_compute_access_url` | computation | self | `account` |  |  |
| `_compute_narration` | computation | self | `account`, `l10n_gcc_invoice` | depends: `move_type`, `partner_id`, `partner_id.lang`, `company_id` |  |
| `_get_partner_credit_warning_exclude_amount` | preparation rule | self | `account`, `sale` |  |  |
| `_compute_partner_credit_warning` | computation | self | `account` | depends: `company_id`, `partner_id`, `tax_totals`, `currency_id` |  |
| `_build_credit_warning_message` | internal rule | self, record, current_amount, exclude_current, exclude_amount | `account` |  | Build the warning message that will be displayed in a yellow banner on top of the current record if the partner exceeds a credit limit (set on the company or the partner itself). :param record:                  The record where the warning will appear (Invoice, Sales Order...). :param float current_amount:    The partner's outstanding credit amount from the current document. :param bool exclude_current:    DEPRECATED in favor of parameter `exclude_amount`:                                 Whether to exclude `current_amount` from the credit to invoice. :param float exclude_amount:    The amount  |
| `_compute_quick_edit_mode` | computation | self | `account` | depends: `journal_id.type`, `company_id` |  |
| `_compute_quick_encoding_vals` | computation | self | `account` | depends: `quick_edit_total_amount`, `invoice_line_ids.price_total`, `tax_totals` |  |
| `_compute_duplicated_ref_ids` | computation | self | `account` | depends: `ref`, `move_type`, `partner_id`, `invoice_date`, `tax_totals`, `currency_id` |  |
| `_fetch_duplicate_reference` | internal rule | self, matching_states | `account` |  |  |
| `_compute_is_draft_duplicated_ref_ids` | computation | self | `account` | depends: `duplicated_ref_ids` |  |
| `_compute_display_qr_code` | computation | self | `account` | depends: `company_id` |  |
| `_compute_display_link_qr_code` | computation | self | `account` | depends: `company_id` |  |
| `_compute_amount_total_words` | computation | self | `account` | depends: `amount_total`, `currency_id` |  |
| `_compute_incoterm` | computation | self | `account` | depends: `company_id`, `move_type` |  |
| `_compute_linked_attachment_id` | computation | self, attachment_field, binary_field | `account` |  | Helper to retreive Attachment from Binary fields This is needed because fields.Many2one('ir.attachment') makes all attachments available to the user. |
| `_compute_incoterm_location` | computation | self | `account`, `purchase_stock`, `sale_stock` | depends: `line_ids.sale_line_ids.order_id`; depends: `purchase_id` |  |
| `_compute_invoice_incoterm_placeholder` | computation | self | `account` | depends: `company_id.incoterm_id` |  |
| `_compute_abnormal_warnings` | computation | self | `account` | depends: `partner_id`, `invoice_date`, `amount_total` | Assign warning fields based on historical data.  The last invoices (between 10 and 30) are used to compute the normal distribution. If the amount or days between invoices of the current invoice falls outside of the boundaries of the Bell curve, we warn the user. |
| `_compute_alerts` | computation | self | `account` | depends: `state`, `invoice_line_ids`, `tax_lock_date_message`, `auto_post`, `auto_post_until`, `is_being_sent`, `partner_credit_warning`, `abnormal_amount_warning`, `abnormal_date_warning` |  |
| `_compute_taxes_legal_notes` | computation | self | `account` | depends: `line_ids.tax_ids` |  |
| `_compute_next_payment_date` | computation | self | `account` | depends: `line_ids.payment_date`, `line_ids.reconciled` |  |
| `_compute_display_send_button` | computation | self | `account_peppol`, `account` | depends: `move_type`, `state` |  |
| `_compute_highlight_send_button` | computation | self | `account`, `l10n_my_edi` | depends: `is_being_sent`, `invoice_pdf_report_id`; depends: `l10n_my_invoice_need_edi`, `l10n_my_edi_state` |  |
| `_compute_is_sale_installed` | computation | self | `account` |  |  |
| `_compute_reconciled_payment_ids` | computation | self | `account` | depends: `line_ids.matched_debit_ids`, `line_ids.matched_credit_ids`, `matched_payment_ids`, `matched_payment_ids.state` | Retrieve the payments reconciled to the invoices through the reconciliation (account.partial.reconcile) |
| `_search_next_payment_date` | search rule | self, operator, value | `account` |  |  |
| `_compute_checked` | computation | self | `account` | depends: `state`, `journal_id.type` |  |
| `_compute_no_followup` | computation | self | `account` | depends: `line_ids.no_followup` |  |
| `_inverse_no_followup` | inverse computation | self | `account` |  |  |
| `_get_alerts` | preparation rule | self | `account` |  |  |
| `_search_journal_group_id` | search rule | self, operator, value | `account` |  |  |
| `_search_reconciled_payment_ids` | search rule | self, operator, value | `account` |  |  |
| `_inverse_delivery_date` | inverse computation | self | `account`, `l10n_hu_edi` |  |  |
| `_inverse_tax_totals` | inverse computation | self | `account` |  |  |
| `_inverse_amount_total` | inverse computation | self | `account` |  |  |
| `_inverse_partner_id` | on change | self | `account` | onchange: `partner_id` |  |
| `_inverse_company_id` | on change | self | `account` | onchange: `company_id` |  |
| `_inverse_currency_id` | on change | self | `account` | onchange: `currency_id` |  |
| `_inverse_journal_id` | on change | self | `account` | onchange: `journal_id` |  |
| `_inverse_payment_reference` | on change | self | `account` | onchange: `payment_reference` |  |
| `_inverse_invoice_payment_term_id` | on change | self | `account` | onchange: `invoice_payment_term_id` |  |
| `_inverse_name` | inverse computation | self | `account` |  |  |
| `_onchange_date` | on change | self | `account` | onchange: `date` |  |
| `_onchange_invoice_vendor_bill` | on change | self | `account` | onchange: `invoice_vendor_bill_id` |  |
| `_onchange_fpos_id_show_update_fpos` | on change | self | `account` | onchange: `fiscal_position_id` |  |
| `_onchange_partner_id` | on change | self | `account`, `l10n_se`, `purchase` | onchange: `partner_id`; onchange: `partner_id`, `company_id` | If Vendor Bill and Vendor OCR is set, add it. |
| `_onchange_name_warning` | on change | self | `account`, `l10n_in` | onchange: `name`, `highest_name`; onchange: `name` |  |
| `_onchange_journal_id` | on change | self | `account` | onchange: `journal_id` |  |
| `_onchange_invoice_cash_rounding_id` | on change | self | `account` | onchange: `invoice_cash_rounding_id` |  |
| `_check_balanced` | validation | self, container | `account` |  | Assert the move is fully balanced debit = credit. An error is raised if it's not the case. |
| `_get_unbalanced_moves` | preparation rule | self, container | `account` |  |  |
| `_check_fiscal_lock_dates` | validation | self | `account` |  |  |
| `_require_bill_date_for_autopost` | validation | self | `account` | constrains: `auto_post`, `invoice_date` | Vendor bills must have an invoice date set to be posted. Require it for auto-posted bills. |
| `_check_journal_move_type` | validation | self | `account`, `hr_expense` | constrains: `journal_id`, `move_type` |  |
| `_validate_taxes_country` | validation | self | `account` | constrains: `line_ids`, `fiscal_position_id`, `company_id` | By playing with the fiscal position in the form view, it is possible to keep taxes on the invoices from a different country than the one allowed by the fiscal country or the fiscal position. This contrains ensure such account.move cannot be kept, as they could generate inconsistencies in the reports. |
| `_check_invoice_currency_rate` | validation | self | `account` | constrains: `invoice_currency_rate` | Ensure the currency rate is strictly positive when invoice currency differs from company currency. |
| `action_add_from_catalog` | user action | self | `account` |  |  |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `account` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `account` |  |  |
| `_default_order_line_values` | preparation rule | self, child_field | `account` |  |  |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `account` |  |  |
| `_get_product_price_and_data` | preparation rule | self, product | `account` |  | This function will return a dict containing the price of the product. If the product is a sale document then we return the list price (which is the "Sales Price" in a product) otherwise we return the standard_price (which is the "Cost" in a product). In case of a purchase document, it's possible that we have special price for certain partner. We will check the sellers set on the product and update the price and min_qty for it if needed. |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, section_id, **kwargs | `account` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, section_id, child_field, **kwargs | `account`, `stock_landed_costs` |  | Update account_move_line information for a given product or create a new one if none exists yet. :param int product_id: The product, as a `product.product` id. :param int quantity: The quantity selected in the catalog :param int section_id: The id of section selected in the catalog. :return: The unit price of the product, based on the pricelist of the          sale order and the quantity selected. :rtype: float |
| `_is_readonly` | internal rule | self | `account` |  | Check if the move has been canceled |
| `_get_parent_field_on_child_model` | preparation rule | self | `account` |  |  |
| `_is_line_valid_for_section_line_count` | internal rule | self, line | `account` |  | Check if a line is valid for inclusion in the section's line count.  :param recordset line: A record of a move line. :return: True if this line is a valid, else False. :rtype: bool |
| `_is_eligible_for_early_payment_discount` | internal rule | self, currency, reference_date | `account` |  |  |
| `_early_payment_discount_move_types` | internal rule | self | `account` |  |  |
| `_synchronize_business_models` | internal rule | self, changed_fields | `account` |  | Ensure the consistency between: account.payment & account.move account.bank.statement.line & account.move  The idea is to call the method performing the synchronization of the business models regarding their related journal entries. To avoid cycling, the 'skip_account_move_synchronization' key is used through the context.  :param changed_fields: A set containing all modified fields on account.move. |
| `_recompute_cash_rounding_lines` | internal rule | self | `account` |  | Handle the cash rounding feature on invoices.  In some countries, the smallest coins do not exist. For example, in Switzerland, there is no coin for 0.01 CHF. For this reason, if invoices are paid in cash, you have to round their total amount to the smallest coin that exists in the currency. For the CHF, the smallest coin is 0.05 CHF.  There are two strategies for the rounding:  1) Add a line on the invoice for the rounding: The cash rounding line is added as a new invoice line. 2) Add the rounding in the biggest tax amount: The cash rounding line is added as a new tax line on the tax having t |
| `_get_automatic_balancing_account` | preparation rule | self | `account`, `l10n_au` |  | Small helper for special cases where we want to auto balance a move with a specific account. |
| `_sync_unbalanced_lines` | internal rule | self, container | `account` |  |  |
| `_sync_rounding_lines` | internal rule | self, container | `account` |  |  |
| `_sync_dynamic_line_needed_values` | internal rule | self, values_list | `account` | model |  |
| `_sync_tax_lines` | internal rule | self, container | `account` |  |  |
| `_sync_non_deductible_base_lines` | internal rule | self, container | `account` |  |  |
| `_sync_dynamic_line` | internal rule | self, existing_key_fname, needed_vals_fname, needed_dirty_fname, line_type, container | `account` |  |  |
| `_sync_invoice` | internal rule | self, container | `account` |  |  |
| `_get_sync_stack` | preparation rule | self, container | `account`, `l10n_in` |  |  |
| `_sync_dynamic_lines` | internal rule | self, container | `account` |  |  |
| `check_field_access_rights` | operation | self, operation, field_names | `account` | model |  |
| `_get_default_read_fields` | preparation rule | self | `account` | model |  |
| `read` | lifecycle override | self, fields, load | `account` |  |  |
| `search_read` | lifecycle override | self, domain, fields, offset, limit, order, **read_kwargs | `account` | model |  |
| `copy_data` | lifecycle override | self, default | `account`, `l10n_it_edi_doi`, `stock_account` |  |  |
| `copy` | lifecycle override | self, default | `account`, `mrp_account` |  |  |
| `_get_copy_message_content` | preparation rule | self, default | `account_debit_note`, `account` |  | Hook method to customize the message content when copying a move. This method can be overridden by other modules to add custom logic. :param default: The default values dict passed to copy method :return: The message content string |
| `_sanitize_vals` | internal rule | self, vals | `account` |  |  |
| `_stolen_move` | internal rule | self, vals | `account` |  |  |
| `_get_protected_vals` | preparation rule | self, vals, records | `account`, `sale_stock` |  |  |
| `create` | lifecycle override | self, vals_list | `account`, `l10n_gcc_invoice`, `purchase` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account`, `l10n_sa`, `purchase` |  |  |
| `check_move_sequence_chain` | operation | self | `account` |  |  |
| `_get_unlink_logger_message` | preparation rule | self | `account` |  | Before unlink, get a log message for audit trail if restricted. Logger is added here because in api ondelete, account.move.line is deleted, and we can't get total amount |
| `_unlink_forbid_parts_of_chain` | internal rule | self | `account` | ondelete | For a user with Billing/Bookkeeper rights, when the fidu mode is deactivated, moves with a sequence number can only be deleted if they are the last element of a chain of sequence. If they are not, deleting them would create a gap. If the user really wants to do this, he still can explicitly empty the 'name' field of the move; but we discourage that practice. If a user is a Billing Administrator/Accountant or if fidu mode is activated, we show a warning, but they can delete the moves even if it creates a sequence gap. |
| `_unlink_account_audit_trail_except_once_post` | internal rule | self | `account` | ondelete |  |
| `unlink` | lifecycle override | self | `account`, `sale_expense`, `sale` |  |  |
| `_compute_display_name` | computation | self | `account` | depends: `partner_id`, `date`, `state`, `move_type`; depends_context: `input_full_display_name` |  |
| `onchange` | lifecycle override | self, values, field_names, fields_spec | `account` |  |  |
| `_collect_tax_cash_basis_values` | internal rule | self | `account` |  | Collect all information needed to create the tax cash basis journal entries: - Determine if a tax cash basis journal entry is needed. - Compute the lines to be processed and the amounts needed to compute a percentage. :return: A dictionary:     * move:                     The current account.move record passed as parameter.     * to_process_lines:         A tuple (caba_treatment, line) where:                                     - caba_treatment is either 'tax' or 'base', depending on what should                                       be considered on the line when generating the caba entry.     |
| `_must_check_constrains_date_sequence` | internal rule | self | `account` |  |  |
| `_get_last_sequence_domain` | preparation rule | self, relaxed | `account_debit_note`, `account`, `l10n_ar`, `l10n_br`, `l10n_cl`, `l10n_ec`, `l10n_latam_invoice_document`, `l10n_uy` |  | Override to give sequence names in the same journal their own, independent numbering. |
| `_get_starting_sequence` | preparation rule | self | `account_debit_note`, `account`, `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_latam_invoice_document`, `l10n_lk_invoice`, `l10n_uy` |  | If use documents then will create a new starting sequence using the document type code prefix and the journal document number with a 8 padding number |
| `_get_sequence_date_range` | preparation rule | self, reset | `account` |  |  |
| `_get_invoice_reference_euro_invoice` | preparation rule | self | `account` |  | This computes the reference based on the RF Creditor Reference. The data of the reference is the journal short code and the database id number of the invoice. For instance, if a journal code is INV and an invoice is issued with id 37, the check number is 67 so the reference will be 'RF67 INV0 0003 7'. |
| `_get_invoice_reference_euro_partner` | preparation rule | self | `account` |  | This computes the reference based on the RF Creditor Reference. The data of the reference is the user defined reference of the partner or the database id number of the parter. For instance, if an invoice is issued for the partner with internal reference 'food buyer 654', the digits will be extracted and used as the data. This will lead to a check number equal to 00 and the reference will be 'RF00 654'. If no reference is set for the partner, its id in the database will be used. |
| `_get_invoice_reference_number_invoice` | preparation rule | self | `account` |  | This computes the reference based on the Number format. Return the number of the invoice, defined on the journal sequence. |
| `_get_invoice_reference_number_partner` | preparation rule | self | `account` |  | This computes the reference based on the Number format. The data used is the reference set on the partner or its database id otherwise. For instance if the reference of the customer is 'customer 97', the reference will be '97'. |
| `_get_invoice_reference_system_invoice` | preparation rule | self | `account`, `l10n_hr_edi` |  | This computes the reference based on the system format. We simply return the number of the invoice, defined on the journal sequence. |
| `_get_invoice_reference_system_partner` | preparation rule | self | `account` |  | This computes the reference based on the system format. The data used is the reference set on the partner or its database id otherwise. For instance if the reference of the customer is 'dumb customer 97', the reference will be 'CUST/dumb customer 97'. |
| `_get_invoice_computed_reference` | preparation rule | self | `account` |  |  |
| `_get_frequent_account_and_taxes` | preparation rule | self, company_id, partner_id, move_type | `account` | model | Returns the most used accounts and taxes for a given partner and company, eventually filtered according to the move type. |
| `_get_quick_edit_suggestions` | preparation rule | self | `account` |  | Returns a dictionnary containing the suggested values when creating a new line with the quick_edit_total_amount set. We will compute the price_unit that has to be set with the correct that in order to match this total amount. If the vendor/customer is set, we will suggest the most frequently used account for that partner as the default one, otherwise the default of the journal. |
| `_quick_edit_mode_suggest_invoice_date` | on change | self | `account` | onchange: `quick_edit_mode`, `journal_id`, `company_id` | Suggest the Customer Invoice/Vendor Bill date based on previous invoice and lock dates |
| `_onchange_quick_edit_total_amount` | on change | self | `account` | onchange: `quick_edit_total_amount`, `partner_id` | Creates a new line with the suggested values (for the account, the price_unit, and the tax) such that the total amount matches the quick total amount. |
| `_onchange_quick_edit_line_ids` | on change | self | `account` | onchange: `invoice_line_ids` |  |
| `_check_total_amount` | validation | self, amount_total | `account` |  | Verifies that the total amount corresponds to the quick total amount chosen as some rounding errors may appear. In such a case, we round up the tax such that the total is equal to the quick total amount set E.g.: 100€ including 21% tax: base = 82.64, tax = 17.35, total = 99.99 The tax will be set to 17.36 in order to have a total of 100.00 |
| `_get_integrity_hash_fields` | preparation rule | self | `account` |  |  |
| `_get_integrity_hash_fields_and_subfields` | preparation rule | self | `account` |  |  |
| `_get_move_hash_domain` | preparation rule | self, common_domain, force_hash | `account` | model | Returns a search domain on model account.move checking whether they should be hashed. :param common_domain: a search domain that will be included in the returned domain in any case :param force_hash: if True, we'll check all moves posted, independently of journal settings |
| `_is_move_restricted` | internal rule | self, move, force_hash | `account` | model | Returns whether a move should be hashed (depending on journal settings) :param move: the account.move we check :param force_hash: if True, we'll check all moves posted, independently of journal settings |
| `_hash_moves` | internal rule | self, **kwargs | `account` |  |  |
| `_get_chain_info` | preparation rule | self, force_hash, include_pre_last_hash, early_stop | `account` |  | All records in `self` must belong to the same journal and sequence_prefix |
| `_get_chains_to_hash` | preparation rule | self, force_hash, raise_if_gap, raise_if_no_document, include_pre_last_hash, early_stop | `account` |  | From a recordset of moves, retrieve the chains of moves that need to be hashed by taking into account the last move of each chain of the recordset. So if we have INV/1, INV/2, INV/3, INV4 that are not hashed yet in the database but self contains INV/2, INV/3, we will return INV/1, INV/2 and INV/3. Not INV/4. :param force_hash: if True, we'll check all moves posted, independently of journal settings :param raise_if_gap: if True, we'll raise an error if a gap is detected in the sequence :param raise_if_no_document: if True, we'll raise an error if no document needs to be hashed :param include_pr |
| `_calculate_hashes` | internal rule | self, previous_hash | `account` |  | :return: dict of move_id: hash |
| `_apply_delta_recurring_entries` | internal rule | self, date, date_origin, period | `account` | model | Advances date by `period` months, maintaining original day of the month if possible. |
| `_copy_recurring_entries` | internal rule | self | `account` |  | Creates a copy of a recurring (periodic) entry and adjusts its dates for the next period. Meant to be called right after posting a periodic entry. Copies extra fields as defined by _get_fields_to_copy_recurring_entries(). |
| `_get_fields_to_copy_recurring_entries` | preparation rule | self, values | `account` |  | Determines which extra fields to copy when copying a recurring entry. To be extended by modules that add fields with copy=False (implicit or explicit) whenever the opposite behavior is expected for recurring invoices. |
| `_extend_with_attachments` | internal rule | self, files_data, new | `account` |  |  |
| `_get_edi_creation` | preparation rule | self | `account` |  | Get an environment to import documents from other sources.  Allow to edit the current move or create a new one. This will prevent computing the dynamic lines at each invoice line added and only compute everything at the end. |
| `_disable_discount_precision` | internal rule | self | `account` |  | Disable the user defined precision for discounts.  This is useful for importing documents coming from other softwares and providers. The reasonning is that if the document that we are importing has a discount, it shouldn't be rounded to the local settings. |
| `_reason_cannot_decode_has_invoice_lines` | internal rule | self | `account` |  | Helper to get a reason why an invoice cannot be decoded if it has invoice lines. |
| `_post_process_link_to_purchase_order` | internal rule | self, invoice | `account_edi_ubl_cii`, `account` | model |  |
| `_prepare_tax_lines_for_taxes_computation` | preparation rule | self, tax_amls, round_from_tax_lines | `account`, `l10n_sa_edi` |  | If the final invoice has downpayment lines, we skip the tax correction, as we need to recalculate tax amounts without taking into account those lines |
| `_prepare_invoice_aggregated_taxes` | preparation rule | self, filter_invl_to_apply, filter_tax_values_to_apply, grouping_key_generator, round_from_tax_lines, postfix_function | `account` |  | This method is deprecated and will be removed in the next version. Use the following pattern instead:  base_amls = self.line_ids.filtered(lambda x: x.display_type == 'product') base_lines = [self._prepare_product_base_line_for_taxes_computation(x) for x in base_amls] tax_amls = self.line_ids.filtered('tax_repartition_line_id') tax_lines = [self._prepare_tax_line_for_taxes_computation(x) for x in tax_amls] AccountTax._add_tax_details_in_base_lines(base_lines, self.company_id) AccountTax._round_base_lines_tax_details(base_lines, self.company_id, tax_lines=tax_lines)  def grouping_function(base_l |
| `_get_invoice_counterpart_amls_for_early_payment_discount_per_payment_term_line` | preparation rule | self | `account` |  | Helper to get the values to create the counterpart journal items on the register payment wizard and the bank reconciliation widget in case of an early payment discount. When the early payment discount computation is included, we need to compute the base amounts / tax amounts for each receivable / payable but we need to take care about the rounding issues. For others computations, we need to balance the discount you get.  :return: A list of values to create the counterpart journal items split in 3 categories:     * term_lines:   The journal items containing the discount amounts for each receiva |
| `_get_invoice_counterpart_amls_for_early_payment_discount` | preparation rule | self, aml_values_list, open_balance | `account` | model | Helper to get the values to create the counterpart journal items on the register payment wizard and the bank reconciliation widget in case of an early payment discount by taking care of the payment term lines we are matching and the exchange difference in case of multi-currencies.  :param aml_values_list: A list of dictionaries containing:     * aml:              The payment term line we match.     * amount_currency:  The matched amount_currency for this line.     * balance:          The matched balance for this line (could be different in case of multi-currencies). :param open_balance:    The |
| `_affect_tax_report` | internal rule | self | `account` |  |  |
| `_get_move_display_name` | preparation rule | self, show_ref | `account` |  | Helper to get the display name of an invoice depending of its type. :param show_ref:    A flag indicating of the display name must include or not the journal entry reference. :return:            A string representing the invoice. |
| `_get_reconciled_amls` | preparation rule | self | `account` |  | Helper used to retrieve the reconciled move lines on this journal entry |
| `_get_reconciled_payments` | preparation rule | self | `account` |  | Helper used to retrieve the reconciled payments on this journal entry |
| `_get_reconciled_statement_lines` | preparation rule | self | `account` |  | Helper used to retrieve the reconciled statement lines on this journal entry |
| `_get_reconciled_invoices` | preparation rule | self | `account` |  | Helper used to retrieve the reconciled invoices on this journal entry |
| `_get_all_reconciled_invoice_partials` | preparation rule | self | `account` |  |  |
| `_get_reconciled_invoices_partials` | preparation rule | self | `account` |  | Helper to retrieve the details about reconciled invoices. :return A list of tuple (partial, amount, invoice_line). |
| `_reconcile_reversed_moves` | internal rule | self, reverse_moves, move_reverse_cancel | `account` |  | Reconciles moves in self and reverse moves :param move_reverse_cancel: parameter used when lines are reconciled                             will determine whether the tax cash basis journal entries should be created :param reverse_moves:       An account.move recordset, reverse of the current self. :return:                    An account.move recordset, reverse of the current self. |
| `_reverse_moves` | internal rule | self, default_values_list, cancel | `account`, `hr_expense`, `l10n_ar`, `l10n_it_edi`, `sale_expense`, `sale` |  | Reverse a recordset of account.move. If cancel parameter is true, the reconcilable or liquidity lines of each original move will be reconciled with its reverse's. :param default_values_list: A list of default values to consider per move.                             ('type' & 'reversed_entry_id' are computed in the method). :return:                    An account.move recordset, reverse of the current self. |
| `_can_be_unlinked` | internal rule | self | `account`, `l10n_in` |  |  |
| `_is_protected_by_audit_trail` | internal rule | self | `account` |  |  |
| `_unlink_or_reverse` | internal rule | self | `account` |  |  |
| `_post` | internal rule | self, soft | `account_edi`, `account_fleet`, `account_payment_interco`, `account_peppol_response`, `account`, `l10n_ar`, `l10n_cl`, `l10n_de`, `l10n_dk_nemhandel_response`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_hu`, `l10n_in_edi`, `l10n_in`, `l10n_it_edi_doi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_latam_invoice_document`, `l10n_sa`, `l10n_tr_nilvera_einvoice`, `l10n_vn_edi_viettel`, `product_email_template`, `purchase_stock`, `sale`, `stock_account` |  | Post/Validate the documents.  Posting the documents will give it a number, and check that the document is complete (some fields might not be required if not posted but are required otherwise). If the journal is locked with a hash table, it will be impossible to change some fields afterwards.  :param bool soft: if True, future documents are not immediately posted,     but are set to be auto posted automatically at the set accounting date.     Nothing will be performed on those documents before the accounting date. :returns: the Model<account.move> documents that have been posted |
| `_set_next_made_sequence_gap` | internal rule | self, made_gap | `account`, `l10n_latam_invoice_document` |  |  |
| `_get_sequence_suffix` | preparation rule | self | `account` |  | Return this move's sequence suffix (the part of `name` right after the number), or '' if it doesn't have a real sequence assigned yet.  Avoids calling `_get_sequence_format_param` on an unset/placeholder sequence (e.g. '/'), which some localizations treat as an unexpected format. |
| `_update_sequence_made_gap` | internal rule | self, invalidate_current | `account` |  | Update the field made_sequence_gap on the current, next and previous moves.  Either: - we changed something related to the sequence on the current moves, so we need to set the   sequence as broken on the next moves before updating (invalidate_current=True) - we are filling a gap, so we need to update the next move to remove the flag (invalidate_current=False) |
| `_find_and_set_purchase_orders` | internal rule | self, po_references, partner_id, amount_total, from_ocr, timeout | `account`, `purchase` |  | Finds related purchase orders that (partially) match the vendor bill and links the matching lines on this vendor bill.  :param po_references: a list of potential purchase order references/names :param partner_id: the vendor id matched on the vendor bill :param amount_total: the total amount of the vendor bill :param from_ocr: indicates whether this vendor bill was created from an OCR scan (less reliable) :param timeout: the max time the line matching algorithm can take before timing out |
| `_link_bill_origin_to_purchase_orders` | internal rule | self, timeout | `account` |  |  |
| `_autopost_bill` | internal rule | self | `account` |  |  |
| `_show_autopost_bills_wizard` | internal rule | self | `account` |  |  |
| `open_payments` | operation | self | `account` |  |  |
| `open_reconcile_view` | operation | self | `account` |  |  |
| `action_open_business_doc` | user action | self | `account` |  |  |
| `action_update_fpos_values` | user action | self | `account` |  |  |
| `open_created_caba_entries` | operation | self | `account` |  |  |
| `open_adjusting_entries` | operation | self | `account` |  |  |
| `open_adjusting_entry_origin_moves` | operation | self | `account` |  |  |
| `action_switch_move_type` | user action | self | `account` |  |  |
| `get_currency_rate` | operation | self, company_id, to_currency_id, date | `account` |  |  |
| `refresh_invoice_currency_rate` | operation | self | `account` |  |  |
| `action_register_payment` | user action | self | `account` |  |  |
| `action_force_register_payment` | user action | self | `account` |  |  |
| `action_duplicate` | user action | self | `account` |  |  |
| `action_send_and_print` | user action | self | `account_peppol`, `account`, `l10n_dk_nemhandel` |  |  |
| `action_invoice_sent` | user action | self | `account`, `l10n_my_edi` |  | Open a window to compose an email, with the edi invoice template message loaded by default |
| `action_invoice_download_pdf` | user action | self, target | `account` |  |  |
| `action_move_download_all` | user action | self | `account` |  |  |
| `action_print_pdf` | user action | self | `account` |  |  |
| `preview_invoice` | operation | self | `account`, `website_sale` |  |  |
| `action_reverse` | user action | self | `account` |  |  |
| `action_post` | user action | self | `account`, `pos_sale`, `sale_timesheet`, `sale` |  |  |
| `_get_moves_requiring_confirmation` | preparation rule | self | `account` |  | Return the subset of moves that require confirmation before validation. |
| `action_validate_moves_with_confirmation` | user action | self | `account` |  | If 'restrict_mode_hash_table' is enabled or future-dated moves, open a confirmation wizard; otherwise, validate moves directly. |
| `js_assign_outstanding_line` | operation | self, line_id | `account` |  | Called by the 'payment' widget to reconcile a suggested journal item to the present invoice.  :param line_id: The id of the line to reconcile with the current invoice. |
| `js_remove_outstanding_partial` | operation | self, partial_id | `account` |  | Called by the 'payment' widget to remove a reconciled entry to the present invoice.  :param partial_id: The id of an existing partial reconciled with the current invoice. |
| `button_set_checked` | user action | self | `account` |  |  |
| `check_selected_moves` | operation | self | `account` |  |  |
| `set_moves_checked` | operation | self, is_checked | `account` |  |  |
| `button_draft` | user action | self | `account_edi`, `account`, `l10n_eg_edi_eta`, `l10n_es_edi_tbai`, `l10n_fr_pdp`, `l10n_in_edi`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_latam_check`, `l10n_pl_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tr_nilvera_einvoice`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `point_of_sale`, `purchase_stock`, `sale_expense`, `sale`, `stock_account` |  | When going from canceled => draft, we ensure to clear the edi fields so that the invoice can be resent if required. |
| `_get_fields_to_detach` | preparation rule | self | `account_edi_ubl_cii`, `account`, `l10n_eg_edi_eta`, `l10n_es_edi_facturae`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_my_edi`, `l10n_pl_edi`, `l10n_vn_edi_viettel` |  | " Returns a list of field names to detach on resetting an invoice to draft. Can be overridden by other modules to add more fields. |
| `_should_detach_attachments` | internal rule | self | `account`, `l10n_it_edi` |  |  |
| `_detach_attachments` | internal rule | self | `account`, `l10n_it_edi` |  | Called by button_draft to detach specific attachments for the current journal entries to allow regeneration. |
| `_unlink_next_draft_auto_post_moves` | internal rule | self | `account` |  | Deletes auto_post recurrence following each move in self only if that next recurrence is in draft. |
| `_check_draftable` | validation | self | `account`, `l10n_gr_edi_e_invoo` |  |  |
| `button_hash` | user action | self | `account` |  |  |
| `button_request_cancel` | user action | self | `account`, `l10n_hu_edi`, `l10n_in_edi`, `l10n_my_edi`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` |  | Hook allowing the localizations to request a cancellation from the government before cancelling the invoice. |
| `button_cancel` | user action | self | `account_edi`, `account_peppol_response`, `account`, `hr_expense`, `l10n_dk_nemhandel_response`, `l10n_fr_pdp`, `pos_sale`, `sale`, `stock_account` |  |  |
| `action_toggle_block_payment` | user action | self | `account` |  |  |
| `action_activate_currency` | user action | self | `account` |  |  |
| `action_delete_duplicates` | user action | self | `account` |  |  |
| `_get_mail_template` | preparation rule | self | `account` |  | :return: the correct mail template based on the current move type |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `account` |  |  |
| `_get_report_base_filename` | preparation rule | self | `account`, `l10n_sa_edi` |  | Generate the name of the invoice PDF file according to ZATCA business rules: Seller Vat Number (BT-31), Date (BT-2), Time (KSA-25), Invoice Number (BT-1) |
| `_autopost_draft_entries` | internal rule | self, batch_size | `account` |  | This method is called from a cron job. It is used to post entries such as those created by the module account_asset and recurring entries created in _post(). |
| `_cron_account_move_send` | background operation | self, job_count | `account` | model | Process invoices generation and sending asynchronously. :param job_count: maximum number of jobs to process if specified. |
| `_get_available_action_reports` | preparation rule | self, is_invoice_report | `account` |  |  |
| `_is_action_report_available` | internal rule | self, action_report, is_invoice_report | `account` |  |  |
| `_get_suitable_journal_ids` | preparation rule | self, move_type, company | `account` | model | Return the suitable journals for the given move type and company (current company if False). |
| `_get_invoice_filter_type_domain` | preparation rule | self, move_type | `account` | model |  |
| `get_invoice_types` | operation | self, include_receipts | `account` | model |  |
| `is_invoice` | operation | self, include_receipts | `account` |  |  |
| `is_entry` | operation | self | `account` |  |  |
| `is_receipt` | operation | self | `account` |  |  |
| `get_sale_types` | operation | self, include_receipts | `account` | model |  |
| `is_sale_document` | operation | self, include_receipts, move_type | `account` |  |  |
| `get_purchase_types` | operation | self, include_receipts | `account` | model |  |
| `is_purchase_document` | operation | self, include_receipts, move_type | `account` |  |  |
| `get_inbound_types` | operation | self, include_receipts | `account` | model |  |
| `is_inbound` | operation | self, include_receipts | `account` |  |  |
| `get_outbound_types` | operation | self, include_receipts | `account` | model |  |
| `is_outbound` | operation | self, include_receipts | `account` |  |  |
| `_get_action_with_base_document_layout_configurator` | preparation rule | self, report_action | `account` |  |  |
| `_get_installments_data` | preparation rule | self | `account` |  |  |
| `_get_invoice_next_payment_values` | preparation rule | self, custom_amount | `account` |  |  |
| `_get_invoice_portal_extra_values` | preparation rule | self, custom_amount | `account` |  |  |
| `_get_accounting_date` | preparation rule | self, invoice_date, has_tax, lock_dates | `account` |  | Get correct accounting date for previous periods, taking tax lock date and affected journal into account. When registering an invoice in the past, we still want the sequence to be increasing. We then take the last day of the period, depending on the sequence format.  If there is a tax lock date and there are taxes involved, we register the invoice at the last date of the first open period. :param invoice_date (datetime.date): The invoice date :param has_tax (bool): Iff any taxes are involved in the lines of the invoice :param lock_dates: Like result from `_get_violated_lock_dates`;             |
| `_get_violated_lock_dates` | preparation rule | self, invoice_date, has_tax | `account` |  | Get all the lock dates affecting the current invoice_date. :param invoice_date: The invoice date :param has_tax: If any taxes are involved in the lines of the invoice :return: a list of tuples containing the lock dates affecting this move, ordered chronologically. |
| `_get_lock_date_message` | preparation rule | self, invoice_date, has_tax | `account` |  | Get a message describing the latest lock date affecting the specified date. :param invoice_date: The date to be checked :param has_tax: If any taxes are involved in the lines of the invoice :return: a message describing the latest lock date affecting this move and the date it will be          accounted on if posted, or False if no lock dates affect this move. |
| `_move_dict_to_preview_vals` | internal rule | self, move_vals, currency_id | `account` | model |  |
| `_generate_qr_code` | internal rule | self, silent_errors | `account`, `l10n_id`, `l10n_in` |  | Generates and returns a QR-code generation URL for this invoice, raising an error message if something is misconfigured.  The chosen QR generation method is the one set in qr_method field if there is one, or the first eligible one found. If this search had to be performed and and eligible method was found, qr_method field is set to this method before returning the URL. If no eligible QR method could be found, we return None. |
| `_generate_portal_payment_qr` | internal rule | self | `account_payment`, `account` |  |  |
| `_get_portal_payment_link` | preparation rule | self | `account_payment`, `account` |  |  |
| `_generate_and_send` | internal rule | self, force_synchronous, allow_fallback_pdf, **custom_settings | `account` |  | Generate the pdf and electronic format(s) for the current invoices and send them given default settings (on partner or company) or given provided custom_settings. :param force_synchronous: whether to process (as)synchronously (! only relevant for batch sending (multiple invoices)) :param allow_fallback_pdf:  In case of error when generating the documents for invoices, generate a                             proforma PDF report instead. :param custom_settings: custom settings to create the wizard (! only relevant for single sending (one invoice)) (Since default settings are use for batch sending |
| `_get_invoice_pdf_proforma` | preparation rule | self | `account` |  | Generate the Proforma of the invoice. :return dict: the Proforma's data such as {'filename': 'INV_2024_0001_proforma.pdf', 'filetype': 'pdf', 'content': ...} |
| `_get_invoice_legal_documents` | preparation rule | self, filetype, allow_fallback | `account_edi_ubl_cii`, `account`, `l10n_es_edi_facturae`, `l10n_it_edi`, `l10n_vn_edi_viettel_pos` |  | Retrieve the invoice legal document of type filetype. :param filetype: the type of legal document to retrieve. Example: 'pdf'. :param bool allow_fallback: if True, returns a Proforma if the PDF invoice doesn't exist. :return dict: the invoice PDF data such as {'filename': 'INV_2024_0001.pdf', 'filetype': 'pdf', 'content':...} To extend to add more supported filetypes. |
| `_get_invoice_legal_documents_all` | preparation rule | self, allow_fallback | `account` |  | Retrieve the invoice legal attachments: PDF, XML, ... :param bool allow_fallback: if True, returns a Proforma if the PDF invoice doesn't exist. :return list: a list of the attachments data such as [{'filename': 'INV_2024_0001.pdf', 'filetype': 'pdf', 'content': ...}, ...] |
| `_message_set_main_attachment_id` | messaging hook | self, attachments, force, filter_xml | `account_edi`, `account`, `l10n_it` |  |  |
| `_get_invoice_report_filename` | preparation rule | self, extension, report | `account`, `l10n_sa_edi` |  | Get the filename of the generated invoice report with extension file. |
| `_get_invoice_mail_template_dynamic_report_filename` | preparation rule | self, report, extension | `account` |  | Get the filename of the generated invoice report for a dynamic report. |
| `_get_invoice_proforma_pdf_report_filename` | preparation rule | self | `account` |  | Get the filename of the generated proforma PDF invoice report. |
| `_prepare_edi_vals_to_export` | preparation rule | self | `account` |  | The purpose of this helper is to prepare values in order to export an invoice through the EDI system. This includes the computation of the tax details for each invoice line that could be very difficult to handle regarding the computation of the base amount.  :return: A python dict containing default pre-processed values. |
| `_get_discount_allocation_account` | preparation rule | self | `account` |  |  |
| `_get_available_invoice_template_pdf_report_ids` | preparation rule | self | `account` |  | Helper to get available invoice template pdf reports |
| `_is_user_able_to_review` | internal rule | self | `account` |  |  |
| `_field_will_change` | internal rule | self, record, vals, field_name | `account` | model |  |
| `_cleanup_write_orm_values` | internal rule | self, record, vals | `account` | model |  |
| `_disable_recursion` | internal rule | self, container, key, default, target | `account` |  | Apply the context key to all environments inside this context manager.  If the value linked to the key is the same as the target, yield `True`. Check for the key both in the record's context and in the recursion stack.  :param container: deprecated, not used anymore :param key: The context key to apply to the recordsets. :param default: the default value of the context key, if it isn't defined                 yet in the context :param target: the value of the context key meaning that we shouldn't                recurse :return: True iff we should just exit the context manager |
| `_mailing_get_default_domain` | messaging hook | self, mailing | `account` |  |  |
| `_routing_check_route` | internal rule | self, message, message_dict, route, raise_exception | `account` | model |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `account` | model |  |
| `_attachment_fields_to_clear` | internal rule | self | `account` |  |  |
| `_message_post_after_hook` | messaging hook | self, new_message, message_values | `account` |  | This method processes the attachments of a new mail.message. It handles the 3 following situations: (1) receiving an e-mail from a mail alias. In that case, we potentially want to split the attachments into several invoices. (2) receiving an e-mail / posting a message on an existing invoice via the webclient:     (2)(a): If the poster is an internal user, we enhance the invoice with the attachments.     (2)(b): Otherwise, we don't do any further processing. (3) posting a message on an invoice in application code. In that case, don't do anything.  Furthermore, in cases (1) and (2), we decide fo |
| `_creation_subtype` | internal rule | self | `account` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `account` |  |  |
| `_creation_message` | internal rule | self | `account`, `hr_expense` |  |  |
| `_notify_by_email_prepare_rendering_context` | internal rule | self, message, msg_vals, model_description, force_email_company, force_email_lang, force_record_name | `account_peppol`, `account` |  |  |
| `_get_mail_thread_data_attachments` | preparation rule | self | `account`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` |  |  |
| `_conditional_add_to_compute` | internal rule | self, fname, condition | `account` |  |  |
| `_action_invoice_ready_to_be_sent` | internal rule | self | `account`, `sale` |  | Hook allowing custom code when an invoice becomes ready to be sent by mail to the customer. For example, when an EDI document must be sent to the government and be signed by it. |
| `_is_ready_to_be_sent` | internal rule | self | `account_edi`, `account` |  | Helper telling if a journal entry is ready to be sent by mail to the customer.  :return: True if the invoice is ready, False otherwise. |
| `_can_force_cancel` | internal rule | self | `account` |  | Hook to indicate whether it should be possible to force-cancel this invoice, that is, cancel it without waiting for the cancellation request to succeed. |
| `_send_only_when_ready` | internal rule | self | `account` |  |  |
| `_invoice_paid_hook` | internal rule | self | `account`, `event_booth_sale`, `sale` |  | Hook to be overrided called when the invoice moves to the paid state. |
| `_get_lines_onchange_currency` | preparation rule | self | `account`, `stock_account` |  |  |
| `_get_invoice_in_payment_state` | preparation rule | self | `account` | model | Hook to give the state when the invoice becomes fully paid. This is necessary because the users working with only invoicing don't want to see the 'in_payment' state. Then, this method will be overridden in the accountant module to enable the 'in_payment' state. |
| `_get_name_invoice_report` | preparation rule | self | `account`, `l10n_ae`, `l10n_ar`, `l10n_au`, `l10n_cl`, `l10n_gcc_invoice`, `l10n_gr_edi`, `l10n_hu_edi`, `l10n_in`, `l10n_jo_edi`, `l10n_lk_invoice`, `l10n_mu_account`, `l10n_my_edi`, `l10n_nz`, `l10n_sa`, `l10n_th`, `l10n_zm_account` |  | This method need to be inherit by the localizations if they want to print a custom invoice report instead of the default one. For example please review the l10n_ar module |
| `_is_downpayment` | internal rule | self | `account`, `pos_sale`, `sale` |  | Return true if the invoice is a downpayment. Down-payments can be created from a sale order. This method is overridden in the sale order module. |
| `_refunds_origin_required` | internal rule | self | `account`, `l10n_es_edi_tbai` |  |  |
| `_set_reversed_entry` | internal rule | self, credit_note | `account` |  | Try to find the original invoice for a single credit_note. |
| `get_invoice_localisation_fields_required_to_invoice` | operation | self, country_id | `account` | model | Returns the list of fields that needs to be filled when creating an invoice for the selected country. This is required for some flows that would allow a user to request an invoice from the portal. Using these, we can get their information and dynamically create form inputs based for the fields required legally for the company country_id. The returned fields must be of type ir.model.fields in order to handle translations  :param country_id: The country for which we want the fields. :return: an array of ir.model.fields for which the user should provide values. |
| `get_extra_print_items` | operation | self | `account_edi_ubl_cii`, `account`, `l10n_es_edi_facturae`, `l10n_it_edi` |  | Helper to dynamically add items in the 'Print' menu of list and form of account.move. |
| `_get_move_zip_export_docs` | preparation rule | self | `account` |  |  |
| `_get_move_lines_to_report` | preparation rule | self | `account` |  |  |
| `_can_commit` | internal rule |  | `account` |  | Helper to know if we can commit the current transaction or not.  :returns: True if commit is acceptable, False otherwise. |
| `get_import_templates` | operation | self | `account` | model |  |
| `_compute_debit_count` | computation | self | `account_debit_note` | depends: `debit_note_ids` |  |
| `action_view_debit_notes` | user action | self | `account_debit_note` |  |  |
| `action_debit_note` | user action | self | `account_debit_note` |  |  |
| `_compute_edi_state` | computation | self | `account_edi` | depends: `edi_document_ids.state` |  |
| `_compute_edi_show_force_cancel_button` | computation | self | `account_edi` | depends: `edi_document_ids.state` |  |
| `_compute_edi_error_count` | computation | self | `account_edi` | depends: `edi_document_ids.error` |  |
| `_compute_edi_error_message` | computation | self | `account_edi` | depends: `edi_error_count`, `edi_document_ids.error`, `edi_document_ids.blocking_level` |  |
| `_compute_edi_web_services_to_process` | computation | self | `account_edi` | depends: `edi_document_ids`, `edi_document_ids.state`, `edi_document_ids.blocking_level`, `edi_document_ids.edi_format_id`, `edi_document_ids.edi_format_id.name` |  |
| `_check_edi_documents_for_reset_to_draft` | validation | self | `account_edi`, `l10n_es_edi_sii` |  |  |
| `_compute_edi_show_cancel_button` | computation | self | `account_edi`, `l10n_sa_edi` | depends: `edi_document_ids.state`; depends: `state`, `edi_document_ids.state` | Override to hide the EDI Cancellation button at all times for ZATCA Invoices |
| `_compute_edi_show_abandon_cancel_button` | computation | self | `account_edi` | depends: `edi_document_ids.state` |  |
| `_prepare_edi_tax_details` | preparation rule | self, filter_to_apply, filter_invl_to_apply, grouping_key_generator | `account_edi` |  | Compute amounts related to taxes for the current invoice.  :param filter_to_apply:         Optional filter to exclude some tax values from the final results.                                 The filter is defined as a method getting a dictionary as parameter                                 representing the tax values for a single repartition line.                                 This dictionary contains:      'base_line_id':             An account.move.line record.     'tax_id':                   An account.tax record.     'tax_repartition_line_id':  An account.tax.repartition.line record.      |
| `button_force_cancel` | user action | self | `account_edi` |  | Cancel the invoice without waiting for the cancellation request to succeed. |
| `_edi_allow_button_draft` | internal rule | self | `account_edi`, `l10n_es_edi_sii` |  |  |
| `button_cancel_posted_moves` | user action | self | `account_edi` |  | Mark the edi.document related to this move to be canceled. |
| `button_abandon_cancel_posted_posted_moves` | user action | self | `account_edi` |  | Cancel the request for cancellation of the EDI. |
| `_get_edi_document` | preparation rule | self, edi_format | `account_edi` |  |  |
| `_get_edi_attachment` | preparation rule | self, edi_format | `account_edi` |  |  |
| `button_process_edi_web_services` | user action | self | `account_edi` |  |  |
| `action_process_edi_web_services` | user action | self, with_commit | `account_edi` |  |  |
| `_retry_edi_documents_error` | internal rule | self | `account_edi`, `l10n_sa_edi` |  | Called when edi_documents need to be retried. |
| `action_retry_edi_documents_error` | user action | self | `account_edi` |  |  |
| `_process_attachments_for_template_post` | background operation | self, mail_template | `account_edi` |  | Add Edi attachments to templates. |
| `_compute_filename` | computation | self | `account_edi_ubl_cii` | depends: `ubl_cii_xml_file` | Compute the filename based on the uploaded file. |
| `action_invoice_download_ubl` | user action | self | `account_edi_ubl_cii` |  |  |
| `action_group_ungroup_lines_by_tax` | user action | self | `account_edi_ubl_cii` |  | This action allows the user to reload an imported move, grouping or not lines by tax |
| `_ungroup_lines` | internal rule | self | `account_edi_ubl_cii` |  | Ungroup lines using the original file, used to import the move |
| `_group_lines_by_tax` | internal rule | self | `account_edi_ubl_cii` |  | Group lines by tax, based on the invoice lines |
| `_get_line_vals_group_by_tax` | preparation rule | self, partner | `account_edi_ubl_cii` |  | Create a collection of dicts containing the values to create invoice lines, grouped by tax and deferred date if present. :param partner: partner linked to the move |
| `_check_move_for_group_ungroup_lines_by_tax` | validation | self | `account_edi_ubl_cii`, `purchase_edi_ubl_bis3` |  | Perform checks to evaluate if a move is eligible to grouping/ungrouping |
| `_has_lines_grouped` | internal rule | self | `account_edi_ubl_cii` |  | Check if the move has its lines grouped :return: True if lines look like they're grouped, False otherwise |
| `_get_import_file_type` | preparation rule | self, file_data | `account_edi_ubl_cii`, `l10n_anz_ubl_pint`, `l10n_dk_nemhandel`, `l10n_dk_oioubl`, `l10n_es_edi_facturae`, `l10n_hr_edi`, `l10n_it_edi`, `l10n_jp_ubl_pint`, `l10n_my_ubl_pint`, `l10n_ro_edi`, `l10n_sg_ubl_pint`, `l10n_tr_nilvera_einvoice` |  | Identify UBL files. |
| `_unwrap_attachment` | internal rule | self, file_data, recurse | `account_edi_ubl_cii`, `l10n_es_edi_facturae`, `l10n_it_edi` |  | Unwrap UBL AttachedDocument files, which are wrappers around an inner file. |
| `_ubl_parse_attached_document` | internal rule | self, tree | `account_edi_ubl_cii` | model | In UBL, an AttachedDocument file is a wrapper around multiple different UBL files. According to the specifications the original document is stored within the top most Attachment node either as an Attachment/EmbeddedDocumentBinaryObject or (in special cases) a CDATA string stored in Attachment/ExternalReference/Description.  We must parse this before passing the original file to the decoder to figure out how best to handle it. |
| `_get_edi_decoder` | preparation rule | self, file_data, new | `account_edi_ubl_cii`, `l10n_es_edi_facturae`, `l10n_hu_edi_receive`, `l10n_it_edi`, `l10n_pl_edi` |  |  |
| `_need_ubl_cii_xml` | internal rule | self, ubl_cii_format | `account_edi_ubl_cii`, `l10n_dk_nemhandel`, `l10n_fr_pdp_pos`, `l10n_fr_pdp` |  |  |
| `_is_exportable_as_self_invoice` | internal rule | self | `account_edi_ubl_cii` |  |  |
| `_get_line_vals_list` | preparation rule | self, lines_vals | `account_edi_ubl_cii` | model | Get invoice line values list.  :param list[tuple] lines_vals: List of values `[(name, qty, price, tax), ...]`. :returns: List of invoice line values. |
| `_get_specific_tax` | preparation rule | self, name, amount_type, amount, tax_type | `account_edi_ubl_cii` |  |  |
| `_compute_authorized_transaction_ids` | computation | self | `account_payment` | depends: `transaction_ids` |  |
| `_compute_transaction_count` | computation | self | `account_payment` | depends: `transaction_ids` |  |
| `_compute_amount_paid` | computation | self | `account_payment` | depends: `transaction_ids` | Sum all the transaction amount for which state is in 'authorized' or 'done' |
| `_has_to_be_paid` | internal rule | self | `account_payment` |  |  |
| `_get_online_payment_error` | preparation rule | self | `account_payment` |  | Returns the appropriate error message to be displayed if _has_to_be_paid() method returns False. |
| `get_portal_last_transaction` | operation | self | `account_payment` | private |  |
| `payment_action_capture` | operation | self | `account_payment` |  | Capture all transactions linked to this invoice. |
| `payment_action_void` | operation | self | `account_payment` |  | Void all transactions linked to this invoice. |
| `action_view_payment_transactions` | user action | self | `account_payment` |  |  |
| `_get_default_payment_link_values` | preparation rule | self | `account_payment` |  |  |
| `_interco_filter_moves` | internal rule | self | `account_payment_interco` |  |  |
| `_check_interco_clearing` | validation | self, payments | `account_payment_interco` |  |  |
| `action_cancel_peppol_documents` | user action | self | `account_peppol` |  |  |
| `_compute_peppol_move_state` | computation | self | `account_peppol_response`, `account_peppol`, `l10n_fr_pdp` | depends: `state`; depends: `state`, `peppol_response_ids.peppol_state`; depends: `peppol_response_ids`, `peppol_response_ids.peppol_state` |  |
| `_compute_peppol_is_sent` | computation | self | `account_peppol` | depends: `peppol_move_state` |  |
| `action_peppol_cancel_and_remove_sequence` | user action | self | `account_peppol` |  |  |
| `action_peppol_reset_documents` | user action | self, ids_to_delete | `account_peppol` |  |  |
| `_compute_peppol_can_send_response` | computation | self | `account_peppol_response`, `l10n_fr_pdp` | depends: `peppol_response_ids.peppol_state`; depends: `pdp_can_send_response` |  |
| `action_peppol_send_approval_response` | user action | self | `account_peppol_response` |  |  |
| `action_peppol_open_rejection_wizard` | user action | self | `account_peppol_response` |  |  |
| `action_open_peppol_reponses` | user action | self | `account_peppol_response` |  |  |
| `_compute_team_id` | computation | self | `sale` | depends: `invoice_user_id` |  |
| `_compute_origin_so_count` | computation | self | `sale` | depends: `line_ids.sale_line_ids` |  |
| `_compute_sale_warning_text` | computation | self | `sale` | depends: `partner_id.name`, `partner_id.sale_warn_msg`, `invoice_line_ids.product_id.sale_line_warn_msg`, `invoice_line_ids.product_id.display_name` |  |
| `action_view_source_sale_orders` | user action | self | `sale` |  |  |
| `_get_sale_order_invoiced_amount` | preparation rule | self, order | `sale` |  | Consider all lines on any invoice in self that stem from the sales order `order`. (All those invoices belong to order.company_id) This function returns the sum of the totals of all those lines. Note that this amount may be bigger than `order.amount_total`. |
| `_stock_account_prepare_realtime_out_lines_vals` | internal rule | self | `stock_account` |  | Prepare values used to create the journal items (account.move.line) corresponding to the Cost of Good Sold lines (COGS) for customer invoices.  Example:  Buy a product having a cost of 9 being a storable product and having a perpetual valuation in FIFO. Sell this product at a price of 10. The customer invoice's journal entries looks like:  Account                                     \| Debit \| Credit --------------------------------------------------------------- 200000 Product Sales                        \|       \| 10.0 --------------------------------------------------------------- 101200 |
| `_get_anglo_saxon_price_ctx` | preparation rule | self | `sale_stock`, `stock_account` |  | To be overriden in modules overriding _get_cogs_value to optimize computations that only depend on account.move and not account.move.line |
| `_stock_account_get_last_step_stock_moves` | internal rule | self | `point_of_sale`, `purchase_stock`, `sale_stock`, `stock_account` |  | To be overridden for customer invoices and vendor bills in order to return the stock moves related to the invoices in self. |
| `_get_invoiced_lot_values` | preparation rule | self | `point_of_sale`, `sale_stock`, `stock_account` |  | Get and prepare data to show a table of invoiced lot on the invoice's report. |
| `_compute_nb_expenses` | computation | self | `hr_expense` |  |  |
| `_check_expense_ids` | validation | self | `hr_expense` | constrains: `expense_ids` |  |
| `action_open_expense` | user action | self | `hr_expense` |  |  |
| `_compute_origin_pos_count` | computation | self | `point_of_sale` | depends: `pos_order_ids` |  |
| `action_view_source_pos_orders` | user action | self | `point_of_sale` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_l10n_gcc_get_invoice_title` | internal rule | self | `l10n_ae`, `l10n_gcc_invoice`, `l10n_sa_edi`, `l10n_sa` |  | To be overriden by inheriting modules implementing a custom invoice title |
| `_num2words` | internal rule | self, number, lang | `l10n_gcc_invoice` |  |  |
| `_load_narration_translation` | internal rule | self | `l10n_gcc_invoice` |  |  |
| `_l10n_ae_is_simplified` | internal rule | self | `l10n_ae` |  | Returns True if the customer is an individual, i.e: The invoice is B2C |
| `_compute_l10n_latam_manual_document_number` | computation | self | `l10n_latam_invoice_document` | depends: `l10n_latam_document_type_id`, `journal_id` | Indicates if this document type uses a sequence or if the numbering is made manually |
| `_is_manual_document_number` | internal rule | self | `l10n_ar`, `l10n_cl`, `l10n_latam_invoice_document` |  | Document number should be manual input by user when the journal use documents and  * if sales journal and not a ARCA pos (liquido producto case) * if purchase journal and not a ARCA pos (regular case of vendor bills)  All the other cases the number should be automatic set, wiht only one exception, for pre-printed/online ARCA POS type, the first numeber will be always set manually by the user and then will be computed automatically from there |
| `_compute_l10n_latam_use_documents` | computation | self | `l10n_latam_invoice_document` | depends: `journal_id` |  |
| `_compute_l10n_latam_document_number` | computation | self | `l10n_latam_invoice_document` | depends: `name` |  |
| `_inverse_l10n_latam_document_number` | on change | self | `l10n_ar`, `l10n_latam_invoice_document`, `l10n_pe` | onchange: `l10n_latam_document_type_id`, `l10n_latam_document_number`, `partner_id` | Inherit to complete the l10n_latam_document_number with the expected 8 characters after that a '-'  After formatting the document number with zfill(8), the name field is also synchronized to ensure both fields remain consistent.  Example: Change F01-32 by F01-00000032, to avoid incorrect values on the reports |
| `_onchange_l10n_latam_document_type_id` | on change | self | `l10n_latam_invoice_document` | onchange: `l10n_latam_document_type_id` |  |
| `_deduce_sequence_number_reset` | internal rule | self, name | `l10n_latam_invoice_document`, `l10n_lk_invoice` | model | LK sequences never reset. |
| `_skip_format_document_number` | internal rule | self | `l10n_ec`, `l10n_latam_invoice_document` |  | Hook to be overridden in localisation |
| `_check_l10n_latam_documents` | validation | self | `l10n_latam_invoice_document` | constrains: `state`, `l10n_latam_document_type_id` | This constraint checks that if a invoice is posted and does not have a document type configured will raise an error. This only applies to invoices related to journals that has the "Use Documents" set as True. And if the document type is set then check if the invoice number has been set, because a posted invoice without a document number is not valid in the case that the related journals has "Use Docuemnts" set as True |
| `_check_invoice_type_document_type` | validation | self | `l10n_ar`, `l10n_latam_invoice_document` | constrains: `move_type`, `l10n_latam_document_type_id` | LATAM module define that we are not able to use debit_note or invoice document types in an invoice refunds, However for Argentinian Document Type's 99 (internal type = invoice) we are able to used in a refund invoices.  In this method we exclude the argentinian documents that can be used as invoice and refund from the generic constraint |
| `_get_l10n_latam_documents_domain` | preparation rule | self | `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_latam_invoice_document`, `l10n_pe`, `l10n_uy` |  | If this is a reversal or debit, suggest only related subtypes |
| `_compute_l10n_latam_available_document_types` | computation | self | `l10n_latam_invoice_document` | depends: `journal_id`, `partner_id`, `company_id`, `move_type`, `debit_origin_id` |  |
| `_compute_l10n_latam_document_type` | computation | self | `l10n_ar`, `l10n_br`, `l10n_latam_invoice_document` | depends: `l10n_latam_available_document_type_ids` | We correct the default document type in vendor bills in case the partner is foreign (code 8) so that it is always 'Foreign invoices and receipts'. |
| `_search_l10n_latam_use_documents` | search rule | self, operator, value | `l10n_latam_invoice_document` |  |  |
| `_l10n_ar_get_document_number_parts` | internal rule | self, document_number, document_type_code | `l10n_ar` | model |  |
| `_check_moves_use_documents` | validation | self | `l10n_ar` | constrains: `move_type`, `journal_id` | Do not let to create not invoices entries in journals that use documents |
| `_get_afip_invoice_concepts` | preparation rule | self | `l10n_ar` |  | Return the list of values of the selection field. |
| `_compute_l10n_ar_afip_concept` | computation | self | `l10n_ar` | depends: `invoice_line_ids`, `invoice_line_ids.product_id`, `invoice_line_ids.product_id.type`, `journal_id` |  |
| `_get_concept` | preparation rule | self | `l10n_ar` |  | Method to get the concept of the invoice considering the type of the products on the invoice |
| `_get_l10n_ar_codes_used_for_inv_and_ref` | preparation rule | self | `l10n_ar` | model | List of document types that can be used as an invoice and refund. This list can be increased once needed and demonstrated. As far as we've checked document types of wsfev1 don't allow negative amounts so, for example document 61 could not be used as refunds. |
| `_check_argentinean_invoice_taxes` | validation | self | `l10n_ar` |  |  |
| `_set_afip_service_dates` | internal rule | self | `l10n_ar` |  |  |
| `_set_afip_responsibility` | internal rule | self | `l10n_ar` |  | We save the information about the receptor responsability at the time we validate the invoice, this is necessary because the user can change the responsability after that any time |
| `_onchange_afip_responsibility` | on change | self | `l10n_ar` | onchange: `partner_id` |  |
| `_onchange_partner_journal` | on change | self | `l10n_ar` | onchange: `partner_id` | This method is used when the invoice is created from the sale or subscription |
| `_get_formatted_sequence` | preparation rule | self, number | `l10n_ar` |  |  |
| `_l10n_ar_get_amounts` | internal rule | self, base_lines | `l10n_ar` |  | Method used to prepare data to present amounts and taxes related amounts when creating an electronic invoice for argentinean and the txt files for digital VAT books. Only take into account the argentinean taxes |
| `_get_vat` | preparation rule | self, base_lines | `l10n_ar` |  | Applies on wsfe web service and in the VAT digital books |
| `_l10n_ar_get_invoice_totals_for_report` | internal rule | self | `l10n_ar` |  | If the invoice document type indicates that vat should not be detailed in the printed report (result of _l10n_ar_include_vat()) then we overwrite tax_totals field so that includes taxes in the total amount, otherwise it would be showing amount_untaxed in the amount_total |
| `_l10n_ar_get_invoice_custom_tax_summary_for_report` | internal rule | self | `l10n_ar` |  | Get a new tax details for RG 5614/2024 to show ARCA VAT and Other National Internal Taxes. |
| `_l10n_ar_include_vat` | internal rule | self | `l10n_ar` |  |  |
| `_l10n_ar_is_transparency_document` | internal rule | self | `l10n_ar` |  |  |
| `_l10n_ar_is_tax_group_other_national_ind_tax` | internal rule | self, tax_group | `l10n_ar` | model |  |
| `_l10n_ar_is_tax_group_vat` | internal rule | self, tax_group | `l10n_ar` | model |  |
| `_l10n_ar_is_tax_group_iibb_perception` | internal rule | self, tax_group | `l10n_ar` | model |  |
| `_compute_website_id` | computation | self | `website_sale` | depends: `partner_id` |  |
| `_compute_l10n_ar_withholding_ids` | computation | self | `l10n_ar_withholding` | depends: `line_ids` |  |
| `_get_invoice_reference_be_partner` | preparation rule | self | `l10n_be` |  | This computes the reference based on the belgian national standard “OGM-VCS”. For instance, if an invoice is issued for the partner with internal reference 'food buyer 654', the digits will be extracted and used as the data. This will lead to a check number equal to 72 and the reference will be '+++000/0000/65472+++'. If no reference is set for the partner, its id in the database will be used. |
| `_get_invoice_reference_be_invoice` | preparation rule | self | `l10n_be` |  | This computes the reference based on the belgian national standard “OGM-VCS”. The data of the reference is the database id number of the invoice. For instance, if an invoice is issued with id 654, the check number is 72 so the reference will be '+++000/0000/65472+++'. |
| `reflect_cancelled_sol` | operation | self, isCancelled | `pos_sale` |  |  |
| `_l10n_bg_document_type_selection_values` | internal rule | self | `l10n_bg_ledger` |  |  |
| `_compute_l10n_bg_document_type` | computation | self | `l10n_bg_ledger` | depends: `journal_id`, `move_type` |  |
| `_compute_l10n_bg_document_number` | computation | self | `l10n_bg_ledger` | depends: `l10n_bg_document_type`, `move_type`, `state`, `ref`, `name` |  |
| `_compute_l10n_ch_qr_is_valid` | computation | self | `l10n_ch` | depends: `partner_id`, `currency_id` |  |
| `get_l10n_ch_qrr_number` | operation | self | `l10n_ch` |  | Generates the QRR reference. QRR references are 27 characters long.  The invoice sequence number is used, removing each of its non-digit characters, and pad the unused spaces on the left of this number with zeros. The last digit is a checksum (mod10r). |
| `_compute_qrr_number` | computation | self, invoice_ref | `l10n_ch` | model |  |
| `_get_invoice_reference_ch_invoice` | preparation rule | self | `l10n_ch` |  | This sets QRR reference number which is generated based on customer's `Bank Account` and set it as `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard |
| `_get_invoice_reference_ch_partner` | preparation rule | self | `l10n_ch` |  | This sets QRR reference number which is generated based on customer's `Bank Account` and set it as `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard |
| `space_qrr_reference` | operation | self, qrr_ref | `l10n_ch` | model | Makes the provided QRR reference human-friendly, spacing its elements by blocks of 5 from right to left. |
| `space_scor_reference` | operation | self, iso11649_ref | `l10n_ch` | model | Makes the provided SCOR reference human-friendly, spacing its elements by blocks of 5 from right to left. |
| `l10n_ch_action_print_qr` | operation | self | `l10n_ch` |  | Checks that all invoices can be printed in the QR format. If so, launches the printing action. Else, triggers the l10n_ch wizard that will display the informations. |
| `_l10n_ch_dispatch_invoices_to_print` | internal rule | self | `l10n_ch` |  |  |
| `_check_l10n_latam_document_number_is_numeric` | validation | self | `l10n_cl` | constrains: `l10n_latam_document_number` |  |
| `_check_document_types_post` | validation | self | `l10n_cl` |  |  |
| `_l10n_cl_onchange_journal` | on change | self | `l10n_cl` | onchange: `journal_id` |  |
| `_l10n_cl_get_formatted_sequence` | internal rule | self, number | `l10n_cl` |  |  |
| `_format_lang_totals` | internal rule | self, value, currency | `l10n_cl` |  |  |
| `_l10n_cl_get_invoice_totals_for_report` | internal rule | self | `l10n_cl` |  |  |
| `_l10n_cl_include_sii` | internal rule | self | `l10n_cl` |  |  |
| `_l10n_cl_get_amounts` | internal rule | self | `l10n_cl` |  | This method is used to calculate the amount and taxes required in the Chilean localization electronic documents. |
| `_l10n_cl_get_withholdings` | internal rule | self | `l10n_cl` |  | This method calculates the section of withholding taxes, or 'other' taxes for the Chilean electronic invoices. These taxes are not VAT taxes in general; they are special taxes (for example, alcohol or sugar-added beverages, withholdings for meat processing, fuel, etc. The taxes codes used are included here: [15, 17, 18, 19, 24, 25, 26, 27, 271] http://www.sii.cl/declaraciones_juradas/ddjj_3327_3328/cod_otros_imp_retenc.pdf The need of the tax is not just the amount, but the code of the tax, the percentage amount and the amount :return: |
| `_float_repr_float_round` | internal rule | self, value, decimal_places | `l10n_cl` |  |  |
| `_check_fapiao` | validation | self | `l10n_cn` | constrains: `fapiao` |  |
| `check_cn2an` | operation | self | `l10n_cn` | model |  |
| `_convert_to_amount_in_word` | internal rule | self, number | `l10n_cn` | model | Convert number to `amount in words` for Chinese financial usage. |
| `_count_attachments` | internal rule | self | `l10n_cn` |  |  |
| `_onchange_purchase_auto_complete` | on change | self | `purchase` | onchange: `purchase_vendor_bill_id`, `purchase_id` | Load from either an old purchase order, either an old vendor bill.  When setting a 'purchase.bill.union' in 'purchase_vendor_bill_id': * If it's a vendor bill, 'invoice_vendor_bill_id' is set and the loading is done by '_onchange_invoice_vendor_bill'. * If it's a purchase order, 'purchase_id' is set and this method will load lines.  /!\ All this not-stored fields must be empty at the end of this function. |
| `_compute_is_purchase_matched` | computation | self | `purchase` | depends: `line_ids.purchase_line_id` |  |
| `_compute_origin_po_count` | computation | self | `purchase` | depends: `line_ids.purchase_line_id` |  |
| `_compute_purchase_order_name` | computation | self | `purchase` | depends: `purchase_order_count` |  |
| `_compute_purchase_warning_text` | computation | self | `purchase` | depends: `partner_id.name`, `partner_id.purchase_warn_msg`, `invoice_line_ids.product_id.purchase_line_warn_msg`, `invoice_line_ids.product_id.display_name` |  |
| `action_purchase_matching` | user action | self | `purchase` |  |  |
| `action_view_source_purchase_orders` | user action | self | `purchase` |  |  |
| `_add_purchase_order_lines` | internal rule | self, purchase_order_lines | `purchase` |  | Creates new invoice lines from purchase order lines |
| `_find_matching_subset_po_lines` | internal rule | self, po_lines_with_amount, goal_total, timeout | `purchase` |  | Finds the purchase order lines adding up to the goal amount.  The problem of finding the subset of `po_lines_with_amount` which sums up to `goal_total` reduces to the 0-1 Knapsack problem. The dynamic programming approach to solve this problem is most of the time slower than this because identical sub-problems don't arise often enough. It returns the list of purchase order lines which sum up to `goal_total` or an empty list if multiple or no solutions were found.  :param po_lines_with_amount: a dict (str: float\|recordset) containing:     * line: an `purchase.order.line`     * amount_to_invoic |
| `_find_matching_po_and_inv_lines` | internal rule | self, po_lines, inv_lines, timeout | `purchase` |  | Finds purchase order lines that match some of the invoice lines.  We try to find a purchase order line for every invoice line matching on the unit price and having at least the same quantity to invoice.  :param po_lines: list of purchase order lines that can be matched :param inv_lines: list of invoice lines to be matched :param timeout: how long this function can run before we consider it too long :return: a tuple (list, list) containing:     * matched 'purchase.order.line'     * tuple of purchase order line ids and their matched 'account.move.line' |
| `_set_purchase_orders` | internal rule | self, purchase_orders, force_write | `purchase` |  | Link the given purchase orders to this vendor bill and add their lines as invoice lines.  :param purchase_orders: a list of purchase orders to be linked to this vendor bill :param force_write: whether to delete all existing invoice lines before adding the vendor bill lines |
| `_match_purchase_orders` | internal rule | self, po_references, partner_id, amount_total, from_ocr, timeout | `purchase` |  | Tries to match open purchase order lines with this invoice given the information we have.  :param po_references: a list of potential purchase order references/names :param partner_id: the vendor id inferred from the vendor bill :param amount_total: the total amount of the vendor bill :param from_ocr: indicates whether this vendor bill was created from an OCR scan (less reliable) :param timeout: the max time the line matching algorithm can take before timing out :return: tuple (str, recordset, dict) containing:     * the match method:         * `total_match`: purchase order reference(s) and tot |
| `_get_invoice_reference_dk_fik` | preparation rule | self, prefix, max_digits | `l10n_dk_fik` |  |  |
| `_get_invoice_reference_dk_fik_71_invoice` | preparation rule | self | `l10n_dk_fik` |  |  |
| `_get_invoice_reference_dk_fik_75_invoice` | preparation rule | self | `l10n_dk_fik` |  |  |
| `_compute_nemhandel_move_state` | computation | self | `l10n_dk_nemhandel_response`, `l10n_dk_nemhandel` | depends: `state`; depends: `state`, `nemhandel_response_ids.nemhandel_state` |  |
| `_get_ubl_cii_builder_from_xml_tree` | preparation rule | self, tree | `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_tr_nilvera_einvoice` | model |  |
| `action_cancel_nemhandel_documents` | user action | self | `l10n_dk_nemhandel` |  |  |
| `_compute_nemhandel_can_send_response` | computation | self | `l10n_dk_nemhandel_response` | depends: `nemhandel_response_ids.nemhandel_state` |  |
| `action_nemhandel_send_approval_response` | user action | self | `l10n_dk_nemhandel_response` |  |  |
| `action_nemhandel_open_rejection_wizard` | user action | self | `l10n_dk_nemhandel_response` |  |  |
| `action_open_nemhandel_reponses` | user action | self | `l10n_dk_nemhandel_response` |  |  |
| `_get_l10n_ec_documents_allowed` | preparation rule | self, identification_code | `l10n_ec` | model |  |
| `_get_ec_formatted_sequence` | preparation rule | self, number | `l10n_ec` |  |  |
| `_compute_eta_long_id` | computation | self | `l10n_eg_edi_eta` | depends: `l10n_eg_eta_json_doc_file` |  |
| `_compute_eta_qr_code_str` | computation | self | `l10n_eg_edi_eta` | depends: `invoice_date`, `l10n_eg_uuid`, `l10n_eg_long_id` |  |
| `_compute_eta_response_data` | computation | self | `l10n_eg_edi_eta` | depends: `l10n_eg_eta_json_doc_file` |  |
| `action_post_sign_invoices` | user action | self | `l10n_eg_edi_eta` |  |  |
| `action_get_eta_invoice_pdf` | user action | self | `l10n_eg_edi_eta` |  | This is a pdf with the structure from the government.  While we can use our own format, some clients appreciate this to verify that all the data is there in case of confusion. |
| `_l10n_eg_edi_exchange_currency_rate` | internal rule | self | `l10n_eg_edi_eta` |  | Calculate the rate based on the balance and amount_currency, so we recuperate the one used at the time |
| `_compute_l10n_es_is_simplified` | computation | self | `l10n_es_pos`, `l10n_es` | depends: `partner_id`, `line_ids.balance`, `reversed_entry_id` |  |
| `_l10n_es_is_dua` | internal rule | self | `l10n_es_edi_sii`, `l10n_es` |  |  |
| `_compute_l10n_es_edi_facturae_reason_code` | computation | self | `l10n_es_edi_facturae` | depends: `country_code` |  |
| `_compute_l10n_es_payment_means` | computation | self | `l10n_es_edi_facturae` | depends: `country_code` |  |
| `_l10n_es_edi_facturae_export_data_check` | internal rule | self | `l10n_es_edi_facturae` |  | This function checks the Settings, Company, Partners involved in the sending activity and returns an errors dictionary ready for the actionable_errors widget to display. |
| `_l10n_es_edi_facturae_get_default_enable` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_filename` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_tax_period` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_refunded_invoices` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_corrective_data` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_administrative_centers` | internal rule | self, partner | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_get_tax_node_from_tax_data` | internal rule | self, values, round | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_facturae_convert_payment_terms_to_installments` | internal rule | self | `l10n_es_edi_facturae` |  | Convert the payments terms to a list of <Installment> elements to be used in the <PaymentDetails> node of the Facturae XML generation. |
| `_l10n_es_edi_facturae_prepare_inv_line` | internal rule | self, base_line, aggregated_values | `l10n_es_edi_facturae` |  | Convert the invoice lines to a list of items required for the Facturae xml generation  :return: A tuple containing the Face items, the taxes and the invoice totals data. |
| `_l10n_es_edi_facturae_export_facturae` | internal rule | self | `l10n_es_edi_facturae` |  | Produce the Facturae XML data for the invoice.  :return: (data needed to render the full template, data needed to render the signature template) |
| `_l10n_es_edi_facturae_render_facturae` | internal rule | self | `l10n_es_edi_facturae` |  | Produce the Facturae XML file for the invoice.  :return: rendered xml file string. :rtype:  str |
| `_import_invoice_facturae` | internal rule | self, invoice, file_data, new | `l10n_es_edi_facturae` |  |  |
| `_import_get_partner` | internal rule | self, tree, is_bill | `l10n_es_edi_facturae` |  |  |
| `_import_extract_partner_values` | internal rule | self, party_node | `l10n_es_edi_facturae` |  |  |
| `_import_create_or_retrieve_partner` | internal rule | self, partner_vals | `l10n_es_edi_facturae` |  |  |
| `_import_invoice_facturae_invoice` | internal rule | self, invoice, partner, tree | `l10n_es_edi_facturae` |  |  |
| `_import_invoice_fill_lines` | internal rule | self, invoice, tree, ref_multiplier | `l10n_es_edi_facturae` |  |  |
| `_import_fill_invoice_line_taxes` | internal rule | self, invoice, line_vals, tax_ids, tax_nodes, is_withheld, is_purchase | `l10n_es_edi_facturae` |  |  |
| `_search_tax_for_import` | search rule | self, company, amount, is_fixed, is_withheld, is_purchase, price_included | `l10n_es_edi_facturae` |  |  |
| `_search_product_for_import` | search rule | self, item_description | `l10n_es_edi_facturae` |  |  |
| `action_invoice_download_facturae` | user action | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_facturae_sign_xml` | internal rule | self, edi_data, signature_data | `l10n_es_edi_facturae` |  | Signs the given XML data with the certificate and private key.  :param etree._Element edi_data: The XML data to sign. :param dict signature_data: The signature data to use. :return: The signed XML data string. :rtype: str |
| `_compute_l10n_es_edi_is_required` | computation | self | `l10n_es_edi_sii` | depends: `move_type`, `company_id`, `invoice_line_ids.tax_ids` |  |
| `_l10n_es_edi_get_period` | internal rule | self | `l10n_es_edi_sii` |  |  |
| `_compute_l10n_es_tbai_state` | computation | self | `l10n_es_edi_tbai` | depends: `l10n_es_tbai_post_document_id.state`, `l10n_es_tbai_cancel_document_id.state` |  |
| `_compute_l10n_es_tbai_is_required` | computation | self | `l10n_es_edi_tbai` | depends: `move_type`, `company_id` |  |
| `_l10n_es_tbai_unlink_except_in_chain` | internal rule | self | `l10n_es_edi_tbai` | ondelete |  |
| `_l10n_es_tbai_check_can_send` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_attachment_name` | internal rule | self, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_create_edi_document` | internal rule | self, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_post_document_in_chatter` | internal rule | self, message, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_lock_move` | internal rule | self | `l10n_es_edi_tbai` |  | Acquire a write lock on the invoices in self. |
| `l10n_es_tbai_resend_bill` | operation | self | `l10n_es_edi_tbai` |  |  |
| `l10n_es_tbai_send_bill` | operation | self | `l10n_es_edi_tbai` |  |  |
| `l10n_es_tbai_cancel` | operation | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_post` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_values` | internal rule | self, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_attachment_values` | internal rule | self, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_invoice_values` | internal rule | self, cancel | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_credit_note_values` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_tbai_get_vendor_bill_values_batuz` | internal rule | self | `l10n_es_edi_tbai` |  | For the vendor bills for Bizkaia, the structure is different than the regular Ticketbai XML (LROE) |
| `_l10n_es_tbai_get_vendor_bill_tax_values` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_edi_verifactu_clave_regimen_selection` | internal rule | self | `l10n_es_edi_verifactu` | model |  |
| `_l10n_es_edi_verifactu_get_tax_applicability` | internal rule | self | `l10n_es_edi_verifactu` |  | Currently we only support a single Veri*Factu Tax Applicability per Veri*Factu document. In `_check_record_values` of model 'l10n_es_edi_verifactu.document' we check: There is only a single Veri*Factu Tax Applicability on the whole move. |
| `_l10n_es_edi_verifactu_get_available_clave_regimens_map` | internal rule | self | `l10n_es_edi_verifactu` | model | Return dictionary (Veri*Factu Tax Applicability -> set(operation types)) |
| `_l10n_es_edi_verifactu_get_suggested_clave_regimen` | internal rule | self | `l10n_es_edi_verifactu` |  | Currently we only support a single Clave Regimen per Veri*Factu document. |
| `_compute_l10n_es_edi_verifactu_available_clave_regimens` | computation | self | `l10n_es_edi_verifactu` | depends: `invoice_line_ids.tax_ids` |  |
| `_compute_l10n_es_edi_verifactu_clave_regimen` | computation | self | `l10n_es_edi_verifactu` | depends: `invoice_line_ids.tax_ids` |  |
| `_compute_l10n_es_edi_verifactu_state` | computation | self | `l10n_es_edi_verifactu` | depends: `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state` |  |
| `_compute_l10n_es_edi_verifactu_qr_code` | computation | self | `l10n_es_edi_verifactu` | depends: `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.json_attachment_id` |  |
| `_compute_l10n_es_edi_verifactu_warning` | computation | self | `l10n_es_edi_verifactu` | depends: `state`, `l10n_es_edi_verifactu_state`, `l10n_es_edi_verifactu_document_ids`, `l10n_es_edi_verifactu_document_ids.state`, `l10n_es_edi_verifactu_document_ids.errors` |  |
| `_compute_l10n_es_edi_verifactu_show_cancel_button` | computation | self | `l10n_es_edi_verifactu` | depends: `l10n_es_edi_verifactu_state` |  |
| `_l10n_es_edi_verifactu_action_go_to_journal_entry` | internal rule | self, move | `l10n_es_edi_verifactu` | model |  |
| `l10n_es_edi_verifactu_button_cancel` | operation | self | `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_check` | internal rule | self, cancellation | `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_get_record_values` | internal rule | self, cancellation | `l10n_es_edi_verifactu_pos`, `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_create_documents` | internal rule | self, cancellation | `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_mark_for_next_batch` | internal rule | self, cancellation | `l10n_es_edi_verifactu` |  |  |
| `number2numeric` | operation | self, number | `l10n_fi` | model |  |
| `get_finnish_check_digit` | operation | self, base_number | `l10n_fi` | model |  |
| `get_rf_check_digits` | operation | self, base_number | `l10n_fi` | model |  |
| `compute_payment_reference_finnish` | operation | self, number | `l10n_fi` | model |  |
| `compute_payment_reference_finnish_rf` | operation | self, number | `l10n_fi` | model |  |
| `_get_invoice_reference_fi_rf_invoice` | preparation rule | self | `l10n_fi` |  |  |
| `_get_invoice_reference_fi_rf_partner` | preparation rule | self | `l10n_fi` |  |  |
| `_get_invoice_reference_fi_invoice` | preparation rule | self | `l10n_fi` |  |  |
| `_get_invoice_reference_fi_partner` | preparation rule | self | `l10n_fi` |  |  |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `l10n_fr_account` | model |  |
| `_compute_l10n_fr_is_company_french` | computation | self | `l10n_fr_account` | depends: `company_id.country_code` |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `l10n_fr_pdp` | model |  |
| `_compute_pdp_lifecycle_residual` | computation | self | `l10n_fr_pdp` | depends: `line_ids.matched_debit_ids.debit_move_id`, `line_ids.matched_credit_ids.credit_move_id`, `peppol_message_uuid`, `peppol_response_ids`, `pdp_ppf_move_state`, `payment_state` |  |
| `_compute_pdp_ppf_state` | computation | self | `l10n_fr_pdp` | depends: `peppol_message_uuid`, `peppol_move_state`, `peppol_response_ids`, `peppol_response_ids.peppol_state`, `peppol_response_ids.response_code` |  |
| `_compute_pdp_can_send_response` | computation | self | `l10n_fr_pdp` | depends: `peppol_move_state`, `peppol_message_uuid` |  |
| `_compute_pdp_uses_pdp` | computation | self | `l10n_fr_pdp` | depends: `company_id` |  |
| `_compute_pdp_is_sent` | computation | self | `l10n_fr_pdp` | depends: `peppol_is_sent`, `pdp_uses_pdp`, `move_type` |  |
| `_pdp_get_reconciled_amls` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_payment_date` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_paid_amount` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_paid_lifecycle_total_amount` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_response_status` | internal rule | self | `l10n_fr_pdp` |  | Return the PDP response status of the message |
| `_pdp_get_tax_extract_state` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_pdp_get_lifecycle_state` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_default_notes` | internal rule | self | `l10n_fr_pdp` |  |  |
| `action_pdp_open_response_wizard` | user action | self, **wizard_kwargs | `l10n_fr_pdp` |  |  |
| `_message_track` | messaging hook | self, fields_iter, initial_values_dict | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_message_log_ereporting_status` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_compute_l10n_fr_pdp_last_flow_id` | computation | self | `l10n_fr_pdp` | depends: `commercial_partner_id`, `company_id`, `date`, `l10n_fr_pdp_flow_10_operation_type`, `l10n_fr_pdp_flow_10_report_type`, `l10n_fr_pdp_has_error`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type` |  |
| `_compute_l10n_fr_pdp_status` | computation | self | `l10n_fr_pdp` | depends: `commercial_partner_id`, `company_id`, `date`, `l10n_fr_pdp_flow_10_report_type`, `l10n_fr_pdp_has_error`, `l10n_fr_pdp_last_flow_id`, `l10n_fr_pdp_last_flow_id.state`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `state` |  |
| `_compute_l10n_fr_pdp_has_error` | computation | self | `l10n_fr_pdp` | depends: `company_id`, `commercial_partner_id`, `l10n_fr_pdp_flow_10_report_type`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `name`, `state` |  |
| `_compute_l10n_fr_pdp_flow_10_operation_type` | computation | self | `l10n_fr_pdp` | depends: `company_id`, `commercial_partner_id`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `state` |  |
| `_compute_l10n_fr_pdp_flow_10_report_type` | computation | self | `l10n_fr_pdp` | depends: `date`, `company_id`, `commercial_partner_id`, `l10n_fr_pdp_flow_10_operation_type`, `line_ids.matched_credit_ids.credit_move_id`, `line_ids.matched_debit_ids.debit_move_id`, `move_type`, `state` |  |
| `_compute_l10n_fr_pdp_error_message` | computation | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_matched_transactions` | internal rule | self | `l10n_fr_pdp_pos`, `l10n_fr_pdp` |  | If self is a payment move, this method returns the transactions it's paying. |
| `_l10n_fr_pdp_is_sale` | internal rule | self | `l10n_fr_pdp_pos`, `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_is_purchase` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_get_l10n_fr_pdp_errors` | preparation rule | self, lazy | `l10n_fr_pdp` |  | Return the list of validation errors for this move in the context of PDP reporting. |
| `_l10n_fr_pdp_get_referenced_documents` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_transaction_type` | internal rule | self | `l10n_fr_pdp` |  | Classify invoice for PDP reporting: b2c, b2bi, or False (domestic B2B). |
| `_l10n_fr_pdp_reports_pos_is_transaction_entry` | internal rule | self | `l10n_fr_pdp_pos` |  |  |
| `_compute_from_l10n_gr_edi_document_ids` | computation | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` | depends: `l10n_gr_edi_document_ids`; depends: `l10n_gr_edi_document_ids`, `l10n_gr_edi_document_ids.attachment_id`, `l10n_gr_edi_document_ids.mydata_cls_mark`, `l10n_gr_edi_document_ids.mydata_mark`, `l10n_gr_edi_document_ids.state` |  |
| `_compute_l10n_gr_edi_alerts` | computation | self | `l10n_gr_edi` | depends: `country_code`, `state` |  |
| `_compute_l10n_gr_edi_enable_fields` | computation | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` | depends: `state`, `l10n_gr_edi_state` |  |
| `_compute_l10n_gr_edi_payment_method` | computation | self | `l10n_gr_edi` | depends: `country_code` |  |
| `_compute_l10n_gr_edi_available_inv_type` | computation | self | `l10n_gr_edi` | depends: `move_type` |  |
| `_compute_l10n_gr_edi_inv_type` | computation | self | `l10n_gr_edi` | depends: `fiscal_position_id`, `l10n_gr_edi_available_inv_type` |  |
| `_compute_l10n_gr_edi_need_fields` | computation | self | `l10n_gr_edi` | depends: `l10n_gr_edi_inv_type` |  |
| `_l10n_gr_edi_create_error_document` | internal rule | self, values | `l10n_gr_edi` |  | Creates `l10n_gr_edi.document` of state `invoice_error` or `bill_error`. :param values: dictionary in the format of: {'error': <str>, 'xml_content': <optional/str>} |
| `_l10n_gr_edi_create_sent_document` | internal rule | self, values | `l10n_gr_edi` |  | Creates `l10n_gr_edi.document` of state `invoice_sent` or `bill_sent`. :param values: dictionary in the format of: {     'mydata_mark': <str>,     'mydata_cls_mark': <optional/str>,     'mydata_url': <str>,     'xml_content': <str>, } |
| `_l10n_gr_edi_generate_xml_content` | internal rule | self, xml_template, xml_vals | `l10n_gr_edi` | model |  |
| `_l10n_gr_edi_eligible_for_mydata` | internal rule | self | `l10n_gr_edi` |  | Shorthand for getting the eligibility of the current move to send to myDATA. |
| `_l10n_gr_edi_get_extra_invoice_report_values` | internal rule | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` |  | Get the values used to render the invoice PDF. |
| `_l10n_gr_edi_add_address_vals` | internal rule | self, values | `l10n_gr_edi` |  | Adds all the address values needed for the `invoice_vals` dictionary. The only guaranteed keys in to add in the dictionary is the issuer's VAT, country code, and branch number. Everything else is only displayed on some specific case/configuration. The appended dictionary will have the following additional keys: {     'issuer_vat_number': <str>,     'issuer_country': <str>,     'issuer_branch': <int>,     'issuer_name': <str \| None>,     'issuer_postal_code': <str \| None>,     'issuer_city': <str \| None>,     'counterpart_vat': <str \| None>,     'counterpart_country': <str \| None>,     ' |
| `_l10n_gr_edi_add_payment_method_vals` | internal rule | self, values | `l10n_gr_edi` |  | Adds payment values needed for the `invoice_vals` dictionary. The appended dictionary will have the following additional key: { 'payment_details': [ { 'type': <str>, 'amount': <float> }, ... ] } :param dict values: :rtype: dict[str, list[dict]] |
| `_l10n_gr_edi_common_base_line_details_values` | internal rule | self, base_line | `l10n_gr_edi` | model | Returns additional income/expense classification items ("icls"/"ecls") if needed for the detail values. The returned format is: {'ecls': [ {'category': <str>, 'type': <str>, 'amount': <float>}, ... ], 'icls': <same_as_ecls> } :param dict base_line: dictionary obtained from the tax computation helper methods; such as `_get_rounded_base_and_tax_lines`. :rtype: dict[str, list[dict]] |
| `_l10n_gr_edi_add_sum_classification_vals` | internal rule | self, values | `l10n_gr_edi` | model | Aggregates all amounts from the common categories and types of the list vals from the `details` key, and then add them to the `values` dictionary parameter. [!WARNING!] The `values` parameter **must** have the `details` key. let `XCLSList` be a list with format of: [     {'category': <str>, 'type': <str>, 'amount': <float>},     ..., ] All in all, a subset of the `values` parameter should follow the following type formats: {     'details': {         'icls': XCLSList,         'ecls': XCLSList,     } } The `values` dictionary will then be appended with the following keys: {     'summary_icls |
| `_l10n_gr_edi_get_invoices_xml_vals` | internal rule | self | `l10n_gr_edi` |  | Generates a dictionary containing the values needed for rendering `l10n_gr_edi.mydata_invoice` XML. :return: dict |
| `_l10n_gr_edi_get_expense_classification_xml_vals` | internal rule | self | `l10n_gr_edi` |  | Generates a dictionary containing the values needed for rendering `l10n_gr_edi.mydata_expense_classification` XML. :return: dict |
| `_l10n_gr_edi_get_pre_error_dict` | internal rule | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` |  | Try to catch all possible errors before sending to myDATA. Returns an error dictionary in the format of Actionable Error JSON. |
| `_l10n_gr_edi_get_pre_error_string` | internal rule | self | `l10n_gr_edi` |  |  |
| `_l10n_gr_edi_handle_send_result` | internal rule | self, result, xml_vals | `l10n_gr_edi` | model | Handle the result object received from sending xml to myDATA. Create the related error/sent document with the necessary values. |
| `_l10n_gr_edi_send_invoices` | internal rule | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` |  | Send batches of invoice SendInvoice XML to myDATA. |
| `_l10n_gr_edi_send_expense_classification` | internal rule | self | `l10n_gr_edi` |  | Send batches of bill SendExpensesClassification XML to myDATA. |
| `l10n_gr_edi_try_send_invoices` | operation | self | `l10n_gr_edi_e_invoo`, `l10n_gr_edi` |  |  |
| `l10n_gr_edi_try_send_expense_classification` | operation | self | `l10n_gr_edi` |  |  |
| `_l10n_gr_edi_try_send_batch` | internal rule | self | `l10n_gr_edi` |  | Only available for Vendor Bills. In case of invoices, user should use Send & Print instead. |
| `_l10n_gr_edi_get_provider_invoice_id` | internal rule | self | `l10n_gr_edi_e_invoo` |  | Return an invoice ID that is unique across the system databases. |
| `_l10n_gr_edi_prepare_invoice_proxy_request` | internal rule | self, invoice_datetime | `l10n_gr_edi_e_invoo` |  |  |
| `_l10n_gr_edi_prepare_invoice_submission` | internal rule | self | `l10n_gr_edi_e_invoo` |  |  |
| `_l10n_gr_edi_handle_invoice_proxy_result` | internal rule | self, document, result | `l10n_gr_edi_e_invoo` |  |  |
| `_compute_l10n_hr_payment_unreported` | computation | self | `l10n_hr_edi` | depends: `l10n_hr_edi_addendum_id.payment_reported_amount`, `amount_residual`, `amount_total` |  |
| `_search_l10n_hr_payment_unreported` | search rule | self, operator, value | `l10n_hr_edi` |  |  |
| `_check_l10n_hr_process_type` | validation | self | `l10n_hr_edi` | constrains: `move_type`, `l10n_hr_process_type` |  |
| `_compute_l10n_hr_process_type` | computation | self | `l10n_hr_edi` | depends: `move_type`, `l10n_hr_process_type` |  |
| `_get_l10n_hr_fiscalization_number` | preparation rule | self, name | `l10n_hr_edi` |  | Extract the fiscal numbering triple (ex. 1/1/1) from the document name. Only applies for Croatian sales invoices/credit notes. Expected name pattern is produced by the overridden `sequence.mixin` logic. |
| `_get_l10n_hr_fiscal_user_id_domain` | preparation rule | self | `l10n_hr_edi` |  |  |
| `UNUSED_get_ubl_cii_builder_from_xml_tree` | operation | self, tree | `l10n_hr_edi` | model |  |
| `l10n_hr_edi_mer_action_reject` | operation | self | `l10n_hr_edi` |  |  |
| `l10n_hr_edi_mer_action_fetch_status` | operation | self | `l10n_hr_edi` |  | Fetch and update the status of a single document on MojEracun. |
| `l10n_hr_edi_mer_action_report_paid` | operation | self | `l10n_hr_edi` |  |  |
| `_check_posted_if_active` | validation | self | `l10n_hu_edi` | constrains: `l10n_hu_edi_state`, `state` | Enforce the constraint that you cannot reset to draft / cancel a posted invoice if it was already sent to NAV. |
| `_compute_message_html` | computation | self | `l10n_hu_edi` | depends: `l10n_hu_edi_messages` |  |
| `_compute_l10n_hu_edi_attachment_filename` | computation | self | `l10n_hu_edi` | depends: `name`, `ref` |  |
| `l10n_hu_edi_button_update_status` | operation | self, from_cron | `l10n_hu_edi` |  | Attempt to update the status of the invoices in `self` |
| `l10n_hu_edi_button_hide_banner` | operation | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_get_valid_actions` | internal rule | self | `l10n_hu_edi` |  | If any NAV 3.0 flows are applicable to the given invoice, return them, else None. |
| `_l10n_hu_get_chain_base` | internal rule | self | `l10n_hu_edi` |  | Get the base invoice of the invoice chain. |
| `_l10n_hu_get_chain_invoices` | internal rule | self | `l10n_hu_edi` |  | Given base invoices, get all invoices in the chain. |
| `_l10n_hu_get_currency_rate` | internal rule | self | `l10n_hu_edi` |  | Get the invoice currency / HUF rate.  We don't use `invoice_currency_rate` to avoid rounding error as 1/0.002470 ≃ 404.87, and we want exactly 404.87, i.e. the rate given by the MNB of Hungary, to avoid NAV error upon XML submission. |
| `_l10n_hu_edi_set_chain_index` | internal rule | self | `l10n_hu_edi` |  | Set the l10n_hu_invoice_chain_index field. |
| `_l10n_hu_edi_acquire_lock` | internal rule | self | `l10n_hu_edi` |  | Acquire a write lock on the invoices in self. |
| `_l10n_hu_edi_check_invoices` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_upload` | internal rule | self, connection | `l10n_hu_edi` |  | Generate invoice XMLs and send to NAV. |
| `_l10n_hu_edi_get_operation_type` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_upload_single_batch` | internal rule | self, connection | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_query_status` | internal rule | self, connection | `l10n_hu_edi` |  | Check the NAV invoice status. |
| `_l10n_hu_edi_query_status_single_batch` | internal rule | self, connection | `l10n_hu_edi` |  | Check the NAV status for invoices that share the same transaction code (uploaded in a single batch). |
| `_l10n_hu_edi_process_query_transaction_result` | internal rule | self, processing_result, annulment_status | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_request_cancel` | internal rule | self, connection, code, reason | `l10n_hu_edi` |  | Send a cancellation request for all invoices in `self`. |
| `_l10n_hu_edi_request_cancel_single_batch` | internal rule | self, connection, code, reason | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_generate_xml` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_get_electronic_invoice_template` | internal rule | self | `l10n_hu_edi` |  | For feature extensibility. |
| `_l10n_hu_edi_get_invoice_values` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_get_invoice_totals_for_report` | internal rule | self | `l10n_hu_edi` |  | In Hungary, tax amounts should appear negative on credit notes. We therefore apply a post-processing to the tax totals to make them negative. |
| `_l10n_hu_edi_parse_digest_response` | internal rule | self, response_xml, company | `l10n_hu_edi_receive` | model |  |
| `_l10n_hu_edi_parse_query_invoice_data_response` | internal rule | self, response_xml, company | `l10n_hu_edi_receive` | model |  |
| `_l10n_hu_edi_parse_invoice_data_xml` | internal rule | self, invoice_data_xml, common_move_vals, company | `l10n_hu_edi_receive` | model |  |
| `_l10n_hu_edi_parse_invoice_xml` | internal rule | self, invoice_xml, company | `l10n_hu_edi_receive` | model | Returns a tuple of (move_vals, post_process_data)  :return: tuple(move_vals, post_process_data)     * move_vals: dict of values to create an `account.move`.     * post_process_data: dict containing additional data used during     move post-processing, with the following shape:         - gross_total (float): Gross total parsed from the XML.         - missing_taxes_error (Markup \| None): HTML formatted message         listing missing taxes, if any. |
| `_l10n_hu_edi_post_process_data` | internal rule | self, moves, post_process_data_list | `l10n_hu_edi_receive` | model |  |
| `_l10n_id_cron_update_payment_status` | internal rule | self | `l10n_id` |  | This cron will:     - Get all invoices that are not paid, and have details about QRIS qr codes.     - For each invoices, get information about the payment state of the QR using the API.     - If the QR is not paid and it has been more than 30m, we discard that qr id (no longer valid)     - If it is paid, we will register the payment on the invoices. |
| `action_l10n_id_update_payment_status` | user action | self | `l10n_id` |  | This action will:     - Get all invoices that are not paid, and have details about QRIS qr codes.     - For each invoices, get information about the payment state of the QR using the API.     - If the QR is not paid and it has been more than 30m, we discard that qr id (no longer valid)     - If it is paid, we will register the payment on the invoices. |
| `_l10n_id_update_payment_status` | internal rule | self | `l10n_id` |  | Starts by fetching the QR statuses for the invoices in self, then update said invoices based on the statuses |
| `_l10n_id_get_qris_qr_statuses` | internal rule | self | `l10n_id` |  | Query the API in order to get updated information on the status of each QR codes linked to the invoices in self. If the QR has been paid, only the paid information is returned.  :return: a list with the format:     {         invoice: {             'paid': True,             'qr_statuses': [],         },         invoice: {             'paid': False,             'qr_statuses': [],         }     } |
| `_l10n_id_process_invoices` | internal rule | self, invoices_statuses | `l10n_id` |  | Receives the list of invoices and their statuses, and update them using it. For paid invoices we will register the payment and log a note, while for unpaid ones we will discard expired QR data and keep the non-expired ones for the next run. |
| `_compute_kode_transaksi` | computation | self | `l10n_id_efaktur_coretax` | depends: `partner_id` |  |
| `_compute_l10n_id_coretax_efaktur_available` | computation | self | `l10n_id_efaktur_coretax` | depends: `partner_id`, `line_ids.tax_ids` | Similar use case as l10n_id_need_kode_transaksi from l10n_id_efaktur  helps to check whether or not some fields need to be visible or not |
| `_compute_l10n_id_coretax_facility_info` | computation | self | `l10n_id_efaktur_coretax` | depends: `l10n_id_coretax_add_info_07`, `l10n_id_coretax_add_info_08` |  |
| `_compute_l10n_id_coretax_add_info` | computation | self | `l10n_id_efaktur_coretax` | depends: `l10n_id_coretax_facility_info_07`, `l10n_id_coretax_facility_info_08` |  |
| `_validate_tax_groups` | internal rule | self | `l10n_id_efaktur_coretax` |  |  |
| `download_efaktur` | operation | self | `l10n_id_efaktur_coretax` |  | OVERRIDE l10n_id_efaktur  Change the flow of efaktur downloading. Collects data needed for efaktur and generate the xml file. |
| `download_xml` | operation | self | `l10n_id_efaktur_coretax` |  |  |
| `_l10n_id_coretax_build_invoice_vals` | internal rule | self, vals | `l10n_id_efaktur_coretax` |  | Fill in vals with invoice-related information |
| `prepare_efaktur_vals` | operation | self | `l10n_id_efaktur_coretax` |  | Get information required from invoice and lines to generate E-Faktur that will be used to load in the XML template later on |
| `_compute_l10n_in_gst_treatment` | computation | self | `l10n_in` | depends: `partner_id` |  |
| `_compute_l10n_in_state_id` | computation | self | `l10n_in_pos`, `l10n_in` | depends: `partner_id`, `partner_shipping_id`, `company_id`; depends: `pos_session_ids`, `reversed_pos_order_id` |  |
| `_compute_l10n_in_warning` | computation | self | `l10n_in_edi`, `l10n_in` | depends: `invoice_line_ids.l10n_in_hsn_code`, `company_id.l10n_in_hsn_code_digit`, `invoice_line_ids.tax_ids`, `commercial_partner_id.l10n_in_pan_entity_id`, `invoice_line_ids.price_total` |  |
| `_compute_l10n_in_show_gstin_status` | computation | self | `l10n_in` | depends: `partner_id`, `state`, `payment_state`, `l10n_in_gst_treatment` |  |
| `_compute_l10n_in_partner_gstin_status_and_date` | computation | self | `l10n_in` | depends: `partner_id` |  |
| `_compute_l10n_in_withholding_line_ids` | computation | self | `l10n_in` | depends: `line_ids`, `l10n_in_is_withholding` |  |
| `_compute_l10n_in_total_withholding_amount` | computation | self | `l10n_in` |  |  |
| `_compute_l10n_in_display_higher_tcs_button` | computation | self | `l10n_in` | depends: `l10n_in_warning` |  |
| `action_l10n_in_withholding_entries` | user action | self | `l10n_in` |  |  |
| `action_l10n_in_apply_higher_tax` | user action | self | `l10n_in` |  |  |
| `_get_l10n_in_invalid_tax_lines` | preparation rule | self | `l10n_in` |  |  |
| `_get_sections_aggregate_sum_by_pan` | preparation rule | self, section_alert, commercial_partner_id | `l10n_in` |  |  |
| `_l10n_in_is_warning_applicable` | internal rule | self, section_id | `l10n_in` |  |  |
| `_get_l10n_in_tds_tcs_applicable_sections` | preparation rule | self | `l10n_in` |  |  |
| `_get_tcs_applicable_lines` | preparation rule | self, lines | `l10n_in` |  |  |
| `l10n_in_verify_partner_gstin_status` | operation | self | `l10n_in` |  |  |
| `_l10n_in_get_warehouse_address` | internal rule | self | `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_in` |  | Return address where goods are delivered/received for Invoice/Bill |
| `_l10n_in_get_hsn_summary_table` | internal rule | self | `l10n_in` |  |  |
| `_l10n_in_get_bill_from_irn` | internal rule | self, irn | `l10n_in` |  |  |
| `_l10n_in_prepare_tax_details` | internal rule | self | `l10n_in` | model |  |
| `_get_l10n_in_seller_buyer_party` | preparation rule | self | `l10n_in` |  |  |
| `_l10n_in_extract_digits` | internal rule | self, string | `l10n_in` | model |  |
| `_l10n_in_is_service_hsn` | internal rule | self, hsn_code | `l10n_in` | model |  |
| `_l10n_in_round_value` | internal rule | self, amount, precision_digits | `l10n_in` | model | This method is call for rounding. If anything is wrong with rounding then we quick fix in method |
| `_get_l10n_in_tax_details_by_line_code` | preparation rule | self, tax_details | `l10n_in` | model |  |
| `_l10n_in_edi_get_iap_buy_credits_message` | internal rule | self | `l10n_in` | model |  |
| `_sync_l10n_in_gstr_section` | internal rule | self, moves | `l10n_in` |  |  |
| `_get_l10n_in_invoice_label` | preparation rule | self | `l10n_in` |  |  |
| `_compute_l10n_in_edi_content` | computation | self | `l10n_in_edi` |  |  |
| `action_export_l10n_in_edi_content_json` | user action | self | `l10n_in_edi` |  |  |
| `action_l10n_in_edi_force_cancel` | user action | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_need_cancel_request` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_check_einvoice_eligible` | internal rule | self | `l10n_in_edi` |  |  |
| `_get_l10n_in_edi_response_json` | preparation rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_lock_invoice` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_optional_field_validation` | internal rule | self, partner | `l10n_in_edi` |  | Validates optional partner fields (e.g., email, phone, street2) for e-invoicing, which are not mandatory in the government API JSON schema. Returns error messages for posting in the chatter. |
| `_l10n_in_edi_send_invoice` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_cancel_invoice` | internal rule | self | `l10n_in_edi` |  |  |
| `_get_l10n_in_edi_partner_details` | preparation rule | self, partner, set_vat, set_phone_and_email, is_overseas, pos_state_id | `l10n_in_edi` | model | Create the dictionary based partner details if set_vat is true then, vat(GSTIN) and legal name(LglNm) is added if set_phone_and_email is true then phone and email is add if set_pos is true then state code from partner  or passed state_id is added as POS(place of supply) if is_overseas is true then pin is 999999 and GSTIN(vat) is URP and Stcd is . if pos_state_id is passed then we use set POS |
| `_get_l10n_in_edi_line_details` | preparation rule | self, index, line, line_tax_details | `l10n_in_edi` |  | Create the dictionary with line details |
| `_l10n_in_edi_generate_invoice_json_managing_negative_lines` | internal rule | self, json_payload | `l10n_in_edi` |  | Set negative lines against positive lines as discount with same HSN code and tax rate With negative lines product name \| hsn code \| unit price \| qty \| discount \| total ============================================================= product A    \| 123456   \| 1000       \| 1   \| 100      \|  900 product B    \| 123456   \| 1500       \| 2   \| 0        \| 3000 Discount     \| 123456   \| -300       \| 1   \| 0        \| -300 Converted to without negative lines product name \| hsn code \| unit price \| qty \| discount \| total ============================================================= pr |
| `_l10n_in_edi_generate_invoice_json` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_get_supply_type` | internal rule | self, is_igst_amount | `l10n_in_edi` |  |  |
| `_l10n_in_check_einvoice_validation` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_connect_to_server` | internal rule | self, url_end_point, json_payload, params | `l10n_in_edi` |  | url_end_point possible values (generate, getirnbydocdetails, generate_ewaybill_by_irn, get_ewaybill_by_irn, cancel) is used to get the EDI response from the server |
| `_get_l10n_in_ewaybill_form_action` | preparation rule | self | `l10n_in_ewaybill` |  |  |
| `action_l10n_in_ewaybill_create` | user action | self | `l10n_in_ewaybill` |  |  |
| `action_open_l10n_in_ewaybill` | user action | self | `l10n_in_ewaybill` |  |  |
| `_compute_l10n_in_ewaybill_details` | computation | self | `l10n_in_ewaybill` | depends: `l10n_in_ewaybill_ids.state` |  |
| `_stock_account_prepare_anglo_saxon_in_lines_vals` | internal rule | self | `purchase_stock` |  | Prepare values used to create the journal items (account.move.line) corresponding to the price difference lines for vendor bills. It only concerns the quantities that have been delivered before the bill Example: Buy a product having a cost of 9 and a supplier price of 10 and being a storable product and having a perpetual valuation in FIFO. Deliver the product and then post the bill. The vendor bill's journal entries looks like:  Account                                     \| Debit \| Credit --------------------------------------------------------------- 101120 Stock Account                    |
| `_compute_l10n_it_payment_method` | computation | self | `l10n_it_edi` | depends: `line_ids.matching_number`, `payment_state`, `matched_payment_ids` |  |
| `_compute_l10n_it_document_type` | computation | self | `l10n_it_edi` | depends: `state` |  |
| `_compute_l10n_it_partner_pa` | computation | self | `l10n_it_edi` | depends: `commercial_partner_id.l10n_it_pa_index`, `company_id` |  |
| `_compute_l10n_it_partner_is_public_administration` | computation | self | `l10n_it_edi` | depends: `commercial_partner_id.l10n_it_pa_index`, `company_id` |  |
| `_compute_l10n_it_edi_button_label` | computation | self | `l10n_it_edi` | depends: `country_code`, `l10n_it_edi_proxy_mode` |  |
| `_compute_l10n_it_edi_is_self_invoice` | computation | self | `l10n_it_edi` | depends: `move_type`, `line_ids.tax_tag_ids` | Italian EDI requires Vendor bills coming from EU countries to be sent as self-invoices. We recognize these cases based on the taxes that target the VJ tax grids, which imply the use of VAT External Reverse Charge. |
| `_l10n_it_edi_exempt_reason_tag_mapping` | internal rule | self | `l10n_it_edi` |  |  |
| `_parse_xml_with_recovery` | internal rule | self, content, name | `l10n_it_edi` |  |  |
| `_get_xml_tree` | preparation rule | self, file_data | `l10n_it_edi` |  | Some FatturaPA XMLs need to be parsed with `recover=True`, and some have signatures that need to be removed prior to parsing. |
| `_check_l10n_it_edi_xml_content` | validation | self, file_data | `l10n_it_edi` |  | Checks if the XML root tag starts with 'FatturaElettronica'. Handles both standard XML and signed p7m files. |
| `_is_l10n_it_edi_import_file` | internal rule | self, file_data | `l10n_it_edi` |  |  |
| `_inverse_l10n_it_edi_state` | inverse computation | self | `l10n_it_edi` |  |  |
| `action_l10n_it_edi_send` | user action | self | `l10n_it_edi` |  | Checks that the invoice data is coherent. Attaches the XML file to the invoice. Sends the invoice to the SdI. |
| `action_check_l10n_it_edi` | user action | self | `l10n_it_edi` |  |  |
| `action_invoice_download_fatturapa` | user action | self | `l10n_it_edi` |  |  |
| `_l10n_it_edi_ready_for_xml_export` | internal rule | self | `l10n_it_edi` |  |  |
| `_l10n_it_edi_add_base_lines_xml_values` | internal rule | self, base_lines_aggregated_values, is_downpayment | `l10n_it_edi` |  |  |
| `_l10n_it_edi_get_tax_lines_xml_values` | internal rule | self, base_lines_aggregated_values, values_per_grouping_key | `l10n_it_edi` |  |  |
| `_l10n_it_edi_is_neg_split_payment` | internal rule | self, tax_data | `l10n_it_edi` | model |  |
| `_l10n_it_edi_grouping_function_base_lines` | internal rule | self, base_line, tax_data | `l10n_it_edi` | model |  |
| `_l10n_it_edi_grouping_function_tax_lines` | internal rule | self, base_line, tax_data | `l10n_it_edi` | model |  |
| `_l10n_it_edi_grouping_function_total` | internal rule | self, base_line, tax_data | `l10n_it_edi` | model |  |
| `_l10n_it_edi_get_oss_line_values` | internal rule | self, aml, base_line, vat_tax, n7_tax, n22_tax | `l10n_it_edi` |  |  |
| `_l10n_it_edi_get_values` | internal rule | self, pdf_values | `l10n_it_edi`, `l10n_it_stock_ddt` |  |  |
| `_l10n_it_edi_services_or_goods` | internal rule | self | `l10n_it_edi` |  | Services and goods have different tax grids when VAT is Reverse Charged, and they can't be mixed in the same invoice, because the TipoDocumento depends on which which kind of product is bought and it's unambiguous. |
| `_l10n_it_edi_goods_in_italy` | internal rule | self | `l10n_it_edi` |  | There is a specific TipoDocumento (Document Type TD19) and tax grid (VJ3) for goods that are phisically in Italy but are in a VAT deposit, meaning that the goods have not passed customs. |
| `_l10n_it_edi_is_simplified` | internal rule | self | `l10n_it_edi` |  | Simplified Invoices are a way for the invoice issuer to create an invoice with limited data. Example: a consultant goes to the restaurant and wants the invoice instead of the receipt, to be able to deduct the expense from his Taxes. The Italian State allows the restaurant to issue a Simplified Invoice with the VAT number only, to speed up times, instead of requiring the address and other information about the buyer. The maximum threshold is 400 Euro, except for the forfettario tax regime (RF19), which can issue simplified invoices without the amount limit.  Deprecated since 18.0: use `not _l10 |
| `_l10n_it_edi_is_simplified_checks` | internal rule | self | `l10n_it_edi` |  | Warnings can be ignored by setting `l10n_it_document_type == 'TD07'` in the optional `l10n_it_edi_ndd` module |
| `_l10n_it_edi_is_professional_fees` | internal rule | self | `l10n_it_edi` |  | This function returns a boolean value based on the comparison of the lines values with a product. If one line has the tag for professional fee then we return True |
| `_l10n_it_edi_features_for_document_type_selection` | internal rule | self | `l10n_it_edi`, `l10n_it_stock_ddt` |  | Returns a dictionary of features to be compared with the TDxx FatturaPA document type requirements. |
| `_l10n_it_edi_document_type_mapping` | internal rule | self | `l10n_it_edi`, `l10n_it_stock_ddt` |  | Returns a dictionary with the required features for every TDxx FatturaPA document type |
| `_l10n_it_edi_get_document_type` | internal rule | self | `l10n_it_edi` |  | If the user has selected a document type, generally use that. Retrieve document type from the move. If not set, compare the features of the invoice to the requirements of each Document Type (TDxx) FatturaPA until you find a valid one. If the user has selected (the default) TD01 and the partner has no complete address then we can't issue a TD01 invoice - but if it's possible to issue a simplified invoice, then switch automatically to the TD07. |
| `_l10n_it_edi_is_simplified_document_type` | internal rule | self, document_type | `l10n_it_edi` |  |  |
| `_l10n_it_buyer_seller_info` | internal rule | self | `l10n_it_edi` | model |  |
| `cron_l10n_it_edi_download_and_update` | operation | self | `l10n_it_edi` |  | Crons run with sudo(), with empty recordset. Remember that. |
| `_l10n_it_edi_download_invoices` | internal rule | self, proxy_user | `l10n_it_edi` |  | Check the proxy for incoming invoices for a specified proxy user. :return: True if there remain some invoices on the server to be downloaded, False otherwise. |
| `_l10n_it_edi_process_downloads` | internal rule | self, invoices_data, proxy_user | `l10n_it_edi` |  | Every attachment will be committed if stored succesfully. Also moves will be committed one by one, even if imported incorrectly. |
| `_l10n_it_edi_check_and_decrypt_content` | internal rule | self, filename, content, key, proxy_user | `l10n_it_edi` |  | Check whether an incoming file from the SdI should be created as a new attachment, and try to decrypt it.  :param filename:       name of the file to be saved. :param content:        encrypted content of the file to be saved. :param key:            key to decrypt the file. :param proxy_user:     the AccountEdiProxyClientUser to use for decrypting the file |
| `_l10n_it_edi_process_downloads_attachments` | internal rule | self, company_id, attachment_vals | `l10n_it_edi` |  |  |
| `_l10n_it_edi_search_partner` | internal rule | self, company, vat, codice_fiscale, email, destination_code | `l10n_it_edi` |  |  |
| `_l10n_it_edi_search_tax_for_import` | internal rule | self, company, percentage, extra_domain, l10n_it_exempt_reason | `l10n_it_edi` |  | Returns the VAT, Withholding or Pension Fund tax that suits the conditions given and matches the percentage found in the XML for the company. |
| `_l10n_it_edi_get_extra_info` | internal rule | self, company, document_type, body_tree, incoming | `l10n_it_edi` |  | This function is meant to collect other information that has to be inserted on the invoice lines by submodules. :return: extra_info, messages_to_log |
| `_l10n_it_edi_get_payment_info` | internal rule | self, tree | `l10n_it_edi` |  | Map XML payment data from node //DatiPagamento/DettaglioPagamento to dict |
| `_l10n_it_edi_get_partner_info` | internal rule | self, node | `l10n_it_edi` |  | Map the XML partner data to dict from one of the nodes: - CedentePrestatore (seller) - CessionarioCommittente (buyer)  The Codice Fiscale can have two forms:     - personal: GTTPLA84T04F205G     - equal to the company VAT: 01234567890  A company may be part of a VAT group:     - the VAT is shared and the Codice Fiscale is the old VAT number before grouping     - Both VAT and Codice Fiscale must be specified  So in case:     - Partner is domestic (country == IT)     - VAT is not specified     - the Codice Fiscale looks like 01234567890 we also set it as VAT number |
| `_l10n_it_edi_import_invoice` | internal rule | self, invoice, data, is_new | `l10n_it_edi` |  | Decode a FatturaPA attachment into the system move.  :param data:   the dictionary with the content to be imported                keys: 'name', 'raw', 'xml_tree', 'import_file_type' :param is_new: whether the move is newly created or to be updated :returns:      the imported move |
| `_is_prediction_enabled` | internal rule | self | `l10n_it_edi` | model |  |
| `_get_prediction_cache_value` | preparation rule | self, key, predict_function | `l10n_it_edi` |  |  |
| `_l10n_it_edi_import_line` | internal rule | self, element, move_line, extra_info | `l10n_it_edi` |  |  |
| `_l10n_it_edi_format_errors` | internal rule | self, header, errors | `l10n_it_edi` |  |  |
| `_compose_info_message` | internal rule | self, tree, tags | `l10n_it_edi` |  |  |
| `_l10n_it_edi_export_data_check` | internal rule | self | `l10n_it_edi` |  | This function checks the Settings, Company, Partners, Moves involved in the sending activity and returns an errors dictionary ready for the actionable_errors widget to display. |
| `_l10n_it_edi_build_move_error` | internal rule | self, message, records, level | `l10n_it_edi` |  |  |
| `_l10n_it_edi_base_export_check` | internal rule | self | `l10n_it_edi` |  |  |
| `_l10n_it_edi_export_taxes_check` | internal rule | self | `l10n_it_edi` |  |  |
| `_l10n_it_edi_get_max_limit_per_tax` | internal rule | self, kind_code | `l10n_it_edi` |  |  |
| `_l10n_it_edi_check_lines_for_tax_kind` | internal rule | self, kind_code, kind_desc, min_len | `l10n_it_edi` |  |  |
| `_l10n_it_edi_get_formatters` | internal rule | self | `l10n_it_edi` |  |  |
| `_l10n_it_edi_render_xml` | internal rule | self, pdf_values | `l10n_it_edi` |  | Create the xml file content. :return:    The XML content as bytestring. |
| `_l10n_it_edi_get_attachment_values` | internal rule | self, pdf_values | `l10n_it_edi` |  |  |
| `_l10n_it_edi_generate_filename` | internal rule | self | `l10n_it_edi` |  | Returns a name conform to the Fattura pa Specifications: See ES documentation 2.2 |
| `_l10n_it_edi_send` | internal rule | self, attachments_vals | `l10n_it_edi` |  |  |
| `_l10n_it_edi_upload_error_message` | internal rule | self, error_code, error_description | `l10n_it_edi` |  | Translate server errors with the client user's language. |
| `_l10n_it_edi_upload_single` | internal rule | self, file | `l10n_it_edi` |  | Upload file to the SdI. :param file:    A dictionary {filename, base64_xml}. :returns:        A dictionary. * message:       Message from fatturapa. * transactionId: The fatturapa ID of this request. * error:         An eventual error. |
| `_l10n_it_edi_upload` | internal rule | self, files | `l10n_it_edi` |  | Upload files to the SdI. :param files:    A list of dictionary {filename, base64_xml}. :returns:        A dict mapping each input filename to the result returned by _l10n_it_edi_upload_single |
| `_l10n_it_edi_update_send_state` | internal rule | self | `l10n_it_edi` |  | Check if the current invoices have been processed by the SdI. |
| `_l10n_it_edi_parse_notification` | internal rule | self, notification | `l10n_it_edi` |  |  |
| `_l10n_it_edi_transform_notification` | internal rule | self, parsed_notification | `l10n_it_edi` |  | Reads the notification XML coming from the EDI Proxy Server Recovers information about the new state. Computes whether the EDI Proxy Server is to be acked, and whether the id_transaction has to be reset. |
| `_l10n_it_edi_write_send_state` | internal rule | self, transformed_notification, message | `l10n_it_edi` |  | Update the record with the data coming from the IAP server. Eventually post the message. Commit the transaction. |
| `_l10n_it_edi_get_message` | internal rule | self, transformed_notification | `l10n_it_edi` |  | The status change will be notified in the chatter of the move. Compute the message from the notification information coming from the EDI Proxy Server |
| `_get_pension_fund_tax_for_line` | preparation rule | self, element, extra_info | `l10n_it_edi` | model | Apply the pension fund on all lines that have the related AliquotaIVA If there are AssoSoftware specific AltriDatiGestionale 'AswCassPre' tags that specify which lines have pension funds, only apply to them. |
| `_get_l10_it_edi_get_taxable_amount_from_summary_data` | preparation rule | self, element | `l10n_it_edi` | model |  |
| `_compute_l10n_it_edi_doi_date` | computation | self | `l10n_it_edi_doi` | depends: `invoice_date` |  |
| `_compute_l10n_it_edi_doi_use` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `country_code`, `move_type` |  |
| `_compute_l10n_it_edi_doi_id` | computation | self | `l10n_it_edi_doi` | depends: `company_id`, `partner_id.commercial_partner_id`, `l10n_it_edi_doi_date`, `currency_id` |  |
| `_compute_l10n_it_edi_doi_amount` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `tax_totals`, `move_type` | Consider all the lines in self that belong to declaration of intent `declaration` and have the special declaration of intent tax applied. This function computes the signed sum of the price_total of all those lines (the tax amount of the lines is always 0). The direction_sign determines the sign: 1 (-1) for inbound (outbound) types. |
| `_compute_l10n_it_edi_doi_warning` | computation | self | `l10n_it_edi_doi` | depends: `l10n_it_edi_doi_id`, `l10n_it_edi_doi_amount`, `state` |  |
| `_check_l10n_it_edi_doi_id` | validation | self | `l10n_it_edi_doi` | constrains: `l10n_it_edi_doi_id` |  |
| `action_open_declaration_of_intent` | user action | self | `l10n_it_edi_doi` |  |  |
| `_l10n_it_edi_invoice_is_direct` | internal rule | self | `l10n_it_stock_ddt` |  | An invoice is only direct if the Transport Documents are all done the same day as the invoice. |
| `_get_ddt_values` | preparation rule | self | `l10n_it_stock_ddt` |  | We calculate the link between the invoice lines and the deliveries related to the invoice through the links with the sale order(s).  We assume that the first picking was invoiced first. (FIFO) :return: a dictionary with as key the picking and value the invoice line numbers (by counting) |
| `_compute_ddt_ids` | computation | self | `l10n_it_stock_ddt` | depends: `invoice_line_ids`, `invoice_line_ids.sale_line_ids` |  |
| `get_linked_ddts` | operation | self | `l10n_it_stock_ddt` |  |  |
| `_compute_l10n_jo_edi_is_needed` | computation | self | `l10n_jo_edi_pos`, `l10n_jo_edi` | depends: `country_code`, `move_type` |  |
| `_compute_l10n_jo_edi_uuid` | computation | self | `l10n_jo_edi` | depends: `l10n_jo_edi_is_needed` |  |
| `_compute_l10n_jo_edi_computed_xml` | computation | self | `l10n_jo_edi` | depends: `state`, `l10n_jo_edi_is_needed` |  |
| `_compute_l10n_jo_edi_invoice_type` | computation | self | `l10n_jo_edi` | depends: `partner_id.country_code` |  |
| `download_l10n_jo_edi_computed_xml` | operation | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_qr_code_src` | internal rule | self | `l10n_jo_edi` |  |  |
| `_is_sales_refund` | internal rule | self | `l10n_jo_edi` |  |  |
| `_get_invoice_scope_code` | preparation rule | self | `l10n_jo_edi` |  |  |
| `_get_invoice_payment_method_code` | preparation rule | self | `l10n_jo_edi` |  |  |
| `_get_invoice_tax_payer_type_code` | preparation rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_build_jofotara_headers` | internal rule | self | `l10n_jo_edi` |  |  |
| `_send_l10n_jo_edi_request` | internal rule | self, params, headers | `l10n_jo_edi` |  |  |
| `_submit_to_jofotara` | internal rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_edi_get_xml_attachment_name` | internal rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_validate_config` | internal rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_validate_fields` | internal rule | self | `l10n_jo_edi` |  |  |
| `_mark_sent_jo_edi` | internal rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_edi_send` | internal rule | self | `l10n_jo_edi` |  |  |
| `_compute_l10n_ke_cu_show_send_button` | computation | self | `l10n_ke_edi_tremol` | depends: `country_code`, `l10n_ke_cu_qrcode`, `state`, `move_type`, `company_id` |  |
| `_l10n_ke_fmt` | internal rule | self, string, length, ljust | `l10n_ke_edi_tremol` |  | Function for common formatting behaviour  :param string: string to be formatted/encoded :param length: integer length to justify (if enabled), and then truncate the string to :param ljust:  boolean representing whether the string should be justified :returns:      byte-string justified/truncated, with all non-alphanumeric characters removed |
| `_l10n_ke_validate_move` | internal rule | self | `l10n_ke_edi_tremol` |  | Returns list of errors related to misconfigurations per move  Find misconfigurations on the move, the lines of the move, and the taxes on those lines that would result in rejection by the KRA. |
| `_l10n_ke_fiscal_device_details_filled` | internal rule | self | `l10n_ke_edi_tremol` |  |  |
| `_l10n_ke_cu_open_invoice_message` | internal rule | self | `l10n_ke_edi_tremol` |  | Serialise the required fields for opening an invoice  :returns: a list containing one byte-string representing the <CMD> and           <DATA> of the message sent to the fiscal device. |
| `_l10n_ke_cu_lines_messages` | internal rule | self | `l10n_ke_edi_tremol` |  | Serialise the data of each line on the invoice  This function transforms the lines in order to handle the differences between the KRA expected data and the lines in system.  If a discount line (as a negative line) has been added to the invoice lines, find a suitable line/lines to distribute the discount accross  :returns: List of byte-strings representing each command <CMD> and the           <DATA> of the line, which will be sent to the fiscal device           in order to add a line to the opened invoice. |
| `_l10n_ke_get_cu_messages` | internal rule | self | `l10n_ke_edi_tremol` |  | Composes a list of all the command and data parts of the messages required for the fiscal device to open an invoice, add lines and subsequently close it. |
| `l10n_ke_action_cu_post` | operation | self | `l10n_ke_edi_tremol` |  | Returns the client action descriptor dictionary for sending the invoice(s) to the control unit (the fiscal device). |
| `l10n_ke_cu_responses` | operation | self, responses | `l10n_ke_edi_tremol` |  | Set the fields related to the fiscal device on the invoice.  This is intended to be utilized by an RPC call from the javascript client action. The fields are prefixed with l10n_ke_cu_*, which refers to the fact that they originate from the control unit. |
| `_lk_sql_seq_regex` | internal rule | self | `l10n_lk_invoice` |  | PSQL-safe pattern for LK names (no named groups, no lazy quantifiers).  The Python `LK_TAX_INVOICE_REGEX` uses named groups (`(?P<name>...)`), which PostgreSQL's `~` operator does not support. `_make_regex_non_capturing` converts them to non-capturing groups, but it is not sufficient on its own: it leaves the lazy quantifier of the suffix group (`\D*?`) untouched. Since that group only matches non-digit characters at the end of the name, its greedy equivalent (`\D*`) matches exactly the same names, so it is used instead. |
| `_constrains_l10n_lk_sequence_length` | validation | self | `l10n_lk_invoice` | constrains: |  |
| `_l10n_lk_is_tax_invoice_company` | internal rule | self | `l10n_lk_invoice` |  | Whether this invoice qualifies as a tax invoice under LK VAT law.  Requires both parties to be VAT-registered (the customer's status is that of its commercial partner) and all lines to be 18%/zero-rated (gazette s.4.2).  Excludes debit notes.  Controls PDF-level display. |
| `_l10n_lk_has_taxable_taxes` | internal rule | self | `l10n_lk_invoice` |  | All product lines must carry 18% or zero-rated taxes only (gazette s.4.2).  WHT/AIT withholding taxes are ignored for this determination: they are deducted at payment time and do not qualify (nor disqualify) a line as a taxable supply. |
| `_l10n_lk_use_tax_invoice_sequence` | internal rule | self | `l10n_lk_invoice` |  | Use YYMMM_QQQQ_XXXXX format for all LK sale documents from VAT-registered companies.  Unlike _l10n_lk_is_tax_invoice_company, does not check partner VAT or line taxability. |
| `_get_last_sequence` | preparation rule | self, relaxed, with_prefix | `l10n_lk_invoice` |  | Override to fetch the last LK sequence using a custom regex pattern.  The standard method uses sequence_prefix for filtering, but LK sequences use YYMMM_JOURNAL_SEQ format where the journal code is part of the name, not the prefix. We use a PSQL regex via _lk_sql_seq_regex to match LK-specific pattern and fetch the correct last sequence. |
| `_sequence_matches_date` | internal rule | self | `l10n_lk_invoice` |  | LK sequences never reset, so the standard date check (which depends on the reset frequency) does not apply. |
| `_get_sequence_format_param` | preparation rule | self, previous | `l10n_lk_invoice` |  | Parse an LK name into format params, extracting year/month/journal_code/seq/suffix. |
| `_get_next_sequence_format` | preparation rule | self | `l10n_lk_invoice` |  | Update month/year from the invoice date even though the sequence never resets, so the date portion stays accurate. |
| `_is_last_from_seq_chain` | internal rule | self | `l10n_lk_invoice` |  | LK sequences span months, so the standard prefix comparison cannot detect whether a newer entry exists in a different month. |
| `_is_end_of_seq_chain` | internal rule | self | `l10n_lk_invoice` |  | Normalize LK batch keys to journal_code only, so invoices from different months but the same journal are grouped together. |
| `_compute_l10n_my_edi_state` | computation | self | `l10n_my_edi` | depends: `l10n_my_edi_document_ids.myinvois_state` |  |
| `_compute_l10n_my_edi_display_tax_exemption_reason` | computation | self | `l10n_my_edi` | depends: `company_id`, `invoice_line_ids.tax_ids` | Some users will never use tax-exempt taxes, so it's better to only show the field when necessary. |
| `_compute_l10n_my_invoice_need_edi` | computation | self | `l10n_my_edi` | depends: `move_type`, `state`, `country_code`, `l10n_my_edi_state`, `company_id` |  |
| `action_l10n_my_edi_update_status` | user action | self | `l10n_my_edi` |  |  |
| `action_l10n_my_edi_send_invoice` | user action | self | `l10n_my_edi` |  | This action will create the MyInvois Document for this invoice if it does not already exist. Once done, it will trigger the sending of said document to the platform. |
| `action_show_myinvois_documents` | user action | self | `l10n_my_edi` |  |  |
| `_create_myinvois_document` | internal rule | self | `l10n_my_edi` |  | Helper which creates and link one MyInvois Document per invoice in self. |
| `_l10n_my_edi_get_proxy_user` | internal rule | self | `l10n_my_edi` |  | Helper to retrieve the proxy user related to the company of the record. |
| `_l10n_my_edi_cancel_moves` | internal rule | self | `l10n_my_edi` |  | Try to cancel the moves in self if allowed by the lock date. |
| `_generate_myinvois_qr_code` | internal rule | self | `l10n_my_edi` |  |  |
| `_get_active_myinvois_document` | preparation rule | self, including_in_progress | `l10n_my_edi` |  | Shortcut to get the active document of all invoice in self. |
| `_get_invoice_reference_no_invoice` | preparation rule | self | `l10n_no` |  | This computes the reference based on the system format. We calculat reference using invoice number and partner id and added control digit at last. |
| `_get_invoice_reference_no_partner` | preparation rule | self | `l10n_no` |  | This computes the reference based on the system format. We calculat reference using invoice number and partner id and added control digit at last. |
| `_get_kid_number` | preparation rule | self | `l10n_no` |  |  |
| `action_open_l10n_ph_2307_wizard` | user action | self | `l10n_ph` |  |  |
| `_l10n_pl_edi_check_mandatory_fields` | internal rule | self | `l10n_pl_edi` |  |  |
| `_l10n_pl_edi_get_ksef_invoice_type` | internal rule | self | `l10n_pl_edi` |  | Determines the specific TRodzajFaktury for KSeF. |
| `_l10n_pl_edi_get_related_invoices` | internal rule | self | `l10n_pl_edi` |  | Returns a list of related invoice numbers for ZAL and ROZ types. Safely checks for Sale Order links. |
| `_l10n_pl_edi_get_xml_values` | internal rule | self | `l10n_pl_edi_jst`, `l10n_pl_edi` |  | Prepares a dictionary of values to be passed to the QWeb template. |
| `_l10n_pl_edi_render_xml` | internal rule | self | `l10n_pl_edi` |  | Renders the QWeb template, removes empty lines. |
| `_l10n_pl_edi_generate_qr_link` | internal rule | self | `l10n_pl_edi` |  |  |
| `_l10n_pl_edi_generate_qr` | internal rule | self | `l10n_pl_edi` |  |  |
| `_l10n_pl_edi_get_status_mapping` | internal rule | self | `l10n_pl_edi` |  | Returns a dictionary mapping KSeF status codes to (system_status, message). |
| `action_l10n_pl_edi_update_invoice_status` | user action | self | `l10n_pl_edi` |  |  |
| `action_l10n_pl_edi_get_invoice_UPO` | user action | self | `l10n_pl_edi` |  |  |
| `_cron_l10n_pl_edi_check_invoice_status` | background operation | self | `l10n_pl_edi` |  | get all moves that are in state sent run action_update_invoice_status on all of them |
| `l10n_pl_edi_get_ksef_bill_vals_from_xml` | operation | self, xml_content | `l10n_pl_edi` | model |  |
| `_cron_l10n_pl_edi_download_bills` | background operation | self | `l10n_pl_edi` | model |  |
| `_l10n_pl_edi_download_bills_from_ksef` | internal rule | self | `l10n_pl_edi` | model |  |
| `_handle_download_bills_from_ksef_error` | internal rule | self, error | `l10n_pl_edi` |  |  |
| `_fetch_bills_metadata` | internal rule | self, service | `l10n_pl_edi` |  |  |
| `_fetch_bills_data` | internal rule | self, service, bills_to_fetch | `l10n_pl_edi` |  |  |
| `_decode_fa3_ksef` | internal rule | self, invoice, file_data, new | `l10n_pl_edi` |  |  |
| `_compute_l10n_ro_edi_state` | computation | self | `l10n_ro_edi` | depends: `l10n_ro_edi_document_ids` |  |
| `_l10n_ro_edi_get_pre_send_errors` | internal rule | self, xml_data | `l10n_ro_edi` |  | Compute all possible common errors before sending the XML to the SPV |
| `_l10n_ro_edi_send_invoice` | internal rule | self, xml_data | `l10n_ro_edi` |  | This method send xml_data to the Romanian SPV using the single invoice's (self) data. The invoice's company and move_type will be used to calculate the required params in the send request. The state of the document deletion/creation are as follows:   - Pre-check any errors from the invoice's pre_send check before sending      - if error -> create a new error document (previous documents are preserved for traceability)     - else -> continue to the next step   - Send to E-Factura, and based on the result:      - if error -> create a new error document (previous documents are preserved for trace |
| `_l10n_ro_edi_fetch_invoice_sent_documents` | internal rule | self | `l10n_ro_edi` |  | This method loops over all invoice with sending document in `self`. For each of them, it pre-checks errors and make a fetch request for the invoice. Then:   - if no answer is received, it will do nothing on the current invoice  - if there is an error during the communication with the server -> log it in the chatter  - else (receives `key_download`) -> immediately make a download request and process it:     - if there is an error during the communication with the server -> log it in the chatter     - if 'nok', then the invoice has been refused by ANAF -> create a refused document     - if 'ok', |
| `_l10n_ro_edi_fetch_invoices` | internal rule | self | `l10n_ro_edi` | model | Synchronize bills/invoices from SPV |
| `_l10n_ro_edi_process_invoice_accepted_messages` | internal rule | self, sent_invoices_accepted_messages | `l10n_ro_edi` | model | Process the validation messages of invoices sent  It will also attempt to recover the original invoices, that are missing their index, by matching the name returned by the server and the one in the database.  note: There is an edge case where 2 messages have the same invoice name but different indexes in their data; this could be due to a resequencing of the invoice and/or re-sending of an invoice. In that case coupled with name matching where none of the two invoices received an index, all signatures are added to the invoice; the user will have to manually update/select the correct one.  For  |
| `_l10n_ro_edi_process_invoice_refused_messages` | internal rule | self, sent_invoices_refused_messages | `l10n_ro_edi` | model | Process the refusal messages of invoices sent  For refused invoices, it is impossible to recover the original invoice from the message content like in `_l10n_ro_edi_process_invoice_accepted_messages` since the message only contains the index and error message (as relevant information). |
| `_l10n_ro_edi_process_bill_messages` | internal rule | self, received_bills_messages | `l10n_ro_edi` | model | Create bill received on the SPV, it it does not already exists. |
| `action_l10n_ro_edi_fetch_invoices` | user action | self | `l10n_ro_edi` |  |  |
| `_compute_l10n_rs_edi_is_eligible` | computation | self | `l10n_rs_edi` | depends: `country_code`, `move_type` |  |
| `_compute_l10n_rs_tax_date_obligations_code` | computation | self | `l10n_rs_edi` | depends: `country_code` |  |
| `_compute_l10n_rs_edi_uuid` | computation | self | `l10n_rs_edi` | depends: `l10n_rs_edi_is_eligible` |  |
| `_l10n_rs_edi_send` | internal rule | self, send_to_cir | `l10n_rs_edi` |  |  |
| `_l10n_rs_edi_get_attachment_values` | internal rule | self, xml | `l10n_rs_edi` |  |  |
| `_l10n_rs_edi_get_xml_attachment_name` | internal rule | self | `l10n_rs_edi` |  |  |
| `_compute_qr_code_str` | computation | self | `l10n_sa_edi_pos`, `l10n_sa_edi`, `l10n_sa` | depends: `amount_total_signed`, `amount_tax_signed`, `l10n_sa_confirmation_datetime`, `company_id`, `company_id.vat`; depends: `amount_total_signed`, `amount_tax_signed`, `l10n_sa_confirmation_datetime`, `company_id`, `company_id.vat`, `journal_id`, `journal_id.l10n_sa_production_csid_json`, `edi_document_ids`, `l10n_sa_invoice_signature`, `l10n_sa_chain_index`, `state` | Generate the qr code for Saudi e-invoicing. Specs are available at the following link at page 23 https://zatca.gov.sa/ar/E-Invoicing/SystemsDevelopers/Documents/20210528_ZATCA_Electronic_Invoice_Security_Features_Implementation_Standards_vShared.pdf |
| `get_l10n_sa_confirmation_datetime_sa_tz` | operation | self | `l10n_sa` |  |  |
| `_l10n_sa_reset_confirmation_datetime` | internal rule | self | `l10n_sa_edi`, `l10n_sa` |  | OVERRIDE: we want rejected phase 2 invoices to keep the original confirmation datetime |
| `_l10n_sa_get_adjustment_reason` | internal rule | self | `l10n_sa` |  |  |
| `_compute_show_l10n_sa_reason` | computation | self | `l10n_sa` |  |  |
| `_get_iso_format_asia_riyadh_date` | preparation rule | self, separator | `l10n_sa` |  |  |
| `_get_l10n_sa_totals` | preparation rule | self | `l10n_sa_edi`, `l10n_sa` |  |  |
| `_l10n_sa_is_legal` | internal rule | self | `l10n_sa_edi`, `l10n_sa` |  |  |
| `_get_normalized_l10n_sa_confirmation_datetime` | preparation rule | self, invoice_date, invoice_time | `l10n_sa` |  | Ensures the confirmation datetime does not exceed the current time in Asia/Riyadh to prevent ZATCA rejections. |
| `_l10n_sa_is_simplified` | internal rule | self | `l10n_sa_edi`, `l10n_sa` |  | Returns True if the customer is an individual, i.e: The invoice is B2C :return: |
| `_prevent_zatca_rejected_invoice_deletion` | internal rule | self | `l10n_sa_edi` | ondelete |  |
| `_l10n_sa_get_qr_code_encoding` | internal rule | self, tag, field, int_length | `l10n_sa_edi` |  | Helper function to encode strings for the QR code generation according to ZATCA specs |
| `_l10n_sa_check_billing_reference` | internal rule | self | `l10n_sa_edi` |  | Make sure credit/debit notes have a either a reveresed move or debited move or a customer reference |
| `_l10n_sa_get_qr_code` | internal rule | self, company_id, unsigned_xml, certificate, signature, is_b2c | `l10n_sa_edi` | model | Generate QR code string based on XML content of the Invoice UBL file, X509 Production Certificate and company info.  :return b64 encoded QR code string |
| `_l10n_sa_generate_unsigned_data` | internal rule | self | `l10n_sa_edi` |  | Generate UUID and digital signature to be used during both Signing and QR code generation. It is necessary to save the signature as it changes everytime it is generated and both the signing and the QR code expect to have the same, identical signature. |
| `_l10n_sa_log_results` | internal rule | self, xml_content, response_data, error | `l10n_sa_edi` |  | Save submitted invoice XML hash in case of either Rejection or Acceptance. |
| `_is_l10n_sa_eligibile_invoice` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_is_in_chain` | internal rule | self | `l10n_sa_edi` |  | If the invoice was successfully posted and confirmed by the government, then this would return True. If the invoice timed out, then its edi_document should still be in the 'to_send' state. |
| `action_show_chain_head` | user action | self | `l10n_sa_edi` |  | Action to show the chain head of the invoice |
| `l10n_sa_pos_ensure_invoice_pdf` | operation | self | `l10n_sa_edi_pos` |  | Generate the invoice PDF on demand for SA POS.  ZATCA clearance/reporting is already processed at checkout time. This generates only the PDF (no ZATCA re-submission, no email) so that "Reprint Invoice" serves the real signed invoice rather than a proforma. |
| `_get_invoice_reference_se_ocr2` | preparation rule | self, reference | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr3` | preparation rule | self, reference | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr4` | preparation rule | self, reference | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr2_invoice` | preparation rule | self | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr3_invoice` | preparation rule | self | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr4_invoice` | preparation rule | self | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr2_partner` | preparation rule | self | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr3_partner` | preparation rule | self | `l10n_se` |  |  |
| `_get_invoice_reference_se_ocr4_partner` | preparation rule | self | `l10n_se` |  |  |
| `_l10n_se_check_payment_reference` | validation | self | `l10n_se` | constrains: `payment_reference`, `state` |  |
| `_l10n_sg_get_uuid` | internal rule | self | `l10n_sg_ubl_pint` |  | SG Pint requires us to generate a uuid, to avoid storing a new field on the move, we derive it from the dbuuid and the move id. |
| `_get_invoice_reference_si_partner` | preparation rule | self | `l10n_si` |  | Generate the Slovenian structured payment reference using the partner's ID. Format: SI01 (P1-P2-P3)K - P1: Last two digits of the invoice year - P2: Partner ID - P3: Journal ID - K: Check digit  :return: the formatted structured reference string (SI01...) |
| `_get_invoice_reference_si_invoice` | preparation rule | self | `l10n_si` |  | Generate the Slovenian structured payment reference using the invoice sequence number.  Format: SI01 (P1-P2-P3)K - P1: Last two digits of the invoice year - P2: Trailing digits of the invoice name (sequence number) - P3: Journal ID - K: Check digit  :return: the formatted structured reference string (SI01...) |
| `_build_invoice_reference` | internal rule | self, p3 | `l10n_si` |  | Builds the reference using a shared structure for both methods. |
| `_l10n_tr_types_to_update_status` | internal rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_document_category` | internal rule | self, invoice_channel | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_category_move_type` | internal rule | self, document_category | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_submit_einvoice` | internal rule | self, xml_file, customer_alias | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_submit_earchive` | internal rule | self, xml_file | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_submit_document` | internal rule | self, xml_file, endpoint, post_series | `l10n_tr_nilvera_einvoice` |  | Submits an e-invoice or e-archive document to Nilvera for processing.  :param xml_file: The XML file to be submitted. :type xml_file: file-like object :param endpoint: The Nilvera API endpoint for submission. :type endpoint: str :param post_series: Whether to attempt posting the series/sequence to Nilvera if it is missing.                     Defaults to True. Useful for avoiding an infinite loop. :type post_series: bool :raises UserError: If the API key lacks necessary rights (401 or 403 responses), if the response                     indicates a client error (4xx), or if a server error occur |
| `_l10n_tr_nilvera_post_series` | internal rule | self, endpoint, client | `l10n_tr_nilvera_einvoice` |  | Post the series to Nilvera based on the endpoint. |
| `_l10n_tr_nilvera_get_submitted_document_status` | internal rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_nilvera_last_fetch_date` | preparation rule | self, invoice_channel, journal_type | `l10n_tr_nilvera_einvoice` |  | Fetches the last fetched date for Nilvera e-invoice synchronization specific to the current company. If no value exists, it sets a default date of one month prior to the current date, stores it, and returns it. |
| `_l10n_tr_nilvera_get_documents` | internal rule | self, invoice_channel, document_category, journal_type | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_nilvera_invoice_journal` | internal rule | self, journal_type | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_get_document_category_default_journal` | internal rule | self, journal_type | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_build_document_uuids_list` | internal rule | self, response | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_get_invoice_from_uuid` | internal rule | self, client, journal, document_uuid, document_category, invoice_channel | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_add_pdf_to_invoice` | internal rule | self, client, invoice, document_uuid, document_category, invoice_channel | `l10n_tr_nilvera_einvoice` |  |  |
| `l10n_tr_nilvera_get_pdf` | operation | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_einvoice_get_error_messages_from_response` | internal rule | self, response | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_einvoice_check_invalid_subscription_dates` | internal rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_einvoice_check_negative_lines` | internal rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_l10n_tr_nilvera_einvoice_check_lines_missing_taxes` | internal rule | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_get_partner_l10n_tr_nilvera_customer_alias_name` | preparation rule | self | `l10n_tr_nilvera_einvoice_extended`, `l10n_tr_nilvera_einvoice` |  |  |
| `_get_invoice_nilvera_pdf_report_filename` | preparation rule | self | `l10n_tr_nilvera_einvoice` |  | Get the filename of the Nilvera PDF invoice report. |
| `_l10n_tr_nilvera_company_get_documents` | internal rule | self, invoice_channel, category, journal_type | `l10n_tr_nilvera_einvoice` |  |  |
| `_cron_nilvera_get_new_einvoice_purchase_documents` | background operation | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_cron_nilvera_get_new_einvoice_sale_documents` | background operation | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_cron_nilvera_get_new_earchive_sale_documents` | background operation | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_cron_nilvera_get_invoice_status` | background operation | self | `l10n_tr_nilvera_einvoice` |  |  |
| `_cron_nilvera_get_sale_pdf` | background operation | self, batch_size | `l10n_tr_nilvera_einvoice` |  | Fetches the Nilvera generated PDFs for the sales generated on the system. |
| `_compute_l10n_tr_exemption_code_domain_list` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `l10n_tr_gib_invoice_scenario`, `l10n_tr_gib_invoice_type`, `l10n_tr_is_export_invoice` |  |
| `_compute_l10n_tr_gib_invoice_type` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `l10n_tr_gib_invoice_scenario`, `l10n_tr_is_export_invoice` |  |
| `_compute_l10n_tr_exemption_code_id` | computation | self | `l10n_tr_nilvera_einvoice_extended` | depends: `l10n_tr_gib_invoice_scenario`, `l10n_tr_gib_invoice_type`, `partner_id` |  |
| `_compute_is_print` | computation | self | `l10n_tw_edi_ecpay` | depends: `l10n_tw_edi_love_code`, `l10n_tw_edi_carrier_type`, `partner_id` |  |
| `_compute_love_code` | computation | self | `l10n_tw_edi_ecpay` | depends: `l10n_tw_edi_is_print`, `l10n_tw_edi_carrier_type`, `partner_id` |  |
| `_compute_carrier_info` | computation | self | `l10n_tw_edi_ecpay` | depends: `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code` |  |
| `_compute_l10n_tw_edi_invoice_type` | computation | self | `l10n_tw_edi_ecpay` | depends: `invoice_line_ids.tax_ids` |  |
| `_compute_l10n_tw_edi_is_zero_tax_rate` | computation | self | `l10n_tw_edi_ecpay` | depends: `invoice_line_ids.tax_ids` |  |
| `_compute_l10n_tw_edi_is_b2b` | computation | self | `l10n_tw_edi_ecpay` | depends: `partner_id` |  |
| `_l10n_tw_edi_check_tax_type_on_invoice_lines` | internal rule | self | `l10n_tw_edi_ecpay` |  | Check the tax type and special tax type on the invoice lines |
| `_l10n_tw_edi_determine_tax_types` | internal rule | self | `l10n_tw_edi_ecpay` |  | Calculate and return the tax type, special tax type and is zero tax rate included based on the taxes on invoice lines  :return: A tuple containing the tax type information.     - tax_type (str): The tax type ("1", "2", "3", "4") or "9" for mixed taxes.     - special_tax_type (int): The special tax type code.     - is_zero_tax_rate (bool): True if it is zero tax rate. |
| `_l10n_tw_edi_convert_currency_to_twd` | internal rule | self, amount | `l10n_tw_edi_ecpay` |  | Convert currency to TWD if the currency is not TWD |
| `_reformat_phone_number` | internal rule | self, phone | `l10n_tw_edi_ecpay` | model | Cleans and reformats a phone number string by handling different input formats.  The method first replaces a leading plus sign ('+') with a '0' and removes any following space. It then removes all non-digit characters (including spaces, dashes, and parentheses) to return a clean, continuous number string.  :param phone: The phone number as a string. :type phone: str :return: A cleaned and reformatted phone number string. :rtype: str  Example:     # Replaces leading '+' with '0' and removes spaces/dashes     _reformat_phone_number('+1 555-123-4567')  # returns '05551234567'      # Removes space |
| `_l10n_tw_edi_check_before_generate_invoice_json` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_tw_edi_prepare_item_list` | internal rule | self, json_data, is_allowance | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_tw_edi_generate_invoice_json` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_tw_edi_check_before_generate_issue_allowance_json` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_tw_edi_generate_issue_allowance_json` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_tw_edi_send_create_buyer` | internal rule | self | `l10n_tw_edi_ecpay` |  | Create a buyer before issuing B2B invoices |
| `_l10n_tw_edi_send` | internal rule | self, json_content | `l10n_tw_edi_ecpay` |  | Issuing an e-invoice by calling the Ecpay API and update the invoicing result in the system |
| `_l10n_tw_edi_update_ecpay_invoice_info` | internal rule | self | `l10n_tw_edi_ecpay` |  | Searching the e-invoice information from Ecpay API and update the invoice information in the system |
| `_l10n_tw_edi_run_invoice_invalid` | internal rule | self | `l10n_tw_edi_ecpay` |  | Cancelling the e-invoice by calling the Ecpay API and update the invoice information in the system |
| `_l10n_tw_edi_issue_allowance` | internal rule | self, json_content | `l10n_tw_edi_ecpay` |  | Issuing an allowance by calling the Ecpay API and update the refund invoice information in the system Two methods to issue the allowance 1. Endpoint: /Allowance     General allowance, which requires merchants or sellers to get the agreement from the customer first     (not by using ECPay system)     and then to send an API request to ECPay to issue an allowance. 2. Endpoint: /AllowanceByCollegiate     Sending an API request to ECPay and ECPay will send an e-mail notification with a link to the customer to     get his/her agreement     ONce the customer clicks the link, an allowance will be issued in |
| `_l10n_tw_edi_print_invoice` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_uy_get_formatted_sequence` | internal rule | self, number | `l10n_uy` |  |  |
| `_compute_l10n_vn_edi_invoice_state` | computation | self | `l10n_vn_edi_viettel` | depends: `payment_state` | Automatically set the state to payment_state_to_update when the payment state is updated.  This is a bit simplistic, as it can be wrongly set (for example, no need to send when going from in_payment to paid) But this shouldn't be an issue since the logic to send the update will check if anything need to change. |
| `_compute_l10n_vn_edi_invoice_symbol` | computation | self | `l10n_vn_edi_viettel` | depends: `company_id`, `partner_id` | Use the property l10n_vn_edi_symbol to set a default invoice symbol. |
| `_l10n_vn_edi_fetch_invoice_file_data` | internal rule | self, file_format | `l10n_vn_edi_viettel` |  | Helper to try fetching a few time in case the files are not yet ready. |
| `_l10n_vn_edi_try_fetch_invoice_file_data` | internal rule | self, file_format | `l10n_vn_edi_viettel` |  | Query sinvoice in order to fetch the data representation of the invoice, either zip or pdf. |
| `_l10n_vn_edi_fetch_invoice_xml_file_data` | internal rule | self | `l10n_vn_edi_viettel` |  | Query sinvoice in order to fetch the xsl and xml data representation of the invoice.  Returns a list of tuple with both file names, mimetype, content and the field it should be stored in. |
| `_l10n_vn_edi_fetch_invoice_pdf_file_data` | internal rule | self | `l10n_vn_edi_viettel` |  | Query sinvoice in order to fetch the pdf data representation of the invoice.  Returns a tuple with the pdf name, mimetype, content and field. |
| `_l10n_vn_edi_fetch_invoice_files` | internal rule | self | `l10n_vn_edi_viettel` |  | Fetches the SInvoice XML and PDF data from the SInvoice server if self is a sent invoice. The files are saved in the l10n_vn_edi_sinvoice_pdf_file_id and l10n_vn_edi_sinvoice_xml_file_id. |
| `action_l10n_vn_edi_update_payment_status` | user action | self | `l10n_vn_edi_viettel` |  | Send a request to update the payment status of the invoice. |
| `_l10n_vn_need_cancel_request` | internal rule | self | `l10n_vn_edi_viettel` |  |  |
| `_l10n_vn_edi_check_invoice_configuration` | internal rule | self | `l10n_vn_edi_viettel` |  | Some checks that are used to avoid common errors before sending the invoice. |
| `_l10n_vn_edi_send_invoice` | internal rule | self, invoice_json_data | `l10n_vn_edi_viettel` |  | Send an invoice to the SInvoice system.  Handles lookup on the system in order to ensure that the invoice was not sent successfully yet in case of timeout or other unforeseen error. |
| `_l10n_vn_edi_cancel_invoice` | internal rule | self, reason, agreement_document_name, agreement_document_date | `l10n_vn_edi_viettel` |  | Send a request to cancel the invoice. |
| `_l10n_vn_edi_generate_invoice_json` | internal rule | self | `l10n_vn_edi_viettel` |  | Return the dict of data that will be sent to the api in order to create the invoice. |
| `_l10n_vn_edi_add_general_invoice_information` | internal rule | self, json_values | `l10n_vn_edi_viettel` |  | General invoice information, such as the model number, invoice symbol, type, date of issues, ... |
| `_l10n_vn_edi_add_buyer_information` | internal rule | self, json_values | `l10n_vn_edi_viettel_pos`, `l10n_vn_edi_viettel` |  | Create and return the buyer information for the current invoice. |
| `_l10n_vn_edi_add_seller_information` | internal rule | self, json_values | `l10n_vn_edi_viettel` |  | Create and return the seller information for the current invoice. |
| `_l10n_vn_edi_add_payment_information` | internal rule | self, json_values | `l10n_vn_edi_viettel` |  | Create and return the payment information for the current invoice. Not fully supported. |
| `_l10n_vn_edi_add_item_information` | internal rule | self, json_values | `l10n_vn_edi_viettel` |  | Create and return the items information for the current invoice. |
| `_l10n_vn_edi_add_tax_breakdowns` | internal rule | self, json_values | `l10n_vn_edi_viettel` |  | Create and return the tax breakdown of the current invoice. |
| `_l10n_vn_edi_lookup_invoice` | internal rule | self | `l10n_vn_edi_viettel` |  | Lookup on invoice, returning its current details on SInvoice. |
| `_l10n_vn_edi_get_access_token` | internal rule | self | `l10n_vn_edi_viettel` |  | Return an access token to be used to contact the API. Either take a valid stored one or get a new one. |
| `_l10n_vn_edi_get_credentials_company` | internal rule | self | `l10n_vn_edi_viettel` |  | The company holding the credentials could be one of the parent companies. We need to ensure that:     - We use the credentials of the parent company, if no credentials are set on the child one.     - We store the access token on the appropriate company, based on which holds the credentials. |
| `_l10n_vn_edi_format_date` | internal rule | self, date | `l10n_vn_edi_viettel` | model | All APIs for Sinvoice uses the same time format, being the current hour, minutes and seconds converted into seconds since unix epoch, but formatting like milliseconds since unix epoch. It means that the time will end in 000 for the milliseconds as they are not as of today used by the system. |
| `_l10n_vn_edi_format_phone_number` | internal rule | self, number | `l10n_vn_edi_viettel` | model | Simple helper that takes in a phone number and try to format it to fit sinvoice format. SInvoice only allows digits, so we will remove any (, ), -, + characters. |
| `_l10n_vn_edi_is_sent` | internal rule | self | `l10n_vn_edi_viettel` |  | Small helper that returns true if self has been sent to sinvoice. |
| `l10n_vn_edi_fetch_invoice_files` | operation | self | `l10n_vn_edi_viettel_pos` |  | Fetches the SInvoice XML and PDF data from the SInvoice server if self is a sent invoice. The files are saved in the l10n_vn_edi_sinvoice_pdf_file_id and l10n_vn_edi_sinvoice_xml_file_id. |
| `_compute_wip_production_count` | computation | self | `mrp_account` | depends: `wip_production_ids` |  |
| `action_view_wip_production` | user action | self | `mrp_account` |  |  |
| `_compute_landed_costs_visible` | computation | self | `stock_landed_costs` | depends: `line_ids`, `line_ids.is_landed_costs_line` |  |
| `button_create_landed_costs` | user action | self | `stock_landed_costs` |  | Create a `stock.landed.cost` record associated to the account move of `self`, each `stock.landed.costs` lines mirroring the current `account.move.line` of self. |
| `action_view_landed_costs` | user action | self | `stock_landed_costs` |  |  |
| `invoice_validate_send_email` | operation | self | `product_email_template` |  |  |
| `_get_action_per_item` | preparation rule | self | `sale_project` |  |  |
| `_compute_timesheet_total_duration` | computation | self | `sale_timesheet` | depends: `timesheet_ids`, `company_id.timesheet_encode_uom_id` |  |
| `_compute_timesheet_count` | computation | self | `sale_timesheet` | depends: `timesheet_ids` |  |
| `action_view_timesheet` | user action | self | `sale_timesheet` |  |  |
| `_link_timesheets_to_invoice` | internal rule | self, start_date, end_date | `sale_timesheet` |  | Search timesheets from given period and link this timesheets to the invoice  When we create an invoice from a sale order, we need to link the timesheets in this sale order to the invoice. Then, we can know which timesheets are invoiced in the sale order. :param start_date: the start date of the period :param end_date: the end date of the period |
| `_get_range_dates` | preparation rule | self, order | `sale_timesheet` |  |  |
| `unlink_snailmail_letters` | operation | self | `snailmail_account` | ondelete |  |

## Validation and error messages (167)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search_default_journal` | UserError | error_msg | `account` |
| `_inverse_company_id` | ValidationError | We can't leave this document without any company. Please select a company for this document. | `account` |
| `_onchange_partner_id` | RedirectWarning | msg | `account` |
| `_check_balanced` | UserError | error_msg | `account` |
| `_check_balanced` | UserError | The entry is not balanced. | `account` |
| `_check_fiscal_lock_dates` | UserError | message | `account` |
| `_require_bill_date_for_autopost` | ValidationError | For this entry to be automatically posted, it required a bill date. | `account` |
| `_check_journal_move_type` | ValidationError | Cannot create a purchase document in a non purchase journal | `account` |
| `_check_journal_move_type` | ValidationError | Cannot create a sale document in a non sale journal | `account` |
| `_validate_taxes_country` | ValidationError | This entry contains one or more taxes that are incompatible with your fiscal country. Check company fiscal country in the settings and tax country in taxes configuration. | `account` |
| `_validate_taxes_country` | ValidationError | This entry contains taxes that are not compatible with your fiscal position. Check the country set in fiscal position and in your tax configuration. | `account` |
| `_check_invoice_currency_rate` | ValidationError | The currency rate must be strictly positive. | `account` |
| `create` | UserError | You cannot create a move already in the posted state. Please create a draft move and post it after. | `account` |
| `write` | AccessError | You don't have the access rights to perform this action. | `account` |
| `write` | ValidationError | Validated entries can only be changed by your accountant. | `account` |
| `write` | UserError | This document is protected by a hash. Therefore, you cannot edit the following fields: %s. | `account` |
| `write` | UserError | You cannot edit the journal of an account move if it has been posted once, unless the name is removed or set to "/". This might create a gap in the sequence. | `account` |
| `write` | UserError | You cannot edit the journal of an account move with a sequence number assigned, unless the name is removed or set to "/". This might create a gap in the sequence. | `account` |
| `write` | UserError | You cannot modify the following readonly fields on the posted move %(move)s: %(fields)s | `account` |
| `write` | UserError | The Journal Entry sequence is not conform to the current format. Only the Accountant can change it. | `account` |
| `_unlink_forbid_parts_of_chain` | UserError | You cannot delete this entry, as it has already consumed a sequence number and is not the last one in the chain. You should probably revert it instead. | `account` |
| `_unlink_account_audit_trail_except_once_post` | UserError | To keep the restrictive audit trail, you can not delete journal entries once they have been posted. Instead, you can cancel the journal entry. | `account` |
| `_get_invoice_computed_reference` | UserError | The combination of reference model and reference type on the journal is not implemented | `account` |
| `_get_chains_to_hash` | UserError | An error occurred when computing the inalterability. All entries have to be reconciled. | `account` |
| `_get_chains_to_hash` | UserError | This move could not be locked either because some move with the same sequence prefix has a higher number. You may need to resequence it. | `account` |
| `_get_chains_to_hash` | UserError | An error occurred when computing the inalterability. A gap has been detected in the sequence. | `account` |
| `_post` | AccessError | You don't have the access rights to post an invoice. | `account` |
| `_post` | UserError | msg | `account` |
| `_post` | UserError | You cannot post an entry with an archived analytic account: %s | `account` |
| `_post` | RedirectWarning | The company bank account (%(account_number)s) linked to this invoice is not trusted. Go to the Bank Settings, double-check that it is yours or correct the number, and click on Send Money to trust it. | `account` |
| `_post` | UserError | The bank account of your company is not trusted. Please ask an admin or someone with approval rights to check it. | `account` |
| `action_switch_move_type` | ValidationError | You cannot switch the type of a document with an existing sequence number. | `account` |
| `action_switch_move_type` | ValidationError | This action isn't available for this document. | `account` |
| `action_register_payment` | UserError | You can only register payment for posted journal entries. | `account` |
| `action_force_register_payment` | UserError | You cannot register payments for miscellaneous entries. | `account` |
| `action_force_register_payment` | UserError | You cannot register payments for blocked invoices. | `account` |
| `action_validate_moves_with_confirmation` | UserError | There are no journal items in the draft state to post. | `account` |
| `button_draft` | UserError | Only posted/cancelled journal entries can be reset to draft. | `account` |
| `button_draft` | UserError | You can't reset to draft those journal entries. You need to request a cancellation instead. | `account` |
| `_check_draftable` | UserError | You cannot reset to draft an exchange difference journal entry. | `account` |
| `_check_draftable` | UserError | You cannot reset to draft a tax cash basis journal entry. | `account` |
| `_check_draftable` | UserError | You cannot reset to draft a locked journal entry. | `account` |
| `button_request_cancel` | UserError | You can only request a cancellation for invoice sent to the government. | `account` |
| `button_cancel` | UserError | Only draft journal entries can be cancelled. | `account` |
| `action_toggle_block_payment` | UserError | You can't block a paid invoice. | `account` |
| `_generate_qr_code` | UserError | error_msg | `account` |
| `_get_available_invoice_template_pdf_report_ids` | UserError | There is no template that applies to invoices. | `account` |
| `_post` | UserError | Invalid invoice configuration:  %s | `account_edi` |
| `button_draft` | UserError | You can't edit the following journal entry %s because an electronic document has already been sent. Please use the 'Request EDI Cancellation' button instead. | `account_edi` |
| `_ungroup_lines` | UserError | error_message | `account_edi_ubl_cii` |
| `_ungroup_lines` | UserError | Cannot decode origin file, try by importing it again | `account_edi_ubl_cii` |
| `_ungroup_lines` | UserError | error_message | `account_edi_ubl_cii` |
| `_group_lines_by_tax` | UserError | You can only group lines of an invoice | `account_edi_ubl_cii` |
| `_check_move_for_group_ungroup_lines_by_tax` | UserError | You can only (un)group lines of a draft invoice | `account_edi_ubl_cii` |
| `action_cancel_peppol_documents` | UserError | Cannot cancel an entry that has already been sent to PEPPOL | `account_peppol` |
| `_check_expense_ids` | ValidationError | Each expense paid by the company must have a distinct and dedicated journal entry. | `hr_expense` |
| `_post` | UserError | We do not accept the usage of document types on receipts yet. | `l10n_latam_invoice_document` |
| `_check_l10n_latam_documents` | ValidationError | The journal require a document type but not document type has been selected on invoices %s. | `l10n_latam_invoice_document` |
| `_check_l10n_latam_documents` | ValidationError | Please set the document number on the following invoices %s. | `l10n_latam_invoice_document` |
| `_check_invoice_type_document_type` | ValidationError | You can not use a %s document type with a refund invoice | `l10n_latam_invoice_document` |
| `_check_invoice_type_document_type` | ValidationError | You can not use a %s document type with a invoice | `l10n_latam_invoice_document` |
| `_check_moves_use_documents` | ValidationError | The selected Journal can't be used in this transaction, please select one that doesn't use documents as these are just for Invoices. | `l10n_ar` |
| `_check_argentinean_invoice_taxes` | UserError | There should be a single tax from the “VAT“ tax group per line, but this is not the case for line “%s”. Please add a tax to this line or check the tax configuration's advanced options for the corresponding field “Tax Group”. | `l10n_ar` |
| `_check_argentinean_invoice_taxes` | UserError | On invoice id “%s” you must use VAT Not Applicable on every line. | `l10n_ar` |
| `_check_argentinean_invoice_taxes` | UserError | On invoice id “%s” you must use a VAT tax that is not VAT Not Applicable | `l10n_ar` |
| `_onchange_partner_journal` | RedirectWarning | msg | `l10n_ar` |
| `_inverse_l10n_latam_document_number` | UserError | The document number can not be changed for this journal, you can only modify the POS number if there is not posted (or posted before) invoices | `l10n_ar` |
| `l10n_ch_action_print_qr` | UserError | Only customers invoices can be QR-printed. | `l10n_ch` |
| `_check_l10n_latam_document_number_is_numeric` | ValidationError | The DTE document number (folio) must contain only digits. | `l10n_cl` |
| `_check_document_types_post` | ValidationError | Tax payer type and vat number are mandatory for this type of document. Please set the current tax payer type of this customer | `l10n_cl` |
| `_check_document_types_post` | ValidationError | The DIN document is intended to be used only with RUT 60805000-0 (Tesorería General de La República) | `l10n_cl` |
| `_check_document_types_post` | ValidationError | The tax payer type of this supplier is incorrect for the selected type of document. | `l10n_cl` |
| `_check_document_types_post` | ValidationError | You need a journal without the use of documents for foreign suppliers | `l10n_cl` |
| `_check_document_types_post` | ValidationError | Document types for foreign customers must be export type (codes 110, 111 or 112) or you should define the customer as an end consumer and use receipts (codes 39 or 41) | `l10n_cl` |
| `_check_document_types_post` | ValidationError | Tax payer type and vat number are mandatory for this type of document. Please set the current tax payer type of this supplier | `l10n_cl` |
| `_check_document_types_post` | ValidationError | The tax payer type of this supplier is not entitled to deliver fees documents | `l10n_cl` |
| `_check_document_types_post` | ValidationError | The tax payer type of this supplier is not entitled to deliver imports documents | `l10n_cl` |
| `_check_fapiao` | ValidationError | Fapiao number is an 8-digit number. Please enter a correct one. | `l10n_cn` |
| `_get_invoice_reference_dk_fik` | ValidationError | FIK %(prefix)s reference cannot be generated: invoice number '%(invoice)s' has more than %(max_digits)s digits. | `l10n_dk_fik` |
| `action_cancel_nemhandel_documents` | UserError | Cannot cancel an entry that has already been sent to Nemhandel | `l10n_dk_nemhandel` |
| `action_post_sign_invoices` | UserError | Please only sign invoices from one company at a time | `l10n_eg_edi_eta` |
| `action_post_sign_invoices` | ValidationError | Please setup a personal drive for company %s | `l10n_eg_edi_eta` |
| `action_post_sign_invoices` | ValidationError | Please setup the certificate on the thumb drive menu | `l10n_eg_edi_eta` |
| `_l10n_es_edi_facturae_get_corrective_data` | UserError | The credit note/refund appears to have been issued manually. For the purpose of generating a Facturae document, it's necessary that the credit note/refund is created directly from the associated invoice/bill. | `l10n_es_edi_facturae` |
| `_l10n_es_edi_facturae_export_facturae` | UserError | The company needs a set tax identification number or VAT number | `l10n_es_edi_facturae` |
| `_l10n_es_edi_facturae_export_facturae` | UserError | The partner needs a set tax identification number or VAT number | `l10n_es_edi_facturae` |
| `_l10n_es_edi_facturae_export_facturae` | UserError | The partner needs a set country | `l10n_es_edi_facturae` |
| `_l10n_es_facturae_sign_xml` | UserError | No valid certificate found | `l10n_es_edi_facturae` |
| `button_draft` | UserError | You cannot reset to draft an entry that has been posted to TicketBAI's chain | `l10n_es_edi_tbai` |
| `_l10n_es_tbai_unlink_except_in_chain` | UserError | You cannot delete a move that has a TicketBAI chain id. | `l10n_es_edi_tbai` |
| `_l10n_es_tbai_lock_move` | UserError | Cannot send this entry as it is already being processed. | `l10n_es_edi_tbai` |
| `l10n_es_tbai_resend_bill` | UserError | error | `l10n_es_edi_tbai` |
| `l10n_es_tbai_send_bill` | UserError | error | `l10n_es_edi_tbai` |
| `l10n_es_tbai_cancel` | UserError | You cannot reset to draft a locked journal entry. | `l10n_es_edi_tbai` |
| `l10n_es_tbai_cancel` | UserError | error | `l10n_es_edi_tbai` |
| `l10n_es_tbai_cancel` | UserError | edi_document.response_message | `l10n_es_edi_tbai` |
| `number2numeric` | UserError | Invoice number must contain numeric characters | `l10n_fi` |
| `action_pdp_open_response_wizard` | UserError | Cannot send response for any of the journal entries. | `l10n_fr_pdp` |
| `l10n_gr_edi_try_send_expense_classification` | UserError | error_message | `l10n_gr_edi` |
| `l10n_gr_edi_try_send_expense_classification` | UserError | error_message | `l10n_gr_edi` |
| `_l10n_gr_edi_try_send_batch` | UserError | You should use Send & Print wizard for sending customer invoices to myDATA. | `l10n_gr_edi` |
| `_l10n_gr_edi_try_send_batch` | UserError | Some of the selected moves does not meet the requirements to be sent to myDATA. | `l10n_gr_edi` |
| `_check_draftable` | UserError | You cannot reset this invoice to draft. | `l10n_gr_edi_e_invoo` |
| `_check_l10n_hr_process_type` | ValidationError | Business Process Type P9 can only be used with credit notes. | `l10n_hr_edi` |
| `_check_l10n_hr_process_type` | ValidationError | Credit notes must use Business Process Type P9 or P10. | `l10n_hr_edi` |
| `_post` | UserError | This vendor bill is already rejected according to the Tax Authority. | `l10n_hr_edi` |
| `_check_posted_if_active` | ValidationError | Cannot reset to draft or cancel invoice %s because an electronic document was already sent to NAV! | `l10n_hu_edi` |
| `l10n_hu_edi_button_update_status` | UserError | error_text | `l10n_hu_edi` |
| `_l10n_hu_edi_acquire_lock` | UserError | Could not acquire lock on invoices - is another user performing operations on them? | `l10n_hu_edi` |
| `_l10n_hu_edi_get_invoice_values` | UserError | Please create a sales tax with type ATK (outside the scope of the VAT Act). | `l10n_hu_edi` |
| `download_efaktur` | UserError | You are not allowed to generate e-Faktur document from invoices coming from different companies | `l10n_id_efaktur_coretax` |
| `download_efaktur` | ValidationError | '\n - '.join(err_messages) | `l10n_id_efaktur_coretax` |
| `download_efaktur` | RedirectWarning | msg | `l10n_id_efaktur_coretax` |
| `_post` | UserError | Please set a valid TIN Number on the Place of Supply %s | `l10n_in` |
| `_post` | RedirectWarning | msg | `l10n_in` |
| `_post` | ValidationError | Partner %(partner_name)s (%(partner_id)s) GSTIN is required under GST Treatment %(name)s | `l10n_in` |
| `_l10n_in_lock_invoice` | UserError | This electronic document is being processed already. | `l10n_in_edi` |
| `action_l10n_in_ewaybill_create` | UserError | Ewaybill already created for this move. | `l10n_in_ewaybill` |
| `action_check_l10n_it_edi` | UserError | This move is not waiting for updates from the SdI. | `l10n_it_edi` |
| `_l10n_it_edi_send` | UserError | This document is being sent by another process already. | `l10n_it_edi` |
| `_l10n_it_edi_update_send_state` | UserError | An error occurred while downloading updates from the Proxy Server: (%(code)s) %(message)s | `l10n_it_edi` |
| `_l10n_it_edi_update_send_state` | UserError | An error occurred while downloading updates from the Proxy Server: (%(code)s) %(message)s | `l10n_it_edi` |
| `_check_l10n_it_edi_doi_id` | UserError | '\n'.join(validity_errors) | `l10n_it_edi_doi` |
| `_post` | UserError | '\n'.join(errors) | `l10n_it_edi_doi` |
| `download_l10n_jo_edi_computed_xml` | ValidationError | The following errors have to be fixed in order to create an XML: | `l10n_jo_edi` |
| `l10n_ke_action_cu_post` | UserError | An OSCU has been initialized for this company. Please send the e-invoice via Send and Print -> Send to eTIMS instead. | `l10n_ke_edi_tremol` |
| `l10n_ke_action_cu_post` | UserError | error_msg | `l10n_ke_edi_tremol` |
| `_constrains_l10n_lk_sequence_length` | UserError | Invoice number exceeds %(max)d characters: %(name)s | `l10n_lk_invoice` |
| `_get_last_sequence` | ValidationError | %(field_name)s is not a stored field | `l10n_lk_invoice` |
| `action_invoice_sent` | UserError | You cannot send invoices that are currently being validated. Please wait for the validation to complete. | `l10n_my_edi` |
| `action_open_l10n_ph_2307_wizard` | UserError | Only Vendor Bills are available. | `l10n_ph` |
| `action_l10n_pl_edi_get_invoice_UPO` | UserError | This invoice does not have a KSeF Invoice Reference Number. It may not have been sent yet. | `l10n_pl_edi` |
| `action_l10n_pl_edi_get_invoice_UPO` | UserError | You can only download a UPO for an 'Accepted' invoice. Please update the status first. | `l10n_pl_edi` |
| `action_l10n_pl_edi_get_invoice_UPO` | UserError | The KSeF service returned empty UPO content. | `l10n_pl_edi` |
| `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Purchase tax corresponding to '%s' required for the KSeF import was not found in the system. | `l10n_pl_edi` |
| `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Currency '%s' from the KSeF bill was not found. | `l10n_pl_edi` |
| `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Purchase tax corresponding to '%s' required for the KSeF import was not found in the system. | `l10n_pl_edi` |
| `l10n_pl_edi_get_ksef_bill_vals_from_xml` | UserError | Tax corresponding to '%s' required to derive the net unit price from gross price during KSeF import could not be interpreted. | `l10n_pl_edi` |
| `_handle_download_bills_from_ksef_error` | UserError | error.get('message') | `l10n_pl_edi` |
| `_l10n_ro_edi_fetch_invoices` | UserError | result['error'] | `l10n_ro_edi` |
| `_get_normalized_l10n_sa_confirmation_datetime` | UserError | Please set the Invoice Date to be either less than or equal to today as per the Asia/Riyadh time zone, since ZATCA does not allow future-dated invoicing. | `l10n_sa` |
| `_prevent_zatca_rejected_invoice_deletion` | UserError | The Invoice(s) are linked to a validated EDI document and cannot be modified according to ZATCA rules | `l10n_sa_edi` |
| `button_draft` | UserError | The Invoice(s) are linked to a validated EDI document and cannot be modified according to ZATCA rules | `l10n_sa_edi` |
| `_get_invoice_reference_se_ocr4` | UserError | OCR Reference Number length is greater than allowed. Allowed length in invoice journal setting is %s. | `l10n_se` |
| `_l10n_se_check_payment_reference` | ValidationError | Vendor require OCR Number as payment reference. Payment reference isn't a valid OCR Number. | `l10n_se` |
| `button_draft` | UserError | You cannot reset to draft an entry that has been sent to Nilvera. | `l10n_tr_nilvera_einvoice` |
| `_post` | UserError | To preserve accounting integrity and comply with legal requirements, invoices cannot be reused once an error occurs. Please create a new invoice to continue. | `l10n_tr_nilvera_einvoice` |
| `_l10n_tr_nilvera_submit_document` | UserError | Oops, seems like you're unauthorised to do this. Try another API key with more rights or contact Nilvera. | `l10n_tr_nilvera_einvoice` |
| `_l10n_tr_nilvera_submit_document` | UserError | error_message | `l10n_tr_nilvera_einvoice` |
| `_l10n_tr_nilvera_submit_document` | UserError | Server error from Nilvera, please try again later. | `l10n_tr_nilvera_einvoice` |
| `_l10n_tw_edi_check_before_generate_invoice_json` | UserError | 'Error:\n' + '\n'.join(errors) | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | You cannot issue an allowance for invoice %(invoice_number)s as it was not sent to Ecpay. | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | Customer email is needed for notification | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_check_before_generate_issue_allowance_json` | UserError | Customer %(notify_way)s is needed for notification | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_update_ecpay_invoice_info` | UserError | The invoice: %(invoice_name)s has no related number | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_run_invoice_invalid` | UserError | You cannot invalidate an invoice that was not sent to Ecpay. | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_run_invoice_invalid` | UserError | The invoice: %(invoice_id)s has already been invalidated | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_run_invoice_invalid` | UserError | Fail to invalidate invoice. Error message: %(error_message)s | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_issue_allowance` | UserError | Fail to issue allowance for ECpay invoice. Error message: %(error_message)s | `l10n_tw_edi_ecpay` |
| `_l10n_tw_edi_print_invoice` | UserError | You cannot print an invoice that was not sent to Ecpay, without the print flag, or that is invalid. | `l10n_tw_edi_ecpay` |
| `_l10n_vn_edi_fetch_invoice_files` | UserError | Please send the invoice to SInvoice before fetching the tax invoice files. | `l10n_vn_edi_viettel` |
| `action_l10n_vn_edi_update_payment_status` | UserError | error_message | `l10n_vn_edi_viettel` |
| `action_l10n_vn_edi_update_payment_status` | UserError | error | `l10n_vn_edi_viettel` |
| `action_l10n_vn_edi_update_payment_status` | UserError | error_message | `l10n_vn_edi_viettel` |
| `_l10n_vn_edi_cancel_invoice` | UserError | error | `l10n_vn_edi_viettel` |
| `_l10n_vn_edi_cancel_invoice` | UserError | error_message | `l10n_vn_edi_viettel` |
| `_check_move_for_group_ungroup_lines_by_tax` | UserError | You can only (un)group lines of an invoice not linked to a purchase order | `purchase_edi_ubl_bis3` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | yes | yes | yes | yes | `account` |
| `base.group_portal` | no | yes | no | no | `account` |
| `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Account Entry | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| All Journal Entries | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Portal Personal Account Invoices | `[(4, ref('base.group_portal'))]` | `[('state', 'not in', ('cancel', 'draft')), ('move_type', 'in', ('out_invoice', 'out_refund', 'in_invoice', 'in_refund')), ('partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |
| Readonly Move | `[(4, ref('account.group_account_readonly'))]` | `[(1, '=', 1)]` | True | False | False | False |
| Readonly Move | `[(4, ref('account.group_account_invoice'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Expense Team Approver Account Move | `[(4, ref('hr_expense.group_hr_expense_team_approver'))]` | `[('expense_ids', '!=', False)]` | True | True | True | True |
| Invoice POS User | `[(4, ref('group_pos_user'))]` | `[('pos_order_ids', '!=', False)]` | True | True | True | True |
| Purchase User Account Move | `[(4, ref('purchase.group_purchase_user'))]` | `[('move_type', 'in', ('in_invoice', 'in_refund', 'in_receipt'))]` | True | True | True | True |
| Personal Invoices | `[(4, ref('sales_team.group_sale_salesman'))]` | `[('move_type', 'in', ('out_invoice', 'out_refund')), '\|', ('invoice_user_id', '=', user.id), ('invoice_user_id', '=', False)]` | True | True | True | True |
| All Invoices | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[('move_type', 'in', ('out_invoice', 'out_refund'))]` | True | True | True | True |

## Views (147)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_move_tree` | list |  | `company_currency_id`, `made_sequence_gap`, `invoice_date`, `date`, `name`, `checked`, `partner_id`, `ref`, `journal_id`, `company_id`, `amount_total_signed`, `state`, `currency_id`, `activity_ids` |  |  | `account` |
| `account.view_move_tree_multi_edit` | xpath | `account.view_move_tree` |  |  |  | `account` |
| `account.view_invoice_tree` | list |  | `made_sequence_gap`, `duplicated_ref_ids`, `is_exact_move_duplicate`, `name`, `checked`, `invoice_partner_display_name`, `invoice_partner_display_name`, `invoice_date`, `invoice_date`, `date`, `invoice_date_due`, `invoice_origin`, `payment_reference`, `ref`, `invoice_user_id`, `activity_ids`, `company_id`, `company_id`, `amount_untaxed_in_currency_signed`, `amount_tax_signed`, `amount_total_in_currency_signed`, `amount_residual_signed`, `currency_id`, `company_currency_id`, `status_in_payment`, `move_sent_values`, `move_type`, `abnormal_amount_warning`, `abnormal_date_warning` | `Pay` |  | `account` |
| `account.view_duplicated_moves_tree_js` | xpath | `account.view_invoice_tree` |  |  |  | `account` |
| `account.view_out_invoice_tree` | button | `account.view_invoice_tree` |  | `action_force_register_payment`, `Send` |  | `account` |
| `account.view_out_credit_note_tree` | button | `account.view_invoice_tree` |  | `action_force_register_payment`, `Send` |  | `account` |
| `account.view_in_invoice_tree` | xpath | `account.view_invoice_tree` |  |  |  | `account` |
| `account.view_in_invoice_bill_tree` | field | `account.view_in_invoice_tree` | `currency_id` |  |  | `account` |
| `account.view_in_invoice_refund_tree` | field | `account.view_in_invoice_tree` | `currency_id` |  |  | `account` |
| `account.view_account_move_kanban` | kanban |  | `currency_id`, `checked`, `partner_id`, `journal_id`, `amount_total_in_currency_signed`, `name`, `date`, `activity_ids`, `state` |  |  | `account` |
| `account.view_move_form` | form |  | `state`, `state`, `alerts`, `duplicated_ref_ids`, `duplicated_ref_ids`, `payment_count`, `adjusting_entries_count`, `adjusting_entry_origin_label`, `adjusting_entry_origin_moves_count`, `id`, `state`, `company_id`, `journal_id`, `show_name_warning`, `posted_before`, `move_type`, `payment_state`, `invoice_filter_type_domain`, `suitable_journal_ids`, `currency_id`, `company_currency_id`, `commercial_partner_id`, `bank_partner_id`, `display_qr_code`, `show_reset_to_draft_button`, `expected_currency_rate`, `invoice_has_outstanding`, `is_move_sent`, `invoice_pdf_report_id`, `need_cancel_request`, `has_reconciled_entries`, `restrict_mode_hash_table`, `inalterable_hash`, `country_code`, `display_inactive_currency_warning`, `statement_line_id`, `statement_id`, `origin_payment_id`, `tax_country_id`, `tax_calculation_rounding_method`, `tax_cash_basis_created_move_ids`, `quick_edit_mode`, `hide_post_button`, `quick_encoding_vals`, `show_delivery_date`, `is_being_sent`, `show_update_fpos`, `is_sale_installed`, `move_type`, `highest_name`, `name`, `partner_id`, `partner_shipping_id`, `quick_edit_total_amount`, `ref`, `ref`, `tax_cash_basis_origin_move_id`, `invoice_vendor_bill_id`, `invoice_date`, `invoice_date` | `Post`, `Confirm`, `Send`, `Send`, `Print`, `Print`, `Pay`, `Pay`, `Preview`, `Reverse Entry`, `Credit Note`, `Cancel Entry`, `Cancel`, `Reset to Draft`, `Lock`, `Request Cancel`, `Reviewed`, `action_delete_duplicates`, `action_delete_duplicates`, `action_activate_currency`, `action_activate_currency`, `action_open_business_doc`, `open_payments`, `open_reconcile_view`, `open_created_caba_entries`, `open_adjusting_entries`, `open_adjusting_entry_origin_moves`, `Update Taxes and Accounts`, `refresh_invoice_currency_rate`, `refresh_invoice_currency_rate` |  | `account` |
| `account.account_move_view_activity` | activity |  | `currency_id`, `name`, `amount_total`, `commercial_partner_id`, `state` |  |  | `account` |
| `account.view_account_move_filter` | search |  | `name`, `name`, `ref`, `invoice_date`, `date`, `amount_total`, `partner_id`, `journal_id` |  | `Unposted`, `Posted`, `Not Secured`, `Reversed`, `Manual`, `Sales`, `Purchases`, `Bank`, `Cash`, `Credit`, `Miscellaneous`, `Date`, `Invoice Date`, `Partner`, `Journal`, `Status`, `Payment Method`, `Date`, `Invoice Date`, `Company` | `account` |
| `account.view_account_invoice_filter` | search |  | `name`, `name`, `ref`, `payment_reference`, `amount_total`, `journal_id`, `journal_group_id`, `partner_id`, `invoice_user_id`, `date`, `next_payment_date`, `line_ids`, `activity_user_id`, `activity_type_id` |  | `myinvoices`, `Draft`, `Posted`, `Cancelled`, `Not Secured`, `Not Sent`, `Invoices`, `Receipts`, `Credit Notes`, `To Review`, `To pay`, `In payment`, `Overdue`, `Invoice Date`, `Accounting Date`, `Due Date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Salesperson`, `Partner`, `Status`, `Payment Method`, `Journal`, `Company`, `Invoice Date`, `Due Date`, `Accounting Date`, `Sequence Prefix` | `account` |
| `account.view_account_bill_filter` | field | `account.view_account_invoice_filter` | `name` |  |  | `account` |
| `account.view_account_move_with_gaps_in_sequence_filter` | filter | `account.view_account_invoice_filter` |  |  | `due_date`, `Irregular Sequences` | `account` |
| `account_debit_note.view_move_form_debit` | div | `account.view_move_form` | `debit_note_count` | `action_view_debit_notes` |  | `account_debit_note` |
| `account_debit_note.view_account_move_filter_debit` | filter | `account.view_account_move_filter` |  |  | `reversed`, `Debit Note` | `account_debit_note` |
| `account_debit_note.view_account_invoice_filter_debit` | filter | `account.view_account_invoice_filter` |  |  | `out_refund`, `Debit Notes` | `account_debit_note` |
| `account_edi.view_out_invoice_tree_inherit` | field | `account.view_out_invoice_tree` | `status_in_payment`, `edi_state`, `edi_blocking_level`, `edi_error_message` |  |  | `account_edi` |
| `account_edi.view_out_credit_note_tree_inherit` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `edi_state`, `edi_blocking_level`, `edi_error_message` |  |  | `account_edi` |
| `account_edi.view_in_invoice_refund_tree_inherit` | field | `account.view_in_invoice_refund_tree` | `status_in_payment`, `edi_state`, `edi_blocking_level`, `edi_error_message` |  |  | `account_edi` |
| `account_edi.view_in_bill_tree_inherit` | field | `account.view_in_invoice_bill_tree` | `status_in_payment`, `edi_state`, `edi_blocking_level`, `edi_error_message` |  |  | `account_edi` |
| `account_edi.view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Electronic invoicing processing needed`, `Electronic invoicing state` | `account_edi` |
| `account_edi.view_move_form_inherit` | xpath | `account.view_move_form` | `edi_show_cancel_button`, `edi_show_abandon_cancel_button`, `edi_show_force_cancel_button` | `Request EDI Cancellation`, `Call off EDI Cancellation` |  | `account_edi` |
| `account_fleet.view_move_form` | xpath | `account.view_move_form` | `need_vehicle`, `vehicle_id` |  |  | `account_fleet` |
| `account_fleet.account_move_view_tree` | xpath | `account.view_move_tree` |  |  |  | `account_fleet` |
| `account_payment.account_invoice_view_form_inherit_payment` | xpath | `account.view_move_form` |  |  |  | `account_payment` |
| `account_peppol.account_peppol_view_move_form` | header | `account.view_move_form` |  | `Cancel PEPPOL` |  | `account_peppol` |
| `account_peppol.account_peppol_view_out_invoice_tree_inherit` | field | `account.view_out_invoice_tree` | `status_in_payment`, `peppol_move_state` |  |  | `account_peppol` |
| `account_peppol.account_peppol_view_out_credit_note_tree_inherit` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `peppol_move_state` |  |  | `account_peppol` |
| `account_peppol.account_peppol_view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Peppol status` | `account_peppol` |
| `account_peppol_advanced_fields.view_move_form_inherit_peppol` | xpath | `account.view_move_form` |  |  |  | `account_peppol_advanced_fields` |
| `account_peppol_response.account_peppol_response_view_move_form` | xpath | `account_peppol.account_peppol_view_move_form` | `peppol_move_state` |  |  | `account_peppol_response` |
| `hr_expense.view_move_form_inherit_expense` | xpath | `account.view_move_form` | `nb_expenses` | `action_open_expense` |  | `hr_expense` |
| `hr_expense.view_move_list_expense` | list |  | `made_sequence_gap`, `name`, `partner_id`, `date`, `invoice_date_due`, `invoice_origin`, `payment_reference`, `ref`, `activity_ids`, `company_id`, `company_id`, `amount_untaxed_in_currency_signed`, `amount_tax_signed`, `amount_total_in_currency_signed`, `amount_total_in_currency_signed`, `amount_residual_signed`, `currency_id`, `company_currency_id`, `checked`, `status_in_payment` |  |  | `hr_expense` |
| `l10n_ae.view_move_form` | xpath | `account.view_move_form` | `l10n_gcc_invoice_tax_amount` |  |  | `l10n_ae` |
| `l10n_ar.view_account_move_filter` | field | `account.view_account_move_filter` | `partner_id`, `l10n_ar_afip_responsibility_type_id` |  |  | `l10n_ar` |
| `l10n_ar.view_move_form` | group | `account.view_move_form` | `l10n_ar_afip_concept`, `l10n_ar_afip_service_start`, `l10n_ar_afip_service_end` |  |  | `l10n_ar` |
| `l10n_bg_ledger.l10n_bg_move_view_form` | xpath | `account.view_move_form` | `l10n_bg_document_type`, `l10n_bg_document_number`, `l10n_bg_exemption_reason` |  |  | `l10n_bg_ledger` |
| `l10n_bg_ledger.l10n_bg_move_view_tree` | xpath | `account.view_invoice_tree` | `l10n_bg_document_number`, `l10n_bg_document_type` |  |  | `l10n_bg_ledger` |
| `l10n_bg_ledger.l10n_bg_move_view_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Document Type` | `l10n_bg_ledger` |
| `l10n_cl.view_move_form_inherit_l10n_cl` | form | `account.view_move_form` | `l10n_latam_internal_type` |  |  | `l10n_cl` |
| `l10n_cl.view_latam_form_inherit_l10n_cl` | field | `l10n_latam_invoice_document.view_move_form` | `l10n_latam_document_number` |  |  | `l10n_cl` |
| `l10n_cl.view_complete_invoice_refund_tree` | list |  | `l10n_latam_document_type_id_code`, `l10n_latam_document_number`, `partner_id_vat`, `partner_id`, `invoice_date`, `invoice_date_due`, `date`, `payment_reference`, `invoice_user_id`, `company_id`, `invoice_origin`, `amount_untaxed_signed`, `amount_tax_signed`, `amount_total_signed`, `amount_residual_signed`, `currency_id`, `company_currency_id`, `state`, `payment_state`, `move_type` |  |  | `l10n_cl` |
| `l10n_cn.view_invoice_tree_inherit_i10n_cn` | xpath | `account.view_invoice_tree` | `fapiao` |  |  | `l10n_cn` |
| `l10n_cn.account_move_form_l10n_cn` | xpath | `account.view_move_form` | `move_type`, `fapiao` |  |  | `l10n_cn` |
| `l10n_dk_nemhandel.l10n_dk_nemhandel_view_move_form` | header | `account.view_move_form` |  | `Cancel Nemhandel` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_out_invoice_tree_inherit` | field | `account.view_out_invoice_tree` | `status_in_payment`, `nemhandel_move_state` |  |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_out_credit_note_tree_inherit` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `nemhandel_move_state` |  |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel.nemhandel_view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Nemhandel status` | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel_response.nemhandel_response_view_move_form` | xpath | `l10n_dk_nemhandel.l10n_dk_nemhandel_view_move_form` | `nemhandel_move_state` |  |  | `l10n_dk_nemhandel_response` |
| `l10n_eg_edi_eta.view_move_form_inherit` | xpath | `account.view_move_form` |  | `Sign Invoice` |  | `l10n_eg_edi_eta` |
| `l10n_es.view_move_form_inherit` | xpath | `account.view_move_form` | `l10n_es_is_simplified` |  |  | `l10n_es` |
| `l10n_es_edi_facturae.view_move_form` | xpath | `account.view_move_form` | `l10n_es_invoicing_period_start_date`, `l10n_es_invoicing_period_end_date`, `l10n_es_payment_means` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_sii.view_move_form_inherit_l10n_es_edi` | xpath | `account.view_move_form` | `l10n_es_registration_date`, `l10n_es_edi_csv` |  |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.view_move_form_inherit_l10n_es_edi_tbai` | xpath | `account.view_move_form` | `l10n_es_tbai_is_required`, `l10n_es_tbai_post_document_id` | `Send Bill to TicketBAI`, `Resend to TicketBAI`, `TicketBAI Cancel` |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_account_move_filter` | xpath | `account.view_account_move_filter` |  |  | `Veri*Factu State` | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Veri*Factu State` | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_move_tree` | field | `account.view_move_tree` | `state`, `l10n_es_edi_verifactu_state` |  |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_invoice_tree` | field | `account.view_invoice_tree` | `status_in_payment`, `l10n_es_edi_verifactu_state` |  |  | `l10n_es_edi_verifactu` |
| `l10n_es_edi_verifactu.view_move_form_inherit_l10n_es_edi_verifactu` | xpath | `account.view_move_form` | `l10n_es_edi_verifactu_warning`, `l10n_es_edi_verifactu_warning`, `l10n_es_edi_verifactu_warning` |  |  | `l10n_es_edi_verifactu` |
| `l10n_fr_facturx_chorus_pro.view_move_form_inherit_chorus_pro` | xpath | `account.view_move_form` | `buyer_reference`, `contract_reference`, `purchase_order_reference` |  |  | `l10n_fr_facturx_chorus_pro` |
| `l10n_fr_pdp.l10n_fr_pdp_view_out_invoice_tree` | field | `account_peppol.account_peppol_view_out_invoice_tree_inherit` | `peppol_move_state`, `pdp_ppf_move_state` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_out_credit_note_tree` | field | `account_peppol.account_peppol_view_out_credit_note_tree_inherit` | `peppol_move_state`, `pdp_ppf_move_state` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_in_invoice_refund_tree_inherit` | field | `account.view_in_invoice_refund_tree` | `status_in_payment`, `peppol_move_state` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_in_bill_tree_inherit` | field | `account.view_in_invoice_bill_tree` | `status_in_payment`, `peppol_move_state` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_move_form` | button | `account_peppol.account_peppol_view_move_form` |  | `button_cancel`, `Cancel` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_view_account_invoice_filter` | filter | `account_peppol.account_peppol_view_account_invoice_filter` |  |  | `peppol_ready` | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_out_invoice_tree` | field | `account.view_out_invoice_tree` | `status_in_payment`, `l10n_fr_pdp_status` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_out_credit_note_tree` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `l10n_fr_pdp_status` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_form` | button | `account.view_move_form` |  | `button_cancel`, `Cancel` |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_in_invoice_tree` | field | `account.view_in_invoice_tree` | `status_in_payment`, `l10n_fr_pdp_status` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_kanban` | xpath | `account.view_account_move_kanban` | `l10n_fr_pdp_status` |  |  | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_view_move_search` | xpath | `account.view_account_invoice_filter` |  |  | `PDP Pending`, `PDP Ready`, `PDP Error`, `PDP Sent`, `PDP Out Of Scope`, `Group By PDP Status`, `Group By Invoice Month`, `Group By Last PDP Flow` | `l10n_fr_pdp` |
| `l10n_fr_pdp.l10n_fr_pdp_list_view_move_ereporting` | list |  | `name`, `partner_id`, `invoice_date`, `date`, `amount_untaxed_in_currency_signed`, `amount_total_in_currency_signed`, `company_id`, `currency_id`, `company_currency_id`, `status_in_payment`, `l10n_fr_pdp_status` |  |  | `l10n_fr_pdp` |
| `l10n_gr_edi.account_move_form_inherit_l10n_gr_edi` | header | `account.view_move_form` |  | `Send Expense Classifications to myDATA` |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_out_invoice_tree_inherit_l10n_gr_edi` | field | `account.view_out_invoice_tree` | `status_in_payment`, `l10n_gr_edi_state` |  |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_out_credit_note_tree_inherit_l10n_gr_edi` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `l10n_gr_edi_state` |  |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_in_invoice_bill_tree_inherit_l10n_gr_edi` | field | `account.view_in_invoice_tree` | `status_in_payment`, `l10n_gr_edi_state` |  |  | `l10n_gr_edi` |
| `l10n_gr_edi.view_account_invoice_filter_inherit_l10n_gr_edi` | xpath | `account.view_account_invoice_filter` | `l10n_gr_edi_state` |  |  | `l10n_gr_edi` |
| `l10n_gr_edi_e_invoo.account_move_form_inherit_l10n_gr_edi_e_invoo` | xpath | `l10n_gr_edi.account_move_form_inherit_l10n_gr_edi` |  |  |  | `l10n_gr_edi_e_invoo` |
| `l10n_hr_edi.account_move_form_inherit` | xpath | `account.view_move_form` |  | `MER: Reject eRacun` |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_invoice_tree_inherit` | xpath | `account.view_invoice_tree` |  | `MER: Report payments` |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_account_invoice_filter_inherit` | xpath | `account.view_account_invoice_filter` |  |  | `Is a MER document`, `Has unreported payments` | `l10n_hr_edi` |
| `l10n_hu_edi.view_invoice_tree_inherit_l10n_hu_edi` | xpath | `account.view_invoice_tree` | `l10n_hu_edi_state` |  |  | `l10n_hu_edi` |
| `l10n_hu_edi.view_move_form_inherit_l10n_hu_edi` | xpath | `account.view_move_form` |  | `Update Status` |  | `l10n_hu_edi` |
| `l10n_hu_edi_receive.l10n_hu_edi_receive_view_in_invoice_bill_tree` | list | `account.view_in_invoice_bill_tree` |  |  |  | `l10n_hu_edi_receive` |
| `l10n_id_efaktur_coretax.account_move_efaktur_form_view` | xpath | `account.view_move_form` | `l10n_id_kode_transaksi`, `l10n_id_coretax_document`, `l10n_id_coretax_efaktur_available`, `l10n_id_coretax_add_info_07`, `l10n_id_coretax_facility_info_07`, `l10n_id_coretax_add_info_08`, `l10n_id_coretax_facility_info_08`, `l10n_id_coretax_custom_doc`, `l10n_id_coretax_custom_doc_month_year` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_id_efaktur_coretax.view_account_invoice_filter` | field | `account.view_account_invoice_filter` | `name`, `l10n_id_coretax_document` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_in.invoice_form_inherit_l10n_in` | xpath | `account.view_move_form` | `l10n_in_warning` |  |  | `l10n_in` |
| `l10n_in_edi.invoice_form_inherit_l10n_in_edi` | xpath | `account.view_move_form` | `l10n_in_edi_cancel_reason`, `l10n_in_edi_cancel_remarks` |  |  | `l10n_in_edi` |
| `l10n_in_edi.view_out_invoice_tree_inherit_l10n_in_edi` | field | `account.view_out_invoice_tree` | `status_in_payment`, `l10n_in_edi_status` |  |  | `l10n_in_edi` |
| `l10n_in_edi.view_out_credit_note_tree_inherit_l10n_in_edi` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `l10n_in_edi_status` |  |  | `l10n_in_edi` |
| `l10n_in_edi.l10n_in_edi_inherit_account_move_search_view` | xpath | `account.view_account_invoice_filter` |  |  | `Indian E-Invoices To Send`, `Indian E-Invoices In Error` | `l10n_in_edi` |
| `l10n_in_ewaybill.invoice_form_inherit_l10n_in_ewaybill` | xpath | `account.view_move_form` |  | `Create e-Waybill` |  | `l10n_in_ewaybill` |
| `l10n_in_ewaybill.view_invoice_list_inherit_l10n_in_ewaybill` | xpath | `account.view_invoice_tree` | `l10n_in_ewaybill_name` |  |  | `l10n_in_ewaybill` |
| `l10n_it_edi.view_invoice_tree_inherit` | field | `account.view_invoice_tree` | `status_in_payment`, `l10n_it_edi_transaction`, `l10n_it_edi_attachment_name`, `l10n_it_edi_state` |  |  | `l10n_it_edi` |
| `l10n_it_edi.view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` | `l10n_it_edi_transaction`, `l10n_it_edi_attachment_name`, `l10n_it_edi_state` |  |  | `l10n_it_edi` |
| `l10n_it_edi.account_invoice_form_l10n_it` | data | `account.view_move_form` | `l10n_it_edi_is_self_invoice`, `l10n_it_edi_header`, `l10n_it_edi_button_label`, `l10n_it_edi_attachment_name`, `l10n_it_edi_transaction`, `l10n_it_edi_attachment_file`, `l10n_it_stamp_duty`, `l10n_it_ddt_id`, `l10n_it_document_type`, `l10n_it_payment_method`, `l10n_it_partner_pa`, `l10n_it_origin_document_type`, `l10n_it_origin_document_name`, `l10n_it_origin_document_date`, `l10n_it_cig`, `l10n_it_cup`, `l10n_it_edi_state` | `Send to SDI`, `Check Sending` |  | `l10n_it_edi` |
| `l10n_it_edi_doi.view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` | `l10n_it_edi_doi_id` |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_move_tree` | list |  | `made_sequence_gap`, `name`, `invoice_partner_display_name`, `invoice_date`, `date`, `currency_id`, `state`, `l10n_it_edi_doi_amount` |  |  | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_move_form` | div | `account.view_move_form` |  | `action_open_declaration_of_intent` |  | `l10n_it_edi_doi` |
| `l10n_it_stock_ddt.account_invoice_view_form_inherit_ddt` | xpath | `account.view_move_form` | `l10n_it_ddt_count` | `get_linked_ddts` |  | `l10n_it_stock_ddt` |
| `l10n_jo_edi.view_move_form` | xpath | `account.view_move_form` | `l10n_jo_edi_invoice_type` |  |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_out_invoice_tree` | field | `account.view_out_invoice_tree` | `payment_reference`, `l10n_jo_edi_invoice_type`, `l10n_jo_edi_state`, `l10n_jo_edi_error` |  |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_out_credit_note_tree` | field | `account.view_out_credit_note_tree` | `payment_reference`, `l10n_jo_edi_invoice_type`, `l10n_jo_edi_state`, `l10n_jo_edi_error` |  |  | `l10n_jo_edi` |
| `l10n_jo_edi.view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `l10n_jo_edi_state`, `JoFotara Invoice Type` | `l10n_jo_edi` |
| `l10n_ke.view_move_form` | xpath | `account.view_move_form` | `l10n_ke_wh_certificate_number`, `l10n_ke_wh_certificate_date` |  |  | `l10n_ke` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_form` | xpath | `account.view_move_form` | `l10n_ke_cu_qrcode`, `l10n_ke_cu_show_send_button` | `Send To Fiscal Device` |  | `l10n_ke_edi_tremol` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_tree_view` | field | `account.view_out_invoice_tree` | `status_in_payment`, `l10n_ke_cu_invoice_number` |  |  | `l10n_ke_edi_tremol` |
| `l10n_ke_edi_tremol.l10n_ke_inherit_account_move_search_view` | xpath | `account.view_account_invoice_filter` | `l10n_ke_cu_invoice_number` |  |  | `l10n_ke_edi_tremol` |
| `l10n_latam_invoice_document.view_account_invoice_filter` | field | `account.view_account_invoice_filter` | `partner_id`, `l10n_latam_document_type_id` |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_account_move_filter` | field | `account.view_account_move_filter` | `partner_id`, `l10n_latam_document_type_id` |  |  | `l10n_latam_invoice_document` |
| `l10n_latam_invoice_document.view_move_form` | form | `account.view_move_form` | `l10n_latam_available_document_type_ids`, `l10n_latam_use_documents`, `l10n_latam_manual_document_number` |  |  | `l10n_latam_invoice_document` |
| `l10n_my_edi.view_move_form_inherit_l10n_my_myinvois` | xpath | `account.view_move_form` | `l10n_my_edi_document_ids` | `action_show_myinvois_documents` |  | `l10n_my_edi` |
| `l10n_my_edi.view_invoice_list_inherit_l10n_my_myinvois` | field | `account.view_invoice_tree` | `status_in_payment`, `l10n_my_edi_state` |  |  | `l10n_my_edi` |
| `l10n_pl.view_move_form_l10n_pl` | xpath | `account.view_move_form` |  |  |  | `l10n_pl` |
| `l10n_pl_edi.view_move_form_l10n_pl_edi` | xpath | `account.view_move_form` |  | `Check Sending`, `Download UPO` |  | `l10n_pl_edi` |
| `l10n_ro_edi.account_move_form_inherit_l10n_ro_edi` | xpath | `account.view_move_form` | `l10n_ro_edi_state`, `l10n_ro_edi_index` |  |  | `l10n_ro_edi` |
| `l10n_ro_edi.out_invoice_tree_inherit_l10n_ro_edi` | field | `account.view_out_invoice_tree` | `status_in_payment`, `l10n_ro_edi_state` |  |  | `l10n_ro_edi` |
| `l10n_ro_edi.out_credit_note_tree_inherit_l10n_ro_edi` | field | `account.view_out_credit_note_tree` | `status_in_payment`, `l10n_ro_edi_state` |  |  | `l10n_ro_edi` |
| `l10n_ro_edi.in_invoice_tree_inherit_l10n_ro_edi` | field | `account.view_in_invoice_tree` | `status_in_payment`, `l10n_ro_edi_state` |  |  | `l10n_ro_edi` |
| `l10n_ro_edi.l10n_ro_edi_view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` | `l10n_ro_edi_state` |  |  | `l10n_ro_edi` |
| `l10n_rs.view_move_form_inherit` | xpath | `account.view_move_form` | `l10n_rs_turnover_date` |  |  | `l10n_rs` |
| `l10n_rs_edi.view_move_form` | xpath | `account.view_move_form` | `l10n_rs_tax_date_obligations_code` |  |  | `l10n_rs_edi` |
| `l10n_sa.view_move_form_inherit_l10n_sa` | group | `account.view_move_form` | `l10n_sa_reason` |  |  | `l10n_sa` |
| `l10n_sa_edi.view_account_form_inherit` | xpath | `account.view_move_form` |  |  |  | `l10n_sa_edi` |
| `l10n_sg.view_invoice_form_l10n_sg` | xpath | `account.view_move_form` | `l10n_sg_permit_number`, `l10n_sg_permit_number_date` |  |  | `l10n_sg` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_move_form` | xpath | `account.view_move_form` |  |  |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_account_invoice_filter` | xpath | `account.view_account_invoice_filter` |  |  | `Nilvera Status` | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice.account_nilvera_view_invoice_tree` | button | `account.view_invoice_tree` |  | `action_force_register_payment`, `Fetch Nilvera PDF` |  | `l10n_tr_nilvera_einvoice` |
| `l10n_tr_nilvera_einvoice_extended.account_move_form_view_l10n_tr_nilvera_extended` | xpath | `account.view_move_form` |  |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.view_move_form_inherit_ecpay` | xpath | `account.view_move_form` | `l10n_tw_edi_state`, `l10n_tw_edi_ecpay_invoice_id`, `l10n_tw_edi_invoice_create_date`, `l10n_tw_edi_invalidate_reason`, `l10n_tw_edi_is_print`, `l10n_tw_edi_love_code`, `l10n_tw_edi_carrier_type`, `l10n_tw_edi_carrier_number`, `l10n_tw_edi_carrier_number_2`, `l10n_tw_edi_clearance_mark`, `l10n_tw_edi_zero_tax_rate_reason`, `l10n_tw_edi_refund_invoice_number`, `l10n_tw_edi_refund_state`, `l10n_tw_edi_refund_agreement_type`, `l10n_tw_edi_allowance_notify_way` |  |  | `l10n_tw_edi_ecpay` |
| `l10n_vn.view_invoice_form_inherit_l10n_vn` | xpath | `account.view_move_form` | `l10n_vn_e_invoice_number` |  |  | `l10n_vn` |
| `l10n_vn_edi_viettel.view_invoice_form_inherit_l10n_vn_edi` | xpath | `account.view_move_form` |  | `Send Payment Status` |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.view_account_invoice_filter_inherit_l10n_vn_edi` | filter | `account.view_account_invoice_filter` | `l10n_vn_edi_invoice_state` |  | `late`, `Need payment state update` | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel.view_invoice_tree_inherit_l10n_vn_edi` | field | `account.view_invoice_tree` | `status_in_payment`, `l10n_vn_edi_invoice_state` |  |  | `l10n_vn_edi_viettel` |
| `mrp_account.view_move_form_inherit_mrp_account` | xpath | `account.view_move_form` | `wip_production_count` | `action_view_wip_production` |  | `mrp_account` |
| `point_of_sale.view_account_journal_pos_user_form` | xpath | `account.view_move_form` | `pos_order_count` | `action_view_source_pos_orders` |  | `point_of_sale` |
| `purchase.view_move_form_inherit_purchase` | field | `account.view_move_form` | `invoice_vendor_bill_id`, `purchase_id`, `purchase_vendor_bill_id` |  |  | `purchase` |
| `sale.account_invoice_groupby_inherit` | field | `account.view_account_invoice_filter` | `invoice_user_id`, `team_id` |  |  | `sale` |
| `sale.account_invoice_view_tree` | field | `account.view_invoice_tree` | `invoice_user_id`, `team_id` |  |  | `sale` |
| `sale.account_invoice_form` | xpath | `account.view_move_form` | `team_id` |  |  | `sale` |
| `sale_timesheet.account_invoice_view_form_inherit_sale_timesheet` | xpath | `account.view_move_form` | `timesheet_count`, `timesheet_total_duration`, `timesheet_encode_uom_id` | `%(sale_timesheet.action_timesheet_from_invoice)d` |  | `sale_timesheet` |
| `stock_landed_costs.account_view_move_form_inherited` | xpath | `account.view_move_form` | `landed_costs_ids` | `Landed Costs` |  | `stock_landed_costs` |
| `website_sale.account_move_view_form` | group | `account.view_move_form` | `website_id` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_move_journal_line` | Journal Entries | list,kanban,form,activity |  | `{'default_move_type': 'entry', 'search_default_posted':1, 'view_no_maturity': True}` |  | `account` |
| `account.action_account_moves_email_preview` | Journal Entries | list,kanban,form,activity | `[('id', 'in', context.get('active_ids'))]` |  |  | `account` |
| `account.action_move_out_invoice_type` | Invoices | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund', 'out_receipt'])]` | `{'default_move_type': 'out_invoice'}` |  | `account` |
| `account.action_move_out_invoice` | Invoices | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund', 'out_receipt'])]` | `{'search_default_out_invoice': 1, 'search_default_out_receipt': 1, 'default_move_type': 'out_invoice'}` |  | `account` |
| `account.action_move_out_refund_type_non_legacy` | Credit Notes | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` | `{'search_default_out_refund': 1, 'default_move_type': 'out_refund', 'display_account_trust': True}` |  | `account` |
| `account.action_move_in_invoice_type` | Bills | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund', 'in_receipt'])]` | `{'default_move_type': 'in_invoice', 'display_account_trust': True}` |  | `account` |
| `account.action_move_in_invoice` | Bills | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund', 'in_receipt'])]` | `{'search_default_in_invoice': 1, 'search_default_in_receipt': 1, 'default_move_type': 'in_invoice', 'display_account_trust': True}` |  | `account` |
| `account.action_move_in_refund_type` | Refunds | list,kanban,form,activity | `[('move_type', 'in', ['in_invoice', 'in_refund'])]` | `{'search_default_in_refund': 1, 'default_move_type': 'in_refund'}` |  | `account` |
| `account.action_move_line_form` | Entries |  |  | `{'default_move_type': 'entry'}` |  | `account` |
| `account.action_move_out_refund_type` | Credit Notes | list,kanban,form,activity | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` | `{'search_default_out_refund': 1, 'default_move_type': 'out_refund', 'display_account_trust': True}` |  | `account` |
| `account.res_partner_action_supplier_bills` | Vendor Bills | list,form,graph | `[('move_type','in',('in_invoice', 'in_refund'))]` | `{'search_default_partner_id': active_id, 'default_move_type': 'in_invoice', 'default_partner_id': active_id}` |  | `account` |
| `l10n_cl.sale_invoices_credit_notes` | Sale Invoices and Credit Notes | list,form | `[('move_type', 'in', ['out_invoice', 'out_refund'])]` | `{'default_move_type': 'out_invoice'}` | current | `l10n_cl` |
| `l10n_cl.vendor_bills_and_refunds` | Vendor Bills and Refunds | list,form | `[('move_type', 'in', ['in_invoice', 'in_refund'])]` | `{'default_move_type': 'in_invoice'}` | current | `l10n_cl` |
| `l10n_fr_pdp.l10n_fr_pdp_reports_action_error_moves` | E-Reporting Attention Needed | list,form | `[('l10n_fr_pdp_status', '=', 'error')]` | `{'search_default_group_by_move_type': 1, 'group_by': ['l10n_fr_pdp_last_flow_id']}` |  | `l10n_fr_pdp` |
| `purchase.act_res_partner_2_supplier_invoices` | Vendor Bills | list,form,graph | `[('move_type','in',('in_invoice', 'in_refund'))]` | `{'search_default_partner_id': active_id, 'default_move_type': 'in_invoice', 'default_partner_id': active_id}` |  | `purchase` |
| `sale.action_invoice_salesteams` | Invoices | list,form,kanban | `[             ('state', '=', 'posted'),             ('move_type', 'in', ['out_invoice', 'out_refund'])]` | `{                 'search_default_team_id': [active_id],                 'default_team_id': active_id,                 'default_move_type':'out_invoice',                 'move_type':'out_invoice',                 'journal_type': 'sale',             }` |  | `sale` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account.model_account_move_action_share` | Share | code |  | yes |
| `account.action_move_switch_move_type` | Switch into invoice/credit note | code |  | yes |
| `account.action_move_force_register_payment` | Pay | code |  | yes |
| `account.action_move_block_payment` | (Un)Block Payment | code |  | yes |
| `account.accountant_confirm_entries_action` | Review Entries | code |  | yes |
| `account.action_validate_account_moves` | Confirm Entries | code |  | yes |
| `account_edi_ubl_cii.action_group_ungroup_lines_by_tax` | (Un)Group lines by tax | code |  | yes |
| `l10n_eg_edi_eta.action_sign_invoices` | Sign invoices | code |  | yes |
| `l10n_fr_pdp.l10n_fr_pdp_action_open_response_wizard` | Send Response | code |  | yes |
| `l10n_gr_edi.l10n_gr_edi_action_try_send_batch` | Send to myDATA | code |  | yes |
| `l10n_id.action_fetch_qris_status` | Check QRIS Payment Status | code |  | yes |
| `l10n_id_efaktur_coretax.dowload_efaktur_action` | Download e-Faktur | code |  | yes |
| `l10n_ke_edi_tremol.action_send_invoices_to_device` | Send to fiscal device | code |  | yes |
| `l10n_my_edi.invoice_send_to_myinvois` | Send To MyInvois | code |  | yes |
| `l10n_ph.action_account_move_bir_2307` | Download BIR 2307 XLS | code |  | yes |
| `l10n_ro_edi.l10n_ro_edi_action_fetch_ciusro_status` | Fetch E-Factura Status | code |  | yes |
| `l10n_tw_edi_ecpay.action_print_ecpay_invoice` | Print Ecpay invoice | code |  | yes |
| `l10n_vn_edi_viettel.l10n_vn_edi_send_invoice_payment_status` | Send payment status to SInvoice | code |  | yes |
| `l10n_vn_edi_viettel_pos.l10n_vn_edi_pos_fetch_tax_invoice` | Fetch Tax Invoice | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `account.account_invoices` | Invoice PDF | qweb-pdf | `account.report_invoice_with_payments` | `(object._get_report_base_filename())` |  |
| `account.action_account_original_vendor_bill` | Original Bills | qweb-pdf | `account.report_original_vendor_bill` |  | `'original_vendor_bill.pdf'` |
| `account.account_invoices_without_payment` | PDF without Payment | qweb-pdf | `account.report_invoice` | `(object._get_report_base_filename())` |  |
| `account_edi_ubl_cii.action_report_account_invoices_generated_by_system` | Invoice report generated by the system | qweb-pdf | `account_edi_ubl_cii.account_invoices_generated_by_system` |  |  |
| `l10n_ch.l10n_ch_qr_report` | QR-bill | qweb-pdf | `l10n_ch.qr_report_main` | `'QR-bill-%s' % object.name` |  |
| `l10n_cn.account_voucher_cn` | Voucher | qweb-pdf | `l10n_cn.report_voucher` | `'Voucher_%s' % (object.name)` |  |
| `l10n_in.l10n_in_account_invoices_duplicate` | Duplicate (2 Copies) | qweb-pdf | `l10n_in.l10n_in_invoice_document_duplicate` | `"%s (2 Copies)" % (object._get_report_base_filename())` |  |
| `l10n_in.l10n_in_account_invoices_triplicate` | Triplicate (3 Copies) | qweb-pdf | `l10n_in.l10n_in_invoice_document_triplicate` | `"%s (3 Copies)" % (object._get_report_base_filename())` |  |
| `l10n_lk_invoice.action_report_commercial_invoice` | Commercial Invoice | qweb-pdf | `l10n_lk_invoice.report_commercial_invoice` |  |  |
| `l10n_th.action_report_commercial_invoice` | Commercial Invoice | qweb-pdf | `l10n_th.report_commercial_invoice` |  |  |
| `sale_timesheet.timesheet_report_account_move` | Timesheets | qweb-pdf | `sale_timesheet.report_timesheet_account_move` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `account.ir_cron_auto_post_draft_entry` | Account: Post draft entries with auto_post enabled and accounting date up to today | 1 days | `_autopost_draft_entries` |  |
| `account.ir_cron_account_move_send` | Send invoices automatically | 1 days | `_cron_account_move_send` |  |
| `l10n_gr_edi.ir_cron_mydata_fetch_third_party_invoices` | myDATA: Fetch third-party issued invoice and create draft Vendor Bills | 1 days |  |  |
| `l10n_hu_edi.ir_cron_update_status` | NAV 3.0: Update status of pending invoices | 1 days |  |  |
| `l10n_id.qris_fetch_cron` | QRIS Fetch Status | 1 hours | `_l10n_id_cron_update_payment_status` |  |
| `l10n_it_edi.ir_cron_l10n_it_edi_download_and_update` | IT EDI: Receive invoices from the SdI | 1 days | `cron_l10n_it_edi_download_and_update` |  |
| `l10n_pl_edi.cron_auto_checks_the_polish_invoice_status` | Polish eInvoice: automatically check the status of the invoice in ksef | 1 weeks | `_cron_l10n_pl_edi_check_invoice_status` |  |
| `l10n_pl_edi.cron_l10n_pl_edi_ksef_download_bills` | Polish eInvoice: Download vendor bills from KSeF | 3 hours | `_cron_l10n_pl_edi_download_bills` |  |
| `l10n_ro_edi.ir_cron_l10n_ro_edi_refresh_access_token` | E-Factura: Refresh Access Token | 30 days |  |  |
| `l10n_ro_edi.ir_cron_l10n_ro_edi_synchronize_invoices` | E-Factura: Synchronize with ANAF | 1 days |  |  |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_purchase_documents` | Nilvera: retrieve new purchase documents | 12 hours | `_cron_nilvera_get_new_einvoice_purchase_documents` |  |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_einvoice_sale_documents` | Nilvera: retrieve E-Invoice new sale documents | 12 hours | `_cron_nilvera_get_new_einvoice_sale_documents` |  |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_new_earchive_sale_documents` | Nilvera: retrieve new E-Archive sale documents | 12 hours | `_cron_nilvera_get_new_earchive_sale_documents` |  |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_invoice_status` | Nilvera: retrieve invoice status | 12 hours | `_cron_nilvera_get_invoice_status` |  |
| `l10n_tr_nilvera_einvoice.ir_cron_nilvera_get_sale_pdf` | Nilvera: retrieve sale PDFs | 12 hours | `_cron_nilvera_get_sale_pdf` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `account.email_template_edi_invoice` | Invoice: Sending | {{ object.company_id.name }} Invoice (Ref {{ object.name or 'n/a' }}) |
| `account.email_template_edi_credit_note` | Credit Note: Sending | {{ object.company_id.name }} Credit Note (Ref {{ object.name or 'n/a' }}) |
| `account.email_template_edi_self_billing_invoice` | Self-billing invoice: Sending | {{ object.company_id.name }} Self-billing invoice (Ref {{ object.name or 'n/a' }}) |
| `account.email_template_edi_self_billing_credit_note` | Self-billing credit note: Sending | {{ object.company_id.name }} Self-billing credit note (Ref {{ object.name or 'n/a' }}) |
| `account.mail_template_invoice_subscriber` | Journal Notification | {{ object.company_id.name }} - New invoice in {{ object.journal_id.display_name or 'Invoices' }} journal |

Machine-readable definition: `../../../schemas/data/entities/account.move.json`; views: `../../../schemas/interfaces/views/account.move.json`.
