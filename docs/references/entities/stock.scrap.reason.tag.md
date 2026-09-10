# Scrap Reason Tag (`stock.scrap.reason.tag`)

**Transport name:** `stock.scrap.reason.tag`  
**Storage name:** `stock_scrap_reason_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `stock`

Description: Scrap Reason Tag

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `color` | Color | single line text |  | default `#3C3C3C` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | Tag name already exists! | `stock` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `stock.group_stock_user` | yes | yes | yes | no | `stock` |
| `stock.group_stock_manager` | yes | yes | yes | yes | `stock` |

Machine-readable definition: `../../../schemas/data/entities/stock.scrap.reason.tag.json`.
