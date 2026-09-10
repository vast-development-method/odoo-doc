# Qweb Field Image (`ir.qweb.field.image_url`)

**Transport name:** `ir.qweb.field.image_url`  
**Storage name:** `ir_qweb_field_image_url`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `web`

Description: Qweb Field Image

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field.image`

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `_get_src_urls` | preparation rule | self, record, field_name, options | `web` |  |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.image_url.json`.
