# Technical model to group purchase order for call to tenders (`purchase.order.group`)

**Transport name:** `purchase.order.group`  
**Storage name:** `purchase_order_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase_requisition`

Description: Technical model to group PO for call to tenders

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (1)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `order_ids` | Order | one to many | `purchase.order` | inverse field `purchase_group_id` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `write` | lifecycle override | self, vals | `purchase_requisition` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |

Machine-readable definition: `../../../schemas/data/entities/purchase.order.group.json`.
