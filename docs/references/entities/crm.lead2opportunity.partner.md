# Convert Lead to Opportunity (not in mass) (`crm.lead2opportunity.partner`)

**Transport name:** `crm.lead2opportunity.partner`  
**Storage name:** `crm_lead2opportunity_partner`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `crm`

Description: Convert Lead to Opportunity (not in mass)

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Conversion Action | selection |  | computed by rule `_compute_name` and stored |
| `action` | Related Customer | selection |  | required; computed by rule `_compute_action` and stored; precomputed before insertion |
| `lead_id` | Associated Lead | many to one | `crm.lead` | required |
| `lead_partner_name` | Lead Partner Name | single line text |  | related through path `lead_id.partner_name` |
| `lead_contact_name` | Lead Contact Name | single line text |  | related through path `lead_id.contact_name` |
| `duplicated_lead_ids` | Opportunities | many to many | `crm.lead` | computed by rule `_compute_duplicated_lead_ids` and stored |
| `commercial_partner_id` | Company | many to one | `res.partner` | computed by rule `_compute_commercial_partner_id` and stored; restricted by domain `[["is_company", "=", true]]` |
| `partner_id` | Customer | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored |
| `user_id` | Salesperson | many to one | `res.users` | computed by rule `_compute_user_id` and stored |
| `team_id` | Sales Team | many to one | `crm.team` | computed by rule `_compute_team_id` and stored |
| `force_assignment` | Force assignment | boolean |  | default `True`; Help: If checked, forces salesman to be updated on updated opportunities even if already set. |

## Selection values

### `name` (Conversion Action)

| Value | Label |
|---|---|
| `convert` | Convert to opportunity |
| `merge` | Merge with existing opportunities |

### `action` (Related Customer)

| Value | Label |
|---|---|
| `create` | Create a new customer |
| `exist` | Link to an existing customer |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `crm` | model | Allow support of active_id / active_model instead of jut default_lead_id to ease window action definitions, and be backward compatible. |
| `_compute_name` | computation | self | `crm` | depends: `duplicated_lead_ids` |  |
| `_compute_action` | computation | self | `crm` | depends: `lead_id` |  |
| `_compute_duplicated_lead_ids` | computation | self | `crm` | depends: `lead_id`, `partner_id` |  |
| `_compute_commercial_partner_id` | computation | self | `crm` | depends: `partner_id` |  |
| `_compute_partner_id` | computation | self | `crm` | depends: `action`, `lead_id` |  |
| `_compute_user_id` | computation | self | `crm` | depends: `lead_id` |  |
| `_compute_team_id` | computation | self | `crm` | depends: `user_id` | When changing the user, also set a team_id or restrict team id to the ones user_id is member of. |
| `action_apply` | user action | self | `crm` |  |  |
| `_action_merge` | internal rule | self | `crm` |  |  |
| `_action_convert` | internal rule | self | `crm` |  |  |
| `_convert_and_allocate` | internal rule | self, leads, user_ids, team_id | `crm` |  |  |
| `_convert_handle_partner` | internal rule | self, lead, action, partner_id | `crm` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `default_get` | UserError | Closed/Dead leads cannot be converted into opportunities. | `crm` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.view_crm_lead2opportunity_partner` | form |  | `name`, `user_id`, `team_id`, `lead_id`, `duplicated_lead_ids`, `create_date`, `name`, `type`, `contact_name`, `country_id`, `email_from`, `stage_id`, `user_id`, `team_id`, `action`, `commercial_partner_id`, `lead_partner_name`, `partner_id` | `Create Opportunity`, `Cancel` |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.action_crm_lead2opportunity_partner` | Convert to opportunity | form |  |  | new | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead2opportunity.partner.json`; views: `../../../schemas/interfaces/views/crm.lead2opportunity.partner.json`.
