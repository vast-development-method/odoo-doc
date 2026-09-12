# System Parameter (`ir.config_parameter`)

**Transport name:** `ir.config_parameter`  
**Storage name:** `ir_config_parameter`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `analytic`, `auth_oauth`, `crm`, `sale`

Description: System Parameter

## Identity and behavior

- Default ordering: `key`
- Display name field: `key`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (2)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `key` | Key | single line text |  | required |
| `value` | Value | multi line text |  | required |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_key_uniq` | Constraint | `unique (key)` | Key must be unique. | `base` |

## Operations (11)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `init` | lifecycle override | self, force | `auth_oauth`, `base` |  | Initializes the parameters listed in _default_parameters. It overrides existing parameters if force is `True`. |
| `get_param` | operation | self, key, default | `base` | model | Retrieve the value for a given key.  :param string key: The key of the parameter value to retrieve. :param string default: default value if parameter is missing. :return: The value of the parameter, or `default` if it does not exist. :rtype: string |
| `_get_param` | preparation rule | self, key | `base` | model |  |
| `set_param` | operation | self, key, value | `base`, `mail` | model | Sets the value of a parameter.  :param string key: The key of the parameter value to set. :param string value: The value to set. :return: the previous value of the parameter or False if it did          not exist. :rtype: string |
| `create` | lifecycle override | self, vals_list | `base`, `crm`, `mail`, `sale` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `analytic`, `base`, `crm`, `mail`, `sale` |  | When this paramater is changed, dynamic fields needs to be recomputed |
| `unlink` | lifecycle override | self | `base`, `crm`, `sale` |  |  |
| `unlink_default_parameters` | operation | self | `base` | ondelete |  |
| `_sanitize_param_value` | internal rule | self, key, value | `mail` |  | Dispatcher for sanitization logic |
| `_sale_sync_linked_crons` | internal rule | self, unlink | `sale` |  | Synchronize Sales-related crons' `active` field based on linked configuration parameters.  :param bool unlink: Whether this sync is triggered by parameter deletion. :return: None |
| `_get_param_cron_mapping` | preparation rule | self | `sale` |  | Return a mapping of config parameters to linked crons' XMLIDs.  :return: The config-cron mapping. :rtype: dict |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | ValidationError | You cannot rename config parameters with keys %s | `base` |
| `unlink_default_parameters` | ValidationError | You cannot delete the %s record. | `base` |
| `write` | UserError | The value for %s must be the ID to a valid analytic plan that is not a subplan | `analytic` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_ir_config_search` | search |  | `key`, `value` |  |  | `base` |
| `base.view_ir_config_list` | list |  | `key`, `value` |  |  | `base` |
| `base.view_ir_config_form` | form |  | `key`, `value` |  |  | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.ir_config_list_action` | System Parameters |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.config_parameter.json`; views: `../../../schemas/interfaces/views/ir.config_parameter.json`.
