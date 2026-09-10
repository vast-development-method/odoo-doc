# Lunch Supplier (`lunch.supplier`)

**Transport name:** `lunch.supplier`  
**Storage name:** `lunch_supplier`  
**Kind:** persistent entity (one table)  
**Defined by package:** `lunch`

Description: Lunch Supplier

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (42)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Vendor | many to one | `res.partner` | required |
| `name` | Name | single line text |  | related through path `partner_id.name` |
| `email` | Email | single line text |  | related through path `partner_id.email` |
| `email_formatted` | Email Formatted | single line text |  | read only; related through path `partner_id.email_formatted` |
| `phone` | Phone | single line text |  | related through path `partner_id.phone` |
| `street` | Street | single line text |  | related through path `partner_id.street` |
| `street2` | Street2 | single line text |  | related through path `partner_id.street2` |
| `zip_code` | Zip Code | single line text |  | related through path `partner_id.zip` |
| `city` | City | single line text |  | related through path `partner_id.city` |
| `state_id` | State | many to one | `res.country.state` | related through path `partner_id.state_id` |
| `country_id` | Country | many to one | `res.country` | related through path `partner_id.country_id` |
| `company_id` | Company | many to one | `res.company` | related through path `partner_id.company_id` and stored |
| `responsible_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('lunch.group_lunch_manager').id)]`; Help: The responsible is the person that will order lunch for everyone. It will be used as the 'from' when sending the automatic email. |
| `send_by` | Send Order By | selection |  | default `phone` |
| `automatic_email_time` | Order Time | float |  | required; default `12.0` |
| `cron_id` | Cron | many to one | `ir.cron` | required; read only; on delete of the target: cascade |
| `mon` | Mon | boolean |  | default `True` |
| `tue` | Tue | boolean |  | default `True` |
| `wed` | Wed | boolean |  | default `True` |
| `thu` | Thu | boolean |  | default `True` |
| `fri` | Fri | boolean |  | default `True` |
| `sat` | Sat | boolean |  |  |
| `sun` | Sun | boolean |  |  |
| `recurrency_end_date` | Until | date |  | Help: This field is used in order to |
| `available_location_ids` | Location | many to many | `lunch.location` |  |
| `available_today` | This is True when if the supplier is available today | boolean |  | computed by rule `_compute_available_today` (not stored); searchable through a search rule |
| `order_deadline_passed` | Order Deadline Passed | boolean |  | computed by rule `_compute_order_deadline_passed` (not stored) |
| `tz` | Timezone | selection |  | required; default computed dynamically (lambda self: self.env.user.tz or 'UTC') |
| `active` | Active | boolean |  | default `True` |
| `moment` | Moment | selection |  | required; default `am` |
| `delivery` | Delivery | selection |  | default `no_delivery` |
| `topping_label_1` | Extra 1 Label | single line text |  | required; default `Extras` |
| `topping_label_2` | Extra 2 Label | single line text |  | required; default `Beverages` |
| `topping_label_3` | Extra 3 Label | single line text |  | required; default `Extra Label 3` |
| `topping_ids_1` | Topping Identifiers 1 | one to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 1]]`; inverse field `supplier_id` |
| `topping_ids_2` | Topping Identifiers 2 | one to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 2]]`; inverse field `supplier_id` |
| `topping_ids_3` | Topping Identifiers 3 | one to many | `lunch.topping` | restricted by domain `[["topping_category", "=", 3]]`; inverse field `supplier_id` |
| `topping_quantity_1` | Extra 1 Quantity | selection |  | required; default `0_more` |
| `topping_quantity_2` | Extra 2 Quantity | selection |  | required; default `0_more` |
| `topping_quantity_3` | Extra 3 Quantity | selection |  | required; default `0_more` |
| `show_order_button` | Show Order Button | boolean |  | computed by rule `_compute_buttons` (not stored) |
| `show_confirm_button` | Show Confirm Button | boolean |  | computed by rule `_compute_buttons` (not stored) |

## Selection values

### `send_by` (Send Order By)

| Value | Label |
|---|---|
| `phone` | Phone |
| `mail` | Email |

### `moment` (Moment)

| Value | Label |
|---|---|
| `am` | AM |
| `pm` | PM |

### `delivery` (Delivery)

| Value | Label |
|---|---|
| `delivery` | Delivery |
| `no_delivery` | No Delivery |

### `topping_quantity_1` (Extra 1 Quantity)

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

### `topping_quantity_2` (Extra 2 Quantity)

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

### `topping_quantity_3` (Extra 3 Quantity)

| Value | Label |
|---|---|
| `0_more` | None or More |
| `1_more` | One or More |
| `1` | Only One |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_automatic_email_time_range` | Constraint | `CHECK(automatic_email_time >= 0 AND automatic_email_time <= 12)` | Automatic Email Sending Time should be between 0 and 12 | `lunch` |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `lunch` | depends: `phone` |  |
| `_sync_cron` | internal rule | self | `lunch` |  |  |
| `create` | lifecycle override | self, vals_list | `lunch` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `lunch` |  |  |
| `unlink` | lifecycle override | self | `lunch` |  |  |
| `_cancel_future_days` | internal rule | self, weekdays | `lunch` |  |  |
| `_get_current_orders` | preparation rule | self, state | `lunch` |  | Returns today's orders |
| `_send_auto_email` | internal rule | self | `lunch` |  | Send an email to the supplier with the order of the day |
| `_compute_available_today` | computation | self | `lunch` | depends: `recurrency_end_date`, `mon`, `tue`, `wed`, `thu`, `fri`, `sat`, `sun` |  |
| `_available_on_date` | internal rule | self, date | `lunch` |  |  |
| `_compute_order_deadline_passed` | computation | self | `lunch` | depends: `available_today`, `automatic_email_time`, `send_by` |  |
| `_search_available_today` | search rule | self, operator, value | `lunch` |  |  |
| `_compute_buttons` | computation | self | `lunch` |  |  |
| `action_send_orders` | user action | self | `lunch` |  |  |
| `action_confirm_orders` | user action | self | `lunch` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_send_auto_email` | UserError | Cannot send an email to this supplier! | `lunch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_lunch_user` | no | yes | no | no | `lunch` |
| `group_lunch_manager` | yes | yes | yes | yes | `lunch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Lunch supplier: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_supplier_view_tree` | list |  | `name`, `phone`, `email` |  |  | `lunch` |
| `lunch.lunch_supplier_view_form` | form |  | `name`, `partner_id`, `street`, `street2`, `city`, `state_id`, `zip_code`, `country_id`, `active`, `email`, `phone`, `company_id`, `responsible_id`, `tz`, `sun`, `recurrency_end_date`, `delivery`, `available_location_ids`, `send_by`, `automatic_email_time`, `moment`, `active`, `topping_label_1`, `topping_quantity_1`, `topping_ids_1`, `name`, `company_id`, `currency_id`, `price`, `topping_label_2`, `topping_quantity_2`, `topping_ids_2`, `name`, `company_id`, `currency_id`, `price`, `topping_label_3`, `topping_quantity_3`, `topping_ids_3`, `name`, `company_id`, `currency_id`, `price` |  |  | `lunch` |
| `lunch.lunch_supplier_view_kanban` | kanban |  | `display_name`, `city`, `country_id`, `city`, `country_id`, `email` |  |  | `lunch` |
| `lunch.lunch_supplier_view_search` | search |  | `name` |  | `Archived` | `lunch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `lunch.lunch_vendors_action` | Vendors | kanban,list,form |  |  |  | `lunch` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `lunch.lunch_order_mail_supplier` | Lunch: Supplier Order | Orders for {{ ctx.get('order', {}).get('company_name') }} |

Machine-readable definition: `../../../schemas/data/entities/lunch.supplier.json`; views: `../../../schemas/interfaces/views/lunch.supplier.json`.
