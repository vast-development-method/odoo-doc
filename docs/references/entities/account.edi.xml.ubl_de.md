# BIS3 DE (XRechnung) (`account.edi.xml.ubl_de`)

**Transport name:** `account.edi.xml.ubl_de`  
**Storage name:** `account_edi_xml_ubl_de`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: BIS3 DE (XRechnung)

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `account_edi_ubl_cii` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `account_edi_ubl_cii` |  |  |
| `_ubl_add_tax_currency_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_tax_totals_node_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_buyer_reference_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_endpoint_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_tax_scheme_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_legal_entity_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_get_line_allowance_charge_discount_node` | internal rule | self, vals, discount_values | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_de.json`.
