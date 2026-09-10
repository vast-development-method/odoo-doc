# Channel/Course Groups (`slide.channel.tag.group`)

**Transport name:** `slide.channel.tag.group`  
**Storage name:** `slide_channel_tag_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_slides`

Description: Channel/Course Groups

## Identity and behavior

- Mixins (classical inheritance): `website.published.mixin`
- Default ordering: `sequence asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Group Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | required; default `10`; indexed |
| `tag_ids` | Tags | one to many | `slide.channel.tag` | inverse field `group_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_is_published` | preparation rule | self | `website_slides` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_slides` |
| `base.group_portal` | no | yes | no | no | `website_slides` |
| `base.group_user` | no | yes | no | no | `website_slides` |
| `website_slides.group_website_slides_officer` | yes | yes | yes | yes | `website_slides` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_group_view_search` | search |  | `name` |  | `Has Menu Entry` | `website_slides` |
| `website_slides.slide_channel_tag_group_view_form` | form |  | `name`, `is_published`, `tag_ids`, `sequence`, `group_sequence`, `name`, `color` |  |  | `website_slides` |
| `website_slides.slide_channel_tag_group_view_tree` | list |  | `sequence`, `name`, `is_published`, `tag_ids` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_slides.slide_channel_tag_group_action` | Course Groups | list,form |  |  |  | `website_slides` |

Machine-readable definition: `../../../schemas/data/entities/slide.channel.tag.group.json`; views: `../../../schemas/interfaces/views/slide.channel.tag.group.json`.
