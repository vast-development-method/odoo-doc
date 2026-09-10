# Test text message Mailing (`mailing.sms.test`)

**Transport name:** `mailing.sms.test`  
**Storage name:** `mailing_sms_test`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mass_mailing_sms`

Description: Test SMS Mailing

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `numbers` | Number(s) | multi line text |  | required; default computed dynamically (_default_numbers); Help: Carriage-return-separated list of phone numbers |
| `mailing_id` | Mailing | many to one | `mailing.mailing` | required; on delete of the target: cascade |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_numbers` | preparation rule | self | `mass_mailing_sms` |  |  |
| `_prepare_test_trace_values` | preparation rule | self, record, sms_number, sms_uuid, body | `mass_mailing_sms` |  |  |
| `action_send_sms` | user action | self | `mass_mailing_sms` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mass_mailing.group_mass_mailing_user` | yes | yes | yes | no | `mass_mailing_sms` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mass_mailing_sms.mailing_sms_test_view_form` | form |  | `numbers`, `mailing_id` | `Send Test`, `Discard` |  | `mass_mailing_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mass_mailing_sms.mailing_sms_test_action` | Test Mailing | form |  |  | new | `mass_mailing_sms` |

Machine-readable definition: `../../../schemas/data/entities/mailing.sms.test.json`; views: `../../../schemas/interfaces/views/mailing.sms.test.json`.
