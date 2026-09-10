# Product Attribute (`product.attribute`)

**Transport name:** `product.attribute`  
**Storage name:** `product_attribute`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale`, `website_sale_comparison`

Description: Product Attribute

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Attribute | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; Help: If unchecked, it will allow you to hide the attribute without removing it. |
| `create_variant` | Variant Creation | selection |  | required; default `always`; Help: - Instantly: All possible variants are created as soon as the attribute and its values are added to a product.         - Dynamically: Each variant is created only when its corresponding attributes and values are added to a sales order.         - Never: Variants are never created for the attribute.         Note: this cannot be changed once the attribute is used on a product. |
| `display_type` | Display Type | selection |  | required; default `radio`; Help: The display type used in the Product Configurator. |
| `sequence` | Sequence | integer |  | default `20`; indexed; Help: Determine the display order |
| `value_ids` | Values | one to many | `product.attribute.value` | inverse field `attribute_id` |
| `template_value_ids` | Template Values | one to many | `product.template.attribute.value` | inverse field `attribute_id` |
| `attribute_line_ids` | Lines | one to many | `product.template.attribute.line` | inverse field `attribute_id` |
| `product_tmpl_ids` | Related Products | many to many | `product.template` | computed by rule `_compute_products` and stored |
| `number_related_products` | Number Related Products | integer |  | computed by rule `_compute_number_related_products` (not stored) |
| `visibility` | Visibility | selection |  | default `visible` |
| `preview_variants` | On Product Cards | selection |  | default `hidden`; Help: Instantly created variants are available for selection from your /shop page. |
| `is_thumbnail_visible` | Show Thumbnails | boolean |  | Help: Use product variant images instead of the attribute values displays. |
| `category_id` | eCommerce Category | many to one | `product.attribute.category` | indexed; Help: Set a category to regroup similar attributes under the same section in the Comparison page of eCommerce. |

## Selection values

### `create_variant` (Variant Creation)

| Value | Label |
|---|---|
| `always` | Instantly |
| `dynamic` | Dynamically |
| `no_variant` | Never |

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `radio` | Radio |
| `pills` | Pills |
| `select` | Select |
| `color` | Color |
| `multi` | Multi-checkbox |
| `image` | Image |

### `visibility` (Visibility)

| Value | Label |
|---|---|
| `visible` | Visible |
| `hidden` | Hidden |

### `preview_variants` (On Product Cards)

| Value | Label |
|---|---|
| `visible` | Visible |
| `hidden` | Hidden |
| `hover` | Hover |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_multi_checkbox_no_variant` | Constraint | `CHECK(display_type != 'multi' OR create_variant = 'no_variant')` | Multi-checkbox display type is not compatible with the creation of variants | `product` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_number_related_products` | computation | self | `product` | depends: `product_tmpl_ids` |  |
| `_compute_products` | computation | self | `product` | depends: `attribute_line_ids.active`, `attribute_line_ids.product_tmpl_id` |  |
| `_onchange_display_type` | on change | self | `product` | onchange: `display_type` |  |
| `write` | lifecycle override | self, vals | `product` |  | Override to make sure attribute type can't be changed if it's used on a product template.  This is important to prevent because changing the type would make existing combinations invalid without recomputing them, and recomputing them might take too long and we don't want to change products without the user knowing about it. |
| `_unlink_except_used_on_product` | internal rule | self | `product` | ondelete |  |
| `action_archive` | lifecycle override | self | `product` |  |  |
| `action_open_product_template_attribute_lines` | user action | self | `product` | readonly |  |
| `_without_no_variant_attributes` | internal rule | self | `product` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_onchange_disable_preview_variants` | on change | self | `website_sale` | onchange: `create_variant`, `display_type` | The option to preview variants is only available for instantly created single variants. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot change the Variants Creation Mode of the attribute %(attribute)s because it is used on the following products: %(products)s | `product` |
| `_unlink_except_used_on_product` | UserError | You cannot delete the attribute %(attribute)s because it is used on the following products: %(products)s | `product` |
| `action_archive` | UserError | You cannot archive this attribute as there are still products linked to it | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.attribute_tree_view` | list |  | `sequence`, `name`, `display_type`, `create_variant` |  |  | `product` |
| `product.product_attribute_view_form` | form |  | `number_related_products`, `number_related_products`, `name`, `display_type`, `create_variant`, `value_ids`, `sequence`, `name`, `display_type`, `is_custom`, `html_color`, `image`, `default_extra_price` | `action_open_product_template_attribute_lines`, `Add to products`, `Update extra prices` |  | `product` |
| `product.product_attribute_search` | search |  | `name` |  | `Inactive` | `product` |
| `website_sale.product_attribute_view_form` | group | `product.product_attribute_view_form` | `visibility`, `preview_variants`, `is_thumbnail_visible` |  |  | `website_sale` |
| `website_sale.attribute_tree_view` | field | `product.attribute_tree_view` | `create_variant`, `visibility` |  |  | `website_sale` |
| `website_sale_comparison.product_attribute_tree_view_inherit` | field | `product.attribute_tree_view` | `name`, `category_id` |  |  | `website_sale_comparison` |
| `website_sale_comparison.product_attribute_view_form` | group | `website_sale.product_attribute_view_form` | `category_id` |  |  | `website_sale_comparison` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.attribute_action` | Attributes | list,form |  |  |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.pos_menu_products_attribute_action` |  | `point_of_sale.pos_menu_products_configuration` | `product.attribute_action` | 2 | `product.group_product_variant` |
| `purchase.menu_product_attribute_action` | Attributes | `purchase.menu_product_in_config_purchase` | `product.attribute_action` | 1 | `product.group_product_variant` |
| `sale.menu_product_attribute_action` |  |  | `product.attribute_action` | 10 | `product.group_product_variant` |
| `stock.menu_attribute_action` |  | `stock.menu_product_in_config_stock` | `product.attribute_action` | 4 | `product.group_product_variant` |
| `website_sale.menu_product_attribute_action` |  |  | `product.attribute_action` | 5 | `product.group_product_variant` |

Machine-readable definition: `../../../schemas/data/entities/product.attribute.json`; views: `../../../schemas/interfaces/views/product.attribute.json`.
