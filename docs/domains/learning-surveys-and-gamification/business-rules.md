# Learning, Questionnaires and Recognition — Business rules

Every validation, constraint, invariant, permission check and locking rule of the domain, numbered
with the stable prefix `LSG`. Each rule states the entity it applies to, the moment it runs, the
condition that makes it fail and the exact text the system shows. A message reproduced between
backticks is shown verbatim; a placeholder between angle brackets, or written as a percent form, is
replaced by the value described in the rule.

## Index of rule identifiers

| Range | Subject |
|---|---|
| `LSG-001` to `LSG-020` | Questionnaire authoring: structure, scoring, certification, session settings |
| `LSG-021` to `LSG-026` | Creating a participation |
| `LSG-027` to `LSG-031` | Checking a questionnaire before an invitation |
| `LSG-032` to `LSG-045` | Answering: validation of a submitted answer |
| `LSG-046` to `LSG-052` | Live sessions |
| `LSG-053` to `LSG-060` | Invitations and access to a participation |
| `LSG-061` to `LSG-075` | Course definition and enrolment |
| `LSG-076` to `LSG-090` | Course content, quizzes, resources and certifications |
| `LSG-091` to `LSG-115` | Forum: posting, editing, moderating, voting, tagging |
| `LSG-116` to `LSG-128` | Recognition: badges, challenges, goals, ranks and movements |
| `LSG-129` to `LSG-136` | Cross-domain rules |

---

## 1. Questionnaire authoring

<a id="lsg-001"></a>
**LSG-001 — An access token is unique.** On Survey, at every save. The database constraint
`_access_token_unique` refuses a duplicate with `Access token should be unique`.

<a id="lsg-002"></a>
**LSG-002 — A session code is unique.** On Survey, at every save. The constraint
`_session_code_unique` refuses a duplicate with `Session code should be unique`.

<a id="lsg-003"></a>
**LSG-003 — A certification needs a scoring mode.** On Survey, at every save. When
`certification` is true and `scoring_type` is `no_scoring`, the constraint `_certification_check`
refuses the save with `You can only create certifications for surveys that have a scoring
mechanism.`

<a id="lsg-004"></a>
**LSG-004 — The required percentage lies between zero and one hundred.** On Survey, at every save.
The constraint `_scoring_success_min_check` refuses any other value with `The percentage of success
has to be defined between 0 and 100.`

<a id="lsg-005"></a>
**LSG-005 — A time-limited questionnaire needs a positive limit.** On Survey, at every save. The
constraint `_time_limit_check` refuses a missing or non-positive limit with `The time limit needs to
be a positive number if the survey is time limited.`

<a id="lsg-006"></a>
**LSG-006 — An attempt-limited questionnaire needs a positive cap.** On Survey, at every save. The
constraint `_attempts_limit_check` refuses a missing or non-positive cap with `The attempts limit
needs to be a positive number if the survey has a limited number of attempts.`

<a id="lsg-007"></a>
**LSG-007 — One badge per questionnaire.** On Survey, at every save. The constraint `_badge_uniq`
refuses two questionnaires sharing a certification badge with `The badge for each survey should be
unique!`

<a id="lsg-008"></a>
**LSG-008 — Rewarding quick answers needs a default limit.** On Survey, at every save. The
constraint `_session_speed_rating_has_time_limit` refuses a missing or non-positive default with `A
positive default time limit is required when the session rewards quick answers.`

<a id="lsg-009"></a>
**LSG-009 — Roaming and page-by-page answers cannot be combined.** On Survey, when `scoring_type` or
`users_can_go_back` is written. When the mode is `scoring_with_answers_after_page` and roaming is
on, the save is refused with `Combining roaming and "Scoring with answers after each page" is not
possible; please update the following surveys: - %(survey_names)s`, the placeholder being one line
per offending questionnaire, each starting with a hyphen and holding the title.

<a id="lsg-010"></a>
**LSG-010 — A restricted questionnaire keeps its responsible inside the restriction.** On Survey,
when `user_id` or `restrict_user_ids` is written. When the restricted list is non-empty, a
responsible is set, that responsible is only a Questionnaire Officer and is not in the list, the
save is refused with `The access of the following surveys is restricted. Make sure their responsible
still has access to it:  %(survey_names)s`, the placeholder being one line per questionnaire with
its title and its responsible. The form avoids the refusal by appending the responsible to the list
as soon as either field changes.

<a id="lsg-011"></a>
**LSG-011 — A section carries no question shape.** On Survey Question, when `is_page` is written.
The save is refused with `Question type should be empty for these pages: %s`, the placeholder being
the comma-separated titles.

<a id="lsg-012"></a>
**LSG-012 — Text-length bounds are ordered and non-negative.** On Survey Question, at every save.
Three constraints apply: `_positive_len_min` and `_positive_len_max` refuse a negative bound with `A
length must be positive!`, and `_validation_length` refuses an inverted pair with `Max length cannot
be smaller than min length!`

<a id="lsg-013"></a>
**LSG-013 — Numeric bounds are ordered.** On Survey Question, at every save. The constraint
`_validation_float` refuses an inverted pair with `Max value cannot be smaller than min value!`

<a id="lsg-014"></a>
**LSG-014 — Date bounds are ordered.** On Survey Question, at every save. `_validation_date` refuses
an inverted pair with `Max date cannot be smaller than min date!` and `_validation_datetime` refuses
an inverted pair of instants with `Max datetime cannot be smaller than min datetime!`

<a id="lsg-015"></a>
**LSG-015 — A per-question score is never negative.** On Survey Question, at every save. The
constraint `_positive_answer_score` refuses a negative question score with `An answer score for a
non-multiple choice question cannot be negative!` Option scores may be negative; this rule governs
the question score only.

<a id="lsg-016"></a>
**LSG-016 — A scored date question carries its correct answer.** On Survey Question, at every save.
`_scored_date_have_answers` refuses a scored date question with no correct date with `All "Is a
scored question = True" and "Question Type: Date" questions need an answer`, and
`_scored_datetime_have_answers` refuses the same for a date-and-time question with `All "Is a scored
question = True" and "Question Type: Datetime" questions need an answer`.

<a id="lsg-017"></a>
**LSG-017 — A scale is a growing range inside zero and ten.** On Survey Question, at every save. The
constraint `_scale` refuses any other configuration with `The scale must be a growing non-empty
range between 0 and 10 (inclusive)`.

<a id="lsg-018"></a>
**LSG-018 — A time-limited question needs a positive limit.** On Survey Question, at every save. The
constraint `_is_time_limited_have_time_limit` refuses a missing or non-positive limit with `All
time-limited questions need a positive time limit`.

<a id="lsg-019"></a>
**LSG-019 — An answer option is attached to exactly one question.** On Survey Answer Option, when
`question_id` or `matrix_question_id` is written. Both filled, or neither filled, is refused with `A
label must be attached to only one question.`

<a id="lsg-020"></a>
**LSG-020 — An answer option carries a value or a picture.** On Survey Answer Option, at every save.
The constraint `_value_not_empty` refuses a record with neither with `Suggested answer value must
not be empty (a text and/or an image must be provided).`

<a id="lsg-020a"></a>
**LSG-020a — Questions cannot be deleted during a live session.** On Survey Question, at deletion.
When a live session is in progress on the questionnaire, the deletion is refused with `You cannot
delete questions from surveys "%(survey_names)s" while live sessions are in progress.`

---

## 2. Creating a participation

The six guards below run in this order whenever a participation is about to be created, from any
source: the invitation composer, a public landing, a certification content item, a retry or a test.

<a id="lsg-021"></a>
**LSG-021 — A closed questionnaire accepts no new attempt.** When the questionnaire is archived and
the attempt is not a test entry, the creation is refused with `Creating token for closed/archived
surveys is not allowed.`

<a id="lsg-022"></a>
**LSG-022 — Signing in is required and self-registration is allowed.** When the questionnaire
requires signing in, the platform allows self-registration, and neither a user nor a contact is
known, the creation is refused with `Creating token for external people is not allowed for surveys
requesting authentication.`

<a id="lsg-023"></a>
**LSG-023 — Signing in is required and self-registration is closed.** When the questionnaire
requires signing in, the platform does not allow self-registration and no non-anonymous user is
known, the creation is refused with the same text: `Creating token for external people is not
allowed for surveys requesting authentication.`

<a id="lsg-024"></a>
**LSG-024 — An internal questionnaire accepts only internal users.** When the questionnaire is
internal and the caller is not an internal user, the creation is refused with `Creating token for
anybody else than employees is not allowed for internal surveys.`

<a id="lsg-025"></a>
**LSG-025 — No attempt left.** When the attempt allowance is checked and
[`LSG-CALC-006`](calculations.md#lsg-calc-006) yields zero or fewer attempts left, the creation is
refused with `No attempts left.` The same check runs again at each submit, so a participant cannot
open several attempts in advance and spend them later.

<a id="lsg-026"></a>
**LSG-026 — Only a reader may create a test attempt.** When a test attempt is requested by somebody
who may not read the questionnaire, the creation is refused with `Creating test token is not allowed
for you.`

---

## 3. Checking a questionnaire before an invitation

These five checks run in this order when the invitation composer is opened. The first failure stops
the sequence.

<a id="lsg-027"></a>
**LSG-027 — A questionnaire with no question cannot be shared.** Refused with `You cannot send an
invitation for a survey that has no questions.`

<a id="lsg-028"></a>
**LSG-028 — A scored questionnaire must be able to award points.** When the questionnaire is scored
and its maximum obtainable score, computed by
[`LSG-CALC-001`](calculations.md#lsg-calc-001), is zero or lower, the check is refused with `A scored
survey needs at least one question that gives points. Please check answers and their scores.`

<a id="lsg-029"></a>
**LSG-029 — A section-per-page questionnaire needs sections.** When the layout is
`page_per_section` and the questionnaire holds no section, the check is refused with `You cannot
send an invitation for a "One page per section" survey if the survey has no sections.`

<a id="lsg-030"></a>
**LSG-030 — A section-per-page questionnaire needs non-empty sections.** When the layout is
`page_per_section` and no section holds a question, the check is refused with `You cannot send an
invitation for a "One page per section" survey if the survey only contains empty sections.`

<a id="lsg-031"></a>
**LSG-031 — A closed questionnaire cannot be shared.** When the questionnaire is archived, the check
is refused with `You cannot send invitations for closed surveys.`

---

## 4. Answering: validation of a submitted answer

Validation runs on submit, question by question, and produces at most one message per question. When
at least one message is produced and no countdown has run out, the whole submit is refused and
nothing is stored. When a countdown has run out, the messages are ignored, the valid answers are
stored and the participation is completed.

<a id="lsg-032"></a>
**LSG-032 — A text answer is trimmed before it is checked.** Leading and trailing spaces are removed
from a text answer before any other rule applies.

<a id="lsg-033"></a>
**LSG-033 — A mandatory non-choice question needs an answer.** When the answer is empty, the shape
is neither `simple_choice` nor `multiple_choice`, the question is mandatory and roaming is off, the
message shown is the question's own error message, or `This question requires an answer.` when that
field is empty. When the question is not mandatory, or roaming is on, no message is produced and the
skip is recorded.

<a id="lsg-034"></a>
**LSG-034 — A single-line text answer that must be an electronic mail address.** When
`validation_email` is true and the answer is not of the form local part, at sign, host, dot,
extension, the message is `This answer must be an email address`.

<a id="lsg-035"></a>
**LSG-035 — A single-line text answer of the required length.** When `validation_required` is true
and the number of characters is not between the minimum and the maximum inclusive, the message is
the question's validation error message, or `The answer you entered is not valid.` when that field is
empty.

<a id="lsg-036"></a>
**LSG-036 — A numerical answer must be a number.** When the answer cannot be read as a number, the
message is `This is not a number`.

<a id="lsg-037"></a>
**LSG-037 — A numerical answer inside its bounds.** When `validation_required` is true and the
number is not between the minimum and the maximum inclusive, the message is the question's
validation error message, or `The answer you entered is not valid.`

<a id="lsg-038"></a>
**LSG-038 — A date or date-and-time answer must be a date.** When the answer cannot be read as a
date, respectively as an instant, the message is `This is not a date`.

<a id="lsg-039"></a>
**LSG-039 — A date or date-and-time answer inside its bounds.** When `validation_required` is true
and the value falls outside the configured bounds, the message is the question's validation error
message, or `The answer you entered is not valid.` Each bound applies only when it is set: with both
bounds the value must lie between them inclusive; with only a lower bound the value must be at or
after it; with only an upper bound the value must be at or before it.

<a id="lsg-040"></a>
**LSG-040 — A mandatory choice question needs a choice.** The selected options are counted, and one
is added when a comment was typed and the question counts comments as answers. When the count is
zero, the question is mandatory and roaming is off, the message is the question's own error message,
or `This question requires an answer.`

<a id="lsg-041"></a>
**LSG-041 — A one-answer choice question accepts one choice.** When more than one option is selected
on a `simple_choice` question, the message is `For this question, you can only select one answer.`

<a id="lsg-042"></a>
**LSG-042 — A mandatory matrix question needs every row answered.** When the question is mandatory
and the number of answered rows differs from the number of rows, the message is the question's own
error message, or `This question requires an answer.`

<a id="lsg-043"></a>
**LSG-043 — A mandatory scale question needs a value.** When roaming is off, the question is
mandatory and no value was given, the message is the question's own error message, or `This question
requires an answer.`

<a id="lsg-044"></a>
**LSG-044 — An answer row is a skip or an answer, never both.** On Survey Participation Answer, at
every save. When the skip marker equals whether an answer type is filled, the save is refused with
`A question can either be skipped or answered, not both.`

<a id="lsg-045"></a>
**LSG-045 — An answer row carries the value field of its type.** On Survey Participation Answer, at
every save. When the value field matching the answer type is empty, the save is refused with `The
answer must be in the right type`. Two values are tolerated: a numerical answer whose value rounds
to zero at six decimal places, and a scale answer whose value is zero.

<a id="lsg-045a"></a>
**LSG-045a — An existing answer is not silently overwritten.** When an answer already exists for the
question and the caller did not ask for an overwrite, the save is refused with `This answer cannot be
overwritten.`

---

## 5. Live sessions

<a id="lsg-046"></a>
**LSG-046 — Only a Questionnaire Officer opens a session.** Opening a live session by somebody
outside the group is refused with `Only survey users can manage sessions.` The write itself then
runs with elevated rights, so an officer may run a session on a questionnaire owned by somebody
else.

<a id="lsg-047"></a>
**LSG-047 — Only a Questionnaire Officer closes a session.** The same refusal, `Only survey users
can manage sessions.`

<a id="lsg-048"></a>
**LSG-048 — A certification never runs as a live session.** The derived marker that governs the
session buttons is true only when the purpose is `live_session` or `custom` **and** the
questionnaire is not a certification, so the session actions are not offered at all on a
certification.

<a id="lsg-049"></a>
**LSG-049 — Joining by a code that matches nothing.** The code page answers with the status
`survey_wrong` when no questionnaire carries the typed code, and with the same status when the code
matches a certification.

<a id="lsg-050"></a>
**LSG-050 — Joining a session that is not open.** When the code matches a questionnaire whose
session is not open, the page answers with the status `survey_session_not_launched`; for a
Questionnaire Officer the answer additionally carries the questionnaire identifier, so the page can
offer to launch the session.

<a id="lsg-051"></a>
**LSG-051 — Random selection is ignored in a session.** Every attendee is shown the questions the
host presents; the per-participation random draw is not applied.

<a id="lsg-052"></a>
**LSG-052 — Closing a session skips the per-participation side effects.** Because the state of every
participation is written directly, the certificate message, the badge evaluation and the completion
notice do not run for a closed session. The lead-generation package compensates by creating the
leads of the session explicitly at that moment. **Compatibility finding:** a certification could not
be run as a session anyway, so no certificate is lost, but a scored session questionnaire that also
awards a badge would not award it. A corrected behaviour would route the mass completion through the
same completion procedure as an individual submit.

---

## 6. Invitations and access to a participation

<a id="lsg-053"></a>
**LSG-053 — At least one usable recipient.** On the Survey Invitation Wizard, when the invitation is
sent. When neither a contact nor a usable address remains, the operation is refused with `Please
enter at least one valid recipient.`

<a id="lsg-054"></a>
**LSG-054 — A sender address is required.** On the Survey Invitation Wizard and on the Course
Invitation Wizard. When no sender address can be determined — neither the rendered sender of the
template nor the author's formatted address — the operation is refused with `Unable to post message,
please configure the sender's email address.`

<a id="lsg-055"></a>
**LSG-055 — Free addresses are refused on a sign-in questionnaire.** On the Survey Invitation
Wizard, when free addresses are typed. When the questionnaire requires signing in and
self-registration is not allowed, the form is refused with `This survey does not allow external
people to participate. You should create user accounts or update survey access mode accordingly.`

<a id="lsg-056"></a>
**LSG-056 — Typed addresses must be valid.** On the Survey Invitation Wizard. When any typed entry
is not a valid address, the form is refused with `Some emails you just entered are incorrect: %s`,
the placeholder being the comma-separated bad entries. When every entry is valid the field is
rewritten as one normalised address per line.

<a id="lsg-057"></a>
**LSG-057 — Chosen contacts must have accounts on a sign-in questionnaire.** On the Survey
Invitation Wizard. When the questionnaire requires signing in, self-registration is not allowed and
some chosen contacts have no user account, the form is refused with `The following recipients have
no user account: %s. You should create user accounts for them or allow external signup in
configuration.`

<a id="lsg-058"></a>
**LSG-058 — A participation created by the composer skips the attempt check.** Participations
created for the recipients of an invitation are created without checking the remaining attempts, so
an officer can always re-invite; the allowance is enforced when the participant submits.

<a id="lsg-059"></a>
**LSG-059 — A deadline in the past closes a participation.** Every access to a participation whose
deadline has passed is refused and the closed-or-expired page is shown.

<a id="lsg-060"></a>
**LSG-060 — A participation belongs to its owner.** Access is refused, with the access-error page,
when the visitor is anonymous, the participation names a contact and the token did not come from the
address; and when the visitor is signed in and the participation belongs to another contact. The
browser cookie that remembers a participation for one day is ignored when it points at a
participation belonging to somebody else or at a participation that no longer exists.

---

## 7. Course definition and enrolment

<a id="lsg-061"></a>
**LSG-061 — Attendee-only visibility implies invitation.** On Course, at every save. The constraint
`_check_enroll` refuses any other combination with `The Enroll Policy should be set to 'On
Invitation' when visibility is set to 'Course Attendees'`.

<a id="lsg-062"></a>
**LSG-062 — A paid course carries a product.** On Course, at every save, when the course-selling
package is installed. The constraint `_product_id_check` refuses a paid course with no product with
`Product is required for on payment channels.`

<a id="lsg-063"></a>
**LSG-063 — One course per forum.** On Course, at every save, when the course-forum package is
installed. The constraint `_forum_uniq` refuses a second course on the same forum with `Only one
course per forum!`

<a id="lsg-064"></a>
**LSG-064 — One enrolment per contact and course.** On Course Enrolment, at every save. The
constraint `_channel_partner_uniq` refuses a duplicate with `A partner membership to a channel must
be unique!`

<a id="lsg-065"></a>
**LSG-065 — A completion is a percentage.** On Course Enrolment, at every save. The constraint
`_check_completion` refuses a value outside zero to one hundred with `The completion of a channel is
a percentage and should be between 0% and 100.`

<a id="lsg-066"></a>
**LSG-066 — Adding attendees requires write access.** On Course, when attendees are added.
Open-enrolment courses always pass; for the others the acting user must have write access on the
course. When the caller asked for the strict check and a course was filtered out, the operation is
refused with `You are not allowed to add members to this course. Please contact the course
responsible or an administrator.`

<a id="lsg-067"></a>
**LSG-067 — Only a signed-in visitor may request access.** The request answers `You have to sign in
before` for an anonymous visitor.

<a id="lsg-068"></a>
**LSG-068 — Access is requested only on a published course.** The request answers `Course not
published yet` when the course is unpublished.

<a id="lsg-069"></a>
**LSG-069 — An enrolled attendee does not request access.** The request answers `Already member`
when the visitor is already enrolled.

<a id="lsg-070"></a>
**LSG-070 — Access is requested only on an invitation-only course.** The request answers that there
is nothing to do when the enrolment policy is not `invite`.

<a id="lsg-071"></a>
**LSG-071 — One access request per contact.** When a to-do activity already names that contact as
the requesting contact on that course, nothing is created and the answer is `Already Requested`.
Otherwise one to-do activity is scheduled on the course, assigned to the responsible, summarised
`Access Request`, with the note `<visitor name> is requesting access to this course.`

<a id="lsg-072"></a>
**LSG-072 — Granting and refusing access close the activity.** Granting enrols the contact and
closes the activity with the feedback `Access Granted`; refusing closes it with the feedback `Access
Refused` and enrols nobody.

<a id="lsg-073"></a>
**LSG-073 — At least one recipient on the course invitation composer.** Refused with `Please select
at least one recipient.`

<a id="lsg-074"></a>
**LSG-074 — Sharing a course needs a template.** Refused with `Impossible to send emails. Select a
"Channel Share Template" for courses %(course_names)s first`.

<a id="lsg-075"></a>
**LSG-075 — A course review is unique per author.** Posting a second message carrying a rating on
the same course by the same author is refused with `Only a single review can be posted per course.`
Posting one at all requires the course review threshold, otherwise `Not enough karma to review`.

---

## 8. Course content, quizzes, resources and certifications

<a id="lsg-076"></a>
**LSG-076 — A content item holds one payload.** On Course Content, at every save. The constraint
`_exclusion_html_content_and_url` refuses a record holding both an authored body and an external
address with `A slide is either filled with a url or HTML content. Not both.`

<a id="lsg-077"></a>
**LSG-077 — A certification content points at a questionnaire.** On Course Content, at every save,
when the course-certification package is installed. The constraint `_check_survey_id` refuses a
certification with no questionnaire with `A slide of type 'certification' requires a certification.`

<a id="lsg-078"></a>
**LSG-078 — A certification is never a free preview.** On Course Content, at every save. The
constraint `_check_certification_preview` refuses the combination with `A slide of type
certification cannot be previewed.`

<a id="lsg-079"></a>
**LSG-079 — Publishing is restricted.** On Course Content. A reader who may not publish on the
course is refused with `Publishing is restricted to the responsible of training courses or members
of the publisher group for documentation courses`.

<a id="lsg-080"></a>
**LSG-080 — Commenting a content item requires reputation points.** Refused with `Not enough karma
to comment`. A message of any other kind — a notification, a system note — is not gated.

<a id="lsg-081"></a>
**LSG-081 — Marking a content viewed requires enrolment.** Refused with `You cannot mark a slide as
viewed if you are not among its members.`

<a id="lsg-082"></a>
**LSG-082 — Marking a content completed requires the permission.** Refused with `You cannot mark a
slide as completed if you are not among its members.` Through the public endpoint the refusal is
`Slide with questions must be marked as done when submitting all good answers ` when the content
carries questions, and `This slide can not be marked as completed.` otherwise.

<a id="lsg-083"></a>
**LSG-083 — Marking a content uncompleted requires the permission.** Refused with `You cannot mark a
slide as uncompleted if you are not among its members.` Through the public endpoint the refusal is
`This slide can not be marked as uncompleted.`

<a id="lsg-084"></a>
**LSG-084 — Quiz points require enrolment and publication.** Refused with `You cannot mark a slide
quiz as completed if you are not among its members or it is unpublished.` when points are being
granted, and with `You cannot mark a slide quiz as not completed if you are not among its members or
it is unpublished.` when they are being taken back.

<a id="lsg-085"></a>
**LSG-085 — A quiz question has a correct and an incorrect answer.** On Quiz Question, when the
answers are written. A question with no correct answer, or with only correct answers, carries the
error text `This question must have at least one correct answer and one incorrect answer.` and the
save is refused with `All questions must have at least one correct answer and one incorrect answer:
 %s`, the placeholder being one line per offending question as `- <content name>: <question>`.

<a id="lsg-086"></a>
**LSG-086 — A quiz submission covers every question.** When the set of questions covered by the
chosen answers is not exactly the set of questions of the content item, the submission is refused
with the status `slide_quiz_incomplete`. An anonymous visitor is refused with `public_user`, and an
already completed content with `slide_quiz_done`, which also drops the answers saved in the browser
session for that content item.

<a id="lsg-087"></a>
**LSG-087 — One progress record per contact and content.** On Content Progress, at every save. The
constraint `_slide_partner_uniq` refuses a duplicate with `A partner membership to a slide must be
unique!`

<a id="lsg-088"></a>
**LSG-088 — A vote is minus one, zero or one.** On Content Progress, at every save. The constraint
`_check_vote` refuses any other value with `The vote must be 1, 0 or -1.`

<a id="lsg-089"></a>
**LSG-089 — A resource is a file or a link, never both.** On Content Resource, at every save. The
constraint `_check_url` refuses a link resource with no link with `A resource of type url must
contain a link.` and `_check_file_type` refuses a file resource carrying a link with `A resource of
type file cannot contain a link.` Writing a payload on a link resource is refused with `Resource
%(resource_name)s is a link and should not contain a data file`.

<a id="lsg-090"></a>
**LSG-090 — A questionnaire used as a course certification cannot be deleted.** On Survey, at
deletion. The deletion is refused with `Uh-oh! You can’t delete surveys used as a Course
Certification! Otherwise, students might think diplomas just grow on trees. The courses that need
them are: %s`, the placeholder being one line per questionnaire as `- <certification title> (Courses
- <course names>)`.

<a id="lsg-090a"></a>
**LSG-090a — A content tag name is unique.** On Content Tag, at every save. The constraint
`_slide_tag_unique` refuses a duplicate with `A tag must be unique!`

<a id="lsg-090b"></a>
**LSG-090b — Voting on a content item.** The like endpoint refuses an anonymous visitor with
`public_user`, a non-member with `channel_membership_required`, a course that does not allow
comments with `channel_comment_disabled`, and a reader below the vote threshold with
`channel_karma_required`.

<a id="lsg-090c"></a>
**LSG-090c — Sharing one content item needs a template.** Refused with `Impossible to send emails.
Select a "Share Template" for courses %(course_names)s first`.

---

## 9. Forum

Every message in this section carries the numeric value of the applicable threshold of the forum as
its placeholder, printed as a whole number. An administrator satisfies every threshold.

<a id="lsg-091"></a>
**LSG-091 — Asking a question requires reputation points.** Refused with `%d karma required to
create a new question.`

<a id="lsg-092"></a>
**LSG-092 — Answering requires reputation points.** Refused with `%d karma required to answer a
question.`

<a id="lsg-093"></a>
**LSG-093 — A closed or archived question accepts no answer.** Refused with `Posting answer on a
[Deleted] or [Closed] question is not possible.`

<a id="lsg-094"></a>
**LSG-094 — The post hierarchy is not recursive.** On Forum Post, when the parent is written. The
save is refused with `You cannot create recursive forum posts.`

<a id="lsg-095"></a>
**LSG-095 — Editing a post requires reputation points.** Refused with `%d karma required to edit a
post.` The threshold is the own-post one when the reader wrote the post and the all-posts one
otherwise.

<a id="lsg-096"></a>
**LSG-096 — Retagging requires reputation points.** Refused with `%d karma required to retag.` when
the written tag set differs from the stored one.

<a id="lsg-097"></a>
**LSG-097 — Closing or reopening requires reputation points.** Refused with `%d karma required to
close or reopen a post.`

<a id="lsg-098"></a>
**LSG-098 — Archiving or reactivating requires reputation points.** Refused with `%d karma required
to delete or reactivate a post.`

<a id="lsg-099"></a>
**LSG-099 — Deleting a post requires reputation points.** Refused with `%d karma required to unlink
a post.`

<a id="lsg-100"></a>
**LSG-100 — Accepting or refusing an answer requires reputation points.** Refused with `%d karma
required to accept or refuse an answer.` The threshold is the own-question one when the reader asked
the parent question and the all-questions one otherwise.

<a id="lsg-101"></a>
**LSG-101 — Flagging requires reputation points.** Refused with `%d karma required to flag a post.`

<a id="lsg-102"></a>
**LSG-102 — Validating a pending post requires moderation.** Refused with `%d karma required to
validate a post.`

<a id="lsg-103"></a>
**LSG-103 — Refusing a pending post requires moderation.** Refused with `%d karma required to refuse
a post.`

<a id="lsg-104"></a>
**LSG-104 — Marking a post offensive requires moderation.** Refused with `%d karma required to mark
a post as offensive.`

<a id="lsg-105"></a>
**LSG-105 — Converting an answer into a comment requires reputation points.** Refused with `%d karma
required to convert an answer to a comment.` The operation returns without doing anything when the
post has no parent, because a question cannot become a comment.

<a id="lsg-106"></a>
**LSG-106 — Converting a comment into an answer requires reputation points.** When the acting user
wrote the comment and the own threshold is lower than the all threshold, the refusal is `%d karma
required to convert your comment to an answer.`; otherwise it is `%d karma required to convert a
comment to an answer.` The operation returns without doing anything when the comment has no author,
when the author has no user account, or when the author already answered that question.

<a id="lsg-107"></a>
**LSG-107 — Deleting a comment requires reputation points.** Refused with `%d karma required to
delete a comment.` A comment that does not belong to the named post, or that is not attached to a
post at all, is skipped and answered with a negative result.

<a id="lsg-108"></a>
**LSG-108 — Commenting requires reputation points.** Refused with `%d karma required to comment.`

<a id="lsg-109"></a>
**LSG-109 — Pictures and links require the editor threshold.** When the submitted content contains a
picture element, an anchor with an address, or an inline style loading a background picture from an
address, and the author is below the editor threshold, the write is refused with `%d karma required
to post an image or link.` Below the follow threshold every anchor is rewritten so its address is
preserved and a no-follow marker is added; that rewriting never refuses the write.

<a id="lsg-110"></a>
**LSG-110 — Creating a tag requires reputation points.** Refused with `%d karma required to create a
new Tag.` Typed entries below the threshold are dropped silently on the post form; a direct creation
raises the refusal.

<a id="lsg-111"></a>
**LSG-111 — A tag name is unique inside its forum.** On Forum Tag, at every save. The constraint
`_name_uniq` refuses a duplicate with `Tag name already exists!` The same name may exist on two
forums as two records.

<a id="lsg-112"></a>
**LSG-112 — One vote per user and post.** On Post Vote, at every save. The constraint `_vote_uniq`
refuses a duplicate with `Vote already exists!`

<a id="lsg-113"></a>
**LSG-113 — Nobody votes on their own post.** On Post Vote. Refused with `It is not allowed to vote
for its own post.` The public endpoints answer the status `own_post` instead of raising.

<a id="lsg-114"></a>
**LSG-114 — Nobody touches somebody else's vote.** On Post Vote. Refused with `It is not allowed to
modify someone else's vote.` A non-administrator may in addition neither set nor change the owner
and the recipient of a vote, whether through the written values or through a default inherited from
the calling context; those keys are stripped.

<a id="lsg-115"></a>
**LSG-115 — Voting requires reputation points, with two escapes.** Refused with `%d karma required
to upvote.` or `%d karma required to downvote.` Two escapes exist: an upvote is always allowed when
it cancels the reader's own downvote, and a downvote is always allowed when it cancels the reader's
own upvote.

<a id="lsg-115a"></a>
**LSG-115a — Only a moderator reaches a moderation queue.** The validation, flagged, offensive and
closed queues answer not-found to a reader whose balance is below the moderation threshold.

<a id="lsg-115b"></a>
**LSG-115b — One pending question at a time.** A user who already has a question waiting for
validation on a forum is redirected back to the ask page instead of being allowed to ask another.

<a id="lsg-115c"></a>
**LSG-115c — Asking requires a valid electronic mail address.** A user whose account carries no
valid address is redirected to their own profile page carrying the forum identifier, so they can
complete the address first.

<a id="lsg-115d"></a>
**LSG-115d — A post of an author with no reputation is hidden.** The view rule makes a post
invisible when the author's balance is zero or lower, unless the reader is the author or the reader
may close posts.

<a id="lsg-115e"></a>
**LSG-115e — A tag page rejects an invalid pager letter or filter.** An entry that is not a single
alphabetic character is refused with `Bad "tag_char" value "<value>"`; an unknown filter is refused
with `Bad "filters" value "<value>".`

---

## 10. Recognition

<a id="lsg-116"></a>
**LSG-116 — A badge that nobody may send.** Granting a badge whose granting rule is `nobody` is
refused with `This badge can not be sent by users.`

<a id="lsg-117"></a>
**LSG-117 — A badge restricted to named senders.** Granting a badge whose granting rule is `users`
by somebody outside the authorised list is refused with `You are not in the user allowed list.`

<a id="lsg-118"></a>
**LSG-118 — A badge that requires other badges.** Granting a badge whose granting rule is `having`
by somebody who lacks one of the required badges is refused with `You do not have the required
badges.`

<a id="lsg-119"></a>
**LSG-119 — A monthly sending cap.** When the badge is capped and the sender has already reached the
cap this month, the grant is refused with `You have already sent this badge too many time this
month.`

<a id="lsg-120"></a>
**LSG-120 — Nobody grants a badge to themselves.** Refused with `You can not grant a badge to
yourself.` from the general dialogue and with `You can not send a badge to yourself.` from the
employee dialogue.

<a id="lsg-121"></a>
**LSG-121 — A granted badge matches its employee.** On User Badge, when an employee is set that does
not correspond to the user, the save is refused with `The selected employee does not correspond to
the selected user.`

<a id="lsg-122"></a>
**LSG-122 — A challenge with running goals cannot be reset.** On Challenge, when the state is
written back to `draft` while at least one goal is in progress, the write is refused with `You can
not reset a challenge with unfinished goals.`

<a id="lsg-123"></a>
**LSG-123 — A personal challenge reports per user.** Asking for the progress of a challenge whose
display mode is `personal` without naming a user is refused with `Retrieving progress for personal
challenge without user information`.

<a id="lsg-124"></a>
**LSG-124 — A started goal keeps its configuration.** On Goal, when the definition or the user is
written on a goal that is no longer in state `draft`, the write is refused with `Can not modify the
configuration of a started goal`.

<a id="lsg-125"></a>
**LSG-125 — A goal definition carries a usable filter.** On Goal Definition, at every save. When the
stored filter cannot be evaluated against the chosen entity, the save is refused with `The domain for
the definition %(definition)s seems incorrect, please check it.  %(error_message)s`.

<a id="lsg-126"></a>
**LSG-126 — A goal definition carries a usable entity and field.** On Goal Definition, at every
save. When the chosen summed field is not stored the save is refused with `The model configuration
for the definition %(name)s seems incorrect, please check it.  %(field_name)s not stored`; when the
entity or the field cannot be found the save is refused with `The model configuration for the
definition %(name)s seems incorrect, please check it.  %(error)s not found`.

<a id="lsg-127"></a>
**LSG-127 — A rank requires a positive minimum.** On Karma Rank, at every save. The constraint
`_karma_min_check` refuses a minimum of zero or less with `The required karma has to be above 0.`

<a id="lsg-128"></a>
**LSG-128 — A movement is never edited.** Reputation movements are created and, once past,
consolidated; no operation edits an existing movement. Only the group `base.group_system` may create,
change or delete one directly; every other movement is written by the domain on the user's behalf.

---

## 11. Cross-domain rules

<a id="lsg-129"></a>
**LSG-129 — Two contacts enrolled in the same course cannot be merged.** Refused with `You cannot
merge these contacts because multiple contacts are enrolled in the same courses: <course names>`,
because the merged contact would break [`LSG-064`](#lsg-064).

<a id="lsg-130"></a>
**LSG-130 — One unit per course line.** Raising the quantity of an order line whose product carries
the service tracking `course` above one is refused: the quantity is forced back to one and the
message `You can only add a course once in your cart.` is returned. A course line may not be
reordered from the order history.

<a id="lsg-131"></a>
**LSG-131 — A course product may always be added to the cart.** A product used by at least one
published course may be added to the cart whatever the ordinary rules of the storefront say, and it
is allowed to carry a zero price although its service tracking is set.

<a id="lsg-132"></a>
**LSG-132 — Publication is kept in step between a paid course and its product.** Publishing a paid
course publishes its product; unpublishing a paid course unpublishes its product unless another
published course still uses the same product.

<a id="lsg-133"></a>
**LSG-133 — Deactivating a language clears it from every questionnaire.** Writing the active marker
of a language to false removes it from the language list of every questionnaire that referenced it.

<a id="lsg-134"></a>
**LSG-134 — A failing last certification attempt removes the attendee.** When a participation
attached to a progress record reaches the completed state with a failing score and no attempt is
left for that contact, address and invite token, the certification-failure message is sent and the
attendee is removed from the course: the enrolment is archived, the contact is unsubscribed from the
discussion thread, and the link between every participation of that attendee on that course and its
progress record is cleared, so a new attempt pool starts on re-enrolment.

<a id="lsg-135"></a>
**LSG-135 — A certification badge changes category with its course link.** When a course content
item points at a questionnaire that awards a badge, the challenge of that badge is re-categorised
under the learning platform, so the badge appears in the certification list of the public profile
pages. When the content item is deleted, or the questionnaire link is changed, the challenge of the
previous questionnaire is put back under the certification category.

<a id="lsg-136"></a>
**LSG-136 — Reputation points survive leaving a course.** Points earned while progressing are not
taken back when the attendee leaves; the progress records are kept, so re-joining restores the
progress and the same points cannot be earned twice.

---

## 12. Locking and concurrency

| Situation | Rule |
|---|---|
| Incrementing the view counter of a forum question | The counter is incremented without taking a lock. Two simultaneous opens may therefore count as one. This is accepted deliberately: the counter is a popularity indicator, not an accounting figure. |
| Incrementing the anonymous view counter of a content item | The same, without a lock; the identifiers already counted in the browser session stop a refresh from inflating the figure. |
| Incrementing the quiz attempt counter | The same, without a lock. |
| Incrementing an embed counter | The counter row for the pair of content item and address is created on first use and incremented afterwards; two simultaneous first loads may create two rows for the same address, and both are then counted. |
| Opening a live session | The write runs with elevated rights and is flushed before the broadcast is emitted, so every attendee screen sees the new state when it reacts to the broadcast. |
| Generating session codes | Codes already drawn inside the same batch are excluded from the later draws, and the uniqueness constraint is the final arbiter. |
| Running the daily challenge job | The job commits between challenges when it is asked to, so a failure late in the run does not undo the earlier challenges. |
| Consolidating the movement ledger | The consolidation writes the replacement movements, deletes the originals and flushes, all with the balance recomputation suppressed. |
