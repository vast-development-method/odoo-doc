# Marketing Card Campaign Tag (`card.campaign.tag`)

**Transport name:** `card.campaign.tag`  
**Storage name:** `card_campaign_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `marketing_card`

Description: Marketing Card Campaign Tag

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique(name)` | Tags may not reuse existing names. | `marketing_card` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `marketing_card` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `marketing_card.marketing_card_group_user` | no | yes | no | no | `marketing_card` |
| `marketing_card.marketing_card_group_manager` | yes | yes | yes | yes | `marketing_card` |

Machine-readable definition: `../../../schemas/data/entities/card.campaign.tag.json`.
