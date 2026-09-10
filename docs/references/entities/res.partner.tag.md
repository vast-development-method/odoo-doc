# Partner Tags - These tags can be used on website to find customers by sector, or ... (`res.partner.tag`)

**Transport name:** `res.partner.tag`  
**Storage name:** `res_partner_tag`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_customer`

Description: Partner Tags - These tags can be used on website to find customers by sector, or ...

## Identity and behavior

- Mixins (classical inheritance): `website.published.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Category Name | single line text |  | required; translatable |
| `partner_ids` | Partners | many to many | `res.partner` | association table `res_partner_res_partner_tag_rel` |
| `classname` | Class | selection |  | required; default `info`; values provided by rule `get_selection_class`; Help: Bootstrap class to customize the color |
| `active` | Active | boolean |  | default `True` |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_selection_class` | operation | self | `website_customer` | model |  |
| `_default_is_published` | preparation rule | self | `website_customer` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_customer` |
| `base.group_portal` | no | yes | no | no | `website_customer` |
| `base.group_user` | no | yes | no | no | `website_customer` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `website_customer` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Partner Tag: published only | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_customer.view_partner_tag_form` | form |  | `name`, `classname`, `is_published`, `active` |  |  | `website_customer` |
| `website_customer.view_partner_tag_list` | list |  | `name`, `classname`, `is_published`, `active` |  |  | `website_customer` |
| `website_customer.res_partner_tag_view_search` | search |  | `name` |  | `Archived` | `website_customer` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_customer.action_partner_tag_form` | Website Tags |  |  |  |  | `website_customer` |

Machine-readable definition: `../../../schemas/data/entities/res.partner.tag.json`; views: `../../../schemas/interfaces/views/res.partner.tag.json`.
