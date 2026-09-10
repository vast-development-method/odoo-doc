# Universal Business Language 2.1 (JoFotara) (`account.edi.xml.ubl_21.jo`)

**Transport name:** `account.edi.xml.ubl_21.jo`  
**Storage name:** `account_edi_xml_ubl_21_jo`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_jo_edi`

Description: UBL 2.1 (JoFotara)

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (29)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_add_invoice_config_vals` | internal rule | self, vals | `l10n_jo_edi` |  |  |
| `_add_base_lines_edi_ids` | internal rule | self, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_base_lines_vals` | internal rule | self, vals | `l10n_jo_edi` |  |  |
| `format_float` | operation | self, amount, precision_digits | `l10n_jo_edi` |  |  |
| `_get_tax_category_code` | preparation rule | self, customer, supplier, tax | `l10n_jo_edi` |  |  |
| `_sanitize_phone` | internal rule | self, raw | `l10n_jo_edi` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_seller_supplier_party_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_delivery_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_jo_edi` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_jo_edi` |  |  |
| `_add_document_tax_total_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_add_document_allowance_charge_nodes` | internal rule | self, document_node, vals | `l10n_jo_edi` |  |  |
| `_get_document_allowance_charge_node` | preparation rule | self, vals | `l10n_jo_edi` |  | For JO UBL the document allowance charge needs to be the sum of the line discounts. |
| `_get_tax_subtotal_node` | preparation rule | self, vals | `l10n_jo_edi` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_line_id_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_get_line_edi_id` | preparation rule | self, line, default_id | `l10n_jo_edi` |  |  |
| `_add_document_line_gross_subtotal_and_discount_vals` | internal rule | self, vals | `l10n_jo_edi` |  | In JO, because of the precision requirements, we first compute an exact gross unit price, rounded to 9 decimals, and then use it to compute the discount amounts. |
| `_add_invoice_line_amount_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_line_allowance_charge_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_add_document_line_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_line_item_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_add_document_line_tax_category_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_add_invoice_line_price_nodes` | internal rule | self, line_node, vals | `l10n_jo_edi` |  |  |
| `_get_line_discount_allowance_charge_node` | preparation rule | self, vals | `l10n_jo_edi` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_21.jo.json`.
