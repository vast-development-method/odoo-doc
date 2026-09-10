# Accounting Report External Value (`account.report.external.value`)

**Transport name:** `account.report.external.value`  
**Storage name:** `account_report_external_value`  
**Kind:** persistent entity (one table)  
**Defined by package:** `account`

Description: Accounting Report External Value

## Identity and behavior

- Default ordering: `date, id`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `value` | Numeric Value | float |  |  |
| `text_value` | Text Value | single line text |  |  |
| `date` | Date | date |  | required |
| `target_report_expression_id` | Target Expression | many to one | `account.report.expression` | required; on delete of the target: cascade |
| `target_report_line_id` | Target Line | many to one |  | related through path `target_report_expression_id.report_line_id` |
| `target_report_expression_label` | Target Expression Label | single line text |  | related through path `target_report_expression_id.label` |
| `report_country_id` | Country | many to one |  | related through path `target_report_line_id.report_id.country_id` |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `carryover_origin_expression_label` | Origin Expression Label | single line text |  |  |
| `carryover_origin_report_line_id` | Origin Line | many to one | `account.report.line` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `account.group_account_readonly` | no | yes | no | no | `account` |
| `account.group_account_manager` | yes | yes | yes | yes | `account` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Report External Value multi-company | global (all users) | `[('company_id', 'in', company_ids)]` | True | True | True | True |

Machine-readable definition: `../../../schemas/data/entities/account.report.external.value.json`.
