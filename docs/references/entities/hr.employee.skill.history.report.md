# Employee Skills Report (`hr.employee.skill.history.report`)

**Transport name:** `hr.employee.skill.history.report`  
**Storage name:** `hr_employee_skill_history_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Employee Skills Report

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | read only |
| `date` | Date | date |  |  |
| `skill_id` | Skill | many to one | `hr.skill` | read only |
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | read only |
| `level_progress` | Level Progress | float |  | read only |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | no | yes | no | no | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee Skill History Report: HR user | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Employee Skill History Report: employee's manager | `[(4, ref('base.group_user'))]` | `[('employee_id', 'child_of', user.employee_ids.ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_skill_history_report_view_graph` | graph |  | `date`, `skill_id`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_history_report_view_search` | search |  | `skill_id`, `skill_type_id` |  | `Skill`, `Skill Type`, `Date` | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.skill.history.report.json`; views: `../../../schemas/interfaces/views/hr.employee.skill.history.report.json`.
