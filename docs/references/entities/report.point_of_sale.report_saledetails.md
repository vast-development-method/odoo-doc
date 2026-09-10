# Point of Sale Details (`report.point_of_sale.report_saledetails`)

**Transport name:** `report.point_of_sale.report_saledetails`  
**Storage name:** `report_point_of_sale_report_saledetails`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `point_of_sale`

Description: Point of Sale Details

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_date_start_and_date_stop` | preparation rule | self, date_start, date_stop | `point_of_sale` |  |  |
| `_get_domain` | preparation rule | self, date_start, date_stop, config_ids, session_ids | `point_of_sale` |  |  |
| `get_sale_details` | operation | self, date_start, date_stop, config_ids, session_ids, **kwargs | `point_of_sale` | model | Serialise the orders of the requested time period, configs and sessions. :param date_start: The dateTime to start, default today 00:00:00. :type date_start: str. :param date_stop: The dateTime to stop, default date_start + 23:59:59. :type date_stop: str. :param config_ids: Pos Config id's to include. :type config_ids: list of numbers. :param session_ids: Pos Config id's to include. :type session_ids: list of numbers. :returns: dict -- Serialised sales. |
| `_get_product_total_amount` | preparation rule | self, line | `point_of_sale` |  |  |
| `_get_products_and_taxes_dict` | preparation rule | self, line, products, taxes, currency | `point_of_sale` |  |  |
| `_get_total_and_qty_per_category` | preparation rule | self, categories | `point_of_sale` |  |  |
| `_prepare_get_sale_details_args_kwargs` | preparation rule | self, data | `point_of_sale` |  |  |
| `_get_report_values` | preparation rule | self, docids, data | `point_of_sale` | model |  |
| `_get_taxes_info` | preparation rule | self, taxes | `point_of_sale` |  |  |

Machine-readable definition: `../../../schemas/data/entities/report.point_of_sale.report_saledetails.json`.
