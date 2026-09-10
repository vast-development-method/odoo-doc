# Language Export (`base.language.export`)

**Transport name:** `base.language.export`  
**Storage name:** `base_language_export`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Language Export

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | File Name | single line text |  | read only |
| `lang` | Language | selection |  | required; default computed dynamically (NEW_LANG_KEY) |
| `format` | File Format | selection |  | required; default `po` |
| `export_type` | Export Type | selection |  | required; default `module` |
| `modules` | Apps To Export | many to many | `ir.module.module` | restricted by domain `[["state", "=", "installed"]]`; association table `rel_modules_langexport` |
| `model_id` | Model to Export | many to one | `ir.model` | restricted by domain `[["transient", "=", false]]` |
| `model_name` | Model Name | single line text |  | related through path `model_id.model` |
| `domain` | Model Domain | single line text |  | default `[]` |
| `data` | File | binary |  | read only |
| `state` | State | selection |  | default `choose` |

## Selection values

### `format` (File Format)

| Value | Label |
|---|---|
| `csv` | CSV File |
| `po` | PO File |
| `tgz` | TGZ Archive |

### `export_type` (Export Type)

| Value | Label |
|---|---|
| `module` | Module |
| `model` | Model |

### `state` (State)

| Value | Label |
|---|---|
| `choose` | choose |
| `get` | get |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_languages` | preparation rule | self | `base` | model |  |
| `act_getfile` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.wizard_lang_export` | form |  | `name`, `lang`, `format`, `export_type`, `modules`, `model_id`, `model_name`, `domain`, `model_id`, `modules`, `data` | `Export`, `Cancel`, `Close` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_wizard_lang_export` | Export Translation | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.language.export.json`; views: `../../../schemas/interfaces/views/base.language.export.json`.
