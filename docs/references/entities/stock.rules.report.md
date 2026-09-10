# Stock Rules report (`stock.rules.report`)

**Transport name:** `stock.rules.report`  
**Storage name:** `stock_rules_report`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`

Description: Stock Rules report

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required |
| `warehouse_ids` | Warehouses | many to many | `stock.warehouse` | required; Help: Show the routes that apply on selected warehouses. |
| `product_has_variants` | Has variants | boolean |  | required; default  |
| `so_route_ids` | Apply specific routes | many to many | `stock.route` | restricted by domain `[('sale_selectable', '=', True)]`; Help: Choose to apply SO lines specific routes. |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `stock` | model |  |
| `_prepare_report_data` | preparation rule | self | `sale_stock`, `stock` |  |  |
| `print_report` | operation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_stock.view_stock_rules_report_sale` | xpath | `stock.view_stock_rules_report` | `so_route_ids` |  |  | `sale_stock` |
| `stock.view_stock_rules_report` | form |  | `product_tmpl_id`, `product_has_variants`, `product_id`, `warehouse_ids` | `Overview`, `Cancel` |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_stock_rules_report` | Stock Rules Report | form |  |  | new | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.rules.report.json`; views: `../../../schemas/interfaces/views/stock.rules.report.json`.
