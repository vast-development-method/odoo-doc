# The questionnaire engine

Everything a replacement needs in order to reproduce the behaviour of a questionnaire: its five
purposes, its three layouts, the nine question shapes with their validation, the conditional display
mechanism, the four scoring modes, the certification chain, the attempt and time limits, the
invitation mechanism, the live session, the result statistics and the generation of leads.

<a id="purposes"></a>

## 1. Purposes

A questionnaire carries a purpose, which is only a package of defaults applied when the author picks
it in the form. Nothing enforces the package afterwards: an author may change any of the settings
back.

| Purpose | What choosing it forces |
|---|---|
| `survey` | `certification` false, `is_time_limited` false, `scoring_type` `no_scoring`. |
| `live_session` | `access_mode` `public`, `is_attempts_limited` false, `is_time_limited` false, `progression_mode` `percent`, `questions_layout` `page_per_question`, `questions_selection` `all`, `scoring_type` `scoring_with_answers`, `users_can_go_back` false. |
| `assessment` | `access_mode` `token`, `scoring_type` `scoring_with_answers`. |
| `custom` | Nothing. |
| `recruitment` | Nothing; the value exists only when the recruitment interview package is installed and is chosen by that package, never by the questionnaire application. |

The purpose also decides which questionnaires each application lists. The questionnaire application
lists only the four general purposes, which excludes the specialised interview forms; the
participation list, the detailed answer list and the record rules of the officer and the
administrator filter on the same four values.

A questionnaire may be run as a live session only when the derived session marker is true, that is
when the purpose is `live_session` or `custom` **and** the questionnaire is not a certification.

Four sample questionnaires may be loaded into a new record with one action; a fifth, Lead
Qualification, is added by the lead-generation package and is described in section 17.

<a id="structure"></a>

## 2. Structure: sections and questions

A questionnaire owns one flat, ordered list of entries. An entry marked as a page is a **section**;
an entry not so marked is a **question**. The section a question belongs to is not stored by the
author; it is derived by walking the ordered list and remembering the last section seen.

Consequences a replacement must reproduce:

1. A question placed before the first section belongs to no section. In the statistics and on the
   result page such questions are grouped under the label `Uncategorized`.
2. Moving a section moves, implicitly, every question that follows it.
3. A section may carry a description. A section **with** a description is itself a displayable
   screen; a section **without** one is never displayed on its own in the one-page-per-question
   layout.
4. A section may carry a background picture; a question may not. A question inherits the background
   of its section and falls back to the questionnaire background when neither has one.
5. A section may not carry a question shape; the refusal is
   [`LSG-011`](business-rules.md#lsg-011).
6. The ordering key is the sequence, with the identifier as a tie-break. When every entry was
   created without ever being reordered, every sequence may be equal; the position of an entry in
   the ordered list is then used instead, which keeps the derived section assignment correct.

<a id="layouts"></a>

## 3. Layouts and navigation

| Layout | One screen holds | Progress indicator basis |
|---|---|---|
| `page_per_question` | One question, or one section that has a description. | The list of questions actually asked to this participant. |
| `page_per_section` | Every question of one section. | The list of sections. |
| `one_page` | Every question of the questionnaire. | Not displayed. |

The list of screens a participant will see is computed as follows:

- In `page_per_section`, the screens are the sections.
- In `page_per_question` with random selection and no open session, the screens are exactly this
  participation's drawn questions.
- In `page_per_question` otherwise, the screens are the valid sections and the valid questions: a
  section is valid when its description is not empty, and a question is valid unless it is
  conditional and every one of its triggers is invalid (section 6).
- In `one_page`, there is a single screen.

<a id="screen-navigation"></a>

### 3.1 Moving to the next screen

Given the current screen, the next screen is the first later entry that is displayable.

1. When the current screen identifier is zero, the first screen of the list is returned.
2. When the current screen is the last of the list, nothing is returned, which means the
   questionnaire is finished.
3. In `page_per_question`, the candidates are the entries after the current one, in order. A section
   candidate is returned when it holds at least one question that is not currently inactive, or when
   it holds no question at all and its description is not empty. A question candidate is returned
   when it is not currently inactive.
4. In `page_per_section`, the candidates are the sections after the current one, in order. A section
   is returned when it holds at least one question that is not currently inactive, or when it holds
   no question and its description is not empty. When no section qualifies, nothing is returned.

Moving back reverses the candidate list and applies the same tests.

### 3.2 Roaming

Moving back is possible only when all of the following hold:

1. The layout is not `one_page`.
2. Roaming is switched on.
3. The participation is in progress.
4. The participation is not a live-session attempt.
5. The current screen is not the first section of the questionnaire, when the questionnaire has
   sections.
6. In `page_per_question`, the current screen is not the first drawn question.

In `page_per_section` condition 6 does not apply: reaching a section other than the first is enough.

Roaming changes how mandatory questions behave. While roaming is on, a mandatory question may be
left empty without raising a message; the skipped mandatory questions are recorded as skips. After
the participant reaches the end of the questionnaire for the first time, the navigation switches to
a second pass that walks only the skipped mandatory questions:

- The first-submitted marker is set to true when the end is reached and at least one mandatory
  question is still skipped.
- The next skipped screen is the first entry — among the sections in `page_per_section` and among
  the questions otherwise — that carries a skipped mandatory answer and comes after the last
  displayed screen; when the last displayed screen is not in that list, or is the last of it, the
  walk loops back to the first.
- The final submit button appears only when the current screen is the last skipped screen.

Roaming may not be combined with the scoring mode `scoring_with_answers_after_page`; the refusal is
[`LSG-009`](business-rules.md#lsg-009).

## 4. Question shapes

| Shape | The participant enters | Stored in | Scorable |
|---|---|---|---|
| `text_box` | Free multi-line text. | `value_text_box` | No. |
| `char_box` | A single line of text. | `value_char_box` | No. |
| `numerical_box` | A number. | `value_numerical_box` | Yes, when a correct number is set. |
| `scale` | A whole number between the scale minimum and maximum. | `value_scale` | No, but it produces numerical statistics. |
| `date` | A date. | `value_date` | Yes, when a correct date is set. |
| `datetime` | A date and a time. | `value_datetime` | Yes, when a correct instant is set. |
| `simple_choice` | Exactly one of the proposed options. | One answer row with the chosen option. | Yes, when at least one option is marked correct. |
| `multiple_choice` | Any number of the proposed options. | One answer row per chosen option. | Yes, when at least one option is marked correct. |
| `matrix` | For each row, one option in the `simple` subtype or several in the `multiple` subtype. | One answer row per chosen cell, carrying both the row and the option. | No. |

Additional per-shape facts:

- A choice or matrix question may also show a free-text comment box, captioned by the comment
  message. When comments count as answers, a filled comment satisfies the mandatory check and is
  counted as an answer in the statistics under the label `Other (see comments)`; when they do not,
  the comment is stored but never counted.
- An answer option may carry a picture instead of, or beside, its text. When at least one option of
  a question has an empty text, the question is rendered as a picture grid and every option is
  labelled with the capital letter of its position: the first is `A`, the second `B`, through the
  twenty-seventh; beyond that the label is empty.
- A scale question shows three captions: under the lowest value, under the middle value and under
  the highest value.
- A single-line text question may be marked to save the answer as the participant's electronic mail
  address — only when it is also validated as an address — or as the participant's nickname. Either
  marker copies the answer onto the participation, and the pre-filled value is written into the
  participation's answers as soon as the participation is created.

<a id="answer-validation"></a>

## 5. Validation of an answer

Validation happens on submit, question by question, and returns at most one message per question.
The order of the checks is exactly this:

1. A text answer is trimmed of its leading and trailing spaces.
2. **Empty answer to a non-choice question.** When the answer is empty and the shape is neither
   `simple_choice` nor `multiple_choice`: when the question is mandatory and roaming is off, the
   message is the question's own error message, or `This question requires an answer.` when that
   field is empty. When the question is not mandatory, or roaming is on, no message is produced.
3. Otherwise the per-shape check runs.

| Shape | Checks, in order | Message |
|---|---|---|
| `char_box` | The address check, when it is required. | `This answer must be an email address` |
| | The length check, when validation is required. | The question's validation error message, or `The answer you entered is not valid.` |
| `numerical_box` | The answer must read as a number. | `This is not a number` |
| | The range check, when validation is required. | The question's validation error message, or `The answer you entered is not valid.` |
| `date` and `datetime` | The answer must read as a date, respectively as an instant. | `This is not a date` |
| | The range check, when validation is required. Each bound applies only when it is set. | The question's validation error message, or `The answer you entered is not valid.` |
| `simple_choice` and `multiple_choice` | The count of selected options, plus one when a comment was typed and comments count as answers, must not be zero on a mandatory question with roaming off. | The question's own error message, or `This question requires an answer.` |
| | The count must not exceed one on `simple_choice`. | `For this question, you can only select one answer.` |
| `matrix` | On a mandatory question, the number of answered rows must equal the number of rows. | The question's own error message, or `This question requires an answer.` |
| `scale` | On a mandatory question with roaming off, a value must be given. | The question's own error message, or `This question requires an answer.` |

The address check accepts any string of the form local part, at sign, host, dot, extension.

When at least one message is produced and neither countdown has run out, the whole submit is refused
and the messages are returned field by field; nothing is stored. When a countdown has run out, the
messages are ignored, the valid answers are stored and the participation is forced to completed.

## 6. Conditional questions

A question is displayed only when at least one of its **triggering answer options** has been chosen.
An empty trigger list means the question is always displayed.

### 6.1 What may be a trigger

The picker offers only answer options that satisfy all of:

1. They belong to a question of the same questionnaire.
2. That question's shape is `simple_choice` or `multiple_choice`.
3. That question is positioned before the conditional question: either its sequence is strictly
   lower, or the sequences are equal and its identifier is lower.

When a question is later moved before one of its triggers, the misplaced marker becomes true and the
form warns the author. The stored trigger is not removed; the question is treated as invalid for the
screen list instead.

### 6.2 Validity of a trigger at display time

Building the screen list, a conditional question is dropped entirely when **none** of its triggers
is valid. A trigger is valid when all of:

- It is a question, not a section.
- Its shape is `simple_choice` or `multiple_choice`.
- It is positioned before the conditional question.
- It is not itself a conditional question that was already found invalid.

The check runs over the questions sorted in display order, which makes the "already found invalid"
test well defined: a chain of conditional questions is validated from the top down.

### 6.3 Which questions are inactive for a given participation

For a participation, the inactive questions are those that carry at least one triggering option and
none of whose triggering options appears among the options the participant chose. The set of chosen
options is read from every answer row of the participation that points at an option, which
incidentally includes matrix cells; a matrix option can never be a trigger, so it never makes a
question active.

### 6.4 Cleaning

After every submit of a non-session participation, the answers recorded for questions that are
currently inactive are deleted. This matters because a participant may choose a trigger, answer the
conditional question, then go back and un-choose the trigger: without cleaning, the stale answer
would keep contributing to the score and would keep activating further questions.

When a participation is completed, its predefined question set is reduced by the inactive questions,
so the score denominator counts only the questions the participant could reach.

### 6.5 Interaction with random selection

Conditional questions are ignored entirely when the questionnaire draws random questions: the
trigger maps are not built at all in that mode.

### 6.6 Where the decision is taken

- In `page_per_question`, the server decides which question comes next.
- In `page_per_section` and `one_page`, the client shows and hides questions live as options are
  ticked, and the server cleans up afterwards. To make that possible the server hands the client two
  maps — for each conditional question the list of options that trigger it, and for each option the
  list of questions it triggers — plus the list of options already chosen.

## 7. Random question selection

When the selection mode is `random`, the set of questions asked to a participant is drawn once, at
the creation of the participation, and stored on it. The draw is:

1. Every question that belongs to no section is always included.
2. For each section, in order: when the section's random count is strictly between zero and the
   number of questions in the section, that many questions are drawn at random without repetition;
   otherwise every question of the section is included.

Because the draw is stored, resuming an interrupted participation asks exactly the same questions.
Because the score denominator is computed from the drawn set, two participants of the same
questionnaire may have different maximum obtainable scores.

Random selection is ignored during a live session.

## 8. Scoring

### 8.1 Modes

| Mode | Scores computed | Correct answers revealed |
|---|---|---|
| `no_scoring` | No. | Never. |
| `scoring_with_answers_after_page` | Yes. | After each submitted screen, for the scorable questions of that screen that were not inactive. |
| `scoring_with_answers` | Yes. | On the closing screen and on the printable view. |
| `scoring_without_answers` | Yes. | Never; only the final score is shown. |

### 8.2 Where a point comes from

The three cases and the full arithmetic are
[`LSG-CALC-003`](calculations.md#lsg-calc-003); the totals, the percentage and the pass verdict are
[`LSG-CALC-002`](calculations.md#lsg-calc-002); the questionnaire maximum is
[`LSG-CALC-001`](calculations.md#lsg-calc-001); the live-session adjustment is
[`LSG-CALC-005`](calculations.md#lsg-calc-005).

### 8.3 Revealing the correct answers after a page

When the mode is `scoring_with_answers_after_page`, the submit returns, together with the next
screen, a map from question to correct answer for the scorable questions of the submitted screen
that were not inactive:

- For choice questions, the identifiers of the options marked correct.
- For a numerical question, the correct number.
- For a date question, the correct date formatted for the reader.
- For a date-and-time question, the correct instant formatted for the reader in coordinated
  universal time.

Questions with no correct answer are not included.

## 9. Certification

A certification is a scored questionnaire whose certification marker is true. Turning the marker on
while the questionnaire is unscored is impossible: [`LSG-003`](business-rules.md#lsg-003) refuses
it, and the derivation forces the scoring mode to `scoring_without_answers` when a certification is
created with no mode.

What a certification adds:

1. **A printable certificate**, produced as described in
   [`workflows.md`](workflows.md#workflow-7).
2. **An automatic message** carrying that document, sent when the participation reaches the
   completed state, the participation passed, a certificate template is configured and the attempt
   is not a test.
3. **A badge**, through the chain of [`workflows.md`](workflows.md#workflow-6).
4. **A download endpoint**, so a signed-in user may fetch the certificate of any certification they
   passed.
5. **A preview**, so an officer may see the certificate without a real participant.
6. **A validity period** in months, zero meaning that the certification never expires.

A certification may never be run as a live session.

## 10. Access, attempts and time

### 10.1 Access mode and signing in

| Access mode | Sign-in required | Who may start |
|---|---|---|
| `public` | no | Anybody holding the questionnaire link. A participation is created at every landing on the start address. |
| `public` | yes | Anybody holding the link, after signing in. |
| `token` | no | Only somebody holding a participation token, that is, an invited person. |
| `token` | yes | Only an invited person, after signing in. |

The six refusals that stop a participation being created are
[`LSG-021`](business-rules.md#lsg-021) to [`LSG-026`](business-rules.md#lsg-026).

### 10.2 Attempt pools

Attempts are counted only when the questionnaire limits attempts **and** either the access mode is
not `public` or signing in is required. The arithmetic is
[`LSG-CALC-006`](calculations.md#lsg-calc-006).

Attempt limiting is impossible on a questionnaire that holds conditional questions, and impossible
on a fully anonymous public questionnaire; the derivation forces the marker back to false in both
cases.

### 10.3 Time limits

Two independent countdowns exist, with the grace margins of
[`LSG-CALC-007`](calculations.md#lsg-calc-007). Reaching the questionnaire countdown completes the
participation the next time a page is requested; reaching the question countdown ends the answering
window of a session question.

### 10.4 Deadline

A participation may carry a deadline. When the deadline is in the past, every access is refused and
the closed-or-expired page is shown ([`LSG-059`](business-rules.md#lsg-059)). The deadline is
written by the invitation composer and carried over to a retry.

<a id="validity-codes"></a>

## 11. Validity checks performed before any page is served

Evaluated in order, each producing a status word.

| Status | Condition | What the visitor sees |
|---|---|---|
| `survey_wrong` | No questionnaire matches the token. | Redirected to the site root. |
| `token_wrong` | A participation token was given but matches nothing. | The access-error page. |
| `token_required` | No participation token, and either the endpoint requires one or the access mode is `token`. | Redirected to the site root, except on the printable endpoint where an officer may still print. |
| `survey_auth` | Signing in is required and the visitor is anonymous. | The authentication-required page, carrying a link to sign in, or a self-registration link built for the invited contact when that contact has no account and self-registration is allowed. |
| `survey_closed` | The questionnaire is archived and the participation is not a test. | The closed-or-expired page, when the visitor could otherwise answer. |
| `survey_void` | The questionnaire holds no question, or the layout is `page_per_section` and it holds no section. | The empty-questionnaire page, when the visitor could otherwise answer. |
| `answer_deadline` | The participation carries a deadline in the past. | The closed-or-expired page. |
| `answer_wrong_user` | The visitor is anonymous, the participation names a contact and the token did not come from the address; or the visitor is signed in and the participation belongs to another contact. | The access-error page. |

An officer may open the printable view of a questionnaire that is archived, empty or
invitation-only, because those three statuses are tolerated on that endpoint when the reader has
read access.

## 12. Invitations

The composer is the only way to create participations in bulk; the procedure is
[`workflows.md`](workflows.md#workflow-3) and the five preconditions are
[`LSG-027`](business-rules.md#lsg-027) to [`LSG-031`](business-rules.md#lsg-031).

## 13. Live sessions

The whole cycle is [`workflows.md`](workflows.md#workflow-5) and the state machine is
[`state-machines.md`](state-machines.md#machine-2). Three points deserve emphasis here.

**The one-second grace delay.** The current-question start instant is written as the current instant
**plus one second**, and the broadcast carries the *real* current instant. The extra second
compensates the server round trip so that every attendee starts the countdown at the same moment.

**The artificial participation.** Conditional questions are resolved against a participation built
from the most-voted answer of each question already answered in the session; ties go to the first
option encountered with the maximal count.

**The speed bonus.** [`LSG-CALC-005`](calculations.md#lsg-calc-005). The per-question countdown
settings stay aligned with the questionnaire settings by the re-alignment rule of section 2.9 of
[`entities.md`](entities.md).

A live session on a questionnaire with no question renders the empty-questionnaire page rather than
a blank screen.

## 14. Result statistics

The result page is built from a set of participation answers, by default every answer of every
non-test participation that is not in the new state, narrowed as filters are applied.

### 14.1 Base filters of the page

| Toggle | Effect on the participation filter |
|---|---|
| none | The state is not `new`, the attempt is not a test entry, and the questionnaire is this one. |
| finished | The state is `done` instead of "not new". |
| failed | Adds: the pass verdict is false. |
| passed | Adds: the pass verdict is true. |

The failed toggle takes precedence over the passed toggle when both are present.

### 14.2 Answer filters

A visitor may click a bar or a value to restrict the whole page to the participations that gave that
answer. Each filter is encoded as a triple — a marker, a row identifier and a record identifier —
joined by commas, with the triples joined by vertical bars.

- The marker `A` points at an answer option. A row identifier of zero means a choice filter; a
  non-zero row identifier means a matrix-cell filter on that row.
- The marker `L` points at a participation answer row, used for the free-text, numerical, date and
  date-and-time shapes. Only triples whose row identifier is zero are accepted.

Each filter becomes a sub-condition on the answer rows:

| Filter kind | Condition on an answer row |
|---|---|
| Choice option | The question is the option's question and the chosen option is that option. |
| Matrix cell | The question is the option's question, the row is that row and the chosen option is that option. |
| Single-line text | The question is that question and the single-line value contains the value, compared without regard to case. |
| Multi-line text | The question is that question and the free-text value contains the value, compared without regard to case. |
| Number | The question is that question and the numerical value equals the value. |
| Scale | The question is that question and the scale value equals the value. |
| Date | The question is that question and the date equals the value. |
| Date and time | The question is that question and the instant equals the value. |

The filters combine with a logical and over the **participations**: a participation is kept only
when it holds, for each filter separately, at least one answer row matching that filter. Every
answer row of the kept participations is then loaded, so the whole page describes the sub-population
that answered as chosen. Each active filter is displayed as a removable chip carrying the question
title and the answer value, or `<row value> : <option value>` for a matrix cell.

### 14.3 Per-question and questionnaire-level statistics

The per-question summaries, tables and graphs are
[`LSG-CALC-008`](calculations.md#lsg-calc-008); the page totals are
[`LSG-CALC-009`](calculations.md#lsg-calc-009). For the free-text, single-line text, numerical, date
and date-and-time shapes the page additionally lists every non-skipped answer with its value and a
link to the printable view of the participation it belongs to.

### 14.4 Per-participation statistics

For the printable view of one participation, the scored questions of that participation are
classified per section — under the label `Uncategorized` when the question has no section — into
four buckets:

| Bucket | One-answer choice question | Several-answer choice question | Any other scored shape |
|---|---|---|---|
| `correct` | The chosen option is marked correct. | The set of correct options chosen equals the full set of correct options. | The answer is not skipped and is correct. |
| `partial` | The chosen option is not correct but carries a strictly positive score. | The set of correct options chosen is a strict, non-empty subset of the correct options. | Never. |
| `incorrect` | An option was chosen and it is neither correct nor positively scored. | No correct option was chosen and at least one non-skipped option was. | The answer is not skipped and is not correct. |
| `skipped` | No option was chosen. | Nothing was chosen. | The answer is skipped. |

Comment rows are excluded from this classification unless the question counts comments as answers.
Each section carries the four counts plus the number of questions, and the participation carries the
four totals under the labels `Correct`, `Partially`, `Incorrect` and `Unanswered`.

## 15. Printable view

The printable view shows the questions that should have been displayed, that is, the questionnaire's
questions minus the inactive conditional ones. For a live-session participation the inactive set is
computed from the session's most-voted answers rather than from the participation's own answers.

When a participation token is supplied, the answers of that participation are shown; the correct
answers are shown as well when the scoring mode is `scoring_with_answers` or
`scoring_with_answers_after_page`, and the per-participation statistics are attached so the page can
draw the correct, partial, incorrect and unanswered chart.

## 16. Sample questionnaires

Four samples are shipped with the questionnaire application and are loaded into a new record by one
action: a general survey sample, an assessment sample, a live-session sample and a custom sample.
The lead-generation package adds a fifth, described next. A sample creates its sections, its
questions and its answer options in the same operation as the questionnaire itself, so the author
starts from a complete example.

<a id="lead-generation"></a>

## 17. Generating leads from answers

Installed by the lead-generation package.

1. An author marks one or more answer options as lead-generating. A question becomes lead-generating
   when its shape is `simple_choice`, `multiple_choice` or `matrix` and at least one of its options
   is marked. A questionnaire becomes lead-generating when its purpose is `survey`, `live_session`
   or `custom` and at least one of its questions is.
2. When a participation reaches the completed state and the purpose is one of those three, the
   participations that chose at least one lead-generating option are collected and one lead is
   created per participation, in one batch. When a session is closed instead, the participations
   created at or after the session start instant are collected the same way, because the mass
   completion of a session does not run the individual completion procedure.
3. The values common to every lead of a questionnaire are: the medium, taken from or created as a
   campaign medium named `Survey`; the originating questionnaire; the source, taken from or created
   as a campaign source named after the questionnaire title; the sales team of the questionnaire;
   the kind, always an opportunity, because the answers are assumed to qualify the contact; and the
   salesperson, which is the questionnaire's responsible when that person belongs to the chosen
   team, otherwise the team leader, otherwise nobody.
4. The values specific to one lead are: the contact name, taken from the participation's contact
   name, its address, or the nickname; the title, `<participant name> survey results` for an
   ordinary participation and `<participant name> live session results` for a session attempt, where
   the participant name falls back to the nickname, then the address, then `New`; the contact link,
   set when the participation names an active contact; the address, set instead when only an address
   is known; and the description, built from the answers.
5. The description lists one entry per question, in five shapes: `<question> — <answer 1>, <answer
   3>` for a choice question; the question followed by one indented line per answered cell as `<row
   label> — <option>` for a matrix question; the question followed by one indented line per line of
   text for a multi-line answer; `<question> — <answer>` for every other shape; and `<question> —
   Skipped` for a skipped question.
6. The participation keeps a link to the lead it produced, and the questionnaire keeps the list of
   the leads it produced with their count.

The **Lead Qualification** sample shipped by the same package builds a four-question questionnaire
of the purpose `survey`, paginated one question per page, with a numeric progress indicator, the
closing message `Thanks for answering!` and the title `Getting to know you`:

| Question | Shape | Mandatory | Options and which of them generate a lead |
|---|---|---|---|
| `Let's start with a basic question. What's your email address?` | single line text, validated as an address, saved as the participant's address | yes | — |
| `What is the size of your company?` | one-answer choice | yes | `1-10 employees`; `11-100 employees` generates a lead; `100+ employees` |
| `Which of the following best describes your main goal?` | one-answer choice | yes | `Improving efficiency` generates a lead; `Reducing costs`; `Expanding sales` generates a lead |
| `Who will make the final decision on this purchase?` | one-answer choice | yes | `Me` generates a lead; `My Manager/Executive` generates a lead; `A team/committee` |
