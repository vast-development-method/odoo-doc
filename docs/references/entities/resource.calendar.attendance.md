# Work Detail (`resource.calendar.attendance`)

**Transport name:** `resource.calendar.attendance`  
**Storage name:** `resource_calendar_attendance`  
**Kind:** persistent entity (one table)  
**Defined by package:** `resource`  
**Extended by packages:** `hr_work_entry`, `point_of_sale`

Description: Work Detail

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, week_type, dayofweek, hour_from`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `dayofweek` | Day of Week | selection |  | required; default `0`; indexed |
| `hour_from` | Work from | float |  | required; default ; indexed; Help: Start and End time of working. A specific value of 24:00 is interpreted as 23:59:59.999999. |
| `hour_to` | Work to | float |  | required; default  |
| `duration_hours` | Duration (hours) | float |  | computed by rule `_compute_duration_hours` and stored; writable through an inverse rule |
| `duration_days` | Duration (days) | float |  | computed by rule `_compute_duration_days` and stored |
| `calendar_id` | Resource's Calendar | many to one | `resource.calendar` | required; indexed; on delete of the target: cascade |
| `duration_based` | Duration Based | boolean |  | related through path `calendar_id.duration_based` |
| `day_period` | Day Period | selection |  | required; default `morning` |
| `week_type` | Week Number | selection |  | default  |
| `two_weeks_calendar` | Calendar in 2 weeks mode | boolean |  | related through path `calendar_id.two_weeks_calendar` |
| `display_type` | Display Type | selection |  | default ; Help: Technical field for UX purpose. |
| `sequence` | Sequence | integer |  | default `10`; Help: Gives the sequence of this line when displaying the resource calendar. |
| `work_entry_type_id` | Work Entry Type | many to one | `hr.work.entry.type` | default computed dynamically (_default_work_entry_type_id); visible only to groups `hr.group_hr_user` |

## Selection values

### `dayofweek` (Day of Week)

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

### `day_period` (Day Period)

| Value | Label |
|---|---|
| `morning` | Morning |
| `lunch` | Break |
| `afternoon` | Afternoon |
| `full_day` | Full Day |

### `week_type` (Week Number)

| Value | Label |
|---|---|
| `1` | Second |
| `0` | First |

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `line_section` | Section |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_hours` | on change | self | `resource` | onchange: `hour_from`, `hour_to` |  |
| `_check_day_period` | validation | self | `resource` | constrains: `day_period` |  |
| `get_week_type` | operation | self, date | `resource` | model |  |
| `_compute_duration_hours` | computation | self | `resource` | depends: `hour_from`, `hour_to`, `day_period` |  |
| `_inverse_duration_hours` | inverse computation | self | `resource` |  |  |
| `_compute_duration_days` | computation | self | `resource` | depends: `day_period` |  |
| `_compute_display_name` | computation | self | `resource` | depends: `week_type` |  |
| `_copy_attendance_vals` | internal rule | self | `hr_work_entry`, `resource` |  |  |
| `_is_work_period` | internal rule | self | `hr_work_entry`, `resource` |  |  |
| `_default_work_entry_type_id` | preparation rule | self | `hr_work_entry` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_day_period` | UserError | %(att)s is a break attendance, You should not have such record on duration based calendar | `resource` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | yes | `hr` |
| `mrp.group_mrp_user` | yes | yes | yes | yes | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `project.group_project_user` | no | yes | no | no | `project` |
| `base.group_user` | no | yes | no | no | `resource` |
| `base.group_system` | yes | yes | yes | yes | `resource` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.resource_calendar_attendance_view_tree` | field | `resource.view_resource_calendar_attendance_tree` | `week_type`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `hr_work_entry.resource_calendar_attendance_view_form` | field | `resource.view_resource_calendar_attendance_form` | `day_period`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `resource.view_resource_calendar_attendance_tree` | list |  | `sequence`, `display_type`, `display_name`, `name`, `dayofweek`, `day_period`, `duration_hours`, `hour_from`, `hour_to`, `duration_days`, `week_type` |  |  | `resource` |
| `resource.view_resource_calendar_attendance_form` | form |  | `name`, `dayofweek`, `hour_from`, `hour_to`, `day_period`, `duration_days` |  |  | `resource` |

Machine-readable definition: `../../../schemas/data/entities/resource.calendar.attendance.json`; views: `../../../schemas/interfaces/views/resource.calendar.attendance.json`.
