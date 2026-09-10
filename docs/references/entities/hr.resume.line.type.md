# Type of a resume line (`hr.resume.line.type`)

**Transport name:** `hr.resume.line.type`  
**Storage name:** `hr_resume_line_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Type of a resume line

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `is_course` | Course | boolean |  | default  |
| `resume_line_type_properties_definition` | Sections Properties | properties definition |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_resume_line_type_tree_view` | list |  | `sequence`, `name`, `is_course` |  |  | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_resume_type_action` | Resume Sections | list,form |  |  |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.resume.line.type.json`; views: `../../../schemas/interfaces/views/hr.resume.line.type.json`.
