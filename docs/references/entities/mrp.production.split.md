# Wizard to Split a Production (`mrp.production.split`)

**Transport name:** `mrp.production.split`  
**Storage name:** `mrp_production_split`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mrp`

Description: Wizard to Split a Production

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `production_split_multi_id` | Split Productions | many to one | `mrp.production.split.multi` |  |
| `production_id` | Manufacturing Order | many to one | `mrp.production` | read only |
| `product_id` | Product | many to one |  | related through path `production_id.product_id` |
| `product_qty` | Product Qty | float |  | related through path `production_id.product_qty` |
| `product_uom_id` | Product Unit of measure | many to one |  | related through path `production_id.product_uom_id` |
| `production_capacity` | Production Capacity | float |  | related through path `production_id.production_capacity` |
| `production_detailed_vals_ids` | Split Details | one to many | `mrp.production.split.line` | computed by rule `_compute_details` and stored; inverse field `mrp_production_split_id` |
| `valid_details` | Valid | boolean |  | computed by rule `_compute_valid_details` (not stored) |
| `max_batch_size` | Max Batch Size | float |  | computed by rule `_compute_max_batch_size` (not stored); precision `Product Unit` |
| `num_splits` | # Splits | integer |  | read only; computed by rule `_compute_num_splits` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_max_batch_size` | computation | self | `mrp` | depends: `production_id` |  |
| `_compute_num_splits` | computation | self | `mrp` | depends: `max_batch_size` |  |
| `_compute_details` | computation | self | `mrp` | depends: `num_splits` |  |
| `_compute_valid_details` | computation | self | `mrp` | depends: `production_detailed_vals_ids` |  |
| `action_split` | user action | self | `mrp` |  |  |
| `action_prepare_split` | user action | self | `mrp` |  |  |
| `action_return_to_list` | user action | self | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | yes | yes | yes | no | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.view_mrp_production_split_form` | form |  | `production_id`, `product_id`, `product_qty`, `product_uom_id`, `max_batch_size`, `product_uom_id`, `num_splits`, `production_detailed_vals_ids`, `date`, `user_id`, `quantity`, `production_split_multi_id`, `valid_details` | `Split`, `Discard`, `Discard` |  | `mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.action_mrp_production_split` | Split production | form |  |  | new | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.production.split.json`; views: `../../../schemas/interfaces/views/mrp.production.split.json`.
