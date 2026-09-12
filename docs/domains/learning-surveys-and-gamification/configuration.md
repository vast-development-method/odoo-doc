# Learning, Questionnaires and Recognition — Configuration

Every setting, system parameter, shipped record, security group, access right, record rule,
scheduled job, message template and message subtype the domain relies on.

---

## 1. Settings shown on the configuration screen

All five belong to the learning platform and are written on the configuration settings record of the
[Platform foundation](../platform-foundation/) domain.

| Setting | Label | Kind | Effect |
|---|---|---|---|
| `website_slide_google_app_key` | External document service key | text, stored on the site record and visible only to the group `base.group_system` | The key used to reach the external document storage service when retrieving the metadata of an external content item. Without it, an external document or video is created with no title, no illustration and no duration. |
| `module_website_sale_slides` | Sell on eCommerce | switch | Installs the course-selling package, which adds the enrolment policy `payment`, the product link, the one-unit cart rule and the revenue figure. |
| `module_website_slides_forum` | Forum | switch | Installs the course-forum package, which adds the forum link on a course and hands forum access control to the course visibility. |
| `module_website_slides_survey` | Certifications | switch | Installs the course-certification package, which adds the content category `certification`, the attempt pool per enrolment and the certified marker. |
| `module_mass_mailing_slides` | Mailing | switch | Installs the attendee-mailing package, which adds the action opening a mailing addressed to the attendees of the selected courses. |

The questionnaire application, the forum and the recognition machinery expose no switch of their
own; they are configured entirely on their records.

## 2. System parameters

| Parameter | Set by | Meaning |
|---|---|---|
| `auth_signup.invitation_scope` | shipped by the forum package with the value `b2c` | Opens self-registration to external visitors, so a forum member can create an account. The value governs the whole platform, not only the forum. |
| `website_profile.uuid` | generated on first use by the public profile package | A stored secret used to build the address-validation token. It is created the first time a validation message is sent and never rotated automatically. |

Two further secrets are read from the platform rather than stored by this domain: the key used to
sign the course invitation digest, taken from the platform secret with the label
`website_slides-channel-invite`, and the session lifetime used by the daily recognition job to
decide whether a participant's session is still valid.

## 3. Security groups

| Group | Privilege family | Implies | Granted to |
|---|---|---|---|
| `survey.group_survey_user` | Surveys, sequence 20, under the marketing category | — | Assigned by an administrator. Labelled User. |
| `survey.group_survey_manager` | Surveys | `survey.group_survey_user` | The system account and the administrator account by default. Labelled Administrator. |
| `website_slides.group_website_slides_officer` | eLearning, sequence 21, under the site category | `website.group_website_restricted_editor` | Assigned by an administrator. Labelled Officer. |
| `website_slides.group_website_slides_manager` | eLearning | `website_slides.group_website_slides_officer` | The system account and the administrator account by default. Labelled Manager. |

The forum uses no group of its own: it is governed by the reputation-point thresholds, by the
platform groups `base.group_public`, `base.group_portal`, `base.group_user` and
`base.group_erp_manager`, and by `website.group_website_designer` for creating a private forum. The
recognition machinery uses `base.group_user`, `base.group_portal`, `base.group_public`,
`base.group_erp_manager`, `base.group_system` and, when the human-resources package is installed,
`hr.group_hr_user`.

## 4. Access rights

### 4.1 Questionnaires

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| `survey.survey`, `survey.question`, `survey.question.answer` | all users, and `base.group_user` | no | no | no | no |
| the same three | `survey.group_survey_user` | yes | yes | yes | yes |
| the same three | `survey.group_survey_manager` | yes | yes | yes | yes |
| the same three | `website_slides.group_website_slides_officer` | no | yes | no | no |
| `survey.user_input` | all users, and `base.group_user` | no | no | no | no |
| `survey.user_input` | `survey.group_survey_user` | yes | yes | yes | yes |
| `survey.user_input` | `survey.group_survey_manager` | yes | yes | yes | yes |
| `survey.user_input` | `website_slides.group_website_slides_officer` | no | yes | no | no |
| `survey.user_input.line` | `survey.group_survey_user` | no | yes | no | no |
| `survey.user_input.line` | `survey.group_survey_manager` | yes | yes | yes | yes |
| `survey.user_input.line` | `website_slides.group_website_slides_officer` | no | yes | no | no |
| `survey.invite` | `survey.group_survey_user` | yes | yes | yes | no |
| `gamification.badge` | `survey.group_survey_user` | yes | yes | yes | yes |

The asymmetry on the answer rows is deliberate: an officer may delete a whole participation but may
not create or delete an individual answer row, so a score cannot be edited piecemeal.

The recruitment interview package adds read and write for the recruitment manager group and read for
the recruitment officer and interviewer groups on the questionnaire entities, and read on the
participation and its rows.

### 4.2 Learning platform

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| `slide.channel`, `slide.slide` | `base.group_public`, `base.group_portal`, `base.group_user` | no | yes | no | no |
| `slide.channel`, `slide.slide` | `website_slides.group_website_slides_officer` | yes | yes | yes | no |
| `slide.channel`, `slide.slide` | `website_slides.group_website_slides_manager` | yes | yes | yes | yes |
| `slide.channel.partner`, `slide.slide.partner` | all internal users | no | no | no | no |
| `slide.channel.partner`, `slide.slide.partner` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes |
| `slide.question` | `base.group_public`, `base.group_portal`, `base.group_user` | no | yes | no | no |
| `slide.question` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes |
| `slide.answer` | all internal users | no | no | no | no |
| `slide.answer` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes |
| `slide.slide.resource` | `base.group_public` | no | no | no | no |
| `slide.slide.resource` | `base.group_portal`, `base.group_user` | no | yes | no | no |
| `slide.slide.resource` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes |
| `slide.embed` | `base.group_public`, `base.group_portal` | no | yes | no | no |
| `slide.embed` | `base.group_user` | yes | yes | yes | yes |
| `slide.tag`, `slide.channel.tag`, `slide.channel.tag.group` | `base.group_public`, `base.group_portal`, `base.group_user` | no | yes | no | no |
| `slide.tag`, `slide.channel.tag`, `slide.channel.tag.group` | `website_slides.group_website_slides_officer` | yes | yes | yes | yes |
| `slide.channel.invite` | `base.group_user` | yes | yes | yes | no |

### 4.3 Forum

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| `forum.forum` | `base.group_public`, `base.group_portal`, `base.group_user` | no | yes | no | no |
| `forum.forum` | `base.group_erp_manager` | yes | yes | yes | yes |
| `forum.forum` | `website_slides.group_website_slides_officer` | yes | yes | yes | no |
| `forum.post` | `base.group_public` | no | yes | no | no |
| `forum.post` | `base.group_portal`, `base.group_user` | yes | yes | yes | yes |
| `forum.post.vote` | `base.group_portal` | yes | yes | yes | no |
| `forum.post.vote` | `base.group_user` | yes | yes | yes | yes |
| `forum.post.reason` | `base.group_public`, `base.group_portal` | no | yes | no | no |
| `forum.post.reason` | `base.group_user` | yes | yes | yes | yes |
| `forum.tag` | `base.group_public`, `base.group_portal` | yes | yes | no | no |
| `forum.tag` | `base.group_user` | yes | yes | yes | yes |

An anonymous visitor may create a forum tag at the access-right level; the reputation threshold
[`LSG-110`](business-rules.md#lsg-110) is what actually stops them, because an anonymous account
holds no reputation points.

### 4.4 Recognition

| Entity | Group | Create | Read | Update | Delete |
|---|---|---|---|---|---|
| `gamification.goal` | `base.group_user`, `base.group_portal` | no | yes | yes | no |
| `gamification.goal` | `base.group_erp_manager` | yes | yes | yes | yes |
| `gamification.goal.definition`, `gamification.challenge`, `gamification.challenge.line` | `base.group_user`, `base.group_portal` | no | yes | no | no |
| the same three | `base.group_erp_manager` | yes | yes | yes | yes |
| the same three | `hr.group_hr_user` | yes | yes | yes | yes |
| `gamification.badge` | `base.group_user`, `base.group_portal`, `base.group_public` | no | yes | no | no |
| `gamification.badge` | `base.group_erp_manager`, `hr.group_hr_user` | yes | yes | yes | yes |
| `gamification.badge.user` | `base.group_user`, `base.group_portal` | yes | yes | yes | no |
| `gamification.badge.user` | `base.group_public` | no | yes | no | no |
| `gamification.badge.user` | `base.group_erp_manager`, `hr.group_hr_user` | yes | yes | yes | yes |
| `gamification.karma.rank` | `base.group_public`, `base.group_portal`, `base.group_user` | no | yes | no | no |
| `gamification.karma.rank` | `base.group_system`, `website.group_website_restricted_editor` | yes | yes | yes | yes |
| `gamification.karma.tracking` | every other group | no | no | no | no |
| `gamification.karma.tracking` | `base.group_system` | yes | yes | yes | yes |
| `gamification.badge.user.wizard`, `gamification.goal.wizard` | `base.group_user` | yes | yes | yes | no |

## 5. Record rules

### 5.1 Questionnaires

| Rule | Groups | What it allows |
|---|---|---|
| Manager: all | `survey.group_survey_manager` | Every questionnaire, question, answer option, invitation and participation of the four general purposes. |
| Officer: unrestricted or in the restricted list | `survey.group_survey_user` | Read, write, create and delete on questionnaires whose restricted list is empty or names the officer, and on their questions, answer options, participations and answer rows. |
| Learning Officer on a certification: read | `website_slides.group_website_slides_officer` | Read on certifications of the four general purposes whose restricted list is empty or names the officer, and on their questions, answer options, participations and answer rows. |
| Recruitment manager: all recruitment | `hr_recruitment.group_hr_recruitment_manager` | Every record whose purpose is `recruitment`. |
| Recruitment officer | `hr_recruitment.group_hr_recruitment_user` | Read on recruitment questionnaires that are unrestricted or name the officer. |
| Recruitment interviewer | `hr_recruitment.group_hr_recruitment_interviewer` | Read on the recruitment questionnaires of the job positions and applications for which the user is an interviewer, and on the participations of those applications. |

The manager and officer rules on the participation and the answer row are restricted to the four
general purposes, so a questionnaire officer never reads an interview.

### 5.2 Learning platform

| Rule | Groups | What it allows |
|---|---|---|
| Course: always visible, with sub-rules | global | The base rule that lets the sub-rules below add their own visibility. |
| Course: anonymous visitor | `base.group_public` | Published courses whose visibility is `public` or `link`. |
| Course: signed-in visitor | `base.group_portal`, `base.group_user` | Published courses whose visibility is `public`, `connected` or `link`, plus the courses the reader is enrolled in or invited to. |
| Course: officer read all | `website_slides.group_website_slides_officer` | Read on every course. |
| Course: officer create and write own | `website_slides.group_website_slides_officer` | Create and write only on courses the officer is responsible for. |
| Course: manager all | `website_slides.group_website_slides_manager` | Everything. |
| Content: anonymous visitor | `base.group_public` | Published contents of published courses whose visibility is `public` or `link`, and only when the content is a section or a free preview. |
| Content: signed-in visitor | `base.group_portal`, `base.group_user` | The contents the reader uploaded, plus published contents of published courses, restricted to sections and free previews when the reader is only a visitor or an invited attendee, and unrestricted when the reader is enrolled. |
| Content: officer read all, create and write own, manager all | the two learning groups | As for the course, keyed on the responsible of the owning course. |
| Enrolment and progress: officer own courses | `website_slides.group_website_slides_officer` | Create, write and delete on the records of the courses the officer is responsible for. |
| Enrolment and progress: manager all | `website_slides.group_website_slides_manager` | Everything. |
| Resource: attendees | `base.group_portal`, `base.group_user` | Read only when the reader is enrolled in the owning course. |
| Resource: officer and manager | the two learning groups | Read all for an officer; create, write and delete on own courses; everything for a manager. |
| Course tag: anonymous and portal | `base.group_public`, `base.group_portal` | Read only the tags that carry a colour, so internal tags stay hidden. |

### 5.3 Forum

| Rule | Groups | What it allows |
|---|---|---|
| Forum, post and tag: anonymous visitor | `base.group_public` | Only the records of forums whose privacy is `public`. |
| Forum, post and tag: signed-in visitor | `base.group_portal`, `base.group_user` | The records of forums whose privacy is `public` or `connected`, plus those of private forums whose authorised group the reader belongs to. |
| Forum: site designer may create | `website.group_website_designer` | Create only, so a designer may open a private forum. |
| Forum, post and tag: administrator | `base.group_erp_manager` | Everything. |
| Course forum: anonymous visitor | `base.group_public` | The forums, posts and tags attached to a published course whose visibility is `public`. |
| Course forum: signed-in visitor | `base.group_portal`, `base.group_user` | The same for courses whose visibility is `public` or `connected`, plus the courses the reader is enrolled in. |
| Course forum: Learning Officer | `website_slides.group_website_slides_officer` | Everything. |
| Vote: own votes | `base.group_portal`, `base.group_user` | Only the reader's own votes. |
| Vote: all | `base.group_erp_manager` | Everything. |

### 5.4 Recognition

| Rule | Groups | What it allows |
|---|---|---|
| Goal: own goals or a ranking challenge | `base.group_user`, `base.group_portal` | Read and write the reader's own goals, plus the goals of a challenge the reader participates in whose display mode is `ranking`. Creating and deleting are not allowed. |
| Goal: manager | `base.group_erp_manager` | Read and write every goal; creating and deleting are not allowed by this rule. |
| Goal: human-resources officer | `hr.group_hr_user` | Read and write every goal. |
| Goal: company scope | global | Only the goals whose user belongs to one of the reader's companies. |
| Granted badge: own grants | `base.group_user` | Read, write, create and delete the grants the reader created. |
| Granted badge: grants by others | `base.group_user` | Read and create only; a member may not edit or delete somebody else's grant. |
| Granted badge: human-resources officer | `hr.group_hr_user` | Everything. |

## 6. Scheduled jobs

| Job | Name | Interval | What it does |
|---|---|---|---|
| `gamification.ir_cron_check_challenge` | `Gamification: Goal Challenge Check` | every day | Starts planned challenges, closes finished ones, re-measures the goals, regenerates the missing ones, sends the reports and distributes the rewards; see [`workflows.md`](workflows.md#workflow-19). |
| `gamification.ir_cron_consolidate` | `Gamification: Karma tracking consolidation` | every month, first scheduled for the first day of the next month at four in the morning | Consolidates the reputation movements of the month that ended two months earlier; see [`LSG-CALC-033`](calculations.md#lsg-calc-033). |
| the periodic cleanup of expired course invitations | runs as part of the platform's automatic vacuum | — | Deletes every enrolment whose status is `invited`, whose completion is zero, and whose last invitation instant is empty or older than three months. |

## 7. Message templates

| Template | Name | Subject | Written on |
|---|---|---|---|
| `survey.mail_template_user_input_invite` | `Survey: Invite` | `Participate to <questionnaire title> survey` | Survey Participation |
| `survey.mail_template_certification` | `Survey: Certification Success` | `Certification: <questionnaire title>` | Survey Participation |
| `website_slides_survey.mail_template_user_input_certification_failed` | `Survey: Certification Failure` | `You have failed the course: <course name>` | Survey Participation |
| `hr_recruitment_survey.mail_template_applicant_interview_invite` | `Applicant: Interview` | `Participate to <questionnaire title> interview` | Survey Participation |
| `website_slides.slide_template_published` | `Elearning: New Course Content Notification` | `New <content category> published on <course name>` | Course Content |
| `website_slides.slide_template_shared` | `Elearning: Course Share` | `<sender name> shared a <content category> with you!` | Course Content |
| `website_slides.mail_template_channel_shared` | `Channel Shared` | `<sender name> shared a Course` | Course |
| `website_slides.mail_template_channel_completed` | `Elearning: Completed Course` | `Congratulations! You completed <course name>` | Course Enrolment |
| `website_slides.mail_template_slide_channel_enroll` | `Elearning: Add Attendees to Course` | `You have been invited to join <course name>` | Course Enrolment |
| `website_slides.mail_template_slide_channel_invite` | `Elearning: Promotional Course Invitation` | `You have been invited to check out <course name>` | Course Enrolment |
| `gamification.email_template_badge_received` | `Gamification: Badge Received` | `New badge <badge name> granted` | User Badge. The notification sent when a grant is created overrides the subject with `🎉 You've earned the <badge name> badge!` |
| `gamification.email_template_goal_reminder` | `Gamification: Reminder For Goal Update` | rendered from the body alone | Goal |
| `gamification.simple_report_template` | `Gamification: Challenge Report` | rendered from the body alone | Challenge |
| `gamification.mail_template_data_new_rank_reached` | `Gamification: New Rank Reached` | `New rank: <rank name>` | User |
| `website_profile.validation_email` | the address-validation message | rendered from the body alone | User |

The forum ships its own notification layouts for the new-question, new-answer and validation
notices rather than templates.

## 8. Message subtypes

| Subtype | Name | Written on | Followed by default | Meaning |
|---|---|---|---|---|
| `survey.mt_survey_user_input_completed` | `Participation completed` | Survey Participation | no | A participation reached the completed state. Description: `Participation completed.` |
| `survey.mt_survey_survey_user_input_completed` | `Participation completed` | Survey | no | The parent subtype that carries the same notice to the followers of the questionnaire. Description: `New participation completed.` |
| `website_slides.mt_channel_slide_published` | `Presentation Published` | Course | yes | A content item was published on the course. Description: `Presentation Published` |
| `website_forum.mt_question_new` | `New Question` | Forum Post | yes | A question was asked. |
| `website_forum.mt_question_edit` | `Question Edited` | Forum Post | no | A question's title or body changed. |
| `website_forum.mt_answer_new` | `New Answer` | Forum Post | yes | An answer was posted on the question. |
| `website_forum.mt_answer_edit` | `Answer Edited` | Forum Post | no | An answer's title or body changed. |
| `website_forum.mt_forum_question_new` | `New Question` | Forum | yes | The forum-level mirror of the new-question subtype. |
| `website_forum.mt_forum_answer_new` | `New Answer` | Forum | yes | The forum-level mirror of the new-answer subtype. |

## 9. Activity types

The domain defines no activity type of its own. Course access requests use the platform's to-do
type, summarised `Access Request`, assigned to the course responsible, carrying the requesting
contact and the note `<visitor name> is requesting access to this course.` They are closed with the
feedback `Access Granted` or `Access Refused`.

## 10. Shipped records

<a id="shipped-reasons"></a>

### 10.1 Post closing reasons

Thirteen reasons are shipped.

| Reason | Kind |
|---|---|
| `Duplicate post` | `basic` |
| `Off-topic or not relevant` | `basic` |
| `Too subjective and argumentative` | `basic` |
| `Not a real post` | `basic` |
| `Not relevant or out dated` | `basic` |
| `Contains offensive or malicious remarks` | `basic` |
| `Spam or advertising` | `basic` |
| `Too localized` | `basic` |
| `Insulting and offensive language` | `offensive` |
| `Violent language` | `offensive` |
| `Inappropriate and unacceptable statements` | `offensive` |
| `Threatening language` | `offensive` |
| `Racist and hate speech` | `offensive` |

The reason `Contains offensive or malicious remarks` is the **offensive reason** and `Spam or
advertising` is the **spam reason**; both are special-cased by
[`LSG-CALC-022`](calculations.md#lsg-calc-022).

### 10.2 Forums

One forum named `Help` is shipped, with the default mode, the default privacy and the shipped
guidelines page.

### 10.3 Course tag families and tags

| Record | Sequence | Published |
|---|---|---|
| Family `Tags` | 20 | yes |
| Family `Level` | 10 | yes |
| Tag `Basic`, in the family `Level`, colour 10 | — | — |
| Tag `Intermediate`, in the family `Level`, colour 3 | — | — |
| Tag `Advanced`, in the family `Level`, colour 1 | — | — |

### 10.4 Résumé line types

| Record | Name | Sequence | Counts as a course |
|---|---|---|---|
| the certification type, shipped by the skills-certification package | `Internal Certification` | 25 | no |
| the training type, shipped by the skills package of the human-resources domain | — | — | yes; it is the type used for a completed course |

<a id="shipped-recognition-records"></a>

### 10.5 General recognition records

Four goal definitions and two challenges help a new installation get configured. Every one of them
uses the yes-or-no display and the counting computation mode.

| Goal definition | Measures | Filter | Condition |
|---|---|---|---|
| `Set your Timezone` | users | the contact has a time zone | higher |
| `Set your Company Data` | companies | the reader's company still bears the placeholder name | lower |
| `Set your Company Logo` | companies | the reader's company has a logotype | higher |
| `Invite new Users` | users | any user other than the reader | higher |

| Challenge | Population | Periodicity | Display | Report | Category | Lines |
|---|---|---|---|---|---|---|
| `Complete your Profile` | internal users | `once` | personal | never | `other` | `Set your Timezone`, target 1 |
| `Setup your Company` | administrators | `once` | personal | never | `other` | `Set your Company Logo` target 1, `Set your Company Data` target 0, `Invite new Users` target 1 |

Both challenges are shipped in the state `inprogress`.

### 10.6 Badges shipped with the recognition package

| Badge | Granting rule |
|---|---|
| `Good Job` | `everyone` |
| `Problem Solver` | `everyone` |
| `Hidden` | `nobody` |
| `Brilliant` | `everyone` |

### 10.7 Ranks

| Rank | Required balance |
|---|---|
| `Newbie` | 1 |
| `Student` | 100 |
| `Bachelor` | 500 |
| `Master` | 2000 |
| `Doctor` | 10000 |

Each rank carries an illustration, a description shown to its holders and a motivational phrase
shown to those who have not reached it yet. Two starting movements are shipped, one for the system
account and one for the administrator account, both setting the balance to 2500 with the reasons
`I am the Root!` and `I am the Admin!`

### 10.8 Badges shipped with the learning platform

Five badges, each with one goal definition, one challenge and one challenge line. Every challenge
uses the category `slides`, the periodicity `once`, the personal display, no report, a real-time
reward, a population filter selecting the users whose balance is strictly greater than zero, the
state `inprogress` and a target of one. Every goal definition counts records, displays a yes-or-no
result, uses the condition "the higher the better" and is evaluated in batch.

| Badge | Level | Published | What the goal counts |
|---|---|---|---|
| `Get started` — `Register to the platform` | bronze | yes | Active users whose balance is greater than zero, keyed on the user. |
| `Know yourself` — `Complete your profile` | bronze | yes | Users whose contact has a country, a city and an electronic mail address, keyed on the user. |
| `Power User` — `Complete a course` | silver | yes | Course enrolments whose status is `completed`, keyed on the contact. |
| `Certified Knowledge` — `Get a certification` | gold | not published until the course-certification package is installed | Progress records: before that package the filter is deliberately unsatisfiable, and the package replaces it with progress records whose certification succeeded on a content of the `certification` category, keyed on the contact. |
| `Community hero` — `Reach 2000 XP` | gold | yes | Users whose balance is at least 2000, keyed on the user. |

All five badges carry the granting rule `nobody`, so only their challenge may award them.

<a id="shipped-forum-badges"></a>

### 10.9 Badges shipped with the forum

Twenty-nine badges, twenty-seven goal definitions, twenty-eight challenges and twenty-eight
challenge lines. Every challenge uses the category `forum`, the periodicity `once`, the personal
display, no report, a real-time reward and a population filter selecting the users whose balance is
strictly greater than zero. Every goal definition counts records, displays a yes-or-no result, uses
the condition "the higher the better" and is evaluated in batch.

**Question badges.** The goals count the reader's own questions matching the filter.

| Badge | Level | Description | Filter | Target |
|---|---|---|---|---|
| `Popular Question` | bronze | `Asked a question with at least 150 views` | question with at least 150 views | 1 |
| `Notable Question` | silver | `Asked a question with at least 250 views` | question with at least 250 views | 1 |
| `Famous Question` | gold | `Asked a question with at least 500 views` | question with at least 500 views | 1 |
| `Credible Question` | bronze | `Question set as favorite by 1 user` | question bookmarked at least once | 1 |
| `Favorite Question` | silver | `Question set as favorite by 5 users` | question bookmarked at least five times | 1 |
| `Stellar Question` | bronze | `Question set as favorite by 25 users` | question bookmarked at least twenty-five times | 1 |
| `Student` | gold | `Asked first question with at least one up vote` | question with a vote total of at least 1 | 1 |
| `Nice Question` | bronze | `Question voted up 4 times` | question with a vote total of at least 4 | 1 |
| `Good Question` | silver | `Question voted up 6 times` | question with a vote total of at least 6 | 1 |
| `Great Question` | gold | `Question voted up 15 times` | question with a vote total of at least 15 | 1 |
| `Scholar` | gold | `Asked a question and accepted an answer` | question whose answered marker is true | 1 |

**Answer badges.** The goals count the reader's own answers matching the filter.

| Badge | Level | Description | Filter | Target |
|---|---|---|---|---|
| `Teacher` | bronze | `Received at least 3 upvote for an answer for the first time` | answer with a vote total of at least 3 | 1 |
| `Nice Answer` | bronze | `Answer voted up 4 times` | answer with a vote total of at least 4 | 1 |
| `Good Answer` | silver | `Answer voted up 6 times` | answer with a vote total of at least 6 | 1 |
| `Great Answer` | gold | `Answer voted up 15 times` | answer with a vote total of at least 15 | 1 |
| `Enlightened` | silver | `Answer was accepted with 3 or more votes` | accepted answer with a vote total of at least 3 | 1 |
| `Guru` | silver | `Answer accepted with 15 or more votes` | accepted answer with a vote total of at least 15 | 1 |
| `Self-Learner` | gold | `Answered own question with at least 4 up votes` | answer to one's own question with a vote total of at least 4 | 1 |

**Participation badges.**

| Badge | Level | Description | What the goal counts | Target |
|---|---|---|---|---|
| `Autobiographer` | bronze | `Completed own biography` | users whose contact has a country, a city and an address, keyed on the user | 1 |
| `Commentator` | bronze | `Posted 10 comments` | comment messages written by the reader's contact, keyed on the contact | 10 |
| `Chief Commentator` | silver | `Posted 100 comments` | the same definition as `Commentator` | 100 |
| `Pundit` | silver | `Left 10 answers with score of 10 or more` | answers with a vote total of at least 10, keyed on the user | 10 |
| `Taxonomist` | silver | `Created a tag used by 15 questions` | tags the reader created whose post count is at least 15, keyed on the user | 1 |

**Moderation badges.**

| Badge | Level | Description | What the goal counts | Target |
|---|---|---|---|---|
| `Critic` | bronze | `First downvote` | votes of the reader whose value is minus one, keyed on the user | 1 |
| `Supporter` | gold | `First upvote` | votes of the reader whose value is plus one, keyed on the user | 1 |
| `Editor` | gold | `First edit` | edit messages written by the reader's contact, keyed on the contact | 1 |
| `Disciplined` | bronze | `Deleted own post with 3 or more upvotes` | archived posts of the reader with a vote total of at least 3, keyed on the user | 1 |
| `Peer Pressure` | gold | `Deleted own post with 3 or more downvotes` | archived posts of the reader with a vote total of at most minus 3, keyed on the user | 1 |
| `Cleanup` | gold | `First rollback` | — this badge is shipped without a goal definition and without a challenge, so nothing awards it automatically | — |

**Compatibility finding.** `Cleanup` is shipped as a badge that no challenge awards and whose
granting rule does not reserve it for challenges, so it can only ever be granted by hand. A
corrected behaviour would either ship the matching goal definition and challenge or remove the
badge.

### 10.10 Recognition records shipped for the selling pipeline

The selling-recognition package ships ten goal definitions and two challenges, all evaluated in
batch and keyed on the salesperson of the measured record.

| Goal definition | Computation | Measures |
|---|---|---|
| `Total Invoiced` | sum | the untaxed amount of customer invoices that are not cancelled |
| `New Leads` | count | leads and opportunities |
| `Time to Qualify a Lead` | sum | the qualification delay of leads |
| `Days to Close a Deal` | sum | the closing delay of leads and opportunities |
| `New Opportunities` | count | opportunities |
| `New Sales Orders` | count | orders past the draft, sent and cancelled states |
| `Paid Sales Orders` | count | customer invoices that are paid or in payment |
| `Total Paid Sales Orders` | count | the same population, with the untaxed amount named as the summed field |
| `Customer Credit Notes` | count | customer credit notes that are not cancelled |
| `Total Customer Credit Notes` | sum | the untaxed amount of those credit notes |

| Challenge | Lines |
|---|---|
| `Monthly Sales Targets` | one line on `Total Invoiced` |
| `Lead Acquisition` | three lines: `New Leads`, `New Opportunities` and one more of the definitions above |

**Compatibility finding.** `Total Paid Sales Orders` names a field to sum but is configured to count
records, so it reports a record count under a monetary label. A corrected behaviour would set its
computation mode to summing.

### 10.11 Dashboards

One spreadsheet dashboard named `eLearning` is shipped by the dashboard package of the learning
platform, grouped under the site dashboards, restricted to the Learning Manager group, published,
given the sequence 200 and built over the order entity.

## 11. Default values that a rebuild must reproduce

| Where | Field | Default |
|---|---|---|
| Survey | purpose | `custom` |
| Survey | pagination | `page_per_question` |
| Survey | question selection | `all` |
| Survey | progress indicator | `percent` |
| Survey | access mode | `public` |
| Survey | required percentage | 80.00 |
| Survey | attempt cap | 1 |
| Survey | countdown | 10 minutes |
| Survey | certificate layout | `modern_purple` |
| Survey Question | sequence | 10 |
| Survey Question | random count on a section | 1 |
| Survey Question | scale bounds | 0 and 10 |
| Survey Question | matrix subtype | `simple` |
| Survey Answer Option | sequence | 10 |
| Survey Participation | state | `new` |
| Survey Invitation Wizard | handling of existing recipients | `resend` |
| Course | type | `training` |
| Course | sequence | 10 |
| Course | visibility | `public` |
| Course | enrolment policy | `public` |
| Course | featured content | `latest` |
| Course | enrolment message | `Contact Responsible` |
| Course | review reward | 5 |
| Course | completion reward | 10 |
| Course | review threshold | 10 |
| Course | comment threshold | 3 |
| Course | vote threshold | 3 |
| Course Content | category | `document` |
| Course Content | source | `local_file` |
| Course Content | quiz rewards | 10, 7, 5 and 2 |
| Course Enrolment | status | `joined` |
| Course Tag | colour | a random whole number between 1 and 11 |
| Course Tag and Family | sequence | 10 |
| Forum | sequence | 1 |
| Forum | mode | `questions` |
| Forum | privacy | `public` |
| Forum | default ordering | `last_activity_date desc` |
| Forum | first relevance parameter | 0.8 |
| Forum | second relevance parameter | 1.8 |
| Forum | sharing offered | yes |
| Forum | the forty-two reputation settings | as tabulated in sections 19.3 and 19.4 of [`entities.md`](entities.md) |
| Forum Post | state | `active` |
| Post Vote | value | `1` |
| Post Closing Reason | kind | `basic` |
| Badge | level | `bronze` |
| Badge | granting rule | `everyone` |
| Challenge | state | `draft` |
| Challenge | periodicity | `once` |
| Challenge | display mode | `personal` |
| Challenge | report frequency | `never` |
| Challenge | reward in real time | yes |
| Challenge | category | `hr` |
| Challenge Line | sequence | 1 |
| Goal | state | `draft` |
| Goal | current value | 0 |
| Goal Definition | computation mode | `manually` |
| Goal Definition | display mode | `progress` |
| Goal Definition | condition | `higher` |
| Goal Definition | filter | the empty filter |
| Karma Rank | required balance | 1 |
| Karma Movement | reason | `Add Manually` |
| Karma Movement | source | the acting user |
| Content Embed Counter | view count | 1 |
