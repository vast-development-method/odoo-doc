# Activity (`mail.activity`)

**Transport name:** `mail.activity`  
**Storage name:** `mail_activity`  
**Kind:** persistent entity (one table)  
**Defined by package:** `mail`  
**Extended by packages:** `calendar`, `crm`, `website_slides`

Description: Activity

## Identity and behavior

- Default ordering: `date_deadline ASC, id ASC`
- Display name field: `summary`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (27)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `res_model_id` | Document Model | many to one | `ir.model` | indexed; on delete of the target: cascade |
| `res_model` | Related Document Model | single line text |  | read only; related through path `res_model_id.model` and stored; indexed; precomputed before insertion |
| `res_id` | Related Document identifier | many to one by reference |  | indexed |
| `res_name` | Document Name | single line text |  | read only; computed by rule `_compute_res_name` and stored |
| `activity_type_id` | Activity Type | many to one | `mail.activity.type` | default computed dynamically (_default_activity_type); on delete of the target: restrict; restricted by domain `['\|', ('res_model', '=', False), ('res_model', '=', res_model)]` |
| `activity_category` | Activity Category | selection |  | read only; related through path `activity_type_id.category` |
| `activity_decoration` | Activity Decoration | selection |  | read only; related through path `activity_type_id.decoration_type` |
| `icon` | Icon | single line text |  | read only; related through path `activity_type_id.icon` |
| `summary` | Summary | single line text |  |  |
| `note` | Note | rich text |  |  |
| `date_deadline` | Due Date | date |  | required; default computed dynamically (fields.Date.context_today); indexed |
| `date_done` | Done Date | date |  | computed by rule `_compute_date_done` and stored |
| `feedback` | Feedback | multi line text |  |  |
| `automated` | Automated activity | boolean |  | read only; Help: Indicates this activity has been created automatically and not by any user. |
| `attachment_ids` | Attachments | many to many | `ir.attachment` | association table `activity_attachment_rel` |
| `user_id` | Assigned to | many to one | `res.users` | indexed; on delete of the target: cascade |
| `user_tz` | Timezone | selection |  | related through path `user_id.tz` and stored |
| `state` | State | selection |  | computed by rule `_compute_state` (not stored) |
| `recommended_activity_type_id` | Recommended Activity Type | many to one | `mail.activity.type` |  |
| `previous_activity_type_id` | Previous Activity Type | many to one | `mail.activity.type` | read only |
| `has_recommended_activities` | Next activities available | boolean |  | computed by rule `_compute_has_recommended_activities` (not stored) |
| `mail_template_ids` | Mail Template | many to many |  | read only; related through path `activity_type_id.mail_template_ids` |
| `chaining_type` | Chaining Type | selection |  | read only; related through path `activity_type_id.chaining_type` |
| `can_write` | Can Write | boolean |  | computed by rule `_compute_can_write` (not stored) |
| `active` | Active | boolean |  | default `True` |
| `calendar_event_id` | Calendar Meeting | many to one | `calendar.event` | indexed (btree_not_null); on delete of the target: cascade |
| `request_partner_id` | Requesting Partner | many to one | `res.partner` | on delete of the target: cascade |

## Selection values

### `state` (State)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |
| `done` | Done |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (2)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_check_res_id_is_set_if_model` | Constraint | `CHECK(             (COALESCE(res_model, '') <> '' AND (res_id IS NOT NULL AND res_id != 0)) OR             (COALESCE(res_model, '') = '' AND (res_id IS NULL OR res_id = 0))         )` | Activities have to be linked to records with a not null res_id. | `mail` |
| `_check_user_id_is_set_if_model` | Constraint | `CHECK(             (COALESCE(res_model, '') <> '' OR user_id IS NOT NULL)         )` | Activities must be assigned if not attached to a document. | `mail` |

## Operations (40)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `default_get` | lifecycle override | self, fields | `mail` | model |  |
| `_default_activity_type` | preparation rule | self | `mail` | model |  |
| `_default_activity_type_for_model` | preparation rule | self, model | `mail` | model | Take first one found, ordered by sequence. Keep it simple. |
| `_compute_has_recommended_activities` | on change | self | `mail` | onchange: `previous_activity_type_id` |  |
| `_onchange_previous_activity_type_id` | on change | self | `mail` | onchange: `previous_activity_type_id` |  |
| `_compute_date_done` | computation | self | `mail` | depends: `active` |  |
| `_compute_res_name` | computation | self | `mail` | depends: `res_model`, `res_id` |  |
| `_compute_state` | computation | self | `mail` | depends: `active`, `date_deadline` |  |
| `_compute_state_from_date` | computation | self, date_deadline, tz | `mail` | model |  |
| `_compute_can_write` | computation | self | `mail` | depends: `res_model`, `res_id`, `user_id` |  |
| `_onchange_activity_type_id` | on change | self | `mail` | onchange: `activity_type_id` |  |
| `_onchange_recommended_activity_type_id` | on change | self | `mail` | onchange: `recommended_activity_type_id` |  |
| `_check_access` | validation | self, operation | `mail` |  | Determine the subset of `self` for which `operation` is allowed. A custom implementation is done on activities as this document has some access rules and is based on related document for activities that are not covered by those rules.  Access on activities are the following :    * read: access rule AND (assigned to user OR read rights on related documents);   * write: access rule OR (`mail_post_access` or write) rights on related documents);   * create: access rule AND (`mail_post_access` or write) right on related documents;   * unlink: access rule OR (`mail_post_access` or write) r |
| `_make_access_error` | internal rule | self, operation | `mail` |  |  |
| `create` | lifecycle override | self, vals_list | `mail` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `calendar`, `mail` |  |  |
| `unlink` | lifecycle override | self | `mail` |  |  |
| `_search` | search rule | self, domain, offset, limit, order, bypass_access, **kwargs | `mail` | model | Override that adds specific access rights of mail.activity, to remove ids uid could not see according to our custom rules. Please refer to :meth:`_check_access` for more details about those rules.  The method is inspired by what has been done on mail.message. |
| `_compute_display_name` | computation | self | `mail` | depends: `summary`, `activity_type_id` |  |
| `action_notify` | user action | self | `mail` |  |  |
| `action_done` | user action | self | `mail` |  | Wrapper without feedback because web button add context as parameter, therefore setting context to feedback |
| `action_done_redirect_to_other` | user action | self | `mail` |  | Mark activity as done and return action mail.mail_activity_without_access_action.  Goal: Unless "keep done" activity is enabled, when marking an activity as done, the activity is deleted and can no more be displayed. To overcome this, we return an action that will launch the list view displaying the activities corresponding to the active_ids from the context (i.e.: the remaining "other activities"). If the right context is not available, we recompute the activities to display. |
| `action_feedback` | user action | self, feedback, attachment_ids | `mail` |  |  |
| `action_done_schedule_next` | user action | self | `mail` |  | Wrapper without feedback because web button add context as parameter, therefore setting context to feedback |
| `action_feedback_schedule_next` | user action | self, feedback, attachment_ids | `mail` |  |  |
| `_action_done` | internal rule | self, feedback, attachment_ids | `calendar`, `mail` |  | Private implementation of marking activity as done: posting a message, archiving activity (since done), and eventually create the automatical next activity (depending on config). :param feedback: optional feedback from user when marking activity as done :param attachment_ids: list of ir.attachment ids to attach to the posted mail.message :returns (messages, activities) where     - messages is a recordset of posted mail.message     - activities is a recordset of mail.activity of forced automically created activities |
| `action_close_dialog` | user action | self | `mail` | readonly |  |
| `action_open_document` | user action | self | `mail` | readonly | Opens the related record based on the model and ID, or activity if user has no access to the related record. |
| `action_reschedule_today` | user action | self | `mail` |  |  |
| `action_reschedule_tomorrow` | user action | self | `mail` |  |  |
| `action_reschedule_nextweek` | user action | self | `mail` |  |  |
| `action_cancel` | user action | self | `mail` |  |  |
| `activity_format` | operation | self | `mail` | readonly |  |
| `_to_store_defaults` | internal rule | self, target | `calendar`, `mail`, `website_slides` |  |  |
| `get_activity_data` | operation | self, res_model, domain, limit, offset, fetch_done | `mail` | readonly; model | Get aggregate data about records and their activities.  The goal is to fetch and compute aggregated data about records and their activities to display them in the activity views and the chatter. For example, the activity view displays it as a table with columns and rows being respectively the activity_types and the activity_res_ids, and the grouped_activities being the table entries with the aggregated data.  :param str res_model: model of the records to fetch :param list domain: record search domain :param int limit: maximum number of records to fetch :param int offset: offset of the first re |
| `_classify_by_model` | internal rule | self | `mail` |  | To ease batch computation of various activities related methods they are classified by model. Activities not linked to a valid record through res_model / res_id are ignored.  :returns: for each model having at least one activity in self, have   a sub-dict containing     * activities: activities related to that model;     * record IDs: record linked to the activities of that model, in same       order; :rtype: dict |
| `_prepare_next_activity_values` | preparation rule | self | `mail` |  | Prepare the next activity values based on the current activity record and applies _onchange methods :returns a dict of values for the new activity |
| `_gc_delete_old_overdue_activities` | background operation | self | `mail` | autovacuum | Delete old overdue activities - If the config_parameter is deleted or 0, the user doesn't want to run this gc routine - If the config_parameter is set to a negative number, it's an invalid value, we skip the gc routine - If the config_parameter is set to a positive number, we delete only overdue activities which deadline is older than X years |
| `action_create_calendar_event` | user action | self | `calendar`, `crm` |  | Small override of the action that creates a calendar.  If the activity is linked to a crm.lead through the "opportunity_id" field, we include in the action context the default values used when scheduling a meeting from the crm.lead form view. e.g: It will set the partner_id of the crm.lead as default attendee of the meeting. |
| `unlink_w_meeting` | operation | self | `calendar` |  |  |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_user` | yes | yes | yes | yes | `mail` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| mail.activity: user: write/unlink only (created or assigned) | `[Command.link(ref('base.group_user'))]` | `['\|', ('user_id', '=', user.id), ('create_uid', '=', user.id)]` | False | True | False | True |

## Views (11)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.mail_activity_view_form_popup` | xpath | `mail.mail_activity_view_form_popup` |  |  |  | `calendar` |
| `mail.mail_activity_view_form_popup` | form |  | `activity_category`, `res_model`, `res_model_id`, `res_id`, `chaining_type`, `previous_activity_type_id`, `has_recommended_activities`, `recommended_activity_type_id`, `activity_type_id`, `summary`, `date_deadline`, `user_id`, `note`, `id` | `action_open_document`, `Schedule`, `Save`, `Mark Done`, `Discard` |  | `mail` |
| `mail.mail_activity_view_form` | field | `mail.mail_activity_view_form_popup` | `activity_type_id`, `res_name` |  |  | `mail` |
| `mail.mail_activity_view_form_without_record_access` | form |  | `display_name`, `activity_type_id`, `summary`, `date_deadline`, `note` | `Mark Done` |  | `mail` |
| `mail.mail_activity_view_search` | search |  | `res_name`, `user_id`, `activity_type_id` |  | `My Activities`, `Unassigned Activities`, `Overdue`, `Today`, `Tomorrow`, `This week`, `Future`, `Done`, `Deadline`, `Document Model`, `Assigned To`, `Created By`, `Activity Type` | `mail` |
| `mail.mail_activity_view_tree` | list |  | `summary`, `activity_type_id`, `user_id`, `res_name`, `date_deadline`, `date_done`, `feedback` | `Done`, `Cancel`, `Today`, `Tomorrow`, `Next Week`, `Done`, `Cancel` |  | `mail` |
| `mail.mail_activity_view_tree_without_record_access` | xpath | `mail_activity_view_tree` |  |  |  | `mail` |
| `mail.mail_activity_view_tree_open_target` | xpath | `mail_activity_view_tree` |  |  |  | `mail` |
| `mail.mail_activity_view_kanban_open_target` | kanban |  | `active`, `icon`, `res_name`, `res_model_id`, `summary`, `activity_type_id`, `summary`, `user_id`, `date_deadline` | `action_done`, `unlink` |  | `mail` |
| `mail.mail_activity_view_calendar` | calendar |  | `user_id`, `res_name`, `date_deadline`, `summary`, `activity_type_id` |  |  | `mail` |
| `website_slides.mail_activity_view_form` | field | `mail.mail_activity_view_form` | `summary`, `request_partner_id` |  |  | `website_slides` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `mail.mail_activity_action` | Activity Overview | list,form |  | `{             'force_search_count': 1,         }` |  | `mail` |
| `mail.mail_activity_without_access_action` | Other activities | list,form | `['\|', ('id', 'in', context.get('active_ids')), '&', ('res_model', '=', False), ('user_id', '=', uid)]` |  | main | `mail` |
| `mail.mail_activity_action_my` | My Activities | list,kanban,calendar |  | `{             'force_search_count': 1,             'search_default_filter_user_id_uid': 1,             'search_default_filter_date_deadline_past': 1,             'search_default_filter_date_deadline_today': 1,         }` |  | `mail` |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.json`; views: `../../../schemas/interfaces/views/mail.activity.json`.
