# Asset (`ir.asset`)

**Transport name:** `ir.asset`  
**Storage name:** `ir_asset`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `website`, `website`

Description: Asset

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `bundle` | Bundle name | single line text |  | required |
| `directive` | Directive | selection |  | default computed dynamically (APPEND_DIRECTIVE) |
| `path` | Path (or glob pattern) | single line text |  | required |
| `target` | Target | single line text |  |  |
| `active` | active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | required; default computed dynamically (DEFAULT_SEQUENCE) |
| `key` | Key | single line text |  | not copied on duplication |
| `website_id` | Website | many to one | `website` | on delete of the target: cascade |
| `theme_template_id` | Theme Template | many to one | `theme.ir.asset` | indexed (btree_not_null); not copied on duplication |

## Operations (17)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base`, `website` |  | COW for ir.asset. This way editing websites does not impact other websites. Also this way newly created websites will only contain the default assets. |
| `unlink` | lifecycle override | self | `base` |  |  |
| `_get_asset_params` | preparation rule | self | `base`, `website` |  | This method can be overriden to add param _get_asset_paths call. Those params will be part of the orm cache key |
| `_get_asset_bundle_url` | preparation rule | self, filename, unique, assets_params, ignore_params | `base`, `website` |  |  |
| `_parse_bundle_name` | internal rule | self, bundle_name, debug_assets | `base` |  |  |
| `_get_asset_paths` | preparation rule | self, bundle, assets_params | `base` |  | Fetches all asset file paths from a given list of addons matching a certain bundle. The returned list is composed of tuples containing the file path [1], the first addon calling it [0] and the bundle name. Asset loading is performed as follows:  1. All 'ir.asset' records matching the given bundle and with a sequence strictly less than 16 are applied.  3. The manifests of the given addons are checked for assets declaration for the given bundle. If any, they are read sequentially and their operations are applied to the current list.  4. After all manifests have been parsed, the remaining 'ir.ass |
| `_fill_asset_paths` | internal rule | self, bundle, asset_paths, seen, addons, installed, **assets_params | `base` |  | Fills the given AssetPaths instance by applying the operations found in the matching bundle of the given addons manifests. See `_get_asset_paths` for more information.  :param bundle: name of the bundle from which to fetch the file paths :param addons: list of addon names as strings :param asset_paths: the AssetPath object to fill :param seen: a list of bundles already checked to avoid circularity :param assets_params: Keyword arguments:      * css: bool: whether or not to include style files     * js: bool: whether or not to include script files     * xml: bool: whether or not to include temp |
| `_process_path` | background operation | self, bundle, directive, target, path_def, asset_paths, seen, addons, installed, bundle_start_index, **assets_params | `base` |  | This sub function is meant to take a directive and a set of arguments and apply them to the current asset_paths list accordingly.  It is nested inside `_get_asset_paths` since we need the current list of addons, extensions and asset_paths.  :param directive: string :param target: string or None or False :param path_def: string |
| `_get_related_assets` | preparation rule | self, domain, **kwargs | `base`, `website` |  | Returns a set of assets matching the domain, regardless of their active state. This method can be overridden to filter the results. :param domain: search domain :returns: ir.asset recordset |
| `_get_related_bundle` | preparation rule | self, target_path_def, root_bundle | `base` |  | Returns the first bundle directly defining a glob matching the target path. This is useful when generating an 'ir.asset' record to override a specific asset and target the right bundle, i.e. the first one defining the target path.  :param str target_path_def: path to match. :param str root_bundle: bundle from which to initiate the search. :returns: the first matching bundle or None |
| `_get_active_addons_list` | preparation rule | self, **kwargs | `base`, `website` |  | Can be overridden to filter the returned list of active modules. |
| `_topological_sort` | internal rule | self, addons_tuple | `base` | model | Returns a list of sorted modules name accord to the spec in ir.module.module that is, application desc, sequence, name then topologically sorted |
| `_get_installed_addons_list` | preparation rule | self | `base` | model | Returns the list of all installed addons. :returns: string[]: list of module names |
| `_get_paths` | preparation rule | self, path_def, installed | `base` |  | Returns a list of tuple (path, full_path, modified) matching a given glob (path_def). The glob can only occur in the static direcory of an installed addon.  If the path_def matches a (list of) file, the result will contain the full_path and the modified time. Ex: ('/base/static/file.js', '<file path>', 643636800)  If the path_def looks like a non aggregable path (http://, /web/assets), only return the path Ex: ('http://example.com/lib.js', None, -1) The timestamp -1 is given to be thruthy while carrying no information.  If the path_def is not a wildwa |
| `_process_command` | background operation | self, command | `base` |  | Parses a given command to return its directive, target and path definition. |
| `filter_duplicate` | operation | self, website_id | `website` |  | Filter current recordset only keeping the most suitable asset per distinct name. Every non-accessible asset will be removed from the set:    * In non website context, every asset with a website will be removed   * In a website context, every asset from another website |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.asset_view_form` | form |  | `name`, `bundle`, `directive`, `sequence`, `active`, `target`, `path` |  |  | `base` |
| `base.asset_view_tree` | list |  | `name`, `bundle`, `sequence`, `active` |  |  | `base` |
| `base.asset_view_search` | search |  | `name`, `bundle`, `directive`, `sequence`, `path` |  | `Active` | `base` |
| `website.asset_view_form_inherit_website` | field | `base.asset_view_form` | `directive`, `website_id` |  |  | `website` |
| `website.asset_view_tree_inherit_website` | field | `base.asset_view_tree` | `bundle`, `website_id` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_asset` | Assets |  |  | `{'search_default_active': 1}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.asset.json`; views: `../../../schemas/interfaces/views/ir.asset.json`.
