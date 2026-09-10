# Marketing Card (`card.card`)

**Transport name:** `card.card`  
**Storage name:** `card_card`  
**Kind:** persistent entity (one table)  
**Defined by package:** `marketing_card`

Description: Marketing Card

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (7)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `campaign_id` | Campaign | many to one | `card.campaign` | required; indexed; on delete of the target: cascade |
| `res_model` | Resource Model | selection |  | related through path `campaign_id.res_model` |
| `res_id` | Record identifier | many to one by reference |  | required |
| `image` | Image | image |  |  |
| `requires_sync` | Requires Sync | boolean |  | default `True`; Help: Whether the image needs to be updated to match the campaign template. |
| `share_status` | Share Status | selection |  |  |

## Selection values

### `share_status` (Share Status)

| Value | Label |
|---|---|
| `shared` | Shared |
| `visited` | Visited |

## State fields

State machine fields of this entity: `share_status`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_campaign_record_unique` | Constraint | `unique(campaign_id, res_id)` | Each record should be unique for a campaign | `marketing_card` |

## Operations (6)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_display_name` | computation | self | `marketing_card` | depends: `res_model`, `res_id` |  |
| `_compute_res_model` | computation | self | `marketing_card` | depends: `campaign_id` | Compute the res_model once and never update it again. |
| `_gc_card` | background operation | self | `marketing_card` | autovacuum | Remove cards. Social networks are expected to cache the images on their side. |
| `_get_card_url` | preparation rule | self | `marketing_card` |  |  |
| `_get_redirect_url` | preparation rule | self | `marketing_card` |  |  |
| `_get_path` | preparation rule | self, suffix | `marketing_card` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `marketing_card` |
| `base.group_portal` | no | yes | no | no | `marketing_card` |
| `base.group_public` | no | yes | no | no | `marketing_card` |
| `marketing_card.marketing_card_group_manager` | yes | yes | yes | yes | `marketing_card` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `marketing_card.card_card_view_list` | list |  | `create_date`, `create_uid`, `display_name`, `res_model`, `campaign_id`, `share_status` |  |  | `marketing_card` |
| `marketing_card.card_card_view_search` | search |  | `share_status`, `campaign_id` |  | `Shared`, `Visited`, `Campaign` | `marketing_card` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `marketing_card.cards_card_action` | Card | list |  | `{'search_default_by_campaign': True, 'search_default_filter_visited': True}` |  | `marketing_card` |

Machine-readable definition: `../../../schemas/data/entities/card.card.json`; views: `../../../schemas/interfaces/views/card.card.json`.
