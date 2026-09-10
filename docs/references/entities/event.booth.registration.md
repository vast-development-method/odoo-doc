# Event Booth Registration (`event.booth.registration`)

**Transport name:** `event.booth.registration`  
**Storage name:** `event_booth_registration`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_booth_sale`  
**Extended by packages:** `website_event_booth_sale_exhibitor`

Description: Event Booth Registration

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sale_order_line_id` | Sale Order Line | many to one | `sale.order.line` | required; indexed; on delete of the target: cascade |
| `event_booth_id` | Booth | many to one | `event.booth` | required; indexed |
| `partner_id` | Partner | many to one | `res.partner` | related through path `sale_order_line_id.order_partner_id` and stored |
| `contact_name` | Contact Name | single line text |  | computed by rule `_compute_contact_name` and stored |
| `contact_email` | Contact Email | single line text |  | computed by rule `_compute_contact_email` and stored |
| `contact_phone` | Contact Phone | single line text |  | computed by rule `_compute_contact_phone` and stored |
| `sponsor_name` | Sponsor Name | single line text |  |  |
| `sponsor_email` | Sponsor Email | single line text |  |  |
| `sponsor_phone` | Sponsor Phone | single line text |  |  |
| `sponsor_subtitle` | Sponsor Slogan | single line text |  |  |
| `sponsor_website_description` | Sponsor Description | rich text |  |  |
| `sponsor_image_512` | Sponsor Logo | image |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_registration` | Constraint | `unique(sale_order_line_id, event_booth_id)` | There can be only one registration for a booth by sale order line | `event_booth_sale` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_contact_name` | computation | self | `event_booth_sale` | depends: `partner_id` |  |
| `_compute_contact_email` | computation | self | `event_booth_sale` | depends: `partner_id` |  |
| `_compute_contact_phone` | computation | self | `event_booth_sale` | depends: `partner_id` |  |
| `_get_fields_for_booth_confirmation` | preparation rule | self | `event_booth_sale`, `website_event_booth_sale_exhibitor` | model |  |
| `action_confirm` | user action | self | `event_booth_sale` |  |  |
| `_cancel_pending_registrations` | internal rule | self | `event_booth_sale` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | yes | `event_booth_sale` |
| `event.group_event_registration_desk` | no | yes | no | no | `event_booth_sale` |
| `event.group_event_user` | yes | yes | yes | yes | `event_booth_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_booth_sale.event_booth_registration_view_form` | form |  | `partner_id`, `event_booth_id`, `sale_order_line_id`, `contact_name`, `contact_email`, `contact_phone` |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_registration_view_tree` | list |  | `partner_id`, `event_booth_id`, `sale_order_line_id`, `contact_name`, `contact_email`, `contact_phone` |  |  | `event_booth_sale` |

Machine-readable definition: `../../../schemas/data/entities/event.booth.registration.json`; views: `../../../schemas/interfaces/views/event.booth.registration.json`.
