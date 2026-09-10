# Event Booth Template (`event.type.booth`)

**Transport name:** `event.type.booth`  
**Storage name:** `event_type_booth`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_booth`  
**Extended by packages:** `event_booth_sale`

Description: Event Booth Template

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `event_type_id` | Event Category | many to one | `event.type` | required; indexed; on delete of the target: cascade |
| `booth_category_id` | Booth Category | many to one | `event.booth.category` | required; default computed dynamically (_get_default_booth_category); indexed; on delete of the target: restrict |
| `product_id` | Product | many to one |  | related through path `booth_category_id.product_id` |
| `price` | Price | float |  | related through path `booth_category_id.price` and stored |
| `currency_id` | Currency | many to one |  | related through path `booth_category_id.currency_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_booth_category` | preparation rule | self | `event_booth` |  | Assign booth category by default if only one exists |
| `_get_event_booth_fields_whitelist` | preparation rule | self | `event_booth_sale`, `event_booth` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_type_booth_view_form_from_type` | form |  | `name`, `booth_category_id` |  |  | `event_booth` |
| `event_booth.event_type_booth_view_form` | xpath | `event_type_booth_view_form_from_type` | `event_type_id` |  |  | `event_booth` |
| `event_booth.event_type_booth_view_tree_from_type` | list |  | `name`, `booth_category_id` |  |  | `event_booth` |
| `event_booth.event_type_booth_view_tree` | xpath | `event_type_booth_view_tree_from_type` | `event_type_id` |  |  | `event_booth` |
| `event_booth.event_type_booth_view_search` | search |  | `name` |  | `Booth Type` | `event_booth` |
| `event_booth_sale.event_type_booth_view_form_from_type` | field | `event_booth.event_type_booth_view_form_from_type` | `booth_category_id`, `currency_id`, `product_id`, `price` |  |  | `event_booth_sale` |
| `event_booth_sale.event_type_booth_view_tree_from_type` | field | `event_booth.event_type_booth_view_tree_from_type` | `booth_category_id`, `currency_id`, `product_id`, `price` |  |  | `event_booth_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_type_booth_action` | Event Type Booths | list,form |  |  |  | `event_booth` |

Machine-readable definition: `../../../schemas/data/entities/event.type.booth.json`; views: `../../../schemas/interfaces/views/event.type.booth.json`.
