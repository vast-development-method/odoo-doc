# Fleet Analysis Report (`fleet.vehicle.cost.report`)

**Transport name:** `fleet.vehicle.cost.report`  
**Storage name:** `fleet_vehicle_cost_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Fleet Analysis Report

## Identity and behavior

- Default ordering: `date_start desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | read only |
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` | read only |
| `name` | Vehicle Name | single line text |  | read only |
| `driver_id` | Driver | many to one | `res.partner` | read only |
| `fuel_type` | Fuel | single line text |  | read only |
| `date_start` | Date | date |  | read only |
| `vehicle_type` | Vehicle Type | selection |  | read only |
| `cost` | Cost | float |  | read only |
| `cost_type` | Cost Type | selection |  | read only |

## Selection values

### `vehicle_type` (Vehicle Type)

| Value | Label |
|---|---|
| `car` | Car |
| `bike` | Bike |

### `cost_type` (Cost Type)

| Value | Label |
|---|---|
| `contract` | Contract |
| `service` | Service |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_manager` | no | yes | no | no | `fleet` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Costs Analysis: Multi Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_costs_report_view_search` | search |  | `name`, `driver_id`, `date_start` |  | `Service`, `Contract`, `filter_date_start`, `Vehicle`, `Driver` | `fleet` |
| `fleet.fleet_costs_report_view_pivot` | pivot |  | `date_start`, `cost_type`, `vehicle_id`, `cost` |  |  | `fleet` |
| `fleet.fleet_costs_report_view_graph` | graph |  | `date_start`, `cost_type`, `cost` |  |  | `fleet` |
| `fleet.fleet_vechicle_costs_report_view_tree` | list |  | `name`, `driver_id`, `fuel_type`, `date_start`, `cost`, `cost_type`, `company_id` |  |  | `fleet` |
| `fleet.fleet_vechicle_costs_report_view_form` | form |  | `vehicle_id`, `driver_id`, `fuel_type`, `company_id`, `date_start`, `cost`, `cost_type` |  |  | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_costs_reporting_action` | Costs Analysis | graph,pivot |  | `{'search_default_filter_date_start': 1}` |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.cost.report.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.cost.report.json`.
