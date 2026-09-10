# Actions (`ir.actions.actions`)

**Transport name:** `ir.actions.actions`  
**Storage name:** `ir_actions`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Actions

## Identity and behavior

- Default ordering: `name, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (8)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Action Name | single line text |  | required; translatable |
| `type` | Action Type | single line text |  | required |
| `xml_id` | External identifier | single line text |  | computed by rule `_compute_xml_id` (not stored) |
| `path` | Path to show in the uniform resource locator | single line text |  |  |
| `help` | Action Description | rich text |  | translatable; Help: Optional help text for the users with a description of the target view, such as its usage and purpose. |
| `binding_model_id` | Binding Model | many to one | `ir.model` | on delete of the target: cascade; Help: Setting a value makes this action available in the sidebar for the given model. |
| `binding_type` | Binding Type | selection |  | required; default `action` |
| `binding_view_types` | Binding View Types | single line text |  | default `list,form` |

## Selection values

### `binding_type` (Binding Type)

| Value | Label |
|---|---|
| `action` | Action |
| `report` | Report |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_path_unique` | Constraint | `unique(path)` | Path to show in the URL must be unique! Please choose another one. | `base` |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_path` | validation | self | `base` | constrains: `path` |  |
| `_compute_xml_id` | computation | self | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  | unlink ir.action.todo/ir.filters which are related to actions which will be deleted. NOTE: ondelete cascade will not work on ir.actions.actions so we will need to do it manually. |
| `_unlink_check_home_action` | internal rule | self | `base` | ondelete |  |
| `_get_eval_context` | preparation rule | self, action | `base` | model | evaluation context to pass to safe_eval |
| `get_bindings` | operation | self, model_name | `base` | model | Retrieve the list of actions bound to the given model.  :return: a dict mapping binding types to a list of dict describing          actions, where the latter is given by calling the method          ``read`` on the action record. |
| `_get_bindings` | preparation rule | self, model_name | `base` |  |  |
| `_for_xml_id` | internal rule | self, full_xml_id | `base` | model | Returns the action content for the provided xml_id  :param full_xml_id: the namespace-less id of the action (the @id     attribute from the XML file) :return: A read() view of the ir.actions.action safe for web use |
| `_get_action_dict` | preparation rule | self | `base` |  | Returns the action content for the provided action record. |
| `_get_readable_fields` | preparation rule | self | `base` |  | return the list of fields that are safe to read  Fetched via /web/action/load or _for_xml_id method Only fields used by the web client should included Accessing content useful for the server-side must be done manually with superuser |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_path` | ValidationError | The path should contain only lowercase alphanumeric characters, underscore, and dash, and it should start with a letter. | `base` |
| `_check_path` | ValidationError | 'm-' is a reserved prefix. | `base` |
| `_check_path` | ValidationError | 'action-' is a reserved prefix. | `base` |
| `_check_path` | ValidationError | 'new' is reserved, and can not be used as path. | `base` |
| `_check_path` | ValidationError | Path to show in the URL must be unique! Please choose another one. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.action_view` | form |  | `name`, `type` |  |  | `base` |
| `base.action_view_tree` | list |  | `name`, `type` |  |  | `base` |
| `base.action_view_search` | search |  | `name` |  | `Action Type`, `Binding Type`, `Binding Model` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_sequence_actions` | Actions |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.actions.json`; views: `../../../schemas/interfaces/views/ir.actions.actions.json`.
