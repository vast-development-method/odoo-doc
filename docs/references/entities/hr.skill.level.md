# Skill Level (`hr.skill.level`)

**Transport name:** `hr.skill.level`  
**Storage name:** `hr_skill_level`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Skill Level

## Identity and behavior

- Default ordering: `level_progress`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | indexed (btree_not_null); on delete of the target: cascade |
| `name` | Name | single line text |  | required |
| `level_progress` | Progress | integer |  | Help: Progress from zero knowledge (0%) to fully mastered (100%). |
| `default_level` | Default Level | boolean |  | Help: If checked, this level will be the default one selected when choosing this skill. |
| `technical_is_new_default` | Technical Is New Default | boolean |  | computed by rule `_compute_technical_is_new_default` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_level_progress` | Constraint | `CHECK(level_progress BETWEEN 0 AND 100)` | Progress should be a number between 0 and 100. | `hr_skills` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_technical_is_new_default` | computation | self | `hr_skills` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_skills` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.employee_skill_level_view_tree` | list |  | `name`, `level_progress`, `default_level`, `technical_is_new_default` |  |  | `hr_skills` |
| `hr_skills.employee_skill_level_view_form` | form |  | `name`, `level_progress` |  |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.skill.level.json`; views: `../../../schemas/interfaces/views/hr.skill.level.json`.
