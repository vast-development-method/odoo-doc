# Calendar Event (`calendar.event`)

**Transport name:** `calendar.event`  
**Storage name:** `calendar_event`  
**Kind:** persistent entity (one table)  
**Defined by package:** `calendar`  
**Extended by packages:** `calendar_sms`, `crm`, `google_calendar`, `hr_calendar`, `hr_holidays`, `hr_recruitment`, `microsoft_calendar`

Description: Calendar Event

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `google.calendar.sync`, `microsoft.calendar.sync`
- Default ordering: `start desc`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (72)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Meeting Subject | single line text |  | required |
| `description` | Description | rich text |  | Help: When synchronization with an external calendar is active, this description is synchronized         with the one of the associated meeting in that external calendar. Any update will be propagated there         and vice versa. |
| `user_id` | Organizer | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); indexed (btree_not_null) |
| `partner_id` | Scheduled by | many to one | `res.partner` | read only; related through path `user_id.partner_id` |
| `location` | Location | single line text |  | changes are tracked in the message thread |
| `notes` | Notes | rich text |  |  |
| `videocall_location` | Meeting uniform resource locator | single line text |  | computed by rule `_compute_videocall_location` and stored |
| `access_token` | Invitation Token | single line text |  | indexed; not copied on duplication |
| `videocall_source` | Videocall Source | selection |  | computed by rule `_compute_videocall_source` (not stored); on delete of the target: {"google_meet": "set discuss"}; extended by packages `google_calendar` |
| `videocall_channel_id` | Discuss Channel | many to one | `discuss.channel` | indexed (btree_not_null) |
| `privacy` | Privacy | selection |  | Help: People to whom this event will be visible. |
| `effective_privacy` | Effective Privacy | selection |  | computed by rule `_compute_effective_privacy` (not stored); Help: Whether the event is private, considering the user privacy |
| `show_as` | Show as | selection |  | required; default `busy`; Help: If the time is shown as 'busy', this event will be visible to other people with either the full         information or simply 'busy' written depending on its privacy. Use this option to let other people know         that you are unavailable during that period of time.   If the event is shown as 'free', other users know         that you are available during that period of time. |
| `is_highlighted` | Is the Event Highlighted | boolean |  | computed by rule `_compute_is_highlighted` (not stored) |
| `is_organizer_alone` | Is the Organizer Alone | boolean |  | computed by rule `_compute_is_organizer_alone` (not stored); Help: Check if the organizer is alone in the event, i.e. if the organizer is the only one that hasn't declined         the event (only if the organizer is not the only attendee) |
| `active` | Active | boolean |  | default `True`; changes are tracked in the message thread; Help: If the active field is set to false, it will allow you to hide the event alarm information without removing it. |
| `categ_ids` | Tags | many to many | `calendar.event.type` | association table `meeting_category_rel` |
| `start` | Start | date and time |  | required; default computed dynamically (_default_start); changes are tracked in the message thread; indexed; Help: Start date of an event, without time for full days events |
| `stop` | Stop | date and time |  | required; computed by rule `_compute_stop` and stored; default computed dynamically (_default_stop); changes are tracked in the message thread; Help: Stop date of an event, without time for full days events |
| `display_time` | Event Time | single line text |  | computed by rule `_compute_display_time` (not stored) |
| `allday` | All Day | boolean |  | default  |
| `start_date` | Start Date | date |  | computed by rule `_compute_dates` and stored; writable through an inverse rule; changes are tracked in the message thread |
| `stop_date` | End Date | date |  | computed by rule `_compute_dates` and stored; writable through an inverse rule; changes are tracked in the message thread |
| `duration` | Duration | float |  | computed by rule `_compute_duration` and stored |
| `res_id` | Document identifier | many to one by reference |  |  |
| `res_model_id` | Document Model | many to one | `ir.model` | on delete of the target: cascade |
| `res_model` | Document Model Name | single line text |  | read only; related through path `res_model_id.model` and stored |
| `res_model_name` | Resource Model Name | single line text |  | related through path `res_model_id.name` |
| `activity_ids` | Activities | one to many | `mail.activity` | inverse field `calendar_event_id` |
| `attendee_ids` | Participant | one to many | `calendar.attendee` | inverse field `event_id` |
| `current_attendee` | Current Attendee | many to one | `calendar.attendee` | computed by rule `_compute_current_attendee` (not stored); searchable through a search rule |
| `current_status` | Attending? | selection |  | related through path `current_attendee.state` |
| `should_show_status` | Should Show Status | boolean |  | computed by rule `_compute_should_show_status` (not stored) |
| `partner_ids` | Attendees | many to many | `res.partner` | default computed dynamically (_default_partners); association table `calendar_event_res_partner_rel` |
| `invalid_email_partner_ids` | Invalid Email Partner | many to many | `res.partner` | computed by rule `_compute_invalid_email_partner_ids` (not stored) |
| `unavailable_partner_ids` | Unavailable Attendees | many to many | `res.partner` | computed by rule `_compute_unavailable_partner_ids` (not stored) |
| `alarm_ids` | Reminders | many to many | `calendar.alarm` | on delete of the target: restrict; association table `calendar_alarm_calendar_event_rel`; Help: Notifications sent to all attendees to remind of the meeting. |
| `recurrency` | Recurrent | boolean |  |  |
| `recurrence_id` | Recurrence Rule | many to one | `calendar.recurrence` | indexed (btree_not_null) |
| `follow_recurrence` | Follow Recurrence | boolean |  | default  |
| `recurrence_update` | Recurrence Update | selection |  | default `self_only`; not copied on duplication; Help: Choose what to do with other events in the recurrence. Updating All Events is not allowed when dates or time is modified |
| `rrule` | Recurrent Rule | single line text |  | computed by rule `_compute_recurrence` (not stored) |
| `rrule_type_ui` | Repeat | selection |  | computed by rule `_compute_rrule_type_ui` (not stored); Help: Let the event automatically repeat at that interval |
| `rrule_type` | Recurrence | selection |  | computed by rule `_compute_recurrence` (not stored); Help: Let the event automatically repeat at that interval |
| `event_tz` | Timezone | selection |  | computed by rule `_compute_recurrence` (not stored) |
| `end_type` | Recurrence Termination | selection |  | computed by rule `_compute_recurrence` (not stored) |
| `interval` | Repeat On | integer |  | computed by rule `_compute_recurrence` (not stored); Help: Repeat every (Days/Week/Month/Year) |
| `count` | Number of Repetitions | integer |  | computed by rule `_compute_recurrence` (not stored); Help: Repeat x times |
| `mon` | Mon | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `tue` | Tue | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `wed` | Wed | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `thu` | Thu | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `fri` | Fri | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `sat` | Sat | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `sun` | Sun | boolean |  | computed by rule `_compute_recurrence` (not stored) |
| `month_by` | Option | selection |  | computed by rule `_compute_recurrence` (not stored) |
| `day` | Date of month | integer |  | computed by rule `_compute_recurrence` (not stored) |
| `weekday` | Weekday | selection |  | computed by rule `_compute_recurrence` (not stored) |
| `byday` | By day | selection |  | computed by rule `_compute_recurrence` (not stored) |
| `until` | Until | date |  | computed by rule `_compute_recurrence` (not stored) |
| `display_description` | Display Description | boolean |  | computed by rule `_compute_display_description` (not stored) |
| `attendees_count` | Attendees Count | integer |  | computed by rule `_compute_attendees_count` (not stored) |
| `accepted_count` | Accepted Count | integer |  | computed by rule `_compute_attendees_count` (not stored) |
| `declined_count` | Declined Count | integer |  | computed by rule `_compute_attendees_count` (not stored) |
| `tentative_count` | Tentative Count | integer |  | computed by rule `_compute_attendees_count` (not stored) |
| `awaiting_count` | Awaiting Count | integer |  | computed by rule `_compute_attendees_count` (not stored) |
| `user_can_edit` | User Can Edit | boolean |  | computed by rule `_compute_user_can_edit` (not stored) |
| `opportunity_id` | Opportunity | many to one | `crm.lead` | indexed; on delete of the target: set null; restricted by domain `[('type', '=', 'opportunity')]` |
| `google_id` | Google Calendar Event Id | single line text |  | computed by rule `_compute_google_id` and stored |
| `guests_readonly` | Guests Event Modification Permission | boolean |  | default  |
| `applicant_id` | Applicant | many to one | `hr.applicant` | indexed (btree_not_null); on delete of the target: set null |
| `microsoft_recurrence_master_id` | Microsoft Recurrence Master Id | single line text |  |  |

## Selection values

### `videocall_source` (Videocall Source)

| Value | Label |
|---|---|
| `discuss` | Discuss |
| `custom` | Custom |
| `google_meet` | Google Meet |

### `privacy` (Privacy)

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

### `effective_privacy` (Effective Privacy)

| Value | Label |
|---|---|
| `public` | Public |
| `private` | Private |
| `confidential` | Only internal users |

### `show_as` (Show as)

| Value | Label |
|---|---|
| `free` | Available |
| `busy` | Busy |

### `recurrence_update` (Recurrence Update)

| Value | Label |
|---|---|
| `self_only` | This event |
| `future_events` | This and following events |
| `all_events` | All events |

## State fields

State machine fields of this entity: `current_status`. Transitions are specified in the domain documents.

## Operations (143)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `get_state_selections` | operation | self | `calendar` | model |  |
| `default_get` | lifecycle override | self, fields | `calendar`, `crm`, `hr_recruitment` | model |  |
| `_default_partners` | preparation rule | self | `calendar` | model | When active_model is res.partner, the current partners should be attendees |
| `_default_start` | preparation rule | self | `calendar` | model |  |
| `_default_stop` | preparation rule | self | `calendar` | model |  |
| `_compute_should_show_status` | computation | self | `calendar` | depends: `attendee_ids` |  |
| `_compute_current_attendee` | computation | self | `calendar` | depends: `attendee_ids`, `attendee_ids.state` |  |
| `_search_current_attendee` | search rule | self, operator, value | `calendar` |  |  |
| `_compute_attendees_count` | computation | self | `calendar` | depends: `attendee_ids`, `attendee_ids.state`, `partner_ids` |  |
| `_compute_user_can_edit` | computation | self | `calendar` | depends: `partner_ids`; depends_context: `uid` |  |
| `_compute_invalid_email_partner_ids` | computation | self | `calendar` | depends: `partner_ids` |  |
| `_compute_effective_privacy` | computation | self | `calendar` | depends: `privacy`, `user_id` |  |
| `_compute_is_highlighted` | computation | self | `calendar`, `crm`, `hr_recruitment` |  |  |
| `_compute_is_organizer_alone` | computation | self | `calendar` | depends: `partner_id`, `attendee_ids` | Check if the organizer of the event is the only one who has accepted the event. It does not apply if the organizer is the only attendee of the event because it would represent a personnal event. The goal of this field is to highlight to the user that the others attendees are not available for this event. |
| `_compute_display_time` | computation | self | `calendar` |  |  |
| `_compute_dates` | computation | self | `calendar` | depends: `allday`, `start`, `stop` | Adapt the value of start_date(time)/stop_date(time) according to start/stop fields and allday. Also, compute the duration for not allday meeting ; otherwise the duration is set to zero, since the meeting last all the day. |
| `_compute_duration` | computation | self | `calendar` | depends: `stop`, `start` |  |
| `_compute_stop` | computation | self | `calendar` | depends: `start`, `duration` |  |
| `_onchange_date` | on change | self | `calendar` | onchange: `start_date`, `stop_date` | This onchange is required for cases where the stop/start is False and we set an allday event. The inverse method is not called in this case because start_date/stop_date are not used in any compute/related, so we need an onchange to set the start/stop values in the form view |
| `_inverse_dates` | inverse computation | self | `calendar` |  | This method is used to set the start and stop values of all day events. The calendar view needs date_start and date_stop values to display correctly the allday events across several days. As the user edit the {start,stop}_date fields when allday is true, this inverse method is needed to update the  start/stop value and have a relevant calendar view. |
| `_check_closing_date` | validation | self | `calendar` | constrains: `start`, `stop`, `start_date`, `stop_date` |  |
| `_check_organizer_validation_conditions` | validation | self, vals_list | `calendar` |  | Method for check in the microsoft_calendar module that needs to be overridden in appointment. |
| `_compute_rrule_type_ui` | computation | self | `calendar` | depends: `recurrence_id`, `recurrency` |  |
| `_compute_recurrence` | computation | self | `calendar` | depends: `recurrence_id`, `recurrency`, `rrule_type_ui` |  |
| `_compute_display_description` | computation | self | `calendar` | depends: `description` |  |
| `_compute_unavailable_partner_ids` | computation | self | `calendar`, `hr_calendar` | depends: `partner_ids`, `start`, `stop`; depends: `allday` |  |
| `_compute_videocall_location` | computation | self | `calendar` | depends: `videocall_source`, `access_token` |  |
| `_is_partner_unavailable` | internal rule | self, partner, partner_events | `calendar` |  |  |
| `_set_videocall_location` | internal rule | self, vals_list | `calendar` | model |  |
| `_compute_videocall_source` | computation | self | `calendar`, `google_calendar` | depends: `videocall_location` |  |
| `_set_discuss_videocall_location` | internal rule | self | `calendar` |  | This method sets the videocall_location to a discuss route. If no access_token exists for this event, we create one. Note that recurring events will have different access_tokens. This is done by design to prevent users not being able to join a discuss meeting because the base event of the recurrency was deleted. |
| `get_discuss_videocall_location` | operation | self | `calendar` | model |  |
| `create` | lifecycle override | self, vals_list | `calendar`, `crm`, `google_calendar`, `hr_recruitment`, `microsoft_calendar` | model_create_multi |  |
| `_compute_field_value` | computation | self, field | `calendar` |  |  |
| `_fetch_query` | internal rule | self, query, fields | `calendar` |  |  |
| `write` | lifecycle override | self, vals | `calendar`, `google_calendar`, `microsoft_calendar` |  |  |
| `_check_calendar_privacy_write_permissions` | validation | self | `calendar` |  | Checks if current user can write on the events, raising UserError when the event is private. We need to manually call the default Access Error because we can't add an access rule for checking the calendar defaut privacy of an user from a 'calendar.event' record, since it is a res.users field. Otherwise we would have to create a new computed field on that model, which we don't want. |
| `_check_private_event_conditions` | validation | self | `calendar` |  | Checks if the event is private, returning True if the conditions match and False otherwise. |
| `_compute_display_name` | computation | self | `calendar` | depends: `privacy`, `user_id` | Hide private events' name for events which don't belong to the current user. |
| `_read_group` | lifecycle override | self, domain, groupby, aggregates, having, offset, limit, order | `calendar` | model |  |
| `_read_grouping_sets` | internal rule | self, domain, grouping_sets, aggregates, order | `calendar` | model |  |
| `unlink` | lifecycle override | self | `calendar`, `microsoft_calendar` |  |  |
| `copy` | lifecycle override | self, default | `calendar` |  | When an event is copied, the attendees should be recreated to avoid sharing the same attendee records between copies |
| `action_unlink_event` | user action | self, attendee_id, recurrence | `calendar` |  | Delete the event after displaying the delete wizard if necessary.  :param attendee_id: The ID of the attendee for the event :param recurrence: Boolean indicating if the event is recurring :return: Action to delete the event |
| `_mail_get_operation_for_mail_message_operation` | messaging hook | self, message_operation | `calendar` |  |  |
| `_attendees_values` | internal rule | self, partner_commands | `calendar` |  | :param partner_commands: ORM commands for partner_id field (0 and 1 commands not supported) :return: associated attendee_ids ORM commands |
| `_create_videocall_channel` | internal rule | self | `calendar` |  |  |
| `_create_videocall_channel_id` | internal rule | self, name, partner_ids | `calendar` |  |  |
| `_get_default_privacy_domain` | preparation rule | self | `calendar` |  |  |
| `_is_event_over` | internal rule | self | `calendar` |  | Check if the event is over. This method is used to check if the event should trigger invitations with Google Calendar. :return: True if the event is over, False otherwise |
| `set_discuss_videocall_location` | operation | self | `calendar` |  |  |
| `clear_videocall_location` | operation | self | `calendar` |  |  |
| `action_open_calendar_event` | user action | self | `calendar` |  |  |
| `action_sendmail` | user action | self | `calendar` |  |  |
| `action_open_composer` | user action | self | `calendar` |  |  |
| `action_join_video_call` | user action | self | `calendar` |  |  |
| `action_join_meeting` | user action | self, partner_id | `calendar` |  | Method used when an existing user wants to join |
| `action_mass_deletion` | user action | self, recurrence_update_setting | `calendar` |  |  |
| `action_mass_archive` | user action | self, recurrence_update_setting | `calendar`, `google_calendar`, `microsoft_calendar` |  | The aim of this action purpose is to be called from sync calendar module when mass deletion is not possible. |
| `_skip_send_mail_status_update` | internal rule | self | `calendar`, `google_calendar`, `microsoft_calendar` |  | Overridable getter to identify whether to send invitation/cancelation emails. |
| `_get_mail_tz` | preparation rule | self | `calendar` |  |  |
| `_sync_activities` | internal rule | self, fields | `calendar` |  |  |
| `_get_activity_deadline_from_start` | preparation rule | self, start, allday | `calendar` | model |  |
| `_get_trigger_alarm_types` | preparation rule | self | `calendar_sms`, `calendar` |  |  |
| `_setup_event_recurrent_alarms` | internal rule | self, events_by_alarm | `calendar` |  | Setup alarms for recurrent events |
| `_setup_alarms` | internal rule | self | `calendar` |  | Schedule cron triggers for future events |
| `get_next_alarm_date` | operation | self, events_by_alarm | `calendar` |  |  |
| `_apply_recurrence_values` | internal rule | self, values, future | `calendar` |  | Apply the new recurrence rules in `values`. Create a recurrence if it does not exist and create all missing events according to the rrule. If the changes are applied to future events only, a new recurrence is created with the updated rrule.  :param values: new recurrence values to apply :param future: rrule values are applied to future events only if True.                Rrule changes are applied to all events in the recurrence otherwise.                (ignored if no recurrence exists yet). :return: events detached from the recurrence |
| `_get_recurrence_params` | preparation rule | self | `calendar` |  |  |
| `_get_recurrence_params_by_date` | preparation rule | self, event_date | `calendar` | model | Return the recurrence parameters from a date object. |
| `_break_recurrence` | internal rule | self, future | `calendar` |  | Breaks the event's recurrence. Stop the recurrence at the current event if `future` is True, leaving past events in the recurrence. If `future` is False, all events in the recurrence are detached and the recurrence itself is unlinked. :return: detached events excluding the current events |
| `_get_time_update_dict` | preparation rule | self, base_event, time_values | `calendar` |  | Return the update dictionary for shifting the base_event's time to the new date. |
| `_get_archive_values` | preparation rule | self | `calendar`, `google_calendar` | model | Return parameters for archiving events in calendar module. |
| `_check_values_to_sync` | validation | self, values | `calendar`, `google_calendar` | model | Method to be overriden: return candidate values to be synced within rewrite_recurrence function scope. |
| `_get_update_future_events_values` | preparation rule | self | `calendar`, `google_calendar` | model | Return parameters for updating future events within _update_future_events function scope. |
| `_get_remove_sync_id_values` | preparation rule | self | `calendar`, `google_calendar` | model | Return parameters for removing event synchronization id within _update_future_events function scope. |
| `_get_updated_recurrence_values` | preparation rule | self, new_start_date | `calendar` |  | Copy values from current recurrence and update the start date weekday. |
| `_update_future_events` | internal rule | self, values, time_values, recurrence_values | `calendar` |  | Trim the current recurrence detaching the occurrences after current event, deactivate the detached events except for the updated event and apply recurrence values. |
| `_rewrite_recurrence` | internal rule | self, values, time_values, recurrence_values | `calendar` |  | Delete the current recurrence, reactivate base event and apply updated recurrence values. |
| `change_attendee_status` | operation | self, status, recurrence_update_setting | `calendar` |  |  |
| `find_partner_customer` | operation | self | `calendar` |  |  |
| `_get_activity_excluded_models` | preparation rule | self | `calendar` | model | For some models, we don't want to automatically create activities when a calendar.event is created. (This is the case notably for appointment.types) This hook method allows to specify those models. See calendar.event create method for details. |
| `_reset_attendees_status` | internal rule | self | `calendar` |  | Reset attendees status to pending and accept event for current user. |
| `_get_start_date` | preparation rule | self | `calendar` |  | Return the event starting date in the event's timezone. If no starting time is assigned (yet), return today as default :return: date |
| `_range` | internal rule | self | `calendar` |  |  |
| `get_display_time_tz` | operation | self, tz | `calendar` |  | get the display_time of the meeting, forcing the timezone. This method is called from email template, to not use sudo(). |
| `_get_ics_file` | preparation rule | self | `calendar` |  | Returns iCalendar file for the event invitation. :returns a dict of .ics file content for each meeting |
| `_get_contact_details_description` | preparation rule | self, organizer, partners | `calendar` | model | Build sanitized HTML with the organizer details and the details of the contact partner (only when there is a single non-organizer attendee). |
| `_get_new_invited_attendees` | preparation rule | self, current_attendees, previous_attendees, update_vals | `calendar` | model | Get the attendees who must receive an invitation for a modified calendar event. This method is meant to be overridden. |
| `_prepare_partner_contact_details_html` | preparation rule | self, section_title, partner | `calendar` | model |  |
| `_get_customer_description` | preparation rule | self | `calendar` |  | :rtype: str :returns: html Sanitized HTML description for customer to include in calendar exports |
| `_get_customer_summary` | preparation rule | self | `calendar` |  | :rtype: str :returns: The summary to include in calendar exports |
| `_get_display_time` | preparation rule | self, start, stop, zduration, zallday | `calendar` | model | Return date and time (from to from) based on duration with timezone in string. Eg : 1) if user add duration for 2 hours, return : August-23-2013 at (04-30 To 06-30) (Europe/Brussels) 2) if event all day ,return : AllDay, July-31-2013 |
| `_get_duration` | preparation rule | self, start, stop | `calendar` |  | Get the duration value between the 2 given dates. |
| `_get_date_formats` | preparation rule | self | `calendar` | model | get current date and time format, according to the context lang :return: a tuple with (format date, format time) |
| `_get_recurrent_fields` | preparation rule | self | `calendar` | model |  |
| `_get_time_fields` | preparation rule | self | `calendar` | model |  |
| `_get_custom_fields` | preparation rule | self | `calendar` | model |  |
| `_get_public_fields` | preparation rule | self | `calendar` | model |  |
| `get_default_duration` | operation | self | `calendar` | model |  |
| `_do_sms_reminder` | internal rule | self, alarms | `calendar_sms` |  | Send an SMS text reminder to attendees that haven't declined the event |
| `action_send_sms` | user action | self | `calendar_sms` |  |  |
| `_is_crm_lead` | internal rule | self, defaults, ctx | `crm` |  | This method checks if the concerned model is a CRM lead. The information is not always in the defaults values, this is why it is necessary to check the context too. |
| `_compute_google_id` | computation | self | `google_calendar` | depends: `recurrence_id.google_id` |  |
| `_get_google_synced_fields` | preparation rule | self | `google_calendar` | model |  |
| `_restart_google_sync` | internal rule | self | `google_calendar` | model |  |
| `_check_modify_event_permission` | validation | self, values | `google_calendar` |  | Check if event modification attempt by attendee is valid to avoid duplicate events creation. |
| `_get_sync_domain` | preparation rule | self | `google_calendar` |  |  |
| `_odoo_values` | internal rule | self, google_event, default_reminders | `google_calendar` | model |  |
| `_odoo_attendee_commands` | internal rule | self, google_event | `google_calendar` | model |  |
| `_odoo_reminders_commands` | internal rule | self, reminders | `google_calendar` | model |  |
| `_google_values` | internal rule | self | `google_calendar` |  |  |
| `_cancel` | internal rule | self | `google_calendar` |  |  |
| `_get_event_user` | preparation rule | self | `google_calendar` |  |  |
| `_is_google_insertion_blocked` | internal rule | self, sender_user | `google_calendar` |  |  |
| `get_unusual_days` | operation | self, date_from, date_to | `hr_calendar` | model |  |
| `_get_events_interval` | preparation rule | self | `hr_calendar` |  | This method will returned an Intervals object that represent the event's interval based of its parameters.  If an event is scheduled for the entire day, its interval will correspond to the work interval defined by the company's calendar. If an allday event is scheduled on a day when the company is closed, the interval of this event will be empty. |
| `_check_employees_availability_for_event` | validation | self, schedule_by_partner, event_interval | `hr_calendar` |  |  |
| `_need_video_call` | internal rule | self | `hr_holidays` |  | Determine if the event needs a video call or not depending on the model of the event.  This method, implemented and invoked in google_calendar, is necessary due to the absence of a bridge module between google_calendar and hr_holidays. |
| `_get_organizer` | preparation rule | self | `microsoft_calendar` |  |  |
| `_get_microsoft_synced_fields` | preparation rule | self | `microsoft_calendar` | model |  |
| `_restart_microsoft_sync` | internal rule | self | `microsoft_calendar` | model |  |
| `_check_microsoft_sync_status` | validation | self | `microsoft_calendar` |  | Returns True if synchronization with Outlook Calendar is active and False otherwise. The 'microsoft_synchronization_stopped' variable needs to be 'False' and Outlook account must be connected. |
| `_check_organizer_validation` | validation | self, sender_user, partner_included | `microsoft_calendar` |  | Check if the proposed event organizer can be set accordingly. |
| `_check_recurrence_overlapping` | validation | self, new_start | `microsoft_calendar` |  | Outlook does not allow to modify time fields of an event if this event crosses or overlaps the recurrence. In this case a 400 error with the Outlook code "ErrorOccurrenceCrossingBoundary" is returned. That means that the update violates the following Outlook restriction on recurrence exceptions: an occurrence cannot be moved to or before the day of the previous occurrence, and cannot be moved to or after the day of the following occurrence. For example: E1 E2 E3 E4 cannot becomes E1 E3 E2 E4 |
| `_is_matching_timeslot` | internal rule | self, start, stop, allday | `microsoft_calendar` |  | Check if an event matches with the provided timeslot |
| `_forbid_recurrence_update` | internal rule | self | `microsoft_calendar` |  | Suggest user to update recurrences in Outlook due to the Outlook Calendar spam limitation. |
| `_forbid_recurrence_creation` | internal rule | self | `microsoft_calendar` |  | Suggest user to update recurrences in Outlook due to the Outlook Calendar spam limitation. |
| `_recreate_event_different_organizer` | internal rule | self, values, sender_user | `microsoft_calendar` |  | Copy current event values, delete it and recreate it with the new organizer user. |
| `_get_organizer_user_change_info` | preparation rule | self, values | `microsoft_calendar` | model | Return the sender user of the event and the partner ids listed on the event values. |
| `_update_attendee_status` | internal rule | self, attendee_ids | `microsoft_calendar` |  | Merge current status from 'attendees_ids' with new attendees values for avoiding their info loss in write(). Create a dict getting the state of each attendee received from 'attendee_ids' variable and then update their state. :param attendee_ids: List of attendee commands carrying a dict with 'partner_id' and 'state' keys in its third position. |
| `_get_microsoft_sync_domain` | preparation rule | self | `microsoft_calendar` |  |  |
| `_microsoft_to_odoo_values` | internal rule | self, microsoft_event, default_reminders, default_values, with_ids | `microsoft_calendar` | model |  |
| `_microsoft_to_odoo_recurrence_values` | internal rule | self, microsoft_event, default_values | `microsoft_calendar` | model |  |
| `_odoo_attendee_commands_m` | internal rule | self, microsoft_event | `microsoft_calendar` | model |  |
| `_odoo_reminders_commands_m` | internal rule | self, microsoft_event | `microsoft_calendar` | model |  |
| `_get_attendee_status_o2m` | preparation rule | self, attendee | `microsoft_calendar` |  |  |
| `_microsoft_values` | internal rule | self, fields_to_sync, initial_values | `microsoft_calendar` |  |  |
| `_ensure_attendees_have_email` | internal rule | self | `microsoft_calendar` |  |  |
| `_microsoft_values_occurence` | internal rule | self, initial_values | `microsoft_calendar` |  |  |
| `_cancel_microsoft` | internal rule | self | `microsoft_calendar` |  | Cancel an Microsoft event. There are 2 cases:   1) the organizer is an Odoo user: he's the only one able to delete the Odoo event. Attendees can just decline.   2) the organizer is NOT an Odoo user: any attendee should remove the Odoo event. |
| `_get_event_user_m` | preparation rule | self, user_id | `microsoft_calendar` |  | Get the user who will send the request to Microsoft (organizer if synchronized and current user otherwise). |
| `_is_microsoft_insertion_blocked` | internal rule | self, sender_user | `microsoft_calendar` |  |  |

## Validation and error messages (14)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_closing_date` | ValidationError | The ending date and time cannot be earlier than the starting date and time. Meeting “%(name)s” starts at %(start_time)s and ends at %(end_time)s | `calendar` |
| `_check_closing_date` | ValidationError | The ending date cannot be earlier than the starting date. Meeting “%(name)s” starts on %(start_date)s and ends on %(end_date)s | `calendar` |
| `write` | UserError | Unable to save the recurrence with "This Event" | `calendar` |
| `action_open_composer` | UserError | There are no attendees on these events | `calendar` |
| `_get_time_update_dict` | UserError | You can't update a recurrence without base event. | `calendar` |
| `_get_ics_file` | UserError | First you have to specify the date of the invitation. | `calendar` |
| `action_send_sms` | UserError | There are no attendees on these events | `calendar_sms` |
| `_check_modify_event_permission` | ValidationError | The following event can only be updated by the organizer according to the event permissions set on Google Calendar. | `google_calendar` |
| `_check_organizer_validation` | ValidationError | For having a different organizer in your event, it is necessary that the organizer have its the system Calendar synced with Outlook Calendar. | `microsoft_calendar` |
| `_check_organizer_validation` | ValidationError | It is necessary adding the proposed organizer as attendee before saving the event. | `microsoft_calendar` |
| `_check_recurrence_overlapping` | UserError | Outlook limitation: in a recurrence, an event cannot be moved to or before the day of the previous event, and cannot be moved to or after the day of the following event. | `microsoft_calendar` |
| `_forbid_recurrence_update` | UserError | error_msg | `microsoft_calendar` |
| `_forbid_recurrence_creation` | UserError | Due to an Outlook Calendar limitation, recurrent events must be created directly in Outlook Calendar. | `microsoft_calendar` |
| `_ensure_attendees_have_email` | ValidationError | For a correct synchronization between the system and Outlook Calendar, all attendees must have an email address. However, some events do not respect this condition. As long as the events are incorrect, the calendars will not be synchronized. Either update the events/attendees or archive these events %(details)s: %(invalid_events)s | `microsoft_calendar` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_portal` | no | yes | no | no | `calendar` |
| `base.group_user` | yes | yes | yes | yes | `calendar` |
| `base.group_partner_manager` | yes | yes | yes | yes | `calendar` |
| `sales_team.group_sale_manager` | yes | yes | yes | yes | `crm` |
| `sales_team.group_sale_salesman` | yes | yes | yes | no | `crm` |
| `hr_holidays.group_hr_holidays_user` | yes | yes | yes | yes | `hr_holidays` |
| `group_hr_recruitment_user` | yes | yes | yes | yes | `hr_recruitment` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Own events | `[(4, ref('base.group_portal'))]` | `[('partner_ids', 'in', user.partner_id.id)]` | True | True | True | True |
| All Calendar Event for employees | `[(4,ref('base.group_user'))]` | `[(1, '=', 1)]` | True | True | True | True |
| Private events | global (all users) | `['\|', ('privacy', '!=', 'private'), '&', ('privacy', '=', 'private'), '\|', ('user_id', '=', user.id), ('partner_ids', 'in', user.partner_id.id)]` | False | True | True | True |

## Views (12)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `calendar.view_calendar_event_tree` | list |  | `name`, `start`, `stop`, `user_id`, `partner_ids`, `alarm_ids`, `categ_ids`, `recurrency`, `privacy`, `show_as`, `location`, `duration`, `description`, `allday`, `message_needaction` | `Send Mail` |  | `calendar` |
| `calendar.view_calendar_event_form` | form |  | `recurrence_update`, `res_model_name`, `active`, `user_can_edit`, `invalid_email_partner_ids`, `name`, `start_date`, `start`, `stop_date`, `stop`, `unavailable_partner_ids`, `duration`, `allday`, `location`, `videocall_location`, `res_id`, `res_model`, `videocall_source`, `access_token`, `current_status`, `show_as`, `privacy`, `should_show_status`, `attendees_count`, `accepted_count`, `tentative_count`, `declined_count`, `awaiting_count`, `partner_ids`, `notes`, `user_id`, `categ_ids`, `description`, `alarm_ids`, `recurrence_id`, `rrule_type`, `recurrency`, `rrule_type_ui`, `interval`, `rrule_type`, `recurrence_id`, `month_by`, `day`, `byday`, `weekday`, `end_type`, `count`, `until`, `event_tz`, `attendee_ids`, `partner_id`, `email`, `phone`, `state`, `partner_id`, `email`, `state` | `Send email`, `action_open_calendar_event`, `clear_videocall_location`, `action_join_video_call`, `set_discuss_videocall_location`, `EMAIL`, `Send Invitations`, `Uncertain`, `Accept`, `Decline`, `Uncertain`, `Accept`, `Decline` |  | `calendar` |
| `calendar.view_calendar_event_form_quick_create` | form |  | `invalid_email_partner_ids`, `name`, `duration`, `start`, `start_date`, `allday`, `stop`, `partner_ids`, `location`, `recurrence_update`, `access_token`, `videocall_location`, `videocall_source`, `videocall_location`, `privacy`, `notes` | `set_discuss_videocall_location`, `clear_videocall_location` |  | `calendar` |
| `calendar.view_calendar_event_calendar` | calendar |  | `attendees_count`, `accepted_count`, `declined_count`, `user_can_edit`, `is_highlighted`, `privacy`, `effective_privacy`, `recurrency`, `recurrence_update`, `location`, `partner_ids`, `videocall_location`, `res_model_name`, `alarm_ids`, `categ_ids`, `partner_id`, `description` |  |  | `calendar` |
| `calendar.view_calendar_event_search` | search |  | `name`, `partner_ids`, `user_id`, `location`, `show_as`, `categ_ids`, `description` |  | `My Meetings`, `Date`, `Busy`, `Free`, `Default Privacy`, `Public`, `Private`, `Only Internal Users`, `Recurrent`, `Archived`, `Responsible`, `Date` | `calendar` |
| `calendar_sms.view_calendar_event_tree_inherited` | xpath | `calendar.view_calendar_event_tree` |  | `Send SMS` |  | `calendar_sms` |
| `calendar_sms.view_calendar_event_form_inherited` | xpath | `calendar.view_calendar_event_form` |  | `SMS` |  | `calendar_sms` |
| `crm.view_crm_meeting_search` | xpath | `calendar.view_calendar_event_search` | `opportunity_id` |  |  | `crm` |
| `google_calendar.view_google_calendar_event` | field | `calendar.view_calendar_event_calendar` | `location`, `google_id` |  |  | `google_calendar` |
| `hr_calendar.view_calendar_event_calendar` | xpath | `calendar.view_calendar_event_calendar` |  |  |  | `hr_calendar` |
| `hr_holidays.view_calendar_event_form_inherit` | xpath | `calendar.view_calendar_event_form` |  |  |  | `hr_holidays` |
| `microsoft_calendar.view_microsoft_calendar_event` | field | `calendar.view_calendar_event_calendar` | `location`, `microsoft_id` |  |  | `microsoft_calendar` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `calendar.action_calendar_event` | Meetings | calendar,list,form |  |  |  | `calendar` |
| `crm.act_crm_opportunity_calendar_event_new` | Meetings | list,form,calendar |  | `{'default_duration': 4.0, 'default_opportunity_id': active_id}` |  | `crm` |

## Menus

| Menu | Name | Parent | Action | Sequence | Groups |
|---|---|---|---|---|---|
| `calendar.calendar_menu_config` | Configuration | `calendar.mail_menu_calendar` | `calendar.action_calendar_event` | 40 | `base.group_system,base.group_no_one` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `calendar.calendar_template_meeting_update` | Calendar: Event Update | {{object.name}}: Event update |
| `calendar.calendar_template_delete_event` | Calendar: Event Deleted | Deleted event: {{ object.name }} |

Machine-readable definition: `../../../schemas/data/entities/calendar.event.json`; views: `../../../schemas/interfaces/views/calendar.event.json`.
