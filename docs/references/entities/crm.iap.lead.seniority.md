# People Seniority (`crm.iap.lead.seniority`)

**Transport name:** `crm.iap.lead.seniority`  
**Storage name:** `crm_iap_lead_seniority`  
**Kind:** persistent entity (one table)  
**Defined by package:** `crm_iap_mine`

Description: People Seniority

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `reveal_id` | Reveal | single line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Name already exists! | `crm_iap_mine` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `crm_iap_mine` | depends: `name` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm_iap_mine` |

Machine-readable definition: `../../../schemas/data/entities/crm.iap.lead.seniority.json`.
