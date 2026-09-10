# Lunch Order (`lunch.order`)

**Transport name:** `lunch.order`  
**Storage name:** `lunch_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Order

## Identity and behavior

- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (36)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Product Name | single line text |  | read only; related through path `product_id.name` |
| `topping_ids_1` | Extras 1 | many to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 1]]`; association table `lunch_order_topping` |
| `topping_ids_2` | Extras 2 | many to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 2]]`; association table `lunch_order_topping` |
| `topping_ids_3` | Extras 3 | many to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 3]]`; association table `lunch_order_topping` |
| `product_id` | Product | many to one | `lunch.product` | required |
| `category_id` | Product Category | many to one |  | related through path `product_id.category_id` and stored |
| `date` | Order Date | date |  | required; default computed dynamically (fields.Date.context_today) |
| `supplier_id` | Vendor | many to one |  | related through path `product_id.supplier_id` and stored; indexed |
| `available_today` | Available Today | boolean |  | related through path `supplier_id.available_today` |
| `available_on_date` | Available On Date | boolean |  | computed by rule `_compute_available_on_date` (not stored) |
| `order_deadline_passed` | Order Deadline Passed | boolean |  | computed by rule `_compute_order_deadline_passed` (not stored) |
| `user_id` | User | many to one | `res.users` | default computed dynamically (lambda self: self.env.uid) |
| `lunch_location_id` | Lunch Location | many to one | `lunch.location` | default computed dynamically (lambda self: self.env.user.last_lunch_location_id) |
| `note` | Notes | multi line text |  |  |
| `price` | Total Price | monetary |  | read only; computed by rule `_compute_total_price` and stored |
| `active` | Active | boolean |  | default `True` |
| `state` | Status | selection |  | read only; default `new`; indexed |
| `notified` | Notified | boolean |  | default  |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company.id) |
| `currency_id` | Currency | many to one |  | related through path `company_id.currency_id` and stored |
| `quantity` | Quantity | float |  | required; default `1` |
| `display_toppings` | Extras | multi line text |  | computed by rule `_compute_display_toppings` and stored |
| `product_description` | Description | rich text |  | related through path `product_id.description` |
| `topping_label_1` | Topping Label 1 | single line text |  | related through path `product_id.supplier_id.topping_label_1` |
| `topping_label_2` | Topping Label 2 | single line text |  | related through path `product_id.supplier_id.topping_label_2` |
| `topping_label_3` | Topping Label 3 | single line text |  | related through path `product_id.supplier_id.topping_label_3` |
| `topping_quantity_1` | Topping Quantity 1 | selection |  | related through path `product_id.supplier_id.topping_quantity_1` |
| `topping_quantity_2` | Topping Quantity 2 | selection |  | related through path `product_id.supplier_id.topping_quantity_2` |
| `topping_quantity_3` | Topping Quantity 3 | selection |  | related through path `product_id.supplier_id.topping_quantity_3` |
| `image_1920` | Image 1920 | image |  | computed by rule `_compute_product_images` (not stored) |
| `image_128` | Image 128 | image |  | computed by rule `_compute_product_images` (not stored) |
| `available_toppings_1` | Available Toppings 1 | boolean |  | computed by rule `_compute_available_toppings` (not stored); Help: Are extras available for this product |
| `available_toppings_2` | Available Toppings 2 | boolean |  | computed by rule `_compute_available_toppings` (not stored); Help: Are extras available for this product |
| `available_toppings_3` | Available Toppings 3 | boolean |  | computed by rule `_compute_available_toppings` (not stored); Help: Are extras available for this product |
| `display_reorder_button` | Display Reorder Button | boolean |  | computed by rule `_compute_display_reorder_button` (not stored) |
| `display_add_button` | Display Add Button | boolean |  | computed by rule `_compute_display_add_button` (not stored) |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `new` | To Order |
| `ordered` | Ordered |
| `sent` | Sent |
| `confirmed` | Received |
| `cancelled` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_product_date` | Index | `(user_id, product_id, date)` |  | `lunch` |

## Operations (24)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_product_images` | computation | self | `lunch` | depends: `product_id` |  |
| `_compute_available_toppings` | computation | self | `lunch` | depends: `category_id` |  |
| `_compute_display_add_button` | computation | self | `lunch` | depends: `name` |  |
| `_compute_display_reorder_button` | computation | self | `lunch` | depends_context: `show_reorder_button`; depends: `state` |  |
| `_compute_available_on_date` | computation | self | `lunch` | depends: `date`, `supplier_id` |  |
| `_compute_order_deadline_passed` | computation | self | `lunch` | depends: `supplier_id`, `date` |  |
| `_get_topping_ids` | preparation rule | self, field, values | `lunch` |  |  |
| `_extract_toppings` | internal rule | self, values | `lunch` |  | If called in api.multi then it will pop topping_ids_1,2,3 from values |
| `_check_topping_quantity` | validation | self | `lunch` | constrains: `topping_ids_1`, `topping_ids_2`, `topping_ids_3` |  |
| `create` | lifecycle override | self, vals_list | `lunch` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `lunch` |  |  |
| `_find_matching_lines` | internal rule | self, values | `lunch` | model |  |
| `_compute_total_price` | computation | self | `lunch` | depends: `topping_ids_1`, `topping_ids_2`, `topping_ids_3`, `product_id`, `quantity` |  |
| `_compute_display_toppings` | computation | self | `lunch` | depends: `topping_ids_1`, `topping_ids_2`, `topping_ids_3` |  |
| `update_quantity` | operation | self, increment | `lunch` |  |  |
| `add_to_cart` | operation | self | `lunch` |  | This method currently does nothing, we currently need it in order to be able to reuse this model in place of a wizard |
| `_check_wallet` | validation | self | `lunch` |  |  |
| `action_order` | user action | self | `lunch` |  |  |
| `action_reorder` | user action | self | `lunch` |  |  |
| `action_confirm` | user action | self | `lunch` |  |  |
| `action_cancel` | user action | self | `lunch` |  |  |
| `action_reset` | user action | self | `lunch` |  |  |
| `action_send` | user action | self | `lunch` |  |  |
| `action_notify` | user action | self | `lunch` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_topping_quantity` | ValidationError | errors[quantity] % label | `lunch` |
| `_check_wallet` | ValidationError | Oh no! You don’t have enough money in your wallet to order your selected lunch! Contact your lunch manager to add some money to your wallet. | `lunch` |
| `action_order` | ValidationError | Product is no longer available. | `lunch` |
| `action_order` | UserError | The vendor related to this order is not available at the selected date. | `lunch` |
| `action_reorder` | UserError | The vendor related to this order is not available today. | `lunch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | yes | yes | yes | yes | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| lunch.order: Only new and cancelled order lines deleted. | `[(4,ref('lunch.group_lunch_user'))]` | `[('state', 'in', ('new', 'cancelled'))]` | 0 | 0 | 0 | 1 |
| lunch.order: Don't change confirmed order | `[(4, ref('base.group_user'))]` | `[('state', '!=', 'confirmed'), ('user_id', '=', user.id)]` | 0 | True | 0 | 0 |
| manager can do whatever | `[(4, ref('lunch.group_lunch_manager'))]` | `[(1, '=', 1)]` | 0 | True | 0 | 0 |
| Lunch order: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_order_view_search` | search |  | `name`, `user_id` |  | `My Orders`, `Not Received`, `Received`, `Cancelled`, `Today`, `Archived`, `User`, `Vendor`, `Order Date` | `lunch` |
| `lunch.lunch_order_view_tree` | list |  | `date`, `supplier_id`, `product_id`, `display_toppings`, `note`, `user_id`, `lunch_location_id`, `currency_id`, `price`, `state`, `company_id`, `display_reorder_button`, `notified`, `show_order_button`, `show_confirm_button` | `Receive`, `Re-order`, `Confirm`, `Cancel`, `Reset`, `Send Notification`, `Send Orders`, `Confirm Orders` |  | `lunch` |
| `lunch.lunch_order_view_kanban` | kanban |  | `currency_id`, `notified`, `product_id`, `state`, `note`, `price`, `date`, `user_id` |  |  | `lunch` |
| `lunch.lunch_order_view_pivot` | pivot |  | `date`, `supplier_id` |  |  | `lunch` |
| `lunch.lunch_order_view_graph` | graph |  | `product_id` |  |  | `lunch` |
| `lunch.lunch_order_view_form` | form |  | `company_id`, `date`, `currency_id`, `quantity`, `product_id`, `state`, `category_id`, `available_toppings_1`, `available_toppings_2`, `available_toppings_3`, `supplier_id`, `order_deadline_passed`, `available_today`, `image_1920`, `name`, `price`, `topping_label_1`, `topping_ids_1`, `topping_label_2`, `topping_ids_2`, `topping_label_3`, `topping_ids_3`, `product_description`, `note` | `Add To Cart`, `Discard` |  | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_order_action` | My Orders | list,kanban,pivot |  | `{"search_default_is_mine":1, "search_default_group_by_date": 1, 'show_reorder_button': True}` |  | `lunch` |
| `lunch.lunch_order_action_by_supplier` | Today's Orders | list,kanban |  | `{"search_default_group_by_supplier":1, "search_default_date_filter":1}` |  | `lunch` |
| `lunch.lunch_order_action_control_suppliers` | Control Vendors | list,kanban,pivot |  | `{"search_default_group_by_supplier":1}` |  | `lunch` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `lunch.lunch_order_action_confirm` | Lunch: Receive meals | code |  | yes |
| `lunch.lunch_order_action_cancel` | Lunch: Cancel meals | code |  | yes |
| `lunch.lunch_order_action_notify` | Lunch: Send notifications | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/lunch.order.json`; views: `../../../schemas/interfaces/views/lunch.order.json`.
