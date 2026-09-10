# Website Theme Page (`theme.website.page`)

**Transport name:** `theme.website.page`  
**Storage name:** `theme_website_page`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Website Theme Page

## Identity and behavior

- Mixins (classical inheritance): `website.page_options.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `url` | Uniform resource locator | single line text |  |  |
| `view_id` | View | many to one | `theme.ir.ui.view` | required; indexed; on delete of the target: cascade |
| `website_indexed` | Page Indexed | boolean |  | default `True` |
| `is_published` | Is Published | boolean |  |  |
| `is_new_page_template` | New Page Template | boolean |  |  |
| `copy_ids` | Page using a copy of me | one to many | `website.page` | read only; not copied on duplication; inverse field `theme_template_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_base_model` | internal rule | self, website, **kwargs | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/theme.website.page.json`.
