# Pricelist Rule (`product.pricelist.item`)

**Transport name:** `product.pricelist.item`  
**Storage name:** `product_pricelist_item`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `sale`, `point_of_sale`, `website_sale`, `website_event_sale`

Description: Pricelist Rule

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `applied_on, min_quantity desc, categ_id desc, id desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (28)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pricelist_id` | Pricelist | many to one | `product.pricelist` | default computed dynamically (_default_pricelist_id); indexed; on delete of the target: cascade |
| `is_pricelist_required` | Is Pricelist Required | boolean |  | computed by rule `_compute_is_pricelist_required` (not stored) |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored |
| `date_start` | Start Date | date and time |  | Help: Starting datetime for the pricelist item validation The displayed value depends on the timezone set in your preferences. |
| `date_end` | End Date | date and time |  | Help: Ending datetime for the pricelist item validation The displayed value depends on the timezone set in your preferences. |
| `min_quantity` | Min. Quantity | float |  | default ; precision `Product Unit`; Help: For the rule to apply, bought/sold quantity must be greater than or equal to the minimum quantity specified in this field. Expressed in the default unit of measure of the product. |
| `applied_on` | Apply On | selection |  | required; default `3_global`; Help: Pricelist Item applicable on selected option |
| `display_applied_on` | Display Applied On | selection |  | required; default `1_product`; Help: Pricelist Item applicable on selected option |
| `categ_id` | Category | many to one | `product.category` | on delete of the target: cascade; Help: Specify a product category if this rule only applies to products belonging to this category or its children categories. Keep empty otherwise. |
| `product_tmpl_id` | Product | many to one | `product.template` | indexed (btree_not_null); on delete of the target: cascade; must belong to the same company; Help: Specify a template if this rule only applies to one product template. Keep empty otherwise. |
| `product_id` | Variant | many to one | `product.product` | indexed (btree_not_null); on delete of the target: cascade; restricted by domain `[('product_tmpl_id', '=', product_tmpl_id)]`; must belong to the same company; Help: Specify a product if this rule only applies to one product. Keep empty otherwise. |
| `product_uom_name` | Product Unit of measure Name | single line text |  | related through path `product_tmpl_id.uom_name` |
| `product_variant_count` | Product Variant Count | integer |  | related through path `product_tmpl_id.product_variant_count` |
| `base` | Based on | selection |  | required; default `list_price`; Help: Base price for computation. Sales Price: The base price will be the Sales Price. Cost Price: The base price will be the cost price. Other Pricelist: Computation of the base price based on another Pricelist. |
| `base_pricelist_id` | Other Pricelist | many to one | `product.pricelist` | must belong to the same company |
| `compute_price` | Compute Price | selection |  | required; default `fixed`; indexed; Help: Use the discount rules and activate the discount settings in order to show discount to customer. |
| `fixed_price` | Fixed Price | float |  |  |
| `percent_price` | Percentage Price | float |  | Help: You can apply a mark-up by setting a negative discount. |
| `price_discount` | Price Discount | float |  | default ; precision `[16, 2]`; Help: You can apply a mark-up by setting a negative discount. |
| `price_round` | Price Rounding | float |  | Help: Sets the price so that it is a multiple of this value. Rounding is applied after the discount and before the surcharge. To have prices that end in 9.99, round off to 10.00 and set an extra at -0.01 |
| `price_surcharge` | Extra Fee | float |  | Help: Specify the fixed amount to add or subtract (if negative) to the amount calculated with the discount. |
| `price_markup` | Markup | float |  | computed by rule `_compute_price_markup` and stored; writable through an inverse rule; precision `[16, 2]`; Help: You can apply a mark-up on the cost |
| `price_min_margin` | Min. Price Margin | float |  | Help: Specify the minimum amount of margin over the base price. |
| `price_max_margin` | Max. Price Margin | float |  | Help: Specify the maximum amount of margin over the base price. |
| `name` | Name | single line text |  | computed by rule `_compute_name` (not stored); Help: Explicit rule name for this pricelist line. |
| `price` | Price | single line text |  | computed by rule `_compute_price_label` (not stored); Help: Explicit rule name for this pricelist line. |
| `rule_tip` | Rule Tip | single line text |  | computed by rule `_compute_rule_tip` (not stored) |

## Selection values

### `applied_on` (Apply On)

| Value | Label |
|---|---|
| `3_global` | All Products |
| `2_product_category` | Product Category |
| `1_product` | Product |
| `0_product_variant` | Product Variant |

### `display_applied_on` (Display Applied On)

| Value | Label |
|---|---|
| `1_product` | Product |
| `2_product_category` | Category |

### `base` (Based on)

| Value | Label |
|---|---|
| `list_price` | Sales Price |
| `standard_price` | Cost |
| `pricelist` | Other Pricelist |

### `compute_price` (Compute Price)

| Value | Label |
|---|---|
| `percentage` | Discount |
| `formula` | Formula |
| `fixed` | Fixed Price |

## Operations (39)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_pricelist_id` | preparation rule | self | `product` |  |  |
| `_compute_is_pricelist_required` | computation | self | `product` |  |  |
| `_compute_company_id` | computation | self | `product` | depends: `pricelist_id.company_id`, `product_tmpl_id` |  |
| `_compute_currency_id` | computation | self | `product` | depends: `pricelist_id.currency_id`, `company_id` |  |
| `_compute_name` | computation | self | `product` | depends: `applied_on`, `categ_id`, `product_tmpl_id`, `product_id` |  |
| `_get_price_label_base_str` | preparation rule | self | `product` |  | This method allows you to extend it to other modules with other options in the base field to return a different text. |
| `_compute_price_label` | computation | self | `product` | depends: `compute_price`, `fixed_price`, `pricelist_id`, `percent_price`, `price_discount`, `price_markup`, `price_surcharge`, `base`, `base_pricelist_id` |  |
| `_compute_price_markup` | computation | self | `product` | depends: `price_discount` |  |
| `_inverse_price_markup` | inverse computation | self | `product` |  |  |
| `_compute_rule_tip` | computation | self | `product` | depends_context: `lang`; depends: `base`, `compute_price`, `price_discount`, `price_markup`, `price_round`, `price_surcharge` |  |
| `_get_integer` | preparation rule | self, percentage | `product` |  |  |
| `_get_displayed_discount` | preparation rule | self, item | `product` |  |  |
| `_check_base_pricelist_id` | validation | self | `product` | constrains: `base_pricelist_id`, `base` |  |
| `_check_pricelist_recursion` | validation | self | `product` | constrains: `base_pricelist_id`, `pricelist_id`, `base` |  |
| `_check_date_range` | validation | self | `product` | constrains: `date_start`, `date_end` |  |
| `_check_margin` | validation | self | `product` | constrains: `price_min_margin`, `price_max_margin` |  |
| `_check_product_consistency` | validation | self | `product` | constrains: `product_id`, `product_tmpl_id`, `categ_id` |  |
| `_onchange_base` | on change | self | `product` | onchange: `base` |  |
| `_onchange_base_pricelist_id` | on change | self | `product` | onchange: `base_pricelist_id` |  |
| `_onchange_compute_price` | on change | self | `product` | onchange: `compute_price` |  |
| `_onchange_display_applied_on` | on change | self | `product` | onchange: `display_applied_on` |  |
| `_onchange_product_id` | on change | self | `product` | onchange: `product_id` |  |
| `_onchange_product_tmpl_id` | on change | self | `product` | onchange: `product_tmpl_id` |  |
| `_onchange_rule_content` | on change | self | `product` | onchange: `product_id`, `product_tmpl_id`, `categ_id` |  |
| `_onchange_price_round` | on change | self | `product` | onchange: `price_round` |  |
| `_onchange_validity_period` | on change | self | `product` | onchange: `date_start`, `date_end` |  |
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `product` |  |  |
| `_is_applicable_for` | internal rule | self, product, qty_in_product_uom | `product` |  | Check whether the current rule is valid for the given product & qty.  Note: self.ensure_one()  :param product: product record (product.product/product.template) :param float qty_in_product_uom: quantity, expressed in product UoM :returns: Whether rules is valid or not :rtype: bool |
| `_compute_price` | computation | self, product, quantity, uom, date, currency, **kwargs | `product` |  | Compute the unit price of a product in the context of a pricelist application.  Note: self and self.ensure_one()  :param product: recordset of product (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param uom: unit of measure (uom.uom record) :param datetime date: date to use for price computation and currency conversions :param currency: currency (for the case where self is empty) :param dict kwargs: unused parameters available for overrides  :returns: price according to pricelist rule or the product price, expressed in the param        |
| `_compute_base_price` | computation | self, product, quantity, uom, date, currency, **kwargs | `product` |  | Compute the base price for a given rule.  :param product: recordset of product (product.product/product.template) :param float quantity: quantity of products requested (in given uom) :param uom: unit of measure (uom.uom record) :param datetime date: date to use for price computation and currency conversions :param currency: currency in which the returned price must be expressed  :returns: base price, expressed in provided pricelist currency :rtype: float |
| `_compute_price_before_discount` | computation | self, *args, **kwargs | `product` |  | Compute the base price of the given rule, considering chained pricelists.  :param product: recordset of product (product.product/product.template) :param float qty: quantity of products requested (in given uom) :param uom: unit of measure (uom.uom record) :param datetime date: date to use for price computation and currency conversions :param currency: currency in which the returned price must be expressed  :returns: base price, expressed in provided pricelist currency :rtype: float |
| `_is_discount_feature_enabled` | internal rule | self | `sale` | model |  |
| `_show_discount` | internal rule | self | `sale` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_server_date_to_domain` | internal rule | self, domain | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_show_discount_on_shop` | internal rule | self | `website_sale` |  | On ecommerce, formula rules are also expected to show discounts.  Only for /shop, /product, and configurators, not on the cart or the checkout. |
| `_onchange_event_sale_warning` | on change | self | `website_event_sale` | onchange: `applied_on`, `product_id`, `product_tmpl_id`, `min_quantity` |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_base_pricelist_id` | ValidationError | A pricelist item with "Other Pricelist" as base must have a base_pricelist_id. | `product` |
| `_check_pricelist_recursion` | ValidationError | Recursive pricelist rules detected: %s | `product` |
| `_check_date_range` | ValidationError | %(item_name)s: end date (%(end_date)s) should be after start date (%(start_date)s) | `product` |
| `_check_margin` | ValidationError | The minimum margin should be lower than the maximum margin. | `product` |
| `_check_product_consistency` | ValidationError | Please specify the category for which this rule should be applied | `product` |
| `_check_product_consistency` | ValidationError | Please specify the product for which this rule should be applied | `product` |
| `_check_product_consistency` | ValidationError | Please specify the product variant for which this rule should be applied | `product` |
| `_onchange_price_round` | ValidationError | The rounding method must be strictly positive. | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| product pricelist item company rule | global (all users) | `['\|', ('company_id', 'parent_of', company_ids), ('company_id', '=', False)]` | True | True | True | True |
| product pricelist item company rule | global (all users) | `['\|', ('company_id', 'in', [False, website.company_id.id]), ('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_pricelist_item_view_search` | search |  | `pricelist_id`, `company_id`, `currency_id` |  | `Product Rule`, `Variant Rule`, `Active`, `Product`, `Variant`, `Pricelist` | `product` |
| `product.product_pricelist_item_tree_view` | list |  | `pricelist_id`, `name`, `price`, `min_quantity`, `date_start`, `date_end`, `company_id` |  |  | `product` |
| `product.product_pricelist_item_tree_view_from_product` | list |  | `pricelist_id`, `product_id`, `company_id`, `categ_id`, `product_tmpl_id`, `fixed_price`, `min_quantity`, `currency_id`, `date_start`, `date_end`, `applied_on`, `company_id` |  |  | `product` |
| `product.product_pricelist_item_form_view` | form |  | `name`, `company_id`, `price`, `applied_on`, `display_applied_on`, `categ_id`, `product_tmpl_id`, `product_variant_count`, `product_id`, `compute_price`, `fixed_price`, `product_uom_name`, `percent_price`, `base_pricelist_id`, `min_quantity`, `date_start`, `date_end`, `base`, `base_pricelist_id`, `price_discount`, `price_markup`, `price_round`, `price_surcharge`, `price_min_margin`, `price_max_margin`, `rule_tip`, `pricelist_id`, `currency_id`, `company_id` |  |  | `product` |
| `product.product_pricelist_item_product_template_form_view` | field | `product.product_pricelist_item_form_view` | `display_applied_on` |  |  | `product` |
| `product.product_pricelist_item_product_product_form_view` | field | `product.product_pricelist_item_product_template_form_view` | `product_id` |  |  | `product` |
| `sale.product_pricelist_item_form` | group | `product.product_pricelist_item_form_view` | `min_quantity`, `date_start`, `date_end` |  |  | `sale` |
| `website_sale.product_pricelist_item_form` | div | `sale.product_pricelist_item_form` |  |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.product_pricelist_item_action` | Price Rules | list,form |  |  |  | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.pricelist.item.json`; views: `../../../schemas/interfaces/views/product.pricelist.item.json`.
