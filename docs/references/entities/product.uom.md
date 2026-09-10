# Link between products and their UoMs (`product.uom`)

**Transport name:** `product.uom`  
**Storage name:** `product_uom`  
**Kind:** persistent entity (one table)  
**Defined by package:** `product`  
**Extended by packages:** `point_of_sale`

Description: Link between products and their UoMs

## Identity and behavior

- Mixins (classical inheritance): `pos.load.mixin`
- Display name field: `barcode`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `uom_id` | Unit | many to one | `uom.uom` | required; indexed; on delete of the target: cascade |
| `product_id` | Product | many to one | `product.product` | required; indexed; on delete of the target: cascade |
| `barcode` | Barcode | single line text |  | required; indexed (btree_not_null); not copied on duplication |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_barcode_uniq` | Constraint | `unique(barcode)` | A barcode can only be assigned to one packaging. | `product` |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_barcode_uniqueness` | validation | self | `product` | constrains: `barcode` | With GS1 nomenclature, products and packagings use the same pattern. Therefore, we need to ensure the uniqueness between products' barcodes and packagings' ones |
| `_compute_display_name` | computation | self | `product` |  |  |
| `_load_pos_data_fields` | internal rule | self, config | `point_of_sale` | model |  |
| `_load_pos_data_domain` | internal rule | self, data, config | `point_of_sale` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_barcode_uniqueness` | ValidationError | A product already uses the barcode | `product` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_product_manager` | yes | yes | yes | yes | `product` |
| `base.group_user` | no | yes | no | no | `product` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `product.product_uom_list_view` | list |  | `product_id`, `barcode`, `uom_id` |  |  | `product` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `product.report_product_packaging` | Packaging Barcodes (PDF) | qweb-pdf | `product.report_packagingbarcode` | `'Products packaging - %s' % (object.uom_id.name)` |  |
| `stock.label_packaging_barcode` | Packaging Barcodes (ZPL) | qweb-text | `stock.label_packaging_barcode_view` |  |  |

Machine-readable definition: `../../../schemas/data/entities/product.uom.json`; views: `../../../schemas/interfaces/views/product.uom.json`.
