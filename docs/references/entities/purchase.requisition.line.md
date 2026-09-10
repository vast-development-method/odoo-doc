# Purchase Requisition Line (`purchase.requisition.line`)

**Transport name:** `purchase.requisition.line`  
**Storage name:** `purchase_requisition_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase_requisition`  
**Extended by packages:** `purchase_requisition_stock`

Description: Purchase Requisition Line

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Display name field: `product_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | required; restricted by domain `[["purchase_ok", "=", true]]` |
| `product_uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_product_uom_id` and stored; precomputed before insertion |
| `product_qty` | Quantity | float |  | precision `Product Unit` |
| `product_description_variants` | Description | single line text |  |  |
| `price_unit` | Unit Price | float |  | computed by rule `_compute_price_unit` and stored; default  |
| `qty_ordered` | Ordered | float |  | computed by rule `_compute_ordered_qty` (not stored) |
| `requisition_id` | Purchase Agreement | many to one | `purchase.requisition` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | read only; related through path `requisition_id.company_id` and stored |
| `supplier_info_ids` | Supplier Info | one to many | `product.supplierinfo` | inverse field `purchase_requisition_line_id` |
| `move_dest_id` | Downstream Move | many to one | `stock.move` | indexed (btree_not_null) |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_ordered_qty` | computation | self | `purchase_requisition` | depends: `requisition_id.purchase_ids.state` |  |
| `_compute_product_uom_id` | computation | self | `purchase_requisition` | depends: `product_id` |  |
| `_compute_price_unit` | computation | self | `purchase_requisition` | depends: `product_id`, `company_id`, `requisition_id.date_start`, `product_qty`, `product_uom_id`, `requisition_id.vendor_id`, `requisition_id.requisition_type` |  |
| `create` | lifecycle override | self, vals_list | `purchase_requisition` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `purchase_requisition` |  |  |
| `unlink` | lifecycle override | self | `purchase_requisition` |  |  |
| `_create_supplier_info` | internal rule | self | `purchase_requisition` |  |  |
| `_prepare_purchase_order_line` | preparation rule | self, name, product_qty, price_unit, taxes_ids | `purchase_requisition_stock`, `purchase_requisition` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `create` | UserError | You cannot have a negative or unit price of 0 for an already confirmed blanket order. | `purchase_requisition` |
| `write` | UserError | You cannot have a negative or unit price of 0 for an already confirmed blanket order. | `purchase_requisition` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_requisition` |
| `purchase.group_purchase_manager` | no | yes | no | no | `purchase_requisition` |
| `stock.group_stock_manager` | yes | yes | no | no | `purchase_requisition_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase requisition Line multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/purchase.requisition.line.json`.
