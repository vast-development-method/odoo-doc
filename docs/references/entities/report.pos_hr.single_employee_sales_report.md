# Session sales details for a single employee (`report.pos_hr.single_employee_sales_report`)

**Transport name:** `report.pos_hr.single_employee_sales_report`  
**Storage name:** `report_pos_hr_single_employee_sales_report`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `pos_hr`

Description: Session sales details for a single employee

## Identity and behavior

- Mixins (classical inheritance): `report.point_of_sale.report_saledetails`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_domain` | preparation rule | self, date_start, date_stop, config_ids, session_ids, employee_id | `pos_hr` |  |  |
| `_prepare_get_sale_details_args_kwargs` | preparation rule | self, data | `pos_hr` |  |  |
| `get_sale_details` | operation | self, date_start, date_stop, config_ids, session_ids, employee_id | `pos_hr` | model |  |

Machine-readable definition: `../../../schemas/data/entities/report.pos_hr.single_employee_sales_report.json`.
