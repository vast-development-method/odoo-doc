# Edit Attendee Line on Sales Confirmation (`registration.editor.line`)

**Transport name:** `registration.editor.line`  
**Storage name:** `registration_editor_line`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `event_sale`

Description: Edit Attendee Line on Sales Confirmation

## Identity and behavior

- Default ordering: `id desc`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `editor_id` | Editor | many to one | `registration.editor` |  |
| `sale_order_line_id` | Sales Order Line | many to one | `sale.order.line` |  |
| `event_id` | Event | many to one | `event.event` | required |
| `company_id` | Company | many to one |  | related through path `event_id.company_id` |
| `registration_id` | Original Registration | many to one | `event.registration` |  |
| `event_slot_id` | Event Slot | many to one | `event.slot` |  |
| `event_ticket_id` | Event Ticket | many to one | `event.event.ticket` |  |
| `email` | Email | single line text |  |  |
| `phone` | Phone | single line text |  |  |
| `name` | Name | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_prepare_registration_data` | preparation rule | self, include_event_values | `event_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | yes | `event_sale` |

Machine-readable definition: `../../../schemas/data/entities/registration.editor.line.json`.
