# Digest (`digest.digest`)

**Transport name:** `digest.digest`  
**Storage name:** `digest_digest`  
**Kind:** persistent entity (one table)  
**Defined by package:** `digest`  
**Extended by packages:** `account`, `crm`, `im_livechat`, `sale_management`, `hr_recruitment`, `project`, `point_of_sale`, `website_sale`

Description: Digest

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (35)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `user_ids` | Recipients | many to many | `res.users` | restricted by domain `[('share', '=', False)]` |
| `periodicity` | Periodicity | selection |  | required; default `daily` |
| `next_run_date` | Next Mailing Date | date |  |  |
| `currency_id` | Currency | many to one |  | related through path `company_id.currency_id` |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company.id) |
| `available_fields` | Available Fields | single line text |  | computed by rule `_compute_available_fields` (not stored) |
| `is_subscribed` | Is user subscribed | boolean |  | computed by rule `_compute_is_subscribed` (not stored) |
| `state` | Status | selection |  | read only; default `activated` |
| `kpi_res_users_connected` | Connected Users | boolean |  |  |
| `kpi_res_users_connected_value` | Key performance indicator Resource Users Connected Value | integer |  | computed by rule `_compute_kpi_res_users_connected_value` (not stored) |
| `kpi_mail_message_total` | Messages Sent | boolean |  |  |
| `kpi_mail_message_total_value` | Key performance indicator Mail Message Total Value | integer |  | computed by rule `_compute_kpi_mail_message_total_value` (not stored) |
| `kpi_account_total_revenue` | Revenue | boolean |  |  |
| `kpi_account_total_revenue_value` | Key performance indicator Account Total Revenue Value | monetary |  | computed by rule `_compute_kpi_account_total_revenue_value` (not stored) |
| `kpi_crm_lead_created` | New Leads | boolean |  |  |
| `kpi_crm_lead_created_value` | Key performance indicator Customer relationship management Lead Created Value | integer |  | computed by rule `_compute_kpi_crm_lead_created_value` (not stored) |
| `kpi_crm_opportunities_won` | Opportunities Won | boolean |  |  |
| `kpi_crm_opportunities_won_value` | Key performance indicator Customer relationship management Opportunities Won Value | integer |  | computed by rule `_compute_kpi_crm_opportunities_won_value` (not stored) |
| `kpi_livechat_rating` | % of Happiness | boolean |  |  |
| `kpi_livechat_rating_value` | Key performance indicator Livechat Rating Value | float |  | computed by rule `_compute_kpi_livechat_rating_value` (not stored); precision `[16, 2]` |
| `kpi_livechat_conversations` | Conversations handled | boolean |  |  |
| `kpi_livechat_conversations_value` | Key performance indicator Livechat Conversations Value | integer |  | computed by rule `_compute_kpi_livechat_conversations_value` (not stored) |
| `kpi_livechat_response` | Time to answer (sec) | boolean |  |  |
| `kpi_livechat_response_value` | Key performance indicator Livechat Response Value | float |  | computed by rule `_compute_kpi_livechat_response_value` (not stored); precision `[16, 2]` |
| `kpi_all_sale_total` | All Sales | boolean |  |  |
| `kpi_all_sale_total_value` | Key performance indicator All Sale Total Value | monetary |  | computed by rule `_compute_kpi_sale_total_value` (not stored) |
| `kpi_hr_recruitment_new_colleagues` | New Employees | boolean |  |  |
| `kpi_hr_recruitment_new_colleagues_value` | Key performance indicator Human resources Recruitment New Colleagues Value | integer |  | computed by rule `_compute_kpi_hr_recruitment_new_colleagues_value` (not stored) |
| `kpi_project_task_opened` | Open Tasks | boolean |  |  |
| `kpi_project_task_opened_value` | Key performance indicator Project Task Opened Value | integer |  | computed by rule `_compute_project_task_opened_value` (not stored) |
| `kpi_pos_total` | point of sale Sales | boolean |  |  |
| `kpi_pos_total_value` | Key performance indicator Point of sale Total Value | monetary |  | computed by rule `_compute_kpi_pos_total_value` (not stored) |
| `kpi_website_sale_total` | eCommerce Sales | boolean |  |  |
| `kpi_website_sale_total_value` | Key performance indicator Website Sale Total Value | monetary |  | computed by rule `_compute_kpi_website_sale_total_value` (not stored) |

## Selection values

### `periodicity` (Periodicity)

| Value | Label |
|---|---|
| `daily` | Daily |
| `weekly` | Weekly |
| `monthly` | Monthly |
| `quarterly` | Quarterly |

### `state` (Status)

| Value | Label |
|---|---|
| `activated` | Activated |
| `deactivated` | Deactivated |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (44)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_is_subscribed` | computation | self | `digest` | depends: `user_ids` |  |
| `_compute_available_fields` | computation | self | `digest` |  |  |
| `_get_kpi_compute_parameters` | preparation rule | self | `digest` |  | Get the parameters used to computed the KPI value. |
| `_compute_kpi_res_users_connected_value` | computation | self | `digest` |  |  |
| `_compute_kpi_mail_message_total_value` | computation | self | `digest` |  |  |
| `_onchange_periodicity` | on change | self | `digest` | onchange: `periodicity` |  |
| `create` | lifecycle override | self, vals_list | `digest` | model_create_multi |  |
| `action_subscribe` | user action | self | `digest` |  |  |
| `_action_subscribe_users` | internal rule | self, users | `digest` |  | Private method to manage subscriptions. Done as sudo() to speedup computation and avoid ACLs issues. |
| `action_unsubscribe` | user action | self | `digest` |  |  |
| `_action_unsubscribe_users` | internal rule | self, users | `digest` |  | Private method to manage subscriptions. Done as sudo() to speedup computation and avoid ACLs issues. |
| `action_activate` | user action | self | `digest` |  |  |
| `action_deactivate` | user action | self | `digest` |  |  |
| `action_set_periodicity` | user action | self, periodicity | `digest` |  |  |
| `action_send` | user action | self | `digest` |  | Send digests emails to all the registered users. |
| `action_send_manual` | user action | self | `digest` |  | Manually send digests emails to all registered users. In that case do not update periodicity as this is not an automation rule that could be considered as unwanted spam. |
| `_action_send` | internal rule | self, update_periodicity | `digest` |  | Send digests email to all the registered users.  :param bool update_periodicity: if True, check user logs to update   periodicity of digests. Purpose is to slow down digest whose users   do not connect to avoid spam; |
| `_action_send_to_user` | internal rule | self, user, tips_count, consume_tips | `digest` |  |  |
| `_cron_send_digest_email` | background operation | self | `digest` | model |  |
| `_get_unsubscribe_token` | preparation rule | self, user_id | `digest` |  | Generate a secure hash for this digest and user. It allows to unsubscribe from a digest while keeping some security in that process.  :param int user_id: ID of the user to unsubscribe |
| `_compute_kpis` | computation | self, company, user | `digest` |  | Compute KPIs to display in the digest template. It is expected to be a list of KPIs, each containing values for 3 columns display.  :return: result [{     'kpi_name': 'kpi_mail_message',     'kpi_fullname': 'Messages',  # translated     'kpi_action': 'crm.crm_lead_action_pipeline',  # xml id of an action to execute     'kpi_col1': {         'value': '12.0',         'margin': 32.36,         'col_subtitle': 'Yesterday',  # translated     },     'kpi_col2': { ... },     'kpi_col3':  { ... }, }, { ... }] |
| `_compute_tips` | computation | self, company, user, tips_count, consumed | `digest` |  |  |
| `_compute_kpis_actions` | computation | self, company, user | `account`, `crm`, `digest`, `hr_recruitment`, `im_livechat`, `point_of_sale`, `project`, `sale_management`, `website_sale` |  | Give an optional action to display in digest email linked to some KPIs.  :returns: key: kpi name (field name), value: an action that will be   concatenated with /app/action-{action} :rtype: dict |
| `_compute_preferences` | computation | self, company, user | `digest` |  | Give an optional text for preferences, like a shortcut for configuration.  :returns: html to put in template :rtype: str |
| `_get_next_run_date` | preparation rule | self | `digest` |  |  |
| `_compute_timeframes` | computation | self, company | `digest` |  |  |
| `_calculate_company_based_kpi` | internal rule | self, model, digest_kpi_field, date_field, additional_domain, sum_field | `digest` |  | Generic method that computes the KPI on a given model.  :param model: Model on which we will compute the KPI     This model must have a "company_id" field :param digest_kpi_field: Field name on which we will write the KPI :param date_field: Field used for the date range :param additional_domain: Additional domain :param sum_field: Field to sum to obtain the KPI,     if None it will count the number of records |
| `_get_company_field` | preparation rule | self, model | `digest` |  |  |
| `_get_kpi_fields` | preparation rule | self | `digest` |  |  |
| `_get_margin_value` | preparation rule | self, value, previous_value | `digest` |  |  |
| `_check_daily_logs` | validation | self | `digest` |  | Badly named method that checks user logs and slowdown the sending of digest emails based on recipients being away. |
| `_get_next_periodicity` | preparation rule | self | `digest` |  |  |
| `_format_currency_amount` | internal rule | self, amount, currency_id | `digest` |  |  |
| `_compute_kpi_account_total_revenue_value` | computation | self | `account` |  |  |
| `_compute_kpi_crm_lead_created_value` | computation | self | `crm` |  |  |
| `_compute_kpi_crm_opportunities_won_value` | computation | self | `crm` |  |  |
| `_compute_kpi_livechat_rating_value` | computation | self | `im_livechat` |  |  |
| `_compute_kpi_livechat_conversations_value` | computation | self | `im_livechat` |  |  |
| `_compute_kpi_livechat_response_value` | computation | self | `im_livechat` |  |  |
| `_compute_kpi_sale_total_value` | computation | self | `sale_management` |  |  |
| `_compute_kpi_hr_recruitment_new_colleagues_value` | computation | self | `hr_recruitment` |  |  |
| `_compute_project_task_opened_value` | computation | self | `project` |  |  |
| `_compute_kpi_pos_total_value` | computation | self | `point_of_sale` |  |  |
| `_compute_kpi_website_sale_total_value` | computation | self | `website_sale` |  |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_compute_kpi_account_total_revenue_value` | AccessError | Do not have access, skip this data for user's digest email | `account` |
| `_compute_kpi_crm_lead_created_value` | AccessError | Do not have access, skip this data for user's digest email | `crm` |
| `_compute_kpi_crm_opportunities_won_value` | AccessError | Do not have access, skip this data for user's digest email | `crm` |
| `_compute_kpi_sale_total_value` | AccessError | Do not have access, skip this data for user's digest email | `sale_management` |
| `_compute_kpi_hr_recruitment_new_colleagues_value` | AccessError | Do not have access, skip this data for user's digest email | `hr_recruitment` |
| `_compute_project_task_opened_value` | AccessError | Do not have access, skip this data for user's digest email | `project` |
| `_compute_kpi_pos_total_value` | AccessError | Do not have access, skip this data for user's digest email | `point_of_sale` |
| `_compute_kpi_website_sale_total_value` | AccessError | Do not have access, skip this data for user's digest email | `website_sale` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_erp_manager` | yes | yes | yes | yes | `digest` |
| `base.group_user` | no | yes | no | no | `digest` |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.digest_digest_view_form` | xpath | `digest.digest_digest_view_form` | `kpi_account_total_revenue` |  |  | `account` |
| `crm.digest_digest_view_form` | xpath | `digest.digest_digest_view_form` | `kpi_crm_lead_created`, `kpi_crm_opportunities_won` |  |  | `crm` |
| `digest.digest_digest_view_tree` | list |  | `name`, `periodicity`, `next_run_date`, `company_id`, `state` |  |  | `digest` |
| `digest.digest_digest_view_form` | form |  | `is_subscribed`, `state`, `name`, `periodicity`, `company_id`, `next_run_date`, `kpi_res_users_connected`, `kpi_mail_message_total`, `user_ids`, `name`, `email` | `Send Now`, `Deactivate`, `Activate` |  | `digest` |
| `digest.digest_digest_view_search` | search |  | `name`, `user_ids` |  | `Activated`, `Deactivated`, `Periodicity` | `digest` |
| `hr_recruitment.digest_digest_view_form` | xpath | `digest.digest_digest_view_form` | `kpi_hr_recruitment_new_colleagues` |  |  | `hr_recruitment` |
| `im_livechat.digest_digest_view_form_inherit` | xpath | `digest.digest_digest_view_form` | `kpi_livechat_rating`, `kpi_livechat_conversations`, `kpi_livechat_response` |  |  | `im_livechat` |
| `point_of_sale.digest_digest_view_form` | xpath | `digest.digest_digest_view_form` | `kpi_pos_total` |  |  | `point_of_sale` |
| `project.digest_digest_view_form` | xpath | `digest.digest_digest_view_form` | `kpi_project_task_opened` |  |  | `project` |
| `sale_management.digest_digest_view_form` | group | `digest.digest_digest_view_form` |  |  |  | `sale_management` |
| `website_sale.digest_digest_view_form` | group | `digest.digest_digest_view_form` |  |  |  | `website_sale` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `digest.digest_digest_action` | Digest Emails |  |  | `{'search_default_filter_activated': 1}` |  | `digest` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `digest.ir_cron_digest_scheduler_action` | Digest Emails | 1 days | `_cron_send_digest_email` |  |

Machine-readable definition: `../../../schemas/data/entities/digest.digest.json`; views: `../../../schemas/interfaces/views/digest.digest.json`.
