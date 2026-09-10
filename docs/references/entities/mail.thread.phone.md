# Phone Blacklist Mixin (`mail.thread.phone`)

**Transport name:** `mail.thread.phone`  
**Storage name:** `mail_thread_phone`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `phone_validation`

Description: Phone Blacklist Mixin

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `phone_sanitized` | Sanitized Number | single line text |  | computed by rule `_compute_phone_sanitized` and stored; Help: Field used to store sanitized phone number. Helps speeding up searches and comparisons. |
| `phone_sanitized_blacklisted` | Phone Blacklisted | boolean |  | computed by rule `_compute_blacklisted` (not stored); searchable through a search rule; visible only to groups `base.group_user`; Help: If the sanitized phone number is on the blacklist, the contact won't receive mass mailing sms anymore, from any list |
| `phone_blacklisted` | Blacklisted Phone is Phone | boolean |  | computed by rule `_compute_blacklisted` (not stored); visible only to groups `base.group_user`; Help: Indicates if a blacklisted sanitized phone number is a phone number. Helps distinguish which number is blacklisted             when there is both a mobile and phone field in a model. |
| `phone_mobile_search` | Phone Number | single line text |  | searchable through a search rule |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_phone_sanitized_search_index_exists` | internal rule | self | `phone_validation` | model |  |
| `_phone_get_phone_mobile_search_fields` | internal rule | self | `phone_validation` | model | Return stored phone fields to include in phone_mobile_search lookups.  phone_sanitized (E164-normalized) is added alongside the raw _phone_get_number_fields so that searching by a normalized number (e.g. "+3212345678") also matches records whose raw numbers are stored in a different format (e.g. "012345678", "003212345678"). |
| `init` | lifecycle override | self | `phone_validation` |  |  |
| `_search_phone_mobile_search` | search rule | self, operator, value | `phone_validation` |  |  |
| `_compute_phone_sanitized` | computation | self | `phone_validation` | depends: |  |
| `_compute_blacklisted` | computation | self | `phone_validation` | depends: `phone_sanitized` |  |
| `_search_phone_sanitized_blacklisted` | search rule | self, operator, value | `phone_validation` | model |  |
| `_assert_phone_field` | internal rule | self | `phone_validation` |  |  |
| `_phone_get_sanitize_triggers` | internal rule | self | `phone_validation` |  | Tool method to get all triggers for sanitize |
| `_phone_set_blacklisted` | internal rule | self | `phone_validation` |  |  |
| `_phone_reset_blacklisted` | internal rule | self | `phone_validation` |  |  |
| `phone_action_blacklist_remove` | operation | self | `phone_validation` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search_phone_mobile_search` | UserError | Missing definition of phone fields. | `phone_validation` |
| `_search_phone_mobile_search` | UserError | Please enter at least 3 characters when searching a Phone number. | `phone_validation` |
| `_assert_phone_field` | UserError | Invalid primary phone field on model %s | `phone_validation` |
| `_assert_phone_field` | UserError | Invalid primary phone field on model %s | `phone_validation` |
| `phone_action_blacklist_remove` | AccessError | You do not have the access right to unblacklist phone numbers. Please contact your administrator. | `phone_validation` |

Machine-readable definition: `../../../schemas/data/entities/mail.thread.phone.json`.
