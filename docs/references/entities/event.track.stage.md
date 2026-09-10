# Event Track Stage (`event.track.stage`)

**Transport name:** `event.track.stage`  
**Storage name:** `event_track_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`

Description: Event Track Stage

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Stage Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `mail_template_id` | Email Template | many to one | `mail.template` | restricted by domain `[["model", "=", "event.track"]]`; Help: If set an email will be sent to the customer when the track reaches this step. |
| `color` | Color | integer |  |  |
| `description` | Description | multi line text |  | translatable |
| `legend_blocked` | Red Kanban Label | single line text |  | default computed dynamically (lambda s: s.env._('Blocked')); translatable |
| `legend_done` | Green Kanban Label | single line text |  | default computed dynamically (lambda s: s.env._('Ready for Next Stage')); translatable |
| `legend_normal` | Grey Kanban Label | single line text |  | default computed dynamically (lambda s: s.env._('In Progress')); translatable |
| `fold` | Folded in Kanban | boolean |  | Help: This stage is folded in the kanban view when there are no records in that stage to display. |
| `is_visible_in_agenda` | Visible in agenda | boolean |  | computed by rule `_compute_is_visible_in_agenda` and stored; Help: If checked, the related tracks will be visible in the frontend. |
| `is_fully_accessible` | Fully accessible | boolean |  | computed by rule `_compute_is_fully_accessible` and stored; Help: If checked, automatically publish tracks so that access links to customers are provided. |
| `is_cancel` | Cancelled Stage | boolean |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_visible_in_agenda` | computation | self | `website_event_track` | depends: `is_cancel`, `is_fully_accessible` |  |
| `_compute_is_fully_accessible` | computation | self | `website_event_track` | depends: `is_cancel`, `is_visible_in_agenda` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_event_track` |
| `base.group_portal` | no | yes | no | no | `website_event_track` |
| `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_stage_view_search` | search |  | `name` |  |  | `website_event_track` |
| `website_event_track.event_track_stage_view_form` | form |  | `name`, `mail_template_id`, `is_visible_in_agenda`, `is_fully_accessible`, `is_cancel`, `fold`, `color`, `legend_normal`, `legend_blocked`, `legend_done`, `description` |  |  | `website_event_track` |
| `website_event_track.event_track_stage_view_tree` | list |  | `sequence`, `name`, `is_visible_in_agenda`, `is_fully_accessible`, `is_cancel`, `fold` |  |  | `website_event_track` |
| `website_event_track.view_event_track_stage_kanban` | kanban |  | `name` |  |  | `website_event_track` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_stage_action` | Track Stages | list,kanban,form |  |  |  | `website_event_track` |

Machine-readable definition: `../../../schemas/data/entities/event.track.stage.json`; views: `../../../schemas/interfaces/views/event.track.stage.json`.
