# Event Booth Category (`event.booth.category`)

**Transport name:** `event.booth.category`  
**Storage name:** `event_booth_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_booth`  
**Extended by packages:** `event_booth_sale`, `website_event_booth_exhibitor`

Description: Event Booth Category

## Identity and behavior

- Mixins (classical inheritance): `image.mixin`
- Default ordering: `sequence ASC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (15)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `description` | Description | rich text |  | translatable |
| `booth_ids` | Booths | one to many | `event.booth` | visible only to groups `event.group_event_registration_desk`; inverse field `booth_category_id` |
| `product_id` | Product | many to one | `product.product` | required; default computed dynamically (_default_product_id); visible only to groups `event.group_event_registration_desk`; restricted by domain `[["service_tracking", "=", "event_booth"]]` |
| `price` | Price | float |  | computed by rule `_compute_price` and stored; visible only to groups `event.group_event_registration_desk` |
| `price_incl` | Price incl | float |  | computed by rule `_compute_price_incl` (not stored); visible only to groups `event.group_event_registration_desk` |
| `currency_id` | Currency | many to one |  | related through path `product_id.currency_id`; visible only to groups `event.group_event_registration_desk` |
| `price_reduce` | Price Reduce | float |  | computed by rule `_compute_price_reduce` (not stored); visible only to groups `event.group_event_registration_desk` |
| `price_reduce_taxinc` | Price Reduce Tax inc | float |  | computed by rule `_compute_price_reduce_taxinc` (not stored) |
| `image_1920` | Image 1920 | image |  | computed by rule `_compute_image_1920` and stored |
| `use_sponsor` | Create Sponsor | boolean |  | Help: If set, when booking a booth a sponsor will be created for the user |
| `sponsor_type_id` | Sponsor Level | many to one | `event.sponsor.type` |  |
| `exhibitor_type` | Sponsor Type | selection |  |  |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_product_id` | preparation rule | self | `event_booth_sale` |  |  |
| `_check_service_tracking` | validation | self | `event_booth_sale` | constrains: `product_id` |  |
| `_compute_image_1920` | computation | self | `event_booth_sale` | depends: `product_id` |  |
| `_compute_price` | computation | self | `event_booth_sale` | depends: `product_id` | By default price comes from category but can be changed by event people as product may be shared across various categories. |
| `_compute_price_incl` | computation | self | `event_booth_sale` | depends: `product_id`, `product_id.taxes_id`, `price` |  |
| `_compute_price_reduce` | computation | self | `event_booth_sale` | depends_context: ; depends: `product_id`, `price` |  |
| `_compute_price_reduce_taxinc` | computation | self | `event_booth_sale` | depends_context: ; depends: `product_id`, `price_reduce` |  |
| `_init_column` | internal rule | self, column_name | `event_booth_sale` |  | Initialize product_id for existing columns when installing sale bridge, to ensure required attribute is fulfilled. |
| `_get_exhibitor_type` | preparation rule | self | `website_event_booth_exhibitor` | model |  |
| `_onchange_use_sponsor` | on change | self | `website_event_booth_exhibitor` | onchange: `use_sponsor` |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_service_tracking` | ValidationError | The product, %(product_name)s , is used for Event Booth, it must have service_tracking set to "Event Booth". | `event_booth_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `event_booth` |
| `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |
| `base.group_public` | no | yes | no | no | `website_event_booth` |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_booth_category_view_form` | form |  | `image_1920`, `name`, `active`, `description` |  |  | `event_booth` |
| `event_booth.event_booth_category_view_tree` | list |  | `sequence`, `name` |  |  | `event_booth` |
| `event_booth.event_booth_category_view_search` | search |  | `name` |  | `Archived` | `event_booth` |
| `event_booth_sale.event_booth_category_view_form` | group | `event_booth.event_booth_category_view_form` | `currency_id`, `product_id`, `price` |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_category_view_tree` | field | `event_booth.event_booth_category_view_tree` | `name`, `currency_id`, `product_id`, `price` |  |  | `event_booth_sale` |
| `website_event_booth_exhibitor.event_booth_category_view_form` | group | `event_booth.event_booth_category_view_form` | `use_sponsor`, `sponsor_type_id`, `exhibitor_type` |  |  | `website_event_booth_exhibitor` |
| `website_event_booth_exhibitor.event_booth_category_view_tree` | field | `event_booth.event_booth_category_view_tree` | `name`, `use_sponsor`, `sponsor_type_id`, `exhibitor_type` |  |  | `website_event_booth_exhibitor` |
| `website_event_booth_exhibitor.event_booth_category_view_search` | xpath | `event_booth.event_booth_category_view_search` | `use_sponsor`, `sponsor_type_id`, `exhibitor_type` |  | `Sponsor type`, `Exhibitor type` | `website_event_booth_exhibitor` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_booth_category_action` | Booth Category | list,form |  |  |  | `event_booth` |

Machine-readable definition: `../../../schemas/data/entities/event.booth.category.json`; views: `../../../schemas/interfaces/views/event.booth.category.json`.
