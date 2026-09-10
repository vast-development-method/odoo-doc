# Robots.txt Editor (`website.robots`)

**Transport name:** `website.robots`  
**Storage name:** `website_robots`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website`

Description: Robots.txt Editor

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `content` | Content | multi line text |  | default computed dynamically (lambda s: s.env['website'].get_current_website().robots_txt) |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_save` | user action | self | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `website.group_website_designer` | yes | yes | yes | no | `website` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.view_edit_robots` | form |  | `content` | `Save`, `Cancel` |  | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.robots.json`; views: `../../../schemas/interfaces/views/website.robots.json`.
