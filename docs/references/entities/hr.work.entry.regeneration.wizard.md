# Regenerate Employee Work Entries (`hr.work.entry.regeneration.wizard`)

**Transport name:** `hr.work.entry.regeneration.wizard`  
**Storage name:** `hr_work_entry_regeneration_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr_work_entry`

Description: Regenerate Employee Work Entries

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `earliest_available_date` | Earliest date | date |  | computed by rule `_compute_earliest_available_date` (not stored) |
| `earliest_available_date_message` | Earliest Available Date Message | single line text |  | read only; default  |
| `latest_available_date` | Latest date | date |  | computed by rule `_compute_latest_available_date` (not stored) |
| `latest_available_date_message` | Latest Available Date Message | single line text |  | read only; default  |
| `date_from` | From | date |  | required; default computed dynamically (lambda self: self.env.context.get('date_start')) |
| `date_to` | To | date |  | required; computed by rule `_compute_date_to` and stored; default computed dynamically (lambda self: self.env.context.get('date_end')) |
| `employee_ids` | Employees | many to many | `hr.employee` | required; restricted by domain `lambda self: [('company_id', 'in', self.env.companies.ids)]` |
| `validated_work_entry_employee_ids` | Validated Work Entry Employee | many to many | `hr.employee` | computed by rule `_compute_validated_work_entry_employee_ids` (not stored) |
| `search_criteria_completed` | Search Criteria Completed | boolean |  | computed by rule `_compute_search_criteria_completed` (not stored) |
| `valid` | Valid | boolean |  | computed by rule `_compute_valid` (not stored) |

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_date_to` | computation | self | `hr_work_entry` | depends: `date_from` |  |
| `_compute_earliest_available_date` | computation | self | `hr_work_entry` | depends: `employee_ids` |  |
| `_compute_latest_available_date` | computation | self | `hr_work_entry` | depends: `employee_ids` |  |
| `_compute_validated_work_entry_employee_ids` | computation | self | `hr_work_entry` | depends: `date_from`, `date_to`, `employee_ids` |  |
| `_compute_valid` | computation | self | `hr_work_entry` | depends: `validated_work_entry_employee_ids`, `employee_ids` |  |
| `_compute_search_criteria_completed` | computation | self | `hr_work_entry` | depends: `date_from`, `date_to`, `employee_ids` |  |
| `_check_dates` | on change | self | `hr_work_entry` | onchange: `date_from`, `date_to`, `employee_ids` |  |
| `_date_to_string` | internal rule | self, date | `hr_work_entry` | model |  |
| `_work_entry_fields_to_nullify` | internal rule | self | `hr_work_entry` |  |  |
| `regenerate_work_entries` | operation | self, slots, record_ids | `hr_work_entry` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `regenerate_work_entries` | ValidationError | In order to regenerate the work entries, you need to provide the wizard with an employee_id, a date_from and a date_to. | `hr_work_entry` |
| `regenerate_work_entries` | ValidationError | The from date must be >= '%(earliest_available_date)s' and the to date must be <= '%(latest_available_date)s', which correspond to the generated work entries time interval. | `hr_work_entry` |
| `regenerate_work_entries` | ValidationError | No work entry can be regenerated in this range of dates and these employees. | `hr_work_entry` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr.group_hr_manager` | yes | yes | yes | yes | `hr_work_entry` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_regeneration_wizard` | form |  | `employee_ids`, `date_to`, `date_from`, `earliest_available_date_message`, `latest_available_date_message`, `search_criteria_completed` | `Regenerate Work Entries`, `Regenerate Work Entries`, `Cancel` |  | `hr_work_entry` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_work_entry.hr_work_entry_regeneration_wizard_action` | Work Entry Regeneration | form |  |  | new | `hr_work_entry` |

Machine-readable definition: `../../../schemas/data/entities/hr.work.entry.regeneration.wizard.json`; views: `../../../schemas/interfaces/views/hr.work.entry.regeneration.wizard.json`.
