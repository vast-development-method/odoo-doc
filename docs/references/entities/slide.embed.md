# Embedded Slides View Counter (`slide.embed`)

**Transport name:** `slide.embed`  
**Storage name:** `slide_embed`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Embedded Slides View Counter

## Identity and behavior

- Display name field: `website_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `slide_id` | Presentation | many to one | `slide.slide` | required; indexed; on delete of the target: cascade |
| `url` | Third Party Website uniform resource locator | single line text |  |  |
| `website_name` | Website | single line text |  | computed by rule `_compute_website_name` (not stored) |
| `count_views` | # Views | integer |  | default `1` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_website_name` | computation | self | `website_slides` | depends: `url` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | yes | yes | yes | yes | `website_slides` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_embed_view_tree` | list |  | `website_name`, `count_views`, `slide_id` |  |  | `website_slides` |
| `website_slides.slide_embed_view_search` | search |  | `slide_id` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_embed_action` | Embed Views | list,search |  |  |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.embed.json`; views: `../../../schemas/interfaces/views/slide.embed.json`.
