# Client Action (`ir.actions.client`)

**Transport name:** `ir.actions.client`  
**Storage name:** `ir_act_client`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Client Action

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`
- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `type` | Type | single line text |  | default `ir.actions.client` |
| `tag` | Client action tag | single line text |  | required; Help: An arbitrary string, interpreted by the client according to its own needs and wishes. There is no central tag repository across clients. |
| `target` | Target Window | selection |  | default `current` |
| `res_model` | Destination Model | single line text |  | Help: Optional model, mostly used for needactions. |
| `context` | Context Value | single line text |  | required; default `{}`; Help: Context dictionary as Python expression, empty by default (Default: {}) |
| `params` | Supplementary arguments | binary |  | computed by rule `_compute_params` (not stored); writable through an inverse rule; Help: Arguments sent to the client along with the view tag |
| `params_store` | Params storage | binary |  | read only |

## Selection values

### `target` (Target Window)

| Value | Label |
|---|---|
| `current` | Current Window |
| `new` | New Window |
| `fullscreen` | Full Screen |
| `main` | Main action of Current Window |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_params` | computation | self | `base` | depends: `params_store` |  |
| `_inverse_params` | inverse computation | self | `base` |  |  |
| `_get_readable_fields` | preparation rule | self | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_client_action_form` | form |  | `name`, `xml_id`, `binding_type`, `tag`, `type`, `target`, `context`, `help` |  |  | `base` |
| `base.view_client_action_tree` | list |  | `name`, `tag` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_client_actions_report` | Client Actions |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.client.json`; views: `../../../schemas/interfaces/views/ir.actions.client.json`.
