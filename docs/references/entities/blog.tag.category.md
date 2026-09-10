# Blog Tag Category (`blog.tag.category`)

**Transport name:** `blog.tag.category`  
**Storage name:** `blog_tag_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_blog`

Description: Blog Tag Category

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `tag_ids` | Tags | one to many | `blog.tag` | inverse field `category_id` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag category already exists! | `website_blog` |

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
| `website_blog.blog_tag_category_form` | form |  | `name` |  |  | `website_blog` |
| `website_blog.blog_tag_category_tree` | list |  | `name` |  |  | `website_blog` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_blog.action_tag_category` | Tag Category | list,form |  |  |  | `website_blog` |

Machine-readable definition: `../../../schemas/data/entities/blog.tag.category.json`; views: `../../../schemas/interfaces/views/blog.tag.category.json`.
