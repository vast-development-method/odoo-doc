# Project Template create Wizard (`project.template.create.wizard`)

**Transport name:** `project.template.create.wizard`  
**Storage name:** `project_template_create_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`  
**Extended by packages:** `sale_project`

Description: Project Template create Wizard

## Fields (10)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required |
| `date_start` | Start Date | date |  |  |
| `date` | Expiration Date | date |  |  |
| `alias_name` | Alias Name | single line text |  |  |
| `alias_domain_id` | Alias Domain | many to one | `mail.alias.domain` |  |
| `template_id` | Template | many to one | `project.project` | default computed dynamically (lambda self: self.env.context.get('template_id')) |
| `template_has_dates` | Template Has Dates | boolean |  | computed by rule `_compute_template_has_dates` (not stored) |
| `role_to_users_ids` | Role To Users | one to many | `project.template.role.to.users.map` | computed by rule `_compute_role_to_users_ids` and stored; default computed dynamically (_default_role_to_users_ids); inverse field `wizard_id`; extended by packages `sale_project` |
| `partner_id` | Partner | many to one | `res.partner` |  |
| `allow_billable` | Allow Billable | boolean |  | related through path `template_id.allow_billable` |

## Operations (8)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_role_to_users_ids` | preparation rule | self | `project` |  |  |
| `_compute_template_has_dates` | computation | self | `project` | depends: `template_id` |  |
| `_get_template_whitelist_fields` | preparation rule | self | `project`, `sale_project` |  | Whitelist of fields of this wizard that will be used when creating a project from a template. |
| `_create_project_from_template` | internal rule | self | `project` |  |  |
| `create_project_from_template` | operation | self | `project` |  |  |
| `action_open_template_view` | user action | self | `project`, `sale_project` | model |  |
| `_compute_role_to_users_ids` | computation | self | `sale_project` | depends: `template_id` |  |
| `action_create_project_from_so` | user action | self | `sale_project` |  | Create a project either from template or directly if no template is set. |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_user` | no | yes | yes | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_project_view_form_simplified_template` | form |  | `name`, `date`, `date_start`, `alias_name`, `alias_domain_id`, `role_to_users_ids`, `role_id`, `user_ids` | `Create project`, `Discard` |  | `project` |
| `sale_project.project_project_view_form_simplified_template` | field | `project.project_project_view_form_simplified_template` | `date_start`, `partner_id` |  |  | `sale_project` |
| `sale_project.sale_project_view_form_simplified_template` | field | `project.project_project_view_form_simplified_template` | `partner_id` |  |  | `sale_project` |

Machine-readable definition: `../../../schemas/data/entities/project.template.create.wizard.json`; views: `../../../schemas/interfaces/views/project.template.create.wizard.json`.
