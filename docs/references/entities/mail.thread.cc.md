# Email CC management (`mail.thread.cc`)

**Transport name:** `mail.thread.cc`  
**Storage name:** `mail_thread_cc`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Email CC management

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email_cc` | Email cc | single line text |  |  |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_mail_cc_sanitized_raw_dict` | messaging hook | self, cc_string | `mail` |  | return a dict of sanitize_email:raw_email from a string of cc |
| `message_new` | messaging hook | self, msg_dict, custom_values | `mail` | model |  |
| `message_update` | messaging hook | self, msg_dict, update_vals | `mail` |  |  |
| `_message_add_suggested_recipients` | messaging hook | self, force_primary_email | `mail` |  |  |

Machine-readable definition: `../../../schemas/data/entities/mail.thread.cc.json`.
