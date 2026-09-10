# Project Stage (`project.project.stage`)

**Transport name:** `project.project.stage`  
**Storage name:** `project_project_stage`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `project_sms`

Description: Project Stage

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `sequence` | Sequence | integer |  | default `50` |
| `name` | Name | single line text |  | required; translatable |
| `mail_template_id` | Email Template | many to one | `mail.template` | restricted by domain `[["model", "=", "project.project"]]`; Help: If set, an email will be automatically sent to the customer when the project reaches this stage. |
| `fold` | Folded | boolean |  | Help: If enabled, this stage will be displayed as folded in the Kanban and List views of your projects. Projects in a folded stage are considered as closed. |
| `company_id` | Company | many to one | `res.company` |  |
| `color` | Color | integer |  |  |
| `sms_template_id` | text message Template | many to one | `sms.template` | restricted by domain `[["model", "=", "project.project"]]`; Help: If set, an SMS Text Message will be automatically sent to the customer when the project reaches this stage. |

## Operations (4)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `copy_data` | lifecycle override | self, default | `project` |  |  |
| `unlink_wizard` | operation | self, stage_view | `project` |  |  |
| `write` | lifecycle override | self, vals | `project` |  |  |
| `action_unarchive` | lifecycle override | self | `project` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You are not able to switch the company of this stage to %(company_name)s since it currently includes projects associated with %(project_company_name)s. Please ensure that this stage exclusively consists of projects linked to %(company_name)s. | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project Stage: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (8)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_project_stage_view_tree` | list |  | `sequence`, `name`, `mail_template_id`, `company_id`, `color`, `fold` |  |  | `project` |
| `project.project_project_stage_view_form_quick_create` | form |  | `name`, `mail_template_id`, `fold` |  |  | `project` |
| `project.project_project_stage_view_form` | form |  | `name`, `active`, `mail_template_id`, `color`, `sequence`, `fold`, `company_id` |  |  | `project` |
| `project.project_project_stage_view_kanban` | kanban |  | `color`, `name`, `mail_template_id`, `company_id` |  |  | `project` |
| `project.project_project_stage_view_search` | search |  | `name`, `mail_template_id`, `company_id` |  | `Archived`, `Company` | `project` |
| `project_sms.project_project_stage_view_tree_inherit_project_sms` | field | `project.project_project_stage_view_tree` | `mail_template_id`, `sms_template_id` |  |  | `project_sms` |
| `project_sms.project_project_stage_view_form_inherit_project_sms` | field | `project.project_project_stage_view_form` | `mail_template_id`, `sms_template_id` |  |  | `project_sms` |
| `project_sms.project_project_stage_view_search_inherit_project_sms` | field | `project.project_project_stage_view_search` | `mail_template_id`, `sms_template_id` |  |  | `project_sms` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_project_stage_configure` | Project Stages | list,kanban,form |  |  |  | `project` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `project.unlink_project_stage_action` | Delete | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/project.project.stage.json`; views: `../../../schemas/interfaces/views/project.project.stage.json`.
