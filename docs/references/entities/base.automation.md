# Automation Rule (`base.automation`)

**Transport name:** `base.automation`  
**Storage name:** `base_automation`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base_automation`

Description: Automation Rule

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (26)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Automation Rule Name | single line text |  | required; translatable; changes are tracked in the message thread |
| `description` | Description | rich text |  |  |
| `model_id` | Model | many to one | `ir.model` | required; changes are tracked in the message thread; on delete of the target: cascade; restricted by domain `[["abstract", "=", false]]` |
| `model_name` | Model Name | single line text |  | read only; related through path `model_id.model`; writable through an inverse rule |
| `model_is_mail_thread` | Model Is Mail Thread | boolean |  | related through path `model_id.is_mail_thread` |
| `action_server_ids` | Actions | one to many | `ir.actions.server` | computed by rule `_compute_action_server_ids` and stored; inverse field `base_automation_id` |
| `url` | Uniform resource locator | single line text |  | computed by rule `_compute_url` (not stored); Help: Use this URL in the third-party app to call this webhook. |
| `webhook_uuid` | Webhook UUID | single line text |  | read only; default computed dynamically (lambda self: str(uuid4())); not copied on duplication |
| `record_getter` | Record Getter | single line text |  | default `model.env[payload.get('_model')].browse(int(payload.get('_id')))`; Help: This code will be run to find on which record the automation rule should be run. |
| `log_webhook_calls` | Log Calls | boolean |  | default  |
| `active` | Active | boolean |  | default `True`; Help: When unchecked, the rule is hidden and will not be executed. |
| `trigger` | Trigger | selection |  | required; computed by rule `_compute_trigger` and stored; changes are tracked in the message thread |
| `trg_selection_field_id` | Trigger Field | many to one | `ir.model.fields.selection` | computed by rule `_compute_trg_selection_field_id` and stored; restricted by domain `[('field_id', 'in', trigger_field_ids)]`; Help: Some triggers need a reference to a selection field. This field is used to store it. |
| `trg_field_ref_model_name` | Trigger Field Model | single line text |  | computed by rule `_compute_trg_field_ref_model_name` (not stored) |
| `trg_field_ref` | Trigger Reference | many to one by reference |  | computed by rule `_compute_trg_field_ref` and stored; Help: Some triggers need a reference to another field. This field is used to store it. |
| `trg_date_id` | Trigger Date | many to one | `ir.model.fields` | computed by rule `_compute_trg_date_id` and stored; changes are tracked in the message thread; restricted by domain `[('model_id', '=', model_id), ('ttype', 'in', ('date', 'datetime'))]`; Help: When should the condition be triggered.                 If present, will be checked by the scheduler. If empty, will be checked at creation and update. |
| `trg_date_range` | Delay | integer |  | computed by rule `_compute_trg_date_range_data` and stored; changes are tracked in the message thread |
| `trg_date_range_mode` | Delay mode | selection |  | computed by rule `_compute_trg_date_range_data` and stored; changes are tracked in the message thread |
| `trg_date_range_type` | Delay unit | selection |  | computed by rule `_compute_trg_date_range_data` and stored; changes are tracked in the message thread |
| `trg_date_calendar_id` | Use Calendar | many to one | `resource.calendar` | computed by rule `_compute_trg_date_calendar_id` and stored; Help: When calculating a day-based timed condition, it is possible to use a calendar to compute the date based on working days. |
| `filter_pre_domain` | Before Update Domain | single line text |  | computed by rule `_compute_filter_pre_domain` and stored; Help: If present, this condition must be satisfied before the update of the record. Not checked on record creation. |
| `previous_domain` | Previous Domain | single line text |  | default computed dynamically (lambda self: self.filter_domain) |
| `filter_domain` | Apply on | single line text |  | computed by rule `_compute_filter_domain` and stored; Help: If present, this condition must be satisfied before executing the automation rule. |
| `last_run` | Last Run | date and time |  | read only; not copied on duplication |
| `on_change_field_ids` | On Change Fields Trigger | many to many | `ir.model.fields` | computed by rule `_compute_on_change_field_ids` and stored; association table `base_automation_onchange_fields_rel`; Help: Fields that trigger the onchange. |
| `trigger_field_ids` | Trigger Fields | many to many | `ir.model.fields` | computed by rule `_compute_trigger_field_ids` and stored; Help: The automation rule will be triggered if and only if one of these fields is updated.If empty, all fields are watched. |

## Selection values

### `trigger` (Trigger)

| Value | Label |
|---|---|
| `on_stage_set` | Stage is set to |
| `on_user_set` | User is set |
| `on_tag_set` | Tag is added |
| `on_state_set` | State is set to |
| `on_priority_set` | Priority is set to |
| `on_archive` | On archived |
| `on_unarchive` | On unarchived |
| `on_create` | On create |
| `on_create_or_write` | On create and edit |
| `on_write` | On update |
| `on_unlink` | On deletion |
| `on_change` | On UI change |
| `on_time` | Based on date field |
| `on_time_created` | After creation |
| `on_time_updated` | After last update |
| `on_message_received` | On incoming message |
| `on_message_sent` | On outgoing message |
| `on_webhook` | On webhook |

### `trg_date_range_mode` (Delay mode)

| Value | Label |
|---|---|
| `after` | After |
| `before` | Before |

### `trg_date_range_type` (Delay unit)

| Value | Label |
|---|---|
| `minutes` | Minutes |
| `hour` | Hours |
| `day` | Days |
| `month` | Months |

## Operations (50)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_check_trigger` | validation | self | `base_automation` | constrains: `trigger`, `model_id` |  |
| `_check_action_server_model` | validation | self | `base_automation` | constrains: `model_id`, `action_server_ids` |  |
| `_compute_url` | computation | self | `base_automation` | depends: `trigger`, `webhook_uuid` |  |
| `_inverse_model_name` | inverse computation | self | `base_automation` |  |  |
| `_check_time_trigger` | validation | self | `base_automation` | constrains: `trigger`, `trg_date_range` |  |
| `_check_trigger_state` | validation | self | `base_automation` | constrains: `trigger`, `action_server_ids` |  |
| `_compute_action_server_ids` | computation | self | `base_automation` | depends: `model_id` | When changing / setting model, remove actions that are not targeting the same model anymore. |
| `_compute_trg_date_id` | computation | self | `base_automation` | depends: `trigger` |  |
| `_onchange_trg_date_range_data` | on change | self | `base_automation` | onchange: `trg_date_range` |  |
| `_compute_trg_date_range_data` | computation | self | `base_automation` | depends: `trigger` |  |
| `_compute_trg_date_calendar_id` | computation | self | `base_automation` | depends: `trigger`, `trg_date_id`, `trg_date_range_type` |  |
| `_compute_trg_selection_field_id` | computation | self | `base_automation` | depends: `trigger` |  |
| `_compute_trg_field_ref` | computation | self | `base_automation` | depends: `trigger` |  |
| `_compute_trg_field_ref_model_name` | computation | self | `base_automation` | depends: `trigger`, `trg_field_ref` |  |
| `_compute_filter_pre_domain` | computation | self | `base_automation` | depends: `trigger`, `trg_field_ref` |  |
| `_compute_filter_domain` | computation | self | `base_automation` | depends: `trigger`, `trg_selection_field_id`, `trg_field_ref` |  |
| `_compute_on_change_field_ids` | computation | self | `base_automation` | depends: `model_id`, `trigger`, `filter_domain` |  |
| `_compute_trigger_field_ids` | computation | self | `base_automation` | depends: `model_id`, `trigger`, `filter_domain` |  |
| `_compute_trigger` | computation | self | `base_automation` | depends: `model_id` |  |
| `_onchange_domain` | on change | self | `base_automation` | onchange: `filter_domain` |  |
| `_onchange_trigger` | on change | self | `base_automation` | onchange: `trigger` |  |
| `_onchange_trigger_or_actions` | on change | self | `base_automation` | onchange: `trigger`, `action_server_ids` |  |
| `_has_trigger_onchange` | internal rule | self | `base_automation` |  |  |
| `create` | lifecycle override | self, vals_list | `base_automation` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base_automation` |  |  |
| `unlink` | lifecycle override | self | `base_automation` |  |  |
| `copy` | lifecycle override | self, default | `base_automation` |  | Copy the actions of the automation while copying the automation itself. |
| `action_open_scheduled_action` | user action | self | `base_automation` |  |  |
| `action_rotate_webhook_uuid` | user action | self | `base_automation` |  |  |
| `action_view_webhook_logs` | user action | self | `base_automation` |  |  |
| `_get_trigger_specific_field` | preparation rule | self | `base_automation` |  |  |
| `_prepare_loggin_values` | preparation rule | self, **values | `base_automation` |  |  |
| `_execute_webhook` | internal rule | self, payload | `base_automation` |  | Execute the webhook for the given payload. The payload is a dictionnary that can be used by the `record_getter` to identify the record on which the automation should be run. |
| `_update_cron` | internal rule | self | `base_automation` |  | Activate the cron job depending on whether there exists automation rules based on time conditions.  Also update its frequency according to the smallest automation delay, or restore the default 4 hours if there is no time based automation. |
| `_update_registry` | internal rule | self | `base_automation` |  | Update the registry after a modification on automation rules. |
| `_get_actions` | preparation rule | self, records, triggers | `base_automation` |  | Return the automations of the given triggers for records' model. The returned automations' context contain an object to manage processing. |
| `_get_eval_context` | preparation rule | self, payload | `base_automation` |  | Prepare the context used when evaluating python code :returns: dict -- evaluation context given to safe_eval |
| `_get_cron_interval` | preparation rule | self, automations | `base_automation` |  | Return the expected time interval used by the cron, in minutes or hours. |
| `_filter_pre` | internal rule | self, records, feedback | `base_automation` |  | Filter the records that satisfy the precondition of automation `self`. |
| `_filter_post` | internal rule | self, records, feedback | `base_automation` |  |  |
| `_filter_post_export_domain` | internal rule | self, records, feedback | `base_automation` |  | Filter the records that satisfy the postcondition of automation `self`. |
| `_add_postmortem` | internal rule | self, e | `base_automation` | model |  |
| `_process` | background operation | self, records, domain_post | `base_automation` |  | Process automation `self` on the `records` that have not been done yet. |
| `_check_trigger_fields` | validation | self, record | `base_automation` |  | Return whether any of the trigger fields has been modified on `record`. |
| `_register_hook` | internal rule | self | `base_automation` |  | Patch models that should trigger action rules based on creation, modification, deletion of records and form onchanges. |
| `_unregister_hook` | internal rule | self | `base_automation` |  | Remove the patches installed by _register_hook() |
| `_get_calendar` | preparation rule | self, automation, record | `base_automation` | model |  |
| `_check` | validation | self, automatic, use_new_cursor | `base_automation` |  |  |
| `_search_time_based_automation_records` | search rule | self, until | `base_automation` |  |  |
| `_cron_process_time_based_actions` | background operation | self | `base_automation` | model | Execute the time-based automations. |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_trigger` | ValidationError | Mail event can not be configured on model %s. Only models with discussion feature can be used. | `base_automation` |
| `_check_action_server_model` | ValidationError | Target model of actions %(action_names)s are different from rule model. | `base_automation` |
| `_check_time_trigger` | ValidationError | Delay must be positive. Set 'Delay mode' to 'Before' to negate the delay. | `base_automation` |
| `_check_trigger_state` | ValidationError | Following child actions have warnings: %(children)s | `base_automation` |
| `_check_trigger_state` | ValidationError | "On live update" automation rules can only be used with "Execute Python Code" action type. | `base_automation` |
| `_check_trigger_state` | ValidationError | Email, follower or activity action types cannot be used when deleting records, as there are no more records to apply these changes to! | `base_automation` |
| `action_open_scheduled_action` | MissingError | message | `base_automation` |
| `_execute_webhook` | ValidationError | No record to run the automation on was found. | `base_automation` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_system` | yes | yes | yes | yes | `base_automation` |

## Views (4)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base_automation.view_base_automation_form` | form |  | `active`, `model_name`, `name`, `model_id`, `model_id`, `trigger`, `trg_date_id`, `trg_selection_field_id`, `trg_field_ref`, `trg_field_ref_model_name`, `url`, `log_webhook_calls`, `trg_date_range`, `trg_date_range_type`, `trg_date_range_mode`, `trg_date_calendar_id`, `filter_pre_domain`, `previous_domain`, `filter_domain`, `filter_domain`, `trigger_field_ids`, `on_change_field_ids`, `record_getter`, `action_server_ids`, `description` | `Logs`, `Renew`, `Scheduled action` |  | `base_automation` |
| `base_automation.view_base_automation_tree` | list |  | `name`, `trigger`, `model_id` |  |  | `base_automation` |
| `base_automation.view_base_automation_kanban` | kanban |  | `active`, `name`, `model_id`, `trigger`, `on_change_field_ids`, `trg_selection_field_id`, `trg_field_ref`, `trigger_field_ids`, `trg_date_range`, `trg_date_range_type`, `trg_date_range_mode`, `trg_date_id`, `trg_date_range`, `trg_date_range_type`, `trigger`, `trg_date_id`, `trg_date_calendar_id`, `action_server_ids` |  |  | `base_automation` |
| `base_automation.view_base_automation_search` | search |  | `name`, `model_id` |  | `Include Archived` | `base_automation` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base_automation.base_automation_act` | Automation Rules | kanban,list,form |  | `{'search_default_inactive': 1}` |  | `base_automation` |

## Scheduled jobs

| Job | Name | Every | Operation | Priority |
|---|---|---|---|---|
| `base_automation.ir_cron_data_base_automation_check` | Automation Rules: check and execute | 4 hours | `_cron_process_time_based_actions` |  |

Machine-readable definition: `../../../schemas/data/entities/base.automation.json`; views: `../../../schemas/interfaces/views/base.automation.json`.
