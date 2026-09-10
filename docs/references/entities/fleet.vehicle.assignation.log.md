# Drivers history on a vehicle (`fleet.vehicle.assignation.log`)

**Transport name:** `fleet.vehicle.assignation.log`  
**Storage name:** `fleet_vehicle_assignation_log`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `hr_fleet`

Description: Drivers history on a vehicle

## Identity and behavior

- Default ordering: `create_date desc, date_start desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | required; indexed |
| `driver_id` | Driver | many to one | `res.partner` | required |
| `date_start` | Start Date | date |  |  |
| `date_end` | End Date | date |  |  |
| `driver_employee_id` | Driver (Employee) | many to one | `hr.employee` | computed by rule `_compute_driver_employee_id` and stored |
| `attachment_number` | Number of Attachments | integer |  | computed by rule `_compute_attachment_number` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `fleet` | depends: `driver_id`, `vehicle_id` |  |
| `_compute_driver_employee_id` | computation | self | `hr_fleet` | depends: `driver_id` |  |
| `_compute_attachment_number` | computation | self | `hr_fleet` |  |  |
| `action_get_attachment_view` | user action | self | `hr_fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet.fleet_group_user` | yes | yes | yes | yes | `fleet` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_assignation_log_view_list` | list |  | `vehicle_id`, `driver_id`, `date_start`, `date_end` |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_assignation_log_view_list` | field | `fleet.fleet_vehicle_assignation_log_view_list` | `vehicle_id` |  |  | `hr_fleet` |
| `hr_fleet.fleet_vehicle_assignation_log_employee_view_list` | field | `fleet.fleet_vehicle_assignation_log_view_list` | `driver_id` |  |  | `hr_fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.assignation.log.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.assignation.log.json`.
