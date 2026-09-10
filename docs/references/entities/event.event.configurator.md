# Event Configurator (`event.event.configurator`)

**Transport name:** `event.event.configurator`  
**Storage name:** `event_event_configurator`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `event_sale`

Description: Event Configurator

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | read only |
| `event_id` | Event | many to one | `event.event` |  |
| `event_slot_id` | Slot | many to one | `event.slot` | computed by rule `_compute_event_slot_id` and stored; restricted by domain `[('event_id', '=', event_id)]` |
| `event_ticket_id` | Ticket Type | many to one | `event.event.ticket` | computed by rule `_compute_event_ticket_id` and stored; restricted by domain `[('event_id', '=', event_id)]` |
| `is_multi_slots` | Is Multi Slots | boolean |  | related through path `event_id.is_multi_slots` |
| `has_available_tickets` | Has Available Tickets | boolean |  | computed by rule `_compute_has_available_tickets` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `check_event_id` | validation | self | `event_sale` | constrains: `event_id`, `event_slot_id`, `event_ticket_id` |  |
| `_compute_has_available_tickets` | computation | self | `event_sale` | depends: `product_id` |  |
| `_compute_event_slot_id` | computation | self | `event_sale` | depends: `is_multi_slots` | Pre-select the slot of the multi slots event selected if it is the only one |
| `_compute_event_ticket_id` | computation | self | `event_sale` | depends: `event_id` | Pre-select the ticket of the event selected if it is the only one |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_event_id` | ValidationError | '\n'.join(error_messages) | `event_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_sale.event_configurator_view_form` | form |  | `has_available_tickets`, `event_id`, `event_slot_id`, `event_ticket_id`, `product_id` | `Add`, `Discard`, `Close` |  | `event_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_sale.event_configurator_action` | Select an Event | form |  | `{'name_with_seats_availability': True}` | new | `event_sale` |

Machine-readable definition: `../../../schemas/data/entities/event.event.configurator.json`; views: `../../../schemas/interfaces/views/event.event.configurator.json`.
