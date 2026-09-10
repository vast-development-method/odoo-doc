# Survey User Input (`survey.user_input`)

**Transport name:** `survey.user_input`  
**Storage name:** `survey_user_input`  
**Kind:** persistent entity (one table)  
**Defined by package:** `survey`  
**Extended by packages:** `hr_recruitment_survey`, `hr_skills_survey`, `survey_crm`, `website_slides_survey`

Description: Survey User Input

## Identity and behavior

- Mixins (classical inheritance): `mail.thread`, `mail.activity.mixin`
- Default ordering: `create_date desc`
- Display name field: `survey_id`
- Audit fields: `id`, `create_date`, `create_uid`, `write_date`, `write_uid`

## Fields (31)

| Field (storage name) | Full name | Type | Target | Meaning and rules |
|---|---|---|---|---|
| `survey_id` | Survey | many to one | `survey.survey` | required; read only; indexed; on delete of the target: cascade |
| `scoring_type` | Scoring | selection |  | related through path `survey_id.scoring_type` |
| `start_datetime` | Start date and time | date and time |  | read only |
| `end_datetime` | End date and time | date and time |  | read only |
| `deadline` | Deadline | date and time |  | Help: Datetime until customer can open the survey and submit answers |
| `lang_id` | Language | many to one | `res.lang` |  |
| `state` | Status | selection |  | read only; default `new` |
| `test_entry` | Test Entry | boolean |  | read only |
| `last_displayed_page_id` | Last displayed question/page | many to one | `survey.question` |  |
| `is_attempts_limited` | Limited number of attempts | boolean |  | related through path `survey_id.is_attempts_limited` |
| `attempts_limit` | Number of attempts | integer |  | related through path `survey_id.attempts_limit` |
| `attempts_count` | Attempts Count | integer |  | computed by rule `_compute_attempts_info` (not stored) |
| `attempts_number` | Attempt n° | integer |  | computed by rule `_compute_attempts_info` (not stored) |
| `survey_time_limit_reached` | Survey Time Limit Reached | boolean |  | computed by rule `_compute_survey_time_limit_reached` (not stored) |
| `access_token` | Identification token | single line text |  | required; read only; default computed dynamically (lambda self: str(uuid.uuid4())); not copied on duplication |
| `invite_token` | Invite token | single line text |  | read only; not copied on duplication |
| `partner_id` | Contact | many to one | `res.partner` | read only; indexed (btree_not_null) |
| `email` | Email | single line text |  | read only |
| `nickname` | Nickname | single line text |  | Help: Attendee nickname, mainly used to identify them in the survey session leaderboard. |
| `user_input_line_ids` | Answers | one to many | `survey.user_input.line` | inverse field `user_input_id` |
| `predefined_question_ids` | Predefined Questions | many to many | `survey.question` | read only |
| `scoring_percentage` | Score (%) | float |  | computed by rule `_compute_scoring_values` and stored |
| `scoring_total` | Total Score | float |  | computed by rule `_compute_scoring_values` and stored; precision `[10, 2]` |
| `scoring_success` | Quiz Passed | boolean |  | computed by rule `_compute_scoring_success` and stored |
| `survey_first_submitted` | Survey First Submitted | boolean |  |  |
| `is_session_answer` | Is in a Session | boolean |  | Help: Is that user input part of a survey session or not. |
| `question_time_limit_reached` | Question Time Limit Reached | boolean |  | computed by rule `_compute_question_time_limit_reached` (not stored) |
| `applicant_id` | Applicant | many to one | `hr.applicant` | indexed (btree_not_null) |
| `lead_id` | Lead | many to one | `crm.lead` | on delete of the target: set null |
| `slide_id` | Related course slide | many to one | `slide.slide` | Help: The related course slide when there is no membership information |
| `slide_partner_id` | Subscriber information | many to one | `slide.slide.partner` | indexed (btree_not_null); Help: Slide membership information for the logged in user |

## Selection values

### `state` (Status)

| Value | Label |
|---|---|
| `new` | New |
| `in_progress` | In Progress |
| `done` | Completed |

## State fields

State machine fields of this entity: `state`. Transitions are specified in the domain documents.

## Database constraints and indexes (1)

| Name | Kind | Definition | Message | Package |
|---|---|---|---|---|
| `_unique_token` | Constraint | `UNIQUE (access_token)` | An access token must be unique! | `survey` |

## Operations (40)

| Operation | Kind | Arguments | Defined in packages | Triggers and dependencies | Documentation |
|---|---|---|---|---|---|
| `_compute_scoring_values` | computation | self | `survey` | depends: `user_input_line_ids.answer_score`, `user_input_line_ids.question_id`, `predefined_question_ids.answer_score` |  |
| `_compute_scoring_success` | computation | self | `survey` | depends: `scoring_percentage`, `survey_id` |  |
| `_compute_survey_time_limit_reached` | computation | self | `survey` | depends: `start_datetime`, `survey_id.is_time_limited`, `survey_id.time_limit` | Checks that the user_input is not exceeding the survey's time limit. |
| `_compute_question_time_limit_reached` | computation | self | `survey` | depends: `survey_id.session_question_id.time_limit`, `survey_id.session_question_id.is_time_limited`, `survey_id.session_question_start_time` | Checks that the user_input is not exceeding the question's time limit. Only used in the context of survey sessions. |
| `_compute_attempts_info` | computation | self | `survey` | depends: `state`, `test_entry`, `survey_id.is_attempts_limited`, `partner_id`, `email`, `invite_token` |  |
| `create` | lifecycle override | self, vals_list | `survey`, `website_slides_survey` | model_create_multi |  |
| `action_resend` | user action | self | `survey` |  |  |
| `action_print_answers` | user action | self | `survey` |  | Open the website page with the survey form |
| `action_redirect_to_attempts` | user action | self | `survey` |  |  |
| `_generate_invite_token` | internal rule | self | `survey` | model |  |
| `_mark_in_progress` | internal rule | self | `survey` |  | marks the state as 'in_progress' and updates the start_datetime accordingly. |
| `_mark_done` | internal rule | self | `hr_recruitment_survey`, `hr_skills_survey`, `survey_crm`, `survey` |  | This method will: 1. mark the state as 'done' 2. send the certification email with attached document if - The survey is a certification - It has a certification_mail_template_id set - The user succeeded the test 3. Notify survey subtype subscribers of the newly completed input Will also run challenge Cron to give the certification badge if any. |
| `get_start_url` | operation | self | `survey` |  |  |
| `get_print_url` | operation | self | `survey` |  |  |
| `_save_lines` | internal rule | self, question, answer, comment, overwrite_existing | `survey` |  | Save answers to questions, depending on question type.  :param bool overwrite_existing: if an answer already exists for question and user_input_id it will be overwritten (or deleted for 'choice' questions) in order to maintain data consistency. :raises UserError: if line exists and overwrite_existing is False |
| `_save_line_simple_answer` | internal rule | self, question, old_answers, answer | `survey` |  |  |
| `_save_line_choice` | internal rule | self, question, old_answers, answers, comment | `survey` |  |  |
| `_save_line_matrix` | internal rule | self, question, old_answers, answers, comment | `survey` |  |  |
| `_get_line_answer_values` | preparation rule | self, question, answer, answer_type | `survey` |  |  |
| `_get_line_comment_values` | preparation rule | self, question, comment | `survey` |  |  |
| `_prepare_statistics` | preparation rule | self | `survey` |  | Prepares survey.user_input's statistics to display various charts on the frontend. Returns a structure containing answers statistics "by section" and "totals" for every input in self.  e.g returned structure: {     survey.user_input(1,): {         'by_section': {             'Uncategorized': {                 'question_count': 2,                 'correct': 2,                 'partial': 0,                 'incorrect': 0,                 'skipped': 0,             },             'Mathematics': {                 'question_count': 3,                 'correct': 1,                 'partial': 1,       |
| `_multiple_choice_question_answer_result` | internal rule | self, user_input_lines, question_correct_suggested_answers | `survey` |  |  |
| `_simple_choice_question_answer_result` | internal rule | self, user_input_line, question_correct_suggested_answers, question_incorrect_scored_answers | `survey` |  |  |
| `_simple_question_answer_result` | internal rule | self, user_input_line | `survey` |  |  |
| `_get_conditional_values` | preparation rule | self | `survey` |  | For survey containing conditional questions, we need a triggered_questions_by_answer map that contains        {key: answer, value: the question that the answer triggers, if selected}, The idea is to be able to verify, on every answer check, if this answer is triggering the display of another question. If answer is not in the conditional map:    - nothing happens. If the answer is in the conditional map:    - If we are in ONE PAGE survey : (handled at CLIENT side)        -> display immediately the depending question    - If we are in PAGE PER SECTION : (handled at CLIENT side)        - If relat |
| `_get_selected_suggested_answers` | preparation rule | self | `survey` |  | For now, only simple and multiple choices question type are handled by the conditional questions feature. Mapping all the suggested answers selected by the user will also include answers from matrix question type, Those ones won't be used. Maybe someday, conditional questions feature will be extended to work with matrix question. :return: all the suggested answer selected by the user. |
| `_clear_inactive_conditional_answers` | internal rule | self | `survey` |  | Clean eventual answers on conditional questions that should not have been displayed to user. This method is used mainly for page per question survey, a similar method does the same treatment at client side for the other survey layouts. E.g.: if depending answer was uncheck after answering conditional question, we need to clear answers       of that conditional question, for two reasons:       - ensure correct scoring       - if the selected answer triggers another question later in the survey, if the answer is not cleared,         a question that should not be displayed to the user will be.  T |
| `_get_inactive_conditional_questions` | preparation rule | self | `survey` |  |  |
| `_get_print_questions` | preparation rule | self | `survey` |  | Get the questions to display : the ones that should have been answered = active questions     In case of session, active questions are based on most voted answers :return: active survey.question browse records |
| `_get_next_skipped_page_or_question` | preparation rule | self | `survey` |  | Get next skipped question or page in case the option 'can_go_back' is set on the survey It loops to the first skipped question or page if 'last_displayed_page_id' is the last skipped question or page. |
| `_get_skipped_questions` | preparation rule | self | `survey` |  |  |
| `_is_last_skipped_page_or_question` | internal rule | self, page_or_question | `survey` |  | In case of a submitted survey tells if the question or page is the last skipped page or question.  This is used to :  - Display a Submit button if the actual question is the last skipped question. - Avoid displaying a Submit button on the last survey question if there are   still skipped questions before. - Avoid displaying the next page if submitting the latest skipped question.  :param page_or_question: page if survey's layout is page_per_section, question if page_per_question. |
| `_notify_new_participation_subscribers` | internal rule | self | `survey` |  |  |
| `_create_leads_from_generative_answers` | internal rule | self | `survey_crm` |  | This method filters the user inputs in self to only keep those that create a lead. After a step of data preparation, create related leads in batch. |
| `_prepare_common_survey_lead_values` | preparation rule | self, survey | `survey_crm` |  |  |
| `_prepare_user_input_lead_values` | preparation rule | self | `survey_crm` |  | This method prepares the user values dictionary for creating the lead |
| `_prepare_lead_values_from_user_input_lines` | preparation rule | self | `survey_crm` |  | This prepares dict-formatted lead values from user input lines. It formats lead description and get user's nickname and his email, if they're provided. To write the description (notes in CRM) in HTML format, there are 5 cases:     - Suggested answers question: <li>Question — Answer 1, Answer 3</li>     - Matrix question:  <li>                             Question                             <br/>&emsp;Line label 1 — Answer 1                             <br/>&emsp;Line label 1 — Answer 3                             <br/>&emsp;Line label 4 — Answer 2                         </li>     - Long text |
| `action_redirect_lead` | user action | self | `survey_crm` |  | Shows the lead associated, created from inputs |
| `write` | lifecycle override | self, vals | `website_slides_survey` |  |  |
| `_check_for_failed_attempt` | validation | self | `website_slides_survey` |  | If the user fails their last attempt at a course certification, we remove them from the members of the course (and they have to enroll again). They receive an email in the process notifying them of their failure and suggesting they enroll to the course again.  The purpose is to have a 'certification flow' where the user can re-purchase the certification when they have failed it. |

## Validation and error messages (1)

| Raised by operation | Kind | Message | Package |
|---|---|---|---|
| `_save_lines` | UserError | This answer cannot be overwritten. | `survey` |

## Access rights

| Group | Create | Read | Update | Delete | Package |
|---|---|---|---|---|---|
| `hr_recruitment.group_hr_recruitment_manager` | yes | yes | yes | yes | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_user` | no | yes | no | no | `hr_recruitment_survey` |
| `hr_recruitment.group_hr_recruitment_interviewer` | no | yes | no | no | `hr_recruitment_survey` |
| all internal users | no | no | no | no | `survey` |
| `base.group_user` | no | no | no | no | `survey` |
| `group_survey_user` | yes | yes | yes | yes | `survey` |
| `group_survey_manager` | yes | yes | yes | yes | `survey` |
| `website_slides.group_website_slides_officer` | no | yes | no | no | `website_slides_survey` |

## Record rules

| Rule | Groups | Domain | Read | Update | Create | Delete |
|---|---|---|---|---|---|---|
| Survey user input: recruitment manager: all recruitment | `[(4, ref('hr_recruitment.group_hr_recruitment_manager'))]` | `[('survey_id.survey_type', '=', 'recruitment')]` | 1 | 1 | 1 | 1 |
| Survey user input: recruitment officer: unrestricted or in restricted users | `[(4, ref('hr_recruitment.group_hr_recruitment_user'))]` | `[                 '&', ('survey_id.survey_type', '=', 'recruitment'),                 '\|',  ('survey_id.restrict_user_ids', 'in', user.id),                         ('survey_id.restrict_user_ids', '=', False)]` | 1 | 0 | 0 | 0 |
| Survey user input: recruitment interviewer: read survey answers for which they are set as interviewer | `[(4, ref('hr_recruitment.group_hr_recruitment_interviewer'))]` | `[                 '\|',                     ('applicant_id.interviewer_ids', 'in', user.id),                     ('applicant_id.job_id.interviewer_ids', 'in', user.id),                 ]` | 1 | 0 | 0 | 0 |
| Survey user input: manager: all non specialized surveys | `[(4, ref('group_survey_manager'))]` | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | 1 | 1 | 1 | 1 |
| Survey user input: officer: unrestricted survey or in restricted users | `[(4, ref('group_survey_user'))]` | `[                 '&', ('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey')),                 '\|', ('survey_id.restrict_user_ids', 'in', user.id),                      ('survey_id.restrict_user_ids', '=', False)]` | 1 | 1 | 1 | 1 |
| Survey user input: slide channel officer on certification: read | `[(4, ref('website_slides.group_website_slides_officer'))]` | `[('survey_id.certification', '=', True),             ('survey_id.survey_type', 'in', ('survey', 'live_session', 'assessment', 'custom')),             '\|', ('survey_id.restrict_user_ids', '=', False), ('survey_id.restrict_user_ids', 'in', user.id)]` | 1 | 0 | 0 | 0 |

## Views (5)

| View | Type | Inherits | Fields shown | Buttons | Filters and groupings | Package |
|---|---|---|---|---|---|---|
| `survey.survey_user_input_view_search` | search |  | `email`, `partner_id`, `survey_id` |  | `New`, `In Progress`, `Completed`, `Quiz passed`, `Tests Only`, `Exclude Tests`, `Survey`, `Email`, `Partner`, `Language` | `survey` |
| `survey.survey_user_input_view_form` | form |  | `state`, `attempts_count`, `attempts_count`, `test_entry`, `survey_id`, `create_date`, `deadline`, `is_attempts_limited`, `attempts_number`, `attempts_limit`, `test_entry`, `scoring_type`, `scoring_success`, `scoring_percentage`, `partner_id`, `email`, `access_token`, `user_input_line_ids`, `question_sequence`, `create_date`, `page_id`, `question_id`, `answer_type`, `skipped`, `display_name`, `answer_is_correct`, `answer_score` | `Resend Invitation`, `Print`, `action_redirect_to_attempts` |  | `survey` |
| `survey.survey_user_input_view_tree` | list |  | `create_date`, `survey_id`, `nickname`, `partner_id`, `email`, `attempts_number`, `deadline`, `test_entry`, `scoring_success`, `scoring_percentage`, `state` |  |  | `survey` |
| `survey.survey_user_input_viuew_kanban` | kanban |  | `survey_id`, `create_date`, `state` |  |  | `survey` |
| `survey_crm.survey_user_input_view_form` | xpath | `survey.survey_user_input_view_form` |  | `action_redirect_lead` |  | `survey_crm` |

## Window actions

| Action | Name | View modes | Domain | Context | Target | Package |
|---|---|---|---|---|---|---|
| `survey.action_survey_user_input` | Participants | list,kanban,form | `[('survey_id.survey_type', 'in', ('assessment', 'custom', 'live_session', 'survey'))]` | `{'search_default_group_by_survey': True}` |  | `survey` |
| `survey.res_partner_action_certifications` | Certifications Succeeded | list,form |  | `{'search_default_scoring_success': 1}` |  | `survey` |

## Printable reports

| Report | Name | Type | Template | File name rule | Attachment rule |
|---|---|---|---|---|---|
| `survey.certification_report` | Certifications | qweb-pdf | `survey.certification_report_view` | `'Certification - %s' % (object.survey_id.display_name)` | `'certification.pdf'` |

## Email templates

| Template | Name | Subject |
|---|---|---|
| `hr_recruitment_survey.mail_template_applicant_interview_invite` | Applicant: Interview | Participate to {{ object.survey_id.display_name }} interview |
| `survey.mail_template_user_input_invite` | Survey: Invite | Participate to {{ object.survey_id.display_name }} survey |
| `survey.mail_template_certification` | Survey: Certification Success | Certification: {{ object.survey_id.display_name }} |
| `website_slides_survey.mail_template_user_input_certification_failed` | Survey: Certification Failure | You have failed the course: {{ object.slide_partner_id.channel_id.name }} |

Machine-readable definition: `../../../schemas/data/entities/survey.user_input.json`; views: `../../../schemas/interfaces/views/survey.user_input.json`.
