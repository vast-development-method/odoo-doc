# Factur-x/ZUGFeRD Cross Industry Invoice 2.2.0 (`account.edi.xml.cii`)

**Transport name:** `account.edi.xml.cii`  
**Storage name:** `account_edi_xml_cii`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`  
**Extended by packages:** `l10n_fr_pdp`

Description: Factur-x/ZUGFeRD CII 2.2.0

## Identity and behavior

- Mixins (classical inheritance): `account.edi.cii`

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_find_value` | internal rule | self, xpath, tree, nsmap | `account_edi_ubl_cii` |  |  |
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_export_invoice_ecosio_schematrons` | internal rule | self | `account_edi_ubl_cii` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `account_edi_ubl_cii` |  |  |
| `_check_required_tax` | validation | self, vals | `account_edi_ubl_cii` |  |  |
| `_check_non_0_rate_tax` | validation | self, vals | `account_edi_ubl_cii` |  |  |
| `_get_scheduled_delivery_time` | preparation rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_invoicing_period` | preparation rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_exchanged_document_vals` | preparation rule | self, invoice | `account_edi_ubl_cii`, `l10n_fr_pdp` |  |  |
| `_export_invoice_vals` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_export_invoice` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_import_retrieve_partner_vals` | internal rule | self, tree, role | `account_edi_ubl_cii` |  |  |
| `_get_postal_address` | preparation rule | self, tree, role | `account_edi_ubl_cii` |  |  |
| `_import_fill_invoice` | internal rule | self, invoice, tree, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_tax_nodes` | preparation rule | self, tree | `account_edi_ubl_cii` |  |  |
| `_get_document_allowance_charge_xpaths` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_invoice_line_xpaths` | preparation rule | self, document_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_line_xpaths` | preparation rule | self, document_type, qty_factor | `account_edi_ubl_cii` |  |  |
| `_get_import_document_amount_sign` | preparation rule | self, tree | `account_edi_ubl_cii` |  | In factur-x, an invoice has code 380 and a credit note has code 381. However, a credit note can be expressed as an invoice with negative amounts. For this case, we need a factor to take the opposite of each quantity in the invoice. |
| `_export_invoice_new` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_export_invoice_constraints_new` | internal rule | self, invoice, vals | `account_edi_ubl_cii` |  |  |
| `_get_document_nsmap` | preparation rule | self | `account_edi_ubl_cii` |  |  |
| `_get_invoice_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_import_invoice_ubl_cii` | internal rule | self, invoice, file_data, new | `account_edi_ubl_cii` |  | :param account.move invoice: |
| `_import_prepare_missing_customer_create_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.cii.json`.
