# Attribute Value (`product.attribute.value`)

**Transport name:** `product.attribute.value`  
**Storage name:** `product_attribute_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`

Description: Attribute Value

## Identity and behavior

- Default ordering: `attribute_id, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Value | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | indexed; Help: Determine the display order |
| `attribute_id` | Attribute | many to one | `product.attribute` | required; indexed; on delete of the target: cascade; Help: The attribute cannot be changed once the value is used on at least one product. |
| `pav_attribute_line_ids` | Lines | many to many | `product.template.attribute.line` | not copied on duplication; association table `product_attribute_value_product_template_attribute_line_rel` |
| `default_extra_price` | Default Extra Price | float |  |  |
| `is_custom` | Free text | boolean |  | Help: Allow customers to set their own value |
| `html_color` | Color | single line text |  | Help: Here you can set a specific HTML color index (e.g. #ff0000) to display the color if the attribute type is 'Color'. |
| `display_type` | Display Type | selection |  | related through path `attribute_id.display_type` |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color) |
| `image` | Image | image |  | Help: You can upload an image that will be used as the color of the attribute value. |
| `active` | Active | boolean |  | default `True` |
| `is_used_on_products` | Used on Products | boolean |  | computed by rule `_compute_is_used_on_products` (not stored) |
| `default_extra_price_changed` | Default Extra Price Changed | boolean |  | computed by rule `_compute_default_extra_price_changed` (not stored) |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `product` |  |  |
| `_compute_display_name` | computation | self | `product` | depends: `attribute_id`; depends_context: `show_attribute` | Override because in general the name of the value is confusing if it is displayed without the name of the corresponding attribute. Eg. on product list & kanban views, on BOM form view  However during variant set up (on the product template form) the name of the attribute is already on each line so there is no need to repeat it on every value. |
| `_compute_is_used_on_products` | computation | self | `product` | depends: `pav_attribute_line_ids` |  |
| `_compute_default_extra_price_changed` | computation | self | `product` | depends: `default_extra_price` |  |
| `write` | lifecycle override | self, vals | `product` |  |  |
| `check_is_used_on_products` | operation | self | `product` |  |  |
| `_unlink_except_used_on_product` | internal rule | self | `product` | ondelete |  |
| `unlink` | lifecycle override | self | `product` |  |  |
| `_without_no_variant_attributes` | internal rule | self | `product` |  |  |
| `action_add_to_products` | user action | self | `product` | readonly |  |
| `action_update_prices` | user action | self | `product` | readonly |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot change the attribute of the value %(value)s because it is used on the following products: %(products)s | `product` |
| `_unlink_except_used_on_product` | UserError | is_used_on_products | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_attribute_value_list` | list |  | `name`, `default_extra_price` |  |  | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.attribute.value.json`; views: `../../../schemas/interfaces/views/product.attribute.value.json`.
