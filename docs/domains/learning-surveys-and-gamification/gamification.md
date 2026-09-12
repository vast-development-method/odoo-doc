# Recognition: goals, challenges, badges, ranks and the reputation ledger

Everything a replacement needs in order to reproduce the recognition machinery: what a goal
definition measures and how, how a goal is generated and updated, how a challenge assigns goals,
reports on them and distributes rewards, how badges are granted and capped, how ranks are unlocked,
and how every reputation point that moves is recorded and later consolidated.

## 1. The four moving parts

| Part | What it holds |
|---|---|
| Goal Definition | What to measure, over which entity, with which filter, in which computation mode, and whether higher or lower is better. It holds no user and no target. |
| Challenge Line | One definition plus the value to reach. A challenge holds at least one line. |
| Challenge | The population, the periodicity, the rewards, the report schedule and the display mode. |
| Goal | One user's measurement against one definition for one period, with its own state. |

A goal may also exist without a challenge: created by hand, it starts in the draft state and is
measured only when somebody starts it.

<a id="goal-update"></a>

## 2. Measuring a goal

Goals are measured in batches, grouped by definition, because one definition can answer for many
users in one query. The four computation modes behave as follows.

### 2.1 Recorded manually

Nothing is measured. Instead the reminder rule runs: when the goal carries a reminder delay and a
last-update date, and today is at least that many days after the last update, an individual reminder
message is sent to the goal's user, rendered from the shipped goal-reminder template, and the goal
is marked as needing an update. The reminder is sent once, while the goal is in progress.

The user updates such a goal through a small dialogue that writes the current value; it is offered
only when the definition carries no window action of its own, and its title is `Update <definition
name>`.

### 2.2 Number of records

The definition names an entity and a filter. In **batch mode** one grouped query answers for every
user at once: the filter is extended with the condition that the distinctive field is one of the
values produced by the batch expression for the goals being measured, and, when a date field is
named, with the period bounds of each goal; the result is grouped by the distinctive field and
counted; each goal then takes the count whose grouping value equals its own batch expression. Out of
batch mode the filter is evaluated once per goal, with the acting user replaced by the goal's user,
the period bounds appended when a date field is named, and the matching records counted.

### 2.3 Sum on a field

The same as counting, except that the named numeric field is summed instead of the records being
counted. Out of batch mode the sum is attempted only when the named field is a whole number, a
decimal or a monetary amount; otherwise the mode falls back to counting.

### 2.4 A computation script

The definition carries a script that is evaluated once per goal, with the goal itself available to
it. The value the script produces is written as the goal's current value when it is a number;
otherwise the measurement is skipped and the goal keeps its previous value.

### 2.5 Writing the result

For each goal the new value is compared with the stored one; when they are equal nothing is written
at all. Otherwise:

1. The current value is written.
2. When the condition is "the higher the better" and the new value is at least the target, or the
   condition is "the lower the better" and the new value is at most the target, the state becomes
   `reached`. The goal is **not** closed, because a later measurement may take it back out of that
   state.
3. Otherwise, when the goal carries an end date and today is after it, the state becomes `failed`
   and the goal is closed; a closed goal is skipped by every later measurement.

Every write stamps the goal's last-update date with today. When the current value changes on a goal
whose challenge reports on change, an individual report is sent at once to that goal's user.

The completeness shown beside a goal is [`LSG-CALC-030`](calculations.md#lsg-calc-030). The decoration
colour is two when the goal failed after its end date, five when it was reached after its end date,
and zero otherwise.

<a id="challenge-cycle"></a>

## 3. The challenge cycle

### 3.1 The population

A challenge names its participants directly and may also carry a stored filter. Whenever that filter
is written — at creation or later — the users it selects are added to the participant list. Each
update of the challenge recomputes the participants from the filter and replaces the list when it
differs, so people who no longer match are removed and new matches are added. The participant
counter counts only the **active** users among the participants.

The filter proposed on a new challenge selects the active internal users.

### 3.2 Period dates

The periodicity turns today's date into a start date and an end date:

| Periodicity | Start date | End date |
|---|---|---|
| `daily` | Today. | Today. |
| `weekly` | The most recent Monday at or before today. | That Monday plus seven days. |
| `monthly` | The first day of the current month. | The last day of the current month. |
| `yearly` | The first of January of the current year. | The thirty-first of December of the current year. |
| `once` | The challenge's own start date, which may be empty. | The challenge's own end date, which may be empty. |

A challenge with the periodicity `once` and no dates therefore produces goals with no period, which
are always active and never fail.

### 3.3 Generating goals

For each line of the challenge and each participant who has no goal for that line and that period, a
goal is created carrying the definition, the line, the target, the state `inprogress`, the period
dates when they exist, and the reminder delay of the challenge when it is not zero. The initial
current value is deliberately set beyond the target so the first measurement always produces a
change: the target minus one, floored at zero, when higher is better, and the target plus one,
floored at zero, when lower is better.

Goals belonging to users who used to match the challenge but no longer do are deleted first.

### 3.4 The daily job

The job is [`workflows.md`](workflows.md#workflow-19). Two details are worth restating: the job only
re-measures the goals of users who have interacted with the client since the goal was last written
and who hold a session that is still valid, which keeps a large installation from re-measuring
dormant accounts; and the job commits between challenges when it is asked to, so a late failure does
not undo the earlier challenges.

### 3.5 Reports

| Display mode | What is sent |
|---|---|
| `personal` | One message per participant, rendered from the report template in that participant's language and context, holding the participant's own lines. A participant whose goals are not all reached receives nothing, because the serialisation returns an empty list as soon as one of their goals is not in the reached state. A copy goes to the configured discussion channel when one is set. |
| `ranking` | One message posted on the challenge and addressed to every participant, holding, per line, the ranked goals of every participant. A copy goes to the configured discussion channel when one is set. |

The ranking serialisation sorts the goals of a line by completeness descending and then by current
value — descending when higher is better, ascending when lower is better — records each
participant's rank counted from zero, marks the reading participant's own goal, and pads the list
with empty places so at least three places are always shown.

The next report date is [`LSG-CALC-031`](calculations.md#lsg-calc-031). Sending a report stamps the
challenge's last report date with today.

### 3.6 Suggested challenges

A challenge may name users it is only **suggested** to. Such a user may accept it, which posts
`<user name> has joined the challenge` on the challenge, removes them from the suggested list, adds
them to the participants and generates their goals; or discard it, which posts `<user name> has
refused the challenge` and removes them from the suggested list without adding them.

### 3.7 Rewards

The reward check and its messages are section 6.3 of
[`state-machines.md`](state-machines.md#machine-6); the ranking of the best participants is
[`LSG-CALC-032`](calculations.md#lsg-calc-032). Two properties matter for a rebuild:

- **Real-time reward.** When it is on, the badge for every succeeding participant is granted as soon
  as a participant has reached every goal, and a participant who already holds a grant of that badge
  for that challenge is skipped, so the badge is received once. The first, second and third badges
  are still granted only at the end.
- **Rewarding the best even when nobody succeeded.** When that marker is off, only the leading run of
  participants who reached every goal is considered for the top three places, so a challenge nobody
  completed hands out no special badge.

## 4. Badges

### 4.1 Granting rules

A badge names who may grant it: `everyone`, a named list of users, the holders of named badges, or
nobody — the last value reserving the badge for challenges. The four refusals are
[`LSG-116`](business-rules.md#lsg-116) to [`LSG-119`](business-rules.md#lsg-119), and the remaining
allowance of a sender is [`LSG-CALC-034`](calculations.md#lsg-calc-034). An administrator passes
every check.

### 4.2 Counters

A badge carries four public counters — the total grants, the distinct holders, the list of distinct
holders and the grants made this month — and four reader-dependent ones: the grants the reader has
received in total and this month, the grants the reader has **made** this month, and the remaining
allowance. The month always starts on its first day.

A user carries three counters, one per badge level, counting the badges they hold at each level.

### 4.3 Notification

Creating a grant notifies the beneficiary's contact with the shipped badge message, whose subject is
`🎉 You've earned the <badge name> badge!`, rendered from the badge-received template and wrapped in
the standard notification layout. The notification deliberately carries no button to open the
record, because the grant record is of no interest to the beneficiary.

### 4.4 Badges awarded by a goal definition

A badge may also name the goal definitions that unlock it, so that the holders of those goals
receive it automatically. This is the mechanism the shipped forum badges use through their
challenges.

## 5. Ranks

A rank is a name, an illustration, a description, a motivational phrase and a minimum balance. The
minimum must be greater than zero ([`LSG-127`](business-rules.md#lsg-127)).

A user's rank is the highest rank whose minimum their balance reaches; the next rank is the one
above. A user below the lowest rank holds no rank and their next rank is the lowest one. The
progress between the two is [`LSG-CALC-017`](calculations.md#lsg-calc-017).

Ranks are recomputed in two ways. When few users are affected — fewer than three times the number of
ranks — each user is examined in turn. Otherwise the population is walked rank by rank, from the
highest minimum down, writing the rank and the next rank on every user who reaches that minimum and
does not already carry the right pair; a final pass gives the users below the lowest minimum no rank
and the lowest rank as their next one.

Creating a rank recomputes every user whose balance reaches the lowest of the new minimums. Writing a
minimum recomputes the whole population above the lower of the old and the new minimum when the
ordering of the ranks changed, and only the band between the two minimums when it did not.

A user who moves to a new rank receives the shipped rank message, unless the change happens while a
package is being installed. The message offers the destinations contributed by the installed
packages.

## 6. The reputation ledger

### 6.1 Every point is recorded

The balance is not an independent number; it is read from the ledger by
[`LSG-CALC-033`](calculations.md#lsg-calc-033). Every movement carries the old value, the new value,
the instant, the source record and a reason. The reason stored is the caller's reason followed by
the display name and the identifier of the source record in parentheses, so a reader can trace a
movement back to the post, the course, the content item or the user that caused it.

### 6.2 The sources

| Source kind | Added by | Movements it explains |
|---|---|---|
| User | the base package | Manual grants, the balance written at user creation, the three points granted for validating an address. |
| Course | the learning package | `Course Finished`, `Course Set Uncompleted`, `Course Ranked`. |
| Course Content | the learning package | `Quiz Completed`, `Quiz Set Uncompleted`. |
| Forum Post | the forum package | The vote movements, the accept movements, the closing and reopening movements, the offensive movement, the new-question grants, and the two movements caused by deleting an accepted answer. |

### 6.3 The reasons

| Reason recorded | When |
|---|---|
| `Add Manually` | A balance is written directly with no reason supplied. |
| `User Creation` | A user is created with a non-zero starting balance. |
| `Ask a new question` | A question is created active by an author at or above the publish threshold. |
| `Ask a question` | A pending question is validated by a moderator. |
| `Question upvoted`, `Question downvoted`, `Question no more upvoted`, `Question no more downvoted`, `Question no changes` | A vote on a question is cast, changed or cancelled. |
| `Answer upvoted`, `Answer downvoted`, `Answer no more upvoted`, `Answer no more downvoted`, `Answer no changes` | The same on an answer. |
| `User answer accepted` | An answer is accepted; the amount goes to its author. |
| `Accepted answer removed` | That acceptance is withdrawn. |
| `Validate an answer` | An answer is accepted; the amount goes to the acting user. |
| `Remove validated answer` | That acceptance is withdrawn. |
| `The accepted answer is deleted` | An accepted answer is deleted; the amount is removed from its author. |
| `Delete the accepted answer` | The same deletion; the amount is removed from the acting user. |
| `Post is closed and marked as offensive content` | A question is closed with the offensive reason. |
| `Post is closed and marked as spam` | A question is closed with the spam reason. |
| `Reopen a banned question` | Such a question is reopened. |
| `Downvote for posting offensive contents` | A post is marked offensive. |
| `Quiz Completed`, `Quiz Set Uncompleted` | A content quiz is passed or un-passed. |
| `Course Finished`, `Course Set Uncompleted` | A course is completed or falls back below one hundred percent. |
| `Course Ranked` | A review is posted on a course. |
| `Consolidation from <start date> to <end date>` | The monthly consolidation replaces a month of movements. |

### 6.4 Consolidation

The monthly job and its worked example are
[`LSG-CALC-033`](calculations.md#lsg-calc-033). The consolidation is lossy by design: the source and
the reason of every replaced movement disappear, only the net change of the month survives, and no
balance changes because the recomputation is suppressed while it runs.

### 6.5 Who may write the ledger

Only the group `base.group_system` may create, change or delete a movement directly. Every other
movement is written by the domain on the user's behalf, with elevated rights, at the moment the
event happens; [`LSG-128`](business-rules.md#lsg-128).

## 7. Rankings of the public member list

Two helpers serve the public member list and the profile pages:

1. **By gain over a period.** Users satisfying a filter are ordered by the sum, over the movements
   inside the chosen date range, of the difference between the new value and the old value,
   descending, and by identifier descending as a tie-break. The range bounds are optional: an open
   lower bound counts from the beginning of the ledger and an open upper bound counts to the end.
2. **By absolute position.** For a subset of users, the position each of them occupies in the whole
   ranking is returned, computed over the whole population satisfying the filter, so that searching
   the list for a name does not renumber the results from one. The position may be computed either
   over the gain across a range or over the total balance.

## 8. What the domain ships

The shipped goal definitions, challenges, badges and ranks are catalogued in
[`configuration.md`](configuration.md#shipped-recognition-records). In summary: four general goal
definitions and two general challenges that help a new installation get configured; five badges with
their challenges for the learning platform; twenty-nine badges, twenty-seven goal definitions and
twenty-eight challenges for the forum; ten goal definitions and two challenges for the selling
pipeline, added by the selling-recognition package; and five ranks.
