# Create Menu Wizard (`wizard.ir.model.menu.create`)

**Transport name:** `wizard.ir.model.menu.create`  
**Storage name:** `wizard_ir_model_menu_create`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `base`

Description: Create Menu Wizard

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `menu_id` | Parent Menu | many to one | `ir.ui.menu` | required; on delete of the target: cascade |
| `name` | Menu Name | single line text |  | required |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `menu_create` | operation | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | no | `base` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_menu_create` | form |  | `name`, `menu_id` | `Create Menu`, `Cancel` |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.act_menu_create` | Create Menu | form |  | `{'model_id': active_id}` | new | `base` |

Machine-readable definition: `../../../schemas/data/entities/wizard.ir.model.menu.create.json`; views: `../../../schemas/interfaces/views/wizard.ir.model.menu.create.json`.
