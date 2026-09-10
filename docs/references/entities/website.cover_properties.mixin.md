# Cover Properties Website Mixin (`website.cover_properties.mixin`)

**Transport name:** `website.cover_properties.mixin`  
**Storage name:** `website_cover_properties_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: Cover Properties Website Mixin

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `cover_properties` | Cover Properties | multi line text |  | default computed dynamically (lambda s: json_safe.dumps(s._default_cover_properties())) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_cover_properties` | preparation rule | self | `website` |  |  |
| `_get_background` | preparation rule | self, height, width | `website` |  |  |
| `write` | lifecycle override | self, vals | `website` |  |  |

Machine-readable definition: `../../../schemas/data/entities/website.cover_properties.mixin.json`.
