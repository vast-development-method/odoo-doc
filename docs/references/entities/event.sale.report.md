# Event Sales Report (`event.sale.report`)

**Transport name:** `event.sale.report`  
**Storage name:** `event_sale_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_sale`  
**Extended by packages:** `website_event_sale`

Description: Event Sales Report

## Identity and behavior

- Display name field: `sale_order_line_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_type_id` | Event Type | many to one | `event.type` | read only |
| `event_id` | Event | many to one | `event.event` | read only |
| `event_date_begin` | Event Start Date | date |  | read only |
| `event_date_end` | Event End Date | date |  | read only |
| `event_slot_id` | Event Slot | many to one | `event.slot` | read only |
| `event_ticket_id` | Event Ticket | many to one | `event.event.ticket` | read only |
| `event_ticket_price` | Ticket price | float |  | read only |
| `event_registration_create_date` | Registration Date | date |  | read only |
| `event_registration_state` | Registration Status | selection |  | read only |
| `active` | Is registration active (not archived)? | boolean |  |  |
| `event_registration_id` | Event Registration | many to one | `event.registration` | read only |
| `event_registration_name` | Attendee Name | single line text |  | read only |
| `product_id` | Product | many to one | `product.product` | read only |
| `sale_order_id` | Sale Order | many to one | `sale.order` | read only |
| `sale_order_date` | Order Date | date and time |  | read only |
| `sale_order_partner_id` | Customer | many to one | `res.partner` | read only |
| `sale_order_state` | Sale Order Status | selection |  | read only |
| `sale_order_user_id` | Salesperson | many to one | `res.users` | read only |
| `sale_order_line_id` | Sale Order Line | many to one | `sale.order.line` | read only |
| `sale_price` | Revenues | float |  | read only |
| `sale_price_untaxed` | Untaxed Revenues | float |  | read only |
| `invoice_partner_id` | Invoice Address | many to one | `res.partner` | read only |
| `sale_status` | Payment Status | selection |  |  |
| `company_id` | Company | many to one | `res.company` | read only |
| `is_published` | Published Events | boolean |  | read only |

## Selection values

### `event_registration_state` (Registration Status)

| Value | Label |
|---|---|
| `draft` | Unconfirmed |
| `cancel` | Cancelled |
| `open` | Confirmed |
| `done` | Attended |

### `sale_status` (Payment Status)

| Value | Label |
|---|---|
| `to_pay` | Not Sold |
| `sold` | Sold |
| `free` | Free |

## State fields

State machine fields of this entity: `event_registration_state`, `sale_order_state`, `sale_status`. Transitions are specified in the domain documents.

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `event_sale` |  |  |
| `_query` | internal rule | self, with_, select, join, group_by | `event_sale` |  |  |
| `_with_clause` | internal rule | self, *with_ | `event_sale` |  |  |
| `_select_clause` | internal rule | self, *select | `event_sale`, `website_event_sale` |  |  |
| `_from_clause` | internal rule | self, *join_ | `event_sale` |  |  |
| `_group_by_clause` | internal rule | self, *group_by | `event_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_manager` | no | yes | no | no | `event_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Sales Report multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_sale.event_sale_report_view_graph` | graph |  | `sale_price`, `event_registration_create_date`, `event_ticket_price` |  |  | `event_sale` |
| `event_sale.event_sale_report_view_form` | form |  | `event_type_id`, `event_id`, `event_date_begin`, `event_registration_id`, `event_registration_name`, `event_registration_create_date`, `event_slot_id`, `event_ticket_id`, `event_registration_state`, `sale_order_partner_id`, `sale_order_id`, `product_id`, `event_ticket_price`, `sale_price_untaxed`, `sale_price` |  |  | `event_sale` |
| `event_sale.event_sale_report_view_pivot` | pivot |  | `sale_price_untaxed`, `sale_price`, `event_id`, `product_id`, `event_ticket_price` |  |  | `event_sale` |
| `event_sale.event_sale_report_view_tree` | list |  | `event_id`, `event_slot_id`, `event_ticket_id`, `product_id`, `event_ticket_price`, `sale_price_untaxed`, `sale_price`, `event_registration_state`, `sale_order_partner_id`, `invoice_partner_id`, `event_registration_name`, `sale_order_state` |  |  | `event_sale` |
| `event_sale.event_sale_report_view_search` | search |  | `event_id`, `event_registration_name`, `sale_order_partner_id`, `company_id` |  | `Non-free tickets`, `Free`, `Pending payment`, `Sold`, `Registration Date`, `Upcoming/Running`, `Past Events`, `Event Start Date`, `Event End Date`, `Event Type`, `Event`, `Product`, `Slot`, `Ticket`, `Registration Status`, `Sale Order Status`, `Customer` | `event_sale` |
| `website_event_sale.event_sale_report_view_search` | xpath | `event_sale.event_sale_report_view_search` |  |  | `Published Events` | `website_event_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_sale.event_sale_report_action` | Revenues | graph,pivot |  | `{             'search_default_priced_tickets': 1,             'search_default_event_date_start': 1,             'pivot_measures': ['__count__', 'sale_price_untaxed', 'sale_price'],         }` |  | `event_sale` |

Machine-readable definition: `../../../schemas/data/entities/event.sale.report.json`; views: `../../../schemas/interfaces/views/event.sale.report.json`.
