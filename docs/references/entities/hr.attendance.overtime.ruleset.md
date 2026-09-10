# Overtime Ruleset (`hr.attendance.overtime.ruleset`)

**Transport name:** `hr.attendance.overtime.ruleset`  
**Storage name:** `hr_attendance_overtime_ruleset`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_attendance`

Description: Overtime Ruleset

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `description` | Description | rich text |  |  |
| `rule_ids` | Rule | one to many | `hr.attendance.overtime.rule` | inverse field `ruleset_id` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `country_id` | Country | many to one | `res.country` | default computed dynamically (lambda self: self.env.company.country_id) |
| `rate_combination_mode` | Rate Combination Mode | selection |  | required; default `max`; Help: Controls how the rates from the different rules that apply are combined.   Max: use the highest rate. (e.g.: combined for 150% and 120 = 150%)   Sum: sum the *extra* pay (i.e. above 100%).     e.g.: combined rate for 150% & 120% = 100% (baseline) + (150-100)% + (120-100)% = 170% |
| `rules_count` | Rules Count | integer |  | computed by rule `_compute_rules_count` (not stored) |
| `active` | Active | boolean |  | default `True` |

## Selection values

### `rate_combination_mode` (Rate Combination Mode)

| Value | Label |
|---|---|
| `max` | Maximum Rate |
| `sum` | Sum of all rates |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_rules_count` | computation | self | `hr_attendance` |  |  |
| `_attendances_to_regenerate_for` | internal rule | self | `hr_attendance` |  |  |
| `action_regenerate_overtimes` | user action | self | `hr_attendance` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_attendance_manager` | yes | yes | yes | yes | `hr_attendance` |
| `hr.group_hr_manager` | no | yes | no | no | `hr_attendance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Attendance Overtime Ruleset | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | False | False | False |
| Attendance Overtime Ruleset | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_ruleset_view_form` | form |  | `active`, `name`, `rate_combination_mode`, `country_id`, `description`, `rule_ids`, `sequence`, `name`, `base_off`, `information_display`, `amount_rate` | `Regenerate overtimes` |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_ruleset_view_list` | list |  | `name`, `country_id`, `rate_combination_mode`, `rules_count` |  |  | `hr_attendance` |
| `hr_attendance.hr_attendance_overtime_ruleset_view_filter` | search |  |  |  | `At least one rule`, `Archived`, `Rate Mode`, `Country` | `hr_attendance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_attendance.hr_attendance_overtime_ruleset_action` | Rulesets | list,form |  |  |  | `hr_attendance` |

Machine-readable definition: `../../../schemas/data/entities/hr.attendance.overtime.ruleset.json`; views: `../../../schemas/interfaces/views/hr.attendance.overtime.ruleset.json`.
