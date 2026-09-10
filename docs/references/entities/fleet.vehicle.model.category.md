# Category of the model (`fleet.vehicle.model.category`)

**Transport name:** `fleet.vehicle.model.category`  
**Storage name:** `fleet_vehicle_model_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`  
**Extended by packages:** `stock_fleet`

Description: Category of the model

## Identity and behavior

- Default ordering: `sequence asc, id asc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `sequence` | Sequence | integer |  |  |
| `weight_capacity` | Max Weight | float |  |  |
| `weight_capacity_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_capacity_uom_name` (not stored) |
| `volume_capacity` | Max Volume | float |  |  |
| `volume_capacity_uom_name` | Volume unit of measure label | single line text |  | computed by rule `_compute_volume_capacity_uom_name` (not stored) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `UNIQUE (name)` | Category name must be unique | `fleet` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `stock_fleet` |  |  |
| `_compute_weight_capacity_uom_name` | computation | self | `stock_fleet` |  |  |
| `_compute_volume_capacity_uom_name` | computation | self | `stock_fleet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_category_view_tree` | list |  | `sequence`, `name` |  |  | `fleet` |
| `fleet.fleet_vehicle_model_category_view_form` | form |  | `name`, `sequence` |  |  | `fleet` |
| `stock_fleet.fleet_vehicle_model_category_view_tree_stock_fleet` | data | `fleet.fleet_vehicle_model_category_view_tree` | `name`, `weight_capacity`, `volume_capacity` |  |  | `stock_fleet` |
| `stock_fleet.fleet_vehicle_model_category_view_form_stock_fleet` | data | `fleet.fleet_vehicle_model_category_view_form` | `name`, `weight_capacity`, `weight_capacity_uom_name`, `volume_capacity`, `volume_capacity_uom_name` |  |  | `stock_fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_model_category_action` | Categories | list |  |  |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.model.category.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.model.category.json`.
