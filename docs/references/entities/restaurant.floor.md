# Restaurant Floor (`restaurant.floor`)

**Transport name:** `restaurant.floor`  
**Storage name:** `restaurant_floor`  
**Kind:** persistent entity (one table)  
**Defined by package:** `pos_restaurant`  
**Extended by packages:** `pos_self_order`

Description: Restaurant Floor

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Floor Name | single line text |  | required |
| `pos_config_ids` | Point of Sales | many to many | `pos.config` | restricted by domain `[('module_pos_restaurant', '=', True)]` |
| `background_image` | Background Image | binary |  |  |
| `background_color` | Background Color | single line text |  | Help: The background color of the floor in a html-compatible format |
| `table_ids` | Tables | one to many | `restaurant.table` | inverse field `floor_id` |
| `sequence` | Sequence | integer |  | default `1` |
| `active` | Active | boolean |  | default `True` |
| `floor_background_image` | Floor Background Image | image |  |  |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_restaurant` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_restaurant` | model |  |
| `_unlink_except_active_pos_session` | internal rule | self | `pos_restaurant` | ondelete |  |
| `write` | lifecycle override | self, vals | `pos_restaurant` |  |  |
| `rename_floor` | operation | self, new_name | `pos_restaurant` |  |  |
| `sync_from_ui` | operation | self, name, background_color, config_id | `pos_restaurant` | model |  |
| `deactivate_floor` | operation | self, session_id | `pos_restaurant` |  |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_active_pos_session` | UserError | error_msg | `pos_restaurant` |
| `write` | UserError | Please close and validate the following open PoS Session before modifying this floor. Open session: %(session_names)s | `pos_restaurant` |
| `deactivate_floor` | UserError | You cannot delete a floor when orders are still in draft for this floor. | `pos_restaurant` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_restaurant` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_restaurant` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `pos_restaurant.view_restaurant_floor_form` | form |  | `active`, `floor_background_image`, `name`, `pos_config_ids`, `background_color`, `table_ids`, `table_number`, `seats`, `shape`, `height`, `width`, `color`, `active` |  |  | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_tree` | list |  | `sequence`, `name`, `pos_config_ids` |  |  | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_search` | search |  | `name` |  | `Archived` | `pos_restaurant` |
| `pos_restaurant.view_restaurant_floor_kanban` | kanban |  | `name`, `pos_config_ids` |  |  | `pos_restaurant` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `pos_restaurant.action_restaurant_floor_form` | Floor Plans | list,kanban,form |  |  |  | `pos_restaurant` |

Machine-readable definition: `../../../schemas/data/entities/restaurant.floor.json`; views: `../../../schemas/interfaces/views/restaurant.floor.json`.
