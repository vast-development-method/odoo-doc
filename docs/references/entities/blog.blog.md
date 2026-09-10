# Blog (`blog.blog`)

**Transport name:** `blog.blog`  
**Storage name:** `blog_blog`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_blog`

Description: Blog

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `website.seo.metadata`, `website.multi.mixin`, `website.cover_properties.mixin`, `website.searchable.mixin`
- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence) |
| `name` | Blog Name | single line text |  | required; translatable |
| `subtitle` | Blog Subtitle | single line text |  | translatable |
| `active` | Active | boolean |  | default `True` |
| `content` | Content | rich text |  | translatable |
| `blog_post_ids` | Blog Posts | one to many | `blog.post` | inverse field `blog_id` |
| `blog_post_count` | Posts | integer |  | computed by rule `_compute_blog_post_count` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `website_blog` |  |  |
| `_compute_blog_post_count` | computation | self | `website_blog` | depends: `blog_post_ids` |  |
| `write` | lifecycle override | self, vals | `website_blog` |  |  |
| `message_post` | messaging hook | self, parent_id, subtype_id, **kwargs | `website_blog` |  | Temporary workaround to avoid spam. If someone replies on a channel through the 'Presentation Published' email, it should be considered as a note as we don't want all channel followers to be notified of this answer. |
| `all_tags` | operation | self, join, min_limit | `website_blog` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_blog` | model |  |
| `_search_render_results` | search rule | self, fetch_fields, mapping, icon, limit | `website_blog` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_blog` |
| `base.group_portal` | no | yes | no | no | `website_blog` |
| `base.group_user` | no | yes | no | no | `website_blog` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Blog: active only | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('active', '=', True)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_blog.view_blog_blog_list` | list |  | `sequence`, `name`, `blog_post_count`, `website_id`, `active` |  |  | `website_blog` |
| `website_blog.view_blog_blog_form` | form |  | `active`, `name`, `subtitle`, `website_id` |  |  | `website_blog` |
| `website_blog.blog_blog_view_search` | search |  | `name` |  | `Archived` | `website_blog` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_blog.action_blog_blog` | Blogs | list,form |  |  |  | `website_blog` |

Machine-readable definition: `../../../schemas/data/entities/blog.blog.json`; views: `../../../schemas/interfaces/views/blog.blog.json`.
