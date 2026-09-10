# Action uniform resource locator (`ir.actions.act_url`)

**Transport name:** `ir.actions.act_url`  
**Storage name:** `ir_act_url`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Action URL

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`
- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `type` | Type | single line text |  | default `ir.actions.act_url` |
| `url` | Action uniform resource locator | multi line text |  | required |
| `target` | Action Target | selection |  | required; default `new` |

## Selection values

### `target` (Action Target)

| Value | Label |
|---|---|
| `new` | New Window |
| `self` | This Window |
| `download` | Download |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_readable_fields` | preparation rule | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.act_url.json`.
