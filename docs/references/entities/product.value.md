# Product Value (`product.value`)

**Transport name:** `product.value`  
**Storage name:** `product_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock_account`

Description: Product Value

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | indexed |
| `lot_id` | Lot | many to one | `stock.lot` |  |
| `move_id` | Move | many to one | `stock.move` | indexed (btree_not_null) |
| `value` | Value | monetary |  | required; currency taken from `currency_id` |
| `company_id` | Company | many to one | `res.company` | required; computed by rule `_compute_company_id` and stored; precomputed before insertion |
| `currency_id` | Currency | many to one | `res.currency` | related through path `company_id.currency_id` |
| `date` | Date | date and time |  | required; default computed dynamically (fields.Datetime.now) |
| `user_id` | User | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user) |
| `description` | Description | single line text |  |  |
| `current_value` | Current Value | monetary |  | related through path `move_id.value`; currency taken from `currency_id` |
| `current_value_details` | Current Value Details | single line text |  | computed by rule `_compute_current_value_details` (not stored) |
| `current_value_description` | Current Value Description | multi line text |  | computed by rule `_compute_value_description` (not stored) |
| `computed_value_description` | Computed Value Description | multi line text |  | computed by rule `_compute_value_description` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_company_id` | computation | self | `stock_account` | depends: `move_id`, `lot_id`, `product_id` |  |
| `_compute_current_value_details` | computation | self | `stock_account` |  |  |
| `_compute_value_description` | computation | self | `stock_account` |  |  |
| `create` | lifecycle override | self, vals_list | `stock_account` | model_create_multi |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock_account` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock_account.product_value_form_view` | form |  | `move_id`, `currency_id`, `current_value`, `current_value_details`, `value`, `description`, `current_value_description`, `computed_value_description` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_account.product_value_action` | Adjust Valuation | form |  |  |  | `stock_account` |

Machine-readable definition: `../../../schemas/data/entities/product.value.json`; views: `../../../schemas/interfaces/views/product.value.json`.
