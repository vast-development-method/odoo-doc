# Project Sharing (`project.share.wizard`)

**Transport name:** `project.share.wizard`  
**Storage name:** `project_share_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Project Sharing

## Identity and behavior

- Mixins (classical inheritance): `portal.share`

## Fields (3)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `share_link` | Public Link | single line text |  | Help: Anyone with this link can access the project in read mode. |
| `collaborator_ids` | Collaborators | one to many | `project.share.collaborator.wizard` | inverse field `parent_wizard_id` |
| `existing_partner_ids` | Existing Partner | many to many | `res.partner` | computed by rule `_compute_existing_partner_ids` (not stored) |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `project` | model |  |
| `_selection_target_model` | internal rule | self | `project` | model |  |
| `_compute_resource_ref` | computation | self | `project` | depends: `res_model`, `res_id` |  |
| `_compute_existing_partner_ids` | computation | self | `project` | depends: `collaborator_ids` |  |
| `create` | lifecycle override | self, vals_list | `project` | model_create_multi |  |
| `action_share_record` | user action | self | `project` |  |  |
| `action_send_mail` | user action | self | `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | no | `project` |

## Views (2)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_share_wizard_view_form` | form |  | `res_model`, `res_id`, `share_link`, `collaborator_ids`, `partner_id`, `access_mode`, `send_invitation` | `Share Project`, `Save`, `Discard` |  | `project` |
| `project.project_share_wizard_confirm_form` | form |  |  | `Grant Portal Access`, `Discard` |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_share_wizard_action` | Share Project | form |  |  | new | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.share.wizard.json`; views: `../../../schemas/interfaces/views/project.share.wizard.json`.
