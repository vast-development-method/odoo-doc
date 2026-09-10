# Qweb Field Relative (`ir.qweb.field.relative`)

**Transport name:** `ir.qweb.field.relative`  
**Storage name:** `ir_qweb_field_relative`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`

Description: Qweb Field Relative

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_available_options` | operation | self | `base` | model |  |
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `record_to_html` | operation | self, record, field_name, options | `base` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.relative.json`.
