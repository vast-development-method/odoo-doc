# Blog Tag (`blog.tag`)

**Transport name:** `blog.tag`  
**Storage name:** `blog_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_blog`

Description: Blog Tag

## Identity and behavior

- Mixins (classical inheritance): `website.seo.metadata`
- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `category_id` | Category | many to one | `blog.tag.category` | indexed |
| `color` | Color | integer |  |  |
| `post_ids` | Posts | many to many | `blog.post` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `website_blog` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_blog` |
| `base.group_portal` | no | yes | no | no | `website_blog` |
| `base.group_user` | no | yes | no | no | `website_blog` |
| `website.group_website_designer` | yes | yes | yes | yes | `website_blog` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_blog.blog_tag_tree` | list |  | `name`, `category_id`, `color`, `post_ids` |  |  | `website_blog` |
| `website_blog.blog_tag_form` | form |  | `name`, `category_id`, `color`, `post_ids` |  |  | `website_blog` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_blog.action_tags` | Blog Tags | list,form |  |  |  | `website_blog` |

Machine-readable definition: `../../../schemas/data/entities/blog.tag.json`; views: `../../../schemas/interfaces/views/blog.tag.json`.
