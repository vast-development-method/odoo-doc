# Universal Business Language PINT-EU Layer (`account.edi.ubl_pint_eu`)

**Transport name:** `account.edi.ubl_pint_eu`  
**Storage name:** `account_edi_ubl_pint_eu`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `account_edi_ubl_cii`

Description: UBL PINT-EU Layer

## Identity and behavior

- Mixins (classical inheritance): `account.edi.ubl_pint`, `account.edi.ubl_cen_en16931`

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_ubl_add_customization_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_profile_id_node` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_get_delivery_node_from_delivery_address` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_payment_means_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_ubl_add_billing_reference_nodes` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |
| `_export_document_node_constraints` | internal rule | self, vals | `account_edi_ubl_cii` |  |  |

Machine-readable definition: `../../../schemas/data/entities/account.edi.ubl_pint_eu.json`.
