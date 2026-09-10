# Website Technical Page (`website.technical.page`)

**Transport name:** `website.technical.page`  
**Storage name:** `website_technical_page`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Website Technical Page

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Page Name | single line text |  |  |
| `website_url` | Website Page uniform resource locator | single line text |  |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `open_website_url` | operation | self | `website` |  | Opens the technical page for the given URL and website. |
| `get_static_routes` | operation | self | `website` |  | Returns a set of website content static routes. |
| `_table_query` | internal rule | self | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | no | yes | no | no | `website` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_technical_pages_list_view` | list |  | `name`, `website_url` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_technical_pages` | Technical Pages | list |  |  |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.technical.page.json`; views: `../../../schemas/interfaces/views/website.technical.page.json`.
