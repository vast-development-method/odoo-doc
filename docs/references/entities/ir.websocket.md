# websocket message handling (`ir.websocket`)

**Transport name:** `ir.websocket`  
**Storage name:** `ir_websocket`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `bus`  
**Extended by packages:** `html_editor`, `mail`, `mail`, `auth_timeout`, `im_livechat`, `hr_presence`

Description: websocket message handling

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_build_bus_channel_list` | internal rule | self, channels | `bus`, `html_editor`, `im_livechat`, `mail` |  | Return the list of channels to subscribe to. Override this method to add channels in addition to the ones the client sent.  :param channels: The channel list sent by the client. |
| `_serve_ir_websocket` | internal rule | self, event_name, data | `bus`, `mail` |  | Process websocket events. Modules can override this method to handle their own events. But overriding this method is not recommended and should be carefully considered, because at the time of writing this message, the system.sh does not use this method. Each new event should have a corresponding http route and the system.sh infrastructure should be updated to reflect it. On top of that, the event processing is very time, ressource and error sensitive. |
| `_prepare_subscribe_data` | preparation rule | self, channels, last | `bus`, `mail` |  | Parse the data sent by the client and return the list of channels and the last known notification id. This will be used both by the websocket controller and the websocket request class when the `subscribe` event is received.  :param typing.List[str] channels: List of channels to subscribe to sent     by the client. :param int last: Last known notification sent by the client.  :return:     A dict containing the following keys:     - channels (set of str): The list of channels to subscribe to.     - last (int): The last known notification id.  :raise ValueError: If the list of channels is not a  |
| `_after_subscribe_data` | internal rule | self, data | `bus`, `mail` |  | Function invoked after subscribe data have been processed. Modules can override this method to add custom behavior. |
| `_subscribe` | internal rule | self, og_data | `bus`, `mail` |  |  |
| `_on_websocket_closed` | internal rule | self, cookies | `auth_timeout`, `bus`, `mail` |  | Function invoked upon WebSocket termination. Modules can override this method to add custom behavior. |
| `_authenticate` | internal rule | cls | `bus` |  |  |
| `_update_mail_presence` | internal rule | self, inactivity_period | `auth_timeout`, `hr_presence`, `mail` |  | Override to track user inactivity via WebSocket presence updates.  This method extends the base `_update_mail_presence` to update the session's inactivity state using the provided inactivity duration from the frontend.  :param float inactivity_period: Duration of user inactivity in milliseconds. :return: None |

Machine-readable definition: `../../../schemas/data/entities/ir.websocket.json`.
