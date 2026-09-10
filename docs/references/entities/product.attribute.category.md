# Product Attribute Category (`product.attribute.category`)

**Transport name:** `product.attribute.category`  
**Storage name:** `product_attribute_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale_comparison`

Description: Product Attribute Category

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Category Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10`; indexed |
| `attribute_ids` | Related Attributes | one to many | `product.attribute` | restricted by domain `[('category_id', '=', False)]`; inverse field `category_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_sale_comparison` |
| `base.group_portal` | no | yes | no | no | `website_sale_comparison` |
| `base.group_user` | no | yes | no | no | `website_sale_comparison` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale_comparison` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale_comparison.product_attribute_category_tree_view` | list |  | `sequence`, `name`, `attribute_ids` |  |  | `website_sale_comparison` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_sale_comparison.product_attribute_category_action` | Attribute Categories | list |  |  |  | `website_sale_comparison` |

Machine-readable definition: `../../../schemas/data/entities/product.attribute.category.json`; views: `../../../schemas/interfaces/views/product.attribute.category.json`.
