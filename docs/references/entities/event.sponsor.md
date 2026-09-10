# Event Sponsor (`event.sponsor`)

**Transport name:** `event.sponsor`  
**Storage name:** `event_sponsor`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_exhibitor`

Description: Event Sponsor

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `website.published.mixin`, `website.searchable.mixin`
- Default ordering: `sequence, sponsor_type_id`
- Display name field: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `event_id` | Event | many to one | `event.event` | required; indexed |
| `sponsor_type_id` | Sponsorship Level | many to one | `event.sponsor.type` | required; default computed dynamically (lambda self: self._default_sponsor_type_id()) |
| `url` | Sponsor Website | single line text |  | computed by rule `_compute_url` and stored |
| `sequence` | Sequence | integer |  |  |
| `active` | Active | boolean |  | default `True` |
| `subtitle` | Slogan | single line text |  |  |
| `exhibitor_type` | Sponsor Type | selection |  | default `sponsor` |
| `website_description` | Description | rich text |  | computed by rule `_compute_website_description` and stored; translatable |
| `show_on_ticket` | Show on ticket | boolean |  | default `True` |
| `partner_id` | Partner | many to one | `res.partner` | required |
| `partner_name` | Name | single line text |  | related through path `partner_id.name` |
| `partner_email` | Email | single line text |  | related through path `partner_id.email` |
| `partner_phone` | Phone | single line text |  | related through path `partner_id.phone` |
| `name` | Sponsor Name | single line text |  | computed by rule `_compute_name` and stored |
| `email` | Sponsor Email | single line text |  | computed by rule `_compute_email` and stored |
| `phone` | Sponsor Phone | single line text |  | computed by rule `_compute_phone` and stored |
| `image_512` | Logo | image |  | computed by rule `_compute_image_512` and stored |
| `image_256` | Image 256 | image |  | related through path `image_512` |
| `image_128` | Image 128 | image |  | related through path `image_512` |
| `website_image_url` | Image uniform resource locator | single line text |  | computed by rule `_compute_website_image_url` (not stored) |
| `hour_from` | Opening hour | float |  | default `8.0` |
| `hour_to` | End hour | float |  | default `18.0` |
| `event_date_tz` | Timezone | selection |  | read only; related through path `event_id.date_tz` |
| `is_in_opening_hours` | Within opening hours | boolean |  | computed by rule `_compute_is_in_opening_hours` (not stored) |
| `country_id` | Country | many to one | `res.country` | read only; related through path `partner_id.country_id` |
| `country_flag_url` | Country Flag | single line text |  | computed by rule `_compute_country_flag_url` (not stored) |

## Selection values

### `exhibitor_type` (Sponsor Type)

| Value | Label |
|---|---|
| `sponsor` | Footer Logo Only |
| `exhibitor` | Exhibitor |
| `online` | Online Exhibitor |

## Operations (16)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_sponsor_type_id` | preparation rule | self | `website_event_exhibitor` |  |  |
| `_compute_url` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_name` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_email` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_phone` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_image_512` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_website_image_url` | computation | self | `website_event_exhibitor` | depends: `image_512`, `partner_id.image_256` |  |
| `_synchronize_with_partner` | internal rule | self, fname | `website_event_exhibitor` |  | Synchronize with partner if not set. Setting a value does not write on partner as this may be event-specific information. |
| `_compute_website_description` | computation | self | `website_event_exhibitor` | depends: `partner_id` |  |
| `_compute_is_in_opening_hours` | computation | self | `website_event_exhibitor` | depends: `event_id.is_ongoing`, `hour_from`, `hour_to`, `event_id.date_begin`, `event_id.date_end` | Opening hours: hour_from and hour_to are given within event TZ or UTC. Now() must therefore be computed based on that TZ. |
| `_compute_country_flag_url` | computation | self | `website_event_exhibitor` | depends: `partner_id.country_id.image_url` |  |
| `_compute_website_url` | computation | self | `website_event_exhibitor` | depends: `name`, `event_id.name` |  |
| `_compute_website_absolute_url` | computation | self | `website_event_exhibitor` | depends: `event_id.website_id.domain` |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_event_exhibitor` | model |  |
| `get_backend_menu_id` | operation | self | `website_event_exhibitor` |  |  |
| `get_base_url` | operation | self | `website_event_exhibitor` |  | As website_id is not defined on this record, we rely on event website_id for base URL. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_exhibitor` |
| `base.group_public` | no | yes | no | no | `website_event_exhibitor` |
| `base.group_portal` | no | yes | no | no | `website_event_exhibitor` |
| `base.group_user` | no | yes | no | no | `website_event_exhibitor` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Sponsor: public/portal sponsor or published only | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | False | False | False |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_view_search` | search |  | `partner_id`, `event_id`, `name`, `email`, `phone` |  | `Published`, `Archived`, `Exhibitors`, `Online`, `Event`, `Level` | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_form` | form |  | `website_url`, `is_published`, `active`, `image_512`, `name`, `subtitle`, `partner_id`, `email`, `phone`, `url`, `event_id`, `sponsor_type_id`, `exhibitor_type`, `website_published`, `hour_from`, `hour_to`, `event_date_tz`, `show_on_ticket`, `website_description` |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_tree` | list |  | `sequence`, `partner_id`, `name`, `email`, `phone`, `url`, `sponsor_type_id`, `is_published`, `exhibitor_type`, `show_on_ticket` |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_view_kanban` | kanban |  | `image_128`, `name`, `sponsor_type_id`, `exhibitor_type`, `partner_email`, `url` |  |  | `website_event_exhibitor` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_exhibitor.event_sponsor_action` | Event Sponsors | kanban,list,form |  |  |  | `website_event_exhibitor` |
| `website_event_exhibitor.event_sponsor_action_from_event` | Event Sponsors | kanban,list,form |  | `{'search_default_event_id': active_id, 'default_event_id': active_id}` |  | `website_event_exhibitor` |

Machine-readable definition: `../../../schemas/data/entities/event.sponsor.json`; views: `../../../schemas/interfaces/views/event.sponsor.json`.
