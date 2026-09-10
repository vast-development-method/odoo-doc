# Barcode Nomenclature (`barcode.nomenclature`)

**Transport name:** `barcode.nomenclature`  
**Storage name:** `barcode_nomenclature`  
**Kind:** persistent entity (one table)  
**Defined by package:** `barcodes`  
**Extended by packages:** `barcodes_gs1_nomenclature`

Description: Barcode Nomenclature

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Barcode Nomenclature | single line text |  | required; Help: An internal identification of the barcode nomenclature |
| `rule_ids` | Rules | one to many | `barcode.rule` | inverse field `barcode_nomenclature_id`; Help: The list of barcode rules |
| `upc_ean_conv` | Universal Product Code/European Article Number Conversion | selection |  | required; default `always`; Help: UPC Codes can be converted to EAN by prefixing them with a zero. This setting determines if a UPC/EAN barcode should be automatically converted in one way or another when trying to match a rule with the other encoding. |
| `is_gs1_nomenclature` | Is Global Standards One Nomenclature | boolean |  | Help: This Nomenclature use the GS1 specification, only GS1-128 encoding rules is accepted is this kind of nomenclature. |
| `gs1_separator_fnc1` | FNC1 Separator | single line text |  | default `(Alt029\|#\|\x1D)`; Help: Alternative regex delimiter for the FNC1. The separator must not match the begin/end of any related rules pattern. |

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `sanitize_ean` | operation | self, ean | `barcodes` | model | Returns a valid zero padded EAN-13 from an EAN prefix.  :type ean: str |
| `sanitize_upc` | operation | self, upc | `barcodes` | model | Returns a valid zero padded UPC-A from a UPC-A prefix.  :type upc: str |
| `match_pattern` | operation | self, barcode, pattern | `barcodes` |  | Checks barcode matches the pattern and retrieves the optional numeric value in barcode.  :param barcode: :type barcode: str :param pattern: :type pattern: str :return: an object containing:     - value: the numerical value encoded in the barcode (0 if no value encoded)     - base_code: the barcode in which numerical content is replaced by 0's     - match: boolean :rtype: dict |
| `parse_barcode` | operation | self, barcode | `barcodes` |  |  |
| `parse_nomenclature_barcode` | operation | self, barcode | `barcodes_gs1_nomenclature`, `barcodes` |  | Attempts to interpret and parse a barcode.  :param barcode: :type barcode: str :return: A object containing various information about the barcode, like as:      - code: the barcode     - type: the barcode's type     - value: if the id encodes a numerical value, it will be put there     - base_code: the barcode code with all the encoding parts set to       zero; the one put on the product in the backend  :rtype: dict |
| `parse_uri` | operation | self, barcode | `barcodes` | model | Convert supported URI format (lgtin, sgtin, sgtin-96, sgtin-198, sscc and ssacc-96) into a GS1 barcode. :param barcode str: the URI as a string. :rtype: str |
| `_convert_uri_gtin_data_into_tracking_number` | internal rule | self, base_code, data | `barcodes` | model |  |
| `_convert_uri_sscc_data_into_package` | internal rule | self, base_code, data | `barcodes` | model |  |
| `_unlink_except_default` | internal rule | self | `barcodes` | ondelete |  |
| `_check_pattern` | validation | self | `barcodes_gs1_nomenclature` | constrains: `gs1_separator_fnc1` |  |
| `gs1_date_to_date` | operation | self, gs1_date | `barcodes_gs1_nomenclature` | model | Converts a GS1 date into a datetime.date.  :param gs1_date: A year formated as yymmdd :type gs1_date: str :return: converted date :rtype: datetime.date |
| `parse_gs1_rule_pattern` | operation | self, match, rule | `barcodes_gs1_nomenclature` |  |  |
| `gs1_decompose_extended` | operation | self, barcode | `barcodes_gs1_nomenclature` |  | Try to decompose the gs1 extended barcode into several unit of information using gs1 rules.  Return a ordered list of dict |
| `_preprocess_gs1_search_args` | internal rule | self, domain, barcode_types, field | `barcodes_gs1_nomenclature` | model | Helper method to preprocess 'domain' in _search method to add support to search with GS1 barcode result. Cut off the padding if using GS1 and searching on barcode. If the barcode is only digits to keep the original barcode part only. |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_except_default` | UserError | You cannot delete '%(name)s' because it's the default barcode nomenclature. | `barcodes` |
| `_check_pattern` | ValidationError | The FNC1 Separator Alternative is not a valid Regex: %(error)s | `barcodes_gs1_nomenclature` |
| `gs1_date_to_date` | ValidationError | A GS1 barcode nomenclature pattern was matched. However, the barcode failed to be converted to a valid date: '%(error_message)s' | `barcodes_gs1_nomenclature` |
| `parse_gs1_rule_pattern` | ValidationError | There is something wrong with the barcode rule "%s" pattern. If this rule uses decimal, check it can't get sometime else than a digit as last char for the Application Identifier. Check also the possible matched values can only be digits, otherwise the value can't be casted as a measure. | `barcodes_gs1_nomenclature` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `barcodes` |
| `base.group_erp_manager` | yes | yes | yes | yes | `barcodes` |
| `base.group_portal` | no | yes | no | no | `mrp_subcontracting` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `barcodes.view_barcode_nomenclature_form` | form |  | `name`, `upc_ean_conv`, `rule_ids`, `sequence`, `name`, `type`, `encoding`, `pattern` |  |  | `barcodes` |
| `barcodes.view_barcode_nomenclature_tree` | list |  | `name` |  |  | `barcodes` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_nomenclature_form` | xpath | `barcodes.view_barcode_nomenclature_form` |  |  |  | `barcodes_gs1_nomenclature` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_nomenclature_tree` | xpath | `barcodes.view_barcode_nomenclature_tree` | `is_gs1_nomenclature` |  |  | `barcodes_gs1_nomenclature` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `barcodes.action_barcode_nomenclature_form` | Barcode Nomenclatures | list,kanban,form |  |  |  | `barcodes` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `stock.menu_wms_barcode_nomenclature_all` |  | `menu_product_in_config_stock` | `barcodes.action_barcode_nomenclature_form` | 50 | `base.group_no_one` |

Machine-readable definition: `../../../schemas/data/entities/barcode.nomenclature.json`; views: `../../../schemas/interfaces/views/barcode.nomenclature.json`.
