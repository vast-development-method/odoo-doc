# Bill of Material (`mrp.bom`)

**Transport name:** `mrp.bom`  
**Storage name:** `mrp_bom`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mrp`  
**Extended by packages:** `mrp_subcontracting`, `purchase_mrp`, `project_mrp`, `sale_mrp`

Description: Bill of Material

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `product.catalog.mixin`
- Default ordering: `sequence, id`
- Display name field: `product_tmpl_id`
- Display name search fields: `["product_tmpl_id", "code"]`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `code` | Reference | single line text |  |  |
| `active` | Active | boolean |  | default `True` |
| `type` | bill of materials Type | selection |  | required; default `normal`; on delete of the target: {"expression": "{'subcontract': lambda recs: recs.write({'type': 'normal', 'active': False})}"}; extended by packages `mrp_subcontracting` |
| `product_tmpl_id` | Product | many to one | `product.template` | required; indexed; restricted by domain `[('type', '=', 'consu')]`; must belong to the same company |
| `product_id` | Product Variant | many to one | `product.product` | indexed; restricted by domain `['&', ('product_tmpl_id', '=', product_tmpl_id), ('type', '=', 'consu')]`; must belong to the same company; Help: If a product variant is defined the BOM is available only for this product. |
| `bom_line_ids` | bill of materials Lines | one to many | `mrp.bom.line` | inverse field `bom_id` |
| `byproduct_ids` | By-products | one to many | `mrp.bom.byproduct` | inverse field `bom_id` |
| `product_qty` | Quantity | float |  | required; default `1.0`; precision `Product Unit`; Help: This should be the smallest quantity that this product can be produced in. If the BOM contains operations, make sure the work center capacity is accurate. |
| `product_uom_id` | Unit | many to one | `uom.uom` | required; default computed dynamically (_get_default_product_uom_id); Help: Unit of Measure (Unit of Measure) is the unit of measurement for the inventory control |
| `sequence` | Sequence | integer |  |  |
| `operation_ids` | Operations | one to many | `mrp.routing.workcenter` | inverse field `bom_id` |
| `operation_count` | Operations Count | integer |  | computed by rule `_compute_operation_count` (not stored) |
| `show_copy_operations_button` | Show Copy Operations Button | boolean |  | computed by rule `_compute_show_copy_operations_button` (not stored); Help: Technical field used to control the visibility of the 'Copy Existing Operations' button. |
| `ready_to_produce` | Manufacturing Readiness | selection |  | required; default `all_available` |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | restricted by domain `[('code', '=', 'mrp_operation')]`; must belong to the same company; Help: When a procurement has a ‘produce’ route with a operation type set, it will try to create a Manufacturing Order for that product using a BoM of the same operation type.If not,the operation type is not taken into account in the BoM search. That allows to define stock rules which trigger different manufacturing orders with different BoMs. |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); indexed |
| `consumption` | Flexible Consumption | selection |  | required; default `warning`; Help: Defines if you can consume more or less components than the quantity defined on the BoM:   * Allowed: allowed for all manufacturing users.   * Allowed with warning: allowed for all manufacturing users with summary of consumption differences when closing the manufacturing order.   Note that in the case of component Highlight Consumption, where consumption is registered manually exclusively, consumption warnings will still be issued when appropriate also.   * Blocked: only a manager can close a manufacturing order when the BoM consumption is not respected. |
| `possible_product_template_attribute_value_ids` | Possible Product Template Attribute Value | many to many | `product.template.attribute.value` | computed by rule `_compute_possible_product_template_attribute_value_ids` (not stored) |
| `allow_operation_dependencies` | Operation Dependencies | boolean |  | Help: Create operation level dependencies that will influence both planning and the status of work orders upon MO confirmation. If this feature is ticked, and nothing is specified, Odoo will assume that all operations can be started simultaneously. |
| `produce_delay` | Manufacturing Lead Time | integer |  | default ; Help: Average lead time in days to manufacture this product. In the case of multi-level BOM, the manufacturing lead times of the components will be added. In case the product is subcontracted, this can be used to determine the date at which components should be sent to the subcontractor. |
| `days_to_prepare_mo` | Days to prepare Manufacturing Order | integer |  | default ; Help: Create and confirm Manufacturing Orders this many days in advance, to have enough time to replenish components or manufacture semi-finished products. |
| `show_set_bom_button` | Show Set Bill of materials Button | boolean |  | computed by rule `_compute_show_set_bom_button` (not stored) |
| `batch_size` | Batch Size | float |  | default `1.0`; precision `Product Unit`; Help: All automatically generated manufacturing orders for this product will be of this size. |
| `enable_batch_size` | Enable Batch Size | boolean |  | default  |
| `subcontractor_ids` | Subcontractors | many to many | `res.partner` | must belong to the same company; association table `mrp_bom_subcontractor` |
| `project_id` | Project | many to one | `project.project` | restricted by domain `[["is_template", "=", false]]` |

## Selection values

### `type` (bill of materials Type)

| Value | Label |
|---|---|
| `normal` | Manufacture this product |
| `phantom` | Kit |
| `subcontract` | Subcontracting |

### `ready_to_produce` (Manufacturing Readiness)

| Value | Label |
|---|---|
| `all_available` | When all components are available |
| `asap` | When components for 1st operation are available |

### `consumption` (Flexible Consumption)

| Value | Label |
|---|---|
| `flexible` | Allowed |
| `warning` | Allowed with warning |
| `strict` | Blocked |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_qty_positive` | Constraint | `check (product_qty > 0)` | The quantity to produce must be positive! | `mrp` |

## Operations (43)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_product_uom_id` | preparation rule | self | `mrp` |  |  |
| `_compute_possible_product_template_attribute_value_ids` | computation | self | `mrp` | depends: `product_tmpl_id.attribute_line_ids.value_ids`, `product_tmpl_id.attribute_line_ids.attribute_id.create_variant`, `product_tmpl_id.attribute_line_ids.product_template_value_ids.ptav_active` |  |
| `_onchange_product_id` | on change | self | `mrp` | onchange: `product_id` |  |
| `_check_bom_cycle` | validation | self | `mrp` | constrains: `active`, `product_id`, `product_tmpl_id`, `bom_line_ids` |  |
| `_check_bom_lines` | validation | self | `mrp`, `purchase_mrp` | constrains: `product_id`, `product_tmpl_id`, `bom_line_ids`, `byproduct_ids`, `operation_ids` |  |
| `onchange_bom_structure` | on change | self | `mrp` | onchange: `bom_line_ids`, `product_qty`, `product_id`, `product_tmpl_id` |  |
| `onchange_product_tmpl_id` | on change | self | `mrp` | onchange: `product_tmpl_id` |  |
| `create` | lifecycle override | self, vals_list | `mrp` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mrp`, `sale_mrp` |  |  |
| `copy` | lifecycle override | self, default | `mrp` |  |  |
| `name_create` | lifecycle override | self, name | `mrp` | model |  |
| `action_archive` | lifecycle override | self | `mrp` |  |  |
| `action_unarchive` | lifecycle override | self | `mrp` |  |  |
| `_compute_display_name` | computation | self | `mrp` | depends: `code` |  |
| `_compute_operation_count` | computation | self | `mrp` | depends: `operation_ids` |  |
| `_compute_show_copy_operations_button` | computation | self | `mrp` |  |  |
| `action_compute_bom_days` | user action | self | `mrp` |  |  |
| `check_kit_has_not_orderpoint` | validation | self | `mrp` | constrains: `product_tmpl_id`, `product_id`, `type` |  |
| `_check_valid_batch_size` | validation | self | `mrp` | constrains: `enable_batch_size`, `batch_size` |  |
| `_unlink_except_running_mo` | internal rule | self | `mrp` | ondelete |  |
| `_bom_find_domain` | internal rule | self, products, picking_type, company_id, bom_type | `mrp` | model |  |
| `_bom_find` | internal rule | self, products, picking_type, company_id, bom_type | `mrp` | model | Find the first BoM for each products  :param products: `product.product` recordset :return: One bom (or empty recordset `mrp.bom` if none find) by product (`product.product` record) :rtype: defaultdict(`lambda: self.env['mrp.bom']`) |
| `explode` | operation | self, product, quantity, picking_type, never_attribute_values | `mrp` |  | Explodes the BoM and creates two lists with all the information you need: bom_done and line_done Quantity describes the number of times you need the BoM: so the quantity divided by the number created by the BoM and converted into its UoM |
| `_round_last_line_done` | internal rule | self, lines_done | `mrp`, `purchase_mrp` | model |  |
| `get_import_templates` | operation | self | `mrp` | model |  |
| `_set_outdated_bom_in_productions` | internal rule | self | `mrp` |  |  |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `mrp` |  |  |
| `_default_order_line_values` | preparation rule | self, child_field | `mrp` |  |  |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `mrp` |  |  |
| `_get_product_price_and_data` | preparation rule | self, product | `mrp` |  |  |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, child_field, **kwargs | `mrp` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, child_field, **kwargs | `mrp` |  |  |
| `_get_mail_thread_data_attachments` | preparation rule | self | `mrp` |  |  |
| `_get_extra_attachments` | preparation rule | self | `mrp` |  |  |
| `_skip_for_no_variant` | internal rule | self, product, bom_attribule_values, never_attribute_values | `mrp` | model | Controls if a Component/Operation/Byproduct line should be skipped based on the 'no_variant' attributes Cases:     - no_variant:         1. attribute present on the line             => Every attribute on the line must have at least one of its values match with a value passed as args.         2. attribute not present on the line             => valid if the line has no attribute value selected for that attribute     - always and dynamic: match_all_variant_values() |
| `_compute_show_set_bom_button` | computation | self | `mrp` |  |  |
| `action_set_bom_on_orderpoint` | user action | self | `mrp` |  |  |
| `action_open_operation_form` | user action | self | `mrp` |  |  |
| `action_copy_existing_operations` | user action | self | `mrp` |  |  |
| `_bom_subcontract_find` | internal rule | self, product, picking_type, company_id, bom_type, subcontractor | `mrp_subcontracting` |  |  |
| `_check_subcontracting_no_operation` | validation | self | `mrp_subcontracting` | constrains: `operation_ids`, `byproduct_ids`, `type` |  |
| `unlink` | lifecycle override | self | `sale_mrp` |  |  |
| `_ensure_bom_is_free` | internal rule | self | `sale_mrp` |  |  |

## Validation and error messages (14)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_bom_cycle` | ValidationError | The current configuration is incorrect because it would create a cycle between these products: %s. | `mrp` |
| `_check_bom_lines` | ValidationError | You cannot use the 'Apply on Variant' functionality and simultaneously create a BoM for a specific variant. | `mrp` |
| `_check_bom_lines` | ValidationError | The attribute value %(attribute)s set on product %(product)s does not match the BoM product %(bom_product)s. | `mrp` |
| `_check_bom_lines` | ValidationError | By-product %s should not be the same as BoM product. | `mrp` |
| `_check_bom_lines` | ValidationError | By-products cost shares must be positive. | `mrp` |
| `_check_bom_lines` | ValidationError | The total cost share for a BoM's by-products cannot exceed 100. | `mrp` |
| `name_create` | UserError | You cannot create a new Bill of Material from here. | `mrp` |
| `check_kit_has_not_orderpoint` | ValidationError | You can not create a kit-type bill of materials for products that have at least one reordering rule. | `mrp` |
| `_check_valid_batch_size` | ValidationError | The batch size must be positive! | `mrp` |
| `_unlink_except_running_mo` | UserError | You can not delete a Bill of Material with running manufacturing orders. Please close or cancel it first. | `mrp` |
| `_check_subcontracting_no_operation` | ValidationError | You can not set a Bill of Material with operations or by-product line as subcontracting. | `mrp_subcontracting` |
| `_check_bom_lines` | UserError | Components cost share have to be positive or equals to zero. | `purchase_mrp` |
| `_check_bom_lines` | UserError | The total cost share for a BoM's component have to be 100 | `purchase_mrp` |
| `_ensure_bom_is_free` | UserError | As long as there are some sale order lines that must be delivered/invoiced and are related to these bills of materials, you can not remove them. The error concerns these products: %s | `sale_mrp` |

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
| MRP BoMs Subcontractor | `[(4, ref('base.group_portal'))]` | `[('id', 'in', user.partner_id.commercial_partner_id.bom_ids.ids)]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_bom_form_view` | form |  | `active`, `company_id`, `product_tmpl_id`, `allow_operation_dependencies`, `product_id`, `product_id`, `product_qty`, `product_uom_id`, `code`, `type`, `company_id`, `bom_line_ids`, `company_id`, `sequence`, `product_id`, `product_tmpl_id`, `attachments_count`, `product_qty`, `parent_product_tmpl_id`, `product_uom_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids`, `allowed_operation_ids`, `operation_id`, `operation_ids`, `byproduct_ids`, `company_id`, `sequence`, `product_id`, `product_qty`, `product_uom_id`, `cost_share`, `allowed_operation_ids`, `operation_id`, `possible_bom_product_template_attribute_value_ids`, `bom_product_template_attribute_value_ids`, `ready_to_produce`, `consumption`, `allow_operation_dependencies`, `picking_type_id`, `produce_delay`, `days_to_prepare_mo`, `enable_batch_size`, `batch_size`, `product_uom_id` | `%(action_mrp_routing_time)d`, `%(action_report_mrp_bom)d`, `Catalog`, `action_see_attachments`, `action_open_operation_form`, `action_copy_existing_operations`, `Catalog`, `Compute` |  | `mrp` |
| `mrp.mrp_bom_tree_view` | list |  | `active`, `sequence`, `company_id`, `product_tmpl_id`, `code`, `type`, `product_id`, `product_id`, `company_id`, `product_qty`, `product_uom_id` |  |  | `mrp` |
| `mrp.mrp_bom_kanban_view` | kanban |  | `product_tmpl_id`, `product_qty`, `product_uom_id`, `code` |  |  | `mrp` |
| `mrp.view_mrp_bom_filter` | search |  | `code`, `product_tmpl_id`, `bom_line_ids` |  | `Manufacturing`, `Kit`, `Archived`, `Product`, `BoM Type`, `Unit` | `mrp` |
| `mrp.mrp_bom_replenishment_tree_view` | field | `mrp.mrp_bom_tree_view` | `product_uom_id`, `show_set_bom_button` | `Set as Bill of Materials` |  | `mrp` |
| `mrp_subcontracting.mrp_bom_form_view` | xpath | `mrp.mrp_bom_form_view` | `subcontractor_ids` |  |  | `mrp_subcontracting` |
| `project_mrp.mrp_bom_form_view_inherited_project_mrp` | xpath | `mrp.mrp_bom_form_view` | `project_id` |  |  | `project_mrp` |
| `purchase_mrp.mrp_bom_form_view` | xpath | `mrp.mrp_bom_form_view` | `cost_share` |  |  | `purchase_mrp` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp.mrp_bom_form_action` | Bills of Materials | list,kanban,form | `[]` | `{'default_company_id': allowed_company_ids[0]}` |  | `mrp` |
| `mrp.template_open_bom` | Bill of Materials |  | `['\|', ('product_tmpl_id', '=', active_id), ('byproduct_ids.product_id.product_tmpl_id', '=', active_id)]` | `{'default_product_tmpl_id': active_id}` |  | `mrp` |
| `mrp.product_open_bom` | Bill of Materials |  | `[]` | `{'default_product_id': active_id}` |  | `mrp` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `mrp.action_report_bom_structure` | BoM Overview | qweb-pdf | `mrp.report_bom_structure` | `'Bom Overview - %s' % object.display_name` |  |

Machine-readable definition: `../../../schemas/data/entities/mrp.bom.json`; views: `../../../schemas/interfaces/views/mrp.bom.json`.
