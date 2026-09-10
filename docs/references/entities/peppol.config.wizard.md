# Peppol Configuration Wizard (`peppol.config.wizard`)

**Transport name:** `peppol.config.wizard`  
**Storage name:** `peppol_config_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_peppol`

Description: Peppol Configuration Wizard

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `account_peppol_edi_user` | Account the pan-European public procurement online network Electronic data interchange User | many to one |  | related through path `company_id.account_peppol_edi_user` |
| `account_peppol_edi_identification` | Account the pan-European public procurement online network Electronic data interchange Identification | single line text |  | related through path `account_peppol_edi_user.edi_identification` |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `account_peppol_contact_email` | Account the pan-European public procurement online network Contact Email | single line text |  | required; default computed dynamically (lambda self: self.env.company.account_peppol_contact_email) |
| `account_peppol_migration_key` | Account the pan-European public procurement online network Migration Key | single line text |  | related through path `company_id.account_peppol_migration_key` |
| `peppol_activate_self_billing` | Activate self-billing | boolean |  | computed by rule `_compute_peppol_activate_self_billing` (not stored); writable through an inverse rule; Help: If activated, you will be able to send and receive self-billed invoices via Peppol.You can still disable reception by disabling the self-billing document types below. |
| `peppol_self_billing_reception_journal_id` | the pan-European public procurement online network Self Billing Reception Journal | many to one |  | related through path `company_id.peppol_self_billing_reception_journal_id` |
| `service_json` | Service JavaScript Object Notation | structured document |  | computed by rule `_compute_service_json` and stored; Help: JSON representation of peppol services as retrieved from the peppol server. |
| `service_info` | Service Info | rich text |  | computed by rule `_compute_service_info` (not stored) |
| `service_ids` | Service | one to many | `account_peppol.service` | computed by rule `_compute_service_ids` and stored; inverse field `wizard_id` |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`. Transitions are specified in the domain documents.

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_service_json` | computation | self | `account_peppol` | depends: `account_peppol_edi_user`, `account_peppol_proxy_state` |  |
| `_compute_service_info` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state` |  |
| `_compute_service_ids` | computation | self | `account_peppol` | depends: `account_peppol_proxy_state`, `service_json` | Get the selectable document types.  Synthesize a combination of locally available document types and those added to the user on the IAP, add the relevant services. |
| `_compute_peppol_activate_self_billing` | computation | self | `account_peppol` | depends: `company_id.peppol_activate_self_billing_sending` |  |
| `_inverse_peppol_activate_self_billing` | on change | self | `account_peppol` | onchange: `peppol_activate_self_billing` |  |
| `button_sync_form_with_peppol_proxy` | user action | self | `account_peppol` |  | Update the peppol contact email on IAP. Note: The service configuration is DEPRECATED / hidden in the view. Disabling services can lead to complicance issues and is not necessary since all existing services should just work. |
| `button_peppol_unregister` | user action | self | `account_peppol` |  | Unregister the user from Peppol network. |
| `button_peppol_reset_to_sender` | user action | self | `account_peppol` |  | Reset the participant back to sender and unregister it from the SMP |
| `button_peppol_register_sender_as_receiver` | user action | self | `account_peppol` |  | Reset the participant back to sender and unregister it from the SMP |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_peppol.peppol_config_wizard_form` | form |  | `company_id`, `account_peppol_edi_user`, `account_peppol_edi_identification`, `account_peppol_edi_identification`, `account_peppol_contact_email`, `account_peppol_migration_key`, `peppol_activate_self_billing`, `peppol_self_billing_reception_journal_id`, `service_info`, `service_ids`, `wizard_id`, `document_name`, `document_identifier`, `enabled` | `Remove from Peppol`, `Enable reception`, `Disable the reception.`, `Save`, `Discard` |  | `account_peppol` |

Machine-readable definition: `../../../schemas/data/entities/peppol.config.wizard.json`; views: `../../../schemas/interfaces/views/peppol.config.wizard.json`.
