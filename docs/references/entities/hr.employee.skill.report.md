# Employee Skills Report (`hr.employee.skill.report`)

**Transport name:** `hr.employee.skill.report`  
**Storage name:** `hr_employee_skill_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_skills`

Description: Employee Skills Report

## Identity and behavior

- Mixins (classical inheritance): `hr.manager.department.report`
- Default ordering: `employee_id, level_progress desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | read only |
| `department_id` | Department | many to one | `hr.department` | read only |
| `job_id` | Job | many to one | `hr.job` | read only |
| `skill_id` | Skill | many to one | `hr.skill` | read only |
| `skill_type_id` | Skill Type | many to one | `hr.skill.type` | read only |
| `skill_level` | Skill Level | single line text |  | read only |
| `level_progress` | Level Progress | float |  | read only; aggregated with avg |
| `active` | Active | boolean |  | related through path `employee_id.active` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `hr_skills` |  |  |
| `formatted_read_grouping_sets` | operation | self, domain, grouping_sets, aggregates, order | `hr_skills` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | no | yes | no | no | `hr_skills` |
| `base.group_user` | no | yes | no | no | `hr_skills` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Employee Skill Report: HR user | `[(4, ref('hr.group_hr_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Employee Skill Report: employee's manager | `[(4, ref('base.group_user'))]` | `[('has_department_manager_access', '=', True)]` | True | True | True | True |
| Employee Skill Report: Multi-Company Rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_skill_report_view_pivot` | pivot |  | `department_id`, `employee_id`, `skill_type_id`, `skill_id`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_graph` | graph |  | `employee_id`, `skill_type_id`, `skill_id`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_list` | list |  | `employee_id`, `skill_type_id`, `skill_id`, `skill_level`, `level_progress` |  |  | `hr_skills` |
| `hr_skills.hr_employee_skill_report_view_search` | search |  | `employee_id`, `department_id`, `skill_id`, `skill_type_id` |  | `Archived`, `Employee`, `Department`, `Jobs`, `Skill Type`, `Skill` | `hr_skills` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_skills.hr_employee_skill_report_action` | Skills Inventory | list,pivot |  | `{             'search_default_skill_type': 1,             'search_default_skill': 2,         }` |  | `hr_skills` |
| `hr_skills.action_hr_employee_skill_log_department` | Skill History Report | graph,pivot,list |  | `{'fill_temporal': 0, 'search_default_group_by_skill_type_id': 1, 'search_default_group_by_skill_id': 2}` | current | `hr_skills` |

Machine-readable definition: `../../../schemas/data/entities/hr.employee.skill.report.json`; views: `../../../schemas/interfaces/views/hr.employee.skill.report.json`.
