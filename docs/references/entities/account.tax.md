# Tax (`account.tax`)

**Transport name:** `account.tax`  
**Storage name:** `account_tax`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`  
**Extended by packages:** `account_edi_ubl_cii`, `account_tax_python`, `hr_expense`, `l10n_account_withholding_tax`, `point_of_sale`, `l10n_account_withholding_tax_pos`, `l10n_ar_withholding`, `l10n_be`, `l10n_br`, `l10n_cl`, `l10n_de`, `purchase`, `l10n_ec`, `l10n_ee`, `l10n_eg`, `l10n_es`, `l10n_es_edi_facturae`, `l10n_es_edi_verifactu`, `l10n_gr_edi`, `l10n_hr_edi`, `l10n_hu_edi`, `l10n_in`, `l10n_in_pos`, `l10n_it`, `l10n_it_edi`, `l10n_it_edi_doi`, `l10n_jo_edi`, `l10n_ke`, `l10n_lt`, `l10n_mx`, `l10n_my_edi`, `l10n_no`, `l10n_pe`, `l10n_ph`, `l10n_pt`, `l10n_sa_edi`, `l10n_sg_ubl_pint`, `l10n_tr_nilvera_einvoice_extended`, `l10n_tw_edi_ecpay`, `l10n_tw_edi_ecpay_pos`, `l10n_uy`, `pos_account_tax_python`

Description: Tax

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `pos.load.mixin`
- Default ordering: `sequence,id`
- Display name search fields: `["name", "description", "invoice_label"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (105)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tax Name | single line text |  | required; translatable; changes are tracked in the message thread |
| `type_tax_use` | Tax Type | selection |  | required; default `sale`; changes are tracked in the message thread; Help: Determines where the tax is selectable. Note: 'None' means a tax can't be used by itself, however it can still be used in a group. 'adjustment' is used to perform tax adjustment. |
| `tax_scope` | Tax Scope | selection |  | extended by packages `l10n_be` |
| `amount_type` | Tax Computation | selection |  | required; default `percent`; changes are tracked in the message thread; on delete of the target: {"expression": "{'code': lambda recs: recs.write({'amount_type': 'percent', 'active': False})}"}; Help: - Group of Taxes: The tax is a set of sub taxes.     - Fixed: The tax amount stays the same whatever the price.     - Percentage: The tax amount is a % of the price:         e.g 100 * (1 + 10%) = 110 (not price included)         e.g 110 / (1 + 10%) = 100 (price included)     - Percentage Tax Included: The tax amount is a division of the price:         e.g 180 / (1 - 10%) = 200 (not price included)         e.g 200 * (1 - 10%) = 180 (price included); extended by packages `account_tax_python` |
| `fiscal_position_ids` | Fiscal Position | many to many | `account.fiscal.position` | association table `account_fiscal_position_account_tax_rel` |
| `original_tax_ids` | Replaces | many to many | `account.tax` | on delete of the target: cascade; restricted by domain `[             ('type_tax_use', '=', type_tax_use),             ('is_domestic', '=', True),         ]`; association table `account_tax_alternatives`; Help: List of taxes to replace when applying any of the stipulated fiscal positions. |
| `replacing_tax_ids` | Replaced by | many to many | `account.tax` | read only; association table `account_tax_alternatives` |
| `display_alternative_taxes_field` | Display Alternative Taxes Field | boolean |  | computed by rule `_compute_display_alternative_taxes_field` (not stored) |
| `is_domestic` | Is Domestic | boolean |  | computed by rule `_compute_is_domestic` and stored; precomputed before insertion |
| `active` | Active | boolean |  | default `True`; Help: Set active to false to hide the tax without removing it. |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `children_tax_ids` | Children Taxes | many to many | `account.tax` | must belong to the same company; association table `account_tax_filiation_rel` |
| `sequence` | Sequence | integer |  | required; default `1`; Help: The sequence field is used to define order in which the tax lines are applied. |
| `amount` | Amount | float |  | required; default ; changes are tracked in the message thread; precision `[16, 4]` |
| `description` | Description | rich text |  | translatable |
| `invoice_label` | Label on Invoices | single line text |  | translatable |
| `tax_label` | Tax Label | single line text |  | computed by rule `_compute_tax_label` (not stored) |
| `price_include` | Price Include | boolean |  | computed by rule `_compute_price_include` (not stored); searchable through a search rule; Help: Determines whether the price you use on the product and invoices includes this tax. |
| `company_price_include` | Company Price Include | selection |  | related through path `company_id.account_price_include` |
| `price_include_override` | Included in Price | selection |  | changes are tracked in the message thread; Help: Overrides the Company's default on whether the price you use on the product and invoices includes this tax. |
| `include_base_amount` | Affect Base of Subsequent Taxes | boolean |  | default ; changes are tracked in the message thread; Help: If set, taxes with a higher sequence than this one will be affected by it, provided they accept it. |
| `is_base_affected` | Base Affected by Previous Taxes | boolean |  | default `True`; changes are tracked in the message thread; Help: If set, taxes with a lower sequence might affect this one, provided they try to do it. |
| `analytic` | Include in Analytic Cost | boolean |  | Help: If set, the amount computed by this tax will be assigned to the same analytic account as the invoice line (if any) |
| `tax_group_id` | Tax Group | many to one | `account.tax.group` | required; computed by rule `_compute_tax_group_id` and stored; restricted by domain `[('country_id', 'in', (country_id, False))]`; precomputed before insertion |
| `hide_tax_exigibility` | Hide Use Cash Basis Option | boolean |  | read only; related through path `company_id.tax_exigibility` |
| `tax_exigibility` | Tax Exigibility | selection |  | default `on_invoice`; Help: Based on Invoice: the tax is due as soon as the invoice is validated. Based on Payment: the tax is due as soon as the payment of the invoice is received. |
| `cash_basis_transition_account_id` | Cash Basis Transition Account | many to one | `account.account` | restricted by domain `[('account_type', 'not in', ('asset_receivable', 'liability_payable'))]`; must belong to the same company; Help: Account used to transition the tax amount for cash basis taxes. It will contain the tax amount as long as the original invoice has not been reconciled ; at reconciliation, this amount cancelled on this account and put on the regular tax account. |
| `invoice_repartition_line_ids` | Distribution for Invoices | one to many | `account.tax.repartition.line` | computed by rule `_compute_invoice_repartition_line_ids` and stored; restricted by domain `[["document_type", "=", "invoice"]]`; inverse field `tax_id`; Help: Distribution when the tax is used on an invoice |
| `refund_repartition_line_ids` | Distribution for Refund Invoices | one to many | `account.tax.repartition.line` | computed by rule `_compute_refund_repartition_line_ids` and stored; restricted by domain `[["document_type", "=", "refund"]]`; inverse field `tax_id`; Help: Distribution when the tax is used on a refund |
| `repartition_line_ids` | Distribution | one to many | `account.tax.repartition.line` | inverse field `tax_id` |
| `country_id` | Country | many to one | `res.country` | required; computed by rule `_compute_country_id` and stored; precomputed before insertion; Help: The country for which this tax is applicable. |
| `country_code` | Country Code | single line text |  | read only; related through path `country_id.code` |
| `company_country_code` | Fiscal Country Code of the Company | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `is_used` | Tax used | boolean |  | computed by rule `_compute_is_used` (not stored) |
| `repartition_lines_str` | Repartition Lines | single line text |  | computed by rule `_compute_repartition_lines_str` (not stored); changes are tracked in the message thread |
| `invoice_legal_notes` | Legal Notes | rich text |  | translatable; Help: Legal mentions that have to be printed on the invoices. |
| `has_negative_factor` | Has Negative Factor | boolean |  | computed by rule `_compute_has_negative_factor` (not stored) |
| `account_move_line_ids` | Account Move Line | many to many | `account.move.line` | read only; not copied on duplication; association table `account_move_line_account_tax_rel` |
| `account_reconcile_model_line_ids` | Account Reconcile Model Line | many to many | `account.reconcile.model.line` | read only; not copied on duplication; association table `account_reconcile_model_line_account_tax_rel` |
| `ubl_cii_tax_category_code` | Tax Category Code | selection |  | Help: The VAT category code used for electronic invoicing purposes.; extended by packages `l10n_sg_ubl_pint` |
| `ubl_cii_tax_exemption_reason_code` | Tax Exemption Reason Code | selection |  | Help: The reason why the amount is exempted from VAT or why no VAT is being charged, used for electronic invoicing purposes. |
| `ubl_cii_requires_exemption_reason` | Universal Business Language Cross Industry Invoice Requires Exemption Reason | boolean |  | computed by rule `_compute_ubl_cii_requires_exemption_reason` (not stored) |
| `formula` | Formula | multi line text |  | default `price_unit * 0.10`; Help: Compute the amount of the tax.  :param base: float, actual amount on which the tax is applied :param price_unit: float :param quantity: float :param product: A object representing the product |
| `formula_decoded_info` | Formula Decoded Info | structured document |  | computed by rule `_compute_formula_decoded_info` (not stored) |
| `hr_expense_ids` | Human resources Expense | many to many | `hr.expense` | read only; not copied on duplication; association table `expense_tax` |
| `is_withholding_tax_on_payment` | Withhold On Payment | boolean |  | Help: If enabled, this tax will not affect your accounts until the registration of payments. |
| `withholding_sequence_id` | Withholding Sequence | many to one | `ir.sequence` | not copied on duplication; must belong to the same company; Help: This sequence will be used to generate default numbers on payment withholding lines. |
| `pos_order_line_ids` | Point of sale Order Line | many to many | `pos.order.line` | read only; not copied on duplication; association table `account_tax_pos_order_line_rel` |
| `l10n_ar_type_tax_use` | Argentina Tax Type | selection |  | computed by rule `_compute_l10n_ar_type_tax_use` (not stored); writable through an inverse rule |
| `l10n_ar_withholding_payment_type` | Argentina Withholding Payment Type | selection |  | Help: Withholding tax for supplier or customer payments. |
| `l10n_ar_tax_type` | WTH Tax | selection |  |  |
| `l10n_ar_withholding_sequence_id` | WTH Sequence | many to one | `ir.sequence` | not copied on duplication; must belong to the same company; Help: If no sequence provided then it will be required for you to enter withholding number when registering one. |
| `l10n_ar_code` | ARCA Code | single line text |  |  |
| `l10n_ar_non_taxable_amount` | Non Taxable Amount | float |  | precision `Account`; Help: Until this base amount, the tax is not applied. |
| `l10n_ar_minimum_threshold` | Minimum Treshold | float |  | Help: If the calculated withholding tax amount is lower than minimum withholding threshold then it is 0.0. |
| `l10n_ar_state_id` | Jurisdiction | many to one | `res.country.state` | on delete of the target: restrict; restricted by domain `[('country_id', '=?', country_id)]` |
| `l10n_ar_scale_id` | Scale | many to one | `l10n_ar.earnings.scale` | Help: Earnings table scale if tax type is 'Earnings Scale'. |
| `tax_discount` | Discount this Tax in Price | boolean |  | Help: Mark it for (ICMS, PIS e etc.). |
| `base_reduction` | Redution | float |  | required; default ; Help: Um percentual decimal em % entre 0-1. |
| `amount_mva` | MVA Percent | float |  | required; default ; Help: Um percentual decimal em % entre 0-1. |
| `l10n_cl_sii_code` | immediate supply of information Code | integer |  |  |
| `l10n_de_datev_code` | Localization De Datev Code | single line text |  | changes are tracked in the message thread; maximum length 4; Help: 4 digits code use by Datev |
| `purchase_order_line_ids` | Purchase Order Line | many to many | `purchase.order.line` | read only; not copied on duplication; association table `account_tax_purchase_order_line_rel` |
| `l10n_ec_code_base` | Code base | single line text |  | Help: Ecuador: Tax declaration code of the base amount prior to the calculation of the tax. |
| `l10n_ec_code_applied` | Code applied | single line text |  | Help: Ecuador: Tax declaration code of the resulting amount after the calculation of the tax. |
| `l10n_ec_code_ats` | Code ATS | single line text |  | Help: Ecuador: Indicates if the purchase invoice supports tax credit or cost or expenses, conforming table 5 of ATS. |
| `l10n_ee_kmd_inf_code` | KMD INF Code | selection |  | default ; Help: This field is used for the comments/special code column in the KMD INF report. |
| `l10n_eg_eta_code` | ETA Code (Egypt) | selection |  | default  |
| `l10n_es_exempt_reason` | Exempt Reason (Spain) | selection |  |  |
| `l10n_es_type` | Tax Type (Spain) | selection |  | default `sujeto` |
| `l10n_es_bien_inversion` | Bien de Inversion | boolean |  | default  |
| `l10n_es_edi_facturae_tax_type` | Spanish Facturae electronic data interchange Tax Type | selection |  |  |
| `l10n_es_applicability` | Applicability (Spain) | selection |  |  |
| `l10n_gr_edi_default_tax_exemption_category` | Default Tax Exemption Category | selection |  |  |
| `l10n_hr_tax_category_id` | Croatian Tax Expence Category | many to one | `l10n.hr.tax.category` |  |
| `l10n_hu_tax_type` | NAV value-added tax Tax Type | selection |  | Help: Precise identification of the VAT tax for the Hungarian authority. |
| `l10n_hu_tax_reason` | NAV value-added tax Tax Exemption Reason | single line text |  | computed by rule `_compute_l10n_hu_tax_reason` (not stored); Help: May be used to provide support for the use of a VAT-exempt VAT tax type. |
| `l10n_in_reverse_charge` | Reverse charge | boolean |  | Help: Tick this if this tax is reverse charge. Only for Indian accounting |
| `l10n_in_gst_tax_type` | Localization In Goods and services tax Tax Type | selection |  | computed by rule `_compute_l10n_in_gst_tax_type` (not stored) |
| `l10n_in_is_lut` | LUT | boolean |  | Help: Tick this if this tax is used in LUT (Letter of Undertaking) transactions. Only for Indian accounting. |
| `l10n_in_tax_type` | Indian Tax Type | selection |  |  |
| `l10n_in_section_id` | Section | many to one | `l10n_in.section.alert` |  |
| `l10n_in_tds_feature_enabled` | Localization In Tax deducted at source Feature Enabled | boolean |  | related through path `company_id.l10n_in_tds_feature` |
| `l10n_in_tcs_feature_enabled` | Localization In Tax collected at source Feature Enabled | boolean |  | related through path `company_id.l10n_in_tcs_feature` |
| `l10n_it_exempt_reason` | Exoneration | selection |  | Help: Exoneration type |
| `l10n_it_withholding_type` | Withholding tax type (Italy) | selection |  | Help: Withholding tax type. Only for Italian accounting EDI. |
| `l10n_it_withholding_reason` | Withholding tax reason (Italy) | selection |  | Help: Withholding tax reason. Only for Italian accounting EDI. |
| `l10n_it_pension_fund_type` | Pension fund type (Italy) | selection |  | Help: Pension Fund Type. Only for Italian accounting EDI. |
| `l10n_ke_item_code_id` | KRA Item Code | many to one | `l10n_ke.item.code` | Help: KRA code that describes a tax rate or exemption on specific products or services. |
| `l10n_lt_tax_code` | Tax Code | single line text |  | Help: The Lithuanian tax system has a code for standard taxes, to use in iSAF and SAFT. |
| `l10n_mx_factor_type` | Factor Type | selection |  | default `Tasa`; Help: Mexico: 'TipoFactor' is an attribute for CFDI 4.0. This indicates the factor type that is applied to the base of the tax. |
| `l10n_mx_tax_type` | SAT Tax Type | selection |  | computed by rule `_compute_l10n_mx_tax_type` and stored |
| `l10n_my_tax_type` | Malaysian Tax Type | selection |  | computed by rule `_compute_l10n_my_tax_type` and stored |
| `l10n_my_tax_exemption_reason` | Malaysian Tax Exemption Reason | single line text |  | Help: The reason for tax exemption, used when submitting consolidated invoices including this tax. |
| `l10n_no_standard_code` | Standard Tax Code | single line text |  | Help: The Norwegian tax system has a code for standard taxes, to use when reporting to them. |
| `l10n_pe_edi_tax_code` | Code | selection |  | Help: Peru: SUNAT tax code |
| `l10n_pe_edi_unece_category` | UNECE Code | selection |  | Help: Peru: Follow the UN/ECE 5305 standard from the United Nations Economic Commission for Europe for more information http://www.unece.org/trade/untdid/d08a/tred/tred5305.html |
| `l10n_pe_edi_isc_type` | ISC Type | selection |  | Help: Used in Selective Consumption Tax to indicate the type of calculation for the ISC. |
| `l10n_ph_atc` | Philippines ATC | single line text |  |  |
| `l10n_sa_is_retention` | Is Retention | boolean |  | default ; Help: Determines whether or not a tax counts as a Withholding Tax |
| `l10n_sa_exemption_reason_code` | Exemption Reason Code | selection |  | Help: Tax Exemption Reason Code (ZATCA) |
| `l10n_tr_tax_withholding_code_id` | Withholding Reason | many to one | `l10n_tr_nilvera_einvoice_extended.account.tax.code` | restricted by domain `[('code_type', '=', 'withholding')]`; Help: The reason for withholding tax. |
| `l10n_tw_edi_tax_type` | Ecpay Tax Type | selection |  | computed by rule `_compute_l10n_tw_edi_tax_type` and stored |
| `l10n_tw_edi_special_tax_type` | Ecpay Special Tax Type | selection |  |  |
| `l10n_uy_tax_category` | Tax Category | selection |  | Help: UY: Use to group the transactions in the Financial Reports required by DGI |

## Selection values

### `tax_scope` (Tax Scope)

| Value | Label |
|---|---|
| `service` | Services |
| `consu` | Goods |
| `merch` | Merchandise |
| `invest` | Investment |

### `amount_type` (Tax Computation)

| Value | Label |
|---|---|
| `group` | Group of Taxes |
| `fixed` | Fixed |
| `percent` | Percentage |
| `division` | Percentage Tax Included |
| `code` | Custom Formula |

### `price_include_override` (Included in Price)

| Value | Label |
|---|---|
| `tax_included` | Tax Included |
| `tax_excluded` | Tax Excluded |

### `tax_exigibility` (Tax Exigibility)

| Value | Label |
|---|---|
| `on_invoice` | Based on Invoice |
| `on_payment` | Based on Payment |

### `ubl_cii_tax_category_code` (Tax Category Code)

| Value | Label |
|---|---|
| `AE` | AE - Vat Reverse Charge |
| `E` | E - Exempt from Tax |
| `S` | S - Standard rate |
| `Z` | Z - Zero rated goods |
| `G` | G - Free export item, VAT not charged |
| `O` | O - Services outside scope of tax |
| `K` | K - VAT exempt for EEA intra-community supply of goods and services |
| `L` | L - Canary Islands general indirect tax |
| `M` | M - Tax for production, services and importation in Ceuta and Melilla |
| `B` | B - Transferred (VAT), In Italy |
| `SR` | SG - Local supply of goods and services |
| `SRCA-S` | SG - Customer accounting supply made by the supplier |
| `SRCA-C` | SG - Customer accounting supply made by the customer on supplier’s behalf |
| `SROVR-RS` | SG - Supply of remote services accountable by the electronic marketplace under the Overseas Vendor Registration Regime |
| `SROVR-LVG` | SG - Supply of low-value goods accountable by the redeliverer or electronic marketplace on behalf of third-party suppliers |
| `SRRC` | SG - Reverse charge regime for Business-to-Business (“B2B”) supplies of imported services |
| `SRLVG` | SG - Own supply of low-value goods |
| `ZR` | SG - Supplies involving goods for export/ provision of international services |
| `ES33` | SG - Specific categories of exempt supplies listed under regulation 33 of the GST (General) Regulations |
| `ESN33` | SG - Exempt supplies other than those listed under regulation 33 of the GST (General) Regulations |
| `DS` | SG - Supplies required to be reported pursuant to the GST legislation |
| `OS` | SG - Supplies outside the scope of the GST Act |
| `NG` | SG - Supplies from a company which is not registered for GST |
| `NA` | SG - Taxable supplies where GST need not be charged |

### `ubl_cii_tax_exemption_reason_code` (Tax Exemption Reason Code)

| Value | Label |
|---|---|
| `VATEX-EU-79-C` | VATEX-EU-79-C - Exempt based on article 79, point c of Council Directive 2006/112/EC |
| `VATEX-EU-132` | VATEX-EU-132 - Exempt based on article 132 of Council Directive 2006/112/EC |
| `VATEX-EU-132-1A` | VATEX-EU-132-1A - Exempt based on article 132, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1B` | VATEX-EU-132-1B - Exempt based on article 132, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1C` | VATEX-EU-132-1C - Exempt based on article 132, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1D` | VATEX-EU-132-1D - Exempt based on article 132, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1E` | VATEX-EU-132-1E - Exempt based on article 132, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1F` | VATEX-EU-132-1F - Exempt based on article 132, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1G` | VATEX-EU-132-1G - Exempt based on article 132, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1H` | VATEX-EU-132-1H - Exempt based on article 132, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1I` | VATEX-EU-132-1I - Exempt based on article 132, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1J` | VATEX-EU-132-1J - Exempt based on article 132, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1K` | VATEX-EU-132-1K - Exempt based on article 132, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1L` | VATEX-EU-132-1L - Exempt based on article 132, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1M` | VATEX-EU-132-1M - Exempt based on article 132, section 1 (m) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1N` | VATEX-EU-132-1N - Exempt based on article 132, section 1 (n) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1O` | VATEX-EU-132-1O - Exempt based on article 132, section 1 (o) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1P` | VATEX-EU-132-1P - Exempt based on article 132, section 1 (p) of Council Directive 2006/112/EC |
| `VATEX-EU-132-1Q` | VATEX-EU-132-1Q - Exempt based on article 132, section 1 (q) of Council Directive 2006/112/EC |
| `VATEX-EU-135-1` | VATEX-EU-135-1 - Exempt based on article 135, section 1 of Council Directive 2006/112/EC |
| `VATEX-EU-143` | VATEX-EU-143 - Exempt based on article 143 of Council Directive 2006/112/EC |
| `VATEX-EU-143-1A` | VATEX-EU-143-1A - Exempt based on article 143, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1B` | VATEX-EU-143-1B - Exempt based on article 143, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1C` | VATEX-EU-143-1C - Exempt based on article 143, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1D` | VATEX-EU-143-1D - Exempt based on article 143, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1E` | VATEX-EU-143-1E - Exempt based on article 143, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1F` | VATEX-EU-143-1F - Exempt based on article 143, section 1 (f) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1FA` | VATEX-EU-143-1FA - Exempt based on article 143, section 1 (fa) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1G` | VATEX-EU-143-1G - Exempt based on article 143, section 1 (g) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1H` | VATEX-EU-143-1H - Exempt based on article 143, section 1 (h) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1I` | VATEX-EU-143-1I - Exempt based on article 143, section 1 (i) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1J` | VATEX-EU-143-1J - Exempt based on article 143, section 1 (j) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1K` | VATEX-EU-143-1K - Exempt based on article 143, section 1 (k) of Council Directive 2006/112/EC |
| `VATEX-EU-143-1L` | VATEX-EU-143-1L - Exempt based on article 143, section 1 (l) of Council Directive 2006/112/EC |
| `VATEX-EU-144` | VATEX-EU-144 - Exempt based on article 144 of Council Directive 2006/112/EC |
| `VATEX-EU-146-1E` | VATEX-EU-146-1E - Exempt based on article 146 section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148` | VATEX-EU-148 - Exempt based on article 148 of Council Directive 2006/112/EC |
| `VATEX-EU-148-A` | VATEX-EU-148-A - Exempt based on article 148, section (a) of Council Directive 2006/112/EC |
| `VATEX-EU-148-B` | VATEX-EU-148-B - Exempt based on article 148, section (b) of Council Directive 2006/112/EC |
| `VATEX-EU-148-C` | VATEX-EU-148-C - Exempt based on article 148, section (c) of Council Directive 2006/112/EC |
| `VATEX-EU-148-D` | VATEX-EU-148-D - Exempt based on article 148, section (d) of Council Directive 2006/112/EC |
| `VATEX-EU-148-E` | VATEX-EU-148-E - Exempt based on article 148, section (e) of Council Directive 2006/112/EC |
| `VATEX-EU-148-F` | VATEX-EU-148-F - Exempt based on article 148, section (f) of Council Directive 2006/112/EC |
| `VATEX-EU-148-G` | VATEX-EU-148-G - Exempt based on article 148, section (g) of Council Directive 2006/112/EC |
| `VATEX-EU-151` | VATEX-EU-151 - Exempt based on article 151 of Council Directive 2006/112/EC |
| `VATEX-EU-151-1A` | VATEX-EU-151-1A - Exempt based on article 151, section 1 (a) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1AA` | VATEX-EU-151-1AA - Exempt based on article 151, section 1 (aa) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1B` | VATEX-EU-151-1B - Exempt based on article 151, section 1 (b) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1C` | VATEX-EU-151-1C - Exempt based on article 151, section 1 (c) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1D` | VATEX-EU-151-1D - Exempt based on article 151, section 1 (d) of Council Directive 2006/112/EC |
| `VATEX-EU-151-1E` | VATEX-EU-151-1E - Exempt based on article 151, section 1 (e) of Council Directive 2006/112/EC |
| `VATEX-EU-153` | VATEX-EU-153 - Exempt based on article 153 of Council Directive 2006/112/EC |
| `VATEX-EU-159` | VATEX-EU-159 - Exempt based on article 159 of Council Directive 2006/112/EC |
| `VATEX-EU-309` | VATEX-EU-309 - Exempt based on article 309 of Council Directive 2006/112/EC |
| `VATEX-EU-AE` | VATEX-EU-AE - Reverse charge |
| `VATEX-EU-D` | VATEX-EU-D - Intra-Community acquisition from second hand means of transport |
| `VATEX-EU-F` | VATEX-EU-F - Intra-Community acquisition of second hand goods |
| `VATEX-EU-G` | VATEX-EU-G - Export outside the EU |
| `VATEX-EU-I` | VATEX-EU-I - Intra-Community acquisition of works of art |
| `VATEX-EU-IC` | VATEX-EU-IC - Intra-Community supply |
| `VATEX-EU-O` | VATEX-EU-O - Not subject to VAT |
| `VATEX-EU-J` | VATEX-EU-J - Intra-Community acquisition of collectors items and antiques |
| `VATEX-FR-FRANCHISE` | VATEX-FR-FRANCHISE - France domestic VAT franchise in base |
| `VATEX-FR-CNWVAT` | VATEX-FR-CNWVAT - France domestic Credit Notes without VAT, due to supplier forfeit of VAT for discount |
| `VATEX-FR-CGI261-1` | VATEX-FR-CGI261-1 - Exempt based on 1 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-2` | VATEX-FR-CGI261-2 - Exempt based on 2 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-3` | VATEX-FR-CGI261-3 - Exempt based on 3 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-4` | VATEX-FR-CGI261-4 - Exempt based on 4 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-5` | VATEX-FR-CGI261-5 - Exempt based on 5 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-7` | VATEX-FR-CGI261-7 - Exempt based on 7 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261-8` | VATEX-FR-CGI261-8 - Exempt based on 8 of article 261 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261A` | VATEX-FR-CGI261A - Exempt based on article 261 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261B` | VATEX-FR-CGI261B - Exempt based on article 261 B of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-1` | VATEX-FR-CGI261C-1 - Exempt based on 1° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-2` | VATEX-FR-CGI261C-2 - Exempt based on 2° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261C-3` | VATEX-FR-CGI261C-3 - Exempt based on 3° of article 261 C of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1` | VATEX-FR-CGI261D-1 - Exempt based on 1° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-1BIS` | VATEX-FR-CGI261D-1BIS - Exempt based on 1°bis of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-2` | VATEX-FR-CGI261D-2 - Exempt based on 2° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261D-3` | VATEX-FR-CGI261D-3 - Exempt based on 3° of article 261 D of the Code Général des Impôts (CGI ; General tax code) Exonération de TVA - Article 261 D-3° du Code Général des Impôts |
| `VATEX-FR-CGI261D-4` | VATEX-FR-CGI261D-4 - Exempt based on 4° of article 261 D of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-1` | VATEX-FR-CGI261E-1 - Exempt based on 1° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI261E-2` | VATEX-FR-CGI261E-2 - Exempt based on 2° of article 261 E of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI277A` | VATEX-FR-CGI277A - Exempt based on article 277 A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI275` | VATEX-FR-CGI275 - Exempt based on article 275 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-298SEXDECIESA` | VATEX-FR-298SEXDECIESA - Exempt based on article 298 sexdecies A of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-CGI295` | VATEX-FR-CGI295 - Exempt based on article 295 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-AE` | VATEX-FR-AE - Exempt based on 2 of article 283 of the Code Général des Impôts (CGI ; General tax code) |
| `VATEX-FR-F` | VATEX-FR-F - Second-hand sales |
| `VATEX-FR-I` | VATEX-FR-I - Sales of works of art |
| `VATEX-FR-J` | VATEX-FR-J - Sales of antiques |

### `l10n_ar_type_tax_use` (Argentina Tax Type)

| Value | Label |
|---|---|
| `sale` | Sales |
| `purchase` | Purchases |
| `none` | Other |
| `supplier` | Vendor Payment Withholding |
| `customer` | Customer Payment Withholding |

### `l10n_ar_withholding_payment_type` (Argentina Withholding Payment Type)

| Value | Label |
|---|---|
| `supplier` | Vendor Payment |
| `customer` | Customer Payment |

### `l10n_ar_tax_type` (WTH Tax)

| Value | Label |
|---|---|
| `earnings` | Earnings |
| `earnings_scale` | Earnings Scale |
| `iibb_untaxed` | IIBB Untaxed |
| `iibb_total` | IIBB Total Amount |

### `l10n_ee_kmd_inf_code` (KMD INF Code)

| Value | Label |
|---|---|
| `1` | Sale KMS §41/42 |
| `2` | Sale KMS §41^1 |
| `11` | Purchase KMS §29(4)/30/32 |
| `12` | Purchase KMS §41^1 |

### `l10n_eg_eta_code` (ETA Code (Egypt))

| Value | Label |
|---|---|
| `t1_v001` | T1 - V001 - Export |
| `t1_v002` | T1 - V002 - Export to free areas and other areas |
| `t1_v003` | T1 - V003 - Exempted good or service |
| `t1_v004` | T1 - V004 - A non-taxable good or service |
| `t1_v005` | T1 - V005 - Exemptions for diplomats, consulates and embassies |
| `t1_v006` | T1 - V006 - Defence and National security Exemptions |
| `t1_v007` | T1 - V007 - Agreements exemptions |
| `t1_v008` | T1 - V008 - Special Exemption and other reasons |
| `t1_v009` | T1 - V009 - General Item sales |
| `t1_v010` | T1 - V010 - Other Rates |
| `t2_tbl01` | T2 - Tbl01 - Table tax (percentage) |
| `t3_tbl02` | T3 - Tbl02 - Table tax (Fixed Amount) |
| `t4_w001` | T4 - W001 - Contracting |
| `t4_w002` | T4 - W002 - Supplies |
| `t4_w003` | T4 - W003 - Purchases |
| `t4_w004` | T4 - W004 - Services |
| `t4_w005` | T4 - W005 - Sums paid by the cooperative societies for car transportation to their members |
| `t4_w006` | T4 - W006 - Commission agency & brokerage |
| `t4_w007` | T4 - W007 - Discounts & grants & additional exceptional incentives (smoke, cement companies) |
| `t4_w008` | T4 - W008 - All discounts & grants & commissions (petroleum, telecommunications, and other) |
| `t4_w009` | T4 - W009 - Supporting export subsidies |
| `t4_w010` | T4 - W010 - Professional fees |
| `t4_w011` | T4 - W011 - Commission & brokerage _A_57 |
| `t4_w012` | T4 - W012 - Hospitals collecting from doctors |
| `t4_w013` | T4 - W013 - Royalties |
| `t4_w014` | T4 - W014 - Customs clearance |
| `t4_w015` | T4 - W015 - Exemption |
| `t4_w016` | T4 - W016 - advance payments |
| `t5_st01` | T5 - ST01 - Stamping tax (percentage) |
| `t6_st02` | T6 - ST02 - Stamping Tax (amount) |
| `t7_ent01` | T7 - Ent01 - Entertainment tax (rate) |
| `t7_ent02` | T7 - Ent02 - Entertainment tax (amount) |
| `t8_rd01` | T8 - RD01 - Resource development fee (rate) |
| `t8_rd02` | T8 - RD02 - Resource development fee (amount) |
| `t9_sc01` | T9 - SC01 - Service charges (rate) |
| `t9_sc02` | T9 - SC02 - Service charges (amount) |
| `t10_mn01` | T10 - Mn01 - Municipality Fees (rate) |
| `t10_mn02` | T10 - Mn02 - Municipality Fees (amount) |
| `t11_mi01` | T11 - MI01 - Medical insurance fee (rate) |
| `t11_mi02` | T11 - MI02 - Medical insurance fee (amount) |
| `t12_of01` | T12 - OF01 - Other fees (rate) |
| `t12_of02` | T12 - OF02 - Other fees (amount) |
| `t13_st03` | T13 - ST03 - Stamping tax (percentage) |
| `t14_st04` | T14 - ST04 - Stamping Tax (amount) |
| `t15_ent03` | T15 - Ent03 - Entertainment tax (rate) |
| `t15_ent04` | T15 - Ent04 - Entertainment tax (amount) |
| `t16_rd03` | T16 - RD03 - Resource development fee (rate) |
| `t16_rd04` | T16 - RD04 - Resource development fee (amount) |
| `t17_sc03` | T17 - SC03 - Service charges (rate) |
| `t17_sc04` | T17 - SC04 - Service charges (amount) |
| `t18_mn03` | T18 - Mn03 - Municipality Fees (rate) |
| `t18_mn04` | T18 - Mn04 - Municipality Fees (amount) |
| `t19_mi03` | T19 - MI03 - Medical insurance fee (rate) |
| `t19_mi04` | T19 - MI04 - Medical insurance fee (amount) |
| `t20_of03` | T20 - OF03 - Other fees (rate) |
| `t20_of04` | T20 - OF04 - Other fees (amount) |

### `l10n_es_exempt_reason` (Exempt Reason (Spain))

| Value | Label |
|---|---|
| `E1` | Art. 20 |
| `E2` | Art. 21 |
| `E3` | Art. 22 |
| `E4` | Art. 23 y 24 |
| `E5` | Art. 25 |
| `E6` | Otros |

### `l10n_es_type` (Tax Type (Spain))

| Value | Label |
|---|---|
| `exento` | Exento |
| `sujeto` | Sujeto |
| `sujeto_agricultura` | Sujeto Agricultura |
| `sujeto_isp` | Sujeto ISP |
| `no_sujeto` | No Sujeto |
| `no_sujeto_loc` | No Sujeto por reglas de Localization |
| `no_deducible` | No Deducible |
| `retencion` | Retencion |
| `recargo` | Recargo de Equivalencia |
| `dua` | DUA |
| `ignore` | Ignore even the base amount |

### `l10n_es_edi_facturae_tax_type` (Spanish Facturae electronic data interchange Tax Type)

| Value | Label |
|---|---|
| `01` | Value-Added Tax |
| `02` | Taxes on production, services and imports in Ceuta and Melilla |
| `03` | IGIC: Canaries General Indirect Tax |
| `04` | IRPF: Personal Income Tax |
| `05` | Other |
| `06` | ITPAJD: Tax on wealth transfers and stamp duty |
| `07` | IE: Excise duties and consumption taxes |
| `08` | RA: Customs duties |
| `09` | IGTECM: Sales tax in Ceuta and Melilla |
| `10` | IECDPCAC: Excise duties on oil derivates in Canaries |
| `11` | IIIMAB: Tax on premises that affect the environment in the Balearic Islands |
| `12` | ICIO: Tax on construction, installation and works |
| `13` | IMVDN: Local tax on unoccupied homes in Navarre |
| `14` | IMSN: Local tax on building plots in Navarre |
| `15` | IMGSN: Local sumptuary tax in Navarre |
| `16` | IMPN: Local tax on advertising in Navarre |
| `17` | REIVA: Special VAT for travel agencies |
| `18` | REIGIC: Special IGIC: for travel agencies |
| `19` | REIPSI: Special IPSI for travel agencies |
| `20` | IPS: Insurance premiums Tax |
| `21` | SWUA: Surcharge for Winding Up Activity |
| `22` | IVPEE: Tax on the value of electricity generation |
| `23` | Tax on the production of spent nuclear fuel and radioactive waste from the generation of nuclear electric power |
| `24` | Tax on the storage of spent nuclear energy and radioactive waste in centralised facilities |
| `25` | IDEC: Tax on bank deposits |
| `26` | Excise duty applied to manufactured tobacco in Canaries |
| `27` | IGFEI: Tax on Fluorinated Greenhouse Gases |
| `28` | IRNR: Non-resident Income Tax |
| `29` | Corporation Tax |

### `l10n_es_applicability` (Applicability (Spain))

| Value | Label |
|---|---|
| `01` | VAT |
| `02` | IPSI |
| `03` | IGIC |

### `l10n_in_gst_tax_type` (Localization In Goods and services tax Tax Type)

| Value | Label |
|---|---|
| `igst` | igst |
| `cgst` | cgst |
| `sgst` | sgst |
| `cess` | cess |

### `l10n_in_tax_type` (Indian Tax Type)

| Value | Label |
|---|---|
| `gst` | GST |
| `tcs` | TCS |
| `tds_sale` | TDS Sale |
| `tds_purchase` | TDS Purchase |
| `nil_rated` | Nil Rated |
| `exempt` | Exempt |
| `non_gst` | Non-GST |

### `l10n_it_exempt_reason` (Exoneration)

| Value | Label |
|---|---|
| `N1` | [N1] Escluse ex art. 15 |
| `N2.1` | [N2.1] Non soggette ad IVA ai sensi degli artt. Da 7 a 7-septies del DPR 633/72 |
| `N2.2` | [N2.2] Non soggette - altri casi |
| `N3.1` | [N3.1] Non imponibili - esportazioni |
| `N3.2` | [N3.2] Non imponibili - cessioni intracomunitarie |
| `N3.3` | [N3.3] Non imponibili - cessioni verso San Marino |
| `N3.4` | [N3.4] Non imponibili - operazioni assimilate alle cessioni all'esportazione |
| `N3.5` | [N3.5] Non imponibili - a seguito di dichiarazioni d'intento |
| `N3.6` | [N3.6] Non imponibili - altre operazioni che non concorrono alla formazione del plafond |
| `N4` | [N4] Esenti |
| `N5` | [N5] Regime del margine / IVA non esposta in fattura |
| `N6.1` | [N6.1] Inversione contabile - cessione di rottami e altri materiali di recupero |
| `N6.2` | [N6.2] Inversione contabile - cessione di oro e argento puro |
| `N6.3` | [N6.3] Inversione contabile - subappalto nel settore edile |
| `N6.4` | [N6.4] Inversione contabile - cessione di fabbricati |
| `N6.5` | [N6.5] Inversione contabile - cessione di telefoni cellulari |
| `N6.6` | [N6.6] Inversione contabile - cessione di prodotti elettronici |
| `N6.7` | [N6.7] Inversione contabile - prestazioni comparto edile esettori connessi |
| `N6.8` | [N6.8] Inversione contabile - operazioni settore energetico |
| `N6.9` | [N6.9] Inversione contabile - altri casi |
| `N7` | [N7] IVA assolta in altro stato UE (prestazione di servizi di telecomunicazioni, tele-radiodiffusione ed elettronici ex art. 7-octies, comma 1 lett. a, b, art. 74-sexies DPR 633/72) |

### `l10n_mx_factor_type` (Factor Type)

| Value | Label |
|---|---|
| `Tasa` | Tasa |
| `Cuota` | Cuota |
| `Exento` | Exento |

### `l10n_mx_tax_type` (SAT Tax Type)

| Value | Label |
|---|---|
| `isr` | ISR |
| `iva` | IVA |
| `ieps` | IEPS |
| `local` | Local |

### `l10n_my_tax_type` (Malaysian Tax Type)

| Value | Label |
|---|---|
| `01` | Sales Tax |
| `02` | Service Tax |
| `03` | Tourism Tax |
| `04` | High-Value Goods Tax |
| `05` | Sales Tax on Low Value Goods |
| `06` | Not Applicable |
| `E` | Tax exemption (where applicable) |

### `l10n_pe_edi_tax_code` (Code)

| Value | Label |
|---|---|
| `1000` | IGV - General Sales Tax |
| `1016` | IVAP - Tax on Sale Paddy Rice |
| `2000` | ISC - Selective Excise Tax |
| `7152` | ICBPER - Plastic bag tax |
| `9995` | EXP - Exportation |
| `9996` | GRA - Free |
| `9997` | EXO - Exonerated |
| `9998` | INA - Unaffected |
| `9999` | OTHERS - Other taxes |

### `l10n_pe_edi_unece_category` (UNECE Code)

| Value | Label |
|---|---|
| `E` | Exempt from tax |
| `G` | Free export item, tax not charged |
| `O` | Services outside scope of tax |
| `S` | Standard rate |
| `Z` | Zero rated goods |

### `l10n_pe_edi_isc_type` (ISC Type)

| Value | Label |
|---|---|
| `01` | System to value |
| `02` | Application of the Fixed Amount |
| `03` | Retail Price System |

### `l10n_tw_edi_tax_type` (Ecpay Tax Type)

| Value | Label |
|---|---|
| `1` | Taxable |
| `2` | Zero tax rate |
| `3` | Duty free |
| `4` | Taxable (special tax rate) |

### `l10n_tw_edi_special_tax_type` (Ecpay Special Tax Type)

| Value | Label |
|---|---|
| `1` | Saloons and tea rooms, coffee shops and bars offering companionship services: Tax rate is 25% |
| `2` | Night clubs or restaurants providing entertaining show programs: Tax rate is 15% |
| `3` | Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 2% |
| `4` | The sales amounts from reinsurance premiums shall be taxed at 1% |
| `5` | Banking businesses, insurance businesses, trust investment businesses, securities businesses, futures businesses, commercial paper businesses and pawn-broking businesses: Tax rate is 5% |
| `6` | Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after July 2014): Tax rate is 5% |
| `7` | Core business revenues from the banking and insurance business of the banking and insurance industries (Applicable to sales after June 2014): Tax rate is 5% |
| `8` | Duty free or non-output data |

### `l10n_uy_tax_category` (Tax Category)

| Value | Label |
|---|---|
| `vat` | VAT |

## Operations (154)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_constrains_name` | validation | self | `account` | constrains: `company_id`, `name`, `type_tax_use`, `tax_scope`, `country_id` |  |
| `validate_tax_group_id` | validation | self | `account` | constrains: `tax_group_id` |  |
| `_constrains_cash_basis_transition_account` | validation | self | `account` | constrains: `tax_exigibility`, `cash_basis_transition_account_id` |  |
| `name_search` | operation | self, name, domain, operator, limit | `account` | model; readonly |  |
| `_compute_country_id` | computation | self | `account` | depends: `company_id` |  |
| `_compute_tax_group_id` | computation | self | `account` | depends: `company_id`, `country_id` |  |
| `_compute_price_include` | computation | self | `account` | depends: `price_include_override` |  |
| `_search_price_include` | search rule | self, operator, value | `account` |  |  |
| `_hook_compute_is_used` | internal rule | self, tax_to_compute | `account` |  | -- TO BE REMOVED IN MASTER --  Override to compute the ids of taxes used in other modules. It takes as parameter a set of tax ids. It should return a set containing the ids of the taxes from that input set that are used in transactions. |
| `_compute_is_domestic` | computation | self | `account` | depends: `company_id`, `company_id.domestic_fiscal_position_id`, `fiscal_position_ids` |  |
| `_compute_display_alternative_taxes_field` | computation | self | `account` | depends: `fiscal_position_ids` |  |
| `_compute_is_used` | computation | self | `account`, `hr_expense`, `point_of_sale`, `purchase` | depends: `account_move_line_ids`, `account_reconcile_model_line_ids`; depends: `hr_expense_ids`; depends: `pos_order_line_ids`; depends: `purchase_order_line_ids` |  |
| `_compute_repartition_lines_str` | computation | self | `account` | depends: `is_used`, `repartition_line_ids.account_id`, `repartition_line_ids.sequence`, `repartition_line_ids.factor_percent`, `repartition_line_ids.use_in_tax_closing`, `repartition_line_ids.tag_ids` |  |
| `_message_log_repartition_lines` | messaging hook | self, old_values_str, new_values_str | `account` |  |  |
| `_message_log` | messaging hook | self, **kwargs | `account` |  |  |
| `_compute_invoice_repartition_line_ids` | computation | self | `account` | depends: `company_id` |  |
| `_compute_refund_repartition_line_ids` | computation | self | `account` | depends: `company_id` |  |
| `_compute_has_negative_factor` | computation | self | `account` | depends: `invoice_repartition_line_ids.factor`, `invoice_repartition_line_ids.repartition_type` |  |
| `_parse_name_search` | internal rule | name | `account` |  | Parse the name to search the taxes faster. Technical:  0EUM      => 0%E%U%M             21M       => 2%1%M%   where the % represents 0, 1 or multiple characters in a SQL 'LIKE' search.             21" M"    => 2%1% M%             21" M"co  => 2%1% M%c%o% Examples:   0EUM      => VAT 0% EU M.             21M       => 21% M , 21% EU M, 21% M.Cocont and 21% EX M.             21" M"    => 21% M and 21% M.Cocont.             21" M"co  => 21% M.Cocont. |
| `_search` | search rule | self, domain, *args, **kwargs | `account` | model | Intercept the search on `name` to allow searching more freely on taxes when using `like` or `ilike`. |
| `_check_repartition_lines` | validation | self, lines | `account` |  |  |
| `_validate_repartition_lines` | validation | self | `account` | constrains: `invoice_repartition_line_ids`, `refund_repartition_line_ids`, `repartition_line_ids` |  |
| `_check_children_scope` | validation | self | `account` | constrains: `children_tax_ids`, `type_tax_use` |  |
| `_check_company_consistency` | validation | self | `account` | constrains: `company_id` |  |
| `_sanitize_vals` | internal rule | self, vals | `account` |  | Normalize the create/write values. |
| `create` | lifecycle override | self, vals_list | `account` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `account`, `point_of_sale` |  |  |
| `copy_data` | lifecycle override | self, default | `account` |  |  |
| `_compute_display_name` | computation | self | `account` | depends: `type_tax_use`, `tax_scope`; depends_context: `append_fields`, `formatted_display_name` |  |
| `_compute_tax_label` | computation | self | `account`, `l10n_account_withholding_tax` | depends: `name`, `invoice_label` | On withholding taxes, we do not want the label used in reports to fall back to the tax name. This gives the user the flexibility to display or not withholding taxes as they need. |
| `onchange_amount` | on change | self | `account`, `l10n_sa_edi` | onchange: `amount` |  |
| `onchange_amount_type` | on change | self | `account` | onchange: `amount_type` |  |
| `onchange_price_include` | on change | self | `account` | onchange: `price_include` |  |
| `_eval_taxes_computation_prepare_product_fields` | internal rule | self | `account_tax_python`, `account` |  | Get the fields to create the evaluation context from the product for the taxes computation.  This method is not there in the javascript code. Anybody wanted to use the product during the taxes computation js-side needs to preload the product fields using this method.  [!] Only added python-side.  :return: A set of fields to be extracted from the product to evaluate the taxes computation. |
| `_eval_taxes_computation_prepare_product_default_values` | internal rule | self, field_names | `account` | model | Prepare the default values for the product according the fields passed as parameter. The dictionary contains the default values to be considered if there is no product at all.  This method is not there in the javascript code. Anybody wanted to use the product during the taxes computation js-side needs to preload the default product fields using this method.  [!] Only added python-side.  :param field_names: A set of fields returned by '_eval_taxes_computation_get_product_fields'. :return: A mapping <field_name> => <field_info> where field_info is a dict containing:     * type: the type of the f |
| `_eval_taxes_computation_prepare_product_values` | internal rule | self, default_product_values, product | `account` | model | Convert the product passed as parameter to a dictionary to be passed to '_eval_taxes_computation_prepare_context'.  Note: In javascript, this method is not available. You have to ensure the necessary product fields are well loaded to not break the management of taxes with custom formula byt using '_eval_taxes_computation_prepare_product_fields'.  [!] Only added python-side.  :param default_product_values:  The default product values generated by '_eval_taxes_computation_prepare_product_default_values'. :param product:                 An optional product.product record. :return:                 |
| `_eval_taxes_computation_turn_to_product_values` | internal rule | self, product | `account` |  | Helper purely in Python to call:     '_eval_taxes_computation_prepare_product_fields'     '_eval_taxes_computation_prepare_product_default_values'     '_eval_taxes_computation_prepare_product_values' all at once.  [!] Only added python-side.  :param product: An optional product.product record. :return:        The values representing the product. |
| `_eval_taxes_computation_prepare_product_uom_fields` | internal rule | self | `account_tax_python`, `account` |  | Get the fields to create the evaluation context from the product uom for the taxes computation.  This method is not there in the javascript code. Anybody wanted to use the product during the taxes computation js-side needs to preload the product uom fields using this method.  [!] Only added python-side.  :return: A set of fields to be extracted from the product to evaluate the taxes computation. |
| `_eval_taxes_computation_prepare_product_uom_default_values` | internal rule | self, field_names | `account` | model | Prepare the default values for the product uom according the fields passed as parameter. The dictionary contains the default values to be considered if there is no product uom at all.  This method is not there in the javascript code. Anybody wanted to use the product uom during the taxes computation js-side needs to preload the default product uom fields using this method.  [!] Only added python-side.  :param field_names: A set of fields returned by '_eval_taxes_computation_get_product_uom_fields'. :return: A mapping <field_name> => <field_info> where field_info is a dict containing:     * typ |
| `_eval_taxes_computation_prepare_product_uom_values` | internal rule | self, default_product_uom_values, product_uom | `account` | model | Convert the product uom passed as parameter to a dictionary to be passed to '_eval_taxes_computation_prepare_context'.  Note: In javascript, this method takes an additional parameter being the results of the '_eval_taxes_computation_prepare_product_uom_default_values' method because this method is not callable in javascript but must be passed to the client instead.  [!] Only added python-side.  :param default_product_uom_values:  The default product values generated by '_eval_taxes_computation_prepare_product_uom_default_values'. :param product_uom:                 An optional product.uom reco |
| `_eval_taxes_computation_turn_to_product_uom_values` | internal rule | self, product_uom | `account` |  | Helper purely in Python to call:     '_eval_taxes_computation_prepare_product_uom_fields'     '_eval_taxes_computation_prepare_product_uom_default_values'     '_eval_taxes_computation_prepare_product_uom_values' all at once.  [!] Only added python-side.  :param product_uom: An optional product.uom record. :return:            The values representing the product uom. |
| `_flatten_taxes_and_sort_them` | internal rule | self | `account` |  | Flattens the taxes contained in this recordset, returning all the children at the bottom of the hierarchy, in a recordset, ordered by sequence.   Eg. considering letters as taxes and alphabetic order as sequence :   [G, B([A, D, F]), E, C] will be computed as [A, D, F, C, E, G]  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :return: A tuple <sorted_taxes, group_per_tax> where:     - sorted_taxes is a recordset of taxes.     - group_per_tax maps each tax to its parent group of taxes if exists. |
| `_batch_for_taxes_computation` | internal rule | self, special_mode, filter_tax_function | `account` |  | Group the current taxes all together like price-included percent taxes or division taxes.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param special_mode:        The special mode of the taxes computation: False, 'total_excluded' or 'total_included'. :param filter_tax_function: Optional function to filter out some taxes from the computation. :return: A dictionary containing:     * batch_per_tax: A mapping of each tax to its batch.     * group_per_tax: A mapping of each tax retrieved from a group of taxes.     * sorted_taxes: A recordset  |
| `_propagate_extra_taxes_base` | internal rule | self, tax, taxes_data, special_mode | `account` |  | In some cases, depending the computation order of taxes, the special_mode or the configuration of taxes (price included, affect base of subsequent taxes, etc), some taxes need to affect the base and the tax amount of the others. That's the purpose of this method: adding which tax need to be added as an 'extra_base' to the others.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param tax:             The tax for which we need to propagate the tax. :param taxes_data:      The computed values for taxes so far. :param special_mode:    The spec |
| `_eval_tax_amount_fixed_amount` | internal rule | self, batch, raw_base, evaluation_context | `account_tax_python`, `account` |  | Eval the tax amount for a single tax during the first ascending order for fixed taxes.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param batch:               The batch of taxes containing this tax. :param raw_base:            The base on which the tax should be computed. :param evaluation_context:  The context containing all relevant info to compute the tax. :return:                    The tax amount or None if it has be evaluated later. |
| `_eval_tax_amount_price_included` | internal rule | self, batch, raw_base, evaluation_context | `account` |  | Eval the tax amount for a single tax during the descending order for price-included taxes.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param batch:               The batch of taxes containing this tax. :param raw_base:            The base on which the tax should be computed. :param evaluation_context:  The context containing all relevant info to compute the tax. :return:                    The tax amount. |
| `_eval_tax_amount_price_excluded` | internal rule | self, batch, raw_base, evaluation_context | `account` |  | Eval the tax amount for a single tax during the second ascending order for price-excluded taxes.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param batch:               The batch of taxes containing this tax. :param raw_base:            The base on which the tax should be computed. :param evaluation_context:  The context containing all relevant info to compute the tax. :return:                    The tax amount. |
| `_get_tax_details` | preparation rule | self, price_unit, quantity, precision_rounding, rounding_method, product, product_uom, special_mode, manual_tax_amounts, filter_tax_function | `account` |  | Compute the tax/base amounts for the current taxes.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param price_unit:          The price unit of the line. :param quantity:            The quantity of the line. :param precision_rounding:  The rounding precision for the 'round_per_line' method. :param rounding_method:     'round_per_line' or 'round_globally'. :param product:             The product of the line. :param product_uom:         The product uom of the line. :param special_mode:        Indicate a special mode for the taxes computatio |
| `_adapt_price_unit_to_another_taxes` | internal rule | self, price_unit, product, original_taxes, new_taxes, product_uom | `account` | model | From the price unit and taxes given as parameter, compute a new price unit corresponding to the new taxes.  For example, from price_unit=106 and taxes=[6% tax-included], this method can compute a price_unit=121 if new_taxes=[21% tax-included].  The price_unit is only adapted when all taxes in 'original_taxes' are price-included even when 'new_taxes' contains price-included taxes. This is made that way for the following example:  Suppose a fiscal position B2C mapping 15% tax-excluded => 6% tax-included. If price_unit=100 with [15% tax-excluded], the price_unit is computed as 100 / 1.06 instead  |
| `_export_base_line_extra_tax_data` | internal rule | self, base_line | `account` | model | Export the extra values about the taxes engine into the extra_tax_data json field.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_line: A base line generated by '_prepare_base_line_for_taxes_computation'. :return: A dictionary to be stored into the 'extra_tax_data' field. |
| `_import_base_line_extra_tax_data` | internal rule | self, base_line, extra_tax_data | `account` | model | Import the 'extra_tax_data' json value into the base line passed as parameter. For 'manual_tax_amounts', if the setup of the base line has been manually edited, we don't import the custom tax amounts from 'extra_tax_data.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_line:       A base line generated by '_prepare_base_line_for_taxes_computation'. :param extra_tax_data:  The value stored in one of the 'extra_tax_data' field. :return:                The values to be added into the base line. |
| `_reverse_quantity_base_line_extra_tax_data` | internal rule | self, extra_tax_data | `account` | model | Reverse all sign in extra_tax_data using the quantity.  [!] Only added python-side.  :param extra_tax_data: The manual taxes data stored on records. :return: The extra_tax_data but reversed. |
| `_turn_base_line_is_refund_flag_off` | internal rule | self, base_line | `account` |  | Reverse the sign of the quantity plus all data in tax details.  [!] Only added python-side.  :param base_line: The base_line. :return: The base_line that is no longer a refund line. |
| `_turn_base_lines_is_refund_flag_off` | internal rule | self, base_lines | `account` | model | Reverse the sign of the quantity plus all data in tax details.  [!] Only added python-side.  :param base_lines: The base_lines. :return: The base_lines that is no longer a refund lines. |
| `_get_base_line_field_value_from_record` | preparation rule | self, record, field, extra_values, fallback, from_base_line | `account` | model | Helper to extract a default value for a record or something looking like a record.  Suppose field is 'product_id' and fallback is 'self.env['product.product']'  if record is an account.move.line, the returned product_id will be `record.product_id._origin`. if record is a dict, the returned product_id will be `record.get('product_id', fallback)`.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param record:          A record or a dict or a falsy value. :param field:           The name of the field to extract. :param extra_values:    The ext |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self, record, **kwargs | `account`, `hr_expense`, `l10n_in` | model | Convert any representation of a business object ('record') into a base line being a python dictionary that will be used to use the generic helpers for the taxes computation.  The whole method is designed to ease the conversion from a business record. For example, when passing either account.move.line, either sale.order.line or purchase.order.line, providing explicitely a 'product_id' in kwargs is not necessary since all those records already have an `product_id` field.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param record:  A repres |
| `_prepare_tax_line_for_taxes_computation` | preparation rule | self, record, **kwargs | `account`, `hr_expense` | model | Convert any representation of an accounting tax line ('record') into a python dictionary that will be used to use by `_prepare_tax_lines` to detect which tax line could be updated, the ones to be created and the ones to be deleted. We can't use directly an account.move.line because this is also used by - expense (to create the journal entry) - the bank reconciliation widget All fields in this list are the same as the corresponding fields defined in account.move.line.  The mechanism is the same as '_prepare_base_line_for_taxes_computation'.  [!] Only added python-side.  :param record:  A repres |
| `_add_tax_details_in_base_line` | internal rule | self, base_line, company, rounding_method | `account`, `l10n_account_withholding_tax` | model | Perform the taxes computation for the base line and add it to the base line under the 'tax_details' key. Those values are rounded or not depending of the tax calculation method. If you need to compute monetary fields with that, you probably need to call '_round_base_lines_tax_details' after this method.  The added tax_details is a dictionary containing: raw_total_excluded_currency:    The total without tax expressed in foreign currency. raw_total_excluded:             The total without tax expressed in local currency. raw_total_included_currency:    The total tax included expressed in foreign  |
| `_add_tax_details_in_base_lines` | internal rule | self, base_lines, company | `account` | model | Shortcut to call '_add_tax_details_in_base_line' on multiple base lines at once.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:  A list of base lines. :param company:     The company owning the base lines. |
| `_normalize_target_factors` | internal rule | self, target_factors | `account` | model | Normalize the factors passed as parameter to have a distribution having a sum of 1.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param target_factors:      A list of dictionary containing at least 'factor' being the weight                             defining how much delta will be allocated to this factor. :return:                    A list of tuple <index, normalized_factor> for each 'target_factors' passed as parameter. |
| `_distribute_delta_amount_smoothly` | internal rule | self, precision_digits, delta_amount, target_factors | `account` | model | Distribute 'delta_amount' across the factors passed as parameter.  For example, if 'delta_amount' = 0.03 and precision_digits is 3 and target factors is a list of 3 factors: a) {'factor': 0.4} b) {'factor': 0.3} c) {'factor': 0.3} ... it means the delta will be distributed first on a) then b) then c). Since precision_digits = 3, it means we have a delta of "30" tenth of a hundred to be distributed. a) will take 30 * 0.4 = 12 units. b & c) will take 30 * 0.3 = 9 units each. The result of this method will be [0.012, 0.009, 0.009].  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH M |
| `_round_tax_details_tax_amounts` | internal rule | self, base_lines, company, mode | `account`, `l10n_pt` | model | Dispatch the delta in term of tax amounts across the tax details when dealing with the 'round_globally' method. Suppose 2 lines: - quantity=12.12, price_unit=12.12, tax=23% - quantity=12.12, price_unit=12.12, tax=23% The tax of each line is computed as round(12.12 * 12.12 * 0.23) = 33.79 The expected tax amount of the whole document is round(12.12 * 12.12 * 0.23 * 2) = 67.57 The delta in term of tax amount is 67.57 - 33.79 - 33.79 = -0.01  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines genera |
| `_round_tax_details_base_lines` | internal rule | self, base_lines, company, mode | `account`, `l10n_pt` | model | Additional global rounding depending on if the taxes are included or excluded in price.  This method does not modify the rounding in `taxes_data`, rather it computes an adjustment for `tax_details['total_excluded{_currency}']` and stores it as `tax_details['delta_total_excluded{_currency}']`.  Suppose all taxes are price-included. Suppose two price-included taxes of 10%. Suppose a line having price_unit=100.0. The tax amount is computed as 100.0 / 1.2 * 0.1 = 8.333333333 The base amount is computed as 100.0 - (2 * 8.333333333) = 83.333333334 Without doing anything, we end up with a base of 83. |
| `_round_tax_details_tax_amounts_from_tax_lines` | internal rule | self, base_lines, company, tax_lines | `account` | model | If tax lines are provided, the totals will be aggregated according them. At this point, everything is rounded and won't change anymore.  [!] Only added python-side.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company owning the base lines. :param tax_lines:           A optional list of base lines generated using the '_prepare_tax_line_for_taxes_computation'                             method. If specified, the tax amounts will be computed based on those existing tax lines.               |
| `_round_base_lines_tax_details` | internal rule | self, base_lines, company, tax_lines | `account` | model | Round the 'tax_details' added to base_lines with the '_add_accounting_data_to_base_line_tax_details'. This method performs all the rounding and take care of rounding issues that could appear when using the 'round_globally' tax computation method, specially if some price included taxes are involved.  This method copies all float prefixed with 'raw_' in the tax_details to the corresponding float without 'raw_'. In almost all countries, the round globally should be the tax computation method. When there is an EDI, we need the raw amounts to be reported with more decimals (usually 6 to 8). So if y |
| `_prepare_base_line_grouping_key` | preparation rule | self, base_line | `account`, `hr_expense` | model | Used by '_prepare_tax_lines' to build the accounting grouping key to generate the tax lines. This method takes all relevant fields from the base line that will be used to build the grouping_key.  [!] Only added python-side.  :param base_line: A base line generated by '_prepare_base_line_for_taxes_computation'. :return: The grouping key to generate the tax line for a single base line. |
| `_prepare_base_line_tax_repartition_grouping_key` | preparation rule | self, base_line, base_line_grouping_key, tax_data, tax_rep_data | `account`, `l10n_ar_withholding` | model | Used by '_prepare_tax_lines' to build the accounting grouping key to generate the tax lines. This method adds all relevant fields from a single tax data to the grouping key.  [!] Only added python-side.  :param base_line:               A base line generated by '_prepare_base_line_for_taxes_computation'. :param base_line_grouping_key:  The grouping key created by '_prepare_base_line_grouping_key'. :param tax_data:                One of the tax data in base_line['tax_details']['taxes_data']. :param tax_rep_data:            One of the tax repartition data in tax_data['tax_reps_data']. :return: Th |
| `_prepare_tax_line_repartition_grouping_key` | preparation rule | self, tax_line | `account`, `hr_expense` | model | Used by '_prepare_tax_lines' to build the accounting grouping key to know if the tax line could be updated or not when recomputing the tax lines. Take care this method should remain consistent regarding the grouping key built from the base line.  [!] Only added python-side.  :param tax_line: A tax line generated by '_prepare_tax_line_for_taxes_computation'. :return: The grouping key for the tax line passed as parameter. |
| `_add_accounting_data_to_base_line_tax_details` | internal rule | self, base_line, company, include_caba_tags | `account` | model | Add all informations about repartition lines to base_line['tax_details']['taxes_data'].  Considering a single tax_data, this method adds 'tax_reps_data', being a list of python dictionaries containing:     tax_rep:                The account.tax.repartition.line record.     tax_amount_currency:    The tax amount expressed in foreign currency.     tax_amount:             The tax amount expressed in local currency.     account:                The accounting account record to consider for this tax repartition line.     taxes:                  The taxes to be set on the tax line if the tax affects |
| `_add_accounting_data_in_base_lines_tax_details` | internal rule | self, base_lines, company, include_caba_tags | `account` | model | Shortcut to call '_add_accounting_data_to_base_line_tax_details' on multiple base lines at once.  [!] Only added python-side.  :param base_lines:          A list of base lines. :param company:             The company owning the base lines. :param include_caba_tags:   Indicate if the cash basis tags need to be taken into account. |
| `_aggregate_base_line_tax_details` | internal rule | self, base_line, grouping_function | `account` | model | Aggregate the tax details for a single line according a custom grouping function passed as parameter. This method is mainly use for EDI to report some data per line. Most of the time, having the amounts grouped by tax is not enough because some details should be excluded, aggregated together or just moved into a separated section having another grouping key.  In case the base_line has no tax, the grouping_function is called with an empty tax_data to get the grouping key for the line.  Don't forget to call '_add_tax_details_in_base_lines' and '_round_base_lines_tax_details' before calling this  |
| `_aggregate_base_lines_tax_details` | internal rule | self, base_lines, grouping_function | `account` | model | Shortcut to call '_aggregate_base_line_tax_details' on multiple base lines at once.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines. :param grouping_function:   See '_aggregate_base_line_tax_details'. :return                     A list of tuple <base_line, results> that associates the result of                             '_aggregate_base_line_tax_details' for each base line independently. |
| `_aggregate_base_lines_aggregated_values` | internal rule | self, base_lines_aggregated_values | `account` | model | Aggregate the values returned by '_aggregate_base_lines_tax_details' for the whole document. Most of the time, in EDI, you have to report unrounded amounts for each base line first and then, you need to report all rounded amounts for the whole business document.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines_aggregated_values:    The result of '_aggregate_base_lines_tax_details'. :return: A mapping <grouping_key, amounts> where:     grouping_key                is the grouping_key returned by the 'grouping_function' or 'No |
| `_get_tax_totals_summary` | preparation rule | self, base_lines, currency, company, cash_rounding | `account` | model | Compute the tax totals details for the business documents.  Don't forget to call '_add_tax_details_in_base_lines' and '_round_base_lines_tax_details' before calling this method.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param currency:            The tax totals is only available when all base lines share the same currency.                             Since the tax totals can be computed when there is no base line at |
| `_exclude_tax_groups_from_tax_totals_summary` | internal rule | self, tax_totals, ids_to_exclude | `account` | model | Helper to post-process the tax totals and wrap some tax groups into the base amount. It's used in some localizations to exclude some taxes from the details.  [!] Only added python-side.  :param tax_totals:          The tax totals generated by '_get_tax_totals_summary'. :param ids_to_exclude:      The ids of the tax groups to exclude. :return:                    A new tax totals without the excluded ids. |
| `_prepare_tax_lines` | preparation rule | self, base_lines, company, tax_lines | `account` | model | Prepare the tax journal items for the base lines.  After calling '_add_tax_details_in_base_lines', the tax details is there on base lines. After calling '_round_base_lines_tax_details', the tax details is now rounded. After calling '_add_accounting_data_in_base_lines_tax_details', each tax_data in the tax details contains all accounting informations about the repartition lines.  When calling this method, all 'tax_reps_data' in each 'tax_data' will be aggregated all together and rounded. The total tax amount will not change whatever the number of involved accounting grouping keys. The 'sign' va |
| `_can_be_discounted` | internal rule | self | `account` |  | Detect if a tax is affected by the discount.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :return: A boolean. |
| `_merge_tax_details` | internal rule | self, tax_details_1, tax_details_2 | `account` | model | Helper merging 2 tax details together coming from base lines.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param tax_details_1: First tax details. :param tax_details_2: Second tax details. :return: A new tax details combining the 2 passed as parameter. |
| `_fix_base_lines_tax_details_on_manual_tax_amounts` | internal rule | self, base_lines, company, filter_function | `account` | model | Store the tax details into manual_tax_amounts to fix the results.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:      A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:         The company owning the base lines. :param filter_function: An optional function taking <base_line, tax_data> as parameter and telling which tax will have                         its amounts stored. |
| `_split_tax_data` | internal rule | self, base_line, tax_data, company, target_factors | `account` | model | Split a 'tax_data' in pieces according the factors passed as parameter. This method makes sure no amount is lost or gained during the process.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_line:       A base line. :param tax_data:        The 'tax_data' to split. :param company:         The company owning the base lines. :param target_factors:  A list of dictionary containing at least 'factor' being the weight                         defining how much delta will be allocated to this factor. :return                 A list of 'ta |
| `_split_tax_details` | internal rule | self, base_line, company, target_factors | `account` | model | Split the 'tax_details' in pieces according the factors passed as parameter. This method makes sure no amount is lost or gained during the process.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_line:       A base line. :param company:         The company owning the base lines. :param target_factors:  A list of dictionary containing at least 'factor' being the weight                         defining how much delta will be allocated to this factor. :return                 A list of 'tax_details' having the same size as 'target_f |
| `_split_base_line` | internal rule | self, base_line, company, target_factors, populate_function | `account` | model | Split a base lines into multiple ones. When computing taxes, the results should be exactly the same with a single base_line or after the split.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_line:           A base line. :param company:             The company owning the base line. :param target_factors:      A list of dictionary containing at least 'factor' being the weight                             defining how much delta will be allocated to this factor. :param populate_function:   An optional method to change the parameter |
| `_compute_subset_base_lines_total` | computation | self, base_lines, company | `account` | model | Compute the total of the lines passed as parameter.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  DEPRECATED: TO BE REMOVED IN MASTER  :param base_lines:  A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:     The company owning the base lines. :return: The total. |
| `_reduce_base_lines_with_grouping_function` | internal rule | self, base_lines, grouping_function, aggregate_function, computation_key | `account` | model | Create the new base lines that will get the discount. Since they no longer contain fixed taxes, we can remove the quantity and aggregate them depending on the grouping_function passed as parameter.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          The base lines to be aggregated. :param grouping_function:   An optional function taking a base line as parameter and returning a grouping key                             being the way the base lines will be aggregated all together.                             By default, |
| `_apply_base_lines_manual_amounts_to_reach` | internal rule | self, base_lines, company, target_base_amount_currency, target_base_amount, target_tax_amounts_mapping | `account` | model | Fix the tax amounts of the base lines passed as parameter by storing them in 'manual_tax_amounts' and make some adjustement to ensure the total of those lines will be exactly 'target_amount_currency'/'target_amount'.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  DEPRECATED: TO BE REMOVED IN MASTER  :param base_lines:                  A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:                     The company owning the base lines. :param target_base_amount_currency: The expec |
| `_reduce_base_lines_to_target_amount` | internal rule | self, base_lines, company, amount_type, amount, computation_key, grouping_function, aggregate_function | `account` | model | :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param amount_type:         'fixed' or 'percent' indicating the type of the down payment. :param amount:              The amount of the down payment in case of 'fixed' amount_type. Otherwise, a percentage [0-100]. :param computation_key:     The key that will be used to split the base lines to round the tax amounts. :param grouping_function:   An optional function taking a base line as parameter and returning a groupi |
| `_partition_base_lines_taxes` | internal rule | self, base_lines, partition_function | `account` | model | Partition the taxes of base lines passed as parameter.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:              The base lines. :param partition_function:      A function taking <base_line, tax_data> as parameter and returning                                 True if the tax has to be kept or not. :return:                        A tuple <base_lines_partition_taxes, has_taxes_to_exclude> where     * base_lines_partition_taxes:   A list of tuple <base_line, taxes_to_keep, taxes_to_exclude>     * has_taxes_to_exclude:     |
| `_prepare_discountable_base_lines` | preparation rule | self, base_lines, company, exclude_function | `account` | model | Prepare base lines on which we can compute all kind of discount. This method remove all part of base lines / taxes that are not eligible for a discount. Those taxes are given by the '_can_be_discounted' method giving False if not discountable.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param exclude_function:    An optional function taking a base line and a t |
| `_prepare_global_discount_lines` | preparation rule | self, base_lines, company, amount_type, amount, computation_key, grouping_function | `account` | model | Prepare negative lines to be added representing a global discount.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param amount_type:         'fixed' or 'percent' indicating the type of the discount. :param amount:              The amount to be discounted in case of 'fixed' amount_type. Otherwise, a percentage [0-100]. :param computation_key:     The key that will |
| `_prepare_base_lines_for_down_payment` | preparation rule | self, base_lines, company, exclude_function | `account` | model | Prepare base lines on which we can compute down payments. This method wrap all part of base lines / taxes that are not eligible for a down payment into the base amount.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param exclude_function:    An optional function taking a base line and a tax_data as parameter and returning                             a boolean indicating if the tax_data has to be exclude from the computation. :return:                    The nega |
| `_prepare_down_payment_lines` | preparation rule | self, base_lines, company, amount_type, amount, computation_key, grouping_function | `account` | model | Prepare the base lines to be added representing a down payment.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param amount_type:         'fixed' or 'percent' indicating the type of the down payment. :param amount:              The amount of the down payment in case of 'fixed' amount_type. Otherwise, a percentage [0-100]. :param computation_key:     The key that  |
| `_dispatch_taxes_into_new_base_lines` | internal rule | self, base_lines, company, exclude_function | `account` | model | Extract taxes from base lines and turn them into sub-base lines.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company of the base lines. :param exclude_function:    A function taking a base line and a tax_data as parameter and returning                             a boolean indicating if the tax_data has to be exclude or not. :return:                    The new base lines with some extra  |
| `_turn_removed_taxes_into_new_base_lines` | internal rule | self, base_lines, company, grouping_function, aggregate_function | `account` | model | Merge the sub 'removed_taxes_data_base_lines' generated by '_dispatch_taxes_into_new_base_lines' into the parent line.  [!] Only added python-side.  :param base_lines:          A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:             The company owning the base lines. :param grouping_function:   An optional function taking a base line as parameter and returning a grouping key                             being the way the base lines will be aggregated all together.                             By default, the base lines will be aggre |
| `_dispatch_global_discount_lines` | internal rule | self, base_lines, company | `account` | model | Dispatch the global discount lines present inside the base_lines passed as parameter across the others under the 'discount_base_lines' key.  [!] Only added python-side.  :param base_lines:  A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:     The company owning the base lines. :return:            New base lines without any global discount but sub-lines added under the 'discount_base_lines' key. |
| `_squash_global_discount_lines` | internal rule | self, base_lines, company | `account` | model | Merge the sub global discount base lines generated by '_dispatch_global_discount_lines' into the parent line.  [!] Only added python-side.  :param base_lines:  A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:     The company owning the base lines. |
| `_dispatch_return_of_merchandise_lines` | internal rule | self, base_lines, company | `account` | model | Dispatch the return of merchandise lines present inside the base_lines passed as parameter across the others under the 'return_of_merchandise_base_lines' key. What we call a return of merchandise is when the negative line matches exactly the parent line but has a negative quantity. So if you have 2 base lines, one with a quantity of 3 and the other with a quantity of -1, this method tries to reduce the quantity instead of considering the negative lines as a discount.  [!] Only added python-side.  :param base_lines:  A list of base lines generated using the '_prepare_base_line_for_taxes_computa |
| `_squash_return_of_merchandise_lines` | internal rule | self, base_lines, company | `account` | model | Merge the sub return of merchandise base lines generated by '_dispatch_return_of_merchandise_lines' into the parent line.  [!] Only added python-side.  :param base_lines:  A list of base lines generated using the '_prepare_base_line_for_taxes_computation' method. :param company:     The company owning the base lines. |
| `_get_delta_amount_to_reach_target` | preparation rule | self, target_amount, target_currency, raw_current_amount, raw_current_amount_precision_digits | `account` | model | Get the minimum missing amount having 'raw_current_amount_precision_digits' as precision to be added to 'raw_current_amount' to give 'target_amount' after rounding using 'target_currency'.  :param target_amount:                       The amount to reach after rounding the raw amount using 'target_currency'. :param target_currency:                     The currency used to round 'target_amount'. :param raw_current_amount:                  The raw amount that needs to reach 'target_amount'. :param raw_current_amount_precision_digits: The precision of the delta returned by this method. :return:    |
| `_round_raw_total_excluded` | internal rule | self, base_lines, company, precision_digits, apply_strict_tolerance, in_foreign_currency | `account` | model | Round 'raw_total_excluded[_currency]' according 'precision_digits'.  :param base_lines:              A list of python dictionaries created using the '_prepare_base_line_for_taxes_computation' method. :param company:                 The company owning the base lines. :param precision_digits:        The precision to be used to round. :param apply_strict_tolerance:  A flag ensuring a strict equality between rounded and raw amounts such as                                     ROUND(SUM(raw_total_excluded FOREACH base_line), precision_digits)                                     and SUM(total_exclude |
| `_get_gross_total_without_tax` | preparation rule | self, base_line, company, in_foreign_currency, account_discount_base_lines, precision_digits | `account` | model | Infer the gross total without tax from the base line.  :param base_line:                   A base line (see '_prepare_base_line_for_taxes_computation'). :param company:                     The company owning the base line. :param in_foreign_currency:         True if to be applied on amounts expressed in foreign currency,                                     False for amounts expressed in company currency. :param account_discount_base_lines: Account the distributed global discount in 'discount_base_lines'                                     using '_dispatch_global_discount_lines' in 'raw_discoun |
| `_get_price_unit_without_tax` | preparation rule | self, base_line, company, raw_gross_total_excluded, in_foreign_currency, precision_digits | `account` | model | Infer the gross price unit without tax from the base line.  :param base_line:                   A base line (see '_prepare_base_line_for_taxes_computation'). :param company:                     The company owning the base line. :param raw_gross_total_excluded:    The gross total without tax. :param in_foreign_currency:         True if to be applied on amounts expressed in foreign currency,                                     False for amounts expressed in company currency. :param precision_digits:            The precision to be used to round. :return:                            The gross price |
| `_get_discount_amount_without_tax` | preparation rule | self, base_line, company, raw_gross_total_excluded, in_foreign_currency, precision_digits | `account` | model | Infer the discount amount without tax from the base line.  :param base_line:                   A base line (see '_prepare_base_line_for_taxes_computation'). :param company:                     The company owning the base line. :param raw_gross_total_excluded:    The gross total without tax. :param in_foreign_currency:         True if to be applied on amounts expressed in foreign currency,                                     False for amounts expressed in company currency. :param precision_digits:            The precision to be used to round. :return:                            The discount amo |
| `_add_and_round_raw_gross_total_excluded_and_discount` | internal rule | self, base_lines, company, precision_digits, apply_strict_tolerance, in_foreign_currency, account_discount_base_lines | `account` | model | Compute and add 'raw_gross_total_excluded[_currency]' / 'raw_gross_price_unit[_currency]' / 'raw_discount_amount[_currency]' to the tax details according 'precision_digits' / 'in_foreign_currency'.  :param base_lines:                  A list of python dictionaries created using the '_prepare_base_line_for_taxes_computation' method. :param company:                     The company owning the base lines. :param precision_digits:            The precision to be used to round. :param apply_strict_tolerance:      A flag ensuring a strict equality between rounded and raw amounts such as                |
| `_round_raw_gross_total_excluded_and_discount` | internal rule | self, base_lines, company, in_foreign_currency | `account` | model |  |
| `_round_raw_tax_amounts` | internal rule | self, base_lines_aggregated_values, company, precision_digits, apply_strict_tolerance, in_foreign_currency | `account` | model | Round 'raw_tax_amount[_currency]'/'raw_base_amount[_currency]' according 'precision_digits' / 'in_foreign_currency'.  :param base_lines_aggregated_values:    The result of '_aggregate_base_lines_tax_details'. :param company:                         The company owning the base lines. :param precision_digits:                The precision to be used to round. :param apply_strict_tolerance:          A flag ensuring a strict equality between rounded and raw amounts such as                                             ROUND(SUM(raw_tax_amount FOREACH base_line), precision_digits)                      |
| `flatten_taxes_hierarchy` | operation | self | `account` |  |  |
| `get_tax_tags` | operation | self, is_refund, repartition_type | `account` |  |  |
| `compute_all` | operation | self, price_unit, currency, quantity, product, partner, is_refund, handle_price_include, include_caba_tags, rounding_method | `account` |  | Compute all information required to apply taxes (in self + their children in case of a tax group). We consider the sequence of the parent for group of taxes. Eg. considering letters as taxes and alphabetic order as sequence::      [G, B([A, D, F]), E, C] will be computed as [A, D, F, C, E, G]  :param price_unit: The unit price of the line to compute taxes on. :param currency: The optional currency in which the price_unit is expressed. :param quantity: The optional quantity of the product to compute taxes on. :param product: The optional product to compute taxes on.     Used to get the tags to  |
| `_filter_taxes_by_company` | internal rule | self, company_id | `account` |  | Filter taxes by the given company It goes through the company hierarchy until a tax is found |
| `_fix_tax_included_price` | internal rule | self, price, prod_taxes, line_taxes | `account` | model | Subtract tax amount from price when corresponding "price included" taxes do not apply |
| `_fix_tax_included_price_company` | internal rule | self, price, prod_taxes, line_taxes, company_id | `account` | model |  |
| `_get_description_plaintext` | preparation rule | self | `account` |  |  |
| `unlink_except_tax_used` | operation | self | `account` | ondelete |  |
| `_import_retrieve_tax_from_account_default_tax` | internal rule | self, tax_values | `account` | model |  |
| `_import_retrieve_tax_from_invoice_predictive` | internal rule | self, tax_values | `account` | model |  |
| `_import_retrieve_tax_from_price_include_exclude` | internal rule | self, tax_values | `account` | model |  |
| `_import_retrieve_tax` | internal rule | self, search_plan, company, tax_values_list | `account` | model |  |
| `_compute_ubl_cii_requires_exemption_reason` | computation | self | `account_edi_ubl_cii` | depends: `ubl_cii_tax_category_code` |  |
| `_onchange_ubl_cii_tax_category_code` | on change | self | `account_edi_ubl_cii` | onchange: `ubl_cii_requires_exemption_reason` |  |
| `_check_amount_type_code_formula` | validation | self | `account_tax_python` | constrains: `amount_type`, `formula` |  |
| `_compute_formula_decoded_info` | computation | self | `account_tax_python` | depends: `formula` |  |
| `_check_and_normalize_formula` | validation | self, formula | `account_tax_python` | model | Check the formula is passing the minimum check to ensure the compatibility between both evaluation in python & javascript. |
| `_eval_tax_amount_formula` | internal rule | self, raw_base, evaluation_context | `account_tax_python` |  | Evaluate the formula of the tax passed as parameter.  [!] Mirror of the same method in account_tax.js. PLZ KEEP BOTH METHODS CONSISTENT WITH EACH OTHERS.  :param raw_base: :param evaluation_context:  The context created by '_eval_taxes_computation_prepare_context'. :return:                    The tax base amount. |
| `_onchange_is_withholding_tax_on_payment` | on change | self | `l10n_account_withholding_tax` | onchange: `is_withholding_tax_on_payment` | Ensure that we don't keep cash basis enabled if it was before checking the withholding tax option. |
| `_onchange_amount` | on change | self | `l10n_account_withholding_tax` | onchange: `amount` | Reset the is_withholding_tax_on_payment field when the amount is set to positive; as the field will be hidden. |
| `_check_amount_type` | validation | self | `l10n_account_withholding_tax` | constrains: `amount_type`, `is_withholding_tax_on_payment` | The computation of withholding taxes needs to be limited in computation types to ensure that it works as expected. |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_account_withholding_tax_pos`, `l10n_in_pos`, `l10n_tw_edi_ecpay_pos`, `point_of_sale`, `pos_account_tax_python` | model |  |
| `_compute_l10n_ar_type_tax_use` | computation | self | `l10n_ar_withholding` | depends: `type_tax_use`, `l10n_ar_withholding_payment_type` |  |
| `_inverse_l10n_ar_type_tax_use` | on change | self | `l10n_ar_withholding` | onchange: `l10n_ar_type_tax_use` |  |
| `_l10n_es_get_regime_code` | internal rule | self | `l10n_es` |  |  |
| `_l10n_es_get_sujeto_tax_types` | internal rule | self | `l10n_es` | model |  |
| `_l10n_es_get_main_tax_types` | internal rule | self | `l10n_es` | model |  |
| `_l10n_es_edi_verifactu_get_applicability_name_map` | internal rule | self | `l10n_es_edi_verifactu` | model | Return dict: l10n_es_applicability -> human readable string |
| `_l10n_es_edi_verifactu_get_applicability` | internal rule | self | `l10n_es_edi_verifactu` |  | Return the Veri*Factu Tax Applicability for the "first" main tax in self. Fallback to '05' ("Other") if there is no main tax or the applicability is not set on the "first" one. Note: Currently we only support one Veri*Factu Tax Applicability for the whole invoice. |
| `_l10n_es_edi_verifactu_get_suggested_clave_regimen` | internal rule | self, special_regime, forced_tax_applicability | `l10n_es_edi_verifactu` |  | Return a suggested Clave Regimen for the taxes in `self` to be used for the Veri*Factu document. Note: Currently we only support one Clave Regimen for a whole Veri*Factu document. |
| `_l10n_es_edi_verifactu_get_tax_details_functions` | internal rule | self, company | `l10n_es_edi_verifactu` | model |  |
| `_compute_l10n_hu_tax_reason` | computation | self | `l10n_hu_edi` | depends: `l10n_hu_tax_type` |  |
| `_compute_l10n_in_gst_tax_type` | computation | self | `l10n_in` | depends: `country_code`, `invoice_repartition_line_ids.tag_ids` |  |
| `_l10n_in_get_hsn_summary_table` | internal rule | self, base_lines, display_uom | `l10n_in` | model |  |
| `_l10n_it_edi_check_exoneration_with_no_tax` | validation | self | `l10n_it` | constrains: `l10n_it_exempt_reason`, `invoice_legal_notes`, `amount`, `invoice_repartition_line_ids`, `refund_repartition_line_ids` |  |
| `_l10n_it_filter_kind` | internal rule | self, kind | `l10n_it_edi`, `l10n_it` |  |  |
| `_l10n_it_is_split_payment` | internal rule | self | `l10n_it` |  | Split payment means that the Public Administration buyer will pay VAT to the tax agency instead of the vendor |
| `_onchange_l10n_it_withholding_type` | on change | self | `l10n_it_edi` | onchange: `l10n_it_withholding_type` | When no withholding type is selected, there should be no withholding reason, the field is hidden |
| `_validate_withholding` | validation | self | `l10n_it_edi` | constrains: `amount`, `l10n_it_withholding_type`, `l10n_it_withholding_reason`, `l10n_it_pension_fund_type` |  |
| `_never_unlink_declaration_of_intent_tax` | internal rule | self | `l10n_it_edi_doi` | ondelete |  |
| `_l10n_jo_is_exempt_tax` | internal rule | self | `l10n_jo_edi` |  |  |
| `_onchange_l10n_ke_item_code_id` | on change | self | `l10n_ke` | onchange: `amount` | When the amount of the tax changes this field is reset |
| `_compute_l10n_mx_tax_type` | computation | self | `l10n_mx` | depends: `country_id` |  |
| `_compute_l10n_my_tax_type` | computation | self | `l10n_my_edi` | depends: `amount`, `country_id`, `tax_scope` | Compute default tax type based on a few factors. |
| `_l10n_sa_constrain_is_retention` | validation | self | `l10n_sa_edi` | constrains: `l10n_sa_is_retention`, `amount`, `type_tax_use` |  |
| `_compute_l10n_tw_edi_tax_type` | computation | self | `l10n_tw_edi_ecpay` | depends: `country_id`, `amount` |  |
| `_onchange_l10n_tw_edi_tax_type` | on change | self | `l10n_tw_edi_ecpay` | onchange: `l10n_tw_edi_tax_type` |  |
| `_check_special_tax_type_constrains` | validation | self | `l10n_tw_edi_ecpay` | constrains: `l10n_tw_edi_tax_type`, `l10n_tw_edi_special_tax_type` |  |

## Validation and error messages (28)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_name` | ValidationError | Tax names must be unique! %(taxes)s | `account` |
| `validate_tax_group_id` | ValidationError | The tax group must have the same country_id as the tax using it. | `account` |
| `_constrains_cash_basis_transition_account` | ValidationError | The cash basis transition account needs to allow reconciliation. | `account` |
| `_check_repartition_lines` | ValidationError | Invoice and credit note distribution should each contain exactly one line for the base. | `account` |
| `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have the same number of lines. | `account` |
| `_validate_repartition_lines` | ValidationError | Invoice and credit note repartition should have at least one tax repartition line. | `account` |
| `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have a total factor (+) equals to 100. | `account` |
| `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should have a total factor (-) equals to 100. | `account` |
| `_validate_repartition_lines` | ValidationError | Invoice and credit note distribution should match (same percentages, in the same order). | `account` |
| `_check_children_scope` | ValidationError | Recursion found for tax “%s”. | `account` |
| `_check_children_scope` | ValidationError | The application scope of taxes in a group must be either the same as the group or left empty. | `account` |
| `_check_children_scope` | ValidationError | Nested group of taxes are not allowed. | `account` |
| `_check_company_consistency` | UserError | You can't change the company of your tax since there are some journal items linked to it. | `account` |
| `unlink_except_tax_used` | ValidationError | You cannot delete taxes that are currently in use. Consider archiving them instead. | `account` |
| `_eval_tax_amount_formula` | ValidationError | Only primitive types are allowed in python tax formula context. | `account_tax_python` |
| `_check_amount_type` | UserError | Withholding On Payment taxes cannot use the 'Group of Taxes' or the 'Percentage Tax Included' computations. | `l10n_account_withholding_tax` |
| `write` | UserError | It is forbidden to modify a tax used in a POS order not posted. You must close the POS sessions before modifying the tax. | `point_of_sale` |
| `_l10n_it_edi_check_exoneration_with_no_tax` | ValidationError | If the tax amount is 0%, you must enter the exoneration code and the related legal notes. | `l10n_it` |
| `_l10n_it_edi_check_exoneration_with_no_tax` | UserError | Split Payment is not compatible with exoneration of kind 'N6' | `l10n_it` |
| `_validate_withholding` | ValidationError | Tax '%s' has a withholding type so the amount must be negative. | `l10n_it_edi` |
| `_validate_withholding` | ValidationError | Tax '%s' has a withholding type, so the withholding reason must also be specified | `l10n_it_edi` |
| `_validate_withholding` | ValidationError | Tax '%s' has a withholding reason, so the withholding type must also be specified | `l10n_it_edi` |
| `_validate_withholding` | ValidationError | Tax '%s' has one of withholding and pension fund types that do not relate to ENASARCO, and one that does. | `l10n_it_edi` |
| `_validate_withholding` | ValidationError | Tax '%s' has withholding type ENASARCO, the withholding reason should be [ZO] - Other reason. | `l10n_it_edi` |
| `_never_unlink_declaration_of_intent_tax` | UserError | You cannot delete the special tax for Declarations of Intent. | `l10n_it_edi_doi` |
| `_l10n_sa_constrain_is_retention` | UserError | The tax is unable to be set as Retention as the Amount is greater than or equal to 0. | `l10n_sa_edi` |
| `_check_special_tax_type_constrains` | UserError | Invalid special tax type for Duty free tax type. | `l10n_tw_edi_ecpay` |
| `_check_special_tax_type_constrains` | UserError | Zero tax rate and Duty free tax type must have a tax amount of 0. | `l10n_tw_edi_ecpay` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `account` |
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_invoice` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |
| `group_purchase_user` | no | yes | no | no | `purchase` |
| `group_purchase_manager` | no | yes | no | no | `purchase` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `base.group_public` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Tax multi-company | global (all users) | `[('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (43)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_tax_tree` | list |  | `sequence`, `name`, `description`, `type_tax_use`, `original_tax_ids`, `tax_scope`, `invoice_label`, `company_id`, `country_id`, `active` |  |  | `account` |
| `account.view_onboarding_tax_tree` | xpath | `account.view_tax_tree` |  |  |  | `account` |
| `account.account_tax_view_tree` | list |  | `display_name`, `tax_scope`, `description` |  |  | `account` |
| `account.account_tax_fiscal_position_view_tree` | field | `account.account_tax_view_tree` | `display_name`, `type_tax_use` |  |  | `account` |
| `account.view_tax_kanban` | kanban |  | `name`, `type_tax_use`, `tax_scope`, `description` |  |  | `account` |
| `account.view_account_tax_search` | search |  | `name`, `description`, `amount`, `company_id`, `fiscal_position_ids` |  | `Sale`, `Purchase`, `Services`, `Goods`, `Domestic`, `Active`, `Inactive`, `Company`, `Tax Type`, `Tax Scope`, `Fiscal Position` | `account` |
| `account.account_tax_view_search` | search |  | `name`, `company_id` |  |  | `account` |
| `account.view_tax_form` | form |  | `company_id`, `name`, `amount_type`, `active`, `is_used`, `type_tax_use`, `tax_scope`, `amount`, `fiscal_position_ids`, `original_tax_ids`, `country_code`, `invoice_repartition_line_ids`, `refund_repartition_line_ids`, `children_tax_ids`, `sequence`, `name`, `amount_type`, `amount`, `invoice_label`, `description`, `tax_group_id`, `analytic`, `company_id`, `country_id`, `invoice_legal_notes`, `price_include`, `price_include_override`, `include_base_amount`, `is_base_affected`, `hide_tax_exigibility`, `tax_exigibility`, `cash_basis_transition_account_id` |  |  | `account` |
| `account_edi_ubl_cii.view_account_invoice_form_inherit` | xpath | `account.view_tax_form` | `ubl_cii_tax_category_code`, `ubl_cii_tax_exemption_reason_code` |  |  | `account_edi_ubl_cii` |
| `account_tax_python.view_tax_form_inherited` | xpath | `account.view_tax_form` | `formula` |  |  | `account_tax_python` |
| `l10n_account_withholding_tax.view_tax_form` | xpath | `account.view_tax_form` | `is_withholding_tax_on_payment` |  |  | `l10n_account_withholding_tax` |
| `l10n_ar_withholding.view_tax_form-l10n_ar` | field | `account.view_tax_form` | `type_tax_use` |  |  | `l10n_ar_withholding` |
| `l10n_br.view_l10n_br_account_tax_form` | field | `account.view_tax_form` | `price_include`, `tax_discount` |  |  | `l10n_br` |
| `l10n_cl.view_account_tax_form` | field | `account.view_tax_form` | `name`, `l10n_cl_sii_code` |  |  | `l10n_cl` |
| `l10n_cl.view_tax_sii_code_tree` | field | `account.view_tax_tree` | `name`, `l10n_cl_sii_code` |  |  | `l10n_cl` |
| `l10n_de.view_account_tax_form_inherit` | field | `account.view_tax_form` | `include_base_amount`, `l10n_de_datev_code` |  |  | `l10n_de` |
| `l10n_ec.account_tax_form_view` | xpath | `account.view_tax_form` | `l10n_ec_code_base`, `l10n_ec_code_applied`, `l10n_ec_code_ats` |  |  | `l10n_ec` |
| `l10n_ee.view_tax_form_inherit_l10n_ee` | field | `account.view_tax_form` | `country_id`, `l10n_ee_kmd_inf_code` |  |  | `l10n_ee` |
| `l10n_eg.view_account_tax_form` | field | `account.view_tax_form` | `tax_scope`, `l10n_eg_eta_code` |  |  | `l10n_eg` |
| `l10n_eg.view_tax_eta_code_tree` | field | `account.view_tax_tree` | `name`, `l10n_eg_eta_code` |  |  | `l10n_eg` |
| `l10n_es.account_tax_form_inherit_l10n_es_edi` | xpath | `account.view_tax_form` | `l10n_es_type`, `l10n_es_exempt_reason`, `l10n_es_bien_inversion` |  |  | `l10n_es` |
| `l10n_es_edi_facturae.view_tax_tree_inherit_l10n_es_edi_facturae` | field | `account.view_tax_tree` | `country_id`, `country_code`, `l10n_es_edi_facturae_tax_type` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.view_tax_form_inherit_l10n_es_edi_facturae` | field | `account.view_tax_form` | `country_id`, `l10n_es_edi_facturae_tax_type` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.view_tax_tree_inherit_l10n_es_edi_facturae` | field | `account.view_tax_tree` | `country_id`, `country_code`, `l10n_es_edi_facturae_tax_type` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_verifactu.view_tax_form_inherit_l10n_es_edi_verifactu` | field | `account.view_tax_form` | `country_id`, `l10n_es_applicability` |  |  | `l10n_es_edi_verifactu` |
| `l10n_gr_edi.view_account_tax_form_inherit` | field | `account.view_tax_form` | `cash_basis_transition_account_id`, `l10n_gr_edi_default_tax_exemption_category` |  |  | `l10n_gr_edi` |
| `l10n_hr_edi.view_tax_form_inherit` | field | `account.view_tax_form` | `country_id`, `l10n_hr_tax_category_id` |  |  | `l10n_hr_edi` |
| `l10n_hu_edi.view_account_tax_form_l10n_hu_edi` | field | `account.view_tax_form` | `name`, `l10n_hu_tax_type`, `l10n_hu_tax_reason` |  |  | `l10n_hu_edi` |
| `l10n_in.view_tax_form_inherit_l10n_in` | field | `account.view_tax_form` | `tax_scope`, `l10n_in_tax_type` |  |  | `l10n_in` |
| `l10n_it.account_tax_form_l10n_it` | data | `account.view_tax_form` | `l10n_it_exempt_reason` |  |  | `l10n_it` |
| `l10n_it_edi.account_view_tax_form_l10n_it_edi_extended` | xpath | `l10n_it.account_tax_form_l10n_it` | `l10n_it_withholding_type`, `l10n_it_withholding_reason`, `l10n_it_pension_fund_type` |  |  | `l10n_it_edi` |
| `l10n_ke.l10n_ke_inherit_view_tax_tree` | field | `account.view_tax_tree` | `description`, `l10n_ke_item_code_id` |  |  | `l10n_ke` |
| `l10n_ke.l10n_ke_inherit_view_tax_form` | xpath | `account.view_tax_form` | `l10n_ke_item_code_id` |  |  | `l10n_ke` |
| `l10n_lt.account_tax_form_inherit_l10n_lt` | xpath | `account.view_tax_form` | `l10n_lt_tax_code` |  |  | `l10n_lt` |
| `l10n_mx.account_tax_form_inherit_l10n_mx` | xpath | `account.view_tax_form` | `l10n_mx_tax_type` |  |  | `l10n_mx` |
| `l10n_my_edi.view_tax_form_inherit_l10n_my_myinvois` | xpath | `account.view_tax_form` | `l10n_my_tax_type`, `l10n_my_tax_exemption_reason` |  |  | `l10n_my_edi` |
| `l10n_no.account_tax_form_inherit_l10n_no` | xpath | `account.view_tax_form` | `l10n_no_standard_code` |  |  | `l10n_no` |
| `l10n_pe.view_tax_form` | xpath | `account.view_tax_form` | `l10n_pe_edi_tax_code`, `l10n_pe_edi_unece_category`, `l10n_pe_edi_isc_type` |  |  | `l10n_pe` |
| `l10n_ph.view_tax_form` | notebook | `account.view_tax_form` | `l10n_ph_atc` |  |  | `l10n_ph` |
| `l10n_sa_edi.view_tax_form` | xpath | `account.view_tax_form` | `l10n_sa_is_retention`, `l10n_sa_exemption_reason_code` |  |  | `l10n_sa_edi` |
| `l10n_tr_nilvera_einvoice_extended.view_account_tax_form_view_l10n_tr_nilvera_extended` | xpath | `account.view_tax_form` | `l10n_tr_tax_withholding_code_id` |  |  | `l10n_tr_nilvera_einvoice_extended` |
| `l10n_tw_edi_ecpay.ec_view_tax_form_inherit` | xpath | `account.view_tax_form` | `l10n_tw_edi_tax_type`, `l10n_tw_edi_special_tax_type` |  |  | `l10n_tw_edi_ecpay` |
| `l10n_uy.view_tax_form` | field | `account.view_tax_form` | `tax_group_id`, `l10n_uy_tax_category` |  |  | `l10n_uy` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.action_tax_form` | Taxes | list,kanban,form |  | `{'search_default_sale': True, 'search_default_purchase': True, 'active_test': False}` |  | `account` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.menu_action_tax_form_open` |  | `point_of_sale.menu_point_config_product` | `account.action_tax_form` | 40 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/account.tax.json`; views: `../../../schemas/interfaces/views/account.tax.json`.
