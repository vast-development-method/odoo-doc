# Event Template (`event.type`)

**Transport name:** `event.type`  
**Storage name:** `event_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `event`  
**Extended by packages:** `event_booth`, `website_event`, `website_event_track`, `website_event_booth`, `website_event_exhibitor`

Description: Event Template

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Event Template | single line text |  | required; translatable |
| `note` | Note | rich text |  |  |
| `sequence` | Sequence | integer |  | default `10` |
| `event_type_ticket_ids` | Tickets | one to many | `event.type.ticket` | inverse field `event_type_id` |
| `tag_ids` | Tags | many to many | `event.tag` |  |
| `has_seats_limitation` | Limited Seats | boolean |  |  |
| `seats_max` | Maximum Registrations | integer |  | computed by rule `_compute_seats_max` and stored; Help: It will select this default maximum value when you choose this event |
| `default_timezone` | Timezone | selection |  | default computed dynamically (lambda self: self.env.user.tz or 'UTC') |
| `event_type_mail_ids` | Mail Schedule | one to many | `event.type.mail` | default computed dynamically (_default_event_mail_type_ids); inverse field `event_type_id` |
| `ticket_instructions` | Ticket Instructions | rich text |  | translatable; Help: This information will be printed on your tickets. |
| `question_ids` | Questions | many to many | `event.question` | default computed dynamically (_default_question_ids) |
| `event_type_booth_ids` | Booths | one to many | `event.type.booth` | inverse field `event_type_id` |
| `website_menu` | Display a dedicated menu on Website | boolean |  |  |
| `community_menu` | Community Menu | boolean |  | computed by rule `_compute_community_menu` and stored; Help: Display community tab on website |
| `website_track` | Tracks on Website | boolean |  | computed by rule `_compute_website_track_menu_data` and stored |
| `website_track_proposal` | Tracks Proposals on Website | boolean |  | computed by rule `_compute_website_track_menu_data` and stored |
| `booth_menu` | Booths on Website | boolean |  | computed by rule `_compute_booth_menu` and stored |
| `exhibitor_menu` | Showcase Exhibitors | boolean |  | computed by rule `_compute_exhibitor_menu` and stored; Help: Display exhibitors on website, in the footer of every page of the event. |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_event_mail_type_ids` | preparation rule | self | `event` |  |  |
| `_default_question_ids` | preparation rule | self | `event` |  |  |
| `_compute_seats_max` | computation | self | `event` | depends: `has_seats_limitation` |  |
| `_compute_community_menu` | computation | self | `website_event` | depends: `website_menu` |  |
| `_compute_website_track_menu_data` | computation | self | `website_event_track` | depends: `website_menu` | Simply activate or de-activate all menus at once. |
| `_compute_booth_menu` | computation | self | `website_event_booth` | depends: `website_menu` |  |
| `_compute_exhibitor_menu` | computation | self | `website_event_exhibitor` | depends: `website_menu` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_registration_desk` | no | yes | no | no | `event` |
| `event.group_event_manager` | yes | yes | yes | yes | `event` |
| all internal users | no | no | no | no | `website_event` |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `event.view_event_type_form` | form |  | `name`, `default_timezone`, `tag_ids`, `has_seats_limitation`, `seats_max`, `event_type_ticket_ids`, `event_type_mail_ids`, `template_ref`, `interval_nbr`, `interval_unit`, `interval_type`, `question_ids`, `title`, `is_mandatory_answer`, `once_per_order`, `question_type`, `answer_ids`, `is_default`, `is_reusable`, `note`, `ticket_instructions` |  |  | `event` |
| `event.view_event_type_tree` | list |  | `sequence`, `name` |  |  | `event` |
| `event.event_type_view_search` | search |  | `name` |  |  | `event` |
| `event_booth.event_type_view_form` | page | `event.view_event_type_form` | `event_type_booth_ids` |  |  | `event_booth` |
| `website_event.event_type_view_form` | xpath | `event.view_event_type_form` | `website_menu`, `community_menu` |  |  | `website_event` |
| `website_event_booth.event_type_view_form` | xpath | `website_event.event_type_view_form` | `booth_menu` |  |  | `website_event_booth` |
| `website_event_exhibitor.event_type_view_form` | xpath | `website_event.event_type_view_form` | `exhibitor_menu` |  |  | `website_event_exhibitor` |
| `website_event_track.event_type_view_form_inherit_track` | xpath | `website_event.event_type_view_form` | `website_track`, `website_track_proposal` |  |  | `website_event_track` |
| `website_event_track_quiz.event_type_view_form` | xpath | `website_event.event_type_view_form` |  |  |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `event.action_event_type` | Event Templates |  |  |  |  | `event` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `event.menu_event_type` |  |  | `event.action_event_type` |  |  |

Machine-readable definition: `../../../schemas/data/entities/event.type.json`; views: `../../../schemas/interfaces/views/event.type.json`.
