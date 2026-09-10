# Import Module (`base.import.module`)

**Transport name:** `base.import.module`  
**Storage name:** `base_import_module`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base_import_module`

Description: Import Module

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `module_file` | Module .ZIP file | binary |  | required |
| `state` | Status | selection |  | read only; default `init` |
| `import_message` | Import Message | multi line text |  |  |
| `force` | Force init | boolean |  | Help: Force init mode even if installed. (will update `noupdate='1'` records) |
| `with_demo` | Import demo data of module | boolean |  |  |
| `modules_dependencies` | Modules Dependencies | multi line text |  |  |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `init` | init |
| `done` | done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `import_module` | operation | self | `base_import_module` |  |  |
| `get_dependencies_to_install_names` | operation | self | `base_import_module` |  |  |
| `action_module_open` | user action | self | `base_import_module` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base_import_module` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_import_module.view_base_module_import` | form |  | `state`, `modules_dependencies`, `module_file`, `force`, `with_demo`, `import_message` | `Install`, `Cancel`, `Close` |  | `base_import_module` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base_import_module.action_view_base_module_import` | Import Module | form |  |  | new | `base_import_module` |

Machine-readable definition: `../../../schemas/data/entities/base.import.module.json`; views: `../../../schemas/interfaces/views/base.import.module.json`.
