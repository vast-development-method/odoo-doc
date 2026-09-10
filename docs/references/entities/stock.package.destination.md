# Stock Package Destination (`stock.package.destination`)

**Transport name:** `stock.package.destination`  
**Storage name:** `stock_package_destination`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Stock Package Destination

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_line_ids` | Move Line | many to many | `stock.move.line` | required; association table `Products` |
| `location_dest_id` | Destination location | many to one | `stock.location` | required |
| `filtered_location` | Filtered Location | one to many | `stock.location` | computed by rule `_compute_filtered_location` (not stored) |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_filtered_location` | computation | self | `stock` | depends: `move_line_ids` |  |
| `action_done` | user action | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_package_destination_form_view` | form |  | `move_line_ids`, `product_id`, `location_dest_id`, `quantity`, `lot_id`, `product_id`, `quantity`, `location_dest_id`, `filtered_location`, `location_dest_id` | `Confirm`, `Discard` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.package.destination.json`; views: `../../../schemas/interfaces/views/stock.package.destination.json`.
