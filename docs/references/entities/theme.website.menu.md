# Website Theme Menu (`theme.website.menu`)

**Transport name:** `theme.website.menu`  
**Storage name:** `theme_website_menu`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`

Description: Website Theme Menu

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `url` | Uniform resource locator | single line text |  | default  |
| `page_id` | Page | many to one | `theme.website.page` | indexed (btree_not_null); on delete of the target: cascade |
| `new_window` | New Window | boolean |  |  |
| `sequence` | Sequence | integer |  |  |
| `parent_id` | Parent | many to one | `theme.website.menu` | indexed; on delete of the target: cascade |
| `mega_menu_content` | Mega Menu Content | rich text |  |  |
| `mega_menu_classes` | Mega Menu Classes | single line text |  |  |
| `use_main_menu_as_parent` | Use Main Menu As Parent | boolean |  | default `True` |
| `copy_ids` | Menu using a copy of me | one to many | `website.menu` | read only; not copied on duplication; inverse field `theme_template_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_base_model` | internal rule | self, website, **kwargs | `website` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `website` |

Machine-readable definition: `../../../schemas/data/entities/theme.website.menu.json`.
