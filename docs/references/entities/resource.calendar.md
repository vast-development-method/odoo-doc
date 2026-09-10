# Resource Working Time (`resource.calendar`)

**Transport name:** `resource.calendar`  
**Storage name:** `resource_calendar`  
**Kind:** persistent entity (one table)  
**Defined by package:** `resource`  
**Extended by packages:** `hr`, `hr_holidays`, `hr_work_entry`

Description: Resource Working Time

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to false, it will allow you to hide the Working Time without removing it. |
| `attendance_ids` | Working Time | one to many | `resource.calendar.attendance` | computed by rule `_compute_attendance_ids` and stored; inverse field `calendar_id` |
| `attendance_ids_1st_week` | Working Time 1st Week | one to many | `resource.calendar.attendance` | computed by rule `_compute_two_weeks_attendance` (not stored); writable through an inverse rule; inverse field `calendar_id` |
| `attendance_ids_2nd_week` | Working Time 2nd Week | one to many | `resource.calendar.attendance` | computed by rule `_compute_two_weeks_attendance` (not stored); writable through an inverse rule; inverse field `calendar_id` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); indexed (btree_not_null); restricted by domain `lambda self: [('id', 'in', self.env.companies.ids)]` |
| `leave_ids` | Time Off | one to many | `resource.calendar.leaves` | inverse field `calendar_id` |
| `schedule_type` | Schedule Type | selection |  | required; default `fully_fixed`; Help: Choose which level of definition you want to define on your Schedule - Flexible : Define an amount of hours to work on the week. - Fully Fixed : define the days, periods and the start & end time for each period of the day |
| `duration_based` | Attendance based on duration | boolean |  | Help: The hours will be centered around 12:00 to cover the duration for the day |
| `flexible_hours` | Flexible Hours | boolean |  | computed by rule `_compute_flexible_hours` and stored; writable through an inverse rule; Help: When enabled, it will allow employees to work flexibly, without relying on the company's working schedule (working hours). |
| `full_time_required_hours` | Full Time Equivalent | float |  | computed by rule `_compute_full_time_required_hours` and stored; Help: Number of hours to work on the company schedule to be considered as fulltime. |
| `global_leave_ids` | Global Time Off | one to many | `resource.calendar.leaves` | computed by rule `_compute_global_leave_ids` and stored; restricted by domain `[["resource_id", "=", false]]`; inverse field `calendar_id` |
| `hours_per_day` | Average Hour per Day | float |  | computed by rule `_compute_hours_per_day` and stored; precision `[2, 2]`; Help: Average hours per day a resource is supposed to work with this calendar. |
| `hours_per_week` | Hours per Week | float |  | computed by rule `_compute_hours_per_week` and stored; not copied on duplication |
| `is_fulltime` | Is Full Time | boolean |  | computed by rule `_compute_work_time_rate` (not stored) |
| `two_weeks_calendar` | Calendar in 2 weeks mode | boolean |  |  |
| `two_weeks_explanation` | Explanation | single line text |  | computed by rule `_compute_two_weeks_explanation` (not stored) |
| `tz` | Timezone | selection |  | required; default computed dynamically (lambda self: self.env.context.get('tz') or self.env.user.tz or self.env.ref('base.user_admin').tz or 'UTC'); Help: This field is used in order to define in which timezone the resources will work. |
| `tz_offset` | Timezone offset | single line text |  | computed by rule `_compute_tz_offset` (not stored) |
| `work_resources_count` | Work Resources count | integer |  | computed by rule `_compute_work_resources_count` (not stored) |
| `work_time_rate` | Work Time Rate | float |  | computed by rule `_compute_work_time_rate` (not stored); searchable through a search rule; Help: Work time rate versus full time working schedule, should be between 0 and 100 %. |
| `associated_leaves_count` | Time Off Count | integer |  | computed by rule `_compute_associated_leaves_count` (not stored) |

## Selection values

### `schedule_type` (Schedule Type)

| Value | Label |
|---|---|
| `flexible` | Flexible |
| `fully_fixed` | Fully Fixed |

## Operations (47)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `resource` | model |  |
| `_check_attendance_ids` | validation | self | `resource` | constrains: `attendance_ids` |  |
| `_compute_two_weeks_attendance` | computation | self | `resource` | depends: `two_weeks_calendar` |  |
| `_inverse_two_weeks_calendar` | inverse computation | self | `resource` |  |  |
| `_compute_full_time_required_hours` | computation | self | `resource` | depends: `hours_per_week`, `company_id.resource_calendar_id.hours_per_week` |  |
| `_compute_flexible_hours` | computation | self | `resource` | depends: `schedule_type` |  |
| `_inverse_flexible_hours` | inverse computation | self | `resource` |  |  |
| `_compute_attendance_ids` | computation | self | `resource` | depends: `company_id` |  |
| `_onchange_attendance_ids` | on change | self | `resource` | onchange: `attendance_ids` |  |
| `_compute_global_leave_ids` | computation | self | `resource` | depends: `company_id` |  |
| `_compute_hours_per_day` | computation | self | `resource` | depends: `attendance_ids`, `attendance_ids.hour_from`, `attendance_ids.hour_to`, `two_weeks_calendar`, `flexible_hours` | Compute the average hours per day. Cannot directly depend on hours_per_week because of rounding issues. |
| `_compute_hours_per_week` | computation | self | `hr_work_entry`, `resource` | depends: `attendance_ids`, `attendance_ids.hour_from`, `attendance_ids.hour_to`, `two_weeks_calendar`, `flexible_hours`; depends: `attendance_ids.work_entry_type_id.is_leave` | Compute the average hours per week |
| `_compute_two_weeks_explanation` | computation | self | `resource` | depends: `two_weeks_calendar` |  |
| `_compute_tz_offset` | computation | self | `resource` | depends: `tz` |  |
| `_compute_work_resources_count` | computation | self | `resource` |  |  |
| `_compute_work_time_rate` | computation | self | `resource` | depends: `hours_per_week`, `full_time_required_hours` |  |
| `_search_work_time_rate` | search rule | self, operator, value | `resource` | model |  |
| `create` | lifecycle override | self, vals_list | `resource` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `resource` |  |  |
| `switch_calendar_type` | operation | self | `resource` |  |  |
| `switch_based_on_duration` | operation | self | `resource` |  |  |
| `_attendance_intervals_batch` | internal rule | self, start_dt, end_dt, resources, domain, tz, lunch | `resource` |  |  |
| `_handle_flexible_leave_interval` | internal rule | self, dt0, dt1, leave | `resource` |  | Hook method to handle flexible leave intervals. Can be overridden in other modules. |
| `_leave_intervals` | internal rule | self, start_dt, end_dt, resource, domain, tz | `resource` |  |  |
| `_leave_intervals_batch` | internal rule | self, start_dt, end_dt, resources, domain, tz | `resource` |  | Return the leave intervals in the given datetime range. The returned intervals are expressed in specified tz or in the calendar's timezone. |
| `_work_intervals_batch` | internal rule | self, start_dt, end_dt, resources, domain, tz, compute_leaves | `resource` |  | Return the effective work intervals between the given datetimes. |
| `_unavailable_intervals` | internal rule | self, start_dt, end_dt, resource, domain, tz | `resource` |  |  |
| `_unavailable_intervals_batch` | internal rule | self, start_dt, end_dt, resources, domain, tz | `resource` |  | Return the unavailable intervals between the given datetimes. |
| `_check_overlap` | validation | self, attendance_ids | `resource` |  | attendance_ids correspond to attendance of a week, will check for each day of week that there are no superimpose. |
| `_get_attendance_intervals_days_data` | preparation rule | self, attendance_intervals | `resource` |  | helper function to compute duration of `intervals` that have 'resource.calendar.attendance' records as payload (3rd element in tuple). expressed in days and hours.  resource.calendar.attendance records have durations associated with them so this method merely calculates the proportion that is covered by the intervals. |
| `_get_closest_work_time` | preparation rule | self, dt, match_end, resource, search_range, compute_leaves | `resource` |  | Return the closest work interval boundary within the search range. Consider only starts of intervals unless `match_end` is True. It will then only consider ends of intervals. :param dt: reference datetime :param match_end: wether to search for the begining of an interval or the end. :param search_range: time interval considered. Defaults to the entire day of `dt` :rtype: datetime \| None |
| `_get_days_per_week` | preparation rule | self | `resource` |  |  |
| `_get_hours_per_week` | preparation rule | self | `resource` |  | Calculate the average hours worked per week. |
| `_get_hours_per_day` | preparation rule | self | `resource` |  | Calculate the average hours worked per workday. |
| `_get_global_attendances` | preparation rule | self | `hr_work_entry`, `resource` |  |  |
| `_get_unusual_days` | preparation rule | self, start_dt, end_dt, company_id | `resource` |  |  |
| `_get_default_attendance_ids` | preparation rule | self, company_id | `resource` |  | return a copy of the company's calendar attendance or default 40 hours/week |
| `_get_two_weeks_attendance` | preparation rule | self | `resource` |  |  |
| `get_work_hours_count` | operation | self, start_dt, end_dt, compute_leaves, domain | `resource` |  | `compute_leaves` controls whether or not this method is taking into account the global leaves.  `domain` controls the way leaves are recognized. None means default value ('time_type', '=', 'leave')  Counts the number of work hours between two datetimes. |
| `get_work_duration_data` | operation | self, from_datetime, to_datetime, compute_leaves, domain | `resource` |  | Get the working duration (in days and hours) for a given period, only based on the current calendar. This method does not use resource to compute it.  `domain` is used in order to recognise the leaves to take, None means default value ('time_type', '=', 'leave')  Returns a dict {'days': n, 'hours': h} containing the quantity of working time expressed as days and as hours. |
| `plan_hours` | operation | self, hours, day_dt, compute_leaves, domain, resource | `resource` |  | `compute_leaves` controls whether or not this method is taking into account the global leaves.  `domain` controls the way leaves are recognized. None means default value ('time_type', '=', 'leave')  Return datetime after having planned hours |
| `plan_days` | operation | self, days, day_dt, compute_leaves, domain | `resource` |  | `compute_leaves` controls whether or not this method is taking into account the global leaves.  `domain` controls the way leaves are recognized. None means default value ('time_type', '=', 'leave')  Returns the datetime of a days scheduling. |
| `_works_on_date` | internal rule | self, date | `resource` |  |  |
| `_get_hours_for_date` | preparation rule | self, target_date, day_period | `resource` |  | An instance method on a calendar to get the start and end float hours for a given date. :param target_date: The date to find working hours. :param day_period: Optional string ('morning', 'afternoon') to filter for half-days. :return: A tuple of floats (hour_from, hour_to). |
| `_get_working_hours` | preparation rule | self | `resource` |  |  |
| `transfer_leaves_to` | operation | self, other_calendar, resources, from_date | `hr` |  | Transfer some resource.calendar.leaves from 'self' to another calendar 'other_calendar'. Transfered leaves linked to `resources` (or all if `resources` is None) and starting after 'from_date' (or today if None). |
| `_compute_associated_leaves_count` | computation | self | `hr_holidays` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_attendance_ids` | ValidationError | In a calendar with 2 weeks mode, all periods need to be in the sections. | `resource` |
| `_onchange_attendance_ids` | ValidationError | You can't delete section between weeks. | `resource` |
| `_check_overlap` | ValidationError | Attendances can't overlap. | `resource` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `base.group_user` | no | yes | no | no | `resource` |
| `base.group_system` | yes | yes | yes | yes | `resource` |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.resource_calendar_form_inherit` | xpath | `resource.resource_calendar_form` | `associated_leaves_count` | `%(resource_calendar_global_leaves_action_from_calendar)d` |  | `hr_holidays` |
| `hr_holidays.resource_calendar_view_tree` | field | `resource.view_resource_calendar_tree` | `work_resources_count`, `associated_leaves_count` |  |  | `hr_holidays` |
| `resource.view_resource_calendar_search` | search |  | `name`, `company_id` |  | `Archived`, `Partial working schedules`, `Flexible`, `groupby_company` | `resource` |
| `resource.resource_calendar_form` | form |  | `work_resources_count`, `name`, `schedule_type`, `full_time_required_hours`, `work_time_rate`, `company_id`, `tz`, `tz_offset`, `hours_per_day`, `hours_per_week`, `two_weeks_calendar`, `hours_per_day`, `hours_per_week`, `attendance_ids`, `two_weeks_explanation`, `hours_per_day`, `hours_per_week`, `attendance_ids_1st_week`, `two_weeks_explanation`, `hours_per_day`, `hours_per_week`, `attendance_ids_2nd_week` | `%(resource_calendar_leaves_action_from_calendar)d`, `%(resource_resource_action_from_calendar)d`, `switch_based_on_duration`, `switch_based_on_duration`, `switch_calendar_type`, `switch_calendar_type` |  | `resource` |
| `resource.view_resource_calendar_tree` | list |  | `name`, `hours_per_week`, `work_time_rate`, `schedule_type`, `full_time_required_hours`, `work_resources_count`, `company_id` |  |  | `resource` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `resource.action_resource_calendar_form` | Working Schedules | list,form | `['\|', ('company_id', '=', False), ('company_id', 'in', allowed_company_ids)]` |  |  | `resource` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr.menu_resource_calendar_view` | Working Schedules | `menu_config_employee` | `resource.action_resource_calendar_form` | 6 |  |

Machine-readable definition: `../../../schemas/data/entities/resource.calendar.json`; views: `../../../schemas/interfaces/views/resource.calendar.json`.
