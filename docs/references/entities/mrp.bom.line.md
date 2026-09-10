# Bill of Material Line (`mrp.bom.line`)

**Transport name:** `mrp.bom.line`  
**Storage name:** `mrp_bom_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `purchase_mrp`

Description: Bill of Material Line

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `product_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | Component | many to one | `product.product` | required; indexed; restricted by domain `[('type', 'in', ['consu', 'service'])]`; must belong to the same company |
| `product_tmpl_id` | Product Template | many to one | `product.template` | related through path `product_id.product_tmpl_id` and stored; indexed |
| `company_id` | Company | many to one |  | read only; related through path `bom_id.company_id` and stored; indexed |
| `product_qty` | Quantity | float |  | required; default `1.0`; precision `Product Unit` |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; default computed dynamically (_get_default_product_uom_id) |
| `sequence` | Sequence | integer |  | default `1`; Help: Gives the sequence order when displaying. |
| `bom_id` | Parent bill of materials | many to one | `mrp.bom` | required; indexed; on delete of the target: cascade |
| `parent_product_tmpl_id` | Parent Product Template | many to one | `product.template` | related through path `bom_id.product_tmpl_id` |
| `possible_bom_product_template_attribute_value_ids` | Possible Bill of materials Product Template Attribute Value | many to many |  | related through path `bom_id.possible_product_template_attribute_value_ids` |
| `bom_product_template_attribute_value_ids` | Apply on Variants | many to many | `product.template.attribute.value` | on delete of the target: restrict; restricted by domain `[('id', 'in', possible_bom_product_template_attribute_value_ids)]`; Help: BOM Product Variants needed to apply this line. |
| `allowed_operation_ids` | Allowed Operation | one to many | `mrp.routing.workcenter` | related through path `bom_id.operation_ids` |
| `operation_id` | Consumed in Operation | many to one | `mrp.routing.workcenter` | restricted by domain `[('id', 'in', allowed_operation_ids)]`; must belong to the same company; Help: The operation where the components are consumed, or the finished products created. |
| `child_bom_id` | Sub bill of materials | many to one | `mrp.bom` | computed by rule `_compute_child_bom_id` (not stored) |
| `child_line_ids` | bill of materials lines of the referred bom | one to many | `mrp.bom.line` | computed by rule `_compute_child_line_ids` (not stored) |
| `attachments_count` | Attachments Count | integer |  | computed by rule `_compute_attachments_count` (not stored) |
| `tracking` | Tracking | selection |  | related through path `product_id.tracking` |
| `cost_share` | Cost Share (%) | float |  | Help: The percentage of the component repartition cost when purchasing a kit.The total of all components' cost have to be equal to 100. |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_bom_qty_zero` | Constraint | `CHECK (product_qty>=0)` | All product quantities must be greater or equal to 0. Lines with 0 quantities can be used as optional lines.  You should install the mrp_byproduct module if you want to manage extra products on BoMs! | `mrp` |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_product_uom_id` | preparation rule | self | `mrp` |  |  |
| `_compute_child_bom_id` | computation | self | `mrp` | depends: `product_id`, `bom_id` |  |
| `_compute_attachments_count` | computation | self | `mrp` | depends: `product_id` |  |
| `_compute_child_line_ids` | computation | self | `mrp` | depends: `child_bom_id` | If the BOM line refers to a BOM, return the ids of the child BOM lines |
| `onchange_product_id` | on change | self | `mrp` | onchange: `product_id` |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `_skip_bom_line` | internal rule | self, product, never_attribute_values | `mrp` |  | Control if a BoM line should be produced, can be inherited to add custom control. cases:     - no_variant:         1. attribute present on the line             => need to be at least one attribute value matching between the one passed as args and the ones one the line         2. attribute not present on the line             => valid if the line has no attribute value selected for that attribute     - always and dynamic: match_all_variant_values() |
| `action_see_attachments` | user action | self | `mrp` |  |  |
| `action_add_from_catalog` | user action | self | `mrp` |  |  |
| `_get_product_catalog_lines_data` | preparation rule | self, default, **kwargs | `mrp` |  |  |
| `_prepare_bom_done_values` | preparation rule | self, quantity, product, original_quantity, boms_done | `mrp`, `purchase_mrp` |  |  |
| `_prepare_line_done_values` | preparation rule | self, quantity, product, original_quantity, parent_line, boms_done | `mrp`, `purchase_mrp` |  |  |
| `_get_cost_share` | preparation rule | self | `purchase_mrp` |  |  |
| `_get_line_cost_share` | preparation rule | self, product, boms_done | `purchase_mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |
| `stock.group_stock_user` | no | yes | no | no | `mrp` |
| `account.group_account_readonly` | no | yes | no | no | `mrp_account` |
| `account.group_account_invoice` | no | yes | no | no | `mrp_account` |
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `point_of_sale.group_pos_user` | no | yes | no | no | `pos_mrp` |
| `project.group_project_user` | no | yes | no | no | `project_mrp` |
| `purchase.group_purchase_user` | no | yes | no | no | `purchase_mrp` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_mrp` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| MRP BoM Lines Subcontractor | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.bom_line_ids.ids)]` | True | True | True | True |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_bom_line_view_form` | form |  | `product_id`, `parent_product_tmpl_id`, `product_qty`, `product_uom_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids`, `company_id`, `sequence`, `allowed_operation_ids`, `operation_id` |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.bom.line.json`; views: `../../../schemas/interfaces/views/mrp.bom.line.json`.
