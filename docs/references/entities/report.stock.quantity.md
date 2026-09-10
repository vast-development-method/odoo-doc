# Stock Quantity Report (`report.stock.quantity`)

**Transport name:** `report.stock.quantity`  
**Storage name:** `report_stock_quantity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `product_expiry`

Description: Stock Quantity Report

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Date | date |  | read only |
| `product_tmpl_id` | Product Tmpl | many to one | `product.template` | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `state` | State | selection |  | read only |
| `product_qty` | Quantity | float |  | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | read only |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `forecast` | Forecasted Stock |
| `in` | Forecasted Receipts |
| `out` | Forecasted Deliveries |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_product_qty_col` | preparation rule | self | `product_expiry`, `stock` |  |  |
| `init` | lifecycle override | self | `stock` |  | Because we can transfer a product from a warehouse to another one thanks to a stock move, we need to generate some fake stock moves before processing all of them. That way, in case of an interwarehouse transfer, we will have an outgoing stock move for the source warehouse and an incoming stock move for the destination one. To do so, we select all relevant SM (incoming, outgoing and interwarehouse), then we duplicate all these SM and edit the values:     - product_qty is kept if the SM is not the duplicated one or if the SM is an interwarehouse one         otherwise, we set the value to 0 (this |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| report_stock_quantity_flow multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_report_view_graph` | graph |  | `date`, `product_id`, `product_qty` |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/report.stock.quantity.json`; views: `../../../schemas/interfaces/views/report.stock.quantity.json`.
