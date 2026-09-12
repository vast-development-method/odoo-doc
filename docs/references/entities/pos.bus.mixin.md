# Bus Mixin (`pos.bus.mixin`)

**Transport name:** `pos.bus.mixin`  
**Storage name:** `pos_bus_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `point_of_sale`

Description: Bus Mixin

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `access_token` | Security Token | single line text |  | not copied on duplication |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `point_of_sale` | model_create_multi |  |
| `_ensure_access_token` | internal rule | self | `point_of_sale` |  |  |
| `_notify` | internal rule | self, *notifications, private | `point_of_sale` |  | Send a notification to the bus. ex: one notification: `self._notify('STATUS', {'status': 'closed'})` multiple notifications: `self._notify(('STATUS', {'status': 'closed'}), ('TABLE_ORDER_COUNT', {'count': 2}))` |

Machine-readable definition: `../../../schemas/data/entities/pos.bus.mixin.json`.
