# Product Unit of Measure (`uom.uom`)

**Transport name:** `uom.uom`  
**Storage name:** `uom_uom`  
**Kind:** persistent entity (one table)  
**Defined by package:** `uom`  
**Extended by packages:** `product`, `account`, `stock`, `hr_timesheet`, `point_of_sale`, `l10n_ar`, `l10n_cl`, `l10n_eg_edi_eta`, `l10n_es_edi_facturae`, `l10n_hu_edi`, `l10n_id_efaktur_coretax`, `l10n_in`, `l10n_tr_nilvera`

Description: Product Unit of Measure

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Default ordering: `sequence, relative_uom_id, id`
- Hierarchy parent field: `relative_uom_id` with stored hierarchy path
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Unit Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | computed by rule `_compute_sequence` and stored; precomputed before insertion |
| `relative_factor` | Contains | float |  | required; default `1.0`; Help: How much bigger or smaller this unit is compared to the reference UoM for this unit |
| `rounding` | Rounding Precision | float |  | computed by rule `_compute_rounding` (not stored) |
| `active` | Active | boolean |  | default `True`; Help: Uncheck the active field to disable a unit of measure without deleting it. |
| `relative_uom_id` | Reference Unit | many to one | `uom.uom` | indexed (btree_not_null); on delete of the target: cascade |
| `related_uom_ids` | Related UoMs | one to many | `uom.uom` | inverse field `relative_uom_id` |
| `factor` | Absolute Quantity | float |  | computed by rule `_compute_factor` and stored; recursive dependency |
| `parent_path` | Parent Path | single line text |  | indexed |
| `product_uom_ids` | Barcodes | one to many | `product.uom` | restricted by domain `_domain_product_uoms`; inverse field `uom_id` |
| `fiscal_country_codes` | Fiscal Country Codes | single line text |  | computed by rule `_compute_fiscal_country_codes` (not stored) |
| `package_type_id` | Package Type | many to one | `stock.package.type` |  |
| `route_ids` | Routes | many to many |  | related through path `package_type_id.route_ids`; Help: Routes propagated from the package type |
| `timesheet_widget` | Widget | single line text |  |  |
| `is_pos_groupable` | Group Products in point of sale | boolean |  | Help: Check if you want to group products of this unit in point of sale orders |
| `l10n_ar_afip_code` | Code | single line text |  | Help: Argentina: This code will be used on electronic invoice. |
| `l10n_cl_sii_code` | immediate supply of information Code | single line text |  |  |
| `l10n_eg_unit_code_id` | ETA Unit Code | many to one | `l10n_eg_edi.uom.code` | Help: This is the type of unit according to egyptian tax authority |
| `l10n_es_edi_facturae_uom_code` | Spanish electronic data interchange Units | selection |  | required; default `05` |
| `l10n_hu_edi_code` | NAV unit of measure code | selection |  | Help: Choose the corresponding code, or leave blank if none correspond. |
| `l10n_id_uom_code` | E-Faktur unit of measure code | many to one | `l10n_id_efaktur_coretax.uom.code` |  |
| `l10n_in_code` | Indian goods and services tax UQC | single line text |  | Help: Unique Quantity Code (UQC) under GST |

## Selection values

### `l10n_es_edi_facturae_uom_code` (Spanish electronic data interchange Units)

| Value | Label |
|---|---|
| `01` | Units |
| `02` | Hours |
| `03` | Kilograms |
| `04` | Liters |
| `05` | Other |
| `06` | Boxes |
| `07` | Trays, one layer no cover, plastic |
| `08` | Barrels |
| `09` | Jerricans, cylindrical |
| `10` | Bags |
| `11` | Carboys, non-protected |
| `12` | Bottles, non-protected, cylindrical |
| `13` | Canisters |
| `14` | Tetra Briks |
| `15` | Centiliters |
| `16` | Centimeters |
| `17` | Bins |
| `18` | Dozens |
| `19` | Cases |
| `20` | Demijohns, non-protected |
| `21` | Grams |
| `22` | Kilometers |
| `23` | Cans, rectangular |
| `24` | Bunches |
| `25` | Meters |
| `26` | Millimeters |
| `27` | 6-Packs |
| `28` | Packages |
| `29` | Portions |
| `30` | Rolls |
| `31` | Envelopes |
| `32` | Tubs |
| `33` | Cubic meter |
| `34` | Second |
| `35` | Watt |
| `36` | Kilowatt-hour |

### `l10n_hu_edi_code` (NAV unit of measure code)

| Value | Label |
|---|---|
| `PIECE` | Piece |
| `KILOGRAM` | Kilogram |
| `TON` | Ton |
| `KWH` | Kilowatt hour |
| `DAY` | Day |
| `HOUR` | Hour |
| `MINUTE` | Minute |
| `MONTH` | Month |
| `LITER` | Liter |
| `KILOMETER` | Kilometer |
| `CUBIC_METER` | Cubic meter |
| `METER` | Meter |
| `LINEAR_METER` | Linear meter |
| `CARTON` | Carton |
| `PACK` | Package |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_factor_gt_zero` | Constraint | `CHECK (relative_factor!=0)` | The conversion ratio for a unit of measure cannot be 0! | `uom` |

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_unprotected_uom_xml_ids` | internal rule | self | `hr_timesheet`, `uom` |  | Return a list of UoM XML IDs that are not protected by default. Note: Some of these may be protected via overrides in other modules. |
| `_compute_sequence` | computation | self | `uom` | depends: `relative_factor` |  |
| `_compute_rounding` | computation | self | `uom` |  | All Units of Measure share the same rounding precision defined in 'Product Unit'. Set in a compute to ensure compatibility with previous calls to `uom.rounding`. |
| `_compute_factor` | computation | self | `uom` | depends: `relative_factor`, `relative_uom_id`, `relative_uom_id.factor` |  |
| `_onchange_critical_fields` | on change | self | `uom` | onchange: `relative_factor` |  |
| `_check_factor` | validation | self | `uom` | constrains: `relative_factor`, `relative_uom_id` |  |
| `_unlink_except_master_data` | internal rule | self | `uom` | ondelete |  |
| `round` | operation | self, value, rounding_method | `uom` |  | Round the value using the 'Product Unit' precision |
| `compare` | operation | self, value1, value2 | `uom` |  | Compare two measures after rounding them with the 'Product Unit' precision  :param value1: origin value to compare :param value2: value to compare to :return: -1, 0 or 1, if `value1` is lower than, equal to, or greater than `value2`. |
| `is_zero` | operation | self, value | `uom` |  | Check if the value is zero after rounding with the 'Product Unit' precision |
| `_compute_display_name` | computation | self | `uom` | depends: `name`, `relative_factor`, `relative_uom_id`; depends_context: `formatted_display_name` |  |
| `_compute_quantity` | computation | self, qty, to_unit, round, rounding_method, raise_if_failure | `uom` |  | Convert the given quantity from the current UoM `self` into a given one :param qty: the quantity to convert :param to_unit: the destination UomUom record (uom.uom) :param raise_if_failure: only if the conversion is not possible     - if true, raise an exception if the conversion is not possible (different UomUom category),     - otherwise, return the initial quantity |
| `_check_qty` | validation | self, product_qty, uom_id, rounding_method | `uom` |  | Check if product_qty in given uom is a multiple of the packaging qty. If not, rounding the product_qty to closest multiple of the packaging qty according to the rounding_method "UP", "HALF-UP or "DOWN". |
| `_compute_price` | computation | self, price, to_unit | `uom` |  |  |
| `_filter_protected_uoms` | internal rule | self | `uom` |  | Verifies self does not contain protected uoms. |
| `_has_common_reference` | internal rule | self, other_uom | `uom` |  | Check if `self` and `other_uom` have a common reference unit |
| `_domain_product_uoms` | internal rule | self | `product` |  |  |
| `action_open_packaging_barcodes` | user action | self | `product` |  |  |
| `_compute_fiscal_country_codes` | computation | self | `account` | depends_context: `allowed_company_ids` |  |
| `_get_unece_code` | preparation rule | self | `account`, `l10n_tr_nilvera` |  | Returns the UNECE code used for international trading for corresponding to the UoM as per https://unece.org/fileadmin/DAM/cefact/recommendations/rec20/rec20_rev3_Annex2e.pdf |
| `_get_uom_from_unece_code` | preparation rule | self, unece_code | `account` | model |  |
| `write` | lifecycle override | self, vals | `stock` |  |  |
| `_adjust_uom_quantities` | internal rule | self, qty, quant_uom | `stock` |  | This method adjust the quantities of a procurement if its UoM isn't the same as the one of the quant and the parameter 'propagate_uom' is not set. |
| `_load_pos_data_search_read` | internal rule | self, data, config | `point_of_sale` | model |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_factor` | UserError | Reference unit of measure is missing. | `uom` |
| `_unlink_except_master_data` | UserError | The following units of measure are used by the system and cannot be deleted: %s You can archive them instead. | `uom` |
| `write` | UserError | error_msg | `stock` |
| `write` | UserError | error_msg | `stock` |
| `write` | UserError | error_msg | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_timesheet.group_hr_timesheet_user` | no | yes | no | no | `hr_timesheet` |
| `mrp.group_mrp_user` | no | yes | no | no | `mrp` |
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sale` |
| `base.group_system` | yes | yes | yes | yes | `uom` |
| `base.group_user` | no | yes | no | no | `uom` |
| `base.group_public` | no | yes | no | no | `website_sale` |
| `base.group_portal` | no | yes | no | no | `website_sale` |

## Views (16)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.product_uom_form_view_inherit` | xpath | `uom.product_uom_form_view` | `fiscal_country_codes` |  |  | `account` |
| `l10n_ar.product_uom_tree_view` | list | `uom.product_uom_tree_view` | `l10n_ar_afip_code` |  |  | `l10n_ar` |
| `l10n_ar.product_uom_form_view` | div | `uom.product_uom_form_view` | `l10n_ar_afip_code` |  |  | `l10n_ar` |
| `l10n_eg_edi_eta.product_uom_form_view_inherit` | xpath | `uom.product_uom_form_view` | `l10n_eg_unit_code_id` |  |  | `l10n_eg_edi_eta` |
| `l10n_es_edi_facturae.product_uom_tree_view_inherit_l10n_es_edi_facturae` | field | `uom.product_uom_tree_view` | `name`, `l10n_es_edi_facturae_uom_code` |  |  | `l10n_es_edi_facturae` |
| `l10n_es_edi_facturae.product_uom_form_view_inherit_l10n_es_edi_facturae` | field | `uom.product_uom_form_view` | `name`, `l10n_es_edi_facturae_uom_code` |  |  | `l10n_es_edi_facturae` |
| `l10n_hu_edi.uom_uom_form_inherit_l10n_hu_edi` | xpath | `uom.product_uom_form_view` | `l10n_hu_edi_code` |  |  | `l10n_hu_edi` |
| `l10n_id_efaktur_coretax.product_uom_form_view_inherit_coretax` | xpath | `uom.product_uom_form_view` | `l10n_id_uom_code` |  |  | `l10n_id_efaktur_coretax` |
| `l10n_in.product_uom_form_view_inherit_l10n_in` | xpath | `uom.product_uom_form_view` | `l10n_in_code` |  |  | `l10n_in` |
| `point_of_sale.product_uom_form_view_inherit` | div | `uom.product_uom_form_view` | `is_pos_groupable` |  |  | `point_of_sale` |
| `product.uom_uom_form_view_inherit` | xpath | `uom.product_uom_form_view` |  | `action_open_packaging_barcodes` |  | `product` |
| `stock.product_uom_tree_view_inherit` | field | `uom.product_uom_tree_view` | `name`, `package_type_id` |  |  | `stock` |
| `stock.product_uom_form_view_inherit` | div | `uom.product_uom_form_view` | `package_type_id`, `product_uom_ids`, `route_ids` |  |  | `stock` |
| `uom.product_uom_tree_view` | list |  | `sequence`, `name`, `relative_factor`, `relative_uom_id` |  |  | `uom` |
| `uom.product_uom_form_view` | form |  | `name`, `relative_factor`, `relative_uom_id` |  |  | `uom` |
| `uom.uom_uom_view_search` | search |  | `name` |  | `Archived` | `uom` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `uom.product_uom_form_action` | Units & Packagings |  |  |  |  | `uom` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `purchase.menu_purchase_uom_form_action` | Units & Packagings | `purchase.menu_product_in_config_purchase` | `uom.product_uom_form_action` | 10 | `uom.group_uom` |
| `sale.menu_product_uom_form_action` | Units & Packagings |  | `uom.product_uom_form_action` | 35 | `uom.group_uom` |
| `stock.menu_stock_uom_form_action` | Units & Packagings | `menu_product_in_config_stock` | `uom.product_uom_form_action` | 5 | `uom.group_uom` |

Machine-readable definition: `../../../schemas/data/entities/uom.uom.json`; views: `../../../schemas/interfaces/views/uom.uom.json`.
