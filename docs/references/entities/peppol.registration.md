# Peppol Registration (`peppol.registration`)

**Transport name:** `peppol.registration`  
**Storage name:** `peppol_registration`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `account_peppol`

Description: Peppol Registration

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `parent_company_id` | Parent Company | many to one | `res.company` | computed by rule `_compute_parent_company_id` (not stored) |
| `parent_company_name` | Parent Company Name | single line text |  | related through path `parent_company_id.name` |
| `selected_company_id` | Selected Company | many to one | `res.company` | computed by rule `_compute_selected_company_id` (not stored) |
| `display_use_parent_connection_selection` | Display Use Parent Connection Selection | boolean |  | computed by rule `_compute_display_use_parent_connection_selection` (not stored) |
| `use_parent_connection_selection` | Use Parent Connection Selection | selection |  | computed by rule `_compute_use_parent_connection_selection` and stored |
| `use_parent_connection` | Use Parent Connection | boolean |  | computed by rule `_compute_use_parent_connection` (not stored) |
| `is_branch_company` | Is Branch Company | boolean |  |  |
| `active_parent_company` | Active Parent Company | many to one |  | related through path `parent_company_id` |
| `active_parent_company_name` | Active Parent Company Name | single line text |  | related through path `parent_company_name` |
| `can_use_parent_connection` | Can Use Parent Connection | boolean |  | related through path `display_use_parent_connection_selection` |
| `edi_mode` | electronic data interchange mode | selection |  | computed by rule `_compute_edi_mode` (not stored) |
| `edi_user_id` | electronic data interchange user | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_edi_user_id` (not stored) |
| `account_peppol_proxy_state` | Account the pan-European public procurement online network Proxy State | selection |  | related through path `company_id.account_peppol_proxy_state` |
| `peppol_eas` | the pan-European public procurement online network Eas | selection |  | required; computed by rule `_compute_peppol_eas` (not stored); writable through an inverse rule; values provided by rule `_get_peppol_eas_selection` |
| `peppol_warnings` | Peppol warnings | structured document |  | computed by rule `_compute_peppol_warnings` (not stored) |
| `contact_email` | Contact Email | single line text |  | required; related through path `selected_company_id.account_peppol_contact_email` |
| `phone_number` | Phone Number | single line text |  | related through path `selected_company_id.account_peppol_phone_number` |
| `peppol_endpoint` | the pan-European public procurement online network Endpoint | single line text |  | required; related through path `selected_company_id.peppol_endpoint` |
| `smp_registration` | Register as a receiver | boolean |  | computed by rule `_compute_smp_registration_external_provider` (not stored) |
| `peppol_external_provider` | the pan-European public procurement online network External Provider | single line text |  | computed by rule `_compute_smp_registration_external_provider` (not stored) |
| `peppol_can_connect_data` | the pan-European public procurement online network Can Connect Data | structured document |  | computed by rule `_compute_peppol_can_connect_data` (not stored) |
| `display_itsme_login` | Display Itsme Login | boolean |  | computed by rule `_compute_peppol_can_connect_data` (not stored) |
| `display_no_auth_buttons` | Display No Auth Buttons | boolean |  | computed by rule `_compute_peppol_can_connect_data` (not stored) |

## Selection values

### `use_parent_connection_selection` (Use Parent Connection Selection)

| Value | Label |
|---|---|
| `use_parent` | Send from parent company |
| `use_self` | Register this company on peppol |

### `edi_mode` (electronic data interchange mode)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

## State fields

State machine fields of this entity: `account_peppol_proxy_state`. Transitions are specified in the domain documents.

## Operations (28)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_peppol_endpoint` | on change | self | `account_peppol` | onchange: `peppol_endpoint` |  |
| `_onchange_phone_number` | on change | self | `account_peppol` | onchange: `phone_number` |  |
| `_compute_parent_company_id` | computation | self | `account_peppol` | depends: `company_id` |  |
| `_compute_display_use_parent_connection_selection` | computation | self | `account_peppol` | depends: `parent_company_id` |  |
| `_compute_use_parent_connection_selection` | computation | self | `account_peppol` | depends: `display_use_parent_connection_selection` |  |
| `_compute_use_parent_connection` | computation | self | `account_peppol` | depends: `use_parent_connection_selection` |  |
| `_compute_selected_company_id` | computation | self | `account_peppol` | depends: `use_parent_connection` |  |
| `_compute_edi_user_id` | computation | self | `account_peppol` | depends: `selected_company_id.account_edi_proxy_client_ids` |  |
| `_compute_peppol_warnings` | computation | self | `account_peppol` | depends: `selected_company_id`, `peppol_eas`, `peppol_endpoint`, `smp_registration`, `peppol_external_provider`, `use_parent_connection` |  |
| `_compute_edi_mode` | computation | self | `account_peppol` | depends: `selected_company_id`, `edi_user_id`, `peppol_eas` |  |
| `_compute_smp_registration_external_provider` | computation | self | `account_peppol` | depends: `selected_company_id`, `peppol_eas`, `peppol_endpoint` |  |
| `_compute_peppol_can_connect_data` | computation | self | `account_peppol` | depends: `peppol_eas`, `peppol_endpoint` |  |
| `_compute_peppol_eas` | computation | self | `account_peppol` | depends: `selected_company_id.peppol_eas` |  |
| `_inverse_peppol_eas` | inverse computation | self | `account_peppol` |  |  |
| `_get_peppol_eas_selection` | preparation rule | self | `account_peppol` |  |  |
| `_branch_with_same_address` | internal rule | self | `account_peppol` |  |  |
| `_ensure_mandatory_fields` | internal rule | self | `account_peppol` |  |  |
| `_ensure_pdp_not_sent_through_peppol` | internal rule | self | `account_peppol` |  |  |
| `_action_open_peppol_form` | internal rule | self, reopen | `account_peppol` |  |  |
| `_action_send_notification` | internal rule | self, title, message | `account_peppol` |  |  |
| `_ensure_can_connect` | internal rule | self, can_connect_vals, selected_auth | `account_peppol` | model | Checks the answer from the /can_connect endpoint and raises an error if it's invalid. |
| `_generate_connect_token` | internal rule | self, peppol_identifier, company | `account_peppol` | model |  |
| `_decode_connect_token` | internal rule | self, token | `account_peppol` | model |  |
| `_can_connect` | internal rule | self | `account_peppol` |  |  |
| `_create_connection` | internal rule | self, peppol_identifier, db_uuid, company, auth_token | `account_peppol` | model |  |
| `_get_company_details` | preparation rule | self, company | `account_peppol` | model |  |
| `button_register_with_itsme` | user action | self | `account_peppol` |  |  |
| `button_register_peppol_participant` | user action | self, selected_auth | `account_peppol` |  |  |

## Validation and error messages (13)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_ensure_mandatory_fields` | ValidationError | Please select a country for your company. | `account_peppol` |
| `_ensure_mandatory_fields` | ValidationError | Contact email and phone number are required. | `account_peppol` |
| `_ensure_mandatory_fields` | ValidationError | Peppol Address should be provided. | `account_peppol` |
| `_ensure_mandatory_fields` | ValidationError | Peppol ID should be different from main company. | `account_peppol` |
| `_ensure_mandatory_fields` | ValidationError | Cannot register a user with a %s application | `account_peppol` |
| `_ensure_can_connect` | UserError | Could not connect to Proxy Server. | `account_peppol` |
| `_ensure_can_connect` | UserError | Your identifier is invalid. | `account_peppol` |
| `_ensure_can_connect` | UserError | The database you are trying to connect to is not suitable for Peppol. | `account_peppol` |
| `_ensure_can_connect` | UserError | You need to authenticate to continue. | `account_peppol` |
| `_ensure_can_connect` | UserError | Selected authentication method is not available. | `account_peppol` |
| `_ensure_can_connect` | UserError | Your identifier you entered is invalid for Peppol. | `account_peppol` |
| `_ensure_can_connect` | UserError | Your identifier does not have a valid format.%s | `account_peppol` |
| `button_register_peppol_participant` | UserError | A connection to '%s' already exists. | `account_peppol` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `account_peppol` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account_peppol.peppol_registration_form` | form |  | `peppol_warnings`, `parent_company_name`, `use_parent_connection_selection`, `peppol_eas`, `peppol_endpoint`, `contact_email`, `phone_number` | `button_register_with_itsme`, `Activate Peppol`, `Activate Peppol (Test)`, `Activate Peppol (Demo)`, `Cancel` |  | `account_peppol` |

Machine-readable definition: `../../../schemas/data/entities/peppol.registration.json`; views: `../../../schemas/interfaces/views/peppol.registration.json`.
