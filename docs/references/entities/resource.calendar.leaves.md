# Resource Time Off Detail (`resource.calendar.leaves`)

**Transport name:** `resource.calendar.leaves`  
**Storage name:** `resource_calendar_leaves`  
**Kind:** persistent entity (one table)  
**Defined by package:** `resource`  
**Extended by packages:** `hr`, `hr_holidays`, `hr_holidays_attendance`, `hr_work_entry`, `project_timesheet_holidays`

Description: Resource Time Off Detail

## Identity and behavior

- Default ordering: `date_from`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reason | single line text |  |  |
| `company_id` | Company | many to one | `res.company` | read only; computed by rule `_compute_company_id` and stored; default computed dynamically (lambda self: self.env.company) |
| `calendar_id` | Working Hours | many to one | `resource.calendar` | computed by rule `_compute_calendar_id` and stored; indexed; restricted by domain `[('company_id', 'in', [company_id, False])]`; must belong to the same company |
| `date_from` | Start Date | date and time |  | required |
| `date_to` | End Date | date and time |  | required; computed by rule `_compute_date_to` and stored |
| `resource_id` | Resource | many to one | `resource.resource` | indexed; Help: If empty, this is a generic time off for the company. If a resource is set, the time off is only for this resource |
| `time_type` | Time Type | selection |  | default `leave`; Help: Whether this should be computed as a time off or as work time (eg: formation) |
| `holiday_id` | Time Off Request | many to one | `hr.leave` |  |
| `elligible_for_accrual_rate` | Eligible for Accrual Rate | boolean |  | default ; Help: If checked, this time off type will be taken into account for accruals computation. |
| `work_entry_type_id` | Work Entry Type | many to one | `hr.work.entry.type` | visible only to groups `hr.group_hr_user` |
| `timesheet_ids` | Analytic Lines | one to many | `account.analytic.line` | inverse field `global_leave_id` |

## Selection values

### `time_type` (Time Type)

| Value | Label |
|---|---|
| `leave` | Time Off |
| `other` | Other |

## Operations (26)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `resource` | model |  |
| `_compute_calendar_id` | computation | self | `hr`, `resource` | depends: `resource_id.calendar_id`; depends: `date_from` |  |
| `_compute_company_id` | computation | self | `hr_holidays`, `resource` | depends: `calendar_id` |  |
| `_compute_date_to` | computation | self | `resource` | depends: `date_from` |  |
| `check_dates` | validation | self | `resource` | constrains: `date_from`, `date_to` |  |
| `_copy_leave_vals` | internal rule | self | `hr_work_entry`, `resource` |  |  |
| `_check_compare_dates` | validation | self | `hr_holidays` | constrains: `date_from`, `date_to`, `calendar_id` |  |
| `_get_domain` | preparation rule | self, time_domain_dict | `hr_holidays` |  |  |
| `_get_time_domain_dict` | preparation rule | self | `hr_holidays` |  |  |
| `_reevaluate_leaves` | internal rule | self, time_domain_dict | `hr_holidays` |  |  |
| `_convert_timezone` | internal rule | self, utc_naive_datetime, tz_from, tz_to | `hr_holidays` |  | Convert a naive date to another timezone that initial timezone used to generate the date. :param utc_naive_datetime: utc date without tzinfo :type utc_naive_datetime: datetime :param tz_from: timezone used to obtained `utc_naive_datetime` :param tz_to: timezone in which we want the date :return: datetime converted into tz_to without tzinfo :rtype: datetime |
| `_ensure_datetime` | internal rule | self, datetime_representation, date_format | `hr_holidays` |  | Be sure to get a datetime object if we have the necessary information. :param datetime_reprentation: object which should represent a datetime :rtype: datetime if a correct datetime_represtion, None otherwise |
| `_prepare_public_holidays_values` | preparation rule | self, vals_list | `hr_holidays` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_holidays_attendance`, `hr_holidays`, `project_timesheet_holidays` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_holidays_attendance`, `hr_holidays`, `project_timesheet_holidays` |  |  |
| `unlink` | lifecycle override | self | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_get_attendance_domain` | preparation rule | self | `hr_holidays_attendance` |  |  |
| `_update_attendances_overtime` | internal rule | self, domain | `hr_holidays_attendance` |  |  |
| `_get_resource_calendars` | preparation rule | self | `project_timesheet_holidays` |  |  |
| `_work_time_per_day` | internal rule | self, resource_calendars | `project_timesheet_holidays` |  | Get work time per day based on the calendar and its attendances  1) Gets all calendars with their characteristics (i.e.     (a) the leaves in it,     (b) the resources which have a leave,     (c) the oldest and     (d) the latest leave dates    ) for leaves in self (first for calendar's leaves, then for company's global leaves) 2) Search the attendances based on the characteristics retrieved for each calendar.     The attendances found are the ones between the date_from of the oldest leave     and the date_to of the most recent leave. 3) Create a dict as result of this method containing:     { |
| `_timesheet_create_lines` | internal rule | self | `project_timesheet_holidays` |  | Create timesheet leaves for each employee using the same calendar containing in self.calendar_id  If the employee has already a time off in the same day then no timesheet should be created. |
| `_timesheet_prepare_line_values` | internal rule | self, index, employee_id, work_hours_data, day_date, work_hours_count | `project_timesheet_holidays` |  |  |
| `_generate_timesheeets` | internal rule | self | `project_timesheet_holidays` |  |  |
| `_generate_public_time_off_timesheets` | internal rule | self, employees | `project_timesheet_holidays` |  |  |
| `_get_overlapping_hr_leaves` | preparation rule | self, domain | `project_timesheet_holidays` |  | Find leaves with potentially missing timesheets. |
| `_regenerate_hr_leave_timesheets_on_gto_unlinked` | internal rule | self | `project_timesheet_holidays` | ondelete |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `check_dates` | ValidationError | The start date of the time off must be earlier than the end date. | `resource` |
| `_check_compare_dates` | ValidationError | Two public holidays cannot overlap each other for the same working hours. | `hr_holidays` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.group_mrp_manager` | no | yes | no | no | `mrp` |
| `project.group_project_user` | yes | yes | yes | yes | `project` |
| `base.group_user` | yes | yes | yes | yes | `resource` |
| `base.group_system` | yes | yes | yes | yes | `resource` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Time Off Resources: Approver | `[(4, ref('base.group_user'))]` | `[(1,'=',1)]` | True | False | False | False |
| Time Off Resources: All Approver | `[(4, ref('hr_holidays.group_hr_holidays_user'))]` | `[(1,'=',1)]` | True | True | True | True |
| resource.calendar.leaves: employee reads own or global | `[(4, ref('base.group_user'))]` | `['\|', ('resource_id', '=', False), ('resource_id.user_id', 'in', [False, user.id])]` | True | False | False | False |
| resource.calendar.leaves: employee modifies own | `[(4, ref('base.group_user'))]` | `[('resource_id', '!=', False), ('resource_id.user_id', 'in', [False, user.id])]` | False | True | True | True |
| resource.calendar.leaves: admin modifies global | `[(4, ref('base.group_erp_manager'))]` | `[('resource_id', '=', False)]` | False | True | True | True |
| resource.calendar.leaves: multi-company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.resource_calendar_leaves_view_search_inherit` | filter | `resource.view_resource_calendar_leaves_search` |  |  | `resource` | `hr_holidays` |
| `hr_holidays.resource_calendar_leave_form_inherit` | field | `resource.resource_calendar_leave_form` | `name`, `holiday_id` |  |  | `hr_holidays` |
| `hr_holidays.resource_calendar_leaves_tree_inherit` | xpath | `resource.resource_calendar_leave_tree` |  |  |  | `hr_holidays` |
| `hr_work_entry.resource_calendar_leaves_view_search_inherit` | filter | `resource.view_resource_calendar_leaves_search` | `work_entry_type_id` |  | `resource`, `Work Entry Type` | `hr_work_entry` |
| `hr_work_entry.resource_calendar_leave_view_form` | field | `resource.resource_calendar_leave_form` | `resource_id`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `hr_work_entry.resource_calendar_leave_view_tree` | field | `resource.resource_calendar_leave_tree` | `date_to`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `resource.view_resource_calendar_leaves_search` | search |  | `name`, `resource_id`, `company_id`, `calendar_id` |  | `Period`, `Resource`, `Company`, `Date` | `resource` |
| `resource.view_resource_calendar` | calendar |  | `resource_id`, `company_id`, `name` |  |  | `resource` |
| `resource.resource_calendar_leave_form` | form |  | `company_id`, `name`, `calendar_id`, `company_id`, `resource_id`, `date_from`, `date_to` |  |  | `resource` |
| `resource.resource_calendar_leave_tree` | list |  | `name`, `resource_id`, `company_id`, `calendar_id`, `date_from`, `date_to` |  |  | `resource` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.resource_calendar_global_leaves_action_from_calendar` | Public Holidays | list | `[('resource_id', '=', False)]` | `{             'default_calendar_id': active_id,             'search_default_calendar_id': active_id}` |  | `hr_holidays` |
| `hr_holidays.open_view_public_holiday` | Public Holidays | list,form | `[('resource_id', '=', False)]` | `{'search_default_filter_date': True}` |  | `hr_holidays` |
| `resource.action_resource_calendar_leave_tree` | Resource Time Off | list,form,calendar |  |  |  | `resource` |
| `resource.resource_calendar_leaves_action_from_calendar` | Resource Time Off | list,form,calendar |  | `{             'default_calendar_id': active_id,             'search_default_calendar_id': active_id}` |  | `resource` |
| `resource.resource_calendar_closing_days` | Closing Days | calendar,list,form | `[('calendar_id','=',active_id), ('resource_id','=',False)]` | `{'default_calendar_id': active_id}` |  | `resource` |
| `resource.resource_calendar_resources_leaves` | Resources Time Off | calendar,list,form | `[('calendar_id','=',active_id), ('resource_id','!=',False)]` | `{'default_calendar_id': active_id}` |  | `resource` |

Machine-readable definition: `../../../schemas/data/entities/resource.calendar.leaves.json`; views: `../../../schemas/interfaces/views/resource.calendar.leaves.json`.
