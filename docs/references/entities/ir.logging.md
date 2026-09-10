# Logging (`ir.logging`)

**Transport name:** `ir.logging`  
**Storage name:** `ir_logging`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Logging

## Identity and behavior

- Default ordering: `id DESC`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_uid` | Created by | integer |  | read only |
| `create_date` | Created on | date and time |  | read only |
| `write_uid` | Last Updated by | integer |  | read only |
| `write_date` | Last Updated on | date and time |  | read only |
| `name` | Name | single line text |  | required |
| `type` | Type | selection |  | required; indexed |
| `dbname` | Database Name | single line text |  | indexed |
| `level` | Level | single line text |  | indexed |
| `message` | Message | multi line text |  | required |
| `path` | Path | single line text |  | required |
| `func` | Function | single line text |  | required |
| `line` | Line | single line text |  | required |

## Selection values

### `type` (Type)

| Value | Label |
|---|---|
| `client` | Client |
| `server` | Server |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_logging_form_view` | form |  | `create_date`, `create_uid`, `dbname`, `type`, `name`, `level`, `path`, `line`, `func`, `message` |  |  | `base` |
| `base.ir_logging_tree_view` | list |  | `create_date`, `create_uid`, `dbname`, `type`, `name`, `level`, `path`, `line`, `func` |  |  | `base` |
| `base.ir_logging_search_view` | search |  | `dbname`, `type`, `name`, `level`, `message` |  | `Database`, `Level`, `Type`, `Creation Date` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_logging_all_act` | Logging | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.logging.json`; views: `../../../schemas/interfaces/views/ir.logging.json`.
