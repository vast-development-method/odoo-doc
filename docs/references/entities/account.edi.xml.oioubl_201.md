# OIOUBL 2.01 (`account.edi.xml.oioubl_201`)

**Transport name:** `account.edi.xml.oioubl_201`  
**Storage name:** `account_edi_xml_oioubl_201`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_dk_oioubl`

Description: OIOUBL 2.01

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_20`

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_dk_oioubl` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_dk_oioubl` |  |  |
| `_get_currency_decimal_places` | preparation rule | self, currency_id | `l10n_dk_oioubl` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_dk_oioubl` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_dk_oioubl` |  |  |
| `_add_invoice_payment_terms_nodes` | internal rule | self, document_node, vals | `l10n_dk_oioubl` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_dk_oioubl` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_dk_oioubl` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_dk_oioubl` |  |  |
| `_get_document_type_code_node` | preparation rule | self, invoice, invoice_data | `l10n_dk_oioubl` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_dk_oioubl` |  |  |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `l10n_dk_oioubl` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.oioubl_201.json`.
