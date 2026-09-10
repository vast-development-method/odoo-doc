# Module Uninstall (`base.module.uninstall`)

**Transport name:** `base.module.uninstall`  
**Storage name:** `base_module_uninstall`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `base_import_module`

Description: Module Uninstall

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `show_all` | Show All | boolean |  |  |
| `module_ids` | Module(s) | many to many | `ir.module.module` | required; read only; on delete of the target: cascade; restricted by domain `[["state", "in", ["installed", "to upgrade", "to install"]]]` |
| `impacted_module_ids` | Impacted modules | many to many | `ir.module.module` | computed by rule `_compute_impacted_module_ids` (not stored) |
| `model_ids` | Impacted data models | many to many | `ir.model` | computed by rule `_compute_model_ids` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_modules` | preparation rule | self | `base` |  | Return all the modules impacted by self. |
| `_compute_impacted_module_ids` | computation | self | `base` | depends: `module_ids`, `show_all` |  |
| `_modules_to_display` | internal rule | self, modules | `base_import_module`, `base` | model |  |
| `_get_models` | preparation rule | self | `base`, `mail` |  | Return the models (ir.model) to consider for the impact. |
| `_compute_model_ids` | computation | self | `base` | depends: `impacted_module_ids` |  |
| `_onchange_module_ids` | on change | self | `base` | onchange: `module_ids` |  |
| `action_uninstall` | user action | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_base_module_uninstall` | form |  | `show_all`, `impacted_module_ids`, `state`, `icon`, `shortdesc`, `summary`, `name`, `model_ids`, `name`, `count` | `Uninstall`, `Discard` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.module.uninstall.json`; views: `../../../schemas/interfaces/views/base.module.uninstall.json`.
