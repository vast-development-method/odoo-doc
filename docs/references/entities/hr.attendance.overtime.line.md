# Attendance Overtime Line (`hr.attendance.overtime.line`)

**Transport name:** `hr.attendance.overtime.line`  
**Storage name:** `hr_attendance_overtime_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_attendance`  
**Extended by packages:** `hr_holidays_attendance`

Description: Attendance Overtime Line

## Identity and behavior

- Default ordering: `time_start`
- Display name field: `employee_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `employee_id` | Employee | many to one | `hr.employee` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one |  | related through path `employee_id.company_id` |
| `date` | Day | date |  | required; indexed |
| `status` | Status | selection |  | required; computed by rule `_compute_status` and stored; precomputed before insertion |
| `duration` | Extra Hours | float |  | required; default  |
| `manual_duration` | Extra Hours (encoded) | float |  | computed by rule `_compute_manual_duration` and stored |
| `time_start` | Start | date and time |  |  |
| `time_stop` | Stop | date and time |  |  |
| `amount_rate` | Overtime pay rate | float |  | required; default `1.0` |
| `is_manager` | Is Manager | boolean |  | computed by rule `_compute_is_manager` (not stored) |
| `rule_ids` | Applied Rules | many to many | `hr.attendance.overtime.rule` |  |
| `compensable_as_leave` | Compensable as Time Off | boolean |  |  |

## Selection values

### `status` (Status)

| Value | Label |
|---|---|
| `to_approve` | To Approve |
| `approved` | Approved |
| `refused` | Refused |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_overtime_start_before_end` | Constraint | `CHECK (time_stop > time_start)` | Starting time should be before end time. | `hr_attendance` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_status` | computation | self | `hr_attendance` | depends: `employee_id` |  |
| `_compute_manual_duration` | computation | self | `hr_attendance` | depends: `duration` |  |
| `_compute_is_manager` | computation | self | `hr_attendance` | depends: `employee_id` |  |
| `action_approve` | user action | self | `hr_attendance` |  |  |
| `action_refuse` | user action | self | `hr_attendance` |  |  |
| `_linked_attendances` | internal rule | self | `hr_attendance` |  |  |
| `write` | lifecycle override | self, vals | `hr_attendance` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_attendance_officer` | yes | yes | yes | yes | `hr_attendance` |
| `group_hr_attendance_own_reader` | no | yes | no | no | `hr_attendance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Overtime Line multi company rule | global (all users) | `[('employee_id.company_id', 'in', company_ids)]` | True | True | True | True |
| Overtime Line Administrator: Full access | `[(4, ref('hr_attendance.group_hr_attendance_user'))]` | `[(1,'=',1)]` | True | True | True | True |
| Overtime Line Officer: Restrict to managed employees | `[(4, ref('hr_attendance.group_hr_attendance_officer'))]` | `[('employee_id.attendance_manager_id', '=', user.id)]` | True | True | True | True |
| Overtime Line base user: Read his own overtime lines | `[(4, ref('hr_attendance.group_hr_attendance_own_reader'))]` | `[('employee_id.user_id', '=', user.id)]` | 1 | 0 | 0 | 0 |

Machine-readable definition: `../../../schemas/data/entities/hr.attendance.overtime.line.json`.
