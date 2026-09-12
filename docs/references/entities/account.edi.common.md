# Common functions for electronic data interchange documents: generate the data, the constraints, etc (`account.edi.common`)

**Transport name:** `account.edi.common`  
**Storage name:** `account_edi_common`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`  
**Extended by packages:** `account_peppol`, `l10n_fr_pdp`

Description: Common functions for EDI documents: generate the data, the constraints, etc

## Operations (71)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_vals_to_etree` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_etree_to_string` | internal rule | self, tree | `account_edi_ubl_cii` |  |  |
| `_define_document_type` | internal rule | self, vals, document_type | `account_edi_ubl_cii` |  |  |
| `_get_document_type` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_is_document` | internal rule | self, vals, *document_types | `account_edi_ubl_cii` |  |  |
| `module_installed` | operation | self, module_name | `account_edi_ubl_cii` |  |  |
| `format_float` | operation | self, amount, precision_digits | `account_edi_ubl_cii` |  |  |
| `_get_currency_decimal_places` | preparation rule | self, currency_id | `account_edi_ubl_cii` |  |  |
| `_get_uom_unece_code` | preparation rule | self, uom | `account_edi_ubl_cii` |  | list of codes: https://docs.peppol.eu/poacc/billing/3.0/codelist/UNECERec20/ (sorted by letter) |
| `_find_value` | internal rule | self, xpaths, tree, nsmap | `account_edi_ubl_cii` |  | Iteratively queries the tree using the xpaths and returns a result as soon as one is found |
| `_can_export_selfbilling` | internal rule | self | `account_edi_ubl_cii` |  |  |
| `_get_belgian_cocontractant_note` | preparation rule | self, customer, supplier | `account_edi_ubl_cii` |  |  |
| `_is_cocontractant_fiscal_position` | internal rule | self, invoice, customer, supplier | `account_edi_ubl_cii` |  |  |
| `_turn_price_unit_positive` | internal rule | self, vals | `account_edi_ubl_cii` |  | Turn the unit_price positive and the quantity negative of the base_line when negative unit_price. [BR-27]-The Item net price (BT-146) shall NOT be negative. [BR-28]-The Item gross price (BT-148) shall NOT be negative.  :param vals: Some custom data. |
| `_validate_taxes` | internal rule | self, tax_ids | `account_edi_ubl_cii` |  | Validate the structure of the tax repartition lines (invalid structure could lead to unexpected results) |
| `_get_tax_category_code` | preparation rule | self, customer, supplier, tax | `account_edi_ubl_cii` |  | Predicts the tax category code for a tax applied to a given base line. If the tax has a defined category code, it is returned. Otherwise, a reasonable default is provided, though it may not always be accurate.  Source: doc of Peppol (but the CEF norm is also used by factur-x, yet not detailed) https://docs.peppol.eu/poacc/billing/3.0/syntax/ubl-invoice/cac-TaxTotal/cac-TaxSubtotal/cac-TaxCategory/cbc-TaxExemptionReasonCode/ https://docs.peppol.eu/poacc/billing/3.0/codelist/vatex/ https://docs.peppol.eu/poacc/billing/3.0/codelist/UNCL5305/ |
| `_get_tax_exemption_reason` | preparation rule | self, customer, supplier, tax | `account_edi_ubl_cii` |  | Returns the reason and code from the tax if available. If not, it falls back to the default tax exemption reason defined for the respective tax category code.  Note: In Peppol, taxes should be grouped by tax category code but *not* by exemption reason, see https://docs.peppol.eu/poacc/billing/3.0/bis/#_calculation_of_vat |
| `_check_required_fields` | validation | self, record, field_names, custom_warning_message | `account_edi_ubl_cii` |  | Check if at least one of the field_names are set on the record/dict  :param record: either a recordSet or a dict :param field_names: The field name or list of field name that has to                     be checked. If a list is provided, check that at                     least one of them is set. :return: an Error message or None |
| `_invoice_constraints_common` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_default_notes` | preparation rule | self, vals | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_import_invoice_ubl_cii` | internal rule | self, invoice, file_data, new | `account_edi_ubl_cii` |  |  |
| `_add_logs_import_invoice_ubl_cii` | internal rule | self, invoice, invoice_logs | `account_edi_ubl_cii`, `account_peppol` |  |  |
| `_log_import_invoice_ubl_cii` | internal rule | self, invoice, title_logs, invoice_logs, attachments | `account_edi_ubl_cii`, `account_peppol` |  |  |
| `_import_attachments` | internal rule | self, invoice, tree | `account_edi_ubl_cii` |  |  |
| `_import_partner` | internal rule | self, company_id, name, phone, email, vat, peppol_eas, peppol_endpoint, postal_address, **kwargs | `account_edi_ubl_cii` |  | Retrieve the partner, if no matching partner is found, create it (only if he has a vat and a name) |
| `_import_partner_bank` | internal rule | self, invoice, bank_details | `account_edi_ubl_cii` |  |  |
| `_import_document_allowance_charges` | internal rule | self, tree, record, tax_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_import_currency` | internal rule | self, tree, xpath | `account_edi_ubl_cii` |  |  |
| `_import_description` | internal rule | self, tree, xpaths | `account_edi_ubl_cii` |  |  |
| `_import_prepaid_amount` | internal rule | self, invoice, tree, xpath, qty_factor | `account_edi_ubl_cii` |  |  |
| `_import_lines` | internal rule | self, record, tree, xpath, document_type, tax_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_import_rounding_amount` | internal rule | self, invoice, tree, xpath, document_type, qty_factor | `account_edi_ubl_cii` |  | Add an invoice line representing the rounding amount given in the document. - The amount is assumed to be in document currency |
| `_retrieve_invoice_line_vals` | internal rule | self, tree, document_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_retrieve_rebate_val` | internal rule | self, tree, xpath_dict, quantity | `account_edi_ubl_cii` | model |  |
| `_retrieve_charge_allowance_vals` | internal rule | self, tree, xpath_dict, quantity | `account_edi_ubl_cii` | model |  |
| `_get_basis_qty` | preparation rule | self, tree, xpath_dict | `account_edi_ubl_cii` |  | Return the base quantity used to derive the unit price from PriceAmount. The standard UBL/CII divides PriceAmount by BaseQuantity to obtain the unit price upon import. |
| `_retrieve_line_vals` | internal rule | self, tree, document_type, qty_factor | `account_edi_ubl_cii` |  | Read the xml invoice, extract the invoice line values, compute the system values to fill an invoice line form: quantity, price_unit, discount, product_uom_id.  The way of computing invoice line is quite complicated: https://docs.peppol.eu/poacc/billing/3.0/bis/#_calculation_on_line_level (same as in factur-x documentation)  line_net_subtotal = ( gross_unit_price - rebate ) * (delivered_qty / basis_qty) - allow_charge_amount  with (UBL \| CII):     * net_unit_price = 'Price/PriceAmount' \| 'NetPriceProductTradePrice' (mandatory) (BT-146)     * gross_unit_price = 'Price/AllowanceCharge/BaseAmount' |
| `_import_product` | internal rule | self, **product_vals | `account_edi_ubl_cii` |  |  |
| `_retrieve_fixed_tax` | internal rule | self, company_id, fixed_tax_vals | `account_edi_ubl_cii` |  | Retrieve the fixed tax at import, iteratively search for a tax: 1. not price_include matching the name and the amount 2. not price_include matching the amount 3. price_include matching the name and the amount 4. price_include matching the amount |
| `_retrieve_taxes` | internal rule | self, record, line_values, tax_type, tax_exigibility | `account_edi_ubl_cii` |  | Retrieve the taxes on the document line at import.  In a UBL/CII xml, the system "price_include" concept does not exist. Hence, first look for a price_include=False, if it is unsuccessful, look for a price_include=True. |
| `_retrieve_line_charges` | internal rule | self, record, line_values, taxes | `account_edi_ubl_cii` |  | Handle the charges on the document line at import.  For each charge on the line, it creates a new aml. Special case: if the ReasonCode == 'AEO', there is a high chance the xml was produced by the system and the corresponding line had a fixed tax, so it first tries to find a matching fixed tax to apply to the current aml. |
| `_get_document_allowance_charge_xpaths` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_invoice_line_xpaths` | preparation rule | self, invoice_line, qty_factor | `account_edi_ubl_cii` |  |  |
| `_correct_invoice_tax_amount` | internal rule | self, tree, invoice | `account_edi_ubl_cii` |  |  |
| `_import_init_collected_values` | internal rule | self, invoice, file_data | `account_edi_ubl_cii` |  |  |
| `_import_invoice_document_sign` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_update_move_type` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_customer` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_country` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_customer_search_plan` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_check_customer_vat_match` | validation | self, customer, vat, collected_values | `account_edi_ubl_cii` |  | Compare the VAT from an EDI document against a partner's stored VAT, with country-specific normalization where needed. Should stay consistent with `_get_country_specific_vat_variants`. |
| `_import_create_missing_customer` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_prepare_missing_customer_create_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_add_currency` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_partner_bank` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_products_search_plan` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_retrieve_products` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_retrieve_product_uoms` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_retrieve_accounts` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_taxes_search_plan` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_retrieve_taxes` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_get_default_base_line_kwargs` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_get_allowance_charge_line_kwargs` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_line_get_product_base_line_kwargs` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_line_add_optional_fields` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_add_base_lines` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_write_collected_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_fix_taxes_amounts` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_import_invoice_post_processing` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |
| `_generate_pdf_attachment` | internal rule | self, invoice, tree | `account_edi_ubl_cii` |  | ATTEMPTS to create a PDF attachment when the XML file doesn't provide one. |
| `_l10n_fr_pdp_get_profile_id` | internal rule | self, vals | `l10n_fr_pdp` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_taxes` | ValidationError | error_msg | `account_edi_ubl_cii` |

Machine-readable definition: `../../../schemas/data/entities/account.edi.common.json`.
