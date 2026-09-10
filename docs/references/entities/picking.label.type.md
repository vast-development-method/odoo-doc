# Choose whether to print product or lot/sn labels (`picking.label.type`)

**Transport name:** `picking.label.type`  
**Storage name:** `picking_label_type`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `stock`  
**Extended by packages:** `mrp`

Description: Choose whether to print product or lot/sn labels

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `picking_ids` | Picking | many to many | `stock.picking` |  |
| `label_type` | Labels to print | selection |  | required; default `products` |
| `production_ids` | Production | many to many | `mrp.production` |  |

## Selection values

### `label_type` (Labels to print)

| Value | Label |
|---|---|
| `products` | Product Labels |
| `lots` | Lot/SN Labels |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `process` | operation | self | `mrp`, `stock` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `stock.picking_label_type_form` | form |  | `label_type` | `Confirm`, `Cancel` |  | `stock` |

Machine-readable definition: `../../../schemas/data/entities/picking.label.type.json`; views: `../../../schemas/interfaces/views/picking.label.type.json`.
