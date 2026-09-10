# Change Production Qty (`change.production.qty`)

**Transport name:** `change.production.qty`  
**Storage name:** `change_production_qty`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_subcontracting`

Description: Change Production Qty

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `mo_id` | Manufacturing Order | many to one | `mrp.production` | required; on delete of the target: cascade |
| `product_qty` | Quantity To Produce | float |  | required; precision `Product Unit` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mrp` | model |  |
| `_update_finished_moves` | internal rule | self, production, new_qty, old_qty | `mrp` | model | Update finished product and its byproducts. This method only update the finished moves not done or cancel and just increase or decrease their quantity according the unit_ratio. It does not use the BoM, BoM modification during production would not be taken into consideration. |
| `_need_quantity_propagation` | internal rule | self, move, qty | `mrp_subcontracting`, `mrp` | model |  |
| `change_prod_qty` | operation | self | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_change_production_qty_wizard` | form |  | `product_qty`, `mo_id` | `Set Quantity`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_change_production_qty` | Change Quantity To Produce | form |  |  | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/change.production.qty.json`; views: `../../../schemas/interfaces/views/change.production.qty.json`.
