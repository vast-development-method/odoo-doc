# Gamification Goal (`gamification.goal`)

**Transport name:** `gamification.goal`  
**Storage name:** `gamification_goal`  
**Kind:** persistent entity (one table)  
**Defined by package:** `gamification`

Description: Gamification Goal

## Identity and behavior

- Default ordering: `start_date desc, end_date desc, definition_id, id`
- Display name field: `definition_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (21)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `definition_id` | Goal Definition | many to one | `gamification.goal.definition` | required; on delete of the target: cascade |
| `user_id` | User | many to one | `res.users` | required; indexed; on delete of the target: cascade |
| `user_partner_id` | User Partner | many to one | `res.partner` | related through path `user_id.partner_id` |
| `line_id` | Challenge Line | many to one | `gamification.challenge.line` | on delete of the target: cascade |
| `challenge_id` | Challenge | many to one |  | read only; related through path `line_id.challenge_id` and stored; indexed; Help: Challenge that generated the goal, assign challenge to users to generate goals with a value in this field. |
| `start_date` | Start Date | date |  | default computed dynamically (fields.Date.today) |
| `end_date` | End Date | date |  |  |
| `target_goal` | To Reach | float |  | required |
| `current` | Current Value | float |  | required; default  |
| `completeness` | Completeness | float |  | computed by rule `_get_completion` (not stored) |
| `state` | State | selection |  | required; default `draft` |
| `to_update` | To update | boolean |  |  |
| `closed` | Closed goal | boolean |  |  |
| `computation_mode` | Computation Mode | selection |  | related through path `definition_id.computation_mode` |
| `color` | Color Index | integer |  | computed by rule `_compute_color` (not stored) |
| `remind_update_delay` | Remind delay | integer |  | Help: The number of days after which the user assigned to a manual goal will be reminded. Never reminded if no value is specified. |
| `last_update` | Last Update | date |  | Help: In case of manual goal, reminders are sent if the goal as not been updated for a while (defined in challenge). Ignored in case of non-manual goal or goal not linked to a challenge. |
| `definition_description` | Definition Description | multi line text |  | read only; related through path `definition_id.description` |
| `definition_condition` | Definition Condition | selection |  | read only; related through path `definition_id.condition` |
| `definition_suffix` | Suffix | single line text |  | read only; related through path `definition_id.full_suffix` |
| `definition_display` | Display Mode | selection |  | read only; related through path `definition_id.display_mode` |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `draft` | Draft |
| `inprogress` | In progress |
| `reached` | Reached |
| `failed` | Failed |
| `canceled` | Cancelled |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (13)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_color` | computation | self | `gamification` | depends: `end_date`, `last_update`, `state` | Set the color based on the goal's state and completion |
| `_get_completion` | computation | self | `gamification` | depends: `current`, `target_goal`, `definition_id.condition` | Return the percentage of completeness of the goal, between 0 and 100 |
| `_check_remind_delay` | validation | self | `gamification` |  | Verify if a goal has not been updated for some time and send a reminder message of needed.  :return: data to write on the goal object |
| `_get_write_values` | preparation rule | self, new_value | `gamification` |  | Generate values to write after recomputation of a goal score |
| `update_goal` | operation | self | `gamification` |  | Update the goals to recomputes values and change of states  If a manual goal is not updated for enough time, the user will be reminded to do so (done only once, in 'inprogress' state). If a goal reaches the target value, the status is set to reached If the end date is passed (at least +1 day, time not considered) without the target value being reached, the goal is set as failed. |
| `action_start` | user action | self | `gamification` |  | Mark a goal as started.  This should only be used when creating goals manually (in draft state) |
| `action_reach` | user action | self | `gamification` |  | Mark a goal as reached.  If the target goal condition is not met, the state will be reset to In Progress at the next goal update until the end date. |
| `action_fail` | user action | self | `gamification` |  | Set the state of the goal to failed.  A failed goal will be ignored in future checks. |
| `action_cancel` | user action | self | `gamification` |  | Reset the completion after setting a goal as reached or failed.  This is only the current state, if the date and/or target criteria match the conditions for a change of state, this will be applied at the next goal update. |
| `create` | lifecycle override | self, vals_list | `gamification` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `gamification` |  | Overwrite the write method to update the last_update field to today  If the current value is changed and the report frequency is set to On change, a report is generated |
| `get_action` | operation | self | `gamification` |  | Get the ir.action related to update the goal  In case of a manual goal, should return a wizard to update the value :return: action description in a dictionary |
| `_mail_get_partner_fields` | messaging hook | self, introspect_fields | `gamification` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | Can not modify the configuration of a started goal | `gamification` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | yes | no | `gamification` |
| `base.group_erp_manager` | yes | yes | yes | yes | `gamification` |
| `base.group_portal` | no | yes | yes | no | `gamification` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| User can only see his/her goals or goal from the same challenge in board visibility | `[(4, ref('base.group_user')), (4, ref('base.group_portal'))]` | `[                 '\|',                     ('user_id','=',user.id),                     '&',                         ('challenge_id.user_ids','in',user.id),                         ('challenge_id.visibility_mode','=','ranking')]` | True | True | False | False |
| Manager can see any goal | `[(4, ref('base.group_erp_manager'))]` | `[(1, '=', 1)]` | True | True | False | False |
| Multicompany rule on challenges | global (all users) | `[('user_id.company_id', 'in', company_ids)]` | True | True | True | True |
| HR Officer can see any goal | `[(4, ref('hr.group_hr_user'))]` |  | True | True | False | False |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `gamification.goal_list_view` | list |  | `definition_id`, `user_id`, `start_date`, `end_date`, `current`, `target_goal`, `completeness`, `state`, `line_id` |  |  | `gamification` |
| `gamification.goal_form_view` | form |  | `state`, `definition_id`, `user_id`, `challenge_id`, `start_date`, `end_date`, `computation_mode`, `remind_update_delay`, `last_update`, `target_goal`, `definition_suffix`, `current`, `definition_condition` | `Start goal`, `Goal Reached`, `Goal Failed`, `Reset Completion`, `refresh` |  | `gamification` |
| `gamification.goal_search_view` | search |  | `user_id`, `definition_id`, `challenge_id` |  | `My Goals`, `Draft`, `Running`, `Done`, `User`, `Goal Definition`, `State`, `End Date` | `gamification` |
| `gamification.goal_kanban_view` | kanban |  | `state`, `color`, `definition_condition`, `definition_suffix`, `definition_display`, `last_update`, `definition_id`, `user_id`, `user_id`, `current`, `current`, `target_goal`, `start_date`, `end_date` |  |  | `gamification` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `gamification.goal_list_action` | Goals | list,form,kanban |  | `{'search_default_group_by_user': True, 'search_default_group_by_definition': True}` |  | `gamification` |
| `gamification.goals_from_challenge_act` | Related Goals | kanban,list,form |  | `{'search_default_group_by_definition': True, 'search_default_inprogress': True, 'search_default_challenge_id': active_id, 'default_challenge_id': active_id}` |  | `gamification` |
| `hr_gamification.goals_menu_groupby_action2` | Goals History | list,kanban | `[('challenge_id.challenge_category', '=', 'hr')]` | `{'search_default_group_by_user': True, 'search_default_group_by_definition': True}` |  | `hr_gamification` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `gamification.email_template_goal_reminder` | Gamification: Reminder For Goal Update |  |

Machine-readable definition: `../../../schemas/data/entities/gamification.goal.json`; views: `../../../schemas/interfaces/views/gamification.goal.json`.
