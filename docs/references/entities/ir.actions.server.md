# Server Actions (`ir.actions.server`)

**Transport name:** `ir.actions.server`  
**Storage name:** `ir_act_server`  
**Kind:** persistent entity (one table)  
**Defined by package:** `base`  
**Extended by packages:** `mail`, `sms`, `base_automation`, `website`

Description: Server Actions

## Identity and behavior

- Mixins (classical inheritance): `ir.actions.actions`, `mail.thread`, `mail.activity.mixin`
- Default ordering: `sequence,name,id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (57)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Name | single line text |  | computed by rule `_compute_name` and stored; changes are tracked in the message thread; extended by packages `mail` |
| `automated_name` | Automated Name | single line text |  | computed by rule `_compute_name` and stored |
| `type` | Type | single line text |  | default `ir.actions.server` |
| `usage` | Usage | selection |  | required; default `ir_actions_server`; on delete of the target: {"base_automation": "cascade"}; extended by packages `base_automation` |
| `state` | Type | selection |  | required; changes are tracked in the message thread; on delete of the target: {"sms": "cascade"}; Help: Type of server action. The following values are available: - 'Update a Record': update the values of a record - 'Create Activity': create an activity (Discuss) - 'Send Email': post a message, a note or send an email (Discuss) - 'Send SMS': send SMS, log them on documents (SMS)- 'Add/Remove Followers': add or remove followers to a record (Discuss) - 'Create Record': create a new record with new values - 'Execute Code': a block of Python code that will be executed - 'Send Webhook Notification': send a POST request to an external system, also known as a Webhook - 'Multi Actions': define an action that triggers several other server actions; extended by packages `mail`, `sms` |
| `allowed_states` | Allowed states | structured document |  | computed by rule `_compute_allowed_states` (not stored) |
| `sequence` | Sequence | integer |  | default `5`; Help: When dealing with multiple actions, the execution order is based on the sequence. Low number means high priority. |
| `model_id` | Model | many to one | `ir.model` | required; changes are tracked in the message thread; indexed; on delete of the target: cascade; Help: Model on which the server action runs.; extended by packages `mail` |
| `available_model_ids` | Available Models | many to many | `ir.model` | computed by rule `_compute_available_model_ids` (not stored) |
| `model_name` | Model Name | single line text |  | related through path `model_id.model` |
| `warning` | Warning | multi line text |  | computed by rule `_compute_warning` (not stored); recursive dependency |
| `ir_cron_ids` | Scheduled Action | one to many | `ir.cron` | inverse field `ir_actions_server_id` |
| `code` | Python Code | multi line text |  | visible only to groups `base.group_system`; Help: Write Python code that the action will execute. Some variables are available for use; help about python expression is given in the help tab. |
| `show_code_history` | Show Code History | boolean |  | computed by rule `_compute_show_code_history` (not stored) |
| `parent_id` | Parent Action | many to one | `ir.actions.server` | indexed; on delete of the target: cascade |
| `child_ids` | Child Actions | one to many | `ir.actions.server` | restricted by domain `lambda self: str(self._get_children_domain())`; inverse field `parent_id`; Help: Child server actions that will be executed. Note that the last return returned action value will be used as global return value. |
| `crud_model_id` | Record to Create | many to one | `ir.model` | computed by rule `_compute_crud_relations` and stored; writable through an inverse rule; changes are tracked in the message thread; Help: Specify which kind of record should be created. Set this field only to specify a different model than the base model.; extended by packages `mail` |
| `crud_model_name` | Target Model Name | single line text |  | read only; related through path `crud_model_id.model` |
| `link_field_id` | Link Field | many to one | `ir.model.fields` | changes are tracked in the message thread; Help: Specify a field used to link the newly created record on the record used by the server action.; extended by packages `mail` |
| `group_ids` | Allowed Groups | many to many | `res.groups` | association table `ir_act_server_group_rel`; Help: Groups that can execute the server action. Leave empty to allow everybody. |
| `update_field_id` | Field to Update | many to one | `ir.model.fields` | computed by rule `_compute_crud_relations` and stored; on delete of the target: cascade |
| `update_path` | Field to Update Path | single line text |  | default computed dynamically (_default_update_path); changes are tracked in the message thread; Help: Path to the field to update, e.g. 'partner_id.name'; extended by packages `mail` |
| `update_related_model_id` | Update Related Model | many to one | `ir.model` | computed by rule `_compute_crud_relations` and stored |
| `update_field_type` | Update Field Type | selection |  | read only; related through path `update_field_id.ttype` |
| `update_m2m_operation` | Many2many Operations | selection |  | default `add` |
| `update_boolean_value` | Boolean Value | selection |  | default `true` |
| `value` | Value | multi line text |  | changes are tracked in the message thread; Help: For Python expressions, this field may hold a Python expression that can use the same values as for the code field on the server action,e.g. `env.user.name` to set the current user's name as the value or `record.id` to set the ID of the record on which the action is run.  For Static values, the value will be used directly without evaluation, e.g.`42` or `My custom name` or the selected record.; extended by packages `mail` |
| `evaluation_type` | Value Type | selection |  | default `value`; changes are tracked in the message thread; extended by packages `mail` |
| `html_value` | Hypertext markup language Value | rich text |  |  |
| `sequence_id` | Sequence to use | many to one | `ir.sequence` |  |
| `resource_ref` | Record | reference |  | writable through an inverse rule; values provided by rule `_selection_target_model` |
| `selection_value` | Custom Value | many to one | `ir.model.fields.selection` | writable through an inverse rule; on delete of the target: cascade; restricted by domain `[("field_id", "=", update_field_id)]` |
| `value_field_to_show` | Value Field To Show | selection |  | computed by rule `_compute_value_field_to_show` (not stored) |
| `webhook_url` | Webhook uniform resource locator | single line text |  | changes are tracked in the message thread; Help: URL to send the POST request to.; extended by packages `mail` |
| `webhook_field_ids` | Webhook Fields | many to many | `ir.model.fields` | association table `ir_act_server_webhook_field_rel`; Help: Fields to send in the POST request. The id and model of the record are always sent as '_id' and '_model'. The name of the action that triggered the webhook is always sent as '_name'. |
| `webhook_sample_payload` | Sample Payload | multi line text |  | computed by rule `_compute_webhook_sample_payload` (not stored) |
| `followers_type` | Followers Type | selection |  | computed by rule `_compute_followers_type` and stored; Help: - Specific Followers: select specific contacts to add/remove from record's followers.             - Dynamic Followers: all contacts of the chosen record's field will be added/removed from followers. |
| `followers_partner_field_name` | Followers Field | single line text |  | computed by rule `_compute_followers_info` and stored |
| `partner_ids` | Partner | many to many | `res.partner` | computed by rule `_compute_followers_info` and stored |
| `template_id` | Email Template | many to one | `mail.template` | computed by rule `_compute_template_id` and stored; on delete of the target: set null; restricted by domain `[('model_id', '=', model_id)]` |
| `mail_post_autofollow` | Subscribe Recipients | boolean |  | computed by rule `_compute_mail_post_autofollow` and stored |
| `mail_post_method` | Send Email As | selection |  | computed by rule `_compute_mail_post_method` and stored |
| `activity_type_id` | Activity Type | many to one | `mail.activity.type` | computed by rule `_compute_activity_info` and stored; on delete of the target: restrict; restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', model_name)]` |
| `activity_summary` | Title | single line text |  | computed by rule `_compute_activity_info` and stored |
| `activity_note` | Note | rich text |  | computed by rule `_compute_activity_info` and stored |
| `activity_date_deadline_range` | Due Date In | integer |  | computed by rule `_compute_activity_info` and stored |
| `activity_date_deadline_range_type` | Due type | selection |  | computed by rule `_compute_activity_info` and stored |
| `activity_user_type` | User Type | selection |  | computed by rule `_compute_activity_info` and stored; Help: Use 'Specific User' to always assign the same user on the next activity. Use 'Dynamic User' to specify the field name of the user to choose on the record. |
| `activity_user_id` | Responsible | many to one | `res.users` | computed by rule `_compute_activity_user_info` and stored |
| `activity_user_field_name` | User Field | single line text |  | computed by rule `_compute_activity_user_info` and stored |
| `sms_template_id` | text message Template | many to one | `sms.template` | computed by rule `_compute_sms_template_id` and stored; on delete of the target: set null; restricted by domain `[('model_id', '=', model_id)]` |
| `sms_method` | Send text message As | selection |  | computed by rule `_compute_sms_method` and stored |
| `base_automation_id` | Automation Rule | many to one | `base.automation` | indexed (btree_not_null); on delete of the target: cascade |
| `xml_id` | External identifier | single line text |  | computed by rule `_compute_xml_id` (not stored); Help: ID of the action if defined in a XML file |
| `website_path` | Website Path | single line text |  |  |
| `website_url` | Website Url | single line text |  | computed by rule `_get_website_url` (not stored); Help: The full URL to access the server action through the website. |
| `website_published` | Available on the Website | boolean |  | not copied on duplication; Help: A code server action can be executed from the website, using a dedicated controller. The address is <base>/website/action/<website_path>. Set this field as True to allow users to run this action. If it is set to False the action cannot be run through the website. |

## Selection values

### `usage` (Usage)

| Value | Label |
|---|---|
| `ir_actions_server` | Server Action |
| `ir_cron` | Scheduled Action |
| `base_automation` | Automation Rule |

### `state` (Type)

| Value | Label |
|---|---|
| `object_write` | Update Record |
| `object_create` | Create Record |
| `object_copy` | Duplicate Record |
| `code` | Execute Code |
| `webhook` | Send Webhook Notification |
| `multi` | Multi Actions |
| `next_activity` | Create Activity |
| `mail_post` | Send Email |
| `followers` | Add Followers |
| `remove_followers` | Remove Followers |
| `sms` | Send SMS |

### `update_m2m_operation` (Many2many Operations)

| Value | Label |
|---|---|
| `add` | Adding |
| `remove` | Removing |
| `set` | Setting it to |
| `clear` | Clearing it |

### `update_boolean_value` (Boolean Value)

| Value | Label |
|---|---|
| `true` | Yes (True) |
| `false` | No (False) |

### `evaluation_type` (Value Type)

| Value | Label |
|---|---|
| `value` | Update |
| `sequence` | Sequence |
| `equation` | Compute |

### `value_field_to_show` (Value Field To Show)

| Value | Label |
|---|---|
| `value` | value |
| `html_value` | html_value |
| `sequence_id` | sequence_id |
| `resource_ref` | reference |
| `update_boolean_value` | update_boolean_value |
| `selection_value` | selection_value |

### `followers_type` (Followers Type)

| Value | Label |
|---|---|
| `specific` | Specific Followers |
| `generic` | Dynamic Followers |

### `mail_post_method` (Send Email As)

| Value | Label |
|---|---|
| `email` | Email |
| `comment` | Message |
| `note` | Note |

### `activity_date_deadline_range_type` (Due type)

| Value | Label |
|---|---|
| `days` | Days |
| `weeks` | Weeks |
| `months` | Months |

### `activity_user_type` (User Type)

| Value | Label |
|---|---|
| `specific` | Specific User |
| `generic` | Dynamic User (based on record) |

### `sms_method` (Send text message As)

| Value | Label |
|---|---|
| `sms` | SMS (without note) |
| `comment` | SMS (with note) |
| `note` | Note only |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Operations (63)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_update_path` | preparation rule | self | `base` | model |  |
| `create` | lifecycle override | self, vals_list | `base` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `base` |  |  |
| `_compute_show_code_history` | computation | self | `base` | depends: `state`, `code` |  |
| `_warning_depends` | internal rule | self | `base_automation`, `base`, `mail`, `sms` | model |  |
| `_get_warning_messages` | preparation rule | self | `base_automation`, `base`, `mail`, `sms` |  |  |
| `_compute_allowed_states` | computation | self | `base` |  |  |
| `_compute_warning` | computation | self | `base` | depends: |  |
| `_get_children_domain` | preparation rule | self | `base_automation`, `base` | model |  |
| `_generate_action_name` | internal rule | self | `base`, `mail`, `sms` |  |  |
| `_name_depends` | internal rule | self | `base`, `mail`, `sms` |  |  |
| `_compute_name` | computation | self | `base` | depends: |  |
| `_onchange_name` | on change | self | `base` | onchange: `name` |  |
| `_compute_available_model_ids` | computation | self | `base_automation`, `base`, `mail`, `sms` | depends: `state`; depends: `usage` | Stricter model limit: based on automation rule |
| `_compute_crud_relations` | computation | self | `base` | depends: `model_id`, `update_path`, `state` | Compute the crud_model_id and update_field_id fields.  The crud_model_id is the model on which the action will create or update records. In the case of record creation, it is the same as the main model of the action. For record update, it will be the model linked to the last field in the update_path. This is only used for object_create and object_write actions. The update_field_id is the field at the end of the update_path that will be updated by the action - only used for object_write actions. |
| `_traverse_path` | internal rule | self | `base` |  | Traverse the update_path to find the target model and field.  :return: a tuple (model, field) where model is the target model and field is the target field |
| `_get_relation_chain` | preparation rule | self, searched_field_name | `base` |  |  |
| `_compute_webhook_sample_payload` | computation | self | `base` | depends: `state`, `model_id`, `webhook_field_ids`, `name` |  |
| `_check_python_code` | validation | self | `base` | constrains: `code` |  |
| `_check_children` | validation | self | `base` | constrains: `parent_id`, `child_ids` |  |
| `_get_readable_fields` | preparation rule | self | `base` |  |  |
| `_get_runner` | preparation rule | self | `base` |  |  |
| `create_action` | operation | self | `base` |  | Create a contextual action for each server action. |
| `unlink_action` | operation | self | `base` |  | Remove the contextual actions created for the server actions. |
| `history_wizard_action` | operation | self | `base` |  |  |
| `_run_action_code_multi` | background operation | self, eval_context | `base`, `website` | model | Override to allow returning response the same way action is already returned by the basic server action behavior. Note that response has priority over action, avoid using both. |
| `_run_action_multi` | background operation | self, eval_context | `base` |  |  |
| `_run_action_object_write` | background operation | self, eval_context | `base` |  | Apply specified write changes to active_id. |
| `_run_action_webhook` | background operation | self, eval_context | `base` |  | Send a post request with a read of the selected field on active_id. |
| `_run_action_object_copy` | background operation | self, eval_context | `base` |  | Duplicate specified model object. If applicable, link active_id.<self.link_field_id> to the new record. |
| `_run_action_object_create` | background operation | self, eval_context | `base` |  | Create specified model object with specified name contained in value.  If applicable, link active_id.<self.link_field_id> to the new record. |
| `_get_eval_context` | preparation rule | self, action | `base_automation`, `base`, `mail`, `website` | model | Prepare the context used when evaluating python code, like the python formulas or code server actions.  :param action: the current server action :type action: browse record :returns: dict -- evaluation context given to (safe_)safe_eval |
| `run` | operation | self | `base` |  | Runs the server action. For each server action, the :samp:`_run_action_{TYPE}[_multi]` method is called. This allows easy overriding of the server actions.  The `_multi` suffix means the runner can operate on multiple records, otherwise if there are multiple records the runner will be called once for each.  The call context should contain the following keys:  active_id     id of the current object (single mode) active_model     current model that should equal the action's model active_ids (optional)    ids of the current records (mass mode). If `active_ids` and    `active_id` are present |
| `_run` | internal rule | self, records, eval_context | `base` |  |  |
| `_can_execute_action_on_records` | internal rule | self, records | `base` |  |  |
| `_compute_value_field_to_show` | computation | self | `base` | depends: `evaluation_type`, `update_field_id` |  |
| `_selection_target_model` | internal rule | self | `base` | model |  |
| `_set_crud_model_id` | on change | self | `base` | onchange: `crud_model_id` |  |
| `_set_resource_ref` | on change | self | `base` | onchange: `resource_ref` |  |
| `_set_selection_value` | on change | self | `base` | onchange: `selection_value` |  |
| `_eval_value` | internal rule | self, eval_context | `base` |  |  |
| `copy_data` | lifecycle override | self, default | `base` |  |  |
| `action_open_parent_action` | user action | self | `base` |  |  |
| `action_open_scheduled_action` | user action | self | `base` |  |  |
| `_compute_template_id` | computation | self | `mail` | depends: `model_id`, `state` |  |
| `_compute_mail_post_autofollow` | computation | self | `mail` | depends: `state`, `mail_post_method` |  |
| `_compute_mail_post_method` | computation | self | `mail` | depends: `state` |  |
| `_compute_followers_type` | computation | self | `mail` | depends: `model_id`, `state` |  |
| `_compute_followers_info` | computation | self | `mail` | depends: `followers_type` |  |
| `_compute_activity_info` | computation | self | `mail` | depends: `model_id`, `state` |  |
| `_compute_activity_user_info` | computation | self | `mail` | depends: `model_id`, `activity_user_type` |  |
| `_run_action_followers_multi` | background operation | self, eval_context | `mail` |  |  |
| `_run_action_remove_followers_multi` | background operation | self, eval_context | `mail` |  |  |
| `_is_recompute` | internal rule | self | `mail` |  | When an activity is set on update of a record, update might be triggered many times by recomputes. When need to know it to skip these steps. Except if the computed field is supposed to trigger the action |
| `_run_action_mail_post_multi` | background operation | self, eval_context | `mail` |  |  |
| `_run_action_next_activity` | background operation | self, eval_context | `mail` |  |  |
| `_compute_sms_template_id` | computation | self | `sms` | depends: `model_id`, `state` |  |
| `_compute_sms_method` | computation | self | `sms` | depends: `state` |  |
| `_run_action_sms_multi` | background operation | self, eval_context | `sms` |  |  |
| `action_open_automation` | user action | self | `base_automation` |  |  |
| `_compute_xml_id` | computation | self | `website` |  |  |
| `_compute_website_url` | computation | self, website_path, xml_id | `website` |  |  |
| `_get_website_url` | computation | self | `website` | depends: `state`, `website_published`, `website_path`, `xml_id` |  |

## Validation and error messages (8)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_get_relation_chain` | ValidationError | The path contained by the field '%(searched_field)s' contains a non-relational field (%(current_field)s) that is not the last field in the path. You can't traverse non-relational fields (even in the quantum realm). Make sure only the last field in the path is non-relational. | `base` |
| `_check_python_code` | ValidationError | msg | `base` |
| `_check_children` | ValidationError | Recursion found in child server actions | `base` |
| `_check_children` | ValidationError | Following child actions have warnings: %(children)s | `base` |
| `_run_action_webhook` | UserError | I'll be happy to send a webhook for you, but you really need to give me a URL to reach out to... | `base` |
| `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. | `base` |
| `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. | `base` |
| `_can_execute_action_on_records` | AccessError | You don't have enough access rights to run this action. | `base` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `group_system` | yes | yes | yes | yes | `base` |

## Views (9)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `base.view_server_action_form` | form |  | `automated_name`, `name`, `model_id`, `model_id`, `group_ids`, `allowed_states`, `state`, `warning`, `evaluation_type`, `update_path`, `update_field_id`, `update_related_model_id`, `update_m2m_operation`, `value`, `html_value`, `sequence_id`, `resource_ref`, `selection_value`, `update_boolean_value`, `value`, `crud_model_id`, `value`, `resource_ref`, `link_field_id`, `webhook_url`, `webhook_field_ids`, `webhook_sample_payload`, `child_ids`, `code` | `Create Contextual Action`, `Remove Contextual Action`, `Run`, `Code History`, `action_open_parent_action`, `action_open_scheduled_action` |  | `base` |
| `base.view_server_action_kanban` | kanban |  | `state`, `evaluation_type`, `value`, `value_field_to_show`, `update_field_type`, `update_m2m_operation`, `sequence`, `name` | `delete` |  | `base` |
| `base.view_server_action_tree` | list |  | `name`, `model_id`, `state`, `usage` |  |  | `base` |
| `base.view_server_action_search` | search |  | `name`, `model_id`, `state` |  | `Top-level actions`, `Action Type`, `Model`, `Usage` | `base` |
| `base_automation.view_server_action_form` | xpath | `base.view_server_action_form` |  | `action_open_automation` |  | `base_automation` |
| `mail.view_server_action_form_template` | xpath | `base.view_server_action_form` |  |  |  | `mail` |
| `sms.ir_actions_server_view_form` | xpath | `base.view_server_action_form` | `sms_template_id`, `sms_method` |  |  | `sms` |
| `website.view_server_action_form_website` | data | `base.view_server_action_form` | `website_published`, `xml_id`, `website_path`, `website_url` |  |  | `website` |
| `website.view_server_action_search_website` | xpath | `base.view_server_action_search` |  |  | `Website` | `website` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `base.action_server_action` | Server Actions | list,form |  | `{                 'key':'server_action',                 'search_default_toplevel_actions': 1,             }` |  | `base` |

Machine-readable definition: `../../../schemas/data/entities/ir.actions.server.json`; views: `../../../schemas/interfaces/views/ir.actions.server.json`.
