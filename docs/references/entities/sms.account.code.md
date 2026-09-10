# text message Account Verification Code Wizard (`sms.account.code`)

**Transport name:** `sms.account.code`  
**Storage name:** `sms_account_code`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`

Description: SMS Account Verification Code Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Account | many to one | `iap.account` | required |
| `verification_code` | Verification Code | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_register` | user action | self | `sms` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_register` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) | `sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_account_code_view_form` | form |  | `verification_code` | `Register`, `Cancel` |  | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.account.code.json`; views: `../../../schemas/interfaces/views/sms.account.code.json`.
