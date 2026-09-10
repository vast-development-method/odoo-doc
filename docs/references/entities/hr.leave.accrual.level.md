# Accrual Plan Level (`hr.leave.accrual.level`)

**Transport name:** `hr.leave.accrual.level`  
**Storage name:** `hr_leave_accrual_level`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_holidays`  
**Extended by packages:** `hr_holidays_attendance`

Description: Accrual Plan Level

## Identity and behavior

- Default ordering: `sequence asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (30)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | sequence | integer |  | computed by rule `_compute_sequence` and stored; Help: Sequence is generated automatically by start time delta. |
| `accrual_plan_id` | Accrual Plan | many to one | `hr.leave.accrual.plan` | required; default computed dynamically (lambda self: self.env.context.get('active_id', None)); indexed; on delete of the target: cascade |
| `accrued_gain_time` | Accrued Gain Time | selection |  | related through path `accrual_plan_id.accrued_gain_time` |
| `start_count` | Start Count | integer |  | Help: The accrual starts after a defined period from the allocation start date. This field defines the number of days, months or years after which accrual is used. |
| `start_type` | Start Type | selection |  | required; default `day`; Help: This field defines the unit of time after which the accrual starts. |
| `milestone_date` | Milestone Date | selection |  | required; computed by rule `_compute_milestone_date` and stored; writable through an inverse rule; default `creation` |
| `added_value` | Added Value | float |  | required; default `1`; precision `[16, 5]` |
| `added_value_type` | Added Value Type | selection |  | required; computed by rule `_compute_added_value_type` and stored; writable through an inverse rule; precomputed before insertion |
| `frequency` | Frequency | selection |  | required; computed by rule `_compute_frequency` and stored; default `daily`; on delete of the target: {"worked_hours": "cascade"}; extended by packages `hr_holidays_attendance` |
| `week_day` | Allocation on | selection |  | required; default `0` |
| `first_day` | First Day | selection |  | default `1` |
| `second_day` | Second Day | selection |  | default `15` |
| `first_month_day` | First Month Day | selection |  | computed by rule `_compute_first_month_day` and stored; default `1` |
| `first_month` | First Month | selection |  | default `1` |
| `second_month_day` | Second Month Day | selection |  | computed by rule `_compute_second_month_day` and stored; default `1` |
| `second_month` | Second Month | selection |  | default `7` |
| `yearly_month` | Yearly Month | selection |  | default `1` |
| `yearly_day` | Yearly Day | selection |  | computed by rule `_compute_yearly_day` and stored; default `1` |
| `cap_accrued_time` | Cap Accrued Time | boolean |  | Help: When the field is checked the balance of an allocation using this accrual plan will never exceed the specified amount. |
| `maximum_leave` | Maximum Leave | float |  | computed by rule `_compute_maximum_leave` and stored; default ; precision `[16, 2]`; Help: Choose a cap for this accrual. |
| `cap_accrued_time_yearly` | Cap Accrued Time Yearly | boolean |  | Help: When the field is checked the total amount accrued each year will be capped at the specified amount |
| `maximum_leave_yearly` | Maximum Leave Yearly | float |  | precision `[16, 2]` |
| `can_be_carryover` | Can Be Carryover | boolean |  | read only; related through path `accrual_plan_id.can_be_carryover` |
| `action_with_unused_accruals` | Action With Unused Accruals | selection |  | required; computed by rule `_compute_action_with_unused_accruals` and stored; default `lost`; Help: When the Carry-Over Time is reached, according to Plan's setting, select what you want to happen with the unused time off: Lost (time will be reset to zero), Carried over (accrued time carried over to the next period.) |
| `carryover_options` | Carryover Options | selection |  | required; computed by rule `_compute_carryover_options` and stored; default `unlimited`; Help: You can limit the accrued time carried over for the next period. |
| `postpone_max_days` | Postpone Max Days | integer |  | Help: Set a maximum of accruals an allocation keeps at the end of the year. |
| `can_modify_value_type` | Can Modify Value Type | boolean |  | computed by rule `_compute_can_modify_value_type` (not stored); default  |
| `accrual_validity` | Accrual Validity | boolean |  | computed by rule `_compute_accrual_validity` and stored |
| `accrual_validity_count` | Accrual Validity Count | integer |  | default `1`; Help: You can define a period of time where the days carried over will be available |
| `accrual_validity_type` | Accrual Validity Type | selection |  | required; default `day`; Help: This field defines the unit of time after which the accrual ends. |

## Selection values

### `start_type` (Start Type)

| Value | Label |
|---|---|
| `day` | Days |
| `month` | Months |
| `year` | Years |

### `milestone_date` (Milestone Date)

| Value | Label |
|---|---|
| `creation` | At allocation creation |
| `after` | After |

### `added_value_type` (Added Value Type)

| Value | Label |
|---|---|
| `day` | Day(s) |
| `hour` | Hour(s) |

### `frequency` (Frequency)

| Value | Label |
|---|---|
| `hourly` | Hourly |
| `daily` | Daily |
| `weekly` | Weekly |
| `bimonthly` | Twice a month |
| `monthly` | Monthly |
| `biyearly` | Twice a year |
| `yearly` | Yearly |
| `worked_hours` | Per Hour Worked |

### `week_day` (Allocation on)

| Value | Label |
|---|---|
| `0` | Monday |
| `1` | Tuesday |
| `2` | Wednesday |
| `3` | Thursday |
| `4` | Friday |
| `5` | Saturday |
| `6` | Sunday |

### `first_month` (First Month)

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |

### `second_month` (Second Month)

| Value | Label |
|---|---|
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

### `yearly_month` (Yearly Month)

| Value | Label |
|---|---|
| `1` | January |
| `2` | February |
| `3` | March |
| `4` | April |
| `5` | May |
| `6` | June |
| `7` | July |
| `8` | August |
| `9` | September |
| `10` | October |
| `11` | November |
| `12` | December |

### `action_with_unused_accruals` (Action With Unused Accruals)

| Value | Label |
|---|---|
| `lost` | Lost |
| `all` | Carried over |

### `carryover_options` (Carryover Options)

| Value | Label |
|---|---|
| `unlimited` | Unlimited |
| `limited` | Up to |

### `accrual_validity_type` (Accrual Validity Type)

| Value | Label |
|---|---|
| `day` | Days |
| `month` | Months |

## Database constraints and indexes (5)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_start_count_check` | Constraint | `CHECK((start_count > 0 AND milestone_date = 'after') OR (start_count = 0 AND milestone_date = 'creation'))` | You can not start an accrual in the past. | `hr_holidays` |
| `_added_value_greater_than_zero` | Constraint | `CHECK(added_value > 0)` | You must give a rate greater than 0 in accrual plan levels. | `hr_holidays` |
| `_valid_postpone_max_days_value` | Constraint | `CHECK(action_with_unused_accruals <> 'all' OR carryover_options <> 'limited' OR COALESCE(postpone_max_days, 0) > 0)` | You cannot have a maximum quantity to carryover set to 0. | `hr_holidays` |
| `_valid_accrual_validity_value` | Constraint | `CHECK(accrual_validity IS NOT TRUE OR COALESCE(accrual_validity_count, 0) > 0)` | You cannot have an accrual validity time set to 0. | `hr_holidays` |
| `_valid_yearly_cap_value` | Constraint | `CHECK(cap_accrued_time_yearly IS NOT TRUE OR COALESCE(maximum_leave_yearly, 0) > 0)` | You cannot have a cap on yearly accrued time without setting a maximum amount. | `hr_holidays` |

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_dates` | validation | self | `hr_holidays` | constrains: `first_day`, `second_day`, `week_day`, `frequency` |  |
| `_check_maximum_leaves` | validation | self | `hr_holidays` | constrains: `cap_accrued_time`, `maximum_leave` |  |
| `_compute_sequence` | computation | self | `hr_holidays` | depends: `start_count`, `start_type` |  |
| `_compute_can_modify_value_type` | computation | self | `hr_holidays` | depends: `accrual_plan_id`, `accrual_plan_id.level_ids`, `accrual_plan_id.time_off_type_id` |  |
| `_inverse_added_value_type` | inverse computation | self | `hr_holidays` |  |  |
| `_compute_added_value_type` | computation | self | `hr_holidays` | depends: `accrual_plan_id`, `accrual_plan_id.level_ids`, `accrual_plan_id.added_value_type`, `accrual_plan_id.time_off_type_id` |  |
| `_set_day` | internal rule | self, day_field, month_field | `hr_holidays` |  |  |
| `_compute_first_month_day` | computation | self | `hr_holidays` | depends: `first_month` |  |
| `_compute_second_month_day` | computation | self | `hr_holidays` | depends: `second_month` |  |
| `_compute_yearly_day` | computation | self | `hr_holidays` | depends: `yearly_month` |  |
| `_compute_maximum_leave` | computation | self | `hr_holidays` | depends: `cap_accrued_time` |  |
| `_compute_action_with_unused_accruals` | computation | self | `hr_holidays` | depends: `can_be_carryover` |  |
| `_compute_carryover_options` | computation | self | `hr_holidays` | depends: `action_with_unused_accruals` |  |
| `_compute_accrual_validity` | computation | self | `hr_holidays` | depends: `action_with_unused_accruals` |  |
| `_compute_milestone_date` | computation | self | `hr_holidays` | depends: `start_count`, `milestone_date` |  |
| `_inverse_milestone_date` | inverse computation | self | `hr_holidays` |  |  |
| `_get_hourly_frequencies` | preparation rule | self | `hr_holidays_attendance`, `hr_holidays` |  |  |
| `_get_next_date` | preparation rule | self, last_call | `hr_holidays` |  | Returns the next date with the given last call |
| `_get_previous_date` | preparation rule | self, last_call | `hr_holidays` |  | Returns the date a potential previous call would have been at For example if you have a monthly level giving 16/02 would return 01/02 Contrary to `_get_next_date` this function will return the 01/02 if that date is given |
| `_get_level_transition_date` | preparation rule | self, allocation_start | `hr_holidays` |  |  |
| `action_save_new` | user action | self | `hr_holidays` |  |  |
| `_check_worked_hours` | validation | self | `hr_holidays_attendance` | constrains: `frequency` |  |
| `_compute_frequency` | computation | self | `hr_holidays_attendance` | depends: `accrued_gain_time` |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_dates` | ValidationError | error_message | `hr_holidays` |
| `_check_maximum_leaves` | UserError | You cannot have a balance cap on accrued time set to 0. | `hr_holidays` |
| `_get_next_date` | ValidationError | Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly. | `hr_holidays` |
| `_get_previous_date` | ValidationError | Your frequency selection is not correct: please choose a frequency between theses options:Hourly, Daily, Weekly, Twice a month, Monthly, Twice a year and Yearly. | `hr_holidays` |
| `_check_worked_hours` | ValidationError | You can't base accrued time on hours worked, because time is accrued at the start of the period. | `hr_holidays_attendance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_holidays.group_hr_holidays_user` | no | yes | no | no | `hr_holidays` |
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_holidays.hr_accrual_level_view_form` | form |  | `can_modify_value_type`, `added_value`, `added_value_type`, `frequency`, `week_day`, `first_day`, `first_day`, `second_day`, `first_month_day`, `first_month`, `second_month_day`, `second_month`, `yearly_day`, `yearly_month`, `milestone_date`, `start_count`, `start_type`, `action_with_unused_accruals`, `carryover_options`, `postpone_max_days`, `added_value_type`, `accrual_validity`, `accrual_validity_count`, `accrual_validity_type`, `cap_accrued_time_yearly`, `maximum_leave_yearly`, `added_value_type`, `cap_accrued_time`, `maximum_leave`, `added_value_type` | `Save`, `Save & New`, `Discard`, `Delete` |  | `hr_holidays` |
| `hr_holidays_attendance.hr_leave_accrual_level_view_form` | xpath | `hr_holidays.hr_accrual_level_view_form` | `frequency`, `frequency` |  |  | `hr_holidays_attendance` |

Machine-readable definition: `../../../schemas/data/entities/hr.leave.accrual.level.json`; views: `../../../schemas/interfaces/views/hr.leave.accrual.level.json`.
