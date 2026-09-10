# Multi Website Mixin (`website.multi.mixin`)

**Transport name:** `website.multi.mixin`  
**Storage name:** `website_multi_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `website`

Description: Multi Website Mixin

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `website_id` | Website | many to one | `website` | indexed; on delete of the target: restrict; Help: Restrict to a specific website. |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `can_access_from_current_website` | operation | self, website_id | `website` |  |  |

Machine-readable definition: `../../../schemas/data/entities/website.multi.mixin.json`.
