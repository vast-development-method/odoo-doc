# Action Window View (`ir.actions.act_window.view`)

**Transport name:** `ir.actions.act_window.view`  
**Storage name:** `ir_act_window_view`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `web_hierarchy`

Description: Action Window View

## Identity and behavior

- Default ordering: `sequence,id`
- Display name field: `view_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  |  |
| `view_id` | View | many to one | `ir.ui.view` |  |
| `view_mode` | View Type | selection |  | required; on delete of the target: {"hierarchy": "cascade"}; extended by packages `mail`, `web_hierarchy` |
| `act_window_id` | Action | many to one | `ir.actions.act_window` | indexed (btree_not_null); on delete of the target: cascade |
| `multi` | On Multiple Doc. | boolean |  | Help: If set to true, the action will not be displayed on the right toolbar of a form view. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_mode_per_action` | UniqueIndex | `(act_window_id, view_mode)` |  | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.act_window.view.json`.
