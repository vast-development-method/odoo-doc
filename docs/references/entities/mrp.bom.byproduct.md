# Byproduct (`mrp.bom.byproduct`)

**Transport name:** `mrp.bom.byproduct`  
**Storage name:** `mrp_bom_byproduct`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`

Description: Byproduct

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `product_id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `product_id` | By-product | many to one | `product.product` | required; must belong to the same company |
| `company_id` | Company | many to one |  | read only; related through path `bom_id.company_id` and stored; indexed |
| `product_qty` | Quantity | float |  | required; default `1.0`; precision `Product Unit` |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; computed by rule `_compute_product_uom_id` and stored; precomputed before insertion |
| `bom_id` | bill of materials | many to one | `mrp.bom` | indexed; on delete of the target: cascade |
| `allowed_operation_ids` | Allowed Operation | one to many | `mrp.routing.workcenter` | related through path `bom_id.operation_ids` |
| `operation_id` | Produced in Operation | many to one | `mrp.routing.workcenter` | restricted by domain `[('id', 'in', allowed_operation_ids)]`; must belong to the same company |
| `possible_bom_product_template_attribute_value_ids` | Possible Bill of materials Product Template Attribute Value | many to many |  | related through path `bom_id.possible_product_template_attribute_value_ids` |
| `bom_product_template_attribute_value_ids` | Apply on Variants | many to many | `product.template.attribute.value` | on delete of the target: restrict; restricted by domain `[('id', 'in', possible_bom_product_template_attribute_value_ids)]`; Help: BOM Product Variants needed to apply this line. |
| `sequence` | Sequence | integer |  |  |
| `cost_share` | Cost Share (%) | float |  | precision `[5, 2]`; Help: The percentage of the final production cost for this by-product line (divided between the quantity produced).The total of all by-products' cost share must be less than or equal to 100. |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_product_uom_id` | computation | self | `mrp` | depends: `product_id` | Changes UoM if product_id changes. |
| `_skip_byproduct_line` | internal rule | self, product, never_attribute_values | `mrp` |  | Control if a byproduct line should be produced, can be inherited to add custom control. |
| `action_add_from_catalog` | user action | self | `mrp` |  |  |
| `_get_product_catalog_lines_data` | preparation rule | self, default, **kwargs | `mrp` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `mrp.group_mrp_manager` | yes | yes | yes | yes | `mrp` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_bom_byproduct_form_view` | form |  | `allowed_operation_ids`, `company_id`, `product_id`, `product_qty`, `product_uom_id`, `operation_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids` |  |  | `mrp` |

Machine-readable definition: `../../../schemas/data/entities/mrp.bom.byproduct.json`; views: `../../../schemas/interfaces/views/mrp.bom.byproduct.json`.
