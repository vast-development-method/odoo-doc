# France Universal Business Language 2.1 E-Invoicing Format (`account.edi.xml.ubl_21_fr`)

**Transport name:** `account.edi.xml.ubl_21_fr`  
**Storage name:** `account_edi_xml_ubl_21_fr`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_fr_pdp`

Description: France UBL 2.1 E-Invoicing Format

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_bis3`

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_pdp_can_invoice_b2g` | internal rule | self, customer | `l10n_fr_pdp` | model |  |
| `_pdp_is_b2g` | internal rule | self, customer | `l10n_fr_pdp` | model |  |
| `_pdp_needs_b2g_fields` | internal rule | self, customer | `l10n_fr_pdp` | model |  |
| `_export_invoice_filename` | internal rule | self, invoice | `l10n_fr_pdp` |  |  |
| `_export_invoice_constraints` | internal rule | self, invoice, vals | `l10n_fr_pdp` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_fr_pdp` |  |  |
| `_add_invoice_payment_means_nodes` | internal rule | self, document_node, vals | `l10n_fr_pdp` |  |  |
| `_ubl_add_party_identification_nodes` | internal rule | self, vals | `l10n_fr_pdp` |  |  |
| `_ubl_add_party_legal_entity_nodes` | internal rule | self, vals | `l10n_fr_pdp` |  |  |
| `_ubl_add_line_price_node` | internal rule | self, vals, in_foreign_currency | `l10n_fr_pdp` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl_21_fr.json`.
