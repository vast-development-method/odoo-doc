# Version (`hr.version`)

**Transport name:** `hr.version`  
**Storage name:** `hr_version`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_attendance`, `hr_holidays`, `hr_work_entry`, `hr_work_entry_holidays`, `l10n_fr_hr_work_entry_holidays`

Description: Version

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `date_version`
- Display name field: `name`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (69)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; default computed dynamically (lambda self: self.env.company); changes are tracked in the message thread |
| `employee_id` | Employee | many to one | `hr.employee` | changes are tracked in the message thread; indexed; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=', company_id)]` |
| `name` | Name | single line text |  | changes are tracked in the message thread |
| `display_name` | Display Name | single line text |  | computed by rule `_compute_display_name` (not stored) |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread |
| `date_version` | Date Version | date |  | required; default computed dynamically (fields.Date.today); changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `last_modified_uid` | Last Modified by | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.uid); visible only to groups `hr.group_hr_user` |
| `last_modified_date` | Last Modified on | date and time |  | required; default computed dynamically (fields.Datetime.now); visible only to groups `hr.group_hr_user` |
| `country_id` | Nationality (Country) | many to one | `res.country` | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `identification_id` | Identification No | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; Help: Enter the employee's National Identification Number issued by the government (e.g., Aadhaar, SIN, NIN). This is used for official records and statutory compliance. |
| `ssnid` | social security number No | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; Help: Social Security Number |
| `passport_id` | Passport No | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `passport_expiration_date` | Passport Expiration Date | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `sex` | Gender | selection |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; Help: This is the legal sex recognized by the state. |
| `private_street` | Private Street | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `private_street2` | Private Street2 | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `private_city` | Private City | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `allowed_country_state_ids` | Allowed Country State | many to many | `res.country.state` | computed by rule `_compute_allowed_country_state_ids` (not stored); visible only to groups `hr.group_hr_user` |
| `private_state_id` | Private State | many to one | `res.country.state` | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; restricted by domain `[('id', 'in', allowed_country_state_ids)]` |
| `private_zip` | Private Zip | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `private_country_id` | Private Country | many to one | `res.country` | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `distance_home_work` | Home-Work Distance | integer |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `km_home_work` | Home-Work Distance in Km | integer |  | computed by rule `_compute_km_home_work` and stored; writable through an inverse rule; changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `distance_home_work_unit` | Home-Work Distance unit | selection |  | required; default `kilometers`; changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `marital` | Marital Status | selection |  | required; default `single`; changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; values provided by rule `_get_marital_status_selection` |
| `spouse_complete_name` | Spouse Legal Name | single line text |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `spouse_birthdate` | Spouse Birthdate | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `children` | Dependent Children | integer |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `employee_type` | Employee Type | selection |  | required; default `employee`; changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `department_id` | Department | many to one | `hr.department` | changes are tracked in the message thread; indexed; must belong to the same company |
| `member_of_department` | Member of department | boolean |  | computed by rule `_compute_part_of_department` (not stored); searchable through a search rule; Help: Whether the employee is a member of the active user's department or one of it's child department. |
| `job_id` | Job | many to one | `hr.job` | changes are tracked in the message thread; indexed; must belong to the same company |
| `job_title` | Job Title | single line text |  | computed by rule `_compute_job_title` and stored; writable through an inverse rule; changes are tracked in the message thread |
| `is_custom_job_title` | Is Custom Job Title | boolean |  | computed by rule `_compute_is_custom_job_title` and stored; default ; visible only to groups `hr.group_hr_user` |
| `address_id` | Work Address | many to one | `res.partner` | default computed dynamically (_get_default_address_id); changes are tracked in the message thread; must belong to the same company |
| `work_location_id` | Work Location | many to one | `hr.work.location` | changes are tracked in the message thread; restricted by domain `[('address_id', '=', address_id)]` |
| `departure_reason_id` | Departure Reason | many to one | `hr.departure.reason` | changes are tracked in the message thread; not copied on duplication; visible only to groups `hr.group_hr_user`; on delete of the target: restrict |
| `departure_description` | Additional Information | rich text |  | not copied on duplication; visible only to groups `hr.group_hr_user` |
| `departure_date` | Departure Date | date |  | changes are tracked in the message thread; not copied on duplication; visible only to groups `hr.group_hr_user` |
| `resource_calendar_id` | Working Hours | many to one | `resource.calendar` | writable through an inverse rule; changes are tracked in the message thread; must belong to the same company |
| `is_flexible` | Is Flexible | boolean |  | computed by rule `_compute_is_flexible` and stored; visible only to groups `hr.group_hr_user` |
| `is_fully_flexible` | Is Fully Flexible | boolean |  | computed by rule `_compute_is_flexible` and stored; visible only to groups `hr.group_hr_user` |
| `tz` | Tz | selection |  | related through path `employee_id.tz` |
| `contract_date_start` | Contract Start Date | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_manager` |
| `contract_date_end` | Contract End Date | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_manager`; Help: End date of the contract (if it's a fixed-term contract). |
| `trial_date_end` | End of Trial Period | date |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_manager`; Help: End date of the trial period (if there is one). |
| `date_start` | Date Start | date |  | computed by rule `_compute_dates` (not stored); searchable through a search rule; visible only to groups `hr.group_hr_manager` |
| `date_end` | Date End | date |  | computed by rule `_compute_dates` (not stored); searchable through a search rule; visible only to groups `hr.group_hr_manager` |
| `is_current` | Is Current | boolean |  | computed by rule `_compute_is_current` (not stored); visible only to groups `hr.group_hr_manager` |
| `is_past` | Is Past | boolean |  | computed by rule `_compute_is_past` (not stored); visible only to groups `hr.group_hr_manager` |
| `is_future` | Is Future | boolean |  | computed by rule `_compute_is_future` (not stored); visible only to groups `hr.group_hr_manager` |
| `is_in_contract` | Is In Contract | boolean |  | computed by rule `_compute_is_in_contract` (not stored); visible only to groups `hr.group_hr_manager` |
| `contract_template_id` | Contract Template | many to one | `hr.version` | changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; restricted by domain `[('company_id', '=', company_id), ('employee_id', '=', False)]`; Help: Select a contract template to auto-fill the contract form with predefined values. You can still edit the fields as needed after applying the template. |
| `structure_type_id` | Salary Structure Type | many to one | `hr.payroll.structure.type` | computed by rule `_compute_structure_type_id` and stored; default computed dynamically (_default_salary_structure); changes are tracked in the message thread; visible only to groups `hr.group_hr_manager` |
| `active_employee` | Active Employee | boolean |  | related through path `employee_id.active`; visible only to groups `hr.group_hr_user` |
| `currency_id` | Currency | many to one |  | read only; related through path `company_id.currency_id` |
| `wage` | Wage | monetary |  | changes are tracked in the message thread; visible only to groups `hr.group_hr_manager`; aggregated with avg; Help: Employee's monthly gross wage. |
| `contract_wage` | Contract Wage | monetary |  | computed by rule `_compute_contract_wage` (not stored); visible only to groups `hr.group_hr_manager` |
| `company_country_id` | Company country | many to one | `res.country` | read only; related through path `company_id.country_id` |
| `country_code` | Country Code | single line text |  | read only; related through path `company_country_id.code` |
| `contract_type_id` | Contract Type | many to one | `hr.contract.type` | changes are tracked in the message thread; visible only to groups `hr.group_hr_manager` |
| `additional_note` | Additional Note | multi line text |  | changes are tracked in the message thread; not copied on duplication; visible only to groups `hr.group_hr_user` |
| `hr_responsible_id` | human resources Responsible | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; visible only to groups `hr.group_hr_user`; restricted by domain `_get_hr_responsible_domain`; Help: Person responsible for validating the employee's contracts. |
| `ruleset_id` | Ruleset | many to one | `hr.attendance.overtime.ruleset` | default computed dynamically (lambda self: self.env.ref('hr_attendance.hr_attendance_default_ruleset', raise_if_not_found=False)); changes are tracked in the message thread; visible only to groups `hr.group_hr_manager`; restricted by domain `_domain_current_countries` |
| `date_generated_from` | Generated From | date and time |  | required; read only; default computed dynamically (lambda self: datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)); changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `date_generated_to` | Generated To | date and time |  | required; read only; default computed dynamically (lambda self: datetime.now().replace(hour=0, minute=0, second=0, microsecond=0)); changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `last_generation_date` | Last Generation Date | date |  | read only; changes are tracked in the message thread; visible only to groups `hr.group_hr_user` |
| `work_entry_source` | Work Entry Source | selection |  | required; default `calendar`; changes are tracked in the message thread; visible only to groups `hr.group_hr_manager`; Help: Defines the source for work entries generation          Working Schedule: Work entries will be generated from the working hours below.         Attendances: Work entries will be generated from the employee's attendances. (requires Attendance app)         Planning: Work entries will be generated from the employee's planning. (requires Planning app) |
| `work_entry_source_calendar_invalid` | Work Entry Source Calendar Invalid | boolean |  | computed by rule `_compute_work_entry_source_calendar_invalid` (not stored); visible only to groups `hr.group_hr_manager` |

## Selection values

### `sex` (Gender)

| Value | Label |
|---|---|
| `male` | Male |
| `female` | Female |
| `other` | Other |

### `distance_home_work_unit` (Home-Work Distance unit)

| Value | Label |
|---|---|
| `kilometers` | km |
| `miles` | mi |

### `employee_type` (Employee Type)

| Value | Label |
|---|---|
| `employee` | Employee |
| `worker` | Worker |
| `student` | Student |
| `trainee` | Trainee |
| `contractor` | Contractor |
| `freelance` | Freelancer |

### `work_entry_source` (Work Entry Source)

| Value | Label |
|---|---|
| `calendar` | Working Schedule |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_contract_start_date_defined` | Constraint | `CHECK(contract_date_end IS NULL OR contract_date_start IS NOT NULL)` | The contract must have a start date. | `hr` |
| `_check_unique_date_version` | UniqueIndex | `(employee_id, date_version) WHERE active = TRUE AND employee_id IS NOT NULL` | An employee cannot have multiple active versions sharing the same effective date. | `hr` |

## Operations (88)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_address_id` | preparation rule | self | `hr` |  |  |
| `_default_salary_structure` | preparation rule | self | `hr` |  |  |
| `_get_hr_responsible_domain` | preparation rule | self | `hr` |  |  |
| `_compute_company_id` | computation | self | `hr` | depends: `employee_id.company_id` |  |
| `_compute_job_title` | computation | self | `hr` | depends: `job_id.name` |  |
| `_inverse_job_title` | inverse computation | self | `hr` |  |  |
| `_compute_is_custom_job_title` | computation | self | `hr` | depends: `job_id` |  |
| `_compute_allowed_country_state_ids` | computation | self | `hr` | depends: `private_country_id` |  |
| `_check_dates` | validation | self | `hr` | constrains: `employee_id`, `contract_date_start`, `contract_date_end` |  |
| `check_contract_finished` | operation | self | `hr` |  |  |
| `create` | lifecycle override | self, vals_list | `hr_holidays`, `hr` | model_create_multi |  |
| `_unlink_except_last_version` | internal rule | self | `hr` | ondelete |  |
| `write` | lifecycle override | self, vals | `hr_holidays`, `hr_work_entry`, `hr` |  |  |
| `get_formview_action` | operation | self, access_uid | `hr` |  | Override this method in order to redirect many2one towards the right model     - Contract template -> hr.version     - Employee record -> hr.employee(.public) with version_id in context |
| `_compute_display_name` | computation | self | `hr` | depends_context: `lang`; depends: `date_version` |  |
| `_compute_is_current` | computation | self | `hr` |  |  |
| `_compute_is_past` | computation | self | `hr` |  |  |
| `_compute_is_future` | computation | self | `hr` |  |  |
| `_compute_is_in_contract` | computation | self | `hr` |  |  |
| `_is_in_contract` | internal rule | self, date | `hr` |  |  |
| `_is_overlapping_period` | internal rule | self, date_from, date_to | `hr` |  | Return True if the employee is at least in contract one day during the period given :param date date_from: the start of the period :param date date_to: the stop of the period |
| `_is_fully_flexible` | internal rule | self | `hr` |  | return True if the version has a fully flexible working calendar |
| `_compute_is_flexible` | computation | self | `hr` | depends: `resource_calendar_id.flexible_hours` |  |
| `_get_whitelist_fields_from_template` | preparation rule | self | `hr_work_entry`, `hr` | model |  |
| `get_values_from_contract_template` | operation | self, contract_template_id | `hr` |  |  |
| `_compute_contract_wage` | computation | self | `hr` | depends: `wage` |  |
| `_get_contract_wage` | preparation rule | self | `hr` |  |  |
| `_get_contract_wage_field` | preparation rule | self | `hr` |  |  |
| `_get_normalized_wage` | preparation rule | self | `hr` |  | This method is overridden in hr_payroll, as without that module, nothing allows to know there's no way to determine the employee's pay frequency. |
| `_get_valid_employee_for_user` | preparation rule | self | `hr` |  |  |
| `_check_ssnid` | validation | self | `hr` | constrains: `ssnid` |  |
| `_compute_part_of_department` | computation | self | `hr` | depends_context: `uid`, `company`; depends: `department_id` |  |
| `_search_part_of_department` | search rule | self, operator, value | `hr` |  |  |
| `_compute_structure_type_id` | computation | self | `hr` | depends: `company_id` |  |
| `_compute_km_home_work` | computation | self | `hr` | depends: `distance_home_work`, `distance_home_work_unit` |  |
| `_inverse_km_home_work` | inverse computation | self | `hr` |  |  |
| `_compute_dates` | computation | self | `hr` | depends: `contract_date_start`, `contract_date_end`, `date_version`, `employee_id`, `employee_id.version_ids.date_version` |  |
| `_search_start_date` | search rule | self, operator, value | `hr` |  |  |
| `_search_end_date` | search rule | self, operator, value | `hr` |  |  |
| `_get_marital_status_selection` | preparation rule | self | `hr` | model |  |
| `_inverse_resource_calendar_id` | inverse computation | self | `hr` |  |  |
| `_get_salary_costs_factor` | preparation rule | self | `hr` |  |  |
| `_is_struct_from_country` | internal rule | self, country_code | `hr` |  |  |
| `_get_tz` | preparation rule | self | `hr` |  |  |
| `action_open_version` | user action | self | `hr` |  |  |
| `action_open_version_form_view` | user action | self | `hr` |  |  |
| `_domain_current_countries` | internal rule | self | `hr_attendance` | model |  |
| `_get_versions_by_employee_and_date` | preparation rule | self, employee_dates | `hr_attendance` | model |  |
| `_check_contracts` | validation | self | `hr_holidays` | constrains: `contract_date_start`, `contract_date_end` |  |
| `_get_leaves` | preparation rule | self, extra_domain | `hr_holidays` |  |  |
| `_get_leaves_from_vals` | preparation rule | self, vals | `hr_holidays` |  |  |
| `_check_overlapping_contract` | validation | self, leave | `hr_holidays` |  |  |
| `_refuse_leave` | internal rule | self, leave, leaves_state | `hr_holidays` |  |  |
| `_set_leave_draft` | internal rule | self, leave, leaves_state | `hr_holidays` |  |  |
| `_populate_all_new_leave_vals_from_split_leave` | internal rule | self, all_new_leave_origin, all_new_leave_vals, overlapping_contracts, leave, leaves_state | `hr_holidays` |  |  |
| `_create_all_new_leave` | internal rule | self, all_new_leave_origin, all_new_leave_vals | `hr_holidays` |  |  |
| `_update_leave_state` | internal rule | self, leave, leaves_state, refuse_leave | `hr_holidays` |  |  |
| `_compute_work_entry_source_calendar_invalid` | computation | self | `hr_work_entry` | depends: `work_entry_source`, `resource_calendar_id` |  |
| `_get_default_work_entry_type_id` | preparation rule | self | `hr_work_entry` |  |  |
| `_get_default_work_entry_type_overtime_id` | preparation rule | self | `hr_work_entry` |  |  |
| `_get_leave_work_entry_type_dates` | preparation rule | self, leave, date_from, date_to, employee | `hr_work_entry` |  |  |
| `_get_leave_work_entry_type` | preparation rule | self, leave | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_get_more_vals_attendance_interval` | preparation rule | self, interval | `hr_work_entry` |  |  |
| `_get_more_vals_leave_interval` | preparation rule | self, interval, leaves | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_get_bypassing_work_entry_type_codes` | preparation rule | self | `hr_work_entry` |  |  |
| `_get_interval_leave_work_entry_type` | preparation rule | self, interval, leaves, bypassing_codes | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_get_sub_leave_domain` | preparation rule | self | `hr_work_entry_holidays`, `hr_work_entry` |  |  |
| `_get_leave_domain` | preparation rule | self, start_dt, end_dt | `hr_work_entry` |  |  |
| `_get_resource_calendar_leaves` | preparation rule | self, start_dt, end_dt | `hr_work_entry` |  |  |
| `_get_attendance_intervals` | preparation rule | self, start_dt, end_dt | `hr_work_entry` |  |  |
| `_get_lunch_intervals` | preparation rule | self, start_dt, end_dt | `hr_work_entry` |  |  |
| `_get_interval_work_entry_type` | preparation rule | self, interval | `hr_work_entry` |  |  |
| `_get_valid_leave_intervals` | preparation rule | self, attendances, interval | `hr_work_entry` |  |  |
| `_get_real_attendance_work_entry_vals` | preparation rule | self, intervals | `hr_work_entry` |  |  |
| `_get_version_work_entries_values` | preparation rule | self, date_start, date_stop | `hr_work_entry`, `l10n_fr_hr_work_entry_holidays` |  |  |
| `_get_real_attendances` | preparation rule | self, attendances, leaves, worked_leaves | `hr_work_entry` |  |  |
| `_get_work_entries_values` | preparation rule | self, date_start, date_stop | `hr_work_entry` |  | Generate a work_entries list between date_start and date_stop for one version. :return: list of dictionnary. |
| `has_static_work_entries` | operation | self | `hr_work_entry` |  |  |
| `generate_work_entries` | operation | self, date_start, date_stop, force | `hr_work_entry` |  |  |
| `_generate_work_entries` | internal rule | self, date_start, date_stop, force | `hr_work_entry` |  |  |
| `_generate_work_entries_postprocess_adapt_to_calendar` | internal rule | self, vals | `hr_work_entry_holidays`, `hr_work_entry` | model |  |
| `_generate_work_entries_postprocess` | internal rule | self, vals_list | `hr_work_entry` | model |  |
| `_remove_work_entries` | internal rule | self | `hr_work_entry` |  | Remove all work_entries that are outside contract period (function used after writing new start or/and end date) |
| `_cancel_work_entries` | internal rule | self | `hr_work_entry` |  |  |
| `unlink` | lifecycle override | self | `hr_work_entry` |  |  |
| `_recompute_work_entries` | internal rule | self, date_from, date_to | `hr_work_entry` |  |  |
| `_get_fields_that_recompute_we` | preparation rule | self | `hr_work_entry` |  |  |
| `_cron_generate_missing_work_entries` | background operation | self | `hr_work_entry` | model |  |

## Validation and error messages (11)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_dates` | ValidationError | Start date (%(start)s) must be earlier than contract end date (%(end)s). | `hr` |
| `_check_dates` | ValidationError | %s already has a contract running during the selected period.  Please either:  - Change the start date so that it doesn't overlap with the existing contract, or - Create a new employee if this employee should have multiple active contracts. | `hr` |
| `check_contract_finished` | ValidationError | Before creating a new contract, close the current one by setting an end date. | `hr` |
| `_unlink_except_last_version` | ValidationError | Employee %s must always have at least one active version. | `hr` |
| `write` | ValidationError | Cannot unassign all the active versions of an employee. | `hr` |
| `write` | ValidationError | Cannot archive all the active versions of an employee. | `hr` |
| `write` | ValidationError | Cannot modify multiple versions contract dates with different contracts at once. | `hr` |
| `create` | ValidationError | Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.  This error has been triggered by: | `hr_holidays` |
| `write` | ValidationError | Changing the contract on this employee changes their working schedule in a period they already took leaves. Changing this working schedule changes the duration of these leaves in such a way the employee no longer has the required allocation for them. Please review these leaves and/or allocations before changing the contract.  This error has been triggered by: | `hr_holidays` |
| `_generate_work_entries_postprocess` | UserError | Missing timezone for work entries generation. | `hr_work_entry` |
| `_generate_work_entries_postprocess` | UserError | Missing date or duration on work entry | `hr_work_entry` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_manager` | yes | yes | yes | yes | `hr` |
| `group_hr_user` | yes | yes | yes | yes | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR Contract: Contract Manager | `[(4, ref('group_hr_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| HR Contract: Multi Company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_version_list_view` | list |  | `date_version`, `contract_date_start`, `contract_date_end`, `employee_id`, `additional_note`, `currency_id`, `wage`, `contract_type_id`, `structure_type_id`, `job_id`, `department_id`, `hr_responsible_id`, `resource_calendar_id`, `company_id`, `create_uid`, `create_date`, `last_modified_uid`, `last_modified_date` | `View` |  | `hr` |
| `hr.hr_version_graph_view` | graph |  | `date_version`, `wage` |  |  | `hr` |
| `hr.hr_version_pivot_view` | pivot |  | `date_version`, `wage` |  |  | `hr` |
| `hr.hr_version_search_view` | search |  | `employee_id`, `job_id`, `department_id`, `resource_calendar_id` |  | `Running Contract`, `Expired Contracts`, `Future Contracts`, `Contract Start Date`, `Contract End Date`, `Archived`, `Late Activities`, `Today Activities`, `Future Activities`, `Employee`, `Job Position`, `Department`, `Working Schedule`, `Salary Structure Type` | `hr` |
| `hr.hr_contract_template_form_view` | form |  | `name`, `job_id`, `department_id`, `hr_responsible_id`, `currency_id`, `wage`, `contract_type_id`, `structure_type_id`, `resource_calendar_id` |  |  | `hr` |
| `hr.hr_contract_template_list_view` | list |  | `name`, `job_id`, `department_id`, `currency_id`, `wage`, `contract_type_id`, `structure_type_id`, `resource_calendar_id`, `company_id`, `create_uid`, `create_date` |  |  | `hr` |
| `hr_work_entry.hr_contract_template_view_form` | separator | `hr.hr_contract_template_form_view` | `work_entry_source` |  |  | `hr_work_entry` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.action_hr_version` | Employee Records | list,graph,pivot | `[('employee_id', '!=', False)]` |  |  | `hr` |
| `hr.action_hr_contract_templates` | Contract Templates | list,form | `[('employee_id', '=', False)]` |  |  | `hr` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `hr_work_entry.ir_cron_generate_missing_work_entries` | Generate Missing Work Entries | 1 days | `_cron_generate_missing_work_entries` |  |

Machine-readable definition: `../../../schemas/data/entities/hr.version.json`; views: `../../../schemas/interfaces/views/hr.version.json`.
