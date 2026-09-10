# Users Log (`res.users.log`)

**Transport name:** `res.users.log`  
**Storage name:** `res_users_log`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `hr_presence`

Description: Users Log

## Identity and behavior

- Default ordering: `id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `create_uid` | Created by | many to one | `res.users` | read only; indexed |
| `ip` | internet protocol Address | single line text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_gc_user_logs` | background operation | self | `base` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | no | no | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| res.users.log per user | global (all users) | `[('create_uid','=', user.id)]` | False | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/res.users.log.json`.
