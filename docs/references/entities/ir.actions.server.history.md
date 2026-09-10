# Server Action History (`ir.actions.server.history`)

**Transport name:** `ir.actions.server.history`  
**Storage name:** `ir_actions_server_history`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Server Action History

## Identity and behavior

- Default ordering: `create_date desc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `action_id` | Action | many to one | `ir.actions.server` | required; on delete of the target: cascade |
| `code` | Code | multi line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `base` |  |  |
| `_gc_histories` | background operation | self | `base` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | no | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.server.history.json`.
