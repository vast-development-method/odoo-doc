# Slide Tag (`slide.tag`)

**Transport name:** `slide.tag`  
**Storage name:** `slide_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Slide Tag

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_slide_tag_unique` | Constraint | `UNIQUE(name)` | A tag must be unique! | `website_slides` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.view_slide_tag_form` | form |  | `name` |  |  | `website_slides` |
| `website_slides.view_slide_tag_tree` | list |  | `name` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.action_slide_tag` | Content Tags | list,form |  |  |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.tag.json`; views: `../../../schemas/interfaces/views/slide.tag.json`.
