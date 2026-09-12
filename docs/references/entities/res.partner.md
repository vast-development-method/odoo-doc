# Contact (`res.partner`)

**Transport name:** `res.partner`  
**Storage name:** `res_partner`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `bus`, `mail`, `mail`, `product`, `auth_signup`, `portal`, `account`, `account_add_gln`, `account_edi_ubl_cii`, `payment`, `account_peppol`, `account_peppol_response`, `contacts`, `base_address_extended`, `phone_validation`, `base_geolocalize`, `base_vat`, `calendar`, `crm`, `im_livechat`, `mail_plugin`, `sale`, `delivery`, `stock`, `delivery_mondialrelay`, `event`, `google_address_autocomplete`, `hr`, `hr_calendar`, `hr_holidays`, `hr_homeworking`, `hr_homeworking_calendar`, `hr_recruitment`, `survey`, `website`, `website_partner`, `website_slides`, `project`, `partner_autocomplete`, `point_of_sale`, `l10n_anz_ubl_pint`, `l10n_latam_base`, `l10n_ar`, `l10n_ar_pos`, `website_sale`, `l10n_ar_withholding`, `l10n_au`, `l10n_be`, `pos_sale`, `l10n_br`, `l10n_ca`, `l10n_cl`, `l10n_de`, `purchase`, `l10n_dk`, `l10n_dk_nemhandel`, `l10n_dk_nemhandel_response`, `l10n_dk_oioubl`, `l10n_ec`, `l10n_eg_edi_eta`, `l10n_es`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_fi`, `l10n_fr`, `l10n_fr_account`, `l10n_fr_pdp`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_hu`, `l10n_hu_edi`, `l10n_id_efaktur_coretax`, `l10n_in`, `l10n_in_edi`, `l10n_in_pos`, `purchase_stock`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_jp_ubl_pint`, `l10n_ke_edi_tremol`, `l10n_lk_invoice`, `l10n_ma`, `l10n_my_ubl_pint`, `l10n_my_edi`, `l10n_no`, `l10n_nz`, `l10n_pe`, `l10n_pe_pos`, `l10n_ph`, `l10n_pl`, `l10n_pl_edi`, `l10n_pl_edi_jst`, `l10n_ro`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_se`, `l10n_sg`, `l10n_sg_ubl_pint`, `l10n_th`, `l10n_tr_nilvera`, `l10n_tr_nilvera_base_vat`, `l10n_tr_nilvera_edispatch`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_tw_edi_ecpay_website_sale`, `l10n_uy`, `l10n_uz`, `l10n_vn_edi_viettel`, `l10n_vn_edi_viettel_pos`, `loyalty`, `mass_mailing`, `mrp_subcontracting`, `partnership`, `pos_hr`, `pos_loyalty`, `pos_self_order`, `privacy_lookup`, `sale_gelato`, `snailmail`, `snailmail_account`, `website_crm_partner_assign`, `website_customer`, `website_sale_wishlist`

Description: Contact

## Identity and behavior

- Mixins (classical inheritance): `format.address.mixin`, `format.vat.label.mixin`, `avatar.mixin`, `properties.base.definition.mixin`, `bus.listener.mixin`, `mail.activity.mixin`, `mail.thread.blacklist`, `mail.thread.phone`, `website.published.multi.mixin`, `website.seo.metadata`, `pos.load.mixin`
- Default ordering: `complete_name ASC, id DESC`
- Display name search fields: `["complete_name", "email", "ref", "vat", "company_registry"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (293)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | changes are tracked in the message thread; indexed; extended by packages `mail`, `account` |
| `complete_name` | Complete Name | single line text |  | computed by rule `_compute_complete_name` and stored; indexed |
| `parent_id` | Related Company | many to one | `res.partner` | changes are tracked in the message thread; indexed; extended by packages `mail` |
| `parent_name` | Parent name | single line text |  | read only; related through path `parent_id.name` |
| `child_ids` | Contact | one to many | `res.partner` | restricted by domain `[["active", "=", true]]`; inverse field `parent_id` |
| `ref` | Reference | single line text |  | indexed |
| `lang` | Language | selection |  | computed by rule `_compute_lang` and stored; Help: All the emails and documents sent to this contact will be translated in this language. |
| `active_lang_count` | Active Lang Count | integer |  | computed by rule `_compute_active_lang_count` (not stored) |
| `tz` | Timezone | selection |  | default computed dynamically (lambda self: self.env.context.get('tz')); Help: When printing documents and exporting/importing data, time values are computed according to this timezone. If the timezone is not set, UTC (Coordinated Universal Time) is used. Anywhere else, time values are computed according to the time offset of your web client. |
| `tz_offset` | Timezone offset | single line text |  | computed by rule `_compute_tz_offset` (not stored) |
| `user_id` | Salesperson | many to one | `res.users` | computed by rule `_compute_user_id` and stored; changes are tracked in the message thread; precomputed before insertion; Help: The internal user in charge of this contact.; extended by packages `mail` |
| `vat` | Identification Number | single line text |  | writable through an inverse rule; changes are tracked in the message thread; indexed; Help: Identification Number for selected type; extended by packages `mail`, `base_vat`, `l10n_latam_base` |
| `vat_label` | Tax identifier Label | single line text |  | computed by rule `_compute_vat_label` (not stored) |
| `same_vat_partner_id` | Partner with same Tax identifier | many to one | `res.partner` | computed by rule `_compute_same_vat_partner_id` (not stored) |
| `same_company_registry_partner_id` | Partner with same Company Registry | many to one | `res.partner` | computed by rule `_compute_same_vat_partner_id` (not stored) |
| `company_registry` | Company identifier | single line text |  | computed by rule `_compute_company_registry` and stored; indexed (btree_not_null); Help: The registry number of the company. Use it if it is different from the Tax ID. It must be unique across all partners of a same country |
| `company_registry_label` | Company identifier Label | single line text |  | computed by rule `_compute_company_registry_label` (not stored) |
| `company_registry_placeholder` | Company Registry Placeholder | single line text |  | computed by rule `_compute_company_registry_placeholder` (not stored) |
| `bank_ids` | Banks | one to many | `res.partner.bank` | inverse field `partner_id` |
| `website` | Website Link | single line text |  |  |
| `comment` | Notes | rich text |  |  |
| `category_id` | Tags | many to many | `res.partner.category` | default computed dynamically (_default_category) |
| `active` | Active | boolean |  | default `True` |
| `employee` | Employee | boolean |  | computed by rule `_compute_employee` and stored; not copied on duplication; Help: Whether this contact is an Employee.; extended by packages `hr` |
| `function` | Job Position | single line text |  |  |
| `type` | Address Type | selection |  | default `contact`; extended by packages `l10n_es_edi_facturae` |
| `type_address_label` | Address Type Description | single line text |  | computed by rule `_compute_type_address_label` (not stored) |
| `street` | Street | single line text |  |  |
| `street2` | Street2 | single line text |  |  |
| `zip` | Zip | single line text |  |  |
| `city` | City | single line text |  |  |
| `state_id` | State | many to one | `res.country.state` | on delete of the target: restrict; restricted by domain `[('country_id', '=?', country_id)]` |
| `country_id` | Country | many to one | `res.country` | writable through an inverse rule; on delete of the target: restrict; extended by packages `base_vat` |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |
| `partner_latitude` | Geo Latitude | float |  | precision `[10, 7]` |
| `partner_longitude` | Geo Longitude | float |  | precision `[10, 7]` |
| `email` | Email | single line text |  | changes are tracked in the message thread; extended by packages `mail` |
| `email_formatted` | Formatted Email | single line text |  | computed by rule `_compute_email_formatted` (not stored); Help: Format email address "Name <email@domain>" |
| `phone` | Phone | single line text |  | changes are tracked in the message thread; extended by packages `mail` |
| `is_company` | Is a Company | boolean |  | default ; Help: Check if the contact is a company, otherwise it is a person |
| `is_public` | Is Public | boolean |  | computed by rule `_compute_is_public` (not stored) |
| `industry_id` | Industry | many to one | `res.partner.industry` |  |
| `company_type` | Company Type | selection |  | computed by rule `_compute_company_type` (not stored); writable through an inverse rule |
| `company_id` | Company | many to one | `res.company` | indexed |
| `color` | Color Index | integer |  | default  |
| `user_ids` | Users | one to many | `res.users` | inverse field `partner_id` |
| `main_user_id` | Main User | many to one | `res.users` | computed by rule `_compute_main_user_id` (not stored); Help: There can be several users related to the same partner. When a single user is needed, this field attempts to find the most appropriate one. |
| `partner_share` | Share Partner | boolean |  | computed by rule `_compute_partner_share` and stored; Help: Either customer (not a user), either shared user. Indicated the current partner is a customer without access or with a limited access created for sharing data. |
| `contact_address` | Complete Address | single line text |  | computed by rule `_compute_contact_address` (not stored) |
| `commercial_partner_id` | Commercial Entity | many to one | `res.partner` | computed by rule `_compute_commercial_partner` and stored; indexed; recursive dependency |
| `commercial_company_name` | Company Name Entity | single line text |  | computed by rule `_compute_commercial_company_name` and stored |
| `company_name` | Company Name | single line text |  |  |
| `barcode` | Barcode | single line text |  | value is company dependent; not copied on duplication; Help: Use a barcode to identify this contact. |
| `self` | Self | many to one | `res.partner` | computed by rule `_compute_get_ids` (not stored) |
| `application_statistics` | Stats | structured document |  | computed by rule `_compute_application_statistics` (not stored) |
| `channel_ids` | Channels | many to many | `discuss.channel` | not copied on duplication; association table `discuss_channel_member` |
| `channel_member_ids` | Channel Member | one to many | `discuss.channel.member` | inverse field `partner_id` |
| `is_in_call` | Is In Call | boolean |  | computed by rule `_compute_is_in_call` (not stored); visible only to groups `base.group_system` |
| `rtc_session_ids` | Rtc Session | one to many | `discuss.channel.rtc.session` | inverse field `partner_id` |
| `contact_address_inline` | Inlined Complete Address | single line text |  | computed by rule `_compute_contact_address_inline` (not stored); changes are tracked in the message thread |
| `im_status` | IM Status | single line text |  | computed by rule `_compute_im_status` (not stored) |
| `offline_since` | Offline since | date and time |  | computed by rule `_compute_im_status` (not stored) |
| `property_product_pricelist` | Pricelist | many to one | `product.pricelist` | computed by rule `_compute_product_pricelist` (not stored); writable through an inverse rule; restricted by domain `lambda self: [('company_id', 'in', (self.env.company.id, False))]`; Help: Used for sales to the current partner |
| `specific_property_product_pricelist` | Specific Property Product Pricelist | many to one | `product.pricelist` | value is company dependent |
| `signup_type` | Signup Token Type | single line text |  | not copied on duplication; visible only to groups `base.group_erp_manager` |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | computed by rule `_compute_fiscal_country_codes` (not stored) |
| `fiscal_country_group_codes` | Fiscal Country Group Codes | structured document |  | computed by rule `_compute_fiscal_country_group_codes` (not stored) |
| `partner_vat_placeholder` | Partner Value-added tax Placeholder | single line text |  | computed by rule `_compute_partner_vat_placeholder` (not stored) |
| `partner_company_registry_placeholder` | Partner Company Registry Placeholder | single line text |  | computed by rule `_compute_partner_company_registry_placeholder` (not stored) |
| `duplicate_bank_partner_ids` | Duplicate Bank Partner | many to many |  | related through path `bank_ids.duplicate_bank_partner_ids` |
| `credit` | Total Receivable | monetary |  | computed by rule `_credit_debit_get` (not stored); searchable through a search rule; visible only to groups `account.group_account_invoice,account.group_account_readonly`; Help: Total amount this customer owes you. |
| `credit_to_invoice` | Credit To Invoice | monetary |  | computed by rule `_compute_credit_to_invoice` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `credit_limit` | Credit Limit | float |  | value is company dependent; not copied on duplication; visible only to groups `account.group_account_invoice,account.group_account_readonly`; Help: Credit limit specific to this partner. |
| `use_partner_credit_limit` | Partner Limit | boolean |  | computed by rule `_compute_use_partner_credit_limit` (not stored); writable through an inverse rule; visible only to groups `account.group_account_invoice,account.group_account_readonly`; Help: Set a value greater than 0.0 to activate a credit limit check |
| `show_credit_limit` | Show Credit Limit | boolean |  | computed by rule `_compute_show_credit_limit` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `days_sales_outstanding` | Days Sales Outstanding (DSO) | float |  | computed by rule `_compute_days_sales_outstanding` (not stored); Help: [(Total Receivable/Total Revenue) * number of days since the first invoice] for this customer |
| `debit` | Total Payable | monetary |  | computed by rule `_credit_debit_get` (not stored); searchable through a search rule; visible only to groups `account.group_account_invoice,account.group_account_readonly`; Help: Total amount you have to pay to this vendor. |
| `total_invoiced` | Total Invoiced | monetary |  | computed by rule `_invoice_total` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `currency_id` | Currency | many to one | `res.currency` | read only; computed by rule `_get_company_currency` (not stored) |
| `property_account_payable_id` | Account Payable | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `[('account_type', '=', 'liability_payable')]`; must belong to the same company |
| `property_account_receivable_id` | Account Receivable | many to one | `account.account` | value is company dependent; on delete of the target: restrict; restricted by domain `[('account_type', '=', 'asset_receivable')]`; must belong to the same company |
| `property_account_position_id` | Fiscal Position | many to one | `account.fiscal.position` | value is company dependent; must belong to the same company; Help: The fiscal position determines the taxes/accounts used for this contact. |
| `property_payment_term_id` | Customer Payment Terms | many to one | `account.payment.term` | value is company dependent; on delete of the target: restrict; must belong to the same company |
| `property_supplier_payment_term_id` | Vendor Payment Terms | many to one | `account.payment.term` | value is company dependent; must belong to the same company |
| `ref_company_ids` | Companies that refers to partner | one to many | `res.company` | inverse field `partner_id` |
| `supplier_invoice_count` | # Vendor Bills | integer |  | computed by rule `_compute_supplier_invoice_count` (not stored) |
| `account_move_count` | Account Move Count | integer |  | computed by rule `_compute_account_move_count` (not stored); visible only to groups `account.group_account_invoice,account.group_account_readonly` |
| `invoice_ids` | Invoices | one to many | `account.move` | read only; not copied on duplication; inverse field `partner_id` |
| `contract_ids` | Partner Contracts | one to many | `account.analytic.account` | read only; inverse field `partner_id` |
| `bank_account_count` | Bank | integer |  | computed by rule `_compute_bank_count` (not stored) |
| `trust` | Degree of trust you have in this debtor | selection |  | value is company dependent |
| `ignore_abnormal_invoice_date` | Ignore Abnormal Invoice Date | boolean |  | value is company dependent |
| `ignore_abnormal_invoice_amount` | Ignore Abnormal Invoice Amount | boolean |  | value is company dependent |
| `invoice_sending_method` | Invoice sending | selection |  | value is company dependent; extended by packages `account_peppol`, `l10n_dk_nemhandel`, `l10n_hr_edi`, `snailmail_account` |
| `invoice_edi_format` | eInvoice format | selection |  | computed by rule `_compute_invoice_edi_format` (not stored); writable through an inverse rule; extended by packages `account_edi_ubl_cii`, `l10n_anz_ubl_pint`, `l10n_dk_nemhandel`, `l10n_dk_oioubl`, `l10n_es_edi_facturae`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_it_edi`, `l10n_jp_ubl_pint`, `l10n_my_ubl_pint`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_sg_ubl_pint`, `l10n_tr_nilvera`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel` |
| `invoice_edi_format_store` | Invoice Electronic data interchange Format Store | single line text |  | value is company dependent |
| `display_invoice_edi_format` | Display Invoice Electronic data interchange Format | boolean |  | default computed dynamically (lambda self: len(self._fields['invoice_edi_format']._description_selection(self.env))) |
| `invoice_template_pdf_report_id` | Invoice report | many to one | `ir.actions.report` | restricted by domain `[('id', 'in', available_invoice_template_pdf_report_ids)]` |
| `available_invoice_template_pdf_report_ids` | Available Invoice Template Portable Document Format Report | one to many | `ir.actions.report` | computed by rule `_compute_available_invoice_template_pdf_report_ids` (not stored) |
| `display_invoice_template_pdf_report_id` | Display Invoice Template Portable Document Format Report | boolean |  | default computed dynamically (_default_display_invoice_template_pdf_report_id) |
| `supplier_rank` | Supplier Rank | integer |  | default ; not copied on duplication |
| `customer_rank` | Customer Rank | integer |  | default ; not copied on duplication |
| `autopost_bills` | Auto-post bills | selection |  | required; default `ask`; Help: Automatically post bills for this trusted partner |
| `property_outbound_payment_method_line_id` | Property Outbound Payment Method Line | many to one | `account.payment.method.line` | value is company dependent; restricted by domain `lambda self: [('journal_id.active', '=', True), ('payment_type', '=', 'outbound'), ('company_id', 'parent_of', self.env.company.id)]`; must belong to the same company |
| `property_inbound_payment_method_line_id` | Property Inbound Payment Method Line | many to one | `account.payment.method.line` | value is company dependent; restricted by domain `lambda self: [('journal_id.active', '=', True), ('payment_type', '=', 'inbound'), ('company_id', 'parent_of', self.env.company.id)]`; must belong to the same company |
| `global_location_number` | global location number | single line text |  | Help: Global Location Number |
| `is_ubl_format` | Is Universal Business Language Format | boolean |  | computed by rule `_compute_is_ubl_format` (not stored) |
| `is_peppol_edi_format` | Is the pan-European public procurement online network Electronic data interchange Format | boolean |  | computed by rule `_compute_is_peppol_edi_format` (not stored) |
| `peppol_endpoint` | Peppol Endpoint | single line text |  | computed by rule `_compute_peppol_endpoint` and stored; changes are tracked in the message thread; Help: Unique identifier used by the BIS Billing 3.0 and its derivatives, also known as 'Endpoint ID'. |
| `peppol_eas` | Peppol e-address (EAS) | selection |  | computed by rule `_compute_peppol_eas` and stored; changes are tracked in the message thread; Help: Code used to identify the Endpoint for BIS Billing 3.0 and its derivatives.              List available at https://docs.peppol.eu/poacc/billing/3.0/codelist/eas/; extended by packages `account_peppol` |
| `available_peppol_eas` | Available the pan-European public procurement online network Eas | structured document |  | computed by rule `_compute_available_peppol_eas` (not stored) |
| `payment_token_ids` | Payment Tokens | one to many | `payment.token` | inverse field `partner_id` |
| `payment_token_count` | Payment Token Count | integer |  | computed by rule `_compute_payment_token_count` (not stored) |
| `available_peppol_sending_methods` | Available the pan-European public procurement online network Sending Methods | structured document |  | computed by rule `_compute_available_peppol_sending_methods` (not stored) |
| `available_peppol_edi_formats` | Available the pan-European public procurement online network Electronic data interchange Formats | structured document |  | computed by rule `_compute_available_peppol_edi_formats` (not stored) |
| `peppol_verification_state` | Peppol status | selection |  | value is company dependent |
| `peppol_supported_documents` | Supported Peppol Documents | structured document |  |  |
| `peppol_response_support` | Peppol Response Service | boolean |  | computed by rule `_compute_response_support` (not stored) |
| `street_name` | Street Name | single line text |  | computed by rule `_compute_street_data` and stored; writable through an inverse rule |
| `street_number` | House | single line text |  | computed by rule `_compute_street_data` and stored; writable through an inverse rule |
| `street_number2` | Door | single line text |  | computed by rule `_compute_street_data` and stored; writable through an inverse rule |
| `city_id` | City identifier | many to one | `res.city` |  |
| `country_enforce_cities` | Country Enforce Cities | boolean |  | related through path `country_id.enforce_cities` |
| `date_localization` | Geolocation Date | date |  |  |
| `vies_valid` | Intra-Community Valid | boolean |  | computed by rule `_compute_vies_valid` and stored; changes are tracked in the message thread; Help: European VAT numbers are automatically checked on the VIES database. |
| `perform_vies_validation` | Perform Vies Validation | boolean |  | computed by rule `_compute_perform_vies_validation` (not stored) |
| `meeting_count` | # Meetings | integer |  | computed by rule `_compute_meeting_count` (not stored) |
| `meeting_ids` | Meetings | many to many | `calendar.event` | not copied on duplication; association table `calendar_event_res_partner_rel` |
| `calendar_last_notif_ack` | Last notification marked as read from base Calendar | date and time |  | default computed dynamically (fields.Datetime.now) |
| `opportunity_ids` | Opportunities | one to many | `crm.lead` | restricted by domain `[["type", "=", "opportunity"]]`; inverse field `partner_id` |
| `opportunity_count` | Opportunity Count | integer |  | computed by rule `_compute_opportunity_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `user_livechat_username` | User Livechat Username | single line text |  | computed by rule `_compute_user_livechat_username` (not stored) |
| `chatbot_script_ids` | Chatbot Script | one to many | `chatbot.script` | inverse field `operator_partner_id` |
| `livechat_channel_count` | Livechat Channel Count | integer |  | computed by rule `_compute_livechat_channel_count` (not stored) |
| `iap_enrich_info` | in-app purchase Enrich Info | multi line text |  | computed by rule `_compute_partner_iap_info` (not stored); Help: IAP response stored as a JSON string |
| `iap_search_domain` | Search Domain / Email | single line text |  | computed by rule `_compute_partner_iap_info` (not stored) |
| `sale_order_count` | Sale Order Count | integer |  | computed by rule `_compute_sale_order_count` (not stored); visible only to groups `sales_team.group_sale_salesman` |
| `sale_order_ids` | Sales Order | one to many | `sale.order` | inverse field `partner_id` |
| `sale_warn_msg` | Message for Sales Order | multi line text |  |  |
| `property_delivery_carrier_id` | Delivery Method | many to one | `delivery.carrier` | value is company dependent; Help: Used in sales orders. |
| `is_pickup_location` | Is Pickup Location | boolean |  |  |
| `property_stock_customer` | Customer Location | many to one | `stock.location` | value is company dependent; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', allowed_company_ids[0])]`; must belong to the same company; Help: The stock location used as destination when sending goods to this contact. |
| `property_stock_supplier` | Vendor Location | many to one | `stock.location` | value is company dependent; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', allowed_company_ids[0])]`; must belong to the same company; Help: The stock location used as source when receiving goods from this contact. |
| `picking_warn_msg` | Message for Stock Picking | multi line text |  |  |
| `is_mondialrelay` | Is Mondialrelay | boolean |  | computed by rule `_compute_is_mondialrelay` (not stored) |
| `event_count` | # Events | integer |  | computed by rule `_compute_event_count` (not stored); visible only to groups `event.group_event_registration_desk` |
| `static_map_url` | Static Map Uniform resource locator | single line text |  | computed by rule `_compute_static_map_url` (not stored) |
| `static_map_url_is_valid` | Static Map Uniform resource locator Is Valid | boolean |  | computed by rule `_compute_static_map_url_is_valid` (not stored) |
| `employee_ids` | Employees | one to many | `hr.employee` | visible only to groups `hr.group_hr_user`; inverse field `work_contact_id`; Help: Related employees based on their private address |
| `employees_count` | Employees Count | integer |  | computed by rule `_compute_employees_count` (not stored); visible only to groups `hr.group_hr_user` |
| `leave_date_to` | Leave Date To | date |  | computed by rule `_compute_leave_date_to` (not stored) |
| `applicant_ids` | Applicants | one to many | `hr.applicant` | inverse field `partner_id` |
| `certifications_count` | Certifications Count | integer |  | computed by rule `_compute_certifications_count` (not stored) |
| `certifications_company_count` | Company Certifications Count | integer |  | computed by rule `_compute_certifications_company_count` (not stored) |
| `visitor_ids` | Visitors | one to many | `website.visitor` | inverse field `partner_id` |
| `website_description` | Website Partner Full Description | rich text |  | translatable |
| `website_short_description` | Website Partner Short Description | multi line text |  | translatable |
| `is_published` | Is Published | boolean |  | changes are tracked in the message thread |
| `slide_channel_ids` | eLearning Courses | many to many | `slide.channel` | computed by rule `_compute_slide_channel_values` (not stored); searchable through a search rule; visible only to groups `website_slides.group_website_slides_officer` |
| `slide_channel_completed_ids` | Completed Courses | one to many | `slide.channel` | computed by rule `_compute_slide_channel_values` (not stored); searchable through a search rule; visible only to groups `website_slides.group_website_slides_officer` |
| `slide_channel_count` | Course Count | integer |  | computed by rule `_compute_slide_channel_values` (not stored); visible only to groups `website_slides.group_website_slides_officer` |
| `slide_channel_company_count` | Company Course Count | integer |  | computed by rule `_compute_slide_channel_company_count` (not stored); visible only to groups `website_slides.group_website_slides_officer` |
| `project_ids` | Projects | one to many | `project.project` | inverse field `partner_id` |
| `task_ids` | Tasks | one to many | `project.task` | inverse field `partner_id` |
| `task_count` | # Tasks | integer |  | computed by rule `_compute_task_count` (not stored) |
| `pos_order_count` | Point of sale Order Count | integer |  | computed by rule `_compute_pos_order` (not stored); visible only to groups `point_of_sale.group_pos_user`; Help: The number of point of sales orders related to this customer |
| `pos_order_ids` | Point of sale Order | one to many | `pos.order` | read only; inverse field `partner_id` |
| `pos_contact_address` | PoS Address | single line text |  | computed by rule `_compute_pos_contact_address` (not stored) |
| `invoice_emails` | Invoice Emails | single line text |  | read only; computed by rule `_compute_invoice_emails` (not stored) |
| `fiscal_position_id` | Automatic Fiscal Position | many to one | `account.fiscal.position` | computed by rule `_compute_fiscal_position_id` (not stored); Help: Fiscal positions are used to adapt taxes and accounts for particular customers or sales orders/invoices. The default value comes from the customer. |
| `l10n_latam_identification_type_id` | Identification Type | many to one | `l10n_latam.identification.type` | writable through an inverse rule; default computed dynamically (lambda self: self.env.ref('l10n_latam_base.it_vat', raise_if_not_found=False)); indexed (btree_not_null); Help: The type of identification |
| `is_vat` | Is Value-added tax | boolean |  | related through path `l10n_latam_identification_type_id.is_vat` |
| `l10n_ar_vat` | value-added tax | single line text |  | computed by rule `_compute_l10n_ar_vat` (not stored); Help: Computed field that returns VAT or nothing if this one is not set for the partner |
| `l10n_ar_formatted_vat` | Formatted value-added tax | single line text |  | computed by rule `_compute_l10n_ar_formatted_vat` (not stored); Help: Computed field that will convert the given VAT number to the format {person_category:2}-{number:10}-{validation_number:1} |
| `l10n_ar_gross_income_number` | Gross Income Number | single line text |  |  |
| `l10n_ar_gross_income_type` | Gross Income Type | selection |  | Help: Argentina: Type of gross income: exempt, local, multilateral. |
| `l10n_ar_afip_responsibility_type_id` | ARCA Responsibility Type | many to one | `l10n_ar.afip.responsibility.type` | indexed (btree_not_null); Help: Defined by ARCA to identify the type of responsibilities that a person or a legal entity could have and that impacts in the type of operations and requirements they need. |
| `l10n_ar_partner_tax_ids` | Argentinean Withholding Taxes | one to many | `l10n_ar.partner.tax` | inverse field `partner_id` |
| `l10n_br_ie_code` | IE | single line text |  | Help: State Tax Identification Number. Should contain 9-14 digits. |
| `l10n_br_im_code` | IM | single line text |  | Help: Municipal Tax Identification Number |
| `l10n_br_isuf_code` | SUFRAMA code | single line text |  | Help: SUFRAMA registration number. |
| `l10n_ca_pst` | PST number | single line text |  | Help: Canadian Provincial Tax Identification Number (PST) or Québec Sales Tax (QST) |
| `l10n_cl_sii_taxpayer_type` | Taxpayer Type | selection |  | indexed (btree_not_null); Help: 1 - VAT Affected (1st Category) (Most of the cases) 2 - Fees Receipt Issuer (Applies to suppliers who issue fees receipt) 3 - End consumer (only receipts) 4 - Foreigner |
| `l10n_cl_activity_description` | Activity Description | single line text |  | Help: Chile: Economic activity. |
| `property_purchase_currency_id` | Supplier Currency | many to one | `res.currency` | value is company dependent; Help: This currency will be used for purchases from the current partner |
| `purchase_order_count` | Purchase Order Count | integer |  | computed by rule `_compute_purchase_order_count` (not stored); visible only to groups `purchase.group_purchase_user` |
| `purchase_warn_msg` | Message for Purchase Order | multi line text |  |  |
| `receipt_reminder_email` | Receipt Reminder | boolean |  | value is company dependent; Help: Automatically send a confirmation email to the vendor X days before the expected receipt date, asking him to confirm the exact date. |
| `reminder_date_before_receipt` | Days Before Receipt | integer |  | value is company dependent; Help: Number of days to send reminder email before the promised receipt date |
| `buyer_id` | Buyer | many to one | `res.users` |  |
| `nemhandel_verification_state` | Nemhandel endpoint verification | selection |  | value is company dependent |
| `nemhandel_identifier_type` | Nemhandel Endpoint Type | selection |  | computed by rule `_compute_nemhandel_identifier_type` and stored; changes are tracked in the message thread; Help: Unique identifier used by OIOUBL and Nemhandel |
| `nemhandel_identifier_value` | Nemhandel Endpoint | single line text |  | computed by rule `_compute_nemhandel_identifier_value` and stored; changes are tracked in the message thread; Help: Code used to identify the Endpoint on Nemhandel |
| `is_using_nemhandel` | Is Using Nemhandel | boolean |  | computed by rule `_compute_is_using_nemhandel` (not stored) |
| `nemhandel_supported_documents` | Supported Nemhandel Documents | structured document |  |  |
| `nemhandel_response_support` | Nemhandel Response Service | boolean |  | computed by rule `_compute_nemhandel_response_support` (not stored) |
| `l10n_ec_vat_validation` | value-added tax Error message validation | single line text |  | computed by rule `_compute_l10n_ec_vat_validation` (not stored); Help: Error message when validating the Ecuadorian VAT |
| `l10n_eg_building_no` | Building No. | single line text |  |  |
| `l10n_es_edi_facturae_ac_center_code` | Code | single line text |  | maximum length 10; Help: Code of the issuing department. |
| `l10n_es_edi_facturae_ac_role_type_ids` | Roles | many to many | `l10n_es_edi_facturae.ac_role_type` | Help: It indicates the role played by the Operational Point defined as a Workplace/Department. These functions are: - Receiver: Workplace associated to the recipient's tax identification number where the invoice will be received. - Payer: Workplace associated to the recipient's tax identification number responsible for paying the invoice. - Buyer: Workplace associated to the recipient's tax identification number who issued the purchase order. - Collector: Workplace associated to  the issuer's tax identification number responsible for handling the collection. - Fiscal: Workplace associated to the recipient's tax identification number, where an Operational Point mailbox is shared by different client companies with different tax identification numbers and it is necessary to differentiate between where the message is received (shared letterbox) and the workplace where it must be stored (recipient company). |
| `l10n_es_edi_facturae_ac_physical_gln` | Physical global location number | single line text |  | maximum length 14; Help: Identification of the connection point to the VAN EDI (Global Location Number). Barcode of 13 standard positions. Codes are registered in Spain by AECOC. The code is made up of the country code (2 positions) Spain is '84' + Company code (5 positions) + the remaining positions. The last one is the product + check digit. |
| `l10n_es_edi_facturae_ac_logical_operational_point` | Logical Operational Point | single line text |  | maximum length 14; Help: Code identifying the company. Barcode of 13 standard positions. Codes are registered in Spain by AECOC. The code is made up of the country code (2 positions) Spain is '84' + Company code (5 positions) + the remaining positions. The last one is the product + check digit. |
| `l10n_es_edi_facturae_residence_type` | Facturae electronic data interchange Residency Type Code | single line text |  | read only; computed by rule `_compute_l10n_es_edi_facturae_residence_type` (not stored) |
| `l10n_fr_is_french` | Localization Fr Is French | boolean |  | computed by rule `_compute_l10n_fr_is_french` (not stored) |
| `pdp_verification_display_state` | E-Invoicing State | selection |  | computed by rule `_compute_pdp_verification_display_state` (not stored) |
| `l10n_gr_edi_branch_number` | Branch Number | integer |  | computed by rule `_compute_l10n_gr_edi_branch_number` and stored; Help: Branch number in the Tax Registry |
| `l10n_hr_personal_oib` | Personal OIB | single line text |  |  |
| `l10n_hr_business_unit_code` | Business Unit Code | single line text |  | default  |
| `l10n_hu_eu_vat` | Localization Hu Eu Value-added tax | single line text |  | computed by rule `_compute_l10n_hu_eu_vat` (not stored) |
| `l10n_hu_group_vat` | Group Tax identifier | single line text |  | indexed; maximum length 13; Help: If this company belongs to a VAT group, indicate the group's VAT number here. |
| `l10n_id_tku` | TKU | single line text |  | Help: Branch Number of your company, leave empty for headquarters |
| `l10n_id_buyer_document_type` | Document Type | selection |  | default `TIN` |
| `l10n_id_buyer_document_number` | Document Number | single line text |  |  |
| `l10n_id_nik` | NIK | single line text |  |  |
| `l10n_id_pkp` | Is PKP | boolean |  | computed by rule `_compute_l10n_id_pkp` and stored; Help: Denoting whether the following partner is taxable |
| `l10n_id_kode_transaksi` | Invoice Transaction Code | selection |  | default `04`; changes are tracked in the message thread; Help: he first 2 digits of tax code |
| `l10n_in_gst_treatment` | goods and services tax Treatment | selection |  |  |
| `l10n_in_pan_entity_id` | permanent account number | many to one | `l10n_in.pan.entity` | on delete of the target: restrict; Help: PAN enables the department to link all transactions of the person with the department. These transactions include taxpayments, TDS/TCS credits, returns of income/wealth/gift/FBT, specified transactions, correspondence, and so on. Thus, PAN acts as an identifier for the person with the tax department. |
| `l10n_in_tan` | TAN | single line text |  |  |
| `display_pan_warning` | Display pan warning | boolean |  | computed by rule `_compute_display_pan_warning` (not stored) |
| `l10n_in_gst_state_warning` | Localization In Goods and services tax State Warning | single line text |  | computed by rule `_compute_l10n_in_gst_state_warning` (not stored) |
| `l10n_in_is_gst_registered_enabled` | Localization In Is Goods and services tax Registered Enabled | boolean |  | computed by rule `_compute_l10n_in_gst_registered_and_status` (not stored) |
| `l10n_in_gstin_verified_status` | goods and services tax Status | boolean |  | changes are tracked in the message thread |
| `l10n_in_gstin_verified_date` | GSTIN Verified Date | date |  | changes are tracked in the message thread |
| `l10n_in_gstin_status_feature_enabled` | Localization In Gstin Status Feature Enabled | boolean |  | computed by rule `_compute_l10n_in_gst_registered_and_status` (not stored) |
| `purchase_line_ids` | Purchase Lines | one to many | `purchase.order.line` | inverse field `partner_id` |
| `on_time_rate` | On-Time Delivery Rate | float |  | computed by rule `_compute_on_time_rate` (not stored); Help: Over the past x days; the number of products received on time divided by the number of ordered products.x is either the System Parameter purchase_stock.on_time_delivery_days or the default 365 |
| `suggest_based_on` | Suggest Based On | single line text |  | default `30_days` |
| `suggest_days` | Suggest Days | integer |  | default `7` |
| `suggest_percent` | Suggest Percent | integer |  | default `100` |
| `group_rfq` | Group request for quotation | selection |  | required; default `default`; Help: Define if RFQ should be grouped         together based on expected arrival, except for dropship operations.          On Order: Replenishment needs will be grouped together except for MTO.          Daily: Replenishment needs will be grouped if the expected arrival is the same day          Weekly: Replenishment needs will be grouped if the expected arrival is the same week or week day          Always: Replenishment needs will always be grouped. |
| `group_on` | Week Day | selection |  | required; default `default` |
| `l10n_it_pec_email` | PEC e-mail | single line text |  |  |
| `l10n_it_codice_fiscale` | Codice Fiscale | single line text |  | maximum length 16 |
| `l10n_it_pa_index` | Destination Code (SDI) | single line text |  | maximum length 7; Help: Must contain the 6-character (or 7) code, present in the PA Index in the information relative to the electronic invoicing service, associated with the office which, within the addressee administration, deals with receiving (and processing) the invoice. |
| `l10n_it_edi_doi_ids` | Available Declarations of Intent of this partner | one to many | `l10n_it_edi_doi.declaration_of_intent` | restricted by domain `lambda self: [('company_id', '=', self.env.company.id)]`; inverse field `partner_id` |
| `l10n_ke_exemption_number` | Exemption Number | single line text |  | Help: The exemption number of the partner. Provided by the Kenyan government. |
| `l10n_lk_vat_registered` | Sri Lanka: value-added tax Registered | boolean |  | computed by rule `_compute_l10n_lk_vat_registered` and stored; Help: Indicates if this partner is registered for VAT in Sri Lanka. This defaults invoice printout to this partner to tax invoice for taxable supplies. |
| `sst_registration_number` | SST | single line text |  | Help: Malaysian Sales and Service Tax Number |
| `ttx_registration_number` | TTx | single line text |  | Help: Malaysian Tourism Tax Number |
| `l10n_my_tin_validation_state` | Tin Validation State | selection |  | computed by rule `_compute_l10n_my_tin_validation_state` and stored; Help: Technical field, hold the result of TIN validation using MyInvois API. It is non blocking, and will simply help ensure that the customer of an invoice is valid to avoid submission errors. |
| `l10n_my_edi_display_tin_warning` | Localization My Electronic data interchange Display Tin Warning | boolean |  | computed by rule `_compute_l10n_my_edi_display_tin_warning` (not stored) |
| `l10n_my_identification_type` | identifier Type | selection |  | default `BRN`; Help: The identification type and number used by the MyTax/MyInvois system to identify the user. Note: For MyPR and MyKAS to use NRIC scheme |
| `l10n_my_identification_number` | identifier Number | single line text |  |  |
| `l10n_my_identification_number_placeholder` | Localization My Identification Number Placeholder | single line text |  | computed by rule `_compute_l10n_my_identification_number_placeholder` (not stored) |
| `l10n_my_edi_industrial_classification` | Ind. Classification | many to one | `l10n_my_edi.industry_classification` | computed by rule `_compute_l10n_my_edi_industrial_classification` and stored |
| `l10n_my_edi_malaysian_tin` | Malaysian TIN | single line text |  | Help: The value set in this field will be used as TIN for the customer/supplier. If left empty, the Tax ID field will be used. |
| `l10n_no_bronnoysund_number` | Register of Legal Entities (Brønnøysund Register Center) | single line text |  | maximum length 9 |
| `l10n_pe_district` | District | many to one | `l10n_pe.res.city.district` | Help: Districts are part of a province or city. |
| `l10n_pe_district_name` | District name | single line text |  | related through path `l10n_pe_district.name` |
| `branch_code` | Branch Code | single line text |  | computed by rule `_compute_branch_code` and stored; default `000` |
| `first_name` | First Name | single line text |  |  |
| `middle_name` | Middle Name | single line text |  |  |
| `last_name` | Last Name | single line text |  |  |
| `l10n_ph_rdo` | RDO | single line text |  | Help: Revenue District Office |
| `l10n_pl_links_with_customer` | Links With Company | boolean |  | Help: TP: Existing connection or influence between the customer and the supplier |
| `l10n_pl_parent_lgu` | parent LGU | many to one | `res.partner` | Help: The local government unit (LGU) the partner is associated to. If present, it will be used in the FA (3) documents generated for this partner. |
| `nrc` | NRC | single line text |  | Help: Registration number at the Registry of Commerce |
| `l10n_rs_edi_registration_number` | Registration Number | single line text |  | maximum length 13; Help: Company ID ( Matični Broj ) assigned by the Serbian Business Registers Agency (APR) |
| `l10n_rs_edi_public_funds` | JBKJS | single line text |  | maximum length 5; Help: Unique Identifier of Public Funds Users such as Government agencies, public institutions and state-owned enterprises. |
| `l10n_sa_edi_building_number` | Building Number | single line text |  |  |
| `l10n_sa_edi_plot_identification` | Plot Identification | single line text |  |  |
| `l10n_sa_edi_additional_identification_scheme` | Identification Scheme | selection |  | default `OTH`; Help: Additional Identification Scheme for the Seller/Buyer |
| `l10n_sa_edi_additional_identification_number` | Identification Number (SA) | single line text |  | Help: Additional Identification Number for the Seller/Buyer |
| `l10n_se_check_vendor_ocr` | Check Vendor optical character recognition | boolean |  | Help: This Vendor uses OCR Number on their Vendor Bills. |
| `l10n_se_default_vendor_payment_ref` | Default Vendor Payment Ref | single line text |  | Help: If set, the vendor uses the same Default Payment Reference or OCR Number on all their Vendor Bills. |
| `l10n_sg_unique_entity_number` | UEN | single line text |  |  |
| `l10n_th_branch_name` | Localization Th Branch Name | single line text |  | computed by rule `_compute_l10n_th_branch_name` (not stored) |
| `l10n_tr_nilvera_customer_status` | Nilvera Status | selection |  | read only; default `not_checked`; changes are tracked in the message thread; not copied on duplication |
| `l10n_tr_nilvera_customer_alias_id` | Alias | many to one | `l10n_tr.nilvera.alias` | computed by rule `_compute_nilvera_customer_alias_id` and stored; not copied on duplication; restricted by domain `[('partner_id', '=', id)]` |
| `l10n_tr_nilvera_customer_alias_ids` | Localization Tr Nilvera Customer Alias | one to many | `l10n_tr.nilvera.alias` | inverse field `partner_id` |
| `l10n_tr_nilvera_edispatch_customs_zip` | Customs ZIP | single line text |  | maximum length 5; Help: The postal code of the customs office used to ship to the destination country. |
| `l10n_tr_tax_office_id` | Turkish Tax Office | many to one | `l10n_tr_nilvera_einvoice_extended.tax.office` |  |
| `l10n_tw_edi_require_paper_format` | Require Paper Format | boolean |  | Help: If checked, the partner requires paper format for ECPay e-invoices. |
| `l10n_vn_edi_symbol` | Default Symbol | many to one | `l10n_vn_edi_viettel.sinvoice.symbol` | value is company dependent; not copied on duplication; Help: If set, this symbol will be used as the default symbol for all invoices of this customer. |
| `loyalty_card_count` | Active loyalty cards | integer |  | computed by rule `_compute_count_active_cards` (not stored); visible only to groups `base.group_user,point_of_sale.group_pos_user`; extended by packages `pos_loyalty` |
| `property_stock_subcontractor` | Subcontractor Location | many to one | `stock.location` | value is company dependent; Help: The stock location used as source and destination when sending        goods to this contact during a subcontracting process. |
| `is_subcontractor` | Subcontractor | boolean |  | computed by rule `_compute_is_subcontractor` (not stored); searchable through a search rule |
| `bom_ids` | BoMs for which the Partner is one of the subcontractors | many to many | `mrp.bom` | computed by rule `_compute_bom_ids` (not stored) |
| `production_ids` | manufacturing Productions for which the Partner is the subcontractor | many to many | `mrp.production` | computed by rule `_compute_production_ids` (not stored) |
| `picking_ids` | Stock Pickings for which the Partner is the subcontractor | many to many | `stock.picking` | computed by rule `_compute_picking_ids` (not stored) |
| `grade_id` | Partner Level | many to one | `res.partner.grade` | changes are tracked in the message thread |
| `partner_weight` | Level Weight | integer |  | computed by rule `_compute_partner_weight` and stored; changes are tracked in the message thread; Help: This should be a numerical value greater than 0 which will decide the contention for this partner to take this lead/opportunity. |
| `grade_sequence` | Grade Sequence | integer |  | read only; related through path `grade_id.sequence` and stored |
| `activation` | Activation | many to one | `res.partner.activation` | changes are tracked in the message thread; indexed (btree_not_null) |
| `date_partnership` | Partnership Date | date |  |  |
| `date_review` | Latest Review | date |  |  |
| `date_review_next` | Next Review | date |  |  |
| `assigned_partner_id` | Implemented by | many to one | `res.partner` | indexed (btree_not_null) |
| `implemented_partner_ids` | Implementation References | one to many | `res.partner` | inverse field `assigned_partner_id` |
| `implemented_partner_count` | Implemented Partner Count | integer |  | computed by rule `_compute_implemented_partner_count` and stored |
| `website_tag_ids` | Website tags | many to many | `res.partner.tag` | association table `res_partner_res_partner_tag_rel`; Help: Filter published customers on the .../customers website page |
| `wishlist_ids` | Wishlist | one to many | `product.wishlist` | restricted by domain `[["active", "=", true]]`; inverse field `partner_id` |

## Selection values

### `type` (Address Type)

| Value | Label |
|---|---|
| `contact` | Contact |
| `invoice` | Invoice |
| `delivery` | Delivery |
| `other` | Other |
| `facturae_ac` | FACe Center |

### `company_type` (Company Type)

| Value | Label |
|---|---|
| `person` | Person |
| `company` | Company |

### `trust` (Degree of trust you have in this debtor)

| Value | Label |
|---|---|
| `good` | Good Debtor |
| `normal` | Normal Debtor |
| `bad` | Bad Debtor |

### `invoice_sending_method` (Invoice sending)

| Value | Label |
|---|---|
| `manual` | Manual |
| `email` | by Email |
| `peppol` | by Peppol |
| `nemhandel` | By Nemhandel |
| `mojeracun` | by MojEracun |
| `snailmail` | by Post |

### `invoice_edi_format` (eInvoice format)

| Value | Label |
|---|---|
| `facturx` | France (FacturX) |
| `ubl_bis3` | EU Standard (Peppol Bis 3.0) |
| `zugferd` | Germany (ZUGFeRD) |
| `xrechnung` | Germany (XRechnung) |
| `nlcius` | Netherlands (NLCIUS) |
| `ubl_a_nz` | Australia (BIS Billing 3.0 A-NZ) |
| `ubl_sg` | Singapore (BIS Billing 3.0 SG) |
| `pint_anz` | Australia (Peppol Pint AU) |
| `oioubl_21` | OIOUBL 2.1 |
| `oioubl_201` | Denmark (Oioubl) |
| `es_facturae` | Spain (FacturaE) |
| `ubl_21_fr` | France E-Invoicing (UBL 2.1) |
| `ubl_hr` | CIUS HR |
| `it_edi_xml` | Italy (Factura PA) |
| `pint_jp` | Japan (Peppol PINT JP) |
| `pint_my` | Malaysia (Peppol PINT MY) |
| `fa3_pl` | Polish FA3 |
| `ciusro` | Romania (CIUS RO) |
| `pint_sg` | Singapore (Peppol PINT SG) |
| `ubl_tr` | Türkiye (UBL TR 1.2) |
| `tw_ecpay` | ECPay |
| `vn_sinvoice` | Vietnam (SInvoice) |

### `autopost_bills` (Auto-post bills)

| Value | Label |
|---|---|
| `always` | Always |
| `ask` | Ask after 3 validations without edits |
| `never` | Never |

### `peppol_eas` (Peppol e-address (EAS))

| Value | Label |
|---|---|
| `9923` | Albania VAT |
| `9922` | Andorra VAT |
| `0151` | Australia ABN |
| `9914` | Austria UID |
| `9915` | Austria VOKZ |
| `0208` | Belgian Company Registry |
| `9925` | Belgian VAT |
| `9924` | Bosnia and Herzegovina VAT |
| `9926` | Bulgaria VAT |
| `9934` | Croatia VAT |
| `9928` | Cyprus VAT |
| `9929` | Czech Republic VAT |
| `0096` | Denmark P |
| `0184` | Denmark CVR |
| `0198` | Denmark SE |
| `0191` | Estonia Company code |
| `9931` | Estonia VAT |
| `0037` | Finland LY-tunnus |
| `0216` | Finland OVT code |
| `0213` | Finland VAT |
| `0002` | France SIRENE |
| `0009` | France SIRET |
| `9957` | France VAT |
| `0225` | France FRCTC Electronic Address |
| `0240` | France Register of legal persons |
| `0246` | German Electronic Business Address |
| `0204` | Germany Leitweg-ID |
| `9930` | Germany VAT |
| `9933` | Greece VAT |
| `9910` | Hungary VAT |
| `0196` | Iceland Kennitala |
| `9935` | Ireland VAT |
| `0211` | Italia Partita IVA |
| `0097` | Italia FTI |
| `0188` | Japan SST |
| `0221` | Japan IIN |
| `0218` | Latvia Unified registration number |
| `9939` | Latvia VAT |
| `9936` | Liechtenstein VAT |
| `0200` | Lithuania JAK |
| `9937` | Lithuania VAT |
| `9938` | Luxembourg VAT |
| `9942` | Macedonia VAT |
| `0230` | Malaysia |
| `9943` | Malta VAT |
| `9940` | Monaco VAT |
| `9941` | Montenegro VAT |
| `0106` | Netherlands KvK |
| `0190` | Netherlands OIN |
| `9944` | Netherlands VAT |
| `0244` | Nigeria Tax Identification |
| `0192` | Norway Org.nr. |
| `9945` | Poland VAT |
| `9946` | Portugal VAT |
| `9947` | Romania VAT |
| `9948` | Serbia VAT |
| `0195` | Singapore UEN |
| `0245` | SK Tax identification number (DIČ) |
| `9949` | Slovenia VAT |
| `9950` | Slovakia VAT |
| `9920` | Spain VAT |
| `0007` | Sweden Org.nr. |
| `9955` | Sweden VAT |
| `9927` | Swiss VAT |
| `0183` | Swiss UIDB |
| `9952` | Turkey VAT |
| `0235` | UAE Tax Identification Number (TIN) |
| `9932` | United Kingdom VAT |
| `9959` | USA EIN |
| `0060` | DUNS Number |
| `0088` | EAN Location Code |
| `0130` | Directorates of the European Commission |
| `0135` | SIA Object Identifiers |
| `0142` | SECETI Object Identifiers |
| `0193` | UBL.BE party identifier |
| `0199` | Legal Entity Identifier (LEI) |
| `0201` | Codice Univoco Unità Organizzativa iPA |
| `0202` | Indirizzo di Posta Elettronica Certificata |
| `0209` | GS1 identification keys |
| `0210` | Codice Fiscale |
| `9913` | Business Registers Network |
| `9918` | S.W.I.F.T |
| `9919` | Kennziffer des Unternehmensregisters |
| `9951` | San Marino VAT |
| `9953` | Vatican VAT |
| `AN` | O.F.T.P. (ODETTE File Transfer Protocol) |
| `AQ` | X.400 address for mail text |
| `AS` | AS2 exchange |
| `AU` | File Transfer Protocol |
| `EM` | Electronic mail |
| `odemo` | the demonstration data set ID |

### `peppol_verification_state` (Peppol status)

| Value | Label |
|---|---|
| `not_verified` | Unchecked |
| `not_valid` | Partner is not on Peppol |
| `not_valid_format` | Partner cannot receive format |
| `valid` | Partner is on Peppol |

### `l10n_ar_gross_income_type` (Gross Income Type)

| Value | Label |
|---|---|
| `multilateral` | Multilateral |
| `local` | Local |
| `exempt` | Exempt |

### `l10n_cl_sii_taxpayer_type` (Taxpayer Type)

| Value | Label |
|---|---|
| `1` | VAT Affected (1st Category) |
| `2` | Fees Receipt Issuer (2nd category) |
| `3` | End Consumer |
| `4` | Foreigner |

### `nemhandel_verification_state` (Nemhandel endpoint verification)

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `not_valid` | Not on Nemhandel |
| `valid` | Valid |

### `nemhandel_identifier_type` (Nemhandel Endpoint Type)

| Value | Label |
|---|---|
| `0088` | EAN/GLN |
| `0184` | CVR |
| `9918` | IBAN |
| `0198` | SE |

### `pdp_verification_display_state` (E-Invoicing State)

| Value | Label |
|---|---|
| `not_verified` | Not verified yet |
| `pdp_not_valid` | Partner is not in the annuaire |
| `pdp_not_valid_format` | Partner cannot receive format |
| `pdp_valid` | Partner is in the annuaire |
| `peppol_not_valid` | Partner is not on Peppol |
| `peppol_not_valid_format` | Partner cannot receive format |
| `peppol_valid` | Partner is on Peppol |

### `l10n_id_buyer_document_type` (Document Type)

| Value | Label |
|---|---|
| `TIN` | TIN |
| `NIK` | NIK |
| `Passport` | Passport |
| `Other` | Others |

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

### `group_rfq` (Group request for quotation)

| Value | Label |
|---|---|
| `default` | On Order |
| `day` | Daily |
| `week` | Weekly |
| `all` | Always |

### `group_on` (Week Day)

| Value | Label |
|---|---|
| `default` | Expected Date |
| `1` | Monday |
| `2` | Tuesday |
| `3` | Wednesday |
| `4` | Thursday |
| `5` | Friday |
| `6` | Saturday |
| `7` | Sunday |

### `l10n_my_tin_validation_state` (Tin Validation State)

| Value | Label |
|---|---|
| `valid` | Valid |
| `invalid` | Invalid |

### `l10n_my_identification_type` (identifier Type)

| Value | Label |
|---|---|
| `NRIC` | MyKad/MyTentera/MyPR/MyKAS |
| `BRN` | Business Registration Number |
| `PASSPORT` | Passport |
| `ARMY` | Army |

### `l10n_sa_edi_additional_identification_scheme` (Identification Scheme)

| Value | Label |
|---|---|
| `TIN` | Tax Identification Number |
| `CRN` | Commercial Registration Number |
| `MOM` | Momra License |
| `MLS` | MLSD License |
| `700` | 700 Number |
| `SAG` | Sagia License |
| `NAT` | National ID |
| `GCC` | GCC ID |
| `IQA` | Iqama Number |
| `PAS` | Passport ID |
| `OTH` | Other ID |

### `l10n_tr_nilvera_customer_status` (Nilvera Status)

| Value | Label |
|---|---|
| `not_checked` | Not Verified |
| `earchive` | E-Archive |
| `einvoice` | E-Invoice |

## State fields

State machine fields of this entity: `peppol_verification_state`, `nemhandel_verification_state`, `pdp_verification_display_state`, `l10n_my_tin_validation_state`, `l10n_tr_nilvera_customer_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_name` | Constraint | `CHECK( (type='contact' AND name IS NOT NULL) or (type!='contact') )` | Contacts require a name | `base` |
| `_l10n_it_codice_fiscale` | Constraint | `CHECK(l10n_it_codice_fiscale IS NULL OR l10n_it_codice_fiscale = '' OR LENGTH(l10n_it_codice_fiscale) >= 11)` | Codice fiscale must have between 11 and 16 characters. | `l10n_it_edi` |
| `_l10n_it_pa_index` | Constraint | `CHECK(l10n_it_pa_index IS NULL OR l10n_it_pa_index = '' OR LENGTH(l10n_it_pa_index) >= 6)` | Destination Code (SDI) must have between 6 and 7 characters. | `l10n_it_edi` |

## Operations (466)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_category` | preparation rule | self | `base` |  |  |
| `default_get` | lifecycle override | self, fields | `base`, `website_crm_partner_assign` | model | Add the company of the parent as default if we are creating a child partner. |
| `_compute_application_statistics` | computation | self | `base` |  |  |
| `_compute_application_statistics_hook` | computation | self | `account`, `base`, `calendar`, `crm`, `point_of_sale`, `purchase`, `sale` |  | Hook for override, as overriding compute method does not update cache accordingly. All overrides receive False instead of previously assigned value. |
| `_get_street_split` | preparation rule | self | `base_address_extended`, `base` |  |  |
| `_compute_avatar_1920` | computation | self | `base` | depends: `name`, `user_ids.share`, `image_1920`, `is_company`, `type` |  |
| `_compute_avatar_1024` | computation | self | `base` | depends: `name`, `user_ids.share`, `image_1024`, `is_company`, `type` |  |
| `_compute_avatar_512` | computation | self | `base` | depends: `name`, `user_ids.share`, `image_512`, `is_company`, `type` |  |
| `_compute_avatar_256` | computation | self | `base` | depends: `name`, `user_ids.share`, `image_256`, `is_company`, `type` |  |
| `_compute_avatar_128` | computation | self | `base` | depends: `name`, `user_ids.share`, `image_128`, `is_company`, `type` |  |
| `_compute_avatar` | computation | self, avatar_field, image_field | `base` |  |  |
| `_avatar_get_placeholder_path` | internal rule | self | `base`, `delivery_mondialrelay` |  |  |
| `_get_complete_name` | preparation rule | self | `base` |  |  |
| `_compute_complete_name` | computation | self | `base` | depends: `is_company`, `name`, `parent_id.name`, `type`, `company_name`, `commercial_company_name` |  |
| `_compute_lang` | computation | self | `base` | depends: `parent_id` | While creating / updating child contact, take the parent lang by default if any. 0therwise, fallback to default context / DB lang |
| `_compute_active_lang_count` | computation | self | `base` | depends: `lang` |  |
| `_compute_tz_offset` | computation | self | `base` | depends: `tz` |  |
| `_compute_user_id` | computation | self | `base` | depends: `parent_id` | Synchronize sales rep with parent if partner is a person |
| `_compute_main_user_id` | computation | self | `base` | depends_context: `uid`; depends: `user_ids.active`, `user_ids.share` |  |
| `_compute_partner_share` | computation | self | `base` | depends: `user_ids.share`, `user_ids.active` |  |
| `_compute_same_vat_partner_id` | computation | self | `base` | depends: `vat`, `company_id`, `company_registry`, `country_id` |  |
| `_compute_vat_label` | computation | self | `base` | depends_context: `company` |  |
| `_compute_type_address_label` | computation | self | `base` | depends: `parent_id`, `type` |  |
| `_compute_contact_address` | computation | self | `base` | depends: |  |
| `_compute_get_ids` | computation | self | `base` |  |  |
| `_compute_commercial_partner` | computation | self | `base` | depends: `is_company`, `parent_id.commercial_partner_id` |  |
| `_compute_commercial_company_name` | computation | self | `base` | depends: `company_name`, `parent_id.is_company`, `commercial_partner_id.name` |  |
| `_compute_company_registry` | computation | self | `base`, `l10n_be`, `l10n_dk`, `l10n_ro`, `l10n_se` | depends: `vat`, `country_id` |  |
| `_compute_company_registry_label` | computation | self | `base` | depends: `country_id` |  |
| `_get_company_registry_labels` | preparation rule | self | `base`, `l10n_au`, `l10n_de`, `l10n_dk`, `l10n_fi`, `l10n_ma`, `l10n_nz`, `l10n_uz` |  |  |
| `_compute_company_registry_placeholder` | computation | self | `base`, `l10n_dk`, `l10n_fr_account` | depends: `country_id.code`, `ref_company_ids.account_fiscal_country_id.code` |  |
| `_check_parent_id` | validation | self | `base` | constrains: `parent_id` |  |
| `_check_partner_company` | validation | self | `base` | constrains: `company_id` | Check that for every partner which has a company, if there exists a company linked to that partner, the company_id set on the partner is that company |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `onchange_parent_id` | on change | self | `base` | onchange: `parent_id` |  |
| `_onchange_country_id` | on change | self | `base_address_extended`, `base`, `l10n_latam_base` | onchange: `country_id` |  |
| `_onchange_state` | on change | self | `base` | onchange: `state_id` |  |
| `_onchange_company_id` | on change | self | `base` | onchange: `parent_id`, `company_id` |  |
| `_compute_email_formatted` | computation | self | `base` | depends: `name`, `email` | Compute formatted email for partner, using formataddr. Be defensive in computation, notably    * double format: if email already holds a formatted email like     'Name' <email@domain.com> we should not use it as it to compute     email formatted like "Name <'Name' <email@domain.com>>";   * multi emails: sometimes this field is used to hold several addresses     like email1@domain.com, email2@domain.com. We currently let this value     untouched, but remove any formatting from multi emails;   * invalid email: if something is wrong, keep it in email_formatted as     this eases management and und |
| `_compute_company_type` | computation | self | `base` | depends: `is_company` |  |
| `_write_company_type` | internal rule | self | `base` |  |  |
| `onchange_company_type` | on change | self | `base` | onchange: `company_type` |  |
| `_check_barcode_unicity` | validation | self | `base` | constrains: `barcode` |  |
| `_convert_fields_to_values` | internal rule | self, field_names | `base` |  | Returns dict of write() values for synchronizing `field_names` |
| `_address_fields` | internal rule | self | `base_address_extended`, `base`, `l10n_eg_edi_eta`, `l10n_sa_edi` | model | Returns the list of address fields that are synced from the parent. |
| `_formatting_address_fields` | internal rule | self | `base`, `l10n_pe` | model | Returns the list of address fields usable to format addresses. |
| `_get_address_values` | preparation rule | self | `base` |  | Get address values from record if at least one value is set. Otherwise it is considered empty and nothing is returned. |
| `_update_address` | internal rule | self, vals | `base` |  | Filter values from vals that are liked to address definition, and update recordset using super().write to avoid loops and side effects due to synchronization of address fields through partner hierarchy. |
| `_commercial_fields` | internal rule | self | `account`, `base`, `l10n_ar`, `l10n_cl`, `l10n_eg_edi_eta`, `l10n_hu_edi`, `l10n_in`, `l10n_ke_edi_tremol`, `l10n_latam_base`, `l10n_my_edi`, `l10n_my_ubl_pint`, `l10n_ph`, `l10n_ro`, `l10n_sa_edi`, `l10n_tw_edi_ecpay_website_sale` | model | Returns the list of fields that are managed by the commercial entity to which a partner belongs. These fields are meant to be hidden on partners that aren't `commercial entities` themselves, or synchronized at update (if present in _synced_commercial_fields), and will be delegated to the parent `commercial entity`. The list is meant to be extended by inheriting classes. |
| `_synced_commercial_fields` | internal rule | self | `base`, `product` | model | Returns the list of fields that are managed by the commercial entity to which a partner belongs. When modified on a children, update is propagated until the commercial entity. |
| `_get_commercial_values` | preparation rule | self | `base` |  | Get commercial values from record. Return only set values, as they are considered individually, and only set values should be taken into account. |
| `_get_synced_commercial_values` | preparation rule | self | `base` |  | Get synchronized commercial values from ercord. Return only set values as for other commercial values. |
| `_company_dependent_commercial_fields` | internal rule | self | `base` | model |  |
| `_commercial_sync_from_company` | internal rule | self | `base` |  | Handle sync of commercial fields when a new parent commercial entity is set, as if they were related fields |
| `_company_dependent_commercial_sync` | internal rule | self | `base` |  | Propagate sync of company dependant commercial fields to other commpanies. |
| `_commercial_sync_to_descendants` | internal rule | self, fields_to_sync | `base` |  | Handle sync of commercial fields to descendants |
| `_fields_sync` | internal rule | self, values | `base` |  | Sync commercial fields and address fields from company and to children. Also synchronize address to parent. This somehow mimics related fields to the parent, with more control. This method should be called after updating values in cache e.g. self should contain new values.  :param dict values: updated values, triggering sync |
| `_children_sync` | internal rule | self, values | `base` |  |  |
| `_handle_first_contact_creation` | internal rule | self | `base` |  | On creation of first contact for a company (or root) that has no address, assume contact address was meant to be company address |
| `_clean_website` | internal rule | self, website | `base` |  |  |
| `_compute_is_public` | computation | self | `base` |  |  |
| `write` | lifecycle override | self, vals | `account`, `base_geolocalize`, `base_vat`, `base`, `l10n_dk_nemhandel`, `l10n_in`, `mail_plugin`, `partnership`, `snailmail`, `website_sale` |  |  |
| `create` | lifecycle override | self, vals_list | `account_peppol`, `account`, `base_vat`, `base`, `l10n_dk_nemhandel`, `l10n_in`, `mail_plugin` | model_create_multi |  |
| `_unlink_except_user` | internal rule | self | `base` | ondelete |  |
| `_load_records_create` | internal rule | self, vals_list | `base` |  |  |
| `create_company` | operation | self | `base`, `l10n_it_edi` |  |  |
| `_create_contact_parent_company` | internal rule | self | `base_vat`, `base` |  |  |
| `open_commercial_entity` | operation | self | `base`, `point_of_sale` |  | Utility method used to add an "Open Company" button in partner views |
| `_compute_display_name` | computation | self | `base`, `im_livechat`, `l10n_tr_nilvera_einvoice_extended`, `website` | depends: `complete_name`, `email`, `vat`, `state_id`, `country_id`, `commercial_company_name`; depends_context: `show_address`, `partner_show_db_id`, `show_email`, `show_vat`, `lang`, `formatted_display_name`; depends_context: `im_livechat_hide_partner_company`; depends: `website_id`; depends_context: `display_website`; depends: `l10n_tr_tax_office_id` |  |
| `name_create` | lifecycle override | self, name | `base` | model | Override of orm's name_create method for partners. The purpose is to handle some basic formats to create partners using the name_create. If only an email address is received and that the regex cannot find a name, the name will have the email value. If 'force_email' key in context: must find the email address. |
| `find_or_create` | operation | self, email, assert_valid_email | `base`, `mail` | model | Find a partner with the given `email` or use :meth:`name_create` to create a new one.  :param str email: email-like string, which should contain at least one email,     e.g. `"Raoul Grosbedon <r.g@grosbedon.fr>"` :param bool assert_valid_email: raise if no valid email is found :return: newly created record |
| `address_get` | operation | self, adr_pref | `base` |  | Find contacts/addresses of the right type(s) by doing a depth-first-search through descendants within company boundaries (stop at entities flagged `is_company`) then continuing the search at the ancestors that are within the same company boundaries. Defaults to partners of type `'default'` when the exact type is not found, or to the provided partner itself if no type `'default'` is found either. |
| `view_header_get` | operation | self, view_id, view_type | `base` | model |  |
| `_get_default_address_format` | preparation rule | self | `base` | model |  |
| `_get_address_format` | preparation rule | self | `base`, `l10n_pl_edi`, `snailmail` | model |  |
| `_prepare_display_address` | preparation rule | self, without_company | `base` |  |  |
| `_display_address` | internal rule | self, without_company | `base` |  | The purpose of this function is to build and return an address formatted accordingly to the standards of the country where it belongs.  :param without_company: if address contains company :returns: the address formatted in a display that fit its country habits (or the default ones     if not country is specified) :rtype: string |
| `_display_address_depends` | internal rule | self | `base` |  |  |
| `get_import_templates` | operation | self | `base` | model |  |
| `_check_import_consistency` | validation | self, vals_list | `base` | model | The values created by an import are generated by a name search, field by field. As a result there is no check that the field values are consistent with each others. We check that if the state is given a value, it does belong to the given country, or we remove it. |
| `_get_country_name` | preparation rule | self | `base`, `snailmail` |  |  |
| `_get_all_addr` | preparation rule | self | `base`, `hr` |  |  |
| `_get_res_city_by_name` | preparation rule | self, name, country_id | `base_address_extended`, `base` | model |  |
| `_build_vcard` | internal rule | self | `web` |  | Build the partner's vCard. :returns a vobject.vCard object |
| `_get_vcard_file` | preparation rule | self | `web` |  |  |
| `_compute_is_in_call` | computation | self | `mail` | depends: `rtc_session_ids` |  |
| `search_for_channel_invite` | operation | self, search_term, channel_id, limit | `mail` | readonly; model | Returns partners matching search_term that can be invited to a channel.  - If `channel_id` is specified, only partners that can actually be invited to the channel   are returned (not already members, and in accordance to the channel configuration).  - If no matching partners are found and the search term is a valid email address,   then the method may return `selectable_email` as a fallback direct email invite, provided that   the channel allows invites by email. |
| `_search_for_channel_invite` | search rule | self, store, search_term, channel_id, limit | `mail` | readonly; model |  |
| `_search_for_channel_invite_to_store` | search rule | self, store, channel | `im_livechat`, `mail` |  |  |
| `get_mention_suggestions_from_channel` | operation | self, channel_id, search, limit | `mail` | readonly; model | Return 'limit'-first partners' such that the name or email matches a 'search' string. Prioritize partners that are also (internal) users, and then extend the research to all partners. Only members of the given channel are returned. The return format is a list of partner data (as per returned by `_to_store()`). |
| `_compute_contact_address_inline` | computation | self | `mail` | depends: `contact_address` | Compute an inline-friendly address based on contact_address. |
| `_compute_im_status` | computation | self | `hr_holidays`, `hr_homeworking`, `mail` | depends: `user_ids.manual_im_status`, `user_ids.presence_ids.status` |  |
| `_get_needaction_count` | preparation rule | self | `mail` |  | compute the number of needaction of the current partner |
| `_mail_get_partners` | messaging hook | self, introspect_fields | `mail` |  |  |
| `_get_view_cache_key` | preparation rule | self, view_id, view_type, **options | `mail` | model | Add context variable force_email in the key as _get_view depends on it. |
| `_find_or_create_from_emails` | internal rule | self, emails, ban_emails, filter_found, additional_values, no_create, sort_key, sort_reverse | `mail` | model | Based on a list of emails, find or (optionally) create partners. If an email is not unique (e.g. multi-email input), only the first found valid email in input is considered. Filter and sort options allow to tweak the way we link emails to partners (e.g. share partners only, ...).  Optional additional values allow to customize the created partner. Data are given per normalized email as it the creation criterion.  When an email is invalid but not void, it is used for search or create. It allows updating it afterwards e.g. with notifications resend which allows fixing typos / wrong emails.  :para |
| `_get_im_status_access_token` | preparation rule | self | `mail` |  | Return a scoped access token for the `im_status` field. The token is used in `ir_websocket._prepare_subscribe_data` to grant access to presence channels.  :rtype: str |
| `_get_mention_token` | preparation rule | self | `mail` |  | Return a scoped limited access token that indicates the current partner can be mentioned in messages.  :rtype: str |
| `_get_store_mention_fields` | preparation rule | self | `mail` |  |  |
| `_get_store_avatar_card_fields` | preparation rule | self, target | `hr`, `mail` |  |  |
| `_field_store_repr` | internal rule | self, field_name | `mail` |  |  |
| `_to_store_defaults` | internal rule | self, target | `hr_holidays`, `mail` |  |  |
| `get_mention_suggestions` | operation | self, search, limit | `mail` | readonly; model | Return 'limit'-first partners' such that the name or email matches a 'search' string. Prioritize partners that are also (internal) users, and then extend the research to all partners. The return format is a list of partner data (as per returned by `_to_store()`). |
| `_get_mention_suggestions_domain` | preparation rule | self, search | `mail` | model |  |
| `_search_mention_suggestions` | search rule | self, domain, limit, extra_domain | `mail` | model |  |
| `_get_current_persona` | preparation rule | self | `mail` | model |  |
| `_compute_product_pricelist` | computation | self | `product` | depends: `country_id`, `specific_property_product_pricelist`; depends_context: `company`, `country_code` |  |
| `_inverse_product_pricelist` | inverse computation | self | `product` |  |  |
| `_get_signup_url` | preparation rule | self | `auth_signup` |  |  |
| `_get_signup_url_for_action` | preparation rule | self, url, action, view_type, menu_id, res_id, model | `auth_signup` |  | generate a signup url for the given partner ids and action, possibly overriding the url state components (menu_id, id, view_type) |
| `action_signup_prepare` | user action | self | `auth_signup` |  |  |
| `signup_get_auth_param` | operation | self | `auth_signup` |  | Get a signup token related to the partner if signup is enabled. If the partner already has a user, get the login parameter. |
| `signup_cancel` | operation | self | `auth_signup` |  |  |
| `signup_prepare` | operation | self, signup_type | `auth_signup` |  | generate a new token for the partners with the given validity, if necessary |
| `_signup_retrieve_partner` | internal rule | self, token, check_validity, raise_exception | `auth_signup` | model | find the partner corresponding to a token, and possibly check its validity  :param token: the token to resolve :param bool check_validity: if True, also check validity :param bool raise_exception: if True, raise exception instead of returning False :return: partner (browse record) or False (if raise_exception is False) |
| `_signup_retrieve_info` | internal rule | self, token | `auth_signup` | model | retrieve the user info about the token  :rtype: dict \| None :return: a dictionary with the user information if the token is valid,     None otherwise:          db             the name of the database         token             the token, if token is valid         name             the name of the partner, if token is valid         login             the user login, if the user already exists         email             the partner email, if the user does not exist |
| `_get_login_date` | preparation rule | self | `auth_signup` |  |  |
| `_generate_signup_token` | internal rule | self, expiration | `auth_signup` |  | Generate the signup token for the partner in self.  Assume that :attr:`signup_type` is either `'signup'` or `'reset'`.  :param expiration: the time in hours before the expiration of the token :return: the signed payload/token that can be used to reset the          password/signup.  Since `last_login_date` is part of the payload, this token is invalidated as soon as the user logs in. |
| `_get_partner_from_token` | preparation rule | self, token | `auth_signup` | model |  |
| `_get_frontend_writable_fields` | preparation rule | self | `account_peppol`, `account`, `l10n_ar`, `l10n_br`, `l10n_it_edi`, `l10n_latam_base`, `l10n_my_edi`, `l10n_pe`, `l10n_sa_edi`, `portal`, `website_sale` | model | Define the fields a portal/public user can change on their contact and address records.  :rtype: set |
| `_can_edit_country` | internal rule | self | `account`, `portal`, `sale` |  | Can't edit `country_id` if there is (non draft) issued invoices. |
| `can_edit_vat` | operation | self | `account`, `portal`, `sale` |  | `vat` is a commercial field, synced between the parent (commercial entity) and the children. Only the commercial entity should be able to edit it (as in backend). |
| `_can_be_edited_by_current_customer` | internal rule | self, **kwargs | `delivery_mondialrelay`, `portal` |  | Return whether partner can be edited by current user. |
| `_get_current_partner` | preparation rule | self, **kwargs | `portal`, `website_sale` | model | Get main partner of the current user base on logged in user and kwargs. |
| `_get_delivery_address_domain` | preparation rule | self | `delivery`, `portal` |  |  |
| `_compute_fiscal_country_codes` | computation | self | `account` | depends: `company_id`, `country_code`; depends_context: `allowed_company_ids` |  |
| `_compute_fiscal_country_group_codes` | computation | self | `account` | depends: `company_id`; depends_context: `allowed_company_ids` |  |
| `_order` | internal rule | self | `account` |  |  |
| `_credit_debit_get` | internal rule | self | `account` | depends_context: `company` |  |
| `_compute_credit_to_invoice` | computation | self | `account`, `sale` | depends_context: `company` |  |
| `_asset_difference_search` | internal rule | self, account_type, operator, operand | `account` |  |  |
| `_credit_search` | internal rule | self, operator, operand | `account` | model |  |
| `_debit_search` | internal rule | self, operator, operand | `account` | model |  |
| `_invoice_total` | internal rule | self | `account` |  |  |
| `_compute_days_sales_outstanding` | computation | self | `account` | depends: `credit` |  |
| `_compute_available_invoice_template_pdf_report_ids` | computation | self | `account` |  |  |
| `_get_company_currency` | preparation rule | self | `account` |  |  |
| `_default_display_invoice_template_pdf_report_id` | preparation rule | self | `account` |  | Show PDF template selection if there are more than 1 template available for invoices. |
| `_compute_bank_count` | computation | self | `account` |  |  |
| `_compute_supplier_invoice_count` | computation | self | `account` |  |  |
| `_compute_invoice_edi_format` | computation | self | `account` | depends_context: `company`; depends: `country_code` |  |
| `_inverse_invoice_edi_format` | inverse computation | self | `account` |  |  |
| `_compute_use_partner_credit_limit` | computation | self | `account` | depends_context: `company` |  |
| `_inverse_use_partner_credit_limit` | inverse computation | self | `account` |  |  |
| `_compute_show_credit_limit` | computation | self | `account` | depends_context: `company` |  |
| `_get_account_statistics_count` | preparation rule | self | `account` |  |  |
| `_get_suggested_invoice_edi_format` | preparation rule | self | `account`, `l10n_dk_nemhandel`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_it_edi`, `l10n_pl_edi`, `l10n_tr_nilvera` |  |  |
| `_find_accounting_partner` | internal rule | self, partner | `account` |  | Find the partner for which the accounting entries will be created |
| `action_view_partner_invoices` | user action | self | `account` |  |  |
| `_has_invoice` | internal rule | self, partner_domain | `account` |  |  |
| `_unlink_if_partner_in_account_move` | internal rule | self | `account` | ondelete | Prevent the deletion of a partner "Individual", child of a company if: - partner in 'account.move' - state: all states (draft and posted) |
| `_increase_rank` | internal rule | self, field, n | `account` |  |  |
| `_check_vat` | validation | self, validation | `account`, `l10n_latam_base` |  |  |
| `_run_vat_checks` | background operation | self, country, vat, partner_name, validation | `account`, `base_vat` | model | Checks a VAT number syntactically to ensure its validity upon saving.  :param country: a country to check for :param vat: a string with the VAT number to check. :param partner_name: to put into the error message :param validation: if False, it will only return the formatted vat without checking if it valid.     if 'error', an incorrect number will raise and if 'setnull' it will just return an empty vat  :return: A two-elements tuple with:      1. The vat number     2. The country code of the country the VAT number was validated for, if it was validated.        False if it could not be validate |
| `_get_vat_required_valid` | preparation rule | self, company | `account`, `base_vat` |  | Hook for determining VAT validity with more complex VAT requirements. (like VIES) |
| `get_partner_localisation_fields_required_to_invoice` | operation | self, country_id | `account` | model | Returns the list of fields that needs to be filled when creating an invoice for the selected country. This is required for some flows that would allow a user to request an invoice from the portal. Using these, we can get their information and dynamically create form inputs based for the fields required legally for the company country_id. The returned fields must be of type ir.model.fields in order to handle translations  :param country_id: The country for which we want the fields. :return: an array of ir.model.fields for which the user should provide values. |
| `_import_retrieve_customer_from_vat` | internal rule | self, customer_values | `account` | model |  |
| `_get_country_specific_vat_variants` | preparation rule | self, normalized_vat, country_prefix | `account`, `base_vat` | model | Return additional formatted VAT values to consider during EDI partner matching. |
| `_import_retrieve_customer_from_bank_account_number` | internal rule | self, customer_values | `account` | model |  |
| `_import_retrieve_customer_from_phone` | internal rule | self, customer_values | `account` | model |  |
| `_import_retrieve_customer_from_email` | internal rule | self, customer_values | `account` | model |  |
| `_import_retrieve_customer_from_name` | internal rule | self, customer_values | `account` | model |  |
| `_import_retrieve_customer` | internal rule | self, search_plan, company, customer_values_list | `account` | model |  |
| `_retrieve_partner_with_vat` | internal rule | self, vat, extra_domain | `account` | model |  |
| `_retrieve_partner_with_phone_email` | internal rule | self, phone, email, extra_domain | `account` | model |  |
| `_retrieve_partner_with_name` | internal rule | self, name, extra_domain | `account` | model |  |
| `_retrieve_partner` | internal rule | self, name, phone, email, vat, domain, company | `account` |  | Search all partners and find one that matches one of the parameters. :param name:    The name of the partner. :param phone:   The phone or mobile of the partner. :param mail:    The mail of the partner. :param vat:     The vat number of the partner. :param domain:  An extra domain to apply. :param company: The company of the partner. :returns:       A partner or an empty recordset if not found. |
| `_merge_method` | internal rule | self, destination, source | `account` |  | Prevent merging partners that are linked to already hashed journal items. |
| `_deduce_country_code` | internal rule | self | `account`, `l10n_it_edi`, `l10n_no`, `l10n_sg` |  | deduce the country code based on the information available. we have three cases: - country_code is BE but the VAT number starts with FR, the country code is FR, not BE - if a country-specific field is set (e.g. the codice_fiscale), that country is used for the country code - if the VAT number has no ISO country code, use the country_code in that case. |
| `_compute_partner_vat_placeholder` | computation | self | `account` | depends: `country_id` |  |
| `_compute_partner_company_registry_placeholder` | computation | self | `account` | depends: `country_id` | Provides a dynamic placeholder on the company registry field for countries that may need it. Add your country and the value you want in the _ref_company_registry map. |
| `_compute_account_move_count` | computation | self | `account` |  |  |
| `action_open_business_doc` | user action | self | `account` |  |  |
| `_clear_removed_edi_formats` | internal rule | self, *formats | `account` | model | Helper to clear outdated EDI formats.  Usually called as an uninstall hook of modules that add these formats. It avoids the form view to become unusable after module uninstallation. |
| `_check_peppol_fields` | validation | self | `account_edi_ubl_cii` | constrains: `peppol_endpoint` |  |
| `_get_ubl_cii_formats` | preparation rule | self | `account_edi_ubl_cii` | model |  |
| `_get_ubl_cii_formats_info` | preparation rule | self | `account_edi_ubl_cii`, `l10n_anz_ubl_pint`, `l10n_dk_nemhandel`, `l10n_dk_oioubl`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_jp_ubl_pint`, `l10n_my_ubl_pint`, `l10n_ro_edi`, `l10n_sg_ubl_pint`, `l10n_tr_nilvera` | model |  |
| `_get_ubl_cii_formats_by_country` | preparation rule | self | `account_edi_ubl_cii` | model |  |
| `_get_suggested_ubl_cii_edi_format` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_ubl_cii_edi_format` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_suggested_peppol_edi_format` | preparation rule | self | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_get_peppol_edi_format` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_peppol_formats` | preparation rule | self | `account_edi_ubl_cii` | model |  |
| `_peppol_eas_endpoint_depends` | internal rule | self | `account_edi_ubl_cii`, `l10n_it_edi`, `l10n_no`, `l10n_sg` |  |  |
| `_compute_is_ubl_format` | computation | self | `account_edi_ubl_cii` | depends_context: `company`; depends: `invoice_edi_format` |  |
| `_compute_is_peppol_edi_format` | computation | self | `account_edi_ubl_cii` | depends_context: `company`; depends: `invoice_edi_format` |  |
| `_get_peppol_endpoint_value` | preparation rule | self, country_code, field, eas | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_compute_peppol_endpoint` | computation | self | `account_edi_ubl_cii`, `account_peppol` | depends: `peppol_eas` | If the EAS changes and a valid endpoint is available, set it. Otherwise, keep the existing value. |
| `_compute_peppol_eas` | computation | self | `account_edi_ubl_cii`, `account_peppol` | depends: | If the country_code changes, recompute the EAS only if there is a country_code, it exists in the EAS_MAPPING, and the current EAS is not consistent with the new country_code. |
| `_compute_available_peppol_eas` | computation | self | `account_edi_ubl_cii`, `account_peppol` | depends_context: `company`; depends: `company_id`, `peppol_eas`; depends: `peppol_eas` |  |
| `_build_error_peppol_endpoint` | internal rule | self, eas, endpoint | `account_edi_ubl_cii`, `l10n_fr_pdp` |  | This function contains all the rules regarding the peppol_endpoint. |
| `_get_edi_builder` | preparation rule | self, invoice_edi_format | `account_edi_ubl_cii`, `l10n_anz_ubl_pint`, `l10n_dk_nemhandel`, `l10n_dk_oioubl`, `l10n_fr_pdp`, `l10n_hr_edi`, `l10n_jp_ubl_pint`, `l10n_my_ubl_pint`, `l10n_ro_edi`, `l10n_sg_ubl_pint`, `l10n_tr_nilvera` | model |  |
| `_import_retrieve_customer_from_eas_endpoint` | internal rule | self, customer_values | `account_edi_ubl_cii` | model |  |
| `_compute_payment_token_count` | computation | self | `payment` | depends: `payment_token_ids` |  |
| `_onchange_verify_peppol_status` | on change | self | `account_peppol` | onchange: `invoice_edi_format`, `peppol_endpoint`, `peppol_eas` |  |
| `_compute_available_peppol_sending_methods` | computation | self | `account_peppol` | depends_context: `company`; depends: `company_id` |  |
| `_compute_available_peppol_edi_formats` | computation | self | `account_peppol` | depends_context: `company`; depends: `invoice_sending_method` |  |
| `_log_verification_state_update` | internal rule | self, company, old_value, new_value | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_get_participant_info` | preparation rule | self, edi_identification | `account_peppol` | model |  |
| `_check_peppol_participant_exists` | validation | self, participant_info, edi_identification | `account_peppol` | model |  |
| `_peppol_lookup_participant` | internal rule | self, edi_identification | `account_peppol` | model | NAPTR DNS peppol participant lookup through the system's Peppol proxy |
| `_check_document_type_support` | validation | self, participant_info, ubl_cii_format, process_type | `account_peppol` |  |  |
| `_update_peppol_state_per_company` | internal rule | self, vals | `account_peppol` |  |  |
| `button_account_peppol_check_partner_endpoint` | user action | self, company | `account_peppol_response`, `account_peppol`, `l10n_fr_pdp` |  | A basic check for whether a participant is reachable at the given Peppol participant ID - peppol_eas:peppol_endpoint (ex: '9999:test') The SML (Service Metadata Locator) assigns a DNS name to each peppol participant. This DNS name resolves into the SMP (Service Metadata Publisher) of the participant. The DNS address is of the following form: strip-trailing(base32(sha256(lowercase(ID-VALUE))),"=") + "." + ID-SCHEME + "." + SML-ZONE-NAME The lookup should be done on NAPTR DNS from 2025-11-01 (ref:https://peppol.helger.com/public/locale-en_US/menuitem-docs-doc-exchange) |
| `_get_peppol_verification_state` | preparation rule | self, peppol_endpoint, peppol_eas, invoice_edi_format, process_type | `account_peppol`, `l10n_fr_pdp` | model |  |
| `_get_partners_to_skip_peppol_computation` | preparation rule | self | `account_peppol` |  |  |
| `_get_peppol_proxy_identification_info` | preparation rule | self, peppol_eas, peppol_endpoint | `account_peppol`, `l10n_fr_pdp` | model |  |
| `_compute_response_support` | computation | self | `account_peppol_response` | depends: `peppol_supported_documents`, `peppol_verification_state` |  |
| `_peppol_fill_participant_supported_documents` | internal rule | self | `account_peppol_response`, `l10n_fr_pdp` |  |  |
| `_get_backend_root_menu_ids` | preparation rule | self | `contacts` |  |  |
| `_inverse_street_data` | inverse computation | self | `base_address_extended` |  | update self.street based on street_name, street_number and street_number2 |
| `_compute_street_data` | computation | self | `base_address_extended` | depends: `street` | Splits street value into sub-fields. Recomputes the fields of STREET_FIELDS when `street` of a partner is updated |
| `_onchange_city_id` | on change | self | `base_address_extended` | onchange: `city_id` |  |
| `_onchange_phone_validation` | on change | self | `phone_validation` | onchange: `phone`, `country_id`, `company_id` |  |
| `_geo_localize` | internal rule | self, street, zip, city, state, country | `base_geolocalize` | model |  |
| `geo_localize` | operation | self | `base_geolocalize` |  |  |
| `_inverse_vat` | inverse computation | self | `base_vat` |  |  |
| `_onchange_vat` | on change | self | `base_vat`, `l10n_latam_base` | onchange: `vat`, `country_id`; onchange: `vat`, `country_id`, `l10n_latam_identification_type_id` |  |
| `_compute_perform_vies_validation` | computation | self | `base_vat` | depends_context: `company`; depends: `vat` | Determine whether to show VIES validity on the current VAT number |
| `_compute_vies_valid` | computation | self | `base_vat` | depends: `vat` | Check the VAT number with VIES, if enabled. |
| `_split_vat` | internal rule | self, vat | `base_vat` |  |  |
| `_get_iap_vies_credentials` | preparation rule | self | `base_vat` | model | Return a couple (identifier, token) that is going to identify this db to IAP such that only this one can request updates on a previously asked VIES check. If they exist, we simply return them. If they don't, we create them in another cursor to avoid the current transaction to be rolled back after in case of an uncaucht error while the credentials have been registered on IAP. |
| `_get_iap_vies_endpoint` | preparation rule | self | `base_vat` | model |  |
| `_check_vies_iap` | validation | self | `base_vat` |  | Called when VAT is manually edited to query IAP for the validity of the VAT |
| `_cron_check_vies_iap` | background operation | self | `base_vat` | model | Called by cron to check if IAP has any update on a previously requested VAT that was pending |
| `_check_vies_update_iap` | validation | self | `base_vat` |  | Calls IAP for an update of a previously requested VAT validity |
| `_update_vies_status` | internal rule | self, status | `base_vat` |  |  |
| `_check_vat_number` | validation | self, country_code, vat_number | `base_vat` | model | Low-level method directly calling stdnum or our own specific method. |
| `_build_vat_error_message` | internal rule | self, country_code, wrong_vat, record_label | `base_vat` | model |  |
| `check_vat_al` | operation | self, vat | `base_vat` |  | Check Albania VAT number |
| `check_vat_jp` | operation | self, vat | `base_vat` |  |  |
| `check_vat_do` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ro` | operation | self, vat | `base_vat` |  | Check Romanian VAT number that can be for example 'RO1234567897 or 'xyyzzaabbxxxx' or '9000xxxxxxxx'.  - For xyyzzaabbxxxx, 'x' can be any number, 'y' is the two last digit of a year (in the range 00…99),   'a' is a month, b is a day of the month, the number 8 and 9 are Country or district code   (For those twos digits, we decided to let some flexibility  to avoid complexifying the regex and also   for maintainability) - 9000xxxxxxxx, start with 9000 and then is filled by number In the range 0...9  Also stdum also checks the CUI or CIF (Romanian company identifier). So a number like '123456897 |
| `check_vat_gr` | operation | self, vat | `base_vat` |  | Allows some custom test VAT number to be valid to allow testing Greece EDI. |
| `check_vat_gt` | operation | self, vat | `base_vat` |  | Allow some custom Guatemala NIT numbers to pass the test to be used for testing the Guatemalan EDI. |
| `check_vat_hu` | operation | self, vat | `base_vat` |  | Check Hungary VAT number that can be for example 'HU12345676 or 'xxxxxxxx-y-zz' or '8xxxxxxxxy'  - For xxxxxxxx-y-zz, 'x' can be any number, 'y' is a number between 1 and 5 depending on the person and the 'zz'   is used for region code. - 8xxxxxxxxy, Tin number for individual, it has to start with an 8 and finish with the check digit - In case of EU format it will be the first 8 digits of the full VAT |
| `check_vat_ch` | operation | self, vat | `base_vat` |  | Check Switzerland VAT number. |
| `is_valid_ruc_ec` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ec` | operation | self, vat | `base_vat` |  |  |
| `_ie_check_char` | internal rule | self, vat | `base_vat` |  |  |
| `check_vat_ie` | operation | self, vat | `base_vat` |  |  |
| `check_vat_mx` | operation | self, vat | `base_vat` |  | Mexican VAT verification  Verificar RFC México |
| `check_vat_no` | operation | self, vat | `base_vat` |  | Check Norway VAT number.See http://www.brreg.no/english/coordination/number.html |
| `check_vat_pe` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ph` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ru` | operation | self, vat | `base_vat` |  | Check Russia VAT number. Method copied from vatnumber 1.2 lib https://code.google.com/archive/p/vatnumber/ |
| `check_vat_rs` | operation | self, vat | `base_vat` |  |  |
| `check_vat_tr` | operation | self, vat | `base_vat`, `l10n_tr_nilvera_base_vat` |  |  |
| `check_vat_sa` | operation | self, vat | `base_vat` |  | Check company VAT TIN according to ZATCA specifications: The VAT number should start and begin with a '3' and be 15 digits long |
| `check_vat_ua` | operation | self, vat | `base_vat` |  |  |
| `check_vat_uy` | operation | self, vat | `base_vat` |  | Taken from python-stdnum's master branch, as the release doesn't handle RUT numbers starting with 22. origin https://github.com/arthurdejong/python-stdnum/blob/master/stdnum/uy/rut.py FIXME Can be removed when python-stdnum does a new release. |
| `check_vat_uz` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ve` | operation | self, vat | `base_vat` |  |  |
| `check_vat_in` | operation | self, vat | `base_vat`, `l10n_in` |  | This TEST_GST_NUMBER is used as test credentials for EDI but this is not a valid number as per the regular expression so TEST_GST_NUMBER is considered always valid |
| `check_vat_br` | operation | self, vat | `base_vat` |  |  |
| `check_vat_cr` | operation | self, vat | `base_vat` |  |  |
| `check_vat_vn` | operation | self, vat | `base_vat` |  | VAT format validator for Vietnam. Supported formats: - 10-digit format (Enterprise tax ID): e.g., 0101243150 - 13-digit format with branch suffix: e.g., 0101243150-001 - 12-digit format (Personal ID / Citizen ID - CCCD): e.g., 079123456789 (used as tax ID for individuals from July 1st, 2025)  Note: - stdnum.vn.mst.validate() currently only supports 10- and 13-digit VAT numbers - and does not accept the 12-digit personal tax ID (CCCD) format introduced from 01/07/2025. - This helper provides a lightweight format-level validator for use in the meantime. - Can be removed once stdnum.vn.mst adds C |
| `format_vat_al` | operation | self, vat | `base_vat` |  |  |
| `format_vat_eu` | operation | self, vat | `base_vat` |  |  |
| `format_vat_ch` | operation | self, vat | `base_vat` |  |  |
| `format_vat_cl` | operation | self, vat | `base_vat` |  | It is better to always have the - |
| `format_vat_co` | operation | self, vat | `base_vat` |  | It is better to always have the - |
| `format_vat_vn` | operation | self, vat | `base_vat` |  | It is better to always have the - |
| `format_vat_hu` | operation | self, vat | `base_vat` |  | We put the - back as we require it for the EDI and the different parts will make it clear to the user |
| `format_vat_is` | operation | self, vat | `base_vat` |  |  |
| `check_vat_id` | operation | self, vat | `base_vat` |  | Temporary Indonesian VAT validation to support the new format introduced in January 2024. |
| `check_vat_th` | operation | self, vat | `base_vat` |  |  |
| `check_vat_de` | operation | self, vat | `base_vat` |  |  |
| `check_vat_il` | operation | self, vat | `base_vat` |  |  |
| `check_vat_ma` | operation | self, vat | `base_vat` |  |  |
| `format_vat_sm` | operation | self, vat | `base_vat` |  |  |
| `check_vat_tw` | operation | self, vat | `base_vat` |  | Since Feb. 2025, due to the imminent exhaustion of the UBN numbers, the validation logic was changed from using a division by 10 for the final check to using a division by 5, making numbers that were previously invalid now valid.  The stdnum implementation of the VAT validation is not up to date with this latest update, so we implement our own validation to support these new valid UBNs. |
| `_format_vat_number` | internal rule | self, country_code, vat | `base_vat` | model | Low-level method directly calling stdnum or our own specific method returning the formatted VAT. |
| `_convert_hu_local_to_eu_vat` | internal rule | self, local_vat | `base_vat` | model |  |
| `_compute_meeting_count` | computation | self | `calendar` |  |  |
| `_compute_meeting` | computation | self | `calendar` |  |  |
| `get_attendee_detail` | operation | self, meeting_ids | `calendar` |  | Return a list of dict of the given meetings with the attendees details Used by:  - many2many_attendee.js: Many2ManyAttendee - calendar_model.js (calendar.CalendarModel) |
| `_creation_message` | internal rule | self | `calendar` |  |  |
| `_set_calendar_last_notif_ack` | internal rule | self | `calendar` | model |  |
| `schedule_meeting` | operation | self | `calendar` |  |  |
| `_get_busy_calendar_events` | preparation rule | self, start_datetime, end_datetime | `calendar` |  | Get a mapping from partner id to attended events intersecting with the time interval.  :rtype: dict[int, <calendar.event>] |
| `_fetch_children_partners_for_hierarchy` | internal rule | self | `crm` |  |  |
| `_get_contact_opportunities_domain` | preparation rule | self | `crm`, `website_crm_partner_assign` |  |  |
| `_compute_opportunity_count` | computation | self | `crm`, `website_crm_partner_assign` |  |  |
| `action_view_opportunity` | user action | self | `crm` |  |  |
| `_compute_user_livechat_username` | computation | self | `im_livechat` | depends: `user_ids.livechat_username` |  |
| `_compute_livechat_channel_count` | computation | self | `im_livechat` |  |  |
| `_get_store_livechat_username_fields` | preparation rule | self | `im_livechat` |  | Return the fields to be stored for live chat username. |
| `_bus_send_history_message` | internal rule | self, channel, page_history | `im_livechat` |  |  |
| `action_view_livechat_sessions` | user action | self | `im_livechat` |  |  |
| `_compute_partner_iap_info` | computation | self | `mail_plugin` |  |  |
| `_get_sale_order_domain_count` | preparation rule | self | `sale` | model |  |
| `_compute_sale_order_count` | computation | self | `sale` |  |  |
| `_has_order` | internal rule | self, partner_domain | `sale` |  |  |
| `action_view_stock_serial` | user action | self | `stock` |  |  |
| `_compute_is_mondialrelay` | computation | self | `delivery_mondialrelay` | depends: `ref` |  |
| `_mondialrelay_search_or_create` | internal rule | self, data | `delivery_mondialrelay` | model |  |
| `_compute_event_count` | computation | self | `event` |  |  |
| `_compute_static_map_url` | computation | self | `event` | depends: `zip`, `city`, `country_id`, `street` |  |
| `_compute_static_map_url_is_valid` | computation | self | `event` | depends: `static_map_url` | Compute whether the link is valid.  This should only remain valid for a relatively short time. Here, for the duration it is in cache. |
| `action_event_view` | user action | self | `event` |  |  |
| `_google_map_signed_img` | internal rule | self, zoom, width, height | `event` |  | Create a signed static image URL for the location of this partner. |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `google_address_autocomplete`, `partner_autocomplete` | model |  |
| `_compute_employees_count` | computation | self | `hr` |  |  |
| `action_open_employees` | user action | self | `hr` |  |  |
| `_compute_employee` | computation | self | `hr` | depends: `employee_ids` |  |
| `_unlink_contact_rel_employee` | internal rule | self | `hr` | ondelete |  |
| `_action_show` | internal rule | self | `hr` |  | If self is a singleton, directly access the form view. If it is a recordset, open a list view |
| `_get_employees_from_attendees` | preparation rule | self, everybody | `hr_calendar` |  |  |
| `_get_schedule` | preparation rule | self, start_period, stop_period, everybody, merge | `hr_calendar` |  | This method implements the general case where employees might have different resource calendars at different times, even though this is not the case with only this module installed. This way it will work with these other modules by just overriding `_get_calendar_periods`.  :param datetime start_period: the start of the period :param datetime stop_period: the stop of the period :param boolean everybody: represents the "everybody" filter on calendar :param boolean merge: specifies if calendar's work_intervals needs to be merged :return: schedule (merged or not) by partner :rtype: defaultdict |
| `get_working_hours_for_all_attendees` | operation | self, attendee_ids, date_from, date_to, everybody | `hr_calendar` | model |  |
| `_interval_to_business_hours` | internal rule | self, working_intervals | `hr_calendar` |  |  |
| `_compute_leave_date_to` | computation | self | `hr_holidays` |  |  |
| `_get_on_leave_ids` | preparation rule | self | `hr_holidays` | model |  |
| `get_worklocation` | operation | self, start_date, end_date | `hr_homeworking_calendar` |  |  |
| `_compute_certifications_count` | computation | self | `survey` | depends: `is_company` |  |
| `_compute_certifications_company_count` | computation | self | `survey` | depends: `is_company`, `child_ids.certifications_count` |  |
| `action_view_certifications` | user action | self | `survey` |  |  |
| `google_map_img` | operation | self, zoom, width, height | `website` |  |  |
| `google_map_link` | operation | self, zoom | `website` |  |  |
| `_compute_website_url` | computation | self | `website_partner` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `website_partner` |  |  |
| `_compute_slide_channel_values` | computation | self | `website_slides` |  |  |
| `_search_slide_channel_completed_ids` | search rule | self, operator, value | `website_slides` |  |  |
| `_search_slide_channel_ids` | search rule | self, operator, value | `website_slides` |  |  |
| `_compute_slide_channel_company_count` | computation | self | `website_slides` | depends: `is_company`, `child_ids.slide_channel_count` |  |
| `action_view_courses` | user action | self | `website_slides` |  | View partners courses. In singleton mode, return courses followed by all its contacts (if company) or by themselves (if not a company). Otherwise simply set a domain on required partners. The courses to which the partner(s) is not enrolled (e.g. invited) are not shown. |
| `_ensure_same_company_than_projects` | validation | self | `project` | constrains: `company_id`, `project_ids` |  |
| `_ensure_same_company_than_tasks` | validation | self | `project` | constrains: `company_id`, `task_ids` |  |
| `_compute_task_count` | computation | self | `project` |  |  |
| `_create_portal_users` | internal rule | self | `project` |  |  |
| `action_view_tasks` | user action | self | `project` |  |  |
| `_iap_replace_location_codes` | internal rule | self, iap_data | `partner_autocomplete` | model |  |
| `_iap_replace_industry_code` | internal rule | self, iap_data | `partner_autocomplete` | model |  |
| `_iap_replace_language_codes` | internal rule | self, iap_data | `partner_autocomplete` | model |  |
| `_format_data_company` | internal rule | self, iap_data | `partner_autocomplete` | model |  |
| `autocomplete_by_name` | operation | self, query, query_country_id, timeout | `partner_autocomplete` | model |  |
| `autocomplete_by_vat` | operation | self, vat, query_country_id, timeout | `partner_autocomplete` | model |  |
| `_process_enriched_response` | background operation | self, response, error | `partner_autocomplete` | model |  |
| `_validate_partner_autocomplete_response` | internal rule | self, autocomplete_response | `partner_autocomplete` | model |  |
| `enrich_by_duns` | operation | self, duns, timeout | `partner_autocomplete` | model |  |
| `enrich_by_gst` | operation | self, gst, timeout | `partner_autocomplete` | model |  |
| `enrich_by_domain` | operation | self, domain, timeout | `partner_autocomplete` | model |  |
| `iap_partner_autocomplete_get_tag_ids` | operation | self, unspsc_codes | `partner_autocomplete` |  | Called by JS to create the activity tags from the UNSPSC codes |
| `enrich_company_message_post` | operation | self, data | `partner_autocomplete` |  | Post a chatter note containing company enrichment data received from IAP |
| `_compute_pos_contact_address` | computation | self | `point_of_sale` | depends: |  |
| `get_new_partner` | operation | self, config_id, domain, offset | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `l10n_vn_edi_viettel_pos`, `point_of_sale`, `pos_hr` | model |  |
| `_compute_fiscal_position_id` | computation | self | `point_of_sale` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_ar_pos`, `l10n_in_pos`, `l10n_pe_pos`, `point_of_sale`, `pos_sale` | model |  |
| `_compute_pos_order` | computation | self | `point_of_sale` |  |  |
| `_compute_invoice_emails` | computation | self | `point_of_sale` | depends: `email`, `child_ids.type`, `child_ids.email` |  |
| `action_view_pos_order` | user action | self | `point_of_sale` |  | This function returns an action that displays the pos orders from partner. |
| `_unlink_if_pos_no_orders` | internal rule | self | `point_of_sale` | ondelete |  |
| `_run_check_identification` | background operation | self, validation | `l10n_ar`, `l10n_cl`, `l10n_ec`, `l10n_latam_base`, `l10n_uy` |  | Since we validate more documents than the vat for Argentinean partners (CUIT - VAT AR, CUIL, DNI) we extend this method in order to process it. |
| `_compute_l10n_ar_formatted_vat` | computation | self | `l10n_ar` | depends: `l10n_ar_vat` | This will add some dash to the CUIT number (VAT AR) in order to show in his natural format: {person_category}-{number}-{validation_number} |
| `_compute_l10n_ar_vat` | computation | self | `l10n_ar` | depends: `vat`, `l10n_latam_identification_type_id` | We add this computed field that returns cuit (VAT AR) or nothing if this one is not set for the partner. This Validation can be also done by calling ensure_vat() method that returns the cuit (VAT AR) or error if this one is not found |
| `_check_l10n_ar_cuit_number` | validation | self | `l10n_ar` | constrains: `vat`, `l10n_latam_identification_type_id`, `country_id` |  |
| `ensure_vat` | operation | self | `l10n_ar` |  | This method is a helper that returns the VAT number is this one is defined if not raise an UserError.  VAT is not mandatory field but for some Argentinean operations the VAT is required, for eg  validate an electronic invoice, build a report, etc.  This method can be used to validate is the VAT is proper defined in the partner |
| `_get_validation_module` | preparation rule | self | `l10n_ar` |  |  |
| `_l10n_ar_identification_validation` | internal rule | self | `l10n_ar` |  |  |
| `_get_id_number_sanitize` | preparation rule | self | `l10n_ar` |  | Sanitize the identification number. Return the digits/integer value of the identification number If not vat number defined return 0 |
| `_ar_unlink_except_master_data` | internal rule | self | `l10n_ar_pos` | ondelete |  |
| `_onchange_property_product_pricelist` | on change | self | `website_sale` | onchange: `property_product_pricelist` |  |
| `_get_order_fiscal_position_recompute_domain` | preparation rule | self | `website_sale` |  | Return a domain of sale orders for which we should recompute fiscal position after address update. |
| `_format_dotted_vat_cl` | internal rule | self, vat | `l10n_cl` |  |  |
| `_compute_purchase_order_count` | computation | self | `purchase` |  |  |
| `_compute_nemhandel_identifier_type` | computation | self | `l10n_dk_nemhandel` | depends: `country_code`, `vat`, `company_registry` |  |
| `_compute_nemhandel_identifier_value` | computation | self | `l10n_dk_nemhandel` | depends: `country_code`, `vat`, `company_registry`, `nemhandel_identifier_type` |  |
| `_compute_is_using_nemhandel` | computation | self | `l10n_dk_nemhandel` | depends_context: `allowed_company_ids`; depends: `invoice_edi_format` |  |
| `_check_nemhandel_send_oioubl` | validation | self | `l10n_dk_nemhandel` | constrains: `invoice_edi_format`, `invoice_sending_method` |  |
| `_get_nemhandel_participant_info` | preparation rule | self, edi_identification | `l10n_dk_nemhandel` | model |  |
| `_nemhandel_lookup_participant` | internal rule | self, edi_identification | `l10n_dk_nemhandel` | model | NAPTR DNS nemhandel participant lookup through the system's Nemhandel proxy |
| `_l10n_dk_nemhandel_log_verification_state_update` | internal rule | self, company, old_value, new_value | `l10n_dk_nemhandel` |  |  |
| `_check_nemhandel_participant_exists` | validation | self, participant_info, edi_identification | `l10n_dk_nemhandel` | model |  |
| `_update_nemhandel_state_per_company` | internal rule | self, vals | `l10n_dk_nemhandel` |  |  |
| `button_nemhandel_check_partner_endpoint` | user action | self, company | `l10n_dk_nemhandel_response`, `l10n_dk_nemhandel` |  | A basic check for whether a participant is reachable at the given identifier_type and identifier_value |
| `_get_nemhandel_verification_state` | preparation rule | self, invoice_edi_format | `l10n_dk_nemhandel` |  |  |
| `_compute_nemhandel_response_support` | computation | self | `l10n_dk_nemhandel_response` | depends: `nemhandel_supported_documents`, `nemhandel_verification_state` |  |
| `_nemhandel_fill_participant_supported_documents` | internal rule | self | `l10n_dk_nemhandel_response` |  |  |
| `_compute_l10n_ec_vat_validation` | computation | self | `l10n_ec` | depends: `vat`, `country_id`, `l10n_latam_identification_type_id` |  |
| `_l10n_ec_get_identification_type` | internal rule | self | `l10n_ec` |  | Maps the system identification types to Ecuadorian ones. Useful for document type domains, electronic documents, ats, others. |
| `_l10n_es_is_foreign` | internal rule | self | `l10n_es` |  |  |
| `_l10n_es_edi_get_partner_info` | internal rule | self | `l10n_es` |  | Used in SII and Veri*factu |
| `_validate_l10n_es_edi_facturae_ac_physical_gln` | validation | self | `l10n_es_edi_facturae` | constrains: `l10n_es_edi_facturae_ac_physical_gln` |  |
| `_validate_l10n_es_edi_facturae_ac_logical_operational_point` | validation | self | `l10n_es_edi_facturae` | constrains: `l10n_es_edi_facturae_ac_logical_operational_point` |  |
| `_compute_l10n_es_edi_facturae_residence_type` | computation | self | `l10n_es_edi_facturae` | depends: `country_id` |  |
| `_l10n_es_edi_facturae_export_check` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_l10n_es_edi_verifactu_get_values` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_compute_l10n_fr_is_french` | computation | self | `l10n_fr` | depends: `country_code` |  |
| `fields_get` | lifecycle override | self, allfields, attributes | `l10n_fr_pdp` | model |  |
| `_compute_pdp_verification_display_state` | computation | self | `l10n_fr_pdp` | depends: `peppol_verification_state`, `peppol_endpoint`, `peppol_eas`; depends_context: `company` |  |
| `_check_pdp_send_ubl_21_fr` | validation | self | `l10n_fr_pdp` | constrains: `invoice_edi_format`, `invoice_sending_method` |  |
| `_l10n_fr_pdp_is_b2c` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_siren` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_base_identifier` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_get_suggested_pdp_identifier` | preparation rule | self | `l10n_fr_pdp` |  |  |
| `_get_pdp_display_verification_state` | preparation rule | self, state | `l10n_fr_pdp` |  |  |
| `_get_pdp_annuaire_verification_state` | preparation rule | self, edi_identification, invoice_edi_format | `l10n_fr_pdp` | model |  |
| `_pdp_annuaire_lookup_participant` | internal rule | self, edi_identification | `l10n_fr_pdp` | model |  |
| `_get_pdp_receiver_identification_info` | preparation rule | self | `l10n_fr_pdp` |  |  |
| `_auto_init` | lifecycle override | self | `l10n_gr_edi` |  |  |
| `_compute_l10n_gr_edi_branch_number` | computation | self | `l10n_gr_edi` | depends: `country_code` |  |
| `_check_mojeracun_send_ubl_hr` | validation | self | `l10n_hr_edi` | constrains: `invoice_edi_format`, `invoice_sending_method` |  |
| `_compute_l10n_hu_eu_vat` | computation | self | `l10n_hu` | depends: `vat` |  |
| `_run_vies_test` | background operation | self, vat_number, default_country | `l10n_hu_edi` | model | Convert back the hungarian format to EU format: 12345678-1-12 => HU12345678 |
| `_compute_l10n_id_pkp` | computation | self | `l10n_id_efaktur_coretax` | depends: `vat`, `country_code` |  |
| `_compute_l10n_in_gst_state_warning` | computation | self | `l10n_in` | depends: `vat`, `state_id`, `country_id`, `fiscal_country_codes` |  |
| `_compute_display_pan_warning` | computation | self | `l10n_in` | depends: `l10n_in_pan_entity_id` |  |
| `_compute_l10n_in_gst_registered_and_status` | computation | self | `l10n_in` | depends: `company_id.l10n_in_is_gst_registered`, `company_id.l10n_in_gstin_status_feature` |  |
| `_onchange_l10n_in_gst_status` | on change | self | `l10n_in` | onchange: `vat` | Reset GST Status Whenever the `vat` of partner changes |
| `_set_l10n_in_pan_tan_from_vat` | internal rule | self | `l10n_in` |  |  |
| `_l10n_in_search_create_pan_entity_from_vat` | internal rule | self, vat | `l10n_in` |  |  |
| `action_l10n_in_verify_gstin_status` | user action | self | `l10n_in` |  |  |
| `onchange_vat` | on change | self | `l10n_in` | onchange: `vat` |  |
| `_l10n_in_get_partner_vals_by_vat` | internal rule | self, vat | `l10n_in` | model |  |
| `action_update_state_as_per_gstin` | user action | self | `l10n_in` |  |  |
| `_l10n_in_edi_strict_error_validation` | internal rule | self | `l10n_in_edi` |  | This method is used to check the strict validation of the partner data as per government API json schema (https://einv-apisandbox.nic.in/version1.03/generate-irn.html#requestSampleJSON) In case of any error, it will return the error message Note - We stimulate as error message from API, so that user can understand the error Also restrict unwanted request to government servers and avoid getting black listed |
| `_l10n_in_check_einvoice_validation` | internal rule | self | `l10n_in_edi` |  |  |
| `_compute_on_time_rate` | computation | self | `purchase_stock` | depends: `purchase_line_ids` |  |
| `_l10n_it_edi_is_public_administration` | internal rule | self | `l10n_it_edi` |  | Returns True if the destination of the FatturaPA belongs to the Public Administration. |
| `_l10n_it_edi_get_values` | internal rule | self | `l10n_it_edi` |  | Generates all partner values needed by l10n_it_edi XML export.  VAT number: If there is a VAT number and the partner is not in EU, then we use the VAT number as is,     as an alphanumeric value identifying the counterparty, up to a maximum of     28 alphanumeric characters, on which the SdI does not perform validity checks. If there is a VAT number and the partner is in EU, then remove the country prefix If there is no VAT and the partner is not in EU, then the exported value is 'OO99999999999' If there is no VAT and the partner is in EU, then the exported value is '0000000' If there is no VAT |
| `_l10n_it_edi_normalized_codice_fiscale` | internal rule | self, l10n_it_codice_fiscale | `l10n_it_edi` |  | Normalize the Italian Tax Code for export. If the Tax Code is equal to the Italian VAT, it may mistakenly have the country prefix, so we try and remove it if we can |
| `_l10n_it_onchange_vat` | on change | self | `l10n_it_edi` | onchange: `vat`, `country_id` |  |
| `validate_codice_fiscale` | validation | self | `l10n_it_edi` | constrains: `l10n_it_codice_fiscale` |  |
| `_l10n_it_edi_export_check` | internal rule | self, checks | `l10n_it_edi` |  |  |
| `_l10n_it_edi_is_italian` | internal rule | self | `l10n_it_edi` |  |  |
| `l10n_it_edi_doi_action_open_declarations` | operation | self | `l10n_it_edi_doi` |  |  |
| `_compute_l10n_lk_vat_registered` | computation | self | `l10n_lk_invoice` | depends: `vat`, `country_id` |  |
| `_check_company_registry_ma` | validation | self | `l10n_ma` | constrains: `company_registry`, `country_id` |  |
| `_compute_l10n_my_tin_validation_state` | computation | self | `l10n_my_edi` | depends: `l10n_my_identification_type`, `l10n_my_identification_number`, `vat`, `l10n_my_edi_malaysian_tin` | The three @depends are used for the validation. If they change, we will invalidate it and expect the user to revalidate. |
| `_compute_l10n_my_edi_display_tin_warning` | computation | self | `l10n_my_edi` | depends_context: `company`, `l10n_my_identification_number` | We want to display the tin warning for companies registered to use MyInvois. |
| `_compute_l10n_my_identification_number_placeholder` | computation | self | `l10n_my_edi` | depends: `l10n_my_identification_type` | Computes a dynamic placeholder that depends on the selected type to help the user inputs their data. The placeholders have been taken from the MyInvois doc. |
| `_compute_l10n_my_edi_industrial_classification` | computation | self | `l10n_my_edi` |  |  |
| `action_validate_tin` | user action | self | `l10n_my_edi` |  | Calling this action will reach our EDI proxy in order to validate the TIN against the provided identification information. |
| `_l10n_my_edi_get_tin_for_myinvois` | internal rule | self | `l10n_my_edi` |  | Helper to return the VAT number relevant to the situation. |
| `_onchange_l10n_pe_district` | on change | self | `l10n_pe` | onchange: `l10n_pe_district` |  |
| `_onchange_l10n_pe_city_id` | on change | self | `l10n_pe` | onchange: `city_id` |  |
| `_pe_unlink_except_master_data` | internal rule | self | `l10n_pe_pos` | ondelete |  |
| `_compute_branch_code` | computation | self | `l10n_ph` | depends: `vat`, `country_id` |  |
| `_check_l10n_rs_edi_public_funds` | validation | self | `l10n_rs_edi` | constrains: `l10n_rs_edi_public_funds` |  |
| `_check_l10n_rs_edi_registration_number` | validation | self | `l10n_rs_edi` | constrains: `l10n_rs_edi_registration_number` |  |
| `onchange_l10n_se_default_vendor_payment_ref` | on change | self | `l10n_se` | onchange: `l10n_se_default_vendor_payment_ref` |  |
| `_compute_l10n_th_branch_name` | computation | self | `l10n_th` |  |  |
| `_compute_nilvera_customer_alias_id` | computation | self | `l10n_tr_nilvera` | depends: `l10n_tr_nilvera_customer_alias_ids` |  |
| `_send_user_notification` | internal rule | self, type, message, action_button | `l10n_tr_nilvera` |  |  |
| `l10n_tr_check_nilvera_customer` | operation | self | `l10n_tr_nilvera` |  |  |
| `_check_nilvera_customer` | validation | self | `l10n_tr_nilvera` |  |  |
| `_l10n_tr_nilvera_validate_partner_details` | internal rule | self, is_delivery_partner | `l10n_tr_nilvera_edispatch` |  |  |
| `_l10n_tw_edi_formatted_address` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_l10n_uy_build_vat_error_message` | internal rule | self, partner | `l10n_uy` | model | Similar to _build_vat_error_message but using latam doc type name instead of vat_label NOTE: maybe can be implemented in master to l10n_latam_base for the use of different doc types |
| `_l10n_uy_ci_nie_is_valid` | internal rule | self | `l10n_uy` |  | Check if the partner's CI or NIE number is a valid one.  CI:     1) The ID number is taken up to the second to last position, that is, the first 6 or 7 digits.     2) Each digit is multiplied by a different factor starting from right to left, the factors are:         2, 9, 8, 7, 6, 3, 4.     3) The products obtained are added:     4) The base module 10 is calculated on this result to obtain the check digit, expressed in another way,     the next number ending in zero is taken that follows the result of the addition (for the example     would be 60) subtracting the sum itself: 60 - 59 = 1. The  |
| `_compute_count_active_cards` | computation | self | `loyalty` |  |  |
| `action_view_loyalty_cards` | user action | self | `loyalty` |  |  |
| `_compute_bom_ids` | computation | self | `mrp_subcontracting` |  |  |
| `_compute_production_ids` | computation | self | `mrp_subcontracting` |  |  |
| `_compute_picking_ids` | computation | self | `mrp_subcontracting` |  |  |
| `_search_is_subcontractor` | search rule | self, operator, value | `mrp_subcontracting` |  |  |
| `_compute_is_subcontractor` | computation | self | `mrp_subcontracting` |  | Determine whether the partner is a subcontractor (for giving sudo access) |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_load_pos_self_data_read` | internal rule | self, records, config | `pos_self_order` | model | Read specific fields from the given records |
| `action_privacy_lookup` | user action | self | `privacy_lookup` |  |  |
| `_gelato_prepare_address_payload` | internal rule | self | `sale_gelato` |  | Trim address fields according to maximum length allowed by Gelato. |
| `_compute_implemented_partner_count` | computation | self | `website_crm_partner_assign` | depends: `implemented_partner_ids.is_published`, `implemented_partner_ids.active` |  |
| `_compute_partner_weight` | computation | self | `website_crm_partner_assign` | depends: `grade_id.partner_weight` |  |
| `get_backend_menu_id` | operation | self | `website_customer` |  |  |

## Validation and error messages (52)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_id` | ValidationError | You cannot create recursive Partner hierarchies. | `base` |
| `_check_partner_company` | ValidationError | The company assigned to this partner does not match the company this partner represents. | `base` |
| `_check_barcode_unicity` | ValidationError | Another partner already has this barcode | `base` |
| `write` | RedirectWarning | error_msg | `base` |
| `write` | ValidationError | You cannot archive contacts linked to an active user. Ask an administrator to archive their associated user first.  Linked active users : %(names)s | `base` |
| `write` | UserError | The selected company is not compatible with the companies of the related user(s) | `base` |
| `_unlink_except_user` | RedirectWarning | error_msg | `base` |
| `_unlink_except_user` | ValidationError | You cannot delete contacts linked to an active user. Ask an administrator to archive their associated user first.  Linked active users : %(names)s | `base` |
| `name_create` | ValidationError | Couldn't create contact without email address! | `base` |
| `_signup_retrieve_partner` | UserError | Signup token '%s' is not valid or expired | `auth_signup` |
| `write` | UserError | You cannot set a partner as an invoicing address of another if they have a different %(vat_label)s. | `account` |
| `_unlink_if_partner_in_account_move` | UserError | The partner cannot be deleted because it is used in Accounting | `account` |
| `_merge_method` | UserError | Partners that are used in hashed entries cannot be merged. | `account` |
| `_check_peppol_fields` | ValidationError | error | `account_edi_ubl_cii` |
| `_run_vat_checks` | ValidationError | To explicitly indicate no (valid) VAT, use '/' instead. | `base_vat` |
| `_run_vat_checks` | ValidationError | msg | `base_vat` |
| `_run_vat_checks` | ValidationError | msg + '\n\n' + _('If you are trying to input a European number, this is the expected format: ') + _ref_vat[country_code.lower()] | `base_vat` |
| `_get_iap_vies_endpoint` | UserError | Invalid IAP VIES endpoint | `base_vat` |
| `_unlink_contact_rel_employee` | UserError | You cannot delete contact that are linked to an employee, please archive them instead. | `hr` |
| `_unlink_contact_rel_employee` | RedirectWarning | error_msg | `hr` |
| `_ensure_same_company_than_projects` | UserError | Partner company cannot be different from its assigned projects' company | `project` |
| `_ensure_same_company_than_tasks` | UserError | Partner company cannot be different from its assigned tasks' company | `project` |
| `_unlink_if_pos_no_orders` | ValidationError | You cannot delete a customer that has point of sales orders. You can archive it instead. | `point_of_sale` |
| `ensure_vat` | UserError | No VAT configured for partner [%i] %s | `l10n_ar` |
| `_l10n_ar_identification_validation` | ValidationError | The validation digit is not valid for "%s" | `l10n_ar` |
| `_l10n_ar_identification_validation` | ValidationError | Invalid length for "%s" | `l10n_ar` |
| `_l10n_ar_identification_validation` | ValidationError | Only numbers allowed for "%s" | `l10n_ar` |
| `_l10n_ar_identification_validation` | ValidationError | CUIT number must be prefixed with one of the following: %s | `l10n_ar` |
| `_l10n_ar_identification_validation` | ValidationError | repr(error) | `l10n_ar` |
| `_ar_unlink_except_master_data` | UserError | Deleting this partner is not allowed. | `l10n_ar_pos` |
| `_run_check_identification` | ValidationError | The format of your RUN is not valid.  It should be like 76086428-5. | `l10n_cl` |
| `_check_nemhandel_send_oioubl` | ValidationError | On Nemhandel, only OIOUBL 2.1 is supported. | `l10n_dk_nemhandel` |
| `_run_check_identification` | ValidationError | If your identification type is %s, it must be 10 digits | `l10n_ec` |
| `_validate_l10n_es_edi_facturae_ac_physical_gln` | ValidationError | The Physical GLN entered is not valid. | `l10n_es_edi_facturae` |
| `_validate_l10n_es_edi_facturae_ac_logical_operational_point` | ValidationError | The Logical Operational Point entered is not valid. | `l10n_es_edi_facturae` |
| `_check_pdp_send_ubl_21_fr` | ValidationError | For French regulated invoices, only %(format_name)s is supported. | `l10n_fr_pdp` |
| `_check_mojeracun_send_ubl_hr` | ValidationError | On MojEracun, only the Croatian UBL format is supported. | `l10n_hr_edi` |
| `action_l10n_in_verify_gstin_status` | UserError | You must be logged in an Indian company to use this feature | `l10n_in` |
| `action_l10n_in_verify_gstin_status` | ValidationError | Please enter the GSTIN | `l10n_in` |
| `action_l10n_in_verify_gstin_status` | ValidationError | This feature is not activated. Go to Settings to activate this feature. | `l10n_in` |
| `action_l10n_in_verify_gstin_status` | UserError | Unable to connect with GST network | `l10n_in` |
| `action_l10n_in_verify_gstin_status` | UserError | error_messages and '\n'.join(error_messages) or default_error_message | `l10n_in` |
| `action_l10n_in_verify_gstin_status` | UserError | The provided GSTIN is invalid. Please check the GSTIN and try again. | `l10n_in` |
| `validate_codice_fiscale` | UserError | Invalid Codice Fiscale '%s': should be like 'MRTMTT91D08F205J' for physical person and '12345670546' for businesses. | `l10n_it_edi` |
| `_check_company_registry_ma` | ValidationError | ICE number should have exactly 15 digits. | `l10n_ma` |
| `action_validate_tin` | UserError | In order to validate the TIN, you must provide the Identification type and number. | `l10n_my_edi` |
| `action_validate_tin` | UserError | Please register for the E-Invoicing service in the settings first. | `l10n_my_edi` |
| `_pe_unlink_except_master_data` | UserError | Deleting the partner %s is not allowed because it is required by the Peruvian point of sale. | `l10n_pe_pos` |
| `_check_l10n_rs_edi_public_funds` | ValidationError | Public Funds ID(JBKJS) must be exactly five digits | `l10n_rs_edi` |
| `_check_l10n_rs_edi_registration_number` | ValidationError | Customer identification number should be 8 or 13 digits | `l10n_rs_edi` |
| `_run_check_identification` | ValidationError | self._l10n_uy_build_vat_error_message(partner) | `l10n_uy` |
| `write` | UserError | You are trying to assign two different pricelists (one directly and one from grade (%(grade_name)s)). | `partnership` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | no | yes | no | no | `account` |
| `group_public` | no | yes | no | no | `base` |
| `group_portal` | no | yes | no | no | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |
| `group_user` | no | yes | no | no | `base` |
| `sales_team.group_sale_manager` | no | yes | no | no | `crm` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | no | `mrp` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `group_purchase_manager` | yes | yes | yes | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | no | `sale` |
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| res.partner company | global (all users) | `['\|', '\|', ('partner_share', '=', False), ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| res_partner: portal/public: read access on my commercial partner | `[Command.link(ref('base.group_portal')), Command.link(ref('base.group_public'))]` | `[('id', 'child_of', user.commercial_partner_id.id)]` | True | False | False | False |

## Views (121)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.partner_view_buttons` | div | `base.view_partner_form` | `currency_id`, `total_invoiced`, `supplier_invoice_count` | `action_view_partner_invoices`, `%(account.res_partner_action_supplier_bills)d` |  | `account` |
| `account.view_partner_property_form` | xpath | `base.view_partner_form` | `fiscal_country_codes` |  |  | `account` |
| `account.res_partner_view_tree` | xpath | `base.view_partner_tree` | `invoice_sending_method`, `invoice_edi_format`, `currency_id` |  |  | `account` |
| `account.res_partner_view_search` | xpath | `base.view_res_partner_filter` | `fiscal_country_codes` |  | `Customer Invoices`, `Vendor Bills` | `account` |
| `account.partner_missing_account_list_view` | list |  | `name` |  |  | `account` |
| `account_add_gln.view_partner_form_inherit` | xpath | `base.view_partner_form` | `global_location_number` |  |  | `account_add_gln` |
| `account_edi_ubl_cii.view_partner_property_form` | xpath | `account.view_partner_property_form` | `available_peppol_eas`, `peppol_eas`, `peppol_endpoint` |  |  | `account_edi_ubl_cii` |
| `account_peppol.res_partner_form_account_peppol` | data | `account_edi_ubl_cii.view_partner_property_form` | `available_peppol_sending_methods`, `available_peppol_edi_formats`, `bank_account_count`, `is_peppol_edi_format`, `peppol_verification_state`, `peppol_verification_state` | `Verify` |  | `account_peppol` |
| `account_peppol_response.res_partner_form_account_peppol_response` | data | `account_peppol.res_partner_form_account_peppol` | `peppol_verification_state`, `peppol_verification_state` |  |  | `account_peppol_response` |
| `base.view_partner_tree` | list |  | `avatar_128`, `complete_name`, `display_name`, `vat`, `email`, `phone`, `user_id`, `street`, `city`, `state_id`, `country_id`, `application_statistics`, `category_id`, `company_id`, `properties` |  |  | `base` |
| `base.view_partner_simple_form` | form |  | `image_1920`, `company_type`, `name`, `name`, `parent_id`, `function`, `email`, `phone` |  |  | `base` |
| `base.view_partner_address_form` | form |  | `image_1920`, `name`, `type`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `website` |  |  | `base` |
| `base.view_partner_form` | form |  | `same_vat_partner_id`, `vat_label`, `same_company_registry_partner_id`, `company_registry_label`, `same_vat_partner_id`, `vat_label`, `company_registry_label`, `same_vat_partner_id`, `vat_label`, `same_company_registry_partner_id`, `company_registry_label`, `image_1920`, `avatar_128`, `company_type`, `name`, `name`, `email`, `phone`, `parent_id`, `company_name`, `type_address_label`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `function`, `vat`, `website`, `lang`, `category_id`, `properties`, `child_ids`, `color`, `type`, `is_company`, `avatar_128`, `avatar_128`, `name`, `type`, `email`, `phone`, `function`, `city`, `country_id`, `type`, `image_1920`, `name`, `email`, `phone`, `function`, `type_address_label`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `company_id` | `Create` |  | `base` |
| `base.view_res_partner_filter` | search |  | `name`, `parent_id`, `email`, `phone`, `category_id`, `user_id`, `properties` |  | `Persons`, `Companies`, `Archived`, `Salesperson`, `Company`, `Country`, `Properties` | `base` |
| `base.res_partner_kanban_view` | kanban |  | `avatar_128`, `is_company`, `active`, `avatar_128`, `parent_id`, `parent_id`, `avatar_128`, `complete_name`, `display_name`, `email`, `phone`, `city`, `country_id`, `properties`, `application_statistics` |  |  | `base` |
| `base_address_extended.address_street_extended_form` | form |  | `country_enforce_cities`, `parent_id`, `type`, `street`, `street_name`, `street_number`, `street_number2`, `street2`, `city_id`, `city`, `state_id`, `zip`, `country_id` |  |  | `base_address_extended` |
| `base_address_extended.address_street_extended_city_form` | field | `base.view_partner_form` | `city` |  |  | `base_address_extended` |
| `base_geolocalize.view_crm_partner_geo_form` | xpath | `base.view_partner_form` | `partner_latitude`, `partner_longitude`, `date_localization` | `Compute based on address`, `Refresh` |  | `base_geolocalize` |
| `base_setup.res_partner_kanban_view` | xpath | `base.res_partner_kanban_view` | `category_id` |  |  | `base_setup` |
| `base_vat.view_partner_base_vat_form` | xpath | `base.view_partner_form` | `perform_vies_validation` |  |  | `base_vat` |
| `calendar.view_partners_form` | data | `base.view_partner_form` | `meeting_count` | `schedule_meeting` |  | `calendar` |
| `crm.view_partners_form_crm1` | data | `base.view_partner_form` | `opportunity_count` | `action_view_opportunity` |  | `crm` |
| `delivery.view_partner_property_form` | group | `base.view_partner_form` | `property_delivery_carrier_id` |  |  | `delivery` |
| `event.res_partner_view_tree` | div | `base.view_partner_form` | `event_count` | `action_event_view` |  | `event` |
| `google_address_autocomplete.view_partner_form_inherit_address_autocomplete` | xpath | `base.view_partner_form` |  |  |  | `google_address_autocomplete` |
| `google_address_autocomplete.view_partner_address_form_inherit_address_autocomplete` | xpath | `base.view_partner_address_form` |  |  |  | `google_address_autocomplete` |
| `hr.res_partner_view_form` | div | `base.view_partner_form` | `employees_count` | `action_open_employees` |  | `hr` |
| `hr.res_partner_view_search` | xpath | `base.view_res_partner_filter` |  |  | `Employees` | `hr` |
| `hr_calendar.view_res_partner_filter_inherit_calendar` | filter | `base.view_res_partner_filter` |  |  | `type_company`, `My Team` | `hr_calendar` |
| `im_livechat.view_partner_form` | div | `base.view_partner_form` | `livechat_channel_count` | `action_view_livechat_sessions` |  | `im_livechat` |
| `l10n_ar.base_view_partner_form` | xpath | `l10n_latam_base.view_partner_latam_form` | `l10n_ar_afip_responsibility_type_id` |  |  | `l10n_ar` |
| `l10n_ar.view_partner_property_form` | field | `account.view_partner_property_form` | `property_account_position_id`, `l10n_ar_gross_income_type`, `l10n_ar_gross_income_number` |  |  | `l10n_ar` |
| `l10n_ar.view_res_partner_filter` | field | `base.view_res_partner_filter` | `category_id`, `l10n_ar_afip_responsibility_type_id` |  |  | `l10n_ar` |
| `l10n_ar_withholding.view_partner_form` | group | `account.view_partner_property_form` | `l10n_ar_partner_tax_ids`, `tax_id`, `from_date`, `to_date`, `ref` |  |  | `l10n_ar_withholding` |
| `l10n_br.br_partner_address_form` | form |  | `country_enforce_cities`, `parent_id`, `type`, `street`, `street_name`, `street_number`, `street_number2`, `street2`, `city_id`, `city`, `state_id`, `zip`, `country_id` |  |  | `l10n_br` |
| `l10n_br.br_partner_tax_fields_form` | xpath | `account.view_partner_property_form` | `l10n_br_ie_code`, `l10n_br_im_code`, `l10n_br_isuf_code` |  |  | `l10n_br` |
| `l10n_ca.res_partner_form_inherit_ca` | xpath | `account.view_partner_property_form` | `l10n_ca_pst` |  |  | `l10n_ca` |
| `l10n_cl.view_move_form` | field | `account.view_partner_property_form` | `street2` |  |  | `l10n_cl` |
| `l10n_cz.res_partner_view_form_inherit_l10n_cz` | xpath | `account.view_partner_property_form` | `company_registry` |  |  | `l10n_cz` |
| `l10n_dk.view_partner_form_inherit_l10n_dk` | xpath | `base.view_partner_form` |  |  |  | `l10n_dk` |
| `l10n_dk_nemhandel.res_partner_form_l10n_dk_nemhandel` | data | `account_edi_ubl_cii.view_partner_property_form` | `nemhandel_identifier_type`, `nemhandel_identifier_value`, `nemhandel_verification_state` | `Verify` |  | `l10n_dk_nemhandel` |
| `l10n_dk_nemhandel_response.res_partner_form_l10n_dk_nemhandel_response` | data | `l10n_dk_nemhandel.res_partner_form_l10n_dk_nemhandel` | `nemhandel_verification_state`, `nemhandel_verification_state` |  |  | `l10n_dk_nemhandel_response` |
| `l10n_ec.view_partner_form` | div | `base.view_partner_form` | `l10n_ec_vat_validation` |  |  | `l10n_ec` |
| `l10n_eg_edi_eta.eg_partner_address_form` | form |  | `parent_id`, `type`, `l10n_eg_building_no`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id` |  |  | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.view_partner_form_inherit_l10n_es_edi_facturae` | xpath | `base.view_partner_form` | `l10n_es_edi_facturae_ac_center_code`, `l10n_es_edi_facturae_ac_role_type_ids`, `l10n_es_edi_facturae_ac_physical_gln`, `l10n_es_edi_facturae_ac_logical_operational_point` |  |  | `l10n_es_edi_facturae` |
| `l10n_fi.view_partner_form_inherit_l10n_fi` | xpath | `base.view_partner_form` |  |  |  | `l10n_fi` |
| `l10n_fr.view_partner_form_inherit_l10n_fr` | field | `base.view_partner_form` | `company_registry` |  |  | `l10n_fr` |
| `l10n_fr_pdp.res_partner_form_l10n_fr_pdp` | data | `account_peppol_response.res_partner_form_account_peppol_response` |  |  |  | `l10n_fr_pdp` |
| `l10n_gr_edi.view_partner_property_form_inherit_l10n_gr_edi` | field | `account.view_partner_property_form` | `property_account_position_id`, `l10n_gr_edi_branch_number` |  |  | `l10n_gr_edi` |
| `l10n_hr_edi.res_partner_view_form_inherit` | xpath | `account.partner_view_buttons` | `l10n_hr_personal_oib`, `l10n_hr_business_unit_code` |  |  | `l10n_hr_edi` |
| `l10n_hu_edi.view_partner_form_l10n_hu_edi` | xpath | `account.view_partner_property_form` | `l10n_hu_group_vat` |  |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.res_partner_tax_form_view` | xpath | `base.view_partner_form` | `l10n_id_pkp`, `l10n_id_kode_transaksi` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_in.l10n_in_view_partner_form` | xpath | `account.view_partner_property_form` |  |  |  | `l10n_in` |
| `l10n_in.l10n_in_view_partner_tree` | xpath | `base.view_partner_tree` | `l10n_in_pan_entity_id` |  |  | `l10n_in` |
| `l10n_in.l10n_in_view_res_partner_filter` | field | `base.view_res_partner_filter` | `user_id`, `l10n_in_pan_entity_id` |  |  | `l10n_in` |
| `l10n_in.l10n_in_view_partner_base_vat_form` | xpath | `base_vat.view_partner_base_vat_form` | `l10n_in_gstin_verified_date` | `action_l10n_in_verify_gstin_status`, `Check Status` |  | `l10n_in` |
| `l10n_it_edi.res_partner_tree_l10n_it` | xpath | `base.view_partner_tree` | `l10n_it_codice_fiscale`, `l10n_it_pa_index` |  |  | `l10n_it_edi` |
| `l10n_it_edi.res_partner_form_l10n_it` | data | `account.view_partner_property_form` | `l10n_it_pec_email`, `l10n_it_codice_fiscale`, `l10n_it_pa_index` |  |  | `l10n_it_edi` |
| `l10n_it_edi_doi.res_partner_view_search` | xpath | `account.res_partner_view_search` |  |  | `Exceeded Declaration of Intent` | `l10n_it_edi_doi` |
| `l10n_it_edi_doi.view_partner_l10n_form` | div | `base_vat.view_partner_base_vat_form` |  | `l10n_it_edi_doi_action_open_declarations` |  | `l10n_it_edi_doi` |
| `l10n_ke_edi_tremol.res_partner_view_form` | group | `account.view_partner_property_form` | `l10n_ke_exemption_number` |  |  | `l10n_ke_edi_tremol` |
| `l10n_kr.kr_partner_address_form` | form |  | `country_id`, `state_id`, `city`, `street2`, `street`, `zip` |  |  | `l10n_kr` |
| `l10n_latam_base.view_partner_latam_form` | xpath | `base_vat.view_partner_base_vat_form` |  |  |  | `l10n_latam_base` |
| `l10n_lk_invoice.view_partner_form_l10n_lk_vat_registered` | xpath | `base.view_partner_form` | `l10n_lk_vat_registered` |  |  | `l10n_lk_invoice` |
| `l10n_ma.view_partner_property_form` | xpath | `account.view_partner_property_form` |  |  |  | `l10n_ma` |
| `l10n_my_edi.view_partner_form_inherit_l10n_my_myinvois` | group | `account.view_partner_property_form` | `l10n_my_tin_validation_state`, `l10n_my_edi_display_tin_warning`, `l10n_my_identification_type`, `l10n_my_identification_number_placeholder`, `l10n_my_identification_number`, `l10n_my_edi_industrial_classification`, `l10n_my_edi_malaysian_tin` | `action_validate_tin` |  | `l10n_my_edi` |
| `l10n_my_ubl_pint.view_partner_form_inherit_l10n_my_ubl_pint` | xpath | `account.view_partner_property_form` | `sst_registration_number`, `ttx_registration_number` |  |  | `l10n_my_ubl_pint` |
| `l10n_no.view_partner_form_inherit_l10n_no` | xpath | `base.view_partner_form` | `l10n_no_bronnoysund_number` |  |  | `l10n_no` |
| `l10n_nz.view_partner_form_inherit_l10n_nz` | xpath | `base.view_partner_form` |  |  |  | `l10n_nz` |
| `l10n_pe.pe_partner_address_form` | form |  | `country_enforce_cities`, `parent_id`, `type`, `street`, `street2`, `l10n_pe_district`, `city_id`, `city`, `state_id`, `zip`, `country_id` |  |  | `l10n_pe` |
| `l10n_ph.view_partner_form` | xpath | `account.view_partner_property_form` | `branch_code`, `l10n_ph_rdo`, `first_name`, `middle_name`, `last_name` |  |  | `l10n_ph` |
| `l10n_pl.res_partner_account_pl_form` | xpath | `account.view_partner_property_form` | `l10n_pl_links_with_customer` |  |  | `l10n_pl` |
| `l10n_pl_edi_jst.res_partner_account_pl_form` | xpath | `account.view_partner_property_form` | `l10n_pl_parent_lgu` |  |  | `l10n_pl_edi_jst` |
| `l10n_ro.res_partner_form_ro` | field | `account.view_partner_property_form` | `vat`, `nrc` |  |  | `l10n_ro` |
| `l10n_rs_edi.res_partner_view_form` | xpath | `account.view_partner_property_form` | `l10n_rs_edi_registration_number`, `l10n_rs_edi_public_funds` |  |  | `l10n_rs_edi` |
| `l10n_sa_edi.sa_partner_address_form` | form |  | `parent_id`, `type`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `l10n_sa_edi_building_number`, `l10n_sa_edi_plot_identification` |  |  | `l10n_sa_edi` |
| `l10n_sa_edi.view_partner_form` | xpath | `base.view_partner_form` | `l10n_sa_edi_additional_identification_scheme`, `l10n_sa_edi_additional_identification_number` |  |  | `l10n_sa_edi` |
| `l10n_se.se_partner_address_form` | form |  | `parent_id`, `type`, `street`, `street2`, `zip`, `city`, `state_id`, `country_id` |  |  | `l10n_se` |
| `l10n_se.res_partner_ocr_form` | group | `account.view_partner_property_form` | `l10n_se_check_vendor_ocr`, `l10n_se_default_vendor_payment_ref` |  |  | `l10n_se` |
| `l10n_sg.view_partner_form_l10n_sg` | xpath | `account.view_partner_property_form` | `l10n_sg_unique_entity_number` |  |  | `l10n_sg` |
| `l10n_si.si_partner_address_form` | form |  | `street`, `street2`, `zip`, `city`, `country_id` |  |  | `l10n_si` |
| `l10n_sk.res_partner_view_form_inherit_l10n_sk` | xpath | `account.view_partner_property_form` | `company_registry` |  |  | `l10n_sk` |
| `l10n_tr_nilvera.view_partner_property_form_inherit_ubl_tr` | xpath | `account_edi_ubl_cii.view_partner_property_form` | `l10n_tr_nilvera_customer_status`, `l10n_tr_nilvera_customer_alias_id` | `Verify` |  | `l10n_tr_nilvera` |
| `l10n_tr_nilvera_edispatch.view_partner_form_inherit_l10n_tr_nilvera_edispatch` | xpath | `base.view_partner_form` | `fiscal_country_codes`, `l10n_tr_nilvera_edispatch_customs_zip` |  |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_einvoice_extended.view_partner_form_l10n_tr_nilvera_extended` | xpath | `base.view_partner_form` | `l10n_tr_tax_office_id` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_uz.l10n_uz_view_partner_form` | xpath | `account.view_partner_property_form` |  |  |  | `l10n_uz` |
| `l10n_vn_edi_viettel.res_partner_view_from_inherit_l10n_vn_edi` | group | `account.view_partner_property_form` | `l10n_vn_edi_symbol` |  |  | `l10n_vn_edi_viettel` |
| `loyalty.res_partner_form` | div | `base.view_partner_form` | `loyalty_card_count` | `action_view_loyalty_cards` |  | `loyalty` |
| `mail.res_partner_view_form_inherit_mail` | xpath | `base.view_partner_form` | `is_blacklisted` | `mail_action_blacklist_remove` |  | `mail` |
| `mail.res_partner_view_kanban_inherit_mail` | xpath | `base.res_partner_kanban_view` | `activity_ids` |  |  | `mail` |
| `mail.res_partner_view_search_inherit_mail` | filter | `base.view_res_partner_filter` |  |  | `inactive`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities` | `mail` |
| `mail.res_partner_view_tree_inherit_mail` | xpath | `base.view_partner_tree` | `activity_ids` |  |  | `mail` |
| `mail.res_partner_view_activity` | activity |  | `id`, `name`, `parent_id` |  |  | `mail` |
| `mrp_subcontracting.view_partner_mrp_subcontracting_form` | xpath | `stock.view_partner_stock_form` | `property_stock_subcontractor` |  |  | `mrp_subcontracting` |
| `mrp_subcontracting.view_partner_mrp_subcontracting_filter` | xpath | `base.view_res_partner_filter` |  |  | `Subcontractors` | `mrp_subcontracting` |
| `partnership.view_res_partner_filter_assign` | field | `base.view_res_partner_filter` | `user_id`, `specific_property_product_pricelist`, `grade_id` |  |  | `partnership` |
| `partnership.view_res_partner_grade_tree` | field | `base.view_partner_tree` | `vat`, `grade_id` |  |  | `partnership` |
| `partnership.view_res_partner_form` | field | `base.view_partner_form` | `vat`, `grade_id` |  |  | `partnership` |
| `payment.view_partners_form_payment_defaultcreditcard` | div | `base.view_partner_form` | `payment_token_count` | `%(payment.action_payment_token)d` |  | `payment` |
| `phone_validation.res_partner_view_search` | xpath | `base.view_res_partner_filter` | `phone_mobile_search` |  |  | `phone_validation` |
| `point_of_sale.view_partner_property_form` | div | `base.view_partner_form` | `pos_order_count` | `action_view_pos_order` |  | `point_of_sale` |
| `pos_loyalty.res_partner_form` | button | `loyalty.res_partner_form` |  | `action_view_loyalty_cards` |  | `pos_loyalty` |
| `product.view_partner_property_form` | group | `base.view_partner_form` | `property_product_pricelist` |  |  | `product` |
| `project.view_task_partner_info_form` | div | `base.view_partner_form` | `task_count` | `action_view_tasks` |  | `project` |
| `purchase.view_partner_property_form` | group | `base.view_partner_form` | `receipt_reminder_email`, `reminder_date_before_receipt`, `property_purchase_currency_id` |  |  | `purchase` |
| `purchase.res_partner_view_purchase_buttons` | div | `base.view_partner_form` | `purchase_order_count` | `%(purchase.act_res_partner_2_purchase_order)d` |  | `purchase` |
| `purchase_stock.res_partner_view_purchase_buttons_inherit` | xpath | `purchase.view_partner_property_form` | `on_time_rate` | `%(action_purchase_vendor_delay_report)d` |  | `purchase_stock` |
| `sale.res_partner_view_buttons` | div | `base.view_partner_form` | `sale_order_count` | `sale.act_res_partner_2_sale_order` |  | `sale` |
| `sale.res_partner_view_form_payment_defaultcreditcard` | button | `payment.view_partners_form_payment_defaultcreditcard` |  | `%(payment.action_payment_token)d` |  | `sale` |
| `sale.res_partner_view_form_property_inherit` | group | `account.view_partner_property_form` |  |  |  | `sale` |
| `sale_loyalty.res_partner_form` | button | `loyalty.res_partner_form` |  | `action_view_loyalty_cards` |  | `sale_loyalty` |
| `sms.res_partner_view_form` | xpath | `base.view_partner_form` | `phone_sanitized` |  |  | `sms` |
| `stock.view_partner_stock_form` | xpath | `mail.res_partner_view_form_inherit_mail` | `property_stock_customer`, `property_stock_supplier` |  |  | `stock` |
| `stock.view_partner_stock_warnings_form` | group | `base.view_partner_form` | `picking_warn_msg` |  |  | `stock` |
| `survey.res_partner_view_form` | xpath | `base.view_partner_form` | `certifications_count`, `certifications_company_count` | `action_view_certifications`, `action_view_certifications` |  | `survey` |
| `website.view_partner_form_inherit_website` | xpath | `base.view_partner_form` | `website_id` |  |  | `website` |
| `website_crm_partner_assign.view_res_partner_filter_assign_tree` | field | `base.view_partner_tree` | `vat`, `date_review_next`, `activation` |  |  | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_crm_partner_assign_form` | data | `base_geolocalize.view_crm_partner_geo_form` | `activation`, `partner_weight`, `date_review`, `date_review_next`, `date_partnership`, `assigned_partner_id` |  |  | `website_crm_partner_assign` |
| `website_customer.view_partners_form_website` | data | `website_partner.view_partners_form_website` | `website_tag_ids` |  |  | `website_customer` |
| `website_partner.view_partners_form_website` | data | `base.view_partner_form` | `is_published` |  |  | `website_partner` |
| `website_slides.res_partner_view_form` | xpath | `base.view_partner_form` | `slide_channel_count`, `slide_channel_company_count` | `action_view_courses`, `action_view_courses` |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.res_partner_action_customer` | Customers | list,kanban,form |  | `{'search_default_customer': 1,'res_partner_search_mode': 'customer', 'default_is_company': True, 'default_customer_rank': 1}` |  | `account` |
| `account.res_partner_action_supplier` | Vendors | list,kanban,form |  | `{'search_default_supplier': 1,'res_partner_search_mode': 'supplier', 'default_is_company': True, 'default_supplier_rank': 1}` |  | `account` |
| `base.action_partner_form` | Customers | list,kanban,form |  | `{'res_partner_search_mode': 'customer'}` |  | `base` |
| `base.action_partner_customer_form` | Customers | list,kanban,form | `[]` | `{'res_partner_search_mode': 'customer', 'default_is_company': True}` |  | `base` |
| `base.action_partner_supplier_form` | Vendors | kanban,list,form | `[]` | `{'res_partner_search_mode': 'supplier', 'default_is_company': True}` |  | `base` |
| `contacts.action_contacts` | Contacts | list,kanban,form,activity |  | `{'default_is_company': True}` |  | `contacts` |
| `partnership.action_pricelist_partners` | Members / Partners | list,kanban,form,activity | `[('specific_property_product_pricelist', '=', active_id)]` | `{"search_default_specific_property_product_pricelist": active_id, "default_specific_property_product_pricelist": active_id}` |  | `partnership` |
| `partnership.action_grade_partners` | Members / Partners | list,kanban,form,activity | `[('grade_id', '=', active_id)]` | `{"search_default_grade_id": active_id, "default_grade_id": active_id}` |  | `partnership` |
| `point_of_sale.res_partner_action_edit_pos` | Edit Partner | form |  |  | new | `point_of_sale` |
| `website.visitor_partner_action` | Partners | list,form | `[('visitor_ids', 'in', [active_id])]` |  |  | `website` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.menu_account_supplier` | Vendors |  | `account.res_partner_action_supplier` | 200 |  |
| `point_of_sale.menu_point_of_sale_customer` | Customers | `menu_point_of_sale` | `account.res_partner_action_customer` | 100 |  |
| `purchase.menu_procurement_management_supplier_name` | Vendors | `menu_procurement_management` | `account.res_partner_action_supplier` | 15 |  |
| `sale.res_partner_menu` |  |  | `account.res_partner_action_customer` | 40 | `sales_team.group_sale_salesman` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account_peppol.partner_action_verify_peppol` | Verify Peppol | code |  | yes |
| `l10n_dk_nemhandel.partner_action_verify_l10n_dk_nemhandel` | Verify Nemhandel | code |  | yes |
| `l10n_tr_nilvera.action_account_reports_customer_statements_do_followup` | Verify Nilvera Status | code |  | yes |
| `privacy_lookup.ir_action_server_action_privacy_lookup_partner` | Privacy Lookup | code |  | yes |
| `web.download_contact` | Download (vCard) | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `base_vat.vies_iap_check_update` | Base VAT: Sync updates from IAP VIES | 1 days | `_cron_check_vies_iap` |  |

Machine-readable definition: `../../../schemas/data/entities/res.partner.json`; views: `../../../schemas/interfaces/views/res.partner.json`.
