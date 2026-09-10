# Qweb Field Contact (`ir.qweb.field.contact`)

**Transport name:** `ir.qweb.field.contact`  
**Storage name:** `ir_qweb_field_contact`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `html_editor`, `website`

Description: Qweb Field Contact

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field.many2one`

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_available_options` | operation | self | `base`, `website` | model |  |
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `attributes` | operation | self, record, field_name, options, values | `html_editor` | model |  |
| `get_record_to_html` | operation | self, contact_ids, options | `html_editor` | model | Helper to call the rendering of contact field. |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.contact.json`.
