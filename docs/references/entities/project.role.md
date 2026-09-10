# Project Role (`project.role`)

**Transport name:** `project.role`  
**Storage name:** `project_role`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`

Description: Project Role

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; translatable |
| `color` | Color | integer |  | default computed dynamically (_get_default_color) |
| `sequence` | Sequence | integer |  |  |

## Operations (2)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `project` |  |  |
| `copy_data` | lifecycle override | self, default | `project` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_user` | no | yes | no | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_role_view_list` | list |  | `sequence`, `name`, `color` |  |  | `project` |
| `project.project_role_view_form` | form |  | `active`, `name`, `color` |  |  | `project` |
| `project.project_role_view_kanban` | kanban |  | `color`, `name` |  |  | `project` |
| `project.project_role_view_search` | search |  | `name` |  | `Archived` | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_roles_action` | Project Roles | list,kanban,form |  |  |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.role.json`; views: `../../../schemas/interfaces/views/project.role.json`.
