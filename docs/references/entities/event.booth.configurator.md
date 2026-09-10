# Event Booth Configurator (`event.booth.configurator`)

**Transport name:** `event.booth.configurator`  
**Storage name:** `event_booth_configurator`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `event_booth_sale`

Description: Event Booth Configurator

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | read only |
| `sale_order_line_id` | Sale Order Line | many to one | `sale.order.line` | read only |
| `event_id` | Event | many to one | `event.event` | required |
| `event_booth_category_available_ids` | Event Booth Category Available | many to many |  | read only; related through path `event_id.event_booth_category_available_ids` |
| `event_booth_category_id` | Booth Category | many to one | `event.booth.category` | required; computed by rule `_compute_event_booth_category_id` and stored |
| `event_booth_ids` | Booth | many to many | `event.booth` | required; computed by rule `_compute_event_booth_ids` and stored |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_event_booth_category_id` | computation | self | `event_booth_sale` | depends: `event_id` |  |
| `_compute_event_booth_ids` | computation | self | `event_booth_sale` | depends: `event_id`, `event_booth_category_id` |  |
| `_check_if_no_booth_ids` | validation | self | `event_booth_sale` | constrains: `event_booth_ids` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_if_no_booth_ids` | ValidationError | You have to select at least one booth. | `event_booth_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `event_booth_sale` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_booth_sale.event_booth_configurator_view_form` | form |  | `product_id`, `event_id`, `event_booth_category_available_ids`, `event_booth_category_id`, `event_booth_ids` | `Ok`, `Cancel` |  | `event_booth_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_booth_sale.event_booth_configurator_action` | Select an event booth | form |  |  | new | `event_booth_sale` |

Machine-readable definition: `../../../schemas/data/entities/event.booth.configurator.json`; views: `../../../schemas/interfaces/views/event.booth.configurator.json`.
