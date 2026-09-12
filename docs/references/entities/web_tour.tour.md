# Tours (`web_tour.tour`)

**Transport name:** `web_tour.tour`  
**Storage name:** `web_tour_tour`  
**Kind:** persistent entity (one table)  
**Defined by package:** `web_tour`

Description: Tours

## Identity and behavior

- Default ordering: `sequence, name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `step_ids` | Step | one to many | `web_tour.tour.step` | inverse field `tour_id` |
| `url` | Starting uniform resource locator | single line text |  | default `/app` |
| `sharing_url` | Sharing uniform resource locator | single line text |  | computed by rule `_compute_sharing_url` (not stored) |
| `rainbow_man_message` | Rainbow Man Message | rich text |  | default `<b>Good job!</b> You went through all steps of this tour.`; translatable |
| `sequence` | Sequence | integer |  | default `1000` |
| `custom` | Custom | boolean |  |  |
| `user_consumed_ids` | User Consumed | many to many | `res.users` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniq_name` | Constraint | `unique(name)` | A tour already exists with this name . Tour's name must be unique! | `web_tour` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_sharing_url` | computation | self | `web_tour` | depends: `name` |  |
| `consume` | operation | self, tourName | `web_tour` | model |  |
| `get_current_tour` | operation | self | `web_tour` | model |  |
| `get_tour_json_by_name` | operation | self, tour_name | `web_tour` | model |  |
| `_get_tour_json` | preparation rule | self | `web_tour` |  |  |
| `export_js_file` | operation | self | `web_tour` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `web_tour` |
| `base.group_user` | no | yes | no | no | `web_tour` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `web_tour.tour_form` | form |  | `name`, `name`, `sequence`, `url`, `custom`, `sharing_url`, `step_ids`, `sequence`, `trigger`, `run`, `tooltip_position`, `content`, `rainbow_man_message` |  |  | `web_tour` |
| `web_tour.tour_list` | list |  | `sequence`, `name`, `url`, `custom`, `rainbow_man_message`, `name` |  |  | `web_tour` |
| `web_tour.tour_search` | search |  | `name` |  |  | `web_tour` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `web_tour.tour_action` | Tours |  |  |  |  | `web_tour` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `web_tour.tour_export_js_action` | Export JS | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/web_tour.tour.json`; views: `../../../schemas/interfaces/views/web_tour.tour.json`.
