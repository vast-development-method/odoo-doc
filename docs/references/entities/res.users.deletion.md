# Users Deletion Request (`res.users.deletion`)

**Transport name:** `res.users.deletion`  
**Storage name:** `res_users_deletion`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Users Deletion Request

## Identity and behavior

- Display name field: `user_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | User | many to one | `res.users` | on delete of the target: set null |
| `user_id_int` | User Id | integer |  | computed by rule `_compute_user_id_int` and stored |
| `state` | State | selection |  | required; default `todo` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `todo` | To Do |
| `done` | Done |
| `fail` | Failed |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_user_id_int` | computation | self | `base` | depends: `user_id` |  |
| `_gc_portal_users` | background operation | self, batch_size | `base` | model | Remove the portal users that asked to deactivate their account.  (see <res.users>::_deactivate_portal_user)  Removing a user can be an heavy operation on large database (because of create_uid, write_uid on each models, which are not always indexed). Because of that, this operation is done in a CRON. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `base` |
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `base.ir_cron_res_users_deletion` | Base: Portal Users Deletion | 1 days | `_gc_portal_users` | 8 |

Machine-readable definition: `../../../schemas/data/entities/res.users.deletion.json`.
