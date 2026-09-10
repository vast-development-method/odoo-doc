# Task (`project.task`)

**Transport name:** `project.task`  
**Storage name:** `project_task`  
**Kind:** persistent entity (one table)  
**Defined by package:** `project`  
**Extended by packages:** `hr_timesheet`, `project_hr_skills`, `sale_project`, `project_sms`, `project_timesheet_holidays`, `project_todo`, `sale_timesheet`, `website_project`

Description: Task

## Identity and behavior

- Mixins (classical inheritance): `portal.mixin`, `mail.thread.cc`, `mail.activity.mixin`, `rating.mixin`, `mail.tracking.duration.mixin`, `html.field.history.mixin`
- Default ordering: `priority desc, sequence, date_deadline asc, id desc`
- Calendar date field: `date_assign`
- Posting a message requires access: read
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (102)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `active` | Active | boolean |  | default `True` |
| `name` | Title | single line text |  | required; changes are tracked in the message thread; indexed (trigram) |
| `description` | Description | rich text |  |  |
| `priority` | Priority | selection |  | default `0`; changes are tracked in the message thread; indexed |
| `sequence` | Sequence | integer |  | default `10` |
| `stage_id` | Stage | many to one | `project.task.type` | computed by rule `_compute_stage_id` and stored; default computed dynamically (_get_default_stage_id); changes are tracked in the message thread; indexed; on delete of the target: restrict; restricted by domain `[('project_ids', '=', project_id)]` |
| `stage_id_color` | Stage Color | integer |  | related through path `stage_id.color` |
| `tag_ids` | Tags | many to many | `project.tags` |  |
| `state` | State | selection |  | required; computed by rule `_compute_state` and stored; writable through an inverse rule; default `01_in_progress`; changes are tracked in the message thread; indexed; not copied on duplication; recursive dependency |
| `is_closed` | Closed state | boolean |  | computed by rule `_compute_is_closed` (not stored); searchable through a search rule |
| `create_date` | Created On | date and time |  | read only; indexed |
| `write_date` | Last Updated On | date and time |  | read only |
| `date_end` | Ending Date | date and time |  | indexed; not copied on duplication |
| `date_assign` | Assigning Date | date and time |  | read only; not copied on duplication; Help: Date on which this task was last assigned (or unassigned). Based on this, you can get statistics on the time it usually takes to assign tasks. |
| `date_deadline` | Deadline | date and time |  | changes are tracked in the message thread; indexed; not copied on duplication |
| `date_last_stage_update` | Last Stage Update | date and time |  | read only; indexed; not copied on duplication; Help: Date on which the state of your task has last been modified. Based on this information you can identify tasks that are stalling and get statistics on the time it usually takes to move tasks from one stage/state to another. |
| `project_id` | Project | many to one | `project.project` | computed by rule `_compute_project_id` and stored; changes are tracked in the message thread; indexed; restricted by domain `['\|', ('company_id', '=', False), ('company_id', '=?',  company_id), ('is_internal_project', '=', False), ('is_template', 'in', [is_template, False])]`; precomputed before insertion; recursive dependency; extended by packages `hr_timesheet` |
| `display_in_project` | Display In Project | boolean |  | computed by rule `_compute_display_in_project` and stored |
| `task_properties` | Properties | properties |  |  |
| `allocated_hours` | Allocated Time | float |  | changes are tracked in the message thread |
| `subtask_allocated_hours` | Sub-tasks Allocated Time | float |  | computed by rule `_compute_subtask_allocated_hours` (not stored); Help: Sum of the hours allocated for all the sub-tasks (and their own sub-tasks) linked to this task. Usually less than or equal to the allocated hours of this task. |
| `role_ids` | Project Roles | many to many | `project.role` | Help: When you create a project from a template, you can choose which employee takes each role. These employees will be added to the tasks, along with anyone already assigned. |
| `user_ids` | Assignees | many to many | `res.users` | default computed dynamically (_default_user_ids); changes are tracked in the message thread; restricted by domain `[('share', '=', False), ('active', '=', True)]`; association table `project_task_user_rel` |
| `portal_user_names` | Portal User Names | single line text |  | computed by rule `_compute_portal_user_names` (not stored); searchable through a search rule |
| `personal_stage_type_ids` | Personal Stages | many to many | `project.task.type` | not copied on duplication; on delete of the target: restrict; restricted by domain `[('user_id', '=', uid)]`; association table `project_task_user_rel` |
| `personal_stage_id` | Personal Stage State | many to one | `project.task.stage.personal` | computed by rule `_compute_personal_stage_id` (not stored); searchable through a search rule; Help: The current user's personal stage. |
| `personal_stage_type_id` | Personal Stage | many to one | `project.task.type` | related through path `personal_stage_id.stage_id`; restricted by domain `[('user_id', '=', uid)]`; Help: The current user's personal task stage. |
| `partner_id` | Customer | many to one | `res.partner` | computed by rule `_compute_partner_id` and stored; writable through an inverse rule; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `['\|', ('company_id', '=?', company_id), ('company_id', '=', False)]`; recursive dependency; extended by packages `sale_project` |
| `partner_phone` | Contact Number | single line text |  | computed by rule `_compute_partner_phone` and stored; writable through an inverse rule; not copied on duplication |
| `email_from` | Email From | single line text |  |  |
| `email_cc` | Email Cc | single line text |  | Help: Email addresses that were in the CC of the incoming emails from this task and that are not currently linked to an existing customer. |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` and stored; default computed dynamically (_default_company_id); recursive dependency |
| `color` | Color Index | integer |  |  |
| `rating_active` | Stage Rating Status | boolean |  | related through path `stage_id.rating_active` |
| `attachment_ids` | Attachments | one to many | `ir.attachment` | computed by rule `_compute_attachment_ids` (not stored); Help: Attachments that don't come from a message |
| `displayed_image_id` | Cover Image | many to one | `ir.attachment` | restricted by domain `[('res_model', '=', 'project.task'), ('res_id', '=', id), ('mimetype', 'ilike', 'image')]` |
| `parent_id` | Parent Task | many to one | `project.task` | writable through an inverse rule; changes are tracked in the message thread; indexed; restricted by domain `['!', ('id', 'child_of', id), ('project_id', '!=', False)]` |
| `child_ids` | Sub-tasks | one to many | `project.task` | restricted by domain `[["recurring_task", "=", false], "\|", ["parent_id.is_template", "=", true], ["is_template", "=", false]]`; inverse field `parent_id` |
| `subtask_count` | Sub-task Count | integer |  | computed by rule `_compute_subtask_count` (not stored) |
| `closed_subtask_count` | Closed Sub-tasks Count | integer |  | computed by rule `_compute_subtask_count` (not stored) |
| `project_privacy_visibility` | Project Visibility | selection |  | related through path `project_id.privacy_visibility` |
| `subtask_completion_percentage` | Subtask Completion Percentage | float |  | computed by rule `_compute_subtask_completion_percentage` (not stored) |
| `working_hours_open` | Working Hours to Assign | float |  | computed by rule `_compute_elapsed` and stored; precision `[16, 2]`; aggregated with avg |
| `working_hours_close` | Working Hours to Close | float |  | computed by rule `_compute_elapsed` and stored; precision `[16, 2]`; aggregated with avg |
| `working_days_open` | Working Days to Assign | float |  | computed by rule `_compute_elapsed` and stored; aggregated with avg |
| `working_days_close` | Working Days to Close | float |  | computed by rule `_compute_elapsed` and stored; aggregated with avg |
| `website_message_ids` | Website Message | one to many |  | restricted by domain `lambda self: [('model', '=', self._name), ('message_type', 'in', ['email', 'comment', 'email_outgoing', 'auto_comment'])]` |
| `allow_milestones` | Allow Milestones | boolean |  | related through path `project_id.allow_milestones` |
| `milestone_id` | Milestone | many to one | `project.milestone` | computed by rule `_compute_milestone_id` and stored; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `[('project_id', '=', project_id)]`; Help: Deliver your services automatically when a milestone is reached by linking it to a sales order item. |
| `has_late_and_unreached_milestone` | Has Late And Unreached Milestone | boolean |  | computed by rule `_compute_has_late_and_unreached_milestone` (not stored); searchable through a search rule |
| `allow_task_dependencies` | Allow Task Dependencies | boolean |  | related through path `project_id.allow_task_dependencies` |
| `depend_on_ids` | Blocked By | many to many | `project.task` | changes are tracked in the message thread; not copied on duplication; restricted by domain `[('project_id', '!=', False), ('id', '!=', id)]`; association table `task_dependencies_rel` |
| `depend_on_count` | Depending on Tasks | integer |  | computed by rule `_compute_depend_on_count` (not stored) |
| `closed_depend_on_count` | Closed Depending on Tasks | integer |  | computed by rule `_compute_depend_on_count` (not stored) |
| `dependent_ids` | Block | many to many | `project.task` | not copied on duplication; restricted by domain `[('project_id', '!=', False), ('id', '!=', id)]`; association table `task_dependencies_rel` |
| `dependent_tasks_count` | Dependent Tasks | integer |  | computed by rule `_compute_dependent_tasks_count` (not stored) |
| `display_parent_task_button` | Display Parent Task Button | boolean |  | computed by rule `_compute_display_parent_task_button` (not stored) |
| `current_user_same_company_partner` | Current User Same Company Partner | boolean |  | computed by rule `_compute_current_user_same_company_partner` (not stored) |
| `display_follow_button` | Display Follow Button | boolean |  | computed by rule `_compute_display_follow_button` (not stored) |
| `allow_recurring_tasks` | Allow Recurring Tasks | boolean |  | related through path `project_id.allow_recurring_tasks` |
| `recurring_task` | Recurrent | boolean |  |  |
| `recurring_count` | Tasks in Recurrence | integer |  | computed by rule `_compute_recurring_count` (not stored) |
| `recurrence_id` | Recurrence | many to one | `project.task.recurrence` | indexed (btree_not_null); not copied on duplication |
| `repeat_interval` | Repeat Every | integer |  | computed by rule `_compute_repeat` (not stored); default `1` |
| `repeat_unit` | Repeat Unit | selection |  | computed by rule `_compute_repeat` (not stored); default `week` |
| `repeat_type` | Until | selection |  | computed by rule `_compute_repeat` (not stored); default `forever` |
| `repeat_until` | End Date | date |  | computed by rule `_compute_repeat` (not stored) |
| `display_name` | Display Name | single line text |  | writable through an inverse rule; Help: Use these keywords in the title to set new tasks:          30h Allocate 30 hours to the task         #tags Set tags on the task         @user Assign the task to a user         ! Set the task a medium priority         !! Set the task a high priority         !!! Set the task a urgent priority          Make sure to use the right format and order e.g. Improve the configuration screen 5h #feature #v16 @Mitchell !; extended by packages `hr_timesheet` |
| `link_preview_name` | Link Preview Name | single line text |  | computed by rule `_compute_link_preview_name` (not stored) |
| `is_template` | Is Template | boolean |  |  |
| `has_project_template` | Has Project Template | boolean |  | related through path `project_id.is_template` |
| `has_template_ancestor` | Has Template Ancestor | boolean |  | computed by rule `_compute_has_template_ancestor` and stored; searchable through a search rule; recursive dependency |
| `analytic_account_active` | Active Analytic Account | boolean |  | related through path `project_id.analytic_account_active` |
| `allow_timesheets` | Allow timesheets | boolean |  | read only; computed by rule `_compute_allow_timesheets` (not stored); searchable through a search rule |
| `remaining_hours` | Time Remaining | float |  | read only; computed by rule `_compute_remaining_hours` and stored; Help: Number of allocated hours minus the number of hours spent. |
| `remaining_hours_percentage` | Remaining Hours Percentage | float |  | computed by rule `_compute_remaining_hours_percentage` (not stored); searchable through a search rule |
| `effective_hours` | Time Spent | float |  | computed by rule `_compute_effective_hours` and stored |
| `total_hours_spent` | Total Time Spent | float |  | computed by rule `_compute_total_hours_spent` and stored; Help: Time spent on this task and its sub-tasks (and their own sub-tasks). |
| `progress` | Progress | float |  | computed by rule `_compute_progress_hours` and stored; aggregated with avg |
| `overtime` | Overtime | float |  | computed by rule `_compute_progress_hours` and stored |
| `subtask_effective_hours` | Time Spent on Sub-tasks | float |  | computed by rule `_compute_subtask_effective_hours` and stored; recursive dependency; Help: Time spent on the sub-tasks (and their own sub-tasks) of this task. |
| `timesheet_ids` | Timesheets | one to many | `account.analytic.line` | inverse field `task_id` |
| `encode_uom_in_days` | Encode Unit of measure In Days | boolean |  | computed by rule `_compute_encode_uom_in_days` (not stored); default computed dynamically (lambda self: self._uom_in_days()) |
| `user_skill_ids` | User Skill | one to many | `hr.employee.skill` | related through path `user_ids.employee_skill_ids` |
| `sale_order_id` | Sales Order | many to one | `sale.order` | computed by rule `_compute_sale_order_id` and stored; restricted by domain `['\|', '\|', ('partner_id', '=', partner_id), ('partner_id.commercial_partner_id.id', 'parent_of', partner_id), ('partner_id', 'parent_of', partner_id)]`; Help: Sales order to which the task is linked.; extended by packages `sale_timesheet` |
| `sale_line_id` | Sales Order Item | many to one | `sale.order.line` | computed by rule `_compute_sale_line` and stored; changes are tracked in the message thread; indexed (btree_not_null); restricted by domain `lambda self: str(self._domain_sale_line_id())`; recursive dependency; Help: Sales Order Item to which the time spent on this task will be added in order to be invoiced to your customer. By default the sales order item set on the project will be selected. In the absence of one, the last prepaid sales order item that has time remaining will be used. Remove the sales order item in order to make this task non billable. You can also change or remove the sales order item of each timesheet entry individually. |
| `project_sale_order_id` | Project's sale order | many to one | `sale.order` | related through path `project_id.sale_order_id` |
| `sale_order_state` | Sale Order State | selection |  | related through path `sale_order_id.state` |
| `task_to_invoice` | To invoice | boolean |  | computed by rule `_compute_task_to_invoice` (not stored); searchable through a search rule; visible only to groups `sales_team.group_sale_salesman_all_leads` |
| `allow_billable` | Allow Billable | boolean |  | related through path `project_id.allow_billable` |
| `display_sale_order_button` | Display Sales Order | boolean |  | computed by rule `_compute_display_sale_order_button` (not stored) |
| `leave_types_count` | Time Off Types Count | integer |  | computed by rule `_compute_leave_types_count` (not stored) |
| `is_timeoff_task` | Is Time off Task | boolean |  | computed by rule `_compute_is_timeoff_task` (not stored); searchable through a search rule; visible only to groups `hr_timesheet.group_hr_timesheet_user` |
| `pricing_type` | Pricing Type | selection |  | related through path `project_id.pricing_type` |
| `is_project_map_empty` | Is Project map empty | boolean |  | computed by rule `_compute_is_project_map_empty` (not stored) |
| `has_multi_sol` | Has Multi Sol | boolean |  | computed by rule `_compute_has_multi_sol` (not stored) |
| `timesheet_product_id` | Timesheet Product | many to one |  | related through path `project_id.timesheet_product_id` |
| `remaining_hours_so` | Time Remaining on sales order | float |  | computed by rule `_compute_remaining_hours_so` (not stored); searchable through a search rule |
| `remaining_hours_available` | Remaining Hours Available | boolean |  | related through path `sale_line_id.remaining_hours_available` |
| `last_sol_of_customer` | Last Sol Of Customer | many to one | `sale.order.line` | computed by rule `_compute_last_sol_of_customer` (not stored) |
| `partner_name` | Customer Name | single line text |  | related through path `partner_id.name` and stored |
| `partner_company_name` | Company Name | single line text |  | related through path `partner_id.company_name` and stored |

## Selection values

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Low priority |
| `1` | Medium priority |
| `2` | High priority |
| `3` | Urgent |

### `repeat_unit` (Repeat Unit)

| Value | Label |
|---|---|
| `day` | Days |
| `week` | Weeks |
| `month` | Months |
| `year` | Years |

### `repeat_type` (Until)

| Value | Label |
|---|---|
| `forever` | Forever |
| `until` | Until |

## State fields

State machine fields of this entity: `state`, `sale_order_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (3)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_recurring_task_has_no_parent` | Constraint | `CHECK (NOT (recurring_task IS TRUE AND parent_id IS NOT NULL))` | You cannot convert this task into a sub-task because it is recurrent. | `project` |
| `_private_task_has_no_parent` | Constraint | `CHECK (NOT (project_id IS NULL AND parent_id IS NOT NULL))` | A private task cannot have a parent. | `project` |
| `_is_template_idx` | Index | `(is_template) WHERE is_template IS TRUE` |  | `project` |

## Operations (191)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_versioned_fields` | preparation rule | self | `project` |  |  |
| `_get_default_partner_id` | preparation rule | self, project, parent | `project`, `sale_timesheet` | model |  |
| `_get_default_stage_id` | preparation rule | self | `project` |  | Gives default stage_id |
| `_default_user_ids` | preparation rule | self | `project` | model |  |
| `_default_company_id` | preparation rule | self | `project` | model |  |
| `_read_group_stage_ids` | internal rule | self, stages, domain | `project` | model |  |
| `_read_group_personal_stage_type_ids` | internal rule | self, stages, domain | `project` | model |  |
| `_ensure_company_consistency_with_partner` | validation | self | `project` | constrains: `company_id`, `partner_id` | Ensures that the company of the task is valid for the partner. |
| `_ensure_super_task_is_not_private` | validation | self | `project` | constrains: `child_ids`, `project_id` | Ensures that the company of the task is valid for the partner. |
| `TASK_PORTAL_READABLE_FIELDS` | operation | self | `hr_timesheet`, `project`, `sale_project`, `sale_timesheet` |  |  |
| `TASK_PORTAL_WRITABLE_FIELDS` | operation | self | `project` |  |  |
| `_compute_project_id` | computation | self | `project` | depends: `parent_id.project_id` |  |
| `_compute_display_in_project` | computation | self | `project` | depends: `project_id`, `parent_id` |  |
| `_inverse_parent_id` | inverse computation | self | `project` |  |  |
| `_compute_state` | computation | self | `project` | depends: `stage_id`, `depend_on_ids.state` |  |
| `_compute_is_closed` | computation | self | `project` | depends: `state` |  |
| `_search_is_closed` | search rule | self, operator, value | `project` |  |  |
| `_get_rotting_depends_fields` | preparation rule | self | `project` |  |  |
| `_get_rotting_domain` | preparation rule | self | `project` |  |  |
| `OPEN_STATES` | operation | self | `project` |  | Return a list of the technical names complementing the CLOSED_STATES, a.k.a the open states |
| `_onchange_project_id` | on change | self | `project` | onchange: `project_id` |  |
| `is_blocked_by_dependences` | operation | self | `project` |  |  |
| `_inverse_state` | inverse computation | self | `project` |  |  |
| `_compute_personal_stage_id` | computation | self | `project` | depends_context: `uid`; depends: `user_ids` |  |
| `_search_personal_stage_id` | search rule | self, operator, value | `project` | model |  |
| `_get_default_personal_stage_create_vals` | preparation rule | self, user_id | `project` | model |  |
| `_populate_missing_personal_stages` | internal rule | self | `project` |  |  |
| `message_subscribe` | messaging hook | self, partner_ids, subtype_ids | `project` |  |  |
| `_check_no_cyclic_dependencies` | validation | self | `project` | constrains: `depend_on_ids` |  |
| `_get_recurrence_fields` | preparation rule | self | `project` | model |  |
| `_compute_repeat` | computation | self | `project` | depends: `recurring_task` |  |
| `_is_recurrence_valid` | internal rule | self | `project` |  |  |
| `_compute_recurring_count` | computation | self | `project` | depends: `recurrence_id` |  |
| `_compute_depend_on_count` | computation | self | `project` | depends: `depend_on_ids` |  |
| `_compute_dependent_tasks_count` | computation | self | `project` | depends: `dependent_ids` |  |
| `_check_parent_id` | validation | self | `project` | constrains: `parent_id` |  |
| `_get_attachments_search_domain` | preparation rule | self | `project` |  |  |
| `_compute_attachment_ids` | computation | self | `project` |  |  |
| `_compute_elapsed` | computation | self | `project` | depends: `create_date`, `date_end`, `date_assign` |  |
| `_compute_access_url` | computation | self | `project` |  |  |
| `_compute_subtask_allocated_hours` | computation | self | `project` | depends: `child_ids.allocated_hours` |  |
| `_compute_subtask_count` | computation | self | `project` | depends: `child_ids` |  |
| `_compute_partner_phone` | computation | self | `project` | depends: `partner_id.phone` |  |
| `_inverse_partner_phone` | inverse computation | self | `project` |  |  |
| `_onchange_task_company` | on change | self | `project` | onchange: `company_id` |  |
| `_compute_company_id` | computation | self | `project` | depends: `project_id.company_id`, `parent_id.company_id` |  |
| `_compute_stage_id` | computation | self | `project` | depends: `project_id` |  |
| `_compute_portal_user_names` | computation | self | `project` | depends: `user_ids` | This compute method allows to see all the names of assigned users to each task contained in `self`.  When we are in the project sharing feature, the `user_ids` contains only the users if we are a portal user. That is, only the users in the same company of the current user. So this compute method is a related of `user_ids.name` but with more records that the portal user can normally see. (In other words, this compute is only used in project sharing views to see all assignees for each task) |
| `_search_portal_user_names` | search rule | self, operator, value | `project` |  |  |
| `_compute_display_parent_task_button` | computation | self | `project` |  |  |
| `_compute_current_user_same_company_partner` | computation | self | `project` |  |  |
| `_compute_display_follow_button` | computation | self | `project` |  |  |
| `_get_group_pattern` | preparation rule | self | `hr_timesheet`, `project` |  |  |
| `_prepare_pattern_groups` | preparation rule | self | `hr_timesheet`, `project` |  |  |
| `_get_groups_patterns` | preparation rule | self | `project` |  |  |
| `_get_cannot_start_with_patterns` | preparation rule | self | `hr_timesheet`, `project` |  |  |
| `_extract_tags_and_users` | internal rule | self | `project` |  |  |
| `_extract_priority` | internal rule | self | `project` |  |  |
| `_get_groups` | preparation rule | self | `hr_timesheet`, `project` |  |  |
| `_inverse_display_name` | inverse computation | self | `project` |  |  |
| `_compute_link_preview_name` | computation | self | `project` |  |  |
| `_compute_has_template_ancestor` | computation | self | `project` | depends: `is_template`, `parent_id.has_template_ancestor` |  |
| `_search_has_template_ancestor` | search rule | self, operator, value | `project` |  |  |
| `copy_data` | lifecycle override | self, default | `project` |  |  |
| `_create_task_mapping` | internal rule | self, copied_tasks | `project` |  | Thanks to the way create and command.create is handled, the children of a copied task are created in the order in which the original child_ids were iterated, so sorting them by id gives back the index correspondence with the original children. We can use this behavior to create a mapping containing all the original tasks and their copy. :return:     task_mapping: a dict containing the mapping of the original task ids and their copied task (k: original_task.id, v: new_task)     task_dependencies: a dict containing the ids of the dependencies of the original task when they have one.     (k: orig |
| `_portal_get_parent_hash_token` | internal rule | self, pid | `project` |  |  |
| `_resolve_copied_dependencies` | internal rule | self, copied_tasks | `project` |  |  |
| `copy` | lifecycle override | self, default | `project` |  |  |
| `get_empty_list_help` | operation | self, help_message | `project` | model |  |
| `stage_find` | operation | self, section_id, domain, order | `project` |  | Override of the base.stage method Parameter of the stage search taken from the lead:  :param section_id: if set, stages must belong to this section or     be a default stage; if not set, stages must be default stages |
| `_get_view_cache_key` | preparation rule | self, view_id, view_type, **options | `project` | model | The override of fields_get making fields readonly for portal users makes the view cache dependent on the fact the user has the group portal or not |
| `default_get` | lifecycle override | self, fields | `project`, `sale_project` | model |  |
| `_portal_accessible_fields` | internal rule | self | `project` | model | Readable and writable fields by portal users. |
| `fields_get` | lifecycle override | self, allfields, attributes | `project` | model |  |
| `_has_field_access` | internal rule | self, field, operation | `project` |  |  |
| `_ensure_fields_write` | internal rule | self, vals, defaults | `project` |  |  |
| `_set_stage_on_project_from_task` | internal rule | self | `project` |  |  |
| `_load_records_create` | internal rule | self, vals_list | `project` |  |  |
| `create` | lifecycle override | self, vals_list | `project_sms`, `project_todo`, `project`, `sale_project` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `project_sms`, `project`, `sale_project` |  |  |
| `unlink` | lifecycle override | self | `project` |  |  |
| `update_date_end` | operation | self, stage_id | `project` |  |  |
| `_search_on_comodel` | search rule | self, domain, field, comodel, additional_domain | `project` |  | This method is called by `group_expand` methods, whose purpose is to add empty groups to the `read_group` (which otherwise returns groups containing records that match the domain). When specifically filtering on a comodel's field, the result of the `read_group` should contain all matching groups. However, if the search isn't filtered on any comodel's field, the result shouldn't be affected, which explains why we return `False` if `filtered_domain` is empty.  Returns:     False or recordset of the comodel given in parameter. |
| `_compute_partner_id` | computation | self | `project`, `sale_project` | depends: `parent_id.partner_id`, `project_id`; depends: `allow_billable` | Compute the partner_id when the tasks have no partner_id.  Use the project partner_id if any, or else the parent task partner_id. |
| `_compute_milestone_id` | computation | self | `project` | depends: `project_id` |  |
| `_compute_has_late_and_unreached_milestone` | computation | self | `project` |  |  |
| `_search_has_late_and_unreached_milestone` | search rule | self, operator, value | `project` |  |  |
| `_notify_by_email_prepare_rendering_context` | internal rule | self, message, msg_vals, model_description, force_email_company, force_email_lang, force_record_name | `project` |  |  |
| `_send_email_notify_to_cc` | internal rule | self, partners_to_notify | `project` |  |  |
| `_task_message_auto_subscribe_notify` | internal rule | self, users_per_task | `project` | model |  |
| `_message_auto_subscribe` | messaging hook | self, updated_values, followers_existing_policy | `project` |  |  |
| `_message_auto_subscribe_followers` | messaging hook | self, updated_values, default_subtype_ids | `project` |  |  |
| `_track_template` | messaging hook | self, changes | `project` |  |  |
| `_creation_subtype` | internal rule | self | `project` |  |  |
| `_creation_message` | internal rule | self | `project` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `project` |  |  |
| `_mail_get_message_subtypes` | messaging hook | self | `project` |  |  |
| `_notify_get_recipients_groups` | internal rule | self, message, model_description, msg_vals | `project` |  |  |
| `_notify_get_reply_to` | internal rule | self, default, author_id | `project` |  |  |
| `_find_internal_users_from_address_mail` | internal rule | self, emails, project_id | `project` |  |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `project` | model |  |
| `message_update` | messaging hook | self, msg_dict, update_vals | `project` |  |  |
| `_notify_by_email_get_headers` | internal rule | self, headers | `project` |  |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `project` |  |  |
| `_get_projects_to_make_billable_domain` | preparation rule | self, additional_domain | `project`, `sale_project` |  |  |
| `_get_all_subtasks` | preparation rule | self | `project` |  |  |
| `_get_subtask_ids_per_task_id` | preparation rule | self | `project` |  |  |
| `_get_subtasks_recursively` | preparation rule | self | `project` |  |  |
| `action_open_parent_task` | user action | self | `project` |  |  |
| `action_project_sharing_view_parent_task` | user action | self | `project` |  |  |
| `action_open_task` | user action | self | `project` |  |  |
| `action_open_subtasks` | user action | self | `project` |  |  |
| `action_project_sharing_open_task` | user action | self | `project` |  |  |
| `action_project_sharing_open_subtasks` | user action | self | `project` |  |  |
| `action_project_sharing_open_blocking` | user action | self | `project` |  |  |
| `action_dependent_tasks` | user action | self | `project` |  |  |
| `action_recurring_tasks` | user action | self | `project` |  |  |
| `action_project_sharing_recurring_tasks` | user action | self | `project` |  |  |
| `action_open_ratings` | user action | self | `project` |  |  |
| `action_unlink_recurrence` | user action | self | `project` |  |  |
| `action_convert_to_subtask` | user action | self | `project` |  |  |
| `action_convert_to_template` | user action | self | `project` |  |  |
| `action_undo_convert_to_template` | user action | self | `project` |  |  |
| `plan_task_in_calendar` | operation | self, vals | `project` |  |  |
| `_get_template_default_context_whitelist` | preparation rule | self | `project`, `sale_project` | model | Whitelist of fields that can be set through the `default_` context keys when creating a task from a template. |
| `_get_template_field_blacklist` | preparation rule | self | `project` | model | Blacklist of fields to not copy when creating a task from a template. |
| `action_create_from_template` | user action | self, values | `project` |  |  |
| `action_archive` | lifecycle override | self | `project` |  |  |
| `_get_access_action` | preparation rule | self, access_uid, force_website | `project` |  |  |
| `_send_task_rating_mail` | internal rule | self, force_send | `project` |  |  |
| `_rating_get_partner` | internal rule | self | `project`, `sale_project` |  |  |
| `rating_apply` | operation | self, rate, token, rating, feedback, subtype_xmlid, notify_delay_send | `project` |  |  |
| `_rating_apply_get_default_subtype_id` | internal rule | self | `project` |  |  |
| `_rating_get_parent_field_name` | internal rule | self | `project` |  |  |
| `_rating_get_operator` | internal rule | self | `project` |  | Overwrite since we have user_ids and not user_id |
| `_unsubscribe_portal_users` | internal rule | self | `project` |  |  |
| `get_unusual_days` | operation | self, date_from, date_to | `project` | model |  |
| `action_redirect_to_project_task_form` | user action | self | `project` |  |  |
| `_read_group` | lifecycle override | self, domain, groupby, aggregates, having, offset, limit, order | `project` | model |  |
| `project_sharing_toggle_is_follower` | operation | self | `project` |  |  |
| `_compute_subtask_completion_percentage` | computation | self | `project` | depends: `subtask_count`, `closed_subtask_count` |  |
| `_get_allowed_access_params` | preparation rule | self | `project` | model |  |
| `_get_thread_with_access` | preparation rule | self, thread_id, project_sharing_id, token, **kwargs | `project` | model |  |
| `get_mention_suggestions` | operation | self, search, limit | `project` |  | Return the 'limit'-first followers of the given task or followers of its project matching a 'search' string as a list of partner data (returned by `_to_store()`). See similar method for all partners `get_mention_suggestions()`. |
| `get_import_templates` | operation | self | `project` | model |  |
| `_check_project_root` | validation | self | `hr_timesheet` | constrains: `project_id` |  |
| `_uom_in_days` | internal rule | self | `hr_timesheet` |  |  |
| `_compute_encode_uom_in_days` | computation | self | `hr_timesheet` |  |  |
| `_compute_allow_timesheets` | computation | self | `hr_timesheet` | depends: `project_id.allow_timesheets` |  |
| `_search_allow_timesheets` | search rule | self, operator, value | `hr_timesheet` |  |  |
| `_compute_effective_hours` | computation | self | `hr_timesheet` | depends: `timesheet_ids.unit_amount` |  |
| `_compute_progress_hours` | computation | self | `hr_timesheet` | depends: `effective_hours`, `subtask_effective_hours`, `allocated_hours` |  |
| `_compute_remaining_hours_percentage` | computation | self | `hr_timesheet` | depends: `allocated_hours`, `remaining_hours` |  |
| `_search_remaining_hours_percentage` | search rule | self, operator, value | `hr_timesheet` |  |  |
| `_compute_remaining_hours` | computation | self | `hr_timesheet` | depends: `effective_hours`, `subtask_effective_hours`, `allocated_hours` |  |
| `_compute_total_hours_spent` | computation | self | `hr_timesheet` | depends: `effective_hours`, `subtask_effective_hours` |  |
| `_compute_subtask_effective_hours` | computation | self | `hr_timesheet` | depends: `child_ids.effective_hours`, `child_ids.subtask_effective_hours` |  |
| `_extract_allocated_hours` | internal rule | self | `hr_timesheet` |  |  |
| `action_view_subtask_timesheet` | user action | self | `hr_timesheet` |  |  |
| `_get_timesheet` | preparation rule | self | `hr_timesheet`, `sale_timesheet` |  |  |
| `_get_timesheet_report_data` | preparation rule | self | `hr_timesheet` |  |  |
| `_compute_display_name` | computation | self | `hr_timesheet` | depends_context: `hr_timesheet_display_remaining_hours` |  |
| `_unlink_except_contains_entries` | internal rule | self | `hr_timesheet` | ondelete | If some tasks to unlink have some timesheets entries, these timesheets entries must be unlinked first. In this case, a warning message is displayed through a RedirectWarning and allows the user to see timesheets entries to unlink. |
| `_convert_hours_to_days` | internal rule | self, time | `hr_timesheet` | model |  |
| `_get_portal_total_hours_dict` | preparation rule | self | `hr_timesheet` |  |  |
| `_domain_sale_line_id` | internal rule | self | `sale_project` |  |  |
| `_group_expand_sales_order` | internal rule | self, sales_orders, domain | `sale_project` | model |  |
| `_compute_sale_order_id` | computation | self | `sale_project` | depends: `sale_line_id`, `project_id`, `allow_billable`, `project_id.reinvoiced_sale_order_id` |  |
| `_inverse_partner_id` | inverse computation | self | `sale_project`, `sale_timesheet` |  |  |
| `_compute_sale_line` | computation | self | `sale_project`, `sale_timesheet` | depends: `sale_line_id.order_partner_id`, `parent_id.sale_line_id`, `project_id.sale_line_id`, `milestone_id.sale_line_id`, `allow_billable`; depends: `sale_line_id.order_partner_id`, `parent_id.sale_line_id`, `project_id.sale_line_id`, `allow_billable` |  |
| `_compute_display_sale_order_button` | computation | self | `sale_project` | depends: `sale_order_id` |  |
| `_check_sale_line_type` | validation | self | `sale_project` | constrains: `sale_line_id` |  |
| `_ensure_sale_order_linked` | internal rule | self, sol_ids | `sale_project` |  | Orders created from project/task are supposed to be confirmed to match the typical flow from sales, but since we allow SO creation from the project/task itself we want to confirm newly created SOs immediately after creation. However this would leads to SOs being confirmed without a single product, so we'd rather do it on record save. |
| `_get_action_view_so_ids` | preparation rule | self | `sale_project`, `sale_timesheet` |  |  |
| `action_view_so` | user action | self | `sale_project` |  |  |
| `action_project_sharing_view_so` | user action | self | `sale_project` |  |  |
| `_compute_task_to_invoice` | computation | self | `sale_project` | depends: `sale_order_id.invoice_status`, `sale_order_id.order_line` |  |
| `_search_task_to_invoice` | search rule | self, operator, value | `sale_project` | model |  |
| `_onchange_partner_id` | on change | self | `sale_project` | onchange: `sale_line_id` |  |
| `_send_sms` | internal rule | self | `project_sms` |  |  |
| `_compute_leave_types_count` | computation | self | `project_timesheet_holidays` |  |  |
| `_compute_is_timeoff_task` | computation | self | `project_timesheet_holidays` |  |  |
| `_search_is_timeoff_task` | search rule | self, operator, value | `project_timesheet_holidays` |  |  |
| `action_convert_to_task` | user action | self | `project_todo` |  |  |
| `get_todo_views_id` | operation | self | `project_todo` | model | Returns the ids of the main views used in the To-Do app.  :return: a list of views id and views type          e.g. [(kanban_view_id, "kanban"), (list_view_id, "list"), ...] :rtype: list(tuple()) |
| `_compute_remaining_hours_so` | computation | self | `sale_timesheet` | depends: `sale_line_id`, `timesheet_ids`, `timesheet_ids.unit_amount` |  |
| `_search_remaining_hours_so` | search rule | self, operator, value | `sale_timesheet` | model |  |
| `_compute_last_sol_of_customer` | computation | self | `sale_timesheet` |  |  |
| `_compute_is_project_map_empty` | computation | self | `sale_timesheet` | depends: `project_id.sale_line_employee_ids` |  |
| `_compute_has_multi_sol` | computation | self | `sale_timesheet` | depends: `timesheet_ids` |  |
| `_get_last_sol_of_customer_domain` | preparation rule | self | `sale_timesheet` |  |  |

## Validation and error messages (10)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_ensure_company_consistency_with_partner` | ValidationError | The task and the associated partner must be linked to the same company. | `project` |
| `_ensure_super_task_is_not_private` | ValidationError | This task has sub-tasks, so it can't be private. | `project` |
| `_check_no_cyclic_dependencies` | ValidationError | Two tasks cannot depend on each other. | `project` |
| `_check_parent_id` | ValidationError | Error! You cannot create a recursive hierarchy of tasks. | `project` |
| `write` | UserError | Sorry. You can't set a task as its parent task. | `project` |
| `write` | UserError | You can only set a personal stage on a private task. | `project` |
| `_check_project_root` | UserError | This task cannot be private because there are some timesheets linked to it. | `hr_timesheet` |
| `_unlink_except_contains_entries` | RedirectWarning | warning_msg | `hr_timesheet` |
| `_unlink_except_contains_entries` | UserError | This task can’t be deleted because it’s linked to timesheets. Please contact someone with higher access to remove the timesheets first, and then you’ll be able to delete the task. | `hr_timesheet` |
| `_check_sale_line_type` | ValidationError | You cannot link the order item %(order_id)s - %(product_id)s to this task because it is a re-invoiced expense. | `sale_project` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `project.group_project_user` | yes | yes | yes | yes | `project` |
| `base.group_user` | no | yes | no | no | `project` |
| `base.group_portal` | no | yes | no | no | `project` |
| `base.group_user` | yes | yes | yes | yes | `project_todo` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Project/Task: multi-company | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |
| Project/Task: employees: follow required for follower-only projects | `[(4,ref('base.group_user'))]` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | True | False | False | False |
| Project/Task: project manager: see all tasks linked to a project or its own tasks | `[(4,ref('project.group_project_manager'))]` | `[             '\|', ('project_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Project/Task: project users: follow required for follower-only projects | `[(4,ref('project.group_project_user'))]` | `[             '\|',                 '&',                     ('project_id', '!=', False),                     '\|',                         ('project_id.privacy_visibility', 'in', ['employees', 'portal']),                         ('project_id.message_partner_ids', 'in', [user.partner_id.id]),                 '\|',                     ('message_partner_ids', 'in', [user.partner_id.id]),                     # to subscribe check access to the record, follower is not enough at creation                     ('user_ids', 'in', user.id)         ]` | False | True | True | True |
| Project: See private tasks | `[(4,ref('project.group_project_user'))]` | `[             ('project_id.privacy_visibility', 'in', ['employees', 'portal']),             '\|', '\|', ('project_id', '!=', False),                       ('parent_id', '!=', False),                  ('user_ids', 'in', user.id),         ]` | True | True | True | True |
| Project/Task: portal users: can only see a task if he's a collaborator of the project and a follower of the task | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | True | False | False | False |
| Project/Task: portal users: portal user can edit with project sharing feature | `[(4, ref('base.group_portal'))]` | `[             ('project_id.privacy_visibility', 'in', ['invited_users', 'portal']),             ('active', '=', True),             '\|',                 '&',                     ('message_partner_ids', 'child_of', [user.partner_id.commercial_partner_id.id]),                     ('project_id.collaborator_ids.partner_id', 'in', [user.partner_id.id]),                 ('project_id.collaborator_ids', 'any', [                     ('partner_id', '=', user.partner_id.id),                     ('limited_access', '=', False),                 ]),         ]` | False | True | True | False |
| Project/Task: employees: Full access to own private task only | `[(4,ref('base.group_user'))]` | `[('project_id', '=', False), ('user_ids', 'in', user.id), ('parent_id', '=', False)]` | True | True | True | True |

## Views (75)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_timesheet.view_task_form2_inherited` | xpath | `project.view_task_form2` | `allocated_hours`, `effective_hours`, `subtask_effective_hours`, `total_hours_spent`, `remaining_hours`, `progress` |  |  | `hr_timesheet` |
| `hr_timesheet.view_task_tree2_inherited` | field | `project.project_task_view_tree_main_base` | `priority`, `progress`, `effective_hours`, `allocated_hours`, `effective_hours`, `subtask_effective_hours`, `total_hours_spent`, `remaining_hours`, `progress` |  |  | `hr_timesheet` |
| `hr_timesheet.view_task_kanban_inherited_progress` | templates | `project.view_task_kanban` | `progress`, `remaining_hours`, `allocated_hours`, `allow_timesheets`, `encode_uom_in_days` |  |  | `hr_timesheet` |
| `hr_timesheet.project_task_view_search` | xpath | `project.view_task_search_form_project_fsm_base` |  |  | `Timesheets 80%`, `Timesheets >100%` | `hr_timesheet` |
| `hr_timesheet.project_task_view_graph` | xpath | `project.view_project_task_graph` | `allocated_hours`, `allocated_hours`, `remaining_hours`, `remaining_hours`, `effective_hours`, `effective_hours`, `total_hours_spent`, `total_hours_spent`, `overtime`, `overtime`, `subtask_effective_hours`, `subtask_effective_hours`, `progress`, `progress` |  |  | `hr_timesheet` |
| `hr_timesheet.project_task_view_pivot` | xpath | `project.view_project_task_pivot` |  |  |  | `hr_timesheet` |
| `hr_timesheet.project_sharing_inherit_project_task_view_form` | xpath | `project.project_sharing_project_task_view_form` | `allow_timesheets` |  |  | `hr_timesheet` |
| `hr_timesheet.project_sharing_kanban_inherit_project_task_view_kanban` | templates | `project.project_sharing_project_task_view_kanban` | `progress`, `remaining_hours`, `allocated_hours`, `allow_timesheets`, `encode_uom_in_days` |  |  | `hr_timesheet` |
| `project.view_task_search_form_base` | search |  | `name`, `tag_ids`, `stage_id`, `milestone_id`, `partner_id` |  | `Unassigned`, `Favorite Projects`, `Blocking`, `Creation Date`, `Open`, `Closed`, `Closed On`, `Last 30 Days`, `Last 365 Days`, `Templates`, `Stage`, `Milestone`, `Priority`, `Tags`, `Customer`, `Company`, `Creation Date` | `project` |
| `project.view_task_search_form_project_fsm_base` | field | `view_task_search_form_base` | `stage_id`, `user_ids` |  |  | `project` |
| `project.view_task_search_form_project_base` | filter | `view_task_search_form_project_fsm_base` | `activity_user_id`, `activity_type_id` |  | `creation_date_filter`, `Deadline`, `Future`, `This Week`, `Today`, `Overdue` | `project` |
| `project.view_task_search_form` | filter | `view_task_search_form_project_base` | `task_properties` |  | `my_tasks` | `project` |
| `project.view_project_task_graph` | graph |  | `project_id`, `stage_id`, `working_hours_open`, `working_hours_close`, `color`, `sequence`, `stage_id_color`, `rating_last_value` |  |  | `project` |
| `project.view_project_task_graph_inherit` | xpath | `project.view_project_task_graph` |  |  |  | `project` |
| `project.view_project_task_pivot` | pivot |  | `project_id`, `color`, `sequence`, `stage_id_color`, `allocated_hours`, `working_hours_close`, `working_hours_open` |  |  | `project` |
| `project.view_project_task_pivot_inherit` | xpath | `project.view_project_task_pivot` | `user_ids` |  |  | `project` |
| `project.view_task_form2` | form |  | `recurrence_id`, `allow_task_dependencies`, `rating_last_value`, `rating_count`, `allow_milestones`, `parent_id`, `company_id`, `is_closed`, `depend_on_count`, `closed_depend_on_count`, `html_field_history_metadata`, `is_template`, `is_rotting`, `rotting_days`, `stage_id`, `state`, `personal_stage_type_id`, `rating_avg`, `rating_active`, `rating_avg_text`, `recurring_count`, `closed_subtask_count`, `subtask_count`, `subtask_completion_percentage`, `dependent_tasks_count`, `name`, `priority`, `state`, `priority`, `state`, `project_id`, `milestone_id`, `user_ids`, `role_ids`, `active`, `tag_ids`, `partner_id`, `date_deadline`, `recurring_task`, `repeat_interval`, `repeat_unit`, `repeat_type`, `repeat_until`, `allocated_hours`, `task_properties`, `description`, `child_ids`, `active`, `is_template`, `allow_milestones`, `display_in_project`, `sequence`, `id`, `parent_id`, `state`, `name`, `subtask_count`, `closed_subtask_count`, `project_id`, `milestone_id` | `action_open_ratings`, `action_open_parent_task`, `action_recurring_tasks`, `action_open_subtasks`, `action_dependent_tasks` |  | `project` |
| `project.quick_create_task_form` | form |  | `display_name`, `project_id`, `user_ids`, `company_id`, `parent_id`, `description` |  |  | `project` |
| `project.project_task_convert_to_subtask_view_form` | form |  | `project_id`, `company_id`, `parent_id` | `Convert Task`, `Discard` |  | `project` |
| `project.view_task_kanban` | kanban |  | `stage_id`, `rating_count`, `rating_avg`, `rating_active`, `has_late_and_unreached_milestone`, `allow_milestones`, `state`, `subtask_count`, `is_template`, `has_project_template`, `color`, `name`, `parent_id`, `project_id`, `partner_id`, `milestone_id`, `tag_ids`, `date_deadline`, `task_properties`, `displayed_image_id`, `activity_ids`, `priority`, `is_rotting`, `rotting_days`, `user_ids`, `state` |  |  | `project` |
| `project.project_sub_task_view_kanban_mobile` | xpath | `project.view_task_kanban` | `project_id` |  |  | `project` |
| `project.project_task_view_tree_main_base` | list |  | `sequence`, `allow_milestones`, `subtask_count`, `closed_subtask_count`, `id`, `name`, `project_id`, `milestone_id`, `partner_id`, `user_ids`, `company_id`, `company_id`, `date_deadline`, `priority`, `tag_ids`, `create_date`, `date_last_stage_update`, `state`, `stage_id_color`, `is_rotting`, `rotting_days`, `stage_id` |  |  | `project` |
| `project.project_task_view_tree_base` | list | `project_task_view_tree_main_base` |  |  |  | `project` |
| `project.view_task_tree2` | list | `project_task_view_tree_base` |  |  |  | `project` |
| `project.view_task_calendar` | calendar |  | `allow_milestones`, `project_id`, `display_in_project`, `subtask_count`, `milestone_id`, `user_ids`, `partner_id`, `priority`, `tag_ids`, `stage_id_color`, `stage_id`, `personal_stage_id`, `task_properties` |  |  | `project` |
| `project.view_task_all_calendar` | xpath | `view_task_calendar` |  |  |  | `project` |
| `project.project_task_view_activity` | activity |  | `user_ids`, `project_id`, `name`, `project_id`, `user_ids` |  |  | `project` |
| `project.view_task_kanban_inherit_my_task` | xpath | `view_task_kanban` |  |  |  | `project` |
| `project.view_task_kanban_inherit_all_task` | xpath | `view_task_kanban` |  |  |  | `project` |
| `project.open_view_my_tasks_list_view` | list | `view_task_tree2` |  |  |  | `project` |
| `project.open_view_all_tasks_list_view` | list | `view_task_tree2` |  |  |  | `project` |
| `project.view_task_kanban_inherit_view_default_project` | kanban | `view_task_kanban` |  |  |  | `project` |
| `project.quick_create_task_form_inherit_view_default_project` | field | `quick_create_task_form` | `project_id` |  |  | `project` |
| `project.project_task_kanban_view_project_milestone` | xpath | `view_task_kanban` |  |  |  | `project` |
| `project.project_task_tree_view_project_milestone` | xpath | `project_task_view_tree_base` |  |  |  | `project` |
| `project.project_task_pivot_view_project_milestone` | xpath | `view_project_task_pivot` | `milestone_id`, `stage_id` |  |  | `project` |
| `project.project_task_graph_view_project_milestone` | xpath | `view_project_task_graph` | `milestone_id` |  |  | `project` |
| `project.view_task_form_res_partner` | xpath | `view_task_form2` |  |  |  | `project` |
| `project.quick_create_task_form_res_partner` | xpath | `quick_create_task_form` |  |  |  | `project` |
| `project.view_task_kanban_res_partner` | xpath | `view_task_kanban_inherit_all_task` |  |  |  | `project` |
| `project.project_task_templates_list` | field | `project_task_view_tree_base` | `project_id` |  |  | `project` |
| `project.project_task_templates_kanban` | kanban | `view_task_kanban` |  |  |  | `project` |
| `project.view_task_template_search_form` | filter | `view_task_search_form` |  |  | `private_tasks` | `project` |
| `project.project_sharing_quick_create_task_form` | form |  | `name` |  |  | `project` |
| `project.project_sharing_project_task_view_kanban` | kanban |  | `state`, `allow_milestones`, `has_late_and_unreached_milestone`, `color`, `name`, `project_id`, `milestone_id`, `partner_id`, `tag_ids`, `date_deadline`, `displayed_image_id`, `priority`, `portal_user_names`, `state` |  |  | `project` |
| `project.project_sharing_project_task_view_tree` | list | `project_task_view_tree_main_base` |  |  |  | `project` |
| `project.project_sharing_project_task_view_form` | form |  | `stage_id`, `display_parent_task_button`, `recurrence_id`, `recurring_count`, `subtask_count`, `display_in_project`, `dependent_tasks_count`, `name`, `priority`, `state`, `priority`, `state`, `project_id`, `allow_milestones`, `milestone_id`, `user_ids`, `portal_user_names`, `active`, `parent_id`, `company_id`, `state`, `depend_on_count`, `allow_task_dependencies`, `current_user_same_company_partner`, `tag_ids`, `partner_id`, `date_deadline`, `recurring_task`, `repeat_interval`, `repeat_unit`, `repeat_type`, `repeat_until`, `description`, `child_ids`, `project_id`, `display_in_project`, `state`, `sequence`, `state`, `subtask_count`, `closed_subtask_count`, `name`, `allow_milestones`, `milestone_id`, `company_id`, `partner_id`, `user_ids`, `portal_user_names`, `date_deadline`, `priority`, `tag_ids`, `stage_id`, `depend_on_ids`, `project_id`, `sequence`, `state`, `subtask_count`, `closed_subtask_count`, `name`, `allow_milestones` | `action_project_sharing_view_parent_task`, `action_project_sharing_recurring_tasks`, `action_project_sharing_open_subtasks`, `action_project_sharing_open_blocking`, `View Task`, `View Task` |  | `project` |
| `project.project_sharing_project_task_view_search` | filter | `project.view_task_search_form_base` |  |  | `creation_date_filter`, `Deadline`, `Future`, `This Week`, `Today`, `Overdue` | `project` |
| `project.open_view_blocked_by_list_view` | list | `project.open_view_all_tasks_list_view` |  |  |  | `project` |
| `project_account.project_task_form_view_account_inherit` | field | `project.view_task_form2` | `partner_id` |  |  | `project_account` |
| `project_account.project_task_tree_view_account_inherit` | field | `project.project_task_view_tree_base` | `partner_id` |  |  | `project_account` |
| `project_account.project_sharing_project_task_form_view_account_inherit` | field | `project.project_sharing_project_task_view_form` | `partner_id` |  |  | `project_account` |
| `project_hr_skills.view_task_search_form_project_fsm_base_inherit` | field | `project.view_task_search_form_project_fsm_base` | `partner_id`, `user_skill_ids` |  |  | `project_hr_skills` |
| `project_timesheet_holidays.leave_task_form_view` | xpath | `hr_timesheet.view_task_form2_inherited` | `is_timeoff_task` |  |  | `project_timesheet_holidays` |
| `project_todo.project_task_view_todo_kanban` | kanban |  | `color`, `sequence`, `active`, `state`, `color`, `name`, `date_deadline`, `tag_ids`, `displayed_image_id`, `priority`, `activity_ids`, `user_ids`, `state` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_tree` | list |  | `state`, `name`, `user_ids`, `priority`, `date_deadline`, `activity_ids`, `tag_ids`, `personal_stage_type_id` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_form` | form |  | `html_field_history_metadata`, `company_id`, `project_id`, `personal_stage_type_id`, `active`, `name`, `priority`, `state`, `priority`, `state`, `user_ids`, `tag_ids`, `date_deadline`, `description` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_quick_create_form` | form |  | `display_name`, `date_deadline` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_conversion_form` | form |  | `company_id`, `project_id`, `user_ids`, `tag_ids` | `Convert to Task`, `Discard` |  | `project_todo` |
| `project_todo.project_task_view_todo_calendar` | calendar |  | `name`, `priority`, `tag_ids`, `personal_stage_id` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_activity` | activity |  | `user_ids`, `name`, `user_ids` |  |  | `project_todo` |
| `project_todo.project_task_view_todo_search` | search |  | `name`, `tag_ids`, `user_ids`, `personal_stage_type_ids` |  | `Open`, `Closed`, `Closed On`, `Deadline`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Priority`, `Tags`, `Assignees`, `Stage`, `Deadline` | `project_todo` |
| `sale_project.view_sale_project_quick_create_task_form` | xpath | `project.quick_create_task_form` |  |  |  | `sale_project` |
| `sale_project.view_sale_project_inherit_form` | xpath | `project.view_task_form2` |  | `action_view_so` |  | `sale_project` |
| `sale_project.project_task_view_tree_main_base` | field | `project.project_task_view_tree_main_base` | `partner_id`, `allow_billable` |  |  | `sale_project` |
| `sale_project.view_task_tree2_inherit_sale_project` | xpath | `project.project_task_view_tree_base` | `sale_line_id`, `sale_line_id` |  |  | `sale_project` |
| `sale_project.project_task_view_search` | field | `project.view_task_search_form_project_base` | `partner_id`, `sale_order_id` |  |  | `sale_project` |
| `sale_project.view_task_form_res_partner` | xpath | `project.view_task_form_res_partner` |  |  |  | `sale_project` |
| `sale_project.quick_create_task_form_res_partner` | xpath | `project.quick_create_task_form_res_partner` |  |  |  | `sale_project` |
| `sale_project.project_sharing_inherit_project_task_view_form` | div | `project.project_sharing_project_task_view_form` | `display_sale_order_button`, `allow_billable` | `action_project_sharing_view_so` |  | `sale_project` |
| `sale_project.project_sharing_inherit_project_task_view_tree` | field | `project.project_sharing_project_task_view_tree` | `allow_milestones`, `allow_billable` |  |  | `sale_project` |
| `sale_timesheet.view_task_tree2_inherited` | xpath | `hr_timesheet.view_task_tree2_inherited` | `sale_line_id`, `remaining_hours_available`, `remaining_hours_so` |  |  | `sale_timesheet` |
| `sale_timesheet.project_task_view_form_inherit_sale_timesheet` | xpath | `project.view_task_form2` | `is_project_map_empty`, `has_multi_sol` |  |  | `sale_timesheet` |
| `sale_timesheet.project_task_view_search_inherit_sale_timesheet` | filter | `hr_timesheet.project_task_view_search` |  |  | `timesheet_exceeded` | `sale_timesheet` |
| `sale_timesheet.project_sharing_inherit_project_task_view_form` | xpath | `hr_timesheet.project_sharing_inherit_project_task_view_form` |  |  |  | `sale_timesheet` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `project.act_project_project_2_project_task_all` | Tasks | kanban,list,form,calendar,pivot,graph,activity | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` | `{                 'default_project_id': active_id,                 'show_project_update': True,                 'search_default_open_tasks': 1,                 'display_milestone_deadline': True,             }` |  | `project` |
| `project.project_task_action_sub_task` | Sub-tasks | list,kanban,form,calendar,pivot,graph,activity | `[('id', 'child_of', active_id), ('id', '!=', active_id)]` | `{'show_project_update': False, 'default_parent_id': active_id}` |  | `project` |
| `project.action_view_task` | Tasks | kanban,list,form,calendar,pivot,graph,activity | `[('project_id', '!=', False), ('has_template_ancestor', '=', False)]` | `{'search_default_my_tasks': 1}` |  | `project` |
| `project.action_view_my_task` | My Tasks | kanban,list,form,calendar,activity,pivot,graph | `[('user_ids', 'in', uid), ('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` | `{                 'search_default_open_tasks': 1,                 'my_tasks': 1,                 'default_user_ids': [(4, uid)],             }` |  | `project` |
| `project.action_view_all_task` | All Tasks | list,kanban,form,calendar,activity,pivot,graph | `[('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` | `{'search_default_open_tasks': 1, 'default_user_ids': [(4, uid)]}` |  | `project` |
| `project.project_task_action_from_partner` | Partner's Tasks | list,kanban,form,calendar,pivot,graph,activity | `[('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  |  | `project` |
| `project.action_view_task_overpassed_draft` | Overpassed Tasks | list,form,calendar,graph,kanban | `[('is_closed', '=', False), ('date_deadline', '<', 'today'), ('project_id', '!=', False), ('has_template_ancestor', '=', False), ('has_project_template', '=', False)]` |  |  | `project` |
| `project.dblc_proj` | Project's tasks | list,form,calendar,graph,kanban | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` | `{'project_id':active_id}` |  | `project` |
| `project.action_view_task_from_milestone` | Tasks | kanban,list,calendar,pivot,graph,activity,form | `[('milestone_id', '=', active_id)]` | `{'default_milestone_id': active_id}` |  | `project` |
| `project.project_milestone_action_view_tasks` | Tasks.test | kanban,list,form,calendar,pivot,graph,activity | `[('has_template_ancestor', '=', False)]` | `{'default_project_id': active_id}` |  | `project` |
| `project.project_sharing_project_task_action` | Project Sharing | kanban,list,form | `[('project_id', '=', active_id), ('has_template_ancestor', '=', False)]` | `{             'default_project_id': active_id,             'active_id_chatter': active_id,             'delete': false,         }` |  | `project` |
| `project.project_sharing_project_task_action_blocking_tasks` | Blocking | list,kanban,form | `[('depend_on_ids', '=', active_id), ('id', '!=', active_id)]` | `{'default_dependent_ids': active_id}` |  | `project` |
| `project.project_sharing_project_task_action_sub_task` | Sub-tasks | list,kanban,form | `[('id', 'child_of', active_id), ('id', '!=', active_id)]` | `{'default_parent_id': active_id}` |  | `project` |
| `project.project_sharing_project_task_recurring_tasks_action` | Project Sharing Recurrence | list,kanban,form |  |  |  | `project` |
| `project_mail_plugin.project_task_action_form_edit` | Task: redirect to form in edit mode | form |  |  |  | `project_mail_plugin` |
| `project_todo.project_task_action_todo` | To-dos | kanban,form,list,calendar,activity | `[('user_ids', 'in', [uid]), ('project_id', '=', False), ('parent_id', '=', False)]` | `{             'search_default_open_tasks': 1,             'list_view_ref': 'project_todo.project_task_view_todo_tree',             'default_project_id': False,             'show_todo_mail_helper': True,             'show_task_options': False,         }` |  | `project_todo` |
| `project_todo.project_task_action_convert_todo_to_task` | Convert to Task | form |  | `{'dialog_size': 'medium'}` | new | `project_todo` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `project_todo.menu_todo_todos` | To-do |  | `project_todo.project_task_action_todo` |  |  |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `project.action_server_convert_to_subtask` | Convert to Task/Sub-Task | code |  | yes |
| `project.action_server_convert_to_template` | Convert to Template | code |  | yes |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `hr_timesheet.timesheet_report_task` | Timesheets | qweb-pdf | `hr_timesheet.report_project_task_timesheet` |  |  |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `project.mail_template_data_project_task` | Project: Request Acknowledgment | Reception of {{ object.name }} |
| `project.rating_project_request_email_template` | Project: Task Rating Request | {{ object.project_id.company_id.name or user.env.company.name }}: Satisfaction Survey |

Machine-readable definition: `../../../schemas/data/entities/project.task.json`; views: `../../../schemas/interfaces/views/project.task.json`.
