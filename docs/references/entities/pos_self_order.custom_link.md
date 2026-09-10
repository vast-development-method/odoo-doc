# Custom links that the restaurant can configure to be displayed on the self order screen (`pos_self_order.custom_link`)

**Transport name:** `pos_self_order.custom_link`  
**Storage name:** `pos_self_order_custom_link`  
**Kind:** persistent entity (one table)  
**Defined by package:** `pos_self_order`

Description: Custom links that the restaurant can configure to be displayed on the self order screen

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Label | single line text |  | required; translatable |
| `url` | uniform resource locator | single line text |  | required |
| `pos_config_ids` | Points of Sale | many to many | `pos.config` | restricted by domain `[('self_ordering_mode', '!=', 'nothing')]`; Help: Select for which points of sale you want to display this link. Leave empty to display it for all points of sale. You have to select among the points of sale that have the 'QR Code Menu' feature enabled. |
| `style` | Style | selection |  | required; default `primary` |
| `link_html` | Preview | rich text |  | read only; computed by rule `_compute_link_html` and stored |
| `sequence` | Sequence | integer |  | default `1` |

## Selection values

### `style` (Style)

| Value | Label |
|---|---|
| `primary` | Primary |
| `secondary` | Secondary |
| `success` | Success |
| `warning` | Warning |
| `danger` | Danger |
| `info` | Info |
| `light` | Light |
| `dark` | Dark |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_self_data_domain` | internal rule | self, data, config | `pos_self_order` | model |  |
| `_load_pos_self_data_fields` | internal rule | self, config | `pos_self_order` | model |  |
| `_compute_link_html` | computation | self | `pos_self_order` | depends: `name`, `style` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_self_order` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_self_order` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `pos_self_order.custom_link_tree` | list |  | `sequence`, `name`, `url`, `pos_config_ids`, `style`, `link_html` |  |  | `pos_self_order` |

Machine-readable definition: `../../../schemas/data/entities/pos_self_order.custom_link.json`; views: `../../../schemas/interfaces/views/pos_self_order.custom_link.json`.
