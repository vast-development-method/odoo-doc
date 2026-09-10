# Vehicle Status (`fleet.vehicle.state`)

**Transport name:** `fleet.vehicle.state`  
**Storage name:** `fleet_vehicle_state`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Vehicle Status

## Identity and behavior

- Default ordering: `sequence asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  |  |
| `fold` | Folded in Kanban | boolean |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_fleet_state_name_unique` | Constraint | `unique(name)` | State name already exists | `fleet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_state_view_tree` | list |  | `sequence`, `name`, `fold` |  |  | `fleet` |
| `fleet.fleet_vehicle_state_view_form` | form |  | `name`, `sequence`, `fold` |  |  | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_state_action` | Status | list,form |  |  |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.state.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.state.json`.
