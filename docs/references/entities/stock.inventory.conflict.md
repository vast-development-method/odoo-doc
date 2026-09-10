# Conflict in Inventory (`stock.inventory.conflict`)

**Transport name:** `stock.inventory.conflict`  
**Storage name:** `stock_inventory_conflict`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Conflict in Inventory

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `quant_ids` | Quants | many to many | `stock.quant` | association table `stock_conflict_quant_rel` |
| `quant_to_fix_ids` | Conflicts | many to many | `stock.quant` |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_keep_counted_quantity` | user action | self | `stock` |  |  |
| `action_keep_difference` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_inventory_conflict_form_view` | form |  | `quant_ids`, `quant_to_fix_ids`, `id`, `tracking`, `company_id`, `product_id`, `location_id`, `lot_id`, `package_id`, `owner_id`, `quantity`, `inventory_quantity`, `inventory_diff_quantity`, `product_uom_id`, `company_id` | `Keep Counted Quantity`, `Keep Difference`, `Discard` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.inventory.conflict.json`; views: `../../../schemas/interfaces/views/stock.inventory.conflict.json`.
