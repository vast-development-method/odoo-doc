# Purchase Order Line (`purchase.order.line`)

**Transport name:** `purchase.order.line`  
**Storage name:** `purchase_order_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `purchase`  
**Extended by packages:** `purchase_stock`, `stock_landed_costs`, `sale_purchase`, `sale_purchase_stock`, `stock_dropshipping`, `purchase_mrp`, `project_purchase`, `purchase_product_matrix`, `purchase_requisition`, `purchase_requisition_stock`

Description: Purchase Order Line

## Identity and behavior

- Mixins (classical inheritance): `analytic.mixin`
- Default ordering: `order_id, sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (58)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Description | multi line text |  | required; computed by rule `_compute_price_unit_and_date_planned_and_name` and stored |
| `translated_product_name` | Translated Product Name | multi line text |  | computed by rule `_compute_translated_product_name` (not stored) |
| `sequence` | Sequence | integer |  | default `10` |
| `product_qty` | Quantity | float |  | required; precision `Product Unit` |
| `product_uom_qty` | Total Quantity | float |  | computed by rule `_compute_product_uom_qty` and stored |
| `date_planned` | Expected Arrival | date and time |  | computed by rule `_compute_price_unit_and_date_planned_and_name` and stored; indexed; Help: Delivery date expected from vendor. This date respectively defaults to vendor pricelist lead time then today's date. |
| `discount` | Discount (%) | float |  | computed by rule `_compute_price_unit_and_date_planned_and_name` and stored; precision `Discount` |
| `tax_ids` | Taxes | many to many | `account.tax` | association table `account_tax_purchase_order_line_rel` |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom_id` | Unit | many to one | `uom.uom` | on delete of the target: restrict; restricted by domain `[('id', 'in', allowed_uom_ids)]` |
| `product_id` | Product | many to one | `product.product` | indexed (btree_not_null); on delete of the target: restrict; restricted by domain `[["purchase_ok", "=", true]]` |
| `product_type` | Product Type | selection |  | read only; related through path `product_id.type` |
| `price_unit` | Unit Price | float |  | required; computed by rule `_compute_price_unit_and_date_planned_and_name` and stored; aggregated with avg |
| `price_unit_product_uom` | Unit Price Product unit of measure | float |  | computed by rule `_compute_price_unit_product_uom` (not stored); Help: The Price of one unit of the product's Unit of Measure |
| `price_unit_discounted` | Unit Price (Discounted) | float |  | computed by rule `_compute_price_unit_discounted` (not stored) |
| `price_subtotal` | Subtotal | monetary |  | computed by rule `_compute_amount` and stored |
| `price_total` | Total | monetary |  | computed by rule `_compute_amount` and stored |
| `price_tax` | Tax | float |  | computed by rule `_compute_amount` and stored |
| `order_id` | Order Reference | many to one | `purchase.order` | required; indexed; on delete of the target: cascade |
| `company_id` | Company | many to one | `res.company` | read only; related through path `order_id.company_id` and stored |
| `state` | State | selection |  | related through path `order_id.state` |
| `invoice_lines` | Bill Lines | one to many | `account.move.line` | read only; not copied on duplication; inverse field `purchase_line_id` |
| `qty_invoiced` | Billed Qty | float |  | computed by rule `_compute_qty_invoiced` and stored; precision `Product Unit` |
| `qty_received_method` | Received Qty Method | selection |  | computed by rule `_compute_qty_received_method` and stored; on delete of the target: {"expression": "{'stock_moves': _ondelete_stock_moves}"}; Help: According to product configuration, the received quantity can be automatically computed by mechanism:   - Manual: the quantity is set manually on the line   - Stock Moves: the quantity comes from confirmed pickings; extended by packages `purchase_stock` |
| `qty_received` | Received Qty | float |  | computed by rule `_compute_qty_received` and stored; writable through an inverse rule; precision `Product Unit` |
| `qty_received_manual` | Manual Received Qty | float |  | not copied on duplication; precision `Product Unit` |
| `qty_to_invoice` | To Invoice Quantity | float |  | read only; computed by rule `_compute_qty_invoiced` and stored; precision `Product Unit` |
| `qty_received_at_date` | Received | float |  | computed by rule `_compute_qty_received_at_date` (not stored); precision `Product Unit` |
| `qty_invoiced_at_date` | Billed | float |  | computed by rule `_compute_qty_invoiced_at_date` (not stored); precision `Product Unit` |
| `amount_to_invoice_at_date` | Amount | float |  | computed by rule `_compute_amount_to_invoice_at_date` (not stored) |
| `partner_id` | Partner | many to one | `res.partner` | read only; related through path `order_id.partner_id` and stored; indexed (btree_not_null) |
| `currency_id` | Currency | many to one |  | related through path `order_id.currency_id` |
| `date_order` | Order Date | date and time |  | read only; related through path `order_id.date_order` |
| `date_approve` | Confirmation Date | date and time |  | read only; related through path `order_id.date_approve` |
| `tax_calculation_rounding_method` | Tax calculation rounding method | selection |  | read only; related through path `company_id.tax_calculation_rounding_method` |
| `display_type` | Display Type | selection |  | default ; Help: Technical field for UX purpose. |
| `is_downpayment` | Is Downpayment | boolean |  |  |
| `selected_seller_id` | Selected Seller | many to one | `product.supplierinfo` | computed by rule `_compute_selected_seller_id` (not stored); Help: Technical field to get the vendor pricelist used to generate this line |
| `product_template_attribute_value_ids` | Product Template Attribute Value | many to many |  | read only; related through path `product_id.product_template_attribute_value_ids`; extended by packages `purchase_product_matrix` |
| `product_no_variant_attribute_value_ids` | Product attribute values that do not create variants | many to many | `product.template.attribute.value` | on delete of the target: restrict |
| `purchase_line_warn_msg` | Purchase Line Warn Msg | multi line text |  | computed by rule `_compute_purchase_line_warn_msg` (not stored) |
| `parent_id` | Parent Section Line | many to one | `purchase.order.line` | computed by rule `_compute_parent_id` (not stored) |
| `technical_price_unit` | Technical Price Unit | float |  | Help: Technical field for price computation |
| `move_ids` | Reservation | one to many | `stock.move` | read only; not copied on duplication; inverse field `purchase_line_id` |
| `orderpoint_id` | Orderpoint | many to one | `stock.warehouse.orderpoint` | indexed (btree_not_null); not copied on duplication; on delete of the target: set null |
| `move_dest_ids` | Downstream moves alt | many to many | `stock.move` | association table `stock_move_created_purchase_line_rel` |
| `product_description_variants` | Custom Description | single line text |  |  |
| `propagate_cancel` | Propagate cancellation | boolean |  | default `True` |
| `forecasted_issue` | Forecasted Issue | boolean |  | computed by rule `_compute_forecasted_issue` (not stored) |
| `is_storable` | Is Storable | boolean |  | related through path `product_id.is_storable` |
| `location_final_id` | Location from procurement | many to one | `stock.location` |  |
| `sale_order_id` | Sale Order | many to one |  | related through path `sale_line_id.order_id` |
| `sale_line_id` | Origin Sale Item | many to one | `sale.order.line` | indexed (btree_not_null); not copied on duplication |
| `product_template_id` | Product Template | many to one | `product.template` | related through path `product_id.product_tmpl_id`; restricted by domain `[["purchase_ok", "=", true]]` |
| `is_configurable_product` | Is the product configurable? | boolean |  | related through path `product_template_id.has_configurable_attributes` |
| `price_total_cc` | Company Subtotal | monetary |  | computed by rule `_compute_price_total_cc` and stored; currency taken from `company_currency_id` |
| `company_currency_id` | Company Currency | many to one |  | related through path `company_id.currency_id` |
| `on_time_rate_perc` | OTD | float |  | related through path `order_id.on_time_rate_perc` |

## Selection values

### `qty_received_method` (Received Qty Method)

| Value | Label |
|---|---|
| `manual` | Manual |
| `stock_moves` | Stock Moves |

### `display_type` (Display Type)

| Value | Label |
|---|---|
| `line_section` | Section |
| `line_subsection` | Subsection |
| `line_note` | Note |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_accountable_required_fields` | Constraint | `CHECK(display_type IS NOT NULL OR is_downpayment OR (product_id IS NOT NULL AND product_uom_id IS NOT NULL AND date_planned IS NOT NULL))` | Missing required fields on accountable purchase order line. | `purchase` |
| `_non_accountable_null_fields` | Constraint | `CHECK(display_type IS NULL OR (product_id IS NULL AND price_unit = 0 AND product_uom_qty = 0 AND product_uom_id IS NULL AND date_planned is NULL))` | Forbidden values on non-accountable purchase order line | `purchase` |

## Operations (73)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_amount` | computation | self | `purchase` | depends: `product_qty`, `price_unit`, `tax_ids`, `discount` |  |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self | `purchase` |  | Convert the current record to a dictionary in order to use the generic taxes computation method defined on account.tax.  :return: A python dictionary. |
| `_compute_tax_id` | computation | self | `purchase` |  |  |
| `_compute_price_unit_discounted` | computation | self | `purchase` | depends: `discount`, `price_unit` |  |
| `_compute_price_unit_product_uom` | computation | self | `purchase` | depends: `product_uom_id`, `price_unit` |  |
| `_compute_qty_invoiced` | computation | self | `purchase` | depends: `invoice_lines.move_id.state`, `invoice_lines.quantity`, `qty_received`, `product_uom_qty`, `order_id.state` |  |
| `_compute_qty_invoiced_at_date` | computation | self | `purchase` | depends: `qty_invoiced`; depends_context: `accrual_entry_date` |  |
| `_prepare_qty_invoiced` | preparation rule | self | `purchase` |  |  |
| `_get_invoice_lines` | preparation rule | self | `purchase` |  |  |
| `_compute_purchase_line_warn_msg` | computation | self | `purchase` | depends: `product_id.purchase_line_warn_msg` |  |
| `_compute_qty_received_method` | computation | self | `purchase_stock`, `purchase` | depends: `product_id`, `product_id.type` |  |
| `_compute_qty_received` | computation | self | `purchase_stock`, `purchase` | depends: `qty_received_method`, `qty_received_manual`; depends: `move_ids.state`, `move_ids.product_uom`, `move_ids.quantity` |  |
| `_compute_qty_received_at_date` | computation | self | `purchase` | depends: `qty_received`; depends_context: `accrual_entry_date` |  |
| `_prepare_qty_received` | preparation rule | self | `purchase_mrp`, `purchase_stock`, `purchase` |  |  |
| `_inverse_qty_received` | on change | self | `purchase` | onchange: `qty_received` | When writing on qty_received, if the value should be modify manually (`qty_received_method` = 'manual' only), then we put the value in `qty_received_manual`. Otherwise, `qty_received_manual` should be False since the received qty is automatically compute by other mecanisms. |
| `_compute_selected_seller_id` | computation | self | `purchase` | depends: `product_id`, `product_id.seller_ids`, `partner_id`, `product_qty`, `order_id.date_order`, `product_uom_id` |  |
| `_compute_amount_to_invoice_at_date` | computation | self | `purchase` | depends: `price_unit`, `qty_invoiced_at_date`, `qty_received_at_date`, `product_qty`; depends_context: `accrual_entry_date` |  |
| `_get_qty_to_invoice_at_date` | preparation rule | self | `purchase` |  | Return the quantity to invoice at the accrual date, respecting the product's purchase method. |
| `create` | lifecycle override | self, vals_list | `project_purchase`, `purchase_stock`, `purchase` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `purchase_stock`, `purchase` |  |  |
| `_unlink_except_purchase` | internal rule | self | `purchase` | ondelete |  |
| `_get_date_planned` | preparation rule | self, seller, po | `purchase` | model | Return the datetime value to use as Schedule Date (`date_planned`) for PO Lines that correspond to the given product.seller_ids, when ordered at `date_order_str`.  :param Model seller: used to fetch the delivery delay (if no seller                      is provided, the delay is 0) :param Model po: purchase.order, necessary only if the PO line is                  not yet attached to a PO. :rtype: datetime :return: desired Schedule Date for the PO line |
| `_compute_analytic_distribution` | computation | self | `project_purchase`, `purchase` | depends: `product_id`, `order_id.partner_id`; depends: `product_id`, `order_id.partner_id`, `order_id.project_id` |  |
| `onchange_product_id` | on change | self | `purchase` | onchange: `product_id` |  |
| `_product_id_change` | internal rule | self | `purchase` |  |  |
| `_compute_allowed_uom_ids` | computation | self | `purchase` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` |  |
| `_compute_price_unit_and_date_planned_and_name` | computation | self | `purchase_requisition`, `purchase` | depends: `product_qty`, `product_uom_id`, `company_id`, `order_id.partner_id` |  |
| `_reset_price_unit` | internal rule | self, price_unit | `purchase` |  |  |
| `_compute_translated_product_name` | computation | self | `purchase` | depends: `product_id` |  |
| `_compute_product_uom_qty` | computation | self | `purchase` | depends: `product_uom_id`, `product_qty`, `product_id.uom_id` |  |
| `_get_gross_price_unit` | preparation rule | self | `purchase` |  |  |
| `_compute_parent_id` | computation | self | `purchase` |  |  |
| `action_add_from_catalog` | user action | self | `purchase` |  |  |
| `_suggest_quantity` | internal rule | self | `purchase` |  | Suggest a minimal quantity based on the seller |
| `_get_product_catalog_lines_data` | preparation rule | self, **kwargs | `purchase` |  | Return information about purchase order lines in `self`.  If `self` is empty, this method returns only the default value(s) needed for the product catalog. In this case, the quantity that equals 0.  Otherwise, it returns a quantity and a price based on the product of the POL(s) and whether the product is read-only or not.  A product is considered read-only if the order is considered read-only (see `PurchaseOrder._is_readonly` for more details) or if `self` contains multiple records.  Note: This method cannot be called with multiple records that have different products linked.  :raise system.ex |
| `_get_product_purchase_description` | preparation rule | self, product_lang | `purchase` |  |  |
| `_prepare_account_move_line` | preparation rule | self, move | `purchase_stock`, `purchase`, `stock_landed_costs` |  |  |
| `_prepare_add_missing_fields` | preparation rule | self, values | `purchase` | model | Deduce missing required fields from the onchange |
| `_prepare_purchase_order_line` | preparation rule | self, product_id, product_qty, product_uom, company_id, partner_id, po | `purchase` | model |  |
| `_convert_to_middle_of_day` | internal rule | self, date | `purchase` |  | Return a datetime which is the noon of the input date(time) according to order user's time zone, convert to UTC time. |
| `_date_in_the_past` | internal rule | self | `purchase` | model |  |
| `_update_date_planned` | internal rule | self, updated_date | `purchase_stock`, `purchase` |  |  |
| `_track_qty_received` | messaging hook | self, new_qty | `purchase` |  |  |
| `_validate_analytic_distribution` | internal rule | self | `purchase` |  |  |
| `action_open_order` | user action | self | `purchase` |  |  |
| `_merge_po_line` | internal rule | self, rfq_line | `purchase_stock`, `purchase` |  |  |
| `_get_select_sellers_params` | preparation rule | self | `purchase` |  |  |
| `get_parent_section_line` | operation | self | `purchase` |  |  |
| `_ondelete_stock_moves` | internal rule | self | `purchase_stock` |  |  |
| `_get_po_line_moves` | preparation rule | self | `purchase_stock` |  |  |
| `_compute_forecasted_issue` | computation | self | `purchase_stock` | depends: `product_uom_qty`, `date_planned` |  |
| `action_product_forecast_report` | user action | self | `purchase_stock` |  |  |
| `unlink` | lifecycle override | self | `purchase_stock` |  |  |
| `_update_move_date_deadline` | internal rule | self, new_date | `purchase_stock` |  | Updates corresponding move picking line deadline dates that are not yet completed. |
| `_create_or_update_picking` | internal rule | self | `purchase_stock` |  |  |
| `_get_move_dests_initial_demand` | preparation rule | self, move_dests | `purchase_mrp`, `purchase_stock` |  |  |
| `_prepare_stock_moves` | preparation rule | self, picking | `purchase_mrp`, `purchase_stock`, `sale_purchase_stock` |  | Prepare the stock moves data for one order line. This function returns a list of dictionary ready to be used in stock.move's create() |
| `_get_stock_move_price_unit` | preparation rule | self | `purchase_stock` |  |  |
| `_get_qty_procurement` | preparation rule | self | `purchase_mrp`, `purchase_stock` |  |  |
| `_check_orderpoint_picking_type` | validation | self | `purchase_stock` |  |  |
| `_prepare_stock_move_vals` | preparation rule | self, picking, price_unit, product_uom_qty, product_uom | `purchase_stock` |  |  |
| `_prepare_purchase_order_line_from_procurement` | preparation rule | self, product_id, product_qty, product_uom, location_dest_id, name, origin, company_id, values, po | `purchase_stock`, `sale_purchase_stock` | model |  |
| `_create_stock_moves` | internal rule | self, picking | `purchase_stock` |  |  |
| `_find_candidate` | internal rule | self, product_id, product_qty, product_uom, location_id, name, origin, company_id, values | `purchase_stock`, `sale_purchase_stock` |  | Return the record in self where the procument with values passed as args can be merged. If it returns an empty record then a new line will be created. |
| `_get_outgoing_incoming_moves` | preparation rule | self | `purchase_stock` |  |  |
| `_update_qty_received_method` | internal rule | self | `purchase_stock` | model | Update qty_received_method for old PO before install this module. |
| `_get_sale_order_line_product` | preparation rule | self | `purchase_mrp`, `sale_purchase_stock` |  |  |
| `_is_dropshipped` | internal rule | self | `stock_dropshipping` |  |  |
| `_compute_kit_quantities_from_moves` | computation | self, moves, kit_bom | `purchase_mrp` |  |  |
| `_get_upstream_documents_and_responsibles` | preparation rule | self, visited | `purchase_mrp` |  |  |
| `_compute_price_total_cc` | computation | self | `purchase_requisition` | depends: `price_subtotal`, `order_id.currency_rate` |  |
| `action_clear_quantities` | user action | self | `purchase_requisition` |  |  |
| `action_choose` | user action | self | `purchase_requisition` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot change the type of a purchase order line. Instead you should delete the current line and create a new line of the proper type. | `purchase` |
| `_unlink_except_purchase` | UserError | Cannot delete a purchase order line which is in state “%s”. | `purchase` |
| `_check_orderpoint_picking_type` | UserError | The warehouse of operation type (%(operation_type)s) is inconsistent with location (%(location)s) of reordering rule (%(reordering_rule)s) for product %(product)s. Change the operation type or cancel the request for quotation. | `purchase_stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_purchase_user` | yes | yes | yes | yes | `purchase` |
| `group_purchase_manager` | yes | yes | yes | yes | `purchase` |
| `account.group_account_readonly` | no | yes | no | no | `purchase` |
| `account.group_account_invoice` | no | yes | yes | no | `purchase` |
| `base.group_portal` | no | yes | no | no | `purchase` |
| `stock.group_stock_user` | no | yes | no | no | `purchase_stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Purchase Order Line multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |
| Portal Purchase Order Lines | `[(4, ref('base.group_portal'))]` | `[('order_id.partner_id','child_of',[user.commercial_partner_id.id])]` | True | True | True | True |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `purchase.purchase_order_line_tree` | list |  | `order_id`, `name`, `partner_id`, `product_id`, `price_unit`, `product_qty`, `product_uom_id`, `price_subtotal`, `currency_id`, `date_planned` |  |  | `purchase` |
| `purchase.purchase_order_line_form2` | form |  | `order_id`, `date_order`, `partner_id`, `product_id`, `product_qty`, `product_uom_id`, `price_unit`, `tax_ids`, `date_planned`, `company_id`, `analytic_distribution`, `name`, `invoice_lines` |  |  | `purchase` |
| `purchase.purchase_order_line_search` | search |  | `order_id`, `product_id`, `partner_id` |  | `Hide cancelled lines`, `Status`, `filter_date_order`, `Order Date: Last 365 Days`, `Vendor`, `Product`, `Order Reference`, `Order Date` | `purchase` |
| `purchase.purchase_history_tree` | list |  | `currency_id`, `order_id`, `date_approve`, `partner_id`, `company_id`, `product_uom_qty`, `price_unit_product_uom`, `price_subtotal`, `state` |  |  | `purchase` |
| `purchase.purchase_history_pivot` | pivot |  | `price_total`, `product_uom_qty` |  |  | `purchase` |
| `purchase.purchase_history_graph` | graph |  | `date_order`, `product_uom_qty` |  |  | `purchase` |
| `purchase_requisition.purchase_order_line_compare_tree` | list |  | `product_id`, `partner_id`, `order_id`, `state`, `name`, `date_planned`, `product_qty`, `product_uom_id`, `price_unit`, `price_subtotal`, `currency_id`, `price_total_cc`, `company_currency_id` | `Clear Selected`, `Choose`, `Clear` |  | `purchase_requisition` |
| `purchase_requisition_stock.purchase_order_line_compare_tree_inherit_purchase_requisition_stock` | field | `purchase_requisition.purchase_order_line_compare_tree` | `partner_id`, `on_time_rate_perc` |  |  | `purchase_requisition_stock` |
| `purchase_stock.purchase_order_line_view_form_inherit` | xpath | `purchase.purchase_order_line_form2` | `move_ids` |  |  | `purchase_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `purchase.action_purchase_history` |  | list,pivot,graph |  | `{                 'search_default_order_date_last_year': 1,                 'search_default_groupby_product': 1,         }` |  | `purchase` |

Machine-readable definition: `../../../schemas/data/entities/purchase.order.line.json`; views: `../../../schemas/interfaces/views/purchase.order.line.json`.
