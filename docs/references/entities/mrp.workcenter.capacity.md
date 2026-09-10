# Work Center Capacity (`mrp.workcenter.capacity`)

**Transport name:** `mrp.workcenter.capacity`  
**Storage name:** `mrp_workcenter_capacity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Work Center Capacity

## Identity and behavior

- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `workcenter_id` | Work Center | many to one | `mrp.workcenter` | required; indexed |
| `product_id` | Product | many to one | `product.product` |  |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; precomputed before insertion |
| `capacity` | Capacity | float |  | Help: Number of pieces that can be produced in parallel for this product or for all, depending on the unit. |
| `time_start` | Setup Time (minutes) | float |  | default computed dynamically (_default_time_start); Help: Time in minutes for the setup. |
| `time_stop` | Cleanup Time (minutes) | float |  | default computed dynamically (_default_time_stop); Help: Time in minutes for the cleaning. |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_positive_capacity` | Constraint | `CHECK(capacity >= 0)` | Capacity should be a non-negative number. | `mrp` |
| `_workcenter_product_product_uom_unique` | UniqueIndex | `(workcenter_id, COALESCE(product_id, 0), product_uom_id)` | Product/Unit capacity should be unique for each workcenter. | `mrp` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_time_start` | preparation rule | self | `mrp` |  |  |
| `_default_time_stop` | preparation rule | self | `mrp` |  |  |
| `_compute_product_uom_id` | computation | self | `mrp` | depends: `product_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.workcenter.capacity.json`.
