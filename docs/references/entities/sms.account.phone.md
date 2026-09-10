# text message Account Registration Phone Number Wizard (`sms.account.phone`)

**Transport name:** `sms.account.phone`  
**Storage name:** `sms_account_phone`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`

Description: SMS Account Registration Phone Number Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Account | many to one | `iap.account` | required |
| `phone_number` | Phone Number | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_send_verification_code` | user action | self | `sms` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_send_verification_code` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) | `sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_account_phone_view_form` | form |  | `phone_number` | `Send verification code`, `Cancel` |  | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.account.phone.json`; views: `../../../schemas/interfaces/views/sms.account.phone.json`.
