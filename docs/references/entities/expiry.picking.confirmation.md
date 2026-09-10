# Confirm Expiry (`expiry.picking.confirmation`)

**Transport name:** `expiry.picking.confirmation`  
**Storage name:** `expiry_picking_confirmation`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `product_expiry`  
**Extended by packages:** `mrp_product_expiry`

Description: Confirm Expiry

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `lot_ids` | Lot | many to many | `stock.lot` | required; read only |
| `picking_ids` | Picking | many to many | `stock.picking` | read only |
| `description` | Description | single line text |  | computed by rule `_compute_descriptive_fields` (not stored) |
| `show_lots` | Show Lots | boolean |  | computed by rule `_compute_descriptive_fields` (not stored) |
| `production_ids` | Production | many to many | `mrp.production` | read only |
| `workorder_id` | Workorder | many to one | `mrp.workorder` | read only |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_descriptive_fields` | computation | self | `mrp_product_expiry`, `product_expiry` | depends: `lot_ids` |  |
| `process` | operation | self | `product_expiry` |  |  |
| `process_no_expired` | operation | self | `product_expiry` |  | Remove the expired mls and confirm the picking. |
| `confirm_produce` | operation | self | `mrp_product_expiry` |  |  |
| `confirm_workorder` | operation | self | `mrp_product_expiry` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `product_expiry` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_product_expiry.confirm_expiry_view_mrp_inherit` | xpath | `product_expiry.confirm_expiry_view` | `picking_ids`, `production_ids`, `workorder_id` |  |  | `mrp_product_expiry` |
| `product_expiry.confirm_expiry_view` | form |  | `description`, `show_lots`, `lot_ids`, `product_id`, `name` | `Confirm`, `Proceed except expired`, `Discard` |  | `product_expiry` |

Machine-readable definition: `../../../schemas/data/entities/expiry.picking.confirmation.json`; views: `../../../schemas/interfaces/views/expiry.picking.confirmation.json`.
