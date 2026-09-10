# Event Template Ticket (`event.type.ticket`)

**Transport name:** `event.type.ticket`  
**Storage name:** `event_type_ticket`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_product`

Description: Event Template Ticket

## Identity and behavior

- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `name` | Name | single line text |  | required; default computed dynamically (lambda self: _('Registration')); translatable |
| `description` | Description | multi line text |  | computed by rule `_compute_description` and stored; translatable; Help: A description of the ticket that you want to communicate to your customers.; extended by packages `event_product` |
| `event_type_id` | Event Category | many to one | `event.type` | required; on delete of the target: cascade |
| `seats_limited` | Limit Attendees | boolean |  | read only; computed by rule `_compute_seats_limited` and stored |
| `seats_max` | Maximum Attendees | integer |  | Help: Define the number of available tickets. If you have too many registrations you will not be able to sell tickets anymore. Set 0 to ignore this rule set as unlimited. |
| `product_id` | Product | many to one | `product.product` | required; default computed dynamically (_default_product_id); indexed; restricted by domain `[["service_tracking", "=", "event"]]` |
| `currency_id` | Currency | many to one |  | related through path `product_id.currency_id` |
| `price` | Price | float |  | computed by rule `_compute_price` and stored |
| `price_reduce` | Price Reduce | float |  | computed by rule `_compute_price_reduce` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_seats_limited` | computation | self | `event` | depends: `seats_max` |  |
| `_get_event_ticket_fields_whitelist` | preparation rule | self | `event_product`, `event` | model | Whitelist of fields that are copied from event_type_ticket_ids to event_ticket_ids when changing the event_type_id field of event.event |
| `_default_product_id` | preparation rule | self | `event_product` |  |  |
| `_compute_price` | computation | self | `event_product` | depends: `product_id` |  |
| `_compute_description` | computation | self | `event_product` | depends: `product_id` |  |
| `_compute_price_reduce` | computation | self | `event_product` | depends_context: ; depends: `product_id`, `price` |  |
| `_init_column` | internal rule | self, column_name | `event_product` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.event_type_ticket_view_tree_from_type` | list |  | `sequence`, `name`, `description`, `seats_max`, `seats_limited` |  |  | `event` |
| `event.event_type_ticket_view_form_from_type` | form |  | `name`, `description`, `seats_limited`, `seats_max` |  |  | `event` |
| `event.event_type_ticket_view_tree` | xpath | `event_type_ticket_view_tree_from_type` |  |  |  | `event` |
| `event.event_type_ticket_view_form` | xpath | `event_type_ticket_view_form_from_type` | `event_type_id` |  |  | `event` |
| `event_product.event_type_ticket_view_tree_from_type` | field | `event.event_type_ticket_view_tree_from_type` | `name`, `product_id` |  |  | `event_product` |
| `event_product.event_type_ticket_view_form_from_type` | field | `event.event_type_ticket_view_form_from_type` | `name`, `product_id` |  |  | `event_product` |

Machine-readable definition: `../../../schemas/data/entities/event.type.ticket.json`; views: `../../../schemas/interfaces/views/event.type.ticket.json`.
