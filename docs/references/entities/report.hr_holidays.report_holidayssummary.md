# Holidays Summary Report (`report.hr_holidays.report_holidayssummary`)

**Transport name:** `report.hr_holidays.report_holidayssummary`  
**Storage name:** `report_hr_holidays_report_holidayssummary`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `hr_holidays`

Description: Holidays Summary Report

## Operations (10)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_header_info` | preparation rule | self, start_date, holiday_type | `hr_holidays` |  |  |
| `_date_is_day_off` | internal rule | self, date | `hr_holidays` |  |  |
| `_get_day` | preparation rule | self, start_date | `hr_holidays` |  |  |
| `_get_months` | preparation rule | self, start_date | `hr_holidays` |  |  |
| `_get_leaves_summary` | preparation rule | self, start_date, empid, holiday_type | `hr_holidays` |  |  |
| `_get_employees` | preparation rule | self, data | `hr_holidays` |  |  |
| `_get_data_from_report` | preparation rule | self, data | `hr_holidays` |  |  |
| `_get_leaves` | preparation rule | self, date_from, employees, holiday_type, date_to | `hr_holidays` |  |  |
| `_get_holidays_status` | preparation rule | self, data | `hr_holidays` |  |  |
| `_get_report_values` | preparation rule | self, docids, data | `hr_holidays` | model |  |

Machine-readable definition: `../../../schemas/data/entities/report.hr_holidays.report_holidayssummary.json`.
