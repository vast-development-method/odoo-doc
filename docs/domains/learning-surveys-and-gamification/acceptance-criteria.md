# Learning, Questionnaires and Recognition — Acceptance criteria

Numbered Given, When and Then scenarios with concrete starting records, concrete inputs and exact
resulting records, amounts, counts and states. The scenarios cover the ordinary path, every
validation failure, every state transition, the rounding edges and the multi-company and
multi-currency situations where they apply.

Unless a scenario says otherwise, the fixtures are:

- **Officer Ada**, an internal user in the group `survey.group_survey_user`.
- **Manager Mo**, an internal user in the group `survey.group_survey_manager`.
- **Teacher Tom**, an internal user in the groups `website_slides.group_website_slides_officer` and
  `website_slides.group_website_slides_manager`, responsible for the course *Safety Basics*.
- **Learner Lea**, a portal user whose contact is *Lea Ortiz*, holding 120 reputation points.
- **Member Max**, a portal user holding 1 200 reputation points.
- **Visitor Vic**, an anonymous visitor.
- Today is 12 September and the reader's time zone is coordinated universal time.

---

## Part 1 — Authoring a questionnaire

### LSG-AC-001 — A new questionnaire is created with its tokens and its derived settings

**Given** no questionnaire exists.
**When** Ada creates a questionnaire titled `Safety Quiz` and saves it without touching any other
field.
**Then** one Survey exists with the purpose `custom`, the pagination `page_per_question`, the
selection `all`, the progress indicator `percent`, the access mode `public`, the required percentage
80.00, the attempt cap 1, the countdown 10 minutes and the certificate layout `modern_purple`; its
access token is a freshly generated universally unique value; its session code is a four-digit
string that no other questionnaire holds; its scoring mode is `no_scoring`; and its certification
marker is false.

### LSG-AC-002 — Choosing the assessment purpose applies its package of defaults

**Given** the questionnaire of LSG-AC-001.
**When** Ada sets the purpose to `assessment` in the form.
**Then** the access mode becomes `token` and the scoring mode becomes `scoring_with_answers`;
nothing else changes.

### LSG-AC-003 — Choosing the live-session purpose applies eight defaults at once

**Given** a questionnaire whose access mode is `token`, whose attempt cap is on, whose countdown is
on, whose progress indicator is `number`, whose pagination is `one_page`, whose selection is
`random`, whose scoring mode is `no_scoring` and whose roaming is on.
**When** Ada sets the purpose to `live_session`.
**Then** the access mode is `public`, the attempt cap is off, the countdown is off, the progress
indicator is `percent`, the pagination is `page_per_question`, the selection is `all`, the scoring
mode is `scoring_with_answers` and roaming is off.

### LSG-AC-004 — A section may not carry a question shape

**Given** the questionnaire of LSG-AC-001.
**When** Ada creates an entry titled `Introduction`, marks it as a page and leaves the shape
`simple_choice` on it.
**Then** the save is refused with `Question type should be empty for these pages: Introduction` and
no entry is created.

### LSG-AC-005 — A scale must grow inside zero and ten

**Given** the questionnaire of LSG-AC-001.
**When** Ada creates a question of the shape `scale` with a minimum of 5 and a maximum of 5.
**Then** the save is refused with `The scale must be a growing non-empty range between 0 and 10
(inclusive)`.
**And when** Ada sets the maximum to 12.
**Then** the save is refused with the same message.
**And when** Ada sets the minimum to 1 and the maximum to 10.
**Then** the question is saved.

### LSG-AC-006 — Text-length bounds must be ordered and non-negative

**Given** a question of the shape `char_box` with validation required.
**When** Ada sets the minimum length to −1.
**Then** the save is refused with `A length must be positive!`
**And when** Ada sets the minimum length to 10 and the maximum length to 5.
**Then** the save is refused with `Max length cannot be smaller than min length!`

### LSG-AC-007 — A scored date question needs its correct date

**Given** a scored questionnaire.
**When** Ada creates a question of the shape `date`, marks it scored and leaves the correct date
empty.
**Then** the save is refused with `All "Is a scored question = True" and "Question Type: Date"
questions need an answer`.

### LSG-AC-008 — A per-question score may not be negative

**Given** a question of the shape `numerical_box`.
**When** Ada sets its score to −2.00.
**Then** the save is refused with `An answer score for a non-multiple choice question cannot be
negative!`
**And when** Ada instead creates an answer option on a choice question with the score −2.00.
**Then** the option is saved, because option scores may be negative.

### LSG-AC-009 — An answer option belongs to exactly one question

**Given** a choice question `A` and a matrix question `B`.
**When** Ada creates an answer option linked to both `A` and `B`.
**Then** the save is refused with `A label must be attached to only one question.`
**And when** Ada creates an answer option linked to neither.
**Then** the save is refused with the same message.

### LSG-AC-010 — An answer option needs a value or a picture

**Given** a choice question.
**When** Ada creates an option with an empty text and no picture.
**Then** the save is refused with `Suggested answer value must not be empty (a text and/or an image
must be provided).`

### LSG-AC-011 — Roaming cannot be combined with page-by-page answers

**Given** a questionnaire whose scoring mode is `scoring_with_answers_after_page`.
**When** Ada turns roaming on and saves.
**Then** the save is refused with `Combining roaming and "Scoring with answers after each page" is
not possible; please update the following surveys: - Safety Quiz`.

### LSG-AC-012 — Restricting a questionnaire keeps its responsible inside the restriction

**Given** a questionnaire whose responsible is Ada, who holds only the officer group.
**When** Mo sets the restricted list to Officer Bea alone and saves through an interface that does
not apply the form assistance.
**Then** the save is refused with `The access of the following surveys is restricted. Make sure their
responsible still has access to it:` followed by a line naming the questionnaire and Ada.
**And when** the same change is made on the form.
**Then** Ada is appended to the restricted list automatically and the save succeeds with two names
in the list.

### LSG-AC-013 — The maximum obtainable score of a mixed questionnaire

**Given** a questionnaire holding: a numerical question worth 5.00; a several-answer choice question
with options worth 2.50, 2.50 and −1.00; a one-answer choice question with options worth 4.00, 0.00
and −2.00; and a multi-line text question.
**When** the maximum obtainable score is read.
**Then** it is 14.00 points.

### LSG-AC-014 — Attempt limiting is impossible on an anonymous questionnaire

**Given** a questionnaire whose access mode is `public` and which does not require signing in.
**When** Ada turns the attempt cap on.
**Then** the derivation forces it back to false and the attempt cap stays off.
**And when** Ada turns the sign-in requirement on and then the attempt cap on.
**Then** the attempt cap stays on.

### LSG-AC-015 — Attempt limiting is impossible with conditional questions

**Given** a questionnaire that requires signing in and whose attempt cap is on.
**When** Ada makes a question conditional by giving it a triggering answer option.
**Then** the attempt cap is forced back to false.

### LSG-AC-016 — A certification requires a scoring mode

**Given** a questionnaire whose scoring mode is `no_scoring`.
**When** Ada turns the certification marker on and saves.
**Then** the save is refused with `You can only create certifications for surveys that have a
scoring mechanism.`
**And when** Ada creates a fresh questionnaire with the certification marker true and no scoring
mode.
**Then** the derivation sets the scoring mode to `scoring_without_answers` and the record saves.

### LSG-AC-017 — A badge may be awarded only by a certification that requires signing in

**Given** a certification that does not require signing in.
**When** Ada turns the badge marker on.
**Then** the derivation forces it back to false.
**And when** Ada turns the sign-in requirement on and then the badge marker on and saves.
**Then** the badge marker is true, one goal definition named after the questionnaire exists with the
description `Safety Quiz certification passed`, one challenge named `Safety Quiz challenge
certification` exists in the state `inprogress` with the category `certification`, the periodicity
`once`, the personal display, no report and a real-time reward, and one challenge line joins them
with a target of 1.

### LSG-AC-018 — Turning the badge off dismantles the chain

**Given** the records of LSG-AC-017.
**When** Ada turns the badge marker off.
**Then** the badge is archived, the challenge is deleted, the challenge line is deleted with it and
the goal definition is deleted.

### LSG-AC-019 — Two questionnaires may not share one badge

**Given** the certification of LSG-AC-017 and a second certification.
**When** Ada points the second certification at the same badge.
**Then** the save is refused with `The badge for each survey should be unique!`

### LSG-AC-020 — Duplicating a questionnaire re-points its conditional triggers

**Given** a questionnaire holding a choice question `Q1` with the options `Yes` and `No`, and a
conditional question `Q2` triggered by `Yes`.
**When** Ada duplicates the questionnaire.
**Then** the copy holds its own `Q1` with its own two options and its own `Q2`, whose trigger is the
copy's `Yes` option and not the original's; the copy carries a new access token, a new session code,
no badge, no session state and no participation.

---

## Part 2 — Sharing and answering

### LSG-AC-021 — The five pre-invitation checks, in order

**Given** a questionnaire with no question at all.
**When** Ada presses the share action.
**Then** the check is refused with `You cannot send an invitation for a survey that has no
questions.`
**And when** a question is added, the questionnaire is scored and every score is zero.
**Then** the check is refused with `A scored survey needs at least one question that gives points.
Please check answers and their scores.`
**And when** a scoring option worth 3.00 is added and the pagination is set to `page_per_section`
while no section exists.
**Then** the check is refused with `You cannot send an invitation for a "One page per section"
survey if the survey has no sections.`
**And when** a section holding no question is added.
**Then** the check is refused with `You cannot send an invitation for a "One page per section"
survey if the survey only contains empty sections.`
**And when** the question is moved under the section and the questionnaire is archived.
**Then** the check is refused with `You cannot send invitations for closed surveys.`

### LSG-AC-022 — Inviting two contacts and one free address

**Given** a questionnaire whose access mode is `token`, holding one question, not archived; the
contacts *Lea Ortiz* and *Sam Patel* exist; the address `nina@example.com` matches no contact.
**When** Ada opens the composer, picks Lea and Sam, types `nina@example.com`, sets the deadline to
30 September at 23:59 and presses send in the mode `new`.
**Then** three Survey Participations exist, all in the state `new`, each with its own access token
and its own invite token, two naming Lea and Sam and one carrying only the address
`nina@example.com`, all three carrying the deadline 30 September at 23:59; three outgoing messages
exist, each carrying the recipient's own participation address.

### LSG-AC-023 — Resending reuses the most recent participation

**Given** the state after LSG-AC-022, and a second round in which Lea already holds two
participations, the later one created on 5 September.
**When** Ada sends again in the mode `resend` to Lea alone.
**Then** no new participation is created for Lea; the message is built on her participation of 5
September; the composer had shown her in the existing-recipients warning.

### LSG-AC-024 — A free address on a sign-in questionnaire is refused

**Given** a questionnaire that requires signing in on a platform where self-registration is not
allowed.
**When** Ada types `nina@example.com` into the composer.
**Then** the form is refused with `This survey does not allow external people to participate. You
should create user accounts or update survey access mode accordingly.`

### LSG-AC-025 — An invalid address is named back

**Given** a questionnaire whose access mode is `public`.
**When** Ada types `nina@example.com, not-an-address` into the composer.
**Then** the form is refused with `Some emails you just entered are incorrect: not-an-address`.

### LSG-AC-026 — A contact with no account is named back

**Given** a questionnaire that requires signing in on a platform where self-registration is not
allowed, and the contact *Sam Patel* who has no user account.
**When** Ada picks Sam in the composer.
**Then** the form is refused with `The following recipients have no user account: Sam Patel. You
should create user accounts for them or allow external signup in configuration.`

### LSG-AC-027 — Sending with no recipient at all

**Given** the composer open on a valid questionnaire.
**When** Ada presses send with no contact and no address.
**Then** the operation is refused with `Please enter at least one valid recipient.`

### LSG-AC-028 — A public landing creates one participation per visit

**Given** a questionnaire whose access mode is `public`, not archived, holding two questions.
**When** Vic opens the start address twice in two different browsers.
**Then** two Survey Participations exist, both in the state `new`, both with an empty contact and an
empty address, each with its own access token and no invite token.

### LSG-AC-029 — A closed questionnaire refuses a new attempt

**Given** the questionnaire of LSG-AC-028, archived.
**When** Vic opens the start address.
**Then** no participation is created and the refusal recorded is `Creating token for
closed/archived surveys is not allowed.`; the visitor sees the closed-or-expired page.

### LSG-AC-030 — Answering a mandatory question that is left empty

**Given** a participation in progress on a question of the shape `char_box`, mandatory, whose own
error message is empty, with roaming off.
**When** the participant submits an empty answer.
**Then** nothing is stored and the message returned for that question is `This question requires an
answer.`
**And when** the question's own error message is set to `Please tell us your name.`
**Then** the message returned is `Please tell us your name.`

### LSG-AC-031 — Validating an electronic mail address

**Given** a mandatory `char_box` question with the address check on.
**When** the participant submits `lea.ortiz`.
**Then** the message returned is `This answer must be an email address` and nothing is stored.
**And when** the participant submits `lea.ortiz@example.com`.
**Then** one answer row is stored with the answer type `char_box` and the value
`lea.ortiz@example.com`.

### LSG-AC-032 — Validating a text length

**Given** a `char_box` question with validation required, a minimum length of 5, a maximum length of
10 and the validation message `Between five and ten characters, please.`
**When** the participant submits `abc`.
**Then** the message returned is `Between five and ten characters, please.`
**And when** the participant submits `abcdefghijk`, eleven characters.
**Then** the same message is returned.
**And when** the participant submits `abcde`, exactly five characters.
**Then** the answer is stored, because the bounds are inclusive.

### LSG-AC-033 — Validating a number

**Given** a `numerical_box` question with validation required, a minimum of 1.00, a maximum of
10.00 and an empty validation message.
**When** the participant submits `abc`.
**Then** the message returned is `This is not a number`.
**And when** the participant submits `10.5`.
**Then** the message returned is `The answer you entered is not valid.`
**And when** the participant submits `10`.
**Then** the answer is stored with the value 10.00.

### LSG-AC-034 — A one-answer choice question refuses two choices

**Given** a `simple_choice` question with three options.
**When** the participant submits two of them.
**Then** the message returned is `For this question, you can only select one answer.`

### LSG-AC-035 — A comment may satisfy a mandatory choice question

**Given** a mandatory `multiple_choice` question with the comment box shown and comments counting as
answers.
**When** the participant selects no option and types the comment `None of these`.
**Then** the submit succeeds and two rows exist: none of type `suggestion` and one of type
`char_box` carrying the comment.
**And when** comments do not count as answers and the participant does the same.
**Then** the message returned is `This question requires an answer.`

### LSG-AC-036 — A mandatory matrix question needs every row

**Given** a mandatory `matrix` question with three rows and two columns, in the subtype `simple`.
**When** the participant answers two rows.
**Then** the message returned is `This question requires an answer.`
**And when** the participant answers all three.
**Then** three answer rows are stored, each carrying its row and its column.

### LSG-AC-037 — An answer row is a skip or an answer, never both

**Given** an integration writing an answer row directly.
**When** it writes a row with the skip marker true and the answer type `char_box`.
**Then** the save is refused with `A question can either be skipped or answered, not both.`
**And when** it writes a row with the answer type `numerical_box` and an empty numerical value that
does not round to zero at six decimal places.
**Then** the save is refused with `The answer must be in the right type`.
**And when** it writes a row with the answer type `numerical_box` and the value 0.0000001.
**Then** the row is accepted, because the value rounds to zero at six decimal places.

### LSG-AC-038 — Scoring a mixed participation

**Given** the questionnaire of LSG-AC-013 with the required percentage 80.00, and a participation
whose predefined set is the first three questions.
**When** the participant answers the numerical question correctly, ticks the 2.50 and −1.00 options
of the several-answer question, and picks the 4.00 option of the one-answer question, then reaches
the end.
**Then** the total possible score is 14.00, the total score is 10.50, the percentage is 75.00 and
the pass verdict is false.

### LSG-AC-039 — A negative percentage is clamped to zero

**Given** the same questionnaire and a participation whose predefined set is the same three
questions.
**When** the participant answers only the −1.00 and −2.00 options and reaches the end.
**Then** the total score is −3.00 and the stored percentage is 0.00, not −21.43.

### LSG-AC-040 — The pass boundary is inclusive

**Given** a questionnaire with a total possible score of 10.00 and a required percentage of 80.00.
**When** a participation collects exactly 8.00 points.
**Then** the percentage is 80.00 and the pass verdict is **true**.
**And when** it collects 7.99 points.
**Then** the percentage is 79.90 and the pass verdict is false.

### LSG-AC-041 — A conditional question is skipped and its denominator shrinks

**Given** a questionnaire holding `Q1` — a `simple_choice` question with the options `Yes` and `No`
— and `Q2`, a numerical question worth 6.00 triggered by `Yes`; `Q1` carries one correct option
worth 4.00.
**When** the participant answers `No` and reaches the end.
**Then** `Q2` never appears; at completion the predefined set is pruned to `Q1` alone; the total
possible score is 4.00 and the percentage is computed against 4.00, not against 10.00.

### LSG-AC-042 — A trigger that is un-ticked clears the stale answer

**Given** the questionnaire of LSG-AC-041 in a one-page layout, and a participation that answered
`Yes` and then answered `Q2` with the correct number.
**When** the participant goes back, un-ticks `Yes`, ticks `No` and submits.
**Then** the answer row of `Q2` is deleted, the score no longer counts it, and the completion prunes
`Q2` from the predefined set.

### LSG-AC-043 — Random selection draws a stable set

**Given** a questionnaire whose selection is `random`, holding one uncategorised question and one
section of five questions with a random count of 2.
**When** a participation is created and the participant answers the first screen, closes the browser
and resumes through the cookie.
**Then** the predefined set holds exactly three questions — the uncategorised one plus the two drawn
— and the resumed session asks exactly the same three.

### LSG-AC-044 — The attempt allowance

**Given** a questionnaire that requires signing in, with the attempt cap on and a limit of 3, and a
contact who holds two completed non-test participations in one pool, one completed participation in
another pool and one test entry.
**When** the remaining attempts of the first pool are read.
**Then** they are 1.
**And when** the contact completes a third attempt in that pool and asks for a fourth.
**Then** the creation is refused with `No attempts left.`

### LSG-AC-045 — The countdown and its grace margin

**Given** a questionnaire limited to 10 minutes and a participation started at 09:00:00.
**When** the participant submits at 09:10:07.
**Then** the submit is accepted, the valid answers are stored, the validation messages are ignored
and the participation is written to the completed state.
**And when** another participant submits at 09:10:11.
**Then** the submit is refused as unauthorised.

### LSG-AC-046 — A deadline in the past closes the participation

**Given** a participation carrying the deadline 30 August at 23:59.
**When** the participant opens it today.
**Then** access is refused and the closed-or-expired page is shown.

### LSG-AC-047 — A participation belongs to its owner

**Given** a participation naming the contact *Lea Ortiz*.
**When** Member Max, signed in with a different contact, opens its address.
**Then** the access-error page is shown.
**And when** Vic, anonymous, opens the same address without having received it by message.
**Then** the access-error page is shown.

### LSG-AC-048 — Roaming records skips and then walks them

**Given** a questionnaire with roaming on, one page per question, holding three mandatory questions.
**When** the participant answers the first, leaves the second empty, answers the third and submits
the last screen.
**Then** no message is produced for the second question; a skip row is stored for it; the
first-submitted marker becomes true; the navigation then offers the second question again as the
only remaining screen, and the submit button on it is the final one.

### LSG-AC-049 — Revealing the answers after a page

**Given** a questionnaire whose scoring mode is `scoring_with_answers_after_page`, one page per
question, holding a numerical question whose correct answer is 42 and worth 3.00.
**When** the participant submits 42.
**Then** the reply carries the next screen and a map naming that question with the correct value 42.

### LSG-AC-050 — Completing a certification sends the certificate

**Given** a certification requiring signing in, with the required percentage 80.00, a certificate
template configured, and a participation of Lea worth 90.00 percent.
**When** the participation reaches the end.
**Then** the participation is completed, the pass verdict is true, one outgoing message is created
from the certificate template with the certificate document attached, the completion notice is
posted on the participation and re-posted on the questionnaire, and the badge challenge is evaluated
at once.
**And when** the same participation is a test entry.
**Then** no message is sent.

---

## Part 3 — Live sessions

### LSG-AC-051 — Opening a session

**Given** a questionnaire whose purpose is `live_session`, holding three questions, whose pagination
is `one_page`.
**When** Ada presses the action that creates a live session at 10:00:00.
**Then** the pagination is forced to `page_per_question`, the session start instant is 10:00:00, the
current question is empty and the session state is `ready`.

### LSG-AC-052 — Only an officer manages a session

**Given** the questionnaire of LSG-AC-051.
**When** Learner Lea calls the operation that opens the session.
**Then** it is refused with `Only survey users can manage sessions.`

### LSG-AC-053 — Joining by code

**Given** the questionnaire of LSG-AC-051 whose session code is `4821`.
**When** Vic types `4821` on the code page while the session is `ready`.
**Then** the check succeeds and the visitor is redirected to the start address; a participation is
created marked as a session answer.
**And when** Vic types `9999`, which matches nothing.
**Then** the check answers `survey_wrong`.
**And when** Vic types the code of a questionnaire whose session is not open.
**Then** the check answers `survey_session_not_launched`; for Ada the answer additionally carries
the questionnaire identifier.

### LSG-AC-054 — Advancing to the first question

**Given** the session of LSG-AC-051 in the state `ready` with two attendees waiting.
**When** Ada asks for the next question at 10:05:00.
**Then** the session state becomes `in_progress`, the current question is the first question, the
current-question start instant is 10:05:01, and a broadcast carrying the event `next_question` and
the instant 10:05:00 is emitted on the questionnaire access token.

### LSG-AC-055 — The speed bonus on a correct answer

**Given** a session question worth 4.00 points, time limited to 20 seconds, on a questionnaire that
rewards quick answers, and a current-question start instant of 10:05:01.
**When** an attendee answers correctly at 10:05:09, that is 8 seconds later.
**Then** the stored score of that answer row is 3.3333.
**And when** another attendee answers correctly at 10:05:02, that is 1 second later.
**Then** the stored score is 4.00.
**And when** a third attendee answers correctly at 10:05:22, that is 21 seconds later.
**Then** the stored score is 2.00.

### LSG-AC-056 — The leaderboard is sorted on the score before the current question

**Given** three attendees holding 30, 24 and 24 points including the current question, on which they
earned 10, 0 and 6 points, and a current question whose positive option scores sum to 10.
**When** the leaderboard is read.
**Then** the entries carry the updated scores 30, 24 and 24, the scores before the current question
20, 24 and 18, the positions from the first reading 0, 1 and 2, the maximum question score 10 and
the question scores 10, 0 and 6; and the list is returned in the order 24, 20, 18.

### LSG-AC-057 — Closing a session completes every participation

**Given** the session of LSG-AC-054 with four participations, two in progress and two still new.
**When** Ada closes the session.
**Then** the session state is empty, all four participations are in the completed state, a broadcast
carrying the event `end_session` is emitted, no certificate message is sent and no badge is
evaluated; when the lead-generation package is installed, the leads of the session are created at
that moment.

### LSG-AC-058 — A question cannot be deleted while a session runs

**Given** the session of LSG-AC-054 in progress.
**When** Ada deletes one of the questions.
**Then** the deletion is refused with `You cannot delete questions from surveys "Safety Quiz" while
live sessions are in progress.`

---

## Part 4 — Building and publishing a course

### LSG-AC-059 — A new course enrols its creator and its responsible

**Given** no course exists.
**When** Tom creates a course named `Safety Basics` with the description `Everything you must know`
and saves it.
**Then** one Course exists with the type `training`, the visibility `public`, the enrolment policy
`public`, the featured strategy `latest`, the review reward 5, the completion reward 10, the review
threshold 10, the comment threshold 3 and the vote threshold 3; its short description is `Everything
you must know`; its access token is generated; and one Course Enrolment exists for Tom's contact
with the status `joined`.

### LSG-AC-060 — Attendee-only visibility forces invitation

**Given** the course of LSG-AC-059 with the enrolment policy `public`.
**When** Tom sets the visibility to `members` and saves.
**Then** the save is refused with `The Enroll Policy should be set to 'On Invitation' when
visibility is set to 'Course Attendees'`.
**And when** Tom also sets the enrolment policy to `invite`.
**Then** the save succeeds.

### LSG-AC-061 — A paid course needs a product

**Given** the course-selling package installed and the course of LSG-AC-059.
**When** Tom sets the enrolment policy to `payment` with no product.
**Then** the save is refused with `Product is required for on payment channels.`

### LSG-AC-062 — A content item holds one payload only

**Given** the course of LSG-AC-059.
**When** Tom creates a content item of the category `article` with an authored body and also types
an external address.
**Then** the save is refused with `A slide is either filled with a url or HTML content. Not both.`

### LSG-AC-063 — A certification content needs a questionnaire and cannot be previewed

**Given** the course-certification package installed.
**When** Tom creates a content item of the category `certification` with no questionnaire.
**Then** the save is refused with `A slide of type 'certification' requires a certification.`
**And when** Tom points it at a certification and also marks it as a free preview.
**Then** the save is refused with `A slide of type certification cannot be previewed.`

### LSG-AC-064 — The duration of an uploaded document is estimated

**Given** the course of LSG-AC-059.
**When** Tom uploads a portable document of 23 pages as a content item of the category `document`
with no duration.
**Then** the stored duration is 1.9167 hours and the subtype is `pdf`.

### LSG-AC-065 — A section sums the durations of its published items

**Given** a section holding three published content items lasting 0.2500, 0.7500 and 1.3333 hours,
and one uncategorised published item lasting 0.5000 hours in the same course.
**When** the durations are read.
**Then** the section duration is 2.3333 hours and the course duration is 2.83 hours.

### LSG-AC-066 — Ordering places items before sections at an equal sequence

**Given** a course holding, all with the sequence 1: a content item `A`, a section `S` and a content
item `B` created in that order.
**When** the ordered list is read.
**Then** the order is `A`, `B`, `S`, because content items precede sections at an equal sequence and
identifiers break the tie among the items; `A` and `B` belong to no section and are shown under
`Uncategorized`.

### LSG-AC-067 — Publishing posts the new-content message

**Given** the course of LSG-AC-059 with two followers of the new-content subtype, and an unpublished
content item of the category `video`.
**When** Tom publishes it at 11:00:00.
**Then** the publication instant is 11:00:00, one message rendered from the course's new-content
template is posted on the course and addressed to the two followers, and the completion of every
attendee is recomputed.

### LSG-AC-068 — Publishing is restricted on a training course

**Given** a training course whose responsible is Tom, and Officer Ina who holds the learning officer
group but is not the responsible.
**When** Ina tries to publish a content item on it.
**Then** the attempt is refused with `Publishing is restricted to the responsible of training
courses or members of the publisher group for documentation courses`.

### LSG-AC-069 — Archiving a course archives its contents in the right order

**Given** a course holding four published content items and one attendee at 50 percent.
**When** Tom archives the course.
**Then** the course is archived and unpublished first, the four content items are archived
afterwards, no completion message is sent and no reputation point moves.

---

## Part 5 — Enrolling and progressing

### LSG-AC-070 — Adding attendees enrols and messages them

**Given** the published course of LSG-AC-059 and the contacts *Lea Ortiz* and *Sam Patel*.
**When** Tom presses the add action, picks both and sends.
**Then** two Course Enrolments exist with the status `joined`, both contacts are subscribed to the
course discussion thread for the new-content subtype, and two outgoing messages rendered from the
template `Elearning: Add Attendees to Course` are created.

### LSG-AC-071 — Inviting attendees creates pending invitations

**Given** the same course and the two contacts, not yet enrolled.
**When** Tom presses the invite action and sends at 12:00:00.
**Then** two Course Enrolments exist with the status `invited` and a last-invitation instant of
12:00:00, and two outgoing messages rendered from the template `Elearning: Promotional Course
Invitation` are created.

### LSG-AC-072 — An officer without write access cannot add attendees

**Given** an invitation-only course whose responsible is Tom, and Officer Ina who is not the
responsible.
**When** Ina calls the add-members operation with the strict check.
**Then** it is refused with `You are not allowed to add members to this course. Please contact the
course responsible or an administrator.`

### LSG-AC-073 — Requesting access schedules one activity

**Given** an invitation-only published course and Lea, signed in and not enrolled.
**When** Lea presses request access.
**Then** one to-do activity is scheduled on the course, assigned to Tom, summarised `Access
Request`, with the note `Lea Ortiz is requesting access to this course.` and with Lea's contact as
the requesting contact.
**And when** Lea presses it again.
**Then** nothing is created and the answer is `Already Requested`.
**And when** Vic, anonymous, presses it.
**Then** the answer is `You have to sign in before`.
**And when** the course is unpublished and Lea presses it.
**Then** the answer is `Course not published yet`.
**And when** Lea is already enrolled and presses it.
**Then** the answer is `Already member`.

### LSG-AC-074 — Granting access enrols and closes the activity

**Given** the pending request of LSG-AC-073.
**When** Tom grants access.
**Then** one Course Enrolment exists for Lea with the status `joined` and the activity is closed
with the feedback `Access Granted`.
**And when** Tom refuses instead.
**Then** no enrolment exists and the activity is closed with the feedback `Access Refused`.

### LSG-AC-075 — The completion percentage and its rounding

**Given** a course holding 7 published, active, non-section content items and Lea enrolled with the
status `joined`.
**When** Lea completes 3 of them.
**Then** her completed count is 3, her completion is 43 and her status is `ongoing`.
**And when** she completes the remaining 4.
**Then** her completion is 100, her status is `completed`, 10 reputation points are granted with the
reason `Course Finished` and the completion message is sent.

### LSG-AC-076 — Publishing an eighth content does not un-complete a finished attendee

**Given** the state after LSG-AC-075.
**When** Tom publishes an eighth content item.
**Then** Lea's enrolment keeps the status `completed` and her completion stays 100, because the
recomputation skips enrolments already marked completed; no reputation point is taken back from her.

### LSG-AC-077 — A course with no content shows zero rather than an error

**Given** a course holding no published content item and an enrolled attendee.
**When** the completion is recomputed.
**Then** the completed count is 0 and the completion is 0; no division by zero occurs, because the
denominator is one.

### LSG-AC-078 — One enrolment per contact and course

**Given** Lea enrolled in `Safety Basics`.
**When** a second enrolment is written for the same pair.
**Then** the save is refused with `A partner membership to a channel must be unique!`

### LSG-AC-079 — Leaving keeps the progress

**Given** Lea at 43 percent on `Safety Basics` with three completed progress records.
**When** Lea leaves the course.
**Then** her enrolment is archived, she is unsubscribed from the course thread, and the three
progress records still exist.
**And when** she joins again.
**Then** the enrolment is un-archived, the completion is recomputed at once to 43 and the status
becomes `ongoing`; the ten completion points are not granted twice.

### LSG-AC-080 — An expired invitation is deleted

**Given** an enrolment with the status `invited`, a completion of 0 and a last-invitation instant of
1 May.
**When** the periodic cleanup runs on 12 September.
**Then** the enrolment is deleted.
**And when** another enrolment has the status `invited`, a completion of 0 and no last-invitation
instant.
**Then** it is deleted as well.
**And when** a third has the status `invited` and a last-invitation instant of 1 September.
**Then** it is kept.

### LSG-AC-081 — Automatic enrolment by access group

**Given** a course listing the group `Trainees` as an automatic-enrolment group, and the user Nils
who does not belong to it.
**When** Nils is added to the group `Trainees`.
**Then** one Course Enrolment exists for Nils's contact with the status `joined`.

### LSG-AC-082 — The invitation link of an invited attendee

**Given** an enrolment of Lea with the status `invited`, a last-invitation instant of 1 September
and a valid digest.
**When** Vic opens the invitation address while nobody is signed in.
**Then** the course page is shown with the invitation parameters and a banner inviting the visitor
to sign in or register.
**And when** the digest is altered by one character.
**Then** the visitor is sent to the course list with the error `hash_fail`.
**And when** the last-invitation instant is 1 May.
**Then** the visitor is sent to the course list with the error `expired`.
**And when** Member Max, signed in with a different contact, opens the address.
**Then** he is sent to the course list with the error `partner_fail`.

---

## Part 6 — Quizzes and certifications

### LSG-AC-083 — A quiz question needs a correct and an incorrect answer

**Given** a content item of the category `quiz`.
**When** Tom creates a question `What is the first rule?` whose two answers are both marked correct.
**Then** the save is refused with `All questions must have at least one correct answer and one
incorrect answer:` followed by the line `- Safety Basics quiz: What is the first rule?`.

### LSG-AC-084 — Passing a quiz on the third attempt

**Given** a content item carrying two quiz questions with the shipped rewards 10, 7, 5 and 2, and
Lea enrolled with no progress record.
**When** Lea submits a wrong set, then another wrong set, then a fully correct set.
**Then** her attempt counter reads 3, the content is marked completed, 5 reputation points are
granted with the reason `Quiz Completed`, and the course completion is recomputed.

### LSG-AC-085 — Un-completing a quiz takes the same points back

**Given** the state after LSG-AC-084.
**When** Lea marks the content uncompleted.
**Then** 5 reputation points are removed with the reason `Quiz Set Uncompleted` and the completion
marker is cleared; her balance returns to what it was before the quiz.

### LSG-AC-086 — The fourth reward is the ceiling

**Given** the same content item and an attendee whose attempt counter reads 6 after a passing
submission.
**When** the reward is computed.
**Then** the index is clamped to 4 and 2 reputation points are granted.

### LSG-AC-087 — An incomplete quiz submission is refused

**Given** a content item carrying three quiz questions.
**When** Lea submits answers covering only two of them.
**Then** the submission is refused with the status `slide_quiz_incomplete`, the attempt counter is
not incremented and nothing is granted.
**And when** Vic, anonymous, submits.
**Then** the refusal is `public_user`.
**And when** Lea submits again after having completed the content.
**Then** the refusal is `slide_quiz_done` and the answers saved in her browser session for that
content are dropped.

### LSG-AC-088 — Starting a course certification creates a fresh pool

**Given** a course holding a certification content item pointing at a certification whose attempt
cap is 2, and Lea enrolled with no attempt.
**When** Lea presses start.
**Then** one Survey Participation is created for her contact, without the attempt check, carrying
the content item, her progress record and a freshly generated invite token.
**And when** she leaves the certification and presses start again before finishing it.
**Then** the same participation is reused and no second one is created.

### LSG-AC-089 — Passing a certification completes the content and certifies the enrolment

**Given** the participation of LSG-AC-088 and a required percentage of 80.00.
**When** Lea scores 85.00 percent and reaches the end.
**Then** the certification marker of her progress record becomes true, the completion marker of that
progress record is forced true, her enrolment is marked certified, and the course completion is
recomputed.

### LSG-AC-090 — Failing the last attempt removes the attendee

**Given** the same certification with an attempt cap of 2 and Lea having already failed once in this
pool.
**When** Lea fails the second attempt.
**Then** the certification-failure message is sent to Lea; her enrolment is archived; she is
unsubscribed from the course thread; and the link between each of her participations on that course
and its progress record is cleared, so a later re-enrolment starts a new pool.

### LSG-AC-091 — A questionnaire used as a course certification cannot be deleted

**Given** the certification of LSG-AC-088 named `Safety Certification`, used by the course `Safety
Basics`.
**When** Mo deletes it.
**Then** the deletion is refused with `Uh-oh! You can’t delete surveys used as a Course
Certification! Otherwise, students might think diplomas just grow on trees. The courses that need
them are:` followed by the line `- Safety Certification (Courses - Safety Basics)`.

### LSG-AC-092 — A non-member previewing a certification gets a test attempt

**Given** a certification content item on a course Lea is not enrolled in but may see.
**When** Lea presses start.
**Then** a test participation is created carrying the content item and no progress record; nothing
is marked completed whatever her score; the participation is excluded from every statistic.

---

## Part 7 — Selling a course

### LSG-AC-093 — Confirming an order enrols the customer

**Given** the course-selling package installed, the course `Safety Basics` with the enrolment policy
`payment` and the product `Safety Basics access` whose service tracking is `course`, and a customer
*Nova Ltd* with the contact *Lea Ortiz*.
**When** an order for *Lea Ortiz* holding one line of that product is confirmed.
**Then** one Course Enrolment exists for Lea with the status `joined`, and nothing is written on the
order, on the invoice or in the ledger by this domain.

### LSG-AC-094 — A course line is limited to one unit

**Given** a cart holding one line of `Safety Basics access` with the quantity 1.
**When** the customer raises the quantity to 3.
**Then** the quantity is forced back to 1 and the message returned is `You can only add a course
once in your cart.`

### LSG-AC-095 — Publication is kept in step

**Given** two published courses `Safety Basics` and `Safety Refresher` sharing the product `Safety
Basics access`, which is published.
**When** Tom unpublishes `Safety Basics`.
**Then** the product stays published, because `Safety Refresher` still uses it.
**And when** Tom unpublishes `Safety Refresher` as well.
**Then** the product is unpublished.

### LSG-AC-096 — The revenue figure aggregates the product, not the course

**Given** the two courses of LSG-AC-095 sharing one product, with confirmed sales of 1 200.00 in the
product currency.
**When** the revenue is read on either course.
**Then** both show 1 200.00, because the figure aggregates every confirmed sale of the shared
product.

---

## Part 8 — The forum

### LSG-AC-097 — Asking above and below the publish threshold

**Given** a forum with the shipped defaults: ask threshold 3, publish threshold 100, new-question
grant 2.
**When** Member Max, holding 1 200 points, asks a question.
**Then** the post is created in the state `active`, Max gains 2 points with the reason `Ask a new
question`, his balance becomes 1 202, and the new-question notification is posted to the followers
of the forum and of the tags.
**And when** Learner Lea, holding 120 points, asks a question on a forum whose publish threshold is
500.
**Then** the post is created in the state `pending`, no point moves, and a validation notice is
posted as an internal note to the followers who reach the moderation threshold.
**And when** a member holding 2 points asks.
**Then** the creation is refused with `3 karma required to create a new question.`

### LSG-AC-098 — Validating a pending question grants the amount once

**Given** the pending question of LSG-AC-097 and a moderator holding 1 200 points on a forum whose
moderation threshold is 1 000.
**When** the moderator validates it.
**Then** the post state becomes `active`, the archiving marker becomes true, the moderator is
recorded, Lea gains 2 points with the reason `Ask a question`, her balance becomes 122, and the
state notification is posted.

### LSG-AC-099 — Answering a closed question is refused

**Given** a question in the state `close`.
**When** Max answers it.
**Then** the creation is refused with `Posting answer on a [Deleted] or [Closed] question is not
possible.`

### LSG-AC-100 — A vote sequence on an answer

**Given** a forum with the shipped defaults — upvote grant 10, downvote grant −2, upvote threshold
5, downvote threshold 50 — an answer written by Lea whose balance is 100, and Max who may vote.
**When** Max upvotes.
**Then** one Post Vote exists with the value `1`, 10 points are added to Lea with the reason `Answer
upvoted` and her balance becomes 110.
**And when** Max cancels his upvote.
**Then** the vote value becomes `0`, 10 points are removed with the reason `Answer no more upvoted`
and her balance returns to 100.
**And when** Max downvotes.
**Then** the vote value becomes `-1`, 2 points are removed with the reason `Answer downvoted` and
her balance becomes 98.
**And when** Max upvotes again.
**Then** the vote value becomes `1`, 12 points are added with the reason `Answer upvoted` and her
balance becomes 110.

### LSG-AC-101 — Voting on one's own post is refused

**Given** the answer of LSG-AC-100.
**When** Lea upvotes her own answer through the public endpoint.
**Then** the endpoint answers `own_post`, no vote record is created and no point moves.
**And when** the same is attempted through a direct write.
**Then** it is refused with `It is not allowed to vote for its own post.`

### LSG-AC-102 — A low-reputation member may always cancel their own vote

**Given** a forum whose downvote threshold is 50, and a member holding 10 points who already holds
an upvote on a post.
**When** the member presses downvote, which cancels their upvote.
**Then** the vote value becomes `0` and no refusal is raised, because cancelling one's own upvote is
always allowed.

### LSG-AC-103 — Accepting an answer moves two amounts

**Given** a forum with the shipped defaults — accepted grant 15, accepting grant 2, own-question
accept threshold 20 — a question asked by Max, an answer written by Lea whose balance is 100, and
Max's balance of 1 200.
**When** Max accepts Lea's answer.
**Then** Lea gains 15 points with the reason `User answer accepted` and reaches 115; Max gains 2
points with the reason `Validate an answer` and reaches 1 202; the answered marker of the question
becomes true; and the answer is ordered first.
**And when** Max un-accepts it.
**Then** Lea loses 15 with the reason `Accepted answer removed` and Max loses 2 with the reason
`Remove validated answer`.

### LSG-AC-104 — Accepting one's own answer moves nothing

**Given** a question asked by Lea and answered by Lea, and Lea holding enough points to accept.
**When** Lea accepts her own answer.
**Then** the acceptance marker is written and **no** point moves.

### LSG-AC-105 — Deleting an accepted answer costs both parties

**Given** the state after the acceptance in LSG-AC-103, and a moderator holding enough points to
delete.
**When** the moderator deletes the accepted answer.
**Then** 15 points are removed from Lea with the reason `The accepted answer is deleted` and 15
points are removed from the moderator with the reason `Delete the accepted answer`.

### LSG-AC-106 — Closing as spam for a first-time author

**Given** a forum with the shipped flagged grant of −100, and Lea holding 250 points whose only
question on that forum is being closed.
**When** a moderator closes it with the reason `Spam or advertising`.
**Then** the movement applied is −1 000 with the reason `Post is closed and marked as spam`, Lea's
balance becomes −750, and every post of hers becomes invisible to ordinary readers.
**And when** the moderator reopens the question.
**Then** the movement applied is +1 000 with the reason `Reopen a banned question` and her balance
returns to 250.

### LSG-AC-107 — Closing as spam for a frequent author

**Given** the same forum and Lea holding 250 points with four questions on it.
**When** a moderator closes one as spam.
**Then** the movement applied is −100 and her balance becomes 150.

### LSG-AC-108 — Marking a post offensive

**Given** the same forum, an author holding 300 points and a moderator holding 1 200.
**When** the moderator marks a post offensive with the reason `Violent language`.
**Then** the post state becomes `offensive`, the archiving marker becomes false, the moderator, the
closing instant and the reason are recorded, 100 points are removed from the author with the reason
`Downvote for posting offensive contents` and the author's balance becomes 200.

### LSG-AC-109 — Flagging

**Given** an active post and Member Max whose balance reaches the flag threshold of 500.
**When** Max flags it.
**Then** the state becomes `flagged`, Max is recorded as the flagging user and the answer is
`post_flagged_non_moderator`; a moderator doing the same receives `post_flagged_moderator`.
**And when** Max flags it again.
**Then** the answer is `post_already_flagged` and nothing changes.
**And when** Max flags a post whose state is `close`.
**Then** the answer is `post_non_flaggable`.

### LSG-AC-110 — The relevance of a post

**Given** a forum with the shipped relevance parameters 0.8 and 1.8, and a post with a vote total of
10 created 5 days ago.
**When** the relevance is computed.
**Then** it is 0.180211 carried to six decimal places.
**And when** an identical post was created today.
**Then** its relevance is 1.665466, so the fresher post ranks first.

### LSG-AC-111 — Related questions by tag overlap

**Given** the current question carrying the tags A, B and C, a candidate carrying B, C and D, and a
second candidate carrying A, B and C.
**When** the related questions are computed.
**Then** the second candidate scores 1.0, the first scores 0.5, and the second is proposed first.

### LSG-AC-112 — Creating a tag below the threshold

**Given** a forum whose tag-creation threshold is 30, and Lea holding 20 points.
**When** Lea types a new tag entry on the ask form.
**Then** the entry is dropped silently and the question is created without it.
**And when** a tag is created directly by the same member.
**Then** the creation is refused with `30 karma required to create a new Tag.`

### LSG-AC-113 — A tag name is unique inside a forum but not across forums

**Given** the forum `Help` holding the tag `safety`.
**When** a second tag `safety` is created on `Help`.
**Then** the save is refused with `Tag name already exists!`
**And when** a tag `safety` is created on a different forum.
**Then** the save succeeds.

### LSG-AC-114 — A post of an author with no reputation is hidden

**Given** the state after LSG-AC-106, where Lea's balance is −750.
**When** Vic opens the question list.
**Then** none of Lea's posts appears.
**And when** Lea herself opens it.
**Then** her own posts appear.
**And when** a moderator who may close posts opens it.
**Then** her posts appear.

### LSG-AC-115 — Converting an answer into a comment

**Given** an answer written by Lea on a question asked by Max, and Max holding 1 200 points on a
forum whose all-conversion threshold is 500.
**When** Max converts the answer into a comment.
**Then** a comment authored by Lea, dated with the answer's creation instant and carrying the
answer's body stripped of its attributes, styles and classes, is posted on the question; the answer
record is deleted; and the deletion threshold is not applied, because the deletion runs with
elevated rights.

### LSG-AC-116 — Converting a comment into an answer

**Given** a comment authored by Lea on a question, Lea having no answer on that question, and a
moderator holding 1 200 points.
**When** the moderator converts the comment.
**Then** a new answer authored by Lea, titled `Re: <question title>` and carrying the comment body,
is created; the comment is deleted.
**And when** Lea already has an answer on that question.
**Then** nothing happens.

### LSG-AC-117 — Posting a picture below the editor threshold

**Given** a forum whose editor threshold is 30, and a member holding 10 points.
**When** the member submits a body containing a picture element.
**Then** the write is refused with `30 karma required to post an image or link.`
**And when** a member holding 100 points submits a body containing a link, on a forum whose follow
threshold is 500.
**Then** the write succeeds and the link is rewritten with a no-follow marker, its address
unchanged.

### LSG-AC-118 — The moderation queues are closed to ordinary members

**Given** a forum whose moderation threshold is 1 000, and Lea holding 120 points.
**When** Lea opens the validation queue.
**Then** the page answers not-found.

### LSG-AC-119 — Forum statistics

**Given** a forum holding 12 active questions and 3 closed ones, 40 answers spread over them, view
counters summing to 5 300, 4 questions bookmarked at least once, 2 posts waiting for validation and
1 flagged post.
**When** the statistics are read.
**Then** the post count is 15, the view count is 5 300, the answer count is 40, the favourite count
is 4, the posts waiting for validation are 2 and the flagged posts are 1.

### LSG-AC-120 — A course forum follows the course visibility

**Given** the course-forum package installed, a published course whose visibility is `connected` and
a forum attached to it.
**When** the forum is attached.
**Then** the forum's privacy is cleared and its visibility mirrors `connected`; Vic, anonymous, sees
neither the forum nor its posts, while any signed-in visitor does.
**And when** the forum is detached.
**Then** the forum's privacy becomes `private` and it is restricted to the Learning Officer group.

---

## Part 9 — Recognition

### LSG-AC-121 — Generating goals when a challenge starts

**Given** a challenge in the state `draft` with the periodicity `monthly`, two lines with the
targets 10 and 4 both under the condition "the higher the better", and three participants.
**When** the challenge is started on 12 September.
**Then** six Goals exist, one per line and per participant, all in the state `inprogress`, with the
start date 1 September and the end date 30 September. The initial current value of each goal is the
target minus one, floored at zero: 9 for the target of 10 and 3 for the target of 4. That value is
deliberately short of the target so the first measurement always produces a change.
**And when** a third line with the target 1 is added under the same condition.
**Then** its goals start at 0, because the target minus one is 0 and the floor at zero applies.

### LSG-AC-122 — Measuring a counting goal in batch

**Given** a goal definition counting leads owned by the measured user, evaluated in batch, and a
goal for the user Nils with the target 12.
**When** the daily job measures it and Nils owns 7 leads inside the period.
**Then** the current value becomes 7, the completeness is 58.33 and the state stays `inprogress`.

### LSG-AC-123 — Reaching and un-reaching a goal

**Given** the goal of LSG-AC-122.
**When** a later measurement finds 12 leads.
**Then** the current value becomes 12, the completeness is 100 and the state becomes `reached`; the
goal is **not** closed.
**And when** two of those leads are deleted before the end date and the job runs again.
**Then** the current value becomes 10 and the state returns to `inprogress`.

### LSG-AC-124 — Failing after the end date

**Given** the goal of LSG-AC-122 with the end date 30 September and a current value of 7.
**When** the job runs on 1 October and finds 7 again.
**Then** nothing is written, because the value did not change.
**And when** the job runs on 1 October and finds 8.
**Then** the current value becomes 8, the state becomes `failed` and the goal is closed; later
measurements skip it.

### LSG-AC-125 — A ceiling goal is either zero or one hundred

**Given** a goal definition under the condition "the lower the better", a target of 3 and a current
value of 2.4.
**When** the completeness is read.
**Then** it is 100.
**And when** the current value is 3.0.
**Then** it is 0.

### LSG-AC-126 — Rewarding in real time

**Given** a challenge with two lines, a badge for every succeeding participant, the real-time reward
on, and the participant Ada who has just reached both goals.
**When** the reward check runs.
**Then** one User Badge is created for Ada, naming the challenge, and the badge notification is
sent.
**And when** the check runs again the next day and Ada still holds both goals reached.
**Then** no second grant is created.

### LSG-AC-127 — Ranking the best participants

**Given** a challenge with two lines under the condition "the higher the better", targets 10 and 4,
and three participants: Ada at 14 and 5, Ben at 10 and 4, Cleo at 12 and 3.
**When** the top three are computed and the challenge does not reward failures.
**Then** the order is Ada with 265.00, Ben with 200.00 and an empty third place; Cleo is excluded
because she did not reach every goal.
**And when** the challenge does reward failures.
**Then** the third place is Cleo with 195.00.

### LSG-AC-128 — Closing a challenge posts its message

**Given** the challenge of LSG-AC-127 with a badge for every succeeding participant named `Sales
Ace`.
**When** the challenge is closed.
**Then** a message is posted on the challenge, addressed to every participant, reading `The
challenge <name> is finished.` followed by `Reward (badge Sales Ace) for every succeeding user was
sent to Ada, Ben.`
**And when** nobody had reached every goal.
**Then** the second line is `Nobody has succeeded to reach every goal, no badge is rewarded for this
challenge.`

### LSG-AC-129 — A challenge with running goals cannot be reset

**Given** a challenge in the state `inprogress` with one goal still in progress.
**When** somebody writes the state back to `draft`.
**Then** the write is refused with `You can not reset a challenge with unfinished goals.`

### LSG-AC-130 — Granting a badge nobody may send

**Given** a badge whose granting rule is `nobody`.
**When** Member Max tries to grant it.
**Then** the grant is refused with `This badge can not be sent by users.`

### LSG-AC-131 — The monthly sending cap

**Given** a badge limited to 3 grants per person per month, and Max having granted it twice this
month.
**When** the remaining allowance is read.
**Then** it is 1.
**And when** Max grants it a third time and then a fourth.
**Then** the third succeeds and the fourth is refused with `You have already sent this badge too
many time this month.`

### LSG-AC-132 — Granting a badge to oneself

**Given** any badge whose granting rule is `everyone`.
**When** Max grants it to himself from the general dialogue.
**Then** the grant is refused with `You can not grant a badge to yourself.`
**And when** the same is attempted from the employee dialogue.
**Then** the refusal is `You can not send a badge to yourself.`

### LSG-AC-133 — A rank needs a positive minimum

**Given** the rank catalogue.
**When** an administrator creates a rank whose minimum is 0.
**Then** the save is refused with `The required karma has to be above 0.`

### LSG-AC-134 — Progress towards the next rank

**Given** the shipped ranks Newbie at 1, Student at 100, Bachelor at 500, Master at 2 000 and Doctor
at 10 000, and a user holding 350 points.
**When** the profile page is read.
**Then** the current rank is Student, the next rank is Bachelor and the progress is 62.5 percent.
**And when** a user holds 12 000 points.
**Then** the current rank is Doctor, there is no next rank and the progress is 100 percent.
**And when** a user holds 0 points.
**Then** they hold no rank, their next rank is Newbie and the progress is 0 percent.

### LSG-AC-135 — Crossing a rank sends the message

**Given** a user holding 99 points, whose rank is Newbie.
**When** they gain 1 point and reach 100.
**Then** their rank becomes Student, their next rank becomes Bachelor and the shipped rank message
is sent with the subject `New rank: Student`.

### LSG-AC-136 — The balance is read from the ledger

**Given** a user with three movements: from 0 to 10 on 1 September, from 10 to 8 on 5 September and
from 8 to 23 on 9 September.
**When** the balance is read.
**Then** it is 23, the new value of the most recent movement.
**And when** an administrator writes the balance to 30.
**Then** a fourth movement exists with an old value of 23, a new value of 30 and the reason `Add
Manually` followed by the display name and identifier of the acting user in parentheses.

### LSG-AC-137 — Consolidating a month

**Given** the movements of LSG-AC-136 in a month being consolidated.
**When** the monthly job runs.
**Then** one replacement movement exists with the old value 0, the new value 23, a gain of 23, the
tracking instant of the oldest movement, the consolidated marker true and the reason `Consolidation
from <start date> to <end date>`; the three originals are deleted; the user's balance is still 23.

### LSG-AC-138 — Validating an address grants three points once

**Given** a user whose balance is exactly 0 and whose address is `nils@example.com`.
**When** the user follows the validation link issued today.
**Then** the balance becomes 3.
**And when** the user follows it again.
**Then** nothing happens, because the balance is no longer zero.
**And when** a user whose balance is 5 follows a valid link.
**Then** nothing happens.

### LSG-AC-139 — The next report date

**Given** a challenge whose report frequency is monthly and whose last report date is 31 January of
a common year.
**When** the next report date is computed.
**Then** it is 28 February; in a leap year it is 29 February.
**And when** the frequency is `never`.
**Then** the next report date is empty and no scheduled report goes out.

### LSG-AC-140 — A personal report is withheld from a participant with an unreached goal

**Given** a challenge whose display mode is `personal` with two lines, and a participant who has
reached one goal and not the other.
**When** the report is generated.
**Then** that participant receives nothing, because the serialisation returns an empty list as soon
as one of their goals is not reached.

---

## Part 10 — Cross-domain and edge cases

### LSG-AC-141 — Merging two enrolled contacts is refused

**Given** the contacts *Lea Ortiz* and *L. Ortiz*, both enrolled in `Safety Basics`.
**When** an administrator merges them.
**Then** the merge is refused with `You cannot merge these contacts because multiple contacts are
enrolled in the same courses: Safety Basics`.

### LSG-AC-142 — Deactivating a language clears it from the questionnaires

**Given** two questionnaires whose language lists both hold the language *Portuguese*.
**When** an administrator deactivates *Portuguese*.
**Then** neither questionnaire lists it any more; the rest of each list is untouched.

### LSG-AC-143 — A completed course writes a résumé line

**Given** the skills package installed, the résumé type Training, and Lea whose contact is linked to
an employee record.
**When** Lea completes `Safety Basics` on 12 September.
**Then** one résumé line exists for that employee, named `Safety Basics`, dated 12 September, whose
description is the plain text of the course description, of the type Training, of the course kind
`elearning` and linked to the course; the duration is the course duration.
**And when** Lea leaves and completes the course again.
**Then** no second line is written, because one already exists for that employee, that course and
that type.

### LSG-AC-144 — A passed certification writes a résumé line with an expiry

**Given** the skills-certification package installed, the résumé type `Internal Certification`, and
a certification whose validity is 12 months.
**When** Lea passes it on 12 September.
**Then** one résumé line exists for her employee record, named after the questionnaire, dated 12
September, ending on 12 September of the following year, of the certification type and linked to the
questionnaire; its expiry status is `valid`.
**And when** the current date reaches 13 June of the following year, three months before the end.
**Then** the expiry status is `expiring`.
**And when** the current date passes the end date.
**Then** the expiry status is `expired`.
**And when** the validity is 0.
**Then** the line has no end date and its expiry status stays `valid`.

### LSG-AC-145 — A lead is created from a lead-generating answer

**Given** the lead-generation package installed, a questionnaire of the purpose `survey` whose
option `11-100 employees` is lead-generating and whose sales team is *Direct Sales*, and a
participation of the contact *Lea Ortiz*.
**When** the participation reaches the end having chosen that option.
**Then** one lead exists of the kind opportunity, titled `Lea Ortiz survey results`, naming Lea as
its contact, carrying the medium `Survey`, the source named after the questionnaire title, the team
*Direct Sales*, the originating questionnaire and a description listing every question with its
answer; the participation carries the link to that lead.
**And when** the participation is a live-session attempt.
**Then** the title is `Lea Ortiz live session results`.
**And when** the participant is anonymous and answered a question saved as the nickname with `Nina`.
**Then** the contact name of the lead is `Nina` and, when an address was captured, the lead carries
that address instead of a contact link.

### LSG-AC-146 — An embed counter groups anonymous loads

**Given** a content item never embedded before.
**When** the external embed is loaded from a page whose address has the host `partner.example.com`,
then again from the same host, then once from a request with no referring address.
**Then** two Content Embed Counters exist: one for `partner.example.com` with a view count of 2 and
one with an empty address and a view count of 1; the embed count of the content item is 3.

### LSG-AC-147 — The view counters of a content item

**Given** a content item with 12 progress records and 40 anonymous opens counted.
**When** the counters are read.
**Then** the signed-in views are 12, the anonymous views are 40 and the total views are 52.
**And when** an anonymous visitor refreshes the page in the same browser session.
**Then** none of the three counters changes.

### LSG-AC-148 — Likes and dislikes on a content item

**Given** a content item with 12 progress records, 7 of which carry a vote of plus one and 2 a vote
of minus one, in a course holding only that item.
**When** the counters are read.
**Then** the likes are 7, the dislikes are 2 and the course vote total is 5.

### LSG-AC-149 — A course review may be posted once

**Given** a course allowing comments, whose review threshold is 10, and Lea holding 120 points and
enrolled.
**When** Lea posts a message carrying a rating.
**Then** the message is accepted and 5 reputation points are granted with the reason `Course
Ranked`.
**And when** Lea posts a second message carrying a rating on the same course.
**Then** it is refused with `Only a single review can be posted per course.`
**And when** a member holding 3 points tries to post the first review.
**Then** it is refused with `Not enough karma to review`.

### LSG-AC-150 — Commenting a content item below the threshold

**Given** a course whose comment threshold is 3 and a member holding 1 point, enrolled.
**When** the member comments a content item.
**Then** it is refused with `Not enough karma to comment`.
**And when** the platform posts a notification on the same content item.
**Then** it is accepted, because only comments are gated.

### LSG-AC-151 — A resource is a file or a link

**Given** a content item.
**When** Tom creates a resource of the kind `url` with no link.
**Then** the save is refused with `A resource of type url must contain a link.`
**And when** Tom creates a resource of the kind `file` carrying a link.
**Then** the save is refused with `A resource of type file cannot contain a link.`
**And when** Tom writes a payload on an existing link resource named `Handbook`.
**Then** the save is refused with `Resource Handbook is a link and should not contain a data file`.

### LSG-AC-152 — A resource name follows the file until the author renames it

**Given** a resource of the kind `file` whose name is still the default.
**When** a file named `handbook.pdf` is uploaded.
**Then** the name becomes `handbook.pdf` and the download address ends with that name and carries
the download marker.
**And when** the author renames it to `Handbook`.
**Then** the name stays `Handbook` at every later save, and the download address appends the
extension, ending with `Handbook.pdf`.

### LSG-AC-153 — Two questionnaires drawn in one batch never share a session code

**Given** an existing questionnaire whose session code is `4821`.
**When** three questionnaires are created in one operation.
**Then** the three codes drawn are distinct from one another and none of them is `4821`.

### LSG-AC-154 — A content tag is globally unique

**Given** the content tag `safety`.
**When** a second content tag `safety` is created.
**Then** the save is refused with `A tag must be unique!`

### LSG-AC-155 — Course tag filters combine within and across families

**Given** the families `Level` holding `Basic` and `Advanced`, and `Tags` holding `Compliance`; the
course `Safety Basics` carrying `Basic` and `Compliance`; the course `Safety Advanced` carrying
`Advanced`.
**When** a visitor filters on `Basic` and `Advanced`.
**Then** both courses are shown, because the tags of one family combine with a logical or.
**And when** the visitor filters on `Basic` and `Compliance`.
**Then** only `Safety Basics` is shown, because families combine with a logical and.

### LSG-AC-156 — An internal course tag never reaches the public pages

**Given** a course tag whose colour is 0.
**When** Vic opens the course list.
**Then** the tag is not offered as a filter and does not appear on any card, because the record rule
restricts anonymous and portal readers to the tags that carry a colour.

### LSG-AC-157 — A course tag family that is not published is hidden

**Given** a family whose publication marker is false.
**When** Vic opens the course list.
**Then** the family does not appear in the filter panel, although its tags still classify the
courses in the back office.

### LSG-AC-158 — Only a manager may create or delete an answer row

**Given** a completed participation with five answer rows.
**When** Ada, an officer, deletes the whole participation.
**Then** the participation and its five rows are deleted.
**And when** Ada deletes one row alone.
**Then** the access is refused, because an officer holds read access only on answer rows.
**And when** Mo, a manager, deletes one row alone.
**Then** it is deleted.

### LSG-AC-159 — A restricted questionnaire is invisible to other officers

**Given** a questionnaire whose restricted list holds Ada alone, and Officer Bea.
**When** Bea opens the questionnaire list.
**Then** the questionnaire does not appear, and neither do its questions, its answer options, its
participations nor its answer rows.
**And when** Mo, a manager, opens the same list.
**Then** the questionnaire appears.

### LSG-AC-160 — A Learning Officer may read a certification but not change it

**Given** a certification whose restricted list is empty, and Teacher Tom who holds the learning
officer group but no questionnaire group.
**When** Tom opens it.
**Then** he reads the questionnaire, its questions, its answer options, its participations and its
answer rows.
**And when** Tom edits the title.
**Then** the write is refused, because the record rule grants read only.

### LSG-AC-161 — A goal is visible to its owner and, in a ranking challenge, to the other participants

**Given** a challenge whose display mode is `ranking` with the participants Ada, Ben and Cleo, and
their goals.
**When** Ada reads the goal list.
**Then** she sees her own goals and those of Ben and Cleo.
**And when** the display mode is `personal`.
**Then** she sees only her own.

### LSG-AC-162 — A goal is scoped to the reader's companies

**Given** two companies *Alpha* and *Beta*, a goal whose user belongs to *Beta*, and a reader who
belongs only to *Alpha*.
**When** the reader opens the goal list.
**Then** that goal does not appear, because the company rule restricts goals to the users of the
reader's companies.

### LSG-AC-163 — A started goal keeps its configuration

**Given** a goal in the state `inprogress`.
**When** somebody changes its definition.
**Then** the write is refused with `Can not modify the configuration of a started goal`.

### LSG-AC-164 — Answer statistics of a numerical question

**Given** a numerical question that collected the values 3, 7, 7, 10 and 4 in the current
population.
**When** the summary is read.
**Then** the maximum is 10, the minimum is 3, the average is 6.20 and the five most common values
are 7 with a count of 2 and then 3, 10 and 4 each with a count of 1.

### LSG-AC-165 — The global success rate rounds to one decimal place

**Given** a population of 7 participations of which 5 pass.
**When** the result page is read.
**Then** the global success rate is 71.4.

### LSG-AC-166 — Questionnaire statistics over completed and running participations

**Given** six non-test participations: four completed with the percentages 90.00, 75.00, 40.00 and
100.00, three of which pass, and two in progress at 0.00; the completed ones lasted 0.2500, 0.5000,
0.1250 and 0.6250 hours.
**When** the questionnaire card is read.
**Then** the registered count is 6, the attempt count is 4, the success count is 3, the average
score is 50.83, the success ratio is 50 and the average duration is 0.3750 hours.

### LSG-AC-167 — Reputation earned on a course, read back after the fact

**Given** an attendee who completed two quizzes on a course — one on the first attempt and one on
the third — and completed the course itself, with the shipped rewards.
**When** the officer asks how many points that attendee earned on the course.
**Then** the answer is 25: 10 for the first quiz, 5 for the second and 10 for the course.

### LSG-AC-168 — The remaining badge grants of a sender

**Given** a badge capped at 3 grants per person per month and a sender who granted it twice this
month.
**When** the remaining allowance is read.
**Then** it is 1.
**And when** the badge has no cap.
**Then** the allowance reads minus one, which means no limit and is not displayed as a number.
**And when** the sender may not grant the badge at all.
**Then** the allowance reads 0.
