# User/Guest Presence (`mail.presence`)

**Transport name:** `mail.presence`  
**Storage name:** `mail_presence`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: User/Guest Presence

## Identity and behavior

- Mixins (classical inheritance): `bus.listener.mixin`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | Users | many to one | `res.users` | on delete of the target: cascade |
| `guest_id` | Guest | many to one | `mail.guest` | on delete of the target: cascade |
| `last_poll` | Last Poll | date and time |  | default computed dynamically (lambda self: fields.Datetime.now()) |
| `last_presence` | Last Presence | date and time |  | default computed dynamically (lambda self: fields.Datetime.now()) |
| `status` | IM Status | selection |  | default `offline` |

## Selection values

### `status` (IM Status)

| Value | Label |
|---|---|
| `online` | Online |
| `away` | Away |
| `offline` | Offline |

## State fields

State machine fields of this entity: `status`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_guest_unique` | UniqueIndex | `(guest_id) WHERE guest_id IS NOT NULL` |  | `mail` |
| `_user_unique` | UniqueIndex | `(user_id) WHERE user_id IS NOT NULL` |  | `mail` |
| `_partner_or_guest_exists` | Constraint | `CHECK((user_id IS NOT NULL AND guest_id IS NULL) OR (user_id IS NULL AND guest_id IS NOT NULL))` | A mail presence must have a user or a guest. | `mail` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_try_update_presence` | internal rule | self, user_or_guest, inactivity_period | `mail` | model | Updates the last_poll and last_presence of the current user :param inactivity_period: duration in milliseconds |
| `_update_presence` | internal rule | self, user_or_guest, inactivity_period | `mail` | model |  |
| `_send_presence` | internal rule | self, im_status, bus_target | `mail` |  | Send notification related to bus presence update.  :param im_status: 'online', 'away' or 'offline' |
| `_send_status_updated_notification` | internal rule | self, guest_or_user, status, bus_target | `mail` | model |  |
| `_gc_bus_presence` | background operation | self | `mail` | autovacuum |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.presence.json`.
