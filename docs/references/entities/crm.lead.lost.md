# Get Lost Reason (`crm.lead.lost`)

**Transport name:** `crm.lead.lost`  
**Storage name:** `crm_lead_lost`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `crm`

Description: Get Lost Reason

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `lead_ids` | Leads | many to many | `crm.lead` |  |
| `lost_reason_id` | Lost Reason | many to one | `crm.lost.reason` |  |
| `lost_feedback` | Closing Note | rich text |  |  |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_lost_reason_apply` | user action | self | `crm` |  | Mark lead as lost and apply the loss reason |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_lost_view_form` | form |  | `lead_ids`, `lost_reason_id`, `lost_feedback` | `Mark as Lost`, `Discard` |  | `crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `crm.crm_lead_lost_action` | Mark Lost | form |  | `{                 'dialog_size' : 'medium',                 'default_lead_ids': active_ids,             }` | new | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.lost.json`; views: `../../../schemas/interfaces/views/crm.lead.lost.json`.
