# Record Rule (`ir.rule`)

**Transport name:** `ir.rule`  
**Storage name:** `ir_rule`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `website`

Description: Record Rule

## Identity and behavior

- Default ordering: `model_id DESC,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (9)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  |  |
| `active` | Active | boolean |  | default `True`; Help: If you uncheck the active field, it will disable the record rule without deleting it (if you delete a native record rule, it may be re-created when you reload the module). |
| `model_id` | Model | many to one | `ir.model` | required; indexed; on delete of the target: cascade |
| `groups` | Groups | many to many | `res.groups` | on delete of the target: restrict; association table `rule_group_rel` |
| `domain_force` | Domain | multi line text |  |  |
| `perm_read` | Read | boolean |  | default `True` |
| `perm_write` | Write | boolean |  | default `True` |
| `perm_create` | Create | boolean |  | default `True` |
| `perm_unlink` | Delete | boolean |  | default `True` |

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_no_access_rights` | Constraint | `CHECK (perm_read!=False or perm_write!=False or perm_create!=False or perm_unlink!=False)` | Rule must have at least one checked access right! | `base` |

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_eval_context` | internal rule | self | `base`, `website` | model | Returns a dictionary to use as evaluation context for ir.rule domains. Note: company_ids contains the ids of the activated companies by the user with the switch company menu. These companies are filtered and trusted. |
| `_compute_global` | computation | self | `base` | depends: `groups` |  |
| `_check_model_name` | validation | self | `base` | constrains: `model_id` |  |
| `_check_domain` | validation | self | `base` | constrains: `active`, `domain_force`, `model_id` |  |
| `_compute_domain_keys` | computation | self | `base`, `website` |  | Return the list of context keys to use for caching ``_compute_domain``. |
| `_get_failing` | preparation rule | self, for_records, mode | `base` |  | Returns the rules for the mode for the current user which fail on the specified records.  Can return any global rule and/or all local rules (since local rules are OR-ed together, the entire group succeeds or fails, while global rules get AND-ed and can each fail) |
| `_get_rules` | preparation rule | self, model_name, mode | `base` |  | Returns all the rules matching the model for the mode for the current user. |
| `_compute_domain` | computation | self, model_name, mode | `base` | model |  |
| `_compute_domain_context_values` | computation | self | `base` |  |  |
| `unlink` | lifecycle override | self | `base` |  |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_make_access_error` | internal rule | self, operation, records | `base` |  |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_model_name` | ValidationError | Rules can not be applied on the Record Rules model. | `base` |
| `_check_domain` | ValidationError | Invalid domain: %s | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_erp_manager` | yes | yes | yes | yes | `base` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_rule_form` | form |  | `name`, `model_id`, `active`, `perm_read`, `perm_create`, `perm_write`, `perm_unlink`, `domain_force`, `global`, `groups` |  |  | `base` |
| `base.view_rule_tree` | list |  | `name`, `model_id`, `groups`, `domain_force`, `perm_read`, `perm_write`, `perm_create`, `perm_unlink` |  |  | `base` |
| `base.view_rule_search` | search |  | `name`, `domain_force`, `model_id`, `groups` |  | `Global`, `Group-based`, `Full Access`, `Read`, `Write`, `Create`, `Delete`, `Archived`, `Model`, `Group` | `base` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_rule` | Record Rules |  |  |  |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.rule.json`; views: `../../../schemas/interfaces/views/ir.rule.json`.
