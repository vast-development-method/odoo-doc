# Malaysian implementation of ubl for the MyInvois portal (`account.edi.xml.ubl_myinvois_my`)

**Transport name:** `account.edi.xml.ubl_myinvois_my`  
**Storage name:** `account_edi_xml_ubl_myinvois_my`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_my_edi`  
**Extended by packages:** `l10n_my_edi_pos`

Description: Malaysian implementation of ubl for the MyInvois portal

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (44)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_myinvois_document_node` | preparation rule | self, vals | `l10n_my_edi` |  | Entry point of the export of a MyInvois document. The node returned by this function should be passed into dict_to_xml in order to generate the XML file to send to MyInvois. |
| `_add_myinvois_document_config_vals` | internal rule | self, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_base_lines_vals` | internal rule | self, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_tax_grouping_function_vals` | internal rule | self, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_monetary_total_vals` | internal rule | self, vals | `l10n_my_edi_pos`, `l10n_my_edi` |  |  |
| `_get_myinvois_document_address_node` | preparation rule | self, vals | `l10n_my_edi` |  |  |
| `_get_myinvois_document_party_identification_node` | preparation rule | self, vals | `l10n_my_edi` |  | The id vals list must be filled with two values. The TIN, and then one of either:     - Business registration number (BNR)     - MyKad/MyTentera identification number (NRIC)     - Passport number or MyPR/MyKAS identification number (PASSPORT)     - (ARMY) Additionally, companies registered to use SST (sales & services tax) must provide their SST number. Finally, if a supplier is using TTX (tourism tax), once again that number must be provided. |
| `_get_myinvois_document_party_node` | preparation rule | self, vals | `l10n_my_edi` |  |  |
| `_get_tax_category_node` | preparation rule | self, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_header_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_delivery_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_payment_terms_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_exchange_rate_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos`, `l10n_my_edi` |  |  |
| `_add_myinvois_document_line_nodes` | internal rule | self, document_node, vals | `l10n_my_edi` |  |  |
| `_get_myinvois_document_line_node` | preparation rule | self, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_line_vals` | internal rule | self, vals | `l10n_my_edi` |  | Generic helper to calculate the amounts for a document line. |
| `_add_myinvois_document_line_gross_subtotal_and_discount_vals` | internal rule | self, vals | `l10n_my_edi` |  | As we group lines together when consolidating, we lose the discount percentage in the process. During the grouping, we stored the actual amount in the base line, se we will override here in order to use that pre-computed amount. |
| `_add_myinvois_document_line_amount_nodes` | internal rule | self, line_node, vals | `l10n_my_edi` |  |  |
| `_add_myinvois_document_line_item_nodes` | internal rule | self, line_node, vals | `l10n_my_edi` |  |  |
| `_export_myinvois_document_constraints` | internal rule | self, vals | `l10n_my_edi` |  |  |
| `_l10n_my_edi_make_validation_error` | internal rule | self, constraints, code, record_identifier, record_name | `l10n_my_edi` | model | Small helper that add new constrains into provided constrains dict. This helper is mainly there to keep the check method tidy, and focused on its purpose (validating data) |
| `_l10n_my_edi_decode_myinvois_attachment` | internal rule | self, attachment | `l10n_my_edi` | model | Extract data from MyInvois xml. |
| `_l10n_my_edi_get_document_type_code` | internal rule | self, myinvois_document | `l10n_my_edi` | model | Returns the code matching the invoice type, as well as the original document if any. |
| `_l10n_my_edi_get_refund_details` | internal rule | self, invoice | `l10n_my_edi_pos`, `l10n_my_edi` | model | Helper which returns the refunded document in case of out_refund/in_refund. In some cases, such as PoS, we could need a different logic than from the regular flow. :param invoice: The credit note for which we want to get the refunded document. :return: A tuple, where the first parameter indicates if this credit note is a refund and the second the credited/refunded document. |
| `_l10n_my_edi_get_prepaid_amount` | internal rule | self, invoice | `l10n_my_edi` | model | Compute the amount of the invoice that was genuinely paid in advance.  LHDN only recognizes a reconciled payment as a deposit/prepayment if it was made strictly before the invoice date; a payment made on or after the invoice date is a regular settlement, not a prepayment. Additionally, LHDN never accepts a PayableAmount of 0, so a 100% advance payment must not be reported as a prepayment either: in that case, the full invoice amount is reported as payable instead. Credit/debit notes reconciled against the invoice are not prepayments. |
| `_l10n_my_edi_get_formatted_phone_number` | internal rule | self, number | `l10n_my_edi` | model |  |
| `_get_consolidated_invoice_node` | preparation rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_config_vals` | internal rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_base_lines_vals` | internal rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_monetary_total_vals` | internal rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_add_document_tax_grouping_function_vals` | internal rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_accounting_customer_party_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos` |  |  |
| `_get_consolidated_invoice_party_node` | preparation rule | self, vals | `l10n_my_edi_pos` |  |  |
| `_get_address_node` | preparation rule | self, vals | `l10n_my_edi_pos` |  | Generic helper to generate the Address node for a res.partner or res.bank. |
| `_add_consolidated_invoice_monetary_total_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_consolidated_invoice_line_nodes` | internal rule | self, document_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_document_line_item_nodes` | internal rule | self, line_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_document_line_amount_nodes` | internal rule | self, line_node, vals | `l10n_my_edi_pos` |  |  |
| `_add_document_line_gross_subtotal_and_discount_vals` | internal rule | self, vals | `l10n_my_edi_pos` |  | As we group lines together, we lose the discount percentage in the process. During the grouping, we stored the actual amount in the base line, se we will override here in order to use that pre-computed amount. |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_myinvois_my.json`.
