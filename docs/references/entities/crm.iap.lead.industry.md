# customer relationship management in-app purchase Lead Industry (`crm.iap.lead.industry`)

**Transport name:** `crm.iap.lead.industry`  
**Storage name:** `crm_iap_lead_industry`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm_iap_mine`

Description: CRM IAP Lead Industry

## Identity and behavior

- Default ordering: `sequence,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Industry | single line text |  | required; translatable |
| `reveal_ids` | Reveal | single line text |  | required |
| `color` | Color Index | integer |  |  |
| `sequence` | Sequence | integer |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Industry name already exists! | `crm_iap_mine` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |

Machine-readable definition: `../../../schemas/data/entities/crm.iap.lead.industry.json`.
