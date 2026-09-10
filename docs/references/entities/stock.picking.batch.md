# Batch Transfer (`stock.picking.batch`)

**Transport name:** `stock.picking.batch`  
**Storage name:** `stock_picking_batch`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock_picking_batch`  
**Extended by packages:** `delivery_stock_picking_batch`, `l10n_ro_edi_stock_batch`, `stock_fleet`

Description: Batch Transfer

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `name desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (56)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Batch Transfer | single line text |  | required; read only; default computed dynamically (lambda self: _('New')); not copied on duplication |
| `description` | Description | single line text |  |  |
| `user_id` | Responsible | many to one | `res.users` | changes are tracked in the message thread; must belong to the same company |
| `company_id` | Company | many to one | `res.company` | required; read only; default computed dynamically (lambda self: self.env.company); indexed |
| `picking_ids` | Transfers | one to many | `stock.picking` | restricted by domain `[('id', 'in', allowed_picking_ids)]`; must belong to the same company; inverse field `batch_id`; Help: List of transfers associated to this batch |
| `show_check_availability` | Show Check Availability | boolean |  | computed by rule `_compute_move_ids` (not stored) |
| `show_allocation` | Show Allocation Button | boolean |  | computed by rule `_compute_show_allocation` (not stored) |
| `allowed_picking_ids` | Allowed Picking | one to many | `stock.picking` | computed by rule `_compute_allowed_picking_ids` (not stored) |
| `move_ids` | Stock moves | one to many | `stock.move` | computed by rule `_compute_move_ids` (not stored) |
| `move_line_ids` | Stock move lines | one to many | `stock.move.line` | computed by rule `_compute_move_line_ids` (not stored); writable through an inverse rule; searchable through a search rule |
| `state` | State | selection |  | required; read only; computed by rule `_compute_state` and stored; default `draft`; changes are tracked in the message thread; indexed; not copied on duplication |
| `picking_type_id` | Operation Type | many to one | `stock.picking.type` | indexed; not copied on duplication; must belong to the same company |
| `warehouse_id` | Warehouse | many to one | `stock.warehouse` | related through path `picking_type_id.warehouse_id` |
| `picking_type_code` | Picking Type Code | selection |  | related through path `picking_type_id.code` |
| `scheduled_date` | Scheduled Date | date and time |  | computed by rule `_compute_scheduled_date` and stored; not copied on duplication; Help: Scheduled date for the transfers to be processed.               - If manually set then scheduled date for all transfers in batch will automatically update to this date.               - If not manually changed and transfers are added/removed/updated then this will be their earliest scheduled date                 but this scheduled date will not be set for all transfers in batch. |
| `is_wave` | This batch is a wave | boolean |  |  |
| `show_lots_text` | Show Lots Text | boolean |  | computed by rule `_compute_show_lots_text` (not stored) |
| `estimated_shipping_weight` | shipping_weight | float |  | computed by rule `_compute_estimated_shipping_capacity` (not stored); precision `Product Unit` |
| `estimated_shipping_volume` | shipping_volume | float |  | computed by rule `_compute_estimated_shipping_capacity` (not stored); precision `Product Unit` |
| `properties` | Properties | properties |  |  |
| `l10n_ro_edi_stock_document_ids` | Localization Ro Electronic data interchange Stock Document | one to many | `l10n_ro_edi.document` | inverse field `batch_id` |
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
| `vehicle_id` | Vehicle | many to one | `fleet.vehicle` |  |
| `vehicle_category_id` | Vehicle Category | many to one | `fleet.vehicle.model.category` | computed by rule `_compute_vehicle_category_id` and stored |
| `allowed_dock_ids` | Allowed Docks | many to many |  | related through path `picking_type_id.dock_ids` |
| `dock_id` | Dock | many to one | `stock.location` | computed by rule `_compute_dock_id` and stored; restricted by domain `[('id', 'child_of', allowed_dock_ids)]` |
| `vehicle_weight_capacity` | Vehcilce Payload Capacity | float |  | related through path `vehicle_category_id.weight_capacity` |
| `weight_uom_name` | Weight unit of measure label | single line text |  | computed by rule `_compute_weight_uom_name` (not stored) |
| `vehicle_volume_capacity` | Max Volume (m³) | float |  | related through path `vehicle_category_id.volume_capacity` |
| `volume_uom_name` | Volume unit of measure label | single line text |  | computed by rule `_compute_volume_uom_name` (not stored) |
| `driver_id` | Driver | many to one | `res.partner` | computed by rule `_compute_driver_id` and stored |
| `used_weight_percentage` | Weight % | float |  | computed by rule `_compute_capacity_percentage` (not stored) |
| `used_volume_percentage` | Volume % | float |  | computed by rule `_compute_capacity_percentage` (not stored) |
| `end_date` | End Date | date and time |  | computed by rule `_compute_end_date` and stored |
| `has_dispatch_management` | Dispatch Management | boolean |  | related through path `picking_type_id.dispatch_management` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `in_progress` | In progress |
| `done` | Done |
| `cancel` | Cancelled |

## State fields

State machine fields of this entity: `state`, `l10n_ro_edi_stock_state`. Transitions are specified in the domain documents.

## Operations (66)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `stock_picking_batch` | depends: `description`; depends_context: `add_to_existing_batch` |  |
| `_compute_show_lots_text` | computation | self | `stock_picking_batch` | depends: `picking_type_id` |  |
| `_compute_estimated_shipping_capacity` | computation | self | `stock_picking_batch` |  |  |
| `_compute_allowed_picking_ids` | computation | self | `stock_picking_batch` | depends: `company_id`, `picking_type_id`, `state` |  |
| `_compute_move_ids` | computation | self | `stock_picking_batch` | depends: `picking_ids`, `picking_ids.move_line_ids`, `picking_ids.move_ids`, `picking_ids.move_ids.state` |  |
| `_compute_move_line_ids` | computation | self | `stock_picking_batch` | depends: `picking_ids`, `picking_ids.move_line_ids` |  |
| `_search_move_line_ids` | search rule | self, operator, value | `stock_picking_batch` |  |  |
| `_compute_show_allocation` | computation | self | `stock_picking_batch` | depends: `state`, `move_ids`, `picking_type_id` |  |
| `_compute_state` | computation | self | `stock_picking_batch` | depends: `picking_ids`, `picking_ids.state` |  |
| `_compute_scheduled_date` | computation | self | `stock_picking_batch` | depends: `picking_ids`, `picking_ids.scheduled_date` |  |
| `onchange_scheduled_date` | on change | self | `stock_picking_batch` | onchange: `scheduled_date` |  |
| `_set_move_line_ids` | internal rule | self | `stock_picking_batch` |  |  |
| `create` | lifecycle override | self, vals_list | `stock_fleet`, `stock_picking_batch` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock_fleet`, `stock_picking_batch` |  |  |
| `_unlink_if_not_done` | internal rule | self | `stock_picking_batch` | ondelete |  |
| `action_confirm` | user action | self | `stock_picking_batch` |  | Sanity checks, confirm the pickings and mark the batch as confirmed. |
| `action_cancel` | user action | self | `stock_picking_batch` |  |  |
| `action_print` | user action | self | `stock_picking_batch` |  |  |
| `action_done` | user action | self | `l10n_ro_edi_stock_batch`, `stock_picking_batch` |  |  |
| `action_assign` | user action | self | `stock_picking_batch` |  |  |
| `action_put_in_pack` | user action | self, package_id, package_type_id, package_name | `stock_picking_batch` |  | Action to put move lines with 'Done' quantities into a new pack This method follows same logic to stock.picking. |
| `action_view_reception_report` | user action | self | `stock_picking_batch` |  |  |
| `action_open_label_layout` | user action | self | `stock_picking_batch` |  |  |
| `action_merge` | user action | self | `stock_picking_batch` |  |  |
| `action_batch_detailed_operations` | user action | self | `stock_picking_batch` |  |  |
| `action_see_packages` | user action | self | `stock_picking_batch` |  |  |
| `_prepare_name` | preparation rule | self, picking_type, sequence_code, company_id | `stock_picking_batch` | model |  |
| `_sanity_check` | internal rule | self | `stock_picking_batch` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `stock_picking_batch` |  |  |
| `_is_picking_auto_mergeable` | internal rule | self, picking | `delivery_stock_picking_batch`, `stock_picking_batch` |  | Verifies if a picking can be safely inserted into the batch without violating auto_batch_constrains. |
| `_is_line_auto_mergeable` | internal rule | self, num_of_moves, num_of_pickings, weight | `delivery_stock_picking_batch`, `stock_picking_batch` |  | Verifies if a line can be safely inserted into the wave without violating auto_batch_constrains. |
| `_are_moves_auto_mergeable` | internal rule | self, num_of_moves | `stock_picking_batch` |  |  |
| `_are_pickings_auto_mergeable` | internal rule | self, num_of_pickings | `stock_picking_batch` |  |  |
| `_get_merged_batch_vals` | preparation rule | self | `stock_fleet`, `stock_picking_batch` |  |  |
| `_l10n_ro_edi_stock_reset_variable_selection_fields` | on change | self | `l10n_ro_edi_stock_batch` | onchange: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_default_location_type` | computation | self | `l10n_ro_edi_stock_batch` | depends: `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_available_operation_scopes` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_available_location_types` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_operation_type` |  |
| `_compute_l10n_ro_edi_stock_current_document_state` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_current_document_uit` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_document_ids`, `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_enable` | computation | self | `l10n_ro_edi_stock_batch` | depends: `company_id.account_fiscal_country_id.code` |  |
| `_compute_l10n_ro_edi_stock_enable_send` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_enable_fetch` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_enable`, `state`, `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_enable_amend` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_state` |  |
| `_compute_l10n_ro_edi_stock_fields_readonly` | computation | self | `l10n_ro_edi_stock_batch` | depends: `l10n_ro_edi_stock_state` |  |
| `_l10n_ro_edi_stock_validate_fetch_data` | internal rule | self, errors | `l10n_ro_edi_stock_batch` |  |  |
| `action_l10n_ro_edi_stock_send_etransport` | user action | self | `l10n_ro_edi_stock_batch` |  |  |
| `action_l10n_ro_edi_stock_fetch_status` | user action | self | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_get_current_document` | internal rule | self | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_get_all_documents` | internal rule | self, states | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_get_last_document` | internal rule | self, state | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_create_document_stock_sent` | internal rule | self, values | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_create_document_stock_sending_failed` | internal rule | self, values | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_create_document_stock_validated` | internal rule | self, values | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_send_etransport_document` | internal rule | self, send_type | `l10n_ro_edi_stock_batch` |  | Send the eTransport document to anaf :param send_type: 'send' (initial sending of document) \| 'amend' (correct the already sent document) |
| `_l10n_ro_edi_stock_fetch_document_status` | internal rule | self | `l10n_ro_edi_stock_batch` |  |  |
| `_l10n_ro_edi_stock_report_unhandled_document_state` | internal rule | self, state | `l10n_ro_edi_stock_batch` |  |  |
| `_compute_end_date` | computation | self | `stock_fleet` | depends: `scheduled_date` |  |
| `_compute_vehicle_category_id` | computation | self | `stock_fleet` | depends: `vehicle_id` |  |
| `_compute_dock_id` | computation | self | `stock_fleet` | depends: `picking_ids`, `picking_ids.location_id`, `picking_ids.location_dest_id`, `picking_type_id` |  |
| `_compute_weight_uom_name` | computation | self | `stock_fleet` |  |  |
| `_compute_volume_uom_name` | computation | self | `stock_fleet` |  |  |
| `_compute_driver_id` | computation | self | `stock_fleet` | depends: `vehicle_id` |  |
| `_compute_capacity_percentage` | computation | self | `stock_fleet` | depends: `estimated_shipping_weight`, `vehicle_category_id.weight_capacity`, `estimated_shipping_volume`, `vehicle_category_id.volume_capacity` |  |
| `order_on_zip` | operation | self | `stock_fleet` |  |  |
| `_set_moves_destination_to_dock` | internal rule | self | `stock_fleet` |  |  |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_if_not_done` | UserError | You cannot delete Done batch transfers. | `stock_picking_batch` |
| `action_confirm` | UserError | You have to set some pickings to batch. | `stock_picking_batch` |
| `action_merge` | UserError | Please select at least two batch/wave transfers to merge. | `stock_picking_batch` |
| `action_merge` | UserError | Batch/Wave transfers with different operation types cannot be merged. | `stock_picking_batch` |
| `action_merge` | UserError | Batch transfers cannot be merged with wave transfers and vice versa. | `stock_picking_batch` |
| `action_merge` | UserError | Batch/Wave transfers with different states cannot be merged. | `stock_picking_batch` |
| `action_merge` | UserError | You cannot merge done or cancelled batch/wave transfers. | `stock_picking_batch` |
| `_sanity_check` | UserError | The following transfers cannot be added to batch transfer %(batch)s. Please check their states and operation types.  Incompatibilities: %(incompatible_transfers)s | `stock_picking_batch` |
| `action_done` | UserError | All Pickings in a Batch Transfer should have the same Carrier | `l10n_ro_edi_stock_batch` |
| `action_done` | UserError | All Pickings in a Batch Transfer should have the same Commercial Partner | `l10n_ro_edi_stock_batch` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | yes | `stock_picking_batch` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock.picking.batch multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

## Views (16)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_view_batch_form` | xpath | `stock_picking_batch.stock_picking_batch_form` | `l10n_ro_edi_stock_enable_send`, `l10n_ro_edi_stock_enable_fetch`, `l10n_ro_edi_stock_enable_amend` | `Send eTransport`, `Amend eTransport`, `Fetch Status` |  | `l10n_ro_edi_stock_batch` |
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_stock_picking_batch_view_tree` | field | `stock_picking_batch.stock_picking_batch_tree` | `company_id` | `Fetch Status` |  | `l10n_ro_edi_stock_batch` |
| `l10n_ro_edi_stock_batch.l10n_ro_edi_stock_stock_picking_batch_filter` | field | `stock_picking_batch.stock_picking_batch_filter` | `user_id`, `l10n_ro_edi_stock_state` |  |  | `l10n_ro_edi_stock_batch` |
| `stock_fleet.stock_picking_batch_pivot` | pivot |  | `scheduled_date`, `vehicle_id` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_graph` | graph |  | `scheduled_date`, `vehicle_category_id` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_form` | xpath | `stock_picking_batch.stock_picking_batch_form` | `dock_id`, `vehicle_id`, `vehicle_category_id`, `estimated_shipping_weight`, `weight_uom_name`, `used_weight_percentage`, `estimated_shipping_volume`, `volume_uom_name`, `used_volume_percentage` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_tree` | data | `stock_picking_batch.stock_picking_batch_tree` | `user_id`, `user_id`, `vehicle_category_id`, `vehicle_id`, `dock_id`, `used_volume_percentage`, `used_weight_percentage` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_filter` | field | `stock_picking_batch.stock_picking_batch_filter` | `user_id`, `vehicle_id`, `dock_id`, `driver_id` |  |  | `stock_fleet` |
| `stock_fleet.stock_picking_batch_kanban` | data | `stock_picking_batch.stock_picking_batch_kanban` | `dock_id`, `state`, `scheduled_date`, `user_id` |  |  | `stock_fleet` |
| `stock_picking_batch.stock_picking_batch_form` | form |  | `company_id`, `show_check_availability`, `show_allocation`, `picking_type_code`, `is_wave`, `show_lots_text`, `state`, `name`, `user_id`, `picking_type_id`, `scheduled_date`, `description`, `properties`, `move_ids`, `allowed_picking_ids`, `picking_ids`, `picking_ids` | `Confirm`, `Validate`, `Check Availability`, `Validate`, `Check Availability`, `Print`, `Print Labels`, `Cancel`, `Packages`, `action_batch_detailed_operations`, `Allocation`, `Put in Pack` |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_tree` | list |  | `company_id`, `name`, `description`, `scheduled_date`, `user_id`, `picking_type_id`, `company_id`, `state`, `activity_exception_decoration` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_kanban` | kanban |  | `company_id`, `name`, `state`, `description`, `picking_type_id`, `scheduled_date`, `user_id` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_calendar` | calendar |  | `scheduled_date` |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_batch_filter` | search |  | `name`, `picking_type_id`, `user_id` |  | `To Do`, `My Transfers`, `Draft`, `In Progress`, `Done`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Responsible`, `State` | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_wave_tree` | xpath | `stock_picking_batch.stock_picking_batch_tree` |  |  |  | `stock_picking_batch` |
| `stock_picking_batch.stock_picking_wave_kanban` | xpath | `stock_picking_batch.stock_picking_batch_kanban` |  |  |  | `stock_picking_batch` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock_picking_batch.stock_picking_batch_action` | Batch Transfers | list,kanban,form | `[('is_wave', '=', False)]` | `{'search_default_draft': True, 'search_default_in_progress': True}` |  | `stock_picking_batch` |
| `stock_picking_batch.action_picking_tree_wave` | Wave Transfers | list,kanban,form | `[('is_wave', '=', True)]` | `{'search_default_draft': True, 'search_default_in_progress': True}` |  | `stock_picking_batch` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `stock_picking_batch.action_unreserve_batch_picking` | Unreserve | code |  | yes |
| `stock_picking_batch.action_merge_batch_picking` | Merge | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock_picking_batch.action_report_picking_batch` | Batch Transfer | qweb-pdf | `stock_picking_batch.report_picking_batch` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.picking.batch.json`; views: `../../../schemas/interfaces/views/stock.picking.batch.json`.
