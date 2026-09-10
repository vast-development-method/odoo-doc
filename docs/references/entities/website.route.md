# All Website Route (`website.route`)

**Transport name:** `website.route`  
**Storage name:** `website_route`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: All Website Route

## Identity and behavior

- Default ordering: `path`
- Display name field: `path`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `path` | Route | single line text |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_search_display_name` | search rule | self, operator, value | `website` | model |  |
| `name_search` | operation | self, name, domain, operator, limit | `website` | model; readonly |  |
| `_refresh` | internal rule | self | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_website_designer` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.route.json`.
