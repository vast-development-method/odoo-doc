# Website Menu (`website.menu`)

**Transport name:** `website.menu`  
**Storage name:** `website_menu`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website`  
**Extended by packages:** `website_sale`, `website_event`, `website_event_track`

Description: Website Menu

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (16)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Menu | single line text |  | required; translatable |
| `url` | Url | single line text |  | required; computed by rule `_compute_url` and stored; default `#` |
| `page_id` | Related Page | many to one | `website.page` | indexed (btree_not_null); on delete of the target: cascade |
| `controller_page_id` | Related Model Page | many to one | `website.controller.page` | indexed (btree_not_null); on delete of the target: cascade |
| `new_window` | New Window | boolean |  |  |
| `sequence` | Sequence | integer |  | default computed dynamically (_default_sequence) |
| `website_id` | Website | many to one | `website` | on delete of the target: cascade |
| `parent_id` | Parent Menu | many to one | `website.menu` | indexed; on delete of the target: cascade |
| `child_id` | Child Menus | one to many | `website.menu` | inverse field `parent_id` |
| `parent_path` | Parent Path | single line text |  | indexed |
| `is_visible` | Is Visible | boolean |  | computed by rule `_compute_visible` (not stored) |
| `group_ids` | Visible Groups | many to many | `res.groups` | visible only to groups `base.group_user`; Help: User needs to be at least in one of these groups to see the menu |
| `is_mega_menu` | Is Mega Menu | boolean |  | computed by rule `fields.Boolean(compute=_compute_field_is_mega_menu, inverse=_set_field_is_mega_menu)` (not stored); writable through an inverse rule |
| `mega_menu_content` | Mega Menu Content | rich text |  | translatable |
| `mega_menu_classes` | Mega Menu Classes | single line text |  |  |
| `theme_template_id` | Theme Template | many to one | `theme.website.menu` | indexed (btree_not_null); not copied on duplication |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sequence` | preparation rule | self | `website` |  |  |
| `_compute_field_is_mega_menu` | computation | self | `website` | depends: `mega_menu_content` |  |
| `_set_field_is_mega_menu` | internal rule | self | `website` |  |  |
| `_compute_display_name` | computation | self | `website` | depends: `website_id`; depends_context: `display_website` |  |
| `_compute_url` | computation | self | `website` | depends: `page_id`, `is_mega_menu`, `child_id` |  |
| `_validate_parent_menu` | validation | self | `website` | constrains: `parent_id`, `child_id`, `is_mega_menu`, `mega_menu_content` | Ensure valid menu hierarchy and mega menu constraints.  Rules enforced: - Menus must not exceed two levels of nesting. - A mega menu must not have a parent or child. - Menus with children cannot be added as a submenu under another menu. |
| `create` | lifecycle override | self, vals_list | `website` | model_create_multi | In case a menu without a website_id is trying to be created, we duplicate it for every website. Note: Particularly useful when installing a module that adds a menu like       /shop. So every website has the shop menu.       Be careful to return correct record for ir.model.data xml_id in case       of default main menus creation. |
| `write` | lifecycle override | self, vals | `website` |  |  |
| `unlink` | lifecycle override | self | `website_event_track`, `website_event`, `website` |  | Override to synchronize event configuration fields with menu deletion. |
| `_unlink_except_master_tags` | internal rule | self | `website` | ondelete |  |
| `_compute_visible` | computation | self | `website_sale`, `website` |  | Hide '/shop' menus to the public user if only logged-in users can access it. |
| `_clean_url` | internal rule | self | `website` |  |  |
| `_is_active` | internal rule | self | `website` |  | To be considered active, a menu should either:  - have its URL matching the request's URL and have no children - or have a children menu URL matching the request's URL  Matching an URL means, either:  - be equal, eg `/contact/on-site` vs `/contact/on-site` - be equal after unslug, eg `/shop/1` and `/shop/my-super-product-1`  Note that saving a menu URL with an anchor or a query string is considered a corner case, and the following applies:  - anchor/fragment are ignored during the comparison (it would be   impossible to compare anyway as the client is not sending the anchor   to the se |
| `get_tree` | operation | self, website_id, menu_id | `website` | model |  |
| `save` | operation | self, website_id, data | `website_event`, `website` | model | Method context:  This method takes a data argument that follows the following format:  [    { 'id': 4, url: '/mypage' },    { 'id': 'menu_xxx_...', url: '/anotherpage' }  ]   The new menu entries are identified by their ID being a string and not an integer value.  Note that when going through super() call, those id entries are replaced by their created  menu ID (integer), so we need to identify new menu entries before calling super.   Override purpose:   All sub-menus of an event are children of a 'main' website.menu, linking to the event main page.   We abuse that information to determine if  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_validate_parent_menu` | UserError | A mega menu cannot have a parent or child menu. | `website` |
| `_validate_parent_menu` | UserError | Menus with child menus cannot be added as a submenu. | `website` |
| `_validate_parent_menu` | UserError | Menus cannot have more than two levels of hierarchy. | `website` |
| `_unlink_except_master_tags` | UserError | You cannot delete this website menu as this serves as the default parent menu for new websites (e.g., /shop, /event, ...). | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website` |
| `base.group_portal` | no | yes | no | no | `website` |
| `base.group_user` | no | yes | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Website menu: group_ids | global (all users) | `['\|', ('group_ids', '=', False), ('group_ids', 'in', user.all_group_ids.ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website.website_menus_form_view` | form |  | `name`, `url`, `page_id`, `controller_page_id`, `is_mega_menu`, `new_window`, `sequence`, `website_id`, `parent_id`, `group_ids`, `child_id`, `sequence`, `name`, `url` |  |  | `website` |
| `website.menu_tree` | list |  | `sequence`, `website_id`, `name`, `url`, `is_mega_menu`, `new_window`, `parent_id`, `group_ids` |  |  | `website` |
| `website.menu_search` | search |  | `name`, `url`, `website_id` |  | `Name`, `Url`, `Website` | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website.action_website_menu` | Website Menu | list,form |  | `{'search_default_group_by_website_id':1}` | current | `website` |

Machine-readable definition: `../../../schemas/data/entities/website.menu.json`; views: `../../../schemas/interfaces/views/website.menu.json`.
