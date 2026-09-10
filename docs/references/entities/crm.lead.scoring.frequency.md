# Lead Scoring Frequency (`crm.lead.scoring.frequency`)

**Transport name:** `crm.lead.scoring.frequency`  
**Storage name:** `crm_lead_scoring_frequency`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: Lead Scoring Frequency

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `variable` | Variable | single line text |  | indexed |
| `value` | Value | single line text |  |  |
| `won_count` | Won Count | float |  | precision `[16, 1]` |
| `lost_count` | Lost Count | float |  | precision `[16, 1]` |
| `team_id` | Sales Team | many to one | `crm.team` | on delete of the target: cascade |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `base.group_system` | no | yes | no | no | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.scoring.frequency.json`.
