# Product ribbon (`product.ribbon`)

**Transport name:** `product.ribbon`  
**Storage name:** `product_ribbon`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_sale`  
**Extended by packages:** `website_sale_stock`

Description: Product ribbon

## Identity and behavior

- Default ordering: `sequence ASC, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Ribbon Name | single line text |  | required; translatable; maximum length 20 |
| `sequence` | Sequence | integer |  | default `10` |
| `bg_color` | Background Color | single line text |  | required; default `#000000` |
| `text_color` | Text Color | single line text |  | required; default `#FFFFFF` |
| `position` | Position | selection |  | required; default `left` |
| `style` | Style | selection |  | required; default `ribbon`; Help: Defines the display style: - Ribbon: Shows a ribbon banner on the product image. - Badge: Shows a small badge label on the product image. |
| `assign` | Assign | selection |  | required; default `manual`; on delete of the target: {"out_of_stock": "cascade"}; Help: Defines how this ribbon is assigned to products: - Manually: You assign the ribbon manually to products. - Sale: Applied when the product is visibly on sale. - New: Applied based on the New period you will define. - Out Of Stock: Applied when the product is out of stock.; extended by packages `website_sale_stock` |
| `new_period` | New Period | integer |  | default `30` |

## Selection values

### `position` (Position)

| Value | Label |
|---|---|
| `left` | Left |
| `right` | Right |

### `style` (Style)

| Value | Label |
|---|---|
| `ribbon` | Ribbon |
| `tag` | Badge |

### `assign` (Assign)

| Value | Label |
|---|---|
| `manual` | Manually |
| `sale` | On Sale |
| `new` | When New |
| `out_of_stock` | when out of stock |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_assign` | validation | self | `website_sale` | constrains: `assign` | Ensure only one ribbon exists per automatic assign type. This prevents duplicates, since automatic assignment logic always uses the first ribbon with a given assign value. |
| `_get_css_classes` | preparation rule | self | `website_sale` |  | Return the CSS classes for this ribbon based on style and position. rtype: str |
| `_is_applicable_for` | internal rule | self, product, price_data | `website_sale_stock`, `website_sale` |  | Return whether the product matches the criteria of the ribbon automatic assignment.  :param product.product product: the displayed product :param dict price_data: price information for the given product     (sales price for shop page, combination information for product page)  :return: Whether the ribbon matches the given product and price. :rtype: bool |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_assign` | ValidationError | Only one ribbon with the assign %s is allowed. | `website_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |
| `base.group_user` | no | yes | no | no | `website_sale` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_sale.product_ribbon_form_view` | form |  | `name`, `assign`, `new_period`, `position`, `style`, `text_color`, `bg_color` |  |  | `website_sale` |
| `website_sale.product_ribbon_view_tree` | list |  | `sequence`, `name`, `position`, `text_color`, `bg_color` |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_sale.product_ribbon_action` | Product Ribbons | list,form |  | `{'create': True}` |  | `website_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `website_sale.product_catalog_product_ribbons` | Product Ribbons |  | `website_sale.product_ribbon_action` |  |  |

Machine-readable definition: `../../../schemas/data/entities/product.ribbon.json`; views: `../../../schemas/interfaces/views/product.ribbon.json`.
