# Product Category (`product.category`)

**Transport name:** `product.category`  
**Storage name:** `product_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `account`, `delivery`, `stock`, `stock_account`, `point_of_sale`, `mrp_account`

Description: Product Category

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `pos.load.mixin`
- Default ordering: `complete_name`
- Display name field: `complete_name`
- Hierarchy parent field: `parent_id` with stored hierarchy path
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; indexed (trigram) |
| `complete_name` | Complete Name | single line text |  | computed by rule `_compute_complete_name` and stored; recursive dependency |
| `parent_id` | Parent Category | many to one | `product.category` | indexed; on delete of the target: cascade |
| `parent_path` | Parent Path | single line text |  | indexed |
| `child_id` | Child Categories | one to many | `product.category` | inverse field `parent_id` |
| `product_count` | # Products | integer |  | computed by rule `_compute_product_count` (not stored); Help: The number of products under this category (Does not consider the children categories) |
| `product_properties_definition` | Product Properties | properties definition |  |  |
| `property_account_income_categ_id` | Income Account | many to one | `account.account` | value is company dependent; changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `ACCOUNT_DOMAIN`; Help: This account will be used when validating a customer invoice. |
| `property_account_expense_categ_id` | Expense Account | many to one | `account.account` | value is company dependent; changes are tracked in the message thread; on delete of the target: restrict; restricted by domain `ACCOUNT_DOMAIN`; Help: The expense is accounted for when a vendor bill is validated, except in anglo-saxon accounting with perpetual inventory valuation in which case the expense (Cost of Goods Sold account) is recognized at the customer invoice validation. |
| `route_ids` | Routes | many to many | `stock.route` | restricted by domain `[["product_categ_selectable", "=", true]]`; association table `stock_route_categ` |
| `removal_strategy_id` | Force Removal Strategy | many to one | `product.removal` | changes are tracked in the message thread; Help: Set a specific removal strategy that will be used regardless of the source location for this product category.  FIFO: products/lots that were stocked first will be moved out first. LIFO: products/lots that were stocked last will be moved out first. Closest location: products/lots closest to the target location will be moved out first. FEFO: products/lots with the closest removal date will be moved out first (the availability of this method depends on the "Expiration Dates" setting). Least Packages: FIFO but with the least number of packages possible when there are several packages containing the same product. |
| `parent_route_ids` | Parent Routes | many to many | `stock.route` | computed by rule `_compute_parent_route_ids` (not stored) |
| `total_route_ids` | Total routes | many to many | `stock.route` | read only; computed by rule `_compute_total_route_ids` (not stored); searchable through a search rule |
| `putaway_rule_ids` | Putaway Rules | one to many | `stock.putaway.rule` | inverse field `category_id` |
| `packaging_reserve_method` | Reserve Packagings | selection |  | default `partial`; Help: Reserve Only Full Packagings: will not reserve partial packagings. If customer orders 2 pallets of 1000 units each and you only have 1600 in stock, then only 1000 will be reserved Reserve Partial Packagings: allow reserving partial packagings. If customer orders 2 pallets of 1000 units each and you only have 1600 in stock, then 1600 will be reserved |
| `filter_for_stock_putaway_rule` | stock.putaway.rule | boolean |  | searchable through a search rule |
| `anglo_saxon_accounting` | Use Anglo-Saxon Accounting | boolean |  | computed by rule `_compute_anglo_saxon_accounting` (not stored); Help: If checked, the product will be valued using the Anglo-Saxon accounting method. |
| `property_valuation` | Inventory Valuation | selection |  | value is company dependent; changes are tracked in the message thread; Help: Periodic: The accounting entries are suggested manually in the inventory valuation report.         Perpetual: An accounting entry is automatically created to value the inventory when a product is billed or invoiced. |
| `property_cost_method` | Costing Method | selection |  | default computed dynamically (lambda self: self.env.company.cost_method); value is company dependent; changes are tracked in the message thread; Help: Standard Price: The products are valued at their standard cost defined on the product.         Average Cost (AVCO): The products are valued at weighted average cost.         First In First Out (FIFO): The products are valued supposing those that enter the company first will also leave it first. |
| `property_stock_journal` | Stock Journal | many to one | `account.journal` | value is company dependent; Help: When doing automated inventory valuation, this is the Accounting Journal in which entries will be automatically posted when stock moves are processed. |
| `property_stock_valuation_account_id` | Stock Valuation Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; must belong to the same company; Help: When automated inventory valuation is enabled on a product, this account will hold the current value of the products. |
| `property_price_difference_account_id` | Price Difference Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; must belong to the same company; Help: With perpetual valuation, this account will hold the price difference between the standard price and the bill price. |
| `account_stock_variation_id` | Stock Variation Account | many to one | `account.account` | related through path `property_stock_valuation_account_id.account_stock_variation_id` |
| `property_stock_account_production_cost_id` | Production Account | many to one | `account.account` | value is company dependent; on delete of the target: restrict; must belong to the same company; Help: This account will be used as a valuation counterpart for both components and final products for manufacturing orders.                 If there are any workcenter/employee costs, this value will remain on the account once the production is completed. |

## Selection values

### `packaging_reserve_method` (Reserve Packagings)

| Value | Label |
|---|---|
| `full` | Reserve Only Full Packagings |
| `partial` | Reserve Partial Packagings |

### `property_valuation` (Inventory Valuation)

| Value | Label |
|---|---|
| `periodic` | Periodic (at closing) |
| `real_time` | Perpetual (at invoicing) |

### `property_cost_method` (Costing Method)

| Value | Label |
|---|---|
| `standard` | Standard Price |
| `fifo` | First In First Out (FIFO) |
| `average` | Average Cost (AVCO) |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_complete_name` | computation | self | `product` | depends: `name`, `parent_id.complete_name` |  |
| `_compute_product_count` | computation | self | `product` |  |  |
| `_check_category_recursion` | validation | self | `product` | constrains: `parent_id` |  |
| `name_create` | lifecycle override | self, name | `product` | model |  |
| `_compute_display_name` | computation | self | `product` | depends_context: `hierarchical_naming` |  |
| `copy_data` | lifecycle override | self, default | `product` |  |  |
| `_unlink_except_delivery_category` | internal rule | self | `delivery` | ondelete |  |
| `_compute_parent_route_ids` | computation | self | `stock` | depends: `parent_id` |  |
| `_search_total_route_ids` | search rule | self, operator, value | `stock` |  |  |
| `_compute_total_route_ids` | computation | self | `stock` | depends: `route_ids`, `parent_route_ids` |  |
| `_search_filter_for_stock_putaway_rule` | search rule | self, operator, value | `stock` |  |  |
| `_compute_anglo_saxon_accounting` | computation | self | `stock_account` | depends_context: `company` |  |
| `write` | lifecycle override | self, vals | `stock_account` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_category_recursion` | ValidationError | You cannot create recursive categories. | `product` |
| `_unlink_except_delivery_category` | UserError | You cannot delete the deliveries product category as it is used on the delivery carriers products. | `delivery` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.view_category_property_form` | group | `product.product_category_form_view` | `property_account_income_categ_id`, `property_account_expense_categ_id` |  |  | `account` |
| `product.product_category_form_view` | form |  | `product_count`, `name`, `parent_id` | `%(product_template_action_all)d` |  | `product` |
| `product.product_category_list_view` | list |  | `display_name` |  |  | `product` |
| `product.product_category_search_view` | search |  | `name`, `parent_id` |  |  | `product` |
| `stock.product_category_form_view_inherit` | div | `product.product_category_form_view` |  | `Putaway Rules` |  | `stock` |
| `stock_account.view_category_property_form_stock` | group | `stock.product_category_form_view_inherit` | `property_cost_method`, `property_valuation` |  |  | `stock_account` |
| `stock_account.view_category_property_form` | field | `account.view_category_property_form` | `property_account_expense_categ_id`, `property_stock_valuation_account_id`, `account_stock_variation_id`, `property_price_difference_account_id` |  |  | `stock_account` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.product_category_action` | Internal Categories |  |  |  |  | `point_of_sale` |
| `product.product_category_action_form` | Categories |  |  |  |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `account.menu_product_product_categories` | Product Categories |  | `product.product_category_action_form` | 30 |  |
| `purchase.menu_product_category_config_purchase` |  | `purchase.menu_product_in_config_purchase` | `product.product_category_action_form` | 3 |  |
| `sale.menu_product_categories` |  |  | `product.product_category_action_form` | 20 |  |
| `stock.menu_product_category_config_stock` |  | `stock.menu_product_in_config_stock` | `product.product_category_action_form` | 2 |  |

Machine-readable definition: `../../../schemas/data/entities/product.category.json`; views: `../../../schemas/interfaces/views/product.category.json`.
