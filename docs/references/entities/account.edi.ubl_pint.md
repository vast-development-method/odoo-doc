# Universal Business Language PINT (`account.edi.ubl_pint`)

**Transport name:** `account.edi.ubl_pint`  
**Storage name:** `account_edi_ubl_pint`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: UBL PINT

## Identity and behavior

- Mixins (classical inheritance): `account.edi.ubl`

## Operations (30)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_ubl_add_invoice_type_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_credit_note_type_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_notes_nodes_all_invoices` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_notes_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_document_currency_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_tax_currency_code_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_buyer_reference_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_billing_reference_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_get_partner_address_node` | internal rule | self, vals, partner | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_endpoint_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_identification_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_tax_scheme_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_party_legal_entity_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_accounting_supplier_party_tax_scheme_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_endpoint_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_identification_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_postal_address_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_tax_scheme_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_legal_entity_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_delivery_party_contact_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_get_payment_means_payee_financial_account_institution_branch_node_from_partner_bank` | internal rule | self, vals, partner_bank | `account_edi_ubl_cii` |  |  |
| `_ubl_add_payment_means_nodes_all_invoices` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_payment_means_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_get_tax_subtotal_node` | internal rule | self, vals, tax_subtotal | `account_edi_ubl_cii` |  |  |
| `_ubl_tax_totals_node_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `account_edi_ubl_cii` |  |  |
| `_ubl_add_legal_monetary_total_payable_rounding_amount_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_legal_monetary_total_prepaid_payable_amount_node` | internal rule | self, vals, in_foreign_currency | `account_edi_ubl_cii` |  |  |
| `_init_invoice_export_values` | internal rule | self, invoice | `account_edi_ubl_cii` |  |  |
| `_import_prepare_missing_customer_create_values` | internal rule | self, collected_values | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.ubl_pint.json`.
