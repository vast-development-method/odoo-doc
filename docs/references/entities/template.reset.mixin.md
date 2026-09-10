# Template Reset Mixin (`template.reset.mixin`)

**Transport name:** `template.reset.mixin`  
**Storage name:** `template_reset_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`

Description: Template Reset Mixin

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `template_fs` | Template Filename | single line text |  | not copied on duplication; Help: File from where the template originates. Used to reset broken template. |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `_load_records_write` | internal rule | self, values | `mail` |  |  |
| `_override_translation_term` | internal rule | self, module_name, xml_ids | `mail` |  |  |
| `reset_template` | operation | self | `mail` |  | Resets the Template with values given in source file. We ignore the case of template being overridden in another modules because it is extremely less likely to happen. This method also tries to reset the translation terms for the current user lang (all langs are not supported due to costly file operation). |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `reset_template` | UserError | The following email templates could not be reset because their related source files could not be found: - %s | `mail` |

Machine-readable definition: `../../../schemas/data/entities/template.reset.mixin.json`.
