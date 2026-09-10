# Universal Business Language 2.1 (RS eFaktura) (`account.edi.xml.ubl.rs`)

**Transport name:** `account.edi.xml.ubl.rs`  
**Storage name:** `account_edi_xml_ubl_rs`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `l10n_rs_edi`

Description: UBL 2.1 (RS eFaktura)

## Identity and behavior

- Mixins (classical inheritance): `account.edi.xml.ubl_21`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_customization_id` | preparation rule | self, process_type | `l10n_rs_edi` |  |  |
| `_add_invoice_header_nodes` | internal rule | self, document_node, vals | `l10n_rs_edi` |  |  |
| `_get_party_node` | preparation rule | self, vals | `l10n_rs_edi` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.xml.ubl.rs.json`.
