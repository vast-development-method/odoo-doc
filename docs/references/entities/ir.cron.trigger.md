# Triggered actions (`ir.cron.trigger`)

**Transport name:** `ir.cron.trigger`  
**Storage name:** `ir_cron_trigger`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Triggered actions

## Identity and behavior

- Display name field: `cron_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `cron_id` | Cron | many to one | `ir.cron` | required; indexed; on delete of the target: cascade |
| `call_at` | Call At | date and time |  | required; indexed |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_cron_triggers` | background operation | self | `base` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_cron_trigger_view_form` | form |  | `cron_id`, `call_at` |  |  | `base` |
| `base.ir_cron_trigger_view_tree` | list |  | `cron_id`, `call_at` |  |  | `base` |
| `base.ir_cron_trigger_view_search` | search |  | `cron_id`, `call_at` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_cron_trigger_action` | Scheduled Actions Triggers | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.cron.trigger.json`; views: `../../../schemas/interfaces/views/ir.cron.trigger.json`.
