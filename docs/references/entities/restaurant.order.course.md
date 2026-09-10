# point of sale Restaurant Order Course (`restaurant.order.course`)

**Transport name:** `restaurant.order.course`  
**Storage name:** `restaurant_order_course`  
**Kind:** persistent entity (one table)  
**Defined by package:** `pos_restaurant`

Description: POS Restaurant Order Course

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `fired` | Fired | boolean |  | default  |
| `fired_date` | Fired Date | date and time |  |  |
| `uuid` | Uuid | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication |
| `index` | Course index | integer |  | default  |
| `order_id` | Order Ref | many to one | `pos.order` | required; indexed; on delete of the target: cascade |
| `line_ids` | Order Lines | one to many | `pos.order.line` | read only; inverse field `course_id` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `create` | lifecycle override | self, vals_list | `pos_restaurant` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `pos_restaurant` |  |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `pos_restaurant` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `pos_restaurant` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_user` | yes | yes | yes | yes | `pos_restaurant` |

Machine-readable definition: `../../../schemas/data/entities/restaurant.order.course.json`.
