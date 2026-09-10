# Marketing Card Template (`card.template`)

**Transport name:** `card.template`  
**Storage name:** `card_template`  
**Kind:** persistent entity (one table)  
**Defined by package:** `marketing_card`

Description: Marketing Card Template

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `default_background` | Default Background | image |  |  |
| `body` | Body | rich text |  |  |
| `primary_color` | Primary Color | single line text |  | required; default `#f9f9f9` |
| `secondary_color` | Secondary Color | single line text |  | required; default `#000000` |
| `primary_text_color` | Primary Text Color | single line text |  | required; default `#000000` |
| `secondary_text_color` | Secondary Text Color | single line text |  | required; default `#ffffff` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `marketing_card.marketing_card_group_user` | no | yes | no | no | `marketing_card` |
| `base.group_system` | yes | yes | yes | yes | `marketing_card` |

## Views (1)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `marketing_card.card_template_view_form` | form |  | `name`, `default_background`, `primary_color`, `secondary_color`, `primary_text_color`, `secondary_text_color`, `body` |  |  | `marketing_card` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `marketing_card.card_template_action` | Card Template | list,form |  |  |  | `marketing_card` |

Machine-readable definition: `../../../schemas/data/entities/card.template.json`; views: `../../../schemas/interfaces/views/card.template.json`.
