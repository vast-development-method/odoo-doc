# Learning, Questionnaires and Recognition — Workflows

End-to-end operational procedures. Each workflow states its actor, its preconditions, its numbered
steps with the records each step creates or changes, the operations invoked, and the conditions
under which a step fails. Rule identifiers point at [`business-rules.md`](business-rules.md) and
formula identifiers at [`calculations.md`](calculations.md).

| Workflow | Subject |
|---|---|
| [1](#workflow-1) | Building a questionnaire |
| [2](#workflow-2) | Testing and sharing a questionnaire |
| [3](#workflow-3) | Sending invitations |
| [4](#workflow-4) | Answering a questionnaire |
| [5](#workflow-5) | Running a live session |
| [6](#workflow-6) | Creating and removing the certification badge chain |
| [7](#workflow-7) | Producing and delivering a certificate |
| [8](#workflow-8) | Building and publishing a course |
| [9](#workflow-9) | Enrolling attendees |
| [10](#workflow-10) | Progressing through a course |
| [11](#workflow-11) | Taking a quiz |
| [12](#workflow-12) | Taking a course certification |
| [13](#workflow-13) | Selling access to a course |
| [14](#workflow-14) | Leaving a course, and the expiry of invitations |
| [15](#workflow-15) | Asking and answering on a forum |
| [16](#workflow-16) | Moderating a forum |
| [17](#workflow-17) | Voting and accepting an answer |
| [18](#workflow-18) | Converting between answers and comments |
| [19](#workflow-19) | Running the daily recognition job |
| [20](#workflow-20) | Granting a badge by hand |
| [21](#workflow-21) | Consolidating the movement ledger |
| [22](#workflow-22) | The public profile and the address validation |

---

<a id="workflow-1"></a>

## Workflow 1 — Building a questionnaire

**Actor:** Questionnaire Officer. **Precondition:** none.

1. The officer creates a questionnaire, either empty or from one of the shipped samples. The record
   is created with an access token, a session code, a precomputed scoring mode and a precomputed
   certification marker. When a sample is chosen, its sections, questions and answer options are
   created in the same operation.
2. The officer picks a purpose. The form applies the package of defaults of that purpose; nothing
   afterwards enforces it, so any setting may be changed back. The four packages are tabulated in
   [`surveys.md`](surveys.md#purposes).
3. The officer adds sections and questions to the flat ordered list. Each entry is created with a
   sequence; a section is refused a question shape by [`LSG-011`](business-rules.md#lsg-011). A
   question created inside a questionnaire inherits the questionnaire speed settings and is marked
   customised when it differs from them.
4. For each question the officer picks a shape, fills the answer options where the shape needs them,
   sets the validation rules and, for a scored questionnaire, the scores. The constraints
   [`LSG-012`](business-rules.md#lsg-012) to [`LSG-018`](business-rules.md#lsg-018) run at each save.
5. The officer optionally makes a question conditional by picking triggering answer options. The
   picker offers only the options of choice questions positioned before this one; a question later
   moved before one of its triggers is flagged as misplaced.
6. The officer sets the access mode, the sign-in requirement, the attempt cap, the countdown and the
   roaming marker. The derivations force the attempt cap off when the questionnaire is fully
   anonymous or holds conditional questions.
7. The officer sets the scoring mode, the required percentage and, for a certification, the
   certificate template, the certificate message template and the badge. Turning the badge on
   triggers [workflow 6](#workflow-6).
8. The officer optionally restricts the questionnaire to named officers; the responsible is appended
   to the list automatically, and [`LSG-010`](business-rules.md#lsg-010) refuses a save that would
   lock the responsible out.
9. The officer saves. Every constraint of section 1 of [`business-rules.md`](business-rules.md) runs.

**Failure conditions.** Any constraint of [`LSG-001`](business-rules.md#lsg-001) to
[`LSG-020`](business-rules.md#lsg-020) refuses the save with its own message; the officer corrects
the record and saves again.

---

<a id="workflow-2"></a>

## Workflow 2 — Testing and sharing a questionnaire

**Actor:** Questionnaire Officer.

1. **Test.** The officer presses the test action. A participation is created with the test marker
   set; [`LSG-026`](business-rules.md#lsg-026) refuses the creation to somebody who may not read the
   questionnaire. A test attempt may open an archived questionnaire and is excluded from every
   statistic and from the attempt count. The officer is redirected to the start address of that
   participation.
2. **Share by link.** The officer copies the public start address, which carries the questionnaire
   access token. Anybody holding it may start when the access mode is `public`.
3. **Share by message.** The officer presses the share action, which opens the invitation composer;
   see [workflow 3](#workflow-3).
4. **Close.** Archiving the questionnaire closes it: new participations and invitations are refused
   by [`LSG-021`](business-rules.md#lsg-021) and [`LSG-031`](business-rules.md#lsg-031), and the
   certification badge is archived with it.
5. **Reopen.** Un-archiving reverses both effects.

---

<a id="workflow-3"></a>

## Workflow 3 — Sending invitations

**Actor:** Questionnaire Officer. **Precondition:** the questionnaire passes the five checks
[`LSG-027`](business-rules.md#lsg-027) to [`LSG-031`](business-rules.md#lsg-031), which run when the
composer is opened.

1. The officer opens the composer. The subject defaults to the template subject, or to `Participate
   to <questionnaire title>`; the attachments default to the attachments of the chosen template; the
   send marker defaults to true when the access mode is `token`.
2. The officer picks contacts and types free addresses. [`LSG-055`](business-rules.md#lsg-055),
   [`LSG-056`](business-rules.md#lsg-056) and [`LSG-057`](business-rules.md#lsg-057) run as the
   fields change. The composer shows which recipients already hold a participation.
3. The officer chooses the handling of existing recipients: `resend` reuses the most recent existing
   participation of each known recipient, `new` creates a fresh participation for everybody.
4. The officer optionally sets an answer deadline.
5. The officer presses send. The operation then runs:
   1. Every typed address is normalised. When a normalised address matches an existing contact, that
      contact replaces the raw address: every matching contact is taken when signing in is required,
      because each of them may have an account, and only the first one otherwise. Addresses that
      match nothing are kept as raw addresses.
   2. When neither a contact nor a raw address remains, the operation is refused by
      [`LSG-053`](business-rules.md#lsg-053).
   3. The existing participations of the questionnaire for those contacts and addresses are read.
   4. In `resend` mode the most recent existing participation of each known recipient is selected
      for re-sending, and no new participation is created for them. In `new` mode nothing is reused.
   5. For every recipient not being re-sent, a participation is created carrying the composer's
      deadline and **without** the attempt check ([`LSG-058`](business-rules.md#lsg-058)).
   6. For every selected participation an outgoing message is built. The sender is the rendered
      sender of the template when the template defines one, otherwise the author's formatted
      address; when neither exists the operation is refused by
      [`LSG-054`](business-rules.md#lsg-054). The subject and the body are rendered against the
      participation record; when the composer's subject and body are still exactly those of the
      template, the rendering is done in the recipient's own language. The composer's attachments
      are joined. A contact recipient is addressed as a contact; a raw address is addressed as an
      address. The body is wrapped in the chosen notification layout.
6. Each recipient receives a personal address carrying their own participation token.

---

<a id="workflow-4"></a>

## Workflow 4 — Answering a questionnaire

**Actor:** Participant.

1. **Landing.** The participant opens the start address. The validity checks of
   [`surveys.md`](surveys.md#validity-codes) run in order and either serve the page or divert the
   visitor. On a public questionnaire a participation is created at every landing; on a token
   questionnaire the participant must already hold one.
2. **Resuming.** A browser cookie named after the questionnaire access token holds the participation
   token for one day, so a participant who closes the tab resumes the same participation. A cookie
   pointing at somebody else's participation, or at one that no longer exists, is ignored and the
   checks are redone without it.
3. **Start screen.** The participant chooses a display language among the questionnaire languages
   and presses start. The participation moves to in progress and the start instant is written. The
   set of questions to ask was fixed at creation; a random draw is therefore stable across resumes.
4. **Each screen.** The screen holds one question, one section or every question, according to the
   layout. The participant answers and submits.
   1. The answers are validated question by question by
      [`LSG-032`](business-rules.md#lsg-032) to [`LSG-043`](business-rules.md#lsg-043).
   2. When at least one message is produced and no countdown has run out, nothing is stored and the
      messages are returned field by field.
   3. Otherwise the answers of the questions on that screen are written or replaced, and the last
      displayed screen is updated. Re-answering a choice question deletes the previous rows and
      creates new ones; re-answering a single-value question updates the existing row in place.
   4. The answers recorded for questions that are currently inactive are deleted, so a trigger that
      was un-ticked leaves no stale answer behind.
   5. When the scoring mode reveals the answers after each page, the submit returns, with the next
      screen, the correct answers of the scorable questions of the submitted screen that were not
      inactive.
5. **Navigation.** The next screen is the first later entry that is displayable; the rules are in
   [`surveys.md`](surveys.md#screen-navigation). When roaming is allowed the participant may move
   back, and a mandatory question may be left empty until a final pass over the skipped questions.
6. **Completion.** Reaching the end writes the end instant, prunes the question set of the
   conditional questions never triggered, and runs the completion side effects listed in
   [`state-machines.md`](state-machines.md#machine-1): the completion notice, the certificate
   message, the badge evaluation, the résumé line and the lead.
7. **Closing screen.** The participant sees the closing message and, according to the scoring mode,
   the score, the pass verdict and the correct answers. On a certification that was passed, a link
   to download the certificate is offered.
8. **Retry.** When attempts remain, the retry action creates a **new** participation carrying the
   same invite token and the same deadline; the completed one is never reopened.

---

<a id="workflow-5"></a>

## Workflow 5 — Running a live session

**Actor:** Live Session Host, a Questionnaire Officer. **Precondition:** the questionnaire's session
marker is true, which requires the purpose `live_session` or `custom` and no certification.

1. **Open.** The host presses the action that creates a live session
   ([`LSG-046`](business-rules.md#lsg-046)). The layout is forced to one page per question, the
   session start instant is written, the current question is cleared and the state becomes `ready`.
   The host is taken to the session manager.
2. **Join.** Attendees reach the session in one of three ways: the full session link, which is the
   site address followed by `/s/<session code>` and which resolves the code and redirects to the
   start address; the code page at `/s`, where the attendee types the code and the checks
   [`LSG-049`](business-rules.md#lsg-049) and [`LSG-050`](business-rules.md#lsg-050) run; or the
   plain questionnaire link, which works because a session questionnaire has a public access mode.
   Landing on the start address creates a participation marked as a session answer.
3. **Waiting room.** While the state is `ready` the attendees wait. The host screen shows the number
   of distinct creating users among the session's not-yet-completed participations.
4. **Advance.** Each time the host asks for the next question: the state moves to `in_progress` on
   the first request; the next question is computed against the artificial participation described
   in [`state-machines.md`](state-machines.md#machine-2); the questionnaire is written with the new
   current question and with a start instant equal to the current instant plus one second, which
   compensates the round trip so every attendee starts the countdown fairly; and a broadcast
   carrying the event `next_question` and the real current instant is emitted on the questionnaire
   access token, at which every attendee screen requests the next question. When no next question
   exists nothing is written and nothing is broadcast. The host may also move back, and the same
   logic runs in reverse.
5. **Answer.** An attendee's screen asks for the current question, marks the participation in
   progress when it is still new, and renders the question. When the question is time limited the
   screen receives the questionnaire's current-question start instant and the limit converted into
   minutes.
6. **Watch.** The host screen shows, for the current question: the number of attendees who answered
   it, the graph data, the correctness of each option in display order — used to colour the bars,
   with a trailing false appended when comments count as answers — and, for single-line text, date
   and date-and-time questions, up to one hundred raw values.
7. **Reveal.** The host may show the results, which returns the statistics of the current question
   restricted to the answer rows of this session, and may show the correct answers, which reveals
   which bars are correct.
8. **Leaderboard.** When the questionnaire is scored and at least one question saves a nickname, the
   host may show the leaderboard, built by [`LSG-CALC-018`](calculations.md#lsg-calc-018).
9. **Close.** The host closes the session ([`LSG-047`](business-rules.md#lsg-047)). The state is
   cleared, **every** participation of the questionnaire is forced to completed, and a broadcast
   carrying the event `end_session` is emitted. The consequence recorded in
   [`LSG-052`](business-rules.md#lsg-052) applies.

---

<a id="workflow-6"></a>

## Workflow 6 — Creating and removing the certification badge chain

**Actor:** Questionnaire Officer. **Precondition:** the questionnaire is a certification that
requires signing in, so the badge marker may be turned on.

**Creating the chain.** When the badge marker is true at creation, or is turned on later:

1. A goal definition is created with the questionnaire title as its name, the description
   `<title> certification passed`, a filter selecting the participations of this questionnaire whose
   pass verdict is true, the counting computation mode, a yes-or-no display, the condition "the
   higher the better", batch evaluation enabled, the participation's contact as the distinctive
   field and the acting user's contact as the batch expression.
2. A challenge is created named `<title> challenge certification`, rewarding the configured badge,
   in state in progress, with the periodicity `once`, real-time reward, no periodic report, personal
   display, and a population filter selecting the users whose balance is strictly greater than zero.
   Its category is `certification`, or `slides` when at least one course content item already points
   at this questionnaire.
3. One challenge line joins the goal definition to the challenge with a target of one.

**Removing the chain.** Turning the badge marker off archives the badge, then deletes every
challenge that rewards that badge together with the goal definitions those challenges use; the
challenge lines disappear with their challenge. Turning the marker back on un-archives the badge and
runs the creation again. Archiving the questionnaire archives the badge; un-archiving reverses it.

**Awarding.** The badge is granted at the moment a participation is completed: after the state is
written, every challenge rewarding one of the badges of the passed certifications is evaluated at
once, rather than waiting for the daily job.

---

<a id="workflow-7"></a>

## Workflow 7 — Producing and delivering a certificate

**Actor:** the system, on behalf of a participant.

1. A participation of a certification reaches the completed state.
2. The certificate document is rendered from the participation: one page, landscape, no margins. The
   visual template is chosen by the stored layout value, whose two halves give the shape and the
   colour. The page carries the heading `Certificate`; when the participation passed it also carries
   the sub-heading `of achievement`, the line `This certificate is presented to` followed by the
   contact name or, failing that, the electronic mail address, then `by <company name> for
   successfully completing` and the questionnaire title. When the participation did not pass, the
   block `Certification Failed` replaces all of that. At the bottom the page shows the creation date
   of the participation under the caption `Date`, and the company logotype. The `modern` shape also
   shows a seal when the participation passed. The name is sized by
   [`LSG-CALC-019`](calculations.md#lsg-calc-019).
3. When a certificate message template is configured, the participation passed and the attempt is
   not a test, that template is sent to the participant with the document attached.
4. A signed-in user may download the certificate of any certification they passed at any later time.
5. An officer may preview the certificate: a test participation is created, the document is rendered
   inline, and the test participation is deleted immediately afterwards.
6. When the skills-certification package is installed and the participant is an employee, a résumé
   line of the Certification type is written or updated, dated today, ending after the validity in
   months when one is set and never ending when it is zero.

---

<a id="workflow-8"></a>

## Workflow 8 — Building and publishing a course

**Actor:** Learning Officer.

1. The officer creates a course. The creator's contact is enrolled unless an enrolment list is
   supplied; the responsible's contact is enrolled; the members of the automatic-enrolment groups
   are enrolled; the short description is filled from the description when it is empty; an access
   token is generated.
2. The officer picks the type. A training course reserves publication to the responsible; a
   documentation course opens uploading to the upload groups and defaults the comment marker to
   false.
3. The officer sets the visibility and the enrolment policy. [`LSG-061`](business-rules.md#lsg-061)
   forces an attendee-only course to invitation, and
   [`LSG-062`](business-rules.md#lsg-062) requires a product on a paid course.
4. The officer adds content items and sections. Each item carries a category, a source and a
   payload; [`LSG-076`](business-rules.md#lsg-076) forbids two payloads. For an external address the
   metadata is retrieved and the empty fields are filled from it; for an uploaded portable document
   the duration is estimated by [`LSG-CALC-014`](calculations.md#lsg-calc-014).
5. The officer orders the list by dragging. A content item belongs to the last section before it.
   Moving a section moves its items with it: the items are removed from the ordered list of
   identifiers, re-inserted at the position of the target section, and every sequence is rewritten
   as its new index plus one. A newly added item is inserted just before the section that follows
   its own section, and the sequences are rewritten from one.
6. The officer publishes the course and its content items. Publishing a non-section content posts
   the new-content message on the course to the followers of the new-content subtype, using the
   course's new-content template, and writes the publication instant.
   [`LSG-079`](business-rules.md#lsg-079) refuses a publication by somebody who may not publish.
7. The officer sets the reward settings, the featured-content strategy, the tags and the
   prerequisites. A prerequisite may only be a course of the same visibility and publication state.
8. Archiving the course archives its content items and unpublishes it, in the order given in
   [`state-machines.md`](state-machines.md#machine-8).

---

<a id="workflow-9"></a>

## Workflow 9 — Enrolling attendees

Five paths lead to an enrolment.

**A. An officer adds attendees.** The officer presses the add action, which opens the invitation
composer in enrolment mode with the shipped enrolment template. The recipients are filtered by
[`LSG-066`](business-rules.md#lsg-066), added with the status `joined`, subscribed to the course
discussion thread, and messaged. When the send marker is false the operation does nothing at all.

**B. An officer invites attendees.** The same composer opens in invitation mode with the shipped
promotional template. Recipients already holding a pending invitation are separated out and
re-invited rather than re-added; the rest are added with the status `invited`. The last-invitation
instant of every re-invited and newly invited enrolment is set to the current instant. One message is
built per enrolment, rendered against that enrolment in the recipient's own language, wrapped in the
course-invitation layout with the course name as the record name and the responsible's signature.

**C. A visitor joins.** On an open-enrolment course a signed-in visitor presses join and is enrolled
at once with the status `joined`.

**D. A visitor requests access.** On an invitation-only course a signed-in visitor presses request
access. [`LSG-067`](business-rules.md#lsg-067) to [`LSG-071`](business-rules.md#lsg-071) run; a
to-do activity is scheduled for the responsible. Granting enrols the contact and closes the activity
with the feedback `Access Granted`; refusing closes it with `Access Refused`
([`LSG-072`](business-rules.md#lsg-072)).

**E. Automatic enrolment.** Every member of a course's automatic-enrolment groups is enrolled. The
enrolment is refreshed at five moments: when the course is created, when its group list is written,
when a user is created, when a user's group list is written — looking at the newly linked groups and
every group they imply — and when a group's user list is written.

**The personal invitation link.** Each enrolment exposes an address carrying the contact identifier
and a keyed digest over the pair of contact and course, compared in constant time. Opening it
behaves as follows:

1. When the course does not exist, the visitor is sent to the course list with the error
   `no_channel`.
2. The parameters are checked, producing one of: `no_partner` when the contact does not exist;
   `no_channel` when the course does not exist; `no_rights` when the course is not published;
   `expired` when no enrolment exists for that pair, or the enrolment is still pending and its
   last-invitation instant is empty or older than three months; `hash_fail` when the digest does not
   match.
3. On any error the visitor is sent to the course when they already have read access to it, and
   otherwise to the course list with that error.
4. When a user is signed in and is not the invited contact, the visitor is sent to the course list
   with the error `partner_fail`; otherwise to the course when they have read access and to the
   course list with `no_rights` when they do not.
5. When nobody is signed in and the enrolment is **not** pending, the visitor is sent to a
   self-registration page prepared for that contact when the contact has no account, and to the
   sign-in page pre-filled with that contact's login otherwise; both return to the course afterwards.
6. When nobody is signed in and the enrolment is pending, the visitor is sent to the course page
   carrying the invitation parameters, which grants a preview of the course with a banner inviting
   them to sign in or register. The sign-in button of that banner uses a separate identification
   endpoint, which refuses a signed-in visitor with the error `identify_fail`.

---

<a id="workflow-10"></a>

## Workflow 10 — Progressing through a course

**Actor:** Attendee.

1. The attendee opens the course page. The page shows the cover block, the title, the description,
   the rating summary, the attendee actions, the featured content chosen by the promotion strategy,
   the ordered list of contents grouped by section with the completion state of each, the reviews,
   and the publisher actions for a publisher.
2. The attendee opens a content item. A progress record is created or refreshed, so the signed-in
   view counter rises. An anonymous or non-enrolled visitor increments the anonymous counter instead,
   once per content item per browser session.
3. The attendee consumes the payload and marks the item completed when it carries no quiz and is not
   a certification ([`LSG-082`](business-rules.md#lsg-082)).
4. The completion recomputation runs for the attendee's enrolment:
   [`LSG-CALC-011`](calculations.md#lsg-calc-011) writes the completed count and the percentage, and
   the status follows.
5. Reaching one hundred percent grants the course-completion reward once with the reason `Course
   Finished`, sends the completion message and, when the skills package is installed, writes a
   résumé line of the Training type and posts a note on the employee record.
6. The attendee may like or dislike a content item, which writes the vote on the progress record;
   the like endpoint applies [`LSG-090b`](business-rules.md#lsg-090b).
7. The attendee may comment a content item, gated by
   [`LSG-080`](business-rules.md#lsg-080), and may review the course, gated by
   [`LSG-075`](business-rules.md#lsg-075). A review grants the course review reward with the reason
   `Course Ranked`.
8. The next content to take is the first published, active, non-section item the attendee has not
   completed; the continue button points at it. After an item is completed, the page decides which
   section to open next: the first section when the completed item was uncategorised and every
   uncategorised item is now done; the following section when the completed item's section is now
   entirely done and it is not the last; otherwise none.

---

<a id="workflow-11"></a>

## Workflow 11 — Taking a quiz

**Actor:** Attendee.

1. The attendee opens the content item carrying the quiz. The quiz panel is served as data, not as
   markup, and hides what the attendee must not see: the correctness of an answer is disclosed only
   when the attendee has already completed the content or is a site designer, and the per-answer
   comment only to a site designer.
2. The attendee picks one answer per question. Answers may be saved into the browser session before
   signing in, keyed by content item, so an anonymous visitor who registers does not lose their
   work; those saved answers are dropped as soon as the quiz is submitted or the attendee leaves the
   course.
3. The attendee submits the chosen answer identifiers.
   [`LSG-086`](business-rules.md#lsg-086) refuses an anonymous visitor, an already completed content
   and an incomplete submission.
4. The attempt counter of the progress record is incremented by one, without taking a lock.
5. When every chosen answer is correct, the content item is marked completed, which grants the
   reward of the current attempt index by
   [`LSG-CALC-012`](calculations.md#lsg-calc-012) with the reason `Quiz Completed`, and triggers the
   course completion recomputation.
6. The answer is returned per question, carrying the correctness and the per-answer comment,
   together with the completion marker, the new course completion percentage, the points won, the
   points the next attempt would win, the attempt count and the progression of the attendee's rank.
7. Resetting a quiz sets the progress record back to not completed with a zero attempt count. Adding
   or updating a question on a content item also resets the author's own progress record, so the
   author can retake the quiz.

---

<a id="workflow-12"></a>

## Workflow 12 — Taking a course certification

**Actor:** Attendee. **Precondition:** the course holds a content item of the certification category
pointing at a questionnaire.

1. The attendee opens the certification content item and presses the start action. An address is
   produced, according to the attendee's situation:
   - **Enrolled with existing attempts.** The most recent participation attached to their progress
     record is reused and its start address is returned, so an interrupted certification resumes.
   - **Enrolled without attempts.** A participation is created for the attendee's contact, without
     the attempt check, carrying the content item and the progress record, and with a **freshly
     generated invite token**. The fresh token opens a new attempt pool, which is what makes the
     allowance restart when an attendee re-enrols.
   - **Not enrolled.** A test participation is created for the reader's contact, carrying the
     content item but no progress record. Test participations never mark anything completed and are
     excluded from every statistic.
2. The attendee answers the questionnaire as in [workflow 4](#workflow-4).
3. **Passing.** When any participation attached to the progress record passes, the certification
   marker on that progress record becomes true. Two consequences follow: the completion marker of
   the progress record is forced true, which counts the certification towards the course completion;
   and every enrolment of the same contact on the same course that is not yet marked certified
   becomes certified.
4. **Failing the last attempt.** [`LSG-134`](business-rules.md#lsg-134) applies: the failure message
   is sent and the attendee is removed from the course, with the attempt links cleared so a new pool
   starts on re-enrolment.
5. **Badge category.** [`LSG-135`](business-rules.md#lsg-135) keeps the challenge of the
   questionnaire's badge in the right category.

---

<a id="workflow-13"></a>

## Workflow 13 — Selling access to a course

**Actor:** Customer and Salesperson. **Precondition:** the course-selling package is installed, the
course's enrolment policy is `payment` and it carries a product whose service tracking is `course`.

1. The officer links a product to the course. When exactly one product with that service tracking
   exists in the catalogue it becomes the default. The sales description of the product becomes
   `Access to: ` followed by the course names.
2. The customer adds the product to the cart. [`LSG-131`](business-rules.md#lsg-131) lets the
   product be added whatever the ordinary rules say; [`LSG-130`](business-rules.md#lsg-130) keeps the
   quantity at one.
3. The customer pays and the order is confirmed. Confirming the order looks up every course whose
   enrolment policy is `payment` and whose product appears on a line, and enrols the customer into
   each of them with the status `joined`.
4. The revenue of the course is the total of the confirmed sales-analysis amounts of its product,
   shown in the product currency and visible only to the salesperson group.
5. Publication stays in step in both directions by [`LSG-132`](business-rules.md#lsg-132).
6. The ledger consequences belong entirely to the selling and receivable domains; see
   [`accounting-effects.md`](accounting-effects.md).

---

<a id="workflow-14"></a>

## Workflow 14 — Leaving a course, and the expiry of invitations

1. **Leaving.** The attendee presses leave. The enrolment is archived, the contact is unsubscribed
   from the course discussion thread, and the progress records are **kept**, which is deliberate:
   re-joining restores the previous progress and therefore stops the same reputation points being
   earned twice ([`LSG-136`](business-rules.md#lsg-136)). The quiz answers held in the browser
   session for that course are dropped. When the certification package is installed the links
   between the attendee's participations on that course and their progress records are cleared. When
   the skills package is installed a note is posted on the employee record.
2. **Re-joining.** The enrolment is un-archived, the requested status is written and the completion
   is recomputed at once.
3. **Expiry.** The periodic cleanup deletes every enrolment whose status is `invited`, whose
   completion is zero, and whose last invitation instant is either empty or older than three months.
   A missing last-invitation instant counts as expired.

---

<a id="workflow-15"></a>

## Workflow 15 — Asking and answering on a forum

**Actor:** Forum Member.

1. The member opens the ask page. [`LSG-115b`](business-rules.md#lsg-115b) redirects a member who
   already has a question waiting for validation, and
   [`LSG-115c`](business-rules.md#lsg-115c) redirects a member whose account carries no valid
   address.
2. The member writes a title, a body and tags. Typed tag entries are resolved by the forum: an entry
   beginning with an underscore is a new label; an existing tag of that name is reused; otherwise a
   new tag is created when the member reaches the tag-creation threshold
   ([`LSG-110`](business-rules.md#lsg-110)).
3. The member submits. The content is rewritten by the two rules of section 20.5 of
   [`entities.md`](entities.md); [`LSG-109`](business-rules.md#lsg-109) may refuse it.
   [`LSG-091`](business-rules.md#lsg-091) checks the ask threshold. A member at or above the publish
   threshold creates an active question and gains the new-question grant with the reason `Ask a new
   question`; a member below it creates a pending question and gains nothing.
4. The new-question notification is posted to the followers of the forum and of the tags. For a
   pending question a validation notice is posted instead, as an internal note, to the followers who
   reach the moderation threshold.
5. Another member answers. [`LSG-092`](business-rules.md#lsg-092) checks the answer threshold and
   [`LSG-093`](business-rules.md#lsg-093) refuses an answer on a closed or archived question. The
   answer is titled `Re: <question title>`, the new-answer notification is posted on the parent
   question, and the question's last-activity instant is refreshed.
6. Members comment. [`LSG-108`](business-rules.md#lsg-108) checks the comment threshold. A comment on
   an answer also reaches the followers of the parent question who subscribed to the comment
   subtype; the record name shown in the notification is the parent question's title; the message is
   posted with elevated rights so a member who may comment but may not read the follower list still
   notifies them; comments never become inbox notifications, only outgoing messages, which keeps the
   inbox usable on a busy forum; and the question's last-activity instant is refreshed.
7. Editing a post posts a notification: `Answer Edited` under the answer-edited subtype on the
   parent question for an answer, and `Question Edited` under the question-edited subtype on the
   question itself.

---

<a id="workflow-16"></a>

## Workflow 16 — Moderating a forum

**Actor:** Forum Moderator. **Precondition:** the moderator's balance reaches the moderation
threshold; every queue answers not-found otherwise
([`LSG-115a`](business-rules.md#lsg-115a)).

1. **Validation queue.** Holds the posts in state `pending`. Validating a post writes the active
   state, sets the archiving marker to true, records the moderator, grants the new-question amount
   to the author with the reason `Ask a question`, and posts the state notification. Refusing
   records only the moderator; the interface archives the post in a separate step.
2. **Flagged queue.** Holds the posts in state `flagged`, ordered by last change descending and
   optionally narrowed by a title search. A moderator may accept a post back, which returns it to
   active, or mark it offensive, which asks for a reason among the offensive reasons. A batch
   operation marks as offensive every flagged post grouped by author, by author country, or by
   explicit selection, always using the spam reason.
3. **Offensive queue.** Holds the posts in state `offensive`.
4. **Closed queue.** Holds the posts in state `close`.
5. **Closing a question.** Asks for a reason among the basic reasons, records the closing user, the
   closing instant and the reason, and moves points when the reason is the offensive or the spam one
   ([`LSG-CALC-022`](calculations.md#lsg-calc-022)). Closing never applies to answers.
6. **Reopening.** Returns the question to active and gives back the points removed at closing.
7. **Marking offensive.** Removes the flagged amount from the author with the reason `Downvote for
   posting offensive contents`, records the moderator, the closing instant and the reason, and sets
   the archiving marker to false.

---

<a id="workflow-17"></a>

## Workflow 17 — Voting and accepting an answer

**Actor:** Forum Member.

1. **Pressing upvote.** When the reader is the author of the post the endpoint answers `own_post`
   and nothing happens. Otherwise the intent is "upvote" when the reader's current value is not
   strictly positive, and "cancel" when it is.
2. **Pressing downvote.** When the reader is the author the endpoint answers `own_post`. Otherwise
   the intent is "upvote" when the reader's current value is strictly negative — that is, cancelling
   a downvote — and "downvote" otherwise.
3. **The new value** follows from the existing value and the intent:

   | Existing value | Intent upvote | Intent downvote |
   |---|---|---|
   | none | `1` | `-1` |
   | `1` | `1` | `0` |
   | `-1` | `0` | `-1` |
   | `0` | `1` | `-1` |

4. The three checks [`LSG-113`](business-rules.md#lsg-113),
   [`LSG-114`](business-rules.md#lsg-114) and [`LSG-115`](business-rules.md#lsg-115) run, then the
   movement of [`LSG-CALC-021`](calculations.md#lsg-calc-021) is applied to the post's author.
5. The endpoint answers the new vote total of the post and the reader's new value.
6. **Accepting an answer.** Pressing accept refuses with `own_post` when the reader wrote that
   answer; otherwise the acceptance marker is cleared on every **other** answer of the same question
   and toggled on this one. [`LSG-100`](business-rules.md#lsg-100) guards the write. The points move
   as described in section 20.7 of [`entities.md`](entities.md), and the question's answered marker
   becomes true.

---

<a id="workflow-18"></a>

## Workflow 18 — Converting between answers and comments

**Answer to comment.**

1. The operation returns without doing anything when the post has no parent.
2. [`LSG-105`](business-rules.md#lsg-105) checks the conversion threshold.
3. A comment is posted on the parent question, authored by the answer's author, dated with the
   answer's creation instant and carrying the answer's body stripped of its attributes, styles and
   classes.
4. The original answer is deleted with elevated rights, which bypasses the deletion threshold.

**Comment to answer.**

1. The operation returns without doing anything when the comment has no author, or the author has no
   user account, or the author already answered that question.
2. [`LSG-106`](business-rules.md#lsg-106) checks the conversion threshold.
3. A new answer is created, authored by the comment's author, on the question — the parent of the
   commented post when the comment sat on an answer — titled `Re: <question title>` and carrying the
   comment body.
4. The comment is deleted.

**Deleting a comment.** [`LSG-107`](business-rules.md#lsg-107) applies. A comment that does not
belong to the named post, or that is not attached to a post at all, is skipped.

---

<a id="workflow-19"></a>

## Workflow 19 — Running the daily recognition job

**Actor:** the scheduled job runner. **Frequency:** once a day.

1. Every challenge in state `draft` whose start date is today or earlier is written to
   `inprogress`, which recomputes its participants and generates its goals.
2. Every challenge in state `inprogress` whose end date is strictly before today is written to
   `done`, which forces the reward check.
3. The challenges still in progress are then updated:
   1. The goals to re-measure are selected: those belonging to the challenges, not closed, either in
      progress or reached with an end date of yesterday or later, and whose owner has interacted
      with the client since the goal was last written and holds a session that is still valid. This
      keeps the job away from the goals of people who are not using the system.
   2. Those goals are measured; see [`gamification.md`](gamification.md#goal-update).
   3. The participants are recomputed from the stored filter of each challenge, and the missing
      goals are generated.
   4. For each challenge whose last report was not sent today: when today is at or after the next
      report date, the report is sent; otherwise the goals of that challenge that started on or
      after the last report date and ended on or before it are collected, and a final report
      restricted to them is sent when there are any.
   5. The reward check of section 6.3 of [`state-machines.md`](state-machines.md#machine-6) runs.
4. When the job is asked to commit between steps, it commits after each challenge, so a failure late
   in the run does not undo the earlier challenges.

---

<a id="workflow-20"></a>

## Workflow 20 — Granting a badge by hand

**Actor:** any user, within the granting rule of the badge.

1. The user opens the badge and presses the grant action, which opens the granting dialogue with the
   badge pre-filled. From an employee record the employee dialogue opens instead, with the employee
   pre-filled and the user derived from it.
2. The user picks a beneficiary and writes an optional comment.
   [`LSG-120`](business-rules.md#lsg-120) refuses granting to oneself.
3. Confirming creates the grant. The granting check of the badge runs first:
   [`LSG-116`](business-rules.md#lsg-116) to [`LSG-119`](business-rules.md#lsg-119).
4. The grant notifies the beneficiary's contact with the badge message, whose subject is `🎉 You've
   earned the <badge name> badge!`; the notification carries no button to open the record.
5. The badge counters and the three per-level counters of the beneficiary are recomputed.

---

<a id="workflow-21"></a>

## Workflow 21 — Consolidating the movement ledger

**Actor:** the scheduled job runner. **Frequency:** once a month, on the first day of the month.

The procedure and the worked example are in
[`LSG-CALC-033`](calculations.md#lsg-calc-033). In short: the movements of the month that ended two
months earlier are replaced, per user, by one movement carrying the oldest old value and the newest
new value of that month; the originals are deleted; the sources and reasons of the originals are
lost; and the whole operation runs with the balance recomputation suppressed, so no balance changes.

---

<a id="workflow-22"></a>

## Workflow 22 — The public profile and the address validation

**Actor:** any signed-in user.

1. The user opens their public profile page. The page shows the balance, the current rank, the
   progress to the next rank computed by
   [`LSG-CALC-017`](calculations.md#lsg-calc-017), the badges earned and, when the profile is
   published, the biography, the city, the country and the personal site address.
2. The user edits those four fields and the publication marker; the platform allows a user to write
   exactly those about themselves and to read their own balance.
3. The user asks for a validation message. A token is built as a digest of the current day, a stored
   secret, the user identifier and the address, so it is valid for the day it was issued. The
   validation message carries an address holding the token, the user identifier and the address.
4. Following that address compares the token. When it matches **and** the balance is still exactly
   zero, three reputation points are granted; otherwise nothing happens. The visitor is then
   redirected to the address the caller asked for, or to the site root.
5. Reaching a new rank sends the shipped rank message, unless the change happens while a package is
   being installed. The message offers the destinations contributed by the installed packages: `See
   our Forum` pointing at `/forum` and `See our eLearning` pointing at `/slides`.
6. The member list page ranks users by the gain recorded in the movement ledger over a chosen range,
   or by the total balance, using the two ranking helpers described in section 34 of
   [`entities.md`](entities.md).
