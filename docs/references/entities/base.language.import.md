# Language Import (`base.language.import`)

**Transport name:** `base.language.import`  
**Storage name:** `base_language_import`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Language Import

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Language Name | single line text |  | required |
| `code` | ISO Code | single line text |  | required; Help: ISO Language and Country code, e.g. en_US |
| `data` | File | binary |  | required |
| `filename` | File Name | single line text |  | required |
| `overwrite` | Overwrite Existing Terms | boolean |  | default `True`; Help: If you enable this option, existing translations (including custom ones) will be overwritten and replaced by those in this file |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `import_lang` | operation | self | `base` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `import_lang` | UserError | File "%(file_name)s" not imported due to format mismatch or a malformed file. (Valid formats are .csv, .po)  Technical Details: %(error_message)s | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_base_import_language` | form |  | `name`, `code`, `data`, `filename`, `overwrite` | `Import`, `Cancel` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_view_base_import_language` | Import Translation | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.language.import.json`; views: `../../../schemas/interfaces/views/base.language.import.json`.
