# Universal Business Language BIS Billing 3.0.12 (`account.edi.xml.ubl_bis3`)

**Transport name:** `account.edi.xml.ubl_bis3`  
**Storage name:** `account_edi_xml_ubl_bis3`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`  
**Extended by packages:** `account_peppol`, `l10n_fr_facturx_chorus_pro`

Description: UBL BIS Billing 3.0.12

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`, `account.edi.ubl_pint_eu`

## Operations (39)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_is_customer_behind_chorus_pro` | internal rule | self, customer | `account_edi_ubl_cii` | model |  |
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_can_export_selfbilling` | internal rule | self | `account_edi_ubl_cii` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `account_edi_ubl_cii` |  |  |
| `_add_invoice_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_delivery_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_allowance_charge_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_payment_terms_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_tax_total_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_monetary_total_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_id_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_amount_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_period_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_pricing_reference_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_tax_total_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_tax_category_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_item_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_price_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_invoice_line_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_credit_note_line_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii`, `l10n_fr_facturx_chorus_pro` |  |  |
| `_add_invoice_config_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_setup_base_lines` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_base_lines_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_vals` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `account_edi_ubl_cii`, `l10n_fr_facturx_chorus_pro` |  |  |
| `_invoice_constraints_cen_en16931_ubl` | internal rule | self, invoice, vals | `account_edi_ubl_cii` |  |  |
| `_invoice_constraints_peppol_en16931_ubl` | internal rule | self, invoice, vals | `account_edi_ubl_cii`, `account_peppol` |  | corresponds to the errors raised by 'schematron/openpeppol/3.13.0/xslt/PEPPOL-EN16931-UBL.xslt' for invoices in ecosio. This xslt was obtained by transforming the corresponding sch https://docs.peppol.eu/poacc/billing/3.0/files/PEPPOL-EN16931-UBL.sch.  The national rules (https://docs.peppol.eu/poacc/billing/3.0/bis/#national_rules) are included in this file. They always refer to the supplier's country. |
| `_import_order_payment_terms_id` | internal rule | self, company_id, tree, xpath | `account_edi_ubl_cii` |  | Return payment term name from given tree and try to find a match. |
| `_retrieve_order_vals` | internal rule | self, order, tree | `account_edi_ubl_cii` |  |  |
| `_import_order_ubl` | internal rule | self, order, file_data, new | `account_edi_ubl_cii` |  | Common importing method to extract order data from file_data. :param order: Order to fill details from file_data. :param file_data: File data to extract order related data from. :return: True if there's no exception while extraction. :rtype: Boolean |
| `_import_invoice_ubl_cii` | internal rule | self, invoice, file_data, new | `account_edi_ubl_cii` |  | corresponds to the errors raised by 'schematron/openpeppol/3.13.0/xslt/PEPPOL-EN16931-UBL.xslt' for invoices in ecosio. This xslt was obtained by transforming the corresponding sch https://docs.peppol.eu/poacc/billing/3.0/files/PEPPOL-EN16931-UBL.sch.  The national rules (https://docs.peppol.eu/poacc/billing/3.0/bis/#national_rules) are included in this file. They always refer to the supplier's country. |
| `_ubl_add_party_identification_nodes` | internal rule | self, vals | `l10n_fr_facturx_chorus_pro` |  |  |
| `_ubl_add_party_legal_entity_nodes` | internal rule | self, vals | `l10n_fr_facturx_chorus_pro` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_bis3.json`.
