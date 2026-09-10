# Point of Sale Orders Report (`report.pos.order`)

**Transport name:** `report.pos.order`  
**Storage name:** `report_pos_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_hr`

Description: Point of Sale Orders Report

## Identity and behavior

- Default ordering: `date desc`
- Display name field: `order_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Order Date | date and time |  | read only |
| `order_id` | Order | many to one | `pos.order` | read only |
| `partner_id` | Customer | many to one | `res.partner` | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `product_tmpl_id` | Product Template | many to one | `product.template` | read only |
| `state` | Status | selection |  | read only |
| `user_id` | User | many to one | `res.users` | read only |
| `price_total` | Total Price | float |  | read only |
| `price_sub_total` | Subtotal w/o discount | float |  | read only |
| `price_subtotal_excl` | Subtotal w/o Tax | float |  | read only |
| `total_discount` | Total Discount | float |  | read only |
| `average_price` | Average Price | float |  | read only; aggregated with avg |
| `company_id` | Company | many to one | `res.company` | read only |
| `nbr_lines` | Sale Line Count | integer |  | read only |
| `product_qty` | Product Quantity | integer |  | read only |
| `journal_id` | Journal | many to one | `account.journal` | read only |
| `delay_validation` | Delay Validation | integer |  | read only |
| `product_categ_id` | Product Category | many to one | `product.category` | read only |
| `pos_categ_id` | Point of Sale Category | many to one | `pos.category` | read only |
| `invoiced` | Invoiced | boolean |  | read only |
| `config_id` | Point of Sale | many to one | `pos.config` | read only |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | read only |
| `session_id` | Session | many to one | `pos.session` | read only |
| `margin` | Margin | float |  | read only |
| `payment_method_id` | Payment Method | many to one | `pos.payment.method` | read only |
| `employee_id` | Employee | many to one | `hr.employee` | read only |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | New |
| `paid` | Paid |
| `done` | Posted |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_select` | internal rule | self | `point_of_sale`, `pos_hr` |  |  |
| `_from` | internal rule | self | `point_of_sale` |  |  |
| `_group_by` | internal rule | self | `point_of_sale`, `pos_hr` |  |  |
| `init` | lifecycle override | self | `point_of_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | no | yes | no | no | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Point Of Sale Order Analysis multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_report_pos_order_pivot` | pivot |  | `product_categ_id`, `date`, `order_id`, `product_qty`, `price_total` |  |  | `point_of_sale` |
| `point_of_sale.view_report_pos_order_graph` | graph |  | `product_categ_id`, `price_total` |  |  | `point_of_sale` |
| `point_of_sale.report_pos_order_view_tree` | list |  | `date`, `order_id`, `partner_id`, `product_id`, `product_categ_id`, `config_id`, `company_id`, `price_total`, `state` |  |  | `point_of_sale` |
| `point_of_sale.view_report_pos_order_search` | search |  | `date`, `config_id`, `partner_id`, `product_id`, `product_categ_id` |  | `Invoiced`, `Not Invoiced`, `Not Cancelled`, `filter_date`, `User`, `Point of Sale`, `Product`, `Product Category`, `Payment Method`, `Point of Sale Category`, `Order Date` | `point_of_sale` |
| `pos_hr.view_report_pos_order_search_inherit` | xpath | `point_of_sale.view_report_pos_order_search` |  |  | `Employee` | `pos_hr` |
| `pos_hr.report_pos_order_view_tree` | field | `point_of_sale.report_pos_order_view_tree` | `product_categ_id`, `employee_id` |  |  | `pos_hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_report_pos_order_all` | Orders Analysis | graph,pivot |  | `{'group_by':[], 'search_default_not_cancelled': 1}` |  | `point_of_sale` |
| `point_of_sale.action_report_pos_order_all_filtered` | Orders Analysis | graph,pivot |  | `{             'search_default_config_id': [active_id],             'default_config_id': active_id}` |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/report.pos.order.json`; views: `../../../schemas/interfaces/views/report.pos.order.json`.
