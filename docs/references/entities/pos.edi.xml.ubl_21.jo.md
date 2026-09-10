# Universal Business Language 2.1 (JoFotara) for PoS Orders (`pos.edi.xml.ubl_21.jo`)

**Transport name:** `pos.edi.xml.ubl_21.jo`  
**Storage name:** `pos_edi_xml_ubl_21_jo`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_jo_edi_pos`

Description: UBL 2.1 (JoFotara) for PoS Orders

## Identity and behavior

- Mixins (classical inheritance): `pos.edi.xml.ubl_21`

## Operations (32)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `format_float` | operation | self, amount, precision_digits | `l10n_jo_edi_pos` |  |  |
| `_get_tax_category_code` | preparation rule | self, customer, supplier, tax | `l10n_jo_edi_pos` |  |  |
| `_sanitize_phone` | internal rule | self, raw | `l10n_jo_edi_pos` |  |  |
| `_get_pos_order_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_config_vals` | internal rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_currency_vals` | internal rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_base_lines_edi_ids` | internal rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_base_lines_vals` | internal rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_document_line_gross_subtotal_and_discount_vals` | internal rule | self, vals | `l10n_jo_edi_pos` |  | In JO, because of the precision requirements, we first compute an exact gross unit price, rounded to 9 decimals, and then use it to compute the discount amounts. |
| `_add_pos_order_discount_vals` | internal rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_get_payment_method_code` | preparation rule | self, order | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_header_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_seller_supplier_party_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_allowance_charge_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_tax_total_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi_pos` |  |  |
| `_get_pos_order_line_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_get_pos_order_line_id` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_id_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_amount_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_item_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |
| `_sum_tax_details` | internal rule | self, vals, key, include_fixed | `l10n_jo_edi_pos` |  |  |
| `_get_tax_total_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_price_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |
| `_add_pos_order_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi_pos` |  |  |

Machine-readable definition: `../../../schemas/data/entities/pos.edi.xml.ubl_21.jo.json`.
