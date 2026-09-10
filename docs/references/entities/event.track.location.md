# Event Track Location (`event.track.location`)

**Transport name:** `event.track.location`  
**Storage name:** `event_track_location`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`

Description: Event Track Location

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Location | single line text |  | required |
| `sequence` | Sequence | integer |  | default `10`; Help: Define the order in which the location will appear on "Agenda" page |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.view_event_location_form` | form |  | `name`, `sequence` |  |  | `website_event_track` |
| `website_event_track.view_event_location_tree` | list |  | `sequence`, `name` |  |  | `website_event_track` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.action_event_track_location` | Event Locations |  |  |  |  | `website_event_track` |

Machine-readable definition: `../../../schemas/data/entities/event.track.location.json`; views: `../../../schemas/interfaces/views/event.track.location.json`.
