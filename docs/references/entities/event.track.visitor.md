# Track / Visitor Link (`event.track.visitor`)

**Transport name:** `event.track.visitor`  
**Storage name:** `event_track_visitor`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`  
**Extended by packages:** `website_event_track_quiz`

Description: Track / Visitor Link

## Identity and behavior

- Default ordering: `track_id`
- Display name field: `track_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; indexed; on delete of the target: set null |
| `visitor_id` | Visitor | many to one | `website.visitor` | indexed; on delete of the target: cascade |
| `track_id` | Track | many to one | `event.track` | required; indexed; on delete of the target: cascade |
| `is_wishlisted` | Is Wishlisted | boolean |  |  |
| `is_blacklisted` | Is reminder off | boolean |  | Help: As key track cannot be un-favorited, this field store the partner choice to remove the reminder for key tracks. |
| `quiz_completed` | Completed | boolean |  |  |
| `quiz_points` | Quiz Points | integer |  | default  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_partner_id` | computation | self | `website_event_track` | depends: `visitor_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `website_event_track` |
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |

## Views (6)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_visitor_view_search` | search |  | `track_id`, `visitor_id`, `partner_id`, `is_wishlisted` |  | `Track`, `Visitor`, `Customer` | `website_event_track` |
| `website_event_track.event_track_visitor_view_form` | form |  | `track_id`, `is_wishlisted`, `visitor_id`, `partner_id` |  |  | `website_event_track` |
| `website_event_track.event_track_visitor_view_list` | list |  | `track_id`, `visitor_id`, `partner_id`, `is_wishlisted` |  |  | `website_event_track` |
| `website_event_track_quiz.event_track_visitor_view_search` | xpath | `website_event_track.event_track_visitor_view_search` | `quiz_completed` |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_track_visitor_view_form` | xpath | `website_event_track.event_track_visitor_view_form` | `quiz_completed`, `quiz_points` |  |  | `website_event_track_quiz` |
| `website_event_track_quiz.event_track_visitor_view_list` | xpath | `website_event_track.event_track_visitor_view_list` | `quiz_completed`, `quiz_points` |  |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_visitor_action` | Track Visitors | list,form |  | `{'create': False}` |  | `website_event_track` |

Machine-readable definition: `../../../schemas/data/entities/event.track.visitor.json`; views: `../../../schemas/interfaces/views/event.track.visitor.json`.
