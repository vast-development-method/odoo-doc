# Malaysian implementation of Peppol International (PINT) model for Billing (`account.edi.xml.pint_my`)

**Transport name:** `account.edi.xml.pint_my`  
**Storage name:** `account_edi_xml_pint_my`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_my_ubl_pint`

Description: Malaysian implementation of Peppol International (PINT) model for Billing

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_my_ubl_pint` |  |  |
| `_get_customization_id` | preparation rule | self, process_type | `l10n_my_ubl_pint` |  |  |
| `_ubl_default_tax_category_grouping_key` | internal rule | self, base_line, tax_data, vals, currency | `l10n_my_ubl_pint` |  |  |
| `_ubl_add_tax_totals_nodes` | internal rule | self, vals | `l10n_my_ubl_pint` |  |  |
| `_ubl_add_party_tax_scheme_nodes` | internal rule | self, vals | `l10n_my_ubl_pint` |  |  |
| `_ubl_add_accounting_supplier_party_tax_scheme_nodes` | internal rule | self, vals | `l10n_my_ubl_pint` |  |  |
| `_ubl_add_customization_id_node` | internal rule | self, vals | `l10n_my_ubl_pint` |  |  |
| `_ubl_add_profile_id_node` | internal rule | self, vals | `l10n_my_ubl_pint` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_my_ubl_pint` |  |  |
| `_import_ubl_invoice_add_customer_values` | internal rule | self, collected_values | `l10n_my_ubl_pint` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.pint_my.json`.
