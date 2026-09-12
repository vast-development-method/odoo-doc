# Learning, Questionnaires and Recognition — Calculations

Every formula and algorithm of the domain, with the quantities named in words, the order of
evaluation, the rounding rule and at least one worked numeric example carried to the last decimal
the rule produces.

Unless a rule says otherwise:

- percentages are computed as a fraction multiplied by one hundred, and the rounding is stated per
  rule;
- reputation points are whole numbers and never rounded, because every input is a whole number;
- durations are decimal hours with four decimal places on a content item and two decimal places on a
  course;
- a division by zero never occurs: every rule below states the value produced when the denominator
  is zero.

| Rule | Subject |
|---|---|
| [`LSG-CALC-001`](#lsg-calc-001) | Maximum obtainable score of a questionnaire |
| [`LSG-CALC-002`](#lsg-calc-002) | Score, percentage and pass verdict of a participation |
| [`LSG-CALC-003`](#lsg-calc-003) | Score of a single answer |
| [`LSG-CALC-004`](#lsg-calc-004) | Questionnaire statistics: counts, average score, success ratio, average duration |
| [`LSG-CALC-005`](#lsg-calc-005) | Live-session speed bonus |
| [`LSG-CALC-006`](#lsg-calc-006) | Attempts left in a pool |
| [`LSG-CALC-007`](#lsg-calc-007) | Countdowns and their grace margins |
| [`LSG-CALC-008`](#lsg-calc-008) | Per-question result statistics |
| [`LSG-CALC-009`](#lsg-calc-009) | Global success rate of a result page |
| [`LSG-CALC-010`](#lsg-calc-010) | Session code generation |
| [`LSG-CALC-011`](#lsg-calc-011) | Course completion of an attendee |
| [`LSG-CALC-012`](#lsg-calc-012) | Quiz reward of an attempt |
| [`LSG-CALC-013`](#lsg-calc-013) | Duration of a section and of a course |
| [`LSG-CALC-014`](#lsg-calc-014) | Reading-time estimate of an uploaded portable document |
| [`LSG-CALC-015`](#lsg-calc-015) | View, like and vote counters |
| [`LSG-CALC-016`](#lsg-calc-016) | Reputation points earned on a course, after the fact |
| [`LSG-CALC-017`](#lsg-calc-017) | Progress towards the next rank |
| [`LSG-CALC-018`](#lsg-calc-018) | Live-session leaderboard |
| [`LSG-CALC-019`](#lsg-calc-019) | Certificate name sizing |
| [`LSG-CALC-020`](#lsg-calc-020) | Forum relevance ranking |
| [`LSG-CALC-021`](#lsg-calc-021) | Reputation movement of a vote |
| [`LSG-CALC-022`](#lsg-calc-022) | Reputation movement of closing, reopening and marking offensive |
| [`LSG-CALC-023`](#lsg-calc-023) | Related questions by tag overlap |
| [`LSG-CALC-024`](#lsg-calc-024) | Forum statistics |
| [`LSG-CALC-030`](#lsg-calc-030) | Goal completeness |
| [`LSG-CALC-031`](#lsg-calc-031) | Next report date of a challenge |
| [`LSG-CALC-032`](#lsg-calc-032) | Ranking of the best participants of a challenge |
| [`LSG-CALC-033`](#lsg-calc-033) | Reputation balance from the movement ledger, and its consolidation |
| [`LSG-CALC-034`](#lsg-calc-034) | Remaining badge grants of a sender |

---

<a id="lsg-calc-001"></a>

## LSG-CALC-001 — Maximum obtainable score of a questionnaire

The figure shown on the questionnaire form and checked before an invitation is sent.

```formula
maximum obtainable score (points)
    = sum over every question of the questionnaire of
        ( question score (points)
          when that score is non-zero,
          otherwise sum of the option scores (points) of that question
                    that are strictly greater than zero )
```

Evaluation order: for each question, the question score is examined first; only when it is zero are
the option scores summed. Sections contribute nothing because a section has no score. No rounding is
applied; the result carries the precision of the stored decimal scores.

**Worked example.** A questionnaire holds four questions:

| Question | Shape | Question score | Option scores |
|---|---|---|---|
| One | numerical value | 5.00 | — |
| Two | multiple choice, several answers | 0.00 | 2.50, 2.50, −1.00 |
| Three | multiple choice, one answer | 0.00 | 4.00, 0.00, −2.00 |
| Four | multiple lines text box | 0.00 | — |

```formula
question one   = 5.00
question two   = 2.50 + 2.50 = 5.00        (the −1.00 option is excluded)
question three = 4.00                       (the 0.00 and −2.00 options are excluded)
question four  = 0.00
maximum obtainable score = 5.00 + 5.00 + 4.00 + 0.00 = 14.00 points
```

---

<a id="lsg-calc-002"></a>

## LSG-CALC-002 — Score, percentage and pass verdict of a participation

The denominator is **not** the questionnaire maximum of LSG-CALC-001 but the maximum of the
participation's own question set, so a random draw or a conditional path produces a fair percentage.

```formula
total possible score (points)
    = sum over every question in the participation's predefined question set of
        ( highest strictly positive option score of that question
              when the shape is multiple choice with one answer,
          sum of the strictly positive option scores of that question
              when the shape is multiple choice with several answers,
          the question score
              when the question is marked scored and has any other shape,
          zero otherwise )

total score (points) = sum of the scores of every recorded answer of the participation

score percentage = total score ÷ total possible score × 100
```

Rounding and clamping, in this order:

1. When the total possible score is zero, both the percentage and the total are stored as zero and
   nothing further is computed.
2. Otherwise the total score is stored as it is, with two decimal places.
3. The percentage is rounded to two decimal places by the half-away-from-zero rule.
4. A rounded percentage that is not strictly greater than zero is stored as zero, so a participation
   that collected negative option scores never shows a negative percentage.

```formula
pass verdict = true when score percentage ≥ required score percentage of the questionnaire
```

**Worked example.** The questionnaire requires 80.00 percent. The participation's predefined set is
questions one, two and three of LSG-CALC-001, so:

```formula
total possible score = 5.00 + 5.00 + 4.00 = 14.00 points
```

The participant answers the numerical question correctly (5.00), ticks two of the three options of
question two, one worth 2.50 and one worth −1.00, and picks the 4.00 option of question three:

```formula
total score = 5.00 + 2.50 + (−1.00) + 4.00 = 10.50 points
score percentage = 10.50 ÷ 14.00 × 100 = 75.00
```

75.00 is lower than 80.00, so the pass verdict is false.

**Worked example of the clamp.** With a total possible score of 14.00 and answers worth −1.00 and
−2.00, the total score is −3.00 and the raw percentage is −21.428571…; rounded to two decimal places
that is −21.43, which is not strictly greater than zero, so the stored percentage is 0.00 and the
stored total is −3.00.

---

<a id="lsg-calc-003"></a>

## LSG-CALC-003 — Score of a single answer

Computed before the answer row is inserted, so it is available immediately.

1. When the answer is a skip, the score is zero and the answer is not correct.
2. When the question's shape is multiple choice with one answer or with several answers, and the
   answer points at an option, the score is the option's own score — which may be positive, zero or
   negative — and the correctness is the option's correctness marker.
3. When the question's shape is a date, a date and time, or a numerical value, the stored value is
   compared with the question's correct value. When they are exactly equal, the score is the
   question score and the answer is correct; otherwise the score is zero and the answer is not
   correct. A numerical value is compared as a number, a date as a date and an instant as an
   instant.
4. Every other shape earns zero and is never correct.
5. When the score is strictly positive and the live-session speed bonus applies, the score is
   adjusted by LSG-CALC-005.

---

<a id="lsg-calc-004"></a>

## LSG-CALC-004 — Questionnaire statistics

Over the participations of the questionnaire that are **not** test entries:

```formula
registered count   = number of those participations
attempt count      = number of those whose state is done
success count      = number of those whose pass verdict is true
average score (%)  = ( sum of the score percentages of those participations )
                     ÷ registered count
success ratio (%)  = success count ÷ registered count × 100
```

Both quotients produce zero when the registered count is zero. The average score is stored as a
decimal; the success ratio is stored as a whole number, truncated towards zero.

```formula
average duration (hours)
    = ( sum over the completed participations that carry both a start and an end instant of
        ( end instant − start instant ) in hours )
      ÷ number of those participations
```

The average duration is zero when no participation carries both instants.

**Worked example.** Six non-test participations exist. Four are completed; their percentages are
90.00, 75.00, 40.00 and 100.00, and two more are in progress with 0.00. Three of them pass.

```formula
registered count = 6
attempt count    = 4
success count    = 3
average score    = (90.00 + 75.00 + 40.00 + 100.00 + 0.00 + 0.00) ÷ 6 = 305.00 ÷ 6 = 50.8333…
success ratio    = 3 ÷ 6 × 100 = 50
```

The average score is stored as 50.83 when displayed with two decimal places; the success ratio is
stored as the whole number 50.

Durations: the four completed participations lasted 0.2500, 0.5000, 0.1250 and 0.6250 hours.

```formula
average duration = (0.2500 + 0.5000 + 0.1250 + 0.6250) ÷ 4 = 1.5000 ÷ 4 = 0.3750 hours
```

---

<a id="lsg-calc-005"></a>

## LSG-CALC-005 — Live-session speed bonus

Applied only when all of the following hold: the answer score is strictly positive, the
questionnaire rewards quick answers, the participation is a session attempt, and the current
question is time limited.

Quantities: the **maximum-score delay** is a fixed two seconds; the **question time limit** is the
question's own limit in seconds; the **seconds to answer** is the difference between the current
instant and the questionnaire's current-question start instant.

```formula
seconds to answer = current instant − current-question start instant, in seconds
question remaining time = question time limit − seconds to answer
```

1. When the question remaining time is strictly negative, **or** the answered question is not the
   question the session is currently presenting, the score is halved.
2. Otherwise, when the seconds to answer are greater than the maximum-score delay:

```formula
score proportion = ( question time limit − seconds to answer )
                   ÷ ( question time limit − maximum-score delay )
adjusted score   = ( raw score ÷ 2 ) × ( 1 + score proportion )
```

3. Otherwise — the answer arrived within the first two seconds — the score is left untouched.

No rounding is applied; the adjusted score keeps the full precision of the arithmetic and is stored
in the answer's score field. The floor of the adjustment is therefore half the raw score and the
ceiling is the whole raw score.

**Worked example.** The raw score is 4.0 points, the question time limit is 20 seconds and the
attendee answers 8 seconds after the question started.

```formula
question remaining time = 20 − 8 = 12 seconds        (not negative, so branch 2)
score proportion = (20 − 8) ÷ (20 − 2) = 12 ÷ 18 = 0.666666…
adjusted score   = (4.0 ÷ 2) × (1 + 0.666666…) = 2.0 × 1.666666… = 3.333333…
```

The stored score is 3.3333 when carried to four decimal places.

**Worked example of the two-second window.** The same question answered after 1.5 seconds: the
seconds to answer do not exceed the maximum-score delay, so the score stays 4.0.

**Worked example of the late answer.** The same question answered after 21 seconds: the remaining
time is −1, so the score is halved to 2.0.

---

<a id="lsg-calc-006"></a>

## LSG-CALC-006 — Attempts left in a pool

Attempts are counted only when the questionnaire limits attempts **and** either the access mode is
not `public` or signing in is required.

```formula
attempts used = number of participations where
                    the questionnaire is this questionnaire
                AND the attempt is not a test entry
                AND the state is done
                AND ( the contact is this contact, when a contact is known,
                      otherwise the electronic mail address is this address )
                AND ( the invite token is this invite token, when one is given )

attempts left = attempts limit − attempts used
```

The invite token is what separates one pool from another. On a non-public questionnaire with limited
attempts, each newly created participation receives a fresh invite token unless one is supplied; a
retry re-uses the token of the failed participation, so the retry consumes the same pool. On a public
questionnaire no token is generated, because a participation is created at every landing and the
pool must stay global.

**Worked example.** The limit is three. The contact has two completed non-test participations
carrying the invite token in question, plus one completed participation on the same questionnaire
carrying a different invite token, plus one test entry.

```formula
attempts used = 2        (the other pool and the test entry are excluded)
attempts left = 3 − 2 = 1
```

The submit operation re-checks the remaining attempts before storing anything, so a participant
cannot open several participations in advance and spend them after the allowance is exhausted.

---

<a id="lsg-calc-007"></a>

## LSG-CALC-007 — Countdowns and their grace margins

Two independent countdowns exist.

```formula
questionnaire countdown reached
    = questionnaire is time limited
      AND the attempt is not a session attempt
      AND a start instant exists
      AND current instant ≥ start instant + time limit (minutes)

question countdown reached
    = the attempt is a session attempt
      AND the questionnaire has a current-question start instant
      AND the current question is time limited
      AND current instant ≥ current-question start instant + question time limit (seconds)
```

A submit that arrives after a countdown is refused as unauthorised only when the current instant is
beyond the limit **plus** a grace margin: three seconds for a question countdown and ten seconds for
a questionnaire countdown. Inside the margin the submit is accepted, the validation messages are
ignored, the valid answers are stored and the participation is forced to `done`.

**Worked example.** A questionnaire is limited to 10 minutes and the participation started at
09:00:00. At 09:10:07 the participant submits. The countdown is reached because 09:10:07 is at or
after 09:10:00, but 09:10:07 is not beyond 09:10:10, so the submit is accepted, the answers are
stored and the participation is completed. A submit at 09:10:11 is refused.

---

<a id="lsg-calc-008"></a>

## LSG-CALC-008 — Per-question result statistics

For a question, the answer rows in the current population are split first. For a choice or matrix
question the answer rows are those that point at an option, those that are skipped with no answer
type, and — when comments count as answers — those of type `char_box`; the comment rows are those of
type `char_box`. For every other shape the answer rows are all the rows and there are no comment
rows.

```formula
answered count = number of distinct participations behind the non-skipped rows
skipped count  = number of distinct participations behind the skipped rows
```

Per shape, the summary block is:

| Shape | Summary |
|---|---|
| multiple choice, one answer | The options marked correct; the number of participations that chose a correct option; zero partially correct participations. |
| multiple choice, several answers | The options marked correct; the number of participations whose set of chosen correct options equals the full set of correct options; the number of participations whose set of chosen correct options is a strict, non-empty subset of it. |
| numerical value | The maximum, the minimum and the average of the values, the average rounded to two decimal places; the five most common values with their counts; the number of participations that answered correctly. |
| scale | The same, computed on the whole-number scale values. |
| date and date and time | The five most common values with their counts, and the number of participations that answered correctly. |
| every other shape | Nothing. |

```formula
average of a numerical question = sum of the values ÷ number of non-skipped rows
```

rounded to two decimal places, half away from zero, and zero when there is no row.

**Worked example.** A numerical question collected the values 3, 7, 7, 10 and 4.

```formula
maximum = 10
minimum = 3
average = (3 + 7 + 7 + 10 + 4) ÷ 5 = 31 ÷ 5 = 6.2 → 6.20
five most common values = 7 (twice), 3 (once), 10 (once), 4 (once)
```

The table and graph rows per shape are:

| Shape | Table rows | Graph series |
|---|---|---|
| multiple choice, one answer | One row per option, plus one row labelled `Other (see comments)` when comments count as answers; each row carries the option, the count and the caption `<count> Votes`. | One series holding the same entries. |
| multiple choice, several answers | The same rows. | One series named after the question title. |
| matrix | One row per matrix row, each carrying one cell per option with its count. | One series per option, each holding one entry per matrix row. |
| scale | One row per whole number from the scale minimum to the scale maximum inclusive, each with its count and the caption `<count> Votes`. | One series named after the question title. |
| every other shape | The answer rows themselves. | No graph. |

---

<a id="lsg-calc-009"></a>

## LSG-CALC-009 — Global success rate of a result page

Over the participations of the current population — by default every non-test participation of the
questionnaire whose state is not `new`:

```formula
count all       = number of participations in the population
count finished  = number of those whose state is done
count failed    = number of those whose pass verdict is false
count passed    = count all − count failed
global success rate (%) = count passed ÷ count all × 100
```

rounded to one decimal place, half away from zero, and zero when the population is empty.

**Worked example.** The population holds 7 participations, of which 5 pass.

```formula
count passed = 5
global success rate = 5 ÷ 7 × 100 = 71.428571… → 71.4
```

---

<a id="lsg-calc-010"></a>

## LSG-CALC-010 — Session code generation

A batch of codes is needed whenever questionnaires without a code are created.

1. Start with a length of four digits.
2. Draw the requested number of codes plus twenty extra, each a string of that many decimal digits.
3. Remove the codes already used by an existing questionnaire and the codes excluded by the caller —
   the codes drawn for the other questionnaires of the same batch.
4. When enough distinct codes remain, hand back exactly the requested number.
5. Otherwise increase the length by one digit and repeat from step 2, up to a length of ten digits.

The twenty extra codes per round exist to absorb collisions. A code is unique across all
questionnaires; the uniqueness constraint refuses a duplicate with `Session code should be unique`.

**Worked example.** Three questionnaires are created at once and one existing questionnaire already
uses the code `4821`. The first round draws twenty-three four-digit codes; `4821` and any duplicate
inside the round are removed; twenty-one distinct codes remain, so the first three are handed back
and the rest are discarded.

---

<a id="lsg-calc-011"></a>

## LSG-CALC-011 — Course completion of an attendee

```formula
completed contents = number of progress records of this attendee on this course where
                         the progress record is marked completed
                     AND the content item is published
                     AND the content item is active

total contents = number of published, active, non-section content items of the course

completion (%) = 100 × completed contents ÷ maximum( total contents, 1 )
```

The denominator uses one when the course holds no content, which makes the completion zero rather
than undefined. The percentage is rounded to the nearest whole number, half away from zero, and
stored as a whole number between zero and one hundred; the database refuses any value outside that
range with `The completion of a channel is a percentage and should be between 0% and 100.`

The status that follows is `completed` at one hundred, `joined` at zero, and `ongoing` in between.
Enrolments already in status `completed` and enrolments still in status `invited` are skipped
entirely by the recomputation.

**Worked example.** A course holds 7 published active content items and the attendee has completed 3
of them.

```formula
completion = 100 × 3 ÷ 7 = 42.857142… → 43
```

The status becomes `ongoing`.

**Worked example of the boundary.** The same course after the attendee completes a fourth, fifth,
sixth and seventh item: `100 × 7 ÷ 7 = 100`, the status becomes `completed`, the course-completion
reward is granted once and the completion message is sent. An eighth content item is then published:
`100 × 7 ÷ 8 = 87.5 → 88`. The enrolment is already `completed`, so its status does not change; an
enrolment that stood at one hundred percent without having been written as completed has the
course-completion reward taken back.

---

<a id="lsg-calc-012"></a>

## LSG-CALC-012 — Quiz reward of an attempt

Four rewards are configured per content item: the first-attempt reward (ten by default), the
second (seven), the third (five) and the fourth-and-later reward (two).

```formula
reward index = minimum( quiz attempts count after the submission, 4 )
points gained = the reward configured at that index
```

Points are granted only when the progress record exists, its completion marker is actually changing,
its attempt count is not zero and the content item carries at least one question. Granting uses the
reason `Quiz Completed`. Taking the points back multiplies the same amount by minus one and uses the
reason `Quiz Set Uncompleted`.

The panel shown before a submission displays: the **maximum**, which is always the first-attempt
reward; the **gain of the next attempt**, which is the reward at the index equal to the current
attempt count, clamped to the last reward; the **points already won**, which is the reward at the
index one below the attempt count when the quiz is done; and the attempt count itself.

**Worked example.** An attendee fails the quiz twice and passes on the third submission. The attempt
counter reads 3 after the third submission.

```formula
reward index  = minimum(3, 4) = 3
points gained = third-attempt reward = 5
```

Five reputation points are granted with the reason `Quiz Completed`. If the attendee later marks the
content uncompleted, the counter still reads 3, so five points are taken back with the reason `Quiz
Set Uncompleted`.

**Worked example beyond the ladder.** On a sixth submission the counter reads 6 and the index is
clamped to 4, so two points are granted.

---

<a id="lsg-calc-013"></a>

## LSG-CALC-013 — Duration of a section and of a course

```formula
duration of a section (hours)
    = sum of the durations of the published content items that belong to that section

duration of a course (hours)
    = sum of the durations of the published, active, non-section content items of the course
```

The section computation is recursive, because a section's duration depends on the durations of the
items it contains. A content duration is stored with four decimal places; a course duration with
two.

**Worked example.** A section holds three published items lasting 0.2500, 0.7500 and 1.3333 hours,
and the course holds those three plus one uncategorised item lasting 0.5000 hours.

```formula
section duration = 0.2500 + 0.7500 + 1.3333 = 2.3333 hours
course duration  = 2.3333 + 0.5000 = 2.8333 → 2.83 hours
```

---

<a id="lsg-calc-014"></a>

## LSG-CALC-014 — Reading-time estimate of an uploaded portable document

Five minutes are assumed per page.

```formula
estimated duration (hours) = 5 × page count ÷ 60
```

The estimate is applied when the author uploads the file and at creation when no duration was
supplied. When the file cannot be read, no estimate is produced and the duration is left as it is.
The result is stored with four decimal places.

**Worked example.** A document of 23 pages:

```formula
estimated duration = 5 × 23 ÷ 60 = 115 ÷ 60 = 1.916666… → 1.9167 hours
```

For an external video or document the duration comes from the retrieved metadata instead: the
declared running time for a video, and the same five-minutes-per-page estimate applied to the
downloaded content for a portable document.

---

<a id="lsg-calc-015"></a>

## LSG-CALC-015 — View, like and vote counters

```formula
likes of a content item    = number of progress records on it whose vote is +1
dislikes of a content item = number of progress records on it whose vote is −1
signed-in views            = number of progress records on the content item
total views                = signed-in views + anonymous views
embed count                = sum of the view counts of the embed counters of the content item

course visits = sum of the total views of the published, active, non-section contents
course votes  = sum of the likes of those contents − sum of the dislikes of those contents
```

The anonymous view counter is incremented when an anonymous or non-enrolled visitor opens a content
page they have not opened before in the same browser session; the identifiers already counted are
held in the browser session so a refresh does not inflate the figure. Both the anonymous counter and
the total counter are incremented at once, without taking a lock. An enrolled attendee opening a
content item does not touch the anonymous counter; the mark-viewed operation creates or refreshes
their progress record instead.

**Worked example.** A content item has 12 progress records, of which 7 carry a vote of plus one and
2 carry minus one, and 40 anonymous opens have been counted.

```formula
likes = 7
dislikes = 2
signed-in views = 12
total views = 12 + 40 = 52
```

For a course holding only that item: visits 52 and votes 7 − 2 = 5.

---

<a id="lsg-calc-016"></a>

## LSG-CALC-016 — Reputation points earned on a course, after the fact

An officer may ask how many points a set of attendees earned on a set of courses.

```formula
points earned
    = sum over the completed progress records of those attendees on those courses
        whose quiz attempt count is not zero of
            the quiz reward at index minimum( attempt count, 4 )
    + sum over the completed enrolments of those attendees on those courses of
            the course-completion reward of that course
```

The figure is only accurate when the reward settings were not changed after the fact, because it
reads the settings as they stand today rather than the amounts actually granted; the movement ledger
holds the amounts that were really granted.

**Worked example.** One attendee completed two quizzes on a course — one on the first attempt, one
on the third — and completed the course itself. The rewards are the shipped defaults.

```formula
first quiz  = reward at index 1 = 10
second quiz = reward at index 3 = 5
course      = 10
points earned = 10 + 5 + 10 = 25
```

---

<a id="lsg-calc-017"></a>

## LSG-CALC-017 — Progress towards the next rank

```formula
progress (%) = 100 × ( balance − minimum of the current rank )
               ÷ ( minimum of the next rank − minimum of the current rank )
```

The result is forced to one hundred when there is no next rank or when the two minimums are equal. A
user who holds no rank yet is treated as having a current minimum of zero, so the progress is
measured against the lowest rank.

**Worked example.** The shipped ranks are Newbie at 1, Student at 100, Bachelor at 500, Master at
2000 and Doctor at 10000. A user holds 350 points, so the current rank is Student and the next is
Bachelor.

```formula
progress = 100 × (350 − 100) ÷ (500 − 100) = 100 × 250 ÷ 400 = 62.5
```

**Worked example at the top.** A user with 12000 points is at Doctor and has no next rank, so the
progress is one hundred.

---

<a id="lsg-calc-018"></a>

## LSG-CALC-018 — Live-session leaderboard

1. The fifteen participations of this session with the highest total score are read, ordered by
   total score descending, together with their nicknames.
2. When the session is in progress and at least one option of the current question carries a
   non-zero score, the score each of those participations earned **on the current question** is
   computed.
3. Each entry then carries five figures:

```formula
updated score        = total score including the current question
score before         = total score − score earned on the current question
leaderboard position = the rank in the list read at step 1, counted from zero
maximum question score = sum of the strictly positive option scores of the current question,
                         or 1 when that sum is zero
question score       = the points earned on the current question
```

4. The list is re-sorted by the score before the current question, descending.

The client uses the two totals to animate each attendee from the position they held before the
current question to the position they hold after it.

**Worked example.** Three attendees hold 30, 24 and 24 points including the current question, on
which they earned 10, 0 and 6 points respectively. The current question offers options worth 10 and
0.

| Attendee | Updated score | Score before | Position from step 1 | Question score |
|---|---|---|---|---|
| One | 30 | 20 | 0 | 10 |
| Two | 24 | 24 | 1 | 0 |
| Three | 24 | 18 | 2 | 6 |

The maximum question score is 10. Re-sorted by the score before the current question, the order is
Two (24), One (20), Three (18); the animation therefore moves One from second place up to first and
Two from first down to second.

---

<a id="lsg-calc-019"></a>

## LSG-CALC-019 — Certificate name sizing

The certified name is printed at a size that shrinks as the name gets longer, so a long name never
overflows the page.

| Visual shape | Name length in characters | Size band |
|---|---|---|
| `classic` | 36 or more | small |
| `classic` | 35 or fewer | large |
| `modern` | 46 or more | small |
| `modern` | 36 to 45 | medium |
| `modern` | 21 to 35 | large |
| `modern` | 20 or fewer | extra large |

**Worked example.** A `modern_gold` certificate for a contact named with 28 characters uses the
large band; the same certificate for a 47-character name uses the small band.

---

<a id="lsg-calc-020"></a>

## LSG-CALC-020 — Forum relevance ranking

The ranking score stored on every post and used by the relevance ordering.

```formula
days = whole days between the post creation instant and the current instant

relevance = sign of the vote total
            × ( absolute value of ( vote total − 1 ) raised to the power of
                the first relevance parameter )
            ÷ ( ( days + 2 ) raised to the power of the second relevance parameter )
```

The sign function yields plus one for a positive or zero vote total and minus one for a negative
one. A post with no creation instant scores zero. The two parameters default to 0.8 and 1.8. No
rounding is applied; the score is a stored decimal.

**Worked example.** A post has a vote total of 10 and was created 5 days ago, on a forum with the
default parameters.

```formula
sign of 10 = +1
| 10 − 1 | = 9 ;  9 to the power 0.8 = 5.799546…
( 5 + 2 ) = 7 ;   7 to the power 1.8 = 32.181578…
relevance = 1 × 5.799546… ÷ 32.181578… = 0.180211…
```

The stored relevance is 0.180211 when carried to six decimal places.

**Worked example of a fresh post.** The same vote total on a post created today: days is zero, the
denominator is 2 to the power 1.8, which is 3.482202…, and the relevance is 5.799546… ÷ 3.482202… =
1.665466…, so a fresh post outranks an older one with the same votes.

---

<a id="lsg-calc-021"></a>

## LSG-CALC-021 — Reputation movement of a vote

The movement is the difference between the point value of the new vote and the point value of the
old vote, read from the forum's question fields for a question and from its answer fields for an
answer.

```formula
point value of a vote:  −1 → the downvote grant
                         0 → zero
                        +1 → the upvote grant

movement = point value of the new vote − point value of the old vote
```

The recipient is the post's author. The reason recorded on the movement is built from the direction:
`Question upvoted`, `Question downvoted`, `Question no more upvoted`, `Question no more downvoted`
and `Question no changes` for a question, and the same five phrases with `Answer` in place of
`Question` for an answer.

**Worked sequence on an answer** with the shipped defaults, an upvote grant of 10 and a downvote
grant of −2:

| Step | Old vote | New vote | Movement | Reason recorded | Author's balance after, from 100 |
|---|---|---|---|---|---|
| 1 | none, treated as `0` | `1` | 10 − 0 = +10 | `Answer upvoted` | 110 |
| 2 | `1` | `0` | 0 − 10 = −10 | `Answer no more upvoted` | 100 |
| 3 | `0` | `-1` | −2 − 0 = −2 | `Answer downvoted` | 98 |
| 4 | `-1` | `1` | 10 − (−2) = +12 | `Answer upvoted` | 110 |

The vote total of the post, shown beside it, is the sum of the vote values and therefore counts
upvotes minus downvotes, with cancelled votes contributing zero.

**Which direction is checked for permission.** For a creation, the direction is upvote when the
stored value is `1`. For a write, the direction is upvote when the new value is `1`; when the new
value is `0`, the direction is upvote exactly when the old value was `-1`. That is why cancelling
one's own downvote is treated as an upvote and is always allowed.

---

<a id="lsg-calc-022"></a>

## LSG-CALC-022 — Reputation movement of closing, reopening and marking offensive

```formula
base amount = the flagged grant of the forum, as stored (−100 by default)

spam multiplier = 10 when the closing reason is the spam reason
                  AND the post is the author's only question on that forum,
                  otherwise 1

movement on closing   = base amount × spam multiplier
movement on reopening = base amount × spam multiplier × (−1)
movement on marking offensive = base amount
```

Closing applies a movement only when the reason is the offensive reason or the spam reason; every
other reason moves nothing.

**Worked example.** A member with 250 points has asked exactly one question on the forum, and a
moderator closes it as spam.

```formula
movement = −100 × 10 = −1000
```

The balance becomes −750 and the reason recorded is `Post is closed and marked as spam`. Because the
balance is no longer strictly positive, every post of that author becomes invisible to ordinary
readers under the view rule.

Reopening the same question applies −1000 × −1 = +1000 with the reason `Reopen a banned question`,
restoring the balance to 250.

**Worked example without the multiplier.** A member with 250 points who has asked four questions on
the forum has one closed as spam: the movement is −100 × 1 = −100 and the balance becomes 150.

---

<a id="lsg-calc-023"></a>

## LSG-CALC-023 — Related questions by tag overlap

Up to five related questions are proposed under a question, ranked by the overlap of their tag sets.

```formula
overlap = number of tags common to the two questions
          ÷ number of tags carried by either of the two questions
```

The result runs from zero to one, one meaning identical tag sets. Candidates are ordered by overlap
descending and then by last-activity instant descending. A question that carries no tag proposes
nothing.

**Worked example.** The current question carries the tags A, B and C. A candidate carries B, C and
D.

```formula
common tags = B and C = 2
tags on either side = A, B, C, D = 4
overlap = 2 ÷ 4 = 0.5
```

A second candidate carrying A, B and C scores 3 ÷ 3 = 1.0 and is proposed first.

---

<a id="lsg-calc-024"></a>

## LSG-CALC-024 — Forum statistics

Over the questions of the forum — posts with no parent — whose state is `active` or `close`:

```formula
post count      = number of those questions
view count      = sum of the view counters of those questions
answer count    = sum of the answer counts of those questions
favourite count = number of those questions that have at least one bookmark
```

and, over every post of the forum whatever its parent:

```formula
posts waiting validation = number of posts whose state is pending
flagged posts            = number of posts whose state is flagged
```

The last post of a forum is the question with the highest identifier among its active questions. The
usage count of a tag is the number of active, non-archived posts carrying it.

**Worked example.** A forum holds 12 active questions and 3 closed ones, 40 answers spread over
them, view counters summing to 5 300, and 4 questions bookmarked by at least one member. Two
questions wait for validation and one post is flagged.

```formula
post count = 12 + 3 = 15
view count = 5300
answer count = 40
favourite count = 4
posts waiting validation = 2
flagged posts = 1
```

---

<a id="lsg-calc-030"></a>

## LSG-CALC-030 — Goal completeness

```formula
when the condition is "the higher the better":
    completeness (%) = 100 when the current value ≥ the target value,
                       otherwise 100 × current value ÷ target value,
                       and zero when the target value is zero

when the condition is "the lower the better":
    completeness (%) = 100 when the current value < the target value,
                       otherwise 0
```

In the "higher is better" branch the quotient is rounded to two decimal places, half away from zero.
The "lower is better" branch produces only zero or one hundred, because no meaningful percentage
exists below a ceiling.

**Worked example.** A goal asks for 12 new leads and the user has 7.

```formula
completeness = 100 × 7 ÷ 12 = 58.333333… → 58.33
```

**Worked example of a ceiling goal.** A goal asks for a lead qualification delay lower than 3 days
and the user's delay is 2.4 days: the current value is strictly below the target, so the completeness
is 100. At 3.0 days it is 0.

---

<a id="lsg-calc-031"></a>

## LSG-CALC-031 — Next report date of a challenge

```formula
next report date = last report date + offset
```

where the offset is one day for a daily frequency, seven days for a weekly one, one calendar month
for a monthly one and one calendar year for a yearly one. When the frequency is `never` or
`onchange`, the next report date is empty and no scheduled report goes out.

**Worked example.** The last report went out on 31 January and the frequency is monthly. One
calendar month later is 28 February in a common year and 29 February in a leap year; the calendar
addition clamps the day to the last day of the target month.

The daily job sends a report when today is at or after the next report date and the last report was
not sent today. When no report is due, the job still looks for goals of the challenge that started
on or after the last report date and ended on or before it — goals that closed since the last report
— and sends a final report restricted to them.

---

<a id="lsg-calc-032"></a>

## LSG-CALC-032 — Ranking of the best participants of a challenge

For each participant, over the goals of the challenge for the current period:

```formula
all reached = true when every one of those goals is in state reached

total completeness
    = sum over those goals of
        ( 100 × current value ÷ target value        when the condition is "higher is better"
                                                     and the target value is not zero,
          0                                          when the condition is "higher is better"
                                                     and the target value is zero,
          100                                        when the condition is "lower is better"
                                                     and the goal is reached,
          0                                          otherwise )
```

The "higher is better" contribution is deliberately **not** capped at one hundred, so a participant
who exceeded a target outranks one who merely met it.

Participants are sorted by the reached marker first and by the total completeness second, both
descending. When the challenge does not reward participants who failed, only the leading run of
participants who reached every goal is kept. The list is then padded with empty places and truncated
to the requested number, so there is never an empty place between two named participants.

**Worked example.** A challenge has two lines, both "higher is better", with targets 10 and 4.

| Participant | Goal one current | Goal two current | All reached | Total completeness |
|---|---|---|---|---|
| Ada | 14 | 5 | yes | 140.00 + 125.00 = 265.00 |
| Ben | 10 | 4 | yes | 100.00 + 100.00 = 200.00 |
| Cleo | 12 | 3 | no | 120.00 + 75.00 = 195.00 |

The order is Ada, Ben, Cleo. When the challenge does not reward failures, only Ada and Ben are kept
and the third place is empty.

---

<a id="lsg-calc-033"></a>

## LSG-CALC-033 — Reputation balance from the movement ledger, and its consolidation

The balance is not an independent number; it is read from the ledger.

```formula
balance of a user = the new value of that user's most recent movement,
                    ordered by tracking instant descending, then identifier descending,
                    and zero when the user has no movement
```

Writing a balance directly creates a movement whose old value is the current balance and whose new
value is the written figure. Creating a user with a starting balance creates a movement with an old
value of zero and the reason `User Creation`. Every reason recorded on a movement is the caller's
reason followed by the display name and the identifier of the source record in parentheses.

**Monthly consolidation.** The job runs on the first day of each month and consolidates the
movements of the month that ended two months earlier.

1. The range runs from the first instant of that month to the last instant of that month.
2. For each user with at least one unconsolidated movement in the range, the **oldest** old value
   and the **newest** new value are taken.
3. One replacement movement is written per user, carrying those two values, dated at the instant of
   the oldest movement, marked consolidated, with the acting user as its source and the reason
   `Consolidation from <start date> to <end date>`.
4. Every unconsolidated movement in the range is deleted. The sources and reasons of the deleted
   movements are lost.
5. The whole operation runs with the balance recomputation suppressed, so no balance changes.

**Worked example.** In the consolidated month a user's movements were: from 100 to 110, from 110 to
108, from 108 to 123. The replacement carries an old value of 100 and a new value of 123, a gain of
23, and the three original movements are deleted. The user's balance is unchanged at 123.

---

<a id="lsg-calc-034"></a>

## LSG-CALC-034 — Remaining badge grants of a sender

```formula
remaining grants = 0   when the reader may not grant this badge at all
                 = −1  when the badge has no monthly limit
                 = monthly limit − grants of this badge created by the reader
                        since the first day of the current month
```

The value minus one means "no limit" and is not displayed as a number.

**Worked example.** A badge limited to 3 grants per person per month; the reader has already granted
it twice this month.

```formula
remaining grants = 3 − 2 = 1
```

A third grant succeeds; a fourth is refused with `You have already sent this badge too many time
this month.`
