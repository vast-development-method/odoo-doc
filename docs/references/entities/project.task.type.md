# Task Stage (`project.task.type`)

**Transport name:** `project.task.type`  
**Storage name:** `project_task_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `sale_project`, `project_sms`

Description: Task Stage

## Identity and behavior

- Default ordering: `sequence, id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (17)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Name | single line text |  | required; translatable |
| `sequence` | Sequence | integer |  | default `1` |
| `project_ids` | Projects | many to many | `project.project` | default computed dynamically (lambda self: self._get_default_project_ids()); association table `project_task_type_rel`; Help: Projects in which this stage is present. If you follow a similar workflow in several projects, you can share this stage among them and get consolidated information this way. |
| `mail_template_id` | Email Template | many to one | `mail.template` | restricted by domain `[["model", "=", "project.task"]]`; Help: If set, an email will be automatically sent to the customer when the task reaches this stage. |
| `color` | Color | integer |  |  |
| `fold` | Folded | boolean |  |  |
| `rating_template_id` | Rating Email Template | many to one | `mail.template` | restricted by domain `[["model", "=", "project.task"]]`; Help: If set, a rating request will automatically be sent by email to the customer when the task reaches this stage.  Alternatively, it will be sent at a regular interval as long as the task remains in this stage. |
| `auto_validation_state` | Automatic Kanban Status | boolean |  | default ; Help: Automatically modify the state when the customer replies to the feedback for this stage.  * Good feedback from the customer will update the state to 'Approved' (green bullet).  * Neutral or bad feedback will set the kanban state to 'Changes Requested' (orange bullet). |
| `rotting_threshold_days` | Days to rot | integer |  | default ; Help: Day count before tasks in this stage become stale. Set to 0 to disable         Changing this parameter will not affect the rotting status/date of resources last updated before this change. |
| `user_id` | Stage Owner | many to one | `res.users` | computed by rule `_compute_user_id` and stored; default computed dynamically (_default_user_id); indexed |
| `rating_request_deadline` | Rating Request Deadline | date and time |  | computed by rule `_compute_rating_request_deadline` and stored |
| `rating_active` | Send a customer rating request | boolean |  |  |
| `rating_status` | Customer Ratings Status | selection |  | required; default `stage`; Help: Collect feedback from your customers by sending them a rating request when a task enters a certain stage. To do so, define a rating email template on the stage. Rating when changing stage: an email will be automatically sent when a task reaches the stage. Periodic rating: an email will be automatically sent at regular intervals as long as the task remains in the stage. |
| `rating_status_period` | Rating Frequency | selection |  | required; default `monthly` |
| `show_rating_active` | Show Rating Active | boolean |  | computed by rule `_compute_show_rating_active` (not stored) |
| `sms_template_id` | text message Template | many to one | `sms.template` | restricted by domain `[["model", "=", "project.task"]]`; Help: If set, an SMS Text Message will be automatically sent to the customer when the task reaches this stage. |

## Selection values

### `rating_status` (Customer Ratings Status)

| Value | Label |
|---|---|
| `stage` | when reaching this stage |
| `periodic` | on a periodic basis |

### `rating_status_period` (Rating Frequency)

| Value | Label |
|---|---|
| `daily` | Daily |
| `weekly` | Weekly |
| `bimonthly` | Twice a Month |
| `monthly` | Once a Month |
| `quarterly` | Quarterly |
| `yearly` | Yearly |

## State fields

State machine fields of this entity: `rating_status`. Transitions are specified in the domain documents.

## Operations (14)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_project_ids` | preparation rule | self | `project` |  |  |
| `_default_user_id` | preparation rule | self | `project` |  |  |
| `_compute_rating_request_deadline` | computation | self | `project` | depends: `rating_status`, `rating_status_period` |  |
| `unlink_wizard` | operation | self, stage_view | `project` |  |  |
| `write` | lifecycle override | self, vals | `project` |  |  |
| `copy_data` | lifecycle override | self, default | `project` |  |  |
| `_unlink_if_remaining_personal_stages` | internal rule | self | `project` | ondelete | Prepare personal stages for deletion (i.e. move task to other personal stages) and avoid unlink if no remaining personal stages for an active internal user. |
| `_prepare_personal_stages_deletion` | preparation rule | self, remaining_stages_dict, personal_stages_to_update | `project` |  | _prepare_personal_stages_deletion prepare the deletion of personal stages of a single user.     Tasks using that stage will be moved to the first stage with a lower sequence if it exists     higher if not. :param self: project.task.type recordset containing the personal stage of a user              that need to be deleted :param remaining_stages_dict: list of dict representation of the personal stages of a user that                               can be used to replace the deleted ones. Can not be empty.                               e.g: [{'id': stage1_id, 'seq': stage1_sequence}, ...] :param  |
| `action_unarchive` | lifecycle override | self | `project` |  |  |
| `_compute_user_id` | computation | self | `project` | depends: `project_ids` | Fields project_ids and user_id cannot be set together for a stage. It can happen that project_ids is set after stage creation (e.g. when setting demo data). In such case, the default user_id has to be removed. |
| `_check_personal_stage_not_linked_to_projects` | validation | self | `project` | constrains: `user_id`, `project_ids` |  |
| `_send_rating_all` | internal rule | self | `project` | model |  |
| `_compute_show_rating_active` | computation | self | `sale_project` | depends: `project_ids.allow_billable` |  |
| `_onchange_project_ids` | on change | self | `sale_project` | onchange: `project_ids` |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_unlink_if_remaining_personal_stages` | UserError | Each user should have at least one personal stage. Create a new stage to which the tasks can be transferred after the selected ones are deleted. | `project` |
| `_check_personal_stage_not_linked_to_projects` | UserError | A personal stage cannot be linked to a project because it is only visible to its corresponding user. | `project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | no | yes | no | no | `project` |
| `project.group_project_user` | yes | yes | yes | yes | `project` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `base.group_portal` | no | yes | no | no | `project` |
| `base.group_user` | yes | yes | yes | yes | `project_todo` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Task Type: manager sees all | `[(4,ref('project.group_project_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Project/Task Type: see own or unowned stages | global (all users) | `[('user_id', 'in', (False, user.id))]` | True | True | True | True |
| Project/Task Type: write own stages | `[(4,ref('project.group_project_user'))]` | `[('user_id', '=', user.id)]` | False | True | True | True |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `project.task_type_search` | search |  | `name`, `project_ids`, `mail_template_id`, `rating_template_id` |  | `Archived`, `Projects` | `project` |
| `project.task_type_edit` | form |  | `active`, `name`, `fold`, `user_id`, `auto_validation_state`, `project_ids`, `rotting_threshold_days`, `color`, `sequence`, `mail_template_id`, `rating_active`, `rating_status`, `rating_status_period`, `rating_template_id` |  |  | `project` |
| `project.task_type_tree` | list |  | `sequence`, `name`, `rotting_threshold_days`, `mail_template_id`, `project_ids`, `color`, `fold` |  |  | `project` |
| `project.task_type_tree_inherited` | xpath | `task_type_tree` | `rating_template_id` |  |  | `project` |
| `project.view_project_task_type_kanban` | kanban |  | `color`, `name`, `project_ids` |  |  | `project` |
| `project_sms.task_type_edit_view_form_inherit_project_sms` | field | `project.task_type_edit` | `mail_template_id`, `sms_template_id` |  |  | `project_sms` |
| `project_sms.task_type_edit_view_tree_inherit_project_sms` | field | `project.task_type_tree` | `mail_template_id`, `sms_template_id` |  |  | `project_sms` |
| `project_sms.task_type_search_view_search_inherit_project_sms` | field | `project.task_type_search` | `rating_template_id`, `sms_template_id` |  |  | `project_sms` |
| `sale_project.task_type_edit_inherit_sale_project` | xpath | `project.task_type_edit` |  |  |  | `sale_project` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.open_task_type_form` | Task Stages | list,kanban,form | `[('user_id', '=', False)]` | `{'default_user_id': False}` |  | `project` |
| `project.open_task_type_form_domain` | Task Stages | list,kanban,form | `[('project_ids','=', project_id)]` |  |  | `project` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `project.unlink_task_type_action` | Delete | code |  | yes |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `project.ir_cron_rating_project` | Project Stage: Send rating |  days | `_send_rating_all` |  |

Machine-readable definition: `../../../schemas/data/entities/project.task.type.json`; views: `../../../schemas/interfaces/views/project.task.type.json`.
