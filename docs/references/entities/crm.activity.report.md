# customer relationship management Activity Analysis (`crm.activity.report`)

**Transport name:** `crm.activity.report`  
**Storage name:** `crm_activity_report`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: CRM Activity Analysis

## Identity and behavior

- Display name field: `id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (20)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `date` | Completion Date | date and time |  | read only |
| `lead_create_date` | Creation Date | date and time |  | read only |
| `date_conversion` | Conversion Date | date and time |  | read only |
| `date_deadline` | Expected Closing | date |  | read only |
| `date_closed` | Closed Date | date and time |  | read only |
| `author_id` | Assigned To | many to one | `res.partner` | read only |
| `user_id` | Salesperson | many to one | `res.users` | read only |
| `team_id` | Sales Team | many to one | `crm.team` | read only |
| `lead_id` | Opportunity | many to one | `crm.lead` | read only |
| `body` | Activity Description | rich text |  | read only |
| `subtype_id` | Subtype | many to one | `mail.message.subtype` | read only |
| `mail_activity_type_id` | Activity Type | many to one | `mail.activity.type` | read only |
| `country_id` | Country | many to one | `res.country` | read only |
| `company_id` | Company | many to one | `res.company` | read only |
| `stage_id` | Stage | many to one | `crm.stage` | read only |
| `partner_id` | Customer | many to one | `res.partner` | read only |
| `lead_type` | Type | selection |  | Help: Type is used to separate Leads and Opportunities |
| `active` | Active | boolean |  | read only |
| `tag_ids` | Tag | many to many |  | read only; related through path `lead_id.tag_ids` |
| `won_status` | Is Won | selection |  | read only |

## Selection values

### `lead_type` (Type)

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

### `won_status` (Is Won)

| Value | Label |
|---|---|
| `won` | Won |
| `lost` | Lost |
| `pending` | Pending |

## State fields

State machine fields of this entity: `won_status`. Transitions are specified in the domain documents.

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_select` | internal rule | self | `crm` |  |  |
| `_from` | internal rule | self | `crm` |  |  |
| `_join` | internal rule | self | `crm` |  |  |
| `_where` | internal rule | self | `crm` |  |  |
| `init` | lifecycle override | self | `crm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | no | no | no | `crm` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| All Activities | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |
| Personal Activities | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|',('user_id','=',user.id),('user_id','=',False)]` | True | True | True | True |
| CRM Lead Multi-Company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_activity_report_view_graph` | graph |  | `mail_activity_type_id`, `date` |  |  | `crm` |
| `crm.crm_activity_report_view_pivot` | pivot |  | `mail_activity_type_id`, `date` |  |  | `crm` |
| `crm.crm_activity_report_view_tree` | list |  | `date`, `author_id`, `mail_activity_type_id`, `body`, `company_id`, `tag_ids` |  |  | `crm` |
| `crm.crm_activity_report_view_search` | search |  | `mail_activity_type_id`, `lead_id`, `user_id`, `team_id`, `author_id`, `tag_ids` |  | `Leads`, `Opportunities`, `Won`, `Lost`, `Trailing 12 months`, `filter_date`, `Archived`, `Activity`, `Type`, `Assigned To`, `Completion Date`, `Salesperson`, `Sales Team`, `Stage`, `Company`, `Creation Date`, `Expected Closing`, `Closed Date` | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_activity_report_action` | Activities | graph,pivot,list | `[]` | `{                 'search_default_completion_date': 1,                 'pivot_column_groupby': ['subtype_id', 'mail_activity_type_id'],                 'pivot_row_groupby': ['date:month'],                 'graph_mode': 'bar',                 'graph_groupbys': ['date:month', 'subtype_id'],             }` |  | `crm` |
| `crm.crm_activity_report_action_team` | Pipeline Activities | graph,pivot,list | `[]` | `{'search_default_team_id': active_id}` |  | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.activity.report.json`; views: `../../../schemas/interfaces/views/crm.activity.report.json`.
