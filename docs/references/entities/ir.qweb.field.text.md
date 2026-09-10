# Qweb Field Text (`ir.qweb.field.text`)

**Transport name:** `ir.qweb.field.text`  
**Storage name:** `ir_qweb_field_text`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`

Description: Qweb Field Text

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `value_to_html` | operation | self, value, options | `base` | model | Escapes the value and converts newlines to br. This is bullshit. |
| `from_html` | operation | self, model, field, element | `html_editor` | model |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.text.json`.
