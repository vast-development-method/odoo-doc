# User list of blocked 3rd-party domains (`website.custom_blocked_third_party_domains`)

**Transport name:** `website.custom_blocked_third_party_domains`  
**Storage name:** `website_custom_blocked_third_party_domains`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website`

Description: User list of blocked 3rd-party domains

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `content` | Content | multi line text |  | default computed dynamically (lambda s: s.env['website'].get_current_website().custom_blocked_third_party_domains) |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_save` | user action | self | `website` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_save` | ValidationError | _('The following domain is not valid:') + '\n' + domain | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | yes | yes | yes | no | `website` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.view_edit_third_party_domains` | form |  | `content` | `Save`, `Cancel` |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.custom_blocked_third_party_domains.json`; views: `../../../schemas/interfaces/views/website.custom_blocked_third_party_domains.json`.
