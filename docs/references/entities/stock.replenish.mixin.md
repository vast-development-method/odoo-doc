# Product Replenish Mixin (`stock.replenish.mixin`)

**Transport name:** `stock.replenish.mixin`  
**Storage name:** `stock_replenish_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`  
**Extended by packages:** `purchase_stock`, `mrp`, `stock_dropshipping`, `mrp_subcontracting_dropshipping`

Description: Product Replenish Mixin

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `route_id` | Preferred Route | many to one | `stock.route` | must belong to the same company; Help: Apply specific route for the replenishment instead of product's default routes. |
| `allowed_route_ids` | Allowed Route | many to many | `stock.route` | computed by rule `_compute_allowed_route_ids` (not stored) |
| `supplier_id` | Vendor | many to one | `product.supplierinfo` |  |
| `show_vendor` | Show Vendor | boolean |  | computed by rule `_compute_show_vendor` (not stored) |
| `bom_id` | Bill of Material | many to one | `mrp.bom` |  |
| `show_bom` | Show Bill of materials | boolean |  | computed by rule `_compute_show_bom` (not stored) |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_route_ids` | computation | self | `stock` | depends: `product_id`, `product_tmpl_id` |  |
| `_get_allowed_route_domain` | preparation rule | self | `mrp_subcontracting_dropshipping`, `stock_dropshipping`, `stock` |  |  |
| `_compute_show_vendor` | computation | self | `purchase_stock` | depends: `route_id` |  |
| `_get_show_vendor` | preparation rule | self, route | `purchase_stock` |  |  |
| `_compute_show_bom` | computation | self | `mrp` | depends: `route_id` |  |
| `_get_show_bom` | preparation rule | self, route | `mrp` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.replenish.mixin.json`.
