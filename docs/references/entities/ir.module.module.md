# Module (`ir.module.module`)

**Transport name:** `ir.module.module`  
**Storage name:** `ir_module_module`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `account`, `base_import_module`, `base_install_request`, `delivery`, `website`, `point_of_sale`, `website_sale`

Description: Module

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `application desc,sequence,name`
- Display name field: `shortdesc`
- Display name search fields: `["name", "shortdesc", "summary"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (36)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Technical Name | single line text |  | required; read only |
| `category_id` | Category | many to one | `ir.module.category` | read only; indexed |
| `shortdesc` | Module Name | single line text |  | read only; translatable |
| `summary` | Summary | single line text |  | read only; translatable |
| `description` | Description | multi line text |  | read only; translatable |
| `description_html` | Description hypertext markup language | rich text |  | computed by rule `_get_desc` (not stored) |
| `author` | Author | single line text |  | read only |
| `maintainer` | Maintainer | single line text |  | read only |
| `contributors` | Contributors | multi line text |  | read only |
| `website` | Website | single line text |  | read only |
| `installed_version` | Latest Version | single line text |  | computed by rule `_get_latest_version` (not stored) |
| `latest_version` | Installed Version | single line text |  | read only |
| `published_version` | Published Version | single line text |  | read only |
| `url` | uniform resource locator | single line text |  | read only |
| `sequence` | Sequence | integer |  | default `100` |
| `dependencies_id` | Dependencies | one to many | `ir.module.module.dependency` | read only; inverse field `module_id` |
| `country_ids` | Country | many to many | `res.country` | association table `module_country` |
| `exclusion_ids` | Exclusions | one to many | `ir.module.module.exclusion` | read only; inverse field `module_id` |
| `auto_install` | Automatic Installation | boolean |  | Help: An auto-installable module is automatically installed by the system when all its dependencies are satisfied. If the module has no dependency, it is always installed. |
| `state` | Status | selection |  | read only; default `uninstallable`; indexed |
| `demo` | Demo Data | boolean |  | read only; default  |
| `license` | License | selection |  | read only; default `LGPL-3` |
| `menus_by_module` | Menus | multi line text |  | computed by rule `_get_views` and stored |
| `reports_by_module` | Reports | multi line text |  | computed by rule `_get_views` and stored |
| `views_by_module` | Views | multi line text |  | computed by rule `_get_views` and stored |
| `application` | Application | boolean |  | read only |
| `icon` | Icon uniform resource locator | single line text |  |  |
| `icon_image` | Icon | binary |  | computed by rule `_get_icon_image` (not stored) |
| `icon_flag` | Flag | single line text |  | computed by rule `_get_icon_image` (not stored) |
| `to_buy` | the enterprise edition Module | boolean |  | default  |
| `has_iap` | Has In-app purchase | boolean |  | computed by rule `_compute_has_iap` (not stored) |
| `account_templates` | Account Templates | binary |  | computed by rule `_compute_account_templates` (not stored) |
| `imported` | Imported Module | boolean |  |  |
| `module_type` | Module Type | selection |  | default `official` |
| `image_ids` | Screenshots | one to many | `ir.attachment` | read only; restricted by domain `[["res_model", "=", "ir.module.module"], ["mimetype", "=like", "image/%"]]`; inverse field `res_id` |
| `is_installed_on_current_website` | Is Installed On Current Website | boolean |  | computed by rule `_compute_is_installed_on_current_website` (not stored) |

## Selection values

### `license` (License)

| Value | Label |
|---|---|
| `GPL-2` | GPL Version 2 |
| `GPL-2 or any later version` | GPL-2 or later version |
| `GPL-3` | GPL Version 3 |
| `GPL-3 or any later version` | GPL-3 or later version |
| `AGPL-3` | Affero GPL-3 |
| `LGPL-3` | LGPL Version 3 |
| `Other OSI approved licence` | Other OSI Approved License |
| `OEEL-1` | the enterprise edition Edition License v1.0 |
| `OPL-1` | Proprietary License, version 1.0 |
| `Other proprietary` | Other Proprietary |

### `module_type` (Module Type)

| Value | Label |
|---|---|
| `official` | Official Apps |
| `industries` | Industries |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `UNIQUE (name)` | The name of the module must be unique! | `base` |

## Operations (83)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_module_info` | operation | cls, name | `base` |  |  |
| `_get_desc` | computation | self | `base` | depends: `name`, `description` |  |
| `_get_latest_version` | computation | self | `base_import_module`, `base` | depends: `name` |  |
| `_get_views` | computation | self | `base` | depends: `name`, `state` |  |
| `_get_icon_image` | computation | self | `base_import_module`, `base` | depends: `icon` |  |
| `_compute_has_iap` | computation | self | `base` |  |  |
| `_unlink_except_installed` | internal rule | self | `base` | ondelete |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `_get_modules_to_load_domain` | preparation rule | self | `base_import_module`, `base` |  | Domain to retrieve the modules that should be loaded by the registry. |
| `check_external_dependencies` | operation | self, module_name, newstate | `base` |  |  |
| `_state_update` | internal rule | self, newstate, states_to_update, level | `base` |  |  |
| `button_install` | user action | self | `base` |  |  |
| `button_immediate_install` | user action | self | `base` |  | Installs the selected module(s) immediately and fully, returns the next res.config action to execute  :returns: next res.config item to execute :rtype: dict[str, object] |
| `button_reset_state` | user action | self | `base` | model |  |
| `check_module_update` | operation | self | `base` | model |  |
| `module_uninstall` | operation | self | `account`, `base_import_module`, `base` |  | Perform the various steps required to uninstall a module completely including the deletion of all database structures created by the module: tables, columns, constraints, etc. |
| `_remove_copied_views` | internal rule | self | `base` |  | Remove the copies of the views installed by the modules in `self`.  Those copies do not have an external id so they will not be cleaned by `_module_data_uninstall`. This is why we rely on `key` instead.  It is important to remove these copies because using them will crash if they rely on data that don't exist anymore if the module is removed. |
| `downstream_dependencies` | operation | self, known_deps, exclude_states | `base` |  | Return the modules that directly or indirectly depend on the modules in `self`, and that satisfy the `exclude_states` filter. |
| `upstream_dependencies` | operation | self, known_deps, exclude_states | `base` |  | Return the dependency tree of modules of the modules in `self`, and that satisfy the `exclude_states` filter. |
| `next` | operation | self | `base` |  | Return the action linked to an ir.actions.todo is there exists one that should be executed. Otherwise, redirect to /web |
| `_button_immediate_function` | internal rule | self, function | `base`, `website` |  |  |
| `button_immediate_uninstall` | user action | self | `base` |  | Uninstall the selected module(s) immediately and fully, returns the next res.config action to execute |
| `button_uninstall` | user action | self | `base` |  |  |
| `button_uninstall_wizard` | user action | self | `base` |  | Launch the wizard to uninstall the given module. |
| `button_immediate_upgrade` | user action | self | `base` |  | Upgrade the selected module(s) immediately and fully, return the next res.config action to execute |
| `button_upgrade` | user action | self | `base_import_module`, `base` |  |  |
| `get_values_from_terp` | operation | terp | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `update_list` | operation | self | `base`, `website` | model |  |
| `_update_from_terp` | internal rule | self, terp | `base` |  |  |
| `_update_dependencies` | internal rule | self, depends, auto_install_requirements | `base` |  |  |
| `_update_countries` | internal rule | self, countries | `base` |  |  |
| `_update_exclusions` | internal rule | self, excludes | `base` |  |  |
| `_update_category` | internal rule | self, category | `base` |  |  |
| `_update_translations` | internal rule | self, filter_lang, overwrite | `base` |  |  |
| `_check` | validation | self | `base`, `website` |  |  |
| `_get` | internal rule | self, name | `base` |  | Return the (sudoed) `ir.module.module` record with the given name. The result may be an empty recordset if the module is not found. |
| `_get_id` | preparation rule | self, name | `base` |  |  |
| `_installed` | internal rule | self | `base` | model | Return the set of installed modules as a dictionary {name: id} |
| `search_panel_select_range` | operation | self, field_name, **kwargs | `base_import_module`, `base` | model |  |
| `_load_module_terms` | internal rule | self, modules, langs, overwrite | `account`, `base_import_module`, `base`, `website_sale`, `website` | model | Load PO files of the given modules for the given languages. |
| `_extract_resource_attachment_translations` | internal rule | self, module, lang | `base_import_module`, `base` | model |  |
| `_compute_account_templates` | computation | self | `account` | depends: `state` |  |
| `write` | lifecycle override | self, vals | `account`, `website` |  | Override to correctly upgrade themes after upgrade/installation of modules.  # Install      If this theme wasn't installed before, then load it for every website     for which it is in the stream.      eg. The very first installation of a theme on a website will trigger this.      eg. If a website uses theme_A and we install sale, then theme_A_sale will be         autoinstalled, and in this case we need to load theme_A_sale for the website.  # Upgrade      There are 2 cases to handle when upgrading a theme:      * When clicking on the theme upgrade button on the interface,         in which cas |
| `_register_hook` | internal rule | self | `account` |  |  |
| `_get_imported_module_names` | preparation rule | self | `base_import_module` | model |  |
| `_import_module` | internal rule | self, module, path, force, with_demo | `base_import_module` |  |  |
| `_import_zipfile` | internal rule | self, module_file, force, with_demo | `base_import_module` | model |  |
| `web_search_read` | lifecycle override | self, domain, specification, offset, limit, order, count_limit | `base_import_module` | model |  |
| `more_info` | operation | self | `base_import_module` |  |  |
| `web_read` | lifecycle override | self, specification | `base_import_module` |  |  |
| `_get_modules_from_apps` | preparation rule | self, fields, module_type, module_name, domain, limit, offset | `base_import_module` | model |  |
| `_call_apps` | internal rule | self, payload | `base_import_module` | model |  |
| `_get_industry_categories_from_apps` | preparation rule | self | `base_import_module` | model |  |
| `button_immediate_install_app` | user action | self | `base_import_module` |  |  |
| `_get_missing_dependencies` | preparation rule | self, zip_data | `base_import_module` | model |  |
| `_get_missing_dependencies_modules` | preparation rule | self, zip_data | `base_import_module` |  |  |
| `_get_imported_module_translations_for_webclient` | preparation rule | self, module, lang | `base_import_module` | model |  |
| `action_open_install_request` | user action | self | `base_install_request` |  |  |
| `action_view_delivery_methods` | user action | self | `delivery` |  |  |
| `_compute_is_installed_on_current_website` | computation | self | `website` |  | Compute for every theme in `self` if the current website is using it or not.  This method does not take dependencies into account, because if it did, it would show the current website as having multiple different themes installed at the same time, which would be confusing for the user. |
| `_get_module_data` | preparation rule | self, model_name | `website` |  | Return every theme template model of type `model_name` for every theme in `self`.  :param model_name: string with the technical name of the model for which to get data.     (the name must be one of the keys present in `_theme_model_names`) :return: recordset of theme template models (of type defined by `model_name`) |
| `_update_records` | internal rule | self, model_name, website | `website` |  | This method:  - Find and update existing records.      For each model, overwrite the fields that are defined in the template (except few     cases such as active) but keep inherited models to not lose customizations.  - Create new records from templates for those that didn't exist.  - Remove the models that existed before but are not in the template anymore.      See _theme_cleanup for more information.   There is a special 'while' loop around the 'for' to be able queue back models at the end of the iteration when they have unmet dependencies. Hopefully the dependency will be found after all m |
| `_post_copy` | internal rule | self, old_rec, new_rec | `website` |  |  |
| `_theme_load` | internal rule | self, website | `website` |  | For every type of model in `self._theme_model_names`, and for every theme in `self`: create/update real models for the website `website` based on the theme template models.  :param website: `website` model on which to load the themes |
| `_theme_unload` | internal rule | self, website | `website` |  | For every type of model in `self._theme_model_names`, and for every theme in `self`: remove real models that were generated based on the theme template models for the website `website`.  :param website: `website` model on which to unload the themes |
| `_theme_cleanup` | internal rule | self, model_name, website | `website` |  | Remove orphan models of type `model_name` from the current theme and for the website `website`.  We need to compute it this way because if the upgrade (or deletion) of a theme module removes a model template, then in the model itself the variable `theme_template_id` will be set to NULL and the reference to the theme being removed will be lost. However we do want the ophan to be deleted from the website when we upgrade or delete the theme from the website.  `website.page` and `website.menu` don't have `key` field so we don't clean them. |
| `_theme_get_upstream` | internal rule | self | `website` |  | Return installed upstream themes.  :return: recordset of themes `ir.module.module` |
| `_theme_get_downstream` | internal rule | self | `website` |  | Return installed downstream themes that starts with the same name.  eg. For theme_A, this will return theme_A_sale, but not theme_B even if theme B     depends on theme_A.  :return: recordset of themes `ir.module.module` |
| `_theme_get_stream_themes` | internal rule | self | `website` |  | Returns all the themes in the stream of the current theme.  First find all its downstream themes, and all of the upstream themes of both sorted by their level in hierarchy, up first.  :return: recordset of themes `ir.module.module` |
| `_theme_get_stream_website_ids` | internal rule | self | `website` |  | Websites for which this theme (self) is in the stream (up or down) of their theme.  :return: recordset of websites `website` |
| `_theme_upgrade_upstream` | internal rule | self | `website` |  | Upgrade the upstream dependencies of a theme, and install it if necessary. |
| `_theme_remove` | internal rule | self, website | `website` | model | Remove from `website` its current theme, including all the themes in the stream.  The order of removal will be reverse of installation to handle dependencies correctly.  :param website: `website` model for which the themes have to be removed |
| `button_choose_theme` | user action | self | `website` |  | Remove any existing theme on the current website and install the theme `self` instead.  The actual loading of the theme on the current website will be done automatically on `write` thanks to the upgrade and/or install.  When installating a new theme, upgrade the upstream chain first to make sure we have the latest version of the dependencies to prevent inconsistencies.  :return: dict with the next action to execute |
| `button_remove_theme` | user action | self | `website` |  | Remove the current theme of the current website. |
| `button_refresh_theme` | user action | self | `website` |  | Refresh the current theme of the current website.  To refresh it, we only need to upgrade the modules. Indeed the (re)loading of the theme will be done automatically on `write`. |
| `update_theme_images` | operation | self | `website` | model |  |
| `get_themes_domain` | operation | self | `website` |  | Returns the 'ir.module.module' search domain matching all available themes. |
| `_create_model_data` | internal rule | self, views | `website` | model | Creates model data records for newly created view records.  :param views: views for which model data must be created |
| `_generate_primary_snippet_templates` | internal rule | self | `website` |  | Generates snippet templates hierarchy based on manifest entries for use in the configurator and when creating new pages from templates. |
| `_generate_primary_page_templates` | internal rule | self | `website` |  | Generates page templates based on manifest entries. |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |

## Validation and error messages (29)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_installed` | UserError | You are trying to remove a module that is installed or will be installed. | `base` |
| `check_external_dependencies` | UserError | msg | `base` |
| `_state_update` | UserError | Recursion error in modules dependencies! | `base` |
| `_state_update` | UserError | You try to install module "%(module)s" that depends on module "%(dependency)s". But the latter module is not available in your system. | `base` |
| `button_install` | UserError | You are trying to install incompatible modules in category "%(category)s":%(module_list)s | `base` |
| `button_install` | UserError | Modules "%(module)s" and "%(incompatible_module)s" are incompatible. | `base` |
| `_button_immediate_function` | UserError | The method _button_immediate_install cannot be called on init or non loaded registries. Please use button_install instead. | `base` |
| `_button_immediate_function` | UserError | The system is currently processing another module operation. Please try again later or contact your system administrator. | `base` |
| `_button_immediate_function` | UserError | The system is currently processing another module operation. Please try again later or contact your system administrator. | `base` |
| `_button_immediate_function` | UserError | The system is currently processing a scheduled action. Module operations are not possible at this time, please try again later or contact your system administrator. | `base` |
| `button_uninstall` | UserError | Those modules cannot be uninstalled: %s | `base` |
| `button_uninstall` | UserError | One or more of the selected modules have already been uninstalled, if you believe this to be an error, you may try again later or contact support. | `base` |
| `button_upgrade` | UserError | Cannot upgrade module “%s”. It is not installed. | `base` |
| `button_upgrade` | UserError | You try to upgrade the module %(module)s that depends on the module: %(dependency)s. But this module is not available in your system. | `base` |
| `_import_module` | UserError | err | `base_import_module` |
| `_import_module` | UserError | Studio customizations require the system Studio app. | `base_import_module` |
| `_import_module` | UserError | The assets path in the manifest of imported module '%(module_name)s' cannot contain glob wildcards (e.g., *, **). | `base_import_module` |
| `_import_zipfile` | AccessError | Only administrators can install data modules. | `base_import_module` |
| `_import_zipfile` | UserError | Only zip files are supported. | `base_import_module` |
| `_import_zipfile` | UserError | File '%s' exceed maximum allowed file size | `base_import_module` |
| `_import_zipfile` | UserError | No manifest found in '%(modules)s'. Can't import the zip file. | `base_import_module` |
| `_import_zipfile` | UserError | Error while importing module '%(module)s'.   %(error_message)s | `base_import_module` |
| `_get_modules_from_apps` | UserError | The list of industry applications cannot be fetched. Please try again later | `base_import_module` |
| `_get_modules_from_apps` | UserError | Connection to %s failed The list of industry modules cannot be fetched | `base_import_module` |
| `button_immediate_install_app` | UserError | missing_dependencies_description | `base_import_module` |
| `button_immediate_install_app` | UserError | The module %s cannot be downloaded | `base_import_module` |
| `button_immediate_install_app` | UserError | Connection to %(url)s failed, the module %(module)s cannot be downloaded. | `base_import_module` |
| `_get_missing_dependencies_modules` | UserError | File '%s' exceed maximum allowed file size | `base_import_module` |
| `_update_records` | MissingError | error | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | yes | no | no | `base_install_request` |

## Views (15)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_module_filter_inherit_account` | xpath | `base.view_module_filter` |  |  |  | `account` |
| `base.view_module_filter` | search |  | `name`, `category_id`, `category_id` |  | `Apps`, `Extra`, `Installed`, `Not Installed`, `Author`, `Category`, `Status` | `base` |
| `base.module_form` | form |  | `icon_image`, `shortdesc`, `summary`, `author`, `website`, `category_id`, `name`, `license`, `installed_version`, `demo`, `application`, `state`, `dependencies_id`, `name`, `state`, `exclusion_ids`, `name`, `state`, `menus_by_module`, `views_by_module`, `reports_by_module`, `description_html` | `Activate`, `Upgrade`, `Uninstall`, , ,  |  | `base` |
| `base.module_tree` | list |  | `shortdesc`, `name`, `author`, `website`, `installed_version`, `state` | `Activate`, `Upgrade` |  | `base` |
| `base.module_view_kanban` | kanban |  | `to_buy`, `name`, `state`, `summary`, `website`, `application`, `icon`, `icon_flag`, `shortdesc`, `summary`, `name` | `button_immediate_install`, , , ,  |  | `base` |
| `base_import_module.module_view_kanban_apps_inherit` | xpath | `base.module_view_kanban` | `module_type` |  |  | `base_import_module` |
| `base_import_module.module_tree_apps_inherit` | field | `base.module_tree` | `installed_version`, `module_type`, `name` |  |  | `base_import_module` |
| `base_import_module.module_form_apps_inherit` | xpath | `base.module_form` |  |  |  | `base_import_module` |
| `base_import_module.view_module_filter_apps_inherit` | xpath | `base.view_module_filter` | `module_type` |  |  | `base_import_module` |
| `base_install_request.ir_module_module_view_kanban` | button | `base.module_view_kanban` |  | `button_immediate_install`, `action_open_install_request` |  | `base_install_request` |
| `delivery.delivery_provider_module_list` | field | `base.module_tree` | `name` |  |  | `delivery` |
| `delivery.delivery_provider_module_kanban` | button | `base.module_view_kanban` |  | `button_immediate_install`, `Delivery Methods` |  | `delivery` |
| `website.theme_view_kanban` | kanban |  | `icon`, `summary`, `name`, `state`, `url`, `image_ids`, `category_id`, `display_name`, `is_installed_on_current_website` | `button_refresh_theme`, `button_remove_theme`, `button_choose_theme`,  |  | `website` |
| `website.theme_view_search` | search |  | `name`, `category_id` |  | `Author`, `Category` | `website` |
| `website.theme_view_form_preview` | form |  | `url` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `account.open_account_charts_modules` | Chart Templates | kanban,list,form |  | `{                 'search_default_category_id': ref('base.module_category_accounting_localizations_account_charts'),                 'searchpanel_default_category_id': ref('base.module_category_accounting_localizations_account_charts'),             }` |  | `account` |
| `base.open_module_tree` | Apps | kanban,list,form |  | `{'search_default_app':1}` |  | `base` |
| `website.action_website_add_features` | Apps | kanban,list,form | `['!', ('name', '=like', 'theme_%')]` | `{'search_default_category_id': ref('base.module_category_website_website'), 'searchpanel_default_category_id': ref('base.module_category_website')}` |  | `website` |
| `website.theme_install_kanban_action` | Pick a Theme | kanban,form | `obj().get_themes_domain()` |  | fullscreen | `website` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `base.ir_module_reference_print` | Technical guide | qweb-pdf | `base.report_irmodulereference` |  |  |

Machine-readable definition: `../../../schemas/data/entities/ir.module.module.json`; views: `../../../schemas/interfaces/views/ir.module.module.json`.
