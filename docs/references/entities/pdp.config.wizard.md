# Peppol Configuration Wizard (`pdp.config.wizard`)

**Transport name:** `pdp.config.wizard`  
**Storage name:** `pdp_config_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_fr_pdp`

Description: Peppol Configuration Wizard

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_account_peppol_edi_user` (not stored) |
| `account_peppol_edi_identification` | Account the pan-European public procurement online network Electronic data interchange Identification | single line text |  | related through path `account_peppol_edi_user.edi_identification` |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `account_peppol_contact_email` | Account the pan-European public procurement online network Contact Email | single line text |  | required; related through path `company_id.account_peppol_contact_email` |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`. Transitions are specified in the domain documents.

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_account_peppol_edi_user` | computation | self | `l10n_fr_pdp` | depends: `company_id` |  |
| `_action_open` | internal rule | self | `l10n_fr_pdp` |  |  |
| `button_sync_form_with_peppol_proxy` | user action | self | `l10n_fr_pdp` |  | Update the peppol contact email on IAP. Note: The service configuration is DEPRECATED / hidden in the view. Disabling services can lead to complicance issues and is not necessary since all existing services should just work. |
| `button_peppol_unregister` | user action | self | `l10n_fr_pdp` |  | Unregister the user from Peppol network. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_fr_pdp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_config_wizard_form` | form |  | `account_peppol_edi_identification`, `account_peppol_edi_identification`, `account_peppol_contact_email` | `Disconnect French electronic invoicing`, `Save`, `Discard` |  | `l10n_fr_pdp` |

Machine-readable definition: `../../../schemas/data/entities/pdp.config.wizard.json`; views: `../../../schemas/interfaces/views/pdp.config.wizard.json`.
