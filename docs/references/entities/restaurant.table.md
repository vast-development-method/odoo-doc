# Restaurant Table (`restaurant.table`)

**Transport name:** `restaurant.table`  
**Storage name:** `restaurant_table`  
**Kind:** persistent entity (one table)  
**Defined by package:** `pos_restaurant`  
**Extended by packages:** `pos_self_order`

Description: Restaurant Table

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `floor_id` | Floor | many to one | `restaurant.floor` | indexed (btree_not_null) |
| `table_number` | Table Number | integer |  | required; default ; Help: The number of the table as displayed on the floor plan |
| `shape` | Shape | selection |  | required; default `square` |
| `position_h` | Horizontal Position | float |  | default `10`; Help: The table's horizontal position from the left side to the table's center, in pixels |
| `position_v` | Vertical Position | float |  | default `10`; Help: The table's vertical position from the top to the table's center, in pixels |
| `width` | Width | float |  | default `50`; Help: The table's width in pixels |
| `height` | Height | float |  | default `50`; Help: The table's height in pixels |
| `seats` | Seats | integer |  | default `1`; Help: The default number of customer served at this table. |
| `color` | Color | single line text |  | Help: The table's color, expressed as a valid 'background' CSS property value |
| `parent_id` | Parent Table | many to one | `restaurant.table` | Help: The parent table if this table is part of a group of tables |
| `active` | Active | boolean |  | default `True`; Help: If false, the table is deactivated and will not be available in the point of sale |
| `identifier` | Security Token | single line text |  | required; default computed dynamically (lambda self: self._get_identifier()); not copied on duplication |

## Selection values

### `shape` (Shape)

| Value | Label |
|---|---|
| `square` | Square |
| `round` | Round |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `pos_restaurant` | depends: `table_number`, `floor_id` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_restaurant` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_restaurant` | model |  |
| `are_orders_still_in_draft` | operation | self | `pos_restaurant` |  |  |
| `_unlink_except_active_pos_session` | internal rule | self | `pos_restaurant` | ondelete |  |
| `set_parent_id` | operation | self, parent_id, config_id | `pos_restaurant` |  |  |
| `_get_identifier` | preparation rule |  | `pos_self_order` |  |  |
| `_update_identifier` | internal rule | self | `pos_self_order` | model |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `are_orders_still_in_draft` | UserError | You cannot delete a table when orders are still in draft for this table. | `pos_restaurant` |
| `_unlink_except_active_pos_session` | UserError | error_msg | `pos_restaurant` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_restaurant` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_restaurant` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `pos_restaurant.view_restaurant_table_form` | form |  | `table_number`, `seats`, `shape`, `color`, `position_h`, `position_v`, `width`, `height` |  |  | `pos_restaurant` |
| `pos_self_order.pos_self_order_table_form_view` | xpath | `pos_restaurant.view_restaurant_table_form` | `identifier` |  |  | `pos_self_order` |

Machine-readable definition: `../../../schemas/data/entities/restaurant.table.json`; views: `../../../schemas/interfaces/views/restaurant.table.json`.
