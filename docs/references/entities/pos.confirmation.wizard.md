# Confirmation Wizard (`pos.confirmation.wizard`)

**Transport name:** `pos.confirmation.wizard`  
**Storage name:** `pos_confirmation_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`

Description: Confirmation Wizard

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `message` | Message | multi line text |  | read only; default computed dynamically (_default_message) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_selected_orders` | operation | self | `point_of_sale` |  |  |
| `_default_message` | preparation rule | self | `point_of_sale` |  |  |
| `action_confirm` | user action | self | `point_of_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_confirm_action_wizard` | form |  | `message` | `Confirm`, `Cancel` |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_confirm_action_wizard` | Confirm Action | form |  |  | new | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.confirmation.wizard.json`; views: `../../../schemas/interfaces/views/pos.confirmation.wizard.json`.
