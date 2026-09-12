# Model Data (`ir.model.data`)

**Transport name:** `ir.model.data`  
**Storage name:** `ir_model_data`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `website`

Description: Model Data

## Identity and behavior

- Default ordering: `module, model, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | External Identifier | single line text |  | required; Help: External Key/Identifier that can be used for data integration with third-party systems |
| `complete_name` | Complete identifier | single line text |  | computed by rule `_compute_complete_name` (not stored) |
| `model` | Model Name | single line text |  | required |
| `module` | Module | single line text |  | required; default  |
| `res_id` | Record identifier | many to one by reference |  | Help: ID of the target record in the database |
| `noupdate` | Non Updatable | boolean |  | default  |
| `reference` | Reference | single line text |  | read only; computed by rule `_compute_reference` (not stored) |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_nospaces` | Constraint | `CHECK(name NOT LIKE '% %')` | External IDs cannot contain spaces | `base` |
| `_module_name_uniq_index` | UniqueIndex | `(module, name)` |  | `base` |
| `_model_res_id_index` | Index | `(model, res_id)` |  | `base` |

## Operations (20)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_complete_name` | computation | self | `base` | depends: `module`, `name` |  |
| `_compute_reference` | computation | self | `base` | depends: `model`, `res_id` |  |
| `_compute_display_name` | computation | self | `base` | depends: `res_id`, `model`, `complete_name` |  |
| `_xmlid_lookup` | internal rule | self, xmlid | `base` | model | Low level xmlid lookup Return (res_model, res_id) or raise ValueError if not found |
| `_xmlid_to_res_model_res_id` | internal rule | self, xmlid, raise_if_not_found | `base` | model | Return (res_model, res_id) |
| `_xmlid_to_res_id` | internal rule | self, xmlid, raise_if_not_found | `base` | model | Returns res_id |
| `check_object_reference` | operation | self, module, xml_id, raise_on_access_error | `base` | model | Returns (model, res_id) corresponding to a given module and xml_id (cached), if and only if the user has the necessary access rights to see that object, otherwise raise a ValueError if raise_on_access_error is True or returns a tuple (model found, False) |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  | Regular unlink method, but make sure to clear the caches. |
| `_lookup_xmlids` | internal rule | self, xml_ids, model | `base` |  | Look up the given XML ids of the given model. |
| `_update_xmlids` | internal rule | self, data_list, update | `base` | model | Create or update the given XML ids.  :param data_list: list of dicts with keys `xml_id` (XMLID to     assign), `noupdate` (flag on XMLID), `record` (target record). :param update: should be `True` when upgrading a module |
| `_build_insert_xmlids_values` | internal rule | self | `base` |  |  |
| `_build_update_xmlids_query` | internal rule | self, sub_rows, update | `base` |  |  |
| `_load_xmlid` | internal rule | self, xml_id | `base` | model | Simply mark the given XML id as being loaded, and return the corresponding record. |
| `_module_data_uninstall` | internal rule | self, modules_to_remove | `base` | model | Deletes all the records referenced by the ir.model.data entries `ids` along with their corresponding database backed (including dropping tables, columns, FKs, etc, as long as there is no other ir.model.data entry holding a reference to them (which indicates that they are still owned by another module). Attempts to perform the deletion in an appropriate order to maximize the chance of gracefully deleting all records. This step is performed as part of the full uninstallation of a module. |
| `_process_end_unlink_record` | background operation | self, record | `base`, `website` | model |  |
| `_process_end` | background operation | self, modules | `base` | model | Clear records removed from updated module data. This method is called at the end of the module loading process. It is meant to removed records that are no longer present in the updated data. Such records are recognised as the one with an xml id and a module in ir_model_data and noupdate set to false, but not present in self.pool.loaded_xmlids. |
| `toggle_noupdate` | operation | self, model, res_id | `base` | model | Toggle the noupdate flag on the external id of the record |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_object_reference` | AccessError | Not enough access rights on the external ID "%(module)s.%(xml_id)s" | `base` |
| `_module_data_uninstall` | AccessError | Administrator access is required to uninstall a module | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | no | no | no | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_data_form` | form |  | `complete_name`, `module`, `name`, `noupdate`, `write_date`, `create_date`, `display_name`, `model`, `res_id`, `reference` |  |  | `base` |
| `base.view_model_data_list` | list |  | `complete_name`, `display_name`, `model`, `res_id` |  |  | `base` |
| `base.view_model_data_search` | search |  | `name`, `module`, `model`, `res_id`, `noupdate` |  | `Updatable`, `Module`, `Model` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_data` | External Identifiers |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.data.json`; views: `../../../schemas/interfaces/views/ir.model.data.json`.
