# Nemhandel Registration (`nemhandel.registration`)

**Transport name:** `nemhandel.registration`  
**Storage name:** `nemhandel_registration`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `l10n_dk_nemhandel`

Description: Nemhandel Registration

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `contact_email` | Contact Email | single line text |  | required; related through path `company_id.nemhandel_contact_email` |
| `edi_mode` | electronic data interchange mode | selection |  | computed by rule `_compute_edi_mode` (not stored); writable through an inverse rule |
| `edi_user_id` | electronic data interchange user | many to one | `account_edi_proxy_client.user` | computed by rule `_compute_edi_user_id` (not stored) |
| `phone_number` | Phone Number | single line text |  | related through path `company_id.nemhandel_phone_number`; writable through an inverse rule |
| `l10n_dk_nemhandel_proxy_state` | Localization Dk Nemhandel Proxy State | selection |  | related through path `company_id.l10n_dk_nemhandel_proxy_state` |
| `verification_code` | Verification Code | single line text |  | related through path `edi_user_id.nemhandel_verification_code` |
| `identifier_type` | Identifier Type | selection |  | required; related through path `company_id.nemhandel_identifier_type` |
| `identifier_value` | Identifier Value | single line text |  | required; related through path `company_id.nemhandel_identifier_value` |

## Selection values

### `edi_mode` (electronic data interchange mode)

| Value | Label |
|---|---|
| `demo` | Demo |
| `test` | Test |
| `prod` | Live |

## State fields

State machine fields of this entity: `l10n_dk_nemhandel_proxy_state`. Transitions are specified in the domain documents.

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_identifier_value` | on change | self | `l10n_dk_nemhandel` | onchange: `identifier_value` |  |
| `_onchange_phone_number` | on change | self | `l10n_dk_nemhandel` | onchange: `phone_number` |  |
| `_compute_edi_user_id` | computation | self | `l10n_dk_nemhandel` | depends: `company_id.account_edi_proxy_client_ids` |  |
| `_compute_edi_mode` | computation | self | `l10n_dk_nemhandel` | depends: `edi_user_id` |  |
| `_inverse_edi_mode` | inverse computation | self | `l10n_dk_nemhandel` |  |  |
| `_action_open_nemhandel_form` | internal rule | self, reopen | `l10n_dk_nemhandel` |  |  |
| `_action_send_notification` | internal rule | self, title, message | `l10n_dk_nemhandel` |  |  |
| `button_nemhandel_registration_sms` | user action | self | `l10n_dk_nemhandel` |  | The first step of the Nemhandel onboarding. - Creates an EDI proxy user on the iap side, then the client side - Calls /activate_participant to mark the EDI user as nemhandel user - Sends an SMS code |
| `button_nemhandel_receiver_registration` | user action | self | `l10n_dk_nemhandel` |  | The user is registered on the Nemhandel network, i.e. can receive documents from other Nemhandel participants. |
| `button_update_nemhandel_user_data` | user action | self | `l10n_dk_nemhandel` |  | Action for the user to be able to update their contact details any time Calls /update_user on the iap server |
| `send_nemhandel_verification_code` | operation | self | `l10n_dk_nemhandel` |  | Request user verification via SMS Calls the /send_verification_code to activate the participant and send the 6-digit verification code |
| `button_check_nemhandel_verification_code` | user action | self | `l10n_dk_nemhandel` |  | Calls /verify_phone_number to compare user's input and the code generated on the IAP server |
| `button_deregister_nemhandel_participant` | user action | self | `l10n_dk_nemhandel` |  | Deregister the edi user from Nemhandel network |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `button_nemhandel_registration_sms` | UserError | Cannot register a user with a %s application | `l10n_dk_nemhandel` |
| `button_nemhandel_registration_sms` | ValidationError | Please enter a phone number to verify your application. | `l10n_dk_nemhandel` |
| `button_nemhandel_registration_sms` | ValidationError | Please enter a primary contact email to verify your application. | `l10n_dk_nemhandel` |
| `button_nemhandel_registration_sms` | RedirectWarning | Please fill in your company's VAT | `l10n_dk_nemhandel` |
| `button_update_nemhandel_user_data` | ValidationError | Contact email and phone number are required. | `l10n_dk_nemhandel` |
| `button_check_nemhandel_verification_code` | ValidationError | Please first verify your phone number by clicking on 'Send a registration code by SMS'. | `l10n_dk_nemhandel` |
| `button_check_nemhandel_verification_code` | ValidationError | The verification code should contain six digits. | `l10n_dk_nemhandel` |
| `button_check_nemhandel_verification_code` | UserError | errors.get(error_code) or _('Connection error, please try again later.') | `l10n_dk_nemhandel` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_invoice` | yes | yes | yes | yes | `l10n_dk_nemhandel` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_dk_nemhandel.nemhandel_registration_form` | form |  | `identifier_type`, `identifier_value`, `contact_email`, `phone_number`, `phone_number`, `verification_code` | `Activate Nemhandel`, `Activate Nemhandel (Test)`, `Activate Nemhandel (Demo)`, `Confirm`, `Send again`, `Cancel Registration` |  | `l10n_dk_nemhandel` |

Machine-readable definition: `../../../schemas/data/entities/nemhandel.registration.json`; views: `../../../schemas/interfaces/views/nemhandel.registration.json`.
