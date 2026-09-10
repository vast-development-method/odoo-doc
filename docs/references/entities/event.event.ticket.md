# Event Ticket (`event.event.ticket`)

**Transport name:** `event.event.ticket`  
**Storage name:** `event_event_ticket`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_product`, `event_sale`, `pos_event`, `website_event_sale`

Description: Event Ticket

## Identity and behavior

- Mixins (classical inheritance): `event.type.ticket`, `pos.load.mixin`
- Default ordering: `event_id, sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_type_id` | Event Type | many to one |  | on delete of the target: set null |
| `event_id` | Event | many to one | `event.event` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | related through path `event_id.company_id` |
| `start_sale_datetime` | Registration Start | date and time |  |  |
| `end_sale_datetime` | Registration End | date and time |  |  |
| `is_launched` | Are sales launched | boolean |  | computed by rule `_compute_is_launched` (not stored) |
| `is_expired` | Is Expired | boolean |  | computed by rule `_compute_is_expired` (not stored) |
| `sale_available` | Is Available | boolean |  | computed by rule `_compute_sale_available` (not stored); Help: Whether it is possible to sell these tickets |
| `registration_ids` | Registrations | one to many | `event.registration` | inverse field `event_ticket_id` |
| `seats_reserved` | Reserved Seats | integer |  | computed by rule `_compute_seats` (not stored) |
| `seats_available` | Available Seats | integer |  | computed by rule `_compute_seats` (not stored) |
| `seats_used` | Used Seats | integer |  | computed by rule `_compute_seats` (not stored) |
| `seats_taken` | Taken Seats | integer |  | computed by rule `_compute_seats` (not stored) |
| `limit_max_per_order` | Limit per Order | integer |  | default ; Help: Maximum of this product per order. Set to 0 to ignore this rule |
| `is_sold_out` | Sold Out | boolean |  | computed by rule `_compute_is_sold_out` (not stored); Help: Whether seats are not available for this ticket. |
| `color` | Color | single line text |  | default `#875A7B` |
| `price_reduce_taxinc` | Price Reduce Tax inc | float |  | computed by rule `_compute_price_reduce_taxinc` (not stored) |
| `price_incl` | Price include | float |  | computed by rule `_compute_price_incl` (not stored) |

## Operations (18)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `event` | model |  |
| `_compute_is_expired` | computation | self | `event` | depends: `end_sale_datetime`, `event_id.date_tz` |  |
| `_compute_is_launched` | computation | self | `event` | depends: `start_sale_datetime`, `event_id.date_tz` |  |
| `_compute_sale_available` | computation | self | `event_product`, `event` | depends: `is_expired`, `start_sale_datetime`, `event_id.date_tz`, `seats_available`, `seats_max`; depends: `product_id.active` |  |
| `_compute_seats` | computation | self | `event` | depends: `seats_max`, `registration_ids.state`, `registration_ids.active` | Determine available, reserved, used and taken seats. |
| `_compute_is_sold_out` | computation | self | `event` | depends: `seats_limited`, `seats_available`, `event_id.event_registrations_sold_out` |  |
| `_constrains_dates_coherency` | validation | self | `event` | constrains: `start_sale_datetime`, `end_sale_datetime` |  |
| `_constrains_limit_max_per_order` | validation | self | `event` | constrains: `limit_max_per_order`, `seats_max` |  |
| `_compute_display_name` | computation | self | `event` | depends: `seats_max`, `seats_available`; depends_context: `name_with_seats_availability` | Adds ticket seats availability if requested by context. Always display the name without availabilities if the event is multi slots because the availability displayed won't be relative to the possible slot combinations but only relative to the event and this will confuse the user. |
| `_get_current_limit_per_order` | preparation rule | self, event_slot, event | `event` |  | Compute the maximum possible number of tickets for an order, taking into account the given event_slot if applicable. If no ticket is created (alone event), event_id argument is used. Then return the dictionary with False as key. |
| `_get_ticket_multiline_description` | preparation rule | self | `event_sale`, `event` |  | Compute a multiline description of this ticket. It is used when ticket description are necessary without having to encode it manually, like sales information. |
| `_set_tz_context` | internal rule | self | `event` |  |  |
| `_unlink_except_if_registrations` | internal rule | self | `event` | ondelete |  |
| `_compute_price_reduce_taxinc` | computation | self | `event_product` |  |  |
| `_compute_price_incl` | computation | self | `event_product` | depends: `product_id`, `product_id.taxes_id`, `price` |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_event` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_event` | model |  |
| `_show_discount` | internal rule | self | `website_event_sale` |  | Determine if the discount should be shown on the website for this ticket. |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_dates_coherency` | UserError | The stop date cannot be earlier than the start date. Please check ticket %(ticket_name)s | `event` |
| `_constrains_limit_max_per_order` | UserError | The limit per order cannot be greater than the maximum seats number. Please check ticket %(ticket_name)s | `event` |
| `_constrains_limit_max_per_order` | UserError | The limit per order cannot be greater than %(limit_orderable)s. Please check ticket %(ticket_name)s | `event` |
| `_constrains_limit_max_per_order` | UserError | The limit per order must be positive. Please check ticket %(ticket_name)s | `event` |
| `_unlink_except_if_registrations` | UserError | The following tickets cannot be deleted while they have one or more registrations linked to them: - %s | `event` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `event` |
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_user` | yes | yes | yes | yes | `event` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_event` |
| `base.group_public` | no | yes | no | no | `website_event` |
| `base.group_portal` | no | yes | no | no | `website_event` |
| `base.group_user` | no | yes | no | no | `website_event` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event/Ticket: multi-company | global (all users) | `[('event_id.company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Event Ticket: public/portal: published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | True | False | False | False |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_event_ticket_view_tree_from_event` | list |  | `sequence`, `name`, `description`, `start_sale_datetime`, `end_sale_datetime`, `seats_max`, `seats_max`, `seats_taken`, `seats_taken`, `limit_max_per_order`, `color` |  |  | `event` |
| `event.event_event_ticket_view_form_from_event` | form |  | `name`, `description`, `start_sale_datetime`, `end_sale_datetime`, `seats_max`, `seats_reserved` |  |  | `event` |
| `event.event_event_ticket_view_kanban_from_event` | kanban |  | `name`, `seats_reserved` |  |  | `event` |
| `event.event_event_ticket_view_tree` | xpath | `event_event_ticket_view_tree_from_event` |  |  |  | `event` |
| `event.event_event_ticket_form_view` | form |  | `name`, `event_id`, `seats_limited`, `seats_available`, `start_sale_datetime`, `end_sale_datetime`, `seats_max`, `seats_reserved`, `seats_used`, `is_expired` |  |  | `event` |
| `event_product.event_event_ticket_view_tree_from_event` | field | `event.event_event_ticket_view_tree_from_event` | `start_sale_datetime` |  |  | `event_product` |
| `event_product.event_event_ticket_view_form_from_event` | field | `event.event_event_ticket_view_form_from_event` | `name`, `product_id` |  |  | `event_product` |
| `event_product.event_event_ticket_view_kanban_from_event` | field | `event.event_event_ticket_view_kanban_from_event` | `name`, `price` |  |  | `event_product` |
| `event_product.event_event_ticket_form_view` | field | `event.event_event_ticket_form_view` | `end_sale_datetime`, `price`, `price_reduce` |  |  | `event_product` |

Machine-readable definition: `../../../schemas/data/entities/event.event.ticket.json`; views: `../../../schemas/interfaces/views/event.event.ticket.json`.
