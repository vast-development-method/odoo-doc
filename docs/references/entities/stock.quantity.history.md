# Stock Quantity History (`stock.quantity.history`)

**Transport name:** `stock.quantity.history`  
**Storage name:** `stock_quantity_history`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Stock Quantity History

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `inventory_datetime` | Inventory at Date | date and time |  | default computed dynamically (fields.Datetime.now); Help: Choose a date to get the inventory at that date |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `open_at_date` | operation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.view_stock_quantity_history` | form |  | `inventory_datetime` | `Confirm`, `Cancel` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_inventory_at_date` | Inventory at Date | form |  |  | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.quantity.history.json`; views: `../../../schemas/interfaces/views/stock.quantity.history.json`.
