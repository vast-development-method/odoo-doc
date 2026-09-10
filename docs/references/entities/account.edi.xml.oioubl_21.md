# OIOUBL 2.1 (`account.edi.xml.oioubl_21`)

**Transport name:** `account.edi.xml.oioubl_21`  
**Storage name:** `account_edi_xml_oioubl_21`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_dk_nemhandel`

Description: OIOUBL 2.1

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_dk_nemhandel` |  |  |
| `_get_currency_decimal_places` | preparation rule | self, currency_id | `l10n_dk_nemhandel` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `l10n_dk_nemhandel` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_dk_nemhandel` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_dk_nemhandel` |  |  |
| `_get_document_type_code_node` | preparation rule | self, invoice, invoice_data | `l10n_dk_nemhandel` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_dk_nemhandel` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_dk_nemhandel` |  |  |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `l10n_dk_nemhandel` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_dk_nemhandel` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_dk_nemhandel` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_dk_nemhandel` |  |  |
| `_add_invoice_payment_terms_nodes` | internal rule | self, document_node, vals | `l10n_dk_nemhandel` |  |  |
| `_add_document_line_price_nodes` | internal rule | self, line_node, vals | `l10n_dk_nemhandel` |  |  |
| `_retrieve_rebate_val` | internal rule | self, tree, xpath_dict, quantity | `l10n_dk_nemhandel` |  |  |
| `_get_line_discount_allowance_charge_node` | preparation rule | self, vals | `l10n_dk_nemhandel` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.oioubl_21.json`.
