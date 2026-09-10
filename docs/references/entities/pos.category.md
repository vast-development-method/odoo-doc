# Point of Sale Category (`pos.category`)

**Transport name:** `pos.category`  
**Storage name:** `pos_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_self_order`

Description: Point of Sale Category

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Category Name | single line text |  | required; translatable |
| `parent_id` | Parent Category | many to one | `pos.category` | indexed |
| `child_ids` | Children Categories | one to many | `pos.category` | inverse field `parent_id` |
| `sequence` | Sequence | integer |  | Help: Gives the sequence order when displaying a list of product categories. |
| `image_512` | Image | image |  |  |
| `image_128` | Image 128 | image |  | related through path `image_512` and stored |
| `color` | Color | integer |  | default computed dynamically (get_default_color) |
| `hour_until` | Availability Until | float |  | default `24.0`; Help: The product will be available until this hour for online order and self order. |
| `hour_after` | Availability After | float |  | default ; Help: The product will be available after this hour for online order and self order. |
| `has_image` | Has Image | boolean |  | computed by rule `_compute_has_image` (not stored) |
| `pos_config_ids` | Linked PoS Configurations | many to many | `pos.config` |  |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_category_recursion` | validation | self | `point_of_sale` | constrains: `parent_id` |  |
| `get_default_color` | operation | self | `point_of_sale` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_get_hierarchy` | preparation rule | self | `point_of_sale` |  | Returns a list representing the hierarchy of the categories. |
| `_compute_display_name` | computation | self | `point_of_sale` | depends: `parent_id` |  |
| `_unlink_except_session_open` | internal rule | self | `point_of_sale` | ondelete |  |
| `_compute_has_image` | computation | self | `point_of_sale` | depends: `has_image` |  |
| `_get_descendants` | preparation rule | self | `point_of_sale` |  |  |
| `_check_hour` | validation | self | `point_of_sale` | constrains: `hour_until`, `hour_after` |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `pos_self_order` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_category_recursion` | ValidationError | Error! You cannot create recursive categories. | `point_of_sale` |
| `_unlink_except_session_open` | UserError | You cannot delete a point of sale category while a session is still opened. | `point_of_sale` |
| `_check_hour` | ValidationError | The Availability Until must be set between 00:00 and 24:00 | `point_of_sale` |
| `_check_hour` | ValidationError | The Availability After must be set between 00:00 and 24:00 | `point_of_sale` |
| `_check_hour` | ValidationError | The Availability Until must be greater than Availability After. | `point_of_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `base.group_user` | no | yes | no | no | `point_of_sale` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.product_pos_category_form_view` | form |  | `image_512`, `name`, `parent_id`, `color`, `hour_after`, `hour_until` |  |  | `point_of_sale` |
| `point_of_sale.product_pos_category_tree_view` | list |  | `sequence`, `display_name`, `parent_id`, `color` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_category_kanban` | kanban |  | `image_128`, `name` |  |  | `point_of_sale` |
| `pos_self_order.pos_self_order_product_pos_category_form_view` | xpath | `point_of_sale.product_pos_category_form_view` |  |  |  | `pos_self_order` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.product_pos_category_action` | PoS Product Categories | list,kanban,form |  |  |  | `point_of_sale` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `point_of_sale.menu_products_pos_category` |  | `point_of_sale.pos_menu_products_configuration` | `point_of_sale.product_pos_category_action` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/pos.category.json`; views: `../../../schemas/interfaces/views/pos.category.json`.
