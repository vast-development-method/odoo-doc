# Module Activation Review (`base.module.install.review`)

**Transport name:** `base.module.install.review`  
**Storage name:** `base_module_install_review`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base_install_request`

Description: Module Activation Review

## Identity and behavior

- Display name field: `module_id`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `module_id` | Module | many to one | `ir.module.module` | required; read only; on delete of the target: cascade; restricted by domain `[["state", "=", "uninstalled"]]` |
| `module_ids` | Depending Apps | many to many | `ir.module.module` | computed by rule `_compute_modules_description` (not stored) |
| `modules_description` | Modules Description | rich text |  | computed by rule `_compute_modules_description` (not stored) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_modules_description` | computation | self | `base_install_request` | depends: `module_id` |  |
| `_get_depending_apps` | preparation rule | self, module | `base_install_request` | model |  |
| `action_install_module` | user action | self | `base_install_request` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_depending_apps` | UserError | No module selected. | `base_install_request` |
| `_get_depending_apps` | UserError | The module is already installed. | `base_install_request` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base_install_request` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_install_request.base_module_install_review_view_form` | form |  | `module_id`, `module_ids`, `modules_description` | `Install App`, `Cancel` |  | `base_install_request` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base_install_request.action_base_module_install_review` | You are about to install an extra application | form |  | `{ 'default_module_id': active_id }` | new | `base_install_request` |

Machine-readable definition: `../../../schemas/data/entities/base.module.install.review.json`; views: `../../../schemas/interfaces/views/base.module.install.review.json`.
