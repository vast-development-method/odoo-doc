# Mandatory Day (`hr.leave.mandatory.day`)

**Transport name:** `hr.leave.mandatory.day`  
**Storage name:** `hr_leave_mandatory_day`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`

Description: Mandatory Day

## Identity and behavior

- Default ordering: `start_date desc, end_date desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `start_date` | Start Date | date |  | required |
| `end_date` | End Date | date |  | required |
| `color` | Color | integer |  | default computed dynamically (lambda dummy: randint(1, 11)) |
| `resource_calendar_id` | Working Hours | many to one | `resource.calendar` | restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `department_ids` | Departments | many to many | `hr.department` |  |
| `job_ids` | Job Position | many to many | `hr.job` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_date_from_after_day_to` | Constraint | `CHECK(start_date <= end_date)` | The start date must be anterior than the end date. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `hr_holidays` |
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Mandatory Day: multi company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_mandatory_day_view_form` | form |  | `name`, `start_date`, `end_date`, `color`, `company_id` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_mandatory_day_view_list` | list |  | `company_id`, `name`, `company_id`, `department_ids`, `job_ids`, `start_date`, `end_date`, `color` |  |  | `hr_holidays` |
| `hr_holidays.hr_leave_mandatory_day_view_search` | search |  | `name`, `start_date`, `end_date`, `company_id` |  | `Period`, `Department`, `Job`, `Company` | `hr_holidays` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_leave_mandatory_day_action` | Mandatory Days | list,form |  | `{'search_default_filter_date': True}` |  | `hr_holidays` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.mandatory.day.json`; views: `../../../schemas/interfaces/views/hr.leave.mandatory.day.json`.
