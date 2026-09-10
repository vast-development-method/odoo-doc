# Universal Business Language CEN-EN16931 (`account.edi.ubl_cen_en16931`)

**Transport name:** `account.edi.ubl_cen_en16931`  
**Storage name:** `account_edi_ubl_cen_en16931`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: UBL CEN-EN16931

## Identity and behavior

- Mixins (classical inheritance): `account.edi.ubl`

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_ubl_add_line_allowance_charge_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_line_nodes_filter_base_lines` | internal rule | self, vals, filter_function | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_tax_scheme_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_allowance_charge_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_default_tax_category_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_ubl_tax_totals_node_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_export_document_node_constraints` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_init_invoice_export_values` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.ubl_cen_en16931.json`.
