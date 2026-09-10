# Forum Tag (`forum.tag`)

**Transport name:** `forum.tag`  
**Storage name:** `forum_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_forum`

Description: Forum Tag

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `website.searchable.mixin`, `website.seo.metadata`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `color` | Color | integer |  |  |
| `forum_id` | Forum | many to one | `forum.forum` | required; indexed |
| `post_ids` | Posts | many to many | `forum.post` | restricted by domain `[["state", "=", "active"]]`; association table `forum_tag_rel` |
| `posts_count` | Number of Posts | integer |  | computed by rule `_compute_posts_count` and stored |
| `website_url` | Link to questions with the tag | single line text |  | computed by rule `_compute_website_url` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name, forum_id)` | Tag name already exists! | `website_forum` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_posts_count` | computation | self | `website_forum` | depends: `post_ids`, `post_ids.tag_ids`, `post_ids.state`, `post_ids.active` |  |
| `_compute_website_url` | computation | self | `website_forum` | depends: `forum_id`, `forum_id.name`, `name` |  |
| `create` | lifecycle override | self, vals_list | `website_forum` | model_create_multi |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_forum` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | AccessError | %d karma required to create a new Tag. | `website_forum` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | yes | yes | no | no | `website_forum` |
| `base.group_portal` | yes | yes | no | no | `website_forum` |
| `base.group_user` | yes | yes | yes | yes | `website_forum` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website forum tag: Public user can only access to tag linked to public forum | `[(4, ref('base.group_public'))]` | `[('forum_id.privacy', '=', 'public')]` | True | True | True | True |
| Website forum tag: User can only access to tag linked to public (or authorized) forum | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `['\|', ('forum_id.privacy', 'in', ['public', 'connected']), '&', ('forum_id.privacy', '=', 'private'), ('forum_id.authorized_group_id', 'in', user.all_group_ids.ids)]` | True | True | True | True |
| Website forum tag : Manager user can access to all tags | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Website slides forum tag: Public User can only access to tag linked to forum related to public courses | `[(4, ref('base.group_public'))]` | `[('forum_id.slide_channel_ids.website_published', '=', True), ('forum_id.slide_channel_ids.visibility', '=', 'public')]` | True | True | True | True |
| Website forum: Signed In users can access tags linked to public or connected users-visibility courses | `[(4, ref('base.group_portal')), (4, ref('base.group_user'))]` | `[             '&',                 ('forum_id.slide_channel_ids.website_published', '=', True),                 '\|',                     ('forum_id.slide_channel_ids.visibility', 'in', ('public','connected')),                     ('forum_id.slide_channel_ids.is_member', '=', True)             ]` | True | True | True | True |
| Website slides forum tag: website slides officer can access all tag | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_tag_view_list` | list |  | `name`, `color`, `forum_id` |  |  | `website_forum` |
| `website_forum.forum_tag_view_form` | form |  | `name`, `color`, `forum_id` |  |  | `website_forum` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_forum.forum_tag_action` | Forum Tags | list,form |  |  |  | `website_forum` |

Machine-readable definition: `../../../schemas/data/entities/forum.tag.json`; views: `../../../schemas/interfaces/views/forum.tag.json`.
