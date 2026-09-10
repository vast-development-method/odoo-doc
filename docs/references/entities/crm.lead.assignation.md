# Lead Assignation (`crm.lead.assignation`)

**Transport name:** `crm.lead.assignation`  
**Storage name:** `crm_lead_assignation`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website_crm_partner_assign`

Description: Lead Assignation

## Fields (6)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `forward_id` | Partner Assignment | many to one | `crm.lead.forward.to.partner` |  |
| `lead_id` | Lead | many to one | `crm.lead` |  |
| `lead_location` | Lead Location | single line text |  |  |
| `partner_assigned_id` | Assigned Partner | many to one | `res.partner` |  |
| `partner_location` | Partner Location | single line text |  |  |
| `lead_link` | Link to Lead | single line text |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_onchange_lead_id` | on change | self | `website_crm_partner_assign` | onchange: `lead_id` |  |
| `_onchange_partner_assigned_id` | on change | self | `website_crm_partner_assign` | onchange: `partner_assigned_id` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `website_crm_partner_assign` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.assignation.json`.
