# Contract Type (`hr.contract.type`)

**Transport name:** `hr.contract.type`  
**Storage name:** `hr_contract_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr`

Description: Contract Type

## Identity and behavior

- Default ordering: `sequence`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `code` | Code | single line text |  | computed by rule `_compute_code` and stored |
| `sequence` | Sequence | integer |  |  |
| `country_id` | Country | many to one | `res.country` | restricted by domain `lambda self: [('id', 'in', self.env.companies.country_id.ids)]` |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_code` | computation | self | `hr` | depends: `name` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_user` | yes | yes | yes | yes | `hr` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| HR Contract Type: Multi Company | global (all users) | `['\|', ('country_id', '=', False), ('country_id', 'in', user.env.companies.country_id.ids)]` | True | True | True | True |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr.hr_contract_type_view_tree` | list |  | `sequence`, `name`, `code`, `country_id` |  |  | `hr` |
| `hr.hr_contract_type_view_form` | form |  | `name`, `code`, `country_id` |  |  | `hr` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.hr_contract_type_action` | Employment Types | list |  |  |  | `hr` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `hr_recruitment.menu_hr_recruitment_contract_type` |  | `menu_hr_recruitment_config_jobs` | `hr.hr_contract_type_action` | 2 | `hr.group_hr_user` |

Machine-readable definition: `../../../schemas/data/entities/hr.contract.type.json`; views: `../../../schemas/interfaces/views/hr.contract.type.json`.
