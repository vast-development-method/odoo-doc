# Barcode Rule (`barcode.rule`)

**Transport name:** `barcode.rule`  
**Storage name:** `barcode_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `barcodes`  
**Extended by packages:** `barcodes_gs1_nomenclature`, `stock`, `point_of_sale`, `pos_loyalty`

Description: Barcode Rule

## Identity and behavior

- Default ordering: `sequence asc, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Rule Name | single line text |  | required; Help: An internal identification for this barcode nomenclature rule |
| `barcode_nomenclature_id` | Barcode Nomenclature | many to one | `barcode.nomenclature` | indexed (btree_not_null) |
| `sequence` | Sequence | integer |  | Help: Used to order rules such that rules with a smaller sequence match first |
| `encoding` | Encoding | selection |  | required; default computed dynamically (_default_encoding); on delete of the target: {"gs1-128": "set default"}; Help: This rule will apply only if the barcode is encoded with the specified encoding; extended by packages `barcodes_gs1_nomenclature` |
| `type` | Type | selection |  | required; default `product`; on delete of the target: {"coupon": "set default"}; extended by packages `barcodes_gs1_nomenclature`, `stock`, `point_of_sale`, `pos_loyalty` |
| `pattern` | Barcode Pattern | single line text |  | required; default `.*`; Help: The barcode matching pattern |
| `alias` | Alias | single line text |  | required; default `0`; Help: The matched pattern will alias to this barcode |
| `is_gs1_nomenclature` | Is Global Standards One Nomenclature | boolean |  | related through path `barcode_nomenclature_id.is_gs1_nomenclature` |
| `gs1_content_type` | Global Standards One Content Type | selection |  | Help: The GS1 content type defines what kind of data the rule will process the barcode as:        * Date: the barcode will be converted into the system datetime;        * Measure: the barcode's value is related to a specific unit;        * Numeric Identifier: fixed length barcode following a specific encoding;        * Alpha-Numeric Name: variable length barcode. |
| `gs1_decimal_usage` | Decimal | boolean |  | Help: If True, use the last digit of AI to determine where the first decimal is |
| `associated_uom_id` | Associated Unit of measure | many to one | `uom.uom` |  |

## Selection values

### `encoding` (Encoding)

| Value | Label |
|---|---|
| `any` | Any |
| `ean13` | EAN-13 |
| `ean8` | EAN-8 |
| `upca` | UPC-A |
| `gs1-128` | GS1-128 |

### `type` (Type)

| Value | Label |
|---|---|
| `alias` | Alias |
| `product` | Unit Product |
| `quantity` | Quantity |
| `location` | Location |
| `location_dest` | Destination location |
| `lot` | Lot number |
| `package` | Package |
| `use_date` | Best before Date |
| `expiration_date` | Expiration Date |
| `package_type` | Package Type |
| `pack_date` | Pack Date |
| `weight` | Weighted Product |
| `price` | Priced Product |
| `discount` | Discounted Product |
| `client` | Client |
| `cashier` | Cashier |
| `coupon` | Coupon |

### `gs1_content_type` (Global Standards One Content Type)

| Value | Label |
|---|---|
| `date` | Date |
| `measure` | Measure |
| `identifier` | Numeric Identifier |
| `alpha` | Alpha-Numeric Name |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_pattern` | validation | self | `barcodes_gs1_nomenclature`, `barcodes` | constrains: `pattern` |  |
| `_default_encoding` | preparation rule | self | `barcodes_gs1_nomenclature` |  |  |

## Validation and error messages (7)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: braces can only contain N's followed by D's. | `barcodes` |
| `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: a rule can only contain one pair of braces. | `barcodes` |
| `_check_pattern` | ValidationError | The barcode pattern %(pattern)s does not lead to a valid regular expression. | `barcodes` |
| `_check_pattern` | ValidationError | There is a syntax error in the barcode pattern %(pattern)s: empty braces. | `barcodes` |
| `_check_pattern` | ValidationError | '*' is not a valid Regex Barcode Pattern. Did you mean '.*'? | `barcodes` |
| `_check_pattern` | ValidationError | The rule pattern "%s" is not valid, it needs two groups: 	- A first one for the Application Identifier (usually 2 to 4 digits); 	- A second one to catch the value. | `barcodes_gs1_nomenclature` |
| `_check_pattern` | ValidationError | The rule pattern '%(rule)s' is not a valid Regex: %(error)s | `barcodes_gs1_nomenclature` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `barcodes` |
| `base.group_erp_manager` | yes | yes | yes | yes | `barcodes` |
| `group_pos_user` | no | yes | no | no | `point_of_sale` |
| `group_pos_manager` | yes | yes | yes | yes | `point_of_sale` |
| `stock.group_stock_user` | no | yes | no | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `barcodes.view_barcode_rule_form` | form |  | `name`, `sequence`, `type`, `encoding`, `pattern`, `alias` |  |  | `barcodes` |
| `barcodes_gs1_nomenclature.view_barcode_gs1_rule_form` | xpath | `barcodes.view_barcode_rule_form` |  |  |  | `barcodes_gs1_nomenclature` |

Machine-readable definition: `../../../schemas/data/entities/barcode.rule.json`; views: `../../../schemas/interfaces/views/barcode.rule.json`.
