# PDP Registration (`pdp.registration`)

**Transport name:** `pdp.registration`  
**Storage name:** `pdp_registration`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_fr_pdp`

Description: PDP Registration

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `contact_email` | Contact Email | single line text |  | required; related through path `company_id.account_peppol_contact_email` |
| `pdp_identifier` | Pdp Identifier | single line text |  | required; computed by rule `_compute_pdp_identifier` (not stored); writable through an inverse rule; Help: The identifier starts with the SIREN, the part after the SIREN is optional. The expected format of the identifier is: SIREN, SIREN_SIRET, SIREN_SIRET_CodeRoutage or SIREN_SuffixeAdressage |
| `pdp_pilot_phase` | Pilot Phase | boolean |  | related through path `company_id.l10n_fr_pdp_pilot_phase`; Help: Participate in the Pilot Phase of the French E-Invoicing. This way you are able to test it before it becomes mandatory. |
| `edi_mode` | electronic data interchange mode | selection |  | computed by rule `_compute_edi_mode` (not stored) |
| `edi_user_id` | electronic data interchange user | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_edi_user_id` (not stored) |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `warnings` | Warnings | structured document |  | computed by rule `_compute_warnings` (not stored) |
| `siren_number` | Siren Number | single line text |  | computed by rule `_compute_siren_number` and stored |
| `pdp_authentication_uuid` | Authentication in-app purchase UUID | single line text |  | related through path `company_id.pdp_authentication_uuid` and stored |
| `pdp_kyc_status` | Authentication status | selection |  | related through path `company_id.pdp_kyc_status` |
| `auth_url_hash` | Company Authentication in-app purchase UUID | single line text |  | related through path `company_id.pdp_authentication_uuid` |

## Selection values

### `edi_mode` (electronic data interchange mode)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`, `pdp_kyc_status`. Transitions are specified in the domain documents.

## Operations (21)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_pdp_identifier` | on change | self | `l10n_fr_pdp` | onchange: `pdp_identifier` |  |
| `_compute_pdp_identifier` | computation | self | `l10n_fr_pdp` | depends: `company_id.pdp_identifier` |  |
| `_inverse_pdp_identifier` | inverse computation | self | `l10n_fr_pdp` |  |  |
| `_compute_siren_number` | computation | self | `l10n_fr_pdp` | depends: `pdp_identifier` |  |
| `_compute_edi_user_id` | computation | self | `l10n_fr_pdp` | depends: `company_id.account_edi_proxy_client_ids` |  |
| `_compute_edi_mode` | computation | self | `l10n_fr_pdp` | depends: `edi_user_id` |  |
| `_compute_warnings` | computation | self | `l10n_fr_pdp` | depends: `pdp_identifier`, `siren_number` |  |
| `_get_kyc_siren` | preparation rule | self | `l10n_fr_pdp` |  |  |
| `_ensure_mandatory_fields` | internal rule | self | `l10n_fr_pdp` |  |  |
| `_action_send_notification` | internal rule | self, title, message | `l10n_fr_pdp` |  |  |
| `_check_can_register` | validation | self | `l10n_fr_pdp` |  | No longer used, need to remove in master |
| `_action_open_pdp_form` | internal rule | self, reopen | `l10n_fr_pdp` |  |  |
| `button_trigger_authentication` | user action | self | `l10n_fr_pdp` |  |  |
| `_get_status_notification_data` | preparation rule | self | `l10n_fr_pdp` |  |  |
| `_display_status_notification` | internal rule | self | `l10n_fr_pdp` |  |  |
| `display_status_notification_from_uuid` | operation | self | `l10n_fr_pdp` |  |  |
| `button_refresh_authentication` | user action | self | `l10n_fr_pdp` |  |  |
| `button_open_authentication_link` | user action | self | `l10n_fr_pdp` |  |  |
| `button_cancel_authentication` | user action | self | `l10n_fr_pdp` |  |  |
| `button_register_pdp_participant` | user action | self | `l10n_fr_pdp` |  |  |
| `button_deregister_pdp_participant` | user action | self | `l10n_fr_pdp` |  | Deregister the edi user from PDP network |

## Validation and error messages (9)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_ensure_mandatory_fields` | ValidationError | The contact email is required. | `l10n_fr_pdp` |
| `button_trigger_authentication` | ValidationError | Invalid email address '%s' | `l10n_fr_pdp` |
| `button_trigger_authentication` | UserError | error | `l10n_fr_pdp` |
| `button_trigger_authentication` | UserError | Something wrong happened. | `l10n_fr_pdp` |
| `button_open_authentication_link` | UserError | error | `l10n_fr_pdp` |
| `button_open_authentication_link` | UserError | Something wrong happened. | `l10n_fr_pdp` |
| `button_register_pdp_participant` | UserError | Cannot register a user with a '%s' application | `l10n_fr_pdp` |
| `button_register_pdp_participant` | UserError | There is a connection to Peppol (non-PA) already | `l10n_fr_pdp` |
| `button_register_pdp_participant` | UserError | The Identifier is not valid. The expected format is: SIREN, SIREN_SIRET, SIREN_SIRET_CodeRoutage or SIREN_SuffixeAdressage | `l10n_fr_pdp` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_manager` | yes | yes | yes | yes | `l10n_fr_pdp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_fr_pdp.pdp_registration_form` | form |  | `warnings`, `pdp_identifier`, `contact_email` | `Authenticate`, `Refresh`, `Cancel`, `Open link`, `button_register_pdp_participant`, `Cancel Registration` |  | `l10n_fr_pdp` |

Machine-readable definition: `../../../schemas/data/entities/pdp.registration.json`; views: `../../../schemas/interfaces/views/pdp.registration.json`.
