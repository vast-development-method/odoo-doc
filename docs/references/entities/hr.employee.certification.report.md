# Employee Certification Report (`hr.employee.certification.report`)

**Transport name:** `hr.employee.certification.report`  
**Storage name:** `hr_employee_certification_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Employee Certification Report

## Identity and behavior

- Mixins (classical inheritance): `hr.manager.department.report`
- Default ordering: `employee_id, level_progress desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `skill_id` | Skill | many to one | `hr.skill` | read only |
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | read only |
| `skill_level` | Skill Level | single line text |  | read only |
| `level_progress` | Level Progress | float |  | read only; aggregated with avg |
| `active` | Active | boolean |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_skills` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_certification_report_view_pivot` | pivot |  | `employee_id`, `skill_type_id`, `skill_id`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_view_list` | list |  | `employee_id`, `skill_type_id`, `skill_id`, `skill_level`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_certification_report_view_search` | search |  | `employee_id`, `department_id`, `skill_id`, `skill_type_id` |  | `Expired Certification`, `Employee`, `Department`, `Certification Type`, `Certification` | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_certification_report_action` | Certification | list,pivot |  | `{             'search_default_employee': 1,         }` |  | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.certification.report.json`; views: `../../../schemas/interfaces/views/hr.employee.certification.report.json`.
