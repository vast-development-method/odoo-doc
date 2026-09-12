# Learning, Questionnaires and Recognition — Interfaces

Menus, screens, named operations, request endpoints, printable documents, notifications, external
integrations, and import and export. Endpoint paths, operation names and action identifiers are
reproduced exactly because they are contractual; everything else is described in words.

---

## 1. Menus

### 1.1 Questionnaires

| Menu | Label | Parent | Opens | Sequence | Restricted to |
|---|---|---|---|---|---|
| `survey.menu_surveys` | Surveys | top level | — | 130 | `survey.group_survey_user` |
| `survey.menu_survey_form` | Surveys | Surveys | the questionnaire list | 1 | — |
| `survey.menu_survey_type_form1` | Participants | Surveys | the participation list | 1 | — |
| `survey.survey_menu_questions` | Questions & Answers | Surveys | — | 90 | — |
| `survey.menu_survey_question_form1` | Questions | Questions & Answers | the question list, filtered to questions | 2 | — |
| `survey.menu_survey_label_form1` | Suggested Values | Questions & Answers | the answer option list | 3 | — |
| `survey.menu_survey_response_line_form` | Detailed Answers | Questions & Answers | the answer row list | 4 | — |

### 1.2 Learning platform

| Menu | Label | Parent | Opens | Sequence | Restricted to |
|---|---|---|---|---|---|
| `website_slides.website_slides_menu_root` | eLearning | top level | the course overview | 100 | `website_slides.group_website_slides_officer` |
| `website_slides.website_slides_menu_courses` | Courses | eLearning | — | 1 | — |
| `website_slides.website_slides_menu_courses_courses` | Courses | Courses | the course overview | 1 | — |
| `website_slides.website_slides_menu_courses_content` | Contents | Courses | the content list, filtered to non-sections | 2 | — |
| `website_slides_survey.website_slides_menu_courses_certification` | Certifications | Courses | the certification questionnaires | 3 | — |
| `website_slides_forum.website_slides_menu_forum` | Forum | eLearning | — | 2 | — |
| `website_slides_forum.website_slides_menu_forum_forum` | Forums | Forum | the forums attached to a course | 1 | — |
| `website_slides_forum.website_slides_menu_forum_post` | Posts | Forum | the posts of those forums | 2 | — |
| `website_slides.website_slides_menu_report` | Reporting | eLearning | — | 9 | `website_slides.group_website_slides_manager` |
| `website_slides.website_slides_menu_report_courses` | Courses | Reporting | the course analysis | 1 | — |
| `website_slides.website_slides_menu_report_contents` | Contents | Reporting | the content analysis | 2 | — |
| `website_sale_slides.website_slides_menu_report_revenues` | Revenues | Reporting | the revenue analysis | 3 | — |
| `website_slides.website_slides_menu_report_attendees` | Attendees | Reporting | the attendee analysis | 5 | — |
| `website_slides.website_slides_menu_report_reviews` | Reviews | Reporting | the course reviews | 10 | — |
| `website_slides.website_slides_menu_report_quizzes` | Quizzes | Reporting | the quiz analysis | 15 | — |
| `website_slides.website_slides_menu_configuration` | Configuration | eLearning | — | 99 | — |
| `website_slides.website_slides_menu_config_settings` | Settings | Configuration | the configuration screen | 1 | `base.group_system` |
| `website_slides.website_slides_menu_config_course_groups` | Course Groups | Configuration | the course tag families | 2 | — |
| `website_slides.website_slides_menu_config_content_tags` | Content Tags | Configuration | the content tags | 3 | — |
| `website_slides.menu_slide_channel_pages` | Courses | the site content menu | the course pages | 50 | — |

### 1.3 Forum

| Menu | Label | Parent | Opens | Sequence | Restricted to |
|---|---|---|---|---|---|
| `website_forum.menu_website_forum_global` | Forum | the site configuration menu | — | 170 | `website.group_website_designer` |
| `website_forum.menu_forum_global` | Forums | Forum | the forum list | 10 | — |
| `website_forum.menu_forum_rank_global` | Ranks | Forum | the rank list | 20 | — |
| `website_forum.menu_forum_tag_global` | Tags | Forum | the forum tag list | 30 | — |
| `website_forum.menu_forum_badges` | Badges | Forum | the badge list | 40 | — |
| `website_forum.menu_forum_post_reasons` | Close Reasons | Forum | the closing reason list | 50 | — |
| `website_forum.menu_forum_post_pages` | Forum Posts | the site content menu | the post list | 80 | — |

### 1.4 Recognition

| Menu | Label | Parent | Opens | Sequence | Restricted to |
|---|---|---|---|---|---|
| `gamification.gamification_menu` | Gamification Tools | the administration menu | — | — | the developer group |
| `gamification.gamification_challenge_menu` | — | Gamification Tools | the challenge list | 0 | — |
| `gamification.gamification_goal_menu` | — | Gamification Tools | the goal list | 10 | — |
| `gamification.gamification_definition_menu` | — | Gamification Tools | the goal definition list | 20 | — |
| `gamification.gamification_badge_menu` | — | Gamification Tools | the badge list | 30 | — |
| `gamification.gamification_karma_ranks_menu` | — | Gamification Tools | the rank list | 40 | — |
| `gamification.gamification_karma_tracking_menu` | — | Gamification Tools | the movement list | 50 | — |
| `hr_gamification.menu_hr_gamification` | Challenges | the human-resources configuration menu | — | 100 | — |
| `hr_gamification.gamification_badge_menu_hr` | — | Challenges | the badge list | — | — |
| `hr_gamification.gamification_challenge_menu_hr` | — | Challenges | the challenges of the human-resources category | — | `hr.group_hr_user` |
| `hr_gamification.gamification_goal_menu_hr` | — | Challenges | the goals of those challenges | — | `hr.group_hr_user` |

## 2. Screens

| Screen | Entity | Views offered | What it is for |
|---|---|---|---|
| Surveys | Survey | card, list, form, activity | Authoring and monitoring. The card shows the purpose, the question count, the attempts, the success ratio and the average duration; the form holds the question list, the settings panels and the buttons of section 3. |
| Participants | Survey Participation | list, card, form | One row per attempt, with the contact, the address, the attempt number, the deadline, the test marker, the pass verdict, the percentage and the state. |
| Questions | Survey Question | list, form | Every question across questionnaires, grouped by section. |
| Suggested Values | Survey Answer Option | list, form | Every answer option, grouped by question. |
| Detailed Answers | Survey Participation Answer | list, form | Every stored answer row, grouped by questionnaire and by participation. |
| Certifications | Survey | card, list, pivot, graph, form | The questionnaires whose certification marker is true. |
| Courses | Course | card, list, form | Authoring. The card shows the picture, the tags, the visits, the content count, the duration, the review count and the four attendee counters. |
| Course analysis | Course | list, graph, pivot, form | The visits, the rating average, the duration, the attendee count and the completed count, filtered to published courses by default. |
| Contents | Course Content | card, list, form | Every content item, filtered to non-sections, with the reader's own publications shown first. |
| Content analysis | Course Content | graph, list, form, pivot | The visits, the question count and the duration per content. |
| Attendees | Course Enrolment | list, card, graph, pivot | One row per enrolment with the status, the completion, the next content and the course settings. |
| Reviews | Rating | the rating views | The published reviews of the courses. |
| Quizzes | Quiz Question | list, graph, pivot, form | The attempt count, the average attempts and the completed count per question. |
| Course Groups and Content Tags | Course Tag Family, Content Tag | list, form | Configuration. |
| Forums | Forum | list, form | The whole reputation table, the statistics and the two buttons of section 3. |
| Forum Posts | Forum Post | list, card, graph | Moderation from the back office. |
| Forum Tags and Close Reasons | Forum Tag, Post Closing Reason | list, form | Configuration. |
| Challenges | Challenge | card, list, form | The lines, the population, the rewards and the report schedule. |
| Goals | Goal | list, form, card | The measured values, grouped by user and by definition. |
| Goal Definitions | Goal Definition | list, form | Configuration of what is measured. |
| Badges | Badge | card, list, form | The granting rule, the counters and the grant button. |
| Ranks | Karma Rank | list, form | The minimum balance and the holder count. |
| Karma Tracking | Karma Movement | list, form | The whole ledger, filterable by user, by instant and by source kind. |

## 3. Named operations offered as buttons

### 3.1 On a questionnaire

| Operation | Label | Effect |
|---|---|---|
| `action_send_survey` | Share | Opens the invitation composer after running the five checks of [`business-rules.md`](business-rules.md#lsg-027). |
| `action_test_survey` | Test | Creates a test participation and opens the public form on it. |
| `action_start_survey` | — | Opens the public form on a given participation. |
| `action_print_survey` | Print | Opens the printable view. |
| `action_result_survey` | See results | Opens the result page. |
| `action_survey_user_input` | — | Opens the participation list of this questionnaire. |
| `action_survey_user_input_completed` | — | The same, filtered to the completed participations. |
| `action_survey_user_input_certified` | — | The same, filtered to the passing participations. |
| `action_start_session` | Create Live Session | Opens a session; see [`workflows.md`](workflows.md#workflow-5). |
| `action_open_session_manager` | Open Session Manager | Opens the host screen. |
| `action_end_session` | Close Live Session | Closes the session. |
| `action_survey_preview_certification_template` | Preview | Renders the certificate on a temporary test participation. |
| `action_load_survey_template_sample` | — | Loads one of the shipped samples into the record. |
| `action_load_sample_custom` | — | Loads the custom sample. |
| `action_show_sample` | — | Opens the sample chooser. |
| `check_validity` | — | The five pre-invitation checks, callable on its own. |
| `get_start_url`, `get_start_short_url`, `get_print_url` | — | Return the public addresses of the questionnaire. |
| `action_survey_see_leads` | — | Added by the lead-generation package; opens the leads created from this questionnaire. |
| `action_survey_view_slide_channels` | — | Added by the course-certification package; opens the courses that use this questionnaire, with creation disabled because the link runs through a content item. |

### 3.2 On a participation

| Operation | Label | Effect |
|---|---|---|
| `action_resend` | Resend Invitation | Opens the invitation composer pre-filled with this participation's recipient. |
| `action_print_answers` | Print | Opens the printable view of this participation. |
| `action_redirect_to_attempts` | — | Opens the other attempts of the same pool. |
| `action_redirect_lead` | — | Added by the lead-generation package; opens the lead created from this participation. |

### 3.3 On a course

| Operation | Label | Effect |
|---|---|---|
| `action_channel_enroll` | Add Attendees | Opens the invitation composer in enrolment mode. |
| `action_channel_invite` | Invite | Opens the same composer in invitation mode. |
| `action_view_slides` | — | Opens the contents of the course. |
| `action_view_ratings` | — | Opens the reviews of the course. |
| `action_redirect_to_members` | — | Opens the attendees, optionally filtered by status. |
| `action_redirect_to_engaged_members` | — | The attendees whose status is `joined` or `ongoing`. |
| `action_redirect_to_completed_members` | — | The attendees whose status is `completed`. |
| `action_redirect_to_invited_members` | — | The attendees whose status is `invited`. |
| `action_request_access` | — | Requests access on behalf of the reader. |
| `action_grant_access`, `action_refuse_access` | — | Grant or refuse a pending request. |
| `action_view_sales` | — | Added by the course-selling package; opens the sales of the linked product. |
| `action_redirect_to_forum` | — | Added by the course-forum package; opens the attached forum. |
| `action_redirect_to_certified_members` | — | Added by the course-certification package; opens the certified attendees. |
| `action_mass_mailing_attendees` | Contact Attendees | Added by the attendee-mailing package; opens a new mailing named `Mass Mail Course Members` addressed to the contacts enrolled in the selected courses. |

### 3.4 On a content item

| Operation | Label | Effect |
|---|---|---|
| `action_like`, `action_dislike` | — | Write the vote on the reader's progress record. |
| `action_set_viewed` | — | Creates or refreshes the reader's progress record, optionally incrementing the quiz attempt count. |
| `action_mark_completed`, `action_mark_uncompleted` | — | Write the completion marker, with the guards of machine 7 of [`state-machines.md`](state-machines.md#machine-7). |
| `action_view_embeds` | — | Opens the embed counters of this content. |

### 3.5 On a forum, a post, a challenge, a goal and a badge

| Operation | Label | Effect |
|---|---|---|
| `go_to_website` | — | Opens the public page of the forum or of the post. |
| `validate`, `close`, `reopen`, `vote`, `convert_answer_to_comment`, `convert_comment_to_answer`, `unlink_comment`, `mark_as_offensive_batch` | — | The forum operations described in [`workflows.md`](workflows.md#workflow-16) and [`workflows.md`](workflows.md#workflow-18). |
| `action_start` | Start Challenge | Writes the challenge state to `inprogress`. |
| `action_check` | Refresh Challenge | Deletes the goals still in progress and runs the whole update again. |
| `action_report_progress` | Send Report | Sends the report at once without changing the report schedule. |
| `action_view_users` | — | Opens the participants of the challenge. |
| `accept_challenge`, `discard_challenge` | — | A suggested participant joins or refuses. |
| `action_start`, `action_reach`, `action_fail`, `action_cancel` on a goal | Start goal, Goal Reached, Goal Failed, Reset Completion | Write the goal state; see machine 5 of [`state-machines.md`](state-machines.md#machine-5). |
| `get_action` on a goal | — | Returns the action that updates the goal: the definition's own window action when one is set, otherwise the update dialogue for a manually kept goal, otherwise nothing. |
| `check_granting` on a badge | — | Runs the four granting checks. |
| `action_grant_badge` | Grant Badge | Creates the grant. |
| `get_granted_employees` | — | Added by the human-resources package; opens the employees holding the badge. |
| `action_karma_report` on a user | Karma Updates | Opens the movement ledger filtered to that user. |

## 4. Request endpoints

Every path is reproduced exactly. The access column says who may reach it: *anonymous* means no
account is needed, *signed in* means an account is required.

### 4.1 Questionnaires

| Path | Access | Kind | Purpose |
|---|---|---|---|
| `/survey/start/<survey_token>` | anonymous | page | Landing page: creates or resumes a participation and shows the start screen. |
| `/survey/<survey_token>` and `/survey/<survey_token>/<answer_token>` | anonymous | page | The answering screen. |
| `/survey/test/<survey_token>` | signed in | page | Creates a test participation and opens it. |
| `/survey/retry/<survey_token>/<answer_token>` | anonymous | page | Creates a new attempt in the same pool. |
| `/survey/begin/<survey_token>/<answer_token>` | anonymous | operation | Marks the participation in progress and returns the first screen. |
| `/survey/next_question/<survey_token>/<answer_token>` | anonymous | operation | Returns the next screen. |
| `/survey/submit/<survey_token>/<answer_token>` | anonymous | operation | Validates and stores the answers of a screen. |
| `/survey/print/<survey_token>` | anonymous | page | The printable view. |
| `/survey/results/<survey>` | signed in | page | The result page with its filters. |
| `/survey/<survey_token>/get_background_image` | anonymous | file | The questionnaire background. |
| `/survey/<survey_token>/<section_id>/get_background_image` | anonymous | file | A section background. |
| `/survey/get_question_image/<survey_token>/<answer_token>/<question_id>/<suggested_answer_id>` | anonymous | file | The picture of an answer option. |
| `/survey/<survey>/certification_preview` | signed in | page | Renders the certificate on a temporary test participation. |
| `/survey/<survey>/get_certification_preview` | signed in | file | The same document, fetched directly. |
| `/survey/<survey_id>/get_certification` | signed in | file | The certificate of a certification the reader passed. |
| `/survey/session/manage/<survey_token>` | signed in | page | The host screen of a live session. |
| `/survey/session/next_question/<survey_token>` | signed in | operation | Advances the session. |
| `/survey/session/results/<survey_token>` | signed in | operation | Returns the statistics of the current question. |
| `/survey/session/leaderboard/<survey_token>` | signed in | operation | Returns the leaderboard. |
| `/s` | anonymous | page | The code page where an attendee types a session code. |
| `/s/<session_code>` | anonymous | page | Resolves a code and redirects to the start address. |
| `/survey/check_session_code/<session_code>` | anonymous | operation | Checks a code live and returns one of the statuses of [`business-rules.md`](business-rules.md#lsg-049). |

### 4.2 Learning platform

| Path | Access | Kind | Purpose |
|---|---|---|---|
| `/slides` and its paged and tag-filtered forms | anonymous | page | The course list. |
| `/slides/all` and `/slides/all/tag/<slug_tags>` | anonymous | page | The whole catalogue, optionally filtered by tags. |
| `/slides/<channel>` and its paged, tag-filtered and section-filtered forms | anonymous | page | The course page. |
| `/slides/<channel_id>/invite` | anonymous | page | The personal invitation link; the six outcomes are in [`workflows.md`](workflows.md#workflow-9). |
| `/slides/<channel_id>/identify` | anonymous | page | The sign-in button of the invitation banner; refuses a signed-in visitor with `identify_fail`. |
| `/slides/channel/join` | anonymous | operation | Joins an open-enrolment course. |
| `/slides/channel/leave` | signed in | operation | Leaves a course. |
| `/slides/channel/subscribe`, `/slides/channel/unsubscribe` | signed in | operation | Follow and unfollow the course thread. |
| `/slides/channel/send_share_email` | signed in | operation | Shares the whole course. |
| `/slides/channel/tag/search_read`, `/slides/channel/tag/group/search_read`, `/slides/channel/tag/add`, `/slide_channel_tag/add` | signed in | operation | The course tag picker. |
| `/slides/slide/<slide>` | anonymous | page | The content page. |
| `/slides/slide/<slide_id>/share` | anonymous | page | The shared view of a content item, reachable with the course access token. |
| `/slides/slide/<slide>/pdf_content` | anonymous | file | The document payload. |
| `/slides/slide/<slide_id>/get_image` | anonymous | file | The illustration. |
| `/slides/slide/get_html_content` | anonymous | operation | The authored body of an article. |
| `/slides/slide/<slide>/set_completed`, `/slides/slide/set_completed` | signed in, anonymous | page, operation | Mark completed. |
| `/slides/slide/<slide>/set_uncompleted`, `/slides/slide/set_uncompleted` | signed in, anonymous | page, operation | Mark uncompleted. |
| `/slides/slide/like` | anonymous | operation | Like or dislike. |
| `/slides/slide/archive`, `/slides/slide/toggle_is_preview` | signed in | operation | Publisher actions from the public page. |
| `/slides/slide/send_share_email` | signed in | operation | Shares one content item. |
| `/slides/slide/quiz/get`, `/slides/slide/quiz/submit`, `/slides/slide/quiz/reset`, `/slides/slide/quiz/save_to_session`, `/slides/slide/quiz/question_add_or_update` | anonymous or signed in | operation | The quiz panel. |
| `/slides/category/search_read`, `/slides/category/add`, `/slides/tag/search_read`, `/slides/add_slide`, `/slides/prepare_preview` | signed in | operation | The content editor. |
| `/slides/embed/<slide_id>` | anonymous | page | The embedded player for a page of this platform. |
| `/slides/embed_external/<slide_id>` | anonymous | page | The embedded player for a third-party site; loading it increments the embed counter. |
| `/slides_survey/slide/get_certification_url` | signed in | page | Added by the course-certification package; produces the address of the attempt, following [`workflows.md`](workflows.md#workflow-12). |
| `/slides_survey/certification/search_read` | signed in | operation | The certification picker of the content editor. |

### 4.3 Forum

| Path | Access | Kind | Purpose |
|---|---|---|---|
| `/forum` | anonymous | page | The forum list. |
| `/forum/all` and `/forum/<forum>` with their paged and tag-filtered forms | anonymous | page | The question list. |
| `/forum/<forum>/<question>` | anonymous | page | The question page. |
| `/forum/<forum>/faq`, `/forum/<forum>/faq/karma` | anonymous | page | The guidelines and the reputation guide. |
| `/forum/<forum>/tag` and `/forum/<forum>/tag/<tag_char>` | anonymous | page | The tag page. |
| `/forum/get_tags` | anonymous | operation | The tag picker. |
| `/forum/get_url_title` | signed in | operation | Fetches the title of a pasted address for the editor. |
| `/forum/<forum>/ask` | signed in | page | The ask form. |
| `/forum/<forum>/new` and `/forum/<forum>/<post_parent>/reply` | signed in | operation | Creates a question or an answer. |
| `/forum/<forum>/post/<post>/edit`, `/forum/<forum>/post/<post>/save` | signed in | page, operation | Edits a post. |
| `/forum/<forum>/post/<post>/comment` | signed in | operation | Posts a comment. |
| `/forum/<forum>/post/<post>/upvote`, `/forum/<forum>/post/<post>/downvote` | signed in | operation | Votes. |
| `/forum/<forum>/post/<post>/toggle_correct` | signed in | operation | Accepts or un-accepts an answer. |
| `/forum/<forum>/post/<post>/delete` | signed in | operation | Deletes a post. |
| `/forum/<forum>/question/<question>/ask_for_close`, `/close`, `/reopen`, `/delete`, `/undelete` | signed in | page, operation | The closing, reopening and archiving of a question. |
| `/forum/<forum>/question/<question>/toggle_favourite` | signed in | operation | Bookmarks a question. |
| `/forum/<forum>/question/<question>/edit_answer` | signed in | page | Edits one's answer from the question page. |
| `/forum/<forum>/validation_queue`, `/flagged_queue`, `/offensive_posts`, `/closed_posts` | signed in | page | The four moderation queues. |
| `/forum/<forum>/post/<post>/validate`, `/refuse`, `/flag`, `/mark_as_offensive`, `/ask_for_mark_as_offensive` | signed in | page, operation | Moderation actions. |
| `/forum/<post>/ask_for_mark_as_offensive` | signed in | operation | The dialogue that asks for an offensive reason. |
| `/forum/<forum>/post/<post>/convert_to_comment`, `/forum/<forum>/post/<post>/comment/<comment>/convert_to_answer`, `/forum/<forum>/post/<post>/comment/<comment>/delete` | signed in | operation | Conversions and comment deletion. |
| `/forum/<forum>/partner/<partner_id>` | anonymous | page | Redirects to the public profile of a contact. |
| `/forum/user/<user_id>` | anonymous | page | The public profile of a member, in the forum context. |

### 4.4 Public profile

| Path | Access | Kind | Purpose |
|---|---|---|---|
| `/profile/user/<user_id>` | anonymous | page | The public profile. |
| `/profile/avatar/<user_id>` | anonymous | file | The profile picture. |
| `/profile/user/save` | signed in | operation | Saves the four editable fields and the publication marker. |
| `/profile/users` and `/profile/users/page/<page>` | anonymous | page | The member ranking. |
| `/profile/ranks_badges` | anonymous | page | The rank and badge catalogue, with the certification badges grouped separately. |
| `/profile/send_validation_email` | signed in | operation | Sends the address-validation message. |
| `/profile/validate_email` | anonymous | page | Consumes the validation token and grants three reputation points when the balance is still zero. |
| `/profile/validate_email/close` | anonymous | operation | Clears the two session markers that show the validation banner. |

## 5. Printable documents

| Document | Name | Produced from | File name rule |
|---|---|---|---|
| `survey.certification_report` | `Certifications` | a Survey Participation | `Certification - <questionnaire title>`, saved as `certification.pdf` |

The certificate is one page in landscape orientation with no margins; its content, its six visual
templates and its name sizing are described in
[`workflows.md`](workflows.md#workflow-7) and
[`LSG-CALC-019`](calculations.md#lsg-calc-019).

One server action is shipped, `survey.action_survey_print`, labelled `Print Survey` and bound to the
questionnaire form, which opens the printable view of the selected questionnaire.

## 6. Notifications and live updates

| Channel | Event | Payload |
|---|---|---|
| the questionnaire access token | `next_question` | The real current instant, so every attendee screen requests the next question and starts its countdown from the same moment. |
| the questionnaire access token | `end_session` | Nothing beyond the event; every attendee screen closes. |
| the course discussion thread | the new-content subtype | A message rendered from the course's new-content template. |
| the post discussion thread | the four post subtypes | New question, new answer, question edited and answer edited. |
| the forum discussion thread | the two forum subtypes | New question and new answer, mirrored for the followers of the forum. |
| the participation discussion thread | the completion subtype | The completion notice, re-posted as a child message on the questionnaire. |
| a contact's inbox | the badge grant | The badge message; comments on forum posts deliberately never become inbox notifications. |

## 7. External integrations

| Integration | Purpose | What is sent | What is received |
|---|---|---|---|
| The video-sharing service | Metadata of a video content item | The extracted eleven-character key | The title, the description, the illustration and the running time. The running time arrives in a compact duration form and is converted into decimal hours. |
| The external document storage service | Metadata of a document, a picture or a video content item | The extracted file key and the site key held in the settings | The title, the illustration, the media type — from which the precise subtype is derived — and, for a video, the running time in milliseconds; for a portable document the content is downloaded and the five-minutes-per-page estimate is applied. |
| The video-hosting platform | Metadata of a video content item | The extracted numeric key, optionally with its private hash | The title, the description, the illustration and the running time. |

Every retrieval is skipped while a package is being installed and skipped when the caller asks for
it. A retrieval never overwrites a field the author already filled. When a service cannot be
reached, the content item is created or saved with the fields the author supplied and nothing else.

## 8. Import and export

The domain defines no import or export format of its own. Every entity is importable and exportable
through the platform's generic tabular exchange, described in
[`../../data/data-loading-and-exchange.md`](../../data/data-loading-and-exchange.md). Four points
matter for a rebuild:

1. **Access tokens and session codes are not copied on duplication and must not be imported.** A
   questionnaire, a participation and a course each generate their own token; an import that carries
   one risks breaking the uniqueness constraints [`LSG-001`](business-rules.md#lsg-001),
   [`LSG-002`](business-rules.md#lsg-002) and the participation token constraint.
2. **A participation answer row must carry exactly the value field of its answer type**, or
   [`LSG-045`](business-rules.md#lsg-045) refuses it.
3. **A reputation balance must not be imported directly on a user** unless the ledger is imported
   with it: writing the balance creates a movement, so importing balances after importing movements
   produces a second movement per user.
4. **A course enrolment and a content progress record are unique per pair**, so an import must
   deduplicate before loading ([`LSG-064`](business-rules.md#lsg-064) and
   [`LSG-087`](business-rules.md#lsg-087)).

## 9. Reporting views offered

| Report | Entity | Views | Default grouping or filter |
|---|---|---|---|
| Course analysis | Course | list, graph, pivot, form | Published courses. |
| Content analysis | Course Content | graph, list, form, pivot | Published contents, excluding sections. |
| Attendee analysis | Course Enrolment | graph, pivot, list, card | Grouped by status. |
| Quiz analysis | Quiz Question | list, graph, pivot, form | — |
| Revenue analysis | the sales analysis of the selling domain | the sales views | Restricted to the products of the courses. |
| Participation analysis | Survey Participation | list, card, form | Grouped by questionnaire. |
| Detailed answers | Survey Participation Answer | list, form | Grouped by questionnaire and by participation. |
| Forum post analysis | Forum Post | graph | Grouped by forum. |
| Karma tracking | Karma Movement | list, form | Filterable by user, by instant, by source kind and by the consolidated marker. |
| The electronic-learning dashboard | Spreadsheet Dashboard | the dashboard view | Restricted to the Learning Manager group. |
