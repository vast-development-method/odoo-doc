# Default Values (`ir.default`)

**Transport name:** `ir.default`  
**Storage name:** `ir_default`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`

Description: Default Values

## Identity and behavior

- Display name field: `field_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (5)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `field_id` | Field | many to one | `ir.model.fields` | required; indexed; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | indexed; on delete of the target: cascade; Help: If set, action binding only applies for this user. |
| `company_id` | Company | many to one | `res.company` | indexed; on delete of the target: cascade; Help: If set, action binding only applies for this company |
| `condition` | Condition | single line text |  | Help: If set, applies the default upon condition. |
| `json_value` | Default Value (JavaScript Object Notation format) | single line text |  | required |

## Operations (12)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_json_format` | validation | self | `base` | constrains: `json_value`, `field_id` |  |
| `_check_accessible_field_id` | validation | self | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `set` | operation | self, model_name, field_name, value, user_id, company_id, condition | `base` | model | Defines a default value for the given field. Any entry for the same scope (field, user, company) will be replaced. The value is encoded in JSON to be stored to the database.  :param model_name: :param field_name: :param value: :param user_id: may be `False` for all users, `True` for the                 current user, or any user id :param company_id: may be `False` for all companies, `True` for                    the current user's company, or any company id :param condition: optional condition that restricts the                   applicability of the default value; this is an           |
| `_get` | internal rule | self, model_name, field_name, user_id, company_id, condition | `base` | model | Return the default value for the given field, user and company, or `None` if no default is available.  :param model_name: :param field_name: :param user_id: may be `False` for all users, `True` for the                 current user, or any user id :param company_id: may be `False` for all companies, `True` for                    the current user's company, or any company id :param condition: optional condition that restricts the                   applicability of the default value; this is an                   opaque string, but the client typically uses                   single-field |
| `_get_model_defaults` | preparation rule | self, model_name, condition | `base` | model | Return the available default values for the given model (for the current user), as a dict mapping field names to values. |
| `discard_records` | operation | self, records | `base` | model | Discard all the defaults of many2one fields using any of the given records. |
| `discard_values` | operation | self, model_name, field_name, values | `base` | model | Discard all the defaults for any of the given values. |
| `_get_field_column_fallbacks` | preparation rule | self, model_name, field_name | `base` |  |  |
| `_evaluate_condition_with_fallback` | internal rule | self, model_name, field_expr, operator, value | `base` |  | when the field value of the condition is company_dependent without customization, evaluate if its fallback value will be kept by the condition return True/False/None(for unknown) |

## Validation and error messages (5)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_json_format` | ValidationError | Invalid JSON format in Default Value field. | `base` |
| `_check_json_format` | ValidationError | Invalid value in Default Value field. Expected type '%(field_type)s' for '%(model_name)s.%(field_name)s'. | `base` |
| `set` | ValidationError | Invalid value for %(model)s.%(field)s: %(value)s is out of bounds (integers should be between -2,147,483,648 and 2,147,483,647) | `base` |
| `set` | ValidationError | Invalid field %(model)s.%(field)s | `base` |
| `set` | ValidationError | Invalid value for %(model)s.%(field)s: %(value)s | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| all internal users | no | no | no | no | `base` |
| `group_user` | yes | yes | yes | yes | `base` |
| `group_system` | yes | yes | yes | yes | `base` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Defaults: alter personal defaults | `[Command.link(ref('base.group_user'))]` | `[('user_id','=',user.id)]` | False | True | True | True |
| Defaults: alter all defaults | `[Command.link(ref('base.group_system'))]` | `[(1,'=',1)]` | False | True | True | True |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.ir_default_form_view` | form |  | `field_id`, `json_value`, `condition`, `user_id`, `company_id` |  |  | `base` |
| `base.ir_default_tree_view` | list |  | `field_id`, `json_value`, `condition`, `user_id`, `company_id` |  |  | `base` |
| `base.ir_default_search_view` | search |  | `field_id`, `user_id`, `company_id` |  | `User`, `Company` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_default_menu_action` | User-defined Defaults | list,form |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.default.json`; views: `../../../schemas/interfaces/views/ir.default.json`.
