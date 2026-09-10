# Edit Attendee Details on Sales Confirmation (`registration.editor`)

**Transport name:** `registration.editor`  
**Storage name:** `registration_editor`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `event_sale`

Description: Edit Attendee Details on Sales Confirmation

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sale_order_id` | Sales Order | many to one | `sale.order` | required; on delete of the target: cascade |
| `event_registration_ids` | Registrations to Edit | one to many | `registration.editor.line` | inverse field `editor_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `event_sale` | model |  |
| `action_make_registration` | user action | self | `event_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_sale.view_event_registration_editor_form` | form |  | `sale_order_id`, `event_registration_ids`, `event_id`, `registration_id`, `event_slot_id`, `event_ticket_id`, `name`, `email`, `phone`, `company_id`, `sale_order_line_id` | `Create/Update registrations` |  | `event_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_sale.action_sale_order_event_registration` | Event Registrations | form |  | `{}` | new | `event_sale` |

Machine-readable definition: `../../../schemas/data/entities/registration.editor.json`; views: `../../../schemas/interfaces/views/registration.editor.json`.
