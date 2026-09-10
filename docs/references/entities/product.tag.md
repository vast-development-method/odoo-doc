# Product Tag (`product.tag`)

**Transport name:** `product.tag`  
**Storage name:** `product_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`, `website_sale`, `pos_self_order`

Description: Product Tag

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`, `website.multi.mixin`
- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `color` | Color | single line text |  | default `#3C3C3C` |
| `product_template_ids` | Product Templates | many to many | `product.template` | default computed dynamically (_get_default_template_id); association table `product_tag_product_template_rel` |
| `product_product_ids` | Product Variants | many to many | `product.product` | default computed dynamically (_get_default_variant_id); restricted by domain `[('attribute_line_ids', '!=', False), ('product_tmpl_id', 'not in', product_template_ids)]`; association table `product_tag_product_product_rel` |
| `product_ids` | All Product Variants using this Tag | many to many | `product.product` | computed by rule `_compute_product_ids` (not stored); searchable through a search rule |
| `visible_to_customers` | Visible to customers | boolean |  | default `True`; Help: Whether the tag is displayed to customers. |
| `image` | Image | image |  |  |
| `pos_description` | Description | rich text |  | translatable |
| `has_image` | Has Image | boolean |  | computed by rule `_compute_has_image` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `product` |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_template_id` | preparation rule | self | `product` |  |  |
| `_get_default_variant_id` | preparation rule | self | `product` |  |  |
| `_compute_product_ids` | computation | self | `product` | depends: `product_template_ids`, `product_product_ids` |  |
| `copy_data` | lifecycle override | self, default | `product` |  |  |
| `_search_product_ids` | search rule | self, operator, operand | `product` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_compute_has_image` | computation | self | `point_of_sale` | depends: `has_image` |  |
| `write` | lifecycle override | self, vals | `point_of_sale` |  |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `pos_self_order` |  |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |

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
| `point_of_sale.product_tag_form_view_inherit_point_of_sale` | xpath | `product.product_tag_form_view` | `pos_description` |  |  | `point_of_sale` |
| `product.product_tag_form_view` | form |  | `image`, `name`, `visible_to_customers`, `color` |  |  | `product` |
| `product.product_tag_tree_view` | list |  | `sequence`, `name`, `visible_to_customers`, `color`, `image`, `product_template_ids`, `product_product_ids` |  |  | `product` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.product_tag_action` | Product Tags | list,form |  | `{'create': True}` |  | `product` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.pos_menu_products_tag_action` |  | `point_of_sale.pos_menu_products_configuration` | `product.product_tag_action` | 3 |  |
| `sale.menu_product_tags` |  |  | `product.product_tag_action` | 30 |  |
| `website_sale.product_catalog_product_tags` | Product Tags |  | `product.product_tag_action` |  |  |

Machine-readable definition: `../../../schemas/data/entities/product.tag.json`; views: `../../../schemas/interfaces/views/product.tag.json`.
