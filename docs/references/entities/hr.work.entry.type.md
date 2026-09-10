# human resources Work Entry Type (`hr.work.entry.type`)

**Transport name:** `hr.work.entry.type`  
**Storage name:** `hr_work_entry_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_work_entry`  
**Extended by packages:** `hr_work_entry_holidays`

Description: HR Work Entry Type

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (14)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `display_code` | Display Code | single line text |  | translatable; maximum length 3; Help: This code can be changed, it is only for a display purpose (3 letters max) |
| `code` | Payroll Code | single line text |  | required; Help: Careful, the Code is used in many references, changing it could lead to unwanted changes. |
| `external_code` | External Code | single line text |  | Help: Use this code to export your data to a third party |
| `color` | Color | integer |  | default  |
| `sequence` | Sequence | integer |  | default `25` |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to false, it will allow you to hide the work entry type without removing it. |
| `country_id` | Country | many to one | `res.country` | restricted by domain `lambda self: [('id', 'in', self.env.companies.country_id.ids)]` |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |
| `is_leave` | Time Off | boolean |  | default ; Help: Allow the work entry type to be linked with time off types. |
| `is_work` | Working Time | boolean |  | computed by rule `_compute_is_work` (not stored); writable through an inverse rule; Help: If checked, the work entry is counted as work time in the working schedule |
| `amount_rate` | Rate | float |  | default `1.0`; Help: If you want the hours should be paid double, the rate should be 200%. |
| `is_extra_hours` | Added to Monthly Pay | boolean |  | Help: Check this setting if you want the hours to be considered as extra time and added as a bonus to the basic salary. |
| `leave_type_ids` | Time Off Type | one to many | `hr.leave.type` | inverse field `work_entry_type_id`; Help: Work entry used in the payslip. |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_work_entry_type_country` | validation | self | `hr_work_entry` | constrains: `country_id` |  |
| `_check_code_unicity` | validation | self | `hr_work_entry` | constrains: `code`, `country_id` |  |
| `_compute_is_work` | computation | self | `hr_work_entry` | depends: `is_leave` |  |
| `_inverse_is_work` | inverse computation | self | `hr_work_entry` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_work_entry_type_country` | UserError | You can't change the country of this specific work entry type. | `hr_work_entry` |
| `_check_work_entry_type_country` | UserError | You can't change the Country of this work entry type cause it's currently used by the system. You need to delete related working entries first. | `hr_work_entry` |
| `_check_code_unicity` | UserError | The same code cannot be associated to multiple work entry types (%s) | `hr_work_entry` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | no | yes | no | no | `hr_work_entry` |
| `hr.group_hr_manager` | yes | yes | yes | yes | `hr_work_entry` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR Work Entry: Multi Company | global (all users) | `[('country_id', 'in', user.env.companies.mapped('country_id').ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_type_view_search` | search |  | `name` |  | `Archived` | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_tree` | list |  | `name`, `display_code`, `code`, `color`, `country_id` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_form` | form |  | `name`, `active`, `code`, `display_code`, `external_code`, `sequence`, `color`, `country_id`, `amount_rate`, `is_extra_hours` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_type_view_kanban` | kanban |  | `color`, `name`, `code`, `display_code` |  |  | `hr_work_entry` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_type_action` | Work Entry Types | list,kanban,form |  |  |  | `hr_work_entry` |

Machine-readable definition: `../../../schemas/data/entities/hr.work.entry.type.json`; views: `../../../schemas/interfaces/views/hr.work.entry.type.json`.
