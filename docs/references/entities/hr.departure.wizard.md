# Departure Wizard (`hr.departure.wizard`)

**Transport name:** `hr.departure.wizard`  
**Storage name:** `hr_departure_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `hr`  
**Extended by packages:** `hr_fleet`, `hr_holidays`, `hr_maintenance`

Description: Departure Wizard

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `departure_reason_id` | Departure Reason | many to one | `hr.departure.reason` | required; default computed dynamically (lambda self: self.env['hr.departure.reason'].search([], limit=1)) |
| `departure_description` | Additional Information | rich text |  |  |
| `departure_date` | Contract End Date | date |  | required; default computed dynamically (_get_default_departure_date) |
| `employee_ids` | Employees | many to many | `hr.employee` | required; default computed dynamically (_get_default_employee_ids); restricted by domain `_get_domain_employee_ids` |
| `is_user_employee` | User Employee | boolean |  | computed by rule `_compute_is_user_employee` (not stored) |
| `remove_related_user` | Related User | boolean |  | Help: If checked, the related user will be removed from the system. |
| `set_date_end` | Set Contract End Date | boolean |  | default computed dynamically (lambda self: self.env.user.has_group('hr.group_hr_user')); Help: Set the end date on the current contract. |
| `release_campany_car` | Release Company Car | boolean |  | default computed dynamically (lambda self: self.env.user.has_group('fleet.fleet_group_user')) |
| `unassign_equipment` | Free Equiments | boolean |  | default `True`; Help: Unassign Employee from Equipments |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_departure_date` | preparation rule | self | `hr` |  |  |
| `_get_default_employee_ids` | preparation rule | self | `hr` |  |  |
| `_get_domain_employee_ids` | preparation rule | self | `hr` |  |  |
| `_compute_is_user_employee` | computation | self | `hr` | depends: `employee_ids.user_id` |  |
| `action_register_departure` | user action | self | `hr_fleet`, `hr_holidays`, `hr_maintenance`, `hr` |  |  |
| `_free_company_car` | internal rule | self | `hr_fleet` |  | Find all fleet.vehichle.assignation.log records that link to the employee, if there is no  end date or end date > departure date, update the date. Also check fleet.vehicle to see if  there is any record with its driver_id to be the employee, set them to False. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_register_departure` | UserError | Departure date can't be earlier than the start date of current contract. | `hr` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | no | `hr` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_departure_wizard_view_form` | form |  | `employee_ids`, `departure_reason_id`, `departure_date`, `set_date_end`, `remove_related_user`, `departure_description` | `Apply`, `Discard` |  | `hr` |
| `hr_fleet.hr_departure_wizard_view_form` | xpath | `hr.hr_departure_wizard_view_form` |  |  |  | `hr_fleet` |
| `hr_maintenance.hr_departure_wizard_view_form` | xpath | `hr.hr_departure_wizard_view_form` |  |  |  | `hr_maintenance` |

Machine-readable definition: `../../../schemas/data/entities/hr.departure.wizard.json`; views: `../../../schemas/interfaces/views/hr.departure.wizard.json`.
