# Event Track Tag Category (`event.track.tag.category`)

**Transport name:** `event.track.tag.category`  
**Storage name:** `event_track_tag_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`

Description: Event Track Tag Category

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `tag_ids` | Tags | one to many | `event.track.tag` | inverse field `category_id` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `event.group_event_user` | yes | yes | yes | yes | `website_event_track` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_tag_category_view_form` | form |  | `name`, `tag_ids`, `sequence`, `name`, `color` |  |  | `website_event_track` |
| `website_event_track.event_track_tag_category_view_list` | list |  | `sequence`, `name`, `tag_ids` |  |  | `website_event_track` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_tag_category_action` | Track Tag Categories | list,form |  |  |  | `website_event_track` |

Machine-readable definition: `../../../schemas/data/entities/event.track.tag.category.json`; views: `../../../schemas/interfaces/views/event.track.tag.category.json`.
