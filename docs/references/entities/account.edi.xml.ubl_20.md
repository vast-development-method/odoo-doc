# Universal Business Language 2.0 (`account.edi.xml.ubl_20`)

**Transport name:** `account.edi.xml.ubl_20`  
**Storage name:** `account_edi_xml_ubl_20`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: UBL 2.0

## Identity and behavior

- Mixins (classical inheritance): `account.edi.ubl`

## Operations (88)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_find_value` | internal rule | self, xpath, tree, nsmap | `account_edi_ubl_cii` |  |  |
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_document_type_code_node` | preparation rule | self, invoice, invoice_data | `account_edi_ubl_cii` |  | Returns the `DocumentTypeCode` node tag |
| `_export_invoice` | internal rule | self, invoice | `account_edi_ubl_cii` |  | Generates an UBL 2.0 xml for a given invoice. |
| `_get_document_template` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_get_document_nsmap` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `format_float` | operation | self, amount, precision_digits | `account_edi_ubl_cii` |  |  |
| `_get_tags_for_document_type` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_is_document_allowance_charge` | internal rule | self, base_line | `account_edi_ubl_cii` |  | Whether the base line should be treated as a document-level AllowanceCharge. |
| `_get_invoice_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_config_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_dispatch_base_lines_recycling_contribution_taxes` | internal rule | self, base_lines, company, vals | `account_edi_ubl_cii` |  | Extract recycling contribution taxes such as RECUPEL, AUVIBEL, etc from the current base lines. Instead, add them under 'base_line' -> '_ubl_values' -> 'recycling_contribution_data' to be reported as allowances/charges.  From a 'base_line' having     price_unit = 99     tax_ids = RECUPEL of 1 + 21% tax     total_excluded_currency = 99     total_included_currency = 121     taxes_data = [1, 21]     recycling_contribution_data = [] ... turn it to:     price_unit = 99     tax_ids = 21% tax     total_excluded_currency = 99     total_included_currency = 121     taxes_data = [21]     recycling_contri |
| `_turn_emptying_taxes_as_new_base_lines` | internal rule | self, base_lines, company, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_base_lines_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_setup_base_lines` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_currency_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_tax_grouping_function_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_monetary_totals_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_seller_supplier_party_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_delivery_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_payment_terms_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_allowance_charge_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_exchange_rate_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_tax_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_optional_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `add_invoice_optional_nodes` | operation | self, document_node, vals, optional_fields | `account_edi_ubl_cii` |  |  |
| `_get_invoice_line_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_id_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_note_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_amount_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_period_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_tax_total_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_item_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_tax_category_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_price_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_pricing_reference_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_optional_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `add_invoice_line_optional_nodes` | operation | self, line_node, vals, optional_line_fields | `account_edi_ubl_cii` |  |  |
| `_add_document_currency_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  | Add the 'currency_suffix', 'currency_dp' and 'currency_name'. |
| `_add_document_tax_grouping_function_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_monetary_total_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_get_address_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate the Address node for a res.partner or res.bank. |
| `_get_party_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate the Party node for a res.partner. |
| `_get_financial_account_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate the FinancialAccount node for a res.partner.bank |
| `_add_document_tax_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  | Generic helper to fill the TaxTotal and WithholdingTaxTotal nodes for a document. |
| `_add_tax_total_node_in_company_currency` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  | Generic helper to add a TaxTotal section in the company currency. |
| `_get_tax_total_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate a TaxTotal node given a dict of aggregated tax details. |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate a TaxSubtotal node given a tax grouping key dict and associated tax values. |
| `_get_tax_category_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate a TaxCategory node given a tax grouping key dict. |
| `_add_document_monetary_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  | Generic helper to fill the MonetaryTotal node for a document given a list of base_lines. |
| `_get_document_line_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_allowance_charge_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  | Generic helper to fill the AllowanceCharge nodes for a document given a list of base_lines. |
| `_get_document_allowance_charge_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to generate a document-level AllowanceCharge node given a base_line. |
| `_add_document_line_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  | Generic helper to calculate the amounts for a document line. |
| `_add_document_line_total_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_gross_subtotal_and_discount_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_id_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_note_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_amount_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_period_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_item_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_get_line_discount_allowance_charge_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_get_line_fixed_tax_allowance_charge_nodes` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_tax_category_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_tax_total_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_price_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_pricing_reference_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_partner_vals` | internal rule | self, tree, role | `account_edi_ubl_cii` |  | Returns a dict of values that will be used to retrieve the partner |
| `_get_postal_address` | preparation rule | self, tree, role | `account_edi_ubl_cii` |  |  |
| `_import_fill_invoice` | internal rule | self, invoice, tree, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_tax_nodes` | preparation rule | self, tree | `account_edi_ubl_cii` |  |  |
| `_get_document_allowance_charge_xpaths` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_invoice_line_xpaths` | preparation rule | self, document_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_line_xpaths` | preparation rule | self, document_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_product_xpaths` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_correct_invoice_tax_amount` | internal rule | self, tree, invoice | `account_edi_ubl_cii` |  | The tax total may have been modified for rounding purpose, if so we should use the imported tax and not the computed one |
| `_get_import_document_amount_sign` | preparation rule | self, tree | `account_edi_ubl_cii` |  | In UBL, an invoice has tag 'Invoice' and a credit note has tag 'CreditNote'. However, a credit note can be expressed as an invoice with negative amounts. For this case, we need a factor to take the opposite of each quantity in the invoice. |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_20.json`.
