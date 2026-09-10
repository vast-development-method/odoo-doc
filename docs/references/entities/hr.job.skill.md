# Skills for job positions (`hr.job.skill`)

**Transport name:** `hr.job.skill`  
**Storage name:** `hr_job_skill`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Skills for job positions

## Identity and behavior

- Mixins (classical inheritance): `hr.individual.skill.mixin`
- Default ordering: `skill_type_id, skill_level_id desc`
- Display name field: `skill_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `job_id` | Job | many to one | `hr.job` | required; indexed; on delete of the target: cascade |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_linked_field_name` | internal rule | self | `hr_skills` |  |  |
| `_can_edit_certification_validity_period` | internal rule | self | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment_skills` |
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_job_skill_view_form` | form |  | `job_id`, `skill_type_id`, `skill_id`, `skill_level_id` |  |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.job.skill.json`; views: `../../../schemas/interfaces/views/hr.job.skill.json`.
