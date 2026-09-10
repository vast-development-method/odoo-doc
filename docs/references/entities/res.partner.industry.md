# Industry (`res.partner.industry`)

**Transport name:** `res.partner.industry`  
**Storage name:** `res_partner_industry`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Industry

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | translatable |
| `full_name` | Full Name | single line text |  | translatable |
| `active` | Active | boolean |  | default `True` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_user` | no | yes | no | no | `base` |
| `group_system` | yes | yes | yes | yes | `base` |
| `base.group_public` | no | yes | no | no | `website_customer` |
| `base.group_portal` | no | yes | no | no | `website_customer` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.res_partner_industry_view_form` | form |  | `name`, `full_name`, `active` |  |  | `base` |
| `base.res_partner_industry_view_tree` | list |  | `name`, `full_name` |  |  | `base` |
| `base.res_partner_industry_view_search` | search |  | `name`, `full_name` |  | `Archived` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.res_partner_industry_action` | Industries | list,form |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.res_partner_industry_menu` | Industries | `res_partner_menu_config` | `base.res_partner_industry_action` | 4 |  |

Machine-readable definition: `../../../schemas/data/entities/res.partner.industry.json`; views: `../../../schemas/interfaces/views/res.partner.industry.json`.
