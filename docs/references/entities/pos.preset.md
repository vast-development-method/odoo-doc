# Easily load a set of configuration options (`pos.preset`)

**Transport name:** `pos.preset`  
**Storage name:** `pos_preset`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_restaurant`, `pos_self_order`

Description: Easily load a set of configuration options

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Label | single line text |  | required; translatable |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` |  |
| `fiscal_position_id` | Fiscal Position | many to one | `account.fiscal.position` |  |
| `identification` | Identification | selection |  | required; default `none` |
| `is_return` | Return mode | boolean |  | default ; Help: All quantity in the cart will be in negative. Ideal for return managment. |
| `color` | Color | integer |  | default  |
| `image_512` | Image | image |  |  |
| `image_128` | Image 128 | image |  | related through path `image_512` and stored |
| `has_image` | Has Image | boolean |  | computed by rule `_compute_has_image` (not stored) |
| `count_linked_orders` | Count Linked Orders | integer |  | computed by rule `_compute_count_linked_orders` (not stored) |
| `count_linked_config` | Count Linked Config | integer |  | computed by rule `_compute_count_linked_config` (not stored) |
| `use_timing` | Manage orders by time | boolean |  | default  |
| `resource_calendar_id` | Resource | many to one | `resource.calendar` |  |
| `attendance_ids` | Attendances | one to many |  | related through path `resource_calendar_id.attendance_ids` |
| `slots_per_interval` | Capacity | integer |  | default `5` |
| `interval_time` | Interval time (in min) | integer |  | default `20` |
| `use_guest` | Guest | boolean |  | default ; Help: Force guest selection when clicking on order button in PoS restaurant |
| `available_in_self` | Available in self | boolean |  | default  |
| `service_at` | Service at | selection |  | required; default `counter` |
| `mail_template_id` | Email Confirmation | many to one | `mail.template` | restricted by domain `[('model', '=', 'pos.order')]` |

## Selection values

### `identification` (Identification)

| Value | Label |
|---|---|
| `none` | Not required |
| `address` | Address |
| `name` | Name |

### `service_at` (Service at)

| Value | Label |
|---|---|
| `counter` | Pickup zone |
| `table` | Table |
| `delivery` | Delivery |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_slots` | validation | self | `point_of_sale` | constrains: `attendance_ids` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale`, `pos_restaurant`, `pos_self_order` | model |  |
| `_compute_count_linked_orders` | computation | self | `point_of_sale` |  |  |
| `_compute_count_linked_config` | computation | self | `point_of_sale` |  |  |
| `_compute_has_image` | computation | self | `point_of_sale` | depends: `has_image` |  |
| `get_available_slots` | operation | self | `point_of_sale` |  |  |
| `_compute_slots_usage` | computation | self | `point_of_sale` |  |  |
| `action_open_linked_orders` | user action | self | `point_of_sale` |  |  |
| `action_open_linked_config` | user action | self | `point_of_sale` |  |  |
| `_unlink_except_used_preset` | internal rule | self | `point_of_sale` | ondelete |  |
| `_unlink_except_master_presets` | internal rule | self | `pos_restaurant` | ondelete |  |
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |
| `_can_return_content` | internal rule | self, field_name, access_token | `pos_self_order` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_slots` | ValidationError | The start time must be before the end time. | `point_of_sale` |
| `_unlink_except_used_preset` | UserError | You cannot delete a preset that is linked to a POS configuration. | `point_of_sale` |
| `_unlink_except_master_presets` | UserError | You cannot delete the master preset(s). | `pos_restaurant` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | no | yes | no | no | `point_of_sale` |
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_preset_form` | form |  | `count_linked_orders`, `count_linked_config`, `image_512`, `name`, `pricelist_id`, `fiscal_position_id`, `use_timing`, `resource_calendar_id`, `slots_per_interval`, `interval_time`, `identification`, `is_return`, `color`, `attendance_ids` | `action_open_linked_orders`, `action_open_linked_config` |  | `point_of_sale` |
| `point_of_sale.view_pos_preset_tree` | list |  | `name`, `use_timing`, `identification`, `color` |  |  | `point_of_sale` |
| `pos_restaurant.view_pos_preset_form_inherit_pos_restaurant` | xpath | `point_of_sale.view_pos_preset_form` | `use_guest` |  |  | `pos_restaurant` |
| `pos_self_order.view_pos_preset_form` | xpath | `point_of_sale.view_pos_preset_form` | `available_in_self`, `service_at`, `mail_template_id` |  |  | `pos_self_order` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_preset_form` | Presets | list,form |  |  |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.preset.json`; views: `../../../schemas/interfaces/views/pos.preset.json`.
