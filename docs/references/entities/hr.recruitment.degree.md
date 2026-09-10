# Applicant Degree (`hr.recruitment.degree`)

**Transport name:** `hr.recruitment.degree`  
**Storage name:** `hr_recruitment_degree`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Applicant Degree

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Degree Name | single line text |  | required; translatable |
| `score` | Score | float |  | required; default  |
| `sequence` | Sequence | integer |  | default `1` |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | The name of the Degree of Recruitment must be unique! | `hr_recruitment` |
| `_score_range` | Constraint | `check(score >= 0 and score <= 1)` | Score should be between 0 and 100% | `hr_recruitment` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_degree_tree` | list |  | `sequence`, `name`, `score` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_recruitment_degree_form` | form |  | `name`, `score`, `sequence` |  |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_recruitment_degree_action` | Degrees |  |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.recruitment.degree.json`; views: `../../../schemas/interfaces/views/hr.recruitment.degree.json`.
