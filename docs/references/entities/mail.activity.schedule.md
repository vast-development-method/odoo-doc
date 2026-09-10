# Activity schedule plan Wizard (`mail.activity.schedule`)

**Transport name:** `mail.activity.schedule`  
**Storage name:** `mail_activity_schedule`  
**Kind:** transient entity (temporary records used by wizards, vacuumed automatically)  
**Defined by package:** `mail`  
**Extended by packages:** `calendar`, `hr`, `hr_recruitment`

Description: Activity schedule plan Wizard

## Fields (24)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model_id` | Applies to | many to one | `ir.model` | computed by rule `_compute_res_model_id` and stored; on delete of the target: cascade; precomputed before insertion |
| `res_model` | Model | single line text |  |  |
| `res_ids` | Document identifiers | multi line text |  | computed by rule `_compute_res_ids` and stored; precomputed before insertion |
| `is_batch_mode` | Use in batch | boolean |  | computed by rule `_compute_is_batch_mode` (not stored) |
| `company_id` | Company | many to one | `res.company` | computed by rule `_compute_company_id` (not stored) |
| `error` | Error | rich text |  | computed by rule `_compute_error` (not stored) |
| `has_error` | Has Error | boolean |  | computed by rule `_compute_error` (not stored) |
| `warning` | Warning | rich text |  | computed by rule `_compute_error` (not stored) |
| `has_warning` | Has Warning | boolean |  | computed by rule `_compute_error` (not stored) |
| `plan_available_ids` | Plan Available | many to many | `mail.activity.plan` | computed by rule `_compute_plan_available_ids` and stored |
| `plan_id` | Plan | many to one | `mail.activity.plan` | computed by rule `_compute_plan_id` and stored; restricted by domain `[('id', 'in', plan_available_ids)]` |
| `plan_has_user_on_demand` | Plan Has User On Demand | boolean |  | related through path `plan_id.has_user_on_demand` |
| `plan_schedule_line_ids` | Schedule Lines | one to many | `mail.activity.schedule.line` | computed by rule `_compute_plan_schedule_line_ids` (not stored); inverse field `activity_schedule_id` |
| `plan_on_demand_user_id` | Assigned To | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); Help: Choose assignation for activities with on demand assignation. |
| `plan_date` | Plan Date | date |  | computed by rule `_compute_plan_date` and stored |
| `activity_type_id` | Activity Type | many to one | `mail.activity.type` | computed by rule `_compute_activity_type_id` and stored; on delete of the target: set null; restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', res_model)]` |
| `activity_category` | Activity Category | selection |  | read only; related through path `activity_type_id.category` |
| `date_deadline` | Due Date | date |  | computed by rule `_compute_date_deadline` and stored |
| `summary` | Summary | single line text |  | computed by rule `_compute_summary` and stored |
| `note` | Note | rich text |  | computed by rule `_compute_note` and stored |
| `activity_user_id` | Assigned to | many to one | `res.users` | computed by rule `_compute_activity_user_id` and stored |
| `chaining_type` | Chaining Type | selection |  | read only; related through path `activity_type_id.chaining_type` |
| `department_id` | Department | many to one | `hr.department` | computed by rule `_compute_department_id` (not stored) |
| `plan_department_filterable` | Plan Department Filterable | boolean |  | computed by rule `_compute_plan_department_filterable` (not stored) |

## Operations (35)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail` | model |  |
| `_compute_res_model_id` | computation | self | `mail` | depends: `res_model` |  |
| `_compute_res_ids` | computation | self | `mail` | depends_context: `active_ids` |  |
| `_compute_company_id` | computation | self | `mail` | depends: `res_model_id`, `res_ids` |  |
| `_compute_error` | computation | self | `mail` | depends: `company_id`, `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `plan_available_ids`, `activity_type_id`, `activity_user_id` |  |
| `_compute_is_batch_mode` | computation | self | `mail` | depends: `res_ids` |  |
| `_compute_plan_available_ids` | computation | self | `hr`, `mail` | depends: `company_id`, `res_model`; depends: `department_id` |  |
| `_compute_plan_id` | computation | self | `mail` | depends_context: `plan_mode`; depends: `plan_available_ids` |  |
| `_onchange_plan_id` | on change | self | `mail` | onchange: `plan_id` | Reset UX |
| `_compute_plan_date` | computation | self | `hr`, `mail` | depends: `res_model`, `res_ids` |  |
| `_compute_plan_schedule_line_ids` | computation | self | `mail` | depends: `plan_date`, `plan_id`, `plan_on_demand_user_id`, `res_model`, `res_ids` |  |
| `_compute_activity_type_id` | computation | self | `mail` | depends: `res_model` |  |
| `_onchange_activity_type_id` | on change | self | `mail` | onchange: `activity_type_id` | Reset UX |
| `_compute_date_deadline` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_compute_summary` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_compute_note` | computation | self | `mail` | depends: `activity_type_id` |  |
| `_compute_activity_user_id` | computation | self | `mail` | depends: `activity_type_id`, `res_model` |  |
| `_check_consistency` | validation | self | `mail` | constrains: `res_model_id`, `res_ids`, `plan_id`, `plan_on_demand_user_id`, `activity_type_id`, `activity_user_id` |  |
| `_check_res_ids` | validation | self | `mail` | constrains: `res_ids` | Check res_ids is a valid list of integers (or Falsy). |
| `get_model_options` | operation | self | `mail` | readonly; model | Return a list of valid models for a user to define an activity on. |
| `action_schedule_plan` | user action | self | `mail` |  |  |
| `_check_plan_templates_error` | validation | self, applied_on | `mail` |  |  |
| `_check_plan_templates_warning` | validation | self, applied_on | `mail` |  |  |
| `action_schedule_activities` | user action | self | `mail` |  |  |
| `action_schedule_activities_done` | user action | self | `mail` |  |  |
| `_action_schedule_activities` | internal rule | self | `mail` |  |  |
| `_action_schedule_activities_personal` | internal rule | self | `mail` |  |  |
| `_evaluate_res_ids` | internal rule | self | `mail` |  | Parse composer res_ids, which can be: an already valid list or tuple (generally in code), a list or tuple as a string (coming from actions). Void strings / missing values are evaluated as an empty list.  :return: a list of IDs (empty list in case of falsy strings) |
| `_get_applied_on_records` | preparation rule | self | `mail` |  |  |
| `_get_plan_available_base_domain` | preparation rule | self | `mail` |  |  |
| `_plan_filter_activity_templates_to_schedule` | internal rule | self | `mail` |  |  |
| `_onchange_activity_user_id` | on change | self | `mail` | onchange: `activity_user_id`, `activity_type_id` |  |
| `action_create_calendar_event` | user action | self | `calendar` |  |  |
| `_compute_plan_department_filterable` | computation | self | `hr_recruitment`, `hr` | depends: `res_model` |  |
| `_compute_department_id` | computation | self | `hr` | depends: `res_model_id`, `res_ids` |  |

## Validation and error messages (3)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_consistency` | ValidationError | html2plaintext(scheduler.error) | `mail` |
| `_onchange_activity_user_id` | UserError | Selected user '%(user)s' cannot upload documents on model '%(model)s' | `mail` |
| `action_create_calendar_event` | UserError | Scheduling an activity using the calendar is not possible on more than one record. | `calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | no | `mail` |

## Views (3)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.mail_activity_schedule_view_form` | xpath | `mail.mail_activity_schedule_view_form` |  |  |  | `calendar` |
| `hr.mail_activity_schedule_view_form` | xpath | `mail.mail_activity_schedule_view_form` | `department_id` |  |  | `hr` |
| `mail.mail_activity_schedule_view_form` | form |  | `activity_category`, `chaining_type`, `company_id`, `has_error`, `has_warning`, `plan_has_user_on_demand`, `res_ids`, `plan_available_ids`, `plan_id`, `activity_type_id`, `plan_date`, `plan_on_demand_user_id`, `plan_schedule_line_ids`, `responsible_user_id`, `line_description`, `line_date_deadline`, `summary`, `date_deadline`, `activity_user_id`, `res_model`, `note`, `error`, `warning` | `Save`, `Mark Done`, `Discard`, `Schedule`, `Discard` |  | `mail` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr.plan_wizard_action` | Launch Plan | form |  | `{'plan_mode': True, 'active_model': 'hr.employee'}` | new | `hr` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.schedule.json`; views: `../../../schemas/interfaces/views/mail.activity.schedule.json`.
