# Qweb Field Datetime (`ir.qweb.field.datetime`)

**Transport name:** `ir.qweb.field.datetime`  
**Storage name:** `ir_qweb_field_datetime`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`

Description: Qweb Field Datetime

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_available_options` | operation | self | `base` | model |  |
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `attributes` | operation | self, record, field_name, options, values | `html_editor` | model |  |
| `from_html` | operation | self, model, field, element | `html_editor` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `from_html` | ValidationError | The datetime %(value)s does not match the format %(format)s | `html_editor` |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.datetime.json`.
