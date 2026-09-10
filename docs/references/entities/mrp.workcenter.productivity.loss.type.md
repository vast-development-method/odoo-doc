# manufacturing Workorder productivity losses (`mrp.workcenter.productivity.loss.type`)

**Transport name:** `mrp.workcenter.productivity.loss.type`  
**Storage name:** `mrp_workcenter_productivity_loss_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: MRP Workorder productivity losses

## Identity and behavior

- Display name field: `loss_type`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `loss_type` | Category | selection |  | required; default `availability` |

## Selection values

### `loss_type` (Category)

| Value | Label |
|---|---|
| `availability` | Availability |
| `performance` | Performance |
| `quality` | Quality |
| `productive` | Productive |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `mrp` |  | As 'category' field in form view is a Many2one, its value will be in lower case. In order to display its value capitalized 'display_name' is overrided. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.productivity.loss.type.json`.
