# Project Sharing Collaborator Wizard (`project.share.collaborator.wizard`)

**Transport name:** `project.share.collaborator.wizard`  
**Storage name:** `project_share_collaborator_wizard`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `project`

Description: Project Sharing Collaborator Wizard

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `parent_wizard_id` | Parent Wizard | many to one | `project.share.wizard` |  |
| `partner_id` | Collaborator | many to one | `res.partner` | required |
| `access_mode` | Access Mode | selection |  | required; default `read`; Help: Read: collaborators can view tasks but cannot edit them. Edit with limited access: collaborators can view and edit tasks they follow in the Kanban view. Edit: collaborators can view and edit all tasks in the Kanban view. Additionally, they can choose which tasks they want to follow. |
| `send_invitation` | Send Invitation | boolean |  | computed by rule `_compute_send_invitation` and stored; default `True` |

## Selection values

### `access_mode` (Access Mode)

| Value | Label |
|---|---|
| `read` | Read |
| `edit_limited` | Edit with limited access |
| `edit` | Edit |

## Operations (1)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_send_invitation` | computation | self | `project` | depends: `partner_id`, `access_mode` |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.share.collaborator.wizard.json`.
