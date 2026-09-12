# View (`ir.ui.view`)

**Transport name:** `ir.ui.view`  
**Storage name:** `ir_ui_view`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `html_editor`, `mail`, `portal`, `base_import_module`, `web_hierarchy`, `website`, `website`

Description: View

## Identity and behavior

- Mixins (classical inheritance): `website.seo.metadata`
- Default ordering: `priority,name,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (31)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | View Name | single line text |  | required |
| `model` | Model | single line text |  | indexed |
| `key` | Key | single line text |  | indexed (btree_not_null) |
| `priority` | Sequence | integer |  | required; default `16` |
| `type` | View Type | selection |  | extended by packages `mail`, `web_hierarchy` |
| `arch` | View Architecture | multi line text |  | computed by rule `_compute_arch` (not stored); writable through an inverse rule; Help: This field should be used when accessing view arch. It will use translation.                                Note that it will read `arch_db` or `arch_fs` if in dev-xml mode. |
| `arch_base` | Base View Architecture | multi line text |  | computed by rule `_compute_arch_base` (not stored); writable through an inverse rule; Help: This field is the same as `arch` field without translations |
| `arch_db` | Arch Blob | multi line text |  | translatable; Help: This field stores the view arch. |
| `arch_fs` | Arch Filename | single line text |  | Help: File from where the view originates.                                                           Useful to (hard) reset broken views or to read arch from file in dev-xml mode. |
| `arch_updated` | Modified Architecture | boolean |  |  |
| `arch_prev` | Previous View Architecture | multi line text |  | Help: This field will save the current `arch_db` before writing on it.                                                                          Useful to (soft) reset a broken view. |
| `inherit_id` | Inherited View | many to one | `ir.ui.view` | indexed; on delete of the target: restrict |
| `inherit_children_ids` | Views which inherit from this one | one to many | `ir.ui.view` | inverse field `inherit_id` |
| `model_data_id` | Model Data | many to one | `ir.model.data` | computed by rule `_compute_model_data_id` (not stored); searchable through a search rule |
| `xml_id` | External identifier | single line text |  | computed by rule `_compute_xml_id` (not stored); Help: ID of the view defined in xml file |
| `group_ids` | Groups | many to many | `res.groups` | association table `ir_ui_view_group_rel`; Help: If this field is empty, the view applies to all users. Otherwise, the view applies to the users of those groups only. |
| `mode` | View inheritance mode | selection |  | required; default `primary`; Help: Only applies if this view inherits from an other one (inherit_id is not False/Null).  * if extension (default), if this view is requested the closest primary view is looked up (via inherit_id), then all views inheriting from it with this view's model are applied * if primary, the closest primary view is fully resolved (even if it uses a different model than this one), then this view's inheritance specs (<xpath/>) are applied, and the result is used as if it were this view's actual arch. |
| `warning_info` | Warning information | rich text |  | computed by rule `_compute_warning_info` (not stored) |
| `active` | Active | boolean |  | default `True`; Help: If this view is inherited, * if True, the view always extends its parent * if False, the view currently does not extend its parent but can be enabled |
| `model_id` | Model of the view | many to one | `ir.model` | computed by rule `_compute_model_id` (not stored); writable through an inverse rule |
| `invalid_locators` | Invalid Locators | structured document |  | computed by rule `_compute_invalid_locators` (not stored) |
| `customize_show` | Show As Optional Inherit | boolean |  | default  |
| `website_id` | Website | many to one | `website` | on delete of the target: cascade |
| `page_ids` | Page | one to many | `website.page` | inverse field `view_id` |
| `controller_page_ids` | Controller Page | one to many | `website.controller.page` | inverse field `view_id` |
| `first_page_id` | Website Page | many to one | `website.page` | computed by rule `_compute_first_page_id` (not stored); Help: First page linked to this view |
| `track` | Track | boolean |  | default ; Help: Allow to specify for one page of the website to be trackable or not |
| `visibility` | Visibility | selection |  | default  |
| `visibility_password` | Visibility Password | single line text |  | not copied on duplication; visible only to groups `base.group_system` |
| `visibility_password_display` | Visibility Password Display | single line text |  | computed by rule `_get_pwd` (not stored); writable through an inverse rule; visible only to groups `website.group_website_designer` |
| `theme_template_id` | Theme Template | many to one | `theme.ir.ui.view` | indexed (btree_not_null); not copied on duplication |

## Selection values

### `type` (View Type)

| Value | Label |
|---|---|
| `list` | List |
| `form` | Form |
| `graph` | Graph |
| `pivot` | Pivot |
| `calendar` | Calendar |
| `kanban` | Kanban |
| `search` | Search |
| `qweb` | QWeb |
| `activity` | Activity |
| `hierarchy` | Hierarchy |

### `mode` (View inheritance mode)

| Value | Label |
|---|---|
| `primary` | Base view |
| `extension` | Extension View |

### `visibility` (Visibility)

| Value | Label |
|---|---|
| `` | Public |
| `connected` | Signed In |
| `restricted_group` | Restricted Group |
| `password` | With Password |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_inheritance_mode` | Constraint | `CHECK (mode != 'extension' OR inherit_id IS NOT NULL)` | Invalid inheritance mode: if the mode is 'extension', the view must extend an other view | `base` |
| `_qweb_required_key` | Constraint | `CHECK (type != 'qweb' OR key IS NOT NULL)` | Invalid key: QWeb view should have a key | `base` |
| `_model_type_inherit_id` | Index | `(model, inherit_id)` |  | `base` |

## Operations (155)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_arch` | computation | self | `base` | depends: `arch_db`, `arch_fs`, `arch_updated`; depends_context: `read_arch_from_file`, `lang`, `edit_translations`, `check_translations` |  |
| `_inverse_arch` | inverse computation | self | `base` |  |  |
| `_compute_arch_base` | computation | self | `base` | depends: `arch`; depends_context: `read_arch_from_file` |  |
| `_inverse_arch_base` | inverse computation | self | `base` |  |  |
| `reset_arch` | operation | self, mode | `base` |  | Reset the view arch to its previous arch (soft) or its XML file arch if exists (hard). |
| `_compute_model_data_id` | computation | self | `base` | depends: `write_date` |  |
| `_search_model_data_id` | search rule | self, operator, value | `base` |  |  |
| `_compute_model_id` | computation | self | `base` | depends: `model` |  |
| `_inverse_compute_model_id` | inverse computation | self | `base` |  |  |
| `_compute_invalid_locators` | computation | self | `base` | depends: `arch`, `inherit_id` |  |
| `_compute_xml_id` | computation | self | `base` |  |  |
| `_valid_inheritance` | internal rule | self, arch | `base` |  | Check whether view inheritance is based on translated attribute. |
| `_check_xml` | validation | self | `base` |  |  |
| `_check_groups` | validation | self | `base` | constrains: `group_ids`, `inherit_id`, `mode` |  |
| `_check_000_inheritance` | validation | self | `base` | constrains: `inherit_id` |  |
| `_compute_defaults` | computation | self, values | `base` |  |  |
| `_compute_warning_info` | computation | self | `base` | depends: `arch` |  |
| `_validate_xml_encoding` | internal rule | self, text | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base`, `website` | model_create_multi | SOC for ir.ui.view creation. If a view is created without a website_id, it should get one if one is present in the context. Also check that an explicit website_id in create values matches the one in the context. |
| `write` | lifecycle override | self, vals | `base`, `website` |  | COW for ir.ui.view. This way editing websites does not impact other websites. Also this way newly created websites will only contain the default views. |
| `unlink` | lifecycle override | self | `base`, `website` |  | This implements COU (copy-on-unlink). When deleting a generic page website-specific pages will be created so only the current website is affected. |
| `_update_field_translations` | internal rule | self, field_name, translations, digest, source_lang | `base`, `website` |  |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `default_view` | operation | self, model, view_type | `base` | model | Fetches the default view for the provided (model, view_type) pair:  primary view with the lowest priority.  :param str model: :param int view_type: :return: id of the default view of False if none found :rtype: int |
| `_get_default_view_domain` | preparation rule | self, model, view_type | `base` | model |  |
| `_get_inheriting_views_domain` | preparation rule | self | `base`, `website` | model | Return a domain to filter the sub-views to inherit from. |
| `_get_filter_xmlid_query` | preparation rule | self | `base`, `website` | model | This method is meant to be overridden by other modules. |
| `_get_inheriting_views` | preparation rule | self | `base`, `website` | model | Determine the views that inherit from the current recordset, and return them as a recordset, ordered by priority then by id. |
| `_filter_loaded_views` | internal rule | self, check_view_ids | `base` |  | During the module upgrade phase it may happen that a view is present in the database but the fields it relies on are not fully loaded yet. This method only considers views that belong to modules whose code is already loaded. Custom views defined directly in the database are loaded only after the module initialization phase is completely finished. |
| `_check_view_access` | validation | self | `base` |  | Verify that a view is accessible by the current user based on the groups attribute. Views with no groups are considered private. |
| `_raise_view_error` | internal rule | self, message, node, from_exception, from_traceback | `base` |  | Handle a view error by raising an exception.  :param str message: message to raise or log, augmented with contextual                     view information :param node: the lxml element where the error is located (if any) :param BaseException from_exception:     when raising an exception, chain it to the provided one (default:     disable chaining) :param types.TracebackType from_traceback:     when raising an exception, start with this traceback (default: start     at exception creation) |
| `_log_view_warning` | internal rule | self, message, node | `base` |  | Handle a view issue by logging a warning.  :param str message: message to raise or log, augmented with contextual                     view information :param node: the lxml element where the error is located (if any) |
| `locate_node` | operation | self, arch, spec | `base` |  | Locate a node in a source (parent) architecture.  Given a complete source (parent) architecture (i.e. the field `arch` in a view), and a 'spec' node (a node in an inheriting view that specifies the location in the source view of what should be changed), return (if it exists) the node in the source view matching the specification.  :param arch: a parent architecture to modify :param spec: a modifying node in an inheriting view :return: a node in the source matching the spec |
| `inherit_branding` | operation | self, specs_tree | `base` |  |  |
| `_add_validation_flag` | internal rule | self, combined_arch, view, arch | `base` |  | Add a validation flag on elements in `combined_arch` or `arch`. This is part of the partial validation of views.  :param Element combined_arch: the architecture to be modified by `arch` :param view: an optional view inheriting `self` :param Element arch: an optional modifying architecture from inheriting     view `view` |
| `apply_inheritance_specs` | operation | self, source, specs_tree, pre_locate | `base` | model | Apply an inheriting view (a descendant of the base view)  Apply to a source architecture all the spec nodes (i.e. nodes describing where and what changes to apply to some parent architecture) given by an inheriting view.  :param Element source: a parent architecture to modify :param Element specs_tree: a modifying architecture in an inheriting view :param (optional) pre_locate: function that is execute before locating a node.                                 This function receives an arch as argument. :return: a modified source where the specs are applied :rtype: Element |
| `_combine` | internal rule | self, hierarchy | `base` |  | Return self's arch combined with its inherited views archs.  :param hierarchy: mapping from parent views to their child views :return: combined architecture :rtype: Element |
| `get_combined_arch` | operation | self | `base` |  | Return the arch of `self` (as a string) combined with its inherited views. |
| `_get_combined_arch` | preparation rule | self | `base` |  |  |
| `_get_combined_archs` | preparation rule | self | `base` |  | Return the arch of `self` (as an etree) combined with its inherited views. |
| `_get_view_refs` | preparation rule | self, node | `base` |  | Extract the `[view_type]_view_ref` keys and values from the node context attribute, giving the views to use for a field node.  :param node: the field node as an etree :return: a dictonary mapping the `[view_type]_view_ref` key to the xmlid of the view to use for that view type. |
| `_get_cached_template_prefetched_keys` | preparation rule | self | `base`, `website` | model |  |
| `_get_template_minimal_cache_keys` | preparation rule | self | `base`, `website` | model |  |
| `_get_cached_template_info` | preparation rule | self, id_or_xmlid, _view | `base` | model | Return the ir.ui.view id from the xml id, use `_preload_views`. |
| `_get_template_view` | preparation rule | self, id_or_xmlid, raise_if_not_found | `base` | model |  |
| `_get_template_domain` | preparation rule | self, xmlids | `base`, `website` | model | If a website_id is in the context and the given xml_id then try to get the id of the specific view for that website, but fallback to the id of the generic view if there is no specific. If no website_id is in the context, every view with a website will be filtered out.  Archived views are ignored (unless the active_test context is set, but then the ormcache will not work as expected). |
| `_get_template_order` | preparation rule | self | `base`, `website` | model |  |
| `_fetch_template_views` | internal rule | self, ids_or_xmlids | `base`, `website` | model | Return the view corresponding to `template`, which may be a view ID or an XML ID. Note that this method may be overridden for other kinds of template values. |
| `_clear_preload_views_cache_if_needed` | internal rule | self | `base` |  | Invalidate the local cache when the orm cache is cleared |
| `_preload_views` | internal rule | self, refs | `base` |  | Return self's arch combined with its inherited views archs.  :param refs: list of id or xmlid :return: dictionary of preloaded information {id or xmlid: {xmlid, ref, view, error}} |
| `postprocess_and_fields` | operation | self, node, model, **options | `base` |  | Return an architecture and a description of all the fields.  The field description combines the result of fields_get() and postprocess().  :param self: the view to postprocess :param node: the architecture as an etree :param model: the view's reference model name :return: a tuple (arch, fields) where arch is the given node as a     string and fields is the description of all the fields. |
| `_postprocess_access_rights` | internal rule | self, tree | `base` |  | Apply group restrictions: elements with a 'groups' attribute should be removed from the view to people who are not members.  Compute and set on node access rights based on view type. Specific views can add additional specific rights like creating columns for many2one-based grouping views. |
| `_postprocess_debug_to_cache` | internal rule | self, tree | `base` |  | Transform attribute groups="base.group_no_one" into a specific attribute "__debug__" to ease the special treatment of this case.  This feature is temporary because the behavior will be moved and processed in javascript soon. The management of 'base.group_no_one' is not consistent from the start. It is a magic group that historically was added or removed depending on the url used. 'base.group_no_one' should not to be considered as a security group but as a display feature.  Typically the templates do not match the intent when attribute 'groups' contains 'base.group_no_one' and other groups. In  |
| `_postprocess_debug` | internal rule | self, tree | `base` |  | Apply debug mode by making nodes invisible. |
| `_postprocess_view` | internal rule | self, node, model_name, editable, node_info, **options | `base` |  | Process the given architecture, modifying it in-place to add and remove stuff.  :param self: the optional view to postprocess :param node: the combined architecture as an etree :param model_name: the view's reference model name :param editable: whether the view is considered editable :return: the processed architecture's NameManager |
| `_add_missing_fields` | internal rule | self, node, name_manager | `base` |  | Add the fields required for evaluating expressions in the view given by `node`. |
| `_postprocess_on_change` | internal rule | self, arch, model | `base` |  | Add attribute on_change="1" on fields that are dependencies of computed fields on the same view. |
| `_get_x2many_missing_view_archs` | preparation rule | self, field, field_node, node_info | `base` |  | For x2many fields that require to have some multi-record arch (kanban or list) to display the records be available, this function fetches all arch that are needed and return them. The caller function is responsible to do what it needs with them. |
| `_postprocess_attributes` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_calendar` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_field` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_form` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_groupby` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_label` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_search` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_postprocess_tag_list` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_editable_node` | internal rule | self, node, name_manager | `base` |  | Return whether the given node must be considered editable. |
| `_editable_tag_form` | internal rule | self, node, name_manager | `base` |  |  |
| `_editable_tag_list` | internal rule | self, node, name_manager | `base` |  |  |
| `_editable_tag_field` | internal rule | self, node, name_manager | `base` |  |  |
| `_onchange_able_view` | internal rule | self, node | `base` |  |  |
| `_onchange_able_view_form` | internal rule | self, node | `base` |  |  |
| `_onchange_able_view_list` | internal rule | self, node | `base` |  |  |
| `_onchange_able_view_kanban` | internal rule | self, node | `base` |  |  |
| `_modifiers_from_model` | internal rule | self, node | `base` |  |  |
| `_validate_view` | internal rule | self, node, model_name, view_type, editable, node_info | `base` |  | Validate the given architecture node, and return its corresponding NameManager.  :param self: the view being validated :param node: the combined architecture as an etree :param model_name: the reference model name for the given architecture :param view_type: :param editable: whether the view is considered editable :param node_info: :return: the combined architecture's NameManager |
| `_validate_tag_form` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_list` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_graph` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_calendar` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_search` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_field` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_filter` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_button` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_groupby` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_searchpanel` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_label` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_page` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_img` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_a` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_ul` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_validate_tag_div` | internal rule | self, node, name_manager, node_info | `base` |  |  |
| `_check_dropdown_menu` | validation | self, node | `base` |  |  |
| `_check_progress_bar` | validation | self, node | `base` |  |  |
| `_is_qweb_based_view` | internal rule | self, view_type | `base`, `mail`, `web_hierarchy` |  |  |
| `_validate_attributes` | internal rule | self, node, name_manager, node_info | `base` |  | Generic validation of node attributes. |
| `_validate_classes` | internal rule | self, node, expr | `base` |  | Validate the classes present on node. |
| `_validate_fa_class_accessibility` | internal rule | self, node, description | `base` |  |  |
| `_validate_qweb_directive` | internal rule | self, node, directive, view_type | `base` |  | Some views (e.g. kanban, form) generate owl templates from the archs. However, we don't want to see owl directives directly written in archs. There are exceptions though, e.g. the kanban arch defines qweb templates. We thus here validate that the given directive is allowed, according to the view_type. |
| `_validate_expression` | internal rule | self, node, name_manager, py_expression, use, node_info | `base` |  |  |
| `_validate_domain_identifiers` | internal rule | self, node, name_manager, domain, use, target_model, node_info | `base` |  |  |
| `_check_field_paths` | validation | self, node, field_paths, model_name, use | `base` |  | Check whether the given field paths (dot-separated field names) correspond to actual sequences of fields on the given model. |
| `_read_template_keys` | internal rule | self | `base`, `website` |  | Return the list of context keys to use for caching `_read_template`. |
| `_get_view_etrees` | preparation rule | self | `base` |  |  |
| `_contains_branded` | internal rule | self, node | `base` |  |  |
| `_pop_view_branding` | internal rule | self, element | `base` |  |  |
| `distribute_branding` | operation | self, e, branding, parent_xpath, index_map | `base` |  |  |
| `is_node_branded` | operation | self, node | `base` |  | Finds out whether a node is branded or qweb-active (bears a @data-oe-model or a @t-* *which is not t-field* as t-field does not section out views)  :param node: an etree-compatible element to test :type node: etree._Element :rtype: boolean |
| `render_public_asset` | operation | self, template, values | `base`, `website` | readonly; model |  |
| `_render_template` | internal rule | self, template, values | `base`, `website` |  | Render the template. If website is enabled on request, then extend rendering context with website values. |
| `_validate_custom_views` | internal rule | self, model | `base_import_module`, `base` | model | Validate architecture of custom views (= without xml id) for a given model. This method is called at the end of registry update. |
| `_validate_module_views` | internal rule | self, module | `base` | model | Validate the architecture of all the views of a given module that are impacted by view updates, but have not been checked yet. |
| `_create_all_specific_views` | internal rule | self, processed_modules | `base`, `website` |  | To be overriden and have specific view behaviour on create |
| `_get_specific_views` | preparation rule | self | `base` |  | Given a view, return a record set containing all the specific views for that view's key. |
| `_load_records_write` | internal rule | self, values | `base` |  | During module update, when updating a generic view, we should also update its specific views (COW'd). Note that we will only update unmodified fields. That will mimic the noupdate behavior on views having an ir.model.data. |
| `_load_records_write_on_cow` | internal rule | self, cow_view, inherit_id, values | `base`, `website` |  |  |
| `get_view_info` | operation | self | `web` |  |  |
| `_get_view_info` | preparation rule | self | `mail`, `web_hierarchy`, `web` |  |  |
| `_get_cleaned_non_editing_attributes` | preparation rule | self, attributes | `html_editor` |  | Returns a new mapping of attributes -> value without the parts that are not meant to be saved (branding, editing classes, ...). Note that classes are meant to be cleaned on the client side before saving as mostly linked to the related options (so we are not supposed to know which to remove here).  :param attributes: a mapping of attributes -> value :return: a new mapping of attributes -> value |
| `extract_embedded_fields` | operation | self, arch | `html_editor` | model |  |
| `extract_oe_structures` | operation | self, arch | `html_editor` | model |  |
| `get_default_lang_code` | operation | self | `html_editor`, `website` | model |  |
| `save_embedded_field` | operation | self, el | `html_editor` | model |  |
| `save_oe_structure` | operation | self, el | `html_editor` |  |  |
| `_copy_custom_snippet_translations` | internal rule | self, record, html_field | `html_editor` | model | Given a `record` and its HTML `field`, detect any usage of a custom snippet and copy its translations. |
| `_copy_field_terms_translations` | internal rule | self, records_from, name_field_from, record_to, name_field_to | `html_editor` | model | Copy model terms translations from `records_from.name_field_from` to `record_to.name_field_to` for all activated languages if the term in `record_to.name_field_to` is untranslated (the term matches the one in the current language).  For instance, copy the translations of a `product.template.html_description` field to a `ir.ui.view.arch_db` field.  The method takes care of read and write access of both records/fields. |
| `_save_oe_structure_hook` | internal rule | self | `html_editor`, `website` | model |  |
| `_are_archs_equal` | internal rule | self, arch1, arch2 | `html_editor` | model |  |
| `_get_allowed_root_attrs` | preparation rule | self | `html_editor`, `website` | model |  |
| `replace_arch_section` | operation | self, section_xpath, replacement, replace_tail | `html_editor` |  |  |
| `to_field_ref` | operation | self, el | `html_editor` | model |  |
| `to_empty_oe_structure` | operation | self, el | `html_editor` | model |  |
| `_set_noupdate` | internal rule | self | `html_editor`, `website` | model | If website is installed, any call to `save` from the frontend will actually write on the specific view (or create it if not exist yet). In that case, we don't want to flag the generic view as noupdate. |
| `save` | operation | self, value, xpath | `html_editor`, `website` |  | Update a view section. The view section may embed fields to write  Note that `self` record might not exist when saving an embed field  :param str xpath: valid xpath to the tag to replace |
| `_view_get_inherited_children` | internal rule | self, view | `html_editor`, `website` | model |  |
| `_views_get` | internal rule | self, view_id, get_children, bundles, root, visited | `html_editor` | model | For a given view `view_id`, should return:     * the view itself (starting from its top most parent)     * all views inheriting from it, enabled or not       - but not the optional children of a non-enabled child     * all views called from it (via t-call)  :returns: recordset of ir.ui.view |
| `get_related_views` | operation | self, key, bundles | `html_editor`, `website` | model | Get inherit view's informations of the template `key`. returns templates info (which can be active or not) `bundles=True` returns also the asset bundles |
| `_get_snippet_addition_view_key` | preparation rule | self, template_key, key | `html_editor` | model |  |
| `_snippet_save_view_values_hook` | internal rule | self | `html_editor`, `website` | model |  |
| `_find_available_name` | internal rule | self, name, used_names | `html_editor` |  |  |
| `save_snippet` | operation | self, name, arch, template_key, snippet_key, thumbnail_url | `html_editor` | model | Saves a new snippet arch so that it appears with the given name when using the given snippets template.  :param name: the name of the snippet to save :param arch: the html structure of the snippet to save :param template_key: the key of the view regrouping all snippets in     which the snippet to save is meant to appear :param snippet_key: the key (without module part) to identify     the snippet from which the snippet to save originates :param thumbnail_url: the url of the thumbnail to use when displaying     the snippet to save |
| `rename_snippet` | operation | self, name, view_id, template_key | `html_editor` | model |  |
| `delete_snippet` | operation | self, view_id, template_key | `html_editor` | model |  |
| `_validate_tag_hierarchy` | internal rule | self, node, name_manager, node_info | `web_hierarchy` |  |  |
| `_get_pwd` | computation | self | `website` | depends: `visibility_password` |  |
| `_set_pwd` | internal rule | self | `website` |  |  |
| `_compute_first_page_id` | computation | self | `website` |  |  |
| `_compute_display_name` | computation | self | `website` | depends: `website_id`, `key`; depends_context: `display_key`, `display_website` |  |
| `_create_website_specific_pages_for_view` | internal rule | self, new_view, website | `website` |  |  |
| `get_view_hierarchy` | operation | self | `website` |  |  |
| `_build_hierarchy_datastructure` | internal rule | self | `website` |  |  |
| `filter_duplicate` | operation | self | `website` |  | Filter current recordset only keeping the most suitable view per distinct key. Every non-accessible view will be removed from the set:    * In non website context, every view with a website will be removed   * In a website context, every view from another website |
| `_get_cached_visibility` | preparation rule | self | `website` |  |  |
| `_handle_visibility` | internal rule | self, do_raise | `website` |  | Check the visibility set on the main view and raise 403 if you should not have access. Order is: Public, Connected, Has group, Password  It only check the visibility on the main content, others views called stay available in rpc. |
| `_get_base_lang` | preparation rule | self | `website` |  | Returns the default language of the website as the base language if the record is bound to it |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_xml` | ValidationError | Invalid view %(name)s definition in %(file)s | `base` |
| `_check_xml` | ValidationError | Error while validating view (%(view)s):  %(error)s | `base` |
| `_check_groups` | ValidationError | Inherited view cannot have '%(attr)s' defined on the record. Use '%(attr)s' attributes inside the view definition | `base` |
| `_check_000_inheritance` | ValidationError | You cannot create recursive inherited views. | `base` |
| `_validate_xml_encoding` | UserError | Unicode strings with encoding declaration are not supported in XML. Remove the encoding declaration. | `base` |
| `create` | ValidationError | Missing view architecture. | `base` |
| `create` | ValidationError | Invalid view type: '%(view_type)s'. You might have used an invalid starting tag in the architecture. Allowed types are: %(valid_types)s | `base` |
| `_check_view_access` | AccessError | error | `base` |
| `save_embedded_field` | ValidationError | Invalid field value for %(field_name)s: %(value)s | `html_editor` |
| `_copy_custom_snippet_translations` | ValidationError | str(e) | `html_editor` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |
| `group_website_restricted_editor` | no | yes | no | no | `website` |
| `group_website_designer` | yes | yes | yes | yes | `website` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| website_designer: Manage Website and qWeb view | `[(4, ref('group_website_designer'))]` | `[('type', '=', 'qweb')]` | True | True | True | True |
| website_designer: global view | `[(4, ref('group_website_designer'))]` | `[('type', '!=', 'qweb')]` | True | False | False | False |
| Administration Settings: Manage all views | `[(4, ref('base.group_system'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Website View Visibility Public | `[(4, ref('base.group_public'))]` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', False))]` | True | False | False | False |
| Website View Visibility Connected | `[(4, ref('base.group_portal'))]` | `['\|', ('type', '!=', 'qweb'), ('visibility', 'in', ('public', 'connected', False))]` | True | False | False | False |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_view_form` | form |  | `name`, `type`, `model_id`, `priority`, `active`, `inherit_id`, `mode`, `model_data_id`, `xml_id`, `warning_info`, `arch_db`, `arch_base`, `group_ids`, `inherit_children_ids`, `id`, `priority`, `name`, `xml_id`, `active` |  |  | `base` |
| `base.view_view_tree` | list |  | `priority`, `name`, `type`, `model`, `xml_id`, `inherit_id` |  |  | `base` |
| `base.view_view_search` | search |  | `name`, `key`, `model`, `inherit_id`, `type`, `arch_db` |  | `Form`, `List`, `Kanban`, `Search`, `QWeb`, `Modified Architecture`, `Active`, `Inactive`, `Model`, `Type`, `Inherit` | `base` |
| `web.view_view_form_inherit_view` | xpath | `base.view_view_form` | `invalid_locators` |  |  | `web` |
| `website.view_arch_only` | form |  | `arch` |  |  | `website` |
| `website.view_view_form_extend` | field | `base.view_view_form` | `inherit_id` |  |  | `website` |
| `website.view_view_tree_inherit_website` | list | `base.view_view_tree` |  |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_ui_view` | Views |  |  | `{'search_default_active': 1, 'check_translations': 1}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.ui.view.json`; views: `../../../schemas/interfaces/views/ir.ui.view.json`.
