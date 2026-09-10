# Universal Business Language 2.1 (`account.edi.xml.ubl_21`)

**Transport name:** `account.edi.xml.ubl_21`  
**Storage name:** `account_edi_xml_ubl_21`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: UBL 2.1

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_20`

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_get_invoice_node` | preparation rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_allowance_charge_nodes` | internal rule | self, document_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_invoice_line_period_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |
| `_add_document_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_21.json`.
