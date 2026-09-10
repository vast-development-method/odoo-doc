# Partner in-app purchase (`res.partner.iap`)

**Transport name:** `res.partner.iap`  
**Storage name:** `res_partner_iap`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail_plugin`

Description: Partner IAP

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `partner_id` | Partner | many to one | `res.partner` | required; on delete of the target: cascade |
| `iap_search_domain` | Search Domain / Email | single line text |  | Help: Domain used to find the company |
| `iap_enrich_info` | in-app purchase Enrich Info | multi line text |  | read only; Help: IAP response stored as a JSON string |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_partner_id` | Constraint | `UNIQUE(partner_id)` | Only one partner IAP is allowed for one partner | `mail_plugin` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `mail_plugin` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail_plugin.res_partner_iap_view_form` | form |  | `partner_id`, `iap_search_domain`, `iap_enrich_info` |  |  | `mail_plugin` |
| `mail_plugin.res_partner_iap_view_tree` | list |  | `partner_id`, `iap_search_domain` |  |  | `mail_plugin` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail_plugin.res_partner_iap_action` | IAP Partner | list,form |  |  |  | `mail_plugin` |

Machine-readable definition: `../../../schemas/data/entities/res.partner.iap.json`; views: `../../../schemas/interfaces/views/res.partner.iap.json`.
