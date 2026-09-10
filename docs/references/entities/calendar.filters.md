# Calendar Filters (`calendar.filters`)

**Transport name:** `calendar.filters`  
**Storage name:** `calendar_filters`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`

Description: Calendar Filters

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `user_id` | Me | many to one | `res.users` | required; default computed dynamically (lambda self: self.env.user); indexed; on delete of the target: cascade |
| `partner_id` | Employee | many to one | `res.partner` | required; indexed |
| `active` | Active | boolean |  | default `True` |
| `partner_checked` | Checked | boolean |  | default `True` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_user_id_partner_id_unique` | Constraint | `UNIQUE(user_id, partner_id)` | A user cannot have the same contact twice. | `calendar` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `unlink_from_partner_id` | operation | self, partner_id | `calendar` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `calendar` |
| `base.group_system` | yes | yes | yes | yes | `calendar` |

Machine-readable definition: `../../../schemas/data/entities/calendar.filters.json`.
