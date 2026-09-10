# Inventory Adjustment Reference / Reason (`stock.inventory.adjustment.name`)

**Transport name:** `stock.inventory.adjustment.name`  
**Storage name:** `stock_inventory_adjustment_name`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`

Description: Inventory Adjustment Reference / Reason

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `quant_ids` | Quant | many to many | `stock.quant` |  |
| `inventory_adjustment_name` | Inventory Reason | single line text |  | default `Physical Inventory` |
| `counting_date` | Counting Date | date and time |  | default computed dynamically (fields.Datetime.now); Help: Date at which the resulting moves will be dated. |
| `accounting_date` | Accounting Date | date |  | Help: Date at which the accounting entries will be created in case of automated inventory valuation. If empty, the inventory date will be used. |
| `should_show_accounting_date` | Should Show Accounting Date | boolean |  | computed by rule `_compute_should_show_accounting_date` (not stored) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_quants_context` | preparation rule | self | `stock_account`, `stock` |  |  |
| `action_apply` | user action | self | `stock` |  |  |
| `_compute_should_show_accounting_date` | computation | self | `stock_account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_inventory_adjustment_name_form_view` | form |  | `inventory_adjustment_name`, `counting_date` | `Update Quantities`, `Discard` |  | `stock` |
| `stock_account.stock_inventory_adjustment_name_form_view_inherit_stock_account` | xpath | `stock.stock_inventory_adjustment_name_form_view` | `accounting_date` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_inventory_adjustement_name` | Physical Inventory | form |  | `{             'default_quant_ids': active_ids         }` | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.inventory.adjustment.name.json`; views: `../../../schemas/interfaces/views/stock.inventory.adjustment.name.json`.
