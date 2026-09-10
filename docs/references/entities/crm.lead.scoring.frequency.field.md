# Fields that can be used for predictive lead scoring computation (`crm.lead.scoring.frequency.field`)

**Transport name:** `crm.lead.scoring.frequency.field`  
**Storage name:** `crm_lead_scoring_frequency_field`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm`

Description: Fields that can be used for predictive lead scoring computation

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | related through path `field_id.field_description` |
| `field_id` | Field | many to one | `ir.model.fields` | required; on delete of the target: cascade; restricted by domain `[["model_id.model", "=", "crm.lead"]]` |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `crm` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_salesman` | no | yes | no | no | `crm` |
| `base.group_system` | no | yes | no | no | `crm` |

Machine-readable definition: `../../../schemas/data/entities/crm.lead.scoring.frequency.field.json`.
