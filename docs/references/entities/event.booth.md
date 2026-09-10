# Event Booth (`event.booth`)

**Transport name:** `event.booth`  
**Storage name:** `event_booth`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event_booth`  
**Extended by packages:** `event_booth_sale`, `website_event_booth_exhibitor`

Description: Event Booth

## Identity and behavior

- Mixins (classical inheritance): `event.type.booth`, `mail.thread`, `mail.activity.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_type_id` | Event Type | many to one |  | on delete of the target: set null |
| `event_id` | Event | many to one | `event.event` | required; indexed; on delete of the target: cascade |
| `partner_id` | Renter | many to one | `res.partner` | changes are tracked in the message thread; not copied on duplication |
| `contact_name` | Renter Name | single line text |  | computed by rule `_compute_contact_name` and stored; not copied on duplication |
| `contact_email` | Renter Email | single line text |  | computed by rule `_compute_contact_email` and stored; not copied on duplication |
| `contact_phone` | Renter Phone | single line text |  | computed by rule `_compute_contact_phone` and stored; not copied on duplication |
| `state` | Status | selection |  | required; default `available`; changes are tracked in the message thread |
| `is_available` | Is Available | boolean |  | computed by rule `_compute_is_available` (not stored); searchable through a search rule |
| `event_booth_registration_ids` | Event Booth Registration | one to many | `event.booth.registration` | inverse field `event_booth_id` |
| `sale_order_line_registration_ids` | sales order Lines with reservations | many to many | `sale.order.line` | not copied on duplication; visible only to groups `sales_team.group_sale_salesman`; association table `event_booth_registration` |
| `sale_order_line_id` | Final Sale Order Line | many to one | `sale.order.line` | indexed (btree_not_null); not copied on duplication; visible only to groups `sales_team.group_sale_salesman`; on delete of the target: set null |
| `sale_order_id` | Sale Order | many to one |  | read only; related through path `sale_order_line_id.order_id` and stored; indexed (btree_not_null); visible only to groups `sales_team.group_sale_salesman` |
| `is_paid` | Is Paid | boolean |  | not copied on duplication |
| `use_sponsor` | Use Sponsor | boolean |  | related through path `booth_category_id.use_sponsor` |
| `sponsor_type_id` | Sponsor Type | many to one |  | related through path `booth_category_id.sponsor_type_id` |
| `sponsor_id` | Sponsor | many to one | `event.sponsor` | not copied on duplication |
| `sponsor_name` | Sponsor Name | single line text |  | related through path `sponsor_id.name` |
| `sponsor_email` | Sponsor Email | single line text |  | related through path `sponsor_id.email` |
| `sponsor_phone` | Sponsor Phone | single line text |  | related through path `sponsor_id.phone` |
| `sponsor_subtitle` | Sponsor Slogan | single line text |  | related through path `sponsor_id.subtitle` |
| `sponsor_website_description` | Sponsor Description | rich text |  | related through path `sponsor_id.website_description` |
| `sponsor_image_512` | Sponsor Logo | image |  | related through path `sponsor_id.image_512` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `available` | Available |
| `unavailable` | Unavailable |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_contact_name` | computation | self | `event_booth` | depends: `partner_id` |  |
| `_compute_contact_email` | computation | self | `event_booth` | depends: `partner_id` |  |
| `_compute_contact_phone` | computation | self | `event_booth` | depends: `partner_id` |  |
| `_compute_is_available` | computation | self | `event_booth` | depends: `state` |  |
| `_search_is_available` | search rule | self, operator, value | `event_booth` |  |  |
| `create` | lifecycle override | self, vals_list | `event_booth` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `event_booth` |  |  |
| `_post_confirmation_message` | internal rule | self | `event_booth` |  |  |
| `action_confirm` | user action | self, additional_values | `event_booth` |  |  |
| `_action_post_confirm` | internal rule | self, write_vals | `event_booth`, `website_event_booth_exhibitor` |  |  |
| `_unlink_except_linked_sale_order` | internal rule | self | `event_booth_sale` | ondelete |  |
| `action_set_paid` | user action | self | `event_booth_sale` |  |  |
| `action_view_sale_order` | user action | self | `event_booth_sale` |  |  |
| `_get_booth_multiline_description` | preparation rule | self | `event_booth_sale` |  |  |
| `action_view_sponsor` | user action | self | `website_event_booth_exhibitor` |  |  |
| `_get_or_create_sponsor` | preparation rule | self, vals | `website_event_booth_exhibitor` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_linked_sale_order` | UserError | You can't delete the following booths as they are linked to sales orders: %(booths)s | `event_booth_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `event_booth` |
| `event.group_event_registration_desk` | no | yes | no | no | `event_booth` |
| `event.group_event_user` | yes | yes | yes | yes | `event_booth` |
| `event.group_event_manager` | yes | yes | yes | yes | `event_booth` |
| `base.group_public` | no | yes | no | no | `website_event_booth` |
| `base.group_portal` | no | yes | no | no | `website_event_booth` |
| `base.group_user` | no | yes | no | no | `website_event_booth` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Booth: public/portal: published read | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('event_id.website_published', '=', True)]` | True | False | False | False |

## Views (17)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_booth_view_form_from_event` | form |  | `state`, `name`, `booth_category_id`, `partner_id`, `contact_name`, `contact_email`, `contact_phone` |  |  | `event_booth` |
| `event_booth.event_booth_view_form` | field | `event_booth_view_form_from_event` | `booth_category_id`, `event_id` |  |  | `event_booth` |
| `event_booth.event_booth_view_form_simple_from_event` | xpath | `event_booth_view_form_from_event` |  |  |  | `event_booth` |
| `event_booth.event_booth_view_tree_from_event` | list |  | `name`, `booth_category_id`, `partner_id`, `contact_name`, `contact_email`, `contact_phone`, `state` |  |  | `event_booth` |
| `event_booth.event_booth_view_tree` | field | `event_booth_view_tree_from_event` | `name`, `event_id` |  |  | `event_booth` |
| `event_booth.event_booth_view_kanban_from_event` | kanban |  | `name`, `name`, `booth_category_id`, `activity_ids` |  |  | `event_booth` |
| `event_booth.event_booth_view_kanban` | xpath | `event_booth_view_kanban_from_event` | `event_id` |  |  | `event_booth` |
| `event_booth.event_booth_view_form_quick_create` | form |  | `name`, `booth_category_id` |  |  | `event_booth` |
| `event_booth.event_booth_view_search` | search |  | `name`, `contact_name`, `contact_email`, `event_id` |  | `Available`, `Unavailable`, `group_by_state`, `group_by_partner_id`, `group_by_booth_category_id`, `Event` | `event_booth` |
| `event_booth.event_booth_view_graph` | graph |  | `booth_category_id` |  |  | `event_booth` |
| `event_booth.event_booth_view_pivot` | pivot |  | `booth_category_id` |  |  | `event_booth` |
| `event_booth_sale.event_booth_view_form_from_event` | div | `event_booth.event_booth_view_form_from_event` | `sale_order_id` | `action_view_sale_order` |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_tree_from_event` | field | `event_booth.event_booth_view_tree_from_event` | `partner_id`, `currency_id`, `price` |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_search` | xpath | `event_booth.event_booth_view_search` | `sale_order_id` |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_graph` | xpath | `event_booth.event_booth_view_graph` | `price` |  |  | `event_booth_sale` |
| `event_booth_sale.event_booth_view_pivot` | xpath | `event_booth.event_booth_view_pivot` | `price` |  |  | `event_booth_sale` |
| `website_event_booth_exhibitor.event_booth_view_form_from_event` | div | `event_booth.event_booth_view_form_from_event` | `sponsor_id` | `Sponsor` |  | `website_event_booth_exhibitor` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event_booth.event_booth_action` | Booths | kanban,list,form,graph,pivot | `[]` | `{'search_default_group_by_state': 1}` |  | `event_booth` |
| `event_booth.event_booth_action_from_event` | Booths | kanban,list,form,graph,pivot | `[('event_id', '=', active_id)]` | `{'default_event_id': active_id, 'search_default_group_by_state': 1}` |  | `event_booth` |

Machine-readable definition: `../../../schemas/data/entities/event.booth.json`; views: `../../../schemas/interfaces/views/event.booth.json`.
