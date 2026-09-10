# Tour's step (`web_tour.tour.step`)

**Transport name:** `web_tour.tour.step`  
**Storage name:** `web_tour_tour_step`  
**Kind:** persistent entity (one table)  
**Defined by package:** `web_tour`

Description: Tour's step

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `trigger` | Trigger | single line text |  | required |
| `content` | Content | single line text |  |  |
| `tooltip_position` | Tooltip Position | selection |  | default `bottom` |
| `tour_id` | Tour | many to one | `web_tour.tour` | required; indexed; on delete of the target: cascade |
| `run` | Run | single line text |  |  |
| `sequence` | Sequence | integer |  |  |

## Selection values

### `tooltip_position` (Tooltip Position)

| Value | Label |
|---|---|
| `bottom` | Bottom |
| `top` | Top |
| `right` | Right |
| `left` | left |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_steps_json` | operation | self | `web_tour` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `web_tour` |
| `base.group_user` | no | yes | no | no | `web_tour` |

Machine-readable definition: `../../../schemas/data/entities/web_tour.tour.step.json`.
