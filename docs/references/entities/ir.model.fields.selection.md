# Fields Selection (`ir.model.fields.selection`)

**Transport name:** `ir.model.fields.selection`  
**Storage name:** `ir_model_fields_selection`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Fields Selection

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (4)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `field_id` | Field | many to one | `ir.model.fields` | required; indexed; on delete of the target: cascade; restricted by domain `[["ttype", "in", ["selection", "reference"]]]` |
| `value` | Value | single line text |  | required |
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1000` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_selection_field_uniq` | Constraint | `UNIQUE (field_id, value)` | Selections values must be unique per field | `base` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_selection` | preparation rule | self, field_id | `base` |  | Return the given field's selection as a list of pairs (value, string). |
| `_get_selection_data` | preparation rule | self, field_id | `base` |  |  |
| `_reflect_selections` | internal rule | self, model_names | `base` |  | Reflect the selections of the fields of the given models. |
| `_update_selection` | internal rule | self, model_name, field_name, selection | `base` |  | Set the selection of a field to the given list, and return the row values of the given selection records. |
| `_existing_selection_data` | internal rule | self, model_name, field_name | `base` |  | Return the selection data of the given model, by field and value, as a dict {field_name: {value: row_values}}. |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_unlink_if_manual` | internal rule | self | `base` | ondelete |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `_process_ondelete` | background operation | self | `base` |  | Process the 'ondelete' of the given selection values. |
| `_get_records` | preparation rule | self | `base` |  | Return the records having 'self' as a value. |

## Validation and error messages (4)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_reflect_selections` | ValidationError | Fields %s contain a non-str value/label in selection | `base` |
| `create` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! | `base` |
| `write` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! | `base` |
| `_unlink_if_manual` | UserError | Properties of base fields cannot be altered in this manner! Please modify them through Python code, preferably through a custom addon! | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |
| `base.group_user` | no | no | no | no | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_model_fields_selection_form` | form |  | `field_id`, `value`, `name`, `sequence` |  |  | `base` |
| `base.view_model_fields_selection_tree` | list |  | `sequence`, `field_id`, `value`, `name` |  |  | `base` |
| `base.view_model_fields_selection_search` | search |  | `field_id`, `name` |  | `Field` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_model_fields_selection` | Fields Selection |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.model.fields.selection.json`; views: `../../../schemas/interfaces/views/ir.model.fields.selection.json`.
