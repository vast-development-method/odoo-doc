# Department (`hr.department`)

**Transport name:** `hr.department`  
**Storage name:** `hr_department`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_expense`, `hr_holidays`, `hr_recruitment`, `website_hr_recruitment`

Description: Department

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `name`
- Display name field: `complete_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (25)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Department Name | single line text |  | required; translatable |
| `complete_name` | Complete Name | single line text |  | computed by rule `_compute_complete_name` (not stored); searchable through a search rule; recursive dependency |
| `active` | Active | boolean |  | default `True` |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; default computed dynamically (lambda self: self.env.company); changes are tracked in the message thread; indexed; recursive dependency |
| `parent_id` | Parent Department | many to one | `hr.department` | indexed; must belong to the same company |
| `child_ids` | Child Departments | one to many | `hr.department` | inverse field `parent_id` |
| `manager_id` | Manager | many to one | `hr.employee` | changes are tracked in the message thread; restricted by domain `['\|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]` |
| `member_ids` | Members | one to many | `hr.employee` | read only; inverse field `department_id` |
| `has_read_access` | Has Read Access | boolean |  | searchable through a search rule |
| `total_employee` | Total Employee | integer |  | computed by rule `_compute_total_employee` (not stored) |
| `jobs_ids` | Jobs | one to many | `hr.job` | inverse field `department_id` |
| `plan_ids` | Plan | one to many | `mail.activity.plan` | inverse field `department_id` |
| `plans_count` | Plans Count | integer |  | computed by rule `_compute_plan_count` (not stored) |
| `note` | Note | multi line text |  |  |
| `color` | Color Index | integer |  |  |
| `parent_path` | Parent Path | single line text |  | indexed |
| `master_department_id` | Master Department | many to one | `hr.department` | computed by rule `_compute_master_department_id` and stored |
| `expenses_to_approve_count` | Expenses to Approve | integer |  | computed by rule `_compute_expenses_to_approve_count` (not stored) |
| `absence_of_today` | Absence by Today | integer |  | computed by rule `_compute_leave_count` (not stored) |
| `leave_to_approve_count` | Time Off to Approve | integer |  | computed by rule `_compute_leave_count` (not stored) |
| `allocation_to_approve_count` | Allocation to Approve | integer |  | computed by rule `_compute_leave_count` (not stored) |
| `new_applicant_count` | New Applicant | integer |  | computed by rule `_compute_new_applicant_count` (not stored) |
| `new_hired_employee` | New Hired Employee | integer |  | computed by rule `_compute_recruitment_stats` (not stored) |
| `expected_employee` | Expected Employee | integer |  | computed by rule `_compute_recruitment_stats` (not stored) |
| `display_name` | Display Name | single line text |  |  |

## Operations (26)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `hr` | depends_context: `hierarchical_naming` |  |
| `_search_has_read_access` | search rule | self, operator, value | `hr` |  |  |
| `_search_complete_name` | search rule | self, operator, value | `hr` |  |  |
| `name_create` | lifecycle override | self, name | `hr` | model |  |
| `_compute_complete_name` | computation | self | `hr` | depends: `name`, `parent_id.complete_name` |  |
| `_compute_master_department_id` | computation | self | `hr` | depends: `parent_path` |  |
| `_compute_total_employee` | computation | self | `hr` |  |  |
| `_compute_plan_count` | computation | self | `hr` |  |  |
| `_check_parent_id` | validation | self | `hr` | constrains: `parent_id` |  |
| `create` | lifecycle override | self, vals_list | `hr` | model_create_multi |  |
| `_compute_company_id` | computation | self | `hr` | depends: `parent_id`, `parent_id.company_id` |  |
| `write` | lifecycle override | self, vals | `hr` |  | If updating manager of a department, we need to update all the employees of department hierarchy, and subscribe the new manager. |
| `_update_employee_manager` | internal rule | self, manager_id | `hr` |  |  |
| `get_formview_action` | operation | self, access_uid | `hr` |  |  |
| `action_plan_from_department` | user action | self | `hr` |  |  |
| `action_employee_from_department` | user action | self | `hr` |  |  |
| `get_children_department_ids` | operation | self | `hr` |  |  |
| `action_open_view_child_departments` | user action | self | `hr` |  |  |
| `get_department_hierarchy` | operation | self | `hr` |  |  |
| `_compute_expenses_to_approve_count` | computation | self | `hr_expense` |  |  |
| `_compute_leave_count` | computation | self | `hr_holidays` |  |  |
| `_get_action_context` | preparation rule | self | `hr_holidays` |  |  |
| `action_open_leave_department` | user action | self | `hr_holidays` |  |  |
| `action_open_allocation_department` | user action | self | `hr_holidays` |  |  |
| `_compute_new_applicant_count` | computation | self | `hr_recruitment` |  |  |
| `_compute_recruitment_stats` | computation | self | `hr_recruitment` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_parent_id` | ValidationError | You cannot create recursive departments. | `hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |
| `base.group_user` | no | yes | no | no | `hr` |
| `base.group_public` | no | yes | no | no | `website_hr_recruitment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Department multi company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Job department: Public | `[(4, ref('base.group_public'))]` | `['\|', ('jobs_ids.website_published', '=', True), ('child_ids', 'not in', [])]` | True | False | False | False |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.view_department_form` | form |  | `company_id`, `total_employee`, `plans_count`, `active`, `name`, `name`, `manager_id`, `parent_id`, `child_ids`, `company_id`, `color` | `action_employee_from_department`, `action_plan_from_department` |  | `hr` |
| `hr.view_department_tree` | list |  | `company_id`, `name`, `company_id`, `manager_id`, `total_employee`, `parent_id`, `color` |  |  | `hr` |
| `hr.view_department_filter` | search |  | `name`, `manager_id` |  | `Unread Messages`, `Archived` | `hr` |
| `hr.hr_department_view_kanban` | kanban |  | `active`, `color`, `name`, `manager_id`, `company_id`, `total_employee` | `action_employee_from_department` |  | `hr` |
| `hr_attendance.hr_department_view_kanban` | data | `hr.hr_department_view_kanban` |  |  |  | `hr_attendance` |
| `hr_expense.hr_department_view_kanban` | data | `hr.hr_department_view_kanban` | `expenses_to_approve_count` |  |  | `hr_expense` |
| `hr_holidays.hr_department_view_kanban` | data | `hr.hr_department_view_kanban` | `total_employee`, `absence_of_today`, `leave_to_approve_count`, `allocation_to_approve_count`, `absence_of_today` |  |  | `hr_holidays` |
| `hr_org_chart.hr_department_hierarchy_view` | hierarchy |  | `name`, `color`, `total_employee`, `name`, `manager_id` | `action_employee_from_department` |  | `hr_org_chart` |
| `hr_recruitment.hr_department_view_kanban` | data | `hr.hr_department_view_kanban` | `new_applicant_count` |  |  | `hr_recruitment` |
| `hr_skills.hr_department_view_kanban` | xpath | `hr.hr_department_view_kanban` |  |  |  | `hr_skills` |
| `hr_timesheet.hr_department_view_kanban` | data | `hr.hr_department_view_kanban` |  |  |  | `hr_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_department_tree_action` | Departments | list,form,kanban | `[("has_read_access", "=", True)]` |  |  | `hr` |
| `hr.hr_department_kanban_action` | Departments | kanban,list,form | `[("has_read_access", "=", True)]` |  |  | `hr` |
| `hr_recruitment.action_hr_department` | Departments | list,form |  |  |  | `hr_recruitment` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_skills.action_open_skills_log_department` | Skill History Report | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/hr.department.json`; views: `../../../schemas/interfaces/views/hr.department.json`.
