# Choose the sheet layout to print lot labels (`lot.label.layout`)

**Transport name:** `lot.label.layout`  
**Storage name:** `lot_label_layout`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`

Description: Choose the sheet layout to print lot labels

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `move_line_ids` | Move Line | many to many | `stock.move.line` |  |
| `label_quantity` | Quantity to print | selection |  | required; default `lots`; Help: If the UoM of a lot is not 'units', the lot will be considered as a unit and only one label will be printed for this lot. |
| `print_format` | Format | selection |  | required; default `4x12` |

## Selection values

### `label_quantity` (Quantity to print)

| Value | Label |
|---|---|
| `lots` | One per lot/SN |
| `units` | One per unit |

### `print_format` (Format)

| Value | Label |
|---|---|
| `4x12` | 4 x 12 |
| `zpl` | ZPL Labels |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `process` | operation | self | `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.lot_label_layout_form_picking` | form |  | `label_quantity`, `print_format` | `Confirm`, `Cancel` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/lot.label.layout.json`; views: `../../../schemas/interfaces/views/lot.label.layout.json`.
