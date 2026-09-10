# Purchase Report (`purchase.report`)

**Transport name:** `purchase.report`  
**Storage name:** `purchase_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase`  
**Extended by packages:** `purchase_stock`

Description: Purchase Report

## Identity and behavior

- Default ordering: `date_order desc, price_total desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (30)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date_order` | Order Date | date and time |  | read only |
| `state` | Status | selection |  | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `partner_id` | Vendor | many to one | `res.partner` | read only |
| `date_approve` | Confirmation Date | date and time |  | read only |
| `product_uom_id` | Reference Unit of Measure | many to one | `uom.uom` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `currency_id` | Currency | many to one | `res.currency` | read only |
| `user_id` | Buyer | many to one | `res.users` | read only |
| `delay` | Days to Confirm | float |  | read only; precision `[16, 2]`; aggregated with avg; Help: Amount of time between purchase approval and order by date. |
| `delay_pass` | Days to Receive | float |  | read only; precision `[16, 2]`; aggregated with avg; Help: Amount of time between date planned and order by date for each purchase order line. |
| `price_total` | Total | monetary |  | read only |
| `price_average` | Average Cost | monetary |  | read only; aggregated with avg |
| `nbr_lines` | # of Lines | integer |  | read only |
| `category_id` | Product Category | many to one | `product.category` | read only |
| `product_tmpl_id` | Product Template | many to one | `product.template` | read only |
| `country_id` | Partner Country | many to one | `res.country` | read only |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` | read only |
| `commercial_partner_id` | Commercial Entity | many to one | `res.partner` | read only |
| `weight` | Gross Weight | float |  | read only |
| `volume` | Volume | float |  | read only |
| `order_id` | Order | many to one | `purchase.order` | read only |
| `untaxed_total` | Untaxed Total | monetary |  | read only |
| `qty_ordered` | Qty Ordered | float |  | read only |
| `qty_received` | Qty Received | float |  | read only |
| `qty_billed` | Qty Billed | float |  | read only |
| `qty_to_be_billed` | Qty to be Billed | float |  | read only |
| `picking_type_id` | Warehouse | many to one | `stock.warehouse` | read only |
| `effective_date` | Effective Date | date and time |  |  |
| `days_to_arrival` | Effective Days To Arrival | float |  | read only; precision `[16, 2]`; aggregated with avg |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft RFQ |
| `sent` | RFQ Sent |
| `to approve` | To Approve |
| `purchase` | Purchase Order |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_table_query` | internal rule | self | `purchase` |  | Report needs to be dynamic to take into account multi-company selected + multi-currency rates |
| `_select` | internal rule | self | `purchase_stock`, `purchase` |  |  |
| `_from` | internal rule | self | `purchase_stock`, `purchase` |  |  |
| `_where` | internal rule | self | `purchase` |  |  |
| `_group_by` | internal rule | self | `purchase_stock`, `purchase` |  |  |
| `_read_group_select` | internal rule | self, aggregate_spec, query | `purchase` |  | This override allows us to correctly calculate the average price of products. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase Order Report multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase.view_purchase_order_pivot` | pivot |  | `category_id`, `order_id`, `untaxed_total`, `price_total` |  |  | `purchase` |
| `purchase.view_purchase_order_graph` | graph |  | `date_approve`, `untaxed_total` |  |  | `purchase` |
| `purchase.purchase_report_view_tree` | list |  | `date_order`, `order_id`, `partner_id`, `product_id`, `category_id`, `user_id`, `company_id`, `qty_ordered`, `qty_received`, `qty_billed`, `currency_id`, `untaxed_total`, `price_total`, `state` |  |  | `purchase` |
| `purchase.view_purchase_order_search` | search |  | `partner_id`, `product_id`, `user_id`, `company_id`, `date_order`, `date_approve`, `category_id` |  | `Requests for Quotation`, `Purchase Orders`, `Confirmation Date Last Year`, `filter_date_order`, `filter_date_approve`, `Vendor`, `Vendor Country`, `Buyer`, `Product`, `Product Category`, `Status`, `Company`, `Order Date`, `Confirmation Date` | `purchase` |
| `purchase_stock.purchase_report_view_search` | xpath | `purchase.view_purchase_order_search` | `picking_type_id` |  |  | `purchase_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase.action_purchase_order_report_all` | Purchase Analysis | graph,pivot |  | `{                 'search_default_orders': 1,                 'search_default_filter_date_approve': 1}` | current | `purchase` |

Machine-readable definition: `../../../schemas/data/entities/purchase.report.json`; views: `../../../schemas/interfaces/views/purchase.report.json`.
