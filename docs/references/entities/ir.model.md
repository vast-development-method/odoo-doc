# Models (`ir.model`)

**Transport name:** `ir.model`  
**Storage name:** `ir_model`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `bus`, `mail`, `sms`, `spreadsheet`, `website`, `mass_mailing`, `marketing_card`

Description: Models

## Identity and behavior

- Default ordering: `model`
- Display name search fields: `["name", "model"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Model Description | single line text |  | required; translatable |
| `model` | Model | single line text |  | required; default `x_` |
| `order` | Order | single line text |  | required; default `id`; Help: SQL expression for ordering records in the model; e.g. "x_sequence asc, id desc" |
| `info` | Information | multi line text |  |  |
| `field_id` | Fields | one to many | `ir.model.fields` | required; default computed dynamically (_default_field_id); inverse field `model_id` |
| `inherited_model_ids` | Inherited models | many to many | `ir.model` | computed by rule `_inherited_models` (not stored); Help: The list of models that extends the current model. |
| `state` | Type | selection |  | read only; default `manual` |
| `access_ids` | Access | one to many | `ir.model.access` | inverse field `model_id` |
| `rule_ids` | Record Rules | one to many | `ir.rule` | inverse field `model_id` |
| `abstract` | Abstract Model | boolean |  |  |
| `transient` | Transient Model | boolean |  |  |
| `modules` | In Apps | single line text |  | computed by rule `_in_modules` (not stored); Help: List of modules in which the object is defined or inherited |
| `view_ids` | Views | one to many | `ir.ui.view` | computed by rule `_view_ids` (not stored) |
| `count` | Count (Incl. Archived) | integer |  | computed by rule `_compute_count` (not stored); Help: Total number of records in this model |
| `fold_name` | Fold Field | single line text |  | Help: In a Kanban view where columns are records of this model, the value of this (boolean) field determines which column should be folded by default. |
| `is_mail_thread` | Has Mail Thread | boolean |  | default  |
| `is_mail_activity` | Has Mail Activity | boolean |  | default  |
| `is_mail_blacklist` | Has Mail Blacklist | boolean |  | default  |
| `is_mail_thread_sms` | Mail Thread text message | boolean |  | computed by rule `_compute_is_mail_thread_sms` (not stored); searchable through a search rule; default ; Help: Whether this model supports messages and notifications through SMS |
| `website_form_access` | Allowed to use in forms | boolean |  | Help: Enable the form builder feature for this model. |
| `website_form_default_field_id` | Field for custom form data | many to one | `ir.model.fields` | restricted by domain `[('model', '=', model), ('ttype', '=', 'text')]`; Help: Specify the field which will contain meta and custom form fields datas. |
| `website_form_label` | Label for form action | single line text |  | translatable; Help: Form action label. Ex: crm.lead could be 'Send an e-mail' and project.issue could be 'Create an Issue'. |
| `website_form_key` | Website Form Key | single line text |  | Help: Used in FormBuilder Registry |
| `is_mailing_enabled` | Mailing Enabled | boolean |  | computed by rule `_compute_is_mailing_enabled` (not stored); searchable through a search rule; Help: Whether this model supports marketing mailing capabilities (notably email and SMS). |

## Selection values

### `state` (Type)

| Value | Label |
|---|---|
| `manual` | Custom Object |
| `base` | Base Object |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_obj_name_uniq` | Constraint | `UNIQUE (model)` | Each model must have a unique name. | `base` |

## Operations (36)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_field_id` | preparation rule | self | `base` |  |  |
| `_inherited_models` | computation | self | `base` | depends |  |
| `_in_modules` | computation | self | `base` | depends |  |
| `_view_ids` | computation | self | `base` | depends |  |
| `_compute_count` | computation | self | `base` | depends |  |
| `_check_model_name` | validation | self | `base` | constrains: `model` |  |
| `_check_order` | validation | self | `base` | constrains: `order`, `field_id` |  |
| `_check_fold_name` | validation | self | `base` | constrains: `fold_name` |  |
| `_get` | internal rule | self, name | `base` |  | Return the (sudoed) `ir.model` record with the given name. The result may be an empty recordset if the model is not found. |
| `_get_id` | preparation rule | self, name | `base` |  |  |
| `_drop_table` | internal rule | self | `base` |  |  |
| `_unlink_if_manual` | internal rule | self | `base` | ondelete |  |
| `unlink` | lifecycle override | self | `base`, `mail` |  | Delete mail data (followers, messages, activities) associated with the models being deleted. |
| `write` | lifecycle override | self, vals | `base`, `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `name_create` | lifecycle override | self, name | `base` | model | Infer the model from the name. E.g.: 'My New Model' should become 'x_my_new_model'. |
| `_reflect_model_params` | internal rule | self, model | `base`, `mail` |  | Return the values to write to the database for the given model. |
| `_reflect_models` | internal rule | self, model_names | `base` |  | Reflect the given models. |
| `_instanciate_attrs` | internal rule | self, model_data | `base`, `mail` | model | Return the attributes to instanciate a custom model definition class corresponding to ``model_data``. |
| `_is_manual_name` | internal rule | self, name | `base` | model |  |
| `_check_manual_name` | validation | self, name | `base` | model |  |
| `display_name_for` | operation | self, models | `web` | model | Returns the display names from provided models which the current user can access. The result is the same whether someone tries to access an inexistent model or a model they cannot access. :models list(str): list of technical model names to lookup (e.g. `["res.partner"]`) :return: list of dicts of the form `{ "model", "display_name" }` (e.g. `{ "model": "res_partner", "display_name": "Contact"}`) |
| `_display_name_for` | internal rule | self, models | `web` | model |  |
| `_is_valid_for_model_selector` | internal rule | self, model | `web` | model |  |
| `get_available_models` | operation | self | `web` | model | Return the list of models the current user has access to, with their corresponding display name. |
| `_get_definitions` | preparation rule | self, model_names | `mail`, `web` |  |  |
| `_get_model_definitions` | preparation rule | self, model_names_to_fetch | `bus`, `mail` |  |  |
| `_compute_is_mail_thread_sms` | computation | self | `sms` | depends: `is_mail_thread` |  |
| `_search_is_mail_thread_sms` | search rule | self, operator, value | `sms` |  |  |
| `has_searchable_parent_relation` | operation | self, model_names | `spreadsheet` | readonly; model |  |
| `_get_form_writable_fields` | preparation rule | self, property_origins | `website` |  | Restriction of "authorized fields" (fields which can be used in the form builders) to fields which have actually been opted into form builders and are writable. By default no field is writable by the form builder. |
| `get_authorized_fields` | operation | self, model_name, property_origins | `website` | model | Return the fields of the given model name as a mapping like method `fields_get`. |
| `get_compatible_form_models` | operation | self | `website` | model |  |
| `_compute_is_mailing_enabled` | computation | self | `mass_mailing` |  |  |
| `_search_is_mailing_enabled` | search rule | self, operator, value | `mass_mailing` |  |  |
| `_delete_linked_campaigns` | internal rule | self | `marketing_card` | ondelete | Remove campaigns on removed models. |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_model_name` | ValidationError | The model name can only contain lowercase characters, digits, underscores and dots. | `base` |
| `_check_order` | ValidationError | str(e) | `base` |
| `_check_order` | ValidationError | Unable to order by %s: fields used for ordering must be present on the model and stored. | `base` |
| `_check_fold_name` | ValidationError | The value of 'Fold Field' should be a field name of the model. | `base` |
| `_unlink_if_manual` | UserError | Model “%s” contains module data and cannot be removed. | `base` |
| `write` | UserError | Field %s cannot be modified on models. | `base` |
| `_check_manual_name` | ValidationError | The model name must start with 'x_'. | `base` |
| `write` | UserError | Only custom models can be modified. | `mail` |
| `write` | UserError | Field "Mail Thread" cannot be changed to "False". | `mail` |
| `write` | UserError | Field "Mail Activity" cannot be changed to "False". | `mail` |
| `write` | UserError | Field "Mail Blacklist" cannot be changed to "False". | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | no | no | no | `base` |
| `mass_mailing.group_mass_mailing_user` | no | yes | no | no | `mass_mailing` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_form` | form |  | `name`, `model`, `order`, `abstract`, `transient`, `state`, `modules`, `field_id`, `name`, `field_description`, `ttype`, `required`, `readonly`, `index`, `state`, `name`, `field_description`, `ttype`, `help`, `required`, `readonly`, `store`, `index`, `copied`, `state`, `modules`, `translate`, `size`, `relation`, `on_delete`, `relation_field`, `relation_table`, `column1`, `column2`, `domain`, `selection_ids`, `sequence`, `value`, `name`, `related`, `depends`, `compute`, `groups`, `access_ids`, `name`, `group_id`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink`, `rule_ids`, `name`, `groups`, `domain_force`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink`, `info`, `view_ids` | `Create a Menu` |  | `base` |
| `base.view_model_tree` | list |  | `model`, `name`, `state`, `transient` |  |  | `base` |
| `base.view_model_search` | search |  | `name`, `model` |  | `Abstract`, `Transient`, `Custom`, `Base` | `base` |
| `base_sparse_field.model_form_view` | field | `base.view_model_form` | `related`, `serialization_field_id` |  |  | `base_sparse_field` |
| `mail.model_form_view` | field | `base.view_model_form` | `transient`, `is_mail_thread`, `is_mail_activity`, `is_mail_blacklist` |  |  | `mail` |
| `mail.model_search_view` | field | `base.view_model_search` | `model` |  | `Mail Thread`, `Mail Activity`, `Mail Blacklist` | `mail` |
| `website.ir_model_view` | xpath | `base.view_model_form` | `website_form_access`, `website_form_label`, `website_form_default_field_id` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_model` | Models |  |  | `{}` |  | `base` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `base.report_ir_model_overview` | Model Overview | qweb-pdf | `base.report_irmodeloverview` |  |  |

Machine-readable definition: `../../../schemas/data/entities/ir.model.json`; views: `../../../schemas/interfaces/views/ir.model.json`.
