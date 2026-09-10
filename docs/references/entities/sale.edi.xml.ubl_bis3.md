# Sale BIS Ordering 3.5 (`sale.edi.xml.ubl_bis3`)

**Transport name:** `sale.edi.xml.ubl_bis3`  
**Storage name:** `sale_edi_xml_ubl_bis3`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `sale_edi_ubl`

Description: Sale BIS Ordering 3.5

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (32)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_order` | internal rule | self, sale_order | `sale_edi_ubl` |  |  |
| `_get_sale_order_node` | preparation rule | self, vals | `sale_edi_ubl` |  |  |
| `_setup_base_lines` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_config_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_base_lines_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_currency_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_tax_grouping_function_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_monetary_totals_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_header_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_buyer_customer_party_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_seller_supplier_party_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_ubl_get_delivery_node_from_delivery_address` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_delivery_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_payment_terms_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_allowance_charge_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_tax_total_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_monetary_total_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_ubl_add_legal_monetary_total_line_extension_amount_node` | internal rule | self, vals, in_foreign_currency | `sale_edi_ubl` |  |  |
| `_ubl_add_anticipated_monetary_total_node` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_nodes` | internal rule | self, document_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_vals` | internal rule | self, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_id_nodes` | internal rule | self, line_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_amount_nodes` | internal rule | self, line_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_item_nodes` | internal rule | self, line_node, vals | `sale_edi_ubl` |  |  |
| `_add_sale_order_line_price_nodes` | internal rule | self, line_node, vals | `sale_edi_ubl` |  |  |
| `_export_order_vals` | internal rule | self, sale_order | `sale_edi_ubl` |  |  |
| `_ubl_get_line_allowance_charge_discount_node` | internal rule | self, vals, discount_values | `sale_edi_ubl` |  |  |
| `_retrieve_order_vals` | internal rule | self, order, tree | `sale_edi_ubl` |  | Fill order details by extracting details from xml tree. param order: Order to fill details from xml tree. param tree: Xml tree to extract details. :return: list of logs to add warning and information about data from xml. |
| `_import_order_ubl` | internal rule | self, order, file_data, new | `sale_edi_ubl` |  |  |
| `_get_product_xpaths` | preparation rule | self | `sale_edi_ubl` |  | Override of `account.edi.xml.ubl_bis3` to support the `ExtendedID` field used to identify product variants. |
| `_retrieve_line_vals` | internal rule | self, tree, document_type, qty_factor | `sale_edi_ubl` |  | Override of `account.edi.common` to adapt dictionary keys from the base method to be compatible with the `sale.order.line` model. |

Machine-readable definition: `../../../schemas/data/entities/sale.edi.xml.ubl_bis3.json`.
