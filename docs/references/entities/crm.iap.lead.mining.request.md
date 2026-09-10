# customer relationship management Lead Mining Request (`crm.iap.lead.mining.request`)

**Transport name:** `crm.iap.lead.mining.request`  
**Storage name:** `crm_iap_lead_mining_request`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm_iap_mine`

Description: CRM Lead Mining Request

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Request Number | single line text |  | required; read only; default computed dynamically (lambda self: _('New')); not copied on duplication |
| `state` | Status | selection |  | required; default `draft` |
| `lead_number` | Number of Leads | integer |  | required; default `3` |
| `search_type` | Target | selection |  | required; default `companies` |
| `error_type` | Error Type | selection |  | read only; not copied on duplication |
| `lead_type` | Type | selection |  | required; default computed dynamically (_default_lead_type) |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored; on delete of the target: set null; restricted by domain `[('use_opportunities', '=', True)]` |
| `user_id` | Salesperson | many to one | `res.users` | default computed dynamically (lambda self: self.env.user) |
| `tag_ids` | Tags | many to many | `crm.tag` |  |
| `lead_ids` | Generated Lead / Opportunity | one to many | `crm.lead` | inverse field `lead_mining_request_id` |
| `lead_count` | Number of Generated Leads | integer |  | computed by rule `_compute_lead_count` (not stored) |
| `filter_on_size` | Filter on Size | boolean |  | default  |
| `company_size_min` | Size | integer |  | default `1` |
| `company_size_max` | Company Size Max | integer |  | default `1000` |
| `country_ids` | Countries | many to many | `res.country` | default computed dynamically (_default_country_ids) |
| `state_ids` | States | many to many | `res.country.state` |  |
| `available_state_ids` | Available State | one to many | `res.country.state` | computed by rule `_compute_available_state_ids` (not stored) |
| `industry_ids` | Industries | many to many | `crm.iap.lead.industry` |  |
| `contact_number` | Number of Contacts | integer |  | default `10` |
| `contact_filter_type` | Filter on | selection |  | default `role` |
| `preferred_role_id` | Preferred Role | many to one | `crm.iap.lead.role` |  |
| `role_ids` | Other Roles | many to many | `crm.iap.lead.role` |  |
| `seniority_id` | Seniority | many to one | `crm.iap.lead.seniority` |  |
| `lead_credits` | Lead Credits | single line text |  | read only; computed by rule `_compute_tooltip` (not stored) |
| `lead_contacts_credits` | Lead Contacts Credits | single line text |  | read only; computed by rule `_compute_tooltip` (not stored) |
| `lead_total_credits` | Lead Total Credits | single line text |  | read only; computed by rule `_compute_tooltip` (not stored) |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `draft` | Draft |
| `error` | Error |
| `done` | Done |

### `search_type` (Target)

| Value | Label |
|---|---|
| `companies` | Companies |
| `people` | Companies and their Contacts |

### `error_type` (Error Type)

| Value | Label |
|---|---|
| `credits` | Insufficient Credits |
| `no_result` | No Result |

### `lead_type` (Type)

| Value | Label |
|---|---|
| `lead` | Leads |
| `opportunity` | Opportunities |

### `contact_filter_type` (Filter on)

| Value | Label |
|---|---|
| `role` | Role |
| `seniority` | Seniority |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_lead_type` | preparation rule | self | `crm_iap_mine` |  |  |
| `_default_country_ids` | preparation rule | self | `crm_iap_mine` |  |  |
| `_compute_tooltip` | on change | self | `crm_iap_mine` | onchange: `lead_number`, `contact_number` |  |
| `_compute_lead_count` | computation | self | `crm_iap_mine` | depends: `lead_ids.lead_mining_request_id` |  |
| `_compute_team_id` | computation | self | `crm_iap_mine` | depends: `user_id`, `lead_type` | When changing the user, also set a team_id or restrict team id to the ones user_id is member of. |
| `_compute_available_state_ids` | computation | self | `crm_iap_mine` | depends: `country_ids` | States for some specific countries should not be offered as filtering options because they drastically reduce the amount of IAP reveal results.  For example, in Belgium, only 11% of companies have a defined state within the reveal service while the rest of them have no state defined at all.  Meaning specifying states for that country will yield a lot less results than what you could expect, which is not the desired behavior. Obviously all companies are active within a state, it's just a lack of data in the reveal service side.  To help users create meaningful iap searches, we only keep the sta |
| `_onchange_available_state_ids` | on change | self | `crm_iap_mine` | onchange: `available_state_ids` |  |
| `_onchange_lead_number` | on change | self | `crm_iap_mine` | onchange: `lead_number` |  |
| `_onchange_contact_number` | on change | self | `crm_iap_mine` | onchange: `contact_number` |  |
| `_onchange_country_ids` | on change | self | `crm_iap_mine` | onchange: `country_ids` |  |
| `_onchange_company_size_min` | on change | self | `crm_iap_mine` | onchange: `company_size_min` |  |
| `_onchange_company_size_max` | on change | self | `crm_iap_mine` | onchange: `company_size_max` |  |
| `get_empty_list_help` | operation | self, help_message | `crm_iap_mine` | model |  |
| `_prepare_iap_payload` | preparation rule | self | `crm_iap_mine` |  | This will prepare the data to send to the server |
| `_perform_request` | internal rule | self | `crm_iap_mine` |  | This will perform the request and create the corresponding leads. The user will be notified if they don't have enough credits. |
| `_iap_contact_mining` | internal rule | self, params, timeout | `crm_iap_mine` |  |  |
| `_create_leads_from_response` | internal rule | self, result | `crm_iap_mine` |  | This method will get the response from the service and create the leads accordingly |
| `_lead_vals_from_response` | internal rule | self, data | `crm_iap_mine` | model |  |
| `action_draft` | user action | self | `crm_iap_mine` |  |  |
| `action_submit` | user action | self | `crm_iap_mine` |  |  |
| `action_get_lead_action` | user action | self | `crm_iap_mine` |  |  |
| `action_get_opportunity_action` | user action | self | `crm_iap_mine` |  |  |
| `action_buy_credits` | user action | self | `crm_iap_mine` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_perform_request` | UserError | Your request could not be executed: %s | `crm_iap_mine` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm_iap_mine.crm_iap_lead_mining_request_view_form` | form |  | `state`, `error_type`, `lead_type`, `lead_count`, `lead_count`, `name`, `lead_number`, `search_type`, `country_ids`, `available_state_ids`, `state_ids`, `industry_ids`, `filter_on_size`, `company_size_min`, `company_size_max`, `lead_type`, `team_id`, `user_id`, `tag_ids`, `contact_number`, `contact_filter_type`, `preferred_role_id`, `role_ids`, `seniority_id` | `Submit`, `Retry`, `action_buy_credits`, `action_get_opportunity_action`, `action_get_lead_action`, `Generate Leads`, `Cancel` |  | `crm_iap_mine` |
| `crm_iap_mine.crm_iap_lead_mining_request_view_tree` | list |  | `name`, `lead_number`, `search_type`, `country_ids`, `industry_ids`, `team_id`, `user_id`, `tag_ids`, `state` |  |  | `crm_iap_mine` |
| `crm_iap_mine.crm_iap_lead_mining_request_view_search` | search |  | `name`, `industry_ids`, `country_ids`, `team_id`, `user_id`, `tag_ids` |  | `Draft`, `Done`, `Error`, `Leads`, `Opportunities`, `Type`, `Sales Team`, `Salesperson` | `crm_iap_mine` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm_iap_mine.crm_iap_lead_mining_request_action` | Lead Mining Requests | list,form |  |  |  | `crm_iap_mine` |

Machine-readable definition: `../../../schemas/data/entities/crm.iap.lead.mining.request.json`; views: `../../../schemas/interfaces/views/crm.iap.lead.mining.request.json`.
