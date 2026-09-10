# Skill (`hr.skill`)

**Transport name:** `hr.skill`  
**Storage name:** `hr_skill`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Skill

## Identity and behavior

- Default ordering: `sequence, name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | required; indexed; on delete of the target: cascade |
| `color` | Color | integer |  | related through path `skill_type_id.color` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `hr_skills` | depends: `skill_type_id`; depends_context: `from_skill_dropdown` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | yes | yes | no | no | `hr_skills` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.employee_skill_view_tree` | list |  | `name`, `skill_type_id` |  |  | `hr_skills` |
| `hr_skills.hr_skill_view_form` | form |  | `name`, `skill_type_id` |  |  | `hr_skills` |
| `hr_skills.hr_skill_view_search` | search |  | `name`, `skill_type_id` |  | `Skill Type` | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.skill.json`; views: `../../../schemas/interfaces/views/hr.skill.json`.
