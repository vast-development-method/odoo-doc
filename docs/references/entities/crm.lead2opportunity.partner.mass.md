# Convert Lead to Opportunity (in mass) (`crm.lead2opportunity.partner.mass`)

**Transport name:** `crm.lead2opportunity.partner.mass`  
**Storage name:** `crm_lead2opportunity_partner_mass`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `crm`

Description: Convert Lead to Opportunity (in mass)

## Identity and behavior

- Mixins (classical inheritance): `crm.lead2opportunity.partner`

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `lead_id` | Lead | many to one |  |  |
| `lead_tomerge_ids` | Active Leads | many to many | `crm.lead` | default computed dynamically (lambda self: self.env.context.get('active_ids', [])); association table `crm_convert_lead_mass_lead_rel` |
| `user_ids` | Salespersons | many to many | `res.users` |  |
| `deduplicate` | Apply deduplication | boolean |  | default `True`; Help: Merge with existing leads/opportunities of each partner |
| `action` | Related Customer | selection |  | on delete of the target: {"expression": "{'each_exist_or_create': lambda recs: recs.write({'action': 'exist'})}"} |
| `force_assignment` | Force Assignment | boolean |  | default  |

## Operations (9)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_name` | computation | self | `crm` | depends: `duplicated_lead_ids` |  |
| `_compute_action` | computation | self | `crm` | depends: `lead_tomerge_ids` |  |
| `_compute_partner_id` | computation | self | `crm` | depends: `lead_tomerge_ids` |  |
| `_compute_commercial_partner_id` | computation | self | `crm` |  | Setting a company for each lead in mass mode is not supported. |
| `_compute_team_id` | computation | self | `crm` | depends: `user_ids` | When changing the user, also set a team_id or restrict team id to the ones user_id is member of. |
| `_compute_duplicated_lead_ids` | computation | self | `crm` | depends: `lead_tomerge_ids` |  |
| `_convert_and_allocate` | internal rule | self, leads, user_ids, team_id | `crm` |  | When "massively" (more than one at a time) converting leads to opportunities, check the salesteam_id and salesmen_ids and update the values before calling super. |
| `action_mass_convert` | user action | self | `crm` |  |  |
| `_convert_handle_partner` | internal rule | self, lead, action, partner_id | `crm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.view_crm_lead2opportunity_partner_mass` | form |  | `lead_tomerge_ids`, `name`, `deduplicate`, `team_id`, `user_ids`, `force_assignment`, `duplicated_lead_ids`, `create_date`, `name`, `type`, `contact_name`, `country_id`, `email_from`, `stage_id`, `user_id`, `team_id`, `action`, `partner_id` | `Convert to Opportunities`, `Cancel` |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.action_crm_send_mass_convert` | Convert to opportunities | form |  | `{}` | new | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead2opportunity.partner.mass.json`; views: `../../../schemas/interfaces/views/crm.lead2opportunity.partner.mass.json`.
