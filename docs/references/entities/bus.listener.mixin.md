# Can send messages via bus.bus (`bus.listener.mixin`)

**Transport name:** `bus.listener.mixin`  
**Storage name:** `bus_listener_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `bus`  
**Extended by packages:** `mail`

Description: Can send messages via bus.bus

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_bus_send` | internal rule | subchannel | `bus` |  | Send a notification to the webclient. |
| `_bus_channel` | internal rule | self | `bus` |  |  |
| `_bus_send_transient_message` | internal rule | self, channel, content | `mail` |  | Posts a fake message in the given ``channel``, only visible for ``self`` listeners. |

Machine-readable definition: `../../../schemas/data/entities/bus.listener.mixin.json`.
