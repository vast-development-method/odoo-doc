# Project Sales line, employee mapping (`project.sale.line.employee.map`)

**Transport name:** `project.sale.line.employee.map`  
**Storage name:** `project_sale_line_employee_map`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sale_timesheet`

Description: Project Sales line, employee mapping

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (13)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `project_id` | Project | many to one | `project.project` | required; indexed; restricted by domain `[["is_template", "=", false]]` |
| `employee_id` | Employee | many to one | `hr.employee` | required; restricted by domain `[('id', 'not in', existing_employee_ids)]` |
| `existing_employee_ids` | Existing Employee | many to many | `hr.employee` | computed by rule `_compute_existing_employee_ids` (not stored) |
| `sale_line_id` | Sales Order Item | many to one | `sale.order.line` | computed by rule `_compute_sale_line_id` and stored; restricted by domain `lambda self: str(self._domain_sale_line_id())` |
| `sale_order_id` | Sale Order | many to one |  | related through path `project_id.sale_order_id` |
| `company_id` | Company | many to one | `res.company` | related through path `project_id.company_id` |
| `partner_id` | Partner | many to one |  | related through path `project_id.partner_id` |
| `price_unit` | Unit Price | float |  | read only; computed by rule `_compute_price_unit` and stored |
| `currency_id` | Currency | many to one | `res.currency` | computed by rule `_compute_currency_id` and stored |
| `cost` | Cost | monetary |  | computed by rule `_compute_cost` and stored; currency taken from `cost_currency_id`; Help: This cost overrides the employee's default employee hourly wage in employee's HR Settings |
| `display_cost` | Hourly Cost | monetary |  | computed by rule `_compute_display_cost` (not stored); writable through an inverse rule; visible only to groups `project.group_project_manager,hr.group_hr_user`; currency taken from `cost_currency_id` |
| `cost_currency_id` | Cost Currency | many to one | `res.currency` | read only; related through path `employee_id.currency_id` |
| `is_cost_changed` | Is Cost Manually Changed | boolean |  | computed by rule `_compute_is_cost_changed` and stored |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_uniqueness_employee` | Constraint | `UNIQUE(project_id,employee_id)` | An employee cannot be selected more than once in the mapping. Please remove duplicate(s) and try again. | `sale_timesheet` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_domain_sale_line_id` | internal rule | self | `sale_timesheet` |  |  |
| `_compute_existing_employee_ids` | computation | self | `sale_timesheet` | depends: `employee_id`, `project_id.sale_line_employee_ids.employee_id` |  |
| `_compute_sale_line_id` | computation | self | `sale_timesheet` | depends: `partner_id` |  |
| `_compute_price_unit` | computation | self | `sale_timesheet` | depends: `sale_line_id.price_unit` |  |
| `_compute_currency_id` | computation | self | `sale_timesheet` | depends: `sale_line_id.price_unit` |  |
| `_compute_cost` | computation | self | `sale_timesheet` | depends: `employee_id.hourly_cost` |  |
| `_get_working_hours_per_calendar` | preparation rule | self, is_uom_day | `sale_timesheet` |  |  |
| `_compute_display_cost` | computation | self | `sale_timesheet` | depends_context: `company`; depends: `cost`, `employee_id.resource_calendar_id` |  |
| `_inverse_display_cost` | inverse computation | self | `sale_timesheet` |  |  |
| `_compute_is_cost_changed` | computation | self | `sale_timesheet` | depends: `cost` |  |
| `create` | lifecycle override | self, vals_list | `sale_timesheet` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `sale_timesheet` |  |  |
| `_update_project_timesheet` | internal rule | self | `sale_timesheet` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `sale_timesheet` |
| `project.group_project_manager` | yes | yes | yes | yes | `sale_timesheet` |

Machine-readable definition: `../../../schemas/data/entities/project.sale.line.employee.map.json`.
