# Inventory Routes (`stock.route`)

**Transport name:** `stock.route`  
**Storage name:** `stock_route`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `sale_stock`, `stock_delivery`, `purchase_stock`, `mrp`

Description: Inventory Routes

## Identity and behavior

- Default ordering: `sequence`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Route | single line text |  | required; translatable |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to False, it will allow you to hide the route without removing it. |
| `sequence` | Sequence | integer |  | default  |
| `rule_ids` | Rules | one to many | `stock.rule` | inverse field `route_id` |
| `product_selectable` | Applicable on Product | boolean |  | default `True`; Help: When checked, the route will be selectable in the Inventory tab of the Product form. |
| `product_categ_selectable` | Applicable on Product Category | boolean |  | Help: When checked, the route will be selectable on the Product Category. |
| `warehouse_selectable` | Applicable on Warehouse | boolean |  | Help: When a warehouse is selected for this route, this route should be seen as the default route when products pass through this warehouse. |
| `package_type_selectable` | Applicable on Package Type | boolean |  | Help: When checked, the route will be selectable on package types |
| `supplied_wh_id` | Supplied Warehouse | many to one | `stock.warehouse` | indexed (btree_not_null) |
| `supplier_wh_id` | Supplying Warehouse | many to one | `stock.warehouse` |  |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); indexed; Help: Leave this field empty if this route is shared between all companies |
| `product_ids` | Products | many to many | `product.template` | not copied on duplication; must belong to the same company; association table `stock_route_product` |
| `categ_ids` | Product Categories | many to many | `product.category` | not copied on duplication; association table `stock_route_categ` |
| `warehouse_domain_ids` | Warehouse Domain | one to many | `stock.warehouse` | computed by rule `_compute_warehouses` (not stored) |
| `warehouse_ids` | Warehouses | many to many | `stock.warehouse` | not copied on duplication; restricted by domain `[('id', 'in', warehouse_domain_ids)]`; association table `stock_route_warehouse` |
| `sale_selectable` | Selectable on Sales Order Line | boolean |  |  |
| `shipping_selectable` | Applicable on Shipping Methods | boolean |  |  |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `copy_data` | lifecycle override | self, default | `stock` |  |  |
| `_compute_warehouses` | computation | self | `stock` | depends: `company_id` |  |
| `_onchange_company` | on change | self | `stock` | onchange: `company_id` |  |
| `_onchange_warehouse_selectable` | on change | self | `stock` | onchange: `warehouse_selectable` |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `_check_company_consistency` | validation | self | `stock` | constrains: `company_id` |  |
| `_is_valid_resupply_route_for_product` | internal rule | self, product | `mrp`, `purchase_stock`, `stock` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_company_consistency` | ValidationError | Rule %(rule)s belongs to %(rule_company)s while the route belongs to %(route_company)s. | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `base.group_user` | no | yes | no | no | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock_route multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `sale_stock.stock_location_route_view_form_inherit_sale_stock` | xpath | `stock.stock_location_route_form_view` | `sale_selectable` |  |  | `sale_stock` |
| `stock.stock_location_route_tree` | list |  | `sequence`, `name`, `active`, `company_id` |  |  | `stock` |
| `stock.stock_location_route_form_view` | form |  | `company_id`, `name`, `sequence`, `supplied_wh_id`, `active`, `company_id`, `product_categ_selectable`, `product_selectable`, `package_type_selectable`, `warehouse_selectable`, `warehouse_domain_ids`, `warehouse_ids`, `rule_ids`, `sequence`, `action`, `location_src_id`, `location_dest_id` |  |  | `stock` |
| `stock.stock_location_route_view_search` | search |  | `name` |  | `Archived` | `stock` |
| `stock_delivery.stock_location_route_view_form_inherit_stock_delivery` | xpath | `stock.stock_location_route_form_view` | `shipping_selectable` |  |  | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_routes_form` | Routes | list,form |  |  |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.route.json`; views: `../../../schemas/interfaces/views/stock.route.json`.
