# Fields Converter (`ir.fields.converter`)

**Transport name:** `ir.fields.converter`  
**Storage name:** `ir_fields_converter`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`

Description: Fields Converter

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_format_import_error` | internal rule | self, error_type, error_msg, error_params, error_args | `base` | model |  |
| `_get_import_field_path` | preparation rule | self, field, value | `base` |  | Rebuild field path for import error attribution to the right field. This method uses the 'parent_fields_hierarchy' context key built during treatment of one2many fields (_str_to_one2many). As the field to import is the last of the chain (child_id/child_id2/field_to_import), we need to retrieve the complete hierarchy in case of error in order to assign the error to the correct column in the import UI.  :param (str) field: field in which the value will be imported. :param (str or list) value:     - str: in most of the case the value we want to import into a field is a string (or a number).     - |
| `for_model` | operation | self, model, fromtype, savepoint | `base` | model | Returns a converter object for the model. A converter is a callable taking a record-ish (a dictionary representing an odoo record with values of typetag ``fromtype``) and returning a converted records matching what :meth:`odoo.models.Model.write` expects.  :param model: :class:`odoo.models.Model` for the conversion base :param fromtype: :param savepoint: savepoint to rollback to on error :returns: a converter callable :rtype: (record: dict, logger: (field, error) -> None) -> dict |
| `to_field` | operation | self, model, field, fromtype, savepoint | `base` | model | Fetches a converter for the provided field object, from the specified type.  A converter is simply a callable taking a value of type ``fromtype`` (or a composite of ``fromtype``, e.g. list or dict) and returning a value acceptable for a write() on the field ``field``.  By default, tries to get a method on itself with a name matching the pattern ``_$fromtype_to_$field.type`` and returns it.  Converter callables can either return a value and a list of warnings to their caller or raise ``ValueError``, which will be interpreted as a validation & conversion failure.  ValueError can have either one  |
| `_str_to_json` | internal rule | self, model, field, value, savepoint | `base` |  |  |
| `_str_to_properties` | internal rule | self, model, field, value, savepoint | `base` |  |  |
| `_str_to_boolean` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_to_integer` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_to_float` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_id` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_to_date` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_input_tz` | internal rule | self | `base` | model |  |
| `_str_to_datetime` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_get_boolean_translations` | preparation rule | self, src | `base` | model |  |
| `_get_selection_translations` | preparation rule | self, field, src | `base` | model |  |
| `_str_to_selection` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `db_id_for` | operation | self, model, field, subfield, value, savepoint | `base` | model | Finds a database id for the reference ``value`` in the referencing subfield ``subfield`` of the provided field of the provided model.  :param model: model to which the field belongs :param field: relational field for which references are provided :param subfield: a relational subfield allowing building of refs to                  existing records: ``None`` for a name_search,                  ``id`` for an external id and ``.id`` for a database                  id :param value: value of the reference to match to an actual record :param savepoint: savepoint for rollback on errors :return: a pair |
| `_xmlid_to_record_id` | internal rule | self, xmlid, model | `base` |  | Return the record id corresponding to the given external id, provided that the record actually exists; otherwise return ``None``. |
| `_referencing_subfield` | internal rule | self, record | `base` |  | Checks the record for the subfields allowing referencing (an existing record in an other table), errors out if it finds potential conflicts (multiple referencing subfields) or non-referencing subfields returns the name of the correct subfield.  :param record: :return: the record subfield to use for referencing and a list of warnings :rtype: str, list |
| `_str_to_many2one` | internal rule | self, model, field, values, savepoint | `base` | model |  |
| `_str_to_many2one_reference` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_to_many2many` | internal rule | self, model, field, value, savepoint | `base` | model |  |
| `_str_to_one2many` | internal rule | self, model, field, records, savepoint | `base` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.fields.converter.json`.
