# Storage Category (`stock.storage.category`)

**Transport name:** `stock.storage.category`  
**Storage name:** `stock_storage_category`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`

Description: Storage Category

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Storage Category | single line text |  | required |
| `max_weight` | Max Weight | float |  | precision `Stock Weight` |
| `capacity_ids` | Capacity | one to many | `stock.storage.category.capacity` | inverse field `storage_category_id` |
| `product_capacity_ids` | Product Capacity | one to many | `stock.storage.category.capacity` | computed by rule `_compute_storage_capacity_ids` (not stored); writable through an inverse rule |
| `package_capacity_ids` | Package Capacity | one to many | `stock.storage.category.capacity` | computed by rule `_compute_storage_capacity_ids` (not stored); writable through an inverse rule |
| `allow_new_product` | Allow New Product | selection |  | required; default `mixed` |
| `location_ids` | Location | one to many | `stock.location` | inverse field `storage_category_id` |
| `company_id` | Company | many to one | `res.company` |  |
| `weight_uom_name` | Weight unit | single line text |  | computed by rule `_compute_weight_uom_name` (not stored) |

## Selection values

### `allow_new_product` (Allow New Product)

| Value | Label |
|---|---|
| `empty` | If the location is empty |
| `same` | If all products are same |
| `mixed` | Allow mixed products |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_positive_max_weight` | Constraint | `CHECK(max_weight >= 0)` | Max weight should be a positive number. | `stock` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_storage_capacity_ids` | computation | self | `stock` | depends: `capacity_ids` |  |
| `_compute_weight_uom_name` | computation | self | `stock` |  |  |
| `_set_storage_capacity_ids` | internal rule | self | `stock` |  |  |
| `copy_data` | lifecycle override | self, default | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock_storage_category multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_storage_category_form` | form |  | `company_id`, `name`, `allow_new_product`, `max_weight`, `weight_uom_name`, `company_id`, `package_capacity_ids`, `package_type_id`, `quantity`, `company_id`, `product_capacity_ids`, `product_id`, `quantity`, `product_uom_id`, `company_id` | `Locations` |  | `stock` |
| `stock.stock_storage_category_tree` | list |  | `name`, `max_weight`, `allow_new_product`, `company_id` |  |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_storage_category` | Storage Categories | list,form |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.storage.category.json`; views: `../../../schemas/interfaces/views/stock.storage.category.json`.
