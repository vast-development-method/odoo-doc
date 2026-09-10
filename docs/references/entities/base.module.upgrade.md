# Upgrade Module (`base.module.upgrade`)

**Transport name:** `base.module.upgrade`  
**Storage name:** `base_module_upgrade`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Upgrade Module

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `module_info` | Apps to Update | multi line text |  | read only; default computed dynamically (_default_module_info) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_module_list` | operation | self | `base` | model |  |
| `_default_module_info` | preparation rule | self | `base` | model |  |
| `get_view` | lifecycle override | self, view_id, view_type, **options | `base` | model |  |
| `upgrade_module_cancel` | operation | self | `base` |  |  |
| `upgrade_module` | operation | self | `base` |  |  |
| `config` | operation | self | `base` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `upgrade_module` | UserError | The following modules are not installed or unknown: %s | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_base_module_upgrade` | form |  | `module_info` | `Confirm`, `Cancel` |  | `base` |
| `base.view_base_module_upgrade_install` | form |  |  | `Start configuration`, `Cancel` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_view_base_module_upgrade` | Apply Schedule Upgrade | form |  |  | new | `base` |
| `base.action_view_base_module_upgrade_install` | Module Upgrade Install | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/base.module.upgrade.json`; views: `../../../schemas/interfaces/views/base.module.upgrade.json`.
