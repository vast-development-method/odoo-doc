# Reset View Architecture Wizard (`reset.view.arch.wizard`)

**Transport name:** `reset.view.arch.wizard`  
**Storage name:** `reset_view_arch_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Reset View Architecture Wizard

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `view_id` | View | many to one | `ir.ui.view` |  |
| `view_name` | View Name | single line text |  | related through path `view_id.name` |
| `has_diff` | Has Diff | boolean |  | computed by rule `_compute_arch_diff` (not stored) |
| `arch_diff` | Architecture Diff | rich text |  | read only; computed by rule `_compute_arch_diff` (not stored) |
| `reset_mode` | Reset Mode | selection |  | required; default `soft` |
| `compare_view_id` | Compare To View | many to one | `ir.ui.view` |  |
| `arch_to_compare` | Arch To Compare To | multi line text |  | computed by rule `_compute_arch_diff` (not stored) |

## Selection values

### `reset_mode` (Reset Mode)

| Value | Label |
|---|---|
| `soft` | Restore previous version (soft reset). |
| `hard` | Reset to file version (hard reset). |
| `other_view` | Reset to another view. |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `base` | model |  |
| `_compute_arch_diff` | computation | self | `base` | depends: `reset_mode`, `view_id`, `compare_view_id` | Depending of `reset_mode`, return the differences between the current view arch and either its previous arch, its initial arch or another view arch. |
| `reset_view_button` | operation | self | `base` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | ValidationError | Can't compare more than two views. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | no | `base` |
| `base.group_erp_manager` | yes | yes | yes | no | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.reset_view_arch_wizard_view` | form |  | `has_diff`, `view_id`, `view_name`, `compare_view_id`, `reset_mode`, `arch_diff` | `Reset View`, `Cancel` |  | `base` |
| `website.reset_view_arch_wizard_view` | field | `base.reset_view_arch_wizard_view` | `compare_view_id` |  |  | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.reset_view_arch_wizard_action` | Compare/Reset | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/reset.view.arch.wizard.json`; views: `../../../schemas/interfaces/views/reset.view.arch.wizard.json`.
