# customer relationship management Partnership Analysis (`crm.partner.report.assign`)

**Transport name:** `crm.partner.report.assign`  
**Storage name:** `crm_partner_report_assign`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_crm_partner_assign`

Description: CRM Partnership Analysis

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` | read only |
| `grade_id` | Grade | many to one | `res.partner.grade` | read only |
| `activation` | Activation | many to one | `res.partner.activation` | indexed |
| `user_id` | User | many to one | `res.users` | read only |
| `date_review` | Latest Partner Review | date |  |  |
| `date_partnership` | Partnership Date | date |  |  |
| `country_id` | Country | many to one | `res.country` | read only |
| `nbr_opportunities` | # of Opportunity | integer |  | read only |
| `turnover` | Turnover | float |  | read only |
| `date` | Invoice Account Date | date |  | read only |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_table_query` | internal rule | self | `website_crm_partner_assign` |  | CRM Lead Report @param cr: the current row, from the database cursor |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_partner_assign` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| CRM partner assign report: All Assignations | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | True | True | True | True |
| CRM partner assign report: Personal / Global Assignations | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.view_report_crm_partner_assign_filter` | search |  | `user_id`, `grade_id`, `activation` |  | `filter_date_partnership`, `filter_date_review`, `Salesperson`, `Partner`, `Date Partnership`, `Date Review` | `website_crm_partner_assign` |
| `website_crm_partner_assign.view_report_crm_partner_assign_graph` | graph |  | `grade_id`, `nbr_opportunities`, `turnover` |  |  | `website_crm_partner_assign` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.action_report_crm_partner_assign` | Partnership Analysis | graph | `[('grade_id', '!=', False)]` | `{'group_by':[]}` |  | `website_crm_partner_assign` |

Machine-readable definition: `../../../schemas/data/entities/crm.partner.report.assign.json`; views: `../../../schemas/interfaces/views/crm.partner.report.assign.json`.
