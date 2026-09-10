# Putaway Rule (`stock.putaway.rule`)

**Transport name:** `stock.putaway.rule`  
**Storage name:** `stock_putaway_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`

Description: Putaway Rule

## Identity and behavior

- Default ordering: `sequence,product_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Product | many to one | `product.product` | default computed dynamically (_default_product_id); indexed (btree_not_null); on delete of the target: cascade; restricted by domain `[('product_tmpl_id', '=', context.get('active_id', False))] if context.get('active_model') == 'product.template' else [('type', '!=', 'service')]`; must belong to the same company |
| `category_id` | Product Category | many to one | `product.category` | default computed dynamically (_default_category_id); indexed (btree_not_null); on delete of the target: cascade; restricted by domain `[["filter_for_stock_putaway_rule", "=", true]]` |
| `location_in_id` | When product arrives in | many to one | `stock.location` | required; default computed dynamically (_default_location_id); indexed; on delete of the target: cascade; restricted by domain `[('child_ids', '!=', False)]`; must belong to the same company |
| `location_out_id` | Store to sublocation | many to one | `stock.location` | required; on delete of the target: cascade; restricted by domain `[('id', 'child_of', location_in_id)]`; must belong to the same company |
| `sequence` | Priority | integer |  | Help: Give to the more specialized category, a higher priority to have them in top of the list. |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda s: s.env.company.id); indexed |
| `package_type_ids` | Package Type | many to many | `stock.package.type` | must belong to the same company |
| `storage_category_id` | Storage Category | many to one | `stock.storage.category` | computed by rule `_compute_storage_category` and stored; on delete of the target: cascade; must belong to the same company |
| `active` | Active | boolean |  | default `True` |
| `sublocation` | Sublocation | selection |  | default `no` |

## Selection values

### `sublocation` (Sublocation)

| Value | Label |
|---|---|
| `no` | No |
| `last_used` | Last Used |
| `closest_location` | Closest Location |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_category_id` | preparation rule | self | `stock` |  |  |
| `_default_location_id` | preparation rule | self | `stock` |  |  |
| `_default_product_id` | preparation rule | self | `stock` |  |  |
| `_compute_storage_category` | computation | self | `stock` | depends: `sublocation` |  |
| `_onchange_sublocation` | on change | self | `stock` | onchange: `sublocation`, `location_out_id`, `storage_category_id` |  |
| `_onchange_location_in` | on change | self | `stock` | onchange: `location_in_id` |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `_get_last_used_search_domain` | preparation rule | self, product | `stock` |  |  |
| `_get_last_used_location` | preparation rule | self, product | `stock` |  |  |
| `_get_putaway_location` | preparation rule | self, product, quantity, package, packaging, qty_by_location | `stock` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | Changing the company of this record is forbidden at this point, you should rather archive it and create a new one. | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_putaway_list` | list |  | `company_id`, `package_type_ids`, `sequence`, `location_in_id`, `product_id`, `category_id`, `package_type_ids`, `location_out_id`, `sublocation`, `storage_category_id`, `company_id` |  |  | `stock` |
| `stock.view_putaway_search` | search |  | `product_id`, `category_id`, `location_in_id`, `location_out_id` |  | `Rules on Products`, `Rules on Categories`, `Location: When arrives to`, `Location: Store to` | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_putaway_tree` | Putaway Rules | list |  |  |  | `stock` |
| `stock.category_open_putaway` | Putaway Rules |  |  | `{             'search_default_category_id': [active_id],             'fixed_category': True,         }` |  | `stock` |
| `stock.location_open_putaway` | Putaway Rules |  | `['\|', ('location_out_id', '=', active_id), ('location_in_id', '=', active_id)]` | `{'fixed_location': True}` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.putaway.rule.json`; views: `../../../schemas/interfaces/views/stock.putaway.rule.json`.
