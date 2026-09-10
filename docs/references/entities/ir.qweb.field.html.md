# Qweb Field hypertext markup language (`ir.qweb.field.html`)

**Transport name:** `ir.qweb.field.html`  
**Storage name:** `ir_qweb_field_html`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`, `website`

Description: Qweb Field HTML

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `value_to_html` | operation | self, value, options | `base`, `website` | model |  |
| `attributes` | operation | self, record, field_name, options, values | `html_editor` | model |  |
| `from_html` | operation | self, model, field, element | `html_editor` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.html.json`.
