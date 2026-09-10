# Action Window (`ir.actions.act_window`)

**Transport name:** `ir.actions.act_window`  
**Storage name:** `ir_act_window`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Action Window

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`
- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (18)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `type` | Type | single line text |  | default `ir.actions.act_window` |
| `view_id` | View Ref. | many to one | `ir.ui.view` | on delete of the target: set null |
| `domain` | Domain Value | single line text |  | Help: Optional domain filtering of the destination data, as a Python expression |
| `context` | Context Value | single line text |  | required; default `{}`; Help: Context dictionary as Python expression, empty by default (Default: {}) |
| `res_id` | Record identifier | integer |  | Help: Database ID of record to open in form view, when ``view_mode`` is set to 'form' only |
| `res_model` | Destination Model | single line text |  | required; Help: Model name of the object to open in the view window |
| `target` | Target Window | selection |  | default `current` |
| `view_mode` | View Mode | single line text |  | required; default `list,form`; Help: Comma-separated list of allowed view modes, such as 'form', 'list', 'calendar', etc. (Default: list,form) |
| `mobile_view_mode` | Mobile View Mode | single line text |  | default `kanban`; Help: First view mode in mobile and small screen environments (default='kanban'). If it can't be found among available view modes, the same mode as for wider screens is used) |
| `usage` | Action Usage | single line text |  | Help: Used to filter menu and home actions from the user form. |
| `view_ids` | No of Views | one to many | `ir.actions.act_window.view` | inverse field `act_window_id` |
| `views` | Views | binary |  | computed by rule `_compute_views` (not stored); Help: This function field computes the ordered list of views that should be enabled when displaying the result of an action, federating view mode, views and reference view. The result is returned as an ordered list of pairs (view_id,view_mode). |
| `limit` | Limit | integer |  | default `80`; Help: Default limit for the list view |
| `group_ids` | Groups | many to many | `res.groups` | association table `ir_act_window_group_rel` |
| `search_view_id` | Search View Ref. | many to one | `ir.ui.view` |  |
| `embedded_action_ids` | Embedded Action | one to many | `ir.embedded.actions` | computed by rule `_compute_embedded_actions` (not stored) |
| `filter` | Filter | boolean |  |  |
| `cache` | Data Caching | boolean |  | default `True`; Help: If enabled, this action will cache the related data used in list, Kanban and form views with the aim to increase the loading speed |

## Selection values

### `target` (Target Window)

| Value | Label |
|---|---|
| `current` | Current Window |
| `new` | New Window |
| `fullscreen` | Full Screen |
| `main` | Main action of Current Window |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_model` | validation | self | `base` | constrains: `res_model`, `binding_model_id` |  |
| `_compute_views` | computation | self | `base` | depends: `view_ids.view_mode`, `view_mode`, `view_id.type` | Compute an ordered list of the specific view modes that should be enabled when displaying the result of this action, along with the ID of the specific view to use for each mode, if any were required.  This function hides the logic of determining the precedence between the view_modes string, the view_ids o2m, and the view_id m2o that can be set on the action. |
| `_check_view_mode` | validation | self | `base` | constrains: `view_mode` |  |
| `_compute_embedded_actions` | computation | self | `base` |  |  |
| `read` | lifecycle override | self, fields, load | `base` |  | call the method get_empty_list_help of the model and set the window action help message |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `exists` | operation | self | `base` |  |  |
| `_existing` | internal rule | self | `base` | model |  |
| `_get_readable_fields` | preparation rule | self | `base` |  |  |
| `_get_action_dict` | preparation rule | self | `base` |  | Override to return action content with detailed embedded actions data if available.  :return: A dict with updated action dictionary including embedded actions information. |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_model` | ValidationError | Invalid model name “%s” in action definition. | `base` |
| `_check_model` | ValidationError | Invalid model name “%s” in action definition. | `base` |
| `_check_view_mode` | ValidationError | The modes in view_mode must not be duplicated: %s | `base` |
| `_check_view_mode` | ValidationError | No spaces allowed in view_mode: “%s” | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_window_action_tree` | list |  | `name`, `res_model`, `view_id`, `domain`, `context` |  |  | `base` |
| `base.view_window_action_form` | form |  | `name`, `xml_id`, `path`, `res_model`, `usage`, `type`, `target`, `cache`, `view_mode`, `mobile_view_mode`, `view_id`, `search_view_id`, `domain`, `context`, `limit`, `filter`, `help`, `view_ids`, `sequence`, `view_mode`, `view_id`, `sequence`, `view_mode`, `view_id`, `group_ids` |  |  | `base` |
| `base.view_window_action_search` | search |  | `name` |  | `Binding Model`, `Target Window` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_action_window` | Window Actions |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.act_window.json`; views: `../../../schemas/interfaces/views/ir.actions.act_window.json`.
