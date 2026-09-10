# Demo Failure wizard (`ir.demo_failure.wizard`)

**Transport name:** `ir.demo_failure.wizard`  
**Storage name:** `ir_demo_failure_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Demo Failure wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `failure_ids` | Demo Installation Failures | one to many | `ir.demo_failure` | read only; inverse field `wizard_id` |
| `failures_count` | Failures Count | integer |  | computed by rule `_compute_failures_count` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_failures_count` | computation | self | `base` | depends: `failure_ids` |  |
| `done` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.demo_failures_dialog` | form |  | `failures_count`, `failure_ids`, `module_id`, `error` | `Ok` |  | `base` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `base.demo_failure_action` | Failed to install demo data for some modules, demo disabled | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/ir.demo_failure.wizard.json`; views: `../../../schemas/interfaces/views/ir.demo_failure.wizard.json`.
