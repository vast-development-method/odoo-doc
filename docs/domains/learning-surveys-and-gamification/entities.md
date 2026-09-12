# Learning, Questionnaires and Recognition — Entities

Every entity of this domain, field by field: its purpose, its lifecycle, its complete field table,
its relations, its uniqueness rules, its defaults, its derived values with the rules that produce
them, its ordering, its display-name rule, its archival behaviour, its company behaviour and the
extension points other capability packages contribute to it. Validation rules are quoted with the
exact text the system shows.

## Conventions used in this file

Field tables carry these columns:

- **Field** — the storage name, reproduced exactly, in code font. Storage names are contractual: a
  replacement that must import an existing database or serve an existing integration depends on
  them character for character.
- **Full name** — the label the interface shows for that field, in words.
- **Type** — one of `boolean`, `integer`, `decimal`, `monetary`, `text`, `long_text`, `rich_text`,
  `date`, `datetime`, `binary`, `image`, `selection`, `reference`, `many_to_one`, `one_to_many`,
  `many_to_many`, `structured_data`, followed by the entity a relation points at. Every `datetime`
  is stored in coordinated universal time and displayed in the reader's time zone unless a field
  says otherwise.
- **Required**, **Default** — whether a value must be present, and what is proposed when none is
  supplied.
- **Stored** — **stored** means the value lives in the database; **derived** means it is recomputed
  from its inputs. A field marked *derived, stored, editable* is recomputed when its inputs change
  but can be overwritten by hand, and the manual value survives until an input changes again.
  *Precomputed* means the value is produced before the row is inserted, so a record created through
  an integration always carries it.
- **Copied** — whether the value is carried over when the record is duplicated.
- **Meaning and rules** — everything else: indexing, deletion behaviour of a relation, the selection
  values with their labels, the group that may see the field, and the business meaning.

Unless a table states otherwise, a field is optional, writable, stored, copied on duplication and
not indexed.

Every persistent entity also carries the five audit fields `id` (the surrogate integer primary key,
called the identifier in prose), `create_date` (creation instant), `create_uid` (creating user),
`write_date` (last change instant) and `write_uid` (last changing user). They are not repeated in
the per-entity tables. Entities that support archiving carry `active` (boolean, default true;
archived records are excluded from default queries); `active` is listed only where its meaning goes
beyond hiding the record.

Entities marked "carries a discussion thread" also carry the standard thread fields (message list,
follower list, unread counters); entities marked "carries planned activities" also carry the
standard activity fields. Both belong to the
[Messaging and activities](../messaging-and-activities/README.md) domain and are not repeated here.

Entities marked "publishable" carry `is_published` (boolean), `website_published` (boolean view of
the same state), `website_url` (derived public address) and the search-engine metadata fields; those
belong to the [Website and storefront](../website-and-storefront/README.md) domain. Entities marked
"carries an image" carry `image_1920` and the derived sizes `image_1024`, `image_512`, `image_256`
and `image_128`.

A message reproduced between backticks in this folder is shown to the user verbatim; a placeholder
written between angle brackets is replaced by the value named inside it. None of these strings is
authored by this specification; every one is reproduced because a compatible rebuild has to show
the same text.

---

# Part 1. Questionnaire entities

# 1. Survey

**Full name:** Survey. **Transport name** `survey.survey`, **storage name** `survey_survey`.
**Kind:** persistent entity. **Contributed by the capability package** `survey`, extended by
`survey_crm`, `hr_skills_survey`, `hr_recruitment_survey` and `website_slides_survey`. Generated
reference page: [`survey.survey`](../../references/entities/survey.survey.md).
**Carries a discussion thread.** **Carries planned activities.** **Archivable.**

The root of the questionnaire tree. It owns the flat ordered list of sections and questions, the
participations, and every setting that governs access, layout, scoring, certification and live
sessions.

**Default ordering:** by `create_date` descending.

**Display name rule:** the `title`.

**Uniqueness:** `access_token` is unique across all questionnaires; `session_code` is unique across
all questionnaires; `certification_badge_id` is unique across all questionnaires.

**Company scoping:** none. A questionnaire belongs to no company.

**Archiving:** supported. Archiving a questionnaire also archives its certification badge;
un-archiving reverses that. An archived questionnaire refuses new participations and refuses
invitations. The interface calls an archived questionnaire *closed*.

**Duplication:** the copy receives a new access token and a new session code, no badge, no session
state and no participations; sections, questions and answer options are copied, and the conditional
triggers of the copy are then re-pointed to the copied answer options by matching original and copy
position by position. When the caller supplies an explicit question list among the duplication
defaults, the re-pointing is skipped and the copy keeps triggers pointing at the original options.

## 1.1 Identity, ownership and presentation

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `title` | Survey Title | text (translatable) | yes | none | stored | yes | The questionnaire name, shown on its home page, in messages and on the certificate. |
| `survey_type` | Survey Type | selection | yes | `custom` | stored | yes | The purpose. Values: `survey` (Survey), `live_session` (Live session), `assessment` (Assessment), `custom` (Custom), and `recruitment` (Recruitment) added by the recruitment interview package. When the recruitment package is removed, records holding `recruitment` fall back to the default `custom`. Choosing a purpose in the form applies a package of defaults; see section 1.8. |
| `allowed_survey_types` | Allowed survey types | structured_data | no | none | derived | no | The list of purposes the reader may choose from: the four general purposes when the reader belongs to the Questionnaire Officer group, otherwise false. Depends on the reading user. The recruitment package adds `recruitment` to the list for recruitment managers. |
| `color` | Color Index | integer | no | 0 | stored | yes | Decoration index of the card view. |
| `description` | Description | rich_text (translatable) | no | empty | stored | yes | Shown on the questionnaire home page before the participant starts. Help text: "The description will be displayed on the home page of the survey. You can use this to give the purpose and guidelines to your candidates before they start it." |
| `description_done` | End Message | rich_text (translatable) | no | empty | stored | yes | Shown on the closing screen once the questionnaire is completed. |
| `background_image` | Background Image | image | no | none | stored | yes | Full-page background of every questionnaire screen. |
| `background_image_url` | Background Url | text | no | none | derived | no | `/survey/<access token>/get_background_image` when a background image and an access token both exist, otherwise false. |
| `active` | Active | boolean | yes | true | stored | yes | Archiving marker. |
| `user_id` | Responsible | many_to_one to User | no | the creating user | stored | yes | The person answerable for the questionnaire. Restricted to non-shared users. Changes are written into the discussion thread. |
| `restrict_user_ids` | Restricted to | many_to_many to User | no | empty | stored | yes | When non-empty, only these users and the Questionnaire Administrators may read and write the questionnaire, its questions, its answer options and its participations. Restricted to non-shared users. Changes are written into the discussion thread. |
| `lang_ids` | Languages | many_to_many to Language | no | the language of the creating context, otherwise the first installed language | stored | yes | The languages the questionnaire may be displayed in. Help text: "Leave the field empty to support all installed languages." Restricted to active languages. |

## 1.2 Structure and layout

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question_and_page_ids` | Sections and Questions | one_to_many to Survey Question | no | empty | stored | yes | The flat ordered list of sections and questions. Deletion behaviour on the target side: cascade. |
| `page_ids` | Pages | one_to_many to Survey Question | — | — | derived | no | The subset of the list whose `is_page` is true. |
| `question_ids` | Questions | one_to_many to Survey Question | — | — | derived | no | The subset whose `is_page` is false. |
| `question_count` | # Questions | integer | — | — | derived | no | The number of records in `question_ids`. |
| `questions_layout` | Pagination | selection | yes | `page_per_question` | stored | yes | `page_per_question` (One page per question), `page_per_section` (One page per section), `one_page` (One page with all the questions). |
| `questions_selection` | Question Selection | selection | yes | `all` | stored | yes | `all` (All questions) or `random` (Randomized per Section). Help text: "If randomized is selected, you can configure the number of random questions by section. This mode is ignored in live session." |
| `progression_mode` | Display Progress as | selection | no | `percent` | stored | yes | `percent` (Percentage left) or `number` (Number). Help text: "If Number is selected, it will display the number of questions answered on the total number of question to answer." |
| `has_conditional_questions` | Contains conditional questions | boolean | — | — | derived | no | True when at least one entry of the list carries triggering answer options. |

## 1.3 Access, attempts and time

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `access_mode` | Access Mode | selection | yes | `public` | stored | yes | `public` (Anyone with the link) or `token` (Invited people only). |
| `access_token` | Access Token | text | no | a newly generated universally unique value | stored | no | The public identifier that appears in every questionnaire address. Unique. |
| `users_login_required` | Require Login | boolean | no | false | stored | yes | Help text: "If checked, users have to login before answering even with a valid token." |
| `users_can_go_back` | Users can go back | boolean | no | false | stored | yes | Roaming. Help text: "If checked, users can go back to previous pages." |
| `users_can_signup` | Users can signup | boolean | — | — | derived | no | True when the platform's self-registration scope allows an external visitor to create an account. Identical for every questionnaire. |
| `is_attempts_limited` | Limited number of attempts | boolean | no | see derivation | derived, stored, editable | yes | Whether completed attempts per pool are capped. Help text: "Check this option if you want to limit the number of attempts per user". |
| `attempts_limit` | Number of attempts | integer | no | 1 | stored | yes | The cap, when attempts are limited. |
| `is_time_limited` | The survey is limited in time | boolean | no | false | stored | yes | Whether the whole questionnaire runs against a countdown. |
| `time_limit` | Time limit (minutes) | decimal | no | 10 | stored | yes | The countdown length, in minutes. |

## 1.4 Scoring and certification

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `scoring_type` | Scoring | selection | yes | see derivation | derived, stored, editable, precomputed | yes | `no_scoring` (No scoring), `scoring_with_answers_after_page` (Scoring with answers after each page), `scoring_with_answers` (Scoring with answers at the end), `scoring_without_answers` (Scoring without answers). |
| `scoring_success_min` | Required Score (%) | decimal | no | 80.0 | stored | yes | The percentage of the obtainable score a participation must reach to pass. |
| `scoring_max_obtainable` | Maximum obtainable score | decimal | — | — | derived | no | The highest total a participation of this questionnaire could reach; see [`calculations.md`](calculations.md#lsg-calc-001). |
| `certification` | Is a Certification | boolean | no | see derivation | derived, stored, editable, precomputed | yes | Whether a passing participation produces a certificate. |
| `certification_mail_template_id` | Certified Email Template | many_to_one to Message Template | no | none | stored | yes | Restricted to templates written on Survey Participation. Help text: "Automated email sent to the user when they succeed the certification, containing their certification document." |
| `certification_report_layout` | Certification template | selection | no | `modern_purple` | stored | yes | `modern_purple` (Modern Purple), `modern_blue` (Modern Blue), `modern_gold` (Modern Gold), `classic_purple` (Classic Purple), `classic_blue` (Classic Blue), `classic_gold` (Classic Gold). The stored value splits on the underscore into a shape (`modern` or `classic`) and a colour (`purple`, `blue` or `gold`). |
| `certification_give_badge` | Give Badge | boolean | no | see derivation | derived, stored, editable | no | Whether passing awards a badge. Can only be true on a certification that requires signing in. |
| `certification_badge_id` | Certification Badge | many_to_one to Badge | no | none | stored, indexed | no | The badge awarded. Unique across questionnaires. |
| `certification_badge_id_dummy` | Certification Badge | many_to_one | — | — | derived from `certification_badge_id` | no | A second view of the same link, used by the form to offer a simplified badge editor. |
| `certification_validity_months` | Validity | integer | no | 0 | stored | yes | Added by the skills-certification package. Help text: "Specify the number of months the certification is valid after being awarded. Enter 0 for certifications that never expire." |

## 1.5 Live session

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `session_available` | Live session available | boolean | — | — | derived | no | True when `survey_type` is `live_session` or `custom` **and** `certification` is false. Governs whether the session buttons appear. |
| `session_state` | Session State | selection | no | empty | stored | no | Empty means no session is open; `ready` (Ready) is the waiting room; `in_progress` (In Progress) means questions are being presented. |
| `session_code` | Session Code | text | no | see derivation | derived, stored, editable, precomputed | no | The short numeric code attendees type to join. Unique. Help text: "This code will be used by your attendees to reach your session. Feel free to customize it however you like!" |
| `session_link` | Session Link | text | — | — | derived | no | The public address of the session: the site address followed by `/s/<session code>` when a code exists, otherwise the site address followed by the questionnaire start path. |
| `session_question_id` | Current Question | many_to_one to Survey Question | no | none | stored | no | The question currently presented. Help text: "The current question of the survey session." |
| `session_start_time` | Current Session Start Time | datetime | no | none | stored | no | When the open session was started. Separates this session's participations from earlier ones. |
| `session_question_start_time` | Current Question Start Time | datetime | no | none | stored | no | When the current question started being presented. Help text: "The time at which the current question has started, used to handle the timer for attendees." |
| `session_answer_count` | Answers Count | integer | — | — | derived | no | The number of distinct creating users among the participations of this session that are not yet completed and were created at or after `session_start_time`. |
| `session_question_answer_count` | Question Answers Count | integer | — | — | derived | no | The number of distinct participations holding at least one answer line on the current question created at or after `session_start_time`. |
| `session_show_leaderboard` | Show Session Leaderboard | boolean | — | — | derived | no | True when `scoring_type` is not `no_scoring` **and** at least one entry of the list has `save_as_nickname` true. Help text: "Whether or not we want to show the attendees leaderboard for this survey." |
| `session_speed_rating` | Reward quick answers | boolean | no | false | stored | yes | Help text: "Attendees get more points if they answer quickly". |
| `session_speed_rating_time_limit` | Time limit (seconds) | integer | no | 0 | stored | yes | Help text: "Default time given to receive additional points for right answers". |

## 1.6 Results, statistics and links to other domains

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `user_input_ids` | User responses | one_to_many to Survey Participation | — | — | stored, read-only | no | Every attempt recorded against this questionnaire. Deletion behaviour on the target side: cascade. |
| `answer_count` | Registered | integer | — | — | derived | no | The number of participations, whatever their state, excluding test entries. |
| `answer_done_count` | Attempts | integer | — | — | derived | no | The number of participations in state `done`, excluding test entries. |
| `answer_score_avg` | Avg Score (%) | decimal | — | — | derived | no | The mean of `scoring_percentage` over the counted participations. |
| `answer_duration_avg` | Average Duration | decimal | — | — | derived | no | The mean duration, in hours, of the completed participations that carry both a start and an end instant. Help text: "Average duration of the survey (in hours)". |
| `success_count` | Success | integer | — | — | derived | no | The number of participations whose `scoring_success` is true. |
| `success_ratio` | Success Ratio (%) | integer | — | — | derived | no | `success_count` divided by `answer_count`, expressed as a whole percentage. |
| `generate_lead` | Lead Generating | boolean | — | — | derived, stored | yes | Added by the lead-generation package. True when the purpose is `survey`, `live_session` or `custom` and at least one question of the questionnaire is lead-generating. |
| `lead_ids` | Lead | one_to_many to Lead | — | — | stored | no | Added by the lead-generation package. The leads created from participations in this questionnaire; the inverse field on the lead is `origin_survey_id`. |
| `lead_count` | Leads | integer | — | — | derived | no | Added by the lead-generation package. Help text: "Number of leads created by this survey". Zero for a reader who may not read leads. |
| `team_id` | Assign Leads to | many_to_one to Sales Team | no | none | stored, indexed | yes | Added by the lead-generation package. Deletion behaviour: set to empty. |
| `hr_job_ids` | Job Position | one_to_many to Job Position | — | — | stored | no | Added by the recruitment interview package. The job positions that use this questionnaire as their interview form. |
| `slide_ids` | Certification Slides | one_to_many to Course Content | — | — | stored | no | Added by the course-certification package. Help text: "The slides this survey is linked to through the e-learning application". |
| `slide_channel_ids` | Certification Courses | one_to_many to Course | — | — | derived | no | Added by the course-certification package. The distinct courses reached through `slide_ids`. Visible only to the group `website_slides.group_website_slides_officer`. |
| `slide_channel_count` | Courses Count | integer | — | — | derived | no | Added by the course-certification package. The number of those courses. Visible only to the Learning Officer group. |

## 1.7 Derivation rules

1. **`scoring_type`.** When `certification` is true and the stored mode is empty or `no_scoring`,
   the mode becomes `scoring_without_answers`. When the stored mode is empty and the questionnaire
   is not a certification, it becomes `no_scoring`. Otherwise the stored value is kept. The field
   remains editable.
2. **`certification`.** The value becomes false when it is already false or when `scoring_type` is
   `no_scoring`; otherwise the stored value is kept. The field remains editable.
3. **`certification_give_badge`.** The value becomes false when it is already false, or when
   `users_login_required` is false, or when `certification` is false. It can therefore only be
   turned on for a certification that requires signing in.
4. **`is_attempts_limited`.** The value becomes false when it is already false, when the access mode
   is `public` and signing in is not required, or when any entry of the list carries triggering
   answer options. Attempt limiting is therefore impossible on a fully anonymous questionnaire and
   impossible on a questionnaire that has conditional questions.
5. **`session_code`.** For every questionnaire that has no code, one code is drawn; see
   [`calculations.md`](calculations.md#lsg-calc-010). Codes already drawn for the other
   questionnaires of the same batch are excluded.
6. **`session_available`.** True when `survey_type` is `live_session` or `custom` and `certification`
   is false.
7. **`session_show_leaderboard`.** True when `scoring_type` is not `no_scoring` and at least one
   entry has `save_as_nickname` true.
8. **`background_image_url`.** `/survey/<access token>/get_background_image` when both
   `background_image` and `access_token` are set, otherwise false.
9. **`scoring_max_obtainable`.** The sum, over the questions, of the question score when it is
   non-zero, otherwise the sum of the strictly positive option scores of that question.

## 1.8 On-change behaviour in the form

| Field changed | Effect |
|---|---|
| `survey_type` set to `survey` | `certification` becomes false, `is_time_limited` becomes false, `scoring_type` becomes `no_scoring`. |
| `survey_type` set to `live_session` | `access_mode` becomes `public`, `is_attempts_limited` becomes false, `is_time_limited` becomes false, `progression_mode` becomes `percent`, `questions_layout` becomes `page_per_question`, `questions_selection` becomes `all`, `scoring_type` becomes `scoring_with_answers`, `users_can_go_back` becomes false. |
| `survey_type` set to `assessment` | `access_mode` becomes `token`, `scoring_type` becomes `scoring_with_answers`. |
| `survey_type` set to `custom` | Nothing is forced. |
| `session_speed_rating` or `session_speed_rating_time_limit` | Every question of the questionnaire is re-aligned on the new speed settings, so the author sees the impact before saving; see section 2.9. |
| `restrict_user_ids` or `user_id` | When the restricted list is non-empty, the responsible is not in it and the responsible is not a Questionnaire Administrator, the responsible is appended to the restricted list. |

## 1.9 Database constraints

| Name | Statement | Message |
|---|---|---|
| `_access_token_unique` | unique(`access_token`) | `Access token should be unique` |
| `_session_code_unique` | unique(`session_code`) | `Session code should be unique` |
| `_certification_check` | check `scoring_type` is not `no_scoring` or `certification` is false | `You can only create certifications for surveys that have a scoring mechanism.` |
| `_scoring_success_min_check` | check `scoring_success_min` is empty or between 0 and 100 inclusive | `The percentage of success has to be defined between 0 and 100.` |
| `_time_limit_check` | check `is_time_limited` is false or `time_limit` is present and greater than zero | `The time limit needs to be a positive number if the survey is time limited.` |
| `_attempts_limit_check` | check `is_attempts_limited` is false or `attempts_limit` is present and greater than zero | `The attempts limit needs to be a positive number if the survey has a limited number of attempts.` |
| `_badge_uniq` | unique(`certification_badge_id`) | `The badge for each survey should be unique!` |
| `_session_speed_rating_has_time_limit` | check `session_speed_rating` is not true or `session_speed_rating_time_limit` is present and greater than zero | `A positive default time limit is required when the session rewards quick answers.` |

## 1.10 Validation rules

| Trigger | Condition | Message |
|---|---|---|
| Writing `scoring_type` or `users_can_go_back` | `scoring_type` is `scoring_with_answers_after_page` and `users_can_go_back` is true | `Combining roaming and "Scoring with answers after each page" is not possible; please update the following surveys: - %(survey_names)s`, where the placeholder is the list of offending titles, one per line, each prefixed by a hyphen. |
| Writing `user_id` or `restrict_user_ids` | The restricted list is non-empty, a responsible is set, the responsible is only a Questionnaire Officer and is not in the list | `The access of the following surveys is restricted. Make sure their responsible still has access to it:  %(survey_names)s`, where the placeholder lists one line per questionnaire with its title and its responsible. |
| Creating a participation | See rules [`LSG-021`](business-rules.md#lsg-021) to [`LSG-026`](business-rules.md#lsg-026) | Six distinct refusals, quoted there. |
| Opening the invitation composer | See rules [`LSG-027`](business-rules.md#lsg-027) to [`LSG-031`](business-rules.md#lsg-031) | Five distinct refusals, quoted there. |
| Opening or closing a live session | The acting user is not a Questionnaire Officer | `Only survey users can manage sessions.` |
| Deleting | The questionnaire is used as the certification of a course content item | `Uh-oh! You can’t delete surveys used as a Course Certification! Otherwise, students might think diplomas just grow on trees. The courses that need them are: %s`, where the placeholder lists one line per questionnaire as `- <certification title> (Courses - <course names>)`. |

## 1.11 Lifecycle

1. **Creation.** The access token and the session code are generated before the row is inserted.
   `scoring_type` and `certification` are precomputed. When `certification_give_badge` is true, the
   badge trigger is created at once: one goal definition, one challenge and one challenge line; see
   [`workflows.md`](workflows.md#workflow-6).
2. **Writing `certification_give_badge`.** Turning it on un-archives the badge and re-creates the
   goal definition, the challenge and the challenge line. Turning it off archives the badge, then
   deletes every challenge that rewards that badge together with the goal definitions those
   challenges use; challenge lines disappear with their challenge.
3. **Writing `session_speed_rating` or `session_speed_rating_time_limit`.** The questions that were
   never customised adopt the new values; the customised ones keep theirs but lose the customised
   marker when their values now equal the questionnaire values.
4. **Archiving.** The certification badge is archived with the questionnaire; un-archiving reverses
   it.
5. **Deletion.** Refused while the questionnaire backs a course certification, with the message
   quoted above. Otherwise the participations, the sections and the questions are deleted with it.

---

# 2. Survey Question

**Full name:** Survey Question. **Transport name** `survey.question`, **storage name**
`survey_question`. **Kind:** persistent entity. **Contributed by** `survey`, extended by
`survey_crm`. Generated reference page:
[`survey.question`](../../references/entities/survey.question.md).

One entry of the flat ordered list of a questionnaire. When `is_page` is true the entry is a
**section** (a page header); when it is false the entry is a **question**. The two live in one entity
so that an author can drag sections and questions past one another in a single list.

**Default ordering:** by `sequence`, then by identifier.

**Display name rule:** the `title`.

**Hierarchy:** derived, not stored on the author's side. A question belongs to the last section that
appears before it in the ordered list; `page_id` holds that section and is stored; `question_ids` on
a section holds the questions that follow it up to the next section.

**Archiving:** not supported.

## 2.1 Identity and position

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `title` | Title | text (translatable) | yes | none | stored | yes | The question text or the section name. |
| `description` | Description | rich_text (translatable) | no | empty | stored | yes | Extra explanation shown under the question, or the introduction of a section. Help text: "Use this field to add additional explanations about your question or to illustrate it with pictures or a video". A section with an empty description is never a screen of its own. |
| `survey_id` | Survey | many_to_one to Survey | no | none | stored, indexed | yes | The questionnaire. Deletion behaviour: cascade. |
| `sequence` | Sequence | integer | no | 10 | stored | yes | Position in the flat list. |
| `is_page` | Is a page? | boolean | no | false | stored | yes | True for a section, false for a question. |
| `question_ids` | Questions | one_to_many to Survey Question | — | — | derived | no | On a section: the questions that belong to it, in list order. Empty on a question. |
| `page_id` | Page | many_to_one to Survey Question | — | — | derived, stored | yes | On a question: the section it belongs to. Empty on a section. |
| `random_questions_count` | # Questions Randomly Picked | integer | no | 1 | stored | yes | On a section of a questionnaire that draws random questions: how many questions to draw. Help text: "Used on randomized sections to take X random questions from all the questions of that section." |
| `background_image` | Background Image | image | no | none | derived, stored, editable | yes | Available on sections only; forced to false on questions. |
| `background_image_url` | Background Url | text | — | — | derived | no | For a section with its own background, `/survey/<questionnaire access token>/<section identifier>/get_background_image`. For a question, the address of its section when that section has a background. Otherwise the questionnaire background address. |

## 2.2 Shape and answer options

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question_type` | Question Type | selection | no | see derivation | derived, stored, editable | yes | `simple_choice` (Multiple choice: only one answer), `multiple_choice` (Multiple choice: multiple answers allowed), `text_box` (Multiple Lines Text Box), `char_box` (Single Line Text Box), `numerical_box` (Numerical Value), `scale` (Scale), `date` (Date), `datetime` (Datetime), `matrix` (Matrix). Empty on a section. |
| `question_placeholder` | Placeholder | text (translatable) | no | none | derived, stored, editable | yes | Hint text inside the input box. Forced to false for `simple_choice`, `multiple_choice` and `matrix`. |
| `suggested_answer_ids` | Types of answers | one_to_many to Survey Answer Option | no | empty | stored | yes | The proposed choices of a choice question or the columns of a matrix question; the inverse field is `question_id`. Help text: "Labels used for proposed choices: simple choice, multiple choice and columns of matrix". Deletion behaviour on the target side: cascade. |
| `matrix_subtype` | Matrix Type | selection | no | `simple` | stored | yes | `simple` (One choice per row) or `multiple` (Multiple choices per row). |
| `matrix_row_ids` | Matrix Rows | one_to_many to Survey Answer Option | no | empty | stored | yes | The rows of a matrix question; the inverse field is `matrix_question_id`. Help text: "Labels used for proposed choices: rows of matrix". Deletion behaviour on the target side: cascade. |
| `has_image_only_suggested_answer` | Has image only suggested answer | boolean | — | — | derived | no | True when at least one option of the question has an empty text value; drives the picture-grid rendering. |
| `scale_min` | Scale Minimum Value | integer | no | 0 | stored | yes | Lowest value offered by a scale question. |
| `scale_max` | Scale Maximum Value | integer | no | 10 | stored | yes | Highest value offered by a scale question. |
| `scale_min_label` | Scale Minimum Label | text (translatable) | no | none | stored | yes | Caption under the lowest value. |
| `scale_mid_label` | Scale Middle Label | text (translatable) | no | none | stored | yes | Caption under the middle value. |
| `scale_max_label` | Scale Maximum Label | text (translatable) | no | none | stored | yes | Caption under the highest value. |
| `comments_allowed` | Show Comments Field | boolean | no | false | stored | yes | Whether a free-text comment box is shown under a choice or matrix question. |
| `comments_message` | Comment Message | text (translatable) | no | none | stored | yes | Caption of the comment box. |
| `comment_count_as_answer` | Comment is an answer | boolean | no | false | stored | yes | Whether a filled comment satisfies the mandatory check and counts as an answer in the statistics. |

## 2.3 Scoring

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `is_scored_question` | Scored | boolean | no | see derivation | derived, stored, editable | yes | Whether the question contributes to the score. Help text: "Include this question as part of quiz scoring. Requires an answer and answer score to be taken into account." |
| `answer_numerical_box` | Correct numerical answer | decimal | no | 0.0 | stored | yes | Help text: "Correct number answer for this question." |
| `answer_date` | Correct date answer | date | no | none | stored | yes | Help text: "Correct date answer for this question." |
| `answer_datetime` | Correct datetime answer | datetime | no | none | stored | yes | Help text: "Correct date and time answer for this question." |
| `answer_score` | Score | decimal | no | 0.0 | stored | yes | Help text: "Score value for a correct answer to this question." May not be negative. |
| `scoring_type` | Scoring Type | selection | — | — | derived from `survey_id.scoring_type`, read-only | no | Mirror of the questionnaire scoring mode. |

## 2.4 Validation of the participant's answer

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `constr_mandatory` | Mandatory Answer | boolean | no | false | stored | yes | Whether an answer is required. |
| `constr_error_msg` | Error message | text (translatable) | no | none | stored | yes | Shown when a mandatory question is left empty; when it is empty the default `This question requires an answer.` is shown. |
| `validation_required` | Validate entry | boolean | no | see derivation | derived, stored, editable | yes | Whether the format or range check below runs. Meaningful only for `char_box`, `numerical_box`, `date` and `datetime`. |
| `validation_email` | Input must be an email | boolean | no | false | stored | yes | Whether a single-line text answer must be a syntactically valid electronic mail address. |
| `validation_length_min` | Minimum Text Length | integer | no | 0 | stored | yes | Fewest characters accepted. |
| `validation_length_max` | Maximum Text Length | integer | no | 0 | stored | yes | Most characters accepted. |
| `validation_min_float_value` | Minimum value | decimal | no | 0.0 | stored | yes | Lowest number accepted. |
| `validation_max_float_value` | Maximum value | decimal | no | 0.0 | stored | yes | Highest number accepted. |
| `validation_min_date` | Minimum Date | date | no | none | stored | yes | Earliest date accepted. |
| `validation_max_date` | Maximum Date | date | no | none | stored | yes | Latest date accepted. |
| `validation_min_datetime` | Minimum Datetime | datetime | no | none | stored | yes | Earliest instant accepted. |
| `validation_max_datetime` | Maximum Datetime | datetime | no | none | stored | yes | Latest instant accepted. |
| `validation_error_msg` | Validation Error | text (translatable) | no | none | stored | yes | Shown when the format or range check fails; when it is empty the default `The answer you entered is not valid.` is shown. |

## 2.5 Carrying an answer onto the participation

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `save_as_email` | Save as user email | boolean | no | see derivation | derived, stored, editable | yes | On a single-line text question validated as an electronic mail address, the answer is copied into the participation's `email`. Help text: "If checked, this option will save the user's answer as its email address." |
| `save_as_nickname` | Save as user nickname | boolean | no | see derivation | derived, stored, editable | yes | On a single-line text question, the answer is copied into the participation's `nickname`, which is the name shown on the live-session leaderboard. Help text: "If checked, this option will save the user's answer as its nickname." |

## 2.6 Conditional display

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `triggering_answer_ids` | Triggering Answers | many_to_many to Survey Answer Option | no | empty | stored | no | Choosing any one of these options makes this question appear. Empty means the question is always displayed. The picker offers only options of `simple_choice` and `multiple_choice` questions of the same questionnaire positioned before this one. Help text: "Picking any of these answers will trigger this question. Leave the field empty if the question should always be displayed." |
| `triggering_question_ids` | Triggering Questions | many_to_many to Survey Question | — | — | derived | no | The distinct questions that own those options. Help text: "Questions containing the triggering answer(s) to display the current question." |
| `allowed_triggering_question_ids` | Allowed Triggering Questions | many_to_many to Survey Question | — | — | derived | no | Every choice question of the same questionnaire that has at least one option and is positioned before this question. |
| `is_placed_before_trigger` | Is misplaced? | boolean | — | — | derived | no | True when at least one owner of a triggering option is not in the allowed list, that is, the question was moved before one of its triggers. Help text: "Is this question placed before any of its trigger questions?" |

## 2.7 Timing and mirrors

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `is_time_limited` | The question is limited in time | boolean | no | the questionnaire speed marker when the question is created inside a questionnaire | stored | yes | Whether this question has its own countdown. Help text: "Currently only supported for live sessions." |
| `time_limit` | Time limit (seconds) | integer | no | the questionnaire default when the question is created inside a questionnaire | stored | yes | The countdown, in seconds. |
| `is_time_customized` | Customized speed rewards | boolean | no | false | stored | yes | True when the question's countdown settings differ from the questionnaire defaults. |
| `session_available` | Live Session available | boolean | — | — | derived from `survey_id.session_available`, read-only | no | Mirror. |
| `survey_session_speed_rating` | Survey Session Speed Rating | boolean | — | — | derived from `survey_id.session_speed_rating` | no | Mirror. |
| `survey_session_speed_rating_time_limit` | General Time limit (seconds) | integer | — | — | derived from `survey_id.session_speed_rating_time_limit` | no | Mirror. |
| `questions_selection` | Questions Selection | selection | — | — | derived from `survey_id.questions_selection`, read-only | no | Mirror. Help text: "If randomized is selected, add the number of random questions next to the section." |
| `survey_type` | Survey Type | selection | — | — | derived from `survey_id.survey_type` | no | Mirror. |
| `user_input_line_ids` | Answers | one_to_many to Survey Participation Answer | — | — | stored | no | Every non-skipped answer given to this question. Visible only to the group `survey.group_survey_user`. |
| `generate_lead` | Lead Generating | boolean | — | — | derived | no | Added by the lead-generation package. True when the shape is `simple_choice`, `multiple_choice` or `matrix` and at least one option is lead-generating. Help text: "At least one of the question answers can generate leads." |

## 2.8 Derivation rules

1. **`question_type`.** On a section the value is emptied; on a question that has no shape yet it
   becomes `simple_choice`.
2. **`is_scored_question`.** False when the value is undefined or the questionnaire is unscored; for
   a `date` question, true when a correct date is set; for a `datetime` question, true when a
   correct instant is set; for a `numerical_box` question, true when the correct number is non-zero;
   for `simple_choice` and `multiple_choice`, true when at least one option is marked correct; false
   for every other shape. Editable afterwards.
3. **`save_as_email`.** Forced to false unless the shape is `char_box` **and** `validation_email` is
   true.
4. **`save_as_nickname`.** Forced to false unless the shape is `char_box`.
5. **`validation_required`.** Forced to false unless the shape is one of `char_box`,
   `numerical_box`, `date`, `datetime`.
6. **`question_placeholder`.** Forced to false for `simple_choice`, `multiple_choice` and `matrix`.
7. **`background_image`.** Forced to false on questions.
8. **`page_id`.** Walking the questionnaire's list in order, the section in force at the position of
   this question.
9. **`question_ids` on a section.** The questionnaire's questions whose `page_id` is this section,
   in list order.
10. **`allowed_triggering_question_ids`.** The choice questions of the same questionnaire that have
    at least one option and whose `sequence` is lower than this question's sequence, or whose
    sequence is equal and whose identifier is lower. The sequence used is the stored one, not the
    one the client currently shows. For a question not yet saved, every choice question of the
    questionnaire is allowed.

## 2.9 On-change behaviour and speed re-alignment

| Field changed | Effect |
|---|---|
| `question_type` or `validation_required` | `validation_email` becomes false; `validation_length_min` and `validation_length_max` become 0; `validation_min_date`, `validation_max_date`, `validation_min_datetime` and `validation_max_datetime` become empty; `validation_min_float_value` and `validation_max_float_value` become 0. This stops a hidden value from blocking the save. |

**Speed re-alignment.** At creation, when the question belongs to a questionnaire and its countdown
settings differ from the questionnaire defaults — either `is_time_limited` differs from the
questionnaire speed marker, or the question is time limited and its `time_limit` differs from the
questionnaire default — `is_time_customized` is set to true. When the questionnaire speed settings
later change, every question that is not customised takes the new values; every customised question
keeps its values and then loses the customised marker when its `is_time_limited` equals the
questionnaire speed marker and either it is not time limited or its `time_limit` equals the
questionnaire default.

## 2.10 Database constraints

| Name | Statement | Message |
|---|---|---|
| `_positive_len_min` | check `validation_length_min` is at least zero | `A length must be positive!` |
| `_positive_len_max` | check `validation_length_max` is at least zero | `A length must be positive!` |
| `_validation_length` | check `validation_length_min` is at most `validation_length_max` | `Max length cannot be smaller than min length!` |
| `_validation_float` | check `validation_min_float_value` is at most `validation_max_float_value` | `Max value cannot be smaller than min value!` |
| `_validation_date` | check `validation_min_date` is at most `validation_max_date` | `Max date cannot be smaller than min date!` |
| `_validation_datetime` | check `validation_min_datetime` is at most `validation_max_datetime` | `Max datetime cannot be smaller than min datetime!` |
| `_positive_answer_score` | check `answer_score` is at least zero | `An answer score for a non-multiple choice question cannot be negative!` |
| `_scored_datetime_have_answers` | check the question is not scored, or its shape is not `datetime`, or `answer_datetime` is present | `All "Is a scored question = True" and "Question Type: Datetime" questions need an answer` |
| `_scored_date_have_answers` | check the question is not scored, or its shape is not `date`, or `answer_date` is present | `All "Is a scored question = True" and "Question Type: Date" questions need an answer` |
| `_scale` | check the shape is not `scale`, or `scale_min` is at least zero and `scale_max` is at most ten and `scale_min` is strictly lower than `scale_max` | `The scale must be a growing non-empty range between 0 and 10 (inclusive)` |
| `_is_time_limited_have_time_limit` | check `is_time_limited` is not true, or `time_limit` is present and greater than zero | `All time-limited questions need a positive time limit` |

## 2.11 Validation rules and lifecycle

| Trigger | Condition | Message |
|---|---|---|
| Writing `is_page` | A record has `is_page` true and a non-empty `question_type` | `Question type should be empty for these pages: %s`, where the placeholder is the comma-separated titles. |
| Deleting | A live session is in progress on the questionnaire | `You cannot delete questions from surveys "%(survey_names)s" while live sessions are in progress.` |

Duplicating a single question keeps its triggering options pointing at the original options;
re-pointing happens only when the whole questionnaire is duplicated. Deleting a question deletes its
options, its matrix rows and the recorded answers that reference it.

---

# 3. Survey Answer Option

**Full name:** Survey Answer Option (dictionary name Survey Label). **Transport name**
`survey.question.answer`, **storage name** `survey_question_answer`. **Kind:** persistent entity.
**Contributed by** `survey`, extended by `survey_crm`. Generated reference page:
[`survey.question.answer`](../../references/entities/survey.question.answer.md).

A proposed value. The same entity serves three roles: a choice of a `simple_choice` or
`multiple_choice` question, a column of a `matrix` question, and a row of a `matrix` question. The
role follows from which of the two owning links is filled.

**Default ordering:** by `question_id`, then `sequence`, then identifier.

**Display name rule:** for a row or column of a matrix question, the value label alone. Otherwise
`"<question title> : <value label>"`, shortened to at most ninety characters: the question title is
shortened first — never below thirty characters — by the number of characters in excess, then the
whole string is shortened to ninety characters. The shortening marker is three dots.

**Search by name** matches on the question title and on the value.

**Archiving:** not supported.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question_id` | Question | many_to_one to Survey Question | no | none | stored, indexed | yes | The question this is a choice or a column of. Deletion behaviour: cascade. |
| `matrix_question_id` | Question (as matrix row) | many_to_one to Survey Question | no | none | stored, indexed | yes | The matrix question this is a row of. Deletion behaviour: cascade. |
| `sequence` | Label Sequence order | integer | no | 10 | stored | yes | Position among the options or rows. |
| `value` | Suggested value | text (translatable) | no | none | stored | yes | The visible text. May be empty when the option is a picture. |
| `value_image` | Image | image (at most 1024 by 1024) | no | none | stored | yes | A picture shown instead of, or beside, the text. |
| `value_image_filename` | Image Filename | text | no | none | stored | yes | The original file name of the picture. |
| `value_label` | Value Label | text | — | — | derived | no | The text when it is non-empty; otherwise the capital letter at the option's position among the question's options, taken from the twenty-six letters `A` to `Z` in order, so position zero gives `A`, position one gives `B`, position twenty-five gives `Z` and position twenty-six gives the character that follows `Z` in the character ordering, which is not a letter (**compatibility finding**: the twenty-seventh option is labelled with a punctuation character; a corrected behaviour would fall back to an empty label at position twenty-six, or continue with two-letter labels); an empty string at every position from twenty-seven onwards. Help text: "Answer label as either the value itself if not empty or a letter representing the index of the answer otherwise." |
| `is_correct` | Correct | boolean | no | false | stored | yes | Whether choosing this option is a correct answer. |
| `answer_score` | Score | decimal | no | 0.0 | stored | yes | Points added to the participation when this option is chosen. Help text: "A positive score indicates a correct choice; a negative or null score indicates a wrong answer". Negative values are allowed here, unlike the per-question score. |
| `question_type` | Question Type | selection | — | — | derived from `question_id.question_type` | no | Mirror. |
| `scoring_type` | Scoring Type | selection | — | — | derived from `question_id.scoring_type` | no | Mirror. |
| `generate_lead` | Lead creation | boolean | no | false | stored | yes | Added by the lead-generation package. Help text: "Creates a lead when participants choose this answer". |

**Database constraint `_value_not_empty`:** `value` is present or `value_image_filename` is present;
otherwise the save is refused with `Suggested answer value must not be empty (a text and/or an image
must be provided).`

**Validation rule.** On writing `question_id` or `matrix_question_id`: exactly one of the two links
must be filled. When both are filled, or neither, the save is refused with `A label must be attached
to only one question.`

**Lifecycle.** Created, duplicated and deleted with its owning question. It has no state.

---

# 4. Survey Participation

**Full name:** Survey Participation (dictionary name Survey User Input). **Transport name**
`survey.user_input`, **storage name** `survey_user_input`. **Kind:** persistent entity.
**Contributed by** `survey`, extended by `survey_crm`, `hr_skills_survey`, `hr_recruitment_survey`
and `website_slides_survey`. Generated reference page:
[`survey.user_input`](../../references/entities/survey.user_input.md).
**Carries a discussion thread.** **Carries planned activities.**

One attempt of one person at one questionnaire. It is created before the first screen is shown and
it carries the tokens that grant access.

**Default ordering:** by `create_date` descending.

**Display name rule:** the questionnaire title.

**Uniqueness:** `access_token` is unique across all participations.

**Archiving:** not supported.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `survey_id` | Survey | many_to_one to Survey | yes | none | stored, read-only, indexed | yes | The questionnaire attempted. Deletion behaviour: cascade. |
| `state` | Status | selection | yes | `new` | stored, read-only | yes | `new` (New), `in_progress` (In Progress), `done` (Completed). See [`state-machines.md`](state-machines.md#machine-1). |
| `start_datetime` | Start date and time | datetime | no | none | stored, read-only | yes | Written when the participation moves to `in_progress`. |
| `end_datetime` | End date and time | datetime | no | none | stored, read-only | yes | Written when the participation moves to `done`. |
| `deadline` | Deadline | datetime | no | none | stored | yes | Help text: "Datetime until customer can open the survey and submit answers". Written by the invitation composer and carried over on a retry. |
| `lang_id` | Language | many_to_one to Language | no | none | stored | yes | The language chosen on the start screen; used to render every later screen and the printable view. |
| `test_entry` | Test Entry | boolean | no | false | stored, read-only | yes | True for an attempt created by an officer testing the questionnaire, and for the temporary attempt used to preview a certificate. Test attempts are excluded from the statistics and from the attempt count, and they may open an archived questionnaire. |
| `last_displayed_page_id` | Last displayed question/page | many_to_one to Survey Question | no | none | stored | yes | The last screen shown; used to resume and to walk the skipped questions in roaming mode. |
| `access_token` | Identification token | text | yes | a newly generated universally unique value | stored, read-only | no | The private identifier of this attempt; appears in the participation address. Unique. |
| `invite_token` | Invite token | text | no | none | stored, read-only | no | Identifies a pool of attempts. Participations sharing one invite token consume the same allowance. Not unique. |
| `partner_id` | Contact | many_to_one to Contact | no | none | stored, read-only, indexed | yes | The contact behind the attempt, when known. |
| `email` | Email | text | no | none | stored, read-only | yes | The electronic mail address behind the attempt, or the contact's address. May be overwritten by an answer to a question marked `save_as_email`. |
| `nickname` | Nickname | text | no | none | stored | yes | Help text: "Attendee nickname, mainly used to identify them in the survey session leaderboard." Defaults to the user or contact name and may be overwritten by an answer to a question marked `save_as_nickname`. |
| `user_input_line_ids` | Answers | one_to_many to Survey Participation Answer | no | empty | stored | yes | The recorded answers and skips. Deletion behaviour on the target side: cascade. |
| `predefined_question_ids` | Predefined Questions | many_to_many to Survey Question | no | the questions drawn at creation | stored, read-only | yes | The exact set of questions this participation must answer. Fixed at creation so a random draw stays stable across resumes; pruned at completion of the conditional questions that were never triggered. |
| `scoring_percentage` | Score (%) | decimal | no | 0 | derived, stored | yes | The score as a percentage of the maximum obtainable for this participation's own question set; see [`calculations.md`](calculations.md#lsg-calc-002). |
| `scoring_total` | Total Score | decimal (two decimal places) | no | 0 | derived, stored | yes | The sum of the scores of the recorded answers. |
| `scoring_success` | Quiz Passed | boolean | no | false | derived, stored | yes | True when `scoring_percentage` is greater than or equal to the questionnaire's `scoring_success_min`. |
| `survey_first_submitted` | Survey First Submitted | boolean | no | false | stored | yes | True once the participant has reached the end at least once; in roaming mode it switches navigation into the skipped-questions pass. |
| `is_session_answer` | Is in a Session | boolean | no | false | stored | yes | Help text: "Is that user input part of a survey session or not." True when the attempt was created while a session was open. |
| `attempts_count` | Attempts Count | integer | — | — | derived | no | The number of completed, non-test attempts in the same pool. Equals one for any participation that is not a completed non-test attempt of an attempt-limited questionnaire. |
| `attempts_number` | Attempt n° | integer | — | — | derived | no | The ordinal of this attempt within its pool: the count of attempts of the pool with a lower identifier, plus one. |
| `survey_time_limit_reached` | Survey Time Limit Reached | boolean | — | — | derived | no | True when the questionnaire is time limited, the attempt is not a session attempt, a start instant exists, and the current instant is at or after the start instant plus the limit in minutes. |
| `question_time_limit_reached` | Question Time Limit Reached | boolean | — | — | derived | no | True when the attempt is a session attempt, the questionnaire has a current-question start instant, the current question is time limited, and the current instant is at or after that start instant plus the question limit in seconds. |
| `is_attempts_limited` | Limited number of attempts | boolean | — | — | derived from `survey_id.is_attempts_limited` | no | Mirror. |
| `attempts_limit` | Number of attempts | integer | — | — | derived from `survey_id.attempts_limit` | no | Mirror. |
| `scoring_type` | Scoring | selection | — | — | derived from `survey_id.scoring_type` | no | Mirror. |
| `lead_id` | Lead | many_to_one to Lead | no | none | stored | yes | Added by the lead-generation package. Deletion behaviour: set to empty. At most one lead per participation. |
| `applicant_id` | Applicant | many_to_one to Application | no | none | stored, indexed | yes | Added by the recruitment interview package. |
| `slide_id` | Related course slide | many_to_one to Course Content | no | none | stored | yes | Added by the course-certification package. Help text: "The related course slide when there is no membership information". |
| `slide_partner_id` | Subscriber information | many_to_one to Content Progress | no | none | stored, indexed | yes | Added by the course-certification package. Help text: "Slide membership information for the logged in user". Cleared when the attendee leaves the course, so the attempt pool restarts on re-enrolment. |

**Database constraint `_unique_token`:** unique(`access_token`); the refusal is `An access token must
be unique!`

**Validation rule.** Saving an answer for a question that already has one, without asking for an
overwrite, is refused with `This answer cannot be overwritten.`

## 4.1 Lifecycle

1. **Creation.** The set of questions to ask is drawn and written into `predefined_question_ids`
   unless the caller supplies one. When the questionnaire has an open session in state
   `in_progress`, the participation is created directly in state `in_progress` with the start
   instant set. When the questionnaire is attempt limited and the access mode is not `public`, an
   invite token is generated unless the caller supplies one. The contact, the electronic mail
   address and the nickname are filled from the user, from the supplied contact or from the supplied
   address, in that order. Immediately afterwards, every single-line text question marked
   `save_as_email` or `save_as_nickname` receives a pre-filled answer taken from the participation.
2. **Start.** Moving to `in_progress` writes the start instant.
3. **Answering.** Each submitted screen writes or replaces the answers of the questions on that
   screen and updates `last_displayed_page_id`.
4. **Completion.** Moving to `done` writes the end instant, prunes `predefined_question_ids` of the
   conditional questions that were never triggered, posts the completion notice to the followers of
   the questionnaire, sends the certificate message when the questionnaire is a certification, the
   participation passed, a certificate template is configured and the attempt is not a test, runs
   the badge challenge when the questionnaire awards a badge, writes an employee résumé line when
   the skills-certification package is installed, and creates a lead when a lead-generating option
   was chosen.
5. **Deletion.** Deletes its answers. A Questionnaire Officer may delete a participation; only a
   Questionnaire Administrator may create or delete individual participation answers.

---

# 5. Survey Participation Answer

**Full name:** Survey Participation Answer (dictionary name Survey User Input Line).
**Transport name** `survey.user_input.line`, **storage name** `survey_user_input_line`.
**Kind:** persistent entity. **Contributed by** `survey`. Generated reference page:
[`survey.user_input.line`](../../references/entities/survey.user_input.line.md).

One stored answer, or one recorded skip, of one participation to one question. A choice question
with several selected options produces one record per selected option. A matrix question produces
one record per selected cell. A comment produces one extra record of answer type `char_box`.

**Default ordering:** by `question_sequence`, then identifier.

**Display name rule:** derived from the answer type — the text for `char_box`; the free text
shortened to fifty characters with the marker ` [...]` for `text_box`; the number for
`numerical_box`; the date for `date`; the instant converted to the reader's time zone for
`datetime`; the whole number for `scale`; the option value for `suggestion`; `"<option value>: <row
value>"` for a matrix cell; and `Skipped` when none of the above produced a value.

**Archiving:** not supported.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `user_input_id` | User Input | many_to_one to Survey Participation | yes | none | stored, indexed | yes | The participation. Deletion behaviour: cascade. |
| `survey_id` | Survey | many_to_one to Survey | — | the participation's questionnaire | derived from `user_input_id.survey_id`, stored | yes | Denormalised for filtering and for the record rules. |
| `question_id` | Question | many_to_one to Survey Question | yes | none | stored, indexed | yes | The question answered. Deletion behaviour: cascade. |
| `page_id` | Section | many_to_one to Survey Question | — | the question's section | derived from `question_id.page_id` | yes | Denormalised section, used to group the statistics. |
| `question_sequence` | Sequence | integer | — | the question's sequence | derived from `question_id.sequence`, stored | yes | Denormalised ordering key. |
| `lang_id` | Lang | many_to_one to Language | — | the participation's language | derived from `user_input_id.lang_id` | yes | Mirror. |
| `skipped` | Skipped | boolean | no | false | stored | yes | True when the question was displayed and left unanswered. |
| `answer_type` | Answer Type | selection | no | none | stored | yes | `text_box` (Free Text), `char_box` (Text), `numerical_box` (Number), `scale` (Number), `date` (Date), `datetime` (Datetime), `suggestion` (Suggestion). Empty exactly when `skipped` is true. |
| `value_char_box` | Text answer | text | no | none | stored | yes | A single-line text answer, or a comment. |
| `value_text_box` | Free Text answer | long_text | no | none | stored | yes | A multi-line text answer. |
| `value_numerical_box` | Numerical answer | decimal | no | 0.0 | stored | yes | A numerical answer. |
| `value_scale` | Scale value | integer | no | 0 | stored | yes | A scale answer. |
| `value_date` | Date answer | date | no | none | stored | yes | A date answer. |
| `value_datetime` | Datetime answer | datetime | no | none | stored | yes | A date-and-time answer. |
| `suggested_answer_id` | Suggested answer | many_to_one to Survey Answer Option | no | none | stored | yes | The chosen option, for a choice answer or for the column of a matrix cell. |
| `matrix_row_id` | Row answer | many_to_one to Survey Answer Option | no | none | stored | yes | The row of a matrix cell. |
| `answer_score` | Score | decimal | no | 0 | derived, stored, precomputed | yes | Points earned by this single answer, after the live-session speed adjustment; see [`calculations.md`](calculations.md#lsg-calc-003) and [`calculations.md`](calculations.md#lsg-calc-005). |
| `answer_is_correct` | Correct | boolean | no | false | derived, stored, precomputed | yes | Whether this single answer is correct. |

**Validation rules.**

| Trigger | Condition | Message |
|---|---|---|
| Writing `skipped` or `answer_type` | `skipped` equals whether `answer_type` is filled — both set, or both empty | `A question can either be skipped or answered, not both.` |
| Writing `skipped` or `answer_type` | The value field matching `answer_type` is empty, except a `numerical_box` answer whose value rounds to zero at six decimal places and a `scale` answer whose value is zero | `The answer must be in the right type` |

**Lifecycle.** Created, replaced or deleted by the submit operation of the participation. When a
choice question is answered again, the previous records of that question are deleted and new ones
are created; when a single-value question is answered again, the existing record is updated in
place.

---

# 6. Survey Invitation Wizard

**Full name:** Survey Invitation Wizard. **Transport name** `survey.invite`, **storage name**
transient `survey_invite`. **Kind:** transient entity, removed by the periodic cleanup.
**Contributed by** `survey`, extended by `hr_recruitment_survey`. Generated reference page:
[`survey.invite`](../../references/entities/survey.invite.md).

A composer that turns a list of contacts and free electronic mail addresses into participations and
sends each recipient a personal invitation. It carries the shared composer fields of the
[Messaging and activities](../messaging-and-activities/README.md) domain: a template, a subject, a
body, a language and the marker that says whether the body may still be edited.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `survey_id` | Survey | many_to_one to Survey | yes | from the calling context | stored | The questionnaire being shared. The composer renders every placeholder against Survey Participation. |
| `author_id` | Author | many_to_one to Contact | no | the contact of the acting user | stored, indexed | The sender shown on the message. Deletion behaviour: set to empty. |
| `partner_ids` | Recipients | many_to_many to Contact | no | empty | stored | Recipients picked from the contact list. The picker offers only contacts that already have a user account when the questionnaire requires signing in and self-registration is not allowed. |
| `emails` | Additional emails | long_text | no | empty | stored | Extra recipients typed as free text, separated by semicolons, commas, carriage returns or line feeds. |
| `existing_partner_ids` | Existing Partner | many_to_many to Contact | — | — | derived, read-only | Those chosen contacts that already have a participation on this questionnaire. |
| `existing_emails` | Existing emails | long_text | — | — | derived, read-only | Those typed addresses that already have a participation on this questionnaire, one per line. |
| `existing_mode` | Handle existing | selection | yes | `resend` | stored | `new` (New invite) creates a fresh participation for everybody; `resend` (Resend invite) reuses the most recent existing participation of each known recipient. |
| `existing_text` | Resend Comment | long_text | — | — | derived | The warning shown in the form: `The following customers have already received an invite: <names>.` and, on the next line when applicable, `The following emails have already received an invite: <addresses>.` |
| `mail_server_id` | Outgoing mail server | many_to_one to Outgoing Mail Server | no | none | stored | Optional server override. |
| `attachment_ids` | Attachments | many_to_many to Attachment | — | the attachments of the chosen template | derived, stored, editable | Files joined to every invitation; the association table is `survey_mail_compose_message_ir_attachments_rel`. Choosing a template replaces the list; adding a file afterwards does not re-trigger the replacement. |
| `deadline` | Answer deadline | datetime | no | none | stored | Written on every participation created or reused. |
| `send_email` | Send Email | boolean | — | derived | derived with an inverse that does nothing | True when the questionnaire's access mode is `token`. |
| `survey_start_url` | Survey uniform resource locator | text | — | — | derived | The absolute public start address of the questionnaire. |
| `survey_access_mode` | Survey Access Mode | selection | — | — | derived from `survey_id.access_mode`, read-only | Mirror. |
| `survey_users_login_required` | Survey Users Login Required | boolean | — | — | derived from `survey_id.users_login_required`, read-only | Mirror. |
| `survey_users_can_signup` | Survey Users Can Signup | boolean | — | — | derived from `survey_id.users_can_signup` | Mirror. |
| `applicant_id` | Applicant | many_to_one to Application | no | none | stored | Added by the recruitment interview package. |

**Derivation of the subject.** The template subject when a template is chosen and carries one,
otherwise `Participate to <questionnaire title>`.

**On-change behaviour.**

| Field changed | Effect |
|---|---|
| `emails` | Refused with `This survey does not allow external people to participate. You should create user accounts or update survey access mode accordingly.` when the questionnaire requires signing in and self-registration is not allowed. Otherwise each entry is parsed; when any entry is not a valid address the form is refused with `Some emails you just entered are incorrect: %s`; when all are valid the field is rewritten as one normalised address per line. |
| `partner_ids` | When the questionnaire requires signing in, self-registration is not allowed and some chosen contacts have no user account, the form is refused with `The following recipients have no user account: %s. You should create user accounts for them or allow external signup in configuration.` |

**Sending refusals.** `Please enter at least one valid recipient.` when neither a contact nor a
usable address remains, and `Unable to post message, please configure the sender's email address.`
when no sender address can be determined. The full procedure is
[`workflows.md`](workflows.md#workflow-3).

---

# Part 2. Learning platform entities

# 7. Course

**Full name:** Course. **Transport name** `slide.channel`, **storage name** `slide_channel`.
**Kind:** persistent entity. **Contributed by** `website_slides`, extended by `website_sale_slides`,
`website_slides_forum`, `website_slides_survey`, `hr_skills_slides` and `mass_mailing_slides`.
Generated reference page: [`slide.channel`](../../references/entities/slide.channel.md).
**Carries planned activities.** **Publishable per site.** **Carries an image.** **Archivable.**
**Carries a cover block** (background colour, background image, text alignment and height, supplied
by the [Website and storefront](../website-and-storefront/README.md) domain; the shipped defaults
are a transparent background with a linear gradient, zero opacity and a height that fits the
content). **Carries reviews** through the rating component of the same domain; only published
ratings count towards the average.

A container of ordered content items. It carries the enrolment policy, the visibility level, the
reward settings, the message templates, the statistics and the reviews.

**Default ordering:** by `sequence`, then identifier.

**Display name rule:** the `name`.

**Uniqueness:** `forum_id` is unique across courses when the course-forum package is installed.

**Company scoping:** none. A course belongs to a site, not to a company.

**Archiving:** supported. Archiving a course archives every one of its content items and unpublishes
the course. The course is archived **first** and its contents afterwards, precisely so the
completion recomputation triggered by archiving the contents does not see a course with zero
contents and mark every attendee finished. Un-archiving reverses the order: the contents first, the
course last.

**Duplication:** the copy is named `"<name> (copy)"`; the contents are copied with their sequences;
the enrolments, the progress records, the featured-content choice and the access token are not
copied; when the original's visibility is `members` and no enrolment policy is supplied, the copy's
policy is forced to `invite`.

## 7.1 Identity, ownership and presentation

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | yes (with `" (copy)"` appended) | The course title. |
| `active` | Active | boolean | yes | true | stored | yes | Archiving marker; changes are written into the discussion thread. |
| `sequence` | Sequence | integer | no | 10 | stored | yes | Order among courses. |
| `channel_type` | Course type | selection | yes | `training` | stored | yes | `training` (Training) or `documentation` (Documentation). Help text: "Defines the course type (e.g., \"Training\" for interactive learning, or \"Documentation\" for resources and guides)." |
| `user_id` | Responsible | many_to_one to User | no | the creating user | stored | yes | Receives access requests, is enrolled automatically and signs the completion message. |
| `description` | Description | rich_text (translatable) | no | empty | stored | yes | Help text: "The description that is displayed on top of the course page, just below the title". |
| `description_short` | Short Description | rich_text (translatable) | no | copied from `description` when that is filled and this is empty | stored | yes | Help text: "The description that is displayed on the course card". |
| `description_html` | Detailed Description | rich_text (translatable) | no | empty | stored | yes | The long description block of the course page. |
| `color` | Color Index | integer | no | 0 | stored | yes | Help text: "Used to decorate kanban view". |
| `tag_ids` | Tags | many_to_many to Course Tag | no | empty | stored | yes | Association table `slide_channel_tag_rel`. Help text: "Used to categorize and filter displayed channels/courses". |
| `access_token` | Security Token | text | no | a newly generated universally unique value | stored | no | Serves content files to non-members holding a share link. Generated per record, never shared between records. |
| `website_default_background_image_url` | Background image uniform resource locator | text | — | — | derived | no | The placeholder illustration of the course, chosen by `channel_type`. |

## 7.2 Contents and statistics

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `slide_ids` | Slides and categories | one_to_many to Course Content | no | empty | stored | yes | Every content item and every section, in order. |
| `slide_content_ids` | Content | one_to_many to Course Content | — | — | derived | no | The subset that is not a section. |
| `slide_category_ids` | Categories | one_to_many to Course Content | — | — | derived | no | The subset that is a section. |
| `slide_last_update` | Last Update | date | — | today | derived, stored | yes | Set to today whenever the publication state of any content item changes. |
| `slide_partner_ids` | Slide User Data | one_to_many to Content Progress | — | — | stored | no | Every progress record of every attendee on every content item of this course. Visible only to the Learning Officer group. |
| `promote_strategy` | Featured Content | selection | no | `latest` | stored | no | `latest` (Latest Created), `most_voted` (Most Voted), `most_viewed` (Most Viewed), `specific` (Select Manually), `none` (None). Help text: "Defines the content that will be promoted on the course home page". |
| `promoted_slide_id` | Promoted Slide | many_to_one to Course Content | no | none | stored | no | The manually chosen featured content item. |
| `nbr_document` | Documents | integer | — | — | derived, stored | no | Count of published, active, non-section contents of category `document`. |
| `nbr_video` | Videos | integer | — | — | derived, stored | no | The same for `video`. |
| `nbr_infographic` | Infographics | integer | — | — | derived, stored | no | The same for `infographic`. |
| `nbr_article` | Articles | integer | — | — | derived, stored | no | The same for `article`. |
| `nbr_quiz` | Number of Quizs | integer | — | — | derived, stored | no | The same for `quiz`. |
| `nbr_certification` | Number of Certifications | integer | — | — | derived, stored | no | Added by the course-certification package. The same for `certification`. |
| `total_slides` | Number of Contents | integer | — | — | derived, stored | no | The total count of published, active, non-section contents. This is the denominator of every completion percentage. |
| `total_views` | Visits | integer | — | — | derived, stored | no | The sum of the total view counts of those contents. |
| `total_votes` | Votes | integer | — | — | derived, stored | no | The sum of the likes minus the sum of the dislikes of those contents. |
| `total_time` | Duration | decimal (two decimal places) | — | — | derived, stored | no | The sum of the durations of those contents, in hours. |
| `rating_avg_stars` | Rating Average (Stars) | decimal (one decimal place) | — | — | derived | no | The published rating average expressed on the star scale. |

## 7.3 Visibility, enrolment and attendees

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `visibility` | Show Course To | selection | yes | `public` | stored | yes | `public` (Everyone), `connected` (Signed In), `members` (Course Attendees), `link` (Anyone with the link). Help text: "Defines who can access your courses and their content." |
| `enroll` | Enroll Policy | selection | yes | `public` | derived, stored, editable | no | `public` (Open), `invite` (On Invitation), and `payment` (On payment) added by the course-selling package. Help text: "Defines how people can enroll to your Course." When the selling package is removed, courses holding `payment` fall back to `invite`. |
| `enroll_msg` | Enroll Message | rich_text (translatable) | no | `Contact Responsible` | stored | yes | Help text: "Message explaining the enroll process". Shown to a visitor who may not enrol themselves. |
| `enroll_group_ids` | Auto Enroll Groups | many_to_many to Access Group | no | empty | stored | yes | Help text: "Members of those groups are automatically added as members of the channel." |
| `upload_group_ids` | Upload Groups | many_to_many to Access Group | no | empty | stored | yes | Association table `rel_upload_groups`. Visible only to internal users. Help text: "Group of users allowed to publish contents on a documentation course." |
| `channel_partner_ids` | Enrolled Attendees Information | one_to_many to Course Enrolment | — | — | stored | no | The enrolments whose status is not `invited`. Visible only to the Learning Officer group. |
| `channel_partner_all_ids` | All Attendees Information | one_to_many to Course Enrolment | — | — | stored | no | Every enrolment, pending invitations included. Visible only to the Learning Officer group. |
| `partner_ids` | Attendees | many_to_many to Contact | — | — | derived, searchable | no | The contacts of the active enrolments whose status is not `invited`. Help text: "Enrolled partners in the course". |
| `members_count` | # Enrolled Attendees | integer | — | — | derived | no | Enrolments in status `joined`, `ongoing` or `completed`. |
| `members_all_count` | # Enrolled or Invited Attendees | integer | — | — | derived | no | Enrolments in any status. |
| `members_engaged_count` | # Active Attendees | integer | — | — | derived | no | Enrolments in status `joined` or `ongoing`. Help text: "Active attendees include both 'joined' and 'ongoing' attendees." |
| `members_completed_count` | # Completed Attendees | integer | — | — | derived | no | Enrolments in status `completed`. |
| `members_invited_count` | # Invited Attendees | integer | — | — | derived | no | Enrolments in status `invited`. |
| `members_certified_count` | # Certified Attendees | integer | — | — | derived | no | Added by the course-certification package. Enrolments whose `survey_certification_success` is true. |
| `prerequisite_channel_ids` | Prerequisites | many_to_many to Course | no | empty | stored | yes | Association table `slide_channel_prerequisite_slide_channel_rel`. The picker offers only other courses with the same visibility and the same publication state. Help text: "Prerequisite courses to complete before accessing this one." |
| `prerequisite_of_channel_ids` | Prerequisite Of | many_to_many to Course | no | empty | stored | yes | The inverse list over the same association table. Help text: "Courses that have this course as prerequisite." |
| `prerequisite_user_has_completed` | Has Completed Prerequisite | boolean | — | — | derived | no | True when the reader holds a `completed` enrolment on **every** prerequisite; a course with no prerequisite therefore always reports true. |

## 7.4 Reader-dependent markers

| Field | Full name | Type | Stored | Meaning and rules |
|---|---|---|---|---|
| `is_member` | Is Enrolled Attendee | boolean | derived, searchable | True when the reader holds an active enrolment in status `joined`, `ongoing` or `completed`. Always false for the anonymous visitor. Help text: "Is the attendee actively enrolled." |
| `is_member_invited` | Is Invited Attendee | boolean | derived, searchable | True when the reader holds an active enrolment in status `invited`. Help text: "Is the invitation for this attendee pending." |
| `is_visible` | Is Visible On Website | boolean | derived, searchable | True when the visibility is `public`, or the reader is enrolled, or the reader is signed in and the visibility is `connected`. |
| `completed` | Done | boolean | derived | True when the reader's enrolment status is `completed`. |
| `completion` | Completion | integer | derived | One hundred when the reader has completed; otherwise the reader's completed-content count over `total_slides`, as a whole percentage. |
| `can_upload` | Can Upload | boolean | derived | True when the reader is the responsible, or the upload groups are set and the reader belongs to one of them, or the reader is a Learning Manager. |
| `can_publish` | Can Publish | boolean | derived | False when the reader may not upload; otherwise true when the reader is the responsible or a Learning Manager. An invited attendee previewing the course may never publish. |
| `can_review` | Can Review | boolean | derived | True when the reader may publish on the course, or is enrolled and holds at least `karma_review` reputation points. |
| `can_comment` | Can Comment | boolean | derived | True when the reader may publish on the course, or is enrolled and holds at least `karma_slide_comment` reputation points. |
| `can_vote` | Can Vote | boolean | derived | True when the reader may publish on the course, or is enrolled and holds at least `karma_slide_vote` reputation points. |
| `has_requested_access` | Access Requested | boolean | derived | True when a to-do activity on this course names the reader's contact as the requesting contact. |
| `partner_has_new_content` | Partner Has New Content | boolean | derived | True when at least one non-section content published in the last seven days has not been completed by the reader. |

## 7.5 Reward settings and message templates

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `karma_gen_channel_rank` | Course ranked | integer | no | 5 | stored | yes | Reputation points granted for posting a review on the course. |
| `karma_gen_channel_finish` | Course finished | integer | no | 10 | stored | yes | Reputation points granted for completing the course and taken back when it falls back below one hundred percent. A value of zero or less grants nothing. |
| `karma_review` | Add Review | integer | no | 10 | stored | yes | Help text: "Karma needed to add a review on the course". |
| `karma_slide_comment` | Add Comment | integer | no | 3 | stored | yes | Help text: "Karma needed to add a comment on a slide of this course". |
| `karma_slide_vote` | Vote | integer | no | 3 | stored | yes | Help text: "Karma needed to like/dislike a slide of this course." |
| `allow_comment` | Allow rating on Course | boolean | no | see derivation | derived, stored, editable, precomputed | yes | Help text: "Allow Attendees to like and comment your content and to submit reviews on your course." |
| `publish_template_id` | New Content Notification | many_to_one to Message Template | no | the shipped new-content template | stored | yes | Restricted to templates written on Course Content. Help text: "Defines the email your Attendees will receive each time you upload new content." |
| `share_channel_template_id` | Channel Share Template | many_to_one to Message Template | no | the shipped course-share template | stored | yes | Help text: "Email template used when sharing a channel". |
| `share_slide_template_id` | Share Template | many_to_one to Message Template | no | the shipped content-share template | stored | yes | Help text: "Email template used when sharing a slide". |
| `completed_template_id` | Completion Notification | many_to_one to Message Template | no | the shipped completion template | stored | yes | Restricted to templates written on Course Enrolment. Help text: "Defines the email your Attendees will receive once they reach the end of your course." |

## 7.6 Links added by other packages

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `product_id` | Product | many_to_one to Product Variant | no | the single product whose service tracking is `course` when exactly one exists | stored, indexed | yes | Added by the course-selling package. Restricted to products whose service tracking is `course`. |
| `product_sale_revenues` | Total revenues | monetary | — | — | derived | no | Added by the course-selling package. The total of the confirmed sales-analysis amounts of the linked product. Visible only to the group `sales_team.group_sale_salesman`. |
| `currency_id` | Currency | many_to_one to Currency | — | — | derived from `product_id.currency_id` | no | Currency of the revenue figure. |
| `forum_id` | Course Forum | many_to_one to Forum | no | none | stored, indexed | no | Added by the course-forum package. |
| `forum_total_posts` | Number of active forum posts | integer | — | — | derived from `forum_id.total_posts` | no | Added by the course-forum package. |

## 7.7 Derivation rules

1. **`enroll`.** Forced to `invite` for every course whose visibility is `members`.
2. **`allow_comment`.** False when `channel_type` is `documentation`, true otherwise; editable
   afterwards. When it is false, attendees may neither like, dislike, comment nor review.
3. **`is_visible`.** Visibility is `public`, or the reader is enrolled, or the reader is signed in
   and the visibility is `connected`. The searchable form resolves to enrolled courses plus the
   `public` ones for an anonymous visitor, and to enrolled courses plus the `public` and `connected`
   ones for a signed-in visitor. A `link` course therefore never appears in the public list although
   its page is reachable.
4. **`completion`.** See [`calculations.md`](calculations.md#lsg-calc-011).
5. **`can_publish`.** On a training course only the responsible and the Learning Managers may
   publish; on a documentation course the members of the upload groups may publish too, because they
   satisfy `can_upload`.
6. The five attendee counters come from one grouping of the enrolments by status.

## 7.8 Database constraints

| Name | Statement | Message |
|---|---|---|
| `_check_enroll` | check `visibility` is not `members` or `enroll` is `invite` | `The Enroll Policy should be set to 'On Invitation' when visibility is set to 'Course Attendees'` |
| `_product_id_check` | check `enroll` is not `payment` or `product_id` is present | `Product is required for on payment channels.` |
| `_forum_uniq` | unique(`forum_id`) | `Only one course per forum!` |

## 7.9 Validation rules

| Trigger | Condition | Message |
|---|---|---|
| Posting a review | The reader may not review | `Not enough karma to review` |
| Posting a second review | The same author already posted a message carrying a rating on this course | `Only a single review can be posted per course.` |
| Adding attendees | The course is not open-enrolment and the acting user has no write access, and the caller asked for the strict check | `You are not allowed to add members to this course. Please contact the course responsible or an administrator.` |
| Sharing the whole course | The course has no course-share template | `Impossible to send emails. Select a "Channel Share Template" for courses %(course_names)s first` |
| Publishing content | The reader may not publish | `Publishing is restricted to the responsible of training courses or members of the publisher group for documentation courses` |
| Merging contacts | Two contacts being merged are enrolled in the same course | `You cannot merge these contacts because multiple contacts are enrolled in the same courses: <course names>` |

## 7.10 Lifecycle

1. **Creation.** When no enrolment list is supplied and the creator is not the system super-user,
   the creator's contact is enrolled. When a description is supplied and the short description is
   empty, the description is copied into it. After creation the responsible's contact is enrolled
   and the members of the automatic-enrolment groups are enrolled. When the course is sold and
   published, the linked product is published too. When a forum is attached, that forum's privacy is
   cleared.
2. **Writing `user_id`.** The new responsible's contact is enrolled, and every open to-do activity
   on the course is reassigned to the new responsible.
3. **Writing `enroll_group_ids`.** Every member of the resulting groups is enrolled.
4. **Writing the publication marker.** For a sold course, publishing the course publishes the
   product; unpublishing the course unpublishes the product unless another published course still
   uses it.
5. **Writing `forum_id`.** The new forum's privacy is cleared; the previous forum, when different,
   becomes private and is restricted to the Learning Officer group.
6. **Deletion.** The content items are deleted first and explicitly, then the course, so the
   statistics recomputation never runs against deleted contents.

---

# 8. Course Content

**Full name:** Course Content (dictionary name Slides). **Transport name** `slide.slide`,
**storage name** `slide_slide`. **Kind:** persistent entity. **Contributed by** `website_slides`,
extended by `website_slides_survey`. Generated reference page:
[`slide.slide`](../../references/entities/slide.slide.md). **Carries a discussion thread**
(posting a message requires read access). **Publishable.** **Carries an image.** **Archivable.**

One entry of the ordered list of a course. When `is_category` is true the entry is a **section**;
otherwise it is a **content item**.

**Default ordering:** by `sequence` ascending, then `is_category` ascending, then identifier
ascending. At an equal sequence, content items therefore come before sections.

**Alternative orderings used by the public pages:** by sequence; by total views descending (most
viewed); by likes descending (most voted); by publication instant descending (latest).

**Display name rule:** the `name`.

**Hierarchy:** derived. A content item belongs to the last section that appears before it in the
course list; `category_id` stores that section and `slide_ids` on a section lists its items.

**Archiving:** supported. Archiving a published non-section content also unpublishes it.

**Duplication:** the sequence of the copy is forced to zero unless the whole course is being
duplicated or a sequence is supplied, so a duplicate lands first and uncategorised.

## 8.1 Identity, position and publication

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Title | text (translatable) | yes | see derivation | derived, stored, editable | yes | The content title. |
| `channel_id` | Course | many_to_one to Course | yes | none | stored, indexed | yes | The owning course. Deletion behaviour: cascade. |
| `active` | Active | boolean | yes | true | stored | yes | Archiving marker; changes are written into the discussion thread. |
| `sequence` | Sequence | integer | no | 0 | stored | forced to 0 | Position in the course list. |
| `user_id` | Uploaded by | many_to_one to User | no | the acting user | stored | yes | Who uploaded the content. |
| `description` | Description | rich_text (translatable) | no | empty | stored | yes | Shown under the content title. |
| `is_category` | Is a category | boolean | no | false | stored | yes | True for a section. |
| `category_id` | Section | many_to_one to Course Content | — | — | derived, stored, indexed | yes | The section this content belongs to. |
| `slide_ids` | Content | one_to_many to Course Content | — | — | stored | yes | On a section, the content items it contains; the inverse field is `category_id`. |
| `tag_ids` | Tags | many_to_many to Content Tag | no | empty | stored | yes | Association table `rel_slide_tag`. |
| `is_preview` | Allow Preview | boolean | no | see derivation | derived, stored, editable | yes | When true the content is readable without enrolling. Help text: "The course is accessible by anyone : the users don't need to join the channel to access the content of the course." |
| `is_new_slide` | Is New Slide | boolean | — | — | derived | no | True when the content is published and its publication instant is within the last seven days. |
| `date_published` | Publish Date | datetime | no | none | stored, read-only | no | Written when the content is created published and rewritten each time the publication marker is set. Cleared at creation when the creator may not publish. |
| `website_id` | Website | many_to_one to Site | — | — | derived from `channel_id.website_id`, read-only | no | The site of the owning course. |
| `channel_type` | Channel type | selection | — | — | derived from `channel_id.channel_type` | no | Mirror. |
| `channel_allow_comment` | Allows comment | boolean | — | — | derived from `channel_id.allow_comment` | no | Mirror. |

## 8.2 Kind, source and payload

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `slide_category` | Category | selection | yes | `document` | stored | yes | `infographic` (Image), `article` (Article), `document` (Document), `video` (Video), `quiz` (Quiz), and `certification` (Certification) added by the course-certification package. When that package is removed, records holding `certification` fall back to the default `document`. |
| `slide_type` | Slide Type | selection | no | see derivation | derived, stored, editable | yes | `image` (Image), `article` (Article), `quiz` (Quiz), `pdf` (labelled "PDF"), `sheet` (Sheet), `doc` (Document), `slides` (Slides), `youtube_video` (video-sharing service video), `google_drive_video` (external document storage video), `vimeo_video` (video-hosting platform video), `certification` (Certification). Help text: "Subtype of the slide category, allows more precision on the actual file type / source type." When the certification package is removed, records holding `certification` are cleared. |
| `source_type` | Source Type | selection | yes | `local_file` | stored | yes | `local_file` (Upload from Device) or `external` (Retrieve from the external document storage service). |
| `url` | External uniform resource locator | text | no | none | stored | yes | The external address of the video or the external document. |
| `binary_content` | File | binary | no | none | stored | yes | The uploaded payload, kept as an attachment. |
| `image_binary_content` | Image Content | binary | — | — | derived from `binary_content`, editable | yes | The same payload through a picture-only picker. |
| `document_binary_content` | Portable Document Format Content | binary | — | — | derived from `binary_content`, editable | yes | The same payload through a document-only picker. |
| `image_google_url` | Image Link | text | — | — | derived from `url`, editable | yes | The same address as a picture link. |
| `document_google_url` | Document Link | text | — | — | derived from `url`, editable | yes | The same address as a document link. |
| `video_url` | Video Link | text | — | — | derived from `url`, editable | yes | The same address as a video link. |
| `html_content` | Hypertext markup content | rich_text (translatable) | no | empty | stored | yes | The authored body of an article. Help text: "Custom HTML content for slides of category 'Article'." |
| `video_source_type` | Video Source | selection | — | — | derived | no | `youtube` (video-sharing service), `google_drive` (external document storage service), `vimeo` (video-hosting platform), recognised from the address. |
| `youtube_id` | Video identifier on the video-sharing service | text | — | — | derived | no | The eleven-character key extracted from a video-sharing address. |
| `vimeo_id` | Video identifier on the video-hosting platform | text | — | — | derived | no | The six to eleven digit key, optionally followed by a private hash. |
| `google_drive_id` | Identifier on the external document storage service | text | — | — | derived | no | The file key extracted from an address of the form `.../d/<key>`. |
| `slide_icon_class` | Slide Icon class | text | — | — | derived | no | The icon name matching the subtype: a picture icon for `image`, a text-page icon for `article`, a question-mark icon for `quiz`, a portable-document icon for `pdf`, a spreadsheet icon for `sheet`, a word-processor icon for `doc`, a presentation icon for `slides`, the service icon for each of the three video subtypes, a trophy for `certification`, and a generic file icon otherwise. |
| `completion_time` | Duration | decimal (four decimal places) | no | see derivation | derived, stored, editable, recursive | yes | The estimated duration in hours. |
| `slide_resource_ids` | Additional Resource for this slide | one_to_many to Content Resource | no | empty | stored | yes | Extra files and links. |
| `slide_resource_downloadable` | Allow Download | boolean | no | false | stored | yes | Help text: "Allow the user to download the content of the slide." |
| `survey_id` | Certification | many_to_one to Survey | no | none | stored, indexed | yes | Added by the course-certification package. |

## 8.3 Quiz settings

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question_ids` | Questions | one_to_many to Quiz Question | no | empty | stored | yes | The quiz attached to this content item. |
| `questions_count` | Numbers of Questions | integer | — | — | derived | no | The number of quiz questions. |
| `quiz_first_attempt_reward` | Reward: first attempt | integer | no | 10 | stored | yes | Reputation points granted when the quiz is passed on the first attempt. |
| `quiz_second_attempt_reward` | Reward: second attempt | integer | no | 7 | stored | yes | On the second attempt. |
| `quiz_third_attempt_reward` | Reward: third attempt | integer | no | 5 | stored | yes | On the third attempt. |
| `quiz_fourth_attempt_reward` | Reward: every attempt after the third try | integer | no | 2 | stored | yes | On the fourth and every later attempt. |

## 8.4 Attendee data, counters and embedding

| Field | Full name | Type | Stored | Meaning and rules |
|---|---|---|---|---|
| `partner_ids` | Subscribers | many_to_many to Contact | stored, not copied | The contacts holding a progress record; association table `slide_slide_partner`. Visible only to the Learning Officer group. |
| `slide_partner_ids` | Subscribers information | one_to_many to Content Progress | stored, not copied | The progress records. Visible only to the Learning Officer group. |
| `user_membership_id` | Subscriber information | many_to_one to Content Progress | derived | The reader's own progress record. Help text: "Subscriber information for the current logged in user". |
| `user_vote` | User vote | integer | derived | The reader's own vote: plus one, zero or minus one. |
| `user_has_completed` | Is Member | boolean | derived | Whether the reader completed this content. |
| `user_has_completed_category` | Is Category Completed | boolean | derived | Whether the reader completed every content item of this content's section. |
| `can_self_mark_completed` | Can Mark Completed | boolean | derived | Help text: "The slide can be marked as completed even without opening it". True when the content is published, the reader is enrolled, the category is not `quiz` and there is no quiz question. Forced false for a certification. |
| `can_self_mark_uncompleted` | Can Mark Uncompleted | boolean | derived | Help text: "The slide can be marked as not completed and the progression". True when the content is published and the reader is enrolled. Forced false for a certification. |
| `likes` | Likes | integer | derived, stored | The count of progress records whose vote is plus one. |
| `dislikes` | Dislikes | integer | derived, stored | The count of progress records whose vote is minus one. |
| `slide_views` | # of Website Views | integer | derived, stored | The count of progress records on this content, that is, the number of signed-in people who opened it. |
| `public_views` | # of Public Views | integer | stored, read-only, not copied | The count of anonymous opens of the content page. |
| `total_views` | # Total Views | integer | derived, stored | `slide_views` plus `public_views`. |
| `comments_count` | Number of comments | integer | derived | The number of public messages on this content. |
| `embed_ids` | External Slide Embeds | one_to_many to Content Embed Counter | stored | One record per third-party address the content was embedded on. |
| `embed_count` | # of Embeds | integer | derived | The sum of the view counts of those records. |
| `embed_code` | Embed Code | rich_text | derived, read-only | The markup that embeds this content inside a page of this platform. |
| `embed_code_external` | External Embed Code | rich_text | derived, read-only | Help text: "Same as 'Embed Code' but used to embed the content on an external website." Loading it increments the counters. |
| `website_share_url` | Share uniform resource locator | text | derived | `/slides/slide/<identifier>/share`. |
| `nbr_document`, `nbr_video`, `nbr_infographic`, `nbr_article`, `nbr_quiz`, `nbr_certification`, `total_slides` | Per-category counters | integer | derived, stored | On a section only: the counts of the published, non-section content items that belong to it, by category and in total. |

## 8.5 Derivation rules

1. **`name`.** When a certification questionnaire is linked and the title is still empty, the
   questionnaire title is taken.
2. **`category_id`.** Walking the course list sorted by sequence, with content items placed before
   sections at an equal sequence, the section in force at the position of this content item.
3. **`slide_type`.** For `document` with a local file, `pdf`; for `document` with an external
   source, the stored value is kept when it is already one of `pdf`, `sheet`, `doc` or `slides`, and
   otherwise cleared so that the retrieved metadata may fill it; for `infographic`, `image`; for
   `article`, `article`; for `quiz`, `quiz`; for `certification`, `certification`; for `video`, the
   value follows the recognised service — `youtube_video`, `google_drive_video` or `vimeo_video`;
   otherwise cleared.
4. **`completion_time`.** On a section, the sum of the durations of the published content items it
   contains — a recursive computation, because a section's duration depends on its items'. On an
   uploaded portable document, five minutes per page; see
   [`calculations.md`](calculations.md#lsg-calc-014).
5. **`image_1920`.** For a locally uploaded picture, the uploaded payload itself; otherwise the
   stored value, or empty.
6. **`is_preview`.** Forced to true, and the record forced published, at creation and at write for a
   section; forced to false for a certification.
7. **`can_self_mark_completed` and `can_self_mark_uncompleted`.** Forced false for a certification,
   because completion there is decided by the questionnaire result.

## 8.6 Database constraints

| Name | Statement | Message |
|---|---|---|
| `_exclusion_html_content_and_url` | check `html_content` is empty or `url` is empty | `A slide is either filled with a url or HTML content. Not both.` |
| `_check_survey_id` | check `slide_category` is not `certification` or `survey_id` is present | `A slide of type 'certification' requires a certification.` |
| `_check_certification_preview` | check `slide_category` is not `certification` or `is_preview` is false | `A slide of type certification cannot be previewed.` |

## 8.7 Validation rules

| Trigger | Condition | Message |
|---|---|---|
| Posting a comment | The reader lacks the course comment threshold | `Not enough karma to comment` |
| Marking viewed | The reader is not enrolled | `You cannot mark a slide as viewed if you are not among its members.` |
| Marking completed | The reader may not mark it completed | `You cannot mark a slide as completed if you are not among its members.` |
| Marking not completed | The reader may not mark it uncompleted | `You cannot mark a slide as uncompleted if you are not among its members.` |
| Granting quiz points | The reader is not enrolled or the content is unpublished | `You cannot mark a slide quiz as completed if you are not among its members or it is unpublished.` |
| Taking quiz points back | The reader is not enrolled or the content is unpublished | `You cannot mark a slide quiz as not completed if you are not among its members or it is unpublished.` |
| Sharing one content item | The course has no content-share template | `Impossible to send emails. Select a "Share Template" for courses %(course_names)s first` |

## 8.8 On-change behaviour

| Field changed | Effect |
|---|---|
| `url`, `document_google_url`, `image_google_url` or `video_url` | The external metadata of the address is retrieved and every field still empty is filled from it: the title, the description, the illustration, the duration and the precise subtype. Fields the author already filled are never overwritten. |
| `document_binary_content` | For a locally uploaded portable document, the duration is estimated at five minutes per page and written into `completion_time`. |
| `slide_category` | Changing away from `infographic` clears the picture payload; changing away from `document` clears the document payload. This stops a stale payload from breaking the single-payload constraint before the form is saved. |

## 8.9 Lifecycle

1. **Creation.** The publication instant is cleared when the creator may not publish. A section is
   forced to preview and to published. A content created published receives the current instant as
   its publication instant. External metadata is retrieved when an address was supplied — skipped
   while a package is being installed and skipped when the caller asks for it. The duration is
   estimated when none was supplied. A published non-section content posts the new-content message
   on the course and triggers the recomputation of every attendee's completion. When a certification
   questionnaire is linked, the category is forced to `certification` and the badge challenge of
   that questionnaire is re-categorised under the learning platform.
2. **Writing.** Switching the category to `article` clears the external address; switching away from
   `article` clears the authored body. Setting the publication marker rewrites the publication
   instant and posts the new-content message. Changing an address re-runs the metadata retrieval,
   filling only the keys absent from the write and still empty on every record. Changing the
   publication or archiving marker unpublishes archived non-section contents and triggers the
   recomputation of every attendee's completion. Changing the certification questionnaire
   re-categorises the badge challenges of the old and the new questionnaire.
3. **Deletion.** Deleting a section first moves its content items to the front of the course list;
   the record is then deleted and every attendee's completion recomputed. Deleting a certification
   content resets the badge challenge of its questionnaire back to the certification category.

---

# 9. Course Enrolment

**Full name:** Course Enrolment (dictionary name Channel / Partners (Members)). **Transport name**
`slide.channel.partner`, **storage name** `slide_channel_partner`. **Kind:** persistent entity.
**Contributed by** `website_slides`, extended by `website_slides_survey` and `hr_skills_slides`.
Generated reference page:
[`slide.channel.partner`](../../references/entities/slide.channel.partner.md). **Archivable.**

The link between one contact and one course. It holds the attendee status, the completion figures
and the invitation data.

**Display name rule:** the contact name.

**Uniqueness:** one record per course and contact.

**Archiving:** supported. Leaving a course archives the enrolment instead of deleting it, which
preserves the progress records and therefore stops the same reputation points being earned twice.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `channel_id` | Course | many_to_one to Course | yes | none | stored, indexed | The course. Deletion behaviour: cascade. |
| `partner_id` | Partner | many_to_one to Contact | yes | none | stored, indexed | The attendee. Deletion behaviour: cascade. |
| `active` | Active | boolean | yes | true | stored | Archiving marker. |
| `member_status` | Attendee Status | selection | yes | `joined` | stored, read-only | `invited` (Invite Sent), `joined` (Joined), `ongoing` (Ongoing), `completed` (Finished). See [`state-machines.md`](state-machines.md#machine-4). |
| `completion` | % Completed Contents | integer | no | 0 | stored, averaged when grouped | The percentage of the course's published content items completed by this attendee. |
| `completed_slides_count` | # Completed Contents | integer | no | 0 | stored | The number of published, active content items completed by this attendee. |
| `survey_certification_success` | Certified | boolean | no | false | stored | Added by the course-certification package. True once the attendee has passed at least one certification of the course. |
| `next_slide_id` | Next Lesson | many_to_one to Course Content | — | — | derived | The first published, active, non-section content of the course, in sequence then identifier order, that this attendee has not completed. Empty when everything is done. |
| `invitation_link` | Invitation Link | text | — | — | derived | The personal address of the attendee; see [`courses.md`](courses.md#invitation-link). |
| `last_invitation_date` | Last Invitation Date | datetime | no | none | stored | When the last invitation was sent. Drives the three-month expiry of pending invitations. |
| `partner_email` | Partner Email | text | — | — | derived from `partner_id.email`, read-only | Mirror. |
| `channel_user_id` | Responsible | many_to_one to User | — | — | derived from `channel_id.user_id` | The course responsible. |
| `channel_type` | Channel Type | selection | — | — | derived from `channel_id.channel_type` | Mirror. |
| `channel_visibility` | Channel Visibility | selection | — | — | derived from `channel_id.visibility` | Mirror. |
| `channel_enroll` | Channel Enroll | selection | — | — | derived from `channel_id.enroll` | Mirror. |
| `channel_website_id` | Website | many_to_one to Site | — | — | derived from `channel_id.website_id` | Mirror. |
| `nbr_certification` | Nbr Certification | integer | — | — | derived from `channel_id.nbr_certification` | Added by the course-certification package. Mirror. |

**Database constraints.**

| Name | Statement | Message |
|---|---|---|
| `_channel_partner_uniq` | unique(`channel_id`, `partner_id`) | `A partner membership to a channel must be unique!` |
| `_check_completion` | check `completion` is between 0 and 100 inclusive | `The completion of a channel is a percentage and should be between 0% and 100.` |

**Lifecycle.**

1. **Creation.** Created by the enrolment operation with the requested status. Creating it with
   status `joined` also subscribes the contact to the course discussion thread for the new-content
   subtype.
2. **Recomputation of the completion.** Triggered whenever a progress record changes and whenever a
   content item is created published, published, unpublished, archived, un-archived or deleted.
   Enrolments in status `completed` and in status `invited` are skipped entirely. See
   [`calculations.md`](calculations.md#lsg-calc-011).
3. **Reaching one hundred percent.** The status becomes `completed`, the course-completion
   reputation points are granted once, the completion message is sent, and — when the skills
   package is installed — a résumé line of type Training is written for the matching employee.
4. **Dropping back below one hundred percent** because content was added: the course-completion
   reputation points are taken back.
5. **Leaving the course.** The contact is unsubscribed from the discussion thread and the enrolment
   is archived. The progress records are kept.
6. **Deletion.** Deleting the enrolment also deletes every progress record of that contact on the
   content items of that course.
7. **Expiry.** The periodic cleanup deletes every enrolment whose status is `invited`, whose
   completion is zero, and whose last invitation instant is either empty or older than three months.

---

# 10. Content Progress

**Full name:** Content Progress (dictionary name Slide / Partner decorated many-to-many).
**Transport name** `slide.slide.partner`, **storage name** `slide_slide_partner`.
**Kind:** persistent entity. **Contributed by** `website_slides`, extended by
`website_slides_survey`. Generated reference page:
[`slide.slide.partner`](../../references/entities/slide.slide.partner.md).

The link between one contact and one content item. It records that the content was opened, whether
it was completed, the vote cast and how many quiz attempts were made.

**Display name rule:** the contact name.

**Uniqueness:** one record per content item and contact.

**Archiving:** not supported.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `slide_id` | Content | many_to_one to Course Content | yes | none | stored, indexed | The content item. Deletion behaviour: cascade. |
| `partner_id` | Partner | many_to_one to Contact | yes | none | stored, indexed | The attendee. Deletion behaviour: cascade. |
| `channel_id` | Channel | many_to_one to Course | — | the content's course | derived from `slide_id.channel_id`, stored, indexed | Denormalised for grouping and for the record rules. Deletion behaviour: cascade. |
| `slide_category` | Slide Category | selection | — | — | derived from `slide_id.slide_category` | Mirror. |
| `vote` | Vote | integer | no | 0 | stored | Plus one for a like, minus one for a dislike, zero for no vote. |
| `completed` | Completed | boolean | no | false | stored | Whether the attendee finished this content. |
| `quiz_attempts_count` | Quiz attempts count | integer | no | 0 | stored | How many times the attendee submitted the quiz of this content. |
| `user_input_ids` | Certification attempts | one_to_many to Survey Participation | — | — | stored | Added by the course-certification package. The attempts made from this progress record. |
| `survey_certification_success` | Certification Succeeded | boolean | — | — | derived, stored | Added by the course-certification package; the storage name of the field is `survey_scoring_success`. True when at least one of those attempts passed. |

**Database constraints.**

| Name | Statement | Message |
|---|---|---|
| `_slide_partner_uniq` | unique(`slide_id`, `partner_id`) | `A partner membership to a slide must be unique!` |
| `_check_vote` | check `vote` is minus one, zero or one | `The vote must be 1, 0 or -1.` |

**Lifecycle.**

1. **Creation.** Created the first time the attendee opens the content, votes on it, marks it
   completed or starts its certification. Creating a record already marked completed immediately
   triggers the course completion recomputation.
2. **Writing `completed`.** Changing the marker triggers the course completion recomputation for the
   enrolments of the same contact on the same course whose status is neither `completed` nor
   `invited`.
3. **Certification success.** When `survey_scoring_success` becomes true, `completed` is forced
   true, and every enrolment of the same contact on the same course that is not yet marked certified
   becomes certified.
4. **Deletion.** Deleted with the content item, with the contact, or when the attendee's enrolment
   is deleted.

---

# 11. Quiz Question

**Full name:** Quiz Question (dictionary name Content Quiz Question). **Transport name**
`slide.question`, **storage name** `slide_question`. **Kind:** persistent entity.
**Contributed by** `website_slides`. Generated reference page:
[`slide.question`](../../references/entities/slide.question.md).

One question of the quiz attached to a content item. A quiz is one screen of questions, each with a
fixed set of answers; exactly the correct ones must all be chosen for the quiz to be passed.

**Default ordering:** by `sequence`. **Display name rule:** the `question`. **Archiving:** not
supported.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question` | Question Name | text (translatable) | yes | none | stored | yes | The question text. |
| `slide_id` | Content | many_to_one to Course Content | yes | none | stored, indexed | yes | The content item the quiz belongs to. Deletion behaviour: cascade. |
| `sequence` | Sequence | integer | no | 0 | stored | yes | Order among the quiz questions. |
| `answer_ids` | Answer | one_to_many to Quiz Answer | no | empty | stored | yes | The proposed answers. |
| `answers_validation_error` | Error on Answers | text | — | — | derived | no | `This question must have at least one correct answer and one incorrect answer.` when the answer set has no correct answer or only correct answers; an empty string otherwise. |
| `attempts_count` | Attempts Count | integer | — | — | derived | no | The sum of the quiz attempt counts of every progress record on the same content item. Visible only to the Learning Officer group. |
| `attempts_avg` | Attempts Avg | decimal (two decimal places) | — | — | derived | no | That sum divided by the number of progress records on the same content item. Visible only to the Learning Officer group. |
| `done_count` | Done Count | integer | — | — | derived | no | The number of progress records on the same content item that are completed. Visible only to the Learning Officer group. |

**Validation rule.** On writing `answer_ids`, when at least one question of the batch carries a
non-empty validation error, the save is refused with `All questions must have at least one correct
answer and one incorrect answer:  %s`, where the placeholder lists one line per offending question
as `- <content name>: <question>`.

---

# 12. Quiz Answer

**Full name:** Quiz Answer (dictionary name Slide Question's Answer). **Transport name**
`slide.answer`, **storage name** `slide_answer`. **Kind:** persistent entity. **Contributed by**
`website_slides`. Generated reference page:
[`slide.answer`](../../references/entities/slide.answer.md).

**Default ordering:** by `question_id`, then `sequence`, then identifier. **Display name rule:** the
`text_value`.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `question_id` | Question | many_to_one to Quiz Question | yes | none | stored, indexed | yes | The owning question. Deletion behaviour: cascade. |
| `sequence` | Sequence | integer | no | 0 | stored | yes | Order among the answers. |
| `text_value` | Answer | text (translatable) | yes | none | stored | yes | The answer text. |
| `is_correct` | Is correct answer | boolean | no | false | stored | yes | Whether choosing this answer is required for the quiz to be passed. |
| `comment` | Comment | long_text (translatable) | no | none | stored | yes | Help text: "This comment will be displayed to the user if they select this answer". Shown whether the answer is correct or not, and disclosed to a site designer at any time. |

---

# 13. Content Resource

**Full name:** Content Resource (dictionary name Additional resource for a particular slide).
**Transport name** `slide.slide.resource`, **storage name** `slide_slide_resource`.
**Kind:** persistent entity. **Contributed by** `website_slides`. Generated reference page:
[`slide.slide.resource`](../../references/entities/slide.slide.resource.md).

An extra file or link attached to a content item and offered for download beside it.

**Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `slide_id` | Slide | many_to_one to Course Content | yes | none | stored, indexed | yes | The content item. Deletion behaviour: cascade. |
| `resource_type` | Resource Type | selection | yes | none | stored | yes | `file` (File) or `url` (Link). |
| `name` | Name | text | no | `Resource` | derived, stored, editable | yes | Recomputed only while the current name is empty or still the literal default: for a file resource with a payload or a stored file name it becomes that file name; for a link resource it becomes the link. Once the author types their own name the recomputation stops touching it. |
| `data` | Resource | binary | no | none | derived, stored, editable | yes | The uploaded payload. Cleared when the resource type is `url`; left unchanged when it is `file`. |
| `file_name` | File Name | text | no | none | stored | yes | The original file name with its extension, used to build the download address. |
| `link` | Link | text | no | none | derived, stored, editable | yes | The external address. Cleared when the resource type is `file`; left unchanged when it is `url`. |
| `download_url` | Download uniform resource locator | text | — | — | derived | no | Empty when the name is empty. Otherwise the content-serving path of this resource with two query parameters: a download marker set to true and a file name built from `name`, to which the extension of `file_name` is appended when `name` does not already end with it. |
| `sequence` | Sequence | integer | no | 0 | stored | yes | Order among the resources. |

**Database constraints.**

| Name | Statement | Message |
|---|---|---|
| `_check_url` | check `resource_type` is not `url` or `link` is present | `A resource of type url must contain a link.` |
| `_check_file_type` | check `resource_type` is not `file` or `link` is empty | `A resource of type file cannot contain a link.` |

**Validation rule.** Writing a payload on a link resource is refused with `Resource
%(resource_name)s is a link and should not contain a data file`.

---

# 14. Content Embed Counter

**Full name:** Content Embed Counter (dictionary name Embedded Slides View Counter).
**Transport name** `slide.embed`, **storage name** `slide_embed`. **Kind:** persistent entity.
**Contributed by** `website_slides`. Generated reference page:
[`slide.embed`](../../references/entities/slide.embed.md).

A counter of loads of one content item embedded on one third-party address.

**Display name rule:** the `website_name`.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `slide_id` | Presentation | many_to_one to Course Content | yes | none | stored, indexed | The embedded content item. Deletion behaviour: cascade. |
| `url` | Third Party Website uniform resource locator | text | no | none | stored | The address of the embedding page. Empty when the browser sent no referring address; one record then collects every such load. |
| `website_name` | Website | text | — | — | derived | The host part of the address. |
| `count_views` | # Views | integer | no | 1 | stored | How many times the embedded content was loaded from that address. |

**Lifecycle.** Created with a count of one the first time a load is seen from a given address, and
incremented by one on every later load from the same address. The host part is normalised first: an
address with no host is stored as empty.

---

# 15. Content Tag

**Full name:** Content Tag (dictionary name Slide Tag). **Transport name** `slide.tag`,
**storage name** `slide_tag`. **Kind:** persistent entity. **Contributed by** `website_slides`.
Generated reference page: [`slide.tag`](../../references/entities/slide.tag.md).

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | The label. |

**Database constraint `_slide_tag_unique`:** unique(`name`); the refusal is `A tag must be unique!`

---

# 16. Course Tag

**Full name:** Course Tag (dictionary name Channel/Course Tag). **Transport name**
`slide.channel.tag`, **storage name** `slide_channel_tag`. **Kind:** persistent entity.
**Contributed by** `website_slides`. Generated reference page:
[`slide.channel.tag`](../../references/entities/slide.channel.tag.md).

A label attached to courses. Every course tag belongs to a family; filters combine tags of one
family with a logical or, and families with a logical and.

**Default ordering:** by `group_sequence` ascending, then `sequence` ascending.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Name | text (translatable) | yes | none | stored | The label. |
| `sequence` | Sequence | integer | yes | 10 | stored, indexed | Order within the family. |
| `group_id` | Group | many_to_one to Course Tag Family | yes | none | stored, indexed | The owning family. Deletion behaviour: cascade. |
| `group_sequence` | Group sequence | integer | — | the family's sequence | derived from `group_id.sequence`, stored, read-only, indexed | Denormalised ordering key. |
| `channel_ids` | Channels | many_to_many to Course | no | empty | stored | The courses carrying this tag; association table `slide_channel_tag_rel`. |
| `color` | Color Index | integer | no | a random whole number between 1 and 11 inclusive | stored | Help text: "Tag color used in both backend and website. No color means no display in kanban or front-end, to distinguish internal tags from public categorization tags". A tag whose colour is empty or zero is an internal tag and never appears on the public pages. |

---

# 17. Course Tag Family

**Full name:** Course Tag Family (dictionary name Channel/Course Groups). **Transport name**
`slide.channel.tag.group`, **storage name** `slide_channel_tag_group`. **Kind:** persistent entity.
**Contributed by** `website_slides`. **Publishable.** Generated reference page:
[`slide.channel.tag.group`](../../references/entities/slide.channel.tag.group.md).

**Default ordering:** by `sequence` ascending. New families are published by default; only published
families appear in the public filter panel.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Group Name | text (translatable) | yes | none | stored | The family name. |
| `sequence` | Sequence | integer | yes | 10 | stored, indexed | Order among families. |
| `tag_ids` | Tags | one_to_many to Course Tag | no | empty | stored | The tags of the family; the inverse field is `group_id`. |

---

# 18. Course Invitation Wizard

**Full name:** Course Invitation Wizard (dictionary name Channel Invitation Wizard).
**Transport name** `slide.channel.invite`, **storage name** transient `slide_channel_invite`.
**Kind:** transient entity. **Contributed by** `website_slides`. Generated reference page:
[`slide.channel.invite`](../../references/entities/slide.channel.invite.md).

A composer that adds contacts to a course, either as enrolled attendees or as pending invitations,
and sends each of them the matching message. It carries the shared composer fields. The rendering
model is Course Enrolment, so every placeholder — including the personal invitation link — is
evaluated against the recipient's own enrolment record.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `channel_id` | Course | many_to_one to Course | yes | from the calling context | stored | The course. |
| `partner_ids` | Recipients | many_to_many to Contact | no | empty | stored | The recipients. |
| `enroll_mode` | Enroll partners | boolean | no | from the calling action | stored, read-only | Help text: "Whether invited partners will be added as enrolled. Otherwise, they will be added as invited." |
| `send_email` | Send Email | boolean | — | derived | derived, stored | True unless the course is public and the composer is in invitation mode. When it is false the operation does nothing at all. |
| `attachment_ids` | Attachments | many_to_many to Attachment | no | empty | stored | Files joined to every message. |
| `channel_invite_url` | Course Link | text | — | — | derived | The generic course address shown in the form. |
| `channel_visibility` | Channel Visibility | selection | — | — | derived from `channel_id.visibility` | Mirror. |
| `channel_published` | Channel Published | boolean | — | — | derived from `channel_id.is_published` | Mirror. |

**Validation rules.**

| Trigger | Condition | Message |
|---|---|---|
| Sending | The acting user has no electronic mail address | `Unable to post message, please configure the sender's email address.` |
| Sending | No recipient was chosen | `Please select at least one recipient.` |
| Sending | The acting user may not add attendees to the course | `You are not allowed to add members to this course. Please contact the course responsible or an administrator.` |

---

# Part 3. Forum entities

# 19. Forum

**Full name:** Forum. **Transport name** `forum.forum`, **storage name** `forum_forum`.
**Kind:** persistent entity. **Contributed by** `website_forum`, extended by
`website_slides_forum`. Generated reference page:
[`forum.forum`](../../references/entities/forum.forum.md). **Carries a discussion thread.**
**Carries an image.** **Attached to one site or shared across all of them.** **Archivable.**

A question-and-answer space. Beside its identity and its presentation, a forum is above all a
configuration of the reputation-point economy: eight fields that say how many points an event grants
or removes, and twenty-six fields that say how many points an action requires.

**Default ordering:** by `sequence`, then identifier. **Display name rule:** the `name`.

**Archiving:** supported. Archiving a forum archives every one of its posts, archived ones included;
un-archiving reverses it.

## 19.1 Identity and presentation

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Forum Name | text (translatable) | yes | none | stored | The forum name. |
| `sequence` | Sequence | integer | no | 1 | stored | Order among forums. |
| `active` | Active | boolean | yes | true | stored | Archiving marker. |
| `mode` | Mode | selection | yes | `questions` | stored | `questions` (Questions (1 answer)) or `discussions` (Discussions (multiple answers)). Help text: "Questions mode: only one answer allowed  Discussions mode: multiple answers allowed". |
| `privacy` | Privacy | selection | no | `public` | stored | `public` (Public), `connected` (Signed In), `private` (Some users). Help text: "Public: Forum is public Signed In: Forum is visible for signed in users Some users: Forum and their content are hidden for non members of selected group". An empty value marks a forum whose access is governed by the course it is attached to. |
| `authorized_group_id` | Authorized Group | many_to_one to Access Group | no | none | stored | The group allowed on a private forum. Cleared automatically when the privacy becomes `public` or `connected`. |
| `faq` | Guidelines | rich_text (translatable) | no | rendered from the shipped guidelines page at creation | stored | The guidelines page of the forum. |
| `description` | Description | long_text (translatable) | no | none | stored | Short description shown on the forum list and used by the site search. |
| `welcome_message` | Welcome Message | rich_text (translatable) | no | the shipped welcome text | stored | Banner shown to a first-time visitor; dismissible and remembered in a browser cookie. |
| `image_1920` | Image | image | — | see derivation | derived, stored, editable | The illustration. |
| `default_order` | Default | selection | yes | `last_activity_date desc` | stored | `create_date desc` (Newest), `last_activity_date desc` (Last Updated), `vote_count desc` (Most Voted), `relevancy desc` (Relevance), `child_count desc` (Answered). |
| `relevancy_post_vote` | First Relevance Parameter | decimal | no | 0.8 | stored | Help text: "This formula is used in order to sort by relevance. The variable 'votes' represents number of votes for a post, and 'days' is number of days since the post creation". |
| `relevancy_time_decay` | Second Relevance Parameter | decimal | no | 1.8 | stored | The second parameter of the same formula. |
| `allow_share` | Sharing Options | boolean | no | true | stored | Help text: "After posting the user will be proposed to share its question or answer on social networks, enabling social network propagation of the forum content." |

**Derivation of `image_1920`.** When the forum is attached to a course and carries no image of its
own, the course picture is used.

## 19.2 Contents, tags and statistics

| Field | Full name | Type | Stored | Meaning and rules |
|---|---|---|---|---|
| `post_ids` | Posts | one_to_many to Forum Post | stored | Every post of the forum, questions and answers alike. |
| `last_post_id` | Last Post | many_to_one to Forum Post | derived | The question with the highest identifier among the forum's active questions. |
| `total_posts` | # Posts | integer | derived | The count of questions — posts with no parent — whose state is `active` or `close`. |
| `total_views` | # Views | integer | derived | The sum of the view counters of those questions. |
| `total_answers` | # Answers | integer | derived | The sum of the answer counts of those questions. |
| `total_favorites` | # Favorites | integer | derived | The number of those questions that have at least one bookmark. |
| `count_posts_waiting_validation` | Number of posts waiting for validation | integer | derived | The count of posts in state `pending`. |
| `count_flagged_posts` | Number of flagged posts | integer | derived | The count of posts in state `flagged`. |
| `has_pending_post` | Has pending post | boolean | derived | True when the reader has at least one question of their own waiting for validation on this forum. |
| `can_moderate` | Is a moderator | boolean | derived | True when the reader's balance reaches `karma_moderate`. |
| `tag_ids` | Tags | one_to_many to Forum Tag | stored | The tags of the forum. |
| `tag_most_used_ids` | Most used tags | one_to_many to Forum Tag | derived | The five tags with the highest post count, ordered by post count descending, then name, then identifier. |
| `tag_unused_ids` | Unused tags | one_to_many to Forum Tag | derived | The tags whose post count is zero or empty. |
| `slide_channel_ids` | Courses | one_to_many to Course | stored | Added by the course-forum package. Help text: "Edit the course linked to this forum on the course form." |
| `slide_channel_id` | Course | many_to_one to Course | derived, stored | Added by the course-forum package. The first attached course. |
| `visibility` | Visibility | selection | derived from `slide_channel_id.visibility` | Added by the course-forum package. Help text: "Forum linked to a Course, the visibility is the one applied on the course." |

## 19.3 Reputation-point grants

Each of these says how many points move when the named event happens. The sign is used as stored, so
a negative default removes points.

| Field | Full name | Default | Event and beneficiary |
|---|---|---|---|
| `karma_gen_question_new` | Asking a question | 2 | The author asks a question that becomes active — at creation for a trusted author, at validation otherwise. |
| `karma_gen_question_upvote` | Question upvoted | 5 | Somebody upvotes the author's question; the author receives it. |
| `karma_gen_question_downvote` | Question downvoted | −2 | Somebody downvotes the author's question; the author receives it. |
| `karma_gen_answer_upvote` | Answer upvoted | 10 | Somebody upvotes the author's answer; the author receives it. |
| `karma_gen_answer_downvote` | Answer downvoted | −2 | Somebody downvotes the author's answer; the author receives it. |
| `karma_gen_answer_accept` | Accepting an answer | 2 | The acting user accepts somebody else's answer; the **acting user** receives it. |
| `karma_gen_answer_accepted` | Answer accepted | 15 | The author's answer is accepted by somebody else; the answer's author receives it. |
| `karma_gen_answer_flagged` | Answer flagged | −100 | The author's post is closed as offensive, closed as spam, or marked offensive; the author receives it. |

## 19.4 Reputation-point requirements

| Field | Full name | Default | Action gated |
|---|---|---|---|
| `karma_ask` | Ask questions | 3 | Ask a question. |
| `karma_answer` | Answer questions | 3 | Answer a question. |
| `karma_post` | Ask questions without validation | 100 | Publish a question without moderation; below the threshold the question is created in state `pending`. |
| `karma_edit_own` | Edit own posts | 1 | Edit one's own post. |
| `karma_edit_all` | Edit all posts | 300 | Edit anybody's post. |
| `karma_edit_retag` | Change question tags | 75 | Change the tags of a post. |
| `karma_close_own` | Close own posts | 100 | Close or reopen one's own post. |
| `karma_close_all` | Close all posts | 500 | Close or reopen anybody's post. |
| `karma_unlink_own` | Delete own posts | 500 | Archive, un-archive or delete one's own post. |
| `karma_unlink_all` | Delete all posts | 1000 | Archive, un-archive or delete anybody's post. |
| `karma_answer_accept_own` | Accept an answer on own questions | 20 | Accept an answer on a question one asked. |
| `karma_answer_accept_all` | Accept an answer to all questions | 500 | Accept an answer on anybody's question. |
| `karma_comment_own` | Comment own posts | 1 | Comment one's own post. |
| `karma_comment_all` | Comment all posts | 1 | Comment anybody's post. |
| `karma_comment_convert_own` | Convert own comments to answers | 50 | Convert one's own comment into an answer, or one's own answer into a comment. |
| `karma_comment_convert_all` | Convert all comments to answers | 500 | Convert anybody's comment or answer. |
| `karma_comment_unlink_own` | Delete own comments | 50 | Delete one's own comment. |
| `karma_comment_unlink_all` | Delete all comments | 500 | Delete anybody's comment. |
| `karma_upvote` | Upvote | 5 | Cast an upvote. |
| `karma_downvote` | Downvote | 50 | Cast a downvote. |
| `karma_tag_create` | Create new tags | 30 | Create a new tag. |
| `karma_flag` | Flag a post as offensive | 500 | Flag a post. |
| `karma_moderate` | Moderate posts | 1000 | Validate, refuse, handle the flagged queue and mark posts offensive. |
| `karma_editor` | Editor Features: image and links | 30 | Post pictures and links at all. |
| `karma_dofollow` | Nofollow links | 500 | Post links that search engines may follow. Help text: "If the author has not enough karma, a nofollow attribute is added to links". |
| `karma_user_bio` | Display detailed user biography | 750 | Have one's biography displayed beside one's posts. |

## 19.5 Lifecycle

1. **Creation.** The guidelines page is rendered from the shipped template and stored, and the forum
   counter of every site is refreshed.
2. **Writing `privacy`.** Setting it to `public` or `connected` clears `authorized_group_id`.
3. **Writing `active`.** Every post of the forum takes the new value.
4. **Writing `active` or the site link.** The forum counter of every site is refreshed.
5. **Deletion.** The forum counter of every site is refreshed; posts, tags and votes cascade.

---

# 20. Forum Post

**Full name:** Forum Post. **Transport name** `forum.post`, **storage name** `forum_post`.
**Kind:** persistent entity. **Contributed by** `website_forum`. Generated reference page:
[`forum.post`](../../references/entities/forum.post.md). **Carries a discussion thread.**
**Archivable.**

A question or an answer. A post with no parent is a question; a post with a parent is an answer to
that question. The hierarchy is one level deep.

**Default ordering:** by `is_correct` descending, then `vote_count` descending, then
`last_activity_date` descending. The accepted answer therefore always comes first.

**Display name rule:** the `name`.

**Archiving:** supported. Archiving a question archives its answers; un-archiving reverses it.
Archiving is the operation the interface calls deleting.

## 20.1 Content, hierarchy and state

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Title | text | no | none | stored | yes | The title. An answer created through the public pages receives `Re: <question title>`. |
| `forum_id` | Forum | many_to_one to Forum | yes | none | stored, indexed | yes | The forum. Writing this field requires write access on the target forum. |
| `content` | Content | rich_text | no | none | stored | yes | The body; rewritten on create and on write by the link and picture rules of section 20.5. |
| `plain_content` | Plain Content | long_text | — | — | derived, stored | no | The body stripped of its markup, used as the page description and by the site search. |
| `parent_id` | Question | many_to_one to Forum Post | no | none | stored, read-only, indexed | yes | The question this post answers. Empty on a question. Deletion behaviour: cascade. |
| `child_ids` | Post Answers | one_to_many to Forum Post | — | — | stored | no | The answers, restricted to the same forum. |
| `child_count` | Answers | integer | — | — | derived, stored | no | The number of answers. |
| `state` | Status | selection | no | `active` | stored | yes | `active` (Active), `pending` (Waiting Validation), `close` (Closed), `offensive` (Offensive), `flagged` (Flagged). See [`state-machines.md`](state-machines.md#machine-3). |
| `active` | Active | boolean | yes | true | stored | yes | Archiving marker. |
| `views` | Views | integer | no | 0 | stored, read-only | no | How many times the question page was opened; incremented without taking a lock. |
| `last_activity_date` | Last activity on | datetime | yes | the current instant | stored, read-only | yes | Help text: "Field to keep track of a post's last activity. Updated whenever it is replied to, or when a comment is added on the post or one of its replies." |
| `tag_ids` | Tags | many_to_many to Forum Tag | no | empty | stored | yes | Labels; association table `forum_tag_rel`. |
| `relevancy` | Relevance | decimal | — | — | derived, stored | no | The ranking score of the relevance ordering; see [`calculations.md`](calculations.md#lsg-calc-020). |
| `website_url` | Website uniform resource locator | text | — | — | derived | no | `/forum/<forum slug>/<post slug>`; for an answer, the address of the question followed by the anchor `#answer_<identifier>`. |
| `website_id` | Website | many_to_one to Site | — | — | derived from `forum_id.website_id`, read-only | no | Mirror. |
| `website_message_ids` | Website Message | one_to_many to Message | — | — | stored | no | The public messages of the post: those of type electronic mail, comment or outgoing electronic mail. |

## 20.2 Votes, bookmarks and acceptance

| Field | Full name | Type | Stored | Meaning and rules |
|---|---|---|---|---|
| `vote_ids` | Votes | one_to_many to Post Vote | stored | The votes cast on this post. |
| `vote_count` | Total Votes | integer | derived, stored | The sum of the vote values: upvotes minus downvotes; cancelled votes count zero. |
| `user_vote` | My Vote | integer | derived | The reader's own vote value on this post. |
| `favourite_ids` | Favourite | many_to_many to User | stored, copied | The users who bookmarked this question. |
| `favourite_count` | Favorite | integer | derived, stored | The number of those users. |
| `user_favourite` | Is Favourite | boolean | derived | Whether the reader bookmarked this question. |
| `is_correct` | Correct | boolean | stored, copied | Help text: "Correct answer or answer accepted". Only one answer per question may carry it. |
| `has_validated_answer` | Is answered | boolean | derived, stored | Whether any answer of this question is accepted. |
| `self_reply` | Reply to own question | boolean | derived, stored | Whether the author of this answer also asked the question. |
| `uid_has_answered` | Has Answered | boolean | derived | Whether the reader already answered this question. |

## 20.3 Moderation data

| Field | Full name | Type | Stored | Meaning and rules |
|---|---|---|---|---|
| `flag_user_id` | Flagged by | many_to_one to User | stored, copied | Who flagged the post. |
| `moderator_id` | Reviewed by | many_to_one to User | stored, read-only, copied | Who validated, refused or marked the post offensive. |
| `closed_reason_id` | Reason | many_to_one to Post Closing Reason | stored, not copied | Why the post was closed or marked offensive. |
| `closed_uid` | Closed by | many_to_one to User | stored, read-only, not copied | Who closed it. |
| `closed_date` | Closed on | datetime | stored, read-only, not copied | When it was closed. |

## 20.4 Reader-dependent permission fields

All of these depend on the reader, are derived and are never stored. They are the single source of
truth for every guard in the forum. An administrator satisfies every one of them.

| Field | Full name | Value |
|---|---|---|
| `karma_accept` | Convert comment to answer | `karma_answer_accept_own` of the forum when the reader asked the parent question, otherwise `karma_answer_accept_all`. |
| `karma_edit` | Karma to edit | `karma_edit_own` when the reader wrote this post, otherwise `karma_edit_all`. |
| `karma_close` | Karma to close | `karma_close_own` when the reader wrote this post, otherwise `karma_close_all`. |
| `karma_unlink` | Karma to unlink | `karma_unlink_own` when the reader wrote this post, otherwise `karma_unlink_all`. |
| `karma_comment` | Karma to comment | `karma_comment_own` when the reader wrote this post, otherwise `karma_comment_all`. |
| `karma_comment_convert` | Karma to convert comment to answer | `karma_comment_convert_own` when the reader wrote this post, otherwise `karma_comment_convert_all`. |
| `karma_flag` | Flag a post as offensive | `karma_flag` of the forum. |
| `can_ask` | Can Ask | The reader holds at least `karma_ask`. |
| `can_answer` | Can Answer | The reader holds at least `karma_answer`. |
| `can_accept` | Can Accept | The reader holds at least `karma_accept`. |
| `can_edit` | Can Edit | The reader holds at least `karma_edit`. |
| `can_close` | Can Close | The reader holds at least `karma_close`. |
| `can_unlink` | Can Unlink | The reader holds at least `karma_unlink`. |
| `can_upvote` | Can Upvote | The reader holds at least `karma_upvote`, **or** the reader's current vote on this post is minus one — cancelling one's own downvote is always allowed. |
| `can_downvote` | Can Downvote | The reader holds at least `karma_downvote`, **or** the reader's current vote on this post is plus one — cancelling one's own upvote is always allowed. |
| `can_comment` | Can Comment | The reader holds at least `karma_comment`. |
| `can_comment_convert` | Can Convert to Comment | The reader holds at least `karma_comment_convert`. |
| `can_view` | Can View | `can_close` is true, **or** the post is active and either its author's balance is strictly positive or the reader is the author. This is what hides the output of an author whose balance fell to zero or below. Searchable. |
| `can_display_biography` | Is the author's biography visible from his post | The author holds at least `karma_user_bio` and the author's public profile is published. |
| `can_post` | Can Automatically be Validated | The reader holds at least `karma_post`. |
| `can_flag` | Can Flag | The reader holds at least `karma_flag`. |
| `can_moderate` | Can Moderate | The reader holds at least `karma_moderate`. |
| `can_use_full_editor` | Can Use Full Editor | The reader holds at least `karma_editor`. |

Searching on `can_view` resolves to: the posts the reader wrote whose forum's own-close threshold is
at most the reader's balance, plus the posts of other authors whose forum's all-close threshold is
at most the reader's balance, plus the posts whose author's balance is strictly positive and which
are active or authored by the reader. For an administrator the search matches every post.

## 20.5 Content rewriting rules applied on create and on write

1. When the author's balance is below `karma_dofollow`, every anchor in the content is rewritten so
   its address is preserved and a no-follow marker is added.
2. When the author's balance is below `karma_editor` and the content contains a picture element, an
   anchor with an address, or an inline style loading a background picture from an address, the
   write is refused with `%d karma required to post an image or link.`

## 20.6 Validation rules

| Trigger | Condition | Message |
|---|---|---|
| Writing `parent_id` | The parent chain is recursive | `You cannot create recursive forum posts.` |
| Creating an answer | The parent question is closed or archived | `Posting answer on a [Deleted] or [Closed] question is not possible.` |
| Creating a question | `can_ask` is false | `%d karma required to create a new question.` |
| Creating an answer | `can_answer` is false | `%d karma required to answer a question.` |
| Writing `state` to `active` or `close` | `can_close` is false | `%d karma required to close or reopen a post.` |
| Writing `state` to `flagged` | `can_flag` is false | `%d karma required to flag a post.` |
| Writing `active` | `can_unlink` is false | `%d karma required to delete or reactivate a post.` |
| Writing `is_correct` | `can_accept` is false | `%d karma required to accept or refuse an answer.` |
| Writing `tag_ids` with a different set | The reader's balance is below `karma_edit_retag` | `%d karma required to retag.` |
| Writing any field outside the trusted set | `can_edit` is false | `%d karma required to edit a post.` |
| Deleting | `can_unlink` is false | `%d karma required to unlink a post.` |
| Validating a pending post | `can_moderate` is false | `%d karma required to validate a post.` |
| Refusing a pending post | `can_moderate` is false | `%d karma required to refuse a post.` |
| Flagging | `can_flag` is false | `%d karma required to flag a post.` |
| Marking offensive | `can_moderate` is false | `%d karma required to mark a post as offensive.` |
| Converting an answer into a comment | `can_comment_convert` is false | `%d karma required to convert an answer to a comment.` |
| Converting one's own comment into an answer | The balance is below the own threshold and that threshold is lower than the all threshold | `%d karma required to convert your comment to an answer.` |
| Converting somebody else's comment into an answer | The balance is below the all threshold | `%d karma required to convert a comment to an answer.` |
| Deleting a comment | The balance is below the applicable comment-deletion threshold | `%d karma required to delete a comment.` |
| Posting a comment | `can_comment` is false | `%d karma required to comment.` |

In every message above the placeholder is the numeric value of the applicable threshold of the
forum, printed as a whole number.

**Trusted keys.** When `state` is written to `active` or `close`, the write is additionally allowed
to carry `closed_uid`, `closed_date` and `closed_reason_id`; when `state` is written to `flagged`,
it is additionally allowed to carry `flag_user_id`. Any other field in the same write is subject to
the edit threshold.

## 20.7 Lifecycle

1. **Creation.** The content is rewritten by the two rules of section 20.5. Then, per post: an
   answer to a closed or archived question is refused; a question requires the ask threshold; an
   answer requires the answer threshold; a question whose author is below the publish threshold is
   forced into state `pending`; a question that stays `active` grants `karma_gen_question_new` to
   its author with the reason `Ask a new question`. The state notification is then posted.
2. **Writing.** The guards run per post in the order of the table above. After the write, a change
   of the title or of the content posts a notification — `Answer Edited` on the parent question for
   an answer, `Question Edited` on the question itself — and a change of the archiving marker is
   propagated to every answer.
3. **Acceptance.** Writing `is_correct` moves points unless the reader is the author of that answer:
   the answer's author gains or loses `karma_gen_answer_accepted` with the reason `User answer
   accepted` or `Accepted answer removed`, and the acting user gains or loses
   `karma_gen_answer_accept` with the reason `Validate an answer` or `Remove validated answer`.
4. **Deletion.** Refused when the reader lacks the delete threshold. When the deleted post was the
   accepted answer, `karma_gen_answer_accepted` is removed from its author with the reason `The
   accepted answer is deleted` and the same amount is removed from the acting user with the reason
   `Delete the accepted answer`.
5. **Viewing.** Opening a question page increments `views` by one without taking a lock.

---

# 21. Post Vote

**Full name:** Post Vote. **Transport name** `forum.post.vote`, **storage name**
`forum_post_vote`. **Kind:** persistent entity. **Contributed by** `website_forum`.
Generated reference page:
[`forum.post.vote`](../../references/entities/forum.post.vote.md).

One user's vote on one post.

**Default ordering:** by `create_date` descending, then identifier descending.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `post_id` | Post | many_to_one to Forum Post | yes | none | stored, indexed | The post voted on. Deletion behaviour: cascade. |
| `user_id` | User | many_to_one to User | yes | the acting user | stored | Who voted. A non-administrator may neither set nor change it, neither through the values nor through a default inherited from the calling context. Deletion behaviour: cascade. |
| `vote` | Vote | selection | yes | `1` | stored | `1`, `-1` and `0`, each labelled with its own value. Zero means the vote was cancelled; the record is kept rather than deleted. |
| `create_date` | Create Date | datetime | — | — | stored, read-only, indexed | When the vote was cast. |
| `forum_id` | Forum | many_to_one to Forum | — | the post's forum | derived from `post_id.forum_id`, stored, editable, indexed | Denormalised. |
| `recipient_id` | To | many_to_one to User | — | the post's author | derived from `post_id.create_uid`, stored, editable | The user whose balance moves. A non-administrator may neither set nor change it. |

**Database constraint `_vote_uniq`:** unique(`post_id`, `user_id`); the refusal is `Vote already
exists!`

**Validation rules.**

| Trigger | Condition | Message |
|---|---|---|
| Creating or writing | The acting user is the author of the post | `It is not allowed to vote for its own post.` |
| Creating or writing | The vote belongs to another user | `It is not allowed to modify someone else's vote.` |
| Creating or writing an upvote | `can_upvote` on the post is false | `%d karma required to upvote.` |
| Creating or writing a downvote | `can_downvote` on the post is false | `%d karma required to downvote.` |

**Lifecycle.** At creation, the two general checks and the reputation check run, then the movement
from the previous value `0` to the new value is applied. On a write of the value, the direction
checked is: upvote when the new value is `1`; when the new value is `0`, upvote exactly when the old
value was `-1`, so cancelling a downvote is treated as an upvote for the permission check. The
movement is then applied from the old value to the new one; see
[`calculations.md`](calculations.md#lsg-calc-021).

---

# 22. Post Closing Reason

**Full name:** Post Closing Reason. **Transport name** `forum.post.reason`, **storage name**
`forum_post_reason`. **Kind:** persistent entity. **Contributed by** `website_forum`.
Generated reference page:
[`forum.post.reason`](../../references/entities/forum.post.reason.md).

A reusable reason attached to a closed or offensive post.

**Default ordering:** by `name`.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Closing Reason | text (translatable) | yes | none | stored | The reason text shown in the closing dialogue. |
| `reason_type` | Reason Type | selection | no | `basic` | stored | `basic` (Basic) reasons are proposed when closing a post; `offensive` (Offensive) reasons are proposed when marking a post offensive. |

The thirteen shipped reasons are listed in [`configuration.md`](configuration.md#shipped-reasons).
Two of them are special-cased by the closing and reopening logic.

---

# 23. Forum Tag

**Full name:** Forum Tag. **Transport name** `forum.tag`, **storage name** `forum_tag`.
**Kind:** persistent entity. **Contributed by** `website_forum`. Generated reference page:
[`forum.tag`](../../references/entities/forum.tag.md). **Carries a discussion thread**, so a member
may follow a tag and receive the questions carrying it.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Name | text | yes | none | stored | The label. |
| `forum_id` | Forum | many_to_one to Forum | yes | none | stored, indexed | The owning forum. A tag never spans two forums. |
| `color` | Color | integer | no | 0 | stored | Decoration colour. |
| `post_ids` | Posts | many_to_many to Forum Post | no | empty | stored | The active posts carrying this tag; association table `forum_tag_rel`. |
| `posts_count` | Number of Posts | integer | — | — | derived, stored | The number of active, non-archived posts carrying this tag. |
| `website_url` | Link to questions with the tag | text | — | — | derived | The address of the question list filtered on this tag. |

**Database constraint `_name_uniq`:** unique(`name`, `forum_id`); the refusal is `Tag name already
exists!` The same name may therefore exist on two forums as two distinct records.

**Validation rule.** Creating a tag when the reader's balance is below the forum's
`karma_tag_create` is refused with `%d karma required to create a new Tag.`

**Lifecycle.** Tags typed by an author on the post form are resolved by the forum: an entry that
begins with an underscore is a new label whose name follows the underscore; when a tag of that name
already exists on the forum it is reused; otherwise a new tag is created, but only when the author's
balance reaches the tag-creation threshold and the name is non-empty — below the threshold the entry
is dropped silently. Entries that do not begin with an underscore are existing tag identifiers and
are kept as they are.

---

# Part 4. Recognition entities

# 24. Badge

**Full name:** Gamification Badge. **Transport name** `gamification.badge`, **storage name**
`gamification_badge`. **Kind:** persistent entity. **Contributed by** `gamification`, extended by
`hr_gamification`, `survey` and `website_profile`. Generated reference page:
[`gamification.badge`](../../references/entities/gamification.badge.md).
**Carries a discussion thread.** **Carries an image.** **Publishable.** **Archivable.**

A distinction that can be granted by a person or by a challenge.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Badge | text (translatable) | yes | none | stored | The badge name. |
| `active` | Active | boolean | yes | true | stored | Archiving marker. |
| `description` | Description | rich_text (translatable) | no | none | stored | What the badge rewards. |
| `level` | Forum Badge Level | selection | no | `bronze` | stored | `bronze` (Bronze), `silver` (Silver), `gold` (Gold). Drives the three per-level counters on a user. |
| `rule_auth` | Allowance to Grant | selection | yes | `everyone` | stored | `everyone` (Everyone), `users` (A selected list of users), `having` (People having some badges), `nobody` (No one, assigned through challenges). Help text: "Who can grant this badge". |
| `rule_auth_user_ids` | Authorized Users | many_to_many to User | no | empty | stored | Association table `rel_badge_auth_users`. Help text: "Only these people can give this badge". |
| `rule_auth_badge_ids` | Required Badges | many_to_many to Badge | no | empty | stored | Association table `gamification_badge_rule_badge_rel`. Help text: "Only the people having these badges can give this badge". |
| `rule_max` | Monthly Limited Sending | boolean | no | false | stored | Help text: "Check to set a monthly limit per person of sending this badge". |
| `rule_max_number` | Limitation Number | integer | no | 0 | stored | Help text: "The maximum number of time this badge can be sent per month per person." |
| `challenge_ids` | Reward of Challenges | one_to_many to Challenge | — | — | stored | The challenges that award this badge to every succeeding participant. |
| `goal_definition_ids` | Rewarded by | many_to_many to Goal Definition | no | empty | stored | Association table `badge_unlocked_definition_rel`. Help text: "The users that have succeeded these goals will receive automatically the badge." |
| `owner_ids` | Owners | one_to_many to User Badge | — | — | stored | Help text: "The list of instances of this badge granted to users". |
| `granted_count` | Total | integer | — | — | derived | Help text: "The number of time this badge has been received." |
| `granted_users_count` | Number of users | integer | — | — | derived | Help text: "The number of time this badge has been received by unique users." |
| `unique_owner_ids` | Unique Owners | many_to_many to User | — | — | derived | Help text: "The list of unique users having received this badge." |
| `stat_this_month` | Monthly total | integer | — | — | derived | Grants of this badge since the first day of the current month. |
| `stat_my` | My Total | integer | — | — | derived | Grants of this badge received by the reader. |
| `stat_my_this_month` | My Monthly Total | integer | — | — | derived | Grants received by the reader since the first day of the current month. |
| `stat_my_monthly_sending` | My Monthly Sending Total | integer | — | — | derived | Grants **created** by the reader since the first day of the current month. |
| `remaining_sending` | Remaining Sending Allowed | integer | — | — | derived | Zero when the reader may not grant the badge at all; minus one when there is no monthly limit; otherwise `rule_max_number` minus `stat_my_monthly_sending`. |
| `granted_employees_count` | Granted Employees Count | integer | — | — | derived | Added by the human-resources package. The number of employees holding the badge. |
| `survey_ids` | Survey Ids | one_to_many to Survey | — | — | stored | Added by the questionnaire package. The questionnaires that name this badge as their certification badge. |
| `survey_id` | Survey | many_to_one to Survey | — | — | derived, stored | Added by the questionnaire package. The single questionnaire of that list. |

**Granting check.** Before a grant is written the badge is checked, in this order. An administrator
passes unconditionally.

| Condition | Message |
|---|---|
| `rule_auth` is `nobody` | `This badge can not be sent by users.` |
| `rule_auth` is `users` and the reader is not in `rule_auth_user_ids` | `You are not in the user allowed list.` |
| `rule_auth` is `having` and the reader lacks one of `rule_auth_badge_ids` | `You do not have the required badges.` |
| `rule_max` is true and `stat_my_monthly_sending` has reached `rule_max_number` | `You have already sent this badge too many time this month.` |

---

# 25. User Badge

**Full name:** Gamification User Badge. **Transport name** `gamification.badge.user`,
**storage name** `gamification_badge_user`. **Kind:** persistent entity. **Contributed by**
`gamification`, extended by `hr_gamification`. Generated reference page:
[`gamification.badge.user`](../../references/entities/gamification.badge.user.md).
**Carries a discussion thread.**

One grant of one badge to one user.

**Default ordering:** by `create_date` descending. **Display name rule:** the `badge_name`.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `user_id` | User | many_to_one to User | yes | none | stored, indexed | Who received the badge. Deletion behaviour: cascade. |
| `user_partner_id` | User Partner | many_to_one to Contact | — | — | derived from `user_id.partner_id` | The contact notified. |
| `sender_id` | Sender | many_to_one to User | no | none | stored | Who sent it, when it was granted by hand. |
| `badge_id` | Badge | many_to_one to Badge | yes | none | stored, indexed | The badge. Deletion behaviour: cascade. |
| `challenge_id` | Challenge | many_to_one to Challenge | no | none | stored | The challenge that awarded it, when it was awarded automatically. |
| `comment` | Comment | long_text | no | none | stored | A free note from the sender. |
| `badge_name` | Badge Name | text | — | — | derived from `badge_id.name`, writable | The badge name. |
| `level` | Badge Level | selection | — | — | derived from `badge_id.level`, stored, read-only | Mirror. |
| `employee_id` | Employee | many_to_one to Employee | no | none | stored, indexed | Added by the human-resources package. |
| `has_edit_delete_access` | Has Edit Delete Access | boolean | — | — | derived | Added by the human-resources package. True when the reader may edit or delete this grant. |

**Validation rule.** Added by the human-resources package: when `employee_id` is set and does not
correspond to `user_id`, the save is refused with `The selected employee does not correspond to the
selected user.`

**Lifecycle.** Creating a grant first runs the granting check of the badge. Once created, the grant
notifies the beneficiary's contact with the shipped badge message, whose subject is `🎉 You've
earned the <badge name> badge!` and whose body is rendered from the badge-received template. The
notification carries no button to open the record.

---

# 26. Badge Granting Wizard

**Full name:** Gamification User Badge Wizard. **Transport name**
`gamification.badge.user.wizard`, **storage name** transient `gamification_badge_user_wizard`.
**Kind:** transient entity. **Contributed by** `gamification`, extended by `hr_gamification`.
Generated reference page:
[`gamification.badge.user.wizard`](../../references/entities/gamification.badge.user.wizard.md).

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `user_id` | User | many_to_one to User | yes | none | derived, stored | The beneficiary. When the human-resources package is installed the value is derived from the chosen employee. |
| `badge_id` | Badge | many_to_one to Badge | yes | from the calling context | stored | The badge to grant. |
| `comment` | Comment | long_text | no | none | stored | The note written on the grant. |
| `employee_id` | Employee | many_to_one to Employee | no | from the calling context | stored | Added by the human-resources package. |

**Validation rules.** Granting a badge to oneself is refused with `You can not grant a badge to
yourself.` from the general dialogue and with `You can not send a badge to yourself.` from the
employee dialogue.

---

# 27. Challenge

**Full name:** Gamification Challenge. **Transport name** `gamification.challenge`,
**storage name** `gamification_challenge`. **Kind:** persistent entity. **Contributed by**
`gamification`, extended by `survey`, `website_slides` and `website_forum`. Generated reference
page: [`gamification.challenge`](../../references/entities/gamification.challenge.md).
**Carries a discussion thread.**

A set of goals assigned to a population, with a periodicity, rewards and a report schedule.

**Default ordering:** by `end_date`, then `start_date`, then `name`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Copied | Meaning and rules |
|---|---|---|---|---|---|---|---|
| `name` | Challenge Name | text (translatable) | yes | none | stored | yes | The challenge name. |
| `description` | Description | long_text (translatable) | no | none | stored | yes | What the challenge asks for. |
| `state` | State | selection | yes | `draft` | stored | no | `draft` (Draft), `inprogress` (In Progress), `done` (Done). Changes are written into the discussion thread. See [`state-machines.md`](state-machines.md#machine-6). |
| `manager_id` | Responsible | many_to_one to User | no | the acting user | stored | yes | The person answerable for the challenge. |
| `user_ids` | Participants | many_to_many to User | no | see below | stored | yes | Association table `gamification_challenge_users_rel`. |
| `user_domain` | User domain | text | no | the internal-user population | stored | yes | A stored filter that selects the participants. When it is written, the users it selects are added to `user_ids`. The default proposed on a new record selects the active internal users. |
| `user_count` | # Users | integer | — | — | derived | no | The number of **active** users among the participants. |
| `period` | Periodicity | selection | yes | `once` | stored | yes | `once` (Non recurring), `daily` (Daily), `weekly` (Weekly), `monthly` (Monthly), `yearly` (Yearly). Help text: "Period of automatic goal assignment. If none is selected, should be launched manually." |
| `start_date` | Start Date | date | no | none | stored | yes | Help text: "The day a new challenge will be automatically started. If no periodicity is set, will use this date as the goal start date." |
| `end_date` | End Date | date | no | none | stored | yes | Help text: "The day a new challenge will be automatically closed. If no periodicity is set, will use this date as the goal end date." |
| `invited_user_ids` | Suggest to users | many_to_many to User | no | empty | stored | yes | Association table `gamification_invited_user_ids_rel`. Users to whom the challenge is proposed; they accept or discard it. |
| `line_ids` | Lines | one_to_many to Challenge Line | yes | none | stored | yes | Help text: "List of goals that will be set". At least one line is required. |
| `reward_id` | For Every Succeeding User | many_to_one to Badge | no | none | stored, indexed | yes | Awarded to every participant who reaches every goal. |
| `reward_first_id` | For 1st user | many_to_one to Badge | no | none | stored | yes | Awarded to the best participant at the end. |
| `reward_second_id` | For 2nd user | many_to_one to Badge | no | none | stored | yes | Awarded to the second best. |
| `reward_third_id` | For 3rd user | many_to_one to Badge | no | none | stored | yes | Awarded to the third best. |
| `reward_failure` | Reward Bests if not Succeeded? | boolean | no | false | stored | yes | When true, the top three badges may go to participants who did not reach every goal. |
| `reward_realtime` | Reward as soon as every goal is reached | boolean | no | true | stored | yes | Help text: "With this option enabled, a user can receive a badge only once. The top 3 badges are still rewarded only at the end of the challenge." |
| `visibility_mode` | Display Mode | selection | yes | `personal` | stored | yes | `personal` (Individual Goals) or `ranking` (Leader Board (Group Ranking)). |
| `report_message_frequency` | Report Frequency | selection | yes | `never` | stored | yes | `never` (Never), `onchange` (On change), `daily` (Daily), `weekly` (Weekly), `monthly` (Monthly), `yearly` (Yearly). |
| `report_message_group_id` | Send a copy to | many_to_one to Discussion Channel | no | none | stored | yes | Help text: "Group that will receive a copy of the report in addition to the user". |
| `report_template_id` | Report Template | many_to_one to Message Template | yes | the shipped challenge-report template | stored | yes | The template used for the periodic report. |
| `remind_update_delay` | Non-updated manual goals will be reminded after | integer | no | 0 | stored | yes | Help text: "Never reminded if no value or zero is specified." Copied onto every goal generated by the challenge. |
| `last_report_date` | Last Report Date | date | no | today | stored | yes | When the last report went out. |
| `next_report_date` | Next Report Date | date | — | — | derived, stored | yes | `last_report_date` plus one day, seven days, one month or one year according to `report_message_frequency`; empty when the frequency is `never` or `onchange`. |
| `challenge_category` | Appears in | selection | yes | `hr` | stored | yes | `hr` (Human Resources / Engagement), `other` (Settings / Gamification Tools), and, added by other packages, `certification` (Certifications), `slides` (Website / Slides) and `forum` (Website / Forum). Help text: "Define the visibility of the challenge through menus". When the forum package is removed, records holding `forum` fall back to the default `hr`. |

**Validation rules.**

| Trigger | Condition | Message |
|---|---|---|
| Writing `state` to `draft` | At least one goal of the challenge is still in state `inprogress` | `You can not reset a challenge with unfinished goals.` |
| Asking for the progress of a personal challenge without naming a user | — | `Retrieving progress for personal challenge without user information` |

**Lifecycle.** Writing `state` to `inprogress` recomputes the participants from the stored filter
and generates the missing goals. Writing it to `done` forces the reward check. Writing it back to
`draft` is refused while goals are running. The daily job, the goal generation and the reward
distribution are specified in [`gamification.md`](gamification.md#challenge-cycle).

---

# 28. Challenge Line

**Full name:** Challenge Line (dictionary name Gamification generic goal for challenge).
**Transport name** `gamification.challenge.line`, **storage name** `gamification_challenge_line`.
**Kind:** persistent entity. **Contributed by** `gamification`. Generated reference page:
[`gamification.challenge.line`](../../references/entities/gamification.challenge.line.md).

**Default ordering:** by `sequence`, then identifier.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `challenge_id` | Challenge | many_to_one to Challenge | yes | none | stored, indexed | The owning challenge. Deletion behaviour: cascade. |
| `definition_id` | Goal Definition | many_to_one to Goal Definition | yes | none | stored | What is measured. Deletion behaviour: cascade. |
| `sequence` | Sequence | integer | no | 1 | stored | Order among the lines. |
| `target_goal` | Target Value to Reach | decimal | yes | none | stored | The value each participant must reach. |
| `name` | Name | text | — | — | derived from `definition_id.name` | Mirror. |
| `condition` | Condition | selection | — | — | derived from `definition_id.condition`, read-only | Mirror. |
| `definition_suffix` | Unit | text | — | — | derived from `definition_id.suffix`, read-only | Mirror. |
| `definition_monetary` | Monetary | boolean | — | — | derived from `definition_id.monetary`, read-only | Mirror. |
| `definition_full_suffix` | Suffix | text | — | — | derived from `definition_id.full_suffix`, read-only | Mirror. |

---

# 29. Goal

**Full name:** Gamification Goal. **Transport name** `gamification.goal`, **storage name**
`gamification_goal`. **Kind:** persistent entity. **Contributed by** `gamification`.
Generated reference page:
[`gamification.goal`](../../references/entities/gamification.goal.md).

One user's measurement against one definition over one period.

**Default ordering:** by `start_date` descending, then `end_date` descending, then `definition_id`,
then identifier. **Display name rule:** the definition name.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `definition_id` | Goal Definition | many_to_one to Goal Definition | yes | none | stored | What is measured. Deletion behaviour: cascade. |
| `user_id` | User | many_to_one to User | yes | none | stored, indexed | Whose measurement this is. Deletion behaviour: cascade. Access to this field bypasses the usual search restriction so a ranking can name every participant. |
| `user_partner_id` | User Partner | many_to_one to Contact | — | — | derived from `user_id.partner_id` | The contact notified by a reminder. |
| `line_id` | Challenge Line | many_to_one to Challenge Line | no | none | stored | The line that generated this goal. Deletion behaviour: cascade. |
| `challenge_id` | Challenge | many_to_one to Challenge | — | — | derived from `line_id.challenge_id`, stored, read-only, indexed | Help text: "Challenge that generated the goal, assign challenge to users to generate goals with a value in this field." |
| `start_date` | Start Date | date | no | today | stored | Start of the measured period; empty means always active. |
| `end_date` | End Date | date | no | none | stored | End of the measured period; empty means always active. |
| `target_goal` | To Reach | decimal | yes | none | stored | The value to reach. |
| `current` | Current Value | decimal | yes | 0 | stored | The measured value. |
| `completeness` | Completeness | decimal | — | — | derived | The percentage of completion; see [`calculations.md`](calculations.md#lsg-calc-030). |
| `state` | State | selection | yes | `draft` | stored | `draft` (Draft), `inprogress` (In progress), `reached` (Reached), `failed` (Failed), `canceled` (Cancelled). See [`state-machines.md`](state-machines.md#machine-5). |
| `to_update` | To update | boolean | no | false | stored | Set when a reminder was sent for a manually kept goal. |
| `closed` | Closed goal | boolean | no | false | stored | Set when the goal failed; a closed goal is skipped by every later update. |
| `computation_mode` | Computation Mode | selection | — | — | derived from `definition_id.computation_mode`, writable | Mirror. |
| `color` | Color Index | integer | — | — | derived | Two when the goal failed after its end date, five when it was reached after its end date, zero otherwise. |
| `remind_update_delay` | Remind delay | integer | no | 0 | stored | Help text: "The number of days after which the user assigned to a manual goal will be reminded. Never reminded if no value is specified." |
| `last_update` | Last Update | date | — | today at every write | stored | Help text: "In case of manual goal, reminders are sent if the goal as not been updated for a while (defined in challenge). Ignored in case of non-manual goal or goal not linked to a challenge." |
| `definition_description` | Definition Description | long_text | — | — | derived from `definition_id.description`, read-only | Mirror. |
| `definition_condition` | Definition Condition | selection | — | — | derived from `definition_id.condition`, read-only | Mirror. |
| `definition_suffix` | Suffix | text | — | — | derived from `definition_id.full_suffix`, read-only | Mirror. |
| `definition_display` | Display Mode | selection | — | — | derived from `definition_id.display_mode`, read-only | Mirror. |

**Validation rule.** Changing `definition_id` or `user_id` on a goal that is no longer in state
`draft` is refused with `Can not modify the configuration of a started goal`.

**Lifecycle.** Every write stamps `last_update` with today's date. When `current` changes on a goal
whose challenge reports on change, an individual report is sent at once to that goal's user. The
measurement itself is specified in [`gamification.md`](gamification.md#goal-update).

---

# 30. Goal Definition

**Full name:** Gamification Goal Definition. **Transport name** `gamification.goal.definition`,
**storage name** `gamification_goal_definition`. **Kind:** persistent entity. **Contributed by**
`gamification`. Generated reference page:
[`gamification.goal.definition`](../../references/entities/gamification.goal.definition.md).

What to measure, how to measure it, and whether higher or lower is better.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Goal Definition | text (translatable) | yes | none | stored | The definition name. |
| `description` | Goal Description | long_text | no | none | stored | What the goal asks for. |
| `monetary` | Monetary Value | boolean | no | false | stored | Help text: "The target and current value are defined in the company currency." |
| `suffix` | Suffix | text (translatable) | no | none | stored | Help text: "The unit of the target and current values". |
| `full_suffix` | Full Suffix | text | — | — | derived | Help text: "The currency and suffix field". The company currency symbol followed by the suffix when the definition is monetary, otherwise the suffix alone. |
| `computation_mode` | Computation Mode | selection | yes | `manually` | stored | `manually` (Recorded manually), `count` (Automatic: number of records), `sum` (Automatic: sum on a field), `python` (Automatic: execute a specific computation script). Help text: "Define how the goals will be computed. The result of the operation will be stored in the field 'Current'." |
| `display_mode` | Displayed as | selection | yes | `progress` | stored | `progress` (Progressive (using numerical values)) or `boolean` (Exclusive (done or not-done)). |
| `model_id` | Model | many_to_one to Entity Definition | no | none | stored | The entity counted or summed. Deletion behaviour: cascade. |
| `model_inherited_ids` | Model Inherited | many_to_many to Entity Definition | — | — | derived from `model_id.inherited_model_ids` | The entities the measured one inherits from. |
| `field_id` | Field to Sum | many_to_one to Field Definition | no | none | stored | The numeric field summed in `sum` mode. |
| `field_date_id` | Date Field | many_to_one to Field Definition | no | none | stored | Help text: "The date to use for the time period evaluated". Restricted to date and date-and-time fields. |
| `domain` | Filter Domain | text | yes | the empty filter | stored | Help text: "Domain for filtering records. General rule, not user depending, e.g. [('state', '=', 'done')]. The expression can contain reference to 'user' which is a browse record of the current user if not in batch mode." |
| `batch_mode` | Batch Mode | boolean | no | false | stored | Help text: "Evaluate the expression in batch instead of once for each user". |
| `batch_distinctive_field` | Distinctive field for batch user | many_to_one to Field Definition | no | none | stored | Help text: "In batch mode, this indicates which field distinguishes one user from the other, e.g. user_id, partner_id..." |
| `batch_user_expression` | Evaluated expression for batch mode | text | no | none | stored | Help text: "The value to compare with the distinctive field. The expression can contain reference to 'user' which is a browse record of the current user, e.g. user.id, user.partner_id.id..." |
| `compute_code` | Computation script | long_text | no | none | stored | The script evaluated per user in `python` mode; the result it produces is written into the goal's current value. The storage name of the field is `compute_code`. |
| `condition` | Goal Performance | selection | yes | `higher` | stored | `higher` (The higher the better) or `lower` (The lower the better). Help text: "A goal is considered as completed when the current value is compared to the value to reach". |
| `action_id` | Action | many_to_one to Window Action | no | none | stored | Help text: "The action that will be called to update the goal value." |
| `res_id_field` | Identifier Field of user | text | no | none | stored | Help text: "The field name on the user profile (res.users) containing the value for res_id for action." |

**Validation rules.**

| Trigger | Condition | Message |
|---|---|---|
| Creating or writing | The stored filter cannot be evaluated against the chosen entity | `The domain for the definition %(definition)s seems incorrect, please check it.  %(error_message)s` |
| Creating or writing | The chosen summed field is not stored | `The model configuration for the definition %(name)s seems incorrect, please check it.  %(field_name)s not stored` |
| Creating or writing | The chosen entity or field cannot be found | `The model configuration for the definition %(name)s seems incorrect, please check it.  %(error)s not found` |

---

# 31. Goal Update Wizard

**Full name:** Gamification Goal Wizard. **Transport name** `gamification.goal.wizard`,
**storage name** transient `gamification_goal_wizard`. **Kind:** transient entity.
**Contributed by** `gamification`. Generated reference page:
[`gamification.goal.wizard`](../../references/entities/gamification.goal.wizard.md).

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `goal_id` | Goal | many_to_one to Goal | yes | from the calling context | stored | The goal being updated. |
| `current` | Current | decimal | no | the goal's current value | stored | The new measured value. |

Confirming writes the value on the goal and closes the dialogue. The dialogue is offered only for a
goal whose definition is kept manually and that carries no window action of its own; the dialogue
title is `Update <definition name>`.

---

# 32. Karma Rank

**Full name:** Karma Rank (dictionary name Rank based on karma). **Transport name**
`gamification.karma.rank`, **storage name** `gamification_karma_rank`. **Kind:** persistent entity.
**Contributed by** `gamification`. **Carries an image.** Generated reference page:
[`gamification.karma.rank`](../../references/entities/gamification.karma.rank.md).

A named tier unlocked at a minimum reputation-point balance.

**Default ordering:** by `karma_min` ascending.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `name` | Rank Name | long_text (translatable) | yes | none | stored | The rank name. |
| `description` | Description | rich_text (translatable) | no | none | stored | Shown on the profile page of a holder. |
| `description_motivational` | Motivational | rich_text (translatable) | no | none | stored | Help text: "Motivational phrase to reach this rank on your profile page". Shown to somebody who has not reached the rank yet. |
| `karma_min` | Required Karma | integer | yes | 1 | stored | The lowest balance that unlocks the rank. |
| `user_ids` | Users | one_to_many to User | — | — | stored | The users currently at this rank; the inverse field is `rank_id`. |
| `rank_users_count` | # Users | integer | — | — | derived | The number of those users. |

**Database constraint `_karma_min_check`:** check `karma_min` is greater than zero; the refusal is
`The required karma has to be above 0.`

**Lifecycle.** Creating a rank recomputes the rank of every user whose balance reaches the lowest of
the newly created minimums. Writing `karma_min` recomputes the rank of every user in the affected
band — the whole population above the lower of the old and the new minimum when the ordering of the
ranks changed, and only the users between the two minimums when it did not. A user who moves to a
new rank receives the shipped rank message unless the change happens while a package is being
installed.

---

# 33. Karma Movement

**Full name:** Karma Movement (dictionary name Track Karma Changes). **Transport name**
`gamification.karma.tracking`, **storage name** `gamification_karma_tracking`.
**Kind:** persistent entity. **Contributed by** `gamification`, extended by `website_slides` and
`website_forum`. Generated reference page:
[`gamification.karma.tracking`](../../references/entities/gamification.karma.tracking.md).

One recorded change of one user's reputation-point balance. The balance itself is **derived from
this ledger**: a user's balance is the new value of their most recent movement, ordered by tracking
instant descending then identifier descending, and zero when they have no movement at all.

**Default ordering:** by `tracking_date` descending, then identifier descending.
**Display name rule:** the user name.

| Field | Full name | Type | Required | Default | Stored | Meaning and rules |
|---|---|---|---|---|---|---|
| `user_id` | User | many_to_one to User | yes | none | stored, indexed | Whose balance moved. Deletion behaviour: cascade. |
| `old_value` | Old Karma Value | integer | no | the user's balance at creation | stored, read-only | The balance before the movement. |
| `new_value` | New Karma Value | integer | yes | none | stored | The balance after the movement. |
| `gain` | Gain | integer | — | — | derived, writable | `new_value` minus `old_value`. When a caller supplies a gain together with an old value, the new value is computed from them and the gain itself is not stored. |
| `consolidated` | Consolidated | boolean | no | false | stored | True on a movement produced by the monthly consolidation. |
| `tracking_date` | Tracking Date | datetime | — | the current instant | stored, read-only, indexed | When the movement happened. |
| `reason` | Description | long_text | no | `Add Manually` | stored | The recorded reason, followed by the display name and the identifier of the source record in parentheses. |
| `origin_ref` | Source | reference | no | the acting user | stored | The record that caused the movement. |
| `origin_ref_model_name` | Source Type | selection | — | — | derived, stored | The kind of the source record. The base list holds one value, `res.users` (User); the learning package adds `slide.channel` (Course) and `slide.slide` (Quiz); the forum package adds `forum.post` (Forum Post). |

**Lifecycle.** Movements are created, never edited. The monthly consolidation replaces the movements
of a past month by one movement per user carrying the oldest old value and the newest new value of
that month; the source and the reason of the replaced movements are lost and the replacement carries
the reason `Consolidation from <start date> to <end date>`. Consolidation runs with the balance
recomputation suppressed, so no balance changes as a result.

---

# Part 5. Fields added to entities of other domains

# 34. User

Owned by [Identity and access](../identity-and-access/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `karma` | Karma | integer, derived from the movement ledger, stored, writable | The reputation-point balance. Writing it directly creates a movement whose gain is the difference between the written value and the current balance, whose source is the acting user and whose reason is `Add Manually`. Creating a user with a non-zero balance creates a movement with an old value of zero and the reason `User Creation`. |
| `karma_tracking_ids` | Karma Changes | one_to_many to Karma Movement | The movement ledger of this user. Visible only to the group `base.group_system`. |
| `badge_ids` | Badges | one_to_many to User Badge | The badges this user holds. Not copied on duplication. |
| `gold_badge` | Gold badges count | integer, derived | The number of held badges whose level is `gold`. |
| `silver_badge` | Silver badges count | integer, derived | The number of held badges whose level is `silver`. |
| `bronze_badge` | Bronze badges count | integer, derived | The number of held badges whose level is `bronze`. |
| `rank_id` | Rank | many_to_one to Karma Rank, indexed | The highest rank whose minimum the balance reaches. Empty below the lowest rank. |
| `next_rank_id` | Next Rank | many_to_one to Karma Rank | The next rank up. For a user with no rank it is the lowest rank; for a user at the top rank it is empty. |

**Behavioural additions.**

1. Creating a user enrols them into every course whose automatic-enrolment groups the user belongs
   to. Writing the group list enrols them into every course whose automatic-enrolment groups are
   among the newly linked groups and the groups those imply.
2. The public profile exposes the balance as a field the user may read about themselves, and the
   country, the city, the personal site address, the biography and the profile publication marker
   as fields the user may write about themselves.
3. Validating one's electronic mail address through the emailed link grants three reputation points,
   but only when the balance is still exactly zero. The validation token is a digest of the current
   day, a stored secret, the user identifier and the address, so it is valid for the day it was
   issued.
4. The list of destinations proposed after a rank change receives two entries: `See our Forum`
   pointing at `/forum`, added by the forum package, and `See our eLearning` pointing at `/slides`,
   added by the learning package.
5. Two ranking helpers are exposed for the public member list: one orders users by the sum of the
   gains recorded in the movement ledger inside a chosen date range, the other returns the absolute
   position of a subset of users in the whole ranking, by gain over a range or by total balance.

# 35. Access Group

Owned by [Identity and access](../identity-and-access/README.md). Writing the user list of a group
enrols every member of that group into every course that lists it as an automatic-enrolment group.

# 36. Contact

Owned by [Contacts and organizations](../contacts-and-organizations/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `certification_count` | Certifications | integer, derived | The number of completed, passed, non-test participations on certification questionnaires linked to this contact. Zero for a company. |
| `certification_company_count` | Company Certifications | integer, derived | The sum of that count over the child contacts of a company. Zero for an individual. |
| `slide_channel_ids` | Courses | many_to_many to Course, derived and searchable | The courses this contact is enrolled in. |
| `slide_channel_completed_ids` | Completed Courses | one_to_many to Course, derived and searchable | The courses this contact completed. |
| `slide_channel_count` | Course Count | integer, derived | The number of courses the contact is enrolled in. |
| `slide_channel_company_count` | Company Course Count | integer, derived | The sum of that count over the child contacts of a company. Zero for an individual. |

# 37. Contact Merge Wizard

Owned by [Contacts and organizations](../contacts-and-organizations/README.md). Merging is refused
when two of the contacts being merged are enrolled in the same course, with the message `You cannot
merge these contacts because multiple contacts are enrolled in the same courses: <course names>`,
because the merged contact would break the one-enrolment-per-pair uniqueness rule.

# 38. Language

Owned by [Contacts and organizations](../contacts-and-organizations/README.md). Writing the active
marker of a language to false removes that language from the language list of every questionnaire
that referenced it.

# 39. Planned Activity

Owned by [Messaging and activities](../messaging-and-activities/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `request_partner_id` | Requesting Partner | many_to_one to Contact | The contact asking for access to a course. Written on the to-do activity scheduled for the course responsible and read back by the course to answer whether the reader has already asked. |

# 40. Message

Owned by [Messaging and activities](../messaging-and-activities/README.md). A message attached to a
course content item carries, in the payload sent to the reader, the reputation-based permissions of
the owning course, so the comment box can be shown or hidden without a second request.

# 41. Site

Owned by [Website and storefront](../website-and-storefront/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `forum_count` | Forums | integer, read-only, default zero | How many forums are visible on this site. Recomputed whenever a forum is created, deleted, archived, un-archived or re-attached; a forum with no site is counted on every site. |
| `website_slide_google_app_key` | External document service key | text | The key used to reach the external document storage service when retrieving content metadata. Visible only to the group `base.group_system`. |

Two search types are added to the site search: forums with their posts, and courses with their
contents. Two suggested menu entries are added to the site builder: Courses pointing at `/slides`
and Forum pointing at `/forum`.

# 42. Configuration Settings

Owned by [Platform foundation](../platform-foundation/).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `website_slide_google_app_key` | External document service key | text, derived from the site record, writable | The key of the external document storage service. |
| `module_website_sale_slides` | Sell on eCommerce | boolean | Installs the course-selling package. |
| `module_website_slides_forum` | Forum | boolean | Installs the course-forum package. |
| `module_website_slides_survey` | Certifications | boolean | Installs the course-certification package. |
| `module_mass_mailing_slides` | Mailing | boolean | Installs the attendee-mailing package. |

# 43. Product Template and Product Variant

Owned by [Products and catalog](../products-and-catalog/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `channel_ids` (on the variant) | Courses | one_to_many to Course | The courses whose paid access this product grants. |

**Behavioural additions.** The service-tracking selection gains the value `course`; it is excluded
from the tracking values that forbid a zero price, and a product carrying it may always be added to
the cart when at least one published course uses it, whatever the ordinary cart rules say. The
multi-line sales description of such a product becomes `Access to: ` followed by the course names,
on one line for a single course and on separate lines for several.

# 44. Sales Order and Sales Order Line

Owned by [Sales](../sales/README.md). Confirming an order looks up every course whose enrolment
policy is `payment` and whose product appears on a line of the order, and enrols the order's
customer into each of them. Raising the quantity of a line whose product has the service tracking
`course` above one is refused: the quantity is forced back to one and the message `You can only add
a course once in your cart.` is returned. A course line may not be reordered from the order history.

# 45. Lead and Sales Team

Owned by [Customer relationship management](../customer-relationship-management/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `origin_survey_id` (on the lead) | Survey | many_to_one to Survey, indexed | The questionnaire the lead came from. Deletion behaviour: set to empty. |
| `origin_survey_ids` (on the team) | Survey opportunities related to the sales team | one_to_many to Survey | The questionnaires that assign their leads to this team. |

The creation of leads from lead-generating answer options is specified in
[`surveys.md`](surveys.md#lead-generation).

# 46. Job Position and Application

Owned by [Recruitment](../recruitment/README.md). A job position carries the interview
questionnaire; an application carries the participation. The questionnaire purpose selection gains
the value `recruitment`, which is excluded from every questionnaire list of this domain: the
participation lists, the detailed answer lists and the questionnaire menu all filter on the four
general purposes.

# 47. Employee, Public Employee and Résumé Line

Owned by [Human resources core](../human-resources-core/README.md).

| Field added | Full name | Type | Meaning |
|---|---|---|---|
| `subscribed_courses` (on the employee) | Courses | many_to_many to Course, derived from the contact | The courses the employee is enrolled in. |
| `has_subscribed_courses` (on the employee) | Has Subscribed Courses | boolean, derived | True when that list is not empty. |
| `courses_completion_text` (on the employee) | Courses Completion | text, derived | `<completed> / <total>`, the completed course count over the enrolled course count. |
| `channel_id` (on the résumé line) | eLearning Course | many_to_one to Course, derived, stored, indexed | The course the line records; cleared when the line type is not the electronic-learning one. |
| `course_url` (on the résumé line) | Course Link | text, derived from the course | The absolute address of the course. |
| `duration` (on the résumé line) | Duration | integer, derived, stored, writable | The total duration of the course. |
| `course_type` (on the résumé line) | Course Type | selection | Gains the value `elearning` (eLearning); removing the package deletes the lines holding it. |
| `survey_id` (on the résumé line) | Certification | many_to_one to Survey, read-only | The certification the line records. |
| `department_id` (on the résumé line) | Department | many_to_one to Department, derived from the employee, stored | Denormalised for reporting. |
| `expiration_status` (on the résumé line) | Expiration Status | selection, derived, stored | `expired` (Expired) when the end date is today or earlier, `expiring` (Expiring) when the end date is within three months, `valid` (Valid) otherwise. |
| `employee_id` (on the granted badge) | Employee | many_to_one to Employee, indexed | Which employee the badge belongs to. |

**Behavioural additions.** Completing a course writes a résumé line of the Training type, naming the
course, dated today, describing it with the plain text of the course description and linking the
course; a line is written only when none already exists for that employee, that course and that
type. Passing a certification writes or updates a résumé line of the Certification type, naming the
questionnaire, dated today, ending after the validity in months when one is set and never ending
when it is zero. Enrolling, completing and leaving a course each post a note on the employee record.
Duplicating a résumé line names the copy `<name> (copy)`.

# 48. Mailing

Owned by [Marketing and mass mailing](../marketing-and-mass-mailing/README.md). The course form offers an
action that opens a new mailing whose recipient entity is Contact and whose stored filter selects
the contacts enrolled in the selected courses. The action is titled `Mass Mail Course Members`.

# 49. Spreadsheet Dashboard

Owned by [Spreadsheets and dashboards](../spreadsheets-and-dashboards/README.md). One dashboard
named `eLearning` is shipped, grouped under the site dashboards, restricted to the Learning Manager
group, published, given the sequence 200, and built over the order entity.

# 50. Rating

Owned by [Website and storefront](../website-and-storefront/README.md). A course review is a message
carrying a rating. Only published ratings count towards the average; posting one grants the course's
review reward; only one review per author per course is accepted.
