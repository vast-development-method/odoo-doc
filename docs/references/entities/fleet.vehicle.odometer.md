# Odometer log for a vehicle (`fleet.vehicle.odometer`)

**Transport name:** `fleet.vehicle.odometer`  
**Storage name:** `fleet_vehicle_odometer`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `hr_fleet`

Description: Odometer log for a vehicle

## Identity and behavior

- Default ordering: `date desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_vehicle_log_name` and stored |
| `date` | Date | date |  | default computed dynamically (fields.Date.context_today) |
| `value` | Odometer Value | float |  | aggregated with max |
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | required |
| `unit` | Unit | selection |  | read only; related through path `vehicle_id.odometer_unit` |
| `driver_id` | Driver | many to one | `res.partner` | computed by rule `_compute_driver_id` and stored |
| `driver_employee_id` | Driver (Employee) | many to one |  | read only; related through path `vehicle_id.driver_employee_id` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_driver_id` | computation | self | `fleet` | depends: `vehicle_id` |  |
| `_compute_vehicle_log_name` | computation | self | `fleet` | depends: `vehicle_id`, `date` |  |
| `_onchange_vehicle` | on change | self | `fleet` | onchange: `vehicle_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | yes | yes | yes | yes | `fleet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Administrator has all rights on vehicle's vehicle's odometer | `[Command.link(ref('fleet_group_manager'))]` |  | True | True | True | True |
| Fleet odometer: Multi Company | global (all users) | `[('vehicle_id.company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_view_form` | form |  | `vehicle_id`, `value`, `unit`, `date` |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_view_tree` | list |  | `date`, `vehicle_id`, `driver_id`, `value`, `unit` |  |  | `fleet` |
| `fleet.fleet_vehicle_odometer_view_search` | search |  | `vehicle_id`, `driver_id`, `value`, `date` |  | `Vehicle`, `Date` | `fleet` |
| `fleet.fleet_vehicle_odometer_view_graph` | graph |  | `vehicle_id`, `value` |  |  | `fleet` |
| `hr_fleet.fleet_vehicle_odometer_view_tree` | xpath | `fleet.fleet_vehicle_odometer_view_tree` | `driver_employee_id` |  |  | `hr_fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_action` | Odometers | list,form,graph |  |  |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.odometer.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.odometer.json`.
