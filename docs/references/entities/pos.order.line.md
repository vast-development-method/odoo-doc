# Point of Sale Order Lines (`pos.order.line`)

**Transport name:** `pos.order.line`  
**Storage name:** `pos_order_line`  
**Kind:** persistent entity (one table)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_restaurant`, `pos_sale`, `l10n_fr_pos_cert`, `l10n_in_pos`, `pos_event`, `pos_loyalty`, `pos_mrp`, `pos_self_order`

Description: Point of Sale Order Lines

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Display name field: `product_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (49)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `company_id` | Company | many to one | `res.company` | related through path `order_id.company_id` and stored |
| `name` | Line No | single line text |  | required; not copied on duplication |
| `notice` | Discount Notice | single line text |  |  |
| `product_id` | Product | many to one | `product.product` | required; restricted by domain `[["sale_ok", "=", true]]` |
| `attribute_value_ids` | Selected Attributes | many to many | `product.template.attribute.value` |  |
| `custom_attribute_value_ids` | Custom Values | one to many | `product.attribute.custom.value` | inverse field `pos_order_line_id` |
| `price_unit` | Unit Price | float |  |  |
| `qty` | Quantity | float |  | default `1`; precision `Product Unit` |
| `price_subtotal` | Tax Excl. | monetary |  | required; read only |
| `price_subtotal_incl` | Tax Incl. | monetary |  | required; read only |
| `price_extra` | Price extra | float |  |  |
| `price_type` | Price Type | selection |  | default `original` |
| `margin` | Margin | monetary |  | computed by rule `_compute_margin` (not stored) |
| `margin_percent` | Margin (%) | float |  | computed by rule `_compute_margin` (not stored); precision `[12, 4]` |
| `total_cost` | Total cost | float |  | read only |
| `is_total_cost_computed` | Is Total Cost Computed | boolean |  | Help: Allows to know if the total cost has already been computed or not |
| `discount` | Discount (%) | float |  | default  |
| `order_id` | Order Ref | many to one | `pos.order` | required; indexed; on delete of the target: cascade |
| `tax_ids` | Taxes | many to many | `account.tax` | read only; association table `account_tax_pos_order_line_rel` |
| `tax_ids_after_fiscal_position` | Taxes to Apply | many to many | `account.tax` | computed by rule `_get_tax_ids_after_fiscal_position` (not stored) |
| `pack_lot_ids` | Lot/serial Number | one to many | `pos.pack.operation.lot` | inverse field `pos_order_line_id` |
| `product_uom_id` | Product Unit | many to one | `uom.uom` | related through path `product_id.uom_id` |
| `currency_id` | Currency | many to one | `res.currency` | related through path `order_id.currency_id` |
| `full_product_name` | Full Product Name | single line text |  |  |
| `customer_note` | Customer Note | single line text |  |  |
| `refund_orderline_ids` | Refund Order Lines | one to many | `pos.order.line` | inverse field `refunded_orderline_id`; Help: Orderlines in this field are the lines that refunded this orderline. |
| `refunded_orderline_id` | Refunded Order Line | many to one | `pos.order.line` | indexed (btree_not_null); Help: If this orderline is a refund, then the refunded orderline is specified in this field. |
| `refunded_qty` | Refunded Quantity | float |  | computed by rule `_compute_refund_qty` (not stored); Help: Number of items refunded in this orderline. |
| `uuid` | Uuid | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication |
| `note` | Product Note | single line text |  |  |
| `combo_parent_id` | Combo Parent | many to one | `pos.order.line` | indexed (btree_not_null) |
| `combo_line_ids` | Combo Lines | one to many | `pos.order.line` | inverse field `combo_parent_id` |
| `combo_item_id` | Combo Item | many to one | `product.combo.item` |  |
| `is_edited` | Edited | boolean |  | default  |
| `extra_tax_data` | Extra Tax Data | structured document |  |  |
| `course_id` | Course Ref | many to one | `restaurant.order.course` | indexed (btree_not_null); on delete of the target: set null |
| `sale_order_origin_id` | Linked Sale Order | many to one | `sale.order` | indexed (btree_not_null) |
| `sale_order_line_id` | Source Sale Order Line | many to one | `sale.order.line` | indexed (btree_not_null) |
| `down_payment_details` | Down Payment Details | multi line text |  |  |
| `qty_delivered` | Delivery Quantity | float |  | computed by rule `_compute_qty_delivered` and stored; not copied on duplication |
| `l10n_in_hsn_code` | harmonized system nomenclature/SAC Code | single line text |  | computed by rule `_compute_l10n_in_hsn_code` and stored; not copied on duplication |
| `event_ticket_id` | Event Ticket | many to one | `event.event.ticket` |  |
| `event_registration_ids` | Event Registrations | one to many | `event.registration` | inverse field `pos_order_line_id` |
| `is_reward_line` | Is Reward Line | boolean |  | Help: Whether this line is part of a reward or not. |
| `reward_id` | Reward | many to one | `loyalty.reward` | indexed (btree_not_null); on delete of the target: restrict; Help: The reward associated with this line. |
| `coupon_id` | Coupon | many to one | `loyalty.card` | indexed (btree_not_null); on delete of the target: restrict; Help: The coupon used to claim that reward. |
| `reward_identifier_code` | Reward Identifier Code | single line text |  | Help: Technical field used to link multiple reward lines from the same reward together. |
| `points_cost` | Points Cost | float |  | Help: How many point this reward cost on the coupon. |
| `combo_id` | Combo reference | many to one | `product.combo` |  |

## Selection values

### `price_type` (Price Type)

| Value | Label |
|---|---|
| `original` | Original |
| `manual` | Manual |
| `automatic` | Automatic |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_uuid` | Constraint | `unique (uuid)` | An order line with this uuid already exists | `point_of_sale` |

## Operations (31)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `l10n_in_pos`, `point_of_sale`, `pos_event`, `pos_loyalty`, `pos_restaurant`, `pos_sale` | model |  |
| `_is_field_accepted` | internal rule | self, field | `point_of_sale` | model |  |
| `_compute_refund_qty` | computation | self | `point_of_sale` | depends: `refund_orderline_ids`, `refund_orderline_ids.order_id.state` |  |
| `_prepare_refund_data` | preparation rule | self, refund_order, PosPackOperationLot | `point_of_sale`, `pos_sale` |  | This prepares data for refund order line. Inheritance may inject more data here  @param refund_order: the pre-created refund order @type refund_order: pos.order  @param PosPackOperationLot: the pre-created Pack operation Lot @type PosPackOperationLot: pos.pack.operation.lot  @return: dictionary of data which is for creating a refund order line from the original line @rtype: dict |
| `create` | lifecycle override | self, vals_list | `point_of_sale`, `pos_self_order` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `l10n_fr_pos_cert`, `point_of_sale`, `pos_self_order` |  |  |
| `get_existing_lots` | operation | self, company_id, config_id, product_id | `point_of_sale` | model | Return the lots that are still available in the given company. The lot is available if its quantity in the corresponding stock_quant and pos stock location is > 0. |
| `_unlink_except_order_state` | internal rule | self | `point_of_sale` | ondelete |  |
| `_onchange_amount_line_all` | on change | self | `point_of_sale` | onchange: `price_unit`, `tax_ids`, `qty`, `discount`, `product_id` |  |
| `_compute_amount_line_all` | computation | self | `point_of_sale` |  |  |
| `_onchange_product_id` | on change | self | `point_of_sale` | onchange: `product_id` |  |
| `_onchange_qty` | on change | self | `point_of_sale` | onchange: `qty`, `discount`, `price_unit`, `tax_ids` |  |
| `_get_tax_ids_after_fiscal_position` | computation | self | `point_of_sale` | depends: `order_id`, `order_id.fiscal_position_id`, `tax_ids` |  |
| `_prepare_reference_vals` | preparation rule | self | `point_of_sale` |  |  |
| `_prepare_procurement_values` | preparation rule | self | `point_of_sale` |  | Prepare specific key for moves or other components that will be created from a stock rule coming from a sale order line. This method could be override in order to add other custom key that could be used in move/po creation. |
| `_launch_stock_rule_from_pos_order_lines` | internal rule | self | `point_of_sale`, `pos_sale` |  |  |
| `_is_product_storable_fifo_avco` | internal rule | self | `point_of_sale` |  |  |
| `_get_product_cost_with_moves` | preparation rule | self, moves | `point_of_sale`, `pos_mrp` |  |  |
| `_compute_total_cost` | computation | self, stock_moves | `point_of_sale` |  | Compute the total cost of the order lines. :param stock_moves: recordset of `stock.move`, used for fifo/avco lines |
| `_get_stock_moves_to_consider` | preparation rule | self, stock_moves, product | `point_of_sale`, `pos_mrp` |  |  |
| `_compute_margin` | computation | self | `point_of_sale` | depends: `price_subtotal`, `total_cost` |  |
| `_prepare_base_line_for_taxes_computation` | preparation rule | self | `l10n_in_pos`, `point_of_sale` |  |  |
| `_prepare_tax_base_line_values` | preparation rule | self | `point_of_sale` |  | Convert pos order lines into dictionaries that would be used to compute taxes later.  :return: A list of python dictionaries (see '_prepare_base_line_for_taxes_computation' in account.tax). |
| `unlink` | lifecycle override | self | `point_of_sale` |  |  |
| `_get_discount_amount` | preparation rule | self | `point_of_sale` |  |  |
| `_get_discount_amount_for_report` | preparation rule | self | `point_of_sale`, `pos_loyalty` |  |  |
| `_has_discount` | internal rule | self | `point_of_sale`, `pos_loyalty` |  |  |
| `isRefund` | operation | self | `point_of_sale`, `pos_loyalty` |  |  |
| `_compute_qty_delivered` | computation | self | `pos_sale` | depends: `order_id.state`, `order_id.picking_ids`, `order_id.picking_ids.state`, `order_id.picking_ids.move_ids.quantity` |  |
| `_compute_l10n_in_hsn_code` | computation | self | `l10n_in_pos` | depends: `product_id` |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `get_existing_lots` | UserError | No PoS configuration found | `point_of_sale` |
| `_unlink_except_order_state` | UserError | You can only unlink PoS order lines that are related to orders in new or cancelled state. | `point_of_sale` |
| `_onchange_qty` | ValidationError | You cannot refund more than the outstanding quantity for this product. | `point_of_sale` |
| `_prepare_base_line_for_taxes_computation` | UserError | Please define income account for this product: '%(product)s' (id:%(id)d). | `point_of_sale` |
| `write` | UserError | According to the French law, you cannot modify a point of sale order line. Forbidden fields: %s. | `l10n_fr_pos_cert` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Point Of Sale Order Line | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_order_line` | list |  | `product_id`, `qty`, `discount`, `price_unit`, `price_subtotal`, `price_subtotal_incl`, `create_date`, `currency_id` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_line_form` | form |  | `product_id`, `qty`, `discount`, `price_unit`, `create_date`, `currency_id` |  |  | `point_of_sale` |
| `point_of_sale.view_pos_order_tree_all_sales_lines` | list |  | `order_id`, `create_date`, `product_id`, `qty`, `price_unit`, `currency_id` |  |  | `point_of_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_pos_order_line` | Sale line | list |  |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_line_form` | Sale line | form,list |  |  |  | `point_of_sale` |
| `point_of_sale.action_pos_order_line_day` | Sale line | list | `[('create_date', '>=', 'today'), ('create_date', '<', 'today +1d')]` |  |  | `point_of_sale` |
| `point_of_sale.action_pos_all_sales_lines` | All sales lines |  |  |  |  | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.order.line.json`; views: `../../../schemas/interfaces/views/pos.order.line.json`.
