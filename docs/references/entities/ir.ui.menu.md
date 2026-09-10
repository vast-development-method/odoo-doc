# Menu (`ir.ui.menu`)

**Transport name:** `ir.ui.menu`  
**Storage name:** `ir_ui_menu`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `mail`, `hr`, `hr_holidays_attendance`, `hr_recruitment`, `website`, `project`, `hr_timesheet`, `hr_timesheet_attendance`

Description: Menu

## Identity and behavior

- Default ordering: `sequence,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Menu | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `10` |
| `child_id` | Child identifiers | one to many | `ir.ui.menu` | inverse field `parent_id` |
| `parent_id` | Parent Menu | many to one | `ir.ui.menu` | indexed; on delete of the target: restrict |
| `parent_path` | Parent Path | single line text |  | indexed |
| `group_ids` | Groups | many to many | `res.groups` | association table `ir_ui_menu_group_rel`; Help: If you have groups, the visibility of this menu will be based on these groups. If this field is empty, Odoo will compute visibility based on the related object's read access. |
| `complete_name` | Full Path | single line text |  | computed by rule `_compute_complete_name` (not stored); recursive dependency |
| `web_icon` | Web Icon File | single line text |  |  |
| `action` | Action | reference |  |  |
| `web_icon_data` | Web Icon Image | binary |  |  |

## Selection values

### `action` (Action)

| Value | Label |
|---|---|
| `ir.actions.report` | ir.actions.report |
| `ir.actions.act_window` | ir.actions.act_window |
| `ir.actions.act_url` | ir.actions.act_url |
| `ir.actions.server` | ir.actions.server |
| `ir.actions.client` | ir.actions.client |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_complete_name` | computation | self | `base` | depends: `name`, `parent_id.complete_name` |  |
| `_get_full_name` | preparation rule | self, level | `base` |  | Return the full name of ``self`` (up to a certain level). |
| `_read_image` | internal rule | self, path | `base` |  |  |
| `_check_parent_id` | validation | self | `base` | constrains: `parent_id` |  |
| `_visible_menu_ids` | internal rule | self, debug | `base` | model | Return the ids of the menu items visible to the user. |
| `_filter_visible_menus` | internal rule | self | `base` |  | Filter `self` to only keep the menu items that should be visible in the menu hierarchy of the current user. Uses a cache for speeding up the computation. |
| `_compute_display_name` | computation | self | `base` | depends: `parent_id` |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_compute_web_icon_data` | computation | self, web_icon | `base` |  | Returns the image associated to ``web_icon``.  :param str web_icon: a comma-separated value string for either:    * an image icon: ``f"{module},{path}"``   * a built icon: ``f"{icon_class},{icon_color},{background_color}"``  The ``web_icon_data`` computed field uses :meth:`_read_image` for image web icons, and is ``False`` for built icons. |
| `unlink` | lifecycle override | self | `base` |  |  |
| `copy` | lifecycle override | self, default | `base` |  |  |
| `get_user_roots` | operation | self | `base` | model | Return all root menu ids visible for the user.  :return: the root menu ids :rtype: list(int) |
| `_load_menus_blacklist` | internal rule | self | `base`, `hr_holidays_attendance`, `hr_recruitment`, `hr_timesheet_attendance`, `hr_timesheet`, `hr`, `project` |  |  |
| `load_menus_root` | operation | self | `base`, `website` | model |  |
| `load_menus` | operation | self, debug | `base` | model |  |
| `_get_menuitems_xmlids` | preparation rule | self | `base` |  |  |
| `load_web_menus` | operation | self, debug | `web` |  | Loads all menu items (all applications and their sub-menus) and processes them to be used by the webclient. Mainly, it associates with each application (top level menu) the action of its first child menu that is associated with an action (recursively), i.e. with the action to execute when the opening the app.  :return: the menus (including the images in Base64) |
| `_get_best_backend_root_menu_id_for_model` | preparation rule | self, res_model | `mail` | model | Get the best menu root id for the given res_model and the access rights of the user.  When a link to a model was sent to a user it was targeting a page without menu, so it was hard for the user to act on it. The goal of this method is to find the best suited menu to display on a page of a given model.  Technically, the method tries to find a menu root which has a sub menu visible to the user that has an action linked to the given model. If there is more than one possibility, it chooses the preferred one based on the following preference function that determine the sub-menu from which the root  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_id` | ValidationError | Error! You cannot create recursive menus. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.edit_menu_access` | form |  | `name`, `parent_id`, `sequence`, `group_ids`, `complete_name`, `action`, `web_icon`, `web_icon_data`, `child_id`, `sequence`, `name` |  |  | `base` |
| `base.edit_menu` | list |  | `sequence`, `complete_name` |  |  | `base` |
| `base.edit_menu_access_search` | search |  | `name`, `parent_id` |  | `Archived` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.grant_menu_access` | Menu Items |  |  | `{'ir.ui.menu.full_list':True}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.ui.menu.json`; views: `../../../schemas/interfaces/views/ir.ui.menu.json`.
