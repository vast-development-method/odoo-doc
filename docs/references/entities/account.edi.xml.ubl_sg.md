# SG BIS Billing 3.0 (`account.edi.xml.ubl_sg`)

**Transport name:** `account.edi.xml.ubl_sg`  
**Storage name:** `account_edi_xml_ubl_sg`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: SG BIS Billing 3.0

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `account_edi_ubl_cii` |  |  |
| `_ubl_default_tax_category_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_ubl_get_line_allowance_charge_discount_node` | internal rule | self, vals, discount_values | `account_edi_ubl_cii` |  |  |
| `_ubl_add_tax_currency_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_tax_totals_node_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_payment_means_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_sg.json`.
