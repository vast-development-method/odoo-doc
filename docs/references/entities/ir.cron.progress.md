# Progress of Scheduled Actions (`ir.cron.progress`)

**Transport name:** `ir.cron.progress`  
**Storage name:** `ir_cron_progress`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Progress of Scheduled Actions

## Identity and behavior

- Display name field: `cron_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `cron_id` | Cron | many to one | `ir.cron` | required; indexed; on delete of the target: cascade |
| `remaining` | Remaining | integer |  | default  |
| `done` | Done | integer |  | default  |
| `deactivate` | Deactivate | boolean |  |  |
| `timed_out_counter` | Timed Out Counter | integer |  | default  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_cron_progress` | background operation | self | `base` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.cron.progress.json`.
