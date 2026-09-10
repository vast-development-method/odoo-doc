# Stock supplier replenishment information (`stock.replenishment.info`)

**Transport name:** `stock.replenishment.info`  
**Storage name:** `stock_replenishment_info`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `purchase_stock`, `mrp`

Description: Stock supplier replenishment information

## Identity and behavior

- Display name field: `orderpoint_id`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `orderpoint_id` | Orderpoint | many to one | `stock.warehouse.orderpoint` |  |
| `product_id` | Product | many to one | `product.product` | related through path `orderpoint_id.product_id` |
| `product_uom_name` | Product Unit of measure Name | single line text |  | related through path `orderpoint_id.product_uom_name` |
| `product_min_qty` | Min | float |  | required; related through path `orderpoint_id.product_min_qty` |
| `product_max_qty` | Max | float |  | required; related through path `orderpoint_id.product_max_qty` |
| `qty_to_order` | Qty To Order | float |  | related through path `orderpoint_id.qty_to_order` |
| `json_lead_days` | JavaScript Object Notation Lead Days | single line text |  | computed by rule `_compute_json_lead_days` (not stored) |
| `json_replenishment_graph` | JavaScript Object Notation Replenishment Graph | single line text |  | computed by rule `_compute_json_replenishment_graph` (not stored) |
| `based_on` | Based on | selection |  | required; default `one_month`; Help: Estimate the sales volume for the period based on past period or order the forecasted quantity for that period. |
| `percent_factor` | Percent Factor | integer |  | required; default `100` |
| `warehouseinfo_ids` | Warehouseinfo | one to many |  | related through path `orderpoint_id.warehouse_id.resupply_route_ids` |
| `wh_replenishment_option_ids` | Wh Replenishment Option | one to many | `stock.replenishment.option` | computed by rule `_compute_wh_replenishment_options` (not stored); inverse field `replenishment_info_id` |
| `supplierinfo_id` | Supplierinfo | many to one |  | related through path `orderpoint_id.supplier_id` |
| `supplierinfo_ids` | Supplierinfo | many to many | `product.supplierinfo` | computed by rule `_compute_supplierinfo_ids` and stored |
| `show_vendor_tab` | Show Vendor Tab | boolean |  | computed by rule `_compute_show_vendor_tab` (not stored) |
| `bom_id` | Bill of materials | many to one |  | related through path `orderpoint_id.bom_id` |
| `bom_ids` | Bill of materials | many to many | `mrp.bom` | computed by rule `_compute_bom_ids` and stored |
| `show_bom_tab` | Show Bill of materials Tab | boolean |  | computed by rule `_compute_show_bom_tab` (not stored) |

## Selection values

### `based_on` (Based on)

| Value | Label |
|---|---|
| `one_week` | Last 7 days |
| `one_month` | Last 30 days |
| `three_months` | Last 3 months |
| `one_year` | Last 12 months |
| `last_year` | Same month last year |
| `last_year_2` | Next month last year |
| `last_year_3` | After next month last year |
| `last_year_quarter` | Last year quarter |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_wh_replenishment_options` | computation | self | `stock` | depends: `orderpoint_id` |  |
| `_get_lead_days_and_description` | preparation rule | self | `stock` |  |  |
| `_compute_json_lead_days` | computation | self | `stock` | depends: `orderpoint_id` |  |
| `_get_period_of_time` | preparation rule | self | `stock` |  |  |
| `_prepare_graph_data` | preparation rule | self, daily_demand | `stock` |  |  |
| `_compute_json_replenishment_graph` | computation | self | `stock` | depends: `orderpoint_id`, `based_on`, `percent_factor`, `product_min_qty`, `product_max_qty` |  |
| `_compute_supplierinfo_ids` | computation | self | `purchase_stock` | depends: `orderpoint_id` |  |
| `_compute_show_vendor_tab` | computation | self | `purchase_stock` | depends: `orderpoint_id` |  |
| `_compute_bom_ids` | computation | self | `mrp` | depends: `orderpoint_id` |  |
| `_compute_show_bom_tab` | computation | self | `mrp` | depends: `orderpoint_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | no | `stock` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_stock_replenishment_info_stock_mrp_inherit` | xpath | `stock.view_stock_replenishment_info` | `bom_id`, `bom_ids` |  |  | `mrp` |
| `purchase_stock.view_stock_replenishment_info_stock_purchase_inherit` | xpath | `stock.view_stock_replenishment_info` | `supplierinfo_id`, `supplierinfo_ids` |  |  | `purchase_stock` |
| `stock.view_stock_replenishment_info` | form |  | `orderpoint_id`, `qty_to_order`, `product_min_qty`, `product_uom_name`, `product_max_qty`, `product_uom_name`, `json_lead_days`, `based_on`, `percent_factor`, `product_min_qty`, `product_uom_name`, `product_max_qty`, `product_uom_name`, `json_replenishment_graph`, `warehouseinfo_ids`, `wh_replenishment_option_ids` | `Save`, `Close` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_replenishment_info` | Replenishment Information | form |  | `{'default_orderpoint_id': active_id}` | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.replenishment.info.json`; views: `../../../schemas/interfaces/views/stock.replenishment.info.json`.
