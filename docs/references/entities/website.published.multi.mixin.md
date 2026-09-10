# Multi Website Published Mixin (`website.published.multi.mixin`)

**Transport name:** `website.published.multi.mixin`  
**Storage name:** `website_published_multi_mixin`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Multi Website Published Mixin

## Identity and behavior

- Mixins (classical inheritance): `website.published.mixin`, `website.multi.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `website_published` | Website Published | boolean |  | computed by rule `_compute_website_published` (not stored); writable through an inverse rule; searchable through a search rule |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_website_published` | computation | self | `website` | depends: `is_published`, `website_id`; depends_context: `website_id` |  |
| `_inverse_website_published` | inverse computation | self | `website` |  |  |
| `_search_website_published` | search rule | self, operator, value | `website` |  |  |
| `open_website_url` | operation | self | `website` |  |  |

Machine-readable definition: `../../../schemas/data/entities/website.published.multi.mixin.json`.
