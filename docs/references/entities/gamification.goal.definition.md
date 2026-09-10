# Gamification Goal Definition (`gamification.goal.definition`)

**Transport name:** `gamification.goal.definition`  
**Storage name:** `gamification_goal_definition`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`

Description: Gamification Goal Definition

## Identity and behavior

- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (19)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Goal Definition | single line text |  | required; translatable |
| `description` | Goal Description | multi line text |  |  |
| `monetary` | Monetary Value | boolean |  | default ; Help: The target and current value are defined in the company currency. |
| `suffix` | Suffix | single line text |  | translatable; Help: The unit of the target and current values |
| `full_suffix` | Full Suffix | single line text |  | computed by rule `_compute_full_suffix` (not stored); Help: The currency and suffix field |
| `computation_mode` | Computation Mode | selection |  | required; default `manually`; Help: Define how the goals will be computed. The result of the operation will be stored in the field 'Current'. |
| `display_mode` | Displayed as | selection |  | required; default `progress` |
| `model_id` | Model | many to one | `ir.model` | on delete of the target: cascade |
| `model_inherited_ids` | Model Inherited | many to many | `ir.model` | related through path `model_id.inherited_model_ids` |
| `field_id` | Field to Sum | many to one | `ir.model.fields` | restricted by domain `DOMAIN_TEMPLATE % ''` |
| `field_date_id` | Date Field | many to one | `ir.model.fields` | restricted by domain `DOMAIN_TEMPLATE % ", ('ttype', 'in', ('date', 'datetime'))"`; Help: The date to use for the time period evaluated |
| `domain` | Filter Domain | single line text |  | required; default `[]`; Help: Domain for filtering records. General rule, not user depending, e.g. [('state', '=', 'done')]. The expression can contain reference to 'user' which is a browse record of the current user if not in batch mode. |
| `batch_mode` | Batch Mode | boolean |  | Help: Evaluate the expression in batch instead of once for each user |
| `batch_distinctive_field` | Distinctive field for batch user | many to one | `ir.model.fields` | Help: In batch mode, this indicates which field distinguishes one user from the other, e.g. user_id, partner_id... |
| `batch_user_expression` | Evaluated expression for batch mode | single line text |  | Help: The value to compare with the distinctive field. The expression can contain reference to 'user' which is a browse record of the current user, e.g. user.id, user.partner_id.id... |
| `compute_code` | Python Code | multi line text |  | Help: Python code to be executed for each user. 'result' should contains the new current value. Evaluated user can be access through object.user_id. |
| `condition` | Goal Performance | selection |  | required; default `higher`; Help: A goal is considered as completed when the current value is compared to the value to reach |
| `action_id` | Action | many to one | `ir.actions.act_window` | Help: The action that will be called to update the goal value. |
| `res_id_field` | identifier Field of user | single line text |  | Help: The field name on the user profile (res.users) containing the value for res_id for action. |

## Selection values

### `computation_mode` (Computation Mode)

| Value | Label |
|---|---|
| `manually` | Recorded manually |
| `count` | Automatic: number of records |
| `sum` | Automatic: sum on a field |
| `python` | Automatic: execute a specific Python code |

### `display_mode` (Displayed as)

| Value | Label |
|---|---|
| `progress` | Progressive (using numerical values) |
| `boolean` | Exclusive (done or not-done) |

### `condition` (Goal Performance)

| Value | Label |
|---|---|
| `higher` | The higher the better |
| `lower` | The lower the better |

## Operations (5)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_full_suffix` | computation | self | `gamification` | depends: `suffix`, `monetary` |  |
| `_check_domain_validity` | validation | self | `gamification` |  |  |
| `_check_model_validity` | validation | self | `gamification` |  | make sure the selected field and model are usable |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `gamification` |  |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_domain_validity` | UserError | The domain for the definition %(definition)s seems incorrect, please check it.  %(error_message)s | `gamification` |
| `_check_model_validity` | UserError | The model configuration for the definition %(name)s seems incorrect, please check it.  %(field_name)s not stored | `gamification` |
| `_check_model_validity` | UserError | The model configuration for the definition %(name)s seems incorrect, please check it.  %(error)s not found | `gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | no | yes | no | no | `gamification` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.goal_definition_list_view` | list |  | `name`, `computation_mode` |  |  | `gamification` |
| `gamification.goal_definition_form_view` | form |  | `name`, `description`, `computation_mode`, `model_id`, `model_inherited_ids`, `field_id`, `field_date_id`, `domain`, `compute_code`, `condition`, `batch_mode`, `batch_distinctive_field`, `batch_user_expression`, `display_mode`, `suffix`, `monetary`, `action_id`, `res_id_field` |  |  | `gamification` |
| `gamification.goal_definition_search_view` | search |  | `name`, `model_id`, `field_id` |  | `Model`, `Computation Mode` | `gamification` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.goal_definition_list_action` | Goal Definitions | list,form |  |  |  | `gamification` |

Machine-readable definition: `../../../schemas/data/entities/gamification.goal.definition.json`; views: `../../../schemas/interfaces/views/gamification.goal.definition.json`.
