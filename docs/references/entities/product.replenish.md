# Product Replenish (`product.replenish`)

**Transport name:** `product.replenish`  
**Storage name:** `product_replenish`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `purchase_stock`, `mrp`

Description: Product Replenish

## Identity and behavior

- Mixins (classical inheritance): `stock.replenish.mixin`
- Company consistency is checked automatically on company-bound relations

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required |
| `product_has_variants` | Has variants | boolean |  | required; default  |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unity of measure | many to one | `uom.uom` | required; restricted by domain `[('id', 'in', allowed_uom_ids)]` |
| `forecast_uom_id` | Forecast Unit of measure | many to one |  | related through path `product_id.uom_id` |
| `quantity` | Quantity | float |  | required; default `1` |
| `date_planned` | Scheduled Date | date and time |  | required; computed by rule `_compute_date_planned` and stored; precomputed before insertion; Help: Date at which the replenishment should take place. |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | required; must belong to the same company |
| `company_id` | Company | many to one | `res.company` |  |
| `forecasted_quantity` | Forecasted Quantity | float |  | computed by rule `_compute_forecasted_quantity` (not stored) |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_product_id` | on change | self | `stock` | onchange: `product_id`, `warehouse_id` |  |
| `_compute_allowed_uom_ids` | computation | self | `mrp`, `stock` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id`; depends: `product_id.bom_ids`, `product_id.bom_ids.product_uom_id` |  |
| `_compute_forecasted_quantity` | computation | self | `stock` | depends: `warehouse_id`, `product_id` |  |
| `_compute_date_planned` | computation | self | `mrp`, `purchase_stock`, `stock` | depends: `route_id`; depends: `route_id`, `supplier_id` |  |
| `default_get` | lifecycle override | self, fields | `purchase_stock`, `stock` | model |  |
| `_get_date_planned` | preparation rule | self, route_id, **kwargs | `mrp`, `purchase_stock`, `stock` |  |  |
| `launch_replenishment` | operation | self | `stock` |  |  |
| `_prepare_orderpoint_values` | preparation rule | self | `stock` |  |  |
| `_prepare_run_values` | preparation rule | self | `purchase_stock`, `stock` |  |  |
| `_get_record_to_notify` | preparation rule | self, date | `mrp`, `purchase_stock`, `stock` |  |  |
| `_get_replenishment_order_notification_link` | preparation rule | self, move | `mrp`, `purchase_stock`, `stock` |  |  |
| `_compute_allowed_route_ids` | computation | self | `stock` |  |  |
| `_get_replenishment_order_notification` | preparation rule | self, move | `stock` |  |  |
| `_get_route_domain` | preparation rule | self, product_tmpl_id | `mrp`, `purchase_stock`, `stock` |  |  |
| `_onchange_supplier_id` | on change | self | `purchase_stock` | onchange: `route_id` |  |
| `action_stock_replenishment_info` | user action | self | `purchase_stock` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `launch_replenishment` | UserError | error | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase_stock.view_product_replenish_form_inherit_stock` | xpath | `stock.view_product_replenish` | `show_vendor`, `supplier_id` |  |  | `purchase_stock` |
| `stock.view_product_replenish` | form |  | `product_id`, `forecasted_quantity`, `forecast_uom_id`, `product_tmpl_id`, `product_has_variants`, `allowed_route_ids`, `company_id`, `quantity`, `product_uom_id`, `date_planned`, `warehouse_id`, `route_id` | `Confirm`, `Discard` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_product_replenish` | Low on stock? Let's replenish. | form |  |  | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/product.replenish.json`; views: `../../../schemas/interfaces/views/product.replenish.json`.
