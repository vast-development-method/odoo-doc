# Stock average cost Justifier (`stock.avco.report`)

**Transport name:** `stock.avco.report`  
**Storage name:** `stock_avco_report`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock_account`

Description: Stock AVCO Justifier

## Identity and behavior

- Default ordering: `date desc, id desc`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Date | date |  | required |
| `user_id` | User | many to one | `res.users` | required |
| `company_id` | Company | many to one | `res.company` | required |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `product_id` | Product | many to one | `product.product` | required |
| `reference` | Reference | single line text |  | required |
| `description` | Description | multi line text |  | required |
| `res_model_name` | Resource Model Name | selection |  | required |
| `quantity` | Added Quantity | float |  | required |
| `value` | Value | float |  | required |
| `added_value` | Added Value | float |  | computed by rule `_compute_cumulative_fields` (not stored) |
| `total_quantity` | Total Quantity | float |  | computed by rule `_compute_cumulative_fields` (not stored) |
| `total_value` | Total Value | float |  | computed by rule `_compute_cumulative_fields` (not stored) |
| `avco_value` | average cost Value | float |  | computed by rule `_compute_cumulative_fields` (not stored) |
| `justification` | Justification | multi line text |  | computed by rule `_compute_justification` (not stored) |

## Selection values

### `res_model_name` (Resource Model Name)

| Value | Label |
|---|---|
| `stock.move` | Stock Move |
| `product.value` | Product Value |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `stock_account` |  |  |
| `_compute_cumulative_fields` | computation | self | `stock_account` |  |  |
| `_compute_justification` | computation | self | `stock_account` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `stock.group_stock_manager` | no | yes | no | no | `stock_account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock_account.stock_avco_report_view_list` | list |  | `currency_id`, `reference`, `product_id`, `date`, `description`, `quantity`, `added_value`, `value`, `total_value`, `total_quantity`, `avco_value`, `justification` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_account.stock_avco_report_action` | Unit Cost History | list | `[("product_id", "=", active_id)]` |  |  | `stock_account` |

Machine-readable definition: `../../../schemas/data/entities/stock.avco.report.json`; views: `../../../schemas/interfaces/views/stock.avco.report.json`.
