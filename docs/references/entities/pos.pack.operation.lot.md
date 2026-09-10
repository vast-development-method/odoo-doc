# Specify product lot/serial number in pos order line (`pos.pack.operation.lot`)

**Transport name:** `pos.pack.operation.lot`  
**Storage name:** `pos_pack_operation_lot`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`

Description: Specify product lot/serial number in pos order line

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Display name field: `lot_name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pos_order_line_id` | Point of sale Order Line | many to one | `pos.order.line` | indexed (btree_not_null) |
| `order_id` | Order | many to one | `pos.order` | related through path `pos_order_line_id.order_id` |
| `lot_name` | Lot Name | single line text |  |  |
| `product_id` | Product | many to one | `product.product` | related through path `pos_order_line_id.product_id` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.pack.operation.lot.json`.
