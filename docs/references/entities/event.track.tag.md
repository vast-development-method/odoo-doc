# Event Track Tag (`event.track.tag`)

**Transport name:** `event.track.tag`  
**Storage name:** `event_track_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`

Description: Event Track Tag

## Identity and behavior

- Default ordering: `category_id, sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required |
| `track_ids` | Tracks | many to many | `event.track` |  |
| `color` | Color Index | integer |  | default computed dynamically (lambda self: self._default_color()); Help: Note that colorless tags won't be available on the website. |
| `sequence` | Sequence | integer |  | default `10` |
| `category_id` | Category | many to one | `event.track.tag.category` | indexed (btree_not_null); on delete of the target: set null |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `website_event_track` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_color` | preparation rule | self | `website_event_track` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_event_track` |
| `base.group_portal` | no | yes | no | no | `website_event_track` |
| `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Track Tag: public/portal: color = published | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `['&', ('color', '!=', False), ('color', '!=', 0)]` | True | False | False | False |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.view_event_track_tag_form` | form |  | `name`, `color`, `category_id` |  |  | `website_event_track` |
| `website_event_track.view_event_track_tag_tree` | list |  | `sequence`, `name`, `category_id`, `color` |  |  | `website_event_track` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.action_event_track_tag` | Track Tags |  |  |  |  | `website_event_track` |

Machine-readable definition: `../../../schemas/data/entities/event.track.tag.json`; views: `../../../schemas/interfaces/views/event.track.tag.json`.
