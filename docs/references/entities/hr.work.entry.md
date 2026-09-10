# human resources Work Entry (`hr.work.entry`)

**Transport name:** `hr.work.entry`  
**Storage name:** `hr_work_entry`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_work_entry`  
**Extended by packages:** `hr_work_entry_holidays`, `l10n_fr_hr_work_entry_holidays`

Description: HR Work Entry

## Identity and behavior

- Default ordering: `create_date`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `active` | Active | boolean |  | default `True` |
| `employee_id` | Employee | many to one | `hr.employee` | required; indexed; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `version_id` | Employee Record | many to one | `hr.version` | required; indexed |
| `work_entry_source` | Work Entry Source | selection |  | related through path `version_id.work_entry_source` |
| `date` | Date | date |  | required |
| `duration` | Duration | float |  | default `8` |
| `work_entry_type_id` | Work Entry Type | many to one | `hr.work.entry.type` | default computed dynamically (lambda self: self.env['hr.work.entry.type'].search([], limit=1)); indexed; restricted by domain `lambda self: self._get_work_entry_type_domain()` |
| `display_code` | Display Code | single line text |  | related through path `work_entry_type_id.display_code` |
| `code` | Code | single line text |  | related through path `work_entry_type_id.code` |
| `external_code` | External Code | single line text |  | related through path `work_entry_type_id.external_code` |
| `color` | Color | integer |  | read only; related through path `work_entry_type_id.color` |
| `state` | State | selection |  | default `draft` |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company) |
| `conflict` | Conflicts | boolean |  | computed by rule `_compute_conflict` and stored |
| `department_id` | Department | many to one | `hr.department` | related through path `employee_id.department_id` and stored |
| `amount_rate` | Pay rate | float |  |  |
| `country_id` | Country | many to one | `res.country` | related through path `employee_id.company_id.country_id`; searchable through a search rule |
| `leave_id` | Time Off | many to one | `hr.leave` |  |
| `leave_state` | Leave State | selection |  | related through path `leave_id.state` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | New |
| `conflict` | In Conflict |
| `validated` | In Payslip |
| `cancelled` | Cancelled |

## State fields

State machine fields of this entity: `state`, `leave_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_contract_date_start_stop_idx` | Index | `(version_id, date) WHERE state IN ('draft', 'validated')` |  | `hr_work_entry` |

## Operations (28)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_duration` | validation | self | `hr_work_entry` | constrains: `duration` |  |
| `_compute_display_name` | computation | self | `hr_work_entry` | depends: `display_code`, `duration` |  |
| `_compute_name` | computation | self | `hr_work_entry` | depends: `work_entry_type_id`, `employee_id` |  |
| `_compute_conflict` | computation | self | `hr_work_entry` | depends: `state` |  |
| `_onchange_version_id` | on change | self | `hr_work_entry` | onchange: `employee_id`, `date` |  |
| `_set_current_contract` | internal rule | self, vals | `hr_work_entry` | model |  |
| `get_unusual_days` | operation | self, date_from, date_to | `hr_work_entry` | model |  |
| `action_validate` | user action | self | `hr_work_entry` |  | Try to validate work entries. If some errors are found, set `state` to conflict for conflicting work entries and validation fails. :return: True if validation succeeded |
| `action_split` | user action | self, vals | `hr_work_entry` |  |  |
| `_check_if_error` | validation | self | `hr_work_entry` |  |  |
| `_mark_conflicting_work_entries` | internal rule | self, start, stop | `hr_work_entry` |  | Set `state` to `conflict` for work entries where, for the same employee and day, the total duration exceeds 24 hours. Return True if such entries are found. |
| `_get_leaves_entries_outside_schedule` | preparation rule | self | `hr_work_entry` |  |  |
| `_mark_leaves_outside_schedule` | internal rule | self | `hr_work_entry`, `l10n_fr_hr_work_entry_holidays` |  | Check leave work entries in `self` which are completely outside the contract's theoretical calendar schedule. Mark them as conflicting. :return: leave work entries completely outside the contract's calendar |
| `_mark_already_validated_days` | internal rule | self | `hr_work_entry` |  |  |
| `_to_intervals` | internal rule | self | `hr_work_entry` |  |  |
| `_from_intervals` | internal rule | self, intervals | `hr_work_entry` | model |  |
| `create` | lifecycle override | self, vals_list | `hr_work_entry` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_unlink_except_validated_work_entries` | internal rule | self | `hr_work_entry` | ondelete |  |
| `unlink` | lifecycle override | self | `hr_work_entry` |  |  |
| `_reset_conflicting_state` | internal rule | self | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_error_checking` | internal rule | self, start, stop, skip, employee_ids | `hr_work_entry` |  | Context manager used for conflicts checking. When exiting the context manager, conflicts are checked for all work entries within a date range. By default, the start and end dates are computed according to `self` (min and max respectively) but it can be overwritten by providing other values as parameter. :param start: datetime to overwrite the default behaviour :param stop: datetime to overwrite the default behaviour :param skip: If True, no error checking is done |
| `_get_work_entry_type_domain` | preparation rule | self | `hr_work_entry` |  |  |
| `_search_country_id` | search rule | self, operator, value | `hr_work_entry` |  |  |
| `action_approve_leave` | user action | self | `hr_work_entry_holidays` |  |  |
| `action_refuse_leave` | user action | self | `hr_work_entry_holidays` |  |  |
| `_get_leaves_duration_between_two_dates` | preparation rule | self, employee_id, date_from, date_to | `hr_work_entry_holidays` | model |  |
| `_filter_french_part_time_entries` | internal rule | self | `l10n_fr_hr_work_entry_holidays` |  |  |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_duration` | ValidationError | Duration must be positive and cannot exceed 24 hours. | `hr_work_entry` |
| `action_split` | UserError | You can't split a work entry with less than 1 hour. | `hr_work_entry` |
| `action_split` | UserError | Split work entry duration has to be less than the existing work entry duration. | `hr_work_entry` |
| `_unlink_except_validated_work_entries` | UserError | This work entry is validated. You can't delete it. | `hr_work_entry` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_user` | yes | yes | yes | no | `hr_work_entry` |
| `base.group_system` | yes | yes | yes | yes | `hr_work_entry` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR Work Entry Contract: Multi Company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_view_calendar_multi_create_form` | form |  | `work_entry_type_id`, `duration`, `name`, `employee_id`, `color`, `display_code` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_calendar` | calendar |  | `state`, `name`, `duration`, `display_code`, `employee_id`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_form` | form |  | `state`, `state`, `name`, `work_entry_type_id`, `employee_id`, `date`, `duration`, `company_id` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_calendar_gantt_view_form` | xpath | `hr_work_entry.hr_work_entry_view_form` |  |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_tree` | list |  | `date`, `employee_id`, `work_entry_type_id`, `duration`, `state`, `name`, `code`, `external_code` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_pivot` | pivot |  | `duration`, `employee_id`, `work_entry_type_id` |  |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_view_search` | search |  | `employee_id`, `department_id`, `work_entry_type_id`, `name` |  | `Draft`, `Validated`, `Conflicting`, `Date`, `Current Month`, `Active`, `Archived`, `Employee`, `Department`, `Type`, `Date` | `hr_work_entry` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_action_conflict` | Work Entry | list,form,pivot |  | `{'search_default_work_entries_error': 1}` |  | `hr_work_entry` |
| `hr_work_entry.hr_work_entry_action` | Work Entry | list,form,pivot |  | `{'search_default_active_employees': 1}` |  | `hr_work_entry` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `hr_work_entry.action_hr_work_entry_set_to_draft` | Set to Draft | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/hr.work.entry.json`; views: `../../../schemas/interfaces/views/hr.work.entry.json`.
