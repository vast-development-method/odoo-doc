# Learning, Questionnaires and Recognition

This domain specifies three capabilities that share one currency. The first is the questionnaire
engine: an author builds a questionnaire out of sections and questions of nine shapes, decides how
it is paginated, how it is scored, who may open it and for how long, shares it by a public link or
by personal invitations, and reads back the answers as participations that carry a score, a pass or
fail verdict and, when the questionnaire is a certification, a printable certificate. The same
engine runs as a live session in which a host advances the questions and a leaderboard ranks the
attendees. The second is the learning platform: a course is an ordered list of content items
grouped into sections, an attendee enrols freely, on invitation or after paying, walks the contents,
answers quizzes, takes certifications and reaches a completion percentage that closes the course
and sends a congratulation message. The third is the community forum: a question-and-answer space
in which every visible action is gated by a threshold of reputation points and every judged action
moves reputation points between accounts.

The currency they share is the reputation-point balance held on a user account, stored under the
name `karma`. This folder also owns the recognition machinery that turns that balance and other
measurable facts into visible rewards: goal definitions that say what to measure, goals that hold
one user's measurement, challenges that group goals and award badges, badges and the records of who
holds them, ranks that a balance unlocks, and a full ledger of every reputation-point movement.

The domain owns no journal entries. Where a course is sold, the money reaches the ledger through
the selling domain; that boundary is specified in [`accounting-effects.md`](accounting-effects.md).

---

## 1. The questions this domain answers

1. What questionnaires exist, of what purpose, with what questions, what validation and what
   scoring?
2. Who may open a questionnaire, how many times, for how long, and what happens when the time runs
   out?
3. What did each participant answer, what score did that earn, did they pass, and what document
   and badge does passing produce?
4. How does a host run a live session: joining by code, advancing question by question, revealing
   results, ranking attendees, rewarding speed?
5. What courses exist, who may see them, who may join them, and how?
6. What content does a course hold, in what order, of what kind, from what source, lasting how
   long?
7. How far has each attendee progressed, what did that earn them, and when is the course finished?
8. What forums exist, who may read them, and what does each action cost or grant in reputation
   points?
9. What states can a forum post reach, who may move it, and what does each move do to the author's
   balance?
10. What badges, ranks and challenges exist, how are they measured, and who has earned what?
11. Where does every reputation point in the system come from and go to?

---

## 2. Capabilities covered

| Capability | Summary |
|---|---|
| Questionnaire authoring | Title, description, closing message, background image, responsible, restricted author list, language list, four purposes, three pagination layouts, two question-selection modes, two progress indicators. |
| Question shapes | Nine shapes: multiple lines text box, single line text box, numerical value, scale, date, date and time, multiple choice with one answer, multiple choice with several answers, and matrix with one or several choices per row. |
| Answer validation | Mandatory answers with a custom message, electronic mail format, minimum and maximum text length, minimum and maximum number, minimum and maximum date, minimum and maximum date and time, each with a custom message. |
| Conditional questions | A question appears only when one of its triggering answer options was chosen; unreachable triggers are detected, misplaced triggers are flagged, and answers to questions that should not have been shown are removed. |
| Scoring and certification | Four scoring modes, per-option scores that may be negative, a maximum obtainable score, a required percentage, a pass verdict, a certificate in six visual templates, a certificate message and an optional badge. |
| Access, attempts and time | Public link or invitation token, optional sign-in requirement, optional attempt cap per pool, optional overall countdown, optional per-question countdown, roaming back to earlier screens, restriction of the questionnaire to named officers. |
| Invitations | A composer that turns contacts and free electronic mail addresses into participations and personal messages, with an answer deadline and a choice between a fresh invitation and a resend. |
| Live sessions | Session code generation, a short public address, a waiting room, host-driven advance with a one-second grace delay, per-question countdowns, live answer counts, result bar graphs, correct-answer reveal, a leaderboard and a speed bonus. |
| Result analysis | Per-question vote counts, matrix cross-tabulation, scale distribution, numerical minimum, maximum and average, the five most common answers, per-section correct, partially correct, incorrect and unanswered counts, a global success rate, and answer filters that narrow the whole page. |
| Course definition | Two course types, four visibility levels, three enrolment policies, automatic enrolment by access group, prerequisite courses, tags grouped into families, a responsible, publication, a featured-content strategy and four message templates. |
| Course content | Six content categories with eleven derived subtypes, two sources, external metadata retrieval, free previews, duration estimation, downloadable resources, third-party embedding with view counters, likes, dislikes and comments. |
| Attendee progress | Per-content completion, per-course completion percentage, four attendee statuses, the next content to take, reputation points for a quiz that decrease with each attempt, reputation points for finishing a course, and a completion message. |
| Quizzes | Questions attached to a content item, answers marked correct or not, a per-answer comment revealed after submission, an attempt counter and a decreasing reward ladder. |
| Course certifications | A content item of the certification category pointing at a questionnaire, one attempt pool per enrolment, a certified marker on the enrolment, and automatic removal from the course plus a failure message when the last attempt fails. |
| Selling course access | A service product linked to the course, automatic enrolment when the order is confirmed, publication kept in step between course and product, a one-unit cart rule and a revenue figure per course. |
| Forum definition | Two modes, three privacy levels, five default orderings, two relevance parameters, thirteen shipped closing reasons, and forty-two reputation-point settings. |
| Forum posts | Questions and answers in one entity, five states, moderation queues, votes, favourites, tags, accepted answers, flagging, closing, reopening and conversion between answers and comments. |
| Reputation economy | Every action priced in reputation points, every grant recorded as a movement with a reason and a source, and a monthly consolidation of the movement ledger. |
| Recognition | Goal definitions with four computation modes, goals with a five-state lifecycle, challenges with five periodicities and two display modes, badges with four granting rules and a monthly sending cap, ranks unlocked by a balance, and periodic reports. |
| Public profile | A published profile with a biography, a city, a country and a personal site, the balance, the rank, the progress to the next rank, the badges earned and a member ranking. |
| Employee record | A résumé line written when an employee passes a certification or completes a course, with a validity period and an expiry status. |
| Reporting | Participation lists, detailed answer lists, course, content, attendee, review and quiz analyses, forum post analyses, a revenue analysis and an electronic-learning dashboard. |

---

## 3. Actors

| Actor | Description |
|---|---|
| Questionnaire Officer | Creates and edits questionnaires, sections, questions and answer options; sends invitations; opens and closes live sessions; reads participations, answers and statistics. Limited to questionnaires that name no restricted author or that name this officer. |
| Questionnaire Administrator | Everything the Questionnaire Officer does, on every questionnaire, plus creating and deleting individual participation answers. |
| Participant | Anybody answering a questionnaire, identified by a contact, by an electronic mail address, or anonymously by a nickname. |
| Live Session Host | A Questionnaire Officer who opens a session, advances the questions, shows the results and shows the leaderboard. |
| Learning Officer | Creates and edits courses and contents, publishes content on the courses for which the officer is responsible, adds and invites attendees, reads every course. |
| Learning Manager | Everything the Learning Officer does, on every course, plus deletion and the reporting menus. |
| Course Responsible | The user named on a course. Enrolled automatically, receives access requests as planned activities, is the only person who may publish on a training course, and signs the completion message. |
| Attendee | An enrolled contact. Opens contents, takes quizzes and certifications, likes, comments, reviews and accumulates reputation points. |
| Forum Visitor | Reads public forums without signing in. |
| Forum Member | A signed-in user who asks, answers, comments, votes, bookmarks and edits within the limits of the reputation points held. |
| Forum Moderator | A signed-in user whose balance reaches the moderation threshold of the forum. Validates pending posts, refuses them, handles flagged posts, marks posts offensive and reopens closed posts. |
| Site Designer | Creates forums, including private ones, and edits the public pages. |
| Recognition Administrator | Creates goal definitions, challenges, badges and ranks, starts and closes challenges, and grants badges by hand. |
| Human Resources Officer | Grants badges to employees and reads the challenges, goals and badges of the human resources pipeline. |
| Scheduled job runner | Executes the daily challenge check, the monthly reputation-movement consolidation and the automatic removal of expired course invitations. |

---

## 4. Entities this folder owns

### 4.1 Questionnaires

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Survey | `survey.survey` | `survey_survey` | The questionnaire: purpose, layout, access, scoring, certification and live-session settings. |
| Survey Question | `survey.question` | `survey_question` | One entry of the flat ordered list: a section or a question with its shape, validation and scoring. |
| Survey Answer Option | `survey.question.answer` | `survey_question_answer` | One proposed choice, one matrix column or one matrix row, with its score and correctness marker. |
| Survey Participation | `survey.user_input` | `survey_user_input` | One person's attempt: tokens, state, timing, score and pass verdict. |
| Survey Participation Answer | `survey.user_input.line` | `survey_user_input_line` | One stored answer, or one recorded skip, of one participation to one question. |
| Survey Invitation Wizard | `survey.invite` | transient `survey_invite` | The composer that turns contacts and addresses into participations and invitation messages. |

The dictionary name of `survey.question.answer` is Survey Label; this folder calls it Survey Answer
Option throughout, because the record is a proposed answer and not a caption.

### 4.2 Learning platform

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Course | `slide.channel` | `slide_channel` | The container of ordered content items with its visibility, enrolment policy and reward settings. |
| Course Content | `slide.slide` | `slide_slide` | One entry of the ordered list of a course: a section or a content item of one of six categories. |
| Course Enrolment | `slide.channel.partner` | `slide_channel_partner` | The link between one contact and one course: status, completion, invitation data. |
| Content Progress | `slide.slide.partner` | `slide_slide_partner` | The link between one contact and one content item: opened, completed, voted, quiz attempts. |
| Quiz Question | `slide.question` | `slide_question` | One question of the quiz attached to a content item. |
| Quiz Answer | `slide.answer` | `slide_answer` | One proposed answer of a quiz question, with its correctness marker and its comment. |
| Content Resource | `slide.slide.resource` | `slide_slide_resource` | An extra downloadable file or external link attached to a content item. |
| Content Embed Counter | `slide.embed` | `slide_embed` | A counter of loads of one content item embedded on one third-party address. |
| Content Tag | `slide.tag` | `slide_tag` | A globally unique free label attached to content items. |
| Course Tag | `slide.channel.tag` | `slide_channel_tag` | A label attached to courses, belonging to one tag family. |
| Course Tag Family | `slide.channel.tag.group` | `slide_channel_tag_group` | A named family of course tags, published or not, used to combine filters. |
| Course Invitation Wizard | `slide.channel.invite` | transient `slide_channel_invite` | The composer that adds contacts to a course as enrolled or as invited and messages them. |

The dictionary name of `slide.slide` is Slides and that of `slide.slide.partner` is a technical
phrase; this folder calls them Course Content and Content Progress, because those are the records'
business meaning and because the reference pages carry the dictionary names for tooling.

### 4.3 Forum

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Forum | `forum.forum` | `forum_forum` | The question-and-answer space with its mode, privacy, ordering and its whole reputation-point table. |
| Forum Post | `forum.post` | `forum_post` | A question (no parent) or an answer (with a parent), with its state, votes, favourites and tags. |
| Post Vote | `forum.post.vote` | `forum_post_vote` | One user's vote on one post: plus one, zero or minus one. |
| Post Closing Reason | `forum.post.reason` | `forum_post_reason` | A reusable reason for closing a post or for marking it offensive. |
| Forum Tag | `forum.tag` | `forum_tag` | A label scoped to one forum, with a count of the active posts carrying it. |

### 4.4 Recognition

| Full name | Transport name | Storage name | One-line purpose |
|---|---|---|---|
| Badge | `gamification.badge` | `gamification_badge` | A distinction that can be granted by people or by a challenge, with its granting rule and its counters. |
| User Badge | `gamification.badge.user` | `gamification_badge_user` | One grant of one badge to one user, with its sender, its challenge and its comment. |
| Badge Granting Wizard | `gamification.badge.user.wizard` | transient `gamification_badge_user_wizard` | The dialogue that grants a badge to a chosen user or employee. |
| Challenge | `gamification.challenge` | `gamification_challenge` | A set of goals assigned to a population, with a periodicity, rewards and a report schedule. |
| Challenge Line | `gamification.challenge.line` | `gamification_challenge_line` | One goal definition of a challenge with the value to reach. |
| Goal | `gamification.goal` | `gamification_goal` | One user's measurement against one definition over one period. |
| Goal Definition | `gamification.goal.definition` | `gamification_goal_definition` | What to measure, how to measure it, and whether higher or lower is better. |
| Goal Update Wizard | `gamification.goal.wizard` | transient `gamification_goal_wizard` | The dialogue that records the current value of a manually kept goal. |
| Karma Rank | `gamification.karma.rank` | `gamification_karma_rank` | A named tier unlocked at a minimum reputation-point balance. |
| Karma Movement | `gamification.karma.tracking` | `gamification_karma_tracking` | One recorded change of one user's reputation-point balance, with its source and its reason. |

Those thirty-three entities are the whole scope of this folder. Every one of them is specified field
by field in [`entities.md`](entities.md), and each carries a link to its generated reference page
under `../../references/entities/`.

## 5. Entities in the candidate list that this folder does not own

| Entity | Owning domain | Why |
|---|---|---|
| Product Attribute Category (`product.attribute.category`) | [Products and catalog](../products-and-catalog/README.md) | A grouping of product attributes used by the comparison feature of the storefront; it reaches the candidate list only because one storefront package is shared. |

Generic platform entities — stored records and their audit fields, sequences, scheduled jobs,
configuration parameters, printed reports, access rules, message templates and request routing —
are not owned here. They belong to the platform documents listed in section 8.

## 6. Entities of other domains that this folder extends

The extensions are specified here; the entities themselves belong to the domain named.

| Entity | Owning domain | What this domain adds |
|---|---|---|
| User (`res.users`) | [Identity and access](../identity-and-access/README.md) | The reputation-point balance, the movement list, the badge list, the three badge-level counters, the current rank and the next rank; the automatic enrolment into courses whose automatic-enrolment groups the user joins; the fields a user may read and write about themselves on the public profile; the destinations proposed after a rank change. |
| Access Group (`res.groups`) | [Identity and access](../identity-and-access/README.md) | Adding users to a group enrols them into every course that lists the group as an automatic-enrolment group. |
| Contact (`res.partner`) | [Contacts and organizations](../contacts-and-organizations/README.md) | The count of certifications passed, the same count aggregated over the contacts of a company, the courses followed, the courses completed and their two counters. |
| Contact Merge Wizard (`base.partner.merge.automatic.wizard`) | [Contacts and organizations](../contacts-and-organizations/README.md) | Merging is refused when two of the contacts being merged are enrolled in the same course. |
| Language (`res.lang`) | [Contacts and organizations](../contacts-and-organizations/README.md) | Deactivating a language removes it from the language list of every questionnaire. |
| Planned Activity (`mail.activity`) | [Messaging and activities](../messaging-and-activities/README.md) | The requesting contact carried by a course access request. |
| Message (`mail.message`) | [Messaging and activities](../messaging-and-activities/README.md) | A comment attached to a course content item carries the reputation-based permissions of the owning course, so the comment box can be shown or hidden without a second request. |
| Site, Site Menu (`website`, `website.menu`) | [Website and storefront](../website-and-storefront/README.md) | A forum counter per site, the key used to reach the external document storage service, two additional site-search types, the suggested course and forum menu entries. |
| Configuration Settings (`res.config.settings`) | [Platform foundation](../platform-foundation/) | Five learning-platform switches: the external document storage key, selling on the storefront, forums on courses, certifications on courses and mailing to attendees. |
| Product Template and Product Variant (`product.template`, `product.product`) | [Products and catalog](../products-and-catalog/README.md) | The service-tracking value `course`, its exclusion from the tracking blacklist, its tolerance of a zero price, the list of courses sold by the variant, and the sales description that lists them. |
| Sales Order and Sales Order Line (`sale.order`, `sale.order.line`) | [Sales](../sales/README.md) | Confirming an order enrols the customer into every paid course whose product appears on a line; a course line is forced to one unit and may not be reordered. |
| Lead (`crm.lead`) and Sales Team (`crm.team`) | [Customer relationship management](../customer-relationship-management/README.md) | The originating questionnaire on the lead, the questionnaires assigned to a team, and the creation of leads from lead-generating answer options. |
| Job Position (`hr.job`) and Application (`hr.applicant`) | [Recruitment](../recruitment/README.md) | The interview questionnaire attached to a job position and to an application; the fifth questionnaire purpose `recruitment`. |
| Employee (`hr.employee`), Public Employee (`hr.employee.public`) and Résumé Line (`hr.resume.line`) | [Human resources core](../human-resources-core/README.md) | The subscribed courses and the completion text on the employee; the course link, the course duration and the certification link on the résumé line; the résumé lines written when a course is completed or a certification is passed; the employee link on a granted badge. |
| Mailing (`mailing.mailing`) | [Marketing and mass mailing](../marketing-and-mass-mailing/) | The action that opens a new mailing addressed to the attendees of the selected courses. |
| Spreadsheet Dashboard (`spreadsheet.dashboard`) | [Spreadsheets and dashboards](../spreadsheets-and-dashboards/README.md) | The shipped electronic-learning dashboard, restricted to the Learning Manager group. |
| Rating (`rating.rating`) | [Website and storefront](../website-and-storefront/README.md) | Course reviews: the average shown in stars, the one-review-per-author rule and the reputation points a review grants. |

---

## 7. Reading order

1. **[`README.md`](README.md)** — this file: scope, capabilities, actors, entities owned and
   extended, reading order, dependencies and the list of every file in the folder.
2. **[`glossary.md`](glossary.md)** — every term defined. Read it first if any word above is
   unfamiliar; it also reconciles the vocabularies used for the same records.
3. **[`entities.md`](entities.md)** — the complete field tables. Everything else refers back to it.
4. **[`state-machines.md`](state-machines.md)** — the participation lifecycle, the session state,
   the post state, the attendee status, the goal state and the challenge state.
5. **[`calculations.md`](calculations.md)** — the scoring formulas, the speed bonus, the completion
   percentage, the relevance ranking, the reputation movements and the statistics.
6. **[`business-rules.md`](business-rules.md)** — every validation with its exact message, every
   permission check and every deletion protection, numbered `LSG-nnn`.
7. **[`workflows.md`](workflows.md)** — the end-to-end operational procedures.
8. **[`surveys.md`](surveys.md)** — the questionnaire engine in depth.
9. **[`courses.md`](courses.md)** — the learning platform in depth.
10. **[`forums.md`](forums.md)** — the forum in depth.
11. **[`gamification.md`](gamification.md)** — badges, goals, challenges, ranks and the
    reputation-point ledger in depth.
12. **[`accounting-effects.md`](accounting-effects.md)** — the boundary with the ledger.
13. **[`configuration.md`](configuration.md)** — settings, parameters, shipped records, groups,
    access rights, record rules, scheduled jobs and message templates.
14. **[`interfaces.md`](interfaces.md)** — menus, views, named operations, request endpoints,
    printed documents, messages and integrations.
15. **[`acceptance-criteria.md`](acceptance-criteria.md)** — numbered Given, When and Then
    scenarios with concrete numbers.

---

## 8. Files in this folder

| File | Content |
|---|---|
| [`README.md`](README.md) | Scope, capabilities, actors, the entities owned and extended, reading order, dependencies and this list. |
| [`entities.md`](entities.md) | Every entity of the domain, field by field, with types, defaults, derivations, constraints, validation messages, on-change behaviour and lifecycle. |
| [`state-machines.md`](state-machines.md) | Every state field: states with stored value, label and meaning; transition tables with triggers, guards and side effects; the exact refusals; a diagram per machine. |
| [`workflows.md`](workflows.md) | End-to-end procedures: building and sharing a questionnaire, answering it, running a live session, publishing a course, enrolling, progressing, certifying, selling, asking and moderating on a forum, and running the recognition jobs. |
| [`business-rules.md`](business-rules.md) | The numbered rule catalogue with exact messages, guards, permissions, uniqueness rules and content restrictions. |
| [`calculations.md`](calculations.md) | Every formula and algorithm with rounding, precision and worked numeric examples. |
| [`accounting-effects.md`](accounting-effects.md) | Why the domain posts no journal entry of its own and where the revenue of a sold course is recognised. |
| [`configuration.md`](configuration.md) | Settings, system parameters, shipped records, security groups, access rights, record rules, scheduled jobs, message templates and message subtypes. |
| [`interfaces.md`](interfaces.md) | Menus, views, named operations, request endpoints, printable documents, message templates, integrations, import and export. |
| [`acceptance-criteria.md`](acceptance-criteria.md) | Numbered Given, When and Then scenarios with concrete numbers. |
| [`glossary.md`](glossary.md) | Every term of the domain, defined. |
| [`surveys.md`](surveys.md) | The questionnaire engine in depth: purposes, layouts, shapes, validation, conditional display, scoring, certification, attempts, timing, invitations, live sessions and statistics. |
| [`courses.md`](courses.md) | The learning platform in depth: types, visibility, enrolment, categories and subtypes, ordering, publication, completion, quizzes, certifications, resources, embeds, selling and the attendee pages. |
| [`forums.md`](forums.md) | The forum in depth: modes, privacy, post states, the whole reputation-point table, voting, accepting, moderation, tags, favourites, ordering and relevance. |
| [`gamification.md`](gamification.md) | The recognition machinery in depth: goal definitions and their four computation modes, goals, challenges, badges, ranks, the movement ledger and its consolidation. |

---

## 9. Dependencies on other domains

These domains must exist before this one can work.

| Domain | Dependency |
|---|---|
| [Identity and access](../identity-and-access/README.md) | Users, access groups, the anonymous visitor account, the portal account, the self-registration scope, and the user record that carries the reputation-point balance and the rank. |
| [Contacts and organizations](../contacts-and-organizations/README.md) | Contacts identify participants, attendees and post authors; languages drive the questionnaire language list and the translation of every message. |
| [Messaging and activities](../messaging-and-activities/README.md) | The discussion thread on questionnaires, participations, courses, contents, forums, posts, badges and challenges; message subtypes; message templates and their rendering; outgoing messages; planned activities for course access requests. |
| [Website and storefront](../website-and-storefront/README.md) | The public pages, the site record and its address, the publication markers, the cover properties, the search integration, the rating component behind course reviews, and the public profile pages. |
| [Customer portal](../customer-portal/) | The portal discussion component used for comments and reviews on courses and contents. |
| [Platform foundation](../platform-foundation/) | Stored records and their audit fields, attachments, configuration parameters, scheduled jobs, printed reports, request routing and the configuration settings screen. |
| [Products and catalog](../products-and-catalog/README.md) | The service product that represents paid access to a course. Required only when courses are sold. |
| [Sales](../sales/README.md) | The order whose confirmation enrols the buyer, and the revenue analysis behind the per-course revenue figure. Required only when courses are sold. |
| [Customer relationship management](../customer-relationship-management/README.md) | The lead created from a lead-generating answer option, its team, its source and its medium. Required only when lead generation is used. |
| [Recruitment](../recruitment/README.md) | The interview questionnaire attached to a job position and to an application. Required only when recruitment interviews are used. |
| [Human resources core](../human-resources-core/README.md) | The employee record, the résumé line and its types. Required only when skills tracking is used. |
| [Marketing and mass mailing](../marketing-and-mass-mailing/) | The mailing whose recipients are the attendees of a course. Required only when attendee mailing is used. |
| [Spreadsheets and dashboards](../spreadsheets-and-dashboards/README.md) | The dashboard container that holds the shipped electronic-learning dashboard. Required only when that dashboard is used. |
| [Accounts receivable](../accounts-receivable/README.md) and [General ledger](../general-ledger/README.md) | The customer invoice and the journal entry that recognise the revenue of a sold course. Reached only through the selling domain. |

Platform documents this folder relies on:
[`../../overview/security-model.md`](../../overview/security-model.md),
[`../../overview/entity-and-field-system.md`](../../overview/entity-and-field-system.md),
[`../../overview/views-and-actions.md`](../../overview/views-and-actions.md),
[`../../overview/messaging-model.md`](../../overview/messaging-model.md),
[`../../runtime/scheduled-jobs.md`](../../runtime/scheduled-jobs.md),
[`../../runtime/request-lifecycle.md`](../../runtime/request-lifecycle.md),
[`../../runtime/sessions-and-authentication.md`](../../runtime/sessions-and-authentication.md),
[`../../runtime/notification-bus.md`](../../runtime/notification-bus.md),
[`../../runtime/report-rendering.md`](../../runtime/report-rendering.md),
[`../../runtime/mail-gateway.md`](../../runtime/mail-gateway.md),
[`../../runtime/attachments-and-file-store.md`](../../runtime/attachments-and-file-store.md),
[`../../data/domain-model.md`](../../data/domain-model.md),
[`../../data/persistence-identity-and-values.md`](../../data/persistence-identity-and-values.md),
[`../../interfaces/endpoint-catalog.md`](../../interfaces/endpoint-catalog.md),
[`../../interfaces/service-layer.md`](../../interfaces/service-layer.md),
[`../../interfaces/report-and-export-documents.md`](../../interfaces/report-and-export-documents.md).
