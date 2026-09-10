# text message Twilio Connection Wizard (`sms.twilio.account.manage`)

**Transport name:** `sms.twilio.account.manage`  
**Storage name:** `sms_twilio_account_manage`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `sms_twilio`

Description: SMS Twilio Connection Wizard

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `sms_provider` | Text message Provider | selection |  | related through path `company_id.sms_provider` |
| `sms_twilio_account_sid` | Text message Twilio Account Sid | single line text |  | related through path `company_id.sms_twilio_account_sid` |
| `sms_twilio_auth_token` | Text message Twilio Auth Token | single line text |  | related through path `company_id.sms_twilio_auth_token` |
| `sms_twilio_number_ids` | Text message Twilio Number | one to many |  | related through path `company_id.sms_twilio_number_ids` |
| `test_number` | Test Number | single line text |  |  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_reload_numbers` | user action | self | `sms_twilio` |  | Fetch the available numbers from Twilio account |
| `action_send_test` | user action | self | `sms_twilio` |  |  |
| `action_save` | user action | self | `sms_twilio` |  |  |
| `_display_notification` | internal rule | self, notif_type, message | `sms_twilio` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_send_test` | UserError | Please set the number to which you want to send a test SMS. | `sms_twilio` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `sms_twilio` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sms_twilio.sms_twilio_account_manage_view_form` | form |  | `company_id`, `sms_twilio_account_sid`, `sms_twilio_auth_token`, `test_number`, `sms_twilio_number_ids`, `sequence`, `country_id`, `number` | `Send test SMS`, `Reload Numbers from Twilio`, `action_unlink`, `Update Account`, `Discard` |  | `sms_twilio` |

Machine-readable definition: `../../../schemas/data/entities/sms.twilio.account.manage.json`; views: `../../../schemas/interfaces/views/sms.twilio.account.manage.json`.
