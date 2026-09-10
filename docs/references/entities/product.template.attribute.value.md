# Product Template Attribute Value (`product.template.attribute.value`)

**Transport name:** `product.template.attribute.value`  
**Storage name:** `product_template_attribute_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale`, `product_matrix`

Description: Product Template Attribute Value

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `attribute_line_id, product_attribute_value_id, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `ptav_active` | Active | boolean |  | default `True` |
| `name` | Value | single line text |  | related through path `product_attribute_value_id.name` |
| `product_attribute_value_id` | Attribute Value | many to one | `product.attribute.value` | required; indexed; on delete of the target: cascade |
| `attribute_line_id` | Attribute Line | many to one | `product.template.attribute.line` | required; indexed; on delete of the target: cascade |
| `price_extra` | Extra Price | float |  | default ; Help: Extra price for the variant with this attribute value on sale price. eg. 200 price extra, 1000 + 200 = 1200. |
| `currency_id` | Currency | many to one |  | related through path `attribute_line_id.product_tmpl_id.currency_id` |
| `exclude_for` | Exclude for | one to many | `product.template.attribute.exclusion` | inverse field `product_template_attribute_value_id`; Help: Make this attribute value not compatible with other values of the product or some attribute values of optional and accessory products. |
| `product_tmpl_id` | Product Tmpl | many to one |  | related through path `attribute_line_id.product_tmpl_id` and stored; indexed |
| `attribute_id` | Attribute | many to one |  | related through path `attribute_line_id.attribute_id` and stored; indexed |
| `ptav_product_variant_ids` | Related Variants | many to many | `product.product` | read only; association table `product_variant_combination` |
| `html_color` | hypertext markup language Color Index | single line text |  | related through path `product_attribute_value_id.html_color` |
| `is_custom` | Is Custom | boolean |  | related through path `product_attribute_value_id.is_custom` |
| `display_type` | Display Type | selection |  | related through path `product_attribute_value_id.display_type` |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `image` | Image | image |  | related through path `product_attribute_value_id.image` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_attribute_value_unique` | Constraint | `unique(attribute_line_id, product_attribute_value_id)` | Each value should be defined only once per attribute per product. | `product` |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `product` |  |  |
| `_check_valid_values` | validation | self | `product` | constrains: `attribute_line_id`, `product_attribute_value_id` |  |
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `product` |  |  |
| `unlink` | lifecycle override | self | `product` |  | Override to: - Clean up the variants that use any of the values in self:     - Remove the value from the variant if the value belonged to an         attribute line with only one value.     - Unlink or archive all related variants. - Archive the value if unlink is not possible.  Archiving is typically needed when the value is referenced elsewhere (on a variant that can't be deleted, on a sales order line, ...). |
| `_compute_display_name` | computation | self | `product` | depends: `attribute_id` | Override because in general the name of the value is confusing if it is displayed without the name of the corresponding attribute. Eg. on exclusion rules form |
| `_only_active` | internal rule | self | `product` |  |  |
| `_without_no_variant_attributes` | internal rule | self | `product` |  |  |
| `_ids2str` | internal rule | self | `product` |  |  |
| `_get_combination_name` | preparation rule | self | `product` |  | Exclude values from single value lines or from no_variant attributes. |
| `_filter_single_value_lines` | internal rule | self | `product` |  | Return `self` with values from single value lines filtered out depending on the active state of all the values in `self`.  If any value in `self` is archived, archived values are also taken into account when checking for single values. This allows to display the correct name for archived variants.  If all values in `self` are active, only active values are taken into account when checking for single values. This allows to display the correct name for active combinations. |
| `_is_from_single_value_line` | internal rule | self, only_active | `product` |  | Return whether `self` is from a single value line, counting also archived values if `only_active` is False. |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_get_extra_price` | preparation rule | self, combination_info | `website_sale` |  |  |
| `_grid_header_cell` | internal rule | self, fro_currency, to_currency, company, display_extra | `product_matrix` |  | Generate a header matrix cell for 1 or multiple attributes.  :param res.currency fro_currency: :param res.currency to_currency: :param res.company company: :param bool display_extra: whether extra prices should be displayed in the cell     True by default, used to avoid showing extra prices on purchases. :returns: cell with name (and price if any price_extra is defined on self) :rtype: dict |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_valid_values` | ValidationError | The value %(value)s is not defined for the attribute %(attribute)s on the product %(product)s. | `product` |
| `create` | UserError | You cannot update related variants from the values. Please update related values from the variants. | `product` |
| `write` | UserError | You cannot update related variants from the values. Please update related values from the variants. | `product` |
| `write` | UserError | You cannot change the value of the value %(value)s set on product %(product)s. | `product` |
| `write` | UserError | You cannot change the product of the value %(value)s set on product %(product)s. | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_template_attribute_value_view_tree` | list |  | `product_tmpl_id`, `attribute_id`, `name`, `display_type`, `html_color`, `image`, `ptav_active`, `price_extra`, `currency_id` |  |  | `product` |
| `product.product_template_attribute_value_view_form` | form |  | `ptav_active`, `name`, `display_type`, `html_color`, `image`, `price_extra`, `currency_id`, `exclude_for`, `product_tmpl_id`, `value_ids` |  |  | `product` |
| `product.product_template_attribute_value_view_search` | search |  | `name` |  | `Active`, `Inactive` | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.template.attribute.value.json`; views: `../../../schemas/interfaces/views/product.template.attribute.value.json`.
