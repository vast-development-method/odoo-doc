# Lead forward to partner (`crm.lead.forward.to.partner`)

**Transport name:** `crm.lead.forward.to.partner`  
**Storage name:** `crm_lead_forward_to_partner`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `website_crm_partner_assign`

Description: Lead forward to partner

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `forward_type` | Forward selected leads to | selection |  | default computed dynamically (lambda self: self.env.context.get('forward_type') or 'single') |
| `partner_id` | Forward Leads To | many to one | `res.partner` |  |
| `assignation_lines` | Partner Assignment | one to many | `crm.lead.assignation` | inverse field `forward_id` |
| `body` | Contents | rich text |  | Help: Automatically sanitized HTML contents |

## Selection values

### `forward_type` (Forward selected leads to)

| Value | Label |
|---|---|
| `single` | a single partner: manual selection of partner |
| `assigned` | several partners: automatic assignment, using GPS coordinates and partner's grades |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_convert_to_assignation_line` | internal rule | self, lead, partner | `website_crm_partner_assign` | model |  |
| `default_get` | lifecycle override | self, fields | `website_crm_partner_assign` | model |  |
| `action_forward` | user action | self | `website_crm_partner_assign` |  |  |
| `get_lead_portal_url` | operation | self, lead | `website_crm_partner_assign` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `action_forward` | UserError | The Forward Email Template is not in the database | `website_crm_partner_assign` |
| `action_forward` | UserError | Set an email address for the partner %s | `website_crm_partner_assign` |
| `action_forward` | UserError | Set an email address for the partner(s): %s | `website_crm_partner_assign` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `website_crm_partner_assign` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.crm_lead_forward_to_partner_form` | form |  | `forward_type`, `partner_id`, `assignation_lines`, `lead_id`, `lead_location`, `partner_assigned_id`, `partner_location`, `lead_link`, `body` | `Send`, `Cancel` |  | `website_crm_partner_assign` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_crm_partner_assign.crm_lead_forward_to_partner_act` | Forward to Partner | form |  |  | new | `website_crm_partner_assign` |
| `website_crm_partner_assign.action_crm_send_mass_forward` | Forward to partner | form |  | `{'default_composition_mode' : 'mass_mail'}` | new | `website_crm_partner_assign` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `website_crm_partner_assign.email_template_lead_forward_mail` | Lead Forward: Send to partner | Fwd: Lead: {{ ctx['partner_id'].name if ctx.get('partner_id') else ''}} |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.forward.to.partner.json`; views: `../../../schemas/interfaces/views/crm.lead.forward.to.partner.json`.
