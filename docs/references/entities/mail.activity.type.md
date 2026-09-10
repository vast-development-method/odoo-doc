# Activity Type (`mail.activity.type`)

**Transport name:** `mail.activity.type`  
**Storage name:** `mail_activity_type`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `fleet`, `calendar`, `hr_holidays`

Description: Activity Type

## Identity and behavior

- Default ordering: `sequence, id`
- Display name field: `name`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (22)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | required; translatable |
| `summary` | Default Summary | single line text |  | translatable |
| `sequence` | Sequence | integer |  | default `10` |
| `active` | Active | boolean |  | default `True` |
| `create_uid` | Create Uid | many to one | `res.users` | indexed |
| `delay_count` | Schedule | integer |  | default ; Help: Number of days/week/month before executing the action. It allows to plan the action deadline. |
| `delay_unit` | Delay units | selection |  | required; default `days`; Help: Unit of delay |
| `delay_label` | Delay Label | single line text |  | computed by rule `_compute_delay_label` (not stored) |
| `delay_from` | Delay Type | selection |  | required; default `previous_activity`; Help: Type of delay |
| `icon` | Icon | single line text |  | Help: Font awesome icon e.g. fa-tasks |
| `decoration_type` | Decoration Type | selection |  | Help: Change the background color of the related activities of this type. |
| `res_model` | Model | selection |  | Help: Specify a model if the activity should be specific to a model and not available when managing activities for other models. |
| `triggered_next_type_id` | Trigger | many to one | `mail.activity.type` | computed by rule `_compute_triggered_next_type_id` and stored; writable through an inverse rule; on delete of the target: restrict; restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', res_model)]`; Help: Automatically schedule this activity once the current one is marked as done. |
| `chaining_type` | Chaining Type | selection |  | required; default `suggest` |
| `suggested_next_type_ids` | Suggest | many to many | `mail.activity.type` | computed by rule `_compute_suggested_next_type_ids` and stored; writable through an inverse rule; restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', res_model)]`; association table `mail_activity_rel`; Help: Suggest these activities once the current one is marked as done. |
| `previous_type_ids` | Preceding Activities | many to many | `mail.activity.type` | restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', res_model)]`; association table `mail_activity_rel` |
| `category` | Action | selection |  | default `default`; Help: Actions may trigger specific behavior like opening calendar view or automatically mark as done when a document is uploaded; extended by packages `calendar` |
| `mail_template_ids` | Email templates | many to many | `mail.template` |  |
| `default_user_id` | Default User | many to one | `res.users` |  |
| `default_note` | Default Note | rich text |  | translatable |
| `initial_res_model` | Initial model | selection |  | computed by rule `_compute_initial_res_model` (not stored); Help: Technical field to keep track of the model at the start of editing to support UX related behaviour |
| `res_model_change` | Model has change | boolean |  | default  |

## Selection values

### `delay_unit` (Delay units)

| Value | Label |
|---|---|
| `days` | days |
| `weeks` | weeks |
| `months` | months |

### `delay_from` (Delay Type)

| Value | Label |
|---|---|
| `current_date` | after previous activity completion date |
| `previous_activity` | after previous activity deadline |

### `decoration_type` (Decoration Type)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

### `chaining_type` (Chaining Type)

| Value | Label |
|---|---|
| `suggest` | Suggest Next Activity |
| `trigger` | Trigger Next Activity |

### `category` (Action)

| Value | Label |
|---|---|
| `default` | None |
| `upload_file` | Upload Document |
| `phonecall` | Phonecall |
| `meeting` | Meeting |

## Operations (15)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_model_selection` | preparation rule | self | `mail` |  |  |
| `_check_activity_type_res_model` | validation | self | `mail` | constrains: `res_model` |  |
| `_onchange_res_model` | on change | self | `mail` | onchange: `res_model` |  |
| `_compute_initial_res_model` | computation | self | `mail` |  |  |
| `_compute_delay_label` | computation | self | `mail` | depends: `delay_unit`, `delay_count` |  |
| `_compute_suggested_next_type_ids` | computation | self | `mail` | depends: `chaining_type` | suggested_next_type_ids and triggered_next_type_id should be mutually exclusive |
| `_inverse_suggested_next_type_ids` | inverse computation | self | `mail` |  |  |
| `_compute_triggered_next_type_id` | computation | self | `mail` | depends: `chaining_type` | suggested_next_type_ids and triggered_next_type_id should be mutually exclusive |
| `_inverse_triggered_next_type_id` | inverse computation | self | `mail` |  |  |
| `write` | lifecycle override | self, vals | `mail` |  |  |
| `_unlink_except_todo` | internal rule | self | `mail` | ondelete |  |
| `action_archive` | lifecycle override | self | `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  | When removing an activity type, put activities into a Todo. |
| `_get_date_deadline` | preparation rule | self | `mail` |  | Return the activity deadline computed from today or from activity_previous_deadline context variable. |
| `_get_model_info_by_xmlid` | preparation rule | self | `fleet`, `hr_holidays`, `mail` | model | Get model info based on xml ids. |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `write` | UserError | You cannot modify %(activities_names)s target model as they are are required in various apps. | `mail` |
| `_unlink_except_todo` | UserError | You cannot delete %(activity_names)s as it is required in various apps. | `mail` |
| `action_archive` | UserError | The 'To-Do' activity type is used to create reminders from the top bar menu and the command palette. Consequently, it cannot be archived or deleted. | `mail` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `fleet.fleet_group_manager` | yes | yes | yes | yes | `fleet` |
| `hr_expense.group_hr_expense_manager` | yes | yes | yes | yes | `hr_expense` |
| `hr_holidays.group_hr_holidays_manager` | yes | yes | yes | yes | `hr_holidays` |
| `base.group_user` | no | yes | no | no | `mail` |
| `base.group_system` | yes | yes | yes | yes | `mail` |
| `maintenance.group_equipment_manager` | yes | yes | yes | yes | `maintenance` |
| `project.group_project_manager` | yes | yes | yes | yes | `project` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `sale` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `mail.mail_activity_type_view_form` | form |  | `name`, `active`, `category`, `default_user_id`, `res_model`, `summary`, `icon`, `decoration_type`, `delay_count`, `delay_unit`, `delay_from`, `chaining_type`, `triggered_next_type_id`, `suggested_next_type_ids`, `mail_template_ids`, `default_note` |  |  | `mail` |
| `mail.mail_activity_type_view_search` | search |  | `name` |  | `Archived` | `mail` |
| `mail.mail_activity_type_view_tree` | list |  | `sequence`, `name`, `summary`, `delay_label`, `delay_from`, `res_model`, `icon`, `triggered_next_type_id`, `suggested_next_type_ids` |  |  | `mail` |
| `mail.mail_activity_type_view_kanban` | kanban |  | `icon`, `name`, `res_model`, `summary`, `default_user_id` |  |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `fleet.mail_activity_type_action_config_fleet` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'fleet.vehicle.log.contract')]` | `{'default_res_model': 'fleet.vehicle.log.contract'}` |  | `fleet` |
| `hr_expense.mail_activity_type_action_config_hr_expense` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'hr.expense')]` | `{'default_res_model': 'hr.expense'}` |  | `hr_expense` |
| `hr_holidays.mail_activity_type_action_config_hr_holidays` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', 'in', ['hr.leave', 'hr.leave.allocation'])]` | `{'default_res_model': 'hr.leave'}` |  | `hr_holidays` |
| `hr_recruitment.mail_activity_type_action_config_hr_applicant` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'hr.applicant')]` | `{'default_res_model': 'hr.applicant'}` |  | `hr_recruitment` |
| `mail.mail_activity_type_action` | Activity Types | list,kanban,form |  |  |  | `mail` |
| `maintenance.mail_activity_type_action_config_maintenance` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'maintenance.request')]` | `{'default_res_model': 'maintenance.request'}` |  | `maintenance` |
| `project.mail_activity_type_action_config_project_types` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'project.task')]` | `{'default_res_model': 'project.task'}` |  | `project` |
| `sale.mail_activity_type_action_config_sale` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'sale.order')]` | `{'default_res_model': 'sale.order'}` |  | `sale` |
| `sales_team.mail_activity_type_action_config_sales` | Activity Types | list,kanban,form | `['\|', ('res_model', '=', False), ('res_model', '=', 'res.partner')]` |  |  | `sales_team` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `crm.crm_team_menu_config_activity_types` | Activity Types | `crm_team_menu_config_activities` | `sales_team.mail_activity_type_action_config_sales` | 10 |  |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.type.json`; views: `../../../schemas/interfaces/views/mail.activity.type.json`.
