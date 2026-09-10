# Skill level for an applicant (`hr.applicant.skill`)

**Transport name:** `hr.applicant.skill`  
**Storage name:** `hr_applicant_skill`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment_skills`

Description: Skill level for an applicant

## Identity and behavior

- Mixins (classical inheritance): `hr.individual.skill.mixin`
- Default ordering: `skill_type_id, skill_level_id desc`
- Display name field: `skill_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `applicant_id` | Applicant | many to one | `hr.applicant` | required; indexed; on delete of the target: cascade |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_linked_field_name` | internal rule | self | `hr_recruitment_skills` |  |  |
| `_get_current_skills_by_applicant` | preparation rule | self | `hr_recruitment_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_interviewer` | yes | yes | yes | yes | `hr_recruitment_skills` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Applicant Skill: Interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[             '\|',                 ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ('applicant_id.interviewer_ids', 'in', user.id),         ]` | True | True | True | True |
| Applicant Skill: Officer | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment_skills.hr_applicant_skill_view_form` | form |  | `applicant_id`, `skill_type_id`, `skill_id`, `skill_level_id`, `valid_from`, `valid_to` |  |  | `hr_recruitment_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.applicant.skill.json`; views: `../../../schemas/interfaces/views/hr.applicant.skill.json`.
