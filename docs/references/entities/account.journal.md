# Journal (`account.journal`)

**Transport name:** `account.journal`  
**Storage name:** `account_journal`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_check_printing`, `account_debit_note`, `account_edi`, `account_payment`, `account_peppol`, `point_of_sale`, `l10n_latam_invoice_document`, `l10n_ar`, `l10n_latam_check`, `l10n_at`, `l10n_be`, `l10n_bg_ledger`, `l10n_br`, `l10n_ch`, `l10n_de`, `l10n_dk`, `l10n_dk_fik`, `l10n_dk_nemhandel`, `l10n_ec`, `l10n_eg_edi_eta`, `l10n_fi`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_ie`, `l10n_in`, `l10n_lt`, `l10n_ma`, `l10n_nl`, `l10n_no`, `l10n_pl_edi`, `l10n_sa_edi`, `l10n_se`, `l10n_si`, `l10n_tr`, `l10n_tr_nilvera`, `l10n_tr_nilvera_einvoice`

Description: Journal

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `mail.alias.mixin.optional`, `mail.thread`, `mail.activity.mixin`
- Default ordering: `sequence, type, code`
- Display name search fields: `["name", "code"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (107)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `kanban_dashboard` | Kanban Dashboard | multi line text |  | computed by rule `_kanban_dashboard` (not stored) |
| `kanban_dashboard_graph` | Kanban Dashboard Graph | multi line text |  | computed by rule `_kanban_dashboard_graph` (not stored) |
| `json_activity_data` | JavaScript Object Notation Activity Data | multi line text |  | computed by rule `_get_json_activity_data` (not stored) |
| `show_on_dashboard` | Show journal on dashboard | boolean |  | default `True`; Help: Whether this journal should be displayed on the dashboard or not |
| `color` | Color Index | integer |  | default  |
| `current_statement_balance` | Current Statement Balance | monetary |  | computed by rule `_compute_current_statement_balance` (not stored) |
| `has_statement_lines` | Has Statement Lines | boolean |  | computed by rule `_compute_current_statement_balance` (not stored) |
| `entries_count` | Entries Count | integer |  | computed by rule `_compute_entries_count` (not stored) |
| `has_posted_entries` | Has Posted Entries | boolean |  | computed by rule `_compute_has_entries` (not stored) |
| `has_entries` | Has Entries | boolean |  | computed by rule `_compute_has_entries` (not stored) |
| `has_sequence_holes` | Has Sequence Holes | boolean |  | computed by rule `_compute_has_sequence_holes` (not stored) |
| `has_unhashed_entries` | Unhashed Entries | boolean |  | computed by rule `_compute_has_unhashed_entries` (not stored) |
| `last_statement_id` | Last Statement | many to one | `account.bank.statement` | computed by rule `_compute_last_bank_statement` (not stored) |
| `name` | Journal Name | single line text |  | required; translatable |
| `name_placeholder` | Name Placeholder | single line text |  | computed by rule `_compute_name_placeholder` (not stored) |
| `code` | Sequence Prefix | single line text |  | required; computed by rule `_compute_code` and stored; maximum length 5; precomputed before insertion; Help: Shorter name used for display. The journal entries of this journal will also be named using this prefix by default. |
| `active` | Active | boolean |  | default `True`; Help: Set active to false to hide the Journal without removing it. |
| `type` | Type | selection |  | required; Help: Select 'Sale' for customer invoices journals.         Select 'Purchase' for vendor bills journals.         Select 'Cash', 'Bank' or 'Credit Card' for journals that are used in customer or vendor payments.         Select 'General' for miscellaneous operations journals. |
| `is_self_billing` | Self Billing | boolean |  | Help: This journal is for self-billing invoices. Invoices will be created using a different sequence per partner. |
| `default_account_type` | Default Account Type | single line text |  | computed by rule `_compute_default_account_type` (not stored) |
| `default_account_id` | Default Account | many to one | `account.account` | not copied on duplication; on delete of the target: restrict; restricted by domain `_get_default_account_domain`; must belong to the same company |
| `suspense_account_id` | Suspense Account | many to one | `account.account` | computed by rule `_compute_suspense_account_id` and stored; on delete of the target: restrict; restricted by domain `[('account_type', '=', 'asset_current')]`; must belong to the same company; Help: Bank statements transactions will be posted on the suspense account until the final reconciliation allowing finding the right account. |
| `non_deductible_account_id` | Private Share Account | many to one | `account.account` | must belong to the same company; Help: Account used to register the private part of mixed expenses. |
| `restrict_mode_hash_table` | Secure Posted Entries with Hash | boolean |  | Help: If ticked, when an entry is posted, we retroactively hash all moves in the sequence from the entry back to the last hashed entry. The hash can also be performed on demand by the Secure Entries wizard. |
| `sequence` | Sequence | integer |  | default `10`; Help: Used to order Journals in the dashboard view |
| `invoice_reference_type` | Communication Type | selection |  | required; default `invoice`; Help: You can set here the default communication that will appear on customer invoices, once validated, to help the customer to refer to that particular invoice when making the payment. |
| `invoice_reference_model` | Communication Standard | selection |  | required; default computed dynamically (_default_invoice_reference_model); on delete of the target: {"expression": "{'si': lambda recs: recs.write({'invoice_reference_model': 'odoo'})}"}; Help: You can choose different models for each type of reference. The default one is the Odoo reference.; extended by packages `l10n_be`, `l10n_ch`, `l10n_dk_fik`, `l10n_fi`, `l10n_no`, `l10n_se`, `l10n_si` |
| `currency_id` | Currency | many to one | `res.currency` | Help: The currency used to enter statement |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company); indexed; Help: Company related to this journal |
| `country_code` | Country Code | single line text |  | read only; related through path `company_id.account_fiscal_country_id.code` |
| `account_fiscal_country_group_codes` | Account Fiscal Country Group Codes | structured document |  | related through path `company_id.account_fiscal_country_group_codes` |
| `refund_sequence` | Dedicated Credit Note Sequence | boolean |  | computed by rule `_compute_refund_sequence` and stored; Help: Check this box if you don't want to share the same sequence for invoices and credit notes made from this journal |
| `payment_sequence` | Dedicated Payment Sequence | boolean |  | computed by rule `_compute_payment_sequence` and stored; precomputed before insertion; Help: Check this box if you don't want to share the same sequence on payments and bank transactions posted on this journal |
| `invoice_template_pdf_report_id` | Invoice report | many to one | `ir.actions.report` | restricted by domain `[('id', 'in', available_invoice_template_pdf_report_ids)]` |
| `available_invoice_template_pdf_report_ids` | Available Invoice Template Portable Document Format Report | one to many | `ir.actions.report` | computed by rule `_compute_available_invoice_template_pdf_report_ids` (not stored) |
| `display_invoice_template_pdf_report_id` | Display Invoice Template Portable Document Format Report | boolean |  | default computed dynamically (_default_display_invoice_template_pdf_report_id) |
| `sequence_override_regex` | Sequence Override Regex | multi line text |  | Help: Technical field used to enforce complex sequence composition that the system would normally misunderstand. This is a regex that can include all the following capture groups: prefix1, year, prefix2, month, prefix3, seq, suffix. The prefix* groups are the separators between the year, month and the actual increasing sequence number (seq). e.g: ^(?P<prefix1>.*?)(?P<year>\d{4})(?P<prefix2>\D*?)(?P<month>\d{2})(?P<prefix3>\D+?)(?P<seq>\d+)(?P<suffix>\D*?)$ |
| `inbound_payment_method_line_ids` | Inbound Payment Methods | one to many | `account.payment.method.line` | computed by rule `_compute_inbound_payment_method_line_ids` and stored; not copied on duplication; restricted by domain `[["payment_type", "=", "inbound"]]`; must belong to the same company; inverse field `journal_id`; Help: Manual: Get paid by any method outside of Odoo. Payment Providers: Each payment provider has its own Payment Method. Request a transaction on/to a card thanks to a payment token saved by the partner when buying or subscribing online. Batch Deposit: Collect several customer checks at once generating and submitting a batch deposit to your bank. Module account_batch_payment is necessary. SEPA Direct Debit: Get paid in the SEPA zone thanks to a mandate your partner will have granted to you. Module account_sepa is necessary. |
| `outbound_payment_method_line_ids` | Outbound Payment Methods | one to many | `account.payment.method.line` | computed by rule `_compute_outbound_payment_method_line_ids` and stored; not copied on duplication; restricted by domain `[["payment_type", "=", "outbound"]]`; must belong to the same company; inverse field `journal_id`; Help: Manual: Pay by any method outside of Odoo. Check: Pay bills by check and print it from Odoo. SEPA Credit Transfer: Pay in the SEPA zone by submitting a SEPA Credit Transfer file to your bank. Module account_sepa is necessary. |
| `profit_account_id` | Profit Account | many to one | `account.account` | restricted by domain `[('account_type', 'in', ('income', 'income_other'))]`; must belong to the same company; Help: Used to register a profit when the ending balance of a cash register differs from what the system computes |
| `loss_account_id` | Loss Account | many to one | `account.account` | restricted by domain `[('account_type', '=', 'expense')]`; must belong to the same company; Help: Used to register a loss when the ending balance of a cash register differs from what the system computes |
| `company_partner_id` | Account Holder | many to one | `res.partner` | read only; related through path `company_id.partner_id` |
| `bank_account_id` | Bank Account | many to one | `res.partner.bank` | indexed (btree_not_null); not copied on duplication; on delete of the target: restrict; restricted by domain `[('partner_id','=', company_partner_id)]`; must belong to the same company |
| `bank_statements_source` | Bank Feeds | selection |  | default `undefined`; Help: Defines how the bank statements will be registered |
| `bank_acc_number` | Bank Acc Number | single line text |  | related through path `bank_account_id.acc_number` |
| `bank_id` | Bank | many to one | `res.bank` | related through path `bank_account_id.bank_id` |
| `alias_name` | Alias Name | single line text |  | Help: Send one separate email for each invoice. Any file extension will be accepted. Only PDF and XML files will be interpreted by Odoo |
| `journal_group_ids` | Ledger Group | many to many | `account.journal.group` | must belong to the same company |
| `available_payment_method_ids` | Available Payment Method | many to many | `account.payment.method` | computed by rule `_compute_available_payment_method_ids` (not stored) |
| `selected_payment_method_codes` | Selected Payment Method Codes | single line text |  | computed by rule `_compute_selected_payment_method_codes` (not stored) |
| `accounting_date` | Accounting Date | date |  | computed by rule `_compute_accounting_date` (not stored) |
| `display_alias_fields` | Display Alias Fields | boolean |  | computed by rule `_compute_display_alias_fields` (not stored) |
| `has_invalid_statements` | Has Invalid Statements | boolean |  | computed by rule `_compute_has_invalid_statements` (not stored) |
| `show_fetch_in_einvoices_button` | Show E-Invoice Buttons | boolean |  | computed by rule `_compute_show_fetch_in_einvoices_button` (not stored) |
| `show_refresh_out_einvoices_status_button` | Show E-Invoice Status Buttons | boolean |  | computed by rule `_compute_show_refresh_out_einvoices_status_button` (not stored) |
| `incoming_einvoice_notification_email` | Send Copy To | single line text |  | Help: Email addresses that will receive copy for sent and received invoices. Separate entries with ';'. |
| `check_manual_sequencing` | Manual Numbering | boolean |  | default ; Help: Check this option if your pre-printed checks are not numbered. |
| `check_sequence_id` | Check Sequence | many to one | `ir.sequence` | read only; not copied on duplication; Help: Checks numbering sequence. |
| `check_next_number` | Next Check Number | single line text |  | computed by rule `_compute_check_next_number` (not stored); writable through an inverse rule; Help: Sequence number of the next printed check. |
| `bank_check_printing_layout` | Check Layout | selection |  | values provided by rule `_get_check_printing_layouts` |
| `debit_sequence` | Dedicated Debit Note Sequence | boolean |  | computed by rule `_compute_debit_sequence` and stored; Help: Check this box if you don't want to share the same sequence for invoices and debit notes made from this journal |
| `edi_format_ids` | Electronic invoicing | many to many | `account.edi.format` | computed by rule `_compute_edi_format_ids` and stored; restricted by domain `[('id', 'in', compatible_edi_ids)]`; Help: Send XML/EDI invoices |
| `compatible_edi_ids` | Compatible Electronic data interchange | many to many | `account.edi.format` | computed by rule `_compute_compatible_edi_ids` (not stored); Help: EDI format that support moves in this journal |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `is_peppol_journal` | Account used for Peppol | boolean |  | default  |
| `pos_payment_method_ids` | Point of Sale Payment Methods | one to many | `pos.payment.method` | inverse field `journal_id` |
| `l10n_latam_use_documents` | Use Documents? | boolean |  | Help: If active: will be using for legal invoicing (invoices, debit/credit notes). If not set means that will be used to register accounting entries not related to invoicing legal documents. For Example: Receipts, Tax Payments, Register journal entries |
| `l10n_latam_company_use_documents` | Localization Latam Company Use Documents | boolean |  | computed by rule `_compute_l10n_latam_company_use_documents` (not stored) |
| `l10n_ar_afip_pos_system` | ARCA point of sale System | selection |  | computed by rule `_compute_l10n_ar_afip_pos_system` and stored; values provided by rule `_get_l10n_ar_afip_pos_types_selection`; Help: Argentina: Specify which type of system will be used to create the electronic invoice. This will depend on the type of invoice to be created. |
| `l10n_ar_afip_pos_number` | ARCA point of sale Number | integer |  | Help: This is the point of sale number assigned by ARCA in order to generate invoices |
| `company_partner` | Company Partner | many to one | `res.partner` | related through path `company_id.partner_id` |
| `l10n_ar_afip_pos_partner_id` | ARCA point of sale Address | many to one | `res.partner` | restricted by domain `['\|', ('id', '=', company_partner), '&', ('id', 'child_of', company_partner), ('type', '!=', 'contact')]`; Help: This is the address used for invoice reports of this POS |
| `l10n_ar_is_pos` | Is ARCA point of sale? | boolean |  | computed by rule `_compute_l10n_ar_is_pos` and stored; Help: Argentina: Specify if this Journal will be used to send electronic invoices to ARCA. |
| `l10n_bg_customer_invoice` | Customer Invoices | selection |  | default `01`; values provided by rule `_l10n_bg_document_type_selection_values` |
| `l10n_bg_credit_notes` | Credit Notes | selection |  | default `03`; values provided by rule `_l10n_bg_document_type_selection_values` |
| `l10n_bg_debit_notes` | Debit Notes | selection |  | default `02`; values provided by rule `_l10n_bg_document_type_selection_values` |
| `l10n_br_invoice_serial` | Series | single line text |  | not copied on duplication; Help: Brazil: Series number associated with this Journal. If more than one Series needs to be used, duplicate this Journal and assign the new Series to the duplicated Journal. |
| `l10n_dk_fik_creditor_number` | FIK Creditor Number | single line text |  | computed by rule `_compute_l10n_dk_fik_creditor_number` and stored |
| `l10n_dk_nemhandel_proxy_state` | Localization Dk Nemhandel Proxy State | selection |  | related through path `company_id.l10n_dk_nemhandel_proxy_state` |
| `is_nemhandel_journal` | Journal used for Nemhandel | boolean |  |  |
| `l10n_ec_require_emission` | Require Emission | boolean |  | computed by rule `_compute_l10n_ec_require_emission` (not stored); Help: True if an entity and emission point must be set on the journal |
| `l10n_ec_entity` | Emission Entity | single line text |  | not copied on duplication; maximum length 3; Help: Ecuador: Emission entity number that is given by the SRI. |
| `l10n_ec_emission` | Emission Point | single line text |  | not copied on duplication; maximum length 3; Help: Ecuador: Emission point number that is given by the SRI. |
| `l10n_ec_emission_address_id` | Emission address | many to one | `res.partner` | restricted by domain `['\|', ('id', '=', company_partner_id), '&', ('id', 'child_of', company_partner_id), ('type', '!=', 'contact')]`; Help: Ecuador: Address for electronic invoicing. |
| `l10n_eg_branch_id` | Branch | many to one | `res.partner` | not copied on duplication; Help: Address of the subdivision of the company.  You can just put the company partner if this is used for the main branch. |
| `l10n_eg_activity_type_id` | ETA Activity Code | many to one | `l10n_eg_edi.activity.type` | not copied on duplication; Help: This is the activity type of the branch according to Egyptian Tax Authority |
| `l10n_eg_branch_identifier` | ETA Branch identifier | single line text |  | not copied on duplication; Help: This number can be found on the taxpayer profile on the eInvoicing portal. |
| `l10n_hr_business_premises_label` | Business premises label | single line text |  | required; default `1`; maximum length 20; Help: Must contain at least one character and a maximum of 20 numeric (0-9) and/or alphabetic (a-z, A-Z) characters. |
| `l10n_hr_issuing_device_label` | Issuing device label | single line text |  | required; default `1`; Help: Must contain only numeric characters |
| `l10n_hr_business_premises_label_refund` | Business premises label (refund approval) | single line text |  | required; default `1`; maximum length 20; Help: Must contain at least one character and a maximum of 20 numeric (0-9) and/or alphabetic (a-z, A-Z) characters. |
| `l10n_hr_issuing_device_label_refund` | Issuing device label (refund approval) | single line text |  | required; default `2`; Help: Must contain only numeric characters |
| `l10n_hr_mer_connection_state` | Localization Human resources Mer Connection State | selection |  | related through path `company_id.l10n_hr_mer_connection_state` |
| `l10n_hr_is_mer_journal` | Journal used for eRacun via MojEracun | boolean |  | computed by rule `_compute_l10n_hr_is_mer_journal` (not stored) |
| `l10n_sa_csr` | Localization Sa Csr | binary |  | not copied on duplication; visible only to groups `base.group_system`; Help: The Certificate Signing Request that is submitted to the Compliance API |
| `l10n_sa_csr_errors` | Onboarding Errors | rich text |  | not copied on duplication |
| `l10n_sa_compliance_csid_json` | CCSID JavaScript Object Notation | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: Compliance CSID data received from the Compliance CSID API in dumped json format |
| `l10n_sa_production_csid_certificate_id` | PCSID Certificate | many to one | `certificate.certificate` | restricted by domain `[["is_valid", "=", true]]` |
| `l10n_sa_production_csid_json` | PCSID JavaScript Object Notation | single line text |  | not copied on duplication; visible only to groups `base.group_system`; Help: Production CSID data received from the Production CSID API in dumped json format |
| `l10n_sa_production_csid_validity` | Localization Sa Production Csid Validity | date and time |  | related through path `l10n_sa_production_csid_certificate_id.date_end` |
| `l10n_sa_compliance_csid_certificate_id` | CCSID certificate | many to one | `certificate.certificate` | restricted by domain `[["is_valid", "=", true]]` |
| `l10n_sa_compliance_checks_passed` | Compliance Checks Done | boolean |  | default ; not copied on duplication; Help: Specifies if the Compliance Checks have been completed successfully |
| `l10n_sa_chain_sequence_id` | ZATCA account.move chain sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `l10n_sa_latest_submission_hash` | Latest Submission Hash | single line text |  | not copied on duplication; Help: Hash of the latest submitted invoice to be used as the Previous Invoice Hash (KSA-13) |
| `l10n_se_invoice_ocr_length` | optical character recognition Number Length | integer |  | default `6`; Help: Total length of OCR Reference Number including checksum. |
| `l10n_tr_default_sales_return_account_id` | Localization Tr Default Sales Return Account | many to one | `account.account` | computed by rule `_compute_l10n_tr_default_sales_return_account_id` and stored; must belong to the same company |
| `l10n_tr_nilvera_api_key` | Localization Tr Nilvera Application programming interface Key | single line text |  | related through path `company_id.l10n_tr_nilvera_api_key` |
| `is_nilvera_journal` | Journal used for Nilvera | boolean |  |  |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Purchase |
| `cash` | Cash |
| `bank` | Bank |
| `credit` | Credit Card |
| `general` | Miscellaneous |

### `invoice_reference_type` (Communication Type)

| Value | Label |
|---|---|
| `partner` | Based on Customer |
| `invoice` | Based on Invoice |

### `invoice_reference_model` (Communication Standard)

| Value | Label |
|---|---|
| `odoo` | Full Reference (INV/2024/00001) |
| `euro` | European (RF83INV202400001) |
| `number` | Numbers only (202400001) |
| `be` | Belgium (+++000/2024/00182+++) |
| `ch` | Switzerland (12 34560 00103 88500 1000 19188) |
| `dk_fik_71` | Denmark FIK Number (+71) |
| `dk_fik_75` | Denmark FIK Number (+75) |
| `fi` | Finnish Standard Reference (2024000068) |
| `fi_rf` | Finnish Creditor Reference (RF) (RF952024000071) |
| `no` | Norway (000001024000083) |
| `se_ocr2` | Sweden OCR Level 1 & 2 (1255) |
| `se_ocr3` | Sweden OCR Level 3 (12658) |
| `se_ocr4` | Sweden OCR Level 4 (001271) |
| `si` | Slovenian 01 (SI01 25-1235-8403) |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`, `l10n_dk_nemhandel_proxy_state`, `l10n_hr_mer_connection_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_code_company_uniq` | Constraint | `unique (company_id, code)` | Journal codes must be unique per company. | `account` |

## Operations (181)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_current_statement_balance` | computation | self | `account` |  |  |
| `_compute_last_bank_statement` | computation | self | `account` |  |  |
| `_kanban_dashboard` | internal rule | self | `account` |  |  |
| `_kanban_dashboard_graph` | computation | self | `account` | depends: `current_statement_balance` |  |
| `_transform_activity_dict` | internal rule | self, activity_data | `account` |  |  |
| `_get_json_activity_data` | preparation rule | self | `account` |  |  |
| `_query_has_sequence_holes` | internal rule | self | `account` |  |  |
| `_get_moves_to_hash` | preparation rule | self, include_pre_last_hash, early_stop | `account` |  | If we have INV/1, INV/2 not hashed, then INV/3, INV/4 hashed, then INV/5 and INV/6 not hashed :param include_pre_last_hash: if True, this will include INV/1 and INV/2. Otherwise not. :param early_stop: if True, stop searching when we found at least one record :return: |
| `_compute_has_sequence_holes` | computation | self | `account`, `l10n_latam_invoice_document` |  |  |
| `_compute_has_unhashed_entries` | computation | self | `account` |  |  |
| `_compute_has_entries` | computation | self | `account` |  |  |
| `_compute_entries_count` | computation | self | `account` |  |  |
| `_graph_title_and_key` | internal rule | self | `account` |  |  |
| `_get_bank_cash_graph_data` | preparation rule | self | `account` |  | Computes the data used to display the graph for bank and cash journals in the accounting dashboard |
| `_get_sale_purchase_graph_data` | preparation rule | self | `account` |  |  |
| `_get_journal_dashboard_data_batched` | preparation rule | self | `account_check_printing`, `account` |  |  |
| `_fill_dashboard_data_count` | internal rule | self, dashboard_data, model, name, domain | `account` |  | Populate the dashboard data with the result of a count.  :param dashboard_data: a mapping between a journal ids and the data needed to display their                        dashboard kanban card. :type dashboard_data: dict[int, dict] :param model: the model on which to perform the count :type model: str :param name: the name of the variable to inject in the dashboard's data :type name: str :param domain: the domain of records to count |
| `_fill_bank_cash_dashboard_data` | internal rule | self, dashboard_data | `account` |  | Populate all bank and cash journal's data dict with relevant information for the kanban card. |
| `_fill_sale_purchase_dashboard_data` | internal rule | self, dashboard_data | `account` |  | Populate all sale and purchase journal's data dict with relevant information for the kanban card. |
| `_fill_general_dashboard_data` | internal rule | self, dashboard_data | `account` |  | Populate all miscelaneous journal's data dict with relevant information for the kanban card. |
| `_fill_onboarding_data` | internal rule | self, dashboard_data | `account` |  | Populate journals with onboarding data if they have no entries |
| `_get_draft_sales_purchases_query` | preparation rule | self | `account` |  |  |
| `_get_to_pay_select` | preparation rule | self | `account` |  |  |
| `_get_open_sale_purchase_query` | preparation rule | self, journal_type | `account` |  |  |
| `_get_to_check_payment_query` | preparation rule | self | `account` |  |  |
| `_count_results_and_sum_amounts` | internal rule | self, results_dict, target_currency | `account` |  | Loops on a query result to count the total number of invoices and sum their amount_total field (expressed in the given target currency). amount_total must be signed! |
| `_get_journal_dashboard_bank_running_balance` | preparation rule | self | `account` |  |  |
| `_get_direct_bank_payments` | preparation rule | self | `account` |  |  |
| `_get_journal_dashboard_outstanding_payments` | preparation rule | self | `account` |  |  |
| `_get_move_action_context` | preparation rule | self | `account` |  |  |
| `action_create_new` | user action | self | `account` |  |  |
| `_build_no_journal_error_msg` | internal rule | self, company_name, journal_types | `account` |  |  |
| `is_sample_action_available` | operation | self | `account` | model | Used to hide 'try our sample' when demo data is not installed. |
| `action_create_vendor_bill` | user action | self | `account` |  | This function is called by the "try our sample" button of Vendor Bills, visible on dashboard if no bill has been created yet. |
| `to_check_ids` | operation | self | `account` |  |  |
| `_select_action_to_open` | internal rule | self | `account` |  |  |
| `open_action` | operation | self | `account` |  | return action based on type for related journals |
| `open_payments_action` | operation | self, payment_type, mode | `account` |  |  |
| `action_post_all_entries` | user action | self | `account` |  |  |
| `open_action_with_context` | operation | self | `account` |  |  |
| `open_bank_difference_action` | operation | self | `account` |  |  |
| `open_invalid_statements_action` | operation | self | `account` |  |  |
| `_show_sequence_holes` | internal rule | self, domain | `account` |  |  |
| `show_sequence_holes` | operation | self | `account` |  |  |
| `show_unhashed_entries` | operation | self | `account` |  |  |
| `create_bank_statement` | operation | self | `account` |  | return action to create a bank statements. This button should be called only on journals with type =='bank' |
| `create_customer_payment` | operation | self | `account` |  | return action to create a customer payment |
| `create_supplier_payment` | operation | self | `account` |  | return action to create a supplier payment |
| `_default_display_invoice_template_pdf_report_id` | preparation rule | self | `account` |  | Show PDF template selection if there are more than 1 template available for invoices. |
| `_default_inbound_payment_methods` | preparation rule | self | `account` |  |  |
| `_default_outbound_payment_methods` | preparation rule | self | `account_check_printing`, `account`, `l10n_latam_check` |  |  |
| `__get_bank_statements_available_sources` | internal rule | self | `account` |  |  |
| `_get_bank_statements_available_sources` | preparation rule | self | `account` |  |  |
| `_default_invoice_reference_model` | preparation rule | self | `account` |  | Get the invoice reference model according to the company's country. |
| `_get_default_account_domain` | preparation rule | self | `account` |  |  |
| `_compute_has_invalid_statements` | computation | self | `account` |  |  |
| `_compute_display_alias_fields` | computation | self | `account` |  |  |
| `_compute_code` | computation | self | `account` | depends: `type`, `company_id` |  |
| `_get_journals_payment_method_information` | preparation rule | self | `account` |  |  |
| `_compute_available_payment_method_ids` | computation | self | `account` | depends: `outbound_payment_method_line_ids`, `inbound_payment_method_line_ids` | Compute the available payment methods id by respecting the following rules:     Methods of mode 'unique' cannot be used twice on the same company.     Methods of mode 'electronic' cannot be used twice on the same company for the same 'payment_provider_id'.     Methods of mode 'multi' can be duplicated on the same journal. |
| `_compute_default_account_type` | computation | self | `account` | depends: `type` |  |
| `_compute_inbound_payment_method_line_ids` | computation | self | `account`, `l10n_in`, `l10n_ma` | depends: `type`, `currency_id` |  |
| `_compute_outbound_payment_method_line_ids` | computation | self | `account`, `l10n_in`, `l10n_ma` | depends: `type`, `currency_id` |  |
| `_compute_selected_payment_method_codes` | computation | self | `account` | depends: `outbound_payment_method_line_ids`, `inbound_payment_method_line_ids` | Set the selected payment method as a list of comma separated codes like: ,manual,check_printing,... These will be then used to display or not payment method specific fields in the view. |
| `_compute_suspense_account_id` | computation | self | `account` | depends: `company_id`, `type` |  |
| `_compute_accounting_date` | computation | self | `account` | depends: `company_id`; depends_context: `move_date`, `has_tax` |  |
| `_compute_show_fetch_in_einvoices_button` | computation | self | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_pl_edi`, `l10n_tr_nilvera_einvoice` | depends: `type`; depends: `is_peppol_journal`, `account_peppol_proxy_state`; depends: `is_nemhandel_journal`, `l10n_dk_nemhandel_proxy_state`; depends: `type`, `company_id`; depends: `is_nilvera_journal`, `l10n_tr_nilvera_api_key`, `type` |  |
| `_compute_show_refresh_out_einvoices_status_button` | computation | self | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_tr_nilvera_einvoice` | depends: `type`; depends: `account_peppol_proxy_state`; depends: `l10n_dk_nemhandel_proxy_state`; depends: `l10n_tr_nilvera_api_key`, `type` |  |
| `_onchange_type` | on change | self | `account` | onchange: `type` |  |
| `_compute_name_placeholder` | computation | self | `account` | depends: `type` |  |
| `_check_bank_account` | validation | self | `account` | constrains: `type`, `bank_account_id` |  |
| `_check_company_consistency` | validation | self | `account` | constrains: `company_id` |  |
| `_check_type_default_account_id_type` | validation | self | `account` | constrains: `type`, `default_account_id` |  |
| `_check_payment_method_line_ids_multiplicity` | validation | self | `account` | constrains: `inbound_payment_method_line_ids`, `outbound_payment_method_line_ids` | Check and ensure that the payment method lines multiplicity is respected. |
| `_check_auto_post_draft_entries` | validation | self | `account` | constrains: `active` |  |
| `_check_incoming_einvoice_notification_email` | validation | self | `account` | constrains: `type`, `incoming_einvoice_notification_email` |  |
| `_onchange_incoming_einvoice_notification_email` | on change | self | `account` | onchange: `incoming_einvoice_notification_email` |  |
| `_compute_refund_sequence` | computation | self | `account`, `l10n_latam_invoice_document` | depends: `type`; depends: `type`, `l10n_latam_use_documents` |  |
| `_compute_payment_sequence` | computation | self | `account` | depends: `type` |  |
| `_compute_available_invoice_template_pdf_report_ids` | computation | self | `account` |  |  |
| `unlink` | lifecycle override | self | `account` |  |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |
| `write` | lifecycle override | self, vals | `account_edi`, `account`, `l10n_ar` |  |  |
| `_alias_get_creation_values` | internal rule | self | `account` |  |  |
| `_alias_prepare_alias_name` | internal rule | self, alias_name, name, code, jtype, company | `account` | model | Tool method generating standard journal alias, to ensure uniqueness and readability;  reset for other journals than purchase / sale |
| `_ensure_unique_alias` | internal rule | self, vals, company | `account` | model | Check uniqueness of the alias name within the given alias domain. :param vals: the values of the journal. :return: a unique alias name. |
| `_get_next_journal_default_code` | preparation rule | self, journal_type, company, cache, protected_codes | `account` | model |  |
| `_prepare_liquidity_account_vals` | preparation rule | self, company, code, vals | `account`, `l10n_at`, `l10n_de`, `l10n_dk`, `l10n_ie`, `l10n_lt`, `l10n_nl` | model | Set Balance Sheet and SAF-T tags on new bank and cash accounts. |
| `_prepare_credit_account_vals` | preparation rule | self, company, code, vals | `account` | model |  |
| `_create_default_account` | internal rule | self, company, journal_type, vals | `account` | model |  |
| `_fill_missing_values` | internal rule | self, vals, protected_codes | `account` | model |  |
| `create` | lifecycle override | self, vals_list | `account_check_printing`, `account`, `l10n_latam_check` | model_create_multi |  |
| `set_bank_account` | operation | self, acc_number, bank_id | `account` |  | Create a res.partner.bank (if not exists) and set it as value of the field bank_account_id |
| `_assign_outsanding_account_to_payment_method_lines` | internal rule | self, payment_type, payment_method_codes, chart_template | `account` |  | Link bank journal payment method lines to their corresponding outstanding account for the specified chart template.  :param payment_type: Payment direction, either ``'inbound'`` or ``'outbound'``. :param payment_method_codes: Optional list of payment method codes used to restrict     the payment method lines that are updated. :param chart_template: Chart template used to select the relevant bank journals     and determine the outstanding account. |
| `_compute_display_name` | computation | self | `account`, `l10n_br` | depends: `currency_id`; depends: `l10n_br_invoice_serial` |  |
| `action_configure_bank_journal` | user action | self | `account` |  | This function is called by the "configure" button of bank journals, visible on dashboard if no bank statement source has been defined yet |
| `_create_document_from_attachment` | internal rule | self, attachment_ids | `account` |  | Create the invoices from files. |
| `create_document_from_attachment` | operation | self, attachment_ids | `account` |  | Create the invoices from files. :return: A action redirecting to account.move list/form view. |
| `_get_journal_bank_account_balance` | preparation rule | self, domain | `account` |  | Get the bank balance of the current journal by filtering the journal items using the journal's accounts.  /!\ The current journal is not part of the applied domain. This is the expected behavior since we only want a logic based on accounts.  :param domain:  An additional domain to be applied on the account.move.line model. :return:        Tuple having balance expressed in journal's currency                 along with the total number of move lines having the same account as of the journal's default account. |
| `_get_journal_inbound_outstanding_payment_accounts` | preparation rule | self | `account`, `point_of_sale` |  | :return: A recordset with all the account.account used by this journal for inbound transactions. |
| `_get_journal_outbound_outstanding_payment_accounts` | preparation rule | self | `account` |  | :return: A recordset with all the account.account used by this journal for outbound transactions. |
| `_get_available_payment_method_lines` | preparation rule | self, payment_type | `account_payment`, `account` |  | This getter is here to allow filtering the payment method lines if needed in other modules. It does NOT serve as a general getter to get the lines.  For example, it'll be extended to filter out lines from inactive payment providers in the payment module. :param payment_type: either inbound or outbound, used to know which lines to return :return: Either the inbound or outbound payment method lines |
| `_is_payment_method_available` | internal rule | self, payment_method_code, complete_domain | `account` |  | Check if the payment method is available on this journal. |
| `_process_reference_for_sale_order` | background operation | self, order_reference | `account`, `l10n_ch` |  | returns the order reference to be used for the payment. Hook to be overriden: see l10n_ch for an example. |
| `_get_journal_notification_unsubscribe_scope` | preparation rule | self | `account` |  |  |
| `_unsubscribe_invoice_notification_email` | internal rule | self, email_to_remove | `account` |  |  |
| `_notify_einvoices_received` | internal rule | self, moves | `account` |  |  |
| `button_unsubscribe_from_invoice_notifications` | user action | self | `account` |  |  |
| `_notify_invoice_subscribers` | internal rule | self, invoice, mail_params | `account` |  |  |
| `button_fetch_in_einvoices` | user action | self | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_pl_edi`, `l10n_tr_nilvera_einvoice` |  | Abstract method to fetch e-invoices. Should fetch vendor bill invoices synchronously and doesn't return anything. |
| `button_refresh_out_einvoices_status` | user action | self | `account_peppol`, `account`, `l10n_dk_nemhandel`, `l10n_tr_nilvera_einvoice` |  | Abstract method to fetch e-invoice statuses. Should fetch customer invoices statuses synchronously and doesn't return anything. |
| `_get_check_printing_layouts` | preparation rule | self | `account_check_printing` |  | Returns available check printing layouts for the company, excluding disabled options |
| `_compute_check_next_number` | computation | self | `account_check_printing` | depends: `check_manual_sequencing` |  |
| `_inverse_check_next_number` | inverse computation | self | `account_check_printing` |  |  |
| `_create_check_sequence` | internal rule | self | `account_check_printing` |  | Create a check sequence for the journal |
| `action_checks_to_print` | user action | self | `account_check_printing` |  |  |
| `_compute_debit_sequence` | computation | self | `account_debit_note`, `l10n_latam_invoice_document` | depends: `type`; depends: `type`, `l10n_latam_use_documents` |  |
| `_compute_compatible_edi_ids` | computation | self | `account_edi` | depends: `type`, `company_id`, `company_id.account_fiscal_country_id` |  |
| `_compute_edi_format_ids` | computation | self | `account_edi` | depends: `type`, `company_id`, `company_id.account_fiscal_country_id` |  |
| `_unlink_except_linked_to_payment_provider` | internal rule | self | `account_payment` | ondelete |  |
| `_check_type_for_peppol_journal` | validation | self | `account_peppol` | constrains: `type` |  |
| `_check_type` | validation | self | `point_of_sale` | constrains: `type` |  |
| `_check_no_active_payments` | validation | self | `point_of_sale` |  |  |
| `_unlink_journal_except_with_active_payments` | internal rule | self | `point_of_sale` | ondelete |  |
| `_unlink_journal_cascade_pos_payment_methods` | internal rule | self | `point_of_sale` | ondelete |  |
| `action_archive` | lifecycle override | self | `point_of_sale` |  |  |
| `_ensure_company_account_journal` | internal rule | self | `point_of_sale` | model |  |
| `_compute_l10n_latam_company_use_documents` | computation | self | `l10n_latam_invoice_document` | depends: `company_id` |  |
| `_onchange_company` | on change | self | `l10n_latam_invoice_document` | onchange: `company_id`, `type` |  |
| `check_use_document` | validation | self | `l10n_latam_invoice_document` | constrains: `l10n_latam_use_documents` |  |
| `_compute_l10n_ar_is_pos` | computation | self | `l10n_ar` | depends: `country_code`, `type`, `l10n_latam_use_documents` |  |
| `_compute_l10n_ar_afip_pos_system` | computation | self | `l10n_ar` | depends: `l10n_ar_is_pos` |  |
| `_get_l10n_ar_afip_pos_types_selection` | preparation rule | self | `l10n_ar` |  | Return the list of values of the selection field. |
| `_get_journal_letter` | preparation rule | self, counterpart_partner | `l10n_ar` |  | Regarding the ARCA responsibility of the company and the type of journal (sale/purchase), get the allowed letters. Optionally, receive the counterpart partner (customer/supplier) and get the allowed letters to work with him. This method is used to populate document types on journals and also to filter document types on specific invoices to/from customer/supplier |
| `_get_journal_codes_domain` | preparation rule | self | `l10n_ar` |  |  |
| `_get_codes_per_journal_type` | preparation rule | self, afip_pos_system | `l10n_ar` | model |  |
| `_check_afip_pos_system` | validation | self | `l10n_ar` | constrains: `l10n_ar_afip_pos_system` |  |
| `_check_afip_pos_number` | validation | self | `l10n_ar` | constrains: `l10n_ar_afip_pos_number` |  |
| `_onchange_set_short_name` | on change | self | `l10n_ar` | onchange: `l10n_ar_afip_pos_number`, `type` | Will define the ARCA POS Address field domain taking into account the company configured in the journal The short code of the journal only admit 5 characters, so depending on the size of the pos_number (also max 5) we add or not a prefix to identify sales journal. |
| `_get_reusable_payment_methods` | preparation rule | self | `l10n_latam_check` | model | We are able to have multiple times Checks payment method in a journal |
| `_l10n_bg_document_type_selection_values` | internal rule | self | `l10n_bg_ledger` |  |  |
| `_compute_l10n_dk_fik_creditor_number` | computation | self | `l10n_dk_fik` | depends: `invoice_reference_model`, `company_id.bank_ids.acc_number` |  |
| `_check_fik_creditor_number` | validation | self | `l10n_dk_fik` | constrains: `l10n_dk_fik_creditor_number`, `invoice_reference_model` |  |
| `_compute_l10n_ec_require_emission` | computation | self | `l10n_ec` | depends: `type`, `country_code`, `l10n_latam_use_documents` |  |
| `_compute_l10n_hr_is_mer_journal` | computation | self | `l10n_hr_edi` | depends: `company_id.l10n_hr_mer_purchase_journal_id` |  |
| `l10n_hr_mer_get_new_documents` | operation | self | `l10n_hr_edi` |  |  |
| `l10n_hr_mer_get_new_documents_all` | operation | self | `l10n_hr_edi` |  |  |
| `l10n_hr_mer_get_message_status` | operation | self | `l10n_hr_edi` |  |  |
| `_l10n_sa_reset_chain_head_error` | internal rule | self | `l10n_sa_edi` |  | Reset the chain head error from the journal's stuck invoices |
| `_l10n_sa_ready_to_submit_einvoices` | internal rule | self | `l10n_sa_edi` |  | Helper function to know if the required CSIDs have been obtained, and the compliance checks have been completed |
| `_l10n_sa_api_onboard_sanity_checks` | internal rule | self | `l10n_sa_edi` |  | Perform a sanity check to validate that the journal is ready to be onboarded |
| `_l10n_sa_csr_required_fields` | internal rule | self | `l10n_sa_edi` |  | Return the list of fields required to generate a valid CSR as per ZATCA requirements |
| `_l10n_sa_generate_csr` | internal rule | self | `l10n_sa_edi` |  | Generate a CSR for the Journal to be used for the Onboarding process and Invoice submissions |
| `_l10n_sa_get_csid_error` | internal rule | self, csid | `l10n_sa_edi` |  | Return a formatted error string if the CSID response has an 'error' or 'errors' key or doesn't have a 'binarySecurityToken' |
| `_l10n_sa_reset_certificates` | internal rule | self | `l10n_sa_edi` |  | Reset all certificate values, including CSR and compliance checks |
| `_l10n_sa_api_onboard_journal` | internal rule | self, otp | `l10n_sa_edi` |  | Perform the onboarding for the journal. The onboarding consists of three steps:     1.  Get the Compliance CSID     2.  Perform the Compliance Checks     3.  Get the Production CSID |
| `_l10n_sa_get_compliance_CSID` | internal rule | self, otp | `l10n_sa_edi` |  | Request a Compliance Cryptographic Stamp Identifier (CCSID) from ZATCA |
| `_l10n_sa_get_production_CSID` | internal rule | self, OTP | `l10n_sa_edi` |  | Request a Production Cryptographic Stamp Identifier (PCSID) from ZATCA |
| `_l10n_sa_get_compliance_files` | internal rule | self | `l10n_sa_edi` |  | Return the list of files to be used for the compliance checks. |
| `_l10n_sa_run_compliance_checks` | internal rule | self | `l10n_sa_edi` |  | Run Compliance Checks once the CCSID has been obtained.  The goal of the Compliance Checks is to make sure our system is able to produce, sign and send Invoices correctly. For this we use dummy invoice UBL files available under the tests/compliance folder:  Standard Invoice, Standard Credit Note, Standard Debit Note, Simplified Invoice, Simplified Credit Note, Simplified Debit Note.  We read each one of these files separately, sign them, then process them through the Compliance Checks API. |
| `_l10n_sa_prepare_compliance_xml` | internal rule | self, xml_name, xml_raw, certificate, signature | `l10n_sa_edi` |  | Prepare XML content to be used for Compliance checks |
| `_l10n_sa_prepare_invoice_xml` | internal rule | self, xml_content | `l10n_sa_edi` |  | Prepare the XML content of the test invoices before running the compliance checks |
| `_l10n_sa_edi_icv_onboarding` | internal rule | self | `l10n_sa_edi` |  | Onboarding method to create or reset ICV sequence for the journal |
| `_l10n_sa_edi_create_new_chain` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_edi_get_next_chain_index` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_get_last_posted_invoice` | internal rule | self | `l10n_sa_edi` |  | Returns the last invoice posted to this journal's chain. That invoice may have been received by the govt or not (eg. in case of a timeout). Only upon confirmed reception/refusal of that invoice can another one be posted. |
| `_l10n_sa_api_get_compliance_CSID` | internal rule | self, otp | `l10n_sa_edi` |  | API call to the Compliance CSID API to generate a CCSID certificate, password and compliance request_id Requires a CSR token and a One Time Password (OTP) |
| `_l10n_sa_api_get_production_CSID` | internal rule | self, CCSID_data | `l10n_sa_edi` |  | API call to the Production CSID API to generate a PCSID certificate, password and production request_id Requires a requestID from the Compliance CSID API |
| `_l10n_sa_api_renew_production_CSID` | internal rule | self, PCSID_data, OTP | `l10n_sa_edi` |  | API call to the Production CSID API to renew a PCSID certificate, password and production request_id Requires an expired Production CSIDPCSID_data |
| `_l10n_sa_api_compliance_checks` | internal rule | self, xml_content, CCSID_data | `l10n_sa_edi` |  | API call to the COMPLIANCE endpoint to generate a security token used for subsequent API calls Requires a CSR token and a One Time Password (OTP) |
| `_l10n_sa_get_api_clearance_url` | internal rule | self, invoice | `l10n_sa_edi` |  | Return the API to be used for clearance. To be overridden to account for other cases, such as reporting. |
| `_l10n_sa_api_clearance` | internal rule | self, invoice, xml_content, PCSID_data | `l10n_sa_edi` |  | API call to the CLEARANCE/REPORTING endpoint to sign an invoice     - If SIMPLIFIED invoice: Reporting     - If STANDARD invoice: Clearance |
| `_l10n_sa_request_production_csid` | internal rule | self, csid_data, renew, otp | `l10n_sa_edi` |  | Generate company Production CSID data |
| `_l10n_sa_api_get_pcsid` | internal rule | self | `l10n_sa_edi` |  | Get CSIDs required to perform ZATCA api calls, and regenerate them if they need to be regenerated. |
| `_l10n_sa_call_api` | internal rule | self, request_data, request_url, method | `l10n_sa_edi` |  | Helper function to make api calls to the ZATCA API Endpoint |
| `_l10n_sa_api_headers` | internal rule | self | `l10n_sa_edi` |  | Return the base headers to be included in ZATCA API calls |
| `_l10n_sa_authorization_header` | internal rule | self, CSID_data | `l10n_sa_edi` |  | Compute the Authorization header by combining the CSID and the Secret key, then encode to Base64 |
| `_l10n_sa_load_edi_demo_data` | internal rule | self | `l10n_sa_edi` |  |  |
| `_check_l10n_se_invoice_ocr_length` | validation | self | `l10n_se` | constrains: `l10n_se_invoice_ocr_length` |  |
| `_compute_l10n_tr_default_sales_return_account_id` | computation | self | `l10n_tr` | depends: `type`, `company_id.country_code` |  |
| `_check_api_key` | validation | self | `l10n_tr_nilvera_einvoice` |  |  |

## Validation and error messages (47)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_create_vendor_bill` | UserError | self._build_no_journal_error_msg(self.env.company.display_name, ['purchase']) | `account` |
| `action_create_vendor_bill` | UserError | You may only use samples in demo mode, try uploading one of your invoices instead. | `account` |
| `_check_bank_account` | ValidationError | The bank account of a bank journal must belong to the same company (%s). | `account` |
| `_check_bank_account` | ValidationError | The holder of a journal's bank account must be the company (%s). | `account` |
| `_check_company_consistency` | UserError | You can't change the company of your journal since there are some journal entries linked to it. | `account` |
| `_check_type_default_account_id_type` | ValidationError | The type of the journal's default credit/debit account shouldn't be 'receivable' or 'payable'. | `account` |
| `_check_payment_method_line_ids_multiplicity` | ValidationError | Some payment methods supposed to be unique already exists somewhere else. (%s) | `account` |
| `_check_payment_method_line_ids_multiplicity` | ValidationError | You can't have two payment method lines of the same payment type (%(payment_type)s) and with the same name (%(name)s) on a single journal. | `account` |
| `_check_auto_post_draft_entries` | ValidationError | You can not archive a journal containing draft journal entries.  To proceed: 1/ go to Accounting > Accounting > Journal Entries 2/ filter on this journal and on 'Unposted' entries 3/ select them all and post or delete them through the action menu | `account` |
| `copy_data` | UserError | Could not compute any code for the copy automatically. Please create it manually. | `account` |
| `write` | UserError | You cannot modify the field %s of a journal that already has accounting entries. | `account` |
| `write` | UserError | The partners of the journal's company and the related bank account mismatch. | `account` |
| `_fill_missing_values` | UserError | Cannot generate an unused journal code. Please change the name for journal %s. | `account` |
| `_create_document_from_attachment` | UserError | No attachment was provided | `account` |
| `_create_document_from_attachment` | UserError | self.env['account.journal']._build_no_journal_error_msg(self.env.company.display_name, [journal_type]) | `account` |
| `_create_document_from_attachment` | UserError | The journal in which to upload the invoice is not specified. | `account` |
| `_inverse_check_next_number` | ValidationError | Next Check Number should only contains numbers. | `account_check_printing` |
| `_inverse_check_next_number` | ValidationError | The last check number was %s. In order to avoid a check being rejected by the bank, you can only use a greater number. | `account_check_printing` |
| `_inverse_check_next_number` | ValidationError | The check number you entered (%(num)s) exceeds the maximum allowed value of %(max)d. Please enter a smaller number. | `account_check_printing` |
| `write` | UserError | Cannot deactivate (%s) on this journal because not all documents are synchronized | `account_edi` |
| `_unlink_except_linked_to_payment_provider` | UserError | You must first deactivate a payment provider before deleting its journal. Linked providers: %s | `account_payment` |
| `_check_type_for_peppol_journal` | ValidationError | You can't change the type of a journal used for Peppol invoice reception toa type different than 'Purchase'. Please change the journal used for Peppol reception before changing the type of this journal. | `account_peppol` |
| `_check_type` | ValidationError | This journal is associated with a payment method. You cannot modify its type | `point_of_sale` |
| `_check_no_active_payments` | ValidationError | You can not archive this journal because it is set on the following payment method : %s. | `point_of_sale` |
| `check_use_document` | ValidationError | You can not modify the field "Use Documents?" if there are validated invoices in this journal! | `l10n_latam_invoice_document` |
| `_get_journal_letter` | RedirectWarning | msg | `l10n_ar` |
| `_check_afip_pos_system` | ValidationError | '\n'.join((_('The pos system %(system)s can not be used on a purchase journal (id %(id)s)', system=x.l10n_ar_afip_pos_system, id=x.id) for x in journals)) | `l10n_ar` |
| `_check_afip_pos_number` | ValidationError | Please define an ARCA POS number | `l10n_ar` |
| `_check_afip_pos_number` | ValidationError | Please define a valid ARCA POS number (5 digits max) | `l10n_ar` |
| `write` | UserError | You can not change %s journal's configuration if it already has validated invoices | `l10n_ar` |
| `_check_fik_creditor_number` | ValidationError | FIK Creditor Number must be exactly 8 digits. | `l10n_dk_fik` |
| `_l10n_sa_api_onboard_sanity_checks` | UserError | Oops! The journal is stuck. Please submit the pending invoices to ZATCA and try again. | `l10n_sa_edi` |
| `_l10n_sa_generate_csr` | UserError | Please set the following on %(company_name)s: %(fields)s | `l10n_sa_edi` |
| `_l10n_sa_get_compliance_CSID` | UserError | Please check the details below and onboard the journal again: %s | `l10n_sa_edi` |
| `_l10n_sa_get_production_CSID` | UserError | str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_get_production_CSID` | UserError | Could not obtain Production CSID: %s | `l10n_sa_edi` |
| `_l10n_sa_get_production_CSID` | UserError | The Journal is valid until (%s) and can only be renewed upon expiry. | `l10n_sa_edi` |
| `_l10n_sa_run_compliance_checks` | UserError | Please change the (%s)'s country to Saudi Arabia and try again. | `l10n_sa_edi` |
| `_l10n_sa_run_compliance_checks` | UserError | str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_run_compliance_checks` | UserError | Markup("<p class='mb-0'>%s</p>") % str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_run_compliance_checks` | UserError | Markup("<p class='mb-0'>%s</p>") % str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_api_get_compliance_CSID` | UserError | The OTP is invalid. Please try again. | `l10n_sa_edi` |
| `_l10n_sa_api_get_compliance_CSID` | UserError | str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_api_get_pcsid` | UserError | str(ERROR_MESSAGE) | `l10n_sa_edi` |
| `_l10n_sa_api_get_pcsid` | UserError | The Journal is not valid anymore. Please Renew it. | `l10n_sa_edi` |
| `_check_l10n_se_invoice_ocr_length` | ValidationError | OCR Reference Number length need to be greater than 5. Please correct settings under invoice journal settings. | `l10n_se` |
| `_check_api_key` | RedirectWarning | Please configure your Nilvera API key | `l10n_tr_nilvera_einvoice` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `hr_expense.group_hr_expense_team_approver` | no | yes | no | no | `hr_expense` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `group_purchase_manager` | no | yes | no | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `stock.group_stock_manager` | no | yes | yes | no | `sale_stock` |
| `stock.group_stock_manager` | no | yes | no | no | `stock_account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Journal multi-company | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (27)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_account_journal_tree` | list |  | `sequence`, `name`, `type`, `journal_group_ids`, `currency_id`, `code`, `default_account_id`, `active`, `company_id`, `company_id` |  |  | `account` |
| `account.view_account_journal_form` | form |  | `company_id`, `bank_statements_source`, `name_placeholder`, `name`, `active`, `type`, `code`, `company_id`, `country_code`, `default_account_type`, `default_account_id`, `default_account_id`, `suspense_account_id`, `non_deductible_account_id`, `profit_account_id`, `loss_account_id`, `refund_sequence`, `payment_sequence`, `invoice_template_pdf_report_id`, `currency_id`, `company_partner_id`, `bank_account_id`, `bank_id`, `bank_statements_source`, `bank_statements_source`, `available_payment_method_ids`, `inbound_payment_method_line_ids`, `available_payment_method_ids`, `payment_type`, `company_id`, `sequence`, `payment_method_id`, `name`, `payment_account_id`, `outbound_payment_method_line_ids`, `available_payment_method_ids`, `payment_type`, `company_id`, `sequence`, `payment_method_id`, `name`, `payment_account_id`, `selected_payment_method_codes`, `restrict_mode_hash_table`, `is_self_billing`, `display_alias_fields`, `alias_name`, `alias_domain_id`, `incoming_einvoice_notification_email`, `invoice_reference_type`, `invoice_reference_model` | `%(action_account_moves_all_a)d` |  | `account` |
| `account.account_journal_view_kanban` | kanban |  | `name`, `type` |  |  | `account` |
| `account.view_account_journal_search` | search |  | `name`, `activity_user_id`, `activity_type_id` |  | `Favorites`, `Sales`, `Purchases`, `Liquidity`, `Miscellaneous`, `Archived` | `account` |
| `account.account_journal_dashboard_kanban_view` | kanban |  | `id`, `type`, `color`, `kanban_dashboard`, `has_entries`, `has_posted_entries`, `activity_ids`, `activity_state`, `alias_domain_id`, `bank_account_id`, `show_fetch_in_einvoices_button`, `show_refresh_out_einvoices_status_button`, `name`, `company_id`, `alias_id`, `color`, `show_on_dashboard`, `kanban_dashboard_graph` | `action_create_new`, `action_configure_bank_journal`, `open_action`, `action_create_new`, `action_create_new` |  | `account` |
| `account_check_printing.account_journal_dashboard_kanban_view_inherited` | xpath | `account.account_journal_dashboard_kanban_view` |  |  |  | `account_check_printing` |
| `account_check_printing.view_account_journal_form_inherited` | xpath | `account.view_account_journal_form` | `check_sequence_id`, `check_manual_sequencing`, `check_next_number`, `bank_check_printing_layout` |  |  | `account_check_printing` |
| `account_debit_note.view_account_journal_form_inherit_debit_note` | field | `account.view_account_journal_form` | `refund_sequence`, `debit_sequence` |  |  | `account_debit_note` |
| `account_edi.view_account_journal_form_inherited` | xpath | `account.view_account_journal_form` |  |  |  | `account_edi` |
| `account_payment.view_account_journal_form` | xpath | `account.view_account_journal_form` | `code`, `payment_provider_id`, `payment_provider_state` | `SETUP` |  | `account_payment` |
| `l10n_ar.view_account_journal_form` | field | `l10n_latam_invoice_document.view_account_journal_form` | `l10n_latam_use_documents`, `l10n_ar_is_pos`, `company_partner`, `l10n_ar_afip_pos_system`, `l10n_ar_afip_pos_number`, `l10n_ar_afip_pos_partner_id` |  |  | `l10n_ar` |
| `l10n_bg_ledger.l10n_bg_journal_view_form` | xpath | `account.view_account_journal_form` | `l10n_bg_customer_invoice`, `l10n_bg_credit_notes`, `l10n_bg_debit_notes` |  |  | `l10n_bg_ledger` |
| `l10n_br.view_account_journal_form` | field | `l10n_latam_invoice_document.view_account_journal_form` | `type`, `l10n_br_invoice_serial` |  |  | `l10n_br` |
| `l10n_dk.l10n_dk_view_account_journal_form_inherited` | field | `account.view_account_journal_form` | `profit_account_id` |  |  | `l10n_dk` |
| `l10n_dk_fik.view_account_journal_form_l10n_dk_fik` | xpath | `account.view_account_journal_form` | `l10n_dk_fik_creditor_number` |  |  | `l10n_dk_fik` |
| `l10n_dk_nemhandel.account_journal_dashboard_kanban_view` | data | `account.account_journal_dashboard_kanban_view` |  |  |  | `l10n_dk_nemhandel` |
| `l10n_ec.view_account_journal_form` | field | `l10n_latam_invoice_document.view_account_journal_form` | `l10n_latam_use_documents`, `l10n_ec_require_emission`, `l10n_ec_entity`, `l10n_ec_emission`, `l10n_ec_emission_address_id` |  |  | `l10n_ec` |
| `l10n_eg_edi_eta.view_account_journal_form_inherit_l10n_eg_edi` | xpath | `account.view_account_journal_form` | `l10n_eg_branch_id`, `l10n_eg_activity_type_id`, `l10n_eg_branch_identifier` |  |  | `l10n_eg_edi_eta` |
| `l10n_fr_pdp.account_journal_dashboard_kanban_view` | data | `account.account_journal_dashboard_kanban_view` |  |  |  | `l10n_fr_pdp` |
| `l10n_hr_edi.account_journal_dashboard_kanban_view` | xpath | `account.account_journal_dashboard_kanban_view` | `l10n_hr_is_mer_journal`, `l10n_hr_mer_connection_state` |  |  | `l10n_hr_edi` |
| `l10n_hr_edi.view_account_journal_form_inherit` | xpath | `account.view_account_journal_form` | `l10n_hr_business_premises_label`, `l10n_hr_issuing_device_label`, `l10n_hr_business_premises_label_refund`, `l10n_hr_issuing_device_label_refund` |  |  | `l10n_hr_edi` |
| `l10n_in.view_account_journal_form_inherit_l10n_in` | field | `account.view_account_journal_form` | `profit_account_id` |  |  | `l10n_in` |
| `l10n_it_edi.view_account_journal_form_l10n_it` | xpath | `account.view_account_journal_form` | `l10n_it_payment_method` |  |  | `l10n_it_edi` |
| `l10n_latam_invoice_document.view_account_journal_form` | form | `account.view_account_journal_form` | `country_code`, `l10n_latam_company_use_documents` |  |  | `l10n_latam_invoice_document` |
| `l10n_mx.view_account_journal_form_inherit` | field | `account.view_account_journal_form` | `restrict_mode_hash_table` |  |  | `l10n_mx` |
| `l10n_sa_edi.view_account_journal_form` | xpath | `account.view_account_journal_form` | `l10n_sa_csr`, `l10n_sa_compliance_csid_json`, `l10n_sa_production_csid_json`, `l10n_sa_compliance_checks_passed`, `l10n_sa_csr_errors`, `l10n_sa_production_csid_validity` | `%(l10n_sa_edi_otp_wizard_act_window)d`, `%(l10n_sa_edi_otp_wizard_act_window)d`, `%(l10n_sa_edi_otp_wizard_act_window)d`, `%(l10n_sa_edi_otp_wizard_act_window)d` |  | `l10n_sa_edi` |
| `l10n_se.view_account_journal_se_ocr_form` | xpath | `account.view_account_journal_form` | `l10n_se_invoice_ocr_length` |  |  | `l10n_se` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_account_journal_form` | Journals | list,kanban,form |  |  |  | `account` |
| `account.open_account_journal_dashboard_kanban` | Dashboard | kanban,form | `[]` | `{'search_default_dashboard':1}` |  | `account` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `account.mail_template_einvoice_notification` | New eInvoices Notification | New Electronic Invoices Received |

Machine-readable definition: `../../../schemas/data/entities/account.journal.json`; views: `../../../schemas/interfaces/views/account.journal.json`.
