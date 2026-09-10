# Companies (`res.company`)

**Transport name:** `res.company`  
**Storage name:** `res_company`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `mail`, `product`, `resource`, `account`, `account_check_printing`, `account_edi_proxy_client`, `payment`, `account_payment_interco`, `account_peppol`, `account_peppol_response`, `auth_ldap`, `barcodes`, `sms`, `base_vat`, `sale`, `stock`, `stock_account`, `sale_stock`, `sale_management`, `hr`, `hr_attendance`, `hr_expense`, `hr_holidays`, `hr_presence`, `hr_recruitment`, `social_media`, `website`, `hr_timesheet`, `l10n_account_withholding_tax`, `partner_autocomplete`, `point_of_sale`, `l10n_gcc_invoice`, `l10n_latam_invoice_document`, `l10n_latam_base`, `l10n_ar`, `website_sale`, `l10n_ar_withholding`, `l10n_din5008`, `l10n_au`, `l10n_br`, `l10n_ca`, `l10n_cl`, `l10n_cz`, `l10n_de`, `purchase`, `l10n_dk_nemhandel`, `l10n_ec`, `l10n_ee`, `l10n_eg_edi_eta`, `l10n_es`, `l10n_es_edi_facturae`, `l10n_es_edi_sii`, `l10n_es_edi_tbai`, `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_eu_oss`, `l10n_fr`, `l10n_fr_account`, `l10n_fr_hr_holidays`, `l10n_fr_pdp`, `l10n_fr_pdp_pos`, `l10n_fr_pos_cert`, `l10n_gr_edi`, `l10n_gr_edi_e_invoo`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_hu_edi_receive`, `l10n_in`, `l10n_in_edi`, `l10n_in_ewaybill`, `l10n_in_pos`, `purchase_stock`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_jo_edi`, `l10n_jo_edi_pos`, `l10n_ke`, `l10n_ke_edi_tremol`, `l10n_lk_invoice`, `l10n_mx`, `l10n_my_ubl_pint`, `l10n_my_edi`, `l10n_nl`, `l10n_no`, `l10n_pe`, `l10n_ph`, `l10n_pl`, `l10n_pl_edi`, `l10n_ro_edi`, `l10n_rs_edi`, `l10n_sa_edi`, `l10n_sa_edi_pos`, `l10n_se`, `l10n_sg`, `l10n_sk`, `l10n_tr_nilvera`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_uy`, `l10n_vn_edi_viettel`, `l10n_vn_edi_viettel_pos`, `lunch`, `mass_mailing`, `mrp`, `mrp_account`, `stock_landed_costs`, `mrp_subcontracting`, `stock_dropshipping`, `mrp_subcontracting_dropshipping`, `partnership`, `project_timesheet_holidays`, `sale_gelato`, `sms_twilio`, `snailmail`, `spreadsheet_account`, `stock_sms`, `website_mass_mailing`

Description: Companies

## Identity and behavior

- Mixins (classical inheritance): `format.address.mixin`, `format.vat.label.mixin`, `mail.thread`, `pos.load.mixin`
- Default ordering: `sequence, name`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (439)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Company Name | single line text |  | required; related through path `partner_id.name` and stored |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10`; Help: Used to order Companies in the company switcher |
| `parent_id` | Parent Company | many to one | `res.company` | indexed; on delete of the target: restrict |
| `child_ids` | Branches | one to many | `res.company` | inverse field `parent_id` |
| `all_child_ids` | All Child | one to many | `res.company` | inverse field `parent_id` |
| `parent_path` | Parent Path | single line text |  | indexed |
| `parent_ids` | Parent | many to many | `res.company` | computed by rule `_compute_parent_ids` (not stored) |
| `root_id` | Root | many to one | `res.company` | computed by rule `_compute_parent_ids` (not stored) |
| `partner_id` | Partner | many to one | `res.partner` | required; indexed |
| `report_header` | Company Tagline | rich text |  | translatable; Help: Company tagline, which is included in a printed document's header or footer (depending on the selected layout). |
| `report_footer` | Report Footer | rich text |  | translatable; Help: Footer text displayed at the bottom of all reports. |
| `company_details` | Company Details | rich text |  | translatable; Help: Header text displayed at the top of all reports. |
| `is_company_details_empty` | Is Company Details Empty | boolean |  | computed by rule `_compute_empty_company_details` (not stored) |
| `logo` | Company Logo | binary |  | related through path `partner_id.image_1920`; default computed dynamically (_get_logo) |
| `logo_web` | Logo Web | binary |  | computed by rule `_compute_logo_web` and stored |
| `uses_default_logo` | Uses Default Logo | boolean |  | computed by rule `_compute_uses_default_logo` and stored |
| `currency_id` | Currency | many to one | `res.currency` | required; default computed dynamically (lambda self: self._default_currency_id()) |
| `user_ids` | Accepted Users | many to many | `res.users` | association table `res_company_users_rel` |
| `street` | Street | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `street2` | Street2 | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `zip` | Zip | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `city` | City | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `state_id` | Fed. State | many to one | `res.country.state` | computed by rule `_compute_address` (not stored); writable through an inverse rule; restricted by domain `[('country_id', '=?', country_id)]` |
| `bank_ids` | Bank | one to many |  | related through path `partner_id.bank_ids` |
| `country_id` | Country | many to one | `res.country` | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |
| `email` | Email | single line text |  | related through path `partner_id.email` and stored |
| `phone` | Phone | single line text |  | related through path `partner_id.phone` and stored |
| `website` | Website | single line text |  | related through path `partner_id.website` |
| `vat` | Tax identifier | single line text |  | related through path `partner_id.vat` |
| `company_registry` | Company identifier | single line text |  | related through path `partner_id.company_registry` |
| `company_registry_placeholder` | Company Registry Placeholder | single line text |  | computed by rule `_compute_company_registry_placeholder` (not stored); related through path `partner_id.company_registry_placeholder`; extended by packages `account` |
| `paperformat_id` | Paper format | many to one | `report.paperformat` | default computed dynamically (lambda self: self.env.ref('base.paperformat_euro', raise_if_not_found=False)) |
| `external_report_layout_id` | Document Template | many to one | `ir.ui.view` |  |
| `font` | Font | selection |  | default `Lato` |
| `primary_color` | Primary Color | single line text |  |  |
| `secondary_color` | Secondary Color | single line text |  |  |
| `color` | Color | integer |  | computed by rule `_compute_color` (not stored); writable through an inverse rule |
| `layout_background` | Layout Background | selection |  | required; default `Blank` |
| `layout_background_image` | Background Image | binary |  |  |
| `uninstalled_l10n_module_ids` | Uninstalled Localization Module | many to many | `ir.module.module` | computed by rule `_compute_uninstalled_l10n_module_ids` (not stored) |
| `alias_domain_id` | Email Domain | many to one | `mail.alias.domain` | default computed dynamically (lambda self: self._default_alias_domain_id()); indexed (btree_not_null) |
| `bounce_email` | Bounce Email | single line text |  | computed by rule `_compute_bounce` (not stored) |
| `bounce_formatted` | Bounce | single line text |  | computed by rule `_compute_bounce` (not stored) |
| `catchall_email` | Catchall Email | single line text |  | computed by rule `_compute_catchall` (not stored) |
| `catchall_formatted` | Catchall | single line text |  | computed by rule `_compute_catchall` (not stored) |
| `default_from_email` | Default From | single line text |  | read only; related through path `alias_domain_id.default_from_email` |
| `email_formatted` | Formatted Email | single line text |  | computed by rule `_compute_email_formatted` (not stored) |
| `email_primary_color` | Email Button Text | single line text |  | default `#FFFFFF` |
| `email_secondary_color` | Email Button Color | single line text |  | default `#875A7B` |
| `resource_calendar_ids` | Working Hours | one to many | `resource.calendar` | inverse field `company_id` |
| `resource_calendar_id` | Default Working Hours | many to one | `resource.calendar` | on delete of the target: restrict |
| `fiscalyear_last_day` | Fiscalyear Last Day | integer |  | required; default `31` |
| `fiscalyear_last_month` | Fiscalyear Last Month | selection |  | required; default `12` |
| `fiscalyear_lock_date` | Global Lock Date | date |  | changes are tracked in the message thread; Help: Any entry up to and including that date will be postponed to a later time, in accordance with its journal's sequence. |
| `tax_lock_date` | Tax Return Lock Date | date |  | changes are tracked in the message thread; Help: Any entry with taxes up to and including that date will be postponed to a later time, in accordance with its journal's sequence. The tax lock date is automatically set when the tax closing entry is posted. |
| `sale_lock_date` | Sales Lock Date | date |  | changes are tracked in the message thread; Help: Any sales entry prior to and including this date will be postponed to a later date, in accordance with its journal's sequence. |
| `purchase_lock_date` | Purchase Lock date | date |  | changes are tracked in the message thread; Help: Any purchase entry prior to and including this date will be postponed to a later date, in accordance with its journal's sequence. |
| `hard_lock_date` | Hard Lock Date | date |  | changes are tracked in the message thread; Help: Any entry up to and including that date will be postponed to a later time, in accordance with its journal sequence. This lock date is irreversible and does not allow any exception. |
| `user_fiscalyear_lock_date` | User Fiscalyear Lock Date | date |  | computed by rule `_compute_user_fiscalyear_lock_date` (not stored) |
| `user_tax_lock_date` | User Tax Lock Date | date |  | computed by rule `_compute_user_tax_lock_date` (not stored) |
| `user_sale_lock_date` | User Sale Lock Date | date |  | computed by rule `_compute_user_sale_lock_date` (not stored) |
| `user_purchase_lock_date` | User Purchase Lock Date | date |  | computed by rule `_compute_user_purchase_lock_date` (not stored) |
| `user_hard_lock_date` | User Hard Lock Date | date |  | computed by rule `_compute_user_hard_lock_date` (not stored) |
| `transfer_account_id` | Inter-Banks Transfer Account | many to one | `account.account` | restricted by domain `[('reconcile', '=', True), ('account_type', '=', 'asset_current')]`; must belong to the same company; Help: Intermediary account used when moving money from a liquidity account to another |
| `expects_chart_of_accounts` | Expects a Chart of Accounts | boolean |  | default `True` |
| `chart_template` | Chart Template | selection |  | values provided by rule `_chart_template_selection` |
| `bank_account_code_prefix` | Prefix of the bank accounts | single line text |  |  |
| `cash_account_code_prefix` | Prefix of the cash accounts | single line text |  |  |
| `default_cash_difference_income_account_id` | Cash Difference Income | many to one | `account.account` | must belong to the same company |
| `default_cash_difference_expense_account_id` | Cash Difference Expense | many to one | `account.account` | must belong to the same company |
| `account_journal_suspense_account_id` | Journal Suspense Account | many to one | `account.account` | must belong to the same company |
| `account_journal_early_pay_discount_gain_account_id` | Cash Discount Write-Off Gain Account | many to one | `account.account` | must belong to the same company |
| `account_journal_early_pay_discount_loss_account_id` | Cash Discount Write-Off Loss Account | many to one | `account.account` | must belong to the same company |
| `transfer_account_code_prefix` | Prefix of the transfer accounts | single line text |  |  |
| `account_sale_tax_id` | Default Sale Tax | many to one | `account.tax` | must belong to the same company |
| `account_purchase_tax_id` | Default Purchase Tax | many to one | `account.tax` | must belong to the same company |
| `account_purchase_receipt_fiscal_position_id` | Default Purchase Receipt Fiscal Position | many to one | `account.fiscal.position` | must belong to the same company |
| `tax_calculation_rounding_method` | Tax Calculation Rounding Method | selection |  | default `round_globally` |
| `currency_exchange_journal_id` | Exchange Gain or Loss Journal | many to one | `account.journal` | restricted by domain `[["type", "=", "general"]]` |
| `income_currency_exchange_account_id` | Gain Exchange Rate Account | many to one | `account.account` | restricted by domain `[('internal_group', '=', 'income')]`; must belong to the same company |
| `expense_currency_exchange_account_id` | Loss Exchange Rate Account | many to one | `account.account` | restricted by domain `[('account_type', 'in', ('expense', 'expense_other'))]`; must belong to the same company |
| `anglo_saxon_accounting` | Use anglo-saxon accounting | boolean |  |  |
| `bank_journal_ids` | Bank Journals | one to many | `account.journal` | restricted by domain `[["type", "=", "bank"]]`; inverse field `company_id` |
| `incoterm_id` | Default incoterm | many to one | `account.incoterms` | Help: International Commercial Terms are a series of predefined commercial terms used in international transactions. |
| `qr_code` | Display quick response-code on invoices | boolean |  |  |
| `link_qr_code` | Display Link quick response-code | boolean |  |  |
| `display_invoice_amount_total_words` | Total amount of invoice in letters | boolean |  |  |
| `display_invoice_tax_company_currency` | Taxes in company currency | boolean |  | default `True` |
| `account_use_credit_limit` | Sales Credit Limit | boolean |  | Help: Enable the use of credit limit on partners. |
| `batch_payment_sequence_id` | Batch Payment Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `account_opening_move_id` | Opening Journal Entry | many to one | `account.move` | Help: The journal entry containing the initial balance of all this company's accounts. |
| `account_opening_journal_id` | Opening Journal | many to one | `account.journal` | related through path `account_opening_move_id.journal_id`; Help: Journal where the opening entry of this company's accounting has been posted. |
| `account_opening_date` | Opening Entry | date |  | Help: That is the date of the opening entry. |
| `invoice_terms` | Default Terms and Conditions | rich text |  | translatable |
| `terms_type` | Terms & Conditions format | selection |  | default `plain` |
| `invoice_terms_html` | Default Terms and Conditions as a Web page | rich text |  | computed by rule `_compute_invoice_terms_html` and stored; translatable |
| `account_default_pos_receivable_account_id` | Default PoS Receivable Account | many to one | `account.account` | must belong to the same company |
| `expense_accrual_account_id` | Expense Accrual Account | many to one | `account.account` | restricted by domain `[('internal_group', '=', 'liability'), ('account_type', 'not in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company; Help: Account used to move the period of an expense |
| `revenue_accrual_account_id` | Revenue Accrual Account | many to one | `account.account` | restricted by domain `[('internal_group', '=', 'asset'), ('account_type', 'not in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company; Help: Account used to move the period of a revenue |
| `automatic_entry_default_journal_id` | Automatic Entry Default Journal | many to one | `account.journal` | restricted by domain `[('type', '=', 'general')]`; must belong to the same company; Help: Journal used by default for moving the period of an entry |
| `domestic_fiscal_position_id` | Domestic Fiscal Position | many to one | `account.fiscal.position` | computed by rule `_compute_domestic_fiscal_position_id` and stored |
| `account_fiscal_country_id` | Fiscal Country | many to one | `res.country` | computed by rule `compute_account_tax_fiscal_country` and stored; Help: The country to use the tax reports from for this company |
| `account_fiscal_country_group_codes` | Account Fiscal Country Group Codes | structured document |  | computed by rule `_compute_account_fiscal_country_group_codes` (not stored) |
| `account_enabled_tax_country_ids` | l10n-used countries | many to many | `res.country` | computed by rule `_compute_account_enabled_tax_country_ids` (not stored); Help: Technical field containing the countries for which this company is using tax-related features(hence the ones for which l10n modules need to show tax-related fields). |
| `tax_exigibility` | Use Cash Basis | boolean |  |  |
| `tax_cash_basis_journal_id` | Cash Basis Journal | many to one | `account.journal` | must belong to the same company |
| `account_cash_basis_base_account_id` | Base Tax Received Account | many to one | `account.account` | must belong to the same company; Help: Account that will be set on lines created in cash basis journal entry and used to keep track of the tax base amount. |
| `account_storno` | Storno accounting | boolean |  | computed by rule `_compute_account_storno` and stored |
| `display_account_storno` | Display Account Storno | boolean |  | computed by rule `_compute_display_account_storno` (not stored) |
| `fiscal_position_ids` | Fiscal Position | one to many | `account.fiscal.position` | inverse field `company_id` |
| `multi_vat_foreign_country_ids` | Foreign value-added tax countries | many to many | `res.country` | computed by rule `_compute_multi_vat_foreign_country` (not stored); Help: Countries for which the company has a VAT number |
| `quick_edit_mode` | Quick encoding | selection |  |  |
| `account_discount_income_allocation_id` | Separate account for income discount | many to one | `account.account` |  |
| `account_discount_expense_allocation_id` | Separate account for expense discount | many to one | `account.account` |  |
| `restrictive_audit_trail` | Restrictive Audit Trail | boolean |  | changes are tracked in the message thread; Help: Enable this option to prevent deletion of journal item related logs |
| `force_restrictive_audit_trail` | Force Audit Trail | boolean |  | computed by rule `_compute_force_restrictive_audit_trail` (not stored) |
| `autopost_bills` | Auto-validate bills | boolean |  | default `True` |
| `account_price_include` | Default Sales Price Include | selection |  | required; default `tax_excluded`; Help: Default on whether the sales price used on the product and invoices with this Company includes its taxes. |
| `company_vat_placeholder` | Company Value-added tax Placeholder | single line text |  | computed by rule `_compute_company_vat_placeholder` (not stored) |
| `income_account_id` | Income Account | many to one | `account.account` | restricted by domain `ACCOUNT_DOMAIN`; Help: This account will be used when validating a customer invoice. |
| `expense_account_id` | Expense Account | many to one | `account.account` | restricted by domain `ACCOUNT_DOMAIN`; Help: The expense is accounted for when a vendor bill is validated, except in anglo-saxon accounting with perpetual inventory valuation in which case the expense (Cost of Goods Sold account) is recognized at the customer invoice validation. |
| `price_difference_account_id` | Price Difference Account | many to one | `account.account` | restricted by domain `ACCOUNT_DOMAIN`; Help: During perpetual valuation, this account will hold the price difference between the standard price and the bill price. |
| `account_check_printing_layout` | Check Layout | selection |  | default `disabled`; Help: Select the format corresponding to the check paper you will be printing your checks on. In order to disable the printing feature, select 'None'. |
| `account_check_printing_date_label` | Print Date Label | boolean |  | default `True`; Help: This option allows you to print the date label on the check as per CPA. Disable this if your pre-printed check includes the date label. |
| `account_check_printing_multi_stub` | Multi-Pages Check Stub | boolean |  | Help: This option allows you to print check details (stub) on multiple pages if they don't fit on a single page. |
| `account_check_printing_margin_top` | Check Top Margin | float |  | default `0.25`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `account_check_printing_margin_left` | Check Left Margin | float |  | default `0.25`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `account_check_printing_margin_right` | Right Margin | float |  | default `0.25`; Help: Adjust the margins of generated checks to make it fit your printer's settings. |
| `account_edi_proxy_client_ids` | Account Electronic data interchange Proxy Client | one to many | `account_edi_proxy_client.user` | inverse field `company_id` |
| `account_interco_clearing_journal_id` | Intercompany Clearing Journal | many to one | `account.journal` | restricted by domain `[["type", "=", "general"]]`; must belong to the same company; Help: The accounting journal where Intercompany payments will be cleared |
| `account_interco_payable_id` | Intercompany Clearing Payable Account | many to one | `account.account` | restricted by domain `[["account_type", "=", "liability_payable"], ["reconcile", "=", true]]`; Help: The account where Intercompany invoice payments will be cleared |
| `account_interco_receivable_id` | Intercompany Clearing Receivable Account | many to one | `account.account` | restricted by domain `[["account_type", "=", "asset_receivable"], ["reconcile", "=", true]]`; Help: The account where Intercompany credit note payments will be cleared |
| `account_peppol_contact_email` | Primary contact email | single line text |  | computed by rule `_compute_account_peppol_contact_email` and stored; Help: Primary contact email for Peppol connection related communications and notifications. In particular, this email is used by Odoo to reconnect your Peppol account in case of database change. |
| `account_peppol_migration_key` | Migration Key | single line text |  | visible only to groups `base.group_system` |
| `account_peppol_phone_number` | Mobile number | single line text |  | computed by rule `_compute_account_peppol_phone_number` and stored; Help: This number is used for identification purposes only. |
| `account_peppol_proxy_state` | PEPPOL status | selection |  | required; default `not_registered` |
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_account_peppol_edi_user` (not stored) |
| `peppol_eas` | the pan-European public procurement online network Eas | selection |  | related through path `partner_id.peppol_eas` |
| `peppol_endpoint` | the pan-European public procurement online network Endpoint | single line text |  | related through path `partner_id.peppol_endpoint` |
| `peppol_purchase_journal_id` | Peppol Purchase Journal | many to one | `account.journal` | computed by rule `_compute_peppol_purchase_journal_id` and stored; writable through an inverse rule; restricted by domain `[["type", "=", "purchase"]]` |
| `peppol_external_provider` | the pan-European public procurement online network External Provider | single line text |  | changes are tracked in the message thread |
| `peppol_can_send` | the pan-European public procurement online network Can Send | boolean |  | computed by rule `_compute_peppol_can_send` (not stored) |
| `peppol_parent_company_id` | the pan-European public procurement online network Parent Company | many to one | `res.company` | computed by rule `_compute_peppol_parent_company_id` (not stored) |
| `peppol_metadata` | Peppol Metadata | structured document |  |  |
| `peppol_metadata_updated_at` | Peppol meta updated at | date and time |  |  |
| `peppol_activate_self_billing_sending` | Activate self-billing sending | boolean |  | Help: If activated, you will be able to send vendor bills as self-billed invoices via Peppol. |
| `peppol_self_billing_reception_journal_id` | Self-Billing reception journal | many to one | `account.journal` | computed by rule `_compute_peppol_self_billing_reception_journal_id` and stored; writable through an inverse rule; restricted by domain `[["type", "=", "sale"]]`; Help: Any self-billed invoices / credit notes received via Peppol will be created in draft in this journal. Defaults to the first sale journal. |
| `ldaps` | directory access protocol Parameters | one to many | `res.company.ldap` | visible only to groups `base.group_system`; inverse field `company` |
| `nomenclature_id` | Nomenclature | many to one | `barcode.nomenclature` | default computed dynamically (_get_default_nomenclature) |
| `vat_check_vies` | Verify value-added tax Numbers | boolean |  |  |
| `portal_confirmation_sign` | Online Signature | boolean |  | default `True` |
| `portal_confirmation_pay` | Online Payment | boolean |  |  |
| `prepayment_percent` | Prepayment percentage | float |  | default `1.0`; Help: The percentage of the amount needed to be paid to confirm quotations. |
| `quotation_validity_days` | Default Quotation Validity | integer |  | default `30`; Help: Days between quotation proposal and expiration. 0 days means automatic expiration is disabled |
| `sale_discount_product_id` | Discount Product | many to one | `product.product` | restricted by domain `[["type", "=", "service"], ["invoice_policy", "=", "order"]]`; must belong to the same company; Help: Default product used for discounts |
| `sale_onboarding_payment_method` | Sale onboarding selected payment method | selection |  |  |
| `downpayment_account_id` | Downpayment Account | many to one | `account.account` | changes are tracked in the message thread; restricted by domain `[["account_type", "in", ["income", "income_other", "liability_current"]]]`; Help: This account will be used on Downpayment invoices. |
| `internal_transit_location_id` | Internal Transit Location | many to one | `stock.location` | on delete of the target: restrict; must belong to the same company |
| `stock_move_email_validation` | Email Confirmation picking | boolean |  | default  |
| `stock_mail_confirmation_template_id` | Email Template confirmation picking | many to one | `mail.template` | default computed dynamically (_default_confirmation_mail_template); restricted by domain `[('model', '=', 'stock.picking')]`; Help: Email sent to the customer once the order is done. |
| `annual_inventory_month` | Annual Inventory Month | selection |  | default `12`; Help: Annual inventory month for products not in a location with a cyclic inventory date. Set to no month if no automatic annual inventory. |
| `annual_inventory_day` | Day of the month | integer |  | default `31`; Help: Day of the month when the annual inventory should occur. If zero or negative, then the first day of the month will be selected instead.         If greater than the last day of a month, then the last day of the month will be selected instead. |
| `horizon_days` | Replenishment Horizon | float |  | required; default `365`; Help: Configure your horizon to trigger reordering rules earlier to get                                 a head start on replenishment and avoid delays, or trigger it just-in-time                                 ('0 days') to avoid overstocking. |
| `stock_text_confirmation` | Stock Text Confirmation | boolean |  |  |
| `stock_confirmation_type` | Stock Confirmation Type | selection |  | default `sms` |
| `account_stock_journal_id` | Stock Journal | many to one | `account.journal` | must belong to the same company |
| `account_stock_valuation_id` | Stock Valuation Account | many to one | `account.account` | must belong to the same company |
| `account_production_wip_account_id` | Production work in progress Account | many to one | `account.account` | must belong to the same company |
| `account_production_wip_overhead_account_id` | Production work in progress Overhead Account | many to one | `account.account` | must belong to the same company |
| `inventory_period` | Inventory Period | selection |  | required; default `manual` |
| `inventory_valuation` | Valuation | selection |  | default `periodic` |
| `cost_method` | Cost Method | selection |  | required; default `standard` |
| `security_lead` | Sales Safety Days | float |  | required; default ; Help: Margin of error for dates promised to customers. Products will be scheduled for procurement and delivery that many days earlier than the actual promised date, to cope with unexpected delays in the supply chain. |
| `sale_order_template_id` | Default Sale Template | many to one | `sale.order.template` | restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', id)]`; must belong to the same company |
| `hr_presence_control_email_amount` | # emails to send | integer |  |  |
| `hr_presence_control_ip_list` | Valid internet protocol addresses | single line text |  |  |
| `employee_properties_definition` | Employee Properties | properties definition |  |  |
| `hr_presence_control_login` | Based on user status in system | boolean |  | default `True` |
| `hr_presence_control_email` | Based on number of emails sent | boolean |  |  |
| `hr_presence_control_ip` | Based on internet protocol Address | boolean |  |  |
| `hr_presence_control_attendance` | Based on attendances | boolean |  |  |
| `contract_expiration_notice_period` | Contract Expiry Notice Period | integer |  | default `7` |
| `work_permit_expiration_notice_period` | Work Permit Expiry Notice Period | integer |  | default `60` |
| `overtime_company_threshold` | Tolerance Time In Favor Of Company | integer |  | default  |
| `overtime_employee_threshold` | Tolerance Time In Favor Of Employee | integer |  | default  |
| `hr_attendance_display_overtime` | Display Extra Hours | boolean |  |  |
| `attendance_kiosk_mode` | Attendance Mode | selection |  | default `barcode_manual` |
| `attendance_barcode_source` | Barcode Source | selection |  | default `front` |
| `attendance_kiosk_delay` | Attendance Kiosk Delay | integer |  | default `10` |
| `attendance_kiosk_key` | Attendance Kiosk Key | single line text |  | default computed dynamically (lambda s: uuid.uuid4().hex); not copied on duplication; visible only to groups `hr_attendance.group_hr_attendance_user` |
| `attendance_kiosk_url` | Attendance Kiosk Uniform resource locator | single line text |  | computed by rule `_compute_attendance_kiosk_url` (not stored) |
| `attendance_kiosk_use_pin` | Employee PIN Identification | boolean |  |  |
| `attendance_from_systray` | Attendance From Systray | boolean |  | default  |
| `attendance_overtime_validation` | Extra Hours Validation | selection |  | default `no_validation` |
| `auto_check_out` | Automatic Check Out | boolean |  | default  |
| `auto_check_out_tolerance` | Auto Check Out Tolerance | float |  | default `2` |
| `absence_management` | Absence Management | boolean |  | default  |
| `attendance_device_tracking` | Device & Location Tracking | boolean |  | default  |
| `expense_journal_id` | Default Expense Journal | many to one | `account.journal` | restricted by domain `[('type', '=', 'purchase')]`; must belong to the same company; Help: The company's default journal used when an employee expense is created. |
| `company_expense_allowed_payment_method_line_ids` | Payment methods available for expenses paid by company | many to many | `account.payment.method.line` | restricted by domain `[('payment_type', '=', 'outbound'), ('journal_id', '!=', False), ('journal_id.active', '=', True)]`; must belong to the same company |
| `hr_presence_last_compute_date` | Human resources Presence Last Compute Date | date and time |  |  |
| `job_properties_definition` | Job Properties | properties definition |  |  |
| `social_twitter` | X Account | single line text |  |  |
| `social_facebook` | Facebook Account | single line text |  |  |
| `social_github` | GitHub Account | single line text |  |  |
| `social_linkedin` | LinkedIn Account | single line text |  |  |
| `social_youtube` | Youtube Account | single line text |  |  |
| `social_instagram` | Instagram Account | single line text |  |  |
| `social_tiktok` | TikTok Account | single line text |  |  |
| `social_discord` | Discord Account | single line text |  |  |
| `website_id` | Website | many to one | `website` | computed by rule `_compute_website_id` and stored |
| `project_time_mode_id` | Project Time Unit | many to one | `uom.uom` | default computed dynamically (_default_project_time_mode_id); Help: This will set the unit of measure used in projects and tasks. If you use the timesheet linked to projects, don't forget to setup the right unit of measure in your employees. |
| `timesheet_encode_uom_id` | Timesheet Encoding Unit | many to one | `uom.uom` | default computed dynamically (_default_timesheet_encode_uom_id) |
| `internal_project_id` | Internal Project | many to one | `project.project` | restricted by domain `[["is_template", "=", false]]`; Help: Default project value for timesheet generated from time off type. |
| `withholding_tax_base_account_id` | Withholding Tax Base | many to one | `account.account` | Help: This account will be set on withholding tax base lines. |
| `iap_enrich_auto_done` | Enrich Done | boolean |  |  |
| `point_of_sale_update_stock_quantities` | Update quantities in stock | selection |  | default `real`; Help: At the session closing: A picking is created for the entire session when it's closed  In real time: Each order sent to the server create its own picking |
| `point_of_sale_use_ticket_qr_code` | Self-service invoicing | boolean |  | default `True`; Help: Print information on the receipt to allow the customer to easily access the invoice anytime, from Odoo's portal. |
| `point_of_sale_ticket_unique_code` | Generate a code on ticket | boolean |  | Help: Add a 5-digit code on the receipt to allow the user to request the invoice for an order on the portal. |
| `point_of_sale_ticket_portal_url_display_mode` | Print | selection |  | required; default `qr_code_and_url`; Help: Choose how the URL to the portal will be print on the receipt. |
| `l10n_gcc_dual_language_invoice` | GCC Formatted Invoices | boolean |  |  |
| `l10n_gcc_country_is_gcc` | Localization Gcc Country Is Gcc | boolean |  | computed by rule `_compute_l10n_gcc_country_is_gcc` (not stored) |
| `l10n_ar_gross_income_number` | Gross Income Number | single line text |  | related through path `partner_id.l10n_ar_gross_income_number`; Help: This field is required in order to print the invoice report properly |
| `l10n_ar_gross_income_type` | Gross Income | selection |  | related through path `partner_id.l10n_ar_gross_income_type`; Help: This field is required in order to print the invoice report properly |
| `l10n_ar_afip_responsibility_type_id` | Localization Ar Afip Responsibility Type | many to one |  | related through path `partner_id.l10n_ar_afip_responsibility_type_id`; restricted by domain `[('code', 'in', [1, 4, 6])]` |
| `l10n_ar_company_requires_vat` | Company Requires Vat? | boolean |  | computed by rule `_compute_l10n_ar_company_requires_vat` (not stored) |
| `l10n_ar_afip_start_date` | Activities Start | date |  |  |
| `l10n_ar_tax_base_account_id` | Tax Base Account | many to one | `account.account` | Help: Account that will be set on lines created to represent the tax base amounts. |
| `has_position_column` | Show Position Column in Reports | boolean |  |  |
| `l10n_au_is_gst_registered` | Australia goods and services tax registered | boolean |  | Help: Enable if your company is registered for GST. |
| `l10n_au_trading_name` | Trading Name | single line text |  | Help: The trading name of the company. |
| `l10n_br_ie_code` | IE | single line text |  | related through path `partner_id.l10n_br_ie_code` |
| `l10n_br_im_code` | IM | single line text |  | related through path `partner_id.l10n_br_im_code` |
| `l10n_br_nire_code` | NIRE | single line text |  | Help: State Commercial Identification Number. Should contain 11 digits. |
| `l10n_ca_pst` | PST Number | single line text |  | related through path `partner_id.l10n_ca_pst` |
| `l10n_cl_activity_description` | Company Activity Description | single line text |  | related through path `partner_id.l10n_cl_activity_description` |
| `trade_registry` | Trade Registry | single line text |  | extended by packages `l10n_sk` |
| `l10n_cz_tax_office_id` | Tax Office (CZ) | many to one | `l10n_cz.tax_office` |  |
| `l10n_de_stnr` | St.-Nr. | single line text |  | changes are tracked in the message thread; Help: Tax number. Scheme: ??FF0BBBUUUUP, e.g.: 2893081508152 https://de.wikipedia.org/wiki/Steuernummer |
| `l10n_de_widnr` | W-IdNr. | single line text |  | changes are tracked in the message thread; Help: Business identification number. |
| `po_lock` | Purchase Order Modification | selection |  | default `edit`; Help: Purchase Order Modification used when you want to purchase order editable after confirm |
| `po_double_validation` | Levels of Approvals | selection |  | default `one_step`; Help: Provide a double validation mechanism for purchases |
| `po_double_validation_amount` | Double validation amount | monetary |  | default `5000`; Help: Minimum amount for which a double validation is required |
| `nemhandel_contact_email` | Nemhandel Contact email | single line text |  | computed by rule `_compute_nemhandel_contact_email` and stored; Help: Primary contact email for Nemhandel-related communication |
| `nemhandel_phone_number` | Nemhandel Phone number (for validation) | single line text |  | computed by rule `_compute_nemhandel_phone_number` and stored; Help: You will receive a verification code to this phone number |
| `l10n_dk_nemhandel_proxy_state` | Nemhandel status | selection |  | required; default `not_registered` |
| `nemhandel_identifier_type` | Nemhandel Identifier Type | selection |  | related through path `partner_id.nemhandel_identifier_type` |
| `nemhandel_identifier_value` | Nemhandel Identifier Value | single line text |  | related through path `partner_id.nemhandel_identifier_value` |
| `nemhandel_purchase_journal_id` | Nemhandel Purchase Journal | many to one | `account.journal` | computed by rule `_compute_nemhandel_purchase_journal_id` and stored; restricted by domain `[["type", "=", "purchase"]]` |
| `nemhandel_edi_user` | Nemhandel Electronic data interchange User | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_nemhandel_edi_user` (not stored) |
| `l10n_ee_rounding_difference_loss_account_id` | Localization Ee Rounding Difference Loss Account | many to one | `account.account` | must belong to the same company |
| `l10n_ee_rounding_difference_profit_account_id` | Localization Ee Rounding Difference Profit Account | many to one | `account.account` | must belong to the same company |
| `l10n_eg_client_identifier` | ETA Client identifier | single line text |  | visible only to groups `base.group_erp_manager` |
| `l10n_eg_client_secret` | ETA Secret | single line text |  | visible only to groups `base.group_erp_manager` |
| `l10n_eg_production_env` | In Production Environment | boolean |  |  |
| `l10n_eg_invoicing_threshold` | Invoicing Threshold | float |  | default ; Help: Threshold at which you are required to give the VAT number of the customer. |
| `l10n_es_simplified_invoice_limit` | Simplified Invoice limit amount | float |  | default `400`; Help: Over this amount is not legally possible to create a simplified invoice |
| `l10n_es_edi_facturae_residence_type` | Facturae electronic data interchange Residency Type Code | single line text |  | related through path `partner_id.l10n_es_edi_facturae_residence_type` |
| `l10n_es_edi_facturae_certificate_ids` | Facturae electronic data interchange signing certificate | one to many | `certificate.certificate` | restricted by domain `[["scope", "=", "facturae"]]`; inverse field `company_id` |
| `l10n_es_sii_certificate_id` | Certificate (immediate supply of information) | many to one | `certificate.certificate` | computed by rule `_compute_l10n_es_sii_certificate` and stored |
| `l10n_es_sii_certificate_ids` | Localization Es Immediate supply of information Certificate | one to many | `certificate.certificate` | restricted by domain `[["scope", "=", "sii"]]`; inverse field `company_id` |
| `l10n_es_sii_tax_agency` | Tax Agency for immediate supply of information | selection |  | default  |
| `l10n_es_sii_test_env` | immediate supply of information Test Mode | boolean |  | default `True`; Help: Use the test environment for SII |
| `l10n_es_tbai_certificate_id` | Certificate (TicketBAI) | many to one | `certificate.certificate` | computed by rule `_compute_l10n_es_tbai_certificate` and stored |
| `l10n_es_tbai_certificate_ids` | Localization Es electronic invoicing (Basque) Certificate | one to many | `certificate.certificate` | restricted by domain `[["scope", "=", "tbai"]]`; inverse field `company_id` |
| `l10n_es_tbai_tax_agency` | Tax Agency for electronic invoicing (Basque) | selection |  |  |
| `l10n_es_tbai_license_html` | TicketBAI license | rich text |  | computed by rule `_compute_l10n_es_tbai_license_html` (not stored) |
| `l10n_es_tbai_chain_sequence_id` | TicketBai account.move chain sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `l10n_es_tbai_test_env` | electronic invoicing (Basque) Test Mode | boolean |  | default `True`; Help: Use the test environment for TicketBAI |
| `l10n_es_tbai_is_enabled` | Localization Es electronic invoicing (Basque) Is Enabled | boolean |  | computed by rule `_compute_l10n_es_tbai_is_enabled` (not stored) |
| `l10n_es_edi_verifactu_certificate_ids` | Veri*Factu Certificates | one to many | `certificate.certificate` | inverse field `company_id` |
| `l10n_es_edi_verifactu_required` | Enable Veri*Factu | boolean |  | not copied on duplication |
| `l10n_es_edi_verifactu_test_environment` | Veri*Factu Test Environment | boolean |  | default `True`; not copied on duplication |
| `l10n_es_edi_verifactu_chain_sequence_id` | Veri*Factu Document Chain Sequence | many to one | `ir.sequence` | read only; not copied on duplication |
| `l10n_es_edi_verifactu_next_batch_time` | Veri*Factu Next Batch Time | date and time |  | read only; not copied on duplication; Help: The Datetime at which the next submission to the AEAT can be made. |
| `l10n_es_edi_verifactu_special_vat_regime` | Veri*Factu value-added tax Regime | selection |  | Help: Leave empty for the normal regimen. |
| `l10n_fr_closing_sequence_id` | Sequence to use to build sale closings | many to one | `ir.sequence` | read only |
| `ape` | APE | single line text |  |  |
| `is_france_country` | Is Part of DOM-TOM | boolean |  | computed by rule `_compute_is_france_country` (not stored) |
| `l10n_fr_rounding_difference_loss_account_id` | Localization Fr Rounding Difference Loss Account | many to one | `account.account` | must belong to the same company |
| `l10n_fr_rounding_difference_profit_account_id` | Localization Fr Rounding Difference Profit Account | many to one | `account.account` | must belong to the same company |
| `l10n_fr_reference_leave_type` | Company Paid Time Off Type | many to one | `hr.leave.type` |  |
| `l10n_fr_pdp_send_to_ppf` | Send to PPF | boolean |  | default `True`; visible only to groups `base.group_user`; Help: Activate Flux 1 regulatory data, Flux 6 mandatory statuses and Flux 10 e-reporting generation for this company. |
| `l10n_fr_pdp_pilot_phase` | E-Invoicing Pilot Phase | boolean |  | visible only to groups `base.group_user`; Help: Participate in the Pilot Phase of the French E-Invoicing. This way you are able to test it before it becomes mandatory. |
| `l10n_fr_pdp_annuaire_start_date` | Annuaire Start Date | date |  | visible only to groups `base.group_user`; Help: The date on which the company is registered on the annuaire for the French e-invoicing. |
| `l10n_fr_pdp_registered` | Approved Platform Registerd | boolean |  | computed by rule `_compute_l10n_fr_pdp_registered` (not stored); visible only to groups `base.group_user` |
| `pdp_identifier` | Pdp Identifier | single line text |  | computed by rule `_compute_pdp_identifier` (not stored); writable through an inverse rule; visible only to groups `base.group_user` |
| `l10n_fr_pdp_periodicity` | Flow 10 Report Periodicity | selection |  | required; default `normal_monthly`; visible only to groups `base.group_user`; Help: Legal reporting period for transaction and payments flows according to the TVA regime table.         Real Monthly Normal Regime : transactions reported by decade, payments reported monthly         Real Normal Quarterly Regime : transactions reported monthly, payments reported monthly         Simplified VAT Regime (Monthly) : transactions reported monthly, payments reported monthly         Franchised VAT Regime (Bimonthly) : transactions reported bimonthly, payments reported bimonthly |
| `l10n_fr_f10_enable_reporting` | Enable Flux 10 Reporting | boolean |  | read only; computed by rule `_compute_l10n_fr_f10_enable_reporting` and stored; visible only to groups `base.group_user` |
| `l10n_fr_pdp_flow_10_start_date` | Localization Fr Pdp Flow 10 Start Date | date |  | computed by rule `_compute_l10n_fr_pdp_flow_10_start_date` (not stored); visible only to groups `base.group_user` |
| `pdp_kyc_status` | Pdp Know your customer Status | selection |  | visible only to groups `base.group_user` |
| `pdp_authentication_uuid` | Authentication in-app purchase UUID | single line text |  | visible only to groups `account.group_account_invoice` |
| `l10n_fr_pos_cert_sequence_id` | Localization Fr Point of sale Cert Sequence | many to one | `ir.sequence` |  |
| `l10n_gr_edi_aade_id` | AADE User identifier | single line text |  |  |
| `l10n_gr_edi_aade_key` | AADE Subscription Key | single line text |  |  |
| `l10n_gr_edi_branch_number` | Localization Gr Electronic data interchange Branch Number | integer |  | related through path `partner_id.l10n_gr_edi_branch_number` |
| `l10n_gr_edi_test_env` | Greece Test Environment | boolean |  | default `True`; Help: Enable test environments with credentials obtained from https://mydata-dev-register.azurewebsites.net/ |
| `l10n_hr_mer_username` | MojEracun username | single line text |  | visible only to groups `account.group_account_manager` |
| `l10n_hr_mer_password` | MojEracun password | single line text |  | visible only to groups `account.group_account_manager` |
| `l10n_hr_mer_company_ident` | MojEracun CompanyId | single line text |  | visible only to groups `account.group_account_manager` |
| `l10n_hr_mer_software_ident` | MojEracun SoftwareId | single line text |  | default `Saodoo-001`; Help: Default SoftwareID for Odoo is 'Saodoo-001' |
| `l10n_hr_mer_connection_state` | MojEracun connection status | selection |  | required; computed by rule `_compute_l10n_hr_mojeracun_state` and stored; default `inactive` |
| `l10n_hr_mer_connection_mode` | MojEracun Operating mode | selection |  | default `test` |
| `l10n_hr_mer_purchase_journal_id` | eracun Purchase Journal | many to one | `account.journal` | computed by rule `_compute_l10n_hr_mer_purchase_journal_id` and stored; restricted by domain `[["type", "=", "purchase"]]` |
| `l10n_hu_group_vat` | Localization Hu Group Value-added tax | single line text |  | related through path `partner_id.l10n_hu_group_vat` |
| `l10n_hu_tax_regime` | NAV Tax Regime | selection |  |  |
| `l10n_hu_edi_server_mode` | Server Mode | selection |  | Help: - Production: Sends invoices to the NAV's production system.             - Test: Sends invoices to the NAV's test system.             - Demo: Mocks the NAV system (does not require credentials). |
| `l10n_hu_edi_username` | NAV Username | single line text |  | visible only to groups `base.group_system` |
| `l10n_hu_edi_password` | NAV Password | single line text |  | visible only to groups `base.group_system` |
| `l10n_hu_edi_signature_key` | NAV Signature Key | single line text |  | visible only to groups `base.group_system` |
| `l10n_hu_edi_replacement_key` | NAV Replacement Key | single line text |  | visible only to groups `base.group_system` |
| `l10n_hu_edi_last_transaction_recovery` | Last transaction recovery (in production mode) | date and time |  | default computed dynamically (lambda self: fields.Datetime.now()) |
| `l10n_in_upi_id` | UPI Id | single line text |  |  |
| `l10n_in_hsn_code_digit` | harmonized system nomenclature Code Digit | selection |  | computed by rule `_compute_l10n_in_hsn_code_digit` and stored |
| `l10n_in_edi_production_env` | Indian Production Environment | boolean |  | default `True`; visible only to groups `base.group_system`; Help: Enable the use of production credentials |
| `l10n_in_pan_entity_id` | permanent account number | many to one |  | related through path `partner_id.l10n_in_pan_entity_id` and stored; Help: PAN enables the department to link all transactions of the person with the department. These transactions include taxpayments, TDS/TCS credits, returns of income/wealth/gift/FBT,specified transactions, correspondence, and so on. Thus, PAN acts as an identifier for the person with the tax department. |
| `l10n_in_pan_type` | permanent account number Type | selection |  | related through path `l10n_in_pan_entity_id.type` |
| `l10n_in_tan` | TAN | single line text |  | related through path `partner_id.l10n_in_tan` |
| `l10n_in_gst_state_warning` | Localization In Goods and services tax State Warning | single line text |  | related through path `partner_id.l10n_in_gst_state_warning` |
| `l10n_in_tds_feature` | tax deducted at source | boolean |  | computed by rule `_compute_l10n_in_parent_based_features` and stored; writable through an inverse rule; recursive dependency |
| `l10n_in_tcs_feature` | tax collected at source | boolean |  | computed by rule `_compute_l10n_in_parent_based_features` and stored; writable through an inverse rule; recursive dependency |
| `l10n_in_withholding_account_id` | tax deducted at source Account | many to one | `account.account` | must belong to the same company |
| `l10n_in_withholding_journal_id` | tax deducted at source Journal | many to one | `account.journal` | must belong to the same company |
| `l10n_in_is_gst_registered` | Registered Under goods and services tax | boolean |  | computed by rule `_compute_l10n_in_parent_based_features` and stored; writable through an inverse rule; recursive dependency |
| `l10n_in_gstin_status_feature` | Check goods and services tax Number Status | boolean |  |  |
| `l10n_in_edi_feature` | Indian E-Invoicing | boolean |  |  |
| `l10n_in_edi_username` | E-invoice (IN) Username | single line text |  | visible only to groups `base.group_system` |
| `l10n_in_edi_password` | E-invoice (IN) Password | single line text |  | visible only to groups `base.group_system` |
| `l10n_in_edi_token` | E-invoice (IN) Token | single line text |  | visible only to groups `base.group_system` |
| `l10n_in_edi_token_validity` | E-invoice (IN) Valid Until | date and time |  | visible only to groups `base.group_system` |
| `l10n_in_ewaybill_username` | E-Waybill Username | single line text |  | visible only to groups `base.group_system` |
| `l10n_in_ewaybill_password` | E-Waybill Password | single line text |  | visible only to groups `base.group_system` |
| `l10n_in_ewaybill_auth_validity` | E-Waybill Valid Until | date and time |  | visible only to groups `base.group_system` |
| `l10n_in_ewaybill_feature` | E-Waybill | boolean |  |  |
| `days_to_purchase` | Days to Purchase | float |  | Help: Days needed to confirm a PO, define when a PO should be validated |
| `l10n_it_codice_fiscale` | Codice Fiscale | single line text |  | related through path `partner_id.l10n_it_codice_fiscale` and stored; maximum length 16; Help: Fiscal code of your company |
| `l10n_it_tax_system` | Tax System | selection |  | Help: Please select the Tax system to which you are subjected. |
| `l10n_it_edi_proxy_user_id` | Localization It Electronic data interchange Proxy User | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_l10n_it_edi_proxy_user_id` (not stored) |
| `l10n_it_edi_register` | Localization It Electronic data interchange Register | boolean |  | default  |
| `l10n_it_edi_purchase_journal_id` | Italian Default Purchase Journal | many to one | `account.journal` | computed by rule `_compute_l10n_it_edi_purchase_journal_id` and stored; restricted by domain `[["type", "=", "purchase"]]` |
| `l10n_it_has_eco_index` | Localization It Has Eco Index | boolean |  | Help: The seller/provider is a company listed on the register of companies and as        such must also indicate the registration data on all documents (art. 2250, Italian        Civil Code) |
| `l10n_it_eco_index_office` | Province of the register-of-companies office | many to one | `res.country.state` | restricted by domain `[('country_id','=','IT')]` |
| `l10n_it_eco_index_number` | Number in register of companies | single line text |  | maximum length 20; Help: This field must contain the number under which the        seller/provider is listed on the register of companies. |
| `l10n_it_eco_index_share_capital` | Share capital actually paid up | float |  | Help: Mandatory if the seller/provider is a company with share        capital (SpA, SApA, Srl), this field must contain the amount        of share capital actually paid up as resulting from the last        financial statement |
| `l10n_it_eco_index_sole_shareholder` | Shareholder | selection |  |  |
| `l10n_it_eco_index_liquidation_state` | Liquidation state | selection |  |  |
| `l10n_it_has_tax_representative` | Localization It Has Tax Representative | boolean |  | Help: The seller/provider is a non-resident subject which        carries out transactions in Italy with relevance for VAT        purposes and which takes avail of a tax representative in        Italy |
| `l10n_it_tax_representative_partner_id` | Tax representative partner | many to one | `res.partner` |  |
| `l10n_it_edi_doi_tax_id` | Declaration of Intent Tax | many to one | `account.tax` |  |
| `l10n_it_edi_doi_fiscal_position_id` | Declaration of Intent Fiscal Position | many to one | `account.fiscal.position` |  |
| `l10n_jo_edi_sequence_income_source` | JoFotara Sequence of Income Source | single line text |  |  |
| `l10n_jo_edi_secret_key` | JoFotara Secret Key | single line text |  | visible only to groups `base.group_system` |
| `l10n_jo_edi_client_identifier` | JoFotara Client identifier | single line text |  | visible only to groups `base.group_system` |
| `l10n_jo_edi_taxpayer_type` | JoFotara Taxpayer Type | selection |  | default `sales` |
| `l10n_jo_edi_demo_mode` | JoFotara Demo Mode | boolean |  |  |
| `l10n_jo_edi_pos_enabled` | Localization Jo Electronic data interchange Point of sale Enabled | boolean |  |  |
| `l10n_jo_edi_pos_testing_mode` | Localization Jo Electronic data interchange Point of sale Testing Mode | boolean |  |  |
| `l10n_ke_oscu_is_active` | Is OSCU active? | boolean |  | computed by rule `_compute_l10n_ke_oscu_is_active` (not stored); Help: Whether this company is set up for OSCU flows. |
| `l10n_ke_cu_proxy_address` | Fiscal Device Proxy Address | single line text |  | default `http://localhost:8069`; Help: The address of the proxy server for the fiscal device. |
| `l10n_lk_vat_registered` | Sri Lanka: value-added tax Registered | boolean |  | related through path `partner_id.l10n_lk_vat_registered`; Help: Indicates if this company is registered for VAT in Sri Lanka. This defaults invoice printout to this partner to tax invoice for taxable supplies. |
| `l10n_mx_income_return_discount_account_id` | Income account for returns and discounts | many to one | `account.account` |  |
| `l10n_mx_income_re_invoicing_account_id` | Income account for re-invoicing | many to one | `account.account` |  |
| `sst_registration_number` | Sst Registration Number | single line text |  | related through path `partner_id.sst_registration_number` |
| `ttx_registration_number` | Ttx Registration Number | single line text |  | related through path `partner_id.ttx_registration_number` |
| `l10n_my_edi_proxy_user_id` | Localization My Electronic data interchange Proxy User | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_l10n_my_edi_proxy_user_id` (not stored) |
| `l10n_my_identification_type` | Localization My Identification Type | selection |  | related through path `partner_id.l10n_my_identification_type` |
| `l10n_my_identification_number` | Localization My Identification Number | single line text |  | related through path `partner_id.l10n_my_identification_number` |
| `l10n_my_identification_number_placeholder` | Localization My Identification Number Placeholder | single line text |  | computed by rule `_compute_l10n_my_identification_number_placeholder` (not stored) |
| `l10n_my_edi_industrial_classification` | Localization My Electronic data interchange Industrial Classification | many to one |  | related through path `partner_id.l10n_my_edi_industrial_classification` |
| `l10n_my_edi_mode` | Localization My Electronic data interchange Mode | selection |  | default `test` |
| `l10n_my_edi_default_import_journal_id` | Default import journal | many to one | `account.journal` | restricted by domain `[('type', '=', 'purchase')]`; Help: The journal on which invoices imported from MyInvois will be booked. Leave empty to use the default purchase journal. |
| `l10n_nl_rounding_difference_loss_account_id` | Localization Nl Rounding Difference Loss Account | many to one | `account.account` | must belong to the same company |
| `l10n_nl_rounding_difference_profit_account_id` | Localization Nl Rounding Difference Profit Account | many to one | `account.account` | must belong to the same company |
| `l10n_no_bronnoysund_number` | Localization No Bronnoysund Number | single line text |  | related through path `partner_id.l10n_no_bronnoysund_number` |
| `branch_code` | Company Branch Code | single line text |  | related through path `partner_id.branch_code` |
| `l10n_ph_rdo` | Localization Ph Rdo | single line text |  | related through path `partner_id.l10n_ph_rdo` |
| `l10n_pl_reports_tax_office_id` | Tax Office | many to one | `l10n_pl.l10n_pl_tax_office` | visible only to groups `account.group_account_user` |
| `l10n_pl_edi_register` | KSeF Integration Enabled | boolean |  | computed by rule `_compute_l10n_pl_edi_register` (not stored) |
| `l10n_pl_edi_certificate` | KSeF Certificate | many to one | `certificate.certificate` | visible only to groups `base.group_system` |
| `l10n_pl_edi_access_token` | KSeF Token | single line text |  | read only; not copied on duplication; visible only to groups `base.group_system` |
| `l10n_pl_edi_refresh_token` | KSeF Token Expiration | single line text |  | read only; not copied on duplication; visible only to groups `base.group_system` |
| `l10n_pl_edi_session_id` | Reference number | single line text |  | read only; visible only to groups `base.group_system` |
| `l10n_pl_edi_session_key` | Session key | binary |  | read only; visible only to groups `base.group_system` |
| `l10n_pl_edi_session_iv` | Session iv | binary |  | read only; visible only to groups `base.group_system` |
| `l10n_ro_edi_client_id` | eFactura Client identifier | single line text |  |  |
| `l10n_ro_edi_client_secret` | Client Secret | single line text |  |  |
| `l10n_ro_edi_access_token` | Access Token | single line text |  |  |
| `l10n_ro_edi_refresh_token` | Refresh Token | single line text |  |  |
| `l10n_ro_edi_access_expiry_date` | Access Token Expiry Date | date |  |  |
| `l10n_ro_edi_refresh_expiry_date` | Refresh Token Expiry Date | date |  |  |
| `l10n_ro_edi_callback_url` | Localization Ro Electronic data interchange Callback Uniform resource locator | single line text |  | computed by rule `_compute_l10n_ro_edi_callback_url` (not stored) |
| `l10n_ro_edi_test_env` | Use Test Environment | boolean |  | default `True` |
| `l10n_ro_edi_anaf_imported_inv_journal_id` | Select journal for SPV imported bills | many to one | `account.journal` | computed by rule `_compute_l10n_ro_edi_anaf_imported_inv_journal` and stored; restricted by domain `[('type', '=', 'purchase')]` |
| `l10n_rs_edi_api_key` | eFaktura application programming interface Key | single line text |  |  |
| `l10n_rs_edi_demo_env` | Use Demo Environment | boolean |  | default `True` |
| `l10n_sa_private_key_id` | ZATCA Private key | many to one | `certificate.key` | not copied on duplication; restricted by domain `[["public", "=", false]]`; Help: The private key used to generate the CSR and obtain certificates |
| `l10n_sa_api_mode` | Localization Sa Application programming interface Mode | selection |  | required; default `sandbox`; not copied on duplication; Help: Specifies which API the system should use |
| `l10n_sa_edi_building_number` | Localization Sa Electronic data interchange Building Number | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `l10n_sa_edi_plot_identification` | Localization Sa Electronic data interchange Plot Identification | single line text |  | computed by rule `_compute_address` (not stored); writable through an inverse rule |
| `l10n_sa_edi_additional_identification_scheme` | Localization Sa Electronic data interchange Additional Identification Scheme | selection |  | related through path `partner_id.l10n_sa_edi_additional_identification_scheme` |
| `l10n_sa_edi_additional_identification_number` | Localization Sa Electronic data interchange Additional Identification Number | single line text |  | related through path `partner_id.l10n_sa_edi_additional_identification_number` |
| `l10n_sa_edi_is_production` | Is Production | boolean |  | not copied on duplication |
| `org_number` | Org Number | single line text |  | computed by rule `_compute_org_number` (not stored) |
| `l10n_sg_unique_entity_number` | UEN | single line text |  | related through path `partner_id.l10n_sg_unique_entity_number` |
| `income_tax_id` | Income Tax identifier | single line text |  |  |
| `l10n_tr_nilvera_api_key` | Nilvera application programming interface key | single line text |  | visible only to groups `base.group_system` |
| `l10n_tr_nilvera_use_test_env` | Use testing environment | boolean |  | required; default `True` |
| `l10n_tr_nilvera_purchase_journal_id` | Nilvera Purchase Journal | many to one | `account.journal` | computed by rule `_compute_l10n_tr_nilvera_purchase_journal_id` and stored; writable through an inverse rule; restricted by domain `[["type", "=", "purchase"]]` |
| `l10n_tr_tax_office_id` | Localization Tr Tax Office | many to one |  | related through path `partner_id.l10n_tr_tax_office_id` |
| `l10n_tr_nilvera_export_alias` | Nilvera Export Alias | single line text |  | default `urn:mail:ihracatpk@gtb.gov.tr`; visible only to groups `base.group_system` |
| `l10n_tw_edi_ecpay_staging_mode` | Staging mode | boolean |  | visible only to groups `base.group_system` |
| `l10n_tw_edi_ecpay_merchant_id` | MerchantID | single line text |  | visible only to groups `base.group_system` |
| `l10n_tw_edi_ecpay_hashkey` | Hashkey | single line text |  | visible only to groups `base.group_system` |
| `l10n_tw_edi_ecpay_hashIV` | HashIV | single line text |  | visible only to groups `base.group_system` |
| `l10n_vn_edi_username` | SInvoice Username | single line text |  | visible only to groups `base.group_system` |
| `l10n_vn_edi_password` | Sinvoice Password | single line text |  | visible only to groups `base.group_system` |
| `l10n_vn_edi_token` | Sinvoice Access Token | single line text |  | read only; visible only to groups `base.group_system` |
| `l10n_vn_edi_token_expiry` | Sinvoice Access Token Expiration Date | date and time |  | read only; visible only to groups `base.group_system` |
| `l10n_vn_pos_default_symbol` | Default PoS Symbol | many to one | `l10n_vn_edi_viettel.sinvoice.symbol` |  |
| `lunch_minimum_threshold` | Lunch Minimum Threshold | float |  |  |
| `lunch_notify_message` | Lunch Notify Message | rich text |  | default `Your lunch has been delivered. Enjoy your meal!`; translatable |
| `lc_journal_id` | Lc Journal | many to one | `account.journal` |  |
| `subcontracting_location_id` | Subcontracting Location | many to one | `stock.location` |  |
| `dropship_subcontractor_pick_type_id` | Dropship Subcontractor Pick Type | many to one | `stock.picking.type` |  |
| `partnership_label` | Partnership Label | single line text |  | default computed dynamically (lambda s: s.env._('Members')); translatable; Help: Name used to refer to affiliates: partners, members, alumnis, etc... |
| `leave_timesheet_task_id` | Time Off Task | many to one | `project.task` | restricted by domain `[('project_id', '=', internal_project_id)]` |
| `gelato_api_key` | Gelato application programming interface Key | single line text |  | visible only to groups `base.group_system` |
| `gelato_webhook_secret` | Gelato Webhook Secret | single line text |  | visible only to groups `base.group_system` |
| `sms_provider` | text message Provider | selection |  | default `iap` |
| `sms_twilio_account_sid` | Account SID | single line text |  | visible only to groups `base.group_system` |
| `sms_twilio_auth_token` | Auth Token | single line text |  | visible only to groups `base.group_system` |
| `sms_twilio_number_ids` | Numbers | one to many | `sms.twilio.number` | inverse field `company_id` |
| `snailmail_color` | Snailmail Color | boolean |  | default `True` |
| `snailmail_cover` | Add a Cover Page | boolean |  | default  |
| `snailmail_duplex` | Both sides | boolean |  | default  |
| `stock_sms_confirmation_template_id` | text message Template | many to one | `sms.template` | default computed dynamically (_default_confirmation_sms_picking_template); restricted by domain `[('model', '=', 'stock.picking')]`; Help: SMS sent to the customer once the order is delivered. |
| `has_received_warning_stock_sms` | Has Received Warning Stock Text message | boolean |  |  |

## Selection values

### `font` (Font)

| Value | Label |
|---|---|
| `Lato` | Lato |
| `Roboto` | Roboto |
| `Open_Sans` | Open Sans |
| `Montserrat` | Montserrat |
| `Oswald` | Oswald |
| `Raleway` | Raleway |
| `Tajawal` | Tajawal |
| `Fira_Mono` | Fira Mono |

### `layout_background` (Layout Background)

| Value | Label |
|---|---|
| `Blank` | Blank |
| `Demo logo` | Demo logo |
| `Custom` | Custom |

### `tax_calculation_rounding_method` (Tax Calculation Rounding Method)

| Value | Label |
|---|---|
| `round_globally` | Round per Tax |
| `round_per_line` | Round per Line |

### `terms_type` (Terms & Conditions format)

| Value | Label |
|---|---|
| `plain` | Add a Note |
| `html` | Add a link to a Web Page |

### `quick_edit_mode` (Quick encoding)

| Value | Label |
|---|---|
| `out_invoices` | Customer Invoices |
| `in_invoices` | Vendor Bills |
| `out_and_in_invoices` | Customer Invoices and Vendor Bills |

### `account_price_include` (Default Sales Price Include)

| Value | Label |
|---|---|
| `tax_included` | Tax Included |
| `tax_excluded` | Tax Excluded |

### `account_check_printing_layout` (Check Layout)

| Value | Label |
|---|---|
| `disabled` | None |

### `account_peppol_proxy_state` (PEPPOL status)

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `sender` | Can send but not receive |
| `smp_registration` | Can send, pending registration to receive |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

### `sale_onboarding_payment_method` (Sale onboarding selected payment method)

| Value | Label |
|---|---|
| `digital_signature` | Sign online |
| `paypal` | PayPal |
| `stripe` | Stripe |
| `other` | Pay with another payment provider |
| `manual` | Manual Payment |

### `annual_inventory_month` (Annual Inventory Month)

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

### `stock_confirmation_type` (Stock Confirmation Type)

| Value | Label |
|---|---|
| `sms` | SMS |

### `inventory_period` (Inventory Period)

| Value | Label |
|---|---|
| `manual` | Manual |
| `daily` | Daily |
| `monthly` | Monthly |

### `inventory_valuation` (Valuation)

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

### `cost_method` (Cost Method)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

### `attendance_kiosk_mode` (Attendance Mode)

| Value | Label |
|---|---|
| `barcode` | Barcode / RFID |
| `barcode_manual` | Barcode / RFID and Manual Selection |
| `manual` | Manual Selection |

### `attendance_barcode_source` (Barcode Source)

| Value | Label |
|---|---|
| `scanner` | Scanner |
| `front` | Front Camera |
| `back` | Back Camera |

### `attendance_overtime_validation` (Extra Hours Validation)

| Value | Label |
|---|---|
| `no_validation` | Automatically Approved |
| `by_manager` | Approved by Manager |

### `point_of_sale_update_stock_quantities` (Update quantities in stock)

| Value | Label |
|---|---|
| `closing` | At the session closing |
| `real` | In real time |

### `point_of_sale_ticket_portal_url_display_mode` (Print)

| Value | Label |
|---|---|
| `qr_code` | QR code |
| `url` | URL |
| `qr_code_and_url` | QR code + URL |

### `po_lock` (Purchase Order Modification)

| Value | Label |
|---|---|
| `edit` | Allow to edit purchase orders |
| `lock` | Confirmed purchase orders are not editable |

### `po_double_validation` (Levels of Approvals)

| Value | Label |
|---|---|
| `one_step` | Confirm purchase orders in one step |
| `two_step` | Get 2 levels of approvals to confirm a purchase order |

### `l10n_dk_nemhandel_proxy_state` (Nemhandel status)

| Value | Label |
|---|---|
| `not_registered` | Not registered |
| `in_verification` | In verification |
| `receiver` | Can send and receive |
| `rejected` | Rejected |

### `l10n_es_sii_tax_agency` (Tax Agency for immediate supply of information)

| Value | Label |
|---|---|
| `aeat` | Agencia Tributaria española |
| `gipuzkoa` | Hacienda Foral de Gipuzkoa |
| `bizkaia` | Hacienda Foral de Bizkaia |
| `navarra` | Hacienda Foral de Navarra |

### `l10n_es_tbai_tax_agency` (Tax Agency for electronic invoicing (Basque))

| Value | Label |
|---|---|
| `araba` | Hacienda Foral de Araba |
| `bizkaia` | Hacienda Foral de Bizkaia |
| `gipuzkoa` | Hacienda Foral de Gipuzkoa |

### `l10n_es_edi_verifactu_special_vat_regime` (Veri*Factu value-added tax Regime)

| Value | Label |
|---|---|
| `simplified` | Simplified Regime |
| `reagyp` | REAGYP (Special Regime for Agriculture, Livestock and Fisheries) |
| `recargo` | Recargo de Equivalencia |

### `l10n_fr_pdp_periodicity` (Flow 10 Report Periodicity)

| Value | Label |
|---|---|
| `normal_monthly` | Real Monthly Normal Regime |
| `normal_quarterly` | Real Normal Quarterly Regime |
| `simplified_monthly` | Simplified VAT Regime (Monthly) |
| `simplified_bimonthly` | Franchised VAT Regime (Bimonthly) |

### `pdp_kyc_status` (Pdp Know your customer Status)

| Value | Label |
|---|---|
| `processing` | Processing |
| `success` | Success |
| `fail` | Fail |

### `l10n_hr_mer_connection_state` (MojEracun connection status)

| Value | Label |
|---|---|
| `inactive` | Inactive |
| `active` | Active |

### `l10n_hr_mer_connection_mode` (MojEracun Operating mode)

| Value | Label |
|---|---|
| `prod` | Production |
| `test` | Test |
| `demo` | Demo |

### `l10n_hu_tax_regime` (NAV Tax Regime)

| Value | Label |
|---|---|
| `ie` | Individual Exemption |
| `ca` | Cash Accounting |
| `sb` | Small Business |

### `l10n_hu_edi_server_mode` (Server Mode)

| Value | Label |
|---|---|
| `production` | Production |
| `test` | Test |
| `demo` | Demo |

### `l10n_in_hsn_code_digit` (harmonized system nomenclature Code Digit)

| Value | Label |
|---|---|
| `4` | 4 Digits (turnover < 5 CR.) |
| `6` | 6 Digits (turnover > 5 CR.) |
| `8` | 8 Digits |

### `l10n_it_eco_index_sole_shareholder` (Shareholder)

| Value | Label |
|---|---|
| `NO` | Not a limited liability company |
| `SU` | Socio unico |
| `SM` | Più soci |

### `l10n_it_eco_index_liquidation_state` (Liquidation state)

| Value | Label |
|---|---|
| `LS` | The company is in a state of liquidation |
| `LN` | The company is not in a state of liquidation |

### `l10n_jo_edi_taxpayer_type` (JoFotara Taxpayer Type)

| Value | Label |
|---|---|
| `income` | Unregistered in the sales tax |
| `sales` | Registered in the sales tax |
| `special` | Registered in the special sales tax |

### `l10n_my_edi_mode` (Localization My Electronic data interchange Mode)

| Value | Label |
|---|---|
| `test` | Pre-Production |
| `prod` | Production |

### `l10n_sa_api_mode` (Localization Sa Application programming interface Mode)

| Value | Label |
|---|---|
| `sandbox` | Sandbox |
| `preprod` | Simulation (Pre-Production) |
| `prod` | Production |

### `sms_provider` (text message Provider)

| Value | Label |
|---|---|
| `iap` | Send via Odoo |
| `twilio` | Send via Twilio |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`, `l10n_dk_nemhandel_proxy_state`, `pdp_kyc_status`, `l10n_hr_mer_connection_state`, `l10n_it_eco_index_liquidation_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | The company name must be unique! | `base` |
| `_check_quotation_validity_days` | Constraint | `CHECK(quotation_validity_days >= 0)` | You cannot set a negative number for the default quotation validity. Leave empty (or 0) to disable the automatic expiration of quotations. | `sale` |

## Operations (336)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `copy` | lifecycle override | self, default | `base` |  |  |
| `_get_logo` | preparation rule | self | `base` |  |  |
| `_default_currency_id` | preparation rule | self | `base` |  |  |
| `init` | lifecycle override | self | `base` |  |  |
| `_get_company_root_delegated_field_names` | preparation rule | self | `account`, `base` |  | Get the set of fields delegated to the root company.  Some fields need to be identical on all branches of the company. All fields listed by this function will be copied from the root company and appear as readonly in the form view. :rtype: set |
| `_get_company_address_field_names` | preparation rule | self | `base`, `l10n_sa_edi` |  | Return a list of fields coming from the address partner to match on company address fields. Fields are labeled same on both models. |
| `_get_company_address_update` | preparation rule | self, partner | `base` |  |  |
| `_compute_parent_ids` | computation | self | `base` | depends: `parent_path` |  |
| `_compute_address` | computation | self | `base` | depends: |  |
| `_inverse_street` | inverse computation | self | `base` |  |  |
| `_inverse_street2` | inverse computation | self | `base` |  |  |
| `_inverse_zip` | inverse computation | self | `base` |  |  |
| `_inverse_city` | inverse computation | self | `base` |  |  |
| `_inverse_state` | inverse computation | self | `base` |  |  |
| `_inverse_country` | inverse computation | self | `base` |  |  |
| `_compute_logo_web` | computation | self | `base` | depends: `partner_id.image_1920` |  |
| `_compute_uses_default_logo` | computation | self | `base` | depends: `partner_id.image_1920` |  |
| `_compute_color` | computation | self | `base` | depends: `root_id` |  |
| `_inverse_color` | inverse computation | self | `base` |  |  |
| `_onchange_state` | on change | self | `base` | onchange: `state_id` |  |
| `_onchange_country_id` | on change | self | `base` | onchange: `country_id` |  |
| `_onchange_parent_id` | on change | self | `base` | onchange: `parent_id` |  |
| `_compute_uninstalled_l10n_module_ids` | computation | self | `base` | depends: `country_id` |  |
| `install_l10n_modules` | operation | self | `account`, `base` |  |  |
| `_get_view` | lifecycle override | self, view_id, view_type, **options | `base`, `l10n_sg`, `partner_autocomplete` | model |  |
| `_search_display_name` | search rule | self, operator, value | `base` | model |  |
| `_compute_empty_company_details` | computation | self | `base` | depends: `company_details` |  |
| `create` | lifecycle override | self, vals_list | `account_peppol`, `account`, `base`, `hr_timesheet`, `l10n_fr_pos_cert`, `l10n_fr`, `l10n_in`, `l10n_latam_base`, `partner_autocomplete`, `payment`, `product`, `resource`, `stock`, `web` | model_create_multi | If exists, use specific vat identification.type for the country of the company |
| `cache_invalidation_fields` | operation | self | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  | Unlink the companies and clear the cache to make sure that _get_company_ids of res.users gets only existing company ids. |
| `write` | lifecycle override | self, vals | `account_peppol`, `account`, `base`, `hr_attendance`, `l10n_ar`, `l10n_de`, `l10n_fr_pos_cert`, `l10n_fr`, `l10n_in`, `l10n_sa_edi`, `product`, `web` |  | Delay the automatic creation of pricelists post-company update.  This makes sure that the pricelist(s) automatically created are created with the right currency. |
| `_check_active` | validation | self | `base`, `website` | constrains: `active` |  |
| `_check_root_delegated_fields` | validation | self | `base` | constrains: |  |
| `_get_main_company` | preparation rule | self | `base` | model |  |
| `__accessible_branches` | internal rule | self | `base` |  |  |
| `_accessible_branches` | internal rule | self | `base` |  |  |
| `_all_branches_selected` | internal rule | self | `base` |  | Return whether or all the branches of the companies in self are selected.  Is ``True`` if all the branches, and only those, are selected. Can be used when some actions only make sense for whole companies regardless of the branches. |
| `action_all_company_branches` | user action | self | `base` |  |  |
| `_get_public_user` | preparation rule | self | `base` |  |  |
| `_get_company_partner_ids` | preparation rule | self | `base` |  |  |
| `_get_zeep_cache__` | preparation rule | self | `base` |  | Return a cache bucket used by ``odoo.tools.zeep`` for XSDs/WSDLs. |
| `_get_zeep_client__` | preparation rule | self, url, *args, **kwargs | `base` |  | Return a Zeep Client which uses the ORM cache for XSDs/WSDLs. |
| `_get_asset_style_b64` | preparation rule | self | `web` |  |  |
| `_update_asset_style` | internal rule | self | `web` |  |  |
| `_default_alias_domain_id` | preparation rule | self | `mail` |  |  |
| `_compute_bounce` | computation | self | `mail` | depends: `alias_domain_id`, `name` |  |
| `_compute_catchall` | computation | self | `mail` | depends: `alias_domain_id`, `name` |  |
| `_compute_email_formatted` | computation | self | `mail` | depends: `partner_id`, `catchall_formatted` |  |
| `_activate_or_create_pricelists` | internal rule | self | `product` |  | Manage the default pricelists for needed companies. |
| `_get_default_pricelist_vals` | preparation rule | self | `product`, `website_sale` |  | Add values to the default pricelist at company creation or activation of the pricelist  Note: self.ensure_one()  :rtype: dict |
| `_init_data_resource_calendar` | internal rule | self | `resource` | model |  |
| `_create_resource_calendar` | internal rule | self | `resource` |  |  |
| `_prepare_resource_calendar_values` | preparation rule | self | `resource` |  |  |
| `get_next_batch_payment_communication` | operation | self | `account` |  | When in need of a batch payment communication reference (several invoices paid at the same time) use batch_payment_sequence_id to get it (eventually create it first): e.g BATCH/2024/00001 |
| `_check_audit_trail_restriction` | validation | self | `account` | constrains: `restrictive_audit_trail` |  |
| `_check_set_account_price_include` | validation | self | `account` | constrains: `account_price_include` |  |
| `_check_fiscalyear_last_day` | validation | self | `account` | constrains: `account_opening_move_id`, `fiscalyear_last_day`, `fiscalyear_last_month` |  |
| `_compute_force_restrictive_audit_trail` | computation | self | `account`, `l10n_de`, `l10n_in` | depends: `country_code`; depends: `country_code`, `root_id` |  |
| `_compute_domestic_fiscal_position_id` | computation | self | `account` | depends: `fiscal_position_ids`, `fiscal_position_ids.sequence`, `fiscal_position_ids.country_id`, `fiscal_position_ids.country_group_id` |  |
| `_compute_account_fiscal_country_group_codes` | computation | self | `account` | depends: `account_fiscal_country_id` |  |
| `_compute_multi_vat_foreign_country` | computation | self | `account` | depends: `fiscal_position_ids.foreign_vat` |  |
| `compute_account_tax_fiscal_country` | computation | self | `account` | depends: `country_id` |  |
| `_compute_account_enabled_tax_country_ids` | computation | self | `account` | depends: `account_fiscal_country_id` |  |
| `_compute_invoice_terms_html` | computation | self | `account` | depends: `terms_type` |  |
| `_compute_user_fiscalyear_lock_date` | computation | self | `account` | depends: `fiscalyear_lock_date`; depends_context: `uid`, `ignore_exceptions` |  |
| `_compute_user_tax_lock_date` | computation | self | `account` | depends: `tax_lock_date`; depends_context: `uid`, `ignore_exceptions` |  |
| `_compute_user_sale_lock_date` | computation | self | `account` | depends: `sale_lock_date`; depends_context: `uid`, `ignore_exceptions` |  |
| `_compute_user_purchase_lock_date` | computation | self | `account` | depends: `purchase_lock_date`; depends_context: `uid`, `ignore_exceptions` |  |
| `_compute_user_hard_lock_date` | computation | self | `account` | depends: `hard_lock_date` |  |
| `_compute_account_storno` | computation | self | `account` | depends: `account_fiscal_country_id` |  |
| `_compute_display_account_storno` | computation | self | `account` | depends: `account_fiscal_country_id` |  |
| `_initiate_account_onboardings` | internal rule | self | `account` |  |  |
| `_get_batch_payment_sequence_values` | preparation rule | self | `account` |  |  |
| `_create_batch_payment_sequence` | internal rule | self | `account` |  |  |
| `get_new_account_code` | operation | self, current_code, old_prefix, new_prefix | `account` |  |  |
| `reflect_code_prefix_change` | operation | self, old_code, new_code | `account` |  |  |
| `_get_unreconciled_statement_lines_redirect_action` | preparation rule | self, unreconciled_statement_lines | `account` |  | Get the action redirecting to the statement lines that are not already reconciled. It can i.e. be used when setting a fiscal year lock date or hashing all entries until a certain date.  :param unreconciled_statement_lines: The statement lines. :return: A dictionary representing a window action. |
| `_get_unreconciled_statement_lines_domain` | preparation rule | self, last_date | `account` |  |  |
| `_validate_locks` | internal rule | self, values | `account` |  | Check that the lock date changes are valid. * Check that we do not decrease or remove the hard lock dates. * Check there are no unreconciled bank statement lines in the period we want to lock. * Check there are no unhashed journal entires in the period we want to lock. :param vals: The values passed to the write method. |
| `_get_user_lock_date` | preparation rule | self, soft_lock_date_field, ignore_exceptions | `account` |  | Get the lock date called `soft_lock_date_field` for this company depending on the user. We consider the field and exceptions (except if `ignore_exceptions`) for it in this company and the parent companies. :param str soft_lock_date_field: One of the lock date fields (except 'hard_lock_date'; see SOFT_LOCK_DATE_FIELDS) :param bool ignore_exceptions: Whether we ignore exceptions or not :return the user lock date |
| `_get_user_fiscal_lock_date` | preparation rule | self, journal, ignore_exceptions | `account` |  | Get the fiscal lock date for this company (depending on the affected journal) accounting for potential user exceptions :param bool ignore_exceptions: Whether we ignore exceptions or not :return the lock date |
| `_get_violated_soft_lock_date` | preparation rule | self, soft_lock_date_field, date | `account` |  | Check whether `date` violates the lock date called `soft_lock_date_field`. :param str soft_lock_date_field: One of the lock date fields (except 'hard_lock_date'; see SOFT_LOCK_DATE_FIELDS) :param date: We check whether this date is prior or equal to the lock date. :return the violated lock date as a date (or `None`) |
| `_get_lock_date_violations` | preparation rule | self, accounting_date, fiscalyear, sale, purchase, tax, hard | `account` |  | Get all the lock dates affecting the current accounting_date. :param accounting_date:      The accounting date :param bool fiscalyear:      Whether we should check the `fiscalyear_lock_date` :param bool sale:            Whether we should check the `sale_lock_date` :param bool purchase:        Whether we should check the `purchase_lock_date` :param bool tax:             Whether we should check the `tax_lock_date` :param bool hard:            Whether we should check the `hard_lock_date` :return: a list of tuples containing the lock dates (not ordered chronologically). |
| `_format_lock_dates` | internal rule | self, lock_dates | `account` | model | Format a list of lock dates as a string. :param lock_date_violations: list of tuple (lock_date, lock_date_field) :return: a (localized) string listing all the lock date fields and their values |
| `_get_violated_lock_dates` | preparation rule | self, accounting_date, has_tax, journal | `account` |  | Get all the lock dates affecting the current accounting_date. :param accounting_date: The accounting date :param has_tax: If any taxes are involved in the lines of the invoice :param journal: The affected journal :return: a list of tuples containing the lock dates ordered chronologically. |
| `setting_init_bank_account_action` | operation | self | `account` | model | Called by the 'Bank Accounts' button of the setup bar or from the Financial configuration menu. |
| `setting_init_credit_card_account_action` | operation | self | `account` | model | Called by the Financial configuration menu 'Add a credit card account' |
| `_get_default_opening_move_values` | preparation rule | self | `account` | model | Get the default values to create the opening move.  :return: A dictionary to be passed to account.move.create. |
| `opening_move_posted` | operation | self | `account` |  | Returns true if this company has an opening account move and this move is posted. |
| `get_unaffected_earnings_account` | operation | self | `account` |  | Returns the unaffected earnings account for this company, creating one if none has yet been defined. |
| `_update_opening_move` | internal rule | self, to_update | `account` |  | Create or update the opening move for the accounts passed as parameter.  :param to_update:   A dictionary mapping each account with a tuple (debit, credit).                     A separated opening line is created for both fields. A None value on debit/credit means the corresponding                     line will not be updated. |
| `action_save_onboarding_sale_tax` | user action | self | `account` |  | Set the onboarding step as done |
| `action_save_onboarding_company_data` | user action | self | `account` |  |  |
| `get_chart_of_accounts_or_fail` | operation | self | `account` |  |  |
| `_existing_accounting` | internal rule | self | `account` |  | Return True iff some accounting entries have already been made for the current company. |
| `_chart_template_selection` | internal rule | self | `account` |  |  |
| `_action_check_hash_integrity` | internal rule | self | `account` | model |  |
| `_check_hash_integrity` | validation | self | `account` |  | Checks that all hashed moves have still the same data as when they were hashed and raises an error with the result. |
| `_with_locked_records` | internal rule | self, records, allow_raising | `account` | model | To avoid sending the same records multiple times from different transactions, we use this generic method to lock the records passed as parameter.  :param records: The records to lock. :return: Whether we have locked all records (if there were records to lock) |
| `compute_fiscalyear_dates` | operation | self, current_date | `account` |  | Returns the dates of the fiscal year containing the provided date for this company.  :return: ``{'date_from': ..., 'date_to': ...}`` |
| `_compute_company_vat_placeholder` | computation | self | `account` | depends: `country_id`, `account_fiscal_country_id` |  |
| `_compute_company_registry_placeholder` | computation | self | `account` | depends: `country_id`, `account_fiscal_country_id` | Provides a dynamic placeholder on the company registry field for countries that may need it. Add your country and the value you want in the _ref_company_registry map in the partner.py file. |
| `_set_category_defaults` | internal rule | self | `account`, `stock_account` |  |  |
| `_check_tax_return_configuration` | validation | self | `account` |  | To override in localizations to check if the company is properly configured for tax returns. or related modules are installed. :raises RedirectWarning: if something is wrong configured. |
| `_get_active_peppol_parent_company` | preparation rule | self | `account_peppol` |  | Gets the closest parent company (relative from the current) that has an active peppol connection. :return: res.company record: containing single company if found, empty if not. |
| `_have_unauthorized_peppol_parent_company` | internal rule | self | `account_peppol` |  |  |
| `_reset_peppol_configuration` | internal rule | self, soft | `account_peppol`, `l10n_fr_pdp` |  | Reset all peppol configuration fields to their default value before registering. The EAS, endpoint, email, and phone number will be recomputed so that branch companies that uses their parent configuration can have their default values back (as these fields will be overwritten for them when they register as parent).  :param soft: If True, will only set state to unregistered, but keep peppol config intact, so the user can register again |
| `_check_phonenumbers_import` | validation | self | `account_peppol`, `l10n_dk_nemhandel` | model |  |
| `_sanitize_peppol_phone_number` | internal rule | self, phone_number | `account_peppol` |  |  |
| `_check_peppol_endpoint_number` | validation | self, warning | `account_peppol` |  |  |
| `_peppol_is_french_company` | internal rule | self | `account_peppol` |  |  |
| `_check_account_peppol_phone_number` | validation | self | `account_peppol` | constrains: `account_peppol_phone_number` |  |
| `_check_peppol_endpoint` | validation | self | `account_peppol` | constrains: `peppol_endpoint` |  |
| `_check_peppol_purchase_journal_id` | validation | self | `account_peppol` | constrains: `peppol_purchase_journal_id` |  |
| `_peppol_allows_document_reception` | internal rule | self | `account_peppol`, `l10n_fr_pdp` |  |  |
| `_compute_account_peppol_edi_user` | computation | self | `account_peppol` | depends: `account_edi_proxy_client_ids` |  |
| `_compute_peppol_parent_company_id` | computation | self | `account_peppol` | depends: `peppol_eas`, `peppol_endpoint` |  |
| `_compute_peppol_purchase_journal_id` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state` |  |
| `_inverse_peppol_purchase_journal_id` | inverse computation | self | `account_peppol_response`, `account_peppol` |  |  |
| `_compute_peppol_self_billing_reception_journal_id` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state` |  |
| `_inverse_peppol_self_billing_reception_journal_id` | inverse computation | self | `account_peppol` |  |  |
| `_compute_account_peppol_contact_email` | computation | self | `account_peppol` | depends: `email` |  |
| `_compute_account_peppol_phone_number` | computation | self | `account_peppol` | depends: `phone` |  |
| `_compute_peppol_can_send` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state` |  |
| `_sanitize_peppol_endpoint_in_values` | internal rule | self, values | `account_peppol` | model |  |
| `_peppol_modules_document_types` | internal rule | self | `account_peppol_response`, `account_peppol` |  | Override this function to add supported document types as modules are installed.  :returns: dictionary of the form: {module_name: [(document identifier, document_name)]} |
| `_peppol_supported_document_types` | internal rule | self | `account_peppol`, `l10n_fr_pdp` |  | Returns a flattened dictionary of all supported document types. |
| `_get_peppol_edi_mode` | preparation rule | self, temporary_eas | `account_peppol` |  |  |
| `_get_peppol_webhook_endpoint` | preparation rule | self | `account_peppol` |  |  |
| `_get_company_info_on_peppol` | preparation rule | self, edi_identification | `account_peppol` |  |  |
| `_account_peppol_send_welcome_email` | internal rule | self | `account_peppol` |  |  |
| `_get_peppol_proxy_type` | preparation rule | self | `account_peppol` |  |  |
| `_get_default_nomenclature` | preparation rule | self | `barcodes` |  |  |
| `_get_sms_api_class` | preparation rule | self | `sms_twilio`, `sms` |  |  |
| `_check_prepayment_percent` | validation | self | `sale` | constrains: `prepayment_percent` |  |
| `_default_confirmation_mail_template` | preparation rule | self | `stock` |  |  |
| `_create_transit_location` | internal rule | self | `stock` |  | Create a transit location with company_id being the given company_id. This is needed in case of resuply routes between warehouses belonging to the same company, because we don't want to create accounting entries at that time. |
| `_create_inventory_loss_location` | internal rule | self | `stock` |  |  |
| `_create_production_location` | internal rule | self | `stock` |  |  |
| `_create_scrap_location` | internal rule | self | `stock` |  |  |
| `_create_scrap_sequence` | internal rule | self | `stock` |  |  |
| `create_missing_warehouse` | operation | self | `stock` | model | This hook is used to add a warehouse on the first company of the database |
| `create_missing_transit_location` | operation | self | `stock` | model |  |
| `create_missing_inventory_loss_location` | operation | self | `stock` | model |  |
| `create_missing_production_location` | operation | self | `stock` | model |  |
| `create_missing_scrap_location` | operation | self | `stock` | model |  |
| `create_missing_scrap_sequence` | operation | self | `stock` | model |  |
| `_create_per_company_locations` | internal rule | self | `mrp_subcontracting`, `stock` |  |  |
| `_create_per_company_sequences` | internal rule | self | `mrp_subcontracting_dropshipping`, `mrp`, `stock_dropshipping`, `stock` |  |  |
| `_create_per_company_picking_types` | internal rule | self | `mrp_subcontracting_dropshipping`, `stock_dropshipping`, `stock` |  |  |
| `_create_per_company_rules` | internal rule | self | `mrp_subcontracting_dropshipping`, `stock_dropshipping`, `stock` |  |  |
| `_set_per_company_inter_company_locations` | internal rule | self, inter_company_location | `stock` |  |  |
| `_get_text_validation` | preparation rule | self, confirmation_type | `stock` |  |  |
| `action_close_stock_valuation` | user action | self, at_date, auto_post | `stock_account` |  |  |
| `stock_value` | operation | self, accounts_by_product, at_date | `stock_account` |  |  |
| `stock_accounting_value` | operation | self, accounts_by_product, at_date | `stock_account` |  |  |
| `_action_close_stock_valuation` | internal rule | self, at_date | `stock_account` |  |  |
| `_cron_post_stock_valuation` | background operation | self | `stock_account` | model |  |
| `_get_valuation_product_domain` | preparation rule | self | `mrp_account`, `stock_account` |  |  |
| `_get_accounts_by_product` | preparation rule | self, products | `stock_account` |  |  |
| `_get_extra_balance` | preparation rule | self, vals_list | `stock_account` | model |  |
| `_get_location_valuation_vals` | preparation rule | self, at_date, location_domain | `stock_account` |  |  |
| `_get_stock_valuation_account_vals` | preparation rule | self, accounts_by_product, at_date, extra_aml_vals_list | `stock_account` |  |  |
| `_get_continental_realtime_variation_vals` | preparation rule | self, accounts_by_product, at_date, extra_aml_vals_list | `stock_account` |  | In continental perpetual the inventory variation is never posted. This method compute the variation for a period and post it. |
| `_prepare_inventory_aml_vals` | preparation rule | self, debit_acc, credit_acc, balance, ref, product_id | `stock_account` |  |  |
| `_get_last_closing_date` | preparation rule | self | `stock_account` |  |  |
| `_save_closing_id` | internal rule | self, move_id | `stock_account` |  |  |
| `_default_company_token` | preparation rule | self | `hr_attendance` |  |  |
| `_compute_attendance_kiosk_url` | computation | self | `hr_attendance` | depends: `attendance_kiosk_key` |  |
| `_init_column` | internal rule | self, column_name | `hr_attendance` |  | Initialize the value of the given column for existing rows. Overridden here because we need to generate different access tokens and by default _init_column calls the default method once and applies it for every record. |
| `_regenerate_attendance_kiosk_key` | internal rule | self | `hr_attendance` |  |  |
| `_check_hr_presence_control` | validation | self, at_install | `hr_attendance` |  |  |
| `_action_open_kiosk_mode` | internal rule | self | `hr_attendance` |  |  |
| `_check_country_change_holidays` | validation | self | `hr_holidays` | constrains: `country_id` |  |
| `_compute_website_id` | computation | self | `website` |  |  |
| `action_open_website_theme_selector` | user action | self | `website` | model |  |
| `google_map_img` | operation | self, zoom, width, height | `website` |  |  |
| `google_map_link` | operation | self, zoom | `website` |  |  |
| `_default_project_time_mode_id` | preparation rule | self | `hr_timesheet` | model |  |
| `_default_timesheet_encode_uom_id` | preparation rule | self | `hr_timesheet` | model |  |
| `_check_internal_project_id_company` | validation | self | `hr_timesheet` | constrains: `internal_project_id` |  |
| `_create_internal_project_task` | internal rule | self | `hr_timesheet`, `project_timesheet_holidays` |  |  |
| `iap_enrich_auto` | operation | self | `partner_autocomplete` |  | Enrich company. This method should be called by automatic processes and a protection is added to avoid doing enrich in a loop. |
| `_enrich` | internal rule | self | `partner_autocomplete` |  | This method calls the partner autocomplete service from IAP to enrich partner related fields of the company. |
| `_enrich_extract_m2o_id` | internal rule | self, iap_data, m2o_fields | `partner_autocomplete` |  | Extract m2O ids from data (because of res.partner._format_data_company) |
| `_get_company_domain` | preparation rule | self | `partner_autocomplete` |  | Extract the company domain to be used by IAP services.  The domain is extracted from the website or the email information.  >>> company.email, company._get_company_domain() ("info@proximus.be", "proximus.be") >>> company.website, company._get_company_domain() ("www.info.proximus.be", "proximus.be") |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_es_pos`, `l10n_in_pos`, `l10n_jo_edi_pos`, `l10n_sa_edi_pos`, `point_of_sale` | model |  |
| `validate_lock_dates` | validation | self | `point_of_sale` | constrains: `fiscalyear_lock_date`, `tax_lock_date`, `sale_lock_date`, `hard_lock_date` | This constrains makes it impossible to change the relevant lock dates if some open POS session would violate them. Without that, these POS sessions could not be closed (since the closing entries violate the lock dates). |
| `_compute_l10n_gcc_country_is_gcc` | computation | self | `l10n_gcc_invoice` | depends: `partner_id.country_id.country_group_ids.code` |  |
| `_localization_use_documents` | internal rule | self | `l10n_ar`, `l10n_br`, `l10n_cl`, `l10n_ec`, `l10n_latam_invoice_document`, `l10n_pe`, `l10n_uy` |  | This method is to be inherited by localizations and return True if localization use documents |
| `_is_latam` | internal rule | self | `l10n_ar`, `l10n_br`, `l10n_ec`, `l10n_latam_base`, `l10n_pe`, `l10n_uy` |  | Return whether the given company belong to latam countries or not. |
| `onchange_country` | on change | self | `l10n_ar` | onchange: `country_id` | Argentinean companies use round_globally as tax_calculation_rounding_method |
| `_compute_l10n_ar_company_requires_vat` | computation | self | `l10n_ar` | depends: `l10n_ar_afip_responsibility_type_id` |  |
| `_validate_l10n_de_stnr` | validation | self | `l10n_de` | constrains: `state_id`, `l10n_de_stnr` |  |
| `get_l10n_de_stnr_national` | operation | self | `l10n_de` |  |  |
| `_sanitize_nemhandel_phone_number` | internal rule | self, phone_number | `l10n_dk_nemhandel` |  |  |
| `_check_nemhandel_phone_number` | validation | self | `l10n_dk_nemhandel` | constrains: `nemhandel_phone_number` |  |
| `_check_nemhandel_purchase_journal_id` | validation | self | `l10n_dk_nemhandel` | constrains: `nemhandel_purchase_journal_id` |  |
| `_compute_nemhandel_purchase_journal_id` | computation | self | `l10n_dk_nemhandel` | depends: `l10n_dk_nemhandel_proxy_state` |  |
| `_compute_nemhandel_contact_email` | computation | self | `l10n_dk_nemhandel` | depends: `email` |  |
| `_compute_nemhandel_phone_number` | computation | self | `l10n_dk_nemhandel` | depends: `phone` |  |
| `_compute_nemhandel_edi_user` | computation | self | `l10n_dk_nemhandel` | depends: `account_edi_proxy_client_ids` |  |
| `_get_nemhandel_edi_mode` | preparation rule | self | `l10n_dk_nemhandel` |  |  |
| `_get_nemhandel_webhook_endpoint` | preparation rule | self | `l10n_dk_nemhandel` |  |  |
| `_l10n_es_get_pos_edi_mode` | internal rule | self | `l10n_es_edi_tbai_pos`, `l10n_es_edi_verifactu_pos`, `l10n_es` |  | Return the POS EDI mode for this company. Returns 'tbai', 'verifactu', or False (standard session closing entry). |
| `_l10n_es_edi_facturae_export_check` | internal rule | self | `l10n_es_edi_facturae` |  |  |
| `_compute_l10n_es_sii_certificate` | computation | self | `l10n_es_edi_sii` | depends: `country_id`, `l10n_es_sii_certificate_ids` |  |
| `_compute_l10n_es_tbai_is_enabled` | computation | self | `l10n_es_edi_tbai` | depends: `country_id`, `l10n_es_tbai_tax_agency` |  |
| `_compute_l10n_es_tbai_certificate` | computation | self | `l10n_es_edi_tbai` | depends: `country_id`, `l10n_es_tbai_certificate_ids` |  |
| `_compute_l10n_es_tbai_license_html` | computation | self | `l10n_es_edi_tbai` | depends: `country_id`, `l10n_es_tbai_test_env`, `l10n_es_tbai_tax_agency` |  |
| `_get_l10n_es_tbai_license_dict` | preparation rule | self | `l10n_es_edi_tbai` |  |  |
| `_get_l10n_es_tbai_next_chain_index` | preparation rule | self | `l10n_es_edi_tbai` |  |  |
| `_get_l10n_es_tbai_last_chained_document` | preparation rule | self | `l10n_es_edi_tbai` |  | Returns the last tbai document posted to this company's chain. That tbai document may have been received by the govt or not (eg. in case of a timeout). Only upon confirmed reception/refusal of that tbai document can another one be posted. |
| `_l10n_es_freelancer` | internal rule | self | `l10n_es_edi_tbai` |  |  |
| `_l10n_es_edi_verifactu_get_endpoints` | internal rule | self | `l10n_es_edi_verifactu` |  | For the SOAP endpoints see: https://prewww2.aeat.es/static_files/common/internet/dep/aplicaciones/es/aeat/tikeV1.0/cont/ws/SistemaFacturacion.wsdl |
| `_l10n_es_edi_verifactu_get_certificate` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_get_chain_sequence` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_l10n_es_edi_verifactu_get_last_document` | internal rule | self | `l10n_es_edi_verifactu` |  |  |
| `_map_all_eu_companies_taxes` | internal rule | self | `l10n_eu_oss` | model | Identifies EU companies and calls the _map_eu_taxes function |
| `_map_eu_taxes` | internal rule | self | `l10n_eu_oss` |  | Creates or updates Fiscal Positions for each EU country excluding the company's account_fiscal_country_id |
| `_get_repartition_lines_oss` | preparation rule | self | `l10n_eu_oss` |  |  |
| `_get_oss_account` | preparation rule | self | `l10n_eu_oss` |  |  |
| `_create_oss_account` | internal rule | self | `l10n_eu_oss` |  |  |
| `_get_oss_tags` | preparation rule | self | `l10n_eu_oss` |  |  |
| `_get_country_from_vat` | preparation rule | self | `l10n_eu_oss` |  |  |
| `_get_country_specific_account_tax_fields` | preparation rule | self | `l10n_eu_oss` |  |  |
| `_compute_is_france_country` | computation | self | `l10n_fr` | depends: `country_code` |  |
| `_get_france_country_codes` | preparation rule | self | `l10n_fr` | model | Returns every country code that can be used to represent France |
| `_is_accounting_unalterable` | internal rule | self | `l10n_fr` |  |  |
| `_create_secure_sequence` | internal rule | self, sequence_fields | `l10n_fr` |  | This function creates a no_gap sequence on each company in self that will ensure a unique number is given to all posted account.move in such a way that we can always find the previous move of a journal entry on a specific journal. |
| `_get_fr_reference_leave_type` | preparation rule | self | `l10n_fr_hr_holidays` |  |  |
| `_compute_pdp_identifier` | computation | self | `l10n_fr_pdp` | depends: `peppol_eas`, `peppol_endpoint` |  |
| `_inverse_pdp_identifier` | inverse computation | self | `l10n_fr_pdp` |  |  |
| `_compute_l10n_fr_pdp_registered` | computation | self | `l10n_fr_pdp` | depends: `l10n_fr_pdp_annuaire_start_date`, `account_peppol_proxy_state` |  |
| `_force_update_l10n_fr_f10_moves` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_l10n_fr_pdp_get_f10_moves_query` | internal rule | self, account_ids, date_company_conditions | `l10n_fr_pdp_pos`, `l10n_fr_pdp` |  |  |
| `_check_pdp_identifier` | validation | self, pdp_identifier, warning | `l10n_fr_pdp` | model |  |
| `_l10n_fr_pdp_update_pilot_phase` | internal rule | self, value | `l10n_fr_pdp` |  |  |
| `_pdp_get_flow_10_start_date` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_compute_l10n_fr_pdp_flow_10_start_date` | computation | self | `l10n_fr_pdp` | depends: `l10n_fr_pdp_annuaire_start_date`, `l10n_fr_pdp_periodicity` |  |
| `_compute_l10n_fr_f10_enable_reporting` | computation | self | `l10n_fr_pdp` | depends: `l10n_fr_pdp_send_to_ppf`, `account_fiscal_country_id`, `account_peppol_edi_user` |  |
| `_pdp_get_iap_url` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_refresh_pdp_authentication_status` | internal rule | self, send_bus | `l10n_fr_pdp` |  |  |
| `_action_check_pos_hash_integrity` | internal rule | self | `l10n_fr_pos_cert` |  |  |
| `_check_pos_hash_integrity` | validation | self | `l10n_fr_pos_cert` |  | Checks that all posted or invoiced pos orders have still the same data as when they were posted and raises an error with the result. |
| `_cron_l10n_gr_edi_fetch_invoices` | background operation | self | `l10n_gr_edi` | model | Receive issued myDATA Invoices and create draft Vendor Bills based on the received XML. |
| `_l10n_gr_edi_get_or_create_proxy_user` | internal rule | self | `l10n_gr_edi_e_invoo` |  |  |
| `_l10n_gr_edi_get_proxy_user` | internal rule | self | `l10n_gr_edi_e_invoo` |  |  |
| `_check_l10n_hr_mer_purchase_journal_id` | validation | self | `l10n_hr_edi` | constrains: `l10n_hr_mer_purchase_journal_id` |  |
| `_compute_l10n_hr_mer_purchase_journal_id` | computation | self | `l10n_hr_edi` | depends: `l10n_hr_mer_connection_state` |  |
| `_compute_l10n_hr_mojeracun_state` | computation | self | `l10n_hr_edi` | depends: `l10n_hr_mer_username`, `l10n_hr_mer_password` |  |
| `_l10n_hr_activate_mojeracun` | internal rule | self | `l10n_hr_edi` |  |  |
| `_cron_mer_get_new_documents` | background operation | self | `l10n_hr_edi` |  |  |
| `_cron_mer_update_document_status` | background operation | self | `l10n_hr_edi` |  |  |
| `_cron_mer_archive_signed_xmls` | background operation | self | `l10n_hr_edi` |  |  |
| `_l10n_hr_mer_import_invoice` | internal rule | self, attachment, document | `l10n_hr_edi` |  | Save new documents in an accounting journal, when one is specified on the company. If the document is not an invoice but a rejection notice, log a note on the related move instead. :param attachment: the new document :param document: a dictionary of MER and fiscalization related values for the document :return: `True` if the document was saved, `False` if it was not |
| `_l10n_hr_mer_get_new_documents` | internal rule | self, undelivered_only, slc, from_cron | `l10n_hr_edi` |  | Import documents from MojEracun. Additional arguments included for testing. :param undelivered_only (bool, optional): Import only undelivered documents. Defaults to True. :param slc (tuple of two ints, optional): Import only a slice of the list of documents for testing. Defaults to False. |
| `_l10n_hr_mer_fetch_document_status_company` | internal rule | self, from_cron | `l10n_hr_edi` |  | Fetch and update the status of up to 20000 documents belonging to a company on MojEracun. |
| `_l10n_hr_mer_archive_signed_xmls` | internal rule | self, from_cron | `l10n_hr_edi` |  | Download and archive signed XMLs for sent invoices that haven't been archived yet. :param company: The company record :param from_cron: If True, continue on errors instead of raising |
| `_l10n_hu_edi_configure_company` | internal rule | self | `l10n_hu_edi` |  | Single-time configuration for companies, to be applied when l10n_hu_edi is installed or a new company is created. |
| `_l10n_hu_edi_get_credentials_dict` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_test_credentials` | internal rule | self | `l10n_hu_edi` |  |  |
| `_l10n_hu_edi_recover_transactions` | internal rule | self, connection | `l10n_hu_edi` |  | Recover transactions that are in force but for some reason are not matched to the company's invoices, and update the invoice state correspondingly.  This can happen, for example, if the invoice sending timed out: in that case, we don't have a transaction ID for the invoice. It can also happen if for some reason the transaction ID was overwritten by a new request, but the new request fails with a 'duplicate invoice' error.  To do this, we request a list of all transactions made since l10n_hu_edi_last_transaction_recovery, and then we query the last 10 transactions whose transaction IDs are unkn |
| `l10n_hu_edi_show_nav_sync_button` | operation | self | `l10n_hu_edi_receive` | model |  |
| `l10n_hu_edi_receive_inbound_invoices` | operation | self, datetime_from, datetime_to | `l10n_hu_edi_receive` |  |  |
| `_inverse_l10n_in_tds_feature` | inverse computation | self | `l10n_in` |  |  |
| `_inverse_l10n_in_tcs_feature` | inverse computation | self | `l10n_in` |  |  |
| `_inverse_l10n_in_is_gst_registered` | inverse computation | self | `l10n_in` |  |  |
| `_compute_l10n_in_parent_based_features` | computation | self | `l10n_in` | depends: `parent_id.l10n_in_tds_feature`, `parent_id.l10n_in_tcs_feature`, `parent_id.l10n_in_is_gst_registered` |  |
| `_activate_l10n_in_taxes` | internal rule | self, group_refs, company, active | `l10n_in` |  |  |
| `_compute_l10n_in_hsn_code_digit` | computation | self | `l10n_in` | depends: `vat` |  |
| `onchange_vat` | on change | self | `l10n_in` | onchange: `vat` |  |
| `_update_l10n_in_fiscal_position` | internal rule | self | `l10n_in` |  |  |
| `_update_l10n_in_is_gst_registered` | internal rule | self | `l10n_in` |  |  |
| `action_update_state_as_per_gstin` | user action | self | `l10n_in` |  |  |
| `_l10n_in_edi_token_is_valid` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_get_token` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_edi_authenticate` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_check_einvoice_validation` | internal rule | self | `l10n_in_edi` |  |  |
| `_l10n_in_ewaybill_token_is_valid` | internal rule | self | `l10n_in_ewaybill` |  |  |
| `_check_l10n_it_edi_purchase_journal_id` | validation | self | `l10n_it_edi` | constrains: `l10n_it_edi_purchase_journal_id` |  |
| `_check_eco_admin_index` | validation | self | `l10n_it_edi` | constrains: `l10n_it_has_eco_index`, `l10n_it_eco_index_office`, `l10n_it_eco_index_number`, `l10n_it_eco_index_liquidation_state` |  |
| `_check_eco_incorporated` | validation | self | `l10n_it_edi` | constrains: `l10n_it_has_eco_index`, `l10n_it_eco_index_share_capital`, `l10n_it_eco_index_sole_shareholder` | If the business is incorporated, both these fields must be present. We don't know whether the business is incorporated, but in any case the fields must be both present or not present. |
| `_check_tax_representative` | validation | self | `l10n_it_edi` | constrains: `l10n_it_has_tax_representative`, `l10n_it_tax_representative_partner_id` |  |
| `_compute_l10n_it_edi_proxy_user_id` | computation | self | `l10n_it_edi` | depends: `account_edi_proxy_client_ids`, `l10n_it_codice_fiscale` |  |
| `_compute_l10n_it_edi_purchase_journal_id` | computation | self | `l10n_it_edi` | depends: `country_code` |  |
| `_l10n_it_edi_export_check` | internal rule | self | `l10n_it_edi` |  |  |
| `_onchange_l10n_it_has_tax_represeentative` | on change | self | `l10n_it_edi` | onchange: `l10n_it_has_tax_representative` |  |
| `_l10n_it_get_edi_company` | internal rule | self | `l10n_it_edi` |  |  |
| `_l10n_jo_validate_config` | internal rule | self | `l10n_jo_edi` |  |  |
| `_l10n_jo_build_jofotara_headers` | internal rule | self | `l10n_jo_edi` |  |  |
| `_send_l10n_jo_edi_request` | internal rule | self, params, headers | `l10n_jo_edi` |  |  |
| `_compute_l10n_ke_oscu_is_active` | computation | self | `l10n_ke` |  | Overridden in enterprise when the OSCU module is used in the company |
| `_compute_l10n_my_edi_proxy_user_id` | computation | self | `l10n_my_edi` | depends: `account_edi_proxy_client_ids`, `l10n_my_edi_mode` | Each company is expected to have at most one proxy user for malaysia for each mode. Thus, we can easily find said user. |
| `_compute_l10n_my_identification_number_placeholder` | computation | self | `l10n_my_edi` | depends: `l10n_my_identification_type` | Computes a dynamic placeholder that depends on the selected type to help the user inputs their data. The placeholders have been taken from the MyInvois doc. |
| `_l10n_my_edi_create_proxy_user` | internal rule | self | `l10n_my_edi` |  | This method will create a new proxy user for the current company based on the selected mode, if no users already exists. |
| `_l10n_my_edi_enabled` | internal rule | self | `l10n_my_edi` |  |  |
| `_compute_l10n_pl_edi_register` | computation | self | `l10n_pl_edi` | depends: `l10n_pl_edi_certificate` |  |
| `_cron_l10n_pl_edi_refresh_tokens` | background operation | self | `l10n_pl_edi` | model | Automatically performs a full KSeF authentication to renew both the access token and the refresh token for active companies. |
| `_compute_l10n_ro_edi_callback_url` | computation | self | `l10n_ro_edi` | depends: `country_code` | Callback URLs are used for generating client_id and client_secret from l10n_ro_edi's setting. |
| `_compute_l10n_ro_edi_anaf_imported_inv_journal` | computation | self | `l10n_ro_edi` | depends: `country_code` |  |
| `_l10n_ro_edi_log_message` | internal rule | self, message, func | `l10n_ro_edi` |  |  |
| `_l10n_ro_edi_process_token_response` | internal rule | self, response_json | `l10n_ro_edi` |  | To be called just after processing the json response from https://logincert.anaf.ro/anaf-oauth2/v1/token This method reads and process the json, and writes the token fields on the company. |
| `_l10n_ro_edi_refresh_access_token` | internal rule | self, session | `l10n_ro_edi` |  | Uses the saved client_id, client_secret, and refresh_token on the company (self) to make request to the SPV and renew the company's token fields. |
| `_cron_l10n_ro_edi_refresh_access_token` | background operation | self | `l10n_ro_edi` |  | This CRON method will be run every 30 days to refresh the following fields on the company:   - ``l10n_ro_edi_access_token``  - ``l10n_ro_edi_refresh_token``  - ``l10n_ro_edi_access_expiry_date``  - ``l10n_ro_edi_refresh_expiry_date`` |
| `_cron_l10n_ro_edi_synchronize_invoices` | background operation | self | `l10n_ro_edi` |  | This CRON method will be run every 24 hours to synchronize the invoices and the bills with the ANAF |
| `_l10n_sa_edi_inverse_building_number` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_edi_inverse_plot_identification` | internal rule | self | `l10n_sa_edi` |  |  |
| `_l10n_sa_get_csr_invoice_type` | internal rule | self | `l10n_sa_edi` |  | Return the Invoice Type flag used in the CSR. 4-digit numerical input using 0 & 1 mapped to “TSCZ” where:     -   0: False/Not supported, 1: True/Supported     -   T: Tax Invoice (Standard), S: Simplified Invoice, C & Z will be used in the future and should         always be 0     For example: 1100 would mean the Solution will be generating Standard and Simplified invoices.     We can assume Odoo-powered EGS solutions will always generate both Standard & Simplified invoices :return: |
| `_l10n_sa_check_organization_unit` | internal rule | self | `l10n_sa_edi` |  | Check company Organization Unit according to ZATCA specifications Standards:     BR-KSA-39     BR-KSA-40 See https://zatca.gov.sa/ar/RulesRegulations/Taxes/Documents/20210528_ZATCA_Electronic_Invoice_XML_Implementation_Standard_vShared.pdf |
| `_compute_org_number` | computation | self | `l10n_se` | depends: `vat` |  |
| `_compute_l10n_tr_nilvera_purchase_journal_id` | computation | self | `l10n_tr_nilvera` |  |  |
| `_inverse_l10n_tr_nilvera_purchase_journal_id` | inverse computation | self | `l10n_tr_nilvera` |  |  |
| `_is_ecpay_enabled` | internal rule | self | `l10n_tw_edi_ecpay` |  |  |
| `_get_social_media_links` | preparation rule | self | `mass_mailing`, `website_mass_mailing` |  |  |
| `_create_unbuild_sequence` | internal rule | self | `mrp` |  |  |
| `create_missing_unbuild_sequences` | operation | self | `mrp` | model |  |
| `_create_missing_subcontracting_location` | internal rule | self | `mrp_subcontracting` | model |  |
| `_create_subcontracting_location` | internal rule | self | `mrp_subcontracting` |  |  |
| `_create_dropship_sequence` | internal rule | self | `stock_dropshipping` |  |  |
| `create_missing_dropship_sequence` | operation | self | `stock_dropshipping` | model |  |
| `_create_dropship_picking_type` | internal rule | self | `stock_dropshipping` |  |  |
| `create_missing_dropship_picking_type` | operation | self | `stock_dropshipping` | model |  |
| `_create_dropship_rule` | internal rule | self | `stock_dropshipping` |  |  |
| `create_missing_dropship_rule` | operation | self | `stock_dropshipping` | model |  |
| `_create_subcontracting_dropshipping_sequence` | internal rule | self | `mrp_subcontracting_dropshipping` |  |  |
| `_create_subcontracting_dropshipping_picking_type` | internal rule | self | `mrp_subcontracting_dropshipping` |  |  |
| `_create_subcontracting_dropshipping_rules` | internal rule | self | `mrp_subcontracting_dropshipping` |  |  |
| `_create_missing_subcontracting_dropshipping_rules` | internal rule | self | `mrp_subcontracting_dropshipping` | model |  |
| `_create_missing_subcontracting_dropshipping_sequence` | internal rule | self | `mrp_subcontracting_dropshipping` | model |  |
| `_create_missing_subcontracting_dropshipping_picking_type` | internal rule | self | `mrp_subcontracting_dropshipping` | model |  |
| `_assert_twilio_sid` | internal rule | self | `sms_twilio` |  |  |
| `_action_open_sms_twilio_account_manage` | internal rule | self | `sms_twilio` |  |  |
| `get_fiscal_dates` | operation | self, payload | `spreadsheet_account` | readonly; model |  |
| `_default_confirmation_sms_picking_template` | preparation rule | self | `stock_sms` |  |  |

## Validation and error messages (60)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `copy` | UserError | Duplicating a company is not allowed. Please create a new company instead. | `base` |
| `write` | UserError | The company hierarchy cannot be changed. | `base` |
| `_check_active` | ValidationError | The company %(company_name)s cannot be archived because it is still used as the default company of %(active_users)s users. | `base` |
| `_check_root_delegated_fields` | ValidationError | The %s of a subsidiary must be the same as it's root company. | `base` |
| `_check_audit_trail_restriction` | ValidationError | Can't disable restricted audit trail: forced by localization. | `account` |
| `_check_set_account_price_include` | ValidationError | Cannot change Price Tax computation method on a company that has already started invoicing. | `account` |
| `_check_fiscalyear_last_day` | ValidationError | Invalid fiscal year last day | `account` |
| `_validate_locks` | RedirectWarning | error_msg | `account` |
| `_validate_locks` | RedirectWarning | error_msg | `account` |
| `_validate_locks` | UserError | The Hard Lock Date cannot be removed. | `account` |
| `_validate_locks` | UserError | A new Hard Lock Date must be posterior (or equal) to the previous one. | `account` |
| `write` | UserError | You cannot change the currency of the company since some journal items already exist | `account` |
| `_get_default_opening_move_values` | UserError | Please install a chart of accounts or create a miscellaneous journal before proceeding. | `account` |
| `_update_opening_move` | UserError | You cannot import the "openning_balance" if the opening move (%s) is already posted.                 If you are absolutely sure you want to modify the opening balance of your accounts, reset the move to draft. | `account` |
| `get_chart_of_accounts_or_fail` | RedirectWarning | msg | `account` |
| `_check_hash_integrity` | UserError | Please contact your accountant to print the Hash integrity result. | `account` |
| `_with_locked_records` | UserError | Some documents are being sent by another process already. | `account` |
| `_check_phonenumbers_import` | ValidationError | Please install the phonenumbers library. | `account_peppol` |
| `_sanitize_peppol_phone_number` | ValidationError | error_message | `account_peppol` |
| `_sanitize_peppol_phone_number` | ValidationError | error_message | `account_peppol` |
| `_check_peppol_endpoint` | ValidationError | The Peppol endpoint identification number is not correct. | `account_peppol` |
| `_check_peppol_purchase_journal_id` | ValidationError | A purchase journal must be used to receive Peppol documents. | `account_peppol` |
| `_check_prepayment_percent` | ValidationError | Prepayment percentage must be a valid percentage. | `sale` |
| `action_close_stock_valuation` | UserError | It exists closing entries after the selected date. Cancel them before generate an entry prior to them | `stock_account` |
| `action_close_stock_valuation` | UserError | Everything is correctly closed | `stock_account` |
| `action_close_stock_valuation` | UserError | Please set the Journal for Inventory Valuation in the settings. | `stock_account` |
| `action_close_stock_valuation` | UserError | Please set the Valuation Account for Inventory Valuation in the settings. | `stock_account` |
| `_check_country_change_holidays` | ValidationError | The company country cannot be changed while time off leaves or allocations with the country exist. | `hr_holidays` |
| `_check_active` | ValidationError | The company “%(company_name)s” cannot be archived because it has a linked website “%(website_name)s”. Change that website's company first. | `website` |
| `_check_internal_project_id_company` | ValidationError | The Internal Project of a company should be in that company. | `hr_timesheet` |
| `validate_lock_dates` | ValidationError | Please close all the point of sale sessions in this period before closing it. Open sessions are: %s | `point_of_sale` |
| `write` | UserError | Could not change the ARCA Responsibility of this company because there are already accounting entries. | `l10n_ar` |
| `write` | ValidationError | You cannot change the fiscal country. | `l10n_de` |
| `get_l10n_de_stnr_national` | ValidationError | Your company's SteuerNummer is not compatible with your state | `l10n_de` |
| `get_l10n_de_stnr_national` | ValidationError | Your company's SteuerNummer is not valid | `l10n_de` |
| `_check_phonenumbers_import` | ValidationError | Please install the phonenumbers library. | `l10n_dk_nemhandel` |
| `_sanitize_nemhandel_phone_number` | ValidationError | error_message | `l10n_dk_nemhandel` |
| `_sanitize_nemhandel_phone_number` | ValidationError | error_message | `l10n_dk_nemhandel` |
| `_check_nemhandel_purchase_journal_id` | ValidationError | A purchase journal must be used to receive Nemhandel documents. | `l10n_dk_nemhandel` |
| `_map_eu_taxes` | RedirectWarning | To properly configure OSS tax mapping, the domestic tax group you are using must have the necessary accounts defined. | `l10n_eu_oss` |
| `_get_fr_reference_leave_type` | ValidationError | You must first define a reference time off type for the company. | `l10n_fr_hr_holidays` |
| `_inverse_pdp_identifier` | UserError | The identifier %s is not valid. The expected format is: SIREN, SIREN_SIRET, SIREN_SIRET_CodeRoutage or SIREN_SuffixeAdressage | `l10n_fr_pdp` |
| `_check_pos_hash_integrity` | UserError | Accounting is not unalterable for the company %s. This mechanism is designed for companies where accounting is unalterable. | `l10n_fr_pos_cert` |
| `_check_pos_hash_integrity` | UserError | msg_alert | `l10n_fr_pos_cert` |
| `_check_l10n_hr_mer_purchase_journal_id` | ValidationError | A purchase journal must be used to receive eRacun document via MojEracun. | `l10n_hr_edi` |
| `_l10n_hu_edi_get_credentials_dict` | UserError | Missing NAV credentials for company %s | `l10n_hu_edi` |
| `_l10n_hu_edi_test_credentials` | UserError | NAV Credentials: Please set the hungarian vat number on the company first! | `l10n_hu_edi` |
| `_l10n_hu_edi_test_credentials` | UserError | Incorrect NAV Credentials! Check that your company VAT number is set correctly.  Error details: %s | `l10n_hu_edi` |
| `_check_l10n_it_edi_purchase_journal_id` | ValidationError | The Italian default purchase journal requires a default account. | `l10n_it_edi` |
| `_check_eco_admin_index` | ValidationError | All fields about the Economic and Administrative Index must be completed. | `l10n_it_edi` |
| `_check_eco_incorporated` | ValidationError | If one of Share Capital or Sole Shareholder is present, then they must be both filled out. | `l10n_it_edi` |
| `_check_tax_representative` | ValidationError | You must select a tax representative. | `l10n_it_edi` |
| `_check_tax_representative` | ValidationError | Your tax representative partner must have a tax number. | `l10n_it_edi` |
| `_check_tax_representative` | ValidationError | Your tax representative partner must have a country. | `l10n_it_edi` |
| `_l10n_ro_edi_process_token_response` | ValidationError | Token not found. Response: %s | `l10n_ro_edi` |
| `_l10n_ro_edi_refresh_access_token` | UserError | Client ID and Client Secret field must be filled. | `l10n_ro_edi` |
| `_l10n_ro_edi_refresh_access_token` | UserError | Refresh token not found | `l10n_ro_edi` |
| `write` | UserError | ZATCA API Mode cannot be changed after an invoice has been successfully submitted under the Production Mode. | `l10n_sa_edi` |
| `_assert_twilio_sid` | UserError | Invalid Twilio Account SID: must start with 'AC' and be 34 characters long. | `sms_twilio` |
| `_assert_twilio_sid` | UserError | Invalid Twilio Account SID: must only contain alphanumeric characters after 'AC'. | `sms_twilio` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| company rule portal | `[Command.set([ref('base.group_portal')])]` | `[('id','in', company_ids)]` | True | True | True | True |
| company rule employee | `[Command.set([ref('base.group_user')])]` | `[('id','in', company_ids)]` | True | True | True | True |
| company rule public | `[Command.set([ref('base.group_public')])]` | `[('id','in', company_ids)]` | True | True | True | True |
| company rule erp manager | `[Command.set([ref('base.group_erp_manager')])]` | `[(1,'=',1)]` | True | True | True | True |

## Views (39)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_company_form` | xpath | `base.view_company_form` | `account_enabled_tax_country_ids` |  |  | `account` |
| `account.res_company_view_form_terms` | form |  | `invoice_terms_html` | `Save`, `Discard` |  | `account` |
| `account.res_company_form_view_onboarding` | form |  | `logo`, `name`, `vat`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `company_registry`, `currency_id`, `phone`, `email`, `website` | `Save`, `Discard` |  | `account` |
| `account.res_company_form_view_onboarding_sale_tax` | form |  | `account_sale_tax_id` | `Apply`, `Cancel` |  | `account` |
| `base.view_company_form` | form |  | `logo`, `name`, `partner_id`, `street`, `street2`, `city`, `state_id`, `zip`, `country_id`, `vat`, `company_registry_placeholder`, `company_registry`, `currency_id`, `phone`, `email`, `website`, `parent_id`, `color`, `child_ids` | `Branches` |  | `base` |
| `base.view_company_tree` | list |  | `sequence`, `logo`, `name`, `partner_id`, `child_ids` |  |  | `base` |
| `base.view_res_company_kanban` | kanban |  | `name`, `email`, `phone`, `email`, `phone` |  |  | `base` |
| `google_address_autocomplete.view_company_form_inherit_address_autocomplete` | xpath | `base.view_company_form` |  |  |  | `google_address_autocomplete` |
| `l10n_ar.view_company_form` | field | `base.view_company_form` | `vat`, `l10n_ar_afip_responsibility_type_id`, `l10n_ar_gross_income_type`, `l10n_ar_gross_income_number`, `l10n_ar_afip_start_date` |  |  | `l10n_ar` |
| `l10n_au.view_company_form` | xpath | `base.view_company_form` |  |  |  | `l10n_au` |
| `l10n_br.view_company_form` | xpath | `base.view_company_form` | `l10n_br_ie_code`, `l10n_br_im_code`, `l10n_br_nire_code` |  |  | `l10n_br` |
| `l10n_ca.res_company_form_inherit_ca` | xpath | `base.view_company_form` | `l10n_ca_pst` |  |  | `l10n_ca` |
| `l10n_cl.view_company_l10n_cl_form` | field | `base.view_company_form` | `vat`, `l10n_cl_activity_description` |  |  | `l10n_cl` |
| `l10n_cz.view_company_form_inherit_l10n_ck` | field | `base.view_company_form` | `currency_id`, `trade_registry` |  |  | `l10n_cz` |
| `l10n_de.res_company_form_l10n_de` | field | `account.view_company_form` | `vat`, `l10n_de_stnr`, `l10n_de_widnr` |  |  | `l10n_de` |
| `l10n_dk.view_company_form_inherit_l10n_dk` | xpath | `base.view_company_form` |  |  |  | `l10n_dk` |
| `l10n_es_edi_tbai.res_company_form_l10n_es_edi_tbai` | xpath | `account.view_company_form` | `l10n_es_tbai_license_html` |  |  | `l10n_es_edi_tbai` |
| `l10n_es_edi_verifactu.view_company_form` | xpath | `base.view_company_form` | `l10n_es_edi_verifactu_special_vat_regime` |  |  | `l10n_es_edi_verifactu` |
| `l10n_fi.view_company_form_inherit_l10n_fi` | xpath | `base.view_company_form` |  |  |  | `l10n_fi` |
| `l10n_fr.res_company_form_l10n_fr` | data | `base.view_company_form` | `company_registry`, `ape` |  |  | `l10n_fr` |
| `l10n_gr_edi.view_company_form` | page | `base.view_company_form` | `l10n_gr_edi_test_env`, `l10n_gr_edi_aade_id`, `l10n_gr_edi_aade_key`, `l10n_gr_edi_branch_number` |  |  | `l10n_gr_edi` |
| `l10n_hu_edi.view_company_form_l10n_hu_edi` | xpath | `account.view_company_form` | `account_fiscal_country_id`, `l10n_hu_group_vat` |  |  | `l10n_hu_edi` |
| `l10n_in.view_company_form` | xpath | `base.view_company_form` | `l10n_in_upi_id` |  |  | `l10n_in` |
| `l10n_it_edi.res_company_form_l10n_it` | data | `base.view_company_form` | `l10n_it_codice_fiscale`, `l10n_it_tax_system`, `l10n_it_has_eco_index`, `l10n_it_eco_index_office`, `l10n_it_eco_index_number`, `l10n_it_eco_index_share_capital`, `l10n_it_eco_index_sole_shareholder`, `l10n_it_eco_index_liquidation_state`, `l10n_it_has_tax_representative`, `l10n_it_tax_representative_partner_id` |  |  | `l10n_it_edi` |
| `l10n_lk_invoice.view_company_form_l10n_lk_vat_registered` | xpath | `base.view_company_form` | `l10n_lk_vat_registered` |  |  | `l10n_lk_invoice` |
| `l10n_ma.view_company_form` | xpath | `account.view_company_form` |  |  |  | `l10n_ma` |
| `l10n_my_edi.view_company_form_inherit_l10n_my_myinvois` | xpath | `base.view_company_form` | `l10n_my_identification_type`, `l10n_my_identification_number_placeholder`, `l10n_my_identification_number`, `l10n_my_edi_industrial_classification` |  |  | `l10n_my_edi` |
| `l10n_my_ubl_pint.view_company_form_inherit_l10n_my_ubl_pint` | xpath | `base.view_company_form` | `sst_registration_number`, `ttx_registration_number` |  |  | `l10n_my_ubl_pint` |
| `l10n_no.res_company_form_inherit_no` | xpath | `base.view_company_form` | `l10n_no_bronnoysund_number` |  |  | `l10n_no` |
| `l10n_nz.view_company_form_inherit_l10n_nz` | xpath | `base.view_company_form` |  |  |  | `l10n_nz` |
| `l10n_ph.view_company_form_inherit_l10n_ph` | xpath | `base.view_company_form` | `branch_code`, `l10n_ph_rdo` |  |  | `l10n_ph` |
| `l10n_sa_edi.view_company_form` | xpath | `base.view_company_form` | `l10n_sa_edi_building_number`, `l10n_sa_edi_plot_identification` |  |  | `l10n_sa_edi` |
| `l10n_sg.view_company_form_l10n_sg` | xpath | `base.view_company_form` | `l10n_sg_unique_entity_number` |  |  | `l10n_sg` |
| `l10n_sk.view_company_form_inherit_l10n_ck` | field | `base.view_company_form` | `currency_id`, `trade_registry`, `income_tax_id` |  |  | `l10n_sk` |
| `l10n_tr_nilvera_einvoice_extended.view_company_form_inherit_l10n_tr_nilvera_extended` | xpath | `base.view_company_form` | `l10n_tr_tax_office_id` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_uz.l10n_uz_view_company_form` | xpath | `base.view_company_form` |  |  |  | `l10n_uz` |
| `mail.res_company_view_form` | field | `base.view_company_form` | `parent_id`, `alias_domain_id`, `bounce_formatted`, `catchall_formatted`, `default_from_email` |  |  | `mail` |
| `partner_autocomplete.view_company_form_inherit_partner_autocomplete` | xpath | `base.view_company_form` | `iap_enrich_auto_done` |  |  | `partner_autocomplete` |
| `social_media.view_company_form_inherit_social_media` | xpath | `base.view_company_form` | `social_twitter`, `social_facebook`, `social_github`, `social_linkedin`, `social_youtube`, `social_instagram`, `social_tiktok`, `social_discord` |  |  | `social_media` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_res_company_form` | Companies | list,kanban,form | `[('parent_id', '=', False)]` |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `l10n_es_edi_tbai.menu_l10n_es_edi_tbai_license` | Licenses | `menu_l10n_es_edi_tbai_root` | `base.action_res_company_form` | 90 | `account.group_account_manager` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `account.action_check_hash_integrity` | Data Inalterability Check | code |  | yes |
| `hr_attendance.open_kiosk_url` | Open Kiosk Url | code |  | yes |
| `l10n_fr_pos_cert.action_check_pos_hash_integrity` | POS Inalterability Check | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `account.action_report_account_hash_integrity` | Hash integrity result PDF | qweb-pdf | `account.report_hash_integrity` |  |  |
| `l10n_fr_pos_cert.action_report_pos_hash_integrity` | Hash integrity result PDF | qweb-pdf | `l10n_fr_pos_cert.report_pos_hash_integrity` |  |  |
| `l10n_mt_pos.report_compliance_letter` | Compliance Letter | qweb-pdf | `l10n_mt_pos.report_compliance_letter_template` | `'Compliance Letter'` |  |
| `web.action_report_internalpreview` | Preview Internal Report | qweb-pdf | `web.preview_internalreport` |  |  |
| `web.action_report_externalpreview` | Preview External Report | qweb-pdf | `web.preview_externalreport` |  |  |
| `web.action_report_layout_preview` | Report Layout Preview | qweb-pdf | `web.preview_layout_report` |  |  |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `l10n_hr_edi.ir_cron_mer_get_new_documents` | MojEracun: retrieve new documents | 4 hours | `_cron_mer_get_new_documents` |  |
| `l10n_hr_edi.ir_cron_mer_update_document_status` | MojEracun: update statuses of documents | 4 hours | `_cron_mer_update_document_status` |  |
| `l10n_hr_edi.ir_cron_mer_archive_signed_xmls` | MojEracun: archived signed XMLs | 4 hours | `_cron_mer_archive_signed_xmls` |  |
| `l10n_pl_edi.cron_l10n_pl_edi_refresh_tokens` | Polish eInvoice: Refresh KSeF tokens | 6 days | `_cron_l10n_pl_edi_refresh_tokens` |  |
| `stock_account.ir_cron_post_stock_valuation` | Stock Account: Inventory Valuation Closing | 1 days | `_cron_post_stock_valuation` |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `account_peppol.mail_template_peppol_registration` | Peppol: Registration update | Welcome to Peppol |

Machine-readable definition: `../../../schemas/data/entities/res.company.json`; views: `../../../schemas/interfaces/views/res.company.json`.
