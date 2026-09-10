# Inventory Adjustment Warning (`stock.inventory.warning`)

**Transport name:** `stock.inventory.warning`  
**Storage name:** `stock_inventory_warning`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Inventory Adjustment Warning

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `quant_ids` | Quant | many to many | `stock.quant` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_reset` | user action | self | `stock` |  |  |
| `action_set` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.inventory_warning_reset_view` | form |  |  | `Continue`, `Discard` |  | `stock` |
| `stock.inventory_warning_set_view` | form |  |  | `Continue`, `Discard` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.inventory.warning.json`; views: `../../../schemas/interfaces/views/stock.inventory.warning.json`.
