# ICE Server (`mail.ice.server`)

**Transport name:** `mail.ice.server`  
**Storage name:** `mail_ice_server`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`

Description: ICE Server

## Identity and behavior

- Display name field: `uri`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `server_type` | Type | selection |  | required; default `stun` |
| `uri` | URI | single line text |  | required |
| `username` | Username | single line text |  |  |
| `credential` | Credential | single line text |  |  |

## Selection values

### `server_type` (Type)

| Value | Label |
|---|---|
| `stun` | stun: |
| `turn` | turn: |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_local_ice_servers` | preparation rule | self | `mail` |  | :return: List of up to 5 dict, each of which representing a stun or turn server |
| `_get_ice_servers` | preparation rule | self | `mail` |  | :return: List of dict, each of which representing a stun or turn server,         formatted as expected by the specifications of RTCConfiguration.iceServers |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.view_ice_server_tree` | list |  | `server_type`, `uri`, `username`, `credential` |  |  | `mail` |
| `mail.view_ice_server_form` | form |  | `server_type`, `uri`, `username`, `credential` |  |  | `mail` |
| `mail.view_ice_server_kanban` | kanban |  | `server_type`, `uri`, `username`, `credential` |  |  | `mail` |
| `mail.view_ice_server_search` | search |  | `uri`, `username`, `credential` |  | `STUN`, `TURN`, `Server Type` | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.action_ice_servers` | ICE Servers | list,form,kanban |  |  |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.ice.server.json`; views: `../../../schemas/interfaces/views/mail.ice.server.json`.
