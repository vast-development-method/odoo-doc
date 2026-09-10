# Talent Pool (`hr.talent.pool`)

**Transport name:** `hr.talent.pool`  
**Storage name:** `hr_talent_pool`  
**Kind:** persistent entity (one table)  
**Defined by package:** `hr_recruitment`

Description: Talent Pool

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Title | single line text |  | required; translatable |
| `company_id` | Company | many to one | `res.company` | default computed dynamically (lambda self: self.env.company); changes are tracked in the message thread |
| `pool_manager` | Pool Manager | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; restricted by domain `[('share', '=', False), ('company_ids', 'in', company_id)]` |
| `talent_ids` | Talent | many to many | `hr.applicant` | visible only to groups `base.group_user` |
| `no_of_talents` | # Talents | integer |  | computed by rule `_compute_talent_count` (not stored); Help: The number of talents in this talent pool. |
| `description` | Talent Pool Description | rich text |  |  |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `categ_ids` | Tags | many to many | `hr.applicant.category` |  |

## Operations (3)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `hr_recruitment` |  |  |
| `_compute_talent_count` | computation | self | `hr_recruitment` |  |  |
| `action_talent_pool_add_talents` | user action | self | `hr_recruitment` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| User: All Talent Pools | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[(1, '=', 1)]` | True | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.hr_talent_pool_view_form` | form |  | `name`, `pool_manager`, `categ_ids`, `color`, `company_id`, `description` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_talent_pool_view_list` | list |  | `name`, `pool_manager`, `no_of_talents`, `categ_ids`, `company_id` |  |  | `hr_recruitment` |
| `hr_recruitment.hr_talent_pool_view_kanban` | kanban |  | `name`, `pool_manager`, `company_id`, `color`, `name`, `pool_manager`, `company_id`, `no_of_talents` | `action_talent_pool_add_talents` |  | `hr_recruitment` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment.action_hr_talent_pool` | Talent Pool | kanban,list,form |  |  |  | `hr_recruitment` |

Machine-readable definition: `../../../schemas/data/entities/hr.talent.pool.json`; views: `../../../schemas/interfaces/views/hr.talent.pool.json`.
