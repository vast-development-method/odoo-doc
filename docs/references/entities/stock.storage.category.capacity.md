# Storage Category Capacity (`stock.storage.category.capacity`)

**Transport name:** `stock.storage.category.capacity`  
**Storage name:** `stock_storage_category_capacity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`

Description: Storage Category Capacity

## Identity and behavior

- Default ordering: `storage_category_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `storage_category_id` | Storage Category | many to one | `stock.storage.category` | required; indexed; on delete of the target: cascade |
| `product_id` | Product | many to one | `product.product` | indexed (btree_not_null); on delete of the target: cascade; restricted by domain `[('product_tmpl_id', '=', context.get('active_id', False))] if context.get('active_model') == 'product.template' else [('id', '=', context.get('default_product_id', False))] if context.get('default_product_id') else [('is_storable', '=', True)]`; must belong to the same company |
| `package_type_id` | Package Type | many to one | `stock.package.type` | indexed (btree_not_null); on delete of the target: cascade; must belong to the same company |
| `quantity` | Quantity | float |  | required |
| `product_uom_id` | Product Unit of measure | many to one |  | related through path `product_id.uom_id` |
| `company_id` | Company | many to one | `res.company` | related through path `storage_category_id.company_id` |

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_positive_quantity` | Constraint | `CHECK(quantity > 0)` | Quantity should be a positive number. | `stock` |
| `_unique_product` | Constraint | `UNIQUE(product_id, storage_category_id)` | Multiple capacity rules for one product. | `stock` |
| `_unique_package_type` | Constraint | `UNIQUE(package_type_id, storage_category_id)` | Multiple capacity rules for one package type. | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_storage_category_capacity_tree` | list |  | `storage_category_id`, `product_id`, `package_type_id`, `quantity`, `product_uom_id`, `company_id`, `package_type_id` |  |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_storage_category_capacity` | Storage Category Capacity | list |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.storage.category.capacity.json`; views: `../../../schemas/interfaces/views/stock.storage.category.capacity.json`.
