# Vendor Delay Report (`vendor.delay.report`)

**Transport name:** `vendor.delay.report`  
**Storage name:** `vendor_delay_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase_stock`

Description: Vendor Delay Report

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Vendor | many to one | `res.partner` | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `category_id` | Product Category | many to one | `product.category` | read only |
| `date` | Effective Date | date and time |  | read only |
| `qty_total` | Total Quantity | float |  | read only |
| `qty_on_time` | On-Time Quantity | float |  | read only |
| `on_time_rate` | On-Time Delivery Rate | float |  | read only |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `purchase_stock` |  |  |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `purchase_stock` |  |  |
| `_read_group` | lifecycle override | self, domain, groupby, aggregates, having, offset, limit, order | `purchase_stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_stock` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase_stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase_stock.vendor_delay_report_filter` | search |  | `partner_id`, `product_id` |  | `Effective Date Last Year` | `purchase_stock` |
| `purchase_stock.vendor_delay_report_view_graph` | graph |  | `product_id`, `on_time_rate` |  |  | `purchase_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase_stock.action_purchase_vendor_delay_report` | On-time Delivery | graph |  | `{'search_default_later_than_a_year_ago':1}` | current | `purchase_stock` |

Machine-readable definition: `../../../schemas/data/entities/vendor.delay.report.json`; views: `../../../schemas/interfaces/views/vendor.delay.report.json`.
