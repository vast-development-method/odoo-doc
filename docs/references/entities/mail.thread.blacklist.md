# Mail Blacklist mixin (`mail.thread.blacklist`)

**Transport name:** `mail.thread.blacklist`  
**Storage name:** `mail_thread_blacklist`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Mail Blacklist mixin

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `email_normalized` | Normalized Email | single line text |  | computed by rule `_compute_email_normalized` and stored; Help: This field is used to search on email address as the primary email field can contain more than strictly an email address. |
| `is_blacklisted` | Blacklist | boolean |  | computed by rule `_compute_is_blacklisted` (not stored); searchable through a search rule; visible only to groups `base.group_user`; Help: If the email address is on the blacklist, the contact won't receive mass mailing anymore, from any list |
| `message_bounce` | Bounce | integer |  | default ; Help: Counter of the number of bounced emails for this contact |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_email_normalized` | computation | self | `mail` | depends: |  |
| `_search_is_blacklisted` | search rule | self, operator, value | `mail` | model |  |
| `_compute_is_blacklisted` | computation | self | `mail` | depends: `email_normalized` |  |
| `_assert_primary_email` | internal rule | self | `mail` |  |  |
| `_message_receive_bounce` | messaging hook | self, email, partner | `mail` |  | Override of mail.thread generic method. Purpose is to increment the bounce counter of the record. |
| `_message_reset_bounce` | messaging hook | self, email | `mail` |  | Override of mail.thread generic method. Purpose is to reset the bounce counter of the record. |
| `mail_action_blacklist_remove` | operation | self | `mail` |  |  |
| `_detect_loop_sender_domain` | internal rule | self, email_from_normalized | `mail` | model | Return the domain to be used to detect duplicated records created by alias.  :param email_from_normalized: FROM of the incoming email, normalized |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_assert_primary_email` | UserError | Invalid primary email field on model %s | `mail` |
| `_assert_primary_email` | UserError | Invalid primary email field on model %s | `mail` |
| `mail_action_blacklist_remove` | AccessError | You do not have the access right to unblacklist emails. Please contact your administrator. | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.thread.blacklist.json`.
