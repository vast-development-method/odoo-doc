# Package (`stock.package`)

**Transport name:** `stock.package`  
**Storage name:** `stock_package`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`  
**Extended by packages:** `stock_delivery`

Description: Package

## Identity and behavior

- Default ordering: `name, id`
- Display name field: `complete_name`
- Hierarchy parent field: `parent_package_id` with stored hierarchy path
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (29)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Package Reference | single line text |  | required; indexed (trigram); not copied on duplication |
| `complete_name` | Full Package Name | single line text |  | computed by rule `_compute_complete_name` and stored; recursive dependency |
| `dest_complete_name` | Package Name At Destination | single line text |  | computed by rule `_compute_dest_complete_name` (not stored); recursive dependency |
| `quant_ids` | Bulk Content | one to many | `stock.quant` | read only; restricted by domain `["\|", ["quantity", "!=", 0], ["reserved_quantity", "!=", 0]]`; inverse field `package_id` |
| `contained_quant_ids` | Contained Quant | one to many | `stock.quant` | computed by rule `_compute_contained_quant_ids` (not stored); searchable through a search rule |
| `content_description` | Contents | single line text |  | computed by rule `_compute_content_description` (not stored) |
| `package_type_id` | Package Type | many to one | `stock.package.type` | indexed |
| `location_id` | Location | many to one | `stock.location` | computed by rule `_compute_package_info` and stored; indexed; recursive dependency |
| `location_dest_id` | Destination location | many to one | `stock.location` | computed by rule `_compute_location_dest_id` (not stored); searchable through a search rule |
| `company_id` | Company | many to one | `res.company` | read only; computed by rule `_compute_package_info` and stored; indexed; recursive dependency |
| `owner_id` | Owner | many to one | `res.partner` | read only; computed by rule `_compute_owner_id` (not stored); searchable through a search rule |
| `parent_package_id` | Container | many to one | `stock.package` | indexed (btree_not_null) |
| `child_package_ids` | Contained Packages | one to many | `stock.package` | inverse field `parent_package_id` |
| `all_children_package_ids` | All Children Package | one to many | `stock.package` | computed by rule `_compute_all_children_package_ids` (not stored); searchable through a search rule |
| `package_dest_id` | Destination Container | many to one | `stock.package` | indexed (btree_not_null) |
| `outermost_package_id` | Outermost Destination Container | many to one | `stock.package` | computed by rule `_compute_outermost_package_id` (not stored); searchable through a search rule; recursive dependency |
| `child_package_dest_ids` | Assigned Contained Packages | one to many | `stock.package` | inverse field `package_dest_id` |
| `move_line_ids` | Move Line | one to many | `stock.move.line` | computed by rule `_compute_move_line_ids` (not stored); searchable through a search rule |
| `picking_ids` | Transfers | many to many | `stock.picking` | computed by rule `_compute_picking_ids` (not stored); searchable through a search rule; Help: Transfers in which the Package is set as Destination Package |
| `shipping_weight` | Shipping Weight | float |  | Help: Total weight of the package. |
| `valid_sscc` | Package name is valid SSCC | boolean |  | computed by rule `_compute_valid_sscc` (not stored) |
| `pack_date` | Pack Date | date |  | default computed dynamically (fields.Date.today) |
| `parent_path` | Parent Path | single line text |  | indexed |
| `json_popover` | JavaScript Object Notation data for popover widget | single line text |  | computed by rule `_compute_json_popover` (not stored) |
| `weight` | Weight | float |  | computed by rule `_compute_weight` (not stored); precision `Stock Weight`; Help: Total weight of all the products contained in the package. |
| `weight_uom_name` | Weight unit of measure label | single line text |  | read only; computed by rule `_compute_weight_uom_name` (not stored); default computed dynamically (_get_default_weight_uom) |
| `weight_is_kg` | Technical field indicating whether weight uom is kg or not (i.e. lb) | boolean |  | computed by rule `_compute_weight_is_kg` (not stored) |
| `weight_uom_rounding` | Technical field indicating weight's number of decimal places | float |  | computed by rule `_compute_weight_is_kg` (not stored) |
| `package_carrier_type` | Package Carrier Type | selection |  | related through path `package_type_id.package_carrier_type` |

## Operations (41)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_all_children_package_ids` | computation | self | `stock` | depends: `child_package_ids`, `child_package_ids.parent_path` |  |
| `_compute_display_name` | computation | self | `stock` | depends: `complete_name`, `package_type_id.packaging_length`, `package_type_id.width`, `package_type_id.height`; depends_context: `formatted_display_name`, `show_dest_package`, `show_src_package`, `is_done` |  |
| `_compute_complete_name` | computation | self | `stock` | depends: `name`, `parent_package_id.complete_name` |  |
| `_compute_dest_complete_name` | computation | self | `stock` | depends: `name`, `package_dest_id.dest_complete_name` |  |
| `_compute_contained_quant_ids` | computation | self | `stock` | depends: `quant_ids`, `all_children_package_ids.quant_ids` |  |
| `_compute_content_description` | computation | self | `stock` | depends: `contained_quant_ids` |  |
| `_compute_json_popover` | computation | self | `stock` |  |  |
| `_compute_location_dest_id` | computation | self | `stock` | depends: `move_line_ids` |  |
| `_compute_move_line_ids` | computation | self | `stock` | depends: `location_id`, `child_package_dest_ids` |  |
| `_compute_package_info` | computation | self | `stock` | depends: `child_package_ids`, `child_package_ids.location_id`, `quant_ids` |  |
| `_compute_picking_ids` | computation | self | `stock` | depends: `child_package_dest_ids` |  |
| `_compute_owner_id` | computation | self | `stock` | depends: `quant_ids.owner_id` |  |
| `_compute_outermost_package_id` | computation | self | `stock` | depends: `package_dest_id`, `package_dest_id.outermost_package_id` |  |
| `_compute_valid_sscc` | computation | self | `stock` | depends: `name` |  |
| `_search_all_children_package_ids` | search rule | self, operator, value | `stock` |  |  |
| `_search_contained_quant_ids` | search rule | self, operator, value | `stock` |  |  |
| `_search_location_dest_id` | search rule | self, operator, value | `stock` |  |  |
| `_search_move_line_ids` | search rule | self, operator, value | `stock` |  |  |
| `_search_outermost_package_id` | search rule | self, operator, value | `stock` |  |  |
| `_search_owner` | search rule | self, operator, value | `stock` |  |  |
| `_search_picking_ids` | search rule | self, operator, value | `stock` |  |  |
| `create` | lifecycle override | self, vals_list | `stock` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `unpack` | operation | self | `stock` |  | Unpacks quants directly inside the container, and remove contained packages from this package. |
| `action_add_to_picking` | user action | self | `stock` |  |  |
| `_pre_put_in_pack_hook` | internal rule | self, package_id, package_type_id, package_name, from_package_wizard | `stock_delivery`, `stock` |  |  |
| `_post_put_in_pack_hook` | internal rule | self | `stock_delivery`, `stock` |  |  |
| `action_put_in_pack` | user action | self, package_id, package_type_id, package_name | `stock` |  |  |
| `action_remove_package` | user action | self | `stock` |  | Removes all packages in self from the destination container tree. For move lines directly linked to a package (through result_package_id) - If the entire package is moved, remove the move lines entirely from the picking - Otherwise, just unset the packages as destination package |
| `action_view_picking` | user action | self | `stock` |  |  |
| `_check_move_lines_map_quant` | validation | self, move_lines | `stock` |  | This method checks that all product (quants) of self (package) are well present in the `move_line_ids`. |
| `_get_weight` | preparation rule | self, picking_id | `stock` |  |  |
| `_has_issues` | internal rule | self | `stock` |  |  |
| `_apply_dest_to_package` | internal rule | self, processed_package_ids | `stock` |  | Moves the packages to their new container and checks that no contained quants of the new container would be in different locations. |
| `_get_all_children_package_dest_ids` | preparation rule | self | `stock` |  | Gets all child packages that have the packages in self as their `package_dest_id` recursively. Since we can only have a single _parent field on the model, we need to do this manually. :returns: A dict for each record in self containing all packages that have it as `package_dest_id`, even remotely :returns: A list containing all children ids, regardless of their parents |
| `_get_all_package_dest_ids` | preparation rule | self | `stock` |  | Gets all parent destination packages recursively. Since we can only have a single _parent field on the model, we need to do this manually. :returns: A list containing all parent ids |
| `_apply_package_dest_for_entire_packs` | internal rule | self, allowed_package_ids | `stock` |  | When a package is assigned to a picking, if all of its container is added, then we consider the container to be added itself, unless the container is a reusable package itself. |
| `_compute_weight` | computation | self | `stock_delivery` | depends: `contained_quant_ids`, `package_type_id` |  |
| `_get_default_weight_uom` | preparation rule | self | `stock_delivery` |  |  |
| `_compute_weight_uom_name` | computation | self | `stock_delivery` |  |  |
| `_compute_weight_is_kg` | computation | self | `stock_delivery` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | Cannot remove the location of a non empty package | `stock` |
| `write` | ValidationError | A package can't have one of its contained packages as destination container. | `stock` |
| `write` | UserError | Cannot move an empty package | `stock` |
| `_apply_dest_to_package` | UserError | Packages %(duplicate_names)s are moved to different locations while being in the same container %(container_name)s. | `stock` |
| `_apply_dest_to_package` | UserError | Can't move a container having packages in another location (%(old_location)s) to a different location (%(new_location)s). | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |
| `stock.group_stock_user` | yes | yes | yes | yes | `stock` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| stock_package multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (7)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.stock_package_view_search` | search |  | `name`, `location_id`, `package_type_id` |  | `In internal locations`, `Main Packages`, `Location`, `Package Type` | `stock` |
| `stock.stock_package_view_form` | form |  | `name`, `package_type_id`, `owner_id`, `location_id`, `parent_package_id`, `company_id`, `pack_date`, `contained_quant_ids`, `product_id`, `package_id`, `lot_id`, `quantity`, `product_uom_id`, `product_id`, `lot_id`, `quantity`, `product_uom_id` | `Unpack`, `Package Transfers` |  | `stock` |
| `stock.stock_package_view_list` | list |  | `name`, `parent_package_id`, `package_type_id`, `location_id`, `company_id` |  |  | `stock` |
| `stock.stock_package_view_list_editable` | list |  | `name`, `package_type_id`, `json_popover`, `location_dest_id`, `parent_package_id`, `package_dest_id`, `company_id` | `Put in Pack`, `Remove` |  | `stock` |
| `stock.stock_package_view_add_list` | list |  | `name`, `parent_package_id`, `package_type_id`, `location_id`, `content_description` |  |  | `stock` |
| `stock.stock_package_view_kanban` | kanban |  | `name`, `package_type_id` |  |  | `stock` |
| `stock_delivery.stock_package_view_form` | field | `stock.stock_package_view_form` | `company_id`, `shipping_weight`, `weight_uom_name`, `weight`, `weight_uom_name` |  |  | `stock_delivery` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `stock.action_package_view` | Packages | list,kanban,form |  | `{             'search_default_location': True,             'search_default_internal': True,             }` |  | `stock` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `stock.action_report_package_barcode` | Package Barcode with Contents | qweb-pdf | `stock.report_package_barcode` |  |  |
| `stock.action_report_package_barcode_small` | Package Barcode (PDF) | qweb-pdf | `stock.report_package_barcode_small` |  |  |
| `stock.label_package_template` | Package Barcode (ZPL) | qweb-text | `stock.label_package_template_view` |  |  |

Machine-readable definition: `../../../schemas/data/entities/stock.package.json`; views: `../../../schemas/interfaces/views/stock.package.json`.
