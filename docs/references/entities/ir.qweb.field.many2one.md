# Qweb Field Many to One (`ir.qweb.field.many2one`)

**Transport name:** `ir.qweb.field.many2one`  
**Storage name:** `ir_qweb_field_many2one`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`

Description: Qweb Field Many to One

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `attributes` | operation | self, record, field_name, options, values | `html_editor` | model |  |
| `from_html` | operation | self, model, field, element | `html_editor` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.many2one.json`.
