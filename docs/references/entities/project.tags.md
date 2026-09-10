# Project Tags (`project.tags`)

**Transport name:** `project.tags`  
**Storage name:** `project_tags`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`

Description: Project Tags

## Identity and behavior

- Default ordering: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `color` | Color | integer |  | default computed dynamically (_get_default_color); Help: Transparent tags are not visible in the kanban view of your projects and tasks. |
| `project_ids` | Projects | many to many | `project.project` | association table `project_project_project_tags_rel` |
| `task_ids` | Tasks | many to many | `project.task` |  |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_name_uniq` | Constraint | `unique (name)` | A tag with the same name already exists. | `project` |

## Operations (7)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_color` | preparation rule | self | `project` |  |  |
| `_get_project_tags_domain` | preparation rule | self, domain, project_id | `project` |  |  |
| `formatted_read_group` | operation | self, domain, groupby, aggregates, having, offset, limit, order | `project` | model |  |
| `search_read` | lifecycle override | self, domain, fields, offset, limit, order | `project` | model |  |
| `arrange_tag_list_by_id` | operation | self, tag_list, id_order | `project` | model | Re-order a list of record values (dict) following a given id sequence, in O(n).  :param tag_list: ordered (by id) list of record values, each record being a dict     containing at least an 'id' key  :param id_order: list of value (int) corresponding to the id of the records to re-arrange :returns: Sorted list of record values (dict) |
| `name_search` | operation | self, name, domain, operator, limit | `project` | model |  |
| `name_create` | lifecycle override | self, name | `project` | model |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `base.group_portal` | no | yes | no | no | `project` |
| `base.group_user` | yes | yes | yes | yes | `project_todo` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.project_tags_search_view` | search |  | `name` |  |  | `project` |
| `project.project_tags_form_view` | form |  | `name`, `color` |  |  | `project` |
| `project.project_tags_tree_view` | list |  | `name`, `color` |  |  | `project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.project_tags_action` | Tags |  |  |  |  | `project` |

Machine-readable definition: `../../../schemas/data/entities/project.tags.json`; views: `../../../schemas/interfaces/views/project.tags.json`.
