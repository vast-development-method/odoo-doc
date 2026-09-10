# Warn Insufficient Quantity (`stock.warn.insufficient.qty`)

**Transport name:** `stock.warn.insufficient.qty`  
**Storage name:** `stock_warn_insufficient_qty`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`

Description: Warn Insufficient Quantity

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required |
| `location_id` | Location | many to one | `stock.location` | required; restricted by domain `[('usage', '=', 'internal')]` |
| `quant_ids` | Quant | many to many | `stock.quant` | computed by rule `_compute_quant_ids` (not stored) |
| `quantity` | Quantity | float |  | required |
| `product_uom_name` | Unit | single line text |  | required |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_reference_document_company_id` | preparation rule | self | `stock` |  |  |
| `_compute_quant_ids` | computation | self | `stock` | depends: `product_id` |  |
| `action_done` | user action | self | `stock` |  |  |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_warn_insufficient_qty_form_view` | form |  | `product_id`, `location_id`, `quant_ids`, `location_id`, `lot_id`, `quantity` | `Discard`, `Confirm` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.warn.insufficient.qty.json`; views: `../../../schemas/interfaces/views/stock.warn.insufficient.qty.json`.
