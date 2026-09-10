# Portal Mixin (`portal.mixin`)

**Transport name:** `portal.mixin`  
**Storage name:** `portal_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `portal`

Description: Portal Mixin

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `access_url` | Portal Access uniform resource locator | single line text |  | computed by rule `_compute_access_url` (not stored); Help: Customer Portal URL |
| `access_token` | Security Token | single line text |  | searchable through a search rule; not copied on duplication |
| `access_warning` | Access warning | multi line text |  | computed by rule `_compute_access_warning` (not stored) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_search_access_token` | search rule | self, operator, value | `portal` |  |  |
| `_compute_access_warning` | computation | self | `portal` |  |  |
| `_compute_access_url` | computation | self | `portal` |  |  |
| `_portal_ensure_token` | internal rule | self | `portal` |  | Get the current record access token |
| `_get_share_url` | preparation rule | self, redirect, signup_partner, pid, share_token | `portal` |  | Build the url of the record  that will be sent by mail and adds additional parameters such as access_token to bypass the recipient's rights, signup_partner to allows the user to create easily an account, hash token to allow the user to be authenticated in the chatter of the record portal view, if applicable :param redirect : Send the redirect url instead of the direct portal share url :param signup_partner: allows the user to create an account with pre-filled fields. :param pid: = partner_id - when given, a hash is generated to allow the user to be authenticated     in the portal chatter, if a |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `portal` |  | Instead of the classic form view, redirect to the online document for portal users or if force_website=True. |
| `action_share` | user action | self | `portal` | model |  |
| `get_portal_url` | operation | self, suffix, report_type, download, query_string, anchor | `portal` |  | Get a portal url for this model, including access_token. The associated route must handle the flags for them to have any effect. - suffix: string to append to the url, before the query string - report_type: report_type query string, often one of: html, pdf, text - download: set the download query string to true - query_string: additional query string - anchor: string to append after the anchor # |

Machine-readable definition: `../../../schemas/data/entities/portal.mixin.json`.
