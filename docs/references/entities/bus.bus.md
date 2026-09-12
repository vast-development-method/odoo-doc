# Communication Bus (`bus.bus`)

**Transport name:** `bus.bus`  
**Storage name:** `bus_bus`  
**Kind:** persistent entity (one table)  
**Defined by package:** `bus`

Description: Communication Bus

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `channel` | Channel | single line text |  |  |
| `message` | Message | single line text |  |  |
| `create_date` | Create Date | date and time |  | indexed |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_messages` | background operation | self | `bus` | autovacuum |  |
| `_sendone` | internal rule | self, target, notification_type, message | `bus` | model | Low-level method to send `notification_type` and `message` to `target`.  Using `_bus_send()` from `bus.listener.mixin` is recommended for simplicity and security.  When using `_sendone` directly, `target` (if str) should not be guessable by an attacker. |
| `_ensure_hooks` | internal rule | self | `bus` |  |  |
| `_poll` | internal rule | self, channels, last, ignore_ids | `bus` | model |  |
| `_bus_last_id` | internal rule | self | `bus` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `bus` |

Machine-readable definition: `../../../schemas/data/entities/bus.bus.json`.
