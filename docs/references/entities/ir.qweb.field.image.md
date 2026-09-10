# Qweb Field Image (`ir.qweb.field.image`)

**Transport name:** `ir.qweb.field.image`  
**Storage name:** `ir_qweb_field_image`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `base`  
**Extended by packages:** `web`, `html_editor`, `web_unsplash`

Description: Qweb Field Image

## Identity and behavior

- Mixins (classical inheritance): `ir.qweb.field`

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_src_data_b64` | preparation rule | self, value, options | `base` | model |  |
| `value_to_html` | operation | self, value, options | `base` | model |  |
| `_get_src_urls` | preparation rule | self, record, field_name, options | `web` |  | Considering the rendering options, returns the src and data-zoom-image urls.  :return: src, src_zoom urls :rtype: tuple |
| `record_to_html` | operation | self, record, field_name, options | `web` | model |  |
| `from_html` | operation | self, model, field, element | `html_editor`, `web_unsplash` | model |  |
| `load_local_url` | operation | self, url | `html_editor` |  |  |
| `load_remote_url` | operation | self, url | `html_editor` |  |  |

Machine-readable definition: `../../../schemas/data/entities/ir.qweb.field.image.json`.
