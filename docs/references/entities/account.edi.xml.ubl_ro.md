# CIUS RO (`account.edi.xml.ubl_ro`)

**Transport name:** `account.edi.xml.ubl_ro`  
**Storage name:** `account_edi_xml_ubl_ro`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_ro_edi`  
**Extended by packages:** `l10n_ro_cpv_code`

Description: CIUS RO

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_ro_edi` |  |  |
| `_get_document_type_code_node` | preparation rule | self, invoice, invoice_data | `l10n_ro_edi` |  |  |
| `_ubl_add_tax_currency_code_node` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_get_partner_address_node` | internal rule | self, vals, partner | `l10n_ro_edi` |  |  |
| `_ubl_add_accounting_supplier_party_tax_scheme_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_accounting_supplier_party_legal_entity_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_accounting_customer_party_tax_scheme_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_accounting_customer_party_legal_entity_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_line_item_name_description_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_ubl_add_notes_nodes` | internal rule | self, vals | `l10n_ro_edi` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_ro_edi` |  |  |
| `_import_retrieve_partner_vals` | internal rule | self, tree, role | `l10n_ro_edi` |  |  |
| `_add_invoice_line_item_nodes` | internal rule | self, line_node, vals | `l10n_ro_cpv_code` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_ro.json`.
