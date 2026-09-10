# Universal Business Language 2.1 (ZATCA) (`account.edi.xml.ubl_21.zatca`)

**Transport name:** `account.edi.xml.ubl_21.zatca`  
**Storage name:** `account_edi_xml_ubl_21_zatca`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_sa_edi`  
**Extended by packages:** `l10n_sa_edi_pos`

Description: UBL 2.1 (ZATCA)

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (27)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_sa_edi` |  | Generate the name of the invoice XML file according to ZATCA business rules: Seller Vat Number (BT-31), Date (BT-2), Time (KSA-25), Invoice Number (BT-1) |
| `_add_invoice_config_vals` | internal rule | self, vals | `l10n_sa_edi` |  |  |
| `_add_invoice_base_lines_vals` | internal rule | self, vals | `l10n_sa_edi` |  |  |
| `_add_document_tax_grouping_function_vals` | internal rule | self, vals | `l10n_sa_edi` |  |  |
| `_get_tax_category_code` | preparation rule | self, customer, supplier, tax | `l10n_sa_edi` |  | Override to include/update values specific to ZATCA's UBL 2.1 specs |
| `_get_tax_exemption_reason` | preparation rule | self, customer, supplier, tax | `l10n_sa_edi` |  |  |
| `_is_document_allowance_charge` | internal rule | self, base_line | `l10n_sa_edi` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  |  |
| `_add_invoice_delivery_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  | Override to include/update values specific to ZATCA's UBL 2.1 specs |
| `_get_address_node` | preparation rule | self, vals | `l10n_sa_edi` |  |  |
| `_get_partner_party_identification_number` | preparation rule | self, partner | `l10n_sa_edi` |  | Override to include/update values specific to ZATCA's UBL 2.1 specs |
| `_get_party_node` | preparation rule | self, vals | `l10n_sa_edi` |  |  |
| `_l10n_sa_get_payment_means_code` | internal rule | self, invoice | `l10n_sa_edi_pos`, `l10n_sa_edi` |  | Return payment means code to be used to set the value on the XML file |
| `_add_document_tax_total_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  |  |
| `_add_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  |  |
| `_get_document_allowance_charge_node` | preparation rule | self, vals | `l10n_sa_edi` |  | Charge Reasons & Codes (As per ZATCA): https://unece.org/fileadmin/DAM/trade/untdid/d16b/tred/tred5189.htm As far as ZATCA is concerned, we calculate Allowance/Charge vals for global discounts as a document level allowance, and we do not include any other charges or allowances. |
| `_add_invoice_line_nodes` | internal rule | self, document_node, vals | `l10n_sa_edi` |  |  |
| `_add_document_line_tax_total_nodes` | internal rule | self, line_node, vals | `l10n_sa_edi` |  |  |
| `_add_document_line_item_nodes` | internal rule | self, line_node, vals | `l10n_sa_edi` |  |  |
| `_add_document_line_tax_category_nodes` | internal rule | self, line_node, vals | `l10n_sa_edi` |  |  |
| `_get_prepayment_line_node` | preparation rule | self, vals | `l10n_sa_edi` |  |  |
| `_get_prepayment_line_tax_total_node` | preparation rule | self, vals | `l10n_sa_edi` |  |  |
| `_add_document_line_price_nodes` | internal rule | self, line_node, vals | `l10n_sa_edi` |  | Use 10 decimal places for PriceAmount to satisfy ZATCA validation BR-KSA-EN16931-11 |
| `_l10n_sa_get_namespaces` | internal rule | self | `l10n_sa_edi` |  | Namespaces used in the final UBL declaration, required to canonalize the finalized XML document of the Invoice |
| `_l10n_sa_generate_invoice_xml_sha` | internal rule | self, xml_content | `l10n_sa_edi` |  | Transform, canonicalize then hash the invoice xml content using the SHA256 algorithm, then return the hashed content |
| `_l10n_sa_generate_invoice_xml_hash` | internal rule | self, xml_content, mode | `l10n_sa_edi` |  | Generate the b64 encoded sha256 hash of a given xml string:     - First: Transform the xml content using a pre-hash_invoice.xsl file     - Second: Canonicalize the transformed xml content using the c14n method     - Third: hash the canonicalized content using the sha256 algorithm then encode it into b64 format |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_21.zatca.json`.
