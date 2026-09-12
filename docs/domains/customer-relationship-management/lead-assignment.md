# Lead assignment

Rule-based assignment distributes incoming Leads that nobody has claimed. It runs in two phases: first every unclaimed Lead is allocated to a Sales Team by a weighted random draw proportional to the capacity of the teams, and deduplicated on the way; then, inside each team, the Leads that have a team but no salesperson are distributed to the members according to their daily quota and their filters, and converted into opportunities. This file specifies the configuration, both phases, the arithmetic, the notifications, the manual variant and four worked examples.

The arithmetic used here (daily quota, team capacity, weighted draw, round robin) is also stated in [calculations.md](calculations.md) sections 8.1 to 8.6; this file gives the complete procedure.

## 1. Configuration

### 1.1 The master switch

| Setting | Type | Effect |
|---|---|---|
| Rule-Based Assignment | boolean, stored in a system parameter | When false, no assignment is offered at all: the team screen shows no assignment section, no scheduled run happens, and Leads created without a salesperson fall back on the team leader rule (`LEAD-107`). When true, the assignment section appears on every team and on every membership. |

### 1.2 The mode

| Setting | Values | Effect |
|---|---|---|
| Auto Assignment Action | `manual` "Manually", `auto` "Repeatedly" | `manual` means the assignment only runs when a user presses a button. `auto` activates the scheduled action. |
| Auto Assignment Interval Unit | `minutes`, `hours`, `days`, `weeks` | The unit of the repetition interval. |
| Repeat every | integer | The number of units between two runs. |
| Auto Assignment Next Execution Date | datetime | The instant of the next scheduled run. |

Reading rules. When the master switch is off, or when the scheduled action does not exist, the four settings read as `manual`, `days`, `1` and empty. When the master switch is on, they read the current state of the scheduled action: the mode is `auto` when the scheduled action is active and `manual` otherwise, and the three other values are read from it.

Writing rules. Saving the settings writes on the scheduled action, and only the values that actually changed:

| Property of the scheduled action | Value written |
|---|---|
| active | true when the master switch is on **and** the mode is `auto` |
| interval unit | the chosen unit |
| interval number | the chosen number |
| next execution instant | the chosen next execution date, or the value already on the scheduled action when the chosen one is empty |

Changing the unit or the number recomputes the next execution date as `now + number × unit`, and refuses two cases:

| Condition on the number | Refusal message |
|---|---|
| the number is zero or below | "Repeat frequency should be positive." |
| the number is one hundred or above | "Invalid repeat frequency. Consider changing frequency type instead of using large numbers." |

**Worked example.** At the second of November at 10:00, a user switches the master switch on, chooses `auto`, `hours` and `19`. The next execution date becomes the second of November at 05:00 the following day, that is nineteen hours later, and the scheduled action becomes active. The user then chooses `days` and `2`: the next execution date becomes the fourth of November at 10:00. The user then types the first of November at 10:00 directly: that value is kept as is, because typing the date does not recompute it. The user then switches the mode to `manual`: the scheduled action becomes inactive and the next execution date stays at the first of November at 10:00. The user then switches the master switch off while leaving the mode at `auto`: the scheduled action stays inactive, because the active flag is the conjunction of the two.

### 1.3 Team settings

| Field | Type | Meaning |
|---|---|---|
| `assignment_optout` | boolean | When true, the scheduled run skips the team entirely. A manual run started on that team still works; the assignment button of the settings screen skips it (`LEAD-106`). |
| `assignment_domain` | condition expression, tracked | The extra condition a Lead must satisfy to be allocated to this team. Empty means no extra condition. Validated by `LEAD-084`. |
| `assignment_max` | integer, derived | The sum of the capacities of the active memberships. A team with a capacity of zero never receives anything. |
| `assignment_enabled` | boolean, derived | True when the master switch is on. |
| `assignment_auto_enabled` | boolean, derived | True when the master switch is on and the scheduled action is active. |
| `lead_unassigned_count` | integer, derived | The number of Leads of the team with no salesperson. |
| `lead_all_assigned_month_count` | integer, derived | The sum over the memberships of the leads assigned in the last thirty days. |
| `lead_all_assigned_month_exceeded` | boolean, derived | True when the previous count exceeds the capacity. |

### 1.4 Member settings

| Field | Type | Default | Meaning |
|---|---|---|---|
| `assignment_max` | integer | 30 | The average number of Leads the member can absorb over thirty days. |
| `assignment_optout` | boolean | false | Pauses assignment for this member. |
| `assignment_domain` | condition expression, tracked | empty | The condition a Lead must satisfy for this member to receive it. Validated by `LEAD-085`. |
| `assignment_domain_preferred` | condition expression, tracked | empty | The condition describing the Leads this member should receive first. Validated by `LEAD-086`. |
| `lead_day_count` | integer, derived | | Leads assigned to this member in this team in the last twenty-four hours, archived records included. |
| `lead_month_count` | integer, derived | | The same over thirty days. |

### 1.5 Tuning parameters

| System parameter | Default | Effect |
|---|---|---|
| Assignment delay, in hours | 0 | A Lead is only considered once this many hours have elapsed since its creation, so that other automated processes have time to enrich it first. |
| Assignment commit batch size | 100 | How many assignment steps are grouped in one committed unit of work. |

## 2. Who may run the assignment

The run is refused for a user who is neither a sales administrator nor the system identity:

> "Lead/Opportunities automatic assignment is limited to managers or administrators"

The scheduled action runs under the system identity, which satisfies the rule.

## 3. Entry points

| Entry point | Teams considered | Quota | Creation window |
|---|---|---|---|
| Scheduled action | every team that uses leads or uses opportunities and is not opted out | the remaining daily quota, that is, the base quota minus the leads received in the last twenty-four hours | the last seven days |
| Button on one team or on a selection of teams | the selected teams, opted out or not | the full base quota | unlimited |
| Button on the settings screen | every team that is not opted out | the full base quota | unlimited |

The two variants differ only in those three columns; the algorithm is identical.

## 4. Phase one: allocate Leads to teams

### 4.1 Building the candidate list per team

1. Compute the cut-off instant as the current instant minus the assignment delay expressed in
   hours.
2. For every team in scope, skip the team entirely when its `assignment_max` (lead average
   capacity) is zero.
3. For every remaining team, the candidate list is the set of Leads satisfying all of the following
   at once: the team's assignment condition, when that condition is not empty; a creation instant at
   or before the cut-off; an empty `team_id` (sales team) **and** an empty `user_id` (salesperson);
   a `won_status` (won or lost) other than `won`; and, only when a creation window is in force, a
   creation instant strictly after the current instant minus the window expressed in days.
4. For every candidate, look up its duplicates once and remember them in a cache for the whole run.

Points to note.

- A Lead already attached to a team, or already having a salesperson, is never reconsidered.
- An archived Lead **is** considered, as long as it is not won: the condition is on the won status, not on the active flag. An archived lost Lead has a won status of `lost`, not `won`, and therefore passes the filter; in practice such records are rarely produced without a salesperson.
- The duplicate lookup uses the search of `LEAD-061` on the email of the Lead, with lost records excluded. It is performed once per Lead and cached, so that the loop below never repeats it.
- Teams whose candidate list is empty are still in the population at the start; they are removed the first time they are drawn (see 4.2).

### 4.2 The draw loop

1. The population is the set of teams built above, in any order; the weight of each team is its
   capacity.
2. While the population is not empty, repeat steps 3 to 7.
3. Draw one team from the population with a probability proportional to its weight.
4. Remove from that team's candidate list every Lead already consumed by this run and every Lead
   that no longer exists.
5. When the candidate list of that team is now empty, remove the team from the population and from
   the weights and continue with the next draw.
6. Otherwise take the first remaining candidate of that team and allocate it to the team,
   deduplicating on the way as described in section 4.3. Mark that Lead, its merge survivor and its
   merged-away duplicates as consumed.
7. At every commit batch boundary, delete the accumulated merged-away records and commit.
8. When the population is empty, delete the remaining merged-away records and commit.

Drawing one Lead at a time, rather than a slice per team, is what makes teams with overlapping conditions share the available Leads in proportion to their capacities. A team whose condition matches everything and a team whose condition matches a niche both draw from the same pool; the niche team simply runs out of candidates earlier and leaves the population.

### 4.3 Allocating one Lead with deduplication

**Inputs**: one candidate Lead and the team it was drawn for.

1. Take the cached duplicate set of that Lead, restricted to the records that still exist.
2. When the duplicate set holds more than one record: write the team into `team_id` on the
   candidate **before** merging; merge the duplicate set with no size limit, no forced salesperson,
   no forced team and without deleting the merged-away records; classify the survivor as *merged*;
   classify the other records of the duplicate set as *duplicates*, to be deleted later.
3. Otherwise write the team into `team_id` on the candidate and classify the candidate as
   *assigned*.

The order of the two steps matters and must be reproduced. Writing the team on the candidate **before** the merge means that, when the merge elects a different survivor, the merged values follow the precedence of `LEAD-074`: the first non-empty team in confidence order wins. An existing opportunity that already had a team and a salesperson therefore keeps them, and the new duplicate lead is absorbed into it.

The merge itself uses no size limit, because the caller is the system identity, so `LEAD-067` does not apply.

### 4.4 The result of phase one

For each team, three sets are recorded:

| Set | Content |
|---|---|
| `assigned` | the Leads attached to the team directly, with no duplicate found |
| `merged` | the survivors of a merge, attached to the team |
| `duplicates` | the records merged away, which are deleted before the method returns |

## 5. Phase two: distribute Leads to members and convert them

### 5.1 Building the candidate list per team

1. Keep the teams in scope that have at least one membership.
2. Compute the daily quota of every membership of those teams, as in section 5.2.
3. Group by team the Leads whose `user_id` is empty, whose `date_open` (assignment date) is empty
   and whose `team_id` is one of those teams.

The condition on the assignment date excludes a Lead that once had a salesperson and lost it: such a record is considered already handled and is not redistributed.

### 5.2 The quota

```formula
base quota of a member = round_half_up_to_integer( member capacity ÷ 30 )

quota of a member = base quota                                        , for a manual run
quota of a member = base quota − leads received in the last 24 hours  , for a scheduled run
```

A member is eligible when it is not paused and its quota is strictly positive. Archived memberships are not part of the member list at all.

### 5.3 Ordering the members

1. The eligible memberships are those whose `assignment_optout` (pause assignment) is false and
   whose quota is strictly positive.
2. Order them by quota descending, breaking ties on a fresh random number drawn for this run.
3. When the ordered list is empty, skip this team entirely.

The random tie-breaker is required for fairness: without it, the membership created first would always come first and would take the only Lead of every daily run.

### 5.4 The preference pass

1. The preferring members are the eligible members whose preference condition is set and not empty.
2. For each preferring member, the preferred set is the set of Leads of the team satisfying **both**
   that member's assignment condition and that member's preference condition.
3. The preferred pool is the concatenation of those preferred sets, in the order of the member list.
4. For each Lead of the preferred pool, ordered by probability descending and then by identifier,
   give the Lead to the first preferring member whose preferred set contains it, as described in
   section 5.6.

A Lead that several members prefer appears several times in the concatenated pool; the assignment step is a no-operation the second time, because the Lead is no longer among the candidates of the remaining members.

### 5.5 The main pass

1. The remaining Leads are the Leads of the team minus the Leads assigned in the preference pass.
2. For each eligible member, its accepted set is the subset of the remaining Leads satisfying that
   member's assignment condition.
3. For each remaining Lead, ordered by probability descending and then by identifier, give the Lead
   to the first member of the ordered member list whose accepted set contains it, as described in
   section 5.6.

### 5.6 Giving one Lead to one member

1. Convert the Lead into an opportunity, keeping its current Contact, writing the member's user
   into `user_id`, and suppressing the automatic subscription notifications.
2. Remove the member from the ordered list, and from the preference list when it was there.
3. Decrease the member's quota by one.
4. When the remaining quota is still strictly positive, append the member at the end of the ordered
   list, and at the end of the preference list when it was there.

Removing and re-appending is what produces the round robin: the member that has just been served goes to the back of the queue.

### 5.7 After each team

The work is committed at each commit batch boundary and once more at the end of the team, and the in-memory cache is cleared so that a run over many teams does not accumulate every Lead in memory.

## 6. Notifications

### 6.1 The result message

A manual run returns a success notification titled `Leads Assigned` whose body is built from the following parts, in this order, keeping only the parts whose condition holds.

| Part | Condition | Text |
|---|---|---|
| 1 | at least one record was merged away | `<count> duplicates leads have been merged.` |
| 2a | nothing was allocated and nothing was distributed, one single team, capacity zero | `No allocated leads to <team name> team because it has no capacity. Add capacity to its salespersons.` |
| 2b | nothing was allocated and nothing was distributed, one single team, capacity above zero | `No allocated leads to <team name> team and its salespersons because no unassigned lead matches its domain.` |
| 2c | nothing was allocated and nothing was distributed, several teams | `No allocated leads to any team or salesperson. Check your Sales Teams and Salespersons configuration as well as unassigned leads.` |
| 3a | nothing was allocated but something was distributed, one single team | `No new lead allocated to <team name> team because no unassigned lead matches its domain.` |
| 3b | nothing was allocated but something was distributed, several teams | `No new lead allocated to the teams because no lead match their domains.` |
| 3c | something was allocated, one single team | `<count> leads allocated to <team name> team.` |
| 3d | something was allocated, several teams | `<count> leads allocated among <team count> teams.` |
| 4a | nothing was distributed but something was allocated | `No lead assigned to salespersons because no unassigned lead matches their domains.` |
| 4b | something was distributed | `<count> leads assigned among <member count> salespersons.` |

The allocated count is the number of directly attached Leads plus the number of merge survivors. The distributed count is the number of Leads given to a member.

### 6.2 The note on the team

A manual run also logs one note per team in scope:

> "Lead Assignment requested by *the acting user's name*"
>
> followed by the same message parts, one per line.

A scheduled run logs nothing and returns nothing.

## 7. Worked example A: ten Leads, three members with capacities

### 7.1 The set-up

One team, `Direct Sales`, with an empty assignment condition, three active members and no preference condition anywhere.

| Member | User | Capacity | Base quota | Assignment condition |
|---|---|---|---|---|
| M1 | Alice | 90 | `round_half_up(90 ÷ 30) = 3` | empty |
| M2 | Bob | 60 | `round_half_up(60 ÷ 30) = 2` | empty |
| M3 | Carol | 30 | `round_half_up(30 ÷ 30) = 1` | empty |

Team capacity: `90 + 60 + 30 = 180`.

Ten Leads are already attached to the team with no salesperson and no assignment date. Their probabilities, in the order the database returns them, are:

| Lead | Probability |
|---|---|
| L1 | 82 |
| L2 | 75 |
| L3 | 70 |
| L4 | 64 |
| L5 | 58 |
| L6 | 51 |
| L7 | 45 |
| L8 | 30 |
| L9 | 22 |
| L10 | 9 |

The run is manual, so the quotas are the base quotas whatever the members already received today.

### 7.2 The trace

Ordering by quota descending gives `[M1(3), M2(2), M3(1)]`. There is no preference pass. The main pass walks the Leads by probability descending.

| Step | Ordered list before | Lead | Member | Quota after | Ordered list after |
|---|---|---|---|---|---|
| 1 | M1(3), M2(2), M3(1) | L1 | M1 | M1 = 2 | M2(2), M3(1), M1(2) |
| 2 | M2(2), M3(1), M1(2) | L2 | M2 | M2 = 1 | M3(1), M1(2), M2(1) |
| 3 | M3(1), M1(2), M2(1) | L3 | M3 | M3 = 0 | M1(2), M2(1) |
| 4 | M1(2), M2(1) | L4 | M1 | M1 = 1 | M2(1), M1(1) |
| 5 | M2(1), M1(1) | L5 | M2 | M2 = 0 | M1(1) |
| 6 | M1(1) | L6 | M1 | M1 = 0 | empty |
| 7 | empty | L7 | none | | empty |
| 8 | empty | L8 | none | | empty |
| 9 | empty | L9 | none | | empty |
| 10 | empty | L10 | none | | empty |

### 7.3 The result

| Member | Leads received | Count |
|---|---|---|
| Alice | L1, L4, L6 | 3 |
| Bob | L2, L5 | 2 |
| Carol | L3 | 1 |
| nobody | L7, L8, L9, L10 | 4 |

Six Leads were distributed, which is the sum of the daily quotas. Each of the six was converted into an opportunity, received its member as salesperson, kept its customer, and received a conversion date. The four remaining Leads keep their team and wait for the next run.

Notification: `6 leads assigned among 3 salespersons.` preceded by `No new lead allocated to Direct Sales team because no unassigned lead matches its domain.`, because phase one found nothing to allocate: the ten Leads already had a team.

Over thirty consecutive daily runs with a steady supply of Leads, Alice receives ninety, Bob sixty and Carol thirty, which are exactly their monthly capacities.

### 7.4 The same example with a restricting condition

Carol's assignment condition is a probability of at least seventy-five. The trace changes at step three.

| Step | Ordered list before | Lead | First member accepting it | Quota after | Ordered list after |
|---|---|---|---|---|---|
| 1 | M1(3), M2(2), M3(1) | L1 (82) | M1 | M1 = 2 | M2(2), M3(1), M1(2) |
| 2 | M2(2), M3(1), M1(2) | L2 (75) | M2 | M2 = 1 | M3(1), M1(2), M2(1) |
| 3 | M3(1), M1(2), M2(1) | L3 (70) | M3 refuses (70 < 75), M1 accepts | M1 = 1 | M3(1), M2(1), M1(1) |
| 4 | M3(1), M2(1), M1(1) | L4 (64) | M3 refuses, M2 accepts | M2 = 0 | M3(1), M1(1) |
| 5 | M3(1), M1(1) | L5 (58) | M3 refuses, M1 accepts | M1 = 0 | M3(1) |
| 6 | M3(1) | L6 (51) | M3 refuses | unchanged | M3(1) |
| 7 to 10 | M3(1) | L7 to L10 | M3 refuses every one | unchanged | M3(1) |

Result: Alice receives L1, L3 and L5; Bob receives L2 and L4; Carol receives nothing, because no remaining Lead reaches a probability of seventy-five. Five Leads were distributed. Carol stays at the head of the list with her quota intact.

### 7.5 The same example with a preference condition

Carol's assignment condition is empty again, but her preference condition is the tag list contains the tag Strategic. L8 and L9 carry that tag; no other Lead does.

The preference pass runs first, over the Leads matching both Carol's assignment condition (empty, so everything) and her preference condition, ordered by probability descending: L8 (30), then L9 (22).

| Step | Preference list | Ordered list | Lead | Member | Quota after |
|---|---|---|---|---|---|
| p1 | M3(1) | M1(3), M2(2), M3(1) | L8 | M3 | M3 = 0 |
| p2 | empty | M1(3), M2(2) | L9 | none | |

Carol's quota is now zero, so she leaves both lists. The main pass then walks the remaining Leads, which are all of them except L8, by probability descending, over the list `[M1(3), M2(2)]`:

| Step | Ordered list before | Lead | Member | Quota after | Ordered list after |
|---|---|---|---|---|---|
| 1 | M1(3), M2(2) | L1 | M1 | M1 = 2 | M2(2), M1(2) |
| 2 | M2(2), M1(2) | L2 | M2 | M2 = 1 | M1(2), M2(1) |
| 3 | M1(2), M2(1) | L3 | M1 | M1 = 1 | M2(1), M1(1) |
| 4 | M2(1), M1(1) | L4 | M2 | M2 = 0 | M1(1) |
| 5 | M1(1) | L5 | M1 | M1 = 0 | empty |
| 6 to 10 | empty | L6, L7, L9, L10 | none | | |

Result: Alice receives L1, L3 and L5; Bob receives L2 and L4; Carol receives L8. Six Leads distributed, and the strategic Lead L8 went to Carol even though its probability was the third lowest of the batch, which is exactly the purpose of the preference condition. L9, the second strategic Lead, stays unassigned because Carol's quota was exhausted and she was the only member with that preference.

## 8. Worked example B: allocation to three teams

### 8.1 The set-up

| Team | Capacity | Assignment condition |
|---|---|---|
| `Direct Sales` | 75 | empty |
| `Conversion` | 90 | the priority is `1`, `2` or `3` |
| `International` | 135 | the country is not empty |

Total capacity: 300. Draw probabilities: 25 percent, 30 percent and 45 percent.

Six hundred Leads are unclaimed, all created within the creation window. Of those, 420 have a priority above `0` and 400 have a country.

### 8.2 The candidate lists

| Team | Candidate count |
|---|---|
| `Direct Sales` | 600 Leads |
| `Conversion` | 420 Leads |
| `International` | 400 Leads |

The three lists overlap heavily: a Lead with both a priority and a country is a candidate for all three teams.

### 8.3 The draw

Each iteration draws one team and consumes one Lead from its list. Because a consumed Lead is removed from every list, the teams compete for the shared Leads. `International` exhausts its four hundred candidates first, in expectation after about `400 ÷ 0.45 ≈ 889` iterations, at which point it leaves the population and the remaining probabilities become `75 ÷ 165 ≈ 45.5 %` and `90 ÷ 165 ≈ 54.5 %`. Then `Conversion` exhausts its remaining candidates, and `Direct Sales` takes the rest.

Because the draw is random, the exact split differs from run to run; only the expectation is specified. A replacement is behaviourally equivalent when it draws with the same probabilities, consumes one Lead per draw, removes an exhausted team from the population, and stops when the population is empty. Nothing in the specification fixes a particular random sequence.

### 8.4 The outcome

Every one of the six hundred Leads ends up attached to exactly one team, because `Direct Sales` accepts everything and stays in the population until the pool is empty. If `Direct Sales` did not exist, the Leads with neither a priority nor a country would stay unattached.

## 9. Worked example C: deduplication during allocation

### 9.1 The set-up

An existing opportunity `Master` in the team `Direct Sales`, with the salesperson Alice, a customer `Nibbler`, a probability of 50 and the email `contact@nibbler.example.com`. A new Lead `Duplicate` arrives from the website contact form with no team, no salesperson, a probability of 10 and the email `Duplicate Email <contact@nibbler.example.com>`, which normalizes to the same address.

A second team, `Overflow`, with a capacity of 10 and an empty assignment condition, is running the assignment.

### 9.2 The run

1. `Duplicate` is a candidate for `Overflow`: it has no team and no salesperson, is not won, and matches the empty condition. `Master` is not a candidate, because it already has a team.
2. The duplicate lookup for `Duplicate` searches on the normalized address with lost records excluded and returns both `Master` and `Duplicate`, that is, two records.
3. `Overflow` is drawn and picks `Duplicate`.
4. Because the duplicate set holds more than one record, the team `Overflow` is written on `Duplicate` first, and then the set is merged.
5. The confidence order puts `Master` first: it is an opportunity and `Duplicate` is a lead, so the second component of the key decides immediately.
6. The merged values take the first non-empty value in confidence order, so the merged team is the team of `Master`, that is `Direct Sales`, and the merged salesperson is Alice. The team written on `Duplicate` in step 4 loses, which is the intended outcome.
7. `Duplicate` is deleted; its messages, activities and attachments are now on `Master`.
8. `Master` is classified as a merge survivor for the team `Overflow` in the result structure, even though it carries the team `Direct Sales`; the classification is bookkeeping for the notification, not a statement about the final team.

### 9.3 The outcome

`Master` keeps its team `Direct Sales`, its salesperson Alice, its probability of 50, and gains the history of `Duplicate`. `Duplicate` no longer exists. The notification reports `1 duplicates leads have been merged.`

## 10. Worked example D: the fairness of the random tie-breaker

Two members of the same team, both with a capacity of 150, therefore a base quota of five each, with the first membership created before the second. Exactly one Lead arrives per day, and the run happens once a day, twenty-five hours apart so that the twenty-four hour counter is always empty at the moment of the run.

Without the random tie-breaker, the ordered list would always be `[first, second]`, the first member would always be the first accepting member, and after thirty days the first member would hold thirty Leads and the second none.

With the random tie-breaker, the two members have the same quota at the moment of the sort, so their relative order is decided by a fresh random number each day. Over thirty runs, each member receives about half of the Leads. In a run of thirty days with a fixed random seed, the observed split is fifteen and fifteen.

A replacement is behaviourally equivalent when it breaks quota ties at random and independently at each run. It is **not** equivalent when it breaks ties by creation order, by identifier, or by any other deterministic key.

## 11. Manual salesperson assignment

Outside the rule-based algorithm, a user may assign a fixed list of salespeople to a selection of records. This is what the conversion dialogue and the reseller forwarding use. It ignores capacities, conditions and quotas.

1. When no salesperson is given but a team is, write that team into `team_id` on every record and
   stop.
2. Otherwise let the number of salespeople be *k*. For each position *i* from zero to *k* − 1, write
   salesperson number *i* + 1 into `user_id`, together with the team when one is given, on the
   records occupying positions *i*, *i* + *k*, *i* + 2*k* and so forth in the record list.

**Worked example.** Six records `R1` to `R6` and four salespeople `S1` to `S4`:

```
positions 0, 4 → S1     (R1, R5)
positions 1, 5 → S2     (R2, R6)
positions 2    → S3     (R3)
positions 3    → S4     (R4)
```

**Second worked example.** Ten records and three salespeople:

```
positions 0, 3, 6, 9 → S1     (four records)
positions 1, 4, 7    → S2     (three records)
positions 2, 5, 8    → S3     (three records)
```

The first salesperson receives the extra record when the number of records is not a multiple of the number of salespeople.

## 12. What a replacement must reproduce

| Property | Requirement |
|---|---|
| Two phases | Allocation to teams and distribution to members are distinct; the second only considers Leads that already have a team and no salesperson and no assignment date. |
| Weighted draw | One Lead per draw, team probability proportional to capacity, exhausted teams removed from the population. |
| Deduplication before attachment | The team is written on the candidate first, then the duplicate group is merged with no size limit; the survivor keeps its own team and salesperson when it had them. |
| Quota | `round_half_up(capacity ÷ 30)`, minus the last twenty-four hours for a scheduled run, full for a manual run. |
| Fairness | Members with equal quota are ordered at random at each run. |
| Round robin | The member that receives a Lead goes to the back of the queue and leaves it when its quota reaches zero. |
| Preference first | Leads matching a member's preference condition are distributed in a dedicated pass before everything else. |
| Order of Leads | Within each pass, by probability descending, then by identifier. |
| Conversion | Every Lead given to a member becomes an opportunity, keeps its customer, and produces no subscription notification. |
| Batched commits | Work is committed at regular intervals so that a failure late in the run does not discard the earlier work. |
| Permission | Only a sales administrator or the system identity may run it. |
