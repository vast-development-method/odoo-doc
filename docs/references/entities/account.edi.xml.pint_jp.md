# Japanese implementation of Peppol International (PINT) model for Billing (`account.edi.xml.pint_jp`)

**Transport name:** `account.edi.xml.pint_jp`  
**Storage name:** `account_edi_xml_pint_jp`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_jp_ubl_pint`

Description: Japanese implementation of Peppol International (PINT) model for Billing

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_jp_ubl_pint` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `l10n_jp_ubl_pint` |  |  |
| `_ubl_get_tax_subtotal_node` | internal rule | self, vals, tax_subtotal | `l10n_jp_ubl_pint` |  |  |
| `_ubl_tax_totals_node_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `l10n_jp_ubl_pint` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `l10n_jp_ubl_pint` |  |  |
| `_ubl_add_profile_id_node` | internal rule | self, vals | `l10n_jp_ubl_pint` |  |  |
| `_ubl_add_invoice_period_nodes` | internal rule | self, vals | `l10n_jp_ubl_pint` |  |  |
| `_ubl_add_party_legal_entity_nodes` | internal rule | self, vals | `l10n_jp_ubl_pint` |  |  |
| `_export_document_node_constraints` | internal rule | self, vals | `l10n_jp_ubl_pint` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.pint_jp.json`.
