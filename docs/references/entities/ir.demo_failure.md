# Demo failure (`ir.demo_failure`)

**Transport name:** `ir.demo_failure`  
**Storage name:** `ir_demo_failure`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Demo failure

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `module_id` | Module | many to one | `ir.module.module` | required |
| `error` | Error | single line text |  |  |
| `wizard_id` | Wizard | many to one | `ir.demo_failure.wizard` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.demo_wizard_form_view` | form |  | `module_id`, `wizard_id`, `error` |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.demo_failure.json`; views: `../../../schemas/interfaces/views/ir.demo_failure.json`.
