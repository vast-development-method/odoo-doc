# Sales Team (`crm.team`)

**Transport name:** `crm.team`  
**Storage name:** `crm_team`  
**Kind:** persistent entity (one table)  
**Defined by package:** `sales_team`  
**Extended by packages:** `crm`, `sale`, `sale_crm`, `website_sale`, `pos_sale`, `survey_crm`

Description: Sales Team

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.alias.mixin`
- Default ordering: `sequence ASC, create_date DESC, id DESC`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (36)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Sales Team | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `active` | Active | boolean |  | default `True`; Help: If the active field is set to false, it will allow you to hide the Sales Team without removing it. |
| `company_id` | Company | many to one | `res.company` | indexed |
| `currency_id` | Currency | many to one | `res.currency` | read only; related through path `company_id.currency_id` |
| `user_id` | Team Leader | many to one | `res.users` | restricted by domain `[["share", "!=", true]]`; must belong to the same company |
| `is_membership_multi` | Multiple Memberships Allowed | boolean |  | computed by rule `_compute_is_membership_multi` (not stored); Help: If True, users may belong to several sales teams. Otherwise membership is limited to a single sales team. |
| `member_ids` | Salespersons | many to many | `res.users` | computed by rule `_compute_member_ids` (not stored); writable through an inverse rule; searchable through a search rule; restricted by domain `['&', ('share', '=', False), ('company_ids', 'in', member_company_ids)]`; Help: Users assigned to this team. |
| `member_company_ids` | Member Company | many to many | `res.company` | computed by rule `_compute_member_company_ids` (not stored); Help: UX: Limit to team company or all if no company |
| `member_warning` | Membership Issue Warning | multi line text |  | computed by rule `_compute_member_warning` (not stored) |
| `crm_team_member_ids` | Sales Team Members | one to many | `crm.team.member` | inverse field `crm_team_id`; Help: Add members to automatically assign their documents to this sales team. |
| `crm_team_member_all_ids` | Sales Team Members (incl. inactive) | one to many | `crm.team.member` | inverse field `crm_team_id` |
| `color` | Color Index | integer |  | default computed dynamically (_get_default_color); Help: The color of the channel |
| `favorite_user_ids` | Favorite Members | many to many | `res.users` | default computed dynamically (_get_default_favorite_user_ids); association table `team_favorite_user_rel` |
| `is_favorite` | Show on dashboard | boolean |  | computed by rule `_compute_is_favorite` (not stored); writable through an inverse rule; Help: Favorite teams to display them in the dashboard and access them easily. |
| `dashboard_button_name` | Dashboard Button | single line text |  | computed by rule `_compute_dashboard_button_name` (not stored) |
| `use_leads` | Leads | boolean |  | Help: Check this box to filter and qualify incoming requests as leads before converting them into opportunities and assigning them to a salesperson. |
| `use_opportunities` | Pipeline | boolean |  | default `True`; Help: Check this box to manage a presales process with opportunities. |
| `alias_id` | Alias | many to one |  | Help: The email address associated with this channel. New emails received will automatically create new leads assigned to the channel. |
| `assignment_enabled` | Lead Assign | boolean |  | computed by rule `_compute_assignment_enabled` (not stored) |
| `assignment_auto_enabled` | Auto Assignment | boolean |  | computed by rule `_compute_assignment_enabled` (not stored) |
| `assignment_optout` | Skip auto assignment | boolean |  |  |
| `assignment_max` | Lead Average Capacity | integer |  | computed by rule `_compute_assignment_max` (not stored); Help: Monthly average leads capacity for all salesmen belonging to the team |
| `assignment_domain` | Assignment Domain | single line text |  | changes are tracked in the message thread; Help: Additional filter domain when fetching unassigned leads to allocate to the team. |
| `lead_unassigned_count` | # Unassigned Leads | integer |  | computed by rule `_compute_lead_unassigned_count` (not stored) |
| `lead_all_assigned_month_count` | # Leads/Opps assigned this month | integer |  | computed by rule `_compute_lead_all_assigned_month_count` (not stored); Help: Number of leads and opportunities assigned this last month. |
| `lead_all_assigned_month_exceeded` | Exceed monthly lead assignement | boolean |  | computed by rule `_compute_lead_all_assigned_month_count` (not stored); Help: True if the monthly lead assignment count is greater than the maximum assignment limit, false otherwise. |
| `lead_properties_definition` | Lead Properties | properties definition |  |  |
| `invoiced` | Invoiced This Month | float |  | read only; computed by rule `_compute_invoiced` (not stored); Help: Invoice revenue for the current month. This is the amount the sales channel has invoiced this month. It is used to compute the progression ratio of the current and target revenue on the kanban view. |
| `invoiced_target` | Invoicing Target | float |  | Help: Revenue Target for the current month (untaxed total of paid invoices). |
| `sale_order_count` | # Sale Orders | integer |  | computed by rule `_compute_sale_order_count` (not stored) |
| `website_ids` | Websites | one to many | `website` | inverse field `salesteam_id` |
| `abandoned_carts_amount` | Amount of Abandoned Carts | integer |  | computed by rule `_compute_abandoned_carts` (not stored) |
| `abandoned_carts_count` | Number of Abandoned Carts | integer |  | computed by rule `_compute_abandoned_carts` (not stored) |
| `pos_config_ids` | Point of Sales | one to many | `pos.config` | inverse field `crm_team_id` |
| `origin_survey_ids` | Survey opportunities related to the sales team | one to many | `survey.survey` | inverse field `team_id` |

## Operations (46)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `sales_team` |  |  |
| `_get_default_team_id` | preparation rule | self, user_id, domain | `sales_team` |  | Compute default team id for sales related documents. Note that this method is not called by default_get as it takes some additional parameters and is meant to be called by other default methods.  Heuristic (when multiple match: take from default context value or first sequence ordered)    1- any of my teams (member OR responsible) matching domain, either from      context or based on _order;   2- any of my teams (member OR responsible), either from context or based      on _order;   3- default from context   4- any team matching my company and domain (based on company rule)   5- any team match |
| `_get_default_favorite_user_ids` | preparation rule | self | `sales_team` |  |  |
| `_constrains_company_members` | validation | self | `sales_team` | constrains: `company_id` |  |
| `_compute_is_membership_multi` | computation | self | `sales_team` | depends: `sequence` |  |
| `_compute_member_ids` | computation | self | `sales_team` | depends: `crm_team_member_ids.active` |  |
| `_inverse_member_ids` | inverse computation | self | `sales_team` |  |  |
| `_compute_member_warning` | computation | self | `sales_team` | depends: `is_membership_multi`, `member_ids` | Display a warning message to warn user they are about to archive other memberships. Only valid in mono-membership mode and take into account only active memberships as we may keep several archived memberships. |
| `_search_member_ids` | search rule | self, operator, value | `sales_team` |  |  |
| `_compute_member_company_ids` | computation | self | `sales_team` | depends: `company_id`, `name` | Available companies for members. Either team company if set, either any company if not set on team. |
| `_compute_is_favorite` | computation | self | `sales_team` |  |  |
| `_inverse_is_favorite` | inverse computation | self | `sales_team` |  |  |
| `_compute_dashboard_button_name` | computation | self | `crm`, `sale_crm`, `sale`, `sales_team` |  | Sets the adequate dashboard button name depending on the Sales Team's options |
| `create` | lifecycle override | self, vals_list | `sales_team` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `crm`, `sales_team` |  |  |
| `_unlink_except_default` | internal rule | self | `sales_team` | ondelete |  |
| `action_primary_channel_button` | user action | self | `crm`, `sale_crm`, `sale`, `sales_team` |  | Skeleton function to be overloaded It will return the adequate action depending on the Sales Team's options. |
| `_add_members_to_favorites` | internal rule | self | `sales_team` |  |  |
| `_compute_assignment_max` | computation | self | `crm` | depends: `crm_team_member_ids.assignment_max` |  |
| `_compute_assignment_enabled` | computation | self | `crm` |  |  |
| `_compute_lead_unassigned_count` | computation | self | `crm` |  |  |
| `_compute_lead_all_assigned_month_count` | computation | self | `crm` | depends: `crm_team_member_ids.lead_month_count`, `assignment_max` |  |
| `_onchange_use_leads_opportunities` | on change | self | `crm` | onchange: `use_leads`, `use_opportunities` |  |
| `_constrains_assignment_domain` | validation | self | `crm` | constrains: `assignment_domain` |  |
| `unlink` | lifecycle override | self | `crm` |  | When unlinking, concatenate `crm.lead.scoring.frequency` linked to the team into "no team" statistics. |
| `_alias_get_creation_values` | internal rule | self | `crm` |  |  |
| `_cron_assign_leads` | background operation | self, force_quota, creation_delta_days | `crm` | model | Cron method assigning leads. Leads are allocated to all teams and assigned to their members.  The cron is designed to run at least once a day or more. A number of leads will be assigned each time depending on the daily leads already assigned. This allows the assignment process based on the cron to work on a daily basis without allocating too much leads on members if the cron is executed multiple times a day. The daily quota of leads can be forcefully assigned with force_quota (ignoring the daily leads already assigned).  See `CrmTeam.action_assign_leads()` and its sub methods for more detail |
| `action_assign_leads` | user action | self | `crm` |  | Manual (direct) leads assignment. This method both    * assigns leads to teams given by self;   * assigns leads to salespersons belonging to self;  See sub methods for more details about assign process.  :returns: action, a client notification giving some insights on assign   process; |
| `_action_assign_leads` | internal rule | self, force_quota, creation_delta_days | `crm` |  | Private method for lead assignment. This method both    * assigns leads to teams given by self;   * assigns leads to salespersons belonging to self;  See sub methods for more details about assign process.  :param bool force_quota: Assign the full daily quota without taking into account                          the leads already assigned today :param int creation_delta_days: Take into account all leads created in the last nb days (by default 7).                                 If set to zero we take all the past leads.  :returns: 2-elements tuple (teams_data, members_data) as a   structure-base |
| `_action_assign_leads_logs` | internal rule | self, teams_data, members_data | `crm` |  | Tool method to prepare notification about assignment process result.  :param teams_data: see `CrmTeam._allocate_leads()`; :param members_data: see `CrmTeam._assign_and_convert_leads()`;  :returns: list of formatted logs, ready to be formatted into a nice plaintext or html message at caller's will :rtype: list[str] |
| `_allocate_leads` | internal rule | self, creation_delta_days | `crm` |  | Allocate leads to teams given by self. This method sets `team_id` field on lead records that are unassigned (no team and no responsible). No salesperson is assigned in this process. Its purpose is simply to allocate leads within teams.  This process allocates all available leads on teams weighted by their maximum assignment by month that indicates their relative workload.  Heuristic of this method is the following:   * find unassigned leads for each team, aka leads being     * without team, without user -> not assigned;     * not won nor inactive -> live leads;     * created in the last crea |
| `_allocate_leads_deduplicate` | internal rule | self, leads, duplicates_cache | `crm` |  | Assign leads to sales team given by self by calling lead tool method _handle_salesmen_assignment. In this method we deduplicate leads allowing to reduce number of resulting leads before assigning them to salesmen.  :param leads: recordset of leads to assign to current team; :param duplicates_cache: if given, avoid to perform a duplicate search   and fetch information in it instead; |
| `_get_lead_to_assign_domain` | preparation rule | self | `crm` |  |  |
| `_assign_and_convert_leads` | internal rule | self, force_quota | `crm` |  | Main processing method to assign leads to sales team members. It also converts them into opportunities. This method should be called after `_allocate_leads` as this method assigns leads already allocated to the member's team. Its main purpose is therefore to distribute team workload on its members based on their capacity.  This method follows the following heuristic     * Get quota per member     * Find all leads to be assigned per team     * Sort list of members per number of leads received in the last 24h     * Assign the lead using round robin         * Find the first member with a compat |
| `action_your_pipeline` | user action | self | `crm` | model |  |
| `action_opportunity_forecast` | user action | self | `crm` | model |  |
| `action_open_leads` | user action | self | `crm` |  |  |
| `action_open_unassigned_leads` | user action | self | `crm` |  |  |
| `_action_update_to_pipeline` | internal rule | self, action | `crm` | model |  |
| `_compute_invoiced` | computation | self | `sale` |  |  |
| `_compute_sale_order_count` | computation | self | `sale` |  |  |
| `_in_sale_scope` | internal rule | self | `sale` |  |  |
| `update_invoiced_target` | operation | self, value | `sale` |  |  |
| `_unlink_except_used_for_sales` | internal rule | self | `sale` | ondelete | If more than 5 active SOs, we consider this team to be actively used. 5 is some random guess based on "user testing", aka more than testing CRM feature and less than use it in real life use cases. |
| `_compute_abandoned_carts` | computation | self | `website_sale` |  |  |
| `get_abandoned_carts` | operation | self | `website_sale` |  |  |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_constrains_company_members` | UserError | The following team members are not allowed in company '%(company)s' of the Sales Team '%(team)s': %(users)s | `sales_team` |
| `_unlink_except_default` | UserError | Cannot delete default team "%s" | `sales_team` |
| `_constrains_assignment_domain` | ValidationError | Assignment domain for team %(team)s is incorrectly formatted | `crm` |
| `_action_assign_leads` | UserError | Lead/Opportunities automatic assignment is limited to managers or administrators | `crm` |
| `_unlink_except_used_for_sales` | UserError | Team %(team_name)s has %(sale_order_count)s active sale orders. Consider cancelling them or archiving the team instead. | `sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `point_of_sale.group_pos_manager` | yes | yes | yes | yes | `pos_sale` |
| `base.group_user` | no | yes | no | no | `sales_team` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `sales_team` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sales_team` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| POS Sales Team | `[(4, ref('point_of_sale.group_pos_manager'))]` | `[(1,'=',1)]` | True | True | True | True |
| All Salesteam | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1,'=',1)]` | True | True | True | True |
| Sales Team multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_team_view_tree` | field | `sales_team.crm_team_view_tree` | `name`, `alias_id` |  |  | `crm` |
| `crm.sales_team_form_view_in_crm` | xpath | `sales_team.crm_team_view_form` | `use_leads`, `use_opportunities` | `Assign Leads` |  | `crm` |
| `crm.crm_team_view_kanban_dashboard` | data | `sales_team.crm_team_view_kanban_dashboard` | `alias_name`, `alias_domain`, `use_opportunities`, `use_leads`, `alias_id`, `lead_unassigned_count` |  |  | `crm` |
| `sale.crm_team_salesteams_view_form` | field | `sales_team.crm_team_view_form` | `company_id`, `invoiced_target` |  |  | `sale` |
| `sale.crm_team_view_kanban_dashboard` | xpath | `sales_team.crm_team_view_kanban_dashboard` | `invoiced_target` |  |  | `sale` |
| `sale_crm.crm_team_salesteams_view_form_in_sale_crm` | data | `crm.sales_team_form_view_in_crm` |  |  |  | `sale_crm` |
| `sales_team.crm_team_view_search` | search |  | `name`, `user_id`, `member_ids` |  | `Archived`, `Company`, `Team Leader` | `sales_team` |
| `sales_team.crm_team_view_form` | form |  | `member_warning`, `name`, `active`, `sequence`, `is_membership_multi`, `user_id`, `company_id`, `currency_id`, `member_company_ids`, `member_ids`, `avatar_128`, `name`, `email`, `crm_team_member_ids` | `crm_team_activate_multi_membership` |  | `sales_team` |
| `sales_team.crm_team_view_tree` | list |  | `sequence`, `name`, `active`, `user_id`, `company_id` |  |  | `sales_team` |
| `sales_team.crm_team_view_kanban` | kanban |  | `name`, `user_id` |  |  | `sales_team` |
| `sales_team.crm_team_view_kanban_dashboard` | kanban |  | `color`, `name`, `company_id`, `user_id` |  |  | `sales_team` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `sales_team.crm_team_action_sales` | Sales Teams | kanban,form |  | `{'in_sales_app': True}` |  | `sales_team` |
| `sales_team.crm_team_action_pipeline` | Teams | kanban,form |  | `{}` |  | `sales_team` |
| `sales_team.crm_team_action_config` | Sales Teams | list,form |  | `{}` |  | `sales_team` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.sales_team_menu_team_pipeline` | Teams | `crm_menu_sales` | `sales_team.crm_team_action_pipeline` | 4 |  |
| `crm.crm_team_config` | Sales Teams | `crm_menu_config` | `sales_team.crm_team_action_config` | 5 |  |
| `sale.report_sales_team` | Sales Teams |  | `sales_team.crm_team_action_sales` | 30 | `sales_team.group_sale_manager` |
| `sale.sales_team_config` | Sales Teams |  | `sales_team.crm_team_action_config` | 20 |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `crm.action_your_pipeline` | Crm: My Pipeline | code |  | yes |
| `crm.action_opportunity_forecast` | Crm: Forecast | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `crm.ir_cron_crm_lead_assign` | CRM: Lead Assignment | 1 days | `_cron_assign_leads` |  |

Machine-readable definition: `../../../schemas/data/entities/crm.team.json`; views: `../../../schemas/interfaces/views/crm.team.json`.
