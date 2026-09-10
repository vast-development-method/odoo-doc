# Fields (`ir.model.fields`)

**Transport name:** `ir.model.fields`  
**Storage name:** `ir_model_fields`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `base_sparse_field`, `website`

Description: Fields

## Identity and behavior

- Default ordering: `name, id`
- Display name field: `field_description`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (45)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Field Name | single line text |  | required; default `x_`; indexed |
| `model` | Model Name | single line text |  | required; indexed; Help: The technical name of the model this field belongs to |
| `relation` | Related Model | single line text |  | Help: For relationship fields, the technical name of the target model |
| `relation_field` | Relation Field | single line text |  | Help: For one2many fields, the field on the target model that implement the opposite many2one relationship |
| `relation_field_id` | Relation field | many to one | `ir.model.fields` | computed by rule `_compute_relation_field_id` and stored; on delete of the target: cascade |
| `model_id` | Model | many to one | `ir.model` | required; indexed; on delete of the target: cascade; Help: The model this field belongs to |
| `field_description` | Field Label | single line text |  | required; default ; translatable |
| `help` | Field Help | multi line text |  | translatable |
| `ttype` | Field Type | selection |  | required; on delete of the target: {"serialized": "cascade"}; extended by packages `base_sparse_field` |
| `selection` | Selection Options (Deprecated) | single line text |  | computed by rule `_compute_selection` (not stored); writable through an inverse rule |
| `selection_ids` | Selection Options | one to many | `ir.model.fields.selection` | inverse field `field_id` |
| `copied` | Copied | boolean |  | computed by rule `_compute_copied` and stored; Help: Whether the value is copied when duplicating a record. |
| `related` | Related Field Definition | single line text |  | Help: The corresponding related field, if any. This must be a dot-separated list of field names. |
| `related_field_id` | Related Field | many to one | `ir.model.fields` | computed by rule `_compute_related_field_id` and stored; on delete of the target: cascade |
| `required` | Required | boolean |  |  |
| `readonly` | Readonly | boolean |  |  |
| `index` | Indexed | boolean |  |  |
| `translate` | Translatable | selection |  | Help: Whether values for this field can be translated (enables the translation mechanism for that field) |
| `company_dependent` | Company Dependent | boolean |  | read only; Help: Whether values for this field is company dependent |
| `size` | Size | integer |  |  |
| `state` | Type | selection |  | required; read only; default `manual`; indexed |
| `on_delete` | On Delete | selection |  | default `set null`; Help: On delete property for many2one fields |
| `domain` | Domain | single line text |  | default `[]`; Help: The optional domain to restrict possible values for relationship fields, specified as a Python expression defining a list of triplets. For example: [('color','=','red')] |
| `groups` | Groups | many to many | `res.groups` | association table `ir_model_fields_group_rel` |
| `group_expand` | Expand Groups | boolean |  | Help: If checked, all the records of the target model will be included in a grouped result (e.g. 'Group By' filters, Kanban columns, etc.). Note that it can significantly reduce performance if the target model of the field contains a lot of records; usually used on models with few records (e.g. Stages, Job Positions, Event Types, etc.). |
| `selectable` | Selectable | boolean |  | default `True` |
| `modules` | In Apps | single line text |  | computed by rule `_in_modules` (not stored); Help: List of modules in which the field is defined |
| `relation_table` | Relation Table | single line text |  | Help: Used for custom many2many fields to define a custom relation table name |
| `column1` | Column 1 | single line text |  | Help: Column referring to the record in the model table |
| `column2` | Column 2 | single line text |  | Help: Column referring to the record in the comodel table |
| `compute` | Compute | multi line text |  | Help: Code to compute the value of the field. Iterate on the recordset 'self' and assign the field's value:      for record in self:         record['size'] = len(record.name)  Modules time, datetime, dateutil are available. |
| `depends` | Dependencies | single line text |  | Help: Dependencies of compute method; a list of comma-separated field names, like      name, partner_id.name |
| `store` | Stored | boolean |  | default `True`; Help: Whether the value is stored in the database. |
| `currency_field` | Currency field | single line text |  | Help: Name of the Many2one field holding the res.currency |
| `sanitize` | Sanitize hypertext markup language | boolean |  | default `True` |
| `sanitize_overridable` | Sanitize hypertext markup language overridable | boolean |  | default  |
| `sanitize_tags` | Sanitize hypertext markup language Tags | boolean |  | default `True` |
| `sanitize_attributes` | Sanitize hypertext markup language Attributes | boolean |  | default `True` |
| `sanitize_style` | Sanitize hypertext markup language Style | boolean |  | default  |
| `sanitize_form` | Sanitize hypertext markup language Form | boolean |  | default `True` |
| `strip_style` | Strip Style Attribute | boolean |  | default  |
| `strip_classes` | Strip Class Attribute | boolean |  | default  |
| `tracking` | Enable Ordered Tracking | integer |  | Help: If set every modification done to this field is tracked. Value is used to order tracking values. |
| `serialization_field_id` | Serialization Field | many to one | `ir.model.fields` | on delete of the target: cascade; restricted by domain `[('ttype','=','serialized'), ('model_id', '=', model_id)]`; Help: If set, this field will be stored in the sparse structure of the serialization field, instead of having its own database column. This cannot be changed after creation. |
| `website_form_blacklisted` | Blacklisted in web forms | boolean |  | default `True`; indexed; Help: Blacklist this field for web forms |

## Selection values

### `translate` (Translatable)

| Value | Label |
|---|---|
| `standard` | Translate as a whole |
| `html_translate` | Translate HTML terms |
| `xml_translate` | Translate XML terms |

### `state` (Type)

| Value | Label |
|---|---|
| `manual` | Custom Field |
| `base` | Base Field |

### `on_delete` (On Delete)

| Value | Label |
|---|---|
| `cascade` | Cascade |
| `set null` | Set NULL |
| `restrict` | Restrict |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_unique` | Constraint | `UNIQUE(model, name)` | Field names must be unique per model. | `base` |
| `_size_gt_zero` | Constraint | `CHECK (size>=0)` | Size of the field cannot be negative. | `base` |
| `_name_manual_field` | Constraint | `CHECK (state != 'manual' OR name LIKE 'x\_%')` | Custom fields must have a name that starts with 'x_'! | `base` |

## Operations (42)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_relation_field_id` | computation | self | `base` | depends: `relation`, `relation_field` |  |
| `_compute_related_field_id` | computation | self | `base` | depends: `related` |  |
| `_compute_selection` | computation | self | `base` | depends: `selection_ids` |  |
| `_inverse_selection` | inverse computation | self | `base` |  |  |
| `_compute_copied` | computation | self | `base` | depends: `ttype`, `related`, `compute` |  |
| `_in_modules` | computation | self | `base` | depends |  |
| `_check_domain` | validation | self | `base` | constrains: `domain` |  |
| `_check_name` | validation | self | `base` | constrains: `name` |  |
| `_related_field` | internal rule | self | `base` |  | Return the ``ir.model.fields`` record corresponding to ``self.related``. |
| `_check_related` | validation | self | `base` | constrains: `related` |  |
| `_onchange_related` | on change | self | `base` | onchange: `related` |  |
| `_onchange_relation` | on change | self | `base` | onchange: `relation` |  |
| `_check_relation` | validation | self | `base` | constrains: `relation` |  |
| `_check_depends` | validation | self | `base` | constrains: `depends` | Check whether all fields in dependencies are valid. |
| `_onchange_compute` | on change | self | `base` | onchange: `compute` |  |
| `_check_relation_table` | validation | self | `base` | constrains: `relation_table` |  |
| `_check_currency_field` | validation | self | `base` | constrains: `currency_field` |  |
| `_custom_many2many_names` | internal rule | self, model_name, comodel_name | `base` | model | Return default names for the table and columns of a custom many2many field. |
| `_onchange_ttype` | on change | self | `base` | onchange: `ttype`, `model_id`, `relation` |  |
| `_onchange_relation_table` | on change | self | `base` | onchange: `relation_table` |  |
| `_check_on_delete_required_m2o` | validation | self | `base` | constrains: `required`, `ttype`, `on_delete` |  |
| `_get` | internal rule | self, model_name, name | `base` |  | Return the (sudoed) `ir.model.fields` record with the given model and name. The result may be an empty recordset if the model is not found. |
| `_get_ids` | preparation rule | self, model_name | `base` |  |  |
| `_drop_column` | internal rule | self | `base` |  |  |
| `_prepare_update` | preparation rule | self | `base` |  | Check whether the fields in ``self`` may be modified or removed. This method prevents the modification/deletion of many2one fields that have an inverse one2many, for instance. |
| `unlink` | lifecycle override | self | `base`, `mail` |  | When unlinking fields populate tracking value table with relevant information. That way if a field is removed (custom tracked, migration or any other reason) we keep the tracking and its relevant information. Do it only when unlinking fields so that we don't duplicate field information for most tracking. |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base_sparse_field`, `base` |  |  |
| `_compute_display_name` | computation | self | `base` | depends: `field_description`, `model` |  |
| `_reflect_field_params` | internal rule | self, field, model_id | `base`, `mail` |  | Return the values to write to the database for the given field. |
| `_reflect_fields` | internal rule | self, model_names | `base_sparse_field`, `base` |  | Reflect the fields of the given models. |
| `_all_manual_field_data` | internal rule | self | `base` |  |  |
| `_get_manual_field_data` | preparation rule | self, model_name | `base` |  | Return the given model's manual field data. |
| `_instanciate_attrs` | internal rule | self, field_data | `base_sparse_field`, `base`, `mail` |  | Return the parameters for a field instance for ``field_data``. |
| `_is_manual_name` | internal rule | self, name | `base` | model |  |
| `get_field_string` | operation | self, model_name | `base` | model | Return the translation of fields strings in the context's language. Note that the result contains the available translations only.  :param model_name: the name of a model :return: the model's fields' strings as a dictionary `{field_name: field_string}` |
| `get_field_help` | operation | self, model_name | `base` | model | Return the translation of fields help in the context's language. Note that the result contains the available translations only.  :param model_name: the name of a model :return: the model's fields' help as a dictionary `{field_name: field_help}` |
| `get_field_selection` | operation | self, model_name, field_name | `base` | model | Return the translation of a field's selection in the context's language. Note that the result contains the available translations only.  :param model_name: the name of the field's model :param field_name: the name of the field :return: the fields' selection as a list |
| `_get_fields_cached` | preparation rule | self, model_name | `base` | model | Return the translated information of all model field's in the context's language. Note that the result contains the available translations only.  :param model_name: the name of the field's model :return: {field_name: {id, help, field_description, [selection]}} |
| `init` | lifecycle override | self | `website` |  |  |
| `_check_if_used_in_website_form` | validation | self | `website` | ondelete | Prevent field deletion if used in a website form. |
| `formbuilder_whitelist` | operation | self, model, fields | `website` | model | :param str model: name of the model on which to whitelist fields :param list(str) fields: list of fields to whitelist on the model :return: nothing of import |

## Validation and error messages (30)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_domain` | ValidationError | An error occurred while evaluating the domain: %(error)s | `base` |
| `_check_name` | ValidationError | msg | `base` |
| `_related_field` | UserError | Unknown field name "%(field_name)s" in related field "%(related_field)s" | `base` |
| `_related_field` | UserError | Non-relational field name "%(field_name)s" in related field "%(related_field)s" | `base` |
| `_related_field` | UserError | Field "%(field_name)s" in related path "%(related_field)s" is not searchable. Non-searchable fields cannot be used in related fields. | `base` |
| `_check_related` | ValidationError | Related field "%(related_field)s" does not have type "%(type)s" | `base` |
| `_check_related` | ValidationError | Related field "%(related_field)s" does not have comodel "%(comodel)s" | `base` |
| `_check_relation` | ValidationError | Unknown model name '%s' in Related Model | `base` |
| `_check_depends` | UserError | Empty dependency in “%s” | `base` |
| `_check_depends` | UserError | Compute method cannot depend on field 'id' | `base` |
| `_check_depends` | UserError | Unknown field “%(field)s” in dependency “%(dependency)s” | `base` |
| `_check_depends` | UserError | Non-relational field “%(field)s” in dependency “%(dependency)s” | `base` |
| `_check_currency_field` | ValidationError | Currency field does not have type many2one | `base` |
| `_check_currency_field` | ValidationError | Currency field should have a res.currency relation | `base` |
| `_check_currency_field` | ValidationError | Currency field is empty and there is no fallback field in the model | `base` |
| `_check_currency_field` | ValidationError | Unknown field specified “%s” in currency_field | `base` |
| `_check_on_delete_required_m2o` | ValidationError | The m2o field %s is required but declares its ondelete policy as being 'set null'. Only 'restrict' and 'cascade' make sense. | `base` |
| `_prepare_update` | UserError | This column contains module data and cannot be removed! | `base` |
| `_prepare_update` | UserError | The field '%(field)s' cannot be removed because the field '%(other_field)s' depends on it. | `base` |
| `_prepare_update` | UserError | Cannot rename/delete fields that are still present in views: Fields: %(fields)s View: %(view)s | `base` |
| `create` | UserError | Model %s does not exist! | `base` |
| `create` | UserError | Many2one %(field)s on model %(model)s does not exist! | `base` |
| `write` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! | `base` |
| `write` | UserError | Changing the model of a field is forbidden! | `base` |
| `write` | UserError | Changing the type of a field is not yet supported. Please drop it and create it again! | `base` |
| `write` | UserError | Can only rename one field at a time! | `base` |
| `write` | UserError | Changing the storing system for field "%s" is not allowed. | `base_sparse_field` |
| `write` | UserError | Renaming sparse field "%s" is not allowed | `base_sparse_field` |
| `_reflect_fields` | UserError | Serialization field "%(serialization_field)s" not found for sparse field %(sparse_field)s! | `base_sparse_field` |
| `_check_if_used_in_website_form` | ValidationError | The field '%(field)s' cannot be deleted because it is referenced in a website view. Model: %(model)s View: %(view)s | `website` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | no | no | no | `base` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_fields_form` | form |  | `name`, `field_description`, `model_id`, `ttype`, `help`, `required`, `readonly`, `store`, `index`, `copied`, `state`, `modules`, `translate`, `size`, `relation`, `group_expand`, `on_delete`, `relation_field`, `relation_table`, `column1`, `column2`, `domain`, `currency_field`, `sanitize`, `sanitize_overridable`, `sanitize_tags`, `sanitize_attributes`, `sanitize_style`, `sanitize_form`, `strip_style`, `strip_classes`, `selection_ids`, `sequence`, `value`, `name`, `related`, `depends`, `compute`, `groups` |  |  | `base` |
| `base.view_model_fields_tree` | list |  | `name`, `field_description`, `model_id`, `ttype`, `state`, `index`, `store`, `readonly`, `relation` |  |  | `base` |
| `base.view_model_fields_search` | search |  | `name`, `model_id`, `ttype`, `required`, `readonly`, `relation` |  | `Required`, `Readonly`, `Custom`, `Base`, `Translate`, `Model`, `Field Type` | `base` |
| `base_sparse_field.field_form_view` | field | `base.view_model_fields_form` | `related`, `serialization_field_id` |  |  | `base_sparse_field` |
| `mail.field_form_view` | field | `base.view_model_fields_form` | `copied`, `state`, `tracking` |  |  | `mail` |
| `website.ir_model_fields_view` | xpath | `base.view_model_fields_form` | `website_form_blacklisted` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_fields` | Fields |  |  | `{}` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.fields.json`; views: `../../../schemas/interfaces/views/ir.model.fields.json`.
