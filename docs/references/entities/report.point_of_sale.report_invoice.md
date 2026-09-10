# Point of Sale Invoice Report (`report.point_of_sale.report_invoice`)

**Transport name:** `report.point_of_sale.report_invoice`  
**Storage name:** `report_point_of_sale_report_invoice`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `point_of_sale`

Description: Point of Sale Invoice Report

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_report_values` | preparation rule | self, docids, data | `point_of_sale` | model |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_report_values` | UserError | No link to an invoice for %s. | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/report.point_of_sale.report_invoice.json`.
