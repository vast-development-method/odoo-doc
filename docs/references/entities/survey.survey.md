# Survey (`survey.survey`)

**Transport name:** `survey.survey`  
**Storage name:** `survey_survey`  
**Kind:** persistent entity (one table)  
**Defined by package:** `survey`  
**Extended by packages:** `hr_recruitment_survey`, `hr_skills_survey`, `survey_crm`, `survey_crm`, `website_slides_survey`

Description: Survey

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `create_date DESC`
- Display name field: `title`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (66)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `survey_type` | Survey Type | selection |  | required; default `custom`; on delete of the target: {"recruitment": "set default"}; extended by packages `hr_recruitment_survey` |
| `lang_ids` | Languages | many to many | `res.lang` | default computed dynamically (lambda self: self.env['res.lang']._lang_get(self.env.context.get('lang') or self.env['res.lang'].get_installed()[0][0])); restricted by domain `lambda self: [('id', 'in', [lang.id for lang in self.env['res.lang']._get_active_by('code').values()])]`; Help: Leave the field empty to support all installed languages. |
| `allowed_survey_types` | Allowed survey types | structured document |  | computed by rule `_compute_allowed_survey_types` (not stored) |
| `title` | Survey Title | single line text |  | required; translatable |
| `color` | Color Index | integer |  | default  |
| `description` | Description | rich text |  | translatable; Help: The description will be displayed on the home page of the survey. You can use this to give the purpose and guidelines to your candidates before they start it. |
| `description_done` | End Message | rich text |  | translatable; Help: This message will be displayed when survey is completed |
| `background_image` | Background Image | image |  |  |
| `background_image_url` | Background Url | single line text |  | computed by rule `_compute_background_image_url` (not stored) |
| `active` | Active | boolean |  | default `True` |
| `user_id` | Responsible | many to one | `res.users` | default computed dynamically (lambda self: self.env.user); changes are tracked in the message thread; restricted by domain `[["share", "=", false]]` |
| `restrict_user_ids` | Restricted to | many to many | `res.users` | changes are tracked in the message thread; restricted by domain `[["share", "=", false]]` |
| `question_and_page_ids` | Sections and Questions | one to many | `survey.question` | inverse field `survey_id` |
| `page_ids` | Pages | one to many | `survey.question` | computed by rule `_compute_page_and_question_ids` (not stored) |
| `question_ids` | Questions | one to many | `survey.question` | computed by rule `_compute_page_and_question_ids` (not stored) |
| `question_count` | # Questions | integer |  | computed by rule `_compute_page_and_question_ids` (not stored) |
| `questions_layout` | Pagination | selection |  | required; default `page_per_question` |
| `questions_selection` | Question Selection | selection |  | required; default `all`; Help: If randomized is selected, you can configure the number of random questions by section. This mode is ignored in live session. |
| `progression_mode` | Display Progress as | selection |  | default `percent`; Help: If Number is selected, it will display the number of questions answered on the total number of question to answer. |
| `user_input_ids` | User responses | one to many | `survey.user_input` | read only; inverse field `survey_id` |
| `access_mode` | Access Mode | selection |  | required; default `public` |
| `access_token` | Access Token | single line text |  | default computed dynamically (lambda self: self._get_default_access_token()); not copied on duplication |
| `users_login_required` | Require Login | boolean |  | Help: If checked, users have to login before answering even with a valid token. |
| `users_can_go_back` | Users can go back | boolean |  | Help: If checked, users can go back to previous pages. |
| `users_can_signup` | Users can signup | boolean |  | computed by rule `_compute_users_can_signup` (not stored) |
| `answer_count` | Registered | integer |  | computed by rule `_compute_survey_statistic` (not stored) |
| `answer_done_count` | Attempts | integer |  | computed by rule `_compute_survey_statistic` (not stored) |
| `answer_score_avg` | Avg Score (%) | float |  | computed by rule `_compute_survey_statistic` (not stored) |
| `answer_duration_avg` | Average Duration | float |  | computed by rule `_compute_answer_duration_avg` (not stored); Help: Average duration of the survey (in hours) |
| `success_count` | Success | integer |  | computed by rule `_compute_survey_statistic` (not stored) |
| `success_ratio` | Success Ratio (%) | integer |  | computed by rule `_compute_survey_statistic` (not stored) |
| `scoring_type` | Scoring | selection |  | required; computed by rule `_compute_scoring_type` and stored; precomputed before insertion |
| `scoring_success_min` | Required Score (%) | float |  | default `80.0` |
| `scoring_max_obtainable` | Maximum obtainable score | float |  | computed by rule `_compute_scoring_max_obtainable` (not stored) |
| `is_attempts_limited` | Limited number of attempts | boolean |  | computed by rule `_compute_is_attempts_limited` and stored; Help: Check this option if you want to limit the number of attempts per user |
| `attempts_limit` | Number of attempts | integer |  | default `1` |
| `is_time_limited` | The survey is limited in time | boolean |  |  |
| `time_limit` | Time limit (minutes) | float |  | default `10` |
| `certification` | Is a Certification | boolean |  | computed by rule `_compute_certification` and stored; precomputed before insertion |
| `certification_mail_template_id` | Certified Email Template | many to one | `mail.template` | restricted by domain `[('model', '=', 'survey.user_input')]`; Help: Automated email sent to the user when they succeed the certification, containing their certification document. |
| `certification_report_layout` | Certification template | selection |  | default `modern_purple` |
| `certification_give_badge` | Give Badge | boolean |  | computed by rule `_compute_certification_give_badge` and stored; not copied on duplication |
| `certification_badge_id` | Certification Badge | many to one | `gamification.badge` | indexed (btree_not_null); not copied on duplication |
| `certification_badge_id_dummy` | Certification Badge | many to one |  | related through path `certification_badge_id` |
| `session_available` | Live session available | boolean |  | computed by rule `_compute_session_available` (not stored) |
| `session_state` | Session State | selection |  | not copied on duplication |
| `session_code` | Session Code | single line text |  | computed by rule `_compute_session_code` and stored; not copied on duplication; precomputed before insertion; Help: This code will be used by your attendees to reach your session. Feel free to customize it however you like! |
| `session_link` | Session Link | single line text |  | computed by rule `_compute_session_link` (not stored) |
| `session_question_id` | Current Question | many to one | `survey.question` | not copied on duplication; Help: The current question of the survey session. |
| `session_start_time` | Current Session Start Time | date and time |  | not copied on duplication |
| `session_question_start_time` | Current Question Start Time | date and time |  | not copied on duplication; Help: The time at which the current question has started, used to handle the timer for attendees. |
| `session_answer_count` | Answers Count | integer |  | computed by rule `_compute_session_answer_count` (not stored) |
| `session_question_answer_count` | Question Answers Count | integer |  | computed by rule `_compute_session_question_answer_count` (not stored) |
| `session_show_leaderboard` | Show Session Leaderboard | boolean |  | computed by rule `_compute_session_show_leaderboard` (not stored); Help: Whether or not we want to show the attendees leaderboard for this survey. |
| `session_speed_rating` | Reward quick answers | boolean |  | Help: Attendees get more points if they answer quickly |
| `session_speed_rating_time_limit` | Time limit (seconds) | integer |  | Help: Default time given to receive additional points for right answers |
| `has_conditional_questions` | Contains conditional questions | boolean |  | computed by rule `_compute_has_conditional_questions` (not stored) |
| `hr_job_ids` | Job Position | one to many | `hr.job` | inverse field `survey_id` |
| `certification_validity_months` | Validity | integer |  | Help: Specify the number of months the certification is valid after being awarded. Enter 0 for certifications that never expire. |
| `generate_lead` | Lead Generating | boolean |  | computed by rule `_compute_generate_lead` and stored |
| `lead_count` | Leads | integer |  | computed by rule `_compute_lead_count` (not stored); Help: Number of leads created by this survey |
| `lead_ids` | Lead | one to many | `crm.lead` | inverse field `origin_survey_id` |
| `team_id` | Assign Leads to | many to one | `crm.team` | indexed (btree_not_null); on delete of the target: set null |
| `slide_ids` | Certification Slides | one to many | `slide.slide` | inverse field `survey_id`; Help: The slides this survey is linked to through the e-learning application |
| `slide_channel_ids` | Certification Courses | one to many | `slide.channel` | computed by rule `_compute_slide_channel_data` (not stored); visible only to groups `website_slides.group_website_slides_officer`; Help: The courses this survey is linked to through the e-learning application |
| `slide_channel_count` | Courses Count | integer |  | computed by rule `_compute_slide_channel_data` (not stored); visible only to groups `website_slides.group_website_slides_officer` |

## Selection values

### `survey_type` (Survey Type)

| Value | Label |
|---|---|
| `survey` | Survey |
| `live_session` | Live session |
| `assessment` | Assessment |
| `custom` | Custom |
| `recruitment` | Recruitment |

### `questions_layout` (Pagination)

| Value | Label |
|---|---|
| `page_per_question` | One page per question |
| `page_per_section` | One page per section |
| `one_page` | One page with all the questions |

### `questions_selection` (Question Selection)

| Value | Label |
|---|---|
| `all` | All questions |
| `random` | Randomized per Section |

### `progression_mode` (Display Progress as)

| Value | Label |
|---|---|
| `percent` | Percentage left |
| `number` | Number |

### `access_mode` (Access Mode)

| Value | Label |
|---|---|
| `public` | Anyone with the link |
| `token` | Invited people only |

### `scoring_type` (Scoring)

| Value | Label |
|---|---|
| `no_scoring` | No scoring |
| `scoring_with_answers_after_page` | Scoring with answers after each page |
| `scoring_with_answers` | Scoring with answers at the end |
| `scoring_without_answers` | Scoring without answers |

### `certification_report_layout` (Certification template)

| Value | Label |
|---|---|
| `modern_purple` | Modern Purple |
| `modern_blue` | Modern Blue |
| `modern_gold` | Modern Gold |
| `classic_purple` | Classic Purple |
| `classic_blue` | Classic Blue |
| `classic_gold` | Classic Gold |

### `session_state` (Session State)

| Value | Label |
|---|---|
| `ready` | Ready |
| `in_progress` | In Progress |

## State fields

State machine fields of this entity: `session_state`. Transitions are specified in the domain documents.

## Database constraints and indexes (8)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_access_token_unique` | Constraint | `unique(access_token)` | Access token should be unique | `survey` |
| `_session_code_unique` | Constraint | `unique(session_code)` | Session code should be unique | `survey` |
| `_certification_check` | Constraint | `CHECK( scoring_type!='no_scoring' OR certification=False )` | You can only create certifications for surveys that have a scoring mechanism. | `survey` |
| `_scoring_success_min_check` | Constraint | `CHECK( scoring_success_min IS NULL OR (scoring_success_min>=0 AND scoring_success_min<=100) )` | The percentage of success has to be defined between 0 and 100. | `survey` |
| `_time_limit_check` | Constraint | `CHECK( (is_time_limited=False) OR (time_limit is not null AND time_limit > 0) )` | The time limit needs to be a positive number if the survey is time limited. | `survey` |
| `_attempts_limit_check` | Constraint | `CHECK( (is_attempts_limited=False) OR (attempts_limit is not null AND attempts_limit > 0) )` | The attempts limit needs to be a positive number if the survey has a limited number of attempts. | `survey` |
| `_badge_uniq` | Constraint | `unique (certification_badge_id)` | The badge for each survey should be unique! | `survey` |
| `_session_speed_rating_has_time_limit` | Constraint | `CHECK (session_speed_rating != TRUE OR session_speed_rating_time_limit IS NOT NULL AND session_speed_rating_time_limit > 0)` | A positive default time limit is required when the session rewards quick answers. | `survey` |

## Operations (86)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `action_load_survey_template_sample` | user action | self, template_key | `survey` | model |  |
| `get_survey_templates_data` | operation | self | `survey_crm`, `survey` | model |  |
| `_get_survey_template_values` | preparation rule | self, template_key | `survey_crm`, `survey` |  |  |
| `_prepare_survey_template_values` | preparation rule | self | `survey` | model |  |
| `_prepare_assessment_template_values` | preparation rule | self | `survey` | model |  |
| `_prepare_live_session_template_values` | preparation rule | self | `survey` | model |  |
| `action_load_sample_custom` | user action | self | `survey` | model |  |
| `action_show_sample` | user action | self | `survey` |  |  |
| `_get_default_access_token` | preparation rule | self | `survey` | model |  |
| `default_get` | lifecycle override | self, fields | `survey` | model |  |
| `_compute_background_image_url` | computation | self | `survey` | depends: `background_image`, `access_token` |  |
| `_compute_scoring_max_obtainable` | computation | self | `survey` | depends: `question_and_page_ids`, `question_and_page_ids.suggested_answer_ids`, `question_and_page_ids.suggested_answer_ids.answer_score` |  |
| `_compute_users_can_signup` | computation | self | `survey` |  |  |
| `_compute_survey_statistic` | computation | self | `survey` | depends: `user_input_ids.state`, `user_input_ids.test_entry`, `user_input_ids.scoring_percentage`, `user_input_ids.scoring_success` |  |
| `_compute_answer_duration_avg` | computation | self | `survey` | depends: `user_input_ids.survey_id`, `user_input_ids.start_datetime`, `user_input_ids.end_datetime` |  |
| `_compute_page_and_question_ids` | computation | self | `survey` | depends: `question_and_page_ids` |  |
| `_compute_is_attempts_limited` | computation | self | `survey` | depends: `question_and_page_ids.triggering_answer_ids`, `users_login_required`, `access_mode` |  |
| `_compute_session_answer_count` | computation | self | `survey` | depends: `session_start_time`, `user_input_ids` | We have to loop since our result is dependent of the survey.session_start_time. This field is currently used to display the count about a single survey, in the context of sessions, so it should not matter too much. |
| `_compute_session_question_answer_count` | computation | self | `survey` | depends: `session_question_id`, `session_start_time`, `user_input_ids.user_input_line_ids` | We have to loop since our result is dependent of the survey.session_question_id and the survey.session_start_time. This field is currently used to display the count about a single survey, in the context of sessions, so it should not matter too much. |
| `_compute_session_code` | computation | self | `survey` | depends: `access_token` |  |
| `_compute_session_link` | computation | self | `survey` | depends: `session_code` |  |
| `_compute_session_show_leaderboard` | computation | self | `survey` | depends: `scoring_type`, `question_and_page_ids.save_as_nickname` |  |
| `_compute_has_conditional_questions` | computation | self | `survey` | depends: `question_and_page_ids.triggering_answer_ids` |  |
| `_compute_certification` | computation | self | `survey` | depends: `scoring_type` |  |
| `_compute_certification_give_badge` | computation | self | `survey` | depends: `users_login_required`, `certification` |  |
| `_compute_scoring_type` | computation | self | `survey` | depends: `certification` |  |
| `_compute_session_available` | computation | self | `survey` | depends: `survey_type`, `certification` |  |
| `_compute_allowed_survey_types` | computation | self | `hr_recruitment_survey`, `survey` | depends_context: `uid`; depends: `survey_type` |  |
| `_onchange_survey_type` | on change | self | `survey` | onchange: `survey_type` |  |
| `_onchange_session_speed_rating` | on change | self | `survey` | onchange: `session_speed_rating`, `session_speed_rating_time_limit` | Show impact on questions in the form view (before survey is saved). |
| `_onchange_restrict_user_ids` | on change | self | `survey` | onchange: `restrict_user_ids`, `user_id` | Add survey user_id to restrict_user_ids when: - restrict_user_ids is not False - user_id is not part of restrict_user_ids - user_id is not a survey manager |
| `_check_scoring_after_page_availability` | validation | self | `survey` | constrains: `scoring_type`, `users_can_go_back` |  |
| `_check_survey_responsible_access` | validation | self | `survey` | constrains: `user_id`, `restrict_user_ids` | When:     - a survey access is restricted to a list of users     - and there is a survey responsible,     - and this responsible is not survey manager (just survey officer), check the responsible is part of the list. |
| `create` | lifecycle override | self, vals_list | `survey` | model_create_multi |  |
| `write` | lifecycle override | self, vals | `survey` |  |  |
| `copy` | lifecycle override | self, default | `survey` |  | Correctly copy the 'triggering_answer_ids' field from the original to the clone.  This needs to be done in post-processing to make sure we get references to the newly created answers from the copy instead of references to the answers of the original. This implementation assumes that the order of created answers will be kept between the original and the clone, using 'zip()' to match the records between the two.  Note that when `question_ids` is provided in the default parameter, it falls back to the standard copy, meaning that triggering logic will not be maintained. |
| `copy_data` | lifecycle override | self, default | `survey` |  |  |
| `action_archive` | lifecycle override | self | `survey` |  |  |
| `action_unarchive` | lifecycle override | self | `survey` |  |  |
| `_create_answer` | internal rule | self, user, partner, email, test_entry, check_attempts, **additional_vals | `survey` |  | Main entry point to get a token back or create a new one. This method does check for current user access in order to explicitly validate security.    :param user: target user asking for a token; it might be void or a                public user in which case an email is welcomed;   :param email: email of the person asking the token is no user exists; |
| `_check_answer_creation` | validation | self, user, partner, email, test_entry, check_attempts, invite_token | `survey` |  | Ensure conditions to create new tokens are met. |
| `_prepare_user_input_predefined_questions` | preparation rule | self | `survey` |  | Will generate the questions for a randomized survey. It uses the random_questions_count of every sections of the survey to pick a random number of questions and returns the merged recordset |
| `_can_go_back` | internal rule | self, answer, page_or_question | `survey` |  | Check if the user can go back to the previous question/page for the currently viewed question/page. Back button needs to be configured on survey and, depending on the layout: - In 'page_per_section', we can go back if we're not on the first page - In 'page_per_question', we can go back if:   - It is not a session answer (doesn't make sense to go back in session context)   - We are not on the first question   - The survey does not have pages OR this is not the first page of the survey     (pages are displayed in 'page_per_question' layout when they have a description, see PR#44271) |
| `_has_attempts_left` | internal rule | self, partner, email, invite_token | `survey` |  |  |
| `_get_number_of_attempts_lefts` | preparation rule | self, partner, email, invite_token | `survey` |  | Returns the number of attempts left. |
| `_get_pages_or_questions` | preparation rule | self, user_input | `survey` | model | Returns the pages or questions (depending on the layout) that will be shown to the user taking the survey. In 'page_per_question' layout, we also want to show pages that have a description. |
| `_get_pages_and_questions_to_show` | preparation rule | self | `survey` |  | Filter question_and_pages_ids to include only valid pages and questions.  Pages are invalid if they have no description. Questions are invalid if they are conditional and all their triggers are invalid. Triggers are invalid if they:   - Are a page (not a question)   - Have the wrong question type (`simple_choice` and `multiple_choice` are supported)   - Are misplaced (positioned after the conditional question)   - They are themselves conditional and were found invalid |
| `_get_next_page_or_question` | preparation rule | self, user_input, page_or_question_id, go_back | `survey` |  | Generalized logic to retrieve the next question or page to show on the survey. It's based on the page_or_question_id parameter, that is usually the currently displayed question/page.  There is a special case when the survey is configured with conditional questions: - for "page_per_question" layout, the next question to display depends on the selected answers and   the questions 'hierarchy'. - for "page_per_section" layout, before returning the result, we check that it contains at least a question   (all section questions could be disabled based on previously selected answers)  The whole logic  |
| `_is_first_page_or_question` | internal rule | self, page_or_question | `survey` |  | This method checks if the given question or page is the first one to display. If the first section of the survey as a description, this will be the first screen to display. else, the first question will be the first screen to be displayed. This method is used for survey session management where the host should not be able to go back on the first page or question. |
| `_is_last_page_or_question` | internal rule | self, user_input, page_or_question | `survey` |  | Check if the given question or page is the last one, accounting for conditional questions.  A question/page will be determined as the last one if any of the following is true:   - The survey layout is "one_page",   - There are no more questions/page after `page_or_question` in `user_input`,   - All the following questions are conditional AND were not triggered by previous answers.     Not accounting for the question/page own conditionals. |
| `_get_survey_questions` | preparation rule | self, answer, page_id, question_id | `survey` |  | Returns a tuple containing: the survey question and the passed question_id / page_id based on the question_layout and the fact that it's a session or not.  Breakdown of use cases: - We are currently running a session   We return the current session question and it's id - The layout is page_per_section   We return the questions for that page and the passed page_id - The layout is page_per_question   We return the question for the passed question_id and the question_id - The layout is one_page   We return all the questions of the survey and None  In addition, we cross the returned questions with |
| `_get_conditional_maps` | preparation rule | self | `survey` |  |  |
| `_session_open` | internal rule | self | `survey` |  | The session start is sudo'ed to allow survey user to manage sessions of surveys they do not own.  We flush after writing to make sure it's updated before bus takes over. |
| `_get_session_next_question` | preparation rule | self, go_back | `survey` |  |  |
| `_get_session_most_voted_answers` | preparation rule | self | `survey` |  | In sessions of survey that has conditional questions, as the survey is passed at the same time by many users, we need to extract the most chosen answers, to determine the next questions to display. |
| `_prepare_leaderboard_values` | preparation rule | self | `survey` |  | The leaderboard is descending and takes the total of the attendee points minus the current question score. We need both the total and the current question points to be able to show the attendees leaderboard and shift their position based on the score they have on the current question. This prepares a structure containing all the necessary data for the animations done on the frontend side. The leaderboard is sorted based on attendees score *before* the current question. The frontend will shift positions around accordingly. |
| `check_validity` | operation | self | `survey` |  |  |
| `action_send_survey` | user action | self | `survey` |  | Open a window to compose an email, pre-filled with the survey message |
| `action_start_survey` | user action | self, answer | `survey` |  | Open the website page with the survey form |
| `action_print_survey` | user action | self, answer | `survey` |  | Open the website page with the survey printable view |
| `action_result_survey` | user action | self | `survey` |  | Open the website page with the survey results view |
| `action_test_survey` | user action | self | `survey` |  | Open the website page with the survey form into test mode |
| `action_survey_user_input_completed` | user action | self | `hr_recruitment_survey`, `survey` |  |  |
| `action_survey_user_input_certified` | user action | self | `survey` |  |  |
| `action_survey_user_input` | user action | self | `survey` |  |  |
| `action_survey_preview_certification_template` | user action | self | `survey` |  |  |
| `action_start_session` | user action | self | `survey` |  | Sets the necessary fields for the session to take place and starts it. The write is sudo'ed because a survey user can start a session even if it's not their own survey. |
| `action_open_session_manager` | user action | self | `survey` |  |  |
| `action_end_session` | user action | self | `survey_crm`, `survey` |  | The write is sudo'ed because a survey user can end a session even if it's not their own survey. |
| `get_start_url` | operation | self | `survey` |  |  |
| `get_start_short_url` | operation | self | `survey` |  | See controller method docstring for more details. |
| `get_print_url` | operation | self | `survey` |  |  |
| `_prepare_statistics` | preparation rule | self, user_input_lines | `survey` |  |  |
| `_prepare_challenge_category` | preparation rule | self | `survey`, `website_slides_survey` |  |  |
| `_create_certification_badge_trigger` | internal rule | self | `survey` |  |  |
| `_handle_certification_badges` | internal rule | self, vals | `survey` |  |  |
| `_generate_session_codes` | internal rule | self, code_count, excluded_codes | `survey` |  | Generate {code_count} session codes for surveys.  We try to generate 4 digits code first and see if we have {code_count} unique ones. Then we raise up to 5 digits if we need more, etc until up to 10 digits. (We generate an extra 20 codes per loop to try to mitigate back luck collisions). |
| `_get_supported_lang_codes` | preparation rule | self | `survey` |  |  |
| `get_formview_id` | operation | self, access_uid | `hr_recruitment_survey` |  |  |
| `_prepare_lead_qualification_template_values` | preparation rule | self | `survey_crm` | model |  |
| `_compute_generate_lead` | computation | self | `survey_crm` | depends: `survey_type`, `question_ids` |  |
| `_compute_lead_count` | computation | self | `survey_crm` | depends: `lead_ids` |  |
| `action_survey_see_leads` | user action | self | `survey_crm` |  | Shows the leads created from the current survey |
| `_compute_slide_channel_data` | computation | self | `website_slides_survey` | depends: `slide_ids.channel_id` |  |
| `_unlink_except_linked_to_course` | internal rule | self | `website_slides_survey` | ondelete |  |
| `action_survey_view_slide_channels` | user action | self | `website_slides_survey` |  | Redirect to the channels using the survey as a certification. Open in no-create as link between those two comes through a slide, hard to keep as default values. |

## Validation and error messages (16)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_check_scoring_after_page_availability` | ValidationError | Combining roaming and "Scoring with answers after each page" is not possible; please update the following surveys: - %(survey_names)s | `survey` |
| `_check_survey_responsible_access` | ValidationError | The access of the following surveys is restricted. Make sure their responsible still has access to it:  %(survey_names)s | `survey` |
| `_check_answer_creation` | UserError | Creating token for closed/archived surveys is not allowed. | `survey` |
| `_check_answer_creation` | UserError | Creating token for anybody else than employees is not allowed for internal surveys. | `survey` |
| `_check_answer_creation` | UserError | No attempts left. | `survey` |
| `_check_answer_creation` | UserError | Creating test token is not allowed for you. | `survey` |
| `_check_answer_creation` | UserError | Creating token for external people is not allowed for surveys requesting authentication. | `survey` |
| `_check_answer_creation` | UserError | Creating token for external people is not allowed for surveys requesting authentication. | `survey` |
| `check_validity` | UserError | You cannot send an invitation for a survey that has no questions. | `survey` |
| `check_validity` | UserError | A scored survey needs at least one question that gives points. Please check answers and their scores. | `survey` |
| `check_validity` | UserError | You cannot send invitations for closed surveys. | `survey` |
| `check_validity` | UserError | You cannot send an invitation for a "One page per section" survey if the survey has no sections. | `survey` |
| `check_validity` | UserError | You cannot send an invitation for a "One page per section" survey if the survey only contains empty sections. | `survey` |
| `action_start_session` | AccessError | Only survey users can manage sessions. | `survey` |
| `action_end_session` | AccessError | Only survey users can manage sessions. | `survey` |
| `_unlink_except_linked_to_course` | ValidationError | Uh-oh! You can’t delete surveys used as a Course Certification! Otherwise, students might think diplomas just grow on trees. The courses that need them are: %s | `website_slides_survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| all internal users | no | no | no | no | `survey` |
| `base.group_user` | no | no | no | no | `survey` |
| `group_survey_user` | yes | yes | yes | yes | `survey` |
| `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey survey: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey: recruitment interviewer: send surveys to applicants for which they are set as interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[('survey_type', '=', 'recruitment'),                 '\|', ('hr_job_ids.interviewer_ids', 'in', user.id),                      ('hr_job_ids.application_ids.interviewer_ids', 'in', user.id)                 ]` | 1 | 0 | 0 | 0 |
| Survey: manager: all | `[(4, ref('group_survey_manager'))]` | `[(1, '=', 1)]` | 1 | 1 | 1 | 1 |
| Survey: officer: unrestricted survey or in restricted users | `[(4, ref('group_survey_user'))]` | `[                 '\|', ('restrict_user_ids', 'in', user.id), ('restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 1 |
| Survey: slide channel officer on certification: read | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('certification', '=', True),             ('survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('restrict_user_ids', '=', False),('restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |

## Views (16)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment_survey.survey_survey_view_form` | xpath | `survey.survey_survey_view_form` |  |  |  | `hr_recruitment_survey` |
| `hr_recruitment_survey.survey_survey_view_kanban` | button | `survey.survey_survey_view_kanban` |  | `action_send_survey` |  | `hr_recruitment_survey` |
| `hr_skills_survey.survey_survey_view_form` | xpath | `survey.survey_survey_view_form` | `certification_validity_months` |  |  | `hr_skills_survey` |
| `survey.survey_survey_view_form` | form |  | `id`, `session_state`, `question_ids`, `session_available`, `answer_count`, `success_count`, `answer_done_count`, `background_image`, `allowed_survey_types`, `survey_type`, `title`, `user_id`, `active`, `has_conditional_questions`, `lang_ids`, `restrict_user_ids`, `question_and_page_ids`, `sequence`, `title`, `background_image`, `question_type`, `is_time_limited`, `time_limit`, `is_time_customized`, `constr_mandatory`, `is_page`, `questions_selection`, `survey_id`, `random_questions_count`, `triggering_question_ids`, `triggering_answer_ids`, `questions_layout`, `progression_mode`, `questions_selection`, `users_can_go_back`, `access_mode`, `users_login_required`, `is_attempts_limited`, `attempts_limit`, `is_time_limited`, `time_limit`, `scoring_type`, `scoring_success_min`, `certification`, `certification_report_layout`, `certification_mail_template_id`, `certification_give_badge`, `certification_badge_id`, `certification_badge_id_dummy`, `access_token`, `session_code`, `session_link`, `session_speed_rating`, `session_speed_rating_time_limit`, `description`, `description_done` | `Share`, `See results`, `Create Live Session`, `Open Session Manager`, `Close Live Session`, `Test`, `Reopen`, `Close`, `action_survey_user_input`, `action_survey_user_input_certified`, `action_survey_user_input_completed`, `copy`, `Preview` |  | `survey` |
| `survey.survey_survey_view_tree` | list |  | `active`, `certification`, `title`, `user_id`, `answer_duration_avg`, `answer_count`, `answer_done_count`, `success_count`, `success_ratio`, `answer_score_avg` | `certification` |  | `survey` |
| `survey.survey_survey_view_kanban` | kanban |  | `active`, `certification`, `create_date`, `scoring_type`, `session_state`, `session_available`, `color`, `title`, `user_id`, `question_count`, `user_id`, `answer_duration_avg`, `answer_count`, `answer_done_count`, `success_ratio`, `activity_ids` | `Share`, `Test`, `See results`, `Start Live Session`, `End Live Session` |  | `survey` |
| `survey.survey_survey_view_activity` | activity |  | `user_id`, `title` |  |  | `survey` |
| `survey.survey_survey_view_search` | search |  | `title`, `question_and_page_ids`, `user_id`, `restrict_user_ids` |  | `Is a Certification`, `Is not a Certification`, `Archived`, `My Activities`, `Late Activities`, `Today Activities`, `Upcoming Activities`, `Responsible`, `group_by_restrict_user_ids` | `survey` |
| `survey.survey_survey_view_graph` | graph |  | `color` |  |  | `survey` |
| `survey.survey_survey_view_pivot` | pivot |  | `color` |  |  | `survey` |
| `survey_crm.survey_survey_view_form` | xpath | `survey.survey_survey_view_form` | `lead_count` | `action_survey_see_leads` |  | `survey_crm` |
| `survey_crm.survey_survey_view_kanban` | xpath | `survey.survey_survey_view_kanban` | `lead_count` |  |  | `survey_crm` |
| `website_slides_survey.survey_survey_view_tree_slides` | field | `survey.survey_survey_view_tree` | `title` |  |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_search_slides` | xpath | `survey.survey_survey_view_search` |  |  |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_form` | xpath | `survey.survey_survey_view_form` | `slide_channel_count` | `action_survey_view_slide_channels` |  | `website_slides_survey` |
| `website_slides_survey.survey_survey_view_kanban` | xpath | `survey.survey_survey_view_kanban` | `slide_channel_count` |  |  | `website_slides_survey` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `hr_recruitment_survey.survey_survey_action_recruitment` | Interviews | kanban,list,activity,form | `[('survey_type', '=', 'recruitment')]` | `{                 'default_survey_type': 'recruitment',             }` |  | `hr_recruitment_survey` |
| `survey.action_survey_form` | Surveys | kanban,list,form,activity |  |  |  | `survey` |
| `website_slides_survey.survey_survey_action_slides` | Certifications | kanban,list,pivot,graph,form | `[('certification', '=', True)]` | `{'default_certification': True, 'default_scoring_type': 'scoring_with_answers'}` |  | `website_slides_survey` |

## Server actions

| Action | Name | State | Binding | Custom logic |
|---|---|---|---|---|
| `survey.action_survey_print` | Print Survey | code |  | yes |

Machine-readable definition: `../../../schemas/data/entities/survey.survey.json`; views: `../../../schemas/interfaces/views/survey.survey.json`.
