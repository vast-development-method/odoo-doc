# Action Window Close (`ir.actions.act_window_close`)

**Transport name:** `ir.actions.act_window_close`  
**Storage name:** `ir_actions`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Action Window Close

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `type` | Type | single line text |  | default `ir.actions.act_window_close` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_readable_fields` | preparation rule | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.act_window_close.json`.
