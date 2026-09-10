# Product Template Attribute Line (`product.template.attribute.line`)

**Transport name:** `product.template.attribute.line`  
**Storage name:** `product_template_attribute_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale`, `website_sale_comparison`

Description: Product Template Attribute Line

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, attribute_id, id`
- Display name field: `attribute_id`
- Display name search fields: `["attribute_id", "value_ids"]`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `product_tmpl_id` | Product Template | many to one | `product.template` | required; indexed; on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `10` |
| `attribute_id` | Attribute | many to one | `product.attribute` | required; indexed; on delete of the target: restrict |
| `value_ids` | Values | many to many | `product.attribute.value` | on delete of the target: restrict; restricted by domain `[('attribute_id', '=', attribute_id)]`; association table `product_attribute_value_product_template_attribute_line_rel` |
| `value_count` | Value Count | integer |  | computed by rule `_compute_value_count` and stored |
| `product_template_value_ids` | Product Attribute Values | one to many | `product.template.attribute.value` | inverse field `attribute_line_id` |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_value_count` | computation | self | `product` | depends: `value_ids` |  |
| `_onchange_attribute_id` | on change | self | `product` | onchange: `attribute_id` |  |
| `_check_valid_values` | validation | self | `product` | constrains: `active`, `value_ids`, `attribute_id` |  |
| `create` | lifecycle override | self, vals_list | `product` | model_create_multi | Override to: - Activate archived lines having the same configuration (if they exist)     instead of creating new lines. - Set up related values and related variants.  Reactivating existing lines allows to re-use existing variants when possible, keeping their configuration and avoiding duplication. |
| `write` | lifecycle override | self, vals | `product` |  | Override to: - Add constraints to prevent doing changes that are not supported such     as modifying the template or the attribute of existing lines. - Clean up related values and related variants when archiving or when     updating `value_ids`. |
| `unlink` | lifecycle override | self | `product` |  | Override to: - Archive the line if unlink is not possible. - Clean up related values and related variants.  Archiving is typically needed when the line has values that can't be deleted because they are referenced elsewhere (on a variant that can't be deleted, on a sales order line, ...). |
| `_update_product_template_attribute_values` | internal rule | self | `product` |  | Create or unlink `product.template.attribute.value` for each line in `self` based on `value_ids`.  The goal is to delete all values that are not in `value_ids`, to activate those in `value_ids` that are currently archived, and to create those in `value_ids` that didn't exist.  This is a trick for the form view and for performance in general, because we don't want to generate in advance all possible values for all templates, but only those that will be selected. |
| `_without_no_variant_attributes` | internal rule | self | `product` |  |  |
| `_is_configurable` | internal rule | self | `product` |  |  |
| `action_open_attribute_values` | user action | self | `product` | readonly |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_prepare_single_value_for_display` | preparation rule | self | `website_sale` |  | On the product page group together the attribute lines that concern the same attribute and that have only one value each.  Indeed those are considered informative values, they do not generate choice for the user, so they are displayed below the configurator.  The returned attributes are ordered as they appear in `self`, so based on the order of the attribute lines. |
| `_prepare_single_value_including_multi_type_for_display` | preparation rule | self | `website_sale` |  | On the product page group together the attribute lines that concern the same attribute and that have only one value each.  Unlike `_prepare_single_value_for_display`, this method also includes the attribute lines with a display type 'multi' |
| `_prepare_categories_for_display` | preparation rule | self | `website_sale_comparison` |  | On the product page group together the attribute lines that concern attributes that are in the same category.  The returned categories are ordered following their default order.  :return: OrderedDict [{     product.attribute.category: [product.template.attribute.line] }] |
| `_prepare_categories_for_display_in_specs_table` | preparation rule | self | `website_sale_comparison` |  | Prepare attribute categories for display in a specs table.  Filters out attribute lines that have a single value and whose value is marked as custom, then call _prepare_categories_for_display to group the remaining attribute lines by category.  :return: OrderedDict [{ product.attribute.category: [product.template.attribute.line] }] |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_valid_values` | ValidationError | The attribute %(attribute)s must have at least one value for the product %(product)s. | `product` |
| `_check_valid_values` | ValidationError | On the product %(product)s you cannot associate the value %(value)s with the attribute %(attribute)s because they do not match. | `product` |
| `write` | UserError | You cannot move the attribute %(attribute)s from the product %(product_src)s to the product %(product_dest)s. | `product` |
| `write` | UserError | On the product %(product)s you cannot transform the attribute %(attribute_src)s into the attribute %(attribute_dest)s. | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `product` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_template_attribute_line_form` | form |  | `attribute_id`, `value_ids`, `name`, `html_color`, `name` |  |  | `product` |
| `product.product_template_attribute_line_view_tree` | list |  | `product_tmpl_id`, `attribute_id`, `value_ids` |  |  | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.template.attribute.line.json`; views: `../../../schemas/interfaces/views/product.template.attribute.line.json`.
