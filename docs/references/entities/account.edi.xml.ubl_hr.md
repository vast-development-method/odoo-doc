# CIUS human resources (`account.edi.xml.ubl_hr`)

**Transport name:** `account.edi.xml.ubl_hr`  
**Storage name:** `account_edi_xml_ubl_hr`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_hr_edi`

Description: CIUS HR

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_hr_edi` |  |  |
| `_get_document_template` | preparation rule | self, vals | `l10n_hr_edi` |  |  |
| `_get_document_nsmap` | preparation rule | self, vals | `l10n_hr_edi` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_hr_edi` |  |  |
| `_invoice_constraints_eracun_new` | internal rule | self, invoice, vals | `l10n_hr_edi` |  |  |
| `_get_invoice_node` | preparation rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_id_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_profile_id_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_copy_indicator_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_issue_date_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_invoice_type_code_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_credit_note_type_code_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_billing_reference_nodes` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_add_hr_extension_node` | internal rule | self, document_node | `l10n_hr_edi` |  | This function constructs hrextac node from existing data within the document. The structure mostly follows that of 'cac:TaxTotal' node of a UBL 2.1/BIS 3 document, but requires additional data compared to the totals/subtotals nodes in UBL HR format. To avoid making additional queries and possible desyncs, we calculate all the data we need while assembling normal subtotals, then trim out the extra bits. |
| `_ubl_add_party_endpoint_id_node` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_ubl_add_party_identification_nodes` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_add_invoice_accounting_supplier_party_nodes` | internal rule | self, document_node, vals | `l10n_hr_edi` |  |  |
| `_ubl_default_tax_category_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `l10n_hr_edi` |  |  |
| `_ubl_get_tax_category_node` | internal rule | self, vals, tax_category | `l10n_hr_edi` |  |  |
| `_ubl_get_line_item_node_classified_tax_category_node` | internal rule | self, vals, tax_category | `l10n_hr_edi` |  |  |
| `_setup_base_lines` | internal rule | self, vals | `l10n_hr_edi` |  |  |
| `_import_ubl_invoice_write_collected_values` | internal rule | self, collected_values | `l10n_hr_edi` |  |  |
| `_import_ubl_invoice_line_prepare_classified_tax_category_tax_values` | internal rule | self, collected_values, tax_category_tree | `l10n_hr_edi` |  |  |
| `_retrieve_rejection_reference` | internal rule | self, attachment | `l10n_hr_edi` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_hr.json`.
