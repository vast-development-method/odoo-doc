# Country Group (`res.country.group`)

**Transport name:** `res.country.group`  
**Storage name:** `res_country_group`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `product`, `account`

Description: Country Group

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `code` | Code | single line text |  |  |
| `country_ids` | Countries | many to many | `res.country` | association table `res_country_res_country_group_rel` |
| `pricelist_ids` | Pricelists | many to many | `product.pricelist` | association table `res_country_group_pricelist_rel` |
| `exclude_state_ids` | Fiscal Exceptions | many to many | `res.country.state` | Help: Those states are ignored by the fiscal positions |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_code_uniq` | Constraint | `unique(code)` | The country group code must be unique! | `base` |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_sanitize_vals` | internal rule | self, vals | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `base` |
| `base.group_portal` | no | yes | no | no | `base` |
| `base.group_user` | no | yes | no | no | `base` |
| `group_partner_manager` | yes | yes | yes | yes | `base` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `account.country_group_form_inherit_account` | field | `base.view_country_group_form` | `country_ids`, `exclude_state_ids` |  |  | `account` |
| `base.view_country_group_tree` | list |  | `name`, `code` |  |  | `base` |
| `base.view_country_group_form` | form |  | `name`, `code`, `country_ids` |  |  | `base` |
| `product.inherits_website_sale_country_group_form` | group | `base.view_country_group_form` | `pricelist_ids` |  |  | `product` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_country_group` | Country Group |  |  |  |  | `base` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `contacts.menu_country_group` | Country Group | `menu_localisation` | `base.action_country_group` | 3 |  |

Machine-readable definition: `../../../schemas/data/entities/res.country.group.json`; views: `../../../schemas/interfaces/views/res.country.group.json`.
