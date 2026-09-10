# People Role (`crm.iap.lead.role`)

**Transport name:** `crm.iap.lead.role`  
**Storage name:** `crm_iap_lead_role`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm_iap_mine`

Description: People Role

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Role Name | single line text |  | required; translatable |
| `reveal_id` | Reveal | single line text |  | required |
| `color` | Color Index | integer |  |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Role name already exists! | `crm_iap_mine` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `crm_iap_mine` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |

Machine-readable definition: `../../../schemas/data/entities/crm.iap.lead.role.json`.
