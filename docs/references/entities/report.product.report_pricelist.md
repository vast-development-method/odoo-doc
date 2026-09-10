# Pricelist Report (`report.product.report_pricelist`)

**Transport name:** `report.product.report_pricelist`  
**Storage name:** `report_product_report_pricelist`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `product`

Description: Pricelist Report

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_report_values` | preparation rule | self, docids, data | `product` |  |  |
| `get_html` | operation | self, data | `product` | readonly; model |  |
| `_get_report_data` | preparation rule | self, data, report_type | `product` |  |  |
| `_get_product_data` | preparation rule | self, is_product_tmpl, product, pricelist, quantities | `product` |  |  |

Machine-readable definition: `../../../schemas/data/entities/report.product.report_pricelist.json`.
