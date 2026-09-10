# Point of Sale Daily Report (`pos.daily.sales.reports.wizard`)

**Transport name:** `pos.daily.sales.reports.wizard`  
**Storage name:** `pos_daily_sales_reports_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `point_of_sale`  
**Extended by packages:** `pos_hr`

Description: Point of Sale Daily Report

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `pos_session_id` | Point of sale Session | many to one | `pos.session` | required |
| `add_report_per_employee` | Add a report per each employee | boolean |  | default `True` |
| `employee_ids` | Employee | many to many | `hr.employee` | computed by rule `_compute_employee_ids` (not stored) |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_report_data` | preparation rule | self | `point_of_sale`, `pos_hr` |  |  |
| `generate_report` | operation | self | `point_of_sale` |  |  |
| `_compute_employee_ids` | computation | self | `pos_hr` | depends: `pos_session_id` |  |
| `_onchange_pos_session_id` | on change | self | `pos_hr` | onchange: `pos_session_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_pos_manager` | yes | yes | yes | no | `point_of_sale` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.view_pos_daily_sales_reports_wizard` | form |  | `pos_session_id` | `Print`, `Cancel` |  | `point_of_sale` |
| `pos_hr.view_pos_daily_sales_reports_wizard` | xpath | `point_of_sale.view_pos_daily_sales_reports_wizard` |  |  |  | `pos_hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `point_of_sale.action_report_pos_daily_sales_reports` | Session Report | form |  |  | new | `point_of_sale` |

Machine-readable definition: `../../../schemas/data/entities/pos.daily.sales.reports.wizard.json`; views: `../../../schemas/interfaces/views/pos.daily.sales.reports.wizard.json`.
