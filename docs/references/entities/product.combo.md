# Product Combo (`product.combo`)

**Transport name:** `product.combo`  
**Storage name:** `product_combo`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale_stock`

Description: Product Combo

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `sequence` | Sequence | integer |  | default `10`; not copied on duplication |
| `company_id` | Company | many to one | `res.company` | indexed |
| `combo_item_ids` | Combo Item | one to many | `product.combo.item` | inverse field `combo_id` |
| `combo_item_count` | Product Count | integer |  | computed by rule `_compute_combo_item_count` (not stored) |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` (not stored) |
| `base_price` | Combo Price | float |  | computed by rule `_compute_base_price` (not stored); Help: The minimum price among the products in this combo. This value will be used to prorate the price of this combo with respect to the other combos in a combo product. This heuristic ensures that whatever product the user chooses in a combo, it will always be the same price. |
| `qty_max` | Maximum quantity | integer |  | default `1`; Help: Maximum number of items to select in the combo. |
| `qty_free` | Free quantity | integer |  | default `1`; Help: Number of free items included in the combo. |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_combo_item_count` | computation | self | `product` | depends: `combo_item_ids` |  |
| `_compute_currency_id` | computation | self | `product` | depends: `company_id` |  |
| `_compute_base_price` | computation | self | `product` | depends: `combo_item_ids` |  |
| `_check_combo_item_ids_not_empty` | validation | self | `product` | constrains: `combo_item_ids` |  |
| `_check_combo_item_ids_no_duplicates` | validation | self | `product` | constrains: `combo_item_ids` |  |
| `_check_company_id` | validation | self | `product` | constrains: `company_id` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_read` | internal rule | self, records, config | `point_of_sale` | model |  |
| `_check_qty_max` | validation | self | `point_of_sale` | constrains: `qty_max` |  |
| `_check_qty_free` | validation | self | `point_of_sale` | constrains: `qty_free` |  |
| `_check_qty_max_greater_than_qty_free` | validation | self | `point_of_sale` | constrains: `qty_max`, `qty_free` |  |
| `_get_max_quantity` | preparation rule | self, website, sale_order, **kwargs | `website_sale_stock` |  | The max quantity of a combo is the max quantity of its combo item with the highest max quantity. If one of the combo items has no max quantity, then the combo also has no max quantity.  Note: self.ensure_one()  :param website website: The website for which to compute the max quantity. :return: The max quantity of the combo. :rtype: float \| None |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_combo_item_ids_not_empty` | ValidationError | A combo choice must contain at least 1 product. | `product` |
| `_check_combo_item_ids_no_duplicates` | ValidationError | A combo choice can't contain duplicate products. | `product` |
| `_check_qty_max` | ValidationError | The maximum quantity of a combo must be greater or equal to 1. | `point_of_sale` |
| `_check_qty_free` | ValidationError | The free quantity of a combo must be greater or equal to 0. | `point_of_sale` |
| `_check_qty_max_greater_than_qty_free` | ValidationError | The free quantity must be smaller or equal to the maximum quantity. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Product combo multi-company rule | global (all users) | `['\|', ('company_id', '=', False), ('company_id', 'parent_of', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.product_combo_view_form` | field | `product.product_combo_view_form` | `product_id` |  |  | `point_of_sale` |
| `product.product_combo_view_form` | form |  | `name`, `company_id`, `combo_item_ids`, `currency_id`, `product_id`, `lst_price`, `extra_price` |  |  | `product` |
| `product.product_combo_view_tree` | list |  | `currency_id`, `sequence`, `name`, `base_price`, `combo_item_count` |  |  | `product` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.product_combo_action` | Combo Choices | list,form |  |  |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.menu_product_combo` | Combo Choices | `point_of_sale.pos_config_menu_catalog` | `product.product_combo_action` | 15 |  |
| `sale.menu_product_combos` | Combo Choices |  | `product.product_combo_action` | 15 |  |
| `website_sale.menu_product_combos` | Combo Choices |  | `product.product_combo_action` | 6 |  |

Machine-readable definition: `../../../schemas/data/entities/product.combo.json`; views: `../../../schemas/interfaces/views/product.combo.json`.
