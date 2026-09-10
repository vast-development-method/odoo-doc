# Partner Grade (`res.partner.grade`)

**Transport name:** `res.partner.grade`  
**Storage name:** `res_partner_grade`  
**Kind:** persistent entity (one table)  
**Defined by package:** `partnership`  
**Extended by packages:** `website_crm_partner_assign`

Description: Partner Grade

## Identity and behavior

- Mixins (classical inheritance): `website.published.mixin`
- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `sequence` | Sequence | integer |  | default `10` |
| `active` | Active | boolean |  | default `True` |
| `name` | Level Name | single line text |  | translatable |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company) |
| `default_pricelist_id` | Default Pricelist | many to one | `product.pricelist` |  |
| `partners_count` | Partners Count | integer |  | computed by rule `_compute_partners_count` (not stored) |
| `partners_label` | Partners Label | single line text |  | related through path `company_id.partnership_label` |
| `partner_weight` | Level Weight | integer |  | default `1`; Help: Gives the probability to assign a lead to this partner. (0 means no assignment.) |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_partners_count` | computation | self | `partnership` |  |  |
| `_compute_website_url` | computation | self | `website_crm_partner_assign` |  |  |
| `_default_is_published` | preparation rule | self | `website_crm_partner_assign` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `partnership` |
| `base.group_system` | yes | yes | yes | yes | `partnership` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `partnership` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `partnership` |
| `base.group_portal` | no | yes | no | no | `website_crm_partner_assign` |
| `base.group_public` | no | yes | no | no | `website_crm_partner_assign` |
| `account.group_account_readonly` | no | yes | no | no | `website_crm_partner_assign` |
| `account.group_account_invoice` | no | yes | no | no | `website_crm_partner_assign` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Portal/Public user: read only website published | `[(4, ref('base.group_portal')), (4, ref('base.group_public'))]` | `[('website_published','=', True)]` | True | True | True | True |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `partnership.view_partner_grade_tree` | list |  | `sequence`, `name` |  |  | `partnership` |
| `partnership.res_partner_grade_view_search` | search |  | `name` |  | `Archived` | `partnership` |
| `partnership.view_partner_grade_form` | form |  | `partners_count`, `partners_label`, `name`, `default_pricelist_id` | `partnership.action_grade_partners` |  | `partnership` |
| `website_crm_partner_assign.view_partner_grade_form` | div | `partnership.view_partner_grade_form` | `is_published` |  |  | `website_crm_partner_assign` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `partnership.res_partner_grade_action` | Levels |  |  |  |  | `partnership` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `partnership.menu_res_partner_grade_action` |  | `crm_menu_partners` | `partnership.res_partner_grade_action` | 1 |  |

Machine-readable definition: `../../../schemas/data/entities/res.partner.grade.json`; views: `../../../schemas/interfaces/views/res.partner.grade.json`.
