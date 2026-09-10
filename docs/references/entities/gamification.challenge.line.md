# Gamification generic goal for challenge (`gamification.challenge.line`)

**Transport name:** `gamification.challenge.line`  
**Storage name:** `gamification_challenge_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`

Description: Gamification generic goal for challenge

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `challenge_id` | Challenge | many to one | `gamification.challenge` | required; indexed; on delete of the target: cascade |
| `definition_id` | Goal Definition | many to one | `gamification.goal.definition` | required; on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `1` |
| `target_goal` | Target Value to Reach | float |  | required |
| `name` | Name | single line text |  | related through path `definition_id.name` |
| `condition` | Condition | selection |  | read only; related through path `definition_id.condition` |
| `definition_suffix` | Unit | single line text |  | read only; related through path `definition_id.suffix` |
| `definition_monetary` | Monetary | boolean |  | read only; related through path `definition_id.monetary` |
| `definition_full_suffix` | Suffix | single line text |  | read only; related through path `definition_id.full_suffix` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | no | yes | no | no | `gamification` |
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_gamification` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.challenge_line_list_view` | list |  | `definition_id`, `target_goal` |  |  | `gamification` |

Machine-readable definition: `../../../schemas/data/entities/gamification.challenge.line.json`; views: `../../../schemas/interfaces/views/gamification.challenge.line.json`.
