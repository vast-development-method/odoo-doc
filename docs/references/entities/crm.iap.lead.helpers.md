# Helper methods for crm_iap_mine modules (`crm.iap.lead.helpers`)

**Transport name:** `crm.iap.lead.helpers`  
**Storage name:** `crm_iap_lead_helpers`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm_iap_mine`

Description: Helper methods for crm_iap_mine modules

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_notify_no_more_credit` | internal rule | self, service_name, model_name, notification_parameter | `crm_iap_mine` | model | Notify about the number of credit. In order to avoid to spam people each hour, an ir.config_parameter is set |
| `lead_vals_from_response` | operation | self, lead_type, team_id, tag_ids, user_id, company_data, people_data | `crm_iap_mine` | model |  |
| `_find_state_id` | internal rule | self, state_code, country_id | `crm_iap_mine` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `crm_iap_mine` |

Machine-readable definition: `../../../schemas/data/entities/crm.iap.lead.helpers.json`.
