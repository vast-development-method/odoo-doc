# Activity Mixin (`mail.activity.mixin`)

**Transport name:** `mail.activity.mixin`  
**Storage name:** `mail_activity_mixin`  
**Kind:** abstract entity (a mixin of fields and operations reused by other entities, no table of its own)  
**Defined by package:** `mail`  
**Extended by packages:** `calendar`

Description: Activity Mixin

## Fields (11)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `activity_ids` | Activities | one to many | `mail.activity` | visible only to groups `base.group_user`; inverse field `res_id` |
| `activity_state` | Activity State | selection |  | computed by rule `_compute_activity_state` (not stored); searchable through a search rule; visible only to groups `base.group_user`; Help: Status based on activities Overdue: Due date is already passed Today: Activity date is today Planned: Future activities. |
| `activity_user_id` | Responsible User | many to one | `res.users` | read only; computed by rule `_compute_activity_user_id` (not stored); searchable through a search rule; visible only to groups `base.group_user` |
| `activity_type_id` | Next Activity Type | many to one | `mail.activity.type` | related through path `activity_ids.activity_type_id`; searchable through a search rule; visible only to groups `base.group_user` |
| `activity_type_icon` | Activity Type Icon | single line text |  | related through path `activity_ids.icon` |
| `activity_date_deadline` | Next Activity Deadline | date |  | read only; computed by rule `_compute_activity_date_deadline` (not stored); searchable through a search rule; visible only to groups `base.group_user` |
| `my_activity_date_deadline` | My Activity Deadline | date |  | read only; computed by rule `_compute_my_activity_date_deadline` (not stored); searchable through a search rule; visible only to groups `base.group_user` |
| `activity_summary` | Next Activity Summary | single line text |  | related through path `activity_ids.summary`; searchable through a search rule; visible only to groups `base.group_user` |
| `activity_exception_decoration` | Activity Exception Decoration | selection |  | computed by rule `_compute_activity_exception_type` (not stored); searchable through a search rule; Help: Type of the exception activity on record. |
| `activity_exception_icon` | Icon | single line text |  | computed by rule `_compute_activity_exception_type` (not stored); Help: Icon to indicate an exception activity. |
| `activity_calendar_event_id` | Next Activity Calendar Event | many to one | `calendar.event` | computed by rule `_compute_activity_calendar_event_id` (not stored); visible only to groups `base.group_user` |

## Selection values

### `activity_state` (Activity State)

| Value | Label |
|---|---|
| `overdue` | Overdue |
| `today` | Today |
| `planned` | Planned |

### `activity_exception_decoration` (Activity Exception Decoration)

| Value | Label |
|---|---|
| `warning` | Alert |
| `danger` | Error |

## State fields

State machine fields of this entity: `activity_state`. Transitions are specified in the domain documents.

## Operations (25)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_default_activity_type` | preparation rule | self | `mail` |  | Define a default fallback activity type when requested xml id wasn't found.  Can be overriden to specify the default activity type of a model. It is only called in in activity_schedule() for now. |
| `_compute_activity_exception_type` | computation | self | `mail` | depends: `activity_ids.activity_type_id.decoration_type`, `activity_ids.activity_type_id.icon` |  |
| `_compute_activity_user_id` | computation | self | `mail` | depends: `activity_ids.user_id` |  |
| `_search_activity_exception_decoration` | search rule | self, operator, operand | `mail` |  |  |
| `_compute_activity_state` | computation | self | `mail` | depends: `activity_ids.state` |  |
| `_search_activity_state` | search rule | self, operator, value | `mail` |  |  |
| `_compute_activity_date_deadline` | computation | self | `mail` | depends: `activity_ids.date_deadline` |  |
| `_search_activity_date_deadline` | search rule | self, operator, operand | `mail` |  |  |
| `_search_activity_user_id` | search rule | self, operator, operand | `mail` | model |  |
| `_search_activity_type_id` | search rule | self, operator, operand | `mail` | model |  |
| `_search_activity_summary` | search rule | self, operator, operand | `mail` | model |  |
| `_compute_my_activity_date_deadline` | computation | self | `mail` | depends: `activity_ids.date_deadline`, `activity_ids.user_id`; depends_context: `uid` |  |
| `_search_my_activity_date_deadline` | search rule | self, operator, operand | `mail` |  |  |
| `_read_group_groupby` | internal rule | self, alias, groupby_spec, query | `mail` |  |  |
| `action_reschedule_my_next_today` | user action | self | `mail` |  |  |
| `action_reschedule_my_next_tomorrow` | user action | self | `mail` |  |  |
| `action_reschedule_my_next_nextweek` | user action | self | `mail` |  |  |
| `activity_send_mail` | operation | self, template_id | `mail` |  | Automatically send an email based on the given mail.template, given its ID. |
| `activity_search` | operation | self, act_type_xmlids, user_id, additional_domain, only_automated | `mail` |  | Search automated activities on current record set, given a list of activity types xml IDs. It is useful when dealing with specific types involved in automatic activities management.  :param act_type_xmlids: list of activity types xml IDs :param user_id: if set, restrict to activities of that user_id; :param additional_domain: if set, filter on that domain; :param only_automated: if unset, search for all activities, not only automated ones; |
| `activity_schedule` | operation | self, act_type_xmlid, date_deadline, summary, note, **act_values | `mail` |  | Schedule an activity on each record of the current record set. This method allow to provide as parameter act_type_xmlid. This is an xml_id of activity type instead of directly giving an activity_type_id. It is useful to avoid having various "env.ref" in the code and allow to let the mixin handle access rights.  Note that unless specified otherwise in act_values, the activities created will have their "automated" field set to True.  :param date_deadline: the day the activity must be scheduled on the timezone of the user must be considered to set the correct deadline |
| `_activity_schedule_with_view` | internal rule | self, act_type_xmlid, date_deadline, summary, views_or_xmlid, render_context, **act_values | `mail` |  | Helper method: Schedule an activity on each record of the current record set. This method allow to the same mecanism as `activity_schedule`, but provide 2 additionnal parameters: :param views_or_xmlid: record of ir.ui.view or string representing the xmlid     of the qweb template to render :type views_or_xmlid: string or recordset :param render_context: the values required to render the given qweb template :type render_context: dict |
| `activity_reschedule` | operation | self, act_type_xmlids, user_id, date_deadline, new_user_id, only_automated | `mail` |  | Reschedule some automated activities. Activities to reschedule are selected based on type xml ids and optionally by user. Purpose is to be able to   * update the deadline to date_deadline;  * update the responsible to new_user_id; |
| `activity_feedback` | operation | self, act_type_xmlids, user_id, feedback, attachment_ids, only_automated | `mail` |  | Set activities as done, limiting to some activity types and optionally to a given user. |
| `activity_unlink` | operation | self, act_type_xmlids, user_id, only_automated | `mail` |  | Unlink activities, limiting to some activity types and optionally to a given user. |
| `_compute_activity_calendar_event_id` | computation | self | `calendar` | depends: `activity_ids.calendar_event_id` | This computes the calendar event of the next activity. It evaluates to false if there is no such event. |

Machine-readable definition: `../../../schemas/data/entities/mail.activity.mixin.json`.
