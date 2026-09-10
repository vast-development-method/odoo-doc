# Update the probabilities (`crm.lead.pls.update`)

**Transport name:** `crm.lead.pls.update`  
**Storage name:** `crm_lead_pls_update`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `crm`

Description: Update the probabilities

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pls_start_date` | Pls Start Date | date |  | required; default computed dynamically (_get_default_pls_start_date) |
| `pls_fields` | Pls Fields | many to many | `crm.lead.scoring.frequency.field` | default computed dynamically (_get_default_pls_fields) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_pls_start_date` | preparation rule | self | `crm` |  |  |
| `_get_default_pls_fields` | preparation rule | self | `crm` |  |  |
| `action_update_crm_lead_probabilities` | user action | self | `crm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | yes | `crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_pls_update_view_form` | form |  | `pls_fields`, `pls_start_date` | `Update`, `Discard` |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_pls_update_action` | Update Probabilities | form |  |  | new | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.pls.update.json`; views: `../../../schemas/interfaces/views/crm.lead.pls.update.json`.
