# text message Account Sender Name Wizard (`sms.account.sender`)

**Transport name:** `sms.account.sender`  
**Storage name:** `sms_account_sender`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms`

Description: SMS Account Sender Name Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `account_id` | Account | many to one | `iap.account` | required |
| `sender_name` | Sender Name | single line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_sender_name` | validation | self | `sms` | constrains: `sender_name` |  |
| `action_set_sender_name` | user action | self | `sms` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_sender_name` | ValidationError | Your sender name must be between 3 and 11 characters long and only contain alphanumeric characters. | `sms` |
| `action_set_sender_name` | ValidationError | ERROR_MESSAGES.get(status, ERROR_MESSAGES['unknown_error']) | `sms` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms.sms_account_sender_view_form` | form |  | `sender_name` | `Set sender name`, `Skip for now` |  | `sms` |

Machine-readable definition: `../../../schemas/data/entities/sms.account.sender.json`; views: `../../../schemas/interfaces/views/sms.account.sender.json`.
