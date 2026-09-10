# Vehicle Tag (`fleet.vehicle.tag`)

**Transport name:** `fleet.vehicle.tag`  
**Storage name:** `fleet_vehicle_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `fleet`

Description: Vehicle Tag

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required; translatable |
| `color` | Color | integer |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `fleet` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `fleet_group_user` | no | yes | no | no | `fleet` |
| `fleet_group_manager` | yes | yes | yes | yes | `fleet` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_tag_view_view_form` | form |  | `name` |  |  | `fleet` |
| `fleet.fleet_vehicle_tag_view_view_tree` | list |  | `name`, `color` |  |  | `fleet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.fleet_vehicle_tag_action` | Tags |  |  |  |  | `fleet` |

Machine-readable definition: `../../../schemas/data/entities/fleet.vehicle.tag.json`; views: `../../../schemas/interfaces/views/fleet.vehicle.tag.json`.
