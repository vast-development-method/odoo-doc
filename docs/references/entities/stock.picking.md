# Transfer (`stock.picking`)

**Transport name:** `stock.picking`  
**Storage name:** `stock_picking`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_account`, `sale_stock`, `stock_delivery`, `stock_picking_batch`, `delivery_stock_picking_batch`, `point_of_sale`, `l10n_ar_stock`, `pos_sale`, `repair`, `l10n_in_stock`, `l10n_in_ewaybill_stock`, `purchase_stock`, `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_it_stock_ddt`, `l10n_ro_edi_stock`, `l10n_ro_edi_stock_batch`, `l10n_tr_nilvera_edispatch`, `mrp`, `product_expiry`, `mrp_subcontracting`, `stock_dropshipping`, `mrp_subcontracting_dropshipping`, `mrp_subcontracting_purchase`, `pos_repair`, `project_stock`, `sale_project_stock`, `stock_fleet`, `stock_sms`, `website_sale_stock`

Description: Transfer

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `priority desc, scheduled_date asc, id desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (139)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Reference | single line text |  | read only; default `/`; indexed (trigram); not copied on duplication |
| `origin` | Source Document | single line text |  | indexed (trigram); Help: Reference of the document |
| `note` | Notes | rich text |  |  |
| `backorder_id` | Back Order of | many to one | `stock.picking` | read only; indexed (btree_not_null); not copied on duplication; must belong to the same company; Help: If this shipment was split, then this field links to the shipment which contains the already processed part. |
| `backorder_ids` | Back Orders | one to many | `stock.picking` | inverse field `backorder_id` |
| `return_id` | Return of | many to one | `stock.picking` | read only; indexed (btree_not_null); not copied on duplication; must belong to the same company; Help: If this picking was created as a return of another picking, this field links to the original picking. |
| `return_ids` | Returns | one to many | `stock.picking` | inverse field `return_id` |
| `return_count` | # Returns | integer |  | computed by rule `_compute_return_count` (not stored) |
| `move_type` | Shipping Policy | selection |  | required; computed by rule `_compute_move_type` and stored; precomputed before insertion; Help: It specifies goods to be deliver partially or all at once |
| `state` | Status | selection |  | read only; computed by rule `_compute_state` and stored; changes are tracked in the message thread; indexed; not copied on duplication; Help: * Draft: The transfer is not confirmed yet. Reservation doesn't apply.  * Waiting another operation: This transfer is waiting for another operation before being ready.  * Waiting: The transfer is waiting for the availability of some products. (a) The shipping policy is "As soon as possible": no product could be reserved. (b) The shipping policy is "When all products are ready": not all the products could be reserved.  * Ready: The transfer is ready to be processed. (a) The shipping policy is "As soon as possible": at least one product has been reserved. (b) The shipping policy is "When all products are ready": all product have been reserved.  * Done: The transfer has been processed.  * Cancelled: The transfer has been cancelled. |
| `reference_ids` | References | many to many | `stock.reference` | read only; related through path `move_ids.reference_ids` |
| `priority` | Priority | selection |  | default `0`; Help: Products will be reserved first for the transfers with the highest priorities. |
| `scheduled_date` | Scheduled Date | date and time |  | computed by rule `_compute_scheduled_date` and stored; writable through an inverse rule; default computed dynamically (fields.Datetime.now); changes are tracked in the message thread; indexed; Help: Scheduled time for the first part of the shipment to be processed. Setting manually a value here would set it as expected date for all the stock moves. |
| `date_deadline` | Deadline | date and time |  | computed by rule `_compute_date_deadline` and stored; Help: In case of outgoing flow, validate the transfer before this date to allow to deliver at promised date to the customer.         In case of incoming flow, validate the transfer before this date in order to have these products in stock at the date promised by the supplier |
| `has_deadline_issue` | Is late | boolean |  | computed by rule `_compute_has_deadline_issue` and stored; default ; Help: Is late or will be late depending on the deadline and scheduled date |
| `date_done` | Date of Transfer | date and time |  | not copied on duplication; Help: Date at which the transfer has been processed or cancelled. |
| `delay_alert_date` | Delay Alert Date | date and time |  | computed by rule `_compute_delay_alert_date` (not stored); searchable through a search rule |
| `json_popover` | JavaScript Object Notation data for the popover widget | single line text |  | computed by rule `_compute_json_popover` (not stored) |
| `location_id` | Source Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; must belong to the same company; precomputed before insertion |
| `location_dest_id` | Destination Location | many to one | `stock.location` | required; computed by rule `_compute_location_id` and stored; must belong to the same company; precomputed before insertion |
| `move_ids` | Stock Moves | one to many | `stock.move` | inverse field `picking_id` |
| `has_scrap_move` | Has Scrap Moves | boolean |  | computed by rule `_has_scrap_move` (not stored) |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | required; default computed dynamically (_default_picking_type_id); changes are tracked in the message thread; indexed |
| `warehouse_address_id` | Warehouse Address | many to one | `res.partner` | related through path `picking_type_id.warehouse_id.partner_id` |
| `picking_type_code` | Picking Type Code | selection |  | read only; related through path `picking_type_id.code` |
| `picking_type_entire_packs` | Picking Type Entire Packs | boolean |  | related through path `picking_type_id.show_entire_packs` |
| `use_create_lots` | Use Create Lots | boolean |  | related through path `picking_type_id.use_create_lots` |
| `use_existing_lots` | Use Existing Lots | boolean |  | related through path `picking_type_id.use_existing_lots` |
| `partner_id` | Contact | many to one | `res.partner` | indexed (btree_not_null); must belong to the same company |
| `company_id` | Company | many to one | `res.company` | read only; related through path `picking_type_id.company_id` and stored; indexed |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; not copied on duplication; restricted by domain `lambda self: [('all_group_ids', 'in', self.env.ref('stock.group_stock_user').id)]` |
| `move_line_ids` | Operations | one to many | `stock.move.line` | inverse field `picking_id` |
| `packages_count` | Packages Count | integer |  | computed by rule `_compute_packages_count` (not stored) |
| `package_history_ids` | Transfered Packages | many to many | `stock.package.history` | not copied on duplication |
| `show_check_availability` | Show Check Availability | boolean |  | computed by rule `_compute_show_check_availability` (not stored); Help: Technical field used to compute whether the button "Check Availability" should be displayed. |
| `show_allocation` | Show Allocation | boolean |  | computed by rule `_compute_show_allocation` (not stored); Help: Technical Field used to decide whether the button "Allocation" should be displayed. |
| `owner_id` | Assign Owner | many to one | `res.partner` | indexed (btree_not_null); must belong to the same company; Help: When validating the transfer, the products will be assigned to this owner. |
| `printed` | Printed | boolean |  | not copied on duplication |
| `signature` | Signature | image |  | not copied on duplication; Help: Signature |
| `is_signed` | Is Signed | boolean |  | computed by rule `_compute_is_signed` (not stored) |
| `is_locked` | Is Locked | boolean |  | default `True`; not copied on duplication; Help: When the picking is not done this allows changing the initial demand. When the picking is done this allows changing the done quantities. |
| `is_date_editable` | Is Scheduled Date Editable | boolean |  | computed by rule `_compute_is_date_editable` (not stored) |
| `weight_bulk` | Bulk Weight | float |  | computed by rule `_compute_bulk_weight` (not stored); Help: Total weight of products which are not in a package. |
| `shipping_weight` | Weight for Shipping | float |  | computed by rule `_compute_shipping_weight` and stored; precision `Stock Weight`; Help: Total weight of packages and products not in a package. Packages with no shipping weight specified will default to their products' total weight. This is the weight used to compute the cost of the shipping. |
| `shipping_volume` | Volume for Shipping | float |  | computed by rule `_compute_shipping_volume` (not stored) |
| `product_id` | Product | many to one | `product.product` | read only; related through path `move_ids.product_id` |
| `lot_id` | Lot/Serial Number | many to one | `stock.lot` | read only; related through path `move_line_ids.lot_id` |
| `show_operations` | Show Operations | boolean |  | related through path `picking_type_id.show_operations` |
| `show_lots_text` | Show Lots Text | boolean |  | computed by rule `_compute_show_lots_text` (not stored) |
| `has_tracking` | Has Tracking | boolean |  | computed by rule `_compute_has_tracking` (not stored) |
| `products_availability` | Product Availability | single line text |  | computed by rule `_compute_products_availability` (not stored); Help: Latest product availability status of the picking |
| `products_availability_state` | Products Availability State | selection |  | computed by rule `_compute_products_availability` (not stored); searchable through a search rule |
| `picking_properties` | Properties | properties |  |  |
| `show_next_pickings` | Show Next Pickings | boolean |  | computed by rule `_compute_show_next_pickings` (not stored) |
| `search_date_category` | Date Category | selection |  | read only; searchable through a search rule |
| `partner_country_id` | Partner Country | many to one | `res.country` | related through path `partner_id.country_id` |
| `picking_warning_text` | Picking Instructions | multi line text |  | computed by rule `_compute_picking_warning_text` (not stored); Help: Internal instructions for the partner or its parent company as set by the user. |
| `country_code` | Country Code | single line text |  | related through path `company_id.account_fiscal_country_id.code` |
| `sale_id` | Sales Order | many to one | `sale.order` | computed by rule `_compute_sale_id` and stored; writable through an inverse rule; indexed (btree_not_null) |
| `carrier_price` | Shipping Cost | float |  |  |
| `delivery_type` | Delivery Type | selection |  | read only; related through path `carrier_id.delivery_type` |
| `allowed_carrier_ids` | Allowed Carrier | many to many | `delivery.carrier` | computed by rule `_compute_allowed_carrier_ids` (not stored) |
| `carrier_id` | Carrier | many to one | `delivery.carrier` | restricted by domain `[('id', 'in', allowed_carrier_ids)]`; must belong to the same company |
| `weight` | Weight | float |  | computed by rule `_cal_weight` and stored; precision `Stock Weight`; Help: Total weight of the products in the picking. |
| `carrier_tracking_ref` | Tracking Reference | single line text |  | not copied on duplication |
| `carrier_tracking_url` | Tracking uniform resource locator | single line text |  | computed by rule `_compute_carrier_tracking_url` (not stored) |
| `weight_uom_name` | Weight unit of measure label | single line text |  | read only; computed by rule `_compute_weight_uom_name` (not stored); default computed dynamically (_get_default_weight_uom) |
| `is_return_picking` | Is Return Picking | boolean |  | computed by rule `_compute_return_picking` (not stored) |
| `return_label_ids` | Return Label | one to many | `ir.attachment` | computed by rule `_compute_return_label` (not stored) |
| `destination_country_code` | Destination Country | single line text |  | related through path `partner_id.country_id.code` |
| `integration_level` | Integration Level | selection |  | related through path `carrier_id.integration_level` |
| `batch_id` | Batch Transfer | many to one | `stock.picking.batch` | indexed; not copied on duplication; must belong to the same company; Help: Batch associated to this transfer |
| `batch_sequence` | Sequence | integer |  |  |
| `pos_session_id` | Point of sale Session | many to one | `pos.session` | indexed |
| `pos_order_id` | Point of sale Order | many to one | `pos.order` | indexed |
| `l10n_ar_delivery_guide_number` | Delivery Guide No. | single line text |  | read only; not copied on duplication |
| `l10n_ar_cai_data` | CAI Data | structured document |  | not copied on duplication |
| `l10n_ar_allow_generate_delivery_guide` | Localization Ar Allow Generate Delivery Guide | boolean |  | computed by rule `_compute_l10n_ar_delivery_guide_flags` (not stored) |
| `l10n_ar_allow_send_delivery_guide` | Localization Ar Allow Send Delivery Guide | boolean |  | computed by rule `_compute_l10n_ar_delivery_guide_flags` (not stored) |
| `repair_ids` | Repair | one to many | `repair.order` | inverse field `picking_id` |
| `nbr_repairs` | Number of repairs linked to this picking | integer |  | computed by rule `_compute_nbr_repairs` (not stored) |
| `l10n_in_ewaybill_ids` | Ewaybill | one to many | `l10n.in.ewaybill` | inverse field `picking_id` |
| `l10n_in_ewaybill_name` | Indian Ewaybill Number | single line text |  | computed by rule `_compute_l10n_in_ewaybill_details` (not stored) |
| `l10n_in_ewaybill_feature_enabled` | Localization In Electronic waybill Feature Enabled | boolean |  | related through path `company_id.l10n_in_ewaybill_feature` |
| `purchase_id` | Purchase Orders | many to one | `purchase.order` | read only; related through path `move_ids.purchase_line_id.order_id` |
| `days_to_arrive` | Days To Arrive | date and time |  | computed by rule `_compute_effective_date` (not stored); searchable through a search rule; not copied on duplication |
| `delay_pass` | Delay Pass | date and time |  | computed by rule `_compute_date_order` (not stored); searchable through a search rule; indexed; not copied on duplication |
| `l10n_it_transport_reason` | Transport Reason | selection |  | default `sale`; changes are tracked in the message thread |
| `l10n_it_transport_method` | Transport Method | selection |  | default `sender` |
| `l10n_it_transport_method_details` | Transport Note | single line text |  |  |
| `l10n_it_parcels` | Parcels | integer |  | default `1` |
| `l10n_it_ddt_number` | transport document Number | single line text |  | read only |
| `l10n_it_show_print_ddt_button` | Localization It Show Print Transport document Button | boolean |  | computed by rule `_compute_l10n_it_show_print_ddt_button` (not stored) |
| `l10n_ro_edi_stock_document_ids` | Localization Ro Electronic data interchange Stock Document | one to many | `l10n_ro_edi.document` | inverse field `picking_id` |
| `l10n_ro_edi_stock_document_uit` | eTransport UIT | single line text |  | computed by rule `_compute_l10n_ro_edi_stock_current_document_uit` (not stored) |
| `l10n_ro_edi_stock_state` | eTransport Status | selection |  | computed by rule `_compute_l10n_ro_edi_stock_current_document_state` and stored |
| `l10n_ro_edi_stock_operation_type` | eTransport Operation Type | selection |  |  |
| `l10n_ro_edi_stock_available_operation_scopes` | Localization Ro Electronic data interchange Stock Available Operation Scopes | single line text |  | computed by rule `_compute_l10n_ro_edi_stock_available_operation_scopes` (not stored) |
| `l10n_ro_edi_stock_operation_scope` | Operation Scope | selection |  |  |
| `l10n_ro_edi_stock_vehicle_number` | Vehicle Number | single line text |  | maximum length 20 |
| `l10n_ro_edi_stock_trailer_1_number` | Trailer 1 Number | single line text |  | maximum length 20 |
| `l10n_ro_edi_stock_trailer_2_number` | Trailer 2 Number | single line text |  | maximum length 20 |
| `l10n_ro_edi_stock_available_start_loc_types` | Localization Ro Electronic data interchange Stock Available Start Loc Types | single line text |  | computed by rule `_compute_l10n_ro_edi_stock_available_location_types` (not stored) |
| `l10n_ro_edi_stock_start_loc_type` | Start Location Type | selection |  | computed by rule `_compute_l10n_ro_edi_stock_default_location_type` and stored |
| `l10n_ro_edi_stock_available_end_loc_types` | Localization Ro Electronic data interchange Stock Available End Loc Types | single line text |  | computed by rule `_compute_l10n_ro_edi_stock_available_location_types` (not stored) |
| `l10n_ro_edi_stock_end_loc_type` | End Location Type | selection |  | computed by rule `_compute_l10n_ro_edi_stock_default_location_type` and stored |
| `l10n_ro_edi_stock_start_bcp` | Start Border Crossing Point | selection |  |  |
| `l10n_ro_edi_stock_start_customs_office` | Start Customs Office | selection |  |  |
| `l10n_ro_edi_stock_end_bcp` | End Border Crossing Point | selection |  |  |
| `l10n_ro_edi_stock_end_customs_office` | End Customs Office | selection |  |  |
| `l10n_ro_edi_stock_remarks` | Remarks | multi line text |  |  |
| `l10n_ro_edi_stock_enable` | Localization Ro Electronic data interchange Stock Enable | boolean |  | computed by rule `_compute_l10n_ro_edi_stock_enable` (not stored) |
| `l10n_ro_edi_stock_enable_send` | Localization Ro Electronic data interchange Stock Enable Send | boolean |  | computed by rule `_compute_l10n_ro_edi_stock_enable_send` (not stored) |
| `l10n_ro_edi_stock_enable_fetch` | Localization Ro Electronic data interchange Stock Enable Fetch | boolean |  | computed by rule `_compute_l10n_ro_edi_stock_enable_fetch` (not stored) |
| `l10n_ro_edi_stock_enable_amend` | Localization Ro Electronic data interchange Stock Enable Amend | boolean |  | computed by rule `_compute_l10n_ro_edi_stock_enable_amend` (not stored) |
| `l10n_ro_edi_stock_fields_readonly` | Localization Ro Electronic data interchange Stock Fields Readonly | boolean |  | computed by rule `_compute_l10n_ro_edi_stock_fields_readonly` (not stored) |
| `l10n_tr_nilvera_dispatch_type` | Dispatch Type | selection |  | default `SEVK`; changes are tracked in the message thread; not copied on duplication; Help: Used to populate the type of dispatch. |
| `l10n_tr_nilvera_carrier_id` | Carrier (TR) | many to one | `res.partner` | not copied on duplication; Help: Used when the dispatch is made through a third-party carrier company. Populating this makes the Vehicle Plate and Drivers optional. |
| `l10n_tr_nilvera_buyer_id` | Buyer | many to one | `res.partner` | not copied on duplication; Help: Used for the original party who purchases the good when the Delivery Address is for another recipient |
| `l10n_tr_nilvera_seller_supplier_id` | Seller Supplier | many to one | `res.partner` | not copied on duplication; Help: Used for the information of the supplier of the goods in the delivery note. |
| `l10n_tr_nilvera_buyer_originator_id` | Buyer Originator | many to one | `res.partner` | not copied on duplication; Help: Used for the original initiator of the goods acquisition and requesting process. |
| `l10n_tr_nilvera_delivery_printed_number` | Printed Delivery Note Number | single line text |  | not copied on duplication |
| `l10n_tr_nilvera_delivery_date` | Printed Delivery Note Date | date |  | not copied on duplication |
| `l10n_tr_vehicle_plate` | Vehicle Plate | many to one | `l10n_tr.nilvera.trailer.plate` | not copied on duplication; restricted by domain `[('plate_number_type', '=', 'vehicle')]`; Help: Used to input the plate number of the truck. |
| `l10n_tr_nilvera_trailer_plate_ids` | Trailer Plates | many to many | `l10n_tr.nilvera.trailer.plate` | not copied on duplication; restricted by domain `[('plate_number_type', '=', 'trailer')]`; association table `l10n_tr_nilvera_delivery_vehicle_rel`; Help: Used to input the plate numbers of the trailers attached to the truck. |
| `l10n_tr_nilvera_driver_ids` | Drivers | many to many | `res.partner` | not copied on duplication; Help: Used for the individuals driving the truck. |
| `l10n_tr_nilvera_delivery_notes` | Delivery Notes | single line text |  | not copied on duplication |
| `l10n_tr_nilvera_dispatch_state` | e-Dispatch State | selection |  | changes are tracked in the message thread; not copied on duplication |
| `l10n_tr_nilvera_edispatch_warnings` | Localization Tr Nilvera Edispatch Warnings | structured document |  | computed by rule `_compute_edispatch_warnings` (not stored) |
| `has_kits` | Has Kits | boolean |  | computed by rule `_compute_has_kits` (not stored) |
| `production_count` | Count of manufacturing order generated | integer |  | computed by rule `_compute_mrp_production_ids` (not stored); visible only to groups `mrp.group_mrp_user` |
| `production_ids` | Manufacturing Orders | one to many | `mrp.production` | computed by rule `_compute_production_ids` (not stored); visible only to groups `mrp.group_mrp_user` |
| `production_group_id` | Production Group | many to one | `mrp.production.group` | related through path `move_ids.production_group_id` |
| `show_subcontracting_details_visible` | Show Subcontracting Details Visible | boolean |  | computed by rule `_compute_show_subcontracting_details_visible` (not stored) |
| `is_dropship` | Is a Dropship | boolean |  | computed by rule `_compute_is_dropship` (not stored) |
| `subcontracting_source_purchase_count` | Number of subcontracting purchase order Source | integer |  | computed by rule `_compute_subcontracting_source_purchase_count` (not stored); Help: Number of subcontracting Purchase Order Source |
| `project_id` | Project | many to one | `project.project` | restricted by domain `[["is_template", "=", false]]` |
| `zip` | Zip | single line text |  | related through path `partner_id.zip`; searchable through a search rule |
| `website_id` | Website | many to one | `website` | read only; related through path `sale_id.website_id` and stored; Help: Website where this order has been placed, for eCommerce orders. |

## Selection values

### `move_type` (Shipping Policy)

| Value | Label |
|---|---|
| `direct` | As soon as possible |
| `one` | When all products are ready |

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `waiting` | Waiting Another Operation |
| `confirmed` | Waiting |
| `assigned` | Ready |
| `done` | Done |
| `cancel` | Cancelled |

### `products_availability_state` (Products Availability State)

| Value | Label |
|---|---|
| `available` | Available |
| `expected` | Expected |
| `late` | Late |

### `search_date_category` (Date Category)

| Value | Label |
|---|---|
| `before` | Before |
| `yesterday` | Yesterday |
| `today` | Today |
| `day_1` | Tomorrow |
| `day_2` | The day after tomorrow |
| `after` | After |

### `l10n_it_transport_reason` (Transport Reason)

| Value | Label |
|---|---|
| `sale` | Sale |
| `outsourcing` | Outsourcing |
| `evaluation` | Evaluation |
| `gift` | Gift |
| `transfer` | Transfer |
| `substitution` | Returned goods |
| `attemped_sale` | Attempted Sale |
| `loaned_use` | Loaned for Use |
| `repair` | Repair |

### `l10n_it_transport_method` (Transport Method)

| Value | Label |
|---|---|
| `sender` | Sender |
| `recipient` | Recipient |
| `courier` | Courier service |

### `l10n_tr_nilvera_dispatch_type` (Dispatch Type)

| Value | Label |
|---|---|
| `SEVK` | Online |
| `MATBUDAN` | Pre-printed |

### `l10n_tr_nilvera_dispatch_state` (e-Dispatch State)

| Value | Label |
|---|---|
| `to_send` | To Send |
| `sent` | Sent |

## State fields

State machine fields of this entity: `state`, `products_availability_state`, `l10n_ro_edi_stock_state`, `l10n_tr_nilvera_dispatch_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name, company_id)` | Reference must be unique per company! | `stock` |

## Operations (226)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_picking_type_id` | preparation rule | self | `stock` |  |  |
| `_compute_has_tracking` | computation | self | `stock` |  |  |
| `_compute_move_type` | computation | self | `sale_stock`, `stock` | depends: `picking_type_id`; depends: `move_ids.sale_line_id` |  |
| `_compute_has_deadline_issue` | computation | self | `stock` | depends: `date_deadline`, `scheduled_date` |  |
| `_search_date_category` | search rule | self, operator, value | `stock` |  |  |
| `_compute_delay_alert_date` | computation | self | `stock` | depends: `move_ids.delay_alert_date` |  |
| `_compute_is_signed` | computation | self | `stock` | depends: `signature` |  |
| `_compute_is_date_editable` | computation | self | `stock_account`, `stock` |  |  |
| `_compute_products_availability` | computation | self | `stock` | depends: `state`, `picking_type_code`, `scheduled_date`, `move_ids`, `move_ids.forecast_availability`, `move_ids.forecast_expected_date` |  |
| `_compute_show_lots_text` | computation | self | `mrp_subcontracting`, `stock` | depends: `move_line_ids`, `picking_type_id.use_create_lots`, `picking_type_id.use_existing_lots`, `state`; depends: `move_ids.is_subcontract`, `move_ids.has_tracking` |  |
| `_compute_json_popover` | computation | self | `stock` |  |  |
| `_compute_state` | computation | self | `stock` | depends: `move_type`, `move_ids.state`, `move_ids.picking_id` | State of a picking depends on the state of its related stock.move - Draft: only used for "planned pickings" - Waiting: if the picking is not ready to be sent so if   - (a) no quantity could be reserved at all or if   - (b) some quantities could be reserved and the shipping policy is "deliver all at once" - Waiting another move: if the picking is waiting for another move - Ready: if the picking is ready to be sent so if:   - (a) all quantities are reserved or if   - (b) some quantities could be reserved and the shipping policy is "as soon as possible"   - (c) it's an incoming picking - Done: if |
| `_compute_scheduled_date` | computation | self | `stock` | depends: `move_ids.state`, `move_ids.date`, `move_type` |  |
| `_compute_bulk_weight` | computation | self | `stock` | depends: `move_line_ids`, `move_line_ids.result_package_id`, `move_line_ids.product_uom_id`, `move_line_ids.quantity` |  |
| `_compute_shipping_weight` | computation | self | `stock` | depends: `move_line_ids.result_package_id`, `move_line_ids.result_package_id.package_type_id`, `move_line_ids.result_package_id.shipping_weight`, `move_line_ids.result_package_id.outermost_package_id`, `move_line_ids.result_package_id.outermost_package_id.package_type_id`, `move_line_ids.result_package_id.outermost_package_id.shipping_weight`, `weight_bulk` |  |
| `_compute_shipping_volume` | computation | self | `stock` |  |  |
| `_compute_date_deadline` | computation | self | `stock` | depends: `move_ids.date_deadline`, `move_ids.state`, `move_type` |  |
| `_set_scheduled_date` | internal rule | self | `stock` |  |  |
| `_has_scrap_move` | internal rule | self | `stock` |  |  |
| `_compute_packages_count` | computation | self | `stock` |  |  |
| `_compute_show_check_availability` | computation | self | `stock` | depends: `state`, `move_ids.product_uom_qty`, `picking_type_code` | According to `picking.show_check_availability`, the "check availability" button will be displayed in the form view of a picking. |
| `_compute_show_allocation` | computation | self | `stock` | depends: `state`, `move_ids`, `picking_type_id` |  |
| `_compute_location_id` | computation | self | `mrp_subcontracting`, `stock` | depends: `picking_type_id`, `partner_id` |  |
| `_compute_return_count` | computation | self | `stock` | depends: `return_ids` |  |
| `_compute_picking_warning_text` | computation | self | `stock` | depends: `partner_id.name`, `partner_id.parent_id.name` |  |
| `_get_next_transfers` | preparation rule | self | `stock` |  |  |
| `_compute_show_next_pickings` | computation | self | `stock` | depends: `move_ids.move_dest_ids` |  |
| `_search_products_availability_state` | search rule | self, operator, value | `stock` |  |  |
| `_get_show_allocation` | preparation rule | self, picking_type_id | `stock` |  | Helper method for computing "show_allocation" value. Separated out from _compute function so it can be reused in other models (e.g. batch). |
| `get_empty_list_help` | operation | self, help_message | `stock` | model |  |
| `_search_delay_alert_date` | search rule | self, operator, value | `stock` | model |  |
| `_onchange_picking_type` | on change | self | `stock` | onchange: `picking_type_id`, `partner_id` |  |
| `_onchange_location_id` | on change | self | `stock` | onchange: `location_id` |  |
| `create` | lifecycle override | self, vals_list | `stock_picking_batch`, `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock_fleet`, `stock_picking_batch`, `stock` |  |  |
| `unlink` | lifecycle override | self | `stock` |  |  |
| `do_print_picking` | user action | self | `stock` |  |  |
| `should_print_delivery_address` | operation | self | `stock` |  |  |
| `_is_to_external_location` | internal rule | self | `stock_dropshipping`, `stock` |  |  |
| `action_confirm` | user action | self | `stock_picking_batch`, `stock` |  |  |
| `action_assign` | user action | self | `stock` |  | Check availability of picking moves. This has the effect of changing the state and reserve quants on available moves, and may also impact the state of the picking as it is computed based on move's states. @return: True |
| `action_cancel` | user action | self | `stock_picking_batch`, `stock` |  |  |
| `action_detailed_operations` | user action | self | `mrp`, `stock` |  |  |
| `action_next_transfer` | user action | self | `stock` |  |  |
| `_action_done` | internal rule | self | `l10n_it_stock_ddt`, `mrp_subcontracting`, `purchase_stock`, `sale_stock`, `stock` |  | Call `_action_done` on the `stock.move` of the `stock.picking` in `self`. This method makes sure every `stock.move.line` is linked to a `stock.move` by either linking them to an existing one or a newly created one.  If the context key `cancel_backorder` is present, backorders won't be created.  :return: True :rtype: bool |
| `_intercompany_unpack` | internal rule | self | `stock` |  |  |
| `_send_confirmation_email` | internal rule | self | `point_of_sale`, `stock_delivery`, `stock_sms`, `stock` |  |  |
| `_check_move_lines_map_quant_package` | validation | self, package | `stock` |  |  |
| `_get_entire_pack_location_dest` | preparation rule | self, move_line_ids | `stock` |  |  |
| `_is_single_transfer` | internal rule | self | `stock_picking_batch`, `stock` |  |  |
| `_check_entire_pack` | validation | self | `stock` |  | This function check if entire packs are moved in the picking |
| `_get_lot_move_lines_for_sanity_check` | preparation rule | self, none_done_picking_ids, separate_pickings | `stock` |  | Get all move_lines with tracked products that need to be checked over in the sanity check. :param none_done_picking_ids: Set of all pickings ids that have no quantity set on any move_line. :param separate_pickings: Indicates if pickings should be checked independently for lot/serial numbers or not. |
| `_sanity_check` | internal rule | self, separate_pickings | `stock` |  | Sanity check for `button_validate()` :param separate_pickings: Indicates if pickings should be checked independently for lot/serial numbers or not. |
| `do_unreserve` | user action | self | `stock` |  |  |
| `button_validate` | user action | self | `l10n_ro_edi_stock`, `l10n_tr_nilvera_edispatch`, `sale_project_stock`, `stock_delivery`, `stock_picking_batch`, `stock` |  |  |
| `action_split_transfer` | user action | self | `stock` |  |  |
| `_pre_action_done_hook` | internal rule | self | `product_expiry`, `stock_sms`, `stock` |  |  |
| `_should_show_transfers` | internal rule | self | `stock_picking_batch`, `stock` |  | Whether the different transfers should be displayed on the pre action done wizards. |
| `_should_ignore_backorders` | internal rule | self | `stock` |  | Checks if the `create_backorder` setting from the picking type should be ignored. |
| `_get_without_quantities_error_message` | preparation rule | self | `stock` |  | Returns the error message raised in validation if no quantities are reserved. The purpose of this method is to be overridden in case we want to adapt this message.  :return: Translated error message :rtype: str |
| `_action_generate_backorder_wizard` | internal rule | self, show_transfers | `stock` |  |  |
| `action_toggle_is_locked` | user action | self | `stock` |  |  |
| `_check_backorder` | validation | self | `stock` |  |  |
| `_autoconfirm_picking` | internal rule | self | `stock` |  | Automatically run `action_confirm` on `self` if one of the picking's move was added after the initial call to `action_confirm`. Note that `action_confirm` will only work on draft moves. |
| `_get_moves_to_backorder` | preparation rule | self | `stock` |  |  |
| `_create_backorder_picking` | internal rule | self | `stock` |  |  |
| `_create_backorder` | internal rule | self, backorder_moves | `stock_picking_batch`, `stock` |  | This method is called when the user chose to create a backorder. It will create a new picking, the backorder, and move the stock.moves that are not `done` or `cancel` into it. |
| `_log_activity_get_documents` | internal rule | self, orig_obj_changes, stream_field, stream, groupby_method | `stock` |  | Generic method to log activity. To use with _log_activity method. It either log on uppermost ongoing documents or following documents. This method find all the documents and responsible for which a note has to be log. It also generate a rendering_context in order to render a specific note by documents containing only the information relative to the document it. For example we don't want to notify a picking on move that it doesn't contain.  :param dict orig_obj_changes: contain a record as key and the     change on this record as value.     eg: {'move_id': (new product_uom_qty, old product_uom_ |
| `_log_activity` | internal rule | self, render_method, documents | `stock` |  | Log a note for each documents, responsible pair in documents passed as argument. The render_method is then call in order to use a template and render it with a rendering_context.  :param dict documents: A tuple (document, responsible) as key.     An activity will be log by key. A rendering_context as value.     If used with _log_activity_get_documents. In 'DOWN' stream     cases the rendering_context will be a dict with format:     {'stream_object': ('orig_object', new_qty, old_qty)}     'UP' stream will add all the documents browsed in order to     get the final/upstream document present in t |
| `_log_less_quantities_than_expected` | internal rule | self, moves | `sale_stock`, `stock` |  | Log an activity on picking that follow moves. The note contains the moves changes and all the impacted picking.  :param dict moves: a dict with a move as key and tuple with new and old quantity as value. eg: {move_1 : (4, 5)} |
| `_less_quantities_than_expected_add_documents` | internal rule | self, moves, documents | `mrp`, `stock` |  |  |
| `_get_impacted_pickings` | preparation rule | self, moves | `stock` |  | This function is used in _log_less_quantities_than_expected the purpose is to notify a user with all the pickings that are impacted by an action on a chained move. param: 'moves' contain moves that belong to a common picking. return: all the pickings that contain a destination moves (direct and indirect) from the moves given as arguments. |
| `action_put_in_pack` | user action | self, package_id, package_type_id, package_name | `stock` |  |  |
| `get_action_click_graph` | operation | self | `mrp`, `repair`, `stock` | model |  |
| `_get_action` | preparation rule | self, action_xmlid | `stock` |  |  |
| `get_action_picking_tree_incoming` | operation | self | `stock` | model |  |
| `get_action_picking_tree_outgoing` | operation | self | `stock` | model |  |
| `get_action_picking_tree_internal` | operation | self | `stock` | model |  |
| `calculate_date_category` | operation | self, datetime | `stock` | model | Assigns given datetime to one of the following categories: - "before" - "yesterday" - "today" - "day_1" (tomorrow) - "day_2" (the day after tomorrow) - "after"  The categories are based on current user's timezone (e.g. "today" will last between 00:00 and 23:59 local time). The datetime itself is assumed to be in UTC. If the datetime is falsy, this function returns "". |
| `date_category_to_domain` | operation | self, field_name, date_category | `stock` | model | Given a date category, returns a list of tuples of operator and value that can be used in a domain to filter records based on their scheduled date.  Args:     date_category (str): The date category to use for the computation.         Allowed values are:         * "before"         * "yesterday"         * "today"         * "day_1"         * "day_2"         * "after"  Returns:     a list of tuples:         each tuple consists of an operator and a value that can be used in         a domain to filter records based on their scheduled date.         The operator can be "<" or ">=". The value is a date |
| `button_scrap` | user action | self | `stock` |  |  |
| `action_add_entire_packs` | user action | self, package_ids | `stock` |  |  |
| `action_see_move_scrap` | user action | self | `stock` |  |  |
| `action_see_packages` | user action | self | `stock` |  |  |
| `action_see_package_histories` | user action | self | `stock` |  |  |
| `action_picking_move_tree` | user action | self | `stock` |  |  |
| `action_view_reception_report` | user action | self | `stock` |  |  |
| `action_open_label_layout` | user action | self | `stock` |  |  |
| `action_open_label_type` | user action | self | `stock` |  |  |
| `_attach_sign` | internal rule | self | `stock` |  | Render the delivery report in pdf and attach it to the picking in `self`. |
| `action_see_returns` | user action | self | `stock` |  |  |
| `_get_report_lang` | preparation rule | self | `stock` |  |  |
| `_get_autoprint_report_actions` | preparation rule | self | `stock` |  |  |
| `_get_packages_for_print` | preparation rule | self | `stock` |  |  |
| `_can_return` | internal rule | self | `sale_stock`, `stock` |  |  |
| `_add_reference` | internal rule | self, reference | `stock` |  | link the given references to the list of references. |
| `_remove_reference` | internal rule | self, reference | `stock` |  | remove the given references from the list of references. |
| `_prepare_entire_pack_move_line_vals` | preparation rule | self, packages | `stock` |  | Prepares the move line values for every packages within packages and their children that contain products. |
| `_check_backdate_allowed` | validation | self | `stock_account` | constrains: `date_done` |  |
| `_is_date_in_lock_period` | internal rule | self | `stock_account` |  |  |
| `_compute_sale_id` | computation | self | `sale_stock` | depends: `reference_ids.sale_ids`, `move_ids.sale_line_id.order_id` |  |
| `_set_sale_id` | internal rule | self | `sale_stock` |  |  |
| `_auto_init` | lifecycle override | self | `sale_stock` |  | Create related field here, too slow when computing it afterwards through _compute_related.  Since group_id.sale_id is created in this module, no need for an UPDATE statement. |
| `_get_default_weight_uom` | preparation rule | self | `stock_delivery` |  |  |
| `_compute_weight_uom_name` | computation | self | `stock_delivery` |  |  |
| `_compute_allowed_carrier_ids` | computation | self | `stock_delivery` | depends: `partner_id`, `carrier_id.max_weight`, `carrier_id.max_volume`, `carrier_id.must_have_tag_ids`, `carrier_id.excluded_tag_ids`, `move_ids.product_id.product_tag_ids`, `move_ids.product_id.weight`, `move_ids.product_id.volume` |  |
| `_compute_carrier_tracking_url` | computation | self | `stock_delivery` | depends: `carrier_id`, `carrier_tracking_ref` |  |
| `_compute_return_picking` | computation | self | `stock_delivery` | depends: `carrier_id`, `move_ids` |  |
| `_compute_return_label` | computation | self | `stock_delivery` |  |  |
| `get_multiple_carrier_tracking` | operation | self | `stock_delivery` |  |  |
| `_cal_weight` | computation | self | `stock_delivery` | depends: `move_ids.weight` |  |
| `_carrier_exception_note` | internal rule | self, exception | `stock_delivery` |  |  |
| `send_to_shipper` | operation | self | `stock_delivery` |  |  |
| `_check_carrier_details_compliance` | validation | self | `stock_delivery` |  | Hook to check if a delivery is compliant in regard of the carrier. |
| `print_return_label` | operation | self | `stock_delivery` |  |  |
| `_get_matching_delivery_lines` | preparation rule | self | `stock_delivery` |  |  |
| `_prepare_sale_delivery_line_vals` | preparation rule | self | `stock_delivery` |  |  |
| `_add_delivery_cost_to_so` | internal rule | self | `stock_delivery` |  |  |
| `open_website_url` | operation | self | `stock_delivery` |  |  |
| `cancel_shipment` | operation | self | `stock_delivery` |  |  |
| `_get_estimated_weight` | preparation rule | self | `stock_delivery` |  |  |
| `_should_generate_commercial_invoice` | internal rule | self | `l10n_in_stock`, `stock_delivery` |  |  |
| `action_add_operations` | user action | self | `stock_picking_batch` |  |  |
| `_find_auto_batch` | internal rule | self | `stock_picking_batch` |  |  |
| `_is_auto_batchable` | internal rule | self, picking | `delivery_stock_picking_batch`, `stock_picking_batch` |  | Verifies if a picking can be put in a batch with another picking without violating auto_batch constrains. |
| `_get_possible_pickings_domain` | preparation rule | self | `delivery_stock_picking_batch`, `stock_picking_batch` |  |  |
| `_get_possible_batches_domain` | preparation rule | self | `delivery_stock_picking_batch`, `stock_picking_batch` |  |  |
| `_get_auto_batch_description` | preparation rule | self | `delivery_stock_picking_batch`, `stock_picking_batch` |  | Get the description of the automatically created batch based on the grouped pickings and grouping criteria |
| `_add_to_wave_post_picking_split_hook` | internal rule | self | `stock_picking_batch` |  |  |
| `assign_batch_user` | operation | self, user_id | `stock_picking_batch` |  |  |
| `action_view_batch` | user action | self | `stock_picking_batch` |  |  |
| `_prepare_picking_vals` | preparation rule | self, partner, picking_type, location_id, location_dest_id | `point_of_sale` |  |  |
| `_create_picking_from_pos_order_lines` | internal rule | self, location_dest_id, lines, picking_type, partner | `point_of_sale` | model | We'll create some picking based on order_lines |
| `_prepare_stock_move_vals` | preparation rule | self, first_line, order_lines | `point_of_sale` |  |  |
| `_create_move_from_pos_order_lines` | internal rule | self, lines | `point_of_sale`, `pos_repair`, `pos_sale` |  |  |
| `_link_owner_on_return_picking` | internal rule | self, lines | `point_of_sale` |  | This method tries to retrieve the owner of the returned product |
| `_compute_l10n_ar_delivery_guide_flags` | computation | self | `l10n_ar_stock` | depends: `state`, `l10n_ar_delivery_guide_number`, `picking_type_id.l10n_ar_document_type_id` | Compute flags for allowing delivery guide generation and sending. - Generation allowed if: state is 'done', document type exists, and no guide number. - Send allowed if: guide number exists. |
| `l10n_ar_action_create_delivery_guide` | operation | self | `l10n_ar_stock` |  | Create the delivery guide number and store CAI data for the stock picking. |
| `l10n_ar_action_send_delivery_guide` | operation | self | `l10n_ar_stock` |  | Send the delivery guide to the partner. |
| `_compute_nbr_repairs` | computation | self | `repair` | depends: `repair_ids` |  |
| `action_repair_return` | user action | self | `repair` |  |  |
| `action_view_repairs` | user action | self | `repair` |  |  |
| `_get_l10n_in_dropship_dest_partner` | preparation rule | self | `l10n_in_purchase_stock`, `l10n_in_stock` |  | To be overriden by `l10n_in_purchase_stock` will be ideal to use it for `l10n_in_ewaybill_stock` returns destination partner from purchase_id |
| `_l10n_in_get_invoice_partner` | internal rule | self | `l10n_in_sale_stock`, `l10n_in_stock` |  | To be overriden by `l10n_in_sale_stock` will be ideal to use it for `l10n_in_ewaybill_stock` returns invoice partner from sale_id |
| `_l10n_in_get_fiscal_position` | internal rule | self | `l10n_in_purchase_stock`, `l10n_in_sale_stock`, `l10n_in_stock` |  | To be inherited by `l10n_in_*_stock` will be ideal to use it for `l10n_in_ewaybill_stock` returns fiscal position from order |
| `_get_l10n_in_ewaybill_form_action` | preparation rule | self | `l10n_in_ewaybill_stock` |  |  |
| `action_l10n_in_ewaybill_create` | user action | self | `l10n_in_ewaybill_stock` |  |  |
| `action_open_l10n_in_ewaybill` | user action | self | `l10n_in_ewaybill_stock` |  |  |
| `_compute_l10n_in_ewaybill_details` | computation | self | `l10n_in_ewaybill_stock` | depends: `l10n_in_ewaybill_ids.state` |  |
| `_compute_effective_date` | computation | self | `purchase_stock` | depends: `state`, `location_dest_id.usage`, `date_done` |  |
| `_compute_date_order` | computation | self | `purchase_stock` |  |  |
| `_search_days_to_arrive` | search rule | self, operator, value | `purchase_stock` | model |  |
| `_search_delay_pass` | search rule | self, operator, value | `purchase_stock` | model |  |
| `_compute_l10n_it_show_print_ddt_button` | computation | self | `l10n_it_stock_ddt` | depends: `country_code`, `picking_type_code`, `state`, `is_locked`, `move_ids`, `location_id`, `location_dest_id` |  |
| `_l10n_ro_edi_stock_reset_variable_selection_fields` | on change | self | `l10n_ro_edi_stock` | onchange: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_default_location_type` | computation | self | `l10n_ro_edi_stock` | depends: `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_available_operation_scopes` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_available_location_types` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_current_document_state` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_current_document_uit` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_enable` | computation | self | `l10n_ro_edi_stock_batch`, `l10n_ro_edi_stock` | depends: `company_id.account_fiscal_country_id.code`; depends: `batch_id`, `company_id`, `picking_type_code` |  |
| `_compute_l10n_ro_edi_stock_enable_send` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_enable_fetch` | computation | self | `l10n_ro_edi_stock` | depends: `company_id`, `state`, `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_enable_amend` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_fields_readonly` | computation | self | `l10n_ro_edi_stock` | depends: `l10n_ro_edi_stock_state` |  |
| `_l10n_ro_edi_stock_validate_carrier` | internal rule | self | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_validate_carrier_filter` | internal rule | self, picking | `l10n_ro_edi_stock_batch`, `l10n_ro_edi_stock` | model |  |
| `_l10n_ro_edi_stock_validate_data` | internal rule | self, data | `l10n_ro_edi_stock` | model |  |
| `_l10n_ro_edi_stock_validate_fetch_data` | internal rule | self, errors | `l10n_ro_edi_stock` |  |  |
| `action_l10n_ro_edi_stock_send_etransport` | user action | self | `l10n_ro_edi_stock` |  |  |
| `action_l10n_ro_edi_stock_fetch_status` | user action | self | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_get_current_document` | internal rule | self | `l10n_ro_edi_stock` |  | Returns the most recently created document in l10n_ro_edi_stock_document_ids |
| `_l10n_ro_edi_stock_get_all_documents` | internal rule | self, states | `l10n_ro_edi_stock` |  | Returns filtered documents by state |
| `_l10n_ro_edi_stock_get_last_document` | internal rule | self, state | `l10n_ro_edi_stock` |  | Returns the most recently created document with the given state |
| `_l10n_ro_edi_stock_create_document_stock_sent` | internal rule | self, values | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_create_document_stock_sending_failed` | internal rule | self, values | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_create_document_stock_validated` | internal rule | self, values | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_send_etransport_document` | internal rule | self, send_type | `l10n_ro_edi_stock` |  | Send the eTransport document to anaf :param send_type: 'send' (initial sending of document) \| 'amend' (correct the already sent document) |
| `_l10n_ro_edi_stock_fetch_document_status` | internal rule | self | `l10n_ro_edi_stock` |  |  |
| `_l10n_ro_edi_stock_get_template_data` | internal rule | self, data | `l10n_ro_edi_stock` | model | Returns the data necessary to render the eTransport template |
| `_l10n_ro_edi_stock_get_available_location_types` | internal rule | self, operation_type, location | `l10n_ro_edi_stock` | model | :return comma separated list of available location types for the start or end location based on the operation type |
| `_l10n_ro_edi_stock_get_cod` | internal rule | self, record | `l10n_ro_edi_stock` | model | :return the records vat in the format required by anaf |
| `_l10n_ro_edi_stock_get_gross_weight` | internal rule | self, move | `l10n_ro_edi_stock` | model | :return the gross weight of a stock.move |
| `_l10n_ro_edi_stock_report_unhandled_document_state` | internal rule | self, state | `l10n_ro_edi_stock` |  | Reports an unknown document state from anaf to the user in the chatter |
| `_compute_edispatch_warnings` | computation | self | `l10n_tr_nilvera_edispatch` | depends: `l10n_tr_nilvera_carrier_id`, `l10n_tr_nilvera_buyer_id`, `l10n_tr_nilvera_seller_supplier_id`, `l10n_tr_nilvera_buyer_originator_id`, `l10n_tr_nilvera_delivery_printed_number`, `l10n_tr_nilvera_delivery_date`, `l10n_tr_vehicle_plate`, `l10n_tr_nilvera_trailer_plate_ids`, `l10n_tr_nilvera_driver_ids`, `partner_id` |  |
| `_l10n_tr_validate_edispatch_on_done` | internal rule | self | `l10n_tr_nilvera_edispatch` |  |  |
| `_l10n_tr_validate_edispatch_fields` | internal rule | self | `l10n_tr_nilvera_edispatch` |  |  |
| `_l10n_tr_generate_edispatch_xml` | internal rule | self | `l10n_tr_nilvera_edispatch` |  |  |
| `action_generate_l10n_tr_edispatch_xml` | user action | self, is_list | `l10n_tr_nilvera_edispatch` |  |  |
| `action_mark_l10n_tr_edispatch_status` | user action | self | `l10n_tr_nilvera_edispatch` |  |  |
| `_get_tag_text` | preparation rule | self, xpath, tree, default | `l10n_tr_nilvera_edispatch` |  |  |
| `_get_partner_vals_from_xml` | preparation rule | self, tree, xpath | `l10n_tr_nilvera_edispatch` |  |  |
| `_create_partner_from_xml` | internal rule | self, partner_vals | `l10n_tr_nilvera_edispatch` |  |  |
| `_find_or_create_products_from_xml` | internal rule | self, receipt_lines | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_receipt_lines` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_vehicle_plate` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_trailer_plate_ids` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_drivers` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_matbudan_data` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_partners` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_import_edispatch_fields` | internal rule | self, tree | `l10n_tr_nilvera_edispatch` |  |  |
| `_update_data_from_xml` | internal rule | self, file_data | `l10n_tr_nilvera_edispatch` |  |  |
| `_l10n_tr_create_receipts_from_attachment` | internal rule | self, attachments | `l10n_tr_nilvera_edispatch` |  |  |
| `l10n_tr_import_ereceipts` | operation | self, attachment_ids | `l10n_tr_nilvera_edispatch` |  |  |
| `_compute_has_kits` | computation | self | `mrp` | depends: `move_ids` |  |
| `_compute_production_ids` | computation | self | `mrp` | depends: `reference_ids.production_ids` |  |
| `_compute_mrp_production_ids` | computation | self | `mrp` | depends: `production_ids` |  |
| `action_view_mrp_production` | user action | self | `mrp` |  |  |
| `_check_expired_lots` | validation | self | `product_expiry` |  |  |
| `_action_generate_expired_wizard` | internal rule | self | `product_expiry` |  |  |
| `_compute_show_subcontracting_details_visible` | computation | self | `mrp_subcontracting` | depends: `move_ids.show_subcontracting_details_visible` |  |
| `action_show_subcontract_details` | user action | self | `mrp_subcontracting` |  |  |
| `_is_subcontract` | internal rule | self | `mrp_subcontracting` |  |  |
| `_get_subcontract_production` | preparation rule | self | `mrp_subcontracting` |  |  |
| `_get_warehouse` | preparation rule | self, subcontract_move | `mrp_subcontracting_dropshipping`, `mrp_subcontracting` |  |  |
| `_prepare_subcontract_mo_vals` | preparation rule | self, subcontract_move, bom | `mrp_subcontracting_dropshipping`, `mrp_subcontracting` |  |  |
| `_get_subcontract_mo_confirmation_ctx` | preparation rule | self | `mrp_subcontracting_purchase`, `mrp_subcontracting` |  |  |
| `_subcontracted_produce` | internal rule | self, subcontract_details | `mrp_subcontracting` |  |  |
| `_compute_is_dropship` | computation | self | `mrp_subcontracting_dropshipping`, `stock_dropshipping` | depends: `location_dest_id.usage`, `location_dest_id.company_id`, `location_id.usage`, `location_id.company_id` |  |
| `_compute_subcontracting_source_purchase_count` | computation | self | `mrp_subcontracting_purchase` | depends: `move_ids.move_dest_ids.raw_material_production_id` |  |
| `action_view_subcontracting_source_purchase` | user action | self | `mrp_subcontracting_purchase` |  |  |
| `_get_subcontracting_source_purchase` | preparation rule | self | `mrp_subcontracting_purchase` |  |  |
| `_search_zip` | search rule | self, operator, value | `stock_fleet` |  |  |
| `_reset_location` | internal rule | self | `stock_fleet` |  |  |
| `_check_warn_sms` | validation | self | `stock_sms` |  |  |
| `_action_generate_warn_sms_wizard` | internal rule | self | `stock_sms` |  |  |

## Validation and error messages (22)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_set_scheduled_date` | UserError | You cannot change the Scheduled Date on a cancelled transfer. | `stock` |
| `write` | UserError | Changing the operation type of this record is forbidden at this point. | `stock` |
| `action_assign` | UserError | Nothing to check the availability for. | `stock` |
| `_sanity_check` | UserError | You can’t validate an empty transfer. Please add some products to move before proceeding. | `stock` |
| `_sanity_check` | UserError | self._get_without_quantities_error_message() | `stock` |
| `_sanity_check` | UserError | You need to supply a Lot/Serial number for products %s. | `stock` |
| `_sanity_check` | UserError | message.lstrip() | `stock` |
| `action_split_transfer` | UserError | %s: Nothing to split. Fill the quantities you want in a new transfer in the done quantities | `stock` |
| `action_split_transfer` | UserError | %s: Nothing to split, all demand is done. For split you need at least one line not fully fulfilled | `stock` |
| `action_split_transfer` | UserError | %s: Can't split: quantities done can't be above demand | `stock` |
| `_check_backdate_allowed` | ValidationError | You cannot modify the scheduled date of operation %s because it falls within a locked fiscal period. | `stock_account` |
| `open_website_url` | UserError | Your delivery method has no redirect on courier provider's website to track this order. | `stock_delivery` |
| `l10n_ar_action_create_delivery_guide` | UserError | The delivery guide number %s exceeds the range specified in the CAI. Please update the range or use a different CAI with a different range. | `l10n_ar_stock` |
| `l10n_ar_action_send_delivery_guide` | UserError | The partner does not have an email address. | `l10n_ar_stock` |
| `action_l10n_in_ewaybill_create` | UserError | Please set HSN code in below products:  %s | `l10n_in_ewaybill_stock` |
| `action_l10n_in_ewaybill_create` | UserError | Ewaybill already created for this picking. | `l10n_in_ewaybill_stock` |
| `_l10n_ro_edi_stock_validate_carrier` | UserError | The picking %(picking_name)s is missing a delivery carrier. | `l10n_ro_edi_stock` |
| `_l10n_ro_edi_stock_validate_carrier` | UserError | The delivery carrier of %(picking_name)s is missing the partner field value. | `l10n_ro_edi_stock` |
| `action_generate_l10n_tr_edispatch_xml` | UserError | Error occurred in generating XML for following records: - %s | `l10n_tr_nilvera_edispatch` |
| `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s must be validated before validating the stock picking. | `sale_project_stock` |
| `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s is cancelled. You cannot validate a stock picking on a cancelled Sales Order. | `sale_project_stock` |
| `button_validate` | UserError | The Sales Order %(order)s linked to the Project %(project)s is currently locked. You cannot validate a stock picking on a locked Sales Order. Please create a new SO linked to this Project. | `sale_project_stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_pos_user` | yes | yes | yes | yes | `point_of_sale` |
| `purchase.group_purchase_user` | yes | yes | yes | yes | `purchase_stock` |
| `purchase.group_purchase_manager` | yes | yes | yes | yes | `purchase_stock` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `sale_stock` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale_stock` |
| `base.group_portal` | no | yes | no | no | `sale_stock` |
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `account.group_account_readonly` | no | yes | no | no | `stock_account` |
| `account.group_account_invoice` | yes | yes | yes | no | `stock_account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Stock Pickings Subcontractor | `[(4, ref('base.group_portal'))]` | `[('partner_id.commercial_partner_id', '=', user.partner_id.commercial_partner_id.id)]` | True | True | True | True |
| Portal Follower Transfers | `[(4, ref('base.group_portal'))]` | `['\|', ('partner_id', '=', user.partner_id.id), ('sale_id.partner_id', '=', user.partner_id.id)]` | True | True | True | True |

## Views (41)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ar_stock.stock_picking_form_inherit_l10n_ar_stock` | field | `stock.view_picking_form` | `origin`, `l10n_ar_delivery_guide_number` |  |  | `l10n_ar_stock` |
| `l10n_in_ewaybill_stock.view_picking_form_inherit_ewaybill` | xpath | `stock.view_picking_form` |  | `Create e-Waybill / Challan` |  | `l10n_in_ewaybill_stock` |
| `l10n_in_ewaybill_stock.view_picking_list_inherit_ewaybill` | xpath | `stock.vpicktree` | `l10n_in_ewaybill_name` |  |  | `l10n_in_ewaybill_stock` |
| `l10n_it_stock_ddt.view_picking_form_inherit_l10n_it_ddt` | xpath | `stock.view_picking_form` | `l10n_it_show_print_ddt_button` | `Print` |  | `l10n_it_stock_ddt` |
| `l10n_it_stock_ddt.view_picking_search_inherit_l10n_it_ddt` | field | `stock.view_picking_internal_search` | `origin`, `l10n_it_ddt_number` |  |  | `l10n_it_stock_ddt` |
| `l10n_it_stock_ddt.view_picking_tree_inherit_l10n_it_ddt` | field | `stock.vpicktree` | `origin`, `l10n_it_ddt_number` |  |  | `l10n_it_stock_ddt` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_view_picking_form` | xpath | `stock.view_picking_form` | `l10n_ro_edi_stock_enable_send`, `l10n_ro_edi_stock_enable_fetch`, `l10n_ro_edi_stock_enable_amend` | `Send eTransport`, `Amend eTransport`, `Fetch Status` |  | `l10n_ro_edi_stock` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_stock_picking_view_tree` | xpath | `stock.vpicktree` |  | `Fetch Status` |  | `l10n_ro_edi_stock` |
| `l10n_ro_edi_stock.l10n_ro_edi_stock_stock_picking_filter` | field | `stock.view_picking_internal_search` | `lot_id`, `l10n_ro_edi_stock_state` |  |  | `l10n_ro_edi_stock` |
| `l10n_tr_nilvera_edispatch.vpicktree_inherit_l10n_tr_nilvera_edispatch` | list | `stock.vpicktree` |  |  |  | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.view_picking_internal_search_inherit_l10n_tr_nilvera_edispatch` | xpath | `stock.view_picking_internal_search` |  |  | `GİB e-Dispatch State` | `l10n_tr_nilvera_edispatch` |
| `l10n_tr_nilvera_edispatch.view_picking_form_inherit_l10n_tr_nilvera_edispatch` | xpath | `stock.view_picking_form` |  | `Generate GİB e-Dispatch (XML)` |  | `l10n_tr_nilvera_edispatch` |
| `mrp.view_picking_form_inherit_mrp` | xpath | `stock.view_picking_form` | `production_count` | `action_view_mrp_production` |  | `mrp` |
| `mrp_subcontracting.stock_picking_form_view` | xpath | `stock.view_picking_form` | `show_subcontracting_details_visible` | `Subcontracting Productions` |  | `mrp_subcontracting` |
| `mrp_subcontracting.subcontracting_portal_production_form_view` | form |  | `name`, `state`, `scheduled_date`, `date_deadline`, `origin`, `move_ids`, `id`, `product_id`, `show_details_visible`, `description_picking`, `date`, `date_deadline`, `product_uom_qty`, `product_qty`, `quantity`, `product_uom`, `show_subcontracting_details_visible` | `action_show_details`, `Register components for subcontracted product` |  | `mrp_subcontracting` |
| `mrp_subcontracting_purchase.stock_picking_form_mrp_subcontracting` | xpath | `stock.view_picking_form` | `subcontracting_source_purchase_count` | `action_view_subcontracting_source_purchase` |  | `mrp_subcontracting_purchase` |
| `project_stock.view_picking_form_inherit_project_stock` | xpath | `stock.view_picking_form` | `project_id` |  |  | `project_stock` |
| `purchase_stock.view_picking_form` | xpath | `stock.view_picking_form` |  |  |  | `purchase_stock` |
| `repair.repair_view_picking_form` | xpath | `stock.view_picking_form` | `repair_ids`, `nbr_repairs` | `action_view_repairs` |  | `repair` |
| `sale_stock.view_picking_form` | xpath | `stock.view_picking_form` | `sale_id` |  |  | `sale_stock` |
| `stock.stock_picking_view_activity` | activity |  | `name`, `scheduled_date` |  |  | `stock` |
| `stock.stock_picking_calendar` | calendar |  | `partner_id`, `origin`, `picking_type_id`, `state`, `picking_properties` |  |  | `stock` |
| `stock.stock_picking_kanban` | kanban |  | `scheduled_date`, `picking_type_id`, `priority`, `name`, `state`, `picking_properties`, `partner_id`, `activity_ids`, `json_popover`, `scheduled_date`, `user_id` |  |  | `stock` |
| `stock.vpicktree` | list |  | `company_id`, `priority`, `name`, `location_id`, `location_dest_id`, `partner_id`, `is_signed`, `user_id`, `scheduled_date`, `picking_type_code`, `products_availability_state`, `products_availability`, `date_deadline`, `date_done`, `origin`, `backorder_id`, `picking_type_id`, `company_id`, `state`, `activity_exception_decoration`, `json_popover` | `Unreserve`, `Check Availability` |  | `stock` |
| `stock.view_picking_form` | form |  | `state`, `state`, `picking_warning_text`, `return_count`, `priority`, `picking_type_code`, `name`, `partner_id`, `picking_type_id`, `location_id`, `location_dest_id`, `location_id`, `location_dest_id`, `backorder_id`, `use_create_lots`, `scheduled_date`, `json_popover`, `date_deadline`, `products_availability_state`, `products_availability`, `date_done`, `origin`, `owner_id`, `picking_properties`, `move_ids`, `company_id`, `state`, `picking_type_id`, `location_id`, `location_dest_id`, `partner_id`, `picking_code`, `show_details_visible`, `additional`, `move_lines_count`, `is_locked`, `is_storable`, `has_tracking`, `product_id`, `forecast_availability`, `packaging_uom_qty`, `packaging_uom_id`, `location_final_id`, `description_picking`, `date`, `date_deadline`, `is_quantity_done_editable`, `show_quant`, `show_lots_text`, `show_lots_m2o`, `is_initial_demand_editable`, `display_import_lot`, `has_lines_without_result_package`, `package_ids`, `product_uom_qty`, `forecast_expected_date`, `product_qty`, `quantity`, `product_uom`, `picked` | `Mark as Todo`, `Check Availability`, `Validate`, `Validate`, `Print`, `Print`, `Return`, `Cancel`, `action_see_returns`, `Scraps`, `Packages`, `Packages`, `%(action_stock_report)d`, `action_view_reception_report`, `action_picking_move_tree`, `action_detailed_operations`, `action_next_transfer`, `Details`, `Put in Pack` |  | `stock` |
| `stock.view_picking_internal_search` | search |  | `name`, `partner_id`, `origin`, `product_id`, `picking_type_id`, `move_line_ids`, `lot_id`, `activity_user_id`, `activity_type_id` |  | `To Do`, `My Transfers`, `Starred`, `Draft`, `Waiting`, `Ready`, `Receipts`, `Deliveries`, `Internal`, `Before`, `Late`, `Yesterday`, `Today`, `Tomorrow`, `The day after tomorrow`, `After`, `Late Availability`, `Backorders`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Warnings`, `Status`, `Scheduled Date`, `Source Document`, `Destination Country`, `Operation Type`, `Properties` | `stock` |
| `stock_account.view_picking_form` | xpath | `stock.view_picking_form` |  |  |  | `stock_account` |
| `stock_delivery.view_picking_withcarrier_out_form` | data | `stock.view_picking_form` | `is_return_picking`, `carrier_id`, `integration_level`, `delivery_type`, `carrier_tracking_ref`, `weight`, `weight_uom_name`, `shipping_weight`, `weight_uom_name` | `Cancel`, `Tracking`, `Send to Shipper`, `Print Return Label` |  | `stock_delivery` |
| `stock_delivery.delivery_tracking_url_warning_form` | form |  |  | `OK` |  | `stock_delivery` |
| `stock_delivery.vpicktree_view_tree` | xpath | `stock.vpicktree` | `carrier_tracking_ref`, `carrier_id`, `destination_country_code`, `weight`, `shipping_weight` |  |  | `stock_delivery` |
| `stock_dropshipping.view_picking_internal_search_inherit_stock_dropshipping` | xpath | `stock.view_picking_internal_search` |  |  | `Dropships` | `stock_dropshipping` |
| `stock_fleet.vpicktree` | field | `stock.vpicktree` | `picking_type_id`, `zip`, `shipping_weight`, `shipping_volume` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_tree_inherit_stock_fleet` | field | `stock_picking_batch.stock_picking_view_batch_tree_ref` | `zip` |  |  | `stock_fleet` |
| `stock_picking_batch.view_picking_form_inherited` | xpath | `stock.view_picking_form` | `state` |  |  | `stock_picking_batch` |
| `stock_picking_batch.view_picking_internal_search_inherit_stock_picking_batch` | xpath | `stock.view_picking_internal_search` | `batch_id` |  |  | `stock_picking_batch` |
| `stock_picking_batch.view_picking_internal_search_inherit` | filter | `stock.view_picking_internal_search` |  |  | `picking_type`, `Batch Transfer` | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_form_inherit` | div | `stock.view_picking_form` |  | `Batch` |  | `stock_picking_batch` |
| `stock_picking_batch.vpicktree` | field | `stock.vpicktree` | `picking_type_id`, `batch_id` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_view_batch_tree_ref` | xpath | `stock.vpicktree` |  |  |  | `stock_picking_batch` |
| `website_sale_collect.stock_picking_form` | button | `stock_delivery.view_picking_withcarrier_out_form` |  | `send_to_shipper` |  | `website_sale_collect` |
| `website_sale_stock.view_picking_form_inherit_website_sale_stock` | xpath | `stock.view_picking_form` | `website_id` |  |  | `website_sale_stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mrp_subcontracting.subcontracting_portal_view_production_action` | Subcontracting Portal | form |  |  |  | `mrp_subcontracting` |
| `stock.action_picking_tree_all` | Transfers | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'default_company_id': allowed_company_ids[0]}` |  | `stock` |
| `stock.action_picking_tree_incoming` | Receipts | list,kanban,form,calendar,activity |  | `{'contact_display': 'partner_address', 'restricted_picking_type_code': 'incoming', 'search_default_reception': 1}` |  | `stock` |
| `stock.action_picking_tree_outgoing` | Deliveries | list,kanban,form,calendar,activity |  | `{'contact_display': 'partner_address', 'restricted_picking_type_code': 'outgoing', 'search_default_delivery': 1}` |  | `stock` |
| `stock.action_picking_tree_internal` | Internal Transfers | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'restricted_picking_type_code': 'internal', 'search_default_internal': 1}` |  | `stock` |
| `stock.stock_picking_action_picking_type` | All Transfers |  |  | `{'contact_display': 'partner_address'}` |  | `stock` |
| `stock.action_picking_tree_ready` | To Do | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'search_default_available': 1}` |  | `stock` |
| `stock.action_picking_tree_graph` | To Do | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'search_default_available': 1, 'search_default_waiting': 1}` |  | `stock` |
| `stock.action_picking_tree_waiting` | Waiting Transfers | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'search_default_waiting': 1}` |  | `stock` |
| `stock.action_picking_tree_late` | Late Transfers | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'search_default_late': 1}` |  | `stock` |
| `stock.action_picking_tree_backorder` | Backorders | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'search_default_backorder': 1}` |  | `stock` |
| `stock.action_picking_form` | New Transfer | form |  | `{                     'search_default_picking_type_id': [active_id],                     'default_picking_type_id': active_id,                     'contact_display': 'partner_address',             }` |  | `stock` |
| `stock_delivery.act_delivery_trackers_url` | Display tracking links | form |  |  | new | `stock_delivery` |
| `stock_dropshipping.action_picking_tree_dropship` | Dropships | list,kanban,form,calendar |  | `{'contact_display': 'partner_address', 'default_company_id': allowed_company_ids[0], 'restricted_picking_type_code': 'dropship', 'search_default_dropships': 1}` |  | `stock_dropshipping` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `l10n_tr_nilvera_edispatch.action_export_l10n_tr_nilvera_edispatch_list` | Generate GİB e-Dispatch (XML) | code |  | yes |
| `l10n_tr_nilvera_edispatch.action_mark_l10n_tr_nilvera_edispatch_status` | Mark as sent (GİB e-Dispatch) | code |  | yes |
| `repair.action_create_repair_order` | Create Repair | code |  | yes |
| `stock.action_validate_picking` | Validate | code |  | yes |
| `stock.action_unreserve_picking` | Unreserve | code |  | yes |
| `stock.action_print_labels` | Labels | code | report | yes |
| `stock.action_toggle_is_locked` | Lock/Unlock | code |  | yes |
| `stock.action_scrap` | Scrap | code |  | yes |
| `stock.click_dashboard_graph` | stock.click_dashboard_graph | code |  | yes |
| `stock.method_action_picking_tree_incoming` | stock.method_action_picking_tree_incoming | code |  | yes |
| `stock.method_action_picking_tree_outgoing` | stock.method_action_picking_tree_outgoing | code |  | yes |
| `stock.method_action_picking_tree_internal` | stock.method_action_picking_tree_internal | code |  | yes |
| `stock.stock_split_picking` | Split | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `l10n_ar_stock.action_delivery_guide_report_pdf` | Delivery Guide (AR) | qweb-pdf | `l10n_ar_stock.report_delivery_guide` | `'Remito - %s' % (object.l10n_ar_delivery_guide_number or 's/n')` | `'Remito - %s.pdf' % (object.l10n_ar_delivery_guide_number or 's/n')` |
| `l10n_it_stock_ddt.action_report_ddt` | DDT report | qweb-pdf | `l10n_it_stock_ddt.report_ddt` | `'DDT - %s - %s' % (object.partner_id.name or '', object.l10n_it_ddt_number)` |  |
| `stock.stock_reception_report_action` | Reception Report | qweb-pdf | `stock.report_reception` |  |  |
| `stock.action_report_picking` | Picking Operations | qweb-pdf | `stock.report_picking` | `'Picking Operations - %s - %s' % (object.partner_id.name or '', object.name)` |  |
| `stock.action_report_delivery` | Delivery Slip | qweb-pdf | `stock.report_deliveryslip` | `'Delivery Slip - %s - %s' % (object.partner_id.name or '', object.name)` |  |
| `stock.action_report_picking_packages` | Packages | qweb-pdf | `stock.report_picking_packages` | `'Packages - %s' % (object.name)` |  |
| `stock.return_label_report` | Return slip | qweb-pdf | `stock.report_return_document` |  |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `l10n_ar_stock.email_template_ar_remitos_delivery_guide` | Delivery guide: Send by email | {{ object.company_id.name }} Document (Ref {{ object.l10n_ar_delivery_guide_number }}) |
| `stock.mail_template_data_delivery_confirmation` | Shipping: Send by Email | {{ object.company_id.name }} Delivery Order (Ref {{ object.name or 'n/a' }}) |

Machine-readable definition: `../../../schemas/data/entities/stock.picking.json`; views: `../../../schemas/interfaces/views/stock.picking.json`.
