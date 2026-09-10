# Blog Post (`blog.post`)

**Transport name:** `blog.post`  
**Storage name:** `blog_post`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_blog`

Description: Blog Post

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `website.seo.metadata`, `website.published.multi.mixin`, `website.page_visibility_options.mixin`, `website.cover_properties.mixin`, `website.searchable.mixin`
- Default ordering: `id DESC`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Title | single line text |  | required; default ; translatable |
| `subtitle` | Sub Title | single line text |  | translatable |
| `author_id` | Author | many to one | `res.partner` | default computed dynamically (lambda self: self.env.user.partner_id); indexed (btree_not_null) |
| `author_avatar` | Avatar | binary |  | related through path `author_id.image_128` |
| `author_name` | Author Name | single line text |  | related through path `author_id.display_name` and stored |
| `active` | Active | boolean |  | default `True` |
| `blog_id` | Blog | many to one | `blog.blog` | required; default computed dynamically (lambda self: self.env['blog.blog'].search([], limit=1)); indexed; on delete of the target: cascade |
| `tag_ids` | Tags | many to many | `blog.tag` |  |
| `content` | Content | rich text |  | default computed dynamically (_default_content); translatable |
| `teaser` | Teaser | multi line text |  | computed by rule `_compute_teaser` (not stored); writable through an inverse rule; translatable |
| `teaser_manual` | Teaser Content | multi line text |  | translatable |
| `website_message_ids` | Website Message | one to many |  | restricted by domain `lambda self: [('model', '=', self._name), ('message_type', '=', 'comment'), '&', ('is_internal', '=', False), ('subtype_id.internal', '=', False)]` |
| `create_date` | Created on | date and time |  | read only |
| `published_date` | Published Date | date and time |  |  |
| `post_date` | Publishing date | date and time |  | computed by rule `_compute_post_date` and stored; writable through an inverse rule; Help: The blog post will be visible for your visitors as of this date on the website if it is set as published. |
| `create_uid` | Created by | many to one | `res.users` | read only |
| `write_date` | Last Updated on | date and time |  | read only |
| `write_uid` | Last Contributor | many to one | `res.users` | read only |
| `visits` | No of Views | integer |  | read only; default ; not copied on duplication |
| `website_id` | Website | many to one |  | read only; related through path `blog_id.website_id` and stored |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_website_url` | computation | self | `website_blog` |  |  |
| `_default_content` | preparation rule | self | `website_blog` |  |  |
| `_compute_teaser` | computation | self | `website_blog` | depends: `content`, `teaser_manual` |  |
| `_set_teaser` | internal rule | self | `website_blog` |  |  |
| `_compute_post_date` | computation | self | `website_blog` | depends: `create_date`, `published_date` |  |
| `_set_post_date` | internal rule | self | `website_blog` |  |  |
| `_check_for_publication` | validation | self, vals | `website_blog` |  |  |
| `create` | lifecycle override | self, vals_list | `website_blog` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_blog` |  |  |
| `copy_data` | lifecycle override | self, default | `website_blog` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `website_blog` |  | Instead of the classic form view, redirect to the post on website directly if user is an employee or if the post is published. |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `website_blog` |  |  |
| `_notify_thread_by_inbox` | internal rule | self, message, recipients_data, msg_vals, **kwargs | `website_blog` |  |  |
| `_default_website_meta` | preparation rule | self | `website_blog` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_blog` | model |  |

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
| Blog Post: public: published only | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_blog.view_blog_post_form` | form |  | `is_published`, `blog_id`, `active`, `name`, `subtitle`, `tag_ids`, `website_id`, `author_id`, `create_date`, `visits`, `post_date`, `write_uid`, `write_date`, `website_meta_title`, `website_meta_description`, `website_meta_keywords` |  |  | `website_blog` |
| `website_blog.blog_post_view_kanban` | kanban |  | `name`, `website_id`, `blog_id`, `post_date`, `author_id`, `is_published` |  |  | `website_blog` |
| `website_blog.view_blog_post_search` | search |  | `name`, `write_uid`, `blog_id` |  | `Archived`, `Blog`, `Author`, `Last Contributor` | `website_blog` |
| `website_blog.view_blog_post_list` | list |  | `active`, `name`, `website_url`, `author_id`, `blog_id`, `create_uid`, `write_uid`, `write_date`, `is_seo_optimized`, `is_published`, `website_id` |  |  | `website_blog` |
| `website_blog.blog_post_view_form_add` | form |  | `website_url`, `blog_id`, `name` |  |  | `website_blog` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_blog.action_blog_post` | Blog Post Pages | list,kanban,form |  | `{'create_action': 'website_blog.blog_post_action_add'}` |  | `website_blog` |
| `website_blog.blog_post_action_add` | New Blog Post | form |  |  | new | `website_blog` |

Machine-readable definition: `../../../schemas/data/entities/blog.post.json`; views: `../../../schemas/interfaces/views/blog.post.json`.
