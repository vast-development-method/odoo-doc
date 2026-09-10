# Repair Order (`repair.order`)

**Transport name:** `repair.order`  
**Storage name:** `repair_order`  
**Kind:** persistent entity (one table)  
**Defined by package:** `repair`  
**Extended by packages:** `l10n_din5008_repair`, `mrp_repair`, `purchase_repair`

Description: Repair Order

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `product.catalog.mixin`
- Default ordering: `priority desc, create_date desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (46)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Repair Reference | single line text |  | required; read only; default computed dynamically (lambda self: _('New')); indexed (trigram); not copied on duplication |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company); indexed |
| `state` | Status | selection |  | read only; default `draft`; changes are tracked in the message thread; indexed; not copied on duplication; Help: * The 'New' status is used when a user is encoding a new and unconfirmed repair order. * The 'Confirmed' status is used when a user confirms the repair order. * The 'Under Repair' status is used when the repair is ongoing. * The 'Repaired' status is set when repairing is completed. * The 'Cancelled' status is used when user cancel repair order. |
| `priority` | Priority | selection |  | default `0` |
| `partner_id` | Customer | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; indexed; must belong to the same company; Help: Choose partner for whom the order will be invoiced and delivered. You can find a partner by its Name, TIN, Email or Internal Reference. |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); must belong to the same company |
| `internal_notes` | Internal Notes | rich text |  |  |
| `tag_ids` | Tags | many to many | `repair.tags` |  |
| `under_warranty` | Under Warranty | boolean |  | Help: If ticked, the sales price will be set to 0 for all products transferred from the repair order. |
| `schedule_date` | Scheduled Date | date and time |  | required; default computed dynamically (fields.Datetime.now); indexed; not copied on duplication |
| `search_date_category` | Date Category | selection |  | read only; searchable through a search rule |
| `repair_properties` | Properties | properties |  |  |
| `move_id` | Inventory Move | many to one | `stock.move` | read only; changes are tracked in the message thread; not copied on duplication; must belong to the same company |
| `product_id` | Product to Repair | many to one | `product.product` | restricted by domain `[('type', '=', 'consu'), '\|', ('company_id', '=', company_id), ('company_id', '=', False), '\|', ('id', 'in', picking_product_ids), ('id', '=?', picking_product_id)]`; must belong to the same company |
| `product_qty` | Product Quantity | float |  | computed by rule `_compute_product_qty` and stored; default `1.0`; precision `Product Unit` |
| `allowed_uom_ids` | Allowed Unit of measure | many to many | `uom.uom` | computed by rule `_compute_allowed_uom_ids` (not stored) |
| `product_uom` | Unit | many to one | `uom.uom` | computed by rule `compute_product_uom` and stored; restricted by domain `[('id', 'in', allowed_uom_ids)]`; precomputed before insertion |
| `lot_id` | Lot/Serial | many to one | `stock.lot` | computed by rule `compute_lot_id` and stored; restricted by domain `[('id', 'in', allowed_lot_ids)]`; must belong to the same company; Help: Products repaired are all belonging to this lot |
| `tracking` | Product Tracking | selection |  | related through path `product_id.tracking` |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; computed by rule `_compute_picking_type_id` and stored; default computed dynamically (_default_picking_type_id); indexed; restricted by domain `[('code', '=', 'repair_operation'), ('company_id', '=', company_id)]`; must belong to the same company; precomputed before insertion |
| `reference_ids` | References | many to many | `stock.reference` | not copied on duplication; association table `stock_reference_repair_rel` |
| `location_id` | Component Source Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the components of product to repair is located. |
| `product_location_src_id` | Product Source Location | many to one | `stock.location` | required; computed by rule `_compute_product_location_src_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the product to repair is located. |
| `product_location_dest_id` | Product Destination Location | many to one | `stock.location` | required; computed by rule `_compute_product_location_dest_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the repaired product is located. |
| `location_dest_id` | Added Parts Destination Location | many to one | `stock.location` | required; read only; related through path `picking_type_id.default_location_dest_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the repaired product is located. |
| `parts_location_id` | Removed Parts Destination Location | many to one | `stock.location` | required; read only; related through path `picking_type_id.default_remove_location_dest_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the repair parts are located. |
| `recycle_location_id` | Recycled Parts Destination Location | many to one | `stock.location` | required; computed by rule `_compute_recycle_location_id` and stored; indexed; must belong to the same company; precomputed before insertion; Help: This is the location where the repair parts are located. |
| `move_ids` | Parts | one to many | `stock.move` | restricted by domain `[["repair_line_type", "!=", false]]`; must belong to the same company; inverse field `repair_id` |
| `parts_availability` | Component Status | single line text |  | computed by rule `_compute_parts_availability` (not stored); Help: Latest parts availability status for this RO. If green, then the RO's readiness status is ready. |
| `parts_availability_state` | Parts Availability State | selection |  | computed by rule `_compute_parts_availability` (not stored) |
| `is_parts_available` | All Parts are available | boolean |  | computed by rule `_compute_availability_boolean` and stored; default  |
| `is_parts_late` | Any Part is late | boolean |  | computed by rule `_compute_availability_boolean` and stored; default  |
| `sale_order_id` | Sale Order | many to one | `sale.order` | read only; indexed (btree_not_null); not copied on duplication; must belong to the same company; Help: Sale Order from which the Repair Order comes from. |
| `sale_order_line_id` | Sale Order Line | many to one | `sale.order.line` | read only; not copied on duplication; must belong to the same company; Help: Sale Order Line from which the Repair Order comes from. |
| `repair_request` | Repair Request | multi line text |  | related through path `sale_order_line_id.name`; Help: Sale Order Line Description. |
| `picking_id` | Transfer | many to one | `stock.picking` | indexed (btree_not_null); not copied on duplication; restricted by domain `[('return_id', '!=', False), ('product_id', '=?', product_id)]`; must belong to the same company; Help: Transfer from which the product to be repaired is picked |
| `picking_product_ids` | Picking Product | one to many | `product.product` | computed by rule `_compute_picking_product_ids` (not stored) |
| `picking_product_id` | Picking Product | many to one |  | related through path `picking_id.product_id` |
| `allowed_lot_ids` | Allowed Lot | one to many | `stock.lot` | computed by rule `_compute_allowed_lot_ids` (not stored) |
| `has_uncomplete_moves` | Has Uncomplete Moves | boolean |  | computed by rule `_compute_has_uncomplete_moves` (not stored) |
| `unreserve_visible` | Allowed to Unreserve Production | boolean |  | computed by rule `_compute_unreserve_visible` (not stored); Help: Technical field to check when we can unreserve |
| `reserve_visible` | Allowed to Reserve Production | boolean |  | computed by rule `_compute_unreserve_visible` (not stored); Help: Technical field to check when we can reserve quantities |
| `picking_type_visible` | Picking Type Visible | boolean |  | computed by rule `_compute_picking_type_visible` (not stored) |
| `l10n_din5008_printing_date` | Localization Din5008 Printing Date | date |  | default computed dynamically (fields.Date.today) |
| `production_count` | Count of manufacturing orders generated | integer |  | computed by rule `_compute_production_count` (not stored); visible only to groups `mrp.group_mrp_user` |
| `purchase_count` | Count of generated purchase orders | integer |  | computed by rule `_compute_purchase_count` (not stored); visible only to groups `purchase.group_purchase_user` |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | New |
| `confirmed` | Confirmed |
| `under_repair` | Under Repair |
| `done` | Repaired |
| `cancel` | Cancelled |

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Normal |
| `1` | Urgent |

### `search_date_category` (Date Category)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

### `parts_availability_state` (Parts Availability State)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

## State fields

State machine fields of this entity: `state`, `parts_availability_state`. Transitions are specified in the domain documents.

## Operations (58)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_picking_type_id` | preparation rule | self | `repair` | model |  |
| `_compute_picking_type_visible` | computation | self | `repair` |  |  |
| `_compute_product_qty` | computation | self | `repair` | depends: `product_id`, `picking_id`, `lot_id` |  |
| `_compute_allowed_uom_ids` | computation | self | `repair` | depends: `product_id`, `product_id.uom_id`, `product_id.uom_ids`, `product_id.seller_ids`, `product_id.seller_ids.product_uom_id` |  |
| `_compute_partner_id` | computation | self | `repair` | depends: `picking_id` |  |
| `_compute_picking_product_ids` | computation | self | `repair` | depends: `picking_id` |  |
| `_compute_allowed_lot_ids` | computation | self | `repair` | depends: `product_id`, `company_id`, `picking_id`, `picking_id.move_ids`, `picking_id.move_ids.lot_ids` |  |
| `compute_product_uom` | computation | self | `repair` | depends: `product_id`, `product_id.uom_id` |  |
| `compute_lot_id` | computation | self | `repair` | depends: `product_id`, `lot_id`, `lot_id.product_id`, `picking_id` |  |
| `_compute_picking_type_id` | computation | self | `repair` | depends: `company_id` |  |
| `_compute_location_id` | computation | self | `repair` | depends: `picking_type_id` |  |
| `_compute_product_location_src_id` | computation | self | `repair` | depends: `picking_type_id` |  |
| `_compute_product_location_dest_id` | computation | self | `repair` | depends: `picking_type_id` |  |
| `_compute_recycle_location_id` | computation | self | `repair` | depends: `picking_type_id` |  |
| `_compute_parts_availability` | computation | self | `repair` | depends: `state`, `schedule_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` |  |
| `_compute_availability_boolean` | computation | self | `repair` | depends: `parts_availability_state` |  |
| `_compute_has_uncomplete_moves` | computation | self | `repair` | depends: `move_ids.quantity`, `move_ids.product_uom_qty`, `move_ids.product_uom.rounding` |  |
| `_compute_unreserve_visible` | computation | self | `repair` | depends: `move_ids`, `state`, `move_ids.product_uom_qty` |  |
| `_search_date_category` | search rule | self, operator, value | `repair` |  |  |
| `onchange_product_uom` | on change | self | `repair` | onchange: `product_uom` |  |
| `_onchange_location_picking` | on change | self | `repair` | onchange: `location_id`, `picking_id` |  |
| `default_get` | lifecycle override | self, fields | `repair` | model |  |
| `create` | lifecycle override | self, vals_list | `mrp_repair`, `repair` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `mrp_repair`, `repair` |  |  |
| `_unlink_except_confirmed` | internal rule | self | `repair` | ondelete |  |
| `action_generate_serial` | user action | self | `repair` |  |  |
| `action_assign` | user action | self | `repair` |  |  |
| `action_create_sale_order` | user action | self | `repair` |  |  |
| `action_repair_cancel` | user action | self | `repair` |  |  |
| `action_repair_cancel_draft` | user action | self | `repair` |  |  |
| `action_repair_done` | user action | self | `repair` |  | Creates stock move for final product of repair order. Writes move_id and move_ids state to 'done'. Writes repair order state to 'Repaired'. @return: True |
| `action_repair_end` | user action | self | `repair` |  | Checks before action_repair_done. @return: True |
| `action_repair_start` | user action | self | `repair` |  | Writes repair order state to 'Under Repair' |
| `action_unreserve` | user action | self | `repair` |  |  |
| `action_validate` | user action | self | `repair` |  |  |
| `action_view_sale_order` | user action | self | `repair` |  |  |
| `print_repair_order` | operation | self | `repair` |  |  |
| `_action_repair_confirm` | internal rule | self | `repair` |  | Repair order state is set to 'Confirmed'. @param *arg: Arguments @return: True |
| `_get_location` | preparation rule | self, field | `repair` |  |  |
| `_get_picking_type` | preparation rule | self | `repair` |  |  |
| `_update_sale_order_line_price` | internal rule | self | `repair` |  |  |
| `_get_sale_order_values` | preparation rule | self | `repair` |  |  |
| `_create_sale_order` | internal rule | self | `repair` |  |  |
| `action_add_from_catalog` | user action | self | `repair` |  |  |
| `_default_order_line_values` | preparation rule | self, child_field | `repair` |  |  |
| `_get_product_catalog_domain` | preparation rule | self | `repair` |  |  |
| `_get_product_catalog_order_data` | preparation rule | self, products, **kwargs | `repair` |  |  |
| `_get_product_price_and_data` | preparation rule | self, product | `repair` |  |  |
| `_get_product_catalog_record_lines` | preparation rule | self, product_ids, **kwargs | `repair` |  |  |
| `_is_display_stock_in_catalog` | internal rule | self | `repair` |  |  |
| `_update_order_line_info` | internal rule | self, product_id, quantity, **kwargs | `repair` |  |  |
| `message_post` | messaging hook | self, **kwargs | `repair` |  |  |
| `_compute_production_count` | computation | self | `mrp_repair` | depends: `reference_ids.production_ids` |  |
| `action_explode` | user action | self | `mrp_repair` |  |  |
| `action_view_mrp_productions` | user action | self | `mrp_repair` |  |  |
| `_get_action_add_from_catalog_extra_context` | preparation rule | self | `mrp_repair` |  |  |
| `_compute_purchase_count` | computation | self | `purchase_repair` | depends: `move_ids.created_purchase_line_ids.order_id` |  |
| `action_view_purchase_orders` | user action | self | `purchase_repair` |  |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_generate_serial` | UserError | Please set the first Serial Number or a default sequence | `repair` |
| `action_repair_cancel` | UserError | You cannot cancel a Repair Order that's already been completed | `repair` |
| `action_repair_done` | ValidationError | Serial number is required for product to repair : %s | `repair` |
| `action_repair_end` | UserError | Repair must be under repair in order to end reparation. | `repair` |
| `action_validate` | UserError | You can not enter negative quantities. | `repair` |
| `_create_sale_order` | UserError | You cannot create a quotation for a repair order that is already linked to an existing sale order. Concerned repair order(s): %(ref_str)s | `repair` |
| `_create_sale_order` | UserError | You need to define a customer for a repair order in order to create an associated quotation. Concerned repair order(s): %(ref_str)s | `repair` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `repair` |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mrp_repair.view_repair_order_form_inherit` | xpath | `repair.view_repair_order_form` | `production_count` | `action_view_mrp_productions` |  | `mrp_repair` |
| `purchase_repair.view_repair_order_form_inherit` | xpath | `repair.view_repair_order_form` | `purchase_count` | `action_view_purchase_orders` |  | `purchase_repair` |
| `repair.repair_order_view_activity` | activity |  | `user_id`, `name`, `product_id`, `schedule_date` |  |  | `repair` |
| `repair.view_repair_order_tree` | list |  | `company_id`, `priority`, `name`, `schedule_date`, `product_id`, `parts_availability_state`, `parts_availability`, `product_qty`, `product_uom`, `user_id`, `partner_id`, `picking_id`, `sale_order_id`, `location_id`, `company_id`, `state`, `activity_exception_decoration` |  |  | `repair` |
| `repair.view_repair_order_form` | form |  | `has_uncomplete_moves`, `unreserve_visible`, `reserve_visible`, `state`, `sale_order_id`, `priority`, `name`, `allowed_lot_ids`, `repair_request`, `partner_id`, `product_id`, `lot_id`, `product_qty`, `product_uom`, `picking_id`, `under_warranty`, `schedule_date`, `user_id`, `company_id`, `tag_ids`, `parts_availability_state`, `parts_availability`, `picking_type_id`, `repair_properties`, `move_ids`, `company_id`, `state`, `repair_line_type`, `picking_type_id`, `location_id`, `location_dest_id`, `partner_id`, `picking_code`, `show_details_visible`, `additional`, `move_lines_count`, `is_locked`, `is_storable`, `has_tracking`, `display_assign_serial`, `product_id`, `forecast_availability`, `description_picking`, `date`, `date_deadline`, `product_uom_qty`, `forecast_expected_date`, `product_qty`, `quantity`, `product_uom`, `picked`, `lot_ids`, `internal_notes` | `Confirm Repair`, `Start Repair`, `End Repair`, `End Repair`, `Check availability`, `Unreserve`, `Create Quotation`, `Cancel Repair`, `Set to Draft`, `Sale Order`, `Product Moves`, `action_generate_serial`, `Catalog`, `Details` |  | `repair` |
| `repair.view_repair_kanban` | kanban |  | `name`, `state`, `product_id`, `tag_ids`, `partner_id` |  |  | `repair` |
| `repair.view_repair_order_form_filter` | search |  | `name`, `product_id`, `partner_id`, `sale_order_id` |  | `New`, `Confirmed`, `Under Repair`, `Repaired`, `Cancelled`, `Returned`, `Before`, `Yesterday`, `Today`, `Tomorrow`, `The day after tomorrow`, `After`, `Ready`, `Late`, `filter_create_date`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Customer`, `Product`, `Status`, `Company`, `Properties` | `repair` |
| `repair.view_repair_graph` | graph |  | `create_date`, `product_id` |  |  | `repair` |
| `repair.view_repair_pivot` | pivot |  | `create_date`, `product_id` |  |  | `repair` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `repair.action_repair_order_form` | Repair Orders | form |  |  |  | `repair` |
| `repair.action_repair_order_tree` | Repair Orders | list,kanban,graph,pivot,form,activity |  |  |  | `repair` |
| `repair.action_repair_order_graph` | Repair Orders Analysis | list,kanban,graph,pivot,form |  | `{                 'search_default_product': 1,                 'search_default_createDate': 1,             }` |  | `repair` |
| `repair.action_picking_repair` | Repair Orders | list,kanban,form | `[('picking_type_id', '=', active_id)]` | `{'default_picking_type_id': active_id}` |  | `repair` |
| `repair.action_picking_repair_graph` | Repair Orders | list,kanban,form | `[]` | `{'search_default_filter_confirmed': 1}` |  | `repair` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `repair.action_report_repair_order` | Repair Order | qweb-pdf | `repair.report_repairorder2` | `('Repair Order - %s' % (object.name))` |  |

Machine-readable definition: `../../../schemas/data/entities/repair.order.json`; views: `../../../schemas/interfaces/views/repair.order.json`.
