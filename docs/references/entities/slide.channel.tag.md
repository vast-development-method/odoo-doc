# Channel/Course Tag (`slide.channel.tag`)

**Transport name:** `slide.channel.tag`  
**Storage name:** `slide_channel_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Channel/Course Tag

## Identity and behavior

- Default ordering: `group_sequence asc, sequence asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | required; default `10`; indexed |
| `group_id` | Group | many to one | `slide.channel.tag.group` | required; indexed; on delete of the target: cascade |
| `group_sequence` | Group sequence | integer |  | read only; related through path `group_id.sequence` and stored; indexed |
| `channel_ids` | Channels | many to many | `slide.channel` | association table `slide_channel_tag_rel` |
| `color` | Color Index | integer |  | default computed dynamically (lambda self: randint(1, 11)); Help: Tag color used in both backend and website. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Channel Tag: public/portal: color = published | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_view_search` | search |  | `name`, `group_id` |  |  | `website_slides` |
| `website_slides.slide_channel_tag_view_form` | form |  | `name`, `group_id` |  |  | `website_slides` |
| `website_slides.slide_channel_tag_view_tree` | list |  | `sequence`, `group_sequence`, `name`, `group_id` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_action` | Course Tags | list,form |  |  |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.channel.tag.json`; views: `../../../schemas/interfaces/views/slide.channel.tag.json`.
