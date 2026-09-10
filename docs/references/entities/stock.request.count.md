# Stock Request an Inventory Count (`stock.request.count`)

**Transport name:** `stock.request.count`  
**Storage name:** `stock_request_count`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Stock Request an Inventory Count

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `inventory_date` | Scheduled at | date |  | required; default computed dynamically (fields.Datetime.now); Help: Choose a date to get the inventory at that date |
| `user_id` | Assign to | many to one | `res.users` | restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('stock.group_stock_user').id)]` |
| `quant_ids` | Quant | many to many | `stock.quant` |  |
| `show_expected_quantity` | Show Expected Quantity | boolean |  | computed by rule `_compute_show_expected_quantity` (not stored); writable through an inverse rule; Help: If the user can see the expected quantity or not |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_show_expected_quantity` | computation | self | `stock` |  |  |
| `_set_show_expected_quantity` | internal rule | self | `stock` |  |  |
| `action_request_count` | user action | self | `stock` |  |  |
| `_get_quants_to_count` | preparation rule | self | `stock` |  |  |
| `_get_values_to_write` | preparation rule | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_inventory_request_count_form_view` | form |  | `quant_ids`, `user_id`, `inventory_date`, `show_expected_quantity` | `Confirm`, `Discard` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_request_count` | Inventory Request | form |  | `{             'default_quant_ids': active_ids         }` | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.request.count.json`; views: `../../../schemas/interfaces/views/stock.request.count.json`.
