# Salary Structure Type (`hr.payroll.structure.type`)

**Transport name:** `hr.payroll.structure.type`  
**Storage name:** `hr_payroll_structure_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`

Description: Salary Structure Type

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Salary Structure Type | single line text |  |  |
| `default_resource_calendar_id` | Working Hours | many to one | `resource.calendar` | default computed dynamically (lambda self: self.env.company.resource_calendar_id) |
| `country_id` | Country | many to one | `res.country` | default computed dynamically (lambda self: self.env.company.country_id); restricted by domain `lambda self: [('id', 'in', self.env.companies.country_id.ids)]` |
| `country_code` | Country Code | single line text |  | related through path `country_id.code` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_manager` | yes | yes | yes | yes | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR Payroll Structure Type: Multi Company | global (all users) | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.mapped('country_id').ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/hr.payroll.structure.type.json`.
