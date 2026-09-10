# Demo (`ir.demo`)

**Transport name:** `ir.demo`  
**Storage name:** `ir_demo`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Demo

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `install_demo` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.demo_force_install_form` | form |  |  | `Oops, no!`, `Yes, I understand the risks` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.demo_force_install_action` | Load demo data | form |  |  | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.demo.json`; views: `../../../schemas/interfaces/views/ir.demo.json`.
