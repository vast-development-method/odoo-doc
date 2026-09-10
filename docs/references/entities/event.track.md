# Event Track (`event.track`)

**Transport name:** `event.track`  
**Storage name:** `event_track`  
**Kind:** persistent entity (one table)  
**Defined by package:** `website_event_track`  
**Extended by packages:** `mass_mailing_event_track`, `website_event_track_live`, `website_event_track_quiz`

Description: Event Track

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`, `website.seo.metadata`, `website.published.mixin`, `website.searchable.mixin`
- Default ordering: `priority desc, date`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (62)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `name` | Title | single line text |  | required; translatable |
| `event_id` | Event | many to one | `event.event` | required; indexed |
| `active` | Active | boolean |  | default `True` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread |
| `company_id` | Company | many to one | `res.company` | related through path `event_id.company_id` |
| `tag_ids` | Tags | many to many | `event.track.tag` |  |
| `description` | Description | rich text |  | translatable |
| `color` | Agenda Color | integer |  |  |
| `priority` | Priority | selection |  | required; default `1` |
| `stage_id` | Stage | many to one | `event.track.stage` | required; default computed dynamically (_get_default_stage_id); changes are tracked in the message thread; indexed; not copied on duplication; on delete of the target: restrict |
| `legend_blocked` | Kanban Blocked Explanation | single line text |  | read only; related through path `stage_id.legend_blocked` |
| `legend_done` | Kanban Valid Explanation | single line text |  | read only; related through path `stage_id.legend_done` |
| `legend_normal` | Kanban Ongoing Explanation | single line text |  | read only; related through path `stage_id.legend_normal` |
| `kanban_state` | Kanban State | selection |  | required; default `normal`; not copied on duplication; Help: A track's kanban state indicates special situations affecting it:  * Grey is the default situation  * Red indicates something is preventing the progress of this track  * Green indicates the track is ready to be pulled to the next stage |
| `kanban_state_label` | Kanban State Label | single line text |  | computed by rule `_compute_kanban_state_label` and stored; changes are tracked in the message thread |
| `partner_id` | Contact | many to one | `res.partner` |  |
| `partner_name` | Name | single line text |  | computed by rule `_compute_partner_name` and stored; changes are tracked in the message thread |
| `partner_email` | Email | single line text |  | computed by rule `_compute_partner_email` and stored; changes are tracked in the message thread |
| `partner_phone` | Phone | single line text |  | computed by rule `_compute_partner_phone` and stored; changes are tracked in the message thread |
| `partner_biography` | Biography | rich text |  | computed by rule `_compute_partner_biography` and stored |
| `partner_function` | Job Position | single line text |  | computed by rule `_compute_partner_function` and stored |
| `partner_company_name` | Company Name | single line text |  | computed by rule `_compute_partner_company_name` and stored |
| `partner_tag_line` | Tag Line | single line text |  | computed by rule `_compute_partner_tag_line` (not stored); Help: Description of the partner (name, function and company name) |
| `image` | Speaker Photo | image |  | computed by rule `_compute_partner_image` and stored |
| `contact_email` | Contact Email | single line text |  | computed by rule `_compute_contact_email` and stored; changes are tracked in the message thread |
| `contact_phone` | Contact Phone | single line text |  | computed by rule `_compute_contact_phone` and stored; changes are tracked in the message thread |
| `location_id` | Location | many to one | `event.track.location` |  |
| `date` | Track Date | date and time |  | computed by rule `_compute_date` and stored; writable through an inverse rule |
| `date_end` | Track End Date | date and time |  | computed by rule `_compute_end_date` and stored; writable through an inverse rule |
| `duration` | Duration | float |  | default `0.5` |
| `is_track_live` | Is Track Live | boolean |  | computed by rule `_compute_track_time_data` (not stored) |
| `is_track_soon` | Is Track Soon | boolean |  | computed by rule `_compute_track_time_data` (not stored) |
| `is_track_today` | Is Track Today | boolean |  | computed by rule `_compute_track_time_data` (not stored) |
| `is_track_upcoming` | Is Track Upcoming | boolean |  | computed by rule `_compute_track_time_data` (not stored) |
| `is_track_done` | Is Track Done | boolean |  | computed by rule `_compute_track_time_data` (not stored) |
| `is_one_day` | Is One Day | boolean |  | computed by rule `_compute_field_is_one_day` (not stored) |
| `track_start_remaining` | Minutes before track starts | integer |  | computed by rule `_compute_track_time_data` (not stored); Help: Remaining time before track starts (seconds) |
| `track_start_relative` | Minutes compare to track start | integer |  | computed by rule `_compute_track_time_data` (not stored); Help: Relative time compared to track start (seconds) |
| `website_image` | Website Image | image |  |  |
| `website_image_url` | Image uniform resource locator | single line text |  | computed by rule `_compute_website_image_url` (not stored) |
| `header_visible` | Header Visible | boolean |  | related through path `event_id.header_visible` |
| `footer_visible` | Footer Visible | boolean |  | related through path `event_id.footer_visible` |
| `event_track_visitor_ids` | Track Visitors | one to many | `event.track.visitor` | visible only to groups `event.group_event_user`; inverse field `track_id` |
| `is_reminder_on` | Is Reminder On | boolean |  | computed by rule `_compute_is_reminder_on` (not stored) |
| `wishlist_visitor_ids` | Visitor Wishlist | many to many | `website.visitor` | computed by rule `_compute_wishlist_visitor_ids` (not stored); searchable through a search rule; visible only to groups `event.group_event_user` |
| `wishlist_visitor_count` | # Wishlisted | integer |  | computed by rule `_compute_wishlist_visitor_ids` (not stored); visible only to groups `event.group_event_user` |
| `wishlisted_by_default` | Always Wishlisted | boolean |  | Help: If set, the talk will be set as favorite for each attendee registered to the event. |
| `website_cta` | Magic Button | boolean |  | Help: Display a Call to Action button to your Attendees while they watch your Track. |
| `website_cta_title` | Button Title | single line text |  |  |
| `website_cta_url` | Button Target uniform resource locator | single line text |  |  |
| `website_cta_delay` | Show Button | integer |  |  |
| `is_website_cta_live` | Is call to action Live | boolean |  | computed by rule `_compute_cta_time_data` (not stored); Help: CTA button is available |
| `website_cta_start_remaining` | Minutes before call to action starts | integer |  | computed by rule `_compute_cta_time_data` (not stored); Help: Remaining time before CTA starts (seconds) |
| `youtube_video_url` | YouTube Video Link | single line text |  |  |
| `youtube_video_id` | YouTube video identifier | single line text |  | computed by rule `_compute_youtube_video_id` (not stored); Help: Extracted from the video URL and used to infer various links (embed/thumbnail/...) |
| `is_youtube_replay` | Is YouTube Replay | boolean |  | Help: Check this option if the video is already available on YouTube to avoid showing 'Direct' options (Chat, ...) |
| `is_youtube_chat_available` | Is Chat Available | boolean |  | computed by rule `_compute_is_youtube_chat_available` (not stored) |
| `quiz_id` | Quiz | many to one | `event.quiz` | computed by rule `_compute_quiz_id` and stored; visible only to groups `event.group_event_user` |
| `quiz_ids` | Quizzes | one to many | `event.quiz` | inverse field `event_track_id` |
| `quiz_questions_count` | # Quiz Questions | integer |  | computed by rule `_compute_quiz_questions_count` (not stored); visible only to groups `event.group_event_user` |
| `is_quiz_completed` | Is Quiz Done | boolean |  | computed by rule `_compute_quiz_data` (not stored) |
| `quiz_points` | Quiz Points | integer |  | computed by rule `_compute_quiz_data` (not stored) |

## Selection values

### `priority` (Priority)

| Value | Label |
|---|---|
| `0` | Low |
| `1` | Medium |
| `2` | High |
| `3` | Highest |

### `kanban_state` (Kanban State)

| Value | Label |
|---|---|
| `normal` | Grey |
| `done` | Green |
| `blocked` | Red |

## State fields

State machine fields of this entity: `kanban_state`. Transitions are specified in the domain documents.

## Operations (50)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_get_default_stage_id` | preparation rule | self | `website_event_track` | model |  |
| `_compute_website_url` | computation | self | `website_event_track` | depends: `name` |  |
| `_compute_kanban_state_label` | computation | self | `website_event_track` | depends: `stage_id`, `kanban_state` |  |
| `_compute_partner_name` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_partner_email` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_partner_phone` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_partner_biography` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_partner_function` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_partner_company_name` | computation | self | `website_event_track` | depends: `partner_id`, `partner_id.company_type` |  |
| `_compute_partner_tag_line` | computation | self | `website_event_track` | depends: `partner_name`, `partner_function`, `partner_company_name` |  |
| `_compute_partner_image` | computation | self | `website_event_track` | depends: `partner_id` |  |
| `_compute_contact_email` | computation | self | `website_event_track` | depends: `partner_id`, `partner_id.email` |  |
| `_compute_contact_phone` | computation | self | `website_event_track` | depends: `partner_id`, `partner_id.phone` |  |
| `_compute_date` | computation | self | `website_event_track` | depends: `date_end`, `duration` |  |
| `_inverse_date` | inverse computation | self | `website_event_track` |  |  |
| `_compute_end_date` | computation | self | `website_event_track` | depends: `date`, `duration` |  |
| `_inverse_end_date` | inverse computation | self | `website_event_track` |  |  |
| `_compute_website_image_url` | computation | self | `website_event_track_live`, `website_event_track` | depends: `image`, `partner_id.image_256`; depends: `youtube_video_id`, `is_youtube_replay`, `date_end`, `is_track_done` |  |
| `_compute_is_reminder_on` | computation | self | `website_event_track` | depends: `wishlisted_by_default`, `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.partner_id`, `event_track_visitor_ids.is_wishlisted`, `event_track_visitor_ids.is_blacklisted`; depends_context: `uid` |  |
| `_compute_wishlist_visitor_ids` | computation | self | `website_event_track` | depends: `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.is_wishlisted` |  |
| `_search_wishlist_visitor_ids` | search rule | self, operator, operand | `website_event_track` |  |  |
| `_compute_track_time_data` | computation | self | `website_event_track` | depends: `date`, `date_end` | Compute start and remaining time for track itself. Do everything in UTC as we compute only time deltas here. |
| `_compute_cta_time_data` | computation | self | `website_event_track` | depends: `date`, `date_end`, `website_cta`, `website_cta_delay` | Compute start and remaining time for track itself. Do everything in UTC as we compute only time deltas here. |
| `_compute_field_is_one_day` | computation | self | `website_event_track` | depends: `date`, `date_end`, `event_id` |  |
| `create` | lifecycle override | self, vals_list | `website_event_track` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `website_event_track` |  |  |
| `_synchronize_with_stage` | internal rule | self, stage | `website_event_track` |  |  |
| `_search_get_detail` | search rule | self, website, order, options | `website_event_track` | model |  |
| `_mail_get_timezone` | messaging hook | self | `website_event_track` |  |  |
| `_message_add_default_recipients` | messaging hook | self | `website_event_track` |  |  |
| `_message_post_after_hook` | messaging hook | self, message, msg_vals | `website_event_track` |  |  |
| `_track_template` | messaging hook | self, changes | `website_event_track` |  |  |
| `_track_subtype` | messaging hook | self, init_values | `website_event_track` |  |  |
| `open_track_speakers_list` | operation | self | `website_event_track` |  |  |
| `get_backend_menu_id` | operation | self | `website_event_track` |  |  |
| `_get_event_track_visitors` | preparation rule | self, force_create | `website_event_track` |  |  |
| `_get_ics_file` | preparation rule | self | `website_event_track` |  | Return iCalendar file for the event track. :return: a dict of .ics file content for each event track |
| `_get_track_suggestions` | preparation rule | self, restrict_domain, limit | `website_event_track` |  | Returns the next tracks suggested after going to the current one given by self. Tracks always belong to the same event.  Heuristic is    * live first;   * then ordered by start date, finished being sent to the end;   * wishlisted (manually or by default);   * tag matching with current track;   * location matching with current track;   * finally a random to have an "equivalent wave" randomly given;  :param restrict_domain: an additional domain to restrict candidates; :param limit: number of tracks to return; |
| `_get_track_calendar_description` | preparation rule | self | `website_event_track` |  |  |
| `_get_track_calendar_reminder_dates` | preparation rule | self | `website_event_track` |  | Get dates of the event if the track does not have any. A warning is added in elements that display times of the event instead of those of the track. This way, visitors can add a reminder and check later if the track has been updated since the sending of the mail. |
| `_get_track_calendar_reminder_times_warning` | preparation rule | self | `website_event_track` |  | Generate a warning indicating that the times displayed correspond to those of the event because the track does not have any, to avoid misunderstanding from visitors. |
| `_get_track_calendar_urls` | preparation rule | self | `website_event_track` |  |  |
| `_mailing_get_default_domain` | messaging hook | self, mailing | `mass_mailing_event_track` |  |  |
| `_compute_youtube_video_id` | computation | self | `website_event_track_live` | depends: `youtube_video_url` |  |
| `_compute_is_youtube_chat_available` | computation | self | `website_event_track_live` | depends: `youtube_video_url`, `is_youtube_replay`, `date`, `date_end`, `is_track_upcoming`, `is_track_live` |  |
| `_compute_quiz_id` | computation | self | `website_event_track_quiz` | depends: `quiz_ids.event_track_id` |  |
| `_compute_quiz_questions_count` | computation | self | `website_event_track_quiz` | depends: `quiz_id.question_ids` |  |
| `_compute_quiz_data` | computation | self | `website_event_track_quiz` | depends: `quiz_id`, `event_track_visitor_ids.visitor_id`, `event_track_visitor_ids.partner_id`, `event_track_visitor_ids.quiz_completed`, `event_track_visitor_ids.quiz_points`; depends_context: `uid` |  |
| `action_add_quiz` | user action | self | `website_event_track_quiz` |  |  |
| `action_view_quiz` | user action | self | `website_event_track_quiz` |  |  |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_search_wishlist_visitor_ids` | UserError | Unsupported 'Not In' operation on track wishlist visitors | `website_event_track` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `base.group_public` | no | yes | no | no | `website_event_track` |
| `base.group_portal` | no | yes | no | no | `website_event_track` |
| `base.group_user` | no | yes | no | no | `website_event_track` |
| `event.group_event_user` | yes | yes | yes | no | `website_event_track` |
| `event.group_event_manager` | yes | yes | yes | yes | `website_event_track` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Event Tracks: public/portal: published | `[(4, ref('base.group_public')), (4, ref('base.group_portal'))]` | `[('website_published', '=', True)]` | True | False | False | False |

## Views (10)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `website_event_track.event_track_view_form_quick_create` | form |  | `event_id`, `name`, `date` |  |  | `website_event_track` |
| `website_event_track.view_event_track_kanban` | kanban |  | `website_url`, `legend_blocked`, `legend_normal`, `legend_done`, `color`, `name`, `duration`, `tag_ids`, `priority`, `activity_ids`, `kanban_state`, `partner_id` |  |  | `website_event_track` |
| `website_event_track.view_event_track_calendar` | calendar |  | `location_id`, `event_id`, `partner_id`, `user_id` |  |  | `website_event_track` |
| `website_event_track.view_event_track_search` | search |  | `name`, `tag_ids`, `partner_id`, `location_id`, `event_id`, `stage_id` |  | `My Tracks`, `Published`, `Unread Messages`, `Always Wishlisted`, `filter_date`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Future Activities`, `Responsible`, `Stage`, `Date`, `Event`, `Location` | `website_event_track` |
| `website_event_track.view_event_track_form` | form |  | `stage_id`, `website_url`, `wishlist_visitor_count`, `wishlist_visitor_count`, `is_published`, `legend_blocked`, `legend_normal`, `legend_done`, `name`, `website_image`, `kanban_state`, `date`, `location_id`, `duration`, `active`, `wishlisted_by_default`, `company_id`, `user_id`, `event_id`, `tag_ids`, `color`, `partner_id`, `contact_email`, `contact_phone`, `partner_name`, `partner_email`, `partner_phone`, `partner_function`, `partner_company_name`, `image`, `partner_biography`, `description`, `website_cta`, `website_cta_title`, `website_cta_url`, `website_cta_delay` | `%(website_event_track.website_visitor_action_from_track)d` |  | `website_event_track` |
| `website_event_track.view_event_track_tree` | list |  | `name`, `active`, `partner_name`, `partner_id`, `partner_email`, `partner_phone`, `event_id`, `wishlisted_by_default`, `wishlist_visitor_count`, `stage_id`, `color`, `activity_exception_decoration`, `location_id` |  |  | `website_event_track` |
| `website_event_track.view_event_track_graph` | graph |  | `location_id`, `duration`, `website_cta_delay`, `color` |  |  | `website_event_track` |
| `website_event_track_live.event_track_view_form` | xpath | `website_event_track.view_event_track_form` | `youtube_video_url`, `is_youtube_replay` |  |  | `website_event_track_live` |
| `website_event_track_live.event_track_view_list` | xpath | `website_event_track.view_event_track_tree` | `youtube_video_url` |  |  | `website_event_track_live` |
| `website_event_track_quiz.event_track_view_form` | field | `website_event_track.view_event_track_form` | `stage_id`, `quiz_id` | `Add Quiz` |  | `website_event_track_quiz` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `website_event_track.action_event_track` | Event Tracks | kanban,list,form,calendar,graph,activity |  |  |  | `website_event_track` |
| `website_event_track.action_event_track_from_event` | Event Tracks | kanban,list,form,calendar,graph,activity |  | `{'search_default_event_id': active_id, 'default_event_id': active_id}` |  | `website_event_track` |
| `website_event_track.event_track_action_from_visitor` | Wishlisted Tracks | kanban,list,form | `[('wishlist_visitor_ids', 'in', [active_id])]` |  |  | `website_event_track` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `website_event_track.mail_template_data_track_confirmation` | Event: Track Confirmation | Confirmation of {{ object.name }} |
| `website_event_track.mail_template_data_track_reminder` | Add reminder via email | Add talk reminder: {{ object.name }} |

Machine-readable definition: `../../../schemas/data/entities/event.track.json`; views: `../../../schemas/interfaces/views/event.track.json`.
