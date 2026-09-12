# Qweb Field (`ir.qweb.field`)

**Transport name:** `ir.qweb.field`  
**Storage name:** `ir_qweb_field`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`

Description: Qweb Field

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_available_options` | operation | self | `base` | model | Get the available option informations.  :rtype: dict[str, dict[str, Any]] :return: A dictionnary that maps option names' to their settings.      The settings are dict themselves and have the following keys:      type          Guaranteed, one of `'string'`, `'integer'`, `'float'`,         `'model'`, `'array'`, or `'selection'`.      string          Guaranteed      description          Optional      required          Optional, is assumed `False` when absent, otherwise         is either `True` or a string.      params          Optional      default_value          Optional, the def |
| `attributes` | operation | self, record, field_name, options, values | `base`, `html_editor` | model | attributes(record, field_name, field, options, values)  Generates the metadata attributes (prefixed by `data-oe-`) for the root node of the field conversion.  The default attributes are:  * `model`, the name of the record's model * `id` the id of the record to which the field belongs * `type` the logical field type (widget, may not match the field's   `type`, may not be any Field subclass name) * `translate`, a boolean flag (`0` or `1`) denoting whether the   field is translatable * `readonly`, has this attribute if the field is readonly * `expression`, the original express |
| `value_to_html` | operation | self, value, options | `base` | model | value_to_html(value, field, options=None)  Converts a single value to its HTML version/output :rtype: unicode |
| `record_to_html` | operation | self, record, field_name, options | `base` | model | record_to_html(record, field_name, options)  Converts the specified field of the `record` to HTML  :rtype: unicode |
| `user_lang` | operation | self | `base` | model | user_lang()  Fetches the res.lang record corresponding to the language code stored in the user's context.  :returns: Model[res.lang] |
| `value_from_string` | operation | self, value | `html_editor` |  |  |
| `from_html` | operation | self, model, field, element | `html_editor` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.json`.
