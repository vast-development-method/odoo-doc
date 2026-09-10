# PoS Order Universal Business Language 2.1 builder (`pos.edi.xml.ubl_21`)

**Transport name:** `pos.edi.xml.ubl_21`  
**Storage name:** `pos_edi_xml_ubl_21`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `pos_edi_ubl`

Description: PoS Order UBL 2.1 builder

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (27)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_pos_order` | internal rule | self, pos_order | `pos_edi_ubl` |  |  |
| `_get_pos_order_node` | preparation rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_config_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_base_lines_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_currency_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_tax_grouping_function_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_monetary_totals_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_header_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_payment_means_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_allowance_charge_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_tax_total_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_monetary_total_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_nodes` | internal rule | self, document_node, vals | `pos_edi_ubl` |  |  |
| `_get_pos_order_line_node` | preparation rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_vals` | internal rule | self, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_id_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_note_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_amount_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_period_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_tax_total_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_item_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_tax_category_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_add_pos_order_line_price_nodes` | internal rule | self, line_node, vals | `pos_edi_ubl` |  |  |
| `_export_pos_order_constraints` | internal rule | self, pos_order, vals | `pos_edi_ubl` |  |  |

Machine-readable definition: `../../../schemas/data/entities/pos.edi.xml.ubl_21.json`.
