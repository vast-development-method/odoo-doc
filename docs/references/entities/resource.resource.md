# Resources (`resource.resource`)

**Transport name:** `resource.resource`  
**Storage name:** `resource_resource`  
**Kind:** persistent entity (one table)  
**Defined by package:** `resource`  
**Extended by packages:** `resource_mail`, `hr`, `hr_holidays`, `hr_skills`

Description: Resources

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to False, it will allow you to hide the resource record without removing it. |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `resource_type` | Type | selection |  | required; default `user` |
| `user_id` | User | many to one | `res.users` | indexed (btree_not_null); not copied on duplication; Help: Related user name for the resource to manage its access.; extended by packages `hr` |
| `avatar_128` | Avatar 128 | image |  | computed by rule `_compute_avatar_128` (not stored) |
| `share` | Share | boolean |  | related through path `user_id.share` |
| `email` | Email | single line text |  | related through path `user_id.email` |
| `phone` | Phone | single line text |  | related through path `user_id.phone` |
| `time_efficiency` | Efficiency Factor | float |  | required; default `100`; Help: This field is used to calculate the expected duration of a work order at this work center. For example, if a work order takes one hour and the efficiency factor is 100%, then the expected duration will be one hour. If the efficiency factor is 200%, however the expected duration will be 30 minutes. |
| `calendar_id` | Working Time | many to one | `resource.calendar` | writable through an inverse rule; default computed dynamically (lambda self: self.env.company.resource_calendar_id); restricted by domain `[('company_id', '=', company_id)]`; Help: Define the working schedule of the resource. If not set, the resource will have fully flexible working hours.; extended by packages `hr` |
| `tz` | Timezone | selection |  | required; default computed dynamically (lambda self: self.env.context.get('tz') or self.env.user.tz or 'UTC') |
| `color` | Color | integer |  | default computed dynamically (_default_color) |
| `im_status` | Im Status | single line text |  | related through path `user_id.im_status` |
| `employee_id` | Employee | one to many | `hr.employee` | must belong to the same company; inverse field `resource_id` |
| `job_title` | Job Title | single line text |  | computed by rule `_compute_job_title` (not stored) |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` (not stored) |
| `work_location_id` | Work Location | many to one |  | related through path `employee_id.work_location_id` |
| `work_email` | Work Email | single line text |  | related through path `employee_id.work_email` |
| `work_phone` | Work Phone | single line text |  | related through path `employee_id.work_phone` |
| `show_hr_icon_display` | Show Human resources Icon Display | boolean |  | related through path `employee_id.show_hr_icon_display` |
| `hr_icon_display` | Human resources Icon Display | selection |  | related through path `employee_id.hr_icon_display` |
| `leave_date_to` | Leave Date To | date |  | related through path `user_id.leave_date_to` |
| `employee_skill_ids` | Employee Skill | one to many |  | related through path `employee_id.employee_skill_ids` |

## Selection values

### `resource_type` (Type)

| Value | Label |
|---|---|
| `user` | Human |
| `material` | Material |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_time_efficiency` | Constraint | `CHECK(time_efficiency>0)` | Time efficiency must be strictly positive | `resource` |

## Operations (27)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `resource` | model |  |
| `_compute_avatar_128` | computation | self | `hr`, `resource` | depends: `user_id`; depends: `employee_id` |  |
| `create` | lifecycle override | self, vals_list | `resource` | model_create_multi |  |
| `copy_data` | lifecycle override | self, default | `resource` |  |  |
| `write` | lifecycle override | self, vals | `resource` |  |  |
| `_onchange_company_id` | on change | self | `resource` | onchange: `company_id` |  |
| `_onchange_user_id` | on change | self | `resource` | onchange: `user_id` |  |
| `_get_work_interval` | preparation rule | self, start, end | `resource` |  |  |
| `_adjust_to_calendar` | internal rule | self, start, end, compute_leaves | `resource` |  | Adjust the given start and end datetimes to the closest effective hours encoded in the resource calendar. Only attendances in the same day as `start` and `end` are considered (respectively). If no attendance is found during that day, the closest hour is None. e.g. simplified example:      given two attendances: 8am-1pm and 2pm-5pm, given start=9am and end=6pm      resource._adjust_to_calendar(start, end)      >>> {resource: (8am, 5pm)} :return: Closest matching start and end of working periods for each resource :rtype: dict(resource, tuple(datetime \| None, datetime \| None)) |
| `_get_unavailable_intervals` | preparation rule | self, start, end | `resource` |  | Compute the intervals during which employee is unavailable with hour granularity between start and end Note: this method is used in enterprise (forecast and planning) |
| `_get_calendars_validity_within_period` | preparation rule | self, start, end, default_company | `hr`, `resource` |  | Gets a dict of dict with resource's id as first key and resource's calendar as secondary key The value is the validity interval of the calendar for the given resource.  Here the validity interval for each calendar is the whole interval but it's meant to be overriden in further modules handling resource's employee contracts. |
| `_get_valid_work_intervals` | preparation rule | self, start, end, calendars, compute_leaves | `resource` |  | Gets the valid work intervals of the resource following their calendars between ``start`` and ``end``  This methods handle the eventuality of a resource having multiple resource calendars, see _get_calendars_validity_within_period method for further explanation.  For flexible calendars and fully flexible resources: -> return the whole interval |
| `_is_fully_flexible` | internal rule | self | `resource` |  | employee has a fully flexible schedule has no working calendar set |
| `_get_calendar_at` | preparation rule | self, date_target, tz | `hr`, `resource` |  |  |
| `_is_flexible` | internal rule | self | `resource` |  | An employee is considered flexible if the field flexible_hours is True on the calendar or the employee is not assigned any calendar, in which case is considered as Fully flexible. |
| `_get_flexible_resources_default_work_intervals` | preparation rule | self, start, end | `resource` |  |  |
| `_get_flexible_resources_calendars_validity_within_period` | preparation rule | self, start, end | `hr`, `resource` |  |  |
| `_format_leave` | internal rule | self, leave, resource_hours_per_day, resource_hours_per_week, ranges_to_remove, start_day, end_day, locale | `hr_holidays`, `resource` |  |  |
| `_get_flexible_resource_valid_work_intervals` | preparation rule | self, start, end | `resource` |  |  |
| `_get_flexible_resource_work_hours` | preparation rule | self, intervals, flexible_resources_hours_per_day, flexible_resources_hours_per_week, work_hours_per_day | `resource` |  |  |
| `_default_color` | preparation rule | self | `resource_mail` |  |  |
| `get_avatar_card_data` | operation | self, fields | `resource_mail` |  |  |
| `_compute_job_title` | computation | self | `hr` | depends: `employee_id` |  |
| `_compute_department_id` | computation | self | `hr` | depends: `employee_id` |  |
| `_inverse_calendar_id` | inverse computation | self | `hr` |  |  |
| `_get_resource_without_contract` | preparation rule | self | `hr` |  |  |
| `_get_contracts_valid_periods` | preparation rule | self, start, end | `hr` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |
| `hr.group_hr_manager` | yes | yes | yes | yes | `hr` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `base.group_system` | no | yes | no | no | `resource` |
| `base.group_user` | no | yes | no | no | `resource` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| resource.resource multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `resource.view_resource_resource_search` | search |  | `name`, `resource_type`, `user_id`, `calendar_id`, `company_id` |  | `Human`, `Material`, `Archived`, `User`, `Type`, `Company`, `Working Time` | `resource` |
| `resource.resource_resource_form` | form |  | `active`, `company_id`, `name`, `user_id`, `resource_type`, `company_id`, `calendar_id`, `tz`, `time_efficiency` |  |  | `resource` |
| `resource.resource_resource_tree` | list |  | `company_id`, `name`, `user_id`, `company_id`, `calendar_id`, `tz`, `resource_type`, `time_efficiency` |  |  | `resource` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `resource.action_resource_resource_tree` | Resources | list,form |  | `{}` |  | `resource` |
| `resource.resource_resource_action_from_calendar` | Resources | list,form |  | `{             'default_calendar_id': active_id,             'search_default_calendar_id': active_id}` |  | `resource` |

Machine-readable definition: `../../../schemas/data/entities/resource.resource.json`; views: `../../../schemas/interfaces/views/resource.resource.json`.
