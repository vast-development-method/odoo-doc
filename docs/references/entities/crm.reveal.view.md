# customer relationship management Reveal View (`crm.reveal.view`)

**Transport name:** `crm.reveal.view`  
**Storage name:** `crm_reveal_view`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_crm_iap_reveal`

Description: CRM Reveal View

## Identity and behavior

- Default ordering: `id desc`
- Display name field: `reveal_ip`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `reveal_ip` | internet protocol Address | single line text |  |  |
| `reveal_rule_id` | Lead Generation Rule | many to one | `crm.reveal.rule` | indexed (btree_not_null) |
| `reveal_state` | State | selection |  | default `to_process`; indexed |
| `create_date` | Create Date | date and time |  | indexed |

## Selection values

### `reveal_state` (State)

| Value | Label |
|---|---|
| `to_process` | To Process |
| `not_found` | Not Found |

## State fields

State machine fields of this entity: `reveal_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_ip_rule_id` | UniqueIndex | `(reveal_rule_id,reveal_ip)` |  | `website_crm_iap_reveal` |
| `_state_create_date` | Index | `(reveal_state,create_date)` |  | `website_crm_iap_reveal` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_clean_reveal_views` | internal rule | self | `website_crm_iap_reveal` | model | Remove old views (> 1 month) |
| `_create_reveal_view` | internal rule | self, website_id, url, ip_address, country_code, state_code, rules_excluded | `website_crm_iap_reveal` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_crm_iap_reveal` |
| `sales_team.group_sale_salesman` | no | yes | no | no | `website_crm_iap_reveal` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| CRM Reveal Views: All Views | `[(4, ref('sales_team.group_sale_salesman_all_leads'))]` | `[(1, '=', 1)]` | True | True | True | True |
| CRM Reveal Views: Personal / Global Views | `[(4, ref('sales_team.group_sale_salesman'))]` | `['\|', ('reveal_rule_id.user_id', '=', user.id), ('reveal_rule_id.user_id', '=', False)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_view_form` | form |  | `reveal_state`, `reveal_ip`, `reveal_rule_id`, `create_date` |  |  | `website_crm_iap_reveal` |
| `website_crm_iap_reveal.crm_reveal_view_tree` | list |  | `reveal_ip`, `reveal_rule_id`, `reveal_state`, `create_date` |  |  | `website_crm_iap_reveal` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_crm_iap_reveal.crm_reveal_view_action` | Lead Generation Views | list,form |  |  |  | `website_crm_iap_reveal` |

Machine-readable definition: `../../../schemas/data/entities/crm.reveal.view.json`; views: `../../../schemas/interfaces/views/crm.reveal.view.json`.
