# Maintenance Request (`maintenance.request`)

**Transport name:** `maintenance.request`  
**Storage name:** `maintenance_request`  
**Kind:** persistent entity (one table)  
**Defined by package:** `maintenance`  
**Extended by packages:** `hr_maintenance`

Description: Maintenance Request

## Identity and behavior

- Mixins (classical inheritance): `mail.thread.cc`, `mail.activity.mixin`
- Default ordering: `id desc`
- Company consistency is checked automatically on company-bound relations
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (30)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Subjects | single line text |  | required |
| `company_id` | Company | many to one | `res.company` | required; default computed dynamically (lambda self: self.env.company) |
| `description` | Description | rich text |  |  |
| `request_date` | Request Date | date |  | default computed dynamically (fields.Date.context_today); changes are tracked in the message thread; Help: Date requested for the maintenance to happen |
| `owner_user_id` | Created by User | many to one | `res.users` | computed by rule `_compute_owner` and stored; default computed dynamically (lambda s: s.env.uid); extended by packages `hr_maintenance` |
| `category_id` | Category | many to one | `maintenance.equipment.category` | read only; related through path `equipment_id.category_id` and stored; indexed (btree_not_null) |
| `equipment_id` | Equipment | many to one | `maintenance.equipment` | indexed; on delete of the target: restrict; restricted by domain `['\|', ('employee_id', '=', employee_id), ('employee_id', '=', False)]`; must belong to the same company; extended by packages `hr_maintenance` |
| `user_id` | Technician | many to one | `res.users` | computed by rule `_compute_user_id` and stored; changes are tracked in the message thread |
| `stage_id` | Stage | many to one | `maintenance.stage` | default computed dynamically (_default_stage); changes are tracked in the message thread; not copied on duplication; on delete of the target: restrict |
| `priority` | Priority | selection |  |  |
| `color` | Color Index | integer |  |  |
| `close_date` | Close Date | date |  | Help: Date the maintenance was finished. |
| `kanban_state` | Kanban State | selection |  | required; default `normal`; changes are tracked in the message thread |
| `archive` | Archive | boolean |  | default ; Help: Set archive to true to hide the maintenance request without deleting it. |
| `maintenance_type` | Maintenance Type | selection |  | default `corrective` |
| `schedule_date` | Scheduled Date | date and time |  | Help: Date the maintenance team plans the maintenance.  It should not differ much from the Request Date. |
| `schedule_end` | Scheduled End | date and time |  | computed by rule `_compute_schedule_end` and stored; Help: Expected completion date and time of the maintenance request. |
| `maintenance_team_id` | Team | many to one | `maintenance.team` | required; computed by rule `_compute_maintenance_team_id` and stored; default computed dynamically (_get_default_team_id); indexed; must belong to the same company |
| `duration` | Duration | float |  | computed by rule `_compute_duration` and stored; Help: Duration in hours. |
| `done` | Done | boolean |  | related through path `stage_id.done` |
| `instruction_type` | Instruction | selection |  | default `text` |
| `instruction_pdf` | Portable Document Format | binary |  |  |
| `instruction_google_slide` | Google Slide | single line text |  | Help: Paste the url of your Google Slide. Make sure the access to the document is public. |
| `instruction_text` | Text | rich text |  |  |
| `recurring_maintenance` | Recurrent | boolean |  | computed by rule `_compute_recurring_maintenance` and stored |
| `repeat_interval` | Repeat Every | integer |  | default `1` |
| `repeat_unit` | Repeat Unit | selection |  | default `week` |
| `repeat_type` | Until | selection |  | default `forever` |
| `repeat_until` | End Date | date |  |  |
| `employee_id` | Employee | many to one | `hr.employee` | default computed dynamically (_default_employee_get) |

## Selection values

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Very Low |
| `1` | Low |
| `2` | Normal |
| `3` | High |

### `kanban_state` (Kanban State)

| Value | Label |
|---|---|
| `normal` | In Progress |
| `blocked` | Blocked |
| `done` | Ready for next stage |

### `maintenance_type` (Maintenance Type)

| Value | Label |
|---|---|
| `corrective` | Corrective |
| `preventive` | Preventive |

### `instruction_type` (Instruction)

| Value | Label |
|---|---|
| `pdf` | PDF |
| `google_slide` | Google Slide |
| `text` | Text |

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

State machine fields of this entity: `kanban_state`. Transitions are specified in the domain documents.

## Operations (23)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_stage` | preparation rule | self | `maintenance` |  |  |
| `_creation_subtype` | internal rule | self | `maintenance` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `maintenance` |  |  |
| `_get_default_team_id` | preparation rule | self | `maintenance` |  |  |
| `archive_equipment_request` | operation | self | `maintenance` |  |  |
| `reset_equipment_request` | operation | self | `maintenance` |  | Reinsert the maintenance request into the maintenance pipe in the first stage |
| `_check_schedule_end` | validation | self | `maintenance` | constrains: `schedule_end` |  |
| `_compute_schedule_end` | computation | self | `maintenance` | depends: `schedule_date` |  |
| `_compute_duration` | computation | self | `maintenance` | depends: `schedule_date`, `schedule_end` |  |
| `_check_repeat_interval` | validation | self | `maintenance` | constrains: `repeat_interval` |  |
| `_compute_maintenance_team_id` | computation | self | `maintenance` | depends: `company_id`, `equipment_id` |  |
| `_compute_user_id` | computation | self | `maintenance` | depends: `company_id`, `equipment_id` |  |
| `_compute_recurring_maintenance` | computation | self | `maintenance` | depends: `maintenance_type` |  |
| `create` | lifecycle override | self, vals_list | `hr_maintenance`, `maintenance` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `hr_maintenance`, `maintenance` |  |  |
| `_need_new_activity` | internal rule | self, vals | `maintenance` |  |  |
| `_get_activity_note` | preparation rule | self | `maintenance` |  |  |
| `activity_update` | operation | self | `maintenance` |  | Update maintenance activities based on current record set state. It reschedule, unlink or create maintenance request activities. |
| `_add_followers` | internal rule | self | `maintenance` |  |  |
| `_read_group_stage_ids` | internal rule | self, stages, domain | `maintenance` | model | Read group customization in order to display all the stages in the kanban view, even if they are empty |
| `_default_employee_get` | preparation rule | self | `hr_maintenance` |  |  |
| `_compute_owner` | computation | self | `hr_maintenance` | depends: `employee_id` |  |
| `message_new` | messaging hook | self, msg_dict, custom_values | `hr_maintenance` | model |  |

## Validation and error messages (2)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_schedule_end` | ValidationError | End date cannot be earlier than start date. | `maintenance` |
| `_check_repeat_interval` | ValidationError | The repeat interval cannot be less than 1. | `maintenance` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `maintenance` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Users are allowed to access their own maintenance requests | `[(4, ref('base.group_user'))]` | `['\|', '\|', ('owner_user_id', '=', user.id), ('message_partner_ids', 'in', [user.partner_id.id]), ('user_id', '=', user.id)]` | True | True | True | True |
| Administrator of maintenance requests | `[(4, ref('group_equipment_manager'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Maintenance Request Multi-company rule | global (all users) | `[('company_id', 'in', company_ids + [False])]` | True | True | True | True |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_maintenance.maintenance_request_view_search_inherit_hr` | xpath | `maintenance.hr_equipment_request_view_search` | `employee_id` |  |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_form_inherit_hr` | xpath | `maintenance.hr_equipment_request_view_form` | `employee_id` |  |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_kanban_inherit_hr` | xpath | `maintenance.hr_equipment_request_view_kanban` | `employee_id` |  |  | `hr_maintenance` |
| `hr_maintenance.maintenance_request_view_tree_inherit_hr` | xpath | `maintenance.hr_equipment_request_view_tree` | `employee_id` |  |  | `hr_maintenance` |
| `maintenance.hr_equipment_request_view_search` | search |  | `name`, `category_id`, `user_id`, `equipment_id`, `owner_user_id`, `stage_id`, `maintenance_team_id` |  | `My Maintenances`, `To Do`, `Done`, `Blocked`, `Ready`, `High-priority`, `Unscheduled`, `filter_request_date`, `filter_schedule_date`, `filter_close_date`, `Unread Messages`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Active`, `Cancelled`, `Assigned to`, `Category`, `Stage`, `Created By` | `maintenance` |
| `maintenance.maintenance_request_view_activity` | activity |  | `user_id`, `user_id`, `name`, `equipment_id` |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_form` | form |  | `company_id`, `category_id`, `stage_id`, `kanban_state`, `name`, `owner_user_id`, `equipment_id`, `category_id`, `request_date`, `done`, `close_date`, `archive`, `maintenance_type`, `maintenance_team_id`, `user_id`, `schedule_date`, `schedule_end`, `recurring_maintenance`, `repeat_interval`, `repeat_unit`, `repeat_type`, `repeat_until`, `priority`, `email_cc`, `company_id`, `description`, `instruction_type`, `instruction_pdf`, `instruction_google_slide`, `instruction_text` | `Cancel`, `Reopen Request` |  | `maintenance` |
| `maintenance.hr_equipment_request_view_kanban` | kanban |  | `archive`, `color`, `name`, `owner_user_id`, `equipment_id`, `category_id`, `schedule_date`, `priority`, `activity_ids`, `kanban_state`, `user_id` |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_tree` | list |  | `message_needaction`, `name`, `request_date`, `owner_user_id`, `user_id`, `category_id`, `stage_id`, `company_id`, `activity_exception_decoration` |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_graph` | graph |  | `user_id`, `stage_id`, `duration`, `color`, `repeat_interval` |  |  | `maintenance` |
| `maintenance.hr_equipment_request_view_pivot` | pivot |  | `user_id`, `stage_id`, `color` |  |  | `maintenance` |
| `maintenance.hr_equipment_view_calendar` | calendar |  | `user_id`, `priority`, `maintenance_type`, `recurring_maintenance`, `repeat_interval`, `repeat_unit`, `repeat_type`, `repeat_until`, `done`, `archive`, `duration` |  |  | `maintenance` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `maintenance.hr_equipment_request_action` | Maintenance Requests | kanban,list,form,pivot,graph,calendar,activity |  | `{             'search_default_active': True,             'default_user_id': uid         }` |  | `maintenance` |
| `maintenance.hr_equipment_request_action_link` | Maintenance Requests | kanban,list,form,pivot,graph,calendar,activity |  | `{             'search_default_category_id': [active_id],             'search_default_active': True,             'default_category_id': active_id,         }` |  | `maintenance` |
| `maintenance.hr_equipment_request_action_from_equipment` | Maintenance Requests | kanban,list,form,pivot,graph,calendar,activity | `[('equipment_id', '=', active_id)]` | `{             'search_default_active': True,             'default_equipment_id': active_id,         }` |  | `maintenance` |
| `maintenance.hr_equipment_todo_request_action_from_dashboard` | Maintenance Requests | kanban,list,form,pivot,graph,calendar,activity | `[('maintenance_team_id', '=', active_id), ('maintenance_type', 'in', context.get('maintenance_type', ['preventive', 'corrective']))]` | `{             'search_default_active': True,             'search_default_maintenance_team_id': active_id,             'default_maintenance_team_id': active_id,         }` |  | `maintenance` |
| `maintenance.hr_equipment_request_action_cal` | Maintenance Requests | calendar,kanban,list,form,pivot,graph,activity |  | `{             'search_default_active': True,             'search_default_todo': True,         }` |  | `maintenance` |
| `maintenance.maintenance_request_action_reports` | Maintenance Requests Analysis | graph,pivot,kanban,list,form,calendar,activity |  | `{             'search_default_active': True,         }` |  | `maintenance` |

Machine-readable definition: `../../../schemas/data/entities/maintenance.request.json`; views: `../../../schemas/interfaces/views/maintenance.request.json`.
