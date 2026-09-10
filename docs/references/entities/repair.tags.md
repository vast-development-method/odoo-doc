# Repair Tags (`repair.tags`)

**Transport name:** `repair.tags`  
**Storage name:** `repair_tags`  
**Kind:** persistent entity (one table)  
**Defined by package:** `repair`

Description: Repair Tags

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Tag Name | single line text |  | required |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `repair` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `repair` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `repair` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `repair.view_repair_tag_tree` | list |  | `name`, `color` |  |  | `repair` |
| `repair.view_repair_tag_search` | search |  | `name` |  |  | `repair` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `repair.action_repair_order_tag` | Tags |  |  |  |  | `repair` |

Machine-readable definition: `../../../schemas/data/entities/repair.tags.json`; views: `../../../schemas/interfaces/views/repair.tags.json`.
