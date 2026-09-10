# customer relationship management Lead Generation Rules (`crm.reveal.rule`)

**Transport name:** `crm.reveal.rule`  
**Storage name:** `crm_reveal_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_crm_iap_reveal`

Description: CRM Lead Generation Rules

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Rule Name | single line text |  | required |
| `active` | Active | boolean |  | default `True` |
| `country_ids` | Countries | many to many | `res.country` | Help: Only visitors of following countries will be converted into leads/opportunities (using GeoIP). |
| `website_id` | Website | many to one | `website` | Help: Restrict Lead generation to this website. |
| `state_ids` | States | many to many | `res.country.state` | Help: Only visitors of following states will be converted into leads/opportunities. |
| `regex_url` | uniform resource locator Expression | single line text |  | Help: Regex to track website pages. Leave empty to track the entire website, or / to target the homepage. Example: /page* to track all the pages which begin with /page |
| `sequence` | Sequence | integer |  | Help: Used to order the rules with same URL and countries. Rules with a lower sequence number will be processed first. |
| `industry_tag_ids` | Industries | many to many | `crm.iap.lead.industry` | Help: Leave empty to always match. Odoo will not create lead if no match |
| `filter_on_size` | Filter on Size | boolean |  | default `True`; Help: Filter companies based on their size. |
| `company_size_min` | Company Size | integer |  | default  |
| `company_size_max` | Company Size Max | integer |  | default `1000` |
| `contact_filter_type` | Filter On | selection |  | required; default `role` |
| `preferred_role_id` | Preferred Role | many to one | `crm.iap.lead.role` |  |
| `other_role_ids` | Other Roles | many to many | `crm.iap.lead.role` |  |
| `seniority_id` | Seniority | many to one | `crm.iap.lead.seniority` |  |
| `extra_contacts` | Number of Contacts | integer |  | default `1`; Help: This is the number of contacts to track if their role/seniority match your criteria. Their details will show up in the history thread of generated leads/opportunities. One credit is consumed per tracked contact. |
| `lead_for` | Data Tracking | selection |  | required; default `companies`; Help: Choose whether to track companies only or companies and their contacts |
| `lead_type` | Type | selection |  | required; default `opportunity` |
| `suffix` | Suffix | single line text |  | Help: This will be appended in name of generated lead so you can identify lead/opportunity is generated with this rule |
| `team_id` | Sales Team | many to one | `crm.team` | on delete of the target: set null |
| `tag_ids` | Tags | many to many | `crm.tag` |  |
| `user_id` | Salesperson | many to one | `res.users` |  |
| `priority` | Priority | selection |  |  |
| `lead_ids` | Generated Lead / Opportunity | one to many | `crm.lead` | inverse field `reveal_rule_id` |
| `lead_count` | Number of Generated Leads | integer |  | computed by rule `_compute_lead_count` (not stored) |
| `opportunity_count` | Number of Generated Opportunity | integer |  | computed by rule `_compute_lead_count` (not stored) |

## Selection values

### `contact_filter_type` (Filter On)

| Value | Label |
|---|---|
| `role` | Role |
| `seniority` | Seniority |

### `lead_for` (Data Tracking)

| Value | Label |
|---|---|
| `companies` | Companies |
| `people` | Companies and their Contacts |

### `lead_type` (Type)

| Value | Label |
|---|---|
| `lead` | Lead |
| `opportunity` | Opportunity |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_limit_extra_contacts` | Constraint | `check(extra_contacts >= 1 and extra_contacts <= 5)` | Maximum 5 contacts are allowed! | `website_crm_iap_reveal` |

## Operations (19)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_lead_count` | computation | self | `website_crm_iap_reveal` |  |  |
| `_check_regex_url` | validation | self | `website_crm_iap_reveal` | constrains: `regex_url` |  |
| `create` | lifecycle override | self, vals_list | `website_crm_iap_reveal` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_crm_iap_reveal` |  |  |
| `unlink` | lifecycle override | self | `website_crm_iap_reveal` |  |  |
| `action_get_lead_tree_view` | user action | self | `website_crm_iap_reveal` |  |  |
| `action_get_opportunity_tree_view` | user action | self | `website_crm_iap_reveal` |  |  |
| `_get_active_rules` | preparation rule | self | `website_crm_iap_reveal` | model | Returns informations about the all rules. The return is in the form : {     'country_rules': {         'BE': [0, 1],         'US': [0]     },     'rules': [     {         'id': 0,         'regex': ***,         'website_id': 1,         'country_codes': ['BE', 'US'],         'state_codes': [('BE', False), ('US', 'NY'), ('US', 'CA')]     },     {         'id': 1,         'regex': ***,         'website_id': 1,         'country_codes': ['BE'],         'state_codes': [('BE', False)]     }     ] } |
| `_add_to_country` | internal rule | self, country_rules, country, rule_index | `website_crm_iap_reveal` |  | Add the rule index to the country code in the country_rules |
| `_match_url` | internal rule | self, website_id, url, country_code, state_code, rules_excluded | `website_crm_iap_reveal` |  | Return the matching rule based on the country, the website and URL. |
| `_process_lead_generation` | background operation | self, autocommit | `website_crm_iap_reveal` | model | Cron Job for lead generation from page view |
| `_unlink_unrelevant_reveal_view` | internal rule | self | `website_crm_iap_reveal` | model | We don't want to create the lead if in past (<6 months) we already created lead with given IP. So, we unlink crm.reveal.view with same IP as a already created lead. |
| `_get_reveal_views_to_process` | preparation rule | self | `website_crm_iap_reveal` | model | Return list of reveal rule ids grouped by IPs |
| `_prepare_iap_payload` | preparation rule | self, pgv | `website_crm_iap_reveal` |  | This will prepare the page view and returns payload Payload sample {     ips: {         '192.168.1.1': [1,4],         '192.168.1.6': [2,4]     },     rules: {         1: {rule_data},         2: {rule_data},         4: {rule_data}     } } |
| `_get_rules_payload` | preparation rule | self | `website_crm_iap_reveal` |  |  |
| `_perform_reveal_service` | internal rule | self, server_payload | `website_crm_iap_reveal` |  |  |
| `_iap_contact_reveal` | internal rule | self, params, timeout | `website_crm_iap_reveal` |  |  |
| `_create_lead_from_response` | internal rule | self, result | `website_crm_iap_reveal` |  | This method will get response from service and create the lead accordingly |
| `_lead_vals_from_response` | internal rule | self, result | `website_crm_iap_reveal` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_regex_url` | ValidationError | Enter Valid Regex. | `website_crm_iap_reveal` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_crm_iap_reveal` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_iap_reveal` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| CRM Reveal Rules: All Rules | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | True | True | True | True |
| CRM Reveal Rules: Personal / Global Rules | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('user_id', '=', user.id), ('user_id', '=', False)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_rule_form` | form |  | `opportunity_count`, `lead_count`, `name`, `lead_for`, `active`, `country_ids`, `website_id`, `state_ids`, `regex_url`, `sequence`, `extra_contacts`, `industry_tag_ids`, `filter_on_size`, `company_size_min`, `company_size_max`, `extra_contacts`, `contact_filter_type`, `preferred_role_id`, `other_role_ids`, `seniority_id`, `lead_type`, `lead_type`, `suffix`, `team_id`, `user_id`, `tag_ids`, `priority` | `action_get_opportunity_tree_view`, `action_get_lead_tree_view` |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_rule_tree` | list |  | `sequence`, `name`, `lead_type` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_rule_view_search` | search |  | `name` |  | `Archived` | `website_crm_iap_reveal` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_rule_action` | Visits to Leads Rules | list,form |  |  |  | `website_crm_iap_reveal` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `website_crm_iap_reveal.ir_cron_crm_reveal_lead` | Lead Generation: Leads/Opportunities Generation | 1 days | `_process_lead_generation` |  |

Machine-readable definition: `../../../schemas/data/entities/crm.reveal.rule.json`; views: `../../../schemas/interfaces/views/crm.reveal.rule.json`.
