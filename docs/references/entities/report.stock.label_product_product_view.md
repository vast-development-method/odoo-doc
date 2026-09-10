# Product Label Report (`report.stock.label_product_product_view`)

**Transport name:** `report.stock.label_product_product_view`  
**Storage name:** `report_stock_label_product_product_view`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `stock`

Description: Product Label Report

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_report_values` | preparation rule | self, docids, data | `stock` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_report_values` | UserError | Product model not defined, Please contact your administrator. | `stock` |

Machine-readable definition: `../../../schemas/data/entities/report.stock.label_product_product_view.json`.
