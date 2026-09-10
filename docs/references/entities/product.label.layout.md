# Choose the sheet layout to print the labels (`product.label.layout`)

**Transport name:** `product.label.layout`  
**Storage name:** `product_label_layout`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `product`  
**Extended by packages:** `stock`

Description: Choose the sheet layout to print the labels

## Fields (12)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `print_format` | Format | selection |  | required; default `2x7xprice`; on delete of the target: {"zpl": "set default", "zplxprice": "set default"}; extended by packages `stock` |
| `custom_quantity` | Copies | integer |  | required; default `1` |
| `product_ids` | Product | many to many | `product.product` |  |
| `product_tmpl_ids` | Product Tmpl | many to many | `product.template` |  |
| `extra_html` | Extra Content | rich text |  | default  |
| `rows` | Rows | integer |  | computed by rule `_compute_dimensions` (not stored) |
| `columns` | Columns | integer |  | computed by rule `_compute_dimensions` (not stored) |
| `pricelist_id` | Pricelist | many to one | `product.pricelist` |  |
| `move_ids` | Move | many to many | `stock.move` |  |
| `move_quantity` | Quantity to print | selection |  | required; default `custom` |
| `zpl_template` | ZPL Template | selection |  | required; default `normal` |
| `zpl_preview` | ZPL Preview | image |  | read only; default computed dynamically (_get_zpl_label_placeholder) |

## Selection values

### `print_format` (Format)

| Value | Label |
|---|---|
| `dymo` | Dymo |
| `2x7xprice` | 2 x 7 with price |
| `4x7xprice` | 4 x 7 with price |
| `4x12` | 4 x 12 |
| `4x12xprice` | 4 x 12 with price |
| `zpl` | ZPL Labels |
| `zplxprice` | ZPL Labels with price |

### `move_quantity` (Quantity to print)

| Value | Label |
|---|---|
| `move` | Operation Quantities |
| `custom` | Custom |

### `zpl_template` (ZPL Template)

| Value | Label |
|---|---|
| `normal` | Normal (2.25" x 1.25") |
| `small` | Small (1.25" x 1.00") |
| `alternative` | Alternative (2.00" x 1.00") |
| `jewelry` | Jewelry (2.20" x 0.50") |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_dimensions` | computation | self | `product` | depends: `print_format` |  |
| `_prepare_report_data` | preparation rule | self | `product`, `stock` |  |  |
| `process` | operation | self | `product` |  |  |
| `_get_zpl_label_placeholder` | preparation rule | self | `stock` | model |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_prepare_report_data` | UserError | You need to set a positive quantity. | `product` |
| `_prepare_report_data` | UserError | No product to print, if the product is archived please unarchive it before printing its label. | `product` |
| `process` | UserError | Unable to find report template for %s format | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `product` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_label_layout_form` | form |  | `product_ids`, `product_tmpl_ids`, `custom_quantity`, `print_format`, `pricelist_id`, `extra_html` | `Print`, `Discard` |  | `product` |
| `stock.product_label_layout_form_picking` | xpath | `product.product_label_layout_form` | `move_quantity` |  |  | `stock` |
| `stock.product_label_layout_form_stock` | xpath | `product.product_label_layout_form` | `zpl_template`, `zpl_preview` |  |  | `stock` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `product.action_open_label_layout` | Print Labels |  |  |  | new | `product` |

Machine-readable definition: `../../../schemas/data/entities/product.label.layout.json`; views: `../../../schemas/interfaces/views/product.label.layout.json`.
