# Quotation Template Line (`sale.order.template.line`)

**Transport name:** `sale.order.template.line`  
**Storage name:** `sale_order_template_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_management`  
**Extended by packages:** `sale_project`

Description: Quotation Template Line

## Identity and behavior

- Default ordering: `sale_order_template_id, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sale_order_template_id` | Quotation Template Reference | many to one | `sale.order.template` | required; indexed; on delete of the target: cascade |
| `sequence` | Sequence | integer |  | default `10`; Help: Gives the sequence order when displaying a list of sale quote lines. |
| `company_id` | Company | many to one |  | related through path `sale_order_template_id.company_id` and stored; indexed |
| `product_id` | Product | many to one | `product.product` | restricted by domain `lambda self: self._product_id_domain()`; must belong to the same company |
| `name` | Description | multi line text |  | translatable |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | computed by rule `_compute_product_uom_id` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `product_uom_qty` | Quantity | float |  | required; default `1`; precision `Product Unit` |
| `display_type` | Display Type | selection |  | default  |
| `parent_id` | Parent Section Line | many to one | `sale.order.template.line` | computed by rule `_compute_parent_id` (not stored) |
| `is_optional` | Optional Line | boolean |  | default  |

## Selection values

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_accountable_product_id_required` | Constraint | `CHECK(display_type IS NOT NULL OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL))` | Missing required product and UoM on accountable sale quote line. | `sale_management` |
| `_non_accountable_fields_null` | Constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND product_uom_qty = 0 AND product_uom_id IS NULL))` | Forbidden product, quantity and UoM on non-accountable sale quote line | `sale_management` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_allowed_uom_ids` | computation | self | `sale_management` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids` |  |
| `_compute_product_uom_id` | computation | self | `sale_management` | depends: `product_id` |  |
| `_compute_parent_id` | computation | self | `sale_management` |  |  |
| `create` | lifecycle override | self, vals_list | `sale_management` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `sale_management` |  |  |
| `_product_id_domain` | internal rule | self | `sale_management` | model | Returns the domain of the products that can be added to the template. |
| `_prepare_order_line_values` | preparation rule | self | `sale_management`, `sale_project` |  | Give the values to create the corresponding order line.  :return: `sale.order.line` create values :rtype: dict |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot change the type of a sale quote line. Instead you should delete the current line and create a new line of the proper type. | `sale_management` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale_management` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_management` |

Machine-readable definition: `../../../schemas/data/entities/sale.order.template.line.json`.
