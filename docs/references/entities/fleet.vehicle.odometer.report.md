# Fleet Odometer Analysis Report (`fleet.vehicle.odometer.report`)

**Transport name:** `fleet.vehicle.odometer.report`  
**Storage name:** `fleet_vehicle_odometer_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Fleet Odometer Analysis Report

## Identity and behavior

- Default ordering: `recorded_date desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | read only |
| `category_id` | Category | many to one |  | related through path `vehicle_id.category_id` |
| `model_id` | Model | many to one |  | related through path `vehicle_id.model_id` |
| `fuel_type` | Fuel Type | selection |  | related through path `vehicle_id.fuel_type` |
| `mileage_delta` | Mileage Delta | float |  | read only |
| `odometer_value` | Odometer Value | float |  | read only |
| `recorded_date` | Date | date |  | read only |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_report_view_search` | search |  |  |  | `Date`, `Vehicle`, `Category`, `Fuel Type`, `Model` | `fleet` |
| `fleet.fleet_vehicle_odometer_report_view_graph` | graph |  | `mileage_delta` |  |  | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_odometer_reporting_action` | Odometer Analysis | graph | `[('vehicle_id.active', '=', True)]` | `{'search_default_groupby_date': 1, 'search_default_groupby_category': 1}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.odometer.report.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.odometer.report.json`.
