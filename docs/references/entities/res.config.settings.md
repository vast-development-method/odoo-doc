# Config Settings (`res.config.settings`)

**Transport name:** `res.config.settings`  
**Storage name:** `res_config_settings`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `base_setup`, `mail`, `product`, `analytic`, `auth_signup`, `portal`, `digest`, `account`, `account_check_printing`, `fleet`, `payment`, `account_payment`, `account_payment_interco`, `account_peppol`, `auth_ldap`, `auth_oauth`, `auth_password_policy`, `auth_totp_mail`, `base_geolocalize`, `base_vat`, `cloud_storage`, `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage_migration`, `crm`, `crm_iap_enrich`, `sale`, `stock`, `stock_account`, `sale_stock`, `event`, `sale_management`, `google_address_autocomplete`, `google_calendar`, `google_gmail`, `google_recaptcha`, `hr`, `hr_attendance`, `hr_expense`, `maintenance`, `hr_presence`, `hr_recruitment`, `website`, `website_slides`, `project`, `hr_timesheet`, `l10n_account_withholding_tax`, `partner_autocomplete`, `point_of_sale`, `l10n_gcc_invoice`, `l10n_gcc_pos`, `website_payment`, `website_sale`, `l10n_ar_website_sale`, `l10n_ar_withholding`, `l10n_din5008`, `pos_restaurant`, `pos_sale`, `purchase`, `l10n_dk_nemhandel`, `l10n_eg_edi_eta`, `l10n_es`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_verifactu`, `l10n_es_pos`, `l10n_eu_oss`, `l10n_fr_hr_holidays`, `l10n_fr_pdp`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_in`, `l10n_in_edi`, `l10n_in_ewaybill`, `purchase_stock`, `l10n_it_edi`, `l10n_jo_edi`, `l10n_jo_edi_pos`, `l10n_ke_edi_tremol`, `l10n_mx`, `l10n_my_edi`, `l10n_pl`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_tr_nilvera`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_vn_edi_viettel`, `l10n_vn_edi_viettel_pos`, `lunch`, `mass_mailing`, `website_event_track`, `microsoft_calendar`, `microsoft_outlook`, `mrp`, `stock_landed_costs`, `product_expiry`, `partnership`, `pos_adyen`, `pos_discount`, `pos_hr`, `pos_imin`, `pos_self_order`, `pos_online_payment_self_order`, `pos_self_order_sale`, `pos_sms`, `project_timesheet_holidays`, `purchase_requisition`, `sale_gelato`, `sale_timesheet`, `sms_twilio`, `snailmail`, `stock_sms`, `web_unsplash`, `website_cf_turnstile`, `website_livechat`, `website_sale_autocomplete`, `website_sale_stock`, `website_sale_collect`, `website_sale_mass_mailing`

Description: Config Settings

## Fields (690)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `web_app_name` | Web App Name | single line text |  |  |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `is_root_company` | Is Root Company | boolean |  | computed by rule `_compute_is_root_company` (not stored) |
| `module_base_import` | Allow users to import data from comma-separated values/XLS/XLSX/ODS files | boolean |  |  |
| `module_google_calendar` | Allow the users to synchronize their calendar  with Google Calendar | boolean |  |  |
| `module_microsoft_calendar` | Allow the users to synchronize their calendar with Outlook Calendar | boolean |  |  |
| `module_mail_plugin` | Allow integration with the mail plugins | boolean |  |  |
| `module_auth_oauth` | Use external authentication providers (open authorization) | boolean |  |  |
| `module_auth_ldap` | directory access protocol Authentication | boolean |  |  |
| `module_account_inter_company_rules` | Manage Inter Company | boolean |  |  |
| `module_voip` | Phone | boolean |  |  |
| `module_web_unsplash` | Unsplash Image Library | boolean |  |  |
| `module_sms` | text message | boolean |  |  |
| `module_partner_autocomplete` | Partner Autocomplete | boolean |  |  |
| `module_base_geolocalize` | GeoLocalize | boolean |  |  |
| `module_google_recaptcha` | reCAPTCHA | boolean |  |  |
| `module_website_cf_turnstile` | Cloudflare Turnstile | boolean |  |  |
| `module_google_address_autocomplete` | Google Address Autocomplete | boolean |  |  |
| `report_footer` | Custom Report Footer | rich text |  | related through path `company_id.report_footer`; Help: Footer text displayed at the bottom of all reports. |
| `group_multi_currency` | Multi-Currencies | boolean |  | Help: Allows to work in a multi currency environment |
| `external_report_layout_id` | External Report Layout | many to one |  | related through path `company_id.external_report_layout_id` |
| `show_effect` | Show Effect | boolean |  |  |
| `company_count` | Number of Companies | integer |  | computed by rule `_compute_company_count` (not stored) |
| `active_user_count` | Number of Active Users | integer |  | computed by rule `_compute_active_user_count` (not stored) |
| `language_count` | Number of Languages | integer |  | computed by rule `_compute_language_count` (not stored) |
| `company_name` | Company Name | single line text |  | related through path `company_id.display_name` |
| `company_informations` | Company Informations | multi line text |  | computed by rule `_compute_company_informations` (not stored) |
| `company_country_code` | Company Country Code | single line text |  | read only; related through path `company_id.country_id.code` |
| `company_country_group_codes` | Company Country Group Codes | structured document |  | related through path `company_id.country_id.country_group_codes` |
| `profiling_enabled_until` | Profiling enabled until | date and time |  |  |
| `external_email_server_default` | Use Custom Email Servers | boolean |  |  |
| `fail_counter` | Fail Mail | integer |  | computed by rule `_compute_fail_counter` (not stored) |
| `alias_domain_id` | Alias Domain | many to one | `mail.alias.domain` | related through path `company_id.alias_domain_id`; Help: If you have setup a catch-all email domain redirected to the system server, enter the domain name here. |
| `module_google_gmail` | Support Gmail Authentication | boolean |  |  |
| `module_microsoft_outlook` | Support Outlook Authentication | boolean |  |  |
| `restrict_template_rendering` | Restrict Template Rendering | boolean |  | Help: Users will still be able to render templates. However only Mail Template Editors will be able to create new dynamic templates or modify existing ones. |
| `use_twilio_rtc_servers` | Use Twilio ICE servers | boolean |  | Help: If you want to use twilio as TURN/STUN server provider |
| `twilio_account_sid` | Account SID | single line text |  |  |
| `twilio_account_token` | Account Auth Token | single line text |  |  |
| `use_sfu_server` | Use SFU server | boolean |  | Help: If you want to setup SFU server for large group calls. |
| `sfu_server_url` | SFU Server uniform resource locator | single line text |  |  |
| `sfu_server_key` | SFU Server key | single line text |  | Help: Base64 encoded key |
| `email_primary_color` | Email Primary Color | single line text |  | related through path `company_id.email_primary_color` |
| `email_secondary_color` | Email Secondary Color | single line text |  | related through path `company_id.email_secondary_color` |
| `tenor_api_key` | Klipy application programming interface key | single line text |  | Help: Add a Klipy GIF API key to enable GIFs support. https://docs.klipy.com/getting-started If you were using a Tenor GIF API key (service shutdown on June 30, 2026), please replace it here with a Klipy GIF API key |
| `google_translate_api_key` | Message Translation application programming interface Key | single line text |  | Help: A valid Google API key is required to enable message translation. https://cloud.google.com/translate/docs/setup |
| `group_uom` | Units of Measure & Packagings | boolean |  |  |
| `group_product_variant` | Variants | boolean |  |  |
| `module_loyalty` | Promotions, Coupons, Gift Card & Loyalty Program | boolean |  |  |
| `group_product_pricelist` | Pricelists | boolean |  |  |
| `product_weight_in_lbs` | Weight unit of measure | selection |  | default `0` |
| `product_volume_volume_in_cubic_feet` | Volume unit of measure | selection |  | default `0` |
| `group_analytic_accounting` | Analytic Accounting | boolean |  |  |
| `auth_signup_reset_password` | Enable password reset from Login page | boolean |  |  |
| `auth_signup_uninvited` | Customer Account | selection |  | computed by rule `_compute_auth_signup_uninvited` (not stored); writable through an inverse rule; default ; extended by packages `website` |
| `auth_signup_template_user_id` | Template user for new users created through signup | many to one | `res.users` |  |
| `portal_allow_api_keys` | Customer application programming interface Keys | boolean |  | computed by rule `_compute_portal_allow_api_keys` (not stored); writable through an inverse rule |
| `digest_emails` | Digest Emails | boolean |  |  |
| `digest_id` | Digest Email | many to one | `digest.digest` |  |
| `has_accounting_entries` | Has Accounting Entries | boolean |  | computed by rule `_compute_has_chart_of_accounts` (not stored) |
| `currency_id` | Currency | many to one | `res.currency` | required; related through path `company_id.currency_id`; Help: Main currency of the company.; extended by packages `lunch` |
| `currency_exchange_journal_id` | Currency Exchange Journal | many to one | `account.journal` | related through path `company_id.currency_exchange_journal_id`; restricted by domain `[('type', '=', 'general')]`; must belong to the same company; Help: The accounting journal where automatic exchange differences will be registered |
| `income_currency_exchange_account_id` | Gain Exchange Rate Account | many to one | `account.account` | related through path `company_id.income_currency_exchange_account_id`; restricted by domain `[('internal_group', '=', 'income')]`; must belong to the same company |
| `expense_currency_exchange_account_id` | Loss Exchange Rate Account | many to one | `account.account` | related through path `company_id.expense_currency_exchange_account_id`; restricted by domain `[('account_type', 'in', ('expense', 'expense_other'))]`; must belong to the same company |
| `has_chart_of_accounts` | Company has a chart of accounts | boolean |  | computed by rule `_compute_has_chart_of_accounts` (not stored) |
| `chart_template` | Chart Template | selection |  | default computed dynamically (lambda self: self.env.company.chart_template) |
| `sale_tax_id` | Default Sale Tax | many to one | `account.tax` | related through path `company_id.account_sale_tax_id`; must belong to the same company; extended by packages `point_of_sale` |
| `purchase_tax_id` | Default Purchase Tax | many to one | `account.tax` | related through path `company_id.account_purchase_tax_id`; must belong to the same company |
| `account_price_include` | Default Sales Price Include | selection |  | required; related through path `company_id.account_price_include`; Help: Default on whether the sales price used on the product and invoices with this Company includes its taxes. |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | related through path `company_id.tax_calculation_rounding_method` |
| `account_journal_suspense_account_id` | Bank Suspense | many to one | `account.account` | related through path `company_id.account_journal_suspense_account_id`; restricted by domain `[('account_type', 'in', ('asset_current', 'liability_current'))]`; must belong to the same company; Help: Bank Transactions are posted immediately after import or synchronization. Their counterparty is the bank suspense account. Reconciliation replaces the latter by the definitive account(s). |
| `transfer_account_id` | Internal Transfer | many to one | `account.account` | related through path `company_id.transfer_account_id`; restricted by domain `[["reconcile", "=", true], ["account_type", "=", "asset_current"]]`; must belong to the same company; Help: Intermediary account used when moving from a liquidity account to another. |
| `module_account_accountant` | Accounting | boolean |  |  |
| `group_cash_rounding` | Cash Rounding | boolean |  |  |
| `show_sale_receipts` | Sale Receipt | boolean |  |  |
| `module_account_budget` | Budget Management | boolean |  |  |
| `module_account_payment` | Invoice Online Payment | boolean |  |  |
| `module_account_reports` | Dynamic Reports | boolean |  |  |
| `module_account_check_printing` | Allow check printing and deposits | boolean |  |  |
| `module_account_batch_payment` | Use batch payments | boolean |  | Help: This allows you grouping payments into a single batch and eases the reconciliation process. -This installs the account_batch_payment module. |
| `module_account_iso20022` | Single Euro Payments Area Credit Transfer / ISO20022 | boolean |  |  |
| `module_account_sepa_direct_debit` | Use Single Euro Payments Area Direct Debit | boolean |  |  |
| `module_account_bank_statement_import_qif` | Import .qif files | boolean |  |  |
| `module_currency_rate_live` | Automatic Currency Rates | boolean |  |  |
| `module_account_intrastat` | Intrastat | boolean |  |  |
| `module_product_margin` | Allow Product Margin | boolean |  |  |
| `module_account_extract` | Document Digitization | boolean |  |  |
| `module_account_invoice_extract` | Invoice Digitization | boolean |  | computed by rule `_compute_module_account_invoice_extract` and stored |
| `module_account_bank_statement_extract` | Bank Statement Digitization | boolean |  | computed by rule `_compute_module_account_bank_statement_extract` and stored |
| `module_snailmail_account` | Snailmail | boolean |  |  |
| `module_account_peppol` | PEPPOL Invoicing | boolean |  |  |
| `tax_exigibility` | Cash Basis | boolean |  | related through path `company_id.tax_exigibility` |
| `tax_cash_basis_journal_id` | Tax Cash Basis Journal | many to one | `account.journal` | related through path `company_id.tax_cash_basis_journal_id`; must belong to the same company |
| `account_cash_basis_base_account_id` | Base Tax Received Account | many to one | `account.account` | related through path `company_id.account_cash_basis_base_account_id`; must belong to the same company |
| `account_fiscal_country_id` | Fiscal Country Code | many to one |  | related through path `company_id.account_fiscal_country_id` |
| `qr_code` | Display Single Euro Payments Area quick response-code | boolean |  | related through path `company_id.qr_code` |
| `link_qr_code` | Display Link quick response-code | boolean |  | related through path `company_id.link_qr_code` |
| `incoterm_id` | Default incoterm | many to one | `account.incoterms` | related through path `company_id.incoterm_id`; Help: International Commercial Terms are a series of predefined commercial terms used in international transactions. |
| `invoice_terms` | Terms & Conditions | rich text |  | related through path `company_id.invoice_terms` |
| `invoice_terms_html` | Terms & Conditions as a Web page | rich text |  | related through path `company_id.invoice_terms_html` |
| `terms_type` | Terms Type | selection |  | related through path `company_id.terms_type` |
| `display_invoice_amount_total_words` | Total amount of invoice in letters | boolean |  | related through path `company_id.display_invoice_amount_total_words` |
| `display_invoice_tax_company_currency` | Taxes in company currency | boolean |  | related through path `company_id.display_invoice_tax_company_currency` |
| `preview_ready` | Display preview button | boolean |  | computed by rule `_compute_terms_preview` (not stored) |
| `use_invoice_terms` | Default Terms & Conditions | boolean |  |  |
| `account_use_credit_limit` | Sales Credit Limit | boolean |  | related through path `company_id.account_use_credit_limit`; Help: Enable the use of credit limit on partners. |
| `account_default_credit_limit` | Default Credit Limit | monetary |  | computed by rule `_compute_account_default_credit_limit` (not stored); writable through an inverse rule; Help: This is the default credit limit that will be used on partners that do not have a specific limit on them. |
| `country_code` | Country Code | single line text |  | read only; related through path `company_id.account_fiscal_country_id.code` |
| `account_storno` | Storno accounting | boolean |  | related through path `company_id.account_storno` |
| `display_account_storno` | Display Account Storno | boolean |  | related through path `company_id.display_account_storno` |
| `group_sale_delivery_address` | Customer Addresses | boolean |  |  |
| `quick_edit_mode` | Quick encoding | selection |  | related through path `company_id.quick_edit_mode` |
| `account_journal_early_pay_discount_loss_account_id` | Early Discount Loss | many to one | `account.account` | related through path `company_id.account_journal_early_pay_discount_loss_account_id`; restricted by domain `[('account_type', 'in', ('expense', 'expense_other', 'income', 'income_other'))]`; must belong to the same company; Help: Account for the difference amount after the expense discount has been granted |
| `account_journal_early_pay_discount_gain_account_id` | Early Discount Gain | many to one | `account.account` | related through path `company_id.account_journal_early_pay_discount_gain_account_id`; restricted by domain `[('account_type', 'in', ('income', 'income_other', 'expense', 'expense_other'))]`; must belong to the same company; Help: Account for the difference amount after the income discount has been granted |
| `account_discount_income_allocation_id` | Vendor Bills Discounts Account | many to one | `account.account` | related through path `company_id.account_discount_income_allocation_id`; restricted by domain `[('account_type', 'in', ('income', 'income_other', 'expense', 'expense_other'))]` |
| `account_discount_expense_allocation_id` | Customer Invoices Discounts Account | many to one | `account.account` | related through path `company_id.account_discount_expense_allocation_id`; restricted by domain `[('account_type', 'in', ('income', 'income_other', 'expense', 'expense_other'))]` |
| `is_account_peppol_eligible` | PEPPOL eligible | boolean |  | computed by rule `_compute_is_account_peppol_eligible` (not stored) |
| `restrictive_audit_trail` | Restricted Audit Trail | boolean |  | related through path `company_id.restrictive_audit_trail` |
| `force_restrictive_audit_trail` | Forced Audit Trail | boolean |  | related through path `company_id.force_restrictive_audit_trail` |
| `autopost_bills` | Autopost Bills | boolean |  | related through path `company_id.autopost_bills` |
| `income_account_id` | Income Account | many to one |  | related through path `company_id.income_account_id`; must belong to the same company |
| `expense_account_id` | Expense Account | many to one |  | related through path `company_id.expense_account_id`; must belong to the same company |
| `account_check_printing_layout` | Check Layout | selection |  | related through path `company_id.account_check_printing_layout`; Help: Select the format corresponding to the check paper you will be printing your checks on. In order to disable the printing feature, select 'None'. |
| `account_check_printing_date_label` | Print Date Label | boolean |  | related through path `company_id.account_check_printing_date_label`; Help: This option allows you to print the date label on the check as per CPA. Disable this if your pre-printed check includes the date label. |
| `account_check_printing_multi_stub` | Multi-Pages Check Stub | boolean |  | related through path `company_id.account_check_printing_multi_stub`; Help: This option allows you to print check details (stub) on multiple pages if they don't fit on a single page. |
| `account_check_printing_margin_top` | Check Top Margin | float |  | related through path `company_id.account_check_printing_margin_top`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `account_check_printing_margin_left` | Check Left Margin | float |  | related through path `company_id.account_check_printing_margin_left`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `account_check_printing_margin_right` | Check Right Margin | float |  | related through path `company_id.account_check_printing_margin_right`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `delay_alert_contract` | Delay alert contract outdated | integer |  | default `30` |
| `active_provider_id` | Active Provider | many to one | `payment.provider` | computed by rule `_compute_active_provider_id` (not stored) |
| `has_enabled_provider` | Has Enabled Provider | boolean |  | computed by rule `_compute_has_enabled_provider` (not stored) |
| `onboarding_payment_module` | Onboarding Payment Module | selection |  | computed by rule `_compute_onboarding_payment_module` (not stored) |
| `pay_invoices_online` | Pay Invoices Online | boolean |  |  |
| `account_interco_clearing_journal_id` | Account Interco Clearing Journal | many to one | `account.journal` | related through path `company_id.account_interco_clearing_journal_id`; must belong to the same company |
| `account_interco_payable_id` | Account Interco Payable | many to one | `account.account` | related through path `company_id.account_interco_payable_id`; must belong to the same company |
| `account_interco_receivable_id` | Account Interco Receivable | many to one | `account.account` | related through path `company_id.account_interco_receivable_id`; must belong to the same company |
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | many to one |  | related through path `company_id.account_peppol_edi_user` |
| `account_peppol_edi_mode` | Account the pan-European public procurement online network Electronic data interchange Mode | selection |  | related through path `account_peppol_edi_user.edi_mode` |
| `account_peppol_contact_email` | Account the pan-European public procurement online network Contact Email | single line text |  | computed by rule `_compute_account_peppol_contact_email` (not stored); writable through an inverse rule |
| `account_peppol_eas` | Account the pan-European public procurement online network Eas | selection |  | related through path `company_id.peppol_eas` |
| `account_peppol_edi_identification` | Account the pan-European public procurement online network Electronic data interchange Identification | single line text |  | related through path `account_peppol_edi_user.edi_identification` |
| `account_peppol_endpoint` | Account the pan-European public procurement online network Endpoint | single line text |  | related through path `company_id.peppol_endpoint` |
| `account_peppol_migration_key` | Account the pan-European public procurement online network Migration Key | single line text |  | related through path `company_id.account_peppol_migration_key` |
| `account_peppol_phone_number` | Account the pan-European public procurement online network Phone Number | single line text |  | related through path `company_id.account_peppol_phone_number` |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `account_peppol_purchase_journal_id` | Account the pan-European public procurement online network Purchase Journal | many to one |  | related through path `company_id.peppol_purchase_journal_id` |
| `peppol_external_provider` | the pan-European public procurement online network External Provider | single line text |  | related through path `company_id.peppol_external_provider` |
| `peppol_purchase_journal_required` | the pan-European public procurement online network Purchase Journal Required | boolean |  | computed by rule `_compute_peppol_purchase_journal_required` (not stored) |
| `peppol_use_parent_company` | the pan-European public procurement online network Use Parent Company | boolean |  | computed by rule `_compute_peppol_use_parent_company` (not stored) |
| `peppol_parent_company_name` | the pan-European public procurement online network Parent Company Name | single line text |  | computed by rule `_compute_peppol_use_parent_company` (not stored) |
| `account_is_token_out_of_sync` | Account Is Token Out Of Sync | boolean |  | related through path `account_peppol_edi_user.is_token_out_of_sync` |
| `peppol_participation_role` | the pan-European public procurement online network Participation Role | selection |  | computed by rule `_compute_peppol_participation_role` (not stored); writable through an inverse rule |
| `ldaps` | directory access protocol Parameters | one to many |  | related through path `company_id.ldaps` |
| `auth_oauth_google_enabled` | Allow users to sign in with Google | boolean |  |  |
| `auth_oauth_google_client_id` | Client identifier | single line text |  |  |
| `server_uri_google` | Server uri | single line text |  |  |
| `minlength` | Minimum Password Length | integer |  | default ; Help: Minimum number of characters passwords must contain, set to 0 to disable. |
| `auth_totp_enforce` | Enforce two-factor authentication | boolean |  |  |
| `auth_totp_policy` | Two-factor authentication enforcing policy | selection |  |  |
| `geoloc_provider_id` | application programming interface | many to one | `base.geo_provider` | default computed dynamically (lambda x: x.env['base.geocoder']._get_provider()) |
| `geoloc_provider_techname` | Geoloc Provider Techname | single line text |  | read only; related through path `geoloc_provider_id.tech_name` |
| `geoloc_provider_googlemap_key` | Google Map application programming interface Key | single line text |  | Help: Visit https://developers.google.com/maps/documentation/geocoding/get-api-key for more information. |
| `vat_check_vies` | Verify value-added tax Numbers | boolean |  | related through path `company_id.vat_check_vies` |
| `cloud_storage_provider` | Cloud Storage Provider for new attachments | selection |  | extended by packages `cloud_storage_azure`, `cloud_storage_google` |
| `cloud_storage_min_file_size_mb` | Minimum File Size (MB) | float |  |  |
| `cloud_storage_min_file_size` | Minimum File Size (bytes) | integer |  | default computed dynamically (DEFAULT_CLOUD_STORAGE_MIN_FILE_SIZE); Help: webclient can upload files larger than the minimum file size         (in bytes) as url attachments to the server and then upload the file to         the cloud storage. |
| `cloud_storage_azure_account_name` | Azure Account Name | single line text |  |  |
| `cloud_storage_azure_container_name` | Azure Container Name | single line text |  |  |
| `cloud_storage_azure_tenant_id` | Azure Tenant identifier | single line text |  |  |
| `cloud_storage_azure_client_id` | Azure Client identifier | single line text |  |  |
| `cloud_storage_azure_client_secret` | Azure Client Secret | single line text |  |  |
| `cloud_storage_azure_invalidate_user_delegation_key` | Invalidate Cached Azure User Delegation Key | boolean |  |  |
| `cloud_storage_google_bucket_name` | Google Bucket Name | single line text |  |  |
| `cloud_storage_google_service_account_key` | Google Service Account Key | binary |  |  |
| `cloud_storage_google_account_info` | Google Service Account Info | single line text |  | computed by rule `_compute_cloud_storage_google_account_info` and stored |
| `cloud_storage_migration_progress` | Migration Progress | integer |  | Help: Shows the progress of cloud storage migration (current/total) |
| `cloud_storage_migration_message_model_ids` | Message Attachments | one to many | `ir.model` | computed by rule `_compute_cloud_storage_migration_message_model_ids` (not stored); writable through an inverse rule; Help: Migrate Models' Message Attachments |
| `cloud_storage_migration_message_models` | Cloud Storage Migration Message Models | single line text |  |  |
| `cloud_storage_migration_all_model_ids` | All Attachments | one to many | `ir.model` | computed by rule `_compute_cloud_storage_migration_all_model_ids` (not stored); writable through an inverse rule; Help: Migrate Models' All Attachments |
| `cloud_storage_migration_all_models` | Cloud Storage Migration All Models | single line text |  |  |
| `group_use_lead` | Leads | boolean |  |  |
| `group_use_recurring_revenues` | Recurring Revenues | boolean |  |  |
| `is_membership_multi` | Multi Teams | boolean |  |  |
| `module_partnership` | Membership / Partnership | boolean |  |  |
| `crm_use_auto_assignment` | Rule-Based Assignment | boolean |  |  |
| `crm_auto_assignment_action` | Auto Assignment Action | selection |  | computed by rule `_compute_crm_auto_assignment_data` and stored; Help: Manual assign allow to trigger assignment from team form view using an action button. Automatic configures a cron running repeatedly assignment in all teams. |
| `crm_auto_assignment_interval_type` | Auto Assignment Interval Unit | selection |  | computed by rule `_compute_crm_auto_assignment_data` and stored; Help: Interval type between each cron run (e.g. each 2 days or each 2 hours) |
| `crm_auto_assignment_interval_number` | Repeat every | integer |  | computed by rule `_compute_crm_auto_assignment_data` and stored; Help: Number of interval type between each cron run (e.g. each 2 days or each 4 days) |
| `crm_auto_assignment_run_datetime` | Auto Assignment Next Execution Date | date and time |  | computed by rule `_compute_crm_auto_assignment_data` and stored |
| `module_crm_iap_mine` | Generate new leads based on their country, industries, size, etc. | boolean |  |  |
| `module_crm_iap_enrich` | Enrich your leads automatically with company data based on their email address. | boolean |  |  |
| `module_website_crm_iap_reveal` | Create Leads/Opportunities from your website's traffic | boolean |  |  |
| `lead_enrich_auto` | Enrich lead automatically | selection |  | default `auto` |
| `lead_mining_in_pipeline` | Create a lead mining request directly from the opportunity pipeline. | boolean |  |  |
| `predictive_lead_scoring_start_date` | Lead Scoring Starting Date | date |  | computed by rule `_compute_pls_start_date` (not stored); writable through an inverse rule |
| `predictive_lead_scoring_start_date_str` | Lead Scoring Starting Date in String | single line text |  |  |
| `predictive_lead_scoring_fields` | Lead Scoring Frequency Fields | many to many | `crm.lead.scoring.frequency.field` | computed by rule `_compute_pls_fields` (not stored); writable through an inverse rule |
| `predictive_lead_scoring_fields_str` | Lead Scoring Frequency Fields in String | single line text |  |  |
| `predictive_lead_scoring_field_labels` | Predictive Lead Scoring Field Labels | single line text |  | computed by rule `_compute_predictive_lead_scoring_field_labels` (not stored) |
| `default_invoice_policy` | Invoicing Policy | selection |  | default `order` |
| `group_auto_done_setting` | Lock Confirmed Sales | boolean |  |  |
| `group_discount_per_so_line` | Discounts | boolean |  |  |
| `group_proforma_sales` | Pro-Forma Invoice | boolean |  | Help: Allows you to send pro-forma invoice. |
| `group_warning_sale` | Sale Order Warnings | boolean |  |  |
| `automatic_invoice` | Automatic Invoice | boolean |  | Help: The invoice is generated automatically and available in the customer portal when the transaction is confirmed by the payment provider. The invoice is marked as paid and the payment is registered in the payment journal defined in the configuration of the payment provider. This mode is advised if you issue the final invoice at the order and not after the delivery. |
| `invoice_mail_template_id` | Email Template | many to one | `mail.template` | restricted by domain `[["model", "=", "account.move"]]`; Help: Email sent to the customer once the invoice is available. |
| `quotation_validity_days` | Quotation Validity Days | integer |  | related through path `company_id.quotation_validity_days` |
| `portal_confirmation_sign` | Portal Confirmation Sign | boolean |  | related through path `company_id.portal_confirmation_sign` |
| `portal_confirmation_pay` | Portal Confirmation Pay | boolean |  | related through path `company_id.portal_confirmation_pay` |
| `prepayment_percent` | Prepayment Percent | float |  | related through path `company_id.prepayment_percent` |
| `downpayment_account_id` | Downpayment Account | many to one |  | related through path `company_id.downpayment_account_id` |
| `module_delivery` | Delivery Methods | boolean |  | extended by packages `stock` |
| `module_delivery_bpost` | bpost Connector | boolean |  | extended by packages `stock` |
| `module_delivery_dhl` | DHL Express Connector | boolean |  | extended by packages `stock` |
| `module_delivery_easypost` | Easypost Connector | boolean |  | extended by packages `stock` |
| `module_delivery_envia` | Envia.com Connector | boolean |  | extended by packages `stock` |
| `module_delivery_fedex_rest` | FedEx Connector | boolean |  | extended by packages `stock` |
| `module_delivery_sendcloud` | Sendcloud Connector | boolean |  | extended by packages `stock` |
| `module_delivery_shiprocket` | Shiprocket Connector | boolean |  | extended by packages `stock` |
| `module_delivery_starshipit` | Starshipit Connector | boolean |  | extended by packages `stock` |
| `module_delivery_ups_rest` | UPS Connector | boolean |  | extended by packages `stock` |
| `module_delivery_usps_rest` | USPS Connector | boolean |  | extended by packages `stock` |
| `module_product_email_template` | Specific Email | boolean |  |  |
| `module_sale_amazon` | Amazon Sync | boolean |  |  |
| `module_sale_commission` | Commissions | boolean |  |  |
| `module_sale_gelato` | Gelato | boolean |  |  |
| `module_sale_loyalty` | Coupons & Loyalty | boolean |  |  |
| `module_sale_margin` | Margins | boolean |  |  |
| `module_sale_pdf_quote_builder` | Portable Document Format Quote builder | boolean |  |  |
| `module_sale_product_matrix` | Sales Grid Entry | boolean |  |  |
| `module_sale_shopee` | Shopee Sync | boolean |  |  |
| `module_product_expiry` | Expiration Dates | boolean |  | Help: Track following dates on lots & serial numbers: best before, removal, end of life, alert.   Such dates are set automatically at lot/serial number creation based on values set on the product (in days). |
| `group_stock_production_lot` | Lots & Serial Numbers | boolean |  |  |
| `group_stock_lot_print_gs1` | Print Global Standards One Barcodes for Lots & Serial Numbers | boolean |  |  |
| `group_lot_on_delivery_slip` | Display Lots & Serial Numbers on Delivery Slips | boolean |  |  |
| `group_stock_tracking_lot` | Packages | boolean |  |  |
| `group_stock_tracking_owner` | Consignment | boolean |  |  |
| `group_stock_adv_location` | Multi-Step Routes | boolean |  | Help: Add and customize route operations to process product moves in your warehouse(s): e.g. unload > quality control > stock for incoming products, pick > pack > ship for outgoing products.   You can also set putaway strategies on warehouse locations in order to send incoming products into specific child locations straight away (e.g. specific bins, racks). |
| `group_warning_stock` | Warnings for Stock | boolean |  |  |
| `group_stock_sign_delivery` | Signature | boolean |  |  |
| `module_stock_picking_batch` | Batch, Wave & Cluster Transfers | boolean |  |  |
| `module_stock_barcode` | Barcode Scanner | boolean |  |  |
| `module_stock_barcode_barcodelookup` | Stock Barcode Database | boolean |  |  |
| `stock_move_email_validation` | Stock Move Email Validation | boolean |  | related through path `company_id.stock_move_email_validation` |
| `module_stock_sms` | text message Confirmation | boolean |  |  |
| `module_quality_control` | Quality | boolean |  | extended by packages `mrp` |
| `module_quality_control_worksheet` | Quality Worksheet | boolean |  | extended by packages `mrp` |
| `group_stock_multi_locations` | Storage Locations | boolean |  | Help: Store products in specific locations of your warehouse (e.g. bins, racks) and to track inventory accordingly. |
| `annual_inventory_month` | Annual Inventory Month | selection |  | related through path `company_id.annual_inventory_month` |
| `annual_inventory_day` | Annual Inventory Day | integer |  | related through path `company_id.annual_inventory_day` |
| `group_stock_reception_report` | Reception Report | boolean |  |  |
| `module_stock_dropshipping` | Dropshipping | boolean |  | extended by packages `purchase_stock` |
| `barcode_separator` | Separator | single line text |  | Help: Character(s) used to separate data contained within an aggregate barcode (i.e. a barcode containing multiple barcode encodings) |
| `module_stock_fleet` | Dispatch Management System | boolean |  |  |
| `replenish_on_order` | Replenish on Order (make to order) | boolean |  | computed by rule `_compute_replenish_on_order` (not stored); writable through an inverse rule |
| `stock_text_confirmation` | Stock Text Validation with stock move | boolean |  | related through path `company_id.stock_text_confirmation` |
| `stock_confirmation_type` | Stock Text Validation type | selection |  | related through path `company_id.stock_confirmation_type` |
| `horizon_days` | Horizon Days | float |  | related through path `company_id.horizon_days` |
| `module_stock_landed_costs` | Landed Costs | boolean |  | Help: Affect landed costs on reception operations and split them among products to update their cost price. |
| `group_lot_on_invoice` | Display Lots & Serial Numbers on Invoices | boolean |  |  |
| `security_lead` | Security Lead Time | float |  | related through path `company_id.security_lead` |
| `use_security_lead` | Security Lead Time for Sales | boolean |  | Help: Margin of error for dates promised to customers. Products will be scheduled for delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain. |
| `default_picking_policy` | Picking Policy | selection |  | required; default `direct` |
| `google_maps_static_api_key` | Google Maps application programming interface key | single line text |  | computed by rule `_compute_maps_static_api_key` and stored |
| `google_maps_static_api_secret` | Google Maps application programming interface secret | single line text |  | computed by rule `_compute_maps_static_api_secret` and stored |
| `module_event_sale` | Tickets with Sale | boolean |  |  |
| `module_pos_event` | Tickets with PoS | boolean |  |  |
| `module_website_event_track` | Tracks and Agenda | boolean |  |  |
| `module_website_event_track_live` | Live Mode | boolean |  |  |
| `module_website_event_track_quiz` | Quiz on Tracks | boolean |  |  |
| `module_website_event_exhibitor` | Advanced Sponsors | boolean |  |  |
| `use_event_barcode` | Use Event Barcode | boolean |  | Help: Enable or Disable Event Barcode functionality. |
| `barcode_nomenclature_id` | Barcode Nomenclature | many to one | `barcode.nomenclature` | related through path `company_id.nomenclature_id`; extended by packages `point_of_sale` |
| `module_website_event_sale` | Online Ticketing | boolean |  |  |
| `module_event_booth` | Booth Management | boolean |  |  |
| `use_google_maps_static_api` | Google Maps static application programming interface | boolean |  | default computed dynamically (_default_use_google_maps_static_api) |
| `group_sale_order_template` | Quotation Templates | boolean |  |  |
| `company_so_template_id` | Default Template | many to one |  | related through path `company_id.sale_order_template_id`; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `google_places_api_key` | Google Places application programming interface Key | single line text |  |  |
| `cal_client_id` | Client_id | single line text |  | default  |
| `cal_client_secret` | Client_key | single line text |  | default  |
| `cal_sync_paused` | Google Synchronization Paused | boolean |  | Help: Indicates if synchronization with Google Calendar is paused or not. |
| `google_gmail_client_identifier` | Gmail Client Id | single line text |  |  |
| `google_gmail_client_secret` | Gmail Client Secret | single line text |  |  |
| `enable_recaptcha` | Enable reCAPTCHA | boolean |  | visible only to groups `base.group_system` |
| `recaptcha_public_key` | Site Key | single line text |  | visible only to groups `base.group_system` |
| `recaptcha_private_key` | Secret Key | single line text |  | visible only to groups `base.group_system` |
| `recaptcha_min_score` | Minimum score | float |  | default `0.7`; visible only to groups `base.group_system`; Help: By default, should be one of 0.1, 0.3, 0.7, 0.9. 1.0 is very likely a good interaction, 0.0 is very likely a bot |
| `resource_calendar_id` | Company Working Hours | many to one | `resource.calendar` | related through path `company_id.resource_calendar_id` |
| `module_hr_presence` | Advanced Presence Control | boolean |  |  |
| `module_hr_skills` | Skills Management | boolean |  |  |
| `hr_presence_control_login` | Human resources Presence Control Login | boolean |  | related through path `company_id.hr_presence_control_login` |
| `hr_presence_control_email` | Human resources Presence Control Email | boolean |  | related through path `company_id.hr_presence_control_email` |
| `hr_presence_control_ip` | Human resources Presence Control Internet protocol | boolean |  | related through path `company_id.hr_presence_control_ip` |
| `module_hr_attendance` | Module Human resources Attendance | boolean |  | related through path `company_id.hr_presence_control_attendance` |
| `hr_presence_control_email_amount` | Human resources Presence Control Email Amount | integer |  | related through path `company_id.hr_presence_control_email_amount` |
| `hr_presence_control_ip_list` | Human resources Presence Control Internet protocol List | single line text |  | related through path `company_id.hr_presence_control_ip_list` |
| `contract_expiration_notice_period` | Contract Expiry Notice Period | integer |  | related through path `company_id.contract_expiration_notice_period` |
| `work_permit_expiration_notice_period` | Work Permit Expiry Notice Period | integer |  | related through path `company_id.work_permit_expiration_notice_period` |
| `overtime_company_threshold` | Tolerance Time In Favor Of Company | integer |  |  |
| `overtime_employee_threshold` | Tolerance Time In Favor Of Employee | integer |  |  |
| `hr_attendance_display_overtime` | Human resources Attendance Display Overtime | boolean |  | related through path `company_id.hr_attendance_display_overtime` |
| `attendance_kiosk_mode` | Attendance Kiosk Mode | selection |  | related through path `company_id.attendance_kiosk_mode` |
| `attendance_barcode_source` | Attendance Barcode Source | selection |  | related through path `company_id.attendance_barcode_source` |
| `attendance_kiosk_delay` | Attendance Kiosk Delay | integer |  | related through path `company_id.attendance_kiosk_delay` |
| `attendance_kiosk_url` | Attendance Kiosk Uniform resource locator | single line text |  | related through path `company_id.attendance_kiosk_url` |
| `attendance_kiosk_use_pin` | Attendance Kiosk Use Pin | boolean |  | related through path `company_id.attendance_kiosk_use_pin` |
| `attendance_from_systray` | Attendance From Systray | boolean |  | related through path `company_id.attendance_from_systray` |
| `attendance_overtime_validation` | Attendance Overtime Validation | selection |  | related through path `company_id.attendance_overtime_validation` |
| `auto_check_out` | Auto Check Out | boolean |  | related through path `company_id.auto_check_out` |
| `auto_check_out_tolerance` | Auto Check Out Tolerance | float |  | related through path `company_id.auto_check_out_tolerance` |
| `absence_management` | Absence Management | boolean |  | related through path `company_id.absence_management` |
| `attendance_device_tracking` | Attendance Device Tracking | boolean |  | related through path `company_id.attendance_device_tracking` |
| `hr_expense_alias_prefix` | Default Alias Name for Expenses | single line text |  | computed by rule `_compute_hr_expense_alias_prefix` and stored |
| `hr_expense_alias_domain_id` | Human resources Expense Alias Domain | many to one | `mail.alias.domain` | computed by rule `_compute_hr_expense_alias_domain_id` (not stored); writable through an inverse rule |
| `hr_expense_use_mailgateway` | Let your employees record expenses by email | boolean |  |  |
| `module_hr_payroll_expense` | Reimburse Expenses in Payslip | boolean |  |  |
| `module_hr_expense_extract` | Send bills to optical character recognition to generate expenses | boolean |  |  |
| `module_hr_expense_stripe` | Link your stripe issuing account to manage company credit cards for your employees through the system | boolean |  |  |
| `expense_journal_id` | Expense Journal | many to one | `account.journal` | related through path `company_id.expense_journal_id`; restricted by domain `[('type', '=', 'purchase')]`; must belong to the same company |
| `company_expense_allowed_payment_method_line_ids` | Company Expense Allowed Payment Method Line | many to many | `account.payment.method.line` | related through path `company_id.company_expense_allowed_payment_method_line_ids`; must belong to the same company |
| `module_maintenance_worksheet` | Custom Maintenance Worksheets | boolean |  |  |
| `module_website_hr_recruitment` | Online Posting | boolean |  |  |
| `module_hr_recruitment_survey` | Interview Forms | boolean |  |  |
| `module_hr_recruitment_extract` | Send CV to optical character recognition to fill applications | boolean |  |  |
| `website_id` | website | many to one | `website` | default computed dynamically (_default_website); on delete of the target: cascade |
| `website_name` | Website Name | single line text |  | related through path `website_id.name` |
| `website_domain` | Website Domain | single line text |  | related through path `website_id.domain` |
| `website_homepage_url` | Website Homepage Uniform resource locator | single line text |  | related through path `website_id.homepage_url` |
| `website_company_id` | Website Company | many to one |  | related through path `website_id.company_id` |
| `website_logo` | Website Logo | binary |  | related through path `website_id.logo` |
| `language_ids` | Language | many to many |  | related through path `website_id.language_ids` |
| `website_language_count` | Number of languages | integer |  | read only; related through path `website_id.language_count` |
| `website_default_lang_id` | Default language | many to one |  | related through path `website_id.default_lang_id` |
| `website_default_lang_code` | Default language code | single line text |  | related through path `website_id.default_lang_id.code` |
| `shared_user_account` | Shared Customer Accounts | boolean |  | computed by rule `_compute_shared_user_account` (not stored); writable through an inverse rule |
| `website_cookies_bar` | Website Cookies Bar | boolean |  | related through path `website_id.cookies_bar` |
| `website_block_third_party_domains` | Block 3rd-party domains | boolean |  | related through path `website_id.block_third_party_domains` |
| `google_analytics_key` | Google Analytics Key | single line text |  | related through path `website_id.google_analytics_key` |
| `google_search_console` | Google Search Console Key | single line text |  | related through path `website_id.google_search_console` |
| `plausible_shared_key` | Plausible auth Key | single line text |  | related through path `website_id.plausible_shared_key` |
| `plausible_site` | Plausible Site (e.g. domain.com) | single line text |  | related through path `website_id.plausible_site` |
| `cdn_activated` | Cdn Activated | boolean |  | related through path `website_id.cdn_activated` |
| `cdn_url` | Cdn Uniform resource locator | single line text |  | related through path `website_id.cdn_url` |
| `cdn_filters` | Cdn Filters | multi line text |  | related through path `website_id.cdn_filters` |
| `favicon` | Favicon | binary |  | related through path `website_id.favicon` |
| `social_default_image` | Default Social Share Image | binary |  | related through path `website_id.social_default_image` |
| `group_multi_website` | Multi-website | boolean |  |  |
| `has_google_analytics` | Google Analytics | boolean |  | computed by rule `_compute_has_google_analytics` (not stored); writable through an inverse rule |
| `has_google_search_console` | Google Search Console | boolean |  | computed by rule `_compute_has_google_search_console` (not stored); writable through an inverse rule |
| `has_default_share_image` | Use a image by default for sharing | boolean |  | computed by rule `_compute_has_default_share_image` (not stored); writable through an inverse rule |
| `has_plausible_shared_key` | Plausible Analytics | boolean |  | computed by rule `_compute_has_plausible_shared_key` (not stored); writable through an inverse rule |
| `module_website_livechat` | Module Website Livechat | boolean |  |  |
| `website_slide_google_app_key` | Website Slide Google App Key | single line text |  | related through path `website_id.website_slide_google_app_key` |
| `module_website_sale_slides` | Sell on eCommerce | boolean |  |  |
| `module_website_slides_forum` | Forum | boolean |  |  |
| `module_website_slides_survey` | Certifications | boolean |  |  |
| `module_mass_mailing_slides` | Mailing | boolean |  |  |
| `module_hr_timesheet` | Task Logs | boolean |  |  |
| `group_project_stages` | Project Stages | boolean |  |  |
| `module_project_timesheet_holidays` | Time Off | boolean |  | computed by rule `_compute_timesheet_modules` and stored |
| `reminder_user_allow` | Employee Reminder | boolean |  |  |
| `reminder_allow` | Approver Reminder | boolean |  |  |
| `project_time_mode_id` | Project Time Unit | many to one | `uom.uom` | related through path `company_id.project_time_mode_id`; Help: This will set the unit of measure used in projects and tasks. If you use the timesheet linked to projects, don't forget to setup the right unit of measure in your employees. |
| `is_encode_uom_days` | Is Encode Unit of measure Days | boolean |  | computed by rule `_compute_is_encode_uom_days` (not stored) |
| `timesheet_encode_method` | Encoding Method | selection |  | required; computed by rule `_compute_timesheet_encode_method` (not stored); writable through an inverse rule |
| `withholding_tax_base_account_id` | Withholding Tax Base Account | many to one |  | related through path `company_id.withholding_tax_base_account_id` |
| `partner_autocomplete_insufficient_credit` | Insufficient credit | boolean |  | computed by rule `_compute_partner_autocomplete_insufficient_credit` (not stored) |
| `pos_config_id` | Point of Sale | many to one | `pos.config` | default computed dynamically (lambda self: self._default_pos_config()) |
| `module_pos_adyen` | Adyen Payment Terminal | boolean |  | Help: The transactions are processed by Adyen. Set your Adyen credentials on the related payment method. |
| `module_pos_stripe` | Stripe Payment Terminal | boolean |  | Help: The transactions are processed by Stripe. Set your Stripe credentials on the related payment method. |
| `module_pos_viva_com` | Viva.com Payment Terminal | boolean |  | Help: The transactions are processed by Viva.com on terminal or tap on phone. |
| `module_pos_razorpay` | Razorpay Payment Terminal | boolean |  | Help: The transactions are processed by Razorpay. Set your Razorpay credentials on the related payment method. |
| `module_pos_mercado_pago` | Mercado Pago Payment Terminal | boolean |  | Help: The transactions are processed by Mercado Pago. Set your Mercado Pago credentials on the related payment method. |
| `module_pos_pine_labs` | Pine Labs Payment Terminal | boolean |  | Help: The transactions are processed by Pine Labs. Set your Pine Labs credentials on the related payment method. |
| `module_pos_qfpay` | QFPay Payment Terminal | boolean |  | Help: The transactions are processed by QFPay. Set your QFPay credentials on the related payment method. |
| `module_pos_pricer` | Pricer electronic price tags | boolean |  | Help: Display the price of your products through electronic price tags |
| `update_stock_quantities` | Update Stock Quantities | selection |  | related through path `company_id.point_of_sale_update_stock_quantities` |
| `account_default_pos_receivable_account_id` | Default Account Receivable (PoS) | many to one |  | related through path `company_id.account_default_pos_receivable_account_id`; must belong to the same company |
| `is_kiosk_mode` | Is Kiosk Mode | boolean |  | default  |
| `pos_customer_display_bg_img` | Point of sale Customer Display Bg Img | image |  | related through path `pos_config_id.customer_display_bg_img` |
| `pos_customer_display_bg_img_name` | Point of sale Customer Display Bg Img Name | single line text |  | related through path `pos_config_id.customer_display_bg_img_name` |
| `pos_use_presets` | Point of sale Use Presets | boolean |  | related through path `pos_config_id.use_presets` |
| `pos_default_preset_id` | Point of sale Default Preset | many to one | `pos.preset` | related through path `pos_config_id.default_preset_id` |
| `pos_available_preset_ids` | Point of sale Available Preset | many to many | `pos.preset` | related through path `pos_config_id.available_preset_ids` |
| `pos_module_pos_discount` | Point of sale Module Point of sale Discount | boolean |  | related through path `pos_config_id.module_pos_discount` |
| `pos_module_pos_hr` | Point of sale Module Point of sale Human resources | boolean |  | related through path `pos_config_id.module_pos_hr` |
| `pos_module_pos_restaurant` | Point of sale Module Point of sale Restaurant | boolean |  | related through path `pos_config_id.module_pos_restaurant` |
| `pos_module_pos_appointment` | Point of sale Module Point of sale Appointment | boolean |  | related through path `pos_config_id.module_pos_appointment` |
| `pos_module_pos_avatax` | Point of sale Module Point of sale Avatax | boolean |  | related through path `pos_config_id.module_pos_avatax` |
| `pos_is_order_printer` | Point of sale Is Order Printer | boolean |  | computed by rule `_compute_pos_printer` and stored |
| `pos_printer_ids` | Point of sale Printer | many to many |  | related through path `pos_config_id.printer_ids` |
| `pos_allowed_pricelist_ids` | Point of sale Allowed Pricelist | many to many | `product.pricelist` | computed by rule `_compute_pos_allowed_pricelist_ids` (not stored) |
| `pos_amount_authorized_diff` | Point of sale Amount Authorized Diff | float |  | related through path `pos_config_id.amount_authorized_diff` |
| `pos_available_pricelist_ids` | Available Pricelists | many to many | `product.pricelist` | computed by rule `_compute_pos_pricelist_id` and stored |
| `pos_cash_control` | Point of sale Cash Control | boolean |  | related through path `pos_config_id.cash_control` |
| `pos_cash_rounding` | Cash Rounding (PoS) | boolean |  | related through path `pos_config_id.cash_rounding` |
| `pos_company_has_template` | Point of sale Company Has Template | boolean |  | related through path `pos_config_id.company_has_template` |
| `pos_default_bill_ids` | Point of sale Default Bill | many to many |  | related through path `pos_config_id.default_bill_ids` |
| `pos_default_fiscal_position_id` | Default Fiscal Position | many to one | `account.fiscal.position` | computed by rule `_compute_pos_fiscal_positions` and stored; must belong to the same company |
| `pos_fiscal_position_ids` | Fiscal Positions | many to many | `account.fiscal.position` | computed by rule `_compute_pos_fiscal_positions` and stored; must belong to the same company |
| `pos_has_active_session` | Point of sale Has Active Session | boolean |  | related through path `pos_config_id.has_active_session` |
| `pos_iface_available_categ_ids` | Available PoS Product Categories | many to many | `pos.category` | computed by rule `_compute_pos_iface_available_categ_ids` and stored |
| `pos_iface_big_scrollbars` | Point of sale Iface Big Scrollbars | boolean |  | related through path `pos_config_id.iface_big_scrollbars` |
| `pos_iface_group_by_categ` | Point of sale Iface Group By Categ | boolean |  | related through path `pos_config_id.iface_group_by_categ` |
| `pos_iface_cashdrawer` | Cashdrawer | boolean |  | computed by rule `_compute_pos_iface_cashdrawer` and stored |
| `pos_iface_electronic_scale` | Electronic Scale | boolean |  | computed by rule `_compute_pos_iface_electronic_scale` and stored |
| `pos_iface_print_auto` | Point of sale Iface Print Auto | boolean |  | related through path `pos_config_id.iface_print_auto` |
| `pos_iface_print_skip_screen` | Point of sale Iface Print Skip Screen | boolean |  | related through path `pos_config_id.iface_print_skip_screen` |
| `pos_iface_print_via_proxy` | Print via Proxy | boolean |  | computed by rule `_compute_pos_iface_print_via_proxy` and stored |
| `pos_iface_scan_via_proxy` | Scan via Proxy | boolean |  | computed by rule `_compute_pos_iface_scan_via_proxy` and stored |
| `pos_iface_tax_included` | Point of sale Iface Tax Included | selection |  | related through path `pos_config_id.iface_tax_included` |
| `pos_iface_tipproduct` | Point of sale Iface Tipproduct | boolean |  | related through path `pos_config_id.iface_tipproduct` |
| `pos_invoice_journal_id` | Point of sale Invoice Journal | many to one |  | related through path `pos_config_id.invoice_journal_id` |
| `pos_is_header_or_footer` | Point of sale Is Header Or Footer | boolean |  | related through path `pos_config_id.is_header_or_footer` |
| `pos_is_margins_costs_accessible_to_every_user` | Point of sale Is Margins Costs Accessible To Every User | boolean |  | related through path `pos_config_id.is_margins_costs_accessible_to_every_user` |
| `pos_is_posbox` | Point of sale Is Posbox | boolean |  | related through path `pos_config_id.is_posbox` |
| `pos_journal_id` | Point of sale Journal | many to one |  | related through path `pos_config_id.journal_id` |
| `pos_limit_categories` | Point of sale Limit Categories | boolean |  | related through path `pos_config_id.limit_categories` |
| `pos_manual_discount` | Point of sale Manual Discount | boolean |  | related through path `pos_config_id.manual_discount` |
| `pos_only_round_cash_method` | Point of sale Only Round Cash Method | boolean |  | related through path `pos_config_id.only_round_cash_method` |
| `pos_other_devices` | Point of sale Other Devices | boolean |  | related through path `pos_config_id.other_devices` |
| `pos_payment_method_ids` | Point of sale Payment Method | many to many |  | related through path `pos_config_id.payment_method_ids` |
| `pos_picking_policy` | Point of sale Picking Policy | selection |  | related through path `pos_config_id.picking_policy` |
| `pos_picking_type_id` | Point of sale Picking Type | many to one |  | related through path `pos_config_id.picking_type_id` |
| `pos_pricelist_id` | Default Pricelist | many to one | `product.pricelist` | computed by rule `_compute_pos_pricelist_id` and stored |
| `pos_proxy_ip` | internet protocol Address | single line text |  | related through path `pos_config_id.proxy_ip` |
| `pos_receipt_footer` | Receipt Footer | multi line text |  | computed by rule `_compute_pos_receipt_header_footer` and stored |
| `pos_receipt_header` | Receipt Header | multi line text |  | computed by rule `_compute_pos_receipt_header_footer` and stored |
| `pos_restrict_price_control` | Point of sale Restrict Price Control | boolean |  | related through path `pos_config_id.restrict_price_control` |
| `pos_rounding_method` | Point of sale Rounding Method | many to one |  | related through path `pos_config_id.rounding_method` |
| `pos_route_id` | Point of sale Route | many to one |  | related through path `pos_config_id.route_id` |
| `pos_selectable_categ_ids` | Point of sale Selectable Categ | many to many | `pos.category` | computed by rule `_compute_pos_selectable_categ_ids` (not stored) |
| `pos_set_maximum_difference` | Point of sale Set Maximum Difference | boolean |  | related through path `pos_config_id.set_maximum_difference` |
| `pos_ship_later` | Point of sale Ship Later | boolean |  | related through path `pos_config_id.ship_later` |
| `pos_tax_regime_selection` | Point of sale Tax Regime Selection | boolean |  | related through path `pos_config_id.tax_regime_selection` |
| `pos_tip_product_id` | Tip Product | many to one | `product.product` | computed by rule `_compute_pos_tip_product_id` and stored |
| `pos_use_pricelist` | Point of sale Use Pricelist | boolean |  | related through path `pos_config_id.use_pricelist` |
| `pos_warehouse_id` | Warehouse (PoS) | many to one |  | related through path `pos_config_id.warehouse_id` |
| `point_of_sale_use_ticket_qr_code` | Point Of Sale Use Ticket Quick response Code | boolean |  | related through path `company_id.point_of_sale_use_ticket_qr_code` |
| `pos_auto_validate_terminal_payment` | Automatically validates orders paid with a payment terminal. | boolean |  | related through path `pos_config_id.auto_validate_terminal_payment` |
| `pos_trusted_config_ids` | Point of sale Trusted Config | many to many |  | related through path `pos_config_id.trusted_config_ids`; restricted by domain `[('id', '!=', pos_config_id), ('module_pos_restaurant', '=', False)]` |
| `point_of_sale_ticket_unique_code` | Point Of Sale Ticket Unique Code | boolean |  | related through path `company_id.point_of_sale_ticket_unique_code` |
| `pos_show_product_images` | Point of sale Show Product Images | boolean |  | related through path `pos_config_id.show_product_images` |
| `pos_show_category_images` | Point of sale Show Category Images | boolean |  | related through path `pos_config_id.show_category_images` |
| `point_of_sale_ticket_portal_url_display_mode` | Point Of Sale Ticket Portal Uniform resource locator Display Mode | selection |  | required; related through path `company_id.point_of_sale_ticket_portal_url_display_mode` |
| `pos_note_ids` | Point of sale Note | many to many |  | related through path `pos_config_id.note_ids` |
| `pos_module_pos_sms` | Point of sale Module Point of sale Text message | boolean |  | related through path `pos_config_id.module_pos_sms` |
| `pos_is_closing_entry_by_product` | Point of sale Is Closing Entry By Product | boolean |  | related through path `pos_config_id.is_closing_entry_by_product` |
| `pos_order_edit_tracking` | Point of sale Order Edit Tracking | boolean |  | related through path `pos_config_id.order_edit_tracking` |
| `pos_basic_receipt` | Point of sale Basic Receipt | boolean |  | related through path `pos_config_id.basic_receipt` |
| `pos_fallback_nomenclature_id` | Point of sale Fallback Nomenclature | many to one |  | related through path `pos_config_id.fallback_nomenclature_id`; restricted by domain `[('id', '!=', barcode_nomenclature_id)]` |
| `group_pos_preset` | Presets | boolean |  | Help: Hide or show the Presets menu in the Point of Sale configuration. |
| `pos_epson_printer_ip` | Point of sale Epson Printer Internet protocol | single line text |  | related through path `pos_config_id.epson_printer_ip` |
| `pos_use_fast_payment` | Point of sale Use Fast Payment | boolean |  | related through path `pos_config_id.use_fast_payment` |
| `pos_fast_payment_method_ids` | Point of sale Fast Payment Method | many to many |  | related through path `pos_config_id.fast_payment_method_ids` |
| `l10n_gcc_dual_language_invoice` | Localization Gcc Dual Language Invoice | boolean |  | related through path `company_id.l10n_gcc_dual_language_invoice` |
| `l10n_gcc_country_is_gcc` | Localization Gcc Country Is Gcc | boolean |  | related through path `company_id.l10n_gcc_country_is_gcc` |
| `l10n_gcc_dual_language_receipt` | Localization Gcc Dual Language Receipt | boolean |  | related through path `pos_config_id.l10n_gcc_dual_language_receipt` |
| `group_show_uom_price` | Base Unit Price | boolean |  | default  |
| `group_product_price_comparison` | Comparison Price | boolean |  | Help: Add a strikethrough price to your /shop and product pages for comparison purposes.It will not be displayed if pricelists apply. |
| `group_gmc_feed` | Google Merchant Center | boolean |  | related through path `website_id.enabled_gmc_src` |
| `module_website_sale_autocomplete` | Address Autocomplete | boolean |  |  |
| `module_website_sale_collect` | Click & Collect | boolean |  |  |
| `add_to_cart_action` | Add To Cart Action | selection |  | related through path `website_id.add_to_cart_action` |
| `cart_recovery_mail_template` | Cart Recovery Mail Template | many to one |  | related through path `website_id.cart_recovery_mail_template_id` |
| `cart_abandoned_delay` | Cart Abandoned Delay | float |  | related through path `website_id.cart_abandoned_delay` |
| `send_abandoned_cart_email` | Abandoned Email | boolean |  | related through path `website_id.send_abandoned_cart_email` |
| `salesperson_id` | Salesperson | many to one |  | related through path `website_id.salesperson_id` |
| `salesteam_id` | Salesteam | many to one |  | related through path `website_id.salesteam_id` |
| `website_sale_prevent_zero_price_sale` | Prevent Sale of Zero Priced Product | boolean |  | related through path `website_id.prevent_zero_price_sale` |
| `website_sale_contact_us_button_url` | Button Url | single line text |  | related through path `website_id.contact_us_button_url` |
| `show_line_subtotals_tax_selection` | Show Line Subtotals Tax Selection | selection |  | related through path `website_id.show_line_subtotals_tax_selection` |
| `confirmation_email_template_id` | Confirmation Email Template | many to one |  | related through path `website_id.confirmation_email_template_id` |
| `account_on_checkout` | Customer Accounts | selection |  | required; computed by rule `_compute_account_on_checkout` (not stored); writable through an inverse rule |
| `ecommerce_access` | Ecommerce Access | selection |  | related through path `website_id.ecommerce_access` |
| `l10n_ar_website_sale_show_both_prices` | Localization Ar Website Sale Show Both Prices | boolean |  | related through path `website_id.l10n_ar_website_sale_show_both_prices` |
| `l10n_ar_tax_base_account_id` | Tax Base Account | many to one | `account.account` | related through path `company_id.l10n_ar_tax_base_account_id`; Help: Account that will be set on lines created to represent the tax base amounts. |
| `has_position_column` | Has Position Column | boolean |  | related through path `company_id.has_position_column` |
| `pos_floor_ids` | Point of sale Floor | many to many |  | related through path `pos_config_id.floor_ids` |
| `pos_iface_printbill` | Point of sale Iface Printbill | boolean |  | computed by rule `_compute_pos_module_pos_restaurant` and stored |
| `pos_iface_splitbill` | Point of sale Iface Splitbill | boolean |  | computed by rule `_compute_pos_module_pos_restaurant` and stored |
| `pos_set_tip_after_payment` | Point of sale Set Tip After Payment | boolean |  | computed by rule `_compute_pos_set_tip_after_payment` and stored |
| `pos_default_screen` | Point of sale Default Screen | selection |  | related through path `pos_config_id.default_screen` |
| `pos_crm_team_id` | Sales Team (PoS) | many to one |  | related through path `pos_config_id.crm_team_id` |
| `pos_down_payment_product_id` | Point of sale Down Payment Product | many to one |  | related through path `pos_config_id.down_payment_product_id` |
| `lock_confirmed_po` | Lock Confirmed Orders | boolean |  | default computed dynamically (lambda self: self.env.company.po_lock == 'lock') |
| `po_lock` | Purchase Order Modification * | selection |  | related through path `company_id.po_lock` |
| `po_order_approval` | Purchase Order Approval | boolean |  | default computed dynamically (lambda self: self.env.company.po_double_validation == 'two_step') |
| `po_double_validation` | Levels of Approvals * | selection |  | related through path `company_id.po_double_validation` |
| `po_double_validation_amount` | Minimum Amount | monetary |  | related through path `company_id.po_double_validation_amount`; currency taken from `company_currency_id` |
| `company_currency_id` | Company Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id` |
| `group_warning_purchase` | Purchase Warnings | boolean |  |  |
| `module_account_3way_match` | 3-way matching: purchases, receptions and bills | boolean |  |  |
| `module_purchase_requisition` | Purchase Agreements | boolean |  |  |
| `module_purchase_product_matrix` | Purchase Grid Entry | boolean |  |  |
| `group_send_reminder` | Receipt Reminder | boolean |  | default `True`; Help: Allow automatically send email to remind your vendor the receipt date |
| `nemhandel_edi_user` | Nemhandel Electronic data interchange User | many to one |  | related through path `company_id.nemhandel_edi_user` |
| `nemhandel_edi_mode` | Nemhandel electronic data interchange operating mode | selection |  | related through path `nemhandel_edi_user.edi_mode` |
| `nemhandel_contact_email` | Nemhandel Contact Email | single line text |  | related through path `company_id.nemhandel_contact_email` |
| `nemhandel_identifier_type` | Nemhandel Identifier Type | selection |  | related through path `company_id.nemhandel_identifier_type` |
| `nemhandel_identifier_value` | Nemhandel Identifier Value | single line text |  | related through path `company_id.nemhandel_identifier_value` |
| `nemhandel_edi_identification` | Nemhandel identification | single line text |  | related through path `nemhandel_edi_user.edi_identification` |
| `nemhandel_phone_number` | Nemhandel Phone Number | single line text |  | related through path `company_id.nemhandel_phone_number` |
| `l10n_dk_nemhandel_proxy_state` | Localization Dk Nemhandel Proxy State | selection |  | related through path `company_id.l10n_dk_nemhandel_proxy_state` |
| `nemhandel_purchase_journal_id` | Nemhandel Purchase Journal | many to one |  | related through path `company_id.nemhandel_purchase_journal_id` |
| `l10n_eg_client_identifier` | Localization Eg Client Identifier | single line text |  | related through path `company_id.l10n_eg_client_identifier` |
| `l10n_eg_client_secret` | Localization Eg Client Secret | single line text |  | related through path `company_id.l10n_eg_client_secret` |
| `l10n_eg_production_env` | Localization Eg Production Env | boolean |  | related through path `company_id.l10n_eg_production_env` |
| `l10n_eg_invoicing_threshold` | Localization Eg Invoicing Threshold | float |  | related through path `company_id.l10n_eg_invoicing_threshold` |
| `l10n_es_simplified_invoice_limit` | Localization Es Simplified Invoice Limit | float |  | related through path `company_id.l10n_es_simplified_invoice_limit` |
| `l10n_es_sii_certificate_ids` | Localization Es Immediate supply of information Certificate | one to many |  | related through path `company_id.l10n_es_sii_certificate_ids` |
| `l10n_es_sii_tax_agency` | Localization Es Immediate supply of information Tax Agency | selection |  | related through path `company_id.l10n_es_sii_tax_agency` |
| `l10n_es_sii_test_env` | Localization Es Immediate supply of information Test Env | boolean |  | related through path `company_id.l10n_es_sii_test_env` |
| `l10n_es_tbai_certificate_ids` | Localization Es electronic invoicing (Basque) Certificate | one to many |  | related through path `company_id.l10n_es_tbai_certificate_ids` |
| `l10n_es_tbai_tax_agency` | Localization Es electronic invoicing (Basque) Tax Agency | selection |  | related through path `company_id.l10n_es_tbai_tax_agency` |
| `l10n_es_tbai_test_env` | Localization Es electronic invoicing (Basque) Test Env | boolean |  | related through path `company_id.l10n_es_tbai_test_env` |
| `l10n_es_edi_verifactu_required` | Localization Es Electronic data interchange Verifactu Required | boolean |  | related through path `company_id.l10n_es_edi_verifactu_required` |
| `l10n_es_edi_verifactu_certificate_ids` | Localization Es Electronic data interchange Verifactu Certificate | one to many |  | related through path `company_id.l10n_es_edi_verifactu_certificate_ids` |
| `l10n_es_edi_verifactu_test_environment` | Localization Es Electronic data interchange Verifactu Test Environment | boolean |  | related through path `company_id.l10n_es_edi_verifactu_test_environment` |
| `l10n_es_edi_verifactu_special_vat_regime` | Localization Es Electronic data interchange Verifactu Special Value-added tax Regime | selection |  | related through path `company_id.l10n_es_edi_verifactu_special_vat_regime` |
| `pos_l10n_es_simplified_invoice_journal_id` | Point of sale Localization Es Simplified Invoice Journal | many to one |  | related through path `pos_config_id.l10n_es_simplified_invoice_journal_id` |
| `l10n_eu_oss_eu_country` | Is European country? | boolean |  | computed by rule `_compute_l10n_eu_oss_european_country` (not stored) |
| `l10n_fr_reference_leave_type` | Localization Fr Reference Leave Type | many to one | `hr.leave.type` | related through path `company_id.l10n_fr_reference_leave_type` |
| `l10n_fr_pdp_pilot_phase` | Pilot Phase | boolean |  | computed by rule `_compute_l10n_fr_pdp_pilot_phase` (not stored); writable through an inverse rule; Help: Participate in the Pilot Phase of the French E-Invoicing. This way you are able to test it before it becomes mandatory. |
| `l10n_fr_pdp_send_to_ppf` | Send to PPF | boolean |  | related through path `company_id.l10n_fr_pdp_send_to_ppf`; Help: Activate Flux 1 regulatory data, Flux 6 mandatory statuses and Flux 10 e-reporting generation for this company. |
| `l10n_fr_pdp_annuaire_start_date` | Annuaire Start Date | date |  | related through path `company_id.l10n_fr_pdp_annuaire_start_date`; Help: The date on which the company is registered on the annuaire for the French e-invoicing. |
| `l10n_fr_pdp_registered` | Approved Platform Registered | boolean |  | related through path `company_id.l10n_fr_pdp_registered` |
| `l10n_fr_pdp_periodicity` | Flow 10 Report Periodicity | selection |  | required; related through path `company_id.l10n_fr_pdp_periodicity`; Help: Legal reporting period for transaction and payments flows according to the TVA regime table.         Real Monthly Normal Regime : transactions reported by decade, payments reported monthly         Real Normal Quarterly Regime : transactions reported monthly, payments reported monthly         Simplified VAT Regime (Monthly) : transactions reported monthly, payments reported monthly         Franchised VAT Regime (Bimonthly) : transactions reported bimonthly, payments reported bimonthly |
| `l10n_gr_edi_aade_id` | Localization Gr Electronic data interchange Aade | single line text |  | related through path `company_id.l10n_gr_edi_aade_id` |
| `l10n_gr_edi_aade_key` | Localization Gr Electronic data interchange Aade Key | single line text |  | related through path `company_id.l10n_gr_edi_aade_key` |
| `l10n_gr_edi_branch_number` | Localization Gr Electronic data interchange Branch Number | integer |  | related through path `company_id.l10n_gr_edi_branch_number` |
| `l10n_gr_edi_test_env` | Localization Gr Electronic data interchange Test Env | boolean |  | related through path `company_id.l10n_gr_edi_test_env` |
| `l10n_hr_mer_connection_state` | Localization Human resources Mer Connection State | selection |  | related through path `company_id.l10n_hr_mer_connection_state` |
| `l10n_hr_mer_connection_mode` | Localization Human resources Mer Connection Mode | selection |  | related through path `company_id.l10n_hr_mer_connection_mode` |
| `l10n_hr_mer_username` | Localization Human resources Mer Username | single line text |  | related through path `company_id.l10n_hr_mer_username` |
| `l10n_hr_mer_password` | Localization Human resources Mer Password | single line text |  | related through path `company_id.l10n_hr_mer_password` |
| `l10n_hr_mer_company_ident` | Localization Human resources Mer Company Ident | single line text |  | related through path `company_id.l10n_hr_mer_company_ident` |
| `l10n_hr_mer_company_bu` | Localization Human resources Mer Company Bu | single line text |  | related through path `company_id.partner_id.l10n_hr_business_unit_code` |
| `l10n_hr_mer_software_ident` | Localization Human resources Mer Software Ident | single line text |  | related through path `company_id.l10n_hr_mer_software_ident` |
| `l10n_hr_mer_purchase_journal_id` | Localization Human resources Mer Purchase Journal | many to one |  | related through path `company_id.l10n_hr_mer_purchase_journal_id` |
| `l10n_hu_tax_regime` | Localization Hu Tax Regime | selection |  | related through path `company_id.l10n_hu_tax_regime` |
| `l10n_hu_edi_server_mode` | Localization Hu Electronic data interchange Server Mode | selection |  | related through path `company_id.l10n_hu_edi_server_mode` |
| `l10n_hu_edi_username` | Localization Hu Electronic data interchange Username | single line text |  | related through path `company_id.l10n_hu_edi_username` |
| `l10n_hu_edi_password` | Localization Hu Electronic data interchange Password | single line text |  | related through path `company_id.l10n_hu_edi_password` |
| `l10n_hu_edi_signature_key` | Localization Hu Electronic data interchange Signature Key | single line text |  | related through path `company_id.l10n_hu_edi_signature_key` |
| `l10n_hu_edi_replacement_key` | Localization Hu Electronic data interchange Replacement Key | single line text |  | related through path `company_id.l10n_hu_edi_replacement_key` |
| `l10n_hu_edi_is_active` | Localization Hu Electronic data interchange Is Active | boolean |  | computed by rule `_compute_l10n_hu_edi_is_active` (not stored) |
| `group_l10n_in_reseller` | Manage Reseller(E-Commerce) | boolean |  |  |
| `l10n_in_edi_production_env` | Indian Production Environment | boolean |  | related through path `company_id.l10n_in_edi_production_env` |
| `l10n_in_gsp` | GSP | selection |  | writable through an inverse rule; Help: Select the GST Suvidha Provider (GSP) you want to use for GST services. |
| `l10n_in_hsn_code_digit` | Localization In Harmonized system nomenclature Code Digit | selection |  | related through path `company_id.l10n_in_hsn_code_digit` |
| `l10n_in_tds_feature` | Localization In Tax deducted at source Feature | boolean |  | related through path `company_id.l10n_in_tds_feature` |
| `l10n_in_tcs_feature` | Localization In Tax collected at source Feature | boolean |  | related through path `company_id.l10n_in_tcs_feature` |
| `l10n_in_withholding_account_id` | Localization In Withholding Account | many to one |  | related through path `company_id.l10n_in_withholding_account_id` |
| `l10n_in_withholding_journal_id` | Localization In Withholding Journal | many to one |  | related through path `company_id.l10n_in_withholding_journal_id` |
| `l10n_in_tan` | Localization In Tan | single line text |  | related through path `company_id.l10n_in_tan` |
| `l10n_in_is_gst_registered` | Localization In Is Goods and services tax Registered | boolean |  | related through path `company_id.l10n_in_is_gst_registered` |
| `l10n_in_gstin` | goods and services tax Number | single line text |  | related through path `company_id.vat` |
| `l10n_in_gstin_status_feature` | Localization In Gstin Status Feature | boolean |  | related through path `company_id.l10n_in_gstin_status_feature` |
| `l10n_in_gst_efiling_feature` | goods and services tax E-Filing & Matching Feature | boolean |  |  |
| `l10n_in_fetch_vendor_edi_feature` | Fetch Vendor E-Invoiced Document | boolean |  |  |
| `l10n_in_enet_vendor_batch_payment_feature` | ENet Vendor Batch Payment | boolean |  |  |
| `module_l10n_in_reports` | goods and services tax E-Filing & Matching | boolean |  |  |
| `module_l10n_in_edi` | Indian Electronic Invoicing | boolean |  |  |
| `module_l10n_in_ewaybill` | Indian Electronic Waybill | boolean |  |  |
| `l10n_in_edi_feature` | Localization In Electronic data interchange Feature | boolean |  | related through path `company_id.l10n_in_edi_feature` |
| `l10n_in_edi_username` | Indian electronic data interchange username | single line text |  | related through path `company_id.l10n_in_edi_username` |
| `l10n_in_edi_password` | Indian electronic data interchange password | single line text |  | related through path `company_id.l10n_in_edi_password` |
| `l10n_in_ewaybill_username` | Indian Ewaybill username | single line text |  | related through path `company_id.l10n_in_ewaybill_username` |
| `l10n_in_ewaybill_password` | Indian Ewaybill password | single line text |  | related through path `company_id.l10n_in_ewaybill_password` |
| `l10n_in_ewaybill_feature` | Localization In Electronic waybill Feature | boolean |  | related through path `company_id.l10n_in_ewaybill_feature` |
| `days_to_purchase` | Days To Purchase | float |  | related through path `company_id.days_to_purchase` |
| `is_installed_sale` | Is the Sale Module Installed | boolean |  |  |
| `l10n_it_edi_register` | Localization It Electronic data interchange Register | boolean |  | computed by rule `_compute_l10n_it_edi_register` (not stored); writable through an inverse rule |
| `l10n_it_edi_purchase_journal_id` | Localization It Electronic data interchange Purchase Journal | many to one |  | related through path `company_id.l10n_it_edi_purchase_journal_id` |
| `l10n_it_edi_show_purchase_journal_id` | Localization It Electronic data interchange Show Purchase Journal | boolean |  | computed by rule `_compute_l10n_it_edi_show_purchase_journal_id` (not stored) |
| `use_root_proxy_user` | Use Root Proxy User | boolean |  | computed by rule `_compute_use_root_proxy_user` (not stored) |
| `l10n_jo_edi_sequence_income_source` | JoFotara Sequence of Income Source | single line text |  | related through path `company_id.l10n_jo_edi_sequence_income_source` |
| `l10n_jo_edi_secret_key` | JoFotara Secret Key | single line text |  | related through path `company_id.l10n_jo_edi_secret_key` |
| `l10n_jo_edi_client_identifier` | JoFotara Client identifier | single line text |  | related through path `company_id.l10n_jo_edi_client_identifier` |
| `l10n_jo_edi_taxpayer_type` | JoFotara Taxpayer Type | selection |  | related through path `company_id.l10n_jo_edi_taxpayer_type` |
| `l10n_jo_edi_demo_mode` | JoFotara Demo Mode | boolean |  | related through path `company_id.l10n_jo_edi_demo_mode` |
| `l10n_jo_edi_pos_enabled` | Localization Jo Electronic data interchange Point of sale Enabled | boolean |  | related through path `company_id.l10n_jo_edi_pos_enabled` |
| `l10n_jo_edi_pos_testing_mode` | Localization Jo Electronic data interchange Point of sale Testing Mode | boolean |  | related through path `company_id.l10n_jo_edi_pos_testing_mode` |
| `l10n_ke_cu_proxy_address` | Localization Ke Cu Proxy Address | single line text |  | related through path `company_id.l10n_ke_cu_proxy_address` |
| `l10n_mx_account_income_return_discount_id` | Income Returns and Discounts Account | many to one | `account.account` | related through path `company_id.l10n_mx_income_return_discount_account_id`; restricted by domain `[('account_type', '=', 'income')]` |
| `l10n_my_edi_mode` | Localization My Electronic data interchange Mode | selection |  | related through path `company_id.l10n_my_edi_mode` |
| `l10n_my_edi_default_import_journal_id` | Localization My Electronic data interchange Default Import Journal | many to one |  | related through path `company_id.l10n_my_edi_default_import_journal_id` |
| `l10n_my_edi_proxy_user_id` | Localization My Electronic data interchange Proxy User | many to one |  | related through path `company_id.l10n_my_edi_proxy_user_id` |
| `l10n_my_edi_company_vat` | Localization My Electronic data interchange Company Value-added tax | single line text |  | related through path `company_id.vat` |
| `l10n_my_accept_processing` | Localization My Accept Processing | boolean |  |  |
| `l10n_pl_reports_tax_office_id` | Localization Pl Reports Tax Office | many to one |  | related through path `company_id.l10n_pl_reports_tax_office_id` |
| `l10n_pl_edi_certificate` | KSeF Certificate | many to one | `certificate.certificate` | computed by rule `_compute_l10n_pl_edi_certificate` (not stored); writable through an inverse rule |
| `l10n_pl_edi_access_token` | KSeF Access Token | single line text |  | read only; related through path `company_id.l10n_pl_edi_access_token` |
| `l10n_pl_edi_register` | Allow KSeF integration | boolean |  | related through path `company_id.l10n_pl_edi_register` |
| `l10n_ro_edi_client_id` | Localization Ro Electronic data interchange Client | single line text |  | related through path `company_id.l10n_ro_edi_client_id` |
| `l10n_ro_edi_client_secret` | Localization Ro Electronic data interchange Client Secret | single line text |  | related through path `company_id.l10n_ro_edi_client_secret` |
| `l10n_ro_edi_access_token` | Localization Ro Electronic data interchange Access Token | single line text |  | related through path `company_id.l10n_ro_edi_access_token` |
| `l10n_ro_edi_refresh_token` | Localization Ro Electronic data interchange Refresh Token | single line text |  | related through path `company_id.l10n_ro_edi_refresh_token` |
| `l10n_ro_edi_access_expiry_date` | Localization Ro Electronic data interchange Access Expiry Date | date |  | related through path `company_id.l10n_ro_edi_access_expiry_date` |
| `l10n_ro_edi_refresh_expiry_date` | Localization Ro Electronic data interchange Refresh Expiry Date | date |  | related through path `company_id.l10n_ro_edi_refresh_expiry_date` |
| `l10n_ro_edi_callback_url` | Localization Ro Electronic data interchange Callback Uniform resource locator | single line text |  | related through path `company_id.l10n_ro_edi_callback_url` |
| `l10n_ro_edi_test_env` | Localization Ro Electronic data interchange Test Env | boolean |  | related through path `company_id.l10n_ro_edi_test_env` |
| `l10n_ro_edi_anaf_imported_inv_journal_id` | Localization Ro Electronic data interchange Anaf Imported Inv Journal | many to one |  | related through path `company_id.l10n_ro_edi_anaf_imported_inv_journal_id` |
| `l10n_rs_edi_api_key` | Localization Rs Electronic data interchange Application programming interface Key | single line text |  | related through path `company_id.l10n_rs_edi_api_key` |
| `l10n_rs_edi_demo_env` | Localization Rs Electronic data interchange Demo Env | boolean |  | related through path `company_id.l10n_rs_edi_demo_env` |
| `l10n_sa_api_mode` | Localization Sa Application programming interface Mode | selection |  | related through path `company_id.l10n_sa_api_mode` |
| `l10n_tr_nilvera_api_key` | Nilvera application programming interface key | single line text |  | related through path `company_id.l10n_tr_nilvera_api_key` |
| `l10n_tr_nilvera_use_test_env` | Use testing environment | boolean |  | required; related through path `company_id.l10n_tr_nilvera_use_test_env` |
| `l10n_tr_nilvera_purchase_journal_id` | Localization Tr Nilvera Purchase Journal | many to one |  | related through path `company_id.l10n_tr_nilvera_purchase_journal_id` |
| `l10n_tr_nilvera_vat` | Nilvera value-added tax | single line text |  | related through path `company_id.vat` |
| `l10n_tr_nilvera_export_alias` | Nilvera Export Alias | single line text |  | related through path `company_id.l10n_tr_nilvera_export_alias` |
| `l10n_tw_edi_ecpay_staging_mode` | Localization Tw Electronic data interchange Ecpay Staging Mode | boolean |  | related through path `company_id.l10n_tw_edi_ecpay_staging_mode` |
| `l10n_tw_edi_ecpay_merchant_id` | Localization Tw Electronic data interchange Ecpay Merchant | single line text |  | related through path `company_id.l10n_tw_edi_ecpay_merchant_id` |
| `l10n_tw_edi_ecpay_hashkey` | Localization Tw Electronic data interchange Ecpay Hashkey | single line text |  | related through path `company_id.l10n_tw_edi_ecpay_hashkey` |
| `l10n_tw_edi_ecpay_hashIV` | Localization Tw Electronic data interchange Ecpay hashIV | single line text |  | related through path `company_id.l10n_tw_edi_ecpay_hashIV` |
| `l10n_vn_edi_username` | Localization Vn Electronic data interchange Username | single line text |  | related through path `company_id.l10n_vn_edi_username` |
| `l10n_vn_edi_password` | Localization Vn Electronic data interchange Password | single line text |  | related through path `company_id.l10n_vn_edi_password` |
| `l10n_vn_edi_default_symbol` | Default Symbol | many to one | `l10n_vn_edi_viettel.sinvoice.symbol` | computed by rule `_compute_l10n_vn_edi_default_symbol` (not stored); writable through an inverse rule; visible only to groups `base.group_system`; Help: This is the symbol that will be used on partners that do not have a specific symbol on them. |
| `l10n_vn_edi_pos_default_symbol` | Default PoS Symbol | many to one |  | related through path `company_id.l10n_vn_pos_default_symbol`; Help: Default Sinvoice Symbol for PoS. |
| `pos_l10n_vn_pos_symbol` | Point of sale Localization Vn Point of sale Symbol | many to one |  | related through path `pos_config_id.l10n_vn_pos_symbol` |
| `pos_l10n_vn_auto_send_to_sinvoice` | Point of sale Localization Vn Auto Send To Sinvoice | boolean |  | related through path `pos_config_id.l10n_vn_auto_send_to_sinvoice` |
| `company_lunch_minimum_threshold` | Maximum Allowed Overdraft | float |  | related through path `company_id.lunch_minimum_threshold` |
| `company_lunch_notify_message` | Lunch notification message | rich text |  | related through path `company_id.lunch_notify_message` |
| `group_mass_mailing_campaign` | Mailing Campaigns | boolean |  | Help: This is useful if your marketing campaigns are composed of several emails |
| `mass_mailing_outgoing_mail_server` | Dedicated Server | boolean |  | Help: Use a specific mail server in priority. Otherwise the system relies on the first outgoing mail server available (based on their sequencing) as it does for normal mails. |
| `mass_mailing_mail_server_id` | Mail Server | many to one | `ir.mail_server` |  |
| `show_blacklist_buttons` | Blacklist Option when Unsubscribing | boolean |  | Help: Allow the recipient to manage themselves their state in the blacklist via the unsubscription page. |
| `mass_mailing_reports` | 24H Stat Mailing Reports | boolean |  | Help: Check how well your mailing is doing a day after it has been sent. |
| `mass_mailing_split_contact_name` | Split First and Last Name | boolean |  | Help: Separate Mailing Contact Names into two fields |
| `events_app_name` | Events App Name | single line text |  | related through path `website_id.events_app_name` |
| `cal_microsoft_client_id` | Microsoft Client_id | single line text |  | default  |
| `cal_microsoft_client_secret` | Microsoft Client_key | single line text |  | default  |
| `cal_microsoft_sync_paused` | Microsoft Synchronization Paused | boolean |  | Help: Indicates if synchronization with Outlook Calendar is paused or not. |
| `microsoft_outlook_client_identifier` | Outlook Client Id | single line text |  |  |
| `microsoft_outlook_client_secret` | Outlook Client Secret | single line text |  |  |
| `group_mrp_byproducts` | By-Products | boolean |  |  |
| `module_mrp_mps` | Master Production Schedule | boolean |  |  |
| `module_mrp_plm` | Product Lifecycle Management (PLM) | boolean |  |  |
| `module_mrp_subcontracting` | Subcontracting | boolean |  |  |
| `group_mrp_routings` | manufacturing Work Orders | boolean |  |  |
| `group_unlocked_by_default` | Unlock Manufacturing Orders | boolean |  |  |
| `group_mrp_reception_report` | Allocation Report for Manufacturing Orders | boolean |  |  |
| `group_mrp_workorder_dependencies` | Work Order Dependencies | boolean |  |  |
| `lc_journal_id` | Default Journal | many to one | `account.journal` | related through path `company_id.lc_journal_id` |
| `group_expiry_date_on_delivery_slip` | Display Expiration Dates on Delivery Slips | boolean |  |  |
| `partnership_label` | Partnership Label | single line text |  | required; related through path `company_id.partnership_label` |
| `pos_adyen_ask_customer_for_tip` | Point of sale Adyen Ask Customer For Tip | boolean |  | computed by rule `_compute_pos_adyen_ask_customer_for_tip` and stored |
| `pos_discount_pc` | Point of sale Discount Pc | float |  | related through path `pos_config_id.discount_pc` |
| `pos_discount_product_id` | Point of sale Discount Product | many to one | `product.product` | computed by rule `_compute_pos_discount_product_id` and stored |
| `pos_basic_employee_ids` | Point of sale Basic Employee | many to many |  | related through path `pos_config_id.basic_employee_ids`; Help: If left empty, all employees can log in to PoS |
| `pos_advanced_employee_ids` | Point of sale Advanced Employee | many to many |  | related through path `pos_config_id.advanced_employee_ids`; Help: Employees linked to users with the PoS Manager role are automatically added to this list |
| `pos_minimal_employee_ids` | Point of sale Minimal Employee | many to many |  | related through path `pos_config_id.minimal_employee_ids`; Help: If left empty, all employees can log in to PoS |
| `pos_self_ordering_service_mode` | Point of sale Self Ordering Service Mode | selection |  | required; related through path `pos_config_id.self_ordering_service_mode` |
| `pos_self_ordering_mode` | Point of sale Self Ordering Mode | selection |  | required; related through path `pos_config_id.self_ordering_mode` |
| `pos_self_ordering_default_language_id` | Point of sale Self Ordering Default Language | many to one |  | related through path `pos_config_id.self_ordering_default_language_id` |
| `pos_self_ordering_available_language_ids` | Point of sale Self Ordering Available Language | many to many |  | related through path `pos_config_id.self_ordering_available_language_ids` |
| `pos_self_ordering_image_home_ids` | Point of sale Self Ordering Image Home | many to many |  | related through path `pos_config_id.self_ordering_image_home_ids` |
| `pos_self_ordering_image_background_ids` | Point of sale Self Ordering Image Background | many to many |  | related through path `pos_config_id.self_ordering_image_background_ids` |
| `pos_self_ordering_image_brand` | Point of sale Self Ordering Image Brand | image |  | related through path `pos_config_id.self_ordering_image_brand` |
| `pos_self_ordering_image_brand_name` | Point of sale Self Ordering Image Brand Name | single line text |  | related through path `pos_config_id.self_ordering_image_brand_name` |
| `pos_self_ordering_pay_after` | Point of sale Self Ordering Pay After | selection |  | required; related through path `pos_config_id.self_ordering_pay_after` |
| `pos_self_ordering_default_user_id` | Point of sale Self Ordering Default User | many to one |  | related through path `pos_config_id.self_ordering_default_user_id` |
| `pos_self_order_online_payment_method_id` | Point of sale Self Order Online Payment Method | many to one |  | related through path `pos_config_id.self_order_online_payment_method_id` |
| `pos_sms_receipt_template_id` | Point of sale Text message Receipt Template | many to one | `sms.template` | related through path `pos_config_id.sms_receipt_template_id` |
| `internal_project_id` | Internal Project | many to one |  | required; related through path `company_id.internal_project_id`; restricted by domain `[('company_id', '=', company_id), ('is_template', '=', False)]`; Help: The default project used when automatically generating timesheets via time off requests. You can specify another project on each time off type individually. |
| `leave_timesheet_task_id` | Time Off Task | many to one |  | related through path `company_id.leave_timesheet_task_id`; restricted by domain `[('company_id', '=', company_id), ('project_id', '=?', internal_project_id), ('has_template_ancestor', '=', False)]`; Help: The default task used when automatically generating timesheets via time off requests. You can specify another task on each time off type individually. |
| `group_purchase_alternatives` | Purchase Alternatives | boolean |  |  |
| `gelato_api_key` | Gelato Application programming interface Key | single line text |  | related through path `company_id.gelato_api_key` |
| `gelato_webhook_secret` | Gelato Webhook Secret | single line text |  | related through path `company_id.gelato_webhook_secret` |
| `invoice_policy` | Invoice Policy | boolean |  | Help: Timesheets taken when invoicing time spent |
| `sms_provider` | Text message Provider | selection |  | required; related through path `company_id.sms_provider` |
| `snailmail_color` | Print In Color | boolean |  | related through path `company_id.snailmail_color` |
| `snailmail_cover` | Add a Cover Page | boolean |  | related through path `company_id.snailmail_cover` |
| `snailmail_duplex` | Print Both sides | boolean |  | related through path `company_id.snailmail_duplex` |
| `snailmail_cover_readonly` | Snailmail Cover Readonly | boolean |  | computed by rule `_compute_cover_readonly` (not stored) |
| `stock_sms_confirmation_template_id` | Stock Text message Confirmation Template | many to one |  | related through path `company_id.stock_sms_confirmation_template_id` |
| `unsplash_access_key` | Access Key | single line text |  |  |
| `unsplash_app_id` | Application identifier | single line text |  |  |
| `turnstile_site_key` | CF Site Key | single line text |  | visible only to groups `base.group_system` |
| `turnstile_secret_key` | CF Secret Key | single line text |  | visible only to groups `base.group_system` |
| `channel_id` | Website Live Channel | many to one | `im_livechat.channel` | related through path `website_id.channel_id` |
| `website_google_places_api_key` | Website's Google Places application programming interface Key | single line text |  | related through path `website_id.google_places_api_key` |
| `default_allow_out_of_stock_order` | Continue selling when out-of-stock | boolean |  | default `True` |
| `default_available_threshold` | Show Threshold | float |  | default `5.0` |
| `default_show_availability` | Show availability Qty | boolean |  | default  |
| `website_warehouse_id` | Website Warehouse | many to one | `stock.warehouse` | related through path `website_id.warehouse_id`; restricted by domain `[('company_id', '=', website_company_id)]` |
| `is_newsletter_enabled` | Is Newsletter Enabled | boolean |  | computed by rule `_compute_is_newsletter_enabled` and stored |
| `newsletter_id` | Newsletter | many to one |  | related through path `website_id.newsletter_id` |

## Selection values

### `product_weight_in_lbs` (Weight unit of measure)

| Value | Label |
|---|---|
| `0` | Kilograms (kg) |
| `1` | Pounds (lb) |

### `product_volume_volume_in_cubic_feet` (Volume unit of measure)

| Value | Label |
|---|---|
| `0` | Cubic Meters (m³) |
| `1` | Cubic Feet (ft³) |

### `auth_signup_uninvited` (Customer Account)

| Value | Label |
|---|---|
| `b2b` | On invitation |
| `b2c` | Free sign up |

### `onboarding_payment_module` (Onboarding Payment Module)

| Value | Label |
|---|---|
| `mercado_pago` | Mercado Pago |
| `razorpay` | Razorpay |
| `stripe` | Stripe |

### `peppol_participation_role` (the pan-European public procurement online network Participation Role)

| Value | Label |
|---|---|
| `sending_and_receiving` | Sending & Receiving |
| `sending_only` | Sending Only |

### `auth_totp_policy` (Two-factor authentication enforcing policy)

| Value | Label |
|---|---|
| `employee_required` | Employees only |
| `all_required` | All users |

### `cloud_storage_provider` (Cloud Storage Provider for new attachments)

| Value | Label |
|---|---|
| `azure` | Azure Cloud Storage |
| `google` | Google Cloud Storage |

### `crm_auto_assignment_action` (Auto Assignment Action)

| Value | Label |
|---|---|
| `manual` | Manually |
| `auto` | Repeatedly |

### `crm_auto_assignment_interval_type` (Auto Assignment Interval Unit)

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hours` | Hours |
| `days` | Days |
| `weeks` | Weeks |

### `lead_enrich_auto` (Enrich lead automatically)

| Value | Label |
|---|---|
| `manual` | Enrich leads on demand only |
| `auto` | Enrich all leads automatically |

### `default_invoice_policy` (Invoicing Policy)

| Value | Label |
|---|---|
| `order` | Invoice what is ordered |
| `delivery` | Invoice what is delivered |

### `default_picking_policy` (Picking Policy)

| Value | Label |
|---|---|
| `direct` | Ship products as soon as available, with back orders |
| `one` | Ship all products at once |

### `timesheet_encode_method` (Encoding Method)

| Value | Label |
|---|---|
| `hours` | Hours / Minutes |
| `days` | Days / Half-Days |

### `account_on_checkout` (Customer Accounts)

| Value | Label |
|---|---|
| `optional` | Optional |
| `disabled` | Disabled |
| `mandatory` | Mandatory |

### `l10n_in_gsp` (GSP)

| Value | Label |
|---|---|
| `bvm` | BVM IT Consulting |
| `tera` | Tera Software (Deprecated) |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`, `l10n_dk_nemhandel_proxy_state`, `l10n_hr_mer_connection_state`. Transitions are specified in the domain documents.

## Operations (235)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_valid_field_parameter` | lifecycle override | self, field, name | `base` |  |  |
| `copy` | lifecycle override | self, default | `base` |  |  |
| `_install_modules` | internal rule | self, modules | `base` | model | Install the requested modules.  :param modules: a recordset of ir.module.module records :return: the next action to execute |
| `_get_classified_fields` | preparation rule | self, fnames | `base` | model | return a dictionary with the fields classified by category:  .. code-block:: python      {   'default': [('default_foo', 'model', 'foo'), ...],         'group':   [('group_bar', [browse_group], browse_implied_group), ...],         'module':  [('module_baz', browse_module), ...],         'config':  [('config_qux', 'my.parameter'), ...],         'other':   ['other_field', ...],     } |
| `get_values` | operation | self | `auth_oauth`, `auth_totp_mail`, `base`, `cloud_storage_google`, `cloud_storage_migration`, `cloud_storage`, `crm_iap_enrich`, `google_recaptcha`, `hr_attendance`, `hr_expense`, `l10n_in`, `mass_mailing`, `portal`, `purchase_stock` | model | Return values for the fields other that `default`, `group` and `module` |
| `default_get` | lifecycle override | self, fields | `base` | model |  |
| `set_values` | operation | self | `account`, `auth_oauth`, `base`, `cloud_storage_azure`, `cloud_storage`, `crm_iap_enrich`, `crm`, `google_recaptcha`, `hr_attendance`, `hr_expense`, `l10n_in`, `mass_mailing`, `mrp`, `point_of_sale`, `product`, `project`, `purchase`, `sale_management`, `sale`, `stock`, `website_sale_mass_mailing`, `website_sale` |  | Set values for the fields other that `default`, `group` and `module` |
| `execute` | operation | self | `base` |  | Called when settings are saved.  This method will call `set_values` and will install/uninstall any modules defined by `module_` Boolean fields and then trigger a web client reload.  .. warning::      This method **SHOULD NOT** be overridden, in most cases what you want to override is     `~set_values()` since `~execute()` does little more than simply call `~set_values()`.      The part that installs/uninstalls modules **MUST ALWAYS** be at the end of the     transaction, otherwise there's a big risk of registry <-> database desynchronisation. |
| `cancel` | operation | self | `base` |  |  |
| `_compute_display_name` | computation | self | `base` |  | Override display_name method to return an appropriate configuration wizard name, and not the generated name. |
| `get_option_path` | operation | self, menu_xml_id | `base` | model | Fetch the path to a specified configuration view and the action id to access it.  :param string menu_xml_id: the xml id of the menuitem where the view is located,     structured as follows: module_name.menuitem_xml_id (e.g.: "sales_team.menu_sale_config") :return: a 2-value tuple where    - t[0]: string: full path to the menuitem (e.g.: "Settings/Configuration/Sales")   - t[1]: int or long: id of the menuitem's action |
| `get_option_name` | operation | self, full_field_name | `base` | model | Fetch the human readable name of a specified configuration option.  :param string full_field_name: the full name of the field, structured as follows:     model_name.field_name (e.g.: "sale.config.settings.fetchmail_lead") :return: human readable name of the field (e.g.: "Create leads from incoming mails") :rtype: str |
| `get_config_warning` | operation | self, msg | `base` | model | Helper: return a Warning exception with the given message where the `%(field:xxx)s` and/or `%(menu:yyy)s` are replaced by the human readable field's name and/or menuitem's full path.  Usage: ------ Just include in your error message `%(field:model_name.field_name)s` to obtain the human readable field's name, and/or %(menu:module_name.menuitem_xml_id)s to obtain the menuitem's full path.  Example of use: ---------------  .. code-block:: python      raise env['ir..config.settings'](_(         "Error: this action is prohibited. You should check the "         "field %(field:sale.config.setti |
| `create` | lifecycle override | self, vals_list | `base`, `event`, `hr_presence`, `l10n_hu_edi`, `point_of_sale`, `pos_hr` | model_create_multi |  |
| `action_open_template_user` | user action | self | `base` |  |  |
| `open_company` | operation | self | `base_setup` |  |  |
| `open_new_user_default_groups` | operation | self | `base_setup` |  |  |
| `_prepare_report_view_action` | preparation rule | self, template | `base_setup` | model |  |
| `edit_external_header` | operation | self | `base_setup` |  |  |
| `_compute_company_count` | computation | self | `base_setup` | depends: `company_id` |  |
| `_compute_active_user_count` | computation | self | `base_setup` | depends: `company_id` |  |
| `_compute_language_count` | computation | self | `base_setup` | depends: `company_id` |  |
| `_compute_company_informations` | computation | self | `base_setup`, `l10n_sa_edi` | depends: `company_id` |  |
| `_compute_is_root_company` | computation | self | `base_setup` | depends: `company_id` |  |
| `_compute_fail_counter` | computation | self | `mail` |  |  |
| `open_email_layout` | operation | self | `mail` |  |  |
| `open_mail_templates` | operation | self | `mail` |  |  |
| `_onchange_group_sale_pricelist` | on change | self | `product` | onchange: `group_product_pricelist` |  |
| `_compute_portal_allow_api_keys` | computation | self | `portal` |  |  |
| `_inverse_portal_allow_api_keys` | inverse computation | self | `portal` |  |  |
| `_compute_is_account_peppol_eligible` | computation | self | `account` | depends: `country_code` |  |
| `reload_template` | operation | self | `account`, `l10n_in` |  |  |
| `_compute_account_default_credit_limit` | computation | self | `account` | depends: `company_id` |  |
| `_inverse_account_default_credit_limit` | inverse computation | self | `account` |  |  |
| `_compute_has_chart_of_accounts` | computation | self | `account` | depends: `company_id` |  |
| `_compute_module_account_invoice_extract` | computation | self | `account` | depends: `module_account_extract` |  |
| `_compute_module_account_bank_statement_extract` | computation | self | `account` | depends: `module_account_extract` |  |
| `onchange_analytic_accounting` | on change | self | `account` | onchange: `group_analytic_accounting` |  |
| `onchange_module_account_budget` | on change | self | `account` | onchange: `module_account_budget` |  |
| `_onchange_tax_exigibility` | on change | self | `account` | onchange: `tax_exigibility` |  |
| `_compute_terms_preview` | computation | self | `account` | depends: `terms_type` |  |
| `action_update_terms` | user action | self | `account` |  |  |
| `action_eu_oss_tax_mapping` | user action | self | `account` |  |  |
| `_compute_active_provider_id` | computation | self | `payment`, `website_payment` | depends: `company_id`; depends: `company_id`, `website_id` |  |
| `_compute_has_enabled_provider` | computation | self | `payment`, `website_payment` | depends: `company_id`; depends: `company_id`, `website_id` |  |
| `_get_active_providers_domain` | preparation rule | self, enabled_only | `payment`, `website_payment` |  | Return the domain to search for active providers.  :param bool enabled_only: Whether only enabled providers should be considered active. :return: The active providers domain. :rtype: Domain |
| `_compute_onboarding_payment_module` | computation | self | `payment` | depends: `company_id.currency_id`, `company_id.country_id.is_stripe_supported_country` |  |
| `action_view_active_provider` | user action | self | `payment` |  |  |
| `_start_payment_onboarding` | internal rule | self, menu_id | `payment` |  | Install the onboarding module, configure the provider and run the onboarding action.  :param int menu_id: The menu from which the onboarding is started, as an `ir.ui.menu` id. :return: The action returned by `action_start_onboarding`. :rtype: dict |
| `_get_peppol_proxy_type` | preparation rule | self | `account_peppol` |  |  |
| `_compute_peppol_use_parent_company` | computation | self | `account_peppol` | depends: `company_id.peppol_parent_company_id` |  |
| `_compute_peppol_participation_role` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state` |  |
| `_inverse_peppol_participation_role` | inverse computation | self | `account_peppol` |  |  |
| `_compute_account_peppol_contact_email` | computation | self | `account_peppol` | depends: `company_id.account_peppol_contact_email` |  |
| `_inverse_account_peppol_contact_email` | inverse computation | self | `account_peppol` |  |  |
| `_compute_peppol_purchase_journal_required` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state`, `peppol_participation_role` |  |
| `action_open_peppol_form` | user action | self | `account_peppol`, `l10n_fr_pdp` |  |  |
| `button_open_peppol_config_wizard` | user action | self | `account_peppol`, `l10n_fr_pdp` |  |  |
| `button_peppol_disconnect_branch_from_parent` | user action | self | `account_peppol` |  |  |
| `button_peppol_register_sender_as_receiver` | user action | self | `account_peppol` |  | Register the existing user as a receiver. |
| `button_reconnect_this_database` | user action | self | `account_peppol` |  | Re-establish an out-of-sync connection |
| `button_disconnect_this_database` | user action | self | `account_peppol` |  | Disconnect the current database from the Peppol network. This does not delete or affect the IAP connection, which will remain intact. So don't use this to deregister the participant/connection. |
| `button_peppol_deregister` | user action | self | `account_peppol` |  | Unregister the user from Peppol network. |
| `button_peppol_reregister` | user action | self | `account_peppol` |  |  |
| `_get_pdp_module_info` | preparation rule | self | `account_peppol` | model |  |
| `get_uri` | operation | self | `auth_oauth` | model |  |
| `_on_change_mins` | on change | self | `auth_password_policy` | onchange: `minlength` | Password lower bounds must be naturals |
| `_onchange_auth_totp_enforce` | on change | self | `auth_totp_mail` | onchange: `auth_totp_enforce` |  |
| `_setup_cloud_storage_provider` | internal rule | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Setup the cloud storage provider and check the validity of the account info after saving the config in settings. return: None |
| `_get_cloud_storage_configuration` | preparation rule | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Return the configuration for the cloud storage provider. If the cloud storage provider is not fully configured, return an empty dict. :return: A configuration dict |
| `_check_cloud_storage_uninstallable` | validation | self | `cloud_storage_azure`, `cloud_storage_google`, `cloud_storage` |  | Check if the cloud storages provider is used by any attachments :raise UserError: when the cloud storage provider cannot be uninstalled |
| `_compute_cloud_storage_google_account_info` | on change | self | `cloud_storage_google` | onchange: `cloud_storage_google_service_account_key` |  |
| `_compute_cloud_storage_migration_message_model_ids` | computation | self | `cloud_storage_migration` |  |  |
| `_inverse_cloud_storage_migration_message_model_ids` | inverse computation | self | `cloud_storage_migration` |  |  |
| `_compute_cloud_storage_migration_all_model_ids` | computation | self | `cloud_storage_migration` |  |  |
| `_inverse_cloud_storage_migration_all_model_ids` | inverse computation | self | `cloud_storage_migration` |  |  |
| `action_open_cloud_storage_migration_configurations` | user action | self | `cloud_storage_migration` |  |  |
| `_compute_crm_auto_assignment_data` | computation | self | `crm` | depends: `crm_use_auto_assignment` |  |
| `_onchange_crm_auto_assignment_run_datetime` | on change | self | `crm` | onchange: `crm_auto_assignment_interval_type`, `crm_auto_assignment_interval_number` |  |
| `_compute_pls_fields` | computation | self | `crm` | depends: `predictive_lead_scoring_fields_str` | As config_parameters does not accept m2m field, we get the fields back from the Char config field, to ease the configuration in config panel |
| `_inverse_pls_fields_str` | inverse computation | self | `crm` |  | As config_parameters does not accept m2m field, we store the fields with a comma separated string into a Char config field |
| `_compute_pls_start_date` | computation | self | `crm` | depends: `predictive_lead_scoring_start_date_str` | As config_parameters does not accept Date field, we get the date back from the Char config field, to ease the configuration in config panel |
| `_inverse_pls_start_date_str` | inverse computation | self | `crm` |  | As config_parameters does not accept Date field, we store the date formated string into a Char config field |
| `_compute_predictive_lead_scoring_field_labels` | computation | self | `crm` | depends: `predictive_lead_scoring_fields` |  |
| `_get_crm_auto_assignmment_run_datetime` | preparation rule | self, run_datetime, run_interval, run_interval_number | `crm` |  |  |
| `action_crm_assign_leads` | user action | self | `crm` |  |  |
| `_onchange_group_discount_per_so_line` | computation | self | `sale` | depends: `group_discount_per_so_line` |  |
| `_onchange_group_product_variant` | on change | self | `sale` | onchange: `group_product_variant` | The product Configurator requires the product variants activated. If the user disables the product variants -> disable the product configurator as well |
| `_onchange_portal_confirmation_pay` | on change | self | `sale` | onchange: `portal_confirmation_pay` |  |
| `_onchange_prepayment_percent` | on change | self | `sale` | onchange: `prepayment_percent` |  |
| `_onchange_quotation_validity_days` | on change | self | `sale` | onchange: `quotation_validity_days` |  |
| `action_sale_start_payment_onboarding` | user action | self | `sale` |  |  |
| `_compute_replenish_on_order` | computation | self | `stock` |  |  |
| `_inverse_replenish_on_order` | inverse computation | self | `stock` |  |  |
| `_onchange_group_stock_multi_locations` | on change | self | `stock` | onchange: `group_stock_multi_locations` |  |
| `_onchange_group_stock_production_lot` | on change | self | `product_expiry`, `stock` | onchange: `group_stock_production_lot` |  |
| `_onchange_stock_confirmation_fields` | on change | self | `stock` | onchange: `stock_confirmation_type`, `stock_text_confirmation` |  |
| `onchange_adv_location` | on change | self | `stock` | onchange: `group_stock_adv_location` |  |
| `_onchange_use_security_lead` | on change | self | `sale_stock` | onchange: `use_security_lead` |  |
| `_default_use_google_maps_static_api` | preparation rule | self | `event` |  |  |
| `_compute_maps_static_api_key` | computation | self | `event` | depends: `use_google_maps_static_api` | Clear API key on disabling google maps. |
| `_compute_maps_static_api_secret` | computation | self | `event` | depends: `use_google_maps_static_api` | Clear API secret on disabling google maps. |
| `_onchange_module_website_event_track` | on change | self | `event` | onchange: `module_website_event_track` | Reset sub-modules, otherwise you may have track to False but still have track_live or track_quiz to True, meaning track will come back due to dependencies of modules. |
| `_check_google_maps_static_api_secret` | validation | self | `event` |  |  |
| `write` | lifecycle override | self, vals | `event` |  |  |
| `regenerate_kiosk_key` | operation | self | `hr_attendance` |  |  |
| `_compute_hr_expense_alias_prefix` | computation | self | `hr_expense` | depends: `hr_expense_use_mailgateway` |  |
| `_compute_hr_expense_alias_domain_id` | computation | self | `hr_expense` | depends: `hr_expense_use_mailgateway` |  |
| `_inverse_hr_expense_alias_domain_id` | inverse computation | self | `hr_expense` |  |  |
| `_default_website` | preparation rule | self | `website` |  |  |
| `_compute_shared_user_account` | computation | self | `website` | depends: `website_id` |  |
| `_onchange_shared_key` | on change | self | `website` | onchange: `plausible_shared_key` |  |
| `_inverse_shared_user_account` | inverse computation | self | `website` |  |  |
| `_compute_auth_signup_uninvited` | computation | self | `website` | depends: `website_id.auth_signup_uninvited` |  |
| `_inverse_auth_signup_uninvited` | inverse computation | self | `website` |  |  |
| `_compute_has_plausible_shared_key` | computation | self | `website` | depends: `website_id` |  |
| `_inverse_has_plausible_shared_key` | inverse computation | self | `website` |  |  |
| `_compute_has_google_analytics` | computation | self | `website` | depends: `website_id` |  |
| `_inverse_has_google_analytics` | inverse computation | self | `website` |  |  |
| `_compute_has_google_search_console` | computation | self | `website` | depends: `website_id` |  |
| `_inverse_has_google_search_console` | inverse computation | self | `website` |  |  |
| `_compute_has_default_share_image` | computation | self | `website` | depends: `website_id` |  |
| `_inverse_has_default_share_image` | inverse computation | self | `website` |  |  |
| `_onchange_language_ids` | on change | self | `website` | onchange: `language_ids` |  |
| `action_website_create_new` | user action | self | `website` |  |  |
| `action_open_robots` | user action | self | `website` |  |  |
| `action_open_blocked_third_party_domains` | user action | self | `website` |  |  |
| `_compute_timesheet_encode_method` | computation | self | `hr_timesheet` | depends: `company_id` |  |
| `_inverse_timesheet_encode_method` | inverse computation | self | `hr_timesheet` |  |  |
| `_compute_is_encode_uom_days` | computation | self | `hr_timesheet` | depends: `timesheet_encode_method` |  |
| `_compute_timesheet_modules` | computation | self | `hr_timesheet` | depends: `module_hr_timesheet` |  |
| `_compute_partner_autocomplete_insufficient_credit` | computation | self | `partner_autocomplete` |  |  |
| `redirect_to_buy_autocomplete_credit` | operation | self | `partner_autocomplete` |  |  |
| `_default_pos_config` | preparation rule | self | `point_of_sale` |  |  |
| `open_payment_method_form` | operation | self | `point_of_sale` |  |  |
| `action_pos_config_create_new` | user action | self | `point_of_sale` |  |  |
| `action_pos_printer_dialog` | user action | self | `point_of_sale` |  |  |
| `pos_close_ui` | operation | self | `point_of_sale`, `pos_self_order` |  |  |
| `pos_open_ui` | operation | self | `point_of_sale` |  |  |
| `_is_cashdrawer_displayed` | internal rule | self, res_config | `point_of_sale`, `pos_imin` | model |  |
| `_compute_pos_printer` | computation | self | `point_of_sale` | depends: `pos_module_pos_restaurant`, `pos_config_id` |  |
| `_compute_pos_iface_available_categ_ids` | computation | self | `point_of_sale` | depends: `pos_limit_categories`, `pos_config_id` |  |
| `_compute_pos_selectable_categ_ids` | computation | self | `point_of_sale` | depends: `pos_iface_available_categ_ids` |  |
| `_compute_pos_iface_cashdrawer` | computation | self | `point_of_sale` | depends: `pos_iface_print_via_proxy`, `pos_config_id`, `pos_epson_printer_ip`, `pos_other_devices` |  |
| `_compute_pos_receipt_header_footer` | computation | self | `point_of_sale` | depends: `pos_is_header_or_footer`, `pos_config_id` |  |
| `_compute_pos_fiscal_positions` | computation | self | `point_of_sale` | depends: `pos_tax_regime_selection`, `pos_config_id` |  |
| `_compute_pos_tip_product_id` | computation | self | `point_of_sale` | depends: `pos_iface_tipproduct`, `pos_config_id` |  |
| `_compute_pos_pricelist_id` | computation | self | `point_of_sale`, `pos_self_order` | depends: `pos_use_pricelist`, `pos_config_id`, `pos_journal_id`; depends: `pos_self_ordering_mode` |  |
| `_compute_pos_allowed_pricelist_ids` | computation | self | `point_of_sale` | depends: `pos_available_pricelist_ids`, `pos_use_pricelist` |  |
| `_compute_pos_iface_print_via_proxy` | computation | self | `point_of_sale` | depends: `pos_is_posbox`, `pos_config_id` |  |
| `_compute_pos_iface_scan_via_proxy` | computation | self | `point_of_sale` | depends: `pos_is_posbox`, `pos_config_id` |  |
| `_compute_pos_iface_electronic_scale` | computation | self | `point_of_sale` | depends: `pos_is_posbox`, `pos_config_id` |  |
| `_onchange_trusted_config_ids` | on change | self | `point_of_sale` | onchange: `pos_trusted_config_ids` |  |
| `_onchange_epson_printer_ip` | on change | self | `point_of_sale` | onchange: `pos_epson_printer_ip` |  |
| `action_w_payment_start_payment_onboarding` | user action | self | `website_payment` |  |  |
| `_compute_account_on_checkout` | computation | self | `website_sale` | depends: `website_id.account_on_checkout` |  |
| `_inverse_account_on_checkout` | inverse computation | self | `website_sale` |  |  |
| `action_view_delivery_provider_modules` | user action | self | `website_sale` |  |  |
| `action_open_abandoned_cart_mail_template` | user action | self | `website_sale` | readonly |  |
| `action_open_extra_info` | user action | self | `website_sale` |  |  |
| `action_open_sale_mail_templates` | user action | self | `website_sale` | readonly |  |
| `action_open_product_feeds` | user action | self | `website_sale` | readonly | Open the list view to manage the feed specific to the current website. |
| `_compute_pos_module_pos_restaurant` | computation | self | `pos_restaurant` | depends: `pos_module_pos_restaurant`, `pos_config_id` |  |
| `_compute_pos_set_tip_after_payment` | computation | self | `pos_restaurant` | depends: `pos_iface_tipproduct`, `pos_config_id` |  |
| `_onchange_group_product_variant_purchase` | on change | self | `purchase` | onchange: `group_product_variant` | If the user disables the product variants -> disable the product configurator as well |
| `_onchange_module_purchase_product_matrix` | on change | self | `purchase` | onchange: `module_purchase_product_matrix` | The product variant grid requires the product variants activated If the user enables the product configurator -> enable the product variants as well |
| `_compute_nemhandel_edi_user` | computation | self | `l10n_dk_nemhandel` | depends: `company_id.account_edi_proxy_client_ids` |  |
| `action_open_nemhandel_form` | user action | self | `l10n_dk_nemhandel` |  |  |
| `button_update_nemhandel_user_data` | user action | self | `l10n_dk_nemhandel` |  | Action for the user to be able to update their contact details any time Calls /update_user on the iap server |
| `button_deregister_nemhandel_participant` | user action | self | `l10n_dk_nemhandel` |  | Deregister the edi user from Nemhandel network |
| `_compute_l10n_eu_oss_european_country` | computation | self | `l10n_eu_oss` | depends: `company_id` |  |
| `action_open_pdp_form` | user action | self | `l10n_fr_pdp` |  |  |
| `button_open_pdp_config_wizard` | user action | self | `l10n_fr_pdp` |  |  |
| `_compute_l10n_fr_pdp_pilot_phase` | computation | self | `l10n_fr_pdp` | depends: `company_id.l10n_fr_pdp_pilot_phase` |  |
| `_inverse_l10n_fr_pdp_pilot_phase` | inverse computation | self | `l10n_fr_pdp` |  |  |
| `_pdp_ensure_selection_value` | internal rule | self, model_name, field_name, new_value | `l10n_fr_pdp` | model |  |
| `button_l10n_hr_activate_mojeracun` | user action | self | `l10n_hr_edi` |  |  |
| `button_l10n_hr_deactivate_mojeracun` | user action | self | `l10n_hr_edi` |  |  |
| `_compute_l10n_hu_edi_is_active` | computation | self | `l10n_hu_edi` | depends: `company_id.l10n_hu_edi_server_mode` |  |
| `_update_l10n_in_feature` | internal rule | self, column | `l10n_in` |  | This way, after installing the module, the field will already be set for the active company. |
| `l10n_in_edi_buy_iap` | operation | self | `l10n_in` |  |  |
| `_l10n_in_check_gst_number` | internal rule | self | `l10n_in` |  |  |
| `_l10n_in_is_first_time_setup` | internal rule | self | `l10n_in` |  | Check if at least one company for India has been configured with the localization settings. If not, it means it's the first time setup. |
| `_l10n_in_gsp_provider_changed` | internal rule | self | `l10n_in_edi`, `l10n_in_ewaybill`, `l10n_in` |  | Hook to be overridden in other modules to handle GSP provider change. |
| `_set_l10n_in_gsp` | internal rule | self | `l10n_in` |  |  |
| `l10n_in_edi_test` | operation | self | `l10n_in_edi` |  |  |
| `l10n_in_ewaybill_test` | operation | self | `l10n_in_ewaybill` |  |  |
| `_create_proxy_user` | internal rule | self, company_id, edi_mode | `l10n_it_edi` |  |  |
| `_compute_l10n_it_edi_show_purchase_journal_id` | computation | self | `l10n_it_edi` | depends: `company_id` |  |
| `_compute_l10n_it_edi_register` | computation | self | `l10n_it_edi` | depends: `company_id` |  |
| `_set_l10n_it_edi_register` | internal rule | self | `l10n_it_edi` |  |  |
| `_compute_use_root_proxy_user` | computation | self | `l10n_it_edi` | depends: `company_id.account_edi_proxy_client_ids`, `company_id.account_edi_proxy_client_ids.active` |  |
| `_onchange_l10n_my_edi_mode` | on change | self | `l10n_my_edi` | onchange: `l10n_my_edi_mode` | This onchange is mostly here to improve usability by avoiding the need to save when changing the mode. |
| `action_l10n_my_edi_allow_processing` | user action | self | `l10n_my_edi` |  | We always expect the user to give his consent by pressing the button, in any mode, to enable the edi. |
| `action_l10n_my_edi_unregister` | user action | self | `l10n_my_edi` |  | Send a notification to the proxy to free the ID (vat) of the user, and archive the local proxy user. Useful if there has been a misconfiguration or the user wishes to use a new database/... |
| `action_open_company_form` | user action | self | `l10n_my_edi` |  | This will be used to ease the configuration by allowing to quickly access the company. |
| `_compute_l10n_pl_edi_certificate` | computation | self | `l10n_pl_edi` | depends: `company_id` |  |
| `_l10n_pl_edi_reset` | on change | self | `l10n_pl_edi` | onchange: `l10n_pl_edi_register` |  |
| `_set_l10n_pl_edi_certificate` | internal rule | self | `l10n_pl_edi` |  |  |
| `_l10n_pl_edi_ksef_authenticate` | internal rule | self | `l10n_pl_edi` |  | Orchestrates the entire authentication flow using the service. |
| `button_l10n_ro_edi_generate_token` | user action | self | `l10n_ro_edi` |  | Redirects to controllers/main.py ~ `authorize` method |
| `nilvera_ping` | operation | self | `l10n_tr_nilvera` |  | Test the connection and the API key. |
| `_compute_l10n_vn_edi_default_symbol` | computation | self | `l10n_vn_edi_viettel` | depends: `company_id` |  |
| `_inverse_l10n_vn_edi_default_symbol` | inverse computation | self | `l10n_vn_edi_viettel` |  |  |
| `_onchange_mass_mailing_outgoing_mail_server` | on change | self | `mass_mailing` | onchange: `mass_mailing_outgoing_mail_server` |  |
| `_onchange_group_unlocked_by_default` | on change | self | `mrp` | onchange: `group_unlocked_by_default` | When changing this setting, we want existing MOs to automatically update to match setting. |
| `_onchange_group_lot_on_delivery_slip` | on change | self | `product_expiry` | onchange: `group_lot_on_delivery_slip` |  |
| `_onchange_module_product_expiry` | on change | self | `product_expiry` | onchange: `module_product_expiry` |  |
| `_onchange_partnership_label` | on change | self | `partnership` | onchange: `partnership_label` |  |
| `_compute_pos_adyen_ask_customer_for_tip` | computation | self | `pos_adyen` | depends: `pos_iface_tipproduct`, `pos_config_id` |  |
| `_compute_pos_discount_product_id` | computation | self | `pos_discount` | depends: `company_id`, `pos_module_pos_discount`, `pos_config_id` |  |
| `_onchange_minimal_employee_ids` | on change | self | `pos_hr` | onchange: `pos_minimal_employee_ids` |  |
| `_onchange_basic_employee_ids` | on change | self | `pos_hr` | onchange: `pos_basic_employee_ids` |  |
| `_onchange_advanced_employee_ids` | on change | self | `pos_hr` | onchange: `pos_advanced_employee_ids` |  |
| `_onchange_default_user` | on change | self | `pos_self_order` | onchange: `pos_self_ordering_default_user_id` |  |
| `_onchange_pos_self_order_service_mode` | on change | self | `pos_self_order` | onchange: `pos_self_ordering_service_mode` |  |
| `_onchange_pos_self_order_kiosk_default_language` | on change | self | `pos_self_order` | onchange: `pos_self_ordering_default_language_id`, `pos_self_ordering_available_language_ids` |  |
| `_onchange_pos_self_order_kiosk` | on change | self | `pos_self_order_sale`, `pos_self_order` | onchange: `pos_self_ordering_mode`, `pos_module_pos_restaurant`; onchange: `pos_self_ordering_mode` |  |
| `_onchange_pos_payment_method_ids` | on change | self | `pos_self_order` | onchange: `pos_payment_method_ids` |  |
| `_onchange_pos_self_order_pay_after` | on change | self | `pos_self_order` | onchange: `pos_self_ordering_pay_after`, `pos_self_ordering_mode` |  |
| `custom_link_action` | operation | self | `pos_self_order` |  |  |
| `_generate_excel` | internal rule | self, rows, headers | `pos_self_order` |  |  |
| `get_pos_qr_stands` | operation | self | `pos_self_order` |  | Redirect to the get the free stands with the data of QR codes for the current POS config |
| `generate_qr_codes_zip` | operation | self | `pos_self_order` |  |  |
| `generate_qr_codes_page` | operation | self | `pos_self_order` |  | Generate the data needed to print the QR codes page |
| `preview_self_order_app` | operation | self | `pos_self_order` |  |  |
| `update_access_tokens` | operation | self | `pos_self_order` |  |  |
| `_onchange_timesheet_project_id` | on change | self | `project_timesheet_holidays` | onchange: `internal_project_id` |  |
| `_onchange_timesheet_task_id` | on change | self | `project_timesheet_holidays` | onchange: `leave_timesheet_task_id` |  |
| `action_open_sms_twilio_account_manage` | user action | self | `sms_twilio` |  |  |
| `_is_layout_cover_required` | internal rule | self | `snailmail` |  |  |
| `_onchange_layout` | on change | self | `snailmail` | onchange: `external_report_layout_id` |  |
| `_compute_cover_readonly` | computation | self | `snailmail` | depends: `external_report_layout_id` |  |
| `action_view_in_store_delivery_methods` | user action | self | `website_sale_collect` |  | Return an action to browse pickup delivery methods in list view, or in form view if there is only one. |
| `_compute_is_newsletter_enabled` | computation | self | `website_sale_mass_mailing` | depends: `website_id` | Computing newsletter setting when changing the website in the res.config.settings page to show the correct value in the checkbox. |

## Validation and error messages (38)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `copy` | UserError | Cannot duplicate configuration! | `base` |
| `execute` | AccessError | Only administrators can change the settings | `base` |
| `action_open_template_user` | UserError | Invalid template user. It seems it has been deleted. | `base` |
| `open_email_layout` | UserError | This layout seems to no longer exist. | `mail` |
| `set_values` | UserError | Please configure the Cloud Storage before enabling it | `cloud_storage` |
| `_setup_cloud_storage_provider` | ValidationError | The connection string is not allowed to upload blobs to the container. %s | `cloud_storage_azure` |
| `_setup_cloud_storage_provider` | ValidationError | The connection string is not allowed to download blobs from the container. %s | `cloud_storage_azure` |
| `_check_cloud_storage_uninstallable` | UserError | Some Azure attachments are in use, please migrate their cloud storages before disable this module | `cloud_storage_azure` |
| `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to upload blobs to the bucket. %s | `cloud_storage_google` |
| `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to download blobs from the bucket. %s | `cloud_storage_google` |
| `_setup_cloud_storage_provider` | ValidationError | The account info is not allowed to set the bucket's CORS. %s | `cloud_storage_google` |
| `_check_cloud_storage_uninstallable` | UserError | Some Google attachments are in use, please migrate cloud storages before disable the provider | `cloud_storage_google` |
| `_onchange_crm_auto_assignment_run_datetime` | UserError | Repeat frequency should be positive. | `crm` |
| `_onchange_crm_auto_assignment_run_datetime` | UserError | Invalid repeat frequency. Consider changing frequency type instead of using large numbers. | `crm` |
| `set_values` | UserError | You can't deactivate the multi-location if you have more than once warehouse by company | `stock` |
| `set_values` | UserError | You have product(s) in stock that have lot/serial number tracking enabled.  Switch off tracking on all the products before switching off this setting. | `stock` |
| `_check_google_maps_static_api_secret` | UserError | Please enter a valid base64 secret | `event` |
| `button_update_nemhandel_user_data` | ValidationError | Contact email is required | `l10n_dk_nemhandel` |
| `l10n_in_edi_buy_iap` | ValidationError | Please ensure that at least one Indian service and production environment is enabled, and save the configuration to proceed with purchasing credits. | `l10n_in` |
| `_l10n_in_check_gst_number` | RedirectWarning | Please set a valid GST number on company. | `l10n_in` |
| `l10n_in_edi_test` | UserError | '\n'.join(['[%s] %s' % (e.get('code'), e.get('message')) for e in response['error']]) | `l10n_in_edi` |
| `l10n_in_edi_test` | UserError | Incorrect username or password, or the GST number on company does not match. | `l10n_in_edi` |
| `l10n_in_ewaybill_test` | UserError | Incorrect username or password, or the GST number on company does not match. | `l10n_in_ewaybill` |
| `l10n_in_ewaybill_test` | UserError | e.get_all_error_message() | `l10n_in_ewaybill` |
| `action_l10n_my_edi_unregister` | UserError | An unexpected error occurred while unregistering. Please try again later. | `l10n_my_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | A polish VAT number must be set on your company. | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | Please set up a valid KSeF Certificate, with its Private Key set | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | The selected certificate record (%(name)s) is missing a private key. | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | UserError | KSeF certificate and private key are not set. | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | Failed to initiate KSeF authentication. | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | Authentication with KSeF failed. | `l10n_pl_edi` |
| `_l10n_pl_edi_ksef_authenticate` | ValidationError | Failed to retrieve access or refresh tokens. | `l10n_pl_edi` |
| `_onchange_default_user` | ValidationError | The user must be a POS user | `pos_self_order` |
| `_onchange_pos_payment_method_ids` | ValidationError | You cannot add cash payment methods in kiosk mode. | `pos_self_order` |
| `_onchange_pos_self_order_pay_after` | ValidationError | Only pay after each is available with kiosk mode. | `pos_self_order` |
| `generate_qr_codes_zip` | ValidationError | QR codes can only be generated in mobile or consultation mode. | `pos_self_order` |
| `generate_qr_codes_zip` | ValidationError | In Self-Order mode, you must have at least one table to generate QR codes | `pos_self_order` |
| `generate_qr_codes_page` | ValidationError | In Self-Order mode, you must have at least one table to generate QR codes | `pos_self_order` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (147)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `country_code`, `has_chart_of_accounts`, `has_accounting_entries`, `chart_template`, `module_account_peppol`, `sale_tax_id`, `purchase_tax_id`, `account_price_include`, `tax_calculation_rounding_method`, `tax_exigibility`, `tax_cash_basis_journal_id`, `account_cash_basis_base_account_id`, `account_fiscal_country_id`, `currency_id`, `group_multi_currency`, `module_currency_rate_live`, `module_snailmail_account`, `group_sale_delivery_address`, `group_cash_rounding`, `module_account_intrastat`, `incoterm_id`, `show_sale_receipts`, `use_invoice_terms`, `terms_type`, `invoice_terms`, `preview_ready`, `account_use_credit_limit`, `account_default_credit_limit`, `display_invoice_amount_total_words`, `display_invoice_tax_company_currency`, `group_uom`, `module_account_payment`, `link_qr_code`, `module_account_batch_payment`, `module_account_sepa_direct_debit`, `qr_code`, `autopost_bills`, `module_account_check_printing`, `module_account_iso20022`, `module_account_extract`, `currency_exchange_journal_id`, `income_currency_exchange_account_id`, `expense_currency_exchange_account_id`, `account_journal_suspense_account_id`, `transfer_account_id`, `account_discount_expense_allocation_id`, `account_discount_income_allocation_id`, `account_journal_early_pay_discount_gain_account_id`, `account_journal_early_pay_discount_loss_account_id`, `income_account_id`, `expense_account_id`, `module_account_bank_statement_import_qif`, `module_account_reports`, `group_analytic_accounting`, `module_account_budget`, `module_product_margin`, `restrictive_audit_trail`, `is_account_peppol_eligible`, `account_storno`, `quick_edit_mode` | `Reload`, `OSS Tax mapping`, `Currencies`, `Cash Roundings`, `Update Terms`, `Units & Packagings` |  | `account` |
| `account.res_config_settings_view_form_base_setup` | xpath | `base_setup.res_config_settings_view_form` |  |  |  | `account` |
| `account_check_printing.res_config_settings_view_form` | setting | `account.res_config_settings_view_form` | `account_check_printing_layout`, `account_check_printing_multi_stub`, `account_check_printing_margin_top`, `account_check_printing_margin_left`, `account_check_printing_margin_right`, `account_check_printing_date_label` |  |  | `account_check_printing` |
| `account_payment.res_config_settings_view_form` | field | `account.res_config_settings_view_form` | `module_account_payment`, `pay_invoices_online` |  |  | `account_payment` |
| `account_payment_interco.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `account_interco_clearing_journal_id`, `account_interco_payable_id`, `account_interco_receivable_id` |  |  | `account_payment_interco` |
| `account_peppol.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `account_peppol_edi_identification`, `account_peppol_edi_mode`, `peppol_external_provider`, `account_peppol_edi_identification`, `account_peppol_edi_mode`, `account_peppol_contact_email`, `peppol_participation_role`, `peppol_parent_company_name`, `account_peppol_edi_identification`, `account_peppol_purchase_journal_id` | `Advanced Configuration`, `Disconnect Peppol`, `Activate Peppol`, `Register with the system`, `Disconnect`, `Reconnect this database`, `Disconnect this database` |  | `account_peppol` |
| `account_update_tax_tags.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` |  | `Update tax tags on existing Journal Entries` |  | `account_update_tax_tags` |
| `auth_ldap.res_config_settings_view_form` | setting | `base_setup.res_config_settings_view_form` |  | `LDAP Server` |  | `auth_ldap` |
| `auth_oauth.res_config_settings_view_form` | div | `base_setup.res_config_settings_view_form` |  | `OAuth Providers` |  | `auth_oauth` |
| `auth_password_policy.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `minlength` |  |  | `auth_password_policy` |
| `auth_signup.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `auth_signup_uninvited`, `auth_signup_reset_password` | `Default Access Rights` |  | `auth_signup` |
| `auth_totp_mail.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `auth_totp_enforce`, `auth_totp_policy` |  |  | `auth_totp_mail` |
| `base.res_config_settings_view_form` | form |  |  |  |  | `base` |
| `base_geolocalize.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `geoloc_provider_id`, `geoloc_provider_techname`, `geoloc_provider_googlemap_key` |  |  | `base_geolocalize` |
| `base_setup.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `is_root_company`, `active_user_count`, `language_count`, `company_id`, `company_name`, `company_informations`, `company_count`, `external_report_layout_id`, `company_id`, `module_account_inter_company_rules`, `module_sms`, `module_partner_autocomplete`, `module_base_import`, `show_effect`, `web_app_name`, `module_mail_plugin`, `module_auth_oauth`, `module_auth_ldap`, `module_web_unsplash`, `module_base_geolocalize`, `module_google_recaptcha`, `module_website_cf_turnstile`, `module_google_address_autocomplete`, `profiling_enabled_until` | `Manage Users`, `Add Languages`, `Manage Languages`, `Update Info`, `Manage Companies`, `Configure Document Layout`, `Edit Layout`, `Preview Document`, `Default Access Rights`, `Manage API Keys` |  | `base_setup` |
| `base_vat.res_config_settings_view_form` | setting | `account.res_config_settings_view_form` | `vat_check_vies` |  |  | `base_vat` |
| `calendar.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `module_microsoft_calendar`, `module_google_calendar` |  |  | `calendar` |
| `certificate.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` |  | `Certificates`, `Keys` |  | `certificate` |
| `cloud_storage.cloud_storage_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `cloud_storage_provider`, `cloud_storage_min_file_size_mb` |  |  | `cloud_storage` |
| `cloud_storage_azure.cloud_storage_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `cloud_storage_azure_account_name`, `cloud_storage_azure_container_name`, `cloud_storage_azure_tenant_id`, `cloud_storage_azure_client_id`, `cloud_storage_azure_client_secret`, `cloud_storage_azure_invalidate_user_delegation_key` |  |  | `cloud_storage_azure` |
| `cloud_storage_google.cloud_storage_google_config_settings_view_form` | xpath | `cloud_storage.cloud_storage_config_settings_view_form` | `cloud_storage_google_bucket_name`, `cloud_storage_google_service_account_key`, `cloud_storage_google_account_info` |  |  | `cloud_storage_google` |
| `cloud_storage_migration.cloud_storage_migration_config_settings_view_form` | xpath | `cloud_storage.cloud_storage_config_settings_view_form` | `cloud_storage_migration_progress`, `cloud_storage_migration_message_model_ids`, `cloud_storage_migration_all_model_ids` | `Cron Job`, `Parameters`, `Attachments Report` |  | `cloud_storage_migration` |
| `crm.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_use_recurring_revenues`, `group_use_lead`, `is_membership_multi`, `module_partnership`, `predictive_lead_scoring_fields_str`, `predictive_lead_scoring_start_date_str`, `predictive_lead_scoring_field_labels`, `predictive_lead_scoring_start_date`, `crm_use_auto_assignment`, `crm_auto_assignment_action`, `crm_auto_assignment_interval_number`, `crm_auto_assignment_interval_type`, `crm_auto_assignment_run_datetime`, `module_crm_iap_enrich`, `lead_enrich_auto`, `module_crm_iap_mine`, `module_website_crm_iap_reveal` | `Manage Recurring Plans`, `Update Probabilities`, `action_crm_assign_leads` |  | `crm` |
| `crm_iap_enrich.res_config_settings_view_form` | field | `crm.res_config_settings_view_form` | `lead_enrich_auto` |  |  | `crm_iap_enrich` |
| `crm_iap_mine.res_config_settings_view_form` | setting | `crm.res_config_settings_view_form` |  |  |  | `crm_iap_mine` |
| `delivery.res_config_settings_view_form` | setting | `sale.res_config_settings_view_form` |  | `Shipping Methods` |  | `delivery` |
| `digest.res_config_settings_view_form` | xpath | `mail.res_config_settings_view_form` | `digest_emails`, `digest_id` | `Configure Digest Emails` |  | `digest` |
| `event.res_config_settings_view_form` | xpath | `mail.res_config_settings_view_form` | `module_website_event_track`, `module_website_event_track_live`, `module_website_event_track_quiz`, `module_website_event_exhibitor`, `module_event_booth`, `module_event_sale`, `module_pos_event`, `module_website_event_sale`, `use_event_barcode`, `barcode_nomenclature_id` |  |  | `event` |
| `fleet.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `delay_alert_contract` |  |  | `fleet` |
| `google_address_autocomplete.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `google_places_api_key` |  |  | `google_address_autocomplete` |
| `google_calendar.res_config_settings_view_form` | div | `calendar.res_config_settings_view_form` | `cal_client_id`, `cal_client_secret`, `cal_sync_paused` |  |  | `google_calendar` |
| `google_gmail.res_config_settings_view_form` | div | `mail.res_config_settings_view_form` | `google_gmail_client_identifier`, `google_gmail_client_secret` |  |  | `google_gmail` |
| `google_recaptcha.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` |  |  |  | `google_recaptcha` |
| `hr.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `module_hr_attendance`, `hr_presence_control_login`, `module_hr_presence`, `hr_presence_control_email`, `hr_presence_control_ip`, `hr_presence_control_email_amount`, `hr_presence_control_ip_list`, `module_hr_skills`, `resource_calendar_id`, `contract_expiration_notice_period`, `work_permit_expiration_notice_period` |  |  | `hr` |
| `hr_attendance.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `attendance_kiosk_mode`, `attendance_from_systray`, `auto_check_out`, `auto_check_out_tolerance`, `absence_management`, `attendance_device_tracking`, `attendance_barcode_source`, `attendance_kiosk_delay`, `attendance_kiosk_use_pin`, `attendance_kiosk_url`, `overtime_company_threshold`, `overtime_employee_threshold`, `hr_attendance_display_overtime`, `attendance_overtime_validation` | `Generate new URL` |  | `hr_attendance` |
| `hr_expense.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `hr_expense_use_mailgateway`, `hr_expense_alias_prefix`, `hr_expense_alias_domain_id`, `module_hr_payroll_expense`, `module_hr_expense_extract`, `module_hr_expense_stripe`, `expense_journal_id`, `company_expense_allowed_payment_method_line_ids` |  |  | `hr_expense` |
| `hr_recruitment.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `module_website_hr_recruitment`, `module_hr_recruitment_survey`, `module_hr_recruitment_extract` |  |  | `hr_recruitment` |
| `hr_recruitment_survey.res_config_settings_view_form` | setting | `hr_recruitment.res_config_settings_view_form` |  | `Interview Survey` |  | `hr_recruitment_survey` |
| `hr_timesheet.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `project_time_mode_id`, `timesheet_encode_method`, `is_encode_uom_days`, `reminder_user_allow`, `reminder_allow`, `module_project_timesheet_holidays` |  |  | `hr_timesheet` |
| `iap.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` |  | `View My Services` |  | `iap` |
| `l10n_account_withholding_tax.res_config_settings_form` | block | `account.res_config_settings_view_form` | `withholding_tax_base_account_id` |  |  | `l10n_account_withholding_tax` |
| `l10n_ar.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` |  |  |  | `l10n_ar` |
| `l10n_ar_website_sale.res_config_settings_view_form` | setting | `website_sale.res_config_settings_view_form` | `l10n_ar_website_sale_show_both_prices` |  |  | `l10n_ar_website_sale` |
| `l10n_ar_withholding.res_config_settings_view_form` | xpath | `l10n_ar.res_config_settings_view_form` | `l10n_ar_tax_base_account_id` |  |  | `l10n_ar_withholding` |
| `l10n_cl.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` |  |  |  | `l10n_cl` |
| `l10n_din5008.res_config_settings_view_form` | setting | `account.res_config_settings_view_form` | `has_position_column` |  |  | `l10n_din5008` |
| `l10n_dk_nemhandel.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `nemhandel_edi_identification`, `nemhandel_purchase_journal_id`, `nemhandel_contact_email` | `Update contact details`, `Deregister`, `Start sending via Nemhandel` |  | `l10n_dk_nemhandel` |
| `l10n_eg_edi_eta.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `country_code`, `l10n_eg_production_env`, `l10n_eg_client_identifier`, `l10n_eg_client_secret`, `l10n_eg_invoicing_threshold` |  |  | `l10n_eg_edi_eta` |
| `l10n_es.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_es_simplified_invoice_limit` |  |  | `l10n_es` |
| `l10n_es_edi_sii.res_config_settings_view_form` | xpath | `l10n_es.res_config_settings_view_form` |  |  |  | `l10n_es_edi_sii` |
| `l10n_es_edi_tbai.res_config_settings_view_form` | xpath | `l10n_es.res_config_settings_view_form` |  |  |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_es_edi_verifactu_required`, `l10n_es_edi_verifactu_test_environment`, `l10n_es_edi_verifactu_special_vat_regime` | `%(l10n_es_edi_verifactu_certificate_action)d` |  | `l10n_es_edi_verifactu` |
| `l10n_es_pos.res_config_settings_view_form` | xpath | `point_of_sale.res_config_settings_view_form` | `pos_l10n_es_simplified_invoice_journal_id` |  |  | `l10n_es_pos` |
| `l10n_fr_hr_holidays.res_config_settings_view_form` | block | `base.res_config_settings_view_form` | `company_country_code`, `l10n_fr_reference_leave_type` |  |  | `l10n_fr_hr_holidays` |
| `l10n_fr_pdp.res_config_settings_view_form` | xpath | `account_peppol.res_config_settings_view_form` |  |  |  | `l10n_fr_pdp` |
| `l10n_fr_pos_cert.res_config_settings_view_form` | form | `point_of_sale.res_config_settings_view_form` | `country_code` |  |  | `l10n_fr_pos_cert` |
| `l10n_gcc_invoice.res_config_settings_view_form_inherit_l10n_gcc_invoice` | block | `account.res_config_settings_view_form` | `l10n_gcc_dual_language_invoice` |  |  | `l10n_gcc_invoice` |
| `l10n_gcc_pos.res_config_settings_view_form_inherit_l10n_gcc_pos` | block | `point_of_sale.res_config_settings_view_form` | `l10n_gcc_dual_language_receipt` |  |  | `l10n_gcc_pos` |
| `l10n_gr_edi.res_config_settings_form_inherit_l10n_gr_edi` | xpath | `account.res_config_settings_view_form` | `l10n_gr_edi_aade_id`, `l10n_gr_edi_aade_key`, `l10n_gr_edi_branch_number`, `l10n_gr_edi_test_env` |  |  | `l10n_gr_edi` |
| `l10n_hr_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_hr_mer_username`, `l10n_hr_mer_password`, `l10n_hr_mer_company_ident`, `l10n_hr_mer_company_bu`, `l10n_hr_mer_software_ident`, `l10n_hr_mer_connection_state`, `l10n_hr_mer_purchase_journal_id`, `l10n_hr_mer_connection_mode` | `Activate`, `Deactivate` |  | `l10n_hr_edi` |
| `l10n_hu_edi.res_config_settings_form_inherit_l10n_hu_edi` | xpath | `account.res_config_settings_view_form` | `l10n_hu_edi_is_active`, `l10n_hu_edi_server_mode`, `l10n_hu_edi_username`, `l10n_hu_edi_password`, `l10n_hu_edi_signature_key`, `l10n_hu_edi_replacement_key`, `l10n_hu_tax_regime` |  |  | `l10n_hu_edi` |
| `l10n_in.res_config_settings_view_form_inherit_l10n_in` | block | `account.res_config_settings_view_form` | `group_l10n_in_reseller` |  |  | `l10n_in` |
| `l10n_in_edi.res_config_settings_view_form_inherit_l10n_in_edi` | xpath | `account.res_config_settings_view_form` | `l10n_in_edi_feature` |  |  | `l10n_in_edi` |
| `l10n_in_ewaybill.res_config_settings_view_form_inherit_l10n_in_edi_ewaybill` | xpath | `account.res_config_settings_view_form` | `l10n_in_ewaybill_feature` |  |  | `l10n_in_ewaybill` |
| `l10n_in_pos.res_config_settings_view_form_l10n_in_pos_inherit` | xpath | `point_of_sale.res_config_settings_view_form` |  |  |  | `l10n_in_pos` |
| `l10n_it_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_it_edi_show_purchase_journal_id`, `l10n_it_edi_register`, `l10n_it_edi_purchase_journal_id` |  |  | `l10n_it_edi` |
| `l10n_jo_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` |  |  |  | `l10n_jo_edi` |
| `l10n_jo_edi_pos.res_config_settings_view_form` | block | `point_of_sale.res_config_settings_view_form` | `l10n_jo_edi_pos_enabled`, `l10n_jo_edi_pos_testing_mode` |  |  | `l10n_jo_edi_pos` |
| `l10n_ke_edi_tremol.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_ke_cu_proxy_address` |  |  | `l10n_ke_edi_tremol` |
| `l10n_mx.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_mx_account_income_return_discount_id` |  |  | `l10n_mx` |
| `l10n_my_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_my_edi_proxy_user_id`, `l10n_my_edi_mode`, `l10n_my_accept_processing`, `l10n_my_edi_company_vat` | `Register`, `Unregister` |  | `l10n_my_edi` |
| `l10n_nl.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` |  |  |  | `l10n_nl` |
| `l10n_pl.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_pl_reports_tax_office_id` |  |  | `l10n_pl` |
| `l10n_pl_edi.res_config_settings_view_form_l10n_pl_edi` | xpath | `account.res_config_settings_view_form` | `l10n_pl_edi_register`, `l10n_pl_edi_certificate` |  |  | `l10n_pl_edi` |
| `l10n_ro_edi.res_config_settings_form_inherit_l10n_ro_edi` | xpath | `account.res_config_settings_view_form` | `l10n_ro_edi_callback_url`, `l10n_ro_edi_client_id`, `l10n_ro_edi_client_secret`, `l10n_ro_edi_access_token`, `l10n_ro_edi_refresh_token`, `l10n_ro_edi_access_expiry_date`, `l10n_ro_edi_refresh_expiry_date`, `l10n_ro_edi_anaf_imported_inv_journal_id`, `l10n_ro_edi_test_env` | `Generate Token` |  | `l10n_ro_edi` |
| `l10n_ro_edi_stock.res_config_settings_form_inherit_l10n_ro_edi` | xpath | `account.res_config_settings_view_form` |  |  |  | `l10n_ro_edi_stock` |
| `l10n_rs_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_rs_edi_api_key`, `l10n_rs_edi_demo_env` |  |  | `l10n_rs_edi` |
| `l10n_sa_edi.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `country_code`, `l10n_sa_api_mode` |  |  | `l10n_sa_edi` |
| `l10n_tr_nilvera.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_tr_nilvera_vat`, `l10n_tr_nilvera_api_key`, `l10n_tr_nilvera_purchase_journal_id`, `l10n_tr_nilvera_use_test_env` | `nilvera_ping` |  | `l10n_tr_nilvera` |
| `l10n_tr_nilvera_einvoice_extended.res_config_settings_view_form_l10n_tr_nilvera_extended` | xpath | `l10n_tr_nilvera.res_config_settings_view_form` | `l10n_tr_nilvera_export_alias` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.res_config_settings_view_form` | xpath | `account.res_config_settings_view_form` | `l10n_tw_edi_ecpay_staging_mode`, `l10n_tw_edi_ecpay_merchant_id`, `l10n_tw_edi_ecpay_hashkey`, `l10n_tw_edi_ecpay_hashIV` |  |  | `l10n_tw_edi_ecpay` |
| `l10n_vn_edi_viettel.res_config_settings_view_form_inherit_l10n_vn_edi` | xpath | `account.res_config_settings_view_form` | `l10n_vn_edi_username`, `l10n_vn_edi_password`, `l10n_vn_edi_default_symbol` |  |  | `l10n_vn_edi_viettel` |
| `l10n_vn_edi_viettel_pos.res_config_settings_view_form_inherit_l10n_vn_edi_pos_account` | xpath | `account.res_config_settings_view_form` | `l10n_vn_edi_pos_default_symbol` |  |  | `l10n_vn_edi_viettel_pos` |
| `l10n_vn_edi_viettel_pos.res_config_settings_view_form_inherit_l10n_vn_edi_pos_pos` | xpath | `point_of_sale.res_config_settings_view_form` | `pos_l10n_vn_auto_send_to_sinvoice`, `pos_l10n_vn_pos_symbol` |  |  | `l10n_vn_edi_viettel_pos` |
| `lunch.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `currency_id`, `company_lunch_minimum_threshold`, `company_lunch_notify_message` |  |  | `lunch` |
| `mail.res_config_settings_view_form` | div | `base_setup.res_config_settings_view_form` | `external_email_server_default`, `alias_domain_id`, `module_google_gmail`, `module_microsoft_outlook`, `restrict_template_rendering`, `use_twilio_rtc_servers`, `twilio_account_sid`, `twilio_account_token`, `use_sfu_server`, `sfu_server_url`, `sfu_server_key`, `tenor_api_key`, `google_translate_api_key` | `Incoming Email Servers`, `Outgoing Email Servers`, `Activity Types`, `Configure ICE Servers` |  | `mail` |
| `maintenance.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `module_maintenance_worksheet` |  |  | `maintenance` |
| `mass_mailing.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_mass_mailing_campaign`, `mass_mailing_split_contact_name`, `show_blacklist_buttons`, `mass_mailing_reports`, `mass_mailing_outgoing_mail_server`, `mass_mailing_mail_server_id` | `Configure Outgoing Mail Servers` |  | `mass_mailing` |
| `microsoft_calendar.res_config_settings_view_form` | div | `calendar.res_config_settings_view_form` | `cal_microsoft_client_id`, `cal_microsoft_client_secret`, `cal_microsoft_sync_paused` |  |  | `microsoft_calendar` |
| `microsoft_outlook.res_config_settings_view_form` | div | `base_setup.res_config_settings_view_form` | `microsoft_outlook_client_identifier`, `microsoft_outlook_client_secret` |  |  | `microsoft_outlook` |
| `mrp.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_mrp_routings`, `group_mrp_workorder_dependencies`, `module_mrp_subcontracting`, `module_stock_barcode`, `module_quality_control`, `module_quality_control_worksheet`, `group_unlocked_by_default`, `group_mrp_byproducts`, `group_mrp_reception_report`, `module_mrp_mps` | `Work Centers` |  | `mrp` |
| `partner_autocomplete.res_config_settings_view_form` | setting | `base_setup.res_config_settings_view_form` |  |  |  | `partner_autocomplete` |
| `partnership.res_config_settings_view_form` | field | `crm.res_config_settings_view_form` | `module_partnership`, `partnership_label` |  |  | `partnership` |
| `point_of_sale.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `is_kiosk_mode`, `pos_selectable_categ_ids`, `pos_has_active_session`, `pos_allowed_pricelist_ids`, `pos_cash_control`, `pos_iface_print_via_proxy`, `pos_company_has_template`, `group_cash_rounding`, `pos_config_id`, `pos_module_pos_restaurant`, `pos_use_presets`, `pos_available_preset_ids`, `pos_default_preset_id`, `pos_payment_method_ids`, `pos_auto_validate_terminal_payment`, `pos_cash_rounding`, `pos_rounding_method`, `pos_only_round_cash_method`, `pos_use_fast_payment`, `pos_fast_payment_method_ids`, `pos_set_maximum_difference`, `pos_amount_authorized_diff`, `pos_iface_tipproduct`, `pos_tip_product_id`, `pos_module_pos_hr`, `pos_iface_big_scrollbars`, `pos_trusted_config_ids`, `pos_show_product_images`, `pos_show_category_images`, `pos_module_pos_appointment`, `pos_iface_group_by_categ`, `pos_limit_categories`, `pos_iface_available_categ_ids`, `pos_is_margins_costs_accessible_to_every_user`, `sale_tax_id`, `account_default_pos_receivable_account_id`, `pos_order_edit_tracking`, `pos_tax_regime_selection`, `pos_default_fiscal_position_id`, `pos_fiscal_position_ids`, `pos_journal_id`, `pos_invoice_journal_id`, `pos_is_closing_entry_by_product`, `pos_module_pos_avatax`, `pos_use_pricelist`, `pos_available_pricelist_ids`, `pos_pricelist_id`, `pos_restrict_price_control`, `pos_iface_tax_included`, `pos_manual_discount`, `pos_module_pos_discount`, `module_pos_pricer`, `module_loyalty`, `pos_is_header_or_footer`, `pos_receipt_header`, `pos_receipt_footer`, `pos_iface_print_auto`, `pos_iface_print_skip_screen`, `pos_module_pos_sms`, `point_of_sale_use_ticket_qr_code` | `+ New Shop`, `pos_close_ui`, `Configure Presets`, `Payment Methods`, `Cash Roundings`, `Point of Sales`, `PoS Product Categories`, `Taxes`, `Fiscal Positions`, `Pricelists`, `Payment method`, `Payment method`, `Payment method`, `Payment method`, `Payment method`, `Payment method`, `Payment method`, `Add Printer`, `Manage Printers`, `Notes` |  | `point_of_sale` |
| `portal.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `portal_allow_api_keys` |  |  | `portal` |
| `pos_adyen.res_config_settings_view_form` | xpath | `point_of_sale.res_config_settings_view_form` | `pos_adyen_ask_customer_for_tip` |  |  | `pos_adyen` |
| `pos_discount.res_config_settings_view_form` | div | `point_of_sale.res_config_settings_view_form` | `pos_discount_product_id`, `pos_discount_pc` |  |  | `pos_discount` |
| `pos_hr.res_config_settings_view_form` | xpath | `point_of_sale.res_config_settings_view_form` | `pos_advanced_employee_ids`, `pos_basic_employee_ids`, `pos_minimal_employee_ids` |  |  | `pos_hr` |
| `pos_imin.res_config_settings_view_form_inherit_pos_imin` | xpath | `point_of_sale.res_config_settings_view_form` |  |  |  | `pos_imin` |
| `pos_loyalty.res_config_view_form_inherit_pos_loyalty` | xpath | `point_of_sale.res_config_settings_view_form` |  | `Discount & Loyalty`, `Gift cards & eWallet` |  | `pos_loyalty` |
| `pos_online_payment.res_config_settings_view_form` | xpath | `point_of_sale.res_config_settings_view_form` |  |  |  | `pos_online_payment` |
| `pos_online_payment_self_order.res_config_settings_view_form_menu` | xpath | `point_of_sale.res_config_settings_view_form` | `pos_self_order_online_payment_method_id` | `Payment Methods` |  | `pos_online_payment_self_order` |
| `pos_restaurant.res_config_settings_view_form` | div | `point_of_sale.res_config_settings_view_form` |  |  |  | `pos_restaurant` |
| `pos_sale.res_config_settings_view_form` | block | `point_of_sale.res_config_settings_view_form` | `pos_crm_team_id`, `pos_down_payment_product_id` |  |  | `pos_sale` |
| `pos_self_order.res_config_settings_view_form_menu` | block | `point_of_sale.res_config_settings_view_form` | `pos_self_ordering_mode`, `pos_self_ordering_service_mode`, `pos_self_ordering_default_user_id`, `pos_self_ordering_pay_after`, `pos_self_ordering_image_home_ids`, `pos_self_ordering_default_language_id`, `pos_self_ordering_available_language_ids`, `pos_self_ordering_image_background_ids` | `Preview Web interface`, `Home buttons`, `Print QR Codes`, `Download QR Codes`, `Free Metal / Wood Stands`, `Reset QR Codes`, `Add Languages` |  | `pos_self_order` |
| `pos_self_order_sale.res_config_settings_view_form_menu` | setting | `pos_sale.res_config_settings_view_form` |  |  |  | `pos_self_order_sale` |
| `pos_sms.pos_sms_res_config_settings_view_form` | div | `point_of_sale.res_config_settings_view_form` | `pos_sms_receipt_template_id` |  |  | `pos_sms` |
| `product.res_config_settings_view_form` | xpath | `base_setup.res_config_settings_view_form` | `product_weight_in_lbs`, `product_volume_volume_in_cubic_feet` |  |  | `product` |
| `product_expiry.res_config_settings_view_form_stock` | xpath | `stock.res_config_settings_view_form` | `group_expiry_date_on_delivery_slip` |  |  | `product_expiry` |
| `project.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_project_stages`, `module_hr_timesheet` | `Configure Stages` |  | `project` |
| `project_timesheet_holidays.res_config_settings_view_form` | xpath | `hr_timesheet.res_config_settings_view_form` | `internal_project_id`, `leave_timesheet_task_id` |  |  | `project_timesheet_holidays` |
| `purchase.res_config_settings_view_form_purchase` | xpath | `base.res_config_settings_view_form` | `po_double_validation`, `company_currency_id`, `po_lock`, `po_order_approval`, `po_double_validation_amount`, `lock_confirmed_po`, `group_warning_purchase`, `module_purchase_requisition`, `group_send_reminder`, `module_account_3way_match`, `group_product_variant`, `module_purchase_product_matrix`, `group_uom` | `Attributes`, `Units & Packagings` |  | `purchase` |
| `purchase_requisition.res_config_settings_view_form_purchase_requisition` | xpath | `purchase.res_config_settings_view_form_purchase` | `group_purchase_alternatives` |  |  | `purchase_requisition` |
| `purchase_stock.res_config_settings_view_form_purchase` | xpath | `purchase.res_config_settings_view_form_purchase` | `is_installed_sale`, `module_stock_dropshipping`, `replenish_on_order` |  |  | `purchase_stock` |
| `purchase_stock.res_config_settings_view_form_stock` | div | `stock.res_config_settings_view_form` | `days_to_purchase` |  |  | `purchase_stock` |
| `sale.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_product_variant`, `module_sale_product_matrix`, `group_uom`, `module_product_email_template`, `group_discount_per_so_line`, `module_loyalty`, `group_product_pricelist`, `auth_signup_uninvited`, `module_sale_margin`, `portal_confirmation_sign`, `portal_confirmation_pay`, `prepayment_percent`, `onboarding_payment_module`, `active_provider_id`, `quotation_validity_days`, `group_warning_sale`, `module_sale_pdf_quote_builder`, `group_auto_done_setting`, `group_proforma_sales`, `module_delivery`, `module_delivery_bpost`, `module_delivery_easypost`, `module_delivery_sendcloud`, `module_delivery_shiprocket`, `module_delivery_starshipit`, `module_delivery_envia`, `default_invoice_policy`, `automatic_invoice`, `invoice_mail_template_id`, `module_sale_commission`, `module_sale_amazon`, `module_sale_gelato`, `module_sale_shopee` | `Attributes`, `Units & Packagings`, `Pricelists`, `action_sale_start_payment_onboarding`, `Activate Stripe`, `View Alternatives`, `action_view_active_provider`, `View Other Providers` |  | `sale` |
| `sale.res_config_settings_view_form_sale_inherit` | xpath | `account.res_config_settings_view_form` | `downpayment_account_id` |  |  | `sale` |
| `sale_gelato.res_config_settings_form` | div | `sale.res_config_settings_view_form` | `gelato_api_key`, `gelato_webhook_secret` | `Manage Delivery Methods` |  | `sale_gelato` |
| `sale_management.res_config_settings_view_form` | setting | `sale.res_config_settings_view_form` | `group_sale_order_template`, `company_so_template_id` | `Quotation Templates` |  | `sale_management` |
| `sale_pdf_quote_builder.res_config_settings_view_form` | div | `sale_management.res_config_settings_view_form` |  | `Headers/Footers` |  | `sale_pdf_quote_builder` |
| `sale_stock.res_config_settings_view_form_stock` | setting | `stock.res_config_settings_view_form` | `default_picking_policy` |  |  | `sale_stock` |
| `sale_timesheet.res_config_settings_view_form` | xpath | `hr_timesheet.res_config_settings_view_form` | `invoice_policy` | `Configure your services` |  | `sale_timesheet` |
| `sms.res_config_settings_view_form` | setting | `base_setup.res_config_settings_view_form` |  |  |  | `sms` |
| `sms_twilio.res_config_settings_view_form` | xpath | `sms.res_config_settings_view_form` | `sms_provider` | `Configure Twilio Account` |  | `sms_twilio` |
| `snailmail_account.res_config_settings_view_form` | setting | `account.res_config_settings_view_form` | `snailmail_color`, `snailmail_duplex`, `snailmail_cover` |  |  | `snailmail_account` |
| `stock.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `group_stock_tracking_lot`, `module_stock_picking_batch`, `group_warning_stock`, `module_quality_control`, `module_quality_control_worksheet`, `annual_inventory_day`, `annual_inventory_month`, `group_stock_reception_report`, `module_stock_barcode`, `module_stock_barcode_barcodelookup`, `module_delivery`, `module_stock_fleet`, `stock_move_email_validation`, `stock_text_confirmation`, `module_stock_sms`, `stock_confirmation_type`, `group_stock_sign_delivery`, `module_delivery_bpost`, `module_delivery_easypost`, `module_delivery_sendcloud`, `module_delivery_shiprocket`, `module_delivery_starshipit`, `module_delivery_envia`, `group_product_variant`, `group_uom`, `group_stock_production_lot`, `group_stock_lot_print_gs1`, `barcode_separator`, `module_product_expiry`, `group_lot_on_delivery_slip`, `group_stock_tracking_owner`, `group_stock_multi_locations`, `group_stock_adv_location`, `horizon_days`, `module_stock_dropshipping`, `replenish_on_order` | `Attributes`, `Units & Packagings`, `Locations`, `Set Warehouse Routes` |  | `stock` |
| `stock_account.res_config_settings_view_form` | block | `stock.res_config_settings_view_form` | `module_stock_landed_costs`, `group_lot_on_invoice` |  |  | `stock_account` |
| `stock_landed_costs.res_config_settings_view_form` | div | `stock.res_config_settings_view_form` | `lc_journal_id` |  |  | `stock_landed_costs` |
| `stock_sms.res_config_settings_view_form_stock` | xpath | `stock.res_config_settings_view_form` |  |  |  | `stock_sms` |
| `web_unsplash.res_config_settings_view_form` | div | `base_setup.res_config_settings_view_form` | `unsplash_access_key`, `unsplash_app_id` |  |  | `web_unsplash` |
| `website.res_config_settings_view_form_inherit_auth_signup` | xpath | `auth_signup.res_config_settings_view_form` |  |  |  | `website` |
| `website.res_config_settings_view_form` | xpath | `base.res_config_settings_view_form` | `website_id`, `website_domain`, `language_ids`, `website_language_count`, `website_default_lang_id`, `website_name`, `favicon`, `website_company_id`, `shared_user_account`, `auth_signup_uninvited`, `module_website_livechat`, `has_plausible_shared_key`, `plausible_shared_key`, `plausible_site`, `has_google_analytics`, `google_analytics_key`, `cdn_activated`, `cdn_url`, `cdn_filters`, `has_default_share_image`, `social_default_image`, `has_google_search_console`, `google_search_console`, `website_cookies_bar`, `website_block_third_party_domains` | `+ New Website`, `Install new languages`, `Default Access Rights`, `See Analytics Report`, `Edit robots.txt`, `Add domains to the block list` |  | `website` |
| `website_cf_turnstile.res_config_settings_view_form` | div | `base_setup.res_config_settings_view_form` | `turnstile_site_key`, `turnstile_secret_key` |  |  | `website_cf_turnstile` |
| `website_crm_iap_reveal.res_config_settings_view_form` | setting | `crm.res_config_settings_view_form` |  |  |  | `website_crm_iap_reveal` |
| `website_event_track.res_config_settings_view_form` | xpath | `website.res_config_settings_view_form` | `events_app_name` |  |  | `website_event_track` |
| `website_livechat.res_config_settings_view_form` | setting | `website.res_config_settings_view_form` | `channel_id` |  |  | `website_livechat` |
| `website_payment.res_config_settings_view_form` | setting | `website.res_config_settings_view_form` | `onboarding_payment_module`, `active_provider_id` | `action_w_payment_start_payment_onboarding`, `Activate Stripe`, `Find another provider`, `action_view_active_provider`, `Find another provider` |  | `website_payment` |
| `website_sale.res_config_settings_view_form_inherit_sale` | setting | `sale.res_config_settings_view_form` |  |  |  | `website_sale` |
| `website_sale.res_config_settings_view_form` | setting | `website_payment.res_config_settings_view_form` | `module_website_sale_autocomplete` |  |  | `website_sale` |
| `website_sale_autocomplete.res_config_settings_view_form_inherit_autocomplete_googleplaces` | xpath | `website_sale.res_config_settings_view_form` | `website_google_places_api_key` |  |  | `website_sale_autocomplete` |
| `website_sale_collect.res_config_settings_form` | setting | `website_sale.res_config_settings_view_form` |  | `Configure Pickup Locations` |  | `website_sale_collect` |
| `website_sale_loyalty.res_config_settings_view_form_inherit_website_sale_loyalty` | setting | `website_sale.res_config_settings_view_form` |  | `Loyalty Programs` |  | `website_sale_loyalty` |
| `website_sale_mass_mailing.res_config_settings_view_form` | setting | `website_sale.res_config_settings_view_form` | `is_newsletter_enabled`, `newsletter_id` |  |  | `website_sale_mass_mailing` |
| `website_sale_stock.res_config_settings_view_form` | setting | `website_sale.res_config_settings_view_form` | `website_company_id`, `website_warehouse_id`, `default_allow_out_of_stock_order`, `default_show_availability`, `default_available_threshold` |  |  | `website_sale_stock` |
| `website_slides.res_config_settings_view_form` | xpath | `website.res_config_settings_view_form` | `module_website_slides_survey`, `module_website_sale_slides`, `module_mass_mailing_slides`, `module_website_slides_forum`, `website_slide_google_app_key` |  |  | `website_slides` |
| `website_slides_forum.res_config_settings_view_form` | xpath | `website_slides.res_config_settings_view_form` |  | `Manage Forums` |  | `website_slides_forum` |
| `website_slides_survey.res_config_settings_view_form` | xpath | `website_slides.res_config_settings_view_form` |  | `Manage Certifications` |  | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_open_settings` | Settings | form |  | `{'module': 'general_settings', 'bin_size': False}` |  | `account` |
| `account.action_account_config` | Settings | form |  | `{'module' : 'account', 'bin_size': False}` |  | `account` |
| `base.res_config_setting_act_window` | Settings | form |  |  |  | `base` |
| `base_setup.action_general_configuration` | Settings | form |  | `{'module' : 'general_settings', 'bin_size': False}` |  | `base_setup` |
| `calendar.calendar_settings_action` | Settings | form |  | `{'module' : 'calendar', 'bin_size': False}` |  | `calendar` |
| `crm.crm_config_settings_action` | Settings | form |  | `{'module' : 'crm', 'bin_size': False}` |  | `crm` |
| `event.action_event_configuration` | Settings | form |  | `{'module' : 'event', 'bin_size': False}` |  | `event` |
| `fleet.fleet_config_settings_action` | Settings | form |  | `{'module' : 'fleet', 'bin_size': False}` |  | `fleet` |
| `hr.hr_config_settings_action` | Settings | form |  | `{'module' : 'hr', 'bin_size': False}` |  | `hr` |
| `hr_attendance.action_hr_attendance_settings` | Settings | form |  | `{'module' : 'hr_attendance', 'bin_size': False}` |  | `hr_attendance` |
| `hr_expense.action_hr_expense_configuration` | Settings | form |  | `{'module' : 'hr_expense', 'bin_size': False}` |  | `hr_expense` |
| `hr_recruitment.action_hr_recruitment_configuration` | Settings | form |  | `{'module' : 'hr_recruitment', 'bin_size': False}` |  | `hr_recruitment` |
| `hr_timesheet.hr_timesheet_config_settings_action` | Settings | form |  | `{'module' : 'hr_timesheet', 'bin_size': False}` |  | `hr_timesheet` |
| `lunch.lunch_config_settings_action` | Settings | form |  | `{'module' : 'lunch', 'bin_size': False}` |  | `lunch` |
| `maintenance.action_maintenance_configuration` | Settings | form |  | `{'module' : 'maintenance', 'bin_size': False}` |  | `maintenance` |
| `mass_mailing.action_mass_mailing_configuration` | Settings | form |  | `{'module' : 'mass_mailing', 'bin_size': False}` |  | `mass_mailing` |
| `mrp.action_mrp_configuration` | Settings | form |  | `{'module' : 'mrp', 'bin_size': False}` |  | `mrp` |
| `point_of_sale.action_pos_configuration` | Settings | form |  | `{'module' : 'point_of_sale', 'bin_size': False}` |  | `point_of_sale` |
| `project.project_config_settings_action` | Settings | form |  | `{'module' : 'project', 'bin_size': False}` |  | `project` |
| `purchase.action_purchase_configuration` | Settings | form |  | `{'module' : 'purchase', 'bin_size': False}` |  | `purchase` |
| `sale.action_sale_config_settings` | Settings | form |  | `{'module' : 'sale_management', 'bin_size': False}` |  | `sale` |
| `stock.action_stock_config_settings` | Settings | form |  | `{'module' : 'stock', 'bin_size': False}` |  | `stock` |
| `website.action_website_configuration` | Settings | form |  | `{'module' : 'website', 'bin_size': False}` |  | `website` |
| `website_slides.website_slides_action_settings` | Settings | form |  | `{'module': 'website_slides', 'bin_size': False}` |  | `website_slides` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.crm_config_settings_menu` | Settings | `crm_menu_config` | `crm.crm_config_settings_action` | 0 | `base.group_system` |
| `l10n_fr_hr_holidays.hr_holidays_menu_configuration` | Settings | `hr_holidays.menu_hr_holidays_configuration` | `hr.hr_config_settings_action` | 10 | `base.group_system` |

Machine-readable definition: `../../../schemas/data/entities/res.config.settings.json`; views: `../../../schemas/interfaces/views/res.config.settings.json`.
